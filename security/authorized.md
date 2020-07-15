* springboot+shiro 按钮级别的权限控制例子
==============================
* shiroConfiguration配置
```java
    public class ShiroConfiguration {
    	/**
    	 * Shiro的Web过滤器Factory 命名:shiroFilter
    	 */
    	@Bean(name = "shiroFilter")
    	public ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager) {
    		ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
    		//Shiro的核心安全接口,这个属性是必须的
    		shiroFilterFactoryBean.setSecurityManager(securityManager);
            // 设置默认登录的 URL，身份认证失败会访问该 URL
            shiroFilterFactoryBean.setLoginUrl("/login");
            // 设置成功之后要跳转的链接
            shiroFilterFactoryBean.setSuccessUrl("/index");
            // 设置未授权界面，权限认证失败会访问该 URL
            shiroFilterFactoryBean.setUnauthorizedUrl("/unauthorized");
    		/*定义shiro过滤链  Map结构*/
    		Map<String, String> filterChainDefinitionMap = new LinkedHashMap<>();
             /* 过滤链定义，从上向下顺序执行，
              authc:所有url都必须认证通过才可以访问; anon:所有url都都可以匿名访问 */
            /*设置静态资源，无需过滤即可访问的地址*/
    		filterChainDefinitionMap.put("/", "anon");
    		filterChainDefinitionMap.put("/static/**", "anon");
    		filterChainDefinitionMap.put("/login/auth", "anon");
    		filterChainDefinitionMap.put("/login/logout", "anon");
    		filterChainDefinitionMap.put("/error", "anon");
    		filterChainDefinitionMap.put("/**", "authc");
    		shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
    		return shiroFilterFactoryBean;
    	}
    
    	/**
    	 * 不指定名字的话，自动创建一个方法名第一个字母小写的bean
    	 */
    	@Bean
    	public SecurityManager securityManager() {
    		DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
    		securityManager.setRealm(userRealm());
    		return securityManager;
    	}
    	/**
    	 * Shiro Realm 继承自AuthorizingRealm的自定义Realm,即指定Shiro验证用户登录的类为自定义的
    	 */
    	@Bean
    	public UserRealm userRealm() {
    		UserRealm userRealm = new UserRealm();
    		return userRealm;
    	}
    	/**
    	 * Shiro生命周期处理器
    	 */
    	@Bean
    	public LifecycleBeanPostProcessor lifecycleBeanPostProcessor() {
    		return new LifecycleBeanPostProcessor();
    	}    
    	/**
    	 * 开启Shiro的注解(如@RequiresRoles,@RequiresPermissions),需借助SpringAOP扫描使用Shiro注解的类,并在必要时进行安全逻辑验证
    	 * 配置以下两个bean(DefaultAdvisorAutoProxyCreator(可选)和AuthorizationAttributeSourceAdvisor)即可实现此功能
    	 */	
    	@Bean
    	public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor() {
    		AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
    		authorizationAttributeSourceAdvisor.setSecurityManager(securityManager());
    		return authorizationAttributeSourceAdvisor;
    	}
    }
```
* 自定义Realm,在Shiro中，用户角色和权限的获取是在Realm的doGetAuthorizationInfo()方法中实现的,重写该方法
```java
    public class UserRealm extends AuthorizingRealm {

    	@Override
    	@SuppressWarnings("unchecked")
    	protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
    		Session session = SecurityUtils.getSubject().getSession();
    		//查询用户的权限
    		JSONObject permission = (JSONObject) session.getAttribute(Constants.SESSION_USER_PERMISSION);
    		logger.info("permission的值为:" + permission);
    		logger.info("本用户权限为:" + permission.get("permissionList"));
    		//为当前用户设置角色和权限
    		SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();
    		authorizationInfo.addStringPermissions((Collection<String>) permission.get("permissionList"));
    		return authorizationInfo;
    	}
    
    	/**
    	 * 验证当前登录的Subject
    	 * LoginController.login()方法中执行Subject.login()时 执行此方法
    	 */
    	@Override
    	protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authcToken) throws AuthenticationException {
    		String loginName = (String) authcToken.getPrincipal();
    		// 获取用户密码
    		String password = new String((char[]) authcToken.getCredentials());
    		JSONObject user = loginService.getUser(loginName, password);
    		if (user == null) {
    			//没找到帐号
    			throw new UnknownAccountException();
    		}
    		//交给AuthenticatingRealm使用CredentialsMatcher进行密码匹配，如果觉得人家的不好可以自定义实现
    		SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(
    				user.getString("username"),
    				user.getString("password"),
    				//ByteSource.Util.bytes("salt"), salt=username+salt,采用明文访问时，不需要此句
    				getName()
    		);
    		//session中不需要保存密码
    		user.remove("password");
    		//将用户信息放入session中
    		SecurityUtils.getSubject().getSession().setAttribute(Constants.SESSION_USER_INFO, user);
    		return authenticationInfo;
    	}
    }

```

