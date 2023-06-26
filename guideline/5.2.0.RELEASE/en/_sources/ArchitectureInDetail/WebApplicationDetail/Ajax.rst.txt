Ajax
================================================================================

.. only:: html

 .. contents:: Table of contents
    :depth: 3
    :local:

Overview
--------------------------------------------------------------------------------

This section explains how to implement applications that use Ajax.

 .. todo::
    
    **TBD**

        Details regarding client side implementation etc. will follow in subsequent versions.

Ajax is the generic term used for group of techniques that perform the following asynchronous processes.

* Screen operations performed on the browser
* HTTP communication with the server triggered by a screen operation and reflecting back the communication results to the user interface

| Ajax is often used to improve usability. This is because, screen operations can be continued during HTTP communication.
| Typical examples of Ajax are (a) Providing suggestions while searching words and (b) Real time search of a search site.

|

.. _ajax_how_to_use:

How to use
--------------------------------------------------------------------------------

Application settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Application settings for Ajax are explained below.

.. warning:: **DoS attack measures at the time of StAX(Streaming API for XML) use**

    If the StAX is used to parse the XML format data, protect DoS attack.
    For details, refer to \ `CVE-2015-3192 - DoS Attack with XML Input <http://pivotal.io/security/cve-2015-3192>`_\ .


Settings to enable the Ajax functionality in Spring MVC
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Content-Type (such as ``"application/xml"``, ``"application/json"`` etc.) used in Ajax communication is set such that it can be handled by the handler method of Controller.

- :file:`spring-mvc.xml`

 .. code-block:: xml

    <mvc:annotation-driven /> <!-- (1) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | Sr. No.
     - | Description
   * - | (1)
     - | If ``<mvc:annotation-driven>`` element is specified, the functionality required for Ajax communication is enabled.
       | Therefore, special settings are not necessary for Ajax communication.

 .. note::
 
    Functionalities required for Ajax communication specifically refer to the ones provided by ``org.springframework.http.converter.HttpMessageConverter`` class.

    ``HttpMessageConverter`` performs the following roles.

    * Creating Java object from data stored in the request body.
    * Creating the data to be written to the response Body from Java object.

|

