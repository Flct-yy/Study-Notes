# JS手撕：性能优化、渲染技巧与定时器实现

在前端开发中，我们经常会遇到「大量数据渲染卡顿」「频繁事件触发导致性能损耗」「自定义定时逻辑」等问题，今天就来拆解7个高频实用的JS代码片段，用「通俗话术+专业解析」的方式，讲懂每一行代码的作用、核心原理和实际应用场景，帮你吃透这些前端必备技能。

## 一、万条数据渲染优化：避免卡顿的核心技巧

前端渲染大量数据（比如10万条）时，直接一次性插入DOM会导致主线程阻塞，页面出现卡顿、掉帧，甚至浏览器崩溃。这段代码的核心思路是「分批渲染+文档片段+requestAnimationFrame」，从根源上减少DOM操作带来的性能损耗。

### 核心代码（带详细注释）

```js
// 延迟0ms执行，让DOM先渲染完毕，避免阻塞主线程
// 这里的setTimeout(fn, 0)不是真的延迟，而是把回调放到宏任务队列，等待当前同步代码和DOM渲染完成后执行
setTimeout(() => {
  // 总共需要渲染10万条数据（实际开发中可根据需求调整）
  const total = 100000;
  // 每一批次渲染20条，防止一次性渲染过多导致卡顿
  // 批次大小可优化：一般建议20-50条，过多仍会卡顿，过少则渲染次数过多
  const once = 20;
  // 计算总共需要分多少批次（向上取整，避免最后一批数据不足20条被遗漏）
  const loopCount = Math.ceil(total / once);
  // 记录当前已经渲染了多少批次，用于判断是否渲染完成
  let countOfRender = 0;
  // 获取页面中的ul容器，所有li都会插入到这个容器中
  const ul = document.querySelector('ul');

  // 每一批次添加DOM的函数：核心是「减少DOM操作次数」
  function add() {
    // 创建文档片段（DocumentFragment），临时存放当前批次的li
    // 重点：fragment不属于页面DOM树，向它添加子元素不会触发页面重排重绘，相当于“临时仓库”
    const fragment = document.createDocumentFragment();

    // 循环生成当前批次的20个li
    for (let i = 0; i < once; i++) {
      const li = document.createElement('li');
      // 给li填充随机数字（实际开发中可替换为真实业务数据，比如列表项内容）
      li.innerText = Math.floor(Math.random() * total);
      // 先放入片段中，此时不触发任何页面渲染
      fragment.appendChild(li);
    }

    // 一次性将20条li插入ul，只触发一次DOM重排（关键优化点）
    // 对比：如果每次循环都appendChild到ul，会触发20次重排，性能极差
    ul.appendChild(fragment);
    // 已渲染批次+1，更新渲染进度
    countOfRender += 1;

    // 继续执行下一批渲染
    loop();
  }

  // 控制渲染节奏：使用requestAnimationFrame，跟随浏览器刷新频率执行
  // 浏览器每秒刷新约60次（16.67ms/帧），requestAnimationFrame会在每帧开始时执行回调
  // 好处：避免渲染操作与浏览器刷新冲突，保证页面流畅不卡顿
  function loop() {
    // 如果还没渲染完所有批次，就继续下一批
    if (countOfRender < loopCount) {
      window.requestAnimationFrame(add);
    }
  }

  // 启动第一轮渲染
  loop();
}, 0);
```

### 关键知识点解析

- **setTimeout(fn, 0)**：不是延迟执行，而是将回调推入宏任务队列，确保当前同步代码和DOM初始化完成后再执行，避免DOM未挂载时查询不到ul容器。

- **分批渲染**：将10万条数据拆分为5000批次（100000/20），每次只渲染20条，降低单次DOM操作的压力。

- **DocumentFragment**：前端性能优化神器，作为临时DOM容器，所有操作都在内存中完成，最后一次性插入页面，只触发1次重排重绘，比直接操作真实DOM性能提升10倍以上。

- **requestAnimationFrame**：与setTimeout相比，它能跟随浏览器刷新频率执行，避免“掉帧”，尤其适合大量DOM渲染、动画等场景，保证页面流畅度。

### 实际应用场景

大数据列表渲染（如后台管理系统的订单列表、日志列表）、长列表滚动加载（结合滚动事件，滚动到底部时加载下一批），避免一次性渲染大量数据导致页面卡死。

