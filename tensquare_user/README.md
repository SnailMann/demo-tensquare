## tensquare-user

用户模块微服务

### 注意

- 关于消息中间件的队列需要人为在MQ后台创建
- 仅仅使用Spring Security的BCrypt加密功能，对密码进行加密，认证鉴权方面，我们使用JWT去实现

### 要做的事情

- 用户分普通用户和管理员用户
- 用户可以被关注
- 使用短信来接收验证码
- 用户注册

### 技术实现

#### Spring Security

- 因为只使用Security进行加密，所以我们需要覆盖默认拦截配置，开放所有路径，不拦截，关闭csrf拦截
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {

        /**
         * authorizeRequests是Security权限认证注解配置的开端，表明开启权限及说明执行什么路径所需的权限
         * 权限认证分为两个部分：
         * (1) 第一部分是，要拦截的路径
         * (2) 第二部分是，执行拦截路径所需要的是什么权限
         * 我们这里仅仅是想用Spring Security对密码进行加密，而不是使用Security进行认证和鉴权，所以前提就是要先让
         * Security对所有的路径保持开发，并且关闭csrf功能，允许跨域
         */
        http
                .authorizeRequests()
                .antMatchers("/**").permitAll() //所有路径，全允许通过，不需要权限
                .anyRequest().authenticated() //代表任何请求，都需要认证后，才能访问，既需要登录认证
                .and().csrf().disable();
    }
}
```

- 对密码进行BCrypt算法加密
```java
    /**
     * 用于对密码进行BCrypt算法加密
     *
     * @return
     */
    @Bean
    BCryptPasswordEncoder bCryptPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
```
- 对同样的密码进行加密，最终的结果是不同的，因为salt也是随机的

#### JWT权限认证