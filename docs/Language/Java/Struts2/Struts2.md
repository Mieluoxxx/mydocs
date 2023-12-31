---
title: Struts2入门
permalink: /docs/Java/Struts2
---

## 第一个Struts2项目

### MVC设计模式

MVC设计模式是软件工程中的一种软件架构模式,其把1软件系统分为3个基本部分:控制器(controller),视图(View)和模型(Model)

(1) 控制器 -- 负责转发请求,对请求进行处理

(2) 视图 -- 提供用户交互界面

(3) 模型 -- 功能接口,用于编写程序的业务逻辑功能,数据库访问功能等

### 编写web.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <filter>
        <filter-name>struts2</filter-name>
        <filter-class>org.apache.struts2.dispatcher.filter.StrutsPrepareAndExecuteFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>struts2</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```

### 编写HelloStrutsAction

```java
public class HelloStrutsAction extends ActionSupport {
    public String execute() throws Exception {
        return "success";
    }
}
```

### 配置struts.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE struts PUBLIC
        "-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"
        "http://struts.apache.org/dtds/struts-2.5.dtd">

<struts>
    <package name="default" namespace="/" extends="struts-default">
        <action name="hello" class="com.struts.action.HelloStrutsAction">
<!--            定义处理结果（动作返回值）与视图资源之间的对应关系-->
            <result name="success">/success.jsp</result>
        </action>
    </package>
</struts>
```

### 配置index.jsp

```jsp
<%@ page contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<!DOCTYPE html>
<html>
<head>
  <title>JSP - Hello World</title>
</head>
<body>
<h1><%= "Hello World!" %></h1>
<br/>
<a href="${pageContext.request.contextPath}/hello.action">Hello Servlet</a>
</body>
</html>
```

### 配置success.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
成功！
</body>
</html>

```

## Struts2实现登陆功能

### 编写login.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<form action="login" method="post">
    用户名: <input type="text" name="username"><br>
    密码: <input type="password" name="password"><br>
    <input type="submit" value="登录">
</form>
</body>
</html>

```

### 编写LoginAction

```java
@Getter
@Setter
public class LoginAction extends ActionSupport {
    private String username;
    private String password;
    @Override
    public String execute() throws Exception {
        if (this.username.equals("admin") && this.password.equals("admin")) {
            return "success";
        } else {
            return "error";
        }
    }
}
```

### 在struts.xml中添加action

```xml
<action name="login" class="com.struts.action.LoginAction">
    <result name="success">/successLogin.jsp</result>
    <result name="error">/errorLogin.jsp</result>
</action>
```

### 编写successLogin.jsp

```jsp
<%@ taglib prefix="s" uri="/struts-tags" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<h1>Success</h1>
<h2>欢迎您：<s:property value="username"/> !</h2>
</body>
</html>

```

### 编写errorLogin.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<h2>登陆失败</h2>
</body>
</html>
```


## Action 访问 Servlet API

### ActionContext

#### 配置login.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<form action="${pageContext.request.contextPath}/login" method="post">
    用户名: <input type="text" name="username"><br>
    密码: <input type="password" name="password"><br>
    <input type="submit" value="登录">
</form>
</body>
</html>
```

#### 配置struts.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE struts PUBLIC
        "-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"
        "http://struts.apache.org/dtds/struts-2.5.dtd">

<struts>
    <package name="default" namespace="/" extends="struts-default">
        <action name="login" class="com.struts.action.LoginAction">
            <result name="success">/success.jsp</result>
            <result name="error">/error.jsp</result>
        </action>
    </package>
</struts>
```

#### 配置LoginAction.java

```
@Getter
@Setter
public class LoginAction extends ActionSupport {
    private String username;
    private String password;
    @Override
    public String execute() throws Exception {
        // 通过ActionContext静态方法获取ActionContext对象
        ActionContext context = ActionContext.getContext();
        Map<String, Object> session = context.getSession();
        if (username.equals("admin") && password.equals("admin")) {
            session.put("username", username);
            session.put("password", password);
            session.put("success", "登录成功!");
            return SUCCESS;
        } else {
            session.put("error", "用户名或密码错误!");
            return ERROR;
        }
    }
}
```

#### 配置success.jsp

```xml
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Success页面</title>
</head>
<body>
<h1>${sessionScope.success}</h1>
<h3>用户名：${sessionScope.username}</h3>
<h3>密码：${sessionScope.password}</h3>
</body>
</html>

```

#### 配置error.jsp

```xml
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>errorLogin</title>
</head>
<body>
<h2>${sessionScope.error}</h2>
</body>
</html>
```

### ServletRequestAware

#### 编写MyAwareAction

```java
public class MyAwareAction extends ActionSupport implements ServletRequestAware {
    HttpServletRequest request;
    @Override
    public void setServletRequest(HttpServletRequest request) {
        this.request = request;
    }

    @Override
    public String execute() throws Exception {
        request.setAttribute("message", "直接访问Servlet API成功！");
        return SUCCESS;
    }
}
```

#### 编写struts.xml

```xml
<action name="message" class="com.struts.action.MyAwareAction">
<result name="success">/message.jsp</result>
</action>
```

#### 编写message.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Message页面</title>
</head>
<body>
<h2>${requestScope.message}</h2>
</body>
</html>

```

### ServletActionContext

#### 编写InfoAction.java

```java
public class InfoAction extends ActionSupport {
    @Override
    public String execute() throws Exception {
        ServletActionContext.getRequest().setAttribute("Info", "通过ServletActionContext直接访问Servlet API成功！");
        return SUCCESS;
    }
}
```

