# Shiro之Session管理

本篇是Shiro系列第四篇，Shiro中的过滤器初始化流程和实现原理。Shiro基于URL的权限控制是通过Filter实现的，本篇从我们注入的**ShiroFilterFactoryBean**开始入手，翻看源码追寻Shiro中的过滤器的实现原理。

Session

![img](https://img.guitu18.com/markdown/0054a1f3e86934bbe45475471b609988.png)

SessionManager

![img](https://img.guitu18.com/markdown/21d5ef8cdf2be85454c96db7f40bfd61.png)

我们在配置Shiro时配置了一个DefaultWebSecurityManager，先来看下

![img](https://img.guitu18.com/markdown/6394d95a75c064a81028bb444f99e084.png)

DefaultWebSecurityManager

```java
public DefaultWebSecurityManager() {
    super();
    ((DefaultSubjectDAO) this.subjectDAO).setSessionStorageEvaluator(new DefaultWebSessionStorageEvaluator());
    this.sessionMode = HTTP_SESSION_MODE;
    setSubjectFactory(new DefaultWebSubjectFactory());
    setRememberMeManager(new CookieRememberMeManager());
    setSessionManager(new ServletContainerSessionManager());
}
```

在它的构造方法中注入了一个`ServletContainerSessionManager`

```java
public class ServletContainerSessionManager implements WebSessionManager {
    public Session getSession(SessionKey key) throws SessionException {
        if (!WebUtils.isHttp(key)) {
            String msg = "SessionKey must be an HTTP compatible implementation.";
            throw new IllegalArgumentException(msg);
        }
        HttpServletRequest request = WebUtils.getHttpRequest(key);
        Session session = null;
        HttpSession httpSession = request.getSession(false);
        if (httpSession != null) {
            session = createSession(httpSession, request.getRemoteHost());
        }
        return session;
    }
    
    private String getHost(SessionContext context) {
        String host = context.getHost();
        if (host == null) {
            ServletRequest request = WebUtils.getRequest(context);
            if (request != null) {
                host = request.getRemoteHost();
            }
        }
        return host;
    }

    protected Session createSession(SessionContext sessionContext) throws AuthorizationException {
        if (!WebUtils.isHttp(sessionContext)) {
            String msg = "SessionContext must be an HTTP compatible implementation.";
            throw new IllegalArgumentException(msg);
        }
        HttpServletRequest request = WebUtils.getHttpRequest(sessionContext);
        HttpSession httpSession = request.getSession();
        String host = getHost(sessionContext);
        return createSession(httpSession, host);
    }
    
    protected Session createSession(HttpSession httpSession, String host) {
        return new HttpServletSession(httpSession, host);
    }
}
```

`ServletContainerSessionManager`本身并不管理会话，它最终操作的还是HttpSession，所以只能在Servlet容器中起作用，它不能支持除使用HTTP协议的之外的任何会话。

所以一般我们配置Shiro都会配置一个`DefaultWebSessionManager`，它继承了`DefaultSessionManager`，看看DefaultSessionManager的构造方法：

```java
public DefaultSessionManager() {
    this.deleteInvalidSessions = true;
    this.sessionFactory = new SimpleSessionFactory();
    this.sessionDAO = new MemorySessionDAO();
}
```

这里的sessionDAO初始化了一个`MemorySessionDAO`，它其实就是一个Map，在内存中通过键值对管理Session。

```java
public MemorySessionDAO() {
    this.sessions = new ConcurrentHashMap<Serializable, Session>();
}
```

HttpServletSession

```java
public class HttpServletSession implements Session {
    public HttpServletSession(HttpSession httpSession, String host) {
        if (httpSession == null) {
            String msg = "HttpSession constructor argument cannot be null.";
            throw new IllegalArgumentException(msg);
        }
        if (httpSession instanceof ShiroHttpSession) {
            String msg = "HttpSession constructor argument cannot be an instance of ShiroHttpSession.  This " +
                    "is enforced to prevent circular dependencies and infinite loops.";
            throw new IllegalArgumentException(msg);
        }
        this.httpSession = httpSession;
        if (StringUtils.hasText(host)) {
            setHost(host);
        }
    }
    protected void setHost(String host) {
        setAttribute(HOST_SESSION_KEY, host);
    }
    public void setAttribute(Object key, Object value) throws InvalidSessionException {
        try {
            httpSession.setAttribute(assertString(key), value);
        } catch (Exception e) {
            throw new InvalidSessionException(e);
        }
    }
}
```

Shiro的HttpServletSession只是对javax.servlet.http.HttpSession进行了简单的封装，所以在Web应用中对Session的相关操作最终都是对javax.servlet.http.HttpSession进行的，比如上面代码中的setHost()是将内容以键值对的形式保存在httpSession中。

先来看下这张图：

![img](https://img.guitu18.com/markdown/2244742c38299f67980bf22c951c7d0c.png)

先了解上面这几个类的关系和作用，然后我们想管理Shiro中的一些数据就非常方便了。

**SessionDao**是Session管理的顶层接口，定义了Session的增删改查相关方法。

```java
public interface SessionDAO {
    Serializable create(Session session);
    Session readSession(Serializable sessionId) throws UnknownSessionException;
    void update(Session session) throws UnknownSessionException;
    void delete(Session session);
    Collection<Session> getActiveSessions();
}
```

AbstractSessionDao是一个抽象类，在它的构造方法中定义了JavaUuidSessionIdGenerator作为SessionIdGenerator用于生成SessionId。它虽然实现了create()和readSession()两个方法，但具体的流程调用的是它的两个抽象方法doCreate()和doReadSession()，需要它的子类去干活。

```java
public abstract class AbstractSessionDAO implements SessionDAO {
    private SessionIdGenerator sessionIdGenerator;
    public AbstractSessionDAO() {
        this.sessionIdGenerator = new JavaUuidSessionIdGenerator();
    }
    
    public Serializable create(Session session) {
        Serializable sessionId = doCreate(session);
        verifySessionId(sessionId);
        return sessionId;
    }
    protected abstract Serializable doCreate(Session session);

    public Session readSession(Serializable sessionId) throws UnknownSessionException {
        Session s = doReadSession(sessionId);
        if (s == null) {
            throw new UnknownSessionException("There is no session with id [" + sessionId + "]");
        }
        return s;
    }
    protected abstract Session doReadSession(Serializable sessionId);
}
```

看上面那张类图AbstractSessionDao的子类有三个，查看源码发现CachingSessionDAO是一个抽象类，它并没有实现这两个方法。在它的子类EnterpriseCacheSessionDAO中实现了doCreate()和doReadSession()，但doReadSession()是一个空实现直接返回null。

```java
public EnterpriseCacheSessionDAO() {
        setCacheManager(new AbstractCacheManager() {
            @Override
            protected Cache<Serializable, Session> createCache(String name) throws CacheException {
                return new MapCache<Serializable, Session>(name, new ConcurrentHashMap<Serializable, Session>());
            }
        });
    }
```

EnterpriseCacheSessionDAO依赖于它的父级CachingSessionDAO，在他的构造方法中向父类注入了一个AbstractCacheManager的匿名实现，它是一个基于内存的SessionDao，它所创建的MapCache就是一个Map。

我们在Shiro配置类里通过 sessionManager.setSessionDAO(new EnterpriseCacheSessionDAO()); 来使用它。然后在CachingSessionDAO.getCachedSession() 打个断点测试一下，可以看到cache就是一个ConcurrentHashMap，在内存中以Key-Value的形式保存着JSESSIONID和Session的映射关系。

![img](https://img.guitu18.com/markdown/e1fda9b782144a1193abf3d55c4f114a.png)

再来看AbstractSessionDao的第三个实现MemorySessionDAO，它就是一个基于内存的SessionDao，简单直接，构造方法直接new了一个ConcurrentHashMap。

```java
    public MemorySessionDAO() {
        this.sessions = new ConcurrentHashMap<Serializable, Session>();
    }
```

那么它和EnterpriseCacheSessionDAO有啥区别，其实EnterpriseCacheSessionDAO只是CachingSessionDAO的一个默认实现，在CachingSessionDAO中cacheManager是没有默认值的，在EnterpriseCacheSessionDAO的构造方法将其初始化为一个ConcurrentHashMap。

如果我们直接用EnterpriseCacheSessionDAO其实和MemorySessionDAO其实没有什么区别，都是基于Map的内存型SessionDao。而CachingSessionDAO的目的是为了方便扩展的，用户可以继承CachingSessionDAO并注入自己的Cache实现，比如以Redis缓存Session的RedisCache。

其实在业务上如果需要Redis来管理Session，那么直接继承AbstractSessionDao更好，有Redis支撑中间的Cache就是多余的，这样还可以做分布式或集群环境的Session共享（分布式或集群环境如果中间还有一层Cache那么还要考虑同步问题，所以集群环境千万不要继承EnterpriseCacheSessionDAO来实现Session共享）。

例子：项目集群环境下单点登录频繁掉线的问题

其实就是中间那层Cache造成的，业务代码中RedisSessionDao是继承自EnterpriseCacheSessionDAO的，这样一来那在Redis之上还有一个基于内存的Cache层。此时用户的Session如果发生变更，虽然Redis中的Session是同步的，Cache层没有同步，导致的现象就是用户在一台服务器的Session是有效的，另一台服务器Cache中的Session还是旧的，然后用户就被迫下线了。

最后，关于Shiro中Session管理（持久化/共享）的一些总结：

- 如果只是在单机环境下需要做Session持久化（服务器重启保持Session在线），那么最好继承EnterpriseCacheSessionDAO来增加本地Session缓存以减少I/O开销，否则大量的`doReadSession()`调用会造成I/O甚至网络压力（单机环境下能省一点是一点嘛，集群就没办法省了）；
- 如果是在集群环境下做Session共享，千万不要继承EnterpriseCacheSessionDAO，会产生服务器间的本地Session缓存不同步问题，直接继承AbstractSessionDAO即可；







> 转载：https://www.cnblogs.com/guitu18/p/11770790.html





