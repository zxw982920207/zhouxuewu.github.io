---
title: Java 形参与实参
date: 2017-9-30
tags: Java
---
前几天在头条上看到一道经典面试题,引发了一些思考。也是写这篇文章的导火索。
### 背景
请看题:
```java
public	class Main {
    public static void main(String[] args) {
        Integer a = 1;
        Integer b = 2;
        System.out.println("a=" + a + ",b=" + b);
        swap(a, b);
        System.out.println("a=" + a + ",b=" + b);
    }

    private static void swap(Integer numa, Integer numb) {
        //请实现
    }
}
```
看到这个题后 瞬间觉得有坑。也觉得为什么要书写一个`swap`方法呢？如下实现不是更简单:
```java
public static void main(String[] args) {
        Integer a = 1; 
        Integer b = 2;
        System.out.println("a=" + a + ",b=" + b);
        Integer tmp = a;
        a = b;
        b = tmp;
        System.out.println("a=" + a + ",b=" + b);
    }
```
输出:
```java
a=1,b=2
a=2,b=1
```
完美实现交换。但是请注意，这是一道面试题，要的就是考验一些知识点。所以还是老老实实的实现`swap`方法吧。
有的同学可能会想，`Integer` 是一个包装类型,是对Int的装箱和拆箱操作。其实也是一个对象。既然是对象，直接更改对象的引用不就行了？  
思路没问题，我们首先看看实现:   
```java
 private static void swap(Integer numa, Integer numb) {
        Integer tmp = numa;
        numa = numb;
        numb = tmp;
        System.out.println("numa=" + numa + ",numb=" + numb);
 }
```
输出: 
```java
a=1,b=2
numa=2,numb=1
a=1,b=2
```
不出意外,没有成功  
这是什么原因呢？
技术老手一看就知道问题出在[形参和实参](https://www.baidu.com/s?wd=java%E5%BD%A2%E5%8F%82%E5%92%8C%E5%AE%9E%E5%8F%82%E7%9A%84%E5%8C%BA%E5%88%AB)
混淆了
### JAVA的形参和实参的区别:
**形参** 顾名思义:就是形式参数，用于定义方法的时候使用的参数，是用来接收调用者传递的参数的。
	形参只有在方法被调用的时候，虚拟机才会分配内存单元，在方法调用结束之后便会释放所分配的内存单元。
	因此,形参只在方法内部有效，所以针对引用对象的改动也无法影响到方法外。
	
**实参** 顾名思义:就是实际参数，用于调用时传递给方法的参数。实参在传递给别的方法之前是要被预先赋值的。
	在本例中 swap 方法 的numa, numb 就是形参，传递给 swap 方法的 a,b 就是实参
	
注意:  
在`值传递`调用过程中，只能把实参传递给形参，而不能把形参的值反向作用到实参上。在函数调用过程中，形参的值发生改变，而实参的值不会发生改变。  
而在`引用传递`调用的机制中，实际上是将实参引用的地址传递给了形参，所以任何发生在形参上的改变也会发生在实参变量上。  
那么问题来了，什么是`值传递`和`引用传递`
### 值传递和引用传递
在谈`值传递`和`引用传递`之前先了解下 Java的数据类型有哪些
#### JAVA的数据类型
Java 中的数据类型分为两大类，`基本类型`和`对象类型`。相应的，变量也有两种类型：`基本类型`和`引用类型`
`基本类型`的变量保存`原始值`，即它代表的值就是数值本身,`原始值`一般对应在内存上的`栈区`  
而`引用类型`的变量保存`引用值`，`引用值`指向内存空间的地址。代表了某个对象的引用，而不是对象本身。对象本身存放在这个引用值所表示的地址的位置。`被引用的对象`对应内存上的`堆内存区`。  
基本类型包括：`byte`,`short`,`int`,`long`,`char`,`float`,`double`,`boolean` 这八大基本数据类型
引用类型包括：`类类型`，`接口类型`和`数组`  
#### 变量的基本类型和引用类型的区别
基本数据类型在声明时系统就给它分配空间
```java
	int a;//虽然没有赋值，但声明的时候虚拟机就会 分配 4字节 的内存区域,而引用数据类型不同，它声明时只给变量分配了引用空间，而不分配数据空间:	
	String str;//声明的时候没有分配数据空间，只有 4byte 的引用大小，在栈区，而在堆内存区域没有任何分配
	str.length(); //这个操作就会报错，因为堆内存上还没有分配内存区域，而 a = 1; 这个操作就不会报错。```
好了，Java的数据类型说完了，继续我们的`值传递`和`引用传递`的话题。
先背住一个概念:`基本类型`的变量是`值传递`；`引用类型`的变量
结合前面说的 `形参`和`实参`。
#### 值传递
方法调用时，实际参数把它的值传递给对应的形式参数，函数接收的是原始值的一个copy，
此时内存中存在两个相等的基本类型，即实际参数和形式参数，后面方法中的操作都是对形参这个值的修改，不影响实际参数的值
####  引用传递
也称为`地址传递`，`址传递`。方法调用时，实际参数的引用(地址，而不是参数的值)被传递给方法中相对应的形式参数，函数接收的是原始值的内存地址
在方法执行中，形参和实参内容相同，指向同一块内存地址，方法执行中对引用的操作将会影响到实际对象
通过例子来说话:
```java
 static class Person {
        int age;
        Person(int age) {
            this.age = age;
        }
    }
    
    private static void test() {
        int a = 100;
        testValueT(a);
        System.out.println("a=" + a);
        Person person = new Person(20);
        testReference(person);
        System.out.println("person.age=" + person.age);
    }
    
    private static void testValueT(int a) {
        a = 200;
        System.out.println("int testValueT a=" + a);
    }
    
    private static void testReference(Person person) {
        person.age = 10;
    }
```
输出：
```java
int testValueT a=200
a=100
person.age=10
```
看见 `值传递` a的值并没有改变，而 `引用传递`的 persion.age已经改变了
有人说
```java
private static void testReference(Person person) {
        person = new Person(100);
}
```
为什么 输出的 person.age 还是20呢？  
我想说 了解一下什么是`引用类型`吧？ 方法内把 `形参`的地址引用换成了另一个对象，并没有改变这个对象,并不能影响 外边`实参`还引用原来的对象，因为 形参只在方法内有效哦。

有人或许还有疑问，按照文章开头的例子，`Integer`也是 `引用类型`该当如何呢？
其实 类似的 `String`,`Integer`,`Float`,`Double`,`Short`,`Byte`,`Long`,`Character`等等基本包装类型类。因为他们本身没有提供方法去改变内部的值，例如`Integer` 内部有一个`value` 来记录`int`基本类型的值，但是没有提供修改它的方法，而且 也是`final`类型的，无法通过`常规手段`更改。  
所以虽然他们是`引用类型`的，但是我们可以认为它是`值传递`,这个也只是`认为`,事实上还是`引用传递`,`址传递`。

----------------

好了，基础知识补充完毕，然我们回到面试题吧

-----------------
__回归正题__

```java
private static void swap(Integer numa, Integer numb) {
        Integer tmp = numa;
        numa = numb;
        numb = tmp;
        System.out.println("numa=" + numa + ",numb=" + numb);
 }
```
通过补习基础知识，我们很明显知道 上面这个方法实现替换 是不可行的。因为`Interger`虽然是`引用类型`  
但是上述操作只是改变了`形参`的引用，而没有改变`实参`对应的`对象`。
  
那么思路来了，我们`通过特殊手段`改变 `Integer`内部的`value`属性
```java
private static void swap(Integer numa, Integer numb) {
        Integer tmp = numa;
        try {
            Field field = Integer.class.getDeclaredField("value");
            field.setAccessible(true);
            field.set(numa, numb);//成功的将numa 引用的 1的对象 值改为 2
            field.set(numb, tmp); //由于 tmp 也是指向 numa 未改变前指向的堆 即对象1 ，经过前一步，已经将对象1的值改为了2，自然 numb 也是2，所以改动失效
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

```
输出结果:
 ```java
 a=1,b=2
 a=2,b=2
 ```
[又来疑问了]()？为何 `a`的值改变成功，而`b`的改变失败呢？

*见代码注释* 
所以其实 `field.set(numb, tmp);` 是更改成功的，只是 tmp 经过前一行代码的执行，已经变成了 2。
那么如何破呢？
我们有了一个思路，既然是 `tmp`的引用的对象值变量，那么我让`tmp`不引用 `numa`了
```java
 private static void swap(Integer numa, Integer numb) {
        int tmp = numa.intValue();//tmp 定义为基本数据类型
        try {
            Field field = Integer.class.getDeclaredField("value");
            field.setAccessible(true);
            field.set(numa, numb);//这个时候并不改变 tmp 的值
            field.set(numb, tmp);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```
这种情况下 对 `numa` 这个对象的修改就不会导致 `tmp` 的值变化了,看一下运行结果

	a=1,b=2
	a=2,b=2
	
这是为啥？有没有`快疯`啦？
难道我们的思路错了？
先别着急，我们看看这个例子：
仅仅是将前面的例子 `a`的值改为 129，`b`的值改为130

```java
public static void main(String[] args) {
        Integer a = 129;
        Integer b = 130;

        System.out.println("a=" + a + ",b=" + b);
        swap(a, b);
        System.out.println("a=" + a + ",b=" + b);
    }

    private static void swap(Integer numa, Integer numb) {
        int tmp = numa.intValue();
        try {
            Field field = Integer.class.getDeclaredField("value");
            field.setAccessible(true);
            field.set(numa, numb);
            field.set(numb, tmp);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```
运行结果:
```java
a=129,b=130
a=130,b=129
```
有没有`怀疑人生`？我们的思路没有问题啊?为什么 换个数值就行了呢？
我们稍微修改一下程序
```java
public static void main(String[] args) {
        Integer a = new Integer(1);
        Integer b = new Integer(2);

        System.out.println("a=" + a + ",b=" + b);
        swap(a, b);
        System.out.println("a=" + a + ",b=" + b);
    }

    private static void swap(Integer numa, Integer numb) {
        int tmp = numa.intValue();
        try {
            Field field = Integer.class.getDeclaredField("value");
            field.setAccessible(true);
            field.set(numa, numb);
            field.set(numb, tmp);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```
 运行结果:
```java
 	a=1,b=2
 	a=2,b=1
```
_哎？为啥 1 和 2 也可以了?_  
_我们这时肯定猜想和`Integer`的装箱 拆箱有关_
### 装箱，拆箱 概念
#### Integer的装箱操作
为什么 `Integer a = 1` 和 `Integer a = new Integer(1)` 效果不一样
那就瞅瞅源码吧？

```java
    public Integer(int value) {
        this.value = value;
    }
    
    /**
     * Returns an {@code Integer} instance representing the specified
     * {@code int} value.  If a new {@code Integer} instance is not
     * required, this method should generally be used in preference to
     * the constructor {@link #Integer(int)}, as this method is likely
     * to yield significantly better space and time performance by
     * caching frequently requested values.
     *
     * This method will always cache values in the range -128 to 127,
     * inclusive, and may cache other values outside of this range.
     *
     * @param  i an {@code int} value.
     * @return an {@code Integer} instance representing {@code i}.
     * @since  1.5
     */
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }   
```
通过注释知道，java推荐 `Integer.valueOf` 方式初始化一个`Interger`因为有 缓存了`-128 - 127`的数字
我们直接定义 `Integer a = 1` 具有这个功能，所以 Jvm 底层实现 是通过 `Integer.valueOf`这个方法
再看 `field.set(numb, tmp);` 
我们打断点，发现通过反射设置 `value`时 竟然走了 `Integer.valueOf` 方法 
下面是 我们调用 `swap`前后的 `IntegerCache.cache` 值得变化
##### 反射修改前:  
![](/img/java_before_change.jpg "")  
##### 反射修改后  
![](/img/java_after_chang.jpg "")  
在反射修改前

```java
IntegerCache.cache[128]=0
IntegerCache.cache[129]=1
IntegerCache.cache[130]=2
```
通过反射修改后

```java
IntegerCache.cache[128]=0
IntegerCache.cache[129]=2
IntegerCache.cache[130]=2
```

再调用 `field.set(numb, tmp)` tmp这时等于1 对应的 角标 129 ,但是这个值已经变成了2
所以出现了刚才 `奇怪的结果`
原来都是`缓存的锅`
下面趁机再看个例子 加深理解

```java
Integer testA = 1;
Integer testB = 1;

Integer testC = 128;
Integer testD = 128;
System.out.println("testA=testB " + (testA == testB) + ",\ntestC=testD " + (testC == testD));	
```
 输出结果:
 
 ```java
 testA=testB true,
 testC=testD false
 ```
通过这小示例，在 -128 到 127的数字都走了缓存，这样 `testA` 和 `testB`引用的是同一片内存区域的同一个对象。
而 `testC ` `testD ` 数值大于127 所以 没有走缓存，相当于两个`Integer`对象，在堆内存区域有两个对象。
两个对象自如不相等。  
在前面的示例中 我们 通过
```java
Integer a = new Integer(1);
Integer b = new Integer(2);
```
方式初始化 `a`,`b` 我们的交换算法没有问题，也是这个原因。

##### 那么到目前为止我们的`swap` 方法可以完善啦
```java
private static void swap(Integer numa, Integer numb) {
        int tmp = numa.intValue();
        try {
            Field field = Integer.class.getDeclaredField("value");
            field.setAccessible(true);
            field.set(numa, numb);
            field.set(numb, new Integer(tmp));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```
只需将之前的 `field.set(numb, tmp)` 改为 `field.set(numb, new Integer(tmp))`

到此, 这个面试我们已经通过了，还有一个疑问我没有解答。
为什么 `field.set(numb, tmp)` 会执行 `Integer.valueOf()` 而 `field.set(numb, new Integer(tmp))` 不会执行。
这就是`Integer的装箱`操作，当 给 `Integer.value` 赋值 `int`时，JVM 检测到 `int不是Integer类型`,需要装箱，才执行了`Integer.valueOf()`方法。而`field.set(numb, new Integer(tmp))` 设置的 是Integer类型了，就不会再拆箱后再装箱。
### Over Thanks

> 注：（转载）[DailyCast博客-Java形参与实参](https://dailycast.github.io/Java-形参与实参)