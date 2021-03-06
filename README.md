# Spring REST Tutorials
## What is webservice, and it's types?
Consider a case where your application need to provide data to other application. To achieve this you can give access to
your DB to client application and Business Layer and Access layer code in a jar. But that is not secure and also it 
exposes to security risks to your data if client application have bugs or vulnerabilities. 

Webservices are designed to solve this kind of scenario; Web Services exposes the application data to other application
over HTTP protocol and in a specific format. Let's see 3 key points for webservice:
1. Designed for application-to-application interaction
2. Should be interoperable - Not platform dependent
3. Should allow communication over a network

### Hows for webservice
1. How does data exchange between applications takes place?

It is in the form of HTTP Request and Response over the network. Also this can be done by MQ or event publish and consume
mechanism.

2. How can we make web services platform independent?

By making the request and response format platform independent. There are two standard formats which are right now 
supported with different platforms; those are XML and JSON. One format now days in come popularity which is gRPC.


3. How does Client application know the format of Request and Response?

Service Definition provided by webservice. Service definition contains Format and Structure of the Request and Response
with Endpoint and HTTP method or Queue Names and Connection.

### Types of Webservices
* **SOAP Web Service**
 
In SOAP, we use XML as Request and Response format. It defines a specific Structure for request and response in the
format of Envelope, Header and Body. Header contains meta-information about request and Body is input for request.
It provides the Service Definition in the form WSDL.

* **REST Web Service**

REpresentation State Transfer(REST) is widely used now-a-days for the webservices. With REST, we can best utilize of 
HTTP. In REST, we use most of HTTP components; we have HTTP headers, body, methods and status codes. Anything you want
to expose to other application are called Resource. In REST we have resource URI(Uniform Resource Identifier) to 
identify a particular resource. A Resource can different representation in REST; it could be XML, JSON, SAML or YML.
The resource uses the HTTP methods to segregate actions on resources like POST is used for create, DELETE is used for 
delete, PUT for Update and Get for retrieval of resource.

REST transport type is HTTP as we are using HTTP method and Status code for request and response. There is no standard
format for Service definition only documentation or WADL(Web Application Definition Language) or Swagger.


## REST Web Services with Spring Boot
### Setting up Spring BOOT Project for Writing REST API 
To start with; Let's create a project with the spring starter project. Considering this you have working knowledge of 
Spring JPA and Liquibase; we have included them as well. While starting the project DB table is created in H2 DB which
you can access on your local system by accessing URL .

### Application we are going to build in this tutorial
For demonstrating the REST API and how to design them effectively we are going to create an application. This application
will include users and user's todo task list. The application will register user with basic details; and then give them
access to create todo task and manage their tasks.

Initially we are going to expose the API for the CRUD operations on users and their tasks. Then we will try to write 
better logic with them with versioning of API, and we will also understand Richardson Maturity model for REST API and
HATEOAS. Let's get started.

### Database and Spring JPA setup
We have created liquibase setup for creating the table and inserting demo data into table. You can find the liquibase 
changelog in directory [here](src/main/resources/db/changelog).

Also, we have created the entity object of Tables and PagingAndSorting Repository for corresponding entity. Please find
the links below:
* [Entities](src/main/java/com/codewithnaman/entity)
* [Repositories](src/main/java/com/codewithnaman/respository)

### Create REST API
#### Create Simple API
For creating the REST API endpoint; we need to create a controller which will create a bean of our class. We will use 
annotation `@RestController` which is combination of annotation `@Controller` and `@ResponseBody`. In Java Class, we will
create an endpoint using the `@GetMapping` annotation over the method whose return type is response object and may take
parameters depending on the request and bindings. `@GetMapping` is equivalent to `@RequestMapping(method=RequestMethod.GET)`.
Similar like this we have `@PostMapping`,`@PutMapping`,`@DeleteMapping` and `@PathMapping`.

For testing purpose we have created a controller which will return result as "Test OK!"; where we used `@RestController`
and `@GetMapping` for creating an endpoint. Example [TestController](src/main/java/com/codewithnaman/controller/TestController.java).

We can pass the http path directly to GetMapping as argument or also as part of value. If we are defining the multiple
attribute for annotation GetMapping then we will use path parameter and pass it like `@GetMapping(path="test")`; if
there is single parameter path need to set then we can also provide like `@GetMapping("test")`.

Similar like above Let's start writing the REST API to return the users. Let's first write to get all the users. The API
code look like below:
```text
 @GetMapping("users")
    public List<User> getAllUsers(){
        return retrieveUserService.getAllUsers();
    }
```
Let's hit the API; I am hitting the API using the IntelliJ HTTP Client, and it returned the response like below:
```text
GET http://localhost:8080/users

HTTP/1.1 200 
Content-Type: application/json
Transfer-Encoding: chunked
Date: Mon, 28 Dec 2020 09:35:05 GMT
Keep-Alive: timeout=60
Connection: keep-alive

[
  {
    "id": 1,
    "userName": "admin",
    "firstName": "admin",
    "lastName": "admin",
    "metadata": {
      "createDate": "2020-12-28T15:04:59.471",
      "updateDate": "2020-12-28T15:04:59.471"
    },
    "tasks": []
  },
  {
    "id": 2,
    "userName": "user",
    "firstName": "user",
    "lastName": "user",
    "metadata": {
      "createDate": "2020-12-28T15:04:59.471",
      "updateDate": "2020-12-28T15:04:59.471"
    },
    "tasks": []
  }
]

Response code: 200; Time: 313ms; Content length: 336 bytes
```

Right now API returned the two users which we have been the setup as part of the DB setup. Let's now say we want to 
take parameter username and find the user; then return that particular user. Let's take parameter as part of URL and find
the user.

#### Create API taking parameter from URL
To take parameters from an url we pass the path url, and the parameter inside the {} braces. For example, we will provide
the path like /users/{searchBy}/{username}; Where searchBy and username are variables in the path. To retrieve the path 
variables value; we need to add variables in our method and to bind it.To do this we use the `@PathVariable` annotation.

