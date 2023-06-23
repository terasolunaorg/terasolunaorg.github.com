Implementation Example of Typical Security Requirements
********************************************************************************

.. only:: html

 .. contents:: Table of Contents
    :depth: 3
    :local:

Introduction
================================================================================

Topics described in this chapter
--------------------------------------------------------------------------------

* Example of implementation method to meet the typical security requirements using TERASOLUNA Server Framework for Java (5.x)
* Implementation method and source code description using the sample application shown in :ref:`app-description-sec`
  
.. warning::
    * The implementation methods described in this chapter is just an example, and implementation must be carried out in the actual development as per the requirement.
    * Since it does not guarantee an exhaustive implementation of security measures, additional measures should be considered if necessary

Target Readers
--------------------------------------------------------------------------------

* Must understand the contents of :doc:`../ImplementationAtEachLayer/index`
* Must understand the contents of :doc:`./SpringSecurity`, :doc:`./Authentication`, :doc:`./Authorization`
* Must complete implementation of :doc:`./Tutorial`

.. _app-description-sec:

Description of application
================================================================================

| This section explains about a specific implementation method for security measures by using a sample application that meets the typical security requirements.
| The list of security requirements that describe the implementation in this section is shown below. The functions of the sample application used as a base and the specifications for authentication and authorization are also shown below.
| Henceforth, this sample application will be referred to as 'the application'.

.. _sec-requirements:

Security requirements
--------------------------------------------------------------------------------

