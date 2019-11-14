# JavaScript篇

## 构造函数

**(2) 手动添加一个基本数据类型的返回值，最终还是返回 `this`。**

```jsx
function Person2() {
  this.age = 28;
  return 50;
}

var p2 = new Person2();
console.log(p2.age);   // 28
p2: {
  age: 28
}
```

如果上面是一个普通函数的调用，那么返回值就是 50。

**(3) 手动添加一个复杂数据类型(对象)的返回值，最终返回该对象**

直接上例子

```jsx
function Person3() {
  this.height = '180';
  return ['a', 'b', 'c'];
}

var p3 = new Person3();
console.log(p3.height);  // undefined
console.log(p3.length);  // 3
console.log(p3[0]);      // 'a'
```

再来一个例子

```jsx
function Person4() {
  this.gender = '男';
  return { gender: '中性' };
}

var p4 = new Person4();
console.log(p4.gender);  // '中性'
```

来源：  https://www.jianshu.com/p/95a5faee17f1 

## 原型对象

每个实例对象都有一个私有属性`__proto__`指向原型对象， 该原型对象也有一个自己的原型对象( __proto__ ) ，层层向上直到一个对象的原型对象为 `null`。根据定义，`null` 没有原型，并作为这个**原型链**中的最后一个环节。 

 ![img](https://img2018.cnblogs.com/blog/850375/201907/850375-20190708151322530-1608157973.png) 



## 继承方法

在JS中，任何函数都可以被添加到其他对象里作为对象的一个属性。

当继承的函数被调用时，`this`指的是当前继承的对象，而不是继承的函数的原型对象。

```js
var o = {
  a: 2,
  m: function(){
    return this.a + 1;
  }
};

console.log(o.m()); // 3
// 当调用 o.m 时，'this' 指向了 o.	

var p = Object.create(o);
// p是一个继承自 o 的对象

p.a = 4; // 创建 p 的自身属性 'a'
console.log(p.m()); // 5
// 调用 p.m 时，'this' 指向了 p
// 又因为 p 继承了 o 的 m 函数
// 所以，此时的 'this.a' 即 p.a，就是 p 的自身属性 'a' 
```

