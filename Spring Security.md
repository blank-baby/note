# Spring Security

## 1.引入依赖 

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
    <version>2.4.4</version>
</dependency>
```

## 2.认证

导入依赖直接会起作用，不做任何配置会自动检测是否登录，没有登录访问任何请求会跳转到自带的登录页面

- 这个需要用户是post提交的登录页面验证的三种方式

进行密码验证的是UsernamePasswordAuthenticationFilter这个类，一般根据自己需要验证的用户重新写



### 1.默认用户名是：user  密码是项目启动的时候会自动打印出来密码

### 2.在配置文件中配置

```yaml
spring:
  security:
    user:
      name: jiruixin
      password: 111
```

### 3.配置类里面重写

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder()
    {
        return new BCryptPasswordEncoder();
    }

    //重写这个类可以自己进行用户名的验证
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception{
        auth.inMemoryAuthentication().withUser("user").password(bCryptPasswordEncoder().encode("111")).roles("admin");
    }
}
```

## 3.自定义登录页面验证

```java
package com.runningone.rsp.config;

import com.runningone.rsp.security.UserDetailsServiceImpl;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.authentication.TestingAuthenticationToken;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.context.SecurityContext;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private UserDetailsServiceImpl userDetailsService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {

        http.authorizeRequests()
                .antMatchers("/login.html","/fail").permitAll()
                .anyRequest().authenticated();
        
        http.formLogin()
            
           		//这是自定义的页面
                .loginPage("/login.html")
            
            	//这是也表单中匹配的action地址
                .loginProcessingUrl("/login")
            
            	//这是成功以后跳转的路径
                .successForwardUrl("/hello")
            
            	//这是失败以后跳转的路径
                .failureForwardUrl("/fail");

		//需要关闭这个才能进行密码的验证
        http.csrf().disable();

    }

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder()
    {
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception{
        auth.inMemoryAuthentication().withUser("user").password(bCryptPasswordEncoder().encode("111")).roles("admin");
    }
}

```

- **记录采坑**

如果你要配置了访问地址的公共路径，这是在配置SpringSecurity的路径要忽略这个，按正常配置来，但是在表单中的action路径要加上你配置的公共路径

```yaml
server:
  servlet:
    context-path: /rsp
```



## 4.自定义校验

```java
package com.runningone.rsp.security;

import com.runningone.rsp.comon.BaseException;
import com.runningone.rsp.dao.mapper.UserMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.stereotype.Component;
import com.runningone.rsp.pojo.User;


@Component
public class UserDetailsServiceImpl implements UserDetailsService {

    @Autowired
    private DefaultPwdEncoder defaultPwdEncoder;

    @Autowired
    private UserMapper userMapper;

    @Autowired
    private PermissionService permissionService;


    /**
     * 登录认证,这里需要可以写自己的校验密码，结果返回一个LoginUser(该类需要继承 UserDetails 这个接口)
     * 
     *
     * @param username
     * @return
     */
    @Override
    public UserDetails loadUserByUsername(String username){

        User user = userMapper.selectUserphone(username);
        if ("0".equals(user.getIsenable())) {
            throw new BaseException("该账号已经被禁用");
        }
        
        //后续会拿你这传的的user的password和你输入的password做对比
        return new LoginUser(user, permissionService.getRolePermission(user));

        //List<GrantedAuthority> auths = AuthorityUtils.commaSeparatedStringToAuthorityList("admin");
        //return new User(username, defaultPwdEncoder.encode("111"), auths);
    }
}

```

## 5.各个类的功能介绍

>SpringSecurity的本质就是过滤器链，下面黄色为主要的过滤器

