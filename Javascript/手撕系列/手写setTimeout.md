# 手写setTimeout

https://github.com/sisterAn/JavaScript-Algorithms/issues/98

https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestAnimationFrame

https://juejin.cn/post/6844903565048152078

https://blog.csdn.net/baidu_28196435/article/details/86575636

**`window.requestAnimationFrame()`** 告诉浏览器——你希望执行一个动画，并且要求浏览器在下次重绘之前调用指定的回调函数更新动画。该方法需要传入一个回调函数作为参数，该回调函数会在浏览器下一次重绘之前执行

```javascript
let setTimeout = (fn, timeout, ...args) => {
  // 初始当前时间
  const start = +new Date()
  let timer, now
  const loop = () => {
    timer = window.requestAnimationFrame(loop)
    // 再次运行时获取当前时间
    now = +new Date()
    // 当前运行时间 - 初始当前时间 >= 等待时间 ===>> 跳出
    if (now - start >= timeout) {
      fn.apply(this, args)
      window.cancelAnimationFrame(timer)
    }
  }
  window.requestAnimationFrame(loop)
}

mySetTimeout(() => {
  console.log("haha");
}, 3000);
```
