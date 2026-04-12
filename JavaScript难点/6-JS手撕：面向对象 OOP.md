# JS 手撕：对象创建、继承全解析

在 JavaScript 中，对象是核心数据类型，几乎所有业务开发都离不开对象的创建与复用。很多新手会困惑“为什么创建对象有这么多种方式？”“哪种继承方式最靠谱？”，本文将用「通俗类比+专业拆解」的方式，把 6 种对象创建方法、7 种继承方式讲透，同时补充底层原理和实战选型建议，兼顾入门理解与面试备考。

# 一、JS 对象创建方法（6种，按常用度排序）

创建对象的核心是“封装属性和方法”，不同方式的区别在于「代码简洁度」「复用性」「性能」，我们逐个拆解，结合代码示例和场景分析，让你一看就懂。

## 1. 对象字面量 `{}`（最常用、最简单，入门首选）

这是最直观、最简洁的创建方式，直接用 `{}` 包裹键值对，相当于“随手创建一个独立对象”，不用额外定义模板，适合快速创建单个对象。

**专业说明**：对象字面量是 ES5 引入的语法，底层会隐式调用 `Object()` 构造函数，但省略了冗余代码，JS 引擎会对其进行优化，执行效率高于显式调用 `new Object()`。

```javascript
// 基础写法（键名无特殊字符，可省略引号）
const person = {
  name: "张三",
  age: 20,
  // 方法简写（ES6 语法，等价于 sayHello: function() {}）
  sayHello() {
    console.log(`你好，我是${this.name}`);
  }
};

// 特殊场景写法（键名含空格、特殊字符，需加引号）
const user = {
  "user-name": "lisi",
  "age+1": 21,
  ["say" + "Hi"]() { // 计算属性名（ES6）
    console.log("Hi~");
  }
};

// 使用方式（两种均可，推荐点语法，更简洁）
person.sayHello(); // 输出：你好，我是张三
user["user-name"]; // 输出：lisi
user.sayHi(); // 输出：Hi~

```

### 核心特点

- ✅ 最简单、代码量最少，上手无门槛，日常开发高频使用

- ✅ 适合创建「单个独立对象」（比如配置对象、单个用户信息）

- ❌ 不适合批量创建多个相似对象（比如创建10个用户，会出现大量重复代码，维护成本高）

- ✅ 支持 ES6 语法糖（方法简写、计算属性名），写法更灵活

---

## 2. 构造函数（`new 函数名()`，批量创建基础方案）

如果需要创建多个结构相似的对象（比如多个用户、多个商品），就需要一个“模板”——构造函数。构造函数本质是一个普通函数，通过 `this` 绑定属性和方法，配合 `new` 关键字生成实例，实现批量创建。

**通俗类比**：构造函数就像“工厂模具”，`new` 关键字就像“启动模具生产”，每个实例都是模具生产出的“产品”，结构一致但内容可自定义。

```javascript
// 定义构造函数（首字母大写，约定俗成，区分普通函数）
function Person(name, age) {
  // this 指向当前创建的实例（new 关键字自动绑定）
  this.name = name; // 实例属性（每个实例独有）
  this.age = age;
  // 方法直接写在构造函数内（每个实例都会单独创建一个该方法）
  this.sayHello = function () {
    console.log(`我是${this.name}，今年${this.age}岁`);
  };
}

// 创建实例（new 关键字不可省略，否则 this 指向 window）
const p1 = new Person("张三", 20);
const p2 = new Person("李四", 21);

// 使用实例
p1.sayHello(); // 输出：我是张三，今年20岁
p2.sayHello(); // 输出：我是李四，今年21岁

// 验证：两个实例的方法是不同的（内存地址不一样）
console.log(p1.sayHello === p2.sayHello); // 输出：false

```

### 核心特点

- ✅ 适合「批量创建对象」，通过参数传递，实现实例属性自定义

- ✅ 可通过 `instanceof` 判断实例类型（比如 p1 instanceof Person → true），便于类型校验

