> 对于大部分的对象而言，程序里会有一个引用变量来引用该对象，这是最常见的引用方法。除此之外，java.lang.ref包下还提供了3个类：SoftReference、WeakReference和PhantomReference。它们分别代表了系统对对象的另外3中引用方式：软引用、弱引用和虚引用。


Java中四种引用的区别和关联：
1. **强引用**。这是Java中最常见的引用方式。程序创建一个对象，并把这个对象赋给一个引用变量，程序通过该引用变量来操作实际的对象。当一个对象被一个或者多个引用变量引用时，它处于可达状态，不能被系统垃圾回收机制回收。
2. **软引用**。当一个对象只有软引用时，它有可能被垃圾回收机制回收。对于只有软引用的对象而言，当系统内存空间足够时，它不会被系统回收，程序也可以使用该对象。当系统内存空间不足时，系统可能会回收它。软引用通常用于对内存敏感的程序中。
3. **弱引用**。弱引用和软引用很像，但它的级别比软引用更低。对于只有弱引用的对象而言，当系统垃圾回收机制运行时，不管系统内存是否足够，总会回收该对象所占用的内存。当然并不是说当一个对象只有软引用时，它会立即被回收，正如那些失去引用的对象一样，必须等到系统垃圾回收机制运行时才会被回收。
4. **虚引用**。虚引用完全类似于没有引用。虚引用对对象本身没有什么太大的影响。虚引用主要用于跟踪对象被垃圾回收的状态，虚引用不能单独使用，虚引用必须和引用队列（ReferenceQueue）搭配使用。

软、弱和虚引用都包含了一个get()方法，用于获取被它们所引用的对象。区别是虚引用的get()方法只会返回null。

引用队列由java.lang.ReferenceQueue类表示，它用于保存被回收后对象的引用。与软引用和弱引用不同的是，虚引用在对象被释放之前，将把它对应的虚引用添加到它关联的引用队列中，这使得可以在对象被回收之前采取行动。

下面的代码示范了弱引用所引用的对象被系统垃圾回收的过程：
```java
import java.lang.ref.WeakReference;

public class Test{
	public static void main(String[] args){
		String str = new String("AmosH");
		WeakReference wr = new WeakReference(str);
		//建立弱引用，此弱引用指向"AmosH"字符串
		//注意这里一定要使用new来创建一个字符串对象
		//否则会该字符串会被保留在常量值而非堆内存中
		str = null;
		//切断"AmosH"字符串的强引用
		System.out.println(wr.get());
		//output "AmosH"
		//此时弱引用依然有效
		System.gc();
		System.runFinalization();
		//调用垃圾回收机制
		System.out.println(wr.get());
		//output null
		//弱引用已经被回收
	}
}
```

弱引用和软引用可以单独使用，但是虚引用不能单独使用。虚引用的主要作用是搭配引用队列来跟踪对象被垃圾回收的状态，程序可以通过检查与虚引用关联的引用队列中是否已经包含了该虚引用，从而了解虚引用所引用的对象是否即将被释放。

下面代码示范了虚引用对象被系统垃圾回收的过程：
```java
import java.lang.ref.PhantomReference;
import java.lang.ref.ReferenceQueue;

public class Sample {
	public static void main(String[] args){
		String str = new String("AmosH");
		ReferenceQueue rq = new ReferenceQueue();
		//创建一个引用队列
		PhantomReference pr = new PhantomReference(str,rq);
		//创建一个虚引用，并且将该引用和rq引用队列关联
		str = null;
		//切断"AmosH"字符串的引用
		System.out.println(pr.get());
		//output null，因为系统无法通过虚引用的get()方法获取被引用对象
		System.gc();
		System.runFinalization();
		//强制垃圾回收
		System.out.println(rq.poll() == pr);
		//output true
		//虚引用被回收
	}
}
```

使用这些引用类可以避免在程序执行期间将对象留在内存中。如果以软引用、弱引用和虚引用的方式引用对象，垃圾回收器就可以随意的释放对象。如果希望尽可能减小程序在其生命周期中所占用的内存大小时，这些引用类就会很有用处。

但是要注意的是，使用了这些特殊的引用类，就不能保留对对象的强引用，这会浪费这些引用类所提供的好处。

由于垃圾回收的不确定性，当程序希望从软、弱引用中获取被引用对象时，可能这个被引用对象已经被释放了。如果需要使用那个被引用的对象，则必须重新创建该对象：
```java
obj = wr.get();
//获取引用所引用的对象
//如果被取出的对象为null
if(obj==null){
	obj = recreateIt();
	//重建该对象并使用一个强引用来引用它
	//这里使用的伪代码，需要进行自定义
	wr = new WeakReference(obj);
	//重建这个弱引用
}
...//操作obj对象
obj = null;
//再次切断obj和对象之间的关系
```