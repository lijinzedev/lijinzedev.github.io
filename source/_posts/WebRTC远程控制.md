---
title: WebRTC远程控制
top: false
cover: false
toc: true
mathjax: true
categories:
  - WebRTC
tags:
  - WebRTC
date: 2022-02-16 16:52:54
password:
summary:
---

# WebRTC远程控制

1. 傀儡端告知控制端本机控制码
2. 控制端输入控制码连接傀儡端
3. 傀儡端将捕获的画面传给控制端
4. 控制端的鼠标和键盘指令传送至傀儡端
5. 傀儡端响应控制指令

![image-20220217113404616](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202202171134664.png)

# 一、WebRTC

![image-20220218170034934](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202202181700072.png)

## 1 MediaStream API

- 媒体内容的流
- 一个流对象可以包含多轨道，包括音频和视频轨道等
- 能通过 WebRTC 传输
- 通过 标签可以播放

![image-20220218170635265](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202202181706311.png)

> **如何捕获桌面/窗口流？**

**navigator.mediaDevices.getUserMedia（MediaStreamConstraints）**

返回：Promise，成功后 resolve 回调一个` MediaStream 实例对象 `

参数：MediaStreamConstraints 

* **audio: Boolean | MediaTrackConstraints**  
* **video: Boolean | MediaTrackConstraints**  
  *  **width：分辨率** 
  * **height：分辨率** 
  * **frameRate： 帧率(调整流畅度)，比如 { ideal: 10, max: 15 }**

```javascript
async function getScreenStream(){
    const sources = await desktopCapturer.getSources({types: ['screen']})
    navigator.webkitGetUserMedia({
        audio:false,
        video: {
            mandatory: {
                chromeMediaSource: 'desktop',
                chromeMediaSourceId: sources[0].id,
                maxWidth: window.screen.width,
                maxHeight: window.screen.height
            }
        }
        // 捕获成功放在callback中的第一个参数
    },(stream)=>{
        peer.emit('add-stream', stream)
    }, (err)=> {
        // 这里必须写，不然报错
        console.log(err)
    })
}

```

> **如何播放媒体流对象**

```javascript
var video = document.querySelector('video')
video.srcObject = stream
// 在视频加载完元数据之后播放
video.onloadedmetadata = function(e) {
 video.play();
}

```

## 2 WebRtc 传输桌面流

