# Catalog
1.  Creating REST
2.  Consuming REST services
3.  Sending messages asychronously
4.  Integrating Spring
---

# 1 Creating REST service
## 1.1Writing RESTful controllers
> writing a few new Spring MVC controllers that expose backend functionality with REST endpoints to be consumed by a rich web frontend.
### 1.1.1 Retrieving data from the server
```
@RestController
@RequestMapping(path="/design",
                produces="application/json")
@CrossOrigin(origins="*")
public class DesignTacosController {
	
	private TacoRepository tacoRepo;
	
	 @Autowired
	  EntityLinks entityLinks;

	 public DesignTacoController(TacoRepository tacoRepo) { 
		 this.tacoRepo = tacoRepo;
	 }
	 
	 @GetMapping("/recent")
	 public Iterable<Taco> recentTacos() {
		 PageRequest page = PageRequest.of(
				 0, 12, Sort.by("createdAt").descending());
	 return tacoRepo.findAll(page).getContent(); }

}
```
The `@RestController` annotation serves two purposes
1. It’s a stereotype annotation like `@Controller` and `@Service` that marks a class for discovery by component scanning.
2. All handler methods in the controller should have their return value written directly to the body of the response instead of `@Controller`+ `@ResponseBody`

* `produces="application/json"` : Only handle requests if the request’s *Accept* header includes “application/json”.
* `@CrossOrigin`:  allows clients from any domain to consume the API. RESTful web service will include CORS access control headers in its response

```
@GetMapping("/{id}")
public Taco tacoById(@PathVariable("id") Long id) {
  Optional<Taco> optTaco = tacoRepo.findById(id); 
  if (optTaco.isPresent()) {
      return new ResponseEntity<>(optTaco.get(), HttpStatus.OK); 
      }
  return new ResponseEntity<>(null, HttpStatus.NOT_FOUND);
}

```
* By returning null, the client receives a response with an empty body and an HTTP status code of 200 (OK).
* `@PathVariable`: Using a placeholder variable in the handler method’s path and accepting a path variable.
* `ResponseEntity`: Represents the whole HTTP response: status code, headers, and body. 

### 1.1.2 Sending data to the server
```
@PostMapping(consumes="application/json") 
@ResponseStatus(HttpStatus.CREATED)
public Taco postTaco(@RequestBody Taco taco) {
  return tacoRepo.save(taco);
}
```
* The `consumes` attribute is to request input what `produces` is to request output.
* `@RequestBody`: The body of the request should be converted to a Taco object and bound to the parameter
* `@@ResponseStatus`: Mark a method or an exception class with a status code and reason that should be returned.

### 1.1.3 Updating data on the server
`PUT` or `PATCH`
* `PUT` does a wholesale replacement of the resource data, If any of the order’s properties are omitted, that property’s value would be overwritten with null
* `PATCH`handle requests to do just a partial update,
```

@PatchMapping(path="/{orderId}", consumes="application/json")
public Order patchOrder(@PathVariable("orderId") Long orderId,@RequestBody Order patch) {
      if(patch.getDeliveryStreet() != null) { 
        order.setDeliveryStreet(patch.getDeliveryStreet());
      }
```
The patching approach has a couple of limitations:
* If null values are meant to specify no change, how can the client indicate that a field should be set to null?
* There’s no way of removing or adding a subset of items from a collection.it must send the complete altered collection to do update operations.

### 1.1.4 Deleting data from the server
```
@DeleteMapping("/{orderId}") 
@ResponseStatus(code=HttpStatus.NO_CONTENT)
public void deleteOrder(@PathVariable("orderId") Long orderId) {
    try { 
          repo.deleteById(orderId);
      } catch (EmptyResultDataAccessException e) {}
}
```
* Responses to DELETE requests typically have no body
* `@ResponseStatus(code=HttpStatus.NO_CONTENT)`: communicate an HTTP status code to let the client know not to expect any content
---
## 1.2 Enabling hypermedia
> Hypermedia as the Engine of Application State, or HATEOAS, is a means of creating self-describing APIs wherein resources returned from an API contain links to related resources. 

