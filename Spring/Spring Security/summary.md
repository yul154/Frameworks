DelegatingFilterProxy
* to enable Spring Security, you just have to register the Spring Security filter, DelegatingFilterProxy. The DelegatingFilterProxy will forward all requests to other Spring Security filters to perform  authentication or authorization checks
* in order to make a bridge/link between web application and spring security filter chain
* it does not do ant real work and it delegates the request to the spring managed bean called “springSecurityFilterChain”

FilterChainProxy(springSecurityFilterChain)
* a single filter that chains together multiple additional filters.
* When it receives a request, it iterates through all the SecurityFilterChains in order to find one that matches the request
* springSecurityFilterChain: a collection of filters

how authentication requests are handleds
* there are many filters that do a multitude of security tasks
1. SecurityContextpersistencefilter: which store authenticated principal like username, role. This filter will only execute once per request,This filter MUST be executed BEFORE any authentication processing mechanisms.
2. authenticationfilters(basic, digest, Open ID):The filter will extract credentials from the request and create authentication request token, once you authenication is sucessful, the filter place put authenticated principal into the security context, so all furture request don't need re-authenc
3. remembermeauthenticationfilter
4. anonymousauthenticationfilter:add anomumousauthentication object to securitycontext
5. exceptiontranslationfilter: Handles any AccessDeniedException and AuthenticationException thrown within the filter chain.

authentication filter type
* basic:
* open:
* digest: encrypted 
* usernamepassword:

1. authentication Filter   extract credentials from the request and create an authentication request token
2. The filter delegates authentication to authentication manager which return an authenticated principal token to filter
3. authentication manager delegate many authentication provider
4. Authentication provider need to retrieve (userdetailservice)the principle details from an identity store(database)
5. Once authentication provide receive the user details, it will match the credentials and return a new authenticated token back to manager



Common security threats

Inject secrity  helps prevent attacks such as CSRF(user unwant actions )Cross Site Scripting (XSS) and other code injection attacks


Oauth2
 To speed up the registration process and simplify authentication for the user, many sites outsource authentication to third parties, like Facebook, Google, or Okta. This is done through OAuth. 




# Spring Security
* Session
```
http
 .sessionManagement()
 .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
```
## Spring authentication components
* SecurityContext
* SecurityContextHolder
* Authentication: It's a interface
* UserDetails
* UserDetailsService: LoadUserByUsername
* AuthenticationManager: returning a fully populated Authentication object (including granted authorities) if successful.
## Spring Authorization Components
## Spring Authentication flow
## Spring security filter types
* DelegatingFilterProxy: look through the Spring Application Context for a bean “springSecurityFilterChain”.
* springSecurityFilterChain: 
  * Spring Security’s web infrastructure should only be used by delegating to an instance of FilterChainProxy. The security filters should not be used by themselves.
  * Entrace of filters is SpringSecurityFilterChain
 
