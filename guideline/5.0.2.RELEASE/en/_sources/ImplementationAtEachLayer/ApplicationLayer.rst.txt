Implementation of Application Layer
================================================================================

.. only:: html

 .. contents:: Index
    :depth: 3
    :local:

This chapter explains implementation of application layer of a web application that uses HTML forms. 

.. note::

   Refer to the following page for the implementation required for Ajax development and REST API.

   - :doc:`../ArchitectureInDetail/Ajax`

|

Implementation of application layer is given in the following 3 steps.

#. | :ref:`controller-label`
   | Controller receives the request, calls business logic, updates model, decides View. Thereby, controls one complete round of operations after receiving the request. 
   | It is the most important part in the implementation of application layer.
#. | :ref:`formobject`
   | Form object transfers the values between HTML form and application.
#. | :ref:`view`
   | View (JSP) acquires the data from model (form object, domain object etc.) and generates screen (HTML).


.. _controller-label:

Implementing Controller
--------------------------------------------------------------------------------
| Implementation of Controller is explained below.
| The Controller performs the following roles.

#. | **Provides a method to receive the request.**
   | Receives requests by implementing methods to which \ ``@RequestMapping``\  annotation is assigned.
#. | **Performs input validation of request parameter.**
   | For request which requires input validation of request parameters, it can be  performed by specifying \ ``@Validated``\  annotation to form object argument of method which receives the request.
   | Perform single item check using Bean Validation and correlation check using Spring Validator or Bean Validation.
#. | **Calls business logic.**
   | Controller does not implement business logic but delegates by calling Service method.
#. | **Reflects the output of business logic to Model.**
   | The output can be referred in View by reflecting domain object returned from Service method to \ ``Model``\ .
#. | **Returns View name corresponding to business logic output.**
   | Controller does not implement the logic of rendering the view. Rendering is done in View technologies like JSP etc.
   | Controller only returns the name of the view in which the rendering logic is implemented.
   | The View corresponding to view name is resolved by \ ``ViewResolver``\  provided by Spring Framework.

.. figure:: images_ApplicationLayer/application_logic-of-controller.png
   :alt: responsibility of logic
   :width: 80%
   :align: center
 
   **Picture - Logic of controller**

.. note::
 
 **It is recommended that controller implements only the routing logic** such as calling business logic, reflecting output of the business logic to \ ``Model``\ , deciding the View name is implemented in the Controller.

|

The implementation of Controller is explained by focusing on the following points.

- :ref:`controller-new-label`
- :ref:`controller_mapping-label`
- :ref:`controller_method_argument-label`
- :ref:`controller_method_return-label`

|

.. _controller-new-label:

Creating Controller class
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| **Controller class is created with @Controller annotation added to POJO class (Annotation-based Controller).**
| Controller in Spring MVC can also be created by implementing \ ``org.springframework.web.servlet.mvc.Controller``\  interface (Interface-based Controller). However, it is preferred to avoid using it as it is Deprecated from Spring 3 onwards.

 .. code-block:: java

    @Controller
    public class SampleController {
        // ...
    }

|
|

.. _controller_mapping-label:

Mapping request and processing method
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| \ ``@RequestMapping``\  annotation is assigned to the method that receives request.
| In this document, the method to which \ ``@RequestMapping``\  is added, is called as "processing method".

 .. code-block:: java

    @RequestMapping(value = "hello")
    public String hello() {
        // ...
    }

|

The rules for mapping the incoming request with a processing method can be specifying as attributes of \ ``@RequestMapping``\  annotation. 

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 10 80

   * - Sr. No.
     - Attribute name
     - Description
   * - 1.
     - value
     - | Specify request path which needs to be mapped (multiple values allowed).
   * - 2.
     - method
     - | Specify HTTP method (\ ``RequestMethod``\  type) which needs to be mapped (multiple methods allowed).
       | GET/POST are mainly used for mapping requests from HTML form, while other HTTP methods (such as PUT/DELETE) are used for mapping requests from REST APIs as well. 
   * - 3.
     - params
     - | Specify request parameters which need to be mapped (multiple parameters allowed).
       | Request parameters are mainly used for mapping request from HTML form. If this mapping method is used, the case of mapping multiple buttons on HTML page can be implemented easily.
   * - 4.
     - headers
     - | Specify request headers which need to be mapped (multiple headers allowed).
       | Mainly used while mapping REST API and Ajax requests.
   * - 5.
     - consumes
     - | Mapping can be performed using Content-Type header of request. Specify media type which needs to be mapped (multiple types allowed).
       | Mainly used while mapping REST API and Ajax requests.
   * - 6.
     - produces
     - | Mapping can be performed using Accept header of request. Specify media type which needs to be mapped (multiple types allowed).
       | Mainly used while mapping REST API and Ajax requests.

 .. note:: **Combination of mapping**

    Complex mapping can be performed by combining multiple attributes, but considering maintainability, mapping should be defined and designed in the simplest way possible .
    It is recommended to consider combining 2 attributes (value attribute and any other 1 attribute).

|

| 6 examples of mapping are shown below.

- :ref:`controller-mapping-path-label`
- :ref:`controller-mapping-method-label`
- :ref:`controller-mapping-params-label`
- :ref:`controller-mapping-headers-label`
- :ref:`controller-mapping-contenttype-label`
- :ref:`controller-mapping-accept-label`

