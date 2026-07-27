## 前言

TypeScript 作为 JavaScript 的静态类型超集，已经成为现代前端工程化的标准配置。它通过在编译期进行类型检查，能够提前捕获大量潜在的运行时错误，同时借助类型注解与类型推断，大幅提升了代码的可读性、可维护性与团队协作效率。对于中大型项目、长期维护的库与复杂业务系统，TypeScript 带来的类型收益会随着项目迭代愈发明显。

本文从最基础的类型系统讲起，逐步深入到联合类型、泛型编程、工具类型与高阶类型技巧，最终通过两个完整的工业级实战案例，展示 TypeScript 在真实项目中的落地方式。全文兼顾概念讲解、代码示例与工程实践，无论你是刚接触 TS 的初学者，还是希望系统巩固进阶知识的开发者，都能通过这篇文章建立完整的 TypeScript 知识体系。

---

## 目录

1. TypeScript 基础类型系统

2. 类型进阶：联合、交叉与字面量

3. 类型断言与类型守卫

4. type vs interface：终极对比

5. 泛型：类型编程的核心武器

6. 工具类型：TypeScript 内置类型工具箱

7. 函数重载：精准的类型声明

8. 类型收窄：让 TypeScript 更智能

9. 实战：泛型封装通用拖拽组件

10. 实战：全局状态管理的 TS 类型约束

11. 进阶技巧与最佳实践

---

## 一、TypeScript 基础类型系统

类型系统是 TypeScript 的核心能力，而基础类型则是构建整个类型大厦的最小单元。和 JavaScript 运行时才确定变量类型不同，TypeScript 要求我们在编码阶段显式声明类型，从而在编译期就能发现类型不匹配的问题。大部分简单场景下 TypeScript 可以通过类型推断自动识别类型，但在函数参数、返回值、模块接口等位置，显式的类型声明能让代码意图更加清晰。

### 1\.1 原始类型

原始类型对应 JavaScript 的七种基础数据类型，是 TypeScript 中最常用的类型注解。其中 `string`、`number`、`boolean` 是最基础的三类，分别对应字符串、数字与布尔值。

`null` 和 `undefined` 是比较特殊的两个类型，在严格模式下它们是其他类型的子类型，只能赋值给自身和 `unknown`、`any`。`bigint` 用于处理超过 Number 安全整数范围的大整数运算，`symbol` 则用于创建唯一的对象属性键，常用于避免对象属性名冲突，或者实现类的私有属性模拟。

```typescript
// 基本原始类型
let name: string = 'John';
let age: number = 30;
let isActive: boolean = true;
let nothing: null = null;
let undefinedValue: undefined = undefined;
let bigNumber: bigint = 9007199254740991n;
let symbolKey: symbol = Symbol('key');

// symbol 作为对象属性键，保证属性名唯一
const uniqueKey = Symbol('unique');
const obj = {
  [uniqueKey]: 'secret value'
};
```

### 1\.2 数组与元组

数组是开发中最常用的集合类型，TypeScript 提供了两种声明方式：`类型[]` 和 `Array<类型>` 泛型写法，二者功能完全等价，选择哪种取决于团队代码风格。

只读数组通过 `readonly` 关键字声明，禁止对数组元素进行修改，适合用于不应被变更的常量集合，能避免意外的副作用。

元组是一种特殊的数组，它规定了固定的长度和每个位置的类型，非常适合用于函数返回多个返回值的场景，比如 React Hooks 的返回值就是典型的元组类型。元组还支持可选元素和剩余元素语法，进一步提升了灵活性。

```typescript
// 数组定义方式
let numbers: number[] = [1, 2, 3];
let strings: Array<string> = ['a', 'b', 'c'];

// 只读数组，禁止修改元素，适合常量配置
let readonlyArr: readonly number[] = [1, 2, 3];
// readonlyArr[0] = 10; // 错误：只读数组不可修改

// 元组：固定长度和类型的数组，常用于多返回值
let tuple: [string, number, boolean] = ['hello', 42, true];
let [str, num, bool] = tuple;

// 元组的可选元素
let optionalTuple: [string, number?] = ['hello'];
// 剩余元素，元组开头固定，后面可跟任意数量指定类型元素
let restTuple: [string, ...number[]] = ['start', 1, 2, 3];
```

### 1\.3 any、unknown、never、void

这四个特殊类型是 TypeScript 类型系统的边界类型，各自对应完全不同的使用场景，也是面试中的高频考点。

`any` 相当于关闭类型检查，变量可以被赋值为任意类型，也可以调用任意方法，完全退化为 JavaScript。虽然使用灵活，但会丢失所有类型安全保障，实际开发中应当尽量避免滥用，仅在临时兼容旧代码时谨慎使用。

`unknown` 被称为「安全的 any」，它和 any 一样可以接收任意类型的值，但 unknown 类型的变量不能直接调用任何方法，必须先通过类型守卫收窄类型后才能使用，是处理动态数据、第三方接口返回值的推荐类型。

`void` 用于标记函数没有返回值，对应函数中没有 return 语句或者 return 不带值的情况。

`never` 表示永远不会有返回值的函数，比如函数内部抛出错误、或者是死循环函数，它是所有类型的子类型，但没有任何类型可以赋值给 never。

```typescript
// any - 关闭类型检查（谨慎使用，丢失所有类型安全）
let anything: any = 42;
anything = 'string';
anything.toUpperCase();

// unknown - 安全的 any（必须先收窄类型才能使用）
let unknownValue: unknown = 'hello';
// unknownValue.toUpperCase(); // 直接调用会报错
if (typeof unknownValue === 'string') {
  unknownValue.toUpperCase(); // 收窄后可正常使用
}

// void - 函数无返回值，仅用于函数返回类型声明
function logMessage(msg: string): void {
  console.log(msg);
  // 不返回任何值
}

// never - 永远不会正常返回的函数
function throwError(message: string): never {
  throw new Error(message);
}

function infiniteLoop(): never {
  while (true) {}
}
```

---

## 二、类型进阶：联合、交叉与字面量

掌握基础类型后，我们需要通过类型组合来构建更复杂的类型描述。联合类型、交叉类型与字面量类型是 TypeScript 中最基础的类型组合方式，它们让我们能够用基础类型拼接出贴合业务语义的复杂类型。

### 2\.1 联合类型（Union Types）

联合类型用 `|` 符号连接多个类型，表示变量可以是其中任意一种类型，非常适合描述「或」的语义。比如状态可以是成功、失败、加载中三种之一，ID 可以是字符串或者数字，这些场景都非常适合用联合类型表达。

联合类型的变量在未收窄之前，只能访问所有类型共有的属性和方法，必须通过类型守卫收窄到具体类型后，才能访问对应类型的专属属性。

```typescript
// 基础联合类型，描述有限的状态集合
type Status = 'pending' | 'success' | 'error';
let currentStatus: Status = 'pending';

// 多类型联合，支持不同数据类型的组合
type ID = string | number;
function getUserId(id: ID): string {
  return `User: ${id}`;
}

// 联合类型必须收窄后才能访问专属属性
function processValue(value: string | number) {
  if (typeof value === 'string') {
    return value.toUpperCase();
  } else {
    return value.toFixed(2);
  }
}
```

### 2\.2 交叉类型（Intersection Types）

交叉类型用 `&` 符号连接多个类型，表示同时拥有所有类型的属性，对应「且」的语义，最常用于对象类型的合并。

需要注意的是，如果交叉的两个原始类型存在冲突，最终结果会变成 never 类型，比如 `string & number` 是不存在的，结果为 never。交叉类型在对象合并、Mixin 模式、组件 Props 组合等场景中非常实用。

```typescript
// 合并多个对象类型的属性
interface Person {
  name: string;
  age: number;
}

interface Employee {
  employeeId: string;
  department: string;
}

type EmployeePerson = Person & Employee;

const employee: EmployeePerson = {
  name: 'Alice',
  age: 30,
  employeeId: 'EMP001',
  department: 'Engineering'
};

// 交叉类型常用于泛型对象合并函数的返回类型
function merge<T, U>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 };
}

const merged = merge({ a: 1 }, { b: 2 });
// merged 类型为 { a: number } & { b: number }
```

### 2\.3 字面量类型（Literal Types）

字面量类型把类型约束为具体的某个值，而不是某一类值，是实现类型级枚举的基础。字符串字面量、数字字面量和布尔字面量是最常用的三类，配合联合类型可以实现比传统枚举更轻量的状态约束。

模板字面量类型是 TypeScript 4\.1 新增的强大特性，它支持对字符串类型进行拼接、大小写转换等操作，能够生成非常灵活的类型集合，在事件系统、路由配置等场景有大量应用。

```typescript
// 字符串字面量，约束为固定的几个字符串值
type Direction = 'up' | 'down' | 'left' | 'right';
function move(direction: Direction) {
  // ...
}

// 数字字面量
type Dice = 1 | 2 | 3 | 4 | 5 | 6;

// 布尔字面量
type Truthy = true;
type Falsy = false;

// 模板字面量类型（TS 4.1+），支持字符串类型运算
type EventName<T extends string> = `on${Capitalize<T>}`;
type ClickEvent = EventName<'click'>; // 推导结果为 'onClick'
```

### 2\.4 对象类型

对象是 JavaScript 中最核心的数据结构，TypeScript 提供了丰富的语法来描述对象的结构。可选属性用 `?` 标记，表示该属性可以不存在；只读属性用 `readonly` 标记，禁止修改属性值，适合用于配置对象等不应被修改的场景。

索引签名用于描述键名不固定的字典类对象，只要键和值符合类型要求就可以任意添加属性。`keyof` 操作符可以提取对象类型的所有键组成联合类型，是泛型编程中非常常用的工具。

```typescript
// 基础对象类型声明
interface User {
  id: number;
  name: string;
  email?: string; // 可选属性，可不存在
  readonly createdAt: Date; // 只读属性，初始化后不可修改
}

// 索引签名，描述字典结构，键值类型统一
interface Dictionary {
  [key: string]: string;
}

const dict: Dictionary = {
  hello: 'world',
  foo: 'bar'
};

// 索引访问类型，提取对象某个属性的类型
type UserName = User['name']; // string
// keyof 提取所有键组成联合类型
type UserKeys = keyof User; // 'id' | 'name' | 'email' | 'createdAt'
```

---

## 三、类型断言与类型守卫

