# HTTP和WebSocket协议

### 协议基础

仔细去看这两个协议，其实都非常简单，但任何一个事情想做到完美都会慢慢地变得异常复杂，各种细节。这里只会简单地描述两个协议的结构，并不会深入到很深的细节之处，对于理解http已经足够了。

#### HTTP

HTTP的地址格式如下：



```bash
http_URL = "http:" "//" host [ ":" port ] [ abs_path [ "?" query ]]
协议和host不分大小写
```

##### HTTP消息

一个HTTP消息可能是request或者response消息，两种类型的消息都是由开始行（start-line），零个或多个header域，一个表示header域结束的空行（也就是，一个以CRLF为前缀的空行），一个可能为空的消息主体（message-body）。一个合格的HTTP客户端不应该在消息头或者尾添加多余的CRLF，服务端也会忽略这些字符。

header的值不包括任何前导或后续的LWS（线性空白），线性空白可能会出现在域值（filed-value）的第一个非空白字符之前或最后一个非空白字符之后。前导或后续的LWS可能会被移除而不会改变域值的语意。任何出现在filed-content之间的LWS可能会被一个SP（空格）代替。header域的顺序不重要，但建议把常用的header放在前边（协议里这么说的）。

##### Request消息

RFC2616中这样定义HTTP Request 消息：



```csharp
Request = Request-Line
          *(( general-header 
            | request-header（跟本次请求相关的一些header）
            | entity-header ) CRLF)（跟本次请求相关的一些header）
          CRLF
          [ message-body ]  
```

一个HTTP的request消息以一个请求行开始，从第二行开始是header，接下来是一个空行，表示header结束，最后是消息体。

请求行的定义如下：



```dart
//请求行的定义
Request-Line = Method SP Request-URL SP HTTP-Version CRLF

//方法的定义
Method = "OPTIONS" | "GET" | "HEAD"  |"POST" |"PUT" |"DELETE" |"TRACE" |"CONNECT"  | extension-method

//资源地址的定义
Request-URI   ="*" | absoluteURI | abs_path | authotity（CONNECT）
```

Request消息中使用的header可以是general-header或者request-header，request-header（后边会讲解）。其中有一个比较特殊的就是Host，Host会与reuqest Uri一起来作为Request消息的接收者判断请求资源的条件，方法如下：

1. 如果Request-URI是绝对地址（absoluteURI），这时请求里的主机存在于Request-URI里。任何出现在请求里Host头域值应当被忽略。
2. 假如Request-URI不是绝对地址（absoluteURI），并且请求包括一个Host头域，则主机由该Host头域值决定。
3. 假如由规则１或规则２定义的主机是一个无效的主机，则应当以一个400（错误请求）错误消息返回。

##### Response消息

响应消息跟请求消息几乎一模一样，定义如下：



```csharp
   Response      = Status-Line              
                   *(( general-header        
                    | response-header       
                    | entity-header ) CRLF)  
                   CRLF
                   [ message-body ]     
```

可以看到，除了header不使用request-header之外，只有第一行不同，响应消息的第一行是状态行，其中就包含大名鼎鼎的**返回码**。

Status-Line的内容首先是协议的版本号，然后跟着返回码，最后是解释的内容，它们之间各有一个空格分隔，行的末尾以一个回车换行符作为结束。定义如下：



```undefined
   Status-Line = HTTP-Version SP Status-Code SP Reason-Phrase CRLF
```

##### 返回码

返回码是一个3位数，第一位定义的返回码的类别，总共有5个类别，它们是：



```jsx
  - 1xx: Informational - Request received, continuing process

  - 2xx: Success - The action was successfully received,
    understood, and accepted

  - 3xx: Redirection - Further action must be taken in order to
    complete the request

  - 4xx: Client Error - The request contains bad syntax or cannot
    be fulfilled

  - 5xx: Server Error - The server failed to fulfill an apparently
    valid request
```

RFC2616中接着又给出了一系列返回码的扩展，这些都是我们平时会用到的，但是**那些只是示例，HTTP1.1不强制通信各方遵守这些扩展的返回码**，通信各方在返回码的实现上只需要遵守以上边定义的这5种类别的定义，意思就是，返回码的第一位要严格按照文档中所述的来，其他的随便定义。

