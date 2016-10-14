.. _SpringSecuritySessionManagement:

Session Management
================================================================================

.. only:: html

 .. contents:: Table of contents
    :local:

Overview
--------------------------------------------------------------------------------

This chapter explains the "Security measures required for session management in Web applications" and "Session related functions provided by Spring Security".

.. _SpringSecuritySessionManagementSecurityMeasure:

Security measures at the time of using a session
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Generally, countermeasures are required for the following attacks for session management in Web application.

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - Countermeasure
      - Description
    * - | Session hijacking attack
      - | An attack wherein the session ID is stolen using communication eavesdropping, analogy from the regularity and cross-site scripting and using the system with stolen ID by pretending to be the user.
    * - | Session fixation attack
      - | The attacker allows the other user to log in to the system by using the previously issued session ID. The attacker uses the system by pretending to be the logged in user.

Countermeasures for session hijacking attacks
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Countermeasures for session hijacking attacks enable you to only prevent the session ID from being stolen.
If it is stolen, application server cannot determine
whether the request is from an authorized user or from the attacker.

Following countermeasures are necessary to protect the application from such session hijacking attacks.

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table:: **Countermeasures for session hijacking attacks**
    :header-rows: 1
    :widths: 30 70

    * - Countermeasures
      - Description
    * - | Create a session ID that is difficult to presume.
      - | Use a random value for Session ID that is difficult to presume (Secure) instead of a value such as sequential number that can be easily cracked.
        | Basically, the mechanism provided by the application server to create session ID should be used.
    * - | Encrypt communication using HTTPS
      - | Encrypt the communication with HTTPS protocol that includes exchange of information which can cause a lot of trouble when stolen.
        | Since communication eavesdropping can be done easily by using a free software etc., it is important to keep it encrypted to prevent it from being decrypted even if it is eavesdropped.
    * - | Link Session ID using a Cookie
      - | While linking the session ID between client and server, set it to be linked by using a Cookie and disable the URL Rewriting function.
    * - | Specify \ ``HttpOnly``\  attribute of Cookie
      - | If you specify \ `` HttpOnly`` \ attribute of Cookie, session ID  cannot be stolen using cross site scripting since Cookie cannot be accessed from JavaScript.
    * - | Specify \ ``Secure``\  attribute in Cookie
      - | If you specify \ `` Secure`` \ attribute in Cookie, the risk of session ID getting stolen when you accidentally use HTTP communication is reduced since Cookie is sent to the server only during HTTPS communication.

.. note:: **URL Rewriting**

    URL Rewriting is a mechanism to retain the session with the client that cannot use Cookie.
    Particularly, the Session ID between client and server is linked by including Session ID as a request parameter of the URL.

    * Example of URL Rewriting

        .. code-block:: guess

            http://localhost:8080/;jsessionid=7E6EDE4D3317FC5F14FD912BEAC96646

    \ ``jsessionid=7E6EDE4D3317FC5F14FD912BEAC96646``\  is a part of Session ID for which URL Rewriting is done.
    In the Servlet API specifications, URL rewriting is likely to be done when the following methods are called. These methods are also called in the JSP tag library provided by JSTL and Spring.

    * \ ``HttpServletResponse#encodeURL(String)``\
    * \ ``HttpServletResponse#encodeRedirectURL(String)``\

If URL Rewriting is done, session ID in the URL is exposed resulting in higher risk of session ID being stolen.
Therefore, it is recommended to disable the URL Rewriting function of Servlet container while supporting only the clients that can use the Cookie.

|

Countermeasures for Session fixation attack
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Following countermeasures are necessary to protect the application from Session fixation attack.

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table:: **Countermeasures for Session fixation attack**
    :header-rows: 1
    :widths: 30 70

    * - Countermeasure
      - Description
    * - | Disable URL Rewriting function.
      - | If URL Rewriting function is disabled, session ID issued previously cannot be used by the attacker and a new session is started.
    * - | Change session ID after login
      - | By changing the session ID after login, the session ID issued previously cannot be used by the attacker.

|

Session management function provided by Spring Security
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Following functions related to session are mainly provided in Spring Security.

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: **Session related functions**
    :header-rows: 1
    :widths: 25 75

    * - Function
      - Description
    * - | Security measures
      - | Countermeasures for attacks using session ID of session hijacking attacks.
    * - | Lifecycle control
      - | Function to control the lifecycle of the session from generation to discard of a session.
    * - | Timeout control
      - | Function to discard a session due to timeout.
    * - | Multiple login control
      - | Function to control a session if the same user logs in for multiple times.

