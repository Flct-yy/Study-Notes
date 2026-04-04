# JavaScript ES6

## JavaScript用处及特点

- Web:网站的开发一直都是使用JavaScript;
- 移动端:借助ReactNative、Weex、Uniapp等框架实现跨平台开发;
- 小程序:离不开JavaScript;
- 桌面端:借助Electron来开发;
- 服务器端:借助Node环境使用JavaScript来开发。
- 机器学习:TensorFlow.js、ONNX.js

JS是弱类型语言，大量隐式类型转换，类型检查全部推迟到运行时变量、属性、函数都没有静态签名束缚。

## JS 标识符
由英文字母大小写，数字，下划线组成，不能以数字开头，严格分大小写，不可以使用关键字和保留字。
## 变量

- 声明变量：var、let、const
- 变量类型：基本类型、引用类型
-   基本类型：string、number、boolean、null、undefined
-   引用类型：Object、Array、Function、Date、RegExp、Map、Set、WeakMap、WeakSet

var 变量声明提升：变量声明会被提升到代码的开头，但是不会被初始化，需要手动初始化。

### undefined 和 null
underfined 和 null 区别
  undefined:未定义类型
    变量已声明但未赋值;
    对象属性/数组元素不存在;
    函数没return值
  null:空类型
    代码主动“清空"对象引用或返回值
    本身是一个对象，但是没有任何值。

### Boolean
非空字符串，非0数字，非空对象，条件语句本身满足时为true，其余均为false。

### Number
JS中只有number数字类型(64位双精度浮点数),不论是整数还是小数都属于数值类型，没有int,float, long
short等整数类型。
NaN：Not a Number，表示非数值，NaN与任何值都不相等，包括它自己,属于Number类型。

