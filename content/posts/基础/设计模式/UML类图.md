---
title: 设计模式-UML类图
tags:
  - 设计模式
categories: 基础
copyright: true
---

![UML类图](https://raw.githubusercontent.com/wangxiaohong123/p-bed/main/uPic/UML类图.png)

##### 类和接口

矩形表示类或者接口，接口分2层，第一层是类名，类名的上方使用\<\<interface\>\>标记，图里最下面是接口的另一种画法，棒棒糖表示法，圆圈表示接口，下面的文字是接口名，棒棒糖的另一端是指向实现类。

类可以是3层也可以是两层，第一层是类名，斜体表示抽象类，第二层是变量，第三层是方法。

类和接口的变量或者方法前面如果是'+'表示public，'-'表示private，'#'表示protected。

##### 关系表示

1.   继承：使用空心三角形+实线；

2.   实现：使用空心三角形+虚线；

3.   关联：使用实线箭头，表示一个类需要知道另一个类时表示关联关系：

     ```java
     public class Penguin extends Bird {
         // 在企鹅(Penguin)类中，引用气候(Climate)对象
         private Climate climate;
     }
     ```

4.   依赖：使用虚线箭头，这种关系是临时性的，通常是作为方法的参数传递：

     ```java
     public class Animal {
         // 新陈代谢方法需要用到氧气和水
         public Metabolism metabolism(Oxygen oxygen, Water water) {}
     }
     ```

5.   聚合：使用空心菱形+实线箭头，聚合是弱拥有：

     ```java
     public class WideGooseAggregate {
         // 雁群类中有大雁数组对象
         private WideGoose[] arrayWideGoose;
     }
     ```

6.   组合：使用实心菱形+实线箭头，组合是强拥有，部分和整体的生命周期一样，组合是关联的一种，组合强调的是不同整体不能同时拥有同一部分：

     ```java
     public class Bird {
         // 鸟和翅膀在语义上是组合关系，他的代码和关联是一样的
         private Wing wing;
         public Bird() {
             // 类初始化时实例化翅膀
             this.wing = new Wing();
         }
     }
     ```

关联和组合的区分只能通过上下文语义，通过代码是看不出来的。