任何人接收到一个不认识的返回码xyz，都可以把它当做x00来对待。对于不认识的返回码的响应消息，不可以缓存。

##### Header

RFC2616中定义了4种header类型，在通信各方都认可的情况下，请求头可以被扩展的（可信的扩展只能等到协议的版本更新），如果接收者收到了一个不认识的请求头，这个头将会被当做实体头。4种头类型如下：

1. 通用头（General Header Fields）：可用于request，也可用于response的头，但不可作为实体头，只能作为消息的头。

   

   ```ruby
   general-header = Cache-Control            ; Section 14.9
                 | Connection               ; Section 14.10
                 | Date                     ; Section 14.18
                 | Pragma                   ; Section 14.32
                 | Trailer                  ; Section 14.40
                 | Transfer-Encoding        ; Section 14.41
                 | Upgrade                  ; Section 14.42
                 | Via                      ; Section 14.45
                 | Warning                  ; Section 14.46
   ```

2. 请求头（Request Header Fields）：被请求发起端用来改变请求行为的头。

   

   ```ruby
   request-header = Accept                   ; Section 14.1
                  | Accept-Charset           ; Section 14.2
                  | Accept-Encoding          ; Section 14.3
                  | Accept-Language          ; Section 14.4
                  | Authorization            ; Section 14.8
                  | Expect                   ; Section 14.20
                  | From                     ; Section 14.22
                  | Host                     ; Section 14.23
                  | If-Match                 ; Section 14.24
                  | If-Modified-Since        ; Section 14.25
                  | If-None-Match            ; Section 14.26
                  | If-Range                 ; Section 14.27
                  | If-Unmodified-Since      ; Section 14.28
                  | Max-Forwards             ; Section 14.31
                  | Proxy-Authorization      ; Section 14.34
                  | Range                    ; Section 14.35
                  | Referer                  ; Section 14.36
                  | TE                       ; Section 14.39
                  | User-Agent               ; Section 14.43
   ```

3. 响应头（Response Header Fields）：被服务器用来对资源进行进一步的说明。

   

   ```ruby
   response-header = Accept-Ranges           ; Section 14.5
                   | Age                     ; Section 14.6
                   | ETag                    ; Section 14.19
                   | Location                ; Section 14.30
                   | Proxy-Authenticate      ; Section 14.33
                   | Retry-After             ; Section 14.37
                   | Server                  ; Section 14.38
                   | Vary                    ; Section 14.44
                   | WWW-Authenticate        ; Section 14.47
   ```

4. 实体头（Entity Header Fields）：如果消息带有消息体，实体头用来作为元信息；如果没有消息体，就是为了描述请求的资源的信息。

   

   ```ruby
   entity-header  = Allow                    ; Section 14.7
                  | Content-Encoding         ; Section 14.11
                  | Content-Language         ; Section 14.12
                  | Content-Length           ; Section 14.13
                  | Content-Location         ; Section 14.14
                  | Content-MD5              ; Section 14.15
                  | Content-Range            ; Section 14.16
                  | Content-Type             ; Section 14.17
                  | Expires                  ; Section 14.21
                  | Last-Modified            ; Section 14.29
                  | extension-header
   ```

##### 消息体（Message Body）和实体主体（Entity Body）

如果有Transfer-Encoding头，那么消息体解码完了就是实体主体，如果没有Transfer-Encoding头，消息体就是实体主体。



```csharp
   message-body = entity-body
                | <entity-body encoded as per Transfer-Encoding>
```

在request消息中，消息头中含有Content-Length或者Transfer-Encoding，标识会有一个消息体跟在后边。如果请求的方法不应该含有消息体（如OPTION），那么request消息一定不能含有消息体，即使客户端发送过去，服务器也不会读取消息体。

在response消息中，是否存在消息体由请求方法和返回码来共同决定。像1xx，204，304不会带有消息体。

##### 消息体的长度

消息体长度的确定有一下几个规则，它们顺序执行：

