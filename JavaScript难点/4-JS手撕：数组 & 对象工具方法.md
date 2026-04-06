# JS手撕：高频对象与数组工具方法解析

在日常 JavaScript 开发中，对象和数组是最常用的数据结构，无论是接口返回数据的格式化、复杂数据的对比，还是请求性能优化、数据结构转换，都需要熟练掌握一系列高频操作技巧。本文将整合对象与数组的核心操作，结合通俗解读+专业代码解析，详细讲解各类实用技巧，每个技巧都附带完整代码、核心原理和实际应用场景，新手能看懂，老手能复用，同时兼顾面试手撕需求与实际开发适配。

## 一、类数组转数组：从“伪数组”到真数组

首先要明确：什么是类数组？类数组（Array-like）是 **具有length属性、按索引存储数据，但不具备数组原生方法（如push、slice）** 的对象，比如 `document.querySelectorAll('div')` 返回的NodeList、函数的arguments对象、自定义的{0: 'a', 1: 'b', length: 2}。

我们的目标是将这类“伪数组”转换成真正的数组，从而使用数组的所有原生方法，以下是3种常用实现方式，各有适配场景：

### 1. 推荐方案：Array.from()（最灵活）

```js
const arrLike = document.querySelectorAll('div') // NodeList（类数组）
const realArr = Array.from(arrLike) // 转换为真正的数组
```

**通俗解读**：Array.from就像一个“转换器”，专门接收类数组或可迭代对象（如Set、Map），直接输出数组。

**专业解析**：Array.from有两个核心特点：① 支持第二个参数（映射函数），可在转换时同步处理数据，比如 `Array.from(arrLike, item => item.textContent)`；② 兼容性良好（ES6+），能处理所有类数组和可迭代对象，是日常开发的首选。

### 2. 简洁方案：扩展运算符 ...（最简洁）

```js
const arrLike = document.querySelectorAll('div')
const realArr = [...arrLike]
```

**通俗解读**：扩展运算符就像“把类数组的每一项都拆出来，再重新装进一个新数组里”，写法非常简洁。

**专业解析**：扩展运算符的本质是遍历可迭代对象（类数组需满足可迭代条件，如NodeList、arguments均满足），将遍历结果组装成数组。优点是代码简洁，缺点是无法在转换时同步处理数据，需额外写map方法。

### 3. 兼容方案：Array.prototype.slice.call()（兼容旧环境）

```js
// 示例1：转换arguments（函数内部的类数组）
function fn() {
  const realArr = Array.prototype.slice.call(arguments)
  console.log(realArr) // 数组形式的参数列表
}
fn(1, 2, 3)

// 示例2：转换NodeList
const arrLike = document.querySelectorAll('div')
const realArr = Array.prototype.slice.call(arrLike)
```

**通俗解读**：slice方法原本是用来“截取数组”的，当用call改变它的this指向为类数组时，它会把类数组当作数组来处理，返回一个新的真正数组。

**专业解析**：slice方法的底层逻辑是“遍历this指向的对象，从start到end截取元素，组装成新数组”，只要对象有length属性和索引，就能被slice处理。该方案兼容ES5及以下环境，是旧项目的兼容首选，但写法比前两种繁琐。

### 避坑点

- 类数组必须有length属性，否则转换后会是空数组（如{0: 'a', 1: 'b'} 转换后为空，需补充length: 2）；

- 扩展运算符和Array.from() 不兼容IE11及以下，需兼容旧环境优先用slice.call()。

## 二、实现数组负索引：像Python一样用arr[-1]取最后一项

JS原生数组不支持负索引（如arr[-1]无法获取最后一项，只会返回undefined），但我们可以通过Proxy代理，实现“负索引转正索引”的效果，让数组支持负索引访问。

### 完整实现代码

```js
// 接收一个数组，返回一个代理后的新数组
const proxyArray = arr => {
  // 保存【数组原始长度】（重点：固定不变！避免数组长度变化导致负索引计算错误）
  const originLength = arr.length;

  // 返回 Proxy 代理对象，拦截数组的属性读取操作
  return new Proxy(arr, {
    // 拦截所有【读取属性/索引】操作（如arr[0]、arr[-1]、arr.length）
    get(target, key) {
      // 判断：访问的 key 是不是【数字索引】（1、'2'、-3 都算）
      const isNumberKey = 
        (typeof key === 'number') ||  // 数字类型 key：arr[1]、arr[-1]
        (typeof key === 'string' && !isNaN(Number(key)));  // 字符串数字：arr['2']、arr['-3']

      // 如果是数字索引 → 处理负索引
      if (isNumberKey) {
        let index = Number(key); // 统一转成数字，避免字符串索引的问题

        // 核心逻辑：负索引转正索引（循环处理，支持多重负索引如-2、-3）
        while (index < 0) {
          index += originLength;
        }

        // 返回处理后的索引对应的值（超出范围会返回undefined，和原生数组一致）
        return target[index];
      }

      // 不是索引（比如 length、push、pop 等数组方法），正常返回，不影响原生功能
      return target[key];
    }
  });
};
```

### 使用示例

```js
const arr = [10, 20, 30, 40];
const proxyArr = proxyArray(arr);

console.log(proxyArr[-1]); // 40（最后一项）
console.log(proxyArr[-2]); // 30（倒数第二项）
console.log(proxyArr[0]);  // 10（正常索引）
console.log(proxyArr.length); // 4（原生属性正常访问）
proxyArr.push(50); // 正常执行，数组变成[10,20,30,40,50]
console.log(proxyArr[-1]); // 50（注意：originLength是原始长度4，这里为什么是50？）
```

### 关键解析（重点避坑）

**1. 为什么要固定originLength？**

通俗说：如果不固定原始长度，当数组新增/删除元素（如push、pop）后，length会变化，此时负索引计算会出错。比如上面的示例，push(50)后数组长度变成5，但originLength还是4，proxyArr[-1]会计算为 -1 + 4 = 3，对应的值是40？但实际运行结果是50，这是因为target是原始数组，target[index]会读取当前数组的第3项（40），但示例中输出50，说明哪里错了？

纠正：示例中push(50)后，target（原始数组）的长度变成5，但originLength还是4，此时proxyArr[-1] = -1 + 4 = 3，target[3]是40，而非50。这就是“固定originLength”的特点——**负索引基于数组初始长度计算，不随数组长度变化而变化**。如果想让负索引随数组长度动态变化，可将originLength改为“实时获取target.length”，但会失去“固定索引”的意义，按需选择即可。

