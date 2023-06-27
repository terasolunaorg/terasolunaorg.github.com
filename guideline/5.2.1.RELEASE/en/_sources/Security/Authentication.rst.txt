.. _SpringSecurityAuthentication:

Authentication
================================================================================

.. only:: html

 .. contents:: Table of contents
    :local:
    
.. _SpringSecurityAuthenticationOverview:

Overview
--------------------------------------------------------------------------------
This chapter explains about Authentication function provided by Spring Security.

Authentication process checks the validity of user using the application.

The standard method to check the validity of a user is to register the users who can use the application in a data store and
and verify the authentication information (user name, password etc) input by the user.
Although a relational database is commonly used in a data store wherein user information is registered, directory service or external system etc are also used.

Further, there are several methods which ask a user to input authentication information.
Although a method wherein input form of HTML or a method wherein an authentication method of HTTP standard defined in RFC (Basic authentication or Digest authentication etc) are used in general,
the methods like OpenID authentication or single sign-on authentication are also used.

This section introduces an implementation example wherein the authentication is done by verifying authentication information input in HTML input form and user information stored in the relational database,
and explains how to use authentication function of Spring Security.

|

Authentication process architecture
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Security performs authentication with the following flow.

.. figure:: ./images_Authentication/AuthenticationArchitecture.png
    :width: 100%

    **Authentication process architecture**

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Client specifies credentials (user name and password) for the path wherein authentication is performed and sends a request.
    * - | (2)
      - | Authentication Filter fetches credentials from the request and calls authentication process of \ ``AuthenticationManager``\  class.
    * - | (3)
      - | \ ``ProviderManager``\  (Implementation class of \ ``AuthenticationManager``\  used by default) delegates actual authentication process to implementation class of \ ``AuthenticationProvider``\  interface.

|

.. _SpringSecurityAuthenticationFilter:

Authentication Filter
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Authentication Filter is a servlet filter which offers implementation for the authentication method.
Primary authentication methods supported by Spring Security are as given below..

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: **Main Authentication Filter offered by Spring Security**
    :header-rows: 1
    :widths: 25 75

    * - Class Name
      - Description
    * - | \ ``UsernamePasswordAuthenticationFilter``\
      - | Fetch credentials from HTTP request parameter in servlet filter class for form authentication.
    * - | \ ``BasicAuthenticationFilter``\
      - | Fetch credentials from authentication header of HTTP request in servlet filter class for Basic authentication.
    * - | \ ``DigestAuthenticationFilter``\
      - | Fetch credentials from authentication header of HTTP request in servlet filter class for Digest authentication.
    * - | \ ``RememberMeAuthenticationFilter``\
      - | Fetch credentials from Cookies of HTTP request in servlet filter class for Remember Me authentication.
        | If Remember Me authentication is enabled, user stays logged in even if browser is closed and a session timeout has occurred.

These servlet filters are one of the Authentication filters introduced in :ref:`SpringSecurityProcess`\.

.. note::

    When an authentication process not supported by Spring Security must be adopted,
    it must be incorporated in Spring Security after creating a \ ``Authentication Filter``\  for adopting this authentication method.

|

AuthenticationManager
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``AuthenticationManager``\  is an interface for implementing the authentication process.
In default implementation (\ ``ProviderManager``\ ) offered by Spring Security, a system is adopted wherein
actual authentication process is delegated to \ ``AuthenticationProvider``\  and process results of authentication process performed by \ ``AuthenticationProvider``\  are handled.

|

AuthenticationProvider
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``AuthenticationProvider``\  is an interface which provides implementation of authentication process.
A key \ ``AuthenticationProvider``\  implementation class offered by Spring Security is as given below.

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: **A key AuthenticationProvider offered by Spring Security**
    :header-rows: 1
    :widths: 25 75

    * - Class Name
      - Description
    * - | \ ``DaoAuthenticationProvider``\
      - | Implementation class which performs authentication process by checking user credentials and user status registered in the data store.
        | Fetch credentials and user status required for checking, from the class which implements \ ``UserDetails``\  interface.

.. note::

    When an authentication process not offered by Spring Security must be adopted,
    it must be incorporated in Spring Security after creating an \ ``AuthenticationProvider``\  for implementing authentication process.

|

.. _howtouse_springsecurity:

How to use
--------------------------------------------------------------------------------

Bean definition example and implementation method required for using authentication function is explained.

This section, as explained in :ref:`SpringSecurityAuthenticationOverview`\ ,
explains a method to carry out authentication process by verifying credentials input in HTML input form and user information stored in the relational database.

.. _form-login:

Form authentication
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Security performs form authentication as per the flow given below.

.. figure:: ./images_Authentication/AuthenticationForm.png
    :width: 100%

    **Form authentication system**

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Client sends credentials (user name and password) as request parameters for the path which carries out form authentication.
    * - | (2)
      - | \ ``UsernamePasswordAuthenticationFilter``\  class fetches credentials from request parameter and calls authentication process of \ ``AuthenticationManager``\ .
    * - | (3)
      - | \ ``UsernamePasswordAuthenticationFilter``\  class handles the authentication results returned from \ ``AuthenticationManager``\ .
        | Call \ ``AuthenticationSuccessHandler``\  method when the authentication is successful, call \ ``AuthenticationFailureHandler``\  method when a failure occurs in the authentication and perform screen transition.

|

.. _form-login-usage:

Applying form authentication
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Bean is defined as below when form authentication is used.

* Definition example of spring-security.xml

.. code-block:: xml

    <sec:http>
        <sec:form-login />    <!-- (1) -->
        <!-- omitted -->
    </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Form authentication is enabled when \ ``<sec:form-login>``\  tag is defined.

.. tip:: **About auto-config attribute**

    An \ ``auto-config``\  attribute which specifies whether the settings for form authentication (\ ``<sec:form-login>``\  tag), Basic authentication (\ ``<sec:http-basic>``\  tag) and logout (\ ``<sec:logout>``\  tag) are performed automatically, is provided in \ ``<sec:http>``\ .
    Default value is \ ``false``\  (not set automatically) and it is recommended to use default value even in the reference document of Spring Security.

    Even in this guideline, a style wherein a tag is specified explicitly is recommended.

     .. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
     .. list-table::
         :header-rows: 1
         :widths: 25 75

         * - Element name
           - Description
         * - | ``<form-login>``\
           - | Security Filter(\ ``UsernamePasswordAuthenticationFilter``\ ) which performs form authentication process is applied.
         * - | \ ``<http-basic>``\
           - | Security Filter(\ ``BasicAuthenticationFilter``\ ) which performs Basic authentication in conformance with RFC1945 is applied.
             | For detailed usage method, refer \ `JavaDoc of BasicAuthenticationFilter <http://docs.spring.io/spring-security/site/docs/4.0.4.RELEASE/apidocs/org/springframework/security/web/authentication/www/BasicAuthenticationFilter.html>`_\ .
         * - | \ ``<logout>``\
           - | Security Filter(\ ``LogoutFilter``\ ) which performs logout process is applied.
             | For details of logout process, refer "\ :ref:`SpringSecurityAuthenticationLogout`\ ".

    Further note that when ``auto-config``\  is not defined, form authentication (\ ``<sec:form-login>``\  tag) or Basic authentication (\ ``<sec:http-basic>``\  tag) must be defined.
    This is required to meet the specification of Spring Security wherein a Bean must be defined for one or more Authentication Filters in a single \ ``SecurityFilterChain``\  (\ ``<sec:http>``\).

.. _form-login-default-operation:

Default operation
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

If user account is accessed for \ ``"/login"``\  using GET method in the default operation of Spring Security, a default login form provided by Spring Security is displayed
and when login button is clicked, \ ``"/login"``\  is accessed by POST method and authentication process is carried out.

|

.. _SpringSecurityAuthenticationLoginForm:

Creating login form
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Although a login form for form authentication is provided in Spring Security as a default, it is rarely used as it is.
Below, a method wherein an auto-created login form is applied in Spring Security is explained.

At first, a JSP for displaying a login form is created.
An implementation example wherein a login form is displayed after receiving a request in Spring MVC is given below.

* How to create a JSP for displaying a login form (xxx-web/src/main/webapp/WEB-INF/views/login/loginForm.jsp)

.. code-block:: jsp

    <%@ page contentType="text/html;charset=UTF-8" pageEncoding="UTF-8" %>
    <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
    <%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags" %>
    <%-- omitted --%>
    <div id="wrapper">
        <h3>Login Screen</h3>
        <%-- (1) --%>
        <c:if test="${param.containsKey('error')}">
            <t:messagesPanel messagesType="error"
                messagesAttributeName="SPRING_SECURITY_LAST_EXCEPTION"/> <%-- (2) --%>
        </c:if>
        <form:form action="${pageContext.request.contextPath}/login" method="post"> <%-- (3) --%>
            <table>
                <tr>
                    <td><label for="username">User Name</label></td>
                    <td><input type="text" id="username" name="username"></td>
                </tr>
                <tr>
                    <td><label for="password">Password</label></td>
                    <td><input type="password" id="password" name="password"></td>
                </tr>
                <tr>
                    <td>&nbsp;</td>
                    <td><button>Login</button></td>
                </tr>
            </table>
        </form:form>
    </div>
    <%-- omitted --%>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | An area to display authentication error.
    * - | (2)
      - | Output an exception message which is output at the time of authentication error.
        | It is recommended to output by using \ ``<t:messagesPanel>``\  tag provided by a common library.
        | For how to use \ ``<t:messagesPanel>``\  tag, refer "\ :doc:`../ArchitectureInDetail/WebApplicationDetail/MessageManagement`\ ".
        | Note that, when an authentication error occurs, exception object is stored in the session or request scope with the attribute name \ ``"SPRING_SECURITY_LAST_EXCEPTION"``\ .
    * - | (3)
      - | Login form to enter user name and password.
        | Send user name and password to request parameters \ ``username``\  and \ ``password``\  respectively.
        | Also note that token value for CSRF measures is sent to request parameter by using \ ``<form:form>``\ .
        | CSRF measures are explained in ":ref:`SpringSecurityCsrf`".

|

Next, login form thus created is applied to Spring Security.

* Definition example of spring-security.xml

