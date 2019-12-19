---
layout: post
comments: true
title: Spring OAuth2 사용시 cors 설정
tags: [spring, security, cors]
---

### CORS  

Cross Origin Resource Sharing(CORS)는 이름에서 알 수 있듯이 Origin (출처)를 교차하여 자원을 공유한다는 뜻으로, 도메인혹은 포트가 다른 서비스의 자원을 요청하는 내용을 담고 있습니다.  
일반적으로 자바스크립트에서는 Same origin policy (동일 출처 정책) 보안 정책을 가지고있어, 다른 Origin에 접근시 `No Access-Control-Allow-Origin Header`라는 오류를 노출하게 되어 있습니다.  
  
그렇다면 해당 CORS 오류를 해결하기위한 방법은 무엇이 있을까요?   

- 동일한 도메인 서버에서 자원을 요청한다.   
가장 간단한 방법으로, 보안정책을 따르는 방법입니다. 요청하려는 자원서버를 동일한 도메인으로 통일한다면 CORS 보안정책을 목적상 안전한 origin으로 판단하기때문에, 이 오류는 나지 않을것 입니다.  
하지만 이미 이 오류가 나고있다는것은 도메인이 분리되어있는 뜻 일텐데, 다른방법을 알아보도록 하겠습니다.   

- 서버에서 origin을 설정해준다.  
서버에서 요청받을때, 해당 Client의 Origin에대해서 허가해주는 정책을 추가해줍니다.  

---

### Spring boot cors 설정  

```java
// WebConfig.java

@Configuration
public class WebConfig extends WebMvcConfigurerAdapter {
 
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedMethods("*");
    }
}
```

```java
// SecurityConfig.java

@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.cors();
    }
 
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        final CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(ImmutableList.of("*"));
        configuration.setAllowedMethods(ImmutableList.of("HEAD","GET", "POST", "PUT", "DELETE", "PATCH"));
        configuration.setAllowCredentials(true);
        configuration.setAllowedHeaders(ImmutableList.of("Authorization", "Cache-Control", "Content-Type"));
        final UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
}
```

일반적으로 위와같은 설정을 통해서 Spring security + cors을 설정 할 수 있습니다.   
하지만 OAuth2를 사용한다면, 위와 같은 설정이라면 OPTIONS method 로 들어오는 pre-flight call 에서 토큰인증이 실패하여 401 오류와 함께 CORS 오류가 일어나게 됩니다.   
  
이는, /check/token 과정을통해 security context 설정 후 request matcher를 하는 과정이 진행되는데 이보다 더 앞단계에서 토큰헤더를 포함하지 않는 OPTIONS method에 대해 검증해제 로직이 들어가야하는데, SecurityConfig는 그보다 더 늦게 로직이 동작하는것으로 파악됩니다.  
  
OAuth2를 함께 사용할때 CORS를 설정하는 방법은 다음과 같습니다.  

---

### Spring CORS with OAuth2  

OAuth2를 사용할때 CORS 설정을 할수 있는 방법은 두가지 방법이 있습니다.  

- Cors filter 구현  

```java
@Component
@Order(Ordered.HIGHEST_PRECEDENCE)
public class CorsFilter implements Filter {
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
        throws IOException, ServletException
    {
        HttpServletResponse response = (HttpServletResponse) res;
        HttpServletRequest request = (HttpServletRequest) req;
        response.setHeader("Access-Control-Allow-Origin", "*");
        response.setHeader("Access-Control-Allow-Credentials", "true");
        response.setHeader("Access-Control-Allow-Methods","*");
        response.setHeader("Access-Control-Max-Age", "3600");
        response.setHeader("Access-Control-Allow-Headers",
            "Origin, X-Requested-With, Content-Type, Accept, Key, Authorization");
 
        if ("OPTIONS".equalsIgnoreCase(request.getMethod())) {
            response.setStatus(HttpServletResponse.SC_OK);
        } else {
            chain.doFilter(req, res);
        }
    }
 
    public void init(FilterConfig filterConfig) {
        // not needed
    }
 
    public void destroy() {
        //not needed
    }
}
```

Filter를 implement 받은 corsFilter를 직접 구현하여 최상단에서 Cors 설정을 해 줄수 있습니다.  

- ResourceServerConfig 구현  

```java
@EnableResourceServer
@Configuration
public class ResourceServerConfig extends ResourceServerConfigurerAdapter
{
    @Override
    public void configure(HttpSecurity http) throws Exception
    {
        http.authorizeRequests()
            .antMatchers(HttpMethod.OPTIONS).permitAll()
            .antMatchers("/api/**").hasAnyRole("USER", "ADMIN")
            .anyRequest().authenticated();
    }
}
```

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer
{
    @Override
    public void addCorsMappings(CorsRegistry registry)
    {
        registry
            .addMapping("/**")
            .allowedMethods("*");
    }
}
```

---

### 결론

OAuth2 인증 사용시 CORS에 걸리지 않으려면 OAuth2 token 인증 보다 앞단계의 필터/인터셉터에서 path 검증로직을 일어나야합니다. 이를 위한 두가지 방법으로는 @Order(Ordered.HIGHEST_PRECEDENCE) 를 통해 custom cors filter를 작성해주는 방법과 ResourceServerConfig에서 설정을 해주는방법이 있습니다.  

