# JS手写：DOM操作 & 浏览器API高频场景详解

在前端开发中，我们经常会遇到一些重复且基础的需求——比如解析URL参数、给大量元素绑定点击事件、实现图片懒加载等。这些功能看似简单，但写得不够严谨就容易出现bug（比如中文参数乱码、事件绑定冗余、滚动加载卡顿）。

今天就整理了7个前端高频实用JS功能，用“通俗话+专业解析”的方式，把每个功能的原理、代码细节和使用场景讲透，还附上了可直接复制使用的优化版代码，新手也能快速套用。

## 一、URL参数解析：把URL里的参数“拆”成可直接用的对象

### 通俗解读

我们经常会看到这样的URL：`https://xxx.com/list?page=1&size=10&keyword=前端`，里面的page、size、keyword就是参数。这个功能就是把这些参数“拆出来”，变成一个对象（比如{page:1, size:10, keyword:"前端"}），不用我们手动去切割字符串，省心又不易错。

### 专业解析

核心利用浏览器原生的`URL`对象，它能自动解析URL的协议（http/https）、主机（xxx.com）、路径（/list）和查询参数（?后面的内容）；再配合`URLSearchParams`处理查询参数，同时解决中文乱码（用decodeURIComponent解码）、空值、重复key、数字类型转换等常见问题。

### 完整代码（可直接复制）

```js
function parseParam(url) {
  // 创建URL对象，自动解析协议、域名、路径、参数（原生API，无需手动切割）
  const urlObj = new URL(url);
  // 获取查询参数部分（即?后面的内容，不含?）
  const queryParams = new URLSearchParams(urlObj.search);
  const paramsObj = {};

  // 遍历所有查询参数，处理各种边界情况
  for (let [key, value] of queryParams.entries()) {
    // 空值处理：如果参数值是空字符串或null，统一赋值为true（比如?flag&name=xxx，flag对应true）
    if (value === '' || value == null) {
      paramsObj[key] = true;
    } else {
      // 解码参数：处理中文、特殊字符（比如%E5%89%8D%E7%AB%AF解码为“前端”）
      let val = decodeURIComponent(value);
      // 纯数字字符串转为数字类型（比如"123"转为123，避免后续使用时还要手动转换）
      val = /^\d+$/.test(val) ? parseFloat(val) : val;

      // 重复key处理：如果参数有多个相同key（比如?tag=js&tag=html），转为数组形式
      if (paramsObj.hasOwnProperty(key)) {
        paramsObj[key] = [].concat(paramsObj[key], val);
      } else {
        paramsObj[key] = val;
      }
    }
  }

  return paramsObj;
}

// 示例使用
const url = "https://api.example.com/data?id=123&name=张三&page=2&tag=js&tag=html&flag";
const params = parseParam(url);
console.log(params);
// 输出：{id: 123, name: "张三", page: 2, tag: ["js", "html"], flag: true}
```

### 关键注意点

- URL对象是浏览器原生API，无需引入第三方库，兼容现代浏览器（IE不支持，需兼容IE可额外处理）；

- 中文参数必须用decodeURIComponent解码，因为URL中中文会被自动编码为%开头的字符；

- 重复key（如?tag=js&tag=html）处理为数组，避免后面的参数覆盖前面的。

## 二、事件委托：一次绑定，搞定所有子元素的事件

### 通俗解读

如果页面有100个按钮，给每个按钮都绑定点击事件，会占用很多内存，而且新增按钮还得重新绑定。事件委托就是“找一个父容器”，只给父容器绑定一次事件，不管里面有多少个子元素（哪怕是后来新增的），点击子元素时都会触发父容器的事件，再通过判断点击的是哪个子元素，执行对应逻辑。

### 专业解析

利用DOM事件的“冒泡机制”（子元素的事件会向上传递给父元素），将事件绑定在父容器上，通过event.target获取真正被点击的子元素，再通过matches()方法匹配目标元素选择器，实现“一次绑定，多元素复用”，优化性能并简化代码。

