# 3강. Security Config 클래스

- 스프링 시큐리티 의존성을 추가해두면 요청을 가로챔. 그리고 다음을 검증함.
    - 접근 권한이 있는 지
    - 로그인 되어 있는 지
    - role을 갖고 있는 지

---

- SecurityConfig: 인가 작업 커스텀

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration // 스프링 부트에서 config 파일 인식
@EnableWebSecurity // 시큐리티 기능을 활성화
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests((auth) -> auth
                .requestMatchers("/","login").permitAll()
                .requestMatchers("/admin").hasRole("ADMIN")
                .requestMatchers("/my/**").hasAnyRole("ADMIN","USER")
                .anyRequest().authenticated()
        );
        return http.build();
    }
}
```

- permitAll → 모든 사용자에게 오픈할 url
- hasRole → Role이 있어야 접근 가능한 url
- authenricated → 로그인 해야만 접근 가능한 url
- denyAll → 모든 사용자의 접근을 막을 url

---

- Q. 위의 예시 코드에 따르면 "/","login"는 로그인이나 Role 없이도 접근 가능한 url인 거고, "/admin", "/my/**"는 로그인이 필요하고, Role도 필요한 url인 거고, "/space"는 로그인이 필요하고, Role은 필요없는 url이라고 볼 수 있을까요?
    
    ### 각 URL 접근 조건 해석
    
    1. **`"/", "login"` → `permitAll()`**
        - 로그인 여부, Role 여부 전혀 상관없이 접근 가능
        - 예: 메인 페이지(`/`), 로그인 페이지(`/login`)
    2. **`"/admin"` → `hasRole("ADMIN")`**
        - 반드시 **로그인**이 되어 있어야 하고
        - 로그인된 사용자의 권한(Role)이 `ROLE_ADMIN` 이어야 접근 가능
    3. **`"/my/**"` → `hasAnyRole("ADMIN","USER")`**
        - 반드시 **로그인** 필요
        - 권한이 `ROLE_ADMIN` 또는 `ROLE_USER` 중 하나여야 접근 가능
        - `/my/profile`, `/my/settings` 같은 모든 하위 경로 포함
    4. **`anyRequest().authenticated()`**
        - 위에서 명시하지 않은 모든 요청에 적용
        - 로그인만 되어 있으면 접근 가능
        - 별도의 Role은 필요 없음
    
    
    ## 따라서 `/space`는?
    
    - 위에서 직접 매칭한 패턴이 없으므로 → `anyRequest().authenticated()` 규칙 적용
    - 즉, **로그인만 필요하고, Role은 필요 없는 URL** ✅
    
    
    ## 👉 요약:
    
    - `permitAll()` → 로그인 필요 없음
    - `authenticated()` → 로그인만 필요
    - `hasRole("X")` → 로그인 + 특정 Role 필요
    - `hasAnyRole("X","Y")` → 로그인 + 여러 Role 중 하나 필요