TypeScript 的类型推断并非万能，当开发者比编译器更清楚某个值的类型时，我们可以通过类型断言手动指定类型；而类型守卫则是更安全、更推荐的类型收窄方式，能够在运行时校验类型的同时，让编译器自动收窄类型。

### 3\.1 类型断言

类型断言相当于告诉编译器「我比你更清楚这个值的类型，听我的」，它不会进行任何运行时检查，纯粹是编译期的类型覆盖，因此使用时需要谨慎，避免错误断言导致运行时bug。

TypeScript 提供了两种断言语法：`as` 语法和尖括号语法，功能完全一致，不过在 JSX 中尖括号会和标签冲突，因此推荐统一使用 `as` 语法。

非空断言 `!` 是一种特殊的断言，用于告诉编译器某个值一定不是 null 或 undefined，适合在我们确定值存在，但编译器无法推断的场景。

`as const` 常量断言是非常实用的技巧，它可以把对象或数组的类型收窄为最精确的字面量类型，并且自动加上只读属性，非常适合定义常量配置。

```typescript
// 两种断言语法，推荐统一使用 as 语法
const element = document.getElementById('app') as HTMLDivElement;
const input = <HTMLInputElement>document.getElementById('input'); // 旧语法，JSX中不可用

// 非空断言，明确告诉编译器该值不为 null/undefined
const maybeValue: string | undefined = getValue();
const valueLength = maybeValue!.length;

// const 断言，将对象收窄为字面量只读类型
const config = {
  name: 'app',
  version: '1.0.0'
} as const;
// config.name 类型为 'app'，而不是宽泛的 string
```

### 3\.2 类型守卫

类型守卫是一种特殊的函数，它的返回值是类型谓词 `value is T`，当函数返回 true 时，编译器会自动把参数的类型收窄为 T。相比类型断言，类型守卫有真实的运行时校验，更加安全，是处理 unknown 类型、联合类型的推荐方式。

`typeof` 守卫适合基础类型判断，`instanceof` 守卫适合类实例判断，`in` 操作符守卫适合判断对象是否存在某个属性，三种守卫分别对应不同的使用场景。

```typescript
// typeof 类型守卫，用于基础类型判断
function isString(value: unknown): value is string {
  return typeof value === 'string';
}

// instanceof 类型守卫，用于类实例判断
class Animal {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}

class Dog extends Animal {
  bark() { console.log('woof'); }
}

function isDog(animal: Animal): animal is Dog {
  return animal instanceof Dog;
}

// in 操作符守卫，通过属性是否存在区分对象类型
interface Circle {
  kind: 'circle';
  radius: number;
}

interface Square {
  kind: 'square';
  sideLength: number;
}

function getArea(shape: Circle | Square): number {
  if ('radius' in shape) {
    return Math.PI * shape.radius ** 2;
  } else {
    return shape.sideLength ** 2;
  }
}

// 自定义类型守卫，可实现任意复杂的类型校验逻辑
function isValidUser(user: any): user is User {
  return user && typeof user.id === 'number' && typeof user.name === 'string';
}
```

---

## 四、type vs interface：终极对比

`type` 和 `interface` 是 TypeScript 中定义对象类型最常用的两种方式，很多初学者会困惑它们的区别。实际上两者在大部分场景下功能重叠，但各自有独特的特性，适用于不同的开发场景。

### 4\.1 核心区别

|特性|`type`|`interface`|
|---|---|---|
|声明合并|❌ 不支持|✅ 支持（自动合并同名接口）|
|扩展方式|`&`（交叉类型）|`extends`|
|元组类型|✅ 支持|❌ 不支持|
|联合类型|✅ 支持|❌ 不支持|
|映射类型|✅ 支持|❌ 不支持|
|计算属性|✅ 支持|❌ 不支持|
|类实现|✅ 支持|✅ 支持|
|性能|一般|更优（有内部缓存）|

### 4\.2 详细对比与使用示例

interface 最独特的特性是声明合并，也就是同名的 interface 会自动合并属性，这个特性非常适合库开发场景——库开发者可以预留扩展接口，用户可以通过声明同名 interface 来扩展类型定义，比如 Vue 3 的全局组件类型扩展就是利用这个特性。

interface 的另一个优势是性能更好，TypeScript 对 interface 有专门的缓存优化，在大型项目中大量使用 interface 会比 type 有更好的类型检查性能。

type 的优势在于灵活性，它支持联合类型、元组类型、映射类型、条件类型等高级类型操作，能够表达更复杂的类型逻辑，是类型编程的主力工具。业务开发中需要组合复杂类型时，type 会更加得心应手。

```typescript
// ========== interface 的特性 ==========
// 1. 声明合并（自动合并同名接口，库开发必备）
interface User {
  id: number;
}

interface User {
  name: string;
}
// 最终 User 类型自动合并为 { id: number; name: string }

// 2. 继承，语义更清晰
interface Animal {
  name: string;
}

interface Dog extends Animal {
  breed: string;
}

// 3. 类实现契约
class Person implements User {
  id = 1;
  name = 'John';
}

// ========== type 的特性 ==========
// 1. 联合类型，interface 无法实现
type Status = 'active' | 'inactive';

// 2. 元组类型
type Tuple = [string, number];

// 3. 映射类型，类型编程的核心能力
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

// 4. 计算属性，支持键名重映射
type EventMap<T> = {
  [K in keyof T as `on${Capitalize<string & K>}`]: (event: T[K]) => void;
};

// 5. 条件类型，实现类型级别的分支逻辑
type IsString<T> = T extends string ? true : false;

// 6. 交叉类型合并对象
type Combined = { a: number } & { b: string };
```

### 4\.3 使用建议

实际开发中不需要严格区分两者，遵循以下原则即可：

- 定义对象结构、API 响应类型、类的契约时，优先使用 interface，它的语义更清晰，性能更好

- 需要联合类型、元组、工具类型、映射类型等高级类型操作时，使用 type

- 库开发、需要支持类型扩展的场景，必须使用 interface 来支持声明合并

- 团队内部保持统一风格即可，不必过度纠结

```typescript
// ✅ 推荐使用 interface 的场景：
// - 定义对象结构（尤其是 API 响应）
// - 需要声明合并（库开发）
// - 类实现契约
interface APIResponse {
  data: any;
  status: number;
  message: string;
}

// ✅ 推荐使用 type 的场景：
// - 联合类型
// - 元组类型
// - 工具类型
// - 映射/条件类型
type APIStatus = 200 | 201 | 400 | 401 | 500;
type Nullable<T> = T | null;
```

---

## 五、泛型：类型编程的核心武器

泛型是 TypeScript 从简单类型注解进阶到类型编程的核心，它允许我们把类型作为参数传入，从而实现类型的复用，同时保留完整的类型安全。没有泛型的话，我们要么写大量重复的类型定义，要么用 any 丢失类型安全，泛型完美解决了这个问题。

### 5\.1 基础泛型

最简单的泛型就是泛型函数，我们在函数名后用尖括号声明类型参数 `T`，就可以在参数、返回值中使用这个类型。调用时既可以显式指定类型，也可以让 TypeScript 根据传入的参数自动推断类型，大部分场景下类型推断都能正常工作，不需要手动指定。

泛型约束通过 `extends` 关键字实现，它要求传入的类型参数必须满足某个结构，比如我们要访问参数的 length 属性，就必须约束 T 必须拥有 length 属性，否则编译器会报错。

泛型支持多个参数，也支持设置默认值，和函数参数的使用逻辑非常相似。

```typescript
// 最简单的泛型函数，身份函数
function identity<T>(value: T): T {
  return value;
}

// 显式指定类型
const result1 = identity<string>('hello');
// 类型推断，自动根据参数推断出 T 为 number
const result2 = identity(42);

// 泛型约束，要求 T 必须拥有 length 属性
interface Lengthwise {
  length: number;
}

function logLength<T extends Lengthwise>(value: T): T {
  console.log(value.length);
  return value;
}

// 多个泛型参数
function merge<T, U>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 };
}

// 泛型默认值，不传时默认使用 string
function createArray<T = string>(length: number, value: T): T[] {
  return Array(length).fill(value);
}

const arr1 = createArray(3, 'hello'); // 推断为 string[]
const arr2 = createArray<number>(3, 42); // 显式指定为 number[]
```

### 5\.2 泛型接口与类

泛型不仅可以用在函数上，也可以用在接口和类上，实现类型化的数据结构封装。比如栈、队列这类通用数据结构，用泛型封装后可以适配任意元素类型，同时保留完整的类型检查。

`new` 操作符约束是泛型中一个实用的技巧，它可以约束传入的参数是构造函数，从而实现通用的工厂函数。

```typescript
// 泛型接口，描述泛型函数类型
interface GenericIdentityFn<T> {
  (arg: T): T;
}

// 泛型类，封装通用栈数据结构
class Stack<T> {
  private items: T[] = [];

  push(item: T): void {
    this.items.push(item);
  }

  pop(): T | undefined {
    return this.items.pop();
  }

  peek(): T | undefined {
    return this.items[this.items.length - 1];
  }

  isEmpty(): boolean {
    return this.items.length === 0;
  }
}

// 使用时指定元素类型
const numberStack = new Stack<number>();
numberStack.push(1);
numberStack.push(2);
console.log(numberStack.pop()); // 2

// 泛型约束 + new 操作符，约束传入的是构造函数
function createInstance<T>(constructor: new () => T): T {
  return new constructor();
}
```

### 5\.3 高级泛型技巧

掌握基础泛型后，结合 keyof、条件类型、infer 等特性，我们可以实现非常强大的类型编程能力。本节将深入三个面试高频考点：infer 原理、协变/逆变、分布式条件类型。

#### 5.3.1 infer 关键字——类型推导的核心机制

`infer` 用于在条件类型中声明一个待推导的类型变量，语法为 `T extends SomeType<infer U> ? U : never`。当 TypeScript 判断 `T` 是否匹配 `SomeType<infer U>` 时，会尝试从 `T` 的结构中推导出 `U` 的具体类型。

理解 infer 的关键在于：它不是声明一个"任意"类型，而是声明一个"待填充"的占位符，由 TypeScript 在类型匹配过程中自动填充。

