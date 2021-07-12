# 手写setInterval

```javascript
function mySetInterval(fn, interval) {
  let timer;
  let startTime = Date.now();
  const loop = () => {
    timer = window.requestAnimationFrame(loop);
    if (Date.now() - startTime >= interval) {
      fn.call(this, timer);
      startTime = Date.now();
    }
  };
  timer = window.requestAnimationFrame(loop);
  return timer;
}

let a = 0;
mySetInterval((timer) => {
  console.log(1);
  a++;
  if (a === 3) cancelAnimationFrame(timer);
}, 1000);
```
