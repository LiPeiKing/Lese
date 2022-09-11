## Shiro之SessionListener

### 背景

实现一个 **session** 过期，然后系统自动校验，给用户提示后跳转到登录页的功能

### 方案思路

1. 首先知道会话什么时候超时，利用shiro中的session会话监听，每隔一段时间监听。

2. 当监听会话过期时，利用WebSocket推送消息到前台提示"您长时间没有操作，请重新登录！！！"的提示框

   

### 实现步骤

#### 1、创建 配置session会话监听类ShiroSessionListener(这个类后面会继续改造)

```java
package com.ktamr.shiro.web.session;


import com.ktamr.WebSocket.WebSocketServer;
import com.ktamr.shiro.web.filter.LogoutFilter;
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.session.Session;
import org.apache.shiro.session.SessionListener;
import org.apache.shiro.subject.SimplePrincipalCollection;
import org.apache.shiro.subject.Subject;
import org.apache.shiro.subject.support.DefaultSubjectContext;

import javax.servlet.http.HttpSession;
import java.io.IOException;
import java.io.Serializable;


/**
 * 配置session监听
 */
public class ShiroSessionListener implements SessionListener  {

    /**
     * 会话创建时触发
     *
     * @param session
     */
    @Override
    public void onStart(Session session) {

    }

    /**
     * 退出会话时触发
     *
     * @param session
     */
    @Override
    public void onStop(Session session) {

    }

    /**
     * 会话过期时触发
     *
     * @param session
     */

    @Override
    public void onExpiration(Session session) {
       System.out.println("我过期了");
    }

}
```

#### 2、在权限配置加载ShiroConfig类中进行监听的配置

##### 配置session监听的Bean

```java
/**
  * 配置session监听
  *
  * @return
  */
@Bean("sessionListener")
public ShiroSessionListener sessionListener() {
    return new ShiroSessionListener();
}
```

##### 添加监听的配置

```java
@Bean
public DefaultWebSessionManager sessionManager(){
    ...
    // 配置session监听
    Collection<SessionListener> listeners = new ArrayList<>();
    listeners.add(sessionListener());
    defaultSessionManager.setSessionListeners(listeners);
    // 监听时间单位 ms
    defaultSessionManager.setSessionValidationInterval(10 * 1000);
    return defaultSessionManager;
}
```

#### 3、使用WebSocket实现过期后进行推送到前端数据

##### 首先加入maven依赖

```pom
<!--WebSocket的支持-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

##### 开启WebSocket支持

```java
package com.sitech.bds.websocket;

import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;
import org.springframework.stereotype.Component;

import javax.websocket.OnClose;
import javax.websocket.OnError;
import javax.websocket.OnMessage;
import javax.websocket.OnOpen;
import javax.websocket.Session;
import javax.websocket.server.PathParam;
import javax.websocket.server.ServerEndpoint;
import java.io.IOException;
import java.util.concurrent.ConcurrentHashMap;

@ServerEndpoint("/session/{sessionId}")
@Component
@Slf4j
public class SessionWebSocketServer {

    /**
     * 存放每个客户端对应的MyWebSocket对象。
     */
    private static final ConcurrentHashMap<String, SessionWebSocketServer> SESSION_WEB_SOCKET_SERVER_MAP = new ConcurrentHashMap<>();

    /**
     * sessionId
     */
    String sessionId;

    /**
     * session
     */
    private Session session;

    /**
     * 连接建立成功调用的方法
     */
    @OnOpen
    public void onOpen(Session session, @PathParam("sessionId") String sessionId) {
        this.session = session;
        this.sessionId = sessionId;
        SESSION_WEB_SOCKET_SERVER_MAP.remove(sessionId);
        SESSION_WEB_SOCKET_SERVER_MAP.put(sessionId, this);
        log.info("session: {} websocket 已建立!", sessionId);
    }

    /**
     * 连接关闭调用的方法
     */
    @OnClose
    public void onClose() {
        SESSION_WEB_SOCKET_SERVER_MAP.remove(sessionId);
    }

    /**
     * 收到客户端消息后调用的方法
     *
     * @param message 客户端发送过来的消息
     */
    @OnMessage
    public void onMessage(String message, Session session) {
        log.info("用户消息:" + sessionId + ",报文:" + message);
    }

    /**
     * @param session session
     * @param error error
     */
    @OnError
    public void onError(Session session, Throwable error) {
        log.error("用户错误:" + sessionId + ",原因:" + error.getMessage());
        error.printStackTrace();
    }


    /**
     * 发送自定义消息
     */
    public static void sendInfo(String message, @PathParam("sessionId") String sessionId) throws IOException {
        log.info("发送消息到:" + sessionId + "，报文:" + message);
        if (StringUtils.isNotEmpty(sessionId) && SESSION_WEB_SOCKET_SERVER_MAP.containsKey(sessionId)) {
            SESSION_WEB_SOCKET_SERVER_MAP.get(sessionId).sendMessage(message);
        } else {
            log.error("用户" + sessionId + ",不在线！");
        }
    }

    /**
     * 发送消息
     * @param message   message
     * @throws IOException
     */
    public void sendMessage(String message) throws IOException {
        this.session.getBasicRemote().sendText(message);
    }

}
```

#### 4、前台连接websocket

```javascript
$(function(){
    // websocket连接
    sessionSocketConnect();
});

function getSessionId() {
    var c_name = 'COOKIE_BIG_SCREEN';
    if (document.cookie.length > 0) {
        c_start = document.cookie.indexOf(c_name + "=");
        if (c_start != -1) {
            c_start = c_start + c_name.length + 1;
            c_end = document.cookie.indexOf(";", c_start);
            if (c_end == -1) {
                c_end = document.cookie.length;
                return unescape(document.cookie.substring(c_start, c_end));
            }
        } else {
            return null;
        }
    }
}

function sessionSocketConnect() {
    let websocket = null;
    websocket = new WebSocket("ws://" + document.location.host+"/session/" + getSessionId());
    websocket.onopen = function() {
        console.log("websocket已打开");
    };
    websocket.onmessage = function(msg) {
        console.log(msg.data);
    };
    websocket.onclose = function() {console.log(GOBLE_TIP['WS_CON_CLOSE']);};
    websocket.onerror = function() {toastr["error"](GOBLE_TIP['WS_CON_ERROR']);}
}
```

#### 5、进行后台发送前台数据

```java
package com.sitech.bds.listener;

import com.sitech.bds.websocket.SessionWebSocketServer;
import org.apache.shiro.session.Session;
import org.apache.shiro.session.SessionListener;

import java.io.IOException;

public class ShiroSessionListener implements SessionListener {

    @Override
    public void onStart(Session session) {

    }

    @Override
    public void onStop(Session session) {

    }

    @Override
    public void onExpiration(Session session) {
        try {
            SessionWebSocketServer.sendInfo("您长时间没有操作，请重新登录！！！", session.getId().toString());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```



Tips:

本想通过前台JS起定时器调用后台校验Session是否过期的接口，但是尝试多种方式拿不到session，进入到了知识盲区。



后记：

```java
Subject current = SecurityUtils.getSubject();
// 判断session是否失效
Objects.isNull(current.getSession().getAttribute("user"));
```

这种判断session是否过期的前提用户登录认证后设置过

```java
SecurityUtils.getSubject().getSession().setAttribute("user",user);
```

调用后台接口时还须设置该接口不需要认证

```java
chainDefinition.addPathDefinition("/sessionId","anon");
```









