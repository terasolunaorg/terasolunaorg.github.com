Database Access (Common)
================================================================================

.. only:: html

 .. contents:: Table of Contents
    :local:
    :depth: 3


.. todo::

    **TBD**

    The following topics in this chapter are currently under study.

    * | About multiple datasources
      | For details, \ :ref:`About multiple datasources of Overview <data-access-common_todo_multiple_datasource_overview>`\  and \ :ref:`Settings for using multiple datasources of How to extends <data-access-common_todo_multiple_datasource_howtoextends>`\ .
    * | About handling of unique constraint errors and pessimistic exclusion errors when using JPA
      | For details, refer to \ :ref:`Overview - About exception handling <data-access-common_todo_exception>`\ .


Overview
--------------------------------------------------------------------------------

This chapter explains the method of accessing the data stored in RDBMS.

For the O/R Mapper dependent part, refer to

* \ :doc:`DataAccessMyBatis3`\
* \ :doc:`DataAccessJpa`\


About JDBC DataSource
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| RDBMS can be accessed from the application by referring to JDBC datasource.
| JDBC datasource can be used to exclude settings such as JDBC driver load, connection information (URL, user, password etc.) from application.
| Therefore, RDBMS to be used and environment in which the application is to be deployed need not be taken into account.

 .. figure:: images/dataaccess_common-datasource.png
    :alt: about data source
    :width: 90%
    :align: center

    **Picture - About JDBC DataSource**
    
| JDBC datasource is implemented from Application Server, OSS library, Third-Party library, Spring Framework etc.; hence it is necessary to select the datasource based on project requirements and deployment environment.
| The typical datasources are introduced below.

 * :ref:`datasource_application_server-label`
 * :ref:`datasource_oss_thirdparty-label`
 * :ref:`datasource_spring_framework-label`


.. _datasource_application_server-label:

JDBC datasource provided by Application Server
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| When datasource is to be used in Web application, normally JDBC datasource provided by Application Server is used.
| JDBC datasource of Application Server provides functionalities required in web application such as Connection Pooling as standard functionalities.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table:: **Datasources provided by Application Server**
    :header-rows: 1
    :widths: 10 35 55

    * - Sr. No.
      - Application Server
      - Reference page
    * - 1.
      - Apache Tomcat 8
      - | Refer to \ `Apache Tomcat 8 User Guide(The Tomcat JDBC Connection Pool) <http://tomcat.apache.org/tomcat-8.0-doc/jdbc-pool.html>`_\ .
        | Refer to \ `Apache Tomcat 8 User Guide(JNDI Datasource HOW-TO) <http://tomcat.apache.org/tomcat-8.0-doc/jndi-datasource-examples-howto.html>`_\  (Apache Commons DBCP 2).
    * - 2.
      - Apache Tomcat 7
      - | Refer to \ `Apache Tomcat 7 User Guide (The Tomcat JDBC Connection Pool) <http://tomcat.apache.org/tomcat-7.0-doc/jdbc-pool.html>`_\ .
        | Refer to \ `Apache Tomcat 7 User Guide (JNDI Datasource HOW-TO) <http://tomcat.apache.org/tomcat-7.0-doc/jndi-datasource-examples-howto.html>`_\  (Apache Commons DBCP).
    * - 3.
      - Oracle WebLogic Server 12c
      - Refer to \ `Oracle WebLogic Server Product Documentation <http://docs.oracle.com/middleware/1221/wls/INTRO/jdbc.htm>`_\ .
    * - 4.
      - IBM WebSphere Application Server Version 8.5
      - Refer to \ `WebSphere Application Server Online information center <http://www.ibm.com/support/knowledgecenter/SSEQTP_8.5.5/com.ibm.websphere.wlp.doc/ae/twlp_dep_configuring_ds.html?lang=en>`_\ .
    * - 5.
      - JBoss Enterprise Application Platform 6.4
      - Refer to \ `JBoss Enterprise Application Platform Product Documentation <https://access.redhat.com/documentation/en-US/JBoss_Enterprise_Application_Platform/6.4/html/Administration_and_Configuration_Guide/chap-Datasource_Management.html>`_\ .


.. _datasource_oss_thirdparty-label:

JDBC datasource provided by OSS/Third-Party library
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| When JDBC datasource of Application Server is not used, JDBC datasource of OSS/Third-Party library should be used.
| This guideline introduces only Apache Commons DBCP; however other libraries can also be used.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table:: **JDBC datasource provided by OSS/Third-Party library**
    :header-rows: 1
    :widths: 10 35 55

    * - Sr. No.
      - Library name
      - Description
    * - 1.
      - Apache Commons DBCP
      - Refer to \ `Apache Commons DBCP <http://commons.apache.org/proper/commons-dbcp/index.html>`_\ .


.. _datasource_spring_framework-label:

JDBC datasource provided by Spring Framework
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Implementation class of JDBC datasource of Spring Framework cannot be used as datasource of Web application since it does not provide connection pooling.
| In Spring Framework, implementation class and adapter class of JDBC datasource are provided; however they are introduced as  \ :ref:`appendix_datasource_of_spring-label`\  of Appendix, since usage is restricted.


About transaction management
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| When transactions are to be stored using Spring Framework functionality, PlatformTransactionManager needs to be selected based on project requirements and deployment environment.
| For details, refer to \ :ref:`service_enable_transaction_management`\  of \ :doc:`../ImplementationAtEachLayer/DomainLayer`\ .


