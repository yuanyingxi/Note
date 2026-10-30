# 1. Promise 基础
## 1.0 什么是异步？

**异步（Asynchronous）** 是一种执行方式，指执行某项操作时，**不需要让 JavaScript 主线程一直等待操作完成，可以先执行其他任务，等操作完成后再处理结果**。

**例如，前端向后端发送网络请求，可能需要等待几百毫秒甚至几秒。如果采用同步方式，后续代码就必须等待请求完成才能执行；采用异步方式，则可以在等待请求的过程中继续执行其他代码。**

### 1.0.1 同步与异步的区别

|类型|执行特点|
|---|---|
|同步|按照代码顺序执行，前面的操作未完成，后面的代码就需要等待|
|异步|发起操作后不必一直等待，可以先执行其他任务，操作完成后再处理结果|

### 1.0.2 常见的异步任务

- **网络请求：** 向后端请求用户信息，等待服务器返回数据。
    
- **定时器：** 使用 `setTimeout()` 安排一段代码稍后执行。
    
- **文件操作：** 异步读取文件，等读取完成后处理数据。
    
- **事件处理：** 等待用户点击按钮、输入内容等事件发生。
    

### 1.0.3 异步任务如何处理结果？

异步操作完成后，通常需要通过某种机制执行后续代码，例如：

- **回调函数（Callback）：** 将函数传递给其他代码，在操作完成后由其他代码调用。
    
- **Promise：** 用于表示异步操作的结果，通过 `then()`、`catch()` 处理成功或失败的情况。
    
- **async/await：** 基于 Promise 提供的语法，让异步代码更接近同步代码的写法。
    

**注意：**

1. **异步不等于同时执行多个 JavaScript 任务**。JavaScript 主线程上的代码通常仍然是一个任务接一个任务地执行。
    
2. 异步操作等待期间，主线程可以继续处理其他任务，不需要一直阻塞等待。
    
3. 单纯使用 Promise 并不能让耗时的 CPU 计算自动变成非阻塞操作，这类计算通常需要 Web Worker 等机制。
    

**记忆重点：异步的核心价值是避免主线程因为等待某项耗时操作而停下来，让程序在等待期间可以继续处理其他任务。**

## 1.1 什么是 Promise？

**Promise 是 JavaScript 中处理异步操作的一种机制，用于表示一个未来才会得到的结果。**

例如，**向服务器发送请求后，需要等待服务器返回数据。使用 Promise，可以在请求成功或失败后执行相应的操作。**

Promise 有三种状态：

|状态|含义|能否继续改变|
|---|---|---|
|`pending`|等待中|可以变成成功或失败|
|`fulfilled`|已成功|不可改变|
|`rejected`|已失败|不可改变|

Promise 的状态只能从 `pending` 变为 `fulfilled` 或 `rejected`，一旦进入终态，就不能再改变。

## 1.2 如何创建 Promise？

通过 `new Promise()` 可以创建一个 Promise 对象，**表示一个操作未来可能成功或失败的结果**。

```js
const p = new Promise((resolve, reject) => {
    const success = true;

    if (success) {
        resolve("操作成功");
    } else {
        reject("操作失败");
    }
});

p.then(result => {
    console.log(result);
}).catch(error => {
    console.log(error);
});
```

这段代码可以分成三个步骤。

**第一步：创建 Promise 对象**

```js
const p = new Promise((resolve, reject) => {
    // 在这里编写操作逻辑
});
```

`new Promise()` 接收一个函数作为参数，这个函数称为**执行器函数（Executor）**。

**创建 Promise 时，JavaScript 会立即执行这个函数，并自动向它传入两个函数：**

- `resolve(value)`：通知 Promise 操作成功，并传递成功结果。
    
- `reject(reason)`：通知 Promise 操作失败，并传递失败原因。
    

这里的 **`resolve` 和 `reject`** 并不是我们自己定义的，而是 **Promise 提供并传入的两个函数**。我们只需要在执行器函数中，根据操作结果决定调用哪一个。

**第二步：调用 resolve() 或 reject()，确定 Promise 的状态**

```js
resolve("操作成功");
```

调用 `resolve()`，表示操作成功，Promise 状态变为 `fulfilled`（已成功）。

```js
reject("操作失败");
```

调用 `reject()`，表示操作失败，Promise 状态变为 `rejected`（已失败）。

