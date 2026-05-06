---
name: springboot-security
description: Java Spring Boot 服务中关于认证/授权、验证、CSRF、密钥、头部、速率限制和依赖安全的 Spring Security 最佳实践。
origin: ECC
---

# Spring Boot 安全审查

在添加身份验证、处理输入、创建端点或处理密钥时使用。

## 何时激活

* 添加身份验证（JWT、OAuth2、基于会话）
* 实现授权（@PreAuthorize、基于角色的访问）
* 验证用户输入（Bean 验证、自定义验证器）
* 配置 CORS、CSRF 或安全标头
* 管理密钥（Vault、环境变量）
* 添加速率限制或暴力破解防护
* 扫描依赖项以查找 CVE

## 身份验证

* 首选无状态 JWT 或带有吊销列表的不透明令牌
* 对于会话，使用 `httpOnly`、`Secure`、`SameSite=Strict` cookie
* 使用 `OncePerRequestFilter` 或资源服务器验证令牌

```java
@Component
public class JwtAuthFilter extends OncePerRequestFilter {
  private final JwtService jwtService;

  public JwtAuthFilter(JwtService jwtService) {
    this.jwtService = jwtService;
  }

  @Override
  protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
      FilterChain chain) throws ServletException, IOException {
    String header = request.getHeader(HttpHeaders.AUTHORIZATION);
    if (header != null && header.startsWith("Bearer ")) {
      String token = header.substring(7);
      Authentication auth = jwtService.authenticate(token);
      SecurityContextHolder.getContext().setAuthentication(auth);
    }
    chain.doFilter(request, response);
  }
}
```

## 授权

* 启用方法安全：`@EnableMethodSecurity`
* 使用 `@PreAuthorize("hasRole('ADMIN')")` 或 `@PreAuthorize("@authz.canEdit(#id)")`
* 默认拒绝；仅公开必需的权限范围

```java
@RestController
@RequestMapping("/api/admin")
public class AdminController {

  @PreAuthorize("hasRole('ADMIN')")
  @GetMapping("/users")
  public List<UserDto> listUsers() {
    return userService.findAll();
  }

  @PreAuthorize("@authz.isOwner(#id, authentication)")
  @DeleteMapping("/users/{id}")
  public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
    userService.delete(id);
    return ResponseEntity.noContent().build();
  }
}
```

## 输入验证

* 在控制器上使用带有 `@Valid` 的 Bean 验证
* 在 DTO 上应用约束：`@NotBlank`、`@Email`、`@Size`、自定义验证器
* 在渲染前使用白名单清理任何 HTML

```java
// BAD: No validation
@PostMapping("/users")
public User createUser(@RequestBody UserDto dto) {
  return userService.create(dto);
}

// GOOD: Validated DTO
public record CreateUserDto(
    @NotBlank @Size(max = 100) String name,
    @NotBlank @Email String email,
    @NotNull @Min(0) @Max(150) Integer age
) {}

@PostMapping("/users")
public ResponseEntity<UserDto> createUser(@Valid @RequestBody CreateUserDto dto) {
  return ResponseEntity.status(HttpStatus.CREATED)
      .body(userService.create(dto));
}
```

## SQL 注入防护

* 使用 Spring Data 存储库或参数化查询
* 对于原生查询，使用 `:param` 绑定；切勿拼接字符串

```java
// BAD: String concatenation in native query
@Query(value = "SELECT * FROM users WHERE name = '" + name + "'", nativeQuery = true)

// GOOD: Parameterized native query
@Query(value = "SELECT * FROM users WHERE name = :name", nativeQuery = true)
List<User> findByName(@Param("name") String name);

// GOOD: Spring Data derived query (auto-parameterized)
List<User> findByEmailAndActiveTrue(String email);
```

## 密码编码

* 始终使用 BCrypt 或 Argon2 哈希密码 — 切勿存储明文
* 使用 `PasswordEncoder` bean，而非手动哈希

```java
@Bean
public PasswordEncoder passwordEncoder() {
  return new BCryptPasswordEncoder(12); // cost factor 12
}

// In service
public User register(CreateUserDto dto) {
  String hashedPassword = passwordEncoder.encode(dto.password());
  return userRepository.save(new User(dto.email(), hashedPassword));
}
```

