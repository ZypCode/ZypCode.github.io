---
title: 《白帽子讲Web安全》随笔
date: 2019-02-26 15:09:34
categories:
- 随笔
tags:
- Web安全
---
目前在阅读《白帽子讲Web安全》，在此做一些随笔记录。
## 第1章 我的安全世界观
- 在早期互联网中，黑客们的主要攻击目标是网络、操作系统以及软件等领域。而运营商、防火墙对于网络的封锁，使得暴露在互联网上的非Web服务越来越少，且Web技术的逐渐成熟，使得其成为了黑客攻击的主要目标。
- Web攻击技术的发展：
	+ Web1.0时代，主要关注的是服务器端动态脚本的安全问题。
		* SQL注入的出现是Web安全史上的一个里程碑。
		* XSS(跨站脚本攻击)的出现是另一个里程碑。
	+ Web2.0的兴起，XSS，CSRF等攻击变得更强大，Web攻击的思路也由服务器端转向了客户端。
- 安全问题中，极端条件意味着小概率及高成本，因此会根据成本设计安全方案，并将一些可能性较大的条件作为决策的主要依据。
- 安全问题的组成属性总结为三要素，简称CIA：
	+ 机密性(Confidentiality)：要求保护数据内容不能泄露，加密为常用手段。
	+ 完整性(Integrity)：要求保护数据内容是完整、没有被篡改的，数字签名为常用手段。
	+ 可用性(Availability)：要求保护资源是"随需而得"。
		* 如停车场100个车位，被坏人用100个大石头把车位都占用了，停车场无法继续提供正常服务。
		* 安全领域这种攻击叫拒绝服务攻击(DoS)，该攻击破坏的就是安全的可用性。
- 安全评估过程可以分为4个阶段：
	+ 资产等级划分：帮助明确目标是什么，要保护什么。
	+ 威胁分析：可能造成危险的来源称为威胁，威胁建模的一个方法是微软提出的STRIDE模型
		* 伪装、篡改、抵赖、信息泄露、拒绝服务、提升权限
	+ 风险分析：可能会出现的损失称为风险，衡量风险的一个模型也是微软提出的DREAD模型(书中Page14)
	+ 确认解决方案
		* 优秀的方案应该能够有效解决问题、用户体验好、高性能、低耦合、易于扩展与升级。
- 纵深防御原则，包含2层含义：
	+ 首先，要在各个不同层面、不同方面实施安全方案，避免出现疏漏，不同安全方案之间需要相互配合，构成一个整体。
	+ 其次，要在正确的地方做正确的事情，即在解决根本问题的地方实施针对性的安全方案。

## 第2章 浏览器安全

- 同源策略(Same Origin Policy)是浏览器最核心也最基本的安全功能，它限制了来自不同源的"document"或脚本，对当前"document"读取或设置某些属性。
	+ 受到同源策略约束就不能跨域访问资源，但随着业务发展，跨域请求的需求愈发强烈。

- 挂马：在网页中插入一段恶意代码，利用浏览器漏洞执行任意代码的攻击方式被称为挂马。
- 沙箱SandBox：可以让不受信任的网页代码、JS代码运行在一个受到限制的环境中，从而保护本地桌面系统的安全。
- 恶意网站拦截：目前主要通过推送恶意网址黑名单的方式，另外主流浏览器也开始支持EV SSL证书来增强对安全网站的识别。
	+ PhishTank是一个免费提供恶意网站黑名单的组织，更新频繁。
	+ 谷歌也公开了内部使用的SafeBrowsing API。

## 第3章 跨站脚本攻击(XSS)

- XSS攻击：通常指黑客通过"HTML注入"篡改了网页，插入了恶意脚本（这些脚本被称为XSS Payload），从而在用户浏览网页时控制用户浏览器的一种攻击。
- XSS根据效果不同主要分为以下几类：
  - 反射型XSS（非持久型XSS）
    - 只是简单地把用户输入的数据"反射"给浏览器，需要用户点击链接才能攻击成功。
  - 存储型XSS（持久型XSS）
    - 会把用户输入的数据"存储"在服务器端，具有很强稳定性。
  - DOM Based XSS
    - 从效果上看也是反射型XSS，出于历史原因单独分类。
    - 通过修改页面的DOM节点形成的XSS。
