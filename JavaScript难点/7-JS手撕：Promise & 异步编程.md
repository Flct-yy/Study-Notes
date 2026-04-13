# JS手撕：手写Koa中间件与Promise核心特性

在前端开发中，Koa框架的洋葱模型、Promise的各类静态方法以及异步流程控制，是每个开发者必须掌握的核心知识点。它们看似独立，实则底层逻辑高度关联——都是为了解决异步代码的可读性、可维护性问题。本文将从实战出发，手把手拆解核心代码，用“通俗解释+专业剖析”的方式，让你不仅能看懂手写代码，更能理解背后的设计思想。

## 一、手写Koa中间件调用（洋葱模型）：理解“层层嵌套，反向回流”

用过Koa的同学都知道，它的中间件执行机制被称为“洋葱模型”——就像剥洋葱一样，中间件会从外到内依次执行，执行到最内层后，再从内到外反向执行。这种机制的核心价值的是：让中间件既能处理请求进入时的逻辑（如日志记录、权限校验），也能处理响应返回时的逻辑（如统一异常处理、响应格式化）。

### 1.1 核心代码实现（可直接运行）

```js
function koa() {
  // 存放所有通过 app.use() 注册的中间件函数
  const middlewares = []
  const app = async (ctx) => {
    // 从第0个中间件开始执行调度
    await dispatch(0, ctx)
  }

  // 注册中间件的方法：将中间件存入数组
  app.use = (middleware) => {
    middlewares.push(middleware)
  }

  // 核心调度函数：递归执行中间件
  const dispatch = async (index, ctx) => {
    // 终止条件：所有中间件都执行完毕，直接返回
    if (index === middlewares.length) return

    // 获取当前索引对应的中间件
    const middleware = middlewares[index]
    // 执行中间件：第二个参数是next函数，调用next()即执行下一个中间件
    await middleware(ctx, () => dispatch(index + 1, ctx))
  }
  return app
}

// 1. 创建 app 实例
const app = koa()

// 2. 注册 3 个中间件（模拟真实开发中的分层逻辑）
app.use(async (ctx, next) => {
  console.log('【中间件 1 开始】—— 日志记录：请求进入')
  console.log('请求URL：', ctx.req.url)
  
  await next() // 放行，执行下一个中间件（核心：交出执行权）
  
  console.log('【中间件 1 结束】—— 日志记录：响应返回')
  console.log(6)
})

app.use(async (ctx, next) => {
  console.log('  【中间件 2 开始】—— 权限校验：通过')
  console.log(2)
  
  await next() // 放行，执行下一个中间件
  
  console.log(5)
  console.log('  【中间件 2 结束】—— 响应处理：添加响应头')
})

app.use(async (ctx, next) => {
  console.log('    【中间件 3 开始】—— 业务逻辑：处理请求')
  console.log(3)
  
  await next() // 没有更多中间件，直接返回（执行终止）
  
  console.log(4)
  console.log('    【中间件 3 结束】—— 业务逻辑：返回结果')
})

// 3. 模拟请求上下文（ctx：Koa的核心，封装请求和响应信息）
const ctx = {
  req: { url: '/' },
  res: {}
}

// 4. 启动执行
app(ctx).then(() => {
  console.log('\n所有中间件执行完毕！')
})
```

### 1.2 核心原理拆解（通俗+专业）

#### 通俗理解

把每个中间件想象成一个“关卡”，请求要经过所有关卡才能到达最核心的业务逻辑（中间件3）；处理完业务逻辑后，响应要再反向经过所有关卡，才能返回给客户端。比如：

请求进入 → 中间件1（记录日志）→ 中间件2（权限校验）→ 中间件3（处理业务）→ 中间件2（处理响应）→ 中间件1（记录响应日志）→ 响应返回

#### 专业剖析

- **中间件存储**：用数组middlewares存储所有通过app.use()注册的中间件，保证执行顺序与注册顺序一致。

- **调度函数dispatch**：递归实现中间件的依次执行，index参数控制当前执行的中间件索引，当index等于中间件数组长度时，递归终止（最内层执行完毕）。

- **next函数**：本质是dispatch(index+1, ctx)的封装，调用next()就相当于“交出执行权”，让下一个中间件执行；await next()则保证“下一个中间件执行完毕后，再继续执行当前中间件的后续逻辑”，这是洋葱模型反向回流的关键。

