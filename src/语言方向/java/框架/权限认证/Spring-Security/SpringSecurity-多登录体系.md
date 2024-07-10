[toc]

---

 

# 前言

有时系统会存在多个用户登录体系，比如系统用户和外部用户。那我们如果使用 `SpringSecurity` 如何配置多个用户登录体系。

这里在若依框架的基础上实现多用户登录。



# ruoyi

## 登录用户

1. 将原先的 `LoginUser` 抽出通用父类 `BaseLoginUser`
2. 增加外部用户 `TapLoginUser`

```java

public abstract class BaseLoginUser implements UserDetails {

    /**
     * 用户ID
     */
    private Long userId;

    /**
     * 用户唯一标识
     */
    private String token;

    /**
     * 登录时间
     */
    private Long loginTime;

    /**
     * 过期时间
     */
    private Long expireTime;

    /**
     * 登录IP地址
     */
    private String ipaddr;

    /**
     * 登录地点
     */
    private String loginLocation;

    /**
     * 浏览器类型
     */
    private String browser;

    /**
     * 操作系统
     */
    private String os;


    /**
     * 手机号
     */
    private String phone;

//    public abstract String getTokenKey();

    public String getToken() {
        return token;
    }

    public void setToken(String token) {
        this.token = token;
    }

    public Long getLoginTime() {
        return loginTime;
    }

    public void setLoginTime(Long loginTime) {
        this.loginTime = loginTime;
    }

    public String getIpaddr() {
        return ipaddr;
    }

    public void setIpaddr(String ipaddr) {
        this.ipaddr = ipaddr;
    }

    public String getLoginLocation() {
        return loginLocation;
    }

    public void setLoginLocation(String loginLocation) {
        this.loginLocation = loginLocation;
    }

    public String getBrowser() {
        return browser;
    }

    public void setBrowser(String browser) {
        this.browser = browser;
    }

    public String getOs() {
        return os;
    }

    public void setOs(String os) {
        this.os = os;
    }

    public Long getExpireTime() {
        return expireTime;
    }

    public void setExpireTime(Long expireTime) {
        this.expireTime = expireTime;
    }

    public Long getUserId() {
        return userId;
    }

    public void setUserId(Long userId) {
        this.userId = userId;
    }

    public String getPhone() {
        return phone;
    }

    public void setPhone(String phone) {
        this.phone = phone;
    }
}

```



```java

/**
 * 登录用户身份权限
 *
 * @author ruoyi
 */
public class LoginUser extends BaseLoginUser implements UserDetails {
    private static final long serialVersionUID = 1L;

    /**
     * 部门ID
     */
    private Long deptId;


    /**
     * 权限列表
     */
    private Set<String> permissions;

    /**
     * 用户信息
     */
    private SysUser user;

    public LoginUser() {
    }

    public LoginUser(SysUser user, Set<String> permissions) {
        this.user = user;
        this.permissions = permissions;
    }

    public LoginUser(Long userId, Long deptId, SysUser user, Set<String> permissions) {
        this.setUserId(userId);
        this.deptId = deptId;
        this.user = user;
        this.permissions = permissions;
    }

    public Long getDeptId() {
        return deptId;
    }

    public void setDeptId(Long deptId) {
        this.deptId = deptId;
    }


    @JSONField(serialize = false)
    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getUserName();
    }

    /**
     * 账户是否未过期,过期无法验证
     */
    @JSONField(serialize = false)
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    /**
     * 指定用户是否解锁,锁定的用户无法进行身份验证
     *
     * @return
     */
    @JSONField(serialize = false)
    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    /**
     * 指示是否已过期的用户的凭据(密码),过期的凭据防止认证
     *
     * @return
     */
    @JSONField(serialize = false)
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    /**
     * 是否可用 ,禁用的用户不能身份验证
     *
     * @return
     */
    @JSONField(serialize = false)
    @Override
    public boolean isEnabled() {
        return true;
    }


    public Set<String> getPermissions() {
        return permissions;
    }

    public void setPermissions(Set<String> permissions) {
        this.permissions = permissions;
    }

    public SysUser getUser() {
        return user;
    }

    public void setUser(SysUser user) {
        this.user = user;
    }

    @JSONField(serialize = false)
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return Collections.singletonList(() -> "SYS");
    }

//    @Override
//    public String getTokenKey() {
//        return CacheConstants.LOGIN_SYS_TOKEN_KEY;
//    }
}

```

