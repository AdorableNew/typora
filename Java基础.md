# 一.数据类型

## 基本类型
- byte/8

- char/16

- short/16

- int/32

- float/32

- long/64

- double/64

- boolean/~

boolean 只有两个值 :  true、false，可以使用 1 bit 来存储，具体大小没有明确规定。JVM 会再编译时期将 boolean 类型的数据转换为 int，使用 1 来表示 true，0 表示 false。JVM 支持 boolean 数组，但是是通过读写 byte 数组来实现的。

## 包装类型

基本类型都有对应的包装类型，基本类型与其他对应的包装类型之间的赋值使用自动装箱与拆箱完成。

~~~java
Integer x = 2;   //装箱 调用了 Integer.valueOf(2)
int y = x;       //拆箱 调用了 X.intValue()
~~~

## 缓存池

new Integer(123) 与 Integer.valueOf(123) 的区别在于：

- new Integer(123) 每次都会新建一个对象

- Integer.valueOf(123) 会使用缓存池中的对象，多次调用会取得同一个对象的引用。
~~~java
Integer x = new Integer(123);
Integer y = new Integer(123);
System.out.print(x==y); //false
Integer z = Integer.valueOf(123);
Integer k = Integer.valueOf(123);
System.out.print(z==k); //true
~~~

  valueOf() 方法的实现比较简单，就是先判断值是否在缓存池中，如果在的话就直接返回缓存池的内容。

~~~java

~~~

