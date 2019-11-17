# 步骤：
## 1. 加载驱动、连接
```java
//JDBCUtil部分操作
public static Connection getMysqlConnection() throws ClassNotFoundException, SQLException {
    Class.forName("com.mysql.cj.jdbc.Driver");
    return DriverManager.getConnection("jdbc:mysql://localhost:3306/mydb?useSSL=false&serverTimezone=UTC",
            "root", "123456");
}
```
## 2. 语句
```java
String sql = "insert into users(username,password) values(?,?)";
PreparedStatement preparedStatement = null;
preparedStatement = conn.prepareStatement(sql);
preparedStatement.setString(1, user.getUsername());
```
## 3. 结果
```java
ResultSet rSet = ps.executeQuery();
while (rSet.next()) {
    User user = new User();
    user.setId(rSet.getInt(1));
    user.setUsername(rSet.getString(2));
    user.setPassword(rSet.getString(3));
    all.add(user);
}
//**.close()
return all;
```
## 4. 返回结果集：
- ps.execute()返回是否有结果集
- executeQuery()运行select语句返回ResultSet结果
- executeQuery();运行insert update delete返回修改行数

## 5. 关闭顺序:
**resultset→statement→conn**

> **三个关闭一定要分开用try catch关闭，否则一个出错其他无法关闭三个关闭一定要分开用try catch关闭，否则一个出错其他无法关闭**

# JDBCUtil：
## JDBC 工具类:
```java
public class JDBCUtil {
    public static Connection getMysqlConnection() throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.cj.jdbc.Driver");
        return DriverManager.getConnection("jdbc:mysql://localhost:3306/mydb?useSSL=false&serverTimezone=UTC",
                "root", "123456");
    }
    public static void close(Connection conn, ResultSet rs, PreparedStatement st) throws SQLException {
        st.close();
        rs.close();
        conn.close();
    }
}
```

## Properties存储常用数据：
随便新建一个配置文件（Test.properties）
```xml
name=JJ
Weight=4444
Height=3333
```
```java
//ServletContext context=this.getServletContext();
//filepath=context.getRealPath("/")+"//properties//1.properties";
 public class getProperties {
      public static void main(String[] args) throws FileNotFoundException, IOException {
          Properties pps = new Properties();
          pps.load(new FileInputStream("Test.properties"));
          Enumeration enum1 = pps.propertyNames();//得到配置文件的名字
          while(enum1.hasMoreElements()) {
              String strKey = (String) enum1.nextElement();
              String strValue = pps.getProperty(strKey);
              System.out.println(strKey + "=" + strValue);
         }
     }
 }
```

- ### 根据key读取value
- ### 读取properties的全部信息
- ### 写入新的properties信息
```java
//关于Properties类常用的操作
public class TestProperties {
    //根据Key读取Value
    public static String GetValueByKey(String filePath, String key) {
        Properties pps = new Properties();
        try {
            InputStream in = new BufferedInputStream (new FileInputStream(filePath));  
            pps.load(in);
            String value = pps.getProperty(key);
            System.out.println(key + " = " + value);
            return value;
            
        }catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }
    //读取Properties的全部信息
    public static void GetAllProperties(String filePath) throws IOException {
        Properties pps = new Properties();
        InputStream in = new BufferedInputStream(new FileInputStream(filePath));
        pps.load(in);
        Enumeration en = pps.propertyNames(); //得到配置文件的名字
        
        while(en.hasMoreElements()) {
            String strKey = (String) en.nextElement();
            String strValue = pps.getProperty(strKey);
            System.out.println(strKey + "=" + strValue);
        }

    }
    //写入Properties信息
    public static void WriteProperties (String filePath, String pKey, String pValue) throws IOException {
        Properties pps = new Properties();
        
        InputStream in = new FileInputStream(filePath);
        //从输入流中读取属性列表（键和元素对） 
        pps.load(in);
        //调用 Hashtable 的方法 put。使用 getProperty 方法提供并行性。  
        //强制要求为属性的键和值使用字符串。返回值是 Hashtable 调用 put 的结果。
        OutputStream out = new FileOutputStream(filePath);
        pps.setProperty(pKey, pValue);
        //以适合使用 load 方法加载到 Properties 表中的格式，  
        //将此 Properties 表中的属性列表（键和元素对）写入输出流  
        pps.store(out, "Update " + pKey + " name");
    }

    public static void main(String [] args) throws IOException{
        //String value = GetValueByKey("Test.properties", "name");
        //System.out.println(value);
        //GetAllProperties("Test.properties");
        WriteProperties("Test.properties","long", "212");
    }
}
```

## ORM:
表结构跟类对应，表中的字段和类的属性对应，表中的记录和对象对应表中记录封装到Object[]中，多条用List封装到Map<String,Object>中
```java
Map<String,Object> row=new HashMap<String,Object>();
```
读多条用List<Map<String,Object>>
```java
List<Map<String,Object>> list=new ArrayList<Map<String,Object>>();
list.add(...);
for(String key:row.keySet()){
    ....
}
```
### 封装到javabean中，实现Serializable接口（包装类，默认为Null）
如User类，实现getter、setter
```java
ResultSet:
rs.next()首先指向首条元组，有下一条一直返回true
rs.  //获取第n列数据
getInt(n)
getString(n)
```

**大量批处理用stmt不用ps：**
**ps预编译空间有限**
批处理部分：
```java
conn.setAutoCommit(false);//设为手动提交
//操作部分.....
conn.commit();
ps.addBatch(sql);//将给定的sql语句添加到ps对象命令列表中
ps.executeBatch();//批量执行
```
## CLOB-大数据：
从文件中读文本：
```java
ps.setClob(1,new FileReader(new File("1.txt")));
```
字符串读入CLOB:
```java
ps.setClob(1,new BufferedReader(new InputStreamReader(new ByteArrayInputSream("abcdefg".getBytes())));
```
读CLOB:
```java
ResultSet rs = stmt.executeQuery("SELECT * FROM Test1");
while(rs.next()){
    Clob c=rs.getClob("myInfo");
    Reader r=c.getCharacterStream();
    int temp=0;
    while((temp=r.read())!=-1){
        System.out.println(...))
    }
}
```
## BLOB-二进制大数据：（存图片等）
```java
ps.setBlob(1,new FileInputStream("d:\1.png"));
ps.execute();
```
### CLOB BLOB均以流的方式操作

## 事务：
**一组同时执行成功/失效的SQL语句，是数据库操作的一个执行单元**
### 四大特点(ACID):

1. atomicity	原子性
2. consistency	一致性
3. isolation	隔离性
4. durability	持久性

## 数据库中日期格式：
1. Java.sql.
2. Date(年月日)
3. Time(时分秒)
4. Timestamp(年月日时分秒)