### 完整代码（可直接复制）

```js
/**
 * 事件委托（代理）
 * @param {string} eventType 事件类型 click/input 等（比如"click"、"input"）
 * @param {string|Element} elDelegate 委托父元素（选择器字符串或DOM对象）
 * @param {string} selector 真正要触发的目标元素选择器（比如"#btn"、".item"）
 * @param {Function} fn 触发的回调函数（this指向目标元素）
 */
function on(eventType, elDelegate, selector, fn) {
  // 1. 处理委托父元素：如果传入的是选择器字符串，自动转为DOM对象
  if (!(elDelegate instanceof Element) && typeof elDelegate === 'string') {
    elDelegate = document.querySelector(elDelegate);
  }

  // 安全判断：如果没找到父元素，直接退出，避免报错
  if (!elDelegate) return null;

  // 2. 给父元素绑定事件，利用事件冒泡机制
  elDelegate.addEventListener(eventType, (e) => {
    let el = e.target; // 真正被点击/触发的元素（子元素）

    // 3. 向上查找匹配selector的元素（防止点击的是子元素的子节点）
    while (el && !el.matches(selector)) {
      if (el === elDelegate) { // 查到委托父元素还没匹配到，说明不是目标元素，停止查找
        el = null;
        break;
      }
      el = el.parentNode; // 向上查找父级节点
    }

    // 4. 如果找到目标元素，执行回调，this指向目标元素
    el && fn.call(el, e, el);
  });

  return elDelegate;
}

// HTML示例
/*

*/

// 示例使用1：单个目标元素
on('click', '#box', '#btn', function(e, el){
  console.log('点击成功！');
  console.log(this); // this 指向 #btn（目标元素）
  console.log(el); // el 也是目标元素，和this一致
});

// 示例使用2：多个目标元素（.item）
on('click', '#box', '.item', function(e, el){
  console.log('点击了item按钮：', el.innerText);
});
```

### 关键注意点

- 委托父元素必须是目标元素的祖先节点（比如按钮的父div、body、document）；

- 不要阻止事件冒泡（e.stopPropagation()），否则事件无法传递到父元素，委托失效；

- 新增的子元素（比如通过JS动态添加的按钮），无需重新绑定事件，会自动触发委托的事件。

## 三、滚动加载：滚动到底部自动加载更多内容

### 通俗解读

我们刷朋友圈、逛电商列表时，往下滚动页面，到底部后会自动加载更多内容，这就是滚动加载。核心就是“判断页面是否滚动到底部”，如果到了，就执行加载数据的逻辑。

### 专业解析

通过监听window的scroll事件，获取三个关键高度：可视区域高度（屏幕能看到的页面高度）、滚动条已滚动距离、页面总高度（包括看不见的部分）。核心判断公式：`可视区域高度 + 已滚动距离 ≥ 页面总高度`，满足该条件即表示滚动到底部。

### 完整代码（可直接复制）

```js
// 监听滚动事件
window.addEventListener('scroll', function() {
    // 1. 可视区域高度（屏幕能看到的高度，不同设备可能不同）
    const clientHeight = document.documentElement.clientHeight;
    // 2. 滚动条卷上去的高度（已滚动的距离，兼容不同浏览器）
    const scrollTop = document.documentElement.scrollTop || document.body.scrollTop;
    // 3. 整个页面总高度（包括看不见的部分，即页面完整高度）
    const scrollHeight = document.documentElement.scrollHeight;

    // 核心判断：可视高度 + 已滚动距离 ≥ 总高度 → 滚动到底部（加10是为了提前加载，优化体验）
    if (clientHeight + scrollTop >= scrollHeight - 10) {
        console.log("滚动到底部啦！");
        // 这里写加载更多逻辑（比如请求接口、渲染列表）
        // loadMoreData(); // 假设这是加载更多数据的函数
    }
}, false);

// 优化建议：滚动事件会频繁触发，可结合节流函数（参考后面的图片懒加载），避免性能消耗
// 比如：window.addEventListener('scroll', throttle(handleScroll, 300), false);
```

