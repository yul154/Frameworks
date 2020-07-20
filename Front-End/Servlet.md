Servlet
------------
> A servlet is a small Java program that runs within a Web server. Servlets receive and respond to requests from Web clients, usually across HTTP, the HyperText Transfer Protocol.

# Architrecyture of Servlets
<img width="922" alt="Screen Shot 2019-07-25 at 7 13 47 PM" src="https://user-images.githubusercontent.com/27160394/61914949-5c09e280-af10-11e9-8951-3ed303a3b1aa.png">

1. Client types in a URL at the browser, send an HTTP request for the servlet
2. Web container got the request, create two objects(`HttpServletResponse`,`HttpServlerRequest`)
3. Container finds the correct Servle based on the URL in the request, Allocates a threat for that request, Pass the request and respnose to the Servlet thread
4. The container call the Servlet's service method(`doGet`,`doPost`)
5. The methods generates dynamic pages and stuffs the pages into response object
6. The container convert response obejct into an HTTP response and send it back to Client
7. Delete the request and response obejcts.

# Servlets Class Hierarchy
## `Servlet`(interface)
> All servlets have to implement this interface directly or by extending subclass(`GenericServlet`,`HttpServlet`)
* `destory`: Cleans up whatever resources are being held, and make sure that any persistent state is synchronized with Servlet's current in-memory state
* `getServletConfig`: Return Servlet config object which contains ant initialization parameters anD startup configuration
* `getServletInfo`: Return a String containing Servlet information
* `init`: Initilize the Servlet, will execute once before any request is Serviced
* `Service`: Carry out a single request from the client, it will send method information to `doGet()`,`doPost()`
## `GenericServlet`
> an abstract class that implements Servlet interface
* protocol independent: it handles all types of protocols, HTTP, SMTP, FTP
* Can't track Session data
## `HttpServlet`
> an abstract class which is a direct subclass of `GenericServlet`
* Only handles HTTP protocol
* Supports `doGet`,`doPost`
* All methods accepts `HttpServletResponse` and `HttpServlerRequest` objects and throw ServletException and IOException
### `HTTPServletRequest` Variable Methods
* `getParameter`: ReturnS the value of a request parameter
* `getParameterValues`:Return an array of String object containing all of the value for given parameter name
* `getParameterMap()`: Return a java.util.Map of the parameter of this request
* `getParameternames()`:Returns an Enumeration of String objects containing the names of the parameters contained in this request
### Handle data
* `GET` method use to read data, `POST` method use to submit data for adding a new resource

`GET`|`POST`
-----|--------
back button reload page is Harmless| Data will be re-submitted and an alert is displayed
Request bookmarked| can't be book marked
Request can be Cached| Cache is not supported
Parameter values will be maintained at the Browser's History| Parameters are not saved in Browser's History
Size of data is limited supports 2048 characters| no size restrictions
Only support ASCII Characters| No restrictions on type of data submitted
Less Secured|secured

### Handle HTTP Requests and Responses
#### Request Headers
* `getHeaader(String name)`: return specify request header as a string
* `getHeaderNames()`:returns all head names as an enumeration 
* `getAuthType()`: return the name of the authentication schema used to protect the Servlet
* `getRemoteUser()`: return the name of the user making the request if the user has been authenticated
* `getRemoteAddr()`: return the IP address of the client or the last proxy address that sent a request
* `getContentLength()`: return the value of the content length header as an int
* `getContentType()`: return the value of the content type header as a string
* `getDateHeader(String name)`: returns the value of specified request header as a long value that represents a date object
* `getIntHeader(String name)`: return value of specified header as an integer
#### Response Headers
* `setHeader(String name, String value)`: set response header with the given name and value.
* `setIntHeader(String name, int value)`: set the response header which requests an integer as its value.
* `setDateHeader(String name, long date)`: set the response header which requests value as a timestamp.
* `setContentType(String type)`: set the content type of header of the response being sent to the client.
* `containsHeader(String name)`: return true if the header exists.
* `setContentLength(int len)`: set the length of the content body in the response in HttpServlet.
* `setLocale(Locale loc)`: set the locale of the response
#### Http Status code
<img width="990" alt="Screen Shot 2019-07-26 at 4 53 29 PM" src="https://user-images.githubusercontent.com/27160394/61980878-fc6e0e80-afc5-11e9-9703-46eebdcc65e5.png">

