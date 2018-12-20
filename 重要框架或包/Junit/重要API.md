
# Junit中的一些重要API

> Junit中的重要代码都包含在四个核心类中

|序号 |   类名    |    描述   |
|:----:|:----------:|:---------:|
|  1  |   Assert       | assert 方法的集合 |
|2|TestCase|一个定义了运行多重测试的固定装置|
|3|TestRunner|TestResult 集合了执行测试样例的所有结果|
|4|TestSuite|TestSuite 是测试的集合|

## Assert类

这个类提供了一系列的编写测试的有用的声明方法。只有失败的声明方法才会被记录。Assert 类的重要方法列式如下：
1. **void assertEquals(boolean expected, boolean actual)**  检查两个变量或者等式是否相等
2. **void assertFalse(boolean condition)** 检查条件是否是假
3. **void assertNotNull(Object object)** 检查对象不为空
4. **void assertNull(Object object)** 检查对象为空
5. **void assertTrue(boolean condition)** 检查条件为真
6. **void fail()** 在没有报告的情况下使测试不通过

使用示例：
```java
import org.junit.Test;
import static org.junit.Assert.*;
public class TestJunit1 {
   @Test
   public void testAdd() {
      //test data
      int num= 5;
      String temp= null;
      String str= "Junit is working fine";

      //check for equality
      assertEquals("Junit is working fine", str);

      //check for false condition
      assertFalse(num > 6);

      //check for not null value
      assertNotNull(str);
   }
}
```

## TestCase类

该类的定义为：
```java
public abstract class TestCase extends Assert implements Test
```
测试样例定义了运行多重测试的固定格式。TestCase 类的一些重要方法列式如下：
1. **int countTestCases()** 为被run(TestResult result) 执行的测试案例计数
2. **TestResult createResult()** 创建一个默认的 TestResult 对象
3. **String getName()** 获取测试的名称
4. **TestResult run()**  一个运行这个测试的方便的方法，收集由TestResult 对象产生的结果
5. **void run(TestResult result)** 在 TestResult 中运行测试案例并收集结果
6. **void setName(String name)** 设置测试类的名称
7. **void setUp()** 创建一个固定装置比如打开网络连接
8. **void tearDown()** 拆除一个固定装置，比如关闭一个网络连接

使用示例：
```java
import junit.framework.TestCase;
import org.junit.Before;
import org.junit.Test;
public class TestJunit2 extends TestCase  {
   protected double fValue1;
   protected double fValue2;

   @Before 
   public void setUp() {
      fValue1= 2.0;
      fValue2= 3.0;
   }

   @Test
   public void testAdd() {
      //count the number of test cases
      System.out.println("No of Test Case = "+ this.countTestCases());

      //test getName 
      String name= this.getName();
      System.out.println("Test Case Name = "+ name);

      //test setName
      this.setName("testNewAdd");
      String newName= this.getName();
      System.out.println("Updated Test Case Name = "+ newName);
   }
   //tearDown used to close the connection or clean up activities
   public void tearDown(  ) {
   }
}
```

## TestResult 类

该类的定义为：
```java
public class TestResult extends Object
```
该类会收集所有测试案例的结果，它是收集参数层面的一个实例。这个实验框架区分失败和错误。失败是可以预料的并且可以通过假设来检查。

其中一些比较重要的方法如下：
1. **void addError(Test test, Throwable t)** 在错误列表中添加一个错误
2. **void addFailure(Test test, AssertionFailedError t)** 在失败列表中添加一个失败
3. **void endTest(Test test)** 显示测试被编译的结果
4. **int errorCount()** 获取被检测出的错误的数量
5. **Enumeration errors()** 返回错误的详细信息
6. **int failureCount()** 获取被检测出的失败的数量
7. **void run(TestCase test)** 运行 TestCase 
8. **int int runCount()** 获取运行测试的数量
9. **void startTest(Test test)** 声明一个测试即将开始
10. **void stop()** 标明测试必须结束

使用示例：
```java
import org.junit.Test;
import junit.framework.AssertionFailedError;
import junit.framework.TestResult;

public class TestJunit3 extends TestResult {
   // add the error
   public synchronized void addError(Test test, Throwable t) {
      super.addError((junit.framework.Test) test, t);
   }

   // add the failure
   public synchronized void addFailure(Test test, AssertionFailedError t) {
      super.addFailure((junit.framework.Test) test, t);
   }
   @Test
   public void testAdd() {
   // add any test
   }

   // Marks that the test run should stop.
   public synchronized void stop() {
   //stop the test here
   }
}
```

## TestSuite 类
该类的定义为：
```java
public class TestSuite extends Object implements Test
```

该类中一些比较重要的方法：
1. **void addTest(Test test)** 在套中加入测试。
2. **void addTestSuite(Class<? extends TestCase> testClass)** 将已经给定的类中的测试加到套中
3. **int countTestCases()**  对这个测试即将运行的测试案例进行计数
4. **String getName()** 返回套的名称。
5. **void run(TestResult result)** 在 TestResult 中运行测试并收集结果
6. **void setName(String name)** 设置套的名称。
7. **Test testAt(int index)** 在给定的目录中返回测试。
8. **int testCount()** 返回套中测试的数量。
9. **static Test warning(String message)** 返回会失败的测试并且记录警告信息

使用示例：
```java
import junit.framework.*;
public class JunitTestSuite {
   public static void main(String[] a) {
      // add the test's in the suite
      TestSuite suite = new TestSuite(TestJunit1.class, TestJunit2.class, TestJunit3.class );
      TestResult result = new TestResult();
      suite.run(result);
      System.out.println("Number of test cases = " + result.runCount());
    }
}
```