## 二、手撕防抖：解决频繁触发的“性能杀手”

防抖（Debounce）的核心逻辑：**频繁触发同一事件时，只在最后一次触发后，延迟指定时间执行回调函数**。比如搜索框输入、窗口resize、滚动事件，频繁触发会导致性能损耗，防抖能有效“合并”触发次数。

### 1. 基础版防抖（延迟执行）

```js
// 防抖：频繁触发时，只在**最后一次触发后延迟执行**
function debounce(callback, wait) {
  // 定时器标识：用闭包保存，避免污染全局变量，且能在多次调用中共享状态
  let timer = null;

  // 返回一个可调用的包装函数，接收原函数的参数
  return function (...args) {
    // 保存原this（解决事件回调中this指向window的问题，比如btn点击事件中this应指向btn）
    const context = this;

    // 再次触发时，清除之前的定时器 → 重新计时（核心：取消上一次的延迟执行）
    if (timer) clearTimeout(timer);

    // 新建定时器：延迟 wait 毫秒后执行回调
    timer = setTimeout(() => {
      // 恢复原函数的this指向和参数，保证回调函数执行时上下文正确
      callback.apply(context, args);
      // 执行完清空timer，方便垃圾回收，避免内存泄漏
      timer = null;
    }, wait);
  };
}
```

### 2. 完整版防抖（支持立即执行/延迟执行）

基础版防抖是“延迟执行”（触发后等待wait时间再执行），但实际开发中有时需要“立即执行”（第一次触发就执行，之后频繁触发不执行，直到wait时间后才可再次执行），比如按钮点击防重复提交。

```js
// 完整版防抖：支持 立即执行(immediate=true) / 延迟执行(immediate=false)
function debounce(callback, wait, immediate) {
  let timer = null; // 闭包缓存定时器，共享状态

  return function () {
    // 每次进入先清除上一次定时器 → 重新计时（无论立即还是延迟，都要取消上一次）
    if (timer) clearTimeout(timer);

    // ========== 立即执行模式 ==========
    if (immediate) {
      // timer为null时表示可以立即执行（首次触发或wait时间已过）
      const callNow = !timer;

      // 设置定时器：wait时间后把timer置空，解锁“立即执行”权限
      // 作用：这段时间内再次触发，callNow会为false，不会执行回调
      timer = setTimeout(() => {
        timer = null;
      }, wait);

      // 满足立即执行条件时，调用原函数，恢复this和参数
      if (callNow) {
        callback.apply(this, arguments);
      }

    // ========== 常规延迟执行模式 ==========
    } else {
      // 延迟wait时间执行，每次触发都重置定时器
      timer = setTimeout(() => {
        callback.apply(this, arguments);
        timer = null; // 执行后清空，垃圾回收
      }, wait);
    }
  };
}
```

### 关键知识点解析

- **闭包的作用**：保存timer变量，让多次触发的事件能共享同一个定时器标识，实现“清除上一次定时器”的逻辑，这是防抖的核心。

- **this指向修复**：事件回调中this默认指向window（如addEventListener中的回调），通过context = this + apply(context, args)，让原函数this指向正确的元素（如按钮、输入框）。

- **立即执行vs延迟执行**：
        

    - 立即执行（immediate=true）：适合防重复提交（按钮点击后立即执行，wait时间内不可再次点击）；

    - 延迟执行（immediate=false）：适合搜索框联想（输入停止后wait时间，再发送请求，避免频繁请求接口）。

### 实际应用场景

搜索框输入联想、窗口resize事件（调整窗口大小时，避免频繁计算布局）、滚动事件（滚动到底部加载更多，避免频繁触发）、按钮防重复提交。

## 三、手撕节流：固定频率执行，避免过度触发

节流（Throttle）的核心逻辑：**频繁触发同一事件时，按照固定的时间间隔执行回调函数**，无论触发多少次，都不会超过这个频率。和防抖的区别：防抖是“最后一次触发后执行”，节流是“固定频率持续执行”。

### 1. 立即触发版节流（停止触发后不执行最后一次）