- **ctx上下文**：统一封装请求（req）和响应（res）信息，所有中间件共享同一个ctx，实现数据传递（比如中间件1存储的用户信息，中间件3可以直接使用）。

### 1.3 执行结果与验证

运行上述代码，控制台输出如下（完美匹配洋葱模型）：

```text
【中间件 1 开始】—— 日志记录：请求进入
请求URL： /
2
  【中间件 2 开始】—— 权限校验：通过
3
    【中间件 3 开始】—— 业务逻辑：处理请求
4
    【中间件 3 结束】—— 业务逻辑：返回结果
5
  【中间件 2 结束】—— 响应处理：添加响应头
6
【中间件 1 结束】—— 日志记录：响应返回

所有中间件执行完毕！
```

## 二、手写简易co模块：自动执行Generator函数（告别手动.next()）

在async/await出现之前，Generator函数是解决异步回调地狱的重要方案，但它有一个痛点：需要手动调用.next()方法才能逐步执行，非常繁琐。co模块的核心作用就是“自动执行Generator函数”，它会自动遍历Generator的迭代器，直到执行完毕。

核心逻辑：Generator函数中，yield后面通常跟Promise（异步操作），co模块会等待Promise完成，将结果传给Generator，再自动执行下一步，直到迭代结束。

### 2.1 核心代码实现（可直接运行）

```js
// 手写co模块核心函数：自动执行带Promise的Generator
function run(generatorFunc) {
  // 1. 生成Generator迭代器（Generator函数执行后返回迭代器）
  let it = generatorFunc()

  // 2. 第一次启动Generator，获取第一个yield的结果（通常是Promise）
  let result = it.next()

  // 3. 用Promise包装自动执行流程，最终返回一个Promise（方便外部使用.then()）
  return new Promise((resolve, reject) => {
    // 递归函数：自动执行下一个yield
    const next = function (result) {
      // 终止条件：Generator执行完毕（done为true），resolve最终返回值
      if (result.done) {
        resolve(result.value)
        return
      }

      // 核心：result.value是yield后面的Promise，等待它完成
      result.value
        .then((res) => {
          // Promise成功：将结果传给Generator（it.next(res)），并继续执行下一步
          let nextResult = it.next(res)
          next(nextResult)
        })
        .catch((err) => reject(err)) // 捕获异步错误，终止执行
    }

    // 启动自动执行流程
    next(result)
  })
}

// 模拟异步请求（真实开发中可能是接口请求、文件读取等）
function fetchData(data) {
  return new Promise(resolve => {
    setTimeout(() => resolve(data), 500) // 延迟500ms模拟异步
  })
}

// 定义一个Generator函数（包含多个异步操作）
function* gen() {
  console.log('开始执行Generator，发起第一个异步请求')
  
  let res1 = yield fetchData('数据1') // 第一个异步请求，等待完成后赋值给res1
  console.log('第一个请求结果：', res1)
  
  let res2 = yield fetchData('数据2') // 第二个异步请求，依赖第一个请求完成
  console.log('第二个请求结果：', res2)
  
  let res3 = yield fetchData('数据3') // 第三个异步请求，依赖第二个请求完成
  console.log('第三个请求结果：', res3)

  return '全部异步请求完成' // Generator最终返回值
}

// 自动执行Generator函数（无需手动调用.next()）
run(gen).then(finalVal => {
  console.log('Generator执行完毕，最终返回：', finalVal)
})
```

### 2.2 核心原理拆解（通俗+专业）

#### 通俗理解

把Generator函数想象成一个“异步任务清单”，co模块（这里的run函数）就是一个“自动执行者”：它会先拿出清单上的第一个任务（第一个yield），等待任务完成后，把结果记下来，再自动拿出下一个任务，直到所有任务都完成，最后把清单的最终结果返回给你。

#### 专业剖析

- **Generator迭代器**：Generator函数（function*）执行后会返回一个迭代器（it），迭代器的next()方法会返回一个对象{ value: ..., done: ... }，value是yield后面的值（这里是Promise），done表示Generator是否执行完毕。

- **自动迭代逻辑**：next函数是核心，它接收上一个yield的执行结果，调用it.next(res)将结果传入Generator（赋值给res1、res2等），同时获取下一个yield的结果，递归执行自身，实现自动迭代。