| In the following explanation, it is prerequisite to define the processing method in the Controller class.

 .. code-block:: java
    :emphasize-lines: 1-2

    @Controller // (1)
    @RequestMapping("sample") // (2)
    public class SampleController {
        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - With \ ``@Controller``\ , it is recognized as Annotation-based controller class and becomes the target of component scan.
   * - | (2)
     - All the processing methods in this class are mapped to URLs with "sample" by adding ``@RequestMapping("sample")``\  annotation at class level. 

|

.. _controller-mapping-path-label:

Mapping with request path
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
In case of the following definition, if the URL ``"sample/hello"`` is accessed, then ``hello()`` method is executed.

 .. code-block:: java

    @RequestMapping(value = "hello")
    public String hello() {

| When multiple values are specified, it is handled by 'OR' condition.
| In case of following definition, if ``"sample/hello"`` or ``"sample/bonjour"`` is accessed, then ``hello()`` method is executed. 

 .. code-block:: java

    @RequestMapping(value = {"hello", "bonjour"})
    public String hello() {

Pattern can be specified instead of a specific value for request path. For details of specifying patterns, refer to reference documentation of Spring Framework.

- `URI Template Patterns <http://docs.spring.io/spring/docs/4.1.9.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-requestmapping-uri-templates>`_\
- `URI Template Patterns with Regular Expressions <http://docs.spring.io/spring/docs/4.1.9.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-requestmapping-uri-templates-regex>`_\
- `Path Patterns <http://docs.spring.io/spring/docs/4.1.9.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-requestmapping-patterns>`_\
- `Patterns with Placeholders <http://docs.spring.io/spring/docs/4.1.9.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-requestmapping-placeholders>`_\

|

.. _controller-mapping-method-label:

Mapping by HTTP method
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
In case of the following definition, if the URL ``"sample/hello"`` is accessed with POST method, then  ``hello()`` method is executed.
For the list of supported HTTP methods, refer to `Javadoc of RequestMethod <http://docs.spring.io/spring/docs/4.1.9.RELEASE/javadoc-api/org/springframework/web/bind/annotation/RequestMethod.html>`_.
When not specified, all supported HTTP methods are mapped.

 .. code-block:: java

    @RequestMapping(value = "hello", method = RequestMethod.POST)
    public String hello() {


| When multiple values are specified, it is handled by 'OR' condition.
| In case of following definition, if ``"sample/hello"`` is accessed with GET or HEAD method, then ``hello()`` method is executed.

 .. code-block:: java

    @RequestMapping(value = "hello", method = {RequestMethod.GET, RequestMethod.HEAD})
    public String hello() {

|

.. _controller-mapping-params-label:

Mapping by request parameter
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| In case of following definition, if the URL ``sample/hello?form`` is accessed, then ``hello()`` method is executed. 
| When request is sent as a POST request, request parameters may exist in request body even if they do not exist in URL.

 .. code-block:: java

    @RequestMapping(value = "hello", params = "form")
    public String hello() {


| When multiple values are specified, it is handled by 'AND' condition.
| In case of following definition, if the URL ``"sample/hello?form&formType=foo"`` is accessed, then ``hello()`` method is executed. 

 .. code-block:: java

    @RequestMapping(value = "hello", params = {"form", "formType=foo"})
    public String hello(@RequestParam("formType") String formType) {

Supported formats are as follows.

 .. tabularcolumns:: |p{0.08\linewidth}|p{0.25\linewidth}|p{0.67\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 8 25 67

   * - Sr. No.
     - Format
     - Explanation
   * - 1.
     - paramName
     - Mapping is performed when request parameter of the specified parameName exists.
   * - 2.
     - !paramName
     - Mapping is performed when request parameter of the specified parameName does not exist.
   * - 3.
     - paramName=paramValue
     - Mapping is performed when value of the specified parameName is paramValue.
   * - 4.
     - paramName!=paramValue
     - Mapping is performed when value of the specified parameName is not paramValue.

|

.. _controller-mapping-headers-label:

Mapping using request header
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Refer to the details on the following page to mainly use the controller to map REST API and Ajax requests.

- :doc:`../ArchitectureInDetail/Ajax`


.. _controller-mapping-contenttype-label:

Mapping using Content-Type header
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Refer to the details on the following page to mainly use the controller to map REST API and Ajax requests.

- :doc:`../ArchitectureInDetail/Ajax`


.. _controller-mapping-accept-label:

Mapping using Accept header
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Refer to the details on the following page to mainly use the controller to map REST API and Ajax requests.

- :doc:`../ArchitectureInDetail/Ajax`

|
|

.. _controller-mapping-policy-label:

Mapping request and processing method
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Mapping by the following method is recommended.

- | **Grouping of URL of request is done for each unit of business flow or functional flow.**
  | URL grouping means defining \ ``@RequestMapping(value = "xxx")``\  as class level annotation.

- | **Use the same URL for requests for screen transitions within same functional flow**
  | The same URL means the value of 'value' attribute of \ ``@RequestMapping(value = "xxx")``\  must be same.
  | Determining which processing method is used for a particular request with same functional flow is performed using HTTP method and HTTP parameters.

The following is an example of mapping between incoming request and processing method by a sample application with basic screen flow.

 * :ref:`controller-mapping-policy-sampleapp-overview-label`
 * :ref:`controller-mapping-policy-sampleapp-url-design-label`
 * :ref:`controller-mapping-policy-sampleapp-mapping-design-label`
 * :ref:`controller-mapping-policy-sampleapp-form-impl-label`
 * :ref:`controller-mapping-policy-sampleapp-confirm-impl-label`
 * :ref:`controller-mapping-policy-sampleapp-redo-impl-label`
 * :ref:`controller-mapping-policy-sampleapp-create-impl-label`

|

.. _controller-mapping-policy-sampleapp-overview-label:

Overview of sample application
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Functional overview of sample application is as follows.

- | Provides functionality of performing CRUD operations of Entity.
- | Following 5 operations are provided.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table:: 
    :header-rows: 1
    :widths: 10 20 70

    * - Sr. No.
      - Operation name
      - Overview
    * - 1.
      - Fetching list of Entities
      - Fetch list of all the created Entities to be displayed on the list screen.
    * - 2.
      - Create Entity
      - Create a new Entity with the specified contents. Screen flow (form screen, confirmation screen, completion screen) exists for this process.
    * - 3.
      - Fetching details of Entity
      - Fetch Entity of specified ID to be displayed on the details screen.
    * - 4.
      - Entity update
      - Update Entity of specified ID. Screen flow (form screen, confirmation screen, completion screen) exists for this process.
    * - 5.
      - Entity delete
      - Delete Entity of specified ID.

- | Screen flow of all functions is as follows.
  | It is not mentioned in screen flow diagram however, when input validation error occurs, form screen is displayed again.
  
.. figure:: images_ApplicationLayer/application_sample-screen-flow.png
   :alt: Screen flow of entity management function
   :width: 90%
   :align: center
 
   **Picture - Screen flow of entity management function**

|

.. _controller-mapping-policy-sampleapp-url-design-label:

Request URL
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Design the URL of the required requests.

- | Request URLs of all the requests required by the process flow are grouped.
  | This functionality performs CRUD operations of Entity called ABC, therefore URL that starts with ``"/abc/"`` is considered. 
  
- Design request URL for each operation of the functionality.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
 .. list-table:: 
    :header-rows: 1
    :widths: 10 30 60

    * - Sr. No.
      - Operation name
      - URL for each operation (pattern)
    * - 1.
      - Fetching list of Entities
      - /abc/list
    * - 2.
      - Create Entity
      - /abc/create
    * - 3.
      - Fetching details of Entity
      - /abc/{id}
    * - 4.
      - Entity update
      - /abc/{id}/update
    * - 5.
      - Entity delete
      - /abc/{id}/delete

 .. note::
 
     ``"{id}"`` specified in URL of 'Fetching details of Entity', 'Entity update', 'Entity delete' operations is called as, `URI Template Pattern <http://docs.spring.io/spring/docs/4.1.9.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-requestmapping-uri-templates>`_\  and any value can be specified.
     In this sample application, Entity ID is specified.

 Assigned URL of each operation of screen flow diagram is mapped as shown below:

.. figure:: images_ApplicationLayer/application_sample-screen-flow-assigned-url.png
   :alt: Screen flow of entity management function and corresponding assigned URL
   :width: 90%
   :align: center
 
   **Picture - Screen flow of entity management function and corresponding assigned URL**

|

.. _controller-mapping-policy-sampleapp-mapping-design-label:

Mapping request and processing method
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Design the mapping between incoming request and processing method.
| The following is the mapping design which is designed according to mapping policy.

 .. tabularcolumns:: |p{0.05\linewidth}|p{0.20\linewidth}|p{0.15\linewidth}|p{0.22\linewidth}|p{0.10\linewidth}|p{0.13\linewidth}|p{0.15\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 5 20 15 22 10 13 15

   * - | Sr. No.
     - | Operation name
     - | URL
     - | Request name
     - | HTTP
       | Method
     - | HTTP
       | Parameter
     - | Processing method
   * - 1.
     - Fetching list of Entities
     - /abc/list
     - List display
     - GET
     - \-
     - list
   * - 2.
     - Create New Entity
     - /abc/create
     - Form display
     - \-
     - form
     - createForm
   * - 3.
     - 
     - 
     - Displaying input confirmation 
     - POST
     - confirm
     - createConfirm
   * - 4.
     - 
     - 
     - Form re-display
     - POST
     - redo
     - createRedo
   * - 5.
     - 
     - 
     - Entity Creation
     - POST
     - \-
     - create
   * - 6.
     - 
     - 
     - Displaying completion of Entity Creation
     - GET
     - complete
     - createComplete
   * - 7.
     - Fetching details of Entity 
     - /abc/{id}
     - Display details of Entity 
     - GET
     - \-
     - read
   * - 8.
     - Entity update
     - /abc/{id}/update
     - Displaying Form 
     - \-
     - form
     - updateForm
   * - 9.
     - 
     - 
     - Displaying confirmation of user input
     - POST
     - confirm
     - updateConfirm
   * - 10.
     - 
     - 
     - Form re-display
     - POST
     - redo
     - updateRedo
   * - 11.
     - 
     - 
     - Update
     - POST
     - \-
     - update
   * - 12.
     - 
     - 
     - Displaying completion of update process
     - GET
     - complete
     - updateComplete
   * - 13.
     - Entity delete
     - /abc/{id}/delete
     - Delete
     - POST
     - \-
     - delete
   * - 14.
     - 
     - 
     - Displaying completion of delete process
     - GET
     - complete
     - deleteComplete

| Multiple requests exist for each of Create Entity, Entity Update and Entity Delete functions. Therefore switching of processing methods is done using HTTP method and HTTP parameters.
| The following is the flow of requests in case of multiple requests in a function like "Create New Entity".
| All URLs are ``"/abc/create"`` and determining the processing method is done based on combination of HTTP method and HTTP parameters.

.. figure:: images_ApplicationLayer/applicationScreenflow.png
   :alt: Request flow of entity create processing
   :width: 90%
   :align: center
 
   **Picture - Request flow of entity create processing**

|

| Implementation of processing method for "Create New Entity" is shown below.
| Here, the purpose is to understand mapping between request and processing method and therefore focus must on \ ``@RequestMapping``\ .
| The details of argument and return value (view name and view) of processing method are explained in the next chapter. 

- :ref:`controller-mapping-policy-sampleapp-form-impl-label`
- :ref:`controller-mapping-policy-sampleapp-confirm-impl-label`
- :ref:`controller-mapping-policy-sampleapp-redo-impl-label`
- :ref:`controller-mapping-policy-sampleapp-create-impl-label`
- :ref:`controller-mapping-policy-sampleapp-complete-impl-label`
- :ref:`controller-mapping-policy-sampleapp-multi-impl-label`

|

.. _controller-mapping-policy-sampleapp-form-impl-label:

Implementing form display
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
In order to display the form, ``form`` is specified as HTTP parameter.

 .. code-block:: java
    :emphasize-lines: 1,4

    @RequestMapping(value = "create", params = "form") // (1)
    public String createForm(AbcForm form, Model model) {
        // omitted
        return "abc/createForm"; // (2)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - Specify ``"form"`` as value of ``params`` attribute.
   * - | (2)
     - Return view name of JSP to render form screen.

 .. note::
    In this processing method, ``method`` attribute is not specified since it is not required for HTTP GET method.

|

Example of implementation of sections other than processing method is explained below.

Besides implementing the processing method for form display, points mentioned below are required:

- Implement generation process of form object. Refer to :ref:`formobject` for the details of form object.
- Implement View of form screen. Refer to :ref:`view` for the details of View.

Use the following form object.

 .. code-block:: java

  public class AbcForm implements Serializable {
      private static final long serialVersionUID = 1L;

      @NotEmpty
      private String input1;

      @NotNull
      @Min(1)
      @Max(10)
      private Integer input2;

      // omitted setter&getter
  }

Creating an object of AbcForm.

 .. code-block:: java

    @ModelAttribute
    public AbcForm setUpAbcForm() {
        return new AbcForm();
    }


Create view(JSP) of form screen.

 .. code-block:: jsp
    :emphasize-lines: 12

    <h1>Abc Create Form</h1>
    <form:form modelAttribute="abcForm"
      action="${pageContext.request.contextPath}/abc/create">
      <form:label path="input1">Input1</form:label>
      <form:input path="input1" />
      <form:errors path="input1" />
      <br>
      <form:label path="input2">Input2</form:label>
      <form:input path="input2" />
      <form:errors path="input2" />
      <br>
      <input type="submit" name="confirm" value="Confirm" /> <!-- (1) -->
    </form:form>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - Specify  \ ``name="confirm"``\  parameter for submit button to transit to confirmation screen.

|

The operations are explained below.

| Sending the request for form display.
| Access ``"abc/create?form"`` URL.
| Since ``form`` is specified in the URL as an HTTP parameter, ``createForm`` method of controller is called and form screen is displayed.

 .. figure:: images_ApplicationLayer/applicationCreateFormDisplay.png
   :width: 90%

|

.. _controller-mapping-policy-sampleapp-confirm-impl-label:

Implementing the display of user input confirmation screen
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
To check user input in the form, data is sent by POST method and ``confirm`` is specified as HTTP parameter.

 .. code-block:: java
    :emphasize-lines: 1,5,8

    @RequestMapping(value = "create", method = RequestMethod.POST, params = "confirm") // (1)
    public String createConfirm(@Validated AbcForm form, BindingResult result,
            Model model) {
        if (result.hasErrors()) {
            return createRedo(form, model); // return "abc/createForm"; (2)
        }
        // omitted
        return "abc/createConfirm"; // (3)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - Specify "RequestMethod.POST" in ``method`` attribute and "confirm" in ``params`` attribute.
   * - | (2)
     - In case of input validation errors, it is recommended to call the processing method of form re-display.
   * - | (3)
     - Return view-name of JSP to render the screen for user input confirmation.

 .. note::
    POST method is specified to prevent displaying confidential information such as password and other personal information etc. in the address bar.
    (Needless to say that these security measures not sufficient and needs more secure measures such as SSL etc.)

|

Example of implementation of sections other than processing method is explained below.

Besides implementing processing method for user input confirmation screen, points mentioned below are required.

- Implement view of user-input confirmation screen. Refer to :ref:`view` for the details of view.

Create  the view (JSP) for user input confirmation screen.

 .. code-block:: jsp
    :emphasize-lines: 6,10,12-13

    <h1>Abc Create Form</h1>
    <form:form modelAttribute="abcForm"
      action="${pageContext.request.contextPath}/abc/create">
      <form:label path="input1">Input1</form:label>
      ${f:h(abcForm.input1)}
      <form:hidden path="input1" /> <!-- (1) -->
      <br>
      <form:label path="input2">Input2</form:label>
      ${f:h(abcForm.input2)}
      <form:hidden path="input2" /> <!-- (1) -->
      <br>
      <input type="submit" name="redo" value="Back" /> <!-- (2) -->
      <input type="submit" value="Create" /> <!-- (3) -->
    </form:form>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - The values entered on form screen is set as the hidden fields of HTML form since they must be sent back to the server when Create or Back buttons are clicked.
   * - | (2)
     - Specify \ ``name="redo"``\  parameter for submit button to return to form screen.
   * - | (3)
     - Parameter name need not be specified for submit button. Submit button will do the actual create operation.

 .. note::
    In the above example, HTML escaping is performed as an XSS countermeasure using ``f:h()`` function while displaying the user input values.
    For details, refer to :doc:`Cross Site Scripting <../Security/XSS>`.

|

The operations are explained below.

| Send the request for displaying user input confirmation.
| Enter ``"aa"`` in Input1 and ``"5"`` in Input2 and click Confirm button on form screen.
| After clicking Confirm button, ``"abc/create?confirm"`` URI gets accessed using POST method.
| Since HTTP parameter ``confirm`` is present in the URI, ``createConfirm`` method of controller is called and user input confirmation screen is displayed.

 .. figure:: images_ApplicationLayer/applicationCreateConfirmDisplay.png
   :width: 90%

Since HTTP parameters are sent across through HTTP POST method after clicking the Confirm button, it does not appear in URI. However, "confirm" is included as HTTP parameter.

 .. figure:: images_ApplicationLayer/applicationCreateConfirmNetwork.png
   :width: 90%

|

.. _controller-mapping-policy-sampleapp-redo-impl-label:

Implementing 'redisplay of form' 
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
"redo" is specified as HTTP parameter to indicate that form needs to be redisplayed.

 .. code-block:: java
    :emphasize-lines: 1,4

    @RequestMapping(value = "create", method = RequestMethod.POST, params = "redo") // (1)
    public String createRedo(AbcForm form, Model model) {
        // omitted
        return "abc/createForm"; // (2)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - Specify "RequestMethod.POST" in ``method`` attribute and "redo" in ``params`` attribute.
   * - | (2)
     - Return view name of JSP to render the form screen.

|

Operation is described below.

| Send the request to redisplay the form screen.
| Click Back button on user input confirmation screen.
| When Back button is clicked, "abc/create?redo" URI gets accessed through HTTP POST method.
| Since "redo" HTTP parameter is present in the URI, ``createRedo`` method of controller is invoked and form screen is redisplayed.

 .. figure:: images_ApplicationLayer/applicationCreateConfirmDisplay.png
   :width: 90%

Since HTTP parameters are sent across through HTTP POST method after clicking the Back button, it does not appear in URI. However, "redo" is included as HTTP parameter.
Moreover, since input values of form had been sent as hidden fields, input values can be restored on redisplayed form screen.

 .. figure:: images_ApplicationLayer/applicationBackToCreateFormDisplay.png
   :width: 90%

 .. figure:: images_ApplicationLayer/applicationBackToCreateFormNetwork.png
   :width: 90%

.. note::

    In order to implement back button functionality, setting ``onclick="javascript:history.back()"`` is also one of the ways.
    Both the methods differ in the following ways. Appropriate method must be selected as per requirement.

    * Behavior when "Back button on browser" is clicked.
    * Behavior when page having Back button is accessed and Back button is clicked.
    * History of browser

|

.. _controller-mapping-policy-sampleapp-create-impl-label:

Implementing 'create new user' business logic
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| To register input contents of form, the data (hidden parameters) to be registered is sent with HTTP POST method.
| Sorting is not carried out using HTTP parameters since new request will be the main request of this operation.
| Since the state of database changes in this process, it should not be executed multiple times due to double submission.
| Therefore, it is 'redirected' to the next screen (create complete screen) instead of directly displaying View (screen) after 
| completing this process. This pattern is called as POST-Redirect-GET(PRG) pattern. For the details of :abbr:`PRG (Post-Redirect-Get)` pattern 
| refer to :doc:`../ArchitectureInDetail/DoubleSubmitProtection` .

 .. code-block:: java
    :emphasize-lines: 1,7

    @RequestMapping(value = "create", method = RequestMethod.POST) // (1)
    public String create(@Validated AbcForm form, BindingResult result, Model model) {
        if (result.hasErrors()) {
            return createRedo(form, model); // return "abc/createForm";
        }
        // omitted
        return "redirect:/abc/create?complete"; // (2)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - Specify ``RequestMethod.POST`` in ``method`` attribute. Do not specify ``params`` attribute.
   * - | (2)
     - Return URL to the request needs to be redirected as view name in order to use :abbr:`PRG (Post-Redirect-Get)`  pattern.

 .. note:: 
    It can be redirected to "/xxx" by returning "redirect:/xxx" as view name.

.. warning::
    PRG pattern is used to avoid double submission when the browser gets reloaded by clicking F5 button. However, as a countermeasure for double submission,
    it is necessary to use TransactionTokenCheck functionality.
    For details of TransactionTokenCheck, refer to :doc:`../ArchitectureInDetail/DoubleSubmitProtection` .

|

Operation is described below.

| Click 'Create' button on input confirmation screen.
| After clicking 'Create' button,  ``"abc/create"`` URL is accessed through POST method.
| Since HTTP parameters are not sent for identifying a button, it is considered as main request of Entity create process and 'create' method of Controller is invoked.

| 'Create' request does not return to the screen directly, but it is redirected to create complete display (``"/abc/create?complete"``). Hence HTTP status is changed to 302.

 .. figure:: images_ApplicationLayer/applicationCreateNetwork.png
   :width: 90%


|

.. _controller-mapping-policy-sampleapp-complete-impl-label:

Implementing notification of create new user process completion
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
In order to notify the completion of create process, ``complete`` must be present in the request as HTTP parameter.

 .. code-block:: java
    :emphasize-lines: 1,4

    @RequestMapping(value = "create", params = "complete") // (1)
    public String createComplete() {
        // omitted
        return "abc/createComplete"; // (2)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - Specify ``"complete"`` in ``params`` attribute.
   * - | (2)
     - Return View name of JSP to render the create completion screen.

 .. note::
    In this processing method, ``method`` attribute is not specified since it is not required for HTTP GET method.

|

Operation is described below.

| After completing creation of user, access URI (``"/abc/create?complete"``) is specified as redirect destination.
| Since HTTP parameter is ``complete``, createComplete() method of controller is called and create completion screen is displayed.


 .. figure:: images_ApplicationLayer/applicationCreateCompleteDisplay.png
   :width: 90%

 .. figure:: images_ApplicationLayer/applicationCreateCompleteNetwork.png
   :width: 90%

 .. note::
    Since PRG pattern is used, even if browser is reloaded, create completion screen is only re-displayed without re-executing create process.

|

.. _controller-mapping-policy-sampleapp-multi-impl-label:

Placing multiple buttons on HTML form
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
To place multiple buttons on a single form, send HTTP parameter to identify the corresponding button and so that the processing method of controller can be switched.
An example of Create button and Back button on input confirmation screen of sample application is explained here.

'Create' button to perform 'user creation' and 'Back' button to redisplay 'create form' exists on the form of input confirmation screen as shown below.

.. figure:: images_ApplicationLayer/applicationControllerBackToForm.png
   :alt: Multiple button in the HTML form
   :width: 80%
   :align: center
 
   **Picture - Multiple button in the HTML form**

To redisplay 'create form' using request ( ``"/abc/create?redo"`` ) when Back button is clicked,
the following code is required in HTML form.

 .. code-block:: jsp
    :emphasize-lines: 1

    <input type="submit" name="redo" value="Back" /> <!-- (1) -->
    <input type="submit" value="Create" />

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - In input confirmation screen ( ``"abc/createConfirm.jsp"`` ), specify \ ``name="redo"``\  parameter for Back button.

For the operations when Back button is clicked, refer to :ref:`controller-mapping-policy-sampleapp-redo-impl-label`.

|

.. _controller-mapping-policy-sampleapp-all-impl-label:

Source code of controller of sample application
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Source-code of controller after implementing create process of sample application are shown below.
| Fetching list of Entities, Fetching detail of Entity, Entity update, Entity delete are implemented using the same guidelines.

 .. code-block:: java

    @Controller
    @RequestMapping("abc")
    public class AbcController {

        @ModelAttribute
        public AbcForm setUpAbcForm() {
            return new AbcForm();
        }

        // Handling request of "/abc/create?form"
        @RequestMapping(value = "create", params = "form")
        public String createForm(AbcForm form, Model model) {
            // omitted
            return "abc/createForm";
        }

        // Handling request of "POST /abc/create?confirm"
        @RequestMapping(value = "create", method = RequestMethod.POST, params = "confirm")
        public String createConfirm(@Validated AbcForm form, BindingResult result,
                Model model) {
            if (result.hasErrors()) {
                return createRedo(form, model);
            }
            // omitted
            return "abc/createConfirm";
        }

        // Handling request of "POST /abc/create?redo"
        @RequestMapping(value = "create", method = RequestMethod.POST, params = "redo")
        public String createRedo(AbcForm form, Model model) {
            // omitted
            return "abc/createForm";
        }

        // Handling request of "POST /abc/create"
        @RequestMapping(value = "create", method = RequestMethod.POST)
        public String create(@Validated AbcForm form, BindingResult result, Model model) {
            if (result.hasErrors()) {
                return createRedo(form, model);
            }
            // omitted
            return "redirect:/abc/create?complete";
        }

        // Handling request of "/abc/create?complete"
        @RequestMapping(value = "create", params = "complete")
        public String createComplete() {
            // omitted
            return "abc/createComplete";
        }

    }

|
|

.. _controller_method_argument-label:

Regarding arguments of processing method
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`The arguments of processing method can be used to fetch various values <http://docs.spring.io/spring/docs/4.1.9.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-arguments>`_;
however, as a principle rule, the following should not be fetched using arguments of processing method of controller.

* ServletRequest
* HttpServletRequest
* org.springframework.web.context.request.WebRequest
* org.springframework.web.context.request.NativeWebRequest
* java.io.InputStream
* java.io.Reader
* java.io.OutputStream
* java.io.Writer 
* java.util.Map
* org.springframework.ui.ModelMap

.. note::
    When generalized values like HttpServletRequest, getAttribute/setAttribute of HttpSession and get/put of Map are allowed, liberal use of these can degrade the maintainability of
    the project with an increase in project size. 

    When common parameters (request parameters) need to be stored in JavaBean and passed as an argument to a method of controller,
    it can be implemented using :ref:`methodargumentresolver` as described later.

|

Arguments depending on the purpose of usage are described below.

- :ref:`controller_method_argument-model-label`
- :ref:`controller_method_argument-pathvariable-label`
- :ref:`controller_method_argument-requestparam-label`
- :ref:`controller_method_argument-form-label`
- :ref:`controller_method_argument-validation-label`
- :ref:`controller_method_argument-redirectattributes-label`
- :ref:`controller_method_argument-redirectattributes-param-label`
- :ref:`controller_method_argument-redirectattributes-path-label`
- :ref:`controller_method_argument-cookievalue-label`
- :ref:`controller_method_argument-cookiewrite-label`
- :ref:`controller_method_argument-pagination-label`
- :ref:`controller_method_argument-upload-label`
- :ref:`controller_method_argument-message-label`

|

.. _controller_method_argument-model-label:

Passing data to screen (View)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
To pass data to be displayed on screen (View), fetch ``org.springframework.ui.Model``\  (Hereafter called as ``Model``) as argument of processing method and
add the data (Object) to \ ``Model``\  object.

- SampleController.java

 .. code-block:: java
    :emphasize-lines: 2-4

    @RequestMapping("hello")
    public String hello(Model model) { // (1)
        model.addAttribute("hello", "Hello World!"); // (2)
        model.addAttribute(new HelloBean("Bean Hello World!")); // (3)
        return "sample/hello"; // returns view name
    }

- hello.jsp

 .. code-block:: jsp
    :emphasize-lines: 1-2

    Message : ${f:h(hello)}<br> <%-- (4) --%>
    Message : ${f:h(helloBean.message)}<br> <%-- (5) --%>

- HTML of created by View(hello.jsp)

 .. code-block:: html
    :emphasize-lines: 1-2

    Message : Hello World!<br> <!-- (6) -->
    Message : Bean Hello World!<br> <!-- (6) -->


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     -  \ Fetch ``Model``\  object as argument.
   * - | (2)
     - | Call \ ``addAttribute``\  method of \ ``Model``\  object received as argument, and add the data to \ ``Model``\  object.
       | For example, ``"HelloWorld!"`` string is added to the attribute name ``"hello"``.
   * - | (3)
     - | If first argument of \ ``addAttribute``\  method is omitted, the class name beginning with lower case letter will become the attribute name.
       | For example, the result of ``model.addAttribute("helloBean", new HelloBean());`` is same as the result of ``model.addAttribute(new HelloBean());``
   * - | (4)
     - | In View (JSP), it is possible to acquire the data added to \ ``Model``\  object by specifying "${Attribute name}".
       | For example, HTML escaping is performed using  "${f:h(Attribute name)}" function of EL expression.
       | For details of functions of EL expression that perform HTML escaping, refer to :doc:`Cross Site Scripting <../Security/XSS>`.
   * - | (5)
     - | The values of JavaBean stored in \ ``Model``\  can be acquired by specifying "${Attribute name.property name}".
   * - | (6)
     - | JSP is output in HTML format.

 .. note::
   Even though the \ ``Model``\  is not used, it can be specified as an argument. Even if it is not required at the initial stage of implementation,
   it can be used later (so that the signature of methods need not be changed in future).

 .. note::
   The value can also be referred from the module which is not managed under Spring MVC (for example, ServletFilter, etc.) since 
   ``addAttribute`` in  ``Model`` performs a ``setAttribute`` in ``HttpServletRequest``.

|

.. _controller_method_argument-pathvariable-label:

Retrieving values from URL path
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| To retrieve values from URL path, add \ ``@PathVariable``\  annotation to argument of processing method of controller.
| In order to retrieve values from the path using \ ``@PathVariable``\  annotation, value of \ ``@RequestMapping``\  annotation must contain those values in the form of variables (for example, {id}).

 .. code-block:: java
    :emphasize-lines: 1,3,4

    @RequestMapping("hello/{id}/{version}") // (1)
    public String hello(
            @PathVariable("id") String id, // (2)
            @PathVariable Integer version, // (3)
            Model model) {
        // do something
        return "sample/hello"; // returns view name
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify the portion to be extracted as path variable in the value of \ ``@RequestMapping``\  annotation. Specify path variable in "{variable name}" format.
       | For example, 2 path variables such as  ``"id"`` and ``"version"`` are specified. 
   * - | (2)
     - | Specify variable name of path variable in \ ``@PathVariable``\  annotation.
       | For example, when the URL ``"sample/hello/aaaa/1"`` is accessed, the string ``"aaaa"`` is passed to argument "id".
   * - | (3)
     - | Value attribute of ``@PathVariable``\  annotation can be omitted. When it is omitted, the argument name is considered as the request parameter name.
       | In the above example, when the URL ``"sample/hello/aaaa/1"`` is accessed, value ``"1"`` is passed to argument "version".
       | However, in this method compilation needs to be done by specifying either of:

       * \ ``-g``\  option (mode to output debug information)
       * \ ``-parameters``\  option added from Java8 (mode to generate metadata for reflection in the method parameters)


 .. note::
    Binding argument can be of any data type other than string. In case of different data type, \ ``org.springframework.beans.TypeMismatchException``\  is thrown and default response is 400 (Bad Request). 
    For example, when the URL ``"sample/hello/aaaa/v1"`` is accessed, an exception is thrown since ``"v1"`` cannot be converted into Integer type.

 .. warning::
    When omitting the value attribute of ``@PathVariable``\  annotation, the application to be deployed needs to be compiled by specifying \ ``-g``\  option or \ ``-parameters``\  option which is added from Java8.
    When these options are specified, there is a likely impact on memory and processing performance since information or processing required for debugging gets appended to the class after compilation.
    Basically, it is recommended to explicitly specify the value attribute.

|

.. _controller_method_argument-requestparam-label:

Retrieving request parameters individually
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
To retrieve request parameters individually, add \ ``@RequestParam``\  annotation to argument.

 .. code-block:: java
    :emphasize-lines: 3-6

    @RequestMapping("bindRequestParams")
    public String bindRequestParams(
            @RequestParam("id") String id, // (1)
            @RequestParam String name, // (2)
            @RequestParam(value = "age", required = false) Integer age, // (3)
            @RequestParam(value = "genderCode", required = false, defaultValue = "unknown") String genderCode, // (4)
            Model model) {
        // do something
        return "sample/hello"; // returns view name
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify request parameter name in the value attribute of \ ``@RequestParam``\  annotation.
       | For example, when the URL ``"sample/hello?id=aaaa"`` is accessed, the string ``"aaaa"`` is passed to argument "id".
   * - | (2)
     - | value attribute of ``@RequestParam``\  annotation can be omitted. When it is omitted, the argument name becomes the request parameter name. 
       | For example, when the URL ``"sample/hello?name=bbbb&...."`` is accessed, string ``"bbbb"`` is passed to argument "name".
       | However, in this method compilation needs to be done by specifying either of:

       * \ ``-g``\  option (mode to output debug information)
       * \ ``-parameters``\  option added from Java8 (mode to generate metadata for reflection in the method parameters)

   * - | (3)
     - | By default, an error occurs if the specified request parameter does not exist. When request parameter is not required, specify ``false`` in the ``required`` attribute.
       | For example, when it is accessed where request parameter ``age`` does not exist, \ ``null``\  is passed to argument "age".
   * - | (4)
     - | When default value is to be used if the specified request parameter does not exist, specify the default value in defaultValue attribute.
       | For example, when it is accessed where request parameter ``genderCode`` does not exist, ``"unknown"`` is passed to argument "genderCode".


 .. note::
    When it is accessed without specifying mandatory parameters, \ ``org.springframework.web.bind.MissingServletRequestParameterException``\  is thrown and default operation is responded with 400 (Bad Request).
    However, when defaultValue attribute is specified, the value specified in defaultValue attribute is passed without throwing exception.

 .. note::
    Binding argument can be of any data type. In case the data type do not match, \ ``org.springframework.beans.TypeMismatchException``\  is thrown and default response is 400 (Bad Request).
    For example, when ``"sample/hello?age=aaaa&..."`` URL is accessed, exception is thrown since ``aaaa`` cannot be converted into Integer.

|

**Binding to form object must be done only when any of the following conditions are met.**

- If request parameter is an item in the HTML form.
- If request parameter is not an item in HTML form, however, input validation other than mandatory check needs to be performed.
- If error details of input validation error needs to be output for each parameter.
- If there are 3 or more request parameters. (maintenance and readability point of view)

|

.. _controller_method_argument-form-label:

Retrieving request parameters collectively
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Use form object to collectively fetch all the request parameters.
| Form object is JavaBean representing HTML form. For the details of form object, refer to :ref:`formobject`.

Following is an example that shows the difference between processing method that fetches each request parameter using ``@RequestParam`` and the same processing method when fetching request parameters in a form object

Processing method that receives request parameter separately using ``@RequestParam`` is as shown below.

 .. code-block:: java

    @RequestMapping("bindRequestParams")
    public String bindRequestParams(
            @RequestParam("id") String id,
            @RequestParam String name,
            @RequestParam(value = "age", required = false) Integer age,
            @RequestParam(value = "genderCode", required = false, defaultValue = "unknown") String genderCode,
            Model model) {
        // do something
        return "sample/hello"; // returns view name
    }

| Create form object class
| For jsp of HTML form corresponding to this form object, refer to :ref:`formobjectjsp`.

 .. code-block:: java

    public class SampleForm implements Serializable{
        private static final long serialVersionUID = 1477614498217715937L;

        private String id;
        private String name;
        private Integer age;
        private String genderCode;

        // omit setters and getters

    }

 .. note::
  **Request parameter name should match with form object property name.**

  When parameters ``"id=aaa&name=bbbb&age=19&genderCode=men?tel=01234567"`` are sent to the above form object,
   the values of ``id`` , ``name`` , ``age`` , ``genderCode`` matching with the property name, are stored, however ``tel`` is not included in form object, as it does not have matching property name.

  Make changes such that request parameters which were being fetched individually using ``@RequestParam`` now get fetched as form object.

 .. code-block:: java
    :emphasize-lines: 2

    @RequestMapping("bindRequestParams")
    public String bindRequestParams(@Validated SampleForm form, // (1)
            BindingResult result,
            Model model) {
        // do something
        return "sample/hello"; // returns view name
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - Receive \ ``SampleForm``\  object as argument.

 .. note::
    When form object is used as argument, unlike \ ``@RequestParam``\ ,
    mandatory check is not performed. \ **When using form object,** :ref:`controller_method_argument-validation-label` **should be performed as described below**\.

.. warning::
    Domain objects such as Entity, etc. can also be used as form object without any changes required. 
    However, the parameters such as password for confirmation, agreement confirmation checkbox, etc. should exist only on WEB screen.
    Since the fields depending on such screen items should not be added to domain objects, it is recommended to create class for form object separate from domain object. 
    When a domain object needs to be created from request parameters, values must first be received in form object and then copied to domain object from form object.

|

.. _controller_method_argument-validation-label:

Performing input validation
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
When performing input validation for the form object, add \ ``@Validated``\  annotation to form object argument, and specify \ ``org.springframework.validation.BindingResult``\  (Hereafter 
called as \ ``BindingResult``\ ) to argument immediately after form object argument.

Refer to :doc:`../ArchitectureInDetail/Validation`  for the details of input validation.

Add annotations required in input validation to the fields of form object class.

 .. code-block:: java

    public class SampleForm implements Serializable {
        private static final long serialVersionUID = 1477614498217715937L;

        @NotNull
        @Size(min = 10, max = 10)
        private String id;

        @NotNull
        @Size(min = 1, max = 10)
        private String name;

        @Min(1)
        @Max(100)
        private Integer age;

        @Size(min = 1, max = 10)
        private Integer genderCode;

        // omit setters and getters
    }


| Add \ ``@Validated``\  annotation to form object argument.
| Input validation is performed for the argument with ``@Validated`` annotation before the processing method of controller is executed. The check result is stored in the argument \ ``BindingResult``\  which immediately follows form object argument.
| The type conversion error that occurs when a data-type other than String is specified in form object, is also stored in \ ``BindingResult``\ .

 .. code-block:: java
    :emphasize-lines: 2,3,5

    @RequestMapping("bindRequestParams")
    public String bindRequestParams(@Validated SampleForm form, // (1)
            BindingResult result, // (2)
            Model model) {
        if (result.hasErrors()) { // (3)
            return "sample/input"; // back to the input view 
        }
        // do something
        return "sample/hello"; // returns view name
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - Add \ ``@Validated``\  annotation to \ ``SampleForm``\  argument, and mark it as target for input validation. 
   * - | (2)
     - Specify \ ``BindingResult``\  in the argument where input validation result is stored.
   * - | (3)
     - Check if input validation error exists. If there is an error, ``true`` is returned.

|

.. _controller_method_argument-redirectattributes-label:

Passing data while redirecting request
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
To redirect after executing a processing method ofcontroller and to pass data along with it, fetch \ ``org.springframework.web.servlet.mvc.support.RedirectAttributes``\  (Henceforth called as \ ``RedirectAttributes``\ ) as an argument of processing method, 
and add the data to ``RedirectAttributes`` object.

- SampleController.java

 .. code-block:: java
    :emphasize-lines: 2-5,10

    @RequestMapping("hello")
    public String hello(RedirectAttributes redirectAttrs) { // (1)
        redirectAttrs.addFlashAttribute("hello", "Hello World!"); // (2)
        redirectAttrs.addFlashAttribute(new HelloBean("Bean Hello World!")); // (3)
        return "redirect:/sample/hello?complete"; // (4)
    }

    @RequestMapping(value = "hello", params = "complete")
    public String helloComplete() { 
        return "sample/complete"; // (5)
    }

- complete.jsp

 .. code-block:: jsp
    :emphasize-lines: 1-2

    Message : ${f:h(hello)}<br> <%-- (6) --%>
    Message : ${f:h(helloBean.message)}<br> <%-- (7) --%>

- HTML of created by View(complete.jsp)

 .. code-block:: html
    :emphasize-lines: 1-2

    Message : Hello World!<br> <!-- (8) -->
    Message : Bean Hello World!<br> <!-- (8) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - Fetch \ ``RedirectAttributes``\  object as argument of the processing method of controller.
   * - | (2)
     - | Call \ ``addFlashAttribute``\  method of \ ``RedirectAttributes``\  and add the data to \ ``RedirectAttributes``\  object. 
       | For example, the string data ``"HelloWorld!"`` is added to attribute name ``"hello"``. 
   * - | (3)
     - | If first argument of \ ``addFlashAttribute``\  method is omitted, the class name beginning with lower case letter becomes the attribute name.
       | For example, the result of ``model.addFlashAttribute("helloBean", new HelloBean());`` is same as ``model.addFlashAttribute(new HelloBean());``.
   * - | (4)
     - | Send a redirect request to another URL which will display the next screen instead of displaying screen (View) directly.
   * - | (5)
     - | In the processing method after redirection, return view name of the screen that displays the data added in (2) and (3).
   * - | (6)
     - | In the View (JSP) side, the data added to \ ``RedirectAttributes``\  object can be obtained by specifying "${attribute name}".
       | For example, HTML escaping is performed using "${f:h(attribute name)}" function of EL expression.
       | For the details of functions of EL expression that performs HTML escaping, refer to :doc:`Cross Site Scripting <../Security/XSS>`.
   * - | (7)
     - | The value stored in \ ``RedirectAttributes``\  can be obtained from JavaBean by using "${Attribute name.Property name}". 
   * - | (8)
     - | HTML output.

.. warning::
    The data cannot be passed to redirect destination even though it is added to ``Model``.
 
.. note::

    It is similar to the \ ``addAttribute``\  method of \ ``Model``\ . However survival time of data differs. 
    In \ ``addFlashAttribute``\  of \ ``RedirectAttributes``\ , the data is stored in a scope called flash scope.
    Data of only 1 request (G in PRG pattern) can be referred after redirect. The data from the second request onwards is deleted.


.. figure:: images_ApplicationLayer/applicationFlashscope.png
   :alt: Survival time of flush scope
   :width: 80%
   :align: center

   **Picture - Survival time of flush scope**
 
|

.. _controller_method_argument-redirectattributes-param-label:

Passing request parameters to redirect destination
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
When request parameters are to be set dynamically to redirect destination, add the values to be passed to \ ``RedirectAttributes``\  object of argument. 

 .. code-block:: java
    :emphasize-lines: 4

    @RequestMapping("hello")
    public String hello(RedirectAttributes redirectAttrs) {
        String id = "aaaa";
        redirectAttrs.addAttribute("id", id); // (1)
        // must not return "redirect:/sample/hello?complete&id=" + id;
        return "redirect:/sample/hello?complete";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify request parameter name in argument ``name and request parameter value in argument ``value`` and call \ ``addAttribute``\  method of \ ``RedirectAttributes``\  object. 
       | In the above example, it is redirected to ``"/sample/hello?complete&id=aaaa"``.
       
.. warning::
    In the above example, the result is the same as of ``return "redirect:/sample/hello?complete&id=" + id;``\  (as shown in the commented out line in the above example).
    However, since URL encoding is also performed if \ ``addAttribute``\  method of ``RedirectAttributes``\  object is used,
    the request parameters that needs to be inserted dynamically **should be set to the request parameter using addAttribute method and should not be set to redirect URL specified as return value.**
    The request parameters which are not to be inserted dynamically ("complete" as in the above example), can be directly specified in the redirect URL specified as the return value.

|

.. _controller_method_argument-redirectattributes-path-label:

Inserting values in redirect destination URL path
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
To insert values in redirect destination URL path dynamically, add the value to be inserted in \ ``RedirectAttributes``\  object of argument as shown in the example to set request parameters.

 .. code-block:: java
    :emphasize-lines: 4,6

    @RequestMapping("hello")
    public String hello(RedirectAttributes redirectAttrs) {
        String id = "aaaa";
        redirectAttrs.addAttribute("id", id); // (1)
        // must not return "redirect:/sample/hello/" + id + "?complete";
        return "redirect:/sample/hello/{id}?complete"; // (2)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify attribute name and the value using \ ``addAttribute``\  method of \ ``RedirectAttributes``\  object. 
   * - | (2)
     - | Specify the path of the variable "{Attribute name}"  to be inserted in the redirect URL. 
       | In the above example, it is redirected to ``"/sample/hello/aaaa?complete"``.

.. warning::
    In the above example, the result is same as of ``"redirect:/sample/hello/" + id + "?complete";``\  (as shown in the commented out line in the above example).
    However, since URL encoding is also performed when using \ ``addAttribute``\  method of ``RedirectAttributes``\  object,
    the path values to be inserted dynamically **should be inserted using addAttribute method and path variable and should not be set to redirect URL specified as return value.**

|

.. _controller_method_argument-cookievalue-label:

Acquiring values from Cookie
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Add \ ``@CookieValue``\  annotation to the argument of processing method to acquire the values from a cookie. 

 .. code-block:: java
    :emphasize-lines: 2

    @RequestMapping("readCookie")
    public String readCookie(@CookieValue("JSESSIONID") String sessionId, Model model) { // (1)
        // do something
        return "sample/readCookie"; // returns view name
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify name of the cookie in the ``value`` attribute of \ ``@CookieValue``\  annotation. 
       | In the above example, "JSESSIONID" value is passed from cookie to sessionId argument.

.. note::
    As in the case of ``@RequestParam`` , it has ``required`` attribute and ``defaultValue`` attribute. Also, the data type of the argument need not be String. 
     Refer to :ref:`controller_method_argument-requestparam-label` for details.

|

.. _controller_method_argument-cookiewrite-label:

Writing values in Cookie
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| To write values in cookie, call \ ``addCookie``\  method of \ ``HttpServletResponse``\  object directly and add the value to cookie.
| Since there is no way to write to cookie in Spring MVC  (3.2.3 version), ** Only in this case, HttpServletResponse can fetched as an argument of processing method of controller.** 

 .. code-block:: java
    :emphasize-lines: 3,5

    @RequestMapping("writeCookie")
    public String writeCookie(Model model,
            HttpServletResponse response) { // (1)
        Cookie cookie = new Cookie("foo", "hello world!");
        response.addCookie(cookie); // (2)
        // do something
        return "sample/writeCookie";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * -  Sr. No.
     - Description
   * - | (1)
     - Specify \ ``HttpServletResponse``\  object as argument to write to cookie. 
   * - | (2)
     - | Generate \ ``Cookie``\  object and add to \ ``HttpServletResponse``\  object. 
       | For example, ``"hello world!"``  value is assigned to Cookie name ``"foo"``. 

.. tip::

    No difference compared to use of \ ``HttpServletResponse``\  which fetched as an argument of processing method, however,  \ ``org.springframework.web.util.CookieGenerator``\  class is provided by Spring
    as a class to write values in cookie. It should be used if required. 

|

.. _controller_method_argument-pagination-label:

Retrieving pagination information
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Pagination related information is required for the requests performing list search. 
| Fetching ``org.springframework.data.domain.Pageable``\  (henceforth called as \ ``Pageable``\ ) object as an argument of processing method enables to handle pagination related information (page count, fetch record count) easily.

 Refer to :doc:`../ArchitectureInDetail/Pagination`  for details.

|

.. _controller_method_argument-upload-label:

Retrieving uploaded file
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Uploaded file can be obtained in 2 ways.

- Provide ``MultipartFile`` property in form object.
- Use \ ``org.springframework.web.multipart.MultipartFile``\  as an argument of processing method having \ ``@RequestParam``\  annotation.

Refer to :doc:`../ArchitectureInDetail/FileUpload`  for details.

|

.. _controller_method_argument-message-label:

Displaying result message on the screen
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\ ``Model``\  object or \ ``RedirectAttributes``\  object can be obtained as an argument of processing method and 
result message of business logic execution can be displayed by adding \ ``ResultMessages``\  object to Model or RedirectAttributes.


Refer to :doc:`../ArchitectureInDetail/MessageManagement`  for details.

|
|

.. _controller_method_return-label:

Regarding return value of processing method
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
`For return values of processing methods, various values can be fetched <http://docs.spring.io/spring/docs/4.1.9.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-return-types>`_ ; however,
only the following values should be used.


- String (for logical name of view)

Return types depending on the purpose of usage are described below:

- :ref:`controller_method_return-html-label`
- :ref:`controller_method_return-download-label`

|

.. _controller_method_return-html-label:

HTML response
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| To get HTML response to display the output of processing method, it has to return view name of JSP.
| \ ``ViewResolver``\  when generating HTML using JSP will be an inherited class of \ ``UrlBasedViewResolver``\  (\ ``InternalViewResolver``\  and``TilesViewResolver``\ , etc).

| An example using \ ``InternalViewResolver``\  for JSP is given below; however, it is recommended to use \ ``TilesViewResolver``\  when the screen layout is in a templated format.
| Refer to :doc:`../ArchitectureInDetail/TilesLayout`  for the usage of \ ``TilesViewResolver``\ .

- spring-mvc.xml

Example of definition when \ ``<bean>``\  element is to be used
 
 .. code-block:: xml

    <!-- (1) -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/" /> <!-- (2) -->
        <property name="suffix" value=".jsp" /> <!-- (3) -->
        <property name="order" value="1" /> <!-- (4) -->
    </bean>

 Example of definition when using \ ``<mvc:view-resolvers>``\  element added from Spring Framework 4.1

 .. code-block:: xml

    <mvc:view-resolvers>
        <mvc:jsp prefix="/WEB-INF/views/" /> <!-- (5) -->
    </mvc:view-resolvers>


- SampleController.java

 .. code-block:: java
    :emphasize-lines: 4

    @RequestMapping("hello")
    public String hello() {
        // omitted
        return "sample/hello"; // (6)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - Define \ ``InternalViewResolver``\  for JSP.
   * - | (2)
     - Specify base directory (prefix of file path) where JSP files are stored.

       By specifying prefix, there is no need to specify physical storage location of JSP files at the time of returning View name in Controller.
   * - | (3)
     - Specify extension (suffix of file path) of JSP file.

       By specifying suffix, specifying extension of JSP files at the time of returning View name in Controller is no longer needed.
   * - | (4)
     - Specify execution order when multiple \ ``ViewResolver``\  are specified.

       It can be specified in the range of \ ``Integer``\  and executed sequentially from smallest value.
   * - | (5)
     - Define \ ``InternalViewResolver``\  for JSP using \ ``<mvc:jsp>``\  element added from Spring Framework 4.1.

       * In \ ``prefix``\  attribute, specify base directory (prefix of file path) where JSP file is stored.
       * It need  not be explicitly specified in \ ``suffix``\  attribute as \ ``".jsp"``\  is used as default value.

       .. note::

           When \ ``<mvc:view-resolvers>``\  element is used, it is possible to define \ ``ViewResolver``\  in simple way.
           Hence this guideline recommends to use \ ``<mvc:view-resolvers>``\ .

   * - | (6)
     - When View name ``"sample/hello"`` is the return value of processing method, ``"/WEB-INF/views/sample/hello.jsp"`` is called and HTML is sent as response.

.. note::
    HTML output is generated using JSP in the above example, however, even if HTML is generated using other template engine such as Velocity, FreeMarker, return value of processing method will be ``"sample/hello``. 
    ``ViewResolver`` takes care of task to determine which template engine is to be used.

|

.. _controller_method_return-download-label:

Responding to downloaded data
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| In order to return the data stored in db as download data (\ ``"application/octet-stream"``\ ), it is recommended to create a view 
| for generating response data (download process).The processing method adds the data to be downloaded to \ ``Model``\  and returns 
| name of the view which performs the actual download process.

| The solution to create a separate ViewResolver to resolve a view using its view name, however, \ ``BeanNameViewResolver``\  provided by Spring Framework is recommended.
| Refer to :doc:`../ArchitectureInDetail/FileDownload`  for the details of download processing.

- spring-mvc.xml

 Example of definition when \ ``<bean>``\  element is to be used

 .. code-block:: xml
    :emphasize-lines: 1-4

    <!-- (1) -->
    <bean class="org.springframework.web.servlet.view.BeanNameViewResolver">
        <property name="order" value="0"/> <!-- (2) -->
    </bean>

    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/" />
        <property name="suffix" value=".jsp" />
        <property name="order" value="1" />
    </bean>

 Example of definition when using \ ``<mvc:view-resolvers>``\  element added from Spring Framework 4.1

 .. code-block:: xml
    :emphasize-lines: 2

    <mvc:view-resolvers>
        <mvc:bean-name /> <!-- (3) -->
        <mvc:jsp prefix="/WEB-INF/views/" />
    </mvc:view-resolvers>

- SampleController.java

 .. code-block:: java
    :emphasize-lines: 4

    @RequestMapping("report")
    public String report() {
        // omitted
        return "sample/report"; // (4)
    }


- XxxExcelView.java

 .. code-block:: java
    :emphasize-lines: 1-2

    @Component("sample/report") // (5)
    public class XxxExcelView extends AbstractExcelView { // (6)
        @Override
        protected void buildExcelDocument(Map<String, Object> model,
                HSSFWorkbook workbook, HttpServletRequest request,
                HttpServletResponse response) throws Exception {
            HSSFSheet sheet;
            HSSFCell cell;

            sheet = workbook.createSheet("Spring");
            sheet.setDefaultColumnWidth(12);

            // write a text at A1
            cell = getCell(sheet, 0, 0);
            setText(cell, "Spring-Excel test");

            cell = getCell(sheet, 2, 0);
            setText(cell, (Date) model.get("serverTime")).toString());
        }
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - Define \ ``BeanNameViewResolver``\ .

       \ ``BeanNameViewResolver``\  is a class that resolves View by searching for the bean which matches with the returned View name, from application context.
   * - | (2)
     - When \ ``InternalViewResolver``\  for JSP and \ ``TilesViewResolver``\  are to be used together, it is recommended to give it a higher priority compared to these \ ``ViewResolver``\ .
       In the above example, by specifying ``"0"``, View is resolved by \ ``BeanNameViewResolver``\  prior to \ ``InternalViewResolver``\ .
   * - | (3)
     - Define \ ``BeanNameViewResolver``\  using \ ``<mvc:bean-name>``\  element added from Spring Framework 4.1.

       When defining \ ``ViewResolver``\  using \ ``<mvc:view-resolvers>``\  element, definition order of \ ``ViewResolver``\  specified in child element will be the priority order.
       In the above example, by defining it above (\ ``<mvc:jsp>``\ ) element in order to define \ ``InternalViewResolver``\  for JSP, View is resolved by ``BeanNameViewResolver``\  prior to \ ``InternalViewResolver``\  for JSP.

       .. note::

           When \ ``<mvc:view-resolvers>``\  element is used, it is possible to define \ ``ViewResolver``\  in a simple way.
           Hence, this guideline recommends to use \ ``<mvc:view-resolvers>``\ .
   * - | (4)
     - When View name ``"sample/report"`` is the return value of processing method, the data generated by View instance which is registered in step (5), is responded as download data.
   * - | (5)
     - Register View object as Bean by specifying View name to the name of component.

       In above example, ``x.y.z.app.views.XxxExcelView`` instance is registered as a bean with bean name (view name) as ``"sample/report"`` .
   * - | (6)
     - Example of View implementation.

       Implementation of View class that inherits ``org.springframework.web.servlet.view.document.AbstractExcelView`` and generates Excel data.

|
|

Implementing the process
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| The point here is that **do not implement business logic in controller** .
| Business logic must be implemented in Service. Controller must call the service methods in which the business logic is implemented.
| Refer to :doc:`DomainLayer`  for the details of implementation of business logic.

.. note::
    Controller should be used only for routing purposes (mapping requests to corresponding business logic) and deciding the screen transition for each request as well as setting model data. Thereby, controller should be simple as much as possible.
    By consistently following this policy, the contents of controller become clear which ensures maintainability of controller even if the size of development is large. 

|

Operations to be performed in controller are shown below:

- :ref:`controller_logic_correlationcheck-label`
- :ref:`controller_logic_businesslogic-label`
- :ref:`controller_logic_domainobject-label`
- :ref:`controller_logic_formobject-label`

|

.. _controller_logic_correlationcheck-label:

Correlation check of input value
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Correlation check of input values should be done using ``Validation`` class which implements \ ``org.springframework.validation.Validator``\  interface. 
| Bean Validation can also be used for correlation check of input values.
| Refer to :doc:`../ArchitectureInDetail/Validation`  for the details of implementation of correlation check.

| The implementation of correlation check itself should not be written in the processing method of controller. However, it is necessary to add the  \ ``Validator``\  to \ ``org.springframework.web.bind.WebDataBinder``\ . 

 .. code-block:: java
    :emphasize-lines: 2,6

    @Inject
    PasswordEqualsValidator passwordEqualsValidator; // (1)

    @InitBinder
    protected void initBinder(WebDataBinder binder){
        binder.addValidators(passwordEqualsValidator); // (2)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - Inject \ ``Validator``\  that performs correlation check.
   * - | (2)
     - | Add the injected \ ``Validator``\  to \ ``WebDataBinder``\ .
       | Adding the above to \ ``WebDataBinder``\  enables correlation check by executing \ ``Validator``\  before the processing method gets called.

|

.. _controller_logic_businesslogic-label:

Calling business logic
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Execute business logic by injecting the Service in which business logic is implemented and calling the injected Service method.

 .. code-block:: java
    :emphasize-lines: 2,6

    @Inject
    SampleService sampleService; // (1)

    @RequestMapping("hello")
    public String hello(Model model){
        String message = sampleService.hello(); // (2)
        model.addAttribute("message", message);
        return "sample/hello";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Inject the ``Service`` in which business logic is implemented.
   * - | (2)
     - Call the injected ``Service`` method to execute business logic.

|

.. _controller_logic_domainobject-label:

Reflecting values to domain object
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| In this guideline, it is recommended to bind the data sent by HTML form to form object instead of the domain object.
| Therefore, the controller should perform the process of reflecting the values of form object to domain object which is then passed to the method of service class.

 .. code-block:: java
    :emphasize-lines: 4,11-12

    @RequestMapping("hello")
    public String hello(@Validated SampleForm form, BindingResult result, Model model){
        // omitted
        Sample sample = new Sample(); // (1)
        sample.setField1(form.getField1());
        sample.setField2(form.getField2());
        sample.setField3(form.getField3());
        // ...
        // and more ...
        // ...
        String message = sampleService.hello(sample); // (2)
        model.addAttribute("message", message); // (3)
        return "sample/hello";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Create domain object and reflect the values bound to form object in the domain object. 
   * - | (2)
     - | Call the method of service class to execute business logic.
   * - | (3)
     - | Add the data returned from business logic to \ ``Model``\ .

| The process of reflecting values to domain object should be implemented by the processing method of controller. However considering the readability of processing 
| method in case of large amount of code, it is recommended to delegate the process to Helper class.
| Example of delegating the process to Helper class is shown below:

- SampleController.java

 .. code-block:: java
    :emphasize-lines: 2,7

    @Inject
    SampleHelper sampleHelper; // (1)

    @RequestMapping("hello")
    public String hello(@Validated SampleForm form, BindingResult result){
        // omitted
        String message = sampleHelper.hello(form); // (2)
        model.addAttribute("message", message);
        return "sample/hello";
    }
    
- SampleHelper.java

 .. code-block:: java
    :emphasize-lines: 6

    public class SampleHelper {
    
        @Inject
        SampleService sampleService;
        
        public String hello(SampleForm form){ // (3)
            Sample sample = new Sample();
            sample.setField1(form.getField1());
            sample.setField2(form.getField2());
            sample.setField3(form.getField3());
            // ...
            // and more ...
            // ...
            String message = sampleService.hello(sample);
            return message;
        }
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - Inject object of Helper class in controller.
   * - | (2)
     - Value is reflected to the domain object by calling the method of the injected Helper class.
       Delegating the process to Helper class enables to keep the implementation of controller simple.
   * - | (3)
     - Call the Service class method to execute the business logic after creating domain object.

 .. note::
    Bean conversion functionality can be used as an alternative way to delegate the process of reflecting form object values, to Helper class.
    Refer to :doc:`../ArchitectureInDetail/Utilities/Dozer`  for the details of Bean conversion functionality.

|

.. _controller_logic_formobject-label:

Reflecting values to form object
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| In this guideline, it is recommended that form object (and not domain object) must be used to for that data which is to be bound to HTML form.
| For this, it is necessary to reflect the values of domain object (returned by service layer) to form object. This conversion should be performed in controller class.


 .. code-block:: java
    :emphasize-lines: 4,5,11

    @RequestMapping("hello")
    public String hello(SampleForm form, BindingResult result, Model model){
        // omitted
        Sample sample = sampleService.getSample(form.getId()); // (1)
        form.setField1(sample.getField1()); // (2)
        form.setField2(sample.getField2());
        form.setField3(sample.getField3());
        // ...
        // and more ...
        // ...
        model.addAttribute(sample); // (3)
        return "sample/hello";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Call the method of service class in which business logic is implemented and fetch domain object.
   * - | (2)
     - | Reflect values of acquired domain object to form object.
   * - | (3)
     - | When there are fields only for display, add domain object to \ ``Model``\  so that data can be referred.

 .. note::
    In JSP, it is recommended to refer the values from domain object instead of form object for the fields to be only displayed on the screen.

The process of reflecting value to form object should be implemented by the processing method of controller.
However considering the readability of processing method in case of large amount of code, it is recommended to delegate the process to Helper class method.

- SampleController.java

 .. code-block:: java
    :emphasize-lines: 5

    @RequestMapping("hello")
    public String hello(@Validated SampleForm form, BindingResult result){
        // omitted
        Sample sample = sampleService.getSample(form.getId());
        sampleHelper.applyToForm(sample, form); // (1)
        model.addAttribute(sample);
        return "sample/hello";
    }
    
- SampleHelper.java

 .. code-block:: java
    :emphasize-lines: 2

    public void applyToForm(SampleForm destForm, Sample srcSample){
        destForm.setField1(srcSample.getField1()); // (2)
        destForm.setField2(srcSample.getField2());
        destForm.setField3(srcSample.getField3());
        // ...
        // and more ...
        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Call the method to reflect the values of domain object to form object.
   * - | (2)
     - | Reflect the values of domain object to form object.

 .. note::
    Bean conversion functionality can be used as an alternative method to delegate the process to Helper class.
    Refer to :doc:`../ArchitectureInDetail/Utilities/Dozer`  for the details of Bean conversion functionality.

|
|

.. _formobject:

Implementing form object
--------------------------------------------------------------------------------
Form object is the object (JavaBean) which represents HTML form and plays the following role.

#. **Holds business data stored in the database so that it can be referred by HTML form (JSP).**
#. **Holds request parameters sent by HTML form so that they can be referred by processing method of controller.**

.. figure:: ./images_ApplicationLayer/applicationFormobject.png
   :width: 80%
   :align: center

|

Implementation of form object can be described by focusing on the following points.

- :ref:`formobject_new-label`
- :ref:`formobject_init-label`
- :ref:`formobject_bindhtmlform-label`
- :ref:`formobject_bindrequestparam-label`

|

.. _formobject_new-label:

Creating form object
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Create form object as a JavaBean.
Spring Framework provides the functionality to convert and bind the request parameters (string) sent by HTML form to the format defined in form object.
Hence, the fields to be defined in form object need not only be in \ ``java.lang.String``\  format.

 .. code-block:: java

    public class SampleForm implements Serializable {
        private String id;
        private String name;
        private Integer age;
        private String genderCode;
        private Date birthDate;
        // omitted getter/setter
    }

 .. tip:: **Regarding the mechanism provided by Spring Framework that performs format conversion**

    Spring Framework executes format conversion using the following 3 mechanisms and supports conversion to basic format as standard. Refer to linked page for the details of each conversion function.

    * `Spring Type Conversion <http://docs.spring.io/spring/docs/4.1.9.RELEASE/spring-framework-reference/html/validation.html#core-convert>`_\
    * `Spring Field Formatting <http://docs.spring.io/spring/docs/4.1.9.RELEASE/spring-framework-reference/html/validation.html#format>`_\
    * `java.beans.PropertyEditor implementations <http://docs.spring.io/spring/docs/4.1.9.RELEASE/spring-framework-reference/html/validation.html#beans-beans-conversion>`_\

 .. warning::

    In form object, it is recommended to maintain only the fields of HTML form and not the fields which are just displayed on the screen.
    If display only fields are also maintained in form object, more memory will get consumed at the time of storing form object in HTTP session object causing memory exhaustion.
    In order to display the values of display only fields on the screen, it is recommended to add objects of domain layer (such as Entity) to request scope by using (\ ``Model.addAttribute``\ ).

|

Number format conversion of fields
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Number format can be specified for each field using \ ``@NumberFormat``\  annotation.

 .. code-block:: java
    :emphasize-lines: 2

    public class SampleForm implements Serializable {
        @NumberFormat(pattern = "#,#") // (1)
        private Integer price;
        // omitted getter/setter
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify the number format of request parameter sent by HTML form. For example, binding of value formatted by "," is possible since ""#, #"" format is specified as pattern.
       | When value of request parameter is ""1,050"", Integer object of ""1050"" will bind to the property ``price`` of form object.

Attributes of ``@NumberFormat`` annotation are given below.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 10 80

   * - Sr. No.
     - Attribute name
     - Description
   * - 1.
     - style
     - Specify number format style (NUMBER, CURRENCY, PERCENT). For details refer to `Javadoc of NumberFormat.Style <http://docs.spring.io/spring/docs/4.1.9.RELEASE/javadoc-api/org/springframework/format/annotation/NumberFormat.Style.html>`_\ .
   * - 2.
     - pattern
     - Specify number format of Java. Refer to 'Javadoc of DecimalFormat <http://docs.oracle.com/javase/8/docs/api/java/text/DecimalFormat.html>'_\ for details.

|

.. _ApplicationLayer-DateTimeFormat:

Date and time format conversion of fields
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Date and time format for each field can be specified using \ ``@DateTimeFormat``\  annotation.

 .. code-block:: java
    :emphasize-lines: 2

    public class SampleForm implements Serializable {
        @DateTimeFormat(pattern = "yyyyMMdd") // (1)
        private Date birthDate;
        // omitted getter/setter
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - Specify the date and time format of request parameter sent by HTML form. For example, ``"yyyyMMdd"`` format is specified as pattern.
       When the value of request parameter is ``"20131001"``, Date object of 1st October, 2013 will bind to property ``birthDate`` of form object.

Attributes of \ ``@DateTimeFormat``\  annotation are given below.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 10 80

   * - Sr. No.
     - Attribute name
     - Description
   * - 1.
     - iso
     - Specify ISO date and time format. For details refer to `Javadoc of DateTimeFormat.ISO <http://docs.spring.io/spring/docs/4.1.9.RELEASE/javadoc-api/org/springframework/format/annotation/DateTimeFormat.ISO.html>`_\ .
   * - 2.
     - pattern
     - Specify Java date and time format. Refer to 'Javadoc of SimpleDateFormat <http://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html>'_\ for details.
   * - 3.
     - style
     - | Specify style of date and time as two-digit string.
       | First digit will be style of date and second digit will be style of time.
       | Values that can be specified as style are given below.
       |
       | S : Format same as \ ``java.text.DateFormat.SHORT``\ .
       | M : Format same as \ ``java.text.DateFormat.MEDIUM``\ .
       | L : Format same as \ ``java.text.DateFormat.LONG``\ .
       | F : Format same as \ ``java.text.DateFormat.FULL``\ .
       | - : A style meaning omissions.
       |
       | Example of specification and conversion)
       | MM : Dec 9, 2013 3:37:47 AM
       | M- : Dec 9, 2013
       | -M : 3:41:45 AM

|

DataType conversion in controller
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\ ``@InitBinder``\  annotation can be used to define datatype conversions at controller level.

 .. code-block:: java
    :emphasize-lines: 1,5

    @InitBinder // (1)
    public void initWebDataBinder(WebDataBinder binder) {
        binder.registerCustomEditor(
                Long.class,
                new CustomNumberEditor(Long.class, new DecimalFormat("#,#"), true)); // (2)
    }

 .. code-block:: java
    :emphasize-lines: 1

    @InitBinder("sampleForm") // (3)
    public void initSampleFormWebDataBinder(WebDataBinder binder) {
        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | If a method with \ ``@InitBinder``\  annotation is provided, it is called before executing the binding process and thereby default operations can be customized.
   * - | (2)
     - | For example, ``"#. #"`` format is specified for a field of type Long. This enables binding of value formatted with ",".
   * - | (3)
     - | Default operation for each form object can be customized by specifying it in the value attribute of \ ``@InitBinder``\  annotation. 
       | In the above example, the method is called before binding form object ``"sampleForm"``. 

|

Specifying annotation for input validation
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Since form object is validated using Bean Validation, it is necessary to specify the annotation which indicates constraints of the field.
Refer to :doc:`../ArchitectureInDetail/Validation`  for the details of input validation.

|

.. _formobject_init-label:

Initializing form object
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Form object can also be called as form-backing bean and binding can be performed using \ ``@ModelAttribute``\  annotation.
Initialize form-backing bean by the method having \ ``@ModelAttribute``\  annotation.
In this guideline, such methods are called as ModelAttribute methods and defined with method names like \ ``setUpXxxForm``\ .

 .. code-block:: java
    :emphasize-lines: 1

    @ModelAttribute // (1)
    public SampleForm setUpSampleForm() {
        SampleForm form = new SampleForm();
        // populate form
        return form;
    }

 .. code-block:: java
    :emphasize-lines: 1

    @ModelAttribute("xxx") // (2)
    public SampleForm setUpSampleForm() {
        SampleForm form = new SampleForm();
        // populate form
        return form;
    }

 .. code-block:: java
    :emphasize-lines: 3

    @ModelAttribute
    public SampleForm setUpSampleForm(
            @CookieValue(value = "name", required = false) String name, // (3)
            @CookieValue(value = "age", required = false) Integer age,
            @CookieValue(value = "birthDate", required = false) Date birthDate) {
        SampleForm form = new SampleForm();
        form.setName(name);
        form.setAge(age);
        form.setBirthDate(birthDate);
        return form;
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90
   
   * - Sr. No.
     - Description
   * - | (1)
     - | Class name beginning with lower case letter will become the attribute name to add to \ ``Model``\ . In the above example, ``"sampleForm"`` is the attribute name.
       | The returned object is added to \ ``Model``\  and an appropriate process \ ``model.addAttribute(form)``\  is executed.
   * - | (2)
     - | When attribute name is to be specified to add to \ ``Model``\ , specify it in the value attribute of \ ``@ModelAttribute``\  annotation. In the above example, ``"xxx"`` is the attribute name.
       | For returned object, appropriate process "model.addAttribute("xxx", form)"\ is executed and it is returned to \ ``Model``\ .
       | When attribute name other than default value is specified, it is necessary to specify \ ``@ModelAttribute("xxx")``\  at the time of specifying form object as an argument of processing method.
   * - | (3)
     - | ModelAttribute method can pass the parameters required for initialization as with the case of processing method. In the above example, value of cookie is specified using \ ``@CookieValue``\  annotation.

.. note::
    When form object is to be initialized with default values, it should be done using ModelAttribute method.
    In point (3) in above example , value is fetched from cookie, However, fixed value defined in constant class can be set directly.

.. note::
    Multiple ModelAttribute methods can be defined in the controller. Each method is executed before calling processing method of controller.

.. warning::
    If ModelAttribute method is executed for each request, initialization needs to be repeated for each request and unnecessary objects will get created.
    So, for form objects which are required only for specific requests, should be created inside processing method of controller and not through the use of ModelAttribute method.

|

.. _formobjectjsp:

.. _formobject_bindhtmlform-label:

Binding to HTML form
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| It is possible to bind form object added to the \ ``Model``\  to HTML form(JSP) using \ ``<form:xxx>``\  tag.
| For the details of \ ``<form:xxx>``\  tag, refer to `Using Spring's form tag library <http://docs.spring.io/spring/docs/4.1.9.RELEASE/spring-framework-reference/html/view.html#view-jsp-formtaglib>`_\ .

 .. code-block:: jsp
    :emphasize-lines: 1

    <%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %> <!-- (1) -->

 .. code-block:: jsp
    :emphasize-lines: 2,3

    <form:form modelAttribute="sampleForm"
               action="${pageContext.request.contextPath}/sample/hello"> <!-- (2) -->
        Id         : <form:input path="id" /><form:errors path="id" /><br /> <!-- (3) -->
        Name       : <form:input path="name" /><form:errors path="name" /><br />
        Age        : <form:input path="age" /><form:errors path="age" /><br />
        Gender     : <form:input path="genderCode" /><form:errors path="genderCode" /><br />
        Birth Date : <form:input path="birthDate" /><form:errors path="birthDate" /><br />
    </form:form>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90
   
   * - Sr. No.
     - Description
   * - | (1)
     - Define taglib to use \ ``<form:form>``\  tag.
   * - | (2)
     - Specify form object stored in \ ``Model``\  in the ``modelAttribute`` attribute of \ ``<form:form>``\  tag.
   * - | (3)
     - Specify property name of form object in path attribute of \ ``<form:input>``\  tag.

|

.. _formobject_bindrequestparam-label:

Binding request parameters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
It is possible to bind the request parameters sent by HTML form to form object and pass it as an argument to the processing method of controller.

 .. code-block:: java
    :emphasize-lines: 3
    
    @RequestMapping("hello")
    public String hello(
            @Validated SampleForm form, // (1)
            BindingResult result,
            Model model) {
        if (result.hasErrors()) {
            return "sample/input";
        }
        // process form...
        return "sample/hello";
    }

 .. code-block:: java
    :emphasize-lines: 10
    
    @ModelAttribute("xxx")
    public SampleForm setUpSampleForm() {
        SampleForm form = new SampleForm();
        // populate form
        return form;
    }

    @RequestMapping("hello")
    public String hello(
            @ModelAttribute("xxx") @Validated SampleForm form, // (2)
            BindingResult result,
            Model model) {
        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90
   
   * - Sr. No.
     - Description
   * - | (1)
     - | Form object is passed as an argument to the processing method of controller after reflecting request parameters to the form object.
   * - | (2)
     - | When the attribute name is specified in ModelAttribute method, it is necessary to explicitly specify attribute name of form object as \ ``@ModelAttribute("xxx")``\ .

.. warning::

    When attribute name specified by ModelAttribute method and attribute name specified in the \ ``@ModelAttribute("xxx")``\  in the argument of processing method are different, 
    it should be noted that a new instance is created other than the instance created by ModelAttribute method.
    When attribute name is not specified with ``@ModelAttribute`` in the argument to processing method, the attribute name is deduced as the class name with first letter in lower case.

|

Determining binding result
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Error (including input validation error) that occurs while binding request parameter sent by HTML form to form object, is stored in \ ``org.springframework.validation.BindingResult``\ .

 .. code-block:: java
    :emphasize-lines: 4,6

    @RequestMapping("hello")
    public String hello(
            @Validated SampleForm form,
            BindingResult result, // (1)
            Model model) {
        if (result.hasErrors()) { // (2)
            return "sample/input";
        }
        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - When \ ``BindingResult``\  is declared immediately after form object, it is possible to refer to the error inside the processing method of controller.
   * - | (2)
     - Calling \ ``BindingResult.hasErrors()``\  can determine whether any error occurred in the input values of form object.

It is also possible to determine field errors, global errors (correlated check errors at class level) separately. These can be used separately if required.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.40\linewidth}|p{0.50\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 40 50

   * - Sr. No.
     - Method
     - Description
   * - 1.
     - ``hasGlobalErrors()``
     - Method to determine the existence of global errors.
   * - 2.
     - ``hasFieldErrors()``
     - Method to determine the existence of field errors.
   * - 3.
     - ``hasFieldErrors(String field)``
     - Method to determine the existence of errors related to specified field.

|

.. _view:

Implementing View
--------------------------------------------------------------------------------
View plays the following role.

#. | **View generates response (HTML) as per the requirements of the client.**
   | View retrieves the required data from model (form object or domain object) and generates response in the format which is required by the client for rendering.

|

Implementing JSP
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Implement View using JSP to generate response(HTML) as per the requirement of the client.
| Use the class provided by Spring Framework as the ``ViewResolver`` for calling JSP. Refer to :ref:`controller_method_return-html-label` for settings of ``ViewResolver``.

Basic implementation method of JSP is described below.

- :ref:`view_jsp_include-label`
- :ref:`view_jsp_out-label`
- :ref:`view_jsp_outnumber-label`
- :ref:`view_jsp_outdate-label`
- :ref:`view_jsp_form-label`
- :ref:`view_jsp_errors-label`
- :ref:`view_jsp_resultmessages-label`
- :ref:`view_jsp_codelist-label`
- :ref:`view_jsp_message-label`
- :ref:`view_jsp_if-label`
- :ref:`view_jsp_forEach-label`
- :ref:`view_jsp_pagination-label`
- :ref:`view_jsp_authorization-label`

In this chapter, usage of main JSP tag libraries are described. However, refer to respective documents for the detailed usage since all JSP tag libraries are not described here.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 30 60

   * - Sr. No.
     - JSP tag library name
     - Document
   * - 1.
     - Spring's form tag library
     - - `<http://docs.spring.io/spring/docs/4.1.9.RELEASE/spring-framework-reference/html/view.html#view-jsp-formtaglib>`_\
       - `<http://docs.spring.io/spring/docs/4.1.9.RELEASE/spring-framework-reference/html/spring-form.tld.html>`_\
   * - 2.
     - Spring's tag library
     - - `<http://docs.spring.io/spring/docs/4.1.9.RELEASE/spring-framework-reference/html/spring.tld.html>`_\
   * - 3.
     - JSTL
     - - `<http://download.oracle.com/otndocs/jcp/jstl-1.2-mrel2-eval-oth-JSpec/>`_\
   * - 4.
     - Common library's tags & el functions
     - - [:doc:`../Appendix/TagLibAndELFunctions`] of this guideline

 .. warning::

    If terasoluna-gfw-web 1.0.0.RELEASE is being used, \ ``action``\  tag must be always be specified while using \ ``<form:form>``\  tag of Spring's form tag library.
    
    terasoluna-gfw-web 1.0.0.RELEASE has a dependency on Spring MVC(3.2.4.RELEASE). In this version of Spring MVC, if \ ``action``\  attribute of \ ``<form:form>``\  tag is not specified, it will
    expose a vulnerability of XSS(Cross-site scripting).
    For further details regarding the vulnerability, refer to \ `CVE-2014-1904 of National Vulnerability Database (NVD)  <http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-1904>`_\ .

    Also, terasoluna-gfw-web 1.0.1.RELEASE have been upgraded to Spring MVC(3.2.10.RELEASE and above); hence this vulnerability is not present.


|

.. _view_jsp_include-label:

Creating common JSP for include
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Create a JSP that contains directive declaration which are required by all the JSP files of the project.
By specifying this JSP in ``<jsp-config>/<jsp-property-group>/<include-prelude>`` element of ``web.xml``, eliminates the need to declare these directives and each and every JSP file of the project.
Further, this file is provided in blank project also.

- include.jsp

 .. code-block:: jsp
    :emphasize-lines: 1,4,8

    <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%> <%-- (1) --%>
    <%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt"%>

    <%@ taglib uri="http://www.springframework.org/tags" prefix="spring"%> <%-- (2) --%>
    <%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>
    <%@ taglib uri="http://www.springframework.org/security/tags" prefix="sec"%>

    <%@ taglib uri="http://terasoluna.org/functions" prefix="f"%> <%-- (3) --%>
    <%@ taglib uri="http://terasoluna.org/tags" prefix="t"%>

- web.xml

 .. code-block:: xml
    :emphasize-lines: 7

    <jsp-config>
        <jsp-property-group>
            <url-pattern>*.jsp</url-pattern>
            <el-ignored>false</el-ignored>
            <page-encoding>UTF-8</page-encoding>
            <scripting-invalid>false</scripting-invalid>
            <include-prelude>/WEB-INF/views/common/include.jsp</include-prelude> <!-- (4) -->
        </jsp-property-group>
    </jsp-config>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | JSP tag libraries of JSTL are declared. In this example, ``core`` and ``fmt`` are used.
   * - | (2)
     - | JSP tag libraries of Spring Framework are declared. In this example, ``spring``, ``form`` and ``sec`` are used.
   * - | (3)
     - | JSP tag libraries provided by common library are declared.
   * - | (4)
     - | The contents specified in JSP to be included (\ ``/WEB-INF/views/common/include.jsp``\ ) are included at the beginning of each JSP (file specified in \ ``<url-pattern>``\ ).

 .. note::
 
   Refer to ``JSP.1.10 Directives`` of `JavaServer Pages Specification(Version2.2) <http://download.oracle.com/otndocs/jcp/jsp-2.2-mrel-eval-oth-JSpec/>`_\ for details of directives.
 
 .. note::
 
  Refer to ``JSP.3.3 JSP Property Groups`` of `JavaServer Pages Specification(Version2.2) <http://download.oracle.com/otndocs/jcp/jsp-2.2-mrel-eval-oth-JSpec/>`_\ for details of <jsp-property-group> element.

|

.. _view_jsp_out-label:

Displaying value stored in model
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
To display the value stored in ``Model`` (form object or domain object) in HTML, use EL expressions or JSP tag libraries provided by JSTL.

Display using EL expressions.

- SampleController.java

 .. code-block:: java
    :emphasize-lines: 3

    @RequestMapping("hello")
    public String hello(Model model) {
        model.addAttribute(new HelloBean("Bean Hello World!")); // (1)
        return "sample/hello"; // returns view name
    }

- hello.jsp

 .. code-block:: jsp
    :emphasize-lines: 1

    Message : ${f:h(helloBean.message)} <%-- (2) --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Add \ ``HelloBean``\  object to \ ``Model``\  object.
   * - | (2)
     - | In View(JSP), data added to the \ ``Model``\  object can be retrieved by describing ``${Attribute name.Property name of JavaBean}``.
       | In this example, HTML escaping is performed using ``${f:h(Attribute name.Property name of JavaBean)}`` function of EL expression.

 .. note::
    Since HTML escaping function (``f:h``) is provided in the common components, always use it if EL expressions are used to output values in HTML.
    For details of function of EL expression that perform HTML escaping, refer to :doc:`Cross Site Scripting <../Security/XSS>`.

Display using ``<c:out>`` tag provided by JSP tag library of JSTL.

 .. code-block:: jsp
    :emphasize-lines: 1

    Message : <c:out value="${helloBean.message}" /> <%-- (1) --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify the values fetched using EL expressions in ``value`` attribute of ``<c:out>`` tag. HTML escaping is also performed. 

 .. note::
    Refer to ``CHAPTER 4 General-Purpose Actions`` of `JavaServer Pages Standard Tag Library(Version 1.2) <http://download.oracle.com/otndocs/jcp/jstl-1.2-mrel2-eval-oth-JSpec/>`_\ for the details of ``<c:out>``.

|

.. _view_jsp_outnumber-label:

Displaying numbers stored in model
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Use JSP tag library provided by JSTL to output format number.

| Display using ``<fmt:formatNumber>`` tag provided by JSP tag library of JSTL. 

 .. code-block:: jsp
    :emphasize-lines: 1

    Number Item : <fmt:formatNumber value="${helloBean.numberItem}" pattern="0.00" /> <%-- (1) --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify the value acquired by EL expressions in  the value attribute of ``<fmt:formatNumber>`` tag. Specify the format to be displayed in pattern attribute. For example, "``0.00``" is specified .
       | When the value acquired in ``${helloBean.numberItem}`` is "``1.2``" temporarily, "``1.20``" is displayed on the screen.

.. note::
     Refer to ``CHAPTER 9 Formatting Actions`` of `JavaServer Pages Standard Tag Library(Version 1.2) <http://download.oracle.com/otndocs/jcp/jstl-1.2-mrel2-eval-oth-JSpec/>`_\ for the details of ``<fmt:formatNumber>``.

|

.. _view_jsp_outdate-label:

Displaying date and time stored in model
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Use JSP tag library provided by JSTL to output format date and time value.

Display using ``<fmt:formatDate>`` tag provided by JSP tag library of JSTL.

 .. code-block:: jsp
    :emphasize-lines: 1

    Date Item : <fmt:formatDate value="${helloBean.dateItem}" pattern="yyyy-MM-dd" /> <%-- (1) --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify the value fetched using EL expression in ``value`` attribute of ``<fmt:formatDate>`` tag. Specify the format to be displayed in ``pattern`` attribute. In this example, "``yyyy-MM-dd``" is specified. 
       | When value received for ``${helloBean.dateItem}`` is 2013-3-2, "``2013-03-02``" is displayed on the screen.

.. note::
    Refer to ``CHAPTER 9 Formatting Actions`` of `JavaServer Pages Standard Tag Library(Version 1.2) <http://download.oracle.com/otndocs/jcp/jstl-1.2-mrel2-eval-oth-JSpec/>`_\  for details of ``<fmt:formatDate>``.

.. note::
    JSP tag library provided by Joda Time should be used to use ``org.joda.time.DateTime`` as date and time object type.
    Refer to :doc:`../ArchitectureInDetail/Utilities/JodaTime`  for the details of Joda Time.

|

.. _view_jsp_form-label:

Binding form object to HTML form
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Use JSP tag library provided by Spring Framework to bind form object to HTML form and to display the values stored in form object.

Bind using ``<form:form>`` tag provided by Spring Framework.

 .. code-block:: jsp
    :emphasize-lines: 2-3

    <form:form action="${pageContext.request.contextPath}/sample/hello"
               modelAttribute="sampleForm"> <%-- (1) --%>
        Id : <form:input path="id" /> <%-- (2) --%>
    </form:form>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90
   
   * - Sr. No.
     - Description
   * - | (1)
     - Specify attribute name of form object stored in \ ``Model``\  in ``modelAttribute`` attribute of \ ``<form:form>``\  tag. 
   * - | (2)
     - Specify name of property to bind in the ``path`` attribute of \ ``<form:xxx>``\  tag. ``xxx`` part changes along with each input element.

.. note::
    For the details of \ ``<form:form>``\  , \ ``<form:xxx>``\  tag refer to `Using Spring's form tag library <http://docs.spring.io/spring/docs/4.1.9.RELEASE/spring-framework-reference/html/view.html#view-jsp-formtaglib>`_\ .

|

.. _view_jsp_errors-label:

Displaying input validation errors
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
To display the contents of input validation error, use JSP tag library provided by Spring Framework.

| Display using ``<form:errors>`` tag provided by Spring Framework.
| Refer to :doc:`../ArchitectureInDetail/Validation` for details.

 .. code-block:: jsp
    :emphasize-lines: 3

    <form:form action="${pageContext.request.contextPath}/sample/hello"
               modelAttribute="sampleForm">
        Id : <form:input path="id" /><form:errors path="id" /><%-- (1) --%>
    </form:form>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90
   
   * - Sr. No.
     - Description
   * - | (1)
     - | Specify name of the property to display the error in ``path`` attribute of \ ``<form:errors>``\  tag.

|

.. _view_jsp_resultmessages-label:

Displaying message of processing result
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
To display the message notifying the output of processing the request, use JSP tag library provided in common components.

| Use ``<t:messagesPanel>`` tag provided in common components.
| Refer to :doc:`../ArchitectureInDetail/MessageManagement`  for details.

 .. code-block:: jsp
    :emphasize-lines: 3

    <div class="messages">
        <h2>Message pattern</h2>
        <t:messagesPanel /> <%-- (1) --%>
    </div>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90
   
   * - Sr. No.
     - Description
   * - | (1)
     - | Messages stored with attribute name "``resultMessages``" are output.

|

.. _view_jsp_codelist-label:

Displaying codelist
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
To display the codelist (provided in common components), use JSP tag library provided by Spring Framework.

| Codelist can be referred from JSP in the same way as ``java.util.Map`` interface.
| Refer to :doc:`../ArchitectureInDetail/Codelist`  for details.

Display codelist in select box.

 .. code-block:: jsp
    :emphasize-lines: 3

    <form:select path="orderStatus">
        <form:option value="" label="--Select--" />
        <form:options items="${CL_ORDERSTATUS}" /> <%-- (1) --%>
    </form:select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90
   
   * - Sr. No.
     - Description
   * - | (1)
     - Codelist (``java.util.Map`` interface) is stored with name (``"CL_ORDERSTATUS"``) as attribute name.
       Therefore, in JSP, codelist (``java.util.Map`` interface) can be accessed using EL expression.
       Codelist can be displayed in select box by passing the object of ``Map`` interface to ``items`` attribute of ``<form:options>``.

Label part is displayed on the screen for the value selected in select box.

 .. code-block:: jsp

    Order Status : ${f:h(CL_ORDERSTATUS[orderForm.orderStatus])}

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90
   
   * - Sr. No.
     - Description
   * - | (1)
     - In the same way as in case of creating select box, Codelist (``java.util.Map`` interface) is stored with name (``"CL_ORDERSTATUS"``) as attribute name.
       If value selected in select box is specified as key of the fetched ``Map`` interface, it is possible to display the code name. 

|

.. _view_jsp_message-label:

Displaying fixed text content
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Strings for screen name, element name and guidance etc can be directly written in JSP when internationalization is not required.
| However, when internationalization is required, display the values acquired from property file using JSP tag library provided by Spring Framework.

| Display using ``<spring:message>`` tag provided by Spring Framework.
| Refer to :doc:`../ArchitectureInDetail/Internationalization`  for details.

- properties

 .. code-block:: properties
    :emphasize-lines: 1-2
    
    # (1)
    label.orderStatus=Order status

- jsp

 .. code-block:: jsp
    :emphasize-lines: 1

    <spring:message code="label.orderStatus" text="Order Status" /> : <%-- (2) --%>
        ${f:h(CL_ORDERSTATUS[orderForm.orderStatus])}

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90
   
   * - Sr. No.
     - Description
   * - | (1)
     - Define the string of label in properties file.
   * - | (2)
     - If key in the properties file is specified in ``code`` attribute of ``<spring:message>``, string corresponding to the key is displayed.
     
.. note::
     The value specified in ``text`` attribute is displayed when property value could not be acquired.

|

.. _view_jsp_if-label:

Switching display according to conditions
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
When display is to be switched according to some value in model, use JSP tag library provided by JSTL.

Switch display using ``<c:if>`` tag or ``<c:choose>`` provided by JSP tag library of JSTL.

Switching display by using ``<c:if>``.

 .. code-block:: jsp
    :emphasize-lines: 1

    <c:if test="${orderForm.orderStatus != 'complete'}"> <%-- (1) --%>
            <%-- ... --%>
    </c:if>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90
   
   * - Sr. No.
     - Description
   * - | (1)
     - Put the condition for entering the branch in ``test`` attribute of ``<c:if>``. In this example, when order status is not ``'complete'``, the contents of the branch will be displayed.

Switching display using ``<c:choose>``.

 .. code-block:: jsp
    :emphasize-lines: 2,8

    <c:choose>
        <c:when test="${customer.type == 'premium'}"> <%-- (1) --%>
            <%-- ... --%>
        </c:when>
        <c:when test="${customer.type == 'general'}">
            <%-- ... --%>
        </c:when>
        <c:otherwise> <%-- (2) --%>
            <%-- ... --%>
        </c:otherwise>
    </c:choose>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90
   
   * - Sr. No.
     - Description
   * - | (1)
     - Put the condition for entering the branch in ``test`` attribute of ``<c:when>``. In this example, when customer type is ``'premium'``, the contents of the branch will be displayed.
       When condition specified in ``test`` attribute is ``false``, next ``<c:when>`` tag is processed.
   * - | (2)
     - When result of ``test`` attribute of all ``<c:when>`` tags is ``false``, ``<c:otherwise>`` tag is evaluated.

.. note::
    Refer to ``CHAPTER 5 Conditional Actions`` of `JavaServer Pages Standard Tag Library(Version 1.2) <http://download.oracle.com/otndocs/jcp/jstl-1.2-mrel2-eval-oth-JSpec/>`_\ for details.

|

.. _view_jsp_forEach-label:

Repeated display of collection elements
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
To repeat display of collection stored in model, use JSP tag library provided by JSTL.

Repeated display can be done using ``<c:forEach>`` provided by JSP tag library of JSTL.


 .. code-block:: jsp
    :emphasize-lines: 6,8-9

    <table>
        <tr>
            <th>No</th>
            <th>Name</th>
        </tr>
        <c:forEach var="customer" items="${customers}" varStatus="status"> <%-- (1) --%>
            <tr>
                <td>${status.count}</td> <%-- (2) --%>
                <td>${f:h(customer.name)}</td> <%-- (3) --%>
            </tr>
        </c:forEach>
    </table>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90
   
   * - Sr. No.
     - Description
   * - | (1)
     - By specifying the collection object in ``items`` attribute of ``<c:forEach>`` tag, ``<c:forEach>`` tag is repeatedly executed to iterate over the collection.
       While iterating over the collection, if the current element is to be referred inside ``<c:forEach>`` tag, it can be done by specifying a variable name for the 
       current element in ``var`` attribute. 
   * - | (2)
     - By specifying a variable name in ``varStatus`` attribute,  current position (count) of iteration in ``<c:forEach>``  tag can be fetched.
       Refer to `JavaDoc <http://download.oracle.com/otndocs/jcp/jstl-1.2-mrel2-eval-oth-JSpec/>`_\ of ``javax.servlet.jsp.jstl.core.LoopTagStatus`` for attributes other than count.
   * - | (3)
     - This is the value acquired from the object stored in variable specified by ``var`` attribute of ``<c:forEach>`` tag.

.. note::
    Refer to ``CHAPTER 6 Iterator Actions`` of `JavaServer Pages Standard Tag Library(Version 1.2) <http://download.oracle.com/otndocs/jcp/jstl-1.2-mrel2-eval-oth-JSpec/>`_\ for details.

|

.. _view_jsp_pagination-label:

Displaying link for pagination
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
To display links of pagination on the screen while displaying the list, use JSP tag library provided in common components.

Display the link for pagination using ``<t:pagination>`` provided in common components.
Refer to :doc:`../ArchitectureInDetail/Pagination`  for details.


|

.. _view_jsp_authorization-label:

Switching display according to authority
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
To switch display according to authority of the user who has logged in, use JSP tag library provided by Spring Security.

Switch display using ``<sec:authorize>`` provided by Spring Security.
Refer to :doc:`../Security/Authorization`  for details.


|
|

Implementing JavaScript
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
When it is necessary to control screen items (controls of hide/display, activate/deactivate, etc.) after screen rendering, control items using JavaScript.

.. todo::

    **TBD**

    Details will be included in the coming versions.

|

Implementing style sheet
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| It is recommended to specify attribute values related to screen design in style sheet (css file) and not in JSP(HTML) directly.
| In JSP(HTML), specify ``id`` attribute to identify the items uniquely and ``class`` attribute which indicates classification of elements. 
| While, specify values related to actual location of elements or appearance should be specified in style sheet (css file).
| By configuring in this way, it is possible to reduce the design related aspects from the implementation of JSP.
| Simultaneously, if there is any change in design, only style sheet (css file) is modified and not JSP.

.. note::
    When form is created using ``<form:xxx>``, ``id`` attribute will be set automatically. Application developer should specify ``class`` attribute.

|

Implementing common logic
--------------------------------------------------------------------------------

|

Implementing common logic to be executed before and after calling controller
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Here, common logic indicates processes which are required to be executed before and after execution the controller.

|

Implementing Servlet Filter
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Common processes independent of Spring MVC are implemented using Servlet Filter.
| However, when common processes are to be executed only for the certain requests mapped with processing method of controller, 
| then it should be implemented using Handler Interceptor and not Servlet Filter.

| Samples of Servlet Filter are given below.
| In sample code, value is stored in ``MDC`` in order to output the log of IP address of client.

- java

 .. code-block:: java
    :emphasize-lines: 1

    public class ClientInfoPutFilter extends OncePerRequestFilter { // (1)

        private static final String ATTRIBUTE_NAME = "X-Forwarded-For";
        protected final void doFilterInternal(HttpServletRequest request,
                HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
            String remoteIp = request.getHeader(ATTRIBUTE_NAME);
            if (remoteIp == null) {
                remoteIp = request.getRemoteAddr();
            }
            MDC.put(ATTRIBUTE_NAME, remoteIp);
            try {
                filterChain.doFilter(request, response);
            } finally {
                MDC.remove(ATTRIBUTE_NAME);
            }
        }
    }

- web.xml

 .. code-block:: xml
    :emphasize-lines: 1,5

    <filter> <!-- (2) -->
        <filter-name>clientInfoPutFilter</filter-name>
        <filter-class>x.y.z.ClientInfoPutFilter</filter-class>
    </filter>
    <filter-mapping> <!-- (3) -->
        <filter-name>clientInfoPutFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90
   
   * - Sr. No.
     - Description
   * - | (1)
     - In sample, it is guaranteed that it is executed only once for similar requests by creating the Servlet Filter as subclass 
       of ``org.springframework.web.filter.OncePerRequestFilter`` provided by Spring Framework.
   * - | (2)
     - Register the created Servlet Filter in ``web.xml``.
   * - | (3)
     - Specify URL pattern to apply the registered Servlet Filter.


Servlet Filter can also be defined as Bean of Spring Framework.

- web.xml

 .. code-block:: xml
    :emphasize-lines: 3

    <filter>
        <filter-name>clientInfoPutFilter</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class> <!-- (1) -->
    </filter>
    <filter-mapping>
        <filter-name>clientInfoPutFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

- applicationContext.xml

 .. code-block:: xml
    :emphasize-lines: 1

    <bean id="clientInfoPutFilter" class="x.y.z.ClientInfoPutFilter" /> <!-- (2) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90
   
   * - Sr. No.
     - Description
   * - | (1)
     - In sample, process is delegated to Servlet Filter defined in step (2) by specifying ``org.springframework.web.filter.DelegatingFilterProxy`` 
       provided by Spring Framework in Servlet Filter class.
   * - | (2)
     - Add the Servlet Filter class to Bean definition file (``applicationContext.xml``).
       At this time, ``id`` attribute of bean definition should be assigned with the filter name (value specified in ``<filter-name>`` tag) specified in ``web.xml``.

|

Implementing HandlerInterceptor
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Common processes dependent on Spring MVC are implemented using HandlerInterceptor.
| HandlerInterceptor can execute common processes only for the requests which are allowed by the application since it is called after the determining the processing method 
| to which the request is to be mapped to.

HandlerInterceptor can execute the process keeping in mind the following 3 points.

- | Before executing processing method of controller
  | Implemented as ``HandlerInterceptor#preHandle`` method.
- | After successfully executing processing method of controller
  | Implemented as ``HandlerInterceptor#postHandle`` method.
- | After completion of processing method of controller (executed irrespective of Normal / Abnormal)
  | Implemented as ``HandlerInterceptor#afterCompletion`` method.

| Sample of HandlerInterceptor is given below.
| In sample code, log of info level is output after successfully executing controller process.

 .. code-block:: java
    :emphasize-lines: 1

    public class SuccessLoggingInterceptor extends HandlerInterceptorAdapter { // (1)

        private static final Logger logger = LoggerFactory
                .getLogger(SuccessLoggingInterceptor.class);

        @Override
        public void postHandle(HttpServletRequest request,
                HttpServletResponse response, Object handler,
                ModelAndView modelAndView) throws Exception {
            HandlerMethod handlerMethod = (HandlerMethod) handler;
            Method m = handlerMethod.getMethod();
            logger.info("[SUCCESS CONTROLLER] {}.{}", new Object[] {
                    m.getDeclaringClass().getSimpleName(), m.getName()});
        }

    }

- spring-mvc.xml

 .. code-block:: xml
    :emphasize-lines: 4-5,7

    <mvc:interceptors>
        <!-- ... -->
        <mvc:interceptor>
            <mvc:mapping path="/**" /> <!-- (2) -->
            <mvc:exclude-mapping path="/resources/**" /> <!-- (3) -->
            <mvc:exclude-mapping path="/**/*.html" />
            <bean class="x.y.z.SuccessLoggingInterceptor" /> <!-- (4) -->
        </mvc:interceptor>
        <!-- ... -->
    </mvc:interceptors>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90
   
   * - Sr. No.
     - Description
   * - | (1)
     - In sample, HandlerInterceptor is created as the subclass of ``org.springframework.web.servlet.handler.HandlerInterceptorAdapter`` provided by Spring Framework.
       Since ``HandlerInterceptorAdapter`` provides blank implementation of ``HandlerInterceptor`` interface, it is not required to implement unnecessary methods in the subclass.
   * - | (2)
     - Specify the pattern of path, where the created HandlerInterceptor is to be applied.
   * - | (3)
     - Specify the pattern of path, where the created HandlerInterceptor need not be applied.
   * - | (4)
     - Add the created HandlerInterceptor to ``<mvc:interceptors>`` tag of ``spring-mvc.xml``.

|

Implementing common processes of controller
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Here, common process indicates the process that should be commonly implemented in all the controllers.

.. _methodargumentresolver:

Implementing HandlerMethodArgumentResolver
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
When an object that is not supported by default in Spring Framework is to be passed as controller argument,
HandlerMethodArgumentResolver is to be implemented in order to enable controller to be able to receive the argument.

| Sample of HandlerMethodArgumentResolver is given below.
| In sample code, common request parameters are converted to JavaBean which are then passed to processing method of controller.


- JavaBean

 .. code-block:: java
    :emphasize-lines: 1

    public class CommonParameters implements Serializable { // (1)
    
        private String param1;
        private String param2;
        private String param3;
        
        // ....
    
    }


- HandlerMethodArgumentResolver

 .. code-block:: java
    :emphasize-lines: 2,6,13

    public class CommonParametersMethodArgumentResolver implements
                                                       HandlerMethodArgumentResolver { // (2)
    
        @Override
        public boolean supportsParameter(MethodParameter parameter) {
            return CommonParameters.class.equals(parameter.getParameterType()); // (3)
        }
    
        @Override
        public Object resolveArgument(MethodParameter parameter,
                ModelAndViewContainer mavContainer, NativeWebRequest webRequest,
                WebDataBinderFactory binderFactory) throws Exception {
            CommonParameters params = new CommonParameters(); // (4)
            params.setParam1(webRequest.getParameter("param1"));
            params.setParam2(webRequest.getParameter("param2"));
            params.setParam3(webRequest.getParameter("param3"));
            return params;
        }


- Controller

 .. code-block:: java
    :emphasize-lines: 2

    @RequestMapping(value = "home")
    public String home(CommonParameters commonParams) { // (5)
        logger.debug("param1 : {}",commonParams.getParam1());
        logger.debug("param2 : {}",commonParams.getParam2());
        logger.debug("param3 : {}",commonParams.getParam3());
        // ...
        return "sample/home";

    }

- spring-mvc.xml

 .. code-block:: xml
    :emphasize-lines: 4

    <mvc:annotation-driven>
        <mvc:argument-resolvers>
            <!-- ... -->
            <bean class="x.y.z.CommonParametersMethodArgumentResolver" /> <!-- (6) -->
            <!-- ... -->
        </mvc:argument-resolvers>
    </mvc:annotation-driven>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90
   
   * - Sr. No.
     - Description
   * - | (1)
     - JavaBean that retains common parameters.
   * - | (2)
     - Implement ``org.springframework.web.method.support.HandlerMethodArgumentResolver`` interface.
   * - | (3)
     - Determine parameter type. For example, when type of JavaBean that retains common parameters is specified as argument of controller, 
       resolveArgument method of this class is called.
   * - | (4)
     - Fetch the values of request parameters, set the parameters and return the JavaBean that retains the value of common parameters.
   * - | (5)
     - | Specify JavaBean that retains common parameters in the argument of processing method of controller.
       | Object returned in step (4) is passed.
   * - | (6)
     - Add the created HandlerMethodArgumentResolver to ``<mvc:argument-resolvers>`` tag of ``spring-mvc.xml``.

.. note::
    When parameters are to be passed commonly to processing methods of all controllers, it is effective to convert the parameters to JavaBean using HandlerMethodArgumentResolver.
    Parameters referred here are not restricted to request parameters.

|

.. _application_layer_controller_advice:

Implementing \"@ControllerAdvice"\ 
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
In a class with \ ``@ControllerAdvice``\  annotation,
implement common processes which are to be executed in multiple Controllers.

When a class with \ ``@ControllerAdvice``\  annotation is created, processes implemented using

- method with ``@InitBinder``
- method with ``@ExceptionHandler``
- method with ``@ModelAttribute`` 

can be applied to multiple Controllers.

.. tip::

    \ ``@ControllerAdvice``\  annotation is a mechanism added from Spring Framework 3.2;
    however, since the processing was applied to all Controllers, it could only implement common processes of entire application.

    From Spring Framework 4.0, it has been improved in such a way that controller can be specified flexibly for applying common processes.
    With this improvement, it is possible to implement a common process in various granularities.

|

.. _application_layer_controller_advice_attribute:

Methods to specify Controller (methods to specify attributes) for applying common processes are described below.

.. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 15 75

   * - Sr. No.
     - Attribute
     - Description and example of specification
   * - | (1)
     - ``annotations``
     - Specify annotation.

       Common processes are applied for the Controllers with specified annotations.
       An example of specifications is given below.

       .. code-block:: java

           @ControllerAdvice(annotations = LoginFormModelAttributeSetter.LoginFormModelAttribute.class)
           public class LoginFormModelAttributeSetter {
               @Target(ElementType.TYPE)
               @Retention(RetentionPolicy.RUNTIME)
               public static @interface LoginFormModelAttribute {}
               // ...
           }

       .. code-block:: java

           @LoginFormModelAttribute
           @Controller
           public class WelcomeController {
               // ...
           }

       .. code-block:: java

           @LoginFormModelAttribute
           @Controller
           public class LoginController {
               // ...
           }

       In the above example, \ ``@LoginFormModelAttribute``\  annotation is assigned to \ ``WelcomeController``\  and \ ``LoginController``\ ,
       hence, common processes are applied to \ ``WelcomeController``\  and \ ``LoginController``\ .
   * - | (2)
     - ``assignableTypes``
     - Specify a class or an instance.

       Common processes are applied for the Controllers which can be assigned (can be casted) to the specified class or interface.
       When using this attribute, it is recommended to adopt a style that specifies a marker interface to indicate that it is a Controller using common process, in attribute value.
       When this style is adopted, only the marker interface for common processes to be used needs to be implemented at Controller side.
       An example of specifications is given below.

       .. code-block:: java

           @ControllerAdvice(assignableTypes = ISODateInitBinder.ISODateApplicable.class)
           public class ISODateInitBinder {
               public static interface ISODateApplicable {}
               // ...
           }

       .. code-block:: java

           @Controller
           public class SampleController implements ISODateApplicable {
               // ...
           }

       In the above example, \ ``SampleController``\  implements \ ``@ISODateApplicable``\  interface (Marker interface),
       hence, common processes are applied to \ ``SampleController``\ .
   * - | (3)
     - ``basePackageClasses``
     - Specify a class or an interface.

       Common processes are applied for the Controllers under the package of specified class or interface.

       When using this attribute, it is recommended to adopt a style that specifies,

       * a class with \ ``@ControllerAdvice``\ 
       * a marker interface to identify the package

       in attribute value.
       An example to specify the same is given below.

       .. code-block:: java

           package com.example.app

           @ControllerAdvice(basePackageClasses = AppGlobalExceptionHandler.class)
           public class AppGlobalExceptionHandler {
               // ...
           }

       .. code-block:: java

           package com.example.app.sample

           @Controller
           public class SampleController {
               // ...
           }

       In the above example, \ ``SampleController``\  is stored under the package (\ ``com.example.app``\ ) in which (\ ``AppGlobalExceptionHandler``\ ) class with \ ``@ControllerAdvice``\ is stored.
       Hence, common processes are applied to \ ``SampleController``\ .

       .. code-block:: java

           package com.example.app.common

           @ControllerAdvice(basePackageClasses = AppPackage.class)
           public class AppGlobalExceptionHandler {
               // ...
           }

       .. code-block:: java

           package com.example.app

           public interface AppPackage {
           }

       When package level of class with \ ``@ControllerAdvice``\  is different than the class where Controller is stored, or when common processes are to be applied to multiple base packages,
       it is better to create a marker interface to identify the packages.
   * - | (4)
     - ``basePackages``
     - Specify the package name.

       Common processes are applied for the Controllers under the specified package.
       An example of specifying the same is given below.

       .. code-block:: java

           @ControllerAdvice(basePackages = "com.example.app")
           public class AppGlobalExceptionHandler {
               // ...
           }
   * - | (5)
     - ``value``
     - Alias to \ ``basePackages``\ .

       The operation is same as when \ ``basePackages``\  attribute is specified.
       An example of specifying the same is given below.

       .. code-block:: java

           @ControllerAdvice("com.example.app")
           public class AppGlobalExceptionHandler {
               // ...
           }

.. tip::

    \ ``basePackageClasses``\  attribute / \ ``basePackages``\  attribute / \ ``value``\  attribute are the attributes
    to specify base package that stores the Controller for applying common processes. However,
    when \ ``basePackageClasses``\  attribute is used, 


    * It is possible to prevent specifying the package that does not exist.
    * It is possible to get linked and change the package name on IDE.

    Therefore, it is known as Type-safe specification method.

|

| Implementation sample of \ ``@InitBinder``\  method is given below.
| In sample code, date that can be specified in request parameter is set to ``"yyyy/MM/dd"`` .

 .. code-block:: java
    :emphasize-lines: 1,2,5-6

    @ControllerAdvice // (1)
    @Order(0) // (2)
    public class SampleControllerAdvice {
    
        // (3)
        @InitBinder
        public void initBinder(WebDataBinder binder) {
            SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy/MM/dd");
            dateFormat.setLenient(false);
            binder.registerCustomEditor(Date.class,
                    new CustomDateEditor(dateFormat, true));
        }
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90
   
   * - Sr. No.
     - Description
   * - | (1)
     - It indicates that it is Bean of ControllerAdvice by assigning the \ ``@ControllerAdvice``\  annotation.
   * - | (2)
     - Specify priority for common processes by assigning the \ ``@Order``\  annotation. It should be specified when multiple ControllerAdvice have ordering for example there are dependencies between them. Otherwise it is not necessary.
   * - | (3)
     - Implement \ ``@InitBinder``\  method. \ ``@InitBinder``\  method is applied to all Controllers.

|

| Implementation sample of \ ``@ExceptionHandler``\  method is given below.
| In sample code, View of lock error screen is returned by handling ``org.springframework.dao.PessimisticLockingFailureException``.

 .. code-block:: java
    :emphasize-lines: 1-2

    // (1)
    @ExceptionHandler(PessimisticLockingFailureException.class)
    public String handlePessimisticLockingFailureException(
            PessimisticLockingFailureException e) {
        return "error/lockError";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90
   
   * - Sr. No.
     - Description
   * - | (1)
     - Implement \ ``@ExceptionHandler``\  method. \ ``@ExceptionHandler``\  method is applied to all Controllers.

|

| Implementation sample of \ ``@ModelAttribute``\  method is given below.
| In sample code, common request parameters are converted to JavaBean and stored in ``Model``.

- ControllerAdvice

 .. code-block:: java
    :emphasize-lines: 1-2

    // (1)
    @ModelAttribute
    public CommonParameters setUpCommonParameters(
            @RequestParam(value = "param1", defaultValue="def1") String param1,
            @RequestParam(value = "param2", defaultValue="def2") String param2,
            @RequestParam(value = "param3", defaultValue="def3") String param3) {
        CommonParameters params = new CommonParameters();
        params.setParam1(param1);
        params.setParam2(param2);
        params.setParam3(param3);
        return params;
    }
    
- Controller

 .. code-block:: java
    :emphasize-lines: 2

    @RequestMapping(value = "home")
    public String home(@ModelAttribute CommonParameters commonParams) { // (2)
        logger.debug("param1 : {}",commonParams.getParam1());
        logger.debug("param2 : {}",commonParams.getParam2());
        logger.debug("param3 : {}",commonParams.getParam3());
        // ...
        return "sample/home";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table:: 
   :header-rows: 1
   :widths: 10 90
   
   * - Sr. No.
     - Description
   * - | (1)
     - Implement \ ``@ModelAttribute``\  method. \ ``@ModelAttribute``\  method is applied to all Controllers.
   * - | (2)
     - Object created by \ ``@ModelAttribute`` \ method is passed.


|

Prevention of double submission
--------------------------------------------------------------------------------
Measures should be taken to prevent double submission as the same process gets executed multiple times by clicking Send button multiple times or 
refreshing (refresh using F5 button) the Finish screen.

For the problems occurring when countermeasures are not taken and details of implementation method, refer to :doc:`../ArchitectureInDetail/DoubleSubmitProtection` .

|

Usage of session
--------------------------------------------------------------------------------
| In the default operations of Spring MVC, model (form object, domain object etc.) is not stored in session.
| When it is to be stored in session, it is necessary to assign  \ ``@SessionAttributes``\  annotation to the controller class.
| When input forms are split on multiple screens, usage of \ ``@SessionAttributes``\  annotation should be studied since model (form object, domain object etc.) 
| can be shared between multiple requests for executing a series of screen transitions.
| However, whether to use \ ``@SessionAttributes``\  annotation should be determined after confirming the warning signs of using the session.

For details of session usage policy and implementation at the time of session usage, refer to :doc:`../ArchitectureInDetail/SessionManagement` .


.. raw:: latex

   \newpage
