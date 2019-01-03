> MyBatis真正强大之处在于它的映射器。因为它异常强大并且编写相对简单，不仅比传统编写SQL语句做的更好并且能节省将近95%的代码量

## XML中顶级元素汇总
1. **cache：** 给定命名空间的缓存配置
2. **cache-ref:** 其他给定命名空间缓存配置的引用
3. **resultMap:** 最复杂也是最强大的元素，用来描述如何从数据库结果集中加载对象
4. **sql:** 可以被其他语句引用的重复语句块
5. **insert:** 映射插入语句
6. **update:** 映射更新语句
7. **delete:** 映射删除语句
8. **select:** 映射查询语句

## select
查询语句是MyBatis中使用最多的元素之一。一个简单的查询元素是非常简单的，看如下示例：
```xml
<select id="selectPerson" parameterType="int" resultType="hashmap">
  SELECT * FROM PERSON WHERE ID = #{id}
</select>
```

这个语句的ID为selectPerson ，并且接受一个int类型的参数，返回一个HashMap类型的对象。其中键名为列名，值为结果行中的对应值。

当然，select元素中有很多属性可以允许我们配置，用以决定每条语句的作用细节：

具体的相关其他属性可以查阅官网：
[Mapper XML 文件](http://www.mybatis.org/mybatis-3/zh/sqlmap-xml.html)


## insert，update和delete
数据变更语句insert，update和delete的实现都非常接近：
```xml
<insert
  id="insertAuthor"
  parameterType="domain.blog.Author"
  flushCache="true"
  statementType="PREPARED"
  keyProperty=""
  keyColumn=""
  useGeneratedKeys=""
  timeout="20">

<update
  id="updateAuthor"
  parameterType="domain.blog.Author"
  flushCache="true"
  statementType="PREPARED"
  timeout="20">

<delete
  id="deleteAuthor"
  parameterType="domain.blog.Author"
  flushCache="true"
  statementType="PREPARED"
  timeout="20">
```
具体的相关其他属性可以查阅官网：
[Mapper XML 文件](http://www.mybatis.org/mybatis-3/zh/sqlmap-xml.html)

## sql 
这个元素可以被用来定义可重复的SQL代码段，可以包含在其他语句中。考虑下面这个例子：
```xml
<sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password </sql>
```

它可以被包含在其他语句中，比如：
```xml
<select id="selectUsers" resultType="map">
  select
    <include refid="userColumns"><property name="alias" value="t1"/></include>,
    <include refid="userColumns"><property name="alias" value="t2"/></include>
  from some_table t1
    cross join some_table t2
</select>
```

## 参数 Parameters

一般来说，元素的parameterType会被设置为int或者String之类的原生类或者简单的数据结构。但是当传入一个复杂的对象时，行为就会变得有些不同。比如：
```xml
<insert id="insertUser" parameterType="User">
  insert into users (id, username, password)
  values (#{id}, #{username}, #{password})
</insert>
```

如果User类型的参数对象被传入到语句中，id、username、password属性就会被查找并且传入预处理语句的参数中。

## resultMap

resultMap元素是MyBatis中最重要最强大的元素，它可以让我们从90%的JDBCResultSets数据提取代码中解放出来，并在一些条件下允许我们做一些JDBC所不支持的事情。

ResultMap的设计思想是：**简单的语句不需要明确的结果映射，而复杂一点的语句只需要描述他们的关系就行了。**

一个简单的映射语句如下，它没有明确resultMap，知识简单的将所有的列映射到HashMap的键上，这由resultType指定：
```xml
<select id="selectUsers" resultType="map">
  select id, username, hashedPassword
  from some_table
  where id = #{id}
</select>
```
虽然如此在大部分情况下都够用了，但是HashMap不是一个很好的模型，开发人员可能更加倾向于使用JavaBean或者POJO作为模型。MyBatis对两者都支持。

我们假定现在已经拥有一个名叫User的JavaBean，它定义了id、username以及hashedPassword属性。

这样的一个JavaBean可以被映射到ResultSet，就像映射到HashMap中一样简单：
```xml
<select id="selectUsers" resultType="com.someapp.model.User">
  select id, username, hashedPassword
  from some_table
  where id = #{id}
</select>
```

如果列名和属性名并不能完全吻合的话，我们也可以使用自行定义的外部ResultMap：
```xml
<resultMap id="userResultMap" type="User">
  <id property="id" column="user_id" />
  <result property="username" column="user_name"/>
  <result property="password" column="hashed_password"/>
</resultMap>
```

之后只要在需要使用它的元素中对它进行引用就好了：
```xml
<select id="selectUsers" resultMap="userResultMap">
  select user_id, user_name, hashed_password
  from some_table
  where id = #{id}
</select>
```
