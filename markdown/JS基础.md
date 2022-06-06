

# JS基础

## js数据类型

- 基本类型：Number、Boolean、String、null、undefined、symbol（ES6 新增的），BigInt（ES2020） 
- 引用类型：Object，对象子类型（Array，Function） 

## JS 整数是怎么表示的？

- 通过 Number 类型来表示，遵循 IEEE754 标准，通过 64 位来表示一个数字，（1 + 11 + 52），最大安全数字是 Math.pow(2, 53) - 1，对于 16 位十进制。（符号位 + 指数位 + 小数部分有效位）

## Number() 的存储空间是多大？如果后台发送了一个超过最大自己的数字怎么办

- Math.pow(2, 53) ，53 为有效数字，会发生截断，等于 JS 能支持的最大数字。 

## new 一个函数发生了什么

构造调用：

- 创造一个全新的对象
- 这个对象会被执行 [[Prototype]] 连接，将这个新对象的 [[Prototype]] 链接到这个构造函数.prototype 所指向的对象
- 这个新对象会绑定到函数调用的 this
- 如果函数没有返回其他对象，那么 new 表达式中的函数调用会自动返回这个新对象

## new 一个构造函数，如果函数返回 `return {}` 、 `return null` ， `return 1` ， `return true` 会发生什么情况？

如果函数返回一个对象，那么new 这个函数调用返回这个函数的返回对象，否则返回 new 创建的新对象

## 问：`symbol` 有什么用处

可以用来表示一个独一无二的变量防止命名冲突。但是面试官问还有吗？我没想出其他的用处就直接答我不知道了，还可以利用 `symbol` 不会被常规的方法（除了 `Object.getOwnPropertySymbols` 外）遍历到，所以可以用来模拟私有变量。

主要用来提供遍历接口，布置了 `symbol.iterator` 的对象才可以使用 `for···of` 循环，可以统一处理数据结构。调用之后回返回一个遍历器对象，包含有一个 next 方法，使用 next 方法后有两个返回值 value 和 done 分别表示函数当前执行位置的值和是否遍历完毕。

```javascript
Symbol.for("bar") === Symbol.for("bar")
// true

Symbol("bar") === Symbol("bar")
// false
```

 除了定义自己使用的 Symbol 值以外，ES6 还提供了 11 个内置的 Symbol 值，指向语言内部使用的方法。 

如 Symbol.iterator

个人认为es6是为了不与其他库定义的属性有冲突才设计了Symbol

## 问：NaN 是什么，用 typeof 会输出什么？

Not a Number，表示非数字，typeof NaN === 'number'

## 问： 函数中的arguments是数组吗？类数组转数组的方法了解一下？

是类数组，是属于鸭子类型的范畴，长得像数组，

- ... 运算符
- Array.from
- Array.prototype.slice.apply(arguments)

## 问：用过 TypeScript 吗？它的作用是什么？

为 JS 添加类型支持，以及提供最新版的 ES 语法的支持，是的利于团队协作和排错，开发大型项目

## 问：ES6 之前使用 prototype 实现继承

Object.create() 会创建一个 “新” 对象，然后将此对象内部的 [[Prototype]] 关联到你指定的对象（Foo.prototype）。Object.create(null) 创建一个空 [[Prototype]] 链接的对象，这个对象无法进行委托。

```
function Foo(name) {
  this.name = name;
}

Foo.prototype.myName = function () {
  return this.name;
}

// 继承属性，通过借用构造函数调用
function Bar(name, label) {
  Foo.call(this, name);
  this.label = label;
}

// 继承方法，创建备份
Bar.prototype = Object.create(Foo.prototype);

// 必须设置回正确的构造函数，要不然在会发生判断类型出错
Bar.prototype.constructor = Bar;

 // 必须在上一步之后
Bar.prototype.myLabel = function () {
  return this.label;
}

var a = new Bar("a", "obj a");

a.myName(); // "a"
a.myLabel(); // "obj a"

```

## 问:如果一个构造函数，bind了一个对象，用这个构造函数创建出的实例会继承这个对象的属性吗？为什么？

不会继承，因为根据 this 绑定四大规则，new 绑定的优先级高于 bind 显示绑定，通过 new 进行构造函数调用时，会创建一个新对象，这个新对象会代替 bind 的对象绑定，作为此函数的 this，并且在此函数没有返回对象的情况下，返回这个新建的对象

## 手动实现new

```javascript
function objectFactory() {
	let result
    const constructor = Array.prototype.shift.apply(arguments);
    if(typeof constructor !== 'function') {
       throw new Error('type Error')
    }
    const newObj = Object.create(constructor.prototype);
    result = constructor.apply(newObj, arguments);
    
    const flag = result && (typeof result === 'object' || typeof result === 'function')
    
    return flag ? result || newObj
}
    
// 使用方式：
// objectFactory(构造函数， 初始化参数)
    
    function _new(fn, ...arg) {
        const obj = Object.create(fn.prototype);
        const ret = fn.apply(obj, arg);
        return ret instanceof Object ? ret : obj;
    }
```