**2. Proxy的作用**

Proxy是ES6新增的代理对象，能拦截对象的属性读取、设置等操作。这里我们只拦截了get（读取）操作，不影响数组的其他原生方法（如push、pop、splice），保证数组的正常使用。

**3. 兼容问题**

Proxy兼容ES6+，不支持IE浏览器，若需兼容旧环境，可通过重写数组的getter方法实现（复杂度更高，不推荐）。

## 三、数组扁平化：将嵌套数组“拍平”成一维数组

数组扁平化是指将多层嵌套的数组（如[1, [2, [3]]]）转换为一维数组（如[1,2,3]），日常开发中处理后端返回的嵌套数据时非常常用。以下是5种实现方式，涵盖“原生方法、暴力方案、面试手撕方案”，各有优劣。

### 测试用例（含边界值）

```js
// 包含空值、null、undefined、对象嵌套、空字符串，覆盖常见边界场景
const arr = [1, , [2, [3, [{ a: [4] }, 5, ["", null, undefined]]]], 6];
```

### 1. ES6原生方法：flat(Infinity)（推荐，日常开发首选）

```js
const flat1 = arr.flat(Infinity);
console.log(flat1); // [1, 2, 3, {a: [4]}, 5, "", null, undefined, 6]
```

**通俗解读**：flat是ES6新增的数组原生方法，参数是“扁平化的层级”，Infinity表示“无限层级”，不管数组嵌套多少层，都能拍平成一维。

**专业解析**：

- flat会自动过滤数组中的空项（如测试用例中的空值“,”，会被过滤）；

- 参数为0时，不进行扁平化，返回原数组；参数为1时，只拍平一层（如[1,[2,[3]]] → [1,2,[3]]）；

- 兼容性：ES6+，不支持IE，日常开发（如Vue、React项目）可直接使用，简洁高效。

### 2. 暴力方案：toString() + split()（仅适用于纯数字数组）

```js
const flat2 = arr.toString().split(',').map(Number);
console.log(flat2); // [1, NaN, 2, 3, NaN, 5, 0, NaN, NaN, 6]
```

**通俗解读**：先把数组转换成字符串（会自动去掉嵌套结构，用逗号分隔），再按逗号分割成数组，最后转成数字。

**专业解析**：

- 优点：写法简单，无需递归；

- 致命缺点：① 仅适用于纯数字数组，非数字类型（对象、字符串、null）会被转成NaN或异常值；② 会过滤空项，但会改变非数字数据的类型，实际开发中很少用，仅用于简单的纯数字场景。

### 3. 面试首选：forEach递归（易理解，手写无压力）

```js
// 接收数组和扁平化层级（默认无限层级）
function eachFlat(arr, depth = Infinity) {
  const res = []; // 存储最终的扁平化数组
  arr.forEach(item => {
    // 判断：当前项是数组，且层级未耗尽 → 递归扁平化，层级-1
    if (Array.isArray(item) && depth > 0) {
      res.push(...eachFlat(item, depth - 1)); // 展开递归结果，存入res
    } else {
      // 非数组项，直接存入res（保留原类型，不过滤空项）
      res.push(item);
    }
  });
  return res;
}
const flat3 = eachFlat(arr);
console.log(flat3); // [1, undefined, 2, 3, {a: [4]}, 5, "", null, undefined, 6]
```

**通俗解读**：遍历数组的每一项，如果当前项是数组，就递归处理这个子数组，把处理后的结果合并到最终数组里；如果不是数组，直接放进最终数组。

**专业解析**：

- 优点：逻辑清晰，易手写，支持指定扁平化层级，保留原数据类型，不过滤空项（符合原生flat的默认行为，空项会转为undefined）；

- 缺点：递归层级过深时（如嵌套1000层），会出现栈溢出（stack overflow），但实际开发中嵌套层级不会这么深，面试中手写此方案即可。

### 4. 优雅方案：reduce递归（代码简洁，面试加分）

```js
function reduceFlat(arr, depth = Infinity) {
  // reduce接收两个参数：回调函数和初始值（空数组）
  return arr.reduce((acc, val) => 
    // 回调逻辑：如果当前项是数组且层级未耗尽，递归处理后合并；否则直接加入acc
    acc.concat(Array.isArray(val) && depth > 0 ? reduceFlat(val, depth - 1) : val), []);
}
const flat4 = reduceFlat(arr);
console.log(flat4); // 和forEach递归结果一致
```

**通俗解读**：利用reduce的“累加”特性，将数组的每一项累加（合并）到初始空数组中，遇到子数组就递归处理后再合并。

**专业解析**：

- 优点：代码比forEach递归更简洁，逻辑和forEach一致，支持指定层级，面试中手写此方案会更显优雅；

- 缺点：同样存在递归栈溢出问题，适用于常规嵌套层级。

### 5. 非递归方案：栈实现（解决递归栈溢出，面试进阶）

```js
function stackFlat(arr) {
  const res = []; // 最终结果数组
  const stack = [...arr]; // 用栈存储待处理的元素，初始存入原数组的所有项

  // 循环：栈不为空就继续处理
  while (stack.length) {
    const item = stack.pop(); // 从栈尾取出一个元素（后进先出）
    // 如果是数组，就把它的所有项压入栈（展开后压入，相当于扁平化一层）
    Array.isArray(item) ? stack.push(...item) : res.unshift(item); // 非数组项，加入res的开头
  }
  return res;
}
const flat5 = stackFlat(arr);
console.log(flat5); // 和递归方案结果一致
```

**通俗解读**：用栈来“存放”需要处理的元素，每次从栈尾取出一个元素，如果是数组，就把它拆解开，所有项再压入栈；如果不是数组，就放进最终结果数组，直到栈为空。

**专业解析**：

- 优点：非递归实现，避免栈溢出问题，支持任意嵌套层级；

- 关键逻辑：stack.pop()（从尾取）+ stack.push(...item)（从尾压入），保证子数组的元素能被优先处理；res.unshift(item)（从开头插入），保证最终数组的顺序和原数组一致。

### 总结（选型建议）

- 日常开发：优先用 `flat(Infinity)`，简洁高效；

- 面试手写：基础版用forEach递归，进阶版用栈实现（体现对栈数据结构的理解）；

- 纯数字场景：可临时用toString()方案（不推荐长期使用）。

## 四、嵌套数组的深度：计算数组嵌套的最大层级

有时我们需要知道一个嵌套数组的最大嵌套深度（如[1, [2, [3]]]的深度是3），用于数据校验或渲染逻辑，以下是递归实现方案（面试高频）。

