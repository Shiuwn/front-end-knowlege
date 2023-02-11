# 手写p-limit

前段时间公众号上推送了一篇文章，看标题大概是讲如何自己手写一个`p-limit`，本来打算仔细拜读，但是事情太多忘记了，等我有了空闲的时候消息已经积压了很多，已经不方便找了。之前我做过一些并发的优化处理，便自己想着实现一下这个库的核心功能。

我们先来看这个库大概是怎么使用的。

```js
import pLimit from 'p-limit';

const limit = pLimit(1);

const input = [
	limit(() => fetchSomething('foo')),
	limit(() => fetchSomething('bar')),
	limit(() => doSomething())
];

// Only one promise is run at once
const result = await Promise.all(input);
console.log(result);
```

根据`p-limit`给出的代码示例，我们可以先把主要接口的轮廓写出来。

```ts
const pLimit = (limit: number) => {
  return (cb: () => Promise) => {
    return new Promise((resolve, reject) => {})
  }
}
const limit = pLimit(1)
const input = [
  limit(() => Promise.resolve(1)),
  limit(() => Promise.resolve(2)),
]
// ...
```
这样就搭好了`p-limit`的基本轮廓。接下来我们来实现核心的功能。

`p-limit`的功能是实现对`Promise`的节流，在我们利用`limit(() => Promise.resolve(1))`这种形式执行代码时，肯定是把待执行的函数加入到一个`调度器`，由这个`调度器`控制并发的执行。接下来我们实现以下这个调度器。

```ts
class Scheduler {
  limit = 1 // 并发限制数
  list: any[] = [] // 执行环境栈数组
  count: 0 // 当前执行的数量
  constructor(limit: number) {
    this.limit = limit
    this.count = 0
  }

  do() {
    // 执行环境数组不为空且没有达到并发限制
    while (this.list.length && this.count < this.limit) {
      // 取出执行环境
      const { callback: cb, resolve, reject } = this.list.shift()
      // 当前执行数量增加
      this.count++
      // 用Promise.resolve兼容callback结果不是Promise的情况
      Promise.resolve(cb()).then((res) => {
        // 执行成功当前执行数量减少
        this.count--
        // 导出结果
        resolve(res)
        // 回到开始继续执行
        this.do()
      }).catch(reject)
    }
  }
  add(context: any) {
    this.list.push(context)
  }
}
```
这个`调度器`包含两个方法，其中`add`是添加执行的环境，包括执行的`回调函数`、`resolve`和`reject`。`do`方法主要控制并发的执行，当执行栈数组非空且当前正在执行的回调函数的数量小于限制数量时就取出执行环境，执行回调函数。

我们把`调度器`放在`pLimit`函数内部。
```ts
const pLimit = (count: number = 1) => {
  // 生成一个调度器对象
  const scheduler = new Scheduler(count)
  return function limit<T>(cb: () => (Promise<T> | T)) {
    return new Promise((resolve, reject) => {
      // 添加执行环境
      scheduler.add({ callback: cb, resolve, reject })
      // 执行调度器
      scheduler.do()
    })
  }
}
```
这样我们就完成了`p-limit`的基本功能。

全部的代码长这样。
```ts
class Scheduler {
  limit = 1
  list: any[] = []
  count: 0
  constructor(limit: number) {
    this.limit = limit
    this.count = 0
  }

  do() {
    while (this.list.length && this.count < this.limit) {
      const { callback: cb, resolve, reject } = this.list.shift()
      this.count++
      Promise.resolve(cb()).then((res) => {
        this.count--
        resolve(res)
        this.do()
      }).catch(reject)
    }
  }
  add(context: any) {
    this.list.push(context)
  }
}
const pLimit = (count: number = 1) => {
  const scheduler = new Scheduler(count)
  return function limit<T>(cb: () => (Promise<T> | T)) {
    return new Promise((resolve, reject) => {
      scheduler.add({ callback: cb, resolve, reject })
      scheduler.do()
    })
  }
}

const limit = pLimit(1)

const ps = [
  limit(() => Promise.resolve(1)),
  limit(() => Promise.resolve(2)),
  limit(() => Promise.resolve(3)),
]

Promise.all(ps).then(console.log)
```