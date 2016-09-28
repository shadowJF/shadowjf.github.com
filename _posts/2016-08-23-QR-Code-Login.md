---
layout: post_layout
title: 二维码登录技术研究
time: 2016年08月25日 星期四
location: 北京
pulished: true
tag: tech
excerpt_separator: "----"
---

好久没更博了，最近正好在研究二维码登录技术，借此记录下最近的研究结果

我将从以下三个方面进行介绍：

- 微信、钉钉二维码扫描登录
- 自实现二维码扫描登录
- 完整的二维码扫描登录方案

----



### **一. 微信、钉钉二维码扫描登录** ###

　　微信、钉钉二维码扫描登录都是借助其开放平台来支持第三方网站通过扫描它们提供的二维码来获取用户相应的微信、钉钉身份信息，并使用该身份登录其网站。

　　使用微信、钉钉开放平台提供的二维码扫描登录有如下两种方式：

- 开放平台提供扫码登录页面
- 二维码内嵌到网站页面

　　对于第一种方式，开放平台提供扫码登录页面，我们在第三方网站上点击《通过微信登录》按钮后，会跳转到微信域下的二维码界面，然后，打开微信，扫描二维码，点击授权网站登录按钮，接着网页上会自动跳转到第三方网站的页面，并用微信用户身份登录成功。

　　对于第二种方式，开放平台提供JS，让网站生成二维码并内嵌在网页内，供用户扫描，这样，不用再跳转到微信、钉钉域下进行二维码扫描，提升了扫码登录的流畅性和成功率。用户扫描后，跟第一种方式一样操作，便登录成功了。

#### 　　**原理** ####


　　微信、钉钉的原理基本一致，可能有细微差别，这里我通过钉钉的实现原理来进行说明。

　　**步骤１**：

　　**对于第一种方式**，首先，在网站上点击《通过钉钉登录》后，需要跳转到如下链接：
	
	　　https://oapi.dingtalk.com/connect/qrconnect?appid=APPID&response_type=code≻ope=snsapi_login&state=STATE&redirect_uri=REDIRECT_URI

　　url里的参数需要换成第三方Web系统对应的参数。在钉钉用户扫码登录并确认后，会302到你指定的redirect_uri，并向url参数中追加临时授权码code及state两个参数。

　　**对于第二种方式**，首先在页面中先引入如下JS文件（支持https）：

	　　<script src="g.alicdn.com/dingding/dinglogin/0.0.2/ddLogin.js"></script>

　　然后在需要使用二维码登录的地方，实例化如下JS对象：

	　　var obj = DDLogin({
     　　id:"login_container",//这里需要你在自己的页面定义一个HTML标签并设置id，例如<div id="login_container"></div>或<span id="login_container"></span>
     　　goto: "",
     　　style: "",
     　　href: "",
     　　width : "300px",
     　　height: "300px"
 	　　});
　　

　　其中goto参数需要这样构造：https://oapi.dingtalk.com/connect/oauth2/sns_authorize?appid=APPID&response_type=code&scope=snsapi_login&state=STATE&redirect_uri=REDIRECT_URI

　　并且要将goto参数urlencode编码

　　您引入的js会在获取用户扫描之后将获取的loginTmpCode通过window.parent.postMessage(loginTmpCode,’*’);返回给您的网站。您可以通过以下代码获取这个loginTmpCode：

	　　<br>var hanndleMessage = function (event) {
    　　var loginTmpCode = event.data; //拿到loginTmpCode后就可以在这里构造跳转链接进行跳转了
    　　var origin = event.origin;
	　　};
	　　if (typeof window.addEventListener != 'undefined') {
    　　window.addEventListener('message', hanndleMessage, false);
	　　} else if (typeof window.attachEvent != 'undefined') {
    　　window.attachEvent('onmessage', hanndleMessage);
	　　}

　　通过JS获取到loginTmpCode后，需要由你构造并跳转到如下链接。
https://oapi.dingtalk.com/connect/oauth2/sns_authorize?appid=APPID&response_type=code≻ope=snsapi_login&state=STATE&redirect_uri=REDIRECT_URI&loginTmpCode=loginTmpCode

　　此链接处理成功后，会302到你goto参数指定的redirect_uri，并向url参数中追加临时授权码code及state两个参数。

![]({{site.pictureurl}}26.jpg?raw=true)

　　**步骤２**：

　　跳转网页拿到code后，需要通过code换取用户身份信息，微信和钉钉通过code换取用户身份的步骤有些许不同

　　钉钉参考：[https://open-doc.dingtalk.com/docs/doc.htm?spm=a219a.7629140.0.0.yCl0vv&treeId=168&articleId=104882&docType=1](https://open-doc.dingtalk.com/docs/doc.htm?spm=a219a.7629140.0.0.yCl0vv&treeId=168&articleId=104882&docType=1)

　　微信参考：[https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419316505&token=&lang=zh_CN](https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419316505&token=&lang=zh_CN)


### **二. 自实现二维码扫描登录** ###

　　若不借助微信、钉钉开放平台的帮助，而想自己完整实现一套二维码扫描登录方案，可以这样实现，首先参考下图：

![]({{site.pictureurl}}27.jpg?raw=true)

　　首先，在待登录网站，生成一个二维码，该二维码本质上是一个url，带上一个唯一的key的参数，当生成该二维码后，二维码端会不断地轮询服务器，来查找对应的key是否已获得用户信息。若获得，则将用户信息返回给二维码端，并进行登录

　　用户获得二维码后，用网站相应的手机App扫描该二维码，跳转到二维码所代表的url上，在该url上，需要获取用户的身份信息，当用户点击授权登录后，需要将该用户身份信息连同key一起发送到服务器端，服务器端，将用户身份信息写到key对应的value中。

　　这样，当用户授权登陆后，二维码端自然就能获得用户的身份信息了，也就可以登录了。

### **三. 完整的二维码扫描登录方案** ###

　　当然作为一个完整的扫码登录方案，我们应该将这两种方式结合起来，既支持通过微信、钉钉扫码登录、也支持通过自己的手机App登录，这样才算是完整的解决方案。