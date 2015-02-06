Spring Security Tutorial
================================================================================

.. only:: html

.. contents:: Table of Contents
   :depth: 3
   :local:


Introduction
--------------------------------------------------------------------------------

Topics covered in this Tutorial
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* Basic authentication/authorization using Spring Security
* Login using the account information in the database
* Fetching authenticated account object

Target Audience
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* Completed implementation of :doc:`../TutorialTodo/index`\ 
* Understanding of basic operations of Maven


About application to be created
--------------------------------------------------------------------------------

Overview of application
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* Login is possible. Account information for login has been stored in the database.
* There is a Welcome screen and Account information display screen which can be viewed only by the logged in users.
* Logout is possible.

Overview of application is shown in the following figure.

.. figure:: ./images_Tutorial/security_tutorial_applicatioin_overview.png
   :width: 80%

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
      - Displays logged in account information.
    * - 5
      - Logout
      - GET
      - /logout
      - Performs logout (performed by Spring Security)

Creating environment
--------------------------------------------------------------------------------

Creating project
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Create \ `TERASOLUNA Server Framework for Java (5.x) template <https://github.com/terasolunaorg/terasoluna-gfw-web-blank>`_\  using Maven archetype.
| For the method  of importing Spring tool suite and method to start the application server, refer to \ :ref:`CreateProjectFromBlank_create-new-project`\ .

.. code-block:: console

    $ mvn archetype:generate -DarchetypeCatalog=http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases
    [INFO] Scanning for projects...
    [INFO]
    [INFO] ------------------------------------------------------------------------
    [INFO] Building Maven Stub Project (No POM) 1
    [INFO] ------------------------------------------------------------------------
    [INFO]
    [INFO] >>> maven-archetype-plugin:2.2:generate (default-cli) @ standalone-pom >>>
    [INFO]
    [INFO] <<< maven-archetype-plugin:2.2:generate (default-cli) @ standalone-pom <<<
    [INFO]
    [INFO] --- maven-archetype-plugin:2.2:generate (default-cli) @ standalone-pom ---
    [INFO] Generating project in Interactive mode
    [INFO] No archetype defined. Using maven-archetype-quickstart (org.apache.maven.archetypes:maven-archetype-quickstart:1.0)
    Choose archetype:
    1: http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases -> org.terasoluna.gfw.blank:terasoluna-gfw-web-blank-archetype (Blank project using TERASOLUNA Server Framework for Java (5.x))
    2: http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases -> org.terasoluna.gfw.blank:terasoluna-gfw-web-blank-jpa-archetype (Blank project using TERASOLUNA Server Framework for Java (5.x) (JPA))
    3: http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases -> org.terasoluna.gfw.blank:terasoluna-gfw-web-blank-mybatis2-archetype (Blank project using TERASOLUNA Server Framework for Java (5.x) (MyBatis2))

The user is asked to select a type. Select "3" since this option uses MyBatis2 to access data.

.. code-block:: console

    Choose a number or apply filter (format: [groupId:]artifactId, case sensitive contains): : 3


groupId, artifactId, version and package are as follows:

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table::
    :widths: 25 75
    :stub-columns: 1

    * - groupId
      - com.example.security
    * - artifactId
      - first-springsecurity
    * - version
      - 1.0-SNAPSHOT
    * - package
      - com.example.security

.. code-block:: console

    Define value for property 'groupId': : com.example.security
    Define value for property 'artifactId': : first-springsecurity
    Define value for property 'version':  1.0-SNAPSHOT: :
    Define value for property 'package':  com.example.security: :
    Confirm properties configuration:
    groupId: com.example.security
    artifactId: first-springsecurity
    version: 1.0-SNAPSHOT
    package: com.example.security
     Y: :
    [INFO] ----------------------------------------------------------------------------
    [INFO] Using following parameters for creating project from Archetype: terasoluna-gfw-web-blank-mybatis2-archetype:1.0.0.RELEASE
    [INFO] ----------------------------------------------------------------------------
    [INFO] Parameter: groupId, Value: com.example.security
    [INFO] Parameter: artifactId, Value: first-springsecurity
    [INFO] Parameter: version, Value: 1.0-SNAPSHOT
    [INFO] Parameter: package, Value: com.example.security
    [INFO] Parameter: packageInPathFormat, Value: com/example/security
    [INFO] Parameter: package, Value: com.example.security
    [INFO] Parameter: version, Value: 1.0-SNAPSHOT
    [INFO] Parameter: groupId, Value: com.example.security
    [INFO] Parameter: artifactId, Value: first-springsecurity
    [INFO] project created from Archetype in dir: /Users/xxx/first-springsecurity
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 51.891s
    [INFO] Finished at: Mon Dec 02 14:03:11 JST 2013
    [INFO] Final Memory: 13M/116M
    [INFO] ------------------------------------------------------------------------


