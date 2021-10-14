# Catalog
1. Autoconfiguring Spring Security 
2. Defining custom user storage 
3. Customizing the login page
4. Securing against CSRF attacks 
5. Knowing your user
---
##  4.2 Configuring Spring Security
```
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

}
```
 Spring Security offers several options for configuring a user store
* An in-memory user store
* A JDBC-based user store
* An LDAP-backed user store 
* A custom user details service

 Configure user store by overriding a `configure()` method defined in the `WebSecurityConfigurerAdapter` configuration base class.
 ```
@Override
protected void configure(AuthenticationManagerBuilder auth)
throws Exception {
 ```
 ### 4.2.1 In-memory user store
 > Keep user information in memory

Using in memory when only a handful of users, none of which are likely to change
```
@Override
	protected void configure(AuthenticationManagerBuilder auth)
	    throws Exception {
		auth.inMemoryAuthentication()
				.withUser("username")
				.password("password")
				.authorities("ROLE_USER")
				.and()
				.withUser("another user")
				.password("pw")
				.authorities("ROLE_USER");	
	}
```
### 4.2.2 JDBC-based user store
> User information is often maintained in a relational database

* Must also set the *DataSource* so that it knows how to access the database
```
@Autowired
    DataSource dataSource;

	@Override
    protected void configure(AuthenticationManagerBuilder auth)
        throws Exception {
		auth .jdbcAuthentication().dataSource(dataSource);
	}
```
* SQL queries that will be performed when looking up user details
```
auth.jdbcAuthentication().dataSource(dataSource)
		.usersByUsernameQuery(
			    "select username, password, enabled from Users " + "where username=?") 
		.authoritiesByUsernameQuery(
			    "select username, authority from UserAuthorities " +"where username=?");
```

WORKING WITH ENCODED PASSWORDS
Specify a password encoder by calling the `passwordEncoder()` method
```
.authoritiesByUsernameQuery( "select username, authority from UserAuthorities " +"where username=?")
.passwordEncoder(new StandardPasswordEncoder("53cr3t");
```
The `passwordEncoder()` method accepts any implementation of Spring Security’s `PasswordEncoder` interface. 
* `BCryptPasswordEncoder`—Applies bcrypt strong hashing encryption
*  `NoOpPasswordEncoder`—Applies no encoding
* `Pbkdf2PasswordEncoder`—Applies PBKDF2 encryption
* `SCryptPasswordEncoder`—Applies scrypt hashing encryption
* `StandardPasswordEncoder`—Applies SHA-256 hashing encryption

Customerize `PasswordEncoder` interface
```
public interface PasswordEncoder {
  String encode(CharSequence rawPassword);
  boolean matches(CharSequence rawPassword, String encodedPassword);
}
```
### 4.2.3 LDAP-backed user store
> Configure Spring Security to rely on another common source of user data
* LDAP is a lightweight version of the Directory Access Protocol (DAP)
* LDA authentication accomplishes this goal by storing data in the LDAP directory and authenticating users to access the directory
```
@Override
	protected void configure(AuthenticationManagerBuilder auth) throws Exception {
			auth
				.ldapAuthentication() 
				.userSearchBase("ou=people") 
        .userSearchFilter("(uid={0})") 
        .groupSearchBase("ou=groups") 
        .groupSearchFilter("member={0}");
	}
```
* The `userSearchFilter()` and `groupSearchFilter()` methods are used to provide filters for the base LDAP queries, which are used to search for users and groups.
* The `userSearchBase()` method provides a base query for finding users. Likewise, the `groupSearchBase()` method specifies the base query for finding groups.

**CONFIGURING PASSWORD COMPARISON**
The default strategy for authenticating against LDAP 
1. To perform a bind operation, authenticating the user directly to the LDAP server. 
2. Another option is to perform a comparison operation. Sending the entered password to the LDAP directory and asking the server to compare the password against a user’s password attribute
```
.passwordCompare()
.passwordEncoder(new BCryptPasswordEncoder()) 
.passwordAttribute("passcode");
```
**REFERRING TO A REMOTE LDAP SERVER**
By default, Spring Security’s LDAP authentication assumes that the LDAP server is listening on port 33389 on localhost
* if your LDAP server is on another machine. use the `contextSource()` method to configure the location:
```
.contextSource()
.url("ldap://tacocloud.com:389/dc=tacocloud,dc=com");
```
**CONFIGURING AN EMBEDDED LDAP SERVER**
specify the root suffix for the embed- ded server via the `root()` method:
```
.contextSource() .root("dc=tacocloud,dc=com");
```
When the LDAP server starts
* it will attempt to load data from any LDIF files that it can find in the classpath. 
* LDIF (LDAP Data Interchange Format) is a standard way of representing LDAP data in a plain text file.
* Each record is composed of one or more lines, each containing a name:value pair.
```
.ldif("classpath:users.ldif");
```
### 4.2.4 Customizing user authentication
Leverage the Spring Data repository used to store users.
1. Create the domain object and repository interface that represents and persists user information.
  *  Implementations of `UserDetails` will provide some essential user information to the framework,
  *  The `getAuthorities()` method should return a collection of authorities granted to the user. 
  *  The various `is___Expired()` methods return a boolean to indicate whether or not the user’s account is enabled or expired.
  *  defines a `findByUsername()` method that you’ll use in the user details ser- vice to look up a User by their username.