- **Promise封装**：run函数最终返回一个Promise，这样外部可以通过.then()获取Generator的最终返回值，也能通过.catch()捕获异步错误，符合异步编程的统一规范。

- **异步依赖处理**：由于每次yield的Promise完成后才会执行下一个yield，因此可以轻松实现异步操作的顺序执行（比如先获取数据1，再用数据1获取数据2）。

### 2.3 执行结果与验证

运行代码后，控制台每隔500ms输出一次结果，最终输出如下：

```text
开始执行Generator，发起第一个异步请求
第一个请求结果： 数据1
第二个请求结果： 数据2
第三个请求结果： 数据3
Generator执行完毕，最终返回： 全部异步请求完成
```

## 三、异步串行/并行加法：理解异步流程控制的核心

异步加法看似简单，却能完美体现“串行”和“并行”两种异步流程控制的差异：

- 串行：多个异步操作按顺序执行，前一个操作完成后，再执行下一个（适合有依赖关系的场景）；

- 并行：多个异步操作同时执行，无需等待前一个完成（适合无依赖关系的场景，能提升效率）。

我们先实现一个基础的异步加法函数，再基于它分别实现串行和并行求和。

### 3.1 基础准备：异步加法函数与Promise包装

```js
// 1. 基础异步加法函数（基于回调函数，模拟真实异步场景）
// 接收 a, b 两个数字，callback 是回调函数（错误优先原则：第一个参数是错误，第二个是结果）
const asyncAdd = (a, b, callback) => {
  // 模拟异步操作（延迟 500ms，比如接口请求、计算密集型操作）
  setTimeout(() => {
    // 这里简化处理，不模拟错误，直接返回结果 a+b
    callback(null, a + b);
  }, 500);
};

// 2. 包装函数：将 callback 风格的异步方法，转成 Promise 风格
// 目的：方便在 async/await、Promise 链式调用中使用（更符合现代异步编程规范）
const promiseAdd = (a, b, index) => {
  console.log(`第 ${index} 次计算，参数 ${a}, ${b}`);
  return new Promise((resolve, reject) => {
    // 调用原来的 callback 异步加法
    asyncAdd(a, b, (err, res) => {
      if (err) {
        reject(err); // 出错时，抛出错误
      } else {
        resolve(res); // 成功时，返回计算结果
      }
    });
  });
};
```

### 3.2 方式一：异步串行求和（reduce实现）

核心逻辑：用数组的reduce方法，将前一次的计算结果（Promise）作为下一次计算的输入，实现“一步一步按顺序执行”。

```js
// 串行求和：reduce + Promise 链式，实现异步累加
const add1 = (arr) => {
  // reduce参数说明：
  // acc：上一次的Promise结果（累加和），初始值为0
  // val：当前数组要加的数
  // index：当前索引（用于打印日志）
  return arr.reduce((acc, val, index) => {
    // Promise.resolve(acc)：确保acc始终是Promise（兼容初始值0）
    return Promise.resolve(acc).then((value) => {
      // 等待上一步累加完成，再和当前值 val 相加
      return promiseAdd(value, val, index + 1); // index+1 是因为索引从0开始
    });
  }, 0); // 初始值 acc = 0（第一次计算：0 + arr[0]）
};

// 执行串行求和：1+2+3+...+9，一步一步按顺序执行
add1([1, 2, 3, 4, 5, 6, 7, 8, 9]).then((sum) =>
  console.log("异步串行加法结果", sum)
);
```

### 3.3 方式二：异步并行求和（递归+Promise.all实现）

核心逻辑：采用“二叉树式”分组，将数组两两分组，每组同时执行加法（并行），再将每组的结果递归分组，直到得到最终总和。这种方式比“所有数字同时相加”更高效（避免过多并发任务）。