Promise 有三种状态：

|状态|含义|
|---|---|
|`pending`|等待中，尚未确定结果|
|`fulfilled`|已成功|
|`rejected`|已失败|

**注意：**

- Promise 创建后，初始状态为 `pending`。
    
- 第一次有效地调用 `resolve()` 或 `reject()` 后，Promise 的状态就会确定。
    
- 状态一旦确定，就不会再改变。后续调用另一个函数也不会改变结果。
    

因此，通常通过 `if-else` 根据实际操作结果，选择调用 `resolve()` 或 `reject()`。

**第三步：通过 then() 和 catch() 处理结果**

```js
p.then(result => {
    console.log(result);
}).catch(error => {
    console.log(error);
});
```

- `then()`：注册成功后的处理函数，通过参数 `result` 接收 `resolve()` 传递的成功结果。
    
- `catch()`：注册失败后的处理函数，通过参数 `error` 接收 `reject()` 传递的失败原因。
    

**因此：**

- **Promise 成功时，执行 `then()` 中的回调函数。**
    
- **Promise 失败时，执行 `catch()` 中的回调函数。**
    

#### Promise.resolve() 和 Promise.reject()

除了使用 `new Promise()`，还可以直接使用这两个静态方法创建已经确定状态的 Promise：

```js
// 创建一个成功的 Promise
const p1 = Promise.resolve(100);

// 创建一个失败的 Promise
const p2 = Promise.reject("出错了");
```

- `Promise.resolve(value)`：创建一个成功的 Promise，并传入成功结果。
    
- `Promise.reject(reason)`：创建一个失败的 Promise，并传入失败原因。
    

**总结：** `new Promise()` 用于创建 Promise 对象，并在执行器函数中根据操作结果调用 `resolve()` 或 `reject()` 确定状态；之后通过 `then()` 和 `catch()` 分别处理成功和失败的结果。

#### 如果操作失败，应该怎么处理？

可以使用 `catch()` 处理失败情况：

```js
const p = new Promise((resolve, reject) => {
    reject("失败");
});

p.then(result => {
    console.log(result); // 成功
}).catch(error => {
    console.log(error); // 失败
});
```

这里调用了 `reject("失败")`，因此成功处理函数不会执行，`catch()` 中的失败处理函数会接收到 `"失败"`。

#### 总结

可以把整个过程理解为：

```text
创建 Promise
    |
    v
执行传入的函数
    |
    v
根据操作结果调用 resolve 或 reject
    |
    v
Promise 成功或失败
    |
    v
通过 then() 或 catch() 处理结果
```

**补充：** 上面的示例为了方便理解，直接调用了 `resolve()` 或 `reject()`，并没有真正执行异步操作。实际开发中，可以在 Promise 内执行异步任务，并在任务完成后通知 Promise 结果。

## 1.3 Promise 的其他方法

**有时候我们已经有了一个结果，但仍然希望使用 Promise 的方式处理它。**

**这样做的好处是：无论结果是立即得到的，还是异步操作之后才得到的，都可以使用统一的 Promise 处理方式。**

```js
// 创建一个成功的 Promise
Promise.resolve(100);

// 创建一个失败的 Promise
Promise.reject("出错了");

// 处理成功结果
Promise.resolve(100).then(value => {
  console.log(value);
});

// 处理失败结果
Promise.reject("出错了").catch(error => {
  console.log(error);
});
```

- `Promise.resolve(value)`：返回一个成功完成的 Promise，结果通常为传入的值。
    
- `Promise.reject(reason)`：返回一个失败的 Promise。
    
- `.then()`：处理成功结果，也可以同时注册失败处理函数。
    
- `.catch()`：处理失败。
    
- `.finally()`：无论成功还是失败，都会执行指定的清理操作。
    

# 2. Promise 链式调用
## 2.1 什么是链式调用？

`then()` 会返回一个**新的 Promise**，因此可以连续调用多个 `then()`，把多个操作串联起来。

```js
Promise.resolve(1)
  .then(value => {
    return value + 1;
  })
  .then(value => {
    return value * 2;
  })
  .then(value => {
    console.log(value); // 4
  });
```

执行过程：

1. 第一个 `then()` 接收到 `1`，返回 `2`。
    
2. 第二个 `then()` 接收到 `2`，返回 `4`。
    