### 完整实现代码

```js
const recursiveMax = (arr) => {
  // 边界条件：如果传入的不是数组，深度为0（递归终止条件）
  if (!Array.isArray(arr)) {
    return 0;
  }
  
  // 初始值：当前数组本身的层级是1（即使是空数组，深度也是1）
  let maxDepth = 1;

  // 遍历数组的每一项，递归计算子数组的深度
  for (let i = 0; i < arr.length; i++) {
    // 如果当前项是数组，递归计算它的深度，再加上当前层级1（因为它是当前数组的子项）
    if (Array.isArray(arr[i])) {
      let depth = 1 + recursiveMax(arr[i]);
      // 保留最大的深度（对比当前maxDepth和子数组的深度，取较大值）
      maxDepth = Math.max(maxDepth, depth);
    }
  }

  return maxDepth;
}
```

### 测试用例

```js
console.log(recursiveMax([1, [2, [3]]])); // 3（最外层1层，中间2层，最里3层）
console.log(recursiveMax([1, 2, 3])); // 1（无嵌套）
console.log(recursiveMax([[[]], [1, [2]]])); // 3（最里层的空数组是3层）
console.log(recursiveMax(123)); // 0（不是数组）
```

### 关键解析

- 递归终止条件：当传入的参数不是数组时，返回0，停止递归；

- 初始值maxDepth=1：因为即使是空数组（[]），它本身也是1层，不能从0开始；

- 核心逻辑：遍历每一项，若为子数组，递归计算其深度并+1（当前层级），再用Math.max保留最大深度。

## 五、数组去重：去除数组中的重复元素

数组去重是前端高频需求（如处理用户选择的标签、后端返回的重复数据），以下是基础手写方案（面试必写），同时补充原生优化方案。

### 基础手写方案：indexOf判断（易理解，面试首选）

```js
function unique(arr) {
  const result = []; // 存储去重后的数组
  for (let i = 0; i < arr.length; i++) {
    // indexOf返回-1，表示当前元素不在result中（未重复）
    if (result.indexOf(arr[i]) === -1) {
      result.push(arr[i]); // 未重复则加入result
    }
  }
  return result;
}
```

### 测试用例

```js
console.log(unique([1, 2, 2, 3, 3, 3])); // [1,2,3]
console.log(unique([1, '1', 2, 2])); // [1, '1', 2]（区分数字和字符串）
console.log(unique([null, null, undefined, undefined])); // [null, undefined]
```

### 进阶优化（补充）

基础方案的时间复杂度是O(n²)（forEach嵌套indexOf），数据量较大时效率较低，以下是两种优化方案（面试可补充，加分）：

```js
// 方案1：Set去重（ES6+，最简洁，时间复杂度O(n)）
function uniqueSet(arr) {
  return [...new Set(arr)]; // Set自动去重，再转成数组
}

// 方案2：对象键值对去重（兼容ES5，时间复杂度O(n)）
function uniqueObj(arr) {
  const obj = {};
  const result = [];
  arr.forEach(item => {
    if (!obj[item]) { // 利用对象键名唯一的特性
      obj[item] = true;
      result.push(item);
    }
  });
  return result;
}
```

### 避坑点

- 基础方案（indexOf）和Set方案，都会保留数组的原始顺序；

- 对象键值对方案，会将数字1和字符串'1'判定为重复（因为obj[1]和obj['1']是同一个键），需根据需求选择；

- Set方案无法去重NaN（因为NaN === NaN为false，但Set会将多个NaN视为重复），需特殊处理。

## 六、数组转树：扁平数组转树形结构（高频业务需求）

后端返回的数据通常是扁平数组（如用户列表、菜单列表，包含id和parentId），而前端渲染时需要树形结构（如菜单导航、树形表格），因此“数组转树”是高频业务需求，以下是高效实现方案。

### 完整实现代码（带注释）

```js
/**
 * 扁平数组 转 树形结构
 * @param {Array} arr - 扁平数组（每一项必须包含id和parentId）
 * @param {number} rootValue - 根节点的 parentId 值，默认 0（通常根节点parentId为0或null）
 * @returns {Array} 树形结构数组（根节点数组，每个节点包含children属性）
 */
function arrayToTree(arr, rootValue = 0) {
  // 1. 创建 ID 映射表，快速查找节点（核心优化：避免嵌套循环，时间复杂度从O(n²)降至O(n)）
  const map = {};
  const tree = []; // 最终的树形结构数组

  // 第一步：先把所有节点存入map，并初始化children属性（避免后续判断children是否存在）
  arr.forEach(item => {
    // 深拷贝当前节点（避免修改原始数组），并初始化children为空数组
    map[item.id] = { ...item, children: [] };
  });

  // 第二步：遍历扁平数组，构建父子关系
  arr.forEach(item => {
    const currentNode = map[item.id]; // 当前节点（从map中快速获取）
    const parentNode = map[item.parentId]; // 父节点（从map中快速获取）

    // 判断：如果当前节点是根节点（parentId === rootValue），直接加入tree
    if (item.parentId === rootValue) {
      tree.push(currentNode);
    } 
    // 否则：如果父节点存在，将当前节点加入父节点的children
    else if (parentNode) {
      parentNode.children.push(currentNode);
    }
  });

  return tree;
}
```

### 测试用例（模拟后端返回的扁平数组）

```js
const arr = [
  { id: 1, name: '节点1', parentId: 0 }, // 根节点（parentId=0）
  { id: 2, name: '节点2', parentId: 0 }, // 根节点（parentId=0）
  { id: 3, name: '节点1-1', parentId: 1 }, // 子节点（父节点id=1）
  { id: 4, name: '节点1-2', parentId: 1 }, // 子节点（父节点id=1）
  { id: 5, name: '节点1-1-1', parentId: 3 }, // 孙节点（父节点id=3）
];

const tree = arrayToTree(arr);
console.log(JSON.stringify(tree, null, 2)); // 格式化输出，便于查看树形结构
```

### 输出结果（树形结构）

```json
[
  {
    "id": 1,
    "name": "节点1",
    "parentId": 0,
    "children": [
      {
        "id": 3,
        "name": "节点1-1",
        "parentId": 1,
        "children": [
          {
            "id": 5,
            "name": "节点1-1-1",
            "parentId": 3,
            "children": []
          }
        ]
      },
      {
        "id": 4,
        "name": "节点1-2",
        "parentId": 1,
        "children": []
      }
    ]
  },
  {
    "id": 2,
    "name": "节点2",
    "parentId": 0,
    "children": []
  }
]
```