### Filters
> a small amount of code that executes before or after serving up a web resource
* Modify the request from clients before the request accesses the back-end resources
* Manipulate response from the server before it reaches to the client.
<img width="807" alt="Screen Shot 2019-07-30 at 12 36 06 PM" src="https://user-images.githubusercontent.com/27160394/62147945-a8fe0800-b2c6-11e9-8026-de091a737a7d.png">

#### Functions
1. perform application task
2. reuse without rebuilding the application
3. configuratiion settings
4. chained togther
5. initialization parameters
#### Useage
1. record all the incoming requests
2. log the IP addresses of the computers
3. conversions
4. Data compressions
5. Encryption and Decryption
6. Input validations
7. Authentication and Authorization
8. Audit access to sensitive resources
9. Email system administrators on every application error
10. Make the application perform better

#### Life cycle
1. Load Filter Class
2. Create Filter instance
3. Call `init()`
4. Call `doFilter()`
5. Call `destory()`

#### How a filter works
1. `web.xml` declare that a filter should be invoked for a servlet
2. `web.xml` applies all filters associated with servlet
3. For each filter, the web container invokes(`init()`,`doFilter()`,`destroy()`)

## Handling exceptions in Servlets
### Process
1. Servlet throw a exception
2. Web container search the configurations in `web.xml`
3. Use the exception type element for a match with th throw exception type
4. Use error-page elements in `web.xml` to specify the invocation of Servlet in response to certain exceptions
### `web.xml` Configuration
* use `<error-page>`
   * configure based on error code:`<error-code>`
   * configure error page based on exception type:`<exception-type>`
   * Generio error handler for all exception:`<exception-type>java.lang.Throwable </exception-type>`
   * url-pattern:`<location>`
* Request attributes-Exceptions

<img width="477" alt="Screen Shot 2019-07-30 at 3 11 34 PM" src="https://user-images.githubusercontent.com/27160394/62157966-59c2d200-b2dc-11e9-8cf4-11b428f1b2c9.png">.




# Servlet Life Cycle
1. Loading of Servlet class
2. Create the Servlet instance((after once, the same identical requests are execute on the same servlet instance))
3. Call the `init()` method: Provide the privilege to configure a particular Servlet before the first request is processed,specify init parameters to the Servlet in `web.xml` or annotations
4. Call the `service()` method: To process the request, have to override one of `doGet`or `doPost`
5. Call the `destroy()` method: When Servlet is unloaded by the Servlet container
* First three steps executed once

# Web application work

