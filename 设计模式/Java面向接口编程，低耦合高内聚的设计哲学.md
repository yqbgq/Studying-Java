> 接口体现的是一种规范和实现分离的设计哲学，充分利用接口可以极大的降低程序中各个模块之间的耦合，提高系统的可维护性以及可扩展性。

因此，很多的软件架构设计理念都倡导“面向接口编程”而不是面向实现类编程，以期通过这种方式来降低程序的耦合。

但是在讨论这些之前，我们先要搞清楚一个问题：

**接口还是抽象类？**

为什么会有这个问题，因为在某些情况下，接口和抽象类很像：

* 接口和抽象类都不能被实例化，它们都位于继承树的顶端，用于被其它类实现或者继承。
* 接口和抽象类都可以包含抽象方法，实现接口或者继承抽象类的普通子类都必须实现这些抽象方法。

接口作为系统和外界交互的窗口，体现的是一种规范。规定了实现者必须向外提供哪些服务，调用者可以调用哪些服务。当在一个程序中使用接口时，接口是多个模块之间的耦合标准；当在多个应用程序之间使用接口时，接口是多个程序之间的通信标准。

而抽象类则不同，它作为多个子类的共同父类，体现的是一种模板式设计。它可以作为系统实现的中间产品，但是必须要进行完善才能成为最终产品。

除此之外，接口和抽象类在用法上也存在差别：

* 接口只能包含抽象方法和默认方法，不能为普通方法提供方法实现；抽象类则可以包含普通方法。
* 接口中不能定义静态方法；抽象类可以。
* 接口不能定义普通成员变量；抽象类可以。
* 接口不包含构造器；抽象类可以包含构造器，但并不是用于创建对象，而是用于让其他子类调用这些构造器来完成属于抽象类的初始化操作。
* 一个类只能有一个直接父类，包括抽象类；但一个类可以直接实现多个接口，通过实现多个接口可以弥补单继承的不足。

下面介绍两种常见的应用场景来示范面向接口编程的优势。

## 简单的工厂模式

有这样一个场景：程序中的每一个Computer类都需要集成一个Printer类，以完成设备的输入输出，示例如下：
```java
public class Computer{
     public static void main(String []args){
        Printer p = new Printer();
        p.getMsg("Amos");
        p.getMsg("H's blog");
        p.printMsg();
     }
}

class Printer{
    private String msg = "";
    public void getMsg(String message){
        msg += message;
    }
    public void printMsg(){
        System.out.println(msg);
    }
}
```
但是，当我们在重构代码的时候，需要使用BetterPrinter类以替换Printer类时，如果只有一个类集成了Printer那么还好，但是如果有成百上千个类集成了这个类呢？那么工作量将会是巨大的！

为了解决这个问题，我们可以使用接口实现工厂模式，让Computer类集成一个PrinterFactory类，将Computer类和Printer类的实现完全分隔开来。当从Printer类切换到BetterPrinter类时对系统完全没有影响。

示例如下：
```java
class Computer{
    private Out p;
    public Computer(Out printer){
        this.p = printer;
    }
    public void keyIn(String msg){
        p.getMsg(msg);
    }
    public void printMsg(){
        p.printMsg();
    }
}

class Printer implements Out{
    private String msg = "";
    public void getMsg(String message){
        msg += message;
    }
    public void printMsg(){
        System.out.println(msg);
    }
}

interface Out{
    String mes = "";
    void getMsg(String message);
    void printMsg();
}

public class PrinterFactory{
    public Out out(){
        return new Printer();
    }
    public static void main(String[] args){
        PrinterFactory pf = new PrinterFactory();
        Computer c = new Computer(pf.out());
        c.keyIn("Amos");
        c.keyIn("H's blog");
        c.printMsg();
        //output AmosH's blog
    }
}
```
该实现使用工厂类用来生产Printer对象实例，而Computer类则只接收工厂类返回的对象用以初始化，分隔了和Printer对象实现之间的耦合。当系统需要使用BetterPrinter类替换Printer类时，只要BetterPrinter类实现了Out接口，然后在PrinterFactory工厂类中修改产生的对象即可。

下面是替换示例：
```java
class Computer{
    private Out p;
    public Computer(Out printer){
        this.p = printer;
    }
    public void keyIn(String msg){
        p.getMsg(msg);
    }
    public void printMsg(){
        p.printMsg();
    }
}

class BetterPrinter implements Out{
    //全新的打印类，限制只能存储5次输入
    private String msg = "";
    private int count = 5;
    public void getMsg(String message){
        msg += message;
        count--;
        if(count==0){
            printMsg();
            count = 5;
        }
    }
    public void printMsg(){
        System.out.println(msg);
        msg="";
        count = 5;
    }
}

interface Out{
    String mes = "";
    void getMsg(String message);
    void printMsg();
}

public class PrinterFactory{
    public Out out(){
        return new BetterPrinter();
    }
    public static void main(String[] args){
        PrinterFactory pf = new PrinterFactory();
        Computer c = new Computer(pf.out());
        c.keyIn("Amos");
        c.keyIn("H's ");
        c.keyIn("blog ");
        c.keyIn("Wel");
        c.keyIn("come");
        //output AmosH's blog Welcome
        c.keyIn("Amos");
        c.keyIn("H's ");
        c.keyIn("blog ");
        c.keyIn("Wel");
        c.keyIn("come");
        //output AmosH's blog Welcome
        c.keyIn("end this");
        c.printMsg();
        //output end this
    }
}
```
## 命令模式

考虑这样一个问题：在某些情形下，某个方法需要完成某一个特殊的行为但这个实现无法确定，必须在调用该方法时指定具体的处理行为。

这就意味着，我们必须把“处理方法”作为参数传入该方法，这个“处理行为”用编程实现来说就是一段代码，那么如何把代码传入该方法呢？

我们可以使用Lambda表达式作为处理方法传入，但是当涉及一些比较复杂的处理行为时，这往往是不够的。

我们可以考虑使用Command接口定义一个方法，用这个方法来封装“处理行为”。以下以遍历一个数组作为示范：

```java
public class Test{
    public static void main(String[] args){
        int[] target = {1,-1,10,-20};
        Process pc = new Process();
        pc.process(target,new PrintCmd());
        //output 当前元素为正数1
        //output 当前元素为负数-1
        //output 当前元素为正数10
        //output 当前元素为负数-20
        System.out.println("=======分割线========");
        pc.process(target,new AddCmd());
        //output 数组的和为-10
    }
}

interface Command{
    void process(int[] target);
}

class Process{
    public void process(int[] target,Command cmd){
        cmd.process(target);
    }
}

class PrintCmd implements Command{
    public void process(int[] target){
        for(int i:target){
            if(i<0){
                System.out.println("当前元素为负数"+i);
            }else{
                System.out.println("当前元素为正数"+i);
            }
        }
    }
}

class AddCmd implements Command{
    private int count;
    public void process(int[] target){
        for(int i:target){
            count += i;
        }
        System.out.println("数组的和为"+count);
    }
}
```
对于PrintCmd和AddCmd而言，真正有用的就是process方法，该方法体传入Process的实例中用以处理数组。实现了process()方法和“处理方法”的相分离。