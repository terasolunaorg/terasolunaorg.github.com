Database Access (MyBatis3)
================================================================================

.. only:: html

 .. contents:: Table of Contents
    :local:
    :depth: 3


.. _DataAccessMyBatis3Overview:

Overview
--------------------------------------------------------------------------------

This chapter describes how to access database using \ `MyBatis3 <http://www.mybatis.org/mybatis-3/>`_\ .

This guideline presumes the use of Mapper interface of MyBatis3 as a Repository interface.
Refer to ":ref:`repository-label`" for Repository interface.

| The architecture to access database by using MyBatis3 and MyBatis-Spring is explained in the Overview.
| For more information, refer ":ref:`DataAccessMyBatis3HowToUse`".

 .. figure:: images_DataAccessMyBatis3/DataAccessMyBatis3Scope.png
    :alt: Scope of description
    :width: 100%
    :align: center

    **Picture - Scope of description**

|

.. _DataAccessMyBatis3OverviewAboutMyBatis3:

About MyBatis3
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| MyBatis3 is a type of O/R mapper which is developed for mapping SQL to objects.
 and not for mapping the records stored in database to objects.
| Thus, it is an effective O/R mapper to access denormalized databases or to obtain full control of SQL execution in an application
 without entrusting the SQL statement execution to the O/R mapper.

In this guideline, CRUD operation of Entity is performed by using Mapper interface added from MyBatis3.
Refer to ":ref:`DataAccessMyBatis3AppendixAboutMapperMechanism`" for details of Mapper interface.

This guideline does not cover explanation of all the functionalities of MyBatis3,
Hence, it is recommended to refer "\ `MyBatis 3 REFERENCE DOCUMENTATION <http://mybatis.github.io/mybatis-3/>`_ \" .

|

.. _DataAccessMyBatis3OverviewAboutComponentConstitutionOfMyBatis3:

Component structure of MyBatis3
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| The explanation about main components of MyBatis3 (configuration file) is given below.
| In MyBatis3, SQL execution and O/R mapping is implemented by integrating the following components with each other based on the definition of configuration file.

 .. tabularcolumns:: |p{0.1\linewidth}|p{0.2\linewidth}|p{0.6\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - Sr. No.
      - Component/Configuration file
      - Description
    * - (1)
      - MyBatis configuration file
      - XML file that explains operation settings of MyBatis3.

        It is a file that explains details such as connecting destination for database, path of mapping file, operation settings of MyBatis and so on.
        It is not necessary to specify connecting destination of database and mapping file path settings in the configuration file, when using it by integrating with Spring.
        However, settings are performed when changing or extending default operations of MyBatis3.
    * - (2)
      - ``org.apache.ibatis.session.``
        ``SqlSessionFactoryBuilder``
      - A component to read MyBatis configuration file and generate \ ``SqlSessionFactory`` \ .

        This component is not directly handled by the application class when used by integrating with Spring.
    * - (3)
      - ``org.apache.ibatis.session.``
        ``SqlSessionFactory``
      - A component to generate \ ``SqlSession`` \ .

        This component is not directly handled by the application class when used by integrating with Spring.
    * - (4)
      - ``org.apache.ibatis.session.``
        ``SqlSession``
      - A component to provide API for SQL execution and transaction control.

        It is the component that plays the most important role when accessing database using MyBatis3.
        
        When this component is used by integrating with Spring, it is not directly handled by the application class.
    * - (5)
      - Mapper interface
      - An interface to call the SQL defined in mapping file in typesafe.

        Developer needs to create only the interface, as MyBatis3 automatically generates an implementation class for the Mapper interface.
    * - (6)
      - Mapping file

      - XML file that explains SQL and O/R mapping settings.

|

| Flow by which main components of MyBatis3 access the database, is explained below.
| Process for accessing database can be broadly divided into 2 types.

* Processes that are performed at the start of the application. Processes (1) to (3) mentioned below correspond to this type.
* Processes that are performed for each request from the client. Processes (4) to (10) mentioned below correspond to this type.

 .. figure:: images_DataAccessMyBatis3/DataAccessMyBatis3RelationshipOfComponents.png
    :alt: Relationship of MyBatis3 components
    :width: 100%
    :align: center

    **Picture - Relationship of MyBatis3 components**

| Processes that are performed at the start of the application are executed with following flow.
| Refer to ":ref:`DataAccessMyBatis3OverviewAboutComponentConstitutionOfMyBatisSpring`" for the flow when integrating with Spring.

 .. tabularcolumns:: |p{0.1\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80


    * - Sr. No.
      - Description
    * - (1)
      - Application requests building \ ``SqlSessionFactory`` \  for \ ``SqlSessionFactoryBuilder`` \ .
    * - (2)
      - \ ``SqlSessionFactoryBuilder`` \  reads MyBatis configuration file for generating \ ``SqlSessionFactory`` \ .
    * - (3)
      - \ ``SqlSessionFactoryBuilder`` \  generates \ ``SqlSessionFactory`` \  based on the definition of MyBatis configuration file.

|

| Processes that are performed for each request from the client are executed with the following flow.
| Refer to ":ref:`DataAccessMyBatis3OverviewAboutComponentConstitutionOfMyBatisSpring`" for the flow when integrating with Spring.

 .. tabularcolumns:: |p{0.1\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80


    * - Sr. No.
      - Description
    * - (4)
      - Client requests a process for the application.
    * - (5)
      - Application fetches \ ``SqlSession`` \  from \ ``SqlSessionFactory`` \  that is built by using \ ``SqlSessionFactoryBuilder`` \ .
    * - (6)
      - \ ``SqlSessionFactory`` \  generates \ ``SqlSession`` \  and returns it to the application.
    * - (7)
      - Application fetches the implementation object of Mapper interface from \ ``SqlSession`` \ .
    * - (8)
      - Application calls Mapper interface method.
      
        Refer to ":ref:`DataAccessMyBatis3AppendixAboutMapperMechanism`" for mechanism of Mapper interface.
    * - (9)
      - Implementation object of Mapper interface calls \ ``SqlSession`` \  method and requests SQL execution.
    * - (10)
      - \ ``SqlSession`` \ fetches the SQL to be executed from mapping file and executes SQL.

 .. tip:: **Transaction control**

    Commit and rollback for the transaction are performed by calling \ ``SqlSession`` \  API from application code.
    However, it is not described in the above flow.
    
    When integrated with Spring, Spring transaction control functionality performs commit and rollback.
    As a result, the API controlling \ ``SqlSession`` \  is not called directly from application class.


|

.. _DataAccessMyBatis3OverviewAboutMyBatisSpring:

Regarding MyBatis3 and Spring integration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| \ `MyBatis-Spring <http://mybatis.github.io/spring/>`_ \  is provided by MyBatis as the library to integrate MyBatis3 and Spring.
| MyBatis3 components can be stored on Spring DI container by using this library.

There are following advantages of using MyBatis-Spring.

* MyBatis3 SQL can be executed in the transactions managed by Spring. Hence, it is not necessary to control the transactions that are dependent on MyBatis3 API.

* MyBatis3 exception is converted to a generic exception (\``org.springframework.dao.DataAccessException``\) provided by Spring. Hence, exception process that is not dependent on MyBatis3 API can be implemented.

* The overall initialization process for using MyBatis3 is performed by MyBatis-Spring API. Hence, MyBatis3 API need not be used directly.

* Mapper object can be injected in Singleton Service class to generate a thread safe Mapper object.


This guideline presumes the use of MyBatis-Spring.

This guideline does not cover explanation of all the functionalities of MyBatis-Spring,
Hence, it is recommended to refer "\ `Mybatis-Spring REFERENCE DOCUMENTATION <http://mybatis.github.io/spring/>`_ \" .

|

.. _DataAccessMyBatis3OverviewAboutComponentConstitutionOfMyBatisSpring:

Component structure of MyBatis-Spring
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Main components of MyBatis-Spring are explained here.
| In MyBatis-Spring, MyBatis3 and Spring are integrated by integrating the following components.

 .. tabularcolumns:: |p{0.1\linewidth}|p{0.2\linewidth}|p{0.6\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - Sr. No.
      - Component/Configuration file
      - Description
    * - (1)
      - ``org.mybatis.spring.``
        ``SqlSessionFactoryBean``
      - Component that builds \ ``SqlSessionFactory`` \  and stores objects on Spring DI container.

        In standard MyBatis3, \ ``SqlSessionFactory`` \  is built based on the information defined in MyBatis configuration file. However,
        By using \ ``SqlSessionFactoryBean`` \ , \ ``SqlSessionFactory`` \  can be built even in the absence of MyBatis configuration file.
        It can also be used in combination.
    * - (2)
      - ``org.mybatis.spring.mapper.``
        ``MapperFactoryBean``
      - Component that builds Singleton Mapper object and stores objects on Spring DI container.

        Mapper object generated by MyBatis3 standard mechanism is not thread safe.
        Hence, it was necessary to assign an instance for each thread.
        Mapper object created by MyBatis-Spring component can generate a thread safe Mapper object.
        As a result, DI can be applied to Singleton components like Service etc.
    * - (3)
      - ``org.mybatis.spring.``
        ``SqlSessionTemplate``
      - \ ``SqlSession`` \  component of Singleton version that implements \ ``SqlSession`` \  interface.

        \ ``SqlSession`` \  object generated by MyBatis3 standard mechanism is not thread safe.
        Hence, it was necessary to assign an instance for each thread.
        \ ``SqlSession`` \  object generated by MyBatis-Spring component can generate a thread safe \ ``SqlSession`` \  object.
        As a result, DI can be applied to Singleton components like Service etc.

        However, this guideline does not assume handling \ ``SqlSession`` \  directly.

|

The flow by which the main components of MyBatis-Spring access the database is explained below.
Processes to access database can be broadly divided into two types.

* Processes that are performed at the start of the application. Processes (1) to (4) mentioned below correspond to this type.
* Processes that are performed for each request from the client. Processes (5) to (11) mentioned below correspond to this type.


 .. figure:: images_DataAccessMyBatis3/DataAccessMyBatisSpringRelationshipOfComponents.png
    :alt: Relationship of MyBatis-Spring components
    :width: 100%
    :align: center

    **Picture - Relationship of MyBatis-Spring components**


Processes that are performed at the start of the application are executed by the following flow.

 .. tabularcolumns:: |p{0.1\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80


    * - Sr. No.
      - Description
    * - (1)
      - \ ``SqlSessionFactoryBean`` \  requests building \ ``SqlSessionFactory`` \  for \ ``SqlSessionFactoryBuilder`` \.
    * - (2)
      - \ ``SqlSessionFactoryBuilder`` \  reads MyBatis configuration file for generating \ ``SqlSessionFactory`` \.
    * - (3)
      - \ ``SqlSessionFactoryBuilder`` \  generates \ ``SqlSessionFactory`` \  based on the definition of MyBatis configuration file.

        \ ``SqlSessionFactory`` \  thus generated is stored by the Spring DI container.
    * - (4)
      - \ ``MapperFactoryBean`` \  generates a thread safe \ ``SqlSession`` \  (\ ``SqlSessionTemplate`` \ ) and
        a thread safe Mapper object (Proxy object of Mapper interface).

        Mapper object thus generated is stored by Spring DI container and DI is applied for Service class etc.
        Mapper object provides a thread safe implementation by using thread safe \ ``SqlSession`` \  (\ ``SqlSessionTemplate`` \ ).

|

Processes that are performed for each request from client are executed by the following flow.

 .. tabularcolumns:: |p{0.1\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80


    * - Sr. No.
      - Description
    * - (5)
      - Client requests a process for the application.
    * - (6)
      - Application (Service) calls the method of Mapper object (Proxy object that implements Mapper interface) injected by DI container.

        Refer to ":ref:`DataAccessMyBatis3AppendixAboutMapperMechanism`" for Mapper interface mechanism.
    * - (7)
      - Mapper object calls \ ``SqlSession`` \  (\ ``SqlSessionTemplate`` \ ) method corresponding to the called method.
    * - (8)
      - \ ``SqlSession`` \  (\ ``SqlSessionTemplate`` \ ) calls the proxy enabled and thread safe \ ``SqlSession`` \  method.
    * - (9)
      - Proxy enabled and thread safe \ ``SqlSession`` \  uses MyBatis3 standard \ ``SqlSession`` \  assigned to the transaction.

        When \ ``SqlSession`` \  assigned to the transaction does not exist, \ ``SqlSessionFactory`` \  method is called
        to fetch \ ``SqlSession`` \  of standard MyBatis3.
    * - (10)
      - \ ``SqlSessionFactory`` \  returns MyBatis3 standard \ ``SqlSession`` \ .

        Since the returned MyBatis3 standard \ ``SqlSession`` \  is assigned to the transaction, if it is within the same transaction, same \ ``SqlSession`` \  is used
        without creating a new one.
    * - (11)
      - MyBatis3 standard \ ``SqlSession`` \  fetches SQL to be executed from mapping file and executes the SQL.

 .. tip:: **Transaction control**

    Although it is not explained in the flow, the commit and rollback of transaction is performed by Spring transaction control function.
    
    Refer to ":ref:`service_transaction_management`" for how to control transaction using Spring transaction control function.
    

|


.. _DataAccessMyBatis3HowToUse:

How to use
--------------------------------------------------------------------------------

Actual configuration and implementation methods for accessing database using MyBatis3 are explained below.

The explanation hereafter can be broadly classified as below.


 .. tabularcolumns:: |p{0.1\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60


    * - Sr. No.
      - Classification
      - Description
    * - (1)
      - Overall application settings
      - Settings for using MyBatis3 in an application and
        for changing MyBatis3 operations, are explained below.

        The contents explained here \ **are required when application architecture performs settings at the time of project start-up.**\
        Therefore, application developers need not be aware of these contents separately.
        
        Following sections correspond to this classification.
        
        * :ref:`DataAccessMyBatis3HowToUseSettingsPomXml`
        * :ref:`DataAccessMyBatis3HowToUseSettingsCooperateWithMyBatis3AndSpring`
        * :ref:`DataAccessMyBatis3HowToUseSettingsMyBatis3`
        
        When a project is generated from a blank project <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank#multi-blank-project-with-mybatis3>`_ \  for MyBatis3,
        a major part of the configuration explained above is already configured. Hence, the application architect
        determines the project characteristics and adds or changes the configuration as required.

    * - (2)
      - How to implement data access process
      - How to implement basic data access process using MyBatis3 is explained.
      
        The contents explained here \ **are necessary for the application developers at the time of implementation.**\
        
        Following sections correspond to this classification.
        
        * :ref:`DataAccessMyBatis3HowToDababaseAccess`
        * :ref:`DataAccessMyBatis3HowToUseResultSetMapping`
        * :ref:`DataAccessMyBatis3HowToUseFind`
        * :ref:`DataAccessMyBatis3HowToUseCreate`
        * :ref:`DataAccessMyBatis3HowToUseUpdate`
        * :ref:`DataAccessMyBatis3HowToUseDelete`
        * :ref:`DataAccessMyBatis3HowToUseDynamicSql`
        * :ref:`DataAccessMyBatis3HowToUseLikeEscape`
        * :ref:`DataAccessMyBatis3HowToUseSqlInjectionCountermeasure`

|

.. _DataAccessMyBatis3HowToUseSettingsPomXml:

pom.xml settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| When MyBatis3 is used in the infrastructure layer, dependency relation with terasoluna-gfw-mybatis3-dependencies is added to \ :file:`pom.xml`\.
| In case of multi project configuration, it is added to \ :file:`pom.xml`\  (:file:`projectName-domain/pom.xml`) of domain project.

| When a project is generated from a blank project <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank#multi-blank-project-with-mybatis3>`_ \  for MyBatis3, dependency relation with terasoluna-gfw-mybatis3-dependencies is already configured.

 .. code-block:: xml
    :emphasize-lines: 22-27

    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
            http://maven.apache.org/maven-v4_0_0.xsd">

        <modelVersion>4.0.0</modelVersion>
        <artifactId>projectName-domain</artifactId>
        <packaging>jar</packaging>

        <parent>
            <groupId>com.example</groupId>
            <artifactId>mybatis3-example-app</artifactId>
            <version>1.0.0-SNAPSHOT</version>
            <relativePath>../pom.xml</relativePath>
        </parent>

        <dependencies>
        
            <!-- omitted -->

            <!-- (1) -->
            <dependency>
                <groupId>org.terasoluna.gfw</groupId>
                <artifactId>terasoluna-gfw-mybatis3-dependencies</artifactId>
                <type>pom</type>
            </dependency>

            <!-- omitted -->

        </dependencies>

        <!-- omitted -->

    </project>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Add terasoluna-gfw-mybatis3 to dependencies.
        Dependency relation with MyBatis3 and MyBatis-Spring is defined in terasoluna-gfw-mybatis3.
        
 .. note::
 	
 		In the above setting example, since it is assumed that the dependent library version is managed by the parent project terasoluna-gfw-parent, specifying the version in pom.xml is not necessary.

 .. Warning:: **Configuration while using in Java SE 7 environment**

    terasoluna-gfw-mybatis3-dependencies configures dependency relation considering Java SE 8 as a prerequisite. Java SE 8 dependency library should be excluded as below while using in Java SE 7 environment.
    For java SE 8 dependency library, refer "\ :ref:`frameworkstack_using_oss_version` \" of architectural overview.

   .. code-block:: xml
    :emphasize-lines: 4-9

            <dependency>
                <groupId>org.terasoluna.gfw</groupId>
                <artifactId>terasoluna-gfw-mybatis3-dependencies</artifactId>
                <exclusions>
                    <exclusion>
                        <groupId>org.mybatis</groupId>
                        <artifactId>mybatis-typehandlers-jsr310</artifactId>
                    </exclusion>
                </exclusions>
            </dependency>



|

.. _DataAccessMyBatis3HowToUseSettingsCooperateWithMyBatis3AndSpring:

Settings for integration of MyBatis3 and Spring
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _DataAccessMyBatis3HowToUseSettingsDataSource:

Datasource settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When MyBatis3 and Spring are integrated, the datasource managed by Spring DI container should be used.

When a project is generated from a blank project <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank#multi-blank-project-with-mybatis3>`_ \  for MyBatis3, datasource of Apache Commons DBCP is already configured.
Hence the settings should be changed in accordance with project requirements.

Refer to "\ :ref:`data-access-common_howtouse_datasource` \" for the details on how to configure datasource.

|

.. _DataAccessMyBatis3HowToUseSettingsTransactionManager:

Transaction control settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| When MyBatis3 and Spring are integrated,
 \ ``PlatformTransactionManager`` \  managed by Spring DI container should be used for transaction control.

| When using local transaction, \ ``DataSourceTransactionManager`` \  that performs transaction control by calling JDBC API, is used.

| When a project is generated from a blank project <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank#multi-blank-project-with-mybatis3>`_ \  for MyBatis3, \ ``DataSourceTransactionManager`` \  is already configured.

Configuration example is as given below.

- :file:`projectName-env/src/main/resources/META-INF/spring/projectName-env.xml`

 .. code-block:: xml
    :emphasize-lines: 15-22

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:jee="http://www.springframework.org/schema/jee"
        xmlns:jdbc="http://www.springframework.org/schema/jdbc"
        xsi:schemaLocation="http://www.springframework.org/schema/jdbc
            http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
            http://www.springframework.org/schema/jee
            http://www.springframework.org/schema/jee/spring-jee.xsd
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd">

        <!-- omitted -->

        <!-- (1) -->
        <bean id="transactionManager"
            class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <!-- (2) -->
            <property name="dataSource" ref="dataSource" />
            <!-- (3) -->
            <property name="rollbackOnCommitFailure" value="true" />
        </bean>

        <!-- omitted -->

    </beans>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Specify \ ``org.springframework.jdbc.datasource.DataSourceTransactionManager`` \  as \ ``PlatformTransactionManager`` \.
    * - (2)
      - Specify configured datasource bean in \ ``dataSource`` \  property.

        When SQL is executed in the transaction, connection is fetched from datasource specified here.
    * - (3)
      - \ Rollback process is called when an error occurs during commit.

        By adding this setting, risk of "Unintentional commit which occurs when a connection with undefined operation returns to connection pool (commit while reusing a connection, implicit commit at the time of closing a connection etc)" can be reduced.
        However, since an error is likely to occur at the time of rollback, it should be noted that a risk of occurrence of unintentional commit is still a possibility.

 .. note:: **bean ID of PlatformTransactionManager**
 
    It is recommended to specify \ ``transactionManager`` \  in id attribute.
    
    If a value other than \ ``transactionManager`` \  is specified,
    the same value must be specified in transaction-manager attribute of \ ``<tx:annotation-driven>`` \  tag.
    

|

When a transaction manager provided by application server is used, use \ ``org.springframework.transaction.jta.JtaTransactionManager`` \ 
that performs transaction control by calling JTA API.

Configuration example is as given below.

- :file:`projectName-env/src/main/resources/META-INF/spring/projectName-env.xml`

 .. code-block:: xml
    :emphasize-lines: 6,13-14,18-19

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:jee="http://www.springframework.org/schema/jee"
        xmlns:jdbc="http://www.springframework.org/schema/jdbc"
        xmlns:tx="http://www.springframework.org/schema/tx"
        xsi:schemaLocation="http://www.springframework.org/schema/jdbc
            http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
            http://www.springframework.org/schema/jee
            http://www.springframework.org/schema/jee/spring-jee.xsd
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/tx
            http://www.springframework.org/schema/tx/spring-tx.xsd">

        <!-- omitted -->

        <!-- (1) -->
        <tx:jta-transaction-manager />

        <!-- omitted -->

    </beans>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - If \ ``<tx:jta-transaction-manager />`` \  is specified, 
        bean definition of optimum \ ``JtaTransactionManager`` \  is performed for the application server.

|

.. _DataAccessMyBatis3HowToUseSettingsMyBatis-Spring:

MyBatis-Spring settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When MyBatis3 and Spring are integrated, it is necessary to carry out following

* Generation of \ ``SqlSessionFactory``\  that customizes the processes necessary for integrating MyBatis3 and Spring
* Generation of thread safe Mapper object (Proxy object of Mapped interface)

by using MyBatis-Spring components.

When a project is generated from a blank project <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank#multi-blank-project-with-mybatis3>`_ \  of MyBatis3, the settings for integration of MyBatis3 and Spring
are already configured.

Configuration example is as given below.

- :file:`projectName-domain/src/main/resources/META-INF/spring/projectName-infra.xml`

 .. code-block:: xml
    :emphasize-lines: 4,7-8,12-20,22-23

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:mybatis="http://mybatis.org/schema/mybatis-spring"
        xsi:schemaLocation="http://www.springframework.org/schema/beans 
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://mybatis.org/schema/mybatis-spring
            http://mybatis.org/schema/mybatis-spring.xsd">

        <import resource="classpath:/META-INF/spring/projectName-env.xml" />

        <!-- (1) -->
        <bean id="sqlSessionFactory"
            class="org.mybatis.spring.SqlSessionFactoryBean">
            <!-- (2) -->
            <property name="dataSource" ref="dataSource" />
            <!-- (3) -->
            <property name="configLocation"
                value="classpath:/META-INF/mybatis/mybatis-config.xml" />
        </bean>

        <!-- (4) -->
        <mybatis:scan base-package="com.example.domain.repository" />

    </beans>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80

   * - Sr. No.
     - Description
   * - (1)
     - Define the bean for \ ``SqlSessionFactoryBean`` \  as the component for generating \ ``SqlSessionFactory`` \.
   * - (2)
     - Specify a bean of configured datasource in \ ``dataSource`` \  property.

       When SQL is executed in MyBatis3 process, the connection is fetched from the datasource specified here.
   * - (3)
     - Specify MyBatis configuration file path in \ ``configLocation`` \  property.

       The file specified is read when generating \ ``SqlSessionFactory`` \.
   * - (4)
     - Define \ ``<mybatis:scan>`` \  for scanning Mapper interface and specify the base package that stores the Mapper interface
       in \ ``base-package`` \  attribute.

       The Mapper interface stored under specified package is scanned and
       a thread safe Mapper object (Proxy object of Mapper interface) is automatically generated.

       **[Package decided for each project should be the specified package]**

 .. note:: **How to configure MyBatis3**

    When \ ``SqlSessionFactoryBean`` \  is used, MyBatis3 configuration can be specified directly
    in the bean property rather than MyBatis configuration file.
    However, in this guideline, it is recommended to specify MyBatis3 settings in the MyBatis standard configuration file.

|

.. _DataAccessMyBatis3HowToUseSettingsMyBatis3:

MyBatis3 settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| A mechanism to customize MyBatis3 operations is provided in MyBatis3.
| MyBatis3 operations can be customized by adding a configuration value in MyBatis configuration file.

| Only those configuration fields that are not dependent on application characteristics are explained here.
| For other configuration fields,
 refer to "\ `MyBatis 3 REFERENCE DOCUMENTATION(Configuration XML) <http://mybatis.github.io/mybatis-3/configuration.html>`_ \"
 and configure in accordance with application characteristics.
| Default configuration can be maintained, however it should be changed when required in accordance with application characteristics.

 .. note:: **Storage location for MyBatis configuration file**
 
    In this guideline, it is recommended to store MyBatis configuration file
    in \ :file:`projectName-domain/src/main/resources/META-INF/mybatis/mybatis-config.xml`\.

    When a project is generated from a blank project <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank#multi-blank-project-with-mybatis3>`_ \  of MyBatis3, the file described above is already stored.

|

.. _DataAccessMyBatis3HowToUseSettingsDefaultFetchSize:

\ ``fetchSize``\  settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| When a query that returns a large amount of data is to be described, an appropriate \ ``fetchSize``\  must be specified for JDBC driver.
| \ ``fetchSize``\  is a parameter that specifies data record count that can be fetched in a single communication between JDBC driver and database.

Since default value of JDBC driver is used if \ ``fetchSize``\  is not specified,
following issues are likely to appear due to JDBC driver that is being used.

* "Performance degradation" when default value of JDBC driver is small
* "Out of memory" when default value of JDBC driver is large or has no restriction

MyBatis3 can specify \ ``fetchSize``\  by using 2 methods shown below to control the occurrences of these issues.

* Specifying "default \ ``fetchSize``\ " applicable for all queries
* Specifying "\ ``fetchSize``\ for query unit" applicable to a specific query

 .. note:: **"Regarding default fetchSize"**

    "Default \ ``fetchSize``\ "can be used in MyBatis3.3.0 and subsequent versions supported in terasoluna-gfw-mybatis3 5.2.x.RELEASE.


How to specify "default \ ``fetchSize``\ " is shown below.


- ``projectName-domain/src/main/resources/META-INF/mybatis/mybatis-config.xml``

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration PUBLIC "-//mybatis.org/DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>

        <settings>
            <!-- (1) -->
            <setting name="defaultFetchSize" value="100" />
        </settings>

    </configuration>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Specify data record count fetched in a single communication, in \ ``defaultFetchSize``\ .

\

 .. note:: **How to specify "fetchsize of query unit"**

    When \ ``fetchSize``\  is to be specified in query unit, a value must be specified in \ ``fetchSize``\  attribute of
    XML element (\ ``<select>``\  element) to describe SQL for search.

Note that, when a query for returning large volume of data is to be described,
usage of ":ref:`DataAccessMyBatis3HowToExtendResultHandler`" must also be explored.

|

.. _DataAccessMyBatis3HowToUseSettingsExecutorType:

SQL execution mode settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

MyBatis3 provides following three modes to execute SQL.

| The mode to be used should be determined based on characteristics and constraints of each mode, and performance requirements.
| Refer to ":ref:`DataAccessMyBatis3HowToExtendExecutorType`" for how to configure an execution mode.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.65\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 15 65

    * - Sr. No.
      - Mode
      - Description
    * - (1)
      - SIMPLE
      - Creates a new \``java.sql.PreparedStatement``\  for each SQL execution.

        It is a default behavior for MyBatis wherein a blank project <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank#multi-blank-project-with-mybatis3>`_ \  also operates in a \ ``SIMPLE``\  mode.
    * - (2)
      - REUSE
      - Caches and reuses ``PreparedStatement``\.

        If \ ``REUSE``\  mode is used when same SQL is to be executed for multiple times in the same transaction,
        enhanced performance can be expected as compared to \ ``SIMPLE``\  mode.

        This is because it analyses SQL and reduces number of executions for the process that generates \ ``PreparedStatement``\.
    * - (3)
      - BATCH
      - Performs batch execution for Update SQL. (Executes SQL by using \ ``java.sql.Statement#executeBatch()``\ )

        If \ ``BATCH``\  mode is used to execute a large number of Update SQLs in succession, in the same transaction,
        improved performance can be expected as compared to \ ``SIMPLE``\  mode or \ ``REUSE``\  mode.

        This is because it reduces

        * Number of executions for the process that generates \ ``PreparedStatement``\  by analyzing the SQL
        * Number of communications with the server

        

        However, when \ ``BATCH``\  mode is used, MyBatis behavior operates in \ ``SIMPLE``\  mode or in a mode different from ``SIMPLE``\  mode.
        Refer to ":ref:`DataAccessMyBatis3HowToExtendExecutorTypeBatchNotes`" for basic differences and precautions.

|

.. _DataAccessMyBatis3HowToUseSettingsTypeAlias:

TypeAlias settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When TypeAlias is used, an alias (short name) can be assigned for the Java class specified in mapping file.

When TypeAlias is not used, it is necessary to specify the fully qualified class name (FQCN) of Java class in \ ``type`` \  attribute, \ ``parameterType`` \  attribute and \ ``resultType`` \  attribute specified in a mapping file 
As a result, decrease in description efficiency and increase in typographical errors of mapping file are the areas of concern.

In this guideline, it is recommended to use TypeAlias to improve description efficiency, reduce typographical errors and improve readability.

When project is generated from a blank project <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank#multi-blank-project-with-mybatis3>`_ \  for MyBatis3,
the class stored under the package (\ ``${projectPackage}.domain.model``\ ) that stores Entity is considered as a target for TypeAlias.
Settings should be added as and when required.

How to configure a TypeAlias is given below.

- :file:`projectName-domain/src/main/resources/META-INF/mybatis/mybatis-config.xml`

 .. code-block:: xml
    :emphasize-lines: 7-8

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
      PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
        <typeAliases>
            <!-- (1) -->
            <package name="com.example.domain.model" />
        </typeAliases>
    </configuration>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80

   * - Sr. No.
     - Description
   * - (1)
     - In \ ``name`` \  attribute of \ ``package`` \  element, specify the package name in which the class that sets alias is stored.
     
       The part from which package is removed acts as an alias for the class that is stored under the specified package.
       In the above example, the alias for ``com.example.domain.model.Account`` \  class is \ ``Account`` \.

       **[Package decided for each project should be the specified package]**


 .. tip:: ** How to configure Type Alias in class unit**
 
    A method to configure TypeAlias in class unit and a method to clearly specify an alias are provided in Type Alias settings.
    Refer to ":ref:`DataAccessMyBatis3AppendixSettingsTypeAlias`" of Appendix for details.

|

The description example of mapping file when using TypeAlias is as below.

 .. code-block:: xml
    :emphasize-lines: 8,13,19

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

    <mapper namespace="com.example.domain.repository.account.AccountRepository">

        <resultMap id="accountResultMap"
            type="Account">
            <!-- omitted -->
        </resultMap>

        <select id="findOne"
            parameterType="string"
            resultMap="accountResultMap">
            <!-- omitted -->
        </select>

        <select id="findByCriteria"
            parameterType="AccountSearchCriteria"
            resultMap="accountResultMap">
            <!-- omitted -->
        </select>

    </mapper>

 .. tip:: **MyBatis3 standard Alias name**
 
    An alias name is already configured for general Java classes like primitive type and primitive wrapper type.

    Refer to "\ `MyBatis 3 REFERENCE DOCUMENTATION(Configuration XML-typeAliases-) <http://mybatis.github.io/mybatis-3/configuration.html#typeAliases>`_ \" for the alias names which are already configured.
    

|

.. _DataAccessMyBatis3HowToUseSettingsMappingNullAndJdbcType:

Mapping settings of NULL value and JDBC type
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| An error may occur when setting the column value to null for a database that is being used (JDBC driver).
| This issue can be resolved by the JDBC driver by configuring \ ``null``\  value and specifying a recognizable JDBC type.

| When setting \ ``null``\  value, if an error accompanied by following stack traces occurs, mapping of \ ``null``\  value and JDBC type becomes necessary.
| By default, a generic JDBC type called \ ``OTHER``\  is specified in MyBatis3. However, an error may occur in JDBC driver due to \ ``OTHER``\.

 .. code-block:: guess
    :emphasize-lines: 1

    java.sql.SQLException: Invalid column type: 1111
        at oracle.jdbc.driver.OracleStatement.getInternalType(OracleStatement.java:3916) ~[ojdbc6-11.2.0.2.0.jar:11.2.0.2.0]
        at oracle.jdbc.driver.OraclePreparedStatement.setNullCritical(OraclePreparedStatement.java:4541) ~[ojdbc6-11.2.0.2.0.jar:11.2.0.2.0]
        at oracle.jdbc.driver.OraclePreparedStatement.setNull(OraclePreparedStatement.java:4523) ~[ojdbc6-11.2.0.2.0.jar:11.2.0.2.0]
        ...

 .. note:: **Operations when using Oracle**
 
    When Oracle is used as a database and if default settings is used as is, it has been confirmed that errors occur.
    Although behavior may change depending on the version, it should be described as 'change in the settings may be required when Oracle is used'.

    The version wherein error is confirmed to occur in Oracle 11g R1. The error can be resolved by changing the settings 
    wherein \ ``NULL`` \ type of JDBC type is mapped.

|

How to change the default behavior of MyBatis3 is given below.

- :file:`projectName-domain/src/main/resources/META-INF/mybatis/mybatis-config.xml`

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration PUBLIC "-//mybatis.org/DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>

        <settings>
            <!-- (1) -->
            <setting name="jdbcTypeForNull" value="NULL" />
        </settings>

    </configuration>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80

   * - Sr. No.
     - Description
   * - (1)
     - Specify JDBC type in jdbcTypeForNull.

       In the above example, \ ``NULL``\ type is specified as JDBC type of \ ``null``\  value.



 .. tip:: **How to resolve at item level**

    As an alternate method of resolving the error, an appropriate JDBC type supporting the Java type is set 
    in the inline parameters of property wherein  \ ``null``\  value may be set.

    However, when JDBC type is individually set in inline parameter, description content of mapping file and occurrence of specified mistakes may increase.
    Hence, in this guideline, it is recommended to resolve the errors in the overall configuration.
    If errors are not resolved even after changing the overall configuration, individual setting can be applied only for the property wherein an error has occurred.


|


.. _DataAccessMyBatis3HowToUseSettingsTypeHandler:

TypeHandler settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``TypeHandler`` \  is used when mapping Java class and JDBC type.

Basically, it is used when

* A Java class object is set as a bind parameter of \ ``java.sql.PreparedStatement`` \  while executing an SQL.
* A value is fetched from \ ``java.sql.ResultSet`` \  that is obtained as SQL execution result.



A \ ``TypeHandler`` \  is provided by MyBatis3 for general Java classes like primitive type and primitive wrapper type class.
Specific settings are not required.

**Configuration while using JSR-310 Date and Time API**

When a class which represents date and time offered by JSR-310 Date and Time API in MyBatis3 is used, \ ``TypeHandler`` \  offered by a library different from  that of MyBatis (\ ``mybatis-typehandlers-jsr310`` \) is used.
While using, configuration to recognise \ ``TypeHandler`` \  is added to \ ``mybatis-config.xml`` \, in MyBatis.
    

 .. code-block:: xml
 
      <typeHandlers>
          <typeHandler handler="org.apache.ibatis.type.InstantTypeHandler" />         <!-- (1) -->
          <typeHandler handler="org.apache.ibatis.type.LocalDateTimeTypeHandler" />   <!-- (2) -->
          <typeHandler handler="org.apache.ibatis.type.LocalDateTypeHandler" />       <!-- (3) -->
          <typeHandler handler="org.apache.ibatis.type.LocalTimeTypeHandler" />       <!-- (4) -->
          <typeHandler handler="org.apache.ibatis.type.OffsetDateTimeTypeHandler" />  <!-- (5) -->
          <typeHandler handler="org.apache.ibatis.type.OffsetTimeTypeHandler" />      <!-- (6) -->
          <typeHandler handler="org.apache.ibatis.type.ZonedDateTimeTypeHandler" />   <!-- (7) -->
          <typeHandler handler="org.apache.ibatis.type.YearTypeHandler" />            <!-- (8) -->
          <typeHandler handler="org.apache.ibatis.type.MonthTypeHandler" />           <!-- (9) -->
      </typeHandlers>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80

   * - Sr. No.
     - Description
   * - (1)
     - A \ ``TypeHandler`` \  to map \ ``java.time.Instant`` \  in \ ``java.sql.Timestamp`` \.
   * - (2)
     - A \ ``TypeHandler`` \  to map \ ``java.time.LocalDateTime`` \  in \ ``java.sql.Timestamp`` \.
   * - (3)
     - A \ ``TypeHandler`` \  to map \ ``java.time.LocalDate`` \  in \ ``java.sql.Date`` \
   * - (4)
     - A \ ``TypeHandler`` \  to map \ ``java.time.LocalTime`` \  in \ ``java.sql.Time`` \
   * - (5)
     - A \ ``TypeHandler`` \  to map \ ``java.time.OffsetDateTime`` \  in \ ``java.sql.Timestamp`` \
   * - (6)
     - A \ ``TypeHandler`` \  to map \ ``java.time.OffsetTime`` \  in \ ``java.sql.Time`` \
   * - (7)
     - A \ ``TypeHandler`` \  to map \ ``java.time.ZonedDateTime`` \  in \ ``java.sql.Timestamp`` \
   * - (8)
     - A \ ``TypeHandler`` \  to map \ ``java.time.Year`` \  in primitive type int
   * - (9)
     - A \ ``TypeHandler`` \  to map \ ``java.time.Month`` \  in primitive type int

 .. tip::

        Since \ ``TypeHandler`` \  is auto-detected in MyBatis 3.4, above configuration is not required.

.. tip::

    Refer to "\ `MyBatis 3 REFERENCE DOCUMENTATION(Configuration XML-typeHandlers-) <http://mybatis.github.io/mybatis-3/configuration.html#typeHandlers>`_ \"  for a \ ``TypeHandler`` \  provided by MyBatis3.
    

.. tip:: **Enum type mapping**

    Enum type is mapped with a constant identifier (string) of Enum in the default behavior of MyBatis3.

    In case of Enum type shown below,
    it is mapped with strings like \ ``"WAITING_FOR_ACTIVE"`` \ , \ ``"ACTIVE"`` \ , \ ``"EXPIRED"`` \ , \ ``"LOCKED"`` \
    and stored in the table.

     .. code-block:: java

        package com.example.domain.model;

        public enum AccountStatus {
            WAITING_FOR_ACTIVE, ACTIVE, EXPIRED, LOCKED
        }

    In MyBatis, Enum type can be mapped with the numeric value (order in which the constants are defined). For how to map Enum type with a numeric value, 
    refer to "\ `MyBatis 3 REFERENCE DOCUMENTATION(Configuration XML-Handling Enums-) <http://mybatis.github.io/mybatis-3/configuration.html#Handling_Enums>`_ \".


|

Creating a \ ``TypeHandler`` \  is required while mapping a Java class and JDBC type not supported by MyBatis3.

Basically, it is necessary to create a \ ``TypeHandler`` \  in the following cases

* A file data with large capacity (binary data) is retained in \ ``java.io.InputStream`` \  type and mapped in \ ``BLOB`` \  type of JDBC type.
* A large capacity text data is retained as \ ``java.io.Reader`` \  type and mapped in \ ``CLOB`` \  type of JDBC type.
* \ ``org.joda.time.DateTime`` \  type of ":doc:`../GeneralFuncDetail/JodaTime`" that is recommended to be used in this guideline is mapped with \ ``TIMESTAMP`` \  type of JDBC type.
* etc ...



Refer to ":ref:`DataAccessMyBatis3HowToExtendTypeHandler`" for creating the three types of \ ``TypeHandler`` \  described above.


|

How to apply a \ ``TypeHandler`` \  thus created in MyBatis is explained below.

- :file:`projectName-domain/src/main/resources/META-INF/mybatis/mybatis-config.xml`

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration PUBLIC "-//mybatis.org/DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>

        <typeHandlers>
            <!-- (1) -->
            <package name="com.example.infra.mybatis.typehandler" />
        </typeHandlers>

    </configuration>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80

   * - Sr. No.
     - Description
   * - (1)
     - Configure \ ``TypeHandler`` \  in MyBatis configuration file.

       Specify a package name wherein the created \ ``TypeHandler`` \  is stored, in the name attribute of \ ``package``\  element.
       The \ ``TypeHandler`` \  stored under specified package is automatically detected by MyBatis.

 .. tip::

    In the above example, although \ ``TypeHandler`` \  stored under specified package is automatically detected by MyBatis,
    it can also be configured in class unit.

    \ ``typeHandler``\  element is used when setting \ ``TypeHandler`` \  in class unit.

    - :file:`projectName-domain/src/main/resources/META-INF/mybatis/mybatis-config.xml`

     .. code-block:: xml
        :emphasize-lines: 2

        <typeHandlers>
            <typeHandler handler="xxx.yyy.zzz.CustomTypeHandler" />
            <package name="com.example.infra.mybatis.typehandler" />
        </typeHandlers>

    |

    Further, when a bean stored by DI container is to be used in \ ``TypeHandler`` \,
    \ ``TypeHandler`` \  can be specified in bean definition file.

    - :file:`projectName-domain/src/main/resources/META-INF/spring/projectName-infra.xml`

     .. code-block:: xml
        :emphasize-lines: 16-20

        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
               xmlns:tx="http://www.springframework.org/schema/tx" xmlns:mybatis="http://mybatis.org/schema/mybatis-spring"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/tx
            http://www.springframework.org/schema/tx/spring-tx.xsd
            http://mybatis.org/schema/mybatis-spring
            http://mybatis.org/schema/mybatis-spring.xsd">

            <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
                <property name="dataSource" ref="oracleDataSource" />
                <property name="configLocation"
                    value="classpath:/META-INF/mybatis/mybatis-config.xml" />
                <property name="typeHandlers">
                    <list>
                        <bean class="xxx.yyy.zzz.CustomTypeHandler" />
                    </list>
                </property>
            </bean>

        </beans>

    |

    The mapping of Java class wherein \ ``TypeHandler`` \  is applied and JDBC type is specified as below.

    * Specify as an attribute value of \ ``typeHandler``\  element in MyBatis configuration file
    * Specify in ``@org.apache.ibatis.type.MappedTypes``\  annotation and \ ``@org.apache.ibatis.type.MappedJdbcTypes``\  annotation
    * Specify by inheriting a base class (\ ``org.apache.ibatis.type.BaseTypeHandler``\) of \ ``TypeHandler`` \  provided by MyBatis3

    

    For details, refer to "\ `MyBatis 3 REFERENCE DOCUMENTATION(Configuration XML-typeHandlers-) <http://mybatis.github.io/mybatis-3/configuration.html#typeHandlers>`_ \".


 .. tip::

    Although each of the above example is a configuration method to be applied to overall application,
    an individual \ ``TypeHandler`` \  can also be specified for each field.
    It is used while overwriting a \ ``TypeHandler`` \  that is applicable for overall application.

     .. code-block:: xml
        :emphasize-lines: 6-7,31-32

        <?xml version="1.0" encoding="UTF-8" ?>
        <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
        <mapper namespace="com.example.domain.repository.image.ImageRepository">
            <resultMap id="resultMapImage" type="Image">
                <id property="id" column="id" />
                <!-- (2) -->
                <result property="imageData" column="image_data" typeHandler="XxxBlobInputStreamTypeHandler" />
                <result property="createdAt" column="created_at"  />
            </resultMap>
            <select id="findOne" parameterType="string" resultMap="resultMapImage">
                SELECT
                    id
                    ,image_data
                    ,created_at
                FROM
                    t_image
                WHERE
                    id = #{id}
            </select>
            <insert id="create" parameterType="Image">
                INSERT INTO
                    t_image
                (
                    id
                    ,image_data
                    ,created_at
                )
                VALUES
                (
                    #{id}
                    /* (3) */
                    ,#{imageData,typeHandler=XxxBlobInputStreamTypeHandler}
                    ,#{createdAt}
                )
            </insert>
        </mapper>

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 80

        * - Sr. No.
          - Description
        * - (2)
          - Specify a \ ``TypeHandler`` \  that is applicable to \ ``typeHandler``\  attribute of \ ``id``\  or \ ``result``\  element
            while fetching the value from search result (\ ``ResultSet``\).
        * - (3)
          - Specify a \ ``TypeHandler`` \  that is applicable to \ ``typeHandler``\  attribute of inline parameters
            while configuring a value in the \ ``PreparedStatement``\.

    It is recommended to set TypeAlias in \ ``TypeHandler`` \  class when TypeHandler is to be individually specified for each field.
    Refer to ":ref:`DataAccessMyBatis3HowToUseSettingsTypeAlias`" for how to configure TypeAlias.



|

.. _DataAccessMyBatis3HowToDababaseAccess:

Implementation of database access process
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A basic implementation method for accessing a database by using MyBatis3 function is explained below.

.. _DataAccessMyBatis3HowToDababaseAccessCreateRepository:

Creating Repository interface
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
A Repository interface is created for each Entity.

 .. code-block:: java

    package com.example.domain.repository.todo;

    // (1)
    public interface TodoRepository {
    }
    
 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Create a Repository interface as an interface for Java.
      
        In the above example, a Repository interface is created for an Entity called \ ``Todo``\.

|

.. _DataAccessMyBatis3HowToDababaseAccessCreateMappingFile:

Creating Mapping file
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
A mapping file is created for Repository interface.

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <!-- (1)  -->
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">
    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Specify a fully qualified class name (FQCN) of Repository interface in \ ``namespace``\  attribute of \ ``mapper``\  element.

 .. note:: **Destination to store a mapping file**
 
    The mapping file can be stored at either of the locations given below.
    
    * A directory that conforms to the determined rules to enable MyBatis3 to automatically read the mapping file
    * An arbitrary directory
    
    
    
    \ **In this guideline, it is recommended to use a mechanism wherein  mapping file is stored in the directory conforming to the rules determined by MyBatis3 thus enabling automatic reading of file.**\
    
    It is necessary to store the mapping file on the class path at a level same as the package hierarchy of Repository interface
    to enable automatic reading of mapping file.
    
    In particular,
    a mapping file (\ :file:`TodoRepository.xml`\ ) for Repository interface called \ ``com.example.domain.repository.todo.TodoRepository``\
    should be stored in \ ``projectName-domain/src/main/resources/com/example/domain/repository/todo``\  directory.

|

.. _DataAccessMyBatis3HowToDababaseAccessCrud:

CRUD process implementation
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
How to implement a CRUD process and considerations when implementing SQL are explained here.

How to implement following processes for the basic CRUD operation is explained.

* :ref:`DataAccessMyBatis3HowToUseResultSetMapping`
* :ref:`DataAccessMyBatis3HowToUseFind`
* :ref:`DataAccessMyBatis3HowToUseCreate`
* :ref:`DataAccessMyBatis3HowToUseUpdate`
* :ref:`DataAccessMyBatis3HowToUseDelete`
* :ref:`DataAccessMyBatis3HowToUseDynamicSql`

 .. note::

    It is important to note that the searched Entity is 
    cached in the area called as local cache while implementing CRUD process by using MyBatis3.

    Default behavior of local cache provided by MyBatis3 is as given below.

    * Local cache is managed in a transaction unit.
    * Entity is cached for each "statement ID + pattern of built SQL + parameter value bound to the built SQL + page position (fetch range)".

    In other words, when all the search APIs provided by MyBatis3 are called by the same parameter in a process within the same transaction,
    the instance of cached Entity is returned without executing the SQL from 2nd time onwards.
    

    Here, it should be noted that **Entity returned by MyBatis API and Entity managed by local cache consist of the same instance**.

 .. tip::

    Local cache can also be changed so as to be managed in statement unit.
    When the local cache is to be managed in statement unit, MyBatis executes SQL each time and fetches the latest Entity.

|

The considerations while implementing SQL are explained below.

* :ref:`DataAccessMyBatis3HowToUseLikeEscape`
* :ref:`DataAccessMyBatis3HowToUseSqlInjectionCountermeasure`

|

Before explaining the basic implementation, the components to be registered are explained below.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.25\linewidth}|p{0.55\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 25 55

    * - Sr. No.
      - Component
      - Description
    * - (1)
      - Entity
      - A JavaBean class that retains business process data handled by the application.
      
        Refer to ":ref:`domainlayer_entity`" for details of Entity.
    * - (2)
      - Repository interface
      - An interface that defines the method to perform CRUD operation of Entity.
      
        Refer to ":ref:`repository-label`" for details of Repository.
    * - (3)
      - Service class
      - A class for executing business process logic.
      
        Refer to ":ref:`service-label`" for details of Service.

 .. note::

    In this guideline, Mapper interface of MyBatis3 is called as Repository interface in order to standardize the architecture terminology
    

The explanation hereafter is given presuming that the user has read ":ref:`domainlayer_entity`" ":ref:`repository-label`" and ":ref:`service-label`".

|

.. _DataAccessMyBatis3HowToUseResultSetMapping:

How to map a JavaBean in Search results
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
How to map a JavaBean in the search results is explained before explaining the search process of Entity.

Two methods of automatic and manual, are provided in MyBatis3 
to map JavaBean (Entity) in the search results (\ ``ResultSet``\).
Since both the methods have distinct features, \ **a mapping method to be used should be determined by considering the project features and features of SQL to be executed by the application.**\

 .. note:: **Mapping method to be used**

    The guideline provides two proposals such as

    * Automatic mapping is used for simple mapping (mapping to a single object) whereas manual mapping is used in case of advanced mapping (mapping to related objects) is necessary.
    * A uniform manual mapping is used

    It is not mandatory to use any one of the two methods proposed above and they can be considered as one of the alternatives.

    \ **Architect should clearly identify the criteria for selecting manual mapping and automatic mapping for the programmers
    and look for a uniform mapping method for the entire application.**\

The respective features and examples for automatic mapping and manual mapping are explained below.

|

.. _DataAccessMyBatis3HowToUseResultMappingByAuto:

Automatic mapping for search results
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

In MyBatis3, a mechanism which automatically performs the mapping by matching column name and property name is provided
as a method to map search result (\ ``ResultSet``\) column and JavaBean property.

 .. note:: **Features of automatic mapping**

    When automatic mapping is used, only SQL to be executed is described in the mapping file
    thus reducing the description content of mapping file.
    
    By reducing the description, simple mistakes and modification locations while changing a column name or a property name can be reduced as well.
    
    However, automatic mapping can only be used for single object.
    Manual mapping is required when mapping for the nested related objects.

 .. tip:: **Column names**
 
     Column name mentioned here does not signify the physical column name of a table
     but refers to the column which contains the search result (\ ``ResultSet``\ ) fetched by executing the SQL.
     Therefore, by using AS clause, matching a physical column name and a JavaBean property name is comparatively easier.
     

|

How to map search results in JavaBean using automatic mapping is shown below.

- :file:`projectName-domain/src/main/resources/com/example/domain/repository/todo/TodoRepository.xml`

 .. code-block:: xml
    :emphasize-lines: 8, 10

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">
    
        <select id="findOne" parameterType="string" resultType="Todo">
            SELECT
                todo_id AS "todoId", /* (1) */
                todo_title AS "todoTitle",
                finished, /* (2) */
                created_at AS "createdAt",
                version
            FROM
                t_todo
            WHERE
                todo_id = #{todoId}
        </select>
    
    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80

   * - Sr. No.
     - Description
   * - (1)
     - When physical column name of table and JavaBean property name are different, automatic mapping can be applied by using AS clause for matching.
   * - (2)
     - When physical column name of table and JavaBean property name match, there is no need to specify the AS clause.

- JavaBean

 .. code-block:: java

    package com.example.domain.model;
    
    import java.io.Serializable;
    import java.util.Date;
    
    public class Todo implements Serializable {
    
        private static final long serialVersionUID = 1L;
    
        private String todoId;
    
        private String todoTitle;
    
        private boolean finished;
    
        private Date createdAt;
    
        private long version;
    
        public String getTodoId() {
            return todoId;
        }
    
        public void setTodoId(String todoId) {
            this.todoId = todoId;
        }
    
        public String getTodoTitle() {
            return todoTitle;
        }
    
        public void setTodoTitle(String todoTitle) {
            this.todoTitle = todoTitle;
        }
    
        public boolean isFinished() {
            return finished;
        }
    
        public void setFinished(boolean finished) {
            this.finished = finished;
        }
    
        public Date getCreatedAt() {
            return createdAt;
        }
    
        public void setCreatedAt(Date createdAt) {
            this.createdAt = createdAt;
        }
    
        public long getVersion() {
            return version;
        }
    
        public void setVersion(long version) {
            this.version = version;
        }
    
    }

 .. tip:: **How to map a column name separated by an underscore and a property name in camel case format**
 
         In the above example, the difference between a column name separated by an underscore and a property name in camel case format is resolved by using AS clause.
         However, it can be implemented by changing MyBatis3 configuration
         if only the difference between a column name separated by an underscore and a property name in camel case format is to be resolved.

|

When the physical column name of a table is separated by an underscore,
automatic mapping can be performed in JavaBean property in the camel case format
by adding following settings to MyBatis configuration file (\ :file:`mybatis-config.xml`\).

- :file:`projectName-domain/src/main/resources/META-INF/mybatis/mybatis-config.xml`

 .. code-block:: xml
    :emphasize-lines: 8-9

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
      PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>

        <settings>
            <!-- (3) -->
            <setting name="mapUnderscoreToCamelCase" value="true" />
        </settings>

    </configuration>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (3)
      - Add the settings to set \ ``mapUnderscoreToCamelCase`` \  to \ `true`\.
      
        When it is set to \ `true`\, the column name separated by an underscore is automatically converted to camel case format.
        As a typical example, when column name is \ ``"todo_id"``\ , it is converted to \ ``"todoId"``\  and mapping is performed.

- :file:`projectName-domain/src/main/resources/com/example/domain/repository/todo/TodoRepository.xml`

 .. code-block:: xml
    :emphasize-lines: 8-12

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">
    
        <select id="findOne" parameterType="string" resultType="Todo">
            SELECT
                todo_id, /* (4) */
                todo_title,
                finished,
                created_at,
                version
            FROM
                t_todo
            WHERE
                todo_id = #{todoId}
        </select>
    
    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80

   * - Sr. No.
     - Description
   * - (4)
     - A simple SQL can be fetched since AS clause is not required to resolve the difference between the column name separated by an underscore and the property name in the camel case format.

|

.. _DataAccessMyBatis3HowToUseResultMappingByManual:

Manual mapping of search results
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

MyBatis3 provides a mechanism to manually map search result (\ ``ResultSet``\ ) column and JavaBean property
by defining their association in the mapping file.

 .. note:: **Features of manual mapping**

    When manual mapping is used, the association between search result (\ ``ResultSet``\ ) column and JavaBean property
    is defined for each item one by one in the mapping file.
    Therefore, mapping with extremely high flexibility and complexity can be achieved.

    Manual mapping is a method to effectively map the search results (\ ``ResultSet``\) column and JavaBean property for the cases given below.
    
     * When data model (JavaBean) that handles the application and physical table layout do not match
     * When JavaBean has a nested structure (separate JavaBean is nested)

    Also, manual mapping can be mapped efficiently compared with automatic mapping.
    If prevail efficiency of processing, it is desirable to use manual mapping instead of automatic mapping.


|

| How to map search results in JavaBean using manual mapping is given below.
| Since the idea is to explain how to use manual mapping, a simple example wherein automatic mapping can also be performed, is used for the explanation.

For a hands-on implementation example, refer to 

* "\ `MyBatis 3 REFERENCE DOCUMENTATION(Mapper XML Files-Advanced Result Maps-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#Advanced_Result_Maps>`_ \"
* ":ref:`DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnce`"
* ":ref:`DataAccessMyBatis3AppendixNestedSelect`"



- :file:`projectName-domain/src/main/resources/com/example/domain/repository/todo/TodoRepository.xml`

 .. code-block:: xml
    :emphasize-lines: 6-7, 8-9, 10-14, 17-18

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <!-- (1) -->
        <resultMap id="todoResultMap" type="Todo">
            <!-- (2) -->
            <id column="todo_id" property="todoId" />
            <!-- (3) -->
            <result column="todo_title" property="todoTitle" />
            <result column="finished" property="finished" />
            <result column="created_at" property="createdAt" />
            <result column="version" property="version" />
        </resultMap>

        <!-- (4) -->
        <select id="findOne" parameterType="string" resultMap="todoResultMap">
            SELECT
                todo_id,
                todo_title,
                finished,
                created_at,
                version
            FROM
                t_todo
            WHERE
                todo_id = #{todoId}
        </select>

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Define the mapping of search results (\ ``ResultSet``\ ) and JavaBean, in \ ``<resultMap>``\  element.
      
        Specify the ID to identify mapping in \ ``id``\  attribute and the JavaBean class name (or alias) to be mapped, in \ ``type``\  attribute.
        
        Refer to "\ `MyBatis 3 REFERENCE DOCUMENTATION(Mapper XML Files-resultMap-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#resultMap>`_ \" for details of \ ``<resultMap>``\  element, .
    * - (2)
      - Map search results (\ ``ResultSet``\) ID (PK) column and JavaBean property.
      
        Specify mapping of ID (PK) by using \ ``<id>``\  element.
        Specify search result (\ ``ResultSet``\ ) column name in \ ``column``\  attribute and JavaBean property name in \ ``property``\  attribute.
        
        Refer to "\ `MyBatis 3 REFERENCE DOCUMENTATION(Mapper XML Files-id & result-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#id__result>`_ \" for details of \ ``<id>``\  element.
    * - (3)
      - Map a column other than ID (PK) column of search results (\ ``ResultSet``\ ) and JavaBean property.
      
        Specify mapping for column other than ID (PK) using \ ``<result>``\  element.
        Specify search result (\ ``ResultSet``\) column name in \ ``column``\  attribute and JavaBean property name in \ ``property``\  attribute.

        Refer to "\ `MyBatis 3 REFERENCE DOCUMENTATION(Mapper XML Files-id & result-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#id__result>`_ \" for details of \ ``<result>``\  element.
    * - (4)
      - Specify mapping definition ID to be applied, in \ ``resultMap``\  attribute of \ ``<select>``\  element.

 .. note:: **How to use id element and result element**
 
    \ ``<id>``\  element and \ ``<result>``\  element can both be used
    for mapping search results (\ ``ResultSet``\ ) column and JavaBean property.
    However, it is recommended to use \ ``<id>``\  element for mapping of ID (PK) column.
    
    This is because, when \ ``<id>``\  element is used for the mapping of ID (PK) column, the performance of mapping process for related objects and the cache control process of objects provided by MyBatis3
    can show overall improvement.

|

.. _DataAccessMyBatis3HowToUseFind:

Search process for Entity
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
How to implement a search process of Entity for different purposes, is explained below.

Read ":ref:`DataAccessMyBatis3HowToUseResultSetMapping`" before reading how to implement the search process for Entity.

The explanation below is the example wherein a setting is enabled to automatically map column name separated by an underscore in property name with camel case.

- :file:`projectName-domain/src/main/resources/META-INF/mybatis/mybatis-config.xml`

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
      PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>

        <settings>
            <setting name="mapUnderscoreToCamelCase" value="true" />
        </settings>

    </configuration>

|

.. _DataAccessMyBatis3HowToUseFindOne:

Fetching a single key Entity
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Implementation example wherein, a single Entity is fetched by specifying PK rather than configuring PK in a single column, is given below.

* Define method in Repository interface.

 .. code-block:: java

    package com.example.domain.repository.todo;

    import com.example.domain.model.Todo;

    public interface TodoRepository {

        // (1)
        Todo findOne(String todoId);

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - In the above example, \ ``findOne``\  method is defined as the method to fetch a single Todo object matching with \ ``todoId``\  (PK) specified in the argument.
        

|

* Define SQL in the mapping file.

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <!-- (2) -->
        <select id="findOne" parameterType="string" resultType="Todo">
            /* (3) */
            SELECT
                todo_id,
                todo_title,
                finished,
                created_at,
                version
            FROM
                t_todo
            /* (4) */
            WHERE
                todo_id = #{todoId}
        </select>

    </mapper>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 10 70

    * - Sr. No.
      - Attribute
      - Description
    * - (2)
      - \-
      - Implements SQL in \ ``select``\  element with search result  0 to 1 record.
      
        In the above example, the SQL fetching a record that matches with ID (PK) is implemented.

        For details of \ ``select``\  element,
        refer to "`MyBatis3 REFERENCE DOCUMENTATION (Mapper XML Files-select-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#select>`_\".

    * -
      - id
      - Specifies method name of the method defined in Repository interface.
    * -
      - parameterType
      - Specifies parameter fully qualified class name (or alias name).
    * -
      - resultType
      - Specifies the fully qualified class name (or alias) of JavaBean that maps the search results (\ ``ResultSet``\ ).
      
        When manual mapping is used, specify mapping definition to be applied, by using \ ``resultMap``\  attribute in place of \ ``resultType``\  attribute.
        
        Refer to ":ref:`DataAccessMyBatis3HowToUseResultMappingByManual`" for manual mapping.
    * - (3)
      - \-
      - Specify the column to be fetched.
      
        In the above example, automatic mapping is used as the method to map search results (\ ``ResultSet``\ ) to JavaBean.
        Refer to ":ref:`DataAccessMyBatis3HowToUseResultMappingByAuto`" for automatic mapping.
    * - (4)
      - \-
      - Specify search conditions in WHERE clause.
      
        Specify the value to be bound in search condition as the bind value of \ ``#{variableName}``\  format. In the above example,
        \ ``#{todoId}``\  acts as the bind variable.
        
        When argument type of Repository interface is of simple type like \ ``String``\, any name can be specified
        as the bind variable name, however when the argument type is JavaBean,
        JavaBean property name must be specified in the bind variable name.

 .. note:: **Simple type bind variable name**
 
    In case of a simple type like \ ``String``\, there is no restriction for the bind variable name, however, it is recommended to use the value same as the argument name of the method.

|

* Apply DI to Repository in Service class and call the interface method of Repository.

 .. code-block:: java

    package com.example.domain.service.todo;

    import javax.inject.Inject;

    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;

    import com.example.domain.model.Todo;
    import com.example.domain.repository.todo.TodoRepository;

    @Transactional
    @Service
    public class TodoServiceImpl implements TodoService {

        // (5)
        @Inject
        TodoRepository todoRepository;

        @Transactional(readOnly = true)
        @Override
        public Todo getTodo(String todoId) {
            // (6)
            Todo todo = todoRepository.findOne(todoId);
            if (todo == null) { // (7)
                throw new ResourceNotFoundException(ResultMessages.error().add(
                        "e.ex.td.5001", todoId));
            }
            return todo;
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (5)
      - Apply DI to Repository interface in Service class.
    * - (6)
      - Call Repository interface method and fetch 1 Entity.
    * - (7)
      - Since \ ``null``\  is returned when the search result shows 0 records,
        if required, implement the process when Entity cannot be fetched.

        In the above example, when Entity cannot be fetched, "resource not detected" error is generated.

|

Fetching Entity of composite key
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| The implementation example of fetching a single Entity by specifying PK rather than configuring PK in multiple columns, is given below.
| Basic settings are same as while configuring PK in a single column, however the way to specify a method argument for Repository interface is different.

* Defining the method in Repository interface.

 .. code-block:: java

    package com.example.domain.repository.order;
    
    import org.apache.ibatis.annotations.Param;
    
    import com.example.domain.model.OrderHistory;
    
    public interface OrderHistoryRepository {
    
       // (1)
       OrderHistory findOne(@Param("orderId") String orderId,
               @Param("historyId") int historyId);
    
    }
   
 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Define an argument corresponding to the column that configures PK, in the method.

        In the above example, \ ``orderId``\  and \ ``historyId``\  are defined in the argument as PK for the table that manages change history of orders received.

 .. tip:: **Bind variable name while specifying multiple method arguments**
 
    When multiple method arguments of Repository interface are specified, it is recommended to specify \ ``@org.apache.ibatis.annotations.Param``\  annotation in the argument.
    "Bind variable name" specified while selecting the value from mapping file is specified in the \ ``value``\  attribute of \ ``@Param``\  annotation.
     
    As shown in the above example, the value specified in the argument can be bound in SQL by specifying \ ``#{orderId}``\  and \ ``#{historyId}``\  from mapping file.

     .. code-block:: xml
    
        <?xml version="1.0" encoding="UTF-8"?>
        <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
        <mapper namespace="com.example.domain.repository.order.OrderHistoryRepository">
    
            <select id="findOne" resultType="OrderHistory">
                SELECT
                    order_id,
                    history_id,
                    order_name,
                    operation_type,
                    created_at"
                FROM
                    t_order_history
                WHERE
                    order_id = #{orderId}
                AND
                    history_id = #{historyId}
            </select>
            
        </mapper>

    Although it is not mandatory to specify \ ``@Param``\  annotation,
    if it is not specified, a mechanical bind variable name needs to be specified as given below.
    The bind variable name when \  ``@Param``\  annotation is not specified is formed as, " "param" + declared position of the argument(start from 1)",
    and thus can hamper maintainability and readability of the source code.
    
     .. code-block:: xml
    
        <!-- omitted -->
    
        WHERE
            order_id = #{param1}
        AND
            history_id = #{param2}

        <!-- omitted -->

|

.. _DataAccessMyBatis3HowToUseFindMultiple:

Entity search
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Implementation example is given below wherein a SQL with search results 0 to N records is executed and multiple records of Entity are fetched.

 .. warning::

    If the search results data is in a large quantity, using ":ref:`DataAccessMyBatis3HowToExtendResultHandler`" should be considered.

|

* Defining the method for fetching multiple records of Entity.

 .. code-block:: java

    package com.example.domain.repository.todo;

    import java.util.List;

    import com.example.domain.model.Todo;

    public interface TodoRepository {

        // (1)
        List<Todo> findAllByCriteria(TodoCriteria criteria);

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - In the above example, \ ``findAllByCriteria``\  is defined as the method to fetch multiple records of Todo object in a list format that matches with
        the JavaBean (\ ``TodoCriteria``\ ) retaining the search conditions.

 .. tip::

    In the above example, the return value of method is specified as \ ``java.util.List``\  however,
    search results can also be received as \ ``java.util.Map``\.

    When the results are received in \ ``Map``\,

    * PK value is stored in \ ``key``\  of \ ``Map``\
    * Entity object is stored in \ ``value``\  of \ ``Map``\.



    When search results are received by \ ``Map``\ , \ ``java.util.HashMap``\  instance is returned.
    Hence, it should be noted that the alignment sequence of \ ``Map``\ is not guaranteed.

    Implementation example is given below.

     .. code-block:: java

        package com.example.domain.repository.todo;

        import java.util.Map;

        import com.example.domain.model.Todo;
        import org.apache.ibatis.annotations.MapKey;

        public interface TodoRepository {

            @MapKey("todoId")
            Map<String, Todo> findAllByCriteria(TodoCriteria criteria);

        }

    When search results are received by \ ``Map``\ , \ ``@org.apache.ibatis.annotations.MapKey``\  annotation is specified in the method.
    Property name that is handled as \ ``key``\  of \ ``Map``\  is specified in the \ ``value``\  attribute of annotation.
    In the above example, PK of Todo object (\ ``todoId``\ ) is specified.



|

* Create JavaBean that retains the search conditions.

 .. code-block:: java

    package com.example.domain.repository.todo;

    import java.io.Serializable;
    import java.util.Date;

    public class TodoCriteria implements Serializable {

        private static final long serialVersionUID = 1L;

        private String title;

        private Date createdAt;

        public String getTitle() {
            return title;
        }

        public void setTitle(String title) {
            this.title = title;
        }

        public Date getCreatedAt() {
            return createdAt;
        }

        public void setCreatedAt(Date createdAt) {
            this.createdAt = createdAt;
        }

    }

 .. note:: **Creating a JavaBean for retaining search conditions**

    Although it is not mandatory to create a JavaBean for retaining search conditions, it is recommended to create one,
    to clearly identify the role of the stored value. However, implementation can also be performed without creating a JavaBean.
    
    \ **The decision standards of the cases for which JavaBean is created and those for which it is not created should be clearly stated to the programmers by the Architect,
    so that an overall uniform application can be created.**\

    Implementation example when JavaBean is not created is given below.

     .. code-block:: java

        package com.example.domain.repository.todo;

        import java.util.List;

        import com.example.domain.model.Todo;

        public interface TodoRepository {

            List<Todo> findAllByCriteria(@Param("title") String title,
                    @Param("createdAt") Date createdAt);

        }

    When JavaBean is not created, the search conditions are declared one by one in an argument
    and "bind variable name" is specified in \ ``value``\  attribute of \ ``@Param``\  annotation.
    Multiple search conditions can be passed to SQL by defining the method described above.

|

* Define SQL in the mapping file.

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <!-- (2) -->
        <select id="findAllByCriteria" parameterType="TodoCriteria" resultType="Todo">
            <![CDATA[
            SELECT
                todo_id,
                todo_title,
                finished,
                created_at,
                version
            FROM
                t_todo
            WHERE
                todo_title LIKE #{title} || '%' ESCAPE '~'
            AND
                created_at < #{createdAt}
            /* (3) */
            ORDER BY
                todo_id
            ]]>
        </select>

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (2)
      - Implement the SQL with search results 0 to N records in \ ``select``\  element.
      
        In the above example, Todo records that match with the conditions specified in \ ``todo_title``\  and \ ``created_at``\  are fetched.
    * - (3)
      - Specify sort condition.
      
        When multiple records are to be fetched, sort condition is specified.
        Particularly, sort condition must be specified in the SQL that fetches the record displayed on the screen.

 .. tip:: **How to use CDATA section**
 
    When a XML character (\ ``"<"``\  or \ ``">"``\  etc.) that needs to be escaped in SQL is specified,
    the readability of SQL can be maintained by using CDATA section.
    When CDATA section is not used, entity reference characters such as \ ``"&lt;"``\ , \ ``"&gt;"``\  need to be specified,
    and may lead to reduced SQL readability.
    
    In the above example, CDATA section is specified since \ ``"<"``\  is used as the condition for \ ``created_at``\.

|

.. _DataAccessMyBatis3HowToUseCount:

Fetching Entity records
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
The implementation example of fetching Entity records matching with search conditions is given below.

* Defining the method for fetching Entity records matching with search conditions.

 .. code-block:: java

    package com.example.domain.repository.todo;

    public interface TodoRepository {

        // (1)
        long countByFinished(boolean finished);

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Specify numeric type (\ ``int``\  or \ ``long``\  etc.) for the return value of method used to fetch records.
      
        In the above example, \ ``long``\  is specified.

|

* Define SQL in the mapping file.

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <!-- (2) -->
        <select id="countByFinished" parameterType="_boolean" resultType="_long">
            SELECT
                COUNT(*)
            FROM
                t_todo
            WHERE
                finished = #{finished}
        </select>

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (2)
      - Execute the SQL that fetches the records.
      
        Type of return value is specified in \ ``resultType``\  attribute.
      
        In the above example, primitive type alias name for specifying   \ ``long``\  is specified.

 .. tip:: **Primitive type alias name**
 
    \ ``"_"``\  (underscore) should be specified at the beginning of the primitive type alias name.
    When \ ``"_"``\(underscore) is not specified, it is handled as primitive wrapper type (\ ``java.lang.Long``\  etc.) alias.

|

.. _DataAccessMyBatis3HowToUseFindPageUsingMyBatisFunction:

Pagination search of Entity (MyBatis3 standard method)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Implementation example to search an Entity by using MyBatis3 functionality for specifying the fetching scope, is given below.

``org.apache.ibatis.session.RowBounds``\  class is provided in MyBatis as the class to specify the fetch range.
and in SQL, it is not necessary to describe the conditions for fetch range.

 .. warning:: **Precautions when large number of data records match the search conditions**
 
    Standard MyBatis method is to move the cursor and skip the data which is outside the fetch range of search results (\ ``ResultSet``\ ).
    Hence, in proportion to the data records that match with search conditions, issues like memory exhaustion or performance degradation of cursor movement are more likely to occur.

    According to JDBC result set type, the cursor movement processing supports following 2 types. Default behavior is
    dependent on the default result set type of JDBC driver.

    * When result set type is \ ``FORWARD_ONLY``\ , \ ``ResultSet#next()``\  is repeatedly called and data outside the fetching range is skipped.
    * When result set type is \ ``SCROLL_SENSITIVE``\  or \ ``SCROLL_INSENSITIVE``\ , \ ``ResultSet#absolute(int)``\  is called and data outside the scope of fetching range is skipped.

    Performance degradation can be restricted to a minimum by using \ ``ResultSet#absolute(int)``\  however,
    it is dependent on the implementation of JDBC driver. If process same as \ ``ResultSet#next()``\  is performed internally,
    it is not possible to prevent memory exhaustion or performance deterioration.

    \ **When there is a possibility of large number of data records matching the search conditions, SQL refine method should be adopted
    instead of pagination search which is a MyBatis3 standard method.**\

|

* Defining the method for performing Entity pagination search.

 .. code-block:: java

    ackage com.example.domain.repository.todo;

    import java.util.List;

    import org.apache.ibatis.session.RowBounds;

    import com.example.domain.model.Todo;

    public interface TodoRepository {

        // (1)
        long countByCriteria(TodoCriteria criteria);

        // (2)
        List<Todo> findPageByCriteria(TodoCriteria criteria,
            RowBounds rowBounds);

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Define the method that fetches total records of Entity matching with search conditions.
    * - (2)
      - Define the method that extracts those Entities that fall in the fetching range from the Entities matching with search conditions.
      
        \ ``RowBounds``\  that retains the information of fetch range (offset and limit) is specified as the argument of defined method.

|

* Define SQL in the mapping file.

  Since MyBatis3 performs the process to extract records of corresponding range from the search results, it is not necessary to filter the records within the fetch range using SQL.

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <select id="countByCriteria" parameterType="TodoCriteria" resultType="_long">
            <![CDATA[
            SELECT
                COUNT(*)
            FROM
                t_todo
            WHERE
                todo_title LIKE #{title} || '%' ESCAPE '~'
            AND
                created_at < #{createdAt}
            ]]>
        </select>

        <select id="findPageByCriteria" parameterType="TodoCriteria" resultType="Todo">
            <![CDATA[
            SELECT
                todo_id,
                todo_title,
                finished,
                created_at,
                version
            FROM
                t_todo
            WHERE
                todo_title LIKE #{title} || '%' ESCAPE '~'
            AND
                created_at < #{createdAt}
            ORDER BY
                todo_id
            ]]>
        </select>

    </mapper>

 .. note:: **Standardization of WHERE clause**
 
    When pagination search is performed, it is recommended to standardize the WHERE clause specified in "SQL that fetches total number of records for Entities matching with search condition"
    and "SQL that fetches the list of the Entities matching with search conditions", using include function of MyBatis3.
    
    
    A standardized WHERE clause of above SQL is defined as below.
    Refer to ":ref:`DataAccessMyBatis3HowToExtendSqlShare`" for details.

     .. code-block:: xml
        :emphasize-lines: 1, 15, 27

        <sql id="findPageByCriteriaWherePhrase">
            <![CDATA[
            WHERE
                todo_title LIKE #{title} || '%' ESCAPE '~'
            AND
                created_at < #{createdAt}
            ]]>
        </sql>

        <select id="countByCriteria" parameterType="TodoCriteria" resultType="_long">
            SELECT
                COUNT(*)
            FROM
                t_todo
            <include refid="findPageByCriteriaWherePhrase"/>
        </select>

        <select id="findPageByCriteria" parameterType="TodoCriteria" resultType="Todo">
            SELECT
                todo_id,
                todo_title,
                finished,
                created_at,
                version
            FROM
                t_todo
            <include refid="findPageByCriteriaWherePhrase"/>
            ORDER BY
                todo_id
        </select>

 .. note:: **How to explicitly specify a result set type**

    Result set type is specified in \ ``resultType``\  attribute when it is to be specified explicitly.
    When the default result set type of JDBC driver is \ ``FORWARD_ONLY``\ , it is recommended to specify \ ``SCROLL_INSENSITIVE``\ .

     .. code-block:: xml
        :emphasize-lines: 2

        <select id="findPageByCriteria" parameterType="TodoCriteria" resultType="Todo"
            resultSetType="SCROLL_INSENSITIVE">
            <!-- omitted -->
        </select>

|

* Implementing pagination search process in Service class.

 .. code-block:: java

    // omitted

    @Transactional
    @Service
    public class TodoServiceImpl implements TodoService {
    
        @Inject
        TodoRepository todoRepository;
        
        // omitted

        @Transactional(readOnly = true)
        @Override
        public Page<Todo> searchTodos(TodoCriteria criteria, Pageable pageable) {
            // (3)
            long total = todoRepository.countByCriteria(criteria);
            List<Todo> todos;
            if (0 < total) {
                // (4)
                RowBounds rowBounds = new RowBounds(pageable.getOffset(), 
                    pageable.getPageSize());
                // (5)
                todos = todoRepository.findPageByCriteria(criteria, rowBounds);
            } else {
                // (6)
                todos = Collections.emptyList();
            }
            // (7)
            return new PageImpl<>(todos, pageable, total);
        }

        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (3)
      - First, fetch the total Entity records matching with search condition.
    * - (4)
      - Generate \ ``RowBounds``\  object that specifies fetch range of pagination search when Entities matching with search conditions exist.

        Specify "skip record" in the first argument (\ ``offset``\ ) and
        "maximum fetch records" in the second argument (\ ``limit``\ ) of \ ``RowBounds``\.
        For the values to be specified as argument, it is advisable to specify the values fetched by calling \ ``getOffset``\  method and \ ``getPageSize``\  method of \ ``Pageable``\  object
        provided by Spring Data Commons.

        Basically, the fetch range is

        * Records 1st to 20th when \ ``0``\  is specified in offset and \ ``20``\  is specified in limit
        * Records 21st to 40th when \ ``20``\ is specified in offset and \ ``20``\  is specified in limit

        

    * - (5)
      - Call Repository method and fetch Entities in the fetch range that match with search conditions.
    * - (6)
      - When the Entities that match with search conditions do not exist, set empty list in the search results.
    * - (7)
      - Create and return page information (\ ``org.springframework.data.domain.PageImpl``\).

|

.. _DataAccessMyBatis3HowToUseFindPageUsingSqlFilter:

Pagination search for Entity (SQL refinement method)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Implementation example to search an Entity by using range search mechanism provided by database, is given below.

Since SQL refinement method uses range search mechanism provided by database,
Entity of fetch range can be fetched efficiently as compared to standard method of MyBatis3.

 .. note::

    \ **It is recommended to adopt the SQL refining method when a large volume of data matching with search condition exists.**\ 

|

* Defining the method for performing Entity pagination search.

 .. code-block:: java

    package com.example.domain.repository.todo;
    
    import java.util.List;
    
    import org.apache.ibatis.annotations.Param;
    import org.springframework.data.domain.Pageable;
    
    import com.example.domain.model.Todo;
    
    public interface TodoRepository {
    
        // (1)
        long countByCriteria(
                @Param("criteria") TodoCriteria criteria);

        // (2)
        List<Todo> findPageByCriteria(
                @Param("criteria") TodoCriteria criteria,
                @Param("pageable") Pageable pageable);
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Define a method that fetches total Entity records matching with search conditions.
    * - (2)
      - Define a method to extract entities that can be fetched from the Entities matching with search conditions.

        \ ``org.springframework.data.domain.Pageable``\  that retains the information within the fetch range (offset and limit) is specified as an argument for defined method.

 .. note:: **Reason why the argument specifies @Param annotation for a single method**
 
     In the above example, the argument specifies \ ``@Param``\  annotation for a single method (\ ``countByCriteria``\).
     This is to standardize WHERE clause and the SQL executed when \ ``findPageByCriteria``\  method is called.
     
     By specifying bind variable name in the argument using \ ``@Param``\ annotation, nested structure of bind variable name specified in SQL is combined.
     
     
     A typical SQL implementation example is given below.

|

* Define SQL in the mapping file.

  Fetch range records are refined by SQL.

 .. code-block:: xml
    :emphasize-lines: 8, 36-37, 38-39

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <sql id="findPageByCriteriaWherePhrase">
            <![CDATA[
            /* (3) */
            WHERE
                todo_title LIKE #{criteria.title} || '%' ESCAPE '~'
            AND
                created_at < #{criteria.createdAt}
            ]]>
        </sql>
    
        <select id="countByCriteria" resultType="_long">
            SELECT
                COUNT(*)
            FROM
                t_todo
            <include refid="findPageByCriteriaWherePhrase" />
        </select>
    
        <select id="findPageByCriteria" resultType="Todo">
            SELECT
                todo_id,
                todo_title,
                finished,
                created_at,
                version
            FROM
                t_todo
            <include refid="findPageByCriteriaWherePhrase" />
            ORDER BY
                todo_id
            LIMIT
                #{pageable.pageSize} /* (4) */
            OFFSET
                #{pageable.offset}  /* (4) */
        </select>

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (3)
      - \ ``@Param("criteria")``\  is specified in the arguments of \ ``countByCriteria``\  and \ ``findPageByCriteria``\  methods,
        Hence, the bind variable name specified in SQL is in \ ``criteria.field name``\  format.
    * - (4)
      - Extract only necessary records by using the fetch range mechanism provided by database.
      
        "Skip record" is stored in \ ``offset``\  of \ ``Pageable``\  object whereas
        "maximum fetch records" is stored in \ ``pageSize``\.

        Above example is the implementation example with H2 Database.

|

* Implement a pagination search process in the Service class.

 .. code-block:: java

    // omitted

    @Transactional
    @Service
    public class TodoServiceImpl implements TodoService {
    
        @Inject
        TodoRepository todoRepository;
        
        // omitted

        @Transactional(readOnly = true)
        @Override
        public Page<Todo> searchTodos(TodoCriteria criteria,
                Pageable pageable) {
            long total = todoRepository.countByCriteria(criteria);
            List<Todo> todos;
            if (0 < total) {
                // (5)
                todos = todoRepository.findPageByCriteria(criteria,
                        pageable);
            } else {
                todos = Collections.emptyList();
            }
            return new PageImpl<>(todos, pageable, total);
        }

        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (5)
      - Call Repository method and fetch Entity within the fetch range matching with search condition.
      
        \ ``Pageable``\  object received by the argument can be passed as it is when calling Repository method.

|

.. _DataAccessMyBatis3HowToUseCreate:

Entity registration process
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

how to register an Entity for different purposes is explained with implementation example.

.. _DataAccessMyBatis3HowToUseCreateOne:

Registering a single Entity record
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Implementation example for registering a single Entity record is given below.

* Defining the method in Repository interface.

 .. code-block:: java

    package com.example.domain.repository.todo;
    
    import com.example.domain.model.Todo;
    
    public interface TodoRepository {
    
        // (1)
        void create(Todo todo);
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - In the above example, \ ``create``\  method is defined as the method for registering a single Todo object specified in the argument.
        

\

 .. note:: **Return value of the method that registers Entity**
 
    Return value for the method that registers Entity can be \ ``void``\ .

    However, when SQL that inserts selected results is executed,
    \ ``boolean``\  or numeric value type (\ ``int``\  or \ ``long``\ ) should be set as the return value based on application requirements.

    * When \ ``boolean``\  is specified as return value, \ ``false``\  is returned when 0 records are registered and ``true``\  is returned when 1 or more records are registered.
    * When numeric value type is specified as return value, number of registered records is returned.

|

* Define SQL in the mapping file.

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <!-- (2) -->
        <insert id="create" parameterType="Todo">
            INSERT INTO
                t_todo
            (
                todo_id,
                todo_title,
                finished,
                created_at,
                version
            )
            /* (3) */
            VALUES
            (
                #{todoId},
                #{todoTitle},
                #{finished},
                #{createdAt},
                #{version}
            )
        </insert>

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (2)
      - Implement the INSERT SQL in the insert element.
      
        Specify name of the method defined in Repository interface, in \ ``id``\  attribute.

        For details of \ ``insert``\  element,
        refer to "`MyBatis3 REFERENCE DOCUMENTATION (Mapper XML Files-insert, update and delete-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#insert_update_and_delete>`_\".

    * - (3)
      - Specify configuration value at the time of record registration in VALUE clause.
      
        The value to be bound in VALUE clause is specified as the bind variable of #{variableName} format.
        In the above example, since JavaBean (\ ``Todo```\ ) is specified as an argument of Repository interface,
        JavaBean property name is specified in the bind variable name.

|

* Apply DI to Repository in Service class and call Repository interface method.

 .. code-block:: java


    package com.example.domain.service.todo;

    import java.util.UUID;

    import javax.inject.Inject;

    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;
    import org.terasoluna.gfw.common.date.jodatime.JodaTimeDateFactory;

    import com.example.domain.model.Todo;
    import com.example.domain.repository.todo.TodoRepository;

    @Transactional
    @Service
    public class TodoServiceImpl implements TodoService {

        // (4)
        @Inject
        TodoRepository todoRepository;

        @Inject
        JodaTimeDateFactory dateFactory;

        @Override
        public Todo create(Todo todo) {
            // (5)
            todo.setTodoId(UUID.randomUUID().toString());
            todo.setCreatedAt(dateFactory.newDate());
            todo.setFinished(false);
            todo.setVersion(1);
            // (6)
            todoRepository.create(todo);
            // (7)
            return todo;
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (4)
      - Apply DI to Repository interface in Service class.
    * - (5)
      - Set the value for Entity object passed in the argument based on the application requirements.

        In the above example,

        * "UUID" as an ID
        * "System date and time" as registration date and time"
        * "\ ``false``\  : Incomplete" in the completion flag
        * "\ ``1``\" in the version
        
        are set.
    * - (6)
      - Call Repository interface method and register the single Entity record.
    * - (7)
      - Return registered Entity.
      
        When registration value is set in the Service class process, it is recommended to return the registered Entity object as return value.




.. _DataAccessMyBatis3HowToUseGenId:

Generating key
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

An implementation example is given in ":ref:`DataAccessMyBatis3HowToUseCreateOne`" for generating key (ID) in Service class.

However, MyBatis3 provides a mechanism for generating key in mapping file.

 .. note:: **Cases wherein key generation functionality of MyBatis3 is used**
 
    When a database function (function or ID column etc.) is used for generating key,
    it is recommended to use the mechanism of MyBatis3 key generation functionality.

|

There are two kinds of methods for generating key.

* A method wherein the result obtained by calling the function etc. provided by database, is handled as the key
* A method wherein the result obtained by calling ID column provided by database (IDENTITY type, AUTO_INCREMENT type etc.) + \ ``Statement#getGeneratedKeys()``\  added by JDBC3.0 is handled as the key.

|

Method wherein result obtained by calling the function etc. provided by database is handled as a key, is explained first.
In the example given below, H2 Database is used as the database.

 .. code-block:: xml
    :emphasize-lines: 7-11

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <insert id="create" parameterType="Todo">
            <!-- (1) -->
            <selectKey keyProperty="todoId" resultType="string" order="BEFORE">
                /* (2) */
                SELECT RANDOM_UUID()
            </selectKey>
            INSERT INTO
                t_todo
            (
                todo_id,
                todo_title,
                finished,
                created_at,
                version
            )
            VALUES
            (
                #{todoId},
                #{todoTitle},
                #{finished},
                #{createdAt},
                #{version}
            )
        </insert>

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 10 70

    * - Sr. No.
      - Attribute
      - Description
    * - (1)
      - \-
      - Implements the SQL to generate key in \ ``selectKey``\  element.

        In the above example, UUID is fetched by using the function provided by database.

        For details of \ ``selectKey``\ ,
        refer to "`MyBatis3 REFERENCE DOCUMENTATION (Mapper XML Files-insert, update and delete-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#insert_update_and_delete>`_\".
    * -
      - keyProperty
      - Specifies the property name of Entity that stores fetched key value.

        In the above example, key that is generated in \ ``todoId``\  property of Entity, is set.
    * -
      - resultType
      - Specifies the type of key value to be fetched by executing the SQL.
    * -
      - order
      - Specifies the timing when the SQL for key generation (\ ``BEFORE``\ or \ ``AFTER``\) is executed.

        * When \ ``BEFORE``\  is specified, INSERT statement is executed after the results obtained by executing SQL specified in \ ``selectKey``\  element are reflected in Entity.
        * When \ ``AFTER``\  is specified, SQL specified in \ ``selectKey``\  element is executed after executing INSERT statement and fetched value is reflected in Entity.
    * - (2)
      - \-
      - Implement the SQL for generating key.

        In the above example, the function that generates UUID of H2 Database is called and key is generated.
        Implementation wherein value fetched from sequence object is formatted in a string can be cited as the typical example of key generation.

|

Next, the method wherein result obtained by calling ID column provided by database + \ ``Statement#getGeneratedKeys()``\  added by JDBC3.0ratedKeys() is handled as a key, is explained.
In the example given below, H2 Database is used as the database.

 .. code-block:: xml
    :emphasize-lines: 6-7

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.audit.AuditLogRepository">

        <!-- (3) -->
        <insert id="create" parameterType="Todo" useGeneratedKeys="true" keyProperty="logId">
            INSERT INTO
                t_audit_log
            (
                level,
                message,
                created_at,
            )
            VALUES
            (
                #{level},
                #{message},
                #{createdAt},
            )
        </insert>
        
    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 10 70

    * - Sr. No.
      - Attribute
      - Description
    * - (3)
      - useGeneratedKeys
      - When \ ``true``\  is specified, function that fetches key by calling ID column + \ ``Statement#getGeneratedKeys()``\  can be used.

        For details of \ ``useGeneratedKeys``\ ,
        refer to "`MyBatis3 REFERENCE DOCUMENTATION (Mapper XML Files-insert, update and delete-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#insert_update_and_delete>`_\".
    * -
      - keyProperty
      - Specify property name of the Entity that stores the key value which is automatically incremented in the database.

        In the above example, key value fetched by \ ``Statement#getGeneratedKeys()``\  in \ ``logId``\  property of Entity, is set after executing INSERT statement.


|

.. _DataAccessMyBatis3HowToUseCreateMultiple:

Batch registration of Entity
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Implementation example for registering Entity in a batch is shown below.

The methods to perform batch registration of Entity are as given below.

* Execute INSERT statement that registers multiple records at the same time.

* There is a method to use the JDBC batch update functionality



Refer to ":ref:`DataAccessMyBatis3HowToExtendExecutorTypeBatch`" for details on how to use the JDBC batch update functionality.

How to execute the INSERT statement that registers multiple records at the same time, is explained below.
In the example below, H2 Database is used as the database.


* Defining the method in Repository interface.

 .. code-block:: java

    package com.example.domain.repository.todo;
    
    import java.util.List;

    import com.example.domain.model.Todo;
    
    public interface TodoRepository {
    
        // (1)
        void createAll(List<Todo> todos);
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - In the above example, \ ``createAll``\  method is defined as the method to perform batch registration for a list of Todo objects specified in the argument.
        

|

* Define the SQL in mapping file.

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <insert id="createAll" parameterType="list">
            INSERT INTO
                t_todo
            (
                todo_id,
                todo_title,
                finished,
                created_at,
                version
            )
            /* (2) */
            VALUES
            /* (3) */
            <foreach collection="list" item="todo" separator=",">
            (
                #{todo.todoId},
                #{todo.todoTitle},
                #{todo.finished},
                #{todo.createdAt},
                #{todo.version}
            )
            </foreach>
        </insert>

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 10 70

    * - Sr. No.
      - Attribute
      - Description
    * - (2)
      - \-
      - Specifies the configuration value at the time of registering records in VALUE clause.
    * - (3)
      - \-
      - Repeats the process for the list of Todo objects passed as argument, by using \ ``foreach``\  element.

        For details of \ ``foreach``\  details,
        refer to "`MyBatis3 REFERENCE DOCUMENTATION (Dynamic SQL-foreach-) <http://mybatis.github.io/mybatis-3/dynamic-sql.html#foreach>`_\".
    * -
      - collection
      - Specifies the collection for processing.
      
        In the example given above, the process is repeated for the list of Repository method arguments.
        When \ ``@Param``\  is not specified in the argument of Repository method, \ ``"list"``\  is specified.
        When \ ``@Param``\  is specified, the value specified in \ ``value``\  attribute of \ ``@Param``\  is specified.
    * -
      - item
      - Specifies the local variable name that retains one element from the list.
      
        JavaBean property can be accessed from the SQL in \ ``foreach``\  element, in #{Local variable name.Property name} format.
    * -
      - separator
      - Specifies the string to separate elements in the list.
      
        In the above example, by specifying \ ``","``\ , the VALUE clause for each element is separated with \ ``","``\ .

\

 .. note:: ** Precautions when using SQL that registers multiple records at the same time**

    When SQL that registers multiple records concurrently is executed, ":ref:`DataAccessMyBatis3HowToUseGenId`" described earlier cannot be used.

|

* Following SQL is generated and executed.

 .. code-block:: sql

    INSERT INTO
        t_todo
    (
        todo_id,
        todo_title,
        finished,
        created_at,
        version
    )
    VALUES 
    (
        '99243507-1b02-45b6-bfb6-d9b89f044e2d',
        'todo title 1',
        false,
        '09/17/2014 23:59:59.999',
        1
    )
    , 
    (
        '66b096f1-791f-412f-9a0a-ee4a3a9186c2',
        'todo title 2',
        0,
        '09/17/2014 23:59:59.999',
        1
    ) 

 .. tip::

    The support status and syntax for the SQL that performs batch registration differ depending on database and version.
    The links for reference pages of major databases are given below.

    * `Oracle 12c <http://docs.oracle.com/database/121/SQLRF/statements_9014.htm>`_
    * `DB2 10.5 <https://www.ibm.com/support/knowledgecenter/SSEPGG_10.5.0/com.ibm.db2.luw.sql.ref.doc/doc/r0000970.html>`_
    * `PostgreSQL 9.4 <http://www.postgresql.org/docs/9.4/static/sql-insert.html>`_
    * `MySQL 5.7 <http://dev.mysql.com/doc/refman/5.7/en/insert.html>`_

|

.. _DataAccessMyBatis3HowToUseUpdate:

Update process of Entity
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Entity update method for different purposes is explained with implementation example.


.. _DataAccessMyBatis3HowToUseUpdateOne:

Updating a single Entity
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Implementation example for updating a single Entity is given below.

 .. note::

     Hereafter, an implementation example is explained wherein optimistic locking is performed by using version column.
     However, process related to optimistic locking need not be performed when optimistic locking is not required.

     Refer to ":doc:`ExclusionControl` for details of exclusive control.

|

* Defining the method in Repository interface.

 .. code-block:: java

    package com.example.domain.repository.todo;
    
    import com.example.domain.model.Todo;
    
    public interface TodoRepository {
    
        // (1)
        boolean update(Todo todo);
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - In the above example, \ ``update``\  method is defined as the method to update single Todo object specified in the argument.
        

\

 .. note:: **Return value of the method that updates a single Entity**
 
    The return value of the method that updates single Entity can be \ ``boolean``\.

    However, when multiple records are obtained as update result and it is necessary to handle it as data mismatch error,
    numeric value type (\ ``int``\  or \ ``long``\ ) needs to be specified as return value and it needs to be checked that a single update record exists.
    When main key is used as the update condition, return value can be set as \ ``boolean``\  since multiple records are not obtained as update result.

    * When \ ``boolean``\  is specified as return value, \ ``false``\  is returned when update records are 0 and \ ``true``\ is returned when update records are 1 or more.
    * When numeric value is specified as return value, number of update records is returned.


|

* Defining SQL in mapping file.

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <!-- (2) -->
        <update id="update" parameterType="Todo">
            UPDATE
                t_todo
            SET
                todo_title = #{todoTitle},
                finished = #{finished},
                version = version + 1
            WHERE
                todo_id = #{todoId}
            AND
                version = #{version}
        </update>

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (2)
      - Implement the UPDATE SQL in \ ``update``\  element.
      
        Specify the method name defined in Repository interface, in \ ``id``\  attribute.

        For details of \ ``update``\  element,
        refer to "`MyBatis3 REFERENCE DOCUMENTATION (Mapper XML Files-insert, update and delete-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#insert_update_and_delete>`_\".

        The value to be bound in SET clause and WHERE clause is specified as the bind variable with #{variableName} format.
        In the above example, since a JavaBean (\ ``Todo``\ ) is specified as argument of Repository interface,
        JavaBean property name is specified in the bind variable name.

|

* Apply DI to Repository in Service class and call Repository interface method.

 .. code-block:: java


    package com.example.domain.service.todo;

    import javax.inject.Inject;

    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;

    import com.example.domain.model.Todo;
    import com.example.domain.repository.todo.TodoRepository;

    @Transactional
    @Service
    public class TodoServiceImpl implements TodoService {

        // (3)
        @Inject
        TodoRepository todoRepository;

        @Override
        public Todo update(Todo todo) {

            // (4)
            Todo currentTodo = todoRepository.findOne(todo.getTodoId());
            if (currentTodo == null || currentTodo.getVersion() != todo.getVersion()) {
                throw new ObjectOptimisticLockingFailureException(Todo.class, todo
                        .getTodoId());
            }

            // (5)
            currentTodo.setTodoTitle(todo.getTodoTitle());
            currentTodo.setFinished(todo.isFinished());

            // (6)
            boolean updated = todoRepository.update(currentTodo);
            // (7)
            if (!updated) {
                throw new ObjectOptimisticLockingFailureException(Todo.class,
                        currentTodo.getTodoId());
            }
            currentTodo.setVersion(todo.getVersion() + 1);

            return currentTodo;
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (3)
      - Apply DI to Repository interface in Service class.
    * - (4)
      - Fetch the Entity to be updated from database.
      
        In the above example, when Entity is updated (records are deleted or version is updated),
        optimistic locking exception (\ ``org.springframework.orm.ObjectOptimisticLockingFailureException``\ ) provided by Spring Framework is generated.
    * - (5)
      - Reflect update details for the Entity to be updated.

        In the above example, "Title" and "Complete flag" are reflected. When there are few update items, the process can be performed as per the implementation example given above.
        However, when update items are more in number, it is recommended to use ":doc:`../GeneralFuncDetail/Dozer`".
    * - (6)
      - Call the Repository interface method and update single Entity record.
    * - (7)
      - Determine update results of Entity.
      
        In the above example, when Entity is not updated (records are deleted or version is updated),
        optimistic locking exception (\ ``org.springframework.orm.ObjectOptimisticLockingFailureException``\) provided by Spring Framework is generated.


 .. tip::

    In the above example, when update process is successful,

     .. code-block:: java

        currentTodo.setVersion(todo.getVersion() + 1);

    is obtained.

    It is a process to combine the version updated in database and the version that stores an Entity.
    
    If database status and Entity status are not matched when referring a version in the call source (Controller or JSP etc.) process,
    data mismatch may occur and application is not executed as anticipated.




.. _DataAccessMyBatis3HowToUseUpdateMultiple:

Batch update of Entity
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Implementation example wherein Entity is updated in batch is given below.

The methods to update Entity in batch are as below.

* Execute UPDATE statement that updates multiple records simultaneously

* Use JDBC batch update functionality



Refer to ":ref:`DataAccessMyBatis3HowToExtendExecutorTypeBatch`" for the details on how to use the JDBC batch update functionality.

|

How to execute UPDATE statement that concurrently updates multiple records is explained here.

* Defining the method in Repository interface.

 .. code-block:: java

    package com.example.domain.repository.todo;

    import com.example.domain.model.Todo;
    import org.apache.ibatis.annotations.Param;

    import java.util.List;

    public interface TodoRepository {

        // (1)
        int updateFinishedByTodIds(@Param("finished") boolean finished,
                                   @Param("todoIds") List<String> todoIds);

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - In the above example, \ ``updateFinishedByTodIds``\  method is defined as the method to update \ ``finished``\  column of the records corresponding to list of IDs specified in the argument.
        

 .. note:: **Return value for the method that updates Entity in batch**

    Return value of the method that updates Entity in batch should preferably be of numeric type (\ ``int``\  or \ ``long``\ ).
    When it is set as numeric type return value, number of updated records can be fetched.

|

* Defining the SQL in mapping file.

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <update id="updateFinishedByTodIds">
            UPDATE
                t_todo
            SET
                finished = #{finished},
                /* (2) */
                version = version + 1
            WHERE
                /* (3) */
                <foreach item="todoId" collection="todoIds"
                         open="todo_id IN (" separator="," close=")">
                    #{todoId}
                </foreach>
        </update>

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 10 70

    * - Sr. No.
      - Attribute
      - Description
    * - (2)
      - \-
      - Updates version column when an optimistic locking is applied using a version column.

        If the version is not updated, optimistic locking control does not operate properly.
        Refer to ":doc:`ExclusionControl`" for details of exclusive control.
    * - (3)
      - \-
      - Specifies the update conditions for updating multiple records in WHERE clause.
    * -
      - \-
      - Repeats process for the list of IDs passed by argument, using \ ``foreach``\  element.

        In the above example, IN clause is generated from the list of IDs passed by argument.

        Refer "`MyBatis3 REFERENCE DOCUMENTATION (Dynamic SQL-foreach-) <http://mybatis.github.io/mybatis-3/dynamic-sql.html#foreach>`_\"  for details of \ ``foreach``\.
        
    * -
      - collection
      - Specify collection for a process.

        In the above example, the process is repeated for a list of IDs (\ ``todoIds``\) of Repository method arguments.
    * -
      - item
      - Specify local variable name that retains 1 element in the list.
    * -
      - separator
      - Specify the string for separating elements in the list.

        In the above example, \ ``","``\,  which is the separator character of IN clause, is specified.




.. _DataAccessMyBatis3HowToUseDelete:

Delete process for Entity
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _DataAccessMyBatis3HowToUseDeleteOne:


Deleting a single Entity
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Implementation example for deleting a single Entity is given below.

 .. note::

     Explanation hereafter shows an implementation example wherein an optimistic locking is performed by using version column.
     However, it is not necessary to perform the processes related to optimistic locking when optimistic locking is not required.

     Refer to ":doc:`ExclusionControl`" for details of exclusive control.

|

* Defining method in Repository interface.

 .. code-block:: java

    package com.example.domain.repository.todo;

    import com.example.domain.model.Todo;

    public interface TodoRepository {

        // (1)
        boolean delete(Todo todo);

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - In the above example, \ ``delete``\  method is defined as the method to delete single Todo object specified in the argument.
        


 .. note:: ** Return value for the method that deletes a single Entity**

    The return value of the method that deletes a single Entity can be \ ``boolean``\.

    However, when the return value is to be handled as a data mismatch error due to multiple deletion results,
    it is necessary to set numeric value type (\ ``int``\  or \ ``long``\ ) as the return value and to check whether a single deletion record exists.
    When main key is used as the delete condition, return value can be set to \ ``boolean``\  since, multiple deleted records are not obtained.

    * When \ ``boolean``\  is specified as return value, \ ``false``\ is returned when deleted records are 0 and \ ``true``\  is returned when deleted records are 1 or more.
    * When numeric value type is specified as a return value, the number of deleted records is returned.


|

* Defining a SQL in mapping file.


 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <!-- (2) -->
        <delete id="delete" parameterType="Todo">
            DELETE FROM
                t_todo
            WHERE
                todo_id = #{todoId}
            AND
                version = #{version}
        </delete>

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (2)
      - Implement DELETE SQL in \ ``delete``\  element.

        Specify method name for the method defined in Repository interface, in \ ``id``\  attribute.

        Refer to "`MyBatis3 REFERENCE DOCUMENTATION (Mapper XML Files-insert, update and delete-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#insert_update_and_delete>`_\" for details of \ ``delete``\  element.
        

        The value to be bound in WHERE clause is specified as a bind variable of #{variableName} format.
        In the above example, JavaBean property name is specified in the bind variable name since a JavaBean (\ ``Todo``\ ) is specified as argument of Repository interface.
        

|

* Apply DI to the Repository in Service class and call the method for Repository interface.


 .. code-block:: java

    package com.example.domain.service.todo;

    import javax.inject.Inject;

    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;

    import com.example.domain.model.Todo;
    import com.example.domain.repository.todo.TodoRepository;

    @Transactional
    @Service
    public class TodoServiceImpl implements TodoService {

        // (3)
        @Inject
        TodoRepository todoRepository;

        @Override
        public Todo delete(String todoId, long version) {

            // (4)
            Todo currentTodo = todoRepository.findOne(todoId);
            if (currentTodo == null || currentTodo.getVersion() != version) {
                throw new ObjectOptimisticLockingFailureException(Todo.class, todoId);
            }

            // (5)
            boolean deleted = todoRepository.delete(currentTodo);
            // (6)
            if (!deleted) {
                throw new ObjectOptimisticLockingFailureException(Todo.class,
                        currentTodo.getTodoId());
            }

            return currentTodo;
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (3)
      - Apply DI to Repository interface in Service class.
    * - (4)
      - Fetch the Entity to be deleted from database.

        In the above example, when Entity is updated (records are deleted or version is updated),
        optimistic locking exception (\ ``org.springframework.orm.ObjectOptimisticLockingFailureException``\ ) provided by Spring Framework is generated.
    * - (5)
      - Call Repository interface method and delete a single Entity.
    * - (6)
      - Determine the deletion result of Entity.

        In the above example, when Entity is not deleted (records are deleted or version is updated)
        optimistic locking exception (\ ``org.springframework.orm.ObjectOptimisticLockingFailureException``\ ) provided by Spring Framework is generated.

|

.. _DataAccessMyBatis3HowToUseDeleteMultiple:


Batch deletion of Entity
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""


Implementation example wherein Entity is deleted in batch is given below.

The methods to delete Entity in batch are as given below.

* Execute a DELETE statement that concurrently deletes multiple records

* Use JDBC batch update functionality



Refer to ":ref:`DataAccessMyBatis3HowToExtendExecutorTypeBatch`" for how to use JDBC batch update functionality.

|

The method to execute DELETE statement that concurrently deletes multiple records, is explained below.

* Defining method in Repository interface.

 .. code-block:: java

    package com.example.domain.repository.todo;

    public interface TodoRepository {

        // (1)
        int deleteOlderFinishedTodo(Date criteriaDate);

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - In the above example, \ ``deleteOlderFinishedTodo``\  method is defined as the method to delete finished records that were created prior to the standard date.
        

 .. note:: **Return value for the method that deletes Entity in batch**

    Return value for the method that deletes Entity in batch can be numeric value type (\ ``int``\ or \ ``long``\ ).
    When the return value is of numeric type, number of deleted records can be fetched.

|

* Defining SQL in mapping file.

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <delete id="deleteOlderFinishedTodo" parameterType="date">
            <![CDATA[
            DELETE FROM
                t_todo
            /* (2) */
            WHERE
                finished = TRUE
            AND
                created_at  < #{criteriaDate}
            ]]>
        </delete>

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (3)
      - Specify delete conditions for updating multiple records in WHERE clause.

        In the above example,

        * Finished (\ ``finished``\  is \ ``TRUE``\ )
        * Created before standard date (\ ``created_at``\  before standard date)

        are specified as delete conditions.

|

.. _DataAccessMyBatis3HowToUseDynamicSql:

Implementing dynamic SQL
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

An implementation example wherein dynamic SQL is built, is given below.

MyBatis3 provides a mechanism through which dynamic SQL is built by using OGNL base expression (Expression language) and XML elements for building dynamic SQL.


Refer to "MyBatis3 REFERENCE DOCUMENTATION (Dynamic SQL) <http://mybatis.github.io/mybatis-3/dynamic-sql.html>`_\" for details of dynamic SQL.


MyBatis3 provides following XML elements for building a dynamic SQL.


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 10 70

    * - Sr. No.
      - Element name
      - Description
    * - 1.
      - \ ``if``
      - Element that builds SQL only when it matches with the condition.
    * - 2.
      - \ ``choose``\
      - Element that builds SQL by selecting one of the options from multiple options, that matches with the condition.
    * - 3.
      - \ ``where``\
      - Element that assigns or removes prefix and suffix for the built WHERE clause.
    * - 4.
      - \ ``set``\
      - Element that assigns or removes prefix or suffix for the built SET clause.
    * - 5.
      - \ ``foreach``\
      - Element that repeats a process for a collection or an array
    * - 6.
      - \ ``bind``\
      - Element that stores the results of OGNL expression in the variable.

        Variable stored by using \ ``bind``\  variable can be referred in SQL.

 .. tip::

    Although it is not given in the list, \ ``trim``\  element is provided as the XML element for building dynamic SQL.

    \ ``trim``\  element is a more generalized XML element as compared to \ ``where``\  element and \ ``set``\  element.

    In most of the cases, \ ``where``\  element and \ ``set``\  element can meet the requirements. Hence, description of \ ``trim``\  element is omitted in this guideline.
    Refer to "`MyBatis3 REFERENCE DOCUMENTATION (Dynamic SQL-trim, where, set-) <http://mybatis.github.io/mybatis-3/dynamic-sql.html#trim_where_set>`_\" when it is necessary to use \ ``trim``\  element.
    
    

|

Implementation of if element
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\ ``if``\  element is the XML element that builds SQL only when it matches with specified conditions.

 .. code-block:: xml

    <select id="findAllByCriteria" parameterType="TodoCriteria" resultType="Todo">
        SELECT
            todo_id,
            todo_title,
            finished,
            created_at,
            version
        FROM
            t_todo
        WHERE
            todo_title LIKE #{todoTitle} || '%' ESCAPE '~'
        <!-- (1) -->
        <if test="finished != null">
            AND
                finished = #{finished}
        </if>
        ORDER BY
            todo_id
    </select>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Specify the condition in \ ``test``\  attribute of \ ``if``\  element.

        In the above example, when \ ``finished``\  is specified as the search condition, conditions for \ ``finished``\  column are added to SQL.

SQL (WHERE clause) generated by dynamic SQL described above consists of 2 patterns.

 .. code-block:: sql

    -- (1) finished == null
    ...
    WHERE
        todo_title LIKE ? || '%' ESCAPE '~'
    ORDER BY
        todo_id

 .. code-block:: sql

    -- (2) finished != null
    ...
    WHERE
        todo_title LIKE ? || '%' ESCAPE '~'
    AND
        finished = ?
    ORDER BY
        todo_id

|

Implementation of choose element
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``choose``\  element is the XML element for building SQL by selecting one option that matches the condition from a set of conditions.

 .. code-block:: xml

    <select id="findAllByCriteria" parameterType="TodoCriteria" resultType="Todo">
        SELECT
            todo_id,
            todo_title,
            finished,
            created_at,
            version
        FROM
            t_todo
        WHERE
            todo_title LIKE #{todoTitle} || '%' ESCAPE '~'
        <!-- (1) -->
        <choose>
            <!-- (2) -->
            <when test="createdAt != null">
                AND
                    created_at <![CDATA[ > ]]> #{createdAt}
            </when>
            <!-- (3) -->
            <otherwise>
                AND
                    created_at <![CDATA[ > ]]> CURRENT_DATE
            </otherwise>
        </choose>
        ORDER BY
            todo_id
    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Specify the condition to build SQL by specifying \ ``when``\  element and \ ``otherwise``\  element in \ ``choose``\  element.
    * - (2)
      - Specify the condition in \ ``test``\  attribute of \ ``when``\  element.

        In the above example, when \ ``createdAt``\  is specified as a search condition,
        a condition wherein, values of \ ``create_at``\ column extract the records after the specified date, is added to SQL.
    * - (3)
      - Specify all the SQLs in \``otherwise``\  element that are built when the conditions do not match with \ ``when``\  element \.

        In the above example, the condition wherein \ ``create_at``\  column values extract the records after the current date (records that are created on that day) is added to SQL.


SQL (WHERE clause) that is generated by dynamic SQL described above consists of 2 patterns.

 .. code-block:: sql

    -- (1) createdAt!=null
    ...
    WHERE
        todo_title LIKE ? || '%' ESCAPE '~'
    AND
        created_at   >   ?
    ORDER BY
        todo_id

 .. code-block:: sql

    -- (2) createdAt==null
    ...
    WHERE
        todo_title LIKE ? || '%' ESCAPE '~'
    AND
        created_at > CURRENT_DATE
    ORDER BY
        todo_id

|

Implementation of where element
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``where``\  element is the XML element for dynamically generating WHERE clause.

When \ ``where``\  element is used,

* Assigning WHERE clause
* Removal of AND clause and OR clause

are performed. Hence, WHERE clause can be built easily.

 .. code-block:: xml

    <select id="findAllByCriteria2" parameterType="TodoCriteria" resultType="Todo">
        SELECT
            todo_id,
            todo_title,
            finished,
            created_at,
            version
        FROM
            t_todo
        <!-- (1) -->
        <where>
            <!-- (2) -->
            <if test="finished != null">
                AND
                    finished = #{finished}
            </if>
            <!-- (3) -->
            <if test="createdAt != null">
                AND
                    created_at <![CDATA[ > ]]> #{createdAt}
            </if>
        </where>
        ORDER BY
            todo_id
    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Implement the dynamic SQL for building WHERE clause in \ ``where``\  element.

        According to the SQL built in \ ``where``\  element, processes such as assigning WHERE clause, removing AND clause and OR clause etc. can be performed.
    * - (2)
      - Build dynamic SQL.

        In the above example, when \ ``finished``\  is specified as a search condition,
        the condition for \ ``finished``\  column is added to SQL.
    * - (3)
      - Build dynamic SQL.

        In the above example, when \ ``createdAt``\  is specified as a search condition,
        the condition for \ ``created_at``\  column is added to SQL.


The SQL (WHERE clause) that is generated by dynamic SQL described above consists of 4 patterns as given below.

 .. code-block:: sql

    -- (1) finished != null && createdAt != null
    ...
    FROM
        t_todo
    WHERE
        finished = ?
    AND
        created_at  >  ?
    ORDER BY
        todo_id

 .. code-block:: sql

    -- (2) finished != null && createdAt == null
    ...
    FROM
        t_todo
    WHERE
        finished = ?
    ORDER BY
        todo_id

 .. code-block:: sql

    -- (3) finished == null && createdAt != null
    ...
    FROM
        t_todo
    WHERE
        created_at  >  ?
    ORDER BY
        todo_id

 .. code-block:: sql

    -- (4) finished == null && createdAt == null
    ...
    FROM
        t_todo
    ORDER BY
        todo_id

|

Implementation example for set element
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``set``\  element is the XML element for automatically generating SET clause.

When \ ``set``\  element is used,

* Assigning SET clause
* Removal of comma at the end

are performed. Hence SET clause can be easily built.

 .. code-block:: xml

    <update id="update" parameterType="Todo">
        UPDATE
            t_todo
        <!-- (1)  -->
        <set>
            version = version + 1,
            <!-- (2) -->
            <if test="todoTitle != null">
                todo_title = #{todoTitle}
            </if>
        </set>
        WHERE
            todo_id = #{todoId}
    </update>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Implement the dynamic SQL to build SET clause in \ ``set``\  element.

        Assigning SET clause and removal of comma at the end is performed according to the SQL that is built in \ ``set``\  element.
    * - (2)
      - Build a dynamic SQL.

        In the above example, when \ ``todoTitle``\  is specified as an update item,
        \ ``todo_title``\  column is added to SQL as an update column.

SQL generated by dynamic SQL described above consists of 2 patterns as below.

 .. code-block:: sql

    -- (1) todoTitle != null
    UPDATE
        t_todo
    SET
        version = version + 1,
        todo_title = ?
    WHERE
        todo_id = ?

 .. code-block:: sql

    -- (2) todoTitle == null
    UPDATE
        t_todo
    SET
       version = version + 1
    WHERE
        todo_id = ?

|

Implementation example of foreach element
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``foreach``\  element is the XML element for repeating a process for a collection or an array.

 .. code-block:: xml

    <select id="findAllByCreatedAtList" parameterType="list" resultType="Todo">
        SELECT
            todo_id,
            todo_title,
            finished,
            created_at,
            version
        FROM
            t_todo
        <where>
            <!-- (1) -->
            <if test="list != null">
                <!-- (2) -->
                <foreach collection="list" item="date" separator="OR">
                <![CDATA[
                    (created_at >= #{date} AND created_at < DATEADD('DAY', 1, #{date}))
                ]]>
                </foreach>
            </if>
        </where>
        ORDER BY
            todo_id
    </select>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.65\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 15 65

    * - Sr. No.
      - Attribute
      - Description
    * - (1)
      - \-
      - Performs \ ``null``\  check for the collection or array for which process is repeated.

        Null check is not necessary when \ ``null``\  value is not obtained.
    * - (2)
      - \-
      - Repeat the process for the collection or array and build the dynamic SQL, by using \ ``foreach``\  element.

        In the above example, WHERE clause is built for searching the record wherein date of record creation matches with any of the specified dates (date list).
    * -
      - collection
      - Specify the collection or array for which a process is repeated, in \ ``collection``\  attribute.

        In the above example, collection specified in the Repository method argument is specified.
        
    * -
      - item
      - Specify the local variable name that retains one element in the list, in \ ``item``\  attribute.

        In the above example, since the date list is specified in \ ``collection``\  attribute,
        variable name called \ ``date``\  is specified.
    * -
      - separator
      - Specify the separator string between elements in \ ``separator``\  attribute.

        In the above example, WHERE clause of OR condition is built.

 .. tip::

    \ ``foreach``\  element consists of following attributes although they are not used in the above example.

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.65\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 15 65

        * - Sr. No.
          - Attribute
          - Description
        * - (1)
          - open
          - Specify the string that is set before processing the elements that are at the beginning of the collection.
        * - (2)
          - close
          - Specify the string that is set after processing the elements at the end of the collection.
        * - (3)
          - index
          - Specify the variable name that stores the loop number.

    There are only a few cases that use \ ``index``\  attribute, however \ ``open``\  attribute and \ ``close``\  attribute are used to generate IN clause etc. dynamically.
    

    How to use \ ``foreach``\ element while creating an IN clause is explained.

     .. code-block:: xml

        <foreach collection="list" item="statusCode"
                open="AND order_status IN ("
                separator=","
                close=")">
            #{statusCode}
        </foreach>

    Following SQL is built.

     .. code-block:: sql

        -- list=['accepted','checking']
        ...
        AND order_status IN (?,?)


|  SQL (WHERE clause) generated by dynamic SQL described above consists of following 3 patterns.

 .. code-block:: sql

    -- (1) list=null or statusCodes=[]
    ...
    FROM
        t_todo
    ORDER BY
        todo_id

 .. code-block:: sql

    -- (2) list=['2014-01-01']
    ...
    FROM
        t_todo
    WHERE
        (created_at >= ? AND created_at < DATEADD('DAY', 1, ?))
    ORDER BY
        todo_id

 .. code-block:: sql

    -- (3) list=['2014-01-01','2014-01-02']
    ...
    FROM
        t_todo
    WHERE
        (created_at >= ? AND created_at < DATEADD('DAY', 1, ?))
    OR
        (created_at >= ? AND created_at < DATEADD('DAY', 1, ?))
    ORDER BY
        todo_id

|

Implementation example for bind element
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``bind``\  element is the XML element for storing OGNL expression result in the variable.

 .. code-block:: xml

    <select id="findAllByCriteria" parameterType="TodoCriteria" resultType="Todo">
        <!-- (1) -->
        <bind name="escapedTodoTitle"
              value="@org.terasoluna.gfw.common.query.QueryEscapeUtils@toLikeCondition(todoTitle)" />
        SELECT
            todo_id,
            todo_title,
            finished,
            created_at,
            version
        FROM
            t_todo
        WHERE
            /* (2) */
            todo_title LIKE #{escapedTodoTitle} || '%' ESCAPE '~'
        ORDER BY
            todo_id
    </select>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.65\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 15 65

    * - Sr. No.
      - Attribute
      - Description
    * - (1)
      - \-
      - Store results of OGNL expression in the variable, using \ ``bind``\  element.

        In the above example, the results obtained by calling the method using OGNL expression, are stored in the variable.
    * -
      - name
      - Specify variable name in \ ``name``\  attribute.

        The variable specified here can be used as SQL bind variable.
    * -
      - value
      - Specify OGNL expression in \ ``value``\  attribute.

        Results obtained by executing OGNL expression are stored in the variable specified by \ ``name``\  attribute.

        In the above example, the results obtained by calling method (\ ``QueryEscapeUtils#toLikeCondition(String)``\ ) provided by common library
        are stored in the variable \ ``escapedTodoTitle``\ .
    * - (2)
      - \-
      - Specify the variable created by using \ ``bind``\  element as the bind variable.

        In the above example, the variable created by using \ ``bind``\  element (\ ``escapedTodoTitle``\ ) is specified as the bind variable.

 .. tip::

    In the above example, although the variable created by using \ ``bind``\  variable is specified as the bind variable,
    it can also be used as substitution variable.

    Refer to ":ref:`DataAccessMyBatis3HowToUseSqlInjectionCountermeasure`" for bind variable and substitution variable.
    
|


.. _DataAccessMyBatis3HowToUseLikeEscape:

Escape during LIKE search
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When performing LIKE search, the value to be used as search condition should be escaped for LIKE search.

The escape process for LIKE search
can be implemented by using the method of \ ``org.terasoluna.gfw.common.query.QueryEscapeUtils`` \  class provided by common library.

Refer to ":ref:`data-access-common_appendix_like_escape`" for specifications of the escape process provided by common library.


 .. code-block:: xml

    <select id="findAllByCriteria" parameterType="TodoCriteria" resultType="Todo">
        <!-- (1) -->
        <bind name="todoTitleContainingCondition"
              value="@org.terasoluna.gfw.common.query.QueryEscapeUtils@toContainingCondition(todoTitle)" />
        SELECT
            todo_id,
            todo_title,
            finished,
            created_at,
            version
        FROM
            t_todo
        WHERE
            /* (2) (3) */
            todo_title LIKE #{todoTitleContainingCondition} ESCAPE '~'
        ORDER BY
            todo_id
    </select>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Call the Escape process method for LIKE search provided by common library, by using \ ``bind``\  element (OGNL expression).

        In the above example, escape process is performed for partial match and is stored in \ ``todoTitleContainingCondition``\  variable.
        \ ``QueryEscapeUtils@toContainingCondition(String)``\  is the method that assigns \ "``%``"\  before and after the escaped string.
    * - (2)
      - Specify the string that performs escape process for partial match, as bind variable of LIKE clause.
    * - (3)
      - Specify escape character in ESCAPE clause.

        Since \ ``"~"``\  is used as an escape character in the Escape process provided by common library,
        \ ``'~'``\ is specified in ESCAPE clause.

 .. tip::

     In the above example, a method that performs the Escape process for partial match is called. However, methods that perform the following processes are also provided.

     * Escape for starting-with match (\ ``QueryEscapeUtils@toStartingWithCondition(String)``\)
     * Escape for ends-with match (\ ``QueryEscapeUtils@toEndingWithCondition(String)``\)
     * Escape only (\ ``QueryEscapeUtils@toLikeCondition(String)``\)

     

     Refer to ":ref:`data-access-common_appendix_like_escape`" for details.

 .. note::

     In the above example, the method that performs Escape process in the mapping file is called, however
     Escape process can also be called as a Service process before calling the Repository method.

     As role of component, it is appropriate that Escape process is performed in mapping file.
     Hence, in this guideline, it is recommended to perform Escape process in mapping file.

|

.. _DataAccessMyBatis3HowToUseSqlInjectionCountermeasure:

SQL Injection countermeasures
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It is important to take precautions when building SQL to avoid occurrence of SQL Injection.

MyBatis3 provides following two methods as the mechanism to embed values in SQL.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
     :header-rows: 1
     :widths: 10 20 60

     * - Sr. No.
       - Method
       - Description
     * - (1)
       -  Embedding value by using bind variable
       - When this method is used, the value is embedded after building SQL by using \ ``java.sql.PreparedStatement`` \.
         Hence, **the value can be safely embedded.**

         **When the value entered by user is to be embedded in SQL, as a rule, bind variable should be used.**
     * - (2)
       - Embedding value by using substitution variable
       - When this method is used, the value is substituted as a string while building SQL.
         Hence **Safe embedding of value cannot be guaranteed.**

 .. warning::

    When the value entered by the user is embedded by using substitution variable,
    it should be noted that the risk of SQL Injection is high.

    When the value entered by the user needs to be embedded by using a substitution variable,
    the input check must be performed in order to ensure that SQL Injection has not occurred.

    Basically, **it is strongly recommended not to use the value entered by the user as it is.**



How to embed using a bind variable
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

How to use a bind variable is shown below.

 .. code-block:: xml

    <insert id="create" parameterType="Todo">
        INSERT INTO
            t_todo
        (
            todo_id,
            todo_title,
            finished,
            created_at,
            version
        )
        VALUES
        (
            /* (1) */
            #{todoId},
            #{todoTitle},
            #{finished},
            #{createdAt},
            #{version}
        )
    </insert>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :widths: 10 80
    :header-rows: 1

    * - Sr. No.
      - Description
    * - (1)
      - Enclose the property name of the property that stores bind value using  \ ``#{`` \  and \ ``}`` \  and specify it as bind variable.
        

 .. tip::

     A number of attributes can be specified in the bind variable.

     The attributes that can be specified are as below.

     * javaType
     * jdbcType
     * typeHandler
     * numericScale
     * mode
     * resultMap
     * jdbcTypeName

     

     Basically, MyBatis simply selects an appropriate behavior just by specifying the property name.
     The attributes described above can be specified when MyBatis does not select an appropriate behavior.

     Refer to "\ `MyBatis3 REFERENCE DOCUMENTATION(Mapper XML Files-Parameters-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#Parameters>`_ " for how to use attributes.
     



|

How to embed using a substitution variable
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

How to use a substitution variable is described below.


* Defining the method in Repository interface.

 .. code-block:: java

    public interface TodoRepository {
        List<Todo> findAllByCriteria(@Param("criteria") TodoCriteria criteria,
                                     @Param("direction") String direction);
    }

* Implementing SQL in mapping file.

 .. code-block:: xml

    <select id="findAllByCriteria" parameterType="TodoCriteria" resultType="Todo">
        <bind name="todoTitleContainingCondition"
              value="@org.terasoluna.gfw.common.query.QueryEscapeUtils@toContainingCondition(criteria.todoTitle)" />
        SELECT
            todo_id,
            todo_title,
            finished,
            created_at,
            version
        FROM
            t_todo
        WHERE
            todo_title LIKE #{todoTitleContainingCondition} ESCAPE '~'
        ORDER BY
            /* (1) */
            todo_id ${direction}
    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :widths: 10 80
    :header-rows: 1

    * - Sr. No.
      - Description
    * - (1)
      - Enclose the property name of the property that stores the value to be substituted by \ ``${`` \  and \ ``}`` \
        and specify as substitution variable. In the above example, \ ``${direction}`` \  part is substituted
        by \ ``"DESC"`` \  or \ ``"ASC"`` \ .

 .. warning::

     **Embedding value by a substitution variable must be used only after ensuring that the value is safe for the application**
     **and by restricting its use to table name, column name and sort conditions.**

    For example, the pair of code value and value to be embedded in SQL is stored in \ ``Map``\  as shown below.

      .. code-block:: java

        Map<String, String> directionMap = new HashMap<String, String>();
        directionMap.put("1", "ASC");
        directionMap.put("2", "DESC");

    The value entered should be handled as code value and is expected to be converted to a safe value inside the process executing the SQL.

      .. code-block:: java

        String direction = directionMap.get(directionCode);
        todoRepository.findAllByCriteria(criteria, direction);

    In the above example, \ ``Map``\  is used.
    However, "\ :doc:`../WebApplicationDetail/Codelist` \"  provided by common library can also be used.
    If "\ :doc:`../WebApplicationDetail/Codelist` \" is used, the value entered can be checked.
    Hence, the value can be safely embedded.

    - :file:`projectName-domain/src/main/resources/META-INF/spring/projectName-codelist.xml`

      .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd">

            <bean id="CL_DIRECTION" class="org.terasoluna.gfw.common.codelist.SimpleMapCodeList">
                <property name="map">
                    <map>
                        <entry key="1" value="ASC" />
                        <entry key="2" value="DESC" />
                    </map>
                </property>
            </bean>
        </beans>

    - Service class

      .. code-block:: java

        @Inject
        @Named("CL_DIRECTION")
        CodeList directionCodeList;

        // ...

        public List<Todo> searchTodos(TodoCriteria criteria, String directionCode){
            String direction = directionCodeList.asMap().get(directionCode);
            List<Todo> todos = todoRepository.findAllByCriteria(criteria, direction);
            return todos;
        }

|

.. _DataAccessMyBatis3HowToExtend:

How to extend
--------------------------------------------------------------------------------


.. _DataAccessMyBatis3HowToExtendSqlShare:

Sharing SQL statement
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

How to share a SQL statement in multiple SQLs is explained.

In MyBatis3, SQL statement (or a part of SQL statement) can be shared by using \ ``sql``\  element and \ ``include``\  element.


 .. note:: **How to use a shared SQL statement**

    When pagination search is to be performed, WHERE clause of "SQL that fetches total records of Entity matching with search conditions" and 
    "SQL that fetches a list of Entities matching with search conditions" should be shared.

|

Implementation example of mapping file is as given below.

 .. code-block:: xml
    :emphasize-lines: 1-2, 16-17, 29-30

    <!-- (1)  -->
    <sql id="findPageByCriteriaWherePhrase">
        <![CDATA[
        WHERE
            todo_title LIKE #{title} || '%' ESCAPE '~'
        AND
            created_at < #{createdAt}
        ]]>
    </sql>

    <select id="countByCriteria" resultType="_long">
        SELECT
            COUNT(*)
        FROM
            t_todo
        <!-- (2)  -->
        <include refid="findPageByCriteriaWherePhrase"/>
    </select>

    <select id="findPageByCriteria" resultType="Todo">
        SELECT
            todo_id,
            todo_title,
            finished,
            created_at,
            version
        FROM
            t_todo
        <!-- (2)  -->
        <include refid="findPageByCriteriaWherePhrase"/>
        ORDER BY
            todo_id
    </select>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :widths: 10 80
    :header-rows: 1

    * - Sr. No.
      - Description
    * - (1)
      - Implement SQL statement to be shared by multiple SQLs, in \ ``sql``\  element.

        Specify an ID unique to the mapping file, in \ ``id``\  attribute.
    * - (2)
      - Specify the INCLUDE SQL by using \ ``include``\  element.

        Specify the INCLUDE SQL ID (value specified in \ ``id``\  attribute of \ ``sql``\  element), in \ ``refid``\  attribute.


|

.. _DataAccessMyBatis3HowToExtendTypeHandler:


Implementation of TypeHandler
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When it is necessary to perform mapping with the Java class not supported by MyBatis3 standard
and when it is necessary to change the standard behavior of MyBatis3, a unique \ ``TypeHandler`` \  should be created.

How to implement the \ ``TypeHandler`` \  is explained using the examples given below.

* :ref:`DataAccessMyBatis3HowToExtendTypeHandlerBlob`
* :ref:`DataAccessMyBatis3HowToExtendTypeHandlerClob`
* :ref:`DataAccessMyBatis3HowToExtendTypeHandlerJoda`



Refer to ":ref:`DataAccessMyBatis3HowToUseSettingsTypeHandler`" for how to apply a created \ ``TypeHandler`` \  in an application.


 .. note:: **Preconditions for implementation of BLOB and CLOB**

    A method added from JDBC 4.0 is used for the implementation of BLOB and CLOB.

    When using a JDBC driver that is not compatible with JDBC 4.0 or a 3rd party wrapper class,
    it must be noted that the operation may not work in the implementation example explained below.
    When the operation is to be performed in an environment wherein the driver is not compatible with JDBC 4.0,
    the implementation must be changed to suit the compatible version of JDBC driver to be used.

    For example, a lot of methods added by JDBC 4.0 are not implemented in JDBC driver for PostgreSQL9.3 (\ ``postgresql-9.3-1102-jdbc41.jar``\ ).
    

|

.. _DataAccessMyBatis3HowToExtendTypeHandlerBlob:

Implementing the TypeHandler for BLOB
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

MyBatis3 provides a \ ``TypeHandler`` \  for mapping BLOB in \ ``byte[]``\ .
However, when the data to be handled is very large, it is necessary to map in \ ``java.io.InputStream``\ .

How to implement a \ ``TypeHandler`` \  for mapping BLOB in \ ``java.io.InputStream``\  is given below.

 .. code-block:: java

    package com.example.infra.mybatis.typehandler;

    import org.apache.ibatis.type.BaseTypeHandler;
    import org.apache.ibatis.type.JdbcType;
    import org.apache.ibatis.type.MappedTypes;

    import java.io.InputStream;
    import java.sql.*;

    // (1)
    public class BlobInputStreamTypeHandler extends BaseTypeHandler<InputStream> {

        // (2)
        @Override
        public void setNonNullParameter(PreparedStatement ps, int i, InputStream parameter,
                                        JdbcType jdbcType) throws SQLException {
            ps.setBlob(i, parameter);
        }

        // (3)
        @Override
        public InputStream getNullableResult(ResultSet rs, String columnName)
                throws SQLException {
            return toInputStream(rs.getBlob(columnName));
        }

        // (3)
        @Override
        public InputStream getNullableResult(ResultSet rs, int columnIndex)
                throws SQLException {
            return toInputStream(rs.getBlob(columnIndex));
        }

        // (3)
        @Override
        public InputStream getNullableResult(CallableStatement cs, int columnIndex)
                throws SQLException {
            return toInputStream(cs.getBlob(columnIndex));
        }

        private InputStream toInputStream(Blob blob) throws SQLException {
            // (4)
            if (blob == null) {
                return null;
            } else {
                return blob.getBinaryStream();
            }
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Specify \ ``BaseTypeHandler``\  provided by MyBatis3 in parent class.

        In such cases, specify \ ``InputStream``\  in the generic type of \ ``BaseTypeHandler``\.
    * - (2)
      - Implement the process that configures \ ``InputStream``\  in \ ``PreparedStatement``\.
    * - (3)
      - Fetch \ ``InputStream``\  from \ ``Blob``\  that is fetched from \ ``ResultSet``\  or \ ``CallableStatement``\  and return as a return value.
    * - (4)
      - Since the fetched \ ``Blob``\  can become \ ``null``\  in case of the column which allows \ ``null``\ , \ ``InputStream``\  must be fetched only after performing \ ``null``\  check.
        

        In the implementation example described above, a private method is created since same process is required for all three methods.

|

.. _DataAccessMyBatis3HowToExtendTypeHandlerClob:

Implementing the TypeHandler for CLOB
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

MyBatis3 provides a \ ``TypeHandler`` \  for mapping CLOB in \ ``java.lang.String``\.
However, when the data to be handled is very large, it is necessary to map it in \ ``java.io.Reader``\.

How to implement the \ ``TypeHandler`` \  for mapping CLOB in \ ``java.io.Reader``\ is given below.

 .. code-block:: java

    package com.example.infra.mybatis.typehandler;

    import org.apache.ibatis.type.BaseTypeHandler;
    import org.apache.ibatis.type.JdbcType;

    import java.io.Reader;
    import java.sql.*;

    // (1)
    public class ClobReaderTypeHandler extends BaseTypeHandler<Reader> {

        // (2)
        @Override
        public void setNonNullParameter(PreparedStatement ps, int i, Reader parameter,
                                        JdbcType jdbcType) throws SQLException {
            ps.setClob(i, parameter);
        }

        // (3)
        @Override
        public Reader getNullableResult(ResultSet rs, String columnName)
            throws SQLException {
            return toReader(rs.getClob(columnName));
        }

        // (3)
        @Override
        public Reader getNullableResult(ResultSet rs, int columnIndex)
            throws SQLException {
            return toReader(rs.getClob(columnIndex));
        }

        // (3)
        @Override
        public Reader getNullableResult(CallableStatement cs, int columnIndex)
            throws SQLException {
            return toReader(cs.getClob(columnIndex));
        }

        private Reader toReader(Clob clob) throws SQLException {
            // (4)
            if (clob == null) {
                return null;
            } else {
                return clob.getCharacterStream();
            }
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Specify \ ``BaseTypeHandler``\  provided by MyBatis3 in parent class.

        In such cases, specify \ ``Reader``\  in generic type of \ ``BaseTypeHandler``\.
    * - (2)
      - Implement a process that sets \ ``Reader``\  in \ ``PreparedStatement``\.
    * - (3)
      - Fetch \ ``Reader``\  from \ ``Clob``\  that is fetched from \ ``ResultSet``\  or \ ``CallableStatement``\  and return it as the return value.
    * - (4)
      - Since fetched \ ``Clob``\  can become \ ``null``\  in the column that allows \ ``null``\ , \ ``Reader``\  needs to be fetched only after performing \ ``null``\  check.
        

        In the implementation example described above, a private method is created since same process is required for all three methods.

|

.. _DataAccessMyBatis3HowToExtendTypeHandlerJoda:

Implementing TypeHandler for Joda-Time
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| MyBatis3 does not support Joda-time classes (\ ``org.joda.time.DateTime``\ , ``org.joda.time.LocalDateTime``\ , \ ``org.joda.time.LocalDate``\  etc.).
| Hence, when Joda-Time class is used in the field of Entity class, it is necessary to provide a \ ``TypeHandler`` \  for Joda-Time.

How to implement a \ ``TypeHandler`` \  for mapping ``org.joda.time.DateTime``\  and \ ``java.sql.Timestamp``\  is shown below.

 .. note::

    Other classes provided by Joda-Time (\ ``LocalDateTime``\ , \ ``LocalDate``\ , \ ``LocalTime``\  etc.) can also be implemented in the same way.


 .. code-block:: java

    package com.example.infra.mybatis.typehandler;

    import java.sql.CallableStatement;
    import java.sql.PreparedStatement;
    import java.sql.ResultSet;
    import java.sql.SQLException;
    import java.sql.Timestamp;

    import org.apache.ibatis.type.BaseTypeHandler;
    import org.apache.ibatis.type.JdbcType;
    import org.joda.time.DateTime;

    // (1)
    public class DateTimeTypeHandler extends BaseTypeHandler<DateTime> {

        // (2)
        @Override
        public void setNonNullParameter(PreparedStatement ps, int i,
                DateTime parameter, JdbcType jdbcType) throws SQLException {
            ps.setTimestamp(i, new Timestamp(parameter.getMillis()));
        }

        // (3)
        @Override
        public DateTime getNullableResult(ResultSet rs, String columnName)
                throws SQLException {
            return toDateTime(rs.getTimestamp(columnName));
        }

        // (3)
        @Override
        public DateTime getNullableResult(ResultSet rs, int columnIndex)
                throws SQLException {
            return toDateTime(rs.getTimestamp(columnIndex));
        }

        // (3)
        @Override
        public DateTime getNullableResult(CallableStatement cs, int columnIndex)
                throws SQLException {
            return toDateTime(cs.getTimestamp(columnIndex));
        }

        private DateTime toDateTime(Timestamp timestamp) {
            // (4)
            if (timestamp == null) {
                return null;
            } else {
                return new DateTime(timestamp.getTime());
            }
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Specify the \ ``BaseTypeHandler``\  provided by MyBatis3 in parent class.

        In such cases, specify \ ``DateTime``\  in the generic type of \ ``BaseTypeHandler``\.
    * - (2)
      - Convert \ ``DateTime``\  to \ ``Timestamp``\  and implement the process configured in \``PreparedStatement``\ .
    * - (3)
      - Convert \ ``Timestamp``\  fetched from \ ``ResultSet``\  or \ ``CallableStatement``\  to \ ``DateTime``\  and return as a return value.
    * - (4)
      - Since \ ``Timestamp``\ can become \ ``null``\  in the column that allows \ ``null``\ , it needs to be converted to \ ``DateTime``\  only after performing \ ``null``\ check.
        

        In the implementation example described above, a private method is created since same process is required for all three methods.


|

.. _DataAccessMyBatis3HowToExtendResultHandler:

Implementation of ResultHandler
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

MyBatis3 provides a mechanism wherein search results are processed for each record.

When this mechanism is used and processes like,

* Value fetched by DB is processed in Java process
* Values etc. fetched by DB are aggregated as Java process

are performed, the amount of memory consumed simultaneously can be restricted to a minimum.

For example, when the process is to be implemented wherein, search results are downloaded as data in CSV format,
it is advisable to process the search results per record.

 .. note::

    **It is strongly recommended to use this mechanism when the quantity of search results may be very large and when it is necessary to**
    **process the search result for each record at a time in Java process.**

    When this mechanism of processing the search result for each record is not used,
    all the search result data, "Size of one data record * number of search result records", is  stored in the memory at the same time,
    and the data cannot be marked for GC till the process is completed for entire data.

    In contrast, when a mechanism wherein the search results are processed one at a time, is used,
    only the "size of one data record" is stored in the memory and that one data record can be marked for GC once the process for that data record is completed.
    

    For example, when "size of one data record" is \ ``2KB``\  and "number of search results" are \ ``10,000``\  records, the concurrent memory consumption is as below.

    * \ ``20MB``\  memory is consumed while performing the process collectively
    * \ ``2KB``\  memory is consumed while performing the process per record

    No particular issues have been observed in case of an application operated by a single thread.
    However, problems may occur in case of an application like Web application that is operated by multiple threads.

    If the process is performed for 100 threads at the same time, the concurrent memory consumption is as below.

    * \ ``2GB``\  memory is consumed while performing the process collectively
    * \ ``200KB``\  memory is consumed while performing the process per record

    

    The results are as below.

    * When the process is performed collectively, depending on the maximum heap size specified, system failure due to memory exhaustion and performance degradation due to frequent occurrence of full GC are more likely to occur.
    * When the process is performed per record, memory exhaustion or high-cost GC process can be controlled.

    Please note that the numbers used above are just the guidelines and not the actual measured values.

|

How to implement a process wherein the search results are downloaded as CSV data is given below.

* Defining the method in Repository interface.

 .. code-block:: java

    public interface TodoRepository {

        // (1) (2)
        void collectAllByCriteria(TodoCriteria criteria, ResultHandler<Todo> resultHandler);

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Specify \ ``org.apache.ibatis.session.ResultHandler``\  as an argument of the method.
    * - (2)
      - Specify \ ``void``\  type as the return value of the method.

        Precaution should be taken as, \ ``ResultHandler``\  is not called when a type other than \ ``void``\  is specified.

|

* Defining SQL in mapping file.

 .. code-block:: xml

    <!-- (3) -->
    <select id="collectAllByCriteria" parameterType="TodoCriteria" resultType="Todo">
        SELECT
            todo_id,
            todo_title,
            finished,
            created_at,
            version
        FROM
            t_todo
        <where>
            <if test="title != null">
                <bind name="titleContainingCondition"
                      value="@org.terasoluna.gfw.common.query.QueryEscapeUtils@toContainingCondition(title)" />
                todo_title LIKE #{titleContainingCondition} ESCAPE '~'
            </if>
            <if test="createdAt != null">
                <![CDATA[
                AND created_at < #{createdAt}
                ]]>
            </if>
        </where>
    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (3)
      - Mapping file is implemented in the same way as the normal search process.

 .. warning:: **Specifying fetchSize attribute**

    When a query to return a large amount of data is to be described, an appropriate value should be set in \ ``fetchSize``\  attribute.
    \ ``fetchSize``\  is a parameter which specifies data record count to be fetched in a single communication between JDBC driver and database.
    Note that, "default \ ``fetchSize``\ " can be specified in MyBatis configuration file, in MyBatis3.3.0 and subsequent versions which are supported in terasoluna-gfw-mybatis3 5.2.x.RELEASE.
    
    Refer ":ref:`DataAccessMyBatis3HowToUseSettingsDefaultFetchSize`" for details of \ ``fetchSize``\ .
    


|

* Apply DI to Repository in Service class and call Repository interface method.

 .. code-block:: java

    public class TodoServiceImpl implements TodoService {

        private static final DateTimeFormatter DATE_FORMATTER =
            DateTimeFormat.forPattern("yyyy/MM/dd");

        @Inject
        TodoRepository todoRepository;

        public void downloadTodos(TodoCriteria criteria,
            final BufferedWriter downloadWriter) {

            // (4)
            ResultHandler<Todo> handler = new ResultHandler<Todo>() {
                @Override
                public void handleResult(ResultContext<? extends Todo> context) {
                    Todo todo = context.getResultObject();
                    StringBuilder sb = new StringBuilder();
                    try {
                        sb.append(todo.getTodoId());
                        sb.append(",");
                        sb.append(todo.getTodoTitle());
                        sb.append(",");
                        sb.append(todo.isFinished());
                        sb.append(",");
                        sb.append(DATE_FORMATTER.print(todo.getCreatedAt().getTime()));
                        downloadWriter.write(sb.toString());
                        downloadWriter.newLine();
                    } catch (IOException e) {
                        throw new SystemException("e.xx.fw.9001", e);
                    }
                }
            };

            // (5)
            todoRepository.collectAllByCriteria(criteria, handler);

        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (4)
      - Generate \ ``ResultHandler``\  instance.

        Implement the process that is performed for each record in \ ``handleResult``\  method of \ ``ResultHandler``\.

        In the above example, \ ``ResultHandler``\  implementation class is not created and \ ``ResultHandler``\  is implemented as an anonymous object.
        Implementation class can also be created, however when it is not required to be shared by multiple processes, it need not be created.
    * - (5)
      - Call the method of Repository interface.

        When calling the method, specify ``ResultHandler``\  instance generated in (4), in the argument.

        When \ ``ResultHandler``\ is used, MyBatis repeats the following processes for the number of search results.

        * Records are fetched from search results and are mapped to JavaBean.
        * \ ``handleResult(ResultContext)``\  method of \ ``ResultHandler``\  instance is called.

 .. warning:: **Precautions while using ResultHandler**

    When \ ``ResultHandler``\  is used, following two points should be considered.

    * MyBatis3 provides a mechanism wherein the search results are stored in local cache and global binary cache to improve the efficiency of search process. However, the data returned from the method which uses \ ``ResultHandler``\  as an argument, is not cached.
    * When \ ``ResultHandler``\  is used for the statement that maps data of multiple rows in a single Java object by using manual mapping, an object in the incomplete state (the status of related Entity object prior to mapping) can be passed.

 .. tip:: **ResultContext method**

    Following method is provided in \ ``ResultContext``\  which is an argument of \ ``ResultHandler#handleResult``\  method.

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.65\linewidth}|
     .. list-table::
         :header-rows: 1
         :widths: 10 15 65

         * - Sr. No.
           - Method
           - Description
         * - (1)
           - getResultObject
           - A method for fetching object which mapped search results.
         * - (2)
           - getResultCount
           - A method for fetching the call count of \ ``ResultHandler#handleResult``\  method.
         * - (3)
           - stop
           - A method to notify MyBatis to stop the processing for the subsequent records.
             It is advisable to use this method when all the subsequent records are to be deleted.

    A method \ ``isStopped``\  is also provided in \ ``ResultContext``\  .However, its description is omitted since it is used by MyBatis.

|

.. _DataAccessMyBatis3HowToExtendExecutorType:

Using SQL execution mode
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Following three types of modes are provided in MyBatis3 to execute SQL. Default is \ ``SIMPLE``\ .

Here,

* How to use execution mode
* Precautions while using Repository of batch mode

| are explained.
| Refer to ":ref:`DataAccessMyBatis3HowToUseSettingsExecutorType`" for the explanation of execution mode.

.. _DataAccessMyBatis3HowToExtendExecutorTypeReuse:

Using PreparedStatement reuse mode
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When the execution mode is changed from \ ``SIMPLE``\  to \``REUSE``\  mode, the handling of \ ``PreparedStatement``\  in MyBatis changes.
However, there is no change in behavior (use method) of MyBatis.

How to change the execution mode from default (\ ``SIMPLE``\ ) to \ ``REUSE``\  is shown below.

* Settings are added to :file:`projectName-domain/src/main/resources/META-INF/mybatis/mybatis-config.xml`.

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
            PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
        <settings>
            <!-- (1) -->
            <setting name="defaultExecutorType" value="REUSE"/>
        </settings>
    </configuration>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Change \ ``defaultExecutorType``\  to \ ``REUSE``\ .

        When the settings given above are performed, default behavior changes to PreparedStatement reuse mode.

|

.. _DataAccessMyBatis3HowToExtendExecutorTypeBatch:

Using batch mode
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When all the Update system methods of Mapper interface are to be called in a batch mode,
execution mode can be changed to \ ``BATCH``\  mode using the method same as ":ref:`DataAccessMyBatis3HowToExtendExecutorTypeReuse`"


However, since batch mode has a number of constraints,
in actual application development, it is presumed to be used in combination with \ ``SIMPLE``\  or \ ``REUSE``\  mode.

For example,

* Batch mode is used in the process wherein the highest priority is to meet performance requirements associated with update of large amount of data.
* \ ``SIMPLE``\  or  \ ``REUSE``\  mode is used for a process wherein it is necessary to determine the update results to maintain data consistency such as in optimistic locking control etc.



 .. warning:: ** Precautions while using the execution mode in combination**

    When multiple execution modes are to be used in the application,
    **it should be noted that the execution mode cannot be changed within the same transaction.**

    For example, if multiple execution modes are used within the same transaction, MyBatis detects inconsistency and throws an error.

    This signifies that the processes below cannot be performed within the same transaction

    * Calling XxxRepository method in \ ``BATCH``\  mode
    * Calling YyyRepository method in \ ``REUSE``\  mode.

    

    Service or Repository acts as the transaction boundary for the application that is created based on these guidelines.
    
    Hence, **when multiple execution modes are to be used in the application,**
    **it is necessary to identify the execution mode while designing a Service or a Repository.**

    Transaction can be separated by specifying \ ``@Transactional(propagation = Propagation.REQUIRES_NEW)``\
    as method annotation for Service or Repository.
    Refer to ":ref:`service_transaction_management`" for details of transaction control.

|

Hereafter,

* How to configure for using multiple execution modes in combination
* How to implement an application

are explained.

|

.. _DataAccessMyBatis3HowToExtendExecutorTypeBatchIndividualSetting:

Settings for creating an individual batch mode Repository
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

When a batch mode Repository is to be created for a specific Repository,
a Bean can be defined for the Repository by using \ ``org.mybatis.spring.mapper.MapperFactoryBean``\  provided by MyBatis-Spring.


A Bean is registered for

* A repository of \ ``REUSE``\  mode as a Repository for normal use
* A Repository of \ ``BATCH``\  mode for a specific Repository

in the configuration example below.

- Bean definition is added to - :file:`projectName-domain/src/main/resources/META-INF/spring/projectName-infra.xml`.

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:context="http://www.springframework.org/schema/context"
           xmlns:mybatis="http://mybatis.org/schema/mybatis-spring"
           xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd
           http://mybatis.org/schema/mybatis-spring
           http://mybatis.org/schema/mybatis-spring.xsd">

        <bean id="sqlSessionFactory"
              class="org.mybatis.spring.SqlSessionFactoryBean">
            <property name="dataSource" ref="dataSource"/>
            <property name="configLocation"
                      value="classpath:META-INF/mybatis/mybatis-config.xml"/>
        </bean>

        <!-- (1) -->
        <bean id="sqlSessionTemplate"
              class="org.mybatis.spring.SqlSessionTemplate">
            <constructor-arg index="0" ref="sqlSessionFactory"/>
            <constructor-arg index="1" value="REUSE"/>
        </bean>

        <mybatis:scan base-package="com.example.domain.repository"
                      template-ref="sqlSessionTemplate"/> <!-- (2) -->

        <!-- (3) -->
        <bean id="batchSqlSessionTemplate"
              class="org.mybatis.spring.SqlSessionTemplate">
            <constructor-arg index="0" ref="sqlSessionFactory"/>
            <constructor-arg index="1" value="BATCH"/>
        </bean>

        <!-- (4) -->
        <bean id="todoBatchRepository"
              class="org.mybatis.spring.mapper.MapperFactoryBean">
            <!-- (5) -->
            <property name="mapperInterface"
                      value="com.example.domain.repository.todo.TodoRepository"/>
            <!-- (6) -->
            <property name="sqlSessionTemplate" ref="batchSqlSessionTemplate"/>
        </bean>

    </beans>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Define the Bean for \ ``SqlSessionTemplate``\  to be used in the Repository for normal use.
    * - (2)
      - Scan the Repository for normal use and register a Bean.

        Specify \ ``SqlSessionTemplate``\  defined in (1), in \ ``template-ref``\  attribute.
    * - (3)
      - Define a Bean for \ ``SqlSessionTemplate``\  in order to use in the Repository of batch mode.
    * - (4)
      - Define a Bean for the Repository in batch mode.

        Specify a value that does not overlap with the Bean name of Repository scanned in (2), in \ ``id``\  attribute.
        The Bean name of Repository scanned in (2) is a value for which the interface name is changed to "lowerCamelCase".

        In the above example, \ ``TodoRepository``\  for batch mode is registered in a Bean called \ ``todoBatchRepository``\.
    * - (5)
      - Specify an interface name (FQCN) for Repository that uses a batch mode, in \ ``mapperInterface``\  property.
        
    * - (6)
      - Specify \ ``SqlSessionTemplate``\  for batch mode that is defined in (3), in \ ``sqlSessionTemplate``\  property.
        

 .. note::

    If a Bean is defined for ``SqlSessionTemplate``\ , following WARN log is output when the application is terminated.

    This is because \ ``close``\  method is called while terminating ApplicationContext of Spring since \``java.io.Closeable``\  is inherited by \ ``SqlSession``\  interface.
    

     .. code-block:: text

        21:12:35.999 [Thread-2] WARN  o.s.b.f.s.DisposableBeanAdapter - Invocation of destroy method 'close' failed on bean with name 'sqlSessionTemplate'
        java.lang.UnsupportedOperationException: Manual close is not allowed over a Spring managed SqlSession
            at org.mybatis.spring.SqlSessionTemplate.close(SqlSessionTemplate.java:310) ~[mybatis-spring-1.2.2.jar:1.2.2]
            at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_20]
            at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_20]
            at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_20]
            at java.lang.reflect.Method.invoke(Method.java:483) ~[na:1.8.0_20]

    If there are no specific issues related to system operation, this measure is not required since it does not affect the application behavior even after the log is output.

    However, in case of any system operation issues like log monitoring etc,
    log output can be controlled by specifying the method (\ ``destroy-method``\  attribute) that is called while terminating Spring ApplicationContext.
    

    In the example below, it is specified such that \ ``getExecutorType``\  method is called.
    \ ``getExecutorType``\  is the method which is only used for returning the execution mode specified in constructor argument
    and has no impact on other operations.

     .. code-block:: xml
        :emphasize-lines: 3

        <bean id="batchSqlSessionTemplate"
              class="org.mybatis.spring.SqlSessionTemplate"
              destroy-method="getExecutorType">
            <constructor-arg index="0" ref="sqlSessionFactory"/>
            <constructor-arg index="1" value="BATCH"/>
        </bean>

|

.. _DataAccessMyBatis3HowToExtendExecutorTypeBatchBulkSetting:

Settings for creating Batch mode Repository in batch
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

When a batch mode Repository is to be created in a batch, a Bean can be defined for the Repository
by using the scan function (\ ``mybatis:scan``\  element) provided by MyBatis-Spring.


In the configuration example below, Bean is registered for the Repositories of \ ``REUSE``\  mode and \ ``BATCH``\  mode, with respect to all the Repositories.

* Create \ ``BeanNameGenerator``\ .

 .. code-block:: java

    package com.example.domain.repository;

    import org.springframework.beans.factory.config.BeanDefinition;
    import org.springframework.beans.factory.support.BeanDefinitionRegistry;
    import org.springframework.beans.factory.support.BeanNameGenerator;
    import org.springframework.util.ClassUtils;

    import java.beans.Introspector;

    // (1)
    public class BachRepositoryBeanNameGenerator implements BeanNameGenerator {
        // (2)
        @Override
        public String generateBeanName(BeanDefinition definition, BeanDefinitionRegistry registry) {
            String defaultBeanName = Introspector.decapitalize(ClassUtils.getShortName(definition
                    .getBeanClassName()));
            return defaultBeanName.replaceAll("Repository", "BatchRepository");
        }
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Create a class that generates the Bean name to be registered in Spring ApplicationContext.

        This class is necessary to avoid duplication in the bean name of \ ``REUSE``\  mode Repository for normal use and
        the Bean name of \ ``BATCH``\  mode.
    * - (2)
      - Implement the method for generating a Bean name.

        In the above example, duplication of Bean name for \ ``REUSE``\  mode Repository for normal use can be prevented by setting \ ``BatchRepository``\  as the Bean name suffix.
        

|

* Bean definition is added to :file:`projectName-domain/src/main/resources/META-INF/spring/projectName-infra.xml`.

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:context="http://www.springframework.org/schema/context"
           xmlns:mybatis="http://mybatis.org/schema/mybatis-spring"
           xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd
           http://mybatis.org/schema/mybatis-spring
           http://mybatis.org/schema/mybatis-spring.xsd">

        <bean id="sqlSessionFactory"
              class="org.mybatis.spring.SqlSessionFactoryBean">
            <property name="dataSource" ref="dataSource"/>
            <property name="configLocation"
                      value="classpath:META-INF/mybatis/mybatis-config.xml"/>
        </bean>

        <!-- ... -->

        <bean id="batchSqlSessionTemplate"
              class="org.mybatis.spring.SqlSessionTemplate">
            <constructor-arg index="0" ref="sqlSessionFactory"/>
            <constructor-arg index="1" value="BATCH"/>
        </bean>

        <!-- (3) -->
        <mybatis:scan base-package="com.example.domain.repository"
            template-ref="batchSqlSessionTemplate"
            name-generator="com.example.domain.repository.BatchRepositoryBeanNameGenerator"/>

    </beans>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.65\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 15 65

    * - Sr. No.
      - Attribute
      - Description
    * - (3)
      - \-
      - Register the Bean for batch mode  Repository by using \ ``mybatis:scan``\  element.
    * -
      - base-package
      - Specify the base package that scans the Repository.

        Repository interface that exists under specified package is scanned
        and a Bean is registered in Spring ApplicationContext.
    * -
      - template-ref
      - Specify Bean of \ ``SqlSessionTemplate``\  for batch mode.
    * -
      - name-generator
      - Specify a class for generating the Bean name of scanned Repository.

        Basically, specify the class name (FQCN) for the class created in (1).

        If class name is not specified, Bean names are duplicated.
        Hence, batch mode Repository is not registered in Spring ApplicationContext.

|

.. _DataAccessMyBatis3HowToExtendExecutorTypeBatchUsage:

How to use the Repository of batch mode
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

An implementation example on how to access database by using batch mode Repository is given below.

 .. code-block:: java

    @Transactional
    @Service
    public class TodoServiceImpl implements TodoService {

        // (1)
        @Inject
        @Named("todoBatchRepository")
        TodoRepository todoBatchRepository;

        @Override
        public void updateTodos(List<Todo> todos) {
            for (Todo todo : todos) {
                // (2)
                todoBatchRepository.update(todo);
            }
        }

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Inject the Repository of batch mode.
    * - (2)
      - Call the method for batch mode Repository and update Entity.

        In case of batch mode Repository, since the SQL is not executed in the timing when method is called.
        As a result, the update results returned from method need to be ignored.

        The SQL for updating an Entity is executed in a batch immediately before committing a transaction
        and the transaction is committed if there is no error.

 .. note:: **Batch execution timing**

    Timing of SQL batch execution is as below.

    * Immediately before the transaction is committed
    * Immediately before executing the query (SELECT)

    Notes of sequence of calling repository method refer to ":ref:`DataAccessMyBatis3HowToExtendExecutorTypeBatchNotesMethodCallOrder`".

|

.. _DataAccessMyBatis3HowToExtendExecutorTypeBatchNotes:

Precautions when using batch mode Repository
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

It is important to note the following points in implementation of Service class when using batch mode Repository.


* :ref:`DataAccessMyBatis3HowToExtendExecutorTypeBatchNotesUpdateResult`
* :ref:`DataAccessMyBatis3HowToExtendExecutorTypeBatchNotesDuplicate`
* :ref:`DataAccessMyBatis3HowToExtendExecutorTypeBatchNotesMethodCallOrder`

.. _DataAccessMyBatis3HowToExtendExecutorTypeBatchNotesUpdateResult:

Determination of update results
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

When batch mode Repository is used, the validity of update results cannot be checked.

Update results returned from Mapper interface method when a batch mode is used, are as follows.

* \ `Fixed value <http://mybatis.github.io/mybatis-3/apidocs/reference/org/apache/ibatis/executor/BatchExecutor.html#BATCH_UPDATE_RETURN_VALUE>`_\ (\ ``org.apache.ibatis.executor.BatchExecutor#BATCH_UPDATE_RETURN_VALUE``\ )  when return value is numeric (\ ``int``\  or \ ``long``\ )
* \ ``false``\  when return value is \ ``boolean``\



This is due to the mechanism wherein SQL is not executed within the timing of calling  Mapper interface method
but is queued for batch execution (\ ``java.sql.Statement#addBatch()``\ ).

It signifies that the implementation below cannot be performed.

 .. code-block:: java

    @Transactional
    @Service
    public class TodoServiceImpl implements TodoService {

        @Inject
        @Named("todoBatchRepository")
        TodoRepository todoBatchRepository;

        @Override
        public void updateTodos(List<Todo> todos) {
            for (Todo todo : todos) {
                boolean updateSuccess = todoBatchRepository.update(todo);
                // (1)
                if (!updateSuccess) {
                    // ...
                }
            }
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - When the implementation is performed as described below, update results are always returned as \ ``false``\  resulting
        in the execution of process at the time of update failure.


Based on the application requirement, it is also necessary to check the validity of update results executed in the batch.
In such cases, "a method to execute SQL queued for batch execution" is provided in the Mapper interface."

In MyBatis 3.2, \ ``flushStatements``\  method of \ ``org.apache.ibatis.session.SqlSession``\  interface must be called directly, however,
a method is supported in MyBatis 3.3.0 and subsequent versions supported in terasoluna-gfw-mybatis3 5.2.x.RELEASE wherein a method which assigns
\ ``@org.apache.ibatis.annotations.Flush``\  annotation in Mapper interface is created.

 .. warning:: **Regarding update results returned by JDBC driver while using batch mode**

    Although it has been described earlier that update results at the time of batch execution can be received when a method which assigns \ ``@Flush``\  annotation (and \ ``flushStatements``\  method of \ ``SqlSession``\  interface) is used,
    it cannot be guaranteed that the update results returned from JDBC driver can be used as "number of processed records".

    Since it depends on implementation of JDBC driver to be used, the specifications of JDBC driver to be used must be checked in advance.

How to create and call a method which assigns \ ``@Flush``\  annotation is given below.
|

 .. code-block:: java

    public interface TodoRepository {
        // ...
        @Flush // (1)
        List<BatchResult> flush();
    }

 .. code-block:: java

    @Transactional
    @Service
    public class TodoServiceImpl implements TodoService {

        @Inject
        @Named("todoBatchRepository")
        TodoRepository todoBatchRepository;

        @Override
        public void updateTodos(List<Todo> todos) {
        
            for (Todo todo : todos) {
                todoBatchRepository.update(todo);                
            }

            List<BatchResult> updateResults = todoBatchRepository.flush(); // (2)

            // Validate update results
            // ...

        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Create a method which assigns \ ``@Flush``\  annotation (hereafter referred to as "\ ``@Flush``\  method").

        When it is necessary to determine update results, specify list type of \ ``org.apache.ibatis.executor.BatchResult``\  as a return value.
        When it is not necessary to determine update results (when only a database error like a unique constraint violation is to be handled), the return value can be \ ``void``\ .
    * - (2)
      - Call \ ``@Flush``\  method within the timing in which SQL queued for batch execution is to be executed.
        If \ ``@Flush``\  method is called, \ ``flushStatements``\  method of \ ``SqlSession``\  object associated with Mapper interface is called and
        SQL queued for batch execution is executed.

        When it is necessary to determine update results, validity of update results returned from \ ``@Flush``\  method is checked.

|

.. _DataAccessMyBatis3HowToExtendExecutorTypeBatchNotesDuplicate:

How to detect the unique constraint violation
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

When batch mode Repository is used,
it is not possible to detect database errors like unique constraint violation etc. as a Service process.

This is due to the mechanism wherein SQL is not executed within the timing of calling a Mapper interface method
and is queued for the batch execution (\ ``java.sql.Statement#addBatch()``\ ).
It signifies that the implementation given below cannot be performed.

 .. code-block:: java

    @Transactional
    @Service
    public class TodoServiceImpl implements TodoService {

        @Inject
        @Named("todoBatchRepository")
        TodoRepository todoBatchRepository;

        @Override
        public void storeTodos(List<Todo> todos) {
            for (Todo todo : todos) {
                try {
                    todoBatchRepository.create(todo);
                // (1)
                } catch (DuplicateKeyException e) {
                    // ....
                }
            }
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - When the implementation is performed as described above,
        \ ``org.springframework.dao.DuplicateKeyException``\  exception does not occur within that timing.
        Hence, the process after \ ``DuplicateKeyException``\  supplement is not executed.

        This is because SQL batch execution is performed after termination of Service process (just before the transaction is committed).
        


Depending on application requirement, it is necessary to detect a unique constraint violation at the time of batch execution.
In such cases, "a method to execute SQL queued for batch execution (\ ``@Flush``\  method)" must be provided in Mapper interface.
Refer ":ref:`DataAccessMyBatis3HowToExtendExecutorTypeBatchNotesUpdateResult`" described earlier for details of \ ``@Flush``\  method.


|

.. _DataAccessMyBatis3HowToExtendExecutorTypeBatchNotesMethodCallOrder:

Sequence of calling Repository method
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
Batch mode is used to improve the performance of update process.
However, if the calling sequence of Repository method is incorrect, no improvement is observed in the performance.

It is important to understand the MyBatis specifications given below, in order to improve the performance using batch mode.

* When query (SELECT) is executed, SQL waiting in the queue till then is executed in batch.
* \ ``PreparedStatement``\  is generated for each update process (Repository method) called in succession and SQL is queued.

It signifies that if the implementation is performed as below, advantages of using batch mode cannot be obtained.

* Example１

 .. code-block:: java

    @Transactional
    @Service
    public class TodoServiceImpl implements TodoService {

        @Inject
        @Named("todoBatchRepository")
        TodoRepository todoBatchRepository;

        @Override
        public void storeTodos(List<Todo> todos) {
            for (Todo todo : todos) {
                // (1)
                Todo currentTodo = todoBatchRepository.findOne(todo.getTodoId());
                if (currentTodo == null) {
                    todoBatchRepository.create(todo);
                } else{
                    todoBatchRepository.update(todo);
                }
            }
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - When the implementation is performed as described in the above example, since a query is executed at the start of iteration process,
        SQL is executed in a batch for each record.
        This is almost same as the execution in simple mode (\ ``SIMPLE``\ ).

        When the process described above is necessary,
        it is more effective to use Repository of PreparedStatement reuse mode (\ ``REUSE``\ ).

* Example 2

 .. code-block:: java

    @Transactional
    @Service
    public class TodoServiceImpl implements TodoService {

        @Inject
        @Named("todoBatchRepository")
        TodoRepository todoBatchRepository;

        @Override
        public void storeTodos(List<Todo> todos) {
            for (Todo todo : todos) {
                // (2)
                todoBatchRepository.create(todo);
                todoBatchRepository.createHistory(todo);
            }
        }

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (2)
      - When the process described above is necessary, Repository method is called alternately.
        Hence, \ ``PreparedStatement``\  is generated for each record.
        This is almost same as executing a process in simple mode (\ ``SIMPLE``\ ).

        When the process described above is necessary,
        it is more effective to use PreparedStatement reuse mode  Repository (\ ``REUSE``\ ).

|

.. _DataAccessMyBatis3HowToExtendStoredProcedure:

Implementation of a stored procedure
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

How to call a stored procedure or function registered in database from MyBatis3 is explained.


A function registered in PostgreSQL is called in the implementation example explained below.

* Register a stored procedure (function).

 .. code-block:: guess

    /* (1) */
    CREATE FUNCTION findTodo(pTodoId CHAR)
    RETURNS TABLE(
        todo_id CHAR,
        todo_title VARCHAR,
        finished BOOLEAN,
        created_at TIMESTAMP,
        version BIGINT
    ) AS $$ BEGIN RETURN QUERY
    SELECT
        t.todo_id,
        t.todo_title,
        t.finished,
        t.created_at,
        t.version
    FROM
        t_todo t
    WHERE
        t.todo_id = pTodoId;
    END;
    $$ LANGUAGE plpgsql;

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - It is the function used to fetch records for specified ID.

|

* Defining the method in Repository interface.

 .. code-block:: java

    // (2)
    public interface TodoRepository extends Repository {
        Todo findOne(String todoId);
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (2)
      - The interface should be same as the interface used while executing the SQL.

|

* Call a stored procedure in the mapping file.

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <!-- (3) -->
        <select id="findOne" parameterType="string" resultType="Todo"
                statementType="CALLABLE">
            <!-- (4) -->
            {call findTodo(#{todoId})}
        </select>

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (3)
      - Implement the statement to call the stored procedure.

        Specify \ ``CALLABLE``\  in \ ``statementType``\  attribute when calling the stored procedure.
        When \ ``CALLABLE``\  is specified,
        the stored procedure is called by using \ ``java.sql.CallableStatement``\ .

        Specify \ ``resultType``\  attribute or \ ``resultMap``\  attribute in order to map the OUT parameter in JavaBean.
        
    * - (4)
      - Call the stored procedure.

        When the stored procedure (faction) is to be called,

        specify in \ ``{call Procedure or Function name (IN parameter...)}``\  format.

        

        In the above example, the procedure is called by specifying an ID in the IN parameter for a function called \ ``findTodo``\ .
        

|

.. _DataAccessMyBatis3Appendix:

Appendix
--------------------------------------------------------------------------------

.. _DataAccessMyBatis3AppendixAboutMapperMechanism:

Mapper interface mechanism
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| When using a Mapper interface, the developer can execute SQL only by creating a Mapper interface and mapping file.
| when executing an application, implementation class of Mapper interface is generated by MyBatis3 using the Proxy function of JDK.
 Hence, the developer need not create an implementation class of Mapper interface.

| It is not necessary to define inheritance and annotation of interface provided by MyBatis3, for the Mapper interface and it
 can be simply created as a Java interface.
| Example of how to create a Mapper interface and mapping file, and example of how to use it in the application (Service) are given below.
| Here, the focus should be on the main points related to explanation of code since the aim is to form an image of the deliverables to be created by developers.

- How to create a Mapper interface

  In this guideline, since it is presumed that that the Mapper interface of MyBatis3 is used as Repository interface,
  interface name is in the format,  "Entity name" + \ ``"Repository"`` \.

 .. code-block:: java

    package com.example.domain.repository.todo;

    import com.example.domain.model.Todo;

    public interface TodoRepository {
        Todo findOne(String todoId);
    }

- How to create a mapping file

  In the mapping file, FQCN (fully qualified class name) of Mapper interface is specified as namespace.
  Its association with SQL that is executed while calling the method defined in the Mapper interface
  can be formed by specifying a method name in id attribute of various statement tags (insert/update/delete/select tags).

 .. code-block:: xml
    :emphasize-lines: 4, 12

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org/DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <resultMap id="todoResultMap" type="Todo">
            <result column="todo_id" property="todoId" />
            <result column="title" property="title" />
            <result column="finished" property="finished" />
        </resultMap>

        <select id="findOne" parameterType="String" resultMap="todoResultMap">
          SELECT
            todo_id,
            title,
            finished
          FROM
            t_todo
          WHERE
            todo_id = #{todoId}
        </select>

    </mapper>

- How to use Mapper interface in an application (Service)

  When a method of Mapper interface is to be called from an application (Service), a method of Mapper object injected by Spring (DI container) is called.
  The application (Service) transparently executes SQL by calling Mapper object method and can obtain SQL execution results.

 .. code-block:: java
    :emphasize-lines: 12

    package com.example.domain.service.todo;

    import com.example.domain.model.Todo;
    import com.example.domain.repository.todo.TodoRepository;

    public class TodoServiceImpl implements TodoService {

        @Inject
        TodoRepository todoRepository;

        public Todo getTodo(String todoId){
            Todo todo = todoRepository.findOne(todoId);
            if(todo == null){
                throw new ResourceNotFoundException(
                    ResultMessages.error().add("e.ex.td.5001" ,todoId));
            }
            return todo;
        }

    }

|

The process flow up to SQL execution when Mapper interface method is called, is shown below.


 .. figure:: images_DataAccessMyBatis3/DataAccessMyBatis3MapperMechanism.png
    :alt: Mapper mechanism
    :width: 100%
    :align: center

    **Picture - Mapper mechanism**

 .. raw:: latex

    \newpage

 .. tabularcolumns:: |p{0.1\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80
    :class: longtable


    * - Sr. No.
      - Description
    * - (1)
      - Application calls the method defined in Mapper interface.

        The implementation class of Mapper interface (Proxy object of Mapper interface) is generated by MyBatis3 components at application startup time.
    * - (2)
      - Proxy object of Mapper interface calls invoke method of \ ``MapperProxy`` \ .

        Role of \ ``MapperProxy`` \  is to handle the method calling of Mapper interface.
    * - (3)
      - \ ``MapperProxy`` \  generates \ ``MapperMethod`` \  corresponding to the called Mapper interface method and calls execute method.

        \ ``MapperMethod`` \  plays the role of calling \ ``SqlSession`` \  method corresponding to the called Mapper interface method.
    * - (4)
      - \ ``MapperMethod`` \  calls \ ``SqlSession`` \  method.

        When a \ ``SqlSession`` \  method is called, a key (hereafter referred to as "Statement ID") is passed in order to specify the SQL statement to be executed.
    * - (5)
      - \ ``SqlSession`` \  fetches a SQL statement from a mapping file using the specified statement ID as a key.
    * - (6)
      - \ ``SqlSession`` \  sets the value in bind variable specified in the SQL statement fetched by mapping file and executes SQL.
    * - (7)
      - Mapper interface (\ ``SqlSession`` \ ) converts SQL execution results to JavaBean etc. and returns it to the application.

        When number of records or number of updated records are to be fetched, primitive type or primitive wrapper type etc. form the return values.

 .. raw:: latex

    \newpage

 .. tip:: **Statement ID**

    Statement ID is a key to specify the SQL statement to be executed.
    It is generated in accordance with the rule \ **"FQCN of Mapper interface + "." + name of the called Mapper interface method"**\.

    When SQL statement corresponding to the statement ID generated by \ ``MapperMethod`` \  is to be defined in the mapping file,
    it is necessary to specify "FQCN of Mapper interface" in the namespace of mapping file
    and "method name of Mapper interface" in id attribute of various statement tags.

|

.. _DataAccessMyBatis3AppendixSettingsTypeAlias:

TypeAlias settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
TypeAlias should basically be configured per package by using \ ``package`` \  element. However, following methods can also be used.

* A method to configure an alias name per class
* A method to overwrite the alias name which is assigned as default (a method that specifies an optional alias name)



Configuring TypeAlias per class
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
TypeAlias can also be configured per class.

- :file:`projectName-domain/src/main/resources/META-INF/mybatis/mybatis-config.xml`

 .. code-block:: xml
    :emphasize-lines: 2-4

    <typeAliases>
        <!-- (1) -->
        <typeAlias
            type="com.example.domain.repository.account.AccountSearchCriteria" />
        <package name="com.example.domain.model" />
    </typeAliases>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80

   * - Sr. No.
     - Description
   * - (1)
     - Specify the fully qualified class name (FQCN) of the class for which alias is to be set, in \ ``type`` \  attribute of \ ``typeAlias`` \  element.

       In the above example, alias name of \ ``com.example.domain.repository.account.AccountSearchCriteria`` \  class becomes
       \ ``AccountSearchCriteria`` \  (part after removing package part).
       
       When an optional value is to be specified in the alias name, optional alias name can be specified in \ ``alias`` \  attribute of \ ``typeAlias`` \  element.


Overwriting alias name assigned as default
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
When alias is set by using \ ``package`` \  element or
when alias is set by omitting \ ``alias`` \  attribute of \ ``typeAlias`` \  element,
alias for TypeAlias is a part obtained after removing the package part from fully qualified class name (FQCN).

When optional alias is to be used instead of default alias,
it can be specified by specifying \ ``@org.apache.ibatis.type.Alias`` \  annotation in the class wherein TypeAlias to is to be configured.


- Java class for configuring alias

 .. code-block:: java
    :emphasize-lines: 3

    package com.example.domain.model.book;

    @Alias("BookAuthor") // (1)
    public class Author {
       // ...
    }
    
 .. code-block:: java
    :emphasize-lines: 3

    package com.example.domain.model.article;

    @Alias("ArticleAuthor") // (1)
    public class Author {
       // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80

   * - Sr. No.
     - Description
   * - (1)
     - Specify the alias name in \ ``value`` \  attribute of \ ``@Alias`` \  annotation.

       In the above example, the alias name for \ ``com.example.domain.model.book.Author`` \  class is
       \ ``BookAuthor`` \.
       
       When a class with same name is stored in a different package, different alias names can be configured for each class by using this method.
       However, in this guideline, it is recommended to design the class name such that duplication is avoided.
       In the above example, \ ``BookAuthor`` \  and \ ``ArticleAuthor`` \  should be considered as class names.

 .. tip::
 
    An alias name for TypeAlias is applied in the following priority order.
    
     * Value specified for \ ``alias`` \  attribute of \ ``typeAlias`` \  element
     * Value specified for \ ``value`` \  attribute of ``@Alias`` \  annotation
     * Alias name assigned as default (part after removing the package name from fully qualified class name)
    
    


|

.. _DataAccessMyBatis3AppendixSwitchingSqlByDatabase:

SQL switching by the database
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

MyBatis3 provides a mechanism (\ ``org.apache.ibatis.mapping.VendorDatabaseIdProvider``\ ) wherein vendor information of database that is connected from JDBC driver, is fetched
and SQL to be used is switched.

This mechanism is effective when building an application that can support multiple databases as operating environment.

 .. note::

    In this guideline, it is recommended to manage the components and configuration file that are dependent on the environment by a sub project called [projectName]-env
    and create components and configuration file that are in execution environment at the time of building.
    

    [projectName]-env is a sub-project to absorb differences in each of the following

    * Development environment (local PC environment)
    * Various test environments
    * Commercial environment

    
    It can also be used in the development of an application that supports multiple databases.

    Basically, it is recommended to manage environment-dependent components and configuration file
    by using a sub-project called [projectName]-env.
    However, the mechanism below can also be used if only a minor SQL difference is to be absorbed.

    **Architects should try to achieve a uniform implementation for overall application by clearly identifying a guideline**
    **on how to implement SQL environment dependency based on the differences in the database.**

|

- Bean is defined in :file:`projectName-domain/src/main/resources/META-INF/spring/projectName-infra.xml`.

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:mybatis="http://mybatis.org/schema/mybatis-spring"
        xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://mybatis.org/schema/mybatis-spring
            http://mybatis.org/schema/mybatis-spring.xsd
        ">

        <import resource="classpath:/META-INF/spring/projectName-env.xml" />

        <!-- (1) -->
        <bean id="databaseIdProvider"
              class="org.apache.ibatis.mapping.VendorDatabaseIdProvider">
            <!-- (2) -->
            <property name="properties">
                <props>
                    <prop key="H2">h2</prop>
                    <prop key="PostgreSQL">postgresql</prop>
                </props>
            </property>
        </bean>

        <bean id="sqlSessionFactory"
            class="org.mybatis.spring.SqlSessionFactoryBean">
            <property name="dataSource" ref="dataSource" />
            <!-- (3) -->
            <property name="databaseIdProvider" ref="databaseIdProvider"/>
            <property name="configLocation"
                value="classpath:/META-INF/mybatis/mybatis-config.xml" />
        </bean>

        <mybatis:scan base-package="com.example.domain.repository" />

    </beans>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Define a Bean for \ ``VendorDatabaseIdProvider``\  provided by MyBatis3.

        \ ``VendorDatabaseIdProvider``\  is a class
        for handling a product name (\ ``java.sql.DatabaseMetaData#getDatabaseProductName()``\) of a database that is fetched from JDBC driver, as a database ID.
    * - (2)
      - Specify the mapping of database product name fetched from JDBC driver and database ID in \ ``properties``\  property.

        Refer to "`MyBatis3 REFERENCE DOCUMENTATION(Configuration-databaseIdProvider-) <http://mybatis.github.io/mybatis-3/configuration.html#databaseIdProvider>`_\" for specifications of mapping.
    * - (3)
      - Specify \ ``DatabaseIdProvider``\  defined in (1) for the \ ``databaseIdProvider``\  property of \ ``SqlSessionFactoryBean``\  that uses database ID
        

        By specifying this, it is possible to refer the database ID from mapping file.

 .. note::

    In this guideline, it is recommended to use a method to map the product name of database and database ID, by specifying \ ``properties``\  property
    

    This is due to possible change in the product name of database fetched from JDBC driver, based on the JDBC version.
    
    When \ ``properties``\  property is used, the difference between product names of each version to be used can be managed at a single location.
    

|

- Implementing mapping file.

 .. code-block:: xml

    <insert id="create" parameterType="Todo">
        <!-- (1) -->
        <selectKey keyProperty="todoId" resultType="string" order="BEFORE"
                   databaseId="h2">
            SELECT RANDOM_UUID()
        </selectKey>
        <selectKey keyProperty="todoId" resultType="string" order="BEFORE"
                   databaseId="postgresql">
            SELECT UUID_GENERATE_V4()
        </selectKey>

        INSERT INTO
          t_todo
        (
            todo_id
            ,todo_title
            ,finished
            ,created_at
            ,version
        )
        VALUES
        (
            #{todoId}
            ,#{todoTitle}
            ,#{finished}
            ,#{createdAt}
            ,#{version}
        )
    </insert>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - When statement elements (\ ``select``\  element, \ ``update``\  element, \ ``sql``\  element etc.) are to be changed for each database,
        specify the database ID in \ ``databaseId``\  attribute of each element.

        When \ ``databaseId``\  attribute is specified, the statement element that matches with database ID is used.

        In the above example, ID is generated, by calling the UUID generation function specific for each database.

 .. tip::

    In the above example, \ ``UUID_GENERATE_V4()``\  is called as the UUID generation function for PostgreSQL.
    However, this function belongs to a sub-module called `uuid-ossp <http://www.postgresql.org/docs/9.4/static/uuid-ossp.html>`_\.

    When this function is to be used, uuid-ossp module should be enabled.

 .. tip::

    Database ID can also be referred in OGNL base expression (Expression language).

    It signifies that database ID can be used as a condition for dynamic SQL.
    How to implement is given below.

     .. code-block:: xml

        <select id="findAllByCreatedAtBefore" parameterType="_int" resultType="Todo">
            SELECT
                todo_id,
                todo_title,
                finished,
                created_at,
                version
            FROM
                t_todo
            WHERE
                <choose>
                    <!-- (2) -->
                    <when test="_databaseId == 'h2'">
                        <bind name="criteriaDate"
                              value="'DATEADD(\'DAY\',#{days} * -1,#{currentDate})'"/>
                    </when>
                    <when test="_databaseId == 'postgresql'">
                        <bind name="criteriaDate"
                              value="'#{currentDate}::DATE - (#{days} * INTERVAL \'1 DAY\')'"/>
                    </when>
                </choose>
                <![CDATA[
                    created_at < ${criteriaDate}
                ]]>
        </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (2)
      - Database ID is stored in a specific variable called \ ``_databaseId``\, in a OGNL base expression (Expression language).
        

        In the above example, the condition for extracting records that are created prior to "System date - specified date" is specified by using database function.
        


|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnce:

How to fetch a related Entity by a single SQL
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

How to fetch a main Entity and a related Entity by a single SQL is explained below.

When a mechanism that fetches a main Entity and a related Entity together is to be used,
it is not necessary to build an Entity (JavaBean) in the Service class and Service class
can solely focus on implementation of business logic (business rules).

| Further, this method can also be used to avoid the N+1.
| Refer to ":ref:`data-access-common_howtosolve_n_plus_1`" for N+1.

 .. warning::

    It is important to note the following points when main Entity and related Entity are to be fetched together.

    * In the explanation below, all the related Entities are fetched together by a single SQL.
      However, when it is to be used in the actual project,
      only the related Entities that are required in the process should be fetched.
      If unused related Entities are fetched together, it results in generation of unnecessary objects and as mapping process is performed,
      it causes efficiency deterioration.
      **Particularly, in the SQL that performs list search, in many cases, only the required related Entities are fetched.**

    \

    * For the related Entities which are used less frequently, the entities can be fetched independently without fetching them together.
      
      If the related Entities that are used less frequently are fetched simultaneously,
      unnecessary objects are generated and as mapping process is performed, it can cause efficiency deterioration.

    \

    * When multiple related Entities with 1:N relation are to be included,
      it is advisable to adopt the method wherein the main Entity and related Entity are fetched separately.
      When there are multiple related Entities with 1:N relation,
      it becomes necessary to fetch unnecessary data from DB causing deterioration inefficiency.
      Refer to ":ref:`data-access-common_howtosolve_n_plus_1`" for how to fetch main Entity and related Entity separately.
      

 .. tip::

    Following methods are provided when it is necessary to separately fetch the related Entities that are used less frequently.

    * By calling a method (SQL) that fetches a related Entity in the process of Service class.
    * By marking the related Entity as "Lazy Load" and transparently executing the SQL when Getter method is called.

    

    When "Lazy Load" mechanism is used,
    it is not necessary to build an Entity (JavaBean) in Service class and
    Service class can solely focus on implementation of business logic (business rules).

    **When "Lazy Load" is used in SQL that performs list search, N+1 can occur. Hence adequate care must be taken.**

    Refer to ":ref:`DataAccessMyBatis3AppendixNestedSelectLazySetting`" for how to use "Lazy Load".


|

An implementation example is explained below wherein order data handled by a shopping site
is fetched together in a single SQL and mapped in main Entity and related Entity.

The implementation method explained here is just an example.
MyBatis3 provides a lot of functions that are not explained in this section and can perform a much higher level of mapping.

Refer to "\ `MyBatis3 REFERENCE DOCUMENTATION(Mapper XML Files-Result Maps-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#Result_Maps>`_ \" for details of mapping function of MyBatis3.


|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceTable:

Table layout and data
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Table used in the explanation is as given below.

 .. figure:: ../../ImplementationAtEachLayer/images/service_entity_table_layout.png
    :alt: ER diagram
    :width: 100%
    :align: center

    **Picture - ER diagram**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|p{0.50\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 15 15 50

    * - Sr. No.
      - Category
      - Table name
      - Description
    * - (1)
      - Transaction system
      - t_order
      - A table that stores order data.

        1 record is stored for one order.
    * - (2)
      -
      - t_order_item
      - A table that stores the product data purchased by one order.

        When multiple products are purchased in a single order, the number of product records is stored.
    * - (3)
      -
      - t_order_coupon
      - A table that stores data of coupons that are used in a single order.

        When multiple coupons are used in a single order, the number of coupon records is stored.
        When coupons are not used, records are not stored.
    * - (4)
      - Master system
      - m_item
      - A master table that defines the product.
    * - (5)
      -
      - m_category
      - A master table that defines product category.
    * - (6)
      -
      - m_item_category
      - A master table that defines the category to which a product belongs.

        It stores the mapping of product and category.
        It is a model wherein one product can belong to multiple categories.
    * - (7)
      -
      - m_coupon
      - A master table that defines a coupon.
    * - (8)
      - Code
      - c_order_status
      - A code table that defines the order status.

|

SQL (DDL and DML) for creating the storage data and the table layout used in the explanation are as follows.
(SQL is for H2 Database)

* DDL for creating a master table

 .. code-block:: sql

    CREATE TABLE m_item (
        code CHAR(10),
        name NVARCHAR(256),
        price INTEGER,
        CONSTRAINT m_item_pk PRIMARY KEY(code)
    );

    CREATE TABLE m_category (
        code CHAR(10),
        name NVARCHAR(256),
        CONSTRAINT m_category_pk PRIMARY KEY(code)
    );

    CREATE TABLE m_item_category (
        item_code CHAR(10),
        category_code CHAR(10),
        CONSTRAINT m_item_category_pk PRIMARY KEY(item_code, category_code),
        CONSTRAINT m_item_category_fk1 FOREIGN KEY(item_code) REFERENCES m_item(code),
        CONSTRAINT m_item_category_fk2 FOREIGN KEY(category_code) REFERENCES m_category(code)
    );

    CREATE TABLE m_coupon (
        code CHAR(10),
        name NVARCHAR(256),
        price INTEGER,
        CONSTRAINT m_coupon_pk PRIMARY KEY(code)
    );

* DDL for creating a code table

 .. code-block:: sql

    CREATE TABLE c_order_status (
        code VARCHAR(10),
        name NVARCHAR(256),
        CONSTRAINT c_order_status_pk PRIMARY KEY(code)
    );

* DDL for creating a transaction table

 .. code-block:: sql

    CREATE TABLE t_order (
        id INTEGER,
        status_code VARCHAR(10),
        CONSTRAINT t_order_pk PRIMARY KEY(id),
        CONSTRAINT t_order_fk FOREIGN KEY(status_code) REFERENCES c_order_status(code)
    );

    CREATE TABLE t_order_item (
        order_id INTEGER,
        item_code CHAR(10),
        quantity INTEGER,
        CONSTRAINT t_order_item_pk PRIMARY KEY(order_id, item_code),
        CONSTRAINT t_order_item_fk1 FOREIGN KEY(order_id) REFERENCES t_order(id),
        CONSTRAINT t_order_item_fk2 FOREIGN KEY(item_code) REFERENCES m_item(code)
    );

    CREATE TABLE t_order_coupon (
        order_id INTEGER,
        coupon_code CHAR(10),
        CONSTRAINT t_order_coupon_pk PRIMARY KEY(order_id, coupon_code),
        CONSTRAINT t_order_coupon_fk1 FOREIGN KEY(order_id) REFERENCES t_order(id),
        CONSTRAINT t_order_coupon_fk2 FOREIGN KEY(coupon_code) REFERENCES m_coupon(code)
    );

* DML for data input

 .. code-block:: sql

    -- Setup master tables
    INSERT INTO m_item VALUES ('ITM0000001','Orange juice',100);
    INSERT INTO m_item VALUES ('ITM0000002','NotePC',100000);

    INSERT INTO m_category VALUES ('CTG0000001','Drink');
    INSERT INTO m_category VALUES ('CTG0000002','PC');
    INSERT INTO m_category VALUES ('CTG0000003','Hot selling');

    INSERT INTO m_item_category VALUES ('ITM0000001','CTG0000001');
    INSERT INTO m_item_category VALUES ('ITM0000002','CTG0000002');
    INSERT INTO m_item_category VALUES ('ITM0000002','CTG0000003');

    INSERT INTO m_coupon VALUES ('CPN0000001','Join coupon',3000);
    INSERT INTO m_coupon VALUES ('CPN0000002','PC coupon',30000);

    -- Setup code tables
    INSERT  INTO  c_order_status VALUES ('accepted','Order accepted');
    INSERT  INTO  c_order_status VALUES ('checking','Stock checking');
    INSERT  INTO  c_order_status VALUES ('shipped','Item Shipped');

    -- Setup transaction tables
    INSERT INTO t_order VALUES (1,'accepted');
    INSERT INTO t_order VALUES (2,'checking');

    INSERT INTO t_order_item VALUES (1,'ITM0000001',1);
    INSERT INTO t_order_item VALUES (1,'ITM0000002',2);
    INSERT INTO t_order_item VALUES (2,'ITM0000001',3);
    INSERT INTO t_order_item VALUES (2,'ITM0000002',4);

    INSERT INTO t_order_coupon VALUES (1,'CPN0000001');
    INSERT INTO t_order_coupon VALUES (1,'CPN0000002');

    COMMIT;

|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceEntity:

Entity class diagram
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

In the implementation example, the records stored in the table above are mapped in the following Entity (JavaBean).

 .. figure:: images/dataaccess_entity.png
    :alt: Class(JavaBean) diagram
    :width: 100%
    :align: center

    **Picture - Class(JavaBean) diagram**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.65\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 15 65
    :class: longtable

    * - Sr. No.
      - Class name
      - Description
    * - (1)
      - Order
      - JavaBean that represents 1 record of t_order table.

        A single record of \ ``OrderStatus``\  and multiple records of \ ``OrderItem``\  and \ ``OrderCoupon``\  are stored as related Entities.

         .. code-block:: java

            public class Order implements Serializable {
                private static final long serialVersionUID = 1L;
                private int id;
                private OrderStatus orderStatus;
                List<OrderItem> orderItems;
                List<OrderCoupon> orderCoupons;
                // ...
            }

    * - (2)
      - OrderItem
      - JavaBean that represents 1 record of t_order_item table.

        \ ``Item``\  is stored as related Entity.

         .. code-block:: java

            public class OrderItem implements Serializable {
                private static final long serialVersionUID = 1L;
                private int orderId;
                private Item item;
                private int quantity;
                // ...
            }

    * - (3)
      - OrderCoupon
      - A JavaBean that represents 1 record of t_order_coupon table.

        \ ``Coupon``\  is stored as related Entity.

         .. code-block:: java

            public class OrderCoupon implements Serializable {
                private static final long serialVersionUID = 1L;
                private int orderId;
                private Coupon coupon;
                // ...
            }

    * - (4)
      - Item
      - JavaBean that represents 1 record of m_item table.

        Multiple, associated \ ``Category``\  records are stored as related objects.
        Mapping with \ ``Category``\  is performed by m_item_category table.

         .. code-block:: java

            public class Item implements Serializable {
                private static final long serialVersionUID = 1L;
                private String code;
                private String name;
                private int price;
                private List<Category> categories;
                // ...
            }

    * - (5)
      - Category
      - JavaBean that represents 1 record of m_category table.

         .. code-block:: java

            public class Category implements Serializable {
                private static final long serialVersionUID = 1L;
                private String code;
                private String name;
                // ...
            }

    * - (6)
      - Coupon
      - JavaBean that represents 1 record of m_coupon table.

         .. code-block:: java

            public class Coupon implements Serializable {
                private static final long serialVersionUID = 1L;
                private String code;
                private String name;
                private int price;
                // ...
            }

    * - (7)
      - OrderStatus
      - A JavaBean that represents 1 record of c_order_status table.

         .. code-block:: java

            public class OrderStatus implements Serializable {
                private static final long serialVersionUID = 1L;
                private String code;
                private String name;
                // ...
            }

 .. raw:: latex

    \newpage

|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceRepository:

Implementation of Repository interface
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Following methods are implemented in the example.

* A method that fetches 1 record of Order object (\ ``findOne``\ )
* A method that fetches an Order object of corresponding page (\ ``findPage``\ )



 .. code-block:: java

    package com.example.domain.repository.order;

    import com.example.domain.model.Order;

    import java.util.List;

    public interface OrderRepository {

        Order findOne(int id);

        List<Order> findPage(@Param("pageable") Pageable pageable);

    }

|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceSql:

SQL implementation
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When related Entities are to be fetched together by a single SQL,
JOIN is performed on the tables to be fetched and all the records required for mapping are fetched.

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
    <mapper namespace="com.example.domain.repository.order.OrderRepository">

        <!-- (1) -->
        <sql id="selectFromJoin">
            SELECT
                /* (2) */
                o.id,
                /* (3) */
                o.status_code,
                os.name AS status_name,
                /* (4) */
                oi.quantity,
                i.code AS item_code,
                i.name AS item_name,
                i.price AS item_price,
                /* (5) */
                ct.code AS category_code,
                ct.name AS category_name,
                /* (6) */
                cp.code AS coupon_code,
                cp.name AS coupon_name,
                cp.price AS coupon_price
            FROM
                ${orderTable} o
            /* (7) */
            INNER JOIN c_order_status os ON os.code = o.status_code
            INNER JOIN t_order_item oi ON oi.order_id = o.id
            INNER JOIN m_item i ON i.code = oi.item_code
            INNER JOIN m_item_category ic ON ic.item_code = i.code
            INNER JOIN m_category ct ON ct.code = ic.category_code
            /* (8) */
            LEFT JOIN t_order_coupon oc ON oc.order_id = o.id
            LEFT JOIN m_coupon cp ON cp.code = oc.coupon_code
        </sql>

        <!-- (9) -->
        <select id="findOne" parameterType="_int" resultMap="orderResultMap">
            <bind name="orderTable" value="'t_order'" />
            <include refid="selectFromJoin"/>
            WHERE
                o.id = #{id}
            ORDER BY
                item_code ASC,
                category_code ASC,
                coupon_code ASC
        </select>

        <!-- (10) -->
        <select id="findPage" resultMap="orderResultMap">
            <bind name="orderTable" value="
                '(
                  SELECT
                      *
                  FROM
                      t_order
                  ORDER BY
                      id DESC
                  LIMIT #{pageable.pageSize}
                  OFFSET #{pageable.offset}
                  )'" />
            <include refid="selectFromJoin"/>
            ORDER BY
                id DESC,
                item_code ASC,
                category_code ASC,
                coupon_code ASC
        </select>

        <!-- ... -->

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Implement SELECT clause, FROM clause and JOIN clause for \ ``findOne``\  method and \ ``findPage``\  method.

        In the above example, the common location for \ ``findOne``\  method and \ ``findPage``\  method is standardized.
    * - (2)
      - Fetch the data required for generating an Order object.
    * - (3)
      - Fetch the data required for generating an OrderStatus object.

        Care must be taken to avoid duplicating the column name to be fetched.
        In the above example, \ ``name``\  column is duplicated.
        Hence, an alias name (\ ``status_``\  prefix) is specified using AS clause.
    * - (4)
      - Fetch the data required for generating an OrderItem object and Item object.

        Care must be taken to avoid duplication of column name to be fetched.
        In the above example, \ ``code``\ ,\ ``name``\  and  \ ``price``\  are duplicated.
        Hence, an alias name (\ ``item_``\  prefix) is specified using AS clause.
    * - (5)
      - Fetch the data required for generating Category object.

        Care must be taken to avoid duplication of column name to be fetched.
        In the above example, \ ``code``\  and \ ``name``\  are duplicated.
        Hence, an alias name (\ ``category_``\  prefix) is specified using AS clause.
    * - (6)
      - Fetch the data required for generating an OrderCoupon object and Coupon object.

        Care must be taken to avoid duplication of column name to be fetched.
        In the above example, \ ``code``\ ,\ ``name``\  and \ ``price``\ are duplicated.
        Hence, alias name (\ ``coupon_``\  prefix) is specified using AS clause.
    * - (7)
      - Perform JOIN for the tables that store data required for generating related objects.
    * - (8)
      - Perform Outer JOIN for the table wherein records might not be stored.
        When coupon is not used, it is necessary to perform Outer JOIN since records are not stored in t_order_coupon.
        This is also applicable for t_coupon to be joined with t_order_coupon.
    * - (9)
      - Implement the SQL for \ ``findOne``\  method.

        Specify alignment sequence of the Entity with 1:N relation, in ORDER BY clause.
        In the above example, sorting is done in the ascending order of PK.
    * - (10)
      - Implement SQL for \ ``findPage``\  method.
        Specify alignment sequence of Entity that has a 1:N relation with Order, in ORDER BY clause.
        In the above example, Order is sorted in descending sequence of PK (new sequence) and related Entity is sorted in the ascending order of PK.

 .. tip::

    When it is necessary to perform a pagination search while fetching related Entities with 1:N relation together by a single SQL,
    \ ``RowBounds``\  provided by MyBatis3 cannot be used.

    Following can be considered as alternative methods.

    * At first, a method is called which searches only main Entity and related Entities are then fetched by calling a separate method.
    * A virtual table that stores only main Entity within the page range is created by SQL
      and all the records necessary for mapping are fetched by performing JOIN on the records of virtual table (\ ``findPage``\  shown in the above example is implemented using this pattern)

    

|

If SQL (findPage) above is executed, following records are fetched.
The order records are 2. However, a total of 9 records are fetched since the records are joined with a related table that stores multiple records.


The breakdown is

* 1st to 3rd rows are records for generating \ ``Order``\  object with order ID \ ``2``\ 
* 4th to 9th rows are records for generating \ ``Order``\  object with order ID \ ``1``\ 



How to map search results (\ ``ResultSet``\ ) to JavaBean is explained below using a record wherein order ID is \ ``1``\ .



 .. figure:: images/dataaccess_sql_result.png
    :alt: Result Set of findPage
    :width: 100%
    :align: center

    **Picture - Result Set of findPage**

|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceMapping:

Implementation of mapping
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The definition for mapping the records described above in the \ ``Order``\  object and related Entity is given below.

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
    <mapper namespace="com.example.domain.repository.order.OrderRepository">

        <!-- ... -->

        <!-- (1) -->
        <resultMap id="orderResultMap" type="Order">
            <id property="id" column="id"/>
            <!-- (2) -->
            <result property="orderStatus.code" column="status_code" />
            <result property="orderStatus.name" column="status_name" />
            <!-- (3) -->
            <collection property="orderItems" ofType="OrderItem">
                <id property="orderId" column="id"/>
                <id property="item.code" column="item_code"/>
                <result property="quantity" column="quantity"/>
                <association property="item" resultMap="itemResultMap"/>
            </collection>
            <!-- (4) -->
            <collection property="orderCoupons" ofType="OrderCoupon"
                        notNullColumn="coupon_code">
                <id property="orderId" column="id"/>
                <!-- (5) -->
                <id property="coupon.code" column="coupon_code"/>
                <result property="coupon.name" column="coupon_name"/>
                <result property="coupon.price" column="coupon_price"/>
            </collection>
        </resultMap>

        <!-- (6) -->
        <resultMap id="itemResultMap" type="Item">
            <id property="code" column="item_code"/>
            <result property="name" column="item_name"/>
            <result property="price" column="item_price"/>
            <!-- (7) -->
            <collection property="categories" ofType="Category">
                <id property="code" column="category_code"/>
                <result property="name" column="category_name"/>
            </collection>
        </resultMap>

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Definition to map fetched records in \ ``Order``\  object.
        Perform mapping of related Entities (\ ``OrderStatus``\ , \ ``OrderItem``\  and \ ``OrderCoupon``\ ).
    * - (2)
      - Definition to map fetched records in \ ``OrderStatus``\  object.
    * - (3)
      - Definition to map fetched records in \ ``OrderItem``\  object.
        Mapping to related Entities (\ ``Item``\ ) is delegated to another \ ``resultMap``\ (6).
    * - (4)
      - Definition to map fetched records in \ ``OrderCoupon``\  object.
    * - (5)
      - Definition to map fetched records in \ ``Coupon``\  object.
    * - (6)
      - Definition to map fetched records in \ ``Item``\  object.
    * - (7)
      - Definition to map fetched records in \ ``Category``\ object.

|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceMappingOrder:

Mapping to Order object
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Perform mapping to \ ``Order``\  object.

 .. code-block:: xml
    :emphasize-lines: 1-4

    <!-- (1) -->
    <resultMap id="orderResultMap" type="Order">
        <!-- (2) -->
        <id property="id" column="id"/>
        <result property="orderStatus.code" column="status_code" />
        <result property="orderStatus.name" column="status_name" />
        <collection property="orderItems" ofType="OrderItem">
            <id property="orderId" column="id"/>
            <id property="item.code" column="item_code"/>
            <result property="quantity" column="quantity"/>
            <association property="item" resultMap="itemResultMap"/>
        </collection>
        <collection property="orderCoupons" ofType="OrderCoupon"
                    notNullColumn="coupon_code">
            <id property="orderId" column="id"/>
            <id property="coupon.code" column="coupon_code"/>
            <result property="coupon.name" column="coupon_name"/>
            <result property="coupon.price" column="coupon_price"/>
        </collection>
    </resultMap>

 .. figure:: images/dataaccess_resultmap_order.png
    :alt: ResultMap for Order
    :width: 100%
    :align: center

    **Picture - ResultMap for Order**


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Map search results to \ ``Order``\  object.

        Specify the class to be mapped in \ ``type``\  attribute.
    * - (2)
      - Value of \ ``id``\  column of fetched records is set in \ ``Order#id``\  property.

        Since \ ``id``\  column is PK, mapping is performed by using \ ``id``\  element.
        If \ ``id``\  element is used, records are grouped by the specified property values.
        Basically, they are grouped in two groups - \ ``id=1``\  and \ ``id=2``\.
        Two \ ``Order``\  objects are generated.


|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceMappingOrderStatus:

Mapping to OrderStatus object
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Perform mapping to \ ``OrderStatus``\  object.

 .. note::

    For creating Entity class of ":ref:`domainlayer_entity`",
    "Code table is handled as a basic type of java.lang.String rather than handling as an Entity.".
    This is because the data stored in the code table uses a different mechanism like ":doc:`../WebApplicationDetail/Codelist`" etc.


    The purpose of this section is to explain how to map to a related Entity (JavaBean), 
    hence the explanation on code table also being handled as an Entity is added.

    In the actual project, it is recommended to create an Entity based on the policies for creating an Entity class.


 .. code-block:: xml
    :emphasize-lines: 3-6

    <resultMap id="orderResultMap" type="Order">
        <id property="id" column="id"/>
        <!-- (1) -->
        <result property="orderStatus.code" column="status_code" />
        <!-- (2) -->
        <result property="orderStatus.name" column="status_name" />
        <collection property="orderItems" ofType="OrderItem">
            <id property="orderId" column="id"/>
            <id property="item.code" column="item_code"/>
            <result property="quantity" column="quantity"/>
            <association property="item" resultMap="itemResultMap"/>
        </collection>
        <collection property="orderCoupons" ofType="OrderCoupon"
                    notNullColumn="coupon_code">
            <id property="orderId" column="id"/>
            <id property="coupon.code" column="coupon_code"/>
            <result property="coupon.name" column="coupon_name"/>
            <result property="coupon.price" column="coupon_price"/>
        </collection>
    </resultMap>

 .. figure:: images/dataaccess_resultmap_orderstatus.png
    :alt: ResultMap for OrderStatus
    :width: 100%
    :align: center

    **Picture - ResultMap for OrderStatus**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Set the value of \ ``status_code``\  column of fetched records
        in \ ``OrderStatus#code``\  property.
        
    * - (2)
      - Set the value of \ ``status_name``\  column of fetched records
        in \ ``OrderStatus#name``\  property.

 .. note::

    The value of records that are grouped in \ ``id``\  column is set in \ ``OrderStatus``\  object.

|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceMappingOrderItem:

Mapping to OrderItem object
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Perform mapping to ``OrderItem``\  object.

 .. code-block:: xml
    :emphasize-lines: 5-15

    <resultMap id="orderResultMap" type="Order">
        <id property="id" column="id"/>
        <result property="orderStatus.code" column="status_code" />
        <result property="orderStatus.name" column="status_name" />
        <!-- (1) -->
        <collection property="orderItems" ofType="OrderItem">
            <!-- (2) -->
            <id property="orderId" column="id"/>
            <!-- (3) -->
            <id property="item.code" column="item_code"/>
            <!-- (4) -->
            <result property="quantity" column="quantity"/>
            <!-- (5) -->
            <association property="item" resultMap="itemResultMap"/>
        </collection>
        <collection property="orderCoupons" ofType="OrderCoupon"
                    notNullColumn="coupon_code">
            <id property="orderId" column="id"/>
            <id property="coupon.code" column="coupon_code"/>
            <result property="coupon.name" column="coupon_name"/>
            <result property="coupon.price" column="coupon_price"/>
        </collection>
    </resultMap>

 .. figure:: images/dataaccess_resultmap_orderitem.png
    :alt: ResultMap for OrderItem
    :width: 100%
    :align: center

    **Picture - ResultMap for OrderItem**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Map search results in \ ``OrderItem``\  object
        and add to \ ``Order#orderItems``\  property.

        When search results are to be mapped to the related Entities with 1:N relation, \ ``collection``\  element is used.
        Refer to "`MyBatis3 REFERENCE DOCUMENTATION(Mapper XML Files-collection-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#collection>`_\" 
        for details of "\ ``collection``\  element.
    * - (2)
      - Set the value of \ ``id``\  column of fetched records in \ ``OrderItem#orderId``\  property.

        Since \ ``id``\  column is PK, the mapping is performed by using \ ``id``\  element.
    * - (3)
      - Set the value of \ ``item_code``\  column of fetched records in \ ``Item#code``\  property.


        Since \ ``item_code``\ column is PK, the mapping is performed by using \ ``id``\  element.
        If \ ``id``\  element is used, the records are grouped in the specified property value.
        Basically, they are grouped in two groups - \ ``Item#code=ITM0000001``\  and \ ``Item#code=ITM0000002``\ .
        Two \ ``OrderItem``\  objects are generated.
    * - (4)
      - Set the value of \ ``quantity``\  column of fetched records in \ ``OrderItem#quantity``\  property.
    * - (5)
      - Generation of \ ``Item``\  object is delegated to a different \ ``resultMap``\ ,
        and the generated object is set in \ ``OrderItem#item``\  property.
        For actual mapping,
        refer to ":ref:`DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceMappingItem`" .
        
        When mapping is to be performed in the related Entities with a 1:1 relation, \ ``association``\  element is used.
        For details of \ ``association``\  element,
        refer to "`MyBatis3 REFERENCE DOCUMENTATION (Mapper XML Files-association-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#association>`_\" .
        

 .. note::

    In \ ``OrderItem``\  object,
    values of records that are grouped in \ ``id``\  column and \ ``item_code``\  column are set.

|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceMappingItem:

Mapping to Item object
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Perform mapping to \ ``Item``\  object.

 .. code-block:: xml
    :emphasize-lines: 1-8

    <!-- (1) -->
    <resultMap id="itemResultMap" type="Item">
        <!-- (2) -->
        <id property="code" column="item_code"/>
        <!-- (3) -->
        <result property="name" column="item_name"/>
        <!-- (4) -->
        <result property="price" column="item_price"/>
        <collection property="categories" ofType="Category">
            <id property="code" column="category_code"/>
            <result property="name" column="category_name"/>
        </collection>
    </resultMap>

 .. figure:: images/dataaccess_resultmap_item.png
    :alt: ResultMap for Item
    :width: 100%
    :align: center

    **Picture - ResultMap for Item**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Map search results to \ ``Item``\  object.

        Specify the class to be mapped in \ ``type``\  attribute.
    * - (2)
      - Set the value of \ ``item_code``\  column of fetched records in \ ``Item#code``\ .

        Since \ ``item_code``\  column is PK, mapping is performed by using \ ``id``\  element.
    * - (3)
      - Set the value of \ ``item_name``\  column of fetched records in \ ``Item#name``\ .
    * - (4)
      - Set the value of \ ``item_price``\  column of fetched records in \ ``Item#price``\ .

 .. note::

    In \ ``Item``\  object,
    values of records that are grouped in \ ``id``\  column and \ ``item_code``\  column are set.

|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceMappingCategory:

Mapping to Category object
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Perform mapping to \ ``Category``\  object.

 .. code-block:: xml
    :emphasize-lines: 5-11

    <resultMap id="itemResultMap" type="Item">
        <id property="code" column="item_code"/>
        <result property="name" column="item_name"/>
        <result property="price" column="item_price"/>
        <!-- (1) -->
        <collection property="categories" ofType="Category">
            <!-- (2) -->
            <id property="code" column="category_code"/>
            <!-- (3) -->
            <result property="name" column="category_name"/>
        </collection>
    </resultMap>


 .. figure:: images/dataaccess_resultmap_category.png
    :alt: ResultMap for Category
    :width: 100%
    :align: center

    **Picture - ResultMap for Category**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Map the search results to \ ``Category``\  object and add to \ ``Item#categories``\  property.

        When mapping to related Entities with 1:N relation, \ ``collection``\  element is used.
        For details of \ ``collection``\  element,
        refer to "`MyBatis3 REFERENCE DOCUMENTATION (Mapper XML Files-collection-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#collection>`_\" .
        
    * - (2)
      - Set the value of \ ``category_code``\  column of fetched records in \ ``Category#code``\ .

        Since \ ``category_code``\  column is  PK, the mapping is performed by using \ ``id``\  element.
        If \ ``id``\  element is used, the records are grouped by specified property values.

        Basically, following objects are generated

        * \ ``Category``\  object of \ ``Category#code=CTG0000001``\  as category of \ ``Item#code=ITM0000001``\ 

        * Two \ ``Category``\  objects of \ ``Category#code=CTG0000002``\  and \ ``Category#code=CTG0000003``\ as category of \ ``Item#code=ITM0000002``\ .

    * - (3)
      - Set the value of \ ``category_name``\  column of fetched records in \ ``Category#name`` \ .

 .. note::

    In \ ``Category``\  object, 
    values of records that are grouped in \ ``id``\  column, \ ``item_code``\  column and \ ``category_code``\  column are set.

|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceMappingOrderCoupon:

Mapping to OrderCoupon object
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Perform mapping to \ ``OrderCoupon``\  object.

 .. code-block:: xml
    :emphasize-lines: 11-16

    <resultMap id="orderResultMap" type="Order">
        <id property="id" column="id"/>
        <result property="orderStatus.code" column="status_code" />
        <result property="orderStatus.name" column="status_name" />
        <collection property="orderItems" ofType="OrderItem">
            <id property="orderId" column="id"/>
            <id property="item.code" column="item_code"/>
            <result property="quantity" column="quantity"/>
            <association property="item" resultMap="itemResultMap"/>
        </collection>
        <!-- (1) -->
        <collection property="orderCoupons" ofType="OrderCoupon" notNullColumn="coupon_code">
            <!-- (2) -->
            <id property="orderId" column="id"/>
            <!-- (3) -->
            <id property="coupon.code" column="coupon_code"/>
            <result property="coupon.name" column="coupon_name"/>
            <result property="coupon.price" column="coupon_price"/>
        </collection>
    </resultMap>

 .. figure:: images/dataaccess_resultmap_ordercoupon.png
    :alt: ResultMap for OrderCoupon
    :width: 100%
    :align: center

    **Picture - ResultMap for OrderCoupon**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Map the search results to \ ``OrderCoupon``\  object and add to \ ``Order#orderCoupons``\  property.

        When mapping to related Entities with 1:N relation, \ ``collection``\  element is used.
        For details of \ ``collection``\  element,
        refer to "`MyBatis3 REFERENCE DOCUMENTATION (Mapper XML Files-collection-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#collection>`_\" .
        

        It is important to note that \ ``notNullColumn``\  attribute is specified in the above example.

        This setting is to avoid the generation of \ ``OrderCoupon``\  object
        when no records exist in \ ``t_coupon``\  table.        
        In this implementation example, records of \ ``t_coupon``\  table are not stored
        in the data with \ ``id``\  column \ ``2``\ , hence from the search results, it is understood that the values of
        \ ``coupon_code``\ , \ ``coupon_name``\  and \ ``coupon_price``\  are \ ``null``\ .

        It is not necessary to specify \ ``notNullColumn``\  attribute if the columns that are mapped to \ ``OrderCoupon``\  object
        are restricted to the three columns described above.
        However, in the implementation example, since the value of \ ``id``\  column is mapped in \ ``OrderCoupon#orderId``\  property,
        \ ``notNullColumn``\  attribute should be specified.        
        This is because MyBatis generates an object when the value which is not \ ``null``\  is set in the column for mapping.
        
        As shown in the above example, if \ ``coupon_code``\  column is specified in the \ ``notNullColumn``\  attribute, the object is generated only when the value of
        \ ``coupon_code``\  column is not \ ``null``\ (in other words, when records exist).
        Multiple columns can be specified in \ ``notNullColumn``\ attribute.
    * - (2)
      - Set the value of \ ``id``\  column of fetched records in \ ``OrderCoupon#orderId``\  property.

        Since \ ``orderId``\  is PK, \ ``id``\  element is used.
    * - (3)
      - Set the value of \ ``coupon_code``\  column of fetched records in \ ``Coupon#code``\ .

        Since \ ``coupon_code``\  column is PK, the mapping is performed by using \ ``id``\  element.
        If \ ``id``\ element is used, the records are grouped by the specified property value.

        Basically, they are grouped in 2 groups - \ ``Coupon#code=CPN0000001``\  and \ ``Coupon#code=CPN0000002``\ .
        Then, two \ ``OrderCoupon``\  objects are generated.

|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceMappingCoupon:

Mapping to Coupon object
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''


Perform mapping to \ ``Coupon``\  object.

 .. code-block:: xml
    :emphasize-lines: 13-18

    <resultMap id="orderResultMap" type="Order">
        <id property="id" column="id"/>
        <result property="orderStatus.code" column="status_code" />
        <result property="orderStatus.name" column="status_name" />
        <collection property="orderItems" ofType="OrderItem">
            <id property="orderId" column="id"/>
            <id property="item.code" column="item_code"/>
            <result property="quantity" column="quantity"/>
            <association property="item" resultMap="itemResultMap"/>
        </collection>
        <collection property="orderCoupons" ofType="OrderCoupon" notNullColumn="coupon_code">
            <id property="orderId" column="id"/>
            <!-- (1) -->
            <id property="coupon.code" column="coupon_code"/>
            <!-- (2) -->
            <result property="coupon.name" column="coupon_name"/>
            <!-- (3) -->
            <result property="coupon.price" column="coupon_price"/>
        </collection>
    </resultMap>


 .. figure:: images/dataaccess_resultmap_coupon.png
    :alt: ResultMap for Coupon
    :width: 100%
    :align: center

    **Picture - ResultMap for Coupon**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Set the value of \ ``coupon_code``\  column of fetched records in \ ``Coupon#code``\ .
    * - (2)
      - Set the value of \ ``coupon_name``\  column of fetched records in \ ``Coupon#name``\ .
    * - (3)
      - Set the value of \ ``coupon_price``\  column of fetched records in \ ``Coupon#price``\ .

 .. note::

    In \ ``Coupon``\  object,
    values of records that are grouped in \ ``id``\  column and \ ``coupon_code``\  column are set.


|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceObject:

Object diagram after mapping
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

The status of \ ``Order``\  object and related Entities that are actually mapped is as given below.

 .. figure:: images/dataaccess_object.png
    :alt: Mapped object diagram
    :width: 90%
    :align: center

    **Picture - Mapped object diagram**


| Records and columns mapped to \ ``Order``\  object are as follows:
| By specifying groupBy attribute, grayed out part is merged with the non-grayed part.

 .. figure:: images/dataaccess_sql_result_used.png
    :alt: Valid Result Set
    :width: 100%
    :align: center

    **Picture - Valid Result Set**

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsWarningSqlMapping:

 .. warning::

     It should be noted that when the records with 1:N relationship are mapped using JOINs, fetching the data of grayed out part becomes unnecessary.

     If the same SQL is used in a process that does not use data from the N part, it results in fetching of unnecessary data. Therefore, two separate SQLs, one that fetches the N part and another that does not fetch the N part should be created.




|

.. _DataAccessMyBatis3AppendixNestedSelect:

How to fetch a related Entity using a nested SQL
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

MyBatis3 provides a method wherein a related Entity is fetched at the time of mapping by using a different SQL (nested SQL).

When a related Entity is fetched by using a nested SQL, following can be performed easily.

* Individual SQL definition
* Mapping definition of \ ``resultMap``\  element


 .. warning::

     It should be noted that even if the definitions are simplified, if nested SQL is used extensively,
     it can cause N+1.
     
     Default MyBatis3 behavior while using nested SQL is "Eager Load".
     It signifies that SQL is executed regardless of whether related Entities are used and the following risks are more likely to occur.

     * Unnecessary SQL execution and data fetching
     * N+1


 .. tip::

    MyBatis3 provides an option to change the behavior to "Lazy Load" while fetching a related Entity using a nested SQL.
    

    Refer to ":ref:`DataAccessMyBatis3AppendixNestedSelectLazySetting`" for how to use "Lazy Load".

|

.. _DataAccessMyBatis3AppendixNestedSelectMapping:

How to fetch a related Entity using a nested SQL
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Implementation example wherein a related Entity is fetched by using a nested SQL is shown below.

 .. code-block:: xml
    :emphasize-lines: 5-7

    <resultMap id="itemResultMap" type="Item">
        <id property="code" column="item_code"/>
        <result property="name" column="item_name"/>
        <result property="price" column="item_price"/>
        <!-- (1) -->
        <collection property="categories" column="item_code"
            select="findAllCategoryByItemCode" />
    </resultMap>

    <select id="findAllCategoryByItemCode"
        parameterType="string" resultType="Category">
        SELECT
            ct.code,
            ct.name
        FROM
            m_item_category ic
        INNER JOIN m_category ct ON ct.code = ic.category_code
        WHERE
            ic.item_code = #{itemCode}
        ORDER BY
            code
    </select>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Specify the statement ID of an SQL to be called in \ ``select``\  attribute of 
        \ ``association``\  element or \ ``collection``\  element.        

        Specify the column name that stores the parameter value to be passed to SQL, in \ ``column``\  attribute.
        In the above example, value of \ ``item_code``\  column is passed as a parameter of \ ``findAllCategoryByItemCode``\ .

        For details of attributes that can be specified,
        refer to "`MyBatis3 REFERENCE DOCUMENTATION(Mapper XML Files-Nested Select for Association-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#Nested_Select_for_Association>`_\" .
        

 .. note::

    In the above example, since \ ``fetchType``\  attribute is not specified, 
    whether to use "Lazy Load" or "Eager Load" depends on configuration of overall application.
    
    For configuration of overall application,
    refer to ":ref:`DataAccessMyBatis3AppendixNestedSelectLazySettingMyBatis`".

|

.. _DataAccessMyBatis3AppendixNestedSelectLazySetting:

Settings to apply Lazy Load for related Entity
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Although default behavior of MyBatis3 while fetching a related Entity by using a nested SQL is "Eager Load",
"Lazy Load" can also be used.

The minimum necessary settings for using "Lazy Load" and how to use "Lazy Load" are explained below.

For the configuration values that are not explained, 
refer to "`MyBatis3 REFERENCE DOCUMENTATION(Mapper XML Files-settings-) <http://mybatis.github.io/mybatis-3/configuration.html#settings>`_\" .

|

.. _DataAccessMyBatis3AppendixNestedSelectLazySettingLibrary:

Adding a Bytecode Manipulation Library
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

When "Lazy Load" is used, one of the libraries given below is necessary in order to generate a Proxy object for implementing "Lazy Load".

* JAVASSIST
* CGLIB


CGLIB had been used as default until MyBatis 3.2 series.
From  MyBatis 3.3.0, JAVASSIST has been used as default at the MyBatis 3.3.0 or later version, which is supported by terasoluna-gfw-mybatis3 5.2.x.RELEASE. 
In addition, it is possible to use "Lazy Load" without adding the library because the JAVASSIST is embedded in the MyBatis since MyBatis 3.3.0.

 .. note::

    When CGLIB is to be used in MyBatis 3.3.0 or later,

    * CGLIB artifact should be added to \ :file:`pom.xml`\
    * "\ ``proxyFactory=CGLIB``\" should be added to MyBatis configuration file (:file:`projectName-domain/src/main/resources/META-INF/mybatis/mybatis-config.xml`)


    Refer to "`MyBatis3 PROJECT DOCUMENTATION(Project Dependencies-compile-) <http://mybatis.github.io/mybatis-3/dependencies.html#compile>`_\"  for information about CGLIB artifact.
    

|

.. _DataAccessMyBatis3AppendixNestedSelectLazySettingMyBatis:

MyBatis settings for using Lazy Load
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

In MyBatis3, availability of "Lazy Load" can be specified in the 2 locations given below.

* Overall configuration of application (MyBatis configuration file)
* Individual configuration (mapping file)


* Overall configuration of application is specified in MyBatis configuration file (:file:`projectName-domain/src/main/resources/META-INF/mybatis/mybatis-config.xml`).
  

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
            PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
        <settings>
            <!-- (1) -->
            <setting name="lazyLoadingEnabled" value="true"/>
        </settings>
    </configuration>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Specify default behavior of application as \ ``lazyLoadingEnabled``\ .

        * \ ``true``\ : "Lazy Load"
        * \ ``false``\ : "Eager Load" (Default)

        When \ ``fetchType``\  attribute of \ ``association``\  element and \ ``collection``\  element is to be specified,
        the specified value of \ ``fetchType``\  attribute is given priority.

 .. warning::

    When \ ``select``\  attribute of \ ``association``\  element or \ ``collection``\  element is used in "\ ``false``\ : "Eager Load"" state, 
    exercise caution as SQL is executed at the time of mapping.


    It is recommended to set \ ``lazyLoadingEnabled``\  to \ ``true``\  unless there is a specific reason.

|

* Individual settings are specified in \ ``fetchType``\  attribute of \ ``association``\  element and \ ``collection``\  element of mapping file.
  

 .. code-block:: xml
    :emphasize-lines: 7

        <resultMap id="itemResultMap" type="Item">
            <id property="code" column="item_code"/>
            <result property="name" column="item_name"/>
            <result property="price" column="item_price"/>
            <!-- (2) -->
            <collection property="categories" column="item_code"
                fetchType="lazy"
                select="findAllCategoryByItemCode" />
        </resultMap>

        <select id="findAllCategoryByItemCode"
            parameterType="string" resultType="Category">
            SELECT
                ct.code,
                ct.name
            FROM
                m_item_category ic
            INNER JOIN m_category ct ON ct.code = ic.category_code
            WHERE
                ic.item_code = #{itemCode}
            ORDER BY
                code
        </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (2)
      - Specify \ ``lazy``\  or \ ``eager``\  in \ ``fetchType``\  attribute of \ ``association``\  element or \ ``collection``\  element.

        If \ ``fetchType``\  attribute is specified, overall configuration of application can be overwritten.

|

.. _DataAccessMyBatis3AppendixNestedSelectLazySettingTiming:

Settings for controlling execution timing of Lazy Load
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

MyBatis3 provides an option to control the timing in which "Lazy Load" is executed.

The settings to control the timing in which "Lazy Load" is executed is 
specified in MyBatis configuration file (:file:`projectName-domain/src/main/resources/META-INF/mybatis/mybatis-config.xml`).

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
            PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
        <settings>
            <!-- (1) -->
            <setting name="aggressiveLazyLoading" value="false"/>
        </settings>
    </configuration>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - Sr. No.
      - Description
    * - (1)
      - Specify the timing in which "Lazy Load" is executed in \ ``aggressiveLazyLoading``\ .

        * \ ``true``\ : Executed when getter method of the object that stores the property for which "Lazy Load" is to be performed, is called (default)
        * \ ``false``\ : Executed when getter method of the property for which "Lazy Load" is to be performed, is called


 .. warning::

    In case of "\ ``true``\" (default), exercise caution since SQL is likely to be executed for fetching the unused data.

    Basically, there is a case wherein the following mapping is carried out 
    and only the property for which "Lazy load" is not to be performed, is accessed.
    In case of "\ ``true``\" (default), "Lazy Load" is executed even without directly 
    accessing the property for which "Lazy Load" is to be performed.    

    It is recommended to set \ ``aggressiveLazyLoading``\  to "\ ``false``\  unless there is a specific reason.

    * Entity

      .. code-block:: java

        public class Item implements Serializable {
            private static final long serialVersionUID = 1L;
            private String code;
            private String name;
            private int price;
            private List<Category> categories;
            // ...
        }

    * Mapping file

      .. code-block:: xml

        <resultMap id="itemResultMap" type="Item">
            <id property="code" column="item_code"/>
            <result property="name" column="item_name"/>
            <result property="price" column="item_price"/>
            <collection property="categories" column="item_code"
                fetchType="lazy" select="findByItemCode" />
        </resultMap>

    * Application code (Service)

      .. code-block:: java
        :emphasize-lines: 2-3

            Item item = itemRepository.findOne(itemCode);
            // (2)
            String code = item.getCode();
            String name = item.getName();
            String price = item.getPrice();
            // ...
        }

      .. tabularcolumns:: |p{0.15\linewidth}|p{0.75\linewidth}|
      .. list-table::
        :header-rows: 1
        :widths: 15 75

        * - Sr. No.
          - Description
        * - (2)
          - In the above example, \ ``categories``\  property targeted for "Lazy Load" is not accessed.
            However, "Lazy Load" is executed while accessing \ ``Item#code``\  property.

            In case of "\ ``false``\" (default), "Lazy Load" is not executed in the cases described above.


.. raw:: latex

   \newpage