[electron desktopCapture](https://www.electronjs.org/zh/docs/latest/api/desktop-capturer)

![image-20220223175406625](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202202231754190.png)

### SDP

SDP（Session Description Protocol）是一种会话描述协议，用来描述多媒体 会话，主要用于协商双方通讯过程，传递基本信息。

- SDP的格式包含多行，每行为`<type>=<value>`
- `<type>`：字符，代表特定的属性，比如v，代表版本
- `<value>`：结构化文本，格式与属性类型有关，UTF8编码

## 3 NAT (网络地址转换)

![image-20220223180011243](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202202231800295.png)

P2P数据交换是需要通过服务器，如果你要发信息的话，先传到服务端，然后发给对应的用户。如果走P2P连接，肯定会比走服务器来的快而且还更安全。

为什么要做一层服务端的转发，其实答案非常多，其中一个原因就是我们在端到端的通信时，需要知道对方的公网IP和端口号，实际上这不是一件容易的事情，因为我们的网络环境里充斥着NAT技术，NAT是网络地址转换的一个缩写，为什么这个技术会出现呢，如果大家对网络知识有一定了解的话，IPv4地址早就不够用了，它不够用主要有两个原因：

第一个原因是IPv4，它本来就是一个32位的整数，理论上只能支持40多亿的地址，这个数远远小于世界的总人口；

第二个原因是IP地址在地理位置上得分配不均，美国非常的多，中国是非常稀缺的，中国人均只有0.06个地址，而占据世界人口56%的亚洲只能够分到9%的地址。

于是人类为了解决地址的问题，NAT就出现了，在NAT内每个设备它都会有一个独立的局域网地址，然后它们在跟外网连接的时候会共用同一个公网IP，而NAT它负责维护一个包括本地IP端口和外网IP端口的一个映射表。

![image-20220223180025152](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202202231800186.png)

这里面就会涉及到NAT打洞，基本方法就是由服务端跟其中一方ClientB建立一个连接，这时候NAT里面就会建立一个端口号的内外网的一个映射，之后我们服务端就可以知道ClientB外网的IP和端口，然后传给ClientA，最后ClinetA它就可以直接利用NAT打好了洞，然后跟ClientB进行一个通讯。在webRTC里面已经有一个集成好的机制，就是STUN服务，当ClientA和ClientB要做P2P连接的时候，它首先第一步需要跟我们的服务器做一个穿越打洞，然后将打洞的结果传到ClientB下，同样ClientB也需要做一个类似的操作，这样子我们通过服务器的帮助下，这样ClientA和ClientB就能拿到对方真实公网的IP和端口。

### webRTC的NAT穿透是一整个机制，我们管它叫ICE

ICE（Interactive Connectivity Establishment）交互式连接创建

- 优先STUN (Session Traversal Utilities for NAT)，NAT会话穿越应用程序
- 备选TURN (Traversal Using Relay NAT) ，中继NAT实现的穿透
  1. Full Cone NAT - 完全锥形NAT
  2. Restricted Cone NAT - 限制锥形NAT
  3. Port Restricted Cone NAT 端口限制锥形NAT
  4. Symmetric NAT 对称NAT


![image-20220223180039579](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202202231800617.png)

![image-20220223180049286](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202202231800317.png)

首先控制端，会先发起一个询址，然后我们的STUN服务会将这个洞打好，然后返回给我们的控制端，这个时候控制端就知道自己的外网的IP和端口，随后我们需要通过一定的介质然后给到傀儡端，这里面跟PeerConnection的SDP传输是一样的，你可以通过任何的介质来传输，像邮件、微信什么都可以，傀儡端拿到了IceEvent之后，它会通过addIceCandidate的方法添加我们的代理，这样的话，我们的傀儡端就知道控制端的一个外网IP了，类似的傀儡端也会拿到自己的IP和端口给到控制端，控制端添加ICE代理，这样子，我们的P2P才是真正的建立成功。

## 4 信令服务

>  就是webRTC之间传递消息的服务器，实现连接两端

![image-20220223180143347](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202202231801391.png)



## 5 RTCDataChannel

![image-20220223180257566](https://raw.githubusercontent.com/lijinzedev/picture/main/img/202202231802633.png)

```javascript
var pc = new RTCPeerConnection();

var dc = pc.createDataChannel("robotchannel"); // 创建一个 datachannel

dc.onmessage = function (event) { //接收消息

 console.log("received: " + event.data);

};

dc.onopen = function () { // 建立成功

 console.log("datachannel open");

};

dc.onclose = function () { // 关闭

 console.log("datachannel close");

};

dc.send('text') // 发送消息

dc.close() // 关闭

pc.ondatachannel = function(e) {} // 发现新的datachannel
```



# 二、robotjs

[官网](https://robotjs.io/docs/)

## 2.1 安装

* 手动编译 
  * npm rebuild —runtime=electron —disturl=https://atom.io/download/atom-shell \  —target= —abi=<对应版本abi>
  * process.versions.electron，可以看到electron版本
  * process.versions.node 可以看到 node 版本，之后再 abi_crosswalk 查找 abi 
* electron-rebuild 
  * npm install electron-rebuild —save-dev 
  * npx electron-rebuild

## 2.2 编译

请确保在对应的平台已经安装对应的依赖

```
Windows
  windows-build-tools npm package (npm install --global --production windows-build-tools 
  from an elevated PowerShell or CMD.exe)
Mac
  Xcode Command Line Tools.
Linux
  Python (v2.7 recommended, v3.x.x is not supported).
  make.
  A C/C++ compiler like GCC.
  libxtst-dev and libpng++-dev (sudo apt-get install libxtst-dev libpng++-dev).

```

> 需要关闭app.allowRendererProcessReuse

```javascript
import {app, BrowserWindow} from 'electron'
import {create} from './mainWindow'
import {ipc} from "./ipc";

app.allowRendererProcessReuse = false  // 默认为true,防止在渲染器进程中加载非上下文感知的本机模块。

app.on('ready', () => {
    process.env['ELECTRON_DISABLE_SECURITY_WARNINGS'] = 'true'; // 关闭web安全警告
    ipc()
    create()
})

```



# 三、vkey

# 四、代码片段

## 客户端

### robot.js

```javascript
const {ipcMain} = require("electron")
const robot = require('robotjs')
const vkey = require('vkey')
// 鼠标滚动系数
const SCROLL_SPEED_MULTIPLIER = 2;
let mouseSpeed = 20;

function handleMouse(data) {
    let {clientX, clientY, screen, video} = data
    // data {clientX, clientY, screen: {width, height}, video: {width, height}}
    let x = clientX * screen.width / video.width
    let y = clientY * screen.height / video.height
    console.log(x, y)
    robot.moveMouse(x, y)
    robot.mouseClick()
}
function handledbClick(data){
    let {clientX, clientY, screen, video} = data
    // data {clientX, clientY, screen: {width, height}, video: {width, height}}
    let x = clientX * screen.width / video.width
    let y = clientY * screen.height / video.height
    console.log(x, y)
    robot.moveMouse(x, y)
    robot.mouseClick('left',true)
}


function handleMove(data){
    let {clientX, clientY, screen, video} = data
    // data {clientX, clientY, screen: {width, height}, video: {width, height}}
    let x = clientX * screen.width / video.width
    let y = clientY * screen.height / video.height
    console.log(x, y)
    robot.moveMouse(x, y)
}
function handlewheel(data) {

    console.log(data)
    let isMac = 1;
    // mac 平台方向相反
    if (process.platform === 'darwin') {
        isMac = -1;
    }
    if (data) {
        // 向下
        robot.scrollMouse(0, mouseSpeed * SCROLL_SPEED_MULTIPLIER * isMac);
    } else {
        // 向上
        robot.scrollMouse(0, -mouseSpeed * SCROLL_SPEED_MULTIPLIER * isMac);
    }
}

function handleKey(data) {
    // data {keyCode, meta, alt, ctrl, shift}
    const modifiers = []
    if (data.meta) modifiers.push('meta')
    if (data.shift) modifiers.push('shift')
    if (data.alt) modifiers.push('alt')
    if (data.ctrl) modifiers.push('ctrl')
    let key = vkey[data.keyCode].toLowerCase()
    if (key === '<space>') key = ' '
    console.log('handleKey--->' + key)
    if (key[0] !== '<') { //<shift>
        robot.keyTap(key, modifiers)
    } else {
        if (key === '<enter>') robot.keyTap('enter')
        else if (key === '<backspace>') robot.keyTap('backspace')
        else if (key === '<up>') robot.keyTap('up')
        else if (key === '<down>') robot.keyTap('down')
        else if (key === '<left>') robot.keyTap('left')
        else if (key === '<right>') robot.keyTap('right')
        else if (key === '<delete>') robot.keyTap('delete')
        else if (key === '<home>') robot.keyTap('home')
        else if (key === '<end>') robot.keyTap('end')
        else if (key === '<page-up>') robot.keyTap('pageup')
        else if (key === '<page-down>') robot.keyTap('pagedown')
        else if (key === '<control>') robot.keyTap('control')
        else console.log('did not type ' + key)
    }
}
function handleEvent(event) {
    console.log("handleEvent: " + JSON.stringify(event));
    if(!event.modifier) {
        event.modifier = "none";
    }
    let type = event.type;
    switch(type) {
        case 'keydown':
        case 'keyup': {
            let key = event.key;
            let state = type === 'keydown' ? 'down' : 'up';
            if(key.length === 1 && state === 'down') {
                robot.keyTap(key, event.modifier);
            }
            else {
                robot.keyToggle(key, state, event.modifier);
            }
            break;
        }
        case 'click':
        case 'dblclick': {
            let button = event.button;
            let double = type === 'dblclick';
            robot.mouseClick(button, double);
            break;
        }
        case 'mousedown':
        case 'mouseup': {
            let button = event.button;
            let state = type === 'mousedown' ? 'down' : 'up';
            robot.mouseToggle(state, button);
            break;
        }
        case 'mousemove': {
            let {x, y, screen, video} = event
            // data {clientX, clientY, screen: {width, height}, video: {width, height}}
             x = x * screen.width / video.width
             y = y * screen.height / video.height
            let modifier = event.modifier;
            if(modifier === 'left') {
                robot.dragMouse(x, y);
            }
            else {
                robot.moveMouse(x, y);
            }
            break;
        }
        case 'mousewheel':
        case 'wheel': {
            robot.scrollMouse(event.x, event.y);
            break;
        }
    }
}
module.exports = function () {


    // ipcMain.on('robot', (e, type, data) => {
    ipcMain.on('robot', (e,data) => {
        console.log(data)
        handleEvent(data)
        // console.log(data);
        // if (type === 'mouse') {
        //     handleMouse(data);
        // } else if (type === 'key') {
        //     handleKey(data);
        // } else if (type === 'wheel') {
        //     handlewheel(data);
        // }else if (type==='move'){
        //     handleMove(data)
        // }else if (type==='dbclick'){
        //     handledbClick(data);
        // }
    })
}


```

### screen.vue

```html
<template>
  <div>
    <Header></Header>
    <div class="mainAuth">
      <div class="container">
        <div class="video-box">
          <video id="localVideo" autoplay ref="localVideo"></video>
        </div>
        <div>
          <el-button class="primary" @click="callAdmin">远程协助</el-button>
        </div>
      </div>
    </div>
    <Footer></Footer>
  </div>
</template>
<style>
.container {
  width: 100%;
  display: flex;
  display: -webkit-flex;
  justify-content: space-around;
  padding-top: 20px;
}

.video-box {
  position: relative;
  width: 800px;
  height: 400px;
}

#remoteVideo {
  width: 100%;
  height: 100%;
  display: block;
  object-fit: cover;
  border: 1px solid #eee;
  background-color: #F2F6FC;
}

#localVideo {
  position: absolute;
  right: 0;
  bottom: 0;
  width: 240px;
  height: 120px;
  object-fit: cover;
  border: 1px solid #eee;
  background-color: #EBEEF5;
}
</style>
<script>
import Header from "@/components/Header.vue";
import Footer from "@/components/Footer.vue";
import {BASE_URL} from "@/api/config.js";
import da from "element-ui/src/locale/lang/da";
import {ipcRenderer, desktopCapturer} from 'electron'

let localStream, peerConn, dc;
let connectedUser = null;

export default {
  components: {
    Header,
    Footer,
  },
  data() {
    return {
      socket: null,
      call_username: '',
      socketUrl: `http://192.168.1.134:12007/screen/2/MACHINE`
    };
  },
  mounted() {

  },
  methods: {
    async callAdmin() {
      // 1. 开启websocket
      await this.openSocket();
      // 2. 初始化本地视频连接
      await this.initCreate();
      // 3. 创建peerConn连接
      this.createConnection();

      // todo 这里需要进行一次请求，请求获取优先级最高的管理员的ID
      connectedUser = '1'
      // 4. 发送视频邀请
      this.call();

      // 5. 生成Offer协议
      // 6. 设置本地描述
    },
    // 呼叫
    call() {
      this.send({
        event: 'call'
      });
    },
    handleLeave() {
      connectedUser = null;
      this.remote_video = '';
      peerConn.close();
      peerConn.onicecandidate = null;
      peerConn.onaddstream = null;
      this.$refs.localVideo.srcObject = null;
      localStream = null;
      if (peerConn.signalingState === 'closed') {
        // this.initCreate();
      }
    },
    handleAccept(data) {
      if (data.accept) {
        // Create an offer
        peerConn.createOffer(
            {
              offerToReceiveAudio: false,
              offerToReceiveVideo: true
            }
        ).then(offer => {
              this.send({
                event: 'offer',
                offer: offer,
              });
              peerConn.setLocalDescription(offer);
            },
            error => {
              alert('Error when creating an offer');
            });
      } else {
        alert('对方已拒绝');
      }

    },
    createConnection() {
      peerConn = new RTCPeerConnection();


      peerConn.addStream(localStream);
      peerConn.onaddstream = e => {

      };
      peerConn.onicecandidate = event => {
        setTimeout(() => {
          if (event.candidate) {
            this.send({
              event: 'candidate',
              candidate: event.candidate,
            });
          }
        });
      };

      dc = peerConn.createDataChannel('robotchannel', {reliable: false});

      dc.onopen = function () {
        console.log('opened')
      }

      dc.onmessage = function (e) {
        // console.log('onmessage', e, JSON.parse(e.data))
        // let {type, data} = JSON.parse(e.data)

        // console.log('robot', type, data)
        // if (type === 'mouse' || type === 'move' || type === 'dbclick') {
        //   data.screen = {
        //     width: window.screen.width,
        //     height: window.screen.height
        //   }
        // }
        // ipcRenderer.send('robot', type, data)
        let data = JSON.parse(e.data)
        data.screen = {
          width: window.screen.width,
          height: window.screen.height
        }
        ipcRenderer.send('robot', data)
      }
      dc.onerror = (e) => {
        console.log(e)
      }


    },
    async initCreate() {
      const self = this;
      const sources = await desktopCapturer.getSources({types: ['screen']})
      const screenStream = await navigator.mediaDevices.getUserMedia({
        audio: false,
        video: {
          mandatory: {
            chromeMediaSource: 'desktop',
            chromeMediaSourceId: sources[0].id,
            maxWidth: window.screen.width,
            maxHeight: window.screen.height
          }
        }
      })
      this.$refs.localVideo.srcObject = screenStream;
      this.$refs.localVideo.volume = 0.0;
      localStream = screenStream;

    },
    // 开启websocket
    async openSocket() {
      if (typeof WebSocket == "undefined") {
        console.log("您的浏览器不支持WebSocket");
      } else {
        console.log("您的浏览器支持WebSocket");
        //实现化WebSocket对象，指定要连接的服务器地址与端口  建立连接
        //等同于socket = new WebSocket("ws://localhost:8888/xxxx/im/25");
        //var socketUrl="${request.contextPath}/im/"+$("#userId").val();
        var socketUrl = this.socketUrl;
        console.log(BASE_URL);
        socketUrl = socketUrl.replace("https", "ws").replace("http", "ws");
        console.log(socketUrl);
        if (this.socket != null) {
          this.socket.close();
          this.socket = null;
        }
        this.socket = new WebSocket(socketUrl);
        //打开事件
        this.socket.onopen = function () {
          console.log("websocket已打开");
          //socket.send("这是来自客户端的消息" + location.href + new Date());
        };
        //获得消息事件
        this.socket.onmessage = this.webSocketOnMessage;
        //关闭事件
        this.socket.onclose = this.webSocketClose;
        //发生了错误事件
        this.socket.onerror = this.webSocketOnError;
      }
    },
    reject() {
      this.send({
        event: 'accept',
        accept: false,
      });
      this.accept_video = false;
    },
    accept() {
      this.send({
        event: 'accept',
        accept: true,
      });
      this.accept_video = false;
    },

    send(arg) {
      if (typeof WebSocket == "undefined") {
        console.log("您的浏览器不支持WebSocket");
      } else {
        arg.connectedUser = connectedUser;

        let json = JSON.stringify(arg);
        console.log(json)
        this.socket.send(json);
      }
    },
    webSocketOnError() {
      //连接建立失败重连
      this.openSocket();
      this.handleLeave();
    },
    webSocketOnMessage(e) {
      //数据接收
      console.log(e.data);
      let data = JSON.parse(e.data);
      // 更新用户信息
      if (data.userInfo) {
        this.userInfo = data.userInfo;
      }
      switch (data.event) {
        case 'show':
          this.users = data.allUsers;
          break;
        case 'join':
          break;
        case 'call':
          break;
        case 'accept':
          this.handleAccept(data);
          break;
        case 'offer':
          break;
        case 'candidate':
          this.handleCandidate(data)
          break;
        case 'msg':
          alert(data.message);
          break;
        case 'answer':
          this.handleAnswer(data);
          break;
        case 'leave':
          this.handleLeave(data)
          break;
        default:
          break;
      }
    },
    handleCandidate(data) {
      // ClientB 通过 PeerConnection 的 AddIceCandidate 方法保存起来
      peerConn.addIceCandidate(new RTCIceCandidate(data.candidate));
    },
    handleAnswer(data) {
      peerConn.setRemoteDescription(new RTCSessionDescription(data.answer));

    },

    webSocketClose(e) {
      //关闭
      console.log("断开连接", e);
      this.handleLeave();
    },
    hangUp() {
      this.send({
        event: 'leave',
      });
      this.handleLeave();
    },
    destroyed() {
      if (this.socket != null) {
        this.socket.close();
      }
      this.handleLeave();
    },
    goBack() {
      if (this.socket != null) {
        this.socket.close();
      }
      this.handleLeave();
    },
  }
}

