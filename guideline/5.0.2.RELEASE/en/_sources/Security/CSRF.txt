CSRF Countermeasures
================================================================================

.. only:: html

 .. contents:: Table of contents
    :local:
    
Overview
--------------------------------------------------------------------------------

| Cross-Site Request Forgery (hereinafter CSRF) is an attack that forces a user to perform unwanted actions on a different website in which the user is authenticated.
| This is usually carried out by executing a script or by automatic transfer (HTTP redirect).

| Following are the methods to prevent server side CSRF attack.

* Embedding confidential information (token)
* Re-entering password
* 'Refer' check

| In OWASP, the method to use token pattern is \ `recommended <https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)_Prevention_Cheat_Sheet#General_Recommendation:_Synchronizer_Token_Pattern>`_\ .

.. figure:: ./images/csrf_check_other_site.png
   :alt: csrf check other site
   :width: 60%
   :align: center

   **Picture - csrf check other site**

.. note::

  **About OWASP**

  Open Web Application Security Project (OWASP) is not-for-profit international organization dedicated to
  enable organizations to develop and maintain applications that can be trusted. It advocates measures such as effective approach etc. with respect to security.

    https://www.owasp.org/index.php/Main_Page

| As mentioned earlier, there are multiple ways to avoid CSRF. However, Spring Security provides a library that uses fixed tokens.
| In this, one fixed token is used for each session and the same value is used for all requests.

| When the default HTTP method is other than GET, HEAD, TRACE and OPTIONS,
| CSRF token included in the request is checked and if the values do not match, an error (HTTP Status: 403 [Forbidden]) is thrown.

.. figure:: ./images/csrf_check_kind.png
   :alt: csrf check other kind
   :width: 50%
   :align: center
 
**Picture - csrf check other kind**

.. tip::

  CSRF token check is the check that verifies an invalid update request from a different site and throws an error.
  To check and enforce users to maintain the order (series of business flows), refer \ :ref:`double-submit_transactiontokencheck`\ .

.. warning:: **CSRF measures at the time of the multi-part request (file upload)**

    About CSRF measures during file upload, \ :ref:`file upload Servlet Filter setting <file-upload_setting_servlet_filter>`\ should be followed.




|

How to use
--------------------------------------------------------------------------------

Spring Security settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Settings to use the CSRF functionality provided by Spring Security, are explained here.
Use of web.xml configured using \ :ref:`Spring Security's How to use <howtouse_springsecurity>`\ , is assumed.

.. _csrf_spring-security-setting:

spring-security.xml settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The locations where additional settings are required, are highlighted.


.. code-block:: xml
   :emphasize-lines: 3-4,8-

    <sec:http auto-config="true" use-expressions="true" >
        <!-- omitted -->
        <sec:csrf />  <!-- (1) -->
        <sec:access-denied-handler ref="accessDeniedHandler"/>  <!-- (2) -->
        <!-- omitted -->
    </sec:http>

    <bean id="accessDeniedHandler"
        class="org.springframework.security.web.access.DelegatingAccessDeniedHandler">  <!-- (3) -->
        <constructor-arg index="0">  <!-- (4) -->
            <map>
                <entry
                    key="org.springframework.security.web.csrf.InvalidCsrfTokenException">  <!-- (5) -->
                    <bean
                        class="org.springframework.security.web.access.AccessDeniedHandlerImpl">  <!-- (5) -->
                        <property name="errorPage"
                            value="/WEB-INF/views/common/error/invalidCsrfTokenError.jsp" />  <!-- (5) -->
                    </bean>
                </entry>
                <entry
                    key="org.springframework.security.web.csrf.MissingCsrfTokenException">  <!-- (6) -->
                    <bean
                        class="org.springframework.security.web.access.AccessDeniedHandlerImpl">  <!-- (6) -->
                        <property name="errorPage"
                            value="/WEB-INF/views/common/error/missingCsrfTokenError.jsp" />  <!-- (6) -->
                    </bean>
                </entry>
            </map>
        </constructor-arg>
        <constructor-arg index="1">  <!-- (7) -->
            <bean
                class="org.springframework.security.web.access.AccessDeniedHandlerImpl">  <!-- (8) -->
                <property name="errorPage"
                    value="/WEB-INF/views/common/error/accessDeniedError.jsp" />  <!-- (8) -->
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
     - | Spring Security's CSRF token check functionality can be used by defining \ ``<sec:csrf>``\  element in \ ``<sec:http>``\  element.
       | For HTTP methods that are checked by default, refer to \ :ref:`Here <csrf_default-add-token-method>`\ .
       | For details, refer to \ `Spring Security Reference Document <http://docs.spring.io/spring-security/site/docs/3.2.9.RELEASE/reference/htmlsingle/#csrf-configure>`_\ .
   * - | (2)
     - | Define Handler that changes the view to be displayed according to each type of exception when the exception inheriting \ ``AccessDeniedException``\  occurs.
       | It is possible to display all the exceptions on the same screen by specifying the destination jsp in ``error-page`` attribute.
       | Refer to \ :ref:`Here <csrf_403-webxml-setting>`\  when the exceptions are not handled by Spring Security functionality.
   * - | (3)
     - | To change error page, specify \ ``org.springframework.security.web.access.DelegatingAccessDeniedHandler``\  in class of the Handler provided by Spring Security.
   * - | (4)
     - | Using the first argument of constructor, set the screen that changes the display with each type of exception other than the default exceptions (exception that inherits \ ``AccessDeniedException``\ ) in Map format.
   * - | (5)
     - | Specify the exception that inherits  \ ``AccessDeniedException``\ , in key.
       | Specify \ ``org.springframework.security.web.access.AccessDeniedHandlerImpl`` provided by Spring Security as the implementation class.
       | Specify the view to be displayed in value, by specifying errorPage in property name.
   * - | (6)
     - | Define the change in display if the exception type differs from (5).
   * - | (7)
     - | Specify the default view (exception that inherits \ ``AccessDeniedException``\  and which is not specified by the first argument of constructor and \ ``AccessDeniedException``\ ) using second argument of constructor.
   * - | (8)
     - | Specify \ ``org.springframework.security.web.access.AccessDeniedHandlerImpl`` provided by Spring Security, as the implementation class.
       | Specify the view to be displayed in value, by specifying errorPage in property name.