### 关键注意点

- scroll事件会频繁触发（滚动过程中每秒触发几十次），建议结合节流函数（后面会讲），减少函数执行次数，优化性能；

- 滚动到底部的判断可加一个小偏移量（比如-10），让加载提前触发，避免用户看到底部空白再加载；

- 加载数据时，建议添加“加载中”状态，防止用户多次触发加载。

## 四、图片懒加载：减少页面加载时间，提升体验

### 通俗解读

页面有很多图片时，如果一打开就加载所有图片，会导致页面加载变慢、卡顿。图片懒加载就是“只加载屏幕能看到的图片”，用户往下滚动页面，图片进入视野后再加载，既节省带宽，又提升页面加载速度。

### 专业解析

核心思路：先给图片设置自定义属性（比如data-src）存储真实图片地址，src属性设为占位图（或空）；监听scroll事件，判断图片是否进入可视区域，若进入，则将data-src的值赋给src，实现图片加载。同时用节流函数限制scroll事件触发频率，避免性能消耗。

### 完整代码（可直接复制）

```js
// 节流函数：限制函数在指定时间内只能执行一次（优化scroll事件频繁触发）
function throttle(fn, delay) {
  let timer = null;
  return function (...args) {
    if (!timer) {
      fn.apply(this, args); // 执行函数
      timer = setTimeout(() => {
        timer = null; // 延迟后重置timer，允许下次执行
      }, delay);
    }
  };
}

// 图片懒加载核心函数
function lazyload() {
  const imgs = document.getElementsByTagName('img'); // 获取所有图片元素（注意加s，避免报错）
  const viewHeight = document.documentElement.clientHeight; // 可视区域高度
  // 滚动距离（兼容不同浏览器）
  const scrollTop = document.documentElement.scrollTop || document.body.scrollTop;

  // 遍历所有图片，判断是否进入可视区域
  for (let i = 0; i < imgs.length; i++) {
    const img = imgs[i];
    const offsetTop = img.offsetTop; // 图片到页面顶部的距离

    // 判断：图片顶部距离 ≤ 可视区域高度 + 滚动距离 → 图片进入可视区域
    if (offsetTop < viewHeight + scrollTop) {
      // 优化：已经加载过的图片跳过，避免重复赋值（防止scroll事件重复触发导致的冗余操作）
      if (img.src === img.dataset.src) continue;

      // 开始加载图片：将data-src（真实地址）赋给src
      img.src = img.dataset.src;
    }
  }
}

// 监听scroll事件，用节流函数限制触发频率（300ms执行一次）
window.addEventListener('scroll', throttle(lazyload, 300));

// 页面刚打开时，执行一次懒加载（加载可视区域内的图片）
window.addEventListener('load', lazyload);

// HTML示例
/*
<!-- 占位图用1x1透明图，或loading图片，data-src存储真实图片地址 -->

*
```

### 关键注意点

- 图片必须设置src属性（可设为占位图），否则会出现图片占位空白；

- 节流函数的延迟时间（300ms）可根据需求调整，延迟太短达不到优化效果，太长会影响体验；

- 页面加载完成后（window.load）必须执行一次lazyload，避免可视区域内的图片无法加载。

## 五、统计HTML页面标签：快速了解页面结构

### 通俗解读

有时候我们需要知道一个页面用了多少种标签、每种标签用了多少次（比如做页面优化、排查冗余标签），这个功能就能自动统计，不用手动一个个数，还能按使用次数排序。

### 专业解析

利用`document.getElementsByTagName('*')`获取页面所有元素，转为数组后提取每个元素的标签名；用Set统计标签种类（Set自动去重），用reduce统计每种标签的数量，最后用sort排序，返回标签种类数和各标签数量（从多到少）。

