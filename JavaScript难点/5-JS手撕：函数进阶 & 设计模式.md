# JS手撕：函数进阶 & 设计模式解析

在 JavaScript 开发中，无论是日常业务开发还是面试考察，有一批高频代码片段始终贯穿其中——它们涵盖函数封装、设计模式、异步处理等核心场景，既能提升开发效率，也是理解 JS 底层逻辑的关键。本文将以「通俗解读+专业拆解」的方式，逐一看懂这些实用代码，帮你吃透背后的原理，做到会用也会讲。

## 一、函数柯里化（Currying）

### 通俗理解

柯里化就像「分步点餐」：比如点一杯奶茶，不用一次性说清“中杯、少糖、常温”，可以先选“中杯”，再选“少糖”，最后选“常温”，每一步都记录你的选择，等所有选项凑齐，再最终下单（执行函数）。核心是“把多参数函数拆成单参数（或部分参数）的嵌套函数，逐步收集参数，最终执行”。

### 专业拆解（附代码解析）

柯里化的核心价值是**参数复用、延迟执行**，下面这段工具函数是面试中最常考的实现方式，逐行拆解其逻辑：

```js
// 定义柯里化工具函数，接收原函数 fn + 初始参数
function curry(fn) {
  // 1. 校验入参：必须是函数，否则抛出类型错误（健壮性处理）
  if (typeof fn !== "function") throw new TypeError("Expected a function");
  
  // 2. 获取原函数【需要的必填参数个数】（函数的 length 属性 = 形参数量）
  // 比如 fn(a,b,c)，fn.length 就是 3，代表需要3个参数才能执行
  const requiredArgsLength = fn.length;
  
  // 3. 截取除了第一个参数（fn）之外的所有【初始参数】
  // arguments 是类数组（不能直接用数组方法），用 slice 转成真正的数组
  const initialArgs = [].slice.call(arguments, 1);

  // 4. 内部柯里化核心函数：接收新传入的参数
  function _curry(...newArgs) {
    // 合并：初始参数 + 本次传入的新参数（收集所有已传入的参数）
    const allArgs = [...initialArgs, ...newArgs];
    
    // 5. 判断：参数是否凑够了原函数需要的数量
    if (allArgs.length >= requiredArgsLength) {
      // ✅ 凑够了：执行原函数，传入所有参数（用 apply 绑定 this，保证上下文正确）
      return fn.apply(this, allArgs);
    } else {
      // ❌ 没凑够：递归调用 curry，继续收集参数（把已收集的 allArgs 作为初始参数传入）
      return curry.call(this, fn, ...allArgs);
    }
  }

  // 6. 返回内部收集参数的函数（不立即执行，延迟到参数凑够后执行）
  return _curry;
}
```

### 用法示例

```js
// 原函数：求三个数的和（需要3个参数）
function add(a, b, c) {
  return a + b + c;
}

// 柯里化处理
const curryAdd = curry(add);

// 分步传参（延迟执行）
curryAdd(1)(2)(3); // 6（分步传参，凑够3个执行）
curryAdd(1,2)(3); // 6（部分传参，再补全）
curryAdd(1)(2,3); // 6（任意分步组合）
```

### 关键注意点

- 函数的 length 属性：仅统计“未指定默认值的形参”，如果形参有默认值（如 add(a=0,b)），length 会计算到第一个默认值参数为止（此时 add.length = 0）。

- 递归收集参数：每次传参不足时，都会返回一个新的 _curry 函数，继续收集参数，直到满足要求。

## 二、函数组合（Compose）

### 通俗理解

函数组合就像「流水线作业」：比如生产一瓶饮料，先“加水”，再“加糖”，最后“装瓶”，每个步骤都是一个函数，组合起来就是“加水→加糖→装瓶”的完整流程，前一个函数的输出是后一个函数的输入。核心是“将多个单参数函数组合成一个函数，从右往左依次执行”。

### 专业拆解（附代码解析）

函数组合是函数式编程的核心技巧，常用于简化多步骤逻辑（如数据处理、中间件），下面是最简洁的实现方式：

```js
function compose(...funcs) {
  // 没有传入函数，直接返回参数本身（边界处理：传入空函数时，不改变输入）
  if (funcs.length === 0) {
    return arg => arg;
  }

  // 只有一个函数，直接返回该函数（边界处理：无需组合，直接执行）
  if (funcs.length === 1) {
    return funcs[0];
  }

  // ✅ 核心：用 reduce 实现函数组合，从右往左执行
  // reduce 遍历 funcs，将前一个函数 a 和当前函数 b 组合成 (args) => a(b(...args))
  // 比如 compose(f1,f2,f3) 最终变成 (args) => f1(f2(f3(...args)))
  return funcs.reduce((a, b) => (...args) => a(b(...args)));
}
```