- ❌ 核心缺点：方法会在每个实例中重复创建（比如上面的 sayHello 方法，p1 和 p2 各有一个，占用额外内存），实例越多，内存浪费越严重

- ❌ 不适合复杂场景（比如方法复用、继承）

---

## 3. 原型模式（构造函数 + 原型，原生最优方案）

为了解决“构造函数方法重复创建”的问题，原型模式应运而生。核心思路：将**公共方法挂载到构造函数的原型（prototype）上**，所有实例共享同一个原型对象，因此公共方法只需要创建一次，节省内存。

**专业原理**：JS 中每个函数都有 `prototype` 属性（原型对象），每个实例都有 `__proto__` 属性（隐式原型），实例的 `__proto__` 会指向其构造函数的 `prototype`。当访问实例的方法时，JS 会先在实例自身查找，找不到就去原型对象中查找，这就是“原型链查找”。

```javascript
// 1. 定义构造函数（只放实例独有属性）
function Person(name, age) {
  this.name = name;
  this.age = age;
}

// 2. 公共方法挂载到原型上（所有实例共享）
Person.prototype.sayHello = function () {
  console.log(`我是${this.name}，今年${this.age}岁`);
};
// 原型上也可以添加公共属性
Person.prototype.gender = "男";

// 3. 创建实例
const p1 = new Person("张三", 20);
const p2 = new Person("李四", 21);

// 使用实例
p1.sayHello(); // 输出：我是张三，今年20岁
p2.sayHello(); // 输出：我是李四，今年21岁
console.log(p1.gender); // 输出：男（原型上的公共属性）

// 验证：两个实例的方法是同一个（内存地址相同）
console.log(p1.sayHello === p2.sayHello); // 输出：true

```

### 核心特点

- ✅ 公共方法「共享」，只创建一次，大幅节省内存，性能最优

- ✅ 兼顾批量创建和复用性，是原生 JS 中「最推荐的批量创建方案」

- ✅ 支持原型链查找，可灵活扩展公共方法/属性

- ✅ 企业开发中，原生 JS 场景首选此方案（比如封装原生组件、工具类）

---

## 4. `Object.create()`（基于原型创建，灵活继承方案）

`Object.create()` 是 ES5 引入的方法，核心功能是「基于一个现有对象作为原型，创建一个新对象」。它跳过了构造函数，直接通过原型对象生成新实例，适合灵活实现原型继承，或创建无原型的空对象。

**通俗理解**：相当于“复制一个现有对象的‘模板’，再基于这个模板创建新对象”，新对象会继承模板对象的所有属性和方法，同时可以自定义自己的属性。

```javascript
// 1. 定义原型对象（模板对象）
const personProto = {
  name: "默认名字", // 原型属性（可被实例覆盖）
  sayHello() {
    console.log(`我是${this.name}`);
  },
  // 原型上的方法也可以访问原型上的其他属性
  showDefaultName() {
    console.log("默认名字：", this.name);
  }
};

// 2. 创建新对象（继承 personProto）
const p1 = Object.create(personProto);
p1.name = "张三"; // 覆盖原型上的 name 属性
p1.age = 20; // 新增实例独有属性

// 3. 使用新对象
p1.sayHello(); // 输出：我是张三（访问实例自身的 name）
p1.showDefaultName(); // 输出：默认名字：张三（this 指向 p1）

// 特殊场景：创建空原型对象（无任何继承，适合做纯净的映射表）
const emptyObj = Object.create(null);
console.log(emptyObj.toString); // 输出：undefined（没有继承 Object 的原型方法）

```

### 核心特点

- ✅ 灵活实现「原型继承」，无需定义构造函数，直接复用现有对象的原型

- ✅ 可创建「空原型对象」（`Object.create(null)`），避免继承 Object 的原型方法（比如 toString、hasOwnProperty），适合做纯净的数据容器

