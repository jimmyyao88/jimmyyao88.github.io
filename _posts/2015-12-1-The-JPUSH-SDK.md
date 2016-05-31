---
layout: post
title: 使用swift集成JPUSH SDK
category: note
excerpt: jpush sdk swift version
---
首先声明这不是广告,我和JPush 没有任何利益关系…...为了节省成本，团队里更多的会去考虑到去使用各种xaas 服务，所以当时在选择push service 的时候 ,觉得pusher 做的真的很好，但是离国内最近的机房只有东京和新加坡，一个socket连接延迟还是很明显的，所以就只能在JPush 和 leancloud里选了，最后还是选择了JPush。我自己又嫌Objective-C的反人类语法恶心，然后项目用的本来就是swift ，所以就记个JPUSH 的swift sdk 集成笔记，免得下次又被坑在什么地方。

其实集成的时候没什么任何问题，大部分时间都被坑在了Apple的那些证书上，配置的时候一定要看清楚报的具体的错误信息

### 前端ios的集成: appdelegate.swift

先 bridge 里去添加 头文件 来让OC 和 swift 混编 这个不说了

```swift
    //重写 didFinishLaunchingWithOptions 
    func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
    	//用来获取获取ios 系统内部通知权限
        registerForPushNotifications(application)
		//初始化JPush SDK 
        JPUSHService.setupWithOption(launchOptions, appKey: "YOUrKEY", channel: "App Store", apsForProduction: false)
        //用来获取应用内消息  会在networkDidRecieveMessage 这个函数里面 去执行具体的behavior
        let defaultCenter:NSNotificationCenter = NSNotificationCenter.defaultCenter()
 defaultCenter.addObserver(self,selector:#selector(AppDelegate.networkDidReceiveMessage(_:)), name: kJPFNetworkDidReceiveMessageNotification, object: nil)
        return true
    } 
    func registerForPushNotifications(application: UIApplication) {
    //设置类型
        let notificationSettings = UIUserNotificationSettings(
            forTypes: [.Badge, .Sound, .Alert], categories: nil)
        application.registerUserNotificationSettings(notificationSettings)
    }
    
    func application(application: UIApplication, didRegisterUserNotificationSettings notificationSettings: UIUserNotificationSettings) {
        if notificationSettings.types != .None {
            application.registerForRemoteNotifications()
        }
    }
    
    func application(application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: NSData) {
        print("deviceToken",deviceToken)
        //向JPUSH 后端发送 device token
        JPUSHService.registerDeviceToken(deviceToken)
        
    }
    
    func application(application: UIApplication, didReceiveRemoteNotification userInfo: [NSObject : AnyObject]) {
    	这个方法下接收JPUSH 发送的信息
        JPUSHService.handleRemoteNotification(userInfo)
        print("GOT PUSH NOTIFICATION")
    }
    
```

### 后端service: 使用node 向JPUSH 后端发送推送和应用内消息

这里很方便的是JPUSH官方已经给node封装好了JPUSH的那些RESTFUL接口了

```shell
npm install jpush-sdk --save
```

controller：

```javascript
var JPush = require('jpush-sdk');
var client = JPush.buildClient('YOUR KEY', 'YOUR KEY');

exports.pushNotification = function(req,res){
  client.push()
      .setPlatform(JPush.ALL)
      .setAudience(JPush.ALL)
      .setNotification('Hi, JPush', JPush.ios('ios alert', 'happy.caf', 5))
      .send(function(err, response) {
          if (err) {
              console.log(err.message);
          } else {
              console.log('Sendno: ' + res.sendno);
              console.log('Msg_id: ' + res.msg_id);
              res.send('success');
          }
      });
}


exports.pushSocketNotification = function(req,res){
  console.log("pushSocketNotification");
  client.push()
      .setPlatform(JPush.ALL)
      .setAudience(JPush.ALL)
      .setMessage('Good Luck')
      .send(function(err, response) {
          if (err) {
              if (err instanceof JPush.APIConnectionError) {
                  console.log(err.message);
                  //Response Timeout means your request to the server may have already received, please check whether or not to push
                  console.log(err.isResponseTimeout);
              } else if (err instanceof  JPush.APIRequestError) {
                  console.log(err.message);
              }
          } else {
              console.log('Sendno: ' + res.sendno);
              console.log('Msg_id: ' + res.msg_id);
              res.send("success");
          }
      });
}
```

给两个函数都写一个http 路由做成一个通用一点的service 会方便一些。



PEACE





