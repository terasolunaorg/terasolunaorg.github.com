Authorization
================================================================================

.. only:: html

 .. contents:: Table of contents
    :local:

Overview
--------------------------------------------------------------------------------
| This chapter explains about Authorization functionality provided by Spring Security.

| As authorization is implemented using access authorization functionality of Spring Security, it is a pre-requisite that authentication functionality of Spring Security is used.
| For details on authentication method using Spring Security, refer to \ :doc:`Authentication`\ .

| Following 3 are the target resources of access authorization.

#. Web (Request URL)

   * Authority required to access specific URLs can be set.

#. Screen fields (JSP)

   * Authority required to display specific elements in a screen can be set.

#. Method

   * Authority required to execute specific methods in a screen can be set.

| In Spring Security, the functionality is implemented by describing access authorization information in configuration file or by using annotations.

Access authorization (Request URL)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. figure:: ./images/Authorization_Filter_overview.png
   :alt: Authorization (Request URL)
   :width: 60%

   **Picture - Authorization (Request URL)**

#. Spring Security's filter chain interrupts the process for user requests.
#. Target URL for authorization control and request are matched and a query is sent to access authorization manager regarding decision on access authorization.
#. Access authorization manager checks the user authority and access authorization information
   and throws access denial exception when the required role is not assigned.
#. Process is continued if the required role is assigned.

|

Access authorization (JSP)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. figure:: ./images/Authorization_Jsp_overview.png
   :alt: Authorization(JSP)
   :width: 60%

   **Picture - Authorization(JSP)**

#. The servlet generated from JSP inquires with access authorization manager.
#. Access authorization manager checks the user authority and access authorization information.
   If the required role is not assigned, it does not evaluate the internal tag.
#. It evaluates internal tag if the required role is assigned.

|

Access authorization (Method)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. figure:: ./images/Authorization_Method_overview.png
   :alt: Authorization(Method)
   :width: 60%

   **Picture - Authorization(Method)**

#. Spring container generates an interceptor for the target object on the basis of access authorization information and interrupts the process.
#. Interceptor inquires with access authorization manager on the basis of set roles.
#. Access authorization manager checks user authority and access authorization information
   and throws access denial exception when the required role is not assigned.
#. The process is continued if the required role is assigned. (Authority can be checked after executing the process as per the settings).

|

How to use
--------------------------------------------------------------------------------
| How to use access authorization (Request URL), access authorization (JSP) and access authorization (Method) is explained here.

Access authorization (Request URL)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| The contents to be described in the Spring Security configuration file in order to use access authorization (Request URL) functionality, are shown below.
| For basic settings, refer to \ :doc:`SpringSecurity`\ .

.. _authorization-intercept-url:

Setting \ ``<sec:intercept-url>``\  element
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| By describing the target URL to be controlled and the role to be authorized in the \ ``<sec:intercept-url>``\  element which is a child element of \ ``<sec:http>``\  element,
| authorization can be controlled for each URL path.

| Setting example is described below.

* spring-security.xml

  .. code-block:: xml
  
    <sec:http auto-config="true" use-expressions="true">
        <sec:intercept-url pattern="/admin/*" access="hasRole('ROLE_ADMIN')"/>
        <!-- omitted -->
    </sec:http>
  
  .. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 20 80
  
     * - | Attribute name
       - | Description
     * - | \ ``pattern``\
       - | Describes the target URL pattern for access authorization. Wild cards "*" and "**" can be used.
         | "*" is used only for URL patterns of same level while "**" targets all URLs under specified level for authorization setting.
     * - | \ ``access``\
       - | Specifies the access control type and accessible role using Spring EL.
     * - | \ ``method``\
       - | Specifies the HTTP method (GET, POST etc.). Matches only the specified method with URL pattern.
         | When not specified, any HTTP method is applied. It can be utilized in the Web services that mainly use REST.
     * - | \ ``requires-channel``\ 
       - | Specifies "http" or "https". It enforces access to the specified protocol.
         | When not specified, both protocols can be accessed.

  | For attributes other than the above, refer to \ `<intercept-url> <http://docs.spring.io/spring-security/site/docs/3.2.9.RELEASE/reference/htmlsingle/#nsa-intercept-url>`_\ .