```js
// 节流：固定频率执行，立即触发，停止触发后不执行最后一次
function throttle(callback, wait) {
  // 上一次执行回调的时间戳（初始为0，确保第一次触发能立即执行）
  let previous = 0;

  return function(...args) {
    // 获取当前时间戳
    const now = Date.now();
    // 核心逻辑：当前时间 - 上一次执行时间 >= 等待时间，才执行回调
    if (now - previous >= wait) {
      // 执行回调，恢复this和参数
      callback.apply(this, args);
      // 更新上一次执行时间戳为当前时间，开始下一个周期
      previous = now;
    }
  };
}
```

### 2. 延迟触发版节流（停止触发后仍执行最后一次）

立即触发版节流的问题：如果停止触发时，距离上一次执行已经超过wait时间，不会执行最后一次触发的回调。延迟触发版可以解决这个问题，适合需要“收尾”的场景（如滚动加载，即使停止滚动，也要执行最后一次加载逻辑）。

```js
// 节流：固定频率执行，延迟触发，停止触发后仍执行最后一次
function throttle(callback, wait) {
  let timer = null; // 用定时器控制延迟执行

  return function(...args) {
    const context = this;
    // 核心逻辑：没有定时器才创建，不重置计时（保证固定频率）
    if (!timer) {
      timer = setTimeout(() => {
        // 延迟wait时间执行回调
        callback.apply(context, args);
        // 执行后清空定时器，允许下一次创建
        timer = null;
      }, wait);
    }
  };
}
```

### 关键知识点解析

- **时间戳版（立即触发）**：通过对比当前时间和上一次执行时间，控制执行频率，优点是简单高效，缺点是停止触发后不会执行最后一次。

- **定时器版（延迟触发）**：通过定时器控制执行时机，优点是停止触发后仍会执行最后一次，缺点是首次触发会延迟wait时间才执行。

- **防抖vs节流**：
        

    - 防抖：合并多次触发，只执行最后一次（比如搜索输入）；

    - 节流：控制触发频率，固定间隔执行（比如滚动加载、鼠标移动绘制）。

### 实际应用场景

滚动事件（监听滚动位置，固定频率更新导航栏状态）、鼠标移动事件（绘制canvas，避免频繁重绘）、resize事件（固定频率调整页面布局）、高频点击事件（如游戏中的攻击按钮，控制点击频率）。

## 四、自定义定时器：可递增延迟的MySetInterval

原生setInterval的缺点：间隔时间固定，无法实现“每次执行延迟递增”的需求（比如第一次延迟100ms，第二次延迟200ms，第三次延迟300ms...）。这段代码通过类封装，实现了“基础延迟+递增步长”的自定义定时器，灵活满足复杂定时需求。

### 核心代码（带详细注释）

```js
class MySetInterval {
  /**
   * @param {Function} fn 要执行的函数（回调函数）
   * @param {number} base 基础延迟 a（第一次执行的延迟时间）
   * @param {number} step 每次递增 b（每次执行的延迟比上一次多b ms）
   * @param  {...any} args 传递给 fn 的参数（可选）
   */
  constructor(fn, base, step, ...args) {
    this.fn = fn;         // 要执行的回调函数
    this.base = base;     // 基础延迟时间（ms）
    this.step = step;     // 延迟递增步长（ms）
    this.args = args;     // 传递给回调函数的参数
    this.count = 0;       // 记录执行次数（用于计算当前延迟）
    this.timer = null;    // 定时器ID，用于停止定时器
  }

  // 启动定时器
  start() {
    // 计算当前批次的延迟：a + count * b（第一次count=0，延迟a；第二次count=1，延迟a+b，以此类推）
    const delay = this.base + this.count * this.step;

    // 用setTimeout模拟递归执行，实现“递增延迟”
    this.timer = setTimeout(() => {
      // 执行用户传入的回调函数，并传递参数
      this.fn(...this.args);
      // 执行次数+1，为下一次延迟计算做准备
      this.count++;
      // 递归调用start，启动下一次定时执行
      this.start();
    }, delay);
  }

  // 停止定时器（必须有，避免内存泄漏）
  stop() {
    // 清除当前定时器
    clearTimeout(this.timer);
    // 清空timer，方便垃圾回收，也避免重复停止
    this.timer = null;
  }
}

// 使用示例
const timer = new MySetInterval(() => {
  console.log('自定义定时器执行');
}, 100, 50); // 第一次延迟100ms，第二次150ms，第三次200ms...
timer.start(); // 启动
// timer.stop(); // 停止（需要时调用）
```

