在写vue数据劫持的时候遇到以下bug，在ES6入门·阮一峰-第2章-let和const命令-基本用法中，就有学到过，正向反馈学习成果，立即解决。
```javascript
var data = {name: 'jack', age: 23, hobby: 'travel'}
var obj = {}
for(var key in data){
    Object.defineProperty(obj, key, {
        get(){
            return data[key]
        },
        set(newValue){
            data[key] = newValue
        }
    })
}
```
结果输出Obj是这样的:
```javascript
age: "travel"
hobby: "travel"
name: "travel"
get age: ƒ get()
set age: ƒ set(newValue)
get hobby: ƒ get()
set hobby: ƒ set(newValue)
get name: ƒ get()
set name: ƒ set(newValue)
__proto__: Object
```
其本质原因就在于`var key`，最后方法调用`get`的时候，全局只有一个`key`, 其值是`hobby`, 所以会如此。改成let就会改变这个问题。