About declaration of transaction boundary/attribute
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Transaction boundary and transaction attributes should be declared by specifying  \ ``@Transactional``\  annotation in Service.
| For details, refer to \ :ref:`service_transaction_management`\  of \ :doc:`../ImplementationAtEachLayer/DomainLayer`\  .


About exclusion control of data
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| When updating data, it is necessary to execute exclusion control to ensure data consistency and integrity.
| For details on exclusion control of data, refer to \ :doc:`ExclusionControl`\ .


About exception handling
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| In Spring Framework, a function is provided to convert JDBC exception (\ ``java.sql.SQLException``\ ) and O/R Mapper specific exception to data access exception (subclass of (\ ``org.springframework.dao.DataAccessException``\ ) provided by Spring Framework.
| For the class which is converted to data access exception of Spring Framework, refer to \ :ref:`appendix_dataaccessexception_converter_class-label`\  of Appendix.

| The converted data access exception need not be handled in application code; however, some errors (such as unique constraint violation, exclusion error etc.) need to be handled as per the requirements.
| When handling data access exception, exception of subclass notifying error details should be caught instead of \ ``DataAccessException``\ .
| Typical subclasses which are likely to be handled in application code are as follows:

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table:: **Subclasses of DB access exception, which are likely to be handled**
    :header-rows: 1
    :widths: 10 35 55

    * - Sr. No.
      - Class name
      - Description
    * - 1.
      - | org.springframework.dao.
        | DuplicateKeyException
      - | Exception that occurs in case of unique constraint violation.
    * - 2.
      - | org.springframework.dao.
        | OptimisticLockingFailureException
      - | Exception that occurs in case of optimistic locking failure. It occurs when same data is updated with different logic.
        | This exception occurs when JPA is used as O/R Mapper. MyBatis does not have optimistic locking function; hence this exception does not occur from O/R Mapper.
    * - 3.
      - | org.springframework.dao.
        | PessimisticLockingFailureException
      - | Exception that occurs in case of pessimistic locking failure. It occurs when same data is locked with different logic and the lock is not released even after "waiting for unlocking" timeout period has elapsed.

 .. note::

    When optimistic locking is to be implemented using MyBatis in O/R Mapper, it should be implemented as Service or Repository process.

    As a method of notifying the optimistic locking failure to Controller, this guideline recommends generation of \ ``OptimisticLockingFailureException``\  and exception of its child class.

    This is to make implementation of application layer (implementation of Controller) independent of O/R Mapper to be used.


.. _data-access-common_todo_exception:

 .. todo::

    **It has been recently found that using JPA (Hibernate) results in occurrence of unexpected errors.**

    * In case of unique constraint violation, \ ``org.springframework.dao.DataIntegrityViolationException``\  occurs and not \ ``DuplicateKeyException``\ .


See the example below for handling unique constraint violation as business exception.

 .. code-block:: java

     try {
         accountRepository.saveAndFlash(account);
     } catch(DuplicateKeyException e) { // (1)
         throw new BusinessException(ResultMessages.error().add("e.xx.xx.0002"), e); // (2)
     }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Exception (DuplicateKeyException) that occurs in case of unique constraint violation is caught.
    * - | (2)
      - | Business exception indicating that there is duplicate data is thrown.
        | When exception is caught, make sure to specify the cause of exception (\ ``e``\ ) in business exception.

About multiple datasources
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Multiple datasources may be required depending on the application.
| Typical cases wherein multiple datasources are required, are shown below.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.30\linewidth}|p{0.30\linewidth}|
 .. list-table:: **Typical case where multiple datasources are required**
    :header-rows: 1
    :widths: 10 30 30 30

    * - Sr. No.
      - Case
      - Example
      - Feature
    * - 1.
      - When database and schema are divided according to data (tables).
      - When group of tables maintaining customer information and group of tables maintaining invoice information are stored in separate database and schema.
      - The data to be handled in the process is fixed; hence the datasource to be used can be defined statically.
    * - 2.
      - When database and schema to be used are divided according to users (login users).
      - When database and schema are divided according to users (Multitenant etc.).
      - The datasource to be used differs depending on users; hence the datasource to be used dynamically can be defined.

 .. _data-access-common_todo_multiple_datasource_overview:

 .. todo::

    **TBD**

    The following details will be added in future.

    * Conceptual diagram


About common library classes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Common library provides classes that carry out following processes.
| For more details about common library, refer to links given below.

* :ref:`data-access-common_appendix_like_escape`
* :ref:`data-access-common_appendix_sequencer`

|

How to use
--------------------------------------------------------------------------------

.. _data-access-common_howtouse_datasource:

Datasource settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Settings when using DataSource defined in Application Server
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| When using datasource defined in Application Server, it is necessary to perform settings in Bean definition file to register the object fetched through JNDI as a bean. 
| Settings when PostgreSQL is used as database and Tomcat7 is used as Application Server are given below.

- :file:`xxx-context.xml` (Tomcat config file)

  .. code-block:: xml

    <!-- (1) -->
    <Resource
       type="javax.sql.DataSource"
       name="jdbc/SampleDataSource"
       driverClassName="org.postgresql.Driver"
       url="jdbc:postgresql://localhost:5432/terasoluna"
       username="postgres"
       password="postgres"
       defaultAutoCommit="false"
       /> <!-- (2) -->

