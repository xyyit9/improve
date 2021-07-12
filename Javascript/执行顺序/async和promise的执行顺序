# async/await 和 promise 的执行顺序

```javascript
async function async1() {
  console.log('async1 start')
  await async2()
  console.log('async1 end')
}

async function async2() {
  console.log('async2')
}

console.log('script start')

setTimeout(function() {
  console.log('setTimeout')
}, 0)

async1()

new Promise(function(resolve) {
  console.log('promise1')
  resolve()
}).then(function() {
  console.log('promise2')
})

console.log('script end')
```

https://juejin.cn/post/6844903734321872910#comment这篇中的执行顺序是不对的，但是思路很清晰

https://juejin.cn/post/6844903735290757133 解答了上篇中执行顺序不对的情况，详细描述整个浏览器演进流程

在老版本的chrome中：在每个 `await` (in ES2017) 引擎必须创建两个额外的 Promise（即使右侧已经是一个 Promise）并且它需要至少三个 microtask 队列 ticks，则改写是

```javascript
function async1(){
  console.log('2 async1 start')
  return async2()
    .then(() => {

    }).then(() => {

    }).then(() => {
      console.log('6 async1 end')
    })
}
function async2(){
  console.log('3 async2')
  // 这个函数返回一个 promise，如果不返回 promise，则应该在 async1 函数里面给 async2 函数的执行结果包一层 Promise
  return Promise.resolve(undefined)
}
```

因此输出为：

```
promise2
async1 end
```

在73版本的浏览器中，改进了这个bug，因此可以改写为：

```javascript
function async1(){
  console.log('2 async1 start')
  return async2().then(() => {
    console.log('6 async1 end')
  })
}
function async2(){
  console.log('3 async2')
  // 这个函数返回一个 promise，如果不返回 promise，则应该在 async1 函数里面给 async2 函数的执行结果包一层 Promise
  return Promise.resolve(undefined)
}
```

因此输出为：

```
async1 end
promise2
```

整个题目的输出为：

![image-20210707193904877](assets/image-20210707193904877.png)