## CSRF 防护

* 对于浏览器会话应用，保持 CSRF 启用；在表单/标头中包含令牌
* 对于使用 Bearer 令牌的纯 API，禁用 CSRF 并依赖无状态身份验证

```java
http
  .csrf(csrf -> csrf.disable())
  .sessionManagement(sm -> sm.sessionCreationPolicy(SessionCreationPolicy.STATELESS));
```

## 密钥管理

* 源代码中不包含密钥；从环境变量或 Vault 加载
* 保持 `application.yml` 不含凭据；使用占位符
* 定期轮换令牌和数据库凭据

```yaml
# BAD: Hardcoded in application.yml
spring:
  datasource:
    password: mySecretPassword123

# GOOD: Environment variable placeholder
spring:
  datasource:
    password: ${DB_PASSWORD}

# GOOD: Spring Cloud Vault integration
spring:
  cloud:
    vault:
      uri: https://vault.example.com
      token: ${VAULT_TOKEN}
```

## 安全标头

```java
http
  .headers(headers -> headers
    .contentSecurityPolicy(csp -> csp
      .policyDirectives("default-src 'self'"))
    .frameOptions(HeadersConfigurer.FrameOptionsConfig::sameOrigin)
    .xssProtection(Customizer.withDefaults())
    .referrerPolicy(rp -> rp.policy(ReferrerPolicyHeaderWriter.ReferrerPolicy.NO_REFERRER)));
```

## CORS 配置

* 在安全过滤器级别配置 CORS，而非每个控制器
* 限制允许的来源 — 生产环境中切勿使用 `*`

```java
@Bean
public CorsConfigurationSource corsConfigurationSource() {
  CorsConfiguration config = new CorsConfiguration();
  config.setAllowedOrigins(List.of("https://app.example.com"));
  config.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE"));
  config.setAllowedHeaders(List.of("Authorization", "Content-Type"));
  config.setAllowCredentials(true);
  config.setMaxAge(3600L);

  UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
  source.registerCorsConfiguration("/api/**", config);
  return source;
}

// In SecurityFilterChain:
http.cors(cors -> cors.configurationSource(corsConfigurationSource()));
```

## 速率限制

* 在开销大的端点上应用 Bucket4j 或网关级限制
* 记录并告警突发流量；返回 429 并提供重试提示

```java
// Using Bucket4j for per-endpoint rate limiting
@Component
public class RateLimitFilter extends OncePerRequestFilter {
  private final Map<String, Bucket> buckets = new ConcurrentHashMap<>();

  private Bucket createBucket() {
    return Bucket.builder()
        .addLimit(Bandwidth.classic(100, Refill.intervally(100, Duration.ofMinutes(1))))
        .build();
  }

  @Override
  protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
      FilterChain chain) throws ServletException, IOException {
    String clientIp = request.getRemoteAddr();
    Bucket bucket = buckets.computeIfAbsent(clientIp, k -> createBucket());

    if (bucket.tryConsume(1)) {
      chain.doFilter(request, response);
    } else {
      response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
      response.getWriter().write("{\"error\": \"Rate limit exceeded\"}");
    }
  }
}
```

## 依赖项安全

* 在 CI 中运行 OWASP 依赖项检查 / Snyk
* 保持 Spring Boot 和 Spring Security 使用受支持的版本
* 对已知 CVE 使构建失败

## 日志记录与 PII

* 切勿记录密钥、令牌、密码或完整的 PAN 数据
* 对敏感字段进行脱敏；使用结构化 JSON 日志记录

## 文件上传

* 验证大小、内容类型和扩展名
* 存储在 Web 根目录之外；必要时进行扫描

## 发布前检查清单

* \[ ] 身份验证令牌已验证且正确过期
* \[ ] 每个敏感路径都有授权防护
* \[ ] 所有输入均已验证和清理
* \[ ] 没有字符串拼接的 SQL
* \[ ] CSRF 策略与应用类型匹配
* \[ ] 密钥已外部化；未提交任何密钥
* \[ ] 安全标头已配置
* \[ ] API 已启用速率限制
* \[ ] 依赖项已扫描并保持最新
* \[ ] 日志不包含敏感数据

**记住**：默认拒绝、验证输入、最小权限、优先通过配置实现安全。
