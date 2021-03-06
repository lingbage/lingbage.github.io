---
title: Java到底是传值还是传引用?
date: 2016-04-13 09:53:32
tags:
- Java
- 值传递
- 引用传递
categories: Java
---
# 1、正确看待传值还是传引用的问题 #

首先说，为什么会有这样一个问题。实际上，问题来源于C，而不是Java。

C 语言中有一种数据类型叫做指针，于是将一个数据作为参数传递给某个函数的时候，就有两种方式：传值，或是传指针，它们的区别，可以用一个简单的例子说明：
<!-- more -->

```c
void Exchg1(int x, int y) /* 传值*/
{
	int tmp;
	tmp = x;
	x = y;
	y = tmp;
	printf("x = %d, y = %d.\n", x, y);
}
void Exchg2(int *px, int *py) /* 传指针*/
{
    int tmp = *px;
    *px = *py;
    *py = tmp;
    printf("*px = %d, *py = %d.\n", *px, *py);
}
main()
{
    int a = 4;
    int b = 6;
    Exchg1(a, b);
    printf("a = %d, b = %d.\n”, a, b);
	a=4;b=6;
    Exchg2(&a, &b);
    printf("a = %d, b = %d.\n", a, b);
}
```

依次输出为：

```
-----------------------------------------
x = 6, y = 4.
a = 4, b = 6.
-----------------------------------------
*px = 6, *py = 4.
a = 6, b = 4.
-----------------------------------------
```

大家可以明显的看到，按指针或者引用传递参数可以方便的修改通过参数传递进来的值，而按值传递就不行。

在Java 成长起来的时候，许多的 C 程序员开始转向学习 Java，他们发现，使用类似例子中的Exchg的方法仍然不能改变通过参数传递进来的简单数据类型的值，但是如果是一个对象，则可能将其成员随意更改。

于是他们觉得这很像是 C 语言中传值/传指针的问题。但是 Java 中没有指针，那么这个问题就演变成了传值/传引用的问题。可惜将这个问题放在 Java 中进行讨论并不恰当。

讨论这样一个问题的最终目的只是为了搞清楚载何种情况下，才能在方法函数中方便的更改参数的值并使之长期有效。

如果不弄清楚这个问题，看似不影响你写出一段美妙的代码，但却有极大的隐患。不一定什么时候你就会莫名其妙，为什么我的返回参数没有起作用，为什么我传进去的参数变成了这样？

# 2、基本数据类型的参数传递 #

Java 方法的参数是基本类型的时候，是按值传递的 (pass by value)，这一点是毋庸置疑的。我们可以通过一个简单的例子来说明：

```java
public static void switchValue (int a,int b){
	int c=a;
	a=b;
	b=c;
}
public static void main(String[] args) throws ParseException {
	int x=0;
	int y=1;
	switchValue(x,y);
	System.out.println("x="+x);
	System.out.println("y="+y);
}
```

输出结果为：

```
------------------------
x=0
y=1
------------------------
```

不难看出，虽然在方法switchValue中交换了两个参数的值，但对参数源并没有影响。也就是说基本数据类型的参数时，是传值的。传入的参数其实是参数源的一个拷贝，在实际方法中改变参数的拷贝，并不会影响参数源的值。

# 3、对象做为参数的传递 #

Java是传值还是传引用，问题主要出在对象传递上。前边也讲到基本数据类型是按照值传递的。要讨论这个问题，必须要知道什么是引用。

简单的说，引用其实就像是一个对象的名字或者别名 (alias)，一个对象在内存中会请求一块空间来保存数据，根据对象的大小，它可能需要占用的空间大小也不等。访问对象的时候，我们不会直接是访问对象在内存中的数据，而是通过引用去访问。引用也是一种数据类型，我们可以把它想象为类似 C 语言中指针的东西（实际有很大的区别），它指示了对象在内存中的地址——只不过我们不能够观察到这个地址究竟是什么。(注：Java里面没有指针的概念，并不是指Java中没有指针，Java的设计者巧妙的对指针的操作进行了管理。而在懂C的Java程序员眼中，Java到处都是精美绝伦的指针。)

