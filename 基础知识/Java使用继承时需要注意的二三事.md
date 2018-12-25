>继承是实现类复用的重要手段，它能够有效减少重复代码的数量，但同时也带来一个最大的坏处：破坏封装。

子类拓展父类时，子类从父类继承得到了成员变量和方法，如果访问权限允许，子类可以直接访问父类的成员变量和方法，相当于子类可以直接复用父类的成员变量和方法，十分便利。

但是在继承关系中，子类可以直接访问父类的内部信息和方法，导致了父类和子类的严重耦合。从这个角度看，父类的实现细节对子类不再透明，子类可以访问父类的成员变量和方法，可以随意改变父类的实现细节（比如通过方法重写来改变父类的实现方法），从而导致子类可以**恶意篡改父类的方法**。

示例如下：
```java
public class Main {
    public static void main(String[] args){
        person p = new student();
        p.talk();//最终输出I'm a student
    }
}
class person{
	public void talk(){
		System.out.print("I'm a person");
	}
}

class student extends person{
	public void talk(){
		System.out.print("I'm a student");
	}
}
```

可以看到，尽管上面申明的是person引用变量，但是实际上引用的是一个student对象。所以在调用p的talk()方法时执行的不再是person的talk()方法，而是student的talk()方法。

为了保证良好的封装性，不会被子类随意更改，设计父类时应该遵循如下规则：
1. **尽量隐藏父类的内部数据**。把父类的所有成员变量设置为private访问类型，不要让子类直接访问父类的成员变量。
2. **不要让子类随意访问、修改父类的方法。**父类中的那些仅为辅助其他的工具方法应该使用private访问控制符修饰。如果父类中的方法需要被外部类调用，但又不希望子类重写，则要使用final修饰符来修饰该方法。如果希望该方法被子类重写但不被外部类调用，则应该使用protected来修饰该方法。
3. **尽量不要在父类构造器调用要被子类重写的方法**。

关于最后一点，考虑以下示例：
```java
public class Main {
    public static void main(String[] args){
        student p = new student();//输出子类重写父类方法子类重写父类方法
    }
}
class person{
	public void talk(){
		System.out.print("父类方法");
	}
	public person(){
		talk();
	}
}

class student extends person{
	public void talk(){
		System.out.print("子类重写父类方法");
	}
	public student(){
		talk();
	}
}
```
当系统尝试创建student对象时，会执行其父类构造器，如果父类构造器调用了被其子类重写的方法，则变成调用被子类重写的方法。

如果想把某些类设置为最终类，即不能被当成父类，则可以使用final修饰这个类。除此之外，使用private修饰这个类的所有构造器也可以使得子类无法调用该类的构造器，也就无法继承该类。对于把所有构造器使用private修饰的父类而言，可以另外提供一个静态方法，用于创建该类实例。

**那么究竟在什么时候需要从父类派生新的子类呢**？不仅需要保证子类是一种特殊的父类，还要具备以下两个条件之一：

* 子类需要增加额外的属性，而不仅仅是属性值的改变。
* 子类需要增加自己的独特行为（包括增加新的方法或者重写父类方法）