3. 第三个 `then()` 接收到 `4`，输出 `4`。
    

## 2.2 then 的返回值规则

**上一个 `then()` 的返回值，会影响下一个 `then()` 接收到的结果。**

| `then()` 中的操作 | 下一个 Promise 的结果       |
| ------------- | --------------------- |
| 返回普通值         | 成功，结果为该值              |
| 返回 Promise    | 等待该 Promise，并采用它的最终状态 |
| 抛出异常 `throw`  | 变为失败状态                |

示例：

```js
Promise.resolve(1)
  .then(value => value + 1)
  .then(value => {
    throw new Error("出错了");
  })
  .catch(error => {
    console.log(error.message); // 出错了
  });
```

**注意：** 使用花括号编写箭头函数时，需要显式写 `return`。

```js
// 正确：返回 2
.then(value => {
  return value + 1;
})

// 错误：没有 return，下一个 then 得到 undefined
.then(value => {
  value + 1;
})
```

## 2.3 catch 和 finally

```js
Promise.reject("出错了")
  .catch(error => {
    console.log(error);
    return "恢复成功";
  })
  .then(value => {
    console.log(value); // 恢复成功
  })
  .finally(() => {
    console.log("操作结束");
  });
```

- `catch()` 可以捕获前面链式调用中传递下来的失败。
    
- `catch()` 返回的也是 Promise，因此捕获错误后可以继续链式调用。
    
- `catch()` 如果正常返回一个值，后续链条可以恢复为成功状态。
    
- `finally()` 适合执行清理操作，通常不会改变原来的成功结果或失败原因；但如果它自身抛错或返回失败的 Promise，链条也会失败。
    

# 3. async / await
## 3.1 什么是 async / await？

**`async/await` 是基于 Promise 的异步编程语法，可以让异步代码写起来更接近同步代码，阅读起来更加直观。**

```js
async function getData() {
  const result = await Promise.resolve("数据");
  console.log(result);
}

getData(); // 数据
```

## 3.2 async 的作用

**`async` 函数始终返回一个 Promise。**

```js
async function getNumber() {
  return 100;
}

getNumber().then(value => {
  console.log(value); // 100
});
```

即使直接返回普通值，也会得到一个成功完成的 Promise。

如果没有显式返回值，Promise 成功结果为 `undefined`。

## 3.3 await 的作用

**`await` 用于等待一个 Promise 的结果，然后取出它成功时返回的结果，并暂停当前 async 函数后续代码的执行。如果 Promise 失败，`await` 则会抛出异常，可以通过 `try-catch` 捕获。**

```js
async function getData() {
  console.log("开始");

  const result = await Promise.resolve("数据");

  console.log(result);
  console.log("结束");
}

getData();

console.log("外部代码");
```

输出：

```text
开始
外部代码
数据
结束
```

原因：

1. 调用 `getData()`，先同步执行并输出“开始”。
    
2. 遇到 `await`，当前 async 函数暂停后续执行。
    
3. 外部代码继续执行，输出“外部代码”。
    
4. 等待结果处理完成后，async 函数继续执行，输出“数据”和“结束”。
    

**注意：`await` 不会阻塞整个 JavaScript 主线程，只会暂停当前 async 函数后续代码的执行。**

## 3.4 async/await 与 Promise 的关系

在实际开发中，经常需要向服务器发送请求，等待服务器返回数据，再进行后续处理。

例如：获取用户信息，并输出用户姓名。

### 3.4.1 使用 Promise 处理

```js
// 获取用户信息，返回一个 Promise
function getUser() {
    return fetch("/api/user")
        .then(response => response.json());
}

// 获取数据并处理
getUser()
    .then(user => {
        console.log(user.name);
    })
    .catch(error => {
        console.log("请求失败", error);
    });
```

执行过程：

1. 调用 `getUser()`，向服务器发送请求。
    
2. 请求尚未完成时，返回一个处于 `pending` 状态的 Promise。
    
3. 服务器返回数据后，Promise 成功，将用户信息传递给 `then()`。
    
4. `then()` 中的回调函数获取用户信息，并输出姓名。
    
5. 如果请求过程中发生错误，则通过 `catch()` 处理。
    

### 3.4.2 使用 async/await 处理