- :file:`xxx-env.xml`

 .. code-block:: xml

    <jee:jndi-lookup id="dataSource" jndi-name="jdbc/SampleDataSource" /> <!-- (3) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 10 80

    * - Sr. No.
      - Attribute name
      - Description
    * - | (1)
      - \-
      - Define datasource.
    * - |
      - type
      - Specify resource type. Specify \ ``javax.sql.DataSource``\ .
    * - |
      - name
      - Specify resource name. The name specified here is JNDI name.
    * - |
      - driverClassName
      - Specify JDBC driver class. In the example, JDBC driver class provided by PostgreSQL is specified.
    * - |
      - url
      - Specify URL. [Needs to be changed as per environment]
    * - |
      - username
      - Specify user name. [Needs to be changed as per environment]
    * - |
      - password
      - Specify password of user. [Needs to be changed as per environment]
    * - |
      - defaultAutoCommit
      - Specify default value of auto commit flag. Specify 'false'. It is forcibly set to 'false' when it is under Transaction Management.
    * - | (2)
      - \-
      - | In case of Tomcat7, tomcat-jdbc-pool is used if factory attribute is omitted.
        | For more details about settings, refer to \ `Attributes of The Tomcat JDBC Connection Pool <http://tomcat.apache.org/tomcat-7.0-doc/jdbc-pool.html#Attributes>`_\ .
    * - | (3)
      - \-
      - Specify JNDI name of datasource. In case of Tomcat, specify the value specified in resource name "(1)-name" at the time of defining datasource.


Settings when using DataSource for which Bean is defined
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| When using datasource of OSS/Third-Party library or JDBC datasource of Spring Framework without using the datasource provided by Application Server, 
| bean for DataSource class needs to be defined in Bean definition file.
| Settings when PostgreSQL is used as database and Apache Commons DBCP is used as datasource are given below.

- :file:`xxx-env.xml`

 .. code-block:: xml

    <bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource"
        destroy-method="close">                                           <!-- (1) (8) -->
        <property name="driverClassName" value="org.postgresql.Driver" /> <!-- (2) -->
        <property name="url" value="jdbc:postgresql://localhost:5432/terasoluna" /> <!-- (3) -->
        <property name="username" value="postgres" />                     <!-- (4) -->
        <property name="password" value="postgres" />                     <!-- (5) -->
        <property name="defaultAutoCommit" value="false"/>               <!-- (6) -->
        <!-- (7) -->
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Specify implementation class of datasource. In the example, datasource class (\ ``org.apache.commons.dbcp2.BasicDataSource``\ ) provided by Apache Commons DBCP is specified.
    * - | (2)
      - Specify JDBC driver class. In the example, JDBC driver class provided by PostgreSQL is specified.
    * - | (3)
      - Specify URL. [Needs to be changed as per environment]
    * - | (4)
      - Specify user name. [Needs to be changed as per environment]
    * - | (5)
      - Specify password of user. [Needs to be changed as per environment]
    * - | (6)
      - Specify default value of auto commit flag. Specify 'false'. It is forcibly set to 'false' when it is under Transaction Management.
    * - | (7)
      - | In BasicDataSource, configuration values common in JDBC, JDBC driver specific properties values, connection pooling configuration values can be specified other than the values mentioned above.
        | For more details about settings, refer to \ `DBCP Configuration <http://commons.apache.org/proper/commons-dbcp/configuration.html>`_\ .
    * - | (8)
      - | In the example, values are specified directly; however, for fields where configuration values change with the environment, actual configuration values should be specified in properties file using Placeholder(${...}).
        | For Placeholder, refer to \ ``PropertyPlaceholderConfigurer``\  of \ `Spring Reference Document <http://docs.spring.io/spring/docs/4.2.4.RELEASE/spring-framework-reference/html/beans.html#beans-factory-extension-factory-postprocessors>`_\ .


Settings to enable transaction management
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
For basic settings to enable transaction management, refer to \ :ref:`service_enable_transaction_management`\  of \ :doc:`../ImplementationAtEachLayer/DomainLayer`\ .

For PlatformTransactionManager, the class to be used changes depending on the O/R Mapper used; hence for detailed settings, refer to:

* \ :doc:`DataAccessMyBatis3`\
* \ :doc:`DataAccessJpa`\


.. _DataAccessCommonDataSourceDebug:

JDBC debug log settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| When more detailed information than the log output using O/R Mapper(MyBatis, Hibernate) is required, the information output using log4jdbc(log4jdbc-remix) can be used.
| For details on log4jdbc, refer to \ `log4jdbc project page <https://code.google.com/p/log4jdbc/>`_\ .
| For details on log4jdbc-remix, refer to \ `log4jdbc-remix project page <https://code.google.com/p/log4jdbc-remix/>`_\ .

\

 .. warning::

    **When Log4jdbcProxyDataSource offered by log4jdbc-remix is used, substantial overheads are likely to occur even if the log level is set in the configuration other than debug.**
    **Therefore, it is recommended to use this setting for debugging, and connect to database without passing through Log4jdbcProxyDataSource while its release during performance test enviroment and commercial environment.**


Settings related to datasource provided by log4jdbc
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

- :file:`xxx-env.xml`

 .. code-block:: xml

    <jee:jndi-lookup id="dataSourceSpied" jndi-name="jdbc/SampleDataSource" /> <!-- (1) -->

    <bean id="dataSource" class="net.sf.log4jdbc.Log4jdbcProxyDataSource"> <!-- (2) -->
        <constructor-arg ref="dataSourceSpied" /> <!-- (3) -->
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Define actual datasource. In the example, the datasource fetched through JNDI from Application Server is being used.
    * - | (2)
      - Specify \ ``net.sf.log4jdbc.Log4jdbcProxyDataSource``\  provided by log4jdbc.
    * - | (3)
      - In constructor, specify bean which is an actual datasource.

 .. warning::

    **When the application is to be released in performance test environment or production environment, Log4jdbcProxyDataSource should not be used as datasource.**

    Specifically, exclude settings of (2) and (3) and change bean name of \ ``"dataSourceSpied"``\  to \ ``"dataSource"``\ .