- ChannelProcessingFilter
- WebAsyncManagerIntegrationFilter
- SecurityContextPersistenceFilter
- HeaderWriterFilter
- CorsFilter
- CsrfFilter
- LogoutFilter
- OAuth2AuthorizationRequestRedirectFilter
- Saml2WebSsoAuthenticationRequestFilter
- X509AuthenticationFilter
- AbstractPreAuthenticatedProcessingFilter
- CasAuthenticationFilter
- OAuth2LoginAuthenticationFilter
- Saml2WebSsoAuthenticationFilter
- **UsernamePasswordAuthenticationFilter**
- OpenIDAuthenticationFilter
- DefaultLoginPageGeneratingFilter
- DefaultLogoutPageGeneratingFilter
- ConcurrentSessionFilter
- DigestAuthenticationFilter
- BearerTokenAuthenticationFilter
- BasicAuthenticationFilter
- RequestCacheAwareFilter
- SecurityContextHolderAwareRequestFilter
- JaasApiIntegrationFilter
- RememberMeAuthenticationFilter
- AnonymousAuthenticationFilter
- OAuth2AuthorizationCodeGrantFilter
- SessionManagementFilter
- **ExceptionTranslationFilter**
- **FilterSecurityInterceptor**
- SwitchUserFilter

### 1.验证入口的接口类和实现类

> AuthenticationManager（提供校验的接口，它的authenticate方法可以来校验你传入的token）

> AuthenticationManagerDelegator (AuthenticationManager的实现类)，这是一个内部静态类

```java
@Configuration(proxyBeanMethods = false)
@Import(ObjectPostProcessorConfiguration.class)
public class AuthenticationConfiguration {

	/**
	 * Prevents infinite recursion in the event that initializing the
	 * AuthenticationManager.
	 *
	 * @author Rob Winch
	 * @since 4.1.1
	 */
	static final class AuthenticationManagerDelegator implements AuthenticationManager {

		private AuthenticationManagerBuilder delegateBuilder;

		private AuthenticationManager delegate;

		private final Object delegateMonitor = new Object();

		AuthenticationManagerDelegator(AuthenticationManagerBuilder delegateBuilder) {
			Assert.notNull(delegateBuilder, "delegateBuilder cannot be null");
			this.delegateBuilder = delegateBuilder;
		}

		@Override
		public Authentication authenticate(Authentication authentication) throws AuthenticationException {
			if (this.delegate != null) {
				return this.delegate.authenticate(authentication);
			}
			synchronized (this.delegateMonitor) {
				if (this.delegate == null) {
					this.delegate = this.delegateBuilder.getObject();
					this.delegateBuilder = null;
				}
			}
			return this.delegate.authenticate(authentication);
		}
	}

}
```

> AuthenticationManagerDelegator (AuthenticationManager实现类)

```java
public class GlobalMethodSecurityBeanDefinitionParser implements BeanDefinitionParser {
	/**
	 * Delays the lookup of the AuthenticationManager within MethodSecurityInterceptor, to
	 * prevent issues like SEC-933.
	 *
	 * @author Luke Taylor
	 * @since 3.0
	 */
	static final class AuthenticationManagerDelegator implements AuthenticationManager, BeanFactoryAware {

		private AuthenticationManager delegate;

		private final Object delegateMonitor = new Object();

		private BeanFactory beanFactory;

		private final String authMgrBean;

		AuthenticationManagerDelegator(String authMgrBean) {
			this.authMgrBean = StringUtils.hasText(authMgrBean) ? authMgrBean : BeanIds.AUTHENTICATION_MANAGER;
		}

	}
}

```

> AuthenticationManagerDelegator (AuthenticationManager)

