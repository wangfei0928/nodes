# Shiro

## 1. Shiro是什么？

Shiro是Apache的一个Java安全（权限框架），Shiro可以完成认证、授权、加密、会话管理、与 Web 集成、缓存等。

## 2. Shiro和SpringSecurity对比

1. Spring Security 基于 Spring 开发，项目若使用 Spring 作为基础，配合 Spring  Security 做权限更加方便，而 Shiro 需要和 Spring 进行整合开发；
2.  Spring Security 功能比 Shiro 更加丰富些，例如安全维护方面； 
3. Spring Security 社区资源相对比 Shiro 更加丰富； 
4. Shiro 的配置和使用比较简单，Spring Security 上手复杂些； 
5. Shiro 依赖性低，不需要任何框架和容器，可以独立运行.Spring Security 依赖 Spring 容器;
6. Shiro 不仅仅可以使用在 web 中，它可以工作在任何应用环境中。在集群会话时 Shiro 最重要的一个好处或许就是它的会话是独立于容器的。

## 3. 基本功能点

![](./pic/QQ截图20230705155118.png)

功能简介  

（1）Authentication：身份认证/登录，验证用户是不是拥有相应的身份； 

（2）Authorization：授权，即权限验证，验证某个已认证的用户是否拥有某个权限；即 判断用 户是否能进行什么操作，如：验证某个用户是否拥有某个角色。或者细粒度的验证 某个用户 对某个资源是否具有某个权限； 

（3）Session Manager：会话管理，即用户登录后就是一次会话，在没有退出之前，它的 所有 信息都在会话中；会话可以是普通 JavaSE 环境，也可以是 Web 环境的； 

（4）Cryptography：加密，保护数据的安全性，如密码加密存储到数据库，而不是明文存 储； 

（5）Web Support：Web 支持，可以非常容易的集成到 Web 环境； 