Let's see this in example:
```text
 @GetMapping("users/{searchBy}/{idOrUserName}")
    public User getAllUsers(@PathVariable String searchBy,@PathVariable String idOrUserName){
        switch (searchBy){
            case "username":
                return retrieveUserService.getUserByUserName(idOrUserName);
            default:
                throw new RuntimeException("Invalid search criteria provided");
        }
    }
```
The URL parameters are binding to method variable which we used to search the user in the service. Let's hit the endpoint
and see the results.
```text
GET http://localhost:8080/users/username/admin

HTTP/1.1 200 
Content-Type: application/json
Transfer-Encoding: chunked
Date: Mon, 28 Dec 2020 10:00:29 GMT
Keep-Alive: timeout=60
Connection: keep-alive

{
  "id": 1,
  "userName": "admin",
  "firstName": "admin",
  "lastName": "admin",
  "metadata": {
    "createDate": "2020-12-28T15:22:33.098",
    "updateDate": "2020-12-28T15:22:33.098"
  },
  "tasks": []
}

Response code: 200; Time: 46ms; Content length: 168 bytes
```

#### Create API taking input as part of body
Let's create the user using REST API. For taking the User Inputs we have created DTO object named CreateUserRequest. This will
define the format of your input which can be JSON,XML etc. The DTO fields represent the tag and value be defined accroding
to content-type.

To Map the input over the HTTP to our DTO Object we will use the annotation `@RequestBody`. So whatever JSON body we pass
from the HTTP Request Body will be parsed, and our DTO object will be initialized with the value passed in request body.
After getting the DTO object initialized with the values; we can perform our business validation and persist the data or
further flow depending on the Business Flow.

Let's see the example of this:
```text
 @PostMapping("/users")
    public User registerUserUser(@RequestBody CreateUserRequest createUserRequest){
        return this.createUserService.createUser(createUserRequest);
    }
```
This will take the input; initialize the CreateUserRequest DTO and pass it to service to create user. Let's hit the endpoint
and see what will be the output.

**Request :**
```text
POST http://localhost:8080/users
Content-Type: application/json

{
"userName":"user1",
"firstName":"user1",
"lastName":"user1"
}
```

**Response :**
```text
POST http://localhost:8080/users

HTTP/1.1 200 
Content-Type: application/json
Transfer-Encoding: chunked
Date: Mon, 28 Dec 2020 10:38:38 GMT
Keep-Alive: timeout=60
Connection: keep-alive

{
  "id": 3,
  "userName": "user1",
  "firstName": "user1",
  "lastName": "user1",
  "metadata": {
    "createDate": "2020-12-28T16:08:38.789",
    "updateDate": "2020-12-28T16:08:38.789"
  },
  "tasks": null
}

Response code: 200; Time: 109ms; Content length: 170 bytes

```
We can see the right now status code 200, But for the resource creation right HTTP code would be 201. Also, we are 
sending the Complete object of the user, What if we can send the URI and user can call the URI whenever he wants to get
the information of user.

For giving the URI and HTTP status code we will use the ResponseEntity; which will create the response object for us, 
and we will set HTTP status code with this; so user will get proper HTTP code for creation of resource. Let's make the 
changes for this.
```text
 @PostMapping("/users")
    public ResponseEntity<Object> registerUserUser(@RequestBody CreateUserRequest createUserRequest) {
        User registeredUser = this.createUserService.createUser(createUserRequest);
        URI registeredUserUri = ServletUriComponentsBuilder
                .fromCurrentRequest()
                .path("/{id}")
                .buildAndExpand(registeredUser.getId())
                .toUri();
        return ResponseEntity.created(registeredUserUri).build();
    }
```
So we are building the uri from the current request appending the id of created resource and ResponseEntity.created will
also return the status code as 201. Let's restart the application and send the request again.

```text
POST http://localhost:8080/users

HTTP/1.1 201 
Location: http://localhost:8080/users/3
Content-Length: 0
Date: Mon, 28 Dec 2020 11:08:09 GMT
Keep-Alive: timeout=60
Connection: keep-alive

<Response body is empty>

Response code: 201; Time: 395ms; Content length: 0 bytes
```
As we can see the Response code this time is 201 and in headers we have Location attribute which contains the location 
of the created resource.