Creating application
--------------------------------------------------------------------------------

Implementing domain layer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The flow of authentication process of Spring Security is as follows:

#. Search user information from the entered \ ``username``\ .
#. When user information exists, compare the password having user information with the hashed password.
#. When the passwords match, authentication is considered successful.

When user information is not found and if the passwords do not match, authentication fails.

In domain layer, process of fetching the Account object from user name is essential. The process is executed in the following order.

#. Creation of Domain Object(Account)
#. Creation of AccountRepository
#. Creation of AccountService


Use the following for Account table (add post processing for DDL script).

.. code-block:: sql

  CREATE TABLE account(
      username varchar(128),w
      password varchar(128),
      first_name varchar(128),
      last_name varchar(128),
      constraint pk_tbl_account primary key (username)
  );


Creating Domain Object
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Provide the following \ ``Account``\  class. This class contains authentication information (user name and password).

* src/main/java/com/example/security/domain/model/Account.java

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

Creating AccountRepository
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Implement data access logic for fetching Account object from user name in \ ``AccountRepository``\ .


* src/main/java/com/example/security/domain/repository/account/AccountRepository.java

  First, define the interface. Then define \ ``findOne(username)``\ for fetching Account object from user name.

  .. code-block:: java
  
    package com.example.security.domain.repository.account;
  
    import com.example.security.domain.model.Account;
  
    public interface AccountRepository {
        Account findOne(String username);
    }


* src/main/java/com/example/security/domain/repository/account/AccountRepositoryImpl.java

  Implement data access logic in \ ``AccountRepositoryImpl``\ .

  .. code-block:: java
  
    package com.example.security.domain.repository.account;
  
    import javax.inject.Inject;
  
    import org.springframework.stereotype.Repository;
  
    import jp.terasoluna.fw.dao.QueryDAO;
  
    import com.example.security.domain.model.Account;
  
    @Repository
    public class AccountRepositoryImpl implements AccountRepository {
        @Inject
        QueryDAO queryDAO;
  
        @Override
        public Account findOne(String username) {
            Account account = queryDAO.executeForObject("account.findOne",
                    username, Account.class);
            return account;
        }
  
    }

  
* src/main/resources/META-INF/mybatis/sql/account-sqlmap.xml

  Define SQL corresponding to SQLID \ ``"account.findOne"``\  for fetching \ ``Account``\  in SQLMap file.

  .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE sqlMap 
                PUBLIC "-//ibatis.apache.org//DTD SQL Map 2.0//EN"
                "http://ibatis.apache.org/dtd/sql-map-2.dtd">

    <sqlMap namespace="account">
        <resultMap id="account"
            class="com.example.security.domain.model.Account">
            <result property="username" column="username" />
            <result property="password" column="password" />
            <result property="firstName" column="first_name" />
            <result property="lastName" column="last_name" />
        </resultMap>


        <select id="findOne" parameterClass="java.lang.String"
            resultMap="account"><![CDATA[
    SELECT username, 
           password, 
           first_name, 
           last_name 
    FROM   account 
    WHERE  username = #value# 
    ]]></select>
    </sqlMap>

Creation of AccountService
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* src/main/java/com/example/security/domain/service/account/AccountService.java

 Implement business logic for fetching  \ ``Account``\  object from user name in \ ``AccountService``\ .

  This logic is later used from authentication service of Spring Security; hence class name should be \ ``AccountSharedService``\ .


  .. code-block:: java

    package com.example.security.domain.service.account;

    import com.example.security.domain.model.Account;

    public interface AccountSharedService {
        Account findOne(String username);
    }

* src/main/java/com/example/security/domain/service/account/AccountServiceImpl.java

  As a result of data access, if the corresponding \ ``Account``\  does not exist, \ ``ResourceNotFoundException``\  is thrown.

  .. code-block:: java

    package com.example.security.domain.service.account;

    import javax.inject.Inject;

    import org.springframework.stereotype.Service;
    import org.terasoluna.gfw.common.exception.ResourceNotFoundException;

    import com.example.security.domain.model.Account;
    import com.example.security.domain.repository.account.AccountRepository;

    @Service
    public class AccountSharedServiceImpl implements AccountSharedService {
        @Inject
        AccountRepository accountRepository;

        @Override
        public Account findOne(String username) {
            Account account = accountRepository.findOne(username);
            if (account == null) {
                throw new ResourceNotFoundException("The given account is not found! username="
                        + username);
            }
            return account;
        }

    }

