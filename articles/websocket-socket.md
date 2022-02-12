## Websocket 与 Socket 的区别

WebSocket protocol 是HTML5一种新的协议，同HTTP一样也是应用层的协议，是建立在TCP之上的，它实现了浏览器与服务器全双工通信。客户端与服务端的握手需要借助HTTP请求完成。

Socket其实并不是一个协议，而是为了方便使用TCP或UDP而抽象出来的一层，是位于应用层和传输控制层之间的一组接口。在设计模式中，Socket其实就是一个门面模式，它把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。

#### 区别

- Socket是传输控制层协议，WebSocket是应用层协议
- Socket是一组接口，WebSocket是一个与HTTP属于同一层次的应用层协议，可与服务端进行全双工通信



欢迎阅读 [聊聊WebSocket与Socket.IO](http://localhost:4000/2021/05/15/websocket-socketio/)

