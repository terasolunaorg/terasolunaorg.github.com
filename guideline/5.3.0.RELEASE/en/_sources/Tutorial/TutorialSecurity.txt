Spring Security Tutorial
================================================================================

.. only:: html

.. contents:: Table of Contents
   :depth: 3
   :local:


Introduction
--------------------------------------------------------------------------------

Topics covered in this tutorial
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* Basic authentication/authorization using Spring Security
* Login using the account information in the database
* How to fetch authenticated account object

Target Audience
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* Completed implementation of :doc:`./TutorialTodo`\ (MyBatis3 should be used for implementing infrastructure layer) 
* Understands the basic operations of Maven

Verification environment
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* Similar to :doc:`./TutorialTodo`\ . 

|

Overview of application to be created
--------------------------------------------------------------------------------

* Login to application is possible by specifying ID and password on login page.
* Store account information required for login in the database.
* There is a Welcome page and Account information display page which can be viewed only by the logged in users.
* Logout from application is possible.

Overview of application is shown in the following figure.

.. figure:: ./images_Security/security_tutorial_applicatioin_overview.png
   :width: 90%

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
      - Login form display
      - GET
      - /login.jsp
      - Displays login form
    * - 2
      - Login
      - POST
      - /authentication
      - Authenticates using username and password entered from login form (performed by Spring Security)
    * - 3
      - Welcome page display
      - GET
      - /
      - Displays Welcome page.
    * - 4
      - Account information display
      - GET
      - /account
      - Displays account information of logged-in user.
    * - 5
      - Logout
      - POST
      - /logout
      - Performs logout (performed by Spring Security)

|

Creating environment
--------------------------------------------------------------------------------

Creating a project
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create \ `A blank project of TERASOLUNA Server Framework for Java (5.x) <https://github.com/terasolunaorg/terasoluna-gfw-web-blank>`_\  using Maven archetype.

In this tutorial, a blank project is created for MyBatis3.

Basic knowledge such as how to import to Spring Tool Suite(STS), how to start an application server, etc. is omitted in this tutorial,
since it is already described in :doc:`./TutorialTodo`.

.. code-block:: console

    mvn archetype:generate -B^
     -DarchetypeGroupId=org.terasoluna.gfw.blank^
     -DarchetypeArtifactId=terasoluna-gfw-web-blank-mybatis3-archetype^
     -DarchetypeVersion=5.3.0.RELEASE^
     -DgroupId=com.example.security^
     -DartifactId=first-springsecurity^
     -Dversion=1.0.0-SNAPSHOT

|

Most of the settings which are required for executing this tutorial are already performed in blank project.
It is not mandatory to understand these settings just for executing the tutorial; however,
it is recommended that you understand the settings which are required to run the application.

For description about settings required to run the application (configuration file),
refer to ":ref:`SecurityTutorialAppendixConfigurationFiles`".

|

Creating an application
--------------------------------------------------------------------------------

Implementing domain layer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The flow of authentication process of Spring Security is as follows:

#. Search user information from the entered \ ``username``\ .
#. When user information exists, compare the password stored in the corresponding user information with the hashed password that has been entered.
#. When passwords match, authentication is considered to be successful.

If user information is not found or if the passwords do not match, authentication fails.

In domain layer, process to fetch Account object from user name is essential. The process is implemented in the following order.

#. Creation of Domain Object(\ ``Account``\ )
#. Creation of \ ``AccountRepository``\ 
#. Creation of \ ``AccountSharedService``\ 

|

Creating a Domain Object
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Create \ ``Account``\  class that stores authentication information (user name and password).
| ``src/main/java/com/example/security/domain/model/Account.java``

.. code-block:: java
  
    package com.example.security.domain.model;
  
    import java.io.Serializable;
  
    public class Account implements Serializable {
        private static final long serialVersionUID = 1L;
  
        private String username;
  
        private String password;
  
        private String firstName;
  
        private String lastName;
  
        public String getUsername() {
            return username;
        }
  
        public void setUsername(String username) {
            this.username = username;
        }
  
        public String getPassword() {
            return password;
        }
  
        public void setPassword(String password) {
            this.password = password;
        }
  
        public String getFirstName() {
            return firstName;
        }
  
        public void setFirstName(String firstName) {
            this.firstName = firstName;
        }
  
        public String getLastName() {
            return lastName;
        }
  
        public void setLastName(String lastName) {
            this.lastName = lastName;
        }
  
        @Override
        public String toString() {
            return "Account [username=" + username + ", password=" + password
                    + ", firstName=" + firstName + ", lastName=" + lastName + "]";
        }
    }

|

Creating AccountRepository
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Implement a process to fetch \ ``Account``\  object from the database.

| Create \ ``AccountRepository``\  interface.
| ``src/main/java/com/example/security/domain/repository/account/AccountRepository.java``

.. code-block:: java
  
    package com.example.security.domain.repository.account;
  
    import com.example.security.domain.model.Account;

    public interface AccountRepository {
        Account findOne(String username);
    }

|

