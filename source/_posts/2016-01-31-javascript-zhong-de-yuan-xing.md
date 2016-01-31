---
layout: post
title: "JavaScript 中的原型"
date: 2016-01-31 22:33:23 +0800
comments: true
categories: 
---

##prototype
每个JavaScript function对象都有一个prototype属性，prototype也是一个对象。

######为什么需要prototype？
假设没有prototype属性，创建实例对象：

{% codeblock lang:javascript %}
function Persion(name) {
  this.name = name;
  this.species = "人类";
}
// 生成两个实例对象
var p1 = new Persion("p1");
var p2 = new Persion("p2");
{% endcodeblock %}
	
Persion的实例的属性species的值都是相同的，但是每个实例对象也还都保留着species属性的副本，这样浪费了资源，也无法做到数据共享。引入prototype后可以这样改写：

{% codeblock lang:javascript %}
function Persion(name) {
  this.name = name;
}
Persion.prototype.species = "人类";
var p1 = new Persion("p1");
var p2 = new Persion("p2");
p1.species // 人类
p2.species // 人类
{% endcodeblock %}

实例对象一旦创建，将自动引用prototype对象的属性和方法。实例对象p1和p2的species属性引用自prototype，改变prototype的pecies的值，p1和p2 species的值也会跟着改变。实例对象需要共享的属性和方法，都放在prototype里，不需要共享的属性和方法放在构造函数中。
	
##constructor
constructor是创建对象的构造函数，如上面例子中Persion就是一个constructor函数，通过`new constructor`生成一个实例对象。JavaScript内置constructor有：

{% codeblock lang:javascript %}
var x1 = new Object();    // A new Object object
var x2 = new String();    // A new String object
var x3 = new Number();    // A new Number object
var x4 = new Boolean()    // A new Boolean object
var x5 = new Array();     // A new Array object
var x6 = new RegExp();    // A new RegExp object
var x7 = new Function();  // A new Function object
var x8 = new Date();      // A new Date object
{% endcodeblock %}

##\__proto__
__proto__是普通对象的隐式属性，在new的时候，会指向prototype所指的对象；
__ptoto__实际上是某个实体对象的属性，而prototype则是属于构造函数的属性。__ptoto__只能在学习或调试的环境下使用。

{% codeblock lang:javascript %}
function Point(x, y) {
  this.x = x;
  this.y = y;
}
var myPoint = new Point(); // true
myPoint.__proto__ == Point.prototype // true
myPoint.__proto__.__proto__ == Object.prototype // true
myPoint instanceof Point; // true
myPoint instanceof Object; // true	
{% endcodeblock %}

##参考文章
[Javascript继承机制的设计思想](http://www.ruanyifeng.com/blog/2011/06/designing_ideas_of_inheritance_mechanism_in_javascript.html)   
[继承与原型链](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)