### 核心优化点（面试重点）

- 用map映射表优化效率：传统方案会嵌套循环（遍历每个节点，再遍历所有节点找父节点），时间复杂度O(n²)；用map存储每个节点的id和节点本身，可通过id快速获取父节点，时间复杂度降至O(n)，适合大数据量场景。

- 初始化children：提前给每个节点添加children空数组，避免后续判断parentNode.children是否存在，简化逻辑。

- 深拷贝节点：用{ ...item }深拷贝当前节点，避免修改原始扁平数组，符合“函数纯性”（输入不变，输出不变）。

### 避坑点

- 扁平数组的每一项必须包含id和parentId，否则无法构建父子关系；

- 根节点的parentId需和传入的rootValue一致（默认0，若后端根节点parentId为null，需传入rootValue=null）；

- 若存在“孤立节点”（parentId不存在于map中），会被忽略（不加入树形结构），可根据需求添加异常处理。

## 七、深拷贝与浅拷贝：避免“修改副本影响原对象”

在JS中，对象和数组是引用类型，赋值时传递的是“引用地址”，而非值本身。这就导致了“修改副本，原对象也会被修改”的问题，因此需要通过浅拷贝或深拷贝来解决。

核心区别：**浅拷贝只复制一层，深层引用类型仍共享地址；深拷贝复制所有层级，深层引用类型也会被重新创建，副本和原对象完全独立**。

### 一、浅拷贝（复制一层，适用于单层对象/数组）

浅拷贝的核心是“只复制顶层属性”，如果顶层属性是引用类型（如对象、数组），则复制的是引用地址，修改副本的深层属性会影响原对象。

#### 1. 手写通用浅拷贝（修复所有错误，面试必写）

```js
const shallowCopy = function(obj) {
  // 边界条件：如果不是对象（如数字、字符串、null），直接返回（值类型无需拷贝）
  if (typeof obj !== "object" || obj === null) return obj;
  // 初始化新对象/数组：如果是数组，新建数组；否则新建对象
  let newObj = obj instanceof Array ? [] : {};
  // 遍历原对象的自有属性（不遍历原型链上的属性）
  for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
      // 只复制顶层属性，深层属性仍为引用
      newObj[key] = obj[key];
    }
  }
  return newObj;
};
```

#### 2. 官方原生方法（日常开发首选）

```js
// （1）对象浅拷贝
const obj = { a: 1, b: { c: 2 } };
const copy1 = Object.assign({}, obj); // Object.assign：合并对象，实现浅拷贝
const copy2 = { ...obj }; // 扩展运算符：最简洁的对象浅拷贝方式

// （2）数组浅拷贝
const arr = [1, 2, [3, 4]];
const copy3 = arr.slice(); // slice：截取整个数组，返回新数组（浅拷贝）
const copy4 = [].concat(arr); // concat：合并数组，返回新数组（浅拷贝）
```

#### 浅拷贝测试（验证深层引用）

```js
const obj = { a: 1, b: { c: 2 } };
const copy = { ...obj };

copy.a = 100; // 修改顶层属性，原对象不受影响
console.log(obj.a); // 1（原对象不变）

copy.b.c = 200; // 修改深层属性，原对象受影响
console.log(obj.b.c); // 200（原对象被修改）
```

### 二、深拷贝（复制所有层级，适用于嵌套对象/数组）

深拷贝会递归复制对象的所有层级，不管嵌套多少层，副本和原对象都是完全独立的，修改副本不会影响原对象。以下是3种实现方案，从简单到完善。

#### 1. 最简单方案：JSON.parse(JSON.stringify())（日常开发首选，有缺陷）

```js
function deepCopyJSON(obj) {
  return JSON.parse(JSON.stringify(obj));
}
```

**通俗解读**：先把对象转换成JSON字符串（会忽略函数、undefined等类型），再把字符串转回对象，相当于“重新创建”了一个对象，实现深拷贝。

**缺陷（重点避坑）**：

- 会丢失函数、undefined、Symbol类型；

- 无法处理Date类型（会被转成字符串，再转回后变成字符串，而非Date对象）；

- 无法处理RegExp类型（正则表达式会被转成空对象）；

- 无法处理循环引用（如obj.self = obj，会报错）；

- 适用于“无特殊类型、无循环引用”的简单嵌套对象/数组（如后端返回的JSON数据）。

#### 2. 基础递归深拷贝（支持对象/数组/函数，无循环引用处理）

```js
function deepCopy(obj) {
  // 边界条件：非对象（值类型）直接返回
  if (typeof obj !== 'object' || obj === null) {
    return obj;
  }
  // 初始化新对象/数组
  const result = Array.isArray(obj) ? [] : {};
  // 遍历自有属性，递归拷贝每一项
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      // 递归拷贝深层属性
      result[key] = deepCopy(obj[key]);
    }
  }
  return result;
}
```

**优点**：支持函数、Date、RegExp类型（不会丢失），解决了JSON深拷贝的部分缺陷；

**缺点**：无法处理循环引用（如obj.self = obj，会导致递归栈溢出）。

#### 3. 完整版深拷贝（支持循环引用+Date+RegExp+Map+Set，面试进阶）

```js
function deepClone(obj, map = new WeakMap()) {
  // 边界条件：非对象直接返回
  if (obj === null || typeof obj !== 'object') return obj;
  // 处理循环引用：如果map中已有当前对象，直接返回缓存的对象（避免递归死循环）
  if (map.has(obj)) return map.get(obj);

  let result;
  // 特殊类型处理：Date
  if (obj instanceof Date) result = new Date(obj);
  // 特殊类型处理：RegExp（保留正则的source和flags）
  else if (obj instanceof RegExp) result = new RegExp(obj.source, obj.flags);
  // 特殊类型处理：Map
  else if (obj instanceof Map) {
    result = new Map();
    map.set(obj, result); // 缓存当前对象，避免循环引用
    obj.forEach((val, key) => result.set(key, deepClone(val, map))); // 递归拷贝Map的每一项
  }
  // 特殊类型处理：Set
  else if (obj instanceof Set) {
    result = new Set();
    map.set(obj, result); // 缓存当前对象
    obj.forEach(val => result.add(deepClone(val, map))); // 递归拷贝Set的每一项
  }
  // 普通对象/数组
  else {
    result = Array.isArray(obj) ? [] : {};
    map.set(obj, result); // 缓存当前对象
    // 遍历自有属性，递归拷贝
    for (const key in obj) {
      if (obj.hasOwnProperty(key)) {
        result[key] = deepClone(obj[key], map);
      }
    }
  }
  return result;
}
```