#### Exception Handling in case of REST API
We have retrieved the user and created the resource. Let's see what happen when we request for a user which does not 
exist. Let's try one request and see:
```text
GET http://localhost:8080/users/username/user500

HTTP/1.1 500 
Content-Type: application/json
Transfer-Encoding: chunked
Date: Tue, 29 Dec 2020 03:57:34 GMT
Connection: close

{
  "timestamp": "2020-12-29T03:57:34.936+00:00",
  "status": 500,
  "error": "Internal Server Error",
  "trace": "java.util.NoSuchElementException: No value present\r\n\tat java.util.Optional.get(Optional.java:135)\r\n\tat com.codewithnaman.service.user.RetrieveUserService.getUserByUserName(RetrieveUserService.java:26)\r\n\tat com.codewithnaman.controller.user.RetrieveUserController.getAllUsers(RetrieveUserController.java:29)\r\n\tat sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)\r\n\tat sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)\r\n\tat sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)\r\n\tat java.lang.reflect.Method.invoke(Method.java:498)\r\n\tat org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:197)\r\n\tat org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:141)\r\n\tat org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:106)\r\n\tat org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:894)\r\n\tat org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:808)\r\n\tat org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87)\r\n\tat org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1061)\r\n\tat org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:961)\r\n\tat org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1006)\r\n\tat org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:898)\r\n\tat javax.servlet.http.HttpServlet.service(HttpServlet.java:626)\r\n\tat org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:883)\r\n\tat javax.servlet.http.HttpServlet.service(HttpServlet.java:733)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:231)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)\r\n\tat org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)\r\n\tat org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:100)\r\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)\r\n\tat org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:93)\r\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)\r\n\tat org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:201)\r\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)\r\n\tat org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:202)\r\n\tat org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:97)\r\n\tat org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:542)\r\n\tat org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:143)\r\n\tat org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92)\r\n\tat org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:78)\r\n\tat org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:343)\r\n\tat org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:374)\r\n\tat org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:65)\r\n\tat org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:888)\r\n\tat org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1597)\r\n\tat org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49)\r\n\tat java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)\r\n\tat java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)\r\n\tat org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)\r\n\tat java.lang.Thread.run(Thread.java:748)\r\n",
  "message": "No value present",
  "path": "/users/username/user500"
}

Response code: 500; Time: 140ms; Content length: 5306 bytes
```
We get the response code as 500 and standard response which spring configured. Let's say we want to provide custom HTTP 
code and message for now in this response. Then we need to create an exception class for UserNotFound and annotate with
`@ResponseStatus` and status code as argument of the annotation. Let's try it and throw this exception when user is not 
found and see the results.
* Exception class for this
```text
package com.codewithnaman.exception.user;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;

@ResponseStatus(HttpStatus.NOT_FOUND)
public class UserNotFoundException extends RuntimeException {

    public UserNotFoundException(String userNameOrID){
        super("User Not found with provided input : "+userNameOrID);

    }
}
```
* Response from the API:
```text
GET http://localhost:8080/users/username/user500

HTTP/1.1 404 
Content-Type: application/json
Transfer-Encoding: chunked
Date: Tue, 29 Dec 2020 04:05:04 GMT
Keep-Alive: timeout=60
Connection: keep-alive

{
  "timestamp": "2020-12-29T04:05:04.771+00:00",
  "status": 404,
  "error": "Not Found",
  "trace": "com.codewithnaman.exception.user.UserNotFoundException: User Not found with provided input : user500\r\n\tat com.codewithnaman.service.user.RetrieveUserService.getUserByUserName(RetrieveUserService.java:32)\r\n\tat com.codewithnaman.controller.user.RetrieveUserController.getAllUsers(RetrieveUserController.java:29)\r\n\tat sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)\r\n\tat sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)\r\n\tat sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)\r\n\tat java.lang.reflect.Method.invoke(Method.java:498)\r\n\tat org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:197)\r\n\tat org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:141)\r\n\tat org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:106)\r\n\tat org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:894)\r\n\tat org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:808)\r\n\tat org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87)\r\n\tat org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1061)\r\n\tat org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:961)\r\n\tat org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1006)\r\n\tat org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:898)\r\n\tat javax.servlet.http.HttpServlet.service(HttpServlet.java:626)\r\n\tat org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:883)\r\n\tat javax.servlet.http.HttpServlet.service(HttpServlet.java:733)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:231)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)\r\n\tat org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)\r\n\tat org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:100)\r\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)\r\n\tat org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:93)\r\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)\r\n\tat org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:201)\r\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)\r\n\tat org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:202)\r\n\tat org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:97)\r\n\tat org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:542)\r\n\tat org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:143)\r\n\tat org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92)\r\n\tat org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:78)\r\n\tat org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:343)\r\n\tat org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:374)\r\n\tat org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:65)\r\n\tat org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:888)\r\n\tat org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1597)\r\n\tat org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49)\r\n\tat java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)\r\n\tat java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)\r\n\tat org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)\r\n\tat java.lang.Thread.run(Thread.java:748)\r\n",
  "message": "User Not found with provided input : user500",
  "path": "/users/username/user500"
}

Response code: 404; Time: 294ms; Content length: 5322 bytes
```

As we can see the message is what we provided in exception and HTTP code to 404. Let's say we don't want to use the 
message format provided by the spring, and we want to produce our own message for application. For doing that we have 
two ways :
1. We can extend the class ResponseEntityExceptionHandler and override the handleException method for our exceptions and
   annotate with `@ControllerAdvice` and in `@ExceptionHandler`, we pass our class.
2. We can create a class annotate with `@ControllerAdvice` and in `@ExceptionHandler`, we pass our class.


Let's understand difference between both; By extending the class we get other methods to override which handle the other
type of exception like mediatype, method not support, method argument etc.

Let's first create a class with Annotation only and apply our Controller Advice There.
```java
package com.codewithnaman.exception.handlers;

import com.codewithnaman.dto.error.BusinessExceptionErrorResponse;
import com.codewithnaman.exception.BusinessException;
import com.codewithnaman.exception.EntityNotFoundException;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

@ControllerAdvice
public class BusinessExceptionHandler {

    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<BusinessExceptionErrorResponse> handleBusinessException(BusinessException exception) {
        if (exception instanceof EntityNotFoundException) {
            return new ResponseEntity(
                    new BusinessExceptionErrorResponse(exception.getErrorCode(), exception.getMessage()),
                    HttpStatus.NOT_FOUND);
        }
        return new ResponseEntity(
                new BusinessExceptionErrorResponse(exception.getErrorCode(), exception.getMessage()),
                HttpStatus.BAD_REQUEST);
    }
}
```
In this class we are catching all BusinessException and Returning the response depending on which type of Business
Exception occurred.

Let's hit the request and see the response:
```text
GET http://localhost:8080/users/username/user500

HTTP/1.1 404 
Content-Type: application/json
Transfer-Encoding: chunked
Date: Tue, 29 Dec 2020 04:34:53 GMT
Keep-Alive: timeout=60
Connection: keep-alive

{
  "errorCode": "RESOURCE_NOT_FOUND",
  "errorMessage": "User Not found with provided input : user500"
}

Response code: 404; Time: 279ms; Content length: 96 bytes
```
Here we can see the custom response we have created with status code we provided.

Let's consider one more case where we are creating the user; and we need to validate the input. Let's first try without 
making any changes and what is output we are getting:

**Request :**
```text
POST http://localhost:8080/users
Content-Type: application/json

{
"firstName":"user1",
"lastName":"user1"
}
```
**Response :**
```text
POST http://localhost:8080/users

HTTP/1.1 500 
Content-Type: application/json
Transfer-Encoding: chunked
Date: Tue, 29 Dec 2020 04:37:39 GMT
Connection: close

{
  "timestamp": "2020-12-29T04:37:39.181+00:00",
  "status": 500,
  "error": "Internal Server Error",
  "trace": "org.springframework.dao.DataIntegrityViolationException: not-null property references a null or transient value : com.codewithnaman.entity.User.userName; nested exception is org.hibernate.PropertyValueException: not-null property references a null or transient value : com.codewithnaman.entity.User.userName\r\n\tat org.springframework.orm.jpa.vendor.HibernateJpaDialect.convertHibernateAccessException(HibernateJpaDialect.java:294)\r\n\tat org.springframework.orm.jpa.vendor.HibernateJpaDialect.translateExceptionIfPossible(HibernateJpaDialect.java:233)\r\n\tat org.springframework.orm.jpa.AbstractEntityManagerFactoryBean.translateExceptionIfPossible(AbstractEntityManagerFactoryBean.java:551)\r\n\tat org.springframework.dao.support.ChainedPersistenceExceptionTranslator.translateExceptionIfPossible(ChainedPersistenceExceptionTranslator.java:61)\r\n\tat org.springframework.dao.support.DataAccessUtils.translateIfNecessary(DataAccessUtils.java:242)\r\n\tat org.springframework.dao.support.PersistenceExceptionTranslationInterceptor.invoke(PersistenceExceptionTranslationInterceptor.java:152)\r\n\tat org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)\r\n\tat org.springframework.data.jpa.repository.support.CrudMethodMetadataPostProcessor$CrudMethodMetadataPopulatingMethodInterceptor.invoke(CrudMethodMetadataPostProcessor.java:174)\r\n\tat org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)\r\n\tat org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:97)\r\n\tat org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)\r\n\tat org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:215)\r\n\tat com.sun.proxy.$Proxy102.save(Unknown Source)\r\n\tat com.codewithnaman.service.user.CreateUserService.createUser(CreateUserService.java:27)\r\n\tat com.codewithnaman.controller.user.CreateUserController.registerUserUser(CreateUserController.java:25)\r\n\tat sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)\r\n\tat sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)\r\n\tat sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)\r\n\tat java.lang.reflect.Method.invoke(Method.java:498)\r\n\tat org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:197)\r\n\tat org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:141)\r\n\tat org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:106)\r\n\tat org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:894)\r\n\tat org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:808)\r\n\tat org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87)\r\n\tat org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1061)\r\n\tat org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:961)\r\n\tat org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1006)\r\n\tat org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:909)\r\n\tat javax.servlet.http.HttpServlet.service(HttpServlet.java:652)\r\n\tat org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:883)\r\n\tat javax.servlet.http.HttpServlet.service(HttpServlet.java:733)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:231)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)\r\n\tat org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)\r\n\tat org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:100)\r\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)\r\n\tat org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:93)\r\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)\r\n\tat org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:201)\r\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)\r\n\tat org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:202)\r\n\tat org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:97)\r\n\tat org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:542)\r\n\tat org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:143)\r\n\tat org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92)\r\n\tat org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:78)\r\n\tat org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:343)\r\n\tat org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:374)\r\n\tat org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:65)\r\n\tat org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:888)\r\n\tat org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1597)\r\n\tat org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49)\r\n\tat java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)\r\n\tat java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)\r\n\tat org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)\r\n\tat java.lang.Thread.run(Thread.java:748)\r\nCaused by: org.hibernate.PropertyValueException: not-null property references a null or transient value : com.codewithnaman.entity.User.userName\r\n\tat org.hibernate.engine.internal.Nullability.checkNullability(Nullability.java:111)\r\n\tat org.hibernate.engine.internal.Nullability.checkNullability(Nullability.java:55)\r\n\tat org.hibernate.action.internal.AbstractEntityInsertAction.nullifyTransientReferencesIfNotAlready(AbstractEntityInsertAction.java:116)\r\n\tat org.hibernate.action.internal.EntityIdentityInsertAction.execute(EntityIdentityInsertAction.java:72)\r\n\tat org.hibernate.engine.spi.ActionQueue.execute(ActionQueue.java:645)\r\n\tat org.hibernate.engine.spi.ActionQueue.addResolvedEntityInsertAction(ActionQueue.java:282)\r\n\tat org.hibernate.engine.spi.ActionQueue.addInsertAction(ActionQueue.java:263)\r\n\tat org.hibernate.engine.spi.ActionQueue.addAction(ActionQueue.java:317)\r\n\tat org.hibernate.event.internal.AbstractSaveEventListener.addInsertAction(AbstractSaveEventListener.java:330)\r\n\tat org.hibernate.event.internal.AbstractSaveEventListener.performSaveOrReplicate(AbstractSaveEventListener.java:287)\r\n\tat org.hibernate.event.internal.AbstractSaveEventListener.performSave(AbstractSaveEventListener.java:193)\r\n\tat org.hibernate.event.internal.AbstractSaveEventListener.saveWithGeneratedId(AbstractSaveEventListener.java:123)\r\n\tat org.hibernate.event.internal.DefaultPersistEventListener.entityIsTransient(DefaultPersistEventListener.java:185)\r\n\tat org.hibernate.event.internal.DefaultPersistEventListener.onPersist(DefaultPersistEventListener.java:128)\r\n\tat org.hibernate.event.internal.DefaultPersistEventListener.onPersist(DefaultPersistEventListener.java:55)\r\n\tat org.hibernate.event.service.internal.EventListenerGroupImpl.fireEventOnEachListener(EventListenerGroupImpl.java:102)\r\n\tat org.hibernate.internal.SessionImpl.firePersist(SessionImpl.java:720)\r\n\tat org.hibernate.internal.SessionImpl.persist(SessionImpl.java:706)\r\n\tat sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)\r\n\tat sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)\r\n\tat sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)\r\n\tat java.lang.reflect.Method.invoke(Method.java:498)\r\n\tat org.springframework.orm.jpa.ExtendedEntityManagerCreator$ExtendedEntityManagerInvocationHandler.invoke(ExtendedEntityManagerCreator.java:362)\r\n\tat com.sun.proxy.$Proxy98.persist(Unknown Source)\r\n\tat sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)\r\n\tat sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)\r\n\tat sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)\r\n\tat java.lang.reflect.Method.invoke(Method.java:498)\r\n\tat org.springframework.orm.jpa.SharedEntityManagerCreator$SharedEntityManagerInvocationHandler.invoke(SharedEntityManagerCreator.java:311)\r\n\tat com.sun.proxy.$Proxy98.persist(Unknown Source)\r\n\tat org.springframework.data.jpa.repository.support.SimpleJpaRepository.save(SimpleJpaRepository.java:557)\r\n\tat sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)\r\n\tat sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)\r\n\tat sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)\r\n\tat java.lang.reflect.Method.invoke(Method.java:498)\r\n\tat org.springframework.data.repository.core.support.RepositoryMethodInvoker$RepositoryFragmentMethodInvoker.lambda$new$0(RepositoryMethodInvoker.java:289)\r\n\tat org.springframework.data.repository.core.support.RepositoryMethodInvoker.doInvoke(RepositoryMethodInvoker.java:137)\r\n\tat org.springframework.data.repository.core.support.RepositoryMethodInvoker.invoke(RepositoryMethodInvoker.java:121)\r\n\tat org.springframework.data.repository.core.support.RepositoryComposition$RepositoryFragments.invoke(RepositoryComposition.java:524)\r\n\tat org.springframework.data.repository.core.support.RepositoryComposition.invoke(RepositoryComposition.java:285)\r\n\tat org.springframework.data.repository.core.support.RepositoryFactorySupport$ImplementationMethodExecutionInterceptor.invoke(RepositoryFactorySupport.java:531)\r\n\tat org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)\r\n\tat org.springframework.data.repository.core.support.QueryExecutorMethodInterceptor.doInvoke(QueryExecutorMethodInterceptor.java:156)\r\n\tat org.springframework.data.repository.core.support.QueryExecutorMethodInterceptor.invoke(QueryExecutorMethodInterceptor.java:131)\r\n\tat org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)\r\n\tat org.springframework.transaction.interceptor.TransactionInterceptor$1.proceedWithInvocation(TransactionInterceptor.java:123)\r\n\tat org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction(TransactionAspectSupport.java:388)\r\n\tat org.springframework.transaction.interceptor.TransactionInterceptor.invoke(TransactionInterceptor.java:119)\r\n\tat org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)\r\n\tat org.springframework.dao.support.PersistenceExceptionTranslationInterceptor.invoke(PersistenceExceptionTranslationInterceptor.java:137)\r\n\t... 59 more\r\n",
  "message": "not-null property references a null or transient value : com.codewithnaman.entity.User.userName; nested exception is org.hibernate.PropertyValueException: not-null property references a null or transient value : com.codewithnaman.entity.User.userName",
  "path": "/users"
}

Response code: 500; Time: 131ms; Content length: 12711 bytes
```