|

.. tabularcolumns:: |p{0.40\linewidth}|p{0.60\linewidth}|
.. list-table:: Type of exception occurred due to CSRF countermeasure, with inherited \ ``AccessDeniedException``\ .
   :header-rows: 1
   :widths: 40 60

   * - Exception
     - Reason 
   * - | org.springframework.security.web.csrf.
       | InvalidCsrfTokenException
     - | It occurs when the CSRF token requested from client does not match with the CSRF token stored at server side.
   * - | org.springframework.security.web.csrf.
       | MissingCsrfTokenException
     - | It occurs when the CSRF token does not exist at server side.
       | As per default setting, since the token is stored in HTTP session, having no CSRF token implies that the HTTP session is discarded (session timed out).
       |
       | \ ``MissingCsrfTokenException``\  occurs when storage location of CSRF token is changed to cache server or DB using \ ``token-repository-ref``\  attribute of \ ``<sec:csrf>``\  element, and when CSRF token is deleted from the storage location.
       | It implies that when token is not stored in HTTP session, session timeout cannot be detected using this functionality.

.. note::

    When HTTP session is to be used as storage location of CSRF token,
    the session timeout can be detected for the request of CSRF token check.

    The operation after detecting session timeout differs depending on the specifications of \ ``invalid-session-url``\  attribute of \ ``<session-management>``\  element.

    * When \ ``invalid-session-url``\  attribute is specified, it is redirected to the path specified in \ ``invalid-session-url``\  once the session is created.
    * When \ ``invalid-session-url``\  attribute is not specified, it is handled according to the definition of \ ``org.springframework.security.web.access.AccessDeniedHandler``\  specified in \ ``<access-denied-handler>``\  element.

    When session timeout needs to be detected for the request which does not fall under CSRF token check,
    it is advisable to detect it by specifying \ ``invalid-session-url``\  attribute of \ ``<session-management>``\  element.
    For details, refer to ":ref:`authentication_session-timeout`".

|

.. _csrf_403-webxml-setting:

.. note::

  **Error handling when <sec:access-denied-handler> settings are omitted**

  It is possible to transit to any page by performing the following settings in web.xml.

  **web.xml**

    .. code-block:: xml

        <error-page>
            <error-code>403</error-code>  <!-- (1) -->
            <location>/WEB-INF/views/common/error/accessDeniedError.jsp</location>  <!-- (2) -->
        </error-page>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - Sr. No.
         - Description
       * - | (1)
         - | Set status code 403 in error-code element.
       * - | (2)
         - | Set transition path in location element.

.. _csrf_change-httpstatus403:

.. note::

  **When status code other than 403 is to be returned**

  If status code other than 403 is to be returned when the CSRF token in request does not match, it is necessary to create an independent AccessDeniedHandler
  that implements  \ ``org.springframework.security.web.access.AccessDeniedHandler``\  interface.