</script>

```

## Web端

```html
<html>
<head>
    <meta charset="UTF-8">
    <title>Screen!</title>
    <style>
        * {
            margin: 0
        }

        #screen-video {
            width: 100%;
            height: 100%;
            object-fit: fill
        }
    </style>
</head>
<style>

</style>
<body>
<div id="app">
    <video id="screen-video" ref="screen" autoplay></video>
    <div class="preview" v-if="accept_video">
        <div class="preview-wrapper">
            <div class="preview-container">
                <div class="preview-body">
                    <h4>您有视频邀请，是否接受?</h4>
                    <button class="btn-success btn" @click="accept">接受</button>
                    <button class="btn-danger btn" style="margin-right: 70px" @click="reject">拒绝</button>
                </div>
                <div class="confirm" @click="closePreview">×</div>
            </div>
        </div>
    </div>
</div>
<script src="../js/jquery.min.js"></script>
<!-- import Vue before Element -->
<script src="../js/vue.js"></script>
<script>
    let video = document.getElementById('screen-video')


    let peerConn, dc;
    let connectedUser = null;
    let vue = new Vue({
        el: '#app',
        data: {
            socket: null,
            userInfo: "",
            users: '',
            count: 1,
            elementLeft: "",
            elementTop: "",
            element: null,
            keyModifier: null,
            mouseModifier: null,
            buttons: {0: 'left', 1: 'middle', 2: 'right'},
            call_username: '',
            remote_video: '',
            accept_video: false,
            socketUrl: `http://192.168.1.134:12007/screen/1/ADMIN`
        },
        methods: {

            openSocket() {
                if (typeof WebSocket == "undefined") {
                    console.log("您的浏览器不支持WebSocket");
                } else {
                    console.log("您的浏览器支持WebSocket");
                    //实现化WebSocket对象，指定要连接的服务器地址与端口  建立连接
                    //等同于socket = new WebSocket("ws://localhost:8888/xxxx/im/25");
                    //var socketUrl="${request.contextPath}/im/"+$("#userId").val();
                    var socketUrl = this.socketUrl;
                    socketUrl = socketUrl.replace("https", "ws").replace("http", "ws");
                    console.log(socketUrl);
                    if (this.socket != null) {
                        this.socket.close();
                        this.socket = null;
                    }
                    this.socket = new WebSocket(socketUrl);
                    //打开事件
                    this.socket.onopen = function () {
                        console.log("websocket已打开");
                        //socket.send("这是来自客户端的消息" + location.href + new Date());
                    };
                    //获得消息事件
                    this.socket.onmessage = this.webSocketOnMessage;
                    //关闭事件
                    this.socket.onclose = this.webSocketClose;
                    //发生了错误事件
                    this.socket.onerror = this.webSocketOnError;
                }
            },
            webSocketOnError() {//连接建立失败重连
                this.openSocket();
                this.handleLeave();
            },
            webSocketOnMessage(e) { //数据接收
                //数据接收
                console.log(e.data);

                let data = JSON.parse(e.data);
                // 更新用户信息
                if (data.userInfo) {
                    this.userInfo = data.userInfo;
                }
                switch (data.event) {
                    case 'show':
                        this.users = data.allUsers;
                        break;
                    case 'join':
                        break;
                    case 'call':
                        this.handleCall(data)
                        break;
                    case 'accept':
                        break;
                    case 'offer':
                        this.handleOffer(data)
                        break;
                    case 'candidate':
                        this.handleCandidate(data)
                        break;
                    case 'msg':
                        alert(data.message);
                        break;
                    case 'answer':
                        break;
                    case 'leave':
                        this.handleLeave(data);
                        break;
                    default:
                        break;
                }

            },
            webSocketClose(e) {  //关闭
                console.log('断开连接', e);
                this.handleLeave();
            },

            createConnection() {
                let self = this;
                peerConn = new RTCPeerConnection();


                peerConn.onaddstream = e => {
                    this.$refs.screen.srcObject = e.stream;
                };

                peerConn.onicecandidate = event => {
                    setTimeout(() => {
                        if (event.candidate) {
                            this.send({
                                event: 'candidate',
                                candidate: event.candidate,
                            });
                        }
                    });
                };
                peerConn.ondatachannel = (e) => {

                    console.log('data', e)
                    dc = e.channel;
                    e.channel.onopen = (e) => {
                        let element = self.$refs.screen;

                        // Bind Key listeners
                        window.onkeydown = self.handleEvent;
                        window.onkeyup = self.handleEvent;
                        // window.addEventListener("keyup", self.handleEvent);
                        // window.addEventListener("keydown", self.handleEvent);
                        // Bind Mouse listeners
                        //element.addEventListener("click", self.handleEvent);
                        window.oncontextmenu=function(e){
                            e.preventDefault();
                        }
                        window.onmousedown = self.handleEvent;
                        window.onmouseup = self.handleEvent;
                        window.onmousemove = self.handleEvent;
                        window.onmousewheel = self.handleEvent;
                        window.onwheel = self.handleEvent;
                        window.ondblclick = self.handleEvent;

                        // element.addEventListener("dblclick", self.handleEvent);
                        // element.addEventListener("mousemove", self.handleEvent);
                        // element.addEventListener("mouseup", self.handleEvent);
                        // element.addEventListener("mousedown", self.handleEvent);
                        // element.addEventListener("mouseswheel", self.handleEvent);
                        // element.addEventListener("wheel", self.handleEvent);

                        // Init element
                        var rect = element.getBoundingClientRect();
                        this.elementLeft = rect.left;
                        this.elementTop = rect.top;
                        this.element = element;
                        /*                        console.log("open")

                                                window.onkeydown = function (e) {
                                                    // data {keyCode, meta, alt, ctrl, shift}
                                                    let data = {
                                                        keyCode: e.keyCode,
                                                        shift: e.shiftKey,
                                                        meta: e.metaKey,
                                                        control: e.ctrlKey,
                                                        alt: e.altKey
                                                    }
                                                    let type = 'key'
                                                    console.log(data)
                                                    dc.send(JSON.stringify({type, data}))
                                                }

                                                window.onmouseup = function (e) {
                                                    // data {clientX, clientY, screen: {width, height}, video: {width, height}}
                                                    let data = {}
                                                    data.clientX = e.clientX
                                                    data.clientY = e.clientY
                                                    data.video = {
                                                        width: self.$refs.screen.getBoundingClientRect().width,
                                                        height: self.$refs.screen.getBoundingClientRect().height
                                                    }
                                                    console.log(data)
                                                    let type = 'mouse'
                                                    dc.send(JSON.stringify({type, data}))
                                                }
                                                window.onmousewheel = function (event) {
                                                    let data = event.wheelDelta > 0;
                                                    let type = 'wheel'
                                                    dc.send(JSON.stringify({type, data}))
                                                }

                                                self.$refs.screen.ondblclick=(e)=>{
                                                    console.log("dblclick")
                                                    let data = {}
                                                    data.clientX = e.offsetX
                                                    data.clientY = e.offsetY
                                                    data.video = {
                                                        width: self.$refs.screen.getBoundingClientRect().width,
                                                        height: self.$refs.screen.getBoundingClientRect().height
                                                    }
                                                    console.log(data)
                                                    let type = 'dbclick'
                                                    dc.send(JSON.stringify({type, data}))

                                                }
                                        /!*        window.onmousemove = function (e) {
                                                    let data = {}
                                                    data.clientX = e.clientX
                                                    data.clientY = e.clientY
                                                    data.video = {
                                                        width: self.$refs.screen.getBoundingClientRect().width,
                                                        height: self.$refs.screen.getBoundingClientRect().height
                                                    }
                                                    console.log(data)
                                                    let type = 'move'

                                                    var timerB = setTimeout(() => {
                                                        clearTimeout(timerB);
                                                        dc.send(JSON.stringify({type, data}))
                                                    }, 200);

                                                }*!/*/
                    }
                    e.channel.onmessage = (e) => {

                    }
                }
            },
            sendInput(message) {
                console.log(message);

                dc.send(JSON.stringify(message))
            },
            handleEvent(event) {
                event.preventDefault();
                var self = this;
                var type = event.type;
                console.log(event)
                switch (type) {
                    case 'keydown':
                    case 'keyup': {
                        var key = event.key.toLowerCase();
                        if (key === ' ') key = 'space';
                        key = key.replace('arrow', '');
                        self.sendInput({
                            type: type,
                            key: key,
                            modifier: self.keyModifier
                        });
                        if (event.keyCode >= 16 && event.keyCode <= 18) {
                            self.keyModifier = type === 'keydown' ? key : null;
                        }
                        break;
                    }
                    case 'click':
                    case 'dblclick': {
                        var button = self.buttons[event.button] || null;
                        self.sendInput({
                            type: type,
                            button: button
                        });
                        break;
                    }
                    case 'mousedown':
                    case 'mouseup': {
                        var button = self.buttons[event.button] || null;
                        let video = {
                            width: this.$refs.screen.getBoundingClientRect().width,
                            height: this.$refs.screen.getBoundingClientRect().height
                        }
                        self.sendInput({
                            type: type,
                            button: button,
                            video:video,
                            x: event.x - self.elementLeft,
                            y: event.y - self.elementTop
                        });
                        self.mouseModifier = type === 'mousedown' ? button : null;
                        break;
                    }
                    case 'mousemove': {
                        let video = {
                            width: this.$refs.screen.getBoundingClientRect().width,
                            height: this.$refs.screen.getBoundingClientRect().height
                        }
                        self.sendInput({
                            type: type,
                            video:video,
                            x: event.x - self.elementLeft,
                            y: event.y - self.elementTop,
                            modifier: self.mouseModifier
                        });
                        break;
                    }
                    case 'mousewheel':
                    case 'wheel': {
                        self.sendInput({
                            type: type,
                            x: event.deltaX * -1,
                            y: event.deltaY * -1
                        });
                        break;
                    }
                }
                //如果事件是可取消的，则 preventDefault() 方法会取消该事件，这意味着属于该事件的默认操作将不会发生。

                return false;
            },
            handleCall(data) {
                this.accept_video = true;
                connectedUser = data.connectedUser;

            },
            handleCandidate(data) {
                // ClientB 通过 PeerConnection 的 AddIceCandidate 方法保存起来
                peerConn.addIceCandidate(new RTCIceCandidate(data.candidate));
            },
            reject() {
                this.send({
                    event: 'accept',
                    accept: false,
                });
                this.accept_video = false;
            },
            handleOffer(data) {
                this.createConnection();
                peerConn.setRemoteDescription(new RTCSessionDescription(data.offer));
                // Create an answer to an offer
                peerConn.createAnswer(
                    answer => {
                        peerConn.setLocalDescription(answer);
                        this.send({
                            event: 'answer',
                            answer: answer,
                        });
                    },
                    error => {
                        alert('Error when creating an answer');
                    }
                );

            },
            dbClick(e) {
                console.log(e)
            },
            accept() {

                this.send({
                    event: 'accept',
                    accept: true,
                });
                this.accept_video = false;
            },
            send(arg) {
                if (typeof WebSocket == "undefined") {
                    console.log("您的浏览器不支持WebSocket");
                } else {
                    arg.connectedUser = connectedUser;
                    let json = JSON.stringify(arg);
                    console.log(json)
                    this.socket.send(json);
                }
            },
            closePreview() {
                this.accept_video = false;
            },
            hangUp() {
                this.send({
                    event: 'leave',
                });
                this.handleLeave();
            },
            handleLeave() {
                alert('通话已结束');
                connectedUser = null;
                this.remote_video = '';
                peerConn.close();
                peerConn.onicecandidate = null;
                peerConn.onaddstream = null;

            },
        },
        created() {

        },
        mounted() {
            this.openSocket();


        },
        destroyed() {
            if (this.socket != null) {
                this.socket.close() //离开路由之后断开websocket连接
            }
            this.handleLeave();
        },
    })