```typescript
// 提取函数返回值类型
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

// 提取函数参数类型（元组）
type MyParameters<T> = T extends (...args: infer P) => any ? P : never;

// 提取 Promise 内部值类型
type UnwrapPromise<T> = T extends Promise<infer V> ? V : T;

// 提取数组元素类型
type ElementType<T> = T extends (infer U)[] ? U : never;

// 实战：提取函数第一个参数的类型
type FirstArg<T> = T extends (arg: infer F, ...rest: any[]) => any ? F : never;

type Fn = (name: string, age: number) => void;
type NameType = FirstArg<Fn>; // string
```

`infer` 的匹配规则遵循协变/逆变位置（见 5.3.2），同一个类型变量在多个位置出现时，会推导为交叉类型或联合类型，这是实现 `UnionToIntersection` 的理论基础。

#### 5.3.2 协变与逆变

TypeScript 的类型参数在复合类型中会表现出不同的变体行为，理解变体对面试和高级类型编程至关重要。

- **协变（Covariant）**：保持子类型关系。如果 `A extends B`，则 `SomeType<A> extends SomeType<B>`。
  - 示例：`Promise<Dog>` 可赋值给 `Promise<Animal>`（假设 Dog 是 Animal 的子类型）。
  - 函数的返回值类型是协变的。

- **逆变（Contravariant）**：反转子类型关系。如果 `A extends B`，则 `SomeType<B> extends SomeType<A>`。
  - 示例：`(animal: Animal) => void` 可赋值给 `(dog: Dog) => void`。
  - 函数的参数类型是逆变的（strictFunctionTypes 开启时）。

- **不变（Invariant）**：既不协变也不逆变，要求精确匹配。

- **双变（Bivariant）**：既协变又逆变，默认关闭 strictFunctionTypes 时函数参数表现为双变。

```typescript
// 协变——返回值类型
type ReturnTypeFunc<T> = () => T;
// 如果 Dog extends Animal，则 ReturnTypeFunc<Dog> extends ReturnTypeFunc<Animal>

// 逆变——参数类型（strictFunctionTypes 模式下）
type ParamFunc<T> = (arg: T) => void;
// 如果 Dog extends Animal，则 ParamFunc<Animal> extends ParamFunc<Dog>
// 即：参数类型逆变的含义是"能处理 Animal 的函数，当然也能处理 Dog"

// 面试题：利用逆变实现 UnionToIntersection
type UnionToIntersection<U> = 
  (U extends any ? (arg: U) => void : never) extends (arg: infer I) => void ? I : never;

// 原理拆解：
// 1. 分布式条件类型将联合类型拆开，每个分支返回 (arg: T) => void
// 2. 得到一个函数联合类型
// 3. 利用函数参数位置的逆变特性，将联合类型转换为交叉类型

type Test = UnionToIntersection<{a: number} | {b: string}>;
// 结果：{a: number} & {b: string}
```

#### 5.3.3 分布式条件类型

当条件类型 `T extends U ? X : Y` 中的 `T` 是一个裸类型参数（即未被包裹在元组、数组或其他泛型中）且是联合类型时，条件类型会"分布式"地应用到联合类型的每个成员上。

```typescript
// 分布式条件类型：T 被展开为联合类型的每个成员
type ToArray<T> = T extends any ? T[] : never;
type Result = ToArray<string | number>;
// 实际推导：string[] | number[]（不是 (string | number)[]）

// 手动展开验证：
// 1. ToArray<string> → string[]
// 2. ToArray<number> → number[]
// 3. 结果为 string[] | number[]

// 关闭分布式条件类型：将 T 包裹在元组中
type ToArrayNonDist<T> = [T] extends [any] ? T[] : never;
type Result2 = ToArrayNonDist<string | number>;
// 结果：(string | number)[]，因为 [string | number] extends [any] 是整体判断

// 实战：精确的 Exclude 实现
type MyExclude<T, U> = T extends U ? never : T;
type TestExclude = MyExclude<'a' | 'b' | 'c', 'a'>;
// 结果：'b' | 'c'（分布式条件类型逐个判断）

// 利用分布式条件类型提取特定类型
type ExtractString<T> = T extends string ? T : never;
type StringsOnly = ExtractString<string | number | boolean>;
// 结果：string
```

### 5.4 递归泛型与模板字面量类型

#### 5.4.1 递归泛型

递归泛型通过在类型定义中引用自身，实现对嵌套对象的深度处理。每次递归调用都会处理当前层级，然后递归处理下一层级，直到满足终止条件。

```typescript
// 深度只读——递归处理嵌套对象
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object
    ? T[P] extends Function
      ? T[P]
      : DeepReadonly<T[P]>
    : T[P];
};

// 深度 Partial——每层属性都变为可选
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object
    ? T[P] extends Function
      ? T[P]
      : DeepPartial<T[P]>
    : T[P];
};

// 深度 Required——每层属性都变为必选
type DeepRequired<T> = {
  [P in keyof T]-?: T[P] extends object
    ? T[P] extends Function
      ? T[P]
      : DeepRequired<T[P]>
    : T[P];
};

// 实战：递归提取对象所有路径
type Paths<T, Prefix extends string = ''> = {
  [K in keyof T]: T[K] extends object
    ? Paths<T[K], `${Prefix}${Extract<K, string>}.`>
    : `${Prefix}${Extract<K, string>}`;
}[keyof T];

type Config = {
  server: { host: string; port: number };
  auth: { token: string };
};

type ConfigPaths = Paths<Config>;
// 结果：'server.host' | 'server.port' | 'auth.token'

// 递归元组类型处理
type TupleToStrings<T extends any[]> = T extends [infer F, ...infer R]
  ? F extends string
    ? [F, ...TupleToStrings<R>]
    : TupleToStrings<R>
  : [];
```

#### 5.4.2 模板字面量类型

模板字面量类型（Template Literal Types，TS 4.1+）允许在类型层面构造字符串，配合 infer 和递归可以实现类型级别的字符串解析。

```typescript
// 基础模板字面量类型
type EventName = `on${Capitalize<string>}`;
type ClickHandler = `onClick`; // 合法

// 联合类型自动展开
type Size = 'small' | 'medium' | 'large';
type Color = 'red' | 'blue';
type CssClass = `${Size}-${Color}`;
// 结果：'small-red' | 'small-blue' | 'medium-red' | 'medium-blue' | 'large-red' | 'large-blue'

// 配合 infer 解析字符串
type ExtractRouteParams<T extends string> =
  T extends `${string}:${infer Param}/${infer Rest}`
    ? { [K in Param | keyof ExtractRouteParams<Rest>]: string }
    : T extends `${string}:${infer Param}`
    ? { [K in Param]: string }
    : {};

type UserRoute = '/users/:id/posts/:postId';
type UserParams = ExtractRouteParams<UserRoute>;
// 结果：{ id: string; postId: string }

// 实战：事件监听器类型推导
type EventNameType = 'focus' | 'blur' | 'change';
type EventHandler = `on${Capitalize<EventNameType>}`;
// 结果：'onFocus' | 'onBlur' | 'onChange'
```

#### 5.4.3 递归 + 模板字面量综合实战

将递归泛型和模板字面量类型结合，可以实现类型安全的深层属性更新函数。

```typescript
// 深层路径更新
type UpdateType<T, Path extends string, Value> =
  Path extends `${infer Key}.${infer Rest}`
    ? Key extends keyof T
      ? { [K in keyof T]: K extends Key ? UpdateType<T[K], Rest, Value> : T[K] }
      : never
    : Path extends keyof T
    ? { [K in keyof T]: K extends Path ? Value : T[K] }
    : never;

type Config = {
  app: {
    theme: 'light' | 'dark';
    layout: { header: boolean; sidebar: boolean };
  };
};

type UpdatedConfig = UpdateType<Config, 'app.layout.sidebar', false>;
// 结果：app.layout.sidebar 变为 false 类型
```

---

## 六、工具类型：TypeScript 内置类型工具箱

TypeScript 内置了大量实用的工具类型，它们都是基于泛型、条件类型、映射类型实现的常用类型转换工具，掌握这些工具类型可以大幅提升开发效率，避免重复造轮子。

### 6\.1 核心工具类型详解

以下是开发中最常用的 12 个工具类型，每个都对应明确的使用场景：

1. **Partial\<T\>**：把 T 的所有属性变成可选，非常适合部分更新对象的场景，比如 update 函数的参数

2. **Required\<T\>**：和 Partial 相反，把所有可选属性变成必选

3. **Readonly\<T\>**：把所有属性变成只读，用于定义不可变配置

4. **Pick\<T, K\>**：从 T 中选取指定的几个属性组成新类型

5. **Omit\<T, K\>**：从 T 中排除指定的几个属性，和 Pick 相反

6. **Exclude\<T, U\>**：从联合类型 T 中排除属于 U 的部分

7. **Extract\<T, U\>**：从联合类型 T 中提取属于 U 的部分，和 Exclude 相反

8. **NonNullable\<T\>**：从 T 中排除 null 和 undefined

9. **ReturnType\<T\>**：获取函数类型的返回值类型

10. **Parameters\<T\>**：获取函数类型的参数组成的元组类型

11. **Record\<K, T\>**：创建键为 K 类型、值为 T 类型的对象类型，适合字典结构

12. **Awaited\<T\>**：获取 Promise 包裹的内部类型

```typescript
// 示例对象类型
interface User {
  id: number;
  name: string;
  email: string;
}

// 1. Partial<T> - 所有属性变为可选，适合更新接口参数
type PartialUser = Partial<User>;
// { id?: number; name?: string; email?: string; }

// 2. Required<T> - 所有属性变为必选
type RequiredUser = Required<PartialUser>;
// { id: number; name: string; email: string; }

// 3. Readonly<T> - 所有属性变为只读
type ReadonlyUser = Readonly<User>;
// { readonly id: number; readonly name: string; readonly email: string; }

// 4. Pick<T, K> - 选取部分属性，比如用户基础信息
type UserBasic = Pick<User, 'id' | 'name'>;
// { id: number; name: string; }

// 5. Omit<T, K> - 排除部分属性，比如返回用户信息时去掉敏感字段
type UserWithoutEmail = Omit<User, 'email'>;
// { id: number; name: string; }

// 6. Exclude<T, U> - 从联合类型中排除
type T1 = Exclude<'a' | 'b' | 'c', 'a' | 'b'>; // 'c'

// 7. Extract<T, U> - 从联合类型中提取
type T2 = Extract<'a' | 'b' | 'c', 'a' | 'b'>; // 'a' | 'b'

// 8. NonNullable<T> - 排除 null 和 undefined
type T3 = NonNullable<string | number | null | undefined>; // string | number

// 9. ReturnType<T> - 获取函数返回类型
function getUser() {
  return { id: 1, name: 'John' };
}
type UserReturn = ReturnType<typeof getUser>; // { id: number; name: string }

// 10. Parameters<T> - 获取函数参数类型
type GetUserParams = Parameters<typeof getUser>; // []

// 11. Record<K, T> - 创建键值对类型，适合字典
type UserMap = Record<string, User>;
// { [key: string]: User }

// 12. Awaited<T> - 获取 Promise 返回值类型
type PromiseResult = Awaited<Promise<string>>; // string
```