### 用法示例

```js
// 步骤1：将数字转为字符串
const toString = num => num + "";
// 步骤2：给字符串加前缀
const addPrefix = str => "num_" + str;
// 步骤3：将字符串转为大写
const toUpperCase = str => str.toUpperCase();

// 组合函数：从右往左执行 → toString → addPrefix → toUpperCase
const transform = compose(toUpperCase, addPrefix, toString);

// 执行：123 → "123" → "num_123" → "NUM_123"
transform(123); // "NUM_123"
```

### 关键注意点

- 执行顺序：**从右往左**，这是 compose 的默认规则（与 pipe 相反，pipe 是从左往右）。

- 参数传递：组合后的函数接收的参数，会全部传给最右边的函数，后续函数仅接收前一个函数的返回值，因此建议每个组合的函数都是“单输入、单输出”。

## 三、模拟 call 方法

### 通俗理解

call 方法的作用是「给函数换个“主人”」：比如小明有一个“吃饭”函数，小红想借用这个函数（让函数里的 this 指向小红），就可以用 call 实现。核心是“改变函数内部的 this 指向，并立即执行函数”。

### 专业拆解（附代码解析）

call 是 Function.prototype 上的方法，所有函数都能调用。其底层逻辑是“将函数挂载到目标对象上，作为对象的方法调用（此时 this 指向该对象），执行后删除临时方法，避免污染原对象”，具体实现如下：

```js
Function.prototype.mycall2 = function (thisArg, ...args) {
  // 1. 校验：调用 mycall2 的必须是函数，否则报错（健壮性处理）
  if (typeof this !== "function") {
    throw new TypeError(this + " is not a function");
  }

  // 2. 确定 this 指向：传入 null/undefined 时，this 指向全局对象（浏览器是 window，Node 是 global）
  let context = thisArg == null ? globalThis : Object(thisArg);

  // 3. 创建唯一 Symbol 属性，防止覆盖对象原有属性（比如对象本身就有 fn 方法，避免冲突）
  const fn = Symbol("fn");

  // 4. 把当前函数（this 指向的就是调用 mycall2 的函数）挂载到 context 上
  context[fn] = this; 

  // 5. 执行函数，传入参数，接收执行结果（作为对象方法调用，this 自然指向 context）
  const result = context[fn](...args);

  // 6. 删掉临时挂载的属性，不污染原对象（核心：用完即删，保持对象纯净）
  delete context[fn];

  // 7. 返回函数执行结果（与原生 call 行为一致，返回函数执行后的结果）
  return result
};
```

### 用法示例

```js
function sayHi() {
  console.log(`Hi, 我是 ${this.name}，年龄 ${this.age}`);
}

const person1 = { name: "张三", age: 20 };
const person2 = { name: "李四", age: 22 };

// 用自定义的 mycall2 改变 this 指向
sayHi.mycall2(person1); // Hi, 我是 张三，年龄 20
sayHi.mycall2(person2, 123); // Hi, 我是 李四，年龄 22（多余参数不影响，函数不接收即可）
```

### 关键注意点

- thisArg 处理：如果传入 null/undefined，this 指向 globalThis（全局对象）；如果传入基本类型（如 123、"abc"），会被 Object() 转成对应包装对象（如 Number、String）。

- Symbol 作用：确保临时属性唯一，避免覆盖目标对象已有的属性，是实现的关键细节。

## 四、模拟 apply 方法

### 通俗理解

apply 和 call 几乎一样，都是“改变函数 this 指向并立即执行”，唯一区别是「传参方式」：call 是“逐个传参”（比如 call(obj, 1, 2, 3)），apply 是“数组传参”（比如 apply(obj, [1,2,3])），相当于“批量传参”。

### 专业拆解（附代码解析）

apply 的实现逻辑和 call 高度一致，核心差异在于“处理参数的方式”，具体实现如下：

```js
Function.prototype.myapply2 = function (thisArg, argsArray) {
  // 1. 必须是函数才能调用（和 call 一致的健壮性校验）
  if (typeof this !== "function") {
    throw new TypeError(this + " is not a function");
  }

  // 2. 处理 this 指向（和 call 完全一致）
  let context = thisArg == null ? globalThis : Object(thisArg);

  // 3. 处理参数：不传 argsArray / 传 null → 默认为空数组（避免解构报错）
  // ?? 是空值合并运算符，只有当 argsArray 为 null/undefined 时，才返回 []
  const args = argsArray ?? [];

  // 4. 唯一 Symbol 防止属性冲突（和 call 一致）
  const fn = Symbol("fn");
  context[fn] = this;

  // 5. 执行函数：用扩展运算符 ... 将数组参数拆成逐个参数，和 call 逻辑一致
  const result = context[fn](...args);

  // 6. 清理临时属性，不污染原对象（和 call 一致）
  delete context[fn];

  return result;
};
```