* 对没有登录的请求进行拦截, 全部返回json信息. 覆盖掉shiro原本的跳转login.jsp的拦截方式
```java

public class AjaxPermissionsAuthorizationFilter extends FormAuthenticationFilter {

	@Override
	protected boolean onAccessDenied(ServletRequest request, ServletResponse response) {
		JSONObject jsonObject = new JSONObject();
		jsonObject.put("code", ErrorEnum.E_20011.getErrorCode());
		jsonObject.put("msg", ErrorEnum.E_20011.getErrorMsg());
		PrintWriter out = null;
		HttpServletResponse res = (HttpServletResponse) response;
		try {
			res.setCharacterEncoding("UTF-8");
			res.setContentType("application/json");
			out = response.getWriter();
			out.println(jsonObject);
		} catch (Exception e) {
		} finally {
			if (null != out) {
				out.flush();
				out.close();
			}
		}
		return false;
	}

	@Bean
	public FilterRegistrationBean registration(AjaxPermissionsAuthorizationFilter filter) {
		FilterRegistrationBean registration = new FilterRegistrationBean(filter);
		registration.setEnabled(false);
		return registration;
	}
}
```
* 设置subject需要的权限
```java
public class UserController {
	/**
	 * 查询用户列表
	 */
	@RequiresPermissions("user:list")
	@GetMapping("/list")
	public JSONObject listUser(HttpServletRequest request) {
		return userService.listUser(CommonUtil.request2Json(request));
	}

	@RequiresPermissions("user:add")
	@PostMapping("/addUser")
	public JSONObject addUser(@RequestBody JSONObject requestJson) {
		return userService.addUser(requestJson);
	}

	@RequiresPermissions("user:update")
	@PostMapping("/updateUser")
	public JSONObject updateUser(@RequestBody JSONObject requestJson) {
		return userService.updateUser(requestJson);
	}

	@RequiresPermissions(value = {"user:add", "user:update"}, logical = Logical.OR)
	@GetMapping("/getAllRoles")
	public JSONObject getAllRoles() {
		return userService.getAllRoles();
	}

	/**
	 * 角色列表
	 */
	@RequiresPermissions("role:list")
	@GetMapping("/listRole")
	public JSONObject listRole() {
		return userService.listRole();
	}

	/**
	 * 查询所有权限, 给角色分配权限时调用
	 */
	@RequiresPermissions("role:list")
	@GetMapping("/listAllPermission")
	public JSONObject listAllPermission() {
		return userService.listAllPermission();
	}

	/**
	 * 新增角色
	 */
	@RequiresPermissions("role:add")
	@PostMapping("/addRole")
	public JSONObject addRole(@RequestBody JSONObject requestJson) {
		return userService.addRole(requestJson);
	}

	/**
	 * 修改角色
	 */
	@RequiresPermissions("role:update")
	@PostMapping("/updateRole")
	public JSONObject updateRole(@RequestBody JSONObject requestJson) {
		return userService.updateRole(requestJson);
	}

	/**
	 * 删除角色
	 */
	@RequiresPermissions("role:delete")
	@PostMapping("/deleteRole")
	public JSONObject deleteRole(@RequestBody JSONObject requestJson) {
		return userService.deleteRole(requestJson);
	}
}
```
* 数据库中对不同角色权限进行分配


![RUNOOB 图标](http://localhost:3000/images/auth.jpg)