| Define SQL for fetching single \ ``Account``\  record, in Mapper file.
| ``src/main/resources/com/example/security/domain/repository/account/AccountRepository.xml``

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.security.domain.repository.account.AccountRepository">

        <resultMap id="accountResultMap" type="Account">
            <id property="username" column="username" />
            <result property="password" column="password" />
            <result property="firstName" column="first_name" />
            <result property="lastName" column="last_name" />
        </resultMap>

        <select id="findOne" parameterType="String" resultMap="accountResultMap">
            SELECT
                username,
                password,
                first_name,
                last_name
            FROM
                account
            WHERE
                username = #{username}
        </select>
    </mapper>

|

Creating AccountSharedService
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Implement a business process to fetch \ ``Account``\  object from user name.

Since this process is to be used from Spring Security's Authentication service, interface name would be \ ``AccountSharedService``\  and class name would be \ ``AccountSharedServiceImpl``\ .

.. note::

    This guideline does not recommend calling a Service from another Service.

    To have common domain layer process (service),
    it is recommended to name it as \ ``XxxSharedService``\  instead of \ ``XxxService``\  to indicate that
    it is a service common across various service processes.

    The application created in this tutorial does not require common services.
    However, in general application, it is assumed to have common services for processing the account information.
    Therefore, in this tutorial, process to fetch the account information is implemented as SharedService.

|


| Create \ ``AccountSharedService``\  interface.
| ``src/main/java/com/example/security/domain/service/account/AccountSharedService.java``

.. code-block:: java

    package com.example.security.domain.service.account;

    import com.example.security.domain.model.Account;

    public interface AccountSharedService {
        Account findOne(String username);
    }

|

| Create \ ``AccountSharedServiceImpl``\  class.
| ``src/main/java/com/example/security/domain/service/account/AccountSharedServiceImpl.java``

