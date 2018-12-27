> Java在JDK1.5中增加泛型支持在很大程度上都是为了让集合能够记住其元素的数据类型。

在没有泛型之前，一旦把一个对象放入Java的集合中，集合不会记忆对象的类型，并会把所有的对象当成Object类型处理。当程序从集合中取出对象时，就需要进行强制类型转换。这种强制类型转换不仅会让代码更加臃肿，而且容易引起ClassCastException异常。


从Java5以后，Java引入了参数化类型的概念，允许程序在创建集合时指定集合元素的类型：

```java
List<String> temp = new ArrayList<String>();
```
上面的程序创建了一个只能放入String类型的List集合，可以看到我们在尖括号中放了一个数据类型，表明这个集合接口、集合类只能保存特定的对象。

当我们试图向这个集合中放入其他类型的对象时，会引发编译错误。

上面的代码不仅比不使用泛型更加健壮，而且程序更加简洁，无需使用强制类型转换。

包含泛型声明的类型可以在定义变量、创建对象时传入一个类型实参，从而动态地生成无数个逻辑上的子类，但这种子类在物理上并不存在。

让我们看看Java5以后使用泛型改写的List接口和Map的代码片段：
```java
//定义接口时指定了一个类型形参，该形参名为E
public interface List<E> extends Collection<E> {
	//在该接口中，E可以作为类型使用
	 void add(int index, E element);
	 ListIterator<E> listIterator();
}
//定义该接口时指定了两个类型形参，其参数名为K和V
public interface Map<K, V> {
    V put(K key, V value);
    //Set<K>作为一个特殊的类型使用
    Set<K> keySet();
}
```
当然，不止在集合中可以应用泛型声明，我们可以在任何类、接口中增加泛型声明。下面是示例：
```java
class ReturnResult<T>{
	private T msg;
	public ReturnResult(T get){
		msg = get;
	}
	public T getResult(){
		return msg;
	}
}

public class Test {
	public static void main(String[] args){
		ReturnResult<String> rr1 = new ReturnResult<>("AmosH");
		ReturnResult<Integer> rr2 = new ReturnResult<>(1998);
		//output AmosH
		System.out.println(rr1.getResult());
		//output 1998
		System.out.println(rr2.getResult());
	}
}
```
上面的程序定义了一个带泛型声明的ReturnResult<T>类，使用这个类可以返回任何类型的对象。

要注意的是，即使使用了泛型，想要重写类的构造器的话，构造器名依然是原来的类名，不需要带泛型声明。

最后值得一提的是，并不存在泛型类。

虽然我们可以把ArrayList<String>类当成ArrayList的子类，但实际上，系统并没有为ArrayList<String>生成新的class文件，而且也不会将它当成新类来处理。

参考下面这个例子，会输出true而不是false
```java
public class Test {
	public static void main(String[] args){
		ArrayList<Integer> array1 = new ArrayList<>();
		ArrayList<String> array2 = new ArrayList<>();
		// output true
		System.out.print(array1.getClass() == array2.getClass());
	}
}
```
不管泛型的类型形参传入哪一种类型实参，对于Java来说，它们依然被当成同一个类处理，在内存中也只占用一块内存空间，因此静态方法、静态初始块以及静态变量的声明或者初始化中不允许使用类型形参。
