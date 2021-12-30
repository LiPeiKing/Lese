# Shiro认证授在项目中的使用逻辑梳理

## 前言

基础介绍见前序附件

## 自定义Realm

```java
public class MyRealm extends AuthorizingRealm {

   @Autowired
   UserService userService;
   @Autowired
   RoleService roleService;
   @Autowired
   ResourceService resourceService;
   @Autowired
   HttpSession session;

   //授权
   @Override
   protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {

      String username = (String) super.getAvailablePrincipal(principals);
      User user = userService.findByUsername(username);
      if(null==user){return null;}
      List<Resource> resources =  resourceService.findRoleResources(user.getRoles(),null, Const.RESOURCE_TYPE_FUNCTION);
      SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
      if(null!=user.getRoles()&&user.getRoles().size()>0){
         for(Role r:user.getRoles()){
            info.addRole(r.getName());
         }
         String permission = "role>*>"+ RoleUtils.getMaxRegion(user.getRoles());
         //System.out.println("----------->"+permission);
         info.addObjectPermission(new RoleWildCardPermission(permission));
      }
      if(null!=resources&&resources.size()>0){
         for(Resource resource :resources){
            info.addStringPermission(resource.getPath());
         }
      }
      return info;
   }
   //认证
   @Override
   protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
      UsernamePasswordToken upt = (UsernamePasswordToken)token;
      String username = upt.getUsername();
      User user = userService.findByUsername(username);
      Boolean isCasLogin = (Boolean) session.getAttribute(Const.THIRD_PARTY_LOGIN_FLAG);
      if(null==user){
         throw new UnknownAccountException();
      }else if(user.getStatus() == 0){
         throw new LockedAccountException();
      }
      SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(upt.getUsername(),user.getPassword(),getName());
      return authenticationInfo;
   }
}
```

自定义

## 认证

```java
public String login(HttpSession session, HttpServletRequest request, User user, Boolean rememberMe, RedirectAttributes redirectAttributes){
        String redirectUrl = getRedirectUrl(request);
        session.setAttribute(Const.THIRD_PARTY_LOGIN_FLAG,false); //第三方可信系统访问标识
        Subject current = SecurityUtils.getSubject();
        if(null!=current&&!current.isAuthenticated()){
            UsernamePasswordToken upt = null;
//                String validateCode = request.getParameter("validateCode");
//                String sessionVC = (String)current.getSession().getAttribute("validate_code");
//                if(null==sessionVC||!sessionVC.equalsIgnoreCase(validateCode)){
//                    redirectAttributes.addFlashAttribute("loginError", "验证码错误");
//                    return "redirect:/login";
//                }
                upt = new UsernamePasswordToken(user.getUsername(),user.getPassword());
                upt.setRememberMe(true);
//                upt.setRememberMe(null!=rememberMe?rememberMe:false);
            try {
                current.login(upt);
                SetSession(upt.getUsername());
            } catch (UnknownAccountException e) {
                redirectAttributes.addFlashAttribute("loginError", "用户名不存在");
                return "redirect:/login";
            } catch (IncorrectCredentialsException e) {
                redirectAttributes.addFlashAttribute("loginError", "用户名或者密码错误");
                return "redirect:/login";
            } catch (ExcessiveAttemptsException e) {
                redirectAttributes.addFlashAttribute("loginError", "密码错误3次，锁定10分钟");
                return "redirect:/login";
            } catch (LockedAccountException e) {
                redirectAttributes.addFlashAttribute("loginError", "账户已锁定");
                return "redirect:/login";
            } catch (Exception e){
                redirectAttributes.addFlashAttribute("loginError", "用户名或者密码错误");
                return "redirect:/login";
            }
            log.info("用户登录，用户名："+user.getUsername());
            System.out.println("系统session过期时间："+session.getServletContext().getSessionTimeout());
            return "redirect:"+redirectUrl;
        }
        return "redirect:"+redirectUrl;
    }
```

只需关注认证主方法`current.login(upt);`

