# 5강. **BCrypt 암호**

- 스프링 시큐리티는 로그인 시 비밀번호에 대해 암호화한 후 DB에 저장된 (이미 암호화 되어있는) 비밀번호와 대조한다.

```java
// SecurityConfig에 BCryptPasswordEncoder Bean 추가하기
@Bean
public BCryptPasswordEncoder bCryptPasswordEncoder() {
    return new BCryptPasswordEncoder();
}
```