.. _authentication(spring_security)_how_to_use_sessionmanagement:

How to use
--------------------------------------------------------------------------------

Countermeasures for Session hijacking attacks
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The method to disable a URL rewriting function and link session ID using a Cookie is explained.

Disabling URL Rewriting function by Spring Security
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Security provides a mechanism to disable URL Rewriting and this function is applied by default.
When it is necessary to support the clients who cannot use a Cookie, a Bean is defined so as to authorize URL rewriting.

* Definition example of spring-security.xml

.. code-block:: xml

    <sec:http disable-url-rewriting="false"> <!-- Enable URL Rewriting by specifying 'false' -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | In Spring Security, since the value of \ `` disable-url-rewriting`` \  is \ `` true`` \  by default, URL Rewriting is not performed.
        | Set \ ``false``\  in \ ``disable-url-rewriting``\  attribute of \ ``<sec:http>``\  element to enable URL Rewriting.

Disabling URL Rewriting function by Servlet Container
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

A session can be managed securely using the standard specifications of Servlet.

* Definition example of web.xml

.. code-block:: xml

    <session-config>
        <cookie-config>
            <http-only>true</http-only> <!-- (1)  -->
        </cookie-config>
        <tracking-mode>COOKIE</tracking-mode> <!-- (2) -->
    </session-config>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify \ ``true``\  in \ ``<http-only>``\  element while assigning \ ``HttpOnly``\  attribute in Cookie.
        | Default value is set to \ `` true`` \  based on the application server used.
    * - | (3)
      - | Specify \ ``COOKIE``\  in \ ``<tracking-mode>``\  element to disable the URL Rewriting function.

Although it is omitted from the definition example mentioned above, \ ``Secure``\  attribute can be assigned for the Cookie by adding \ ``<secure>true</secure>``\  in \ ``<cookie-config>``\ .
However, to secure the cookie, the method wherein a middleware (SSL accelerators and Web server etc.) that performs HTTPS communication with the client is assigned, is used instead of specifying in \ `` web.xml`` \ .

In actual system development, HTTPS is used very rarely in the local development environment.
Also, even in the production environment, HTTPS is used for SSL accelerators and communication with Web server and there are many cases where communication with application server is carried out using HTTP.
If \ `` Secure`` \  attribute is specified in \ `` web.xml`` \  under such environment, \ ``web.xml``\  and \ ``web-fragment.xml``\  will be provided for each execution environment. It is not recommended since file management becomes complicated.


.. _SpringSecuritySessionManagementSetup:

Applying Session management function
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A method to use session management function of Spring Security is explained.
Define a bean as shown below to use the session management function process of Spring Security.

* Definition example of spring-security.xml

.. code-block:: xml

    <sec:http>
        <!-- omitted -->
        <sec:session-management /> <!-- (1) -->
        <!-- omitted -->
    </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify \ ``<sec:session-management>``\  element as the child element of \ ``<sec:http>``\  element.
        | Session management function is applied when \ ``<sec:session-management>``\  element is specified.

|

Countermeasures for Session fixation attack
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Security provides the following four options to change the session ID when login is successful, as the countermeasures against session fixation attack.

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table:: **Options for Session fixation attack countermeasures**
    :header-rows: 1
    :widths: 30 70

    * - Options
      - Description
    * - | \ ``changeSessionId``\
      - | Change the session ID using \ ``HttpServletRequest#changeSessionId()``\  added in Servlet 3.1.
        | (This is the default operation from Servlet 3.1 container onwards)
    * - | \ ``migrateSession``\
      - | Discard the session that was used before login and create a new session.
        | The objects stored in the session before login are transferred to the new session when this option is used.
        | (This is the default operation in Servlet 3.0 container and earlier versions)
    * - | \ ``newSession``\
      - | This option changes the session ID in the same way as \ `` migrateSession`` \ , however, the objects stored before login are not transferred to the new session.
    * - | \ ``none``\
      - | Spring Security does not change the session ID.

Define a bean as shown below to change the default operation.

* Definition example of spring-security.xml

.. code-block:: xml

    <sec:session-management
            session-fixation-protection="newSession"/> <!-- (1) -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify the countermeasures for session fixation attack in \ ``session-fixation-protection``\  attribute of ``<sec:session-management>``\  element.

.. _SpringSecuritySessionManagementLifecycle:

Controlling session lifecycle
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Security uses HTTP session for sharing objects such as authentication information across requests. The lifecycle of the session (Generating and discarding a session) is controlled in the Spring Security process.

.. note:: **Session information storage destination**

    HTTP session is used in the default implementation provided by Spring Security, however, the architecture also enables storing the objects in other than HTTP session (Database and key-value store etc.).

Generating a session
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The guidelines by which a session can be generated and used in the Spring Security process can be selected from the following options.

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: **Guidelines to generate a session**
    :header-rows: 1
    :widths: 25 75

    * - Option
      - Description
    * - | \ ``always``\
      - | Generate a new session when the session does not exist.
        | If this option is specified, the session is generated even though it is not used in the Spring Security process.
    * - | \ ``ifRequired``\
      - | Generate and use a new session at the time of storing object in the session when the session does not exist.(Default operation)
    * - | \ ``never``\
      - | If the session does not exist, session is not generated and used.
        | However, use the session if it already exists.
    * - | \ ``stateless``\
      - | Session is not generated and used irrespective of whether it exists.

Define a bean as shown below to change the default behaviour.

* Definition example of spring-security.xml

.. code-block:: xml

    <sec:http create-session="stateless"> <!-- (1) -->
        <!-- omitted -->
    </sec:http>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | \ (1)
      - | Specify creation guidelines for the session to be changed in \ ``create-session``\  attribute of \ ``<sec:http>``\  element.

Discarding a session
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Security discards the session at the following timings.

* When logout process is executed
* When authentication process is successful (Session is discarded if \ ``migrateSession``\  or \ ``newSession``\  is used as the countermeasure for Session fixation attack)

.. _SpringSecuritySessionManagementTimeout:

Controlling session timeout
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When an object is to be stored in the session, it is a common practice to ensure that session of the user who is idle for a certain period of time is automatically discarded by specifying an appropriate session time-out value.

Specifying session timeout
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Specify session timeout for Servlet container.
In some cases, the specification method independent of server may be provided depending on the application server, however, the specification methods stated in Servlet standard specifications are described here.

* Definition example of web.xml

.. code-block:: xml

    <session-config>
        <session-timeout>60</session-timeout> <!-- (1) -->
        <!-- omitted -->
    </session-config>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify an appropriate timeout value (in minutes) in \ ``<session-timeout>``\  element.
        |  If a timeout value is not specified, the default value provided by Servlet Container is used.
        | Also, if a value of 0 or less than 0 is specified, the session timeout function of Servlet container is disabled.

.. _SpringSecuritySessionDetectInvalidSession:

Detection of request with invalid session
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Security provides a function to detect a request with an invalid session.
Most of the requests handled as invalid sessions are requests after session timeout.
By default, this functionality is disabled, however, it can be enabled by defining a bean as shown below.

* Definition example of spring-security.xml

.. code-block:: xml

    <sec:session-management
            invalid-session-url="/error/invalidSession"/>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify the path for redirect destination when a request with invalid session is detected in \ ``invalid-session-url``\  attribute of \ ``<sec:session-management>``\  element.

Specifying Exclusion path
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

If the function to detect a request with invalid session is enabled, all the requests that pass through Servlet filter of Spring Security are checked.
Therefore, check is performed even for pages that can be accessed when session is in invalid state.

This operation can be changed by defining a separate bean for the path to be excluded from the check target.
For example, a bean is defined as shown below to specify the path to open the top page (\ ``"/"``\ ) in the exclusion path.

* Definition example of spring-security.xml

.. code-block:: xml

    <!-- (1) -->
    <sec:http pattern="/"> <!-- (2) -->
        <sec:session-management />
    </sec:http>

    <!-- (3) -->
    <sec:http>
        <!-- omitted -->
        <sec:session-management
                invalid-session-url="/error/invalidSession"/>
        <!-- omitted -->
    </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Add new element \ ``<sec:http>``\  element to create \ ``SecurityFilterChain``\  which is to be applied to (\ ``"/"``\ ) path to open the top page.
    * - | (2)
      - | Specify the path pattern to apply \ ``SecurityFilterChain``\  created using \ ``<sec:http>``\  element of (1).
        | Ant-style path notation and regular expression are the two formats that can be specified for the path pattern. By default, it is handled as Ant-style path pattern.
        | Note that, it is also possible to directly specify \ `` RequestMatcher`` \  object without the path pattern.
    * - | (3)
      - | Define \ ``<sec:http>``\  element to create \ ``SecurityFilterChain``\  to be applied to a path that is not defined separately.
        | It should be defined below the \ ``<sec:http>``\  element for separate definition.
        | This is because definition sequence of \ ``<sec:http>``\  element is the priority sequence of \ ``SecurityFilterChain``\ .

