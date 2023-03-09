---
title: 初识Spring Security（二)
abbrlink: 48147
date: 2022-07-02 21:26:10
tags: [Java,Spring Security]
categories: Java
description: Spring Boot集成Spring Security，用户密码加密存储实现及用户登陆。
---

对于用户密码出于安全考虑需要加密存储，Spring Security提供了多种加密方式，官方推荐使用BCryptPasswordEncoder加密方式。其实BCryptPasswordEncoder的实现并非为一种加密算法，而是采用SHA-256 +随机盐+密钥对密码进行加密，SHA系列是Hash算法，其过程是不可逆的。用户注册时，使用SHA-256 +随机盐+密钥把用户输入的密码进行Hash处理，将得到的Hash值存入数据库。用户登陆时候采取同样的算法对密码进行Hash处理后于数据库中存储得密码Hash值进行比较。

## SecurityUtils工具类

Spring框架借助ThreadLocal来保存和传递用户登录信息。我们编写一个工具类方便的获取ThreadLocal中的用户信息名，以及用户密码的加密和比较工作

```Java
public class SecurityUtils {
    /**
     * 用户ID
     **/
    public static String getUserId() {
        try {
            return getLoginUser().getUserId();
        } catch (Exception e) {
            throw new BusinessException(ResultEnum.USER_NOT_EXIST);
        }
    }
    /**
     * 获取部门ID
     **/
    public static String getOrganId() {
        try {
            return getLoginUser().getOrganId();
        } catch (Exception e) {
            throw new BusinessException(ResultEnum.USER_NOT_ORGAN);
        }
    }
    /**
     * 获取用户账户
     **/
    public static String getUsername() {
        try {
            return getLoginUser().getUsername();
        } catch (Exception e) {
            throw new BusinessException(ResultEnum.USER_NOT_ACCOUNT);
        }
    }
    /**
     * 获取用户
     **/
    public static SysUser getLoginUser() {
        try {
            return (SysUser) getAuthentication().getPrincipal();
        } catch (Exception e) {
            throw new BusinessException(ResultEnum.USER_NOT_EXIST);
        }
    }
    /**
     * 获取Authentication
     */
    public static Authentication getAuthentication() {
        return SecurityContextHolder.getContext().getAuthentication();
    }

    /**
     * 生成BCryptPasswordEncoder密码
     *
     * @param password 密码
     * @return 加密字符串
     */
    public static String encryptPassword(String password) {

        return "{bcrypt}" + new BCryptPasswordEncoder().encode(password);
    }
    /**
     * 判断密码是否相同
     *
     * @param rawPassword     真实密码
     * @param encodedPassword 加密后字符
     * @return 结果
     */
    public static boolean matchesPassword(String rawPassword, String encodedPassword) {
        BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
        return passwordEncoder.matches(rawPassword, encodedPassword);
    }
    /**
     * 是否为超级管理员
     *
     * @param userId 用户ID
     * @return 结果
     */
    public static boolean isAdmin(Long userId) {
        return userId != null && 1L == userId;
    }
}
```

## 用户注册

用户注册对用户填写的密码使用**SecurityUtils.encryptPassword()**进行加密处理即可

## 用户登陆流程

1、登陆后台处理调用**authenticationManager.authenticate(new UsernamePasswordAuthenticationToken(username, password))**，该方法会去调用**UserDetailsServiceImpl.loadUserByUsername()**。  

```Java
@Override
public LoginUser authLogin(UserLoginVo userLoginVo) {
    String username = userLoginVo.getUsername();
    String password = userLoginVo.getPassword();
    // 用户验证
    Authentication authentication = null;
    try {
        // 该方法会去调用UserDetailsServiceImpl.loadUserByUsername
        authentication = authenticationManager
                .authenticate(new UsernamePasswordAuthenticationToken(username, password));
    } catch (Exception e) {
        throw new BusinessException(500,e.getMessage());
    }
    LoginUser loginUser = (LoginUser) authentication.getPrincipal();
    // recordLoginInfo(loginUser.getUserId());
    // 生成 token
    String token = JwtTokenUtil.getRefreshToken(username, null);
    // 用户信息
    loginUser.setToken(tokenHead + token);
    return loginUser;
}
```

2、自定义验证类UserDetailsService 实现Security框架UserDetailsService的接口。