![v2-955ff75e5217fe00274a13ca880cb80c_r](https://user-images.githubusercontent.com/27160394/60986170-e0ebde00-a30c-11e9-8d6a-29988c5b3e9f.jpg)

## 1. Browser send a HTTP request to Web server
* Web Container receive web reques.
## 2. Code in Web Server=> Input: HTTP request, Output: HTTP response
* Servlet is process request In Web Container
### Method
#### `javax.servlet`
* `servlet`
* `serletRequest`
* `serletResponse`
* `ServletContext`
* `ServletConfig`

#### `java.servlet.http`
* `HttpServlet`
* `HttpServletResponse`
* `HttpSession`
* `Cookie`
```
@WebServlet(urlPatterns="/login.do")# url assigned particular servlet to handle this request
public class LoginServlet extends HttpServlvlet{
   @Override
   protected void doGet(HttpServletRequest request, HttpServletResponse response)#define a method which response back content 
     PrintWriter out=response.getWriter();# write response content
     //html content
     }
     
```
### Web Container
#### Functions
* Implement servlet class on container
* Manage the lift circle of servlets
* Map URL on specific Servlet
* Deal with HTTP requestion with Servlet program
* Session management, HTTP cache

## 3. Server send a `html` file back to browser
### Ways of response
1. `PrinterWriter`: directly response html content
2. `OutputStream`:response binary data
3. `sendRedirect`: redirect to another URL
4. `sendError`: response error information

## Difference between web server and application server
* Web Server is designed to serve HTTP Content. App Server can also serve HTTP Content but is not limited to just HTTP
* Web Server is mostly designed to serve static content, though most Web Servers have plugins to support scripting languages like Perl, PHP, ASP, JSP etc. through which these servers can generate dynamic HTTP content.
* App Server have components and features to support Application level services such as Connection Pooling, Object Pooling, Transaction Support, Messaging services etc.
* As web servers are well suited for static content and app servers for dynamic content, most of the production environments have web server acting as reverse proxy to app server. That means while servicing a page request, static contents (such as images/Static HTML) are served by web server that interprets the request. Using some kind of filtering technique (mostly extension of requested resource) web server identifies dynamic content request and transparently forwards to app server


# `pox.xml`

# Session
* Http protocol is a stateless so we need to maintain state using session tracking techniques.
## Tracking Session Data
### Hidden Form fields
* It is HTML form element which used to store the value, it will not be visible on the page
* Servlet or server-side code can extract Hidden form value
* Syntax: `<input type="hidden" name="FormElementName",value="need maintained across the pages"/>`
* When we use HTML hidden form field, it's mandatory that in every Servlet page

<img width="961" alt="Screen Shot 2019-07-30 at 4 13 14 PM" src="https://user-images.githubusercontent.com/27160394/62161947-38b2af00-b2e5-11e9-8d5b-2cdcee620745.png">.

### URL Rewriting
* Value appended at the URL can be consider as a qeury string data
* Allow user append additional information which required across the Servlet pages at the end of URL
* Servlet's pages should always be appended with a URL by rewriting the URL dynamically with the values to be passed to the other Servlet pages
* Works only with Hyperlinks
* Can submit only text information
* Complex to implement
<img width="978" alt="Screen Shot 2019-07-31 at 11 46 46 AM" src="https://user-images.githubusercontent.com/27160394/62226893-eb881900-b388-11e9-9c1e-90132c13e2a6.png">

### Cookie

* Used to store the server side information at the client system(Can't across different browser).
* Once set the cookies in one Servlet page, then that valu can be used from other Servlet pages
* Types of Cookie
  1. Session o Temporary Cookies: maintained within the memory allocated at the web browser
  2. Persistent or Permanent Cookies: maintain cookie data within a physical file at Client system
* Process
  1. create an object for the Cookie:`Cookie cookie= new Cookie("key","value")`
  2. Set the expiration period for the Cookie:`cookie.setMaxage(300)`,If not set time,the cookie will be permanent cookies.
  3. add the Cookie to header: `response.addCookie(cookie)`
  4. Retrieve the Cookie: `request.getCookies()`
* Can maintain only String information

### Session
* maintain the user information at th server side temporarily
* Session can b accessed across the servlets
* `HttpServlet` provide a way to identify a user across more than one page request
* Syntax:
  * create an `httpsession` object: `HttpSession session=request.getSession();`
* Create session before send content to client
* Step1
<img width="880" alt="Screen Shot 2019-07-31 at 2 50 55 PM" src="https://user-images.githubusercontent.com/27160394/62239371-be486480-b3a2-11e9-9c79-9cd980685c02.png">

* Step 2
<img width="870" alt="Screen Shot 2019-07-31 at 2 53 00 PM" src="https://user-images.githubusercontent.com/27160394/62239462-e932b880-b3a2-11e9-9ce4-76b417690a50.png">

* Can store String and object of data as value


# Upload a file
* front-end , need to create a form
```
<form method=" action="UploadingServlet" enctype="multipart/form-data">
<p><input type="file" name="file" size="60" /></p>
</form>
```
* back-end(Servlet side): annotate with `MultipartConfig`
```
@MultipartConfig(
		fileSizeThreshold=1024*1024*2,
		maxFileSize=1024*1024*10,
		maxRequestSize=1024*1024*50
		)
```
  
# Annotations
## @WebServlet
<img width="914" alt="Screen Shot 2019-08-01 at 10 23 16 AM" src="https://user-images.githubusercontent.com/27160394/62301335-6b27ed80-b446-11e9-8996-56c6815ed3a2.png">

## @WebInitParams

* Declare a initialization parameter on Servlet or filter within the WebServlet or web filter annotation

<img width="935" alt="Screen Shot 2019-08-01 at 10 30 00 AM" src="https://user-images.githubusercontent.com/27160394/62301850-55ff8e80-b447-11e9-962a-1d95d05ae2f7.png">

## @WebFilter
<img width="904" alt="Screen Shot 2019-08-01 at 10 45 18 AM" src="https://user-images.githubusercontent.com/27160394/62303073-7deff180-b449-11e9-86aa-bd94fdad5ef7.png">

# Asynchronous Servlet Processing
## Traditional Servlet processing
* A server thread to per client request.
* If a request takes more amount of time, the servlet will be sitting idle(Blocking operations)
## Asynchronous Servlet
* A given HTTP worker thread handles an incoming request 
* Passes the request to another background thread which in turn will be responsible for processing the request and send the response back to the client
* The initial HTTP worker thread will return to the HTTP thread pool as soon as it passes the request to the background thread, so it becomes available to process another request.