.. _csrf_spring-mvc-setting:

spring-mvc.xml settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
By using the ``RequestDataValueProcessor`` implementation class for CSRF token, the token can be automatically inserted as 'hidden' field using ``<form:form>`` tag of Spring tag library.

.. code-block:: xml
   :emphasize-lines: 1-2,5-6

    <bean id="requestDataValueProcessor"
        class="org.terasoluna.gfw.web.mvc.support.CompositeRequestDataValueProcessor"> <!-- (1)  -->
        <constructor-arg>
            <util:list>
                <bean
                    class="org.springframework.security.web.servlet.support.csrf.CsrfRequestDataValueProcessor" /> <!-- (2)  -->
                <bean
                    class="org.terasoluna.gfw.web.token.transaction.TransactionTokenRequestDataValueProcessor" />
            </util:list>
        </constructor-arg>
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Perform bean definition for \ ``org.terasoluna.gfw.web.mvc.support.CompositeRequestDataValueProcessor``\  for which multiple
       | \ ``org.terasoluna.gfw.web.mvc.support.RequestDataValueProcessor``\ can be defined.
   * - | (2)
     - | Set the bean definition \ ``org.springframework.security.web.servlet.support.csrf.CsrfRequestDataValueProcessor``\  as the first argument of constructor.

.. note::

  CSRF token is created and checked by \``CsrfFilter``\  which is enabled by setting \ ``<sec:csrf />``\ . Therefore, the developer need not be particularly aware of the CSRF measures in Controller.

.. _csrf_form-tag-token-send:

Sending CSRF token using form
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To send CSRF token using form in JSP, any one of the following processes need to be performed.

* Automatically adding \ ``<input type="hidden">``\  tag with inserted CSRF token, using \ ``<form:form>``\  tag.
* Explicitly adding \ ``<input type="hidden">``\  tag with inserted CSRF token, using \ ``<sec:csrfInput/>``\  tag.


.. _csrf_formformtag-use:

How to insert CSRF token automatically
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When \ ``CsrfRequestDataValueProcessor``\  is defined as per \ :ref:`spring-mvc.xml settings <csrf_spring-mvc-setting>`\ ,
\ ``<input type="hidden">``\  tag with CSRF token is automatically added using \ ``<form:form>``\  tag.

CSRF token need not be considered in JSP implementation.

.. code-block:: jsp

    <form:form method="POST"
      action="${pageContext.request.contextPath}/csrfTokenCheckExample">
      <input type="submit" name="second" value="second" />
    </form:form>

Following HTML is output.

.. code-block:: html

    <form action="/terasoluna/csrfTokenCheckExample" method="POST">
      <input type="submit" name="second" value="second" />
      <input type="hidden" name="_csrf" value="dea86ae8-58ea-4310-bde1-59805352dec7" /> <!-- (1) -->
    </form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | In Spring Security default implementation, CSRF token is inserted by adding \ ``<input type="hidden">``\  tag where \ ``_csrf``\  is set in \ ``name``\  attribute.


CSRF token is created at login.

.. tip::

    If \ ``CsrfRequestDataValueProcessor``\  is used in Spring 4,
    CSRF token inserted \ ``<input type="hidden">``\  tag is output,
    only if the value specified in \ ``method``\  attribute of \ ``<form:form>``\  tag matches with HTTP methods (HTTP methods other than GET, HEAD, TRACE, OPTIONS in Spring Security default implementation) of CSRF token check.

    For example, when GET method is specified in \ ``method``\  attribute as shown below,
    CSRF token inserted \ ``<input type="hidden">``\  tag is not output. 

        .. code-block:: jsp

            <form:form method="GET" modelAttribute="xxxForm" action="...">
                <%-- ... --%>
            </form:form>

    This is as per the following description

        The unique token can also be included in the URL itself, or a URL parameter. However, such placement runs a greater risk that the URL will be exposed to an attacker, thus compromising the secret token.

    in \ `OWASP Top 10 <https://code.google.com/p/owasptop10/>`_\  and it helps in building a secure Web application.

.. _csrf_formtag-use:

How to explicitly insert CSRF token
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When \ ``<form:form>``\  tag is not to be used, \ ``<sec:csrfInput/>``\  tag needs to be added explicitly.

When using \ ``<sec:csrfInput/>``\  tag, CSRF token inserted \ ``<input type="hidden">``\  tag is output.