1. 所有不应该返回内容的Response消息都不应该带有任何的消息体，消息会在第一个空行就被认为是终止了。
2. 如果消息头含有`Transfer-Encoding`，且它的值不是`identity`，那么消息体的长度会使用`chunked`方式解码来确定，直到连接终止。
3. 如果消息头中有`Content-Length`，那么它就代表了`entity-length`和`transfer-length`。如果同时含有`Transfer-Encoding`，则`entity-length`和`transfer-length`可能不会相等，那么`Content-Length`会被忽略。
4. 如果消息的媒体类型是`multipart/byteranges`，并且`transfer-length`也没有指定，那么传输长度由这个媒体自己定义。通常是收发双发定义好了格式， HTTP1.1客户端请求里如果出现Range头域并且带有多个字节范围（byte-range）指示符，这就意味着客户端能解析`multipart/byteranges`响应。
5. 如果是Response消息，也可以由服务器来断开连接，作为消息体结束。

从消息体中得到实体主体，它的类型由两个header来定义，`Content-Type`和`Content-Encoding`（通常用来做压缩）。如果有实体主体，则必须有`Content-Type`,如果没有，接收方就需要猜测，猜不出来就是用`application/octet-stream`。

##### HTTP连接

HTTP1.1的连接默认使用持续连接（persistent connection），持续连接指的是，有时是客户端会需要在短时间内向服务端请求大量的相关的资源，如果不是持续连接，那么每个资源都要建立一个新的连接，HTTP底层使用的是TCP，那么每次都要使用三次握手建立TCP连接，将造成极大的资源浪费。

持续连接可以带来很多的好处：

1. 使用更少的TCP连接，对通信各方的压力更小。
2. 可以使用管道（pipeline）来传输信息，这样请求方不需要等待结果就可以发送下一条信息，对于单个的TCP的使用更充分。
3. 流量更小
4. 顺序请求的延时更小。
5. 不需要重新建立TCP连接就可以传送error，关闭连接等信息。

HTTP1.1的服务器使用TCP的流量控制来控制HTTP的流量，HTTP1.1的客户端在收到服务器连接中发过来的error信息，就要马上关闭此链接。关于HTTP连接还有很多细节，之后再详述。

1. 的长连接（1）。如果你使用Socket来建立TCP的长连接（2），那么，这个长连接（2）跟我们这里要讨论的WebSocket是一样的，实际上TCP长连接就是WebSocket的基础，但是如果是HTTP的长连接，本质上还是Request/Response消息对，仍然会造成资源的浪费、实时性不强等问题。