Here we are getting spring response and the exception from the JPA framework; But what if we want to validate the request
input before making any call to database or service. Let's see how we can do that:
1. We need to annotate the Request DTO object with validation API annotation like @NotNull, @NotEmpty, @Min etc. present
in package `javax.validation.constraints`.
2. We need to annotate the DTO object in the controller with @Valid annotation.

Let's see above two steps in our code:
1. Annotated DTO class:
```java
package com.codewithnaman.dto.user;

import lombok.Data;

import javax.validation.constraints.NotEmpty;
import javax.validation.constraints.Size;

@Data
public class CreateUserRequest {

    @NotEmpty
    @Size(min = 5,max = 255)
    private String userName;

    @NotEmpty
    @Size(min = 3,max = 255)
    private String firstName;

    @NotEmpty
    @Size(min = 2,max = 255)
    private String lastName;
}
```
2. Annotate method in controller:
```text
    @PostMapping("/users")
    public ResponseEntity<Object> registerUserUser(@RequestBody @Valid CreateUserRequest createUserRequest) {
        User registeredUser = this.createUserService.createUser(createUserRequest);
        URI registeredUserUri = ServletUriComponentsBuilder
                .fromCurrentRequest()
                .path("/{id}")
                .buildAndExpand(registeredUser.getId())
                .toUri();
        return ResponseEntity.created(registeredUserUri).build();
    }
```

Let's now hit the request and see what changes are there in output.