| A setting example with roles namely "ROLE_USER" and "ROLE_ADMIN" assigned to login user, is shown below.

* spring-security.xml

  .. code-block:: xml
  
    <sec:http auto-config="true" use-expressions="true">
        <sec:intercept-url pattern="/reserve/*" access="hasAnyRole('ROLE_USER','ROLE_ADMIN')" /> <!-- (1) -->
        <sec:intercept-url pattern="/admin/*" access="hasRole('ROLE_ADMIN')" /> <!-- (2) -->
        <sec:intercept-url pattern="/**" access="denyAll" /> <!-- (3) -->
        <!-- omitted -->
    </sec:http>
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90 
  
     * - | Sr. No.
       - | Description
     * - | (1)
       - | To access "/reserve/\*", either the role "ROLE_USER" or "ROLE_ADMIN" is required.
         | \ ``hasAnyRole``\  will be described later.
     * - | (2)
       - | To access "/admin/\*", the role "ROLE_ADMIN" is required.
         | \ ``hasRole``\  will be described later.
     * - | (3)
       - | \ ``denyAll``\  is set for all patterns
         | and it is set such that no user can access the URLs for which authority settings are not performed.
         | \ ``denyAll``\  will be described later.

  .. note::    **Description order of URL pattern**

   Match the request received from client with the pattern in intercept-url, starting from the top.
   Perform access authorization for the matched pattern. Therefore, pattern should be described from restricted patterns.

| Spring EL is enabled by setting \ ``use-expressions="true"``\  in \ ``<sec:http>``\  attribute.
| Since Spring EL is evaluated by Boolean values, access is authorized when the expression is true.
| Example is shown below.

* spring-security.xml

  .. code-block:: xml

    <sec:http auto-config="true" use-expressions="true">
        <sec:intercept-url pattern="/admin/*" access="hasRole('ROLE_ADMIN')"/>  <!-- (1) -->
        <!-- omitted -->
    </sec:http>
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | By specifying \ ``hasRole('role name')``\ , 'true' is returned if the login user has the specified role.
  
  .. _spring-el:

  | **Example of available expressions list**
  
  .. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 30 70
  
     * - Attribute name
       - Description
     * - | \ ``hasRole('Role name')``\ 
       - | Returns true if the user has the specified role.
     * - | \ ``hasAnyRole('Role 1','Role 2')``\ 
       - | Returns true if the user has any of the specified roles.
     * - | \ ``permitAll``\ 
       - | Always returns true. Note that it is accessible even if not authenticated.
     * - | \ ``denyAll``\ 
       - | Always returns false.
     * - | \ ``isAnonymous()``\ 
       - | Returns true in case of an anonymous user.
     * - | \ ``isAuthenticated()``\ 
       - | Returns true in case of an authenticated user.
     * - | \ ``isFullyAuthenticated()``\ 
       - | Returns false, in case of an anonymous user or authentication by RememberMe functionality.
     * - | \ ``hasIpAddress('IP address')``\ 
       - | Enabled only by the access authorization to request URL and JSP tag.
         | Returns true if there is a request from specified IP address.
  
  | For other available Spring EL, refer to \ `Common built-in expressions <http://docs.spring.io/spring-security/site/docs/3.2.9.RELEASE/reference/htmlsingle/#el-common-built-in>`_\ .

  | Determination can also be performed using operator.
  | In the following example, it can be accessed when it matches with both the role and the requested IP address.

* spring-security.xml

  .. code-block:: xml
  
    <sec:http auto-config="true" use-expressions="true">
        <sec:intercept-url pattern="/admin/*" access="hasRole('ROLE_ADMIN') and hasIpAddress('192.168.10.1')"/>
        <!-- omitted -->
    </sec:http>
  
  | **Available operators list**
  
  .. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 20 80
  
     * - Operator
       - Description
     * - | \ ``[Expression1] and [Expression2]``\ 
       - | Returns true when both Expression1 and Expression2 are true.
     * - | \ ``[Expression1] or [Expression2]``\ 
       - | Returns true when either of the expressions is true.
     * - | \ ``![Expression]``\ 
       - | Returns false when expression is true and true when the expression is false.