.. code-block:: xml

    <sec:http>
      <sec:form-login 
          login-page="/login/loginForm"
          login-processing-url="/login" /> <!-- (1)(2) -->
      <sec:intercept-url pattern="/login/**" access="permitAll"/>  <!-- (3) -->
      <sec:intercept-url pattern="/**" access="isAuthenticated()"/> <!-- (4) -->
    </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify path to display login form in \ ``login-page``\  attribute. 
        | When an anonymous user accesses a Web resource for which authentication is required, the user is redirected to a path specified in the attribute and a login form is displayed.
        | Here, a request is received by Spring MVC and a login form is displayed.
        | For details, refer ":ref:`spring-security-authentication-mvc`".
    * - | (2)
      - | Specify path for performing authentication process in \ ``login-processing-url``\  attribute.
        | Although default path is also \ ``"/login"``\ , it should be explicitly specified.
    * - | (3)
      - | Assign the rights enabling access to all users for the location under \ ``/login``\  path where login form is stored.
        | For how to specify access policy for web resource, refer "\ :ref:`SpringSecurityAuthorization`\".
    * - | (4)
      - | Assign the access rights for web resource handled by the application.
        | In the example above, only authenticated users are granted the rights to access location under root path of web application.
        | For how to specify access policy for web resource, refer "\ :ref:`SpringSecurityAuthorization`\".

.. note:: **Changes in Spring Security 4.0**

    Default values of following configuration are changed from Spring Security version 4.0

    * username-parameter
    * password-parameter
    * login-processing-url
    * authentication-failure-url 

|

.. _SpringSecurityAuthenticationScreenFlowOnSuccess:

Response when authentication is successful
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Security offers \ ``AuthenticationSuccessHandler``\  interface and implementation class as the components
to control the response when the authentication is successful.

.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table:: **Implementation class of AuthenticationSuccessHandler**
    :header-rows: 1
    :widths: 35 65

    * - Implementation class
      - Description
    * - | \ ``SavedRequestAwareAuthenticationSuccessHandler``\
      - | Implementation class that redirects to a URL which the user have attempted to access prior to authentication.
        | **Implementation class used by default.**
    * - | \ ``SimpleUrlAuthenticationSuccessHandler``\
      - | Implementation class which redirects or forwards to \ ``defaultTargetUrl``\ .

Default operation
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

In the default operation of Spring Security, the request that was denied prior to authentication is saved in HTTP session
and when the authentication is successful, the request which was denied access is restored and redirected.
Page is displayed if the authenticated user has the access rights for redirected page, authentication error occurs if the user does not have the access rights.
\ ``SavedRequestAwareAuthenticationSuccessHandler``\  class is used to carry out this operation.

Since the transition destination after explicitly displaying login form and performing authentication is the root path of web application (\ ``"/"``\ ),
as per default configuration of Spring Security, user is redirected to the root path of Web application when the authentication is successful.

|

.. _SpringSecurityAuthenticationScreenFlowOnFailure:

Response when authentication fails
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Security offers \ ``AuthenticationFailureHandler``\  interface and implementation class as the components
to control the response when the authentication fails.

.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table:: **Implementation class of AuthenticationFailureHandler**
    :header-rows: 1
    :widths: 35 65

    * - Implementation class
      - Description
    * - | \ ``SimpleUrlAuthenticationFailureHandler``\
      - | Implementation class which redirects or forwards to a specified path (\ ``defaultFailureUrl``\ ).
    * - | \ ``ExceptionMappingAuthenticationFailureHandler``\
      - | Implementation class which can map authentication exception and URL for transition.
        | Since the exception class generated by Spring Security for each error cause vary, the transition destination can be changed for each error type if this implementation class is used.
    * - | \ ``DelegatingAuthenticationFailureHandler``\
      - | Implementation class which can map authentication exception and \ ``AuthenticationFailureHandler``\ . 
        | Although it resembles \ ``ExceptionMappingAuthenticationFailureHandler``\ , a flexible behaviour can be supported by specifying \ ``AuthenticationFailureHandler``\  for each authentication exception.

Default operation
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

In the default operation of Spring Security, user is redirected to a URL assigned with a query parameter \ ``"error"``\ in a path which displays login form.

As an example, when the path to display login form is \ ``"/login"``\ , the user is redirected to \ ``"/login?error"``\ .
  

|

DB authentication
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Security performs DB authentication in the flow given below.

.. figure:: ./images_Authentication/AuthenticationDatabase.png
    :width: 100%

    **DB authentication system**

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - Sr. No.
      - Description
    * - | (1)
      - | Spring Security receives authentication request from the client and calls authentication process of \ ``DaoAuthenticationProvider``\ .
    * - | (2)
      - | \ ``DaoAuthenticationProvider``\  calls user information fetch process of \ ``UserDetailsService``\ .
    * - | (3)
      - | Implementation class of ``UserDetailsService``\  fetches user information from data store.
    * - | (4)
      - | Implementation class of ``UserDetailsService``\  generates \ ``UserDetails``\  from the user information fetched from data store.
    * - | (5)
      - | \ ``DaoAuthenticationProvider``\  verifies \ ``UserDetails``\  returned from \ ``UserDetailsService``\  and authentication information specified by client, and checks validity of user specified by client.

.. raw:: latex

   \newpage

.. note:: **DB authentication offered by Spring Security**

    Spring Security offers an implementation class to fetch user information from relational database through JDBC.

    * \ ``org.springframework.security.core.userdetails.User``\  (Implementation class of \ ``UserDetails``\ )
    * \ ``org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl`` \  (Implementation class of \ ``UserDetailsService``\ )

    Since these implementation classes only perform authentication processes of minimum level (password verification, determination of user validity), they are used as it is very rarely.
    Hence, this guideline explains how to create implementation classes of \ ``UserDetails``\  and \ ``UserDetailsService``\ .

|

Creating UserDetails
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``UserDetails``\  is an interface which provides credentials (user name and password) and user status necessary in the authentication process and defines following methods.
When \ ``DaoAuthenticationProvider``\  is used as \ ``AuthenticationProvider``\ , implementation class of \ ``UserDetails``\  is created in accordance with the requirements of application.

*UserDetails interface*