- ❌ 不适合批量创建带参数的对象（每次创建都需要手动添加实例属性，无法像构造函数那样通过参数批量赋值）

- ✅ 适合“基于现有对象扩展”的场景（比如修改某个对象，但不想影响原对象）

---

## 5. `class` 语法（ES6 标准，最现代、最优雅）

`class` 是 ES6 引入的语法糖，本质还是「构造函数 + 原型」，只是写法更简洁、语义化更强，解决了原生构造函数+原型写法繁琐、可读性差的问题，是现代 JS 开发（Vue、React 等框架）的首选方案。

**专业说明**：`class` 语法没有改变 JS 原型继承的底层原理，只是对构造函数和原型的封装，让代码结构更清晰，更接近传统面向对象语言（比如 Java、Python）的写法。

```javascript
// 1. 定义 class（相当于构造函数的语法糖）
class Person {
  // 构造方法（相当于构造函数的函数体，new 时自动执行）
  constructor(name, age) {
    this.name = name; // 实例属性
    this.age = age;
  }

  // 实例方法（自动挂载到 Person.prototype 上，所有实例共享）
  sayHello() {
    console.log(`我是${this.name}，今年${this.age}岁`);
  }

  // 静态方法（static 关键字修饰，挂载到类本身，不被实例继承）
  static showClassName() {
    console.log("当前类：Person");
  }

  //  getter/setter（用于控制属性的读取和修改，增强属性安全性）
  get fullInfo() {
    return `${this.name}-${this.age}岁`;
  }
  set fullInfo(info) {
    const [name, age] = info.split("-");
    this.name = name;
    this.age = Number(age);
  }
}

// 2. 创建实例（和 new 构造函数用法一致）
const p1 = new Person("张三", 20);

// 3. 使用实例
p1.sayHello(); // 输出：我是张三，今年20岁
console.log(p1.fullInfo); // 输出：张三-20岁（调用 getter）
p1.fullInfo = "李四-21岁"; // 调用 setter，修改属性
console.log(p1.name); // 输出：李四

// 调用静态方法（只能通过类调用，不能通过实例调用）
Person.showClassName(); // 输出：当前类：Person
p1.showClassName(); // 报错：p1.showClassName is not a function

```

### 核心特点

- ✅ 语法清晰、语义化强，代码结构更规整，可读性远高于原生构造函数+原型

- ✅ 企业开发「首选方案」，适配所有现代框架（Vue3、React 等）

- ✅ 原生支持继承（`extends` 关键字）、静态方法（`static`）、getter/setter 等高级特性，无需手动操作原型

- ✅ 底层还是原型继承，兼顾性能和复用性，同时降低了学习和使用成本

---

## 6. 工厂函数（返回对象的函数，不推荐使用）

工厂函数是一种“模拟类”的方案，核心思路：在函数内部创建一个空对象，添加属性和方法后返回该对象，无需使用 `new` 关键字。它是 ES6 之前，为了简化批量创建对象而出现的过渡方案，现在已基本被 `class` 和原型模式替代。

```javascript
// 定义工厂函数（普通函数，无需首字母大写）
function createPerson(name, age) {
  // 1. 创建空对象
  const o = {};
  // 2. 给空对象添加属性和方法
  o.name = name;
  o.age = age;
  o.getName = function () {
    console.log(this.name);
  };
  // 3. 返回创建好的对象
  return o;
}

// 使用（不用 new，直接调用函数）
const p1 = createPerson("张三", 20);
const p2 = createPerson("李四", 21);

// 调用方法
p1.getName(); // 输出：张三

```

### 核心特点

- ✅ 不用 `new` 关键字，使用简单，无需理解原型和构造函数

- ❌ 核心缺点1：无法识别对象类型（`p1 instanceof createPerson` → false，因为没有构造函数，无法通过 `instanceof` 判断实例归属）

- ❌ 核心缺点2：方法会在每个对象中重复创建，浪费内存（和纯构造函数一样的问题）