```java

/**
 * 登录用户身份权限
 *
 * @author ruoyi
 */
public class LoginTapUser extends BaseLoginUser implements UserDetails {
    private static final long serialVersionUID = 1L;


    /**
     * 昵称
     */
    private String nickName;

    /**
     * 头像
     */
    private String avatar;

    /**
     * 会员等级，0免费会员，1高级会员，2专业会员
     */
    private String memberLevel;

    /**
     * 会员有效期
     */
    @JsonFormat(pattern = "yyyy-MM-dd")
    private Date memberValidDate;

    /**
     * 权限列表
     */
    private Set<String> permissions;

    public LoginTapUser() {
    }

    public LoginTapUser(String memberLevel, Set<String> permissions) {
        this.memberLevel = memberLevel;
        this.memberValidDate = memberValidDate;
        this.permissions = permissions;
    }

    public LoginTapUser(Long userId, String phone, String nickName, String avatar, String memberLevel, Date memberValidDate, Set<String> permissions) {
        this.setUserId(userId);
        this.setPhone(phone);
        this.nickName = nickName;
        this.avatar = avatar;
        this.memberLevel = memberLevel;
        this.memberValidDate = memberValidDate;
        this.permissions = permissions;
    }

    /**
     * 账户是否未过期,过期无法验证
     */
    @JSONField(serialize = false)
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    /**
     * 指定用户是否解锁,锁定的用户无法进行身份验证
     *
     * @return
     */
    @JSONField(serialize = false)
    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    /**
     * 指示是否已过期的用户的凭据(密码),过期的凭据防止认证
     *
     * @return
     */
    @JSONField(serialize = false)
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    /**
     * 是否可用 ,禁用的用户不能身份验证
     *
     * @return
     */
    @JSONField(serialize = false)
    @Override
    public boolean isEnabled() {
        return true;
    }

    public Set<String> getPermissions() {
        return permissions;
    }

    public void setPermissions(Set<String> permissions) {
        this.permissions = permissions;
    }

    @JSONField(serialize = false)
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return Collections.singletonList(() -> "TAP");
    }

    @Override
    public String getPassword() {
        return this.getPhone();
    }

    @Override
    public String getUsername() {
        return this.getPhone();
    }

    public String getMemberLevel() {
        return memberLevel;
    }

    public void setMemberLevel(String memberLevel) {
        this.memberLevel = memberLevel;
    }

    public Date getMemberValidDate() {
        return memberValidDate;
    }

    public void setMemberValidDate(Date memberValidDate) {
        this.memberValidDate = memberValidDate;
    }

    public String getNickName() {
        return nickName;
    }

    public void setNickName(String nickName) {
        this.nickName = nickName;
    }

    public String getAvatar() {
        return avatar;
    }

    public void setAvatar(String avatar) {
        this.avatar = avatar;
    }

//    @Override
//    public String getTokenKey() {
//        return CacheConstants.LOGIN_TAP_TOKEN_KEY;
//    }
}

```



## Filter

```java

/**
 * token过滤器 验证token有效性
 *
 * @author ruoyi
 */
@Component
public class JwtAuthenticationTokenFilter extends OncePerRequestFilter {
    @Autowired
    private TokenService tokenService;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
            throws ServletException, IOException {
        BaseLoginUser loginUser = tokenService.getLoginUser(request);
        if (StringUtils.isNotNull(loginUser) && StringUtils.isNull(SecurityUtils.getAuthentication())) {
            tokenService.verifyToken(loginUser);
            UsernamePasswordAuthenticationToken authenticationToken;
            
            // 增加登录用户判断，生成Token.
            if (loginUser instanceof LoginTapUser) {
                authenticationToken = new TapSmsAuthenticationToken(loginUser, null, null);
            } else {
                authenticationToken = new UsernamePasswordAuthenticationToken(loginUser, null, ((LoginUser) loginUser).getAuthorities());
            }
            authenticationToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
            SecurityContextHolder.getContext().setAuthentication(authenticationToken);
        }
        chain.doFilter(request, response);
    }
}

```