2. write a custom user details service that uses this repository. 
```
public interface UserDetailsService {
  UserDetails loadUserByUsername(String username)
                     throws UsernameNotFoundException;
}

```
3. Use  UserRepository in a custom UserDetailsService implementation
```
@Service
public class UserRepositoryUserDetailsService implements UserDetailsService {
  private UserRepository userRepo;
  @Autowired
  public UserRepositoryUserDetailsService(UserRepository userRepo) {this.userRepo = userRepo; }
  @Override
  public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
      User user = userRepo.findByUsername(username); 
      if (user != null) { return user; }
      throw new UsernameNotFoundException( "User '" + username + "' not found");
  }
}
```
4. Configure your custom user details service with Spring Security in configure() method
```

@Override
protected void configure(AuthenticationManagerBuilder auth)
    throws Exception {  auth .userDetailsService(userDetailsService );}
```
**REGISTERING USERS**
```
@Controller
@RequestMapping("/register")
public class RegistrationController {
	private UserRepository userRepo;
	private PasswordEncoder passwordEncoder;
	
	public RegistrationController(UserRepository userRepo, PasswordEncoder passwordEncoder) {
		this.userRepo = userRepo;
		this.passwordEncoder = passwordEncoder; }
	
	@GetMapping
	 public String registerForm() {
	    return "registration";
	 }
	
	@PostMapping
	public String processRegistration(RegistrationForm form) {
	userRepo.save(form.toUser(passwordEncoder));
	    return "redirect:/login";
	 }
}
```
*  a GET request for `/register` will be handled by the register- Form() method, which simply returns a logical view name of registration.
*  
---
## 4.3 Securing web requests
The homepage, login page, and registra- tion page should be available to unauthenticated users
* To configure these security rules, `WebSecurityConfigurerAdapter’s `other `configure()` method
```
@Override
        protected void configure(HttpSecurity http) throws Exception {}
```
Security rules
1. Requiring that certain security conditions be met before allowing a request to be served
2. Configuring a custom login page
3. Enabling users to log out of the application
4. Configuring cross-site request forgery protection

### 4.3.1 Securing requests

You need to ensure that requests for `/design` and `/orders` are only available to authenticated users; all other requests should be permitted for all users.
```
protected void configure(HttpSecurity http) throws Exception {
  http .authorizeRequests()
       .antMatchers("/design", "/orders") 
       .hasRole("ROLE_USER")
       .antMatchers(“/”, "/**").permitAll() ;
}

```
* The order of these rules is important. Security rules declared first take precedence over those declared lower down.
* essential security rules for request handling are self-limiting, only enabling security rules as defined by those methods. Alternatively, you can use the `access()` method to provide a SpEL expression to declare richer security rules
```
http .authorizeRequests()
    .antMatchers("/design", "/orders") 
    .access("hasRole('ROLE_USER') && " +  "T(java.util.Calendar).getInstance().get("+ "T(java.util.Calendar).DAY_OF_WEEK) == " + "T(java.util.Calendar).TUESDAY")
     .antMatchers(“/”, "/**").access("permitAll") ;
```
### 4.3.2 Creating a custom login page
To replace the built-in login page, you first need to tell Spring Security what path your custom login page will be at. That can be done by calling `formLogin()`
```
.and() .formLogin().loginPage("/login")
```
* Spring Security listens for login requests at `/login` and expects that the username and password fields be named username and password. 
```
.formLogin()
.loginPage("/login") 
.defaultSuccessUrl("/design")
.loginProcessingUrl("/authenticate") 
.usernameParameter("user") .passwordParameter("pwd")
```
### 4.3.3 Logging out

This sets up a security filter that intercepts POST requests to `/logout`.
```
.and() 
.logout()
.logoutSuccessUrl("/")
```
### 4.3.4 Preventing cross-site request forgery
> Cross-site request forgery (CSRF) : subjecting a user to code on a maliciously designed web page that automatically (and usually secretly) submits a form to another application on behalf of a user who is often the victim of the attack.

To protect against such attacks, applications can generate a CSRF token upon displaying a form
* place that token in a hidden field, and then stow it for later use on the server.
* When the form is submitted, the token is sent back to the server along with the rest of the form data.
* The request is then intercepted by the server and compared with the token that was originally generate

Spring Security has built-in CSRF protection. And it’s enabled by default and you don’t need to explicitly configure it
* only need to make sure that any forms your application submits include a field `named _csrf` that contains the CSRF token.
```
<input type="hidden" name="_csrf" th:value="${_csrf.token}"/>
```
---
## 4.4 Knowing your user
* The` @ManyToOne` annotation on this property indicates that an order belongs to a single user, conversely, that a user may have many orders. 

There are several ways to determine who the user is.
* Inject a Principal object into the controller method
* Inject an Authentication object into the controller method 
*  Use SecurityContextHolder to get at the security context
* Use an @AuthenticationPrincipal annotated method
```
@PostMapping
public String processOrder(@Valid Order order, Errors errors, SessionStatus sessionStatus, Principal principal) {User user = userRepository.findByUsername( principal.getName());

User user = userRepository.findByUsername( principal.getName());

order.setUser(user);

@PostMapping
public String processOrder(@Valid Order order, Errors errors, SessionStatus sessionStatus, Authentication authentication) {}
User user = (User) authentication.getPrincipal(); 
order.setUser(user);

@PostMapping
public String processOrder(@Valid Order order, Errors errors, SessionStatus sessionStatus, @AuthenticationPrincipal User user) {
    if (errors.hasErrors()) { return "orderForm" }
    order.setUser(user);
    orderRepo.save(order); 
    sessionStatus.setComplete();
    return "redirect:/";
    
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
User user = (User) authentication.getPrincipal();
```

