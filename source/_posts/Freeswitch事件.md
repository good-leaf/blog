---

title: Freeswitch事件
date: 2019-02-23 14:22:53
updated: 2019-02-23 14:22:53
categories: 开源框架
tags: Freeswitch

---

Freeswitch内核之事件类型

事件 说明  
3 Channel events 信道事件。
3.1 Channel states 信道状态。  
3.2 CHANNEL_CALLSTATE 信道呼叫状态事件。  
3.3 CHANNEL_CREATE 创建事件。  
3.4 CHANNEL_DESTROY 销毁事件。  
3.5 CHANNEL_STATE 呼叫状态事件。当一个信道切换通话状态时发送。此事件并不包含任何附加信息。  
3.6 CHANNEL_ANSWER 呼叫应答事件。  
3.7 CHANNEL_HANGUP 挂机事件。
3.8 CHANNEL_HANGUP_COMPLETE 挂机完成事件。  
3.9 CHANNEL_EXECUTE PBX正在执行呼叫事件。  
3.10 CHANNEL_EXECUTE_COMPLETE 执行完成。  
3.11 CHANNEL_BRIDGE 一个呼叫两个端点之间的桥接事件。  
3.12 CHANNEL_UNBRIDGE 停用桥接事件。
3.13 CHANNEL_PROGRESS 进度事件，外呼时对方提醒。或者入呼时提醒。  
3.14 CHANNEL_PROGRESS_MEDIA 媒体进度事件，外呼时对方提醒。或者入呼时提醒。  
3.15 CHANNEL_OUTGOING 创建一个外呼事件。
3.16 CHANNEL_PARK 一个呼叫被挂起(停放)在PBX中。
3.17 CHANNEL_UNPARK 一个呼叫被取消挂起(停放)在PBX中。
3.18 CHANNEL_APPLICATION 信道产生的应用程序就是事件application=event一般用来捕获呼转  
3.19 CHANNEL_HOLD 信道保持，使用uuid_hold或者接收SDP的readonly  
3.20 CHANNEL_UNHOLD 触发后uuid_hold关闭<uuid>或者接收到INVITE SDP= SendRecv的  
3.21 CHANNEL_ORIGINATE 信道发起事件，触发完成发起（或桥）。
3.22 CHANNEL_UUID uuid事件表示唯一的ID通道已经改变。原来的ID将被报告的旧唯一ID。此事件会发生，当您使用参数origination_uuid时发出命令发起/桥。
4 System events
4.1 SHUTDOWN 设置以启动的FreeSWITCH的关机顺序。  
4.2 MODULE_LOAD 模块加载  
4.3 MODULE_UNLOAD 模块卸载  
4.4 RELOADXML 重新加载已经配置的XML  
4.5 NOTIFY 通知
4.6 SEND_MESSAGE 发送信息  
4.7 RECV_MESSAGE 接收信息  
4.8 REQUEST_PARAMS 请求参数  
4.9 CHANNEL_DATA 信道数据 
4.10 GENERAL 总体
4.11 COMMAND 命令
4.12 SESSION_HEARTBEAT session心跳  
4.13 CLIENT_DISCONNECTED 客户端断开  
4.14 SERVER_DISCONNECTED 服务器断开 
4.15 SEND_INFO 发送信息  
4.16 RECV_INFO 接收信息
4.17 CALL_SECURE 保密呼叫 
4.18 NAT nat
4.19 RECORD_START 开始记录  
4.20 RECORD_STOP 停止记录
4.21 PLAYBACK_START 开始播放  
4.22 PLAYBACK_STOP 停止播放  
4.23 CALL_UPDATE 更新呼叫



https://blog.csdn.net/hry2015/article/details/78347467
