> 因为String对象是不可变对象，因此，在需要对字符串进行修改操作时，String对象总是会产生新的对象，所以性能比较差

## 1.String常量的累加操作
在程序开发中，我们往往无法预知字符串的实际取值，因此这就需要在程序运行过程中通过拼接的方法，动态生成字符串。

因为String对象具有不变性，所以一旦String对象实例生成，就不可能再被改变。因此下面的代码会生成多个String对象：
```java
String result = "welcom "+"to "+"AmosH's blog";
```

这段代码会依次生成“welcome to ”、“welcom to AmosH’s blog”两个对象，所以从理论上来说，这段代码的效率并不高。

如果我们使用StringBuilder来动态构造字符串对象，则需要使用类似如下的代码：

```java
StringBuilder sb = new StringBuilder();
sb.append("welcome ");
sb.append("to");
sb.append("AmosH's blog");
```
但是将这两段代码各自循环五万次，前者耗时0 MS，后者耗时10 MS。

这是因为JVM在编译时对代码进行了充分的优化，对于这些在编译时就能确定取值的字符串操作，在编译时就进行了计算。

因此第一段代码并没有如预期般生成多个对象，而是指向了一个字符串常量。而后者的所有append()方法都将会在运行时如实调用，因此导致了第一段代码更快的结果。

## 2.String变量的累加操作
如果在编译时无法确定字符串的取值，那么对这些变量字符串的累加,JVM又会作何处理呢？
```java
String str1 = "welcom ";
String str2 = "to ";
String str3 = "AmosH's blog";
String result = str1+str2+str3;
```
将上面这段代码片段运行5万次，平均耗时10 MS，这个性能和StringBuilder的性能几乎一样，因为在在编译时，Java会将最后一行代码修改为：
```java
String result = (new StringBuilder(String.valueOf(str1))).append(str2).append(str3).toString();
```
可以看出，在Java编译时，会对字符串处理进行一定的优化，这会使得一些看起来很慢的代码，实际运行起来并不慢。

## 3.构建超大的String对象
综上所述，在代码实现中事先对String对象做的累加操作会在编译时被优化，因此其性能比理论值好得多。但是，在实际的代码实现中，依然推荐显式的使用StringBuilder或者StringBuffer对象来提升程序性能，而不是依靠编译器对程序进行优化。

我们试图构建一个超大的String对象，采用下面三种方法，并重复一万次循环：

**方法一：**
```java
        for(int i=0;i<10000;i++){
            str = str + String.valueOf(i);
        }
```

**方法二：**
```java
        for(int i=0;i<10000;i++){
            str = str.concat(String.valueOf(i));
        }
```
**方法三：**
```java
        StringBuilder sb = new StringBuilder();
        for(int i=0;i<10000;i++){
            sb=sb.append(String.valueOf(i));
        }
```

以上三个代码段都进行了长字符串的连接操作。在同等条件下，方法一耗时387 MS，方法二耗时82 MS，方法三耗时0 MS。

尽管在方法一中，编译器会对其进行优化，但是比那一起会在每次循环中都生成新的StringBuilder对象，从而大大降低了系统的性能。

## 4.StringBuffer和StringBuilder的选择
StringBuilder和StringBuffer都实现了一样的抽象类，拥有几乎相同的对外接口，两者的最大不同在于StringBuffer对几乎所有方法都做了同步，而StringBuilder没有。因此，StringBuilder的性能更好，但是在多线程环境中，使用StringBuffer可以保证线程同步。

无论是StringBuffer还是StringBuilder，在初始化时都可以设置一个容量参数，对应的构造函数如下：
```java
public StringBuilder(int capacity)
public StringBuffer(int capacity)
```
在不指定容量参数时，默认是16个字节。在追加字符串时，如果需要容量超过实际char数组长度，则需要进行扩容。

扩容的策略是将原有的容量翻倍，以新的容量申请内存空间，建立新的char数组，然后将原数组中的内容复制到这个新的数组中。

因此，如果能够预先评估StringBuilder或者StringBuffer的大小，那么将能够有效节省这些操作从而提升系统性能。