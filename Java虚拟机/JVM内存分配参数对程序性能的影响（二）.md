<!-- wp:heading -->
<h2>3.设置新生代</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>  参数-Xmn可用于设置新生代大小。设置一个较大的新生代会减小老年代的大小，这个参数对系统性能以及GC行为有很大的影响。新生代的大小一般设置为整个堆空间的1/4到1/3左右。</p>
<!-- /wp:paragraph -->

<!-- wp:heading -->
<h2>4.设置持久代  </h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>  持久代（方法区）不属于堆的一部分，在Hot Spot虚拟机中，使用-XX:MaxPermSize可以设置持久代的最大值，使用-XX:PermSize可以设置持久代的初始大小。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>  持久代的大小直接决定了系统可以支持多少各类定义和多少常量，设置合理的持久代大小有助于维持系统的稳定。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>  系统支持的最大类数量，与MaxPermSize成正比。一般来说，MaxPermSize设置为64MB已经可以满足绝大部分应用程序正常工作。如果依然出现永久区溢出，可以将MaxPermSize设置为128MB。这是两个常用的永久区取值。如果128MB依然不能满足应用程序需求，那么对于大部分应用程序而言，应该考虑优化系统设计，减少动态类的产生，或者利用GC回收部分驻扎在永久区的无用类信息，以使系统健康运行。</p>
<!-- /wp:paragraph -->

<!-- wp:heading -->
<h2>  5.设置线程栈  </h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>  线程栈是线程的一块私有空间，在JVM中，可以使用-Xss参数设置线程栈的大小。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>  在线程中进行局部变量分配、函数调用时，都需要在栈中开辟空间。如果栈空间分配太小，那么线程在运行时，可能会因为没有足够的空间分配局部变量或者达不到足够的函数调用深度，导致程序异常退出。如果栈空间太大，那么开设线程所需的内存成本就会上升，系统所能支持的线程总数就会下降。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>  因为Java堆也是向操作系统申请内存空间，因此如果堆空间太大，就会导致操作系统可用于线程栈的空间减少，从而间接减少程序所能支持的线程数量。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>  以下代码会尝试尽可能的开设线程，并且在线程数量饱和时，打印已经开设的线程数量：</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>public static class MyThread extends Thread{
        @Override
        public void run(){
            try {
                Thread.sleep(10000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String args[]){
        int i=0;
        try{
            for( i=0;i&lt;10000;i++){
                new MyThread().start();
            }
        }catch(OutOfMemoryError e){
            System.out.println("count thread is "+i);
            e.printStackTrace();
        }
    }</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>  首先使用-Xss1M运行程序，即设置每个线程拥有1MB的栈空间，在我的计算机上，总共可以开设线程1168个。当使用-Xss10M运行程序时，公共可以137个线程。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>  如果尝试在JVM参数中指定堆大小，则会发现系统所支持的线程数和对大小还有关系：</p>
<!-- /wp:paragraph -->

<!-- wp:table -->
<table class="wp-block-table"><tbody><tr><td>        </td><td>-Xss1M</td><td> -Xss20M </td></tr><tr><td>-Xms100m  -Xms100M</td><td>1170</td><td>66</td></tr><tr><td> -Xms300m  -Xms300M </td><td>1139</td><td>64</td></tr><tr><td> -Xms500m  -Xms500M  </td><td>998</td><td>55</td></tr><tr><td> -Xms700m  -Xms700M  </td><td>852</td><td>44</td></tr></tbody></table>
<!-- /wp:table -->

<!-- wp:paragraph -->
<p>  当系统空间不够而无法创建新的线程时，会抛出OutOfMemoryError错误。但这并不是因为堆内存不够而造成的OOM，在这种情况下，应该尝试减少堆内存，以换取更多的系统空间来解决这个问题。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>  综上，如果系统确实需要大量线程并发执行，那么设置一个较小的堆和较小的栈有助于提高系统所能承受的最大线程数。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>  </p>
<!-- /wp:paragraph -->