### 1.2.1.Adding hyperlinks
Spring HATEOAS provides two primary types that represent hyperlinked resources:
* `Resource` :  represents a single resource
* `Resources` : a collection of resources

The links they carry will be included in the JSON (or XML) received by the client.
```
public Resources<Resource<Taco>> recentTacos() {
  PageRequest page = PageRequest.of(0, 12, Sort.by("createdAt").descending());
  List<Taco> tacos = tacoRepo.findAll(page).getContent(); Resources<Resource<Taco>> recentResources = Resources.wrap(tacos);
  //recentResources.add( new Link("http://localhost:8080/design/recent", "recents")); //hardcode a URL 
  recentResources.add(ControllerLinkBuilder.linkTo(DesignTacoController.class).slash("recent").withRel("recents"));

  return recentResources;
}
```
* `ControllerLinkBuilder`: avoid hardcode URL, It is smart enough to know what the hostname
* ` linkTo()`:it canderive the base URL from both the controller’s base path and the method’s mapped path

### 1.2.2 Creating resource assemblers
* `ResourceSupport`:  Inherit a list of `Link` object and methods to manage the list of links.
* Doesn’t include the id property from Taco, The resource’s self link will serve as the identifier for the resource

To aid in converting Taco objects to TacoResource objects, you’re also going to create a resource assembler.
```
public class TacoResourceAssembler extends ResourceAssemblerSupport<Taco, TacoResource> {
public TacoResourceAssembler() { super(DesignTacoController.class, TacoResource.class);
}
  @Override
  protected TacoResource instantiateResource(Taco taco) {
    return new TacoResource(taco);
  }
  @Override
  public TacoResource toResource(Taco taco) {
return createResourceWithId(taco.getId(), taco); }
}
```
`toResource()` appears to have slightly different purposes with `instantiateResource()`
*  `toResource` it to create a Resource object from a domain data,
*  ` instantiateResource() `  instantiate a Resource given a domain data.
*  `instantiateResource()` is intended to only instantiate a Resource object,
*  `toResource()` is intended not only to create the Resource object, but also to populate it with links

```
List<TacoResource> tacoResources = new TacoResourceAssembler().toResources(tacos); //  take advantage of your new TacoResource type
```
###  1.2.3 Naming embedded relationships
`@Relation` annotation can help break the coupling between the JSON field name and the resource type class names as defined in Java.
```
Relation(value="taco", collectionRelation="tacos")
public class TacoResource extends ResourceSupport {
...
}
```
----
## 1.3 Enabling data-backed services
Spring Data REST can automatically creates REST APIs for repositories created by Spring Data
* Spring Data repositories can automatically be exposed as REST APIs using Spring Data REST.

### 1.3.1 Adjusting resource paths and relation names

The `@RestResource` annotation lets you give the entity any relation name and path you want.
  * Spring Data REST tries to pluralize the associated entity class,
```
@RestResource(rel="tacos", path="tacos")
```
### 1.3.2 Adding custom endpoints
`@RepositoryRestController`: controller will have their path prefixed with the value of the `spring.data.rest.base-path` propertyoints.
* it doesn’t ensure that values returned from handler methods are automatically written to the body of the response

### 1.3.4 Adding custom hyperlinks to Spring Data endpoints
Spring Data HATEOAS offers `ResourceProcessor`, an interface for manipulating resources before they’re returned through the API.
```
@Bean
public ResourceProcessor<PagedResources<Resource<Taco>>>
  tacoProcessor(EntityLinks links) {
    return new ResourceProcessor<PagedResources<Resource<Taco>>>() {
      @Override
      public PagedResources<Resource<Taco>> process(
              PagedResources<Resource<Taco>> resource) {\
              resource.add( links.linkFor(Taco.class).slash("recent").withRel("recents")); 
            return resource;} 
       };
}
```
----
# 2 Consuming REST services
> How to use Spring to interact with REST APIs.

A Spring application can consume a REST API with
* RestTemplate — A straightforward, synchronous REST client provided by the core Spring Framework.
* Traverson  —A hyperlink-aware, synchronous REST client provided by Spring HATEOAS. Inspired from a JavaScript library of the same name.
* WebClient — A reactive, asynchronous REST client introduced in Spring 5.
---
## 2.1 Consuming REST endpoints with RestTemplate