## AuthenticationToken

```java

/**
 * Sms-Token
 */
public class TapSmsAuthenticationToken extends UsernamePasswordAuthenticationToken {
    public TapSmsAuthenticationToken(Object principal, Object credentials) {
        super(principal, credentials);
    }

    public TapSmsAuthenticationToken(Object principal, Object credentials, Collection<? extends GrantedAuthority> authorities) {
        super(principal, credentials, authorities);
    }
}

```



## Provider

```java

/**
 * 短信登录
 */
public class TapSmsAuthenticationProvider extends DaoAuthenticationProvider {

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {

        String phone = authentication.getName();
        String code = (String) authentication.getCredentials();
        if (StringUtils.isBlank(phone)) {
            throw new UsernameNotFoundException(MessageUtils.message("valid.phone.notblank"));
        }
        if (StringUtils.isBlank(code)) {
            throw new BadCredentialsException(MessageUtils.message("valid.code.notblank"));
        }

        UserDetailsService userDetailsService = (UserDetailsService) SpringUtils.getBean("tap-user");
        UserDetails userDetails = userDetailsService.loadUserByUsername(phone);

        return new TapSmsAuthenticationToken(userDetails, null, null);
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return authentication.equals(TapSmsAuthenticationToken.class);
    }
}

```

## SecurityConfig

如果需要分离system和tap前缀的角色权限，可以加判断。

```java
                .antMatchers("/system/**").hasAuthority("SYS")
//                .antMatchers("/tap/**").hasAuthority("TAP")
```

