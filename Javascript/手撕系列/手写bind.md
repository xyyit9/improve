# 手写bind

`Object.prototype.toString.call()`等价于` Function.prototype.call.bind(Object.prototype.toString)`

首先我们来实现以下四点特性：

1. 可以指定`this`
2. 返回一个函数
3. 可以传入参数
4. 柯里化

```javascript
Function.prototype.bind2 = function (context) {

  if (typeof this !== "function") {
    throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
  }
  // this为调用者bar函数
  var self = this; 
  // 因为第1个参数是指定的this,所以只截取第1个之后的参数
  var args = Array.prototype.slice.call(arguments, 1); 

  var fNOP = function () {};

  var fBound = function () {
      // 这时的arguments是指bind返回的函数传入的参数, 即bindFoo调用时候的参数
      var bindArgs = Array.prototype.slice.call(arguments); 
      /**
       * 当作为构造函数时，this 指向实例，此时 this instanceof fBound 结果为 true, 可以让实例获得来自绑定函数的值
       * 当作为普通函数时，this 指向 window，此时结果为 false，将绑定函数的 this 指向 context
       * 例：
       * var obj = new bindFoo(20), self是闭包使用，值为bar函数，this为obj, 即bar.apply(obj, args)
       * bindFoo(20),self是闭包使用，值为bar函数, this是window,即bar.apply(window, args)
       */
      return self.apply(this instanceof fNOP ? this : context, args.concat(bindArgs));
  }

  fNOP.prototype = this.prototype;
  // 让fBound继承FNOP，而FNOP的原型等于当前调用者的原型，
  // 即让bindFoo继承bar的原型，bindFoo可以获取到bar原型上的friend
  fBound.prototype = new fNOP();
  return fBound;
}
```