</script>
</body>
</html>
```

## 信令服务器

```java

/*****
 ****/
@Component
@ServerEndpoint(value = "/signaling/{identify}/{type}")
@Slf4j
public class SignalWebSocketServer {

    /****
     * Map，存储用户会话,key用当前的identify
     */
    private static Map<String, Session> sessionMap = new HashMap<String, Session>();

    /****
     * Map，存储Session的ID和identify的映射关系
     */
    private static Map<String, User> idsMap = new HashMap<String, User>();

    /****
     * 1.建立连接后调用该方法
     *  注意：
     *      当前会话属于长连接类型（有状态），因此不能持久化对象到DB中
     */
    @OnOpen
    public void onOpen(@PathParam("identify") String identify, @PathParam("type") String type, Session session) {
        //根据identify获取会话Session
        Session userSession = sessionMap.get(identify);
        if (userSession != null) {
            //根据SessionID和identify移除可能存在的冗余数据
            idsMap.remove(userSession.getId());
            sessionMap.remove(identify);
        }

        //存储用户会话信息
        sessionMap.put(identify, session);
        //存储会话的id和identify的映射关系
        idsMap.put(session.getId(), new User(identify, UserType.get(type), Status.ON_LINE));
        log.info("用户{}类型{}建立链接成功", identify, type);
        processUpdateOnlineUser(null);
    }


