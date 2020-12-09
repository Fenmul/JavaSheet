### Tomcat 
#### 启动存在的问题
1. 黑窗口一闪而过：没有正确配置 JAVA_HOME 的文件目录，原因是批处理文件中依赖于 JAVA_HOME 路径的配置
2. 参考日志文件中的报错信息

`netstat -ano` 找到端口号对应的 pid 然后杀死对应的线程

修改配置文件 server.xml 修改端口号

**设置默认端口号是 80**，80 是 HTTP 协议的默认端口号，在访问的时候可以不用输入端口号，比如一般网站的端口号都是 80 所以不需要输入。

#### tomcat 部署
1. war 包部署，放到 webapp 目录下会自动解压
2. server.xml 文件中在 host 内配置 `<Context docBase="D:\hello" path="\hh"/>` 项目的路径在 docBase 下，虚拟路径为 hh。
3. 在 conf\Catalina\localhost 目录下创建任意名称的 xml 文件，在文件中编写 `<Context docBase="D:\hello" />`，虚拟路径就是文件名，并且是热部署方式 

推荐是使用 方法 3 进行部（了解即可）

#### Tomcat 在 IDEA中的一些使用说明
Tomcat 中真正访问的是 工作空间中 web  对应的文件，会被同步到 Tomcat 中目录下，IDEA 就是利用的第三种方式实现配置的
`C:\Users\foxyuan\.IntelliJIdea2018.3\system\tomcat\Tomcat_8_5_43_corejava\conf\Catalina\localhost`
`<Context path="/servlet" docBase="D:\Idea\corejava\classes\artifacts\web_war_exploded" />`

启动完成自动打开浏览器，After Launcher 勾选即可

web-inf 中的资源服务无法被直接访问

### Servlet 
Server Applet：运行在服务器的小程序，本质上就是一个接口，可以被 tomcat 识别

实现 Servlet 接口就可以被 tomcat 识别

路径的配置在 tomcat 中的 deployment 中配置虚拟的路径
```java

package com.web.server;

import javax.servlet.*;
import java.io.IOException;

/**
 * 1. 实现 Servlet 接口
 * 2. web.xml 配置
 * 3.
 */
public class ServletDemo implements Servlet {
    /**
     * 初始化方法，只会被执行一次（用于加载资源）
     * 可以 load-on-startup 配置是否在服务器启动时加载创建
     * 使用场景：servlet 相互依赖时，控制需要先加载的 servlet
     * Servlet <strong>内存中只存在一个对象</strong>（单例）
     * 存在问题：并发访问时存在线程安全问题，解决方案是不要定义成员变量，采用局部变量解决问题
     * @param servletConfig
     * @throws ServletException
     */
    @Override
    public void init(ServletConfig servletConfig) throws ServletException {
        System.out.println("init");
    }

    @Override
    public ServletConfig getServletConfig() {
        return null;
    }

    // 提供服务的方法，多次执行
    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        System.out.println("Hello Servlet");
    }

    @Override
    public String getServletInfo() {
        return null;
    }

    /**
     * 对象被销毁时调用
     * Servlet 被销毁之前执行，用于释放资源
     */
    @Override
    public void destroy() {
        System.out.println("destory...");
    }
}

```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <!-- 配置了 Servlet 的映射地址和具体的实现类 -->
    <servlet>
        <servlet-name>hello</servlet-name>
        <servlet-class>com.web.server.ServletDemo</servlet-class>
        <!--
        配置值大于等于 0：服务器启动时创建，否则就是调用时被创建
        <load-on-startup></load-on-startup>
        -->
    </servlet>

    <servlet-mapping>
        <servlet-name>hello</servlet-name>
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>