```Java
@Service
public class UserDetailsServiceImpl implements UserDetailsService {
private static final Logger log = LoggerFactory.getLogger(UserDetailsServiceImpl.class);

@Autowired
private SysUserService userService;

/* @Autowired
private SysPermissionService permissionService;*/

@Override
public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
    SysUser user = userService.getUserByUsername(username);
    if (user == null) {
        log.info("登录用户：{} 不存在.", username);
        throw new BusinessException("登录用户：" + username + " 不存在");
    } else if (UserStatus.DELETED.getCode() == user.getDelFlag()) {
        log.info("登录用户：{} 已被删除.", username);
        throw new BusinessException("对不起，您的账号：" + username + " 已被删除");
    } else if (UserStatus.DISABLE.getCode() == user.getUserState()) {
        log.info("登录用户：{} 已被停用.", username);
        throw new BusinessException("对不起，您的账号：" + username + " 已停用");
    }
    return createLoginUser(user);
}

public UserDetails createLoginUser(SysUser user) {
    return new LoginUser(user.getUserId(), user.getOrganId(), user);
}
}
```

3、我们自定义验证类UserDetailsService实现类中，需要实现的**loadUserByUsername**方法回返回一个**UserDetails**接口类，包含非安全相关的信息（如用户昵称，电话号码等），们只存储用户信息，这些信息随后被封装到Authentication对象中。所以我们可以创建其实现类。

```Java
public class LoginUser implements UserDetails {
    private static final long serialVersionUID = 1L;

    /**
     * 用户ID
     */
    private String userId;

    /**
     * 部门ID
     */
    private String deptId;

    /**
     * 用户唯一标识
     */
    private String token;

    /**
     * 登录时间
     */
    private String loginTime;

    /**
     * 过期时间
     */
    private String expireTime;

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
     * 权限列表
     */
    private Set<String> permissions;

    public LoginUser() {
    }

    public LoginUser(SysUser user, Set<String> permissions) {
        this.user = user;
        this.permissions = permissions;
    }

    public LoginUser(String userId, Long String, SysUser user, Set<String> permissions) {
        this.userId = userId;
        this.deptId = deptId;
        this.user = user;
        this.permissions = permissions;
    }
    public LoginUser(String userId, String deptId, SysUser user) {
        this.userId = userId;
        this.deptId = deptId;
        this.user = user;
    }

    /**
     * 用户信息
     */
    private SysUser user;

    public String getDeptId() {
        return deptId;
    }

    public void setDeptId(String deptId) {
        this.deptId = deptId;
    }

    public String getToken() {
        return token;
    }

    public void setToken(String token) {
        this.token = token;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return null;
    }

    @Override
    @JsonIgnore
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getUsername();
    }
    /**
     * 用户是否过期，没有过期就返回true
     */
    @Override
    @JsonIgnore
    public boolean isAccountNonExpired() {
        return true;
    }
    /**
     * 用户是否被锁定，锁定返回true。
     */
    @Override
    @JsonIgnore
    public boolean isAccountNonLocked() {
        return true;
    }
    /**
     * 用户凭证是否可用，可用返回true
     */
    @Override
    @JsonIgnore
    public boolean isCredentialsNonExpired() {
        return true;
    }
    /**
     * 用户是否启用了，启用了返回true
     */
    @Override
    @JsonIgnore
    public boolean isEnabled() {
        return true;
    }
}
```

## 常见错误

- There is no PasswordEncoder mapped for the id “null“ 问题的解决方法  
  
  - 错误  
      登陆报错**There is no PasswordEncoder mapped for the id “null“**
  
  - 原因  
      Spring Security5.x 对所配置的密码必须带上加密方式，如果没有带，就会解析不出来，所以抛错。
  
  - 解决 
      储存密码是添加加密方式， 格式为{xxx}密码。  
    
    | 加密方式    | 原来security 4的密码格式 | 现在security 5的密码格式 |
    | ------- | ----------------- | ----------------- |
    | bcrypt  | password          | {bcrypt}password  |
    | ldap    | password          | {ldap}password    |
    | MD4     | password          | {MD4}password     |
    | MD5     | password          | {MD5}password     |
    | noop    | password          | {noop}password    |
    | pbkdf2  | password          | {pbkdf2}password  |
    | scrypt  | password          | {scrypt}password  |
    | SHA-1   | password          | {SHA-1}password   |
    | SHA-256 | password          | {SHA-256}password |
    | sha256  | password          | {sha256}password  |
