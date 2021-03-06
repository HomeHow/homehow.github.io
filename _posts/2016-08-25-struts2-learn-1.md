---
layout: post
title: struts2学习笔记（一）
subtitle: Struts2-helloworld
date: 2016-08-25T00:00:00.000Z
author: HomeHow
header-img: img/post-bg-js-module.jpg
catalog: true
tags:
  - struts2学习
---
# 1 Struts2简述 #
Struts2 是一个用来开发 MVC 应用程序的框架. 它提供了 Web 应用程序开发过程中的一些常见问题的解决方案:  

* 对来自用户的输入数据进行合法性验证  
* 统一的布局  
* 可扩展性  
* 国际化和本地化  
* 支持 Ajax  
* 表单的重复提交  
* 文件的上传下载  

等等 

 
Struts2 通过拦截器完成了框架的大部分工作。在 Struts2 中插入一个拦截器对象相当简便易行。  

*  Struts2 使用了一个过滤器作为控制器  
*  Struts2 中, HTML 表单将被直接映射到一个 POJO  
*  Struts2 中的验证逻辑编写在 Action 中  
*  Struts2 中任何一个 POJO 都可以是一个 Action 类  
*  Struts2 在页面里使用 OGNL 来显示各种对象模型, 可以不再使用 EL 和 JSTL  

---

# 2 一个简单的Struts2应用 #

## 2.1 需求 ##

![photo1](/img/in-post/struts2-learn-1/1.png)

## 2.2 目录组织 ##

```

├─src
│  │  struts.xml
│  │
│  └─com
│      └─hhl
│          └─struts2
│              └─helloworld
│                      Product.java
│
└─web
    │  index.jsp
    │
    └─WEB-INF
        │  web.xml
        │
        ├─classes
        │  │
        │      ....
        │
        ├─lib
        │      asm-3.3.jar
        │      asm-commons-3.3.jar
        │      asm-tree-3.3.jar
        │      commons-fileupload-1.3.jar
        │      commons-io-2.0.1.jar
        │      commons-lang3-3.1.jar
        │      commons-logging-1.1.3.jar
        │      freemarker-2.3.19.jar
        │      javassist-3.11.0.GA.jar
        │      log4j-1.2.17.jar
        │      ognl-3.0.6.jar
        │      struts2-core-2.3.15.3.jar
        │      xwork-core-2.3.15.3.jar
        │
        └─pages
                details.jsp
                input.jsp
```

### 2.2.1 web.xml文件 ###

主要完成对`StrutsPrepareAndExecuteFilter`的配置（在以前的版本中是对`FilterDispatcher`配置，新版本同样支持用`FilterDispatcher`配置），它的实质是一个过滤器，它负责初始化整个Struts框架并且处理所有的请求。这个过滤器可以包括一些初始化参数，有的参数指定了要加载哪些额外的xml配置文件，还有的会影响struts框架的行为。除了`StrutsPrepareAndExecuteFilter`外，Struts还提供了一个`ActionContexCleanUp`类，它的主要任务是当有其它一些过滤器要访问一个初始化好了的struts框架的时候，负责处理一些特殊的清除任务。
  
如下*web.xml*文件中**5-8**行完成对*filter*的配置。    

**web.xml:**

{% highlight xml linenos %}
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
    <filter>
        <filter-name>struts2</filter-name>
        <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>struts2</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
{% endhighlight %}

### 2.2.2 新建action ###

应用程序可以完成的每一个操作，这里简单的新建一个*action*

**Product.java:**

{% highlight java linenos %}
package com.hhl.struts2.helloworld;

/**
 * Created by HomeHow on 2016/8/30.
 */
public class Product {
    private Integer productId;
    private String productName;
    private String productDesc;

    private double productPrice;

    public Integer getProductId() {
        return productId;
    }

    public void setProductId(Integer productId) {
        this.productId = productId;
    }

    public String getProductName() {
        return productName;
    }

    public void setProductName(String productName) {
        this.productName = productName;
    }

    public String getProductDesc() {
        return productDesc;
    }

    public void setProductDesc(String productDesc) {
        this.productDesc = productDesc;
    }

    public double getProductPrice() {
        return productPrice;
    }

    public void setProductPrice(double productPrice) {
        this.productPrice = productPrice;
    }

    @Override
    public String toString() {
        return "Product [productId=" + productId + ", productName="
                + productName + ", productDesc=" + productDesc
                + ", productPrice=" + productPrice + "]";
    }

