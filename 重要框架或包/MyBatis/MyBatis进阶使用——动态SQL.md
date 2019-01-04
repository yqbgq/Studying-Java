> MyBatis的强大特性之一就是它的动态SQL。如果你有使用JDBC或者其他类似框架的经验，你一定会体会到根据不同条件拼接SQL语句的痛苦。然而利用动态SQL这一特性可以彻底摆脱这一痛苦

MyBatis精简了元素种类，在MyBatis3中，我们只需要学习以下4种元素：
1. **if**
2. **choose(when,otherwise)**
3. **trim(where,set)**
4. **foreach**

## if
动态SQL通常要做的事情就是根据条件包含where子句的一部分，比如：
```xml
<select id="findActiveBlogWithTitleLike"
     resultType="Blog">
  SELECT * FROM BLOG 
  WHERE state = ‘ACTIVE’ 
  <if test="title != null">
    AND title like #{title}
  </if>
</select>
```

这条语句提供了一种可选的查找文本功能。如果没有传入“title”，那么就返回所有处于“ACTIVE”状态的博文。反之则还要进行根据“title”参数的模糊查找。

在一个标签中，可以如上面例子一般，嵌套多个if元素。

## choose, when, otherwise
如果我们不想应用所有的条件语句而是像switch语句一般进行选择的话，那么我们就应该使用choose元素。

修改上面那个例子，如果存在“title”参数则按照“title”查找，如果提供了“author”参数就按照“author”查找。如果两者都没有提供就返回所有符合条件的博文。
```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select>
```

## trim , where , set

那么我们现在希望将所有的where子句的条件都变成动态SQL，可能会编写出这样的代码：
```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG 
  WHERE 
  <if test="state != null">
    state = #{state}
  </if> 
  <if test="title != null">
    AND title like #{title}
  </if>
  <if test="author != null and author.name != null">
    AND author_name like #{author.name}
  </if>
</select>
```

然而如果这些条件一个都没有匹配上的话，结果SQL语句很可能会变成这样：
```sql
SELECT * FROM BLOG
WHERE
```

如果只有第二个匹配上的话，会变成这样：
```sql
SELECT * FROM BLOG
WHERE 
AND title like ‘someTitle’
```

这些查询都会失败，为了解决这个问题，MyBatis有一个简单的解决方案：
```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG 
  <where> 
    <if test="state != null">
         state = #{state}
    </if> 
    <if test="title != null">
        AND title like #{title}
    </if>
    <if test="author != null and author.name != null">
        AND author_name like #{author.name}
    </if>
  </where>
</select>
```

where元素只有在至少一个子元素的条件返回SQL子句的情况下才插入“WHERE”子句。而且若语句的开头为AND或者OR也会将它们去除。

如果希望定制where的行为，我们可以通过自定义trim元素来定制where元素的功能。比如和where元素等价的自定义trim元素为：
```xml
<trim prefix="WHERE" prefixOverrides="AND |OR ">
  ... 
</trim>
```

类似的用于动态更新语句的解决方案叫做set。set元素可以用于动态包含所需要更新的列，而舍弃其他。比如：
```xml
<update id="updateAuthorIfNecessary">
  update Author
    <set>
      <if test="username != null">username=#{username},</if>
      <if test="password != null">password=#{password},</if>
      <if test="email != null">email=#{email},</if>
      <if test="bio != null">bio=#{bio}</if>
    </set>
  where id=#{id}
</update>
```

这里set元素会动态前置SET关键字同时删除无关的逗号。

## foreach
动态SQL的另外一个常用的操作需求就是对一个集合进行遍历通常是在构建IN条件语句的时候，比如：
```xml
<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT *
  FROM POST P
  WHERE ID in
  <foreach item="item" index="index" collection="list"
      open="(" separator="," close=")">
        #{item}
  </foreach>
</select>
```

foreach元素的功能非常强大，它袁旭你指定一个集合，声明可以在元素体内使用的集合项和索引变量。也允许你指定开头和结尾的字符串以及在迭代结果之间放置分隔符。

**注意：** 我们可以使用任何可迭代对象传递给foreach作为集合参数。当使用可迭代对象或者数组时，index是当前迭代的次数，itme是本次迭代获取的元素。当使用Map对象时，index是键，item是值。