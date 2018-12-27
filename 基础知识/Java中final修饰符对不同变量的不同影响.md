> final修饰符可以用来修饰类、方法和变量，用于表示它修饰的类、方法和变量不可改变。final修饰变量时，表示该变量一旦获得了初始值就不可被改变。

由于final变量获取初始值之后就不能重新赋值，所以final修饰成员变量和局部变量时有一定程度的不同。


## final成员变量

成员变量是随类初始化或者对象初始化而初始化的。在初始化时，系统会为类变量或者实例变量分配内存并分配默认值。

对于final修饰的成员变量而言，一旦有了初始值，就不能再被重新赋值。因此，如果没有在定义成员变量时指定初始值，也没有在初始化块、构造器中为成员变量指定初值，那么这些成员变量将一直是系统默认分配的0、’\u0000’、false或者null。这是没有意义的，所以Java语法规定：final修饰的成员变量必须由程序员显示地指定初始值。

## final局部变量

系统不会为局部变量进行初始化，局部变量必须由程序员显示初始化。因此，使用final修饰局部变量时，既可以在定义时指定默认值，也可以不指定默认值。

注意这段代码是有语法错误的：
```java
public class Main {
	public void test(final int a){
		a = 5;//语法错误
	}
    public static void main(String[] args){
    	final int a = 4;
    	Main main = new Main();
    	main.test(a);
    }
}
```
因为形参在传入方式时，会根据系统传入的参数来完成初始化，因此使用final修饰的形参不能被赋值。

## final 修饰的基本类型变量和引用类型变量的区别

当使用final修饰基本类型变量时，不能对基本类型变量重新赋值，因此基本类型变量不能被改变。但是对于引用类型变量而言，他保存的仅仅是一个引用，final只保证这个引用类型变量所引用的地址不会改变，即一直引用一个对象，但这个对象可以被改变。

考虑下面这个示例：
```java
public class Main {
    public static void main(String[] args){
    	final int[] a ;
    	final int b;
    	int[] c = {1,2,3,4};
    	int d = 10;
    	a = c;
    	b = d;
    	c[2] = 10;
    	System.out.println(a[2]);
    	//output 10
    	d = 1;
    	System.out.println(b);
    	//output 10
    }
}
```
## 可以执行“宏替换”的final变量

对于一个final变量来说，只要满足三个条件，这个final变量就不再是一个变量，而是相当于一个直接量。

1. 使用final修饰符修饰
2. 在定义该final变量时指定了初始值
3. 该初始值可以在编译时就被确认下来。

考虑下面的代码演示：
```java
public class Main {
    public static void main(String[] args){
    	final String a="Amos ",b="H";
    	final String c = a+b;
    	final String d = "Amos H";
    	String e="Amos ",f="H";
    	final String g=e+f;
    	System.out.println(c==d);
    	//output true
    	System.out.println(c==g);
    	//output false
    }
}
```
因为变量c和变量d的值在编译的时候就已经确定了，所以它们同时指向常量池中的Amos H 。因此输出true。而对于变量g来说，它在定义的时候系统无法确认它的值，所以自然不能指向常量池。因此自然输出false。

如果希望g也指向常量值，只要将e,f也定义为final变量即可。
