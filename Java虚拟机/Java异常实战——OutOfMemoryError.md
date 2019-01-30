> 在Java虚拟机规范描述中，除了程序计数器外，虚拟机内存的其他几个运行区域都有发生 OOM 异常的可能。在这里，用代码验证各个运行时区域存储的内容并讨论该如何进行处理

## Java堆溢出

Java 堆用于存储对象实例，只要不断创建对象，并且保证 GC Roots 到对象之间有可达路径来避免垃圾回收机制清除这些对象，那么对象数量达到最大堆的容量限制之后就会产生内存溢出异常。

#### 异常再现

代码采用如下虚拟机参数：
```java
-Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError
```

这样 Java 堆的大小将被限制为20 MB 且不可拓展。通过参数    **-XX:+HeapDumpOnOutOfMemoryError** 可以让虚拟机在出现内存溢出异常时 Dump 出当前的内存堆转储快照以便时候进行分析。

采用如下代码进行验证：
```java
public class HeapOOM {

	static class OOMObject {
	}

	public static void main(String[] args) {
		List<OOMObject> list = new ArrayList<OOMObject>();

		while (true) {
			list.add(new OOMObject());
		}
	}
}
```

运行结果：
```java
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid3460.hprof ...
Heap dump file created [28199779 bytes in 0.237 secs]
```

#### 解决方法

Java 堆内存的 OOM 异常是实际应用中常见的内存溢出异常情况，出现时往往会紧跟着提示“Java heap space”。

要解决这个区域的异常，一般的手段是先通过内存映像分析工具，比如 MAT ，确认到底是出现了内存泄漏还是内存溢出。

如果是内存泄漏，可以进一步通过工具查看泄漏对象到 GC Roots 的引用链，找到泄漏对象是通过怎样的途径和 GC Roots 相关联并导致垃圾收集器无法自动回收它们所占的空间。

如果不是内存泄漏，换而言之，内存中的对象确实还有必要存活着，那么就应当检查虚拟机的堆参数，与机器物理内存对比看是否还可以调大。从代码层面上看，是否存在某些对象生命周期过长、持有状态时间过长的情况，尝试减少程序运行期间的内存消耗。

## 虚拟机栈和本地方法栈溢出

由于在 HotSpot 虚拟机中并不区分虚拟机栈或者本地方法栈，因此对于 HotSpot 而言，虽然 -Xoss 参数存在，但是实际上是无效的，栈容量只由 -Xss 参数设定。

#### 异常再现
在单线程下，代码采用如下的虚拟机参数：
```
-Xss128k
```

使用该参数减小栈容量，使用如下代码复现异常：
```java
public class JavaVMStackSOF {

	private int stackLength = 1;

	public void stackLeak() {
		stackLength++;
		stackLeak();
	}

	public static void main(String[] args) throws Throwable {
		JavaVMStackSOF oom = new JavaVMStackSOF();
		try {
			oom.stackLeak();
		} catch (Throwable e) {
			System.out.println("stack length:" + oom.stackLength);
			throw e;
		}
	}
}
```

#### 解决方法
如果使用虚拟机默认参数，栈深度在大多数情况下（因为每个方法压入栈的帧大小并不是一样的，所以只能说在大多数情况下）达到1000 ~ 2000 完全没有问题，对于正常的方法调用（包括递归），这个深度应该完全足够。

但是，如果是因为建立过多的线程导致内存溢出，在不能减少线程数或者更换64位虚拟机的情况下，就只能通过减少最大堆和减少栈容量来换取更多的线程。

## 本机直接内存溢出
DirectMemory 容量可以通过 **-XX ：MaxDirectMemorySize** 指定，如果不指定，则默认与Java最大堆一样。

#### 异常再现

使用以下虚拟机参数：
```java
-Xmx20M -XX:MaxDirectMemorySize=10M
```

使用以下代码重现异常：
```java
public class DirectMemoryOOM {

    private static final int _1MB = 1024 * 1024;

    public static void main(String[] args) throws Exception {
        Field unsafeField = Unsafe.class.getDeclaredFields()[0];
        unsafeField.setAccessible(true);
        Unsafe unsafe = (Unsafe) unsafeField.get(null);
        while (true) {
            unsafe.allocateMemory(_1MB);//直接申请分配内存
        }
    }
}
```

#### 解决方法
由 DirectMemory 导致的内存溢出，一个明显的特征就是在Heap Dump 文件中不会看见明显的异常。

如果发现 OOM 之后Dump文件很小，而程序中又直接或者间接使用了NIO ，那么就可以考虑检查一下是不是这方面的原因。