.. code-block:: java

    public interface UserDetails extends Serializable {
        String getUsername(); // (1)
        String getPassword(); // (2)
        boolean isEnabled(); // (3)
        boolean isAccountNonLocked(); // (4)
        boolean isAccountNonExpired(); // (5)
        boolean isCredentialsNonExpired(); // (6)
        Collection<? extends GrantedAuthority> getAuthorities(); // (7)
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.25\linewidth}|p{0.65\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 25 65
    :class: longtable

    * - Sr. No.
      - Method name
      - Description
    * - | (1)
      - | \ ``getUsername``\
      - | Return user name.
    * - | (2)
      - | \ ``getPassword``\
      - | Return registered password.
        | When the password returned by this method and the password specified by the client do not match, \ ``DaoAuthenticationProvider``\  throws \ ``BadCredentialsException``\ .
    * - | (3)
      - | \ ``isEnabled``\
      - | Determine whether the user is valid. Return \ ``true``\  in case of valid user.
        | When the user is invalid, \ ``DaoAuthenticationProvider``\  throws \ ``DisabledException``\ .
    * - | (4)
      - | \ ``isAccountNonLocked``\
      - | Determine locked status of the account. When the account is not locked, return \ ``true``\ .
        | When the account is locked, \ ``DaoAuthenticationProvider``\  throws \ ``LockedException``\ .
    * - | (5)
      - | \ ``isAccountNonExpired``\
      - | Determine expiry status of the account. Return \ ``true``\  when the account is not expired.
        | When the account is expired, \ ``DaoAuthenticationProvider``\  throws \ ``AccountExpiredException``\ .
    * - | (6)
      - | \ ``isCredentialsNonExpired``\
      - | Determine expiry status of credentials. Return \ ``true``\  when the credentials are not expired.
        | When the credentials are expired, \ ``DaoAuthenticationProvider``\  throws \ ``CredentialsExpiredException``\ .
    * - | (7)
      - | \ ``getAuthorities``\
      - | Return list of rights assigned to the user.
        | This method is used in the authorization process.

.. raw:: latex

   \newpage

.. note:: **Changing transition destination based on authentication exception**

    When the user wants to change the screen transition for each exception thrown by \ ``DaoAuthenticationProvider``\ ,
    \ ``ExceptionMappingAuthenticationFailureHandler``\  must be used as \ ``AuthenticationFailureHandler``\ .

    As an example, when user wants to transit to password change screen in case of password expiry for the user, screen transition can be changed
    by handling \ ``CredentialsExpiredException``\  by using \ ``ExceptionMappingAuthenticationFailureHandler``\ .
    
    For details, refer :ref:`SpringSecurityAuthenticationCustomizingScreenFlowOnFailure`\ .

.. note:: **Credentials provided by Spring Security**

    Although Spring Security provides an implementation class (\ ``org.springframework.security.core.userdetails.User``\ ) for retaining credentials (user name and password) and user status,
    it can store only the information required for the authentication process.
    Since the information of the user (name of the user etc) which is not used in the authentication process is frequently required in the general applications, \ ``User``\  class can rarely be used as it is.

|

Here, an implementation class of \ ``UserDetails``\  which retains account information is created.
This example can also be implemented by inheriting \ ``User``\ . However, here an example wherein \ ``UserDetails``\  is implemented is introduced.

* How to create implementation class of UserDetails


.. code-block:: java

    public class AccountUserDetails implements UserDetails { // (1)

        private final Account account;
        private final Collection<GrantedAuthority> authorities;

        public AccountUserDetails(
            Account account, Collection<GrantedAuthority> authorities) {
            // (2)
            this.account = account;
            this.authorities = authorities;
        }

        // (3)
        public String getPassword() {
            return account.getPassword();
        }
        public String getUsername() {
            return account.getUsername();
        }
        public boolean isEnabled() {
            return account.isEnabled();
        }
        public Collection<GrantedAuthority> getAuthorities() {
            return authorities;
        }

        // (4)
        public boolean isAccountNonExpired() {
            return true;
        }
        public boolean isAccountNonLocked() {
            return true;
        }
        public boolean isCredentialsNonExpired() {
            return true;
        }

        // (5)
        public Account getAccount() {
            return account;
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Create a class which implements \ ``UserDetails``\  interface.
    * - | (2)
      - | Retain user information and rights information in the property.
    * - | (3)
      - | Implement method defined in \ ``UserDetails``\  interface.
    * - | (4)
      - | Although the example of this sections does not implement check for, "Account locking", "Account expiry" and "Credentials expiry", these must be checked in accordance with the requirements.
    * - | (5)
      - | Provide a getter method to enable access to account information in the process after successful completion of authentication.

|

Spring Security provides \ ``User``\  class as an implementation class of \ ``UserDetails``\ .
When you inherit a \ ``User``\  class, credentials and user status can be easily retained.

* How to create UserDetails implementation class which inherits User class

.. code-block:: java

    public class AccountUserDetails extends User {

        private final Account account;

        public AccountUserDetails(Account account, boolean accountNonExpired,
                boolean credentialsNonExpired, boolean accountNonLocked,
                Collection<GrantedAuthority> authorities) {
            super(account.getUsername(), account.getPassword(),
                    account.isEnabled(), true, true, true, authorities);
            this.account = account;
        }

        public Account getAccount() {
            return account;
        }
    }

|

.. _SpringSecurityAuthenticationUserDetailsService:

Creating UserDetailsService
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``UserDetailsService``\  is an interface to fetch credentials and user status required for authentication process
from data store and defines following methods.
When \ ``DaoAuthenticationProvider``\  is used as \ ``AuthenticationProvider``\ ,
implementation class of \ ``UserDetailsService``\  is created in accordance with the requirements of the application.

* UserDetailsService interface

.. code-block:: java

    public interface UserDetailsService {
        UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
    }

|

Here, a service class is created to search account information from the database and
generate an instance of \ ``UserDetails``\ .
In this sample, account information is fetched by using \ ``SharedService``\ .
For \ ``SharedService``\ , refer :ref:`service-label`\ .

* How to create AccountSharedService interface

.. code-block:: java

    public interface AccountSharedService {
        Account findOne(String username);
    }

* How to create implementation class of AccountSharedService

.. code-block:: java

    // (1)
    @Service
    @Transactional
    public class AccountSharedServiceImpl implements AccountSharedService {
        @Inject
        AccountRepository accountRepository;

        // (2)
        @Override
        public Account findOne(String username) {
            Account account = accountRepository.findOneByUsername(username);
            if (account == null) {
                throw new ResourceNotFoundException("The given account is not found! username="
                        + username);
            }
            return account;
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Create a class which implements \ ``AccountSharedService``\  interface and assign \ ``@Service``\ .
        | In the example above, \ ``AccountSharedServiceImpl``\  is registered in DI container using component scan function.
    * - |  (2)
      - | Search account information from the database.
        | When account information is not found, an exception of common library - \ ``ResourceNotFoundException``\  is thrown.
        | For how to create a repository, refer ":doc:`../Tutorial/TutorialSecurity`".

* How to create implementation class of UserDetailsService

.. code-block:: java

    // (1)
    @Service
    @Transactional
    public class AccountUserDetailsService implements UserDetailsService {
        @Inject
        AccountSharedService accountSharedService;

        public UserDetails loadUserByUsername(String username)
                throws UsernameNotFoundException {

            try {
                Account account = accountSharedService.findOne(username);
                // (2)
                return new AccountUserDetails(account, getAuthorities(account));
            } catch (ResourceNotFoundException e) {
                // (3)
                throw new UsernameNotFoundException("user not found", e);
            }
        }

        // (4)
        private Collection<GrantedAuthority> getAuthorities(Account account) {
            if (account.isAdmin()) {
                return AuthorityUtils.createAuthorityList("ROLE_USER", "ROLE_ADMIN");
            } else {
                return AuthorityUtils.createAuthorityList("ROLE_USER");
            }
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Create a class which implements \ ``UserDetailsService``\  interface and assign \ ``@Service``\ .
        | In the example above, \ ``UserDetailsService``\  is registered in DI container using a component scan function.
    * - | (2)
      - | Fetch account information by using \ ``AccountSharedService``\ .
        | When the account information is found, \ ``UserDetails``\  is generated.
        | In the example above, user name, password and user validity status are fetched from account information.
    * - | (3)
      - | When the account information is not found, \ ``UsernameNotFoundException``\  is thrown.
    * - | (4)
      - | Generate information about the rights (role) retained by the user. The rights (role) generated here are used in the authorization process.

.. note:: **Information of rights to be used in the authorization**

    Authorization process of Spring Security handles the rights information starting with \ ``"ROLE_"``\ as a role.
    Hence, when resource access is to be controlled by using role, \ ``"ROLE_"``\  must be assigned as a prefix to the rights information which is being handled as a role.

.. note:: **Concealing authentication exception information**

    In the default operation of Spring Security, error handling is performed after changing \ ``UsernameNotFoundException``\  to an exception called \ ``BadCredentialsException``\ .
    \ ``BadCredentialsException``\  denotes an error in any field of the credentials specified by the client, specific error causes are not notified to the client.

|

.. _AuthenticationProviderConfiguration:

Applying DB authentication
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When the authentication is to be performed by using \ ``UserDetailsService``\  thus created,
\ ``UserDetailsService``\  thus created must be applied by enabling \ ``DaoAuthenticationProvider``\ .

* Definition example of spring-security.xml

.. code-block:: xml

    <sec:authentication-manager> <!-- (1) -->
        <sec:authentication-provider user-service-ref="accountUserDetailsService"> <!-- (2) -->
            <sec:password-encoder ref="passwordEncoder" /> <!-- (3) -->
        </sec:authentication-provider>
    </sec:authentication-manager>

    <bean id="passwordEncoder"
        class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder" /> <!-- (4) -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Define a Bean for \ ``AuthenticationManager``\ .
    * - | (2)
      - | Define ``<sec:authentication-provider>``\  element in the \ ``<sec:authentication-manager>``\  element.
        | Specify a Bean of ``AccountUserDetailsService``\  created by ":ref:`SpringSecurityAuthenticationUserDetailsService`", in ``user-service-ref``\  attribute.
        | Based on this definition, \ ``DaoAuthenticationProvider``\  of default setup is enabled.
    * - | (3)
      - | Specify a Bean of \ ``PasswordEncoder``\  to be used at the time of password verification.
    * - | (4)
      - | Define a Bean for \ ``PasswordEncoder``\  to be used at the time of password verification.
        | In the example above, \ ``BCryptPasswordEncoder``\  is defined which performs password hashing by using BCrypt algorithm.
        | For password hashing, refer ":ref:`SpringSecurityAuthenticationPasswordHashing`".

|

.. _SpringSecurityAuthenticationPasswordHashing:

Password hashing
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When a password is stored in the database, password is not stored as it is and a hash value of the password is generally stored.


Spring Security provides an interface and implementation class for password hashing
and operates in coordination with the authentication function.

There are 2 types of interfaces provided by Spring Security.

* \ ``org.springframework.security.crypto.password.PasswordEncoder``\
* \ ``org.springframework.security.authentication.encoding.PasswordEncoder``\

Although both the interfaces include \ ``PasswordEncoder``\ , \ ``PasswordEncoder``\  of
\ ``org.springframework.security.authentication.encoding``\  package is deprecated.

When there are no specific constraints in the password hashing requirements, using the implementation class of \ ``PasswordEncoder``\  interface of
\ ``org.springframework.security.crypto.password``\  package is recommended.

.. note::

    For how to use deprecated \ ``PasswordEncoder``\ , refer
    ":ref:`AuthenticationHowToExtendUsingDeprecatedPasswordEncoder`".

|

*Definition of a method for org.springframework.security.crypto.password.PasswordEncoder*

.. code-block:: java

    public interface PasswordEncoder {
        String encode(CharSequence rawPassword);
        boolean matches(CharSequence rawPassword, String encodedPassword);
    }

.. tabularcolumns:: |p{0.15\linewidth}|p{0.85\linewidth}|
.. list-table:: **Method defined in PasswordEncoder**
    :header-rows: 1
    :widths: 15 85

    * - Method name
      - Description
    * - | \ ``encode``\
      - | A method for password hashing.
        | It can be used for hashing of passwords which are stored in the data store during account registration process and password change process.
    * - | \ ``matches``\
      - | A method to verify plain text password and hashed password.
        | This method can also be used in the authentication process of Spring Security and can also be used to verify current password and the password used earlier during password change process etc.

|

Spring Security offers following classes as the implementation class of \ ``PasswordEncoder``\  interface.

.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table:: **Implementation class of PasswordEncoder**
    :header-rows: 1
    :widths: 35 65

    * - Implementation class
      - Description
    * - | \ ``BCryptPasswordEncoder``\
      - | Implementation class which performs password hashing and verification by using BCrypt algorithm.
        | **When there are no specific constraints in password hashing requirement, using this class is recommended.**
        | For details, refer \ `JavaDoc of BCryptPasswordEncoder <http://docs.spring.io/spring-security/site/docs/4.0.4.RELEASE/apidocs/org/springframework/security/crypto/bcrypt/BCryptPasswordEncoder.html>`_\ .
    * - | \ ``StandardPasswordEncoder``\
      - | Implementation class which performs password hashing and verification by using SHA-256 algorithm.
        | For details, refer \ `JavaDoc of StandardPasswordEncoder <http://docs.spring.io/spring-security/site/docs/4.0.4.RELEASE/apidocs/org/springframework/security/crypto/password/StandardPasswordEncoder.html>`_\ .
    * - | \ ``NoOpPasswordEncoder``\
      - | Implementation class which does not perform hashing.
        | It is a class used for testing and is not used in the actual application.

This section explains how to use \ ``BCryptPasswordEncoder``\ that has been recommended by Spring Security.

|

BCryptPasswordEncoder
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``BCryptPasswordEncoder``\  is an implementation class which performs password hashing and password verification by using BCrypt algorithm.
16 byte random numbers (\ ``java.security.SecureRandom``\ ) are used in :ref:`Salt<SpringSecurityAuthenticationPasswordHashSalt>`,
:ref:`Stretching<SpringSecurityAuthenticationPasswordHashStength>` is done for 1024 (2 to the power of 10) times by default..

* Definition example of applicationContext.xml

.. code-block:: xml

  <bean id="passwordEncoder"
      class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder" > <!-- (1) -->
      <constructor-arg name="strength" value="11" /> <!-- (2) -->
  </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify \ ``BCryptPasswordEncoder``\  in passwordEncoder class.
    * - | (2)
      - | Specify number of rounds for stretching count of hashing in the argument of constructor.
        | This argument can be omitted and the values that can be specified are in the range from \ ``4``\  to \ ``31``\ .
        | Note that default value is \ ``10``\  when the argument value is not specified.
        | The explanation is omitted in the guideline, however \ ``java.security.SecureRandom.SecureRandom``\  can also be specified as a constructor argument.

.. warning:: **How to use SecureRandom**
  
    When \ ``SecureRandom``\  is to be used in Linux environment, a delay or timeout in the processing is likely to occur.
    This event depends on the random number generator to be used and description is given in the document given below.
  
    * https://docs.oracle.com/javase/8/docs/api/java/security/SecureRandom.html 
  
    When this event occurs, it can be avoided by adding one of the following settings.
  
    * Specify ``-Djava.security.egd=file:/dev/urandom`` while executing Java command.
  
    * Change ``securerandom.source=/dev/random` in ``${JAVA_HOME}/jre/lib/security/java.security`` to ``securerandom.source=/dev/urandom``.
  
    If this event occurs in the version prior to b19 of Java SE 7 (prior to official release), ``/dev/./urandom`` must be specified instead of ``/dev/urandom``. However, algorithm used by \ ``SecureRandom``\  cannot be avoided in case of \ ``NativePRNG``\.

|

\ ``PasswordEncoder``\  is used in the class which carries out a process by using \ ``BCryptPasswordEncoder``\, by injecting from DI container.

.. code-block:: java

    @Service
    @Transactional
    public class AccountServiceImpl implements AccountService {

        @Inject
        AccountRepository accountRepository;

        @Inject
        PasswordEncoder passwordEncoder; // (1)

        public Account register(Account account, String rawPassword) {
            // omitted
            String encodedPassword = passwordEncoder.encode(rawPassword); // (2)
            account.setPassword(encodedPassword);
            // omitted
            return accountRepository.save(account);
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Inject \ ``PasswordEncoder``\ .
    * - | (2)
      - | Call injected \ ``PasswordEncoder``\  method.
        | Here, the password stored in the data store is hashed.

.. _SpringSecurityAuthenticationPasswordHashSalt:

.. note:: **Salt**

    A salt is a string which is added to data targeted for hashing.
    Since number of digits exceed the actual password length by assigning a salt to a password, password analysis using programs like rainbow crack become difficult.
    Note that, **it is recommended to set different values of salt (random values) for each user.**
    This is necessary because if identical salt value is used, the string (password) before hashing can be easily identified from the hash value.

.. _SpringSecurityAuthenticationPasswordHashStength:

.. note:: **Stretching**

    Information related to stored password is repeatedly encrypted by iterating hash function calculation in order to
    extend the time required for password analysis as a countermeasure against password brute force attack.
    However, stretching frequency must be determined considering the system performance since stretching is likely to impact system performance.

    In Spring Security, stretching is done for 1024 times (2 to the power of 10) by default, however this frequency can be changed by constructor argument (\ ``strength``\ ).
    A value from 4 (16 times) to 31 (2, 147, 483, 648 times) can be specified in the \ ``strength``\ .
    More the frequency of stretching, stronger the password. However, it is likely to impact performance due to higher complexity.

|

.. _SpringSecurityAuthenticationEvent:

Handling authentication event
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Security offers a system which links process results of authentication process with other components
using an event notification system offered by Spring Security.

If this system is used, following security requirements can be incorporated in the authentication function of Spring Security.

* Save authentication history of successful and failed authentication in database or log.
* Lock account if wrong password is entered in succession.

Authentication event is notified as given below.

.. figure:: ./images_Authentication/AuthenticationEventNotification.png
    :width: 100%

    **Event notification system**

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Spring Security authentication function passes authentication results (authentication information and authentication exception)
        | in \ ``AuthenticationEventPublisher``\  and requests notification of authentication event.
    * - | (2)
      - | Default implementation class of \ ``AuthenticationEventPublisher``\  interface generates an instance of
        | \ authentication event class corresponding to authentication results, passes to \ ``ApplicationEventPublisher``\  and requests event notification.
    * - | (3)
      - | Implementation class of \ ``ApplicationEventPublisher``\  interface notifies the event to implementation class of \ ``ApplicationListener``\  interface.
    * - | (4)
      - | \ ``ApplicationListenerMethodAdaptor``\, one of the implementation classes of ``ApplicationListener``\  calls a method
        | assigned by \ ``@org.springframework.context.event.EventListener``\  and notifies an event.

.. note:: **Memo**

    Although it was necessary to receive the event by creating an implementation class of \ ``ApplicationListener``\ till Spring 4.1,
    an event can be created in Spring 4.2 only by implementing a method which assigns \ ``@EventListener``\  to POJO.
    Note that, even in subsequent versions of Spring 4.2, an event can be received by creating an implementation class of \ ``ApplicationListener``\  interface as earlier.

Events used by Spring Security can be classified into 2 types - the events which notify successful authentication and the events which notify failure in authentication.
Event class offered by Spring Security are explained below.

|

Authentication successful event
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

There are 3 main events which are notified by Spring Security at the time of successful authentication.
These events are notified in the following order unless an error occurs in between.

.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table:: **Event class which notifies that the authentication is successful**
    :header-rows: 1
    :widths: 35 65

    * - Event class
      - Description
    * - \ ``AuthenticationSuccessEvent``\
      - An event class to notify that the authentication process by \ ``AuthenticationProvider``\  is successful.
        If this event is handled, it can be detected whether the client has specified correct information.
        It must be noted that an error is likely to occur in the subsequent processes after handling this event.
    * - \ ``SessionFixationProtectionEvent``\
      - An event class to notify that session fixation attack countermeasure process (change process of session ID) is successful.
        If this event is handled, session ID after the change can be detected.
    * - \ ``InteractiveAuthenticationSuccessEvent``\
      - An event class to notify that authentication process is entirely successful.
        If this event is handled, it can be detected that the overall authentication process is successful except the screen transition.

|

Authentication failure event
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Main events which are notified by Spring Security at the time of authentication failure are as below.
When authentication fails, any one of the events is notified.

.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table:: **Event class which notifies that authentication has failed**
    :header-rows: 1
    :widths: 35 65

    * - Event class
      - Description
    * - | \ ``AuthenticationFailureBadCredentialsEvent``\
      - | Event class which notifies that \ ``BadCredentialsException``\  has occurred.
    * - | \ ``AuthenticationFailureDisabledEvent``\
      - | Event class which notifies that \ ``DisabledException``\  has occurred.
    * - | \ ``AuthenticationFailureLockedEvent``\
      - | Event class which notifies that \ ``LockedException``\  has occurred.
    * - | \ ``AuthenticationFailureExpiredEvent``\
      - | Event class which notifies that \ ``AccountExpiredException``\  has occurred.
    * - | \ ``AuthenticationFailureCredentialsExpiredEvent``\
      - | Event class which notifies that \ ``CredentialsExpiredException``\  has occurred.
    * - | \ ``AuthenticationFailureServiceExceptionEvent``\
      - | Event class which notifies that \ ``AuthenticationServiceException``\  has occurred.

|

Creating event listener
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When you want to carry out a process by receiving authentication event notification, a class implementing a method that assigns \ ``@EventListener``\  is created and registered in DI container.

* Implementation example for event listener class

.. code-block:: java

    @Component
    public class AuthenticationEventListeners {

        private static final Logger log =
                LoggerFactory.getLogger(AuthenticationEventListeners.class);

    @EventListener // (1) 
    public void handleBadCredentials( 
        AuthenticationFailureBadCredentialsEvent event) { // (2) 
        log.info("Bad credentials is detected. username : {}", event.getAuthentication().getName()); 
        // omitted 
    } 


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Create a method which assigns ``@EventListener``\  to a method.
    * - | (2)
      - | Specify an authentication event class which is to be handled in the method argument.

The example above is an example wherein a class is created that handles \ ``AuthenticationFailureBadCredentialsEvent``\  that is notified when the client specified authentication information is incorrect.
Other events can also be handled in the same manner.

|

.. _SpringSecurityAuthenticationLogout:

Logout
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Security performs logout process as per the flow given below.

.. figure:: ./images_Authentication/AuthenticationLogout.png
    :width: 100%

    **Logout process system**

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Client sends a request to a path intended for logout process.
    * - | (2)
      - | \ ``LogoutFilter``\  calls \ ``LogoutHandler``\  method and performs actual logout process.
    * - | (3)
      - | \ ``LogoutFilter``\  calls \ ``LogoutSuccessHandler``\  method and performs screen transition.

|

A plurality of implementation classes of \ ``LogoutHandler``\  play respective roles as below.

.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table:: **Implementation class of main LogoutHandler**
    :header-rows: 1
    :widths: 35 65

    * - Implementation class
      - Description
    * - | \ ``SecurityContextLogoutHandler``\
      - | Class which clears authentication information of login user and cancels a session.
    * - | \ ``CookieClearingLogoutHandler``\
      - | Class which sends a response for deleting specific cookies.
    * - | \ ``CsrfLogoutHandler``\
      - | Class which deletes a token for CSRF countermeasure.

Since \ ``LogoutHandler``\ automatically sets the class which supports bean definition offered by Spring Security, in \ ``LogoutFilter``\ ,
there is no need for a developer to be specifically aware of the same.
Note that, if :ref:`Remember Me authentication function<SpringSecurityAuthenticationRememberMe>` is enabled, implementation class of \ ``LogoutHandler``\  is also configured to delete the token for Remember Me authentication.

|

Applying logout process
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Define a bean as below for application of logout process.

* Definition example of spring-security.xml

.. code-block:: xml

  <sec:http>
      <!-- omitted -->
      <sec:logout /> <!-- (1) -->
      <!-- omitted -->
  </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Logout process is enabled by defining \ ``<sec:logout>``\  tag.

.. note:: **Changes in Spring Security 4.0**

    Default values of following configuration changes from Spring Security 4.0 version.

    * logout-url 

.. tip:: **Delete Cookie**

   Although description is omitted in the guideline, note that \ ``delete-cookies``\  attribute exists in \ ``<sec:logout>``\  tag for deleting Cookie specified at the time of logout.
   However, it has been reported that a cookie is sometimes not deleted even after using this attribute.

   For details, refer following JIRA of Spring Security.

   * https://jira.spring.io/browse/SEC-2091

Default operation
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

In the default operation of Spring Security, if a request is sent to the path \ ``"/logout"``\ , logout process is carried out.
Logout process consists of "clearing authentication information of logged in user" and "cancelling a session".

Further,

* "Deleting token for CSRF measures" when CSRF measures are adopted
* "Deleting token for Remember Me authentication" when Remember Me authentication function is used

are also carried out.

.. _SpringSecurityAuthenticationLogoutForm:

* Implementation example of JSP for calling logout process

.. code-block:: jsp

    <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
    <%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags" %>
    <%-- omitted --%>
    <form:form action="${pageContext.request.contextPath}/logout" method="post"> <%-- (1) --%>
        <button>Logout</button>
    </form:form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Create a form for logout.
        | Further, token value of CSRF measure is sent in request parameter by using \ ``<form:form>``\ .
        | CSRF measures are explained in ":ref:`SpringSecurityCsrf`".

.. note:: **Sending CSRF tokens**

    If CSRF measures are enabled, tokens for CSRF measures must be sent by using POST method.

|

Response when logout is successful
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Security offers an interface \ ``LogoutSuccessHandler``\ and implementation class
as the components to control response when the logout is successfully completed.

.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table:: **Implementation class of AuthenticationFailureHandler**
    :header-rows: 1
    :widths: 35 65

    * - Implementation class
      - Description
    * - | \ ``SimpleUrlLogoutSuccessHandler``\
      - | Implementation class which redirects in the specified path (\ ``defaultTargetUrl``\ ).


Default operation
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

In default operation of Spring Security, user is redirected to a URL wherein a query parameter called \ ``"logout"``\ 
is assigned to the path for displaying login form.

As an example, when the path to display login form is \ ``"/login"``\ , the user is redirected to \ ``"/login?logout"``\ .


|

.. _SpringSecurityAuthenticationAccess:

Accessing authentication information
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Authentication information of the authenticated user is stored in the session during default implementation of Spring Security.
Authentication information stored in the session is stored in \ ``SecurityContextHolder``\  class by \ ``SecurityContextPersistenceFilter``\  class for each request and can be accessed from anywhere if it is in the same thread.

A method wherein \ ``UserDetails``\  are fetched from authentication information and the information retained in the fetched \ ``UserDetails``\  is accessed.

Access from Java
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

An audit log which records information like "When", "Who", "Which data" and "Type of access" is fetched in the general business application.
"Who" for fulfilling these requirements can be fetched from authentication information.

* Implementation example to access authentication information from Java

.. code-block:: java

    Authentication authentication =
            SecurityContextHolder.getContext().getAuthentication(); // (1)
    String userUuid = null;
    if (authentication.getPrincipal() instanceof AccountUserDetails) {
        AccountUserDetails userDetails =
                AccountUserDetails.class.cast(authentication.getPrincipal()); // (2)
        userUuid = userDetails.getAccount().getUserUuid(); // (3)
    }
    if (log.isInfoEnabled()) {
        log.info("type:Audit\tuserUuid:{}\tresource:{}\tmethod:{}",
                userUuid, httpRequest.getRequestURI(), httpRequest.getMethod());
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Fetch authentication information (\ ``Authentication``\  object) from \ ``SecurityContextHolder``\ .
    * - | (2)
      - | Call \ ``Authentication#getPrincipal()``\  method and fetch \ ``UserDetails``\  object.
        | When there is no authentication (anonymous user), note that a string indicating an anonymous user is returned.
    * - | (3)
      - | Fetch information required for processing from \ ``UserDetails``\ .
        | Here, a value which uniquely identifies the user (UUID) is fetched.

.. warning:: **Access and coupling of authentication information**

    In the default operation of Spring Security, since authentication information is stored in ThreadLocal variable, it can be accessed from anywhere if the thread is identical to a thread which receives request.
    Although this system is convenient, it results in direct dependency of the class which requires authentication information on \ ``SecurityContextHolder``\  class. Hence, care must be taken since it reduces  coupling between components when not used correctly.

    Spring Security also offers a system to maintain coupling between components by coordinating with Spring MVC function.
    How to coordinate with Spring MVC is explained in ":ref:`SpringSecurityAuthenticationIntegrationWithSpringMVC`".
    **This guideline recommends fetching authentication information by using coordination with Spring MVC.**

.. note::

    When a filter (FORM_LOGIN_FILTER) for authentication is to be customized,
    it is necessary to disable the following 2 \ ``SessionAuthenticationStrategy``\  classes, apart from specifying \ ``<sec:concurrency-control>``\  element.

    * | ``org.springframework.security.web.authentication.session.ConcurrentSessionControlAuthenticationStrategy``
      | A class to check number of sessions for each logged in user after successful authentication.

    * | ``org.springframework.security.web.authentication.session.RegisterSessionAuthenticationStrategy``
      | A class to register a session with successful authentication, in session management area.

    In version 1.0.x.RELEASE dependent Spring Security 3.1, \ ``org.springframework.security.web.authentication.session.ConcurrentSessionControlStrategy``\  class is provided; however,
    it is deprecated API from Spring Security 3.2 and it is abolished API from Spring Security 4.0.
    When upgrading version from Spring Security 3.1 to Spring Security 3.2 or later versions, changes need to be made so that it can be used with combination of following classes.

    * ``ConcurrentSessionControlAuthenticationStrategy`` (added in Spring Security 3.2)
    * ``RegisterSessionAuthenticationStrategy`` (added in Spring Security 3.2)
    * ``org.springframework.security.web.authentication.session.SessionFixationProtectionStrategy``

    For specific methods of definition,
    refer to sample code of `Spring Security Reference -Web Application Security (Concurrency Control)- <http://docs.spring.io/spring-security/site/docs/4.0.4.RELEASE/reference/htmlsingle/#concurrent-sessions>`_.

|

Access from JSP
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

User information of login user is displayed on the screen in general Web application.
User information of login user while meeting these requirements can be fetched from authentication information.

* Implementation example for accessing authentication information from JSP

.. code-block:: jsp

    <%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags" %>
    <%-- omitted --%>
    Welcome、
    <sec:authentication property="principal.account.lastName"/> <%-- (1) --%>
    San

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Use \ ``<sec:authentication>``\  tag offered by Spring Security and fetch authentication information (\ ``Authentication``\  object).
        | Specify path for the property to be accessed in \ ``property``\  attribute.
        | When a nested object is to be accessed, property name should be joined by \ ``"."``\ .

.. tip:: **How to display authentication information**

    Although the implementation example for displaying user information which contains authentication information is explained, it is possible to store value in any scope variable by combining \ ``var``\  attribute and \ ``scope``\  attribute.
    When the display contents are to be changed according to the login user status, user information can be stored in the variable and display can be changed by using JSTL tag library etc.

    In the example above, it can also be displayed as described below.
    Since \ ``scope``\  attribute is omitted in this example, \ ``page``\  scope is applied.

        .. code-block:: jsp

            <%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags" %>
            <%-- omitted --%>
            <sec:authentication var="principal" property="principal"/>
            <%-- omitted --%>
            Welcome
            ${f:h(principal.account.lastName)}
            San

|

.. _SpringSecurityAuthenticationIntegrationWithSpringMVC:

Coordination between authentication process and Spring MVC
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Security offers various components for coordinating with Spring MVC.
How to use the components for coordinating with authentication process is explained.

Accessing authentication information
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Security offers \ ``AuthenticationPrincipalArgumentResolver``\  class as a component which delivers authentication information (\ ``UserDetails``\ ) to Spring MVC controller method.
If \ ``AuthenticationPrincipalArgumentResolver``\  is used, \ ``UserDetails``\  interface or instance of its implementation class can be received as a method argument of controller resulting in enhanced coupling of components.

\ ``AuthenticationPrincipalArgumentResolver``\  must be applied to Spring MVC first for receiving Authentication information (\ ``UserDetails``\ ) as a controller argument.
Bean for applying \ ``AuthenticationPrincipalArgumentResolver``\  is as below.
\ Note that \ ``AuthenticationPrincipalArgumentResolver``\  is already cong=figured in the `blank project <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank>`_\ .

* Definition example of spring-mvc.xml

.. code-block:: xml

    <mvc:annotation-driven>
        <mvc:argument-resolvers>
            <!-- omitted -->
            <!-- (1) -->
            <bean class="org.springframework.security.web.method.annotation.AuthenticationPrincipalArgumentResolver" />
            <!-- omitted -->
        </mvc:argument-resolvers>
  </mvc:annotation-driven>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Apply \ ``AuthenticationPrincipalArgumentResolver``\  in Spring MVC as an implementation class of \ ``HandlerMethodArgumentResolver``\ .

|

Following method is created while receiving authentication information (\ ``UserDetails``\ ) by controller method.

* How to create a method which receives authentication information (UserDetails)

.. code-block:: java

    @RequestMapping("account")
    @Controller
    public class AccountController {

        public String view(
                @AuthenticationPrincipal AccountUserDetails userDetails, // (1)
                Model model) {
            model.addAttribute(userDetails.getAccount());
            return "profile";
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Declare an argument to receive authentication information (\ ``UserDetails``\ ) and specify \ ``@org.springframework.security.core.annotation.AuthenticationPrincipal``\  as an argument annotation.
        | \ ``AuthenticationPrincipalArgumentResolver``\  configures authentication information (\ ``UserDetails``\ ) in the argument assigned by \ ``@AuthenticationPrincipal``\ .

|

.. _SpringSecurityAuthenticationHowToExtend:

How to extend
--------------------------------------------------------------------------------

The section describes customization points and extension methods offered by Spring Security.

Spring Security offers a lot of customization points. Since it is not possible to introduce all the customization points here, we will focus on a few typical customization points here.

|

.. _SpringSecurityAuthenticationCustomizingForm:

Customizing form authentication
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Customization points of form authentication process are explained.

Changing authentication path
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

In Spring Security, the path for carrying out authentication process is "\ ``"/login"``\ " by default, however
it can be changed by defining a bean as below.

* Definition example of spring-security.xml

.. code-block:: xml

  <sec:http>
    <sec:form-login login-processing-url="/authentication" /> <!-- (1) --> 
    <!-- omitted -->
  </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify a path for carrying out authentication process in \ ``login-processing-url``\  attribute.

.. note::

    When path of authentication process is changed, request destination of :ref:`Login form <SpringSecurityAuthenticationLoginForm>` must be changed as well.

|

Change request parameter name which sends credentials
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

In Spring Security, request parameters for sending credentials (user name and password) are "\ ``username``\ " and "\ ``password``\ " by default, however,
these can be changed by defining a bean as shown below.

* Definition example for spring-security.xml

.. code-block:: xml

  <sec:http>
      <sec:form-login
          username-parameter="uid"
          password-parameter="pwd" /> <!-- (1) (2) -->
      <!-- omitted -->
  </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify a request parameter name for user name in \ ``username-parameter``\  attribute.
    * - | (2)
      - | Specify a request parameter name for password in \ ``password-parameter``\  attribute.

.. note::

    When request parameter name is changed, field name in :ref:`Login form <SpringSecurityAuthenticationLoginForm>` must also be changed.

|

Customising response when authentication is successful
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Customization points of response when authentication is successful are explained.

Changing default transition destination
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Transition destination (default URL) after displaying login form on its own and by carrying out authentication process
is the root path of Web application (\ ``"/"``\ ), however it can be changed by defining a bean as given below.

* Definition example of spring-security.xml

.. code-block:: xml

  <sec:http>
      <sec:form-login default-target-url="/menu" /> <!-- (1) -->
  </sec:http>

.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify default path for transition in \ ``default-target-url``\  attribute when the authentication is successful.

|

Fixing transition destination
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

In default operation of Spring Security, when a request for a page for which authentication is required is received during unauthenticated stage, the request thus received is temporarily stored in HTTP session and then transition is done to authentication page.
Although request is restored and redirected when the authentication is successful, transition to the same screen can be assured by defining a bean as below.

* Definition example of spring-security.xml

.. code-block:: xml

  <sec:http>
      <sec:form-login
          default-target-url="/menu"
          always-use-default-target="true" /> <!-- (1) -->
  </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify \ ``true``\ in \ ``always-use-default-target``\  attribute.

|

Applying AuthenticationSuccessHandler
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When the requirements are not met only by using the system wherein default operations provided by Spring Security are customised,
implementation class of \ ``AuthenticationSuccessHandler``\  interface can be applied directly by defining a bean as given below.

* Definition example of spring-security.xml

.. code-block:: xml

  <bean id="authenticationSuccessHandler" class="com.example.app.security.handler.MyAuthenticationSuccessHandler"> <!-- (1) -->

  <sec:http>
      <sec:form-login authentication-success-handler-ref="authenticationSuccessHandler" /> <!-- (2) -->
  </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Define a bean for implementation class of \ ``AuthenticationSuccessHandler``\  interface.
    * - | (2)
      - | Specify \ ``authenticationSuccessHandler``\  defined in ``authentication-success-handler-ref``\  attribute.

.. warning:: **Responsibility of AuthenticationSuccessHandler**

    \ ``AuthenticationSuccessHandler``\  is an interface which performs processes for Web layer at the time of successful authentication (mainly processes related to screen transition).
    Therefore, a process dependent on business rule (business logic) like clearing number of authentication failures should not be called through implementation class of this interface.

    ":ref:`SpringSecurityAuthenticationEvent`" system introduced in earlier section should be used for calling the processes that are dependent on business rules.

|

.. _SpringSecurityAuthenticationCustomizingScreenFlowOnFailure:

Customising response at the time of authentication failure
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Customization points for the response at the time of authentication failure are explained.

Changing transition destination
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

In default operation of Spring Security, the user is redirected to a URL wherein a query parameter \ ``"error"``\ is assigned to path for displaying login form, however
it can be changed by defining a bean as given below.

* Definition example of spring-security.xml

.. code-block:: xml

  <sec:http>
      <sec:form-login authentication-failure-url="/loginFailure" /> <!-- (1) -->
  </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr.No.
      - Description
    * - |  (1)
      - | Specify a path for transition in \ ``authentication-failure-url``\  attribute at the time of authentication failure.

|

Applying AuthenticationFailureHandler
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When the requirements are not met only by using the system wherein default operations provided by Spring Security are customised,
implementation class of \ ``AuthenticationFailureHandler``\  interface can be applied directly by defining a bean as below.

* Definition example of spring-security.xml

.. code-block:: xml

   <!-- (1) -->
  <bean id="authenticationFailureHandler"
      class="org.springframework.security.web.authentication.ExceptionMappingAuthenticationFailureHandler" />
      <property name="defaultFailureUrl" value="/login/systemError" /> <!-- (2) -->
      <property name="exceptionMappings"> <!-- (3) -->
          <props>
              <prop key="org.springframework.security.authentication.BadCredentialsException"> <!-- (4) -->
                  /login/badCredentials
              </prop>
              <prop key="org.springframework.security.core.userdetails.UsernameNotFoundException"> <!-- (5) -->
                  /login/usernameNotFound
              </prop>
              <prop key="org.springframework.security.authentication.DisabledException"> <!-- (6) -->
                  /login/disabled
              </prop>
              <!-- omitted -->
          </props>
      </property>
  </bean>

  <sec:http>
      <sec:form-login authentication-failure-handler-ref="authenticationFailureHandler" /> <!-- (7) -->
  </sec:http>


.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 80
    :class: longtable

    * - | Sr. No.
      - | Description
    * - | (1)
      - | Define a bean for implementation class of \ ``AuthenticationFailureHandler``\  interface.
    * - | (2)
      - | Specify default transition destination URL in \ ``defaultFailureUrl``\  attribute.
        | When the exceptions that do not match the definitions given in (4)~(6) below occur, move to transition destination of the configuration.
    * - | (3)
      - | Set implementation class of \ ``org.springframework.security.authentication.AuthenticationServiceException``\  handled in the \ ``exceptionMappings``\  property and transition destination at the time exception occurrence in \ ``Map``\  format.
        | Set implementation class of \ ``org.springframework.security.authentication.AuthenticationServiceException``\  in key and set transition destination URL in value.
    * - | (4)
      - | \ ``BadCredentialsException``\ 
        | It is thrown at the time of authentication error due to password verification failure.
    * - | (5)
      - | \ ``UsernameNotFoundException``\ 
        | It is thrown at the time of authentication error due to invalid user ID (non-existent user ID).
        | When ``org.springframework.security.authentication.dao.AbstractUserDetailsAuthenticationProvider``\  specifies inherited class in
        | the authentication provider, the exception changes to \ ``BadCredentialsException``\  if ``hideUserNotFoundExceptions``\  property is not changed to \ ``false``\ .
    * - | (6)
      - | \  ``DisabledException``\
        | It is thrown at the time of authentication error due to invalid user ID.
    * - | (7)
      - | Set \ ``authenticationFailureHandler``\  in \ ``authentication-failure-handler-ref``\  attribute.

.. raw:: latex

   \newpage

.. note:: **Control during occurrence of exception**

    When an exception defined in \ ``exceptionMappings``\  property occur, the user is redirected to transition destination mapped in the exception, however,
    since exception object thus occurred is not stored in the session scope, error message generated by Spring Security cannot be displayed on the screen.

    Therefore, error message displayed in the transition destination screen must be generated by the process for redirect (Controller or View process).

    Further, it must be added that since the processes referring properties below are not called, operation does not undergo any change even after changing the setup value.

    * ``useForward``
    * ``allowSessionCreation``

|

Customising logout process
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Customization points for logout process are explained.

Changing logout path
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

In Spring Security, default path which carries out logout process is "\ ``"/logout"``\ ", however
it can be changed by defining a bean as given below.

* Definition example of spring-security.xml

.. code-block:: xml

  <sec:http>
      <!-- omitted -->
      <sec:logout logout-url="/auth/logout" /> <!-- (1) -->
      <!-- omitted -->
  </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Set \ ``logout-url``\  attribute and specify path to carry out logout process.

.. note::

    When logout path is changed, request destination of :ref:`logout form <SpringSecurityAuthenticationLogoutForm>` must be changed as well.

.. tip:: **Behaviour during occurrence of system error**
    When system error occurs, discontinuation of operations is likely.
    If the operation is not to be continued after occurrence of system error, adopting following measures is recommended.
    
      * Clear session information when the system error occurs.
      * Clear authentication information when the system error occurs
    
    An example wherein authentication information at the time of system exception occurrence is cleared using an exception handling function of common library is explained.
    For details of exception handling function, refer "\ :doc:`../ArchitectureInDetail/WebApplicationDetail/ExceptionHandling`\ ".

      .. code-block:: java

        // (1)
        public class LogoutSystemExceptionResolver extends SystemExceptionResolver {
            // (2)
            @Override
            protected ModelAndView doResolveException(HttpServletRequest request,
                    HttpServletResponse response, java.lang.Object handler,
                    java.lang.Exception ex) {

                // Carry out SystemExceptionResolver
                ModelAndView resulut = super.doResolveException(request, response,
                        handler, ex);

                // Clear authentication information (2)
                SecurityContextHolder.clearContext();

                return resulut;
            }
        }

      .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
      .. list-table::
          :header-rows: 1
          :widths: 10 90
      
          * - Sr. No.
            - Description
          * - | (1)
            - | Extend \ ``org.terasoluna.gfw.web.exception.SystemExceptionResolver.SystemExceptionResolver``\ .
          * - | (2)
            - | \ Clear authentication information.

    Further, the same requirement can also be met by clearing the session, besides using the method that clears authentication information.
    Implement according to the project requirement.

|

.. _SpringSecurityLogoutCustomizingScreenFlowOnSuccess:

Customising response when logout is successful
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Customization points for the response when logout process is successful is explained.

Changing transition destination
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* Definition example of spring-security.xml

.. code-block:: xml

  <sec:http>
    <!-- omitted -->
    <sec:logout logout-success-url="/logoutSuccess" /> <!-- (1) -->
    <!-- omitted -->
  </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Set \ ``logout-success-url``\  attribute and specify path for the transition when the logout is successful.

|

Applying LogoutSuccessHandler
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* Definition example of spring-security.xml

.. code-block:: xml
  
  <!-- (1) -->
  <bean id="logoutSuccessHandler" class="com.example.app.security.handler.MyLogoutSuccessHandler" /> 

  <sec:http>
      <!-- omitted -->
      <sec:logout success-handler-ref="logoutSuccessHandler" /> <!-- (2) -->
      <!-- omitted -->
  </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Define a bean for implementation class of \ ``LogoutSuccessHandler``\  interface.
    * - | (2)
      - | Set \ ``LogoutSuccessHandler``\ in ``success-handler-ref``\  attribute.

|

.. _SpringSecurityAuthenticationCustomizingMessage:

Customising error message
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When authentication fails, error message provided by Spring Security is displayed, however,
the error message can be changed.

For details of how to change the message, refer \ :doc:`../ArchitectureInDetail/WebApplicationDetail/MessageManagement`\ .

Message during system error
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When an unexpected error occurs (system error etc) during authentication process, \ ``InternalAuthenticationServiceException``\  exception is thrown.
Since cause exception message is set in the message retained by \ ``InternalAuthenticationServiceException``\ , it should not be displayed as it is on the screen.

For example, when a DB access error occurs while fetching user information from database, an exception message retained by \ ``SQLException``\  is displayed on the screen.
The measures like handling \ ``InternalAuthenticationServiceException``\  by using \ ``ExceptionMappingAuthenticationFailureHandler``\  and
transiting to the path for notifying occurrence of system error are necessary for not displaying exception message of system error on the screen.

* Definition example of spring-security.xml

.. code-block:: xml

    <bean id="authenticationFailureHandler"
        class="org.springframework.security.web.authentication.ExceptionMappingAuthenticationFailureHandler">
        <property name="defaultFailureUrl" value="/login?error" />
        <property name="exceptionMappings">
            <props>
                <prop key="org.springframework.security.authentication.InternalAuthenticationServiceException">
                    /login?systemError
                </prop>
                <!-- omitted -->
            </props>
        </property>
    </bean>

  <sec:http>
      <sec:form-login authentication-failure-handler-ref="authenticationFailureHandler" />
  </sec:http>

|

Query parameter (\ ``systemError``\) is used for identifying occurrence of system error and transition is made to login form.
When \ ``systemError``\  is specified in the query parameter, in the login form specified in the transition destination, a fixed error message is displayed instead of an authentication exception message.


* Implementation example of login form

.. code-block:: jsp

    <c:choose>
        <c:when test="${param.containsKey('error')}">
            <span style="color: red;">
                <c:out value="${SPRING_SECURITY_LAST_EXCEPTION.message}"/>
            </span>
        </c:when>
        <c:when test="${param.containsKey('systemError')}">
            <span style="color: red;">
                System Error occurred.
            </span>
        </c:when>
    </c:choose>

.. note::

    An implementation example for transiting to login form is introduced, however it is also possible to move to system error screen.

|

.. _SpringSecurityAuthenticationBeanValidation:

Input check at the time of authentication
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A check is carried out in advance for obvious input error on the authentication page during reduction of load on DB server etc.
In such a case, input check using Bean validation can also be carried out.

Input check using Bean Validation
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

An example of input check using Bean Validation is explained below.
For details of Bean Validation, refer \ :doc:`../ArchitectureInDetail/WebApplicationDetail/Validation`\ .

* Implementation example of form class

.. code-block:: java

    public class LoginForm implements Serializable {

        // omitted
        @NotEmpty // (1)
        private String username;

        @NotEmpty // (1)
        private String password;
        // omitted

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | In this example, \ ``username``\ and \ ``password``\  are mandatory respectively.


* Implementation example of controller class

.. code-block:: java

    @ModelAttribute
    public LoginForm setupForm() { // (1)
        return new LoginForm();
    }

    @RequestMapping(value = "login")
    public String login(@Validated LoginForm form, BindingResult result) {
        // omitted
        if (result.hasErrors()) {
            // omitted
        }
        return "forward:/authenticate"; // (2)
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Initialise \ ``LoginForm``\ .
    * - | (2)
      - | **Forward** to path specified in \ ``login-processing-url``\  attribute of \ ``<sec:form-login>``\ element using "forward".
        | For setup related to authentication, refer \ :ref:`SpringSecurityAuthenticationCustomizingForm`\ .

In addition, authentication path is added to Spring Security servlet filter for Spring Security processing even during the transition using Forward.

* Setup example of web.xml

.. code-block:: xml

    <filter>
        <filter-name>springSecurityFilterChain</filter-name>
        <filter-class>
            org.springframework.web.filter.DelegatingFilterProxy
        </filter-class>
    </filter>
    <filter-mapping>
        <filter-name>springSecurityFilterChain</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    <!-- (1) -->
    <filter-mapping>
        <filter-name>springSecurityFilterChain</filter-name>
        <url-pattern>/authenticate</url-pattern>
        <dispatcher>FORWARD</dispatcher>
    </filter-mapping>    

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify pattern for authentication by using Forward.
        | Here \ ``"/authenticate"``\  is specified as an authentication path.

|

Expansion of authentication process
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In case of authentication requirements which cannot be handled by \ `authentication provider <http://docs.spring.io/spring-security/site/docs/4.0.4.RELEASE/apidocs/org/springframework/security/authentication/AuthenticationProvider.html>`_\  offered by Spring Security,
a class which implements \ ``org.springframework.security.authentication.AuthenticationProvider``\  interface must be created.

Here, an expansion example for performing DB authentication by using 3 parameters - user name, password and \ **Company identifier (unique authentication parameter)**\ is shown below.

.. figure:: ./images_Authentication/Authentication_HowToExtends_LoginForm.png
   :alt: Authentication_HowToExtends_LoginForm
   :width: 50%

A class shown below must be created for implementing above requirements.

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | An implementation class of \ ``org.springframework.security.core.Authentication``\  interface which retains user name, password and company identifier.
        | It is created by inheriting \ ``org.springframework.security.authentication.UsernamePasswordAuthenticationToken``\  class.
    * - | (2)
      - | An implementation class of \ ``org.springframework.security.authentication.AuthenticationProvider``\  which performs DB authentication by using user name, password and company identifier.
        | It is created by inheriting \ ``org.springframework.security.authentication.dao.DaoAuthenticationProvider``\  class.
    * - | (3)
      - | An Authentication Filter class for generating \ ``Authentication``\  which is passed to \ ``AuthenticationManager``\ (\ ``AuthenticationProvider``\ ) by fetching user name, password and company identifier from request parameters.
        | It is created by inheriting \ ``org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter``\  class.

.. note::

    Since a unique parameter is added as an parameter for authentication in this example,
    implementation class of \ ``Authentication``\ interface and Authentication Filter class for generating \ ``Authentication``\  must be expanded.

    When the authentication is to be performed only by user name and password, authentication process can be expanded only by creating implementation class of \ ``AuthenticationProvider``\  interface
    

|

Creating an implementation class of Authentication interface
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``UsernamePasswordAuthenticationToken``\  class is inherited and a class that retains a company identifier (unique authentication parameter) besides user name and password is created.

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
     - | Create a field that retains a company identifier.
   * - | (2)
     - | Create a constructor to be used while creating an instance which retains information prior to authentication (information specified by request parameter).
   * - | (3)
     - | Create a constructor to be used while creating an instance which retains authenticated information.
       | Authenticated state is reached by passing authorization information to the argument of parent class constructor.

|

Creating an implementation class of AuthenticationProvider interface
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``DaoAuthenticationProvider``\  class is inherited and a class which performs DB authentication by using user name, password and company identifier is created.

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
            String companyId = ((SampleUserDetails) userDetails).getAccount().getCompanyId();
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
     - | Call parent class method and implement check process provided by Spring Security.
       | This process also includes password authentication process.
   * - | (2)
     - | When password authentication is successful, check validity of company identifier (unique authentication parameter).
       | In the example above, it is checked whether the requested company identifier and the company identifier retained in the table are identical.
   * - | (3)
     - | When password authentication and unique authentication process are successful, \ ``CompanyIdUsernamePasswordAuthenticationToken``\ is created in an authenticated state and then returned.
   * - | (4)
     - | When castable \ ``Authentication``\  is specified in \ ``CompanyIdUsernamePasswordAuthenticationToken``\ , perform authentication process by using this class.

.. note::

    User existence check, user status check (checks for invalid user, locked user, expired user etc) are carried out as
    processes of parent class before calling \ ``additionalAuthenticationChecks``\  method.

|

.. _authentication_custom_usernamepasswordauthenticationfilter:

Creating Authentication Filter
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``UsernamePasswordAuthenticationFilter``\  class is inherited and
an Authentication Filter class is created for passing authentication information (user name, password and company identifier) to \ ``AuthenticationProvider``\ .

\ ``attemptAuthentication``\  method implementation is customised by copying method of \ ``UsernamePasswordAuthenticationFilter``\  class.

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
            return request.getParameter("companyId");
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Generate an instance of \ ``CompanyIdUsernamePasswordAuthenticationToken``\  from the authentication information fetched from request parameter (user name, password, company identifier).
   * - | (2)
     - | Specify authentication information specified by request parameter (\ ``CompanyIdUsernamePasswordAuthenticationToken``\  instance) and call \ ``authenticate``\  method of \ ``org.springframework.security.authentication.AuthenticationManager``\ .
       | 
       | If \ ``AuthenticationManager``\  method is called, \ ``AuthenticationProvider``\  authentication process gets called.
   * - | (3)
     - | Company identifier is fetched by using request parameter \ ``"companyId"``\ .

|

Modifying login form
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Company identifier is added for login form (JSP) created by \ :ref:`SpringSecurityAuthenticationLoginForm`\ .

.. code-block:: jsp

    <form:form action="${pageContext.request.contextPath}/login" method="post">
        <!-- omitted -->
            <tr>
                <td><label for="username">User Name</label></td>
                <td><input type="text" id="username" name="username"></td>
            </tr>
            <tr>
                <td><label for="companyId">Company Id</label></td>
                <td><input type="text" id="companyId" name="companyId"></td> <!-- (1) -->
            </tr>
            <tr>
                <td><label for="password">Password</label></td>
                <td><input type="password" id="password" name="password"></td>
            </tr>
        <!-- omitted -->
    </form:form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify \ ``"companyId"``\  in input field name of company identifier.

|

Applying expanded authentication process
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

DB authentication process which use user name, password and company identifier (unique authentication parameter) are applied in Spring Security.

* Definition example of spring-security.xml

.. code-block:: xml

    <!-- omitted -->

    <!-- (1) -->
    <sec:http
        entry-point-ref="loginUrlAuthenticationEntryPoint">

        <!-- omitted -->

        <!-- (2) -->
        <sec:custom-filter
            position="FORM_LOGIN_FILTER" ref="companyIdUsernamePasswordAuthenticationFilter" />

        <!-- omitted -->

        <sec:csrf token-repository-ref="csrfTokenRepository" />

        <sec:logout
            logout-url="/logout"
            logout-success-url="/login" />

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
            <bean class="org.springframework.security.web.util.matcher.AntPathRequestMatcher">
                <constructor-arg index="0" value="/authentication" />
                <constructor-arg index="1" value="POST" />
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
    :class: longtable

    * - Sr. No.
      - Description
    * - | (1)
      - | When \ ``"FORM_LOGIN_FILTER"``\  is to be changed by using \ ``<sec:custom-filter>``\  tag of (2), following settings must be applied to \ ``<sec:http>``\  tag attribute.

        * Since auto-configuration cannot be used, \ ``auto-config="false"``\  is specified or \ ``auto-config``\  attribute is deleted.
        * Since \ ``<sec:form-login>``\  tag cannot be used, \ ``AuthenticationEntryPoint``\  is specified explicitly by using \ ``entry-point-ref``\  attribute.

    * - | (2)
      - | Use \ ``<sec:custom-filter>``\  tag and change \ ``"FORM_LOGIN_FILTER"``\ .
        | 
        | Specify \ ``"FORM_LOGIN_FILTER"``\  in \ ``position``\  attribute of \ ``<sec:custom-filter>``\  tag and specify a bean for Authentication Filter expanded in \ ``ref``\  attribute.
    * - | (3)
      - | Specify a bean of \ ``AuthenticationEntryPoint``\ used in \ ``entry-point-ref``\  attribute of \ ``<sec:http>``\  tag.
        | 
        | Here, a bean of \ ``org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint``\  class used while specifying \ ``<sec:form-login>``\  tag is specified.
    * - | (4)
      - | Define a bean of Authentication Filter class used as \ ``"FORM_LOGIN_FILTER"``\ .
        | 
        | Here, a bean of expanded Authentication Filter class (\ ``CompanyIdUsernamePasswordAuthenticationFilter``\ ) is defined.
    * - | (5)
      - | Specify \ ``RequestMatcher``\  instance for detecting a request performing authentication process, in \ ``requiresAuthenticationRequestMatcher``\  property.
        | 
        | It is configured to perform authentication process when a request is raised in the \ ``"/authentication"``\  path.
        | This is similar to specifying \ ``"/authentication"``\  in \ ``login-processing-url``\  attribute of \ ``<sec:form-login>``\  tag.
    * - | (6)
      - | Specify a value set in \ ``alias``\  attribute of \ ``<sec:authentication-manager>``\  tag, in \ ``authenticationManager``\  property.
        | 
        | If \ ``alias``\  attribute of \ ``<sec:authentication-manager>``\  tag is specified,
        | a bean of \ ``AuthenticationManager``\  generated by Spring Security can be used for DI of another bean.
    * - | (6')
      - | Set expanded \ ``AuthenticationProvider``\ (\ ``CompanyIdUsernamePasswordAuthenticationProvider``\ ) for \ ``AuthenticationManager``\  generated by Spring Security.
    * - | (7)
      - | Specify a bean of component (\ ``SessionAuthenticationStrategy``\ ) that controls handling of a session during successful authentication in \ ``sessionAuthenticationStrategy``\  property.
        | 
    * - | (7')
      - | Define a bean of component (\ ``SessionAuthenticationStrategy``\ ) that controls handling of a session during successful authentication.
        | 
        | Here,
         
        * A component to recreate CSRF token (\ ``CsrfAuthenticationStrategy``\ )
        * A component to generate a new session for preventing session fixation attack (\ ``SessionFixationProtectionStrategy``\ )
        
        | offered by Spring Security are enabled.
    * - | (8)
      - | Specify a handler class in \ ``authenticationFailureHandler``\  property which is called at the time of authentication failure.
    * - | (9)
      - | Specify a handler class in \ ``authenticationSuccessHandler``\  property which is called at the time of successful authentication.

.. raw:: latex

   \newpage

.. note:: **Regarding auto-config**

    When Basic authentication process and logout process are to be enabled by specifying \ ``auto-config="false"``\  or by omitting specification, \ ``<sec:http-basic>``\  tag and \ ``<sec:logout>``\  tag must be defined explicitly.

|

.. _AuthenticationHowToExtendUsingDeprecatedPasswordEncoder:

Using PasswordEncoder of deprecated package
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Based on security requirements, it is likely that the implementation is not possible using the class which implements \ ``PasswordEncoder``\  described earlier.
In particular, when it is necessary to follow hashing requirements used in the existing account information, it may not meet the requirements of \ ``PasswordEncoder``\  described earlier.

Basically, existing hashing requirements are as given below.

* Algorithm is SHA-512.
* Stretching frequency is 1000.
* Salt is stored in the account table column and it must be passed from outside of \ ``PasswordEncoder``\ .

In such a case, implementation class of \ ``org.springframework.security.authentication.encoding.PasswordEncoder``\  interface must be used to meet requirements
instead of implementation class of \ ``org.springframework.security.crypto.password.PasswordEncoder``\  interface.

.. warning::

    In Spring Security 3.1.4 and earlier versions, a class that implements \ ``org.springframework.security.authentication.encoding.PasswordEncoder``\
    was used in the hashing, however, it is deprecated in the 3.1.4 and subsequent versions.

|

Using ShaPasswordEncoder
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

This guideline explains the use of \ ``PasswordEncoder``\ of deprecated package using an example of \ ``ShaPasswordEncoder``\ .

When the hashing requirements are as below, requirements can be met by using \ ``ShaPasswordEncoder``\ .

* Algorithm is SHA-512
* Stretching frequency is 1000

|

First, define a bean for \ ``ShaPasswordEncoder``\ .

* Definition example of applicationContext.xml

.. code-block:: xml
  
    <bean id ="passwordEncoder"
        class="org.springframework.security.authentication.encoding.ShaPasswordEncoder"> <!-- (1) -->
        <constructor-arg value="512" /> <!-- (2) -->
        <property name="iterations" value="1000" /> <!-- (3) -->
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
  
    * - Sr. No.
      - Description
    * - | (1)
      - | Define a bean for \ ``org.springframework.security.authentication.encoding.ShaPasswordEncoder``\ .
    * - | (2)
      - | Specify type of SHA algorithm.
        | The values that can be specified are "\ ``1``\ , \ ``256``\ , \ ``384``\ , \ ``512``\ ".
        | Value is "\ ``1``\ " when it is not specified.
    * - | (3)
      - | Specify stretching frequency at the time of hashing.
        | Value is 1 when it is not specified.

|

Next, \ ``ShaPasswordEncoder``\  is applied to authentication process (\ ``DaoAuthenticationProvider``\ ) of Spring Security.

* Definition example of spring-security.xml

.. code-block:: xml
  
    <bean id="authenticationProvider"
        class="org.springframework.security.authentication.dao.DaoAuthenticationProvider">
        <!-- omitted -->
        <property name="saltSource" ref="saltSource" /> <!-- (1) -->
        <property name="userDetailsService" ref="userDetailsService" />
        <property name="passwordEncoder" ref="passwordEncoder" /> <!-- (2) -->
    </bean>
  
    <bean id="saltSource"
        class="org.springframework.security.authentication.dao.ReflectionSaltSource"> <!-- (3) -->
        <property name="userPropertyToUse" value="username" /> <!-- (4) -->
    </bean>
  
.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
  
    * - Sr. No.
      - Description
    * - | (1)
      - | Specify a bean for implementation class of \ ``org.springframework.security.authentication.dao.SaltSource``\  in \ ``saltSource``\  property.
        | \ ``SaltSource``\  is an interface that fetches salt from \ ``UserDetails``\ .
    * - | (2)
      - | Specify a bean for implementation class of \ ``org.springframework.security.authentication.encoding.PasswordEncoder``\  interface in \ ``passwordEncoder``\  property.
        | In the example above, a bean for \ ``ShaPasswordEncoder``\  is specified.
    * - | (3)
      - | Define a bean for \ ``SaltSource``\ .
        | In the example above, a class (\ ``ReflectionSaltSource``\ ) that fetches salt from \ ``UserDetails``\  property using reflection, is used.
    * - | (4)
      - | Specify \ ``UserDetails``\  property where salt is stored.
        | In the example above, a value of \ ``username``\  property of \ ``UserDetails``\  is used as a salt.

|

When deprecated \ ``PasswordEncoder``\  is to be used in the process of application, \ ``PasswordEncoder``\  is used after injection.

* Implementation example of Java class

.. code-block:: java
  
    @Inject
    PasswordEncoder passwordEncoder;
  
    public String register(Customer customer, String rawPassword, String userSalt) {
        // omitted
        String password = passwordEncoder.encodePassword(rawPassword, userSalt); // (1)
        customer.setPassword(password);
        // omitted
    }
  
    public boolean matches(Customer customer, String rawPassword, String userSalt) {
        return passwordEncoder.isPasswordValid(customer.getPassword(), rawPassword, userSalt); // (2)
    }
  
.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
  
    * - Sr. No.
      - Description
    * - | (1)
      - | When password is to be hashed, use \ ``encodePassword``\  method.
        | Specify password, salt string in the method argument, in a sequence.
    * - | (2)
      - | When the password is to be verified, use \ ``isPasswordValid``\  method.
        | Specify hashed password, plain text password and salt string in the method argument, in a sequence.

|

Appendix
--------------------------------------------------------------------------------

.. _spring-security-authentication-mvc:

Receive a request by Spring MVC and display login form
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
A method wherein a request is received by Spring MVC and login form is displayed is explained.

* Definition example of spring-mvc.xml

Definition example of Controller which displays login form.

.. code-block:: java

    @Controller
    @RequestMapping("/login")
    public class LoginController { // (1)

        @RequestMapping
        public String index() {
            return "login";
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Return "login" as view name. src/main/webapp/WEB-INF/views/login.jsp is output by \ ``InternalResourceViewResolver``\

As per this example, it is also possible to substitute by using \ ``<mvc:view-controller>``\  in case of a controller with only one method which simply returns only the view name.

* Definition example of Controller using \ ``<mvc:view-controller>``\ .

.. code-block:: xml

    <mvc:view-controller path="/login" view-name="login" /><!-- (1) -->

|

.. _SpringSecurityAuthenticationRememberMe:

Using Remember Me authentication
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

"\ `Remember Me authentication <http://docs.spring.io/spring-security/site/docs/4.0.4.RELEASE/reference/htmlsingle/#remember-me>`_\ " is one of the functions to enhance
the user experience who frequently access Websites, and retains logged in state of a user for a little longer than normal lifecycle.
If this function is used, the user can login again without entering user name and password by using a Token for Remember Me authentication retained by a Cookie,
even when the browser is closed or session has timed-out.
Note that, this function is enabled only when the user has authorised retaining the logged in state.

Spring Security supports "Remember Me authentication of `Hash-Based Token <http://docs.spring.io/spring-security/site/docs/4.0.4.RELEASE/reference/htmlsingle/#remember-me-hash-token>`_ method and `Persistent Token <http://docs.spring.io/spring-security/site/docs/4.0.4.RELEASE/reference/htmlsingle/#remember-me-persistent-token>`_ method.
Hash-Based Token method is used as a default.

|

When Remember Me authentication is to be used, \ ``<sec:remember-me>``\  tag is added.

* Definition example of spring-security.xml

.. code-block:: xml

    <sec:http>
        <!-- omitted -->
        <sec:remember-me key="terasoluna-tourreservation-km/ylnHv"
            token-validity-seconds="#{30 * 24 * 60 * 60}" />  <!-- (1) (2) -->
        <!-- omitted -->
    </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify a key value in \ ``key``\  attribute for identifying an application that generates a Token for Remember Me authentication.
        | When key value is not specified, a unique value is generated every time an application starts.
        | Note that, when a key value retained by Hash-Based Token and a key value retained by server vary, it is handled as an invalid Token.
        | In other words, when a Hash-Based Token generated before restarting an application is to be handled as a valid Token, \ ``key``\  attribute must be specified.
    * - | (2)
      - | Specify validity period of Token for Remember Me authentication in seconds, in the \ ``token-validity-seconds``\  attribute.
        | When it is not specified, the validity period is 14 days by default.
        | In the example above, 30 days has been set as a validity period.

For attributes other than above, refer \ `Spring Security Reference -The Security Namespace (<remember-me>) - <http://docs.spring.io/spring-security/site/docs/4.0.4.RELEASE/reference/htmlsingle/#nsa-remember-me>`_\ .

.. note:: **Changes in Spring Security 4.0**

    Default value of following settings is changed from Spring Security 4.0 version

    * remember-me-parameter
    * remember-me-cookie

|

A flag (checkbox field) to specify use of "Remember Me authentication" function  is provided in the login form.

* Implementation example of JSP of login form

.. code-block:: jsp

    <form:form action="${pageContext.request.contextPath}/login" method="post">
            <!-- omitted -->
            <tr>
                <td><label for="remember-me">Remember Me : </label></td>
                <td><input name="remember-me" id="remember-me" type="checkbox" checked="checked" value="true"></td> <!-- (1) -->
            </tr>
            <!-- omitted -->
    </form:form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Add a flag (checkbox item) to specify whether "Remember Me authentication" function is used, and specify \ ``remember-me``\  - a default value of \ ``remember-me-parameter``\  in field name (request parameter name).
        | Set \ ``true``\  in \ ``value``\  attribute of checkbox.
        | If authentication process is carried out after ticking the checkbox, "Remember Me authentication" function is applied for subsequent requests.

.. tip:: **About setup value of value attribute**

   Description of setting \ ``true``\  in \ ``value``\  attribute is given in \ `rememberMeRequested - JavaDoc <http://docs.spring.io/autorepo/docs/spring-security/4.1.4.RELEASE/apidocs/org/springframework/security/web/authentication/rememberme/AbstractRememberMeServices.html#rememberMeRequested-javax.servlet.http.HttpServletRequest-java.lang.String->`_\ ,
   however \ ``on``\ , \ ``yes``\  and \ ``1``\  can also be set in the implementation.

.. raw:: latex

   \newpage

