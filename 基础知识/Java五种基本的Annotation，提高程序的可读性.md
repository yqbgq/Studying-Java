> 从JDK5开始，Java增加了对元数据的支持，也就是Annotation（即注解也被翻译为注释）。


这里的Annotation和普通的注释有一定的区别，它是代码中的特殊标记，这些标记可以在编译、类加载或者运行时被读取，并执行相应的处理。通过这样的注解，可以帮助开发人员在不改变原有的逻辑的情况下，在源文件中补充一些信息。而代码分析工具、开发工具和部署工具可以通过这些补充信息进行验证或者进行部署。

Annotation可以用来为程序元素（类、方法、成员变量等）设置元数据，值得一提的是，它不会影响代码的执行。

Java提供了5个基本Annotation的用法——使用Annotation时要在其前面增加@符号，并把该Annotation当成一个修饰符来使用。

5个基本的Annotation如下：

1. **@Override**
2. **@Deprecated**
3. **@SuppressWarnings**
4. **@SafeVarargs**
5. **@FunctionalInterface**

## @Override
@Override用来指定子类必须覆盖父类的方法
```java
public class Fruit {
	public void info(){
		System.out.println("Fruit");
	}
}
class Apple extends Fruit{
	@Override
	public void info(){
		System.out.println("Apple");
	}
}
```
编译上面的程序，可能丝毫看不出@Override的用处所在，因为它的作用是告诉编译器检查这个方法，保证父类要包含一个被该方法重写的方法，否则会编译出错。

使用@Override可以避免开发人员不小心将info方法写成了inf0方法，这样的低级错误可能会成为后期排错时的巨大障碍。

## @Deprecated
@Deprecated用来表示某个程序元素（类、方法等）已经过时，当其他程序使用已经过时的类或者方法时，编译器将会给出警告。

## @SuppressWarnings
@SuppressWarnings指示由它修饰的程序元素以及该元素中的所有子元素取消显示指定的编译器警告。
```java
@SuppressWarnings(value="all")
public class Fruit {
    public static void main(String[] args){
        Fruit.info();
    }
    @Deprecated
    public static void info(){
        System.out.print("Fruit");
    }
}
```
参考上面的代码，当使用了“@SuppressWarnings(value=”all”)”之后，编译器会取消使用了已经被弃用的方法的警告。value变量的值为你希望抑制的警告。

## “堆污染”警告与@SafeVarargs
```java
List ls = new ArrayList<Integer>();
ls.add(20);//添加元素时会引发unchecked警告
//下面的代码会引发“未经检验的转换”的警告，但是编译、运行时完全正常
List<String> list = ls;
//但是只要访问其中的元素，下面代码就会引起运行时异常
System.out.print(list.get(0));
```
Java将引发这种错误的原因称为“堆污染”。当把一个不带泛型的对象赋给一个带泛型的对象时，往往就会产生这种“堆污染”。

```java
class UnSafeVarargs
{
    static <T> T[] foo(T... args) {
        return args;
    }
    
    static <T> void bar(T... args) {
        for(T x : args) {
            System.out.print(x);
        }
    }
}
```
考虑上面这段程序，编译器会在方法定义处提示“Possible heap pollution from parameterized vararg type”。这是因为对于形参的个数可变且又是泛型时，当后续的代码依赖于传入的args中的每个元素的话，它是安全的。如果后续的代码依赖于args整体是T类型的数组的话，它将是不安全的。因为其中的元素可能是整型，也可能是字符串。程序会尝试将它转换成一个T类型的数组并且失败。

因此上面的两个方法中，前者是不安全的，后者是安全的。

在有些时候，开发者不希望看到这个“堆污染”的警告，那么就可以使用以下三种方式来抑制这个警告：

1. 使用@SafeVarargs修饰引发该警告的方法或者构造器
2. 使用@SuppressWarnings(“unchecked”)修饰。
3. 编译时使用-Xlint:varargs选项

显然第三种方式很少用到，通常选择前两种，尤其是第一种，因为它是专门为抑制“堆污染”警告而提供的。

## @FunctionalInterface

如果接口中只有一个抽象方法（可以包括多个默认方法或者多个static方法），该接口就是函数式接口。@FunctionalInterface就是用来指定某个接口必须是函数式接口的。

如果使用了@FunctionalInterface的接口不是函数式接口，那么编译就会出错。它和@Override一样，都是为了帮助程序员避免一些低级错误的。