如果我们定义了不止一个引用指向同一个对象，那么这些引用是不相同的，因为引用也是一种数据类型，需要一定的内存空间来保存。但是它们的值是相同的，都指示同一个对象在内存的中地址。

我们看一下下面这个例子：

```java
	public static void tricky(Point arg1, Point arg2) {
		arg1.x = 100;
		arg1.y = 100;
		Point temp = arg1;
		arg1 = arg2;
		arg2 = temp;
	}

	public static void main(String[] args) throws ParseException {
		Point pnt1 = new Point(0, 0);
		Point pnt2 = new Point(0, 0);
		System.out.println("X: " + pnt1.x + " Y: " + pnt1.y);
		System.out.println("X: " + pnt2.x + " Y: " + pnt2.y);
		System.out.println("-------------------");
		tricky(pnt1, pnt2);
		System.out.println("X: " + pnt1.x + " Y:" + pnt1.y);
		System.out.println("X: " + pnt2.x + " Y: " + pnt2.y);
	}
```

从例子中可以看到tricky方法的功能是，首先改变了arg1的x，y坐标，然后将arg1赋给一个临时对象temp，然后利用temp交换arg1和arg2。在main方法中，声明两个Point对象，其x，y都是0。然后调用tricky方法，但pnt1与pnt2到底发生的什么变化。在看结果之前，大家可以自己想一想。

输出结果：

```
X: 0 Y: 0
X: 0 Y: 0
-------------------
X: 100 Y:100
X: 0 Y: 0
```

从输出结果中可以看到，tricky函数成功地改变了pnt1的值。但是pnt1和pnt2的置换失败了。这正是最令人困惑的地方。

下面来分析下这个方法到底是怎么执行的。

在main()函数当中，pnt1和pnt2仅仅是对象的引用，它的实际的值是pnt1和pin2在内存中的地址，这个地址指向的才是pnt1和pnt2在堆内存中真正存在的对象。

当向tricky函数传递参数时，与基本数据类型一样，它传递的是该对象在内存中地址的拷贝，而不是真实的对象实例。在内存中，pnt1与arg1同时指向new Point(0,0)这个对象。也就是说，当一个对象当做参数传递时，在内存中至少有两个‘引用’同时指向该对象。

在tricky函数内部，arg1.x = 100;arg1.y = 100;这个两句代码执行的操作是，找到arg1这个地址指向的对象Point，将对象Point的x、y更改为100。由于这个操作直接修改了改Point对象，也就是说如果其他‘引用’也指向该对象，也会随之改变。

而在这三句代码Point temp = arg1;arg1 = arg2;arg2 = temp;，这个过程其实是一个地址的交换过程，在上文基本数据类型传参的过程中可以发现，基本类型在方法内部的改变并不会影响参数源的值，所以对于pnt1它的值还是原来的地址，指向的对象还是原来的对象，pnt2也一样。而arg1与arg2在函数tricky中交换后，两个参数的值即它们保存的地址，确实互换了，也就是说arg1与pnt2指向同一个对象，arg2与pnt1指向同一个对象。

**总结一下，当对象作为参数传递时，如果方法内对该对象的成员数据进行操作，则会真正的改变该对象；如果对该对象的地址引用进行操作，则不会改变参数源。**

仔细看可以发现，方法参数与参数源是地址的拷贝，两者并没有太大关系，对象之所以发生改变是因为方法参数与参数源指向的是同一内存地址，数据在内存中改变了，指向这块内存的所有变量都会改变。但单纯的操作该地址值，并不会影响方法外部的参数源。

了解这部分内容后，大家可以把tricky方法中的执行顺序改变一下，先进行交换，然后在赋值，如下：

```java
	public static void tricky(Point arg1, Point arg2){
	    Point temp = arg1;
	    arg1 = arg2;
	    arg2 = temp;
	    arg1.x = 100;
	    arg1.y = 100;
	}
```

然后仍然使用原来的main方法调用，可以自己先分析一下运行过程，看自己分析的结果与程序运行的结果是否一致。

# 4、总结 #
从上面的分析我们可以得到一下结论：
1. 基本数据类型作为参数传递时，传递的值该对象值的拷贝。
2. 对象作为参数传递时，传递的是该对象的地址的拷贝。

两种传递方法，都属于值传递，所以Java的参数传递，只有值传递。