### 关键知识点解析

- **类封装优势**：通过class封装，将定时器的状态（count、timer、base等）挂载到实例上，避免全局变量污染，同时方便调用start和stop方法，逻辑更清晰。

- **递增延迟实现**：通过count记录执行次数，每次执行后count+1，下一次延迟 = 基础延迟 + count * 递增步长，实现“每次延迟递增”的效果。

- **递归setTimeout**：没有使用原生setInterval，而是用setTimeout递归调用start方法，避免setInterval可能出现的“时间漂移”（比如回调执行时间过长，导致下一次执行延迟偏差）。

### 实际应用场景

倒计时递增（比如活动倒计时，后期每秒增加延迟，营造紧迫感）、轮播图渐变（每次切换的延迟递增，实现慢放效果）、接口重试（失败后重试，每次重试延迟递增，避免频繁请求接口）。

## 五、重写setTimeout：用requestAnimationFrame模拟

原生setTimeout的缺点：执行时间不精确，受主线程阻塞影响（比如主线程有耗时操作，setTimeout的回调会延迟执行）。而requestAnimationFrame（rAF）会跟随浏览器刷新频率执行（16.67ms/帧），用它模拟setTimeout，能让延迟执行更精确，同时避免主线程阻塞导致的偏差。

### 核心代码（带详细注释）

```js
// 用 requestAnimationFrame 模拟 setTimeout，提升执行精度
let setTimeout = (fn, timeout, ...args) => {
  const start = Date.now(); // 记录定时器启动的时间戳
  let timer;               // 保存rAF的标识，用于取消定时器

  // 循环执行函数，每帧检查是否达到设定的延迟时间
  const loop = () => {
    // 注册下一次rAF回调，保证循环执行，跟随浏览器刷新频率
    timer = window.requestAnimationFrame(loop);
    // 获取当前时间戳
    const now = Date.now();

    // 核心逻辑：当前时间 - 启动时间 >= 设定的延迟时间，执行回调
    if (now - start >= timeout) {
      // 执行用户传入的回调函数，恢复this和参数
      fn.apply(this, args);
      // 执行完成后，取消rAF循环，避免无限执行
      window.cancelAnimationFrame(timer);
    }
  };

  // 启动rAF循环，开始计时
  window.requestAnimationFrame(loop);
};

// 使用示例（和原生setTimeout用法一致）
setTimeout(() => {
  console.log('用rAF模拟的setTimeout执行');
}, 1000);
```

### 关键知识点解析

- **精度提升原理**：原生setTimeout的延迟是“最小延迟”，如果主线程忙碌，回调会被推迟；而rAF每帧（16.67ms）执行一次loop，每次都检查时间差，一旦达到设定延迟就执行回调，精度更高。

- **循环终止**：通过cancelAnimationFrame(timer)取消rAF循环，避免回调执行后仍继续循环，造成性能损耗。

- **用法兼容**：模拟后的setTimeout用法和原生一致，无需修改现有代码，直接替换即可提升执行精度。

### 实际应用场景

需要精确延迟执行的场景（如动画同步、定时更新DOM）、避免主线程阻塞导致延迟偏差的场景（如复杂页面中的定时任务）。

## 六、模拟sleep函数：让代码“暂停”指定时间

JS中没有原生的sleep函数（即“暂停代码执行指定时间，再继续执行后面的代码”），但可以通过Promise+setTimeout模拟。核心思路：返回一个Promise，在setTimeout延迟后resolve，通过await等待Promise完成，实现代码“暂停”效果。

### 核心代码（带详细注释）

```js
// 休眠函数：等待 time 毫秒后，再继续执行后面的代码
// 核心：通过Promise包裹setTimeout，用resolve触发后续代码执行
function sleep(time) {
  // 返回一个Promise对象，pending状态表示“正在休眠”
  return new Promise(function (resolve) {
    // 定时 time 毫秒后，执行resolve()，让Promise变为fulfilled状态
    // resolve()无参数，仅用于通知“休眠结束”
    setTimeout(resolve, time);
  });
}

// 使用示例（必须配合async/await，因为await只能在async函数中使用）
async function test() {
  console.log('开始执行');
  await sleep(2000); // 暂停2000ms（2秒）
  console.log('2秒后执行'); // 2秒后才会打印这句话
  await sleep(1000); // 再暂停1秒
  console.log('再1秒后执行');
}
```

