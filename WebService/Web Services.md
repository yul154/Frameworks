Web Services
---------------------
* it can allow other software applications to make a call to its methods
* Client and service provider may not develope using same language, even not in same machine
* All basic information (location,signature) provided in a file(XML format)
*  In a request-response conversation, 
  1. the client sends a request of "Content-Type"
  2. expects to receive the response of "Accept" media type.
*  Methods can also have the property of "idempotence": the side-effects of N > 0 identical requests is the same as for a single request.”

# SOAP
* Communicate between client application and web service application follow SOAP format will be SOAP service

# RESTful- Representation State Transfer
* If one software application exposes data to other application follow all guidelines as specified in rest, that application is known as a RESTful web service Application
* a URL identifies a resource.
* same URL with HTTP verb method for different the action performed
* Countless number of intermediaries can be installed between the Client and the Server
* Client need to specify along with the request in what format it wants to response
## Workflow

![Restful-Web-Services-Architecture](https://user-images.githubusercontent.com/27160394/63359475-d0cb1380-c33a-11e9-9c5e-6c1e592a682f.png)

1. Client send request for a resource
2. Server send response back 
  * Representation of resource: copy of requested resource in one of valid formats like xml,json,html
  * Addition Information: describes every basic chararcteristic(FORMATE,SIZE) of this representation of resource
 
## `PUT` request
* Client do update certain information which at the server side
* @ResponseBody in signature convert certain type information into its equivalent java object
* 
```
@RequestMapping(value = "/orders/{id}", method = RequestMethod.PUT)
public String getOrder(@PathVariable("id") final String orderId,@ResponseBody Student student) {
  return "Order ID: " + orderId;
}
```
## `POST` request
*  want users to make a request to create a new resource(user send new information to the server) at the server
```
@RequestMapping(value = "/orders/", method = RequestMethod.POST,consumes = MediaType.APPLICATION_JSON_VALUE, )
public String getOrder(@PathVariable("id") final String orderId,@ResponseBody Student student) {
  return "Order ID: " + orderId;
}
```
## `DELETE` request
```
@RequestMapping(value = "/orders/{id}", method = RequestMethod.DELETE)
public ResponseEntity<Boolean>(@PathVariable("id") final String orderId) {
  return new ResponseEntity<Boolean>(true,HttpStatus.OK);
}
```
## Annotations
### `@PathVariable`
* used for data passed in the URI
* can be used in any type of request method (GET, POST, DELETE, etc).  
```
@RequestMapping(value = "/orders/{id}", consumes = MediaType.APPLICATION_JSON_VALUE, method = RequestMethod.GET)
@ResponseBody
public String getOrder(@PathVariable("id") final String orderId) {
  return "Order ID: " + orderId;
}
```
### `RestController`
* A convenience annotation that is itself annotated with @Controller and @ResponseBody.
* The controller is annotated with the @RestController annotation, therefore the @ResponseBody isn’t required.

## `@RequestMapping`
### `produces`
* Restrict controller support only one format of Accpet
* Narrow the mapping types.
```
@RequestMapping(
  value = "/hello", 
  method = RequestMethod.GET, 
  produces = MediaType.APPLICATION_JSON_VALUE, 
)
```
### `consumes`
* Restrict controller support only one format of content type
```
@RequestMapping(
  value = "/hello", 
  method = RequestMethod.PUT, 
  consumes = MediaType.APPLICATION_JSON_VALUE, 
)
```
### `ResponseEntity`
*  represents the whole HTTP response: 
  1. status code:  declares the status code of the HTTP response.
  2. headers
  3. body
```
 @RequestMapping("/handle")
 public ResponseEntity<String> handle() {
   HttpHeaders responseHeaders = new HttpHeaders();
   responseHeaders.setLocation(location);
   responseHeaders.set("MyResponseHeader", "MyValue");
   return new ResponseEntity<String>("Hello World", responseHeaders, HttpStatus.CREATED);
 }
```
### JSON annotations
#### `ResponseBody`
* Tells a controller that the object returned is automatically serialized into JSON
* Passed back into the HttpResponse object.
#### `JSONproperty`
* Convert object variable name in JOSN to specific name
```
@JsonProperty("name")
private String S_name;
```
#### `@JsonPropertyOrder`
* specify the order of properties on serialization
```
@JsonPropertyOrder({ "name", "id" })
public class MyBean {
    public int id;
    public String name;
}
```
#### `@JsonIgnoreProperties`
* a class-level annotation that marks a property or a list of properties that Jackson will ignore.
```
@JsonIgnoreProperties({ "id" })
public class BeanWithIgnore {
    public int id;
    public String name;
}
```
#### `@JsonInclude`
* exclude properties with empty/null/default values.
```
@JsonInclude(Include.NON_NULL)
public class MyBean {
    public int id;
    public String name;
}
```

## POSTMAN
> a powerful HTTP client for testing web services
* A standard browser tool would only allow you to test get type of request
* It has the ability to make various types of HTTP requests(GET, POST, PUT, PATCH).
* saving environments for later use. 
* converting the API to code for various languages(like JavaScript, Python).





Different betweet r
```e
```s
```
```