    /****
     * 2.关闭链接
     */
    @OnClose
    public void onClose(Session session) {
        //根据Session的ID从idsMap中取出identify
        User identify = idsMap.get(session.getId());
        //从sessionMap中移除identify对应的会话
        sessionMap.remove(identify.getIdentify());
        //移除idsMap中Session的ID对应的identify信息
        idsMap.remove(session.getId());
        log.info("用户{}关闭链接", identify);
        processUpdateOnlineUser(null);
    }


    /****
     * 3.异常处理
     */
    @OnError
    public void onError(Session session, Throwable throwable) {
        log.info("用户：" + idsMap.get(session.getId()) + ",通信发生了异常," + throwable.getMessage());
    }

    /****
     * 4.接收客户端消息
     */
    @OnMessage
    public void onMessage(String message, Session session) throws IOException {
        User user = idsMap.get(session.getId());

        try {
            MessageInfo messageInfo = JSON.parseObject(message, MessageInfo.class);
            log.info("{}事件 用户{} 接收：{}", messageInfo.getEvent(), user, messageInfo);
            // 进行统一的参数校验
            validParam(messageInfo, session);
            // 事件转化为枚举
            Event event = Event.valueOf(messageInfo.getEvent().toUpperCase());
            event.invalid(idsMap, sessionMap, messageInfo, session);
            log.info("转化为对象：{}", messageInfo);
        } catch (SignalingExpection signalingExpection) {
            MessageInfo messageInfo = new MessageInfo();
            messageInfo.setEvent(Event.MSG.name().toLowerCase());
            messageInfo.setMessage(signalingExpection.getMsg());
            session.getBasicRemote().sendText(JSON.toJSONString(messageInfo));
        } catch (Exception e) {
            MessageInfo messageInfo = new MessageInfo();
            messageInfo.setEvent(Event.MSG.name().toLowerCase());
            messageInfo.setMessage("未知错误");
            session.getBasicRemote().sendText(JSON.toJSONString(messageInfo));
        }
    }