```js
// 获取用户信息，返回一个 Promise
function getUser() {
    return fetch("/api/user")   // `fetch()` 是 JavaScript 提供的网络请求 API，调用它会返回一个 Promise
        .then(response => response.json());
}

// 获取数据并处理
async function main() {
    try {
        const user = await getUser();
        console.log(user.name);
    } catch (error) {
        console.log("请求失败", error);
    }
}

main();
```

这里主要有两个关键字：

- **`async`：声明一个异步函数，该函数始终返回一个 Promise。**
    
- **`await`：等待一个 Promise 完成，并获取其成功结果。如果 Promise 失败，则会抛出异常，可以通过 `try-catch` 捕获。**
    

例如：

```js
const user = await getUser();
```

表示等待 `getUser()` 返回的 Promise 成功，并将其中的用户信息赋值给 `user`。

注意：`await` 只会暂停当前 `async` 函数后续代码的执行，不会阻塞整个 JavaScript 程序，其他任务仍然可以继续执行。

### 3.4.3 两者的关系与区别

上面两段代码完成的是同一个任务，只是写法不同：

|对比|Promise|async/await|
|---|---|---|
|处理成功结果|`then()`|`await` 获取结果|
|处理失败结果|`catch()`|`try-catch`|
|代码风格|链式调用|类似同步代码|
|多个连续异步操作|多次链式调用|多个 `await` 顺序执行|

例如，假设需要先获取用户信息，再根据用户 ID 获取订单信息。

使用 Promise：

```js
getUser()
    .then(user => {
        return getOrders(user.id);
    })
    .then(orders => {
        console.log(orders);
    })
    .catch(error => {
        console.log(error);
    });
```

使用 async/await：

```js
async function main() {
    try {
        const user = await getUser();
        const orders = await getOrders(user.id);

        console.log(orders);
    } catch (error) {
        console.log(error);
    }
}

main();
```

可以看到，`async/await` 可以让多个连续的异步操作看起来更接近普通的顺序代码，通常更容易阅读和维护。

**总结：**

- Promise 是 JavaScript 处理异步操作的基础机制。
    
- `async/await` 是基于 Promise 提供的语法，让异步代码更容易编写和理解。
    
- 两者都能处理异步操作，**实际开发中通常更推荐使用 `async/await` 编写多步骤异步业务逻辑**。
    

**注意： `async/await` 并没有取代 Promise。`await` 等待的通常就是 Promise，而异步函数本身也会返回 Promise。**

# 4. Promise 的组合方法

**当需要同时处理多个异步任务时，可以使用 Promise 的静态方法。**

|方法|成功条件|失败条件|返回结果|
|---|---|---|---|
|`Promise.all()`|所有任务成功|任意一个任务失败|所有任务的结果数组|
|`Promise.race()`|第一个完成的任务成功|第一个完成的任务失败|最先完成的任务结果或失败原因|
|`Promise.allSettled()`|等待所有任务完成|不会因为输入任务失败而直接失败|包含每个任务状态和结果的数组|
|`Promise.any()`|任意一个任务成功|所有任务都失败|第一个成功的任务结果|

这里的“完成”包括成功和失败。

## 4.1 Promise.all()

适用于多个任务都成功后，才能继续执行的场景。

```js
const p1 = Promise.resolve("用户信息");
const p2 = Promise.resolve("订单信息");

Promise.all([p1, p2]).then(results => {
  console.log(results);
  // ["用户信息", "订单信息"]
});
```

特点：

- 所有任务成功，才会成功。
    
- 任意一个任务失败，返回的 Promise 就会失败。
    
- 结果数组按照输入顺序排列，不受任务实际完成顺序影响。
    

## 4.2 Promise.race()

返回最先成功或失败的任务所决定的结果。

```js
Promise.race([
  new Promise(resolve => {
    setTimeout(() => resolve("任务 A"), 300);
  }),
  new Promise(resolve => {
    setTimeout(() => resolve("任务 B"), 100);
  })
]).then(result => {
  console.log(result); // 任务 B
});
```

常用于竞速或实现超时控制。

**重要：** `Promise.race()` 只决定最终采用哪个结果，**不会自动取消其他未完成的任务**。

## 4.3 Promise.allSettled()

适用于需要收集所有任务的执行结果，即使其中一些任务失败，也要继续处理其他结果的场景。

