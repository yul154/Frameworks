# Catalog
* Presenting model data in the browser 
* Processing and validating form input 
* Choosing a view template library
---
## 2.1 Displaying information
### 2.1.1 Establishing the domain
> An application’s domain is the subject area that it addresses the ideas and concepts that influence the understanding of the application.
 The Domain layer typically consists of a domain model.
 * A domain model is a representation of the data storage types required by the business logic.
 * It describes the various domain objects (entities), their attributes, roles, and relationships
 * Usually a domain folder contains POJOs (Plain Old Java Objects)
`@Data` annotation at the class level is provided by Lombok make classes only define fields.

`@RequiredArgsConstructor` generates a constructor with 1 parameter for each field 

### 2.1.2 Creating a controller class
Controllers primary job
* Handle HTTP requests 
* Hand a request off to a view to render HTML (browser-displayed)
* Write data directly to the body of a response (RESTful)

```
@Slf4j
@Controller
@RequestMapping("/design")
public class DesignTacoController {
	
	@GetMapping
	public  String showDesignForm(Model model) {
		List<Ingredient> ingredients = Arrays.asList(
		new Ingredient("FLTO", "Flour Tortilla", Type.WRAP),
		new Ingredient("CARN", "Carnitas", Type.PROTEIN),
		new Ingredient("TMTO", "Diced Tomatoes", Type.VEGGIES), new Ingredient("LETC", "Lettuce", Type.VEGGIES),
		new Ingredient("CHED", "Cheddar", Type.CHEESE));
		
		Type[] types = Ingredient.Type.values();
		for (Type type : types) {
			model.addAttribute(type.toString().toLowerCase(), 
					filterByType(ingredients, type));
			}
		model.addAttribute("design", new Taco());
		
		 return "design";

	}

}
```
*  `@Slf4j` is a Lombok-provided annotation that, at runtime, will automatically generate an SLF4J Logger in the class.
* `@GetMapping ` == `@RequestMapping(method=RequestMethod.GET)` -> when an HTTP GET request is received for `/design`, will be called to handle the request.
* prefer to only use `@RequestMapping` at the class level to specify the base path.
* Use the more specific `@GetMapping, @PostMapping,` on each of the handler methods.
* `Model` is an object that ferries data between a controller and whatever view is charged with rendering that data.
* Returning *"design"*, which is the logical name of the view that will be used to render the model to the browser.

### 2.1.3 Designing the view
Spring offers several great options for defining views, including JSP, Thymeleaf, FreeMarker, Mustache, and Groovy-based templates.

View libraries are unable to work with the data in Model,But they can work with servlet request attributes.
* Before Spring hands the request over to a view, it copies the model data into request attributes 
```
<p th:text="${message}">placeholder message</p>
```
* `th:text` attribute is a Thymeleaf-namespaced attribute that performs the replacement
* The `${}` operator tells it to use the value of a request attribute ("message", in this case).
* `th:each`, that iterates over a collection of elements, rendering the HTML once for each item in the collection.
---
## 2.2 Processing form submission
 use `@PostMapping` to handle POST requests.
 ```
 @PostMapping
	public String processDesign(Design design) {
		log.info("processing design...");
		return "redirect:/orders/current";
	}
 ```
 *  Most request-handling methods conclude by returning the logical name of a view
 *  it indicates that after `processDesign()` completes, the user’s browser should be redirected to the relative `path/order/current`.
---
## 2.3 Validating form input
> Spring supports Java’s Bean Validation API (`spring-boot-starter-validation`)

### 2.3.1 Declaring validation rules
```
import javax.validation.constraints.NotNull; 
import javax.validation.constraints.Size;

@Data
public class Taco {
	@NotNull
	@Size(min=5, message="Name must be at least 5 characters long")
	private String name;
	@Size(min=1, message="You must choose at least 1 ingredient")
	private List<String> ingredients;

}
```
* `@NotNull` :must be not null. An empty value, however, is perfectly legal.

*  `@Size`: to declare those validation rules.
```
public class Order {
	@NotBlank(message="Name is required")
	 private String name;
	@NotBlank(message="Street is required")
     private String street;
	@NotBlank(message="City is required")
     private String city;
	@NotBlank(message="State is required")
     private String state;
	@NotBlank(message="Zip is required")
     private String zip;
	@CreditCardNumber(message="Not a valid credit card number")
     private String ccNumber;
	@Pattern(regexp="^(0[1-9]|1[0-2])([\\/])([1-9][0-9])$", message="Must be formatted MM/YY")
     private String ccExpiration;
	@Digits(integer=3, fraction=0, message="Invalid CVV")
     private String ccCVV;

}
```
* `@NotBlank`: must be not null, and the trimmed length must be greater than zero.
* `@Pattern` annotation, providing it with a regular expression that ensures that the property value adheres to the desired format.
* `@Digits` ensure that the value con- tains exactly three numeric digits.
* `@CreditCardNumber` declares that the property’s value must be a valid credit card number that passes the Luhn algorithm check

### 2.3.2 Performing validation at form binding
Specifying that validation should be performed in respective handler methods by add `@Valid` annotation.

```
Public String processDesign(@Valid Taco design, Errors errors){
  if (errors.hasErrors()) return "design";
  log.info("Processing design: " + design);
}
```
* If there are any validation errors, the details of those errors will be captured in an `Errors` object that’s passed into method
* Asking its `hasErrors()` method if there are any validation errors. If there are, the method concludes without processing the Taco and returns the "design" view 

### 2.3.3 Displaying validation errors
Thymeleaf offers convenient access to the `Errors` object via the `fields` property and with its `th:errors` attribute.
```
<span class="validationError"
th:if="${#fields.hasErrors('ccNumber')}"
              th:errors="*{ccNumber}">CC Num Error</span>
```
* `th:if` attribute to decide whether or not display the `<span>.`
* The `fields `property’s `hasErrors() `method checks if there are any errors in the ccNumber field.If so, the <span> will be rendered.
* assuming there are errors in ccNumber field which The th:errors attribute references. it will replace the placeholder of the `<span>` element with the validation message.
---
## 2.4 Working with view controllers
> A view controller that does nothing but forward the request to a view.
```
@Configuration
public class WebConfig implements WebMvcConfigurer  {
	
	 @Override
     public void addViewControllers(ViewControllerRegistry registry) {
		 	registry.addViewController("/").setViewName("home");
	 }

}
```
* `WebMvcConfigurer` defines several methods for configuring Spring MVC.
*  A `ViewControllerRegistry` that you can use to register one or more view controllers.
* View controllers can be used to handle HTTP GET requests for which no model data or processing is required.
---
## 2.5 Choosing a view template library
### 2.5.1 Caching templates
By default, templates are only parsed once, when they’re first used, and the results of that parse are cached for subsequent use.
  
 there’s a way to disable caching there’s a way to disable caching  -> `spring.thymeleaf.cache = false`
  