    public String save(){
        System.out.println("save: " + this);
        return "details";
    }

    public String test(){
        System.out.println("test");
        return "success";
    }

    public Product() {
        System.out.println("Product's constructor...");
    }
}

{% endhighlight %}


### 2.2.3 配置action与struts.xml文件 ###

框架的核心配置文件就是这个默认的*struts.xml*文件，在这个默认的配置文件里面我们可以根据需要再包括其它一些配置文件。在通常的应用开发中，我们可能想为每个不同的模块单独配置一个*struts.xml*文件，这样也利于管理和维护。这也是我们要配置的主要文件  

**struts.xml:**
{% highlight xml linenos %}
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE struts PUBLIC
        "-//Apache Software Foundation//DTD Struts Configuration 2.0//EN"
        "http://struts.apache.org/dtds/struts-2.0.dtd">

<struts>
    <package name="helloWorld" extends="struts-default">
        <action name="product-input">
            <result>/WEB-INF/pages/input.jsp</result>
        </action>
        <action name="product-save" class="com.hhl.struts2.helloworld.Product" method="save">
            <result name="details">/WEB-INF/pages/details.jsp</result>
        </action>
    </package>
</struts>
{% endhighlight %}

### package节点 ###

*package*节点是整个配置的核心部分。每个*package*，从语义上讲，其实代表了每一个独立的模块。在这个模块中，你可以定义隶属于这个模块的行为方式，而与其他的模块没有关系。所以，每个package都有独立的*interceptor*、*result-type*和*action*的定义，绝大多数的*Runtime*配置定义都是通过*package*节点实现的。接下来我们就来详细讨论一下package中的属性和子节点。  
  
1. **name**  
name属性为每个package设置一个唯一的标识，这个标识在所有的*package*定义中不能重复。  
2. **abstract**  
标识这个package的定义是一个抽象定义，也就是允许他仅包含声明式的定义，而不需要在*package*定义中包含*action*的定义。  
3. **extends**  
通过使用*extends*，你可以指定本*package*继承另外一个*package*的所有的配置。当某个*package*继承了另外一个*package*的所有配置，那么你就无需对父*package*中已经声明过的配置定义做再次的定义。  
同时，如果重复定义父*package*中已声明过的配置定义，那么这些重复定义声明将覆盖父*package*中的相关定义。  
4. **namespace**  
*namespace*的作用：对于*action*配置进行逻辑划分。
如果我们不为*package*节点指定*namespace*，*Struts2*默认使用一个空字符串作为默认的*namespace*。当然，也可以使用"/"等字符串来表示*namespace*。  
定义*package*命名空间 该命名空间影响到url的地址，例如此命名空间为/test那么访问是的地址为：
``
http://localhost:8080/struts2/test/XX.action
``  
*Struts2*在根据URL进行寻址的时候，使用以下的步骤：  
1) 根据URL进行*Namespace*和*ActionName*的计算  
2) 根据计算的得到的*Namespace*和*Action*
  

### 2.2.4 其他文件 ###

**index.jsp:**

{% highlight html linenos %}
<%--
  Created by IntelliJ IDEA.
  User: HomeHow
  Date: 2016/8/30
  Time: 15:48
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Insert title here</title>
  </head>
  <body>
    <a href="product-input.action">Product Input</a>

    <br><br>

    <a href="test.action">Test</a>
  </body>
</html>
{% endhighlight %}

**input.jsp:**

{% highlight html linenos %}
<%--
  Created by IntelliJ IDEA.
  User: HomeHow
  Date: 2016/8/30
  Time: 16:06
  To change this template use File | Settings | File Templates.
--%>
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Title</title>
</head>
<body>
<form action="product-save.action" method="post">
    ProductName:<input type="text" name="productName"/>
    <br><br>

    ProductDesc:<input type="text" name="productDesc"/>
    <br><br>

    ProductPrice:<input type="text" name="productPrice"/>
    <br><br>

    <input type="submit" value="Submit"/>
    <br><br>
</form>
</body>
</html>
{% endhighlight %}


**detail.jsp:**

{% highlight html linenos %}
<%--
  Created by IntelliJ IDEA.
  User: HomeHow
  Date: 2016/8/30
  Time: 16:06
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
    <head>
        <title>Title</title>
    </head>
    <body>
        ProductId:${productId}
        <br><br>

        ProductName:${productName}
        <br><br>

        ProductDesc:${productDesc}
        <br><br>

        ProductPice:${productPrice}
        <br><br>
    </body>
</html>
{% endhighlight %}