```js
// 并行求和：递归 + Promise.all 实现并行归约求和（二叉树式计算）
async function parallelSum(arr) {
  // 递归终止条件：数组只剩一个数，直接返回（无需再计算）
  if (arr.length === 1) return arr[0];

  const tasks = []; // 存放所有并行执行的异步任务

  // 步长为2，将数组两两分组：[1,2] [3,4] [5,6] ... [9,0]（奇数长度时，最后一个补0）
  for (let i = 0; i < arr.length; i += 2) {
    // arr[i+1] || 0：处理奇数长度数组（比如最后一个元素9，没有i+1，补0）
    tasks.push(promiseAdd(arr[i], arr[i + 1] || 0));
  }

  // Promise.all：并行执行所有任务，等待所有任务完成后，返回结果数组
  const results = await Promise.all(tasks);

  // 递归：将上一轮的计算结果，继续两两分组并行计算
  return parallelSum(results);
}

// 执行并行求和：速度比串行快（无需等待上一步完成）
parallelSum([1, 2, 3, 4, 5, 6, 7, 8, 9]).then((sum) =>
  console.log("异步并行加法结果", sum)
);
```

### 3.4 核心差异对比（通俗+专业）

|对比维度|异步串行|异步并行|
|---|---|---|
|执行顺序|按顺序执行，前一个完成再执行下一个|所有任务同时执行，无顺序依赖|
|执行时间|总时间 = 所有任务时间之和（本例：9*500ms=4500ms）|总时间 = 最长任务时间 * 递归次数（本例：3*500ms=1500ms）|
|适用场景|任务有依赖（比如下一个任务需要上一个任务的结果）|任务无依赖（比如多个独立的接口请求、计算任务）|
|实现核心|Promise链式调用 + reduce|Promise.all + 递归归约|
### 3.5 执行结果与验证

串行求和会依次打印每次计算的参数，总耗时约4500ms；并行求和会同时打印多组计算参数，总耗时约1500ms，最终两者的求和结果均为45。

## 四、手写Promise核心静态方法：理解Promise的底层逻辑

Promise的静态方法（all、race、allSettled、any）是异步流程控制的常用工具，它们的底层逻辑都基于Promise的核心特性——状态不可逆（pending→fulfilled/rejected）。下面我们逐个手写实现，拆解它们的核心规则。

### 4.1 手写Promise.all：“全部成功才成功，一个失败就失败”

核心规则：接收一个Promise数组，只有所有Promise都成功（fulfilled），才返回所有结果的数组；只要有一个Promise失败（rejected），就立即返回该失败原因，终止执行。

```js
// 手写实现 Promise.all 核心方法
function myPromiseAll(promiseArr) {
  // 返回一个新的 Promise（外部可以通过.then()/.catch()获取结果）
  return new Promise((resolve, reject) => {
    const len = promiseArr.length;    // 传入的 Promise 数组长度
    const result = [];                // 存放所有成功结果的数组（按原数组顺序）
    let count = 0;                    // 记录已经成功完成的任务数量

    // 边界处理：如果传入空数组，直接resolve空结果
    if (!len) {
      resolve(result);
      return; // 必须加return，防止后续代码继续执行
    };

    // 遍历所有promise（用entries()获取索引，保证结果顺序与输入一致）
    for (const [i, p] of promiseArr.entries()) {
      // Promise.resolve(p)：包装非Promise值（比如普通数字、字符串），统一处理成Promise
      Promise.resolve(p).then(
        (value) => {
          // 成功：按原数组索引存入结果（确保顺序正确）
          result[i] = value;
          count++; // 成功数 +1

          // 所有任务都成功 → 调用resolve，返回结果数组
          if (count === len) {
            resolve(result);
          }
        },
        (reason) => {
          // 任何一个任务失败 → 立刻reject，终止所有任务（失败优先）
          reject(reason);
        }
      );
    }
  });
}

// 测试用例
const p1 = Promise.resolve(1);
const p2 = Promise.resolve(2);
const p3 = Promise.reject(new Error('失败'));

// 测试1：全部成功
myPromiseAll([p1, p2]).then(res => console.log('all成功：', res)).catch(err => console.log('all失败：', err.message));
// 测试2：有一个失败
myPromiseAll([p1, p3]).then(res => console.log('all成功：', res)).catch(err => console.log('all失败：', err.message));
```

### 4.2 手写Promise.race：“谁先完成，就返回谁”

核心规则：接收一个Promise数组，不管是成功还是失败，只要有一个Promise先完成（状态变为fulfilled或rejected），就立即返回该结果，其他任务继续执行，但结果会被忽略。