|

.. _SpringSecuritySessionManagementConcurrency:

Controlling multiple logins
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Security provides a function to control multiple logins using the same user name (login ID).
By default this function is disabled. However, it can be enabled by using :ref:`SpringSecurityHowToUseSessionManagementConcurrency`.

.. warning:: **Constraints in multiple login control**

    In the default implementation provided by Spring Security, following constraints are observed since session information of each user is managed within the application server memory.

    First constraint is that the default implementation cannot be used in the system wherein multiple application servers are started concurrently.
    If multiple application servers are to be used concurrently, it is necessary to create the implementation class to manage session information of each user in the shared area such as database or key-value store (Cache server).

    The second constraint is that if session information is restored when application server is stopped or re-started, it may not operate normally.
    Since it has a function to restore the session state at the time of stop or restart depending on the application server to be used, an inconsistency may appear in the actual session state and the session information managed by Spring Security.
    One of the following actions should be taken in case of a likely inconsistency.

    * Do not restore session state of the application server.
    * Implement a mechanism to restore the session information of Spring Security.
    * Store the object in other than HTTP session (Database or key-value store etc.)

This section introduces a method to use default implementation of Spring Security.
HTTP session is used in the default implementation provided by Spring Security, however, the architecture also enables storing objects in other than HTTP session (Database or key-value store etc.).
However, please note that the method introduced here is the **Implementation method with the constraints of the Warning mentioned above**, during the application.

.. Todo::
   The information about the implementation method that does not use in-memory will be added later.

.. _SpringSecurityHowToUseSessionManagementConcurrency:

Enabling session lifecycle detection
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The function to control multiple login manages the session state for each user by using :ref:`session lifecycle (Generating and discarding session) detection mechanism <SpringSecuritySessionManagementLifecycle>`.
Therefore, while using multiple login control function, \ ``HttpSessionEventPublisher``\  class provided by Spring Security must be registered in Servlet Container.

* Definition example of web.xml

.. code-block:: xml

    <listener>
        <!-- (1) -->
        <listener-class>
            org.springframework.security.web.session.HttpSessionEventPublisher
        </listener-class>
    </listener>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Register \ ``HttpSessionEventPublisher``\  as Servlet Listener.

Preventing multiple logins (Pre-measure)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

A bean is defined as shown below if multiple login is to be prevented by generating an authentication error in case of the users who have already logged in with the same user name (login ID).

* Definition example of bean definition file

.. code-block:: xml

    <sec:session-management>
        <sec:concurrency-control
                max-sessions="1"
                error-if-maximum-exceeded="true"/> <!-- (1) (2) -->
    </sec:session-management>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - \ (1)
      - Specify the number of sessions for which concurrent logins are allowed
        in \ ``max-sessions``\  attribute of \ ``<sec:concurrency-control>``\  element.
        Specify \ `` 1`` \  to prevent multiple logins.
    * - \ (2)
      - Specify the operation to be performed when the number of sessions to which user can login concurrently is exceeded,
        in \ ``error-if-maximum-exceeded``\  attribute of \ ``<sec:concurrency-control>``\  element.
        Specify \ ``true``\  if a user who has already logged in is handled
        as a valid user.

Preventing multiple logins (Post-measure)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

In case of the users who have already logged in with the same user name (login ID),
a bean is defined as below if multiple login is to be prevented by invalidating the users
who are already logged in.

* Definition example of spring-security.xml

.. code-block:: xml

    <sec:session-management>
        <sec:concurrency-control
                max-sessions="1"
                error-if-maximum-exceeded="false"
                expired-url="/error/expire"/> <!-- (1) (2) -->
    </sec:session-management>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify the operation to be performed when the number of sessions a user can login concurrently has exceeded, in \ ``error-if-maximum-exceeded``\  attribute of \ ``<sec:concurrency-control>``\  element.
        | Specify \ `` false`` \  if a new logged-in user is handled as a valid user.
    * - | (2)
      - | Specify the path for redirect destination when a request from an invalidated user is detected in \ ``expired-url``\  attribute of \ ``<sec:concurrency-control>``\  element.
        | This is because definition sequence of \ ``<sec:http>``\  element is the priority sequence of \ ``SecurityFilterChain``\ .

.. raw:: latex

   \newpage