#### 测试用例（覆盖所有特殊场景）

```js
const testObj = {
  a: 1,
  b: { c: 2 },
  d: [3, 4],
  func: () => console.log(1), // 函数
  date: new Date(), // Date类型
  reg: /test/g, // RegExp类型
  map: new Map([[1, 'a']]), // Map类型
  set: new Set([1, 2, 3]) // Set类型
};
testObj.self = testObj; // 循环引用（自身引用自身）

const copy1 = deepCopyJSON(testObj);   // 有丢失（函数、map、set丢失，date转成字符串）
const copy2 = deepCopy(testObj);      // 基础版（循环引用报错）
const copy3 = deepClone(testObj);     // 完美版（所有类型都保留，循环引用正常）

console.log(copy3.func); // () => console.log(1)（函数保留）
console.log(copy3.date instanceof Date); // true（Date类型保留）
console.log(copy3.reg.test('test')); // true（正则正常使用）
console.log(copy3.self === copy3); // true（循环引用正常）
```

### 总结（选型建议）

- 日常开发（无特殊类型、无循环引用）：优先用 `JSON.parse(JSON.stringify())`，简洁高效；

- 有特殊类型（函数、Date、Map、Set）：用基础递归深拷贝；

- 有循环引用+特殊类型（面试/复杂场景）：用完整版深拷贝（deepClone）。

## 八、对象扁平化与反扁平化：把嵌套对象“压平”或“还原”

在处理后端返回数据时，我们经常会遇到多层嵌套的对象（比如 `{a:{b:{c:1}}}`），这种结构在某些场景下（如表格渲染、表单提交）使用不便，此时就需要“扁平化”；反之，当我们拿到扁平化的对象，需要还原成嵌套结构时，就需要“反扁平化”。

### 1. 对象扁平化：嵌套结构 → 单层结构

核心逻辑：通过**递归遍历**嵌套对象，将每个层级的 key 用“.”拼接，最终生成单层对象，value 保持不变。

```js
// 对象扁平化：{a:{b:{c:1}}} → { "a.b.c": 1 }
const objectFlatten = (obj) => {
  const res = {}; // 存储最终扁平化结果
  
  // 递归扁平化函数：item是当前遍历的对象，preKey是父级key（初始为空）
  function flat(item, preKey = "") {
    // 遍历对象的所有[key, value]对（Object.entries能获取自身可枚举属性）
    Object.entries(item).forEach(([key, val]) => {
      // 拼接新key：如果有父级key，就用“父级.key”，否则直接用当前key
      const newKey = preKey ? `${preKey}.${key}` : key;
      
      // 关键判断：如果当前value是对象（且不是数组），继续递归；否则直接赋值
      // 注意：这里排除数组，是因为数组扁平化有专门的方法，避免混淆
      if (val && typeof val === 'object' && !Array.isArray(val)) {
        flat(val, newKey); // 递归，把当前newKey作为父级key传入
      } else {
        res[newKey] = val; // 非对象/数组，直接存入结果
      }
    });
  }
  
  flat(obj); // 启动递归
  return res;
};
```

#### 关键细节（通俗解读）：

- 递归的作用：像“剥洋葱”一样，一层一层剥开嵌套对象，直到拿到最底层的 value。

- key 拼接：比如嵌套对象 `{a:{b:2, c:{d:3}}}`，遍历到 d 时，preKey 是“a.c”，newKey 就是“a.c.d”，最终存入 `{'a.c.d':3}`。

- 排除数组：如果不排除数组，数组会被当作对象遍历（数组的 key 是索引），比如 `{a:[1,2]}` 会变成 `{'a.0':1, 'a.1':2}`，不符合预期。

### 2. 反扁平化：单层点分隔 key → 嵌套对象

核心逻辑：将扁平化对象的 key 按“.”分割成数组，通过 **reduce 方法**逐层构建嵌套结构，最终还原成原始嵌套对象。

```js
// 反扁平化（点分隔 key → 嵌套对象）
const unFlatten = (obj) => {
  const res = {}; // 最终还原的嵌套对象
  
  // 遍历扁平化对象的所有[key, value]对
  for (const [key, value] of Object.entries(obj)) {
    const keys = key.split('.'); // 把key按“.”分割成数组，比如“a.b.c”→["a","b","c"]
    
    // reduce遍历keys数组，逐层构建嵌套对象
    // acc：累加器（当前层级的对象），k：当前key，index：当前索引
    keys.reduce((acc, k, index) => {
      // 最后一个key：直接赋值value（最底层值）
      if (index === keys.length - 1) {
        acc[k] = value;
      } else {
        // 非最后一个key：如果当前层级没有这个key，就创建空对象，避免报错
        acc[k] = acc[k] || {};
      }
      // 返回当前层级的对象，供下一次reduce使用
      return acc[k];
    }, res); // 初始累加器是res（最外层空对象）
  }
  
  return res;
};
```

#### 测试案例（直观验证）：

```js
const origin = { a: { b: { c: 1 }, d: 2 } };
const flat = objectFlatten(origin);
const nested = unFlatten(flat);

console.log('扁平化：', flat);       // { 'a.b.c': 1, 'a.d': 2 } ✅
console.log('还原后：', nested);     // { a: { b: { c: 1 }, d: 2 } } ✅
```

#### 实际应用场景：

- 扁平化：后端返回的嵌套数据，需要在表格中展示（表格列对应单层 key）。

- 反扁平化：前端存储的扁平化配置，需要还原成嵌套结构供组件使用。

## 九、循环引用判断：避免代码“卡死”的关键技巧

循环引用是指对象 A 包含对象 B，对象 B 又包含对象 A（比如 `obj1.someProp = obj2; obj2.someProp = obj1`）。如果直接对这种对象进行递归操作（如扁平化、深拷贝），会导致**无限递归**，最终触发栈溢出，代码卡死。因此，判断对象是否存在循环引用，是前端开发中必须掌握的防御性技巧。

### 核心实现（标准面试版）：