#### 编写struts.xml

```xml
<action name="info" class="com.struts.action.InfoAction">
<result name="success">/info.jsp</result>
</action>
```

#### 编写info.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Info页面</title>
</head>
<body>
<h2>${requestScope.Info}</h2>
</body>
</html>

```

## 动态方法调用

动态方法调用解决的问题是：在Web应用中struts.xml可能需要配置大量Action，这会使得struts.xml变得非常臃肿。



动态方法调用是指表单的Action属性不是直接使用某个Action的名称，而是用一种特定的格式。语法如下

```java
actionName!methodName.action
```

其中第一个单词actionName代表Action的名称，“!”右边的methodName代表方法名称，例如`user!login.action`表示调用user这个Action的`login()`方法。



### 动态方法调用实现登陆

#### 编写UserAction.java

```java
@Getter
@Setter
public class UserAction extends ActionSupport {
    private String username;
    private String password;

    public String login() throws Exception {
        ActionContext context = ActionContext.getContext();
        Map<String, Object> session = context.getSession();
        if (username.equals("admin") && password.equals("123456")) {
            session.put("username", username);
            session.put("password", password);
            session.put("success", "登录成功！");
            return SUCCESS;
        } else {
            session.put("errorLogin", "用户名或密码错误！");
            return ERROR;
        }
    }

    public String register() throws Exception {
        ActionContext context = ActionContext.getContext();
        Map<String, Object> session = context.getSession();
        session.put("username", username);
        session.put("password", password);
        session.put("success", "注册成功！");
        return SUCCESS;
    }
}
```

#### 编写login.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <form action="user!login.action" method="post">
        用户名：<input type="text" name="username"><br>
        密码：<input type="password" name="password"><br>
        <input type="submit" value="登录">
    </form>
</body>
</html>
```

#### 编写register.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <form action="user!register" method="post">
        用户名：<input type="text" name="username"><br>
        密码：<input type="password" name="password"><br>
        <input type="submit" value="注册">
    </form>
</body>
</html>
```

#### 编写Struts.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE struts PUBLIC
        "-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"
        "http://struts.apache.org/dtds/struts-2.5.dtd">

<struts>
    <!--启用动态方法-->
    <constant name="struts.enable.DynamicMethodInvocation" value="true"></constant>
    <package name="default" namespace="/" extends="struts-default">
        <default-action-ref name="defaultAction"/>
        <global-allowed-methods>regex:.*</global-allowed-methods>
        <action name="defaultAction" >
            <result>/error.jsp</result>
        </action>

        <action name="login" class="com.struts.action.LoginAction">
            <result name="success">/success.jsp</result>
        </action>

<!--        MyAwareAction-->
        <action name="message" class="com.struts.action.MyAwareAction">
            <result name="success">/message.jsp</result>
        </action>

<!--        InfoAction-->
        <action name="info" class="com.struts.action.InfoAction">
            <result name="success">/info.jsp</result>
        </action>

<!--        UserAction类对应的Action配置-->
        <action name="userLogin" class="com.struts.action.UserAction">
            <result name="success">/successLogin.jsp</result>
            <result name="error">/errorLogin.jsp</result>
        </action>

<!--        使用动态方法-->
        <action name="user" class="com.struts.action.UserAction">
            <result name="success">/success.jsp</result>
            <result name="error">/error.jsp</result>
        </action>
    </package>
</struts>
```

#### 编写success.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Success页面</title>
</head>
<body>
<h1>${sessionScope.success}</h1>
<h3>用户名：${sessionScope.username}</h3>
<h3>密码：${sessionScope.password}</h3>
</body>
</html>
```



### 使用通配符来简化配置

```xml
<action name="user_*" class="com.struts.action.UserAction" method="{1}">
<result name="success">/{1}Success.jsp</result>
<result name="error">/error.jsp</result>
</action>
```



### 配置默认Action

```xml
<action name="defaultAction" >
<result>/error.jsp</result>
</action>
```


## Action获取请求参数

### 属性驱动

```java
@Setter
@Getter
public class StudentAction extends ActionSupport {
    private String studentName;
    private String studentSex;

    @Override
    public String execute() throws Exception {
        return "success";
    }
}

```

### 域对象字段驱动方式

```java
@Data
public class User {
	private String username;
	private Stirng password;
}
```

```java
@Getter
@Setter
public class UserLogin extends ActionSupport {
    public User user;
    @Override
    public String execute() throws Exception {
        String username = user.getUsername();
        String password = user.getPassword();
        if (username.equals("admin") && password.equals("123456")) {
            return SUCCESS;
        } else {
            return ERROR;
        }
    }
}
```


## <result>配置的动态结果

### 在UserAction.java继续配置

```java
public String nextOperation;
public String getNextOperation(String nextOperation) {
    return nextOperation;
}

public void setNextOperation(String nextOperation) {
    this.nextOperation = nextOperation;
}

public String login3(){
    if (username.equals("admin") && password.equals("123456")) {
        nextOperation = "admin";
    } else {
        nextOperation = "general";
    }
    return SUCCESS;
}
```

### 编辑struts.xml

```xml
        <action name="login3" class="com.struts.action.UserAction" method="login3">
            <result name="success" type="redirectAction">${nextOperation}</result>
        </action>
        
        <action name="admin" class="com.struts.action.UserAction">
            <result name="success">/admin.jsp</result>
        </action>
        
        <action name="general" class="com.struts.action.UserAction">
            <result name="success">/general.jsp</result>
        </action>
```

