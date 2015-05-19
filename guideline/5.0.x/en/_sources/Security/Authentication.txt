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
       | When not specified, "/spring_security_login" will be default path and Login screen provided by Spring Security is used.
       | When an "Unauthenticated user" accesses a page that can only be accessed by an "Authenticated user", the unauthenticated user is redirected to this path.

       | **This guideline recommends changing to a system specific value rather than using the default value "/spring_security_login", mentioned above.**\ In this example, "/login" is specified.
   * - | (2)
     - | In \ ``default-target-url``\  attribute, specify the destination path when authentication is successful. 
       | When not specified, "/" will be the default path.

       | When \ ``authentication-success-handler-ref``\ attribute is specified, this setting is not used.
   * - | (3)
     - | Specify the path for performing authentication process in \ ``login-processing-url``\  attribute. 
       | When not specified,"j_spring_security_check" will be the default path.

       | **This guideline recommends changing to a system specific value rather than using the default value  "j_spring_security_check", mentioned above.**\ In this example, "/authentication" is specified.
   * - | (4)
     - | In \ ``always-use-default-target``\  attribute, specify whether it should always transit to the path specified in \ ``default-target-url``\  after a successful login.
       | When it is set as \ ``true``\ , it always transits to the path specified in \ ``default-target-url``\ .
       | When it is set as \ ``false``\  (default), it transits either to the "path for displaying secure page which somebody has tried accessing before the login" or "path specified in \ ``default-target-url``\ ".

       | When \ ``authentication-success-handler-ref``\  attribute is specified, this setting is not used. 
   * - | (5)
     - | In \ ``authentication-failure-url``\ , specify the destination when authentication fails.
       | When not specified, path specified in  \ ``login-page``\  attribute is applicable.

       | When \ ``authentication-failure-handler-ref``\  attribute is specified, this setting is not used.
   * - | (6)
     - | Specify the handler class to be called in case of a failed authentication, in \ ``authentication-failure-handler-ref``\  attribute.
       | For details, refer \ :ref:`authentication-failure-handler-ref`\ .
   * - | (7)
     - | Specify the handler class to be called in case of a successful authentication, in \ ``authentication-success-handler-ref``\  attribute.

For attributes other than those mentioned above, refer to \ `Spring Security manual <http://docs.spring.io/spring-security/site/docs/3.2.5.RELEASE/reference/htmlsingle/#nsa-form-login>`_\ .

.. warning:: **Why it is not recommended to use Spring Security default values, "/spring_security_login, /j_spring_security_check".**

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


 .. note:: **Regarding settings required for accessing exception object of authentication error from JSP**

    Exception object of authentication error is stored in session scope with the attribute name \ ``"SPRING_SECURITY_LAST_EXCEPTION"``\ .
    \ ``session``\ attribute of \ ``page``\ directive of JSP should be set to \ ``true``\  for accessing the object stored in session scope from JSP.

    * ``src/main/webapp/WEB-INF/views/common/include.jsp``

     .. code-block:: jsp

        <%@ page session="true"%>

    Default settings of blank project are such that session scope cannot be accessed from JSP.
    This is to ensure that the session is not used easily.


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
         
   
  .. tip::
   
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

  \ `It is already set in WEB-INF/views/common/include.jsp when using TERASOLUNA Server Framework for Java (5.x) sample <https://github.com/terasolunaorg/terasoluna-gfw-web-blank>`_\ .

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
       | When not specified, "/j_spring_security_logout" will be default path.

       | **In this guideline, it is recommended that you change it to unique system value instead of using the above default value "/j_spring_security_logout".** In this example, "/logout" is specified.
   * - | (2)
     - | Specify the destination path after logout in \ ``logout-success-url``\  attribute.
       | When not specified, "/" will be default path.

       | If \ ``success-handler-ref``\  attribute is specified while this attribute is specified, error will occur during startup process.
   * - | (3)
     - | In \ ``invalidate-session``\  attribute, set whether to discard the session when logging out.
       | By default, it is set to \ ``true``\ .
       | In case of \ ``true``\ , the session is discarded at the time of logout.
   * - | (4)
     - | In \ ``delete-cookies``\  attribute, mention the cookie names to be deleted at the time of logout.
       | When multiple cookies are mentioned, use "," to separate them.
   * - | (5)
     - | Specify the handler class to be called after a successful logout, in \ ``success-handler-ref``\  attribute.

       | If \ ``logout-success-url``\  attribute is specified while this attribute is specified, error will occur during startup process.

