> 文件锁在操作系统中是很平常的事情，当多个运行的程序需要并发修改同一个文件时，程序之间需要某种机制来进行通信，使用文件锁可以有效的阻止多个进程并发修改同一个文件，所以现在的大部分操作系统都提供了文件锁的功能

从JDK1.4的NIO开始，Java开始提供文件锁的支持。文件锁控制文件的全部或者部分字节的访问。

在NIO中，Java提供了FileLock来支持文件锁定功能，在FileChannel中提供了lock()和tryLock()方法来获得文件锁FileLock对象，从而锁定文件。

**lock()和tryLock()方法存在区别：**
* 使用lock()尝试锁定某个文件时，如果获得了文件锁，该方法就会返回该文件锁，否则返回null。
* 使用tryLock()尝试锁定文件时，它将直接返回而不是阻塞。如果获得了文件锁，那么该方法将会返回文件锁，否则返回null。

如果FileChannel只是想要锁定文件的部分内容而不是锁定全部内容，则可以像如下的方法一般使用：
* lock(long position,long size,boolean shared)对文件从position开始，长度为size的内容加锁，该方法是阻塞式的。
* tryLock(long position,long size,boolean shared)非阻塞式的加锁方法。

当shared参数为true时，表示该锁是一个共享锁，允许多个进程来读取该文件，但是阻止其他进程获得对该文件的排它锁。当shared为false时，表示该锁是一个排它锁，它将锁住对该文件的读写。程序可以通过调用FileLock的isShared来判断它获得的锁是否为共享锁。

处理完文件之后通过FileLock的release()方法释放文件锁。

考虑下面这个示例：
```java
import java.io.FileOutputStream;
import java.nio.channels.FileChannel;
import java.nio.channels.FileLock;

public class FileLockTest {
    public static void main(String[] args){
        try(
                //使用FileOutputStream获取FileChannel
                FileChannel channel = new FileOutputStream("a.txt").getChannel()
                )
        {
            //使用非阻塞方式对指定文件进行加锁
            FileLock lock = channel.tryLock();
            //程序暂停1秒
            Thread.sleep(1000);
            //释放锁
            lock.release();
        }catch (Exception ex){
            ex.printStackTrace();
        }
    }
}
```
**注意：**

文件锁虽然可以用于控制并发访问，但是对于高并发访问的情形，还是推荐使用数据库来保存程序信息。

关于文件锁，还要注意的几点是：

1. 在某些平台上，文件锁仅仅是建议性的，并不是强制性的。这意味着，即使一个程序不能获得文件锁，它也可以对该文件进行读写。
2. 在某些平台上，不能同步的锁定一个文件并把它映射到内存中。
3. 文件锁是由Java虚拟机锁持有的，如果两个java程序使用同一个Java虚拟机运行，则他们不能对同一个文件进行加锁。
4. 在某些平台上关闭FileChannel，会释放Java虚拟机在该文件上的所有锁，因此应该避免对同一个被锁定的文件打开多个FileChannel。