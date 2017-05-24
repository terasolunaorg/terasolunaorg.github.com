REST Client (HTTP Client)
================================================================================

.. only:: html

 .. contents:: Index
    :depth: 3
    :local:

.. _RestClientOverview:

Overview
--------------------------------------------------------------------------------

This chapter explains how to call RESTful Web Service(REST API) by using \ ``org.springframework.web.client.RestTemplate``\  offered by Spring Framework.

.. _RestClientOverviewRestTemplate:

What is ``RestTemplate``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``RestTemplate``\  is a class which offers a method for calling REST API(Web API) and
is a HTTP client offered by Spring Framework.

The method by which \ ``RestTemplate``\  access REST API (Web API) is explained before explaining basic implementation method.

.. figure:: ./images_RestClient/RestClientOverview.png
    :alt: Constitution of RestTemplate
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - Sr. No.
      - Component
      - Description
    * - | (1)
      - | Application
      - | Call \ ``RestTemplate``\  method and request to call REST API (Web API).
    * - | (2)
      - | \ ``RestTemplate``\
      - | By using \ ``HttpMessageConverter``\ , convert Java object to message (JSON etc.) which is to be configured in the request body.
    * - | (3)
      - |
      - | Fetch \ ``ClientHttpRequest``\  from \ ``ClientHttpRequestFactory``\  and request to send a message (JSON etc.).
    * - | (4)
      - | \ ``ClientHttpRequest``\
      - | Configure message (JSON etc) in the request body and carry out request in REST API (Web API) through HTTP.
    * - | (5)
      - | \ ``RestTemplate``\
      - | Determine errors and perform error handling for HTTP transmission using \ ``ResponseErrorHandler``\ .
    * - | (6)
      - | \ ``ResponseErrorHandler``\
      - | Fetch response data from \ ``ClientHttpResponse``\ , determine errors and perform error handling.
    * - | (7)
      - | \ ``RestTemplate``\
      - | By using \ ``HttpMessageConverter``\ , convert message configured in response body (JSON etc) to Java object.
    * - | (8)
      - |
      - | Return results (Java object) of calling REST API (Web API) to the application.

.. note:: **Handling asynchronous processing**

    When response received from REST API is to be processed in another thread (asynchronous processing),
    \ ``org.springframework.web.client.AsyncRestTemplate``\  should be used instead of \ ``RestTemplate``\ .
    Refer :ref:`RestClientAsync` for implementation example of asynchronous processing.


.. _RestClientOverviewHttpMessageConverter:

``HttpMessageConverter``
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``org.springframework.http.converter.HttpMessageConverter``\  is an interface which mutually converts Java object handled by application and message (JSON etc) for communicating with server.

When \ ``RestTemplate``\  is used, implementation class of \ ``HttpMessageConverter``\  below is registered by default.

.. raw:: latex

   \newpage