### 用法示例

```js
function sum(a, b, c) {
  return a + b + c;
}

const obj = { name: "测试" };

// 用 myapply2 传参（数组形式）
sum.myapply2(obj, [1, 2, 3]); // 6
sum.myapply2(obj); // 0（args 为空数组，a、b、c 都是 undefined，相加为 0）
```

### 关键注意点

- 参数处理：argsArray 必须是数组（或类数组），如果传入非数组，会报错（原生 apply 也是如此）；如果不传，默认按空数组处理。

- 与 call 的区别：仅传参方式不同，底层执行逻辑完全一致，二者可相互替代（call 能做的，apply 也能做，只是传参麻烦一点）。

## 五、模拟 bind 方法

### 通俗理解

bind 和 call、apply 的区别是「不立即执行」：call/apply 是“改变 this 并马上执行”，bind 是“改变 this 并返回一个新函数，后续需要手动调用这个新函数才会执行”，相当于“提前绑定好 this，后续随时可用”。

### 专业拆解（附代码解析）

bind 的实现比 call/apply 复杂，核心要处理两个点：「参数柯里化」和「new 调用时的 this 指向」，具体实现如下：

```js
Function.prototype.myBind = function(context, ...args) {
  // 1. 调用者必须是函数（健壮性校验）
  if (typeof this !== 'function') {
    throw new TypeError('The bound object must be a function');
  }

  // 2. 保存原函数（关键！因为后续返回的新函数需要执行原函数，this 会被改变，所以提前保存）
  const self = this; 

  // 3. 返回一个新的绑定函数（不立即执行，等待后续调用）
  function boundFunction(...newArgs) {
    // 4. 合并参数（柯里化：bind 时传入的 args + 后续调用新函数时传入的 newArgs）
    const allArgs = args.concat(newArgs);

    // 5. 执行原函数，判断是普通调用还是 new 调用
    // 用 new 调用 boundFunction 时，this 指向 new 出来的实例，此时要忽略之前绑定的 context
    // 否则，this 指向绑定的 context
    return self.apply(
      this instanceof boundFunction ? this : context,
      allArgs
    );
  }

  // 6. 继承原函数的原型，让 new 能正常工作（关键细节）
  // 比如用 new 调用绑定后的函数，实例能访问原函数原型上的属性/方法
  if (this.prototype) {
    function Empty() {} // 空函数作为中间层，避免原型链污染
    Empty.prototype = this.prototype;
    boundFunction.prototype = new Empty();
  }

  return boundFunction;
};
```

### 用法示例

```js
function Person(name, age) {
  this.name = name;
  this.age = age;
  console.log(`我是 ${this.name}，年龄 ${this.age}`);
}

const obj = { name: "默认名称" };

// 1. 普通绑定：提前绑定 this 和部分参数
const boundPerson = Person.myBind(obj, "张三");
boundPerson(20); // 我是 张三，年龄 20（this 指向 obj，合并参数 ["张三", 20]）

// 2. new 调用：忽略绑定的 context，this 指向新实例
const instance = new boundPerson(22); // 我是 undefined，年龄 22（this 指向 instance，name 未赋值）
console.log(instance.age); // 22（实例能访问 age 属性，原型继承生效）
```

### 关键注意点

- new 调用处理：这是 bind 和 call/apply 最大的区别之一，用 new 调用绑定后的函数时，this 会指向新实例，而非绑定的 context。

- 原型继承：通过空函数中间层继承原函数原型，避免直接赋值原型导致的污染（如果直接 boundFunction.prototype = this.prototype，修改 boundFunction 原型会影响原函数原型）。

## 六、实现链式调用

### 通俗理解

链式调用就像「连环操作」：比如买奶茶时，“点单→加珍珠→加冰→付款”，每一步操作完成后，都能继续下一步，不用重复写对象名。核心是“每个方法执行后，返回当前对象（this），让后续方法能继续调用”。

### 专业拆解（附代码解析）

链式调用在 JS 中非常常见（如 jQuery、Promise），实现逻辑极其简单，核心就是「return this」，具体实现如下：