**Request :**
```text
POST http://localhost:8080/users
Content-Type: application/json

{
"firstName":"user1",
"lastName":"user1"
}
```
**Response :**
```text
POST http://localhost:8080/users

HTTP/1.1 400 
Content-Type: application/json
Transfer-Encoding: chunked
Date: Tue, 29 Dec 2020 04:51:54 GMT
Connection: close

{
  "timestamp": "2020-12-29T04:51:54.778+00:00",
  "status": 400,
  "error": "Bad Request",
  "trace": "org.springframework.web.bind.MethodArgumentNotValidException: Validation failed for argument [0] in public org.springframework.http.ResponseEntity<java.lang.Object> com.codewithnaman.controller.user.CreateUserController.registerUserUser(com.codewithnaman.dto.user.CreateUserRequest): [Field error in object 'createUserRequest' on field 'userName': rejected value [null]; codes [NotEmpty.createUserRequest.userName,NotEmpty.userName,NotEmpty.java.lang.String,NotEmpty]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [createUserRequest.userName,userName]; arguments []; default message [userName]]; default message [must not be empty]] \r\n\tat org.springframework.web.servlet.mvc.method.annotation.RequestResponseBodyMethodProcessor.resolveArgument(RequestResponseBodyMethodProcessor.java:139)\r\n\tat org.springframework.web.method.support.HandlerMethodArgumentResolverComposite.resolveArgument(HandlerMethodArgumentResolverComposite.java:121)\r\n\tat org.springframework.web.method.support.InvocableHandlerMethod.getMethodArgumentValues(InvocableHandlerMethod.java:170)\r\n\tat org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:137)\r\n\tat org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:106)\r\n\tat org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:894)\r\n\tat org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:808)\r\n\tat org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87)\r\n\tat org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1061)\r\n\tat org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:961)\r\n\tat org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1006)\r\n\tat org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:909)\r\n\tat javax.servlet.http.HttpServlet.service(HttpServlet.java:652)\r\n\tat org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:883)\r\n\tat javax.servlet.http.HttpServlet.service(HttpServlet.java:733)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:231)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)\r\n\tat org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)\r\n\tat org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:100)\r\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)\r\n\tat org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:93)\r\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)\r\n\tat org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:201)\r\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)\r\n\tat org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:202)\r\n\tat org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:97)\r\n\tat org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:542)\r\n\tat org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:143)\r\n\tat org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92)\r\n\tat org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:78)\r\n\tat org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:343)\r\n\tat org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:374)\r\n\tat org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:65)\r\n\tat org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:888)\r\n\tat org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1597)\r\n\tat org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49)\r\n\tat java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)\r\n\tat java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)\r\n\tat org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)\r\n\tat java.lang.Thread.run(Thread.java:748)\r\n",
  "message": "Validation failed for object='createUserRequest'. Error count: 1",
  "errors": [
    {
      "codes": [
        "NotEmpty.createUserRequest.userName",
        "NotEmpty.userName",
        "NotEmpty.java.lang.String",
        "NotEmpty"
      ],
      "arguments": [
        {
          "codes": [
            "createUserRequest.userName",
            "userName"
          ],
          "arguments": null,
          "defaultMessage": "userName",
          "code": "userName"
        }
      ],
      "defaultMessage": "must not be empty",
      "objectName": "createUserRequest",
      "field": "userName",
      "rejectedValue": null,
      "bindingFailure": false,
      "code": "NotEmpty"
    }
  ],
  "path": "/users"
}

Response code: 400; Time: 284ms; Content length: 6105 bytes
```

We can see the error code change to Bad Request; But still the response is too verbose. Let's Create our own response
for any field validation failure. To achieve the custom response we will do below steps:
1. We will extend class ResponseEntityExceptionHandler and override the handleMethodArgumentNotValid.
2. Return our own format response from this.

Let's have a look on our implementation for this:
```java
package com.codewithnaman.exception.handlers;

import com.codewithnaman.dto.error.FieldValidationErrorResponse;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.context.request.WebRequest;
import org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler;

import java.util.ArrayList;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;

@ControllerAdvice
public class FieldValidationExceptionHandler extends ResponseEntityExceptionHandler {

    @Override
    protected ResponseEntity<Object> handleMethodArgumentNotValid(MethodArgumentNotValidException ex,
                                                                  HttpHeaders headers,
                                                                  HttpStatus status,
                                                                  WebRequest request) {
        List<FieldValidationErrorResponse> fieldValidationResponse = new ArrayList<>();
        Map<String, String> fieldErrors = new LinkedHashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(fieldError ->
                fieldErrors.computeIfAbsent(fieldError.getField(), key -> fieldError.getDefaultMessage())
        );
        fieldErrors.forEach((key, value) -> fieldValidationResponse.add(new FieldValidationErrorResponse(key,value)));
        return new ResponseEntity(fieldValidationResponse,HttpStatus.BAD_REQUEST);
    }
}
```

Let's send hte request now and see the response:

**Request :**
```text
POST http://localhost:8080/users
Content-Type: application/json

{
"firstName":"u",
"lastName":"user1"
}

```
**Response :**
```text
POST http://localhost:8080/users

HTTP/1.1 400 
Content-Type: application/json
Transfer-Encoding: chunked
Date: Tue, 29 Dec 2020 05:50:49 GMT
Connection: close

[
  {
    "fieldName": "userName",
    "errorMessage": "must not be empty"
  },
  {
    "fieldName": "firstName",
    "errorMessage": "size must be between 3 and 255"
  }
]

Response code: 400; Time: 220ms; Content length: 135 bytes
```
We the HTTP status code as Bad Request, and a nice customized message body which will give better information about error
occurred while validation request.

After this we have implemented all the resource endpoints for the user and their todo. Let's see endpoint we expose from
our application:

1. Create User
```text
POST http://localhost:8080/users
Content-Type: application/json

{
"userName": "user2",
"firstName": "user2",
"lastName": "user2"
}
```

2. Retrieve User
```text
###
GET http://localhost:8080/users

GET http://localhost:8080/users/{searchBy}/{idOrUserName}
```

3. Remove User
```text
DELETE http://localhost:8080/users/{idOrUserName}
```

1. Create User Task
```text
POST http://localhost:8080/users/{userId}/tasks
Content-Type: application/json

{
  "taskName": "TEST TASK",
  "taskDescription": "TASK Desp is not found"
}
```

2. Retrieve User Task
```text
GET http://localhost:8080/users/{userId}/tasks/

GET http://localhost:8080/users/{userId}/tasks/{taskId}
```

3. Update User Tasks
```text
PUT http://localhost:8080/users/1/tasks/2/IN_PROGRESS
Content-Type: application/json
```

4. Delete User Task
```text
DELETE http://localhost:8080/users/{userId}/tasks/{taskId}
```

