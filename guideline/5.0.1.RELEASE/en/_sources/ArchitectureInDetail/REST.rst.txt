RESTful Web Service
================================================================================

.. only:: html

 .. contents:: Table of Contents
    :depth: 3
    :local:

.. _RESTOverview:

Overview
--------------------------------------------------------------------------------
This section explains the basic concept of RESTful Web Service and its development by using Spring MVC.

Refer to the following for basic description of architecture, design and implementation of RESTful Web Service

* | ":ref:`RESTAboutResourceOrientedArchitecture`"
  | Basic architecture of RESTful Web Service is explained.

* | ":ref:`RESTHowToDesign`"
  | Points to be considered while designing a RESTful Web Service are explained.

* | ":ref:`RESTHowToUse`"
  | Application structure of RESTful Web Service and API implementation methods are explained.

|

.. _RESTOverviewAboutRESTfulWebService:

What is RESTful Web Service
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| REST is an abbreviation of "\ **RE**\presentational \ **S**\tate \ **T**\ransfer", and is one of the \ **architecture styles**\  
| for building an application, wherein data is exchanged between client and server.
| REST architecture style consists of various important fundamental rules and the services which are in accordance with these rules (system etc.) are expressed as \ **RESTful**\ .
| In other words, "RESTful Web Service" is a Web service that is built in accordance with the fundamental rules of REST.

| \ **The concept of "resource" is of prime importance in RESTful Web Service.**\
| In RESTful Web Service, the information that should be provided to the client is extracted as "resource", from the information stored in database etc. and the CRUD operation for this extracted "resource" is provided to the client using HTTP methods (POST/GET/PUT/DELETE).
| The CRUD operation for "resource" is called "REST API" or "RESTful API" and is mentioned as "REST API" in this guideline.
| Further, JSON or XML that have higher message visibility and data structure expressivity, are used as the message formats while exchanging resources between client and server.

| System configuration of the application that uses RESTful Web Service mainly consists of following 2 patterns.
| The basic architecture for exchanging resources between client and server is explained by using ":ref:`RESTAboutResourceOrientedArchitecture`".

 .. figure:: ./images_REST/RESTExampleSystemConstitution.png
   :alt: Constitution of RESTful Web Service
   :width: 100%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | A resource is directly exchanged between client application with user interface and RESTful Web Service.
        | This pattern is used to separate user interface dependent logic with higher number of requirement & specification changes and the logic for a data model which is more universal with less number of changes.
    * - | (2)
      - | Rather than directly exchanging the resource with client applications having user interface, the resource is exchanged between systems.
        | This pattern is used while building a system wherein, the business data stored by each system is managed centrally.


|

.. _RESTOverviewAboutRESTfulWebServiceDevelopment:

RESTful Web Service development
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
RESTful Web Service is developed in TERASOLUNA Server Framework for Java (5.x) using Spring MVC functionalities.

| The common functionalities necessary for RESTful Web Service development are built in Spring MVC by default.
| As a result, RESTful Web Service development can be initiated without adding any specific settings or implementations.

| Main common functionalities built in Spring MVC by default, are given below.
| These functionalities can only be enabled by specifying annotations in the methods of the Controller that provides REST API.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Function overview
    * - | (1)
      - | It is a function which converts the JSON or XML format message set in request BODY, to Resource object (JavaBean) and delivers it to the Controller class method (REST API).
    * - | (2)
      - | It is a function which implements input validation for the value stored in Resource object (JavaBean) that has been converted from message.
    * - | (3)
      - | It is a function which converts Resource object (JavaBean) returned from the Controller class method (REST API) to JSON or XML format and sets it in response BODY.

 .. note:: **Exception handling**

    It is necessary to implement exception handling for each project since a generic functionality for the same is not provided by Spring MVC.
    For details on exception handling, refer to ":ref:`RESTHowToUseExceptionHandling`".

|

| When RESTful Web Service is developed using Spring MVC, the application is configured as given below. Among these, implementation is necessary for the portion marked with red frame.

 .. figure:: ./images_REST/RESTOverviewApplicationConstitutionOnSpringMVC.png
   :alt: Application constitution of RESTful Web Service on Spring MVC
   :width: 100%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 70

    * - Sr. No.
      - Process layer
      - Description
    * - | (1)
      - | Spring MVC
        | (Framework)
      - | Spring MVC receives a request from client and determines the REST API (processing method of Controller) to be called.
    * - | (2)
      - | 
        | 
      - | Spring MVC converts the JSON format message specified in request BODY to Resource object by using \ ``HttpMessageConverter``\ .
    * - | (3)
      - | 
        | 
      - | Spring MVC performs input validation for the value stored in Resource object using \ ``Validator``\ .
    * - | (4)
      - | 
        | 
      - | Spring MVC calls REST API.
        | Here, the Resource that has been converted from JSON and for which input validation is carried out, is delivered to REST API.
    * - | (5)
      - | REST API
      - | REST API calls Service method and performs the process for DomainObject such as Entity etc.
    * - | (6)
      - | 
      - | Service method calls the Repository method and performs CRUD process for the DomainObject such as Entity etc.
    * - | (7)
      - | Spring MVC
        | (Framework)
      - | Spring MVC converts the Resource object returned from REST API to JSON format message, by using \ ``HttpMessageConverter``\ .
    * - | (8)
      - | 
        | 
      - | Spring MVC sets JSON format message in response BODY and responds to client.


|

Configuration for RESTful Web Service module
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| A lot of RESTful Web Service specific processing can be entrusted to Spring MVC by using the functionalities provided by the framework.
| Therefore, configuration of the module to be developed is almost same as the development of conventional Web application that responds with HTML.
| Configuration elements of the module are explained below.

 .. figure:: ./images_REST/RESTModuleConstitution.png
   :alt: Constitution of Modules
   :width: 100%

* **Module for application layer**

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 20 70
    
        * - Sr. No.
          - Module name
          - Description
        * - | (1)
          - | Controller class
          - | A class that provides REST API.
            | Controller class is created by resource unit and specifies end points (URI) of REST API for each resource.
            | CRUD process for the resource is implemented by delegating it to the Service of domain layer.
        * - | (2)
          - | Resource class
          - | Java Bean representing JSON (or XML) that acts as I/O for REST API.
            | Annotation for Bean Validation and annotation for controlling JSON or XML format are specified in this class.
        * - | (3)
          - | Validator Class
            | (Optional)
          - | Class that implements correlation validation for input value.
            | If the correlation validation for input value is unnecessary, this class need not be created. Hence, it is considered as optional.
            | For input value correlation validation, refer to ":doc:`Validation`".
        * - | (4)
          - | Helper Class
            | (Optional)
          - | Class which implements the process that assists the process to be performed by the Controller.
            | This class is created with the aim of simplifying the Controller processing.
            | Basically, it implements a method that performs conversion of Resource object and DomainObject models.
            | If the model can be converted simply by using copy of the value, ":doc:`Utilities/Dozer`" may be used without creating the Helper class. Hence, it is considered as optional.

|

* **Domain layer module**

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90
    
        * - Sr. No.
          - Description
        * - | (5)
          - | The description is beyond the scope of this section since the module implemented in the domain layer is independent of application type.
            | For role of each module, refer to ":doc:`../Overview/ApplicationLayering`" and for domain layer development, refer to ":doc:`../ImplementationAtEachLayer/DomainLayer`".

|

* **Infrastructure layer module**

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90
    
        * - Sr. No
          - Description
        * - | (6)
          - | The description is beyond the scope of this section since the module implemented in the infrastructure layer is independent of application type.
            | Refer to ":doc:`../Overview/ApplicationLayering`" for role of each module and ":doc:`../ImplementationAtEachLayer/InfrastructureLayer`" for development of infrastructure layer.


|

REST API implementation sample
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Before giving a detailed explanation, an implementation sample of Resource class and Controller class is given below to let one understand the kind of class created in the application layer.
| The implementation sample given below is the REST API of Todo resource which is the topic of :doc:`../TutorialREST/index`.

 .. note::

    \ **It is strongly recommended to practice **\:doc:`../TutorialREST/index`\ ** first, before reading the detailed explanation.**\

    Aim of the tutorial is to emphasize the saying "Practice makes one perfect". Prior to detailed explanation, the user can gain the experience of actually practicing RESTful Web Service development using TERASOLUNA Server Framework for Java (5.x), with the help of this tutorial.
    When this firsthand experience of RESTful Web Service development is followed by reading the detailed explanation, the user gains a deeper understanding of the development.
    
    Especially when the user does not have any experience of RESTful Web Service development, it is recommended to follow a process in the order namely,  "Tutorial practice" --> "Detailed explanation of architecture, design and development (described in subsequent sections) --> "Tutorial revision (Re-practice)".

|

* Resources handled in implementation sample

 Resources handled in the implementation sample (Todo resources) are set in following JSON format.

 .. code-block:: json

    {
        "todoId" : "9aef3ee3-30d4-4a7c-be4a-bc184ca1d558",
        "todoTitle" : "Hello World!",
        "finished" : false,
        "createdAt" : "2014-02-25T02:21:48.493+0000"
    } 

|

* Resource class implementation sample

 Resource class is created as the JavaBean representing the Todo resources shown above.
 

 .. code-block:: java

    package todo.api.todo;

    import java.io.Serializable;
    import java.util.Date;
    
    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;
    
    public class TodoResource implements Serializable {

        private static final long serialVersionUID = 1L;

        private String todoId;
    
        @NotNull
        @Size(min = 1, max = 30)
        private String todoTitle;
    
        private boolean finished;
    
        private Date createdAt;
    
        public String getTodoId() {
            return todoId;
        }
    
        public void setTodoId(String todoId) {
            this.todoId = todoId;
        }
    
        public String getTodoTitle() {
            return todoTitle;
        }
    
        public void setTodoTitle(String todoTitle) {
            this.todoTitle = todoTitle;
        }
    
        public boolean isFinished() {
            return finished;
        }
    
        public void setFinished(boolean finished) {
            this.finished = finished;
        }
    
        public Date getCreatedAt() {
            return createdAt;
        }
    
        public void setCreatedAt(Date createdAt) {
            this.createdAt = createdAt;
        }
    }

|