```js
// 定义一个类（也可以是构造函数）
class class1 {
  constructor() {
    // 可选：初始化一些属性
    this.data = [];
  }
}

// 给类的原型添加方法，每个方法执行后 return this
class1.prototype.method = function (param) {
  console.log("执行方法，参数：", param);
  this.data.push(param); // 可以做一些业务逻辑
  return this; // 必须 return this，才能实现链式调用
};

// 扩展更多方法，同样 return this
class1.prototype.anotherMethod = function (param) {
  console.log("执行另一个方法，参数：", param);
  this.data.push(param);
  return this;
};

// 使用：创建实例后，链式调用方法
const ins = new class1();
ins.method('a').anotherMethod('b').method('c'); 
// 输出：执行方法，参数：a → 执行另一个方法，参数：b → 执行方法，参数：c
console.log(ins.data); // ['a', 'b', 'c']（业务逻辑生效）
```

### 关键注意点

- 核心要求：每个需要链式调用的方法，必须返回 this（当前实例），如果返回其他值，后续链式调用会报错（因为其他值可能没有对应的方法）。

- 适用场景：常用于封装工具类、组件方法（如表单验证、DOM 操作），简化代码写法。

## 七、发布订阅模式（EventEmitter）

### 通俗理解

发布订阅模式就像「公众号订阅」：你（订阅者）关注了一个公众号（发布者），当公众号发布新文章（发布事件）时，所有关注的人都会收到通知（执行订阅的回调）。核心是“解耦发布者和订阅者，二者互不依赖，通过事件仓库传递消息”。

### 专业拆解（附代码解析）

发布订阅模式是前端常用的设计模式，常用于组件通信、事件监听（如 Vue 的事件总线），下面是完整的 EventEmitter 实现，包含订阅、取消订阅、发布、一次性订阅四个核心方法：

```js
class EventEmitter {
  // 1. 构造函数：初始化事件仓库（存储事件名和对应的回调函数数组）
  constructor() {
    // 用 Map 存储：key=事件名（字符串），value=回调函数数组（一个事件可以有多个订阅者）
    this.events = new Map();
  }

  // 2. 订阅事件：监听一个事件，添加回调函数
  on(eventName, listener) {
    // 如果事件不存在，先创建一个空数组（避免后续 push 报错）
    if (!this.events.has(eventName)) {
      this.events.set(eventName, []);
    }
    // 把回调函数 push 进数组（一个事件可以订阅多个回调）
    this.events.get(eventName).push(listener);
  }

  // 3. 取消订阅：移除指定事件的指定回调函数
  off(eventName, listener) {
    // 事件不存在，直接返回（无需处理）
    if (!this.events.has(eventName)) return;

    const listeners = this.events.get(eventName);
    // 找到回调函数在数组中的索引
    const index = listeners.indexOf(listener);
    // 找到并删除对应的函数（splice 会修改原数组）
    if (index !== -1) {
      listeners.splice(index, 1);
    }
  }

  // 4. 发布事件：触发指定事件，执行所有订阅的回调函数，并传递参数
  emit(eventName, ...args) {
    // 事件不存在，直接返回（没有订阅者，无需执行）
    if (!this.events.has(eventName)) return;

    const listeners = this.events.get(eventName);
    // 遍历执行所有回调函数，并传入发布时的参数
    listeners.forEach(listener => listener(...args));
  }

  // 5. 只监听一次：订阅事件后，执行一次回调就自动取消订阅
  once(eventName, listener) {
    // 包装一层函数，执行原回调后，立即取消订阅
    const wrappedListener = (...args) => {
      // 先执行原回调函数
      listener(...args);
      // 执行完立刻删除当前包装函数（取消订阅）
      this.off(eventName, wrappedListener);
    };
    // 订阅包装后的函数（而非原函数，确保执行一次后取消）
    this.on(eventName, wrappedListener);
  }
}
```

### 用法示例

```js
// 创建 EventEmitter 实例（发布者）
const emitter = new EventEmitter();

// 1. 订阅事件（订阅者1）
function callback1(data) {
  console.log("订阅者1收到消息：", data);
}
emitter.on("message", callback1);

// 2. 订阅事件（订阅者2，只监听一次）
emitter.once("message", (data) => {
  console.log("订阅者2收到消息（只一次）：", data);
});

// 3. 发布事件（触发所有订阅者）
emitter.emit("message", "Hello World"); 
// 输出：订阅者1收到消息：Hello World → 订阅者2收到消息（只一次）：Hello World

// 4. 再次发布事件（订阅者2已取消订阅，不再执行）
emitter.emit("message", "再次发送消息");
// 输出：订阅者1收到消息：再次发送消息

// 5. 取消订阅者1的订阅
emitter.off("message", callback1);

// 6. 第三次发布事件（没有订阅者，无输出）
emitter.emit("message", "第三次发送消息");
```

