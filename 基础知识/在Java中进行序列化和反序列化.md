> 对象序列化的目标是将对象保存在磁盘中，或者允许在网络中直接传输对象。

对象序列化允许把内存中的Java对象转换成平台无关的二进制流，从而允许把这种二进制流持久保存在磁盘上或者通过网络将这种二进制流传输到另外一个网络节点。

其他程序一旦获得了这种二进制流，都可以将这种二进制流恢复成原本的Java对象。

## 序列化的含义和意义
序列化机制允许将实现序列化的Java对象转换成字节序列，这些字节序列可以进行持久化或者通过网络传输，使得对象可以脱离程序的运行而独立存在。

序列化将一个Java对象写入IO流中。对象的反序列化可以从IO流中恢复该Java对象。

如果想要让某个对象支持序列化机制，那么必须让它的类是可以序列化的。为了让某个类是可序列化的，那么该类必须实现如下两个接口之一：

* Serializable
* Externalizable

## 使用对象流实现序列化
使用Serializable来实现序列化非常简单，主要让目标类实现该标记接口即可，无需实现任何方法。一旦某个类实现了Serializable接口，该类的对象就是可序列化的。

考虑下面这个例子：

**首先展示用来序列化的Person类：**
```java
public class Person implements java.io.Serializable{
    private String name;
    private int age;
    public Person(String name,int age){
        this.name = name;
        this.age = age;
    }
    public void talk(){
        System.out.println("我是"+name+"我已经"+age+"岁了！");
    }
}
```
**接下里的代码展现了如何序列化和反序列化：**
```java
import java.io.*;

public class Serializable {
    public static void main(String[] args){
        try(
                //创建一个ObjectOutputStream输出流
                ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("F:/test.txt"))
                )
        {
            Person per = new Person("AmosH",20);
            //将per对象写入输出流
            oos.writeObject(per);
        }catch (IOException ex){
            ex.printStackTrace();
        }

        try(
                ObjectInputStream ois = new ObjectInputStream(new FileInputStream("F:/test.txt"))
                )
        {
            //反序列化per对象，如果知道对象的类型，可以直接采用强制类型转换成实际类型
            Person per = (Person)ois.readObject();
            //输出我是AmosH我已经20岁了！
            per.talk();
        }catch (Exception ex){
            ex.printStackTrace();
        }
    }
}
```
需要注意的四点是：

1. 反序列化读取的仅仅是Java对象的数据，而不是Java类，因此采用反序列化回复Java对象时，必须提供该Java对象所属类的class文件，否则将会引起ClassNotFoundException异常。
2. 当反序列化读取Java对象时，无需通过构造器来初始化对象。
3. 如果使用序列化机制在文件中写入了多个Java对象，使用反序列化机制恢复对象时必须按实际写入的顺序读取。
4. 当一个可序列化类有多个父类（包括直接父类和间接父类），这些父类要么包含无参数的构造器，要么也是可序列化的。否则反序列化时将会抛出InvalidClassException异常。如果父类是不可序列化的，只是带有无参数的构造器，则该父类中定义的成员变量值不会序列化到二进制流中。

## 对象引用的序列化
当被序列化的对象中包含的成员变量的类型不是基本类型而是一个引用类型时，那么这个引用类必须是可序列化的，否则拥有该类型成员变量的类也是不可序列化的。

我们假设如下一种情况：

**程序中有一个Student对象和两个Teacher对象，两个Teacher对象中都有成员变量引用这个Student对象。**
在这样的情形下，如果我们序列化这三个对象，因为程序在序列化Teacher对象时也会将其中的引用对象实例序列化，那么似乎程序会向输出流中输出三个Person对象。

如果程序想输出流中输出了三个Person对象，那么程序从输入流中反序列化这些对象时，将会得到三个Student对象，两个Teacher对象所引用的Student对象将不是同一个对象，这显然也违背了Java序列化机制的初衷。

所以，Java序列化机制采用了一种特殊的序列化算法：
* 所有保存到磁盘中的对象都有一个序列化编号。
* 当程序试图序列化一个对象时，程序先会检查该对象是否已经被序列化过。只有该对象并为在本次虚拟机中被序列化过，系统才会将该对象转换成字节序列并输出。
* 如果某个对象已经序列化过，则程序将只是直接输出一个序列化编号而不是再次重新序列化该对象。

但是要注意的是：如果序列化的是一个可变对象时，只有第一次序列化才会将对象转换成字节序列并输出。当对象的实例变量值已经改变之后，及时再次序列化，也只会输出前面的序列化编号。