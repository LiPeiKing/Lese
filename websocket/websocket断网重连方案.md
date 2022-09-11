# websocket断网重连方案

## 背景

由于网络问题websocket连接中断导致无法请求接口，应客户需要网络恢复后不需要其他操作可继续请求接口

## 实现思路

### 一、监听网络恢复

当监听到网络恢复重建websocket连接，使用offline.min.js进行监听。实现后发现该组件监听的是设备网络的连接而不是网络可通信。（也可能是使用不当）

### 二、心跳重连

websocket是前后端交互的长连接，前后端也都可能因为一些情况导致连接失效并且相互之间没有反馈提醒。因此为了保证连接的可持续性和稳定性，websocket心跳重连就应运而生。

在使用原生websocket的时候，如果设备网络断开，不会立刻触发websocket的任何事件，前端也就无法得知当前连接是否已经断开。这个时候如果调用websocket.send方法，浏览器才会发现链接断开了，便会立刻或者一定短时间后（不同浏览器或者浏览器版本可能表现不同）触发onclose函数。

后端websocket服务也可能出现异常，造成连接断开，这时前端也并没有收到断开通知，因此需要前端定时发送心跳消息ping，后端收到ping类型的消息，立马返回pong消息，告知前端连接正常。如果一定时间没收到pong消息，就说明连接不正常，前端便会执行重连。

为了解决以上两个问题，以前端作为主动方，定时发送ping消息，用于检测网络和前后端连接问题。一旦发现异常，前端持续执行重连逻辑，直到重连成功。

## 实现

### 前端

```javascript
GOBLE_OBJ = {
    WEB_SOCKET:null
    // websocket重建锁，避免重复连接
    LOCK_RECONNECT: false,

};


/**
 *  实时消息连接建立
 * @returns socket
 */
function socketConnect(){
    let websocket = null;
    if(typeof(WebSocket) == "undefined") {
        console.log('您的浏览器不支持WebSocket');
    }else{
        websocket = new WebSocket("ws://"+document.location.host+"/ws/"+Math.random());
        websocket.onopen = function() {
            websocket.send(JSON.stringify(getWsParam()));
            //心跳检测重置
            heartCheck.start();
        };
        websocket.onmessage = function(msg) {
            if (msg.data === "WebsocketLive") {
                console.log("websocket心跳检测通过");
                heartCheck.start();
                return;
            }
            if(msg.data){
                // do other
            }else{
                console.log("real time data push result is null.");
            }
        };
        websocket.onclose = function() {
            if(GOBLE_OBJ.WEB_SOCKET != null) {
                console.log('WS_CON_CLOSE');
                GOBLE_OBJ.WEB_SOCKET = null;
                reconnect();
            }
        };
        websocket.onerror = function() {
            if(GOBLE_OBJ.WEB_SOCKET != null) {
                toastr["error"](GOBLE_TIP['WS_CON_ERROR']);
                GOBLE_OBJ.WEB_SOCKET = null;
                reconnect();
            }
         }
    }
    return websocket;
}


function reconnect() {
    if(GOBLE_OBJ.LOCK_RECONNECT) {
        return;
    }
    lockReconnect = true;
    //没连接上会一直重连，设置延迟避免请求过多
    setTimeout(function () {
        //建立socket链接
        console.log('重建socket连接');
        GOBLE_OBJ.WEB_SOCKET = socketConnect();
        GOBLE_OBJ.LOCK_RECONNECT = false;
    }, 30 * 1000);
}

//心跳检测
var heartCheck = {
    timeout: 60 * 1000,//60秒
    timeoutObj: null,
    serverTimeoutObj: null,
    start: function() {
        var self = this;
        this.timeoutObj && clearTimeout(this.timeoutObj);
        this.serverTimeoutObj && clearTimeout(this.serverTimeoutObj);
        this.timeoutObj = setTimeout(function() {
            //这里发送一个心跳，后端收到后，返回一个心跳消息，
            //onmessage拿到返回的心跳就说明连接正常
            console.log('开始心跳检测');
            GOBLE_OBJ.WEB_SOCKET.send("HeartCheck");
            self.serverTimeoutObj = setTimeout(function() {
                //如果超过一定时间还没重置，说明后端主动断开了
                //如果onclose会执行reconnect，我们执行GOBLE_OBJ.WEB_SOCKET.close()就行了.如果直接执行reconnect 会触发onclose导致重连两次
                GOBLE_OBJ.WEB_SOCKET.close();
            }, self.timeout)
        }, this.timeout)
    }
}



```

### 后端

```java
@ServerEndpoint("/ws/{userId}")
@Component
@Slf4j
public class WebSocketServer {

    @OnOpen
    public void onOpen(Session session, @PathParam("userId") String userId) {
		...
    }


    @OnClose
    public void onClose(Session session) {
		...
    }

    @OnMessage
    public void onMessage(String message, Session session) {
        log.debug("用户消息:("+userId+"),报文:"+message);
        if(!StringUtils.isEmpty(message)){
            if (message.equals("HeartCheck")) {
                // 心跳检测不进行业务数据查询
                try {
                    sendMessage("WebsocketLive");
                } catch (IOException e) {
                    e.printStackTrace();
                    log.error("用户: {},websocket心跳检测异常!!!!!!", userId);
                }
                return;
            }
            ...
        }
    }

    /**
     *
     * @param session
     * @param error
     */
    @OnError
    public void onError(Session session, Throwable error) {
        log.error("用户错误:"+this.userId+",原因:"+error.getMessage());
        error.printStackTrace();
    }

    public void sendMessage(String message) throws IOException {
        this.session.getBasicRemote().sendText(message);
    }
}
```