```java
@Order(100)
public abstract class WebSecurityConfigurerAdapter implements WebSecurityConfigurer<WebSecurity> {
	/**
	 * Delays the use of the {@link AuthenticationManager} build from the
	 * {@link AuthenticationManagerBuilder} to ensure that it has been fully configured.
	 *
	 * @author Rob Winch
	 * @since 3.2
	 */
	static final class AuthenticationManagerDelegator implements AuthenticationManager {

		private AuthenticationManagerBuilder delegateBuilder;

		private AuthenticationManager delegate;

		private final Object delegateMonitor = new Object();

		private Set<String> beanNames;

		AuthenticationManagerDelegator(AuthenticationManagerBuilder delegateBuilder, ApplicationContext context) {
			Assert.notNull(delegateBuilder, "delegateBuilder cannot be null");
			Field parentAuthMgrField = ReflectionUtils.findField(AuthenticationManagerBuilder.class,
					"parentAuthenticationManager");
			ReflectionUtils.makeAccessible(parentAuthMgrField);
			this.beanNames = getAuthenticationManagerBeanNames(context);
			validateBeanCycle(ReflectionUtils.getField(parentAuthMgrField, delegateBuilder), this.beanNames);
			this.delegateBuilder = delegateBuilder;
		}

		@Override
		public Authentication authenticate(Authentication authentication) throws AuthenticationException {
			if (this.delegate != null) {
				return this.delegate.authenticate(authentication);
			}
			synchronized (this.delegateMonitor) {
				if (this.delegate == null) {
					this.delegate = this.delegateBuilder.getObject();
					this.delegateBuilder = null;
				}
			}
			return this.delegate.authenticate(authentication);
		}

	}
}

```

>ProviderManager (AuthenticationManager实现类)

```java
public class ProviderManager implements AuthenticationManager, MessageSourceAware, InitializingBean {
	@Override
	public Authentication authenticate(Authentication authentication) throws AuthenticationException {
		Class<? extends Authentication> toTest = authentication.getClass();
		AuthenticationException lastException = null;
		AuthenticationException parentException = null;
		Authentication result = null;
		Authentication parentResult = null;
		int currentPosition = 0;
		int size = this.providers.size();
		for (AuthenticationProvider provider : getProviders()) {
			if (!provider.supports(toTest)) {
				continue;
			}
			if (logger.isTraceEnabled()) {
				logger.trace(LogMessage.format("Authenticating request with %s (%d/%d)",
						provider.getClass().getSimpleName(), ++currentPosition, size));
			}
			try {
				result = provider.authenticate(authentication);
				if (result != null) {
					copyDetails(authentication, result);
					break;
				}
			}
			catch (AccountStatusException | InternalAuthenticationServiceException ex) {
				prepareException(ex, authentication);
				// SEC-546: Avoid polling additional providers if auth failure is due to
				// invalid account status
				throw ex;
			}
			catch (AuthenticationException ex) {
				lastException = ex;
			}
		}
		if (result == null && this.parent != null) {
			// Allow the parent to try.
			try {
				parentResult = this.parent.authenticate(authentication);
				result = parentResult;
			}
			catch (ProviderNotFoundException ex) {
				// ignore as we will throw below if no other exception occurred prior to
				// calling parent and the parent
				// may throw ProviderNotFound even though a provider in the child already
				// handled the request
			}
			catch (AuthenticationException ex) {
				parentException = ex;
				lastException = ex;
			}
		}
		if (result != null) {
			if (this.eraseCredentialsAfterAuthentication && (result instanceof CredentialsContainer)) {
				// Authentication is complete. Remove credentials and other secret data
				// from authentication
				((CredentialsContainer) result).eraseCredentials();
			}
			// If the parent AuthenticationManager was attempted and successful then it
			// will publish an AuthenticationSuccessEvent
			// This check prevents a duplicate AuthenticationSuccessEvent if the parent
			// AuthenticationManager already published it
			if (parentResult == null) {
				this.eventPublisher.publishAuthenticationSuccess(result);
			}

			return result;
		}

		// Parent was null, or didn't authenticate (or throw an exception).
		if (lastException == null) {
			lastException = new ProviderNotFoundException(this.messages.getMessage("ProviderManager.providerNotFound",
					new Object[] { toTest.getName() }, "No AuthenticationProvider found for {0}"));
		}
		// If the parent AuthenticationManager was attempted and failed then it will
		// publish an AbstractAuthenticationFailureEvent
		// This check prevents a duplicate AbstractAuthenticationFailureEvent if the
		// parent AuthenticationManager already published it
		if (parentException == null) {
			prepareException(lastException, authentication);
		}
		throw lastException;
	}
}

```

### 2.具体验证的接口类和实现类