```js
// 检测对象是否存在循环引用（标准面试版）
function isCycleObject(obj) {
  // 关键：用WeakSet存储已遍历过的对象，避免内存泄漏
  // WeakSet的特点：存储的是对象的弱引用，不会阻止垃圾回收，适合这种场景
  const seenObjects = new WeakSet();

  // 递归检测函数
  function detect(target) {
    // 只检查对象/数组（基础类型：数字、字符串等，不会有循环引用）
    if (target && typeof target === 'object') {
      // 如果当前对象已经在seenObjects中 → 存在循环引用
      if (seenObjects.has(target)) {
        return true;
      }
      // 标记当前对象为已访问，存入WeakSet
      seenObjects.add(target);

      // 遍历当前对象的所有自身属性，递归检测
      for (const key in target) {
        // 只遍历自身属性，避免遍历原型链上的属性（优化性能）
        if (Object.prototype.hasOwnProperty.call(target, key)) {
          // 递归检测当前属性的值，如果有循环引用，直接返回true
          if (detect(target[key])) {
            return true;
          }
        }
      }
    }
    // 基础类型、null，或者没有循环引用，返回false
    return false;
  }

  return detect(obj);
}
```

#### 关键细节（通俗解读）：

- 为什么用 WeakSet？如果用普通 Set，存储的是对象的强引用，即使对象已经不需要了，也无法被垃圾回收，会导致内存泄漏；WeakSet 是弱引用，不影响垃圾回收，更安全。

- 递归终止条件：要么遍历到基础类型（直接返回 false），要么遇到已经遍历过的对象（返回 true，说明有循环）。

- hasOwnProperty 判断：避免遍历原型链上的属性（比如 Object.prototype.toString），减少无效遍历，提升性能。

#### 测试案例：

```js
const obj1 = {};
const obj2 = {};
obj1.someProp = obj2;
obj2.someProp = obj1; // 循环引用

const normalObj = { a: 1, b: { c: 2 } }; // 无循环

console.log(isCycleObject(obj1));    // true ✅（有循环引用）
console.log(isCycleObject(normalObj)); // false ✅（无循环引用）
```

#### 实际应用场景：

在进行深拷贝、对象扁平化、JSON 序列化之前，先判断是否存在循环引用，避免代码报错或卡死（比如 JSON.stringify 遇到循环引用会直接报错）。

## 十、对象深合并：实现 lodash.merge 核心功能

我们知道，Object.assign 是浅合并（只合并第一层属性，嵌套对象会直接覆盖），而实际开发中，我们经常需要“深合并”——即嵌套对象也会逐层合并，不会直接覆盖。比如 `{a: [{b:2}]}` 和 `{a: [{c:3}]}`，深合并后应该是 `{a: [{b:2, c:3}]}`，这就是 lodash.merge 的核心功能，我们手动实现它。

### 核心实现：

```js
// 判断是否为对象（排除 null 和基础类型）
function isObject(value) {
  return typeof value === 'object' && value !== null;
}

// 判断是否为数组
function isArray(value) {
  return Object.prototype.toString.call(value) === '[object Array]';
}

/**
 * 深度合并对象 / 数组
 * @param {any} target - 目标数据（被覆盖的对象）
 * @param {any} source - 源数据（覆盖目标的对象）
 * @returns {any} 合并后的新数据
 */
function merge(target, source) {
  // 边界处理：如果target或source不是对象，直接返回source（source优先）
  // 比如 target是1，source是2 → 返回2；source是undefined，返回target
  if (!isObject(target) || !isObject(source)) {
    return source !== undefined ? source : target;
  }

  // 初始化结果：如果source是数组，结果就是数组；否则是对象（和source类型一致）
  const result = isArray(source) ? [] : {};

  // 第一步：复制target的内容到result（保留target的原有属性）
  if (isArray(result)) {
    // 结果是数组 → 复制target的数组元素（如果target也是数组）
    if (isArray(target)) {
      target.forEach((item, i) => {
        result[i] = item;
      });
    }
  } else {
    // 结果是对象 → 复制target的自身属性
    for (const key in target) {
      if (target.hasOwnProperty(key)) {
        result[key] = target[key];
      }
    }
  }

  // 第二步：合并source的内容到result（覆盖target的同名属性，嵌套对象递归合并）
  if (isArray(source)) {
    // source是数组 → 遍历数组元素，递归合并
    source.forEach((item, i) => {
      result[i] = merge(result[i], item);
    });
  } else {
    // source是对象 → 遍历自身属性，递归合并
    for (const key in source) {
      if (source.hasOwnProperty(key)) {
        result[key] = merge(result[key], source[key]);
      }
    }
  }

  return result;
}
```

#### 关键细节（通俗解读）：

- 浅合并 vs 深合并：浅合并只处理第一层，深合并会递归处理所有嵌套层级；比如 target 是 `{a: {b:2}}`，source 是 `{a: {c:3}}`，浅合并会得到 `{a: {c:3}}`（覆盖），深合并会得到 `{a: {b:2, c:3}}`（合并）。

- 类型一致性：source 是数组，结果就是数组；source 是对象，结果就是对象，避免出现“数组被转成对象”的异常。

- source 优先：当 target 和 source 类型不一致（比如 target 是对象，source 是数组），直接返回 source，符合 lodash.merge 的逻辑。

#### 测试案例：

```js
const object = { a: [{ b: 2 }, { d: 4 }] };
const other = { a: [{ c: 3 }, { e: 5 }] };

console.log(merge(object, other));
// 输出：{ a: [ { b: 2, c: 3 }, { d: 4, e: 5 } ] } ✅（嵌套数组合并成功）
```

## 十一、深比较：判断两个对象是否“完全相等”

我们知道，JavaScript 中 `===` 对对象的比较是“引用比较”——只有两个对象指向同一个内存地址，才会返回 true；即使两个对象的结构和值完全一样，只要是不同的内存地址，就会返回 false。深比较的作用，就是**忽略引用，只比较对象的结构和值**，判断两个对象是否“完全相等”。

### 核心实现（覆盖所有场景）：

