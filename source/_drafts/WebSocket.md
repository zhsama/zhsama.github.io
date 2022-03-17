# WebSocket 简介

`Websocket 定义`  [参考规范 rfc6455](https://link.juejin.cn?target=https%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc6455)

**规范解释** `Websocket` 是一种提供客户端(提供不可靠秘钥)与服务端(校验通过该秘钥)进行**双向通信**的协议。

在没有`websocket`协议之前，要提供客户端与服务端实时**双向推送**消息，就会使用`polling`技术，客户端通过`xhr`或`jsonp` **发送**消息给服务端，并通过事件回调来**接收**服务端消息。

这种技术虽然也能保证双向通信，但是有一个不可避免的问题，就是**性能问题**。客户端不断向服务端发送请求，如果客户端并发数过大，无疑导致服务端压力剧增。因此，`websocket`就是解决这一痛点而诞生的。

这里再延伸一些名词:

- **长轮询** 客户端向服务端发送`xhr`请求,服务端接收并`hold`该请求，直到有新消息`push`到客户端，才会主动断开该连接。然后，客户端处理该`response`后再向服务端发起新的请求。以此类推。

> ```
> HTTP1.1`默认使用长连接，使用长连接的`HTTP`协议，会在响应头中加入下面这行信息: `Connection:keep-alive
> ```

- **短轮询**:

客户端不管是否收到服务端的`response`数据，都会定时想服务端发送请求，查询是否有数据更新。

- **长连接** 指在一个`TCP`连接上可以发送多个数据包，在`TCP`连接保持期间，如果没有数据包发送，则双方就需要发送`心跳包`来维持此连接。

> 连接过程: 建立连接——数据传输——...——(发送心跳包，维持连接)——...——数据传输——关闭连接

- **短连接** 指通信双方有数据交互时，建立一个`TCP`连接，数据发送完成之后，则断开此连接。

> 连接过程: 建立连接——数据传输——断开连接——...——建立连接——数据传输——断开连接

**Tips**

> 这里有一个**误解**，长连接和短连接的概念本质上指的是传输层的`TCP`连接，因为`HTTP1.1`协议以后，连接默认都是长连接。没有短连接说法(`HTTP1.0`默认使用短连接)，网上大多数指的长短连接实质上说的就是`TCP`连接。 `http`使用长连接的好处: 当我们请求一个网页资源的时候，会带有很多`js`、`css`等资源文件，如果使用的时短连接的话，就会打开很多`tcp`连接，如果客户端请求数过大，导致`tcp`连接数量过多，对服务端造成压力也就可想而知了。

- **单工** 数据传输的方向唯一，只能由发送方向接收方的单一固定方向传输数据。
- **半双工** 即通信双方既是接收方也是发送方，不过，在某一时刻只能允许向一个方向传输数据。
- **全双工**: 即通信双方既是接收方也是发送方，两端设备可以同时发送和接收数据。

**Tips**

> **单工**、**半双工**和**全双工** 这三者都是建立在	`TCP`协议(传输层上)的概念，不要与应用层进行混淆。

## 特点

- 支持浏览器/Nodejs环境
- 支持双向通信
- API简单易用
- 支持二进制传输
- 减少传输数据量

## 建立连接过程

`Websocket`复用了`HTTP`的握手通道。指的是，客户端发送`HTTP`请求，并在请求头中带上`Connection: Upgrade` 、`Upgrade: websocket`，服务端识别该header之后，进行协议升级，使用`Websocket`协议进行数据通信。

![在这里插入图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/4/6/169f1656159c8519~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

### 参数说明

- `Request URL` 请求服务端地址
- `Request Method` 请求方式 (支持get/post/option)
- `Status Code` 101 Switching Protocols

[RFC 7231 规范定义](https://link.juejin.cn?target=https%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc7231%23section-6.2.2)

> 规范解释: 当收到101请求状态码时，表明服务端理解并同意客户端请求，更改`Upgrade` header字段。服务端也必须在`response`中，生成对应的`Upgrade`值。

- `Connection` 设置`upgrade` header,通知服务端，该`request`类型需要进行升级为`websocket`。 [upgrade_mechanism 规范](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FWeb%2FHTTP%2FProtocol_upgrade_mechanism)
- `Host` 服务端 hostname
- `Origin` 客户端 hostname:port
- `Sec-WebSocket-Extensions` 客户端向服务端发起请求扩展列表(list)，供服务端选择并在响应中返回
- `Sec-WebSocket-Key` 秘钥的值是通过规范中定义的算法进行计算得出，因此是不安全的，但是可以阻止一些误操作的websocket请求。
- `Sec-WebSocket-Accept` 计算公式:  	 1. 获取客户端请求header的值: `Sec-WebSocket-Key` 2. 使用魔数magic = '258EAFA5-E914-47DA-95CA-C5AB0DC85B11' 3. 通过`SHA1`进行加密计算, sha1(Sec-WebSocket-Key + magic) 4. 将值转换为base64
- `Sec-WebSocket-Protocol`  指定有限使用的Websocket协议，可以是一个协议列表(list)。服务端在`response`中返回列表中支持的第一个值。
- `Sec-WebSocket-Version`  指定通信时使用的Websocket协议版本。最新版本:13,[历史版本](https://link.juejin.cn?target=https%3A%2F%2Fwww.iana.org%2Fassignments%2Fwebsocket%2Fwebsocket.xml%23version-number)
- `Upgrade` 通知服务端，指定升级协议类型为`websocket`

### 数据帧格式

数据格式定义参考：[规范 RFC6455](https://link.juejin.cn?target=https%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc6455%23section-5.2)

```
  0                   1                   2                   3
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
 +-+-+-+-+-------+-+-------------+-------------------------------+
 |F|R|R|R| opcode|M| Payload len |    Extended payload length    |
 |I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |
 |N|V|V|V|       |S|             |   (if payload len==126/127)   |
 | |1|2|3|       |K|             |                               |
 +-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
 |     Extended payload length continued, if payload len == 127  |
 + - - - - - - - - - - - - - - - +-------------------------------+
 |                               |Masking-key, if MASK set to 1  |
 +-------------------------------+-------------------------------+
 | Masking-key (continued)       |          Payload Data         |
 +-------------------------------- - - - - - - - - - - - - - - - +
 :                     Payload Data continued ...                :
 + - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +
 |                     Payload Data continued ...                |
 +---------------------------------------------------------------+
复制代码
```

- `FIN`: 1 bit 如果该位值为1，表示这是`message`的最终片段(`fragment`)，如果为0，表示这是一个`message`的第一个片段。
- `RSV1, RSV2, RSV3`: 各占1 bit 一般默认值是0，除非协商扩展，为非零值进行定义，否则收到非零值，并且没有进行协商扩展定义，则`websocket`连接失败。
- `Opcode`: 4 bits 根据操作码(`Opcode`),解析有效载荷数据(`Payload data`).如果接受到未定义操作码，则应该断开`websocket`连接。
- `Mask`: 1 bit 定义是否需要的载荷数据(``Payload data)，进行掩码操作。如果设置值为1，那么在`Masking-key`中会定义一个掩码key,并用这个key对载荷数据进行反掩码(`unmask`)操作。所有从客户端发送到服务端的数据帧(`frame`),mask都被设置为1.
- `Payload length`: 7 bits, 7+16 bits, or 7+64 bits 载荷数据的长度。
- `Masking-key`: 0 or 4 bytes 所有从客户端传送到服务端的数据帧，数据载荷都进行了掩码操作，Mask为1，且携带了4字节的Masking-key。如果Mask为0，则没有Masking-key。
- `Payload data`:  (x+y) bytes

## 心跳检测

为了确保客户端与服务端的长连接正常，有时即使客户端连接中断，但是服务端未触发`onclose`事件，这就有可能导致无效连接占用。所以需要一种机制，确保两端的连接处于正常状态，**心跳检测**就是这种机制。客户端每隔一段时间，会向服务端发送**心跳**(数据包)，服务端也会返回`response`进行反馈连接正常。

# WebSocket 优点

比起传统的轮询方式，WebSocket 可以更好的节省服务器资源和带宽，并且能够进行更加实时地通讯，优势如下：

- **减小带宽开销**。服务器和客户端在连接建立后，相比起 HTTP 请求，交换数据时用于协议控制的数据包头部相对较小，一般只有 2 字节；
- **增强实时性**。服务器可以随时主动给客户端下发数据，相对于 HTTP 请求需要等待客户端发起请求服务端才能响应，延迟明显更少，和传统的轮询比较，WebSocket 也可以在短时间内更有效率地传递数据；
- **维持连接状态**。在一些需要身份认证的场景下， HTTP 请求可能需要在每个请求都携带状态信息（服务器不记录每次的请求和响应信息），而 WebSocket 一次连接建立后就会保持住会话状态，这就使其成为一种有状态的协议，后续通信时就可以省略部分状态信息；
- **更灵活的扩展支持**。根据 RFC6455 协议[2]，开发者可以对 WebSocket 自定义二进制帧，相对 HTTP，可以更轻松地处理二进制内容，此外开发者也自行扩展协议、实现部分自定义的子协议。
- **更好的压缩效果**。WebSocket 在适当的扩展支持下，可以沿用之前内容的上下文，在传递类似的数据时，可以显著地提高压缩率。

![image](https://user-images.githubusercontent.com/33454514/158771990-2812e406-c716-434f-ad24-07e0764550b7.png)

![image](https://user-images.githubusercontent.com/33454514/158772037-7758c2d6-5751-4708-a02d-6d0df60062ef.png)

# WebSocket 使用场景

WebSocket 有这么多优势，那么它适用于哪些场景呢？下面来简单了解下：

- **需要及时响应的场景**。当客户端需要对服务端发生的改变做出快速响应（尤其是客户端无法预测的响应）时，WebSocket 是非常适合的。例如开发一个客服系统，这往往要求实现多个用户实时沟通。如果使用 WebSocket，则每个对话都可以实时发送和接收消息。与 HTTP 相比，WebSocket 不需要考虑发送和接收的每个消息的 HTTP 请求/响应导致的开销，从而会有更高的执行效率。
- **需要实时查询的场景**。例如一名篮球迷想要查询比赛结果，如果比赛是上周结束的，那么比赛结果是固定的，HTTP 在这种情况下就非常适合。但是，如果是当前正在进行的比赛，得分会不断变化，并且更新频繁，在这种情况下，WebSocket 就是更好的选择。
- **小负载的高频消息传递**。如今越来越多的开发人员正在通过移动设备的 GPS 功能来记录 Web 应用程序的方位感知。如果我们需要记录一段时间内用户的位置信息，高频率发送更加细粒度的位置数据，从而起到实时分享功能（例如运动类 APP），WebSocket 所使用的 TCP 连接会让数据交换飞起来。
- **多人协同的场景**。例如近几年发展迅速的在线教育，学生可以足不出户，即可与老师以及其他同学一起进行实时沟通与交流，诸如布置作业、师生互动、问题讨论等等强实时交互类的场景都可交由 WebSocket 协议支撑完成，从而满足低延迟，高及时的场景要求。

# WebSocket API

## [常量](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket#常量)

| **Constant**           | **Value** |
| ---------------------- | --------- |
| `WebSocket.CONNECTING` | `0`       |
| `WebSocket.OPEN`       | `1`       |
| `WebSocket.CLOSING`    | `2`       |
| `WebSocket.CLOSED`     | `3`       |

## [属性](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket#属性)

- [`WebSocket.binaryType`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/binaryType)

  使用二进制的数据类型连接。

- [`WebSocket.bufferedAmount`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/bufferedAmount) 只读

  未发送至服务器的字节数。

- [`WebSocket.extensions`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/extensions) 只读

  服务器选择的扩展。

- [`WebSocket.onclose`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/close_event)

  用于指定连接关闭后的回调函数。

- [`WebSocket.onerror`](https://developer.mozilla.org/zh-CN/docs/conflicting/Web/API/WebSocket/error_event)

  用于指定连接失败后的回调函数。

- [`WebSocket.onmessage`](https://developer.mozilla.org/zh-CN/docs/conflicting/Web/API/WebSocket/message_event)

  用于指定当从服务器接受到信息时的回调函数。

- [`WebSocket.onopen`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/open_event)

  用于指定连接成功后的回调函数。

- [`WebSocket.protocol`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/protocol) 只读

  服务器选择的下属协议。

- [`WebSocket.readyState`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/readyState) 只读

  当前的链接状态。

- [`WebSocket.url`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/url) 只读

  WebSocket 的绝对路径。

## [方法](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket#method_overview)

- [`WebSocket.close([code[, reason\]])`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/close)

  关闭当前链接。

- [`WebSocket.send(data)`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/send)

  对要传输的数据进行排队。

## [事件](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket#事件)

使用 `addEventListener()` 或将一个事件监听器赋值给本接口的 `on*eventname*` 属性，来监听下面的事件。

- [`close`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/close_event)

  当一个 `WebSocket` 连接被关闭时触发。 也可以通过 [`onclose`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/close_event) 属性来设置。

- [`error`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/error_event)

  当一个 `WebSocket` 连接因错误而关闭时触发，例如无法发送数据时。 也可以通过 [`onerror`](https://developer.mozilla.org/zh-CN/docs/conflicting/Web/API/WebSocket/error_event) 属性来设置.

- [`message`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/message_event)

  当通过 `WebSocket` 收到数据时触发。 也可以通过 [`onmessage`](https://developer.mozilla.org/zh-CN/docs/conflicting/Web/API/WebSocket/message_event) 属性来设置。

- [`open`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/open_event)

  当一个 `WebSocket` 连接成功时触发。 也可以通过 [`onopen`](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket/open_event) 属性来设置。

## WebSocket API 兼容性

![在这里插入图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/3/16ecac45c6f6fec3~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

# 代码示例

## Client

```js
const net = require('net');

const heartbeat = 'HEARTBEAT';
const client = new net.Socket();
client.connect(9000, 'localhost', () => {});
client.on('data', (chunk) => {
  const content = chunk.toString();
  if (content === heartbeat) {
    console.log('收到心跳包：', content);
  } else {
    console.log('收到数据：', content);
  }
});

setInterval(() => { // 定时发送数据
  console.log('发送数据', new Date().toUTCString());
  client.write(new Date().toUTCString());
}, 5000);

setInterval(function() { // 定时发送心跳包
  client.write(heartbeat);
}, 10000);
```

## Server

```js
const net = require('net');

const clientList = [];
const heartBeat = 'HEART';

const server = net.createServer();
server.on('connect', (client) => {
  console.log('client connect:', client.remoteAddress + ':' + client.remotePort);
  clientList.push(client);
  client.on('data', (pack) => {
    const content = pack.toString();
    if (content === heartBeat) {
      console.log('get heartBeat from client');
    } else {
      console.log('get data from client', content);
      client.write('server:' + content);
    }
  });
  client.on('end', () => {
    console.log('client end');
    clientList.splice(clientList.indexOf(client), 1);
  });
  client.on('error', () => {
    const index = clientList.indexOf(client);
    console.log('client' + index + 'error');
    clientList.splice(index, 1);
  });
server.listen(9000);

function broadcast() {
  console.log('broadcast heartbeat', clientList.length);
  const cleanup = [];
  for (let i = 0; i < clientList.length; i += 1) {
    if (clientList[i].writable) { // 先检查 sockets 是否可写
      clientList[i].write(heartBeat);
    } else {
      console.log('一个无效的客户端');
      cleanup.push(clientList[i]); // 如果不可写，收集起来销毁。销毁之前要 Socket.destroy() 用 API 的方法销毁。
      clientList[i].destroy();
    }
  }
 
  // Remove dead Nodes out of write loop to avoid trashing loop index
  for (let i = 0; i < cleanup.length; i += 1) {
    console.log('删除无效的客户端:', cleanup[i].name);
    clientList.splice(clientList.indexOf(cleanup[i]), 1);
  }
}

setInterval(broadcast, 10000); // 定时发送心跳包
```

