> 在Java的try/catch语句中，finally总会在最后执行。如果尝试在finally块中返回内容或者抛出异常，可能会遇上一些奇怪的事情

不管异常捕捉块中try代码是否出现异常，也不管究竟是哪一个catch块被执行，甚至在try或者catch块中进行了return或者throw，finally块总会被执行。除非在try块或者catch块中退出了虚拟机比如执行了“System.exit(1)”语句来退出虚拟机，那么finally块最终将失去被执行的机会。

通常情况下，不要在finally块中使用如return或者throw等导致方法终止的语句，一旦在finally块中使用了return或者throw语句，将会导致try块、catch块中的return、throw语句失效。参考下面这个例子：
```java
public class Main {
	public static void main(String args[]){
		//输出 false
		System.out.print(test());
	}
	
	public static boolean test(){
		try{
			return true;
		}finally{
			return false;
		}
	}
}
```
上面程序在finally块中定义了一个return false语句，这将导致try块中的return true失去作用。

当Java程序执行try块、catch块遇到了return或者throw语句，这两个语句都会导致该方法立即结束，但是系统执行这两个语句并不会结束该方法，而是去寻找该异常处理流程中是否包含finally块，如果没有finally块，则立即执行return或者throw语句，方法终止。如果存在，系统立即开始执行finally块，只有finally块执行完之后，才会跳回来执行try、catch块中的return或者throw语句。如果finally块中也使用了return或者throw语句导致方法终止，那么finally块已经终止了方法，系统不会跳回去执行try、catch块中的任何代码。参考下面的代码：
```java
public class Test {
	public static void main(String[] args) throws Exception{
		try{//在try块中尝试抛出异常，发现先输出finally block，然后才处理异常信息
			Test.t1();
		}catch(Exception e){
			System.out.println(e);
		}
		//在try块中尝试进行返回内容，结果先处理finally块，之后才处理return语句
		System.out.println(Test.t2());
	}
	
	public static String t2(){
		try{
			System.out.println("1");
			return "123";
		}
		finally{
			System.out.println("2");
		}
	}
	public static void t1() throws Exception{
		try{
			throw new Exception("exception");
		}
		finally{
			System.out.println("finally block");
		}
	}
}
```
