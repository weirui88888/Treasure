![jssdk-cover](https://show.newarray.vip/blog/jssdk-cover.png)
## 背景

由于之前工作中的一些场景，前端做的网页，在微信浏览器中打开的时候，需要涉及到一些分享相关的功能，而你的网页想要具备分享的功能和定制化，那么你就需要使用 jssdk。其实微信 jssdk 赋予开发者很多能力，包括但不限于，**拍照或从手机相册中选图接口**、**使用微信内置地图查看位置接口**、**调起微信扫一扫接口**、**分享相关**，具体的可以参考[微信开放文档](https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/JS-SDK.html)，本文中仅以**分享功能**作为例子。但是使用它，是有一定的**限制条件**的，在实践的过程中，会遇到一些坑。这里记录下关于其的一些理解和实践。

## 例子

先举个例子，让我们感性的认识到，jssdk 到底具备什么样的能力。下面这场图，表明了不使用 jssdk 分享功能配置以及使用其两者之间最明显的区别

<img src="https://show.newarray.vip/blog/jssdk-1.png" alt="image-20220729120203135" style="zoom: 33%;" />

可以看到，不使用分享相关的功能，会有以下几点不尽人意的地方

- 分享的标题就是默认的网页的标题
- 分享的内容区域，就是网页的地址
- 分享的图片展示区域是一个占位图，不够醒眼

现在这个看脸的社会，长得好看，确实是一种能力。所以企业是不会放过通过一些营销手段进行低成本的获客能力

## 简介

接下来会简要的说明下是如何使用微信 jssdk，以及在使用的时候会遇到哪些问题，进而告知如何避免这些问题

### 如何使用

#### 第一步

第一步先要在微信公众平台上注册一个微信公众号，还必须要是认证的企业或组织，流程走下来大概要个两三天的时间吧，只有认证了的公众号才有微信 JSSDK 的权限，个人的公众号不行，这里的不行，是指个人公众账号，可能只会具备一些基础的 API 能力，像分享这样的功能是无法使用的。账号申请成功后会有 AppID 和 AppSecret，这个相当于你公众号的秘钥，第二步需要用到。然后在 JS 接口安全域名中加入你调微信 SDK 时的页面的地址，在设置了安全域名的路径下才能够成功调 SDK。

从上述可以提炼几点有用信息

- 想要使用微信 SDK 做一些事，首先需要注册一个公众号，且公众号是一个企业公众号
- 需要在微信公众平台配置将来想要使用微信 SDK 的网址的域名
  - 比如你的网址是https://helloworld.newarray.vip/xxx/index.html， 那么你就需要将域名配置成https://helloworld.newarray.vip，这样，该域名下面的服务，也就是我们的网站，才可以使用微信SDK
- 只有指定域名下的网页，才能使用微信 SDK 的一些功能。换句话说，你本地开发环境 IP 访问的网址，是没办法使用的（限制 1，下面会讲到如何规避这种限制）

#### 第二步

需要一个后端接口，这个接口会返回给你一些鉴权信息，明确的告诉你，如何去初始化微信 SDK。而这个后端接口返回给你的鉴权信息，就是通过第一步中提到的 AppID 和 AppSecret，再通过一系列的计算，生成 token 以及签名。初始化之后，我们就可以利用微信 SDK 赋予开发者的能力，做各种事。

从上述可以提炼几点有用信息

- 看起来，光靠前端，是没法使用 SDK 的，需要后端同学的配合（限制 2，下面会讲到如何规避这种限制）
- 我的关注点在怎么能够轻松地使用 SDK，难道我还要明白其中的算法，才能生成签名信息，社死...

#### 第三步

通过引入官网的 SDK 脚本，正式使用

```javascript
<script typet="text/javascript" src="https://res2.wx.qq.com/open/js/jweixin-1.6.0.js"></script>
<script>
// 先初始化，初始化用到的数据是第二步中，后端返回的
wx.config({
  debug: true,
  appId: 'wxcdbc8f41e4d4badd',
  timestamp: 1659065946,
  nonceStr: 'helloworld',
  signature: '7bba9ed97fd48af0a5ccf575c089444417708106',
  jsApiList: ['chooseImage', 'checkJsApi','updateAppMessageShareData']
})

wx.ready(function () {
  // 检查是否支持指定的js接口，比如选择图片、预览图片、自定义分享
  wx.checkJsApi({
  	jsApiList: ['chooseImage', 'previewImage', 'updateAppMessageShareData'], // 需要检测的JS接口列表
  	success: function (res) {
  		console.log(res)
 		}
 })
  // 自定义分享
  wx.updateAppMessageShareData({
    title: '这也是网页的标题', // 分享标题
    desc: '每当你感觉到花瓣慢慢枯萎', // 分享描述
    link: location.href, // 分享链接，该链接域名或路径必须与当前页面对应的公众号 JS 安全域名一致
    imgUrl: 'https://show.newarray.vip/blog/%E7%9C%9F%E7%9A%84%E5%BE%88%E6%A3%92.png', // 分享图标
    success: function () {
      alert('分享成功')
    }
  })
})
</script>
```

### 如何规避上面的一些限制

- 限制一：在特定域名下才有效
- 限制二：需要后端提供签名等信息才能使用 JSSDK

这个时候，你肯定会想以下两个问题：

- 作为一个前端，我可能只是单纯的想要在本地开发的时候，就能够能够直观的看到我的一些业务逻辑是否符合要求，难道非得上生产之后才能验证效果吗？
- 有没有什么办法可以不依赖于后端，我也能拿到初始化 SDK 的签名等信息？

于是乎，我们才真正的进入了本篇文章的**正题**

## 正题

首先，微信还是挺人性化的，给我们提供了一个[测试账号](https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login/)

登陆账号之后，我们会看到如下的界面

<img src="https://show.newarray.vip/blog/jssdk-2.png" alt="image-20220729131043433" style="zoom:67%;" />

针对这张图，我简单的解释下

- 测试号信息提供了 appId 和 appsecret，这个相当于上文中提到的，你已经有了自己的企业公众号，公众号会提供 appId 和 appsecret

- JS 接口安全域名你需要配置成你自己的本地的域名，不同带 PATH，域名即可，配置了之后，该服务下的网页就可以拥有正常使用 SDK 的能力了

  - 可以看到，这里我配置的是 weirui.bybutter.com，并不是我本地的 ip，形如http://192.163.3.108:8080， 这是因为我本地安装了一个[ngrok](https://ngrok.com/download)，对于它，研究不深，就是知道它具备内网穿透的功能，说人话就是，将你本地的ip，映射到一个域名上

  使用它也很简单，比如你的网页启动的时候端口号是 8080，那么你只需要在终端中这样启动即可

  ```javascript
  ngrok -subdomain=weirui 8080
  ```

  这里的三级域名，我设置的是自己的，端口号需要配置成你本地服务的端口号，比如我本地的服务是通过`http-server`启动的，默认的端口号就是 8080，这个时候，没意外的话。你就会发现通过 https://yourdomain/index.html 就已经能够正常的访问你通过`http-server`启动的服务了

- 测试号二维码，是指需要你通过微信扫一扫，关注了测试账号，之后才能在手机或者微信开发者平台上面，才能访问你具备 sdk 的服务

#### 获取鉴权信息

这里没什么特别想表述的，只是罗列出来如何通过官方提供的测试接口在 postman 中去拿到最终我要初始化的配置信息，也不太想去关注其中的实现，因为最终，你生产上的服务，还是需要依赖于后端同学提供这些信息的

##### 1.首先，获取 access_token

```javascript
https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=yourappid&secret=yoursecret
```

postman 中调用该请求后，响应中，会返回给我们 access_token

##### 2.用上一步获取的 access_token 获取 ticket

```
https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token=youraccesstoken&type=jsapi
```

postman 中调用该请求后，响应中有两个字段我们需要关注下，一个是 ticket，这个 ticket 就是生成签名的关键，**还有个 expires_in 字断，没研究，但是语意上可以看出来是指过期时间，估计是超过了指定的时间就失效了，需要你重新生成下**

##### 3.接着，通过[微信 js 签名工具](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fdebug%2Fcgi-bin%2Fsandbox%3Ft%3Djsapisign)去生成初始化需要的签名

<img src="https://show.newarray.vip/blog/jssdk-3.png" alt="image-20220729134018588" style="zoom:67%;" />

四个输入框，分别是

- 第 2 步获取的 ticket
- 随机签名字符串，随便填个就行
- 时间戳，Date.now()/1000 即可
- url 指，你要在哪个服务下使用 SDK，这里提一嘴，真实后端提供给你的签名信息，也是需要根据 url 去拿到的

##### 4.用生成的签名初始化 sdk

```javascript
wx.config({
  debug: true,
  appId: 'wxcdxxxxxxd4badd',
  timestamp: 1659065946,
  nonceStr: 'helloworld',
  signature: '7bba9ed97fd48af0a5ccf575c089444417708106',
  jsApiList: ['chooseImage', 'checkJsApi', 'previewImage', 'updateAppMessageShareData']
})
```

这里我有一点不确定，timestamp 和 nonceStr，是否需要和第三步中填写的保持一致，反正我是用一致的，并没有问题

##### 5.开始使用 SDK 吧

具体的使用规则和注意事项，参考官网文档即可

## 进阶

上述，我们只是通过最简单的 script 脚本的方式，引入了 SDK，进而使用它。然而在实际工程中，我们一般需要进行一下简单的封装，便于各个工程复用，这里不做延伸

另外一个常用的，就是微信开放标签，它的作用就是使得我们的网页，在微信中打开的时候，具备通过开放标签直接跳转企业 app 的能力，方便企业获客和引流，有兴趣的可以去了解下

## 总结

微信 SDK 是我们前端同学在开发过程中，肯定要涉及到的，为了规避掉一些使用限制，我们可以采用一些另辟蹊径的实现。本篇[示范代码](./index.html)在这