.. tabularcolumns:: |p{0.05\linewidth}|p{0.25\linewidth}|p{0.55\linewidth}|p{0.15\linewidth}|
.. list-table:: **HttpMessageConverter registered by default**
    :header-rows: 1
    :widths: 5 25 55 15
    :class: longtable

    * - Sr. No.
      - Class name
      - Description
      - Support type
    * - | (1)
      - | ``org.springframework.http.converter.``
        | ``ByteArrayHttpMessageConverter``
      - | A class for conversion of "HTTP body (text or binary data) ⇔ Byte array".
        | It supports all media types (\ ``*/*``\ ) by default.
      - | ``byte[]``
    * - | (2)
      - | ``org.springframework.http.converter.``
        | ``StringHttpMessageConverter``
      - | A class for conversion of "HTTP body (text) ⇔ String".
        | It supports all text media types (\ ``text/*``\ ) by default.
      - | ``String``
    * - | (3)
      - | ``org.springframework.http.converter.``
        | ``ResourceHttpMessageConverter``
      - | A class for conversion of "HTTP body (binary data) ⇔ Resource object of Spring".
        | It supports all media types (\ ``*/*``\ ) by default.
      - | ``Resource`` [#p1]_
    * - | (4)
      - | ``org.springframework.http.converter.xml.``
        | ``SourceHttpMessageConverter``
      - | A class for conversion of "HTTP body (XML) ⇔ XML source object".
        | It supports media types for XML (\ ``text/xml``\ ,\ ``application/xml``\ ,\ ``application/*-xml``\ ) by default.
      - | ``Source`` [#p2]_
    * - | (5)
      - | ``org.springframework.http.converter.support.``
        | ``AllEncompassingFormHttpMessageConverter``
      - | A class for conversion of "HTTP body ⇔ \ ``MultiValueMap``\  object".
        | It is an extension class of \ ``FormHttpMessageConverter``\  and supports conversion to XML and JSON as a multipart part data.
        | It supports media types for form data (\ ``application/x-www-form-urlencoded``\ ,\ ``multipart/form-data``\ ) by default.

        * When media type is \ ``application/x-www-form-urlencoded``\ , the data is read / written as \ ``MultiValueMap<String, String>``\ .
        * When media type is \ ``multipart/form-data``\ , data is written as \ ``MultiValueMap<String, Object>``\  and \ ``Object``\  is converted by \ ``HttpMessageConveter``\  configured separately in \ ``AllEncompassingFormHttpMessageConverter``\ .
          (Caution: Refer Note)

        | Refer `AllEncompassingFormHttpMessageConverter <https://github.com/spring-projects/spring-framework/blob/v4.2.7.RELEASE/spring-web/src/main/java/org/springframework/http/converter/support/AllEncompassingFormHttpMessageConverter.java>`_\  and
          `FormHttpMessageConverter <https://github.com/spring-projects/spring-framework/blob/v4.2.7.RELEASE/spring-web/src/main/java/org/springframework/http/converter/FormHttpMessageConverter.java>`_\  source for \ ``HttpMessageConveter``\  used for conversion of part data which is registered by default. Note that, it is also possible to register an arbitrary \ ``HttpMessageConverter``\ .
      - | ``MultiValueMap`` [#p3]_

.. raw:: latex

   \newpage

.. note:: **When media type of AllEncompassingFormHttpMessageConverter is multipart/form-data**

    When media type is \ ``multipart/form-data``\ , conversion of "HTTP body from \ ``MultiValueMap``\  object" can be done, however,
    conversion "from HTTP body to \ ``MultiValueMap``\  object" is currently not supported.
    Hence, an independent implementation is required if conversion "from HTTP body to \ ``MultiValueMap``\  object" is to be carried out.

\

.. raw:: latex

   \newpage

.. tabularcolumns:: |p{0.05\linewidth}|p{0.25\linewidth}|p{0.55\linewidth}|p{0.15\linewidth}|
.. list-table:: **HttpMessageConverter that is registered when a dependent library exists on the class path**
    :header-rows: 1
    :widths: 5 25 55 15
    :class: longtable

    * - Sr. No.
      - Class Name
      - Description
      - Support type
    * - | (6)
      - | ``org.springframework.http.converter.feed.``
        | ``AtomFeedHttpMessageConverter``
      - | A class for conversion of "HTTP body (Atom) ⇔ Atom feed object".
        | It supports media type for ATOM (\ ``application/atom+xml``\ ) by default.
        | (It is registered when ROME exists on the class path)
      - | ``Feed`` [#p4]_
    * - | (7)
      - | ``org.springframework.http.converter.feed.``
        | ``RssChannelHttpMessageConverter``
      - | A class for conversion of "HTTP body (RSS) ⇔ Rss channel object".
        | It supports media type for RSS (\ ``application/rss+xml``\ ) by default.
        | (It is registered when ROME exists on the class path)
      - | ``Channel`` [#p5]_
    * - | (8)
      - | ``org.springframework.http.converter.json.``
        | ``MappingJackson2HttpMessageConverter``
      - | A class for conversion of "HTTP body (JSON) ⇔ JavaBean".
        | It supports media type for JSON (\ ``application/json``\ ,\ ``application/*+json``\ ) by default.
        | (It is registered when Jackson2 exists on the class path)
      - | ``Object`` (JavaBean)
        | ``Map``
    * - | (9)
      - | ``org.springframework.http.converter.xml.``
        | ``MappingJackson2XmlHttpMessageConverter``
      - | A class for conversion of "HTTP body (XML) ⇔ JavaBean".
        | It supports media type for XML (\ ``text/xml``\ ,\ ``application/xml``\ ,\ ``application/*-xml``\ ) by default.
        | (It is registered when Jackson-dataformat-xml exists on the class path)
      - | ``Object`` (JavaBean)
        | ``Map``
    * - | (10)
      - | ``org.springframework.http.converter.xml.``
        | ``Jaxb2RootElementHttpMessageConverter``
      - | A class for conversion of "HTTP body (XML) ⇔ JavaBean".
        | It supports media type for XML (\ ``text/xml``\ ,\ ``application/xml``\ ,\ ``application/*-xml``\ ) by default.
        | (It is registered when JAXB exists on the class path)
      - | ``Object`` (JavaBean)
    * - | (11)
      - | ``org.springframework.http.converter.json.``
        | ``GsonHttpMessageConverter``
      - | A class for conversion of "HTTP body (JSON) ⇔ JavaBean".
        | It supports media type for JSON (\ ``application/json``\ ,\ ``application/*+json``\ ) by default.
        | (It is registered when Gson exists on the class path)
      - | ``Object`` (JavaBean)
        | ``Map``

.. raw:: latex

   \newpage

\

.. [#p1] \ ``org.springframework.core.io``\  package class
.. [#p2] \ ``javax.xml.transform``\  package class
.. [#p3] \ ``org.springframework.util``\  package class
.. [#p4] \ ``com.rometools.rome.feed.atom``\  package class
.. [#p5] \ ``com.rometools.rome.feed.rss``\  package class


.. _RestClientOverviewClientHttpRequestFactory:

``ClientHttpRequestFactory``
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``RestTemplate``\  delegates the process of communicating with the server to implementation class of three interfaces given below.

* ``org.springframework.http.client.ClientHttpRequestFactory``
* ``org.springframework.http.client.ClientHttpRequest``
* ``org.springframework.http.client.ClientHttpResponse``

Of the 3 interfaces, the developers are aware of \ ``ClientHttpRequestFactory``\  interface.
\ ``ClientHttpRequestFactory``\  resolves a class (implementation class of \ ``ClientHttpRequest``\  and \ ``ClientHttpResponse``\  interface) which communicates with the server.

Note that, the main implementation class of \ ``ClientHttpRequestFactory``\  offered by Spring Framework is as given below.

.. tabularcolumns:: |p{0.05\linewidth}|p{0.25\linewidth}|p{0.70\linewidth}|
.. list-table:: **Main implementation class of ClientHttpRequestFactory offered by Spring Framework**
   :header-rows: 1
   :widths: 5 25 70

   * - Sr. No.
     - Class Name
     - Description
   * - | (1)
     - | ``org.springframework.http.client.``
       | ``SimpleClientHttpRequestFactory``
     - | An implementation class for communication (synchronous, asynchronous) by using `HttpURLConnection <https://docs.oracle.com/javase/8/docs/api/java/net/HttpURLConnection.html>`_\  API of Java SE standard. (Implementation class used as a default)
   * - | (2)
     - | ``org.springframework.http.client.``
       | ``Netty4ClientHttpRequestFactory``
     - | An implementation class for communication (synchronous, asynchronous) by using `Netty 4 <http://netty.io/>`_\  API.
   * - | (3)
     - | ``org.springframework.http.client.``
       | ``HttpComponentsClientHttpRequestFactory``
     - | An implementation class for synchronous communication by using `Apache HttpComponents HttpClient <http://hc.apache.org/httpcomponents-client-ga/>`_\  API. (HttpClient 4.3 and above versions are required)
   * - | (4)
     - | ``org.springframework.http.client.``
       | ``HttpComponentsAsyncClientHttpRequestFactory``
     - | An implementation class for asynchronous communication by using `Apache HttpComponents HttpAsyncClient <http://hc.apache.org/httpcomponents-asyncclient-dev/>`_\  API. (HttpAsyncClient 4.0 and above version are required)
   * - | (5)
     - | ``org.springframework.http.client.``
       | ``OkHttpClientHttpRequestFactory``
     - | An implementation class for communication (synchronous, asynchronous) by using `Square OkHttp <http://square.github.io/okhttp/>`_\  API.

.. note:: **Regarding implementation class of ClientHttpRequestFactory to be used**

    Default implementation used by \ ``RestTemplate``\  is \ ``SimpleClientHttpRequestFactory``\  and it can also act as an implementation example in this guideline while using \ ``SimpleClientHttpRequestFactory``\ .
    If it does not meet requirements in \ ``HttpURLConnection``\  of Java SE, using libraries like Netty, Apache Http Components can be explored.


.. _RestClientOverviewResponseErrorHandler:

``ResponseErrorHandler``
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``RestTemplate``\  handles the errors during the communication with the server by delegating to \ ``org.springframework.web.client.ResponseErrorHandler``\  interface.



* A method to determine errors (\ ``hasError``\ )
* A method to handle errors (\ ``handleError``\ )

are defined in \ ``ResponseErrorHandler``\ . Spring Framework offers \ ``org.springframework.web.client.DefaultResponseErrorHandler``\  as a default implementation.

\ ``DefaultResponseErrorHandler``\  carries out error handling as below according to values of HTTP status codes which have been sent as a response from the server.

* When response code is standard (2xx), error handling is not carried out.
* When response code is from client error system (4xx), \ ``org.springframework.web.client.HttpClientErrorException``\  is generated.
* When response code is from server error system (5xx), \ ``org.springframework.web.client.HttpServerErrorException``\  is generated.
* When response code is undefined (user defined custom code), \ ``org.springframework.web.client.UnknownHttpStatusCodeException``\  is generated.

.. note:: **How to fetch response data at the time of error**

    Response data at the time of error (HTTP status code,  response header, response body etc) can be fetched by calling getter method of exception class.


.. _RestClientOverviewClientHttpRequestInterceptor:

``ClientHttpRequestInterceptor``
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``org.springframework.http.client.ClientHttpRequestInterceptor``\  is an interface for implementing a common process before and after communicating with the server.

If \ ``ClientHttpRequestInterceptor``\  is used, the common processes like

* Communication log with server
* Configuration of authentication header

can be applied in \ ``RestTemplate``\ .

.. note:: **Action specifications for ClientHttpRequestInterceptor**

    \ ``ClientHttpRequestInterceptor``\  can be used for multiple times and is executed as a chain in a specified sequence.
    This operation is similar to working of a servlet filter and the HTTP communication process by \ ``ClientHttpRequest``\  is registered as a chain destination executed at the end.
    For example, when you want to cancel communication with the server once it fulfils a certain condition, a chain destination need not be called.

    When this system is used, processes like

    * Blocking communication with server
    * Retrying communication process

    can also be applied.


.. _RestClientHowToUse:

How to use
--------------------------------------------------------------------------------

This chapter explains how to implement a client process which uses \ ``RestTemplate``\ .

.. note:: **Regarding HTTP method supported by RestTemplate**

    In this guideline, only the implementation example of client process which use GET method and POST method is introduced, however,
    \ ``RestTemplate``\  supports other HTTP methods (PUT, PATCH, DELETE, HEAD, OPTIONS etc) as well and can be used in the similar way.
    Refer Javadoc of \ `RestTemplate <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html>`_\  for details.

.. _RestClientHowToUseSetup:

\ ``RestTemplate``\  Setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When \ ``RestTemplate``\  is used, \ ``RestTemplate``\  is registered in DI container and injected in the component which uses \ ``RestTemplate``\ .


Dependent library setup
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| spring-web library of Spring Framework is added to \ ``pom.xml``\  for using \ ``RestTemplate``\ .
| In case of multi-project configuration, it is added to \ ``pom.xml``\  of domain project.

.. code-block:: xml

    <dependencies>

        <!-- (1) -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
        </dependency>

    </dependencies>
    
.. note::  
	In the above setting example, since it is assumed that the dependent library version is managed by the parent project terasoluna-gfw-parent , specifying the version in pom.xml is not necessary.  
	The above dependent library used by terasoluna-gfw-parent is defined by \ `Spring IO Platform <http://platform.spring.io/platform/>`_\ .

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Add \ ``spring-web``\  library of Spring Framework to dependencies.


Bean definition of \ ``RestTemplate``\ 
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Define bean for \ ``RestTemplate``\  and register in DI container.

**Definition example of bean definition file (applicationContext.xml)**

.. code-block:: xml

    <bean id="restTemplate" class="org.springframework.web.client.RestTemplate" /> <!-- (1) -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | When \ ``RestTemplate``\  is used similar to default configuration, register a bean by using default constructor.


.. note:: **How to customise RestTemplate**

    When HTTP communication process is to be customised, define a bean as below.

     .. code-block:: xml

        <bean id="clientHttpRequestFactory"
              class="org.springframework.http.client.SimpleClientHttpRequestFactory"> <!-- (1) -->
            <!-- Set properties for customize a http communication (omit on this sample) -->
        </bean>

        <bean id="restTemplate" class="org.springframework.web.client.RestTemplate">
            <constructor-arg ref="clientHttpRequestFactory" /> <!-- (2) -->
        </bean>

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90

        * - Sr. No.
          - Description
        * - | (1)
          - | Define a bean for \ ``ClientHttpRequestFactory``\ .
            | A method to customise timeout configuration is introduced in this guideline. Refer :ref:`RestClientHowToUseTimeoutSettings` for details.
        * - | (2)
          - | Register a bean by using a constructor which specifies \ ``ClientHttpRequestFactory``\  in the argument.

    Also, refer

    * :ref:`RestClientHowToExtendHttpMessageConverter`
    * :ref:`RestClientHowToUseErrorHandlingResponseEntity`
    * :ref:`RestClientHowToExtendClientHttpRequestInterceptor`

    for how to customise \ ``HttpMessageConverter``\ , \ ``ResponseErrorHandler``\  and \ ``ClientHttpRequestInterceptor``\ .


Using \ ``RestTemplate``\ 
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When \ ``RestTemplate``\  is used, \ ``RestTemplate``\  registered in DI container is injected.

**Injection example for RestTemplate**

.. code-block:: java

    @Service
    public class AccountServiceImpl implements AccountService {

        @Inject
        RestTemplate restTemplate;

        // ...

    }


.. _RestClientHowToUseGet:

Sending GET request
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``RestTemplate``\  offers multiple methods to send a GET request.

* Usually, \ ``getForObject``\  method or \ ``getForEntity``\  method are used.
* When a detailed setting such as setting a header is to be carried out, \ ``org.springframework.http.RequestEntity``\  and \ ``exchange``\  methods are used.

Implementation by using \ ``getForObject``\  method
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When only the response body is required to be fetched, \ ``getForObject``\  method is used.

**How to use getForObject method**

Field declaration part

.. code-block:: java

    @Value("${api.url:http://localhost:8080/api}")
    URI uri;


Internal method

.. code-block:: java

    User user = restTemplate.getForObject(uri, User.class); // (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | When ``getForObject``\  method is used, the response body value is sent as a return value.
        | Response body data is returned after it has been converted to Java class specified in the second argument, by using \ ``HttpMessageConverter``\ .


Implementation by using \ ``getForEntity``\  method
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When HTTP status code, response header and response body must be fetched, \ ``getForEntity``\  method is used.

**How to use getForEntity method**

.. code-block:: java

    ResponseEntity<User> responseEntity =
            restTemplate.getForEntity(uri, User.class); // (1)
    HttpStatus statusCode = responseEntity.getStatusCode(); // (2)
    HttpHeaders header = responseEntity.getHeaders(); // (3)
    User user = responseEntity.getBody(); // (4)


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | When ``getForEntity``\  method is used, \ ``org.springframework.http.ResponseEntity``\  is sent as a return value.
    * - | (2)
      - | Fetch HTTP status code by using \ ``getStatusCode``\  method.
    * - | (3)
      - | Fetch response header by using \ ``getHeaders``\  method.
    * - | (4)
      - | Fetch response body by using \ ``getBody``\  method.

.. note:: **ResponseEntity**

    ``ResponseEntity``\  is a class which shows HTTP response and can fetch HTTP status code, response header and response body information.
    Refer Javadoc of \ `ResponseEntity <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/http/ResponseEntity.html>`_\  for details.



Implementation by using \ ``exchange``\  method
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When a request header must be specified, \ ``org.springframework.http.RequestEntity``\  is generated and \ ``exchange``\  method is used.

**How to use exchange method**

import part

.. code-block:: java

    import org.springframework.http.RequestEntity;
    import org.springframework.http.ResponseEntity;


Field declaration part

.. code-block:: java

    @Value("${api.url:http://localhost:8080/api}")
    URI uri;


Internal method

.. code-block:: java

    RequestEntity requestEntity = RequestEntity
            .get(uri)//(1)
            .build();//(2)

    ResponseEntity<User> responseEntity =
            restTemplate.exchange(requestEntity, User.class);//(3)

    User user = responseEntity.getBody();//(4)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Use \ ``get``\  method of ``RequestEntity``\  and generate request builder for GET request.
        | Specify URI in the parameter.
    * - | (2)
      - | Use \ ``build``\  method of ``RequestEntity.HeadersBuilder``\  and create \ ``RequestEntity``\  object.
    * - | (3)
      - | Use ``exchange``\  method and send request. Specify response data type in the second argument.
        | \ ``ResponseEntity<T>``\  is sent as a response. Specify response data type in Type parameter.
    * - | (4)
      - | Use ``getBody``\  method and fetch response body data.

.. note:: **RequestEntity**

    ``RequestEntity``\  is a class which shows HTTP request and can set connection URI, HTTP method, request header and request body.
    Refer Javadoc of \ `RequestEntity <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/http/RequestEntity.html>`_\  for details.

    Also, refer :ref:`RestClientHowToUseRequestHeader` for how to configure a request header.


.. _RestClientHowToUsePost:

Sending POST request
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``RestTemplate``\  offers multiple methods for carrying out POST request.

* Usually, \ ``postForObject``\  and \ ``postForEntity``\  are used.
* When a detailed setting like setting any header is to be carried out, \ ``RequestEntity``\  and \ ``exchange``\  methods are used.

Implementation by using \ ``postForObject``\  method
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When only response body is required to be fetched as POST results, \ ``postForObject``\  method is used.

**How to use postForObject method**

.. code-block:: java


    User user = new User();

    //...

    User user = restTemplate.postForObject(uri, user, User.class); // (1)


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | ``postForObject``\  method can easily implement a POST request.
        | Specify Java object in the second argument which is converted to request body by using ``HttpMessageConverter``\ .
        | When ``postForObject``\  method is used, response body value is sent as a return value.

Implementation using \ ``postForEntity``\  method
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When HTTP status code, response header and response body are to be fetched as POST results, \ ``postForEntity``\  method is used.

**How to use postForEntity method**

.. code-block:: java

    User user = new User();

    //...

    ResponseEntity<User> responseEntity =
            restTemplate.postForEntity(uri, user, User.class); // (1)


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | ``postForEntity``\  method can implement a POST request easily similar to \ ``getForObject``\  method.
        | When ``postForEntity``\  method is used, \ ``ResponseEntity``\  is sent as a return value.
        | Fetch response body value from \ ``ResponseEntity``\ .



Implementation using \ ``exchange``\  method
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When a request header is to be specified, \ ``RequestEntity``\  is generated and \ ``exchange``\  method is used.

**How to use exchange method**

import part

.. code-block:: java

    import org.springframework.http.RequestEntity;
    import org.springframework.http.ResponseEntity;


Field declaration part

.. code-block:: java

    @Value("${api.url:http://localhost:8080/api}")
    URI uri;


Internal method

.. code-block:: java

    User user = new User();

    //...

    RequestEntity<User> requestEntity = RequestEntity//(1)
            .post(uri)//(2)
            .body(user);//(3)

    ResponseEntity<User> responseEntity =
            restTemplate.exchange(requestEntity, User.class);//(4)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Use ``RequestEntity``\  and generate a request. Specify type of the data specified in the request body, in Type parameter.
    * - | (2)
      - | Use ``post``\  method and generate a request builder for POST request. Specify URI in the parameter.
    * - | (3)
      - | Use \ ``body``\  method of ``RequestEntity.BodyBuilder``\  and create \ ``RequestEntity``\  object.
        | Specify Java object that has been converted to request body, in the parameter.
    * - | (4)
      - | Use ``exchange``\  method and send a request.

.. note:: **How to configure a request header**

    Refer :ref:`RestClientHowToUseRequestHeader` for how to configure a request header.


.. _RestClientHowToUseGetCollection:

Fetch data in collection format
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When the message of response body received from server as a response is in collection format, the implementation is as below.

**How to fetch data in collection format**

.. code-block:: java

    ResponseEntity<List<User>> responseEntity = //(1)
        restTemplate.exchange(requestEntity, new ParameterizedTypeReference<List<User>>(){}); //(2)

    List<User> userList = responseEntity.getBody();//(3)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify \ ``List``\<Response data type> in ``ResponseEntity``\  Type parameter.
    * - | (2)
      - | Specify instance of \ ``org.springframework.core.ParameterizedTypeReference``\  in the second argument of ``exchange``\  method, and specify \ ``List``\ <Response data type> in Type parameter.
    * - | (2)
      - | Fetch response body data by ``getBody``\  method.

.. _RestClientHowToUseRequestHeader:

Configuration of request header
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If \ ``RequestEntity``\  and \ ``exchange``\  methods are used, a specific header or any other header can be set by using \ ``RequestEntity``\  method.
Refer Javadoc of \ `RequestEntity <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/http/RequestEntity.html>`_\  for details.

This guideline explains about

* :ref:`RestClientHowToUseRequestHeaderContentType`
* :ref:`RestClientHowToUseRequestHeaderAccept`
* :ref:`RestClientHowToUseRequestHeaderAnyHeader`



.. _RestClientHowToUseRequestHeaderContentType:

Configuration of Content-Type header
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

While sending data to server, a usual Content-Type header must be specified.

**How to configure Content-Type header**

.. code-block:: java

    User user = new User();

    //...

    RequestEntity<User> requestEntity = RequestEntity
            .post(uri)
            .contentType(MediaType.APPLICATION_JSON) // (1)
            .body(user);



.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Use \ ``contentType``\  method of ``RequestEntity.BodyBuilder``\  and specify value of Context-Type header.
        | In the implementation example above, "\ ``application/json``\"  is specified which indicates that the data format is JSON.


.. _RestClientHowToUseRequestHeaderAccept:

Configuration of Accept header
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When the format of data to be fetched from server is specified, Accept header must be specified.
When the server does not support multiple data format responses, Accept header may not be specified explicitly.

**Configuration example of Accept header**

.. code-block:: java

    User user = new User();

    //...

    RequestEntity<User> requestEntity = RequestEntity
            .post(uri)
            .accept(MediaType.APPLICATION_JSON) // (1)
            .body(user);



.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Use \ ``accept``\  method of ``RequestEntity.HeadersBuilder``\  and specify value of Accept header.
        | In the implementation example above, "\ ``application/json``\ " is specified which indicates that format of the data that can be fetched is JSON format.


.. _RestClientHowToUseRequestHeaderAnyHeader:

Configuration of an  arbitrary request header
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

A request header must be specified to access server.

**Configuration example for an arbitrary header**

.. code-block:: java

    User user = new User();

    //...

    RequestEntity<User> requestEntity = RequestEntity
            .post(uri)
            .header("Authorization", "Basic " + base64Credentials) // (1)
            .body(user);



.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Use \ ``header``\  method of ``RequestEntity.HeadersBuilder``\  and specify name and value of request header.
        | In the implementation example above, credentials information necessary for Basic authentication is specified in Authorization header.


.. _RestClientHowToUseErrorHandling:

Error Handling
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _RestClientHowToUseErrorHandlingHandleException:

Exception Handling (Default Behaviour)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Exceptions like

* \ ``HttpClientErrorException``\  when response code is of client error system (4xx)
* \ ``HttpServerErrorException``\  when response code is of server error system (5xx)
* \ ``UnknownHttpStatusCodeException``\  when response code is a undefined code (user defined custom code)

occur in default implementation (\ ``DefaultResponseErrorHandler``\ ) of \ ``RestTemplate``\ , these exceptions must be handled as and when necessary.

**Implementation example of exception handling**

.. note::

    An example of exception handling when a server error has occurred is shown below as an implementation example.

    \ **Appropriate exception handling must be carried out**\  as per requirements of an application.

Field declaration part

.. code-block:: java

    @Value("${api.retry.maxCount}")
    int retryMaxCount;

    @Value("${api.retry.retryWaitTimeCoefficient}")
    int retryWaitTimeCoefficient;


Internal method

.. code-block:: java

    int retryCount = 0;
    while (true) {
        try {

            responseEntity = restTemplate.exchange(requestEntity, String.class);

            if (log.isInfoEnabled()) {
                log.info("Success({}) ", responseEntity.getStatusCode());
            }

            break;

        } catch (HttpServerErrorException e) { // (1)

            if (retryCount == retryMaxCount) {
                throw e;
            }

            retryCount++;

            if (log.isWarnEnabled()) {
                log.warn("An error ({}) occurred on the server. (The number of retries：{} Times)", e.getStatusCode(),
                    retryCount);
            }

            try {
                Thread.sleep(retryWaitTimeCoefficient * retryCount);
            } catch (InterruptedException ie) {
                Thread.currentThread().interrupt();
            }

            //...
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Catch exception and perform error handling. In case of a server error (500 system), catch \ ``HttpServerErrorException``\ .


.. _RestClientHowToUseErrorHandlingResponseEntity:

Returning \ ``ResponseEntity``\  (Error handler extension)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

By setting implementation class of \ ``org.springframework.web.client.ResponseErrorHandler``\  interface in \ ``RestTemplate``\ , an independent error handling process can be carried out.

In the example below, the error handler is extended so as to return \ ``ResponseEntity``\  even when a server error and a client error has occurred.

**How to create an implementation class of error handler**

.. code-block:: java

    import org.springframework.http.client.ClientHttpResponse;
    import org.springframework.web.client.DefaultResponseErrorHandler;

    public class CustomErrorHandler extends DefaultResponseErrorHandler { // (1)

        @Override
        public void handleError(ClientHttpResponse response) throws IOException {
            //Don't throw Exception.
        }

    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Create implementation class of ``ResponseErrorHandler``\  interface.
        | In the implementation example above, \ ``DefaultResponseErrorHandler``\  - an implementation class of default error handler is extended
        | and \ ``ResponseEntity``\  is returned without generating an exception when a server error and client error has occurred.

**Implementation example of bean definition file (applicationContext.xml)**

.. code-block:: xml

    <bean id="customErrorHandler" class="com.example.restclient.CustomErrorHandler" /> <!-- (1) -->

    <bean id="restTemplate" class="org.springframework.web.client.RestTemplate">
        <property name="errorHandler" ref="customErrorHandler" /><!-- (2) -->
    </bean>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Define bean for implementation class of \ ``ResponseErrorHandler``\ .
    * - | (2)
      - | Inject \ ``ResponseErrorHandler``\  bean in \ ``errorHandler``\  property.

**Implementation example of client process**

.. code-block:: java

    int retryCount = 0;
    while (true) {

        responseEntity = restTemplate.exchange(requestEntity, User.class);

        if (responseEntity.getStatusCode() == HttpStatus.OK) { // (1)

            break;

        } else if (responseEntity.getStatusCode() == HttpStatus.SERVICE_UNAVAILABLE) { // (2)

            if (retryCount == retryMaxCount) {
                break;
            }

            retryCount++;

            if (log.isWarnEnabled()) {
                log.warn("An error ({}) occurred on the server. (The number of retries：{} Times)",
                    responseEntity.getStatusCode(), retryCount);
            }

            try {
                Thread.sleep(retryWaitTimeCoefficient * retryCount);
            } catch (InterruptedException ie) {
                Thread.currentThread().interrupt();
            }

            //...
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | In the implementation example above, since error handler is extended so as to return \ ``ResponseEntity``\  even at the time of error, it is necessary to check whether process results are normal after fetching HTTP status code from \ ``ResponseEntity``\  thus returned.
    * - | (2)
      - | HTTP status code can be fetched from returned \ ``ResponseEntity``\  even at the time of error and  process can be controlled corresponding to that value.

.. _RestClientHowToUseTimeoutSettings:

Setting communication timeout
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When a timeout period is to be specified for communicating with server, define a bean as given below.

**Implementation example of bean definition file (applicationContext.xml)**

.. code-block:: xml

    <bean id="clientHttpRequestFactory"
          class="org.springframework.http.client.SimpleClientHttpRequestFactory">
        <property name="connectTimeout" value="${api.connectTimeout: 2000}" /><!-- (1) -->
        <property name="readTimeout" value="${api.readTimeout: 2000}" /><!-- (2) -->
    </bean>

    <bean id="restTemplate" class="org.springframework.web.client.RestTemplate">
        <constructor-arg ref="clientHttpRequestFactory" />
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify connection timeout (milliseconds) with server in \ ``connectTimeout``\  property.
        | When timeout occurs, \ ``org.springframework.web.client.ResourceAccessException``\  is generated.
    * - | (2)
      - | Specify response data read timeout (milliseconds) in \ ``readTimeout``\  property.
        | When timeout occurs, \ ``ResourceAccessException``\  is generated.

.. note:: **Cause Exception during Timeout Occurrence**

    \ ``ResourceAccessException``\  wraps the cause exception. Cause exception during connection timeout and read timeout occurrence is \ ``java.net.SocketTimeoutException``\  for both.
    When default implementation (\ ``SimpleClientHttpRequestFactory``\ ) is used, it must be added that type of timeout occurrence cannot be distinguished by the type of exception class.

    Note that, since operation while using \ ``HttpRequestFactory``\  is not verified, cause exception is likely to be different from the one described above.
    When other \ ``HttpRequestFactory``\  is used, appropriate exception handling must be employed after assessing the exception occurred during the timeout.


.. _RestClientHowToUseHttps:

Using SSL self-signed certificate
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Implementation is as given below when a SSL self-signed certificate is to be used in the test environment.

**Implementation example of FactoryBean**

Implement \ ``org.springframework.beans.factory.FactoryBean``\  to create \ ``org.springframework.http.client.ClientHttpRequestFactory``\   to be passed in constructor argument, in Bean definition of \ ``RestTemplate``\ .

.. code-block:: java

    import java.security.KeyStore;

    import javax.net.ssl.KeyManagerFactory;
    import javax.net.ssl.SSLContext;
    import javax.net.ssl.TrustManagerFactory;

    import org.apache.http.client.HttpClient;
    import org.apache.http.impl.client.HttpClientBuilder;
    import org.springframework.beans.factory.FactoryBean;
    import org.springframework.http.client.ClientHttpRequestFactory;
    import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;

    public class RequestFactoryBean implements
            FactoryBean<ClientHttpRequestFactory> {

        private String keyStoreFileName;

        private char[] keyStorePassword;

        @Override
        public ClientHttpRequestFactory getObject() throws Exception {

            // (1)
            SSLContext sslContext = SSLContext.getInstance("TLS");

            KeyStore ks = KeyStore.getInstance(KeyStore.getDefaultType());
            ks.load(this.getClass().getClassLoader()
                    .getResourceAsStream(this.keyStoreFileName),
                    this.keyStorePassword);

            KeyManagerFactory kmf = KeyManagerFactory.getInstance(KeyManagerFactory
                    .getDefaultAlgorithm());
            kmf.init(ks, this.keyStorePassword);

            TrustManagerFactory tmf = TrustManagerFactory
                    .getInstance(TrustManagerFactory.getDefaultAlgorithm());
            tmf.init(ks);

            sslContext.init(kmf.getKeyManagers(), tmf.getTrustManagers(), null);

            // (2)
            HttpClient httpClient = HttpClientBuilder.create()
                    .setSSLContext(sslContext).build();

            // (3)
            ClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory(
                    httpClient);

            return factory;
        }

        @Override
        public Class<?> getObjectType() {
            return ClientHttpRequestFactory.class;
        }

        @Override
        public boolean isSingleton() {
            return true;
        }

        public void setKeyStoreFileName(String keyStoreFileName) {
            this.keyStoreFileName = keyStoreFileName;
        }

        public void setKeyStorePassword(char[] keyStorePassword) {
            this.keyStorePassword = keyStorePassword;
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Create SSL context based on file name and password of keystore file which is specified in subsequent bean definition.
        | Keystore file of SSL self-signed certificate to be used is placed on the class path.
    * - | (2)
      - | Create \ ``org.apache.http.client.HttpClient``\  which uses SSL context thus created.
    * - | (3)
      - | Create \ ``ClientHttpRequestFactory``\  which uses \ ``HttpClient``\  thus created.


Apache HttpComponents HttpClient library is required in order to use of \ ``HttpClient`` \ and \ ``HttpClientBuilder``\.
Add below Apache HttpComponents HttpClient dependency library into \ :file:`pom.xml`\.


* :file:`pom.xml`

 .. code-block:: xml

    <dependency>
        <groupId>org.apache.httpcomponents</groupId>
        <artifactId>httpclient</artifactId>
    </dependency>

.. note::  
	In the above setting example, since it is assumed that the dependent library version is managed by the parent project terasoluna-gfw-parent , specifying the version in pom.xml is not necessary.  
	The above dependent library used by terasoluna-gfw-parent is defined by \ `Spring IO Platform <http://platform.spring.io/platform/>`_\ .
	
**Implementation example of bean definition file (applicationContext.xml)**

Define \ ``RestTemplate``\ which carries out SSL communication using SSL self-signed certificate.

.. code-block:: xml

    <bean id="httpsRestTemplate" class="org.springframework.web.client.RestTemplate">
        <constructor-arg>
            <bean class="com.example.restclient.RequestFactoryBean"><!-- (1) -->
                <property name="keyStoreFileName" value="${rscl.keystore.filename}" />
                <property name="keyStorePassword" value="${rscl.keystore.password}" />
            </bean>
        </constructor-arg>
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify created \ ``RequestFactoryBean``\  in \ ``RestTemplate``\  constructor.
        | Pass file name and password of keystore file in \ ``RequestFactoryBean``\ .

**How to use RestTemplate**

The method to use \ ``RestTemplate``\  is same as the method when SSL self-signed certificate is not used.



.. _RestClientHowToUseAuthentication:

Basic authentication
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Implementation is as below when a server requests a basic authentication.

**Implementation example of Basic authentication**

Field declaration part

.. code-block:: java


    @Value("${api.auth.userid}")
    String userid;

    @Value("${api.auth.password}")
    String password;


Internal method

.. code-block:: java

    String plainCredentials = userid + ":" + password; // (1)
    String base64Credentials = Base64.getEncoder()
            .encodeToString(plainCredentials.getBytes(StandardCharsets.UTF_8)); // (2)

    RequestEntity requestEntity = RequestEntity
          .get(uri)
          .header("Authorization", "Basic " + base64Credentials) // (3)
          .build();

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Connect user ID and password with "\ ``":"``\ ".
    * - | (2)
      - | Convert (1) to byte array and perform Base64 encoding.
    * - | (3)
      - | Authorization header specifies credentials information of Basic authentication.

.. note::

  \ ``java.util.Base64``\  of Java standard is used for Java SE8 and later versions. Earlier, \ ``org.springframework.security.crypto.codec.Base64``\ of Spring Security is used.


.. _RestClientHowToUseFileUpload:

File upload (multi-part request)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Implementation is as below when file is to be uploaded (multi-part request) using ``RestTemplate``\ .

**Implementation example for file upload**

.. code-block:: java

  MultiValueMap<String, Object> multiPartBody = new LinkedMultiValueMap<>();//(1)
  multiPartBody.add("file", new ClassPathResource("/uploadFiles/User.txt"));//(2)

  RequestEntity<MultiValueMap<String, Object>> requestEntity = RequestEntity
          .post(uri)
          .contentType(MediaType.MULTIPART_FORM_DATA)//(3)
          .body(multiPartBody);//(4)


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Generate \ ``MultiValueMap``\  for storing data sent as a multi-part request.
    * - | (2)
      - | Specify parameter name in key and add file to be uploaded in \ ``MultiValueMap``\ .
        | In the example above, file placed on the class path is added as an uploaded file by specifying parameter name as \ ``file``\ .
    * - | (3)
      - | Specify media type of Content-Type header in \ ``multipart/form-data``\ .
    * - | (4)
      - | Specify \ ``MultiValueMap``\  in the request body wherein the uploaded file has been stored.

.. note:: **Regarding Resource class offered by Spring Framework**

    Spring Framework offers \ ``org.springframework.core.io.Resource``\  as an interface which represents the resource and
    can be used while uploading a file.

    Main implementation classes of \ ``Resource``\  interface are as below.

    * ``org.springframework.core.io.PathResource``
    * ``org.springframework.core.io.FileSystemResource``
    * ``org.springframework.core.io.ClassPathResource``
    * ``org.springframework.core.io.UrlResource``
    * ``org.springframework.core.io.InputStreamResource`` (file name cannot be linked to server)
    * ``org.springframework.core.io.ByteArrayResource`` (file name cannot be linked to server)


.. _RestClientHowToUseFileDownload:

File download
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Implementation is as below when file is to be downloaded using ``RestTeamplate``\ .

**Implementation example of file download (when file size is small)**

.. code-block:: java

    RequestEntity requestEntity = RequestEntity
            .get(uri)
            .build();

    ResponseEntity<byte[]> responseEntity =
            restTemplate.exchange(requestEntity, byte[].class);//(1)

    byte[] downloadContent = responseEntity.getBody();//(2)


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Handle downloaded file with a specified data type. Here, byte array is specified.
    * - | (2)
      - | Fetch data of downloaded file from response body.

.. warning:: **Precautions to be taken while downloading a large file**

    If a large file is fetched in \ ``byte``\  array using \ ``HttpMessageConverter``\  registered as default, \ ``java.lang.OutOfMemoryError``\  is likely to occur.
    Hence, when a large file is to be downloaded, it is necessary to write downloaded data to the file in parts by fetching \ ``InputStream``\  from response.


.. _RestClientHowToUseBigFileDownload:

**Implementation example of file download (when file size is large)**

.. code-block:: java

    // (1)
    final ResponseExtractor<ResponseEntity<File>> responseExtractor = 
            new ResponseExtractor<ResponseEntity<File>>() {

        // (2)
        @Override
        public ResponseEntity<File> extractData(ClientHttpResponse response)
                throws IOException {
            
            File rcvFile = File.createTempFile("rcvFile", "zip");

            FileCopyUtils.copy(response.getBody(), new FileOutputStream(rcvFile));
            
            return ResponseEntity.status(response.getStatusCode())
                    .headers(response.getHeaders()).body(rcvFile);
        }

    };

    // (3)
    ResponseEntity<File> responseEntity = this.restTemplate.execute(targetUri,
            HttpMethod.GET, null, responseExtractor);
    if (HttpStatus.OK.equals(responseEntity.getStatusCode())) {
        File getFile = responseEntity.getBody();
        
        .....
        
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Create a process to create a return value of \ ``RestTemplate#execute``\ ,from the response fetched from \ ``RestTemplate#execute``\ .
    * - | (2)
      - | Read data from response body (\ ``InputStream``\ ) and create a file.
        | Created file, HTTP header and status code are stored in \ ``ResponseEntity<File>``\  and returned.
    * - | (3)
      - | Download file using \ ``RestTemplate#execute``\ .


**Implementation example of file download (when file size is large (example wherein ResponseEntity is not used))**
  
When status code determination and HTTP header reference are not required, \ ``File``\  should be returned instead of \ ``ResponseEntity``\  as given below.
  
.. code-block:: java

    final ResponseExtractor<File> responseExtractor = new ResponseExtractor<File>() {

        @Override
        public File extractData(ClientHttpResponse response)
                throws IOException {

            File rcvFile = File.createTempFile("rcvFile", "zip");

            FileCopyUtils.copy(response.getBody(), new FileOutputStream(
                    rcvFile));

            return rcvFile;
        }

    };

    File getFile = this.restTemplate.execute(targetUri, HttpMethod.GET,
            null, responseExtractor);
    .....


.. _RestClientHowToUseRestFull:

How to handle RESTful URL (URI template) and implementation example
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Implementation can be carried out by using URI template for handling RESTful URL.

**How to use getForObject method**

Field declaration part

.. code-block:: java

    @Value("${api.serverUrl}/api/users/{userId}") // (1)
    String uriStr;


Internal method

.. code-block:: java

    User user = restTemplate.getForObject(uriStr, User.class, "0001"); // (2)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Variable {userId} of URI template is changed to value specified while using ``RestTeamplate``\ .
    * - | (2)
      - | One variable of URI template is replaced with a value specified in third argument of ``getForObject``\  method and processed as "http://localhost:8080/api/users/0001".


**How to use exchange method**

.. code-block:: java

    @Value("${api.serverUrl}/api/users/{action}") // (1)
    String uriStr;


Internal method

.. code-block:: java

    URI targetUri = UriComponentsBuilder.fromUriString(uriStr).
            buildAndExpand("create").toUri(); //(2)

    User user = new User();

    //...

    RequestEntity<User> requestEntity = RequestEntity
            .post(targetUri)
            .body(user);

    ResponseEntity<User> responseEntity = restTemplate.exchange(requestEntity, User.class);


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Variable {action} of URI template is changed to value specified while using ``RestTeamplate``\ .
    * - | (2)
      - | By using ``UriComponentsBuilder``\ , first variable of URI template is replaced by value specified in the argument of ``buildAndExpand``\  and "http://localhost:8080/api/users/create" URI is created.
        | Refer Javadoc of \ `UriComponentsBuilder <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/web/util/UriComponentsBuilder.html>`_\  for details.





.. _RestClientHowToExtend:

How to extend
--------------------------------------------------------------------------------

This chapter explains how to extend \ ``RestTemplate``\ .

.. _RestClientHowToExtendHttpMessageConverter:

How to register an arbitrary \ ``HttpMessageConverter``\ 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If the requirements of message conversion are not met by \ ``HttpMessageConverter``\  registered as default, an arbitrary \ ``HttpMessageConverter``\  can be registered.
However, since \ ``HttpMessageConverter``\  registered as default is deleted, the required \ ``HttpMessageConverter``\  should all be individually registered.

**How to define a bean definition file (applicationContext.xml)**

.. code-block:: xml

    <bean id="jaxb2CollectionHttpMessageConverter"
          class="org.springframework.http.converter.xml.Jaxb2CollectionHttpMessageConverter" /> <!-- (1) -->

    <bean id="restTemplate" class="org.springframework.web.client.RestTemplate">
        <property name="messageConverters"> <!-- (2) -->
            <list>
                <ref bean="jaxb2CollectionHttpMessageConverter" />
            </list>
        </property>
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Define a bean for implementation class of \ ``HttpMessageConverter``\  to be registered.
    * - | (2)
      - | Inject \ ``HttpMessageConverter``\  bean registered in \ ``messageConverters``\ property.


.. _RestClientHowToExtendClientHttpRequestInterceptor:

Application of common process (\ ``ClientHttpRequestInterceptor``\ )
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

By using ``ClientHttpRequestInterceptor``\ , a process can be executed before and after communicating with the server.

Here, implementation example for

* :ref:`RestClientHowToExtendClientHttpRequestInterceptorLogging`
* :ref:`RestClientHowToExtendClientHttpRequestInterceptorBasicAuthentication`

is introduced below.

.. _RestClientHowToExtendClientHttpRequestInterceptorLogging:

Logging process
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When a log for communication with server is to be output, implementation is as below.

**Implementation example of communication log output**

.. code-block:: java

    package com.example.restclient;

    import org.springframework.http.HttpRequest;
    import org.springframework.http.client.ClientHttpRequestExecution;
    import org.springframework.http.client.ClientHttpRequestInterceptor;
    import org.springframework.http.client.ClientHttpResponse;

    public class LoggingInterceptor implements ClientHttpRequestInterceptor { //(1)

        private static final Logger log = LoggerFactory.getLogger(LoggingInterceptor.class);

        @Override
        public ClientHttpResponse intercept(HttpRequest request, byte[] body,
                ClientHttpRequestExecution execution) throws IOException {

            if (log.isInfoEnabled()) {
                String requestBody = new String(body, StandardCharsets.UTF_8);

                log.info("Request Header {}", request.getHeaders()); //(2)
                log.info("Request Body {}", requestBody);
            }

            ClientHttpResponse response = execution.execute(request, body); //(3)
          
            if (log.isInfoEnabled()) {
                log.info("Response Header {}", response.getHeaders()); // (4)
                log.info("Response Status Code {}", response.getStatusCode()); // (5)
            }

            return response; // (6)
        }

    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Implement \ ``ClientHttpRequestInterceptor``\  interface.
    * - | (2)
      - | Implement a common process to be carried out prior to sending a request.
        | In the implementation example above, details of request header and request body are output in a log.
    * - | (3)
      - | Run \ ``execute``\  method of \ ``ClientHttpRequestExecution``\  received as an argument for \ ``intercept``\  method and send a request.
    * - | (4)
      - | Implement a common process which is to be carried out after receiving a response.
        | In the implementation example above, response header details are output in a log.
    * - | (5)
      - | Similar to (4), status code details are output in a log.
    * - | (6)
      - | Return the response received in (3).


.. _RestClientHowToExtendClientHttpRequestInterceptorBasicAuthentication:

Process to configure a request header for Basic authentication
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When it is necessary to configure a request header for Basic authentication to access the server, implementation is as given below.

**Implementation example of request header configuration process for Basic authentication**

.. code-block:: java

    package com.example.restclient;

    import org.springframework.http.HttpRequest;
    import org.springframework.http.client.ClientHttpRequestExecution;
    import org.springframework.http.client.ClientHttpRequestInterceptor;
    import org.springframework.http.client.ClientHttpResponse;

    public class BasicAuthInterceptor implements ClientHttpRequestInterceptor { //(1)

        private static final Logger log = LoggerFactory.getLogger(BasicAuthInterceptor.class);

        @Value("${api.auth.userid}")
        String userid;

        @Value("${api.auth.password}")
        String password;

        @Override
        public ClientHttpResponse intercept(HttpRequest request, byte[] body,
                ClientHttpRequestExecution execution) throws IOException {
          
            String plainCredentials = userid + ":" + password;
            String base64Credentials = Base64.getEncoder()
                    .encodeToString(plainCredentials.getBytes(StandardCharsets.UTF_8));
            request.getHeaders().add("Authorization", "Basic " + base64Credentials); // (1)

            ClientHttpResponse response = execution.execute(request, body);
          
            return response;
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Add a request header for Basic authentication in ``intercept``\  method.


Applying \ ``ClientHttpRequestInterceptor``\
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When \ ``ClientHttpRequestInterceptor``\  created in \ ``RestTemplate``\  is to be applied, define a bean as given below.

**How to define a bean definition file (applicationContext.xml)**

.. code-block:: xml

    <!-- (1) -->
    <bean id="basicAuthInterceptor" class="com.example.restclient.BasicAuthInterceptor" />
    <bean id="loggingInterceptor" class="com.example.restclient.LoggingInterceptor" />

    <bean id="restTemplate" class="org.springframework.web.client.RestTemplate">
        <property name="interceptors"><!-- (2) -->
            <list>
                <ref bean="basicAuthInterceptor" />
                <ref bean="loggingInterceptor" />
            </list>
        </property>
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Define a bean for implementation class of \ ``ClientHttpRequestInterceptor``\ .
    * - | (2)
      - | Inject \ ``ClientHttpRequestInterceptor``\  bean in ``interceptors``\  property.
        | When multiple beans are to be injected, execute the process in a chain sequence starting from top of the list.
        | In the example above, processes prior to request are implemented in the sequence - \ ``BasicAuthInterceptor``\  -> \ ``LoggingInterceptor``\  -> \ ``ClientHttpRequest``\ . (the sequence will be reversed for the processes after receiving a response)


.. _RestClientAsync:

Asynchronous request
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When an asynchronous request is to be carried out, \ ``org.springframework.web.client.AsyncRestTemplate``\  is used.

.. _RestClientAsyncBeanDefinition:

Bean definition for \ ``AsyncRestTemplate``\
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Define a bean for ``AsyncRestTemplate``\ .

**How to define a bean definition file (applicationContext.xml)**

.. code-block:: xml

    <bean id="asyncRestTemplate" class="org.springframework.web.client.AsyncRestTemplate" /> <!-- (1) -->


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | When \ ``AsyncRestTemplate``\  is to be used as per default setup, register a bean by using a default constructor.
        | In case of default configuration, \ ``SimpleClientHttpRequestFactory``\  which has set \ ``org.springframework.core.task.SimpleAsyncTaskExecutor``\  is set as \ ``org.springframework.core.task.AsyncListenableTaskExecutor``\  in \ ``org.springframework.http.client.AsyncClientHttpRequestFactory``\  of \ ``AsyncRestTemplate``\ .


.. note:: **Applying ClientHttpRequestInterceptor to AsyncRestTemplate**

    \ ``ClientHttpRequestInterceptor``\  cannot be applied in \ ``AsyncRestTemplate``\ .
    Hence, a common process must be executed independently.


.. note:: **How to customise AsyncRestTemplate**

    \ ``SimpleAsyncTaskExecutor``\  set as default generates threads without using a thread pool
    and there is no restriction on number of concurrent execution of threads.
    Hence, when the number of threads to be used concurrently is very large, OutOfMemoryError is likely to occur.
    
    By setting a Bean of \ ``org.springframework.core.task.AsyncListenableTaskExecutor``\  interface, in the constructor of \ ``AsyncRestTemplate``\, upper limit for thread pool count can be specified.
    An example of setting \ ``org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor``\ is given below.

     .. code-block:: xml

        <!-- (1) -->
        <bean id="asyncTaskExecutor" class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor">
            <property name="maxPoolSize" value="100" />
        </bean>

        <!-- (2) -->
        <bean id="asyncRestTemplate" class="org.springframework.web.client.AsyncRestTemplate" >
            <constructor-arg index="0" ref="asyncTaskExecutor" />
        </bean>


     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90

        * - Sr. No.
          - Description
        * - | (1)
          - | Define a bean for \ ``AsyncTaskExecutor``\ .
            | Thread operation using a thread pool is carried out by using \ ``ThreadPoolTaskExecutor``\ .
            | Further, number of threads can be controlled by setting \ ``maxPoolSize``\  property.
        * - | (2)
          - | Define a bean for \ ``AsyncRestTemplate``\ .
            | Register a bean by using a constructor which specifies \ ``ThreadPoolTaskExecutor``\ in the argument.

    This guideline introduces an implementation example to customise the task execution process only, however
    HTTP communication process can also be customised for \ ``AsyncRestTemplate``\ .
    Refer Javadoc of \ `AsyncRestTemplate <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/web/client/AsyncRestTemplate.html>`_\  for details.
    
    Also, customisation for other than thread pool size is possible for \ ``ThreadPoolTaskExecutor``\ as well.
    Refer Javadoc of \ `ThreadPoolTaskExecutor <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/scheduling/concurrent/ThreadPoolTaskExecutor.html>`_\  for details.



.. _RestClientAsyncImplementation:

Implementation of asynchronous request
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**Implementation example of asynchronous request**

Field declaration part

.. code-block:: java

    @Inject
    AsyncRestTemplate asyncRestTemplate;


Internal method

.. code-block:: java

    ListenableFuture<ResponseEntity<User>> responseEntity =
            asyncRestTemplate.getForEntity(uri, User.class); // (1)

    responseEntity.addCallback(new ListenableFutureCallback<ResponseEntity<User>>() { // (2)
        @Override
        public void onSuccess(ResponseEntity<User> entity) {
            //...
        }

        @Override
        public void onFailure(Throwable t) {
          //...
        }
    });


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Send asynchronous request by using each method of ``AsyncRestTemplate``\ .
        | In the implementation example above, \ ``getForEntity``\  method is used.
        | ``ResponseEntity``\  wrapped in \ ``org.springframework.util.concurrent.ListenableFuture``\ is sent as return value.
        | How to use each method is similar to \ ``RestTemplate``\ .
    * - | (2)
      - | Register \ ``org.springframework.util.concurrent.ListenableFutureCallback``\  in ``ListenableFuture``\  and implement a process when a response has returned.
        | Implement the process in \ ``onSuccess``\  method when a successful response has returned and implement a process in \ ``onFailure``\ when an error has occurred.


.. _RestClientAppendix:

Appendix
--------------------------------------------------------------------------------

.. _RestClientProxySettings:

How to configure HTTP Proxy server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When the server is to be accessed via HTTP Proxy server, HTTP Proxy server must be configured in system  property or JVM starting argument, or Bean definition of \ ``RestTemplate``\.
When the server is configured in system property or JVM starting argument, it impacts the overall application. Hence, an example wherein HTTP Proxy server is configured for each \ ``RestTemplate``\  is introduced.

HTTP Proxy server for each \ ``RestTemplate``\  can be configured for \ ``SimpleClientHttpRequestFactory``\  which is a default implementation of \ ``ClientHttpRequestFactory``\  interface.
However, since credentials cannot be configured in \ ``SimpleClientHttpRequestFactory``\, \ ``HttpComponentsClientHttpRequestFactory``\  is used while authenticating Proxy.
\ ``HttpComponentsClientHttpRequestFactory``\  is an implementation class of \ ``ClientHttpRequestFactory``\  interface which generates a request using \ ``Apache HttpComponents HttpClient``\.

How to specify a HTTP Proxy server
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Connection destination of HTTP Proxy server for which credentials are essential is specified for \ ``RestTemplate``\  by using \ ``HttpComponentsClientHttpRequestFactory``\.

**pom.xml**

.. code-block:: xml

    <!-- (1) -->
    <dependency>
        <groupId>org.apache.httpcomponents</groupId>
        <artifactId>httpclient</artifactId>
    </dependency>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Add \ ``Apache HttpComponents Client``\  to dependent library of :file:`pom.xml`\  in order to use \ ``Apache HTTP Client``\  which is used in ``HttpComponentsClientHttpRequestFactory``\.

.. note::  
	
	In the above setting example, since it is assumed that the dependent library version is managed by the parent project terasoluna-gfw-parent , specifying the version in pom.xml is not necessary.
	The above dependent library used by terasoluna-gfw-parent is defined by \ `Spring IO Platform <http://platform.spring.io/platform/>`_\ .

**Bean definition file**

.. code-block:: xml

    <!-- (1) -->
    <bean id="proxyHttpClientBuilder" class="org.apache.http.impl.client.HttpClientBuilder" factory-method="create" >
        <!-- (2) -->
        <property name="proxy">
            <bean class="org.apache.http.HttpHost" >
                <constructor-arg index="0" value="${rscl.http.proxyHost}" />    <!-- (3) -->
                <constructor-arg index="1" value="${rscl.http.proxyPort}" />    <!-- (4) -->
            </bean>
        </property>
    </bean>

    <!-- (5) -->
    <bean id="proxyRestTemplate" class="org.springframework.web.client.RestTemplate" >
        <constructor-arg>
            <!-- (6) -->
            <bean class="org.springframework.http.client.HttpComponentsClientHttpRequestFactory">
                <!-- (7) -->
                <constructor-arg>
                    <bean factory-bean="proxyHttpClientBuilder" factory-method="build" />
                </constructor-arg>
            </bean>
        </constructor-arg>
    </bean>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Use \ ``org.apache.http.impl.client.HttpClientBuilder``\  and  configure \ ``org.apache.http.client.HttpClient``\.
    * - | (2)
      - | Configure \ ``org.apache.http.HttpHost``\  which performs HTTP Proxy server setting, in \ ``proxy``\  property of \ ``HttpClientBuilder``\.
    * - | (3)
      - | Set value of key \ ``rscl.http.proxyHost``\  configured in property file in the first argument of \ ``HttpHost``\  constructor, as a host name of HTTP Proxy server.
    * - | (4)
      - | Set value of key \ ``rscl.http.proxyPort``\  configured in property file in the second argument of \ ``HttpHost``\  constructor, as a port number of HTTP Proxy server.
    * - | (5)
      - | Define a Bean for \ ``RestTemplate``\.
    * - | (6)
      - | Configure \ ``HttpComponentsClientHttpRequestFactory``\  in the argument of \ ``RestTemplate``\  constructor.
    * - | (7)
      - | Configure \ ``HttpClient``\  generated from  \ ``HttpClientBuilder``\  in the argument of \ ``HttpComponentsClientHttpRequestFactory``\  constructor.


How to specify credentials information of HTTP Proxy server
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When credentials (user name and password) are required for accessing HTTP Proxy server, credentials are set by using \ ``org.apache.http.impl.client.BasicCredentialsProvider``\.

Since \ ``setCredentials``\  method of \ ``BasicCredentialsProvider``\  contains two arguments, a Bean cannot be generated by using a setter injection. Hence, a Bean is generated by using \ ``org.springframework.beans.factory.FactoryBean``\.

**FactoryBean class**

.. code-block:: java

    import org.apache.http.auth.AuthScope;
    import org.apache.http.auth.UsernamePasswordCredentials;
    import org.apache.http.impl.client.BasicCredentialsProvider;
    import org.springframework.beans.factory.FactoryBean;
    import org.springframework.beans.factory.annotation.Value;

    // (1)
    public class BasicCredentialsProviderFactoryBean implements FactoryBean<BasicCredentialsProvider> {

        // (2)
        @Value("${rscl.http.proxyHost}")
        String host;

        // (3)
        @Value("${rscl.http.proxyPort}")
        int port;

        // (4)
        @Value("${rscl.http.proxyUserName}")
        String userName;

        // (5)
        @Value("${rscl.http.proxyPassword}")
        String password;

        @Override
        public BasicCredentialsProvider getObject() throws Exception {

            // (6)
            AuthScope authScope = new AuthScope(this.host, this.port);

            // (7)
            UsernamePasswordCredentials usernamePasswordCredentials =
                    new UsernamePasswordCredentials(this.userName, this.password);

            // (8)
            BasicCredentialsProvider credentialsProvider = new BasicCredentialsProvider();
            credentialsProvider.setCredentials(authScope, usernamePasswordCredentials);

            return credentialsProvider;
        }

        @Override
        public Class<?> getObjectType() {
            return BasicCredentialsProvider.class;
        }

        @Override
        public boolean isSingleton() {
            return true;
        }
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Define a \ ``BasicCredentialsProviderFactoryBean``\  class which implements \ ``org.springframework.beans.factory.FactoryBean``\.
        | Configure \ ``BasicCredentialsProvider``\  in the type of Bean.
    * - | (2)
      - | Set value of key \ ``rscl.http.proxyHost``\  set in property file in instance variable, as a host name of HTTP Proxy server.
    * - | (3)
      - | Set value of key \ ``rscl.http.proxyPort``\  set in property file in instance variable, as a port number of HTTP Proxy server.
    * - | (4)
      - | Set value of key \ ``rscl.http.proxyUserName``\  set in property file in instance variable, as a user name of HTTP Proxy server.
    * - | (5)
      - | Set value of key \ ``rscl.http.proxyPassword``\  set in property file in instance variable, as a password of HTTP Proxy server.
    * - | (6)
      - | Create \ ``org.apache.http.auth.AuthScope`` \  and configure scope of credentials. This example specifies host name and port number of HTTP Proxy server. For other configuration methods, refer \ `AuthScope (Apache HttpClient API) <https://hc.apache.org/httpcomponents-client-ga/httpclient/apidocs/org/apache/http/auth/AuthScope.html>`_\.
    * - | (7)
      - | Create \ ``org.apache.http.auth.UsernamePasswordCredentials`` \ and configure credentials.
    * - | (8)
      - | Create \ ``org.apache.http.impl.client.BasicCredentialsProvider``\  and configure credentials and its scope by using \ ``setCredentials``\  method.


**Bean definition file**

.. code-block:: xml

    <bean id="proxyHttpClientBuilder" class="org.apache.http.impl.client.HttpClientBuilder" factory-method="create">
        <!-- (1) -->
        <property name="defaultCredentialsProvider">
            <bean class="com.example.restclient.BasicCredentialsProviderFactoryBean" />
        </property>
        <property name="proxy">
            <bean id="proxyHost" class="org.apache.http.HttpHost">
                <constructor-arg index="0" value="${rscl.http.proxyHost}" />
                <constructor-arg index="1" value="${rscl.http.proxyPort}" />
            </bean>
        </property>
    </bean>

    <bean id="proxyRestTemplate" class="org.springframework.web.client.RestTemplate">
        <constructor-arg>
            <bean class="org.springframework.http.client.HttpComponentsClientHttpRequestFactory">
                <constructor-arg>
                    <bean factory-bean="proxyHttpClientBuilder" factory-method="build" />
                </constructor-arg>
            </bean>
        </constructor-arg>
    </bean>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Configure \ ``BasicCredentialsProvider``\ in \ ``defaultCredentialsProvider``\ property of \ ``HttpClientBuilder``\.
        | \ ``BasicCredentialsProvider`` creates a Bean by using \ ``BasicCredentialsProviderFactoryBean``\ which implements \ ``FactoryBean``\.
        

Configuration while using JSR-310 Date and Time API in JSON
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For configuration while using JSR-310 Date and Time API as a property of JavaBean which represents a resource (Resource class),
refer "\ :ref:`RESTAppendixUsingJSR310_JodaTime` \".
