# 手写new
```js
/**
 * 手写new操作符
 * 1.创建一个空对象
 * 2.空对象的__proto__ = 构造函数的prototype
 * 3.将构造函数的this指向新对象
 * 4.将构造函数作用域的属性赋值给新对象
 * 5.返回新的对象
 */
function myNew(Constr, ...args) {
    // 创建一个新对象
    const newObj = {};
    // 链接对象
    newObj.__proto__ = Constr.prototype;
    // 绑定this
    let res = Constr.call(newObj, ...args);
    //判断返回类型
    return res instanceof Object ? res : newObj;
}

// 测试
function Father(name, age) {
    this.name = name;
    this.age = age;
}
const son = myNew(Father, '12', 'Lisa');
console.log(son);
```