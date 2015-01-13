Authentication
================================================================================

.. only:: html

 .. contents:: Table of contents
    :local:
    
Overview
--------------------------------------------------------------------------------
| This chapter explains about Authentication functionality provided by Spring Security.

In Spring Security, user authentication can be implemented by just performing configuration file settings.
DB authentication, LDAP authentication, CAS authentication, JAAS authentication, X509 authentication and Basic authentication are the authentication methods provided by Spring Security. However, only DB authentication is described in this guideline.

.. tip::
    For details of authentication types other than DB authentication, refer to the official document of each authentication method.

  * \ `LDAP Authentication <http://docs.spring.io/spring-security/site/docs/3.2.5.RELEASE/reference/htmlsingle/#ldap>`_\
  * \ `CAS Authentication <http://docs.spring.io/spring-security/site/docs/3.2.5.RELEASE/reference/htmlsingle/#cas>`_\
  * \ `Java Authentication and Authorization Service (JAAS) Provider <http://docs.spring.io/spring-security/site/docs/3.2.5.RELEASE/reference/htmlsingle/#jaas>`_\
  * \ `X.509 Authentication <http://docs.spring.io/spring-security/site/docs/3.2.5.RELEASE/reference/htmlsingle/#x509>`_\
  * \ `Basic and Digest Authentication <http://docs.spring.io/spring-security/site/docs/3.2.5.RELEASE/reference/htmlsingle/#basic>`_\

Login
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Flow of Spring Security login process is as follows:

.. figure:: ./images/Authentication_Login_overview.png
   :alt: Authentication(Login)
   :width: 80%
   :align: center
 
#. Authentication filter is activated on receiving the request specifying authentication.
#. Authentication filter extracts username and password from the request and generates authentication information.
   The  generated authentication information is used as a parameter to execute authentication process of authentication manager.
#. Authentication manager executes authentication process of the specified authentication provider.
   Authentication provider acquires user information from datasource (DB or LDAP) and performs user authentication such as password verification.
   When authentication is successful, authentication information that holds the authenticated information is created and returned to the authentication manager.
   When authentication fails, the authentication manager raises authentication failure exception.
#. Authentication manager returns the received authentication information to authentication filter.
#. Authentication filter stores the received authentication information (authenticated) in session.
#. When authentication is successful, it initializes the session information before authentication and creates new session information.
#. It redirects to the specified path of successful/failed  authentication and returns session ID to client.

Logout
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Flow of Spring Security logout process is as follows:

.. figure:: ./images/Authentication_Logout_overview.png 
   :alt: Authentication(Logout)
   :width: 80%
   :align: center
 

#. Logout filter gets activated on receiving request for logout.
#. It discards the session information.
   It sets a response such that client cookie (Cookie in figure) is discarded.
#. It redirects to the specified logout path.

\
 .. note::
  The session information that remains after logout can be used by a third party. Hence, in order to prevent such spoofing,
  session information is discarded at the time of logout using \ ``org.springframework.security.web.session.ConcurrentSessionFilter``\ .

|

How to use
--------------------------------------------------------------------------------
| Settings to be performed in Spring Security configuration file so as to use authentication functionality, are as follows:
| For basic settings, refer to \ :doc:`SpringSecurity`\ .

Setting \ ``<sec:http>``\  element
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| As shown in the following example, basic settings for authentication functionality of
| Spring Security can be omitted by setting the \ ``auto-config``\  attribute of \ ``<http>``\  element in spring-security.xml to \ ``true``\ .

.. code-block:: xml

  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:sec="http://www.springframework.org/schema/security"
      xmlns:context="http://www.springframework.org/schema/context"
      xsi:schemaLocation="http://www.springframework.org/schema/security
          http://www.springframework.org/schema/security/spring-security.xsd
          http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context
          http://www.springframework.org/schema/context/spring-context.xsd">
      <sec:http auto-config="true" use-expressions="true">  <!-- (1) -->
        <!-- omitted -->
      </sec:http>
  </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | By setting \ ``auto-config="true"``\ ,
       | it is enabled even if \ ``<form-login>``\ , \ ``<http-basic>``\  and \ ``<logout>``\  elements are not set.

