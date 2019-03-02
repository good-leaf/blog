---

title: voip网络核心设备SBC
date: 2019-02-23 14:22:53
updated: 2019-02-23 14:22:53
categories: 开源框架
tags: voip

---

### Voip网络核心设备-SBC

SBC（Session Border Controlle）是目前voip网络中的核心设备，中文意思是会话边界控制器。顾名思义，就是在网络边界处（内网和外网）对会话进行管理的设备。我们提到的会话是指SIP Session。
<!--more-->
![](http://s13.sinaimg.cn/large/001p2KAvzy7g3biBp92ec&690)

### 功能

SBC提供拓隐藏，呼叫路由管理，防攻击， NAT穿越， QOS， TDM接入， B2BUA，语音编码处理， SIP注册等。

SBC在VOIP网络环境中核心功能包括：

- SIP消息的规范话。在复杂的环境中，接入的设备终端可能来自不同的厂家，不同的私有协议标准，SBC必须对SIP消息进行规范化处理，确保其他的通信设备可以相互交互。

- 编码转换。

- 对NAT处理。

- 传真和语音检测功能。

- 具有良好的性能。

### 部署

利用开源平台搭建SBC。openSIPS,kamailio,FreeSwtich。

kamailio + rtpproxy。