- 一个最常见的XSS Payload就是通过读取浏览器的Cookie对象，从而发起Cookie劫持攻击。
- 具体的一些XSS攻击及相关的代码可以在书中3.2.2节查阅。
- 一些XSS攻击平台：
  - Attack API
  - BeEF
  - XSS-Proxy
- XSS Worm
  - 一般来说，用户之间发生交互行为的页面，如果存在存储型XSS，则比较容易发起XSS Worm攻击。
- 想写好XSS Payload，需要有很好的JS功底，调试JS就成了必不可少的技能。介绍几款常用的JS调试工具：
  - Firebug
  - IE 8 Developer Tools
  - Fiddler
  - HttpWatch
- 实际环境中，XSS的利用技巧比较复杂，下面是一些常见的XSS攻击技巧，也是网站在设计安全方案时需要注意的地方：
  - 利用字符编码
  - 绕过长度限制（某些环境下还可以利用注释符绕过长度限制）
  - 使用<base>标签
    - 该标签作用是定义页面上的所有使用"相对路径"标签的hosting地址
  - window.name的妙用
  - 还有一些曾经被认为无法利用的XSS漏洞，都被人找到了利用方法（书中3.2.7节）：Apache Expect Header XSS、Anehta的回旋镖、
- 除了基于HTML的XSS攻击，在Flash中同样也可能造成XSS攻击，通常是在Flash中嵌入ActionScript。
  - 限制Flash动态脚本的最重要的参数是"allowScriptAccess"，其定义了Flash能否与HTML页面进行通信。
- 在Web前端开发中，一些JS开发框架深受开发者欢迎，但一些JS框架也曾暴露过一些XSS漏洞：
  - Dojo
  - YUI
  - jQuery
- XSS的防御

  现在的浏览器都会内置对抗XSS的措施，但网站也应该寻找优秀的解决方案。

  - HttpOnly：由微软提出，禁止页面的JS访问带有HttpOnly属性的Cookie。主要解决的是XSS后的Cookie劫持。

  - 输入检查。

  - 输出检查：在变量输出到HTML页面时，可以使用编码或转义的方式防御XSS攻击。但需要分清楚输出变量的语境，根据情况区分对待：（书中3.3.4节）

    - 在HTML标签中输出
    - 在HTML属性中输出
    - 在"\<script\>"标签中输出
    - 在事件中输出
    - 在CSS中输出
    - 在地址中输出

  - 处理富文本：允许用户提交的自定义HTML代码。

    从输入检查的思路走，因为富文本数据是完整的HTML代码，可以特殊处理。目前有一些比较成熟的开源项目实现了对富文本的XSS检查，如Anti-Samy。

  - DOM Based XSS防御

    DBX是从JS中输出数据到HTML页面中，与前面针对的从服务器应用直接输出到HTML页面不同。

    而从JS输出到HTML页面也相当于一次XSS输出的过程，需要分语境进行编码。

    JS输出到HTML的方式见（书Page106）

## 第4章 跨站点请求伪造(CSRF)

+ CSRF，Cross Site Request Forgery，伪造对其他站点操作的请求。
+ 浏览器所持有的Cookie分2种：
  + Third-party Cookie，本地Cookie。服务器在Set-Cookie时指定了Expire时间，只有到时间后Cookie才会失效，保存在本地。
  + Session Cookie，临时Cookie。没有指定Expire时间，浏览器关闭后失效，保存在浏览器进程的内存空间。
+ 一些浏览器会默认禁止浏览器在img、iframe、script、link标签中发送第三方Cookie
  + 默认拦截：IE 6、7、8、Safari
  + 默认不拦截：Firefox 2、3、Opera、Chrome、Android
+ 拦截第三方Cookie某种程度上降低了CSRF攻击的威力，但P3P的介入使得情况变得复杂。在网站的业务中，P3P头主要用于类似广告等需要跨域访问的页面。




## 未完待续……