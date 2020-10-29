# 使用Thymeleaf fragment组件重用页面布局

通常网站页面之间都会共享相同的布局组件，如header、footer、导航栏、菜单栏等等诸如此类，为了使这些组件可以被相同或不同的页面布局所使用，Thymeleaf模板引擎引入了fragment组件。


## 引入依赖
>在`pom.xml`中添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

## 定义/引用fragment

使用`th:fragment`属性定义一个Thymeleaf fragment组件，通常将所有的组件都放在同一个文件夹(比如命名为fragments)下。

MyTicket中的header组件定义如下(`layout.html`)：

```html
<!-- ##### Header区域 ##### -->
<header class="header-area" th:fragment="site-header">
    <!-- 导航栏区 -->
    <div class="musica-main-menu">
        <div class="classy-nav-container breakpoint-off">
            <div class="container-fluid">
                <!-- 导航栏 -->
                <nav class="classy-navbar justify-content-between" id="musicaNav">

                    <!-- 导航栏logo -->
                    <a href="/" th:href="@{/}" class="nav-brand"><img src="img/core-img/logo.png" alt=""></a>

                    <!-- 导航栏折叠(移动端) -->
                    <div class="classy-navbar-toggler">
                        <span class="navbarToggler"><span></span><span></span><span></span></span>
                    </div>

                    <!-- 菜单栏 -->
                    <div class="classy-menu">

                        <!-- 关闭 -->
                        <div class="classycloseIcon">
                            <div class="cross-wrap"><span class="top"></span><span class="bottom"></span></div>
                        </div>

                        <!-- 导航栏开始 -->
                        <div class="classynav">
                            <ul>
                                <li><a href="/" th:href="@{/}">首页</a></li>
                                ...
                            </ul>
                        </div>
                        <!-- 导航栏结束 -->
                    </div>
                </nav>
            </div>
        </div>
    </div>
</header>
<!-- ##### Header 区域 ##### -->
```

我们定义了一个`site-header`组件，在`homepage.html`中，使用`th:insert`或`th:replace`属性来重用它：

```html
<!-- ##### 导航栏 ##### -->
<header th:replace="layout :: site-header"></header>
<!-- ##### 导航栏 ##### -->
```

当然，也可以在一个html文档中定义多个fragment组件，如`layout.html`也定义了footer：

```html
<!-- ##### Footer 区域 ##### -->
<footer class="footer-area section-padding-100-0" th:fragment="site-footer">
    <div class="container-fluid">
        <div class="row">
            <!-- Footer 组件 -->
            ...
        </div>
    </div>
</footer>
<!-- ##### Footer 区域 ##### -->
```

## 使用DOM选择器加载组件

在Thymeleaf中，除了显式使用`th:fragment`属性定义fragment组件，还可以用DOM选择器(如class名，元素ID，标签名等)来定义并加载组件。如：

```html
<div th:insert="layout :: div.title"></div>
```

使用到`<div>`标签下的`.title`CSS class来作为组件。

## 参数化组件

使用`th:fragment`定义的Thymeleaf fragment组件可以指定参数，就像方法调用一样，比如MyTicket的分页组件(`fragments.html`)：

```html
<body>
    <div th:fragment="pagination(pager,URLparameter)">
        <!--分页-->
        <nav aria-label="Home and navigation pagination" th:if="${pager.getTotalPages() gt 0}">
            <ul class="pagination justify-content-center font-weight-bold" th:with="baseUrl=${URLParameter}">
            ...
            </ul>
        </nav>
    </div>
</body>
```

在使用过程中，将形参替换为实参，比如`homepage.html`：

```html
<!-- 内容详情 -->
<div  class="upcoming-shows-content">
    <!-- 分列显示 -->
    <div th:replace="fragments :: single-event-html"></div>
    <br>
    <br>
    <!-- 分页显示 -->
    <div class="musica-pagination-area">
        <div th:replace="fragments :: pagination(pager=${pager}, URLparameter='/')"></div>
    </div>
</div>
```

参数化组件扩大了我们代码重用的范围。

## fragment表达式

Thymeleaf fragment支持条件表达式来动态加载不同的组件，格式为`templatename :: selector`，例如，我们想根据用户是否为admin加载不同的footer：

```html
<div th:replace="layout :: ${user.admin} ? 'footer-admin' : 'footer'"></div>
```

对于在不同文档中定义的组件，可以用fragment表达式语法来写：

```html
<div th:replace="${user.admin} ? ~{layout :: footer} : 
~{fragments :: footer}"></div>
```

## 创建Layout

使用fragment表达式还可以创建基础layout模板，比如`myLayout.html`：

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org" th:fragment="layout(title, content)">
<head>
    <meta charset="UTF-8">
    <title th:replace="${title}">自定义主题</title>
</head>
<body>

    <h1>发现 | FIND</h1>

    <section th:replace="${content}">
        <p>自定义段落内容</p>
    </section>

    <footer>
        <p>&copy; 2020 MyTicket</p>
    </footer>

</body>
</html>
```

如上定义了`layout`组件，包含参数`title`和`content`，我们可以通过使用fragment表达式替换这些参数所在标签。

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org"
      th:replace="myLayout :: layout(~{::title}, ~{::section})">
<head>
    <title>欢迎来到 MyTicket</title>
</head>
<body>

    <section>
        <p>发现附近的活动.</p>
        <a th:href="@{/search}">搜索</a>
    </section>

</body>
</html>
```
如上，`<html>`标签会被layout组件代替，title参数会被`<title>`标签代替，content参数会被`<section>`标签代替，输出最终的HTML文档。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>欢迎来到 MyTicket</title>
</head>
<body>

    <h1>发现 | FIND</h1>

    <section>
        <p>发现附近的活动.</p>
        <a href="/search">搜索</a>
    </section>

    <footer>
        <p>&copy; 2020 MyTicket</p>
    </footer>

</body>
</html>
```