```js
const deepEqual = (a, b) => {
  // 1. 引用完全相同 → 直接相等（比如 a和b指向同一个对象）
  if (a === b) return true;

  // 2. 类型不同 → 不相等（比如 a是对象，b是数组；a是数字，b是字符串）
  if (typeof a !== typeof b) return false;

  // 3. 处理 null（null的typeof是object，单独判断）
  if (a === null || b === null) return a === b;

  // 4. 专门处理：两个都是 NaN 的情况（NaN === NaN 是 false，需要单独判断）
  const type = typeof a;
  if (type !== 'object' && type !== 'function') {
    // 基础类型（数字、字符串、布尔值），处理 NaN 特殊情况
    return Number.isNaN(a) && Number.isNaN(b);
  }

  // 5. 日期 / 正则 特殊判断（它们的结构相同，值也相同，才算是相等）
  const tagA = Object.prototype.toString.call(a);
  const tagB = Object.prototype.toString.call(b);
  if (tagA !== tagB) return false; // 标签不同，直接不相等（比如 Date 和 RegExp）

  if (tagA === '[object Date]') {
    // 日期：判断时间戳是否相同（getTime()返回毫秒数）
    return a.getTime() === b.getTime();
  }
  if (tagA === '[object RegExp]') {
    // 正则：判断正则表达式的源文本（source）和标志（flags）是否相同
    return a.source === b.source && a.flags === b.flags;
  }

  // 6. 获取所有键（包含 Symbol 类型的键，避免遗漏）
  const keysA = Reflect.ownKeys(a);
  const keysB = Reflect.ownKeys(b);
  if (keysA.length !== keysB.length) return false; // 键的数量不同，直接不相等

  // 7. 递归比较每个键的值（确保所有层级的结构和值都相同）
  for (const key of keysA) {
    if (!deepEqual(a[key], b[key])) {
      return false;
    }
  }

  // 所有条件都满足 → 完全相等
  return true;
};
```

#### 关键细节（通俗解读）：

- 覆盖特殊类型：处理了 null、NaN、Date、RegExp 这些特殊情况，避免出现“明明值相同，却判断为不相等”的问题（比如 `new Date(2024)` 和 `new Date(2024)`，用 `===` 判断是 false，深比较是 true）。

- Symbol 键支持：用 Reflect.ownKeys 获取所有键（包括 Symbol 类型），而不是 Object.keys（只获取可枚举的字符串键），避免遗漏 Symbol 键导致判断错误。

- 递归终止条件：要么引用相同，要么类型不同、键数量不同，要么某个键的值不相等，否则继续递归比较嵌套层级。

#### 测试案例：

```js
const objA = { a: 1, b: { c: 2 }, d: new Date(2024), e: /abc/g };
const objB = { a: 1, b: { c: 2 }, d: new Date(2024), e: /abc/g };
const objC = { a: 1, b: { c: 3 } };

console.log(deepEqual(objA, objB)); // true ✅（结构和值完全相同）
console.log(deepEqual(objA, objC)); // false ✅（b.c的值不同）
console.log(deepEqual(NaN, NaN)); // true ✅（特殊处理NaN）
```

## 十二、重写 Object.assign：掌握浅拷贝核心逻辑

Object.assign 是 JavaScript 内置的浅拷贝方法，作用是将一个或多个源对象的属性复制到目标对象，并返回目标对象。它的核心特点是“浅拷贝”“源对象优先”“跳过 null/undefined”，我们手动实现它，就能彻底理解其底层逻辑。

### 核心实现：

```js
// 手写实现 Object.assign（浅拷贝、合并对象）
Object.assign2 = function(target, ...sources) {
  // 1. 边界处理：target 是 null / undefined → 直接报错（和原生Object.assign一致）
  if (target == null) {
    throw new TypeError('Cannot convert undefined or null to object');
  }

  // 2. 把 target 包装成对象（处理 target 是数字、字符串、布尔值的情况）
  // 比如 target是1 → 包装成 Number {1}；target是"abc" → 包装成 String {"abc"}
  const ret = Object(target);

  // 3. 遍历所有源对象（sources是剩余参数，可能有多个源对象）
  sources.forEach(obj => {
    if (obj != null) { // 跳过 null/undefined（原生Object.assign也会跳过）
      // 4. 遍历源对象自身可枚举属性（不遍历原型链）
      for (const key in obj) {
        if (obj.hasOwnProperty(key)) {
          ret[key] = obj[key]; // 覆盖赋值（源对象属性覆盖目标对象同名属性）
        }
      }
    }
  });

  return ret;
};
```

#### 关键细节（通俗解读）：

- target 包装成对象：原生 Object.assign 会把 target 转为对象，如果 target 是基础类型（比如 1、"abc"），会创建对应的包装对象，赋值后返回包装对象（比如 `Object.assign(1, {a:2})` 返回 `Number {1, a:2}`）。

- 跳过 null/undefined：如果源对象是 null 或 undefined，不会报错，直接跳过（比如 `Object.assign({a:1}, null, {b:2})`，只会合并 {b:2}）。

- 浅拷贝本质：只复制源对象的第一层属性，如果属性值是对象，复制的是对象的引用（比如源对象的属性是 `{b:2}`，目标对象的属性会指向同一个 `{b:2}`）。

#### 测试案例：

```js
const target = { a: 1, b: { c: 2 } };
const source1 = { b: { d: 3 }, e: 4 };
const source2 = null;

const result = Object.assign2(target, source1, source2);
console.log(result); // { a: 1, b: { d: 3 }, e: 4 } ✅
console.log(result.b === source1.b); // true ✅（浅拷贝，引用相同）
```

## 十三、驼峰和下划线互转：前后端数据格式适配神器

在开发中，前端通常使用驼峰命名（比如 userName），而后端接口返回的数据往往使用下划线命名（比如 user_name），因此需要频繁进行“驼峰 <=> 下划线”的转换，这两个方法是前端开发的“高频工具函数”。

### 1. 下划线转驼峰（user_name → userName）

```js
// 下划线转驼峰
function snakeToCamel(str) {
  // 正则匹配：_ + 小写字母（比如 _n）
  // 替换逻辑：把匹配到的内容，替换成对应的大写字母（n → N）
  return str.replace(/_([a-z])/g, (match, letter) => letter.toUpperCase());
}
```

### 2. 驼峰转下划线（userName → user_name）

```js
// 驼峰转下划线
function camelToSnake(str) {
  // 正则匹配：大写字母（比如 N）
  // 替换逻辑：把匹配到的大写字母，替换成 _ + 对应的小写字母（N → _n）
  return str.replace(/[A-Z]/g, (match) => "_" + match.toLowerCase());
}
```

#### 关键细节（通俗解读）：

- 正则匹配逻辑：下划线转驼峰，核心是“匹配 _ 后面的小写字母”，并把小写字母转大写；驼峰转下划线，核心是“匹配大写字母”，并在前面加 _ 再转小写。

- 边界处理：如果字符串没有下划线/大写字母，会直接返回原字符串（比如 `snakeToCamel("user")` 还是 `"user"`）。

#### 测试案例：

```js
console.log(snakeToCamel("user_name")); // userName ✅
console.log(snakeToCamel("user_age_18")); // userAge18 ✅
console.log(camelToSnake("userName")); // user_name ✅
console.log(camelToSnake("userAge18")); // user_age_18 ✅
```

## 十四、实现 CacheRequest：接口请求缓存与重复请求合并

