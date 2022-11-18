# 第一个JDBC程序

导入数据库驱动

![image-20210401101714855](https://gitee.com/sgkurisu/pic-go/raw/master/picture/20210420111805.png)

```java
package com.jdbc;

import java.sql.*;

public class JdbcTest {
    public static void main(String[] args) throws ClassNotFoundException, SQLException {
        //1、加载驱动
        Class.forName("com.mysql.jdbc.Driver");  //固定写法

        //2、用户信息和URL    jdbc:mysql://主机:端口号/数据库名?useUnicode=true&characterEncoding=utf8&useSSL=true
        //useUnicode=true：支持中文编码
        //characterEncoding=utf8：设置编码字符为utf8
        //useSSL=true：使用安全连接
        String url = "jdbc:mysql://localhost:3306/db_studentinfo?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf8&useSSL=false";
        String name = "root";
        String password = "x";

        //3、连接成功，数据库对象
        Connection connection = DriverManager.getConnection(url, name, password);

        //4、执行sql对象
        Statement statement = connection.createStatement();

        //5、使用sql对象执行sql语句，查看结果
        String sql = "select * from student";
        //返回的为链表
        ResultSet resultSet = statement.executeQuery(sql); //查询Query，更新Update
        while (resultSet.next()) {
            System.out.println("stuNo:" + resultSet.getObject("stuNo"));
            System.out.println("Name:" + resultSet.getObject("Name"));
            System.out.println("Password:" + resultSet.getObject("Password"));
            System.out.println("Major:" + resultSet.getObject("Major"));
            System.out.println("Class:" + resultSet.getObject("Class"));
            System.out.println("Phone:" + resultSet.getObject("Phone"));
        }

        //6、释放连接
        resultSet.close();
        statement.close();
        connection.close();

    }
}
```

> 加载驱动

```java
// DriverManager.registerDriver(new com.mysql.jdbc.Driver());不建议使用
Class.forName("com.mysql.jdbc.Driver");  //固定写法
```

> URL

```java
String url = "jdbc:mysql://localhost:3306/db_studentinfo?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf8&useSSL=false";
//serverTimezone为区域时间设置，详见另一篇博客
//mysql默认端口号为3306
//协议://主机:端口号/数据库?参数1&参数2&参数3
```

> Statement

```java
String sql = "select * from student"; //编写sql语句
statement.executeQuery(); //查询
statement.executeUpdate(); //更新、增加、删除，返回影响的行数
statement.execute(); //执行任何sql语句，返回boolean值
```

> ResultSet 查询结果集，返回查询的结果

```java
resultSet.afterLast(); //移动到最后面
resultSet.beforeFirst(); //移动到最前端
resultSet.next(); //移动到下一个
resultSet.absolute(x); //移动到任意行
```

# Statement接口

Statement对象用于向数据库发送sql语句

```java
Statement.executeUpdate(); //增删改
Statement.executeQuery(); //查询
```

由于加载驱动、URL和用户信息、连接数据库、释放资源每次操作均相同，因此写配置文件

```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/db_studentinfo?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf8&useSSL=true
username=root
password=@Kurisu&0725
```

再写工具类

```java
package com.demo.utils;

import java.io.IOException;
import java.io.InputStream;
import java.sql.*;
import java.util.Properties;

public class JdbcUtiles {

    private static String driver = null;
    private static String url = null;
    private static String username = null;
    private static String password = null;

    static {

        try {
            InputStream in = JdbcUtiles.class.getClassLoader().getResourceAsStream("db.properties");
            Properties properties = new Properties();
            properties.load(in);

            driver = properties.getProperty("driver");
            url = properties.getProperty("url");
            username = properties.getProperty("username");
            password = properties.getProperty("password");

            //加载一次驱动
            Class.forName(driver);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }

    }

    // 获取连接
    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(url, username, password);
    }

    //释放资源
    public static void release(Connection connection, Statement statement, ResultSet resultSet) {
        if(resultSet != null) {
            try {
                resultSet.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if(statement != null) {
            try {
                statement.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if(connection != null) {
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }

}
```

然后每次直接调用连接getConnection和释放资源release即可

```java
package com.demo;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

import static com.demo.utils.JdbcUtiles.getConnection;
import static com.demo.utils.JdbcUtiles.release;

public class Test2 {
    public static void main(String[] args) {
        Connection connection = null;
        Statement statement = null;
        ResultSet resultSet = null;

        try {
            connection = getConnection();
            statement = connection.createStatement();
            String sql1 = "INSERT INTO `grade`(`id`, `grade`) VALUES('1', '60'),('2', '70');";
            int i = statement.executeUpdate(sql1);
            if (i > 0){
                System.out.println("插入成功");
            }

        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            release(connection, statement, resultSet);
        }
    }
}
```

# SQL注入问题

```java
package com.demo;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

import static com.demo.utils.JdbcUtiles.getConnection;
import static com.demo.utils.JdbcUtiles.release;

public class Login {
    public static void main(String[] args) {
        //正常
        //new Login().TestLogin("1");
        //非正常
        new Login().TestLogin(" ' ' or 1=1"); //执行1=1，可查所有信息，存在漏洞
    }

    public void TestLogin(String id) {
        Connection connection = null;
        Statement statement = null;
        ResultSet resultSet = null;

        try {
            connection = getConnection();
            statement = connection.createStatement();
            //"SELECT * FROM `grade` WHERE `id`=1;"
            String sql = "SELECT * FROM `grade` WHERE `id`=" + id;
            resultSet = statement.executeQuery(sql);
            while (resultSet.next()) {
                System.out.println("id = " + resultSet.getObject("id") + "   grade = " + resultSet.getObject("grade"));
            }

        } catch (SQLException throwables) {
            throwables.printStackTrace();
        } finally {
            release(connection, statement, resultSet);
        }
    }
}
```

# PreparedStatement

PreparedStatement可用来防止SQL注入问题，将传进来的参数作为字符处理，若传入存在转义字符，则直接忽略

```java
package com.demo;

import java.sql.*;

import static com.demo.utils.JdbcUtiles.getConnection;
import static com.demo.utils.JdbcUtiles.release;

public class Login2 {
    public static void main(String[] args) {
        //正常
        //new Login2().TestLogin(1);
        //非正常
        new Login2().TestLogin(" ' ' or 1=1"); //执行1=1，可查所有信息，存在漏洞
    }

    public void TestLogin(String id) {
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;

        try {
            connection = getConnection();
            //"SELECT * FROM `grade` WHERE `id`=1;"
            String sql = "SELECT * FROM `grade` WHERE `id`=?";
            preparedStatement = connection.prepareStatement(sql);
            
            preparedStatement.setString(1, id); //将不会出现SQL注入的漏洞
            resultSet = preparedStatement.executeQuery();
            while (resultSet.next()) {
                System.out.println("id = " + resultSet.getObject("id") + "   grade = " + resultSet.getObject("grade") + "  date:" + resultSet.getObject("date"));
            }

        } catch (SQLException throwables) {
            throwables.printStackTrace();
        } finally {
            release(connection, preparedStatement, resultSet);
        }
    }
}
```

# IDEA连接Database

IDEA Community 找不到Database

![image-20210403192528648](https://gitee.com/sgkurisu/pic-go/raw/master/picture/20210420111813.png)

需要安装Database插件

![image-20210403192630775](https://gitee.com/sgkurisu/pic-go/raw/master/picture/20210420111815.png)

加载完重启IDEA后

![image-20210403193158058](E:\JavaStudy\JavaSE\image-20210403193158058.png)

![image-20210403193306777](https://gitee.com/sgkurisu/pic-go/raw/master/picture/20210420111818.png)



![image-20210403193417440](https://gitee.com/sgkurisu/pic-go/raw/master/picture/20210420111821.png)

连接失败时注意时区问题，可进入mysql修改全局时区，然后再次连接即可

![image-20210403195842111](https://gitee.com/sgkurisu/pic-go/raw/master/picture/20210420111823.png)

# JDBC操作事务

ACID原则：原子性、一致性、隔离性、持久性

隔离性：多个线程互不干扰。由此产生的问题：脏读，不可重复读，虚读（幻读）

```java
package com.demo;

import com.demo.utils.JdbcUtiles;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class TestTransaction {
    public static void main(String[] args) {

        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;

        try {
            connection = JdbcUtiles.getConnection();
            //关闭自动提交
            connection.setAutoCommit(false);

            String sql1 = "update `grade` set `grade` = `grade` + 10 where id = 1 ";
            preparedStatement = connection.prepareStatement(sql1);
            preparedStatement.executeUpdate();

            String sql2 = "update `grade` set `grade` = `grade` - 10 where id = 2 ";
            preparedStatement = connection.prepareStatement(sql2);
            preparedStatement.executeUpdate();

            //int x = 1/0; //主动测试失败

            //业务完毕，提交事务
            connection.commit();
            System.out.println("事务成功");


        } catch (SQLException e) {
            //事务失败，回滚
            try {
                connection.rollback();
                System.out.println("事务失败");
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        } finally {

            try {
                connection.setAutoCommit(true);
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
            JdbcUtiles.release(connection, preparedStatement, resultSet);
        }

    }
}

```

# 数据库连接池

数据库连接，释放十分浪费资源，因此引出池化技术

最小连接数、最大连接数、等待超时

编写连接池，需实现接口DataSource

开源数据源实现DataSource：DBCP，C3P0，Druid

> DBCP

下载地址

commons-dbcp.jar

http://commons.apache.org/proper/commons-dbcp/

commons-pool.jar

http://commons.apache.org/proper/commons-pool/

dbcpconfig.properties

```properties
#连接设置
driverClassName=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/db_studentinfo?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf8&useSSL=true
username=root
password=@Kurisu&0725

#初始化连接
initialSize=10

#最大连接
maxActive=50

#最大空闲连接
maxIdle=20

#最小空闲连接
minIdle=5

#超时等待 单位毫秒
maxWait=2000
```

DbcpUtiles.java

```java
package com.demo.utils;

import org.apache.commons.dbcp.BasicDataSourceFactory;

import javax.sql.DataSource;
import java.io.IOException;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Properties;

public class DbcpUtiles {

    private static DataSource dataSource = null;
    static {

        try {
            InputStream in = DbcpUtiles.class.getClassLoader().getResourceAsStream("dbcpconfig.properties");

            Properties properties = new Properties();
            properties.load(in);

            //创建数据源   工厂模式-->创建对象
            dataSource = BasicDataSourceFactory.createDataSource(properties);

        } catch (IOException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static Connection getConnection() throws SQLException {
        return dataSource.getConnection(); //从数据源获取连接
    }

    //释放资源
    public static void release(Connection connection, Statement statement, ResultSet resultSet) {
        if(resultSet != null) {
            try {
                resultSet.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if(statement != null) {
            try {
                statement.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if(connection != null) {
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }


}
```

使用相同，只不过是数据源不同

```java
package com.demo;

import com.demo.utils.DbcpUtiles;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class TestDbcp {
    public static void main(String[] args) {
        Connection connection = null;
        Statement statement = null;
        ResultSet resultSet = null;

        try {
            connection = DbcpUtiles.getConnection(); //由DBCP提供数据源
            statement = connection.createStatement();
            String sql1 = "INSERT INTO `grade`(`id`, `grade`) VALUES('1', '60'),('2', '70');";
            int i = statement.executeUpdate(sql1);
            if (i > 0){
                System.out.println("插入成功");
            }

        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            DbcpUtiles.release(connection, statement, resultSet);
        }
    }
}
```

> C3P0

C3P0使用的配置文件是.xml

```java
<c3p0-config>
    <!-- 使用默认的配置读取连接池对象 -->
    <default-config>
        <!--  连接参数 -->
        <property name="driverClass">com.mysql.jdbc.Driver</property>
        <property name="jdbcUrl">jdbc:mysql://localhost:3306/db_studentinfo?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf8&useSSL=true</property>
        <property name="user">root</property>
        <property name="password">@Kurisu&0725</property>

        <!-- 连接池参数 -->
        <!--初始化的申请的连接数量-->
        <property name="initialPoolSize">10</property>
        <!--最大的连接数量-->
        <property name="maxPoolSize">20</property>
        <!--连接超时时间-->
        <property name="checkoutTimeout">3000</property>
    </default-config>

    <named-config name="mysql">
        <!--  连接参数 -->
        <property name="driverClass">com.mysql.jdbc.Driver</property>
        <property name="jdbcUrl">jdbc:mysql://localhost:3306/db_studentinfo?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf8&useSSL=true</property>
        <property name="user">root</property>
        <property name="password">@Kurisu&0725</property>

        <!-- 连接池参数 -->
        <!--初始化的申请的连接数量-->
        <property name="initialPoolSize">10</property>
        <!--最大的连接数量-->
        <property name="maxPoolSize">20</property>
        <!--连接超时时间-->
        <property name="checkoutTimeout">3000</property>
    </named-config>
</c3p0-config>
```

工具类只需改

```java
//不写参数，则为xml文件中的默认配置，加上mysql后则为自主配置的内容
dataSource = new ComboPooledDataSource("mysql");
```