The APIs and concepts; we used this till point we have checked in branch [here](https://github.com/naman-gupta810/spring-rest-tutorials/tree/intial-developed-rest-apis).

Let's move ahead and integrate the HATEOAS with Our API.

### Integrating HATEOAS with our API
HATEOAS(Hypermedia As The Engine Of Application State) will tell that we can provide the links associated with a resource
for further exploration on the web for that particular resource. Let's understand this by our application example.

1. We can provide the link for tasks of user in user resource.
2. We can provide the user link in task description retrieval API.
3. We can provide the link of each user in the users list.
4. We can provide the link of each task in tasks list of user.

We will do for one example; Where we will add user link to task. Let's see code for this below:
```text
  @GetMapping("users/{userId}/tasks/{taskId}")
    public EntityModel<Todo> findAllUserTasks(@PathVariable String userId, @PathVariable Long taskId) throws EntityNotFoundException {
        log.info("Request Received for User ID : {} and Task ID: {}", userId, taskId);
        Todo todo = retrieveUserTaskService.findUserTaskByTaskID(userId, taskId);
        EntityModel<Todo> response = EntityModel.of(todo);
        response.add(linkTo(methodOn(RetrieveUserController.class).getUser("id",userId)).withRel("user"));
        log.info("Task found for the User ID : {} and Task ID : {}", userId, taskId);
        return response;
    }
```

Now Let's hit the endpoint and see this. We hit the endpoint `GET http://localhost:8080/users/1/tasks/2` which response
is:
```text
{
  "id": 2,
  "taskName": "TEST TASK",
  "taskDescription": "TASK Desp is not found",
  "taskStatus": "IN_PROGRESS",
  "startTime": "2020-12-29T14:22:25.74",
  "completionTime": null,
  "metadata": {
    "createDate": "2020-12-29T14:22:25.212",
    "updateDate": "2020-12-29T14:22:25.74"
  },
  "_links": {
    "user": {
      "href": "http://localhost:8080/users/id/1"
    }
  }
}
```
So we embedded the link; We have mainly four classes for this.
* RepresentationModel - This we extend on our response class if we need to add the link to resource itself and put into a 
  collection.
* EntityModel - This is used to return a single resource response and add links to it.
* CollectionModel - This is used when we want to return collection of resources and want to attach the link for collection.
* PagedModel - This is used when we are using pagination for resources. 

### Advanced REST API Concepts
#### Content Negotiation
Right now with every request and response we are using application/json. Just consider the case where we want to consume
the response in application/xml format. To achieve this with our application, we need to add `jackson-dataformat-xml` 
library and restart the application. Let's add this to our `pom.xml` and hit the request to get the response as the xml.

**Request :**
```text
GET http://localhost:8080/users
Accept: application/xml
```
**Response :**
```text
GET http://localhost:8080/users

HTTP/1.1 200 
Content-Type: application/xml;charset=UTF-8
Transfer-Encoding: chunked
Date: Wed, 30 Dec 2020 03:20:52 GMT
Keep-Alive: timeout=60
Connection: keep-alive

<List>
    <item>
        <id>1</id>
        <userName>admin</userName>
        <firstName>admin</firstName>
        <lastName>admin</lastName>
        <metadata>
            <createDate>2020-12-30T08:49:46.141</createDate>
            <updateDate>2020-12-30T08:49:46.141</updateDate>
        </metadata>
    </item>
    <item>
        <id>2</id>
        <userName>user</userName>
        <firstName>user</firstName>
        <lastName>user</lastName>
        <metadata>
            <createDate>2020-12-30T08:49:46.141</createDate>
            <updateDate>2020-12-30T08:49:46.141</updateDate>
        </metadata>
    </item>
</List>

Response code: 200; Time: 212ms; Content length: 450 bytes
```
By adding the library in classpath; Spring configures itself that it can provide and take input in the form of JSON
and XML. For other formats there are libraries available with jackson; if we want to support those we need to just add
those libraries in classpath.

For few formats like YAML there is no implicit converter available; in that case we need to create converter class from
class `AbstractJackson2HttpMessageConverter` and create bean out of it; which will take mapper, and then we can enable the 
conversion from object to that particular format.

#### API Documentation
As, we discussed in the introduction that REST does not have any standard format for the documentation like SOAP. To 
provide the documentation to consumers; there are standard define as part of OpenAPI specification. Let's integrate
tooling which will detect the endpoints in our application and provide a basic documentation for it. For this purpose 
we will use swagger. Let's add the dependencies for Swagger and UI for it and have a look on the same.

We have added below dependencies for the swagger in our `pom.xml`.
```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>${swagger.version}</version>
</dependency>
```

We provided that use the OpenAPI specification 3.0 using the application.yaml configuration.
```yaml
springfox:
  documentation:
    open-api:
      enabled: true
```
Let's start the application and see what we get. After starting the application we can access the API documentation 
on URL `http://localhost:8080/v3/api-docs` and we can access the swagger UI on URL `http://localhost:8080/swagger-ui`.

Let's see some of below screenshot:

**API Docs URL:**
![API Docs](images/Open%20API%20Docs.JPG)

**Swagger UI:**
![Swagger UI](images/Swagger%20UI.JPG)

In this we can see by default some title and other information has been given. Let's provide our custom output and see
the changes in the API Documentation.

For changing the API documentation title and related properties we need to create bean of Docket and need to pass the 
`ApiInfo`. Let's see the code for same.
```java
package com.codewithnaman.configuration;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;

import java.util.ArrayList;

@Configuration
public class TodoApiDefinitionDocumentation {

    public static final Contact CONTACT
            = new Contact(
            "Naman Gupta",
            "http://www.codewithnaman.com",
            "tech.naman.gupta@gmail.com");
    public static final ApiInfo API_INFO
            = new ApiInfo(
            "ToDo Application Service Documentation",
            "This Contains the ToDo Application endpoints in the OpenAPI Specification format 3.0",
            "1.0",
            "urn:tos",
            CONTACT,
            "Apache 2.0",
            "http://www.apache.org/licenses/LICENSE-2.0",
            new ArrayList<>());

    @Bean
    public Docket apiDocket(){
        return new Docket(DocumentationType.OAS_30).apiInfo(API_INFO).select().build();
    }
}
```

This we just want to expose the api which are exposed by our package then we will do the select on package of Docket.
What we changed for this is below:
```java
   @Bean
    public Docket apiDocket(){
        return new Docket(DocumentationType.OAS_30)
                .apiInfo(API_INFO)
                .select().apis(RequestHandlerSelectors.basePackage("com.codewithnaman"))
                .build();
    }
```

Let's create the Documentation for our APIs. We are creating one for example on CreateUserTaskController and see how it
affects our documentation. We have created the documenation in classes 
[CreateUserTaskController](src/main/java/com/codewithnaman/controller/todo/CreateUserTaskController.java) and 
[CreateUserTaskRequest](src/main/java/com/codewithnaman/dto/todo/CreateUserTaskRequest.java). After which the screenshot
of the api-doc is below:
![Post API DOC](images/Post%20Api%20Doc.JPG)

#### Filtering
In this section we will talk about filtering of fields in the response from POJO.

##### Static Filtering
This we have done in our Sample application where we added the @JsonIgnore on property of User tasks and that will be 
not rendered as the part of response. To see example [Click Here](src/main/java/com/codewithnaman/entity/User.java)

##### Dynamic Filtering
Dynamic filtering is used when we want to filter the fields depending on the request. Let's for example we don't want
to include the record metadata when we are fetching all user. Let's modify the method and see it:
```java
    @GetMapping("users")
    public MappingJacksonValue getUser() {
        log.info("Get All user request received");
        List<User> users = retrieveUserService.getAllUsers();
        SimpleBeanPropertyFilter recordMetaDataFilter = SimpleBeanPropertyFilter.serializeAllExcept("metadata");
        FilterProvider userFieldFilter = new SimpleFilterProvider().addFilter("RecordMetaDataFilter", recordMetaDataFilter);
        MappingJacksonValue response = new MappingJacksonValue(users);
        response.setFilters(userFieldFilter);
        log.info("Found number of Users : {}", users.size());
        return response;
    }
```
In above method we created a property filter which will serialize everything except metadata, and then we provided the value
to MappingJacksonValue which will filter the recordmetadata. In addition to this work we need to annotate our User class
with `@JsonFilter("RecordMetaDataFilter")`, where RecordMetaDataFilter is the id of filter creating in controller method.

#### Versioning
Let's undestand how we can version our APIs. Just consider we have one API already exist, but we want to expose the 
endpoint with additional information; How we can do that. Also we need to support existing users of API, Then in this 
case API versioning helps us. There are 4 ways to do this. We will see each one with example.

**1. URI Versioning :**
In this we specify the version in the URI itself and mapped to controller methods. For Example 
[UriTypeVersioningController](src/main/java/com/codewithnaman/controller/versioning/UriTypeVersioningController.java)

Request and response after hitting the URI:
```text
###
GET http://localhost:8080/v1/card-list

[
  "DEBIT",
  "CREDIT"
]

###
GET http://localhost:8080/v2/card-list

[
  "DEBIT",
  "CREDIT",
  "PRE-PAID"
]
```

**2. Parameter Versioning :**
In this we specify the version in query parameter. Let's see example for this in 
[ParameterVersioningController](src/main/java/com/codewithnaman/controller/versioning/ParameterVersioningController.java).

Let's hit the request and see:
```text
###
GET http://localhost:8080/card-list?version=1

[
  "DEBIT",
  "CREDIT"
]

###
GET http://localhost:8080/card-list?version=2

[
  "DEBIT",
  "CREDIT",
  "PRE-PAID"
]
```

**3. Custom Header Versioning :** In header versioning we pass the version parameter as part of header in the request.
Let's see example for this in 
[HeaderVersioningController](src/main/java/com/codewithnaman/controller/versioning/HeaderVersioningController.java).

Let's hit the request and see:
```text
###
GET http://localhost:8080/card-list
X-API-VERSION: 1

[
  "DEBIT",
  "CREDIT"
]

###
GET http://localhost:8080/card-list
X-API-VERSION: 2

[
  "DEBIT",
  "CREDIT",
  "PRE-PAID"
]
```   

**4. Media Type or Accept-Header or MIME Type Versioning :** In this we will pass the version parameter as the 
accept header the value might be like com.codewithnaman-v1+json and com.codewithnaman-v2+json. Let's see example
of this in 
[MimeTypeVersioningController](src/main/java/com/codewithnaman/controller/versioning/MimeTypeVersioningController.java).

Let's hit the request and see the response for this:
```text
###
GET http://localhost:8080/card-list
Accept: application/vnd.com.codewithnaman-v1+json

[
  "DEBIT",
  "CREDIT"
]

###
GET http://localhost:8080/card-list
Accept: application/vnd.com.codewithnaman-v2+json
X-API-VERSION: 2

[
  "DEBIT",
  "CREDIT",
  "PRE-PAID"
]
```

There is no good or bad approach in above versioning schemes. But we need choose our application versioning scheme on
below parameters:

1. URI Pollution: In header and query param versioning we are adding it to URI; so URI may be longer, and we have URI 
length limits.

2. Misuse of HTTP header: In Header API version or Accept header; we are using for versioning purpose which is not
headers for intended to use.
   
3. Caching: The versioning in URI and Query Params can be cached to browser; So sometimes it might need to cache clear
for hitting the URI from application or JS script cache.
   
4. GET method hit from browser: URI and Query param can hit the GET method from the browser; while other can't.

5. API Documentation: We need to put them properly in the API documentation for the consumers.

### Richardson Maturity Model
Richardson introduced a mode to measure how mature an REST API application model is which is known as Richarson Maturity
Model. There are 4 Levels in this model which tells us the maturity or REST API in our application.

**Level 0 (Swamp of Pox)**

In this we have a service already exposing a resource via its URI using only one HTTP verb (usually GET or POST). 
In this phase, you only use HTTP as a transport system (think of it as an RPC using HTTP or SOAP).

**Level 1 (Resources)**

In this we expose the resource as URIs where we don't use the proper HTTP methods.

**Level 2 (HTTP Verbs)**

Level 1 + We start using proper HTTP methods with proper HTTP codes.

**Level 3 (Multimedia)**

Level 2 + HATEOAS. In this we start putting related links in the response. 

### Best Practices with REST API
1. Try to think about from the perspective of Consumer of APIs. Is it Mobile or Web or Data Crawler application. Proper
documentation of API also helps consumer to understand the API and consume it.
   
2. Use Proper HTTP request method and HTTP Response Codes.

3. Use plurals for the resource.

4. Use Noun for the resource; In some cases you have verb but that need to associate with the noun resource; which shows
what action is going to performed and what resource we are performing.