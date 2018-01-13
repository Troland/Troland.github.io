---
title: DCloud 个推总结
date: 2018-01-13 16:54:07
tags:
- app
- mui
- hybrid
categories:
- Tech
---

最近公司有用 DCloud 的产品来开发 app 里面涉及到一些问题听我细细道来。
推送使用的是个推平台，参见[官网](http://getui.com/cn/index.html)。这里分为[客户端](http://docs.getui.com/getui/more/plugin/)和[服务端](http://docs.getui.com/getui/more/plugin/)。DCloud的HBuilder里面有集成推送的 sdk。

推送的步骤可参见官方[文档](http://ask.dcloud.net.cn/article/34)。
大致的推送流程是：服务端发送推送 -> 个推平台会进行处理 -> App 的个推进程收到推送信息 -> App进行消息处理和推送事件的监听。

这里主要遇到的问题有如下：

- 应该采用何种模板才能够让 ios 和 android 客户端都可以收到推送？
  
  这里只能采用透传模板TransmissionTemplate，查看个推官方 [FAQ](http://docs.getui.com/question/getui/ios/)
  
- 关于离线的信息的推送，android和ios应该如何处理？
    
  关于android的离线处理，我询问了安卓的小伙伴，得到的答复是无法保证离线状态下android可以收到推送。ios则可以通aps苹果推送服务来得到推送信息。
  
  这里有一个地方需要注意的是服务端在发送ios离线信息的时候需要在`customMsg`参数下添加传递到客户端的参数比如：
  
  ```
    let payload = new APNPayload();
    let alertMsg = new SimpleAlertMsg();
    alertMsg.alertMsg="";
    payload.alertMsg = alertMsg;
    payload.badge=5;
    payload.contentAvailable =1;
    payload.category="";
    payload.sound="";
    payload.customMsg.body = "rolman";
  ```
  
  ios当app在线的时候会通过`receive`事件创建消息到消息中心，若离线则通过苹果aps推送到消息中心。
  
  需要注意的是： **ios的消息展示方式，这里统一为不管app是在前台还是后台都是把推送消息显示在消息中心，这样就和android的信息展示方式统一。**
 
- 在 ios app 登记的时候必须是苹果推服务证书！否则无法通过 aps 接收推送信息。
   
   查看[这里](http://docs.getui.com/question/getui/ios/)，客户端看下devicetoken是否有获取到，获取到了，应用配置里面上传证书的地方有个测试一下，测试看看返回什么结果，返回测试可用才说明证书环境是一致的。

- 关于角标的处理
    
  　app 角标的处理是个比较复杂的问题，考虑的问题会比较多，这里只做了简单的处理：
  
    ```
      document.addEventListener('newintent', function () {
    		plus.runtime.setBadgeNumber(0);
    	}, false);
    
    	document.addEventListener('resume', function () {
    		plus.runtime.setBadgeNumber(0);
    	}, false);
    
    	document.addEventListener('foreground', function () {
    		plus.runtime.setBadgeNumber(0);
    	}, false);
    ```
  　
- 在`receive`事件中接收到的参数问题
  
  在`receive`事件中传过来的参数`ios`和`android`接收到的数据类型会不一致。就是那个`msg.payload`这个参数即`transmissionContent`，ios解析为object，android解析为string。
  　
- ios 在`receive`事件中必须区分是本地消息还是服务器推送的消息否则会一直循环创建消息。
  
消息处理的代码必须写在 HBuilder 的 manifest.json 文件中的页面入口页面中。

下面列出消息处理的代码：

```
$.plusReady(function() {
    document.addEventListener('newintent', function () {
    	plus.runtime.setBadgeNumber(0);
    }, false);
    
    document.addEventListener('resume', function () {
    	plus.runtime.setBadgeNumber(0);
    }, false);
    
    document.addEventListener('foreground', function () {
    	plus.runtime.setBadgeNumber(0);
    }, false);

    // 监听在线消息事件
    // ios 和 android 传过来的 msg.payload，ios 解析为 object，android 解析为 string
	plus.push.addEventListener('receive', function( msg ) {
	   var payload = (typeof msg.payload == 'string' ?
								       JSON.parse(msg.payload) : msg.payload);


		// 判断是否 ios
		if ($.os.ios) {
			// 只有当不是本地消息的时候才创建
			if (!payload._isLocal) {
				payload._isLocal = true;
				msg.payload = payload;
				plus.push.createMessage(payload.body, JSON.stringify(payload), {
					 title: payload.title,
					 cover: false
				});
			}
		} else {
			plus.push.createMessage(payload.body, msg.payload, {
				 title: payload.title,
				 cover: false
			});
		}
	}, false);

	// 监听在消息通知点击，这里写具体的跳转业务
	plus.push.addEventListener('click', function( msg ) {
		var payload = (typeof msg.payload == 'string' ? JSON.parse(msg.payload) : msg.payload),
		  type = payload.type,
		  openUrl;

	}, false);
});
```

## 总结

这里在进行调试的时候需要考虑蛮多的东西，多去查查个推平台的文档，然后自己可以在本地起个 nodejs 服务器，然后那个苹果的推服务的证书也得测试于是。这样就可以解决问题。

## 思考

这里安卓的离线的消息通知仍然有些问题，关于透传的标准格式会不会触发 click 事件，这里有个[帖子](http://ask.dcloud.net.cn/article/836)。

还有另一个问题就是关于角标的处理，安卓是不是也可以设置？还有如何根据业务，比如当用户读取消息再让角标减少？
  　