## MyBatis 简介

![MyBatis](http://www.mybatis.org/images/mybatis-logo.png)

MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 POJOs(Plain Old Java Objects,普通的 Java对象)映射成数据库中的记录。

## 构建项目

#### 使用gradle
```
compile 'org.mybatis:mybatis:3.4.6'
```

#### 使用Maven
```
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>x.x.x</version>
</dependency>
```

## 准备
#### 使用gradle导入MySQL驱动
```
compile 'mysql:mysql-connector-java:8.0.13'
```

#### 准备数据库
![准备数据库](https://www.amoshuang.com/wp-content/uploads/2019/01/Mybatis入门使用的数据库.png)

## 入门实例

#### SqlSessionFactory

每个基于MyBatis的应用都是以一个SqlSessionFactory的实例为中心的。它可以通过SqlSessionFactoryBuilder产生。而SqlSessionFactoryBuilder的实例则可以从XML配置文件或者一个预先设定好的Configuration的实例构建出来。

MyBatis包含了一个名叫**Resources**的工具类，它包含一些使用方法，可以使得从classpath或者其他位置加载资源文件更加方便。

我们可以采用如下的方式使用XML构建SqlSessionFactory：
```java
String resource = "mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

#### 编写XML配置文件
在XML配置文件中，包含了对MyBatis系统的核心设置，包含获取数据库连接实例的数据源和决定事务作用域和控制方式的事务管理器等。
一个简单的配置文件示例如下：
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--配置MyBatis的环境设置 -->
    <environments default="development">
        <!-- 开发环境下的环境设置-->
        <environment id="development">
            <transactionManager type="JDBC"/>
            <!--配置数据库连接-->
            <dataSource type="POOLED">
                <!--MySQL驱动，使用旧版驱动会报错-->
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <!--相应的URL，在某个驱动版本之后，不使用SSL会报错，不设置时区也会报错，后面设置接受的字符编码等-->
                <property name="url" value="jdbc:mysql://127.0.0.1:3306/java?useSSL=true&amp;
                    serverTimezone=GMT%2B8&amp;useUnicode=true&amp;characterEncoding=utf-8"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>
    <!--SQL映射-->
    <mappers>
        <mapper resource="Mapper/StudentMapper.xml"/>
    </mappers>
</configuration>
```

#### 从 SqlSessionFactory 中获取 SqlSession
SqlSessionFactory 是一个工厂类，用来生产SqlSession类实例。

SqlSession 实例包含了所有面向数据库执行SQL命令所需要的全部方法，可以通过SqlSession实例来直接执行已经映射了的SQL语句，

我们可以使用如下的方法获取SqlSession：

```java
SqlSession sqlSession = sqlSessionFactory.openSession();
```

#### 为结果创建DataObject
为了将从数据库中获取的结果集保存在对象中，我们需要为其创建一个DataObject，按照需求为每个字段创建属性变量。对于一些不需要的属性比如自增的ID，可以无需创建对应的属性变量。

相应的DataObject：
```java
package TestMybatis;

public class DataObject {
    private int id;
    private String name;
    private int num;
    private String birthday;
    private String department;
    //----------Setter and Getter--------------
    public int getId() {
        return id;
    }
    public String getName() {
        return name;
    }
    public int getNum() {
        return num;
    }
    public String getBirthday() {
        return birthday;
    }
    public String getDepartment() {
        return department;
    }
    public void setId(int id) {
        this.id = id;
    }
    public void setName(String name) {
        this.name = name;
    }
    public void setNum(int num) {
        this.num = num;
    }
    public void setBirthday(String birthday) {
        this.birthday = birthday;
    }
    public void setDepartment(String department) {
        this.department = department;
    }
    //----------Setter and Getter--------------
    @Override
    public String toString(){
        return getName() + "   的ID是  "   + getId()  + "  出生于   " + getBirthday() + "  是  " + getDepartment()  + "学院的学生";
    }
}

```

#### 为SQL语句创建映射
在MyBatis中，使用的SQL语句都是通过映射定义的。存在XML映射以及Java注解两种方法。对于简单的SQL语句Java注释可以轻松应对，但是对于一些复杂的语句，还是推荐使用XML进行映射。

以下是一个简单的查询语句映射：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--为该mapper指定一个唯一的namespace-->
<mapper namespace="StudentMapper">
    <!--
    在select标签中定义一个select语句。其唯一的ID属性为getStudent，查询所需要使用到的参数类型为int
    查询返回的类型为TestMybatis.DataObject，即将查询结果封装成一个DataObject类的对象进行返回
    -->
    <select id="getStudent" parameterType="int" resultType="TestMybatis.DataObject">
        select * from student where id=#{id}
    </select>
</mapper>
```

在这里我们就可以向上翻一翻上面的MyBatis的配置文件，在在下方的mapper中引用了我们刚刚编写的StudentMapper.xml，这样就完成了映射的注册。

#### 剩下的几步

最后使用获取的sqlSession进行查询，并且打印返回的实体类：
```java
DataObject data = sqlSession.selectOne("getStudent",100);
System.out.println(data);
```

![MyBatis的查询结果](https://www.amoshuang.com/wp-content/uploads/2019/01/MyBatis的查询结果.png)