### 关键注意点

- 事件仓库：用 Map 存储比对象更灵活，能避免对象属性名的冲突，且能更方便地获取、删除事件。

- once 实现：核心是“包装回调函数”，执行原回调后立即取消订阅，注意不能直接订阅原函数（否则无法取消）。

- 取消订阅：必须传入订阅时的同一个回调函数（不能是匿名函数），否则无法找到并删除。

## 八、单例模式

### 通俗理解

单例模式就像「公司的 CEO」：整个公司只有一个 CEO，无论你什么时候、在哪里找，找到的都是同一个人。核心是“一个类只能创建一个实例，后续所有创建实例的操作，都返回同一个已存在的实例”。

### 专业拆解（附代码解析）

单例模式常用于封装全局工具类、数据库连接、全局状态管理等场景，避免重复创建实例造成资源浪费，下面是最简洁的 ES6 实现方式：

```js
class Singleton {
  // 静态属性：存储唯一实例（静态属性属于类，不属于实例，全局唯一）
  static instance = null;

  constructor() {
    // 关键逻辑：如果已经有实例，直接返回旧实例（阻止创建新实例）
    if (Singleton.instance) {
      return Singleton.instance;
    }
    // 没有实例，创建并保存到静态属性中
    Singleton.instance = this;
    // 初始化实例属性（根据业务需求添加）
    this.data = [];
  }

  // 实例方法（业务逻辑）：添加数据
  addData(item) {
    this.data.push(item);
  }

  // 实例方法（业务逻辑）：获取数据
  getData() {
    return this.data;
  }
}
```

### 用法示例

```js
// 多次创建实例
const instance1 = new Singleton();
const instance2 = new Singleton();
const instance3 = new Singleton();

// 验证：所有实例都是同一个
console.log(instance1 === instance2); // true
console.log(instance1 === instance3); // true

// 操作实例1，instance2、instance3 也会受到影响（因为是同一个实例）
instance1.addData("测试数据");
console.log(instance2.getData()); // ["测试数据"]
console.log(instance3.getData()); // ["测试数据"]
```

### 关键注意点

- 静态属性 instance：必须用 static 修饰，确保属于类本身，而非实例，这样才能全局唯一。

- 构造函数拦截：在 constructor 中判断 instance 是否存在，存在则返回旧实例，阻止新实例创建，这是单例的核心。

- 适用场景：全局工具类（如日期工具、请求工具）、全局状态管理，避免重复创建实例造成资源浪费。

## 九、私有变量的实现（闭包+Symbol）

### 通俗理解

私有变量就像「个人的隐私」：只能自己访问和修改，别人无法直接获取或修改。在 JS 中，没有原生的 private 关键字（ES6 有，但兼容性有限），常用「闭包+Symbol」实现真正的私有变量。

### 专业拆解（附代码解析）

核心逻辑：用立即执行函数（IIFE）创建闭包，闭包内的 Symbol 变量外部无法访问；类内部用这个 Symbol 作为属性名，实现私有属性，具体实现如下：

```js
const Person = (function() {
  // 1. 闭包内的 Symbol，外部无法访问（真正的私有标识）
  // Symbol 具有唯一性，即使外部也创建同名 Symbol，也和这个不是同一个
  const _name = Symbol('name');

  // 2. 定义类，类内部可以访问闭包内的 _name
  class Person {
    constructor(name) {
      // 3. 用 Symbol 作为属性名，实现私有属性（外部无法通过 obj.name 访问）
      this[_name] = name; 
    }

    // 4. 提供公共方法，供外部间接访问私有属性（可控访问）
    getName() {
      return this[_name];
    }

    // 可选：提供公共方法，供外部间接修改私有属性（可控修改）
    setName(newName) {
      this[_name] = newName;
    }
  }

  // 5. 把类返回出去，外部可以创建实例，但无法访问闭包内的 _name
  return Person;
})();
```

### 用法示例

```js
const person = new Person("张三");

// 1. 无法直接访问私有属性（外部没有 _name Symbol，无法获取）
console.log(person.name); // undefined（没有这个公共属性）
console.log(person[_name]); // 报错（_name 是闭包内的变量，外部无法访问）

// 2. 通过公共方法访问和修改私有属性
console.log(person.getName()); // 张三
person.setName("李四");
console.log(person.getName()); // 李四
```

### 关键注意点