.. code-block:: jsp

    <form method="POST"
      action="${pageContext.request.contextPath}/csrfTokenCheckExample">
        <input type="submit" name="second" value="second" />
        <sec:csrfInput/>  <!-- (1) -->
    </form>

Following HTML is output.

.. code-block:: html

    <form action="/terasoluna/csrfTokenCheckExample" method="POST">
      <input type="submit" name="second" value="second" />
      <input type="hidden" name="_csrf" value="dea86ae8-58ea-4310-bde1-59805352dec7"/>  <!-- (2) -->
    </form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify \ ``<sec:csrfInput/>``\  tag to output CSRF token inserted \ ``<input type="hidden">``\  tag.
   * - | (2)
     - | In spring Security default implementation, CSRF token is inserted by adding \ ``<input type="hidden">``\  tag where \ ``_csrf``\  is set in \ ``name``\  attribute.

.. _csrf_default-add-token-method:

.. note::

  When there is no CSRF token in the CSRF token check request (when HTTP default method is other than GET, HEAD, TRACE and OPTIONS) or
  when the token value stored on server is different than the token value sent, access denial process is performed using \ ``AccessDeniedHandler``\  and HttpStatus 403 is returned.
  The specified error page is displayed when \ :ref:`spring-security.xml settings <csrf_spring-security-setting>`\  are described.


.. _csrf_ajax-token-setting:

Sending CSRF token using Ajax
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| \ ``CsrfFilter``\  which is enabled by setting \ ``<sec:csrf />``\ , fetches CSRF token not only from request parameter but also from
| the HTTP request header.
| When Ajax is used, it is recommended to set CSRF token in HTTP header. Even if request is sent in JSON format, it supports CSRF token check.

.. note::
    
    When CSRF token is sent from both the HTTP header and request parameter, HTTP header value is given priority.

| It is explained by using an example in \ :doc:`../ArchitectureInDetail/Ajax`\ . The locations for which additional setting is required are highlighted.

**jsp implementation**

.. code-block:: jsp
   :emphasize-lines: 3-4

    <!-- omitted -->
    <head>
      <sec:csrfMetaTags />  <!-- (1) -->

      <!-- omitted -->
    </head>
    <!-- omitted -->

.. code-block:: jsp
   :emphasize-lines: 3-7

    <script type="text/javascript">
    var contextPath = "${pageContext.request.contextPath}";
    var token = $("meta[name='_csrf']").attr("content");  <!-- (2) -->
    var header = $("meta[name='_csrf_header']").attr("content");  <!-- (3) -->
    $(document).ajaxSend(function(e, xhr, options) {
        xhr.setRequestHeader(header, token);  <!-- (4) -->
    });

    $(function() {
        $('#calcButton').on('click', function() { 
            var $form = $('#calcForm'),
                 $result = $('#result');
            $.ajax({
                url : contextPath + '/sample/calc',
                type : 'POST',
                data: $form.serialize(),
            }).done(function(data) {
                $result.html('add: ' + data.addResult + '<br>'
                             + 'subtract: ' + data.subtractResult + '<br>'
                             + 'multiply: ' + data.multipyResult + '<br>'
                             + 'divide: ' + data.divideResult + '<br>'); // (5)
            }).fail(function(data) {
                // error handling
                alert(data.statusText);
            });
        });
    });
    </script>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | By setting \ ``<sec:csrfMetaTags />``\  tag, the following \ ``meta``\  tags are output by default.

       * \ ``<meta name="_csrf_parameter" content="_csrf" />``\
       * \ ``<meta name="_csrf_header" content="X-CSRF-TOKEN" />``\
       * \ ``<meta name="_csrf" content="dea86ae8-58ea-4310-bde1-59805352dec7" />``\ (random UUID is set as a value of \ ``content``\  attribute)
   * - | (2)
     - | Fetch CSRF token set in \ ``<meta name="_csrf ...>``\  tag.
   * - | (3)
     - | Fetch CSRF header name set in \ ``<meta name="_csrf_header" ...>``\  tag.
   * - | (4)
     - | Set the header name (default:X-CSRF-TOKEN) fetched from \ ``<meta>``\  tag and CSRF token value, in request header.
   * - | (5)
     - | As this code may cause XSS attack, care should be taken while actually writing the JavaScript code.
       | In this example, there is no issue as all the fields namely \ ``data.addResult``\ , \ ``data.subtractResult``\ , \ ``data.multipyResult``\  and \ ``data.divideResult``\  are numerical.

When sending a request in JSON format, it is advisable to set HTTP header in the same way.

.. todo::

  It needs to be modified since \ :doc:`../ArchitectureInDetail/Ajax`\  example is missing.