>AuthenticationProvider (具体验证的接口类)，authenticate方法可以验证

> AbstractUserDetailsAuthenticationProvider (AuthenticationProvider实现类)

```java
public abstract class AbstractUserDetailsAuthenticationProvider
		implements AuthenticationProvider, InitializingBean, MessageSourceAware {
	@Override
	public Authentication authenticate(Authentication authentication) throws AuthenticationException {
		Assert.isInstanceOf(UsernamePasswordAuthenticationToken.class, authentication,
				() -> this.messages.getMessage("AbstractUserDetailsAuthenticationProvider.onlySupports",
						"Only UsernamePasswordAuthenticationToken is supported"));
		String username = determineUsername(authentication);
		boolean cacheWasUsed = true;
		UserDetails user = this.userCache.getUserFromCache(username);
		if (user == null) {
			cacheWasUsed = false;
			try {
				user = retrieveUser(username, (UsernamePasswordAuthenticationToken) authentication);
			}
			catch (UsernameNotFoundException ex) {
				this.logger.debug("Failed to find user '" + username + "'");
				if (!this.hideUserNotFoundExceptions) {
					throw ex;
				}
				throw new BadCredentialsException(this.messages
						.getMessage("AbstractUserDetailsAuthenticationProvider.badCredentials", "Bad credentials"));
			}
			Assert.notNull(user, "retrieveUser returned null - a violation of the interface contract");
		}
		try {
			this.preAuthenticationChecks.check(user);
			additionalAuthenticationChecks(user, (UsernamePasswordAuthenticationToken) authentication);
		}
		catch (AuthenticationException ex) {
			if (!cacheWasUsed) {
				throw ex;
			}
			// There was a problem, so try again after checking
			// we're using latest data (i.e. not from the cache)
			cacheWasUsed = false;
			user = retrieveUser(username, (UsernamePasswordAuthenticationToken) authentication);
			this.preAuthenticationChecks.check(user);
			additionalAuthenticationChecks(user, (UsernamePasswordAuthenticationToken) authentication);
		}
		this.postAuthenticationChecks.check(user);
		if (!cacheWasUsed) {
			this.userCache.putUserInCache(user);
		}
		Object principalToReturn = user;
		if (this.forcePrincipalAsString) {
			principalToReturn = user.getUsername();
		}
		return createSuccessAuthentication(principalToReturn, authentication, user);
	}

}

```

>DaoAuthenticationProvider (AbstractUserDetailsAuthenticationProvider实现类)

```java
public class DaoAuthenticationProvider extends AbstractUserDetailsAuthenticationProvider {

	@Override
	@SuppressWarnings("deprecation")
	protected void additionalAuthenticationChecks(UserDetails userDetails,
			UsernamePasswordAuthenticationToken authentication) throws AuthenticationException {
		if (authentication.getCredentials() == null) {
			this.logger.debug("Failed to authenticate since no credentials provided");
			throw new BadCredentialsException(this.messages
					.getMessage("AbstractUserDetailsAuthenticationProvider.badCredentials", "Bad credentials"));
		}
		String presentedPassword = authentication.getCredentials().toString();
		if (!this.passwordEncoder.matches(presentedPassword, userDetails.getPassword())) {
			this.logger.debug("Failed to authenticate since password does not match stored value");
			throw new BadCredentialsException(this.messages
					.getMessage("AbstractUserDetailsAuthenticationProvider.badCredentials", "Bad credentials"));
		}
	}
}

```

### 3.封装验证信息的接口类和实现类

>Authentication (接口类，用户封装校验信息)

> UsernamePasswordAuthenticationToken  (Authentication实现类)用于校验你的信息

## 6.校验流程

### 1.post表单提交

![image-20210505191608173](C:\Users\DELL\AppData\Roaming\Typora\typora-user-images\image-20210505191608173.png)

### 2.自定义认证

![image-20210505191654153](C:\Users\DELL\AppData\Roaming\Typora\typora-user-images\image-20210505191654153.png)