### 6\.2 工具类型实现原理

理解工具类型的实现原理非常重要，不仅能帮助我们加深对泛型的理解，还能在遇到内置工具不够用的时候，自己实现自定义的工具类型。

大部分基础工具类型都是基于映射类型实现的，也就是 `[P in keyof T]` 这种遍历对象键的语法，配合可选符、readonly 修饰符来实现属性转换。

更复杂的工具类型则会结合条件类型和 infer 关键字，实现类型的提取与转换。

```typescript
// 深入了解工具类型的实现原理
// Partial 实现：遍历所有键，加上可选修饰符
type MyPartial<T> = {
  [P in keyof T]?: T[P];
};

// Readonly 实现：遍历所有键，加上 readonly 修饰符
type MyReadonly<T> = {
  readonly [P in keyof T]: T[P];
};

// Pick 实现：遍历指定的键集合，保留对应属性
type MyPick<T, K extends keyof T> = {
  [P in K]: T[P];
};

// Omit 实现：先排除键，再用 Pick 选取
type MyOmit<T, K extends keyof any> = MyPick<T, Exclude<keyof T, K>>;

// Record 实现：遍历键类型，统一赋值为 T 类型
type MyRecord<K extends keyof any, T> = {
  [P in K]: T;
};

// 自定义工具类型 - 深度 Partial，递归处理嵌套对象
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

// 自定义工具类型 - 深度只读
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

// 自定义工具类型 - 可选字段变必选，减去可选修饰符
type RequiredKeys<T> = {
  [P in keyof T]-?: T[P];
};
```

### 6\.3 高级工具类型

除了基础的对象工具类型，还有一些更进阶的工具类型技巧，能够处理更复杂的场景：

- 从函数中提取参数、返回值

- 从构造函数中提取实例类型

- 字符串操作工具类型，实现类型级别的字符串处理

- 键名重映射，生成 getter/setter 等衍生类型

```typescript
// 1. 函数参数转变为元组类型
type Arguments<T> = T extends (...args: infer A) => any ? A : never;

// 2. 获取构造函数实例类型
type InstanceType<T extends new (...args: any) => any> = 
  T extends new (...args: any) => infer R ? R : any;

// 3. 提取 Promise 内部类型（TS 4.5+ 推荐直接用内置 Awaited）
type UnwrapPromise<T> = T extends Promise<infer U> ? UnwrapPromise<U> : T;

// 4. 字符串操作工具类型（TS 4.1+）
type Greeting = 'Hello, World!';
type UppercaseGreeting = Uppercase<Greeting>; // 'HELLO, WORLD!'
type LowercaseGreeting = Lowercase<Greeting>; // 'hello, world!'
type CapitalizeGreeting = Capitalize<Greeting>; // 'Hello, world!'
type UncapitalizeGreeting = Uncapitalize<Greeting>; // 'hello, World!'

// 5. 自定义模板字面量类型
type EventHandler<T extends string> = `on${Capitalize<T>}`;
type ClickHandler = EventHandler<'click'>; // 'onClick'

// 6. 键名重映射，批量生成 getter 方法类型
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface UserInfo {
  name: string;
  age: number;
}

type UserGetters = Getters<UserInfo>;
// { getName: () => string; getAge: () => number; }
```

---

## 七、函数重载：精准的类型声明

函数重载是 TypeScript 中一个非常实用的特性，它允许我们为同一个函数定义多组不同的参数和返回值类型，从而实现更精准的类型描述。当同一个函数根据不同入参返回不同类型的结果时，联合类型无法精确描述这种对应关系，函数重载就能派上用场。

### 7\.1 基础函数重载

重载的写法分为两部分：首先写多个重载签名，描述不同的参数和返回值组合；最后写一个实现签名，它的参数和返回值必须兼容所有重载签名，并且实现签名不会对外暴露。

调用函数时，TypeScript 会从上到下匹配重载签名，匹配到第一个符合的就使用对应的类型，因此通常把更精确的重载写在前面。

```typescript
// 定义多个函数重载签名（对外可见）
function processInput(input: string): string;
function processInput(input: number): number;
function processInput(input: boolean): boolean;

// 实现签名（必须兼容所有重载，对外不可见）
function processInput(input: string | number | boolean): string | number | boolean {
  if (typeof input === 'string') {
    return input.toUpperCase();
  } else if (typeof input === 'number') {
    return input * 2;
  } else {
    return !input;
  }
}

// 使用时会自动匹配对应的重载，返回精确类型
const result1 = processInput('hello'); // 类型为 string
const result2 = processInput(42); // 类型为 number
const result3 = processInput(true); // 类型为 boolean
```

### 7\.2 复杂重载场景

重载可以处理非常多复杂的场景，比如不同参数数量、不同参数类型对应不同返回值、泛型重载、类方法重载等。

最典型的例子就是 DOM 的 createElement 函数，传入不同的标签名会返回不同的元素实例类型，这就是通过函数重载实现的。泛型和重载结合可以实现更灵活的类型匹配。

```typescript
// 1. 不同参数数量对应不同返回类型
function createElement(tag: 'div', content: string): HTMLDivElement;
function createElement(tag: 'img', src: string, alt: string): HTMLImageElement;
function createElement(tag: string, ...args: any[]): HTMLElement {
  // 实现逻辑
  const element = document.createElement(tag);
  if (tag === 'div' && args.length === 1) {
    element.textContent = args[0];
  } else if (tag === 'img' && args.length === 2) {
    (element as HTMLImageElement).src = args[0];
    (element as HTMLImageElement).alt = args[1];
  }
  return element;
}

// 2. 函数重载 + 泛型
function getData<T extends string>(id: T): Promise<{ id: T; data: string }>;
function getData<T extends number>(id: T): Promise<{ id: T; data: number }>;
function getData(id: string | number): Promise<{ id: string | number; data: any }> {
  return Promise.resolve({ id, data: 'mock data' });
}

// 3. 类方法重载
class Calculator {
  add(a: string, b: string): string;
  add(a: number, b: number): number;
  add(a: any, b: any): any {
    return a + b;
  }
}
```

### 7\.3 重载最佳实践

函数重载虽然强大，但也不能滥用。很多简单场景用联合类型或者可选参数就能解决，过度重载反而会让代码变得复杂难懂。

遵循以下原则可以让重载的使用更加合理：

- 优先使用联合类型、可选参数解决问题，实在无法精确描述时再用重载

- 重载签名数量不宜过多，通常 2\-3 个就足够

- 把最精确的重载写在最前面，最宽泛的写在最后

- 实现签名一定要兼容所有重载，避免类型和实现不一致

```typescript
// ❌ 避免过度重载，简单场景没必要用
function badOverload(a: string): string;
function badOverload(a: number, b: number): number;
function badOverload(a: any, b?: any): any {
  // 实现
}

// ✅ 使用可选参数或联合类型代替，更简洁
function goodOverload(a: string | number, b?: number): string | number {
  if (typeof a === 'string') {
    return a;
  }
  if (typeof a === 'number' && typeof b === 'number') {
    return a + b;
  }
  return 0;
}

// ✅ 优先使用最简单的重载形式
function getValue<T>(value: T): T;
function getValue<T>(value: T, defaultValue: T): T;
function getValue<T>(value: T, defaultValue?: T): T {
  return defaultValue !== undefined ? defaultValue : value;
}
```

---

## 八、类型收窄：让 TypeScript 更智能

类型收窄是 TypeScript 类型推断的核心能力，它指的是编译器根据代码中的判断逻辑，自动把宽泛的类型收窄为更精确的类型。理解类型收窄的各种方式，能让我们写出更符合 TypeScript 习惯的代码，减少不必要的类型断言。

### 8\.1 类型收窄的多种方式

TypeScript 支持多种类型收窄的方式，分别适用于不同场景：

1. **typeof 守卫**：最基础的收窄方式，用于 string、number、boolean、symbol、undefined、function、bigint、object 这八种基础类型判断

2. **instanceof 守卫**：用于判断某个对象是不是某个类的实例，适合类实例的类型区分

3. **in 操作符**：通过判断对象是否拥有某个属性来区分类型，非常适合普通对象类型的区分

4. **可辨识联合**：也叫标签联合，通过一个共有的字面量属性来区分不同类型，是处理复杂联合类型的最佳实践，Redux 的 Action 就是典型的可辨识联合

5. **自定义类型守卫**：通过返回 `value is T` 的函数实现任意逻辑的类型收窄

6. **断言函数**：通过 `asserts value is T` 语法，在函数抛出错误时收窄类型，适合参数校验场景

