>Java中“==”运算符、equals()方法和常量池的关联

Java中测试两个变量是否相等可以使用两种方式：一种是利用“==”运算符，一种是使用equals()方法。

**使用“\=\=”运算符来判断两个变量是否相等时，如果两个变量是基本类型变量，且都是数值类型（不一定要求数据类型严格相等），则只要两个变量的值相等，则返回true。对于两个引用变量来说，只有当它们指向同一个对象时，“==”运算符才返回true，且该操作符不可以用于比较类型上没有父子关系的两个对象**。

在很多情况下，我们希望程序只判断两个引用对象的值是否相等，这种比较规则并不严格要求两个引用变量指向同一个对象。

下面程序演示了使用==运算符和equals()方法判断两个变量是否相等的过程：

```java
public class Main {
    public static void main(String[] args){
    	int a = 65;
    	float b = 65.0f;
    	System.out.println("a 是否和 b相等"+ (a==b));
    	//output a 是否和 b相等true
    	String s1 = "Amos";
    	String s2 = "Amos";
    	System.out.println("s1 是否和 s2相等"+ (s1==s2));
    	//output s1 是否和 s2相等true
    	String s3 = new String("Amos");
    	String s4 = new String("Amos");
    	System.out.println("s3 是否和 s4相等"+ (s3==s4));
    	//output s3 是否和 s4相等false
    	System.out.println("s3 是否和 s4相等"+ (s3.equals(s4)));
    	//output s3 是否和 s4相等true
    }
}
```

可以看到当使用“==”操作符比较s3和s4时，答案是false，但是使用equals()方法进行比较时，答案是true。这就是引用变量相等和值相等的区别。

对于新手而言，String还有一个非常容易迷惑的地方：“Amos”和new  String(“Amos”)有什么区别？

1. 当Java程序直接使用字符串的直接量（包括可以在编译时就计算出来的字符串值）时，JVM会使用常量池来管理这些字符串。
2. 当使用new  String(“Amos”)时，JVM会先使用常量池来管理“Amos”直接量，然后再调用String类的类构造器来创建一个新的String对象，新创建的对象被保存在堆内存中。换句话说，new  String(“Amos”)一共产生了两个字符串对象。
    
常量池专门用于管理在编译时就已经确定的并被保存在已编译的.class文件中的一些数据。它还包含了关于类、方法、接口中的常量，也包括了字符串常量。

下面这个例子演示了JVM使用常量池管理字符串直接量的情形。
```java
public class Main {
    public static void main(String[] args){
    	String s1 = "AmosH's blog";
    	//在常量池中保存了AmosH's blog直接量
    	String s2 = "AmosH's ";
    	String s3 = "blog";
    	String s4 = "AmosH's " + "blog";
    	//s4在编译时已经确定下来，引用常量池中的直接量
    	String s5 = "AmosH" + "'s" + " " + "blog";
    	//s5在编译时已经确定下来，引用常量池中的直接量
    	String s6 = s2 + s3;
    	//s6在编译时无法确定，不能引用常量池中的直接量
    	String s7 = new String("AmosH's blog");
    	//s7使用构造器创建了一个新的String对象
    	System.out.println("is s1 == s4?" + (s1 == s4));
    	//output is s1 == s4?true
    	System.out.println("is s1 == s5?" + (s1 == s5));
    	//output is s1 == s5?true
    	System.out.println("is s1 == s6?" + (s1 == s6));
    	//output is s1 == s6?false
    	System.out.println("is s1 == s7?" + (s1 == s7));
    	//output is s1 == s7?false
    }
}
```
**equals()方法是Object类提供的一个实例方法，因此所有的引用变量都可以调用该方法来判断是否与其他引用变量相等。但是使用这个方法判断两个对象相等的标准与使用==运算符没有什么区别，同样要求两个引用变量指向同一个对象时才会返回true**。

因此，使用Object类提供的equals()方法并没有什么太大的实际意义，如果希望采用自定义的相等标准，则可以通过重写equals()方法来实现。

通常而言，正确的重写equals()方法应该满足以下条件：

1. **自反性**：对任意的x.equals(x)一定返回true
2. **对称性**：对任意的x和y，如果x.equals(y)返回true，则y.equals(x)返回true
3. **传递性**：对任意的x、y和z。如果x.equals(y)返回true，y.equals(z)返回true，则x.equals(z)一定返回true
4. **一致性**：对任意的x和y，如果对象中用于等价比较的信息没有改变，那么无论调用x.equals(y)多少次，返回的结果应该一直保持一致。
5. **对任何不是null的x，x.equals(null)一定返回false**
    