- 闭包的作用：隔离作用域，让 _name Symbol 只能在 IIFE 内部访问，外部无法获取，确保私有性。

- Symbol 的唯一性：即使外部创建 const _name = Symbol('name')，也和闭包内的 _name 不是同一个，无法访问私有属性。

- 可控访问：通过公共方法（getName、setName）访问和修改私有属性，可以在方法中添加校验逻辑（如判断姓名长度），更安全。

## 十、函数字符串转成函数（new Function vs eval）

### 通俗理解

有时候我们会拿到一个「函数字符串」（比如从后端接口获取，或动态拼接），需要把它转成真正的函数才能执行。JS 中有两种常用方式：new Function 和 eval，二者核心区别是「作用域安全」。

### 专业拆解（附代码解析）

两种方式的实现的逻辑不同，安全性也有差异，下面分别实现并对比：

```js
// 1. 使用 new Function（推荐：作用域独立、更安全）
function stringToFunction(funcStr) {
  try {
    // new Function 接收字符串参数，最后一个参数是函数体，前面是形参
    // 这里用 "return " + funcStr，把函数字符串转成函数表达式，执行后返回函数
    const func = new Function('return ' + funcStr)();
    return func;
  } catch (error) {
    console.error('转换失败:', error);
    return null;
  }
}

// 2. 使用 eval（不推荐：能访问当前作用域、不安全）
function stringToFunctionEval(funcStr) {
  try {
    /**
     * 给函数字符串加括号，转成函数表达式（避免被当作语句执行）
     * 比如 funcStr 是 "function add(){}"，加括号后是 "(function add(){})"，eval 执行后返回函数
     */
    const func = eval('(' + funcStr + ')');
    return func;
  } catch (error) {
    console.error('转换失败:', error);
    return null;
  }
}

// 测试示例
const funcStr = 'function add(a, b) { return a + b; }';

// 用 new Function 转换
const add1 = stringToFunction(funcStr);
console.log(add1(1, 2)); // 3（转换成功，能正常执行）

// 用 eval 转换
const add2 = stringToFunctionEval(funcStr);
console.log(add2(3, 4)); // 7（转换成功，能正常执行）
```

### 核心区别（重点）

|方式|作用域|安全性|推荐度|
|---|---|---|---|
|new Function|独立作用域，只能访问全局变量，无法访问当前局部变量|高，不会污染当前作用域，也不会执行恶意代码（相对安全）|推荐|
|eval|能访问当前作用域的所有变量（局部、全局）|低，可能执行恶意代码，也可能污染当前作用域|不推荐（除非明确知道字符串安全）|
### 关键注意点

- new Function 转换时，需要给 funcStr 加 "return "，把函数字符串转成函数表达式，否则会返回 undefined。

- eval 转换时，需要给 funcStr 加括号，避免被 JS 解析器当作语句执行（比如 function add(){} 会被当作函数声明，无法直接返回）。

- 安全性：如果函数字符串来自不可信来源（如用户输入、未知接口），无论哪种方式都有风险，需先做校验。

## 十一、模板字符串执行（with + new Function）

### 通俗理解

有时候我们会有一个「模板字符串」（比如 "${a+b}, ${b}"），需要结合一个对象（比如 {a:1, b:2}），动态替换模板中的变量并执行计算。核心是“用 with 绑定对象作用域，让模板中能直接使用对象的属性”。

### 专业拆解（附代码解析）

实现逻辑：用 new Function 创建动态函数，结合 with 语句将对象作为作用域，让模板字符串能直接访问对象属性，具体实现两种方式：

```js
// 方式1：使用 with（简洁，兼容性好）
// with 可以把一个对象当作作用域，在代码块里直接用属性名，不用写 对象.属性
const sprintf2 = (template, obj) => {
  // 1. 动态创建函数：参数是 obj，函数体是 with(obj){return `模板字符串`}
  const fn = new Function("obj", `with(obj){return \`${template}\`;}`);
  
  // 2. 执行函数，传入 obj，返回模板执行后的结果
  return fn(obj);
};

// 方式2：使用解构赋值（更安全，避免 with 的副作用）
const sprintf3 = (template, obj) => {
  // 用解构赋值，把 obj 的所有属性变成函数内的局部变量
  // 比如 obj = {a:1,b:2}，解构后变成 const {a,b} = obj;
  const fn = new Function(
    "obj",
    `const { ${Object.keys(obj).join(',')} } = obj; return \`${template}\`;`
  );
  return fn(obj);
};

// 测试示例
console.log(sprintf2("a:${a+b},b:${b}", { a: 1, b: 2 }));
// 输出：a:3,b:2（a+b 计算生效，直接使用 obj 的 a、b 属性）

