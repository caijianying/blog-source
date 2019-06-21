---
title: Thymeleaf布局
date: 2018-07-22 18:52:07
tags: 开发笔记
categories:
         - thymeleaf
         - java
         - 模板
---
# Thymeleaf布局

## 前言
在web 开发过程中，经常会有一些公共页面，例如head,foot,menu等，利用layout可以极大的提高开发效率。通过使用Thymeleaf Layout Dialect,可以便捷使用布局，减少代码的重复。
## 正文

### 1.配置

在pom 中增加依赖：
```
<dependency>
	<groupId>nz.net.ultraq.thymeleaf</groupId>
	<artifactId>thymeleaf-layout-dialect</artifactId>
</dependency>
```

Spring Boot 2 java 初始化配置：
```
@Bean
public LayoutDialect layoutDialect() {
    return new LayoutDialect();
}
```
以上配置会使layout 命名空间可以引入五种属性：decorate, title-pattern, insert, replace, fragment

Thymeleaf Layout Dialect 默认可以合并layout与content中head信息，默认在layout 标签之后追加content中的标签。

### 2.布局
Layout.html
```
<!DOCTYPE html>
<html xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout">
<head>
  <title>Layout page</title>
  <script src="common-script.js"></script>
</head>
<body>
  <header>
    <h1>My website</h1>
  </header>
  <section layout:fragment="content">
    <p>Page content goes here</p>
  </section>
  <footer>
    <p>My footer</p>
    <p layout:fragment="custom-footer">Custom footer here</p>
  </footer>  
</body>
</html>
```
Content1.html

```
<!DOCTYPE html>
<html xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
  layout:decorate="~{Layout}">
<head>
  <title>Content page 1</title>
  <script src="content-script.js"></script>
</head>
<body>
  <section layout:fragment="content">
    <p>This is a paragraph from content page 1</p>
  </section>
  <footer>
    <p layout:fragment="custom-footer">This is some footer content from content page 1</p>
  </footer>
</body>
</html>
```
生成结果如下：
```
<!DOCTYPE html>
<html>
<head>
  <title>Content page 1</title>
  <script src="common-script.js"></script>
  <script src="content-script.js"></script>
</head>
<body>
  <header>
    <h1>My website</h1>
  </header>
  <section>
    <p>This is a paragraph from content page 1</p>
  </section>
  <footer>
    <p>My footer</p>
    <p>This is some footer content from content page 1</p>
  </footer>  
</body>
</html>
```
### 3.布局传递数据
Child/content template:
```
<html layout:decorate="your-layout.html" th:with="greeting='Hello!'">
```
Parent/layout template:
```
<html>
  ...
  <p th:text="${greeting}"></p> <!-- You'll end up with "Hello!" in here -->
```
### 4.配置title
Layout.html
```
<!DOCTYPE html>
<html xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout">
<head>
  <title layout:title-pattern="$LAYOUT_TITLE - $CONTENT_TITLE">My website</title>
</head>
...
</html>
```

Content.html

```
<!DOCTYPE html>
<html xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
  layout:decorator="Layout">
<head>
  <title>My blog</title>
</head>
...
</html>
```

生成结果如下：

```
<!DOCTYPE html>
<html>
<head>
  <title>My website - My blog</title>
</head>
...
</html>
```

## 参考
1. [Thymeleaf Layout Dialect](https://ultraq.github.io/thymeleaf-layout-dialect/)