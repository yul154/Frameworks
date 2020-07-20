# Spring Security
> authentication and authorization for web
---------------------------------------------
# Spring Security Architecture
<img width="428" alt="Screen Shot 2019-08-22 at 6 15 45 PM" src="https://user-images.githubusercontent.com/27160394/63553592-e55d0680-c508-11e9-976c-6db66ed88bdd.png">

# Authentication
## Configuration
<img width="412" alt="Screen Shot 2019-08-22 at 6 21 03 PM" src="https://user-images.githubusercontent.com/27160394/63553821-aa0f0780-c509-11e9-90f0-574d1be43af5.png">

## 1 dependencies in `pom.xml`
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```
## 2 Spring Security 5 Configuration 
### 2.1 `web.xml`
* Context Loader Listener
  1. creates a root web-application-context for the web-application and puts it in the ServletContext.
* Config Location: Locating the security config file at
* Application Entry Point: define a filter
* In Spring Boot 2, if we want our own security configuration, we can simply add a custom WebSecurityConfigurerAdapter. This will disable the default auto-configuration and enable our custom security configuration.
```
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE xml>  
    <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee  
         http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"  
         version="3.1">  
          
        <!-- Spring Configuration -->  
        <servlet>  
            <servlet-name>spring</servlet-name>  
            <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>  
            <load-on-startup>1</load-on-startup>  
        </servlet>  
        <servlet-mapping>  
            <servlet-name>spring</servlet-name>  
            <url-pattern>/</url-pattern>  
        </servlet-mapping>  
          
        <listener>  
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>  
    </listener>  
      
    <filter>  
        <filter-name>springSecurityFilterChain</filter-name>  
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>  
    </filter>  
    <filter-mapping>  
        <filter-name>springSecurityFilterChain</filter-name>  
        <url-pattern>/*</url-pattern>  
    </filter-mapping>  
          
        <context-param>  
            <param-name>contextConfigLocation</param-name>  
            <param-value>  
                /WEB-INF/spring-servlet.xml  
                /WEB-INF/spring-security.xml  
            </param-value>  
        </context-param>  
</web-app>  
```
### 2.2 java Configuration
* `@ConditionalOnClass`: allows configuration to be included based on the presence or absence of specific classes
```
@Configuration
@EnableWebSecurity
@ConditionalOnClass(some.class)
public class SecurityConfig extends WebSecurityConfigurerAdapter {
 
    @Autowired
    PasswordEncoder passwordEncoder;
 
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
        .passwordEncoder(passwordEncoder)
        .withUser("user").password(passwordEncoder.encode("123456")).roles("USER")
        .and()
        .withUser("admin").password(passwordEncoder.encode("123456")).roles("USER", "ADMIN");
    }
 
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
 
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
        .antMatchers("/login").permitAll()
        .antMatchers("/admin/**").hasRole("ADMIN")
        .antMatchers("/**").hasAnyRole("ADMIN", "USER")
        .and().formLogin()
        .and().logout().logoutSuccessUrl("/login").permitAll()
        .and().csrf().disable();
    }
}
```
### 3 Initialize spring security
* Bind spring security to web application
#### 3.1.1 web.xml configuration 
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app>
    <filter>
        <filter-name>springSecurityFilterChain</filter-name>
        <filter-class>
            org.springframework.web.filter.DelegatingFilterProxy
        </filter-class>
    </filter>

    <filter-mapping>
        <filter-name>springSecurityFilterChain</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app> 
```
#### 3.1.2 Java configuration 
* `AbstractSecurityWebApplicationInitializer`will register `DelegatingFilterProxy`
```
import org.springframework.security.web.context.AbstractSecurityWebApplicationInitializer;
public class SpringSecurityInitializer extends AbstractSecurityWebApplicationInitializer {
    //no code needed
}
```
* register the `DelegatingFilterProxy` to use the `springSecurityFilterChain` before any other registered Filter
```
import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;
 
public class AppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
 
   @Override
   protected Class<?>[] getRootConfigClasses() {
      return new Class[] { HibernateConfig.class, SecurityConfig.class };
   }
 
   @Override
   protected Class<?>[] getServletConfigClasses() {
      return new Class[] { WebMvcConfig.class };
   }
 
   @Override
   protected String[] getServletMappings() {
      return new String[] { "/" };
   }
}
```
### 3.2 Components
### 3.2.1 DispatcherServlet
### 3.2.2 DelegatingFilterProxy
* Proxy for a standard Servlet Filter, delegating to a Spring-managed bean that implements the Filter interface
* in order to make a bridge/link between web application and spring security filter chain
* it does not do ant real work and it delegates the request to the spring managed bean called “springSecurityFilterChain” 
* forward all requests to othe security filters to perform authentication or authorization checks
#### 3.2.3 `FilterChainProxy`(`springSecurityFilterChain`)
> 
* a single filter that chains together multiple additional filters.
* When it receives a request, it iterates through all the SecurityFilterChains in order to find one that matches the request
* `DelegatingFilterProxy` is to delegate to it.
* lets us add a single entry to configuration file
* deal entirely with the application context file for managing our web security beans.
* `boolean matches(HttpServletRequest request)`: checks if the request applies to this filter chain
* `List<Filter> getFilters()`: return the list of security filters for this chain
```
```
####  Filter interface
* `init(FilterConfig filterConfig)`
> Called by the web container to indicate to a filter that it is being placed into service.
* `doFilter(ServletRequest request, ServletResponse response, FilterChain chain) `
> doFilter(ServletRequest request, ServletResponse response, FilterChain chain).
* `destroy()`:
> Called by the web container to indicate to a filter that it is being taken out of service.
#### 3.2.3.1 Filter Chain
#### a.SecurityContext
* `SecurityContext` holds details of the authenticated principal(username, roles...)
* `SecurityContext` can be set up in the `SecurityContextHolder` at the beginning of a web request
* `SecurityContextHolder` is a helper class, which provide access to the security context.
* `SecurityContextHolder`. uses a ThreadLocal object to store security context,hich means that the security context is always available to methods in the same thread of execution,
* any changes to the `SecurityContext` can be copied to the `HttpSession` when the web request ends
* `SecurityContextHolder` can be modified to contain a valid Authentication request token
* A `SecurityContextRepository` implementation which stores the security context in the HttpSession between requests.
```
public interface SecurityContext extends Serializable {
    Authentication getAuthentication();
    void setAuthentication(Authentication authentication);
}
```
#### b. `SercurityContextPersistenceFilter`
> manage security context 
* This filter will only execute once per request
* This filter MUST be executed BEFORE any authentication processing mechanisms.
* it uses `SecurityContextRepository` to obtain the context which should be used for the current thread of execution 
#### c.`AuthenticationFilters`
<img width="436" alt="Screen Shot 2019-08-22 at 8 05 01 PM" src="https://user-images.githubusercontent.com/27160394/63557787-27417900-c518-11e9-9043-7682d22993cd.png">.
* The filter will extract credentials from the request and create authentication request token
* The filter delegates authentication to an `AuthenticationManager` Which will return authenticate token to filer
* verify an authentication request is to load the corresponding UserDetails and check the loaded password against the one that has been entered by the user.
* after authentication is successful the filter places the authenticated principle into the security context
* All future requests don't need re-authernticate unless the session expires or user logs out
##### c.1 `AuthenticationManager`
* The main interface which provides authentication services in Spring Security 
* The default implementation of `AuthenticationManager`in Spring Security is called `ProviderManager`
* `ProviderManager` delegates to a list of configured AuthenticationProvider
* `ProviderManager` will attempt to clear any sensitive credentials information from the Authentication object which is returned by a successful authentication request.
##### c.2  `AuthenticationProvider`
* Each provider will either throw an exception or return a fully populated Authentication object
* `UserDetails`  represents a principal, but in an extensible and application-specific way.
* It receieved `UserDetails` from `UserDetailsServie`, it will match the credentials against the request
* load the corresponding UserDetails and check the loaded password against the one that has been entered by the user.
* after authentication is successful, it will return a new authenticated token back to the manager
* Only one provider in application
```
public interface AuthenticationProvider {
    
    Authentication authenticate(Authentication authentication)
            throws AuthenticationException;
    
    boolean supports(Class<?> authentication);# Returns true if this AuthenticationProvider supports the indicated Authentication object.
    
}
```
#### c.3 Authentication
* interface
```
public interface Authentication extends Principal, Serializable{
  boolean isAuthenticated();
  Object getPrincipal();
  Object getCredentials;
  Collection<? exntends GrantedAuthority>getAuthorities();
  }
```
* token
<img width="313" alt="Screen Shot 2019-09-04 at 2 32 21 PM" src="https://user-images.githubusercontent.com/27160394/64281294-d3c02980-cf20-11e9-85fd-52689278d796.png">

#### c.4 UserDetails
* It's interface which store user information which is later encapsulated into Authentication objects.
```
public interface UserDetails extends Serializable {
         Collection<? extends GrantedAuthority> getAuthorities();
         String getPassword();
         String getUsername();
         boolean isAccountNonExpired();  
         boolean isAccountNonLocked();
         boolean isCredentialsNonExpired();
         boolean isEnabled();
    }

```
#### c.5 UserDetailsService
* It's a interface which is used to retrieve the user’s authentication and authorization information. 
* It has a single read-only method named as `loadUserByUsername()` which locate the user based on the username.
```
UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
```

#### d.`AnonymousAuthenticationFilter`
* automatically adds an AnonymousAuthenticationToken to the SecurityContextHolder
* There is a corresponding AnonymousAuthenticationProvider, which is chained into the ProviderManager 

#### e. `ExceptionTranslationFilter`
* catch any Spring Security exceptions so that either an HTTP error response can be returned or an appropriate AuthenticationEntryPoint can be launched

#### f. `FilterSecurityInterceptor`
* Performs authorization

### recap Authentication Filter
1. AuthenticationFilter generate an authentication request and delegate the request to an AnthenticationManager
2. AnthenticationManager delegates to AuthenticationProviders
3. The AuthenticationProvides use UserDetailsService to retrieve credentilas from an identity store
4. The AuthenticationProvides return authenticated principal back to the filter which add it to SecurityContext

### 4 Credentials
#### 4.1 Handlin credentials
* Spring support LADP,JDBC
* Configuring JDBC Authentication
```
public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        auth.jdbcAuthentication()
        .datasource(this.datasource)
        .withDefaultSchema()
        .withUser(User.withUsername("user")
        .password(passwordEncoder().encode("pass"))
        .roles("USER"));;
}
```
#### 4.2 Password encoding
* `PasswordEncoder`:Service interface for encoding passwords. The preferred implementation is BCryptPasswordEncoder.
* Need to expose the encoder as a bean in confoguration class, so can autowire in service

#### 4.3 Registration
* By using `antMatchers()`, Registration page doesn't require a user to be authenticated. We specified multiple URL patterns that any user can access
```
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()                                                                
            .antMatchers("/resources/**", "/signup", "/about").permitAll()                  
            .antMatchers("/admin/**").hasRole("ADMIN") //Any URL that starts with "/admin/" will be resticted to users who have the role "ROLE_ADMIN".                                     
            .antMatchers("/db/**").access("hasRole('ROLE_ADMIN') and hasRole('ROLE_DBA')")//Any URL that starts with "/db/" requires the user to have both "ROLE_ADMIN" and "ROLE_DBA"
            .anyRequest().authenticated()                                                   
            .and()
        // ...
        .formLogin();
}`
```
* Validate content
    1. create a new @interface to define our annotation:
    2. create a validator

```
@Documented
@Constraint(validatedBy = UsernameValidator.class)
@Target( { ElementType.METHOD, ElementType.FIELD })
@Retention(RetentionPolicy.RUNTIME)
public @interface Username {
    String message() default "Username already exists";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```
#### 4.4 Securing secrets
* Sprin vault

## Additonal Authentication Layer
### 1 Multi-factor authentication
> requires more than one method of authentication from independent categories of credentials to verify the user's identity for a login or other transaction.
#### 1.1 Spring Security events--Email Verfication
* add dependency
```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```
* Add email server in property
* create the Event and EventListener in order to send the email on successful registration of the user.
* create controller to handle verification

##### 1.2 two-factor verification

## Remember me
* sending an additional remember-me cookies to the user's browser
* Spring support two flavors, hash-bashed token and persistent token
### 1. hash-based token
*  the basic configuration using the `rememberMe()` method 
*  Hash Token cookie contains the following data:
```
base64(username + ":" + expirationTime + ":" +
md5Hex(username + ":" + expirationTime + ":" password + ":" + key))

username:          As identifiable to the `UserDetailsService`
password:          That matches the one in the retrieved UserDetails
expirationTime:    The date and time when the remember-me token expires, expressed in milliseconds
key:               A private key to prevent modification of the remember-me token
```
* this has a potential security issue in that a captured remember-me token will be usable from any user agent until such time as the token expires
### 2. Persisitent Token
> uses a database or other persistent storage mechanism to store the generated tokens.
* Token exists in the database
* cookies only contain a series identifier and token value(changes each time a user using remember-me cookies)
#### 2.1set up
1. create persistent_logins table
2. provide JDBC token repository with data source
3. add it to the remember me filter builder

## Improve log out
```
.and().logout().logoutUrl("/logout").logoutSuccessUrl("/login").deleteCookies("remember-me")
```

## Outsourcing Authentication with OpenID/OAuth2
### 1. OAuth2
* In cofiguration class, set up oauth2Login authentication filter
<img width="397" alt="Screen Shot 2019-09-05 at 2 55 38 PM" src="https://user-images.githubusercontent.com/27160394/64370847-44347c80-cfed-11e9-96d9-39b1ed43cebd.png">
-------------------------------.

#  Common threats
## Default Security Headers
* Spring Security allows users to easily inject the default security headers to assist in protecting their application
* This policy helps prevent attacks such as Cross Site Scripting (XSS) and other code injection attacks by defining content sources which are approved and thus allowing the browser to load them
```
Cache-Control: no-cache, no-store, max-age=0, must-revalidate# Instruct the brower to disable all caching completely
Pragma: no-cache
Expires: 0
X-Content-Type-Options: nosniff# preventing the brower from trying to guess the content type of a request
Strict-Transport-Security: max-age=31536000 ; includeSubDomains
X-Frame-Options: DENY#  whether or not a browser should be allowed to render a page in a <frame> , <iframe> , <embed> or <object> 
X-XSS-Protection: 1; mode=block # stops pages from loading when they detect reflected cross-site scripting (XSS) attacks 
```
## CSRF
* The issue is that the HTTP request from the Browser and the request from the evil website are exactly the same
* To protect against CSRF attacks we need to ensure there is something in the request that the evil site is unable to provide.
* Synchronizer Token Pattern: ensure that each request requires, in addition to our session cookie, a randomly generated token as an HTTP parameter.
* Only use the CSRF Token fro state changing operastions(POST,PUT,DELETE,PATCH)
## Configuration
```
@EnableWebSecurity
@Configuration
public class WebSecurityConfig extends
   WebSecurityConfigurerAdapter {

  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.csrf().disable();
  }
}
```
```
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
        // ...
        .headers().disable();
    }
}
```
##  HSTS(Http Strict Transport Security)
* a malicious user could intercept the initial HTTP request and manipulate the response
* Many users omit the https protocol and this is why HTTP Strict Transport Security (HSTS) was created.
```
@EnableWebSecurity
@Configuration
public class WebSecurityConfig extends
   WebSecurityConfigurerAdapter {

  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http
       ...
      .headers()
        .hsts();
  }
}
```

# Authorization
* ExceptionTranslationFilter: It will invoke the appropriate exception handler 
## 1. Flow

1. the request make its way down the filter chain until it reaches `FilterSercurityInterceptor` whici final filter in the chain
2. `FilterSecurityIntercepot` will call the SecurityMetadataSource to retrieve the security metadata of protfolio URL
3. The SecurityMetadataSource will return the config attribute
4. The interceptor filter will also retrieve the authenticated principle from the security context
5. and then it will call the `AccessDecisionManager` to decide if the request has access to that portfolio URL
6. The manager then ask one or more `AccessDecisionVoters` for whether access should be granted
7. The `AccessDecisionVoters` return their votes to manager who make the final decision

<img width="435" alt="Screen Shot 2019-09-05 at 3 45 46 PM" src="https://user-images.githubusercontent.com/27160394/64375648-40f0bf00-cff4-11e9-9fae-dc041aaf1a28.png">

## 2.Components
### 2.1 `AccessDecisionVoter`
```
@Service
public interface AccessDecisionVoter<Entity> {

    boolean supports(ConfigAttribute attribute) {        
    }

    boolean supports(Class<?> clazz) {
    }
    int vote(Authentication authentication, Entity someEntity,
            Collection<ConfigAttribute> config) {
    }
}
```
* Concrete implementations return an int
* A voting implementation will return ACCESS_ABSTAIN if it has no opinion on an authorization decision. 
* If it does have an opinion, it must return either ACCESS_DENIED or ACCESS_GRANTED.
### 2.2 `AccessDecisionManager`

```
public interface AccessDecisionManager{
    void decide(Authentication authentication, Object secureObject,
        Collection<ConfigAttribute> attrs) 
        throws AccessDeniedException;

    boolean supports(ConfigAttribute attribute);
    boolean supports(Class clazz);
}
```
* Spring Security includes several AccessDecisionManager implementations that are based on voting
<img width="274" alt="Screen Shot 2019-09-05 at 3 39 55 PM" src="https://user-images.githubusercontent.com/27160394/64375214-6cbf7500-cff3-11e9-8913-5b2aa6f01aeb.png">

## 3. SpEL
* supports querying and manipulating an object graph
* within XML-based and annotation-based cofiguration and bean definitions

## 4. URL-level authorization
* to authorize requests to URLs, use authorizeRequest builder
* use `anMacthers` to define different authorization strategies for URLs
* Generally mvcMatcher is more secure than an antMatcher. As an example:
    1. `antMatchers("/secured")` matches only the exact /secured URL
    2. `mvcMatchers("/secured")` matches /secured as well as /secured/, /secured.html, /secured.xyz
```
protected void configure(HttpSecurity http) throws Exception {
    http
        .authorizeRequests()                                                                
            .antMatchers("/resources/**", "/signup", "/about").permitAll()                  
            .antMatchers("/admin/**").hasRole("ADMIN")                                      
            .antMatchers("/db/**").access("hasRole('ROLE_ADMIN') and hasRole('ROLE_DBA')")  
            .antMatchers(HttpMethod.POST,"/support.admin/**").hasRole("ADMIN")
            .anyRequest().authenticated()                                                   
            .and()
        // ...
        .formLogin();
}
```
## 5. Method-level authorization
* to enable method-level authorization, you need to add @EnableGlobaleMethodSecurity annotation
```
@EnableGlobalMethodSecurity(
  prePostEnabled = true, //enables Spring Security pre/post annotations
  securedEnabled = true, //determines if the @Secured annotation should be enabled
  jsr250Enabled = true)// allows us to use the @RoleAllowed annotation
```
* The `@PreAuthorize` annotation checks the given expression before entering the method
* the `@PostAuthorize` annotation verifies it after the execution of the method and could alter the result.
* `@Secured`:used to specify a list of roles on a method
* `@PreFilter` annotation to filter a collection argument before executing the method:
* `@PostFilter` annotation to filter the returned collection of a method 
```
@PreFilter
  (value = "filterObject != authentication.principal.username",
  filterTarget = "usernames")
```
##  6. Domain-level authorization
* `hasPermission`: 
    * hasPermission() expressions are delegated to an instance of PermissionEvaluator
    * It is intended to bridge between the expression system and Spring Security’s ACL system
```
public boolean hasPermission(Authentication authentication, Object targetDomainObject, Object permission) 
```
* PermissionEvaluator

```
public class CustomPermissionEvaluator implements PermissionEvaluator {

    private Logger log = LoggerFactory.getLogger(CustomPermissionEvaluator.class);

    @Override
    public boolean hasPermission(Authentication authentication, Object targetDomainObject, Object permission) {
        CustomUserDetails customUserDetails = (CustomUserDetails) authentication.getPrincipal();
        AbstractEntity abstractEntity = (AbstractEntity) targetDomainObject;
        log.debug("User {} trying to access {}-{} with permission {}",
                customUserDetails.getUsername(),
                abstractEntity.getClass().getSimpleName(),
                abstractEntity.getId(),
                permission.toString());
        return false;
    }

    @Override
    public boolean hasPermission(Authentication authentication, Serializable targetId, String targetType, Object permission) {
        CustomUserDetails customUserDetails = (CustomUserDetails) authentication.getPrincipal();
        log.debug("User {} trying to access {}-{} with permission {}",
                customUserDetails.getUsername(),
                targetType,
                targetId,
                permission.toString());
        return false;
    }
}

@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class MethodSecurityConfig extends GlobalMethodSecurityConfiguration {
 
    @Override
    protected MethodSecurityExpressionHandler createExpressionHandler() {
        DefaultMethodSecurityExpressionHandler expressionHandler = 
          new DefaultMethodSecurityExpressionHandler();
        expressionHandler.setPermissionEvaluator(new CustomPermissionEvaluator());
        return expressionHandler;
    }
}
```
* Spring Access Control List:
    * Currently only supports relational databases
    * helps in defining permissions for specific user/role on a single domain object – instead of across the board,
    * need to create four mandatory tables in our database: ACL_SID,ACL_CLASS,ACL_OBJECT_IDENTITY,ACL_ENTRY