The list of security requirements fulfilled by the application is shown below. The implementation example is described in :ref:`implement-description` for each classification.

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.50\linewidth}|p{0.20\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 20 30 40

    * - Sr. No.
      - Classification
      - Requirement
      - Overview
    * - | (1)
      - :ref:`Force/Prompt password change <password-change>`
      - Force password change when using initial password
      - Forces password change when authentication is successful using initial password
    * - | (2)
      - 
      - Force to change expired password
      - | Force password change when authentication is successful for the users who have not changed the password for a certain period
        | In this application, it is intended only for Administrator
    * - | (3)
      - 
      - Display message prompting password change
      - Displays message prompting password change when authentication is successful for the users who have not changed the password for a certain period
    * - | (4)
      - :ref:`Check password strength <password-strength>`
      - Specify minimum password length
      - Specifies the minimum length that can be set for the password
    * - | (5)
      - 
      - Specify the type of characters for the password
      - Specifies the type of characters (uppercase letters, lowercase letters, numbers, symbols) that must be included in the password
    * - | (6)
      - 
      - Prohibit user name from being used as password
      - Prohibit user name of the account from being used in the password
    * - | (7)
      - 
      - Prohibit reuse of administrator password
      - Prohibit reusing the password which has been recently used by the administrator
    * - | (8)
      - :ref:`Account lockout <account-lock>`
      - Account lockout
      - If authentication of a certain account has failed for more than a specific number of times within a short period, then that account is set to 'authentication disabled' state (lockout state)
    * - | (9)
      - 
      - Specify account lockout duration
      - Specifies the duration for account lockout state
    * - | (10)
      - 
      - Unlock by administrator
      - Administrator can unlock any account
    * - | (11)
      - :ref:`Display date and time of last login <last-login>`
      - Display date and time of last login
      - After successful authentication of an account, displays the date and time of last successful authentication of that account on the top screen
    * - | (12)
      - :ref:`Create authentication information for password reissue <reissue-info-create>`
      - Assign random string to the password reissue URL
      - In order to prevent unauthorized access, a string that is difficult to guess is assigned to URL which is used to access the password reissue screen
    * - | (13)
      - 
      - Issue confidential information for password reissue
      - Create confidential information in advance (Random string) that is difficult to guess, in order to use for user verification at the time of reissuing password
    * - | (14)
      - :ref:`Distribution of authentication information for password reissue <reissue-info-delivery>`
      - Send a mail for password reissue screen URL
      - Send the URL to access the password reissue screen to the registered e-mail address of the account
    * - | (15)
      - 
      - Separate distribution of the password reissue screen URL and confidential information
      - Distribute confidential information to the user using a mode other than e-mail as a precaution against leakage of password reissue screen URL
    * - | (16)
      - :ref:`Verification at the time of executing password reissue <reissue-info-validate>`
      - Set validity period for authentication information for password reissue
      - Set validity period for password reissue screen URL and confidential information, and disable password reissue screen URL and confidential information if the validity period has expired
    * - | (17)
      - :ref:`Set the maximum limit for password reissue failure <reissue-info-invalidate>`
      - Set the maximum limit for password reissue failure
      - Disable password reissue screen URL and confidential information when the authentication fails for a specific number of times at the time of password reissue

Functions
--------------------------------------------------------------------------------

The application consists of following functions in addition to the application created in :doc:`./Tutorial`.

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - Function name
      - Description
    * - Password change function
      - Function to enable logged-in users to change their account password
    * - Account lockout function
      - Function to set an account that has failed to authenticate more than a specific number of times in a short period to the 'authentication disabled' state
    * - Unlock function
      - Function to return the account which is in the 'authentication disabled' state due to the account lockout function, to the 'authentication enabled' state again
    * - Password reissue function
      - Function that can set a new password if the user has forgotten the password, after the confirmation with the user

.. note::
  Since this application is a sample of security measures, it is essentially required.
  Update function for registration information other than user registration function and password is not created.

Specifications for authentication/authorization
--------------------------------------------------------------------------------

In this application, the specifications for authentication/authorization are shown below respectively.

Authentication
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* Initial password to be used for authentication will be issued by the application

Authorization
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* Authentication is required to access the screens other than login screen and the screen used for password reissue
* There are two types of roles, "General user" and "Administrator"
    * A single account can have multiple roles
* Account unlock function can be used only by the account having administrator rights
      
Authentication at the time of reissuing password
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* The following information created by the application is used for the password reissue authentication
    * URL for Password reissue screen
    * Confidential information for authentication
* URL of the password reissue screen generated by the application is in the following format:
    * {baseUrl}/reissue/resetpassword?form&token={token}
        * {baseUrl} : Base URL of application
        * {token} : UUID version4 format string（36 characters including hyphen, 128bit）
* A time-limit of 30 minutes is provided for the password reissue screen URL and authentication is possible only within the validity period
      
Design information
--------------------------------------------------------------------------------

Page transition
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Screen transition diagram is shown below. Screen transition in case of an error is omitted.

.. figure:: ./images/SecureLogin_page_transition.png
   :alt: Page Transition
   :width: 80%
   :align: center

.. tabularcolumns:: |p{0.20\linewidth}|p{0.50\linewidth}|p{0.30\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 50 30

    * - | Sr. No.
      - | Screen name
      - | Access control
    * - | (1)
      - | Login screen
      - | -
    * - | (2)
      - | Top screen
      - | Authenticated users only
    * - | (3)
      - | Account information display screen
      - | Authenticated users only
    * - | (4)
      - | Password change screen
      - | Authenticated users only
    * - | (5)
      - | Password change completion screen
      - | Authenticated users only
    * - | (6)
      - | Unlock screen
      - | Administrator only
    * - | (7)
      - | Unlock completion screen
      - | Administrator only
    * - | (8)
      - | Screen to create authentication information for password reissue
      - | -
    * - | (9)
      - | Screen to complete creation of authentication information for password reissue
      - | -
    * - | (10)
      - | Password reissue screen
      - | -
    * - | (11)
      - | Password reissue completion screen
      - | -

URL List
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
URL list is shown below.

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|p{0.40\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 20 15 15 40

    * - Sr. No.
      - Process name
      - HTTP method
      - URL
      - Description
    * - 1
      - Login screen display
      - GET
      - /login
      - Displays login screen
    * - 2
      - Login
      - POST
      - /login
      - Authenticates by using username and password entered from login screen (performed by Spring Security)
    * - 3
      - Logout
      - POST
      - /logout
      - Performs logout (performed by Spring Security)
    * - 4
      - Top screen display
      - GET
      - /
      - Displays the Top screen
    * - 5
      - Account information display
      - GET
      - /account
      - Displays account information of logged-in user
    * - 6
      - Password change screen display
      - GET
      - /password?form
      - Displays the password change screen
    * - 7
      - Password change
      - POST
      - /password
      - Changes the password for the account using the information provided in the Password change screen
    * - 8
      - Password change completion screen display
      - GET
      - /password?complete
      - Displays password change completion screen
    * - 9
      - Unlock screen display
      - GET
      - /unlock?form
      - Displays the unlock screen
    * - 10
      - Unlock
      - POST
      - /unlock
      - Unlocks the account using the information provided on the unlock screen
    * - 11
      - Unlock completion screen display
      - GET
      - /unlock?complete
      - Displays the unlock completion screen
    * - 12
      - Authentication information creation screen display for password reissue
      - GET
      - /reissue/create?form
      - Displays the screen to create authentication information for password reissue
    * - 13
      - Create authentication information for password reissue
      - POST
      - /reissue/create
      - Creates authentication information for password reissue
    * - 14
      - Authentication information creation completion screen display for password reissue
      - GET
      - /reissue/create?complete
      - Displays the authentication information creation completion screen for password reissue
    * - 15
      - Password reissue screen display
      - GET
      - /reissue/resetpassword?form&token={token}
      - Displays 'User specific password reissue screen display' using the two request parameters
    * - 16
      - Password reissue
      - POST
      - /reissue/resetpassword
      - Reissue password using the information provided in the Password reissue screen
    * - 17
      - Password reissue completion screen display
      - GET
      - /reissue/resetpassword?complete
      - Displays password reissue completion screen

ER diagram
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ER diagram in this application is shown below.

.. figure:: ./images/SecureLogin_ER.png
   :alt: Entity-Relation Diagram
   :width: 80%
   :align: center

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.40\linewidth}|p{0.30\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 20 40 30

    * - Sr. No.
      - Entity name
      - Description
      - Attribute
    * - | (1)
      - | Account
      - | Registered account information of user
      - | username : User name
        | password : Password（Hashed）
        | firstName : First name
        | lastName : Last name
        | email : E-mail address
        | roles : Role(s)
    * - | (2)
      - | Role
      - | Rights to be used in authorization
      - | roleValue : Identifier of role
        | roleLabel : Display name of role
    * - | (3)
      - | Authentication successful event
      - | Information saved when authentication is successful in order to get the last login date and time of account
      - | username : User name
        | authenticationTimestamp : Date and time when authentication is successful
    * - | (4)
      - | Authentication failed event
      - | Information saved when authentication failed to be used by account lockout function
      - | username : User name
        | authenticationTimestamp : Date and time when authentication failed
    * - | (5)
      - | Password change history
      - | Information saved at the time of password change to be used to determine password expiration date
      - | username : User name
        | useFrom : Date and time when changed password is activated
        | password : Changed password
    * - | (6)
      - | Authentication information for password reissue
      - | Information to be used for user verification at the time of password reissue
      - | token : String used to make a unique and difficult to guess password reissue screen URL
        | username : User name
        | secret : String to be used for user verification
        | experyDate : Expiry date of authentication information for password reissue
    * - | (7)
      - | Password reissue failed event
      - | Information saved in password reissue failure to restrict the number of attempts for password reissue
      - | token : token used when failed to reissue password
        | attemptDate : Date and time when password reissue was attempted

.. tip ::

   In order to determine initial password and password expiration, a design can also be adopted wherein the information such as last modified date and time of password is provided by adding a field to the account entity.
   When implementation is done using this method, it is likely to lead to a situation where a column is added for determining various conditions in account table and entries are frequently updated.

   In this application, table is maintained in a simple form. In order to fulfil the requirements by simply using Insert and Delete without unnecessary updates of the entries, a design using event entity such as authentication successful event entity has been adopted.

.. _implement-description:

Implementation method and code description
================================================================================

| Method of implementation in this application and the code are described for each classification of security requirements.
| Only the minimum code required to fulfil the requirements for each classification is described here. Refer to `GitHub <https://github.com/terasolunaorg/tutorial-apps/tree/release/5.1.1.RELEASE/secure-login-demo>`_ for the complete code.
| SQL for initial data registration to run this application is placed `here <https://github.com/terasolunaorg/tutorial-apps/tree/release/5.1.1.RELEASE/secure-login-demo/secure-login-env/src/main/resources/database>`_.

.. note::

   In this application, Lombok is used to eliminate boilerplate code. For Lombok, refer :doc:`../Appendix/Lombok`.

.. _password-change:

Force/Prompt password change
--------------------------------------------------------------------------------

List of requirements to be implemented
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* :ref:`Force password change when initial password is used <sec-requirements>`
* :ref:`Force to change expired administrator password <sec-requirements>`
* :ref:`Display message prompting password change <sec-requirements>`

Working image
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. figure:: ./images/SecureLogin_change_password.png
   :alt: Change Password
   :width: 80%
   :align: center

Implementation method
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| In this application, the history when password is changed is stored as "Password change history" entity in the database. Using this password change history entity, the initial password and password expiration are determined.
| Note that, redirecting to the password change screen and displaying message on the screen are controlled based on the determination result.
| In particular, requirements are fulfilled by implementing and using the following process.

* Saving password change history entity

  When the password is changed, register password change history entity containing following information to the database.

  * User name of the account for which password is changed
  * Date and time when changed password is activated

* Determining initial password and password expiration

  | After authentication, search the password change history entity of the authenticated account from the database. If even a single record is not found, consider that initial password is being used.
  | Otherwise, get the latest password change history entity, calculate the difference between current date and time, and date and time when the password is activated and determine whether the password has expired.

* Forcible redirect to password change screen

  To force password change, the user is redirected to the password change screen in case a request is raised for a screen other than password change screen, when the conditions below are met.

  * When initial password is used by an authenticated user
  * When authenticated user is administrator and password has expired

  Using \ ``org.springframework.web.servlet.handler.HandlerInterceptor`` \ , determine whether the above conditions are met before executing handler method of Controller.

  .. tip ::
     
     There are other methods to redirect to the password change screen after authentication, however, depending on the method, it is likely that user gets access to a screen different from that of a password change screen by clicking the URL directly after redirecting.
     In the method that uses \ ``HandlerInterceptor`` \ , it cannot be avoided by a method wherein URL is directly clicked since the process is executed before executing handler method.

  .. tip ::
     Servlet Filter can also be used instead of \ ``HandlerInterceptor`` \ . For both the descriptions, refer to :ref:`controller-common-process`.
     Here, \ ``HandlerInterceptor`` \  is used to perform processing for only the requests allowed by the application.

* Display message prompting password change

  Call the password expiration determination process described previously in the Controller. Pass the determination result to View, and switch show/hide message in View.

Code description
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The code implemented according to the implementation method mentioned above is described sequentially.

* Saving password change history entity

  A series of implementations to register password change history entity in the database at the time of changing the password is shown below.

  * Implementation of Entity

    Implementation of password change history entity is as below.

    .. code-block:: java

       package org.terasoluna.securelogin.domain.model;

       // omitted

       @Data
       public class PasswordHistory {

           private String username; // (1)

           private String password; // (2)

           private DateTime useFrom; // (3)

       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - Sr. No.
         - Description
       * - | (1)
         - | User name of the account for which the password is changed
       * - | (2)
         - | Password after change
       * - | (3)
         - | Date and time of when changed password is activated

  * Implementation of Repository

    The Repository to register and search password change history entity to the database is shown below.

    .. code-block:: java

       package org.terasoluna.securelogin.domain.repository.passwordhistory;

       // omitted

       public interface PasswordHistoryRepository {

           int create(PasswordHistory history); // (1)

           List<PasswordHistory> findByUseFrom(@Param("username") String username,  
                   @Param("useFrom") LocalDateTime useFrom); // (2)

           List<PasswordHistory> findLatest(
                   @Param("username") String username, @Param("limit") int limit); // (3)

       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - Sr. No.
         - Description
       * - | (1)
         - | A method to register \ ``PasswordHistory`` \  object that has been assigned as an argument, as a record in the database
       * - | (2)
         - | A method to get \ ``PasswordHistory`` \  object newer than the date specifying the date and time when the password is activated, in descending order (new order) by considering user name that has been assigned as an argument, as the key
       * - | (3)
         - | A method to get specified number of \ ``PasswordHistory`` \  objects in new order by considering user name that has been assigned as an argument, as the key

    Mapping file is as described below.

    .. code-block:: xml

       <?xml version="1.0" encoding="UTF-8"?>
       <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

       <mapper
           namespace="org.terasoluna.securelogin.domain.repository.passwordhistory.PasswordHistoryRepository">

           <resultMap id="PasswordHistoryResultMap" type="PasswordHistory">
               <id property="username" column="username" />
               <id property="password" column="password" />
               <id property="useFrom" column="use_from" />
           </resultMap>

           <select id="findByUseFrom" resultMap="PasswordHistoryResultMap">
           <![CDATA[
               SELECT
                   username,
                   password,
                   use_from
               FROM
                   password_history
               WHERE
                   username = #{username} AND
                   use_from >= #{useFrom}
               ORDER BY use_from DESC
           ]]>
           </select>

           <select id="findLatest" resultMap="PasswordHistoryResultMap">
           <![CDATA[
               SELECT
                   username,
                   password,
                   use_from
               FROM
                   password_history
               WHERE
                   username = #{username}
               ORDER BY use_from DESC
               LIMIT #{limit}
           ]]>
           </select>

           <insert id="create" parameterType="PasswordHistory">
           <![CDATA[
               INSERT INTO password_history (
                   username,
                   password,
                   use_from
               ) VALUES (
                   #{username},
                   #{password},
                   #{useFrom}
               )
           ]]>
           </insert>
       </mapper>


  * Service implementation

    Password change history entity operations are also used in :ref:`Check password strength <password-strength>`.
    Therefore, call the Repository method from SharedService as shown below.

    .. code-block:: java

       package org.terasoluna.securelogin.domain.service.passwordhistory;

       // omitted

       @Service
       @Transactional
       public class PasswordHistorySharedServiceImpl implements
               PasswordHistorySharedService {

           @Inject
           PasswordHistoryRepository passwordHistoryRepository;

           @Transactional(propagation = Propagation.REQUIRES_NEW)
           public int insert(PasswordHistory history) {
               return passwordHistoryRepository.create(history);
           }

           @Transactional(readOnly = true)
           public List<PasswordHistory> findHistoriesByUseFrom(String username,
                   LocalDateTime useFrom) {
               return passwordHistoryRepository.findByUseFrom(username, useFrom);
           }

           @Override
           @Transactional(readOnly = true)
           public List<PasswordHistory> findLatest(String username, int limit) {
               return passwordHistoryRepository.findLatest(username, limit);
           }

       }

    Implementation of the process to save password change history entity in the database at the time of changing the password is shown below.

    .. code-block:: java

       package org.terasoluna.securelogin.domain.service.account;

       // omitted

       @Service
       @Transactional
       public class AccountSharedServiceImpl implements AccountSharedService {

           @Inject
           ClassicDateFactory dateFactory;

           @Inject
           PasswordHistorySharedService passwordHistorySharedService;

           @Inject
           AccountRepository accountRepository;

           @Inject
           PasswordEncoder passwordEncoder;

           // omitted

           public boolean updatePassword(String username, String rawPassword) { // (1)
               String password = passwordEncoder.encode(rawPassword);
               boolean result = accountRepository.updatePassword(username, password); // (2)

               LocalDateTime passwordChangeDate = dateFactory.newTimestamp().toLocalDateTime();

               PasswordHistory passwordHistory = new PasswordHistory(); // (3)
               passwordHistory.setUsername(username);
               passwordHistory.setPassword(password);
               passwordHistory.setUseFrom(passwordChangeDate);
               passwordHistorySharedService.insert(passwordHistory); // (4)

               return result;
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
         - | A Method which is called while changing the password
       * - | (2)
         - | Call the process to update the password in the database.
       * - | (3)
         - | Create password change history entity and set user name, changed password, and date and time when changed password is activated.
       * - | (4)
         - | Call the process to register the created password change history entity in the database.


* Determining initial password and password expiration

  Using the password change history entity registered in the database, implementation of the process to determine whether initial password is used and whether the password has expired is shown below.

  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.account;

     // omitted

     @Service
     @Transactional
     public class AccountSharedServiceImpl implements AccountSharedService {

         @Inject
         ClassicDateFactory dateFactory;

         @Inject
         PasswordHistorySharedService passwordHistorySharedService;

         @Value("${security.passwordLifeTimeSeconds}") // (1)
         int passwordLifeTimeSeconds;

         // omitted

        @Transactional(readOnly = true)
        @Override
        @Cacheable("isInitialPassword")
        public boolean isInitialPassword(String username) { // (2)
            List<PasswordHistory> passwordHistories = passwordHistorySharedService
                    .findLatest(username, 1); // (3)
            return passwordHistories.isEmpty(); // (4)
        }

        @Transactional(readOnly = true)
        @Override
        @Cacheable("isCurrentPasswordExpired")
        public boolean isCurrentPasswordExpired(String username) { // (5)
            List<PasswordHistory> passwordHistories = passwordHistorySharedService
                    .findLatest(username, 1); // (6)

            if (passwordHistories.isEmpty()) { // (7)
                return true;
            }

            if (passwordHistories
                    .get(0)
                    .getUseFrom()
                    .isBefore(
                            dateFactory.newTimestamp().toLocalDateTime()
                                    .minusSeconds(passwordLifeTimeSeconds))) { // (8)
                return true;
            }

            return false;
        }

     }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - Sr. No.
       - Description
     * - | (1)
       - | Fetch the length of the time period (in seconds) for which password is valid, from the property file and set.
     * - | (2)
       - | A method which determines whether initial password is used, and returns true if it is used, or else returns false.
     * - | (3)
       - | Call the process to fetch single record of the latest password change history entity from the database.
     * - | (4)
       - | If password change history entity cannot be fetched from the database, determine that initial password is being used and return true. Otherwise, return false.
     * - | (5)
       - | A method which determines whether the password currently being used has expired and returns true if it has expired, or else returns false.
     * - | (6)
       - | Call the process to fetch single record of the latest password change history entity from the database.
     * - | (7)
       - | If password change history entity cannot be fetched from the database, determine that the password has expired and return true.
     * - | (8)
       - | If difference between the current date and time, and the date and time when the password fetched from the password change history entity is activated, is greater than the password validity period set in (1), determine that the password has expired and return true.
     * - | (9)
       - | If any of the conditions of (7), (8) is not met, determine that the password is within the validity period and return false.

  .. tip::

     \ ``@ Cacheable`` \  assigned to isInitialPassword and isCurrentPasswordExpired is an annotation to use the Spring Cache Abstraction function.
     The result for method arguments can be cached by assigning \ ``@Cacheable`` \  annotation.
     Access to database during each initial password and password expiration determination is prevented by the use of the cache thereby preventing performance degradation.
     Refer to `Official document <http://docs.spring.io/spring/docs/4.2.4.RELEASE/spring-framework-reference/html/cache.html>`_ for Cache Abstraction.

     Further, while using cache, it should be noted that it is necessary to clear the cache as and when needed.
     In this application, at the time of changing the password or during logout, clear the cache to determine password expiration and determine initial password again.

     Further, set cache TTL (Time to Live) as needed. Note that TTL is not set depending on the implementation of the cache to be used.


* Forcible redirect to password change screen

  In order to enforce password change, the implementation of the process to be redirected to the password change screen is shown below.

  .. code-block:: java

     package org.terasoluna.securelogin.app.common.interceptor;

     // omitted

     public class PasswordExpirationCheckInterceptor extends
             HandlerInterceptorAdapter { // (1)

         @Inject
         AccountSharedService accountSharedService;

         @Override
         public boolean preHandle(HttpServletRequest request,
                 HttpServletResponse response, Object handler) throws IOException { // (2)
             Authentication authentication = (Authentication) request
                     .getUserPrincipal();

             if (authentication != null) {
                 Object principal = authentication.getPrincipal();
                 if (principal instanceof UserDetails) { // (3)
                     LoggedInUser userDetails = (LoggedInUser) principal; // (4)
                     if ((userDetails.getAccount().getRoles().contains(Role.ADMIN) && accountSharedService
                             .isCurrentPasswordExpired(userDetails.getUsername())) // (5)
                             || accountSharedService.isInitialPassword(userDetails
                                     .getUsername())) { // (6)
                         response.sendRedirect(request.getContextPath() 
                                 + "/password?form"); // (7)
                         return false; // (8)
                     }
                 }
             }

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
       - | Inherit \ ``org.springframework.web.servlet.handler.HandlerInterceptorAdapter`` \  to include the process before the execution of handler method of Controller.
     * - | (2)
       - | A method executed before the execution of handler method of Controller
     * - | (3)
       - | Check whether the fetched user information is an object of \ ``org.springframework.security.core.userdetails.UserDetails`` \ .
     * - | (4)
       - | Fetch \ ``UserDetails`` \  object . In this application, a class called \ ``LoggedInUser`` \  is created and used as the implementation of \ ``UserDetails`` \ .
     * - | (5)
       - | Determine whether the user is an administrator by fetching the role from \ ``UserDetails`` \  object. Then, call the process to determine whether the password has expired. Perform logical AND (And) of these two results.
     * - | (6)
       - | Call the process to determine whether initial password is being used.
     * - | (7)
       - | If either (5) or (6) is true, redirect to the password change screen using \ ``sendRedirect`` \  method of \ ``javax.servlet.http.HttpServletResponse`` \ .
     * - | (8)
       - | Return false to prevent handler method of Controller being executed continuously.

  The settings to enable the redirect process described above are as described below.

  **spring-mvc.xml**

  .. code-block:: xml

    <!-- omitted -->

    <mvc:interceptors>

        <!-- omitted -->

        <mvc:interceptor>
            <mvc:mapping path="/**" /> <!-- (1) -->
            <mvc:exclude-mapping path="/password/**" /> <!-- (2) -->
            <mvc:exclude-mapping path="/reissue/**" /> <!-- (3) -->
            <mvc:exclude-mapping path="/resources/**" />
            <mvc:exclude-mapping path="/**/*.html" />
            <bean
                class="org.terasoluna.securelogin.app.common.interceptor.PasswordExpirationCheckInterceptor" /> <!-- (4) -->
        </mvc:interceptor>

        <!-- omitted -->

    </mvc:interceptors>

    <!-- omitted -->

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - Sr. No.
       - Description
     * - | (1)
       - | Use \ ``HandlerInterceptor`` \  to access all the paths under "/".
     * - | (2)
       - | Exclude the paths under "/password" to prevent redirecting from password change screen to password change screen.
     * - | (3)
       - | Exclude the paths under "/reissue" since it is not necessary to check password expiration at the time of reissuing password.
     * - | (4)
       - | Specify the class of \ ``HandlerInterceptor`` \ .

* Display message prompting password change

  Implementation of Controller to display message prompting password change on top screen is shown below.

  .. code-block:: java

     package org.terasoluna.securelogin.app.welcome;

     // omitted

     @Controller
     public class HomeController {

         @Inject
         AccountSharedService accountSharedService;

         @RequestMapping(value = "/", method = { RequestMethod.GET,
                 RequestMethod.POST })
         public String home(@AuthenticationPrincipal LoggedInUser userDetails, // (1)
                 Model model) {

             Account account = userDetails.getAccount(); // (2)

             model.addAttribute("account", account);
             
             if(accountSharedService.isCurrentPasswordExpired(account.getUsername())){ // (3)
                 ResultMessages messages = ResultMessages.warning().add("w.sl.pe.0001");
                 model.addAttribute(messages);
             }

             // omitted        
             
             return "welcome/home";

         }

     }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - Sr. No.
       - Description
     * - | (1)
       - | Fetch object of \ ``LoggedInUser`` \  for which \ ``UserDetails`` \  is implemented by specifying \ ``AuthenticationPrincipal`` \  annotation.
     * - | (2)
       - | Fetch account information retained by \ ``LoggedInUser`` \ .
     * - | (3)
       - | Call the password expiration determination process by using the user name obtained from account information as an argument. If the result is true, fetch the message from the property file, set it in Model and pass it to View.

  Implementation of View is as follows:

  **Top screen(home.jsp)**

  .. code-block:: jsp

     <!-- omitted -->

     <body>
        <div id="wrapper">
            <span id="expiredMessage">
                <t:messagesPanel /> <!-- (1) -->
            </span>

            <!-- omitted -->

        </div>
     </body>

     <!-- omitted -->

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - Sr. No.
       - Description
     * - | (1)
       - | Using messagesPanel tag, display the password expiration message passed from the Controller.

.. _password-strength:

Check password strength
--------------------------------------------------------------------------------
List of requirements to be implemented
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
* :ref:`Specify minimum number of characters for password <sec-requirements>`
* :ref:`Specify character type for password <sec-requirements>`
* :ref:`Prohibit password containing user name <sec-requirements>`
* :ref:`Prohibit reuse of administrator password <sec-requirements>`

Working image
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. figure:: ./images/SecureLogin_password_validation.png
   :alt: Password Validation
   :width: 80%
   :align: center

Implementation method
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| :doc:`../ArchitectureInDetail/Validation` function can be used to verify the strength of the password specified by the user at the time of password change. In this application, the strength of the password is verified by using Bean Validation.
| Requirements for password strength are wide-ranging and differ depending on the application.
| Use `Passay <http://www.passay.org/>`_ as the library for password validation and create the required Bean Validation annotation.
| Many functions that are commonly used in password validation have been provided in Passay. The functions that have not been provided can also be easily implemented by extending the standard functions.
| Refer to :ref:`Appendix <passay_overview>` for the overview of Passay.
| In particular, describe the following settings and process and fulfil the requirements using them.

* Creating validation rules for Passay

  Create the following validation rules to be used to fulfil the requirements.

    * Validation rule wherein minimum password length is set
    * Validation rule wherein the character type that must be included in the password is set
    * Validation rule to check that the password does not contain user name
    * Validation rule to check that same password has not been used recently

* Creating Passay validator

  Create Passay validator wherein validation rules created above, are set.

* Creating Bean Validation annotation

  Create an annotation for password validation using Passay validator.
  All the validation rules can also be verified by one annotation, however, verifying various rules leads to complex process and reduced visibility. To avoid this, it should be implemented by dividing into two as shown below.

    * Annotation to validate the characteristics of the password

      Check the three validation rules, "Password is longer than the minimum string length", "Password includes characters of the specified character type", and "Password does not contain user name"
    * Annotation to compare with previous password

      Check that the recently used password is not reused by the administrator in recent period of time.

  Any annotation is a correlation input check rule using user name and new password.
  When a violation occurs in either of the inputs of both the rules, respective error message is displayed.

* Password validation

  Perform password validation using the created Bean Validation annotation.

Code description
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The code implemented according to the implementation method mentioned above is described sequentially. Password validation using Passay is described in :ref:`password_validation`.

* Creating validation rules for Passay

  | Most of the verification rules used in this application can be defined by using the class provided in Passay by default.
  | However, in the class provided by Passay, validation rule to compare with the hashed previous password cannot be defined in \ ``org.springframework.security.crypto.password.PasswordEncoder`` \ .
  | Therefore, it is necessary to create a class with individual validation rules by extending the class provided by Passay as shown below.

  .. code-block:: java

     package org.terasoluna.securelogin.app.common.validation.rule;

     // omitted

     public class EncodedPasswordHistoryRule extends HistoryRule { // (1)

         PasswordEncoder passwordEncoder; // (2)

         public EncodedPasswordHistoryRule(PasswordEncoder passwordEncoder) {
             this.passwordEncoder = passwordEncoder;
         }

         @Override
         protected boolean matches(final String clearText,
                 final PasswordData.Reference reference) { // (3)
             return passwordEncoder.matches(clearText, reference.getPassword()); // (4)
         }
     }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - Sr. No.
       - Description
     * - | (1)
       - | Extend \ ``org.passay.HistoryRule`` \  to check that password is not a recently used password.
     * - | (2) 
       - | Inject \ ``PasswordEncoder`` \  used for password hashing.
     * - | (3)
       - | Override the method to compare with previous password.
     * - | (4)
       - | Compare with hashed password using \ ``matches`` \  method of ``PasswordEncoder`` \ .

  Define a Bean for validation rules of Passay as shown below.

  **applicationContext.xml**

  .. code-block:: xml

     <bean id="lengthRule" class="org.passay.LengthRule"> <!-- (1) -->
         <property name="minimumLength" value="${security.passwordMinimumLength}" /> 
     </bean>
     <bean id="upperCaseRule" class="org.passay.CharacterRule"> <!-- (2) -->
         <constructor-arg name="data">
             <util:constant static-field="org.passay.EnglishCharacterData.UpperCase" />
         </constructor-arg>
         <constructor-arg name="num" value="1" />
     </bean>
     <bean id="lowerCaseRule" class="org.passay.CharacterRule"> <!-- (3) -->
         <constructor-arg name="data">
             <util:constant static-field="org.passay.EnglishCharacterData.LowerCase" />
         </constructor-arg>
         <constructor-arg name="num" value="1" />
     </bean>
     <bean id="digitRule" class="org.passay.CharacterRule"> <!-- (4) -->
         <constructor-arg name="data">
             <util:constant static-field="org.passay.EnglishCharacterData.Digit" />
         </constructor-arg>
         <constructor-arg name="num" value="1" />
     </bean>
     <bean id="specialCharacterRule" class="org.passay.CharacterRule"> <!-- (5) -->
         <constructor-arg name="data">
             <util:constant static-field="org.passay.EnglishCharacterData.Special" />
         </constructor-arg>
         <constructor-arg name="num" value="1" />
     </bean>
     <bean id="characterCharacteristicsRule" class="org.passay.CharacterCharacteristicsRule"> <!-- (6) -->
         <property name="rules">
             <list>
                 <ref bean="upperCaseRule" />
                 <ref bean="lowerCaseRule" />
                 <ref bean="digitRule" />
                 <ref bean="specialCharacterRule" />
             </list>
         </property>
         <property name="numberOfCharacteristics" value="3" />
     </bean>
     <bean id="usernameRule" class="org.passay.UsernameRule" /> <!-- (7) -->
     <bean id="encodedPasswordHistoryRule"
         class="org.terasoluna.securelogin.app.common.validation.rule.EncodedPasswordHistoryRule"> <!-- (8) -->
         <constructor-arg name="passwordEncoder" ref="passwordEncoder" />
     </bean>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - Sr. No.
       - Description
     * - | (1)
       - | In \ ``org.passay.LengthRule`` \  property to check the password length, set the minimum length of the password fetched from the property file.
     * - | (2) 
       - | A validation rule to check that one or more single-byte upper-case letters are included. Set \ ``org.passay.EnglishCharacterData.UpperCase`` \  and numerical value 1 in \ ``org.passay.CharacterRule`` \  constructor to check the character type included in the password.
     * - | (3)
       - | A validation rule to check that one or more single-byte lower-case letters are included. Set \ ``org.passay.EnglishCharacterData.LowerCase`` \  and numerical value 1 in \ ``org.passay.CharacterRule`` \  constructor to check the character type included in the password.
     * - | (4)
       - | A validation rule to check that one or more single-byte digits are included. Set \ ``org.passay.EnglishCharacterData.Digit`` \  and numerical value 1 in \ ``org.passay.CharacterRule`` \  constructor to check for the character type included in the password.
     * - | (5)
       - | A validation rule to check that one or more single-byte symbols are included. Set \ ``org.passay.EnglishCharacterData.Special`` \  and numerical value 1 in \ ``org.passay.CharacterRule`` \  constructor to check for the character type included in the password.
     * - | (6)
       - | A validation rule to check that 3 out of the 4 validation rules from (2)-(5) are met. Set Bean list defined in (2)-(5) and numerical value 3 in \ ``org.passay.CharacterCharacteristicsRule`` \  property.
     * - | (7)
       - | A validation rule to check that password does not contain user name
     * - | (8)
       - | A validation rule to check that the password is not included in the passwords used in the past

* Creating Passay validator

  Using validation rules of Passay described above, Bean definition for validator to perform actual validation is shown below.

  **applicationContext.xml**

  .. code-block:: xml

     <bean id="characteristicPasswordValidator" class="org.passay.PasswordValidator"> <!-- (1) -->
         <constructor-arg name="rules">
             <list>
                 <ref bean="lengthRule" />
                 <ref bean="characterCharacteristicsRule" />
                 <ref bean="usernameRule" />
             </list>
         </constructor-arg>
     </bean>
     <bean id="encodedPasswordHistoryValidator" class="org.passay.PasswordValidator"> <!-- (2) -->
         <constructor-arg name="rules">
             <list>
                 <ref bean="encodedPasswordHistoryRule" />
             </list>
         </constructor-arg>
     </bean>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - Sr. No.
       - Description
     * - | (1)
       - | A validator to validate the characteristics of the password. Set a Bean for \ ``LengthRule`` \ , \ ``CharacterCharacteristicsRule`` \ , \ ``UsernameRule`` \  as a property.
     * - | (2)
       - | A validator to check the history of the passwords that were used in the past. Set a Bean for \ ``EncodedPasswordHistoryRule`` \  as a property.

* Creating Bean Validation annotation

  To fulfil the requirements, create two annotations that use the validator described above.

  * Annotation to validate the characteristics of the password

    The implementation of the annotation to check three validation rules - 'password should be longer than the minimum string length, it should contain characters of specified character type and it should not contain user name' is shown below.

    .. code-block:: java

       package org.terasoluna.securelogin.app.common.validation;

       // omitted

       @Documented
       @Constraint(validatedBy = { StrongPasswordValidator.class }) // (1)
       @Target({ TYPE, ANNOTATION_TYPE })
       @Retention(RUNTIME)
       public @interface StrongPassword {
           String message() default "{org.terasoluna.securelogin.app.common.validation.StrongPassword.message}";

           Class<?>[] groups() default {};

           String usernamePropertyName(); // (2)

           String newPasswordPropertyName(); // (3)

           @Target({ TYPE, ANNOTATION_TYPE })
           @Retention(RUNTIME)
           @Documented
           public @interface List {
               StrongPassword[] value();
           }

           Class<? extends Payload>[] payload() default {};
       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - Sr. No.
         - Description
       * - | (1)
         - | Specify \ ``ConstraintValidator`` \  to be used at the time of assigning annotation.
       * - | (2)
         - | A property to specify property name of the user name.
       * - | (3)
         - | A property to specify property name for the password.

    .. code-block:: java

       package org.terasoluna.securelogin.app.common.validation;

       // omitted

       public class StrongPasswordValidator implements
               ConstraintValidator<StrongPassword, Object> {

           @Inject
           @Named("characteristicPasswordValidator") // (1)
           PasswordValidator characteristicPasswordValidator;

           private String usernamePropertyName;

           private String newPasswordPropertyName;

           @Override
           public void initialize(StrongPassword constraintAnnotation) {
               usernamePropertyName = constraintAnnotation.usernamePropertyName();
               newPasswordPropertyName = constraintAnnotation.newPasswordPropertyName();
           }

           @Override
           public boolean isValid(Object value, ConstraintValidatorContext context) {
               BeanWrapper beanWrapper = new BeanWrapperImpl(value);
               String username = (String) beanWrapper.getPropertyValue(usernamePropertyName);
               String newPassword = (String) beanWrapper
                       .getPropertyValue(newPasswordPropertyName);

               RuleResult result = characteristicPasswordValidator
                       .validate(PasswordData.newInstance(newPassword, username, null)); // (2)

               if (result.isValid()) { // (3)
                   return true;
               } else {
                   context.disableDefaultConstraintViolation();
                   for (String message : characteristicPasswordValidator
                           .getMessages(result)) { // (4)
                       context.buildConstraintViolationWithTemplate(message)
                               .addPropertyNode(newPasswordPropertyName)
                               .addConstraintViolation();
                   }
                   return false;
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
         - | Inject validator of Passay.
       * - | (2)
         - | Create an instance of ``org.passay.PasswordData`` wherein password and user name are specified and perform validation by the validator.
       * - | (3)
         - | Confirm the check result, if it is OK, return true, else return false.
       * - | (4)
         - | Fetch and set all the password validation error messages.

  * Annotation to compare with password used in the past

    | Implementation of the annotation to check that the administrator does not reuse the password used earlier within a short period of time, is shown below.
    | Password change history entity is used to get the password used in the past. Refer to :ref:`Force/Prompt password change <password-change>` for the password change history entity.

    .. note ::

       In the setting of "Prevent reuse of password used before specified period", it is possible to reuse a password by repeating a password within a short period of time.
       In order to prevent this, check is performed in this application by setting "Prevent reuse of password used from a certain period onwards

    .. code-block:: java

       package org.terasoluna.securelogin.app.common.validation;

       @Documented
       @Constraint(validatedBy = { NotReusedPasswordValidator.class }) // (1)
       @Target({ TYPE, ANNOTATION_TYPE })
       @Retention(RUNTIME)
       public @interface NotReusedPassword {
           String message() default "{org.terasoluna.securelogin.app.common.validation.NotReusedPassword.message}";

           Class<?>[] groups() default {};

           String usernamePropertyName(); // (2)

           String newPasswordPropertyName(); // (3)

           @Target({ TYPE, ANNOTATION_TYPE })
           @Retention(RUNTIME)
           @Documented
           public @interface List {
               NotReusedPassword[] value();
           }

           Class<? extends Payload>[] payload() default {};
       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - Sr. No.
         - Description
       * - | (1)
         - | Specify \ ``ConstraintValidator`` \  to be used while assigning an annotation.
       * - | (2)
         - | A property to specify the property name of the user name. It is required to search the password used in the past, from the database.
       * - | (3)
         - | A property to specify the property name of the password.

    .. code-block:: java

       package org.terasoluna.securelogin.app.common.validation;

       // omitted

       public class NotReusedPasswordValidator implements
               ConstraintValidator<NotReusedPassword, Object> {

           @Inject
           ClassicDateFactory dateFactory;

           @Inject
           AccountSharedService accountSharedService;

           @Inject
           PasswordHistorySharedService passwordHistorySharedService;

           @Inject
           PasswordEncoder passwordEncoder;

           @Inject
           @Named("encodedPasswordHistoryValidator") // (1)
           PasswordValidator encodedPasswordHistoryValidator;

           @Value("${security.passwordHistoricalCheckingCount}") // (2)
           int passwordHistoricalCheckingCount;

           @Value("${security.passwordHistoricalCheckingPeriod}") // (3)
           int passwordHistoricalCheckingPeriod;

           private String usernamePropertyName;

           private String newPasswordPropertyName;

           private String message;

           @Override
           public void initialize(NotReusedPassword constraintAnnotation) {
               usernamePropertyName = constraintAnnotation.usernamePropertyName();
               newPasswordPropertyName = constraintAnnotation.newPasswordPropertyName();
               message = constraintAnnotation.message();
           }

           @Override
           public boolean isValid(Object value, ConstraintValidatorContext context) {
               BeanWrapper beanWrapper = new BeanWrapperImpl(value);
               String username = (String) beanWrapper.getPropertyValue(usernamePropertyName);
               String newPassword = (String) beanWrapper
                       .getPropertyValue(newPasswordPropertyName);

               Account account = accountSharedService.findOne(username);
               String currentPassword = account.getPassword();

               boolean result = checkNewPasswordDifferentFromCurrentPassword(
                       newPassword, currentPassword, context); // (4)
               if (result && account.getRoles().contains(Role.ADMIN)) { // (5)
                   result = checkHistoricalPassword(username, newPassword, context);
               }

               return result;
           }

           private boolean checkNewPasswordDifferentFromCurrentPassword(
                   String newPassword, String currentPassword,
                   ConstraintValidatorContext context) {
               if (!passwordEncoder.matches(newPassword, currentPassword)) {
                   return true;
               } else {
       	           context.disableDefaultConstraintViolation();
                   context.buildConstraintViolationWithTemplate(message)
                           .addPropertyNode(newPasswordPropertyName).addConstraintViolation();
                   return false;
               }
           }

           private boolean checkHistoricalPassword(String username,
                   String newPassword, ConstraintValidatorContext context) {
               LocalDateTime useFrom = dateFactory.newTimestamp().toLocalDateTime()
                       .minusMinutes(passwordHistoricalCheckingPeriod);
               List<PasswordHistory> historyByTime = passwordHistorySharedService
                       .findHistoriesByUseFrom(username, useFrom);
               List<PasswordHistory> historyByCount = passwordHistorySharedService
                       .findLatest(username, passwordHistoricalCheckingCount);
               List<PasswordHistory> history = historyByCount.size() > historyByTime
                       .size() ? historyByCount : historyByTime; // (6)

               List<PasswordData.Reference> historyData = new ArrayList<>();
               for (PasswordHistory h : history) {
                   historyData.add(new PasswordData.HistoricalReference(h
                           .getPassword())); // (7)
               }

               PasswordData passwordData = PasswordData.newInstance(newPassword,
                       username, historyData); // (8)
               RuleResult result = encodedPasswordHistoryValidator
                       .validate(passwordData); // (9)

               if (result.isValid()) { // (10)
                   return true;
               } else {
       	           context.disableDefaultConstraintViolation();
                   context.buildConstraintViolationWithTemplate(
                           encodedPasswordHistoryValidator.getMessages(result).get(0)) // (11)
                           .addPropertyNode(newPasswordPropertyName).addConstraintViolation();
                   return false;
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
         - | Inject Passay validator.
       * - | (2)
         - | Fetch the threshold to prohibit reuse of password up to a previous date, from the property file and inject it.
       * - | (3)
         - | Fetch the threshold (in seconds) to prohibit the reuse of the password used from a date onwards, from the property file and inject it.
       * - | (4)
         - | Call the process to check whether new password is different from the currently used password. Perform this check regardless of the general user / administrator.
       * - | (5)
         - | In case of administrator, call the process to check that new password is not included in the previously used passwords.
       * - | (6)
         - | Fetch the number of password change history entities specified in (2) and the password change history entities of the period specified in (3) and use the larger number of the two for the subsequent checks.
       * - | (7)
         - | In order to make a comparison with the previous password using Passay validator, fetch the password from the password change history entity and create a list of \ ``org.passay.PasswordData.HistoricalReference`` \ .
       * - | (8)
         - | Create an instance of \ ``org.passay.PasswordData`` \  which specifies password, user name and list of previous passwords.
       * - | (9)
         - | Perform validation by using the validator.
       * - | (10)
         - | Confirm the check result, if it is OK, return true, else return false.
       * - | (11)
         - | Fetch the password validation error messages.


* Password validation

  Perform password validation in the application layer which use Bean Validation annotation.
  Since input check other than Null check is covered by the annotation assigned to Form class, only \ ``@NotNull`` \  is assigned as a single item check.

  .. code-block:: java

     package org.terasoluna.securelogin.app.passwordchange;

     // omitted

     import lombok.Data;

     @Data
     @Compare(source = "newPasssword", destination = "confirmNewPassword", operator = Compare.Operator.EQUAL) // (1)
     @StrongPassword(usernamePropertyName = "username", newPasswordPropertyName = "newPassword") // (2)
     @NotReusedPassword(usernamePropertyName = "username", newPasswordPropertyName = "newPassword") // (3)
     @ConfirmOldPassword(usernamePropertyName = "username", oldPasswordPropertyName = "oldPassword") // (4)
     public class PasswordChangeForm implements Serializable{

         private static final long serialVersionUID = 1L;
         
         @NotNull
         private String username;

         @NotNull
         private String oldPassword;

         @NotNull
         private String newPassword;

         @NotNull
         private String confirmNewPassword;

     }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - Sr. No.
       - Description
     * - | (1)
       - | An annotation to check whether second input of new password is identical with the first input. Refer to :ref:`Validation_terasoluna_gfw_list` for the details.
     * - | (2)
       - | An annotation to verify the characteristic of the password, described above
     * - | (3)
       - | An annotation to compare with the previous password
     * - | (4)
       - | An annotation to check that the entered current password is correct. Definition will be omitted.

  .. code-block:: java

     package org.terasoluna.securelogin.app.passwordchange;

     // omitted

     @Controller
     @RequestMapping("password")
     public class PasswordChangeController {

         @Inject
         PasswordChangeService passwordService;

         // omitted

         @RequestMapping(method = RequestMethod.POST)
         public String change(@AuthenticationPrincipal LoggedInUser userDetails,
                 @Validated PasswordChangeForm form, BindingResult bindingResult, // (1)
                 Model model) {

             Account account = userDetails.getAccount();
             if (bindingResult.hasErrors() ||
                     !account.getUsername().equals(form.getUsername())) { // (2)
                 model.addAttribute(account);
                 return "passwordchange/changeForm";
             }

             passwordService.updatePassword(form.getUsername(),
                     form.getNewPassword());

             return "redirect:/password?complete";
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
       - | A handler method called at the time of changing the password. Perform validation by assigning \ ``@Validated`` \  annotation to Form in the parameter.
     * - | (2)
       - | Confirm that the user name for password change and the user name of the logged-in account are identical. If the two users are different, the user is again taken to the password change screen.

  .. note::

     In this application, user name is fetched from the Form to perform password validation using the user name in Bean Validation.
     It is assumed that in View, the user name set in \ ``Model`` \  is retained as hidden, however, since there is a risk of tampering, user name obtained from the Form before password change is confirmed.

.. _account-lock:

Account lock
--------------------------------------------------------------------------------
List of requirements to be implemented
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
* :ref:`Account lock <sec-requirements>`
* :ref:`Specifying account lockout duration <sec-requirements>`
* :ref:`Unlocking by the administrator <sec-requirements>`

Working image
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* Account lock

.. figure:: ./images/SecureLogin_lockout_ss.png
   :alt: Lockout
   :width: 80%
   :align: center

| In the login form, if you try to authenticate a user name with an incorrect password for a certain number of times, successively in a short duration, then that user's account will be locked.
  Locked account is not authenticated even if a set of correct user name and password is entered.
| Locked status is cancelled after a certain period of time or by unlocking it.

* Unlock

.. figure:: ./images/SecureLogin_unlock_ss.png
   :alt: Unlock
   :width: 80%
   :align: center

Unlock function can be used only when the user having administrator rights has logged in.
If unlocking is carried out by entering the user name for which the locked status is to be resolved, then the account of that user returns to the status wherein authentication can be done again.

Implementation method
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| In Spring Security, an account lockout status can be set for \ ``org.springframework.security.core.userdetails.UserDetails`` \ .
| If "Locked" is set, Spring Security reads that setting and throws \ ``org.springframework.security.authentication.LockedException`` \ .
| By using this function, if only the process set in \ ``UserDetails`` \  is implemented by determining whether the account is locked, lockout function can be implemented.

| In this application, the history of authentication failure is stored in the database as an "authentication failure event" entity, and the lockout status of the account is determined using this authentication failure event entity.
| In particular, each requirement related to account lockout is fulfilled by implementing and using the following three processes.

* Storing authentication failure event entity

  In case of authentication failure due to invalid authentication information input, the events generated by Spring Security are handled and the user name used for authentication and the date and time when authentication was attempted are registered in the database as authentication failure event entity.

* Determining lockout status

  For some accounts, if a certain number of new authentication failure event entities at the current time are more than a certain fixed number, the corresponding account is determined to be locked.
  Call this determination process during authentication and set the determination results in the implementation class of \ ``UserDetails`` \ .

* Deleting authentication failure event entity

  | Delete all authentication failure event entities for an account.
  | Since an account is targeted for lockout only when it fails to authenticate continuously, delete the authentication failure event entity when authentication is successful.
  | Also, since the lockout status of the account is determined using the authentication failure event entity, unlock function can be implemented by deleting the authentication failure event entity.
    Prevent account lockout from being executed by other than administrator using authorization function.

.. warning::

   Since authentication failure event entity is intended only to determine lockout, it is deleted when it is no longer required.
   A separate log should be always saved when authentication log is required.

The working example of lockout function which uses authentication failure event entity is described with the help of the following figure.
Lockout by authentication failure for 3 times and lockout duration of 10 minutes is considered as an example.

.. figure:: ./images/SecureLogin_lockout.png
   :alt: Account Lockout
   :width: 60%
   :align: center
  
.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90
  
   * - Sr. No.
     - Description
   * - | (1)
     - | Authentication with incorrect password has been attempted three times in last 10 minutes, and authentication failure event entities for all the three occasions are stored in the database.
       | Therefore, it is determined that the account is locked.
   * - | (2)
     - | Authentication failure event entities for 3 occasions are stored in the database.
       | However, since authentication failure event entities are only for the two occasions in last 10 minutes, the account is determined to be "not locked".

Similarly, a working example for unlocking is described in the following figure.

.. figure:: ./images/SecureLogin_unlock.png
   :alt: Account Lockout
   :width: 60%
   :align: center

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90
  
   * - Sr. No.
     - Description
   * - | (1)
     - | Authentication with incorrect password has been attempted three times in last 10 minutes.
       | Thereafter, since the authentication failure event entity is deleted, authentication failure event entity is not stored in the database and the account is determined as "not locked".
   
Code description
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* Common part

  In this application, registration, search and deletion of authentication failure event entity for the database is commonly required to implement the functions related to account lockout.
  Therefore, the implementation of domain layer / infrastructure layer related to the authentication failure event entity is shown first.
  
  * Implementation of Entity
  
    The implementation of authentication failure event entity with user name and date and time when authentication was attempted is shown below.
  
    .. code-block:: java
  
      package org.terasoluna.securelogin.domain.model;
      
      // omitted
      
      @Data
      public class FailedAuthentication implements Serializable {
        private static final long serialVersionUID = 1L;
      
        private String username; // (1)
      
        private LocalDateTime authenticationTimestamp; // (2)
      }
      
    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - Sr. No.
         - Description
       * - | (1)
         - | User name used for authentication
       * - | (2)
         - | Date and time when authentication was attempted

  * Implementation of Repository
  
    Repository to search, register and delete authentication failure event entity is shown below.
  
    .. code-block:: java
  
      package org.terasoluna.securelogin.domain.repository.authenticationevent;
      
      // omitted
      
      public interface FailedAuthenticationRepository {
      
        int create(FailedAuthentication event); // (1)
      
        List<FailedAuthentication> findLatest(
                        @Param("username") String username, @Param("count") long count); // (2)
      
        int deleteByUsername(@Param("username") String username); // (3)
      }
    
    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - Sr. No.
         - Description
       * - | (1)
         - | A method to register \ ``FailedAuthentication``\  object that is assigned as an argument, as a record in the database
       * - | (2)
         - | A method to get specified number of \ ``FailedAuthentication``\  objects in a new sequence by considering user name assigned as an argument, as the key
       * - | (3)
         - | A method to delete the authentication failure event entity records collectively by considering user name assigned as an argument, as the key
    
    Mapping file is as below.
  
    .. code-block:: xml
    
      <?xml version="1.0" encoding="UTF-8"?>
      <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
     
      <mapper
        namespace="org.terasoluna.securelogin.domain.repository.authenticationevent.FailedAuthenticationRepository">
      
        <resultMap id="failedAuthenticationResultMap"
                type="FailedAuthentication">
                <id property="username" column="username" />
                <id property="authenticationTimestamp" column="authentication_timestamp" />
        </resultMap>
      
        <insert id="create" parameterType="FailedAuthentication">
          <![CDATA[
              INSERT INTO failed_authentication (
                  username,
                  authentication_timestamp
              ) VALUES (
                #{username},
                  #{authenticationTimestamp}
              )
          ]]>
        </insert>
      
        <select id="findLatest" resultMap="failedAuthenticationResultMap">
             <![CDATA[
                  SELECT
                      username,
                      authentication_timestamp
                  FROM
                      failed_authentication
                  WHERE
                      username = #{username}
                  ORDER BY authentication_timestamp DESC
                  LIMIT #{count}
             ]]>
        </select>
      
        <delete id="deleteByUsername">
           <![CDATA[
                DELETE FROM
                    failed_authentication
                WHERE
                    username = #{username}
           ]]>
        </delete>
      </mapper>
      
  * Implementation of Service
  
    The service to call the method of the created Repository is defined as below.
  
    .. code-block:: java

       package org.terasoluna.securelogin.domain.service.authenticationevent;

       // omitted

       @Service
       @Transactional
       public class AuthenticationEventSharedServiceImpl implements
                       AuthenticationEventSharedService {

           // omitted

           @Inject
           ClassicDateFactory dateFactory;
           
           @Inject
           FailedAuthenticationRepository failedAuthenticationRepository;

           @Inject
           AccountSharedService accountSharedService;

           @Transactional(readOnly = true)
           @Override
           public List<FailedAuthentication> findLatestFailureEvents(
                           String username, int count) {
                   return failedAuthenticationRepository.findLatestEvents(username, count);
           }


           @Transactional(propagation = Propagation.REQUIRES_NEW)
           @Override
           public void authenticationFailure(String username) { // (1)
                if (accountSharedService.exists(username)){
                    FailedAuthentication failureEvents = new FailedAuthentication();
                    failureEvents.setUsername(username);
                    failureEvents.setAuthenticationTimestamp(dateFactory.newTimestamp()
                            .toLocalDateTime());
                
                    failedAuthenticationRepository.create(failureEvents);
                }
            }

           @Override
           public int deleteFailureEventByUsername(String username) {
                   return failedAuthenticationRepository.deleteByUsername(username);
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
         - | A method to create authentication failure event entity and register in the database.
           | If account of the user name received as an argument does not exist, skip the process of registration to the database since it violates the foreign key constraints of the database.
           | Since it is likely that authentication failure event entity is not registered by the exception after executing this method, \ ``REQUIRES_NEW`` \  is specified in the propagation method of transaction.
           
The code implemented according to the implementation method is described below sequentially.

* Storing authentication failure event entity

  Use \ ``@EventListener`` \  annotation to execute the process by handling the event generated at the time of authentication failure.
  For handling of event by using \ ``@EventListener`` \  annotation, refer :ref:`SpringSecurityAuthenticationEvent` .

  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.account;

     // omitted

     @Component
     public class AccountAuthenticationFailureBadCredentialsEventListener{ 

         @Inject
         AuthenticationEventSharedService authenticationEventSharedService;

         @EventListener // (1)
         public void onApplicationEvent(
                         AuthenticationFailureBadCredentialsEvent event) {

             String username = (String) event.getAuthentication().getPrincipal(); // (2)

             authenticationEventSharedService.authenticationFailure(username); // (3)
         }

     }
         
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - Sr. No.
       - Description
     * - | (1)
       - | By assigning \ ``@EventListener`` \  annotation, when authentication fails due to invalid authentication information such as incorrect password etc., \ ``onApplicationEvent`` \  method is executed.
     * - | (2)
       - | Fetch the user name used for authentication from \ ``AuthenticationFailureBadCredentialsEvent`` \  object.
     * - | (3)
       - | Call the process to create authentication failure event entity and register in the database.

* Determining lockout status

  The process to determine account lockout status using authentication failure event entity is described.

  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.account;

     // omitted

     @Service
     @Transactional
     public class AccountSharedServiceImpl implements AccountSharedService {

         // omitted

         @Inject
         ClassicDateFactory dateFactory;

         @Inject
         AuthenticationEventSharedService authenticationEventSharedService;

         @Value("${security.lockingDurationSeconds}") // (1)
         int lockingDurationSeconds;

         @Value("${security.lockingThreshold}") // (2)
         int lockingThreshold;

         @Transactional(readOnly = true)
         @Override
         public boolean isLocked(String username) {
             List<FailedAuthentication> failureEvents = authenticationEventSharedService
                             .findLatestFailureEvents(username, lockingThreshold); // (3)

             if (failureEvents.size() < lockingThreshold) { // (4)
                 return false;
             }

             if (failureEvents
                     .get(lockingThreshold - 1) // (5)
                     .getAuthenticationTimestamp()
                     .isBefore(
                             dateFactory.newTimestamp().toLocalDateTime()
                             .minusSeconds(lockingDurationSeconds))) {
                 return false;
             }

             return true;
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
       - | Specify the lockout duration in seconds. The value defined in Property file is injected.
     * - | (2)
       - | Specify the locking threshold. The account is locked when authentication fails only for the number of times specified here. The value defined in Property file is injected.
     * - | (3)
       - | Fetch the authentication failure event entity in new sequence only for the number same as the locking threshold.
     * - | (4)
       - | If the number of fetched authentication failure event entities is less than the locking threshold value, determine that the account is not locked.
     * - | (5)
       - | If the difference between oldest authentication failure time from the fetched authentication failure event entities and current time is greater than the locking duration, determine that the account is not locked.

  | In ``org.springframework.security.core.userdetails.User`` \  which is the implementation class of \ ``UserDetails`` \ , lockout status can be passed to the constructor.
  | In this application, class that inherits \ ``User`` \  and class that implements \ ``org.springframework.security.core.userdetails.UserDetailsService`` \  are used as shown below.

  .. code-block:: java
  
     package org.terasoluna.securelogin.domain.service.userdetails;

     // omitted

     public class LoggedInUser extends User {

        // omitted

        private final Account account;

        public LoggedInUser(Account account, boolean isLocked,
                        LocalDateTime lastLoginDate, List<SimpleGrantedAuthority> authorities) {
            super(account.getUsername(), account.getPassword(), true, true, true,
                        !isLocked, authorities); // (1)
            this.account = account;

            // omitted
        }

         public Account getAccount() {
             return account;
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
       - | In the constructor of \ ``User`` \  which is the parent class, pass ** Whether the account is locked** in truth-value. Note that it is necessary to pass true if the account is not locked.

  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.userdetails;

     // omitted

     @Service
     public class LoggedInUserDetailsService implements UserDetailsService {

         @Inject
         AccountSharedService accountSharedService;

         @Transactional(readOnly = true)
         @Override
         public UserDetails loadUserByUsername(String username)
                 throws UsernameNotFoundException {
             try {
                Account account = accountSharedService.findOne(username);
                List<SimpleGrantedAuthority> authorities = new ArrayList<>();
                for (Role role : account.getRoles()) {
                    authorities.add(new SimpleGrantedAuthority("ROLE_"
                            + role.getRoleValue()));
                }
                return new LoggedInUser(account,
                        accountSharedService.isLocked(username), // (1)
                        accountSharedService.getLastLoginDate(username),
                        authorities);
             } catch (ResourceNotFoundException e) {
                 throw new UsernameNotFoundException("user not found", e);
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
       - | In constructor of \ ``LoggedInUser`` \ , pass the determination result of lockout status using \ ``isLocked`` \  method.

  Settings to use the created \ ``UserDetailsService`` \  are as follows:

  **spring-security.xml**

  .. code-block:: xml

    <!-- omitted -->
  
    <sec:authentication-manager>
        <sec:authentication-provider
            user-service-ref="loggedInUserDetailsService"> <!-- (1) -->
            <sec:password-encoder ref="passwordEncoder" />
        </sec:authentication-provider>
    </sec:authentication-manager>
    
    <!-- omitted -->
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | Specify Bean id for \ ``UserDetailsService`` \ .

* Deleting authentication failure event entity

  * Deleting authentication failure event entity when authentication is successful

    Since only consecutive authentication failures are used to determine lockout, delete the authentication failure event entity of the account when authentication is successful.
    Create the method to be executed when authentication is successful in the Service created as a common part.

    .. code-block:: java

       package org.terasoluna.securelogin.domain.service.authenticationevent;

       // omitted

       @Service
       @Transactional
       public class AuthenticationEventSharedServiceImpl implements
                       AuthenticationEventSharedService {

           // omitted

           @Transactional(propagation = Propagation.REQUIRES_NEW)
           @Override
           public void authenticationSuccess(String username) {

               // omitted

               deleteFailureEventByUsername(username); // (1)
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
         - | Delete the authentication failure event entity for the account of the user name passed as an argument.


    Use \ ``@EventListener`` \  annotation to execute the process by handling the event generated when authentication is successful.

    .. code-block:: java

       package org.terasoluna.securelogin.domain.service.account;

       // omitted

       @Component
       public class AccountAuthenticationSuccessEventListener{ 

           @Inject
           AuthenticationEventSharedService authenticationEventSharedService;

           @EventListener // (1)
           public void onApplicationEvent(
                           AuthenticationSuccessEvent event) {

               LoggedInUser details = (LoggedInUser) event.getAuthentication()
                       .getPrincipal();

               authenticationEventSharedService.authenticationSuccess(details.getUsername()); // (2)

           }

       }
           
    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - Sr. No.
         - Description
       * - | (1)
         - | By assigning \ ``@EventListener`` \  annotation, \ ``onApplicationEvent`` \  method is executed when authentication is successful.
       * - | (2)
         - | Fetch user name from \ ``AuthenticationSuccessEvent`` \  and call the process to delete authentication failure event entity.
    
    
  * Unlocking

    Since authentication failure event entity is used to determine lockout status, an account can be unlocked by deleting the authentication failure event entity.
    Perform the authorization settings to restrict the usage of unlock function to the user having administrator rights and implement the domain layer / application layer.

    * Authorization settings

      Set the rights for the user who can unlock an account as below.

      **spring-security.xml**

      .. code-block:: xml

        <!-- omitted -->

          <sec:http pattern="/resources/**" security="none" />
          <sec:http>
          
              <!-- omitted -->
              
              <sec:intercept-url pattern="/unlock/**" access="hasRole('ADMIN')" /> <!-- (1) -->
              
              <!-- omitted -->
              
          </sec:http>

        <!-- omitted -->

      .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
      .. list-table::
         :header-rows: 1
         :widths: 10 90
    
         * - Sr. No.
           - Description
         * - | (1)
           - | Restrict the access rights for URL under /unlock to the administrator.

    * Implementation of Service

      .. code-block:: java

         package org.terasoluna.securelogin.domain.service.unlock;

         // omitted

         @Transactional
         @Service
         public class UnlockServiceImpl implements UnlockService {

             @Inject
             AccountSharedService accountSharedService;

             @Inject
             AuthenticationEventSharedService authenticationEventSharedService;

             @Override
             public void unlock(String username) {
                 authenticationEventSharedService
                        .deleteFailureEventByUsername(username); // (1)
             }

         }
        
      .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
      .. list-table::
         :header-rows: 1
         :widths: 10 90
    
         * - Sr. No.
           - Description
         * - | (1)
           - | Unlock an account by deleting the authentication failure event entity.

    * Implementation of Form

      .. code-block:: java

        package org.terasoluna.securelogin.app.unlock;    

        @Data
        public class UnlockForm implements Serializable {

            private static final long serialVersionUID = 1L;

            @NotEmpty
            private String username;
        }
        
    * Implementation of View

      **Top screen(home.jsp)**

      .. code-block:: jsp

        <!-- omitted -->

        <body>
            <div id="wrapper">

                <!-- omitted -->        

                <sec:authorize url="/unlock"> <!-- (1) -->
                <div>
                    <a id="unlock" href="${f:h(pageContext.request.contextPath)}/unlock?form">
                        Unlock Account
                    </a>
                </div>
                </sec:authorize>

                <!-- omitted -->

            </div>
        </body>

        <!-- omitted -->

      .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
      .. list-table::
         :header-rows: 1
         :widths: 10 90
    
         * - Sr. No.
           - Description
         * - | (1)
           - | Display only for the user who has access rights, under /unlock.

      **Unlock form(unlokcForm.jsp)**

      .. code-block:: jsp
      
        <!-- omitted -->

        <body>
            <div id="wrapper">
                <h1>Unlock Account</h1>
                <t:messagesPanel />
                <form:form action="${f:h(pageContext.request.contextPath)}/unlock"
                    method="POST" modelAttribute="unlockForm">
                    <table>
                        <tr>
                            <th><form:label path="username" cssErrorClass="error-label">Username</form:label>
                            </th>
                            <td><form:input path="username" cssErrorClass="error-input" /></td>
                            <td><form:errors path="username" cssClass="error-messages" /></td>
                        </tr>
                    </table>

                    <input id="submit" type="submit" value="Unlock" />
                </form:form>
                <a href="${f:h(pageContext.request.contextPath)}/">go to Top</a>
            </div>
        </body>

        <!-- omitted -->

      **Unlock completion screen(unlockComplete.jsp)**

      .. code-block:: jsp

        <!-- omitted -->

        <body>
            <div id="wrapper">
                  <h1>${f:h(username)}'s account was successfully unlocked.</h1>
                  <a href="${f:h(pageContext.request.contextPath)}/">go to Top</a>
            </div>
        </body>
        
        <!-- omitted -->

    * Implementation of Controller

      .. code-block:: java

         package org.terasoluna.securelogin.app.unlock;

         // omitted

         @Controller
         @RequestMapping("/unlock") // (1)
         public class UnlockController {

             @Inject
             UnlockService unlockService;

             @RequestMapping(params = "form")
             public String showForm(UnlockForm form) {
                 return "unlock/unlockForm";
             }

             @RequestMapping(method = RequestMethod.POST)
             public String unlock(@Validated UnlockForm form,
                     BindingResult bindingResult, Model model,
                     RedirectAttributes attributes) {
                 if (bindingResult.hasErrors()) {
                         return showForm(form);
                 }

                 try {
                     unlockService.unlock(form.getUsername()); // (2)
                     attributes.addFlashAttribute("username", form.getUsername());
                     return "redirect:/unlock?complete";
                 } catch (BusinessException e) {
                     model.addAttribute(e.getResultMessages());
                     return showForm(form);
                 }
             }

             @RequestMapping(method = RequestMethod.GET, params = "complete")
             public String unlockComplete() {
                 return "unlock/unlockComplete";
             }

         }

      .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
      .. list-table::
         :header-rows: 1
         :widths: 10 90
    
         * - Sr. No.
           - Description
         * - | (1)
           - | Map to the URL under /unlock. It can be accessed by administrator only depending on the authorization settings.
         * - | (2)
           - | Call the process to unlock an account by considering the user name obtained from the Form, as an argument.

.. _last-login:

Display the date and time of last login
--------------------------------------------------------------------------------
List of requirements to be implemented
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
* :ref:`Display of previous login date and time <sec-requirements>`

Working image
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. figure:: ./images/SecureLogin_last_login.png
   :alt: Last Login Date
   :width: 80%
   :align: center

Implementation method
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| In this application, the history when authentication is successful is stored in the database as an "authentication successful event" entity, and date and time of previous login for the account is displayed on the top screen using this authentication successful event entity.
| In particular, fulfil the requirements by implementing the following two processes.

* Storing authentication successful event entity

  Handle the event generated by Spring Security when authentication is successful and register the username used for authentication and the date and time when authentication was successful in the database, as an authentication successful event entity.

* Fetch and display date and time of previous login

  At the time of authentication, fetch the latest authentication successful event entity in the account from the database, fetch the authentication successful date and time from the event entity and set in \ ``org.springframework.security.core.userdetails.UserDetails`` \ .
  Format, pass and display the authentication successful date and time retained by \ ``UserDetails`` \  in jsp.

Code description
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* Common part

  In this application, the authentication successful event entity must be registered and searched for the database in order to display previous login date and time.
  Therefore, the implementation of domain layer / infrastructure layer related to the authentication successful event entity is described first.
  
  * Implementation of Entity
  
    The implementation of authentication successful event entity with user name and date and time when authentication was successful is as below.
  
    .. code-block:: java
  
       package org.terasoluna.securelogin.domain.model;

       // omitted

       @Data
       public class SuccessfulAuthentication implements Serializable {

           private static final long serialVersionUID = 1L;

           private String username; // (1)

           private LocalDateTime authenticationTimestamp; // (2)

       }
    
    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - Sr. No.
         - Description
       * - | (1)
         - | User name used for authentication
       * - | (2)
         - | Date and time when authentication is attempted

  * Implementation of Repository
  
    Repository to search and register authentication successful event entity is shown below.
  
    .. code-block:: java
                  
       package org.terasoluna.securelogin.domain.repository.authenticationevent;

       // omitted

       public interface SuccessfulAuthenticationRepository {

           int create(SuccessfulAuthentication event); // (1)

           List<SuccessfulAuthentication> findLatestEvents(
                  @Param("username") String username, @Param("count") long count); // (2)
       }
      
    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - Sr. No.
         - Description
       * - | (1)
         - | A method to register \ ``SuccessfulAuthentication``\  object that is assigned as an argument, as a record in the database
       * - | (2)
         - | A method to fetch specified number of \ ``SuccessfulAuthentication``\  objects in new sequence by considering user name assigned as an argument, as a key
  
    Mapping file is as below.
  
    .. code-block:: xml
  
       <?xml version="1.0" encoding="UTF-8"?>
       <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

       <mapper
           namespace="org.terasoluna.securelogin.domain.repository.authenticationevent.SuccessfulAuthenticationRepository">

           <resultMap id="successfulAuthenticationResultMap"
                   type="SuccessfulAuthentication">
               <id property="username" column="username" />
               <id property="authenticationTimestamp" column="authentication_timestamp" />
           </resultMap>

           <insert id="create" parameterType="SuccessfulAuthentication">
           <![CDATA[
               INSERT INTO successful_authentication (
                   username,
                   authentication_timestamp
               ) VALUES (
                   #{username},
                   #{authenticationTimestamp}
               )
           ]]>
           </insert>

           <select id="findLatestEvents" resultMap="successfulAuthenticationResultMap">
           <![CDATA[
               SELECT
                   username,
                   authentication_timestamp
               FROM
                   successful_authentication
               WHERE
                   username = #{username}
               ORDER BY authentication_timestamp DESC
               LIMIT #{count}
           ]]>
           </select>
       </mapper>
      
  * Implementation of Service
  
    The service to call the methods of the created Repository is shown below.
  
    .. code-block:: java
    
       package org.terasoluna.securelogin.domain.service.authenticationevent;

       // omitted

       @Service
       @Transactional
       public class AuthenticationEventSharedServiceImpl implements
       		AuthenticationEventSharedService {

           // omitted
           
           @Inject
           ClassicDateFactory dateFactory;

           @Inject
           SuccessfulAuthenticationRepository successAuthenticationRepository;

           @Transactional(readOnly = true)
           @Override
           public List<SuccessfulAuthentication> findLatestSuccessEvents(
                           String username, int count) {
               return successAuthenticationRepository.findLatestEvents(username, count);
           }

           @Transactional(propagation = Propagation.REQUIRES_NEW)
           @Override
             public void authenticationSuccess(String username) {
                 SuccessfulAuthentication successEvent = new SuccessfulAuthentication();
                 successEvent.setUsername(username);
                 successEvent.setAuthenticationTimestamp(dateFactory.newTimestamp().toLocalDateTime());

                 successAuthenticationRepository.create(successEvent);
                 deleteFailureEventByUsername(username);
             }

       }
  
The code implemented according to the implementation method is described below sequentially.

* Storing authentication successful event entity

  Use \ ``@EventListener`` \  annotation to execute the process by handling the event generated when authentication is successful.

  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.account;

     // omitted

     @Component
     public class AccountAuthenticationSuccessEventListener{

         @Inject
         AuthenticationEventSharedService authenticationEventSharedService;

         @EventListener // (1)
         public void onApplicationEvent(AuthenticationSuccessEvent event) {
             LoggedInUser details = (LoggedInUser) event.getAuthentication()
                             .getPrincipal(); // (2)

             authenticationEventSharedService.authenticationSuccess(details.getUsername()); // (3)
         }

     }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - Sr. No.
       - Description
     * - | (1)
       - | By assigning \ ``@EventListener`` \  annotation, \ ``onApplicationEvent`` \  method is executed when authentication is successful.
     * - | (2)
       - | Fetch implementation class of \ ``UserDetails`` \  from \ ``AuthenticationSuccessEvent`` \  object. This class is described later.
     * - | (3)
       - | Call the process to create authentication successful event entity and register in the database.

* Fetch and display date and time of previous login

  The Service to fetch date and time of previous login from authentication successful event entity is shown below.

   .. code-block:: java

      package org.terasoluna.securelogin.domain.service.account;

      // omitted

      @Service
      @Transactional
      public class AccountSharedServiceImpl implements AccountSharedService {

          // omitted

          @Inject
          AuthenticationEventSharedService authenticationEventSharedService;

          @Transactional(readOnly = true)
          @Override
          public LocalDateTime getLastLoginDate(String username) {
              List<SuccessfulAuthentication> events = authenticationEventSharedService
                          .findLatestSuccessEvents(username, 1); // (1)

              if (events.isEmpty()) {
                  return null; // (2)
              } else {
                  return events.get(0).getAuthenticationTimestamp(); // (3)
              }
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
       - | Fetch one record of the latest authentication successful event entity by considering the user name assigned as an argument, as the key.
     * - | (2)
       - | Return null if even a single record of authentication successful event entity could not be fetched at the time of initial login.
     * - | (3)
       - | Fetch and return authentication date and time from authentication successful event entity.

  Create a class that inherits \ ``User`` \  and a class that implements \ ``UserDetailsService`` \  as shown below to fetch the date and time of previous login and retain it in \ ``UserDetails`` \  at the time of login.

  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.userdetails;

     // omitted

     public class LoggedInUser extends User {

         private final Account account;

         private final LocalDateTime lastLoginDate; // (1)

         public LoggedInUser(Account account, boolean isLocked,
                         LocalDateTime lastLoginDate, List<SimpleGrantedAuthority> authorities) {

             super(account.getUsername(), account.getPassword(), true, true, true,
                             !isLocked, authorities);
             this.account = account;
             this.lastLoginDate = lastLoginDate; // (2)
         }

         // omitted    

         public LocalDateTime getLastLoginDate() { // (3)
             return lastLoginDate;
         }

     }
    
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - Sr. No.
       - Description
     * - | (1)
       - | Declare a field to retain the date and time of previous login.
     * - | (2)
       - | Set the date and time of previous login assigned as an argument in the field.
     * - | (3)
       - | A method to return the retained date and time of previous login

  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.userdetails;

     // omitted

     @Service
     public class LoggedInUserDetailsService implements UserDetailsService {

         @Inject
         AccountSharedService accountSharedService;

         @Transactional(readOnly = true)
         @Override
         public UserDetails loadUserByUsername(String username)
                     throws UsernameNotFoundException {
             try {
                 Account account = accountSharedService.findOne(username);
                 List<SimpleGrantedAuthority> authorities = new ArrayList<>();
                 for (Role role : account.getRoles()) {
                         authorities.add(new SimpleGrantedAuthority("ROLE_"
                                         + role.getRoleValue()));
                 }
                 return new LoggedInUser(account,
                                 accountSharedService.isLocked(username),
                                 accountSharedService.getLastLoginDate(username), // (1)
                                 authorities);
             } catch (ResourceNotFoundException e) {
                 throw new UsernameNotFoundException("user not found", e);
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
       - | Fetch date and time of previous login by calling Service method and pass it to constructor of \ ``LoggedInUser`` \ .

  Implement application layer to display date and time of previous login on the top screen.

  .. code-block:: java

     package org.terasoluna.securelogin.app.welcome;

     // omitted

     @Controller
     public class HomeController {

     	@Inject
     	AccountSharedService accountSharedService;

     	@RequestMapping(value = "/", method = { RequestMethod.GET,
     			RequestMethod.POST })
     	public String home(@AuthenticationPrincipal LoggedInUser userDetails, // (1)
     			Model model) {

            // omitted
     		
     		LocalDateTime lastLoginDate = userDetails.getLastLoginDate(); // (2)
     		if (lastLoginDate != null) {
     			model.addAttribute("lastLoginDate", lastLoginDate
     					.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"))); // (3)
     		}
     		
     		return "welcome/home";

     	}

     }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - Sr. No.
       - Description
     * - | (1)
       - | Fetch UserDetails object by using \ ``@AuthenticationPrincipal`` \ .
     * - | (2)
       - | Fetch date and time of last login from \ ``LoggedInUserDetails`` \ .
     * - | (3)
       - | Format date and time of last login, set it in Model and pass to View.

  **Top screen(home.jsp)**

  .. code-block:: jsp

    <body>
      <div id="wrapper">

          <!-- omitted -->

          <c:if test="${!empty lastLoginDate}"> <!-- (1) -->
              <p id="lastLogin">
                  Last login date is ${f:h(lastLoginDate)}. <!-- (2) -->
              </p>
          </c:if>

          <!-- omitted -->

      </div>
    </body>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - Sr. No.
       - Description
     * - | (1)
       - | Do not display if date and time of previous login is null.
     * - | (2)
       - | Display the date and time of previous login passed from the Controller.
      
.. _reissue-info-create:

Creating authentication information for password reissue
--------------------------------------------------------------------------------
List of requirements to be implemented
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
* :ref:`Assign random string to the URL for password reissue <sec-requirements>`
* :ref:`Issue confidential information for password reissue <sec-requirements>`
  
Working image
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. figure:: ./images/SecureLogin_password_reissue_generate.png
   :alt: Generate Password Reissue Information 
   :width: 80%
   :align: center

Enter the user name for which password is to be reissued on the screen to generate authentication information for password reissue. At this time, the confidential information and token to be used for authentication during password reissue, are generated.
Confidential information is displayed on the screen and the URL for password reissue screen containing the token is sent to the registered e-mail address of the user.

There is an expiry date to the URL sent by e-mail. Password can be changed by accessing the URL within the expiry date and entering the confidential information and the new password.
If the URL sent by e-mail is accessed after the expiry date, the user is taken to the error screen.

Confidential information and token generation is described from the flow mentioned above.

Implementation method
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| While reissuing the password, an alternative to password is required to verify that the user is the owner of the account.
| In this application, URL of password reissue screen and confidential information are used as the information to verify the user.
| Create a random string and add it to URL to make the password reissue screen URL unique and difficult to guess. Create confidential information which is in the form of a random string and use it for authentication as a measure against accidental leakage of URL.
| Create two random strings by different ways so that that it becomes impossible to guess a string from the other string.
| In particular, fulfil the requirements by implementing the following process.

* Creating and saving authentication information for password reissue

  Store the following information as the authentication information for password reissue in the database.

  * User name: User name of the account for which password is to be reissued
  * Token: Random string generated to make the password reissue screen URL unique and difficult to guess
  * Confidential information: Random string generated for user input at the time of password reissue
  * Expiry date: Expiry date for authentication information for password reissue

  Use \ ``randomUUID`` \  method of \ ``java.util.UUID`` \  class for token generation and Password generation function of Passay for generating confidential information.
  
  Save the confidential information to the database by hashing similar to password.
  Expiry date settings and confirmation process are described in :ref:`Validation at the time of executing password reissue <reissue-info-validate>`.
  Refer to :ref:`Distribute authentication information for password reissue <reissue-info-delivery>` for the method to distribute authentication information for password reissue to the user.

Code description
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* Common part

  In the implementation according to the implementation method mentioned above, the process to register and search the authentication information for password reissue in the database is commonly required.
  Therefore, implementation of Entity and Repository related to the authentication information for password reissue is described first.

  * Creation of Entity

    Create an Entity of authentication information for password reissue.

    .. code-block:: java

       package org.terasoluna.securelogin.domain.model;

       // omitted

       @Data
       public class PasswordReissueInfo {

           private String username; // (1)

           private String token; // (2)

           private String secret; // (3)

           private LocalDateTime expiryDate; // (4)

       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - Sr. No.
         - Description
       * - | (1)
         - | User name for password reissue
       * - | (2)
         - | String that is generated to be included in the URL for password reissue (Token）
       * - | (3)
         - | String to verify the user at the time of password reissue (Confidential information）
       * - | (2)
         - | Expiry date for authentication information for password reissue
           
  * Implementation of Repository

    Repository to search, register and delete the authentication information for password reissue is shown below.

    .. code-block:: java

       package org.terasoluna.securelogin.domain.repository.passwordreissue;

       // omitted

       public interface PasswordReissueInfoRepository {

           void create(PasswordReissueInfo info); // (1)

           PasswordReissueInfo findOne(@Param("token") String token); // (2)

           int delete(@Param("token") String token); // (3)

           // omitted

       }

   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 90
   
      * - Sr. No.
        - Description
      * - | (1)
        - | A method to register \ ``PasswordReissueInfo``\  object that is assigned as an argument, as a record in the database
      * - | (2)
        - | A method to search and fetch \ ``PasswordReissueInfo``\  object by considering the token assigned as an argument, as the key
      * - | (3)
        - | A method to delete \ ``PasswordReissueInfo``\  object by considering the token assigned as an argument, as the key

   Mapping file is as below.

   .. code-block:: xml

      <?xml version="1.0" encoding="UTF-8"?>
      <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

      <mapper
          namespace="org.terasoluna.securelogin.domain.repository.passwordreissue.PasswordReissueInfoRepository">

          <resultMap id="PasswordReissueInfoResultMap" type="PasswordReissueInfo">
              <id property="username" column="username" />
              <id property="token" column="token" />
              <id property="secret" column="secret" />
              <id property="expiryDate" column="expiry_date" />
          </resultMap>

          <select id="findOne" resultMap="PasswordReissueInfoResultMap">
          <![CDATA[
              SELECT
                  username,
                  token,
                  secret,
                  expiry_date
              FROM
                  password_reissue_info
              WHERE
                  token = #{token}
          ]]>
          </select>

          <insert id="create" parameterType="PasswordReissueInfo">
          <![CDATA[
              INSERT INTO password_reissue_info (
                  username,
                  token,
                  secret,
                  expiry_date
              ) VALUES (
                  #{username},
                  #{token},
                  #{secret},
                  #{expiryDate}
              )
          ]]>
          </insert>

          <delete id="delete">
          <![CDATA[
              DELETE FROM
                  password_reissue_info
              WHERE
                  token = #{token}
          ]]>
          </delete>

          <!-- omitted -->

      </mapper>

The code implemented according to the implementation method is described below.

* Generating and storing authentication information for password reissue

  * Definition of password generator

    The definition of password generator and generation rules to use the password generation function of Passay is shown below.
    Refer to :ref:`password_generation` for the password generator and generation rules.

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - Sr. No.
         - Description
       * - | (1)
         - | Define a Bean for password generator to be used in password generation function of Passay
       * - | (2)
         - | Define a Bean for password generation rules to be used in password generation function of Passay. Using the validation rules that were used in :ref:`password-strength`, define generation rules for the password containing one or more characters of single-byte upper case letters, single-byte lower case letters and single-byte digits respectively.

    **applicationContext.xml**

    .. code-block:: xml

       <bean id="passwordGenerator" class="org.passay.PasswordGenerator" /> <!-- (1) -->
       <util:list id="passwordGenerationRules">
           <ref bean="upperCaseRule" />
           <ref bean="lowerCaseRule" />
           <ref bean="digitRule" />
       </util:list>

  * Implementation of Service

    The implementation of the process to create authentication information for password reissue and store in the database is shown below. The authentication information generated in this process is sent by e-mail. Sending the information by e-mail is omitted here as it is described later.

    .. code-block:: java

       package org.terasoluna.securelogin.domain.service.passwordreissue;

       // omitted

       @Service
       @Transactional
       public class PasswordReissueServiceImpl implements PasswordReissueService {

           @Inject
           ClassicDateFactory dateFactory;

           @Inject
           PasswordReissueInfoRepository passwordReissueInfoRepository;

           @Inject
           AccountSharedService accountSharedService;

           @Inject
           PasswordEncoder passwordEncoder;

           @Inject
           PasswordGenerator passwordGenerator; // (1)

           @Resource(name = "passwordGenerationRules")
           List<CharacterRule> passwordGenerationRules; //(2)

           @Value("${security.tokenLifeTimeSeconds}")
           int tokenLifeTimeSeconds; // (3)

           // omitted

           @Override
           public String createAndSendReissueInfo(String username) {
               
               String rowSecret = passwordGenerator.generatePassword(10, passwordGenerationRules); // (4)

               if(!accountSharedService.exists(username)){ // (5)
                   return rowSecret;           
               }
               
               Account account= accountSharedService.findOne(username); // (6)
               
               String token = UUID.randomUUID().toString(); // (7)

               LocalDateTime expiryDate = dateFactory.newTimestamp().toLocalDateTime()
                       .plusSeconds(tokenLifeTimeSeconds); // (8)

               PasswordReissueInfo info = new PasswordReissueInfo(); // (9)
               info.setUsername(username);
               info.setToken(token);
               info.setSecret(passwordEncoder.encode(rowSecret)); // (10)
               info.setExpiryDate(expiryDate);

               passwordReissueInfoRepository.create(info); // (11)

               // omitted (Send E-Mail)

               return rowSecret; // (12)

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
         - | Inject a password generator to be used in password generation function of Passay.
       * - | (2)
         - | Inject password generation rules to be used in password generation function of Passay.
       * - | (3)
         - | Specify the length of the period for which authentication information for password reissue is valid, in seconds. The value defined in the property file is injected.
       * - | (4)
         - | Create a random string of length 10 in accordance with the password generation rules using the password generation function of Passay to use as confidential information.
       * - | (5)
         - | Check whether the account of user name that is passed as an argument, exists. If it does not exist, return dummy confidential information since non-existence of the user is not known.
       * - | (6)
         - | Fetch the account information of the user name included in the authentication information for password reissue.
       * - | (7)
         - | Create a random string using \ ``randomUUID`` \  method of \ ``java.util.UUID`` \  class to use as a token.
       * - | (8)
         - | By adding the value of (3) to current time, calculate expiry date for the authentication information for password reissue.
       * - | (9)
         - | Create authentication information for password reissue and set the user name, token, confidential information and expiry date.
       * - | (10)
         - | Set confidential information in \ ``PasswordReissueInfo`` \  after hashing it.
       * - | (11)
         - | Register the authentication information for password reissue in the database.
       * - | (12)
         - | Return the created confidential information.

  * Implementation of Form

    .. code-block:: java

       package org.terasoluna.securelogin.app.passwordreissue;

       // omitted

       @Data
       public class CreateReissueInfoForm implements Serializable {

           private static final long serialVersionUID = 1L;
       
           @NotEmpty
           private String username;
       }

  * Implementation of View

    **Screen to create authentication information for password reissue(createReissueInfoForm.xml)**

    .. code-block:: jsp

       <!-- omitted -->

       <body>
           <div id="wrapper">
               <h1>Reissue password</h1>
               <t:messagesPanel />
               <form:form
                   action="${f:h(pageContext.request.contextPath)}/reissue/create"
                   method="POST" modelAttribute="createReissueInfoForm">
                   <table>
                       <tr>
                           <th><form:label path="username" cssErrorClass="error-label">Username</form:label>
                           </th>
                           <td><form:input path="username" cssErrorClass="error-input" /></td>
                           <td><form:errors path="username" cssClass="error-messages" /></td>
                       </tr>
                   </table>

                   <input id="submit" type="submit" value="Reissue password" />
               </form:form>
           </div>
       </body>

       <!-- omitted -->

  * Implementation of Controller

    .. code-block:: java

       package org.terasoluna.securelogin.app.passwordreissue;

       // omitted

       @Controller
       @RequestMapping("/reissue")
       public class PasswordReissueController {

           @Inject
           PasswordReissueService passwordReissueService;

           @RequestMapping(value = "create", params = "form")
           public String showCreateReissueInfoForm(CreateReissueInfoForm form) {
               return "passwordreissue/createReissueInfoForm";
           }

           @RequestMapping(value = "create", method = RequestMethod.POST)
           public String createReissueInfo(@Validated CreateReissueInfoForm form,
                   BindingResult bindingResult, Model model,
                   RedirectAttributes attributes) {
               if (bindingResult.hasErrors()) {
                   return showCreateReissueInfoForm(form);
               }

               String rawSecret = passwordReissueService.createAndSendReissueInfo(form.getUsername()); // (1)
               attributes.addFlashAttribute("secret", rawSecret);
               return "redirect:/reissue/create?complete";
           }

           @RequestMapping(value = "create", params = "complete", method = RequestMethod.GET)
           public String createReissueInfoComplete() {
               return "passwordreissue/createReissueInfoComplete";
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
         - | Create authentication information for password reissue from the user name fetched from Form and call the process registered in the database.


.. _reissue-info-delivery:

Distribution of authentication information for password reissue
--------------------------------------------------------------------------------
List of requirements to be implemented
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
* :ref:`Separate distribution for password reissue screen URL and confidential information <sec-requirements>`
* :ref:`Send e-mail for URL of password reissue screen <sec-requirements>`
  
Working image
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. figure:: ./images/SecureLogin_password_reissue_give.png
   :alt: Give Password Reissue Information 
   :width: 80%
   :align: center

Creation of authentication information for password reissue is described in :ref:`reissue-info-create`.
The distribution of the created authentication information is described here.

Perform the authentication for password reissue using the password reissue screen URL and confidential information.
Distribute the information to the user using different methods to prevent the leakage of the information at the same time.
In this application, URL of the password reissue screen is sent to the registered e-mail address of the user, and confidential information is displayed on the screen.

Implementation method
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Split the authentication information created using :ref:`Create authentication information for password reissue <reissue-info-create>` and distribute to the user using separate methods.
| Fulfil the requirements by implementing and using the following two processes.

* Display confidential information on screen

  Distribute the confidential information before hashing that is created using :ref:`Create authentication information for password reissue <reissue-info-create>` to the user by displaying it on the screen.

* Send an e-mail for URL of password reissue screen

  Send the URL of password reissue screen including token that is created using :ref:`Create authentication information for password reissue <reissue-info-create>` by e-mail using the component for Spring Framework Mail linkage.
  

Code description
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The code implemented according to the above implementation method is described sequentially.

* Displaying confidential information on screen

  A series of implementations to call the process to create confidential information from the Controller and display it in View is shown below.

  .. code-block:: java

     package org.terasoluna.securelogin.app.passwordreissue;

     // omitted

     @Controller
     @RequestMapping("/reissue")
     public class PasswordReissueController {

         @Inject
         PasswordReissueService passwordReissueService;

         // omitted

         @RequestMapping(value = "create", method = RequestMethod.POST)
         public String createReissueInfo(@Validated CreateReissueInfoForm form,
                 BindingResult bindingResult, Model model,
                 RedirectAttributes attributes) {
             if (bindingResult.hasErrors()) {
                 return showCreateReissueInfoForm(form);
             }

             String rawSecret = passwordReissueService.createAndSendReissueInfo(form.getUsername()); // (1)
             attributes.addFlashAttribute("secret", rawSecret); // (2)
             return "redirect:/reissue/create?complete"; // (3)
         }

         @RequestMapping(value = "create", params = "complete", method = RequestMethod.GET)
         public String createReissueInfoComplete() {
             return "passwordreissue/createReissueInfoComplete";
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
       - | Call the process to create confidential information.
     * - | (2)
       - | Using RedirectAttributes, pass the confidential information to the redirect destination.
     * - | (3)
       - | Redirect to completion screen of authentication information for password reissue.


  **Screen to complete creation of authentication information for password reissue(createReissueInfoComplete.jsp)**

  .. code-block:: jsp

     <!-- omitted -->

     <body>
         <div id="wrapper">
             <h1>Your Password Reissue URL was successfully generated.</h1>
             The URL was sent to your registered E-mail address.<br /> Please
             access the URL and enter the secret shown below.
             <h3>Secret : <span id=secret>${f:h(secret)}</span></h3> <!-- (1) -->
         </div>
     </body>

     <!-- omitted -->

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - Sr. No.
       - Description
     * - | (1)
       - | Display confidential information on the screen.

* Send a mail for URL of password reissue screen

  The implementation of the process to create URL of password reissue screen from the authentication information for password reissue screen and send it by e-mail is shown below.
  Refer to :doc:`../ArchitectureInDetail/Email` for more information about how to add dependent libraries and how to fetch an e-mail session.

  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.mail;

     // omitted

     @Service
     public class PasswordReissueMailSharedServiceImpl implements PasswordReissueMailSharedService {

         @Inject
         JavaMailSender mailSender; // (1)

         @Inject
         @Named("passwordReissueMessage")
         SimpleMailMessage templateMessage; // (2)

         // omitted

         @Override
         public void send(String to, String text) {
             SimpleMailMessage message = new SimpleMailMessage(templateMessage); // (3)
             message.setTo(to);
             message.setText(text);
             mailSender.send(message);
         }

     }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - Sr. No.
       - Description
     * - | (1)
       - | Inject a Bean for \ ``org.springframework.mail.javamail.JavaMailSender`` \ .
     * - | (2)
       - | Inject a Bean for \ ``org.springframework.mail.SimpleMailMessage`` \  in which email address of the source and e-mail title are set.
         | In this application, only one Bean is defined for \ ``SimpleMailMessage`` \ , however, since multiple Beans are generally defined as a mail template, Bean name is specified by \ ``@Named`` \ .
     * - | (3)
       - | Create an instance of \ ``SimpleMailMessage`` \  from the template, set the destination email address and text assigned as arguments and send.

  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.passwordreissue;

     // omitted

     @Service
     @Transactional
     public class PasswordReissueServiceImpl implements PasswordReissueService {

         @Inject
         ClassicDateFactory dateFactory;

         @Inject
         PasswordReissueMailSharedService mailSharedService;

         @Inject
         AccountSharedService accountSharedService;

         @Inject
         PasswordEncoder passwordEncoder;

         @Value("${security.tokenLifeTimeSeconds}")
         int tokenLifeTimeSeconds;

         @Value("${app.applicationBaseUrl}") // (1)
         String baseUrl;

         @Value("${app.passwordReissueProtocol}")
         String protocol;

         // omitted

         @Override
         public String createAndSendReissueInfo(String username) {
            
             String rowSecret = passwordGenerator.generatePassword(10, passwordGenerationRules);

             if(!accountSharedService.exists(username)){
                 return rowSecret;           
             }
            
             Account account= accountSharedService.findOne(username);
            
             String token = UUID.randomUUID().toString();

             LocalDateTime expiryDate = dateFactory.newTimestamp().toLocalDateTime()
                     .plusSeconds(tokenLifeTimeSeconds);

             PasswordReissueInfo info = new PasswordReissueInfo();
             info.setUsername(username);
             info.setToken(token);
             info.setSecret(passwordEncoder.encode(rowSecret));
             info.setExpiryDate(expiryDate);

             passwordReissueInfoRepository.create(info);

             UriComponentsBuilder uriBuilder = UriComponentsBuilder.fromUriString(baseUrl);
             uriBuilder.pathSegment("reissue").pathSegment("resetpassword")
                     .queryParam("form").queryParam("token", info.getToken());  // (2)
             String passwordResetUrl = uriBuilder.build().encode().toUriString();

             mailSharedService.send(account.getEmail(), passwordResetUrl); // (3)

             return rowSecret;

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
       - | Fetch the base URL to be used in the password reissue screen URL from the property file.
     * - | (2)
       - | Using the value fetched in (1) and the token included in the created authentication information for password reissue, create the URL of password reissue screen to be distributed to the user.
         | Use \ ``org.springframework.web.util.UriComponentsBuilder`` \  to create the URL. \ ``UriComponentsBuilder`` \  is described in :ref:`RESTAppendixHyperMediaLink`.
     * - | (3)
       - | Send an email with the URL of the password reissue screen mentioned in the mail text to the registered e-mail address of the user.


.. _reissue-info-validate:

Validation at the time of executing password reissue
--------------------------------------------------------------------------------
List of requirements to be implemented
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
* :ref:`Setting validity period for authentication information for password reissue <sec-requirements>`
  
Working image
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. figure:: ./images/SecureLogin_password_reissue_execute.png
   :alt: Use Password Reissue Information 
   :width: 80%
   :align: center

The distribution of authentication information for password reissue is described in :ref:`reissue-info-delivery`.
The process for using the distributed authentication information is described below.

As the authentication at the time of password reissue, verify the confidential information and URL of password reissue screen distributed separately in :ref:`reissue-info-delivery`.
Password is reissued only if the set of token and confidential information that is included in the URL is correct.

Moreover, set expiry date to the authentication information so that it is not valid for a long period of time unnecessarily as generally password is reissued immediately after the creation of authentication information.
When the URL for password reissue screen is accessed, password reissue screen is displayed if the authentication information is within the expiry date and transits to error screen after the expiry date.

Implementation method
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Token is included as a request parameter in the URL for password reissue screen sent by e-mail. Fetch the token when the password reissue screen is accessed and search the authentication information for password reissue from the database by considering this token as the key.
| At the time of creating authentication information, set the expiry date in advance and check for expiration when it is obtained from the database. If it is within the expiry date, display the change password screen and accept the input for confidential information and new password.
| If confidential information in the authentication information obtained from the database and the confidential information entered by the user are identical, then authentication is successful and password is reissued.
| In particular, fulfil the requirements by implementing the following three processes.

* Setting expiry date for authentication information for password reissue

  Set expiry date to the authentication information created in the process described in :ref:`reissue-info-create`.

* Check expiry date for authentication information for password reissue

  When the password reissue screen is accessed, fetch the token included in the request parameter and search the authentication information for password reissue stored in the database considering the token as the key.
  Compare the expiry date included in authentication information with current time and after expiry date, transit to the error screen.

* User verification using authentication information for password reissue

  When reissuing the password, check whether the combination of user name, token and the confidential information provided by the user and the authentication information in the database are identical.
  If they are identical, reissue the password. If they are not identical, display the error message.


Code description
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* Setting expiry date for authentication information for password reissue

  The settings for expiry date to the authentication information for password reissue are included in the process described in :ref:`reissue-info-create`. Here, only relevant implementation points are shown again.

  * Implementation of Service

    .. code-block:: java

       package org.terasoluna.securelogin.domain.service.passwordreissue;

       // omitted

       @Service
       @Transactional
       public class PasswordReissueServiceImpl implements PasswordReissueService {

           @Inject
           ClassicDateFactory dateFactory;

           @Inject
           PasswordReissueInfoRepository passwordReissueInfoRepository;

           @Value("${security.tokenLifeTimeSeconds}")
           int tokenLifeTimeSeconds; // (1)

           // omitted

           @Override
           public String createAndSendReissueInfo(String username) {
               
               // omitted

               LocalDateTime expiryDate = dateFactory.newTimestamp().toLocalDateTime()
                       .plusSeconds(tokenLifeTimeSeconds); // (2)

               PasswordReissueInfo info = new PasswordReissueInfo(); // (3)
               info.setUsername(username);
               info.setToken(token);
               info.setSecret(passwordEncoder.encode(rowSecret));
               info.setExpiryDate(expiryDate);

               passwordReissueInfoRepository.create(info); // (4)

               // omitted (Send E-Mail)

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
         - | Specify the length of the period for which authentication information for password reissue is valid, in seconds. The value defined in property file is injected.
       * - | (2)
         - | By adding the value of (1) to current time, calculate expiry date for the authentication information for password reissue.
       * - | (3)
         - | Create authentication information for password reissue and set the user name, token, confidential information and expiry date.
       * - | (4)
         - | Register the authentication information for password reissue in the database.

* Check expiry date of authentication information for password reissue

  The implementation of the process to fetch authentication information for password reissue from the token included in the URL as a request parameter and check whether it is within the expiry date when the password reissue screen is accessed, is shown below.
  In this process, it is also checked whether the maximum limit for password reissue failure has exceeded. However, it is omitted here and will be described later.

  * Implementation of Service

    .. code-block:: java

       package org.terasoluna.securelogin.domain.service.passwordreissue;

       // omitted

       @Service
       @Transactional
       public class PasswordReissueServiceImpl implements PasswordReissueService {

           @Inject
           ClassicDateFactory dateFactory;

           @Inject
           PasswordReissueInfoRepository passwordReissueInfoRepository;

           // omitted

           @Override
           @Transactional(readOnly = true)
           public PasswordReissueInfo findOne(String token) {
               PasswordReissueInfo info = passwordReissueInfoRepository.findOne(token); // (1)

               if (info == null) {
                   throw new ResourceNotFoundException(ResultMessages.error().add(
                           MessageKeys.E_SL_PR_5002, token));
               }

               if (dateFactory.newTimestamp().toLocalDateTime().isAfter(info.getExpiryDate())) { // (2)
                   throw new BusinessException(ResultMessages.error().add(
                           MessageKeys.E_SL_PR_2001));
               }

               // omitted (attempts exceeded upper bounds)

               return info;
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
         - | Fetch authentication information for password reissue from the database by considering the token assigned as an argument, as the key.
       * - | (2)
         - | After the expiry date, throw \ ``org.terasoluna.gfw.common.exception.BusinessException`` \ .

  * Implementation of Controller

    .. code-block:: java

       package org.terasoluna.securelogin.app.passwordreissue;

       // omitted

       @Controller
       @RequestMapping("/reissue")
       public class PasswordReissueController {

           @Inject
           PasswordReissueService passwordReissueService;

           // omitted

           public String showPasswordResetForm(PasswordResetForm form, Model model,
                   @RequestParam("token") String token) { // (1)

               PasswordReissueInfo info = passwordReissueService.findOne(token); // (3)

               form.setUsername(info.getUsername());
               form.setToken(token);
               model.addAttribute("passwordResetForm", form);
               return "passwordreissue/passwordResetForm";
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
         - | Fetch the token included as a request parameter in the URL for password reissue screen.
       * - | (2)
         - | Call the Service method by passing the token in it. Authentication information is fetched from the database and expiry date is checked.

* User verification using authentication information for password reissue

  The implementation of the process to confirm whether the set of confidential information entered by the user on the password reissue screen and the token included in the URL of the password reissue screen is correct, is shown below.
  This confirmation process is a password reissue-specific logic. Since it is a check in which the results vary depending on the contents of the database, it is implemented in the Service without using Bean Validation and Spring Validator.

  * Implementation of Service

    .. code-block:: java

       package org.terasoluna.securelogin.domain.service.passwordreissue;

       // omitted

       public interface PasswordReissueService {

           // omitted

           boolean resetPassword(String username, String token, String secret, // (1)
                   String rawPassword);

           // omitted

       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - Sr. No.
         - Description
       * - | (1)
         - | A method to set the new password after user verification using user name, token and confidential information assigned as arguments


    .. code-block:: java
                    
       package org.terasoluna.securelogin.domain.service.passwordreissue;

       // omitted

       @Service
       @Transactional
       public class PasswordReissueServiceImpl implements PasswordReissueService {

           @Inject
           PasswordReissueFailureSharedService passwordReissueFailureSharedService;

           @Inject
           PasswordReissueInfoRepository passwordReissueInfoRepository;

           @Inject
           AccountSharedService accountSharedService;

           @Inject
           PasswordEncoder passwordEncoder;

           // omitted

           @Override
           public boolean resetPassword(String username, String token, String secret,
                   String rawPassword) {
               PasswordReissueInfo info = this.findOne(token); // (1)
               if (!passwordEncoder.matches(secret, info.getSecret())) { // (2)
                   passwordReissueFailureSharedService.resetFailure(username, token);
                   throw new BusinessException(ResultMessages.error().add(
                       MessageKeys.E_SL_PR_5003));
               }
               failedPasswordReissueRepository.deleteByToken(token);
               passwordReissueInfoRepository.delete(token); // (3)

               return accountSharedService.updatePassword(username, rawPassword); // (4)

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
         - | Using the token assigned as an argument, fetch the authentication information for password reissue from the database. At this time, the expiry date is checked again.
       * - | (2)
         - | Compare the hashed confidential information included in the authentication information for password reissue with the confidential information given as an argument. If they vary, throw \ ``BusinessException`` \ . Password reissue fails in this case.
       * - | (3)
         - | Delete used authentication information from the database in order to prevent it from being reused.
       * - | (4)
         - | Update the account password that has user name that was passed as an argument to the specified new password.

  * Implementation of Form

    Since input check other than Null check is covered depending on the annotation assigned to the class, only \ ``@NotNull`` \  is assigned as a single item check.
       
    .. code-block:: java

       package org.terasoluna.securelogin.app.passwordreissue;

       // omitted

       @Data
       @Compare(source = "newPasssword", destination = "confirmNewPassword", operator = Compare.Operator.EQUAL)
       @StrongPassword(usernamePropertyName = "username", newPasswordPropertyName = "newPassword") // (1)
       @NotReusedPassword(usernamePropertyName = "username", newPasswordPropertyName = "newPassword") // (2)
       public class PasswordResetForm implements Serializable{

           private static final long serialVersionUID = 1L;

           @NotNull
           private String username;

           @NotNull
           private String token;

           @NotNull
           private String secret;

           @NotNull
           private String newPassword;

           @NotNull
           private String confirmNewPassword;
       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - Sr. No.
         - Description
       * - | (1)
         - | An annotation to check the password strength. Refer to :ref:`Check password strength <password-strength>` for details.
       * - | (2)
         - | An annotation to check reuse of password. Refer to :ref:`Check password strength <password-strength>` for details.

  * Implementation of View

    **Password reissue screen(passwordResetForm.jsp)**

    .. code-block:: jsp

       <body>
           <div id="wrapper">
               <h1>Reset Password</h1>
               <t:messagesPanel />
               <form:form
                   action="${f:h(pageContext.request.contextPath)}/reissue/resetpassword"
                   method="POST" modelAttribute="passwordResetForm">
                   <table>
                       <tr>
                           <th><form:label path="username">Username</form:label></th>
                           <td>${f:h(passwordResetForm.username)} <form:hidden
                                   path="username" value="${f:h(passwordResetForm.username)}" />  <!-- (1) -->
                           </td>
                           <td></td>
                       </tr>
                       <form:hidden path="token" value="${f:h(passwordResetForm.token)}" /> <!-- (2) -->
                       <tr>
                           <th><form:label path="secret" cssErrorClass="error-label">Secret</form:label>
                           </th>
                           <td><form:password path="secret" cssErrorClass="error-input" /></td> <!-- (3) -->
                           <td><form:errors path="secret" cssClass="error-messages" /></td>
                       </tr>
                       <tr>
                           <th><form:label path="newPassword" cssErrorClass="error-label">New password</form:label>
                           </th>
                           <td><form:password path="newPassword"
                                   cssErrorClass="error-input" /></td>
                           <td><form:errors path="newPassword" cssClass="error-messages"
                                   htmlEscape="false" /></td>
                       </tr>
                       <tr>
                           <th><form:label path="confirmNewPassword"
                                   cssErrorClass="error-label">New password(Confirm)</form:label></th>
                           <td><form:password path="confirmNewPassword"
                                   cssErrorClass="error-input" /></td>
                           <td><form:errors path="confirmNewPassword"
                                   cssClass="error-messages" /></td>
                       </tr>
                   </table>

                   <input id="submit" type="submit" value="Reset password" />
               </form:form>
           </div>
       </body>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - Sr. No.
         - Description
       * - | (1)
         - | Retain user name as a hidden item.
       * - | (2)
         - | Retain token as a hidden item.
       * - | (3)
         - | Prompt the user to enter confidential information for user verification.

    **Password reissue screen(passwordResetComplete.jsp)**

    .. code-block:: jsp

       <body>
           <div id="wrapper">
               <h1>Your password was successfully reset.</h1>
               <a href="${f:h(pageContext.request.contextPath)}/">go to Top</a>
           </div>
       </body>

  * Implementation of Controller

    .. code-block:: java

       package org.terasoluna.securelogin.app.passwordreissue;

       // omitted

       @Controller
       @RequestMapping("/reissue")
       public class PasswordReissueController {

           @Inject
           PasswordReissueService passwordReissueService;

           // omitted

           @RequestMapping(value = "resetpassword", method = RequestMethod.POST)
           public String resetPassword(@Validated PasswordResetForm form,
                   BindingResult bindingResult, Model model) {
               if (bindingResult.hasErrors()) {
                   return showPasswordResetForm(form, model, form.getUsername(),
                           form.getToken());
               }

               try {
                   passwordReissueService.resetPassword(form.getUsername(),
                           form.getToken(), form.getSecret(), form.getNewPassword()); // (1)
                   return "redirect:/reissue/resetpassword?complete";
               } catch (BusinessException e) {
                   model.addAttribute(e.getResultMessages());
                   return showPasswordResetForm(form, model, form.getUsername(),
                           form.getToken());
               }
           }

           @RequestMapping(value = "resetpassword", params = "complete", method = RequestMethod.GET)
           public String resetPasswordComplete() {
               return "passwordreissue/passwordResetComplete";
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
         - | Pass the user name, token, confidential information and new password to the method of Service. If the combination of user name, token and confidential information is correct, it is updated to the new password.

.. _reissue-info-invalidate:

Setting failure limit for the password reissue
--------------------------------------------------------------------------------
List of requirements to be implemented
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
* :ref:`Setting failure limit for password reissue <sec-requirements>`

Working image
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. figure:: ./images/SecureLogin_invalidate_token.png
   :alt: Page Transition
   :width: 80%
   :align: center

Even if URL for password reissue screen is leaked for some reason, password will not be reissued illegally if the confidential information is not leaked.
Since a random value which cannot be guessed easily is used in the confidential information, it is highly unlikely to break the information easily. However, a maximum limit is set for the number of authentication failures to prevent brute force attack.
When the authentication failures for password reissue exceeds the maximum limit, the password reissue for that URL (token) is disabled.

Implementation method
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| In this application, history of password reissue failure is stored in the database as "password reissue failure event" entity and number of failures for password reissue is measured by using the password reissue failure event entity.
| When the number of failures is greater than the maximum value set in advance, an exception is thrown when the user tries to access password reissue screen.
| In particular, fulfil the requirements by implementing and using following two processes.

* Storing password reissue failure event entity

  When a failure occurs in the user authentication, during the process "Confirmation of user using authentication information for password reissue" in :ref:`reissue-info-validate`, set of the token used and failure date and time are registered in the database as password reissue failure event entity.

* Throwing an exception at the time of password reissue

  When authentication information is fetched from the database for password reissue, number of password reissue failure event entities is measured and an exception is thrown if the number is more than the maximum limit.

.. warning ::

   Since password reissue failure event entity is only intended for measuring number of failures for password reissue, it is deleted when it is no longer required.
   When the log at the time of password reissue failure is required, a separate log must always be maintained.

Code description
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* Common parts

  As a prerequisite, each process mentioned in :ref:`reissue-info-validate` should be implemented.
  Other commonly required implementations related to registration, search and deletion of password reissue failure event entity for the database are shown below.

  * Implementation of Entity

    Implementation of password reissue failure event entity is shown below.

    .. code-block:: java

       package org.terasoluna.securelogin.domain.model;

       // omitted

       @Data
       public class FailedPasswordReissue {

           private String token; // (1)

           private LocalDateTime attemptDate; // (2)

       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - Sr. No.
         - Description
       * - | (1)
         - | A token used for password reissue
       * - | (2)
         - | Date and time when password reissue is attempted

  * Implementation of Repository

    Repository for search, registration and deletion of Entity is shown below.

    .. code-block:: java

       package org.terasoluna.securelogin.domain.repository.passwordreissue;

       // omitted

       public interface FailedPasswordReissueRepository {

           int countByToken(@Param("token") String token); // (1)

           int create(FailedPasswordReissue event); // (2)

           int deleteByToken(@Param("token") String token); // (3)

           // omitted

       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - Sr. No.
         - Description
       * - | (1)
         - | A method which fetches number of \ ``FailedPasswordReissue``\  objects considering token assigned as an argument, as a key.
       * - | (2)
         - | A method which registers \ ``FailedPasswordReissue``\  object assigned as an argument, as the records of the database.
       * - | (3)
         - | A method which deletes \ ``FailedPasswordReissue``\  object considering token assigned as an argument, as a key.

    Mapping file is as given below.

    .. code-block:: xml

       <?xml version="1.0" encoding="UTF-8"?>
       <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

       <mapper
       	namespace="org.terasoluna.securelogin.domain.repository.passwordreissue.FailedPasswordReissueRepository">

       	<select id="countByToken" resultType="_int">
           <![CDATA[
               SELECT
                   COUNT(*)
               FROM
                   failed_password_reissue
               WHERE
                   token = #{token}
           ]]>
       	</select>

        <insert id="create" parameterType="FailedPasswordReissue">
           <![CDATA[
               INSERT INTO failed_password_reissue (
                   token,
                   attempt_date
               ) VALUES (
       	        #{token},
                   #{attemptDate}
               )
           ]]>
       	</insert>

       	<delete id="deleteByToken">
           <![CDATA[
           	DELETE FROM
           		failed_password_reissue
           	WHERE
           		token = #{token}
           ]]>
       	</delete>

       </mapper>

Code implemented in accordance with the implementation method is described here sequentially.

* Storing password reissue failure event entity

  A class which implements the process to be carried out at the time of password reissue failure is shown below.
  
  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.passwordreissue;

     public interface PasswordReissueFailureSharedService {

         void resetFailure(String username, String token);

     }

  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.passwordreissue;

     // omitted

     @Service
     @Transactional
     public class PasswordReissueFailureSharedServiceImpl implements
             PasswordReissueFailureSharedService {

         @Inject
         ClassicDateFactory dateFactory;

         @Inject
         FailedPasswordReissueRepository failedPasswordReissueRepository;

         // omitted

         @Transactional(propagation = Propagation.REQUIRES_NEW) // (1)
         @Override
         public void resetFailure(String username, String token) {
             FailedPasswordReissue event = new FailedPasswordReissue(); // (2)
             event.setToken(token);
             event.setAttemptDate(dateFactory.newTimestamp().toLocalDateTime());
             failedPasswordReissueRepository.create(event); // (3)
         }

     }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - Sr. No.
       - Description
     * - | (1)
       - | It is a method which is called when a failure occurs during password reissue and has been designed to generate a run-time exception in the call source.
         | Therefore, specify a propagation method in "REQUIRES_NEW" in order to perform transaction management separately from that of call source service.
     * - | (2)
       - | Create password reissue failure event entity and specify token and, failure date and time.
     * - | (3)
       - | Register password reissue failure event entity created in (2), in database.

  Call process at the time of password reissue failure from "Confirmation of user using authentication information for password reissue" process of :ref:`reissue-info-validate`.

  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.passwordreissue;

     // omitted

     @Service
     @Transactional
     public class PasswordReissueServiceImpl implements PasswordReissueService {

         @Inject
         PasswordReissueFailureSharedService passwordReissueFailureSharedService;

         @Inject
         PasswordReissueInfoRepository passwordReissueInfoRepository;

         @Inject
         AccountSharedService accountSharedService;

         @Inject
         PasswordEncoder passwordEncoder;

         // omitted

         @Override
         public boolean resetPassword(String username, String token, String secret,
                 String rawPassword) {
             PasswordReissueInfo info = this.findOne(token); // (1)
             if (!passwordEncoder.matches(secret, info.getSecret())) { // (2)
                 passwordReissueFailureSharedService.resetFailure(username, token); // (3)
                 throw new BusinessException(ResultMessages.error().add(  // (4)
                     MessageKeys.E_SL_PR_5003));
             }

             //omitted

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
       - | Fetch authentication information for password reissue from the database, using token assigned as an argument.
     * - | (2)
       - | Compare hashed confidential information included in the authentication information for password reissue and confidential information assigned as an argument.
     * - | (3)
       - | Call a method of SharedService which performs a process at the time of password reissue failure.
     * - | (4)
       - | A run-time exception is thrown, however since the process at the time of password reissue failure is performed in different transaction, it does not leave any impact.

* Throwing an exception at the time of password reissue

  Fetching number of failures at the time of password reissue and implementation of process when the number of failures reach the maximum limit are shown below.

  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.passwordreissue;

     // omitted

     @Service
     @Transactional
     public class PasswordReissueServiceImpl implements PasswordReissueService {

         @Inject
         FailedPasswordReissueRepository failedPasswordReissueRepository;

         @Inject
         PasswordReissueInfoRepository passwordReissueInfoRepository;

         @Value("${security.tokenValidityThreshold}")
         int tokenValidityThreshold; // (1)

         // omitted

         @Override
         @Transactional(readOnly = true)
         public PasswordReissueInfo findOne(String token) {

             // omitted
              
             int count = failedPasswordReissueRepository // (2)
                     .countByToken(token);
             if (count >= tokenValidityThreshold) { // (3)
                 throw new BusinessException(ResultMessages.error().add(
                         MessageKeys.E_SL_PR_5004));
             }

             return info;
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
       - | Fetch and set maximum value for number of failures at the time of password reissue, from property file.
     * - | (2)
       - | Fetch number of password reissue failure event entities from the database, considering token assigned as an argument, as a key.
     * - | (3)
       - | Compare number of failure event entities at the time of password reissue that has been fetched and maximum limit for number of failures, and throw an exception if it exceeds the maximum limit.

Conclusion
================================================================================

| This chapter explains about an example of implementation method for security measures using a sample application.
| Since it is likely that implementation method in this application cannot be used as it is in the actual development, a different customised method must be considered according to the requirements, using details of this chapter as a reference.

Appendix
================================================================================

.. _passay_overview:

Passay
--------------------------------------------------------------------------------

Passay is a library which offers a password validation function and a password generation function.
Passay API consists of following three key components.

* Validation rules

  It defines the conditions which must be fulfilled by password. Validation rules which are used widely like length of the password and characters types to be included can be created easily by using the class offered by library. Besides, the necessary validation rules can also be defined by the user.

* Validator

  A component which performs password validation based on validation rules. Various validation rules can be specified in a single validator.

* Generator

  A component which generates a password in conformance with the validation rules for the assigned character type.

When Passay function is to be used, following definition must be added to pom.xml.

.. code-block:: xml

   <dependencies>
     <dependency>
         <groupId>org.passay</groupId>
         <artifactId>passay</artifactId>
         <version>1.1.0</version>
     </dependency>
   <dependencies>

.. _password_validation:

Password validation
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Overview
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| A schematic diagram of the password validation flow in Passay is shown below.

.. figure:: ./images/SecureLogin_passay.png
   :alt: Password Vaildation
   :width: 60%
   :align: center

.. list-table::
   :header-rows: 1
   :widths: 10 90
  
   * - Sr. No.
     - Description
   * - | (1)
     - | Create an instance of \ ``org.passay.PasswordData`` \  and specify information related to password to be validated.
       | \ ``PasswordData`` \  can include list of passwords used in the past as a property in addition to password and user name.
       | Passwords used in the past are retained as an instance of \ ``org.passay.PasswordData.Reference`` \ .
   * - | (2)
     - | Perform validation for \ ``PasswordData`` \ using a validator, in accordance with the validation rules.
       | Validation rules are created as an instance of implementation class of \ ``org.passay.Rule`` \ . A validator is an instance of \ ``org.passay.PasswordValidator`` \  and can include multiple validation rules as properties.
   * - | (3)
     - | An instance of \ ``org.passay.RuleResult`` \  is created as validation result using a validator.
   * - | (4)
     - | Password validation results can be fetched from \ ``RuleResult`` \  as a \ ``boolean`` \  value. Also, an error message can be fetched from \ ``RuleResult`` \  by using a validator.

Some of the classes of validation rules offered by Passay are shown in the table below.

.. tabularcolumns:: |p{0.20\linewidth}|p{0.40\linewidth}|p{0.40\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 40 40

   * - Class Name
     - Description
     - Key properties
   * - | \ ``LengthRule`` \
     - | A class of validation rules to specify minimum and maximum value for password length.
     - | \ ``minimuxLength`` \ : Minimum value for password length (\ ``int`` \). Specify in constructor or setter.
       | \ ``maximumLength`` \ : Maximum value for password length (\ ``int`` \)。Specify in constructor or setter.
   * - | \ ``CharacterRule`` \
     - | A class of validation rules to specify character types that must be included in the password and the minimum number of characters of that character type.
     - | \ ``characterData``\ : Character type (\ ``org.passay.CharacterData`` \). Specify in constructor.
       | \ ``numberOfCharacters`` \ : Minimum number of characters (\ ``int`` \). Specify in constructor or setter.
   * - | \ ``CharacterCharacteristicsRule`` \
     - | A class of validation rules which specify the number of rules that should be fulfilled, from multiple \ ``CharacterRule`` \ .
     - | \ ``rules``\ : List of validation rules related to character types (\ ``List<CharacterRule>`` \). Specify in setter.
       | \ ``numberOfCharacteristics`` \ : Minimum number of rules that should be fulfilled (\ ``int`` \). Specify in setter.
   * - | \ ``HistoryRule`` \
     - | A class of validation rules which checks the pasword does not match with the password used previously.
     - | None
   * - | \ ``UsernameRule`` \
     - | A class of validation rules to check that password should not contain the user name.
     - | \ ``matchBackwards`` \ : Also check whether the user name has been used in the reverse (\ ``boolean`` \). Specify in constructor or setter.
       | \ ``ignoreCase`` \ : Not case-sensitive (\ ``boolean`` \). Specify in constructor or setter.

In addition, classes of validation rules to check whether a specific character is to be included or to check using a regular expression are also provided.
For details, refer `<http://www.passay.org/>`_.

How to use
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A validator can be created by passing the list of \ ``org.passay.Rule`` \  instance to the constructor of \ ``PasswordValidator`` \ .
DI can be applied by defining a Bean for validator as below which specifies the validation rules.
Note that, when a Bean is to be defined for multiple validation rules, DI must be applied using a Bean name, by combining \ ``@Inject`` \  and \ ``@Named`` \ .

.. code-block:: xml

   <!-- Password Rules. -->
   <bean id="upperCaseRule" class="org.passay.CharacterRule"> <!-- (1) -->
       <constructor-arg name="data">
           <util:constant static-field="org.passay.EnglishCharacterData.UpperCase" /> <!-- (2) -->
       </constructor-arg>
       <constructor-arg name="num" value="1" /> <!-- (3) -->
   </bean>
   <bean id="lowerCaseRule" class="org.passay.CharacterRule"> <!-- (4) -->
       <constructor-arg name="data">
           <util:constant static-field="org.passay.EnglishCharacterData.LowerCase" />
       </constructor-arg>
       <constructor-arg name="num" value="1" />
   </bean>
   <bean id="digitRule" class="org.passay.CharacterRule"> <!-- (5) -->
       <constructor-arg name="data">
           <util:constant static-field="org.passay.EnglishCharacterData.Digit" />
       </constructor-arg>
       <constructor-arg name="num" value="1" />
   </bean>

   <!-- Password Validator. -->
   <bean id="characterPasswordValidator" class="org.passay.PasswordValidator"> <!-- (6) -->
       <constructor-arg name="rules">
           <list>
               <ref bean="upperCaseRule" />
               <ref bean="lowerCaseRule" />
               <ref bean="digitRule" />
           </list>
       </constructor-arg>
   </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.60\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Define a Bean for validation rules for specifying character type that must be included in the password and minimum number of characters for the character type.
   * - | (2) 
     - | Specify character type. Since \ ``org.passay.EnglishCharacterData.UpperCase`` \  is passed, validation rules are set for single byte uppercase characters.
   * - | (3)
     - | Specify number of characters. Since "1" is passed, validation rules are set to check whether one or more single byte uppercase characters are included.
   * - | (4)
     - | It is similar to (1)-(3), however since \ ``org.passay.EnglishCharacterData.UpperCase`` \  is passed as a character type, a bean is defined for the validation rules to check whether one or more single byte lowercase letters are included.
   * - | (5)
     - | It is similar to (1)-(3), however, since \ ``org.passay.EnglishCharacterData.Digit`` \  is passed as a character type, a bean is defined for the validation rules to check whether one or more single byte digit is included.
   * - | (6)
     - | Define a Bean for validator. Pass a list of validation rules to the constructor.

Perform password validation by using created validator.

.. code-block:: java

   @Inject
   PasswordValidator characterPasswordValidator;

   // omitted

   public void validatePassword(String password){

       PasswordData pd = new PasswordData(password); // (1)
       RuleResult result = characterPasswordValidator.validate(pd); // (2)
       if (result.isValid()) { // (3)
          logger.info("Password is valid");
       } else {
          logger.error("Invalid password:");
          for (String msg : characterPasswordValidator.getMessages(result)) { // (4)
              logger.error(msg);
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
     - | Pass the password to be validated to the constructor of \ ``PasswordData`` \  and create an instance.
   * - | (2) 
     - | Pass \ ``PasswordData`` \  in the \ ``validate`` \  method of \ ``PasswordValidator`` \  as an argument and implement password validation.
   * - | (3)
     - | Fetch password validation results in truth value by using \ ``isValid`` \  method of \ ``RuleResult`` \ .
   * - | (4)
     - | Pass \ ``RuleResult`` \  in \ ``getMessages`` \  method of \ ``PasswordValidator`` \  as an argument and fetch error message.

.. _password_generation:

Password generation
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Overview
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| Password generator and generation rules are used in the password generation function for Passay. A generator is an instance of \ ``org.passay.PasswordGenerator`` \ and the generation rules is a list of validation rules related to character type (\ ``org.passay.CharacterRule`` \).
| By assigning the length of the password to be generated and generation rules to the method of generator as arguments, a password which fulfils the generation rules is generated.

How to use
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A method of creation of validation rules related to character type which is included in the generation rules is similar to :ref:`password_validation`.
DI can be applied by defining a bean for generation rules and generator as given below.

.. code-block:: xml

   <!-- Password Rules. -->
   <bean id="upperCaseRule" class="org.passay.CharacterRule"> <!-- (1) -->
       <constructor-arg name="data">
           <util:constant static-field="org.passay.EnglishCharacterData.UpperCase" /> <!-- (2) -->
       </constructor-arg>
       <constructor-arg name="num" value="1" /> <!-- (3) -->
   </bean>
   <bean id="lowerCaseRule" class="org.passay.CharacterRule"> <!-- (4) -->
       <constructor-arg name="data">
           <util:constant static-field="org.passay.EnglishCharacterData.LowerCase" />
       </constructor-arg>
       <constructor-arg name="num" value="1" />
   </bean>
   <bean id="digitRule" class="org.passay.CharacterRule"> <!-- (5) -->
       <constructor-arg name="data">
           <util:constant static-field="org.passay.EnglishCharacterData.Digit" />
       </constructor-arg>
       <constructor-arg name="num" value="1" />
   </bean>

    <!-- Password Generator. -->
    <bean id="passwordGenerator" class="org.passay.PasswordGenerator" /> <!-- (6) -->
    <util:list id="passwordGenerationRules"> <!-- (7) -->
        <ref bean="upperCaseRule" />
        <ref bean="lowerCaseRule" />
        <ref bean="digitRule" />
    </util:list>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Define a Bean for validation rules for specifying the character type that should be included in the password, and minimum number of characters for the character type.
   * - | (2) 
     - | Specify character type. Since \ ``org.passay.EnglishCharacterData.UpperCase`` \  is passed, validation rules are set for single byte uppercase characters.
   * - | (3)
     - | Specify number of characters. Since "1" is passed, validation rules are set to check whether one or more single byte uppercase characters are included.
   * - | (4)
     - | It is similar to (1)-(3), however since \ ``org.passay.EnglishCharacterData.UpperCase`` \  is passed as a character type, a bean is defined for the validation rules to check whether one or more single byte lowercase letters are included.
   * - | (5)
     - | It is similar to (1)-(3), however, since \ ``org.passay.EnglishCharacterData.Digit`` \  is passed as a character type, a bean is defined for the validation rules to check whether one or more single byte digit is included.
   * - | (6)
     - | Define a bean for generator.
   * - | (7)
     - | Define a bean for generation rules. It is defined as a list of validation rules related to character type, defined in (1)-(5).

Create a password by using created generator and generation rules.

.. code-block:: java

   @Inject
   PasswordGenerator passwordGenerator;

   @Resource(name = "passwordGenerationRules")
   List<CharacterRule> passwordGenerationRules;

   // omitted

   public void generatePassword(){

       String password = passwordGenerator.generatePassword(10, passwordGenerationRules); // (1)

   }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | If length of the password to be generated and the generation rules are passed to \ ``generatePassword`` \  method of \ ``PasswordGenerator`` \  as arguments, a password which fulfils the generation rules is created.

.. tip::
     
   When DI is to be applied to a collection for which a Bean is defined, an expected operation is not performed by \ ``@Inject`` \  + \ ``@Named`` \ .
   Therefore, DI is applied by Bean name using \ ``@Resource`` \  instead.

.. raw:: latex

   \newpage

