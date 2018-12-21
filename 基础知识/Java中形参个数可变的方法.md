> Java允许定义形参个数可变的参数，从而允许为方法指定个数可变的形参。

我们在定义方法的时候，在形参类型后面加上“…”，则代表该形参可接受多个参数值。示例如下：

```java
public class sample {
    private static void test(int k, String... books){
        for(String a : books){
            System.out.print(a);
            System.out.print(" ");
        }
        System.out.print("\n");
    }
    public static void main(String[] args){
        String[] temp = {"Amos","H's","blog"};
        test(3, temp);
        test(3,"is","the","best","blog");
    }
}
```
它的输出如下：
```java
Amos H's blog 
is the best blog
```
可变个数形参的本质是一个数组参数，所以，传入多个字符串作为参数值和传入一个字符串数组作为参数值的结果是相同的。

要**注意**的是：

可变个数形参必须放在参数列表的最后，所以一个方法最多只能有一个可变个数形参。
    