```typescript
// 1. typeof 类型守卫，基础类型收窄
function process(value: string | number | boolean) {
  if (typeof value === 'string') {
    // 这里 value 自动收窄为 string
    return value.length;
  }
  if (typeof value === 'number') {
    // 这里 value 自动收窄为 number
    return value.toFixed(2);
  }
  // 剩下的自动收窄为 boolean
  return value ? 'yes' : 'no';
}

// 2. instanceof 类型守卫，类实例收窄
class Cat {
  meow() { console.log('meow'); }
}

class Dog {
  bark() { console.log('woof'); }
}

function makeSound(animal: Cat | Dog) {
  if (animal instanceof Cat) {
    animal.meow();
  } else {
    animal.bark();
  }
}

// 3. in 操作符，通过属性存在性收窄
interface Circle {
  kind: 'circle';
  radius: number;
}

interface Rectangle {
  kind: 'rectangle';
  width: number;
  height: number;
}

function getArea(shape: Circle | Rectangle): number {
  if ('radius' in shape) {
    return Math.PI * shape.radius ** 2;
  }
  return shape.width * shape.height;
}

// 4. 字面量类型收窄（可辨识联合），工业级最佳实践
type Shape = 
  | { kind: 'circle'; radius: number }
  | { kind: 'square'; side: number }
  | { kind: 'rectangle'; width: number; height: number };

function getShapeArea(shape: Shape): number {
  switch (shape.kind) {
    case 'circle':
      return Math.PI * shape.radius ** 2;
    case 'square':
      return shape.side ** 2;
    case 'rectangle':
      return shape.width * shape.height;
    default:
      // never 穷尽检查，新增类型时这里会编译报错，提醒我们处理
      const exhaustiveCheck: never = shape;
      return exhaustiveCheck;
  }
}

// 5. 自定义类型守卫（is 操作符），任意逻辑收窄
function isString(value: unknown): value is string {
  return typeof value === 'string';
}

function isNumber(value: unknown): value is number {
  return typeof value === 'number' && !isNaN(value);
}

function isArray<T>(value: unknown): value is T[] {
  return Array.isArray(value);
}

// 6. 断言函数（asserts），校验失败抛出错误
function assertIsString(value: unknown): asserts value is string {
  if (typeof value !== 'string') {
    throw new Error('Value must be a string');
  }
}

function processString(value: unknown) {
  assertIsString(value);
  // 校验通过后，这里 value 自动收窄为 string
  console.log(value.length);
}
```

### 8\.2 高级类型收窄技巧

掌握基础收窄方式后，还有一些进阶技巧可以处理更复杂的场景：

- 组合多个类型谓词，实现更复杂的类型判断

- 结合可选属性和 in 操作符收窄

- 用类型守卫封装 Result 类型的成功失败判断

- 使用 satisfies 关键字在不丢失字面量类型的同时做类型校验

```typescript
// 1. 使用类型谓词组合，多条件判断
function isNumberOrString(value: unknown): value is number | string {
  return typeof value === 'number' || typeof value === 'string';
}

// 2. 对象属性存在性检查，配合可选属性
interface Config {
  url: string;
  timeout?: number;
  retries?: number;
}

function processConfig(config: Config) {
  if ('timeout' in config) {
    // config.timeout 收窄为 number 类型
    console.log(`Timeout: ${config.timeout}ms`);
  }
  if (config.retries !== undefined) {
    // 也可以用 !== undefined 判断
    console.log(`Retries: ${config.retries}`);
  }
}

// 3. 使用 is 封装 Result 类型判断，非常实用的模式
type Result<T> = 
  | { success: true; data: T }
  | { success: false; error: string };

function isSuccess<T>(result: Result<T>): result is { success: true; data: T } {
  return result.success === true;
}

function processResult<T>(result: Result<T>) {
  if (isSuccess(result)) {
    // 成功分支可以直接访问 data
    console.log(result.data);
  } else {
    // 失败分支可以直接访问 error
    console.error(result.error);
  }
}

// 4. 使用 satisfies 关键字（TS 4.9+），保留字面量类型同时校验
const config = {
  name: 'app',
  port: 3000,
  environment: 'development'
} satisfies { name: string; port: number; environment: 'development' | 'production' };
```

---

## 九、实战：泛型封装通用拖拽组件

学习了这么多理论知识，我们通过一个真实的组件封装案例，看看 TypeScript 泛型在实际项目中是如何落地的。拖拽组件是前端非常常见的通用组件，用泛型封装可以支持任意类型的拖拽元素，同时保留完整的类型安全。

### 9\.1 需求分析

我们要封装一个通用的拖拽组件，满足以下需求：

- 支持任意 HTML 元素的拖拽，不同元素类型保留对应的类型提示

- 支持拖拽开始、拖拽中、拖拽结束三个生命周期回调

- 支持拖拽约束，包括边界限制和轴向限制

- 支持事件订阅模式，可以在外部监听拖拽事件

- 提供启用、禁用、销毁、重置等控制方法

- 全程类型安全，回调参数自动推导正确类型

### 9\.2 类型定义

首先我们定义所有相关的类型，把数据结构和接口契约先确定下来。泛型参数 T 代表拖拽元素的类型，默认是 HTMLElement，用户可以传入更具体的类型比如 HTMLDivElement 来获得更精确的类型提示。

```typescript
// 拖拽配置类型，泛型 T 代表拖拽元素类型
interface DragConfig<T = HTMLElement> {
  element: T;
  onDragStart?: (event: MouseEvent) => void;
  onDragMove?: (event: MouseEvent, data: DragData) => void;
  onDragEnd?: (event: MouseEvent, data: DragData) => void;
  constraints?: DragConstraints;
  disabled?: boolean;
}

// 拖拽过程中的数据，包含位置、偏移量等信息
interface DragData {
  x: number;
  y: number;
  deltaX: number;
  deltaY: number;
  startX: number;
  startY: number;
}

// 拖拽约束配置
interface DragConstraints {
  minX?: number;
  maxX?: number;
  minY?: number;
  maxY?: number;
  axis?: 'x' | 'y' | 'both';
}

// 拖拽状态，用字面量联合类型约束
type DragState = 'idle' | 'dragging' | 'ended';

// 泛型事件映射表，定义事件名和对应回调参数
type DragEventMap<T = HTMLElement> = {
  dragstart: { target: T; event: MouseEvent };
  dragmove: { target: T; event: MouseEvent; data: DragData };
  dragend: { target: T; event: MouseEvent; data: DragData };
};
```

### 9\.3 核心实现

核心的 Draggable 类使用泛型 T 来约束元素类型，这样实例化时传入什么类型的元素，后续所有回调中的 target 都会是对应的类型，不会丢失类型信息。

事件系统也用泛型约束，on 方法只能传入 DragEventMap 中定义的事件名，回调参数会自动匹配对应的类型，实现了类型安全的事件订阅。

```typescript
class Draggable<T extends HTMLElement = HTMLElement> {
  private element: T;
  private config: DragConfig<T>;
  private state: DragState = 'idle';
  private dragData: DragData | null = null;
  private listeners: Map<keyof DragEventMap, Set<Function>> = new Map();

  constructor(config: DragConfig<T>) {
    this.config = config;
    this.element = config.element;
    this.initialize();
  }

  private initialize(): void {
    this.element.addEventListener('mousedown', this.handleMouseDown);
    this.element.style.cursor = 'move';
    this.element.style.userSelect = 'none';
  }

  private handleMouseDown = (e: MouseEvent): void => {
    if (this.config.disabled || e.button !== 0) return;

    const rect = this.element.getBoundingClientRect();
    this.dragData = {
      x: rect.left,
      y: rect.top,
      deltaX: 0,
      deltaY: 0,
      startX: e.clientX,
      startY: e.clientY,
    };

    this.state = 'dragging';
    this.emit('dragstart', { target: this.element, event: e });
    this.config.onDragStart?.(e);

    document.addEventListener('mousemove', this.handleMouseMove);
    document.addEventListener('mouseup', this.handleMouseUp);
  };

  private handleMouseMove = (e: MouseEvent): void => {
    if (this.state !== 'dragging' || !this.dragData) return;

    const { startX, startY } = this.dragData;
    let deltaX = e.clientX - startX;
    let deltaY = e.clientY - startY;

    // 应用约束条件
    const constraints = this.config.constraints;
    if (constraints) {
      if (constraints.axis === 'x') deltaY = 0;
      if (constraints.axis === 'y') deltaX = 0;

      const rect = this.element.getBoundingClientRect();
      const parentRect = this.element.parentElement?.getBoundingClientRect();

      if (parentRect) {
        if (constraints.minX !== undefined) {
          deltaX = Math.max(deltaX, constraints.minX - rect.left);
        }
        if (constraints.maxX !== undefined) {
          deltaX = Math.min(deltaX, constraints.maxX - rect.left);
        }
        if (constraints.minY !== undefined) {
          deltaY = Math.max(deltaY, constraints.minY - rect.top);
        }
        if (constraints.maxY !== undefined) {
          deltaY = Math.min(deltaY, constraints.maxY - rect.top);
        }
      }
    }

    this.dragData.deltaX = deltaX;
    this.dragData.deltaY = deltaY;
    this.dragData.x = this.dragData.startX + deltaX;
    this.dragData.y = this.dragData.startY + deltaY;

    // 应用位移
    this.element.style.transform = `translate(${deltaX}px, ${deltaY}px)`;

    this.emit('dragmove', { 
      target: this.element, 
      event: e, 
      data: this.dragData 
    });
    this.config.onDragMove?.(e, this.dragData);
  };

  private handleMouseUp = (e: MouseEvent): void => {
    if (this.state !== 'dragging') return;

    this.state = 'ended';
    if (this.dragData) {
      this.emit('dragend', { 
        target: this.element, 
        event: e, 
        data: this.dragData 
      });
      this.config.onDragEnd?.(e, this.dragData);
    }

    document.removeEventListener('mousemove', this.handleMouseMove);
    document.removeEventListener('mouseup', this.handleMouseUp);
  };

  // 类型安全的事件订阅
  on<K extends keyof DragEventMap<T>>(
    event: K,
    callback: (data: DragEventMap<T>[K]) => void
  ): void {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, new Set());
    }
    this.listeners.get(event)!.add(callback as Function);
  }

  private emit<K extends keyof DragEventMap<T>>(
    event: K,
    data: DragEventMap<T>[K]
  ): void {
    const callbacks = this.listeners.get(event);
    if (callbacks) {
      callbacks.forEach(cb => cb(data));
    }
  }

  // 公共控制方法
  enable(): void {
    this.config.disabled = false;
  }

  disable(): void {
    this.config.disabled = true;
  }

  destroy(): void {
    this.element.removeEventListener('mousedown', this.handleMouseDown);
    document.removeEventListener('mousemove', this.handleMouseMove);
    document.removeEventListener('mouseup', this.handleMouseUp);
    this.listeners.clear();
  }

  getState(): DragState {
    return this.state;
  }

  // 重置位置到初始状态
  reset(): void {
    this.element.style.transform = 'translate(0, 0)';
    this.dragData = null;
    this.state = 'idle';
  }
}
```

### 9\.4 使用示例

使用时只需要传入元素和配置，所有回调参数都会自动推导正确的类型。如果我们传入的是 HTMLDivElement 类型的元素，那么事件回调中的 target 自动就是 HTMLDivElement 类型，不需要额外断言，这就是泛型封装的价值。