    private void validParam(MessageInfo messageInfo, Session session) {
        if (messageInfo == null) {
            throw new SignalingExpection("消息为空");
        }
        if (messageInfo.getEvent() == null) {
            throw new SignalingExpection("消息类型为空");
        }
    }

    @EventListener
    public void processUpdateOnlineUser(UpdateEvent event) {


        // 客户端
        List<Session> userSessions = idsMap.entrySet().stream().filter(e -> {
            User value = e.getValue();
            return value.getType().eq(UserType.ADMIN);
        }).map(e -> sessionMap.get(e.getValue().getIdentify())).collect(Collectors.toList());

        // 管理端
        List<Session> adminSessions = idsMap.entrySet().stream().filter(e -> {
            User value = e.getValue();
            return value.getType().eq(UserType.MACHINE);
        }).map(e -> sessionMap.get(e.getValue().getIdentify())).collect(Collectors.toList());
        List<User> adminList = idsMap.entrySet().stream().filter(e -> {
            User value = e.getValue();
            return value.getType().eq(UserType.MACHINE);
        }).map(e -> e.getValue()).collect(Collectors.toList());

        List<User> userList = idsMap.entrySet().stream().filter(e -> {
            User value = e.getValue();
            return value.getType().eq(UserType.ADMIN);
        }).filter(e -> {
            User value = e.getValue();
            return value.getStatus().equals(Status.ON_LINE);
        }).map(e -> e.getValue()).collect(Collectors.toList());

        MessageInfo userInfo = new MessageInfo();
        userInfo.setAllUsers(adminList);
        userInfo.setEvent(Event.SHOW.name().toLowerCase());
        String s1 = JSON.toJSONString(userInfo);
        userSessions.stream().forEach(s -> {
            //消息发送
            try {

                s.getBasicRemote().sendText(s1);
            } catch (IOException e) {
                e.printStackTrace();

            }

        });
        MessageInfo adminInfo = new MessageInfo();
        userInfo.setAllUsers(userList);
        userInfo.setEvent(Event.SHOW.name().toLowerCase());
        adminSessions.stream().forEach(s -> {
            //消息发送
            try {
                s.getBasicRemote().sendText(JSON.toJSONString(adminInfo));
            } catch (IOException e) {
                e.printStackTrace();

            }
        });
    }
}