.. code-block:: java

    package com.example.security.domain.service.account;

    import javax.inject.Inject;

    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;
    import org.terasoluna.gfw.common.exception.ResourceNotFoundException;

    import com.example.security.domain.model.Account;
    import com.example.security.domain.repository.account.AccountRepository;

    @Service
    public class AccountSharedServiceImpl implements AccountSharedService {
        @Inject
        AccountRepository accountRepository;

        @Transactional(readOnly=true)
        @Override
        public Account findOne(String username) {
            // (1)
            Account account = accountRepository.findOne(username);
            // (2)
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
      - | Fetch single \ ``Account``\  object that matches with the user name.
    * - | (2)
      - | If \ ``Account``\  that matches with the user name does not exist, throw \ ``ResourceNotFoundException``\  provided by common library.

|

.. _Tutorial_CreateAuthService:

Creating Authentication Service
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Create a class to store authenticated user information which is used in Spring Security.
| ``src/main/java/com/example/security/domain/service/userdetails/SampleUserDetails.java``

.. code-block:: java

    package com.example.security.domain.service.userdetails;

    import org.springframework.security.core.authority.AuthorityUtils;
    import org.springframework.security.core.userdetails.User;

    import com.example.security.domain.model.Account;

    public class SampleUserDetails extends User { // (1)
        private static final long serialVersionUID = 1L;

        private final Account account; // (2)

        public SampleUserDetails(Account account) {
            // (3)
            super(account.getUsername(), account.getPassword(), AuthorityUtils
                    .createAuthorityList("ROLE_USER")); // (4)
            this.account = account;
        }

        public Account getAccount() { // (5)
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
       - | Implement \ ``org.springframework.security.core.userdetails.UserDetails``\  interface.
         | Here, implement project specific \ ``UserDetails``\  class, by inheriting \ ``org.springframework.security.core.userdetails.User`` \  class that implements \ ``UserDetails``\ .
     * - | (2)
       - | Maintain account information of this project in Spring's authentication user class.
     * - | (3)
       - | Call constructor of \ ``User``\  class. The first argument is user name, the second is password and the third is authority list.
     * - | (4)
       - | As a simple implementation, create an authority having only a role named as \ ``"ROLE_USER"``\ .
     * - | (5)
       - | Create getter of account information. This enables fetching of \ ``Account``\  object of login user.

|

| Create a service to fetch authentication user information which is used in Spring Security.
| ``src/main/java/com/example/security/domain/service/userdetails/SampleUserDetailsService.java``

.. code-block:: java

    package com.example.security.domain.service.userdetails;

    import javax.inject.Inject;

    import org.springframework.security.core.userdetails.UserDetails;
    import org.springframework.security.core.userdetails.UserDetailsService;
    import org.springframework.security.core.userdetails.UsernameNotFoundException;
    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;
    import org.terasoluna.gfw.common.exception.ResourceNotFoundException;

    import com.example.security.domain.model.Account;
    import com.example.security.domain.service.account.AccountSharedService;

    @Service
    public class SampleUserDetailsService implements UserDetailsService { // (1)
        @Inject
        AccountSharedService accountSharedService; // (2)

        @Transactional(readOnly=true)
        @Override
        public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
            try {
                Account account = accountSharedService.findOne(username); // (3)
                return new SampleUserDetails(account); // (4)
            } catch (ResourceNotFoundException e) {
                throw new UsernameNotFoundException("user not found", e); // (5)
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
       - | Implement \ ``org.springframework.security.core.userdetails.UserDetailsService``\  interface.
     * - | (2)
       - | Inject \ ``AccountSharedService``\ .
     * - | (3)
       - | Delegate the process of fetching \ ``Account``\  object from \ ``username``\  to \ ``AccountSharedService``\ .
     * - | (4)
       - | Create project specific \ ``UserDetails``\  object using the fetched \ ``Account``\  object, and return as the return value of method.
     * - | (5)
       - | Throw \ ``UsernameNotFoundException``\  when target user is not found.

|


Setting database initialization script
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

In this tutorial, H2 database (in memory database) is used as a database to store account information.
As a result, database initialization is necessary by executing SQL at the time of starting the application server.

| Add the settings for executing SQL script that is used to initialize the database.
| ``src/main/resources/META-INF/spring/first-springsecurity-env.xml``

.. code-block:: xml
    :emphasize-lines: 4,6,30-36

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:jdbc="http://www.springframework.org/schema/jdbc"
        xsi:schemaLocation="
            http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        ">

        <bean id="dateFactory" class="org.terasoluna.gfw.common.date.jodatime.DefaultJodaTimeDateFactory" />

        <bean id="realDataSource" class="org.apache.commons.dbcp2.BasicDataSource"
            destroy-method="close">
            <property name="driverClassName" value="${database.driverClassName}" />
            <property name="url" value="${database.url}" />
            <property name="username" value="${database.username}" />
            <property name="password" value="${database.password}" />
            <property name="defaultAutoCommit" value="false" />
            <property name="maxTotal" value="${cp.maxActive}" />
            <property name="maxIdle" value="${cp.maxIdle}" />
            <property name="minIdle" value="${cp.minIdle}" />
            <property name="maxWaitMillis" value="${cp.maxWait}" />
        </bean>


        <bean id="dataSource" class="net.sf.log4jdbc.Log4jdbcProxyDataSource">
            <constructor-arg index="0" ref="realDataSource" />
        </bean>

        <!-- (1) -->
        <jdbc:initialize-database data-source="dataSource"
            ignore-failures="ALL">
            <!-- (2) -->
            <jdbc:script location="classpath:/database/${database}-schema.sql" encoding="UTF-8" />
            <!-- (3) -->
            <jdbc:script location="classpath:/database/${database}-dataload.sql" encoding="UTF-8" />
        </jdbc:initialize-database>

        <!--  REMOVE THIS LINE IF YOU USE JPA
        <bean id="transactionManager"
            class="org.springframework.orm.jpa.JpaTransactionManager">
            <property name="entityManagerFactory" ref="entityManagerFactory" />
        </bean>
              REMOVE THIS LINE IF YOU USE JPA  -->
        <!--  REMOVE THIS LINE IF YOU USE MyBatis3
        <bean id="transactionManager"
            class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <property name="dataSource" ref="dataSource" />
            <property name="rollbackOnCommitFailure" value="true" />
        </bean>
              REMOVE THIS LINE IF YOU USE MyBatis3  -->
    </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Perform settings to execute SQL script that initializes the database in \ ``<jdbc:initialize-database>``\  tag.

        Define these settings in \ ``first-springsecurity-env.xml``\ , since they are normally used only during development (environment dependent settings).
    * - | (2)
      - Specify the SQL file where DDL statement for creating a table that stores account information is mentioned.

        As per blank project settings, \ ``H2-schema.sql``\  is executed since \ ``database=H2``\  is defined in \ ``first-springsecurity-infra.properties``\ .
    * - | (3)
      - Specify SQL file, where DML statement to register the demo user is mentioned.

        As per blank project settings, \ ``H2-dataload.sql``\ is executed since \ ``database=H2``\  is defined in \ ``first-springsecurity-infra.properties``\ .

|

| Create DDL statement for creating a table that stores account information.
| ``src/main/resources/database/H2-schema.sql``

.. code-block:: sql

    CREATE TABLE account(
        username varchar(128),
        password varchar(60),
        first_name varchar(128),
        last_name varchar(128),
        constraint pk_tbl_account primary key (username)
    );

|
| Create DML statement to register the demo user (username=demo, password=demo).
| ``src/main/resources/database/H2-dataload.sql``

.. code-block:: sql

    INSERT INTO account(username, password, first_name, last_name) VALUES('demo', '$2a$10$oxSJl.keBwxmsMLkcT9lPeAIxfNTPNQxpeywMrF7A3kVszwUTqfTK', 'Taro', 'Yamada'); -- (1)
    COMMIT;

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - As per blank project settings, \ ``org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder``\  is set as a class to hash the password in \ ``applicationContext.xml``\ .

        In this tutorial, a string called \ ``"demo"``\  that is hashed using BCrypt algorithm is inserted in the password in order to perform password hashing using \ ``BCryptPasswordEncoder``\ .

|

Package Explorer after creating Domain Layer
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Confirm the file created in domain layer.

"Hierarchical" is being used for Package Presentation of Package Explorer.

.. figure:: ./images_Security/security_tutorial-domain-layer-package-explorer.png
   :alt: security tutorial domain layer package explorer

|

Implementing Application Layer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Security settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Perform authentication/authorization settings using Spring Security in \ ``spring-security.xml``\ .

Following are URL patterns to be handled by the application created in this tutorial.

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 30 70
   
   * - | URL
     - | Description
   * - | /login.jsp
     - | URL to display login form
   * - | /login.jsp?error=true
     - | URL to display transition page (login page) in case of authentication error
   * - | /login
     - | URL for authentication
   * - | /logout
     - | URL for logout
   * - | /
     - | URL to display welcome page
   * - | /account
     - | URL to display account information of login user.

|

.. _Tutorial_setting-spring-security:

| Add following settings apart from the settings provided by blank project.
| ``src/main/resources/META-INF/spring/spring-security.xml``

.. code-block:: xml
    :emphasize-lines: 12-15,16-19,23-24,31-33,34-35

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:sec="http://www.springframework.org/schema/security"
        xsi:schemaLocation="
            http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        ">

        <sec:http pattern="/resources/**" security="none"/>
        <sec:http>
            <!-- (1) -->
            <sec:form-login
                login-page="/login.jsp"
                authentication-failure-url="/login.jsp?error=true" />
            <!-- (2) -->
            <sec:logout
                logout-success-url="/"
                delete-cookies="JSESSIONID" />
            <sec:access-denied-handler ref="accessDeniedHandler"/>
            <sec:custom-filter ref="userIdMDCPutFilter" after="ANONYMOUS_FILTER"/>
            <sec:session-management />
            <!-- (3) -->
            <sec:intercept-url pattern="/login.jsp" access="permitAll" />
            <sec:intercept-url pattern="/**" access="isAuthenticated()" />
        </sec:http>

        <sec:authentication-manager>
            <!-- com.example.security.domain.service.userdetails.SampleUserDetailsService
              is scanned by component scan with @Service -->
            <!-- (4) -->
            <sec:authentication-provider
                user-service-ref="sampleUserDetailsService">
                <!-- (5) -->
                <sec:password-encoder ref="passwordEncoder" />
            </sec:authentication-provider>
        </sec:authentication-manager>

        <!-- CSRF Protection -->
        <bean id="accessDeniedHandler"
            class="org.springframework.security.web.access.DelegatingAccessDeniedHandler">
            <constructor-arg index="0">
                <map>
                    <entry
                        key="org.springframework.security.web.csrf.InvalidCsrfTokenException">
                        <bean
                            class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
                            <property name="errorPage"
                                value="/WEB-INF/views/common/error/invalidCsrfTokenError.jsp" />
                        </bean>
                    </entry>
                    <entry
                        key="org.springframework.security.web.csrf.MissingCsrfTokenException">
                        <bean
                            class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
                            <property name="errorPage"
                                value="/WEB-INF/views/common/error/missingCsrfTokenError.jsp" />
                        </bean>
                    </entry>
                </map>
            </constructor-arg>
            <constructor-arg index="1">
                <bean
                    class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
                    <property name="errorPage"
                        value="/WEB-INF/views/common/error/accessDeniedError.jsp" />
                </bean>
            </constructor-arg>
        </bean>

        <!-- Put UserID into MDC -->
        <bean id="userIdMDCPutFilter" class="org.terasoluna.gfw.security.web.logging.UserIdMDCPutFilter">
        </bean>

    </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
  
    * - Sr. No.
      - Description
    * - | (1)
      - Perform settings related to login form using \ ``<sec:form-login>``\  tag.

        Perform following settings in \ ``<sec:form-login>``\  tag

        * URL to display login form in \ ``login-page``\  attribute.
        * URL to display destination page in case of an authentication error in \ ``authentication-failure-url``\  attribute
        * URL to perform authentication in \ ``login-processing-url``\  attribute

    * - | (2)
      - Perform settings for logout using \ ``<sec:logout>``\  tag.

        Perform following settings in \ ``<sec:logout>``\  tag.

        * URL to perform logout in \ ``logout-url``\  attribute
        * URL to display destination page after performing logout in \ ``logout-success-url``\  attribute (URL to display welcome page in this tutorial)
        * Cookie name to be deleted at the time of logout in \ ``delete-cookies``\  attribute (Cookie name of session ID in this tutorial)

    * - | (3)
      - Perform authorization settings for each URL using \ ``<sec:intercept-url>``\  tag.

        Perform following settings in \ ``<sec:intercept-url>``\  tag.

        * \ ``permitAll``\  that allows all the users to access the URL to display login form 
        * \ ``isAuthenticated()``\  that allows only the authenticated users to access the URLs other than the URLs mentioned above

        However, all users can access the URL under \ ``/resources/``\ ,  since the settings are such that authentication/authorization is not performed by Spring Security (\ ``<sec:http pattern="/resources/**" security="none"/>``\ ).
    * - | (4)
      - Perform settings of \ ``org.springframework.security.authentication.AuthenticationProvider``\  that performs authentication using \ ``<sec:authentication-provider>``\  tag.

        By default, \ ``UserDetails``\  is fetched using \ ``UserDetailsService``\ , and a class (\ ``org.springframework.security.authentication.dao.DaoAuthenticationProvider``\ ) that performs user authentication by comparing hashed password in \ ``UserDetails``\  with the password specified in login form, is used.

        Specify component bean name where \ ``UserDetailsService``\  interface is implemented in \ ``user-service-ref``\  attribute. In this tutorial, \ ``SampleUserDetailsService``\  class created in domain layer is set.
    * - | (5)
      - Perform settings for a class (\ ``PasswordEncoder``\ ) to hash the password specified in login form using \ ``<sec:password-encoder>``\  tag.

        In this tutorial, \ ``org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder``\  defined in \ ``applicationContext.xml``\  is used.

.. note::

    Default URL provided by Spring Security is changed for the URLs that perform authentication and logout process.

    This is because, a string (\ ``spring_security``\ ) that implies the usage of Spring Security is included in these URLs.
    When default URL is used as it is and if security vulnerability is detected in Spring Security,
    please be careful as it becomes easy to receive attacks from a malicious user.

|

Creating login page
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Create login form on login page.
| ``src/main/webapp/login.jsp``

.. code-block:: jsp
  
    <!DOCTYPE html>
    <html>
    <head>
    <title>Login Page</title>
    <link rel="stylesheet"
        href="${pageContext.request.contextPath}/resources/app/css/styles.css">
    </head>
    <body>
        <div id="wrapper">
            <h3>Login with Username and Password</h3>

            <!-- (1) -->
            <c:if test="${param.containsKey('error')}">
                <!-- (2) -->
                <t:messagesPanel messagesType="error"
                    messagesAttributeName="SPRING_SECURITY_LAST_EXCEPTION" />
            </c:if>

            <!-- (3) -->
            <form:form action="${pageContext.request.contextPath}/login">
                <table>
                    <tr>
                        <td><label for="username">User:</label></td>
                        <td><input type="text" id="username"
                            name="username" value='demo'>(demo)</td><!-- (4) -->
                    </tr>
                    <tr>
                        <td><label for="password">Password:</label></td>
                        <td><input type="password" id="password"
                            name="password" value="demo" />(demo)</td><!-- (5) -->
                    </tr>
                    <tr>
                        <td>&nbsp;</td>
                        <td><input name="submit" type="submit" value="Login" /></td>
                    </tr>
                </table>
            </form:form>
        </div>
    </body>
    </html>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
  
    * - Sr. No.
      - Description
    * - | (1)
      - When authentication fails, \ ``"/login.jsp?error=true"``\  is called and login page is displayed.
        Therefore, use \ ``<c:if>``\  tag, so that error message is displayed only at the time after the display of authentication error.
    * - | (2)
      - Display an error message using \ ``<t:messagesPanel>``\  tag provided by common library.

        When authentication fails, an exception object of authentication error is stored with attribute name \ ``"SPRING_SECURITY_LAST_EXCEPTION"``\  in session scope.
    * - | (3)
      - Set URL for authentication (\ ``"/login"``\ ) in \ ``action``\  attribute of \ ``<form:form>``\  tag. This URL is default for Spring Security.

        Send parameters necessary for authentication (user name and password) using POST method.
    * - | (4)
      - Create a text box to specify user name.

        Spring Security's default parameter name is \ ``username``\ .
    * - | (5)
      - Create a text box to specify password (text box for password).

        Spring Security's default parameter name is \ ``password``\ .

|

| Ensure that exception object of authentication error stored in session scope could be fetched from JSP.
| ``src/main/webapp/WEB-INF/views/common/include.jsp``

.. code-block:: jsp
    :emphasize-lines: 1

    <%@ page session="true"%> <!-- (6) -->
    <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
    <%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt"%>
    <%@ taglib uri="http://www.springframework.org/tags" prefix="spring"%>
    <%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>
    <%@ taglib uri="http://www.springframework.org/security/tags" prefix="sec"%>
    <%@ taglib uri="http://terasoluna.org/tags" prefix="t"%>
    <%@ taglib uri="http://terasoluna.org/functions" prefix="f"%>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (6)
      - Set \ ``session``\  attribute of \ ``page``\  directive to \ ``true``\ .

.. note::

    As per default settings of blank project, session scope cannot be accessed from JSP.
    This is to ensure that the session cannot be easily used; however,
    in case of fetching an exception object of authentication error from JSP, it is necessary to be accessible from a JSP by session scope.

| 

| Try to display the welcome page by entering  http://localhost:8080/first-springsecurity/ in browser address bar.
| Since the user is not logged in, it is transited to the set value of \ ``login-page``\  attribute of \ ``<sec:form-login>``\  tag (http://localhost:8080/first-springsecurity/login.jsp), and the screen below is displayed.

.. figure:: ./images_Security/security_tutorial_login_page.png
   :width: 80%

Accessing account information of login user from JSP
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Access the account information of login user from JSP and display the name.
| ``src/main/webapp/WEB-INF/views/welcome/home.jsp``

.. code-block:: xml
    :emphasize-lines: 10-11,17-18
  
    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="utf-8">
    <title>Home</title>
    <link rel="stylesheet"
        href="${pageContext.request.contextPath}/resources/app/css/styles.css">
    </head>

    <!-- (1) -->
    <sec:authentication property="principal.account" var="account" />

    <body>
        <div id="wrapper">
            <h1>Hello world!</h1>
            <p>The time on the server is ${serverTime}.</p>
            <!-- (2) -->
            <p>Welcome ${f:h(account.firstName)} ${f:h(account.lastName)} !!</p>
            <ul>
                <li><a href="${pageContext.request.contextPath}/account">view account</a></li>
            </ul>
        </div>
    </body>
    </html>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
  
    * - Sr. No.
      - Description
    * - | (1)
      - Access \ ``org.springframework.security.core.Authentication``\  object of login user using \ ``<sec:authentication>``\  tag.

        By using \ ``property``\  attribute, any property retained by \ ``Authentication``\  object can be accessed , and the property value that has been accessed can be stored in any scope using \ ``var``\  attribute.
        Page scope is set by default and referred within this JSP only.
        
        In this tutorial, \ ``Account``\  object of login user is stored in page scope with attribute name \ ``account``\ .
    * - | (2)
      - Access \ ``Account``\  object of login user and display \ ``firstName``\  and \ ``lastName``\ .

|

Click Login button on login page to display welcome page.

.. figure:: ./images_Security/security_tutorial_welcome_page.png
   :width: 70%

Adding logout button
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Add a button to perform logout.
| ``src/main/webapp/WEB-INF/views/welcome/home.jsp``

.. code-block:: xml
    :emphasize-lines: 18-21

    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="utf-8">
    <title>Home</title>
    <link rel="stylesheet"
        href="${pageContext.request.contextPath}/resources/app/css/styles.css">
    </head>

    <sec:authentication property="principal.account" var="account" />

    <body>
        <div id="wrapper">
            <h1>Hello world!</h1>
            <p>The time on the server is ${serverTime}.</p>
            <p>Welcome ${f:h(account.firstName)} ${f:h(account.lastName)} !!</p>
            <p>
                <!-- (1) -->
                <form:form action="${pageContext.request.contextPath}/logout">
                    <button type="submit">Logout</button>
                </form:form>
            </p>
            <ul>
                <li><a href="${pageContext.request.contextPath}/account">view account</a></li>
            </ul>
        </div>
    </body>
    </html>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Add a form for logout using \ ``<form:form>``\  tag.

        Add Logout button by specifying the URL for logout (\ ``"/logout"``\ ) in \ ``action``\  attribute. This URL is default for Spring Security.

|

Click Logout button to log out from the application (login page is displayed).

.. figure:: ./images_Security/security_tutorial_add_logout.png
    :width: 70%


Accessing account information of login user from Controller
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Access account information of login user from Controller and pass it to View.
| ``src/main/java/com/example/security/app/account/AccountController.java``

.. code-block:: java
    :emphasize-lines: 17,19-21
  
    package com.example.security.app.account;

    import org.springframework.security.core.annotation.AuthenticationPrincipal;
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.web.bind.annotation.RequestMapping;

    import com.example.security.domain.model.Account;
    import com.example.security.domain.service.userdetails.SampleUserDetails;

    @Controller
    @RequestMapping("account")
    public class AccountController {

        @RequestMapping
        public String view(
                @AuthenticationPrincipal SampleUserDetails userDetails, // (1)
                Model model) {
            // (2)
            Account account = userDetails.getAccount();
            model.addAttribute(account);
            return "account/view";
        }
    }
  
.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
  
    * - Sr. No.
      - description
    * - | (1)
      - | Receive \ ``UserDetails``\  object of login user by specifying \ ``@AuthenticationPrincipal``\  annotation.
    * - | (2)
      - | Fetch \ ``Account``\  object which is retained by \ ``SampleUserDetails``\  object and store it in \ ``Model``\  in order to pass it to View.

| 

| Access the account information passed from Controller to display the same.
| ``src/main/webapp/WEB-INF/views/account/view.jsp``

.. code-block:: jsp

    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="utf-8">
    <title>Home</title>
    <link rel="stylesheet"
        href="${pageContext.request.contextPath}/resources/app/css/styles.css">
    </head>
    <body>
        <div id="wrapper">
            <h1>Account Information</h1>
            <table>
                <tr>
                    <th>Username</th>
                    <td>${f:h(account.username)}</td>
                </tr>
                <tr>
                    <th>First name</th>
                    <td>${f:h(account.firstName)}</td>
                </tr>
                <tr>
                    <th>Last name</th>
                    <td>${f:h(account.lastName)}</td>
                </tr>
            </table>
        </div>
    </body>
    </html>

| 

Click 'view account' link on welcome page to display "Show account information" page of login user.

.. figure:: ./images_Security/security_tutorial_account_information_page.png
   :width: 80%

Package explorer after creating application layer
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Confirm the file created in application layer.

"Hierarchical" is being used for Package Presentation of Package Explorer.

.. figure:: ./images_Security/security_tutorial-application-layer-package-explorer.png
   :alt: security tutorial application layer package explorer

|

Summary
--------------------------------------------------------------------------------
We have covered the following topics in this tutorial.

* Basic authentication/authorization using Spring Security
* How to customize authentication user object
* Authentication settings using Repository and Service class
* How to access logged in account information from JSP
* How to access logged in account information from Controller

|

Appendix
--------------------------------------------------------------------------------

.. _SecurityTutorialAppendixConfigurationFiles:

Description of configuration file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Describe configuration file to understand which settings are necessary for using Spring Security.

spring-security.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Perform definitions related to Spring Security in \ ``spring-security.xml``\ .

\ ``src/main/resources/META-INF/spring/spring-security.xml``\ of the blank project which has been created has following settings.

.. code-block:: xml
    :emphasize-lines: 10,13,15,17,19,21,25,28,61

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:sec="http://www.springframework.org/schema/security"
        xsi:schemaLocation="
            http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        ">

        <!-- (1) -->
        <sec:http pattern="/resources/**" security="none"/>
        <sec:http>
            <!-- (2) -->
            <sec:form-login/>
            <!-- (3) -->
            <sec:logout/>
            <!-- (4) -->
            <sec:access-denied-handler ref="accessDeniedHandler"/>
            <!-- (5) -->
            <sec:custom-filter ref="userIdMDCPutFilter" after="ANONYMOUS_FILTER"/>
            <!-- (6) -->
            <sec:session-management />
        </sec:http>

        <!-- (7) -->
        <sec:authentication-manager />

        <!-- (4) -->
        <!-- CSRF Protection -->
        <bean id="accessDeniedHandler"
            class="org.springframework.security.web.access.DelegatingAccessDeniedHandler">
            <constructor-arg index="0">
                <map>
                    <entry
                        key="org.springframework.security.web.csrf.InvalidCsrfTokenException">
                        <bean
                            class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
                            <property name="errorPage"
                                value="/WEB-INF/views/common/error/invalidCsrfTokenError.jsp" />
                        </bean>
                    </entry>
                    <entry
                        key="org.springframework.security.web.csrf.MissingCsrfTokenException">
                        <bean
                            class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
                            <property name="errorPage"
                                value="/WEB-INF/views/common/error/missingCsrfTokenError.jsp" />
                        </bean>
                    </entry>
                </map>
            </constructor-arg>
            <constructor-arg index="1">
                <bean
                    class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
                    <property name="errorPage"
                        value="/WEB-INF/views/common/error/accessDeniedError.jsp" />
                </bean>
            </constructor-arg>
        </bean>

        <!-- (5) -->
        <!-- Put UserID into MDC -->
        <bean id="userIdMDCPutFilter" class="org.terasoluna.gfw.security.web.logging.UserIdMDCPutFilter">
        </bean>

    </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Control authentication/authorization for HTTP access using \ ``<sec:http>``\  tag.

        As per the default settings of blank project , URL to access static resources (js, css, image files, etc.) is out of authentication/authorization scope.
    * - | (2)
      - Control login related operation which use form  authentication, by using \ ``<sec:form-login>``\  tag.

        For usage method, refer to ":ref:`form-login`".
    * - | (3)
      - Control logout related operations by using \ ``<sec:logout>``\  tag.

        For usage method, refer to [:ref:`SpringSecurityAuthenticationLogout`].
    * - | (4)
      - Control action after access is denied using \ ``<sec:access-denied-handler>``\  tag.

        Following settings are performed as default settings of blank project.

        * Destination when invalid CSRF token is detected (when \ ``InvalidCsrfTokenException``\  occurs)
        * Destination when CSRF token cannot be fetched from token store (when \ ``MissingCsrfTokenException``\  occurs)
        * Destination when access is denied in authorization (when \ ``AccessDeniedException``\  other than above mentioned occurs)

    * - | (5)
      - Enable servlet filter to store authentication user name of Spring Security in logger MDC.
        Once this setting is enabled, authentication user name is output in log thereby enhancing the traceability.
    * - | (6)
      - Control session management method of Spring Security using \ ``<sec:session-management>``\  tag.

        For usage method, refer to ":ref:`SpringSecuritySessionManagementSetup`".
    * - | (7)
      - Control authentication using \ ``<sec:authentication-manager>``\  tag.

        For usage method, refer to ":ref:`AuthenticationProviderConfiguration`".


|

spring-mvc.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Perform settings to link Spring Security and Spring MVC in \ ``spring-mvc.xml``\ .

\ ``src/main/resources/META-INF/spring/spring-mvc.xml``\  of the blank project that has been created has following settings.
Description of settings not related to Spring Security is omitted.

.. code-block:: xml
    :emphasize-lines: 22-24,87-89

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xmlns:mvc="http://www.springframework.org/schema/mvc"
        xmlns:util="http://www.springframework.org/schema/util"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
        ">

        <context:property-placeholder
            location="classpath*:/META-INF/spring/*.properties" />

        <mvc:annotation-driven>
            <mvc:argument-resolvers>
                <bean
                    class="org.springframework.data.web.PageableHandlerMethodArgumentResolver" />
                <!-- (1) -->
                <bean
                    class="org.springframework.security.web.method.annotation.AuthenticationPrincipalArgumentResolver" />
            </mvc:argument-resolvers>
            <!-- workaround to CVE-2016-5007. -->
            <mvc:path-matching path-matcher="pathMatcher" />
        </mvc:annotation-driven>

        <mvc:default-servlet-handler />

        <context:component-scan base-package="com.example.security.app" />

        <mvc:resources mapping="/resources/**"
            location="/resources/,classpath:META-INF/resources/"
            cache-period="#{60 * 60}" />

        <mvc:interceptors>
            <mvc:interceptor>
                <mvc:mapping path="/**" />
                <mvc:exclude-mapping path="/resources/**" />
                <mvc:exclude-mapping path="/**/*.html" />
                <bean
                    class="org.terasoluna.gfw.web.logging.TraceLoggingInterceptor" />
            </mvc:interceptor>
            <mvc:interceptor>
                <mvc:mapping path="/**" />
                <mvc:exclude-mapping path="/resources/**" />
                <mvc:exclude-mapping path="/**/*.html" />
                <bean
                    class="org.terasoluna.gfw.web.token.transaction.TransactionTokenInterceptor" />
            </mvc:interceptor>
            <mvc:interceptor>
                <mvc:mapping path="/**" />
                <mvc:exclude-mapping path="/resources/**" />
                <mvc:exclude-mapping path="/**/*.html" />
                <bean class="org.terasoluna.gfw.web.codelist.CodeListInterceptor">
                    <property name="codeListIdPattern" value="CL_.+" />
                </bean>
            </mvc:interceptor>
            <!--  REMOVE THIS LINE IF YOU USE JPA
            <mvc:interceptor>
                <mvc:mapping path="/**" />
                <mvc:exclude-mapping path="/resources/**" />
                <mvc:exclude-mapping path="/**/*.html" />
                <bean
                    class="org.springframework.orm.jpa.support.OpenEntityManagerInViewInterceptor" />
            </mvc:interceptor>
                REMOVE THIS LINE IF YOU USE JPA  -->
        </mvc:interceptors>

        <!-- Settings View Resolver. -->
        <mvc:view-resolvers>
            <mvc:bean-name />
            <mvc:tiles />
            <mvc:jsp prefix="/WEB-INF/views/" />
        </mvc:view-resolvers>

        <mvc:tiles-configurer>
            <mvc:definitions location="/WEB-INF/tiles/tiles-definitions.xml" />
        </mvc:tiles-configurer>

        <bean id="requestDataValueProcessor"
            class="org.terasoluna.gfw.web.mvc.support.CompositeRequestDataValueProcessor">
            <constructor-arg>
                <util:list>
                    <!-- (2) -->
                    <bean
                        class="org.springframework.security.web.servlet.support.csrf.CsrfRequestDataValueProcessor" />
                    <bean
                        class="org.terasoluna.gfw.web.token.transaction.TransactionTokenRequestDataValueProcessor" />
                </util:list>
            </constructor-arg>
        </bean>

        <!-- Setting Exception Handling. -->
        <!-- Exception Resolver. -->
        <bean id="systemExceptionResolver"
            class="org.terasoluna.gfw.web.exception.SystemExceptionResolver">
            <property name="exceptionCodeResolver" ref="exceptionCodeResolver" />
            <!-- Setting and Customization by project. -->
            <property name="order" value="3" />
            <property name="exceptionMappings">
                <map>
                    <entry key="ResourceNotFoundException" value="common/error/resourceNotFoundError" />
                    <entry key="BusinessException" value="common/error/businessError" />
                    <entry key="InvalidTransactionTokenException" value="common/error/transactionTokenError" />
                    <entry key=".DataAccessException" value="common/error/dataAccessError" />
                </map>
            </property>
            <property name="statusCodes">
                <map>
                    <entry key="common/error/resourceNotFoundError" value="404" />
                    <entry key="common/error/businessError" value="409" />
                    <entry key="common/error/transactionTokenError" value="409" />
                    <entry key="common/error/dataAccessError" value="500" />
                </map>
            </property>
            <property name="defaultErrorView" value="common/error/systemError" />
            <property name="defaultStatusCode" value="500" />
        </bean>
        <!-- Setting AOP. -->
        <bean id="handlerExceptionResolverLoggingInterceptor"
            class="org.terasoluna.gfw.web.exception.HandlerExceptionResolverLoggingInterceptor">
            <property name="exceptionLogger" ref="exceptionLogger" />
        </bean>
        <aop:config>
            <aop:advisor advice-ref="handlerExceptionResolverLoggingInterceptor"
                pointcut="execution(* org.springframework.web.servlet.HandlerExceptionResolver.resolveException(..))" />
        </aop:config>

        <!-- Setting PathMatcher. -->
        <bean id="pathMatcher" class="org.springframework.util.AntPathMatcher">
            <property name="trimTokens" value="false" />
        </bean>

    </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Settings to ensure that \ ``UserDetails``\  object of login user is received as Controller argument, by specifying \ ``@AuthenticationPrincipal``\  annotation.

        Specify \ ``AuthenticationPrincipalArgumentResolver``\  in \ ``<mvc:argument-resolvers>``\  tag.
    * - | (2)
      - Settings to embed CSRF token value in HTML form using \ ``<form:form>``\  tag (JSP tag library).

        Specify \ ``CsrfRequestDataValueProcessor``\  in \ ``CompositeRequestDataValueProcessor``\  constructor.


.. raw:: latex

   \newpage