- ❌ 不推荐使用：ES6 之后，`class` 和原型模式完全可以替代它，仅作为历史知识点了解即可

## 对象创建选型总结（实战必看）

日常开发中，无需死记所有方法，根据场景选择即可，记住以下 3 条核心规则，覆盖 99% 场景：

1. 创建「单个对象」（比如配置对象、单个数据对象）→ 用 **对象字面量 `{}`**（简洁、高效）

2. 批量创建对象、追求性能和复用性 → 用 **`class`**（现代开发首选，语义化强）

3. 原生 JS 场景、不依赖 ES6 语法 → 用 **构造函数 + 原型**（性能最优，兼容所有环境）

**补充说明**：`Object.create()` 适合特殊场景（比如原型继承、创建空对象）；工厂函数、纯构造函数（方法写在内部）尽量避免使用。

**一句话记忆**：日常开发 90% 场景用「对象字面量 + class」，剩下 10% 用「Object.create()」。

---

# 二、JS 继承方式（7种，从基础到最优，面试重点）

继承的核心是“复用父类的属性和方法”，JS 没有传统面向对象的“类继承”，而是通过「原型链」实现继承。下面从基础到最优，逐个拆解 7 种继承方式，分析其优缺点和适用场景，重点掌握“寄生组合式继承”和“class extends”。

## 1. 原型链继承（最基础，面试常考）

核心原理：**子类的原型 = 父类的实例**，让子类实例通过原型链，继承父类的实例属性和原型方法。这是最基础的继承方式，也是后续所有继承方式的基础。

```javascript
// 父类（构造函数）
function Parent() {
  this.colors = ['red', 'blue']; // 引用类型属性
  this.name = "父类";
}

// 父类原型方法
Parent.prototype.sayName = function() {
  console.log(this.name);
};

// 子类（构造函数）
function Child() {}

// 核心：子类原型 = 父类实例（实现继承）
Child.prototype = new Parent();
// 修复子类原型的 constructor 指向（否则指向 Parent）
Child.prototype.constructor = Child;

// 创建子类实例
const c1 = new Child();
const c2 = new Child();

// 测试继承效果
c1.sayName(); // 输出：父类（继承父类原型方法）
console.log(c1.colors); // 输出：['red', 'blue']（继承父类实例属性）

// 问题演示：引用类型属性被所有子类实例共享
c1.colors.push('green');
console.log(c2.colors); // 输出：['red', 'blue', 'green']（c2 的 colors 也被修改了）

```

### 核心优缺点

- ✅ 优点：实现简单，代码量少，能继承父类的实例属性和原型方法

- ❌ 缺点1：父类的引用类型属性（比如数组、对象）会被「所有子类实例共享」，一个实例修改，其他实例都会受影响（这是最致命的问题）

- ❌ 缺点2：无法向父类构造函数传递参数（比如创建子类实例时，无法给父类的 name 属性赋值）

- ❌ 缺点3：子类实例的 `constructor` 指向会被修改，需要手动修复

---

## 2. 构造函数继承（借用 call/apply，解决引用类型共享问题）

核心原理：**在子类构造函数内部，通过 call/apply 调用父类构造函数**，将父类的 this 绑定到子类实例上，让子类实例单独拥有父类的实例属性，解决原型链继承中“引用类型共享”的问题。

```javascript
// 父类
function Parent(name) {
  this.name = name; // 实例属性
  this.colors = ['red', 'blue']; // 引用类型属性
}

// 父类原型方法
Parent.prototype.sayName = function() {
  console.log(this.name);
};

// 子类
function Child(name, age) {
  // 核心：借用 call 调用父类构造函数，this 指向子类实例
  Parent.call(this, name); // 相当于给子类实例添加父类的实例属性
  this.age = age; // 子类独有属性
}

// 创建子类实例
const c1 = new Child("张三", 20);
const c2 = new Child("李四", 21);

// 测试：引用类型属性不再共享
c1.colors.push('green');
console.log(c1.colors); // 输出：['red', 'blue', 'green']
console.log(c2.colors); // 输出：['red', 'blue']（不受影响）

// 测试：无法继承父类原型方法
c1.sayName(); // 报错：c1.sayName is not a function

```

