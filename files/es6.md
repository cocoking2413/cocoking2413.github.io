##ES6
================
> 教程 http://es6.ruanyifeng.com/

ECMAScript 6 在接下来的一段时间内将成为 ECMAScript的一个标准。这个标准预计在今年的时候就会被签署，不管在Github,还是在很多社区，javascript爱好者已经早已开始拥抱变化，享受ES6 带来的美好,这篇文章将介绍ES6的一些新特性。由于ES6 还没有很好地被浏览器支持，所以这篇文章的ES6代码将使用 Babel 进行编译。

####基础
1. 箭头（Arrow）
    => 是function的简写形式，支持expression 和 statement 两种形式。同时一点很重要的是它拥有词法作用域的this值，帮你很好的解决this的指向问题，这是一个很酷的方式，可以帮你减少一些代码的编写，先来看看它的语法。
```js     
([param] [, param]) => {
        statements
}
param => expression
```
2. 类（class）
    ES6 引入了class（类），让javascript的面向对象编程变得更加容易清晰和容易理解。类只是基于原型的面向对象模式的语法糖。
```js 
            class Animal {
                // 构造方法，实例化的时候将会被调用，如果不指定，那么会有一个不带参数的默认构造函数.
                constructor(name,color) {
                this.name = name;
                this.color = color;
                }
                // toString 是原型对象上的属性
                toString() {
                console.log('name:' + this.name + ',color:' + this.color);

                }
            }
            
            var animal = new Animal('dog','white');
            animal.toString();

            console.log(animal.hasOwnProperty('name')); //true
            console.log(animal.hasOwnProperty('toString')); // false
            console.log(animal.__proto__.hasOwnProperty('toString')); // true

            class Cat extends Animal {
            constructor(action) {
                // 子类必须要在constructor中指定super 方法，否则在新建实例的时候会报错.
                // 如果没有置顶consructor,默认带super方法的constructor将会被添加、
                super('cat','white');
                this.action = action;
            }
            toString() {
                console.log(super.toString());
            }
            }

            var cat = new Cat('catch')
            cat.toString();
            
            // 实例cat 是 Cat 和 Animal 的实例，和Es5完全一致。
            console.log(cat instanceof Cat); // true
            console.log(cat instanceof Animal); // true
```
3. 类的 prototype 属性和 __proto__ 属性  
    在上一篇 javascript面向对象编程 中我们已经了解到一个实例化对象会有一个 __proto__ 指向构造函数的 prototype 属性。在 class 中。同时具有 __proto__ 和 prototype 两个属性，存在两条继承链。
    
    子类的 __proto__ 属性，表示构造函数的继承，总是指向父类。
    子类的 prototype 的 __proto__ 属性表示方法的继承，总是指向父类的 prototype 属性。
```
        class Cat extends Animal {}
        console.log(Cat.__proto__ === Animal); // true
        console.log(Cat.prototype.__proto__ === Animal.prototype); // true
``` 
    我们先来看第一条 Cat.__proto__ === Animal 这条原型链。完成构造函数继承的实质如下：
```
        class Cat extends Animal {
                construcotr() {
                        return Animal.__proto__.call(this);
                }
        }
```        
第二条对原型链 Cat.prototype.__proto__ === Animal.prototype 完成方法的继承，实质如下：
``` 
Cat.prototype.__proto__ = Animal.prototype
```
另外还有还有三种特殊情况。
```
        class A extends Object {}
        console.log(A.__proto__ === Object); // true
        console.log(A.prototype.__proto__ === Object.prototype); 
``` 
A继承Object，A的__prototype__ 指向父类Object. A的 prototype.__proto__ 指向父类Object的prototype。

从上篇文章中的 函数对象的原型 中我们可以了解到，函数是一种特殊的对象，所有函数都是 Function 的实例。
```
class Cat {}
console.log(Cat.__proto__ === Function.prototype); //true
console.log(Cat.prototype.__proto__ === Object.prototype); //true
```        
由于Cat不存在任何继承，就相当于一个普通函数，由于函数都是Function 的实例，所以 Cat.__proto__指向 Function.prototype. 第二条继承链指向父类（Function.prototype） 的prototype属性，所以 Cat.prototype.__proto__ === Object.prototype. Cat调用后会返回Object实例，所以 A.prototype.__proto__ 指向构造函数（Object）的prototype。
```
        class Cat extends null {};
        console.log(Cat.__proto__ === Function.prototype); // true;
        console.log(Cat.prototype.__proto__ === null); //true
```        
Cat是一个普通函数，所以继承 Function.prototype .第二条继承链不继承任何方法，所以 Cat.prototype.__proto__ == null.

4. Module  
    到目前为止,javascript (ES5及以前) 还不能支持原生的模块化，大多数的解决方案都是通过引用外部的库来实现模块化。比如 遵循CMD规范的 Seajs 和AMD的 RequireJS 。在ES6中，模块将作为重要的组成部分被添加进来。模块的功能主要由 export 和 import 组成.每一个模块都有自己单独的作用域，模块之间的相互调用关系是通过 export 来规定模块对外暴露的接口，通过import来引用其它模块提供的接口。同时还为模块创造了命名空间，防止函数的命名冲突。
```
        export,import 命令
        //test.js
        export var name = 'Rainbow'
```        
ES6将一个文件视为一个模块，上面的模块通过 export 向外输出了一个变量。一个模块也可以同时往外面输出多个变量。
```
        //test.js
        var name = 'Rainbow';
        var age = '24';
        export {name, age};
```        
定义好模块的输出以后就可以在另外一个模块通过import引用。
```
        //index.js
        import {name, age} from './test.js'
```        
整体输入，module指令
```
        //test.js
        
        export function getName() {
            return name;
        }
        export function getAge(){
        return age;
        } 
        通过 import * as 就完成了模块整体的导入。

        import * as test form './test.js';
        通过指令 module 也可以达到整体的输入。

        module test from 'test.js';
        test.getName();
        export default
不用关系模块输出了什么，通过 export default 指令就能加载到默认模块，不需要通过 花括号来指定输出的模块,一个模块只能使用 export default 一次

        // default 导出
        export default function getAge() {} 
        
        // 或者写成
        function getAge() {}
        export default getAge;

        // 导入的时候不需要花括号
        import test from './test.js';
一条import 语句可以同时导入默认方法和其它变量.

        import defaultMethod, { otherMethod } from 'xxx.js';
```
####要点


####注意