Setting the URL for which access authorization is not controlled
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Use pattern attribute and security attribute of http element for
| URLs such as top page and login screen, css file path etc. where authentication is not required.

  * spring-security.xml
  
  .. code-block:: xml
  
    <sec:http pattern="/css/*" security="none"/>  <!-- Perform steps (1) and (2) in the specified attribute order -->
    <sec:http pattern="/login" security="none"/>
    <sec:http auto-config="true" use-expressions="true">
        <!-- omitted -->
    </sec:http>
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - | Sr. No.
       - | Description
     * - | (1)
       - | Describe the target URL pattern for which settings are to be performed in \ ``pattern``\  attribute. When not specifying \ ``pattern``\  attribute, match with all patterns.
     * - | (2)
       - | By specifying \ ``none``\  in \ ``security``\  attribute, Spring Security filter chain can be avoided for the path stated in \ ``pattern``\  attribute.


Exception handling in URL pattern
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| \ ``org.springframework.security.access.AccessDeniedException``\  is thrown when an unauthorized URL is accessed.
| As per default setting, \ ``org.springframework.security.web.access.AccessDeniedHandlerImpl``\  which is set in \ ``org.springframework.security.web.access.ExceptionTranslationFilter``\ ,
| returns error code 403.
| By setting the error page at the time of access denial in http element, it is possible to transit to the specified error page.

* spring-security.xml

  .. code-block:: xml
  
    <sec:http auto-config="true" use-expressions="true">
        <!-- omitted -->
        <sec:access-denied-handler error-page="/accessDeneidPage" />  <!-- (1) -->
    </sec:http>
    
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - | Sr. No.
       - | Description
     * - | (1)
       - | Specify destination path in \ ``error-page``\  attribute of \ ``<sec:access-denied-handler>``\  element.


Access authorization (JSP)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Use custom JSP tag \ ``<sec:authorize>``\  provided by Spring Security, to control screen display fields.
| Usage declaration settings of tag library``<%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags" %>``
| is the precondition.