.. note::

  \ ``<form-login>``\ , \ ``<http-basic>``\  and \ ``<logout>``\  elements are explained here.

    .. tabularcolumns:: |p{0.15\linewidth}|p{0.85\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 15 85

       * - Element name
         - Description
       * - | \ ``<form-login>``\ 
         - | \ ``org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter``\  is enabled.
           | UsernamePasswordAuthenticationFilter is the Filter that performs authentication by extracting username and password from the request at the time of POST method.
           | For details, refer \ :ref:`form-login`\ .
       * - | \ ``<http-basic>``\ 
         - | \ ``org.springframework.security.web.authentication.www.BasicAuthenticationFilter``\  is enabled.
           | BasicAuthenticationFilter is the filter that executes Basic authentication process according to RFC1945.
           | For details, refer \ `BasicAuthenticationFilter JavaDoc <http://docs.spring.io/spring-security/site/docs/3.2.5.RELEASE/apidocs/org/springframework/security/web/authentication/www/BasicAuthenticationFilter.html>`_\ .
       * - | \ ``<logout>``\ 
         - | \ ``org.springframework.security.web.authentication.logout.LogoutFilter``\ ,
           | \ ``org.springframework.security.web.authentication.logout.SecurityContextLogoutHandler``\  is enabled.
           | LogoutFilter is the Filter called at the time of Logout.
           | \ ``org.springframework.security.web.authentication.rememberme.TokenBasedRememberMeServices``\  (deleting Cookie) and,
           | SecurityContextLogoutHandler(disabling  session) is called.
           | For details, refer \ :ref:`form-logout`\ .

.. _form-login:

Setting \ ``<sec:form-login>``\  element
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| How to set \ ``<sec:form-login>``\  element is described in this section.
| 
| Attributes of form-login element are as shown below.

spring-security.xml

.. code-block:: xml

  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:sec="http://www.springframework.org/schema/security"
      xmlns:context="http://www.springframework.org/schema/context"
      xsi:schemaLocation="http://www.springframework.org/schema/security
          http://www.springframework.org/schema/security/spring-security.xsd
          http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context
          http://www.springframework.org/schema/context/spring-context.xsd">
    <sec:http auto-config="true" use-expressions="true">
      <sec:form-login login-page="/login"
          default-target-url="/"
          login-processing-url="/authentication"
          always-use-default-target="false"
          authentication-failure-url="/login?error=true"
          authentication-failure-handler-ref="authenticationFailureHandler"
          authentication-success-handler-ref="authenticationSuccessHandler" /> <!-- Perform steps (1) - (7) in the specified attribute order-->
    </sec:http>
  </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify the login form screen path in \ ``login-page``\  attribute.
       | An "unauthenticated user" is forcefully redirected to this path, when he tries to access the page that can be accessed only by
       | an "authenticated user".
   * - | (2)
     - | Specify the destination path in \ ``default-target-url``\  attribute when authentication is successful. When it is not specified, "/" will be the default path.
   * - | (3)
     - | Specify the path that performs authentication process in \ ``login-processing-url``\  attribute. When not specified,"j_spring_security_check" will be the default path.
       | **This guideline recommends changing to a system specific value rather than using the default value  "j_spring_security_check", mentioned above.**\ In this example, "/authentication" is specified.
   * - | (4)
     - | After a successful login, specify whether it should always transit to the path specified in \ ``default-target-url``\ , in the \ ``always-use-default-target``\  attribute.
       | By default it is set as \ ``false``\ . When set as \ ``false``\  and when the base handler class for successful authentication namely, \ ``org.springframework.security.web.authentication.AbstractAuthenticationTargetUrlRequestHandler``\  is specified as
       | redirect destination, it transits to the specified destination. If the redirect destination is not specified, it transits to the path specified in \ ``default-target-url``\ .
   * - | (5)
     - | In case of failed authentication, specify the destination in \ ``authentication-failure-url``\ .
       | When \ ``authentication-failure-handler-ref``\  attribute is not specified, irrespective of the type of authentication error, it uniformly transits to the destination specified for this setting.
   * - | (6)
     - | Specify the handler class to be called in case of a failed authentication, in \ ``default-target-url``\  attribute.
       | For details, refer \ :ref:`authentication-failure-handler-ref`\ .
   * - | (7)
     - | Specify the handler class to be called in case of a successful authentication, in \ ``default-target-url``\  attribute.

For attributes other than those mentioned above, refer to \ `Spring Security manual <http://docs.spring.io/spring-security/site/docs/3.2.5.RELEASE/reference/htmlsingle/#nsa-form-login>`_\ .

.. warning:: **Why it is not recommended to use Spring Security default value, "j_spring_security_check".**

  If default value is used, the fact that the application is using Spring Security is revealed.
  As a result, if any Spring Security related vulnerability is detected, there is a higher risk of receiving an attack due to the vulnerability.
  In order to prevent these risks, it is recommended to avoid using default value.

.. _form-login-JSP:

Creating login form
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Create the login form to be used at the time of authentication in JSP.

* src/main/webapp/WEB-INF/views/login.jsp

  .. code-block:: jsp

      <form:form action="${pageContext.request.contextPath}/authentication" method="post"><!-- (1) -->
          <!-- omitted -->
          <input type="text" id="username" name="j_username"><!-- (2) -->
          <input type="password" id="password" name="j_password"><!-- (5) -->
          <input type="submit" value="Login">
      </form:form>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | Specify the destination for performing authentication process, in the action attribute of form.
         | /authentication specified in the login-processing-url, should be specified as the destination path.
         | Authentication process is executed by accessing ${pageContext.request.contextPath}/authentication.
         | "POST" should be specified as the HTTP method.
     * - | (4)
       - | Element handled as "User ID" in authentication process.
         | Spring Security default value namely, "j_username", should be specified in the name attribute.
     * - | (5)
       - | Element handled as "Password" in authentication process.
         | Spring Security default value namely, "j_password", should be specified in the name attribute.

  Following code is added to display the authentication error message.

  .. code-block:: jsp

      <c:if test="${param.error}"><!-- (1) -->
          <t:messagesPanel
              messagesAttributeName="SPRING_SECURITY_LAST_EXCEPTION"/><!-- (2) -->
      </c:if>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | Determine the error message set in request parameter.
         | Please note that the determination process needs to be changed according to 
         | the value set in the authentication-failure-url attribute of form-login element or the value set in "defaultFailureUrl" of authentication error handler.
         | This example is shown with the setting as authentication-failure-url="/login?error=true".
     * - | (2)
       - | Output the exception message that is to be output at the time of authentication error.
         | It is recommended to output this message by specifying \ ``org.terasoluna.gfw.web.message.MessagesPanelTag``\  provided by common library.
         | For the details of "\ ``<t:messagesPanel>``\" tag, refer \ :doc:`../ArchitectureInDetail/MessageManagement`\ .


* spring-mvc.xml

  Define the Controller that displays login form.

  .. code-block:: xml

    <mvc:view-controller path="/login" view-name="login" /><!-- (1) -->
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | If "/login" is accessed, define the controller that returns only "login" as the view name. src/main/webapp/WEB-INF/views/login.jsp is output by \ ``InternalResourceViewResolver``\ .
         | This simple controller need not be implemented in Java.
         
   
  .. note::
   
      Above settings are identical with next controller.
      
        .. code-block:: java
        
          @Controller
          @RequestMapping("/login")
          public class LoginController {
          
              @RequestMapping
              public String index() {
                  return "login";
              }
          }

      If the Controller with a single method that returns only the view name is necessary, \ ``<mvc:view-controller>``\  may be used.
      
      Advantage of displaying login form through a Controller is that, the CSRF token gets automatically embedded by this. When it is not displayed via a Controller,
      the CSRF token needs to be embedded directly in jsp, as executed in \ :doc:`Tutorial`\ .
      
      The details of creating a Controller are omitted in the Tutorial. As a result, login form is not displayed through a Controller. For details on CSRF measures, refer \ :doc:`CSRF`\ .


Changing attribute name of login form
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

"j_username" and "j_password" are the Spring Security default values. They can be changed to any value by using \ ``<form-login>``\  element settings.

* spring-security.xml


  Attributes of \ ``username``\  and \ ``password``\ 

  .. code-block:: xml

    <sec:http auto-config="true" use-expressions="true">
      <sec:form-login
          username-parameter="username"
          password-parameter="password" /> <!-- Perform steps (1) and (2) in the specified attribute order -->
      <!-- omitted -->
    </sec:http>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | In \ ``username-parameter``\  attribute, \ ``name``\  attribute of input field \ ``username``\  is changed to "username".
     * - | (2)
       - | In \ ``password-parameter``\  attribute, \ ``name``\  attribute of input field  \ ``password``\  is changed to "password".

Setting authentication process
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
In order to set the authentication process in Spring Security, define \ ``AuthenticationProvider``\  and \ ``UserDetailsService``\ .

\ ``AuthenticationProvider``\  plays the following roles.

* Returning authenticated user information in case of successful authentication
* Throwing an exception in case of authentication failure.

\ ``UserDetailsService``\  fetches authenticated user information from the persistence layer.

These classes may each be used as default or may be used by extending individually.
They may also be combined.


\ ``AuthenticationProvider``\  class settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| As \ ``AuthenticationProvider``\  implementation, how to use the provider, \ ``org.springframework.security.authentication.dao.DaoAuthenticationProvider``\  that performs DB authentication, is explained here.

* spring-security.xml

  .. code-block:: xml

      <sec:authentication-manager><!-- (1) -->
          <sec:authentication-provider user-service-ref="userDetailsService"><!-- (2) -->
              <sec:password-encoder ref="passwordEncoder" /><!-- (3) -->
          </sec:authentication-provider>
      </sec:authentication-manager>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | Define \ ``<sec:authentication-provider>``\  element in \ ``<sec:authentication-manager>``\  element. Authentication methods can be combined by specifying multiple elements. However it is not explained here.
     * - | (2)
       - | Define \ ``AuthenticationProvider``\  in \ ``<sec:authentication-provider>``\  element. \ ``DaoAuthenticationProvider``\  is enabled by default. To specify an \ ``AuthenticationProvider``\  other than this, \ specify the Bean ID of target AuthenticationProvider, in `ref attribute <http://docs.spring.io/spring-security/site/docs/3.2.5.RELEASE/reference/htmlsingle/#nsa-authentication-provider>`_\ .
         |
         | Specify the Bean Id of \ ``UserDetailsService``\  that fetches authenticated user information, in \ ``user-service-ref``\  attribute. This setting is mandatory when using \ ``DaoAuthenticationProvider``\ .
         | For details, refer \ :ref:`userDetailsService`\ .
     * - | (3)
       - | Specify the Bean ID of the class that encodes the password entered from Form, at the time of password verification.
         | When it is not specified, password is handled in "Plain Text". For details, refer \ :doc:`PasswordHashing`\ .


| If the requirement is to perform authentication by fetching data from persistence layer using only the "User ID" and "Password", this \ ``DaoAuthenticationProvider``\  may be used.
| Which method is to be used to fetch data from persistence layer, is determined using the \ ``UserDetailsService``\  explained below.

.. _userDetailsService:

\ ``UserDetailsService``\  class settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Set the Bean specified in \ ``userDetailsService``\  property of \ ``AuthenticationProvider``\ .

\ ``UserDetailsService``\  is the interface that includes the next method.

.. code-block:: java

  UserDetails loadUserByUsername(String username) throws UsernameNotFoundException

By executing this interface, authenticated user information can be fetched from any storage location.

Here, \ ``org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl``\  that fetches user information from DB using JDBC, is explained.

In order to use \ ``JdbcDaoImpl``\ , it is advisable to perform following Bean definitions in spring-security.xml.

.. code-block:: xml

  <!-- omitted -->

  <bean id="userDetailsService"
    class="org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl">
    <property name="dataSource" ref="dataSource"/>
  </bean>

| It is assumed that \ ``JdbcDaoImpl``\  defines the default SQL for fetching authenticated user information and authorization information and provides tables corresponding to these. For definitions of assumed tables, refer \ `Spring Security manual <http://docs.spring.io/spring-security/site/docs/3.2.5.RELEASE/reference/htmlsingle/#appendix-schema>`_\ .
| To fetch user information and authorization information from existing tables, the SQL to be executed should be modified according to existing tables.
| Following 3 SQLs are used.

*  \ `User information acquisition query <http://docs.spring.io/spring-security/site/docs/3.2.5.RELEASE/apidocs/constant-values.html#org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl.DEF_USERS_BY_USERNAME_QUERY>`_\

  | By creating a table matching with the query that fetches user information, need of specifying the query to configuration file described later, is eliminated.
  | Fields namely, "username", "password" and "enabled" are mandatory
  | Also, by specifying the query to the configuration file described later and by assigning an alias to the query, there is no issue even if table name and column name do not match.
  | For example, while setting the following SQL, "email" column can be used as "username" wherein, "enabled" field is always \ ``true``\ .

  .. code-block:: sql

    SELECT email AS username, pwd AS password, true AS enabled FROM customer WHERE email = ?

  | "User ID" described earlier in \ :ref:`form-login-JSP`\ , is specified in query parameter.

* \ `User authorities acquisition query <http://docs.spring.io/spring-security/site/docs/3.2.5.RELEASE/apidocs/constant-values.html#org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl.DEF_AUTHORITIES_BY_USERNAME_QUERY>`_\

  | This query fetches authorization information for a user.

* \ `Group authorities acquisition query <http://docs.spring.io/spring-security/site/docs/3.2.5.RELEASE/apidocs/constant-values.html#org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl.DEF_GROUP_AUTHORITIES_BY_USERNAME_QUERY>`_\

  | This query fetches authorization information of the group, to which the user belongs. 'Group authorities' is disabled by default and is also out of this guideline's scope.

| DB definition example and example of Spring Security configuration file are shown below.

| Table definitions
| Define the required table when implementing DB authentication process.
| This table matches with the default query that fetches user information, mentioned earlier
| Therefore, following are the definitions of the minimum necessary tables. (with tentative physical name)

Table name: account

.. tabularcolumns:: |p{0.15\linewidth}|p{0.15\linewidth}|p{0.10\linewidth}|p{0.60\linewidth}|
.. list-table:: 
   :header-rows: 1
   :widths: 15 15 10 60

   * - Logical name
     - Physical name
     - Type
     - Description
   * - User ID
     - username
     - String
     - User ID for uniquely identifying the user.
   * - Password
     - password
     - String
     - User password. Stored in hashed status.
   * - Enabled flag
     - enabled
     - Boolean value
     - Flag indicating invalid user and valid user. The user set to "false" is an invalid user, thus results in throwing an authentication error.
   * - Authority name
     - authority
     - String
     - Not required when authorization functionality is unnecessary.

Following is the example wherein, \ ``JdbcDaoImpl``\  is set through customization.

.. code-block:: xml

  <!-- omitted -->

  <bean id="userDetailsService"
    class="org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl">
    <property name="rolePrefix" value="ROLE_" /><!-- (1) -->
    <property name="dataSource" ref="dataSource" />
    <property name="enableGroups" value="false" /><!-- (2) -->
    <property name="usersByUsernameQuery"
      value="SELECT username, password, enabled FROM account WHERE username = ?" /><!-- (3) -->
    <property name="authoritiesByUsernameQuery"
      value="SELECT username, authority FROM account WHERE username = ?" /><!-- (4) -->
  </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify the prefix of authorization name. When the authorization name stored on DB is "USER", this authenticated user object has the authorization name as "ROLE_USER".
       | It is necessary to set the naming conventions and authorization functionality by combining them. For details of authorization functionality, refer \ :doc:`Authorization`\ .
   * - | (2)     
     - | Specify this when the concept of "Group authorities" is to be used in authorization functionality.
       | Not handled in this guideline.
   * - | (3)
     - | Set the query for fetching user information. Data should be fetched in the order, "User ID", "Password" and "Enabled flag".
       | When authentication is not decided by "Enabled flag", SELECT result of "Enabled flag" is fixed to "true".
       | The query that can uniquely acquire a user, should be described. When multiple records are fetched, the first record is used as user.
   * - | (4)
     - | Set the query that fetches user authority. Data should be acquired in the order, "User ID" and "Authority ID".
       | When authorization functionality is not used, "Authority ID" can be any fixed value.

.. note::
    Authentication that cannot be implemented just by changing query, is necessary to be implemented by extending \ ``UserDetailsService``\ .
    For extension methods, refer \ :ref:`extendsuserdetailsservice`\ .

How to use \ ``UserDetails``\  class
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


| How to use \ ``UserDetails``\  created by \ ``UserDetailsService``\  after successful authentication is explained.


Using \ ``UserDetails``\  object in Java class
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| After successful authentication, \ ``UserDetails``\  class
| is stored in \ ``org.springframework.security.core.context.SecurityContextHolder``\ .

Example of fetching \ ``UserDetails``\  from \ ``SecurityContextHolder``\  is shown below.

.. code-block:: java

  public static String getUsername() {
      Authentication authentication = SecurityContextHolder.getContext()
              .getAuthentication(); // (1)
      if (authentication != null) {
          Object principal = authentication.getPrincipal(); // (2)
          if (principal instanceof UserDetails) {
              return ((UserDetails) principal).getUsername(); // (3)
          }
          return (String) principal.toString();
      }
      return null;
  }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Fetch \ ``org.springframework.security.core.Authentication``\  object from \ ``SecurityContextHolder``\ .
   * - | (2)
     - | Fetch \ ``UserDetails``\  object from \ ``Authentication``\  object.
   * - | (3)
     - | Fetch user name from \ ``UserDetails``\  object.


While the method for fetching \ ``UserDetails``\  object from \ ``SecurityContextHolder``\  is convenient as it can be used from anywhere by static method,
it ends up increasing module coupling. Testing is also difficult to execute.

| \ ``UserDetails``\  object can be fetched by using \ ``@AuthenticationPrincipal``\ .
| To use \ ``@AuthenticationPrincipal``\ , it is necessary to set \ ``org.springframework.security.web.bind.support.AuthenticationPrincipalArgumentResolver``\  in \ ``<mvc:argument-resolvers>``\ .

- :file:`spring-mvc.xml`

.. code-block:: xml
   :emphasize-lines: 5-6

    <mvc:annotation-driven>
        <mvc:argument-resolvers>
            <bean
                class="org.springframework.data.web.PageableHandlerMethodArgumentResolver" />
            <bean
                class="org.springframework.security.web.bind.support.AuthenticationPrincipalArgumentResolver" />
        </mvc:argument-resolvers>
    </mvc:annotation-driven>


As shown below, \ ``UserDetails``\  object can be fetched in Spring MVC Controller, without using \ ``SecurityContextHolder``\ .

.. code-block:: java

    @RequestMapping(method = RequestMethod.GET)
    public String view(@AuthenticationPrincipal SampleUserDetails userDetails, // (1)
            Model model) {
        // get account object
        Account account = userDetails.getAccount(); // (2)
        model.addAttribute(account);
        return "account/view";
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Fetch the information of logged-in user using \ ``@AuthenticationPrincipal``\ .
   * - | (2)
     - | Fetch account information from \ ``SampleUserDetails``\ .

.. note::

    The type of argument attached with \ ``@AuthenticationPrincipal``\  annotation, needs to be the class that inherits \ ``UserDetails``\  type.
    Usually, it is better to use the \ ``UserDetails``\  inheritance class that is created by using \ :ref:`extendsuserdetailsservice`\ .

    \ ``SampleUserDetails``\  class is the class created by using \ :doc:`Tutorial`\ . For details, refer \ :ref:`Tutorial_CreateAuthService`\ .

\ **This method is recommended when accessing UserDetails object in Controller**\ .

.. note::

  It is recommended to use the \ ``UserDetails``\  object information fetched from Controller in Service class rather than using \ ``SecurityContextHolder``\ .

  \ ``SecurityContextHolder``\  should be used only in those methods where \ ``UserDetails``\  object is not passed as argument.

Accessing \ ``UserDetails``\  in JSP
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Spring Security provides JSP taglib as a feature that enables using authentication information in JSP. Following declaration is necessary in order to use taglib.

.. code-block:: jsp

  <%@ taglib uri="http://www.springframework.org/security/tags" prefix="sec"%>

.. note::

  \ `It is already set in WEB-INF/views/common/include.jsp when using TERASOLUNA Global Framework sample <https://github.com/terasolunaorg/terasoluna-gfw-web-blank>`_\ .

| An example of using JSP for displaying authentication is as follows:

.. code-block:: jsp

  <sec:authentication property="principal.username" /><!-- (1) -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | \ ``Authentication``\  object can be accessed using \ ``<sec:authentication>`` \  tag and the property specified in \ ``property``\  attribute can be accessed. In this example, result of \ ``getPrincipal().getUsername()``\  is output.



.. code-block:: jsp

  <sec:authentication property="principal" var="userDetails" /> <!-- (1) -->

  ${f:h(userDetails.username)} <!-- (2) -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Property specified in \ ``property``\  attribute can be stored in variable, by changing it to a  \ ``var``\  attribute name.
   * - | (2)
     - | \ ``UserDetails``\  can be accessed in JSP once it is stored in variable by step (1).

.. note::

  \ ``UserDetails``\  can also be fetched in Controller and added to \ ``Model``\ . However, it is advisable to use JSP tag when displaying it in JSP.


.. note::
  
  \ ``UserDetails``\  created by \ ``JdbcDaoImpl``\  which is explained in :ref:`userDetailsService`\ , stores only the minimum required information such as "User ID" and "Authority".

  When other information related to the user such as "User name" etc. is required to be displayed as screen fields, it is necessary to extend \ ``UserDetails``\  and  \ ``UserDetailsService``\ .
  For extension methods, please refer, \ :ref:`extendsuserdetailsservice`\ .


.. _authentication(spring_security)_how_to_use_sessionmanagement:

Session management in Spring Security
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| How to create session information at login and how to perform the settings when an exception occurs, is explained here.
| By specifying \ ``<session-management>``\  tag, \ ``org.springframework.security.web.session.SessionManagementFilter``\  is enabled.
| Following is the configuration example of spring-security.xml


.. code-block:: xml

  <sec:http auto-config="true" create-session="ifRequired" ><!-- (1) -->
    <!-- omitted -->
    <sec:session-management
      invalid-session-url="/"
      session-authentication-error-url="/"
      session-fixation-protection="migrateSession"
      session-authentication-strategy-ref="sessionStrategy" /><!-- Perform steps (2) to (5) in the specified order of attribute -->
    <!-- omitted -->
  </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify the policy of creating session in \ ``create-session``\  attribute.
       | Following values can be specified.
       | \ ``always``\ : Spring Security creates a new session in case there is no existing session and it reuses a session that it already existing.
       | \ ``ifRequired``\ : Spring Security creates a session if required. It is a default setting. If a session already exists, it reuses this session without creating a new one.
       | \ ``never``\ : Spring Security does not create a session but reuses an existing session.
       | \ ``stateless``\ : Spring Security neither creates a session nor uses an existing one. As a result, authentication is required each time.
   * - | (2)
     - | Specify the transition path when an invalid session ID is requested in \ ``invalid-session-url``\  attribute.
       | When the path is not set, transit to a path which is dependent on \ ``org.springframework.security.web.session.SimpleRedirectInvalidSessionStrategy``\
       | setting.
   * - | (3)
     - | When an exception occurs in \ ``org.springframework.security.web.authentication.session.SessionAuthenticationStrategy``\ 
       | specify the transition path in \ ``session-authentication-error-url``\  attribute.
       | When not specified, it is dependent on \ ``org.springframework.security.web.authentication.SimpleUrlAuthenticationFailureHandler``\
       | setting. 
   * - | (4)
     - | Specify the session management system in \ ``session-fixation-protection``\  attribute.
       | Following values can be specified.
       | \ ``migrateSession``\: It inherits the session information before login (Copy) and only creates a new ID. This is a default setting.
       | \ ``newSession``\: It creates new session details and new ID without inheriting the session information before login.
       |
       | The aim of this functionality is to prevent \ `Session fixation attack <http://docs.spring.io/spring-security/site/docs/3.2.5.RELEASE/reference/htmlsingle/#ns-session-fixation>`_\  by allocating new session ID for each login. Therefore, it is recommended to use this default setting, unless there are other clear reasons for not using it.
   * - | (5)
     - | Specify the Bean ID of \ ``org.springframework.security.core.Authentication.SessionAuthenticationStrategy``\  class that decides the session check behavior in \ ``session-authentication-strategy-ref``\  attribute.


Setting \ ``SessionAuthenticationStrategy``\ 
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Configuration example of \ ``SessionAuthenticationStrategy``\  is shown below.

.. code-block:: xml

    <bean id="sessionStrategy"
        class="org.springframework.security.web.authentication.session.CompositeSessionAuthenticationStrategy">  <!-- (1) -->
        <constructor-arg index="0">
            <list>
                <bean
                    class="org.springframework.security.web.authentication.session.SessionFixationProtectionStrategy" />
                <bean class="org.springframework.security.web.csrf.CsrfAuthenticationStrategy">  <!-- (2) -->
                    <constructor-arg index="0" ref="csrfTokenRepository" />
                </bean>
            </list>
        </constructor-arg>
    </bean>
    <!-- omitted -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify by listing in the constructor arguments of
       | \ ``org.springframework.security.web.authentication.session.CompositeSessionAuthenticationStrategy``\ .
   * - | (2)
     - | In this example, \ ``org.springframework.security.web.csrf.CsrfAuthenticationStrategy``\
       | that perform CSRF check is specified. Foe CSRF measures, refer \ :doc:`CSRF`\ .

Controlling the number of concurrent sessions.
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Spring Security provides (\ `Concurrent Session Control <http://docs.spring.io/spring-security/site/docs/3.2.5.RELEASE/reference/htmlsingle/#concurrent-sessions>`_\ ) functionality that enables to arbitrarily change the number of maximum sessions that one user can store.
| The user mentioned here is the authentication user object fetched by \ ``Authentication.getPrincipal()``\ .

|  For the control method, following patterns are available when it exceeds the maximum number of sessions. They should be suitably used as per business requirements.

#. When a user exceeds the number of maximum sessions, the user having least usage is disabled. (after win)
#. When a user exceeds the number of maximum sessions, new login request is not accepted.(first win)

In both cases, following settings need to be added to web.xml, to enable this functionality.

.. _HttpSessionEventPublisher-ref:

.. code-block:: xml

    <listener>
      <listener-class>org.springframework.security.web.session.HttpSessionEventPublisher</listener-class><!-- (1) -->
    </listener>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | When using Concurrent Session Control, it is necessary to define \ ``org.springframework.security.web.session.HttpSessionEventPublisher``\  in listener.

1. When disabling the user with least usage

  Add following settings to spring-security.xml.

  .. code-block:: xml

      <sec:http auto-config="true" >
        <!-- omitted -->
        <sec:custom-filter position="CONCURRENT_SESSION_FILTER" ref="concurrencyFilter" />  <!-- (1) -->

        <sec:session-management
          session-authentication-strategy-ref="sessionStrategy" />
        <!-- omitted -->
      </sec:http>

      <bean id="concurrencyFilter"
         class="org.springframework.security.web.session.ConcurrentSessionFilter">  <!-- (2) -->
         <constructor-arg index="0" ref="sessionRegistry" />  <!-- (3) -->
         <constructor-arg index="1" value="/" />  <!-- (4) -->
      </bean>

      <bean id="sessionAuthenticationStrategy"
          class="org.springframework.security.web.authentication.session.CompositeSessionAuthenticationStrategy">
          <constructor-arg index="0">
              <list>
                  <!-- omitted -->
                  <bean class=
                      "org.springframework.security.web.authentication.session.ConcurrentSessionControlStrategy">  <!-- (5) -->
                      <constructor-arg index="0" ref="sessionRegistry" />  <!-- (6) -->
                      <property name="maximumSessions" value="1" />  <!-- (7) -->
                  </bean>
              </list>
          </constructor-arg>
      </bean>

      <bean id="sessionRegistry" class="org.springframework.security.core.session.SessionRegistryImpl" />  <!-- (8) -->
      <!-- omitted -->

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | Specify \ ``CONCURRENT_SESSION_FILTER``\  in \ ``position``\  attribute of \ ``<custom-filter>``\  element.
     * - | (2)
       - | Perform Bean definition for \ ``org.springframework.security.web.session.ConcurrentSessionFilter``\  class.
     * - | (3)
       - | Reference specify \ ``org.springframework.security.core.session.SessionRegistryImpl``\  in the first argument of constructor.
     * - | (4)
       - | Specify the path to which the expired session transits, in the second argument of constructor.
     * - | (5)
       - | Specify \ ``org.springframework.security.web.authentication.session.ConcurrentSessionControlStrategy``\  in
         | the first argument of CompositeSessionAuthenticationStrategy.
     * - | (6)
       - | Reference specify \ ``org.springframework.security.core.session.SessionRegistryImpl``\  in the first argument of constructor.
     * - | (7)
       - | Maximum number of sessions allowed for one user can be defined in  \ ``maximumSessions``\  attribute.
         | In the above example, since 1 is specified, the number of allowed sessions for one user is 1.
         | When the user logs in with multiple browsers, the oldest used session is set as 'expired'.
         | When not specified, 1 is set.
     * - | (8)
       - | Specify \ ``org.springframework.security.core.session.SessionRegistryImpl``\  class that implements
         | \ ``org.springframework.security.core.session.SessionRegistry``\  interface.

2. When not accepting new logins

  Following settings are performed in spring-security.xml.

  .. code-block:: xml
      :emphasize-lines: 16

      <bean id="concurrencyFilter"
         class="org.springframework.security.web.session.ConcurrentSessionFilter">
         <constructor-arg index="0" ref="sessionRegistry" />
         <constructor-arg index="1" value="/" />
      </bean>

      <bean id="sessionAuthenticationStrategy"
          class="org.springframework.security.web.authentication.session.CompositeSessionAuthenticationStrategy">
          <constructor-arg index="0">
              <list>
                  <!-- omitted -->
                  <bean class=
                      "org.springframework.security.web.authentication.session.ConcurrentSessionControlStrategy">
                      <constructor-arg index="0" ref="sessionRegistry" />
                      <property name="maximumSessions" value="1" />
                      <property name="exceptionIfMaximumExceeded" value="true"/> <!-- (1) -->
                  </bean>
              </list>
          </constructor-arg>
      </bean>
      <!-- omitted -->

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | When it exceeds maximum number of sessions by setting the \ ``exceptionIfMaximumExceeded``\  attribute to \ ``true``\ ,
         | \ ``org.springframework.security.web.authentication.session.SessionAuthenticationException``\  exception is thrown.
         | As a result, it does not transit to the path defined in the second argument of \ ``ConcurrentSessionFilter``\ . Please take note of same.
         | When  the setting for \ ``exceptionIfMaximumExceeded``\  attribute is omitted, \ ``false``\  is set.

  .. tip::

    \ `<sec:concurrency-control> <http://docs.spring.io/spring-security/site/docs/3.2.5.RELEASE/reference/htmlsingle/#nsa-concurrency-control>`_\  can be used as a child element of \ ``<sec:session-management>``\  element, without specifying the \ ``session-authentication-strategy-ref``\  attribute of ``<sec:session-management>``\  element.

.. _authentication-failure-handler-ref:

Setting the handler class in case of authentication error
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| By performing the settings for \ ``org.springframework.security.web.authentication.ExceptionMappingAuthenticationFailureHandler``\  class in
| \ ``authentication-failure-handler-ref``\  attribute of \ ``<sec:form-login>``\  element,
| exception thrown at the time of authentication error and its corresponding destination, can be specified.
| The specified destination is accessible to unauthenticated user.

spring-security.xml

.. code-block:: xml

    <sec:http auto-config="true" use-expressions="true">
      <sec:form-login login-page="/login"
          authentication-failure-handler-ref="authenticationFailureHandler"
          authentication-success-handler-ref="authenticationSuccessHandler" />
    </sec:http>

    <bean id="authenticationFailureHandler"
    class="org.springframework.security.web.authentication.ExceptionMappingAuthenticationFailureHandler">
    <property name="defaultFailureUrl" value="/login/defaultError" /><!-- (1) -->
      <property name="exceptionMappings"><!-- (2) -->
        <props>
          <prop key=
            "org.springframework.security.authentication.BadCredentialsException"><!-- (3) -->
              /login/badCredentials
          </prop>
          <prop key=
            "org.springframework.security.core.userdetails.UsernameNotFoundException"><!-- (4) -->
              /login/usernameNotFound
          </prop>
          <prop key=
            "org.springframework.security.authentication.DisabledException"><!-- (5) -->
              /login/disabled
          </prop>
          <prop key=
            "org.springframework.security.authentication.ProviderNotFoundException"><!-- (6) -->
              /login/providerNotFound
          </prop>
          <prop key=
            "org.springframework.security.authentication.AuthenticationServiceException"><!-- (7) -->
              /login/authenticationService
          </prop>
          <!-- omitted -->
        </props>
      </property>
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify the default destination path in case of an error.
       | If an exception not defined in the \ ``exceptionMappings``\  property that will be described later occurs, it transits to the destination specified by this property.
   * - | (2)
     - | Specify the exceptions to be caught and their corresponding destinations in a list format.
       | Set the exception class in key and destination in value.


.. _SpringSecurity-Exception:

A typical exception thrown by Spring Security is shown below.

.. tabularcolumns:: |p{0.10\linewidth}|p{0.25\linewidth}|p{0.25\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 25 65

   * - Sr. No.
     - Error type
     - Description
   * - | (3)
     - \ ``BadCredentialsException``\ 
     - It is thrown when authentication error occurs due to failure in password verification.
   * - | (4)
     - \ ``UsernameNotFoundException``\ 
     - | It is thrown when authentication error occurs due to an invalid user ID (non-existent user ID).
       | When specifying the class that inherits \ ``org.springframework.security.authentication.dao.AbstractUserDetailsAuthenticationProvider``\  in authentication provider,
       | if \ ``hideUserNotFoundExceptions``\  is not changed to \ ``false``\ , the above exception is changed to \ ``BadCredentialsException``\ .
   * - | (5)
     - \ ``DisabledException``\ 
     - It is thrown when an authentication error occurs due to invalid user ID.
   * - | (6)
     - \ ``ProviderNotFoundException``\ 
     - | It is thrown at the time of undetected error of authentication provider class.
       | Exception occurs when authentication provider class is invalid due to reasons such as a setting error etc.
   * - | (7)
     - \ ``AuthenticationServiceException``\
     - | It is thrown at the time of authentication service error.
       | It is thrown when certain errors such as DB connection error etc. occur in authentication service.

.. warning::

  In this example, transition is made by handling \ ``UsernameNotFoundException``\ .
  However, if the user is informed that user ID does not exist, presence and absence of specific IDs may be revealed, which is not desirable from the security perspective.
  Therefore, it is recommended that the screen transition and notification message to the user is set such that it does not reveal type of exception.

.. _form-logout:

Setting \ ``<sec:logout>``\  element
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| How to set the \ ``<sec:logout>``\  element, is explained in this section.

spring-security.xml

.. code-block:: xml

  <sec:http auto-config="true" use-expressions="true">
    <!-- omitted -->
    <sec:logout
        logout-url="/logout"
        logout-success-url="/"
        invalidate-session="true"
        delete-cookies="JSESSIONID"
        success-handler-ref="logoutSuccessHandler"
      /> <!-- Perform steps (1) to (5) in the specified attribute order -->
    <!-- omitted -->
  </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify the path for executing logout process in \ ``logout-url``\  attribute.
   * - | (2)
     - | Specify the destination path after logout in \ ``logout-success-url``\  attribute.
   * - | (3)
     - | In \ ``invalidate-session``\  attribute, set whether to discard the session when logging out. By default, it is set to \ ``true``\ . When \ ``true``\ , the session gets cleared at the time of logout.
   * - | (4)
     - | Delete the cookie specified in \ ``delete-cookies``\  attribute. When multiple cookies are to be deleted, use "," to separate them.
   * - | (5)
     - | Specify the handler class that is called after a successful logout, in \ ``success-handler-ref``\  attribute.

.. note::

    CSRF token check is performed when  \ ``<sec:csrf>``\  explained in \ :doc:`./CSRF`\  is used. Therefore, \ **it is necessary to send logout request using POST as well as send the CSRF token.**\ 
    How to embed CSRF token is explained below.

    * \ :ref:`csrf_formformtag-use`\

        .. code-block:: jsp
           :emphasize-lines: 1,4

            <form:form method="POST"
              action="${pageContext.request.contextPath}/logout">
              <input type="submit" value="Logout" />
            </form:form>

        Following HTML is output in such cases. CSRF token is set as hidden.

        .. code-block:: html

            <form id="command" action="/your-context-path/logout" method="POST">
              <input type="submit" value="Logout" />
              <input type="hidden" name="_csrf" value="5826038f-0a84-495b-a851-c363e501b73b" />
            </form>

    * \ :ref:`csrf_formtag-use`\

        .. code-block:: jsp
           :emphasize-lines: 3

            <form  method="POST"
              action="${pageContext.request.contextPath}/logout">
              <sec:csrfInput/>
              <input type="submit" value="Logout" />
            </form>

        In this case as well, following HTML is output as before. CSRF token is set as hidden.

        .. code-block:: html

            <form  method="POST"
              action="/your-context-path/logout">
              <input type="hidden" name="_csrf" value="5826038f-0a84-495b-a851-c363e501b73b" />
              <input type="submit" value="Logout" />
            </form>



Setting \ ``<sec:remember-me>``\  element
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| "\ `Remember Me <http://docs.spring.io/spring-security/site/docs/3.2.5.RELEASE/reference/htmlsingle/#remember-me>`_\ " functionality enhances convenience of the frequent users of the website and,
| is the functionality that stores the login status.
| This functionality stores login information in a cookie, if user has given permission, even after the browser is closed
| and enables the user to login without re-entering the user name and password.

| The attributes of \ ``<sec:remember-me>``\  element are shown below.

spring-security.xml

.. code-block:: xml

  <sec:http auto-config="true" use-expressions="true">
    <!-- omitted -->
    <sec:remember-me key="terasoluna-tourreservation-km/ylnHv"
            token-validity-seconds="#{30 * 24 * 60 * 60}" />  <!-- Perform steps (1) to (2) in the specified attribute order-->
    <!-- omitted -->
  </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify the unique key that stores the cookie for Remember Me functionality in \ ``key``\  attribute.
       | When not specified, it is recommended to specify it in cases where improvement in start-up time has been considered as the unique key is generated at the time of start-up.
   * - | (2)
     - | Specify the validity of the cookie for Remember Me functionality in seconds, in \ ``token-validity-seconds``\  attribute. In this example it is set as 30 days.
       | When not specified, it is valid for 14 days by default.

For attributes other than the above, refer \ `Spring Security manual <http://docs.spring.io/spring-security/site/docs/3.2.5.RELEASE/reference/htmlsingle/#nsa-remember-me>`_\ .

Following flag that enables "Remember Me" functionality needs to be provided in login form.

.. code-block:: jsp
  :emphasize-lines: 7-9

  <form method="post"
    action="${pageContext.request.contextPath}/authentication">
      <!-- omitted -->
      <label for="_spring_security_remember_me">Remember Me : </label>
      <input name="_spring_security_remember_me"
        id="_spring_security_remember_me" type="checkbox"
        checked="checked"> <!-- (1) -->
      <input type="submit" value="LOGIN">
      <!-- omitted -->
  </form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | When requested as \ ``true``\, next authentication can be avoided
       | by setting \ ``_spring_security_remember_me``\  in HTTP parameter.

How to extend
--------------------------------------------------------------------------------

.. _extendsuserdetailsservice:

Extending \ ``UserDetailsService``\ 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| When information other than user ID and password needs to be fetched at the time of authentication,

* \ ``org.springframework.security.core.userdetails.UserDetails``\ 
* \ ``org.springframework.security.core.userdetails.userDetailsService``\ 

need to be implemented.

When attached information such as login user's name and division etc. need to be displayed at all times on screen header, fetching it for each request from DB hampers the efficiency.
This extension is necessary to enable its access from \ ``SecurityContext``\  or \ ``<sec:authentication>``\  tags, by storing it in \ ``UserDetails``\  object.

Extending \ ``UserDetails``\
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Creating \ ``ReservationUserDetails``\  class that also stores customer information other than authentication information.

.. code-block:: java

  public class ReservationUserDetails extends User { // (1)
      // omitted

      private final Customer customer; // (2)

      private static final List<? extends GrantedAuthority> DEFAULT_AUTHORITIES = Collections
              .singletonList(new SimpleGrantedAuthority("ROLE_USER"));         // (3)

      public ReservationUserDetails(Customer customer) {
          super(customer.getCustomerCode(),
                  customer.getCustomerPassword(), true, true, true, true, DEFAULT_AUTHORITIES); // (4)
          this.customer = customer;
      }

      public Customer getCustomer() { // (5)
          return customer;
      }
  }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - |  Inherit \ ``org.springframework.security.core.userdetails.User``\  class which is the default class of \ ``UserDetails``\ .
   * - | (2)
     - | Store the DomainObject class containing authentication information and customer information.
   * - | (3)
     - | Create authorization information using constructor of \ ``org.springframework.security.core.authority.SimpleGrantedAuthority``\ . Here the authority named "ROLE_USER" is provided.
       |
       | This is a simple implementation wherein the authorization information should essentially be fetched from another table in the DB.
   * - | (4)
     - | Set the user ID and password contained in DomainObject, in the constructor of super class.
   * - | (5)
     - | Method to access customer information via \ ``UserDetails``\ .

.. note::

  When the business requirement cannot be implemented just by inheriting \ ``User``\  class, \ ``UserDetails``\  interface may be implemented.

Implementing an independent \ ``UserDetailsService``\
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Create ReservationUserDetailsService class that implements \ ``UserDetailsService``\ .
| In this example, customer information is fetched from DB by injecting  \ ``CustomerSharedService``\  class that implements the process to fetch \ ``Customer``\  object.

.. code-block:: java

  public class ReservationUserDetailsService implements UserDetailsService {
      @Inject
      CustomerSharedService customerSharedService;

      @Override
      public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
          Customer customer = customerSharedService.findOne(username);
          // omitted
          return new ReservationUserDetails(customer);
      }

  }

How to use
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
How to use the created \ ``ReservationUserDetailsService``\  and \ ``ReservationUserDetails``\ , is explained here.

* spring-security.xml

  .. code-block:: xml

    <sec:authentication-manager>
        <sec:authentication-provider user-service-ref="userDetailsService"><!-- (1) -->
            <sec:password-encoder ref="passwordEncoder" />
        </sec:authentication-provider>
    </sec:authentication-manager>

    <bean id="userDetailsService"
        class="com.example.domain.service.userdetails.ReservationUserDetailsService"><!-- (2) -->
    </bean>
    <!-- omitted -->

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | Define the Bean ID of \ ``ReservationUserDetailsService``\  in ref attribute.
     * - | (2)
       - | Perform Bean definition for \ ``ReservationUserDetailsService``\ .

* JSP

   Access \ ``Customer``\  object by using \ ``<sec:authentication>``\  tag.

  .. code-block:: jsp

     <sec:authentication property="principal.customer" var="customer"/><!-- (1) -->
     ${f:h(customer.customerName)}<!-- (1) -->

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - |  \ ``Customer``\  object contained in \ ``ReservationUserDetails``\  is stored in variable.
    * - | (2)
      - | Display the optional property of \ ``Customer``\  object stored in variable.
        | For \ ``f:h()``\ , refer \ :doc:`XSS`\ .

* Controller

  .. code-block:: java

    @RequestMapping(method = RequestMethod.GET)
    public String view(Principal principal, Model model) {
        // get Authentication 
        Authentication authentication = (Authentication) principal;
        // get UserDetails
        ReservationUserDetails userDetails = (ReservationUserDetails) authentication.getPrincipal();
        // get Customer
        Customer customer = userDetails.getCustomer(); // (1)
        // omitted ...
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Fetch the logged-in \ ``Customer``\  object from \ ``ReservationUserDetails``\ .
        | Perform business process by passing this object to Service class.

.. note::

  When customer information is changed, \ ``Customer``\  object contained in \ ``ReservationUserDetails``\  is not changed unless logout is carried out once.
  
  It is recommended not to store the information that may undergo frequent changes or the information which is changed by a user other than the login user (administrator etc.).


Extending \ ``AuthenticationProvider``\ 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. todo::

  Review the details.

| In case of business requirements that cannot be supported by Spring Security's \ `default authentication provider <http://docs.spring.io/spring-security/site/docs/3.2.5.RELEASE/apidocs/org/springframework/security/authentication/AuthenticationProvider.html>`_\ ,
| it is necessary to create the class that implements \ ``org.springframework.security.authentication.AuthenticationProvider``\ .

| It is necessary to return the class that implements \ ``org.springframework.security.core.Authentication``\  as return value, in \ ``AuthenticationProvider``\ .
| It is necessary to set \ ``getPrincipal``\  method (to fetch authentication information)
| and \ ``getCredentials``\  method (to fetch information that guarantees validity of principal) in the class that implements  \ ``Authentication``\ .

| Here, the case that requires authentication information other than user name and password for authentication, is considered. To access these values in \ ``AuthenticationProvider``\ ,
 it is necessary to create a class that inherits.

* \ ``org.springframework.security.authentication.UsernamePasswordAuthenticationToken``\ 
* \ ``org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter``\ 


| An example with user name, \ **Company identifier**\  and password as the authentication information is shown below.

.. figure:: ./images/Authentication_HowToExtends_LoginForm.png
   :alt: Authentication_HowToExtends_LoginForm
   :width: 40%

Extending \ ``UsernamePasswordAuthenticationToken``\ 
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Create an independent \ ``AuthenticationToken``\  that inherits \ ``UsernamePasswordAuthenticationToken``\ .
| By extending \ ``UsernamePasswordAuthenticationToken``\ , company identifier can be included in authentication information.

.. code-block:: java

    // import omitted
    public class CompanyIdUsernamePasswordAuthenticationToken extends
                                                                     UsernamePasswordAuthenticationToken {

        private static final long serialVersionUID = SpringSecurityCoreVersion.SERIAL_VERSION_UID;

        private final String companyId;  // (1)

        public CompanyIdUsernamePasswordAuthenticationToken(
                Object principal, Object credentials, String companyId) {  // (2)
            super(principal, credentials);

            this.companyId = companyId;
        }

        public CompanyIdUsernamePasswordAuthenticationToken(
                Object principal, Object credentials, String companyId,
                Collection<? extends GrantedAuthority> authorities) {  // (3)
            super(principal, credentials, authorities);
            this.companyId = companyId;
        }

        public String getCompanyId() {
            return companyId;
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Create a field for company identifier.
   * - | (2)
     - | Constructor used while creating instance of \ ``CompanyIdUsernamePasswordAuthenticationToken``\ , before authentication.
   * - | (3)
     - | Constructor used while creating instance of \ ``CompanyIdUsernamePasswordAuthenticationToken``\ , after successful authentication.
       | Authentication completion status is reached by passing authorization information collectively, to the constructor argument of parent class.

Implementing an independent \ ``AuthenticationProvider``\ 
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Using \ ``CompanyIdUsernamePasswordAuthenticationToken``\  in an independent \ ``AuthenticationProvider``\ .

.. code-block:: java

    // import omitted
    public class CompanyIdUsernamePasswordAuthenticationProvider implements
                                                                        AuthenticationProvider {
        // omitted

        @Override
        public Authentication authenticate(Authentication authentication) throws AuthenticationException {

            CompanyIdUsernamePasswordAuthenticationToken authenticationToken =
                (CompanyIdUsernamePasswordAuthenticationToken) authentication; // (1)

            // Obtain userName, password, companyId
            String username = authenticationToken.getName();
            String password = authenticationToken.getCredentials().toString();
            String companyId = authenticationToken.getCompanyId();
            
            // Obtain user (and grants if needed) from database or other system
            // UserDetails userDetails = ...
            
            // Business logic
            // ...
            
            return new UsernamePasswordAuthenticationToken(userDetails, authentication.getCredentials(),
                    Collections.singletonList(new SimpleGrantedAuthority("ROLE_USER"))); // (2)
        }

        @Override
        public boolean supports(Class<?> authentication) {
            return CompanyIdUsernamePasswordAuthenticationToken.class
                    .equals(authentication); // (3)
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
     - | Cast the \ ``UsernamePasswordAuthenticationFilter``\  in the independent \ ``AuthenticationToken``\  which is set in extended class.
       | For details, refer \ extension <authentication_custom_usernamepasswordauthenticationfilter> of :ref:`UsernamePasswordAuthenticationFilter`\  which will be described later.
   * - | (2)
     - | After executing the authentication process (fetching user information, password check, company identifier check etc.),
       | create and return \ ``UsernamePasswordAuthenticationToken``\ . By setting authorization information collectively in parameter,
       | authenticated instance namely, \ ``AuthenticationToken``\  is created. \ ``UserDetails``\  is passed to constructor parameter as well.
       |
       | In this example, only a single role is used as it is a simple implementation.
       | For details on authority, refer \ :ref:`User information management class settings <userDetailsService>`\ .
   * - | (3)
     - | \ ``CompanyIdUsernamePasswordAuthenticationToken``\  class, which is an independent \ ``AuthenticationToken``\ 
       | should be supported.

.. _authentication_custom_usernamepasswordauthenticationfilter:

Extending \ ``UsernamePasswordAuthenticationFilter``\ 
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Create an independent \ ``AuthenticationFilter``\  that inherits \ ``UsernamePasswordAuthenticationFilter``\ .
| By extending \ ``UsernamePasswordAuthenticationFilter``\ , \ ``CompanyIdUsernamePasswordAuthenticationToken``\  can be passed to
| \ ``CompanyIdUsernamePasswordAuthenticationProvider``\ .

.. code-block:: java

    // import omitted
    public class CompanyIdUsernamePasswordAuthenticationFilter extends
                                                                      UsernamePasswordAuthenticationFilter {

        @Override
        public Authentication attemptAuthentication(HttpServletRequest request,
                HttpServletResponse response) throws AuthenticationException {

            if (!request.getMethod().equals("POST")) {
                throw new AuthenticationServiceException("Authentication method not supported: "
                        + request.getMethod());
            }

            // Obtain UserName, Password, CompanyId
            String username = super.obtainUsername(request);
            String password = super.obtainPassword(request);
            String companyId = obtainCompanyId(request);

            // username required
            if (!StringUtils.hasText(username)) {
                throw new AuthenticationServiceException("UserName is required");
            }

            // validate password, companyId
            
            // omitted other process

            CompanyIdUsernamePasswordAuthenticationToken authRequest =
                    new CompanyIdUsernamePasswordAuthenticationToken(username, password, companyId);  // (1)

            // Allow subclasses to set the "details" property
            setDetails(request, authRequest);

            return this.getAuthenticationManager().authenticate(authRequest);  // (2)
        }

        protected String obtainCompanyId(HttpServletRequest request) {
            return request.getParameter("j_companyid");  // (3)
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Create an instance of \ ``CompanyIdUsernamePasswordAuthenticationToken``\ 
       | in which the user name, password and company identifier fetched from \ ``HttpServletRequest``\ , have been set.
   * - | (2)
     - | Set the unauthenticated  instance of \ ``CompanyIdUsernamePasswordAuthenticationToken``\
       | as the parameter of \ ``org.springframework.security.authentication.AuthenticationManager.authenticate``\ .
   * - | (3)
     - | Company identifier is fetched from the request parameter.

.. note::

  **Login information input check**

  Sometimes, a check needs to be performed in advance for the obvious input errors that occur when load reduction to DB server etc.is carried out.
  In such cases, the input check process can be performed by extending \ ``UsernamePasswordAuthenticationFilter``\ , similar to
  \ :ref:`authentication_custom_usernamepasswordauthenticationfilter`\ .


How to use
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* Login form page (JSP)

  Adding company identifier to \ :ref:`form-login-JSP`\  example.

  .. code-block:: jsp
    :emphasize-lines: 7-8

    <form:form action="${pageContext.request.contextPath}/authentication" method="post">
        <!-- omitted -->
        <span>User Id</span><br>
        <input type="text"
            id="username" name="j_username"><br>
        <span>Company Id</span><br>
        <input type="text"
            id="companyid" name="j_companyid"><br>  <!-- (1) -->
        <span>Password</span><br>
        <input type="password"
            id="password" name="j_password"><br>
        <!-- omitted -->
    </form:form>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | Specify j_companyid for the input field 'name' of company identifier.

* spring-security.xml

  | Set the independent \ ``AuthenticationProvider``\  and \ ``AuthenticationFilter``\ .

  .. code-block:: xml

    <sec:http auto-config="false" use-expressions="true" entry-point-ref="loginUrlAuthenticationEntryPoint">  <!-- (1) -->
        <!-- omitted -->
        <sec:custom-filter position="FORM_LOGIN_FILTER" ref="companyIdUsernamePasswordAuthenticationFilter" />  <!-- (2) -->

        <!-- omitted -->

        <sec:logout logout-url="/logout" logout-success-url="/"
            delete-cookies="JSESSIONID" invalidate-session="true" />
    </sec:http>

    <bean id="companyIdUsernamePasswordAuthenticationFilter"
        class="com.example.app.common.security.CompanyIdUsernamePasswordAuthenticationFilter">  <!-- (3) -->
        <property name="authenticationManager" ref="authenticationManager" />  <!-- (4) -->
        <property name="authenticationFailureHandler" ref="authenticationFailureHandler" />  <!-- (5) -->
        <property name="authenticationSuccessHandler" ref="authenticationSuccessHandler" />  <!-- (6) -->
        <property name="filterProcessesUrl" value="/authentication" />  <!-- (7) -->
    </bean>

    <bean id="loginUrlAuthenticationEntryPoint" class="org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint">  <!-- (8) -->
        <constructor-arg value="/login" />  <!-- (9) -->
    </bean>

    <sec:authentication-manager alias="authenticationManager">
        <sec:authentication-provider ref="companyIdUsernamePasswordAuthenticationProvider" />  <!-- (10) -->
    </sec:authentication-manager>

    <bean id="companyIdUsernamePasswordAuthenticationProvider"
        class="com.example.app.common.security.CompanyIdUsernamePasswordAuthenticationProvider" />  <!-- (11) -->

    <!-- omitted -->

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | It is not possible to specify auto-config="true" when "FORM_LOGIN_FILTER" is replaced in custom-filter element.
         | Therefore, it is necessary either to delete the specified auto-config or specify auto-config="false".
         | Further, entry-point-ref attribute needs to be specified clearly as the form-login element also cannot be specified.
     * - | (2)
       - | By specifying "FORM_LOGIN_FILTER" in the position attribute of custom-filter element,
         | it can be replaced from UsernamePasswordAuthenticationFilter to CompanyIdUsernamePasswordAuthenticationFilter.
     * - | (3)
       - | Perform bean definition for CompanyIdUsernamePasswordAuthenticationFilter.
     * - | (4)
       - | Specify the value for alias attribute of authentication-manager, in authenticationManager property.
     * - | (5)
       - | Specify the handler class to be called in case of authentication failure, in authenticationFailureHandler property.
     * - | (6)
       - | Specify the handler class to be called in case of successful authentication, in authenticationSuccessHandler property.
     * - | (7)
       - | Specify the path of authentication process in filterProcessesUrl property.
     * - | (8)
       - | Define AuthenticationEntryPoint to be specified in the entry-point-ref attribute of http element.
         | It is the default AuthenticationEntryPoint when form-login element was specified.
         | Perform bean definition of \ ``org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint``\ .
     * - | (9)
       - | Specify the login screen path in constructor argument.
     * - | (10)
       - | Perform reference configuration CompanyIdUsernamePasswordAuthenticationProvider in authentication-provider element.
     * - | (11)
       - | Perform bean definition of CompanyIdUsernamePasswordAuthenticationProvider.

  .. warning::

       To use \ ``<sec:http-basic>``\  and \ ``<sec:logout>``\  elements wherein auto-config="false" has been specified, they need to be clearly defined.

Appendix
--------------------------------------------------------------------------------
| When performing authentication using Spring Security, transit to the path described in configuration file when authentication is successful.
| When there is a business requirement such as "login to read more",
| sometimes the transition destination after login needs to be changed dynamically.

.. figure:: ./images/Authentication_Appendix_ScreenFlow.png
   :alt: Authentication_Appendix_Screen_Flow
   :width: 60%
   :align: center

   **Picture - Screen_Flow**

| In such cases, use \ ``org.terasoluna.gfw.web.security.RedirectAuthenticationHandler``\
| provided by common library.

| Setting example using RedirectAuthenticationHandler is shown below.

**Description example of source screen JSP**

.. code-block:: jsp

  <form:form action="${pageContext.request.contextPath}/login" method="get">
      <!-- omitted -->
    <input type="hidden" name="redirectTo"
      value="${pageContext.request.contextPath}/reservetour/read?
      ${f:query(reserveTourForm)}&page.page=${f:h(param['page.page'])}
      &page.size=${f:h(param['page.size'])}" />  <!-- (1) -->
  </form:form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Setting redirect URL
       | Set the transition destination URL in the hidden "redirectTo" field.
       | The value specified in name should be matched with targetUrlParameter
       | described in configuration file explained later.

**Description example of login screen JSP**

.. code-block:: jsp

  <form:form action="${pageContext.request.contextPath}/authentication" method="post">
       <!-- omitted -->
       <input type="submit"
         value="Login">
       <input type="hidden" name="redirectTo" value="${f:h(param.redirectTo)}" />  <!-- (1) -->
       <!-- omitted -->
  </form:form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Set redirect URL
       | Set the redirect URL passed from source screen using request parameter,
       | to hidden field.

**Spring Security configuration file**

.. code-block:: xml

  <sec:http auto-config="true">
                  <!-- omitted -->
      <sec:form-login login-page="/login" default-target-url="/"
          always-use-default-target="false"
          authentication-failure-handler-ref="authenticationFailureHandler"
          authentication-success-handler-ref="authenticationSuccessHandler" />
                                                                      <!-- (1) -->
                  <!-- omitted -->
  </sec:http>
                  
   <bean id="authenticationSuccessHandler"
      class="org.terasoluna.gfw.web.security.RedirectAuthenticationHandler">
                                                                      <!-- (2) -->
      <property name="targetUrlParameter" value="redirectTo"/>        <!-- (3) -->
  </bean>
                  
  <bean id="authenticationFailureHandler"
      class="org.springframework.security.web.authentication.ExceptionMappingAuthenticationFailureHandler">
                                                                      <!-- (4) -->
      <property name="defaultFailureUrl" value="/login?error=true"/> <!-- (5) -->
      <property name="useForward" value="true"/>                     <!-- (6) -->
  </bean>
                  <!-- omitted -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify BeanId of authentication-failure-handler-ref (handler setting at the time of authentication error) and
       | authentication-success-handler-ref (handler setting in case of successful authentication).
   * - | (2)
     - | Set \ ``org.terasoluna.gfw.web.security.RedirectAuthenticationHandler``\ 
       | as the reference class of authentication-success-handler-ref.
   * - | (3)
     - | It should be matched with the value described in hidden field of JSP.
       | In this example, it is set as "redirectTo". When omitted, "redirectTo" is set.
   * - | (4)
     - | Set \ ``org.springframework.security.web.authentication.ExceptionMappingAuthenticationFailureHandler``\
       | as reference class of authentication-failure-handler-ref.
   * - | (5)
     - | Set the query indicating error and path of login screen, in the transition destination path of failed authentication.
   * - | (6)
     - | When using this function, true should be specified in useForward.
       | By specifying true, it becomes a forward transition from a redirect transition.
       | Since URL of redirect destination is stored in request parameter,
       | it is necessary to ensure that request parameter is stored as it is by setting it to "forward".

.. tip::
  Measures against Open Redirector vulnerability are provided in the RedirectAuthenticationHandler.
  Therefore, it cannot transit to external sites such as "http://google.com", without extending. 
  In order to move to other domain, it is necessary to create the class implementing \ ``org.springframework.security.web.RedirectStrategy``\ .
  The class implementing RedirectStrategy in targetUrlParameterRedirectStrategy of RedirectAuthenticationHandler, is set using setter injection.
  As a precaution while extending, it is necessary to have a structure that does not pose any problems even if the redirectTo value is tampered with.
  For example, any one of the measures such as checking redirect destination domain or specifying redirect destination URL by number (specifying the page number)
  instead of specifying it directly, need to be taken.

.. raw:: latex

   \newpage

