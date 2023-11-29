---
title: "在 Java Web 项目中使用 JDBC 连接数据库"
date: 2023-11-29T19:17:08+08:00
categories: Java
draft: false
---
要在 Java Web 项目中连接数据库，首先要准备数据库连接器。在 MySQL 官方网站上能够找到所有版本的数据库连接器。要在 Java Web 项目中连接数据库只需要使用官方提供的 [JDBC](https://dev.mysql.com/downloads/connector/j/ "Download MySQL Connector/J") ，在打开的网页中选择 Platform Independent 选项并下载即可。

<!--more-->

以 8.2.0 版本为例，在下载完成的压缩包中能够找到 `mysql-connector-j-8.2.0.jar` 文件，该 jar 文件提供了 Java 连接到数据库的所有实现方法。由此引申出一个问题「Jar 包是什么，该怎么用？」

维基百科如是写道：在软件领域，JAR 文件（Java 归档）是一种软件包文件格式，通常用于聚合大量的 Java 类文件、相关的元数据和资源文件到一个文件，以便分发 Java 平台应用软件或库。

这里涉及到了库的概念，JDBC 就是一个提供给 Java 的大型数据库操作库，通过使用 JDBC 提供的各种功能，能够实现使用 Java 对数据库的管理操作。下面演示如何在 Java Web 项目中导入和使用 JDBC 对数据库进行基本的查询操作。

工欲善其事，必先利其器。首先新建动态 Web 项目，配置好 Tomcat 服务器环境，然后再进入 JDBC 配置正题。

![新建动态 Web 项目](../../Paint/新建动态%20Web%20项目.png)

待项目新建完成，可以在 webapp 目录中找到 WEB-INF 文件夹，该文件夹中还存在单独的 lib 文件夹，用于保存在项目中需要访问的各种库，也就是所有的 Jar 包都需要保存在 lib 文件夹中。

![WEB-INF 文件夹](../../Paint/WEB-INF.png)

将 `mysql-connector-j-8.2.0.jar` 文件拖入到 lib 文件夹中，尝试启动服务器，服务器会检查并且读取该 Jar 包，将目光切换到新建的 `index.jsp` 文件，接下来就可以编写代码连接到数据库了。在此之前需要确保 MySQL 服务正在运行中，且所填入的信息均为正确可用的，下面是尝试使用 MySQL 获取数据并且使用表格方式输出的案例。

```jsp
<%@ page import="java.sql.ResultSet"%>
<%@ page import="java.sql.Statement"%>
<%@ page import="java.sql.SQLException"%>
<%@ page import="java.sql.DriverManager"%>
<%@ page import="java.sql.Connection"%>
<%@ page language="java" contentType="text/html; charset=UTF-8"
pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>JDBC 连接输出表格</title>
</head>

<body>
<%
//=== JDBC 创建连接的四个要素
//= DRIVER: 驱动类
//= HOST: 数据库地址
//= USERNAME: 用户名
//= PASSWORD: 密码
String DRIVER = "com.mysql.cj.jdbc.Driver";
String HOST = "jdbc:mysql://localhost:3306/test";
String USERNAME = "root";
String PASSWORD = "root";

// 加载 JDBC 驱动
Class.forName(DRIVER);

// 创建 Connection 对象
Connection connection = DriverManager.getConnection(HOST, USERNAME, PASSWORD);

// 创建 SQL 语句对象
Statement statement = connection.createStatement();

// 定义查询 SQL 语句
String sql = "SELECT * FROM stuInfo";

// 使用 ResultSet 数据集保存数据
ResultSet resultSet = statement.executeQuery(sql);
%>

    <table border=1 style="text-align: center">
        <caption>用户信息</caption>
            <tr>
            <td>编号</td>
            <td>姓名</td>
            <td>年龄</td>
            </tr>
            <% while(resultSet.next()) { %>
            <tr>
            <td><%= resultSet.getString("stuId") %></td>
            <td><%= resultSet.getString("stuName") %></td>
            <td><%= resultSet.getString("stuAge") %></td>
            </tr>
            <% } %>
    </table>
</body>
</html>
```

在案例代码中首先创建了四个字符串类型的变量，分别为 `DRIVER`、`HOST`、`USERNAME`、`PASSWORD`，下表描述这四个变量所保存的内容。

| 变量名   | 内容               |
| -------- | ------------------ |
| DRIVER   | 数据库驱动类       |
| HOST     | 数据库连接地址信息 |
| USERNAME | 数据库用户名       |
| PASSWORD | 数据库密码         |

DRIVER 变量可选的值存在两个，其一为 `com.mysql.cj.jdbc.Driver`，其二为 `com.mysql.jdbc.Driver`，需要依照当前使用的 MySQL 连接器的版本进行选择，版本大于 8.0 的需要使用前者，而版本小于 8.0 的则需要使用后者，本实例中使用的 JDBC 连接器版本为 8.2.0，自然需要使用前者，这是由于 JDBC 在 8.0 之后的版本对包结构的改变。

HOST 变量的写法大有学问，篇幅限制不展开细讲，只需要了解文中所使用的格式 `jdbc:mysql://<地址>:<端口>/<数据库名称>` 即可。一般情况访问本地的 MySQL 服务地址填写 localhost，端口填 3306，然后补充数据库名称。

USERNAME 和 PASSWORD 顾名思义账号密码，只需要填写能够正常登录和管理 MySQL 的账号和密码。

下一步进入到创建链接，这时需要使用 `Class.forName` 方法来加载 JDBC 驱动（这一步是非必须的，在 Java 6 之后 DriverManager 类能够自动检测和注册驱动），使用 Connection 来保存连接对象，并为连接对象创建语句（Statement）对象，完成这些操作之后才能够开始操作数据库。

```jsp
<%
Class.forName(DRIVER);
Connection connection = DriverManager.getConnection(HOST, USERNAME, PASSWORD);
Statement statement = connection.createStatement();
String sql = "SELECT * FROM  stuInfo";
ResultSet resultSet = statement.executeQuery(sql);
%>
```

`Class.forName(DRIVER);`: 这一行代码用于加载数据库驱动程序。`DRIVER`是一个字符串，应该包含你使用的数据库驱动的类名。这一步在Java 6以前是必需的，因为在那之后，`DriverManager`类能够自动检测和注册驱动。

`Connection connection = DriverManager.getConnection(HOST, USERNAME, PASSWORD);`: 这一行建立了与数据库的连接。`HOST`是数据库的URL，`USERNAME`和`PASSWORD`是连接数据库所需的用户名和密码。

`Statement statement = connection.createStatement();`: 这一行创建了一个`Statement`对象，用于执行SQL语句。`Statement`对象允许你向数据库发送SQL查询和更新。

`String sql = "SELECT * FROM stuInfo";`: 这一行定义了一个SQL查询语句，该语句从名为`stuInfo`的表中检索所有列的所有行。

最后使用 `resultSet` 对象来处理查询结果。后续可以通过循环遍历每一行，然后使用 `resultSet.getString("columnName")` 获取每列的值，其中 columnName 是查询结果中的列名。

这样的代码虽然有首，但是无尾。在使用完 resultSet、statement、connection 对象之后需要逐个关闭，首先是避免发生异常，再者是能够回收资源。

```jsp
<%
resultSet.close();
statement.close();
connection.close();
%>
```

有头有尾，实现了在 Java Web 项目中对 MySQL 数据库的查询操作。如果需要对数据库进行增删改，就需要使用 execute 方法进行操作，并且判断语句执行是否成功即可。
