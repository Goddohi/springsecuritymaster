정말 막 적는 곳

# 개념
- SecurityBuilder
  - 빌더 클래스로서 웹 보안을구성하는 빈 객체와 설정클래스들을 생성하은 역할
    - 대표적으로 webSecurity, HttpSecurity
- SecurityConfigurer 
  - Http요청과 관련된 본안처리를 담당하는 필터들을 생성하고 여러 초기화 설정에 관여
    - 여기서 필터는 스프링시큐리티는 모든 인가 인증은 필터에서 진행한다생각하면 좋다.
      - 대표적으로 SecurityContextConfigurer, HttpBasicConfigurer 등등(FormLogin,Csrf,ExceptionGandling,Anonymous)
- 여기서 SecurityBuilder는 SecurityConfigurer를 참조하고 있고
  - 인증 및 인가 초기화 작업은SecurityConfigurer에 의해서 진행

흐름

AutoConfiguration(자동생성) -> 1. 빌더 클래스 생성 -> SecurityBuilder ->  2.설정클래스 생성 -> SecurityConfigurer ->   3.초기화작업 진행 ( 필터, 등등등 )

조금 더 디테일한 내용
```
SecurityBuilder                 SecurityConfigurer  
   |                                              init(B builder)
HTTPSecurity   -- 설정 클래스 생성--> d여러 Configurer - - - - -  -  - - - -  - -> 여러 필터
                                                    configure(B builder)
``` 



- HttpSecurityConfiguration 뜯어보기
```java
    @Bean({"org.springframework.security.config.annotation.web.configuration.HttpSecurityConfiguration.httpSecurity"})
    @Scope("prototype")
    HttpSecurity httpSecurity() throws Exception {
        LazyPasswordEncoder passwordEncoder = new LazyPasswordEncoder(this.context);
        AuthenticationManagerBuilder authenticationBuilder = new DefaultPasswordEncoderAuthenticationManagerBuilder(this.objectPostProcessor, passwordEncoder);
        authenticationBuilder.parentAuthenticationManager(this.authenticationManager());
        authenticationBuilder.authenticationEventPublisher(this.getAuthenticationEventPublisher());
        HttpSecurity http = new HttpSecurity(this.objectPostProcessor, authenticationBuilder, this.createSharedObjects());
        WebAsyncManagerIntegrationFilter webAsyncManagerIntegrationFilter = new WebAsyncManagerIntegrationFilter();
        webAsyncManagerIntegrationFilter.setSecurityContextHolderStrategy(this.securityContextHolderStrategy);
        http
                // 
                .csrf(Customizer.withDefaults()) // 해당 설정에서 csrfconfigurer가 있다 -> 쭉쭉 가보면 SecurityConfigurer 
                
                .addFilter(webAsyncManagerIntegrationFilter)
                .exceptionHandling(Customizer.withDefaults())
                .headers(Customizer.withDefaults())
                .sessionManagement(Customizer.withDefaults())
                .securityContext(Customizer.withDefaults())
                .requestCache(Customizer.withDefaults())
                .anonymous(Customizer.withDefaults())
                .servletApi(Customizer.withDefaults())
                .apply(new DefaultLoginPageConfigurer());
        http.logout(Customizer.withDefaults());
        this.applyCorsIfAvailable(http);
        this.applyDefaultConfigurers(http);
        return http;
    }
이렇게 빈객체를 생성하는데
빈객체를 받아서 초기화 하는 대표적인 곳은 SpringBootWebSecurityConfiguration
```

그후 filter

(필터후 필터가 필요한 핸들러 등등도 추가하고 그런다)