```java

/**
 * spring security配置
 *
 * @author ruoyi
 */
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    /**
     * 自定义用户认证逻辑
     */
    @Resource(name = "sys-user")
    private UserDetailsService userDetailsService;

    / 
    @Resource(name = "tap-user")
    private UserDetailsService tapUserDetailsService;

    /**
     * 认证失败处理类
     */
    @Autowired
    private AuthenticationEntryPointImpl unauthorizedHandler;

    /**
     * 退出处理类
     */
    @Autowired
    private LogoutSuccessHandlerImpl logoutSuccessHandler;

    /**
     * token认证过滤器
     */
    @Autowired
    private JwtAuthenticationTokenFilter authenticationTokenFilter;

    /**
     * 跨域过滤器
     */
    @Autowired
    private CorsFilter corsFilter;

    /**
     * 允许匿名访问的地址
     */
    @Autowired
    private PermitAllUrlProperties permitAllUrl;

    /**
     * 解决 无法直接注入 AuthenticationManager
     *
     * @return
     * @throws Exception
     */
    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        TapSmsAuthenticationProvider dao1 = new TapSmsAuthenticationProvider();
        dao1.setUserDetailsService(tapUserDetailsService);
        dao1.setPasswordEncoder(bCryptPasswordEncoder());

        DaoAuthenticationProvider dao2 = new DaoAuthenticationProvider();
        dao2.setUserDetailsService(userDetailsService);
        dao2.setPasswordEncoder(bCryptPasswordEncoder());

        ProviderManager manager = new ProviderManager(dao1, dao2);
        return manager;
    }

    /**
     * anyRequest          |   匹配所有请求路径
     * access              |   SpringEl表达式结果为true时可以访问
     * anonymous           |   匿名可以访问
     * denyAll             |   用户不能访问
     * fullyAuthenticated  |   用户完全认证可以访问（非remember-me下自动登录）
     * hasAnyAuthority     |   如果有参数，参数表示权限，则其中任何一个权限可以访问
     * hasAnyRole          |   如果有参数，参数表示角色，则其中任何一个角色可以访问
     * hasAuthority        |   如果有参数，参数表示权限，则其权限可以访问
     * hasIpAddress        |   如果有参数，参数表示IP地址，如果用户IP和参数匹配，则可以访问
     * hasRole             |   如果有参数，参数表示角色，则其角色可以访问
     * permitAll           |   用户可以任意访问
     * rememberMe          |   允许通过remember-me登录的用户访问
     * authenticated       |   用户登录后可访问
     */
    @Override
    protected void configure(HttpSecurity httpSecurity) throws Exception {
        // 注解标记允许匿名访问的url
        ExpressionUrlAuthorizationConfigurer<HttpSecurity>.ExpressionInterceptUrlRegistry registry = httpSecurity.authorizeRequests();
        permitAllUrl.getUrls().forEach(url -> registry.antMatchers(url).permitAll());

        httpSecurity
                // CSRF禁用，因为不使用session
                .csrf().disable()
                // 禁用HTTP响应标头
                .headers().cacheControl().disable().and()
                // 认证失败处理类
                .exceptionHandling().authenticationEntryPoint(unauthorizedHandler).and()
                // 基于token，所以不需要session
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS).and()
                // 过滤请求
                .authorizeRequests()
                .antMatchers("/system/**").hasAuthority("SYS")
//                .antMatchers("/tap/**").hasAuthority("TAP")
                // 对于登录login 注册register 验证码captchaImage 允许匿名访问
                .antMatchers("/login", "/register", "/captchaImage", "/tap/login", "/tap/getCode", "/tap/open/*", "/tap/wx/mp/code/login", "/tap/wx/mp/code/login/*").permitAll()
                .antMatchers("/wx/mp/**", "/pay/notify/**").permitAll()
                // 静态资源，可匿名访问
                .antMatchers(HttpMethod.GET, "/", "/*.html", "/**/*.html", "/**/*.css", "/**/*.js", "/profile/**").permitAll()
                .antMatchers("/swagger-ui.html", "/swagger-resources/**", "/webjars/**", "/*/api-docs", "/druid/**").permitAll()
                // 除上面外的所有请求全部需要鉴权认证
                .anyRequest().authenticated()
                .and()
                .headers().frameOptions().disable();
        // 添加Logout filter
        httpSecurity.logout().logoutUrl("/logout").logoutSuccessHandler(logoutSuccessHandler);
        // 添加JWT filter
        httpSecurity.addFilterBefore(authenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);
        // 添加CORS filter
        httpSecurity.addFilterBefore(corsFilter, JwtAuthenticationTokenFilter.class);
        httpSecurity.addFilterBefore(corsFilter, LogoutFilter.class);
    }

    /**
     * 强散列哈希加密实现
     */
    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }

    /**
     * 身份认证接口
     */
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService);
    }
}
```

## Service

### 增加TapUser的登录

```java
// 增加登录服务

private String checkAndLogin(String mobile, String code, String ticket) {
    checkMobileIfAbsent(mobile, ticket);
    Authentication authentication = null;
    try {
        TapSmsAuthenticationToken authenticationToken = new TapSmsAuthenticationToken(mobile, code);
        AuthenticationContextHolder.setContext(authenticationToken);
        // 该方法会去调用UserDetailsServiceImpl.loadUserByUsername
        authentication = authenticationManager.authenticate(authenticationToken);
    } catch (Exception e) {
        // 日志记录
        logger.error("用户登录异常：{}", e.getMessage());
        //            if (e instanceof BadCredentialsException) {
        //                AsyncManager.me().execute(AsyncFactory.recordLogininfor(mobile, Constants.LOGIN_FAIL, MessageUtils.message("user.password.not.match")));
        //                throw new UserPasswordNotMatchException();
        //            } else {
        //                AsyncManager.me().execute(AsyncFactory.recordLogininfor(mobile, Constants.LOGIN_FAIL, e.getMessage()));
        throw new ServiceException(e.getMessage());
        //            }

    } finally {
        AuthenticationContextHolder.clearContext();
    }
    //        AsyncManager.me().execute(AsyncFactory.recordLogininfor(username, Constants.LOGIN_SUCCESS, MessageUtils.message("user.login.success")));
    LoginTapUser loginUser = (LoginTapUser) authentication.getPrincipal();
    recordLoginInfo(loginUser.getUserId());
    // 生成token
    return tokenService.createToken(loginUser);
}
```

