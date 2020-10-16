### 产生原因
　　用户输入的数据没有经过检查，有其余操作，即恶意 SQL。然后变成代码执行，产生了预料之外的结果。**没有运行时的编译，就没有注入，栈溢出、XSS 也是类似。**

- 猜解后台数据库，盗取网站的敏感信息；
- 使用 passWord = "1 or 1 = 1" 来绕过登录验证；
- 添加、修改和删除数据库中的记录。

### 案例分析

#### 猜解后台数据库
　　有个查询用户名的 API 接口，其接口输入参数为 user_id，比如正常的输入为 "SELECT first_name, last_name FROM users WHERE user_id = '15323';" <br />
　　现在输入 "SELECT first_name, last_name FROM users WHERE user_id = '15323' order by 3#`;" ，也是查询 user 表中的数据，但加上 "order by 3" 表示按第三列排序，假设返回错误。再输入 "order by 3" 按第二列排序，最后返回数据，表示该表有两列。<br />
　　知道该表有两列后，在使用 union 联合查询，获取数据库和用户名。因为 union 查询的要和主查询的列数一致，即主查询，查询两个 first_name, last_name，union 查询也是查询两个 database(), user()。

```sql
SELECT first_name, last_name FROM users WHERE user_id = '15323' union select database(), user()#`;
```

#### 绕过登录验证
　　以登录系统为例 http://localhost:8080/martin/login ，正常用户输入用户名和密码，比如 http://localhost:8080/martin/login?userName=admin&passWord=123456 。<br />
　　但是在 SQL 注入中，会输入 http://localhost:8080/martin/login?userName=admin&passWord=1%20or%201=1 。这里将 %20 转为空格，即 passWord 为 "1 or 1 = 1"。由于 SQL 语法的判定，密码输错没关系，因为 or 1 = 1，即可登录成功。

#### 删除表
　　比如在用户登录中，passWord 为 "'';drop table if exists t_user_info;"，有个 ";"，执行完前面的语句，会执行后面的删表语句，导致表被删掉。

### 解决方案

- **预编译，参数化查询。** MyBatis 中 SQL 语句使用 #{password} 占位，不用 ${password}；
- **词法分析，进行过滤。** 前面提到是因为没有检查用户输入的数据，导致的。所以对用户输入的数据，进行检查和过滤。
    1. 对于固定格式的，比如参数为 int 类型，就要确保在 SQL 执行前变量为 int 类型。只要有固定格式，就要按固定格式来检查；
    2. 正则表达式过滤，特别是对引号、特殊字符 or、drop 等进行过滤；
    3. Filter + HttpServletRequestWrapper 过滤，清除、替换掉非法字符。


#### 预编译，参数化查询
　　预编译，先占位，后执行。**在 SQL 语句，使用 #{}，不用 ${}。** 如果要用 ${}，需要对输入参数时间将进行检查过滤，如下为预编译。

```sql
SELECT first_name, last_name FROM users WHERE user_id = #{userId}
```

　　#{} 和 ${} 区别，以 password 传入 "'';drop table if exists t_user_info;" 为例。

- #{}，解析为一个 JDBC 预编译语句（prepared statement）的参数标记符，会将 "'';drop table if exists t_user_info;" 当成 password，然后报错；
- ${}，纯粹的 String 替换，所以 password 为 ""，然后执行后面的语句 "drop table if exists t_user_info;"。

#### 正则表达式过滤
　　针对前面登录案例中的，检查是否存在 or、union 等，存在则返回错误。

#### Filter + HttpServletRequestWrapper 过滤
　　Filter 中拦截请求，在 HttpServletRequestWrapper 中对请求进行检查和过滤。

### reference

- [sql 注入基础原理（超详细）](https://www.jianshu.com/p/078df7a35671)
- [如何从根本上防止 SQL 注入？](https://www.zhihu.com/question/22953267/answer/23192081)

