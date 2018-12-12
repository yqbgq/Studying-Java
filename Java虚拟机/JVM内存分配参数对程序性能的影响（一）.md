使用IDEA的小伙伴可以在Run -> Edit Configurations 设置VM options属性来设置JVM参数：
![](https://www.amoshuang.com/wp-content/uploads/2018/12/设置JVM参数.png)

## 1. 设置最大堆
  使用-Xmx参数可以指定Java应用程序的最大堆。最大堆为新生代和老年代的大小纸盒的最大值，是Java程序的堆上限。

  使用如下代码进行测试，每次循环将会向容器v中增加1M的数据。
  ```java
  public class TestXmx {
	public static void main(String args[]){
		Vector v=new Vector();
		for(int i=1;i&lt;=10;i++){
			byte[] b=new byte[1024*1024];
			v.add(b);             //强引用，GC不能释放空间
			System.out.println(i+"M is allocated");
		}
		System.out.println("Max memory:"+Runtime.getRuntime().maxMemory()/1024/1024+"M");
	}
}
```
  当使用参数-Xmx11M进行运行时，程序可以顺利结束并且没有任何异常。但当使用参数-Xmx5M进行运行时，则会报OutOfMemoryError错误。

  在运行时，可以使用Runtime.getRuntime().maxMemory()方法获取最大堆内存。
  ## 2.设置最小堆内存
  使用参数-XMS 可以用于设置系统的最小堆空间，也就是JVM启动时所占据的操作空间大小。

  Java应用程序在运行时，首先会分配指定大小的内存大小，并尽可能在这个空间段内运行程序。当-Xms指定的内存大小确实无法满足应用程序时，JVM才会向操作系统申请更多的内存，知道内存大小达到-Xmx指定的大小为止。若超过，则抛出OutOfMemoryError错误。

  如果-Xms太小，则会导致JVM为了保证系统尽可能可以在指定内存范围内运行而频繁进行GC操作，以释放失效的内存空间，从而对系统性能产生影响。

  注意：将-XmS值设置为-Xmx的值，可以减少在系统运行初期GC的次数和耗时。