log4jdbc logger settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

- :file:`logback.xml`

 .. code-block:: xml

    <!-- (1) -->
    <logger name="jdbc.sqltiming">
        <level value="debug" />
    </logger>

    <!-- (2) -->
    <logger name="jdbc.sqlonly">
        <level value="warn" />
    </logger>

    <!-- (3) -->
    <logger name="jdbc.audit">
        <level value="warn" />
    </logger>

    <!-- (4) -->
    <logger name="jdbc.connection">
        <level value="warn" />
    </logger>

    <!-- (5) -->
    <logger name="jdbc.resultset">
        <level value="warn" />
    </logger>

    <!-- (6) -->
    <logger name="jdbc.resultsettable">
        <level value="debug" />
    </logger>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Logger to output SQL execution time and SQL statement wherein the value is set in bind variable. Since this SQL contains values for bind variables, it can be executed using DB access tool.
    * - | (2)
      - | Logger to output SQL statement wherein the value is set in bind variable. The difference with (1) is that SQL execution time is not output.
    * - | (3)
      - | Logger to exclude ResultSet interface, call methods of JDBC interface and to output arguments and return values. This log is useful for analyzing the JDBC related issues; however volume of the output log is large.
    * - | (4)
      - | Logger to output connected/disconnected events and number of connections in use. This log is useful for analyzing connection leak, but it need not be output unless there is connection leak issue.
    * - | (5)
      - | Logger to call methods of ResultSet interface and output arguments and return values. This log is useful during analysis when actual result differs from expected result; however volume of the output log is large.
    * - | (6)
      - | Logger to output the contents of ResultSet by converting them into a format so that they can be easily verified. This log is useful during analysis when actual result differs from expected result; however volume of the output log is large.

 .. warning::

    **Large amount of log is output depending on the type of logger; hence only the required logger should be defined or output.**

    In the above sample, log level for loggers which output very useful logs during development, is set to \ ``"debug"``\ .
    As for other loggers, the log level needs to be set to \ ``"debug"``\  whenever required.

    **When the application is to be released in performance test environment or production environment, log using log4jdbc logger should not be output at the time of normal end of process.**

    Typically log level should be set to \ ``"warn"``\ .


Settings of log4jdbc option
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Default operations of log4jdbc can be customized by placing properties file \ :file:`log4jdbc.properties`\ under class path.

- :file:`log4jdbc.properties`

 .. code-block:: properties

     # (1)
     log4jdbc.dump.sql.maxlinelength=0
     # (2)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Specify word-wrap setting for SQL statement. If '0' is specified, SQL statement is not wrapped.
    * - | (2)
      - For option details, refer to \ `log4jdbc project page -Options- <https://code.google.com/p/log4jdbc/#Options>`_\ .

|

How to extend
--------------------------------------------------------------------------------

.. _data-access-common_todo_multiple_datasource_howtoextends:

Settings for using multiple datasources
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

 .. todo::

    **TBD**

    Following details will be added in future.

    * Transaction management method may change depending on the processing pattern (like Update for multiple datasources, Update for a single datasource, Only for reference, No concurrent access etc.), hence breakdown is planned focusing on that area.


Settings to switch the datasource dynamically
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| In order to define multiple datasources and then to switch them dynamically, it is necessary to create a class that inherits \ ``org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource``\  and implement the conditions by which datasource is switched.
| This is to be implemented by mapping the key which is a return value of \ ``determineCurrentLookupKey``\  method with the datasource. For selecting the key, usually context information like authenticated user information, time and locale etc. is to be used.

Implementation of AbstractRoutingDataSource
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| The datasource can be switched dynamically by using the \ ``DataSource``\  which is created by extending \ ``AbstractRoutingDataSource``\ in a same way as the normal datasource.
| The example of switching the datasource based on time is given below.