### 核心优缺点

- ✅ 优点1：避免了引用类型属性被所有实例共享的问题（每个实例都有独立的父类实例属性）

- ✅ 优点2：可以向父类构造函数传递参数（比如上面的 name 参数）

- ❌ 缺点：只能继承父类的「实例属性」，无法继承父类的「原型方法」（父类原型上的方法，子类实例无法访问）

- ❌ 缺点：父类的实例属性会在每个子类实例中重复创建（和纯构造函数一样，浪费内存）

---

## 3. 组合继承（最经典，实战常用过渡方案）

核心原理：**构造函数继承 + 原型链继承 合体**——用构造函数继承解决“引用类型共享”和“传参”问题，用原型链继承解决“继承原型方法”的问题，兼顾了两者的优点，是 ES6 之前最常用的继承方案。

```javascript
// 父类
function Parent(name) {
  this.name = name;
  this.colors = ['red', 'blue'];
}

// 父类原型方法
Parent.prototype.sayName = function() {
  console.log(this.name);
};

// 子类
function Child(name, age) {
  // 1. 构造函数继承：继承父类实例属性，解决传参和引用类型共享问题
  Parent.call(this, name);
  this.age = age;
}

// 2. 原型链继承：继承父类原型方法
Child.prototype = new Parent();
// 修复 constructor 指向
Child.prototype.constructor = Child;

// 创建子类实例
const c1 = new Child("张三", 20);
const c2 = new Child("李四", 21);

// 测试：既能继承原型方法，又能避免引用类型共享
c1.sayName(); // 输出：张三（继承原型方法）
c1.colors.push('green');
console.log(c1.colors); // ['red', 'blue', 'green']
console.log(c2.colors); // ['red', 'blue']（不受影响）

```

### 核心优缺点

- ✅ 优点：既能继承父类的实例属性，又能继承父类的原型方法，兼顾了实用性和复用性

- ✅ 优点：解决了原型链继承和构造函数继承的核心缺点，是 ES6 之前最推荐的继承方案

- ❌ 缺点：父类构造函数被调用了两次（一次是 `new Parent()` 给子类原型赋值，一次是 `Parent.call(this)` 给子类实例赋值），多执行了一遍冗余代码，造成轻微的性能浪费

---

## 4. 原型式继承（基于浅拷贝，简化原型链继承）

核心原理：**基于现有对象浅拷贝创建新对象**，本质是简化版的原型链继承，无需定义构造函数，直接通过一个现有对象作为原型，创建新对象。`Object.create()` 方法的底层实现就是原型式继承。

```javascript
// 模拟 Object.create() 的底层实现（原型式继承核心代码）
function createObj(o) {
  // 定义一个空构造函数
  function F() {}
  // 让空构造函数的原型 = 现有对象 o
  F.prototype = o;
  // 返回空构造函数的实例（该实例的 __proto__ 指向 o）
  return new F();
}

// 现有对象（作为原型）
const parent = {
  name: "父对象",
  colors: ['red', 'blue'],
  sayName() {
    console.log(this.name);
  }
};

// 创建新对象（继承 parent）
const c1 = createObj(parent);
const c2 = createObj(parent);

// 测试继承效果
c1.sayName(); // 输出：父对象（继承原型方法）
console.log(c1.colors); // 输出：['red', 'blue']（继承原型属性）

// 问题：引用类型依然共享
c1.colors.push('green');
console.log(c2.colors); // 输出：['red', 'blue', 'green']

```

### 核心优缺点

- ✅ 优点：实现简单，无需定义构造函数，适合“快速复用现有对象”的场景