</web-app>
```
Servlet 3.0 以后可以使用注解进行配置

`@WebServlet()` 注解中可以定义多个 url 地址 `/*` 可以任意匹配

#### GenericServlet
因为 Servlet 中每次执行的方法主要是 Service 但是实现 Servlet 接口还需要实现其他方法，所以可以通过继承 GenericServlet 抽象类

对于 init、 destory 方法都做了空处理，使用时重写即可

#### HttpServlet （常用）
之所以需要在 GenericServlet 再次继承一次是由于需要判断**请求方式**

HttpServlet 封装了 HTTP ，就是对于 GET 和 POST 请求做了处理

我们可以直接重写 doGet 和 doPost 方法实现（就是 Service 方法）
```java
protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String method = req.getMethod();
        long lastModified;
        if (method.equals("GET")) {
            lastModified = this.getLastModified(req);
            if (lastModified == -1L) {
                this.doGet(req, resp);
            } else {
                long ifModifiedSince;
                try {
                    ifModifiedSince = req.getDateHeader("If-Modified-Since");
                } catch (IllegalArgumentException var9) {
                    ifModifiedSince = -1L;
                }

                if (ifModifiedSince < lastModified / 1000L * 1000L) {
                    this.maybeSetLastModified(resp, lastModified);
                    this.doGet(req, resp);
                } else {
                    resp.setStatus(304);
                }
            }
        } else if (method.equals("HEAD")) {
            lastModified = this.getLastModified(req);
            this.maybeSetLastModified(resp, lastModified);
            this.doHead(req, resp);
        } else if (method.equals("POST")) {
            this.doPost(req, resp);
        } else if (method.equals("PUT")) {
            this.doPut(req, resp);
        } else if (method.equals("DELETE")) {
            this.doDelete(req, resp);
        } else if (method.equals("OPTIONS")) {
            this.doOptions(req, resp);
        } else if (method.equals("TRACE")) {
            this.doTrace(req, resp);
        } else {
            String errMsg = lStrings.getString("http.method_not_implemented");
            Object[] errArgs = new Object[]{method};
            errMsg = MessageFormat.format(errMsg, errArgs);
            resp.sendError(501, errMsg);
        }

    }
```

### HTTP 超文本传输协议
定义了客户端和服务端通信发送数据的格式
- TCP/IP 协议（安全的需要三次握手）
- 响应请求模型，一次请求对应一次响应
- 默认端口是：80
- 无状态：每次请求相互独立

HTTP 1.0 版本建立新连接，而 1.1 复用连接

#### Request 请求消息
1. 请求行
```text
请求方式 请求 URL 请求协议/版本  
GET     /hello     TCP/1.1  
```
* 请求方式
    * HTTP 有 7 中请求方式
    * GET 请求
        1. 参数在请求行中（请求 URL 中）
        2. 存在长度限制
        3. 不安全
    * POST 请求
        1. 参数在请求体中（GET 请求并没有请求体）
        2. 不存在大小限制
        3. 相对安全
   
2. 请求头
`请求头名称：请求头值`
常见的请求头：
    * User-Agent：浏览器版本信息用于解决兼容问题
    * Referer：告诉服务器请求来源（曾用于解决盗链问题，还可以用于广告统计）
         

3. 请求空行
4. 请求体（正文）

```text
GET /servlet/hello HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.142 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8

```
#### Request
1. tomcat 会根据请求 URL 中的资源路径创建对应的 Servlet 对象（这里的 Servlet 对象）
2. **tomcat 会创建 request 和 response 对象**，request 对象中封装了请求消息数据。
3. tomcat 将 request 和 response 对象传递到 service 方法

```text
ServletRequest （接口）
| 继承
HttpRequest （接口）
| 继承 
RequestFacade （Tomcat 中的实现类） 

```
`GET /servlet/hello?name=zhangsan HTTP/1.1`
* 获取请求行：
    * **获取虚拟目录** getContextPath() /servlet
    * 获取 Servlet 路径 getServletPath() /hello
    * 获取 GET 参数 getQueryString name=zhangsan
    * **获取 URI** getRquestURI()  /servlet/hello
    * 获取 URL getRequstURL()  http://localhost/servlet/hello
    * 获取 **客户机的 ip** getRemoteAddr() 
    
URL: 统一资源定位符（http://localhost/servlet/hello）

URI: 统一资源标识符（/servlet/hello）

URI 代表的范围更大一点，URI 可以包括 URL 

* 中文乱码问题
    * GET 在 tomcat 8 以后中文乱码问题被解决了
    * POST 请求仍然存在 `req.setCharacterEncoding("UTF-8")`
    * 由于在获取请求体中参数时，是通过流来处理的所以可以通过上述语句修改编码格式
    
#### 转发：
 `req.getRequestDispatcher("/hello").forward(req, resp);`
1. 地址栏不变
2. 一次请求
3. 不可以跳转到服务器外的地址

**Request 域**：一次请求的范围，多个资源共享数据
 
 
 
