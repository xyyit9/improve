# 手写JSON.stringfy和JSON.parse

JSON.stringfy的一些特性：

1. `undefined`、函数以及 `symbol` 作为对象属性值时 `JSON.stringify()` 对跳过（忽略）它们进行序列化

2. `undefined`、函数以及 `symbol` 作为对象属性值时 `JSON.stringify()` 对跳过（忽略）它们进行序列化

3. `undefined`、函数以及 `symbol` 被 `JSON.stringify()` 作为单独的值进行序列化时，都会返回 `undefined`

4. 转换值如果有 toJSON() 方法，该方法定义什么值将被序列化

5. 布尔值、数字、字符串的包装对象在序列化过程中会自动转换成对应的原始值

6. 对包含循环引用的对象（对象之间相互引用，形成无限循环）执行此方法，会抛出错误。

7. 所有以 symbol 为属性键的属性都会被完全忽略掉，即便 `replacer` 参数中强制指定包含了它们。

8. Date 日期调用了 toJSON() 将其转换为了 string 字符串（同Date.toISOString()），因此会被当做字符串处理。

9. NaN 和 Infinity 格式的数值及 null 都会被当做 null。

10. 其他类型的对象，包括 Map/Set/WeakMap/WeakSet，仅会序列化可枚举的属性。

11. 非数组对象的属性不能保证以特定的顺序出现在序列化后的字符串中。


手写JSON.stringfy和JSON.parse

```javascript
function jsonParse(jsonStr) {
  return eval("(" + jsonStr + ")");
}

function jsonStringify(jsonObj) {
  var result = "",
    curVal;
  if (jsonObj === null) {
    return String(jsonObj);
  }
  switch (typeof jsonObj) {
    case "number":
    case "boolean":
      return String(jsonObj);
    case "string":
      return '"' + jsonObj + '"';
    // 先把undefined、函数以及symbol转换成undefined
    // 如果在数组中，则转为null；如果在对象中，则不加入结果集
    case "undefined":
    case "function":
    case "symbol":
      return undefined;
  }

  switch (Object.prototype.toString.call(jsonObj)) {
    case "[object Array]":
      result += "[";
      for (var i = 0, len = jsonObj.length; i < len; i++) {
        curVal = JSON.stringify(jsonObj[i]);
        result += (curVal === undefined ? null : curVal) + ",";
      }
      // 去除最后一个逗号
      if (result !== "[") {
        result = result.slice(0, -1);
      }
      result += "]";
      return result;
    case "[object Date]":
      return (
        '"' + (jsonObj.toJSON ? jsonObj.toJSON() : jsonObj.toString()) + '"'
      );
    case "[object RegExp]":
      return "{}";
    case "[object Object]":
      result += "{";
      for (i in jsonObj) {
        if (jsonObj.hasOwnProperty(i)) {
          curVal = jsonStringify(jsonObj[i]);
          if (curVal !== undefined) {
            result += '"' + i + '":' + curVal + ",";
          }
        }
      }
      // 去除字符串中的最后一个逗号
      if (result !== "{") {
        result = result.slice(0, -1);
      }
      result += "}";
      return result;
  }
}
```