Creating Authentication Service
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Implement \ ``org.springframework.security.core.userdetails.UserDetails``\  interface for authentication user information to be used in Spring Security.
Here, create project-specific \ ``UserDetails``\  implementation class by inheriting it from \ ``org.springframework.security.core.userdetails.User`` \ class which in turn is implementation class of \ "UserDetails`` \ interface.


* src/main/java/com/example/security/domain/service/userdetails/SampleUserDetails.java

  .. code-block:: java

    package com.example.security.domain.service.userdetails;

    import java.util.Collection;
    import java.util.Collections;

    import org.springframework.security.core.GrantedAuthority;
    import org.springframework.security.core.authority.SimpleGrantedAuthority;
    import org.springframework.security.core.userdetails.User;

    import com.example.security.domain.model.Account;

    public class SampleUserDetails extends User {
        private static final long serialVersionUID = 1L;

        private final Account account; // (1)

        public SampleUserDetails(Account account) {
            super(account.getUsername(), account.getPassword(), createRole(account)); // (2)
            this.account = account;

        }

        private static Collection<? extends GrantedAuthority> createRole(
                Account account) {
            // sample role
            return Collections
                    .singletonList(new SimpleGrantedAuthority("ROLE_USER")); // (3)
        }

        public Account getAccount() { // (4)
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
       - | Maintain account information of this project in authentication user class of Spring.
     * - | (2)
       - | Call the constructor of \ ``User``\  class. The first argument is user name, the second is password and the third is authority list.
     * - | (3)
       - | As a simple implementation, create an authority containing only the role \ ``"ROLE_USER"``\ .
     * - | (4)
       - | Provide getter of account information. This enables fetching of logged in \ ``Account``\  object.


* src/main/java/com/example/security/domain/service/userdetails/SampleUserDetailsService.java

  .. code-block:: java

    package com.example.security.domain.service.userdetails;

    import javax.inject.Inject;

    import org.springframework.security.core.userdetails.UserDetails;
    import org.springframework.security.core.userdetails.UserDetailsService;
    import org.springframework.security.core.userdetails.UsernameNotFoundException;
    import org.springframework.stereotype.Service;
    import org.terasoluna.gfw.common.exception.ResourceNotFoundException;

    import com.example.security.domain.model.Account;
    import com.example.security.domain.service.account.AccountSharedService;

    @Service
    public class SampleUserDetailsService implements UserDetailsService {
        @Inject
        AccountSharedService accountSharedService; // (1)

        @Override
        public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
            try {
                Account account = accountSharedService.findOne(username); // (2)
                return new SampleUserDetails(account); // (3)
            } catch (ResourceNotFoundException e) {
                throw new UsernameNotFoundException("user not found", e); // (4)
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
       - | Inject \ ``AccountSharedService``\.
         | In this guideline, Service calling is deprecated from Service and is being named as \ ``AccountSharedService``\  instead of \ ``AccountService``\ .
     * - | (2)
       - | Delegate the process of fetching \ ``Account``\ object from \ ``username``\  to \ ``AccountSharedService``\ .
     * - | (3)
       - | Create project specific \ ``UserDetails``\  object using the fetched \ ``Account``\  object.
     * - | (4)
       - | When \ ``UserDetailsService``\  cannot find the user, it throws \ ``UsernameNotFoundException``\ .

Package Explorer after Creating Domain Layer
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Hierarchical option is being used for Package Presentation of Package Explorer.

.. figure:: ./images_Tutorial/security_tutorial-domain-layer-package-explorer.png
   :alt: security tutorial domain layer package explorer
   :width: 40%

Implementation of Application Layer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Security settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Perform authentication/authorization settings using Spring Security in spring-security.xml.

Settings for URL are as follows:

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 30 70
   
   * - | Setting name
     - | Setting value
   * - | URL of login form
     - | /login.jsp
   * - | Destination URL when authentication fails
     - | /login.jsp?error=true
   * - | URL for authentication
     - | /authenticate
   * - | Logout URL
     - | /logout
   * - | Destination URL after logout
     - | /

.. _Tutorial_setting-spring-security:

Only the differences with the blank project are given below.

* src/main/resources/META-INF/spring/spring-security.xml

  .. code-block:: xml
     :emphasize-lines: 11-19,32-35
  
      <?xml version="1.0" encoding="UTF-8"?>
      <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:sec="http://www.springframework.org/schema/security"
          xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd
              http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
              http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
  
          <sec:http pattern="/resources/**" security="none" />
          <sec:http auto-config="true" use-expressions="true">
              <sec:form-login login-page="/login.jsp"
                  authentication-failure-url="/login.jsp?error=true"
                  login-processing-url="/authenticate" /><!-- (1) -->
              <sec:logout logout-url="/logout" logout-success-url="/"
                  delete-cookies="JSESSIONID" /><!-- (2) -->
  
              <sec:intercept-url pattern="/login.jsp"
                  access="permitAll" /><!-- (3) -->
              <sec:intercept-url pattern="/**" access="isAuthenticated()" /><!-- (4) -->
  
              <sec:custom-filter ref="csrfFilter" before="LOGOUT_FILTER" />
              <sec:custom-filter ref="userIdMDCPutFilter"
                  after="ANONYMOUS_FILTER" />
              <sec:session-management
                  session-authentication-strategy-ref="sessionAuthenticationStrategy" />
          </sec:http>
  
  
          <sec:authentication-manager>
              <!-- com.example.security.domain.service.userdetails.SampleUserDetails 
                  is scaned by component scan with @Service -->
              <sec:authentication-provider
                  user-service-ref="sampleUserDetailsService"><!-- (5) -->
                  <sec:password-encoder ref="passwordEncoder" /><!-- (6) -->
              </sec:authentication-provider>
          </sec:authentication-manager>
  
          <!-- CSRF Protection -->
          <bean id="csrfTokenRepository"
              class="org.springframework.security.web.csrf.HttpSessionCsrfTokenRepository" />
  
          <bean id="csrfFilter" class="org.springframework.security.web.csrf.CsrfFilter">
              <constructor-arg index="0" ref="csrfTokenRepository" />
              <property name="accessDeniedHandler">
                  <bean
                      class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
                      <property name="errorPage"
                          value="/WEB-INF/views/common/error/csrfTokenError.jsp" />
                  </bean>
              </property>
          </bean>
  
          <bean id="sessionAuthenticationStrategy"
              class="org.springframework.security.web.authentication.session.CompositeSessionAuthenticationStrategy">
              <constructor-arg index="0">
                  <list>
                      <bean
                          class="org.springframework.security.web.authentication.session.SessionFixationProtectionStrategy" />
                      <bean
                          class="org.springframework.security.web.csrf.CsrfAuthenticationStrategy">
                          <constructor-arg index="0"
                              ref="csrfTokenRepository" />
                      </bean>
                  </list>
              </constructor-arg>
          </bean>
  
          <!-- Put UserID into MDC -->
          <bean id="userIdMDCPutFilter"
              class="org.terasoluna.gfw.security.web.logging.UserIdMDCPutFilter">
              <property name="removeValue" value="true" />
          </bean>
  
      </beans>
  
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - Sr. No.
       - Description
     * - | (1)
       - | Perform settings for login form using \ ``<sec:form-login>``\  tag.
         | Set login form URL in \ ``login-page``\  attribute, destination URL when authentication fails in \ ``authentication-failure-url``\  attribute and URL for authentication in \ ``login-processing-url``\  attribute.
     * - | (2)
       - | Perform settings for logout using \ ``<sec:logout>``\  tag. Set URL for logout in \ ``logout-url``\  attribute and destination URL after logout in \ ``logout-success-url``\  attribute.
         | You can specify a Cookie name to be deleted at the time of logout in \ ``delete-cookies``\  attribute.
     * - | (3)
       - | Perform authorization settings at URL level using \ ``<sec:intercept-url>``\  tag. Specify \ ``permitAll``\  allowing all the users to access the login form.
     * - | (4)
       - | Using this setting, specify \ ``isAuthenticated()``\ that allows access only to authenticated users for all URLs except for \ ``/resources/**``\  and \ ``/login.jsp``\  which are being set above.
     * - | (5)
       - | Perform \ ``org.springframework.security.authentication.AuthenticationProvider``\  settings implementing authentication using \ ``<sec:authentication-provider>``\ tag.
         | As per default settings, \ ``org.springframework.security.authentication.dao.DaoAuthenticationProvider``\  is used. It fetches \ ``UserDetails``\  using \ ``UserDetailsService``\ . Then it authenticates the user by comparing the hashed password from \ ``UserDetails``\ and the user-input password which is hashed using \ ``org.springframework.security.crypto.password.PasswordEncoder``\ .
     * - | (6)
       - | Perform \ ``PasswordEncoder``\  settings. Refer to \ ``org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder``\  defined in applicationContext.xml.


Settings of SQL scripts to be executed at the time of starting the application
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* src/main/resources/META-INF/spring/first-springsecurity-env.xml

  Add settings of SQL scripts.
  
  .. code-block:: xml
     :emphasize-lines: 3-4,31-35
  
      <?xml version="1.0" encoding="UTF-8"?>
      <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jdbc="http://www.springframework.org/schema/jdbc"
          xsi:schemaLocation="http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
              http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
  
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
  
          <bean id="transactionManager"
              class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
              <property name="dataSource" ref="dataSource" />
          </bean>
  
          <jdbc:initialize-database data-source="dataSource"
              ignore-failures="ALL"><!-- (1) -->
              <jdbc:script location="classpath:/database/${database}-schema.sql" /><!-- (2) -->
              <jdbc:script location="classpath:/database/${database}-dataload.sql" /><!-- (3) -->
          </jdbc:initialize-database>
      </beans>
  
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - Sr. No.
       - Description
     * - | (1)
       - | Perform settings of Initialization SQL Scripts using \ ``<jdbc:initialize-database>``\ tag.
         | These settings are normally used only during development; hence define in xxx-env.xml.
     * - | (2)
       - | Set DDL. In template settings, it is defined as \ ``database=H2``\  in xxx-infra.properties; hence H2-schema.sql is executed.
     * - | (3)
       - | Set DML. In template settings, it is defined as \ ``database=H2``\  in xxx-infra.properties; hence H2-dataload.sql is executed. 
       
Currently the in-memory H2 database is to be used. DDL and DML are to be created as follows:

* src/main/resources/database/H2-schema.sql

  .. code-block:: sql

      CREATE TABLE account(
          username varchar(128),
          password varchar(128),
          first_name varchar(128),
          last_name varchar(128),
          constraint pk_tbl_account primary key (username)
      );

* src/main/resources/database/H2-dataload.sql

    Add a test user who can log in to the system using username=demo and password=demo.

  
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
       - | In template settings, \ ``org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder``\  is being set for password hashing in applicationContext.xml.
         | Insert a string “demo” which is hashed using BCrypt algorithm, as test data.

Creating Login Screen
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* src/main/webapp/login.jsp

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
    
            <c:if test="${param.error}"><!-- (1) -->
                <t:messagesPanel messagesType="error"
                    messagesAttributeName="SPRING_SECURITY_LAST_EXCEPTION" /><!-- (2) -->
            </c:if>
    
            <form action="${pageContext.request.contextPath}/authenticate"
                method="POST"><!-- (3) -->
                <table>
                    <tr>
                        <td><label for="j_username">User:</label></td>
                        <td><input type="text" id="j_username"
                            name="j_username" value='demo'>(demo)</td><!-- (4) -->
                    </tr>
                    <tr>
                        <td><label for="j_password">Password:</label></td>
                        <td><input type="password" id="j_password"
                            name="j_password" value="demo" />(demo)</td><!-- (5) -->
                    </tr>
                    <tr>
                        <td>&nbsp;</td>
                        <td><input type="hidden"
                            name="${f:h(_csrf.parameterName)}"
                            value="${f:h(_csrf.token)}" /> <input
                            name="submit" type="submit" value="Login" /></td><!-- (6) -->
                    </tr>
                </table>
            </form>
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
       - | When authentication fails, settings are performed to call "/login.jsp?error=true". In this case, to display only the error message, \ ``<c:if>``\  tag should be used.
     * - | (2)
       - | When authentication fails, exception object is stored with attribute name \ ``"SPRING_SECURITY_LAST_EXCEPTION"``\  in session scope.
         | Here, error message should be displayed using \ ``<t:messagesPanel>``\ tag.
     * - | (3)
       - | URL for authentication is set as "/authenticate". User name and password should be posted using this URL for authentication.
     * - | (4)
       - | Request parameter name of user name is \ ``j_username``\  by default.
     * - | (5)
       - | Request parameter name of password is \ ``j_password``\  by default.

| 

When an attempt is made to display the login screen by entering http://localhost:8080/first-springsecurity/ in the address bar of the browser, since the user is not logged-in, http://localhost:8080/first-springsecurity/login.jsp is accessed as per <Tutorial_setting-spring-security>` (1) definition of :ref:`Spring Security settings, and the screen below is displayed.

.. figure:: ./images_Tutorial/security_tutorial_login_page.png
   :width: 80%

Accessing Login account information from JSP
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* src/main/webapp/WEB-INF/views/welcome/home.jsp

  Add the following code.

  .. code-block:: xml
     :emphasize-lines: 11-18
  
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
              <sec:authentication property="principal.account" var="account" /><!-- (1) -->
              <h1>Hello world!</h1>
              <p>Welcome ${f:h(account.firstName)} ${f:h(account.lastName)}</p><!-- (2) -->
  
              <ul>
                  <li><a href="${pageContext.request.contextPath}/account">view account</a></li>
                  <li><a href="${pageContext.request.contextPath}/logout">logout</a></li>
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
       - | It is possible to access logged in \ ``org.springframework.security.core.Authentication``\  object using \ ``<sec:authentication>``\  tag.
         | Any property of \ ``.Authentication``\  object can be accessed using \ ``property``\  attribute and can be set in any scope using \ ``var``\  attribute. Page scope is set by default to enable the user to browse only in this JSP.
         | Store logged in \ ``Account``\  object in variable name \ ``account``\ .
     * - | (2)
       - | Access logged in \ ``Account``\  object to display \ ``firstName``\  and \ ``lastName``\ .

| 

Welcome page is displayed on clicking Login button on Login page.

.. figure:: ./images_Tutorial/security_tutorial_welcome_page.png
   :width: 80%


Creation of Login Account Information Display Page
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* src/main/java/com/example/security/app/account/AccountController.java

  The logged in \ ``UserDetails``\  object is stored in \ ``java.security.Principal``\  object. By receiving \ ``Principal``\  object in the processing method argument of Controller, it is possible to access the logged-in \ ``UserDetails``\  in the Controller.

  .. code-block:: java
  
      package com.example.security.app.account;
  
      import java.security.Principal;
  
      import org.springframework.security.core.Authentication;
      import org.springframework.stereotype.Controller;
      import org.springframework.ui.Model;
      import org.springframework.web.bind.annotation.RequestMapping;
      import org.springframework.web.bind.annotation.RequestMethod;
  
      import com.example.security.domain.model.Account;
      import com.example.security.domain.service.userdetails.SampleUserDetails;
  
      @Controller
      @RequestMapping("account")
      public class AccountController {
  
          @RequestMapping(method = RequestMethod.GET)
          public String view(/* (1) */ Principal principal, Model model) {
              // get login user information
              Authentication authentication = (Authentication) principal; // (2)
              // get UserDetails
              SampleUserDetails userDetails = (SampleUserDetails) authentication
                      .getPrincipal(); // (3)
              // get account object
              Account account = userDetails.getAccount(); // (4)
              model.addAttribute(account);
              return "account/view";
          }
      }
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - Sr. No.
       - Description
     * - | (1)
       - | Receive \ ``Principal``\ object that stores logged in \ ``UserDetails``\  object.
     * - | (2)
       - | \ ``org.springframework.security.core.Authentication``\  also has \ ``Principal``\  interface and \ ``Principal``\  object passed to the controller is actually \ ``Authentication``\  object.
         | In order to access the \ ``UserDetails``\  object, perform casting in \ ``Authentication``\  class.
     * - | (3)
       - | It is possible to fetch the logged in \ ``UserDetails``\  object using \ ``Authentication.getPrincipal()``\  method. Perform casting in project specific \ ``SampleUserDetails``\  class.
     * - | (4)
       - | Fetch logged in \ ``Account``\  object from \ ``SampleUserDetails``\  object.

| 

* src/main/webapp/WEB-INF/views/account/view.jsp

  Description is omitted since only property of \ ``Account``\  object set in Model is output.
  
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

Account Information page is displayed on clicking view account link on Welcome page.

.. figure:: ./images_Tutorial/security_tutorial_account_information_page.png
   :width: 80%


Package explorer after creating application layer
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. figure:: ./images_Tutorial/security_tutorial-application-layer-package-explorer.png
   :alt: security tutorial application layer package explorer
   :width: 40%

Summary
--------------------------------------------------------------------------------
The tutorial covered the following topics:

* Basic authentication/authorization using Spring Security
* Customization of authentication user object
* Authentication settings using Repository and Service class
* Accessing logged in account information from JSP
* Accessing logged in account information from Controller

.. raw:: latex

   \newpage