- Example of implementing a class that inherits \ ``AbstractRoutingDataSource``\ 

 .. code-block:: java

    package com.examples.infra.datasource;

    import javax.inject.Inject;

    import org.joda.time.DateTime;
    import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;
    import org.terasoluna.gfw.common.date.jodatime.JodaTimeDateFactory;

    public class RoutingDataSource extends AbstractRoutingDataSource { // (1)

        @Inject
        JodaTimeDateFactory dateFactory; // (2)

        @Override
        protected Object determineCurrentLookupKey() { // (3)

            DateTime dateTime = dateFactory.newDateTime();
            int hour = dateTime.getHourOfDay();

            if (7 <= hour && hour <= 23) { // (4)
                return "OPEN"; // (5)
            } else {
                return "CLOSE";
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
      - Inherit \ ``AbstractRoutingDataSource``\ .
    * - | (2)
      - Use \ ``JodaTimeDateFactory``\ to fetch time. For details, refer to \ :doc:`./SystemDate`\ .
    * - | (3)
      - Implement \ ``determineCurrentLookupKey``\  method. The datasource to be used is defined by mapping the return value of this method and the \ ``key``\  defined in \ ``targetDataSources``\  of the bean definition file described later.
    * - | (4)
      - In the method, refer to the context information (here Time) and switch the key. Here the implementation should be in accordance with the business requirements. This sample is being implemented so that the time returns different keys as "From 7:00 to 23:59" and  "From 0:00 to 6:59".
    * - | (5)
      - Return the  \ ``key``\  to be mapped with \ ``targetDataSources``\  of the bean definition file described later.

.. note

    When switching the datasource based on authenticated user information (ID or privileges), it is advisable to fetch it using \ ``org.springframework.security.core.context.SecurityContext``\  in \ ``determineCurrentLookupKey``\  method.
    For details on \ ``org.springframework.security.core.context.SecurityContext``\  class, refer to \ :doc:`../Security/Authentication`\ .

Datasource definition
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Define the  \ ``AbstractRoutingDataSource``\ extended class which was created, in bean definition file.

- :file:`xxx-env.xml`

 .. code-block:: xml

    <bean id="dataSource"
        class="com.examples.infra.datasource.RoutingDataSource">  <!-- (1) -->
        <property name="targetDataSources">  <!-- (2) -->
            <map>
                <entry key="OPEN" value-ref="dataSourceOpen" />
                <entry key="CLOSE" value-ref="dataSourceClose" />
            </map>
        </property>
        <property name="defaultTargetDataSource" ref="dataSourceDefault" />  <!-- (3) -->
    </bean>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Define a class that inherits \ ``AbstractRoutingDataSource``\  created earlier.
    * - | (2)
      - Define the datasource to be used. As for \ ``key``\ , define the value that can be returned using \ ``determineCurrentLookupKey``\  method. In \ ``value-ref``\  , specify the datasource to be used for each \ ``key``\ . Define in accordance with the number of datasources to be switched based on \ :ref:`Datasource settings <data-access-common_howtouse_datasource>`\ .
    * - | (3)
      - This datasource is used, when \ ``key``\  specified in \ ``determineCurrentLookupKey``\  method does not exist in \ ``targetDataSources``\ . In case of implementation example, default setting is not used; however, this time \ ``defaultTargetDataSource``\  is being used for description purpose.


|

how to solve the problem
--------------------------------------------------------------------------------
|

.. _data-access-common_howtosolve_n_plus_1:

How to resolve N+1
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
N+1 occurs when more number of SQL statements need to be executed in accordance with the number of records to be fetched from the database. This problem causes high load on the database and deteriorates response time. 

Details are given below.

 .. figure:: images/dataaccess_common-n_plus_1.png
    :alt: about N+1 Problem
    :width: 90%
    :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Search the records matching the search conditions from MainTable.
        | In the above example, col1 of MainTable fetches \ ``'Foo'``\  records and the total records fetched are 20.
    * - | (2)
      - | For each record searched in (1), related records are fetched from SubTable. 
        | In the above example, the id column of SubTable fetches the same records as the id column of records fetched in (1).
        | **This SQL is executed for number of records fetched in (1).**

 | In the above example, \ **SQL is executed totally 21 times.**\
 | Supposing there are 3 SubTables, \ **SQL is executed totally 61 times; hence countermeasures are required.**\


Typical example to resolve N+1 is given below.


Resolving N+1 using JOINs (Join Fetch)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| By performing JOIN on SubTable and MainTable, records of MainTable and SubTable are fetched by executing SQL once.
| When relation of MainTable and SubTable is 1:1, check whether N+1 can be resolved using this method.

 .. figure:: images/dataaccess_common-n_plus_1_solve_join.png
    :alt: about solve N+1 Problem using JOIN
    :width: 90%
    :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | When searching records matching the search conditions, the records are fetched in batch from MainTable and SubTable, by performing JOIN on SubTable.
        | In the above example, col1 of MainTable collectively fetches \ ``'Foo'``\  records and records of SubTable that match the id of the records matching with search conditions.
        | When there are duplicate column names, it is necessary to assign alias name in order to identify the table to which that column belongs.

 | If JOIN (Join Fetch) is used, \ **all the required data can be fetched by executing SQL once.**\

 .. note:: **When performing JOIN by JPQL**

     For example of performing JOIN using JPQL, refer to \ :ref:`data-access-jpa_howtouse_join_fetch`\ .

 .. warning::

    When relation with SubTable is 1:N, the problem can be resolved using JOIN (Join Fetch); however the following points should be noted.

    * When JOIN is performed on records having 1:N relation, unnecessary data is fetched depending on the number of records in SubTable.
      For details, refer to \ :ref:`Notes during collective fetch <DataAccessMyBatis3AppendixAcquireRelatedObjectsWarningSqlMapping>`\ .

    * When using JPA (Hibernate), if N portions in 1:N are multiple, then it is necessary to use \ ``java.util.Set``\  instead of \ ``java.util.List``\  as a collection type storage N portion.


Resolving N+1 by fetching related records in batch
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| There are cases where, it has proved better when the related records are fetched in batch for patterns with multiple 1:N relations etc.; and then sorted by programming.
| When relation with SubTable is 1:N, analyze whether the problem can be resolved using this method.

 .. figure:: images/dataaccess_common-n_plus_1_solve_programing.png
    :alt: about solve N+1 Problem using programing
    :width: 90%
    :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Search the records matching the search conditions from MainTable.
        | In the above example, col1 of MainTable fetches \ ``'Foo'``\  records and the total records fetched are 20.
    * - | (2)
      - | For each record searched in (1), related records are fetched from SubTable. 
        | Related records are not fetched one by one; but the records matching the foreign key of each record fetched in (1), are fetched in batch.
        | In the above example, id column of SubTable collectively fetches same records as id column of records fetched in (1) using IN clause.
    * - | (3)
      - | SubTable records fetched in (2) sorted and merged with records fetched in (1).

 | In the above example, \ **all the required data can be fetched by executing SQL twice.**\
 | Even supposing there are 3 SubTables, \ **SQL needs to be executed totally 4 times.**\

 .. note::

     This method has a special feature. It can fetch only the required data by optimizing SQL execution.
     It is necessary to sort SubTable records by programming; however when there are many SubTables or when number of N records in 1:N is more, there are cases wherein it is better to resolve the problem using this method.

|

Appendix
--------------------------------------------------------------------------------

.. _data-access-common_appendix_like_escape:

Escaping during LIKE search
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
When performing LIKE search, the values to be used as search conditions need to be escaped.

Following class is provided by common library as a component to perform escaping for LIKE search.

.. tabularcolumns:: |p{0.10\linewidth}|p{0.40\linewidth}|p{0.50\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 40 50

    * - Sr. No.
      - Class
      - Description
    * - 1.
      - | org.terasoluna.gfw.common.query.
        | QueryEscapeUtils
      - Utility class that provide methods to perform escaping of SQL and JPQL.

        In this class,

        * method to perform escaping for LIKE search

        is provided.

    * - 2.
      - | org.terasoluna.gfw.common.query.
        | LikeConditionEscape
      - Class to perform escaping for LIKE search.

.. note::

    \ ``LikeConditionEscape``\  is a class added from terasoluna-gfw-common 1.0.2.RELEASE
    to fix "`Bugs related to handling of wildcard characters for LIKE search <https://github.com/terasolunaorg/terasoluna-gfw/issues/78>`_".

    \ ``LikeConditionEscape``\  class plays a role in absorbing the differences in wildcard characters that occur due to difference in database and database versions.

|

Specifications of escaping of common library
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Specifications of escaping provided by common library are as follows:

* Escape character is ``"~"`` .
* Characters to be escaped by default are 2, namely ``"%"`` , ``"_"`` .

.. note::

    Till terasoluna-gfw-common 1.0.1.RELEASE, the characters to be escaped were 4, namely ``"%"`` , ``"_"`` , ``"％"`` , ``"＿"`` ; however,
    it is changed to 2 characters namely ``"%"`` , ``"_"`` from terasoluna-gfw-common 1.0.2.RELEASE
    in order to fix the "`Bugs related to handling of wildcard characters for LIKE search <https://github.com/terasolunaorg/terasoluna-gfw/issues/78>`_ ".

    In addition, a method for escaping that includes double byte characters ``"％"`` , ``"＿"`` as characters to be escaped, is also provided.

|

See the example of escaping below.

**[Example of escaping with default specifications]**

Example of escaping when default values used as characters to be escaped is given below.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.20\linewidth}|p{0.10\linewidth}|p{0.45\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 15 20 10 45

    * - | Sr. No.
      - | Target
        | String
      - | After escaping
        | String
      - | Escaping
        | Flag
      - | Description
    * - 1.
      - ``"a"``
      - ``"a"``
      - OFF
      - Escaping not done as the string does not contain character to be escaped.
    * - 2.
      - ``"a~"``
      - ``"a~~"``
      - ON
      - Escaping done as the string contains escape character.
    * - 3.
      - ``"a%"``
      - ``"a~%"``
      - ON
      - Escaping done as the string contains character to be escaped.
    * - 4.
      - ``"a_"``
      - ``"a~_"``
      - ON
      - Similar to No.3.
    * - 5.
      - ``"_a%"``
      - ``"~_a~%"``
      - ON
      - Escaping done as the string contains characters to be escaped. When there are multiple characters to be escaped, escaping is done for all characters.
    * - 6.
      - ``"a％"``
      - ``"a％"``
      - OFF
      - Similar to No.1.

        From terasoluna-gfw-common 1.0.2.RELEASE, ``"％"`` is handled as character out of escaping scope in default specifications.
    * - 7.
      - ``"a＿"``
      - ``"a＿"``
      - OFF
      - Similar to No.1.

        From terasoluna-gfw-common 1.0.2.RELEASE, ``"＿"`` is handled as character out of escaping scope in default specifications.
    * - 8.
      - ``" "``
      - ``" "``
      - OFF
      - Similar to No.1.
    * - 9.
      - ``""``
      - ``""``
      - OFF
      - Similar to No.1.
    * - 10.
      - ``null``
      - ``null``
      - OFF
      - Similar to No.1.

|

**[Example of escaping when double byte characters are included]**

Example of escaping when double byte characters included as characters to be escaped is given below.

For other than Sr. No. 6 and 7, refer to escaping example of default specifications.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.20\linewidth}|p{0.10\linewidth}|p{0.45\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 15 20 10 45

    * - | Sr. No.
      - | Target
        | String
      - | After escaping
        | String
      - | Escaping
        | Flag
      - | Description
    * - 6.
      - ``"a％"``
      - ``"a~％"``
      - ON
      - Escaping done as string contains characters to be escaped.
    * - 7.
      - ``"a＿"``
      - ``"a~＿"``
      - ON
      - Similar to No.6.

|

About escaping methods provided by common library
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
List of escaping methods for LIKE search of \ ``QueryEscapeUtils``\  class and  \ ``LikeConditionEscape``\  class provided by common library is given below.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 35 55

    * - Sr. No.
      - Method name
      - Description
    * - 1.
      - toLikeCondition(String)
      - | String passed as an argument is escaped for LIKE search.
        | When specifying type of matching (Forward match, Backward match and Partial match) at SQL or JPQL side, perform only escaping using this method.
    * - 2.
      - toStartingWithCondition(String)
      - | After escaping a string passed as an argument for LIKE search, assign ``"%"`` at the end of the string after escaping.
        | This method is used in order to convert into a value for Forward match search.
    * - 3.
      - toEndingWithCondition(String)
      - | After escaping a string passed as an argument for LIKE search, assign ``"%"`` at the beginning of the string after escaping.
        | This method is used in order to convert into a value for Backward match search.
    * - 4.
      - toContainingCondition(String)
      - | After escaping a string passed as an argument for LIKE search, assign ``"%"`` at the beginning and end of the string after escaping.
        | This method is used in order to convert into a value for Partial match search. 

 .. note::

    Methods of No.2, 3, 4 are used when specifying the type of matching (Forward match, Backward match and Partial match) at program side and not at SQL or JPQL side.

|

How to use common library
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
For example of escaping at the time of LIKE search, refer to the document for O/R Mapper to be used.

* When using MyBatis3, refer to \ :ref:`DataAccessMyBatis3HowToUseLikeEscape`\  of \ :doc:`DataAccessMyBatis3`\ .
* When using JPA (Spring Data JPA), refer to \ :ref:`data-access-jpa_howtouse_like_escape`\ of \ :doc:`DataAccessJpa`\ .

.. note::

    API for escaping should be used as per wildcard characters supported by database to be used.

    **[In case of database that supports only "%" , "_" (single byte characters) as wildcard]**

     .. code-block:: java

        String escapedWord = QueryEscapeUtils.toLikeCondition(word);

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90

        * - | Sr. No.
          - | Description
        * - | (1)
          -  Escaping is done by directly using method of \ ``QueryEscapeUtils``\  class.

    **[In case of database that also supports "％" , "＿" (double byte characters) as wildcard]**

     .. code-block:: java

        String escapedWord = QueryEscapeUtils.withFullWidth()  // (2)
                                .toLikeCondition(word);        // (3)


     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90

        * - | Sr. No.
          - | Description
        * - | (2)
          -  Fetch instance of \ ``LikeConditionEscape``\  class by calling \ ``withFullWidth()``\  method of \ ``QueryEscapeUtils``\  method.
        * - | (3)
          -  Perform escaping by using method of \ ``LikeConditionEscape``\  class instance fetched in (2).

|

.. _data-access-common_appendix_sequencer:

About Sequencer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Sequencer is a common library for fetching sequence value.
| Use the sequence value fetched from Sequencer as a configuration value of primary key column of the database.

 .. note:: **Reason for creating Sequencer as a common library**

    The reason for creating Sequencer is that there is no mechanism to format the sequence value as string in ID generator functionality of JPA.
    In actual application development, sometimes the formatted string is also set as primary key; hence Sequencer is provided as common library.

    When value set as primary key is number, it is recommended to use ID generator functionality of JPA. For ID generator functionality of JPA, refer to \ :ref:`data-access-jpa_how_to_use_way_to_add_entity`\  of \ :doc:`DataAccessJpa`\ .

    The primary objective of creating Sequencer is to supplement functions which are not supported by JPA; but it can also be used when sequence value is required in the processes not relating to JPA.

About classes provided by common library
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| List of classes of Sequencer functionality of common library is as follows:
| For usage example, refer to \ :ref:`data-access-common_howtouse_sequencer`\  of How to use.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 60

    * - Sr. No.
      - Class name
      - Description
    * - 1.
      - | org.terasoluna.gfw.common.sequencer.
        | Sequencer
      - | Interface that defines the method to fetch subsequent sequence value (getNext) and method to return current sequence value (getCurrent).
    * - 2.
      - | org.terasoluna.gfw.common.sequencer.
        | JdbcSequencer
      - | Implementation class of ``Sequencer``  interface for JDBC.
        | This class is used to fetch sequence value by executing SQL in the database.
        | For this class, it is assumed that values are fetched from sequence object of the database; however it is also possible to fetch the values from other than sequence object by calling function stored in the database.

.. _data-access-common_howtouse_sequencer:

How to use common library
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Define a bean for Sequencer.

- :file:`xxx-infra.xml`

 .. code-block:: xml

    <!-- (1) -->
    <bean id="articleIdSequencer" class="org.terasoluna.gfw.common.sequencer.JdbcSequencer">
         <!-- (2) -->
        <property name="dataSource" ref="dataSource" />
         <!-- (3) -->
        <property name="sequenceClass" value="java.lang.String" />
        <!-- (4) -->
        <property name="nextValueQuery"
            value="SELECT TO_CHAR(NEXTVAL('seq_article'),'AFM0000000000')" />
        <!-- (5) -->
        <property name="currentValueQuery"
            value="SELECT TO_CHAR(CURRVAL('seq_article'),'AFM0000000000')" />
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Define a bean for class that implements \ ``org.terasoluna.gfw.common.sequencer.Sequencer``\ .
        | In the above example, (\ ``JdbcSequencer``\ ) class for fetching sequence value by executing SQL is specified.
    * - | (2)
      - | Specify the datasource for executing the SQL to fetch sequence value.
    * - | (3)
      - | Specify the type of sequence value to be fetched.
        | In the above example, since conversion to string is done using SQL; \ ``java.lang.String``\  type is specified.
    * - | (4)
      - | Specify SQL for fetching subsequent sequence value.
        | In the above example, sequence value fetched from sequence object of (PostgreSQL) database is formatted as string.
        | When sequence value fetched from the database is \ ``1``\ , \ ``"A0000000001"``\  is returned as return value of \ ``Sequencer#getNext()``\  method.
    * - | (5)
      - | Specify SQL for fetching current sequence value.
        | When sequence value fetched from the database is \ ``2``\ , \ ``"A0000000002"``\  is returned as return value of \ ``Sequencer#getCurrent()``\  method.


Fetch sequence value from Sequencer for which bean is defined.

- Service

 .. code-block:: java

    // omitted

    // (1)
    @Inject
    @Named("articleIdSequencer") // (2)
    Sequencer<String> articleIdSequencer;

    // omitted

    @Transactional
    public Article createArticle(Article inputArticle) {

        String articleId = articleIdSequencer.getNext(); // (3)
        inputArticle.setArticleId(articleId);

        Article savedArticle = articleRepository.save(inputArticle);

        return savedArticle;
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Inject \ ``Sequencer``\  object for which bean is defined.
        | In the above example, since sequence value is fetched as formatted string, \ ``java.lang.String``\  type is specified as generics type of \ ``Sequencer``\ .
    * - | (2)
      - | Specify bean name of the bean to be injected in value attribute of \ ``@javax.inject.Named``\  annotation.
        | In the above example, bean name (\ ``"articleIdSequencer"``\ ) defined in \ :file:`xxx-infra.xml`\  is specified.
    * - | (3)
      - | Call \ ``Sequencer#getNext()``\  method and fetch the subsequent sequence value.
        | In the above example, fetched sequence value is used as Entity ID.
        | When fetching current sequence value, call \ ``Sequencer#getCurrent()``\  method.

 .. tip::

    When \ ``Sequencer``\  for which bean is defined is 1, \ ``@Named``\  annotation can be omitted. When specifying multiple sequencers, bean name needs to be specified using \ ``@Named``\  annotation.

.. _appendix_dataaccessexception_converter_class-label:

Classes provided by Spring Framework for converting to data access exception
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Classes of Spring Framework which play a role in converting an exception to data access exception, are as follows:

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.60\linewidth}|
 .. list-table:: **Classes of Spring Framework for converting to data access exception**
    :header-rows: 1
    :widths: 10 35 60

    * - Sr. No.
      - Class name
      - Description
    * - 1.
      - | org.springframework.jdbc.support.
        | SQLErrorCodeSQLExceptionTranslator
      - When MyBatis or \ ``JdbcTemplate``\  is used, JDBC exception is converted to data access exception of Spring Framework using this class. Conversion rules are mentioned in XML file. XML file used by default is \ ``org/springframework/jdbc/support/sql-error-codes.xml``\  in \ ``spring-jdbc.jar``\ .
        It is also possible to change the default behavior by placing XML file (\ ``sql-error-codes.xml``\ ) just below class path.
    * - 2.
      - | org.springframework.orm.jpa.vendor.
        | HibernateJpaDialect
      - When JPA (Hibernate implementation) is used, O/R Mapper exception (Hibernate exception) is converted to data access exception of Spring Framework using this class.
    * - 3.
      - | org.springframework.orm.jpa.
        | EntityManagerFactoryUtils
      - If an exception that cannot be converted by \ ``HibernateJpaDialect``\  has occurred, JPA exception is converted to data access exception of Spring Framework using this class.
    * - 4.
      - | Sub classes of
        | org.hibernate.dialect.Dialect
      - When JPA (Hibernate implementation) is used, exceptions are converted to JDBC exception and O/R Mapper exception using this class.

.. _appendix_datasource_of_spring-label:

JDBC datasource classes provided by Spring Framework
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Spring Framework provides implementation of JDBC datasource. However since they are very simple classes, they are rarely used in production environment.
| These classes are mainly used during Unit Testing.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table:: **JDBC datasource classes provided by Spring Framework**
    :header-rows: 1
    :widths: 10 35 55

    * - Sr. No.
      - Class name
      - Description
    * - 1.
      - | org.springframework.jdbc.datasource.
        | DriverManagerDataSource
      - Datasource class for creating new connection by calling \ ``java.sql.DriverManager#getConnection``\  when connection fetch request is received from the application.
        When connection pooling is required, Application Server datasource or datasource of OSS/Third-Party library should be used.
    * - 2.
      - | org.springframework.jdbc.datasource.
        | SingleConnectionDataSource
      - Child class of \ ``DriverManagerDataSource``\ .This class provides implementation of single shared connection. This is a datasource class for unit test which works with single thread.
        Even in case of Unit Testing, if this class is used when datasource is to be accessed with multithread, care needs to be taken as it may not show the expected behavior.
    * - 3.
      - | org.springframework.jdbc.datasource.
        | SimpleDriverDataSource
      - Datasource class for creating new connection by calling \ ``java.sql.Driver#getConnection``\  when connection fetch request is received from the application.
        When connection pooling is required, Application Server datasource or datasource of OSS/Third-Party library should be used.


| Spring Framework provides adapter classes with extended JDBC datasource operations. 
| Specific adapter classes are introduced below.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table:: **JDBC datasource adapter classes provided by Spring Framework**
    :header-rows: 1
    :widths: 10 35 55

    * - Sr. No.
      - Class name
      - Description
    * - 1.
      - | org.springframework.jdbc.datasource.
        | TransactionAwareDataSourceProxy
      - Adapter class for converting a datasource which does not store transactions, into a datasource storing Spring Framework transactions.
    * - 2.
      - | org.springframework.jdbc.datasource.lookup.
        | IsolationLevelDataSourceRoute
      - Adapter class for switching the datasource to be used based on independence level of an active transaction.

.. raw:: latex

   \newpage