### 增加TapUser 的 UserDetailService

```java

/**
 * Tap 用户验证处理
 *
 * @author B.Smile
 */
@Service("tap-user")
public class TapUserDetailsServiceImpl implements UserDetailsService {
    private static final Logger log = LoggerFactory.getLogger(TapUserDetailsServiceImpl.class);

    @Autowired
    private ITapUserService userService;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        TapUser user = userService.selectTapUserByPhone(username);
        if (StringUtils.isNull(user)) {
            log.info("登录用户：{} 不存在.", username);
            throw new UsernameNotFoundException(MessageUtils.message("user.not.exists"));
        } else if (UserStatus.DELETED.getCode().equals(user.getDelFlag())) {
            log.info("登录用户：{} 已被删除.", username);
            throw new ServiceException(MessageUtils.message("user.password.delete"));
        } else if (UserStatus.DISABLE.getCode().equals(user.getStatus())) {
            log.info("登录用户：{} 已被停用.", username);
            throw new ServiceException(MessageUtils.message("user.blocked"));
        }

        return createLoginUser(user);
    }

    public UserDetails createLoginUser(TapUser user) {
        return new LoginTapUser(user.getId(), user.getPhone(), user.getNickName(), user.getAvatar(), user.getMemberLevel(), user.getMemberValidDate(), null);
    }
}
```



### 改造TokenService

主要将从 `LoginUser`  改成父类 `BaseLoginUser`