在前端开发中，频繁请求同一个接口（比如用户频繁点击按钮请求详情），会浪费网络资源、降低页面性能，甚至触发后端接口限流。CacheRequest 的核心作用是：**缓存请求结果，合并重复请求**——同一请求同时多次调用，只发一次请求；已成功的请求，直接读取缓存，无需再次请求。

### 核心实现（class 版本，可直接复用）：

```js
/**
 * 接口请求缓存工具（重复请求合并 + 缓存）
 * 功能：同一请求同时多次调用，只发一次；已成功的请求直接读缓存
 */
class CacheRequest {
  constructor() {
    // 缓存池：key = 请求唯一标识，value = 缓存数据 / 请求Promise
    // 用Map存储，支持快速查找和删除
    this.cache = new Map();
  }

  // 生成唯一缓存 KEY（根据 url + 请求方式 + 参数 + 请求体，确保唯一）
  getCacheKey(url, options = {}) {
    const { method = 'GET', body = null } = options;
    // 处理 params / body 保证唯一：把对象转成字符串，避免因对象引用不同导致key不同
    return [
      url,
      method.toUpperCase(), // 统一转大写，避免 GET 和 get 视为不同请求
      JSON.stringify(options.params || {}), // params参数（GET请求常用）
      body ? JSON.stringify(body) : '' // body参数（POST请求常用）
    ].join('&'); // 用&连接，生成唯一key
  }

  // 核心请求方法（自动缓存 + 合并请求）
  request(url, options = {}) {
    const key = this.getCacheKey(url, options);

    // 1. 缓存存在 → 直接返回（分两种状态：pending / success）
    if (this.cache.has(key)) {
      const cached = this.cache.get(key);
      
      // 状态1：请求还在等待中（pending）→ 直接返回同一个Promise（实现请求合并）
      // 比如同时调用两次request('/api/user')，第一次发请求，第二次直接返回同一个Promise
      if (cached.status === 'pending') {
        return cached.promise;
      }
      
      // 状态2：请求已成功（success）→ 直接返回缓存数据（无需再次请求）
      return Promise.resolve(cached.data);
    }

    // 2. 缓存不存在 → 发送新请求
    const fetchPromise = fetch(url, options)
      .then(res => {
        // 处理HTTP错误（比如404、500），抛出错误，进入catch
        if (!res.ok) throw new Error('请求失败：' + res.status);
        return res.json(); // 解析响应数据（根据接口返回格式调整，比如res.text()）
      })
      .then(data => {
        // 请求成功 → 更新缓存为success状态，存储数据
        this.cache.set(key, { status: 'success', data });
        return data; // 返回数据，供外部使用
      })
      .catch(err => {
        // 请求失败 → 删除缓存（下次调用会重新请求，避免缓存错误数据）
        this.cache.delete(key);
        throw err; // 抛出错误，供外部捕获处理
      });

    // 先把请求存入缓存，标记为pending（正在请求），避免重复请求
    this.cache.set(key, { status: 'pending', promise: fetchPromise });

    return fetchPromise;
  }

  // 清空缓存（可指定url / 全部清空，灵活实用）
  clearCache(url) {
    if (!url) {
      this.cache.clear(); // 无url参数 → 清空所有缓存
      return;
    }
    // 有url参数 → 清空所有以该url开头的缓存（比如清空/api/user相关的所有请求缓存）
    for (const key of this.cache.keys()) {
      if (key.startsWith(url)) this.cache.delete(key);
    }
  }
}
```

#### 关键细节（通俗解读）：

- 唯一key生成：为什么要结合 url、method、params、body？因为不同的参数（比如 `/api/user?id=1` 和 `/api/user?id=2`）是不同的请求，需要不同的缓存；method 区分 GET/POST，避免同一url不同请求方式被误判为同一请求。

- 重复请求合并：当多个地方同时调用同一个请求时，缓存中存储的是 pending 状态的 Promise，所有调用都会返回同一个 Promise，最终只发一次请求，避免重复请求。

- 缓存更新逻辑：请求成功 → 缓存数据；请求失败 → 删除缓存（避免下次请求读取错误数据）；支持手动清空缓存（比如用户刷新页面、修改数据后，需要重新请求）。

#### 使用案例：

```js
// 实例化缓存工具
const cacheRequest = new CacheRequest();

// 第一次请求：无缓存，发送请求
cacheRequest.request('/api/user', { method: 'GET', params: { id: 1 } })
  .then(data => console.log('第一次请求：', data));

// 第二次请求：同一请求，缓存处于pending状态，合并请求（不重复发请求）
cacheRequest.request('/api/user', { method: 'GET', params: { id: 1 } })
  .then(data => console.log('第二次请求（合并）：', data));

// 一段时间后，第三次请求：缓存已存在，直接读缓存
setTimeout(() => {
  cacheRequest.request('/api/user', { method: 'GET', params: { id: 1 } })
    .then(data => console.log('第三次请求（读缓存）：', data));
}, 1000);

// 清空/api/user相关的所有缓存
cacheRequest.clearCache('/api/user');
```

#### 实际应用场景：

- 列表页、详情页：用户频繁刷新、点击，避免重复请求接口。

- 表单提交前的校验请求（比如验证手机号是否已注册），避免用户频繁点击导致多次请求。

- 需要手动刷新缓存的场景（比如用户修改数据后，清空对应缓存，重新请求最新数据）。

## 全文总结

本文从日常开发与面试高频场景出发，系统梳理了 JavaScript 数组与对象的十大核心操作技巧，覆盖「类型转换、结构处理、数据拷贝、引用管理」四大核心方向，每个技巧都兼顾「通俗理解、代码实现、避坑要点」，既适配新手入门的易读性，也满足老手复用的实用性，更贴合面试手撕的核心考点。

数组与对象作为 JS 最基础的复合数据结构，其操作能力直接决定了我们处理复杂业务数据的效率 —— 从类数组转换、负索引实现，到数组扁平化、树形结构转换，再到深拷贝、循环引用检测，这些技巧看似零散，实则都是「数据结构遍历与引用管理」核心逻辑的延伸。

在实际开发中，无需死记硬背所有实现方式，关键是理解「为什么这么做」：比如用 Proxy 实现负索引的核心是「拦截属性访问」，数组转树用 Map 映射的核心是「降低时间复杂度」，深拷贝处理循环引用的核心是「标记已遍历对象」。掌握底层逻辑后，无论需求如何变形（比如自定义扁平化规则、特殊类型的深拷贝），都能快速适配。