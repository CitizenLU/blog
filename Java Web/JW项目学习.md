# 1 IDEA 新建一个Tomcat项目



![image-20221220162800525](https://images-lu.oss-cn-shanghai.aliyuncs.com/202212201628581.png)



# 2 一个简单的Servlet



```java
@WebServlet(urlPatterns = "/login")
public class LoginServlet implements Servlet {
    @Override
    public void init(ServletConfig servletConfig) throws ServletException {
        System.out.println("Login 初始化");
    }

    @Override
    public ServletConfig getServletConfig() {
        return null;
    }

    @Override
    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
        String name = req.getParameter("name");
        // 模拟系统处理请求
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        PrintWriter out = res.getWriter();
        out.print("Hello " + name);
    }

    @Override
    public String getServletInfo() {
        return null;
    }

    // 销毁时，发出蜂鸣
    @Override
    public void destroy() {
        Toolkit.getDefaultToolkit().beep();
    }
}
```

Servlet对象由Tomcat服务器负责管理，Tomcat服务器通过读取web.xml创建并运行Servlet对象。

Servlet对象是javax.servlet包中HttpServlet类的子类的一个实例，由服务器负责创建并完成初始化工作。

服务器为每一个用户请求，启动一个对Servlet对象的访问线程。但只有一个Servlet对象为每个请求线程进行服务。

特点：单实例，多线程。

启动时，只会实例化一个LoginServlet，多个客户端的请求，都由同一个LoginServlet对象进行处理的。

例如：如果将示例中的变量name，更改为实例变量时，会产生如下的问题。第二个请求中name为Py，但返回的结果是Bob。

![image-20221220165614397](https://images-lu.oss-cn-shanghai.aliyuncs.com/202212201656430.png)



# 3 ServletConfig 

把返回的值换成中文，则打印的就会有问题。

```java
    @Override
    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
        String name = req.getParameter("name");
        // ...
        PrintWriter out = res.getWriter();
        // 改成中文
        out.print("你好 " + name);
    }
```

![image-20221220192918974](https://images-lu.oss-cn-shanghai.aliyuncs.com/202212201929010.png)

将charset修改为utf-8即可满足需求，修改为如下代码

```java
    @Override
    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
        String name = req.getParameter("name");
        // ...
        res.setContentType("text/html;charset=utf-8");
        res.setCharacterEncoding("utf-8");
        PrintWriter out = res.getWriter();
        // 改成中文
        out.print("你好 " + name);
    }
```

进一步优化。

```java
package login.servlets;

// 配置项charset:utf-8
@WebServlet(urlPatterns = "/login", initParams = {@WebInitParam(name = "charset", value = "utf-8")})
public class LoginServlet extends HttpServlet {
    String charset = null;

    @Override
    public void init(ServletConfig config) throws ServletException {
        System.out.println("Login 初始化");
        // 读取配置项
        this.charset = config.getInitParameter("charset");
    }

    @Override
    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
        String name = req.getParameter("name");
        res.setContentType("text/html;charset=" + this.charset);
        res.setCharacterEncoding(this.charset);
        PrintWriter out = res.getWriter();
        out.print("你好 " + name);
    }

    @Override
    public void destroy() {
        Toolkit.getDefaultToolkit().beep();
    }
}

```

进一步优化，取消`@WebServlet(urlPatterns = "/login", initParams = {@WebInitParam(name = "charset", value = "utf-8")})`这行注解

在web.xml中设置Servlet映射。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <display-name>jw</display-name>
    <welcome-file-list>
        <welcome-file>index.html</welcome-file>
    </welcome-file-list>
    <servlet>
        <servlet-name>login</servlet-name>
        <servlet-class>login.servlets.LoginServlet</servlet-class>
        <init-param>
            <param-name>charset</param-name>
            <param-value>utf-8</param-value>
        </init-param>
        <!-- 在服务器启动时，加载类的优先级 -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>login</servlet-name>
        <url-pattern>/login</url-pattern>
    </servlet-mapping>
</web-app>
```

# 4 第一个接口

首先，创建一个数据库jw，创建表t_admins，有3个字段，分别是id、login_name和pwd。

该接口需要做的是：接收请求中的loginName和pwd字段，从表t_admins中寻找一条记录，满足login_name=loginName且pwd=pwd。如果有这么一条记录，则为登录成功，否则登录失败。

![image-20221225005102657](https://images-lu.oss-cn-shanghai.aliyuncs.com/202212250051746.png)

做法：

## 1、连接数据库。这里使用了一个ConnUtil的工具类，用于连接数据库。

先回顾一下，正常的连接数据库和查询的操作。

```java
public class DB {
    // 数据库的连接
    private Connection connection = null;

    public DB() {
        try {
            // 1.加载mysql数据库的驱动，加载的是mysql-connector-j的jar包
            Class.forName("com.mysql.cj.jdbc.Driver");
            // 2.连接数据库，参数分别为：数据库url，连接的用户名，连接的密码
            connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/jw", "root", "123456");
            System.out.println("Success connect to Mysql" + connection);
        } catch (ClassNotFoundException | SQLException e) {
            e.printStackTrace();
        }
    }

    // 查找数据
    public void query(String loginName, String pwd) {
        String sql = "select * from t_admins where login_name=? and pwd=?";
        try {
            PreparedStatement pst = connection.prepareStatement(sql);
            // 使用参数化sql语句
            pst.setString(1, "admin");
            pst.setString(2, "123456");
            // 执行查询的sql语句，executeQuery会返回结果集
            ResultSet rst = pst.executeQuery();
            // 结果集提供next方法来达到遍历的效果。
            while (rst.next()) {
                String id = rst.getString("id");
                String name = rst.getString("login_name");
                String password = rst.getString("pwd");
                System.out.println("id: " + id + "; name: " + name + "; password: " + password);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        DB db = new DB();
        db.query("admin", "123456");
    }
}
```

可见，连接数据库，需要提供数据库url，连接的用户名和密码，这3个参数。

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.sql.ResultSet;

public class ConnUtil {
    static String user;
    static String pwd;
    static String url;

    static {
        // 该类用于解析properties文件中的数据
        Properties p = new Properties();
        // 参数放在配置文件中，类加载的时候会执行，且只执行一次，避免重复读取
        InputStream in = ConnUtil.class.getResourceAsStream("/db.properties");
        try {
            p.load(in);
            String driverName = p.getProperty("driverName");
            Class.forName(driverName);
            user = p.getProperty("user");
            pwd = p.getProperty("pwd");
            url = p.getProperty("url");
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

    public static Connection getConn() throws SQLException {
        return DriverManager.getConnection(url,user,pwd);
    }
}
```

## 2、数据库的模板，用于做查询后的断开连接、做查询的模板等。

```java
public class JdbcTemplate {
    // 提供了一个查询的模板
    public static <T> T select(String sql, ResultSetExtractor<T> ext, Object... args) throws SQLException {
        T t = null;
        Connection conn = null;
        PreparedStatement stm = null;
        ResultSet rst = null;
        try {
            // 从工具类中，获取一个数据库的连接
            conn = ConnUtil.getConn();
            // 根据可变参数列表，拼接sql语句
            stm = conn.prepareStatement(sql);
            if (args != null) {
                for (int i = 0; i < args.length; i++) {
                    stm.setObject(i + 1, args[i]);
                }
            }
            rst = stm.executeQuery();
            // 调用ResultSetExtractor的extract方法，而该方法由DAO层具体实现
            t = ext.extract(rst);
        } catch (Exception e) {
            // 处理异常，可以处理事务
            e.printStackTrace();
            throw e;
        } finally {
            // 断开连接
            if (rst != null) {
                rst.close();
            }
            if (stm != null) {
                stm.close();
            }
            if (conn != null) {
                conn.close();
            }
        }
        return t;
    }
}
```

## 3、DAO层对象：AdminDAO。实现了ResultSetExtractor的extract方法。这样就可以知道，从数据库中获取的数据，将要如何展示。

DAO是数据库访问对象，是一个面向对象的数据库接口，抽象了数据库的访问。

```java
public class AdminDAO {
    // 数据库的查询操作
    public static AdminUser find(String loginName, String pwd) throws SQLException {
        AdminUser u = null;
        // 从结果集中提取结果
        ResultSetExtractor<AdminUser> ext = (rst) -> {
            AdminUser ur = null;
            if (rst.next()) {
                // 从结果集中获取字段信息，并用AdminUser这样的bean包装起来。
                int id = rst.getInt("id");
                String loginName2 = rst.getString("login_name");
                String pass = rst.getString("pwd");
                ur = new AdminUser(id, loginName2, pass);
            }
            return ur;
        };
        
        String sql = "select * from t_admins where login_name=? and pwd=?";
        u = JdbcTemplate.select(sql, ext, loginName, pwd);
        return u;
    }
}
```

## 4、JavaBean。从数据库中查询到了东西，需要用对象包装保存起来。

包含了需要的字段，以及对应的get/set/constructor方法，重写toString，方便日志打印的需要。

```java
public class AdminUser {
    private int ID;
    private String loginName;
    private String pwd;

    public AdminUser() {
    }

    public AdminUser(int ID, String loginName, String pwd) {
        this.ID = ID;
        this.loginName = loginName;
        this.pwd = pwd;
    }

    public int getID() {
        return ID;
    }

    public void setID(int ID) {
        this.ID = ID;
    }

    public String getLoginName() {
        return loginName;
    }

    public void setLoginName(String loginName) {
        this.loginName = loginName;
    }

    public String getPwd() {
        return pwd;
    }

    public void setPwd(String pwd) {
        this.pwd = pwd;
    }

    @Override
    public String toString() {
        return "AdminUser{" +
                "ID=" + ID +
                ", loginName='" + loginName + '\'' +
                ", pwd='" + pwd + '\'' +
                '}';
    }
}
```

以上4个步骤，即可从一个数据库表中读取数据，并用一个JavaBean来保存需要的数据。

## 5、service层。

简单的系统，service不需要有太多的代码，如果业务罗技比较复杂，则需要在此做处理。

本接口比较简单，所以service层只有一个调用dao层的方法。

```java
public class AdminService {
    public static AdminUser adminLogin(String loginName, String pwd) throws SQLException {
        return AdminDAO.find(loginName, pwd);
    }
}
```

## 6、Controller层。这一层，也是Web服务器接收处理客户端请求的入口。

这一层，主要是实现了Servlet接口。现在一般继承HttpServlet抽象类，该类实现了Servlet接口。继承了HttpServlet的类或实现了Servlet接口的类，它们实例化的对象，一般也称为Servlet对象。

Servlet是运行在web服务器端的应用程序，Servlet对象封装了对HTTP请求的处理，它的运行需要Servlet容器（如Tomcat）的支持。

Servlet生命周期包括：init初始化->service处理客户端请求->destroy销毁。这3个阶段。HttpServlet

HttpServlet已经做了初始化和销毁，在初始化阶段默认读取ServiceConfig配置文件。

所以，HttpServlet重点关注处理客户端请求这个方法。

```java
import beans.AdminUser;
import login.service.AdminService;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.sql.SQLException;

@WebServlet("/login")
public class Login extends HttpServlet {
    private static final long serialVersionUID = 1L;

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 从请求中获取到用户名和密码的参数
        String loginName = request.getParameter("loginName");
        String pwd = request.getParameter("pwd");
        AdminUser user = null;
        PrintWriter out = response.getWriter();
        response.setContentType("text/html;charset=utf-8");
        response.setCharacterEncoding("utf-8");
        try {
            // 调用service层的方法，具体做业务
            user = AdminService.adminLogin(loginName, pwd);
        } catch (SQLException e) {
            e.printStackTrace();
            out.print("账户查询出错!!");
            return;
        }
        // 根据service层的返回结果，返回不同的结果给客户端
        if (null != user) {
            out.print("登录成功!");
        } else {
            out.print("用户名或密码不正确!!!");
        }

    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request, response);
    }
}
```



# 5 实现登录jsp页面

Node.js是一个基于Chrome V8引擎的JavaScript运行环境。

Bootstrap是一个前端框架。

安装node.js和bootstrap之后，通过如下语言将jQuery和bootstrap引入到页面中。

```html
    <script src="js/bower_components/jquery/dist/jquery.js"></script>
    <script src="js/bower_components/bootstrap/dist/js/bootstrap.js"></script>
    <link rel="stylesheet" href="js/bower_components/bootstrap/dist/css/bootstrap.css">
```

# 6 jsp基本原理

Java Server Pages作为servlet的替代。

在servlet中，返回给用户的响应的内容，是通过硬编码，使用printWriter返回的HTML文本内容。

使用这种方式来响应用户的请求，最大的缺点是，实现复杂的HTML页面较困难。

在服务器中，有2个引擎。一个是servlet引擎，另一个是jsp引擎。jsp引擎是jsp文件的容器，负责将jsp转换成servlet，并提供对jsp请求的访问控制。

> jsp页面的基本结构：
>
> 普通HTML标记
>
> jsp标记（jsp指令，jsp动作）
>
> 变量和方法声明
>
> Java表达式

jsp指令是为jsp引擎设计的，并不直接产生任何可见输出，而只是告诉引擎如何处理其余jsp页面，这些指令始终放在`<%@ %>`标记中，两个最重要的指令是：“pagePage”和“Include”。

语法格式，可以有一个或多个属性

```jsp
<%@ 指令 属性="值"%>
<%@ directive attribute1="value1" attribute2="value2" attributeN="valueN"%>
```

![JSP工作原理](https://images-lu.oss-cn-shanghai.aliyuncs.com/202212272340021.jpg)

jsp页面中的代码

使用标记：

```jsp
<%! %>

<%!
  // 成员变量
%>

<%
  // 局部变量
%>

<%
 // Java程序片
%>

// 表达式
<%=name %>

<%-- 注释内容 --%>
```

jsp动作

```jsp
<jsp:>
```



# 7 过滤器

Servlet2.3规范的一个非常重要的功能就是过滤器功能。

![image-20221231024214580](https://images-lu.oss-cn-shanghai.aliyuncs.com/image-20221231024214580.png)

具体的说，过滤器是一个简单的类，它可以实现`javax.servlet.Filter`接口。

过滤器实现类必须实现的3个接口方法：

1、`doFilter(ServletRequest, ServletResponse, FilterChain)`

这是过滤器的主要方法。过滤器所需要完成的整个工作基本上由该方法来完成。

2、`init(FilterConfig)`

在初始化过滤器前，被调用的方法，负责设置Filter对象。

3、`destroy()`

当服务器不需要过滤器时，调用该方法。



# 8 JQuery

jQuery是一个JavaScript函数库，简化了JavaScript编程。

jQuery库包含了以下特性：

HTML元素选取

HTML元素操作

CSS操作

HTML事件函数

JavaScript特效与动画

HTML DOM遍历和修改

AJAX

Utilities













