```js
Promise.allSettled([
  Promise.resolve("成功"),
  Promise.reject("失败")
]).then(results => {
  console.log(results);
});
```

结果的结构类似：

```js
[
  { status: "fulfilled", value: "成功" },
  { status: "rejected", reason: "失败" }
]
```

## 4.4 Promise.any()

适用于多个任务中，只要有一个成功就可以继续的场景。

```js
Promise.any([
  Promise.reject("任务 A 失败"),
  Promise.resolve("任务 B 成功"),
  Promise.resolve("任务 C 成功")
]).then(result => {
  console.log(result); // 任务 B 成功
});
```

即使部分任务失败，只要有一个成功，`Promise.any()` 就可以成功；只有全部失败时才会失败。

# 5. Promise 与微任务

Promise 的 `.then()`、`.catch()`、`.finally()` 回调会以微任务的形式执行。

JavaScript 执行代码时，通常遵循以下顺序：

1. **先执行**当前的**同步代码**。
    
2. 当前同步代码**执行完毕**后，**依次执行微任务队列中的任务**。
    
3. 微任务队列清空后，**再执行后续符合执行条件的定时器等任务**。
    

## 5.1 为什么 Promise 的回调不是立即执行？

```js
console.log("A");

Promise.resolve().then(() => {
    console.log("B");
});

console.log("C");
```

输出：

```text
A
C
B
```

这里需要区分两个概念：

- `Promise.resolve()`：立即创建一个已经成功的 Promise。
    
- `.then()`：注册成功后的回调函数，但不会立即执行，而是**将回调安排到微任务队列中**。
    

执行过程：

```text
1. 输出 A
      |
      v
2. 创建一个已经成功的 Promise
      |
      v
3. 将 then() 中的回调放入微任务队列
      |
      v
4. 继续执行同步代码，输出 C
      |
      v
5. 当前同步代码执行完毕
      |
      v
6. 执行微任务，输出 B
```

**重点：Promise 可以立即确定成功或失败状态，但它的 then() 等回调函数仍然会异步执行。**

## 5.2 为什么 setTimeout(..., 0) 也不是立即执行？

```js
console.log("A");

setTimeout(() => {
    console.log("B");
}, 0);

console.log("C");
```

输出：

```text
A
C
B
```

`setTimeout()` 用于安排定时器回调执行。

这里的 `0` 表示不主动等待额外的指定时间，尽可能早地安排回调执行，但并不意味着回调立即执行。

**原因是 JavaScript 必须先执行完当前同步代码，不能让定时器回调中途插入正在执行的代码。**

因此：

1. 输出 `A`。
    
2. `setTimeout()` 安排定时器回调。
    
3. 继续执行同步代码，输出 `C`。
    
4. 当前同步代码执行完毕后，定时器回调才有机会执行，输出 `B`。
    

**注意：** `0` 不代表精确等待 0 毫秒，实际执行时间还受到定时器调度和当前任务执行情况的影响。

## 5.3 Promise 与 setTimeout 的执行顺序

```js
console.log("A");

Promise.resolve().then(() => {
    console.log("B");
});

setTimeout(() => {
    console.log("C");
}, 0);

console.log("D");
```

输出：

```text
A
D
B
C
```

执行过程：

1. 输出 `A`。
    
2. Promise 的 `.then()` 回调进入微任务队列。
    
3. `setTimeout()` 安排定时器回调。
    
4. 继续执行同步代码，输出 `D`。
    
5. 当前同步代码执行完毕，执行微任务，输出 `B`。
    
6. 微任务队列清空后，执行定时器回调，输出 `C`。
    

因此最终输出：

```text
A
D
B
C
```

## 5.4 如果同步代码很多，then() 会一直等待吗？

**是的。微任务必须等待当前同步代码执行完毕，才能获得执行机会。**

例如：

```js
console.log("A");

Promise.resolve().then(() => {
    console.log("B");
});

// 假设这里有一段耗时的同步代码
for (let i = 0; i < 1000000000; i++) {
    // 消耗 CPU 时间
}

console.log("C");
```

输出顺序：

```text
A
C
B
```

原因是：

- `.then()` 回调进入微任务队列后，不会中断当前同步代码。
    
- 主线程继续执行循环，直到循环结束。
    
- 同步代码全部执行完毕后，才执行 `.then()` 回调。
    

**所以，如果同步代码耗时很长，微任务的执行也会被延迟。**

