

# Key component 

Oauth2:It is an open standard for token-based authentication and authorization on the Internet. It allows an end user's account information to be used by third-party services, 

The client application is the application requesting access to the resources stored on the resource server. 

The resource owner is the person or application that owns the data that is to be shared

Authorization server – The server issuing access tokens to the client after successfully authenticating the resource owner and obtaining authorization.


Resource Server: The server hosting the protected resources, capable of accepting and responding to protected resource requests using access tokens.


# Oauth protocol flow

1. The application requests authorization to access service resources from the user
2. If the user authorized the request, the application receives an authorization grant
3. The application requests an access token from the authorization server (API) by presenting authentication of its own identity, and the authorization grant
4. The authorization server verifies the identity of a resource owner (the user) and provides the tokens。
5. The application requests the resource from the resource server (API) and presents the access token for authentication
6. The resource server asks the authorization server for the user identity associated with the token and whether the token is valid or not. The authorization server makes a call to the database server to retrieve the user info and token details.
7. If the access token is valid, tThe Resource Server provides the protected resources to the client based on the access token.


# Workflow

1. Spring Security handles the Authentication part and Spring Security OAuth2 handles the Authorization part.

## 1. Create the Authorization server
* `@EnableAuthorizationServer`: configure and enable the OAuth 2.0 Authorization Server and enables the different endpoints needed to interact with an authorization server.
* `class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter`

* Client Registration: 
  1.clientId: the client id
  2. secret: the client secret
  3. scope: The scope to which the client is limited
  4. authorizedGrantTypes:·`authorizedGrantTypes("password", "authorization_code", "refresh_token", "implicit")`
  5. authorities: Authorities that are granted to the client (regular Spring Security authorities)

* In the second configure (AuthorizationServerEndpointsConfigurer endpoints) method, we set a few properties (token store, user approvals, and AuthenticationManager) of the Authorization Server endpoints

## 2. Create The Resource Server

> A Resource Server serves resources that are protected by the OAuth2 token. Spring OAuth2 provides a Spring Security authentication filter that implements this protection.

* `@EnableResourceServer`: enables a Spring Security filter that authenticates requests via an incoming OAuth2 token.
* `class ResourceServerConfig extends ResourceServerConfigurerAdapter `

### 2.2 Setup Spring Security:

* Create a class SecurityConfig: `@EnableWebSecurity` and `WebSecurityConfigurerAdapter` work together to provide security to our application.
  1. AuthenticationManager: Authenticates the request
  2. TokenStore: Stores the OAuth2 tokens in memory
  3. TokenStoreUserApprovalHandler: Remembers the approval decisions by consulting existing tokens
  4. TokenApprovalStore: An ApprovalStore that works with an existing TokenStore

*  `@EnableGlobalMethodSecurity` ： enables Spring Security global method security,Set the `prePostEnable` to true to enable Spring Security’s pre post annotations

* Register SpringSecurityFilterChain: create a class `AbstractSecurityWebApplicationInitializer`,ensure springSecurityFilterChain gets registered for every URL in our application.
* Spring Security uses a chain of filters, which will intercept the request, detect authentication, and redirect to authentication entry point or pass the request to authorization service.

## 3. Creating the account-service project

* The @EnableOAuth2Sso will enable configuration for an OAuth2 client in a web application that uses Spring Security and wants to use the Authorization Code Grant from our auth-service and 
* create a WebSecurityConfigurerAdapter with all paths secured.
* the @EnableFeignClients scans for interfaces that declare they as feign clients.








# JWT
> standard for creating JSON-based access tokens that contain some number of claims.

* In order to confirm if they are valid we only need to look at the token itself.
* JWTs can be sent through the HTTP Authorization headers or URI query parameters.

* JWT is just a string that consists of three parts separated by dots: 
  1. Header:  the type of the token
  2. Payload: contains the claims, which statements about an entity (typically, the user) and additional data
  3. Signature: The signature is used to verify that the sender of the JWT, in the case of tokens signed with a private key,,verify the message wasn't changed along the way

## Advantage
* The Access token has no useful information about the user. Resource Server can't locally verfiy it
* In a Microservices world, each service runs an independent application. the service will perform another request to authorization server to check the token
* The JWT contains all information necessary to validate the user on the token itsel，more efficient than looking up an access token in a database to determine who it belongs to and whether it is valid.

## how to implement
* created the JwtTokenStore and JwtAccessTokenConverter beans
* The JdbcTokenStore from Spring Security OAuth2.0 stores token data in a database. The JSON Web Token (JWT) version of the store (JwtTokenStore) encodes all the data about the grant into the token itself. The JwtTokenStore doesn’t persist any data.



# Summary
## @EnableResourceServer vs @EnableOAuth2Sso
* The @EnableResourceServer annotation enables our application to behave as a Resource Server 
* The @EnableOAuth2Sso annotation transforms our application into an OAuth2 client
*  a simple Zuul application that is going to use @EnableOAuth2Sso annotation. It's going to be responsible for authenticating users (with the help of an Authorization Server) and delegate incoming requests to other applications

## How is Security mechanism implemented using Spring?
 Spring makes use of the DelegatingFilterProxy for implementing security mechanisms. It is a Proxy for standard Servlet Filter, delegating to a Spring-managed bean that implements the Filter interface. Its the starting point in the springSecurityFilterChain which instantiates the Spring Security filters according to the Spring configuration
Create a class extends AbstractSecurityWebApplicationInitializer, it will load the springSecurityFilterChain automatically.

## What is OAuth2 Authorization code grant type? 
* Authorization code grant
* Implicit grant
* Resource owner credentials grant
* Client credentials grant
* Refresh token grant

## How to configure Spring Security using Spring Boot?
* adding the spring boot security starter dependency 
* create a new class SecurityConfig that extends the WebSecurityConfigurerAdapter and add @EnableWebSecurity

##  How to do authentication against database tables using Spring Boot Security?
1. In Memory Configuration :authenticationMgr.inMemoryAuthentication()
2. Database Authentication :  auth.jdbcAuthentication().dataSource(dataSource);

## What is the use of Spring Boot Security AuthenticationHandler class?
* redirect different users to different pages depending on the roles assigned to the users.
* we might want users with role USER to be redirected to the welcome page, while users with role ADMIN to be redirected to the add employee page.

## What's the difference between @Secured and @PreAuthorize in spring security?
* If you wanted to do something like access the method only if the user has Role1 and Role2 then you would have to use @PreAuthorize or @Secured
* The real difference is that @PreAuthorize can work with Spring Expression Language (SpEL)

## Annotations
* @PreFilter annotation to filter a collection argument before executing the method:
* @PostFilter:filter the returned collection of a method

## SSL and handshake
* The SSL or TLS handshake enables the SSL or TLS client and server to establish the secret keys with which they communicate.
