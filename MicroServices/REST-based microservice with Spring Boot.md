# Annotations
* `@RestController`:
  * a convenience annotation that combines `@Controller` and `@ResponseBody`
* Swagger:a single annotation @EnableSwagger2
* `@RequestMapping`:  map the request URI to the handler method. 
  * produces:Restrict controller support only one format of Accpet
  * consumes:Restrict controller support only one format of content type
  * `@GetMapping`: `@RequestMapping(method = RequestMethod.GET)`.
* `@PathVariable`: indicates that a method parameter should be bound to a URI template variable
* `@ResponseEntity` : represent the entire HTTP response
*  `@ExceptionHandler`: map custom exceptions on specific status codes
* `RestTemplate`: Rest Clients are good to test our rest web service but most of the times, we need to invoke rest services through our program
* `@LoadBalanced `: Explicitly request the load-balanced template with Ribbon built-in


# Secure a REST API
* `@EnableWebSecurity`: register the Spring security filter is by annotating our config class
* Security configurations: creating a config class that extends `WebSecurityConfigurerAdapter`,annotating it with `@EnableWebSecurity`
* OAuth 2.0 uses the password only for the initial login
* The OAuth login will return 2 tokens
* Future requests have to be executed with the first token (Access Token)
* if the access token expires, the client can assure that the user is logged in permanently by using the refresh token.
* JWT (JSON Web Tokens) extension on top of OAuth allows the authorization without storing the tokens