- ❌ 缺点1：引用类型属性依然被所有实例共享（和原型链继承一样的问题）

- ❌ 缺点2：无法向父原型对象传递参数（只能复用现有对象的属性，无法自定义）

- ✅ 补充：`Object.create()` 就是对这种方式的标准化实现，日常开发中直接用 `Object.create()` 即可，无需手动实现 `createObj`

---

## 5. 寄生式继承（原型式继承 + 对象增强）

核心原理：**在原型式继承的基础上，给新创建的对象添加额外的属性和方法（增强对象）**，本质是对原型式继承的扩展，让新对象拥有更多自定义功能。

```javascript
// 原型式继承 + 对象增强
function createChild(o) {
  // 1. 原型式继承：创建新对象，继承 o
  const clone = Object.create(o);
  // 2. 增强对象：给新对象添加独有属性和方法
  clone.sayHi = function() {
    console.log("Hi~");
  };
  clone.age = 20;
  // 3. 返回增强后的对象
  return clone;
}

// 父原型对象
const parent = {
  name: "父对象",
  sayName() {
    console.log(this.name);
  }
};

// 创建增强后的子类对象
const c1 = createChild(parent);

// 测试
c1.sayName(); // 输出：父对象（继承父原型方法）
c1.sayHi(); // 输出：Hi~（增强的方法）
console.log(c1.age); // 输出：20（增强的属性）

```

### 核心优缺点

- ✅ 优点：简单快捷，能在复用现有对象的同时，给新对象扩展自定义功能

- ❌ 缺点1：引用类型属性依然被共享（继承了原型式继承的问题）

- ❌ 缺点2：增强的方法无法复用（每次创建新对象，都会新建一次增强方法，浪费内存）

- ❌ 适用场景有限：仅适合“一次性创建一个增强对象”的场景，不适合批量创建

---

## 6. 寄生组合式继承（最完美、最优，面试必背）

核心原理：**构造函数继承 + 原型式继承（Object.create）**，解决了组合继承“父构造函数被调用两次”的问题，同时保留了组合继承的所有优点，是 JS 继承的最佳实践，也是 `class extends` 的底层实现原理。

**核心改进**：用 `Object.create(Parent.prototype)` 替代 `new Parent()` 给子类原型赋值，这样只会继承父类的原型，不会调用父类构造函数，避免了冗余代码。

```javascript
// 父类
function Parent(name) {
  this.name = name;
  this.colors = ['red', 'blue'];
}

// 父类原型方法
Parent.prototype.sayName = function() {
  console.log(this.name);
};

// 子类
function Child(name, age) {
  // 1. 构造函数继承：继承父类实例属性，传参，避免引用类型共享
  Parent.call(this, name);
  this.age = age;
}

// 2. 核心改进：用 Object.create 继承父类原型，不调用父类构造函数
Child.prototype = Object.create(Parent.prototype);
// 修复 constructor 指向
Child.prototype.constructor = Child;

// 创建子类实例
const c1 = new Child("张三", 20);
const c2 = new Child("李四", 21);

// 测试：完美解决所有问题
c1.sayName(); // 输出：张三（继承原型方法）
c1.colors.push('green');
console.log(c1.colors); // ['red', 'blue', 'green']
console.log(c2.colors); // ['red', 'blue']（不共享）
console.log(c1.constructor); // 输出：Child（constructor 指向正确）

```

### 核心优缺点

- ✅ 优点1：只调用一次父类构造函数，避免了组合继承的冗余代码，性能最优

- ✅ 优点2：引用类型属性不共享，每个子类实例都有独立的父类实例属性

- ✅ 优点3：能继承父类的实例属性和原型方法，原型链结构正常

- ✅ 优点4：子类实例的 `constructor` 指向正确，无需额外修复（修复仅为规范，不影响功能）

- ❌ 缺点：写法比 `class extends` 繁琐（这也是 ES6 引入 `class` 的原因）

