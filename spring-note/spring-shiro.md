spring-shiro
===

摘要
====

根据shiro的设计思路，用户与角色之前的关系多对
多，角色与权限之间的关系也是多对多。因此，需要建立
五张表，用户表(存储用户名、密码、盐等)、角色表(角
色名称、相关描述等)、权限表(权限名称、相关描述等)
用户-角色对应中间表(用户ID和角色ID作为联合主键)、
角色-权限对应中间表(以角色ID和权限ID为联合主键)
shiro需要根据用户名和密码首先判断登录的用户是否合法
任何再对合法用户授权。这个过程就是Realm的实现过程

流程
====

1、使用用户的登录信息创建令牌
UsernamePasswordToken token = new UsernamePassword
		Token(username, password);
token可以理解为用户令牌，登录的过程被抽象为shiro验
证是否具有合法身份已经相关权限
2、执行登录操作
//注入SecurityManager
SecurityUtils.setSecurityManager(securityManager);
//获取subject单例对象
Subject subject = SecurityUtils.getSubject();
//登录
subject.login(token);
shiro的核心部分是securityManager，它负责安全认证与
授权。shiro本身已经实现了所有的细节，用户可以完全
把它当做一个黑盒来使用。securityUtils对象，本质上就
是一个工厂类似spring中的ApplicationContext。subject
是初学者比较难于理解的对象，很多人以为它可以等同于
user，其实不然。subject中文翻译为项目。 通过令牌
token与项目subject的登录login关系，shiro保证了项目
整体的安全
3、判断用户，shiro本身无法知道所持有令牌的用户是否
合法，因为除了项目的设计人员恐怕谁都无法得知。因此
Realm是整个框架中为数不多的必须由设计者自行实现的
模块，当然shiro提供了多种实现的途径，最常见是数据库
查询
4、authorizationInfo代表了角色的权限信息集合
AuthenticationInfo代表了用户的额角色信息集合。如此
一来，当设计人员对项目中的某一个URL路径设置了只允许
某个角色或具有某种权限才可以访问的控制约束的时候，
shiro就可以通过以上二个对象来判断了

实现
====

先理解几个概念：缓存机制、散列算法、加密算法

1、缓存机制：本质就是将原本只能存储在内存中的数据通
过算法保存到硬盘上，再跟进需求依次取出。可以理解为
ehcache是一个map对象，put保存对象，get获取对象
2、散列算法与加密算法：散列和加密本质上都是讲一个
object编程一串无意义的字符串，不通电是经过散列的对
象是无法复原，是一个单向的过程。
3、在用户注册时，为了保证安全性，最好使用盐
public class PasswordHelper {
	private RandomNumberGenerator rand = new 
		SecureRandomNumberGenerator();
	private String algorithmName = "md5";
	private final int hashIterations = 2;
	
	public void encryptPassword(User user) {
		user.setSalt(rand.nextBytes().toHex());
		String newPassword = new SimpleHash(
			algorithmName,user.getPassword());
		ByteSource.Util.bytes(user.getCredentials
		Salt()),hashIterations).toHex();
		user.setPassword(newPassword);	
	}	
}
4匹配
CredentialsMatcher是一个接口，功能就是用来匹配用户
登录使用的令牌和数据库中保存的用户信息是否匹配。当
然它的功能不仅如此。在此介绍这个接口的一个实现类
:HashedCredentialMatcher
public class RetryLimitHashedCredentialsMatcher
extends HashedcredentialMatcher {
	//声明一个缓存接口，这个接口是shiro缓存管理的
	一部分，具体的事项可以通过外部容器注入
	private CachepasswordRetryCache;
	public RetryLimitHashedCredentialsMatcher(
	CacheManager cacheManager) {
		passwordRetryCache = cacheManager.getCache("passwordRetyCache");
	}
	public boolean doCredentialsMatch(AuthenticationToken token, AuthenticationInfo info) {
		String username = (String)token.getPrincipal();
		AtomicInteger retryCount = passwordRetryCa
		che.get(username);
		if(retryCount == null) {
			retryCount = new AtomicInteger(0);
			passwordRetryCache.put(username,retryCount);
		}
		//自定义一个验证过程，当用户输入错误五次以上
		禁止用户登录一段时间
		if(retryCount.incrementAndGet() > 5) {
			throw new ExcessiveAttemptsException();
		}
		boolean match = super.doCredentialMathch(token, info);
		if(match) {
			passwordRetryCache.remove(username);
		}
		return match;
	}
}
在这个实力设计人员仅仅增加了一个不允许连续错误登录
的判断，真正匹配的过程还是交给它的直接父类去完成。
连续登录错误的判断依靠ehcache缓存来实现
5、获取用户的角色和权限信息
public class UserRealm extends AuthorizingRealm {
	private UserService userService = new UserServiceImpl();
	protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
		String username = (String)principals.getPrimaryPrincipal();
		SimpleAuthorizationInfo anthorizationInfo 
		= new SimpleAuthorizationInfo();
		Set roles = userService.findRoles(username);
		Set roleNames = new HashSet();
		for(Role role : roles) {
			roleNames.add(role.getRole());
		}