console.log(sprintf3("a:${a*2},b:${b+3}", { a: 1, b: 2 }));
// 输出：a:2,b:5（解构赋值后，直接使用 a、b 变量）
```

### 核心区别

- 方式1（with）：简洁高效，但 with 会改变作用域链，可能导致变量查找变慢，且如果模板中使用了未在 obj 中定义的变量，会向上查找全局变量，有一定风险。

- 方式2（解构赋值）：更安全，模板中只能使用 obj 中的属性（未定义的变量会报错），不会向上查找全局变量，推荐使用。

### 关键注意点

- 模板字符串转义：动态创建函数时，模板字符串中的 ` 要转义成 \`，否则会被 JS 解析器当作函数体的结束。

- 属性名处理：如果 obj 的属性名包含特殊字符（如 -、空格），解构赋值会报错，需提前处理属性名。

## 十二、async 优雅处理（错误前置）

### 通俗理解

async/await 是 JS 处理异步的常用方式，但默认需要用 try/catch 捕获错误，代码会显得繁琐。错误前置的核心是“用一个包装函数，统一捕获异步错误，返回 [错误, 结果] 数组，后续直接判断错误即可，不用写 try/catch”。

### 专业拆解（附代码解析）

实现逻辑：封装一个异步包装函数，内部用 try/catch 捕获异步函数的错误，成功则返回 [null, 结果]，失败则返回 [错误, null]，简化错误处理流程：

```js
// 定义一个异步包装函数，接收一个异步函数（或返回 Promise 的函数）
async function errorCaptured(asyncFunc) {
    try {
        // 执行传入的异步函数，等待结果（asyncFunc 是异步函数，用 await 等待）
        let res = await asyncFunc()
        // 成功：返回 [没有错误（null）, 执行结果]
        return [null, res]
    } catch(e) {
        // 失败：返回 [错误信息, 没有结果（null）]
        return [e, null]
    }
}

// 模拟一个异步请求（比如接口请求）
function fetchData() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      // 模拟成功：resolve("成功数据")
      // 模拟失败：reject("网络错误")
      reject("网络错误")
    }, 500)
  })
}

// 使用：无需写 try/catch，直接判断错误
async function demo() {
  // 调用包装函数，解构出错误和结果
  const [err, data] = await errorCaptured(fetchData)

  // 错误判断：有错误则处理，无错误则使用数据
  if (err) {
    console.log("❌ 错误：", err)
    return // 有错误，终止后续逻辑
  }
  console.log("✅ 成功：", data)
}

demo(); // 输出：❌ 错误：网络错误
```

### 核心优势

- 简化代码：不用在每个 async 函数中写 try/catch，统一由包装函数捕获错误，代码更简洁。

- 错误前置：先判断错误，再处理业务逻辑，逻辑更清晰，避免错误导致后续代码报错。

- 通用性强：可用于所有异步场景（接口请求、定时器、文件读取等），只需传入异步函数即可。

### 关键注意点

- asyncFunc 要求：必须是异步函数（async 修饰）或返回 Promise 的函数，否则 await 无法等待，会直接返回同步结果。

- 返回值格式：固定返回 [err, data] 数组，err 为 null 表示成功，data 为 null 表示失败，后续使用需严格遵循这个格式。

## 十三、实现 Promise 任务调度器

### 通俗理解

Promise 任务调度器就像「餐厅排队取号」：餐厅一次只能接待2桌客人（最大并发数），后面来的客人排队，等前面的客人吃完（任务执行完），再依次接待下一桌。核心是“控制并发任务的数量，避免同时执行过多任务导致资源耗尽”。

### 专业拆解（附代码解析）

实际开发中，任务调度器常用于控制接口请求并发数（比如同时请求10个接口，控制最多2个并发），下面实现两种常用版本：通用并发调度器（面试常考）和业务实用版并发请求控制：

```js
// ====================
// 1. 通用并发调度器 Scheduler（面试标准版）
// 核心：控制最大并发数，任务排队执行，执行完一个补一个
// ====================
class Scheduler {
  constructor(maxCount = 2) {
    this.maxCount = maxCount; // 最大并发数（默认2）
    this.queue = [];         // 任务队列（存储等待执行的任务）
    this.running = 0;        // 当前运行中的任务数
  }

  // 添加任务：将任务加入队列（不立即执行）
  add(task) {
    this.queue.push(task);
  }

  // 开始执行任务：初始化启动最大并发数的任务
  start() {
    for (let i = 0; i < this.maxCount; i++) {
      this.run(); // 启动任务执行
    }
  }