## 手写bind

```javascript
Function.prototype.bind = function() {
  var args = Array.prototype.slice.call(arguments, 0)

  var addArgs = args.slice(1)
  var bindThis = args[0]
  var fn = this

  function resutFn() {
    var newFArgs = Array.prototype.slice.call(arguments, 0)
    var concatArgs = addArgs.concat(newFArgs)
    // 也可以用这个方法判断是否是在new console.log(this instanceof resutFn);
    // 或 this.__proto__ === resutFn.prototype
    return fn.apply(resutFn.prototype.isPrototypeOf(this) ? this : bindThis, concatArgs)
  }

  resutFn.prototype = fn.prototype

  return resutFn
}
```



## 解构层级深的对象

```javascript
const res  = {
    data: {
        pageData: {
            total: 0
        }
    }
}

const { data: { pageData: { total } } } = res;

// total -> 0
```

## 类数组转换为数组

 一个拥有 `length` 属性和若干索引属性的对象就可以被称为类数组对象，类数组对象和数组类似，但是`不能调用`数组的方法。常见的类数组对象有 `arguments` 和 `DOM` 方法的返回结果 

常见的几种转换方法

```javascript
Array.from(arrayLike);

Array.prototype.slice.call(arrayLike);

Array.prototype.splice.call(arrayLike, 0);

Array.prototype.concat.apply([], arrayLike);
```

## 数组有哪些原生方法

1. 转换为`字符串`：toString()、toLocalString()、join()

2. 数组`尾部`操作的方法 pop() 和 push()。

3. 数组`首部`操作的方法 shift() 和 unshift()

4. 数组`连接`的方法 concat() ，返回的是拼接好的数组，`不影响`原数组。

5. 数组`截取`办法 slice()，用于截取数组中的一部分返回，`不影响`原数组。

6. 数组`插入`方法 splice()，影响原数组查找特定项的索引的方法

7. indexOf() 和 lastIndexOf()

8. 迭代`方法 every()、some()、filter()、map() 和 forEach() 方法

9. 数组`归并`方法 reduce()

## 变量提升

```
var tmp = new Date();

function fn(){
    console.log(tmp);
    if(false){
        var tmp = 'hello world';
    }
}

fn();  // undefined
```

## ES6模块与CommonJS模块有什么异同？

1. `CommonJS` 模块输出的是一个值的拷贝，`ES6 Module` 输出的是值的引用

2. `CommonJS` 模块是运行时加载，`ES6 Module` 是编译时输出接口。

3. `CommonJS` 是单个值导出，`ES6 Module`可以导出多个

4. `CommonJS` 是动态语法可以写在判断里，`ES6 Module` 静态语法只能写在顶层

5. `CommonJS` 的 this 是当前模块，`ES6 Module` 的 this 是 undefined

## 不用递归将数组转换成tree结构

```javascript
function getTreeData(data, rootId) {
    let res = []
    let dataMap = {}

    for (const item of data) {
      const { id, pid } = item
      let treeItem = dataMap[id]

      if (!treeItem) {
        dataMap[id] = {
          ...item,
          children: []
        }
        treeItem = dataMap[id]
      } else {
        dataMap[id] = {
          ...item,
          ...treeItem
        }
      }
      if (pid === rootId) {
        res.push(treeItem)
      } else {
        if (!dataMap[pid]) {
          dataMap[pid] = {
            children: []
          }
        }

        dataMap[pid].children.push(treeItem)
      }
    }
    return res
  }
```

## WeakMap与Map类似，但有几点区别：

1. WeakMap只接受对象作为key，如果设置其他类型的数据作为key，会报错。

2. WeakMap的key所引用的对象都是弱引用，只要对象的其他引用被删除，垃圾回收机制就会释放该对象占用的内存，从而避免内存泄漏。

3. 由于WeakMap的成员随时可能被垃圾回收机制回收，成员的数量不稳定，所以没有size属性。

4. 没有clear()方法

5. 不能遍历

## MutationObserver

 接口提供了监视对 DOM 树所做更改的能力。 

```javascript
 // 选择需要观察变动的节点
const targetNode = document.getElementById('some-id');

// 观察器的配置（需要观察什么变动）
const config = { attributes: true, childList: true, subtree: true };

// 当观察到变动时执行的回调函数
const callback = function(mutationsList, observer) {
    // Use traditional 'for loops' for IE 11
    for(let mutation of mutationsList) {
        if (mutation.type === 'childList') {
            console.log('A child node has been added or removed.');
        }
        else if (mutation.type === 'attributes') {
            console.log('The ' + mutation.attributeName + ' attribute was modified.');
        }
    }
};

// 创建一个观察器实例并传入回调函数
const observer = new MutationObserver(callback);

// 以上述配置开始观察目标节点
observer.observe(targetNode, config);

// 之后，可停止观察
observer.disconnect();

```