- ✅ 结论：JS 原生继承的「最佳实践」，面试必考，也是 `class extends` 的底层原理

---

## 7. class extends（ES6 语法糖，现代开发首选）

核心原理：ES6 引入的 `extends` 关键字，本质是「寄生组合式继承」的语法糖，写法更简洁、语义化更强，无需手动操作原型和构造函数，是现代 JS 开发（框架、项目）的首选继承方式。

```javascript
// 父类（class 语法）
class Parent {
  constructor(name) {
    this.name = name;
    this.colors = ['red', 'blue'];
  }

  // 父类原型方法
  sayName() {
    console.log(this.name);
  }

  // 父类静态方法
  static showParent() {
    console.log("这是父类");
  }
}

// 子类继承父类（extends 关键字）
class Child extends Parent {
  constructor(name, age) {
    // 核心：super() 相当于 Parent.call(this, name)，调用父类构造函数
    super(name); // 必须在 this 之前调用，否则报错
    this.age = age; // 子类独有属性
  }

  // 子类原型方法
  sayAge() {
    console.log(`我今年${this.age}岁`);
  }
}

// 创建子类实例
const c1 = new Child("张三", 20);

// 测试继承效果
c1.sayName(); // 输出：张三（继承父类原型方法）
c1.sayAge(); // 输出：我今年20岁（子类自有方法）
Parent.showParent(); // 输出：这是父类（父类静态方法）
Child.showParent(); // 输出：这是父类（子类继承父类静态方法）

// 测试引用类型不共享
const c2 = new Child("李四", 21);
c1.colors.push('green');
console.log(c1.colors); // ['red', 'blue', 'green']
console.log(c2.colors); // ['red', 'blue']

```

### 核心特点

- ✅ 写法最优雅、语义化最强，无需手动操作原型，降低学习和使用成本

- ✅ 底层是寄生组合式继承，兼顾性能和实用性，无任何明显缺点

- ✅ 原生支持 `super` 关键字（调用父类构造函数、访问父类方法），支持静态方法继承

- ✅ 现代开发首选，适配所有框架（Vue3、React、Node.js 等）

## 继承方式总结（面试必背）

用一句话总结每种继承的核心特点，面试时直接套用，清晰易懂：

1. 原型链继承：简单但引用类型共享，无法传参

2. 构造函数继承：能传参、避免共享，但无法继承原型方法

3. 组合继承：好用但父构造函数被调用两次，有冗余代码

4. 原型式继承：浅拷贝继承，适合快速复用现有对象

5. 寄生式继承：拷贝+增强，方法无法复用

6. 寄生组合继承：**完美继承，无缺点**，原生最优方案

7. class extends：语法糖，底层是寄生组合继承，现代开发首选

---

# 三、面试高频补充：instanceof 原理与模拟 new

理解了对象创建和继承，就必须掌握 `instanceof` 和 `new` 的底层原理，这两个是面试高频考点，下面用通俗的语言+代码模拟，帮你彻底搞懂。

## 1. instanceof 原理（判断对象类型的核心）

作用：判断一个对象（obj）是否是某个构造函数（constructor）的实例，本质是「沿着对象的原型链向上查找，看是否能找到构造函数的 prototype」。

核心逻辑：

1. 获取构造函数的原型对象（constructor.prototype）；

2. 沿着对象的 `__proto__`（隐式原型）逐级向上查找；

3. 如果找到某个原型对象和构造函数的 prototype 相等，返回 true；

4. 如果查到原型链顶端（null）还没找到，返回 false。