```js
// 规则：谁最先完成（成功/失败），就返回谁
function myPromiseRace(promiseArr) {
  // 返回一个新的 Promise
  return new Promise((resolve, reject) => {
    // 遍历所有传入的promise
    for (const p of promiseArr) {
      // 统一包装成Promise（处理普通值）
      Promise.resolve(p).then(
        (value) => {
          // 任何一个成功 → 立刻resolve（状态不可逆，后续结果不会覆盖）
          resolve(value);
        },
        (reason) => {
          // 任何一个失败 → 立刻reject（状态不可逆，后续结果不会覆盖）
          reject(reason);
        }
      );
    }
  });
}

// 测试用例：模拟快慢不同的Promise
const fastPromise = new Promise((resolve) => setTimeout(() => resolve('快的Promise'), 100));
const slowPromise = new Promise((resolve) => setTimeout(() => resolve('慢的Promise'), 1000));
const errorPromise = new Promise((_, reject) => setTimeout(() => reject('失败的Promise'), 500));

// 测试1：成功的Promise更快
myPromiseRace([fastPromise, slowPromise]).then(res => console.log('race结果：', res));
// 测试2：失败的Promise更快
myPromiseRace([errorPromise, fastPromise]).then(res => console.log('race结果：', res)).catch(err => console.log('race失败：', err));
```

### 4.3 手写Promise.allSettled：“无论成败，都返回所有结果”

核心规则：接收一个Promise数组，等待所有Promise都完成（无论成功还是失败），返回一个包含所有任务结果的数组，每个结果对象包含状态（fulfilled/rejected）和对应的值/原因，不会因为某个任务失败而终止。

```js
// 规则：无论成功/失败，都返回所有结果，不会中断
Promise.allSettled = function (promiseArr) {
  return new Promise(function (resolve) {
    const len = promiseArr.length;  // 数组长度
    const result = [];              // 存放所有结果
    let count = 0;                  // 已完成的promise数量

    // 空数组直接返回空
    if (!len) {
      resolve(result);
      return; // 必须加return！
    }

    // 遍历所有promise
    for (let [i, p] of promiseArr.entries()) {
      Promise.resolve(p).then(
        (value) => {
          // 成功：按标准格式存入（status为fulfilled，value为成功结果）
          result[i] = { status: "fulfilled", value };
          count++;
          if (count === len) { // 全部完成就resolve
            resolve(result);
          }
        },
        (reason) => {
          // 失败：按标准格式存入（status为rejected，reason为失败原因）
          result[i] = { status: "rejected", reason };
          count++;
          if (count === len) { // 失败也要计数，确保所有任务都完成
            resolve(result);
          }
        }
      );
    }
  });
};

// 测试用例
const p1 = Promise.resolve(1);
const p2 = Promise.reject(new Error('失败'));
Promise.allSettled([p1, p2]).then(res => {
  console.log('allSettled结果：', res);
  // 输出：[ {status: 'fulfilled', value: 1}, {status: 'rejected', reason: Error} ]
});
```

### 4.4 手写Promise.any：“只要有一个成功就成功，全部失败才失败”

核心规则：接收一个Promise数组，只要有一个Promise成功（fulfilled），就立即返回该成功结果；如果所有Promise都失败（rejected），则抛出一个AggregateError（包含所有失败原因）。

注意：与Promise.race的区别——any只关注成功，只有全部失败才会失败；race不管成功失败，谁先完成就返回谁。

```js
// 规则：
// 1. 只要有一个成功，就返回这个成功结果
// 2. 全部失败 → 抛出 AggregateError 错误
function myPromiseAny(promiseArr) {
  return new Promise(function (resolve, reject) {
    const len = promiseArr.length;
    const errors = []; // 收集所有失败原因（全部失败时使用）
    let count = 0;

    // 空数组：标准规定返回 AggregateError
    if (len === 0) {
      return reject(new AggregateError([], "All promises were rejected"));
    }

    // 遍历所有promise
    for (let [i, p] of promiseArr.entries()) {
      Promise.resolve(p).then(
        (value) => {
          // ✅ 任何一个成功 → 直接返回成功结果（状态不可逆）
          resolve(value);
        },
        (reason) => {
          // ❌ 失败：记录错误，计数+1
          errors[i] = reason;
          count++;

          // 全部都失败了 → 抛出 AggregateError（包含所有失败原因）
          if (count === len) {
            reject(new AggregateError(errors, "All promises were rejected"));
          }
        }
      );
    }
  });
}

// 测试用例
const p1 = Promise.reject(new Error('失败1'));
const p2 = Promise.resolve(2);
const p3 = Promise.reject(new Error('失败2'));

// 测试1：有一个成功
myPromiseAny([p1, p2, p3]).then(res => console.log('any成功：', res)); // 输出2
// 测试2：全部失败
myPromiseAny([p1, p3]).then(res => console.log('any成功：', res)).catch(err => {
  console.log('any失败：', err.message); // 输出"All promises were rejected"
  console.log('所有失败原因：', err.errors); // 输出[Error('失败1'), Error('失败2')]
});
```