### 完整代码（可直接复制）

```js
function countTagsOnPage() {
  // 1. 获取页面所有元素（*表示匹配所有标签）
  const allTags = document.getElementsByTagName('*');
  // 2. 转为数组，提取每个元素的标签名，并转为大写（统一格式，避免div和DIV重复统计）
  const tagNames = [...allTags].map(el => el.tagName.toUpperCase());
  
  // 3. 统计标签种类（Set自动去重，size就是种类数）
  const totalTagTypes = new Set(tagNames).size;

  // 4. 统计每种标签的数量（用reduce累加）
  const tagCount = tagNames.reduce((acc, tag) => {
    acc[tag] = (acc[tag] || 0) + 1; // 有则加1，无则初始化为1
    return acc;
  }, {});

  // 5. 按标签数量从多到少排序（将对象转为数组，再排序）
  const sorted = Object.entries(tagCount).sort((a, b) => b[1] - a[1]);

  // 返回统计结果：标签种类数、排序后的标签数量
  return {
    totalTagTypes,
    tagCounts: sorted
  };
}

// 使用并打印结果
const res = countTagsOnPage();
console.log('页面标签种类：', res.totalTagTypes);
console.log('各标签数量（从多到少）：', res.tagCounts);
// 示例输出：
// 页面标签种类：8
// 各标签数量（从多到少）：[["DIV", 12], ["SPAN", 8], ["IMG", 5], ["BUTTON", 3], ...]
```

### 关键注意点

- tagName返回的是大写标签名（比如div返回DIV），统一转为大写可避免大小写重复统计；

- document.getElementsByTagName('*')会获取所有元素，包括head、body、script等隐藏元素，若需统计可见元素，可添加筛选条件；

- 排序后的结果是二维数组，每个元素的第一个值是标签名，第二个值是数量。

## 六、点击打印HTML标签名：快速定位元素标签

### 通俗解读

开发时，我们经常需要知道点击的元素是什么标签（比如排查样式问题、调试事件绑定），这个功能就是“点击页面任意元素，自动打印该元素的标签名”，不用手动去开发者工具里查看。

### 专业解析

利用事件委托（前面讲过的知识点），在document上绑定一次click事件，通过event.target获取被点击的具体元素，再用tagName获取该元素的标签名，最后用console.log打印（也可改为弹窗显示）。

### 完整代码（可直接复制）

```js
// 利用事件委托，在document上绑定一次click事件，处理所有元素的点击
document.addEventListener('click', function(event) {
    // event.target 指向被点击的具体元素（最底层子元素）
    const clickedElement = event.target;
    
    // 获取标签名（tagName返回大写形式，如'DIV'、'SPAN'、'IMG'）
    const tagName = clickedElement.tagName;
    
    // 打印标签名（默认控制台打印，可改为弹窗）
    console.log(`点击的元素标签名：${tagName}`);
    // 如需弹窗显示，可取消下面这行的注释
    // alert(`点击的元素标签名：${tagName}`);
});

// 示例：点击页面上的div、span、按钮，控制台会分别打印 DIV、SPAN、BUTTON
```

### 关键注意点

- 点击的是元素的子节点（比如span里的文本），event.target会指向文本节点的父元素（span），不影响标签名获取；

- 可根据需求修改打印方式（控制台打印/弹窗），弹窗适合非开发环境快速查看；

- 若只想打印特定元素的标签名，可添加筛选条件（比如只打印按钮标签：if(tagName === 'BUTTON') { ... }）。

## 七、模拟JSONP：解决跨域请求问题

### 通俗解读

前端请求接口时，经常会遇到“跨域”报错（比如前端域名是a.com，接口域名是b.com），JSONP是一种简单的跨域解决方案。核心就是“通过创建script标签，加载接口地址，利用script标签不受跨域限制的特性，获取后端返回的数据”。

