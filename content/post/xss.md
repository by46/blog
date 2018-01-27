---
title: "Xss"
date: 2018-01-26T09:00:53+08:00
draft: true
---

# DOM XSS

## 什么是DOM XSS
为了理解DOM XSS， 我们首先需要描述一下DOM，并且为什么它与DOM XSS有关。

The Document Object Model是表示和操作HTML中对象的规则。从浏览器的角度来看，基本上， 每个HTML文档都有
与之相关的DOM， 该DOM由表现文档属性的对象组成。执行客户端脚本时， 浏览器向脚本代码提供与之相关的
HTML文档DOM， 因此客户端脚本可以访问文档的大量属性和属性值，然后由浏览器修改文档的表现。

DOM XSS是一种跨站脚本攻击， 在HTML文档中， 不正确的操作DOM，就极易产生DOM XSS。DOM有几个属性特别容易
被攻击者利用制造XSS攻击， `document.url`, `document.location` 和 `document.referrer`属性。

## DOM XSS攻击典型例子
让我们来看一个简单的示例， 该示例提供用于自定义内容的功能， 我们会从编码后的url中获取用户名， 并显示在页面上，

`http://www.example.com/userdashboard.html`资源对应的HTML代码如下：

```HTML
<html>
<head>
<title>Custom Dashboard </title>
...
</head>
Main Dashboard for
<script>
	var pos=document.URL.indexOf("context=")+8;
	document.write(document.URL.substring(pos,document.URL.length));
</script>
...
</html>
```

在浏览器中访问`http://www.example.com/userdashboard.html?context=Mary`资源， 会在页面上显示`Mary`。

但是恶意脚本会被嵌入到url中， 例如下面:

```bash
http://www.example.com/userdashboard.html?context=<script>XSSFunction(somevariable)</script>
http://www.example.com/userdashboard.html#context=<script>XSSFunction(somevariable)</script>
```

此外， 被攻击者的浏览器收到上面的URL， 会向`http://www.example.com`请求静态资源，并且得到上面的HTML代码，
并且，浏览器开始构建页面的DOM， 从`document.url`中获取恶意代码，并通过`document.write`写入到页面， 最终
结果看上就像下面这样：

```HTML
Main Dashboard for <script>XSSFunction(somevariable)</script>

```

最后，浏览器就开始执行这些恶意代码， DOM XSS 攻击被触发，在现实中， 攻击者会通过URL编码来隐藏这些内容，
使它不容易被察觉到，

有些浏览器会自动编码URL中的`<` 和 `>`字符， 在一定程度上能防范DOM XSS攻击。但是，另外某些DOM XSS场景
不需要这些特殊字符，所以浏览器不能防范全部攻击。

## DOM XSS 的不同
通过上面的例子，我们可以得出以下结论：

1. HTML页面是静态的， 不想其他类型XSS攻击， DOM XSS没有任何恶意代码被嵌入页面。
2. 如果在URL中使用`#`资源， 那么这些恶意代码永远不会到达服务端， 那么服务器就更难发现这些问题， 及时
增加服务器攻击防范能力也是枉然。

## DOM XSS防御

DOM XSS攻击很难被服务端和防火墙探测到， 因为恶意代码不会被发送到服务端， 服务端的一切防范措施都失效了。
DOM XSS攻击主要发生在客户端，所以需要加强客服端代码的优化。

### 主要防御措施
1. 避免使用客户端数据执行敏感操作， 例如`rewriting` `redirction`
2. 通过嗅探工具清理容易引起DOM XSS的操作， 例如URL，Location and referrer，特别是操作DOM。
3. 使用入侵预防系统能够检查入站URL参数和防止不适当的页面。