* Attributes list of \ ``<sec:authorize>``\  tag

  .. tabularcolumns:: |p{0.15\linewidth}|p{0.85\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 15 85
  
     * - | Attribute name
       - | Description
     * - | \ ``access``\ 
       - | Describes the access control expression. If true, tag contents are evaluated.
     * - | \ ``url``\ 
       - | Tag contents are evaluated when authority is granted to the set URL. It is used to control link display etc.
     * - | \ ``method``\ 
       - | Specifies the HTTP method (GET, POST etc.).It is used by combining with url attribute and only the specified method
         | is matched with the specified URL pattern. When not specified, GET is applied.
     * - | \ ``ifAllGranted``\ 
       - | Tag contents are evaluated when all the set roles are granted. Role hierarchy functionality is not effective.
     * - | \ ``ifAnyGranted``\ 
       - | Tag contents are evaluated when any one of the set roles is granted. Role hierarchy functionality is not effective.
     * - | \ ``ifNotGranted``\ 
       - | Tag contents are evaluated when the set role is not granted. Role hierarchy functionality is not effective.
     * - | \ ``var``\ 
       - | Declares the variables of page scope that stores tag evaluation result. It is used when same authorization check is performed in a page.

| Example showing use of \ ``<sec:authorize>``\  tag is as follows:

* spring-security.xml

  .. code-block:: jsp
  
    <div>
      <sec:authorize access="hasRole('ROLE_USER')">  <!-- (1) -->
          <p>This screen is for ROLE_USER</p>
      </sec:authorize>
      <sec:authorize url="/admin/menu">  <!-- (2) -->
          <p>
            <a href="/admin/menu">Go to admin screen</a>
          </p>
      </sec:authorize>
    </div>
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - | Sr. No.
       - | Description
     * - | (1)
       - | Tag contents are displayed only when it contains  "ROLE_USER".
     * - | (2)
       - | Tag contents are displayed when access is authorized for "/admin/menu".

  .. warning::

     The authorization process using \ ``<sec:authorize>``\  tag \ **can only be controlled by screen display**\ . As a result, even if the link is not displayed by specific authority, the URL of the link can be directly accessed by guessing the URL.
     Therefore, make sure to use it together with the "Access authorization (request URL)" described earlier or the "Access authorization (Method)" that will be described later and perform the essential access control.


Access authorization (Method)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Authorization control can be performed for method.
| The Bean stored in Spring's DI container is the target of authorization.

| The two authorization methods described earlier were authorization controls in application layer.
| However, method level authorization is controlled with respect to domain layer (Service class).
| It is advisable to set \ ``org.springframework.security.access.prepost.PreAuthorize``\  annotation for the method to be controlled.

* spring-security.xml

  .. code-block:: xml
  
    <sec:global-method-security pre-post-annotations="enabled"/>  <!-- (1) -->
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - | Sr. No.
       - | Description
     * - | (1)
       - | Specify \ ``pre-post-annotations``\  attribute of \ ``<sec:global-method-security>``\  element in \ ``enabled``\ .
         | It is \ ``disabled``\  by default.

* Java code

  .. code-block:: java

    @Service
    @Transactional
    public class UserServiceImpl implements UserService {
        // omitted

        @PreAuthorize("hasRole('ROLE_ADMIN')") // (1)
        @Override
        public User create(User user) {
           // omitted
        }


        @PreAuthorize("isAuthenticated()")
        @Override
        public User update(User user) {
           // omitted
        }
    }

  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - | Sr. No.
       - | Description
     * - | (1)
       - | State the access control expression. The expression is evaluated before executing the method and if it is true, the method is executed.
         | If the expression is false, \ ``org.springframework.security.access.AccessDeniedException``\  is thrown.
         | Expression stated in \ :ref:`authorization-intercept-url`\  and
         | the expression stated in \ `Spring Expression Language (SpEL) <http://docs.spring.io/spring/docs/4.1.9.RELEASE/spring-framework-reference/html/expressions.html>`_\  can be set as the value.

  .. tip::
  
    Following annotations can also be used in the above setting, other than \ ``org.springframework.security.access.prepost.PreAuthorize``\ .
  
    * \ ``org.springframework.security.access.prepost.PostAuthorize``\ 
    * \ ``org.springframework.security.access.prepost.PreFilter``\ 
    * \ ``org.springframework.security.access.prepost.PostFilter``\ 
  
    For their details, refer \ `Spring Security manual <http://docs.spring.io/spring-security/site/docs/3.2.9.RELEASE/reference/htmlsingle/#el-pre-post-annotations>`_\ .

  .. note::

    In Spring Security, authorization can also be controlled by using \ ``javax.annotation.security.RolesAllowed``\  annotation of JSR-250 which is a standard Java specification.
    However, SpEL expressions cannot be stated in \ ``@RolesAllowed``\ . By \ ``@PreAuthorize``\  annotation, using SpEL, authorization control can be performed in the same notations as spring-security.xml settings.


  .. note::
  
    It is recommended to perform authorization control for request path by carrying out settings in spring-security.xml, instead of assigning annotation to Controller method.
    
    If Service is executed only via Web and if authorization control is performed for all the request path patterns, authorization control for Service need not be performed.
    It is advisable to use annotation when it is not known from where Service will be executed and thus authorization control is necessary.

How to extend
--------------------------------------------------------------------------------

Role hierarchy functionality
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Hierarchical relation can be set in roles.
| Roles that are set at higher level can have all the accesses authorized by the lower roles.
| Consider using hierarchy functionality if the role relation is complex.

| It is explained by setting a hierarchical relation wherein ROLE_ADMIN is the higher role and ROLE_USER is set as the lower role.

.. figure:: ./images/Authorization_RoleHierarchy.png
   :alt: RoleHierarchy
   :width: 30%
   :align: center

   **Picture - RoleHierarchy**

| Here, if access authorization is set as follows,
| the user with "ROLE_ADMIN" role can also access "/user/\*" URL.

**Spring Security configuration file**

.. code-block:: xml

  <sec:http auto-config="true" use-expressions="true">
      <sec:intercept-url pattern="/user/*" access="hasAnyRole('ROLE_USER')" />
      <!-- omitted -->
  </sec:http>

| The setting method differs for access authorization (request URL), access authorization (JSP) and access authorization (Method).
| How to use these methods is explained below.


Common settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Required common settings are described here.
| Perform Bean definition of \ ``org.springframework.security.access.hierarchicalroles.RoleHierarchy`` class that stores the hierarchical relation.

* spring-security.xml

  .. code-block:: xml
  
    <bean id="roleHierarchy"
        class="org.springframework.security.access.hierarchicalroles.RoleHierarchyImpl"> <!-- (1) -->
        <property name="hierarchy">
            <value> <!-- (2) -->
                ROLE_ADMIN > ROLE_STAFF
                ROLE_STAFF > ROLE_USER
            </value>
        </property>
    </bean>
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - | Sr. No.
       - | Description
     * - | (1)
       - | Specify the default class of \ ``RoleHierarchy``\  namely, ``org.springframework.security.access.hierarchicalroles.RoleHierarchyImpl``.
     * - | (2)
       - | Define the hierarchical relation in \ ``hierarchy``\  property.
         | Format:
         | [Higher role] > [Lower role]
         | In the example, STAFF can access all the USER authorized resources.
         | ADMIN can access all the resources authorized by USER and STAFF.


How to use it in access authorization (request URL) and access authorization (JSP)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| How to set role hierarchy for request URL and JSP is explained here.

* spring-security.xml

  .. code-block:: xml
  
    <bean id="webExpressionHandler"
        class="org.springframework.security.web.access.expression.DefaultWebSecurityExpressionHandler">  <!-- (1) -->
        <property name="roleHierarchy" ref="roleHierarchy"/>  <!-- (2) -->
    </bean>
  
    <sec:http auto-config="true" use-expression="true">
        <!-- omitted -->
        <sec:expression-handler ref="webExpressionHandler" />  <!-- (3) -->
    </sec:http>
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - | Sr. No.
       - | Explanation
     * - | (1)
       - | Specify \ ``org.springframework.security.web.access.expression.DefaultWebSecurityExpressionHandler``\  in class.
     * - | (2)
       - | Set the Bean ID of \ ``RoleHierarchy``\  in \ ``roleHierarchy``\  property.
     * - | (3)
       - | Specify the Bean ID of handler class that implemented \ ``org.springframework.security.access.expression.SecurityExpressionHandler``\  in \ ``expression-handler``\  element.


How to use it in access authorization (Method)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Role hierarchy settings when authorization control is performed by assigning annotation to Service method, are explained here.


* spring-security.xml

  .. code-block:: xml
  
    <bean id="methodExpressionHandler"
        class="org.springframework.security.access.expression.method.DefaultMethodSecurityExpressionHandler"> <!-- (1) -->
        <property name="roleHierarchy" ref="roleHierarchy"/> <!-- (2) -->
    </bean>
  
    <sec:global-method-security pre-post-annotations="enabled">
        <sec:expression-handler ref="methodExpressionHandler" /> <!-- (3) -->
    </sec:global-method-security>
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - | Sr. No.
       - | Description
     * - | (1)
       - | Specify \ ``org.springframework.security.access.expression.method.DefaultMethodSecurityExpressionHandler``\  in class.
     * - | (2)
       - | Set the Bean ID of \ ``RoleHierarchy``\  in \ ``roleHierarchy``\  property.
     * - | (3)
       - | Specify the Bean ID of handler class that implements \ ``org.springframework.security.access.expression.SecurityExpressionHandler``\  in \ ``expression-handler``\  element.

.. raw:: latex

   \newpage