注意：结合你提供的报错信息 `link hit security strategy`（链接触发安全策略），若使用JSONP时出现该报错，大概率是后端接口的安全策略限制了该请求（比如不允许JSONP请求、域名白名单限制），需联系后端调整安全策略。

### 专业解析

JSONP的核心原理：script标签的src属性不受同源策略限制，可加载任意域名的资源。前端生成唯一回调函数名，拼接在接口URL中；后端接收请求后，返回“回调函数名(数据)”的格式；前端通过全局回调函数，接收并处理后端返回的数据，最后删除临时创建的script标签和全局函数，避免冗余。

### 完整代码（可直接复制）

```js
function JSONP(url, _params = {}) {
  // 1. 生成唯一回调函数名（防止多个JSONP请求冲突，默认用jsonp_+时间戳）
  const callbackName = _params.callback || "jsonp_" + Date.now();
  
  // 2. 处理请求参数（排除callback，因为要单独拼接）
  const params = [];
  for (let key in _params) {
    if (key !== "callback") {
      // 编码参数值，处理中文/特殊字符
      params.push(`${key}=${encodeURIComponent(_params[key])}`);
    }
  }
  // 3. 拼接callback参数（JSONP核心：后端会根据该参数返回对应的回调函数调用）
  params.push(`callback=${callbackName}`);

  // 4. 创建script标签，用于加载接口（script不受跨域限制）
  const script = document.createElement("script");
  // 拼接接口URL和参数（url?key1=value1&key2=value2&callback=xxx）
  script.src = `${url}?${params.join("&")}`;

  // 5. 返回Promise，方便用then/catch处理结果
  return new Promise((resolve, reject) => {
    
    // 6. 挂载全局回调函数（必须在script加载前定义，否则后端返回时函数还不存在）
    window[callbackName] = (result) => {
      try {
        resolve(result); // 成功：将后端返回的数据传入resolve
      } catch (err) {
        reject(err); // 失败：捕获异常并传入reject
      } finally {
        // 7. 清理工作：删除script标签和全局回调函数，避免内存泄漏
        document.body.removeChild(script);
        delete window[callbackName];
      }
    };

    // 8. 处理脚本加载失败（比如网络错误、接口不存在）
    script.onerror = () => {
      reject(new Error("JSONP 请求失败"));
      // 失败也需要清理
      document.body.removeChild(script);
      delete window[callbackName];
    };

    // 9. 把script插入页面（最后执行，确保回调函数已定义）
    document.body.appendChild(script);
  });
}

// 示例使用（结合你提供的URL）
JSONP("https://api.example.com/data", {
  id: 123,
  callback: "getData" // 可选：指定后端约定的回调名，不指定则自动生成
}).then(res => {
  console.log("拿到数据：", res);
}).catch(err => {
  console.log("出错：", err);
  // 若出现 "link hit security strategy" 报错，需检查后端安全策略
});

// 后端返回格式（必须是回调函数调用的形式）
// getData({ "name": "张三", "id": 123 })
// 前端会通过window.getData接收该数据，并传入then的回调函数
```

### 关键注意点

- JSONP只支持GET请求，不支持POST请求（因为script标签的src只能发起GET请求）；

- 回调函数名必须唯一，避免多个JSONP请求冲突（代码中用时间戳保证唯一性）；

- 若出现 `link hit security strategy` 报错，不是前端代码问题，而是后端接口的安全策略限制了该JSONP请求，需联系后端调整（比如添加前端域名到白名单、允许JSONP请求）；

- 请求完成后必须清理script标签和全局函数，避免内存泄漏。

## 总结

以上7个JS功能，覆盖了前端开发中URL处理、事件绑定、性能优化、跨域请求等高频场景，代码均经过优化，可直接复制到项目中使用。

重点提醒：使用JSONP时若遇到 `link hit security strategy` 报错，需排查后端安全策略，而非前端代码；另外，事件绑定和滚动相关功能，建议结合节流函数优化性能，避免频繁触发函数导致页面卡顿。