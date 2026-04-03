# Spring Security App

A demonstration application showcasing Spring Security authentication and authorization features.

## Overview

This application demonstrates fundamental Spring Security concepts including form-based authentication, role-based access control, and security configuration best practices. It serves as a reference implementation for securing Spring Boot applications.

## Features

- Form-based authentication
- In-memory user authentication
- Role-based access control (RBAC)
- Custom login/logout pages
- Session management
- CSRF protection
- Password encoding
- Method-level security
- REST API security

## Technology Stack

- **Spring Boot 2.2.1**
- **Spring Security** - Authentication and authorization
- **Spring Web** - REST API framework
- **Spring Boot Starter Test** - Testing framework
- **Spring Security Test** - Security testing utilities

## Configuration

### Application Properties
```properties
server.port=8083
spring.application.name=spring-security-app

# Security Configuration
spring.security.user.name=admin
spring.security.user.password=admin
spring.security.user.roles=ADMIN

# Session Configuration
server.servlet.session.timeout=30m

# Logging
logging.level.org.springframework.security=DEBUG
```

## Security Configuration

### Main Security Configuration
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
            .withUser("user")
            .password("{noop}password")
            .roles("USER")
            .and()
            .withUser("admin")
            .password("{noop}admin")
            .roles("ADMIN");
    }
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/", "/home", "/public/**").permitAll()
                .antMatchers("/admin/**").hasRole("ADMIN")
                .antMatchers("/user/**").hasAnyRole("USER", "ADMIN")
                .anyRequest().authenticated()
                .and()
            .formLogin()
                .loginPage("/login")
                .defaultSuccessUrl("/dashboard")
                .permitAll()
                .and()
            .logout()
                .logoutSuccessUrl("/login?logout")
                .permitAll()
                .and()
            .csrf().disable(); // Only for demonstration
    }
}
```

## API Endpoints

### Public Endpoints

#### Home Page
```http
GET http://localhost:8083/
```

#### Login Page
```http
GET http://localhost:8083/login
```

#### Public Resources
```http
GET http://localhost:8083/public/info
```

### User Endpoints (Requires USER or ADMIN role)

#### User Dashboard
```http
GET http://localhost:8083/user/dashboard
```

#### User Profile
```http
GET http://localhost:8083/user/profile
```

#### User Settings
```http
GET http://localhost:8083/user/settings
```

### Admin Endpoints (Requires ADMIN role)

#### Admin Dashboard
```http
GET http://localhost:8083/admin/dashboard
```

#### User Management
```http
GET http://localhost:8083/admin/users
```

#### System Settings
```http
GET http://localhost:8083/admin/settings
```

### Authentication Endpoints

#### Login (POST)
```http
POST http://localhost:8083/login
Content-Type: application/x-www-form-urlencoded

username=user&password=password
```

#### Logout
```http
POST http://localhost:8083/logout
```

## Running the Application

### Prerequisites
- Java 8+
- Maven 3.3+

### Development Mode
```bash
mvn spring-boot:run
```

### Production Build
```bash
mvn clean package
java -jar target/spring-security-app-0.0.1-SNAPSHOT.jar
```

## User Credentials

### Default Users
- **User**: `user` / `password` (ROLE_USER)
- **Admin**: `admin` / `admin` (ROLE_ADMIN)

### Access Levels
- **ROLE_USER**: Access to user-specific endpoints
- **ROLE_ADMIN**: Full access to all endpoints

## Security Features Demonstration

### 1. Form-Based Authentication
- Custom login page
- Remember me functionality
- Login failure handling
- Redirect after login

### 2. Role-Based Access Control
- Method-level security annotations
- URL-based access control
- Role hierarchy support

### 3. Session Management
- Session timeout configuration
- Concurrent session control
- Session fixation protection

### 4. CSRF Protection
- CSRF token generation
- Token validation
- Custom CSRF configuration

## Advanced Security Configuration

### Password Encoding
```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication()
        .withUser("user")
        .password(passwordEncoder().encode("password"))
        .roles("USER");
}
```

### Method-Level Security
```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class MethodSecurityConfig extends GlobalMethodSecurityConfiguration {
}

@Service
public class UserService {
    
    @PreAuthorize("hasRole('ADMIN')")
    public void adminMethod() {
        // Admin-only method
    }
    
    @PreAuthorize("hasAnyRole('USER', 'ADMIN')")
    public void userMethod() {
        // User and admin method
    }
}
```

### Custom Authentication Provider
```java
@Component
public class CustomAuthenticationProvider implements AuthenticationProvider {
    
    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        String username = authentication.getName();
        String password = authentication.getCredentials().toString();
        
        // Custom authentication logic
        if (customUserValidation(username, password)) {
            return new UsernamePasswordAuthenticationToken(
                username, password, Collections.singletonList(new SimpleGrantedAuthority("ROLE_USER")));
        } else {
            throw new BadCredentialsException("Authentication failed");
        }
    }
    
    @Override
    public boolean supports(Class<?> authentication) {
        return authentication.equals(UsernamePasswordAuthenticationToken.class);
    }
}
```

## Testing

### Unit Tests
```bash
mvn test
```

### Security Testing
```java
@SpringBootTest
@AutoConfigureMockMvc
public class SecurityTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Test
    public void publicEndpointAccessible() throws Exception {
        mockMvc.perform(get("/"))
            .andExpect(status().isOk());
    }
    
    @Test
    public void userEndpointRequiresAuthentication() throws Exception {
        mockMvc.perform(get("/user/dashboard"))
            .andExpect(status().is3xxRedirection());
    }
    
    @Test
    @WithMockUser(roles = "USER")
    public void userEndpointAccessibleWithUserRole() throws Exception {
        mockMvc.perform(get("/user/dashboard"))
            .andExpect(status().isOk());
    }
    
    @Test
    @WithMockUser(roles = "ADMIN")
    public void adminEndpointAccessibleWithAdminRole() throws Exception {
        mockMvc.perform(get("/admin/dashboard"))
            .andExpect(status().isOk());
    }
}
```

### Integration Testing with cURL
```bash
# Access public endpoint
curl http://localhost:8083/

# Attempt to access protected endpoint (should redirect)
curl -L http://localhost:8083/user/dashboard

# Login and access protected endpoint
curl -c cookies.txt -X POST http://localhost:8083/login \
  -d "username=user&password=password"

curl -b cookies.txt http://localhost:8083/user/dashboard
```

## Development Notes

### Security Best Practices
1. **Never disable CSRF** in production
2. **Use strong password encoding** (BCrypt)
3. **Implement proper logout** handling
4. **Secure session management**
5. **Use HTTPS** in production
6. **Regular security updates**

### Common Security Mistakes
1. Hardcoding credentials in code
2. Disabling security features for convenience
3. Not validating input properly
4. Insufficient logging and monitoring
5. Not implementing proper session management

### Debugging Security Issues
```properties
# Enable debug logging
logging.level.org.springframework.security=DEBUG
logging.level.org.springframework.web=DEBUG

# Enable request debugging
logging.level.org.springframework.web.filter=DEBUG
```

## Customization Examples

### Custom Login Page
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Login</title>
</head>
<body>
    <form th:action="@{/login}" method="post">
        <div>
            <label>Username:</label>
            <input type="text" name="username"/>
        </div>
        <div>
            <label>Password:</label>
            <input type="password" name="password"/>
        </div>
        <div>
            <input type="submit" value="Login"/>
        </div>
        <div th:if="${param.error}">
            <p>Invalid username and password.</p>
        </div>
        <div th:if="${param.logout}">
            <p>You have been logged out.</p>
        </div>
    </form>
</body>
</html>
```

### Custom Success Handler
```java
@Component
public class CustomAuthenticationSuccessHandler implements AuthenticationSuccessHandler {
    
    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, 
                                      HttpServletResponse response,
                                      Authentication authentication) 
                                      throws IOException, ServletException {
        // Custom logic after successful login
        response.sendRedirect("/dashboard");
    }
}
```

## Monitoring and Logging

### Security Event Logging
```java
@Component
public class SecurityAuditLogger implements ApplicationListener<AuthenticationSuccessEvent> {
    
    private static final Logger logger = LoggerFactory.getLogger(SecurityAuditLogger.class);
    
    @Override
    public void onApplicationEvent(AuthenticationSuccessEvent event) {
        String username = event.getAuthentication().getName();
        logger.info("User {} logged in successfully", username);
    }
}
```

### Security Metrics
```java
@RestController
public class SecurityMetricsController {
    
    private final MeterRegistry meterRegistry;
    
    public SecurityMetricsController(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
    }
    
    @EventListener
    public void handleAuthenticationSuccess(AuthenticationSuccessEvent event) {
        meterRegistry.counter("security.authentication.success").increment();
    }
    
    @EventListener
    public void handleAuthenticationFailure(AuthenticationFailureBadCredentialsEvent event) {
        meterRegistry.counter("security.authentication.failure").increment();
    }
}
```

## Integration with Other Services

### OAuth2 Integration (Future Enhancement)
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```

### JWT Token Support (Future Enhancement)
```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```

## Dependencies

Key dependencies and their versions:
- Spring Boot Starter Web
- Spring Boot Starter Security
- Spring Boot Starter Test
- Spring Security Test
- Spring Boot DevTools (optional)