`RestTemplate rest = new RestTemplate();`
1. One accepts a String URL specification with URL parameters specified in a variable argument list.
2. One accepts a String URL specification with URL parameters specified in a Map<String,String>.
3. One accepts a java.net.URI as the URL specification, with no support for parameterized URLs.

### 2.1.1 GETting resources
`getForObject()`:  Retrieve a representation by doing a GET on the URL 
* first parameter passed into url placeholders
* second parameter is the type that the response should be bound to.
*  Using a URI parameter is a bit more involved(URI object is defined from a String specification, and its placeholders filled in from entries in a Map)
```
rest.getForObject("http://localhost:8080/ingredients/{id}", Ingredient.class, urlVariables);

Map<String,String> urlVariables = new HashMap<>(); 
urlVariables.put("id", ingredientId);
URI url = UriComponentsBuilder.fromHttpUrl("http://localhost:8080/ingredients/{id}").build(urlVariables);
return rest.getForObject(url, Ingredient.class);
```
If the client needs more than the payload body, you may want to consider using `getForEntity()`.
* it returns a *ResponseEntity* object that wraps that domain object
* The *ResponseEntity* gives access to additional response details, such as the response headers.

### 1.1.2 PUTting resources
All three overloaded variants of `put()` accept an *Object* that is to be serialized and sent to the given URL. 

### 1.2.4 POSTing resource data
> Create a new resource by POSTing the given object to the URI template, and returns the representation found in the response.

* `postForObject()` : If you wanted to receive the newly created Ingredient resource after the POST request
* `postForLocation()` : returns a URI (response’s `Location` header)of the newly created resource instead of the resource object itself. 

---
## 7.2 Navigating REST APIs with Traverson
Traverson comes with Spring Data HATEOAS as the out-of-the-box solution for consum- ing hypermedia APIs in Spring applications. 
* Traverson enables clients to navigate an API using hyperlinks embedded in theresponses.
```
Traverson traverson = new Traverson(URI.create("http://localhost:8080/api"), MediaTypes.HAL_JSON);
ParameterizedTypeReference<Resources<Ingredient>> ingredientType = new ParameterizedTypeReference<Resources<Ingredient>>() {};
Resources<Ingredient> ingredientRes =traverson .follow("ingredients").toObject(ingredientType);
  
Collection<Ingredient> ingredients = ingredientRes.getContent();
```
* By calling the `follow()` method on the Traverson object, you can navigate to the resource whose link’s relation name
* The `toObject()` method requires that you tell it what kind of object to read the data into.
* `ParameterizedTypeReference` : provide type information for a generic type
* it doesn’t do is offer any methods for writ- ing to or deleting from those

Use RestTemplate and Traverson together to both navigate an API and update or delete resources
* Traverson can still be used to navigate to the link where a new resource will be created
* Then RestTemplate can be given that link to do a POST, PUT, DELETE, 

```
private Ingredient addIngredient(Ingredient ingredient) {
    String ingredientsUrl = traverson .follow("ingredients") .asLink().getHref();
    return rest.postForObject(ingredientsUrl, ingredient,Ingredient.class);
}
```
---
# 3. Sending messages asynchronously
1. Asynchronous messaging
2.Sending messages with JMS, RabbitMQ, and
Kafka
3. Pulling messages from a broker
4. Listening for messages

*Asynchronous messaging* is a way of indirectly sending messages from one application to another without waiting for a response. 

Consider three options that Spring offers for asynchronous messaging: 
* the Java Message Service (JMS) RabbitMQ 
* Advanced Mes- sage Queueing Protocol (AMQP)
* Apache Kafka.

## 3.1 Sending messages with JMS
成功
Consider three options that Spring offers for asynchronous messaging: the Java Message Service (JMS), RabbitMQ and Advanced Mes- sage Queueing Protocol (AMQP), and Apache Kafka.暑期
Consider three options that Spring offers for asynchronous messaging: the Java Message Service (JMS), RabbitMQ and Advanced Mes- sage Queueing Protocol (AMQP), and Apache Kafka.