### 关键知识点解析

- **Promise的作用**：用Promise包裹setTimeout，将“延迟执行”转化为“异步等待”，配合await使用，实现代码的“线性暂停”，避免回调地狱。

- **async/await依赖**：sleep函数返回Promise，必须在async函数中用await调用，才能实现“暂停”效果；如果不用await，代码会继续执行，不会暂停。

- **非阻塞特性**：sleep是异步暂停，不会阻塞主线程，其他异步任务（如接口请求、DOM渲染）可以在sleep期间正常执行，避免页面卡顿。

### 实际应用场景

代码分步执行（如引导页步骤切换，每步间隔1秒）、接口请求重试（失败后sleep1秒再重试）、模拟加载动画（sleep指定时间后隐藏加载框）。

## 七、版本号对比：实现语义化版本排序

在前端开发中，经常需要对版本号进行排序（如npm包版本、项目版本），版本号格式通常为“x.y.z”（如1.0.0、2.3.4、1.10.2），直接字符串排序会出现错误（如1.10.2会排在1.2.0前面），这段代码能正确对比版本号大小并排序。

### 核心代码（带详细注释）

```js
// 版本号对比排序：接收版本号数组，返回按升序排列的数组
var compareVersion = function (versions) {
  // 用数组的sort方法排序，核心是自定义排序规则
  return versions.sort((version1, version2) => {
    // 1. 将版本号按“.”分割，转为数字数组（如"1.10.2" → [1,10,2]）
    // map(Number)：将分割后的字符串转为数字，避免字符串比较的误差
    let s1 = version1.split('.').map(Number);
    let s2 = version2.split('.').map(Number);

    // 2. 逐位对比版本号（从左到右，依次对比主版本、次版本、修订版本）
    // 循环次数取两个版本号数组的最大长度，不足的位补0（如1.0 → [1,0,0]）
    for (let i = 0; i < s1.length || i < s2.length; i++) {
      const v1 = s1[i] || 0; // 版本1当前位的值，无则补0
      const v2 = s2[i] || 0; // 版本2当前位的值，无则补0
      // 若当前位不相等，直接返回差值（正数表示v1>v2，负数表示v1<v2）
      if (v1 !== v2) {
        return v1 - v2;
      }
    }

    // 3. 若所有位都相等，按原字符串排序（处理版本号格式完全一致的情况）
    return version1.localeCompare(version2);
  })
};

// 使用示例
const versions = ['1.10.2', '1.2.0', '2.3.4', '1.0.0', '1.5'];
console.log(compareVersion(versions)); 
// 输出：["1.0.0", "1.2.0", "1.5", "1.10.2", "2.3.4"]
```

### 关键知识点解析

- **版本号分割与转数字**：split('.')将版本号分割为数组，map(Number)转为数字数组，避免“10”作为字符串比“2”小的问题（字符串比较是按字符编码，'10' < '2'）。

- **逐位对比逻辑**：从左到右对比每一位版本号，先对比主版本（第一位），主版本大的版本号更大；主版本相等则对比次版本（第二位），以此类推；不足的位补0（如1.5 → 1.5.0）。

- **兜底逻辑**：若所有位都相等（如1.0.0和1.0.0），用localeCompare按字符串排序，保证排序的稳定性。

### 实际应用场景

npm包版本排序、后台管理系统的版本日志排序、APP版本更新提示（对比当前版本和最新版本，判断是否需要更新）。

## 总结

以上7个代码片段，覆盖了前端开发中「性能优化」「事件处理」「定时任务」「版本对比」四大核心场景，每段代码都包含“核心逻辑+详细注释+知识点解析+实际应用”，既能直接复制使用，也能帮你理解背后的原理。

重点记住：前端性能优化的核心是「减少DOM操作」「避免频繁触发事件」「合理利用异步」，而防抖、节流、分批渲染、rAF都是实现这些目标的关键手段；自定义定时器、sleep、版本对比则是解决实际业务场景的实用工具，掌握这些，能让你的代码更高效、更健壮。