```java
package com.ruoyi.framework.web.service;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.TimeUnit;
import javax.servlet.http.HttpServletRequest;

import com.ruoyi.common.core.domain.model.BaseLoginUser;
import com.ruoyi.common.core.domain.model.LoginTapUser;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import com.ruoyi.common.constant.CacheConstants;
import com.ruoyi.common.constant.Constants;
import com.ruoyi.common.core.domain.model.LoginUser;
import com.ruoyi.common.core.redis.RedisCache;
import com.ruoyi.common.utils.ServletUtils;
import com.ruoyi.common.utils.StringUtils;
import com.ruoyi.common.utils.ip.AddressUtils;
import com.ruoyi.common.utils.ip.IpUtils;
import com.ruoyi.common.utils.uuid.IdUtils;
import eu.bitwalker.useragentutils.UserAgent;
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;

/**
 * token验证处理
 *
 * @author ruoyi
 */
@Component
public class TokenService {
    private static final Logger log = LoggerFactory.getLogger(TokenService.class);

    // 令牌自定义标识
    @Value("${token.header}")
    private String header;

    // 令牌秘钥
    @Value("${token.secret}")
    private String secret;

    // 令牌有效期（默认30分钟）
    @Value("${token.expireTime}")
    private int expireTime;

    // 令牌有效期（默认3天）
    @Value("${token.clientExpireTime}")
    private int clientExpireTime;

    protected static final long MILLIS_SECOND = 1000;

    protected static final long MILLIS_MINUTE = 60 * MILLIS_SECOND;

    private static final Long MILLIS_MINUTE_TEN = 20 * 60 * 1000L;

    private static final Long MILLIS_MINUTE_HOUR = 6 * 60 * 60 * 1000L;

    @Autowired
    private RedisCache redisCache;

    /**
     * 获取用户身份信息
     *
     * @return 用户信息
     */
    public BaseLoginUser getLoginUser(HttpServletRequest request) {
        // 获取请求携带的令牌
        String token = getToken(request);
        if (StringUtils.isNotEmpty(token)) {
            try {
                Claims claims = parseToken(token);
                // 解析对应的权限以及用户信息
                String uuid = (String) claims.get(Constants.LOGIN_USER_KEY);
                String userKey = getTokenKey(uuid);
                BaseLoginUser user = redisCache.getCacheObject(userKey);
                return user;
            } catch (Exception e) {
                log.error("获取用户信息异常'{}'", e.getMessage());
            }
        }
        return null;
    }

    /**
     * 设置用户身份信息
     */
    public void setLoginUser(BaseLoginUser loginUser) {
        if (StringUtils.isNotNull(loginUser) && StringUtils.isNotEmpty(loginUser.getToken())) {
            refreshToken(loginUser);
        }
    }

    /**
     * 删除用户身份信息
     */
    public void delLoginUser(String token) {
        if (StringUtils.isNotEmpty(token)) {
            String userKey = getTokenKey(token);
            redisCache.deleteObject(userKey);
        }
    }

    /**
     * 创建令牌
     *
     * @param loginUser 用户信息
     * @return 令牌
     */
    public String createToken(BaseLoginUser loginUser) {
        String token = IdUtils.fastUUID();
        loginUser.setToken(token);
        setUserAgent(loginUser);
        refreshToken(loginUser);

        Map<String, Object> claims = new HashMap<>();
        claims.put(Constants.LOGIN_USER_KEY, token);
        return createToken(claims);
    }

    /**
     * 验证令牌有效期，相差不足20分钟，自动刷新缓存
     *
     * @param loginUser
     * @return 令牌
     */
    public void verifyToken(BaseLoginUser loginUser) {
        long expireTime = loginUser.getExpireTime();
        long currentTime = System.currentTimeMillis();
        if (loginUser instanceof LoginTapUser) {
            // 最后6小时刷新
            if (expireTime - currentTime <= MILLIS_MINUTE_HOUR) {
                refreshToken(loginUser);
            }
        } else {
            // 最后20分钟刷新
            if (expireTime - currentTime <= MILLIS_MINUTE_TEN) {
                refreshToken(loginUser);
            }
        }

    }

    /**
     * 刷新令牌有效期
     *
     * @param loginUser 登录信息
     */
    public void refreshToken(BaseLoginUser loginUser) {
        loginUser.setLoginTime(System.currentTimeMillis());
        int localExpireTime;
        if (loginUser instanceof LoginTapUser) {
            localExpireTime = clientExpireTime;
        } else {
            localExpireTime = expireTime;
        }
        loginUser.setExpireTime(loginUser.getLoginTime() + localExpireTime * MILLIS_MINUTE);
        // 根据uuid将loginUser缓存
        String userKey = getTokenKey(loginUser.getToken());
        redisCache.setCacheObject(userKey, loginUser, localExpireTime, TimeUnit.MINUTES);
    }

    /**
     * 设置用户代理信息
     *
     * @param loginUser 登录信息
     */
    public void setUserAgent(BaseLoginUser loginUser) {
        UserAgent userAgent = UserAgent.parseUserAgentString(ServletUtils.getRequest().getHeader("User-Agent"));
        String ip = IpUtils.getIpAddr();
        loginUser.setIpaddr(ip);
        loginUser.setLoginLocation(AddressUtils.getRealAddressByIP(ip));
        loginUser.setBrowser(userAgent.getBrowser().getName());
        loginUser.setOs(userAgent.getOperatingSystem().getName());
    }

    /**
     * 从数据声明生成令牌
     *
     * @param claims 数据声明
     * @return 令牌
     */
    private String createToken(Map<String, Object> claims) {
        String token = Jwts.builder()
                .setClaims(claims)
                .signWith(SignatureAlgorithm.HS512, secret).compact();
        return token;
    }

    /**
     * 从令牌中获取数据声明
     *
     * @param token 令牌
     * @return 数据声明
     */
    private Claims parseToken(String token) {
        return Jwts.parser()
                .setSigningKey(secret)
                .parseClaimsJws(token)
                .getBody();
    }

    /**
     * 从令牌中获取用户名
     *
     * @param token 令牌
     * @return 用户名
     */
    public String getUsernameFromToken(String token) {
        Claims claims = parseToken(token);
        return claims.getSubject();
    }

    /**
     * 获取请求token
     *
     * @param request
     * @return token
     */
    private String getToken(HttpServletRequest request) {
        String token = request.getHeader(header);
        if (StringUtils.isNotEmpty(token) && token.startsWith(Constants.TOKEN_PREFIX)) {
            token = token.replace(Constants.TOKEN_PREFIX, "");
        }
        return token;
    }

    private String getTokenKey(String uuid) {
        return CacheConstants.LOGIN_TOKEN_KEY + uuid;
    }
}

```

