> 如果要在子类中调用父类被覆盖的实例方法，可以使用super限定来调用父类被覆盖的实例方法

super是Java中提供的一个关键字，用于限定该对象调用它从父类继承得到的实例变量或方法。正如this不能出现在static修饰的方法中一样，super也不能出现在static修饰的方法中。因为static修饰的方法是属于一个类的，该方法的调用者可能是一个类，而不是对象，因而super限定也就失去了意义。

如果在构造器中使用super，则super用于限定该构造器初始化的是该对象从父类继承得到的实例变量，而不是该类自己的实例变量。

如果子类定义了和父类同名的实例变量，则会发生子类实例变量隐藏父类实例变量的情形，如果一定要访问父类的实例变量的话，可以在子类定义的实例方法中通过super来访问父类中被隐藏的实例变量。

事实上，当系统创建了一个对象时会为该对象分配多块内存，一块用于存储该类自身定义的实例变量，其他用于存储从父类处继承得到的实例变量。

如果实例方法访问名为a的成员变量，但是没有显示指定调用者，则系统查找a的顺序为：

1. 查找该方法中是否有名为a的局部变量。
2. 查找当前类中是否包含名为a的成员变量。
3. 查找对象的直接父类中是否包含名为a的成员变量，依次上溯到所有父类，直到java.lang.Object类。如果最终不能找到名为a的成员变量，则系统抛出编译错误。

如果被覆盖的是类变量，在子类的方法中可以通过父类名作为调用者来访问被覆盖的类变量。