		Set permissions = userService.findPermissions(username);
		Set permissionNames = new HashSet();
		for(Permission permission : permissions) {
			permissionNames.add(permission.getPermission());
		}
	}
	
	doGetAuthenticationInfo(AuthenticationToken 
		token) throws AuthenticationException {
		String username = (String)token.getPrincipal();
		User user = userService.findByUsername(username);
		if(user == null) {
			throw new UnknownAccountException();
		}
		if(user.getLocked() == 0) {
			throw new LockedAccountException();
		}

		SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(user.getUserName(), user.getPassword(), ByteSource.Util.bytes(user.getCredentialsSalt()), getName());
		return authenticationInfo;
	}
}
6、会话，用户的一次登录即为一次会话，shiro也可以代
替Tomcat等容器管理会话。目的是当用户停留在某个页面
长时间无动作的时候，在此对任何链接的访问都会被重定
向到登录页面要求重新输入用户名和密码而不需要再serv
let中不停的判断session中是否包含user对象

spring-shiro-web.xml
====

http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
http://www.springframework.org/schema/context
http://www.springframework.org/schema/context/spring-context-3.1.xsd
http://www.springframework.org/schema/mvc
http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd">

/authc/admin = roles[admin]
/authc/** = authc
/** = anon

需要注意filterChainDefinitions过滤器中对于路径的配
置是有顺序的，当找到匹配的条目之后容器不会再继续寻
找。因此带有通配符的路径要放在后面。三条配置的含义
是： /authc/admin需要用户有用admin权限、/authc/**
用户必须登录才能访问、/**其他所有路径任何人都可以
访问

登录
====

@Controller 
public class LoginController {
	@Autowired 
	private UserService userService;

	@RequestMapping("login")
	public ModelAndView login(@RequestParam("username") String username, @RequestParam("password") String password) {
		UsernamePasswordToken token = new UsernamePasswordToken(username, password);
		Subject subject = securityUtils.getSubject();
		try{
			subject.login(token);
		}catch(IncorrectCredentialException ice) {
			mv.addObject("message","password error")l
			return mv;
		}
		User user = userService.findByUsername(username);
		subject.getSessuib().setAttribute("user",user);
		return new ModelAndView("success");
	}
}
登录完成以后，当前用户信息被保存进session。这个ses
sion是通过shiro管理的会话对象，要获取依然必须通过
shiro。传统的session中不存在user对象

@Controller
@RequestMapping("authc")
//authc/** = authc==任何通过表单登录的用户都可以访问
public class AuthcController {
	@RequestMapping(anyuser)
	public ModelAndView anyuser() {
		Subject subject = securityUtils.getSubject();
		User user = (User)subject.getSession().getAttribute("user");
		return new ModelAndView("inner");
	}

	//authc/admin = user[admin] 只有具备admin角色的用户才可以访问，否则请求将被重定向至登录界面
	@RequestMapping("admin")
	public ModelAndView admin() {
		Subject subject=SecurityUtils.getSubject();
		User user= (User) subject.getSession().getAttribute("user");
		returnnewModelAndView("inner");
		}
	}
}