  // 执行任务核心逻辑：从队列取任务，执行后补充新任务
  run() {
    // 终止条件：队列空了 或 运行中的任务数 >= 最大并发数
    if (!this.queue.length || this.running >= this.maxCount) return;

    this.running++; // 运行中的任务数+1
    const task = this.queue.shift(); // 从队列头部取出一个任务

    // 执行任务（任务是返回 Promise 的函数），执行完后更新状态
    task().finally(() => {
      this.running--; // 任务执行完，运行中的任务数-1
      this.run(); // 递归调用 run，从队列取下一个任务执行
    });
  }
}

// ====================
// 2. 并发请求控制 multiRequest（业务实用版）
// 核心：控制接口请求并发数，收集所有请求结果，最终统一返回
// ====================
function multiRequest(urls, maxNum) {
  const total = urls.length; // 总请求数
  const result = new Array(total).fill(null); // 存储所有请求结果（按顺序）
  let current = 0; // 当前要执行的请求索引
  let finished = 0; // 已完成的请求数

  // 返回 Promise，所有请求完成后 resolve 结果
  return new Promise((resolve) => {
    // 初始启动：启动最大并发数的请求（不超过总请求数）
    for (let i = 0; i < Math.min(maxNum, total); i++) {
      next();
    }

    // 执行下一个请求的逻辑
    function next() {
      if (current >= total) return; // 所有请求都已启动，终止

      const index = current++; // 记录当前请求的索引（确保结果顺序正确）
      // 执行请求（urls 中的每个元素是返回 Promise 的请求函数）
      urls[index]()
        .then((res) => {
          // 请求成功：存储成功结果
          result[index] = { success: true, data: res };
        })
        .catch((err) => {
          // 请求失败：存储失败信息
          result[index] = { success: false, error: err };
        })
        .finally(() => {
          finished++; // 已完成请求数+1
          if (finished === total) {
            resolve(result); // 所有请求完成，返回结果
          }
          next(); // 执行完一个，启动下一个请求
        });
    }
  });
}

// ====================
// 3. 使用 DEMO（可直接运行）
// ====================
// 模拟任务队列（每个任务是返回 Promise 的函数）
const tasks = [
  () => new Promise(r => setTimeout(() => { console.log("任务1"); r(); }, 1000)),
  () => new Promise(r => setTimeout(() => { console.log("任务2"); r(); }, 500)),
  () => new Promise(r => setTimeout(() => { console.log("任务3"); r(); }, 1200)),
  () => new Promise(r => setTimeout(() => { console.log("任务4"); r(); }, 800)),
];

// 测试通用调度器（最大并发数2）
const scheduler = new Scheduler(2);
tasks.forEach(task => scheduler.add(task));
scheduler.start();
// 输出顺序：任务2（500ms）→ 任务1（1000ms）→ 任务4（800ms）→ 任务3（1200ms）

// 模拟请求队列（每个请求是返回 Promise 的函数）
const urls = [
  () => new Promise(resolve => setTimeout(() => resolve("URL1"), 1000)),
  () => new Promise((_, reject) => setTimeout(() => reject("URL2"), 500)),
  () => new Promise(resolve => setTimeout(() => resolve("URL3"), 2000)),
  () => new Promise(resolve => setTimeout(() => resolve("URL4"), 800)),
];

// 测试业务版并发请求控制（最大并发数2）
multiRequest(urls, 2).then(res => {
  console.log("全部请求完成：", res);
  // 输出：[{success:true,data:"URL1"}, {success:false,error:"URL2"}, {success:true,data:"URL3"}, {success:true,data:"URL4"}]
});
```

### 关键注意点

- 通用调度器（Scheduler）：适用于所有 Promise 任务（不局限于请求），核心是“队列+递归补充任务”，控制最大并发数。

- 业务版（multiRequest）：专门用于接口请求，会按请求顺序存储结果（即使某个请求先完成，也会存在对应索引位置），最终统一返回所有结果，符合业务需求。

- 任务要求：无论是调度器还是请求控制，传入的任务/请求必须是「返回 Promise 的函数」，否则无法监听执行完成的状态。

## 总结

以上13个代码片段，覆盖了 JavaScript 中「函数封装、设计模式、异步处理、作用域控制」等核心场景，既是日常开发的高频工具，也是面试中的重点考察内容。

学习这些片段的关键，不是死记代码，而是理解背后的原理（比如闭包、this 指向、Promise 机制），这样才能灵活运用到实际业务中，甚至根据需求修改优化。建议结合示例代码亲手运行，感受每个细节的作用，加深理解。