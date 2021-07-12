# 手写call/apply

模拟实现 `call` 有三步：

- 将函数设置为对象的属性
- 执行函数
- 删除对象的这个属性

```javascript
Function.prototype.call = function (context) {
  // 将函数设为对象的属性
  // 注意：非严格模式下, 
  //   指定为 null 和 undefined 的 this 值会自动指向全局对象(浏览器中就是 window 对象)
  //   值为原始值(数字，字符串，布尔值)的 this 会指向该原始值的自动包装对象(用 Object() 转换）
  context = context ? Object(context) : window; 
  context.fn = this;
    
  // 执行该函数
  // slice() 方法返回一个新的数组对象，这一对象是一个由 begin 和 end 决定的原数组的浅拷贝
  const args = [...arguments].slice(1);
  const result = context.fn(...args);
    
  // 删除该函数
  delete context.fn
  // 注意：函数是可以有返回值的
  return result;
}
```

`call` 和 `apply` 之间唯一的语法区别是 `call` 接受一个参数列表，而 `apply` 则接受带有一个类数组对象。

实现apply

```javascript
Function.prototype.apply = function (context, arr) {
    context = context ? Object(context) : window; 
    context.fn = this;
  
    let result;
    if (!arr) {
        result = context.fn();
    } else {
        result = context.fn(...arr);
    }
      
    delete context.fn
    return result;
}
```