### 如何执行到自定义Realm中doGetAuthenticationInfo方法

#### 1、login方法 看看内部是怎样实现的

![image-20211026162728682](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211026162728682.png)

正如Shiro的官方文档所说, 所有安全操作都会委托给SecurityManager来执行

#### 2、进入securityManager的login方法

![image-20211026162940977](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211026162940977.png)

在这个方法中定义了AuthenticationInfo对象用来接收从Realm传来的认证信息

#### 3、进入authenticate方法

![image-20211026163042269](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211026163042269.png)

继续调用authenticator的authenticate方法, 那么这个authenticator是谁呢?

#### 4、进入authenticator的authenticate方法

![image-20211026163316232](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211026163316232.png)



有两个实现类该进入哪一个？根据包名肯定是抽象类，无法判断debug或者百度一下即可。

![image-20211026163503215](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211026163503215.png)

在它的authenticate方法中又调用了自己的doAuthenticate方法, 但该类中对doAuthenticate并没有具体实现,所以继续往下走, 看看哪个类对它进行了实现

![image-20211026163615710](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211026163615710.png)

这才是之前的那个authenticator的真正实现,它的assertRealmsConfigured方法是判断Realm是否存在,不存在则会抛出IllegalStateException.随后根据Realm的个数来判断执行哪个方法,我只配了一个Realm,故会执doSingleRealmAuthentication,并将realm和token作为参数传入, 当然, 传入的realm实际上就是自定义的MyRealm。

------

**提个问题你怎么知道是自定义的？**

其实我是猜的，继续扒代码   

![image-20211026163936747](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211026163936747.png)

既然有get肯定有set

![image-20211026164043603](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211026164043603.png)

选哪个一个呢？正如Shiro的官方文档所说, 所有安全操作都会委托给SecurityManager来执行

![image-20211026164125537](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211026164125537.png)

![image-20211026164253927](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211026164253927.png)

看到这里基本已经明确，肯定是在ShiroConfig中配置的

![image-20211026164355160](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211026164355160.png)

![image-20211026164433573](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211026164433573.png)

不能说一模一样只能说完全一致

------



#### 5、进入doSingleRealmAuthentication方法

![image-20211026164644819](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211026164644819.png)

执行自Realm的getAuthenticationInfo，但自定义Realm重写的是doGetAuthenticationInfo方法这是为什么？？？？

![image-20211026164946276](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211026164946276.png)

认证逻辑到此结束



## 授权

问题一：根据认证代码走向授权应该也是一样，但为什自定义Realm doGetAuthorizationInfo方法不执行？

猜想：肯定没有代码触发自定义Realm doGetAuthorizationInfo方法不执行，但是该从哪里入手多脸懵逼。。。。。。



### 先看看授权流程

#### 授权流程

