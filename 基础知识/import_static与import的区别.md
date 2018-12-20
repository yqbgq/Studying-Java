我们需要导入一个一个包或者一个类的时候，我们会使用import语句来帮助我们，这样我们在后续的代码中就不需要在使用类时指定类的全名了。

JDK 1.5之后，Java增加了一种静态导入的语法（import static），它用于导入指定类的某个静态成员变量、方法或全部的静态成员变量、方法。

import static 也支持和import一样的 * 操作符，用以导入指定类的所有静态成员变量或方法。

import static应该放在Java源程序中package语句之后，类定义之前，和import没有任何顺序要求。

使用方法示例

```java
import static java.lang.System.*;
import static java.lang.Math.*;

public class sample {
    public static void main(String[] args){
        out.print(PI);
        out.print(sqrt(256));
    }
}
```