```javascript
// 模拟 instanceof 底层实现（myInstanceof）
function myInstanceof(obj, constructor) {
  // 1. 边界判断：如果 obj 是 null/undefined，直接返回 false
  if (obj === null || typeof obj !== 'object' && typeof obj !== 'function') {
    return false;
  }
  // 2. 获取构造函数的原型对象
  const prototype = constructor.prototype;
  // 3. 逐级获取 obj 的隐式原型（__proto__），向上查找
  while (true) {
    // 查到顶端 null，说明没找到，返回 false
    if (obj === null) {
      return false;
    }
    // 找到原型相等，说明是实例，返回 true
    if (obj === prototype) {
      return true;
    }
    // 核心：沿着原型链向上走一层（获取下一个原型对象）
    // 推荐用 Object.getPrototypeOf(obj) 替代 obj.__proto__（更规范）
    obj = Object.getPrototypeOf(obj);
  }
}

// 测试
function Person() {}
const p = new Person();

console.log(myInstanceof(p, Person)); // 输出：true（p 是 Person 的实例）
console.log(myInstanceof(p, Object)); // 输出：true（p 也是 Object 的实例，因为 Person.prototype.__proto__ 指向 Object.prototype）
console.log(myInstanceof([], Array)); // 输出：true
console.log(myInstanceof(123, Number)); // 输出：false（123 是基本类型，不是 Number 的实例）

```

## 2. 模拟 new 关键字（对象创建的核心）

`new` 关键字的作用是“通过构造函数创建实例”，其底层逻辑可拆解为 4 步，我们用代码模拟其实现，就能彻底理解。

new 的核心逻辑：

1. 判断传入的 Constructor 是否是函数，不是则报错；

2. 创建一个空对象，并且让这个空对象的 `__proto__` 指向 Constructor 的 prototype（实现原型继承）；

3. 调用 Constructor 构造函数，将 this 绑定到刚创建的空对象上，并传入参数；

4. 判断构造函数的返回值：如果返回的是对象/函数，就返回这个返回值；否则返回刚创建的空对象。

```javascript
// 模拟 new 关键字（myNew）
function myNew(Constructor, ...args) {
  // 1. 边界判断：如果 Constructor 不是函数，报错
  if (typeof Constructor !== 'function') {
    throw new TypeError(Constructor + ' is not a constructor');
  }

  // 2. 创建空对象，并且让其 __proto__ 指向 Constructor.prototype
  const instance = Object.create(Constructor.prototype);

  // 3. 执行构造函数，this 绑定到 instance，传入参数
  const result = Constructor.apply(instance, args);

  // 4. 判断返回值：如果返回引用类型（对象/函数），则返回该值；否则返回 instance
  // 注意：null 是对象类型，但要排除（因为返回 null 时，依然返回 instance）
  return (typeof result === 'object' && result !== null) || typeof result === 'function' 
    ? result 
    : instance;
}

// 测试1：正常情况
function Person(name) {
  this.name = name;
}
const p1 = myNew(Person, '张三');
console.log(p1.name); // 输出：张三
console.log(p1 instanceof Person); // 输出：true

// 测试2：构造函数返回对象（特殊情况）
function Student(name) {
  this.name = name;
  // 手动返回一个对象
  return {
    name: '李四',
    age: 20
  };
}
const s1 = myNew(Student, '张三');
console.log(s1.name); // 输出：李四（返回的是构造函数手动返回的对象）
console.log(s1 instanceof Student); // 输出：false（因为返回的对象不是 Student 的实例）

// 测试3：构造函数返回基本类型（不影响）
function Teacher(name) {
  this.name = name;
  return 123; // 返回基本类型
}
const t1 = myNew(Teacher, '王五');
console.log(t1.name); // 输出：王五（返回的是 instance）

```

---

# 四、最终总结

1. 对象创建：优先用「对象字面量」（单个对象）和「class」（批量对象），原生场景用「构造函数+原型」，特殊场景用「Object.create()」；

2. 继承：现代开发首选「class extends」，面试重点掌握「寄生组合式继承」，其他方式作为基础知识点了解；

3. 核心底层：理解「原型链」「instanceof 原理」「new 原理」，这是 JS 对象和继承的核心，也是面试高频考点；

4. 一句话实战：日常开发用「对象字面量 + class + extends」，能覆盖所有场景，简洁又高效。