![授权流程](https://img-blog.csdnimg.cn/2019060716582154.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpemhpcWlhbmcxMjE3,size_16,color_FFFFFF,t_70)

1、首先调用 Subject.isPermitted或hasRole 接口，其会委托给 SecurityManager，而 SecurityManager 接着会委托给 Authorizer；
2、Authorizer是真正的授权者，如果调用如 isPermitted(“user:view”)，其首先会通过 PermissionResolver 把字符串转换成相应的 Permission 实例；
3、在进行授权之前，其会调用相应的 Realm 获取 Subject 相应的角色和权限用于匹配传入的角色和权限；
4、Authorizer 会判断 Realm 的角色和权限是否和传入的匹配，如果 有多个Realm，会委托给 ModularRealmAuthorizer 进行循环判断， 如果匹配如 isPermitted或hasRole 会返回true，否则返回false表示 授权失败。

#### Permissions

1、规则

资源标识符：操作：对象实例 ID 即对哪个资源的哪个实例可以进行什么操作。其默认支持通配符权限字符串： :表示资源、操作、实例的分割；, 表示操作的分割，* 表示任意资源、操作、实例。
2、多层次管理：
  例如：user:query、user:edit
  冒号是一个特殊字符，它用来分隔权限字符串的下一部件：第一部分是权限被操作的领域，第二部分是被执行的操作。
  多个值：每个部件能够保护多个值。因此，除了授予用户 user:query 和 user:edit 权限外，也可以简单地授予他们一个：user:query,edit
  还可以用 * 号代替所有的值，如：user:* ， 也可以写：*:query，表示 某个用户在所有的领域都有 query 的权限。

实例：
 通常会使用三个部件：域、操作、被付诸实施的实例。如：user:edit:manager
 也可以使用通配符来定义，如：user:edit:*、user:*:*、 user:*:manager
 部分省略通配符：缺少的部件意味着用户可以访问所有与之匹配的值，比如：user:edit 等价于 user:edit :*、 user 等价于 user:*:*
 注意：通配符只能从字符串的结尾处省略部件，也就 是说 user:edit 并不等价于 user:*:edit

### 代码如何执行

#### 1、进入hasRole方法

![image-20211026170611323](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211026170611323.png)

hasPrincipals方法判断是否登录成功不再展开，着重关注hasRole发现又是一个委托	

#### 2、进入securityManager的hasRole方法

![image-20211026170813651](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211026170813651.png)

又是一个委托

#### 3、进入authorizer的hasRole方法

![image-20211026171344014](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211026171344014.png)

存在三个实现因为本身就是AuthorizingSecurityManager类，所以只有两个。分别进入发现有多个Realm，会委托给 ModularRealmAuthorizer，另外一个就是单个Realm时使用的委托。

#### 4、进入AuthorizingRealm的hasRole方法

![image-20211026171729225](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211026171729225.png)

感觉有点熟悉，继续进入

5、进入AuthorizingRealm的getAuthorizationInfo方法

```java
protected AuthorizationInfo getAuthorizationInfo(PrincipalCollection principals) {

    if (principals == null) {
        return null;
    }

    AuthorizationInfo info = null;

    if (log.isTraceEnabled()) {
        log.trace("Retrieving AuthorizationInfo for principals [" + principals + "]");
    }

    Cache<Object, AuthorizationInfo> cache = getAvailableAuthorizationCache();
    if (cache != null) {
        if (log.isTraceEnabled()) {
            log.trace("Attempting to retrieve the AuthorizationInfo from cache.");
        }
        Object key = getAuthorizationCacheKey(principals);
        info = cache.get(key);
        if (log.isTraceEnabled()) {
            if (info == null) {
                log.trace("No AuthorizationInfo found in cache for principals [" + principals + "]");
            } else {
                log.trace("AuthorizationInfo found in cache for principals [" + principals + "]");
            }
        }
    }


    if (info == null) {
        // Call template method if the info was not found in a cache
        info = doGetAuthorizationInfo(principals);
        // If the info is not null and the cache has been created, then cache the authorization info.
        if (info != null && cache != null) {
            if (log.isTraceEnabled()) {
                log.trace("Caching authorization info for principals: [" + principals + "].");
            }
            Object key = getAuthorizationCacheKey(principals);
            cache.put(key, info);
        }
    }

    return info;
}
```

这个方法先从缓存中获取，查不到则通过doGetAuthorizationInfo方法获取，拿到后放入缓存。

![image-20211026172020557](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211026172020557.png)

这个方法即可执行自定义Realm中的doGetAuthorizationInfo

## 中结

到此授权、认证代码如何执行已梳理清楚本以为差不多了。但实际项目中的Permissions如何定义在哪定义的呢？



![image-20211026172531965](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211026172531965.png)

查看到这些代码这是什么东西？？？？？role:update这又是定的什么？？？



## 注解介绍

### @RequiresAuthentication

验证用户是否登录，等同于方法subject.isAuthenticated() 结果为true时。

### @RequiresUser

验证用户是否被记忆，user有两种含义：

一种是成功登录的（subject.isAuthenticated() 结果为true）；

另外一种是被记忆的（subject.isRemembered()结果为true）。

### @RequiresGuest

验证是否是一个guest的请求，与@RequiresUser完全相反。

 换言之，RequiresUser == !RequiresGuest。

此时subject.getPrincipal() 结果为null.

### @RequiresRoles

例如：@RequiresRoles("aRoleName");

 void someMethod();

如果subject中有aRoleName角色才可以访问方法someMethod。如果没有这个权限则会抛出异常[AuthorizationException](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/AuthorizationException.html)。

### @RequiresPermissions

例如： @RequiresPermissions({"file:read", "write:aFile.txt"} )
 void someMethod();

要求subject中必须同时含有file:read和write:aFile.txt的权限才能执行方法someMethod()。否则抛出异常[AuthorizationException](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/AuthorizationException.html)。



看到这里基本上可以理解注解和Subjet的方法是相同方式。为什么先不回答



## Permissions如何定义

### 梳理代码

permission既然是自定义自然离不开自定义Realm，查看代码发现

```java
protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
   String username = (String) super.getAvailablePrincipal(principals);
   User user = userService.findByUsername(username);
   if(null==user){return null;}
   List<Resource> resources =  resourceService.findRoleResources(user.getRoles(),null, Const.RESOURCE_TYPE_FUNCTION);

   if(null!=resources&&resources.size()>0){
      for(Resource resource :resources){
          // 重要代码
         info.addStringPermission(resource.getPath());
          
      }
   }
   return info;
}
```

### 查看数据库

![image-20211026173458930](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211026173458930.png)

因此重新增加resource以及和role 的关系

![image-20211026173550760](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211026173550760.png)

### 调用接口

```java
@RequiresPermissions("role:get")
@RequestMapping("/list")
@ResponseBody
   public Map<String,Object> list(Page page) throws IOException {
   List<Role> datas = new ArrayList<Role>();
   if(null!=page){
      page.setTotalRecords(roleService.findAllNo());
      datas = roleService.findRoles(page);
   }else{
      datas = roleService.findAll();
   }
   MDC.put("operateContent","角色列表查询");
   LOG.info("");
   result.put("flag", true);
   result.put("page", page);
   result.put("datas", datas);
   return result;
}
```

### 结果

![image-20211026173735596](C:\Users\elwin\AppData\Roaming\Typora\typora-user-images\image-20211026173735596.png)

## 再次中结

到这里项目中认证、授权、权限如何配置，代码中如何使用的基本已梳理清楚。回答注解和Subjet的方法为什是相同效果

## 注解分析

注解自然离不开AOP面向切面编程，所以直接百度。狗头保命

## shiro源码

> Shiro的注解应该是依赖于@Interface的方式由Spring以@Annotation的方式获取，描述不准确进入知识盲区被打脸了。

### PermissionAnnotationHandler

```java
public void assertAuthorized(Annotation a) throws AuthorizationException {
    if (!(a instanceof RequiresPermissions)) return;

    RequiresPermissions rpAnnotation = (RequiresPermissions) a;
    String[] perms = getAnnotationValue(a);
    Subject subject = getSubject();

    if (perms.length == 1) {
        subject.checkPermission(perms[0]);
        return;
    }
    if (Logical.AND.equals(rpAnnotation.logical())) {
        getSubject().checkPermissions(perms);
        return;
    }
    if (Logical.OR.equals(rpAnnotation.logical())) {
        // Avoid processing exceptions unnecessarily - "delay" throwing the exception by calling hasRole first
        boolean hasAtLeastOnePermission = false;
        for (String permission : perms) if (getSubject().isPermitted(permission)) hasAtLeastOnePermission = true;
        // Cause the exception if none of the role match, note that the exception message will be a bit misleading
        if (!hasAtLeastOnePermission) getSubject().checkPermission(perms[0]);
        
    }
}
```

可以看出实际上就是执行assertAuthorized方式中的getSubject().checkPermissions(perms);的方式进行权限验证的



