![img](https:////upload-images.jianshu.io/upload_images/1966024-90ae6a900de164ca.png?imageMogr2/auto-orient/strip|imageView2/2/w/600/format/webp)

HTTP的长连接模型

#### 协议基础

WebSocket的目的是取代HTTP在双向通信场景下的使用，而且它的实现方式有些也是基于HTTP的（WS的默认端口是`80`和`443`）。现有的网络环境（客户端、服务器、网络中间人、代理等）对HTTP都有很好的支持，所以这样做可以充分利用现有的HTTP的基础设施，有点向下兼容的意味。

简单来讲，WS协议有两部分组成：握手和数据传输。

##### 握手（handshake）

出于兼容性的考虑，WS的握手使用HTTP来实现（此文档中提到未来有可能会使用专用的端口和方法来实现握手），客户端的握手消息就是一个「普通的，带有Upgrade头的，HTTP Request消息」。所以这一个小节到内容大部分都来自于RFC2616，这里只是它的一种应用形式，下面是RFC6455文档中给出的一个客户端握手消息示例：



```cpp
    GET /chat HTTP/1.1            //1
    Host: server.example.com   //2
    Upgrade: websocket            //3
    Connection: Upgrade            //4
    Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==            //5
    Origin: http://example.com            //6
    Sec-WebSocket-Protocol: chat, superchat            //7
    Sec-WebSocket-Version: 13            //8
```

可以看到，前两行跟HTTP的Request的起始行一模一样，而真正在WS的握手过程中起到作用的是下面几个header域。

1. Upgrade：upgrade是HTTP1.1中用于定义转换协议的header域。它表示，如果服务器支持的话，客户端希望使用现有的「网络层」已经建立好的这个「连接（此处是TCP连接）」，切换到另外一个「应用层」（此处是WebSocket）协议。
2. Connection：HTTP1.1中规定Upgrade只能应用在「直接连接」中，所以带有Upgrade头的HTTP1.1消息必须含有Connection头，因为Connection头的意义就是，任何接收到此消息的人（往往是代理服务器）都要在转发此消息之前处理掉Connection中指定的域（不转发Upgrade域）。
    如果客户端和服务器之间是通过代理连接的，那么在发送这个握手消息之前首先要发送CONNECT消息来建立直接连接。
3. Sec-WebSocket-＊：第7行标识了客户端支持的子协议的列表（关于子协议会在下面介绍），第8行标识了客户端支持的WS协议的版本列表，第5行用来发送给服务器使用（服务器会使用此字段组装成另一个key值放在握手返回信息里发送客户端）。
4. Origin：作安全使用，防止跨站攻击，浏览器一般会使用这个来标识原始域。

如果服务器接受了这个请求，可能会发送如下这样的返回信息，这是一个标准的HTTP的Response消息。`101`表示服务器收到了客户端切换协议的请求，并且同意切换到此协议。RFC2616规定只有切换到的协议「比HTTP1.1更好」的时候才能同意切换。



```cpp
    HTTP/1.1 101 Switching Protocols //1
    Upgrade: websocket. //2
    Connection: Upgrade. //3
    Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=  //4
    Sec-WebSocket-Protocol: chat. //5
```

##### WebSocket协议Uri

ws协议默认使用`80`端口，wss协议默认使用`443`端口。



```ruby
      ws-URI = "ws:" "//" host [ ":" port ] path [ "?" query ]
      wss-URI = "wss:" "//" host [ ":" port ] path [ "?" query ]

      host = <host, defined in [RFC3986], Section 3.2.2>
      port = <port, defined in [RFC3986], Section 3.2.3>
      path = <path-abempty, defined in [RFC3986], Section 3.3>
      query = <query, defined in [RFC3986], Section 3.4>
```

##### 在客户端发送握手之前要做的一些小事

在握手之前，客户端首先要先建立连接，一个客户端对于一个相同的目标地址（通常是域名或者IP地址，不是资源地址）同一时刻只能有一个处于CONNECTING状态（就是正在建立连接）的连接。从建立连接到发送握手消息这个过程大致是这样的：

1. 客户端检查输入的Uri是否合法。

2. 客户端判断，如果当前已有指向此目标地址（IP地址）的连接（A）仍处于CONNECTING状态，需要等待这个连接（A）建立成功，或者建立失败之后才能继续建立新的连接。
    **PS**：如果当前连接是处于代理的网络环境中，无法判断IP地址是否相同，则认为每一个Host地址为一个单独的目标地址，同时客户端应当限制同时处于CONNECTING状态的连接数。
    **PPS**：这样可以防止一部分的DDOS攻击。
    **PPPS**：客户端并不限制同时处于「已成功」状态的连接数，但是如果一个客户端「持有大量已成功状态的连接的」，服务器或许会拒绝此客户端请求的新连接。

3. 如果客户端处于一个代理环境中，它首先要请求它的代理来建立一个到达目标地址的TCP连接。
    例如，如果客户端处于代理环境中，它想要连接某目标地址的`80`端口，它可能要收现发送以下消息：

   

   ```undefined
          CONNECT example.com:80 HTTP/1.1
          Host: example.com
   ```

如果客户端没有处于代理环境中，它就要首先建立一个到达目标地址的直接的TCP连接。

1. 如果上一步中的TCP连接建立失败，则此WebSocket连接失败。
2. 如果协议是wss，则在上一步建立的TCP连接之上，使用TSL发送握手信息。如果失败，则此WebSocket连接失败；如果成功，则以后的所有数据都要通过此TSL通道进行发送。

##### 对于客户端握手信息的一些小要求

1. 握手必须是RFC2616中定义的Request消息
2. 此Request消息的方法必须是GET，HTTP版本必须大于1.1 。
    以下是某WS的Uri对应的Request消息：
    [ws://example.com/chat](https://links.jianshu.com/go?to=ws%3A%2F%2Fexample.com%2Fchat)
    GET /chat HTTP/1.1
3. 此Request消息中Request-URI部分（RFC2616中的概念）所定义的资型必须和WS协议的Uri中定义的资源相同。
4. 此Request消息中必须含有Host头域，其内容必须和WS的Uri中定义的相同。
5. 此Request消息必须包含Upgrade头域，其内容必须包含websocket关键字。
6. 此Request消息必须包含Connection头域，其内容必须包含Upgrade指令。
7. 此Request消息必须包含Sec-WebSocket-Key头域，其内容是一个Base64编码的16位随机字符。
8. 如果客户端是浏览器，此Request消息必须包含Origin头域，其内容是参考[RFC6454](https://links.jianshu.com/go?to=https%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc6454)。
9. 此Request消息必须包含Sec-WebSocket-Version头域，在此协议中定义的版本号是13。
10. 此Request消息可能包含Sec-WebSocket-Protocol头域，其意义如上文中所述。
11. 此Request消息可能包含Sec-WebSocket-Extensions头域，客户端和服务器可以使用此header来进行一些功能的扩展。
12. 此Request消息可能包含任何合法的头域。如RFC2616中定义的那些。

##### 在客户端接收到Response握手消息之后要做的一些事情

1. 如果返回的返回码不是`101`，则按照RFC2616进行处理。如果是`101`，进行下一步，开始解析header域，所有header域的值不区分大小写。
2. 判断是否含有Upgrade头，且内容包含websocket。
3. 判断是否含有Connection头，且内容包含Upgrade
4. 判断是否含有Sec-WebSocket-Accept头，其内容在下面介绍。
5. 如果含有Sec-WebSocket-Extensions头，要判断是否之前的Request握手带有此内容，如果没有，则连接失败。
6. 如果含有Sec-WebSocket-Protocol头，要判断是否之前的Request握手带有此协议，如果没有，则连接失败。

##### 服务端的概念

服务端指的是所有参与处理WebSocket消息的基础设施，比如如果某服务器使用Nginx（A）来处理WebSocket，然后把处理后的消息传给响应的服务器（B），那么A和B都是这里要讨论的服务端的范畴。

##### 接受了客户端的连接请求，服务端要做的一些事情

如果请求是HTTPS，则首先要使用TLS进行握手，如果失败，则关闭连接，如果成功，则之后的数据都通过此通道进行发送。

之后服务端可以进行一些客户端验证步骤（包括对客户端header域的验证），如果需要，则按照RFC2616来进行错误码的返回。

如果一切都成功，则返回成功的Response握手消息。

##### 服务端发送的成功的Response握手

此握手消息是一个标准的HTTP Response消息，同时它包含了以下几个部分：

1. 状态行（如上一篇RFC2616中所述）
2. Upgrade头域，内容为websocket
3. Connection头域，内容为Upgrade
4. Sec-WebSocket-Accept头域，其内容的生成步骤：
   1. 首先将Sec-WebSocket-Key的内容加上字符串`258EAFA5-E914-47DA-95CA-C5AB0DC85B11`（一个UUID）。
   2. 将#1中生成的字符串进行SHA1编码。
   3. 将#2中生成的字符串进行Base64编码。
5. Sec-WebSocket-Protocol头域（可选）
6. Sec-WebSocket-Extensions头域（可选）

一旦这个握手发出去，服务端就认为此WebSocket连接已经建立成功，处于OPEN状态。它就可以开始发送数据了。

##### WebSocket的一些扩展

Sec-WebSocket-Version可以被通信双方用来支持更多的协议的扩展，RFC6455中定义的值为`13`，WebSocket的客户端和服务端可能回自定义更多的版本号来支持更多的功能。其使用方法如上文所述。

##### 发送数据

WebSocket中所有发送的数据使用帧的形式发送。客户端发送的数据帧都要经过掩码处理，服务端发送的所有数据帧都不能经过掩码处理。否则对方需要发送关闭帧。

一个帧包含一个帧类型的标识码，一个负载长度，和负载。负载包括扩展内容和应用内容。

##### 帧类型

帧类型是由一个4位长的叫Opcode的值表示，任何WebSocket的通信方收到一个位置的帧类型，都要以连接失败的方式断开此连接。
 RFC6455中定义的帧类型如下所示：

1. Opcode == 0 继续

   表示此帧是一个继续帧，需要拼接在上一个收到的帧之后，来组成一个完整的消息。由于这种解析特性，非控制帧的发送和接收必须是相同的顺序。

2. Opcode == 1 文本帧

3. Opcode == 2 二进制帧

4. Opcode == 3 - 7 未来使用（非控制帧）

5. Opcode == 8 关闭连接（控制帧）
    此帧可能会包含内容，以表示关闭连接的原因。
    通信的某一方发送此帧来关闭WebSocket连接，收到此帧的一方如果之前没有发送此帧，则需要发送一个同样的关闭帧以确认关闭。如果双方同时发送此帧，则双方都需要发送回应的关闭帧。
    理想情况服务端在确认WebSocket连接关闭后，关闭相应的TCP连接，而客户端需要等待服务端关闭此TCP连接，但客户端在某些情况下也可以关闭TCP连接。

6. Opcode == 9 Ping
    类似于心跳，一方收到Ping，应当立即发送Pong作为响应。

7. Opcode == 10 Pong
    如果通信一方并没有发送Ping，但是收到了Pong，并不要求它返回任何信息。Pong帧的内容应当和收到的Ping相同。可能会出现一方收到很多的Ping，但是只需要响应最近的那一次就可以了。

8. Opcode == 11 - 15 未来使用（控制帧）

##### 帧的格式

具体的每一项代表什么意思在这里就不做详细的阐述了。



```ruby
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
```

### 与HTTP比较

同样作为应用层的协议，WebSocket在现代的软件开发中被越来越多的实践，和HTTP有很多相似的地方，这里将它们简单的做一个纯个人、非权威的比较：

#### 相同点

1. 都是基于TCP的应用层协议。
2. 都使用Request/Response模型进行连接的建立。
3. 在连接的建立过程中对错误的处理方式相同，在这个阶段WS可能返回和HTTP相同的返回码。
4. 都可以在网络中传输数据。

#### 不同点

1. WS使用HTTP来建立连接，但是定义了一系列新的header域，这些域在HTTP中并不会使用。
2. WS的连接不能通过中间人来转发，它必须是一个直接连接。
3. WS连接建立之后，通信双方都可以在任何时刻向另一方发送数据。
4. WS连接建立之后，数据的传输使用帧来传递，不再需要Request消息。
5. WS的数据帧有序。

### 关于新的HTTP规范

HTTP 1.1在2014年（15年后）被更新为6个单独的「协议说明」，如果去读这6个文档，又是很大的工作量，庆幸有很多文章总结了它们相对于RFC2616的不同之处，RFC7230中也列举了这些feature，所以只是大致把这个看了看。对于这个系列有影响的可能有这两点：

1. Upgrade头域功能的扩展
    `Upgrade`是HTTP中用来进行协议升级的头域，在扩展的协议内容中，客户端发起的协议转换的方式更多，同时服务器也可以选择不接受客户端的协议升级请求；服务端也可以发起协议升级。
2. Uri的格式
    `#fragment`的引入，现在越来越多的人愿意使用fragment来标识网站中的位置了。

完整的变化列表可以在[这里](https://link.jianshu.com?t=http%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc7230%23appendix-A.2)找到。这次变化可以算是一个补丁版，把原来漏掉的小东西补上，版本号无变化。

### WebSocket和Socket的关系

关于这两者的区别，我写了一篇专门的文章来讨论，可以在[WebSocket和Socket的区别](https://www.jianshu.com/p/59b5594ffbb0)看到，简短的答案也放在这里，如果你不想去看另一篇的话。

- 命名方面，Socket是一个深入人心的概念，WebSocket借用了这一概念；
- 使用方面，完全两个东西。

可以把WebSocket想象成HTTP，想一想HTTP和Socket什么关系，WebSocket和Socket就是什么关系。

### WebSocket中数据帧的格式

本文是此系列的最后一篇了，可能是因为懒，很多想写的东西最终都放弃了。关于WebSocket的帧传输这里就不做详细的讲解了，之前的文章试图把很多细节都展开，但发现那样最后只是变成了规则的罗列，如果有兴趣，可以去[这里](https://link.jianshu.com?t=https%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc6455%23section-5.2)看文档。

### WebSocket的实现：socket.IO

socket.IO是一个基于node的实时通信引擎（FEATURING THE FASTEST AND MOST RELIABLE REAL-TIME ENGINE），第一次看到这个框架时，就想到它的底层应该就是WebSocket吧，实时全双工通信，不就是WebSocket的设计目标吗？于是开始对WebSocket底层的实现进行简单的探索。

![img](https:////upload-images.jianshu.io/upload_images/1966024-feda2ab3e8c2e3f3.png?imageMogr2/auto-orient/strip|imageView2/2/w/496/format/webp)

socket.IO主页

socket.IO的底层引擎是[engine.io](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2Fsocketio%2Fengine.io)，engine.io的实现中使用了HTTP和WebSocket两种方式来构建自己的服务端，如果客户端不支持WebSocket协议，则使用轮询的方式来实现实时传输，如果客户端支持WebSocket协议，则使用另一个模块[ws](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2Fwebsockets%2Fws)，ws是一个优秀的WebSocket的JS实现，在ws的README中他们自称为「可能是node平台上最快的WebSocket实现」。

对于普通产品的开发，我们可能不可避免的要照顾到那些从不支持WebSocket的客户端（一般情况下是浏览器）发出的请求，或者有的服务器中不支持部署WebSocket协议的服务端，所以socket.IO这种妥协的方法不失为一个向下兼容的好办法。



![img](https:////upload-images.jianshu.io/upload_images/1966024-6ab12f0b3ab166d1.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

WebSocket连接无法正常建立

如果可以正常建立连接，则会停掉轮询的方式，使用WebSocket进行接下来的连接。

![img](https:////upload-images.jianshu.io/upload_images/1966024-f5d5cd315c3b4fc7.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

WebSocket连接正常建立

#### ws

socket.IO在JS领域算是大名鼎鼎了，截止到今天[socket.IO在github](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2Fsocketio%2Fsocket.io)上的star数量是27543，而作为socket.IO的核心模块，[ws在github](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2Fwebsockets%2Fws)上的star数量却只有4005。

#### 一件有趣的小事

ws的源码中可以搜到`hixie`这个单词，在上一篇关于WebSocket名字由来的文章中介绍过，这个人是WebSocket的命名者。

### 使用socket.IO实现一个在线直播系统

前段时间当时的老板要求做一个「简单的微信上的文字直播程序」，一接到这个需求，我想到的就是`koa + WebSocket + mongoDB`，不得不说，基于node的服务端开发极大的拉低了开发一个服务的门槛，尤其是ES6之后，感觉JS这个以前看起来很随意的语言变得愈加性感起来了 。语法越来越现代化，性能越来越好，而随着npm上的模块越来越多，开发一个业务系统变得空前的简单。

书归正传，经过分析之后，一个「简单的微信上的文字直播程序」大约有3个功能：

- 鉴权
   发布消息的人需要登录之后才能进行操作。
- 发布消息
   直播的播主进行直播，播主的客户端与服务器建立WebSocket连接，在发送消息的同时也作为一个接收消息的客户端。
- 接收消息
   所有打开直播页面的客户端都与服务器建立一个WebSocket连接，第一次进入时拉取部分历史消息，之后每条播主的消息都会同步推送到客户端。

![img](https:////upload-images.jianshu.io/upload_images/1966024-ee3273da37027560.png?imageMogr2/auto-orient/strip|imageView2/2/w/594/format/webp)

直播系统时序图

由于时间仓促，并没有严格保证代码质量，所以就不在这里贴太多的代码了，在这次实现中并没有直接使用socket.IO，而是使用了一个加壳的版本koa-socket，这个库已经很久没有更新了，它接受的函数并不是generator，所以在操作数据库时遇到了一些问题。其中处理发布消息的函数如下：



```tsx
    controllers/socket.js
    exports.postedHandler = ( ctx, data ) => {
        const createdAt = new Date();
        let newPost = {content: data.content, createdAt: createdAt};
        if (data.image) {
            newPost.image = data.image;
        }
        newPost.date = moment(createdAt).format("HH:mm");
        Posts.insert(newPost);
        if (!app) {
            app = require('../app')
        }
        app.io.broadcast('posts', [newPost] )
    }
    
    app.js
    const IO = require( 'koa-socket' )
    ...
    /**some other code**/
    ...
    const io = new IO()
    io.attach( app )
    io.on('post', socket.postedHandler)
```



