> Java类初始化时初始化块、静态初始化块和构造器之间的关系

从某种角度上来看，初始化块是构造器的补充，初始化块总是在构造器执行之前执行。与构造器不同的是，初始化块是一段固定执行的代码，不能接收任何参数。因此初始化块对于同一个类的所有对象所进行的初始化处理完全相同。基于这个原因，如果有一段初始化处理代码对所有对象都通用，且无需接收任何参数，则可以把它们从构造器中提取出来放在初始化块中。

如果定义初始化块时使用了static修饰符，则这个初始化块就变成了静态初始化块，负责对类进行初始化。静态初始化块是类相关的，系统将在类初始化阶段执行静态初始化块，而不是在创建对象时才执行。因此静态初始化块总是比普通初始化块先执行。

系统会在类初始化阶段上溯到最顶层的父类向下执行静态类初始化块，然后再上溯到顶层的父类执行初始化快，最后才是上溯到最顶层父类向下执行构造器。当类已经初始化完成后，这个类在该虚拟机中将一直存在。第二次新建该类的实例时，将不再执行静态初始化块，只执行初始化块和类构造器。

看下面这个例子：
```java
public class Main {
    public static void main(String[] args){
        new second();
        new second();
    }
}

class first{
	static{
		System.out.print("类first的静态初始化块\n");
	}
	{
		System.out.print("类first的初始化块\n");
	}
	public first(){
		System.out.print("类first的无参构造\n");
	}
}

class second extends first{
	static{
		System.out.print("类second的静态初始化块\n");
	}
	{
		System.out.print("类second的初始化块\n");
	}
	public second(){
		System.out.print("类second的无参构造\n");
	}
}
```
最后的输出结果是：
```java
类first的静态初始化块
类second的静态初始化块
类first的初始化块
类first的无参构造
类second的初始化块
类second的无参构造
类first的初始化块
类first的无参构造
类second的初始化块
类second的无参构造
```

实际上，初始化块是一个假象，使用javac编译Java类之后，所有的初始化块将消失，初始化块中的代码将会被“还原”到每个构造器中，且位于所有代码的前面。

静态初始化快和声明静态成员变量的执行有先后顺序。