```typescript
// 基础使用，类型自动推导
const dragElement = document.getElementById('drag-target') as HTMLDivElement;
const draggable = new Draggable({
  element: dragElement,
  constraints: {
    minX: 0,
    maxX: 500,
    minY: 0,
    maxY: 400,
    axis: 'both'
  },
  onDragStart: (e) => {
    console.log('Drag started', e);
  },
  onDragMove: (e, data) => {
    console.log(`Position: (${data.x}, ${data.y})`);
  },
  onDragEnd: (e, data) => {
    console.log(`Final position: (${data.x}, ${data.y})`);
  }
});

// 事件监听，回调参数自动匹配类型
draggable.on('dragstart', ({ target, event }) => {
  console.log('Target element:', target);
});

// 动态更新约束
function updateConstraints() {
  draggable.config.constraints = {
    minX: 0,
    maxX: window.innerWidth - 200,
    minY: 0,
    maxY: window.innerHeight - 200
  };
}

// 控制拖拽状态
setTimeout(() => {
  draggable.disable();
}, 5000);

// 销毁实例，清理事件监听
// draggable.destroy();
```

---

## 十、实战：全局状态管理的 TS 类型约束

第二个实战案例我们来实现一个类型安全的全局状态管理，类似简化版的 Redux。状态管理是 TypeScript 最能发挥价值的场景之一，好的类型约束可以让 dispatch action、获取 state 全程都有类型提示，大大降低状态管理的出错概率。

### 10\.1 需求分析

我们要实现的状态管理器满足以下需求：

- 支持模块化注册状态，每个模块有自己的 state 和 reducer

- 类型安全的 dispatch，只能派发已定义的 action 类型

- 支持中间件机制，可以扩展日志、持久化等能力

- 支持状态订阅，状态变化时触发回调

- 自动推导所有模块的 state 类型和 action 类型，不需要手动声明

### 10\.2 核心类型定义

类型设计是这个案例的核心，我们通过条件类型和映射类型，从模块配置中自动推导组合后的全局状态类型、联合 Action 类型以及 Action 创建函数类型。

核心思路是：

1. 定义单个模块的配置类型 ModuleConfig

2. 通过映射类型遍历所有模块，提取每个模块的 state 类型，组合成全局 state

3. 同样提取所有模块的 action 类型，组成联合的 Action 类型

4. 最后推导出绑定 dispatch 后的 action creators 类型

```typescript
// 基础类型定义
type ActionType = string;
type Reducer<S, A> = (state: S, action: A) => S;
type Middleware<S, A> = (store: Store<S, A>) => (next: (action: A) => void) => (action: A) => void;

// 单个状态模块的配置接口
interface ModuleConfig<S, A extends Action = Action> {
  state: S;
  reducers: Record<string, Reducer<S, A>>;
  actions?: Record<string, (...args: any[]) => A>;
}

// Action 基础类型
interface Action {
  type: ActionType;
  payload?: any;
  meta?: any;
  error?: boolean;
}

// Store 核心接口
interface Store<S, A extends Action = Action> {
  getState(): S;
  dispatch(action: A): void;
  subscribe(listener: (state: S) => void): () => void;
  replaceReducer(reducer: Reducer<S, A>): void;
}

// 模块集合类型
type ModulesMap = Record<string, ModuleConfig<any, any>>;

// 组合所有模块的 state 为全局 state 类型
type CombinedState<T extends ModulesMap> = {
  [K in keyof T]: T[K] extends ModuleConfig<infer S, any> ? S : never;
};

// 组合所有模块的 action 为联合类型
type UnionActions<T extends ModulesMap> = {
  [K in keyof T]: T[K] extends ModuleConfig<any, infer A> ? A : never;
}[keyof T];

// 推导绑定 dispatch 后的 action creators 类型
type ActionCreators<T extends ModulesMap> = {
  [K in keyof T]: T[K] extends ModuleConfig<any, infer A> ? {
    [P in keyof T[K]['actions']]: T[K]['actions'][P] extends (...args: any[]) => A 
      ? (...args: Parameters<T[K]['actions'][P]>) => void 
      : never;
  } : never;
};
```

### 10\.3 实现

GlobalStore 是基础的 Store 实现，负责状态管理、dispatch 和中间件机制。StateManager 是模块化的封装，负责把多个模块组合成一个完整的 Store，并且自动生成绑定好 dispatch 的 action creators。

中间件采用和 Redux 一致的洋葱模型，通过 reduceRight 组合中间件链，保证中间件的执行顺序符合预期。

```typescript
class GlobalStore<S, A extends Action = Action> implements Store<S, A> {
  private state: S;
  private reducer: Reducer<S, A>;
  private listeners: Set<(state: S) => void> = new Set();
  private middlewares: Middleware<S, A>[] = [];
  private isDispatching = false;

  constructor(reducer: Reducer<S, A>, initialState: S) {
    this.reducer = reducer;
    this.state = initialState;
  }

  // 注册中间件
  use(middleware: Middleware<S, A>): this {
    this.middlewares.push(middleware);
    return this;
  }

  getState(): S {
    return this.state;
  }

  dispatch(action: A): void {
    if (this.isDispatching) {
      throw new Error('Reducers may not dispatch actions.');
    }

    try {
      this.isDispatching = true;

      // 组合中间件链，洋葱模型
      const dispatch = (action: A) => {
        this.state = this.reducer(this.state, action);
        this.notifyListeners();
      };

      const chain = this.middlewares.map(middleware => 
        middleware({ getState: this.getState.bind(this), dispatch: this.dispatch.bind(this) })
      );

      const composed = chain.reduceRight(
        (next, middleware) => middleware(next),
        dispatch
      );

      composed(action);
    } finally {
      this.isDispatching = false;
    }
  }

  subscribe(listener: (state: S) => void): () => void {
    this.listeners.add(listener);
    return () => {
      this.listeners.delete(listener);
    };
  }

  private notifyListeners(): void {
    const state = this.getState();
    this.listeners.forEach(listener => listener(state));
  }

  replaceReducer(reducer: Reducer<S, A>): void {
    this.reducer = reducer;
  }

  // 绑定 action creators，自动 dispatch
  bindActionCreators<C extends Record<string, (...args: any[]) => A>>(
    actionCreators: C
  ): {
    [K in keyof C]: (...args: Parameters<C[K]>) => void;
  } {
    const bound: any = {};
    for (const key in actionCreators) {
      bound[key] = (...args: any[]) => {
        this.dispatch(actionCreators[key](...args));
      };
    }
    return bound;
  }
}

// 模块化状态管理器
class StateManager<M extends ModulesMap> {
  private modules: M;
  private store: GlobalStore<CombinedState<M>, UnionActions<M>>;
  private actionCreators: ActionCreators<M>;

  constructor(modules: M) {
    this.modules = modules;
    const initialState = this.getInitialState(modules);
    const rootReducer = this.createRootReducer(modules);
    this.store = new GlobalStore(rootReducer, initialState);
    this.actionCreators = this.createActionCreators(modules) as ActionCreators<M>;
  }

  private getInitialState(modules: M): CombinedState<M> {
    const state = {} as CombinedState<M>;
    for (const key in modules) {
      state[key] = modules[key].state;
    }
    return state;
  }

  private createRootReducer(modules: M): Reducer<CombinedState<M>, UnionActions<M>> {
    return (state: CombinedState<M> | undefined, action: UnionActions<M>): CombinedState<M> => {
      if (!state) {
        return this.getInitialState(modules);
      }

      const newState = { ...state };
      for (const moduleName in modules) {
        const module = modules[moduleName];
        const moduleReducer = module.reducers[action.type];
        if (moduleReducer) {
          newState[moduleName] = moduleReducer(state[moduleName], action);
        }
      }
      return newState;
    };
  }

  private createActionCreators(modules: M): ActionCreators<M> {
    const creators = {} as ActionCreators<M>;
    for (const moduleName in modules) {
      const module = modules[moduleName];
      if (module.actions) {
        const boundActions: any = {};
        for (const actionName in module.actions) {
          boundActions[actionName] = module.actions[actionName];
        }
        creators[moduleName] = boundActions;
      }
    }
    return creators;
  }

  // 获取全局状态
  getState(): CombinedState<M> {
    return this.store.getState();
  }

  // 获取指定模块的状态
  getModuleState<K extends keyof M>(moduleName: K): M[K] extends ModuleConfig<infer S, any> ? S : never {
    return this.getState()[moduleName];
  }

  // 注册中间件
  use(middleware: Middleware<CombinedState<M>, UnionActions<M>>): this {
    this.store.use(middleware);
    return this;
  }

  // 订阅状态变化
  subscribe(listener: (state: CombinedState<M>) => void): () => void {
    return this.store.subscribe(listener);
  }

  // 获取绑定好的 action creators
  getActions(): ActionCreators<M> {
    return this.actionCreators;
  }

  // 直接 dispatch action
  dispatch(action: UnionActions<M>): void {
    this.store.dispatch(action);
  }

  // 创建模块级 dispatch
  createDispatch<K extends keyof M>(moduleName: K): (action: UnionActions<M>) => void {
    return (action: UnionActions<M>) => {
      this.store.dispatch(action);
    };
  }
}
```

### 10\.4 使用示例

使用时只需要定义每个模块的 state、reducer 和 actions，然后传入 StateManager 即可，所有类型都会自动推导。调用 action creator 的时候会有完整的参数提示，state 也有对应的类型，全程类型安全。