Considerations at the time of multipart request (file upload)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| Generally, when a multipart request such as file upload is sent, the value sent by form cannot be fetched in \ ``Filter``\ .
| Therefore, just by the explanation so far, \ ``CsrfFilter``\  is unable to fetch CSRF token at the time of multipart request and thus it is considered as an invalid request.

Hence, countermeasures need to be implemented using any one of the following methods.

* Using \ ``org.springframework.web.multipart.support.MultipartFilter``\ .
* Sending CSRF token using query parameter

.. note::

    Since there are merits and demerits respectively, determine the countermeasure method to be used considering system requirements.

For file upload details, refer to \ :doc:`FileUpload <../ArchitectureInDetail/FileUpload>`\ .


.. _csrf_use-multipart-filter:

How to use MultipartFilter
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| In case of multipart request, normally the value sent from form cannot be fetched in \ ``Filter``\ .
| By using \ ``org.springframework.web.multipart.support.MultipartFilter``\ , the value sent by form 
| can be fetched in \ ``Filter``\  even for a multipart request.

.. warning::

    When \ ``MultipartFilter``\  is used, the upload process is carried out before performing authentication/authorization using \ ``springSecurityFilterChain``\ ,
    thereby allowing unauthenticated/unauthorized user to carry out the uploading (temporary file creation).


To use \ ``MultipartFilter``\ , following settings are recommended.

**web.xml settings**

.. code-block:: xml

    <filter>
        <filter-name>MultipartFilter</filter-name>
        <filter-class>org.springframework.web.multipart.support.MultipartFilter</filter-class> <!-- (1) -->
    </filter>
    <filter>
        <filter-name>springSecurityFilterChain</filter-name> <!-- (2) -->
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>MultipartFilter</filter-name>
        <servlet-name>/*</servlet-name>
    </filter-mapping>
    <filter-mapping>
        <filter-name>springSecurityFilterChain</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Define \ ``org.springframework.web.multipart.support.MultipartFilter``\ .
   * - | (2)
     - | \ ``MultipartFilter``\  should be defined before \ ``springSecurityFilterChain``\ .

**JSP implementation**

.. code-block:: jsp

    <form:form action="${pageContext.request.contextPath}/fileupload"
        method="post" modelAttribute="fileUploadForm" enctype="multipart/form-data">  <!-- (1) -->
        <table>
            <tr>
                <td width="65%"><form:input type="file" path="uploadFile" /></td>
            </tr>
            <tr>
                <td><input type="submit" value="Upload" /></td>
            </tr>
        </table>
    </form:form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | When \ ``CsrfRequestDataValueProcessor``\  is defined as per spring-mvc.xml settings,
       | by using \ ``<form:form>``\  tag, \ ``<input type="hidden">``\  tag with CSRF token is added automatically.
       | Therefore, CSRF token need not be considered in JSP implementation.
       | 
       | **When using <form> tag**
       | CSRF token should be set by \ :ref:`csrf_formtag-use`\ .


How to send CSRF token using query parameter
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

To avoid uploading (temporary file creation) by unauthorized/unauthenticated user,
CSRF token should be sent using query parameter instead of using \ ``MultipartFilter``\ .

.. warning::

    When CSRF token is sent using this method,

    * CSRF token is displayed in browser address bar
    * When bookmarked, CSRF token is registered in bookmark
    * CSRF token is registered in access log of Web server 

    Therefore, risk of misusing CSRF token by attacker is higher as compared to the method using \ ``MultipartFilter``\ .

    As per default implementation of Spring Security, random UUID is generated as CSRF token value,
    therefore, session would not be hijacked even though CSRF token is leaked.

Example of implementation where CSRF token is sent as query parameter is given below.

**JSP implementation**

.. code-block:: jsp

    <form:form action="${pageContext.request.contextPath}/fileupload?${f:h(_csrf.parameterName)}=${f:h(_csrf.token)}"
        method="post" modelAttribute="fileUploadForm" enctype="multipart/form-data"> <!-- (1) -->
        <table>
            <tr>
                <td width="65%"><form:input type="file" path="uploadFile" /></td>
            </tr>
            <tr>
                <td><input type="submit" value="Upload" /></td>
            </tr>
        </table>
    </form:form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Following query needs to be assigned to the action attribute of \ ``<form:form>``\  tag.
       | \ ``?${f:h(_csrf.parameterName)}=${f:h(_csrf.token)}``\
       | Same setting is required even when using \ ``<form>``\  tag.

.. raw:: latex

   \newpage