.. warning:: **Why it is not recommended to use Spring Security default value, "/j_spring_security_logout"**

    If default value is used, the fact that the application is using Spring Security is revealed.
    As a result, if any Spring Security related vulnerability is detected, there is a higher risk of receiving an attack due to the vulnerability.
    In order to prevent these risks, it is recommended to avoid using default value.

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
    public String view(@AuthenticationPrincipal ReservationUserDetails userDetails, Model model) {
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

|

Extending \ ``AuthenticationProvider``\ 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In case of business requirements that cannot be supported by Spring Security's \ `authentication provider <http://docs.spring.io/spring-security/site/docs/3.2.5.RELEASE/apidocs/org/springframework/security/authentication/AuthenticationProvider.html>`_\ ,
it is necessary to create a class that implements \ ``org.springframework.security.authentication.AuthenticationProvider``\  interface.

Here, the extended example of DB authentication using 3 parameters such as user name, password and \ **Company identifier (independent authentication parameter)**\  is given below.

.. figure:: ./images/Authentication_HowToExtends_LoginForm.png
   :alt: Authentication_HowToExtends_LoginForm
   :width: 50%
  
To implement the above requirements, it is necessary to create a class given below.
  
.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Implementation class of \ ``org.springframework.security.core.Authentication``\  interface to store user name, password and company identifier (independent authentication parameter).

        Here, it is created by inheriting \ ``org.springframework.security.authentication.UsernamePasswordAuthenticationToken``\  class.
    * - | (2)
      - Implementation class of \ ``org.springframework.security.authentication.AuthenticationProvider``\  to perform DB authentication using user name, password and company identifier (independent authentication parameter).

        Here, it is created by inheriting \ ``org.springframework.security.authentication.dao.DaoAuthenticationProvider``\  class.
    * - | (3)
      - Servlet filter class to create \ ``Authentication``\  for fetching user name, password and company identifier (independent authentication parameter) from request parameter and passing them to \ ``AuthenticationManager``\  (\ ``AuthenticationProvider``\ ).

        Here, it is created by inheriting \ ``org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter``\  class.

.. tip::
	
    As the example below considers addition of independent parameter as an authentication parameter, 
    it is necessary to extend servlet filter class to generate \ ``Authentication``\  and implementation class of \ ``Authentication``\  interface.

    To authenticate only by user name and password, the authentication process can be extended, 
    only by creating implementation class of \ ``AuthenticationProvider``\  interface.

|

Extending \ ``UsernamePasswordAuthenticationToken``\ 
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Here, by inheriting \ ``UsernamePasswordAuthenticationToken``\  class, 
create a class that stores company identifier (independent authentication parameter) in addition to user name and password.

.. code-block:: java

    // import omitted
    public class CompanyIdUsernamePasswordAuthenticationToken extends
        UsernamePasswordAuthenticationToken {

        private static final long serialVersionUID = SpringSecurityCoreVersion.SERIAL_VERSION_UID;

        // (1)
        private final String companyId;

        // (2)
        public CompanyIdUsernamePasswordAuthenticationToken(
                Object principal, Object credentials, String companyId) {
            super(principal, credentials);

            this.companyId = companyId;
        }

        // (3)
        public CompanyIdUsernamePasswordAuthenticationToken(
                Object principal, Object credentials, String companyId,
                Collection<? extends GrantedAuthority> authorities) {
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
     - Create a field to store company identifier.
   * - | (2)
     - Create a constructor to be used while creating instances for storing the information (information specified in request parameter) before authentication.
   * - | (3)
     - | Create a constructor to be used while creating instances for storing the authenticated information.
       | Authentication completion status is reached by passing authorization information to the constructor argument of parent class.

|

Extending \ ``DaoAuthenticationProvide``\ 
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Here, by inheriting \ ``DaoAuthenticationProvider``\  class, 
create a class that performs DB authentication using user name, password and company identifier.

.. code-block:: java

    // import omitted
    public class CompanyIdUsernamePasswordAuthenticationProvider extends
        DaoAuthenticationProvider {

        // omitted

        @Override
        protected void additionalAuthenticationChecks(UserDetails userDetails,
                UsernamePasswordAuthenticationToken authentication)
                throws AuthenticationException {

            // (1)
            super.additionalAuthenticationChecks(userDetails, authentication);

            // (2)
            CompanyIdUsernamePasswordAuthenticationToken companyIdUsernamePasswordAuthentication =
                (CompanyIdUsernamePasswordAuthenticationToken) authentication;
            String requestedCompanyId = companyIdUsernamePasswordAuthentication.getCompanyId();
            String companyId = ((SampleUserDetails) userDetails)
                    .getAccount().getCompanyId();
            if (!companyId.equals(requestedCompanyId)) {
                throw new BadCredentialsException(messages.getMessage(
                        "AbstractUserDetailsAuthenticationProvider.badCredentials",
                        "Bad credentials"));
            }
        }

        @Override
        protected Authentication createSuccessAuthentication(Object principal,
                Authentication authentication, UserDetails user) {
            String companyId = ((SampleUserDetails) user).getAccount()
                    .getCompanyId();
            // (3)
            return new CompanyIdUsernamePasswordAuthenticationToken(user,
                    authentication.getCredentials(), companyId,
                    user.getAuthorities());
        }

        @Override
        public boolean supports(Class<?> authentication) {
            // (4)
            return CompanyIdUsernamePasswordAuthenticationToken.class
                    .isAssignableFrom(authentication);
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - Call parent class method and execute check process provided by Spring Security.

       Password authentication is performed at this time.
   * - | (2)
     - When password authentication is successful, validate company identifier (independent authentication parameter).

       In the above example, it is checked whether requested company identifier matches with the company identifier stored in the table.
   * - | (3)
     - When password authentication and independent authentication is successful, create and return authenticated \ ``CompanyIdUsernamePasswordAuthenticationToken``\ .
   * - | (4)
     - When \ ``Authentication``\ that can be casted is specified in \ ``CompanyIdUsernamePasswordAuthenticationToken``\ ,
       perform the authentication process using this class.

.. tip::

    User existence check, user status check (check for invalid users, locked users, validity expired users, etc.),
    are performed as processes of parent class before the \ ``additionalAuthenticationChecks``\  method is called.

|

.. _authentication_custom_usernamepasswordauthenticationfilter:

Extending \ ``UsernamePasswordAuthenticationFilter``\ 
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Here, by inheriting \ ``UsernamePasswordAuthenticationFilter``\  class, 
create a servlet filter class to pass the authentication information (user name, password, company identifier) to \ ``AuthenticationProvider``\ .

Implementation of \ ``attemptAuthentication``\  method is customized by copying \ ``UsernamePasswordAuthenticationFilter``\ class method.

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

            // (1)
            // Obtain UserName, Password, CompanyId
            String username = super.obtainUsername(request);
            String password = super.obtainPassword(request);
            String companyId = obtainCompanyId(request);
            if (username == null) {
                username = "";
            } else {
                username = username.trim();
            }
            if (password == null) {
                password = "";
            }
            CompanyIdUsernamePasswordAuthenticationToken authRequest =
                new CompanyIdUsernamePasswordAuthenticationToken(username, password, companyId);

            // Allow subclasses to set the "details" property
            setDetails(request, authRequest);

            return this.getAuthenticationManager().authenticate(authRequest); // (2)
        }

        // (3)
        protected String obtainCompanyId(HttpServletRequest request) {
            return request.getParameter("companyid");
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - Create an instance of \ ``CompanyIdUsernamePasswordAuthenticationToken``\  from authentication information (user name, password, company identifier) fetched from request parameter.
   * - | (2)
     - Call \ ``authenticate``\  method of \ ``org.springframework.security.authentication.AuthenticationManager``\  by specifying 
       authentication information (\ ``CompanyIdUsernamePasswordAuthenticationToken``\  instances) specified in request parameter.

       If \ ``AuthenticationManager``\  method is called, authentication process of \ ``AuthenticationProvider``\  is called.
   * - | (3)
     - Company identifier is fetched from the request parameter called \ ``"companyid"``\ .

.. note:: **Authentication information input check**

    Sometimes, a check needs to be performed in advance for the obvious input errors that occur due to load reduction on DB server.
    In such cases, input check process can be performed by extending \ ``UsernamePasswordAuthenticationFilter``\ , similar to
    \ :ref:`authentication_custom_usernamepasswordauthenticationfilter`\ .

    Input check is not performed in the above mentioned example. 

.. todo::

    Input check of authentication information can also be performed using Bean Validation by handling the request in Controller class.

    Input check method using Bean Validation will be added later.

|

Application of extended authentication process
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Apply the DB authentication function using user name, password, company identifier (independent authentication parameter) in Spring Security.

``spring-security.xml``

 .. code-block:: xml

    <!-- omitted -->

    <!-- (1) -->
    <sec:http
        auto-config="false"
        use-expressions="true"
        entry-point-ref="loginUrlAuthenticationEntryPoint">

        <!-- omitted -->

        <!-- (2) -->
        <sec:custom-filter
            position="FORM_LOGIN_FILTER"
            ref="companyIdUsernamePasswordAuthenticationFilter" />

        <!-- omitted -->

        <sec:csrf token-repository-ref="csrfTokenRepository" />

        <sec:logout
            logout-url="/logout"
            logout-success-url="/login"
            delete-cookies="JSESSIONID" />

        <!-- omitted -->

        <sec:intercept-url pattern="/login" access="permitAll" />
        <sec:intercept-url pattern="/**" access="isAuthenticated()" />

        <!-- omitted -->

    </sec:http>

    <!-- (3) -->
    <bean id="loginUrlAuthenticationEntryPoint"
        class="org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint">
        <constructor-arg value="/login" />
    </bean>

    <!-- (4) -->
    <bean id="companyIdUsernamePasswordAuthenticationFilter"
        class="com.example.app.common.security.CompanyIdUsernamePasswordAuthenticationFilter">
        <!-- (5) -->
        <property name="requiresAuthenticationRequestMatcher">
            <bean class="org.springframework.security.web.authentication.logout.LogoutFilter$FilterProcessUrlRequestMatcher">
                <constructor-arg value="/authentication" />
            </bean>
        </property>
        <!-- (6) -->
        <property name="authenticationManager" ref="authenticationManager" />
        <!-- (7) -->
        <property name="sessionAuthenticationStrategy" ref="sessionAuthenticationStrategy" />
        <!-- (8) -->
        <property name="authenticationFailureHandler">
            <bean class="org.springframework.security.web.authentication.SimpleUrlAuthenticationFailureHandler">
                <constructor-arg value="/login?error=true" />
            </bean>
        </property>
        <!-- (9) -->
        <property name="authenticationSuccessHandler">
            <bean class="org.springframework.security.web.authentication.SimpleUrlAuthenticationSuccessHandler" />
        </property>
    </bean>

    <!-- (6') -->
    <sec:authentication-manager alias="authenticationManager">
        <sec:authentication-provider ref="companyIdUsernamePasswordAuthenticationProvider" />
    </sec:authentication-manager>
    <bean id="companyIdUsernamePasswordAuthenticationProvider"
        class="com.example.app.common.security.CompanyIdUsernamePasswordAuthenticationProvider">
        <property name="userDetailsService" ref="sampleUserDetailsService" />
        <property name="passwordEncoder" ref="passwordEncoder" />
    </bean>

    <!-- (7') -->
    <bean id="sessionAuthenticationStrategy"
        class="org.springframework.security.web.authentication.session.CompositeSessionAuthenticationStrategy">
        <constructor-arg>
            <util:list>
                <bean class="org.springframework.security.web.csrf.CsrfAuthenticationStrategy">
                    <constructor-arg ref="csrfTokenRepository" />
                </bean>
                <bean class="org.springframework.security.web.authentication.session.SessionFixationProtectionStrategy" />
            </util:list>
        </constructor-arg>
    </bean>

    <bean id="csrfTokenRepository"
        class="org.springframework.security.web.csrf.HttpSessionCsrfTokenRepository" />


    <!-- omitted -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - To replace "FORM_LOGIN_FILTER" using \ ``custom-filter``\ element, it is necessary to perform the following settings in attributes of \ ``http``\  element.

        * Since it is not possible to use auto configuration, either set \ ``auto-config="false"``\  or delete \ ``auto-config``\  attribute.
        * Since it is not possible to use \ ``form-login``\  element, explicitly specify \ ``AuthenticationEntryPoint``\  to be used by using \ ``entry-point-ref``\  attribute.
    * - | (2)
      - Replace "FORM_LOGIN_FILTER" by using \ ``custom-filter``\  element.

        Specify \ ``"FORM_LOGIN_FILTER"``\  in \ ``position``\  attribute of \ ``custom-filter``\  element, and specify servlet filter bean ID extended in \ ``ref``\  attribute.
    * - | (3)
      - Define bean of \ ``AuthenticationEntryPoint``\  to be specified in \ ``entry-point-ref``\  attributes of \ ``http``\ element.

        Here, bean of \ ``org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint``\  class used while specifying \ ``form-login``\  element, is defined.
    * - | (4)
      - Define bean of servlet filter to be used as "FORM_LOGIN_FILTER".

        Here, bean of extended servlet filter class (\ ``CompanyIdUsernamePasswordAuthenticationFilter``\ ) is defined.
    * - | (5)
      - Specify \ ``RequestMatcher``\  instance for detecting request to perform authentication process, in \ ``requiresAuthenticationRequestMatcher``\  property.

        Here, if there is a request in \ ``/authentication``\  path, authentication process is performed.
        This is similar to specifying \ ``"/authentication"``\  in \ ``login-processing-url``\  attribute of \ ``form-login``\  element.
    * - | (6)
      - Specify the value which was set in \ ``alias``\  attribute of \ ``authentication-manager``\  element in \ ``authenticationManager``\  property.

        If \ ``alias``\  attribute of \ ``authentication-manager``\ element is specified,
        it is possible to inject dependency (DI) of \ ``AuthenticationManager``\  bean generated by Spring Security, to other bean.
    * - | (6')
      - Set extended \ ``AuthenticationProvider``\  (\ ``CompanyIdUsernamePasswordAuthenticationProvider``\ ) to \ ``AuthenticationManager``\  generated by Spring Security.
    * - | (7)
      - Specify bean of component (\ ``SessionAuthenticationStrategy``\ ) to control the session handling at the time of successful authentication, in \ ``sessionAuthenticationStrategy``\  property.

    * - | (7')
      - Define bean of component (\ ``SessionAuthenticationStrategy``\ ) to control the session handling at the time of successful authentication.

        Here, the following features provided by Spring Security are enabled.
        
        * Component to re-create CSRF token (\ ``CsrfAuthenticationStrategy``\ )
        * Component to generate new session to prevent session fixation attack (\ ``SessionFixationProtectionStrategy``\ )

    * - | (8)
      - | Specify handler class, called at the time of authentication failure, in \ ``authenticationFailureHandler``\ .
    * - | (9)
      - | Specify handler class, called at the time of successful authentication, in \ ``authenticationSuccessHandler``\ .

.. note::

    When \ ``auto-config="false"``\  is specified, \ ``<sec:http-basic>``\  element and \ ``<sec:logout>``\  element will not be enabled if not defined explicitly.

|

Creating login form
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Here, add company identifier for the screen (JSP), introduced in \ :ref:`form-login-JSP`\  .

.. code-block:: jsp
    :emphasize-lines: 5-6

    <form:form action="${pageContext.request.contextPath}/authentication" method="post">
        <!-- omitted -->
        <span>User Id</span><br>
        <input type="text" id="username" name="j_username"><br>
        <span>Company Id</span><br>
        <input type="text" id="companyid" name="companyid"><br>  <!-- (1) -->
        <span>Password</span><br>
        <input type="password" id="password" name="j_password"><br>
        <!-- omitted -->
    </form:form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify \ ``"companyid"``\  in the input field name of company identifier. 

|
Appendix
--------------------------------------------------------------------------------

Authentication Success handler for which destination can be specified
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
In case of authentication using Spring Security, user is transited to the following two paths if authentication is successful.

* Path described in bean definition file (\ ``spring-security.xml``\ ) (Path specified in \ ``default-target-url``\  attribute of \ ``<form-login>``\  element)
* Path for displaying "Secure page for which authentication is necessary" which is accessed before the login.

In common library, in addition to the functionality provided by Spring Security, 
a class (\ ``org.terasoluna.gfw.web.security.RedirectAuthenticationHandler``\ ) is provided which can specify the destination path in request parameter.

\ ``RedirectAuthenticationHandler``\  is a class, created for implementing the mechanism given below.

* For performing login for page display
* For specifying destination page after login at JSP side (source JSP)

.. figure:: ./images/Authentication_Appendix_ScreenFlow.png
   :alt: Authentication_Appendix_Screen_Flow
   :width: 70%
   :align: center

   **Picture - Screen_Flow**

| Example of using \ ``RedirectAuthenticationHandler``\ is shown below.

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
     - | As hidden field, set the "URL of destination page after successful login".
       | Specify "\ ``redirectTo``\ " as hidden field name (request parameter name).
       | 
       | Field name (request parameter name) should match with \ ``targetUrlParameter``\  property value of \ ``RedirectAuthenticationHandler``\ .

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
     - | As hidden field, set the "URL of destination page after successful login", passed by request parameter from source screen.
       | Specify "\ ``redirectTo``\ " as hidden field name (request parameter name).
       | 
       | Field name (request parameter name) should match with \ ``targetUrlParameter``\  property value of \ ``RedirectAuthenticationHandler``\ .

**Spring Security configuration file**

.. code-block:: xml

  <sec:http auto-config="true">
      <!-- omitted -->
      <!-- (1) -->
      <sec:form-login
          login-page="/login"
          login-processing-url="/authentication"
          authentication-failure-handler-ref="authenticationFailureHandler"
          authentication-success-handler-ref="authenticationSuccessHandler" />
      <!-- omitted -->
  </sec:http>

  <!-- (2) -->
  <bean id="authenticationSuccessHandler"
      class="org.terasoluna.gfw.web.security.RedirectAuthenticationHandler">
  </bean>

  <!-- (3) -->
  <bean id="authenticationFailureHandler"
      class="org.springframework.security.web.authentication.ExceptionMappingAuthenticationFailureHandler">
      <property name="defaultFailureUrl" value="/login?error=true"/> <!-- (4) -->
      <property name="useForward" value="true"/> <!-- (5) -->
  </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify BeanId of \ ``authentication-failure-handler-ref``\  (handler setting at the time of authentication error) and \ ``authentication-success-handler-ref``\  (handler setting in case of successful authentication).
   * - | (2)
     - | Define \ ``org.terasoluna.gfw.web.security.RedirectAuthenticationHandler``\ as a bean referred from \ ``authentication-success-handler-ref``\ .
   * - | (3)
     - | Define \ ``org.springframework.security.web.authentication.ExceptionMappingAuthenticationFailureHandler``\  as a bean referred from \ ``authentication-failure-handler-ref``\ .
   * - | (4)
     - | Set destination path at the time of authentication failure.
       | In above example, login screen path and the query (\ ``error=true``\ ) to indicate transition after authentication error, is set.
   * - | (5)
     - | **To use this functionality, useForward should be set to true.**
       | By setting it to \ ``true``\ , Forward is used instead of Redirect, at the time of transiting to the screen (Login screen) which would be displayed in case of authentication failure.

       | This is because "URL of destination page after successful login) needs to be included in the request parameter of authentication request.
       | By using Redirect, if authentication error screen is displayed, "URL of destination page after successful login" cannot be inherited from request parameter. Hence, it is not possible to transit to the specified screen even after login is successful.
       | To avoid this, it is necessary to ensure that "URL of destination page after successful login" is inherited from request parameter using Forward.

.. tip::

  Measures against Open Redirector vulnerability are provided in \ ``RedirectAuthenticationHandler``\ .
  Therefore, user cannot transit to external sites such as "http://google.com".
  In order to move to an external site, it is necessary to create the class implementing \ ``org.springframework.security.web.RedirectStrategy``\ ,
  and it is to be injected to \ ``targetUrlParameterRedirectStrategy``\  property of \ ``RedirectAuthenticationHandler``\ .
  
  As a precaution while extending, it is necessary to have a structure that does not pose any problems even if \ ``redirectTo``\ value is tampered with.
  For example, the measures given below can be considered.
  
  * Specifying the ID of page number etc. instead of directly specifying the destination URL and redirecting to the URL corresponding to that ID.
  * Checking the destination URL, and redirecting only the URL that matches with white list.

.. raw:: latex

   \newpage

