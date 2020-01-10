# EcmaScript6
ES 6语法简介
1．let、const
let和const都是块级作用域, var不是块级;
```js
// let示例
     {
       var a = 1;
       let b = 2;
     }
     a      // 1
     b      // ReferenceError: b is not defined.
```

```js
    //const示例
     const c = 3;
     c = 4;   //TypeError: Assignment to constant variable.
```
2．箭头函数
使用“箭头”（=>）定义函数, 不需要function 关键字，并且还可以省略return关键字
同时，箭头函数内的this指向函数定义时所在的上下文对象，而不是函数执行时的上下文对象。

```js
var f = a => a + 1;
     //等价于
     var f = function(a){
       return a + 1;
     }

function foo(){
       this.bar = 1;
       this.f = (a) => a + this.bar;
     }
     //等价于
     function foo(){
       this.bar = 1;
       this.f = (function(a){
         return a + this.bar
       }).bind(this);
     }
```
箭头函数的参数多于1个或者不需要参数，就需要使用一个圆括号代表参数部分
```js
var f = () => 1;
     var f = (a, b) => a + b;
```
如果函数体内包含的语句多于一条，就需要使用大括号将函数体括起来，使用return语句返回。例如：
```js
var f = (x, y) => { x++; y--; return x + y; }
```
3. 模板字符串

   1. 模板字符串用反引号（`）标识字符串, 还可以用来定义多行字符串，以及在字符串中嵌入变量'Hello, ${name} ! '，功能很强大。

4. 解构赋值

   1. 数组解构: let [a,b,c] = [1, 2, 3];

   2. 对象解构:
          let name = 'Lily';
          let age = 4;
          let person = {name, age};
          person // Object {name: "Lily", age: 4}

   3. 对象反解构:

      let person = {name: 'Lily', age: 4};

      let {name, age} = person;

   4. 数组参数解构:function sum([x, y]){ return x + y; }

   5. 对象参数解构:function sum ({x, y}){ return x + y; }

   6. 解构嵌套数组或对象:

      let [a, [b], c] = [1, [2], 3];

      let {person: {name, age}, foo} = {person: {name: 'Lily', age: 4}, foo: 'foo'};

5. rest参数（形式为...变量名）是一个数组，数组中的元素是多余的参数。注意，rest参数之后不能再有其他参数:

   function languages(lang, ...types){
   ​	console.log(types);
   }
   languages('JavaScript', 'Java', 'Python');  //["Java", "Python"]

6. 扩展运算符是三个点（...），它将一个数组转为用逗号分隔的参数序列，类似于rest参数的逆运算。 let numbers = [1, 2, 3];sum(...numbers);  //6

7. 扩展运算符还常用于合并数组以及与解构赋值结合使用。
   [...arr1, ...arr2, ...arr3];  //['a', 'b', 'c', 'd', 'e'];
   let [a, ...rest] = ['a', 'b', 'c'];
   rest  //['b', 'c']

8. 扩展运算符还可以用于取出参数对象的所有可遍历属性，复制到当前对象之中。例如：let bar = {a: 1, b: 2};let foo = {...bar};

9. class定义:

   1. 构造函数:constructor(name, age)
   2. 数据成员:无需声明, 直接this.argName, this.name;
   3. 方法声明, 无需关键字, getName(){return this.name;}
   4. 通过extends关键字实现继承, 构造函数内调用super(arg)

10. import 引入其他模块, export规定对外暴露接口;

    1. 引入默认接口: import foo from './a';a的全名是a.js, 对外export接口foo
    2. 引入普通接口: import {bar} from './a'; bar是a.js里的其他方法;
    3. 定义导出默认接口:export default foo;

# js
## jQuery
### 遍历对象或者数组:
jQuery.each(obj, function(i, val) {
    text = text + "Key:" + i + ", Value:" + val;
});