@Slf4j
public enum Event implements SignalActionHandler {

    MSG("通知消息"){
        @Override
        public void valid(Map<String, User> users, Map<String, Session> sessionMap, MessageInfo messageInfo, Session session) throws Exception {

        }

        @Override
        public void handler(Map<String, User> users, Map<String, Session> sessionMap, MessageInfo messageInfo, Session session, Session connectedUserSession, User user, User connectedUser) throws Exception {

        }
    },
    SHOW("信息展示"){
        @Override
        public void valid(Map<String, User> users, Map<String, Session> sessionMap, MessageInfo messageInfo, Session session) throws Exception {

        }

        @Override
        public void handler(Map<String, User> users, Map<String, Session> sessionMap, MessageInfo messageInfo, Session session, Session connectedUserSession, User user, User connectedUser) throws Exception {

        }
    },
    CALL("呼叫") {
        @Override
        public void handler(Map<String, User> users, Map<String, Session> sessionMap, MessageInfo messageInfo, Session session, Session connectedUserSession, User user, User connectedUser) throws Exception {

            // 判断用户是否在线
            if (connectedUser.getStatus() == Status.ON_LINE) {
                log.info(messageInfo.toString());
                send(messageInfo, connectedUserSession);
            } else if (connectedUser.getStatus() == Status.BUSY) {
                // 向呼叫方提示用户正忙
                throw SignalingExpection.of("对方正忙");
            }else {
                throw SignalingExpection.of("程序异常！！！");
            }

        }
    },
    OFFER("offer信息") {
        @Override
        public void handler(Map<String, User> users, Map<String, Session> sessionMap, MessageInfo messageInfo, Session session, Session connectedUserSession, User user, User connectedUser) throws IOException {
            //  更新呼叫方用户状态
            user.setStatus(Status.BUSY);
            // todo 更新所有用户状态
            publisher.publishEvent(new UpdateEvent());
            log.info(messageInfo.toString());
            send(messageInfo, connectedUserSession);
        }
    },
    ACCEPT("接受邀请") {
        @Override
        public void handler(Map<String, User> users, Map<String, Session> sessionMap, MessageInfo messageInfo, Session session, Session connectedUserSession, User user, User connectedUser) throws IOException {

            // 如果接受视频邀请
            if (messageInfo.getAccept()) {
                send(messageInfo, connectedUserSession);
            } else {
                //  拒绝视频邀请
                connectedUser.setStatus(Status.ON_LINE);
                messageInfo.setAccept(false);
                send(messageInfo, connectedUserSession);
            }
        }
    },
    ANSWER("answer信息") {
        @Override
        public void handler(Map<String, User> users, Map<String, Session> sessionMap, MessageInfo messageInfo, Session session, Session connectedUserSession, User user, User connectedUser) throws IOException {
            messageInfo.setConnectedUser(user.getIdentify());
            //  更新被呼叫方用户状态
            user.setStatus(Status.BUSY);
            //  更新所有用户状态
            publisher.publishEvent(new UpdateEvent());
            log.info(messageInfo.toString());
            send(messageInfo, connectedUserSession);
        }
    },
    CANDIDATE("candidate信息") {
        @Override
        public void handler(Map<String, User> users, Map<String, Session> sessionMap, MessageInfo messageInfo, Session session, Session connectedUserSession, User user, User connectedUser) throws IOException {
            send(messageInfo, connectedUserSession);
        }
    }, LEAVE("leave信息") {
        @Override
        public void handler(Map<String, User> users, Map<String, Session> sessionMap, MessageInfo messageInfo, Session session, Session connectedUserSession, User user, User connectedUser) throws IOException {
            messageInfo.setConnectedUser(user.getIdentify());
            user.setStatus(Status.ON_LINE);
            connectedUser.setStatus(Status.ON_LINE);
            //  更新所有用户状态
            publisher.publishEvent(new UpdateEvent());
            log.info(messageInfo.toString());
            send(messageInfo, connectedUserSession);
        }
    };