### String
JavaScript的string是一种原始值类型，由16位无符号整数(UTF-16码元)组成的不可变有序序列，用于表示文本数据。
值本身不能改:创建后任何“修改”操作(拼接、大小写转换、切片、替换等)都会返回一个全新的字符串，原字符串在内存中保持不变。
属性/方法:
length:获取字符串长度
indexOf(:返回字符串中指定文本首次出现的索引，找不到返回-1search():搜索特定值的字符串，并返回匹配的位置
slice():根据start和end截取字符串
substring(): 类似于 slice()
replace():替换字符串中指定的值
split():分割字符串转换为数组

拼接字符串
  字符串相加:会自动转换为字符串
  字符串模板:使用`${}`来表示变量，`${}`会被替换为变量的值

### string类型转换
  xx.toString():将xx转换为字符串
  String(xx):将xx转换为字符串
  `${xx}`:将xx转换为字符串
  xx + '':将xx转换为字符串

### Number类型转换
  Number(xx):将xx转换为数字
  parseInt(xx):将xx转换为整数
  parseFloat(xx):将xx转换为浮点数
  
  空字符串/false/null->0
  true->1
  不仅仅是数字的字符串，undefined->NaN

### 算数运算符
  + - * / % **
  数字+字符串，先转成字符串再运算，可用作字符串转换和字符串拼接
  字符串NaN+任何数字=NaN
  boolean+数字，先转成数字再运算
  2 * null =0
  3 * undefined = NaN

  %：a % b = a - (b * Math.floor(a / b)) 所以被除数符号决定结果符号

### 自增/自减 赋值 关系 逻辑
自增/自减
  ++a:先自增1，再赋值给a
  a++:先赋值给a，再自增1
  --a:先自减1，再赋值给a
  a--:先赋值给a，再自减1
  操作数必须是 可以被赋值的对象
  > 123++ 会报错  字面量（如 123、'abc'、true）是只读的，无法被赋值
赋值
  = += -= *= /= %= **=
关系
  > < >= <= == === != !==
  任何数据和NaN比较，都是false
  字符串比较的是Unicode编码
  严格相等===只看“值”和"类型"，不做任何隐式类型转换
逻辑
  && || !
三元
  condition? trueValue : falseValue

### 数组
JavaScript的数组(Array)是一组按顺序排列的值，本质上属于对象类型，但具备一套专用的“类数组”语义和API。核心特点一句话:动态、稀疏、可变、0起始索引、内置方法极其丰富。

#### 属性/方法:
  length:数组长度
  methods[index]:获取数组中元素
  typeof methods //object
  Array.isArray(methods) //true
  methods instanceof Array //true
  push():添加元素
  pop():删除最后一个元素，return删除的元素
  methods.join(,):使用,将数组中的元素拼接成一个字符串
  methods.shift():删除第一个元素，返回删除的元素
  methods.concat(['Transfer']):数组合并，返回一个新的数组，原数组并未改变
  methods.sort():根据元素首字母排序，可传入排序函数(比值函数)
  forEach():遍历数组
  map():遍历返回一个新的list，比如:循环数据生成UI列表
  filter():根据规则返回满足条件的新list
  every():检查每一个元素都满足条件
  some():只要有一个满足条件就返回true
  indexOf():返回第一个符合条件的下标，找不到返回-1
  find():返回第一个符合条件的元素。

#### 数组迭代
  ```javascript
  for(let i = 0; i < arr.length; i++){
    console.log(arr[i]);
  }
  for(let item of arr){
    console.log(item);
  }
  for(let key in arr){
    console.log(key, arr[key]);
  }
  arr.forEach(function(item, index, arr){
    console.log(item, index, arr);
  });
  arr.map(function(item, index, arr){
    return item * 2;
  });
  ```
  forEach 没有返回值， 
  map返回全新数组，原数组不变。

### 对象
object对象类型，用于描述一个对象
```javascript
// 方式1:
const p = new Object();
p.name = 'Vintor';
p.sex = 'man';
// 方式2:
const p = {name: 'Vintor', sex: 'man'};
// 方式3:
const name = 'Vintor';
const sex = 'man';
const p = {namename, sex};
```

### 函数

JavaScript 函数是一种可调用的对象(function object)，用来封装一段可复用的逻辑
拥有代码块+词法环境+this绑定+原型等
可赋值、可传参、可返回、可存数组/Map/Set
函数“既能当普通子程序、又能当类、还能捕获外部变量”
因此支撑起JS的函数式、事件驱动、面向对象、异步并发等全部编程范式

#### 函数预解析
JS代码在逐行执行代码之前,会对代码进行加工，对变量/函数声明提升到作用域最前面，该过程成为预解析

1. 变量声明提升：变量声明会被提升到代码的开头，但是不会被初始化，需要手动初始化。
2. 函数声明提升：函数声明会被提升到代码的开头，但是不会被执行，需要手动调用。

#### 闭包

函数+函数声明时所在的词法作用域链的引用。
外层函数执行完，其活动对象(Activation Object)本该被GC，但只要里层函数还存活并持有对外部变量的引用，活动对象就被保留，形成闭包。
  + 数据隐藏/私有变量(不暴露全局，只给指定接口)
  + 函数工厂(按参数批量生成相似但独立的函数)
  + 事件处理/异步回调(循环注册事件捕获当前索引)
  + 偏函数&柯里化(提前固化部分参数，延迟执行)
  + 迭代器/生成器(维持迭代状态)
  + 模块化(ES5时代)(模拟块级作用域，导出API)
  + React Hooks(useState/useEffect 闭包保存状态与最新 props)

### 类

对已有的「构造函数+原型链」这套机制的语法糖封装。
  类就是带糖衣的构造函数
  类体就是批量往原型上挂方法
  类字段就是帮你往构造函数里插赋值语句

```js
//构造函数本体
function Point(x, y){
  this.x=x;
  this.y = y;
}
// 原型方法
Point.prototype.show =function (){
  console.log(this.x, this.y);
}
new Point(0,0)
// 实例化对象 建空对象——链原型——构造体里this指向它——隐式返回该对象。
```

ES6新增了class关键字，可以定义类，类声明提升，类实例化，类继承，类方法重写等。

### this
  是在函数执行时，动态引用当前函数的「执行上下文(运行环境)」中的对象
  本质上是一个指向当前执行环境所属对象的指针(引用)
  让函数或对象能够灵活访问和操作其所属环境的对象属性/方法，实现代码复用与上下文关联。

  call/apply/bind:改变函数执行时的this指向，改变函数执行时的作用域链。
  call 逐个参数传值，apply 传数组参数，bind 返回一个新的函数，但不会立即执行。

## if else

if else语句是条件语句，根据条件判断执行不同的代码块。

```javascript
if(condition){
  //执行的代码块
}else if(condition2){
  //执行的代码块
}else{
  //其他执行的代码块
}
```

## switch

switch是一种多路分支语句:把一个表达式的值与若干个case标签的全等(===)结果逐个比对，命中就执行对应代码块，并一路向下“穿透"直到遇到break或语句结束;若都没命中可走可选的default。

```javascript
switch(expression){
  case value1:
    //执行的代码块
    break;
  case value2:
    //执行的代码块
    break;
  default:
    //其他执行的代码块
}
```

## 循环

JS中有三种循环结构:for、while、do-while。

## AJAX
浏览器用JavaScript异步发HTTP请求并处理响应(2005提出)
掌握请求生命周期、数据格式、跨域策略，就能让网页“无刷新"与服务器双向通信。
XHR基于回调，易陷入“地狱”;ES6后可包成Promise:
现代替代:Fetch API(ES2017)

## ES6

### 解构赋值

从数组或对象里一次性提取值，直接按对应位置/名字塞进变量，不用再写临时变量或点语法。
数组解构赋值
```js
const [namel, name2] = ['PayMe', 'AlipayHK', 'Hecto'];
const [, [m1, m2]]=[9, [8,7]]://1=9, m1=8, m2=7
```
对象解构赋值
```js
const p = {name: 'Vintor', sex: 'man'};
const {age = '18'}=p || {};
```
注:对象解构，右侧不能是undefined

### 展开运算符
1. 扩展运算符在左侧（剩余参数）
将剩余的数据打包到一个新的数组中：
```javascript
const methods = ['PayMe', 'LinePay', 'Transfer'];
const [ele, ...arr2] = methods;
```

2. 扩展运算符在右侧（展开数组）
解开数组中的数据：
```javascript
const arr1 = ['PayMe', 'LinePay'];
const arr2 = ['Transfer'];
const methods = [...arr1, arr2];  // 注: arr1不能是undefined
```

3. 作为函数形参（收集剩余参数）
将传递的数据保存在数组中：
```javascript
function sum(...values) {}
function sum(a, ...values) {}
```

4. 扩展对象（浅拷贝与合并）
```javascript
const p = {name: 'Vintor', sex: 'man'};
const info = {...p, age: 18};
```

### 箭头函数
ES6 之后可以使用箭头函数定义，并且支持直接在形参处指定默认值：
```javascript
const getInfo = (name = 'Vintor') => {
  // ES6 开始，指定默认值
};
```

核心特征
1.  **没有自己的 `this`** → 永远捕获定义时外层作用域的 `this`
2.  **没有 `arguments` 对象** → 用剩余参数 `...args` 代替
3.  **没有 `super` / `new.target`** → 不能用作构造器
4.  **不能使用 `new`** → `new Arrow()` 会直接抛出 `TypeError`
5.  **没有 `prototype` 属性** → 继承链更轻量

### ES6 Class 语法说明

- ES6 之前没有“类”这个语法，只有“构造函数 + 原型链”。
- ES6 的 `class` 及其关键字（`class`、`extends`、`super`、`static`、`constructor`、私有 `#`）只是把老机制包成语法糖。
- ES6 类 = 语法糖 + 严格模式 + 原生私有 + 简洁继承，底层依旧是 `prototype` 链。

```javascript
class Info {
  private name;
  private age;

  // 构造函数
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  getInfo() {}

  // 静态属性
  static age = 18;
  // 静态方法
  static pay() {}
}
```
#### 类的继承
ES6 前（构造函数 + 原型链）
```javascript
function Person(name, sex) {
  this.name = name;
  this.sex = sex;
  Person.prototype.eat = function() {}
}

function Worker(name, sex, time) {
  Person.call(this, name, sex);
  this.time = time;
  this.work = function() {}
}
// 子类原型设置为父类实例
Worker.prototype = new Person();
Worker.prototype.constructor = Worker;

var worker = new Worker('Vintor', 'man', 18);
worker.eat();
```

ES6 后（class 语法糖）
```javascript
class Person {
  constructor(name, sex) {
    this.name = name;
    this.sex = sex;
  }
  run() {}
}

class Worker extends Person {
  constructor(name, sex, time) {
    super(name, sex);
    this.time = time;
  }
  work() {}
}
```

#### 类的修饰符
- `public`: 共有属性/方法（默认）
- `private`: 当前类中可见，私有属性/方法
- `protected`: 当前类和子类中可见
- `readonly`: 只读属性，不能给其赋值
- `get & set`: “访问器属性”（accessor property）的语法糖，用来把“读取/赋值”动作伪装成普通字段的样子，实则暗中执行函数。

```javascript
class Circle {
  constructor(r) {
    this.r = r;      // 真实字段
  }

  get area() {       // 读取 c.area 时自动执行
    return Math.PI * this.r ** 2;
  }

  set area(value) {  // 赋值 c.area = x 时自动执行
    this.r = Math.sqrt(value / Math.PI);
  }
}

const c = new Circle(2);
console.log(c.area);  // 12.566… 读取时触发 get
c.area = 50;          // 赋值时触发 set
console.log(c.r);     // 3.989… 半径被改写
```

# TypeScript

## 为什么要用TypeScript
开发共识：**错误出现的越早越好**

写代码时发现错误 > 代码编译时发现（IDE的优势所在）> 代码运行期间发现（类型检测）

任何新技术的出现都是为了解决原有技术的某个痛点，虽然JS本身很强大，且ES6、7等的推出，使JS**更加现代，更加安全，更加方便**。

但由于JS的历史原因，存在的痛点：
- ES5及之前`var`的作用域问题；
- 最初JS设计的数组类型内存空间不连续；
- 没有**类型检测**机制。

TypeScript = JavaScript + 类型系统 + 最新 ECMAScript 特性

## 核心特性
- 类型系统（静态类型检查）
- 编译时类型检查，运行时还是纯 JavaScript
- 完全兼容 JavaScript，所有 `.js` 文件都是合法的 `.ts` 文件
- 开源，由微软开发维护

### 工作流程
```
TypeScript代码 (.ts) → TypeScript编译器 → JavaScript代码 (.js) → 浏览器/Node.js
     ↑                                 ↓
  类型检查                       没有类型信息
```

### TS 环境搭建与运行
- TS 最终被编译成 JS 在浏览器/Node 环境中运行：
```bash
# install
npm install typescript -g

# version
tsc --v
```

- TS 运行方式：
  - webpack
  - ts-node
    ```bash
    npm install ts-node -g
    npm install tslib @types/node -g
    ts-node xxx
    ```

## any
某些情况下，变量的类型无法确定，可以使用 `any`，和 OC 中的 `id`、Dart 中的 `dynamic` 类似。
**实际开发中建议少用或不用**。

## enum
将一组可能出现的值列举在一个类型中，枚举的值可以是 string 和 number。

## interface/type
使用类型定义接口
```typescript
type Info = {
  name: string;
  sex?: string;  // 可选
  readonly age: number;  // 只读
}
```

使用 interface 定义接口
```typescript
interface Info {
  name: string;
  sex?: string;  // 可选
  readonly age: number;  // 只读，初始化之后不可被修改
}
```

使用 Type 定义函数类型
```typescript
type CalFunc = (num1: number, num2: number) => number;
```

### type 和 interface 的核心区别对比
| 特性                   | `type` (类型别名)                             | `interface` (接口)                           |
| ---------------------- | --------------------------------------------- | -------------------------------------------- |
| **定义方式**           | 用 `type 名称 = 类型`，支持任意类型           | 用 `interface 名称 { ... }`，仅支持对象/函数 |
| **扩展/继承**          | 用 `&`（交叉类型），不可重复定义              | 用 `extends`，支持重复定义（自动合并）       |
| **支持的类型**         | 基础类型、联合类型、元组、对象、函数等        | 仅对象类型、函数类型                         |
| **实现（implements）** | 类可以 `implements` 类型别名（仅限对象/函数） | 类可以 `implements` 接口                     |
| **声明合并**           | 不支持（重复定义会报错）                      | 支持（同名接口自动合并）                     |

### 使用场景建议
1. **优先用 interface**：
   - 定义对象结构（如 API 响应、组件 props）；
   - 需要声明合并（如扩展第三方库的类型）；
   - 希望代码更符合面向对象风格（类的接口约束）。

2. **优先用 type**：
   - 定义基础类型、联合类型、元组；
   - 需要交叉类型合并多个类型；
   - 定义函数类型（更简洁）。
 
## 接口继承/多继承
使用extends实现继承，多继承中间使用逗号隔开

## 交叉类型
使用 `&` 实现交叉类型，多个类型合并为一个类型

## unknow
`unknown` 类型表示未知类型，可以赋值给任意类型，但不能调用其属性/方法。

### unknown 与 any 的区别
| 特性         | `unknown`（安全的任意类型）                      | `any`（无约束的任意类型）                  |
| ------------ | ------------------------------------------------ | ------------------------------------------ |
| **类型检查** | 强制类型检查，不能直接使用（必须先缩小类型）     | 跳过所有类型检查，可直接调用任意属性/方法  |
| **类型安全** | 高（TS 会保护你不犯错）                          | 低（和写原生 JS 一样，容易出运行时错误）   |
| **使用场景** | 接收未知类型的值（如 API 响应、用户输入）        | 临时兼容 JS 代码、快速迭代（不推荐长期用） |
| **赋值规则** | 可赋值给 `unknown`/`any`，不可直接赋值给其他类型 | 可赋值给任意类型（会污染其他变量的类型）   |

## void
一般用于指定函数没有返回值，函数中不指定返回值类型默认就是void

## 可选类型
和 Swift 类似，当数据的类型可能为空时，可以使用 `?` 和类型组合定义成可选类型。

可选类型定义，可以看成 `类型 | undefined` 的联合类型：
```typescript
interface Info {
  name: string;
  age?: number;
}
```

可选类型取值，使用 `?` 可以防止取值时前面数据（`info`）为 `undefined` 导致 JS Error，调用的时候可以不传：
```typescript
function getAge(info?: Info): number {
  return info?.age || 0;
}
```

方法中参数是可选类型，调用时可不传，但要注意：可选类型作为函数参数类型时，需要放在固定类型后面。
```typescript
getAge(); // 调用时可不传参数
```

## 泛型

```typescript
function getType<T, E>(arg1: T, arg2: E) {

}
```

一般情况下，泛型指定时有以下不成文的规范供参考：
- `T`：Type 的缩写，代表类型
- `K`、`V`：key 和 value 的缩写，代表键值对
- `E`：Element 的缩写，代表元素
- `O`：Object 的缩写，代表对象

注：泛型也可以用在 `interface` 中。

## tuple
元组是一组数据的组合，在 Swift/Python 中也有该类型，**通常用作函数返回值**。

```typescript
const info: [string, string, number] = ['Vintor', 'man', 18];
const name = info[0];
```

元组和数组的区别
- 元组：一般用于存储元素类型不同的数据，每一个元素都有自己的类型。
- 数组：一般用于存储同一类型的数据。

## Union 联合类型
联合类型是由两个或多个不同类型组合而成的类型，其值可以是联合成员中的任一类型。

```typescript
// 定义，CalType 成为类型别名
type CalType = number | string;

function caculate(num1: CalType, num2: CalType) {}

caculate(1, 3);
caculate('1', '3');
```

## 类型断言

类型转换
```typescript
const name: any = 'Vintor';
const str = name as string;
```

强制解包
当确定可选类型的变量一定有值时，可以使用 `!` 强制解包（类似于 Swift），**实际项目开发中不建议使用**：
```typescript
const ret = name!.toUpperCase();
```

## ?? !!
- `!!`：将数据转换成 boolean 类型
- `??`：空值合并，当左边的值为 `null` / `undefined` 时，使用 `??` 后面的值

## 函数重载示例（TypeScript）

当金额计算方法使用联合类型比较麻烦时，可以使用**函数重载**来解决。

1. 重载声明（定义调用签名）
```typescript
// 声明1：接收两个 number，返回 number
function caculate(num1: number, num2: number): number;
// 声明2：接收两个 string，返回 string
function caculate(num1: string, num2: string): string;
```
1. 函数实现（兼容所有声明）
```typescript
// 实现：用 any 兼容所有参数类型
function caculate(num1: any, num2: any): any {
  if (typeof num1 === 'number' && typeof num2 === 'number') {
    return num1 + num2;
  } else if (typeof num1 === 'string' && typeof num2 === 'string') {
    return parseFloat(num1) + parseFloat(num2);
  }
  return 0;
}
```

## 常用工具类型

基础接口定义
```typescript
interface User {
  id: number;
  name: string;
  email: string;
  age?: number; // 可选属性
}
```

| 工具类型       | 作用                                                | 示例代码                                 |
| -------------- | --------------------------------------------------- | ---------------------------------------- |
| `Partial<T>`   | 将类型 `T` 的所有属性变为**可选**                   | `type PartialUser = Partial<User>;`      |
| `Readonly<T>`  | 将类型 `T` 的所有属性变为**只读**                   | `type ReadonlyUser = Readonly<User>;`    |
| `Pick<T, K>`   | 从类型 `T` 中**挑选**指定的属性 `K`                 | `type UserBasicInfo = Pick<User, "id"    \| "name">;` |
| `Omit<T, K>`   | 从类型 `T` 中**排除**指定的属性 `K`                 | `type UserWithoutId = Omit<User, "id">;` |
| `Record<K, T>` | 构建一个键类型为 `K`、值类型为 `T` 的**键值对类型** | `type PageInfo = Record<"home"           \| "about"   \| "contact", { title: string }>;` |

## ts配置
```json
// tsconfig.json 核心配置
{
  "compilerOptions": {
    "target": "ES2020",       // 编译目标版本
    "module": "ESNext",       // 模块系统
    "strict": true,           // 启用严格模式
    "esModuleInterop": true,  // 兼容CommonJS
    "outDir": "./dist"        // 输出目录
  },
  "include": ["src/**/*"],    // 包含的文件
  "exclude": ["node_modules"] // 排除的文件
}
```

## 学习路线与资源

### 学习路径建议
- **基础阶段**：类型系统、接口、类（1-2周）
- **进阶阶段**：泛型、高级类型、装饰器（2-3周）
- **实战阶段**：React/Vue + TS项目、Node.js + TS（1-2个月）

### 推荐资源
- 官方文档：[www.typescriptlang.org](https://www.typescriptlang.org/)
- 练习平台：TypeScript Playground
- 开源项目：Vue 3源码、Ant Design源码
- 书籍：《Effective TypeScript》

### 社区支持
- Stack Overflow的TypeScript标签
- GitHub上的DefinitelyTyped项目
- 中文社区：TypeScript中文网