* Implementation sample for Controller class (REST API)

 Following five REST APIs (Controller processing methods) are created for Todo resource.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.10\linewidth}|p{0.30\linewidth}|p{0.15\linewidth}|p{0.20\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 15 10 30 15 20

    * - | Sr. No.
      - | API Name
      - | HTTP
        | Method
      - | Path
      - | Status
        | Code
      - | Description
    * - | (1)
      - | GET Todos
      - | GET
      - | \ ``/api/v1/todos``\ 
      - | 200
        | (OK)
      - | All Todo resources are fetched.
    * - | (2)
      - | POST Todos
      - | POST
      - | \ ``/api/v1/todos``\ 
      - | 201
        | (Created)
      - | A new Todo resource is created.
    * - | (3)
      - | GET Todo
      - | GET
      - | \ ``/api/v1/todos/{todoId}``\ 
      - | 200
        | (OK)
      - | One Todo resource is fetched.
    * - | (4)
      - | PUT Todo
      - | PUT
      - | \ ``/api/v1/todos/{todoId}``\ 
      - | 200
        | (OK)
      - | Todo resource is updated to "completed".
    * - | (5)
      - | DELETE Todo
      - | DELETE
      - | \ ``/api/v1/todos/{todoId}``\ 
      - | 204
        | (No Content)
      - | Todo resource is deleted.

 .. code-block:: java
    :emphasize-lines: 30-34, 42-45, 51-55, 59-63, 68-72

    package todo.api.todo;

    import java.util.ArrayList;
    import java.util.Collection;
    import java.util.List;

    import javax.inject.Inject;

    import org.dozer.Mapper;
    import org.springframework.http.HttpStatus;

    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.PathVariable;
    import org.springframework.web.bind.annotation.RequestBody;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;

    import org.springframework.web.bind.annotation.ResponseStatus;
    import org.springframework.web.bind.annotation.RestController;

    import todo.domain.model.Todo;
    import todo.domain.service.todo.TodoService;

    @RestController
    @RequestMapping("todos")
    public class TodoRestController {
        @Inject
        TodoService todoService;
        @Inject
        Mapper beanMapper;

        // (1)
        @RequestMapping(method = RequestMethod.GET)

        @ResponseStatus(HttpStatus.OK)
        public List<TodoResource> getTodos() {
            Collection<Todo> todos = todoService.findAll();
            List<TodoResource> todoResources = new ArrayList<>();
            for (Todo todo : todos) {
                todoResources.add(beanMapper.map(todo, TodoResource.class));
            }
            return todoResources;
        }

        // (2)
        @RequestMapping(method = RequestMethod.POST)

        @ResponseStatus(HttpStatus.CREATED)
        public TodoResource postTodos(@RequestBody @Validated TodoResource todoResource) {
            Todo createdTodo = todoService.create(beanMapper.map(todoResource, Todo.class));
            TodoResource createdTodoResponse = beanMapper.map(createdTodo, TodoResource.class);
            return createdTodoResponse;
        }

        // (3)
        @RequestMapping(value="{todoId}", method = RequestMethod.GET)

        @ResponseStatus(HttpStatus.OK)
        public TodoResource getTodo(@PathVariable("todoId") String todoId) {
            Todo todo = todoService.findOne(todoId);
            TodoResource todoResource = beanMapper.map(todo, TodoResource.class);
            return todoResource;
        }

        // (4)
        @RequestMapping(value="{todoId}", method = RequestMethod.PUT)

        @ResponseStatus(HttpStatus.OK)
        public TodoResource putTodo(@PathVariable("todoId") String todoId) {
            Todo finishedTodo = todoService.finish(todoId);
            TodoResource finishedTodoResource = beanMapper.map(finishedTodo, TodoResource.class);
            return finishedTodoResource;
        }
        
        // (5)
        @RequestMapping(value="{todoId}", method = RequestMethod.DELETE)

        @ResponseStatus(HttpStatus.NO_CONTENT)
        public void deleteTodo(@PathVariable("todoId") String todoId) {
            todoService.delete(todoId);
        }

    }



|

.. _RESTAboutResourceOrientedArchitecture:

Architecture
--------------------------------------------------------------------------------
| This section explains the architecture for building a RESTful Web Service.

| Resource Oriented Architecture (ROA) is used as the architecture for building RESTful Web Service.
| ROA is an abbreviation of "\ **R**\esource \ **O**\riented \ **A**\rchitecture" and defines \ **the basic architecture for building a Web Service in accordance with REST architecture style (rules)**\ .
| It is important to thoroughly understand ROA architecture when creating RESTful Web Service.

| This section explains following 7 elements of ROA architecture.
| These form important architectural elements for building RESTful Web Service. However, it is not always necessary to apply all of these elements.
| Necessary elements should be applied after considering the characteristics of the application to be developed.

Following five architectural elements must be applied regardless of the application characteristics.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 35 55

    * - Sr. No.
      - Architecture
      - Architecture overview
    * - | (1)
      - | :ref:`RESTOverviewProvideResourceOnWeb`
      - | It is published as a Web resource through which information stored in the system is provided to the client.
    * - | (2)
      - | :ref:`RESTOverviewAssignURI`
      - |  URI (Universal Resource Identifier) that can uniquely identify a Web resource is assigned to the resource published to the client.
    * - | (3)
      - | :ref:`RESTOverviewOperatedByHttpMethod`
      - | Resource related operations are implemented by using different HTTP methods (GET, POST, PUT and DELETE).
    * - | (4)
      - | :ref:`RESTOverviewResourceRepresentationFormat`
      - | JSON or XML that represents the data structure, is used as resource format.
    * - | (5)
      - | :ref:`RESTOverviewHttpStatusCode`
      - | Appropriate HTTP status code is set in the response returned to the client.

|

Following two architectural elements are applied depending on the characteristics of an application.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 35 55

    * - Sr. No.
      - Architecture
      - Architectural elements
    * - | (6)
      - | :ref:`RESTOverviewClientServerCommunicateOnStateless`
      - | This element enables to perform the process only by the information requested from client, without retaining the application status on the server.
    * - | (7)
      - | :ref:`RESTOverviewHyperMediaLinksToRelatedResources`
      - | It includes links to other resources (URI) inside a resource that are related to the specified resource.


|

.. _RESTOverviewProvideResourceOnWeb:

Publishing as a resource on Web
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| **It is published as a resource on Web as the means to provide information stored in the system to client.**
| It signifies that resources can be accessed using HTTP protocol and URI is used as a method to identify resources.

For example, following information is published on the Web as resource, for a Web system providing shopping site.

* Product information
* Stock information
* Order information
* Member information
* Authentication information for each member (Login ID and password etc.)
* Order history information for each member
* Authentication history information for each member
* and more ...

|

.. _RESTOverviewAssignURI:

Identifying the resource using URI
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| **URI (Universal Resource Identifier) that can uniquely identify a resource on the Web, is assigned to the resource to be published to the client.**
| URL (Uniform Resource Locator), which is a subset of the URI, is actually used.

| In ROA, the ability to access a resource on the Web using URI, is called as "Addressability".
| It signifies that on using the same URI, the same resource can be accessed from anywhere.

| URI assigned to RESTful Web Service is a combination of "\ **a noun that indicates the type of resource**\" and "\ **a value (ID etc.) that uniquely identifies a resource**\ ".
| For example, URI of product information handled by a Web system that provides a shopping site, is given below.

* | \ `http://example.com/api/v1/items`\ 
  | "**items**" portion is the "noun that represents the type of resource". If there are multiple resources, a plural noun is used.
  | In the above example, a plural noun is specified to indicate the product information. It forms the URI for batch operation of product information. If replaced to a file system, it corresponds to a directory.

* | \ `http://example.com/api/v1/items/I312-535-01216`\
  | The part "**I312-535-01216**" in the above URI, represents "the value that identifies the resource" and varies for each resource.
  | In the above example, product ID is specified as the value for uniquely identifying product information. It acts as the URI used to handle specific product information. If replaced by a file system, it corresponds to the files stored in a directory.

|

.. warning::
 
   \ **Verbs that indicate operations cannot be included**\  in the URI assigned to RESTful Web Service are as shown below.
    
    * \ `http://example.com/api/v1/items?get&itemId=I312-535-01216`\
    * \ `http://example.com/api/v1/items?delete&itemId=I312-535-01216`\
    
    URI mentioned in the above example is not suitable to be assigned to RESTful Web Service since it includes verbs like \ **get**\  or \ **delete**\ .
    
    In RESTful Web Service, \ **Resource related operations are represented by using HTTP methods (GET, POST, PUT and DELETE).**\

|

.. _RESTOverviewOperatedByHttpMethod:

Resource operations using HTTP methods
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| **Resource operations can be performed by using HTTP methods (GET, POST, PUT, DELETE).**

| In ROA, HTTP methods are called as "Unified interface".
| It implies that HTTP methods can be executed for all the resources published on the Web and that the meaning of HTTP method does not change with each resource.

The association of resource operations assigned to HTTP methods and the post-conditions ensured by each operation, are explained below.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.35\linewidth}|p{0.35\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 35 35

    * - Sr. No.
      - HTTP method
      - Resource operations
      - Post-conditions that the operation should ensure
    * - | (1)
      - | GET
      - | Resource is fetched.
      - | Safety, idempotency.
    * - | (2)
      - | POST
      - | Resource is created.
      - | Server assigns the URI for created resource, this assigned URI is set to Location header of response and is returned to client.
    * - | (4)
      - | PUT
      - | Resource is created or updated.
      - | Idempotency.
    * - | (5)
      - | PATCH
      - | Resource difference is updated.
      - | Idempotency.
    * - | (6)
      - | DELETE
      - | Resource is deleted.
      - | Idempotency.
    * - | (7)
      - | HEAD
      - | Meta information of resource is fetched.
        | Same process as GET is performed and responds with header only.
      - | Safety, Idempotency.
    * - | (8)
      - | OPTIONS
      - | Responds with a list of HTTP methods that can be used for resources.
      - | Safety, idempotency.

 .. note:: **Ensuring safety and idempotency**
 
    When resource operation is performed using HTTP method, it is necessary to ensure "safety" and "idempotency" as post conditions.

    **[Safety]**

            It ensures that even if a particular value is multiplied several times by 1, the value does not change. (for example, if 10 is multiplied several times by 1, result remains 10).
            This guarantees that even if an operation is carried out for several times, resource status does not change.

    **[Idempotency]**

            It ensures that even if a value is multiplied a number of times by 0, the value remains 0 (for example, if 10 is multiplied a number of times or just once by 0, the result remains 0).
            This signifies that once an operation is performed, resource status does not change even if the same operation is performed later for a number of times.
            However, when another client is modifying the status of the same resource, idempotency need not be ensured and can be handled as a precondition error.
    

 .. tip:: **When client specifies the URI assigned to a resource for creating a resource**
 
    To create a resource, when the URI to be assigned to the resource is specified by client, \ **PUT method is called for the URI assigned to the resource to be created.**\

    When creating a resource using PUT method, the general operation is to,
    
     * Create a resource when no resource exists in the specified URI
     * Modify resource status when a resource already exists
    
    
    
    Following is the difference in process images while creating a resource using PUT and POST methods.
    
    **[Process image while creating a resource using PUT method]**

         .. figure:: ./images_REST/RESTCreatedNewResourceUsingByPutMethod.png
           :alt: Image of processing for creating new resource using by PUT method
           :width: 70%

         .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
         .. list-table::
            :header-rows: 1
            :widths: 10 90
        
            * - Sr. No.
              - Description
            * - | (1)
              - | PUT method is called by specifying URI (ID) of the resource to be created in URI.
            * - | (2)
              - | Entity is created for the ID specified in URI.
                | If the entity has already been created with same ID, the contents are updated.
            * - | (3)
              - | Created or updated resource is sent as a response.

    **[Process image while creating a resource using POST method]**

         .. figure:: ./images_REST/RESTCreatedNewResourceUsingByPostMethod.png
           :alt: Image of processing for create new resource using by POST method
           :width: 70%


         .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
         .. list-table::
            :header-rows: 1
            :widths: 10 90

            * - Sr. No.
              - Description
            * - | (1)
              - | POST method is called.
            * - | (2)
              - | ID that identifies the requested resource is generated.
            * - | (3)
              - | Entity for the ID generated in (2) is created.
            * - | (4)
              - | Created resource is sent as a response.
                | URI for accessing the generated resource is set in the Location header of response.
  
|

.. _RESTOverviewResourceRepresentationFormat:

Using an appropriate format
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
**JSON or XML that indicate data structure, are used for resource format.**

| However, formats other than JSON or XML can also be used depending on the type of resource.
| For example, a resource classified as statistical information can be published with line graph represented in image format (Binary data).

| When multiple formats are supported as resource formats, any of the following methods is used to change the format.


* **Changing the format using an extension.**

  | Response format can be changed by specifying the extension.
  | **This guideline recommends changing the format using extension.**
  | The reasons for recommending this format are, responding format can be easily specified and as the responding format is included in URI, it results in an intuitive URI.

 .. note:: **Examples of URI where format is changed using extension**
    
    * \ `http://example.com/api/v1/items.json`\
    * \ `http://example.com/api/v1/items.xml`\
    * \ `http://example.com/api/v1/items/I312-535-01216.json`\
    * \ `http://example.com/api/v1/items/I312-535-01216.xml`\

|

* **Changing format by using the MIME type in Accept header of request.**

  A typical MIME type used in RESTful Web Service is shown below.

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 30 60
    
        * - Sr. No.
          - Format
          - MIME type
        * - | (1)
          - | JSON
          - | application/json
        * - | (2)
          - | XML
          - | application/xml

|


.. _RESTOverviewHttpStatusCode:

Using the appropriate HTTP status code
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
\ **Appropriate HTTP status code is set in the response to be returned to the client.**\

| Value indicating the method by which server has processed the request received from the client, is set in HTTP status code.
| \ **This is an HTTP specification and it is recommended to conform to the HTTP specifications wherever possible.**\

 .. tip:: **HTTP Specifications**
 
    Refer to `RFC 2616 (Hypertext Transfer Protocol -- HTTP/1.1) - 6.1.1 Status Code and Reason Phrase <http://tools.ietf.org/search/rfc2616#section-6.1.1>`_.

|

| In a traditional Web system wherein HTML is returned in the browser, regardless of the process results, it was common that \ ``"200 OK"``\  was returned as the response and process results were displayed in entity body (HTML),
| In a traditional Web application that returns HTML, there were no issues since an operator (human) determined the process results.
| However, if this structure is used to build a RESTful Web Service, following issues may exist potentially. Hence, it is recommended to set appropriate status codes.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Potential issues
    * - | (1)
      - | Even in cases where only the process result (success and failure) is to be determined, unnecessary process has to be performed, as analysis process is mandatory for entity body.
    * - | (2)
      - | Since it is mandatory to be aware of the unique error codes defined in the system while handling errors, it may adversely affect the architecture (design and implementation) at the client side.
    * - | (3)
      - | Intuitive error analysis may be obstructed when analyzing error causes at client side, since understanding the meaning of unique error codes defined in the system is required for the same.

|

.. _RESTOverviewClientServerCommunicateOnStateless:

Stateless communication between client and server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| **In this communication, only the information requested by the client is processed, without retaining the application status on the server.**

| In ROA, a state wherein application status is not retained on the server, is called "stateless".
| It signifies that application status is not retained in application server memory (HTTP session etc.) and resource related operations can be completed only by using the requested information.
| In this guideline, \ **it is recommended to retain "stateless" state wherever possible.**\

 .. note:: **Application status**
 
    Web page transition status, selection status for input value, pull down/checkbox/radio buttons and authentication status etc. are included in application status.

 .. note:: **Relation with CSRF measures**
 
    Please note that the "Stateless" state between client and server cannot be retained when the CSRF measures described in this guideline are implemented for RESTful Web Service as, the token values for CRSF measures are stored in HTTP sessions.

    As a result, system availability must be considered while implementing CSRF measures.
    
    Following measures need to be implemented for a system that requires high availability.
    
    * Perform AP server clustering and session replication.
    * Use a destination other than AP server memory for storing a session.
    
    
    However, above measures may affect the performance. Hence, it is necessary to consider performance requirements as well.
    
    For CSRF measures, refer to \ :doc:`../Security/CSRF`\ .

 .. todo:: **TBD**

    When high availability is required, it is advisable to review an architecture wherein, "token values for CSRF measures are stored in a destination other than the AP server memory (HTTP session)".
    
    Basic architecture is currently under review and will be documented in subsequent versions.
    
|

.. _RESTOverviewHyperMediaLinksToRelatedResources:

Link to related resource
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| \ **Hypermedia link (URI) to another resource related to the specified resource, is included in the resource.**\

| In ROA, the process of incorporating a hypermedia link for another resource in the resource status display, is called  "Connectivity".
| It signifies that both the linked resources retain this mutual link and all the related resources can be accessed by following this link.

Connectivity of resources is described below, with the example of member information resource of a shopping site.

 .. figure:: ./images_REST/RESTConnectivity.png
   :alt: Image of resource connectivity
   :width: 100%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Following JSON is returned when the member information resource is fetched (\ ``GET http://example.com/api/v1/members/M000000001``\ ).

         .. code-block:: json
            :emphasize-lines: 13-14,17-18

            {
                "memberId" : "M000000001",
                "memberName" : "John Smith",
                "address" : {
                    "address1" : "45 West 36th Street",
                    "address2" : "7th Floor",
                    "city" : "New York",
                    "state" : "NY",
                    "zipCode" : "10018"
                },
                "links" : [
                    {
                        "rel" : "orders",
                        "href" : "http://example.com/api/v1/memebers/M000000001/orders"
                    },
                    {
                        "rel" : "authentications",
                        "href" : "http://example.com/api/v1/memebers/M000000001/authentications"
                    }
                ]
            }

        | Highlighted portion is the hypermedia link (URI) to other related resource.
        | In the above example, connectivity is retained between the order history resource and authentication history resource of each member.
    * - | (2)
      - | Following JSON is returned when order history resource is fetched (\ ``GET http://example.com/api/v1/memebers/M000000001/orders``\ ) using the hypermedia link (URI) set in the returned JSON.

         .. code-block:: json
            :emphasize-lines: 10-11,22-23,30-31
        
            {
                "orders" : [
                    {
                        "orderId" : "029b49d7-0efa-411b-bc5a-6570ce40ead8",
                        "orderDatetime" : "2013-12-27T20:34:50.897Z", 
                        "orderName" : "Note PC",
                        "shopName" : "Global PC Shop",
                        "links" : [
                            {
                                "rel" : "order",
                                "href" : "http://example.com/api/v1/memebers/M000000001/orders/029b49d7-0efa-411b-bc5a-6570ce40ead8"
                            }
                        ]
                    },
                    {
                        "orderId" : "79bf991d-d42d-4546-9265-c5d4d59a80eb",
                        "orderDatetime" : "2013-12-03T19:01:44.109Z", 
                        "orderName" : "Orange Juice 100%",
                        "shopName" : "Global Food Shop",
                        "links" : [
                            {
                                "rel" : "order",
                                "href" : "http://example.com/api/v1/memebers/M000000001/orders/79bf991d-d42d-4546-9265-c5d4d59a80eb"
                            }
                        ]
                    }
                ],
                "links" : [
                    {
                        "rel" : "ownerMember",
                        "href" : "http://example.com/api/v1/memebers/M000000001"
                    }
                ]
            }

        | Highlighted portion is the hypermedia link (URI) for another related resource.
        | In the above example, connectivity is retained between the resource for owner member of order history and the order history resource.
    * - | (3)
      - | Following JSON is returned when the resource for owner member of order history is fetched again (\ ``GET http://example.com/api/v1/memebers/M000000001``\ ) and the resource for authentication history is fetched (\ ``GET http://example.com/api/v1/memebers/M000000001/authentications/``\ ) using hypermedia link (URI) set in the returned JSON.
        
         .. code-block:: json
            :emphasize-lines: 18-19
        
            {
                "authentications" : [
                    {
                        "authenticationId" : "6ae9613b-85b6-4dd1-83da-b53c43994433",
                        "authenticationDatetime" : "2013-12-27T20:34:50.897Z", 
                        "clientIpaddress" : "230.210.3.124",
                        "authenticationResult" : true
                    },
                    {
                        "authenticationId" : "103bf3c5-7707-46eb-b2d8-c00ce6243d5f",
                        "authenticationDatetime" : "2013-12-26T10:03:45.001Z", 
                        "clientIpaddress" : "230.210.3.124",
                        "authenticationResult" : false
                    }
                ],
                "links" : [
                    {
                        "rel" : "ownerMember",
                        "href" : "http://example.com/api/v1/memebers/M000000001"
                    }
                ]
            }
        
        | Highlighted portion is the hypermedia link (URI) to other linked resource.
        | In the above example, connectivity is retained with respect to the resource for owner member of authentication history.

|

| It is not mandatory for a resource to include hypermedia link (URI) to another resource.
| When all the endpoints (URI) of REST API are already published, even if the link for related resource is set in the resource, it is highly unlikely that it will be used.
| Particularly, it makes no sense to provide links for the REST API that exchanges resources between systems, since REST API end points that are already published can be accessed directly.
| There is no need to provide a link where it is not required.

| In contrast, when a resource is to be directly exchanged between a client application with user interface and RESTful Web service, the loose coupling between client and server can be enhanced by providing a link.
| Following are the reasons for enhancing the coupling between client and server.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Reasons to enhance the loose coupling
    * - | (1)
      - | Client application needs to know only the logical name of the link in advance. Hence, it is not necessary to know the specific URI for calling REST API.
    * - | (2)
      - | Since it is not necessary for the client application to know the specific URI, impact on server, owing to change in URI, can be minimized.

Whether to provide a hypermedia link (URI) to other resources should be determined on considering all the points described above.

.. tip:: **Relation with HATEOAS**

    HATEOAS is an abbreviation for "\ **H**\ypermedia \ **A**\s \ **T**\he \ **E**\ngine \ **O**\f \ **A**\pplication \ **S**\tate" and is one of the architectures for creating a RESTful Web application.

    HATEOAS architecture includes the following processes.
    
    * In the resources (JSON or XML) that are exchanged between client and server, the server includes a hypermedia link (URI) to an accessible resource.
    * Client fetches required resources from the server through the hypermedia link in the resource display (JSON or XML), and changes application status (screen status etc.).
    
    Therefore, providing a link for related resources is consistent with the HATEOAS architecture.
    
    When loose coupling between server and client is to be enhanced, please review if using the HATEOAS architecture would be beneficial.
    
    

|

.. _RESTHowToDesign:

How to design
--------------------------------------------------------------------------------
This section explains the design of RESTful Web Service.

.. _RESTHowToDesignExtractResource:

Resource extraction
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
First, the resource published on the Web is extracted.

Precautions while extracting a resource are as given below.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Precautions while extracting a resource
    * - | (1)
      - | Resource published on the web is used as the information managed by database. However, \ **data model of the database must not be published as resource as it is, without careful consideration.**\ 
        | It should be closely investigated, as the fields stored in the database may include some fields that should not be disclosed to the client.
    * - | (2)
      - | \ **When information type is different in spite of being managed by the same table of the database, publishing it as a separate resource may be considered.**\
        | There are cases wherein, even if essentially seen as different information, it is managed by the same table, due to same data structure. Hence, such cases need to be reviewed closely.
    * - | (3)
      - | In RESTful Web Service, the information operated by an event is extracted as a resource.
        | \ **The event itself should not be extracted as a resource.**\
        | 
        | For example, when creating RESTful Web Service to be called from the events (approve, deny, return etc.) generated by work flow functionality, information for managing the workflow status or the workflow itself, is extracted as a resource.

|

.. _RESTHowToDesignAssignURI:

Assigning URI
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
URI is assigned to the extracted resource for identifying it.

It is recommended to use following formats for the URI.

* ``http(s)://{Domain name (:Port number)}/{A value indicating REST API}/{API version}/{path for identifying a resource}``\ 

* ``http(s)://{Domain name indicating REST API(:Port number)}/{API version}/{path for identifying a resource}``\ 

A typical example is given below.

* ``http://example.com/api/v1/members/M000000001``\ 

* ``http://api.example.com/v1/members/M000000001``\ 

|

.. _RESTHowToDesignAssignUriForPublishAPI:

Assigning a URI that indicates the API as REST API
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
It is recommended to include  \ ``api``\  within the URI domain or path, to clearly indicate that the URI is intended for RESTful Web Service (REST API).

Typically, the URI is as given below.

* ``http://example.com/api/...``\ 
* ``http://api.example.com/...``\ 

|

.. _RESTHowToDesignAssignUriForApiVersion:

Assigning a URI for identifying the API version
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
It is recommended to include a value that identifies the API version, in the URI to be published to the client, since it may be necessary to run RESTful Web Service in multiple versions.

Typically, the URI format is as follows.

* ``http://example.com/api/{API version}/{path for identifying a resource}``\ 
* ``http://api.example.com/{API version}/{path for identifying a resource}``\ 

.. todo:: **TBD**
 
    Whether API version should be included in URI, is currently being investigated.

|

.. _RESTHowToDesignAssignUriForResource:

Assigning a path for identifying resource
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| The 2 URLs given below are assigned for resources that are published on Web.
| Following is an example of a URI when publishing member information on Web.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.25\linewidth}|p{0.30\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 35 25 30

    * - Sr. No.
      - URI format
      - Typical example of URI
      - Description
    * - | (1)
      - | /{Noun that represents collection of resources}
      - | /api/v1/members
      - | It is the URI used for batch operations of resources.
    * - | (2)
      - | /{Noun that represents collection of resources/resource identifier (ID etc)}
      - | /api/v1/members/M0001
      - | It is the URI used while operating a specific resource.

|

| The URI for related resources published on Web are nested and then displayed.
| Following example describes the URI for publishing order information for each member on the Web.
    
 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.25\linewidth}|p{0.30\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 35 25 30
    
    * - Sr. No.
      - URI format
      - Typical example of URI
      - Description
    * - | (3)
      - | {Resource URI}/{Noun representing collection of related resources}
      - | /api/v1/members/M0001/orders
      - | It is the URI used at the time of batch operation of related resources.
    * - | (4)
      - | {Resource URI}/{Noun representing collection of related resources}/{Identifier for related resource (ID etc.)}
      - | /api/v1/members/M0001/orders/O0001
      - | It is the URI used when operating a specific related resource.

|

| When the related resource published on Web has a single element, the noun that indicates the related resource should be singular and not plural.
| Following is the example of URI to publish credentials of each member on Web.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.25\linewidth}|p{0.30\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 35 25 30

    * - Sr. No.
      - URI format
      - Typical example of URI
      - Description
    * - | (5)
      - | {URI for resource}/{Noun representing related resource}
      - | /api/v1/members/M0001/credential
      - | It is the URI used when operating a related resource with single element.

|

.. _RESTHowToDesignAssignHttpMethod:

Assigning HTTP methods
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
CRUD operation for resources is published as REST API by assigning the following HTTP methods for the URI assigned to each resource.

 .. note:: **HEAD and OPTIONS method**
 
    Hereafter, HEAD and OPTIONS methods are described as well. However, providing them for REST API is optional.
    
    While creating the REST API conforming to HTTP specifications, it is necessary to provide the HEAD and OPTIONS methods as well. However, it is actually used very rarely and is not required in most of the cases.

|

.. _RESTHowToDesignAssignHttpMethodForCollectionResource:

Assigning HTTP methods for resource collection URI
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 70

    * - Sr. No.
      - HTTP methods
      - Overview of the REST API to be implemented
    * - | (1)
      - | GET
      - | REST API that fetches collection of resources specified in URI, is implemented.
    * - | (2)
      - | POST
      - | REST API that creates and adds the specified resource to the collection is implemented.
    * - | (3)
      - | PUT
      - | REST API that performs batch update for resource specified in URI is implemented.
    * - | (4)
      - | DELETE
      - | REST API that performs batch deletion for resource specified in URI is implemented.
    * - | (5)
      - | HEAD
      - | REST API that fetches meta information of the resource collection specified in URI, is implemented.
        | A process same as GET is performed and only header is sent as response.
    * - | (6)
      - | OPTIONS
      - | REST API that responds with the list of HTTP methods (API) supported by resource collection specified in URI, is implemented.

|

.. _RESTHowToDesignAssignHttpMethodForSpecifiedResource:

Assigning HTTP methods for URI of specific resources
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 70

    * - Sr. No.
      - HTTP methods
      - Overview of REST API to be implemented
    * - | (1)
      - | GET
      - | REST API that fetches the resource specified in URI is implemented.
    * - | (2)
      - | PUT
      - | REST API that creates or updates the resource specified in URI is implemented.
    * - | (3)
      - | DELETE
      - | REST API that deletes the resource specified in URI is implemented.
    * - | (4)
      - | HEAD
      - | A REST API that fetches meta information of the resource specified in URI is implemented.
        | A process same as GET is performed and only header is sent as a response.
    * - | (5)
      - | OPTIONS
      - | REST API that responds with list of HTTP methods (API) supported by the resource specified in URI is implemented.

|

.. _RESTHowToDesignResourceRepresentationFormat:

Resource format
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| \ **It is recommended to use JSON**\  as the format for displaying a resource.
| The explanation hereafter is based on the assumption that JSON is used as the format for displaying a resource.


JSON Field name
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| \ **It is recommended to use "lower camel case"**\  as the JSON field name.
| It is recommended on considering its compatibility with JavaScript, which is assumed as one of the client applications.

| The JSON sample with field name set in "lower camel case", is as given below.
| In "lower camel case", first letter of the word is in lowercase and subsequent first letters of words are in uppercase.

 .. code-block:: json

    {
        "memberId" : "M000000001"
    }

|

NULL and blank characters
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| \ **It is recommended to differentiate NULL and blank characters**\  as JSON values.
| Although, as application process, NULL and blank characters are often equated, it is advisable to differentiate NULL and blank characters as the value to be set in JSON.

| JSON sample wherein NULL and blank characters are differentiated, is given below.

 .. code-block:: json

    {
        "dateOfBirth" : null,
        "address1" : ""
    }


|

Date format
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| \ **It is recommended to use extended ISO-8601 format**\  as the JSON date field format.
| Format other than extended ISO-8601 format can be used. However, it is advisable to use the extended ISO-8601 format, if there is no particular reason otherwise.
| There are two formats in ISO-8601 namely, basic format and extended format, however, readability is higher in extended format.

Basically, there are following three formats.

1. yyyy-MM-dd

 .. code-block:: json

    {
        "dateOfBirth" : "1977-03-12"
    }

2. yyyy-MM-dd'T'HH:mm:ss.SSSZ

 .. code-block:: json

    {
        "lastModifiedAt" : "2014-03-12T22:22:36.637+09:00"
    }

3. yyyy-MM-dd'T'HH:mm:ss.SSS'Z' (format for UTC)

 .. code-block:: json

    {
        "lastModifiedAt" : "2014-03-12T13:11:27.356Z"
    }

|

Hypermedia link format
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| It is recommended to use the following format to create a hypermedia link.
| Sample of recommended format is as given below.

 .. code-block:: json

    {
        "links" : [
            {
                "rel" : "ownerMember",
                "href" : "http://example.com/api/v1/memebers/M000000001"
            }
        ]
    }

 * Link object consisting of 2 fields - \ ``"rel"``\  and \ ``"href"``\  is retained in collection format.
 * Link name for identifying the link is specified in \ ``"rel"``\ .
 * URI to access the resource is specified in \ ``"href"``\ .
 * \ ``"links"``\  is the field which retains the Link object in collection format.

|

Format at the time of error response
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| When an error is detected, it is recommended to use a format that can retain the details of the error occurred.
| Detailed error information should be included, especially when there is a possibility of the error being eliminated owing to re-operation by client.
| In contrast, detailed error information should not be included when an event that exposes system vulnerability occurs. In such cases, the detailed error information should be output to a log.

Following is an example of the response format when error is detected.

 .. code-block:: json
    :emphasize-lines: 10, 20, 23

    {
      "code" : "e.ex.fw.7001",
      "message" : "Validation error occurred on item in the request body.",
      "details" : [ {
        "code" : "ExistInCodeList",
        "message" : "\"genderCode\" must exist in code list of CL_GENDER.",
        "target" : "genderCode"
      } ]
    }

In the above example,

* Error code (code)
* Error message (message)
* Error details list (details)

| are provided as error response formats.
| It is assumed that the error details list is used when input validation error occurs. It is a format that can retain details like the field in which the error occurred and the information of the error.

|

.. _RESTHowToDesignHttpStatusCode:

HTTP Status Code
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
HTTP status code is sent as the response, in accordance with the following guidelines.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Objectives
    * - | (1)
      - | When the request is successful, an HTTP status code indicating success or transfer (2xx or 3xx system) is sent as response.
    * - | (2)
      - | When the cause of request failure lies at client side, an HTTP status code indicating client error (4xx system) is sent as the response.
        | When client is not responsible for request failure however, when the request may be successful through a re-operation by client, it is still considered as client error.
    * - | (3)
      - | When the cause of request failure lies at server side, an HTTP status code indicating server error (5xx system) is sent as the response.

|

.. _RESTHowToDesignHttpStatusCodeForSuccess:

HTTP status codes when the request is successful
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
When the request is successful, following HTTP status codes are sent as responses, depending on status.
 
 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.30\linewidth}|p{0.40\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 30 40

    * - | Sr. No.
      - | HTTP
        | Status codes
      - | Description
      - | Applicable conditions
    * - | (1)
      - | 200
        | OK
      - | HTTP status code notifying that the request was successful.
      - | It is sent as a response when the resource information corresponding to the request is output in the entity body of response, as a result of successful request,
    * - | (2)
      - | 201
        | Created
      - | HTTP status code notifying the creation of a new resource.
      - | It is used when a new resource is created using POST method.
        | URI for created resource is set in the Location header of the response.
    * - | (3)
      - | 204
        | No Content
      - | HTTP status code notifying a successful request.
      - | It is sent as a response when the resource information corresponding to request is not output in the entity body of response, as a result of successful request.

 .. tip::
 
    The difference between \ ``"200 OK``\  and \ ``"204 No Content"``\  is whether the resource information is output/not output in the response body.

|

.. _RESTHowToDesignHttpStatusCodeForClientError:

HTTP status code when the cause of request failure lies at client side
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
When the cause of request failure lies at client side, following HTTP status codes are sent as responses depending on the status.

Status codes that must be identified by individual REST APIs handling the resources, are as given below.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.30\linewidth}|p{0.40\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 30 40

    * - | Sr. No.
      - | HTTP
        | Status code
      - | Description
      - | Applicable conditions
    * - | (1)
      - | 400
        | Bad Request
      - | HTTP status code notifying that the request syntax or requested value is incorrect.
      - | It is sent as a response when an incomplete JSON or XML format specified in entity body is detected or an incomplete input value is specified in JSON or XML format or in the request parameters.
    * - | (2)
      - | 404
        | Not Found
      - | HTTP status code notifying that the specified resource does not exist.
      - | It is sent as a response when resource corresponding to specified URI does not exist in the system.
    * - | (3)
      - | 409
        | Conflict
      - | HTTP status code notifying that the process is terminated due to conflict in resource status when the request status is changed by requested contents.
      - | It is sent as a response when an exclusive error or a business error is detected.
        | Conflict details and error details required to resolve the conflict need to be output to the entity body.

|

| Following status codes need not be identified by individual REST APIs which handle the resources.
| These status codes need to be identified as the framework or common process.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.30\linewidth}|p{0.40\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 30 40

    * - | Sr. No.
      - | HTTP
        | Status codes
      - | Description
      - | Applicable conditions
    * - | (4)
      - | 405
        | Method Not Allowed
      - | HTTP status code notifying that the used HTTP method is not supported by the specified resource.
      - | It is sent as a response when an unsupported HTTP method is used.
        | The list of allowed methods is set in the Allow header of response.
    * - | (5)
      - | 406
        | Not Acceptable
      - | HTTP status code notifying the inability to receive a request, as the resource status cannot be sent as a response in the specified format.
      - | It is sent as a response when, the format specified in extension or Accept header is not supported as a response format.
    * - | (6)
      - | 415
        | Unsupported Media Type
      - | HTTP status code notifying that the request cannot be received, as the format specified in entity body is not supported.
      - | It is sent as a response when an unsupported format is specified in Content-Type header, as request format.

|

.. _RESTHowToDesignHttpStatusCodeForServerError:

HTTP status code when the cause of request failure lies at server side
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
When the cause of request failure lies at server side, HTTP status codes given below are sent as responses, depending on the status.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.30\linewidth}|p{0.40\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 30 40

    * - | Sr. No.
      - | HTTP
        | Status code
      - | Description
      - | Applicable conditions
    * - | (1)
      - | 500
        | Internal Server Error
      - | HTTP status code notifying that an internal error has occurred in the server.
      - | It is sent as a response when an unexpected error has occurred in the server or a status that should not occur during a normal operation is detected.

|

Authentication and Authorization
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo:: **TBD**

    The guidelines for authentication and authorization control are explained here.
    
    Performing authentication and authorization using OAuth2 protocol will be described in subsequent versions.

|

Conditional update control of resource
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo:: **TBD**

    The process for conditional update (exclusive control) of a resource using HTTP header is explained here.
    
    Conditional update using headers like Etag/Last-Modified-Since etc. will be described in subsequent versions.

|

Conditional acquisition control of resource
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo:: **TBD**

    The process for conditional acquisition (304 not modified control) of resource using HTTP header is explained here.

    Conditional acquisition using headers like Etag/Last-Modified etc. will be described in subsequent versions.

|

Cache control of resource
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo:: **TBD**

    Cache control of resources which use HTTP header, is explained here.
    
    Cache control of resources that use headers such as Cache-Control/Pragma/Expires etc. shall be described in subsequent versions.

|

Versioning
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo:: **TBD**

    Version control of RESTful Web Service and details on performing parallel operations in multiple versions, will be described in subsequent versions.
    
|

.. _RESTHowToUse:

How to use
--------------------------------------------------------------------------------
This section explains the basic method to create RESTful Web Service.

.. _RESTHowToUseWebApplicationConstruction:

Web application configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| While building RESTful Web Service, Web application (war) is built by any one of the following configurations.
| **It is recommended to build a Web application that is exclusive to RESTful Web Service unless there is a specific reason otherwise.**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 60

    * - Sr. No.
      - Configuration
      - Description
    * - | (1)
      - | Build an exclusive Web application for RESTful Web Service.
      - | It is recommended to build an exclusive Web application (war) for RESTful Web Service when an independence with client application (UI layer application) that uses RESTful Web Service, is to be ensured (is necessary).
        |
        | This method can be used to create RESTful Web Service when there are multiple client applications using RESTful Web Service.
    * - | (2)
      - | Build by providing \ ``DispatcherServlet``\  for RESTful Web Service.
      - | When it is not necessary to ensure independence of client application (UI layer application) that uses RESTful Web Service, both the client application and RESTful Web Service can be built as a single Web application (war).
        |
        | However, it is strongly recommended to build it by dividing \ ``DispatcherServlet``\  that receives the requests for RESTful Web Service and \ ``DispatcherServlet``\  that receives client application requests.

 .. note:: **Client application (UI layer application)**

    Client application (UI layer application) described here refers to the application that responds with client layer (UI layer) component called CSS (Cascading Style Sheets) and scripts like HTML, JavaScript etc.
    HTML generated by template engine such as JSP, is also considered.

 .. note:: **Why division of DispatcherServlet is recommended**

    In Spring MVC, operation settings of the application are defined for each \ ``DispatcherServlet``\ .
    Therefore, when the requests of RESTful Web Service and client application (UI layer application) are configured to be received from the same \ ``DispatcherServlet``\ , specific operation settings for RESTful Web Service or client application cannot be defined, thus resulting in complex or cumbersome settings.
    
    In this guideline, when RESTful Web Service and client application are to be configured as same Web application, it is recommended to divide \ ``DispatcherServlet``\  to avoid occurrence of the issues described above.

|

Configuration image when building a Web application exclusive to RESTful Web Service, is as follows:

 .. figure:: ./images_REST/RESTWebAppsConstitutionDivideWar.png
   :alt: Constitution of RESTful Web Service
   :width: 100%

|

Configuration image when building RESTful Web Service and client application as a single application, is as follows:

 .. figure:: ./images_REST/RESTWebAppsConstitutionDivideServlet.png
   :alt: Constitution of RESTful Web Service
   :width: 100%

|

.. _RESTHowToUseApplicationSettings:

Application settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Application settings for RESTful Web Service are explained below.

.. warning:: **DoS attack measures at the time of StAX(Streaming API for XML) use**

    If the StAX is used to parse the XML format data, protect DoS attack.
    For details, refer to \ `CVE-2015-3192 - DoS Attack with XML Input <http://pivotal.io/security/cve-2015-3192>`_\ .


.. _RESTHowToUseApplicationSettingsOfSpringMVC:

Settings for activating the Spring MVC components necessary for RESTful Web Service
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Create a bean definition file for RESTful Web Service.
| Definitions that are required to operate the sample indicated in the explanation hereafter, are as follows:

- :file:`spring-mvc-rest.xml`

 .. code-block:: xml
    :emphasize-lines: 22, 30-32, 39-41, 44-47, 51, 61, 65

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans" 
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xmlns:mvc="http://www.springframework.org/schema/mvc"
        xmlns:util="http://www.springframework.org/schema/util"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xsi:schemaLocation="
            http://www.springframework.org/schema/mvc
            http://www.springframework.org/schema/mvc/spring-mvc.xsd
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/util
            http://www.springframework.org/schema/util/spring-util.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd
            http://www.springframework.org/schema/aop
            http://www.springframework.org/schema/aop/spring-aop.xsd
    ">

        <!-- Load properties files for placeholder. -->
        <!-- (1) -->
        <context:property-placeholder 
            location="classpath*:/META-INF/spring/*.properties" />
    
        <bean id="jsonMessageConverter"
            class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
            <property name="objectMapper">
                <bean id="objectMapper" class="com.fasterxml.jackson.databind.ObjectMapper">
                    <!-- (2) -->
                    <property name="dateFormat">
                        <bean class="com.fasterxml.jackson.databind.util.StdDateFormat" />
                    </property>
                </bean>
            </property>
        </bean>
    
        <!-- Register components of Spring MVC. -->
        <!-- (3) -->
         <mvc:annotation-driven>
            <mvc:message-converters register-defaults="false">
                <ref bean="jsonMessageConverter" />
            </mvc:message-converters>
            <!-- (4) -->
            <mvc:argument-resolvers>
                <bean class="org.springframework.data.web.PageableHandlerMethodArgumentResolver" />
            </mvc:argument-resolvers>
        </mvc:annotation-driven>
        
        <!-- Register components of interceptor. -->
        <!-- (5) -->
        <mvc:interceptors>
            <mvc:interceptor>
                <mvc:mapping path="/**" />
                <bean class="org.terasoluna.gfw.web.logging.TraceLoggingInterceptor" />
            </mvc:interceptor>
            <!-- omitted -->
        </mvc:interceptors>
    
        <!-- Scan & register components of RESTful Web Service. -->
        <!-- (6) -->
        <context:component-scan base-package="com.example.project.api" />

        <!-- Register components of AOP. -->
        <!-- (7) -->
        <bean id="handlerExceptionResolverLoggingInterceptor" 
            class="org.terasoluna.gfw.web.exception.HandlerExceptionResolverLoggingInterceptor">
            <property name="exceptionLogger" ref="exceptionLogger" />
        </bean>
        <aop:config>
            <aop:advisor advice-ref="handlerExceptionResolverLoggingInterceptor"
                pointcut="execution(* org.springframework.web.servlet.HandlerExceptionResolver.resolveException(..))" />
        </aop:config>

    </beans>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | When the value defined in the property file needs to be referred by an application layer component, the property file should be read by using \ ``<context:property-placeholder>``\  element.
        | For the details of fetching a value from property file, refer to ":doc:`PropertyManagement`".
    * - | (2)
      - | Add the settings for handling the JSON date field format as extended ISO-8601 format.
    * - | (3)
      - | Perform bean registration for the Spring MVC framework component necessary for providing RESTful Web Service.
        | JSON can be used as a resource format by performing these settings.
        | In the above example, resource format is restricted to JSON since the register-defaults attribute of \ ``<mvc:message-converters``>\  element is set as \ ``false``\ .
        |
        | To use XML as resource format, \ ``MessageConverter``\  for XML, that performs the XXE Injection countermeasure, should be specified. For details on designated methods, refer to "\ :ref:`RESTAppendixEnabledXXEInjectProtection`\" .
    * - | (4)
      - | Add the settings to enable page search functionality.
        | For page search details, refer to ":doc:`Pagination`".
        | This setting is not required if page search is unnecessary, however, it is alright if defined.
    * - | (5)
      - | Perform bean registration for Spring MVC interceptor.
        | In the above example, only the \ ``TraceLoggingInterceptor``\  provided by common library is defined. However, when using JPA as data access, \ ``OpenEntityManagerInViewInterceptor``\  setting needs to be added separately.
        | Refer to \ :doc:`DataAccessJpa`\  for \ ``OpenEntityManagerInViewInterceptor``\ .
    * - | (6)
      - | Scan the application layer components for RESTful Web Service (Controller or Helper class etc.) and perform bean registration.
        | The \ ``"com.example.project.api"``\  part is the \ **package name for each project.**\
    * - | (7)
      - | Specify AOP definition to output the exception handled by Spring MVC framework to a log.
        | Refer to \ :doc:`ExceptionHandling`\  for \ ``HandlerExceptionResolverLoggingInterceptor``\ .

.. _REST_note_changed_jackson_version:

.. note::

    **Points to be noted when changing the jackson version from 1.x.x to 2.x.x**

    * Changed package

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90

        * - version
          - package
        * - | 1.x.x
          - | `org.codehaus.jackson`
        * - | 2.x.x
          - | `com.fasterxml.jackson`

     * Please note that configuration of subordinate package also is also changed.

    * Deprecated List

     * http://fasterxml.github.io/jackson-core/javadoc/2.4/deprecated-list.html
     * http://fasterxml.github.io/jackson-databind/javadoc/2.4/deprecated-list.html
     * http://fasterxml.github.io/jackson-annotations/javadoc/2.4/deprecated-list.html

|

.. _RESTHowToUseApplicationSettingsOfDispatcherServlet:

Servlet settings for RESTful Web Service
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| When RESTful Web Service and client application are built as separate Web applications, the settings are as follows:
| When RESTful Web Service and client application are to be built as same Web application, it is necessary to perform "\ :ref:`RESTAppendixSettingsOfDeployInSameWarFileRestAndClientApplication`\" .

- :file:`web.xml`

 .. code-block:: xml
    :emphasize-lines: 4-5,9-10,14-18

    <!-- omitted -->

    <servlet>
        <!-- (1) -->
        <servlet-name>restAppServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <!-- (2) -->
            <param-value>classpath*:META-INF/spring/spring-mvc-rest.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <!-- (3) -->
    <servlet-mapping>
        <servlet-name>restAppServlet</servlet-name>
        <url-pattern>/api/v1/*</url-pattern>
    </servlet-mapping>

    <!-- omitted -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify a name which shows that it is a  RESTful Web Service servlet, in \ ``<servlet-name>``\  element.
        | In the above example, \ ``"restAppServlet"``\  is specified as the servlet name.
    * - | (2)
      - | Specify a Spring MVC bean definition file used to build \ ``DispatcherServlet``\  for RESTful Web Service.
        | In the above example, \ :file:`META-INF/spring/spring-mvc-rest.xml`\  in class path, is specified as the Spring MVC bean definition file.
    * - | (3)
      - | Specify a servlet path pattern to be mapped with the \ ``DispatcherServlet``\  of RESTful Web Service.
        | In the above example, the servlet path under \ ``"/api/v1/"``\  is mapped with the  \ ``DispatcherServlet``\  for RESTful Web Service.
        | Typically, servlet paths like
        |   \ ``"/api/v1/"``\ 
        |   \ ``"/api/v1/members"``\ 
        |   \ ``"/api/v1/members/xxxxx"``\ 
        | are mapped in the \ ``DispatcherServlet``\ (\ ``"restAppServlet"``\ ) for RESTful Web Service.

 .. tip:: **Value specified in the value attribute of @RequestMapping annotation**

   For the value to be specified in value attribute of \ ``@RequestMapping``\  annotation, specify the value assigned to the part of wild card (\ ``*``\) in \ ``<url-pattern>``\  element.
   
   For example, when \ ``@RequestMapping(value = "members")``\  is specified, it is deployed as the method to perform a process for path \ ``"/api/v1/members"``\ .
   Therefore, it is not necessary to specify the path  (\ ``"api/v1"``\) in value attribute of \ ``@RequestMapping``\  annotation for mapping to divided servlets.
   
   When \ ``@RequestMapping(value = "api/v1/members")``\  is specified, it gets deployed as the method that performs a process for the \ ``"/api/v1/api/v1/members"``\  path. Hence, please take note of same.

|

.. _RESTHowToUseApiImplementation:

REST API implementation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| How to implement REST API is explained here.
| Hereafter, REST API implementation example is explained using member information (Member resource) of a shopping site.

 .. note::

    Domain layer implementation is not explained in this section, however, it is sent as attachment  "\ :ref:`RESTAppendixSoruceCodesOfDomainLayer`\".

    Please refer if required.

|

REST API specifications used in this explanation are as shown below.

|

**Resource format**

 | The resource format of member information should be the following JSON format.
 | In the following example, although all the fields are displayed, they are not used in the requests and responses of all API.
 | For example, \ ``"password"``\  is used only in requests whereas \ ``"createdAt"``\  or \ ``"lastModifiedAt"``\  are used only in responses.

 .. code-block:: json

    {
        "memberId" : "M000000001",
        "firstName" : "Firstname",
        "lastName" : "Lastname",
        "genderCode" : "1",
        "dateOfBirth" : "1977-03-13",
        "emailAddress" : "user1@test.com",
        "telephoneNumber" : "09012345678",
        "zipCode" : "1710051",
        "address" : "Tokyo",
        "credential" : {
            "signId" : "user1@test.com",
            "password" : "zaq12wsx",
            "passwordLastChangedAt" : "2014-03-13T04:39:14.831Z",
            "lastModifiedAt" : "2014-03-13T04:39:14.831Z"
        },
        "createdAt" : "2014-03-13T04:39:14.831Z",
        "lastModifiedAt" : "2014-03-13T04:39:14.831Z"
    }

 .. note::
 
     This section illustrates an example wherein a hypermedia link for related resource is not provided.
     For details on implementation with hypermedia link, refer to ":ref:`RESTAppendixHyperMediaLink`". 

|

**Specifications of resource fields**
 
 The specifications for each field of a resource (JSON) are as shown below.
 
 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.10\linewidth}|p{0.10\linewidth}|p{0.15\linewidth}|p{0.25\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 10 10 15 25

    * - Sr. No.
      - Item name
      - Type
      - I/O specifications
      - Number of digits
        (min-max)
      - Other specifications
    * - | (1)
      - memberId
      - String
      - I/O
      - 10-10
      - It should be "Unspecified" (NULL) at the time of request for POST Members.
    * - | (2)
      - firstName
      - String
      - I/O
      - 1-128
      - \-
    * - | (3)
      - lastName
      - String
      - I/O
      - 1-128
      - \-
    * - | (4)
      - genderCode
      - | String
        | (Code)
      - I/O
      - 1-1
      - | ``"0"`` : UNKNOWN
        | ``"1"`` : MEN
        | ``"2"`` : WOMEN
    * - | (5)
      - dateOfBirth
      - Date
      - I/O
      - \-
      - | yyyy-MM-dd format
        | (extended ISO-8601 format)
    * - | (6)
      - emailAddress
      - | String
        | (E-mail)
      - I/O
      - 1-256
      - \-
    * - | (7)
      - telephoneNumber
      - String
      - I/O
      - 0-20
      - \-
    * - | (8)
      - zipCode
      - String
      - I/O
      - 0-20
      - \-
    * - | (9)
      - address
      - String
      - I/O
      - 0-256
      - \-
    * - | (10)
      - credential
      - | Object
        | (MemberCredential)
      - I/O
      - \-
      - It is specified at the time of request for POST Members.
    * - | (11)
      - credential/signId
      - | String
        | (E-mail)
      - I/O
      - 0-256
      - emailAddress value is applied when not specified.
    * - | (12)
      - | credential/
        | password
      - String
      - I
      - 8-32
      - \-
    * - | (13)
      - | credential/
        | passwordLastChangedAt
      - | Timestamp
        | with time-zone
      - O
      - \-
      - | yyyy-MM-dd'T'HH:mm:ss.SSS'Z' format
        | (extended ISO-8601 format)
    * - | (14)
      - | credential/
        | lastModifiedAt
      - | Timestamp
        | with time-zone
      - O
      - \-
      - | yyyy-MM-dd'T'HH:mm:ss.SSS'Z'format
        | (extended ISO-8601 format)
    * - | (15)
      - createdAt
      - | Timestamp
        | with time-zone
      - O
      - \-
      - | yyyy-MM-dd'T'HH:mm:ss.SSS'Z' format
        | (extended ISO-8601 format)
    * - | (16)
      - lastModifiedAt
      - | Timestamp
        | with time-zone
      - O
      - \-
      - | yyyy-MM-dd'T'HH:mm:ss.SSS'Z' format
        | (extended ISO-8601 format)

|

**REST APIs List**

 APIs given below are used as the REST API to be implemented.
 
 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.10\linewidth}|p{0.25\linewidth}|p{0.15\linewidth}|p{0.25\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 15 10 25 15 25

    * - | Sr. No.
      - | API name
      - | HTTP
        | Method
      - | Resource path
      - | Status
        | Code
      - | API Overview
    * - | (1)
      - :ref:`GET Members <RESTHowToUseApiImplementationOfGetCollection>`
      - GET
      - \ ``/api/v1/members``\ 
      - | 200
        | (OK)
      - | Page is searched for Member resource that matches the condition.
    * - | (2)
      - :ref:`POST Members <RESTHowToUseApiImplementationOfPostCollection>`
      - POST
      - \ ``/api/v1/members``\ 
      - | 201
        | (Created)
      - One Member resource is created.
    * - | (3)
      - :ref:`GET Member <RESTHowToUseApiImplementationOfGetSpecifiedResource>`
      - GET
      - \ ``/api/v1/members/{memberId}``\ 
      - | 200
        | (OK)
      - One Member resource is fetched.
    * - | (4)
      - :ref:`PUT Member <RESTHowToUseApiImplementationOfPutSpecifiedResource>`
      - PUT
      - \ ``/api/v1/members/{memberId}``\ 
      - | 200
        | (OK)
      - One Member resource is updated.
    * - | (5)
      - :ref:`DELETE Member <RESTHowToUseApiImplementationOfDeleteSpecifiedResource>`
      - DELETE
      - \ ``/api/v1/members/{memberId}``\ 
      - | 204
        | (No Content)
      - One Member resource is deleted.

 .. note::
 
     This section focuses on the details of CRUD operation for a resource. Hence, HEAD and OPTIONS methods are not explained.
     To create the RESTful Web Service conforming to HTTP specifications, refer to ":ref:`RESTAppendixRestApiOfHTTPCompliance`".

|

.. _RESTHowToUsePackage:

Creating REST API packages
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Create a package to store REST API class.

| It is recommended to assign \ ``api``\  as the package name for the route package that stores REST API class and to create a package for each resource (lower case of resource name) under the same.
| Resource name in the explanation is \ ``Member``\ . Hence, the package name is \ ``org.terasoluna.examples.rest.api.member``\ .

 .. note::

    Usually, following 4 types of classes are stored in the created package.
    It is recommended to use the following naming rules for name of the class to be created.

     * \ ``[Resource name]Resource``\ 
     * \ ``[Resource name]RestController``\ 
     * \ ``[Resource name]Validator``\  (created when required)
     * \ ``[Resource name]Helper``\  (created when required)

     In the explanation, name of the resource is \ ``Member``\ . As a result, the respective names will be as below.

     * \ ``MemberResource``\ 
     * \ ``MemberRestController``\ 
     * \ ``MemberValidator``\ 
     * \ ``MemberHelper``\ 

    

    When handling a related resource, it is advisable to place the class for related resource also in the same package.

|

| It is recommended to create a package named ``common``\  that stores common parts for REST API just under the route package that stores the REST API class and to create sub packages at functionality level.
| For example, a sub package that stores common parts which perform error handling is created with the name \ ``error``\ .
| The class for exception handling created in the subsequent explanation, is stored in a package called \ ``org.terasoluna.examples.rest.api.common.error``\ .

 .. note::

    As long as it is clear that the package is storing common parts, it can have a name other than \ ``common``\ .

| 


.. _RESTHowToUseResourceClass:

Creating Resource class
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| In this guideline, it is recommended to provide a Resource class as the class representing the resource published on Web (represent JSON or XML).

 .. note:: **Reasons for creating a Resource class**

  The reason for creating a Resource class regardless of DomainObject class (for example, Entity class) being available is,
  user interface information (UI) which is used in the I/O with client and information handled by business process do not necessarily match.
  
  If these are mixed and then used, the application layer may affect the domain layer, resulting in deteriorated maintainability.
  It is recommended to create the DomainObject and Resource class separately and convert data by using BeanMapper like Dozer etc.

|

Role of Resource class is as follows:

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 60

    * - Sr. No.
      - Roles
      - Description
    * - | (1)
      - | To define the data structure of a resource.
      - | Define a data structure of the resource published on Web.
        | Generally, it is very rare to publish the data structure managed by persistence layer of database etc. as it is, as a resource on Web.
    * - | (2)
      - | To define format.
      - | Specify a definition related to resource format using annotation.
        | Annotation to be used differs according to the resource format (JSON/XML etc.). Jackson annotation is used for JSON format whereas JAXB annotation is used for XML format.
    * - | (3)
      - | To define input validation rules.
      - | Specify input validation rules for single item of each field by using Bean Validation annotation.
        | For input validation details, refer to "\ :doc:`Validation`\" .


 .. warning:: **Measures to circular reference**

     When you serialize a Resource class (JavaBean) in JSON or XML format and if property holds an object of cross reference relationship,
     the \ ``StackOverflowError``\  and \ ``OutOfMemoryError``\  occur due to circular reference, hence it is necessary to exercise caution.

     In order to avoid a circular reference,

     * \ ``@com.fasterxml.jackson.annotation.JsonIgnore`` \  annotation to exclude the property from serialization in case of serialized in JSON format using the Jackson
     * \ ``@javax.xml.bind.annotation.XmlTransient`` \  annotation to exclude the property from serialization in case of serialized in XML format using the JAXB

     can be added.

     An example to exclude specific field from serialization while serializing in JSON format using Jackson is given below.

      .. code-block:: java

          public class Order {
              private String orderId;
              private List<OrderLine> orderLines;
              // ...
          }

      .. code-block:: java

          public class OrderLine {
              @JsonIgnore
              private Order order;
              private String itemCode;
              private int quantity;
              // ...
          }

      .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
      .. list-table::
          :header-rows: 1
          :widths: 10 90

          * - Sr. No.
            - Description
          * - | (1)
            - Add \ ``@JsonIgnore``\  annotation to exclude the property from serialization.

|

Example of Resource class creation is shown below.

* :file:`MemberResource.java`

 .. code-block:: java
    :emphasize-lines: 17, 22-27, 67

    package org.terasoluna.examples.rest.api.member;
    
    import java.io.Serializable;
    
    import javax.validation.Valid;
    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Null;
    import javax.validation.constraints.Past;
    import javax.validation.constraints.Size;
    
    import org.hibernate.validator.constraints.Email;
    import org.hibernate.validator.constraints.NotEmpty;
    import org.joda.time.DateTime;
    import org.joda.time.LocalDate;
    import org.terasoluna.gfw.common.codelist.ExistInCodeList;
    
    // (1)
    public class MemberResource implements Serializable {
    
        private static final long serialVersionUID = 1L;
    
        // (2)
        interface PostMembers {
        }
    
        interface PutMember {
        }
    
        @Null(groups = PostMembers.class)
        @NotEmpty(groups = PutMember.class)
        @Size(min = 10, max = 10, groups = PutMember.class)
        private String memberId;
    
        @NotEmpty
        @Size(max = 128)
        private String firstName;
    
        @NotEmpty
        @Size(max = 128)
        private String lastName;
    
        @NotEmpty
        @ExistInCodeList(codeListId = "CL_GENDER")
        private String genderCode;
    
        @NotNull
        @Past
        private LocalDate dateOfBirth;
    
        @NotEmpty
        @Size(max = 256)
        @Email
        private String emailAddress;
    
        @Size(max = 20)
        private String telephoneNumber;
    
        @Size(max = 20)
        private String zipCode;
    
        @Size(max = 256)
        private String address;
    
        @NotNull(groups = PostMembers.class)
        @Null(groups = PutMember.class)
        @Valid
        // (3)
        private MemberCredentialResource credential;
    
        @Null
        private DateTime createdAt;
    
        @Null
        private DateTime lastModifiedAt;
    
        // omitted setter and getter
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | JavaBean representing the Member resource.
    * - | (2)
      - | The interface for specifying validation group of Bean Validation is defined.
        | In this implementation, input validation is grouped, as different input validations are performed for POST and PUT methods.
        | Refer to ":doc:`Validation`" for grouped validation.
    * - | (3)
      - | JavaBean with nested related resource is defined in the field.
        | In this implementation, the member credentials (sign ID and password) are handled as related resources.
        | These are extracted as related resources by considering operations that carry out only the sign ID change and password change.
        | However, REST API implementation for related resources is omitted here.

* :file:`MemberCredentialResource.java`

 .. code-block:: java
    :emphasize-lines: 13, 22

    package org.terasoluna.examples.rest.api.member;
    
    import java.io.Serializable;
    
    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Null;
    import javax.validation.constraints.Size;
    
    import com.fasterxml.jackson.annotation.JsonInclude;

    import org.hibernate.validator.constraints.Email;
    import org.joda.time.DateTime;

    // (4)
    public class MemberCredentialResource implements Serializable {
    
        private static final long serialVersionUID = 1L;
    
        @Size(max = 256)
        @Email
        private String signId;

        // (5)
        @JsonInclude(JsonInclude.Include.NON_NULL)
        @NotNull
        @Size(min = 8, max = 32)
        private String password;
    
        @Null
        private DateTime passwordLastChangedAt;
    
        @Null
        private DateTime lastModifiedAt;
    
        // omitted setter and getter
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (4)
      - | JavaBean that represents Credential resource which is the related resource of Member resource.
    * - | (5)
      - | An annotation is specified so that the field itself is not output in JSON when the value is \ ``null``\ .
        | It is specified so that the field 'password' is not output in responding JSON.
        | In the above example, it is restricted to (\ ``Inclusion.NON_NULL``\ ) for NULL value, however it can also be specified as (\ ``Inclusion.NON_EMPTY``\ ) in case of an empty value.

|

* | Adding the mapping definition of Bean
  | In the subsequent implementations, Entity class and Resource class are copied by using "\ :doc:`Utilities/Dozer`\" .
  | Joda-Time classes namely, ``org.joda.time.DateTime``\  and \ ``org.joda.time.LocalDate``\  are included in the JavaBean shown above. However, Joda-Time objects are not correctly copied if "\ :doc:`Utilities/Dozer`\ " is used for copying.
  | Therefore, it is necessary to apply ":ref:`RESTAppendixCopyJodaObjectByBeanConvert`" to copy the objects correctly.

|

.. _RESTHowToUseControllerClass:

Creating Controller class
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Create controller class for each resource.
| Refer to \ :ref:`Appendix <RESTAppendixSoruceCodesOfMemberRestController>`\  for the source code when implementation of all APIs is completed.

 .. code-block:: java
    :emphasize-lines: 7-8

    package org.terasoluna.examples.rest.api.member;
    
    // omitted
    import org.springframework.web.bind.annotation.RestController;
    // omitted

    @RequestMapping("members") // (1)
    @RestController // (2)
    public class MemberRestController {

        // omitted ...

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Map resource collection URI (Servlet path) for Controller.
        | Typically, specify a servlet path indicating a collection of resources in the value attribute of \ ``@RequestMapping``\  annotation.
        | In the above example, a servlet path called \ ``/api/v1/members``\  is mapped.
    * - | (2)
      - Assign \ ``@RestController``\  annotation for Controller.

        Assigning \ ``@RestController``\  annotation has same meaning of:

        * Assigning \ ``org.springframework.stereotype.Controller``\  annotation in a class
        * Assigning \ ``@org.springframework.web.bind.annotation.ResponseBody``\  annotation in Controller method which is described later.


        By assigning \ ``@ResponseBody``\  to Controller method, the returned Resource object is marshalled in JSON or XML and set in response body.

 .. tip::

    \ ``@RestController``\  is an annotation added from Spring Framework 4.0.

    Due to \ ``@RestController``\  annotation, it is not necessary to assign \ ``@ResponseBody``\  annotation to each method of Controller.
    Hence, it is possible to create Controller for REST API in a simple way.
    For details about \ ``@RestController``\  annotation refer to: \ `Here <http://docs.spring.io/spring/docs/4.1.7.RELEASE/javadoc-api/org/springframework/web/bind/annotation/RestController.html>`_\ .

    An example to create a Controller for REST API by combining \ ``@Controller``\  annotation and \ ``@ResponseBody``\  annotation in a conventional way is given below.

     .. code-block:: java

        @RequestMapping("members")
        @Controller
        public class MemberRestController {

            @RequestMapping(method = RequestMethod.GET)
            @ResponseStatus(HttpStatus.OK)
            @ResponseBody
            public Page<MemberResource> getMembers() {
                // ...
            }

            // ...

        }

|

.. _RESTHowToUseApiImplementationOfGetCollection:

Implementing REST API that fetches collection of resources
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Example to implement the REST API wherein a page search is performed for member resource collection specified by URI.

- | Creating the JavaBean for receiving search conditions
  | When search conditions are necessary to fetch resource collection, create a JavaBean for receiving the search conditions.

 .. code-block:: java
    :emphasize-lines: 1, 5

    // (1)
    public class MembersSearchQuery implements Serializable {
        private static final long serialVersionUID = 1L;
    
        // (2)
        @NotEmpty
        private String name;
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Create a JavaBean for receiving search conditions.
        | When search conditions are not necessary, JavaBean need not be created.
    * - | (2)
      - | Match the property name with the parameter name of request parameter.
        | In the above example, value \ ``"John"``\  is set in the name property of JavaBean for request \ ``/api/v1/members?name=John``\ .

|

* | REST API implementation
  | Implement a process wherein page search is performed for a collection of Member resources.
  
 .. code-block:: java
    :emphasize-lines: 13, 15, 18, 20, 23, 26, 34

    @RequestMapping("members")
    @RestController
    public class MemberRestController {
    
        // omitted

        @Inject
        MemberService memberService;
    
        @Inject
        Mapper beanMapper;
    
        // (3)
        @RequestMapping(method = RequestMethod.GET)
        // (4)

        @ResponseStatus(HttpStatus.OK)
        public Page<MemberResource> getMembers(
                // (5)
                @Validated MembersSearchQuery query,
                // (6)
                Pageable pageable) {
    
            // (7)
            Page<Member> page = memberService.searchMembers(query.getName(), pageable);
    
            // (8)
            List<MemberResource> memberResources = new ArrayList<>();
            for (Member member : page.getContent()) {
                memberResources.add(beanMapper.map(member, MemberResource.class));
            }
            Page<MemberResource> responseResource = new PageImpl<>(memberResources, 
                    pageable, page.getTotalElements());
    
            // (9)
            return responseResource;
        }

        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (3)
      - | Specify \ ``RequestMethod.GET``\  in method attribute of \ ``@RequestMapping``\  annotation.
    * - | (4)
      - | Assign \ ``@org.springframework.web.bind.annotation.ResponseStatus``\  as method annotation and specify the status code returned as response.
        | Set 200 (OK) in the value attribute of \ ``@ResponseStatus``\  annotation.
        
        .. tip:: **How to specify the status code**
        
            A fixed status code sent as response is specified in this example using \ ``@ResponseStatus``\  annotation. However, it can also be specified in Controller logic.

             .. code-block:: java
            
                public ResponseEntity<Page<MemberResource>> getMembers(
                        @Validated MembersSearchQuery query,
                        Pageable pageable) {
                
                    // omitted
                    
                    return ResponseEntity.ok().body(responseResource);
                }

            When it is necessary to change the responding status codes based on process details or process results, \ ``org.springframework.http.ResponseEntity``\  is used, as shown in the above implementation.


    * - | (5)
      - | Specify a JavaBean for receiving search conditions as an argument.
        | When input validation is necessary, assign \ ``@Validated``\  as argument annotation. For input validation details, refer to "\ :doc:`Validation`\ ".
    * - | (6)
      - | When page search is necessary, specify \ ``org.springframework.data.domain.Pageable``\  as an argument.
        | For page search details, refer to ":doc:`Pagination`".
    * - | (7)
      - | Call Service method of domain layer and fetch resource information (Entity etc.) matching with the condition.
        | For domain layer implementation, refer to ":doc:`../ImplementationAtEachLayer/DomainLayer`".
    * - | (8)
      - | Generate resource object that retains information published on the Web based on the resource information matching with the conditions (Entity etc.).
        | By using \ ``org.springframework.data.domain.PageImpl``\  class while sending page search result as response, the fields that are necessary as response at the time of page search, can be sent to the client.
        |
        | In the above example, a Resource object is being generated from Entity by using Bean mapping library. For details on Bean mapping library, refer to "\ :doc:`Utilities/Dozer`\" .
        | **When the quantity of code for generating Resource objects is more, it is recommended to create a method for generating Resource object in Helper class.**
    * - | (9)
      - | Return a Resource object generated in (8).
        | The object returned here is marshalled in JSON or XML and set in response body.

 | Response at the time of using \ ``PageImpl``\  class is as below.
 | Highlighted portion shows the fields specific for page search.
 
 .. code-block:: json
    :emphasize-lines: 37-50
    
    {
      "content" : [ {
        "memberId" : "M000000001",
        "firstName" : "John",
        "lastName" : "Smith",
        "genderCode" : "1",
        "dateOfBirth" : "1977-03-07",
        "emailAddress" : "john.smith@test.com",
        "telephoneNumber" : "09012345678",
        "zipCode" : "1710051",
        "address" : "Tokyo",
        "credential" : {
          "signId" : "john.smit@test.com",
          "passwordLastChangedAt" : "2014-03-13T10:18:08.003Z",
          "lastModifiedAt" : "2014-03-13T10:18:08.003Z"
        },
        "createdAt" : "2014-03-13T10:18:08.003Z",
        "lastModifiedAt" : "2014-03-13T10:18:08.003Z"
      }, {
        "memberId" : "M000000002",
        "firstName" : "Sophia",
        "lastName" : "Smith",
        "genderCode" : "2",
        "dateOfBirth" : "1977-03-07",
        "emailAddress" : "sophia.smith@test.com",
        "telephoneNumber" : "09012345678",
        "zipCode" : "1710051",
        "address" : "Tokyo",
        "credential" : {
          "signId" : "sophia.smith@test.com",
          "passwordLastChangedAt" : "2014-03-13T10:18:08.003Z",
          "lastModifiedAt" : "2014-03-13T10:18:08.003Z"
        },
        "createdAt" : "2014-03-13T10:18:08.003Z",
        "lastModifiedAt" : "2014-03-13T10:18:08.003Z"
      } ],
      "last" : false,
      "totalPages" : 13,
      "totalElements" : 25,
      "size" : 2,
      "number" : 1,
      "sort" : [ {
        "direction" : "DESC",
        "property" : "lastModifiedAt",
        "ignoreCase" : false,
        "nullHandling": "NATIVE",
        "ascending" : false
      } ],
      "numberOfElements" : 2,
      "first" : false
    }

 .. note:: **Points to be noted due to changes in API specifications of Spring Data Commons**

    In case of "terasoluna-gfw-common 5.0.0.RELEASE or later version" dependent spring-data-commons (1.9.1 RELEASE or later),
    there is a change in API specifications of interface for page search functionality (\ ``org.springframework.data.domain.Page``\ ) and class (\ ``org.springframework.data.domain.PageImpl``\  and \ ``org.springframework.data.domain.Sort.Order``\ ).

    Specifically,

    * In \ ``Page``\  interface and \ ``PageImpl``\  class, \ ``isFirst()``\  and \ ``isLast()``\  methods are added in spring-data-commons 1.8.0.RELEASE, and \ ``isFirstPage()``\  and \ ``isLastPage()``\  methods are deleted from spring-data-commons 1.9.0.RELEASE.
    * In \ ``Sort.Order``\  class, \ ``nullHandling``\  property is added in spring-data-commons 1.8.0.RELEASE.

    When using \ ``Page``\  interface (\ ``PageImpl``\  class) as resource object of REST API,
    that application may also need to be modified, as JSON and XML format get changed.

|

* | Adding Bean mapping definition
  | In the above implementation, \ ``Member``\  object and \ ``MemberResource``\  object are copied by using "\ :doc:`Utilities/Dozer`\" .
  | It is not necessary to add Bean mapping definition when simply a copy of field value can be used. However, in the above implementation, the setting needs to be such that \ ``credential.password``\  is not copied while copying \ ``Member``\  object details to \ ``MemberResource``\  object.
  | It is necessary to add Bean mapping definition so that specific fields are not copied.

 .. code-block:: xml
    :emphasize-lines: 1, 10-14
  
    <!-- (11) -->
    <?xml version="1.0" encoding="UTF-8"?>
    <mappings xmlns="http://dozer.sourceforge.net" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://dozer.sourceforge.net
              http://dozer.sourceforge.net/schema/beanmapping.xsd">
    
        <mapping type="one-way">
            <class-a>org.terasoluna.examples.rest.domain.model.MemberCredential</class-a>
            <class-b>org.terasoluna.examples.rest.api.member.MemberCredentialResource</class-b>
            <!-- (12) -->
            <field-exclude>
                <a>password</a>
                <b>password</b>
            </field-exclude>
        </mapping>
    
    </mappings>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (11)
      - | Create a file that defines mapping rules for \ ``Member``\  object and \ ``MemberResource``\  object.
        | It is recommended to create a mapping definition file of Dozer for each resource.
        | 
        | In this implementation, it is stored in \ :file:`/xxx-web/src/main/resources/META-INF/dozer/memberResource-mapping.xml`\ .
    * - | (12)
      - | In the above example, \ ``password``\  field is not copied while copying the details of ``MemberCredential``\  which is a related entity of \ ``Member``\ , to ``MemberCredentialResource``\ , a related resource of \ ``MemberResource``\ .
        | For Bean mapping definition methods, refer to "\ :doc:`Utilities/Dozer`\" .

|

* Request example

 .. code-block:: guess
    :emphasize-lines: 1

    GET /rest-api-web/api/v1/members?name=Smith&page=0&size=2 HTTP/1.1
    Accept: text/plain, application/json, application/*+json, */*
    User-Agent: Java/1.7.0_51
    Host: localhost:8080
    Connection: keep-alive

|

* Response Example

 .. code-block:: guess
    :emphasize-lines: 1

    HTTP/1.1 200 OK
    Server: Apache-Coyote/1.1
    X-Track: fb63a6d446f849feb8ccaa4c9a794333
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Thu, 13 Mar 2014 11:10:43 GMT
    
    {"content":[{"memberId":"M000000001","firstName":"John","lastName":"Smith","genderCode":"1","dateOfBirth":"2013-03-13","emailAddress":"user1394709042120@test.com","telephoneNumber":"09012345678","zipCode":"1710051","address":"Tokyo","credential":{"signId":"user1394709042120@test.com","passwordLastChangedAt":"2014-03-13T11:10:43.066Z","lastModifiedAt":"2014-03-13T11:10:43.066Z"},"createdAt":"2014-03-13T11:10:43.066Z","lastModifiedAt":"2014-03-13T11:10:43.066Z"},{"memberId":"M000000002","firstName":"Sophia","lastName":"Smith","genderCode":"2","dateOfBirth":"2013-03-13","emailAddress":"user1394709043663@test.com","telephoneNumber":"09012345678","zipCode":"1710051","address":"Tokyo","credential":{"signId":"user1394709043663@test.com","passwordLastChangedAt":"2014-03-13T11:10:43.678Z","lastModifiedAt":"2014-03-13T11:10:43.678Z"},"createdAt":"2014-03-13T11:10:43.678Z","lastModifiedAt":"2014-03-13T11:10:43.678Z"}],"last":true,"totalPages":1,"totalElements":2,"size":2,"number":0,"sort":null,"numberOfElements":2,"first":true}

|

 .. tip::
 
    When page search is not necessary, Resource class list may be handled directly.

    Following is the definition of the Controller method used when handling the list of Resource class directly.

     .. code-block:: java
        :emphasize-lines: 4

        @RequestMapping(method = RequestMethod.GET)

        @ResponseStatus(HttpStatus.OK)
        public List<MemberResource> getMembers(
                @Validated MembersSearchQuery query) {
            // omitted
        }

    JSON is as follows when list of Resource class is directly handled.
    
     .. code-block:: json

        [ {
            "memberId" : "M000000001",
            "firstName" : "John",
            "lastName" : "Smith",
            "genderCode" : "1",
            "dateOfBirth" : "1977-03-07",
            "emailAddress" : "john.smith@test.com",
            "telephoneNumber" : "09012345678",
            "zipCode" : "1710051",
            "address" : "Tokyo",
            "credential" : {
              "signId" : "john.smit@test.com",
              "passwordLastChangedAt" : "2014-03-13T10:18:08.003Z",
              "lastModifiedAt" : "2014-03-13T10:18:08.003Z"
            },
            "createdAt" : "2014-03-13T10:18:08.003Z",
            "lastModifiedAt" : "2014-03-13T10:18:08.003Z"
        }, {
            "memberId" : "M000000002",
            "firstName" : "Sophia",
            "lastName" : "Smith",
            "genderCode" : "2",
            "dateOfBirth" : "1977-03-07",
            "emailAddress" : "sophia.smith@test.com",
            "telephoneNumber" : "09012345678",
            "zipCode" : "1710051",
            "address" : "Tokyo",
            "credential" : {
              "signId" : "sophia.smith@test.com",
              "passwordLastChangedAt" : "2014-03-13T10:18:08.003Z",
              "lastModifiedAt" : "2014-03-13T10:18:08.003Z"
            },
            "createdAt" : "2014-03-13T10:18:08.003Z",
            "lastModifiedAt" : "2014-03-13T10:18:08.003Z"
        } ]


|

.. _RESTHowToUseApiImplementationOfPostCollection:

Implementing REST API that adds a resource to collection
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Example of implementation of REST API wherein a specified Member resource is created and added to the collection is given below.

* | REST API implementation
  | Implement a process that creates specified Member resource and adds it to the collection.

 .. code-block:: java
    :emphasize-lines: 7, 9, 12, 16

    @RequestMapping("members")
    @RestController
    public class MemberRestController {
    
        // omitted

        // (1)
        @RequestMapping(method = RequestMethod.POST)
        // (2)
        @ResponseStatus(HttpStatus.CREATED)
        public MemberResource postMember(
                // (3)
                @RequestBody @Validated({ PostMembers.class, Default.class }) 
                MemberResource requestedResource) {

            // (4)
            Member inputMember = beanMapper.map(requestedResource, Member.class);
            Member createdMember = memberService.createMember(inputMember);

            MemberResource responseResource = beanMapper.map(createdMember,
                    MemberResource.class);

            return responseResource;
        }

        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify \ ``RequestMethod.POST``\  in the method attribute of \ ``@RequestMapping``\  annotation.
    * - | (2)
      - | Assign \ ``@ResponseStatus``\  annotation as method annotation and specify responding status code.
        | Set \ **201(Created)**\  in the value attribute of \ ``@ResponseStatus``\  annotation.
    * - | (3)
      - | Specify JavaBean (Resource class) that receives information of newly created resource as an argument.
        | Assign ``@org.springframework.web.bind.annotation.RequestBody``\  as argument annotation.
        | By assigning \ ``@RequestBody``\  annotation, JSON or XML data set in request Body is unmarshalled in Resource object.
        |
        | Assign \ ``@Validated``\  annotation as argument annotation to enable input validation. For details on input validation, refer to "\ :doc:`Validation`\" .
    * - | (3)
      - | Call Service method of domain layer and create a new resource.
        | For domain layer implementation, refer to ":doc:`../ImplementationAtEachLayer/DomainLayer`".

|

* Request example

 .. code-block:: guess
    :emphasize-lines: 1

    POST /rest-api-web/api/v1/members HTTP/1.1
    Accept: text/plain, application/json, application/*+json, */*
    Content-Type: application/json;charset=UTF-8
    User-Agent: Java/1.7.0_51
    Host: localhost:8080
    Connection: keep-alive
    Content-Length: 248
    
    {"firstName":"John","lastName":"Smith","genderCode":"1","dateOfBirth":"2013-03-13","emailAddress":"user1394708306056@test.com","telephoneNumber":"09012345678","zipCode":"1710051","address":"Tokyo","credential":{"signId":null,"password":"zaq12wsx"}}

|

* Response example

 .. code-block:: guess
    :emphasize-lines: 1

    HTTP/1.1 201 Created
    Server: Apache-Coyote/1.1
    X-Track: c7e9c8a9aa4f40ff87f3acdb77baccdf
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Thu, 13 Mar 2014 10:58:26 GMT
    
    {"memberId":"M000000023","firstName":"John","lastName":"Smith","genderCode":"1","dateOfBirth":"2013-03-13","emailAddress":"user1394708306056@test.com","telephoneNumber":"09012345678","zipCode":"1710051","address":"Tokyo","credential":{"signId":"user1394708306056@test.com","passwordLastChangedAt":"2014-03-13T10:58:26.324Z","lastModifiedAt":"2014-03-13T10:58:26.324Z"},"createdAt":"2014-03-13T10:58:26.324Z","lastModifiedAt":"2014-03-13T10:58:26.324Z"}

|

.. _RESTHowToUseApiImplementationOfGetSpecifiedResource:

Implementing REST API that fetches specified resource
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Implementation of REST API that fetches the Member resource specified by URI, is shown below.

* | REST API implementation
  | Implement a process that fetches the Member resource specified by URI.

 .. code-block:: java
    :emphasize-lines: 7, 9, 12, 15


    @RequestMapping("members")
    @RestController
    public class MemberRestController {

        // omitted

        // (1)
        @RequestMapping(value = "{memberId}", method = RequestMethod.GET)

        // (2)
        @ResponseStatus(HttpStatus.OK)
        public MemberResource getMember(
                // (3)
                @PathVariable("memberId") String memberId) {
    
            // (4)
            Member member = memberService.getMember(memberId);
    
            MemberResource responseResource = beanMapper.map(member,
                    MemberResource.class);
    
            return responseResource;
        }

        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify path variable (\ ``{memberId}``\  in the example above) in value attribute whereas \ ``RequestMethod.GET``\  in method attribute of \ ``@RequestMapping``\  annotation.
        | A value that uniquely identifies the resource is specified in \ ``{memberId}``\ .
    * - | (2)
      - | Assign \ ``@ResponseStatus``\  annotation as method annotation and specify the responding status code.
        | Set 200 (OK) in value attribute of \ ``@ResponseStatus``\  annotation.
    * - | (3)
      - | Fetch the value that uniquely identifies the resource from path variable.
        | Value specified in path variable (\ ``{memberId}``\) can be received as method argument by specifying \ ``@PathVariable("memberId")``\  as argument annotation.
        | For details on path variable, refer to ":ref:`controller_method_argument-pathvariable-label`".
        | In the above example, when URI is \ ``/api/v1/members/M12345``\ , \ ``"M12345"``\  is stored in \ ``memberId``\  of argument.
    * - | (4)
      - | Call Service method of domain layer and acquire the resource information (Entity etc.) that matches with the ID fetched from path variable.
        | For domain layer implementation, refer to ":doc:`../ImplementationAtEachLayer/DomainLayer`".

|

* Request example

 .. code-block:: guess
    :emphasize-lines: 1

    GET /rest-api-web/api/v1/members/M000000003 HTTP/1.1
    Accept: text/plain, application/json, application/*+json, */*
    User-Agent: Java/1.7.0_51
    Host: localhost:8080
    Connection: keep-alive

|

* Response Example

 .. code-block:: guess
    :emphasize-lines: 1

    HTTP/1.1 200 OK
    Server: Apache-Coyote/1.1
    X-Track: 275b4e7a61f946eea47672f272315ac2
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Thu, 13 Mar 2014 11:25:13 GMT
    
    {"memberId":"M000000003","firstName":"John","lastName":"Smith","genderCode":"1","dateOfBirth":"2013-03-13","emailAddress":"user1394709913496@test.com","telephoneNumber":"09012345678","zipCode":"1710051","address":"Tokyo","credential":{"signId":"user1394709913496@test.com","passwordLastChangedAt":"2014-03-13T11:25:13.762Z","lastModifiedAt":"2014-03-13T11:25:13.762Z"},"createdAt":"2014-03-13T11:25:13.762Z","lastModifiedAt":"2014-03-13T11:25:13.762Z"}

|

.. _RESTHowToUseApiImplementationOfPutSpecifiedResource:

Implementing REST API that updates specified resource
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Implementation of REST API that updates the Member resource specified in URI, is shown below.

* | REST API implementation
  | Implement a process that updates the Member resource specified in URI.

 .. code-block:: java
    :emphasize-lines: 7, 9, 13, 17

    @RequestMapping("members")
    @RestController
    public class MemberRestController {

        // omitted

        // (1)
        @RequestMapping(value = "{memberId}", method = RequestMethod.PUT)

        // (2)
        @ResponseStatus(HttpStatus.OK)
        public MemberResource putMember(
                @PathVariable("memberId") String memberId,
                // (3)
                @RequestBody @Validated({ PutMember.class, Default.class })
                MemberResource requestedResource) {
    
            // (4)
            Member inputMember = beanMapper.map(
                requestedResource, Member.class);
            Member updatedMember = memberService.updateMember(
                memberId, inputMember);
    
            MemberResource responseResource = beanMapper.map(updatedMember,
                    MemberResource.class);
    
            return responseResource;
        }

        // omitted
        
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify path variable (\ ``{memberId}``\ in the example above) in value attribute whereas \ ``RequestMethod.PUT``\  in "method" attribute of \ ``@RequestMapping``\  annotation.
        | Value that uniquely identifies the resource is specified in \ ``{memberId}``\ .
    * - | (2)
      - | Assign \ ``@ResponseStatus``\ annotation as method annotation and specify the responding status code.
        | Set 200 (OK) in value attribute of \ ``@ResponseStatus``\  annotation.
    * - | (3)
      - | Specify JavaBean (Resource class) for receiving the details of resource update as an argument.
        | By assigning \ ``@RequestBody``\  annotation as argument annotation, JSON or XML data set in request Body is unmarshalled in Resource object.
        |
        | Assign \ ``@Validated``\  annotation as argument annotation to enable input validation.
        | For details on input validation, refer to "\ :doc:`Validation`\" .
    * - | (4)
      - | Call Service method of domain layer and update the resource information (Entity etc.) matching with the ID fetched from path variable.
        | For domain layer implementation, refer to ":doc:`../ImplementationAtEachLayer/DomainLayer`".

|

* Request example

 .. code-block:: guess
    :emphasize-lines: 1

    PUT /rest-api-web/api/v1/members/M000000004 HTTP/1.1
    Accept: text/plain, application/json, application/*+json, */*
    Content-Type: application/json;charset=UTF-8
    User-Agent: Java/1.7.0_51
    Host: localhost:8080
    Connection: keep-alive
    Content-Length: 221
    
    {"memberId":"M000000004","firstName":"John","lastName":"Smith","genderCode":"1","dateOfBirth":"2013-03-08","emailAddress":"user1394710559584@test.com","telephoneNumber":"09012345678","zipCode":"1710051","address":"Tokyo"}

|

* Response example

 .. code-block:: guess
    :emphasize-lines: 1

    HTTP/1.1 200 OK
    Server: Apache-Coyote/1.1
    X-Track: 5e8fea3aae044e94bf20a293e155af57
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Thu, 13 Mar 2014 11:35:59 GMT
    
    {"memberId":"M000000004","firstName":"John","lastName":"Smith","genderCode":"1","dateOfBirth":"2013-03-08","emailAddress":"user1394710559584@test.com","telephoneNumber":"09012345678","zipCode":"1710051","address":"Tokyo","credential":{"signId":"user1394710559584@test.com","passwordLastChangedAt":"2014-03-13T11:35:59.847Z","lastModifiedAt":"2014-03-13T11:35:59.847Z"},"createdAt":"2014-03-13T11:35:59.847Z","lastModifiedAt":"2014-03-13T11:36:00.122Z"}

|


|

.. _RESTHowToUseApiImplementationOfDeleteSpecifiedResource:

Implementing REST API that deletes specified resource
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Implementation of REST API that deletes the Member resource specified by URI is as follows:

* | REST API implementation
  | Implement a process that deletes the Member resource specified by URI.

 .. code-block:: java
    :emphasize-lines: 7, 9, 14

    @RequestMapping("members")
    @RestController
    public class MemberRestController {

        // omitted

        // (1)
        @RequestMapping(value = "{memberId}", method = RequestMethod.DELETE)

        // (2)
        @ResponseStatus(HttpStatus.NO_CONTENT)
        public void deleteMember(
                @PathVariable("memberId") String memberId) {
    
            // (3)
            memberService.deleteMember(memberId);
            
        }

        // omitted
        
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify path variable (\ ``{memberId}``\ in the example above) in value attribute and \ ``RequestMethod.DELETE``\  in method attribute of \ ``@RequestMapping``\  annotation.
    * - | (2)
      - | Assign \ ``@ResponseStatus``\  annotation as method annotation and specify the responding status code.
        | Set \ **204 (NO_CONTENT)**\  in value attribute of \ ``@ResponseStatus``\  annotation.
    * - | (3)
      - | Call Service method of domain layer and delete resource information (Entity etc.) matching with the ID fetched from path variable.
        | For domain layer implementation, refer to ":doc:`../ImplementationAtEachLayer/DomainLayer`".

 .. note::
 
    To set deleted resource information in response BODY, set (200) OK in the status code.

|

* Request example

 .. code-block:: guess
    :emphasize-lines: 1

    DELETE /rest-api-web/api/v1/members/M000000005 HTTP/1.1
    Accept: text/plain, application/json, application/*+json, */*
    User-Agent: Java/1.7.0_51
    Host: localhost:8080
    Connection: keep-alive

|

* Response example

 .. code-block:: guess
    :emphasize-lines: 1

    HTTP/1.1 204 No Content
    Server: Apache-Coyote/1.1
    X-Track: e06c5bd40c864a299c48d9be3f12b2c0
    Date: Thu, 13 Mar 2014 11:40:05 GMT

|


.. _RESTHowToUseExceptionHandling:

Implementing exception handling
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
How to handle the exceptions occurring in RESTful Web Service is explained below.

| RESTful Web Service oriented generic exception handling feature is not provided in Spring MVC.
| Alternately, (\ ``org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler``\ ) is provided as the class that assists in implementing exception handling for RESTful Web Service.
| This guideline recommends a common exception handling method, by creating an exception handling class that inherits the Spring MVC class and assigning \ ``@ControllerAdvice``\  annotation to this exception handling class.

| Method that handles exceptions in Spring MVC framework is implemented in advance in \ ``ResponseEntityExceptionHandler``\  by using \ ``@ExceptionHandler``\  annotation.
| Therefore, individual handling of the exceptions in Spring MVC framework need not be implemented.
| Further, HTTP status codes corresponding to the exceptions handled by \ ``ResponseEntityExceptionHandler``\  are set by the same specifications as \ ``DefaultHandlerExceptionResolver``\ .
| For handled exceptions and HTTP status codes thus set, refer to "\ :ref:`exception-handling-appendix-defaulthandlerexceptionresolver-label`\ ".

| Although an empty response BODY is returned by default implementation of \ ``ResponseEntityExceptionHandler``\ , it can be extended so as to output error information in the response Body.
| This guideline recommends to output appropriate error information in the response Body.

| Process flow wherein, exception handling class that inherits \ ``ResponseEntityExceptionHandler``\  is created  and common exception handling is performed, is described before explaining the typical implementation.
| Please note that, separate implementation is required for the part marked in red frame.

 .. figure:: ./images_REST/RESTHowToUseExceptionHandling.png
   :alt: Image of exception handling by Spring MVC
   :width: 100%


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10  20 70

    * - Sr. No.
      - Processing layer
      - Description
    * - | (1)
        | (2)
      - | Spring MVC
        | (Framework)
      - | Spring MVC receives a request from client and calls REST API.
    * - | (3)
      - | 
        | 
      - | An exception occurs during REST API process.
        | The exception occurred is caught by Spring MVC.
    * - | (4)
      - | 
        | 
      - | Spring MVC delegates the process to exception handling class.
    * - | (5)
      - | Custom Exception Handler
        | (Common Component)
      - | An error object that retains error information is generated in the exception handling class and returned to Spring MVC.
    * - | (6)
      - | Spring MVC
        | (Framework)
      - | Spring MVC converts the error object to JSON format message using \ ``HttpMessageConverter``\ .
    * - | (7)
      - | 
        | 
      - | Spring MVC sets the JSON format error message in response BODY and sends response to the client.


|

.. _RESTHowToUseExceptionHandlingForErrorContentInResponseBody:

Implementation to output error information in response Body
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
* Error information should be in the following JSON format.

 .. code-block:: json
    :emphasize-lines: 10, 20, 23

    {
      "code" : "e.ex.fw.7001",
      "message" : "Validation error occurred on item in the request body.",
      "details" : [ {
        "code" : "ExistInCodeList",
        "message" : "\"genderCode\" must exist in code list of CL_GENDER.",
        "target" : "genderCode"
      } ]
    }

|

* Create JavaBean that retains error information.

 .. code-block:: java
    :emphasize-lines: 9, 19, 22
    
    package org.terasoluna.examples.rest.api.common.error;
    
    import java.io.Serializable;
    import java.util.ArrayList;
    import java.util.List;
    
    import com.fasterxml.jackson.annotation.JsonInclude;

    // (1)
    public class ApiError implements Serializable {
    
        private static final long serialVersionUID = 1L;
    
        private final String code;
    
        private final String message;
    
        @JsonInclude(JsonInclude.Include.NON_EMPTY)
        private final String target; // (2)
    
        @JsonInclude(JsonInclude.Include.NON_EMPTY)
        private final List<ApiError> details = new ArrayList<>(); // (3)
    
        public ApiError(String code, String message) {
            this(code, message, null);
        }
    
        public ApiError(String code, String message, String target) {
            this.code = code;
            this.message = message;
            this.target = target;
        }
    
        public String getCode() {
            return code;
        }
    
        public String getMessage() {
            return message;
        }
    
        public String getTarget() {
            return target;
        }
    
        public List<ApiError> getDetails() {
            return details;
        }
    
        public void addDetail(ApiError detail) {
            details.add(detail);
        }
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Create a class for retaining error information.
        | In the above example, it is the class that retains lists of error codes, error messages, error targets and detailed error information.
    * - | (2)
      - | Field that retains the value for identifying the target where error has occurred.
        | When an error occurs in input validation, there are cases where the value that identifies the field where error has occurred needs to be returned to the client.
        | In such cases, it is necessary to set a field that retains the field name where error has occurred.
    * - | (3)
      - | Field that retains the list of detailed error information.
        | When an error occurs in input validation, it is necessary to return all the error information to the client as there are cases with multiple error causes.
        | In such cases, a field that lists and retains detailed error information is necessary.

 .. tip::   
 
    When the value is \ ``null``\  or empty, it is possible to avoid fields being output to JSON  by specifying \ ``@JsonInclude(JsonInclude.Include.NON_EMPTY)``\  in the field.
    When the condition to disable field output is to be restricted to ``null``\ , it is advisable to specify \ ``@JsonInclude(JsonInclude.Include.NON_NULL)``\ .

|

* Create a class for generating JavaBean that retains error information.

 Refer to \ :ref:`Appendix <RESTAppendixSoruceCodesOfApiErrorCreator>`\  for the source code when implementation of all exception handling is completed.

 .. code-block:: java
    :emphasize-lines: 1, 10

    // (4)
    @Component
    public class ApiErrorCreator {
    
        @Inject
        MessageSource messageSource;

        public ApiError createApiError(WebRequest request, String errorCode,
                String defaultErrorMessage, Object... arguments) {
            // (5)
            String localizedMessage = messageSource.getMessage(errorCode,
                    arguments, defaultErrorMessage, request.getLocale()); 
            return new ApiError(errorCode, localizedMessage);
        }
    
        // omitted
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (4)
      - | If needed, create a class that provides the method for generating error information.
        | Creating this class is not mandatory. However, it is recommended to create it so as to clearly define the role division.
    * - | (5)
      - | Fetch the error message from \ ``MessageSource``\ .
        | For message management methods, refer to "\ :doc:`MessageManagement`\".

 .. tip::

    In the above example, \ ``org.springframework.web.context.request.WebRequest``\  is received as an argument to support localization of messages.
    \ ``WebRequest``\  is not necessary when message localization is not required.
    
    The reason for using \ ``WebRequest``\  as an argument instead of \ ``java.util.Locale``\  is due to an additional requirement wherein, HTTP request details are to be embedded in the error message.
    When there is no such requirement to embed HTTP request details in error, \ ``Locale``\  can also be used.

|

* \ ``ResponseEntityExceptionHandler``\  method is extended and the implementation to output error information in response Body is carried out.

 Refer to \ :ref:`Appendix <RESTAppendixSoruceCodesOfApiGlobalExceptionHandler>`\  for the source code when implementation for all exception handling is completed.

 .. code-block:: java
    :emphasize-lines: 1-2, 10-12, 16, 24

    @ControllerAdvice // (6)
    public class ApiGlobalExceptionHandler extends ResponseEntityExceptionHandler {
    
        @Inject
        ApiErrorCreator apiErrorCreator;
    
        @Inject
        ExceptionCodeResolver exceptionCodeResolver;
    
        // (7)
        @Override
        protected ResponseEntity<Object> handleExceptionInternal(Exception ex,
                Object body, HttpHeaders headers, HttpStatus status,
                WebRequest request) {
            final Object apiError;
            // (8)
            if (body == null) {
                String errorCode = exceptionCodeResolver.resolveExceptionCode(ex);
                apiError = apiErrorCreator.createApiError(request, errorCode, ex
                        .getLocalizedMessage());
            } else {
                apiError = body;
            }
            // (9)
            return ResponseEntity.status(status).headers(headers).body(apiError);
        }
        
        // omitted

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (6)
      - | Create a class that inherits \ ``ResponseEntityExceptionHandler``\  provided by Spring MVC and assign \ ``@ControllerAdvice``\  annotation.
    * - | (7)
      - | Override handleExceptionInternal method of \ ``ResponseEntityExceptionHandler``\ .
    * - | (8)
      - | When the JavaBean output to response Body is not specified, generate a JavaBean object that retains error information.
        | In the above example, the exception class changes the error code by using \ ``ExceptionCodeResolver``\  provided by common library.
        | For setting example of \ ``ExceptionCodeResolver``\ ,  refer to "\ :ref:`RESTHowToUseExceptionHandlingSettingsOfExceptionCodeResolver`\" .
        |
        | When the JavaBean output to response Body is specified, use the specified JavaBean as it is.
        | This process is implemented considering that error information is generated individually in the error handling process for each exception.
    * - | (9)
      - | Set the error information generated in (8) in the 'Body' of HTTP Entity for response and then return the same.
        | Error information thus returned is converted to JSON using framework and sent as a response.
        |
        | Appropriate values are set in the status code by ``ResponseEntityExceptionHandler``\  provided by Spring MVC.
        | Refer to "\ :ref:`exception-handling-appendix-defaulthandlerexceptionresolver-label`\"  for status codes that are set.

 .. tip:: **Attribute of @ControllerAdvice annotation added in Spring Framework 4.0**

    By specifying an attribute of \ ``@ControllerAdvice``\  annotation,
    it has been improved to allow flexibility in specifying Controller to apply a method implemented in the class wherein \ ``@ControllerAdvice``\  is assigned.
    For details about attribute refer to: \ :ref:`Attribute of @ControllerAdvice <application_layer_controller_advice_attribute>`\ .

 .. note:: **Points to be noted while using an attribute of @ControllerAdvice annotation**

    By using an attribute of \ ``@ControllerAdvice``\  annotation, it is possible to share exception handling in respective granularity,
    however, it is advisable not to specify attribute of \ ``@ControllerAdvice``\  annotation for exception handling of common application (class corresponding to \ ``ApiGlobalExceptionHandler``\  class in the above example). 

   When an attribute is specified in \ ``@ControllerAdvice``\  annotation assigned in \ ``ApiGlobalExceptionHandler``\  , a part of exception handling may not be possible that occurs in framework process provided by Spring MVC.

    Specifically, in ``ApiGlobalExceptionHandler``\  class, exception handling is not possible for the exceptions that occur when REST API (process method of Controller) corresponding to the request could not be found,
    hence, it is not possible to correctly respond to errors such as "405 Method Not Allowed", etc.


|

* Response example

 .. code-block:: guess
    :emphasize-lines: 1, 9

    HTTP/1.1 400 Bad Request
    Server: Apache-Coyote/1.1
    X-Track: e60b3b6468194e22852c8bfc7618e625
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Thu, 13 Mar 2014 12:16:55 GMT
    Connection: close
    
    {"code":"e.ex.fw.7001","message":"Validation error occurred on item in the request body.","details":[{"code":"ExistInCodeList","message":"\"genderCode\" must exist in code list of CL_GENDER.","target":"genderCode"}]}

|

.. _RESTHowToUseExceptionHandlingForValidationError:

Implementing input error exception handling
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Implementation for responding to input errors (syntax error, unit item check error, correlated field check error) is explained here.

Following three exceptions need to be handled in order to respond to input errors.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 35 55

    * - Sr. No.
      - Exception
      - Description
    * - | (1)
      - | org.springframework.web.bind.
        | MethodArgumentNotValidException
      - | This exception occurs when there is an  error during the input validation for JSON or XML specified in request BODY.
        | Basically, it occurs when an invalid value is entered in the resource that is specified when POST or PUT method is performed for the resource.
    * - | (2)
      - | org.springframework.validation.
        | BindException
      - | This exception occurs when there is an error during the input validation for request parameter (key=query string of value format).
        | Basically, it occurs when an invalid value is entered in the search conditions specified at the time of GET method of resource collection.
    * - | (3)
      - | org.springframework.http.converter.
        | HttpMessageNotReadableException
      - | This exception occurs when there is an error while generating Resource object from JSON or XML.
        | Basically, it occurs in cases such as invalid JSON or XML syntax or violation of schema definition.

 .. note::
 
    \ ``org.springframework.beans.TypeMismatchException``\  occurs when there is a type conversion error for value while fetching values from request parameter, request header and path variable, by using annotation provided by Spring Framework.
    
    When following annotations are specified as arguments of Controller processing method (argument other than \ ``String``\ ), \ ``TypeMismatchException``\  may occur.
    
     * \ ``@org.springframework.web.bind.annotation.RequestParam``\
     * \ ``@org.springframework.web.bind.annotation.RequestHeader``\
     * \ ``@org.springframework.web.bind.annotation.Pathvariable``\
     * \ ``@org.springframework.web.bind.annotation.MatrixVariable``\
     
    \ ``TypeMismatchException``\  is handled by \ ``ResponseEntityExceptionHandler``\  resulting in 400 (Bad Request). As a result, individual handling is not required.
    
    Refer to "\ :ref:`RESTHowToUseExceptionHandlingSettingsOfExceptionCodeResolver`\" in order to resolve the error codes and error messages to be set in error information.

|

* Method is created to generate error information for input validation errors.

 .. code-block:: java
    :emphasize-lines: 9-10, 26-27

    @Component
    public class ApiErrorCreator {
    
        @Inject
        MessageSource messageSource;

        // omitted

        // (1)
        public ApiError createBindingResultApiError(WebRequest request,
                String errorCode, BindingResult bindingResult,
                String defaultErrorMessage) {
            ApiError apiError = createApiError(request, errorCode,
                    defaultErrorMessage);
            for (FieldError fieldError : bindingResult.getFieldErrors()) {
                apiError.addDetail(createApiError(request, fieldError, fieldError
                        .getField()));
            }
            for (ObjectError objectError : bindingResult.getGlobalErrors()) {
                apiError.addDetail(createApiError(request, objectError, objectError
                        .getObjectName()));
            }
            return apiError;
        }
    
        // (2)
        private ApiError createApiError(WebRequest request,
                DefaultMessageSourceResolvable messageResolvable, String target) {
            String localizedMessage = messageSource.getMessage(messageResolvable,
                    request.getLocale());
            return new ApiError(messageResolvable.getCode(), localizedMessage, target);
        }

        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | A method is created to generate error information for input validation.
        | In the above example, single field check error (\ ``FieldError``\ ) and correlated field check error (\ ``ObjectError``\ ) are added to detailed error information.
        | This method need not be provided when it is not necessary to output error information for each item.
    * - | (2)
      - | A common method is created since same process is implemented for single field check error (\ ``FieldError``\ ) and correlation check error (\ ``ObjectError``\ ).

|

* \ ``ResponseEntityExceptionHandler``\  method is extended and the implementation to output input validation error information in response Body, is performed.

 .. code-block:: java
    :emphasize-lines: 12-14, 21-23, 29-31, 34-36, 44-45

    @ControllerAdvice
    public class ApiGlobalExceptionHandler extends ResponseEntityExceptionHandler {
    
        @Inject
        ApiErrorCreator apiErrorCreator;
    
        @Inject
        ExceptionCodeResolver exceptionCodeResolver;
    
        // omitted

        // (3)
        @Override
        protected ResponseEntity<Object> handleMethodArgumentNotValid(
                MethodArgumentNotValidException ex, HttpHeaders headers,
                HttpStatus status, WebRequest request) {
            return handleBindingResult(ex, ex.getBindingResult(), headers, status,
                    request);
        }
    
        // (4)
        @Override
        protected ResponseEntity<Object> handleBindException(BindException ex,
                HttpHeaders headers, HttpStatus status, WebRequest request) {
            return handleBindingResult(ex, ex.getBindingResult(), headers, status,
                    request);
        }
    
        // (5)
        @Override
        protected ResponseEntity<Object> handleHttpMessageNotReadable(
                HttpMessageNotReadableException ex, HttpHeaders headers,
                HttpStatus status, WebRequest request) {
            if (ex.getCause() instanceof Exception) {
                return handleExceptionInternal((Exception) ex.getCause(), null,
                        headers, status, request);
            } else {
                return handleExceptionInternal(ex, null, headers, status, request);
            }
        }

        // omitted

        // (6)
        protected ResponseEntity<Object> handleBindingResult(Exception ex,
                BindingResult bindingResult, HttpHeaders headers,
                HttpStatus status, WebRequest request) {
            String code = exceptionCodeResolver.resolveExceptionCode(ex);
            String errorCode = exceptionCodeResolver.resolveExceptionCode(ex);
            ApiError apiError = apiErrorCreator.createBindingResultApiError(
                    request, errorCode, bindingResult, ex.getMessage());
            return handleExceptionInternal(ex, apiError, headers, status, request);
        }

        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (3)
      - | Override handleMethodArgumentNotValid method of \ ``ResponseEntityExceptionHandler``\  and extend error handling for \ ``MethodArgumentNotValidException``\ .
        | In the above example, the process is delegated to a common method (6) that handles input validation errors.
        | When it is not necessary to output error information for each item, overriding is not required.
        | 
        | \ **400 (Bad Request)**\  is set in the status code and presence of some flaw in the field value of the specified resource is notified.
    * - | (4)
      - | Override handleBindException method of \ ``ResponseEntityExceptionHandler``\  and extend error handling for \ ``BindException``\ .
        | In the above example, the process is delegated to a common method (6) that handles input validation error.
        | When it is not necessary to output error information for each item, overriding is not required.
        | 
        | \ **400 (Bad Request)**\  is set in the status code and presence of flaw in the specified request parameter is notified.
    * - | (5)
      - | Override handleHttpMessageNotReadable method of \ ``ResponseEntityExceptionHandler``\  and extend error handling for \ ``HttpMessageNotReadableException``\ .
        | In the above example, detailed error handling is performed by using cause exception.
        | If a detailed error handling is not necessary, overriding is not required.
        | 
        | **400 (Bad Request)**\  is set in the status code and presence of flaw in the specified resource format etc. is notified
    * - | (6)
      - | Generate a JavaBean object that retains error information for input validation error.
        | In the above example, this method is created as a common method since same process is implemented in handleMethodArgumentNotValid and handleBindException.

 .. tip:: **Error handling when using JSON**

    When JSON is used as the resource format, following exception is stored as cause exception of \ ``HttpMessageNotReadableException``\ .

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 35 55
    
        * - Sr. No.
          - Exception class
          - Description
        * - | (1)
          - | com.fasterxml.jackson.core.
            | JsonParseException
          - | It occurs when an invalid syntax is included for JSON.
        * - | (2)
          - | com.fasterxml.jackson.databind.exc.
            | UnrecognizedPropertyException
          - | It occurs when a field which does not exist in Resource object is specified in JSON.
        * - | (3)
          - | com.fasterxml.jackson.databind.
            | JsonMappingException
          - | It occurs when a value type conversion error occurs while converting from JSON to Resource object.

|

* Following error response is sent when an input validation error (single field check error, correlated field check error) occurs.

 .. code-block:: guess
    :emphasize-lines: 1, 9

    HTTP/1.1 400 Bad Request
    Server: Apache-Coyote/1.1
    X-Track: 13522b3badf2432ba4cad0dc7aeaee80
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Wed, 19 Feb 2014 05:08:28 GMT
    Connection: close
    
    {"code":"e.ex.fw.7002","message":"Validation error occurred on item in the request parameters.","details":[{"code":"NotEmpty","message":"\"{0}\" may not be empty.","target":"name"}]}

|

* Following error response is sent when JSON errors (format error etc.) occur.

 .. code-block:: guess
    :emphasize-lines: 1, 9

    HTTP/1.1 400 Bad Request
    Server: Apache-Coyote/1.1
    X-Track: ca4c742a6bfd49e5bc01cd0b124738a1
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Wed, 19 Feb 2014 13:32:24 GMT
    Connection: close
    
    {"code":"e.ex.fw.7003","message":"Request body format error occurred."}

|

.. _RESTHowToUseExceptionHandlingForNotFound:

Implementing exception handling for "Resource not found" error
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
When a resource does not exist, implementation for responding to the "resource not found" error, is explained below.

| When a resource matching with the ID fetched from path variable is not found, generate an exception notifying "resource not found" .
| \ ``org.terasoluna.gfw.common.exception.ResourceNotFoundException``\  is provided by common library as an exception notifying "resource not found".
| Implementation is as given below.

|

* When a resource matching with the ID fetched from path variable is not found, generate \ ``ResourceNotFoundException``\ .

 .. code-block:: java
    :emphasize-lines: 4-5

    public Member getMember(String memberId) {
        Member member = memberRepository.findOne(memberId);
        if (member == null) {
            throw new ResourceNotFoundException(ResultMessages.error().add(
                    "e.ex.mm.5001", memberId));
        }
        return member;
    }

|

* Create a method to generate error information for \ ``ResultMessages``\ .

 .. code-block:: java
    :emphasize-lines: 6-7

    @Component
    public class ApiErrorCreator {

        // omitted

        // (1)
        public ApiError createResultMessagesApiError(WebRequest request,
                String rootErrorCode, ResultMessages resultMessages,
                String defaultErrorMessage) {
            ApiError apiError;
            if (resultMessages.getList().size() == 1) {
                ResultMessage resultMessage = resultMessages.iterator().next();
                String errorCode = resultMessage.getCode();
                String errorText = resultMessage.getText();
                if (errorCode == null && errorText == null) {
                    errorCode = rootErrorCode;
                }
                apiError = createApiError(request, errorCode, errorText,
                        resultMessage.getArgs());
            } else {
                apiError = createApiError(request, rootErrorCode,
                        defaultErrorMessage);
                for (ResultMessage resultMessage : resultMessages.getList()) {
                    apiError.addDetail(createApiError(request, resultMessage
                            .getCode(), resultMessage.getText(), resultMessage
                            .getArgs()));
                }
            }
            return apiError;
        }
    
        // omitted
        
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Create a method for generating error information from process results.
        | In the above example, the message information retained by \ ``ResultMessages``\  is set in error information.
        | 
        
 .. note::
 
    In the above example, as \ ``ResultMessages``\  can retain multiple messages, the process is divided as per when a single message is stored and when multiple messages are stored.

    When it is not necessary to support multiple messages, the process wherein the message at the start is generated as error information, may be implemented.


|

* Create a method for handling the exception that notifies "resource not found" error, in the class that performs error handling.

 .. code-block:: java
    :emphasize-lines: 12-13, 17, 22-23

    @ControllerAdvice
    public class ApiGlobalExceptionHandler extends ResponseEntityExceptionHandler {
    
        @Inject
        ApiErrorCreator apiErrorCreator;
    
        @Inject
        ExceptionCodeResolver exceptionCodeResolver;

        // omitted

        // (2)
        @ExceptionHandler(ResourceNotFoundException.class)
        public ResponseEntity<Object> handleResourceNotFoundException(
                ResourceNotFoundException ex, WebRequest request) {
            return handleResultMessagesNotificationException(ex, new HttpHeaders(),
                    HttpStatus.NOT_FOUND, request);
        }

        // omitted

        // (3)
        private ResponseEntity<Object> handleResultMessagesNotificationException(
                ResultMessagesNotificationException ex, HttpHeaders headers,
                HttpStatus status, WebRequest request) {
            String errorCode = exceptionCodeResolver.resolveExceptionCode(ex);
            ApiError apiError = apiErrorCreator.createResultMessagesApiError(
                    request, errorCode, ex.getResultMessages(), ex.getMessage());
            return handleExceptionInternal(ex, apiError, headers, status, request);
        }
        
        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (2)
      - | Add a method for handling \ ``ResourceNotFoundException``\ .
        | \ ``ResourceNotFoundException``\  exception can be handled if \ ``@ExceptionHandler(ResourceNotFoundException.class)``\  is specified as method annotation.
        | In the above example, the process is delegated to method that handles exception of the parent class (\ ``ResultMessagesNotificationException``\ ) of \ ``ResourceNotFoundException``\ .
        |
        | Set \ **404 (Not Found)**\  in the status code and notify a message stating, 'specified resource does not exist in the server'.
    * - | (3)
      - | Generate a JavaBean object that retains error information for "resource not found" error and business error.
        | In the above example, this method is created as a common method since the process is same as that for error handling of business error discussed hereafter.

|

* When resource is not found, following error response is generated.

 .. code-block:: guess
    :emphasize-lines: 1, 8

    HTTP/1.1 404 Not Found
    Server: Apache-Coyote/1.1
    X-Track: 5ee563877f3140fd904d0acf52eba398
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Wed, 19 Feb 2014 08:46:18 GMT
    
    {"code":"e.ex.mm.5001","message":"Specified member not found. member id : M000000001"}

|

.. _RESTHowToUseExceptionHandlingForBusinessError:

Implementing exception handling for business errors
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
An implementation wherein business error is sent as a response on detecting violation of business rule, is explained here.

Perform business rule check as Service process and generate business exception when a business rule violation is detected.
For details on how to detect business error, refer to "\ :ref:`service-return-businesserrormessage-label`\".

* Create a method to handle business exception in the class that performs error handling.

 .. code-block:: java
    :emphasize-lines: 6-7, 11

    @ControllerAdvice
    public class ApiGlobalExceptionHandler extends ResponseEntityExceptionHandler {

        // omitted

        // (1)
        @ExceptionHandler(BusinessException.class)
        public ResponseEntity<Object> handleBusinessException(BusinessException ex,
                WebRequest request) {
            return handleResultMessagesNotificationException(ex, new HttpHeaders(),
                    HttpStatus.CONFLICT, request);
        }

        // omitted

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Add a method for handling \ ``BusinessException``\ .
        | \ ``BusinessException``\  can be handled if \ ``@ExceptionHandler(BusinessException.class)``\  is specified as method annotation.
        | In the above example, the process is delegated to the method that handles the exception of parent class (\ ``ResultMessagesNotificationException``\ ) of \ ``BusinessException``\ .
        |
        | Set **409 (Conflict)**\ in the status code and send a message notifying although there are no errors in the resource itself specified by client, all the conditions necessary for operating the resource stored by the server are not in place.

|

* Following error response is generated when a business error occurs.

 .. code-block:: guess
    :emphasize-lines: 1, 8

    HTTP/1.1 409 Conflict
    Server: Apache-Coyote/1.1
    X-Track: 37c1a899d5f74e7a9c24662292837ef7
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Wed, 19 Feb 2014 09:03:26 GMT
    
    {"code":"e.ex.mm.8001","message":"Cannot use specified sign id. sign id : user1@test.com"}

|

.. _RESTHowToUseExceptionHandlingForExcusionError:

Implementing exception handling for exclusive errors
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| An implementation is explained here wherein, an exclusive error is generated and sent as a response.
| Exclusive error handling is necessary when performing exclusive control.
| For details of exclusive control, refer to "\ :doc:`ExclusionControl`\" .

* Create a method for exclusive error handling in the class that performs error handling.

 .. code-block:: java
    :emphasize-lines: 6-8, 12

    @ControllerAdvice
    public class ApiGlobalExceptionHandler extends ResponseEntityExceptionHandler {
    
        // omitted

        // (1)
        @ExceptionHandler({ OptimisticLockingFailureException.class,
                PessimisticLockingFailureException.class })
        public ResponseEntity<Object> handleLockingFailureException(Exception ex,
                WebRequest request) {
            return handleExceptionInternal(ex, null, new HttpHeaders(),
                    HttpStatus.CONFLICT, request);
        }
    
        // omitted
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Add a method for handling exclusive errors (\ ``OptimisticLockingFailureException``\  and \ ``PessimisticLockingFailureException``\ ).
        | If \ ``@ExceptionHandler({ OptimisticLockingFailureException.class, PessimisticLockingFailureException.class })``\  is specified as method annotation, exception handling of exclusive errors (\ ``OptimisticLockingFailureException``\  and \ ``PessimisticLockingFailureException``\ ) can be performed.
        |
        | Set \ **409(Conflict)**\  in status code and send a message notifying that, 'although there are no flaws in the resource itself specified by client, the conditions for operating the resource could not be fulfilled due to conflict in the process'.

|

* When an exclusive error occurs, following error response is generated.

 .. code-block:: guess
    :emphasize-lines: 1, 8

    HTTP/1.1 409 Conflict
    Server: Apache-Coyote/1.1
    X-Track: 85200b5a51be42b29840e482ee35087f
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Wed, 19 Feb 2014 16:32:45 GMT
    
    {"code":"e.ex.fw.8002","message":"Conflict with other processing occurred."}

|

.. _RESTHowToUseExceptionHandlingForSystemError:

Implementing exception handling for system errors
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
An implementation wherein system error is sent as a response on detecting system abnormality, is explained here.

Generate system exception when any system abnormality is detected.
Refer to "\ :ref:`service-return-systemerrormessage-label`\" for the details on how to detect system errors.

* Create a method to handle system exceptions in the class that performs error handling.

 .. code-block:: java
    :emphasize-lines: 6-7, 11

    @ControllerAdvice
    public class ApiGlobalExceptionHandler extends ResponseEntityExceptionHandler {
    
        // omitted

        // (1)
        @ExceptionHandler(Exception.class)
        public ResponseEntity<Object> handleSystemError(Exception ex,
                WebRequest request) {
            return handleExceptionInternal(ex, null, new HttpHeaders(),
                    HttpStatus.INTERNAL_SERVER_ERROR, request);
        }
    
        // omitted
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Add a method for handling \ ``Exception``\ .
        | \ ``Exception``\  can be handled if \ ``@ExceptionHandler(Exception.class)``\  is specified as method annotation.
        | In the above example, the system exceptions generated from dependent libraries being used, are also handled.
        |
        | Set **500 (Internal Server Error)**\  in status code.

|

* Following error response is generated when a system error occurs.

 .. code-block:: guess
    :emphasize-lines: 1, 9

    HTTP/1.1 500 Internal Server Error
    Server: Apache-Coyote/1.1
    X-Track: 3625d5a040a744e49b0a9b3763a24e9c
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Wed, 19 Feb 2014 12:22:33 GMT
    Connection: close
    
    {"code":"e.ex.fw.9003","message":"System error occurred."}


 .. warning:: **Error message at the time of system error**

    When system error occurs, it is recommended that a simple error message that does not specify the error cause, is set as the message to be returned to the client.
    If an error message specifying the cause of error is set, it may expose system vulnerabilities to the client resulting in security issues.
    
    Cause of error is output to a log, for error analysis.
    This log is output by \ ``ExceptionLogger``\  provided by common library as the default setting of Blank project. As a result, the settings and implementation for log output are not required.

|

.. _RESTHowToUseExceptionHandlingSettingsOfExceptionCodeResolver:

Resolving error codes and messages using ExceptionCodeResolver
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| If \ ``ExceptionCodeResolver``\  provided by common library is used, error codes can be resolved from the exception class.
| Especially, this functionality proves to be convenient, when the cause of error lies at client side and when it is necessary to set the error message corresponding to error cause.

- | :file:`applicationContext.xml`
  | Mapping exception class and error code (exception code).

 .. code-block:: xml
    
    <!-- omitted -->

    <bean id="exceptionCodeResolver"
        class="org.terasoluna.gfw.common.exception.SimpleMappingExceptionCodeResolver">
        <property name="exceptionMappings">
            <map>
                <!-- omitted -->
                <entry key="ResourceNotFoundException"              value="e.ex.fw.5001" />
                <entry key="HttpRequestMethodNotSupportedException" value="e.ex.fw.6001" />
                <entry key="MediaTypeNotAcceptableException"        value="e.ex.fw.6002" />
                <entry key="HttpMediaTypeNotSupportedException"     value="e.ex.fw.6003" />
                <entry key="MethodArgumentNotValidException"        value="e.ex.fw.7001" />
                <entry key="BindException"                          value="e.ex.fw.7002" />
                <entry key="JsonParseException"                     value="e.ex.fw.7003" />
                <entry key="UnrecognizedPropertyException"          value="e.ex.fw.7004" />
                <entry key="JsonMappingException"                   value="e.ex.fw.7005" />
                <entry key="TypeMismatchException"                  value="e.ex.fw.7006" />
                <entry key="BusinessException"                      value="e.ex.fw.8001" />
                <entry key="OptimisticLockingFailureException"      value="e.ex.fw.8002" />
                <entry key="PessimisticLockingFailureException"     value="e.ex.fw.8002" />
                <entry key="DataAccessException"                    value="e.ex.fw.9002" />
                <!-- omitted -->
            </map>
        </property>
        <property name="defaultExceptionCode" value="e.ex.fw.9001" />
    </bean>

    <!-- omitted -->

|

| Configuration example of message corresponding to error code is shown below.
| For message management, refer to "\ :doc:`MessageManagement`\" .

- | :file:`xxx-web/src/main/resources/i18n/application-messages.properties`
  | Message corresponding to error code (exception code) is set for the error that occurs in application layer.

 .. code-block:: properties

    # ---
    # Application common messages
    # ---
    e.ex.fw.5001 = Resource not found.
    
    e.ex.fw.6001 = Request method not supported.
    e.ex.fw.6002 = Specified representation format not supported.
    e.ex.fw.6003 = Specified media type in the request body not supported.
    
    e.ex.fw.7001 = Validation error occurred on item in the request body.
    e.ex.fw.7002 = Validation error occurred on item in the request parameters.
    e.ex.fw.7003 = Request body format error occurred.
    e.ex.fw.7004 = Unknown field exists in JSON.
    e.ex.fw.7005 = Type mismatch error occurred in JSON field.
    e.ex.fw.7006 = Type mismatch error occurred in request parameter or header or path variable.
    
    e.ex.fw.8001 = Business error occurred.
    e.ex.fw.8002 = Conflict with other processing occurred.
    
    e.ex.fw.9001 = System error occurred.
    e.ex.fw.9002 = System error occurred.
    e.ex.fw.9003 = System error occurred.

    # omitted

- | :file:`xxx-web/src/main/resources/ValidationMessages.properties`
  | Set a message corresponding to error code for the error that occurs in the input validation performed using Bean Validation.

 .. code-block:: properties

    # ---
    # Bean Validation common messages
    # ---
    
    # for bean validation of standard
    javax.validation.constraints.AssertFalse.message = "{0}" must be false.
    javax.validation.constraints.AssertTrue.message  = "{0}" must be true.
    javax.validation.constraints.DecimalMax.message  = "{0}" must be less than ${inclusive == true ? 'or equal to ' : ''}{value}.
    javax.validation.constraints.DecimalMin.message  = "{0}" must be greater than ${inclusive == true ? 'or equal to ' : ''}{value}.
    javax.validation.constraints.Digits.message      = "{0}" numeric value out of bounds (<{integer} digits>.<{fraction} digits> expected).
    javax.validation.constraints.Future.message      = "{0}" must be in the future.
    javax.validation.constraints.Max.message         = "{0}" must be less than or equal to {value}.
    javax.validation.constraints.Min.message         = "{0}" must be greater than or equal to {value}.
    javax.validation.constraints.NotNull.message     = "{0}" may not be null.
    javax.validation.constraints.Null.message        = "{0}" must be null.
    javax.validation.constraints.Past.message        = "{0}" must be in the past.
    javax.validation.constraints.Pattern.message     = "{0}" must match "{regexp}".
    javax.validation.constraints.Size.message        = "{0}" size must be between {min} and {max}.

    # for bean validation of hibernate
    org.hibernate.validator.constraints.CreditCardNumber.message        = "{0}" invalid credit card number.
    org.hibernate.validator.constraints.EAN.message                     = "{0}" invalid {type} barcode.
    org.hibernate.validator.constraints.Email.message                   = "{0}" not a well-formed email address.
    org.hibernate.validator.constraints.Length.message                  = "{0}" length must be between {min} and {max}.
    org.hibernate.validator.constraints.LuhnCheck.message               = "{0}" The check digit for ${validatedValue} is invalid, Luhn Modulo 10 checksum failed.
    org.hibernate.validator.constraints.Mod10Check.message              = "{0}" The check digit for ${validatedValue} is invalid, Modulo 10 checksum failed.
    org.hibernate.validator.constraints.Mod11Check.message              = "{0}" The check digit for ${validatedValue} is invalid, Modulo 11 checksum failed.
    org.hibernate.validator.constraints.ModCheck.message                = "{0}" The check digit for ${validatedValue} is invalid, ${modType} checksum failed.
    org.hibernate.validator.constraints.NotBlank.message                = "{0}" may not be empty.
    org.hibernate.validator.constraints.NotEmpty.message                = "{0}" may not be empty.
    org.hibernate.validator.constraints.ParametersScriptAssert.message  = "{0}" script expression "{script}" didn't evaluate to true.
    org.hibernate.validator.constraints.Range.message                   = "{0}" must be between {min} and {max}.
    org.hibernate.validator.constraints.SafeHtml.message                = "{0}" may have unsafe html content.
    org.hibernate.validator.constraints.ScriptAssert.message            = "{0}" script expression "{script}" didn't evaluate to true.
    org.hibernate.validator.constraints.URL.message                     = "{0}" must be a valid URL.

    org.hibernate.validator.constraints.br.CNPJ.message                 = "{0}" invalid Brazilian corporate taxpayer registry number (CNPJ).
    org.hibernate.validator.constraints.br.CPF.message                  = "{0}" invalid Brazilian individual taxpayer registry number (CPF).
    org.hibernate.validator.constraints.br.TituloEleitoral.message      = "{0}" invalid Brazilian Voter ID card number.
    
    # for common library
    org.terasoluna.gfw.common.codelist.ExistInCodeList.message   = "{0}" must exist in code list of {codeListId}.

- | :file:`xxx-domain/src/main/resources/i18n/domain-messages.properties`
  | Set a message corresponding to error code (exception code) for the error in domain layer.

 .. code-block:: properties

    # omitted

    e.ex.mm.5001 = Specified member not found. member id : {0}
    e.ex.mm.8001 = Cannot use specified sign id. sign id : {0}
    # omitted

|


.. _RESTHowToUseExceptionHandlingForServletContainer:

Implementing the error handling notified to Servlet Container
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
When an error occurs in Filter or when error responses are sent by using ``HttpServletResponse#sendError``\ , the errors cannot be handled using exception handling feature of Spring MVC.
Hence, these errors are notified to Servlet Container.

This section explains how to handle the errors notified to Servlet Container.

 .. figure:: ./images_REST/RESTHowToUseErrorHandlingByServletContainer.png
   :alt: Image of error handling processing by servlet container
   :width: 100%


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 70

    * - Sr. No.
      - Processing layer
      - Description
    * - | (1)
      - | Servlet Container
        | (AP Server)
      - | Servlet Container receives a request from the client and performs process.
        | Servlet Container detects an error during the process.
    * - | (2)
      - | 
      - | Servlet Container performs error handling according to the error page definition in \ ``web.xml``\ .
        | If the error is not fatal, Controller for error handling is called and the error is handled.
    * - | (2')
      - | 
      - | In case of a fatal error, a static JSON file provided in advance is fetched and a response is sent to the client.
    * - | (3)
      - | Spring MVC
        | (Framework)
      - | Spring MVC calls the Controller that performs error handling.
    * - | (4)
      - | Controller
        | (Common Component)
      - | An error object that retains error information is generated in the Controller and is returned to Spring MVC.
    * - | (5)
      - | Spring MVC
        | (Framework)
      - | Spring MVC converts the error object to JSON format message by using \ ``HttpMessageConverter``\ .
    * - | (6)
      - | 
      - | Spring MVC sets the JSON format error message in response BODY and sends a response to client.

|

Implementing the Controller that sends error response
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Create a controller that sends the error response for the error notified to Servlet Container.

 .. code-block:: java
    :emphasize-lines: 14-17, 19-20, 22-27, 31, 34

    package org.terasoluna.examples.rest.api.common.error;
    
    import javax.inject.Inject;
    import javax.servlet.RequestDispatcher;
    
    import org.springframework.http.HttpStatus;
    import org.springframework.http.ResponseEntity;

    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestParam;

    import org.springframework.web.context.request.RequestAttributes;
    import org.springframework.web.bind.annotation.RestController;
    import org.springframework.web.context.request.WebRequest;

    // (1)
    @RequestMapping("error")
    @RestController
    public class ApiErrorPageController {
    
        @Inject
        ApiErrorCreator apiErrorCreator; // (2)
    
        // (3)
        @RequestMapping

        public ResponseEntity<ApiError> handleErrorPage(
                @RequestParam("errorCode") String errorCode, // (4)
                WebRequest request) {
            // (5)
            HttpStatus httpStatus = HttpStatus.valueOf((Integer) request
                    .getAttribute(RequestDispatcher.ERROR_STATUS_CODE,
                            RequestAttributes.SCOPE_REQUEST));
            // (6)
            ApiError apiError = apiErrorCreator.createApiError(request, errorCode,
                    httpStatus.getReasonPhrase());
            // (7)
            return ResponseEntity.status(httpStatus).body(apiError);
        }
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Create a Controller class for sending error response.
        | In the above example, it is mapped to Servlet path "\ ``/api/v1/error``\".
    * - | (2)
      - | Inject a class that creates error information.
    * - | (3)
      - | Create a processing method that sends error response.
        | Above example is implemented on considering only the case wherein, error page is handled by using a response code (\ ``<error-code>``\ ).
        | Therefore, separate consideration is required for the case where an error page that is handled by using exception type (\ ``<exception-type>``\ ) is to be processed using this method.
    * - | (4)
      - | Receive the error code as request parameter.
    * - | (5)
      - | Fetch the status code stored in request scope.
    * - | (6)
      - | Generate error information corresponding to the error code received in request parameter.
    * - | (7)
      - | Send the error information fetched in (5) and (6) as a response.

| 

Creating a static JSON file to be sent as response when a fatal error occurs
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Create a static JSON file to be sent as a response when a fatal error occurs.

- :file:`unhandledSystemError.json`

 .. code-block:: json
    
    {"code":"e.ex.fw.9999","message":"Unhandled system error occurred."}

|

Settings for handling an error that is notified to Servlet Container
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Settings for handling an error that is notified to Servlet Container are explained here.

- :file:`web.xml`

 .. code-block:: xml
    :emphasize-lines: 3, 9, 15

    <!-- omitted -->

    <!-- (1) -->
    <error-page>
        <error-code>404</error-code>
        <location>/api/v1/error?errorCode=e.ex.fw.5001</location>
    </error-page>

    <!-- (2) -->
    <error-page>
        <exception-type>java.lang.Exception</exception-type>
        <location>/WEB-INF/views/common/error/unhandledSystemError.json</location>
    </error-page>

    <!-- (3) -->
    <mime-mapping>
        <extension>json</extension>
        <mime-type>application/json;charset=UTF-8</mime-type>
    </mime-mapping>

    <!-- omitted -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Add error page definition for response code if required.
        | In the above example, when  error \ ``"404 Not Found"``\  occurs, Controller (\`` ApiErrorPageController``\ ) that is mapped in request "\ ``/api/v1/error?errorCode=e.ex.fw.5001``\ " is called and error response is sent.
    * - | (2)
      - | Add definition for handling a fatal error.
        | When a fatal error occurs, it is recommended to respond with the static JSON provided in advance, as double failure may occur during the process that creates response information.
        | In the above example, fixed JSON defined in "\ ``/WEB-INF/views/common/error/unhandledSystemError.json``\ " is sent as response.
    * - | (3)
      - | Specify MIME type of json.
        | When multi byte characters are included in the JSON file created in (2), junk characters may be displayed at client side if \ ``charset=UTF-8``\  is not specified.
        | When there are no multi byte characters in the JSON file, this setting is not mandatory. However, it is always safe to incorporate this setting.

|

* When the request is sent to a non-existing path, following error response is sent.

 .. code-block:: guess
    :emphasize-lines: 1, 8

    HTTP/1.1 404 Not Found
    Server: Apache-Coyote/1.1
    X-Track: 2ad50fb5ba2441699c91a5b01edef83f
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Wed, 19 Feb 2014 23:24:20 GMT
    
    {"code":"e.ex.fw.5001","message":"Resource not found."}

|

* When a fatal error occurs, following error response is sent.

 .. code-block:: guess
    :emphasize-lines: 1, 9

    HTTP/1.1 500 Internal Server Error
    Server: Apache-Coyote/1.1
    X-Track: 69db3854a19f439781584321d9ce8336
    Content-Type: application/json
    Content-Length: 68
    Date: Thu, 20 Feb 2014 00:13:43 GMT
    Connection: close
    
    {"code":"e.ex.fw.9999","message":"Unhandled system error occurred."}

|

.. _RESTHowToUseSecurity:

Security measures
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Implementing security measures for RESTful Web Service is explained here.
| This guideline recommends using Spring Security for implementation of security measures.

.. _RESTHowToUseSecurityAuth:

Authentication and Authorization
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. todo:: **TBD**

    How to implement authentication and authorization using OAuth2 (Spring Security OAuth2), will be explained in subsequent versions.

|

.. _RESTHowToUseSecurityCsrf:

CSRF measures
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* Refer to \ :ref:`CSRF Measures <csrf_spring-security-setting>`\  for the setting methods when CSRF measures are carried out for RESTful Web Service.

* Refer to :ref:`RESTAppendixDisabledCSRFProtection`\  for the setting methods when CSRF measures are not carried out for RESTful Web Service.


|

.. _RESTHowToUseConditionalOperations:

Conditional operations for resource
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo:: **TBD**

    How to implement conditional process control using headers like Etag etc. will be explained in subsequent versions.


|

.. _RESTHowToUseCacheControl:

Cache control for resource
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo:: **TBD**

    How to implement cache control using headers like Cache-Control/Expires/Pragma etc. will be explained in subsequent versions.

|

.. _RESTAppendix:

Appendix
--------------------------------------------------------------------------------

.. _RESTAppendixSettingsOfDeployInSameWarFileRestAndClientApplication:

Settings when RESTful Web Service and client application are operated as the same Web application
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _RESTAppendixDivideDispatcherServlet:

How to set \ ``DispatcherServlet``\  for RESTful Web Service
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| When RESTful Web Service and client application are built as same Web application, it is recommended to divide as \ ``DispatcherServlet``\  that receives the requests for RESTful Web Service and \ ``DispatcherServlet``\  that receives client application requests.
| How to divide \ ``DispatcherServlet``\  is explained below.

- :file:`web.xml`

 .. code-block:: xml
    :emphasize-lines: 3,17-19,19-20,24-25,29-33

    <!-- omitted -->

    <!-- (1) -->
    <servlet>
        <servlet-name>appServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath*:META-INF/spring/spring-mvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>appServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!-- (2) -->
    <servlet>
        <servlet-name>restAppServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <!-- (3) -->
            <param-value>classpath*:META-INF/spring/spring-mvc-rest.xml</param-value>
        </init-param>
        <load-on-startup>2</load-on-startup>
    </servlet>
    <!-- (4) -->
    <servlet-mapping>
        <servlet-name>restAppServlet</servlet-name>
        <url-pattern>/api/v1/*</url-pattern>
    </servlet-mapping>

    <!-- omitted -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Request mapping with the \ ``DispatcherServlet``\  that receives request for client application.
    * - | (2)
      - | Add Servlet (\ ``DispatcherServlet``\ ) that receives the request for RESTful Web Service.
        | Specify a name indicating it as RESTful Web Service Servlet in \ ``<servlet-name>``\  element.
        | In the above example, \ ``"restAppServlet"``\  is specified as the Servlet name.
    * - | (3)
      - | Specify Spring MVC bean definition file to be used while building the \ ``DispatcherServlet``\  for RESTful Web Service.
        | In the above example, \ :file:`META-INF/spring/spring-mvc-rest.xml`\  in class path, is specified as the Spring MVC bean definition file.
    * - | (4)
      - | Specify the pattern of servlet path mapped to the \ ``DispatcherServlet``\ for RESTful Web Service.
        | In the above example, the Servlet path under \ ``"/api/v1/"``\  is mapped in the \ ``DispatcherServlet``\  for RESTful Web Service.
        | Typically, Servlet paths like 
        |   \ ``"/api/v1/"``\
        |   \ ``"/api/v1/members"``\
        |   \ ``"/api/v1/members/xxxxx"``\
        | are mapped in the \ ``DispatcherServlet``\  (\ ``"restAppServlet"``\ ) for RESTful Web Service.

 .. tip:: **Value to be specified in the value attribute of @RequestMapping annotation**

   Value in the wild card (\ ``*``\ ) part specified in \ ``<url-pattern>``\  element, is specified as the value in "value" attribute of \ ``@RequestMapping``\  annotation.
   
   For example, when \ ``@RequestMapping(value = "members")``\  is specified, it is deployed as the method that performs process for path \ ``"/api/v1/members"``\ .
   Therefore, it is not necessary to specify the path (\ ``"api/v1"``\ ) that performs mapping in divided Servlets, in the value attribute of \ ``@RequestMapping``\  annotation.
   
   When specified as \ ``@RequestMapping(value = "api/v1/members")``\ , it ends up being deployed as the method performing the process for path \ ``"/api/v1/api/v1/members"``\ . Hence, please take note of same.

|

.. _RESTAppendixHyperMediaLink:

Implementing hypermedia link
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Implementation for including a hypermedia link to a related resource in JSON, is explained here.

Implementing common parts
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* Create a JavaBean that retains the link information.

 .. code-block:: java
    :emphasize-lines: 5
 
    package org.terasoluna.examples.rest.api.common.resource;
    
    import java.io.Serializable;
    
    // (1)
    public class Link implements Serializable {
    
        private static final long serialVersionUID = 1L;

        private final String rel;

        private final String href;
    
        public Link(String rel, String href) {
            this.rel = rel;
            this.href = href;
        }
    
        public String getRel() {
            return rel;
        }
    
        public String getHref() {
            return href;
        }
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Create a JavaBean for link information that retains the link name and URL.

|

* Create an abstract class of Resource that retains collection of link information.

 .. code-block:: java
    :emphasize-lines: 9, 12, 20, 26, 31

    package org.terasoluna.examples.rest.api.common.resource;
    
    import java.net.URI;
    import java.util.LinkedHashSet;
    import java.util.Set;
    
    import com.fasterxml.jackson.annotation.JsonInclude;
    
    // (2)
    public abstract class AbstractLinksSupportedResource {

        // (3)
        @JsonInclude(JsonInclude.Include.NON_EMPTY)
        private final Set<Link> links = new LinkedHashSet<>();
    
        public Set<Link> getLinks() {
            return links;
        }
    
        // (4)
        public AbstractLinksSupportedResource addLink(String rel, URI href) {
            links.add(new Link(rel, href.toString()));
            return this;
        }
    
        // (5)
        public AbstractLinksSupportedResource addSelf(URI href) {
            return addLink("self", href);
        }
    
        // (5)
        public AbstractLinksSupportedResource addParent(URI href) {
            return addLink("parent", href);
        }
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (2)
      - | Create an abstract class (JavaBean) of the Resource that retains collection of link information.
        | It is assumed that this is the class inherited by the Resource class supporting hypermedia link.
    * - | (3)
      - | Define a field that retains information of multiple links.
        | In the above example, \ ``@JsonInclude(JsonInclude.Include.NON_EMPTY)``\  is specified to prevent output to JSON when link is not specified.
    * - | (4)
      - | Create a method to add link information.
    * - | (5)
      - | If required, create a method to add common link information.
        | In the above example, methods to add link information for accessing the resource itself (\ ``"self"``\ ) and to add link information for accessing parent resource (\ ``"parent"``\ ) are created.

|

Implementation for each resource
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* Inherit an abstract class of Resource that retains collection of link information in Resource class.

 .. code-block:: java
    :emphasize-lines: 3

    package org.terasoluna.examples.rest.api.member;
    
    // (1)
    public class MemberResource extends 
        AbstractLinksSupportedResource implements Serializable {

        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Inherit an abstract class of Resource that retains collection of link information.
        | By this, the field (\ ``links``\ ) that retains collection of link information is imported, thus making it a Resource class that supports hypermedia link.


* Adding hypermedia link by REST API process.

 .. code-block:: java
    :emphasize-lines: 11, 19

    @RequestMapping("members")
    @RestController
    public class MemberRestController {

        // omitted


        @RequestMapping(value = "{memberId}", method = RequestMethod.GET)

        @ResponseStatus(HttpStatus.OK)
        public MemberResource getMember(
                @PathVariable("memberId") String memberId
                // (2)
                UriComponentsBuilder uriBuilder) {
    
            Member member = memberService.getMember(memberId);
    
            MemberResource responseResource = beanMapper.map(member,
                    MemberResource.class);
    
            // (3)
            responseResource.addSelf(uriBuilder.path("/members").pathSegment(memberId)
                    .build().toUri());

            return responseResource;
        }

        // omitted

    }
    
 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (2)
      - | Specify \ ``org.springframework.web.util.UriComponentsBuilder``\ 
        | class, provided by Spring MVC, in method argument in order to build URI set in link information.
        | When \ ``UriComponentsBuilder``\  class is specified in method argument of Controller, instance of \ ``org.springframework.web.servlet.support.ServletUriComponentsBuilder``\  class, 
        | that has inherited \ ``UriComponentsBuilder``\  class in Spring MVC, is passed at the time of method execution.
    * - | (3)
      - | Add link information to resource.
        | In the above example, \ ``UriComponentsBuilder``\  class method is called to build URI set in link information and URI to access to one's resource is added to resource.
        |
        | \ ``ServletUriComponentsBuilder``\  instances passed as method argument of Controller are initiated based on the information of \ ``<servlet-mapping>``\  element described in web.xml. It is not resource dependent.
        | By using `URI Template Patterns <http://docs.spring.io/spring/docs/4.1.7.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-requestmapping-uri-templates>`_\  ,etc. provided in Spring Framework,
        | an URI can be built based on the request information. Thus, a generic build process which is independent of a resource, can be implemented.
        | 
        | In the above example, when GET is done for \ ``http://example.com/api/v1/members/M000000001``\ , built URI is same as the requested URI \ ``(http://example.com/api/v1/members/M000000001)``\ .
        | 
        | Method that builds an URI to be set in link information should be implemented as required.

 .. tip::

    When building an URL in \ ``ServletUriComponentsBuilder``\ , by referring to "\ ``X-Forwarded-Host``\  " header, a configuration with a load balancer or Web server between client and the application server is considered.
    However, it should be noted that the expected URI will not be built if it does not match with the path configuration.

* | Response example
  | Following response is obtained in actual operation.

 .. code-block:: guess

    GET /rest-api-web/api/v1/members/M000000001 HTTP/1.1
    Accept: text/plain, application/json, application/*+json, */*
    User-Agent: Java/1.7.0_51
    Host: localhost:8080
    Connection: keep-alive

 .. code-block:: java
    :emphasize-lines: 2-5

    {
      "links" : [ {
        "rel" : "self",
        "href" : "http://localhost:8080/rest-api-web/api/v1/members/M000000001"
      } ],
      "memberId" : "M000000001",
      "firstName" : "John",
      "lastName" : "Smith",
      "genderCode" : "1",
      "dateOfBirth" : "2013-03-14",
      "emailAddress" : "user1394794959984@test.com",
      "telephoneNumber" : "09012345678",
      "zipCode" : "1710051",
      "address" : "Tokyo",
      "credential" : {
        "signId" : "user1394794959984@test.com",
        "passwordLastChangedAt" : "2014-03-14T11:02:41.477Z",
        "lastModifiedAt" : "2014-03-14T11:02:41.477Z"
      },
      "createdAt" : "2014-03-14T11:02:41.477Z",
      "lastModifiedAt" : "2014-03-14T11:02:41.477Z"
    }

|

.. _RESTAppendixRestApiOfHTTPCompliance:

Creating RESTful Web Service conforming to HTTP specifications
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| A part of REST API implementation explained in this version does not conform to HTTP specifications.
| This section explains the implementation that creates the RESTful Web Service conforming to HTTP specifications.

Settings for location header at the time of POST
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| When conforming to HTTP specifications, it is necessary to set the URI of created resource in the response header of POST method ("Location header").
| Implementation to set the URI of created resource in the response header in case of POST ("Location header") is explained here.


Implementation for each resource
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* The URI of created resource is set in Location header by REST API process.

 .. code-block:: java
    :emphasize-lines: 11, 21, 25

    @RequestMapping("members")
    @RestController
    public class MemberRestController {

        // omitted


        @RequestMapping(method = RequestMethod.POST)
        public ResponseEntity<MemberResource> postMembers(
                @RequestBody @Validated({ PostMembers.class, Default.class })
                MemberResource requestedResource,
                // (1)
                UriComponentsBuilder uriBuilder) {
    
            Member creatingMember = beanMapper.map(requestedResource, Member.class);
    
            Member createdMember = memberService.createMember(creatingMember);
    
            MemberResource responseResource = beanMapper.map(createdMember,
                    MemberResource.class);

            // (2)
            URI createdUri = uriBuilder.path("/members/{memberId}")
                    .buildAndExpand(responseResource.getMemberId()).toUri();
    
            // (3)
            return ResponseEntity.created(createdUri).body(responseResource);
        }

        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify \ ``org.springframework.web.util.UriComponentsBuilder``\  class provided by Spring MVC, in method argument in order to build an URI of created resource.
        | When \ ``UriComponentsBuilder``\  class is specified in method argument of Controller, an instance of \ ``org.springframework.web.servlet.support.ServletUriComponentsBuilder``\  class, 
        | that has inherited \ ``UriComponentsBuilder``\  class by Spring MVC, is passed at the time of method execution.
    * - | (2)
      - | Build an URI of created resource.
        | In above example, by \ ``path``\  method, a path using URI Template Patterns is added to \ ``ServletUriComponentsBuilder``\  instance passed as argument,
        | \ ``buildAndExpand``\  method is called and an URI of created resource is built by binding the ID of created resource.
        | 
        | \ ``ServletUriComponentsBuilder``\  instances passed as method argument of Controller are initiated based on the information of \ ``<servlet-mapping>``\  element described in web.xml. It is not resource dependent.
        | By using `URI Template Patterns <http://docs.spring.io/spring/docs/4.1.7.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-requestmapping-uri-templates>`_\  ,etc. provided by Spring Framework,
        | an URI can be built based on the request information. Thus, a generic build process which is independent of a resource, can be implemented.
        | 
        | For example, if POST method is used for \ ``http://example.com/api/v1/members``\ , the built URI will be "requested URI + \ ``"/"``\  + ID of created resource".
        | Typically, if \ ``"M000000001"``\  is specified in ID, it becomes ``http://example.com/api/v1/members/M000000001``\ .
        | 
        | Method that builds an URI to be set in link information should be implemented as required.
    * - | (3)
      - | Create the \ ``org.springframework.http.ResponseEntity``\  instance using following parameters, and return it.

        * Status code : 201(Created)
        * Location header : Created resource's URI
        * Response body : Created resource object

 .. tip::

    When building an URL in \ ``ServletUriComponentsBuilder``\ , by referring to "\ ``X-Forwarded-Host``\  " header, a configuration with a load balancer or Web server between client and the application server is considered.
    However, it should be noted that the expected URI will not be built if it does not match with the path configuration.

* | Response example
  | Following response header is obtained in the actual operation.

 .. code-block:: guess
    :emphasize-lines: 4

    HTTP/1.1 201 Created
    Server: Apache-Coyote/1.1
    X-Track: 693e132312d64998a7d8d6cabf3d13ef
    Location: http://localhost:8080/rest-api-web/api/v1/members/M000000001
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Fri, 14 Mar 2014 12:34:31 GMT

|

.. _RESTAppendixDispatchOptionsMethod:

Setting to dispatch OPTIONS method request to the Controller
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| When conforming to HTTP specifications, it is necessary to return the list of HTTP methods that are allowed to be called for each resource. Therefore, it is necessary to add the setting for dispatching OPTIONS method request, to the Controller.
| By \ ``DispatcherServlet``\  default setting, the request for OPTIONS method is not dispatched in the Controller with the list of methods allowed by \ ``DispatcherServlet``\  being set in the Allow header.

- :file:`web.xml`

 .. code-block:: xml
    :emphasize-lines: 10-14

    <!-- omitted -->

    <servlet>
        <servlet-name>appServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath*:META-INF/spring/spring-mvc-rest.xml</param-value>
        </init-param>
        <!-- (1) -->
        <init-param>
            <param-name>dispatchOptionsRequest</param-name>
            <param-value>true</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <!-- omitted -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | Sr. No.
     - | Description
   * - | (1)
     - | Set initialization parameter (dispatchOptionsRequest) value of \ ``DispatcherServlet``\  that receives RESTful Web Service request to \ ``true``\ .

|

.. _RESTAppendixRestApiOfHTTPComplianceImplementationOfOptionsSpecifiedResource:

Implementing OPTIONS method
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| When conforming to HTTP specifications, it is necessary to return the list of HTTP methods that are allowed to be called for each resource.
| API implementation that responds with list of HTTP methods (REST AP) supported by the resource specified in URI, is shown below.

* | REST API implementation
  | Implement a process wherein, list of HTTP methods (REST API) supported by the resource specified in URI is sent as response.

 .. code-block:: java
    :emphasize-lines: 11, 14

    @RequestMapping("members")
    @RestController
    public class MembersRestController {

        // omitted

        @RequestMapping(value = "{memberId}", method = RequestMethod.OPTIONS)
        public ResponseEntity<Void> optionsMember(
            @PathVariable("memberId") String memberId) {

            // (1)
            memberService.getMember(memberId);

            // (2)
            return ResponseEntity
                    .ok()
                    .allow(HttpMethod.GET, HttpMethod.HEAD, HttpMethod.PUT,
                            HttpMethod.DELETE, HttpMethod.OPTIONS).build();
        }
    
        // omitted

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Call domain layer Service method and check to confirm if a resource matching with the ID fetched from path variable exists.
    * - | (2)
      - | **Set HTTP method supported by resource specified in URI, in Allow header.**

|

* Request example

 .. code-block:: guess
    :emphasize-lines: 1

    OPTIONS /rest-api-web/api/v1/members/M000000004 HTTP/1.1
    Accept: text/plain, application/json, application/*+json, */*
    User-Agent: Java/1.7.0_51
    Host: localhost:8080
    Connection: keep-alive


|

* Response example

 .. code-block:: guess
    :emphasize-lines: 4

    HTTP/1.1 200 OK
    Server: Apache-Coyote/1.1
    X-Track: 6d7bbc818c7f44e7942c54bc0ddc15bb
    Allow: GET,HEAD,PUT,DELETE,OPTIONS
    Content-Length: 0
    Date: Mon, 17 Mar 2014 01:54:27 GMT

|

Implementing HEAD method
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| In order to conform to HTTP specifications, when GET method is implemented, HEAD method also needs to be implemented.
| API implementation that responds with meta-information of the resource specified in URI, is as follows:


* | REST API implementation
  | A process is implemented wherein meta information of the resource specified in URI is fetched.

 .. code-block:: java
    :emphasize-lines: 9

    @RequestMapping("members")
    @RestController
    public class MemberRestController {

        // omitted

        @RequestMapping(value = "{memberId}",
                        method = { RequestMethod.GET,
                                   RequestMethod.HEAD }) // (1)

        @ResponseStatus(HttpStatus.OK)
        public MemberResource getMember(
                @PathVariable("memberId") String memberId) {
            // omitted
        }

        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Add \ ``RequestMethod.HEAD``\  to the method attribute of REST API \ ``@RequestMapping``\  annotation that processes the GET method.
        | HEAD method needs to respond only the header information, by performing the same process as GET method. Therefore, \ ``RequestMethod.HEAD``\  is also specified in the method attribute of \ ``@RequestMapping``\  annotation.
        | It is advisable to perform a process similar to GET process as the Controller process since, the process for emptying response BODY is performed by standard functionality of Servlet API.

 |
 
* Request example
 
 .. code-block:: guess
    :emphasize-lines: 1
 
    HEAD /rest-api-web/api/v1/members/M000000001 HTTP/1.1
    Accept: text/plain, application/json, application/*+json, */*
    User-Agent: Java/1.7.0_51
    Host: localhost:8080
    Connection: keep-alive

 |
 
* Response example

 .. code-block:: guess
    :emphasize-lines: 1, 4, 5

    HTTP/1.1 200 OK
    Server: Apache-Coyote/1.1
    X-Track: 71093a551e624c149867b6bfec486d2c
    Content-Type: application/json;charset=UTF-8
    Content-Length: 452
    Date: Thu, 13 Mar 2014 13:25:23 GMT
 

|



.. _RESTAppendixDisabledCSRFProtection:

Disabling CSRF measures
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Settings that prevent CSRF measures implementation for RESTful Web Service requests, are explained below.

 .. tip::

    Need to use session is eliminated when CSRF measures are not to be implemented.
    
    In the following configuration example, session will not be used in Spring Security process.

|

CSRF measures are enabled as per the default settings of Blank project. By adding following settings, the process for CSRF measures is
prevented for RESTful Web Service requests.

* :file:`spring-security.xml`

 .. code-block:: xml
    :emphasize-lines: 3-10

    <!-- omitted -->

    <!-- (1) -->
    <sec:http
        pattern="/api/v1/**"
        auto-config="true"
        use-expressions="true"
        create-session="stateless">
        <sec:headers />
    </sec:http>

    <sec:http auto-config="true" use-expressions="true">
        <sec:headers>
            <sec:cache-control />
            <sec:content-type-options />
            <sec:hsts />
            <sec:frame-options />
            <sec:xss-protection />
        </sec:headers>
        <sec:csrf />
        <sec:access-denied-handler ref="accessDeniedHandler"/>
        <sec:custom-filter ref="userIdMDCPutFilter" after="ANONYMOUS_FILTER"/>
        <sec:session-management />
    </sec:http>

    <!-- omitted -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Add Spring Security definition for REST API.
       | URL pattern of REST API request path is specified in \ ``pattern``\  attribute of \ ``<sec:http>``\  element.
       | In the above example, request path starting with \ ``/api/v1/``\  is handled as the REST API request path.
       | Further, session is no longer used in Spring Security process by setting \ ``create-session``\  attribute to \ ``stateless``\ .
       |
       | \ ``<sec: csrf>``\  element is not specified to disable the CSRF measures.

|

.. _RESTAppendixEnabledXXEInjectProtection:

Enabling XXE Injection measures
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| When handling XML format data in RESTful Web Service, it is necessary to implement \ `XXE (XML External Entity) Injection <https://www.owasp.org/index.php/XML_External_Entity_(XXE)_Processing>`_\  measures.
| terasoluna-gfw-web 1.0.1.RELEASE or higher versions are Spring MVC (3.2.10.RELEASE and above) dependent. As these Spring MVC versions implement XXE Injection measures, it is not necessary to implement them individually.

 .. warning:: **XXE (XML External Entity) Injection measures**

        When using terasoluna-gfw-web 1.0.0.RELEASE, a class provided by Spring-oxm should be used, as this version is dependent on the Spring MVC version (3.2.4.RELEASE), that does not implement XXE Injection measures.

|

Adding Spring-oxm as dependent artifact.

- :file:`pom.xml`

 .. code-block:: xml

    <!-- omitted -->

    <!-- (1) -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-oxm</artifactId>
        <version>${org.springframework-version}</version> <!-- (2) -->
    </dependency>

    <!-- omitted -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | Sr. No.
     - | Description
   * - | (1)
     - | Add Spring-oxm as a dependent artifact.
   * - | (2)
     - | Spring version should be fetched from the placeholder (${org.springframework-version}) that manages version number of Spring defined in :file:`pom.xml` of terasoluna-gfw-parent.

|

Perform Bean definition for mutual conversion between XML and object by using the class provided by Spring-oxm.

- :file:`spring-mvc-rest.xml`

 .. code-block:: xml

    <!-- omitted -->

    <!-- (1) -->
    <bean id="xmlMarshaller" class="org.springframework.oxm.jaxb.Jaxb2Marshaller">
        <property name="packagesToScan" value="com.examples.app" /> <!-- (2) -->
    </bean>

    <!-- omitted -->

    <mvc:annotation-driven>

        <mvc:message-converters>
            <!-- (3) -->
            <bean class="org.springframework.http.converter.xml.MarshallingHttpMessageConverter">
                <property name="marshaller" ref="xmlMarshaller" /> <!-- (4) -->
                <property name="unmarshaller" ref="xmlMarshaller" /> <!-- (5) -->
            </bean>
        </mvc:message-converters>

        <!-- omitted -->

    </mvc:annotation-driven>

    <!-- omitted -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | Sr. No.
     - | Description
   * - | (1)
     - | Perform Bean definition for \ ``Jaxb2Marshaller``\  provided by Spring-oxm.
       | For \ ``Jaxb2Marshaller``\ , XXE injection measures are carried out by default.
   * - | (2)
     - | Specify package name with JAXB JavaBean (JavaBean assigned with ``javax.xml.bind.annotation.XmlRootElement`` annotation) stored in ``packagesToScan`` property.
       | JAXB JavaBean stored under the specified package is scanned and is registered as JavaBean for marshalling and unmarshalling.
       | JavaBean is scanned by the mechanism same as base-package attribute of ``<context:component-scan>``.
   * - | (3)
     - | Add bean definition of ``MarshallingHttpMessageConverter`` to ``<mvc:message-converters>`` element which is the child element of ``<mvc:annotation-driven>`` element.
   * - | (4)
     - | Specify ``Jaxb2Marshaller`` bean defined in (1) in ``marshaller`` property.
   * - | (5)
     - | Specify ``Jaxb2Marshaller`` bean defined in (1) in ``unmarshaller`` property.

|

.. _RESTAppendixCopyJodaObjectByBeanConvert:

How to copy Joda-Time classes using Dozer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

How to copy Joda-Time classes (\ ``org.joda.time.DateTime``\ , \ ``org.joda.time.LocalDate``\  etc.) using Dozer is explained here.

| Create a custom converter for converting the Joda-Time class.
| For details of custom converter, refer to ":doc:`Utilities/Dozer`".

* :file:`JodaDateTimeConverter.java`

 .. code-block:: java
  
    package org.terasoluna.examples.rest.infra.dozer.converter;
    
    import org.dozer.DozerConverter;
    import org.joda.time.DateTime;
    
    public class JodaDateTimeConverter extends DozerConverter<DateTime, DateTime> {
    
        public JodaDateTimeConverter() {
            super(DateTime.class, DateTime.class);
        }
    
        @Override
        public DateTime convertTo(DateTime source, DateTime destination) {
            // This method not called, because type of from/to is same.
            return convertFrom(source, destination);
        }
    
        @Override
        public DateTime convertFrom(DateTime source, DateTime destination) {
            return source;
        }
    
    }


* :file:`JodaLocalDateConverter.java`

 .. code-block:: java

    package org.terasoluna.examples.rest.infra.dozer.converter;
    
    import org.dozer.DozerConverter;
    import org.joda.time.LocalDate;
    
    public class JodaLocalDateConverter extends
                                       DozerConverter<LocalDate, LocalDate> {
    
        public JodaLocalDateConverter() {
            super(LocalDate.class, LocalDate.class);
        }
    
        @Override
        public LocalDate convertTo(LocalDate source, LocalDate destination) {
            // This method not called, because type of from/to is same.
            return convertFrom(source, destination);
        }
    
        @Override
        public LocalDate convertFrom(LocalDate source, LocalDate destination) {
            return source;
        }
    
    }

| Apply the created Custom converter to Dozer.
| For details of custom converter, refer to :doc:`Utilities/Dozer`.

 .. code-block:: xml
    :emphasize-lines: 1, 10-18
  
    <!-- (1) -->
    <?xml version="1.0" encoding="UTF-8"?>
    <mappings xmlns="http://dozer.sourceforge.net" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="
            http://dozer.sourceforge.net http://dozer.sourceforge.net/schema/beanmapping.xsd
        ">
    
        <configuration>
            <custom-converters>
                <!-- (2) -->
                <converter type="org.terasoluna.examples.rest.infra.dozer.converter.JodaDateTimeConverter">
                    <class-a>org.joda.time.DateTime</class-a>
                    <class-b>org.joda.time.DateTime</class-b>
                </converter>
                <converter type="org.terasoluna.examples.rest.infra.dozer.converter.JodaLocalDateConverter">
                    <class-a>org.joda.time.LocalDate</class-a>
                    <class-b>org.joda.time.LocalDate</class-b>
                </converter>
            </custom-converters>
        </configuration>
    
    </mappings>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Create a file that defines operation settings for Dozer.
        | 
        | In this implementation, it is stored in \ :file:`/xxx-domain/src/main/resources/META-INF/dozer/dozer-configration-mapping.xml`\ .
    * - | (2)
      - | In the above example, custom converter definitions for Joda-Time classes (\ ``org.joda.time.DateTime``\  and \ ``org.joda.time.LocalDate``\ ) are added.


 .. note::
 
    When Dozer is used in domain layer as well, it is recommended to store the file defining Dozer operation settings, in domain layer project (\ ``xxx-domain``\ ).
    
    When Dozer is used only in application layer, it may be stored in application layer project (\ ``xxx-web``\ ).

|


.. _RESTAppendixSoruceCodesOfApplicationLayer:

Source code for application layer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Out of the application layer source codes used in explanation \ :ref:`RESTHowToUse`\ , full version of the source code that was pasted in fragments, is attached herewith.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.45\linewidth}|p{0.45\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 45 45

   * - | Sr. No.
     - | Section
     - | File name
   * - | (1)
     - | :ref:`RESTHowToUseApiImplementation`
     - | :ref:`MemberRestController.java <RESTAppendixSoruceCodesOfMemberRestController>`
   * - | (2)
     - | :ref:`RESTHowToUseExceptionHandling`
     - | :ref:`ApiErrorCreator.java <RESTAppendixSoruceCodesOfApiErrorCreator>`
   * - | (3)
     - | 
     - | :ref:`ApiGlobalExceptionHandler.java <RESTAppendixSoruceCodesOfApiGlobalExceptionHandler>`

 Following files are excluded.

 * JavaBean
 * Configuration file


|

.. _RESTAppendixSoruceCodesOfMemberRestController:

MemberRestController.java
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

:file:`java/org/terasoluna/examples/rest/api/member/MemberRestController.java`

.. code-block:: java

    package org.terasoluna.examples.rest.api.member;
    
    import java.util.ArrayList;
    import java.util.List;
    
    import javax.inject.Inject;
    import javax.validation.groups.Default;
    
    import org.dozer.Mapper;
    import org.springframework.data.domain.Page;
    import org.springframework.data.domain.PageImpl;
    import org.springframework.data.domain.Pageable;
    import org.springframework.http.HttpStatus;

    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.PathVariable;
    import org.springframework.web.bind.annotation.RequestBody;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;

    import org.springframework.web.bind.annotation.ResponseStatus;
    import org.springframework.web.bind.annotation.RestController;
    import org.terasoluna.examples.rest.api.member.MemberResource.PostMembers;
    import org.terasoluna.examples.rest.api.member.MemberResource.PutMember;
    import org.terasoluna.examples.rest.domain.model.Member;
    import org.terasoluna.examples.rest.domain.service.member.MemberService;
    
    @RequestMapping("members")
    @RestController
    public class MemberRestController {
    
        @Inject
        MemberService memberService;
    
        @Inject
        Mapper beanMapper;
    
        @RequestMapping(method = RequestMethod.GET)

        @ResponseStatus(HttpStatus.OK)
        public Page<MemberResource> getMembers(@Validated MembersSearchQuery query,
                Pageable pageable) {
    
            Page<Member> page = memberService.searchMembers(query.getName(), pageable);
    
            List<MemberResource> memberResources = new ArrayList<>();
            for (Member member : page.getContent()) {
                memberResources.add(beanMapper.map(member, MemberResource.class));
            }
            Page<MemberResource> responseResource =
                new PageImpl<>(memberResources, pageable, page.getTotalElements());
    
            return responseResource;
        }
    
        @RequestMapping(method = RequestMethod.POST)

        @ResponseStatus(HttpStatus.CREATED)
        public MemberResource postMembers(@RequestBody @Validated({
                PostMembers.class, Default.class }) MemberResource requestedResource) {
    
            Member creatingMember = beanMapper.map(requestedResource, Member.class);
    
            Member createdMember = memberService.createMember(creatingMember);
    
            MemberResource responseResource = beanMapper.map(createdMember,
                    MemberResource.class);
    
            return responseResource;
        }
    
        @RequestMapping(value = "{memberId}", method = RequestMethod.GET)

        @ResponseStatus(HttpStatus.OK)
        public MemberResource getMember(@PathVariable("memberId") String memberId) {
    
            Member member = memberService.getMember(memberId);
    
            MemberResource responseResource = beanMapper.map(member,
                    MemberResource.class);
    
            return responseResource;
        }
    
        @RequestMapping(value = "{memberId}", method = RequestMethod.PUT)

        @ResponseStatus(HttpStatus.OK)
        public MemberResource putMember(
                @PathVariable("memberId") String memberId,
                @RequestBody @Validated({
                PutMember.class, Default.class }) MemberResource requestedResource) {
    
            Member updatingMember = beanMapper.map(requestedResource, Member.class);
    
            Member updatedMember = memberService.updateMember(memberId,
                    updatingMember);
    
            Member updatedMember = memberService.updateMember(memberId,
                    MemberResource.class);
    
            return responseResource;
        }
    
        @RequestMapping(value = "{memberId}", method = RequestMethod.DELETE)

        @ResponseStatus(HttpStatus.NO_CONTENT)
        public void deleteMember(@PathVariable("memberId") String memberId) {
    
            memberService.deleteMember(memberId);
    
        }
    
    }

|

.. _RESTAppendixSoruceCodesOfApiErrorCreator:

ApiErrorCreator.java
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

:file:`java/org/terasoluna/examples/rest/api/common/error/ApiErrorCreator.java`

.. code-block:: java

    package org.terasoluna.examples.rest.api.common.error;
    
    import javax.inject.Inject;
    
    import org.springframework.context.MessageSource;
    import org.springframework.context.support.DefaultMessageSourceResolvable;
    import org.springframework.stereotype.Component;
    import org.springframework.validation.BindingResult;
    import org.springframework.validation.FieldError;
    import org.springframework.validation.ObjectError;
    import org.springframework.web.context.request.WebRequest;
    import org.terasoluna.gfw.common.message.ResultMessage;
    import org.terasoluna.gfw.common.message.ResultMessages;
    
    @Component
    public class ApiErrorCreator {
    
        @Inject
        MessageSource messageSource;
    
        public ApiError createApiError(WebRequest request, String errorCode,
                String defaultErrorMessage, Object... arguments) {
            String localizedMessage = messageSource.getMessage(errorCode,
                    arguments, defaultErrorMessage, request.getLocale());
            return new ApiError(errorCode, localizedMessage);
        }
    
        public ApiError createBindingResultApiError(WebRequest request,
                String errorCode, BindingResult bindingResult,
                String defaultErrorMessage) {
            ApiError apiError = createApiError(request, errorCode,
                    defaultErrorMessage);
            for (FieldError fieldError : bindingResult.getFieldErrors()) {
                apiError.addDetail(createApiError(request, fieldError, fieldError
                        .getField()));
            }
            for (ObjectError objectError : bindingResult.getGlobalErrors()) {
                apiError.addDetail(createApiError(request, objectError, objectError
                        .getObjectName()));
            }
            return apiError;
        }
    
        private ApiError createApiError(WebRequest request,
                DefaultMessageSourceResolvable messageResolvable, String target) {
            String localizedMessage = messageSource.getMessage(messageResolvable,
                    request.getLocale());
            return new ApiError(messageResolvable.getCode(), localizedMessage, target);
        }
    
        public ApiError createResultMessagesApiError(WebRequest request,
                String rootErrorCode, ResultMessages resultMessages,
                String defaultErrorMessage) {
            ApiError apiError;
            if (resultMessages.getList().size() == 1) {
                ResultMessage resultMessage = resultMessages.iterator().next();
                String errorCode = resultMessage.getCode();
                String errorText = resultMessage.getText();
                if (errorCode == null && errorText == null) {
                    errorCode = rootErrorCode;
                }
                apiError = createApiError(request, errorCode, errorText,
                        resultMessage.getArgs());
            } else {
                apiError = createApiError(request, rootErrorCode,
                        defaultErrorMessage);
                for (ResultMessage resultMessage : resultMessages.getList()) {
                    apiError.addDetail(createApiError(request, resultMessage
                            .getCode(), resultMessage.getText(), resultMessage
                            .getArgs()));
                }
            }
            return apiError;
        }
    
    }

|

.. _RESTAppendixSoruceCodesOfApiGlobalExceptionHandler:

ApiGlobalExceptionHandler.java
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

:file:`java/org/terasoluna/examples/rest/api/common/error/ApiGlobalExceptionHandler.java`

.. code-block:: java

    package org.terasoluna.examples.rest.api.common.error;
    
    import javax.inject.Inject;
    
    import org.springframework.dao.OptimisticLockingFailureException;
    import org.springframework.dao.PessimisticLockingFailureException;
    import org.springframework.http.HttpHeaders;
    import org.springframework.http.HttpStatus;
    import org.springframework.http.ResponseEntity;
    import org.springframework.http.converter.HttpMessageNotReadableException;
    import org.springframework.validation.BindException;
    import org.springframework.validation.BindingResult;
    import org.springframework.web.bind.MethodArgumentNotValidException;
    import org.springframework.web.bind.annotation.ControllerAdvice;
    import org.springframework.web.bind.annotation.ExceptionHandler;
    import org.springframework.web.context.request.WebRequest;
    import org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler;
    import org.terasoluna.gfw.common.exception.BusinessException;
    import org.terasoluna.gfw.common.exception.ExceptionCodeResolver;
    import org.terasoluna.gfw.common.exception.ResourceNotFoundException;
    import org.terasoluna.gfw.common.exception.ResultMessagesNotificationException;
    
    @ControllerAdvice
    public class ApiGlobalExceptionHandler extends ResponseEntityExceptionHandler {
    
        @Inject
        ApiErrorCreator apiErrorCreator;
    
        @Inject
        ExceptionCodeResolver exceptionCodeResolver;
    
        @Override
        protected ResponseEntity<Object> handleExceptionInternal(Exception ex,
                Object body, HttpHeaders headers, HttpStatus status,
                WebRequest request) {
            final Object apiError;
            if (body == null) {
                String errorCode = exceptionCodeResolver.resolveExceptionCode(ex);
                apiError = apiErrorCreator.createApiError(request, errorCode, ex
                        .getLocalizedMessage());
            } else {
                apiError = body;
            }
            return ResponseEntity.status(status).headers(headers).body(apiError);
        }
    
        @Override
        protected ResponseEntity<Object> handleMethodArgumentNotValid(
                MethodArgumentNotValidException ex, HttpHeaders headers,
                HttpStatus status, WebRequest request) {
            return handleBindingResult(ex, ex.getBindingResult(), headers, status,
                    request);
        }
    
        @Override
        protected ResponseEntity<Object> handleBindException(BindException ex,
                HttpHeaders headers, HttpStatus status, WebRequest request) {
            return handleBindingResult(ex, ex.getBindingResult(), headers, status,
                    request);
        }
    
        private ResponseEntity<Object> handleBindingResult(Exception ex,
                BindingResult bindingResult, HttpHeaders headers,
                HttpStatus status, WebRequest request) {
            String errorCode = exceptionCodeResolver.resolveExceptionCode(ex);
            ApiError apiError = apiErrorCreator.createBindingResultApiError(
                    request, errorCode, bindingResult, ex.getMessage());
            return handleExceptionInternal(ex, apiError, headers, status, request);
        }
    
        @Override
        protected ResponseEntity<Object> handleHttpMessageNotReadable(
                HttpMessageNotReadableException ex, HttpHeaders headers,
                HttpStatus status, WebRequest request) {
            if (ex.getCause() instanceof Exception) {
                return handleExceptionInternal((Exception) ex.getCause(), null,
                        headers, status, request);
            } else {
                return handleExceptionInternal(ex, null, headers, status, request);
            }
        }
    
        @ExceptionHandler(ResourceNotFoundException.class)
        public ResponseEntity<Object> handleResourceNotFoundException(
                ResourceNotFoundException ex, WebRequest request) {
            return handleResultMessagesNotificationException(ex, new HttpHeaders(),
                    HttpStatus.NOT_FOUND, request);
        }
    
        @ExceptionHandler(BusinessException.class)
        public ResponseEntity<Object> handleBusinessException(BusinessException ex,
                WebRequest request) {
            return handleResultMessagesNotificationException(ex, new HttpHeaders(),
                    HttpStatus.CONFLICT, request);
        }
    
        private ResponseEntity<Object> handleResultMessagesNotificationException(
                ResultMessagesNotificationException ex, HttpHeaders headers,
                HttpStatus status, WebRequest request) {
            String errorCode = exceptionCodeResolver.resolveExceptionCode(ex);
            ApiError apiError = apiErrorCreator.createResultMessagesApiError(
                    request, errorCode, ex.getResultMessages(), ex.getMessage());
            return handleExceptionInternal(ex, apiError, headers, status, request);
        }
    
        @ExceptionHandler({ OptimisticLockingFailureException.class,
                PessimisticLockingFailureException.class })
        public ResponseEntity<Object> handleLockingFailureException(Exception ex,
                WebRequest request) {
            return handleExceptionInternal(ex, null, new HttpHeaders(),
                    HttpStatus.CONFLICT, request);
        }
    
        @ExceptionHandler(Exception.class)
        public ResponseEntity<Object> handleSystemError(Exception ex,
                WebRequest request) {
            return handleExceptionInternal(ex, null, new HttpHeaders(),
                    HttpStatus.INTERNAL_SERVER_ERROR, request);
        }
    
    }

|

.. _RESTAppendixSoruceCodesOfDomainLayer:

Source code of the domain layer class created at the time of REST API implementation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Source code of domain layer class called from REST API explained in \ :ref:`RESTHowToUse`\  is attached herewith.
| Also, infrastructure layer is implemented by using JPA (Spring Data JPA).

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 35 55

   * - | Sr. No.
     - | Classification
     - | File name
   * - | (1)
     - | model
     - | :ref:`Member.java <RESTAppendixSoruceCodesOfMember>`
   * - | (2)
     - | 
     - | :ref:`MemberCredentia.java <RESTAppendixSoruceCodesOfMemberCredentia>`
   * - | (3)
     - | 
     - | :ref:`Gender.java <RESTAppendixSoruceCodesOfGender>`
   * - | (4)
     - | repository
     - | :ref:`MemberRepository.java <RESTAppendixSoruceCodesOfMemberRepository>`
   * - | (5)
     - | service
     - | :ref:`MemberService.java <RESTAppendixSoruceCodesOfMemberService>`
   * - | (6)
     - | 
     - | :ref:`MemberServiceImpl.java <RESTAppendixSoruceCodesOfMemberServiceImpl>`
   * - | (7)
     - | other
     - | :ref:`DomainMessageCodes.java <RESTAppendixSoruceCodesOfDomainMessageCodes>`
   * - | (8)
     - | 
     - | :ref:`member-mapping.xml <RESTAppendixSoruceCodesOfMemberMappingXml>`




 Following files are excluded.

 * JavaBean other than Entity
 * Configuration files other than Dozer

|

.. _RESTAppendixSoruceCodesOfMember:

Member.java
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

:file:`java/org/terasoluna/examples/rest/domain/model/Member.java`

.. code-block:: java

    package org.terasoluna.examples.rest.domain.model;
    
    import java.io.Serializable;
    
    import javax.persistence.Access;
    import javax.persistence.AccessType;
    import javax.persistence.CascadeType;
    import javax.persistence.Column;
    import javax.persistence.Entity;
    import javax.persistence.Id;
    import javax.persistence.JoinColumn;
    import javax.persistence.OneToOne;
    import javax.persistence.Table;
    import javax.persistence.Transient;
    import javax.persistence.Version;
    
    import org.joda.time.DateTime;
    import org.joda.time.LocalDate;
    import org.springframework.data.annotation.CreatedDate;
    import org.springframework.data.annotation.LastModifiedDate;
    
    @Table(name = "t_member")
    @Entity
    public class Member implements Serializable {
    
        private static final long serialVersionUID = 1L;
    
        @Id
        private String memberId;
    
        private String firstName;
    
        private String lastName;
    
        @Transient
        private Gender gender;
    
        private LocalDate dateOfBirth;
    
        private String emailAddress;
    
        private String telephoneNumber;
    
        private String zipCode;
    
        private String address;
    
        @CreatedDate
        private DateTime createdAt;
    
        @LastModifiedDate
        private DateTime lastModifiedAt;
    
        @Version
        private long version;
    
        @OneToOne(cascade = CascadeType.ALL)
        @JoinColumn(name = "member_id")
        private MemberCredential credential;
    
        public String getMemberId() {
            return memberId;
        }
    
        public void setMemberId(String memberId) {
            this.memberId = memberId;
        }
    
        public String getFirstName() {
            return firstName;
        }
    
        public void setFirstName(String firstName) {
            this.firstName = firstName;
        }
    
        public String getLastName() {
            return lastName;
        }
    
        public void setLastName(String lastName) {
            this.lastName = lastName;
        }
    
        public Gender getGender() {
            return gender;
        }
    
        public void setGender(Gender gender) {
            this.gender = gender;
        }
    
        @Access(AccessType.PROPERTY)
        @Column(name = "gender")
        public String getGenderCode() {
            if (gender == null) {
                return null;
            } else {
                return gender.getCode();
            }
        }
    
        public void setGenderCode(String genderCode) {
            this.gender = Gender.getByCode(genderCode);
        }
    
        public LocalDate getDateOfBirth() {
            return dateOfBirth;
        }
    
        public void setDateOfBirth(LocalDate dateOfBirth) {
            this.dateOfBirth = dateOfBirth;
        }
    
        public String getEmailAddress() {
            return emailAddress;
        }
    
        public void setEmailAddress(String emailAddress) {
            this.emailAddress = emailAddress;
        }
    
        public String getTelephoneNumber() {
            return telephoneNumber;
        }
    
        public void setTelephoneNumber(String telephoneNumber) {
            this.telephoneNumber = telephoneNumber;
        }
    
        public String getZipCode() {
            return zipCode;
        }
    
        public void setZipCode(String zipCode) {
            this.zipCode = zipCode;
        }
    
        public String getAddress() {
            return address;
        }
    
        public void setAddress(String address) {
            this.address = address;
        }
    
        public DateTime getCreatedAt() {
            return createdAt;
        }
    
        public void setCreatedAt(DateTime createdAt) {
            this.createdAt = createdAt;
        }
    
        public DateTime getLastModifiedAt() {
            return lastModifiedAt;
        }
    
        public void setLastModifiedAt(DateTime lastModifiedAt) {
            this.lastModifiedAt = lastModifiedAt;
        }
    
        public long getVersion() {
            return version;
        }
    
        public void setVersion(long version) {
            this.version = version;
        }
    
        public MemberCredential getCredential() {
            return credential;
        }
    
        public void setCredential(MemberCredential credential) {
            this.credential = credential;
        }
    
    }


|

.. _RESTAppendixSoruceCodesOfMemberCredentia:

MemberCredentia.java
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

:file:`java/org/terasoluna/examples/rest/domain/model/MemberCredential.java`

.. code-block:: java

    package org.terasoluna.examples.rest.domain.model;
    
    import java.io.Serializable;
    
    import javax.persistence.Entity;
    import javax.persistence.Id;
    import javax.persistence.Table;
    import javax.persistence.Version;
    
    import org.joda.time.DateTime;
    import org.springframework.data.annotation.LastModifiedDate;
    
    @Table(name = "t_member_credential")
    @Entity
    public class MemberCredential implements Serializable {
    
        private static final long serialVersionUID = 1L;
    
        @Id
        private String memberId;
    
        private String signId;
    
        private String password;
    
        private String previousPassword;
    
        private DateTime passwordLastChangedAt;
    
        @LastModifiedDate
        private DateTime lastModifiedAt;
    
        @Version
        private long version;
    
        public String getMemberId() {
            return memberId;
        }
    
        public void setMemberId(String memberId) {
            this.memberId = memberId;
        }
    
        public String getSignId() {
            return signId;
        }
    
        public void setSignId(String signId) {
            this.signId = signId;
        }
    
        public String getPassword() {
            return password;
        }
    
        public void setPassword(String password) {
            this.password = password;
        }
    
        public String getPreviousPassword() {
            return previousPassword;
        }
    
        public void setPreviousPassword(String previousPassword) {
            this.previousPassword = previousPassword;
        }
    
        public DateTime getPasswordLastChangedAt() {
            return passwordLastChangedAt;
        }
    
        public void setPasswordLastChangedAt(DateTime passwordLastChangedAt) {
            this.passwordLastChangedAt = passwordLastChangedAt;
        }
    
        public DateTime getLastModifiedAt() {
            return lastModifiedAt;
        }
    
        public void setLastModifiedAt(DateTime lastModifiedAt) {
            this.lastModifiedAt = lastModifiedAt;
        }
    
        public long getVersion() {
            return version;
        }
    
        public void setVersion(long version) {
            this.version = version;
        }
    
    }


|

.. _RESTAppendixSoruceCodesOfGender:

Gender.java
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

:file:`java/org/terasoluna/examples/rest/domain/model/Gender.java`

.. code-block:: java

    package org.terasoluna.examples.rest.domain.model;
    
    import java.util.Collections;
    import java.util.HashMap;
    import java.util.Map;
    
    import org.springframework.util.Assert;
    
    public enum Gender {
    
        UNKNOWN("0"), MEN("1"), WOMEN("2");
    
        private static final Map<String, Gender> genderMap;
    
        static {
            Map<String, Gender> map = new HashMap<>();
            for (Gender gender : values()) {
                map.put(gender.code, gender);
            }
            genderMap = Collections.unmodifiableMap(map);
        }
    
        private final String code;
    
        private Gender(String code) {
            this.code = code;
        }
    
        public static Gender getByCode(String code) {
            Gender gender = genderMap.get(code);
            Assert.notNull(gender, "gender code is invalid. code : " + code);
            return gender;
        }
    
        public String getCode() {
            return code;
        }
    
    }
    

|

.. _RESTAppendixSoruceCodesOfMemberRepository:

MemberRepository.java
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

:file:`java/org/terasoluna/examples/rest/domain/repository/member/MemberRepository.java`

.. code-block:: java

    package org.terasoluna.examples.rest.domain.repository.member;
    
    import org.springframework.data.domain.Page;
    import org.springframework.data.domain.Pageable;
    import org.springframework.data.jpa.repository.JpaRepository;
    import org.springframework.data.jpa.repository.Query;
    import org.springframework.data.repository.query.Param;
    import org.terasoluna.examples.rest.domain.model.Member;
    
    public interface MemberRepository extends JpaRepository<Member, String> {
    
        @Query("SELECT m FROM Member m"
                + " WHERE m.firstName LIKE :name% ESCAPE '~'"
                + " OR m.lastName LIKE :name% ESCAPE '~'")
        Page<Member> findPageByContainsName(@Param("name") String name,
                Pageable pageable);
    
    }


|

.. _RESTAppendixSoruceCodesOfMemberService:

MemberService.java
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

:file:`java/org/terasoluna/examples/rest/domain/service/member/MemberService.java`

.. code-block:: java

    package org.terasoluna.examples.rest.domain.service.member;
    
    import org.springframework.data.domain.Page;
    import org.springframework.data.domain.Pageable;
    import org.terasoluna.examples.rest.domain.model.Member;
    
    public interface MemberService {
    
        Page<Member> searchMembers(String name, Pageable pageable);
    
        Member getMember(String memberId);
    
        Member createMember(Member creatingMember);
    
        Member updateMember(String memberId, Member updatingMember);
    
        void deleteMember(String memberId);
    
    }

|

.. _RESTAppendixSoruceCodesOfMemberServiceImpl:

MemberServiceImpl.java
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

:file:`java/org/terasoluna/examples/rest/domain/service/member/MemberServiceImpl.java`

.. code-block:: java

    package org.terasoluna.examples.rest.domain.service.member;
    
    import javax.inject.Inject;
    import javax.inject.Named;
    
    import org.dozer.Mapper;
    import org.joda.time.DateTime;
    import org.springframework.dao.DataIntegrityViolationException;
    import org.springframework.data.domain.Page;
    import org.springframework.data.domain.Pageable;
    import org.springframework.security.crypto.password.PasswordEncoder;
    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;
    import org.springframework.util.StringUtils;
    import org.terasoluna.examples.rest.domain.message.DomainMessageCodes;
    import org.terasoluna.examples.rest.domain.model.Member;
    import org.terasoluna.examples.rest.domain.model.MemberCredential;
    import org.terasoluna.examples.rest.domain.repository.member.MemberRepository;
    import org.terasoluna.gfw.common.date.jodatime.JodaTimeDateFactory;
    import org.terasoluna.gfw.common.exception.BusinessException;
    import org.terasoluna.gfw.common.exception.ResourceNotFoundException;
    import org.terasoluna.gfw.common.message.ResultMessages;
    import org.terasoluna.gfw.common.query.QueryEscapeUtils;
    import org.terasoluna.gfw.common.sequencer.Sequencer;
    
    @Transactional
    @Service
    public class MemberServiceImpl implements MemberService {
    
        @Inject
        MemberRepository memberRepository;
    
        @Inject
        @Named("memberIdSequencer")
        Sequencer<String> sequencer;
    
        @Inject
        JodaTimeDateFactory dateFactory;
    
        @Inject
        PasswordEncoder passwordEncoder;
    
        @Inject
        Mapper beanMapper;
    
        @Transactional(readOnly = true)
        public Page<Member> searchMembers(String name, Pageable pageable) {
    
            // escape to like condition value
            String escapedName = QueryEscapeUtils.toLikeCondition(name);
    
            // find members that matches with search criteria
            return memberRepository.findPageByContainsName(escapedName, pageable);
        }
    
        @Transactional(readOnly = true)
        public Member getMember(String memberId) {
            // find member
            Member member = memberRepository.findOne(memberId);
            if (member == null) {
                // If member is not exists
                throw new ResourceNotFoundException(ResultMessages.error().add(
                        DomainMessageCodes.E_EX_MM_5001, memberId));
            }
            return member;
        }
    
        public Member createMember(Member creatingMember) {
    
            MemberCredential creatingCredential = creatingMember.getCredential();
    
            // get processing current date time
            DateTime currentDateTime = dateFactory.newDateTime();
    
            // set id
            String newMemberId = sequencer.getNext();
            creatingMember.setMemberId(newMemberId);
            creatingCredential.setMemberId(newMemberId);
    
            // decide sign id(email-address)
            String signId = creatingCredential.getSignId();
            if (!StringUtils.hasLength(signId)) {
                signId = creatingMember.getEmailAddress();
                creatingCredential.setSignId(signId.toLowerCase());
            }
    
            // encrypt password
            String rawPassword = creatingCredential.getPassword();
            creatingCredential.setPassword(passwordEncoder.encode(rawPassword));
            creatingCredential.setPasswordLastChangedAt(currentDateTime);
    
            // save member & member credential
            try {
                return memberRepository.saveAndFlush(creatingMember);
            } catch (DataIntegrityViolationException e) {
                // If sign id is already used
                throw new BusinessException(ResultMessages.error().add(
                        DomainMessageCodes.E_EX_MM_8001,
                        creatingCredential.getSignId()), e);
            }
        }
    
        public Member updateMember(String memberId, Member updatingMember) {
            // get member
            Member member = getMember(memberId);
    
            // override updating member attributes
            beanMapper.map(updatingMember, member, "member.update");
    
            // save updating member
            return memberRepository.save(member);
        }
    
        public void deleteMember(String memberId) {
    
            // delete member
            memberRepository.delete(memberId);
    
        }
    
    }

|

.. _RESTAppendixSoruceCodesOfDomainMessageCodes:

DomainMessageCodes.java
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

:file:`java/org/terasoluna/examples/rest/domain/message/DomainMessageCodes.java`

.. code-block:: java

    package org.terasoluna.examples.rest.domain.message;
    
    /**
     * Message codes of domain layer message.
     * @author DomainMessageCodesGenerator
     */
    public class DomainMessageCodes {
    
        private DomainMessageCodes() {
            // NOP
        }
    
        /** e.ex.mm.5001=Specified member not found. member id : {0} */
        public static final String E_EX_MM_5001 = "e.ex.mm.5001";
    
        /** e.ex.mm.8001=Cannot use specified sign id. sign id : {0} */
        public static final String E_EX_MM_8001 = "e.ex.mm.8001";
    }

|

.. _RESTAppendixSoruceCodesOfMemberMappingXml:

member-mapping.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| In the implemented Service class, "\ :doc:`Utilities/Dozer`\" is used while copying the value specified by client in \ ``Member``\  object.
| When it is alright to simply copy the field values, Bean mapping definition need not be added. However, in this implementation it needs to be ensured that, items which are not to be updated (\ ``memberId``\ , \ ``credential``\ , \ ``createdAt``\  and \ ``version``\ ) are not copied.
| Bean mapping definition needs to be added in order to ensure that specific fields are not copied.

:file:`resources/META-INF/dozer/member-mapping.xml`

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <mappings xmlns="http://dozer.sourceforge.net" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://dozer.sourceforge.net
              http://dozer.sourceforge.net/schema/beanmapping.xsd">
    
        <mapping map-id="member.update">
            <class-a>org.terasoluna.examples.rest.domain.model.Member</class-a>
            <class-b>org.terasoluna.examples.rest.domain.model.Member</class-b>
            <field-exclude>
                <a>memberId</a>
                <b>memberId</b>
            </field-exclude>
            <field-exclude>
                <a>credential</a>
                <b>credential</b>
            </field-exclude>
            <field-exclude>
                <a>createdAt</a>
                <b>createdAt</b>
            </field-exclude>
            <field-exclude>
                <a>lastModifiedAt</a>
                <b>lastModifiedAt</b>
            </field-exclude>
            <field-exclude>
                <a>version</a>
                <b>version</b>
            </field-exclude>
        </mapping>
    
    </mappings>

.. raw:: latex

   \newpage