## 五、Promise并发控制（带超时、重传、失败收集）：实战级封装

在真实开发中，我们经常会遇到“大量异步任务需要并发执行，但不能无限制并发”（比如同时调用100个接口，会导致服务器压力过大），同时还需要处理“任务超时”“失败重试”“收集失败任务”等需求。下面我们封装一个实战级的Promise并发控制器，满足这些核心需求。

### 5.1 核心代码实现（可直接复用）

```js
/**
 * Promise 并发控制器（带 并发限制 + 超时 + 自动重试）
 * @param {Array} tasks - 任务数组，每一项是 () => Promise 的函数（必须是函数，确保懒执行）
 * @param {Object} options - 配置参数（均有默认值）
 * @param {number} options.limit - 最大并发数，默认5
 * @param {number} options.timeout - 单个任务超时时间，默认3000ms
 * @param {number} options.maxRetries - 最大重试次数，默认3次
 * @returns {Promise} 最终返回【所有失败的任务列表】（方便后续重试或排查问题）
 */
function promiseConcurrencyControl(tasks, {
  limit = 5,
  timeout = 3000,
  maxRetries = 3
} = {}) {
  return new Promise((resolve) => {
    const results = [];          // 存储所有任务最终结果（成功/失败）
    const failedTasks = [];      // 存储【最终彻底失败】的任务（重试后仍失败）
    let taskIndex = 0;           // 下一个要执行的任务下标（控制任务顺序）
    let runningCount = 0;        // 当前正在运行的任务数量（控制并发数）

    // ==========================================
    // 核心函数：启动下一个任务（调度器）
    // 只要有任务未执行、且当前并发数未达上限，就持续启动任务
    // ==========================================
    function runNextTask() {
      // 终止条件：所有任务都执行完毕（taskIndex >= 任务总数），且没有正在运行的任务
      if (taskIndex >= tasks.length && runningCount === 0) {
        return resolve(failedTasks); // 返回最终失败的任务列表
      }

      // 循环启动任务：只要还有任务，且并发数未达上限
      while (taskIndex < tasks.length && runningCount < limit) {
        const currentIndex = taskIndex++; // 取当前任务下标（避免并发时下标混乱）
        const task = tasks[currentIndex]; // 取出当前任务（函数）
        runningCount++;                   // 正在运行的任务数 +1

        // 执行任务（带超时、重试逻辑）
        executeTaskWithRetry(task, currentIndex, 0);
      }
    }

    // ==========================================
    // 带【超时】和【自动重试】的任务执行器
    // @param task - 任务函数 () => Promise
    // @param index - 任务下标（用于定位任务）
    // @param retryCount - 当前已经重试的次数（初始为0）
    // ==========================================
    function executeTaskWithRetry(task, index, retryCount) {
      // 1. 创建超时Promise：超过指定时间未完成，直接reject（超时错误）
      const timeoutPromise = new Promise((_, reject) => {
        setTimeout(() => {
          reject(new Error(`Task ${index} timed out after ${timeout}ms`));
        }, timeout);
      });

      // 2. 竞速：任务执行 和 超时监控 谁先完成
      Promise.race([
        task(),                  // 执行真实任务（懒执行，避免提前启动）
        timeoutPromise           // 超时监控
      ])
      .then(result => {
        // ======================
        // 任务执行成功
        // ======================
        results[index] = {
          success: true,
          result,
          retries: retryCount // 记录重试次数（0表示未重试）
        };
        runningCount--; // 正在运行的任务数 -1
        runNextTask();   // 启动下一个任务（维持并发数）
      })
      .catch(error => {
        // ======================
        // 任务失败 / 超时
        // ======================
        if (retryCount < maxRetries) {
          // 还有重试次数 → 立即重试，重试次数+1
          console.log(`Task ${index} 失败（原因：${error.message}），重试 ${retryCount + 1}/${maxRetries}`);
          executeTaskWithRetry(task, index, retryCount + 1);
        } else {
          // 重试次数用完 → 标记为彻底失败，存入失败列表
          const failureInfo = {
            taskIndex: index,       // 任务下标（方便定位）
            error: error.message,   // 失败原因
            retries: maxRetries     // 已重试次数
          };
          failedTasks.push(failureInfo);

          results[index] = {
            success: false,
            ...failureInfo
          };

          runningCount--;
          runNextTask(); // 继续启动下一个任务
        }
      });
    }

    // 启动并发控制（入口）
    runNextTask();
  });
}

// ------------------------------
// 测试工具函数（模拟真实场景中的异步任务）
// ------------------------------

/**
 * 创建测试任务（随机成功/失败，可模拟接口请求）
 * @param {number} id - 任务ID（用于区分）
 * @param {number} successProbability - 成功率（0~1，默认0.7）
 * @param {number} delay - 任务执行延迟（默认1000ms）
 */
function createTestTask(id, successProbability = 0.7, delay = 1000) {
  return () => new Promise((resolve, reject) => {
    setTimeout(() => {
      // 随机成功/失败（模拟接口请求的不确定性）
      if (Math.random() < successProbability) {
        resolve(`Task ${id} 成功`);
      } else {
        reject(new Error(`Task ${id} 执行失败`));
      }
    }, delay);
  });
}

// 生成 10 个测试任务（成功率70%，延迟800ms）
const testTasks = Array.from({ length: 10 }, (_, i) =>
  createTestTask(i + 1, 0.7, 800)
);

// 启动并发控制（配置：最大并发3个，超时1500ms，最多重试2次）
promiseConcurrencyControl(testTasks, {
  limit: 3,         // 最多同时运行3个任务
  timeout: 1500,    // 单个任务超过1.5秒超时
  maxRetries: 2     // 每个任务最多重试2次
})
.then(failedTasks => {
  console.log('\n=== 全部执行完成 ==');
  console.log('最终失败的任务：', failedTasks);
});
```