```typescript
// 定义 user 模块
interface User {
  id: number;
  name: string;
  email: string;
}

const userModule: ModuleConfig<User> = {
  state: {
    id: 0,
    name: '',
    email: ''
  },
  reducers: {
    'user/SET_USER': (state, action) => ({
      ...state,
      ...action.payload
    }),
    'user/UPDATE_NAME': (state, action) => ({
      ...state,
      name: action.payload
    }),
    'user/RESET': () => ({
      id: 0,
      name: '',
      email: ''
    })
  },
  actions: {
    setUser: (user: User) => ({
      type: 'user/SET_USER',
      payload: user
    }),
    updateName: (name: string) => ({
      type: 'user/UPDATE_NAME',
      payload: name
    }),
    resetUser: () => ({
      type: 'user/RESET'
    })
  }
};

// 定义 todo 模块
interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

const todoModule: ModuleConfig<Todo[]> = {
  state: [],
  reducers: {
    'todo/ADD': (state, action) => [...state, action.payload],
    'todo/TOGGLE': (state, action) => 
      state.map(todo => 
        todo.id === action.payload 
          ? { ...todo, completed: !todo.completed }
          : todo
      ),
    'todo/DELETE': (state, action) => 
      state.filter(todo => todo.id !== action.payload)
  },
  actions: {
    addTodo: (text: string): Action => ({
      type: 'todo/ADD',
      payload: { id: Date.now(), text, completed: false }
    }),
    toggleTodo: (id: number): Action => ({
      type: 'todo/TOGGLE',
      payload: id
    }),
    deleteTodo: (id: number): Action => ({
      type: 'todo/DELETE',
      payload: id
    })
  }
};

// 创建状态管理器，所有类型自动推导
const modules = {
  user: userModule,
  todo: todoModule
};

const manager = new StateManager(modules);

// 添加日志中间件
const logger: Middleware<any, any> = (store) => (next) => (action) => {
  console.log('Before:', store.getState());
  console.log('Action:', action);
  next(action);
  console.log('After:', store.getState());
};

manager.use(logger);

// 获取 action creators，调用时自动 dispatch
const { user, todo } = manager.getActions();

// 更新状态，参数有完整类型提示
user.setUser({ id: 1, name: 'John Doe', email: 'john@example.com' });
user.updateName('Jane Doe');
todo.addTodo('Learn TypeScript');
todo.addTodo('Build a project');

// 订阅状态变化
manager.subscribe((state) => {
  console.log('State updated:', state);
});

// 获取特定模块状态，返回对应类型
const userState = manager.getModuleState('user');
console.log('User:', userState);

// 也可以直接 dispatch action
manager.dispatch({
  type: 'user/UPDATE_NAME',
  payload: 'Alice Smith'
});
```

---

## 十一、进阶技巧与最佳实践

掌握了核心知识点和实战案例后，我们来总结一些工业级的 TypeScript 最佳实践，以及常见的陷阱和避坑指南，帮助你写出更高质量的 TypeScript 代码。

### 11\.1 类型安全的最佳实践

类型安全是 TypeScript 的核心价值，遵循以下实践可以最大化 TypeScript 的收益：

1. **开启严格模式**：tsconfig 中开启 strict 全套规则，这是类型安全的基础

2. **拒绝 any，拥抱 unknown**：处理动态数据时用 unknown 代替 any，强制进行类型收窄

3. **用 as const 定义常量**：让配置对象保留最精确的字面量类型，同时变成只读

4. **品牌类型区分业务 ID**：避免不同业务的 ID 类型混淆，比如 UserId 和 ProductId 虽然都是 string，但不能互相赋值

5. **模板字面量构建类型安全的事件系统**：事件名和事件数据一一对应，避免事件名写错或者数据不匹配

6. **never 穷尽检查**：在 switch 语句的 default 分支赋值给 never 类型，新增类型时编译报错，强制处理所有情况

```typescript
// 1. 开启严格模式，在 tsconfig.json 中配置
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true
  }
}

// 2. 使用 as const 创建常量配置，保留字面量类型
const APP_CONFIG = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  retries: 3
} as const;

// 3. 使用 satisfies 进行类型检查（TS 4.9+），不丢失字面量类型
const routes = {
  home: '/',
  about: '/about',
  contact: '/contact'
} satisfies Record<string, string>;

// 4. 使用品牌类型避免类型混淆，不同业务ID不能混用
type UserId = string & { __brand: 'UserId' };
type ProductId = string & { __brand: 'ProductId' };

function createUserId(id: string): UserId {
  return id as UserId;
}

function processUser(id: UserId) {
  // 确保传入的是 UserId 类型
}

// 5. 使用模板字面量创建类型安全的事件系统
type EventMap = {
  'user:login': { userId: string; timestamp: number };
  'user:logout': { userId: string };
  'notification:send': { message: string; priority: 'low' | 'high' };
};

type EventName = keyof EventMap;
type EventData<K extends EventName> = EventMap[K];

class EventEmitter {
  private listeners: Map<EventName, Set<Function>> = new Map();

  on<K extends EventName>(event: K, handler: (data: EventData<K>) => void): void {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, new Set());
    }
    this.listeners.get(event)!.add(handler);
  }

  emit<K extends EventName>(event: K, data: EventData<K>): void {
    const callbacks = this.listeners.get(event);
    if (callbacks) {
      callbacks.forEach(cb => cb(data));
    }
  }
}

const emitter = new EventEmitter();
emitter.on('user:login', (data) => {
  console.log(`User ${data.userId} logged in at ${data.timestamp}`);
});

// 6. 使用 Readonly 和 as const 避免副作用
const readonlyArray = [1, 2, 3] as const;
const readonlyObject = { name: 'John', age: 30 } as const;

// 7. 使用 never 进行穷尽性检查，新增类型自动报错
type Animal = 'cat' | 'dog' | 'bird';

function feedAnimal(animal: Animal): void {
  switch (animal) {
    case 'cat':
      console.log('Feed cat');
      break;
    case 'dog':
      console.log('Feed dog');
      break;
    case 'bird':
      console.log('Feed bird');
      break;
    default:
      const exhaustive: never = animal; // 新增类型这里会编译报错
      throw new Error(`Unknown animal: ${animal}`);
  }
}
```

### 11\.2 性能优化技巧

TypeScript 的类型检查在大型项目中可能会变慢，遵循以下技巧可以提升类型检查性能：

1. **优先使用 interface 而非 type**：interface 有内部缓存，大量使用时性能更好

2. **避免不必要的泛型**：简单场景不需要泛型，用具体类型或者联合类型即可

3. **善用 const 断言**：减少类型推断的复杂度

4. **信任类型推断**：不要过度标注类型，让编译器自动推断，既简洁又高效

5. **能用联合类型就不用复杂泛型**：简单的联合类型比复杂的泛型约束性能好很多

```typescript
// 1. 使用接口而不是类型别名，性能更好
// ✅ 推荐
interface User {
  name: string;
  age: number;
}

// ❌ 不推荐（大量使用时影响性能）
type User = {
  name: string;
  age: number;
};

// 2. 避免不必要的泛型
// ❌ 没必要的泛型，增加复杂度
function bad<T>(value: T): T {
  return value;
}

// ✅ 简单场景直接用具体类型
function good(value: any): any {
  return value;
}

// 3. 使用 const 断言提高类型推断性能
const config = {
  name: 'app',
  version: '1.0.0'
} as const;

// 4. 使用类型推断而不是冗余标注
// ✅ 让 TS 自动推断
const numbers = [1, 2, 3];
const result = numbers.map(n => n * 2);

// ❌ 不必要的重复标注，冗余且影响性能
const numbers: number[] = [1, 2, 3];
const result: number[] = numbers.map((n: number): number => n * 2);

// 5. 使用联合类型而不是复杂泛型
// ✅ 简单清晰，性能好
type Status = 'pending' | 'success' | 'error';

// ❌ 过于复杂的泛型，性能差且没必要
type Status<T extends string> = T extends 'pending' | 'success' | 'error' ? T : never;
```

### 11\.3 常见陷阱与解决方案

TypeScript 中有很多常见的坑，提前了解可以避免踩雷：

1. **any 滥用**：这是最常见的问题，用了 any 等于白用 TypeScript，尽量用 unknown 代替

2. **过度类型断言**：断言会绕过类型检查，尽量用类型守卫代替

3. **联合类型过于宽泛**：尽量用精确的字面量联合类型，而不是宽泛的 string/number

4. **函数重载过度**：简单场景用联合类型和可选参数就够了，不要为了重载而过载

5. **类型守卫写错**：注意 typeof 返回的是字符串，不要写成 `value === 'string'` 这种低级错误

```typescript
// 1. any 的滥用是最大的坑
// ❌ 完全丢失类型安全
let data: any = fetchData();

// ✅ 使用 unknown，强制收窄后使用
let data: unknown = fetchData();
if (typeof data === 'object' && data !== null) {
  // 安全使用 data
}

// 2. 类型断言过度使用
// ❌ 错误断言会导致运行时 bug
const value = 'hello' as number;

// ✅ 使用类型守卫，安全收窄
function isNumber(value: unknown): value is number {
  return typeof value === 'number';
}

// 3. 联合类型过于宽泛
// ❌ 太宽泛，失去约束意义
type StringOrNumber = string | number;

// ✅ 用精确的字面量类型
type Status = 200 | 201 | 400 | 404 | 500;

// 4. 函数重载过度
// ❌ 过多的重载，复杂难懂
function process(value: string): string;
function process(value: number): number;
function process(value: boolean): boolean;
function process(value: any): any {
  return value;
}

// ✅ 使用联合类型更简洁
function process(value: string | number | boolean): string | number | boolean {
  return value;
}

// 5. 类型守卫写错是常见低级错误
// ❌ 错误的守卫逻辑，永远不成立
function isString(value: unknown): value is string {
  return value === 'string';
}

// ✅ 正确的 typeof 判断
function isString(value: unknown): value is string {
  return typeof value === 'string';
}
```

### 11\.4 高级设计模式

TypeScript 和经典设计模式结合可以产生非常优雅的实现，以下是几个常用的类型安全设计模式：

1. **Builder 模式**：链式构造复杂对象，每一步都有类型提示

2. **工厂模式**：通用的对象创建工厂，保留实例类型

3. **装饰器模式**：类型安全的方法装饰器

4. **观察者模式**：类型安全的发布订阅实现

5. **策略模式**：可插拔的策略实现，类型统一