The ``HttpMessageConverter`` which is enabled by default on specifying ``<mvc:annotation-driven>``, is as follows.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.15\linewidth}|p{0.45\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 30 15 45

   * - | Sr. No.
     - | Class name
     - | Target
       | Format
     - | Description
   * - 1.
     - | org.springframework.http.converter.json.
       | MappingJackson2HttpMessageConverter
     - | JSON
     - | ``HttpMessageConverter`` to handle JSON as request body or response body.
       | `Jackson <https://github.com/FasterXML/jackson/>`_ system is included in the blank project. Hence, it can be used in its default state.
   * - 2.
     - | org.springframework.http.converter.xml.
       | Jaxb2RootElementHttpMessageConverter
     - | XML
     - | ``HttpMessageConverter`` to handle XML as request body or response body.
       | JAXB2.0 is included  as standard from JavaSE6. Hence it can be used in its default state.

 .. note::

    **Notice If you change from jackson version 1.xx to jackson version 2.xx**, refer to \ :ref:`here <REST_note_changed_jackson_version>`\ .

 .. warning:: **XXE (XML External Entity) Injection measures**

    When handling XML format data in Ajax communication, it is necessary to implement \ `XXE(XML External Entity) Injection <https://www.owasp.org/index.php/XML_External_Entity_(XXE)_Processing>`_\  measure.
    Subsequent versions above terasoluna-gfw-web 1.0.1.RELEASE are Spring MVC (above 3.2.10.RELEASE) version dependent. As these Spring MVC versions implement XXE Injection measures, it is not necessary to implement them independently.
    
    When using terasoluna-gfw-web 1.0.0.RELEASE, since it is dependent on the Spring MVC version (3.2.4.RELEASE) that does not implement XXE Injection, a class provided by Spring-oxm should be used.
    
    - :file:`spring-mvc.xml`
    
     .. code-block:: xml
    
        <!-- (1) -->
        <bean id="xmlMarshaller" class="org.springframework.oxm.jaxb.Jaxb2Marshaller">
            <property name="packagesToScan" value="com.examples.app" /> <!-- (2) -->
        </bean>
    
        <!-- ... -->
    
        <mvc:annotation-driven>
    
            <mvc:message-converters>
                <!-- (3) -->
                <bean class="org.springframework.http.converter.xml.MarshallingHttpMessageConverter">
                    <property name="marshaller" ref="xmlMarshaller" /> <!-- (4) -->
                    <property name="unmarshaller" ref="xmlMarshaller" /> <!-- (5) -->
                </bean>
            </mvc:message-converters>
    
            <!-- ... -->
    
        </mvc:annotation-driven>
    
        <!-- ... -->
    
     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - | Sr. No.
         - | Description
       * - | (1)
         - | Perform the bean definition of ``Jaxb2Marshaller`` provided by Spring-oxm.
           | ``Jaxb2Marshaller`` implements the XXE Injection measures in default state.
       * - | (2)
         - | Specify the package name where the JAXB JavaBean (JavaBean assigned with ``javax.xml.bind.annotation.XmlRootElement``  annotation ) is stored in the ``packagesToScan``  property.
           | JAXB JavaBean stored under the specified package is scanned and registered for marshalling or unmarshalling the JavaBean.
           | It is scanned in the same way as the base-package attribute of ``<context:component-scan>``.
       * - | (3)
         - | Add bean definition of ``MarshallingHttpMessageConverter`` to the ``<mvc:message-converters>`` element that is the child element of ``<mvc:annotation-driven>``.
       * - | (4)
         - | Specify the bean of ``Jaxb2Marshaller`` defined in (1) in ``marshaller`` property.
       * - | (5)
         - | Specify the bean of ``Jaxb2Marshaller`` defined in (1) in ``unmarshaller`` property.
         
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
         - | Add Spring-oxm as dependent artifact.
       * - | (2)
         - | Spring version should be fetched from the placeholder (${org.springframework-version}) that controls the Spring version number defined in :file:`pom.xml` of terasoluna-gfw-parent.



|

Implementing Controller
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Prerequisites for the sample code explained hereafter, are as follows.

* Response data should be in JSON format.
* JQuery should be used at client side. It should be the latest version of 1.x series (1.10.2), which is used while writing this document.

.. warning:: **Measures to circular reference**

    When you serialize a JavaBean in JSON or XML format using \ ``HttpMessageConverter``\  and
    if property holds an object of cross reference relationship,
    the \ ``StackOverflowError``\  and \ ``OutOfMemoryError``\  occur due to circular reference, hence it is necessary to exercise caution.

    In order to avoid a circular reference,

    * \ ``@com.fasterxml.jackson.annotation.JsonIgnore``\  annotation to exclude the property from serialization in case of serialized in JSON format using the Jackson
    * \ ``@javax.xml.bind.annotation.XmlTransient``\  annotation to exclude the property from serialization in case of serialized in XML format using the JAXB

    can be added.

    Below is the example of how to exclude specific field from serialization while serializing in JSON format using the Jackson.

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
           - The \ ``@JsonIgnore``\ annotation is added to exclude the property from serialization.

|

Fetching data
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
How to fetch data using Ajax is explained here.

Following example serves as the Ajax communication that returns a list matching with the search word.

- JavaBean for receiving request data

 .. code-block:: java

    // (1)
    public class SearchCriteria implements Serializable {

        // omitted

        private String freeWord; // (2)

        // omitted setter/getter

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | Sr. No.
     - | Description
   * - | (1)
     - | Create the JavaBean that receives request data.
   * - | (2)
     - | Match property name with parameter name of request parameter.

|

- JavaBean for storing the data to be returned

 .. code-block:: java

    // (3)
    public class SearchResult implements Serializable {

        // omitted

        private List<XxxEntity> list;

        // omitted setter/getter

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | Sr. No.
     - | Description
   * - | (3)
     - | Create the JavaBean for storing the data to be returned.

|

- Controller

 .. code-block:: java

    @RequestMapping(value = "search", method = RequestMethod.GET) // (4)
    @ResponseBody // (5)
    public SearchResult search(@Validated SearchCriteria criteria) { // (6)

        SearchResult searchResult = new SearchResult(); // (7)

        // (8)
        // omitted

        return searchResult; // (9)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | Sr. No.
     - | Description
   * - | (4)
     - | Specify ``RequestMethod.GET`` in the method attribute of ``@RequestMapping`` annotation.
   * - | (5)
     - | Assign ``@org.springframework.web.bind.annotation.ResponseBody`` annotation.
       | By assigning this annotation, the returned object is marshalled in JSON format and set in response body.
   * - | (6)
     - | Specify the JavaBean that receives request data, as an argument.
       | If input validation is required, specify ``@Validated``. For error handling of input validation, refer to ":ref:`ajax_how_to_use_input_error`".
       | For details on input validation, refer to ":doc:`../WebApplicationDetail/Validation`".
   * - | (7)
     - | Create the JavaBean object to store the data to be returned.
   * - | (8)
     - | Search data and store the search result in the object created in (7).
       | In the above example, implementation is omitted.
   * - | (9)
     - | Return the object to be marshalled in response body.

|

- HTML(JSP)

 .. code-block:: jsp

    <!-- omitted -->

    <meta name="contextPath" content="${pageContext.request.contextPath}" />

    <!-- omitted -->

    <!-- (10)  -->
    <form id="searchForm">
      <input name="freeWord" type="text">
      <button onclick="return searchByFreeWord()">Search</button>
    </form>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | Sr. No.
     - | Description
   * - | (10)
     - | Form to enter the search condition.
       | In the above example, it has a text box to enter the search condition and a search button.

 .. code-block:: jsp

    <!-- (11) -->
    <script type="text/javascript"
        src="${pageContext.request.contextPath}/resources/vendor/jquery/jquery-1.10.2.js">
    </script>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | Sr. No.
     - | Description
   * - | (11)
     - | Read the JQuery JavaScript file.
       | In the above example, request is sent to the ``/resources/vendor/jquery/jquery-1.10.2.js`` path, to read the JQuery JavaScript file.
     

 .. note::
 
    Refer to the settings below to read JQuery JavaScript file.
    Setting values provided in the blank project are as follows.
    
    * :file:`spring-mvc.xml`
    
     .. code-block:: xml

        <!-- (12) -->
        <mvc:resources mapping="/resources/**"
            location="/resources/,classpath:META-INF/resources/"
            cache-period="#{60 * 60}" />
    
     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - | Sr. No.
         - | Description
       * - | (12)
         - | Settings for releasing resource files (JavaScript files, Stylesheet files, image files etc.).
           | In the above setting example, when there is a request for path starting with ``/resources/``, the files in ``/resources/`` directory of war file or the ``/META-INF/resources/`` directory of class path are sent as a response.

    |
           
    In the above settings, the JQuery JavaScript file needs to be placed under any one of the following paths.
    
    * | ``/resources/vendor/jquery/jquery-1.10.2.js`` in war file
      | It is ``src/main/webapp/resources/vendor/jquery/jquery-1.10.2.js`` when indicated by the path in the project.
    * | ``/META-INF/resources/vendor/jquery/jquery-1.10.2.js`` in class path
      | It is ``src/main/resources/META-INF/resources/vendor/jquery/jquery-1.10.2.js`` when indicated by the path in the project.
    
|
    
- JavaScript

 .. code-block:: javascript

    var contextPath = $("meta[name='contextPath']").attr("content");

    // (13)
    function searchByFreeWord() {
        $.ajax(contextPath + "/ajax/search", {
            type : "GET",
            data : $("#searchForm").serialize(),
            dataType : "json", // (14)

        }).done(function(json) {
            // (15)
            // render search result
            // omitted

        }).fail(function(xhr) {
            // (16)
            // render error message
            // omitted

        });
        return false;
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | Sr. No.
     - | Description
   * - | (13)
     - | Ajax function that converts search criteria specified in the form to request parameter and sends the request for `/ajax/search` using GET method.
       | In the above example, clicking the button acts as the trigger for Ajax communication. However, by setting key down or key up of text box as the trigger, real time search can be performed.
   * - | (14)
     - | Specify the data format to be received as a response.
       | In the above example, as ``"json"`` is specified, ``"application/json"`` is set in Accept header.
   * - | (15)
     - | Implement the process when Ajax communication ends normally (when Http status code is ``"200"``).
       | In the above example, implementation is omitted.
   * - | (16)
     - | Implement the process when Ajax communication does not end normally (when Http status code is ``"4xx"`` and ``"5xx"``).
       | In the above example, implementation is omitted.
       | For error process implementation, refer to :ref:`ajax_post_formdata`.

 .. tip::

    In the above example, by setting context path (``${pageContext.request.contextPath}`` ) of Web application in HTML``<meta>`` element.
    JSP code is deleted from JavaScript code.

|

| Communication is as follows when "Search" button of Search form is clicked.
| Main points are highlighted.

- Request data

 .. code-block:: guess
    :emphasize-lines: 1,4

    GET /terasoluna-gfw-web-blank/ajax/search?freeWord= HTTP/1.1
    Host: localhost:9999
    Connection: keep-alive
    Accept: application/json, text/javascript, */*; q=0.01
    X-Requested-With: XMLHttpRequest
    User-Agent: Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/30.0.1599.101 Safari/537.36
    Referer: http://localhost:9999/terasoluna-gfw-web-blank/ajax/xxe
    Accept-Encoding: gzip,deflate,sdch
    Accept-Language: en-US,en;q=0.8,ja;q=0.6
    Cookie: JSESSIONID=3A486604D7DEE62032BA6C073FC6BE9F

|

- Response data

 .. code-block:: guess
    :emphasize-lines: 4, 8

    HTTP/1.1 200 OK
    Server: Apache-Coyote/1.1
    X-Track: a8fb8fefaaf64ee2bffc2b0f77050226
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Fri, 25 Oct 2013 13:52:55 GMT

    {"list":[]}

|

.. _ajax_post_formdata:

Posting form data
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
How to post form data and fetch processing result using Ajax, is explained here.

Following example is about the Ajax communication of receiving two numbers and returning the calculation result.

- JavaBean to receive form data

 .. code-block:: java

    // (1)
    public class CalculationParameters implements Serializable {

        // omitted

        private Integer number1;

        private Integer number2;

        // omitted setter/getter

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | Sr. No.
     - | Description
   * - | (1)
     - | Create the JavaBean for receiving form data.

|

- JavaBean that stores processing result

 .. code-block:: java

    // (2)
    public class CalculationResult implements Serializable {

        // omitted

        private int resultNumber;

        // omitted setter/getter

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | Sr. No.
     - | Description
   * - | (2)
     - | Create the JavaBean that stores processing result.

|

- Controller

 .. code-block:: java

    @RequestMapping("xxx")
    @Controller
    public class XxxController {

        @RequestMapping(value = "plusForForm", method = RequestMethod.POST) // (3)
        @ResponseBody
        public CalculationResult plusForForm(
            @Validated CalculationParameters params) { // (4)
            CalculationResult result = new CalculationResult();
            int sum = params.getNumber1() + params.getNumber2();
            result.setResultNumber(sum); // (5)
            return result; // (6)
        }
        
        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | Sr. No.
     - | Description
   * - | (3)
     - | Specify ``RequestMethod.POST`` in the method attribute of ``@RequestMapping`` annotation.
   * - | (4)
     - | Specify the JavaBean for receiving form data as an argument.
       | Specify ``@Validated``  when input validation is required. For handling input validation errors, refer to ":ref:`ajax_how_to_use_input_error`".
       | For details on input validation, refer to ":doc:`../WebApplicationDetail/Validation`".
   * - | (5)
     - | Store the processing result in the object created for the same.
       | In the above example, calculation result of the two numbers fetched from form object, is stored.
   * - | (6)
     - | Return the object to perform marshalling in response body.

|

- HTML (JSP)

 .. code-block:: jsp

    <!-- omitted -->

    <meta name="contextPath" content="${pageContext.request.contextPath}" />

    <sec:csrfMetaTags />

    <!-- omitted -->

    <!-- (7)  -->
    <form id="calculationForm">
        <input name="number1" type="text">+
        <input name="number2" type="text">
        <button onclick="return plus()">=</button>
        <span id="calculationResult"></span> <!-- (8) -->
    </form>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | Sr. No.
     - | Description
   * - | (7)
     - | Form to enter the numerical value to be calculated.
   * - | (8)
     - | Area to display calculation result.
       | In the above example, calculation result is displayed when communication is successful and it is cleared when the communication fails.

|

- JavaScript

 .. code-block:: javascript

    var contextPath = $("meta[name='contextPath']").attr("content");

    // (9)
    var csrfToken = $("meta[name='_csrf']").attr("content");
    var csrfHeaderName = $("meta[name='_csrf_header']").attr("content");
    $(document).ajaxSend(function(event, xhr, options) {
        xhr.setRequestHeader(csrfHeaderName, csrfToken);
    });

    // (10)
    function plus() {
        $.ajax(contextPath + "/ajax/plusForForm", {
            type : "POST",
            data : $("#calculationForm").serialize(),
            dataType : "json"
        }).done(function(json) {
            $("#calculationResult").text(json.resultNumber);

        }).fail(function(xhr) {
            // (11)
            var messages = "";
            // (12)
            if(400 <= xhr.status && xhr.status <= 499){
                // (13)
                var contentType = xhr.getResponseHeader('Content-Type');
                if (contentType != null && contentType.indexOf("json") != -1) {
                    // (14)
                    json = $.parseJSON(xhr.responseText);
                    $(json.errorResults).each(function(i, errorResult) {
                        messages += ("<div>" + errorResult.message + "</div>");
                    });
                } else {
                    // (15)
                    messages = ("<div>" + xhr.statusText + "</div>");
                }
            }else{
                // (16)
                messages = ("<div>" + "System error occurred." + "</div>");
            }
            // (17)
            $("#calculationResult").html(messages);
        });

        return false;
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | Sr. No.
     - | Description
   * - | (9)
     - | To send the request using POST method, CSRF token needs to be set to HTTP header.
       | In the above example, the header name and token value are set in the ``<meta>`` element of HTML and value is fetched by JavaScript.
       | For details on CSRF measures, refer to :doc:`../../Security/CSRF`.
   * - | (10)
     - | Ajax function that converts the numerical value specified in form, to request parameter and sends the request for ``/ajax/plusForForm`` using POST method.
       | In the above example, clicking the button acts as the trigger for Ajax communication however, real time calculation can be implemented by setting lost focus of the text box as the trigger.
   * - | (11)
     - | Implementation of error handling is shown below.
       | For server side implementation of error handling, refer to :ref:`ajax_how_to_use_input_error`.
   * - | (12)
     - | Determine the HTTP status code and type of error.
       | HTTP status code is stored in the ``status`` field of XMLHttpRequest object.
   * - | (13)
     - | Check whether the response data is in JSON format.
       | In the above example, response data format is checked by referring to the value set in the Content-Type of response header.
       | If the format is not checked and if it a format other than JSON, an error occurs while deserializing to JSON object.
       | If error handling is performed easily at the server side, page may be returned in HTML format.
   * - | (14)
     - | Deserialize the response data in JSON object.
       | Response data is stored in the ``responseText`` field of XMLHttpRequest object.
       | In the above example, error information is fetched from the deserialized JSON object and error message is created.
   * - | (15)
     - | Perform the process when the response data is not in JSON format.
       | In the above example, HTTP status text is stored in the error message.
       | HTTP status text is stored in the ``statusText``  field of XMLHttpRequest object.
   * - | (16)
     - | Perform the process when there is a server error.
       | In the above example, message notifying it as a system error is stored in error message.
   * - | (17)
     - | Perform rendering process when there is an error.
       | In the above example, error message is displayed in the area for displaying calculation result.

 .. warning::
 
    In the above example, processes namely, Ajax communication, DOM operation (rendering) and error handling are performed by the same function. It is recommended to split and implement these processes.

 .. todo:: **TBD**
    
    Implementation at client side will be explained in detail, in subsequent versions.

 .. tip::

    In the above example, JSP code is deleted from JavaScript code by setting CSRF token value and CSRF token header name,
    in the ``<meta>`` element of HTML using \ ``<sec:csrfMetaTags />``\ . Please refer, \ :ref:`csrf_ajax-token-setting`\ .

    Please note that, CSRF token value and name of CSRF token header can also be fetched by using  \ ``${_csrf.token}``\  and  \ ``${_csrf.headerName}``\  respectively.

|

| Following communication occurs when the "=" button of search form is clicked.
| Main points are highlighted.

- Request data

 .. code-block:: guess
    :emphasize-lines: 1,5,7,10,16

    POST /terasoluna-gfw-web-blank/ajax/plusForForm HTTP/1.1
    Host: localhost:9999
    Connection: keep-alive
    Content-Length: 19
    Accept: application/json, text/javascript, */*; q=0.01
    Origin: http://localhost:9999
    X-CSRF-TOKEN: a5dd1858-8a4f-4ecc-88bd-a326388ab5c9
    X-Requested-With: XMLHttpRequest
    User-Agent: Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/30.0.1599.101 Safari/537.36
    Content-Type: application/x-www-form-urlencoded; charset=UTF-8
    Referer: http://localhost:9999/terasoluna-gfw-web-blank/ajax/xxe
    Accept-Encoding: gzip,deflate,sdch
    Accept-Language: en-US,en;q=0.8,ja;q=0.6
    Cookie: JSESSIONID=3A486604D7DEE62032BA6C073FC6BE9F

    number1=1&number2=2

|

- Response data

 .. code-block:: guess
    :emphasize-lines: 4, 8

    HTTP/1.1 200 OK
    Server: Apache-Coyote/1.1
    X-Track: c2d5066d0fa946f584536775f07d1900
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Fri, 25 Oct 2013 14:27:55 GMT

    {"resultNumber":3}

|

- Response data in case of an input error

 .. code-block:: guess
    :emphasize-lines: 1, 4, 9

    HTTP/1.1 400 Bad Request
    Server: Apache-Coyote/1.1
    X-Track: cecd7b4d746249178643b7110b0eaa74
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Wed, 04 Dec 2013 15:06:01 GMT
    Connection: close
    
    {"errorResults":[{"code":"NotNull","message":"\"number2\"maynotbenull.","itemPath":"number2"},{"code":"NotNull","message":"\"number1\"maynotbenull.","itemPath":"number1"}]}

|

Posting form data in JSON format
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
How to fetch processing result by converting form data to JSON format and subsequently posting it using Ajax, is explained here.

Difference between this method and  "Posting form data" method, is explained.

- Controller

 .. code-block:: java

    @RequestMapping("xxx")
    @Controller
    public class XxxController {

        @RequestMapping(value = "plusForJson", method = RequestMethod.POST)
        @ResponseBody
        public CalculationResult plusForJson(
                @Validated @RequestBody CalculationParameters params) { // (1)
            CalculationResult result = new CalculationResult();
            int sum = params.getNumber1() + params.getNumber2();
            result.setResultNumber(sum);
            return result;
        }
        
        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | Sr. No.
     - | Description
   * - | (1)
     - | Assign ``@org.springframework.web.bind.annotation.RequestBody`` as the argument annotation of JavaBean for receiving form data.
       | By assigning this annotation, data in JSON format stored in the request body is unmarshalled and converted to object.
       | Specify ``@Validated`` when input validation is required. For error handling of input validation, refer to ":ref:`ajax_how_to_use_input_error`".
       | For details on input validation, refer to :doc:`../WebApplicationDetail/Validation`.

|

- JavaScript/HTML (JSP)

 .. code-block:: javascript

    // (2)
    function toJson($form) {
        var data = {};
        $($form.serializeArray()).each(function(i, v) {
            data[v.name] = v.value;
        });
        return JSON.stringify(data);
    }

    function plus() {

        $.ajax(contextPath + "/ajax/plusForJson", {
            type : "POST",
            contentType : "application/json;charset=utf-8", // (3)
            data : toJson($("#calculationForm")), // (2)
            dataType : "json",
            beforeSend : function(xhr) {
                xhr.setRequestHeader(csrfHeaderName, csrfToken);
            }

        }).done(function(json) {
            $("#calculationResult").text(json.resultNumber);

        }).fail(function(xhr) {
            $("#calculationResult").text("");

        });
        return false;
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | Sr. No.
     - | Description
   * - | (2)
     - | Function to change form input field to JSON string format.
   * - | (3)
     - | Change the media type of Content-Type to ``"application/json"`` as the data stored in request body is in JSON format.


|

| Following communication occurs when "=" button of the search form mentioned above, is clicked.
| Main points are highlighted.

- Request data

 .. code-block:: guess
    :emphasize-lines: 10,16

    POST /terasoluna-gfw-web-blank/ajax/plusForJson HTTP/1.1
    Host: localhost:9999
    Connection: keep-alive
    Content-Length: 31
    Accept: application/json, text/javascript, */*; q=0.01
    Origin: http://localhost:9999
    X-CSRF-TOKEN: 9d4f1e0c-c500-43f3-9125-a7a131ff88fa
    X-Requested-With: XMLHttpRequest
    User-Agent: Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/30.0.1599.101 Safari/537.36
    Content-Type: application/json;charset=UTF-8
    Referer: http://localhost:9999/terasoluna-gfw-web-blank/ajax/xxe?
    Accept-Encoding: gzip,deflate,sdch
    Accept-Language: en-US,en;q=0.8,ja;q=0.6
    Cookie: JSESSIONID=CECD7A6CB0431266B8D1173CCFA66B95

    {"number1":"34","number2":"56"}


|

.. _ajax_how_to_use_input_error:

Input error handling
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
How to perform error handling when an incorrect input value is specified, is explained here.

Input error handling methods are widely classified into the following.

* Method that performs error handling by providing an exception handling method.

* Method that performs error handling by receiving ``org.springframework.validation.BindingResult`` as an argument of Controller handler method.


|

Handling BindException
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| ``org.springframework.validation.BindException`` is an exception class generated when an incorrect input value is specified while sending the data as request parameter for binding to JavaBean.
| To receive request parameter and form data at the time of GET, in ``"application/x-www-form-urlencoded"`` format, exception handling  of ``BindException`` class needs to be performed.

- Controller

 .. code-block:: java

    @RequestMapping("xxx")
    @Controller
    public class XxxController {
    
        // omitted
    
        @ExceptionHandler(BindException.class) // (1)
        @ResponseStatus(value = HttpStatus.BAD_REQUEST) // (2)
        @ResponseBody // (3)
        public ErrorResults handleBindException(BindException e, Locale locale) { // (4)
            // (5)
            ErrorResults errorResults = new ErrorResults();
            for (FieldError fieldError : e.getBindingResult().getFieldErrors()) {
                errorResults.add(fieldError.getCode(),
                        messageSource.getMessage(fieldError, locale),
                            fieldError.getField());
            }
            for (ObjectError objectError : e.getBindingResult().getGlobalErrors()) {
                errorResults.add(objectError.getCode(),
                        messageSource.getMessage(objectError, locale),
                            objectError.getObjectName());
            }
            return errorResults;
        }
    
        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | Sr. No.
     - | Description
   * - | (1)
     - | Define the error handling method in Controller.
       | Assign ``@org.springframework.web.bind.annotation.ExceptionHandler`` annotation to the error handling method and specify the exception type to be handled in the value attribute.
       | In the above example, ``BindException.class`` is specified as the exception for binding.
   * - | (2)
     - | Specify the HTTP status information sent as response.
       | In the above example, ``400`` (Bad Request) is specified.
   * - | (3)
     - | Assign ``@ResponseBody`` annotation to write the returned object in response body.
   * - | (4)
     - | Declare the exception class to be handled as an argument of the error handling method.
   * - | (5)
     - | Implement error handling.
       | In the above example, a JavaBean is created to return the error information.

 .. tip::

    Locale object can be received as an argument while creating a message for error handling by considering internationalization.

|

- JavaBean storing the error information

 .. code-block:: java

    // (6)
    public class ErrorResult implements Serializable {
    
        private static final long serialVersionUID = 1L;
    
        private String code;
    
        private String message;
    
        private String itemPath;
    
        public String getCode() {
            return code;
        }
    
        public void setCode(String code) {
            this.code = code;
        }
    
        public String getMessage() {
            return message;
        }
    
        public void setMessage(String message) {
            this.message = message;
        }
    
        public String getItemPath() {
            return itemPath;
        }
    
        public void setItemPath(String itemPath) {
            this.itemPath = itemPath;
        }
    
    }

 .. code-block:: java

    // (7)
    public class ErrorResults implements Serializable {
    
        private static final long serialVersionUID = 1L;
    
        private List<ErrorResult> errorResults = new ArrayList<ErrorResult>();
    
        public List<ErrorResult> getErrorResults() {
            return errorResults;
        }
    
        public void setErrorResults(List<ErrorResult> errorResults) {
            this.errorResults = errorResults;
        }
    
        public ErrorResults add(String code, String message) {
            ErrorResult errorResult = new ErrorResult();
            errorResult.setCode(code);
            errorResult.setMessage(message);
            errorResults.add(errorResult);
            return this;
        }
    
        public ErrorResults add(String code, String message, String itemPath) {
            ErrorResult errorResult = new ErrorResult();
            errorResult.setCode(code);
            errorResult.setMessage(message);
            errorResult.setItemPath(itemPath);
            errorResults.add(errorResult);
            return this;
        }
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | Sr. No.
     - | Description
   * - | (6)
     - | JavaBean to store one record of error information.
   * - | (7)
     - | JavaBean to store multiple JavaBeans, each of which stores one record of error information.
       | JavaBeans mentioned in (6) are stored as a list.

|

Handling MethodArgumentNotValidException
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| ``org.springframework.web.bind.MethodArgumentNotValidException`` is the exception class generated when an incorrect input value is specified while binding the data stored in the request body to JavaBean using ``@RequestBody`` annotation.
| To receive it in formats such as ``"application/json"`` or ``"application/xml"`` etc., exception handling of ``MethodArgumentNotValidException`` needs to be performed.

- Controller

 .. code-block:: java

    @ExceptionHandler(MethodArgumentNotValidException.class) // (1)
    @ResponseStatus(value = HttpStatus.BAD_REQUEST)
    @ResponseBody
    public ErrorResults handleMethodArgumentNotValidException(
            MethodArgumentNotValidException e, Locale locale) { // (1)
        ErrorResults errorResults = new ErrorResults();

        // implement error handling.
        // omitted

        return errorResults;
    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | Sr. No.
     - | Description
   * - | (1)
     - | Specify ``MethodArgumentNotValidException.class`` as an exception for error handling.
       | Other than this, it is same as ``BindException``.

|

Handling HttpMessageNotReadableException
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| ``org.springframework.http.converter.HttpMessageNotReadableException`` is the exception class generated when a JavaBean could not be created from the data stored in Body, while binding the data stored in the request body to JavaBean, using ``@RequestBody`` annotation.
| To receive it in formats such as ``"application/json"`` or ``"application/xml"`` etc., exception handling of ``MethodArgumentNotValidException`` needs to be performed.

    .. note::

        Causes of specific errors differ depending on the implementation of ``HttpMessageConverter`` or library to be used.

        In ``MappingJackson2HttpMessageConverter`` implementation, wherein data in JSON format is to be converted to JavaBean using Jackson, if a string is specified in the Integer field instead of number, ``HttpMessageNotReadableException`` occurs.

- Controller

 .. code-block:: java

    @ExceptionHandler(HttpMessageNotReadableException.class) // (1)
    @ResponseStatus(value = HttpStatus.BAD_REQUEST)
    @ResponseBody
    public ErrorResults handleHttpMessageNotReadableException(
            HttpMessageNotReadableException e, Locale locale) {  // (1)
        ErrorResults errorResults = new ErrorResults();

        // implement error handling.
        // omitted

        return errorResults;
    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | Sr. No.
     - | Description
   * - | (1)
     - | Specify ``HttpMessageNotReadableException.class`` as the exception of error handling object.
       | Other than this, it is same as ``BindException``.


|

Handling by using BindingResult
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| When same type of JavaBean is returned in case of normal termination and in case of input error, error handling can be performed by receiving ``BindingResult`` as the handler method argument.
| This method can be used irrespective of the request data format.
| When ``BindingResult`` is not to be specified as handler method argument, it is necessary to implement error handling by the exception handling method mentioned earlier.

- Controller

 .. code-block:: java

    @RequestMapping(value = "plus", method = RequestMethod.POST)
    @ResponseBody
    public CalculationResult plus(
            @Validated @RequestBody CalculationParameters params,
            BindingResult bResult) { // (1)
        CalculationResult result = new CalculationResult();
        if (bResult.hasErrors()) { // (2)

            // (3)
            // implement error handling.
            // omitted

            return result; // (4)
        }
        int sum = params.getNumber1() + params.getNumber2();
        result.setResultNumber(sum);
        return result;
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | Sr. No.
     - | Description
   * - | (1)
     - | Declare ``BindingResult`` as a handler method argument.
       | ``BindingResult`` needs to be declared immediately after the JavaBean for input validation.
   * - | (2)
     - | Check whether there is any input value error.
   * - | (3)
     - | If so, perform error handling for input error.
       | In the above example, although error handling is omitted, it is assumed that settings for error message etc. are performed.
   * - | (4)
     - | Return processing result.


 .. note::

    In the above example, HTTP status code ``200`` (OK) is returned as response for both normal process as well as error.
    When it is necessary to classify HTTP status codes as per processing results, it can be implemented by setting ``org.springframework.http.ResponseEntity`` as the return value.
    As another approach, error handling can be implemented by the exception handling method mentioned earlier, without specifying ``BindingResult`` as the handler method argument.

      .. code-block:: java

        @RequestMapping(value = "plus", method = RequestMethod.POST)
        @ResponseBody
        public ResponseEntity<CalculationResult> plus(
                @Validated @RequestBody CalculationParameters params,
                BindingResult bResult) {
            CalculationResult result = new CalculationResult();
            if (bResult.hasErrors()) {

                // implement error handling.
                // omitted

                // (1)
                return ResponseEntity.badRequest().body(result);
            }
            // omitted

            // (2)
            return ResponseEntity.ok().body(result);
        }

      .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
      .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - | Sr. No.
         - | Description
       * - | (1)
         - | Return response data and HTTP status in case of input error.
       * - | (2)
         - | Return response data and HTTP status in case of normal termination.

|

Business error handling
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
How to handle business errors is explained here.

Methods that handle business errors are widely classified as follows.

* Method that performs error handling by providing a business exception handling method.

* Method that catches business exception in the handler method of Controller and performs error handling.


Handling business exception by exception handling method
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Business exceptions are handled by providing an exception handling method same as in case of input error.
| This method is recommended when it is necessary to implement the same error handling in requests for multiple handler methods.

- Controller

 .. code-block:: java

    @ExceptionHandler(BusinessException.class) // (1)
    @ResponseStatus(value = HttpStatus.CONFLICT) // (2)
    @ResponseBody
    public ErrorResults handleHttpBusinessException(BusinessException e, // (1)
            Locale locale) {
        ErrorResults errorResults = new ErrorResults();

        // implement error handling.
        // omitted

        return errorResults;
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | Sr. No.
     - | Description
   * - | (1)
     - | Specify ``BusinessException.class`` as an exception for error handling.
       | Other than this, it is similar to the input error handling for ``BindException``.
   * - | (2)
     - | Specify the HTTP status information sent as response.
       | In the above example, ``409`` (Conflict) is specified.

|

Handling business exception in handler method
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Business exception is caught by enclosing the process where the business error has occurred, in try clause.
| This method is implemented when error handling is different for each request.

- Controller

 .. code-block:: java

    @RequestMapping(value = "plus", method = RequestMethod.POST)
    @ResponseBody
    public ResponseEntity<CalculationResult> plusForJson(
            @Validated @RequestBody CalculationParameters params) {
        CalculationResult result = new CalculationResult();

        // omitted

        // (1)
        try {

            // call service method.
            // omitted

         // (2)
        } catch (BusinessException e) {

            // (3)
            // implement error handling.
            // omitted

            return ResponseEntity.status(HttpStatus.CONFLICT).body(result);
        }

        // omitted

        return ResponseEntity.ok().body(result);
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | Sr. No.
     - | Description
   * - | (1)
     - | Enclose the method call where business exception occurs, in try clause.
   * - | (2)
     - | Catch business exception.
   * - | (3)
     - | Perform the error handling intended for business exception error.
       | In the above example, although error handling is omitted, it is assumed that settings for error message etc. are performed.

.. raw:: latex

   \newpage

