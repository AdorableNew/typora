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
public static Integer valueOf(int i){
    if(i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i+(-InetgerCache.low)];
}
~~~

在 Java 8 中, Integer 缓存池的大小默认为 -128~127.

~~~java
static final int low = -128;
static final int high;
static final Integer cache[];

static{
    // high value may be configured by property
    int h = 127;
    String integerCacheHighPropValue = 
        sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
    if(integerCacheHighPropValue != null){
        try{
            int i = parseInt(integerCacheHighPropValue);
            i = Math.max(i,127);
            // Maximum array size is Integer.MAX_VALUE
            h = Math.min(i,Integer.MAX.VALUE - (-low) -1);
        }cache(NumberFormatException nfe){
            // If the property cannot be parsed into an int, ignore it.
        }
    }
    high = h;
    
    cache = new Integer[(high - low) +1];
    int j= low;
    for(int k = 0; k < cache.length; k ++){
        cache[k] = new Integer(j++);
    }
    
   	assert IntegerCache.high >=127;
}

~~~

编译器会再自动装箱过程调用valueOf()方法,因此多个值相同且值在缓存池范围内的 Integer 实例使用自动装箱来创建, 那么就会引用相同的对象.

~~~java
Integer m = 123;
Integer n = 123;
System.out.println(m==n);// true
~~~

基本类型对应的缓冲池如下:

* boolean values true and false
* all byte values
* short values between -128 and 127
* int values between -124 and 127
* char in range \u0000 to \u007F

在使用这些基本类型对应的包装类型时, 如果该数值范围在缓冲池范围内, 就可以直接使用缓冲池中的对象.

在 jdk 1.8 所有的数值类缓冲池中, Integer 的缓冲池 IntegerCache 很特殊,这个缓冲池的下界是 -128, 上界默认是127,但是这个上界是可调的, 在启动 jvm 的时候,通过 -XX:AutoBoxCacheMax= &lt;size&gt; 来指定这个缓冲池的大小,该选项在 JVM 初始化的时候会设定一个名为 java.lang.IntegerCache.high 系统属性, 然后 IntegerCache 初始化的时候就会读取该系统属性来决定上界.

# 二. String

## 概览

String 被声明为final, 因此它不可被继承.(Integer 等包装类也不可能被继承)

在 Java 8 中, String 内部使用 char 数组存储数据.

~~~java
public final class String
    implement java.io.Serializable,Comparable<String>, CharSequence{
    /** The value is used for character storage. */
    private final char value[];
}
~~~

在 Java 9 之后, String 类的实现改用 byte 数组存储字符串,同时使用 coder 来表示使用了哪种编码

~~~java
public final class String
    implement java.io.Serializable,Comparable<String>,CharSequence{
    /** The value is used for character storage. */
    pirvate final byte[] value;
    
    /** The identifier of the encoding used to encode the bytes in {@code value}. */
    private final byte coder;
}
~~~

value 数组被声明为 final, 这意味着 value 数组初始化之后就不能再引用其他数组. 并且 String 内部没有改变 value 数组的方法, 因此可以保证 String 不可变.

## 不可变的好处

**1. 可以缓存 hash 值 **

因为 String 的 hash 值经常被使用, 例如 String 用做 HashMap 的 key. 不可变的特性可以使得 hash 值也不可变 value 数组的方法, 因此只需要进行一次计算.

**2.String Pool 的需要**

如果一个 String 对象已经被创建过了, 那么就会从 String Pool 中取得引用. 只有 String 是不可变的, 才可能使用 String Pool.

![img](https://camo.githubusercontent.com/152a310f7698bb23ae201081c5c497b5c1d396d9/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f696d6167652d32303139313231303030343133323839342e706e67)

**3.安全性**

String 经常作为参数, String 不可变性可以保证参数不可变. 例如在作为网络连接参数的情况下 如果 String 是可变的, 那么在网络连接过程中, String 被改变, 改变 String 的那一方以为现在连接的是其他主机, 而实际情况却不一定是.

**4.线程安全**

String 不可变性天生具备线程安全, 可以在多个线程中安全的使用.

## String, StringBuffer and StringBuilder

**1.可变性**

* String 不可变
* StringBuffer 和 StringBuilder 可变

**2.线程安全**

* String 不可变, 因此是线程安全的
* StringBuilder 不是线程安全的
* StringBuffer 是线程安全的 ,内部使用 synchronize 进行同步

## String Pool

字符串常量池 (String Pool) 保存着所有字符串字面量 (literal  strings) , 这些字面量在编译时期就确定. 不仅如此, 还可以使用 String 的 intern() 方法在运行过程将字符串添加到 String Pool 中.

当一个字符串调用 intern() 方法时, 如果