**比如，在前端向后端发起网络请求异步任务，即使网络请求已经完成，如果主线程还在执行耗时的同步代码，前端也可能迟迟无法处理响应、更新页面甚至完成页面渲染。**

### 5.5 异步的意义是什么？

**异步并不是让多个 JavaScript 代码同时执行，而是让程序在等待某个耗时操作时，不必一直占用主线程等待。**

例如，发送网络请求：

```js
fetch("/api/user").then(response => {
    console.log("请求完成");
});

console.log("继续执行其他代码");
```

**执行 `fetch()` 后，浏览器负责处理网络请求**。JavaScript 主线程不需要一直等待服务器响应，可以继续执行其他代码。

**等网络请求完成后，再通过 Promise 的回调处理结果。**

**但如果主线程正在执行耗时的同步代码，回调依然需要等待。**

**总结：**

- Promise 表示异步操作的结果。
    
- `.then()` 等回调会进入微任务队列。
    
- 当前同步代码执行完毕后，微任务才会执行。
    
- 异步的价值在于等待耗时操作期间，可以先执行其他任务，而不是让主线程一直等待。
    
- 异步不等于并行，JavaScript 主线程上的代码仍然是一个任务一个任务地执行。

# 6. 错误处理
## 6.1 使用 try/catch 捕获 await 的错误

```js
async function getData() {
  try {
    const result = await Promise.reject("请求失败");
    return result;
  } catch (error) {
    console.log(error); // 请求失败
    return "默认数据";
  }
}
```

当 `await` 等待的 Promise 失败时，会像抛出异常一样进入 `catch`。

## 6.2 Promise 的常见错误处理规则

- `.then()` 中抛出的异常，会让返回的新 Promise 进入失败状态。
    
- `.catch()` 可以捕获前面未处理的失败。
    
- `async` 函数内部抛出的异常，会使该函数返回的 Promise 失败。
    
- `await` 遇到失败的 Promise 时，需要使用 `try/catch` 处理，或者让异常继续向外传递。
    

使用 `fetch()` 请求接口时还要注意：**HTTP 404、500 等状态通常不会让 `fetch()` 自动进入失败状态，需要检查 `response.ok`。**

```js
async function getData() {
  try {
    const response = await fetch("/api/data");

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }

    return await response.json();
  } catch (error) {
    console.error("请求失败", error);
  }
}
```

# 7. 并发与串行：常见面试点

多个相互独立的请求，可以同时启动并用 `Promise.all()` 等待结果。

```js
// 并发执行
const [users, posts] = await Promise.all([
  fetchUsers(),
  fetchPosts()
]);
```

如果依次使用 `await`，后面的操作会等前面的操作完成后才开始。

```js
// 串行执行
const users = await fetchUsers();
const posts = await fetchPosts();
```

因此：

- **并发**：任务之间没有依赖关系，可以同时启动。
    
- **串行**：任务需要按照顺序执行，或者后续任务依赖前面的结果。
    

注意，`Promise.all()` 不会自动把任务变成并发。如果传入的 Promise 是通过先前的函数调用创建的，那么这些操作通常已经启动；`Promise.all()` 负责汇总并等待它们的结果。

# 8. 常见面试问题速记

1. **Promise 有哪三种状态？** `pending`、`fulfilled`、`rejected`。进入终态后不可逆。
    
2. **`then()` 为什么能链式调用？** 因为 `then()` 会返回一个新的 Promise。
    
3. **`then()` 返回普通值、Promise 或抛出异常有什么区别？** 分别对应成功结果、采用另一个 Promise 的最终状态、进入失败状态。
    
4. **`async` 和 `await` 有什么作用？** `async` 函数返回 Promise；`await` 等待结果并暂停当前 async 函数的后续执行。
    
5. **`Promise.all()` 和 `Promise.allSettled()` 有什么区别？** 前者任一任务失败就会失败；后者等待所有任务完成并收集各自结果。
    
6. **`Promise.race()` 能取消其他请求吗？** 不能，它只决定返回哪个最先完成的结果。
    
7. **Promise 回调什么时候执行？** `.then()` 等回调以微任务形式执行，当前同步代码结束后才会运行。
    
8. **多个独立请求如何提高效率？** 可以同时启动请求，再使用 `await Promise.all([...])` 等待结果。