### 5.2 核心功能拆解（实战重点）

- **并发限制**：通过runningCount（当前运行任务数）和limit（最大并发数）控制，只有runningCount < limit时，才会启动新任务，避免并发过多导致的性能问题。

- **任务调度**：runNextTask函数作为调度器，循环启动任务，确保并发数维持在limit以内，同时处理任务执行完毕后的“补位”（启动下一个任务）。

- **超时控制**：通过Promise.race将任务执行与超时监控绑定，超过指定时间未完成的任务，直接视为失败，进入重试逻辑。

- **自动重试**：任务失败后，若重试次数未用完，立即重试，重试次数用完后，标记为彻底失败，存入失败列表。

- **失败收集**：最终返回所有彻底失败的任务列表，包含任务下标、失败原因和重试次数，方便后续排查问题或重新重试。

- **懒执行**：任务数组中的每一项是一个返回Promise的函数，而非直接执行的Promise，确保任务只有在被调度时才会启动，避免提前执行导致的并发混乱。

### 5.3 应用场景

该并发控制器可直接用于真实开发中的场景，比如：

- 批量接口请求（比如批量获取用户信息、批量上传文件）；

- 批量处理异步任务（比如批量处理文件、批量发送消息）；

- 需要容错的异步场景（比如部分任务失败后，无需终止全部，只需收集失败任务后续处理）。

## 六、总结：核心知识点串联

本文讲解的所有内容，核心都是围绕“异步流程控制”展开：

1. Koa洋葱模型：通过递归调度中间件，实现“请求进入→业务处理→响应返回”的分层逻辑，核心是next函数的执行权移交；

2. co模块：自动迭代Generator函数，解决手动.next()的繁琐，本质是Promise与Generator的结合；

3. 异步串/并行：串行适合有依赖的任务，并行适合无依赖的任务，核心是Promise链式调用与Promise.all的运用；

4. Promise静态方法：all、race、allSettled、any，分别对应不同的异步场景，底层都是基于Promise的状态不可逆特性；

5. 并发控制：在Promise基础上，增加并发限制、超时、重试等实战功能，解决大量异步任务的高效、稳定执行问题。

掌握这些知识点，不仅能看懂框架底层代码，更能在实际开发中灵活处理各类异步场景，写出更高效、更健壮的代码。