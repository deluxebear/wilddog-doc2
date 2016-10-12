﻿title: 建立会话
---

完成 Wilddog Video SDK 安装后，即可建立会话。
在发起会话之前，需要初始化 Wilddog Video SDK，并配置本地视频流。
以下展示了如何实现建立会话。

## 初始化 Client

发起会话之前需要通过初始化 Client 来连接客户端和野狗服务器。初始化 Client 时需要指定 Video SDK 的交互路径，客户端和服务器以及客户端之间都是通过该路径进行交互，只有相同交互路径下的 Client 能够发起或加入会话。建议该路径下不要存储其他数据。

选择 `Server-based` 会话时，初始化 Client 时的交互路径应和控制面板中的交互路径保持一致。

需要注意的是，初始化 Client 之前，要先经过身份认证。这里采用匿名登录的方式认证，开发者可以根据需要选择邮箱密码、第三方或自定义方式。

```javascript
var config = {
    authDomain: "<appId>.wilddog.com",
    syncURL: "https://<appId>.wilddogio.com"
};
//初始化Wilddog Sync
wilddog.initializeApp(config);
//创建交互路径的 Wilddog Sync 引用，该路径可以自定义
var ref = wilddog.sync().ref(“你的自定义路径”); 
var client = wilddog.video().client();

//初始化 Video Client 之前，要先经过身份认证。这里采用匿名登录的方式，开发者可以根据需要选择邮箱密码、第三方或自定义方式。
wilddog.auth().signInAnonymously().then(function(user){
    client.init({ref: ref, user: user}, initComplete);
}).catch(function (error) {
    // Handle Errors here.
    console.log(error);
});
```

## 配置本地媒体流

本地媒体流包括音频和视频。需要在发起会话前配置本地媒体流。会话建立后该媒体流会发给其他 Clients。

例如，可以创建一个只有视频且分辨率为 640X480 的流，并展示到页面上：

```javascript
wilddog.video().createStream({
    audio:false,
    video:'standard'
},function(localStream){
    var element = document.getElementById('local_view');
    //绑定到local_view中展示
    localStream.attach(element);
});
```

## 发起会话

会话的建立基于邀请机制，只有另一个 Client 接受了会话邀请，会话才能建立成功。

发起会话示例：

```javascript
client.init({ref:ref,user:user}, function(err){
    //邀请他人加入会话，选择 P2P 模式，localStream 为之前创建的本地流
    client.inviteToConversation({
        mode:'p2p',
        participantId:'被邀请者的Wilddog ID',
        localStream:localStream
    }).then(function(conversation){
        conversation.on('participant_connected', function(participant){
            console.log('A remote Participant connected: ' + participant.participantId);
        });
    });
});
```