（6）Caching：缓存，比如用户登录后，其用户信息、拥有的角色/权限不必每次去查，这 样可 以提高效率；

   (7）Concurrency：Shiro 支持多线程应用的并发验证，即如在一个线程中开启另一个线 程，能把权限自动传播过去； 

（8）Testing：提供测试支持； 

（9）Run As：允许一个用户假装为另一个用户（如果他们允许）的身份进行访问；

（10）Remember Me：记住我，这个是非常常见的功能，即一次登录后，下次再来的话不用 登 录了

## 4. Shiro 架构

（1）Subject：应用代码直接交互的对象是 Subject，也就是说 Shiro 的对外 API 核心 就是 Subject。Subject 代表了当前“用户”， 这个用户不一定 是一个具体的人，与当 前应用交互的任何东西都是 Subject，如网络爬虫， 机器人等；与 Subject 的所有交互 都会委托给 SecurityManager； Subject 其实是一个门面，SecurityManager 才是实际的 执行者； 

（2）SecurityManager：安全管理器；即所有与安全有关的操作都会与 SecurityManager  交互；且其管理着所有 Subject；可以看出它是 Shiro 的核心，它负责与 Shiro 的其他 组件进行交互，它相当于 SpringMVC 中 DispatcherServlet 的角色 

（3）Realm：Shiro 从 Realm 获取安全数据（如用户、角色、权限），就是说 SecurityManager 要验证用户身份，那么它需要从 Realm 获取相应的用户 进行比较以确 定用户身份是否合法；也需要从 Realm 得到用户相应的角色/ 权限进行验证用户是否能进 行操作；可以把 Realm 看成 DataSource

## 5. 登录认证基本流程

（1）收集用户身份/凭证，即如用户名/密码 

（2）调用 Subject.login 进行登录，如果失败将得到相应 的 AuthenticationException  异常，根据异常提示用户 错误信息；否则登录成功 

（3）创建自定义的 Realm 类，继承 org.apache.shiro.realm.AuthenticatingRealm类， 实现 doGetAuthenticationInfo() 方法

代码示例：

pom.xml

```xml
	<dependencies>
        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.1.3</version>
        </dependency>

        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-core</artifactId>
            <version>1.3.2</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
        </dependency>
    </dependencies>
```

resources资源中添加shiro.ini

```ini
[users]
jay=123
```

```java
public class HelloShiro {
    @Test
    public void shiroLogin(){
        //导入INI配置创建工厂
        Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
        //工厂构建安全管理器
        SecurityManager instance = factory.getInstance();
        //使用工具生效安全管理器
        SecurityUtils.setSecurityManager(instance);
        //使用工具获得subject主体
        Subject subject = SecurityUtils.getSubject();
        //构建账户密码
        UsernamePasswordToken usernamePasswordToken = new UsernamePasswordToken("jay", "123");
        //使用subject主体去登陆
        subject.login(usernamePasswordToken);
        //打印登录信息
        System.out.println("登录结果："+subject.isAuthenticated());
    }
}
```

![](./pic/QQ截图20230705163016.png)

## 6. 身份认证流程

（1）首先调用 Subject.login(token) 进行登录，其会自动委托给 SecurityManager  

（2）SecurityManager 负责真正的身份验证逻辑；它会委托给 Authenticator 进行身份验证； 

（3）Authenticator 才是真正的身份验证者，Shiro API 中核心的身份 认证入口点，此 处可以自定义插入自己的实现； 

（4）Authenticator 可能会委托给相应的 AuthenticationStrategy 进行多 Realm 身份 验证，默认ModularRealmAuthenticator 会调用 AuthenticationStrategy 进行多 Realm  身份验证； 

（5） Authenticator 会把相应的 token 传入 Realm，从 Realm 获取身份验证信息，如果没有返回/抛出异常表示身份验证失败了。此处可以配置多个Realm，将按照相应的顺序及策略进行访问。

## 7. 角色、授权

（1）授权，也叫访问控制，即在应用中控制谁访问哪些资源（如访问页面/编辑数据/页面 操作等）。在授权中需了解的几个关键对象：主体（Subject）、资源（Resource）、权限（Permission）、角色（Role）。 

（2）主体(Subject)：访问应用的用户，在 Shiro 中使用 Subject 代表该用户。用户只 有授权 后才允许访问相应的资源。 

（3）资源(Resource)：在应用中用户可以访问的 URL，比如访问 JSP 页面、查看/编辑 某些 数据、访问某个业务方法、打印文本等等都是资源。用户只要授权后才能访问。 

（4）权限(Permission)：安全策略中的原子授权单位，通过权限我们可以表示在应用中 用户 有没有操作某个资源的权力。即权限表示在应用中用户能不能访问某个资源，如：访 问用 户列表页面查看/新增/修改/删除用户数据（即很多时候都是CRUD（增查改删）式权 限控 制）等。权限代表了用户有没有操作某个资源的权利，即反映在某个资源上的操作允 不允 许。 

（5）Shiro 支持粗粒度权限（如用户模块的所有权限）和细粒度权限（操作某个用户的权 限， 即实例级别的） 

（6）角色(Role)：权限的集合，一般情况下会赋予用户角色而不是权限，即这样用户可 以拥有 一组权限，赋予权限时比较方便。典型的如：项目经理、技术总监、CTO、开发工 程师等 都是角色，不同的角色拥有一组不同的权限。





授权的方式有: (1)编程式，通过写if/else代码授权代码块，(2)注解式: `@RequiresRoles("admin")`  (3) JSP标签 `<shiro: hasRole name = "admin">`

## 8. 授权流程

（1）首先调用Subject.isPermitted*/hasRole*接口，其会委托给SecurityManager，而 SecurityManager接着会委托给 Authorizer； 

（2）Authorizer是真正的授权者，如果调用如isPermitted(“user:view”)，其首先会通 过PermissionResolver把字符串转换成相应的Permission实例； 

（3）在进行授权之前，其会调用相应的Realm获取Subject相应的角色/权限用于匹配传入 的角色/权限； 

（4）Authorizer会判断Realm的角色/权限是否和传入的匹配，如果有多个Realm，会委托 给ModularRealmAuthorizer进行循环判断，如果匹配如isPermitted*/hasRole* 会返回 true，否则返回false表示授权失败



权限代码示例：

shiro.ini

```ini
[users]
zhangsan=z3,role1,role2
lisi=l4

[roles]
role1=user:insert,user:select
```

```java
public class HelloShiro {
    @Test
    public void shiroLogin(){
        //导入INI配置创建工厂
        IniSecurityManagerFactory factory = new IniSecurityManagerFactory("classpath:shiro.ini");
        //工厂构建安全管理器
        SecurityManager instance =  factory.getInstance();
        //使用工具生效安全管理器
        SecurityUtils.setSecurityManager(instance);
        //使用工具获得subject主体
        Subject subject = SecurityUtils.getSubject();
        //构建账户密码
        UsernamePasswordToken usernamePasswordToken = new UsernamePasswordToken("zhangsan", "z3");
        //使用subject主体去登陆
        subject.login(usernamePasswordToken);
        //打印登录信息
        System.out.println("登录结果："+subject.isAuthenticated());
        //判断角色
        boolean role1 = subject.hasRole("role1");
        System.out.println("是否有role1:"+role1);
        //判断权限
        boolean permitted = subject.isPermitted("user:insert");
        System.out.println("是否有insert权限："+permitted);
    }
}
```

## 9. 多个realm的认证策略设置

### 多个realm实现原理

当应用程序配置多个 Realm 时，例如：用户名密码校验、手机号验证码校验等等。 Shiro 的 ModularRealmAuthenticator 会使用内部的 AuthenticationStrategy 组件判断认 证是成功还是失败。 

AuthenticationStrategy 是一个无状态的组件，它在身份验证尝试中被询问 4 次（这 4 次交互所需的任何必要的状态将被作为方法参数）： 

（1）在所有 Realm 被调用之前 

（2）在调用 Realm 的 getAuthenticationInfo 方法之前 

（3）在调用 Realm 的 getAuthenticationInfo 方法之后 

（4）在所有 Realm 被调用之后 认证策略的另外一项工作就是聚合所有 Realm 的结果信息封装至一个 AuthenticationInfo 实例中，并将此信息返回，以此作为 Subject 的身份信息。

Shiro中定义了三种认证策略：

| AuthenticationStrategy class | 描述                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| AtLeastOneSuccessfulStrategy | 只要有一个（或更多）的 Realm 验证成功，那么认证将视为成功    |
| FirstSuccessfulStrategy      | 第一个 Realm 验证成功，整体认证将视为成功，且后续 Realm 将被忽略 |
| AllSuccessfulStrategy        | 所有 Realm 成功，认证才视为成功                              |

ModularRealmAuthenticator 内置的认证策略默认实现是 AtLeastOneSuccessfulStrategy 方式。可以通过配置修改策略。

## 10. 记住我功能

Shiro 提供了记住我（RememberMe）的功能，比如访问一些网站时，关闭了浏览器， 下次再打开时还是能记住你是谁， 下次访问时无需再登录即可访问。

基本流程:

（1） 首先在登录页面选中 RememberMe 然后登录成功；如果是浏览器登录，一般会 把 RememberMe 的 Cookie 写到客户端并保存下来； 

（2） 关闭浏览器再重新打开；会发现浏览器还是记住你的； 

（3） 访问一般的网页服务器端，仍然知道你是谁，且能正常访问； 

（4） 但是，如果我们访问电商平台时，如果要查看我的订单或进行支付时，此时还 是需要再进行身份认证的，以确保当前用户还是你。

```java
//5 设置 rememberMe
defaultWebSecurityManager.setRememberMeManager(rememberMeManager());


//cookie 属性设置
 public SimpleCookie rememberMeCookie(){
 	SimpleCookie cookie = new SimpleCookie("rememberMe");
 	//设置跨域
 	//cookie.setDomain(domain);
 	cookie.setPath("/");
 	cookie.setHttpOnly(true);
 	cookie.setMaxAge(30*24*60*60);
 	return cookie;
 }
 //创建 Shiro 的 cookie 管理对象
 public CookieRememberMeManager rememberMeManager(){
 	CookieRememberMeManager cookieRememberMeManager = new CookieRememberMeManager();
 	cookieRememberMeManager.setCookie(rememberMeCookie());
	cookieRememberMeManager.setCipherKey("1234567890987654".getBytes());
 	return cookieRememberMeManager;
 }


//配置 Shiro 内置过滤器拦截范围
 @Bean
 public DefaultShiroFilterChainDefinition shiroFilterChainDefinition(){
 	DefaultShiroFilterChainDefinition definition = new DefaultShiroFilterChainDefinition();
 	//设置不认证可以访问的资源
 	definition.addPathDefinition("/myController/userLogin","anon");
 	definition.addPathDefinition("/myController/login","anon");
	//设置需要进行登录认证的拦截范围
 	definition.addPathDefinition("/**","authc");
 	//添加存在用户的过滤器（rememberMe）
 	definition.addPathDefinition("/**","user");
 	return definition;
 }

（2）修改 controller
//登录认证
@GetMapping("userLogin")
public String userLogin(String name, String pwd,@RequestParam(defaultValue = "false")boolean rememberMe, HttpSession session){
 	//1 获取 Subject 对象
 	Subject subject = SecurityUtils.getSubject();
 	//2 封装请求数据到 token 对象中
 	AuthenticationToken token = new UsernamePasswordToken(name,pwd,rememberMe);
 	//3 调用 login 方法进行登录认证
 	try {
 		subject.login(token);
 		session.setAttribute("user",token.getPrincipal().toString());
 		return "main";
 	} catch (AuthenticationException e) {
 		e.printStackTrace();
 		System.out.println("登录失败");
 		return "登录失败";
 	}
}
//登录认证验证 rememberMe
@GetMapping("userLoginRm")
public String userLogin(HttpSession session) {
 	session.setAttribute("user","rememberMe");
 	return "main";
}

```

## 11. 登出操作

直接通过Shiro过滤器即可实现登出

```java
//配置 Shiro 内置过滤器拦截范围
@Bean
public DefaultShiroFilterChainDefinition shiroFilterChainDefinition(){
 	DefaultShiroFilterChainDefinition definition = new 
	DefaultShiroFilterChainDefinition();
 	//设置不认证可以访问的资源
 	definition.addPathDefinition("/myController/userLogin","anon");
 	definition.addPathDefinition("/myController/login","anon");
 	//配置登出过滤器
 	definition.addPathDefinition("/logout","logout");
 	//设置需要进行登录认证的拦截范围
 	definition.addPathDefinition("/**","authc");
 	//添加存在用户的过滤器（rememberMe）
 	definition.addPathDefinition("/**","user");
 	return definition;
}

```

## 12. 授权、角色认证

### 授权

用户登录后，需要验证是否具有指定角色指定权限。Shiro也提供了方便的工具进行判 断。 这个工具就是Realm的doGetAuthorizationInfo方法进行判断。

触发权限判断的有两种方式 （1） 在页面中通过shiro:   属性判断 （2） 在接口服务中通过注解：@Requires进行判断

### 后端接口服务注解

通过给接口服务方法添加注解可以实现权限校验，可以加在控制器方法上，也可以加 在业务方法上，一般加在控制器方法上。常用注解如下： 

**（1）@RequiresAuthentication**  

验证用户是否登录，等同于方法subject.isAuthenticated() 

**（2）@RequiresUser**  

验证用户是否被记忆： 

登录认证成功subject.isAuthenticated()为true 

登录后被记忆subject.isRemembered()为true 

**（3）@RequiresGuest** 

 验证是否是一个guest的请求，是否是游客的请求

此时subject.getPrincipal()为null 

**（4）@RequiresRoles**  

验证subject是否有相应角色，有角色访问方法，没有则会抛出异常 AuthorizationException。 

例如：@RequiresRoles(“aRoleName”) 

void someMethod(); 

只有subject有aRoleName角色才能访问方法someMethod() 

**（5）@RequiresPermissions** 

 验证subject是否有相应权限，有权限访问方法，没有则会抛出异常 AuthorizationException。

 例如：@RequiresPermissions (“file:read”,”wite:aFile.txt”) 

void someMethod(); subject必须同时含有file:read和wite:aFile.txt

权限才能访问方法someMethod()

### 授权验证-没有角色无法访问

```java
（1）添加 controller 方法，并添加验证角色注解
//登录认证验证角色
@RequiresRoles("admin")
@GetMapping("userLoginRoles")
@ResponseBody
public String userLoginRoles() {
 	System.out.println("登录认证验证角色");
 	return "验证角色成功";
}


（2）修改 main.html
<body>
 <h1>Shiro 登录认证后主页面</h1>
 <br>
 登录用户为：<span th:text="${session.user}"></span>
 <br>
 <a href="/logout">登出</a>
<br>
 <a href="/myController/userLoginRoles">测试授权</a>
</body>
     
（3）修改 MyRealm 方法
//自定义授权方法：获取当前登录用户权限信息，返回给 Shiro 用来进行授权对比
@Override
protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
 	System.out.println("进入自定义授权方法");
 	return null;
}
```

### 授权验证-获取角色进行验证

```java
mapper 方法
@Repository
public interface UserMapper extends BaseMapper<User> {
 	@Select("SELECT NAME FROM role WHERE id IN (SELECT rid FROM role_user WHERE uid=(SELECT id FROM USER WHERE NAME=#{principal}))")
 	List<String> getUserRoleInfoMapper(@Param("principal") String principal);
}

service 实现
//获取用户的角色信息
@Override
public List<String> getUserRoleInfo(String principal) {
 	return userMapper.getUserRoleInfoMapper(principal);
}
MyRealm 方法改造
    
//自定义授权方法：获取当前登录用户权限信息，返回给 Shiro 用来进行授权对比
@Override
protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
 	System.out.println("进入自定义授权方法");
 	//获取当前用户身份信息
 	String principal = principalCollection.getPrimaryPrincipal().toString();
 	//调用接口方法获取用户的角色信息
 	List<String> roles = userService.getUserRoleInfo(principal);
 	System.out.println("当前用户角色信息："+roles);
 	//创建对象，存储当前登录的用户的权限和角色
 	SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
 	//存储角色
 	info.addRoles(roles);
 	//返回
 	return info;
}
```

**授权验证-获取权限进行验证**

```java
mapper 方法
 @Select({
 "<script>",
 "select info FROM permissions WHERE id IN ",
 "(SELECT pid FROM role_ps WHERE rid IN (",
 "SELECT id FROM role WHERE NAME IN ",
 "<foreach collection='roles' item='name' open='(' separator=',' close=')'>",
 "#{name}",
 "</foreach>",
 "))",
 "</script>"
})
List<String> getUserPermissionInfoMapper(@Param("roles")List<String> roles);


service 实现
//获取用户角色的权限信息
@Override
public List<String> getUserPermissionInfo(List<String> roles) {
 	return userMapper.getUserPermissionInfoMapper(roles);
}


MyRealm 方法改造
//自定义授权方法：获取当前登录用户权限信息，返回给 Shiro 用来进行授权对比
@Override
protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
 	System.out.println("进入自定义授权方法");
 	//获取当前用户身份信息
 	String principal = principalCollection.getPrimaryPrincipal().toString();
 	//调用接口方法获取用户的角色信息
 	List<String> roles = userService.getUserRoleInfo(principal);
 	System.out.println("当前用户角色信息："+roles);
 	//调用接口方法获取用户角色的权限信息
 	List<String> permissions = userService.getUserPermissionInfo(roles);
 	System.out.println("当前用户权限信息："+permissions);
 	//创建对象，存储当前登录的用户的权限和角色
 	SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
 	//存储角色
 	info.addRoles(roles);
 	//存储权限信息
 	info.addStringPermissions(permissions);
 	//返回
 	return info;
}

添加 controller 方法
//登录认证验证权限
@RequiresPermissions("user:delete")
@GetMapping("userPermissions")
@ResponseBody
public String userLoginPermissions() {
 	System.out.println("登录认证验证权限");
 	return "验证权限成功";
}

改造 main.html
<body>
 <h1>Shiro 登录认证后主页面</h1>
 <br>
 登录用户为：<span th:text="${session.user}"></span>
 <br>
 <a href="/logout">登出</a>
 <br>
 <a href="/myController/userLoginRoles">测试授权-角色验证</a>
 <br>
 <a href="/myController/userPermissions">测试授权-权限验证</a>
</body>
```

### 授权验证-异常处理

（1）创建认证异常处理类，使用@ControllerAdvice 加@ExceptionHandler 实现特殊异 常处理。

```java
@ControllerAdvice
public class PermissionsException {
 	@ResponseBody
 	@ExceptionHandler(UnauthorizedException.class)
 	public String unauthorizedException(Exception ex){
 		return "无权限";
 	}
 	@ResponseBody
 	@ExceptionHandler(AuthorizationException.class)
 	public String authorizationException(Exception ex){
 		return "权限认证失败";
 	}
}
```

Thymeleaf 中常用的 shiro:属性

**guest 标签**

 `<shiro:guest> </shiro:guest>`

 用户没有身份验证时显示相应信息，即游客访问信息。 

**user 标签** 

 `<shiro:user> </shiro:user>`

用户已经身份验证/记住我登录后显示相应的信息。 

**authenticated 标签**  

`<shiro:authenticated> </shiro:authenticated>`

用户已经身份验证通过，即 Subject.login 登录成功，不是记住我登录的。 

**notAuthenticated 标签**  

`<shiro:notAuthenticated> </shiro:notAuthenticated>`
用户已经身份验证通过，即没有调用 Subject.login 进行登录，包括记住我自动登录的 也属于未进行身份验证。 

**principal 标签**  

`<shiro: principal/> <shiro:principal property="username"/>`

相当于((User)Subject.getPrincipals()).getUsername()。

**lacksPermission 标签** 

`<shiro:lacksPermission name="org:create"> </shiro:lacksPermission>` 

如果当前 Subject 没有权限将显示 body 体内容。 

**hasRole 标签**  

`<shiro:hasRole name="admin"> </shiro:hasRole>`

如果当前 Subject 有角色将显示 body 体内容。 

**hasAnyRoles 标签**  

`<shiro:hasAnyRoles name="admin,user"> </shiro:hasAnyRoles>`

如果当前 Subject 有任意一个角色（或的关系）将显示 body 体内容。 

**lacksRole 标签**  

`<shiro:lacksRole name="abc"> </shiro:lacksRole>`

如果当前 Subject 没有角色将显示 body 体内容。 

**hasPermission 标签**  

`<shiro:hasPermission name="user:create"> </shiro:hasPermission>`

如果当前 Subject 有权限将显示 body 体内容

 

修改前端代码

```html
改造 main.html

<body>
 <h1>Shiro 登录认证后主页面</h1><br>
 登录用户为：<span th:text="${session.user}"></span><br>
 <a href="/logout">登出</a><br>
 <a shiro:hasRole="admin" href="/myController/userLoginRoles">测试授权-角色验证</a><br>
 <a shiro:hasPermission="user:delete" 
href="/myController/userPermissions">测试授权-权限验证</a>
</body>
```

