## 一、项目简介

本项目为仿QQ实时聊天通讯WEB应用。

技术：Spring Boot+WebSocket+Thymeleaf

使用说明笔记（支持全部图片显示，项目介绍更加全面）路径：https://flowus.cn/a5f1d273-e13a-45b8-a416-7bfe22ec1788

## 二、功能模块

本项目主要包含登录注册模块和聊天室两大板块。由于本项目主要是对WebSocket进行聊天室开发进行验证，所以只是对登录注册进行了简单的设计，主体为聊天室功能开发。

- 首先简单介绍一下thymeleaf引擎的作用：

  服务端templates文件夹中存放大量事先编写好的html模板，注意是模板不是html页面，它里面含有html本身不含有的一些语法来渲染data数据，生成前端需要的html页面。

  Thymeleaf模板引擎的机制就是：controller接口将浏览器端需要的html模板和携带的数据交给模板引擎进行渲染，生成完整的html页面，然后将该页面响应给浏览器端。

### (一)登录注册功能

![登录验证机制.drawio.png](https://flowus.cn/preview/77c9d681-20ac-4c62-9e87-e381e9a5d594)

登录功能设计，存在两种情景会进入登录页面：

a.使用链接访问登录接口，登录接口将登录模板简单渲染，生成登录html页面，直接返回给浏览器进行渲染；

b.验证登录时，录账号密码输入错误，将登录页面返回给浏览器，并提示密码错误。

显然登录页面在渲染时需要对第二种情况进行响应。

综上所述，我们的登录验证controller接口在加载登录模板给浏览器前需要传一个数据（verification）给模板，告知模板是以何种方式被请求的，如果是验证失败则模板引擎渲染响应效果。

我们看登录模板（不是页面）有以下代码：

```JavaScript
<script th:inline="javascript">
    var verification = [[${verification}]]
    //这是一段js代码，直接执行layui.use中函数
    layui.use('layer', function(){
        var layer = layui.layer;//layui组件库中弹出框提示
        //js获取跳转该页面时转发的数据
        if (verification === "false") {//登录失败时跳转到login页面
            layer.msg('账号或密码不正确');//类似于alert()
        }
    });
</script>
```

页面传给浏览器之后，执行JavaScript语句，判断何种场景得到的登录页面，如果时验证失败，verification为false，则弹出msg信息告知账号密码不正确。如果是其它情景，则无该响应。

### (二)聊天室功能

聊天室功能主要是借助WebSocket来实现消息的实时转发。而消息的离线转发则借助数据库的读取。

#### 1.实时通讯

其大致工作结构如图：

![websocket转发机制.drawio.png](https://flowus.cn/preview/ee2ee608-850d-48b7-9b5b-1e97f7501ef3)

上图中我们看到webSocketServer主要做消息转发，接收到前端send()的信息之后按照用户池中的session向指定用户转发实时消息。而用户借助onMessage()进行消息的接收。

在聊天室中，前端首先使用new WebSocket("websocketserver路径")来实现与后台websocketserver的链接。这个链接是双向通道的：当链接建立时返回websocket变量，该变量提供onOpen()\onMessage()\send()三种主要方式；而后台server也提供了onOpen()\OnMesssage()等监听方法。前端借助该变量主要借助send()发送消息给server，由其处理之后转发给特定的用户，实现消息的实时转发。

#### 2.离线通讯

关于离线通讯主要借助的数据库。由于我们聊天室需要支持历史记录的加载，因此无论websocketserver转发消息是否成功(目标用户是否在线)，我们都需要对每条消息进行记录。因此我们选择在websocketserver onMessage()方法中添加指定的方法，将信息插入数据库之后，再进行消息的实时转发。

## 三、项目演示

![登录页面](https://flowus.cn/preview/d9bec45e-3a2e-4e0e-a0ce-07ddd4c3c3ae)
登录页面

![主体页面](https://flowus.cn/preview/c56ea678-5742-415e-865b-3efd0fedc67d)
主体页面

![消息传输](https://flowus.cn/preview/b93be5c7-2381-4b34-b164-0957832876e1)
消息传输

![图片实时发送](https://flowus.cn/preview/2965748e-d7a5-4fdf-ac46-1db930e40388)
图片实时发送

## 四、项目的安装与配置

![image.png](https://flowus.cn/preview/5d49bb17-5f72-441a-8436-5b824c573353)

这里我们看到application.properties指定基本配置在dev开发环境中（具体可以搜索springboot多环境配置）

![image.png](https://flowus.cn/preview/a9b3d8d5-b95e-4333-aa1f-95483e6b08ef)

服务器端口设定为9999

数据库采用的是mysql5.7，如果是mysql8也支持。数据库名称为`keshe`，其sql文件保存在demo/src/main/resource/sql/keshe.sql中。

## 五、不足与改进

该项目目前仅支持消息的实时离线转发但不支持图片的离线发送。图片的信息存储不仅仅涉及数据库的更改（历史信息数据表还要进行重新的设计），还需要涉及文件上传和静态资源路径设置。该项目主要是对websocket实时消息转发进行验证性实现，因此对于离线消息的存储不做过多的实现。项目在后续的涉及中会预留图片发送功能、表情功能模块以便其它人对项目后续的扩展应用。



