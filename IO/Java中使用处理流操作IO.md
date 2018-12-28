> 如果使用Java中4种普通的IO的基类，使用它们进行IO处理会有些繁琐，如果想要简化编程，就需要借助处理流了。

使用处理流的典型思路就是：使用处理流来包装节点流，程序通过处理流来执行输入/输出功能跟，让节点流和底层的I/O设备、文件进行交互。

识别处理流非常简单，只要流的构造器参数不是一个物理节点，而是一个已经存在的流，那么这种流就一定是处理流。而所有的节点流都是直接以物理IO节点作为构造器参数的。

使用处理流的优势：

* 对开发人员来说，使用处理流进行输入输出处理更加简单。
* 使用处理流的执行效率更高

下面进行一个简单的处理流展示：
```java
public class IO {
    public static void main(String[] args){
        try(    //打开文件，创建节点流
                FileOutputStream fos = new FileOutputStream("F:/sample.txt");
                //创建一个处理流，包装fos
                PrintStream ps = new PrintStream(fos)
                )
        {   //使用处理流向文件中写入内容
            ps.println("问渠那得清如许");
            ps.println("唯有源头活水来");
        }catch (IOException ex){
            ex.printStackTrace();
        }
        try(    //打开文件并使用处理流进行包装
                FileInputStream fis = new FileInputStream("F:/sample.txt");
                BufferedInputStream bis = new BufferedInputStream(fis)
                )
        {   //读取文件
            byte[] buff = new byte[1024];
            //输出问渠那得清如许
            //输出唯有源头活水来
            while(bis.read(buff) > 0){
                System.out.print(new String(buff));
            }
        }catch (IOException ex){
            ex.printStackTrace();
        }
    }
}
```
由于PrintStream类的输出功能非常强大，通常如果需要输出文本内容，都应该将输出流包装成PrintStream后进行输出。

**注意：**

在使用处理流包装了底层节点流之后，关闭输出/输入流资源的时候，只要关闭最上层的处理流即可。关闭处理流时，系统会自动关闭被该处理流包装的节点流。

java的输入输出流体系提供了接近40个类，这些类看上去杂乱而没有规律，但是如果将其按照功能进行划分，不难发现这些类是有规律的：
![Java中常用输入输出流的分类](https://www.amoshuang.com/wp-content/uploads/2018/10/table.png)

表中的粗体字代表节点流，必须直接和物理节点关联；斜体字代表抽象基类，无法直接创建对象。