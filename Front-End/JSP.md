JSP(Java Server Pages)
--------------
* java not good at writing `html`
* Jsp is easy to write dynamic `html` content rather than in servlet
```
request.getRequestDispatcher("JSP address").forward(request,response);
````
![sa](https://user-images.githubusercontent.com/27160394/60986045-9e2a0600-a30c-11e9-9221-bc518c00f120.jpg)

![v2-955ff75e5217fe00274a13ca880cb80c_r](https://user-images.githubusercontent.com/27160394/60986095-bb5ed480-a30c-11e9-954d-244f49d4acc4.jpg)

# Implicit Objects
- Request
- Response
- Out 
- Session
- pageContext
- Application
- Exception
- Config

# Directives
- Page
- Include
- Taglib(insert java code)
  - JSTL
    - Core
    - Function
    - XML
    - SQL
    - Formatting
```
<%@ @%>
```
  
# Action Tags
>
`<jsp:......../>.`

- Include
- Forward
- useBean
- setProperty
- getProperty

## Scriptlet
```
<%
%>
```
## Declarative Tag
* Declares variable o methods
```
<%!
%>
```
## Expression Tag
* Contains an experssion
```
<%= %>
```
## comment 
```
<% -- comment -- %>
```