    /**
     * 描述 发送信息
     *
     * @param messageInfo 消息体
     * @param session     发送方的session
     * @throws IOException
     */
    protected void send(MessageInfo messageInfo, Session session) throws IOException {
        String s = JSON.toJSONString(messageInfo);
        session.getBasicRemote().sendText(s);
    }


    Event(String desc) {
        this.desc = desc;
    }

    private String desc;
    @Getter
    @Setter
    protected SignalServices signalingServices;
    @Getter
    @Setter
    protected ApplicationContext publisher;
    @Component
    public static class SignalingServicesInjector {
        @Autowired
        private SignalServices signalingServices;
        @Autowired
        private ApplicationContext publisher;
        @PostConstruct
        public void postConstruct() {
            for (Event rt : EnumSet.allOf(Event.class)) {
                rt.setSignalingServices(signalingServices);
            }
            for (Event rt : EnumSet.allOf(Event.class)) {
                rt.setPublisher(publisher);
            }
        }
    }

}
/**
 * 信令消息枚举处理器
 */
public interface SignalActionHandler {
    /**
     * 业务处理方法
     * @param users
     * @param sessionMap
     * @param messageInfo
     * @param session
     * @param connectedUserSession
     * @param user
     * @param connectedUser
     * @throws Exception
     */
    public void handler(Map<String, User> users,
                        Map<String, Session> sessionMap,
                        MessageInfo messageInfo,
                        Session session, Session connectedUserSession, User user, User connectedUser) throws Exception;

    /**
     * 参数校验方法
     * @param users
     * @param sessionMap
     * @param messageInfo
     * @param session
     */
    default void valid(Map<String, User> users, Map<String, Session> sessionMap, MessageInfo messageInfo, Session session) throws Exception {
        // 获取呼叫方用户信息
        String id = session.getId();
        User user = users.get(id);
        if (user == null) {
            throw SignalingExpection.of("连接错误,无当前用户信息");
        }

        // 获取被呼叫方的用户信息
        String connectedUserId = messageInfo.getConnectedUser();
        if (StringUtils.isEmpty(connectedUserId)) {
            throw SignalingExpection.of("connectedUserId不存在连接错误");
        }
        Session connectedUserSession = sessionMap.get(connectedUserId);
        if (connectedUserSession == null) {
            throw SignalingExpection.of("对方已离线，请稍后进行呼叫");
        }
        User connectedUser = users.get(connectedUserSession.getId());
        if (ObjectUtils.isEmpty(connectedUser)) {
            throw SignalingExpection.of("对方已离线，请稍后进行呼叫");
        }
        // 设置消息来源方id
        messageInfo.setConnectedUser(user.getIdentify());
        messageInfo.setUserInfo(connectedUser);
        handler(users,sessionMap,messageInfo,session,connectedUserSession,user,connectedUser);

    };

    /**
     * 模板代理方法
     * @param users
     * @param sessionMap
     * @param messageInfo
     * @param session
     * @throws Exception
     */
    default void invalid(Map<String, User> users, Map<String, Session> sessionMap, MessageInfo messageInfo, Session session) throws Exception {
        valid(users,sessionMap,messageInfo,session);
    }
}

```