```typescript
// 1. 类型安全的 Builder 模式
class UserBuilder {
  private user: Partial<User> = {};

  setName(name: string): this {
    this.user.name = name;
    return this;
  }

  setAge(age: number): this {
    this.user.age = age;
    return this;
  }

  setEmail(email: string): this {
    this.user.email = email;
    return this;
  }

  build(): User {
    if (!this.user.name || !this.user.age) {
      throw new Error('Name and age are required');
    }
    return this.user as User;
  }
}

// 2. 类型安全的工厂模式
interface Factory<T> {
  create(...args: any[]): T;
}

class ProductFactory<T> implements Factory<T> {
  constructor(private constructor: new (...args: any[]) => T) {}

  create(...args: any[]): T {
    return new this.constructor(...args);
  }
}

// 3. 类型安全的装饰器模式
function logMethod<T extends (...args: any[]) => any>(
  target: any,
  propertyKey: string,
  descriptor: TypedPropertyDescriptor<T>
): TypedPropertyDescriptor<T> {
  const original = descriptor.value!;
  descriptor.value = function(...args: any[]) {
    console.log(`Method ${propertyKey} called with args:`, args);
    return original.apply(this, args);
  } as T;
  return descriptor;
}

// 4. 类型安全的观察者模式
interface Observer<T> {
  update(data: T): void;
}

class Subject<T> {
  private observers: Set<Observer<T>> = new Set();

  attach(observer: Observer<T>): void {
    this.observers.add(observer);
  }

  detach(observer: Observer<T>): void {
    this.observers.delete(observer);
  }

  notify(data: T): void {
    this.observers.forEach(observer => observer.update(data));
  }
}

// 5. 类型安全的策略模式
interface Strategy<T> {
  execute(data: T): T;
}

class Context<T> {
  private strategy: Strategy<T>;

  constructor(strategy: Strategy<T>) {
    this.strategy = strategy;
  }

  setStrategy(strategy: Strategy<T>): void {
    this.strategy = strategy;
  }

  executeStrategy(data: T): T {
    return this.strategy.execute(data);
  }
}
```

---

## 总结

### 核心要点回顾

1. **基础类型系统**

    - 掌握原始类型、数组、元组、对象类型的基本用法

    - 重点理解 any、unknown、never、void 的区别和适用场景，这是面试高频考点

2. **类型进阶**

    - 联合类型表示「或」的语义，交叉类型表示「且」的语义

    - 字面量类型配合联合类型可以实现轻量的状态约束

    - 模板字面量类型是强大的字符串类型运算工具

3. **类型安全**

    - 类型断言是编译期的类型覆盖，没有运行时校验，需谨慎使用

    - 类型守卫是更安全的类型收窄方式，优先使用

    - 可辨识联合是处理复杂联合类型的最佳实践，工业级项目广泛使用

4. **泛型编程**

    - 泛型是类型复用的核心，解决了 any 丢失类型安全的问题

    - 泛型约束、默认值、条件类型让泛型更灵活

    - infer 关键字是类型推导神器，是实现高级工具类型的基础

5. **工具类型**

    - 熟练掌握 12 个核心内置工具类型的使用

    - 理解工具类型的实现原理，能够自定义工具类型

    - 工具类型本质上就是类型层面的函数，输入类型输出类型

6. **实战应用**

    - 泛型组件封装可以保留元素类型，提供精确的类型提示

    - 状态管理的类型设计是 TypeScript 集大成的场景，体现类型编程的价值

    - 好的类型设计应该让使用者无感，自动获得类型提示，不需要手动标注

### 学习建议

1. **循序渐进**：从基础类型开始，先会用再深入原理，不要一开始就钻复杂的类型体操

2. **实践为主**：在实际项目中应用，把现有项目改造成 TypeScript 是最好的学习方式

3. **阅读源码**：学习 React、Vue 等优秀库的类型定义，学习工业级的类型设计思路

4. **持续学习**：TypeScript 版本更新很快，关注新特性，比如 4\.9 的 satisfies、5\.0 的装饰器等

---

> TypeScript 不仅仅是 JavaScript 的超集，更是构建大型前端应用的基石。掌握类型系统，培养类型思维，能让你的代码更加健壮、可维护，也能让你在团队协作和面试中更具竞争力。

## 十二、TypeScript 高频 10 问

### Q1：`type` 和 `interface` 有什么区别？分别在什么场景下使用？

**核心区别：**

| 特性 | `type` | `interface` |
|------|--------|-------------|
| 声明合并 | ❌ | ✅ |
| 扩展方式 | `&` | `extends` |
| 联合/元组/映射类型 | ✅ | ❌ |
| 类实现 | ✅ | ✅ |

**使用建议：**
- 对象结构、API 响应、类契约 → `interface`
- 联合类型、元组、工具类型 → `type`
- 库开发需要用户扩展类型 → 必须用 `interface`

```typescript
// interface 支持声明合并（库开发必备）
interface User { id: number; }
interface User { name: string; } // 自动合并为 { id: number; name: string }

// type 支持联合类型
type Status = 'pending' | 'success' | 'error';
```

---

### Q2：`any`、`unknown`、`never`、`void` 的区别是什么？

| 类型 | 含义 | 能否访问属性/方法 | 典型场景 |
|------|------|------------------|----------|
| `any` | 关闭类型检查 | ✅ 任意访问 | 迁移旧代码（尽量少用） |
| `unknown` | 安全的 `any` | ❌ 必须先收窄 | API 响应、动态数据 |
| `never` | 永远不会发生 | N/A | 抛出异常、死循环、穷尽检查 |
| `void` | 无返回值 | N/A | 函数无 `return` |

```typescript
// unknown：必须先收窄才能使用
let data: unknown = fetchData();
if (typeof data === 'string') {
  console.log(data.length); // ✅ 收窄后安全访问
}

// never：穷尽检查
function exhaustiveCheck(value: never): never {
  throw new Error(`未处理的值: ${value}`);
}
```

---

### Q3：什么是泛型？`extends` 和 `infer` 分别用在什么场景？

**泛型**：允许在定义函数/接口/类时使用类型参数，实现类型的复用。

- `extends`：**泛型约束**，限制类型参数必须满足某个结构
- `infer`：**类型推导**，在条件类型中声明待推导的类型变量

```typescript
// extends：约束 T 必须有 length 属性
function logLen<T extends { length: number }>(v: T): T {
  console.log(v.length);
  return v;
}

// infer：提取函数返回值类型
type Return<T> = T extends (...args: any[]) => infer R ? R : never;
```

---

### Q4：如何实现 `DeepReadonly` 和 `DeepPartial`？

利用**递归泛型**和**映射类型**，逐层处理嵌套对象。

```typescript
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object
    ? T[P] extends Function
      ? T[P]
      : DeepReadonly<T[P]>
    : T[P];
};

type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object
    ? T[P] extends Function
      ? T[P]
      : DeepPartial<T[P]>
    : T[P];
};
```

---

### Q5：`keyof` 和 `typeof` 分别用在什么场景？

- `keyof T`：获取类型 `T` 的所有键名组成的联合类型
- `typeof x`：获取变量 `x` 的类型（JS 值 → TS 类型）

```typescript
const user = { name: 'John', age: 30 };
type UserType = typeof user; // { name: string; age: number }
type UserKeys = keyof UserType; // 'name' | 'age'

// 配合使用：类型安全的属性访问
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

---

### Q6：什么是可辨识联合（Discriminated Union）？它解决了什么问题？

通过一个共有的**字面量属性**来区分联合类型中的不同成员，配合 `switch` 实现类型安全的穷尽检查。

```typescript
type Shape =
  | { kind: 'circle'; radius: number }
  | { kind: 'square'; side: number }
  | { kind: 'rectangle'; width: number; height: number };

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case 'circle': return Math.PI * shape.radius ** 2;
    case 'square': return shape.side ** 2;
    case 'rectangle': return shape.width * shape.height;
    default: const _: never = shape; return _; // 穷尽检查
  }
}
// 新增类型时，default 分支会报错，强制处理
```

---

### Q7：如何解决 `Object.keys(obj)` 返回 `string[]` 的类型问题？

TS 默认 `Object.keys` 返回 `string[]`，因为对象的键在运行时可能包含额外属性。解决方案是自定义类型断言函数。

```typescript
// 方案：自定义函数，强制返回 (keyof T)[]
function getKeys<T extends object>(obj: T): (keyof T)[] {
  return Object.keys(obj) as (keyof T)[];
}

const user = { name: 'John', age: 30 };
const keys = getKeys(user); // ('name' | 'age')[]
keys.forEach(key => {
  console.log(user[key]); // ✅ 类型安全
});
```

---

### Q8：什么是装饰器？如何定义一个类装饰器？

装饰器是 TypeScript 5.0 正式支持的元编程特性，用于在声明阶段附加元数据或修改行为。

```typescript
// 类装饰器：记录类创建日志
function Log(target: Function): void {
  console.log(`Class ${target.name} created`);
}

@Log
class User {
  constructor(public name: string) {}
}

// 方法装饰器：记录方法执行时间
function Timing(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const original = descriptor.value;
  descriptor.value = function(...args: any[]) {
    const start = performance.now();
    const result = original.apply(this, args);
    console.log(`${propertyKey} took ${performance.now() - start}ms`);
    return result;
  };
}
```

---

### Q9：`.d.ts` 声明文件是干什么的？`declare` 关键字的作用是什么？

- `.d.ts` 文件只包含**类型声明**，不包含实现，用于为 JavaScript 库提供 TypeScript 类型支持
- `declare` 用于在类型声明中描述已存在的 JavaScript 代码（全局变量、模块、函数等）

```typescript
// declarations.d.ts
declare module 'my-library' {
  export function greet(name: string): string;
  export const version: string;
}

// 描述全局变量
declare const __DEV__: boolean;

// 描述全局函数
declare function toast(message: string, duration?: number): void;
```

---

### Q10：`tsconfig.json` 中 `strict` 模式下包含了哪些选项？

`strict: true` 一次性开启以下 7 项严格检查：

| 选项 | 作用 |
|------|------|
| `noImplicitAny` | 禁止隐式 `any`（变量/参数类型必须显式或可推断） |
| `strictNullChecks` | 严格检查 `null` / `undefined`（不能赋值给其他类型） |
| `strictFunctionTypes` | 严格检查函数参数类型（启用逆变检查） |
| `strictPropertyInitialization` | 类属性必须在构造函数中初始化 |
| `noImplicitThis` | 禁止隐式 `this`（`this` 必须有明确上下文） |
| `alwaysStrict` | 每个文件以 `"use strict"` 模式解析 |
| `useUnknownInCatchVariables` | `catch` 变量类型为 `unknown` 而非 `any` |

```json
{
  "compilerOptions": {
    "strict": true
  }
}
```

---

### 面试应答技巧

1. **遇到不会的题，不要直接说"不知道"** → 说出你知道的相关知识点，比如"我不太确定 `infer` 的具体语法，但我用过条件类型和泛型约束，原理上是……"

2. **展示思路比背诵答案更重要** → 面试官想看到的是你如何思考，而不是你背了多少 API

3. **主动写代码演示** → TypeScript 面试中，能现场写出正确类型定义的印象分会大幅提升

4. **概念 + 示例 + 场景** → 每个问题用"一句话概念 + 一段代码 + 一句话场景"的结构回答，清晰又高效
