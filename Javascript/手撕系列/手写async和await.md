# 手写async/await

https://es6.ruanyifeng.com/#docs/async

原理： https://es6.ruanyifeng.com/#docs/generator-async#co-%E6%A8%A1%E5%9D%97

```javascript
// genFunction参数是generator函数
function asyncToGen(genFunction) {
    return function (...args) {
      // 执行generator会返回一个迭代器gen
      const gen = genFunction.apply(this, args);
      return new Promise((resolve, reject) => {
        function step(key, arg) {
          let genResult;
          try {
            // 执行gen.next()或者gen.throw()
            genResult = gen[key](arg);
          } catch (err) {
            return reject(err);
          }
          const { value, done } = genResult;
          // 如果执行完成，则要终止执行
          if (done) {
            return resolve(value);
          }
          // 如果没有执行完成，则返回一个fullfilled状态的promise，已有的value为参数
          return Promise.resolve(value).then(
            // resolve实参
            (val) => {
              step('next', val);
            },
            // reject实参
            (err) => {
              step('throw', err);
            },
          );
        }
        step('next');
      });
    };
  }
  const getData = () => new Promise(resolve => setTimeout(() => resolve('data'), 1000));
  function* testG() {
    const data = yield getData();
    console.log('data: ', data);
    const data2 = yield getData();
    console.log('data2: ', data2);
    return 'success';
  }
  
  const gen = asyncToGen(testG);
  gen().then(res => console.log(res));
```

以上代码原理如下：

```javascript
// 手动执行
var g = gen();

g.next().value.then(function(data){
  g.next(data).value.then(function(data){
    g.next(data);
  });
});
// 自动执行
function run(gen){
  var g = gen();

  function next(data){
    var result = g.next(data);
    if (result.done) return result.value;
    result.value.then(function(data){
      next(data);
    });
  }

  next();
}

run(gen);
```

