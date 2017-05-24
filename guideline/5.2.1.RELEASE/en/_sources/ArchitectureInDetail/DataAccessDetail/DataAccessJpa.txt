Database Access (JPA)
================================================================================

.. only:: html

 .. contents:: Table of Contents
    :local:
    :depth: 3


.. todo::

    **TBD**

    The following topics in this chapter are currently under study.

    * | :file:`persistence.xml` settings

    * | Implementing dynamic query using QueryDSL

    * | Examples of cases wherein it is desirable to change the default setting values for specifying fetch method of related-entities.

    * | Using multiple PersistenceUnits

    * | Using Native query

Overview
--------------------------------------------------------------------------------

| This section explains the method to access the database using JPA.
| In this guideline, it is assumed that Hibernate is used as JPA provider and Spring Data JPA as JPA wrapper.

 .. figure:: images/dataaccess_jpa.png
    :alt: Target of description
    :width: 100%
    :align: center

    **Picture - Target of description**

.. warning:: 

  The contents described in this chapter may not be applicable for JPA providers other than Hibernate such as EclipseLink etc.

About JPA
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
JPA (Java Persistence API) defines the following as an API of Java: 
 1) a way of mapping the records in a relational database, with the java objects 
 2) a mechanism for reflecting the operations done on the java object, to the records in a relational database.

| JPA defines only specifications, it does not provide implementation.
| JPA implementation is provided as a reference implementation by the vendors developing O/R Mapper such as Hibernate.
| The reference implementation provided by the vendors developing O/R Mapper is called JPA Provider.

O/R Mapping of JPA
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Mapping of Java objects to the relational database records at the time of using JPA is as follows:

 .. figure:: images/dataaccess_jpa_mapping.png
    :alt: Image of O/R Mapping
    :width: 100%
    :align: center

    **Picture - Image of O/R Mapping**

| In JPA, if the value stored in the "managed" entity is changed (by calling setter method), there is a mechanism to reflect the changes in the relational database.
| This mechanism is quite similar to the client software such as Table viewer with Edit functionality.
| In client software such as Table viewer, if the value of viewer is changed, it is reflected in the database. While in JPA, if the value of Java object (JavaBean) called "Entity" is changed, it is reflected in the database.

Basic JPA terminology
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
The basic terminology of JPA is described below.


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 70
    :class: longtable

    * - Sr. No.
      - Term
      - Description
    * - 1.
      - | Entity class
      - | A Java class representing the records in the relational database
        | The class with \ ``@javax.persistence.Entity``\  annotation is an Entity class.
    * - 2.
      - | EntityManager
      - | An interface which provides API necessary for managing the life cycle of entity
        | Using methods of \ ``javax.persistence.EntityManager``\ , the application handles the relational database records as Java objects.
        | When using Spring Data JPA, this interface is usually not used directly; however, if it is necessary to generate a query that cannot be expressed using the Spring Data JPA mechanism, then this interface can be used to fetch the entity.
    * - 3.
      - | TypedQuery
      - | An interface which provides API for searching an entity
        | Using methods of \ ``javax.persistence.TypedQuery``\ , the application searches for the entity matching the specified conditions other than ID.
        | When using Spring Data JPA, this interface is usually not used directly; however, if it is necessary to generate a query that cannot be expressed using Spring Data JPA mechanism, then this interface can be used to search the entity.
        | The method for directly operating (updating or deleting) the entity of persistence layer (DB) matching the conditions, is also provided in this interface.
    * - 4.
      - | PersistenceContext
      - | Area for managing entities
        | The life cycle of an entity that is fetched or created through ``EntityManager`` is managed by storing that entity in this area. The entity managed in this area is referred to as "Managed Entity".
        | This area cannot be directly accessed from the application.
        | Apart from "Managed" entity, the other states of entity are "New", "Removed" and "Detached".
    * - 5.
      - | find method
      - | Method for fetching "managed" entity
        | If the entity corresponding to the ID does not exist in PersistenceContext, get (SELECT) the records stored in the relational database and create a "managed" entity.
    * - 6.
      - | persist method
      - | Method for setting "new" entity created in the application, to "managed" entity
        | It is a method of ``EntityManager`` and all the operations performed to INSERT the records in the relational database are accumulated in PersistenceContext (and later reflected to the database at the time of transaction commit or when the flush method of EntityManager is called).
    * - 7.
      - | merge method
      - | Method for setting the "detached" entity which is not managed in PersistenceContext, to "managed" entity.
        | It is a method of \ ``EntityManager``\  and the all operations performed to UPDATE the records stored in a relational database are accumulated in PersistenceContext (and later reflected to the database at the time of transaction commit or when the flush method of EntityManager is called).
        | However, if the record matching the ID does not exist in the relational database, then INSERT is executed instead of UPDATE.
    * - 8.
      - | remove method
      - | Method for setting the "managed" entity to "removed" entity
        | It is a method of \ ``EntityManager``\  and all the operations performed to DELETE the records stored in a relational database are accumulated in PersistenceContext.
    * - 9.
      - | flush method
      - | Method for forcibly reflecting the operations performed for the entity managed in PersistenceContext to the relational database.
        | It is a method of \ ``EntityManager``\  and the accumulated un-reflected operations are reflected in the relational database.
        | Normally, operations are reflected to the relational database only when a transaction is committed; however, flush method is used when the operation needs to be reflected in the database before committing a transaction.

 .. raw:: latex

    \newpage

Managing life cycle of entity
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
The life cycle of entity is managed as follows:

 .. figure:: images/dataaccess_jpa_lifecycle.png
    :alt: Life cycle of entity
    :width: 100%
    :align: center

    **Picture - Life cycle of entity**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - Sr. No.
      - Description
    * - | (1)
      - | When persist method of \ ``EntityManager``\  is called, the entity ("New" entity) passed as an argument is stored in PersistenceContext as "managed" entity.
    * - | (2)
      - | When find method of \ ``EntityManager``\  is called, a "managed" entity with ID passed as an argument, is returned.
        | If it does not exist in PersistenceContext, the records to be mapped are retrieved from the relational database by executing a query and stored as "managed" entity.
    * - | (3)
      - | When merge method of \ ``EntityManager``\  is called, the state of the entity ("detached") passed as an argument, is merged with "managed" entity.
        | If it does not exist in PersistenceContext, the records to be mapped are retrieved from the relational database by executing a query. The state of the entity passed as an argument is merged after the "managed" entity is stored.
        | Note that when this method is called, the entity passed as an argument does not necessarily get stored as "managed" entity unlike persist method.
    * - | (4)
      - | When remove method of \ ``EntityManager``\  is called, the "managed" entity passed as an argument becomes "removed" entity.
        | If this method is called, it is not possible to retrieve the "removed" entity.
    * - | (5)
      - | When flush method of \ ``EntityManager``\  is called, the operations of the entity accumulated using persist, merge and remove methods are reflected in the relational database.
        | By calling this method, the changes done for an entity are synchronized with the records of relational database.
        | However, the changes made only for the records of relational database are not synchronized with the entity.
        |
        | If the entity is searched by executing a query without using find method of \ ``EntityManager``\ , then prior to the search process, a process similar to flush method is executed in the internal logic of \ ``EntityManager``\  and the operations of the accumulated entity are reflected in the relational database.
        | For timing to reflect the persistence operations at the time of using Spring Data JPA,
        | refer to 
        |  :ref:`Reflection timing of persistence processing (1) <how_to_create_repository_extends_springdata_flush_timing_note1>`
        |  :ref:`Reflection timing of persistence processing (2) <how_to_create_repository_extends_springdata_flush_timing_note2>`

 .. raw:: latex

    \newpage

\

 .. note:: **About other life cycle management methods**

    The detach method, refresh method and clear method are available in \ ``EntityManager``\  to manage the entity life cycle. However,
    when using Spring Data JPA, there is no mechanism to call these methods using the default function, hence only their roles are described below.

    * detach method is used to set a "managed" entity to "detached" entity.
    * refresh method is used to update the "managed" entity as per the state of relational database.
    * clear method is used to delete the entity managed in PersistenceContext and the accumulated operations from the memory.

    clear method can be called by setting the clearAutomatically attribute of  \ ``@Modifying``\  annotation of Spring Data JPA to \ ``true``\ .
    For details, refer to \ :ref:`data-access-jpa_howtouse_querymethod_modifying`\ .

\

 .. note:: **About operations of "new" and "detached" entities**

    The operations performed on "new" and "detached" entities are not reflected in the relational database unless persist method or merge method is called.

About Spring Data JPA
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Spring Data JPA provides the library to create Repository using JPA.

| If Spring Data JPA is used, it is possible to retrieve an entity that matches the specified conditions only by defining the
| query method in the Repository interface; hence the amount of implementation for performing the entity operations can be reduced.
| However, only static query which can be expressed using annotation, can be defined in query method; hence
| it is necessary to implement custom Repository class for the query such as dynamic query which cannot be expressed using annotation.

The basic flow at the time of accessing the database using Spring Data JPA is shown below.

 .. figure:: images/dataaccess_jpa_basic_flow.png
    :alt: Basic flow of Spring Data JPA
    :width: 100%
    :align: center

    **Picture - Basic flow of Spring Data JPA**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - Sr. No.
      - Description
    * - | (1)
      - | Call the method of Repository interface from Service.
        | Entity object, Entity ID etc. are passed as method calling parameters. In the above example, entity is passed, however a primitive value can also be passed.
    * - | (2)
      - | Proxy class which dynamically implements Repository interface, delegates the process to \ ``org.springframework.data.jpa.repository.support.SimpleJpaRepository``\  or custom Repository class.
        | Parameters specified by Service are passed.
    * - | (3)
      - | Repository implementation class calls JPA APIs.
        | The parameters specified by Service and the parameters generated by implementation class of Repository are passed.
    * - | (4)
      - | Hibernate JPA reference implementation calls the Hibernate core APIs.
        | The parameters specified by implementation class of Repository and the parameters generated by Hibernate JPA reference implementation are passed.
    * - | (5)
      - | Hibernate Core API generates SQL and bind values from the specified parameters and passes to JDBC driver.
        | (API of java.sql.PreparedStatement is used for binding the actual values.)
    * - | (6)
      - | JDBC driver executes SQL.

 .. raw:: latex

    \newpage

| When creating the repository using Spring Data JPA, APIs of JPA need not be called directly; however, it is better to know which JPA method is being called by
| methods of Repository interface of Spring Data JPA.
| The JPA methods called by main methods of Repository interface of Spring Data JPA are shown below.

 .. figure:: images/dataaccess_jpa_api-mapping.png
    :alt: API Mapping of Spring Data JPA and JPA
    :width: 90%
    :align: center

    **Picture - API Mapping of Spring Data JPA and JPA**

|

How to use
--------------------------------------------------------------------------------

pom.xml settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
When using JPA (Spring Data JPA) in infrastructure layer, add the following dependency to pom.xml

 .. code-block:: xml

    <!-- (1) -->
    <dependency>
        <groupId>org.terasoluna.gfw</groupId>
        <artifactId>terasoluna-gfw-jpa-dependencies</artifactId>
        <type>pom</type>
    </dependency>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | \ ``terasoluna-gfw-jpa-dependencies``\  where the libraries associated with JPA are defined should be added to dependency.

.. note::  

   In the above setting example, since it is assumed that the dependent library version is managed by the parent project terasoluna-gfw-parent, specifying the version in pom.xml is  not necessary. 
   
Application Settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Datasource settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Set connection information of the database to datasource.
| For datasource settings, refer to \ :ref:`data-access-common_howtouse_datasource`\.

EntityManager settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Perform settings to use \ ``EntityManager``\ .

- xxx-infra.xml

 .. code-block:: xml

     <!-- (1) -->
     <bean id="jpaVendorAdapter"
         class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
         <!-- (2) -->
         <property name="showSql" value="false" />
         <!-- (3) -->
         <property name="database" value="POSTGRESQL" />
     </bean>

     <!-- (4) -->
     <bean id="entityManagerFactory"
         class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
         <!-- (5) -->
         <property name="packagesToScan" value="xxxxxx.yyyyyy.zzzzzz.domain.model" />
         <!-- (6) -->
         <property name="dataSource" ref="dataSource" />
         <!-- (7) -->
         <property name="jpaVendorAdapter" ref="jpaVendorAdapter" />
         <!-- (8) -->
         <property name="jpaPropertyMap">
             <util:map>
                 <entry key="hibernate.hbm2ddl.auto" value="" />
                 <entry key="hibernate.ejb.naming_strategy"
                     value="org.hibernate.cfg.ImprovedNamingStrategy" />
                 <entry key="hibernate.connection.charSet" value="UTF-8" />
                 <entry key="hibernate.show_sql" value="false" />
                 <entry key="hibernate.format_sql" value="false" />
                 <entry key="hibernate.use_sql_comments" value="true" />
                 <entry key="hibernate.jdbc.batch_size" value="30" />
                 <entry key="hibernate.jdbc.fetch_size" value="100" />
             </util:map>
         </property>
     </bean>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify the adapter class associated with JPA provider.
        | Hibernate will be used as JPA provider; hence specify \ ``org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter``\ .
    * - | (2)
      - Set the SQL output flag. In the example, "false: Do not output" has been specified.
    * - | (3)
      - | Set the value corresponding to RDBMS to be used. It is possible to specify the value defined in \ ``org.springframework.orm.jpa.vendor.Database``\  enumerator type.
        | In the example, "PostgreSQL" has been specified.
        | **[The value should be changed according to the database used in the project]**
        | If the database to be used changes with the environment, the value should be defined in properties file.
    * - | (4)
      - | Specify FactoryBean class to create ``javax.persistence.EntityManagerFactory`` instance.
        | Specify ``org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean`` .
    * - | (5)
      - | Specify the package where entity classes are kept.
        | The entity classes of the specified package can be managed using \ ``javax.persistence.EntityManager``\ .
        | **[The value should be changed to the relevant package name according to the project]**
    * - | (6)
      - | Specify the datasource to be used for accessing persistence layer (DB).
    * - | (7)
      - | Specify ``JpaVendorAdapter`` bean.
        | Specify the bean set in (1).
    * - | (8)
      - | Specify the settings to configure ``EntityManager`` of Hibernate.
        | For details, refer to "`Hibernate Reference Documentation <http://docs.jboss.org/hibernate/orm/4.3/manual/en-US/html/ch03.html#configuration-optional>`_\" .

 .. raw:: latex

    \newpage

\

 .. tip::

    When using the Oracle database, ANSI standard SQL JOIN for combining tables, can be used by specifying the following settings in \ ``jpaPropertyMap``\  mentioned in (8).

     .. code-block:: xml

         <bean id="entityManagerFactory"
             class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
             <!-- omitted -->
             <property name="jpaPropertyMap">
                 <util:map>
                     <!-- omitted -->
                     <entry key="hibernate.dialect"
                            value="org.hibernate.dialect.Oracle10gDialect" />  <!-- (9) -->
                 </util:map>
             </property>
         </bean>

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90
        :class: longtable

        * - Sr. No.
          - Description
        * - | (9)
          - | Specify \ ``org.hibernate.dialect.Oracle10gDialect``\  in \ ``"hibernate.dialect"``\ .
            | By specifying \ ``Oracle10gDialect``\ , ANSI standard SQL JOIN clause for combining the tables can be used.

| Perform the following settings when transaction manager (JTA) of the application server is to be used.
| The difference with the case wherein JTA is not used, is explained below.
| For other locations, same settings as the case wherein JTA is not used can be performed.

- xxx-infra.xml

 .. code-block:: xml

     <bean id="entityManagerFactory"
         class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">

         <!-- omitted -->

         <!-- (10) -->
         <property name="jtaDataSource" ref="dataSource" />

         <!-- omitted -->

         <property name="jpaPropertyMap">
             <util:map>

                 <!-- omitted -->

                 <!-- (11)  -->
                 <entry key="hibernate.transaction.jta.platform"
                     value="org.hibernate.service.jta.platform.internal.WeblogicJtaPlatform" />

             </util:map>
         </property>
     </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (10)
      - | Specify the datasource to be used for accessing persistence layer (DB).
        | When using JTA, specify the DataSource defined in application server in \ ``"jtaDataSource"``\  property and not in \ ``"dataSource"``\  property.
        | Refer to \ :ref:`data-access-common_howtouse_datasource`\  of common edition for the method to fetch DataSource defined in application server.
    * - | (11)
      - | Add JTA platform specification in ``"jpaPropertyMap"`` property.
        | The above example illustrates usage of Weblogic JTA.
        | The configurable value (platform) is FQCN of ``org.hibernate.service.jta.platform.spi.JtaPlatform`` implementation class.
        | The implementation class for main application servers is provided by Hibernate.

\

 .. note::

    When it is necessary to switch the transaction manager to be used as per the environment, then it is recommended that you define ``"entityManagerFactory"`` bean in :file:`xxx-env.xml` instead of :file:`xxx-infra.xml`.

    An example wherein it is necessary to change the transaction manager as per environment can be: use of application server without JTA function such as Tomcat in case of local environment,
    and use of application server with JTA function such as Weblogic in case of production environment as well as various test environments.

PlatformTransactionManager settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Perform the following settings when using local transaction.

- xxx-env.xml

 .. code-block:: xml

     <bean id="transactionManager"
         class="org.springframework.orm.jpa.JpaTransactionManager"> <!-- (1) -->
         <property name="entityManagerFactory" ref="entityManagerFactory" /> <!-- (2) -->
     </bean>

 .. tabularcolumns:: |p{0.06\linewidth}|p{0.94\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 6 94

    * - Sr. No.
      - Description
    * - | (1)
      - Specify ``org.springframework.orm.jpa.JpaTransactionManager``. This class controls transaction by calling APIs of JPA.
    * - | (2)
      - | Specify Factory of \ ``EntityManager``\  to be used in the transaction.

Perform the following settings when transaction manager (JTA) of the application server is to be used.

- xxx-env.xml

 .. code-block:: xml

     <tx:jta-transaction-manager /> <!-- (1) -->

 .. tabularcolumns:: |p{0.06\linewidth}|p{0.94\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 6 94

    * - Sr. No.
      - Description
    * - | (1)
      - The most appropriate \ ``org.springframework.transaction.jta.JtaTransactionManager``\  is defined as bean with id as "transactionManager", in the application server on which the application has been deployed.
        This class controls the transaction by calling JTA APIs.

persistence.xml settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| When using ``LocalContainerEntityManagerFactoryBean``, there are no mandatory settings to be performed in :file:`persistence.xml`.

\

 .. todo::

     **TBD**

    Currently, there are no mandatory settings to be performed in :file:`persistence.xml`; however such need may arise in future.

     When using EntityManagerFactory in application server of Java EE, it may be necessary to perform few settings in \ :file:`persistence.xml`\  ; 
     hence we are planning to provide maintenance for such settings in future.

Settings for validating Spring Data JPA
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

- xxx-infra.xml

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:jpa="http://www.springframework.org/schema/data/jpa"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation=".....
        http://www.springframework.org/schema/data/jpa
        http://www.springframework.org/schema/data/jpa/spring-jpa.xsd"> <!-- (1) -->

        <!-- ... -->

    </beans>

 .. code-block:: xml

     <jpa:repositories base-package="xxxxxx.yyyyyy.zzzzzz.domain.repository" /> <!-- (2) -->

 .. tabularcolumns:: |p{0.06\linewidth}|p{0.94\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 6 94

    * - Sr. No.
      - Description
    * - | (1)
      - | Import schema definition for Spring Data JPA configuration and assign (\ ``"jpa"``\ ) as namespace.
    * - | (2)
      - | Specify base package wherein Repository interface and custom Repository class are stored.
        | Interface inheriting ``org.springframework.data.repository.Repository`` and interface with \ ``org.springframework.data.repository.RepositoryDefinition``\  annotation are automatically defined as a bean of Repository class of Spring Data JPA.


- Attributes of <jpa:repositories> element
    | entity-manager-factory-ref, transaction-manager-ref, named-queries-location, query-lookup-strategy, factory-class and repository-impl-postfix are present as attributes.

 .. tabularcolumns:: |p{0.06\linewidth}|p{0.20\linewidth}|p{0.74\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 6 20 74
    :class: longtable

    * - Sr. No.
      - Element
      - Description
    * - 1.
      - entity-manager-factory-ref
      - | Specify Factory for generating ``EntityManager`` to be used in Repository.
        | If multiple Factories of ``EntityManager`` are to be created, then it is necessary to specify the bean to be used.
    * - 2.
      - transaction-manager-ref
      - | Specify ``PlatformTransactionManager`` to be used when the methods of Repository are called.
        | The bean registered with ``"transactionManager"`` bean name is used by default.
        | It needs to be specified when the bean name of ``PlatformTransactionManager``  to be used is not ``"transactionManager"``.
    * - 3.
      - named-queries-location
      - | Specify the location of Spring Data JPA properties file wherein Named Query is specified.
        | "classpath:META-INF/jpa-named-queries.properties" is used by default.
    * - 4.
      - query-lookup-strategy
      - | Specify the method to Lookup the query to be executed when query method is called.
        | By default, it is ``"CREATE_IF_NOT_FOUND"``. For details, refer to `Spring Data Commons - Reference Documentationの "Query lookup strategies" <http://docs.spring.io/spring-data/commons/docs/1.11.4.RELEASE/reference/html/#repositories.query-methods.query-lookup-strategies>`_\ . Use the default settings if there is no specific reason.
    * - 5.
      - factory-class
      - | Specify Factory for generating class to implement the process when the method of Repository interface is called.
        | ``org.springframework.data.jpa.repository.support.JpaRepositoryFactory`` is used by default. Specify the Factory created for changing default implementation of Spring Data JPA or for adding a new method.
        | For how to add a new method, refer to \ :ref:`custommethod_all-label`\ .
    * - 6.
      - repository-impl-postfix
      - | Specify suffix indicating that it is an implementation class of custom Repository.
        | By default, it is ``"Impl"``. For example: when Repository interface name is ``OrderRepository``, ``OrderRepositoryImpl`` will be the implementation class of custom Repository. Use the default settings if there is no specific reason.
        | For custom Repository, refer to ":ref:`custommethod_individual-label`".

 .. raw:: latex

    \newpage

Settings for using JPA annotations
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| To inject ``javax.persistence.EntityManagerFactory``  and  ``javax.persistence.EntityManager`` using the annotations (\ ``javax.persistence.PersistenceContext``\  and \ ``javax.persistence.PersistenceUnit``\ ) provided by JPA, ``org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor`` should be defined as bean.
| When ``<jpa:repositories>`` element is specified, bean is defined by default; hence no need to define it separately.

Settings for converting JPA exception to DataAccessException
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| To convert JPA exception to ``DataAccessException`` of Spring Framework, ``org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor`` should be defined as bean.
| When ``<jpa:repositories>`` element is specified, bean is defined by default; hence no need to define it separately.

OpenEntityManagerInViewInterceptor settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
To perform Lazy Fetching of Entity in application layer such as Controller and JSP etc., the lifetime of ``EntityManager`` should be extended till application layer
using \ ``org.springframework.orm.jpa.support.OpenEntityManagerInViewInterceptor``\ .

 .. figure:: images/dataaccess_jpa_entitymanager-lifetime-interceptor.png
    :alt: Lifetime of EntityManager on OpenEntityManagerInViewInterceptor
    :width: 80%
    :align: center

    **Picture - Lifetime of EntityManager on OpenEntityManagerInViewInterceptor**

When ``OpenEntityManagerInViewInterceptor`` is not to be used, the lifetime of ``EntityManager`` becomes same as that of transaction;
hence it is necessary to either fetch the data required in application layer as a process of Service class or to use Eager Fetch instead of Lazy Fetch.

 .. figure:: images/dataaccess_jpa_entitymanager-lifetime-default.png
    :alt: Default lifetime of EntityManager
    :width: 80%
    :align: center

    **Picture - Default Life time of EntityManager**


Considering the following perspectives, it is recommended that you use Lazy Fetch as fetch method and \ ``OpenEntityManagerInViewInterceptor``\ .

* Fetching as a process of Service class leads to insignificant implementation such as calling only getter method or accessing the collection fetched by calling getter method.
* When Eager Fetch is used, it is likely that the data which is not used in application layer is also fetched impacting the performance.

See the example of ``OpenEntityManagerInViewInterceptor`` settings below.

- spring-mvc.xml

 .. code-block:: xml

    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/**" /> <!-- (1) -->
            <mvc:exclude-mapping path="/resources/**" /> <!-- (1) -->
            <mvc:exclude-mapping path="/**/*.html" /> <!-- (1) -->
            <!-- (2) -->
            <bean
                class="org.springframework.orm.jpa.support.OpenEntityManagerInViewInterceptor" />
        </mvc:interceptor>
    </mvc:interceptors>

 .. tabularcolumns:: |p{0.06\linewidth}|p{0.94\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 6 94

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify the path for which interceptor is to be applied and path for which interceptor is not to be applied.
        | In this example, interceptor is being applied for paths other than paths of resource files (js, css, image etc.) and static web page (HTML).
    * - | (2)
      - | Specify ``org.springframework.orm.jpa.support.OpenEntityManagerInViewInterceptor``.


 .. note:: **Interceptor not to be applied to the path of static resources**

    It is recommended that interceptor not be applied to the path of static resources (js, css, image, html etc.), as there is no data access in such cases. 
    Application of interceptor to the path of static resources leads to execution of unnecessary processes (such as instance generation and close process).

|

| When Lazy Fetch is required in Servlet Filter, it is necessary to extend the lifetime of EntityManager till the Servlet Filter layer using ``org.springframework.orm.jpa.support.OpenEntityManagerInViewFilter``.
| For example, this case is applicable when ``org.springframework.security.core.userdetails.UserDetailsService`` of SpringSecurity is inherited and if Entity object is accessed in the inherited logic.
| However, if Lazy Fetch is not required, there is no need to extend the lifetime of ``EntityManager`` till the Servlet Filter layer.

 .. note:: **About Lazy Fetch in Servlet Filter layer**

    **It is recommended that you design and implement such that Lazy Fetch does not occur in Servlet Filter layer.**
    If ``OpenEntityManagerInViewInterceptor`` is used, it is possible to specify the applicable and non-applicable URL patterns; thus the path for which lifetime of "EntityManager" is to be extended till application layer can also be easily specified.
    For the data access required in Servlet Filter, the data should either be fetched in advance in Service class or should be loaded in advance using Eager Fetch; thereby, avoiding the occurrence of Lazy Fetch.

 .. figure:: images/dataaccess_jpa_entitymanager-lifetime-filter.png
    :alt: Lifetime of EntityManager on OpenEntityManagerInViewFilter
    :width: 80%
    :align: center

    **Picture - Lifetime of EntityManager on OpenEntityManagerInViewFilter**

|

See the example of ``OpenEntityManagerInViewFilter`` settings below.

- web.xml

 .. code-block:: xml

     <!-- (1) -->
     <filter>
         <filter-name>Spring OpenEntityManagerInViewFilter</filter-name>
         <filter-class>org.springframework.orm.jpa.support.OpenEntityManagerInViewFilter</filter-class>
     </filter>
     <!-- (2) -->
     <filter-mapping>
         <filter-name>Spring OpenEntityManagerInViewFilter</filter-name>
         <url-pattern>/*</url-pattern>
     </filter-mapping>

 .. tabularcolumns:: |p{0.06\linewidth}|p{0.94\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 6 94

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify ``org.springframework.orm.jpa.support.OpenEntityManagerInViewFilter``.
        | This Servlet Filter **needs to be defined before the Servlet Filter in which Lazy Fetch occurs.**
    * - | (2)
      - | Specify the pattern of the URL for which filter is to be applied. It is recommended that you apply the filter only to the required path; however, if the settings are complicated, you can also specify "/\*" (All Requests).

 .. note::

     If "/\*" (All Requests) are specified in the pattern of the URL for which ``OpenEntityManagerInViewFilter`` is to be applied, ``OpenEntityManagerInViewInterceptor`` settings are not required.

|

Creating Repository interface
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Spring Data provides the following 3 methods to create entity specific Repository interface.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table::
    :widths: 10 35 55
    :header-rows: 1

    * - Sr. No.
      - How to create
      - Description
    * - 1.
      - :ref:`how_to_create_repository_extends_springdata-label`
      - Create entity specific Repository interface by inheriting from the interface of Spring Data.
        **If there is no specific reason, then it is recommended that you create the entity specific Repository interface using this method.**
    * - 2.
      - :ref:`how_to_create_repository_extends_myinterface-label`
      - Out of all the methods of Repository interface of Spring Data, create a common project specific interface wherein only the required methods are specified. Inherit the common interface to create entity specific Repository interface.
    * - 3.
      - :ref:`how_to_create_repository_notextends-label`
      - Create entity specific Repository interface without inheriting the interface of Spring Data or common project specific common interface.

|

.. _how_to_create_repository_extends_springdata-label:

Inheriting the interface of Spring Data
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
The method to create entity specific Repository interface by inheriting from the interface of Spring Data is explained below.

Interfaces that can be inherited are as follows:

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table::
    :widths: 10 35 55
    :header-rows: 1

    * - Sr. No.
      - Interface
      - Description
    * - 1.
      - | org.springframework.data.repository
        | CrudRepository
      - Repository interface for generic CRUD operations.
    * - 2.
      - | org.springframework.data.repository
        | PagingAndSortingRepository
      - Repository interface wherein Pagination function and Sort function are added to findAll method of ``CrudRepository``.
    * - 3.
      - org.springframework.data.jpa.repository
        JpaRepository
      - | Repository interface that provides JPA specifications dependent methods.
        | ``PagingAndSortingRepository`` is inherited; hence methods of ``PagingAndSortingRepository`` and ``CrudRepository`` can also be used.
        | **If there is no specific reason, it is recommended that you create entity specific Repository interface by inheriting this interface.**

 .. note:: **About default implementation of Repository interface of Spring Data**

    The methods defined in the above interface are implemented using ``org.springframework.data.jpa.repository.support.SimpleJpaRepository``
    of Spring Data JPA.

|

The example is given below.

 .. code-block:: java

    public interface OrderRepository extends JpaRepository<Order, Integer> { // (1)

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Inherit ``JpaRepository`` and specify entity type in generic type ``<T>`` and entity ID type in generic type ``<ID extends Serializable>``.
        | In the above example, \ ``Order``\  type is specified in entity and \ ``Integer``\  type in entity ID.

|

If entity specific Repository interface is created by inheriting ``JpaRepository``, then the following methods can be implemented.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table::
    :widths: 10 35 55
    :header-rows: 1
    :class: longtable

    * - Sr. No.
      - Method
      - Description
    * - 1.
      - <S extends T> S save(S entity)
      - | Method to accumulate persistence operations (INSERT/UPDATE) for the specified entity in ``javax.persistence.EntityManager``.
        | If the value is not set in ID property (property with ``@javax.persistence.Id`` annotation or ``@javax.persistence.EmbeddedId`` annotation), ``persist`` method of ``EntityManager`` is called and when the value is set, ``merge`` method is called.
        | When merge method is called, please note that the returned Entity object is different from the Entity which is passed as an argument.
    * - 2.
      - <S extends T> List<S> save(Iterable<S> entities)
      - | Method to accumulate persistence operations for multiple specified entities in ``EntityManager``.
        | The method is implemented by calling ``<S extends T> S save(S entity)`` method repeatedly.
    * - 3.
      - T saveAndFlush(T entity)
      - | Once the persistence operations for the specified entity are accumulated in ``EntityManager``, this method reflects the accumulated persistence operations (INSERT/UPDATE/DELETE) in persistence layer (DB).
    * - 4.
      - void flush()
      - Method to execute persistence operations (INSERT/UPDATE/DELETE) for the entity accumulated in ``EntityManager`` in persistence layer (DB).
    * - 5.
      - void delete(ID id)
      - | Method to accumulate delete operation for the entity of specified ID, in ``EntityManager``.
        | This method calls ``T findOne(ID)`` method and converts the entity object to "managed" state under ``EntityManager`` and then deletes that object.
        | If entity is not present when ``T findOne(ID)`` method is called, ``org.springframework.dao.EmptyResultDataAccessException`` occurs.
    * - 6.
      - void delete(T entity)
      - | Method to accumulate delete operation for the specified entity, in ``EntityManager``.
    * - 7.
      - void delete(Iterable<? extends T> entities)
      - | Method to accumulate delete operations for the multiple specified entities, in ``EntityManager``.
        | This method is implemented by calling ``void delete(T entity)`` method repeatedly. In order to delete large number of entities, it is desirable to use ``void deleteInBatch(Iterable<T> entities)`` method.
    * - 8.
      - void deleteAll()
      - | Method to accumulate delete operations for all entities, in ``EntityManager``.
        | This method is implemented by repeatedly calling ``void delete(T entity)`` method for the entities fetched by ``List<T> findAll()`` method.
        | In order to delete large number of entities, ``void deleteAllInBatch()`` method should be used. This method loads all the entities to be deleted in the memory, thus causing memory exhaustion.
    * - 9.
      - void deleteInBatch(Iterable<T> entities)
      - | Method to directly delete multiple specified entities from persistence layer (DB).
        | When entities are deleted using this method, entities which are managed (cached) under ``EntityManager`` are not deleted. Hence if ``T findOne(ID id)`` method is called after the entity is deleted using this method, note that the entities managed in ``EntityManager`` will be returned.
        | For a deleted entity, in subsequent processing, if there is a possibility of calling methods like ``T findOne(ID id)`` (which return the Entity object managed (cached) under ``EntityManager``), the corresponding entity should be deleted using ``void delete(Iterable<? extends T> entities)`` method.
    * - 10.
      - void deleteAllInBatch()
      - | Method to directly delete all entities from persistence layer (DB).
        | Similar to "void deleteInBatch(Iterable<T> entities)" method, note that the entity managed (cached) under "EntityManager" is not deleted.
    * - 11.
      - T findOne(ID id)
      - | Method to fetch the entity of specified ID from persistence layer (DB).
        | The entity fetched from persistence layer is managed (cached) by ``EntityManager``; hence 2nd time onwards, persistence layer is not accessed and the cached entity is returned.
    * - 12.
      - List<T> findAll()
      - | Method to fetch all entities from persistence layer (DB).
        | The entity fetched from persistence layer is managed (cached) by ``EntityManager``.
    * - 13.
      - Iterable<T> findAll(Iterable<ID> ids)
      - | Method to fetch entities of multiple specified IDs, from persistence layer (DB).
        | The entities fetched from persistence layer are managed (cached) by ``EntityManager``. The specified IDs are searched using IN clause; hence be careful while using a DB where only limited number of values can be specified in IN clause such as Oracle.
    * - 14.
      - List<T> findAll(Sort sort)
      - | Method to fetch all entities from persistence layer (DB) in the specified sort order.
        | The entity fetched from persistence layer is managed (cached) using ``EntityManager``.
    * - 15.
      - Page<T> findAll(Pageable pageable)
      - | Method to fetch the entity matching the specified page (sort order, page number, number of records to be displayed on page) from persistence layer (DB).
        | The entity fetched from persistence layer is managed (cached) by ``EntityManager``.
    * - 16.
      - boolean exists(ID id)
      - | Method to check whether the entity of specified ID exists.
    * - 17.
      - long count()
      - | Returns the number of entities available.

 .. raw:: latex

    \newpage

 .. warning:: **Behavior when using optimistic locking (@javax.persistence.Version) of JPA**

     When the entity is updated or deleted at the time of using optimistic locking ( ``@Version`` ) of JPA, \ ``org.springframework.dao.OptimisticLockingFailureException``\  occurs.
     ``OptimisticLockingFailureException`` may occur in the following methods.

      * <S extends T> S save(S entity)
      * <S extends T> List<S> save(Iterable<S> entities)
      * T saveAndFlush(T entity)
      * void delete(ID id)
      * void delete(T entity)
      * void delete(Iterable<? extends T> entities)
      * void deleteAll()
      * void flush()

     For details on optimistic locking of JPA, refer to :doc:`ExclusionControl`.

.. _how_to_create_repository_extends_springdata_flush_timing_note1:

 .. note:: **Timing to reflect persistence operations (1)**

    For the entity managed under \ ``EntityManager``\ , accumulated persistence operations are executed just before committing a transaction and reflected in persistence layer (DB).
    Therefore, in order to handle errors such as unique constraint violation in transaction management (Service processing), it is necessary to call "saveAndFlush" method or "flush" method and execute persistence operations for the Entity accumulated in "EntityManager" forcibly.
    If only an error is to be notified to the client, it is OK to perform exception handling in Controller and set an appropriate message.

    \ ``saveAndFlush``\  and \ ``flush``\  are JPA dependent methods, hence do not use these methods if there is no specific purpose.

 - Normal flow

  .. figure:: images/dataaccess_jpa_persistence_flow_normal.png
    :alt: Normal sequence of persistence processing
    :width: 100%
    :align: center

    **Picture - Normal sequence of persistence processing**

 - flush flow

  .. figure:: images/dataaccess_jpa_persistence_flow_flush.png
    :alt: Sequence of persistence processing when flush method is used
    :width: 100%
    :align: center

    **Picture - Sequence of persistence processing when flush method is used**

.. _how_to_create_repository_extends_springdata_flush_timing_note2:

 .. note:: **Timing to reflect persistence operations (2)**

    When the following method is called, in order to avoid inconsistency between the data managed in \ ``EntityManager``\  and persistence layer (DB),
    the persistence operations of the entity accumulated in ``EntityManager`` are reflected in the persistence layer (DB) before the main process is carried out.

     * ``List<T> findAll`` method
     * ``boolean exists(ID id)``
     * ``long count()``

    In case of above methods, query is executed directly in the persistence layer (DB); hence inconsistency may occur unless the operations are reflected in the persistence layer (DB) before the main process is carried out.
    Calling of query methods described later also triggers the reflection of persistence operations for the entity accumulated in ``EntityManager``, in the persistence layer (DB).

 - Flow at the time of issuing queries

  .. figure:: images/dataaccess_jpa_persistence_flow_query.png
    :alt: Sequence of persistence processing when query method is used
    :width: 100%
    :align: center

    **Picture - Sequence of persistence processing when query method is used**

.. _how_to_create_repository_extends_myinterface-label:

Inheriting a common project specific interface in which only the required methods are defined
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Amongst the methods defined in interface of Spring Data, this section defines the method to create entity specific Repository interface by creating and
inheriting a common project specific interface in which only the required methods are defined.

The signature of methods should match with the methods of Repository interface of Spring Data; 

 .. note:: **Assumed cases**

    Amongst the methods of Repository interface of Spring Data, there are methods which are not used or which are not desirable to be used in the actual application.
    In order to remove such methods from Repository interface, refer below.
    The methods defined in interface are implemented using \ ``org.springframework.data.jpa.repository.support.SimpleJpaRepository``\  of Spring Data JPA.


See the example below.

 .. code-block:: java

    @NoRepositoryBean // (1)
    public interface MyProjectRepository<T, ID extends Serializable> extends
            Repository<T, ID> { // (2)

        T findOne(ID id); // (3)

        T save(T entity); // (3)

        // ...

    }

 .. code-block:: java

    public interface OrderRepository extends MyProjectRepository<Order, Integer> { // (4)

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify ``@NoRepositoryBean`` annotation to prevent the instantiation of ``Repository`` interfaces by Spring Data.
    * - | (2)
      - | Define a common interface for the project by inheriting ``org.springframework.data.repository.Repository``.
        | Use generic type since it is a not entity specific interface.
    * - | (3)
      - Select and define the required methods from the methods of Repository interface of Spring Data.
    * - | (4)
      - Inherit this common interface and specify the type of entity in generic type ``<T>`` and type of entity ID in generic type ``<ID extends Serializable>``. In this example, \ ``Order``\  type is specified in entity and \ ``Integer``\  type in entity ID.

.. _how_to_create_repository_notextends-label:

Not inheriting the interface
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
This section explains how to create entity specific Repository interface without inheriting any interface of Spring Data or common interface.

| Specify \ ``@org.springframework.data.repository.RepositoryDefinition``\  annotation as class annotation and specify entity type in domainClass attribute and entity ID type in idClass attribute.
| The methods which have the same signature as methods defined in Repository interface of Spring Data need not be implemented.

\

 .. note:: **Assumed cases**

    Repository can be created in this way when common entity operations are not required.
    The methods having same signature as methods defined in Repository interface of Spring Data are implemented using ``org.springframework.data.jpa.repository.support.SimpleJpaRepository`` provided by Spring Data JPA.


See the example below.

 .. code-block:: java

    @RepositoryDefinition(domainClass = Order.class, idClass = Integer.class) // (1)
    public interface OrderRepository { //(2)

        Order findOne(Integer id); // (3)

        Order save(Order entity); // (3)

        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify ``@RepositoryDefinition`` annotation.
        | In the example, \ ``Order``\  type is specified in domainClass attribute (entity type) and \ ``Integer``\  type in idClass attribute (entity ID type).
    * - | (2)
      - There is no need to inherit the interface (\ ``org.springframework.data.repository.Repository``\ ) of Spring Data.
    * - | (3)
      - Define the methods required for each entity.

.. _data-access-jpa_how_to_use_querymethod:

Adding query method
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| It is difficult to develop the actual application using only the Spring Data interface which is used for performing generic CRUD operations.
| Therefore Spring Data provides a mechanism to add "query methods" for performing any persistence operations (SELECT/UPDATE/DELETE) for the entity specific Repository interface.
| In the added query method, entity operations are performed using query language (JPQL or Native SQL).

 .. note:: **What is JPQL**

   JPQL is an abbreviation of "Java Persistence Query Language" and is the query language to perform entity operations (SELECT/UPDATE/DELETE) corresponding to the records of persistence layer (DB).
   The syntax is similar to SQL; however, JPQL operates the entities mapped to the records of persistence layer instead of operating these records directly.
   The entity operations are reflected to persistence layer (DB) using JPA provider (Hibernate).

   For details on JPQL, refer to `JSR 338: Java Persistence API, Version 2.1 Specification (PDF) "Chapter 4 Query Language" <http://download.oracle.com/otn-pub/jcp/persistence-2_1-fr-eval-spec/JavaPersistence.pdf>`_\ .

Defining query method
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Query method is defined as a method of entity specific Repository interface.

 .. code-block:: java

    public interface OrderRepository extends JpaRepository<Order, Integer> {
        List<Order> findByStatusCode(String statusCode);
    }

Specifying query to be executed
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Query to be executed should be specified at the time of calling query method.
| The method of specifying the query is as below. For details, refer to \ :ref:`how_to_specify_query-label`\ .

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :widths: 10 30 60
    :header-rows: 1

    * - Sr. No.
      - Method to specify a query
      - Description
    * - 1.
      - | :ref:`@Query annotation <how_to_specify_query_annotation-label>`
        | (Spring Data functionality)
      - | In the method to be added to entity specific Repository interface, specify ``@org.springframework.data.jpa.repository.Query`` annotation and the query to be executed.
        | **When there is no specific reason, it is recommended that you specify the query using this method.**
    * - 2.
      - | :ref:`Method name based on naming conventions <how_to_specify_query_mathodname-label>`
        | (Spring Data functionality)
      - | Specify the query to be executed by assigning a method name as per Spring Data naming conventions.
        | Query (JPQL) is generated from the method name using Spring Data JPA functionality. Only a SELECT clause of JPQL can be generated.
        | **For a simple query having few conditions, this method can be used instead of using @Query annotation.** However, for a complex query with many conditions, a simple method name indicating behavior should be used and Query should be specified using ``@Query`` annotation.
    * - 3.
      - | :ref:`Named query of properties file <how_to_specify_query_namedquery_properties-label>`
        | (Spring Data functionality)
      - | Specify the query in a properties file.
        | **The location of method definition (entity specific Repository interface) and location wherein the query is specified (properties file) are separated; hence this way of specifying the query is not recommended.**
        | **However, when using Native SQL as query, check whether it is necessary to define the database dependent SQL in the properties file.**
        | In case of applications for which any database can be selected or when the database changes (or is likely to be changed) depending on execution environment, then it is necessary to specify the Query using this method and manage the Properties file as environment dependent material.

 .. note:: **Using multiple query specification methods**

    Particularly, there is no restriction on using multiple query specification methods. Query specification methods and restriction on their concurrent usage should be determined in accordance with the project.

 .. note:: **Query Lookup methods**

    The operations would be as follows since the Spring Data default setting is ``CREATE_IF_NOT_FOUND``.

    #. Look for the query specified in ``@Query`` annotation.
    #. Look for the corresponding query from Named query.
    #. Create a query (JPQL) from method name and use it.
    #. An error occurs when query (JPQL) cannot be created from method name.

    For details on Query Lookup methods, refer to `Spring Data Commons - Reference Documentation "Defining query methods" -  "Query lookup strategies" <http://docs.spring.io/spring-data/commons/docs/1.11.4.RELEASE/reference/html/#repositories.query-methods.query-lookup-strategies>`_\ .

Fetching entity lock
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| To fetch entity lock, add ``@org.springframework.data.jpa.repository.Lock`` annotation to query method and specify the lock mode.
| For details, refer to :doc:`ExclusionControl`.

 .. code-block:: java

    @Query(value = "SELECT o FROM Order o WHERE o.status.code = :statusCode ORDER BY o.id DESC")
    @Lock(LockModeType.PESSIMISTIC_WRITE) // (1)
    List<Order> findByStatusCode(@Param("statusCode") String statusCode);

 .. code-block:: sql

    -- (2) statusCode='accepted'
    SELECT
            order0_.id AS id1_5_
            ,order0_.status_code AS status2_5_
        FROM
            t_order order0_
        WHERE
            order0_.status_code = 'accepted'
        ORDER BY
            order0_.id DESC
        FOR UPDATE


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify the lock mode in value attribute of ``@Lock`` annotation.
        | For the details on lock mode that can be specified, refer to `Java Platform, Enterprise Edition API Specification <http://docs.oracle.com/javaee/7/api/javax/persistence/LockModeType.html>`_\ .
    * - | (2)
      - | Native SQL converted from JPQL.(DB to be used is PostgreSQL)
        | In the example, ``LockModeType.PESSIMISTIC_WRITE`` has been specified; hence "FOR UPDATE" clause is added to SQL.


.. _data-access-jpa_howtouse_querymethod_modifying:

Operating the entities of Persistence Layer directly
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| It is recommended to perform update and delete operations on entity objects managed in ``EntityManager``.
| However, when entities need to be updated or deleted in a batch, check whether the entities of persistence layer (DB) are operated using query method.

 .. note:: **Reducing the causes of performance degradation**

    Operating the entities of persistence layer directly reduces the frequency of SQLs that would be required to be executed for operating these entities.
    Therefore, in case of applications that demand high performance, the causes of performance degradation can be reduced by operating the entities in batch using this method.
    Such SQLs are as follows:

    * SQL for loading all entity objects in \ ``EntityManager``\ . Need not be executed.
    * SQL for updating and deleting entity. This SQL was earlier required to be executed n times, but now it is sufficient to execute it only once.

 .. note::  **Standards for deciding whether to operate entities of persistence layer directly**

    When operating the entities of persistence layer directly, since there are certain points to be careful about from functionality point of view, **in case of applications which do not demand high performance,
    it is recommended that the batch operations must also be performed through the entity objects managed in EntityManager.**
    For the points to be careful, refer to the example below.

The example of directly operating the entities of persistence layer using query method is shown below.

 .. code-block:: java

    @Modifying // (1)
    @Query("UPDATE OrderItem oi SET oi.logicalDelete = true WHERE oi.id.orderId = :orderId ") // (2)
    int updateToLogicalDelete(@Param("orderId") Integer orderId); // (3)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify ``@org.springframework.data.jpa.repository.Modifying`` annotation indicating that the method is UPDATE query method.
        | If not specified, error will occur at the time of execution.
    * - | (2)
      - | Specify UPDATE or DELETE query.
    * - | (3)
      - | If update count or delete count is required, specify ``int`` or ``java.lang.Integer`` as return value and if count is not required, specify ``void``.

 .. warning:: **Consistency with entities managed in EntityManager**

    When entities of persistence layer are operated directly using query method, there is no change in the entities managed in EntityManager as per the default behavior of Spring Data JPA.
    Therefore, it should be noted that the entity object fetched immediately after calling ``JpaRepository#findOne(ID)`` method would be in a state prior to the state of operating the entities.

    This behavior can be avoided by setting the clearAutomatically attribute of ``@Modifying`` annotation to ``true``.
    When clearAutomatically attribute is set to ``true``, ``clear()`` method of ``EntityManager`` is called after operating the entities of persistence layer directly, and the entity objects managed in ``EntityManager`` and the accumulated persistence operations are deleted from ``EntityManager``.
    Therefore, if ``JpaRepository#findOne(ID)`` method is called immediately, the mechanism is such that the latest entity would be fetched from the persistence layer and ``EntityManager`` status would be synchronized with the persistence layer.

 .. warning:: **Points to be noted while using @Modifying(clearAutomatically = true)**

    By using ``@Modifying(clearAutomatically = true)``, it should be noted that the accumulated persistence operations (INSERT/UPDATE/DELETE) are also deleted from ``EntityManager``.
    Bugs may occur as the required persistence operations may not be reflected in the persistence layer.

    In order to avoid this problem, ``JpaRepository#saveAndFlush(T entity)`` or ``JpaRepository#flush()`` method should be called and the accumulated persistence operations should be reflected in the persistence layer before directly operating the entities of persistence layer.

Setting QueryHints
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
When it is necessary to set a hint in query, add ``@org.springframework.data.jpa.repository.QueryHints``  annotation to query method and
specify QueryHint ( ``@javax.persistence.QueryHint`` ) in value attribute.

 .. code-block:: java

    @Query(value = "SELECT o FROM Order o WHERE o.status.code = :statusCode ORDER BY o.id DESC")
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @QueryHints(value = { @QueryHint(name = "javax.persistence.lock.timeout", value = "0") }) // (1)
    List<Order> findByStatusCode(@Param("statusCode") String statusCode);

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify hint name in name attribute of ``@QueryHint`` annotation and hint value in value attribute.
        | In addition to the hint stipulated in JPA specifications, provider specific hint can be specified.
        | In the above example, lock timeout is set to ``0`` (DB used is PostgreSQL). "FOR UPDATE NOWAIT" clause is added to SQL.

 .. note:: **QueryHints that can be specified in Hibernate**

    QueryHints stipulated in JPA specifications are as follows:
    For details, refer to `JSR 338: Java Persistence API, Version 2.1 Specification (PDF) <http://download.oracle.com/otn-pub/jcp/persistence-2_1-fr-eval-spec/JavaPersistence.pdf>`_\ .

    * ``javax.persistence.query.timeout``
    * ``javax.persistence.lock.timeout``
    * ``javax.persistence.cache.retrieveMode``
    * ``javax.persistence.cache.storeMode``

    For Hibernate specific QueryHints, refer to "3.4.1.8. Query hints" of `Hibernate EntityManager User guide <http://docs.jboss.org/hibernate/entitymanager/3.6/reference/en/html/objectstate.html#d0e1109>`_\ .


.. _how_to_specify_query-label:

Specifying a query while calling a query method
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The method of specifying a query to be executed while calling query method is given below.

* :ref:`how_to_specify_query_annotation-label`
* :ref:`how_to_specify_query_mathodname-label`
* :ref:`how_to_specify_query_namedquery_properties-label`

.. _how_to_specify_query_annotation-label:

Specifying the query using @Query annotation
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Specify the query(JPQL) to be executed in value attribute of ``@Query`` annotation.

 .. code-block:: java

    @Query(value = "SELECT o FROM Order o WHERE o.status.code = :statusCode ORDER BY o.id DESC") // (1)
    List<Order> findByStatusCode(@Param("statusCode") String statusCode);

 .. code-block:: sql

    -- (2) statusCode='accepted'
    SELECT
            order0_.id AS id1_5_
            ,order0_.status_code AS status2_5_
        FROM
            t_order order0_
        WHERE
            order0_.status_code = 'accepted'
        ORDER BY
            order0_.id DESC

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify the query(JPQL) to be executed in value attribute of ``@Query`` annotation.
        | In the above example, query for fetching the ``Order`` object in the descending order of ``id`` property has been specified. Here, the ``code`` property (``String`` type) value of ``status`` property (``OrderStatus`` type) stored in ``Order`` object is matched with the specified parameter value ( ``statusCode`` ).
    * - | (2)
      - Native SQL converted from JPQL. Query(JPQL) specified in value attribute of ``@Query`` annotation is converted to Native SQL of database to be used.

 .. note:: **How to specify Native SQL directly instead of JPQL**

    Native SQL can be specified as query instead of JPQL by setting nativeQuery attribute to ``true``.
    **Fundamentally it is recommended that you use JPQL; however, when there is a need to generate the query that cannot be expressed in JPQL, Native SQL can be specified directly.**
    To specify the database dependent SQL, analyze whether it can be defined in the properties file.

    For method of defining SQL in properties file, refer to ":ref:`how_to_specify_query_namedquery_properties-label`".

 .. note:: **Named Parameters**

    Named parameter can be used by assigning a name to bind parameter of the query and using this assigned name to specify the value.
    To use Named Parameter, add ``@org.springframework.data.repository.query.Param`` annotation to the argument from which the value has to be used to bind to the named parameter in the query. Specify the assigned parameter name in value attribute of param annotation.
    ON the query side, at the position where parameter is to be bound in the query, specify it in the ":parametername"  format.

    **When there is no specific reason, It is recommended to use Named Parameters considering maintainability and readability of query.**

|

| In case of LIKE search, if the type of matching (Forward match, Backward match and Partial match) is fixed, ``"%"`` can be specified in JPQL.
| However, this is in extended Spring Data JPA format and not in standard JPQL, so it can be specified only in JPQL specified with ``@Query`` annotation.
| An error occurs if ``"%"``  is specified in JPQL specified as Named query.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|p{0.50\linewidth}|
 .. list-table::
    :widths: 10 20 20 50
    :header-rows: 1

    * - Sr. No.
      - Type of matching
      - Format
      - Specific example
    * - 1.
      - Forward match
      - | ``:parameterName%``
        | or
        | ``?n%``
      - | ``SELECT a FROM Account WHERE a.firstName LIKE :firstName%``
        | ``SELECT a FROM Account WHERE a.firstName LIKE ?1%``
    * - 2.
      - Backward match
      - | ``%:parameterName``
        | or
        | ``%?n``
      - | ``SELECT a FROM Account WHERE a.firstName LIKE %:firstName``
        | ``SELECT a FROM Account WHERE a.firstName LIKE %?1``
    * - 3.
      - Partial match
      - | ``%:parameterName%``
        | or
        | ``%?n%``
      - | ``SELECT a FROM Account WHERE a.firstName LIKE %:firstName%``
        | ``SELECT a FROM Account WHERE a.firstName LIKE %?1%``

 .. note:: **Escaping at the time of LIKE search**

    Search condition values should be escaped during LIKE search.

    The method for escaping these values is provided in ``org.terasoluna.gfw.common.query.QueryEscapeUtils`` class; this class can be used if it meets the requirements.
    For details on ``QueryEscapeUtils`` class, refer to ":ref:`data-access-common_appendix_like_escape`" of ":doc:`DataAccessCommon`".

 .. note:: **When the type of matching needs to be changed dynamically**

    When it is necessary to change the type of matching (Forward match, Backward match and Partial match) dynamically, 
    ``"%"`` should be added before and after the parameter value (same as conventional method), instead of specifying ``%`` in JPQL.

    The method for converting into search condition value corresponding to the type of matching is provided in ``org.terasoluna.gfw.common.query.QueryEscapeUtils`` class;
    this class can be used if it meets the requirements.
    For details on ``QueryEscapeUtils`` class, refer to ":ref:`data-access-common_appendix_like_escape`" of ":doc:`DataAccessCommon`".

| Sort conditions can be directly specified in query.
| See the example below.

 .. code-block:: sql

    // (1)
    @Query(value = "SELECT o FROM Order o WHERE o.status.code = :statusCode ORDER BY o.id DESC")
    Page<Order> findByStatusCode(@Param("statusCode") String statusCode, Pageable pageable);

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - Specify ``"ORDER BY"`` in query. Specify ``DESC`` for sorting in descending order and ``ASC`` for ascending order. By default it is "ASC", when nothing is specified.

|

| In addition to directly specifying the sort conditions in query, they can be specified in ``org.springframework.data.domain.Sort`` object stored in ``Pageable`` object.
| When specifying the sort conditions using this method, there is no need to specify countQuery attribute.
| Example of sorting  using ``Sort`` object stored in ``Pageable`` object is shown below.

- Controller

 .. code-block:: java

    @RequestMapping("list")
    public String list(@PageableDefault(
                            size=5,
                            sort = "id", // (1)
                            direction = Direction.DESC // (1)
                            ) Pageable pageable,
                              Model model) {
        Page<Order> orderPage = orderService.getOrders(pageable); // (2)
        model.addAttribute("orderPage", orderPage);
        return "order/list";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify the sort conditions. Sort conditions are set in ``Sort`` object that can be fetched by ``Pageable#getSort()`` method.
        | In the above example, DESC is specified as a sort condition for id field.
    * - | (2)
      - Specify ``Pageable`` object and call Service method.

|

- Service (Caller)

 .. code-block:: java

    public String getOrders(Pageable pageable){
        return orderRepository.findByStatusCode("accepted", pageable); // (3)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (3)
      - Call Repository method by specifying ``Pageable`` object passed by Controller.

|

- Repository interface

 .. code-block:: java

    @Query(value = "SELECT o FROM Order o WHERE o.status.code = :statusCode") // (4)
    Page<Order> findByStatusCode(@Param("statusCode") String statusCode, Pageable pageable);

 .. code-block:: sql

    -- (5) statusCode='accepted'
    SELECT
            COUNT(order0_.id) AS col_0_0_
        FROM
            t_order order0_
        WHERE
            order0_.status_code = 'accepted'

    -- (6) statusCode='accepted'
    SELECT
            order0_.id AS id1_5_
            ,order0_.status_code AS status2_5_
        FROM
            t_order order0_
        WHERE
            order0_.status_code = 'accepted'
        ORDER BY
            order0_.id DESC
        LIMIT 5

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (4)
      - Do not specify "ORDER BY" clause in query. No need to specify countQuery attribute also.
    * - | (5)
      - Native SQL for count converted from JPQL.
    * - | (6)
      - | Native SQL for fetching the entity of the specified page location converted from JPQL.
        | Not specified in query; however "ORDER BY" clause is added to the condition specified in ``Sort`` object stored in \ ``Pageable``\  object. In this example, it is SQL for PostgreSQL.

.. _how_to_specify_query_mathodname-label:

Specifying with the method name based on naming conventions
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Specify the query (JPQL) to be executed through method name as per the naming conventions of Spring Data JPA.
| JPQL is created from the method name by the functionality of Spring Data JPA.
| However, this is only possible for SELECT queries and not for UPDATE and DELETE.

For naming conventions for creating JPQL, refer to the following pages.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.45\linewidth}|p{0.45\linewidth}|
 .. list-table::
    :widths: 10 45 45
    :header-rows: 1

    * - Sr. No.
      - Reference page
      - Description
    * - 1.
      - `Spring Data Commons - "Query creation" of Reference Documentation "Defining query methods" <http://docs.spring.io/spring-data/commons/docs/1.11.4.RELEASE/reference/html/#repositories.query-methods.query-creation>`_\
      - This section describes method to specify Distinct, ORDER BY and Case insensitive.
    * - 2.
      - `Spring Data Commons - "Property expressions" of Reference Documentation "Defining query methods" <http://docs.spring.io/spring-data/commons/docs/1.11.4.RELEASE/reference/html/#repositories.query-methods.query-property-expressions>`_\
      - This section describes method to specify the nested entity property in condition.
    * - 3.
      - `Spring Data Commons - "Special parameter handling" of Reference Documentation "Defining query methods" <http://docs.spring.io/spring-data/commons/docs/1.11.4.RELEASE/reference/html/#repositories.special-parameters>`_\
      - This section describes special method arguments (``Pageable`` , ``Sort``).
    * - 4.
      - `Spring Data JPA - "Query creation" of Reference Documentation "Query methods" <http://docs.spring.io/spring-data/jpa/docs/1.9.4.RELEASE/reference/html/#jpa.query-methods.query-creation>`_\
      - This section describes naming conventions (keywords) for creating JPQL.
    * - 5.
      - `Spring Data Commons - Reference Documentation "Appendix C. Repository query keywords" <http://docs.spring.io/spring-data/commons/docs/1.11.4.RELEASE/reference/html/#repository-query-keywords>`_\
      - This section describes naming conventions (keywords) for creating JPQL.

See the example below.

- OrderRepositry.java

 .. code-block:: java

    Page<Order> findByStatusCode(String statusCode, Pageable pageable); // (1)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | When the method name matches with ``^(find|read|get).*By(.+)`` pattern, JPQL is created from method name.
        | In the ``(.+)`` portion, specify the property of entity which forms the query condition or keywords indicating the operation.
        | In the example, ``Order`` object where ``code`` property ( ``String`` type) value of ``status`` property ( ``OrderStatus`` type) stored in ``Order``  object is matched with the specified parameter value ( ``statusCode`` ), is being fetched in page format.

- Count Query

 .. code-block:: sql

    -- (2) JPQL
    SELECT
            COUNT(*)
        FROM
            ORDER AS generatedAlias0
                LEFT JOIN generatedAlias0.status AS generatedAlias1
            WHERE
                generatedAlias1.code = ?1

    -- (3) SQL statusCode='accepted'
    SELECT
            COUNT(*) AS col_0_0_
        FROM
            t_order order0_
                LEFT OUTER JOIN c_order_status orderstatu1_
                    ON order0_.status_code = orderstatu1_.code
        WHERE
            orderstatu1_.code = 'accepted'

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (2)
      - JPQL query for count created from method name.
    * - | (3)
      - Native SQL for count converted from JPQL of step (2).

- Query for fetching entities

 .. code-block:: sql

    -- (4) JPQL
    SELECT
            generatedAlias0
        FROM
            ORDER AS generatedAlias0
                LEFT JOIN generatedAlias0.status AS generatedAlias1
            WHERE
                generatedAlias1.code = ?1
            ORDER BY
                generatedAlias0.id DESC;

    -- (5) statusCode='accepted'
    SELECT
            order0_.id AS id1_5_
            ,order0_.status_code AS status2_5_
        FROM
            t_order order0_
                LEFT OUTER JOIN c_order_status orderstatu1_
                    ON order0_.status_code = orderstatu1_.code
        WHERE
            orderstatu1_.code = 'accepted'
        ORDER BY
            order0_.id DESC
        LIMIT 5

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (4)
      - JPQL query for fetching entities created from method name.
    * - | (5)
      - Native SQL for fetching the entities converted from JPQL of step (4).

.. _how_to_specify_query_namedquery_properties-label:

Specifying as Named query in Properties file
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Specify the query in the properties file (classpath:META-INF/jpa-named-queries.properties) of Spring Data JPA.

| **Consider using this method when it is required to write a database platform specific SQL at the time of using NativeQuery.**
| **Even if it is database platform specific SQL, it is recommended that you use a method to directly specify in @Query annotation when there is no dependency on the execution environment.**

- OrderRepositry.java

 .. code-block:: java

    @Query(nativeQuery = true)
    List<Order> findAllByStatusCode(@Param("statusCode") String statusCode); // (1)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - Regarding the lookup name of named query, class name of entity and method name linked by ``"."`` (dot) is used.
        In the above example, ``"Order.findAllByStatusCode"`` is used as Lookup name.

 .. tip:: **Specifying Lookup name of Named query**

    As per default behavior, Lookup name is contructed by connecting class name of the entity linked with method name using ``"."`` (dot). However, any name can be specified.

    * For fetching entities, specify it in name attribute of ``@Query`` annotation.
    * For count at the time of page search, specify it in countName attribute of ``@Query`` annotation.

 .. code-block:: java

    @Query(name = "OrderRepository.findAllByStatusCode", nativeQuery = true) // (2)
    List<Order> findAllByStatusCode(@Param("statusCode") String statusCode);

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (2)
      - In the above example, ``"OrderRepository.findAllByStatusCode"`` is specified as the Lookup name for the query.

|

- :file:`jpa-named-queries.properties`

 .. code-block:: properties

    # (3)
    Order.findAllByStatusCode=SELECT * FROM order WHERE status_code = :statusCode

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (3)
      - | Specify the SQL to be executed by using lookup name for the query as the key.
        | In the above example, the SQL to be executed has been specified by using ``"Order.findAllByStatusCode"`` as key.

 .. tip::

    The method of specifying Named Query in any properties file instead of the properties file of Spring Data JPA is explained below.

    - ``xxx-infra.xml``

     .. code-block:: xml

         <!-- (4) -->
         <jpa:repositories base-package="xxxxxx.yyyyyy.zzzzzz.domain.repository"
             named-queries-location="classpath:META-INF/jpa/jpa-named-queries.properties" />

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
         :widths: 10 90
         :header-rows: 1
         :class: longtable

         * - Sr. No.
           - Description
         * - | (4)
           - | Specify any properties file in named-queries-location attribute of <jpa:repositories> element.
             | In the above example, \ :file:`META-INF/jpa/jpa-named-queries.properties`\  on class path is used.


Implementing the process to search entities
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The method to search entities is explained below.

Searching all entities matching the conditions
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Call a query method to fetch all entities that match the conditions.

- Repository interface

 .. code-block:: java

    public interface AccountRepository extends JpaRepository<Account, String> {

        // (1)
        @Query("SELECT a FROM Account a WHERE :createdDateFrom <= a.createdDate AND a.createdDate < :createdDateTo ORDER BY a.createdDate DESC")
        List<Account> findByCreatedDate(
                @Param("createdDateFrom") Date createdDateFrom,
                @Param("createdDateTo") Date createdDateTo);

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Define a query method to return ``java.util.List`` interface.

- Service

 .. code-block:: java

    public List<Account> getAccounts(Date targetDate) {
        LocalDate targetLocalDate = new LocalDate(targetDate);
        Date fromDate = targetLocalDate.toDate();
        Date toDate = targetLocalDate.dayOfYear().addToCopy(1).toDate();

        // (2)
        List<Account> accounts = accountRepository.findByCreatedDate(fromDate,
                toDate);
        if (accounts.isEmpty()) { // (3)
            // ...
        }
        return accounts;
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (2)
      - | Call the query method defined in Repository interface.
    * - | (3)
      - | If the search result is 0 records, a blank list is returned. As null is not returned, null check is not required.
        | If needed, implement the process when the search result is 0 records.

|

.. _DataAccessJpaHowToUseFindPage:

Searching page of entities matching the conditions
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Amongst the entities matching the conditions, call a query method to fetch the entities of the specified page.

- Repository interface

 .. code-block:: java

    public interface AccountRepository extends JpaRepository<Account, String> {

        // (1)
        @Query("SELECT a FROM Account a WHERE :createdDateFrom <= a.createdDate AND a.createdDate < :createdDateTo")
        Page<Account> findByCreatedDate(
                @Param("createdDateFrom") Date createdDateFrom,
                @Param("createdDateTo") Date createdDateTo, Pageable pageable);

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Receive ``org.springframework.data.domain.Pageable`` interface as an argument and define query method for returning ``org.springframework.data.domain.Page`` interface.

- Controller

 .. code-block:: java

    @RequestMapping("list")
    public String list(@RequestParam("targetDate") Date targetDate,
                       @PageableDefault(
                           page = 0,
                           value = 5,
                           sort = { "createdDate" },
                           direction = Direction.DESC)
                           Pageable pageable, // (2)
                       Model model) {
        Page<Order> accountPage = accountService.getAccounts(targetDate, pageable);
        model.addAttribute("accountPage", accountPage);
        return "account/list";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (2)
      - | Create object (``org.springframework.data.domain.Pageable``) for paging search provided by Spring Data.
        | For details, refer to ":doc:`../WebApplicationDetail/Pagination`".


- Service

 .. code-block:: java

    public Page<Account> getAccounts(Date targetDate ,Pageable pageable) {

        LocalDate targetLocalDate = new LocalDate(targetDate);
        Date fromDate = targetLocalDate.toDate();
        Date toDate = targetLocalDate.dayOfYear().addToCopy(1).toDate();

        // (3)
        Page<Account> page = accountRepository.findByCreatedDate(fromDate,
                toDate, pageable);
        if (!page.hasContent()) { // (4)
            // ...
        }
        return page;
    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (3)
      - | Call the query method defined in Repository interface.
    * - | (4)
      - | If the search result is 0 records, a blank list will be set in ``Page``  object and ``false``  will be returned for ``Page#hasContent()`` method.
        | If needed, implement the process when the search result is 0 records.


|

Implementing search process as per the dynamic conditions of entities
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
To add query method to Repository for searching the entities as per dynamic conditions, search process should be implemented by creating
custom Repository interface and custom Repository class for the entity specific Repository interface.
For method of creating custom Repository interface and custom Repository class, refer to ":ref:`custommethod_individual-label`".

See the description below to search entities by applying dynamic conditions.

.. todo::

    **TBD**

    Following contents will be added in future.

    * Example illustrating implementation of dynamic query using QueryDSL.

|

Searching all entities matching the dynamic conditions
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Implement and call the query method for fetching all entities matching the dynamic conditions.

See the example below.

Here, the conditions below are specified as dynamic conditions.

* Order ID
* Product name
* Order status (multiple statuses can be specified)

Further, the search is narrowed down using AND operator for the orders matching the specified conditions.
If no condition is specified, a blank list will be returned.

- Criteria (JavaBean)

 .. code-block:: java

    public class OrderCriteria implements Serializable { // (1)

        private Integer id;

        private String itemName;

        private List<String> statusCodes;

        // ...

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Create a Criteria object (JavaBean) storing the search conditions.

- Custom Repository interface

 .. code-block:: java

    public interface OrderRepositoryCustom {

        Page<Order> findAllByCriteria(OrderCriteria criteria); // (2)

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (2)
      - | Define a method in custom Repository interface which receives Criteria object as an argument and returns List of Order objects.

- Custom Repository class

 .. code-block:: java

    public class OrderRepositoryImpl implements OrderRepositoryCustom { // (3)

        @PersistenceContext
        EntityManager entityManager; // (4)

        public List<Order> findAllByCriteria(OrderCriteria criteria) { // (5)

            // Collect dynamic conditions.
            // (6)
            final List<String> andConditions = new ArrayList<String>();
            final List<String> joinConditions = new ArrayList<String>();
            final Map<String, Object> bindParameters = new HashMap<String, Object>();

            // (7)
            if (criteria.getId() != null) {
                andConditions.add("o.id = :id");
                bindParameters.put("id", criteria.getId());
            }
            if (!CollectionUtils.isEmpty(criteria.getStatusCodes())) {
                andConditions.add("o.status.code IN :statusCodes");
                bindParameters.put("statusCodes", criteria.getStatusCodes());
            }
            if (StringUtils.hasLength(criteria.getItemName())) {
                joinConditions.add("o.orderItems oi");
                joinConditions.add("oi.item i");
                andConditions.add("i.name LIKE :itemName ESCAPE '~'");
                bindParameters.put("itemName", QueryEscapeUtils
                        .toLikeCondition(criteria.getItemName()));
            }

            // (8)
            if (andConditions.isEmpty()) {
                return Collections.emptyList();
            }

            // (9)
            // Create dynamic query.
            final StringBuilder queryString = new StringBuilder();

            // (10)
            queryString.append("SELECT o FROM Order o");

            // (11)
            // add join conditions.
            for (String joinCondition : joinConditions) {
                queryString.append(" LEFT JOIN ").append(joinCondition);
            }
            // add conditions.
            Iterator<String> andConditionsIt = andConditions.iterator();
            if (andConditionsIt.hasNext()) {
                queryString.append(" WHERE ").append(andConditionsIt.next());
            }
            while (andConditionsIt.hasNext()) {
                queryString.append(" AND ").append(andConditionsIt.next());
            }

            // (12)
            // add order by condition.
            queryString.append(" ORDER BY o.id");

            // (13)
            // Create typed query.
            final TypedQuery<Order> findQuery = entityManager.createQuery(
                    queryString.toString(), Order.class);
            // Bind parameters.
            for (Map.Entry<String, Object> bindParameter : bindParameters
                    .entrySet()) {
                findQuery.setParameter(bindParameter.getKey(), bindParameter
                        .getValue());
            }

            // (14)
            // Execute query.
            return findQuery.getResultList();

        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1
    :class: longtable

    * - Sr. No.
      - Description
    * - | (3)
      - | Create implementation class of custom Repository interface.
    * - | (4)
      - | Inject ``EntityManager``.
        | Inject using ``@javax.persistence.PersistenceContext`` annotation.
    * - | (5)
      - | Implement the query method to fetch all the entities matching the dynamic conditions.
        | In the above example, the method is not split for explanation purpose; however it can be split if required.
    * - | (6)
      - | Define the variables (list for AND condition, list for join condition, bind parameter map) to build dynamic queries.
        | Set the required information in the variables for the items for which conditions are specified in OrderCriteria object.
    * - | (7)
      - | Determine whether conditions are specified in OrderCriteria object and set the required information for building dynamic queries.
        | In the above example, the information is set to fetch the items wherein ``id`` is completely matched with the specified value, ``statusCodes`` is included in the specified list, and ``itemName`` satisfying forward match with the specified value.
        | For ``itemName``, the related-entities having the values to be compared have a complex nested relation; hence these entities should be joined.
    * - | (8)
      - | In the above example, when conditions are not specified, return a blank list as there is no need to perform search.
    * - | (9)
      - | When conditions are specified, build the query for searching entities.
        | In the above example, query to be executed is built using ``java.lang.StringBuilder`` class.
    * - | (10)
      - | Build static query elements.
        | In the above example, SELECT clause and FROM clause are built as static query elements.
    * - | (11)
      - | Build dynamic query elements.
        | In the above example, conditions to be set in join conditions list (JOIN clause) and AND conditions list (WHERE clause) are built as dynamic query elements.
    * - | (12)
      - | Build the static query elements.
        | In the above example, ORDER BY clause is built as static query elements.
    * - | (13)
      - | Convert the dynamically built query string into ``javax.persistence.TypedQuery``, and set the bind parameters required for executing the query.
    * - | (14)
      - | Execute the dynamically built query and fetch all the entities matching the conditions.

 .. raw:: latex

    \newpage

-  Entity specific Repository interface

 .. code-block:: java

    public interface OrderRepository extends JpaRepository<Order, Integer>,
                                    OrderRepositoryCustom { // (15)
        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (15)
      - | Inherit the custom Repository interface in entity specific Repository interface.


- Service (Caller)

 .. code-block:: java

    // condition values for sample.
    Integer conditionValueOfId = 4;
    List<String> conditionValueOfStatusCodes = Arrays.asList("accepted");
    String conditionValueOfItemName = "Wat";

    // implementation of sample.
    // (16)
    OrderCriteria criteria = new OrderCriteria();
    criteria.setId(conditionValueOfId);
    criteria.setStatusCodes(conditionValueOfStatusCodes);
    criteria.setItemName(conditionValueOfItemName);
    List<Order> orders = orderRepository.findAllByCriteria(criteria); // (17)
    if (orders.isEmpty()) { // (18)
        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (16)
      - | Specify the search conditions in OrderCriteria object.
    * - | (17)
      - | Use OrderCriteria as an argument and call the query method to get all the entities matching the dynamic conditions.
    * - | (18)
      - | Analyze the search results and perform the processing required in case of 0 records.

- Executed JPQL(SQL)

 .. code-block:: sql

    -- (19)
    --   conditionValueOfId=4
    --   conditionValueOfStatusCodes = ["accepted"]
    --   conditionValueOfItemName = "Wat"

    -- JPQL
    SELECT
            o
        FROM
            ORDER o
                JOIN o.orderItems oi
                    JOIN oi.item i
                WHERE
                    o.id = :id
                    AND o.status.code IN :statusCodes
                    AND i.name LIKE :itemName ESCAPE '~'
                ORDER BY
                    o.id

    -- SQL
    SELECT
            order0_.id AS id1_6_
            ,order0_.created_by AS created2_6_
            ,order0_.created_date AS created3_6_
            ,order0_.last_modified_by AS last4_6_
            ,order0_.last_modified_date AS last5_6_
            ,order0_.status_code AS status6_6_
        FROM
            t_order order0_ INNER JOIN t_order_item orderitems1_
                ON order0_.id = orderitems1_.order_id INNER JOIN m_item item2_
                ON orderitems1_.item_code = item2_.code
    WHERE
        order0_.id = 4
        AND (
            order0_.status_code IN ('accepted')
        )
        AND (
            item2_.name LIKE 'Wat%' ESCAPE '~'
        )
    ORDER BY
        order0_.id

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (19)
      - | Example of generated JPQL and SQL when all the conditions are specified.

 .. code-block:: sql

    -- (20)
    --   conditionValueOfId=4
    --   conditionValueOfStatusCodes = ["accepted"]
    --   conditionValueOfItemName = ""
    -- JPQL
    SELECT
            o
        FROM
            ORDER o
        WHERE
            o.id = :id
            AND o.status.code IN :statusCodes
        ORDER BY
            o.id

    -- SQL
    SELECT
            order0_.id AS id1_6_
            ,order0_.created_by AS created2_6_
            ,order0_.created_date AS created3_6_
            ,order0_.last_modified_by AS last4_6_
            ,order0_.last_modified_date AS last5_6_
            ,order0_.status_code AS status6_6_
        FROM
            t_order order0_
        WHERE
            order0_.id = 4
            AND (
                order0_.status_code IN ('accepted')
            )
        ORDER BY
            order0_.id;

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (20)
      - | Example of generated JPQL and SQL when conditions other than ``iteName`` are specified.

 .. code-block:: sql

    -- (21)
    --   conditionValueOfId=4
    --   conditionValueOfStatusCodes = []
    --   conditionValueOfItemName = ""
    -- JPQL
    SELECT
            o
        FROM
            ORDER o
        WHERE
            o.id = :id
        ORDER BY
            o.id

    -- SQL
    SELECT
            order0_.id AS id1_6_
            ,order0_.created_by AS created2_6_
            ,order0_.created_date AS created3_6_
            ,order0_.last_modified_by AS last4_6_
            ,order0_.last_modified_date AS last5_6_
            ,order0_.status_code AS status6_6_
        FROM
            t_order order0_
        WHERE
            order0_.id = 4
        ORDER BY
            order0_.id;

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (21)
      - | Example of generated JPQL and SQL when only ``id`` is specified.


|

Page search for the entities matching the dynamic conditions
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Implement and call the query method to fetch the entities corresponding to the specified page, amongst the entities matching the dynamic conditions.

As shown in the example below, the specification is same as that for normal search except for fetching the corresponding page.
Further, the description for fetching all records is omitted.

- Custom Repository interface

 .. code-block:: java

    public interface OrderRepositoryCustom {

        Page<Order> findPageByCriteria(OrderCriteria criteria, Pageable pageable); // (1)

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Define the query method to fetch the entities corresponding to the specified page, amongst the entities matching the dynamic conditions.


- Custom Repository class

 .. code-block:: java

    public class OrderRepositoryCustomImpl implements OrderRepositoryCustom {

        @PersistenceContext
        EntityManager entityManager;

        public Page<Order> findPageByCriteria(OrderCriteria criteria,
                Pageable pageable) { // (2)

            // collect dynamic conditions.
            final List<String> andConditions = new ArrayList<String>();
            final List<String> joinConditions = new ArrayList<String>();
            final Map<String, Object> bindParameters = new HashMap<String, Object>();

            if (criteria.getId() != null) {
                andConditions.add("o.id = :id");
                bindParameters.put("id", criteria.getId());
            }
            if (!CollectionUtils.isEmpty(criteria.getStatusCodes())) {
                andConditions.add("o.status.code IN :statusCodes");
                bindParameters.put("statusCodes", criteria.getStatusCodes());
            }
            if (StringUtils.hasLength(criteria.getItemName())) {
                joinConditions.add("o.orderItems oi");
                joinConditions.add("oi.item i");
                andConditions.add("i.name LIKE :itemName ESCAPE '~'");
                bindParameters.put("itemName", QueryEscapeUtils.toLikeCondition(criteria
                        .getItemName()));
            }

            if (andConditions.isEmpty()) {
                List<Order> orders = Collections.emptyList();
                return new PageImpl<Order>(orders, pageable, 0); // (3)
            }

            // create dynamic query.
            final StringBuilder queryString = new StringBuilder();
            final StringBuilder countQueryString = new StringBuilder(); // (4)
            final StringBuilder conditionsString = new StringBuilder(); // (4)

            queryString.append("SELECT o FROM Order o");
            countQueryString.append("SELECT COUNT(o) FROM Order o"); // (5)

            // add join conditions.
            for (String joinCondition : joinConditions) {
                conditionsString.append(" JOIN ").append(joinCondition);
            }

            // add conditions.
            Iterator<String> andConditionsIt = andConditions.iterator();
            if (andConditionsIt.hasNext()) {
                conditionsString.append(" WHERE ").append(andConditionsIt.next());
            }
            while (andConditionsIt.hasNext()) {
                conditionsString.append(" AND ").append(andConditionsIt.next());
            }
            queryString.append(conditionsString); // (6)
            countQueryString.append(conditionsString); // (6)

            // add order by condition.
            // (7)
            String orderByString = QueryUtils.applySorting("", pageable.getSort(), "o");
            queryString.append(orderByString);

            // create typed query.
            final TypedQuery<Long> countQuery = entityManager.createQuery(
                    countQueryString.toString(), Long.class); // (8)

            final TypedQuery<Order> findQuery = entityManager.createQuery(
                    queryString.toString(), Order.class);

            // bind parameters.
            for (Map.Entry<String, Object> bindParameter : bindParameters
                    .entrySet()) {
                countQuery.setParameter(bindParameter.getKey(), bindParameter
                        .getValue()); // (8)
                findQuery.setParameter(bindParameter.getKey(), bindParameter
                        .getValue());
            }

            long total = countQuery.getSingleResult().longValue(); // (9)
            List<Order> orders = null;
            if (total != 0) { // (10)
                findQuery.setFirstResult(pageable.getOffset());
                findQuery.setMaxResults(pageable.getPageSize());
                // execute query.
                orders = findQuery.getResultList();
            } else { // (11)
                orders = Collections.emptyList();
            }

            return new PageImpl<Order>(orders, pageable, total); // (12)
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1
    :class: longtable

    * - Sr. No.
      - Description
    * - | (2)
      - | Implement the query method to fetch the entities corresponding to the specified page, amongst the entities matching the dynamic conditions.
        | In the above example, the method is not split for explanation purpose; however it can be split if required.
    * - | (3)
      - | In the above example, when conditions are not specified, return a blank page information as there is no need to perform search.
    * - | (4)
      - | Create variables to build the queries to fetch records and to build the conditions (join condition and AND condition).
        | Same conditions need to be used in query for fetching entities and in query for fetching records; hence variables are created to build the conditions (join condition and AND condition).
    * - | (5)
      - | Build the static query elements of query to fetch records.
        | In the above example, SELECT clause and FROM clause are built as static query components.
    * - | (6)
      - | Build the dynamic query elements for query to fetch entities and query to fetch records.
    * - | (7)
      - | Sort conditions (ORDER BY clause) are built as dynamic query elements for the query to fetch entities.
        | Utility component (``org.springframework.data.jpa.repository.query.QueryUtils``) provided by Spring Data JPA is used for building the ORDER BY clause.
    * - | (8)
      - | Convert the query string for fetching the dynamically built records into ``javax.persistence.TypedQuery`` and set the bind parameter for required for executing the query.
    * - | (9)
      - | Execute the query for fetching the records and get the total number of records matching the conditions.
    * - | (10)
      - | If the entities matching the conditions exist, execute the query for fetching the entities and fetch the information of the corresponding page.
        | When executing the query for fetching entities, specify the start index for fetching records (``TypedQuery#setFirstResult``) and the number of records to be fetched (``TypedQuery#setMaxResults``).
    * - | (11)
      - | If entity matching the conditions does not exist, a blank list should be fetched.
    * - | (12)
      - | Specify entity list of the corresponding page, page information and total number of records matching the conditions as arguments and then create and return the ``Page`` object.

 .. raw:: latex

    \newpage

- Service (Caller)

 .. code-block:: java

    // condition values for sample.
    Integer conditionValueOfId = 4;
    List<String> conditionValueOfStatusCodes = Arrays.asList("accepted");
    String conditionValueOfItemName = "Wat";

    // implementation of sample.
    OrderCriteria criteria = new OrderCriteria();
    criteria.setId(conditionValueOfId);
    criteria.setStatusCodes(conditionValueOfStatusCodes);
    criteria.setItemName(conditionValueOfItemName);
    Page<Order> orderPage = orderRepository.findPageByCriteria(criteria,
            pageable); // (13)
    if (!orderPage.hasContent()) {
        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (13)
      - | Call the query method to fetch the entities corresponding to the specified page, amongst the entities matching the dynamic conditions.


|

Implementing the process to fetch entities
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The method of fetching entities is explained below.

|

Fetching 1 record of entity by specifying ID
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
If the ID (Primary Key) is known, fetch the entity object by calling the findOne method of Repository interface.

 .. code-block:: java

    public Account getAccount(String accountUuid) {
        Account account = accountRepository.findOne(accountUuid); // (1)
        if (account == null) { // (2)
            // ...
        }
        return account;
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify the ID (Primary Key) of entity and call the findOne(ID) method of Repository interface.
    * - | (2)
      - | When the specified entity ID does not exist, the return value would be null, hence null check is necessary.
        | If needed, implement the process which is carried out when the specified entity ID does not exist.

 .. note :: **Returned entity objects**

    When the entity object of specified ID is already managed by ``EntityManager``, entity object managed by ``EntityManager`` is returned without
    accessing the persistence layer (DB).
    Therefore, if findOne method is used, unnecessary access to persistence layer can be controlled.

 .. note :: **Load timing of the related-entity**

    Load of the related-entity during query execution is determined based on the value specified in fetch attribute of annotations
    (``@javax.persistence.OneToOne`` , ``@javax.persistence.OneToMany``, ``@javax.persistence.ManyToOne`` , ``@javax.persistence.ManyToMany`` ).
    
    * In case of ``javax.persistence.FetchType#LAZY`` , related-entity is not covered under JOIN FETCH; hence it is loaded at the time of initial access.
    * In case of ``javax.persistence.FetchType#EAGER`` , related-entity is covered under JOIN FETCH; hence it is loaded at the time of loading the parent-entity.

    Default values of fetch attribute differ depending on annotations. See the default values below:

    * ``@OneToOne`` annotation: ``EAGER``
    * ``@ManyToOne`` annotation: ``EAGER``
    * ``@OneToMany`` annotation: ``LAZY``
    * ``@ManyToMany`` annotation: ``LAZY``

 .. note:: **Sort order of the related-entities having 1:N(N:M) relationship**

    Specify ``@javax.persistence.OrderBy`` annotation on the property of related-entities to control the sort order.

 See the example below.

  .. code-block:: java

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    @OrderBy // (1)
    private Set<OrderItem> orderItems;

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | When value attribute is not specified, the entities are sorted in ascending order of ID.
        | For details, refer to `JSR 338: Java Persistence API, Specification (PDF) of Version 2.1 "11.1.42 OrderBy Annotation" <http://download.oracle.com/otn-pub/jcp/persistence-2_1-fr-eval-spec/JavaPersistence.pdf>`_\ .

 .. todo::

    **TBD**

    Following contents will be added in future.

    * Example of cases wherein it is better to change fetch attribute from default value.

|

Fetching 1 record of entity by specifying conditions other than ID 
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
When the ID is not known, call the query method to search entities by conditions other than ID.

- Repository interface

 .. code-block:: java

    public interface AccountRepository extends JpaRepository<Account, String> {

        // (1)
        @Query("SELECT a FROM Account a WHERE a.accountId = :accountId")
        Account findByAccountId(@Param("accountId") String accountId);

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Define the query method to search 1 record of entity by specifying items other than ID as conditions.

- Service

 .. code-block:: java

    public Account getAccount(String accountId) {
        Account account = accountRepository.findByAccountId(accountId); // (2)
        if (account == null) { // (3)
            // ...
        }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (2)
      - | Call the query method defined in Repository interface.
    * - | (3)
      - | Similar to findOne method of Repository interface, when the entities matching the conditions do not exist, null will be returned. Hence null check is necessary. 
        | If needed, implement the process for the scenario when entities matching the specified conditions do not exist.
        | When multiple entities matching the conditions exist, ``org.springframework.dao.IncorrectResultSizeDataAccessException`` occurs.

 .. note :: **Returned entity objects**

    When a query method is called, the query is always executed on persistence layer (DB).
    However, when the entities which are fetched by executing the query are already being managed in ``EntityManager``, the entity objects fetched from DB are discarded and
    the entity objects managed in ``EntityManager`` are returned.

 .. note :: **Query method using ID + α as condition**

    It is recommended not to create a query method wherein ID + α is used as condition.
    It can be implemented by creating a logic that compares property value of entity objects fetched by calling findOne method.

    The reason for not recommending the creation of this query method is that there is a possibility of the entity objects fetched by executing the query getting discarded and unnecessary query getting executed.
    If the ID is known, it is desirable to use findOne method which prevents execution of unnecessary queries.
    This should be consciously implemented especially in case of applications with high performance requirements.

    However, if the following conditions are applicable, use of query method may reduce the frequency of query execution; hence query method can be used in such cases.

    * Properties of related objects (column of related table) are included in a part of \+ α condition.
    * ``LAZY`` is included in FetchType of the related-entities as a condition.


 .. note :: **Load timing of the related-entities**

    Related-entities specified in JOIN FETCH are loaded immediately after executing the query.

    The related-entities not specified in JOIN FETCH performs the following operations as per the values specified in  fetch attribute of
    associated annotations ( ``@OneToOne`` , ``@OneToMany`` , ``@ManyToOne`` , ``@ManyToMany`` ).

    * ``javax.persistence.FetchType#LAZY`` is covered under Lazy Load; hence the related-entities are loaded at the time of initial access.
    * In case of ``javax.persistence.FetchType#EAGER``, query is executed to load the related-entities and objects of the related-entities are loaded.

 .. note:: Sort order of the related-entities having 1:N(N:M) relationship

   * The sort order of the related-entities specified in JOIN FETCH is controlled by specifying "ORDER BY" clause in JPQL.
   * The sort order of the related-entities loaded after executing the query is controlled by specifying ``@javax.persistence.OrderBy`` annotation to the property of the related-entities.

|


Adding entities
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Example of adding entities is shown below.

.. _data-access-jpa_how_to_use_way_to_add_entity:

How to add entities
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
In order to add an entity, create an entity object and call the save method of Repository interface.

- Service

 .. code-block:: java

    Order order = new Order("accepted"); // (1)
    order = orderRepository.save(order); // (2)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Create an instance of entity object and set the values for required properties.
        | In the above example, ID generator of JPA is used for setting the ID. When ID generator of JPA is to be used, ID should not be set in the application code.
        | If you set the ID in the application code, merge method of ``EntityManager`` gets called leading to execution of unnecessary processing.
    * - | (2)
      - | Call the save method of Repository interface and manage the entity objects created in (1) in ``EntityManager``.
        | Take note that entity object passed as an argument of save method will not be the one managed by ``EntityManager``, but the entity object returned by save method will be the one managed by ``EntityManager``.
        | ID is set by the ID generator of JPA at the time of this process.

 .. note:: **Demerits of the merge method getting called**

    merge method of ``EntityManager`` has a mechanism to fetch the entities having same ID from persistence layer (DB), when the entities are to be managed in ``EntityManager``.
    Process to fetch the entities becomes unnecessary while adding the entities. In case of application with high performance requirements, ID generation timing should also be taken into account.

 .. note:: **Constraint error handling**

    When save method is called, query (INSERT) is not executed in the persistence layer (DB).
    Therefore, when constraint error such as unique constraint violation needs to be handled, saveAndFlush method or flush method should be called
    instead of save method of Repository interface.


- Entity

 .. code-block:: java

    @Entity // (3)
    @Table(name = "t_order") // (4)
    public class Order implements Serializable {

        // (5)
        @SequenceGenerator(name = "GEN_ORDER_ID", sequenceName = "s_order_id",
                           allocationSize = 1)
        @GeneratedValue(strategy = GenerationType.SEQUENCE,
                        generator = "GEN_ORDER_ID")
        @Id // (6)
        private int id;

        // (7)
        @ManyToOne
        @JoinColumn(name = "status_code")
        private OrderStatus status;

        // ...

        public Order(String statusCode) {
            this.status = new OrderStatus(statusCode);
        }

        // ...

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (3)
      - | Assign ``@javax.persistence.Entity`` annotation to the entity class.
    * - | (4)
      - | Specify the table name to be mapped with the entity class in name attribute of ``@javax.persistence.Table`` annotation.
        | The table name need not be specified if it can be resolved from entity name; however, it has to be specified if it cannot be resolved from entity name.
    * - | (5)
      - | The annotations required for using ID generator of JPA are being specified.
        | When using ID generator of JPA, specify ``@javax.persistence.GeneratedValue`` annotation.
        | When using sequence object, ``@javax.persistence.SequenceGenerator`` annotation should be specified. When using table generator, ``@javax.persistence.TableGenerator`` annotation should be specified.
        | In the above example, ID is generated using the sequence object called ``"s_order_id"`` name.
    * - | (6)
      - | Assign ``@javax.persistence.Id`` annotation to the property that holds the primary key.
        | In case of composite key, assign ``@javax.persistence.EmbeddedId`` annotation.
    * - | (7)
      - | Assign the associated annotations (``@OneToOne`` , ``@OneToMany`` ,  ``@ManyToOne`` , ``@ManyToMany`` ) to the property that has a relationship with other entities.

 .. note :: **Annotations for generating IDs**

    For details on each annotation, refer to `JSR 338: Java Persistence API, Version 2.1 Specification (PDF) <http://download.oracle.com/otn-pub/jcp/persistence-2_1-fr-eval-spec/JavaPersistence.pdf>`_\ .

    * ``@GeneratedValue`` : 11.1.20 GeneratedValue Annotation
    * ``@SequenceGenerator`` : 11.1.48 SequenceGenerator Annotation
    * ``@TableGenerator`` : 11.1.50 TableGenerator Annotation

 .. note :: **About method of generating IDs**

    For generating ID, specify the value of ``javax.persistence.GenerationType`` in strategy attribute of ``@GeneratedValue`` annotation.
    Values that can be specified are as follows:

    * ``TABLE``:  Generate ID using persistence layer (DB) table.
    * ``SEQUENCE``: Generate ID using sequence object of persistence layer (DB).
    * ``IDENTITY``: Generate ID using identity column of persistence layer (DB).
    * ``AUTO``: Generate ID by selecting the most appropriate method in persistence layer (DB).

   Generally it is recommended to explicitly specify the type to be used instead of using ``AUTO``.


|

Adding parent-entity and related-entity
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
In order to add parent-entity and related-entity together, call the save method of Repository interface and manage the entity objects under ``EntityManager``.
Then create the related-entity objects and map them with parent-entity objects.

In order to use this method, ``persist`` needs to be included in the cascade operation of the related-entity.

 .. note:: **Behavior of entities in case of cascade operations**

    When the related-entity is specified under cascade operations, JPA operations for parent-entity (``persist``, ``merge``, ``remove``, ``refresh``, ``detach``)
    are linked with related-entity and then carried out.
    
    Mapping with the operations of Repository interface of Spring Data JPA is as follows:

    * save method : ``persist`` or ``merge``
    * delete method : ``remove``

    ``refresh``, ``detach`` are not executed in default implementation of Spring Data JPA.


- Service

 .. code-block:: java

    String itemCode = "ITM0000001";
    int itemQuantity = 10;
    String wayToPay = "card";

    Order order = new Order("accepted");
    order = orderRepository.save(order); // (1)

    OrderItem orderItem = new OrderItem(order.getId(), itemCode,
            itemQuantity);
    order.setOrderItems(Collections.singleton(orderItem));    // (2)
    order.setOrderPay(new OrderPay(order.getId(), wayToPay)); // (2)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | First, call the save method of Repository interface and bring the entity object under ``EntityManager``.
        | In the above example, save method is being called before setting the related-entity objects. This is because it is necessary to generate the ID (orderId) of parent-entity; this ID is used as a part of ID of the related entity.
    * - | (2)
      - | Set the related-entity objects for the parent-entity object (that are to be managed under ``EntityManager``).
        | When committing the transaction, persist operation (INSERT) on parent-entity object is linked with the related-entity objects.

- Entity

 .. code-block:: java

    @Entity
    @Table(name = "t_order")
    public class Order implements Serializable {

        private static final long serialVersionUID = 1L;

        @Id
        @SequenceGenerator(name = "GEN_ORDER_ID", sequenceName = "s_order_id",
                           allocationSize = 1)
        @GeneratedValue(strategy = GenerationType.SEQUENCE,
                        generator = "GEN_ORDER_ID")
        private int id;

        @OneToMany(mappedBy = "order", cascade = CascadeType.ALL,  // (3)
                   orphanRemoval = true)
        @OrderBy
        private Set<OrderItem> orderItems;

        @OneToOne(mappedBy = "order", cascade = CascadeType.ALL,
                  orphanRemoval = true)
        private OrderPay orderPay;

        @ManyToOne
        @JoinColumn(name = "status_code")
        private OrderStatus status;

        // ...

        public Order(String statusCode) {
            this.status = new OrderStatus(statusCode);
        }

        // ...

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (3)
      - | Specify the cascade operation type (``javax.persistence.CascadeType``) in cascade attribute in corresponding annotation.
        | In the above example, all operations are target of cascade to the related-entity.
        | If there is no specific reason, it is recommended to consider all the operations for cascade.

 .. warning :: **Related-entity for which cascade attribute should not be specified**

    If the transaction entity is associated with code and master entities, cascade attribute should not be specified.
    In the above example, ``OrderStatus`` is the code entity; hence cascade attribute should not be specified in ``@ManyToOne`` annotation of ``status`` property of ``Order``.


- Related-entity

 .. code-block:: java

    @Entity
    @Table(name = "t_order_item")
    public class OrderItem implements Serializable {

        @EmbeddedId
        private OrderItemPK id;

        private int quantity;

        @ManyToOne
        @JoinColumn(name = "order_id", insertable = false, updatable = false)
        private Order order;

        // ...

        public OrderItem(Integer orderId, String itemCode, int quantity) {
            this.id = new OrderItemPK(orderId, itemCode);
            this.quantity = quantity;
        }

        // ...

    }

    @Entity
    @Table(name = "t_order_pay")
    public class OrderPay implements Serializable {

        @Id
        @Column(name = "order_id")
        private Integer orderId;

        @Column(name = "way_to_pay")
        private String wayToPay;

        @OneToOne
        @JoinColumn(name = "order_id")
        private Order order;

        // ...

        public OrderPay(int orderId, String wayToPay) {
            this.orderId = orderId;
            this.wayToPay = wayToPay;
        }

        // ...

    }

|

Adding the related-entity
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
In order to add a related-entity, link the newly created related-entity object with the parent-entity object fetched through Repository interface.

For using this method, ``persist`` and ``merge`` should be included in the cascade operation of the related-entity.

 .. code-block:: java

    String itemCode = "ITM0000003";
    int quantity = 30;

    Order order = orderRepository.findOne(orderId); // (1)

    OrderItem orderItem = new OrderItem(order.getId(), itemCode, quantity);
    order.getOrderItems().add(orderItem); // (2)

    OrderPay orderPay = order.getOrderPay();
    if (orderPay == null) {
        order.setOrderPay(new OrderPay(order.getId(), "cash")); // (3)
    } else {
        orderPay.setWayToPay("cash");
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Fetch the entity object using the Repository interface.
    * - | (2)
      - | In case of entity with 1: N relationship, add the related-entity object to the collection fetched from the parent-entity object.
    * - | (3)
      - | In case of entity with 1:1 relationship, set the related-entity object in the parent-entity object.

|

Adding the related-entity directly
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
When a related-entity object is to be added directly without linking it with the parent-entity object, save it using the Repository interface of related-entity.

 .. code-block:: java

    String itemCode = "ITM0000003";
    int quantity = 40;

    OrderItem orderItem = new OrderItem(orderId, itemCode, quantity); // (1)

    orderItemRepository.save(orderItem); // (2)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Create an object of related-entity.
    * - | (2)
      - | Call the save method of Repository interface of the related-entity.

 .. note:: **Merits of saving the related-entity object directly**

     The number of objects created is less. When fetching the parent-entity object, related-entity objects which are not necessary for the processing may also get created.

 .. note:: **Demerits of saving the related-entity directly**

    When ID of the parent-entity is being used as a part of the related-entity ID, the ID should be set before calling the save method.
    If ID is set, merge method of ``EntityManager`` is called as per the default implementation of Spring Data JPA.
    Therefore, the entity with same ID is always fetched from persistence layer (DB).

 .. warning:: **Points to be noted at the time of using parent-entity object after the related-entity is added**

    When the related-entity is added using Repository save method of related-entity, 
    it cannot be fetched through the parent-entity object since it is not linked with the parent-entity object.

    To avoid this problem,

    #. Related-entity object should not be added directly. It should first be linked with the parent-entity object, and then added.
    #. Synchronization with persistence layer (DB) should be done before fetching the parent-entity, using saveAndFlush method.

    In such a case, if there is no specific reason, objects of the related-entity should be added by the former method.
    In the latter method, the problem cannot be avoided if the parent-entity object is already the "managed" entity.

|

Updating entities
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The example of updating entities is explained below.

|

How to update entities
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
In order to update the entity, set the changed value to the entity object fetched using the Repository interface method.

 .. code-block:: java

    Order order = orderRepository.findOne(orderId); // (1)
    order.setStatus(new OrderStatus("checking")); // (2)


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Fetch the entity object using Repository interface method.
    * - | (2)
      - | Update the state of entity object by calling setter method.

 .. note:: **Calling the save method of Repository**

    The entity object fetched using Repository interface method are managed under ``EntityManager``.
    For the entity objects managed under ``EntityManager``, just by changing the state of objects using setter method,
    the changes are reflected in persistence layer (DB) at the time of committing the transaction.
    Therefore, there is no need to explicitly call the save method of Repository interface.

    However, save method needs to be called when the entity objects are not managed under ``EntityManager``.
    For example, when entity object is created based on the request parameters sent from the screen.

|

Updating the related-entity
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
In order to update the related-entity, first fetch the related-entity from the parent entity which in turn can be fetched using Repository interface and then set the values to be updated in related-entity object.

For using this method, ``merge`` should be included in the cascade operation of related-entity.

 .. code-block:: java

    Order order = orderRepository.findOne(orderId); // (1)

    for (OrderItem orderItem : order.getOrderItems()) {
        int newQuantity = quantityMap.get(orderItem.getId().getItemCode());
        orderItem.setQuantity(newQuantity); // (2)
    }

    order.getOrderPay().setWayToPay("cash"); // (3)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Use the Repository interface method to fetch entity objects.
    * - | (2)
      - | In case of entity with 1:N relationship, the state of related-entity object stored in the collection fetched from parent-entity object is to be updated by calling the setter method.
    * - | (3)
      - | In case of entity with 1:1 relationship, the state of related-entity object fetched from parent-entity object is to be updated by calling the setter method.

|


Updating the related-entity directly
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
In order to update the related-entity directly without using the parent-entity, fetch the related-entity directly using its corresponding Repository interface and set the value to be changed.

 .. code-block:: java

    int quantity = 43;

    OrderItem orderItem = orderItemRepository.findOne(new OrderItemPK(
            orderId, itemCode)); // (1)

    orderItem.setQuantity(quantity); // (2)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Call findOne method of Repository interface of the related-entity and fetch the related-entity object.
    * - | (2)
      - | Update the state of related-entity object using setter method.

 .. note:: **Behavior at the time of using parent-entity after the related-entity is updated**

    When the related-entity is updated using save method of Repository for related-entity,
    the related-entity stored in the parent-entity object is also updated unlike the case wherein the related-entity is added.
    This is because the parent-entity stores the reference of same instance which is managed under ``EntityManager``.

|

Updating by using query method
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Use query method to update the entity of persistence layer (DB) directly.
| For details, refer to ":ref:`data-access-jpa_howtouse_querymethod_modifying`".

|

Deleting entities
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Deleting parent-entity and related-entity
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
In order to delete parent-entity and related-entity together, call delete method of Repository interface.

For using this method, ``remove`` should be included in the cascade operation of the related-entity
or the setting for deleting the related-entity should be enabled (orphanRemoval attribute should be set to ``true``).

- Service

 .. code-block:: java

    orderRepository.delete(orderId); // (1)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify ID or entity object and call delete method of Repository interface.

- Entity

 .. code-block:: java

    @Entity
    @Table(name = "t_order")
    public class Order implements Serializable {

        // ...

        @OneToMany(mappedBy = "order",
                   cascade = CascadeType.ALL, orphanRemoval = true) // (2)
        @OrderBy
        private Set<OrderItem> orderItems;

        // ...

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (2)
      - | Include ``remove`` in cascade operations and enable the setting to delete the related-entity (set orphanRemoval attribute of the associated annotation to ``true``).

 .. note:: **Deleting related-entity**

    If you do not want to delete the related-entity object, perform settings such that ``remove`` is not included in cascade operation.
    Also, set orphanRemoval attribute to ``false``.

|

.. _daba-access-jpa_howtouse_remove_relationship:

Deleting the related-entity
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
In order to delete the related-entity, delete the related-entity object from the entity objects fetched through Repository interface.

For using this method, setting to delete the related-entity should be enabled (orphanRemoval attribute of the associated annotation should be set to ``true``).

 .. note:: **Behavior when the setting to delete the related-entity is enabled**

    Behavior when the setting to delete the related-entity is enabled is as follows:

    * In case of entity with 1:N relationship, if the related-entity object is deleted from the collection, it is also deleted from the persistence layer (DB) at the time of committing the transaction.
    * In case of entity with 1:1 relationship, if the related-entity is set to ``null``, it is also deleted from the persistence layer (DB) at the time of committing the transaction.

    When the value of orphanRemoval attribute is set to ``false`` (which is also the default value), the related-entity is cleared from the memory; 
    however persistence operation (UPDATE/DELETE) is not carried out for deleted related-entity object.


- Service

 .. code-block:: java

    Order order = orderRepository.findOne(orderId); // (1)

    // (2)
    Set<OrderItem> orderItemsOfRemoveTarget = new LinkedHashSet<OrderItem>(); // (3)
    for (OrderItem orderItem : order.getOrderItems()) {
        String itemCode = orderItem.getId().getItemCode();
        if (quantityMap.containsKey(itemCode)) {
            int newQuantity = quantityMap.get(itemCode);
            orderItem.setQuantity(newQuantity);
        } else {
            orderItemsOfRemoveTarget.add(orderItem); // (4)
        }
    }
    order.getOrderItems().removeAll(orderItemsOfRemoveTarget); // (5)

    order.setOrderPay(null); // (6)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Fetch the entity object using Repository interface.
    * - | (2)
      - | Describe the example to delete the entity with 1:N relationship.
    * - | (3)
      - | Create a collection for storing the related-entity object to be deleted.
    * - | (4)
      - | Add the related-entity object to be deleted to the collection of (3).
    * - | (5)
      - | Delete it from the collection fetched from the related-entity object to be deleted.
    * - | (6)
      - | In case of entity with 1:1 relationship, set the property to that stores the related-entity object to be deleted to ``null``.

- Entity

 .. code-block:: java

    @Entity
    @Table(name = "t_order")
    public class Order implements Serializable {

        @Id
        @SequenceGenerator(name = "GEN_ORDER_ID", sequenceName = "s_order_id", allocationSize = 1)
        @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "GEN_ORDER_ID")
        private int id;

        @OneToMany(mappedBy = "order", cascade = CascadeType.ALL,
                   orphanRemoval = true) // (7)
        @OrderBy
        private Set<OrderItem> orderItems;

        @OneToOne(mappedBy = "order", cascade = CascadeType.ALL,
                  orphanRemoval = true) // (7)
        private OrderPay orderPay;

        @ManyToOne
        @JoinColumn(name = "status_code")
        private OrderStatus status;

        // ...

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (7)
      - | Enable the setting to delete the related-entity (set orphanRemoval attribute to ``true``).

 .. note :: **Associated annotations that can be specified for orphanRemoval attribute**

    There are 2 associated annotations that can be specified for orphanRemoval attribute; ``@OneToOne``  and  ``@OneToMany``.

|

Deleting the related-entity directly
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
In order to delete the related-entity without using the parent-entity, call the delete method of Repository interface of related-entity.

 .. code-block:: java

    int quantity = 43;

    orderItemRepository.delete(new OrderItemPK(orderId, itemCode)); // (1)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify ID or entity object and call delete method of Repository interface of the related-entity.

 .. warning:: **Points to be noted while deleting the related-entity directly**

    When delete method of Repository for the related-entity is called, it should be noted that related-entity may not get deleted from persistence layer (DB) in some cases.

    Such cases are as follows:

    * findOne method is called in delete method; hence when the relation with parent-entity is ``@OneToOne``, the parent-entity ends up getting managed under ``EntityManager``.
      If the parent-entity ends up getting managed under ``EntityManager``, the related-entity that was to be deleted using delete method may be loaded in the parent-entity object.
      Once it is loaded under the parent-entity object, it will not be deleted from persistence layer (DB).
    * If the parent-entity object is fetched after calling the delete method, the related-entity which is deleted using delete method may be loaded in the parent-entity object.
      Once it is loaded under parent-entity object, it cannot be deleted from persistence layer (DB).

    How to avoid this problem,

    #. Instead of deleting the related-entity object directly, its association with the parent-entity should be deleted.

    It may be possible to avoid this problem by reconsidering the associated annotation; however the best way to avoid this problem is not to delete the related-entity directly.

|

Deleting using query method
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Use query method in order to directly delete the entity from persistence layer (DB).
| For details, refer to ":ref:`data-access-jpa_howtouse_querymethod_modifying`".

 .. warning:: **Handling of related-objects**

    When the entity is deleted directly from the persistence layer (DB) using query method, irrespective of whether or not the associated annotation is specified,
    the related-entity object is not deleted from the persistence layer.

    ``void deleteInBatch(Iterable<T> entities)`` and ``void deleteAllInBatch()`` of Repository interface operate in the same way.

|

.. _data-access-jpa_howtouse_like_escape:

Escaping at the time of LIKE search
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Escaping the values to be used as search criteria should be done for LIKE search.
| Escaping for LIKE search can be done using ``org.terasoluna.gfw.common.query.QueryEscapeUtils`` class method of the common library.
| For specifications of escaping in common library, refer to ":ref:`data-access-common_appendix_like_escape`" of ":doc:`DataAccessCommon`".

For the escaping method provided by common library, see the description below.

|

Usage method when type of matching is to be specified in query
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
When type of matching (Forward match, Backward Match, Partial Match) is to be specified as JPQL, use the method that performs only escaping.

- Repository

 .. code-block:: java

    // (1) (2)
    @Query("SELECT a FROM Article a WHERE"
            + " (a.title LIKE %:word% ESCAPE '~' OR a.overview LIKE %:word% ESCAPE '~')")
    Page<Article> findPageBy(@Param("word") String word, Pageable pageable);

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify wildcard characters ( ``"%"`` or ``"_"`` ) for LIKE search in JPQL to be specified in ``@Query`` annotation.
        | In the above example, the type of matching is set to partial match by specifying wildcard ( ``"%"`` ) before and after the argument ``word``.
    * - | (2)
      - | In case of escaping provided by the common library, ``"~"`` is being used as escape characters; hence specify ``"ESCAPE '~'"`` after LIKE clause.

 .. note :: **About wildcard character "_"**

    As described in [:ref:`how_to_specify_query_annotation-label`], the wildcard character ``"%"`` can be use directly in JPQL only if the ``@Query`` annotation is used.
    The wildcard character ``"_"`` cannot use directly in LIKE search of JPQL. It can be used in the following two ways.
    
    #. Include wildcard character ``"_"`` in the bind variable.
    #. Concatenate the wildcard character ``"_"`` with bind variable using database string concatenation function such as ``CONCAT`` . 

|

- Service

 .. code-block:: java

    @Transactional(readOnly = true)
    public Page<Article> searchArticle(ArticleSearchCriteria criteria,
            Pageable pageable) {

        String escapedWord = QueryEscapeUtils.toLikeCondition(criteria.getWord()); // (3)

        Page<Article> page = articleRepository.findPageBy(escapedWord, pageable); // (4)

        return page;
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (3)
      - | When type of matching of LIKE search is to be specified in query, call ``QueryEscapeUtils#toLikeCondition(String)`` method and perform only escaping for LIKE search.
    * - | (4)
      - | Pass the value escaped for LIKE search as the argument of query method.

|

Usage method when specifying type of matching in logic
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
When the type of matching (Forward match, Backward match, Partial match) is to be determined in logic, use the method that assigns wildcard to the escaped values.

- Repository

 .. code-block:: java

    // (1)
    @Query("SELECT a FROM Article a WHERE"
            + " (a.title LIKE :word ESCAPE '~' OR a.overview LIKE :word ESCAPE '~')")
    Page<Article> findPageBy(@Param("word") String word, Pageable pageable);

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Do not specify wildcard for LIKE search in JPQL to be specified in ``@Query`` annotation.

|

- Service

 .. code-block:: java

    @Transactional(readOnly = true)
    public Page<Article> searchArticle(ArticleSearchCriteria criteria,
            Pageable pageable) {

        String word = QueryEscapeUtils
                .toContainingCondition(criteria.getWord()); // (2)

        Page<Article> page = articleRepository.findPageBy(word, pageable); // (3)

        return page;
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (2)
      - | When the type of matching is to be specified in logic, call any of the following methods and perform escaping for LIKE search and assign wildcard for LIKE search.
        |   ``QueryEscapeUtils#toStartingWithCondition(String)``
        |   ``QueryEscapeUtils#toEndingWithCondition(String)``
        |   ``QueryEscapeUtils#toContainingCondition(String)``
    * - | (3)
      - | Pass the value escaped for LIKE search＋ assigned with wildcard as an argument of query method.

|

.. _data-access-jpa_howtouse_join_fetch:

JOIN FETCH
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| JOIN FETCH is a mechanism to reduce the number of queries generated for fetching entities, by joining and fetching the related-entities in batch.

| When fetching the entities by executing any query, make sure to perform JOIN FETCH for properties where fetch attribute of associated annotation is ``EAGER``.
| If JOIN FETCH is not performed, it may impact performance since queries to fetch the related-entities will be generated separately.

| For properties where fetch attribute of related-annotation is ``LAZY``, consider the frequency of use and determine whether or not to perform JOIN FETCH.
| JOIN FETCH should be performed in case of related-entities that are always referenced. However, if the cases wherein related-entities will be referenced are only few, then it is better to fetch the entities by executing the queries at the time of initial access instead of performing JOIN FETCH.

- Repository

 .. code-block:: java

    @Query("SELECT a FROM Article a"
            + " INNER JOIN FETCH a.articleClass"   // (1)
            + " WHERE a.publishedDate = :publishedDate"
            + " ORDER BY a.articleId DESC")
    List<Article> findAllByPublishedDate(
            @Param("publishedDate") Date publishedDate);

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify in ``" [LEFT [OUTER] | INNER] JOIN FETCH Properties to be joined"``.
        | See the pattern below.
        |
        | 1. ``LEFT JOIN FETCH``
        |    Entity is obtained even if the related-entity does not exist.
        |    SQL will be ``"left outer join RelatedTable r on m.fkColumn = r.fkColumn"``.
        | 2. ``LEFT OUTER JOIN FETCH``
        |    Same as 1.
        | 3. ``INNER JOIN FETCH``
        |    Entity is not obtained if related-entity does not exist.
        |    SQL will be  ``"inner join RelatedTable r on m.fkColumn = r.fkColumn"``.
        | 4. ``JOIN FETCH``
        |    Same as 3.
        |
        | In the above example, articleClass property of Article class is specified as the JOIN FETCH target.

 .. note::

    JOIN FETCH is used as one of the solutions for N+1 problem.
    For N+1 problem, refer to ":ref:`data-access-common_howtosolve_n_plus_1`".


|

How to extend
--------------------------------------------------------------------------------

.. _data-access-jpa_how_to_extends_custommethod:

How to add custom method
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Spring Data provides mechanism to add any custom methods for the Repository interface.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table::
    :widths: 10 35 55
    :header-rows: 1

    * - Sr. No.
      - How to Add
      - Use case
    * - 1.
      - :ref:`custommethod_individual-label`
      - | Use when it is necessary to execute a query that cannot be expressed using the mechanism of query method of Spring Data.
        | Add methods using this method when executing dynamic queries.
    * - 2.
      - :ref:`custommethod_all-label`
      - | Use when it is necessary to execute a generic query that can be used in all Repository interfaces.
        | Add methods using this method when executing project specific generic queries.
        | If it is necessary to change the behavior of default implementation ( ``SimpleJpaRepository`` ) of Spring Data JPA, override the methods using this mechanism.

|

.. _custommethod_individual-label:

Adding individual custom method to entity specific Repository interface
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
See the description below for adding individual custom method to entity specific Repository interface.

- Entity specific custom Repository interface

 .. code-block:: java

    // (1)
    public interface OrderRepositoryCustom {

        // (2)
        Page<Order> findByCriteria(OrderCriteria criteria, Pageable pageable);

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Create a custom interface for defining custom methods.
        | There are no specific restrictions for naming the interface; however it is recommended that you set it to entity specific Repository interface name + ``"Custom"``.
    * - | (2)
      - | Define custom methods.

- Entity specific custom Repository class

 .. code-block:: java

    // (3)
    public class OrderRepositoryImpl implements OrderRepositoryCustom {

        @PersistenceContext
        EntityManager entityManager; // (4)

        // (5)
        public Page<Order> findByCriteria(OrderCriteria criteria, Pageable pageable) {
            // ...
            return new PageImpl<Order>(orders, pageable, totalCount);
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (3)
      - | Create implementation class of custom interface.
        | The class name should be, entity specific Repository interface name + ``"Impl"``.
    * - | (4)
      - | Using ``@javax.persistence.PersistenceContext`` annotation, inject ``javax.persistence.EntityManager`` which is necessary for executing the query.
    * - | (5)
      - | Implement the method defined in custom interface.

- Entity specific Repository interface

 .. code-block:: java

    public interface OrderRepository extends JpaRepository<Order, Integer>,
            OrderRepositoryCustom { // (6)
        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (6)
      - | Inherit custom interface in entity specific Repository interface.
        | Methods of implementation class of custom interface are called at runtime by inheriting Repository interface.

- Service(Caller)

 .. code-block:: java

    public Page<Order> search(OrderCriteria criteria, Pageable pageable) {
        return orderRepository.findByCriteria(criteria, pageable); // (7)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (7)
      - | Methods of entity specific Repository interface can be called similar to other methods.
        | In the above example, if ``OrderRepository#findByCriteria(OrderCriteria, Pageable)`` is called, ``OrderRepositoryImpl#findByCriteria(OrderCriteria, Pageable)`` is executed.

|

.. _custommethod_all-label:

Adding the custom methods to all Repository interfaces in batch
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
See the description below for adding the custom methods to all Repository interfaces in batch.

The method added in the example given below checks the version of the fetched entity and throws an optimistic exclusion error in case of a mismatch.

- Common Repository interface

 .. code-block:: java

    // (1)
    @NoRepositoryBean // (2)
    public interface MyProjectRepository<T, ID extends Serializable> extends
            JpaRepository<T, ID> {

        // (3)
        T findOneWithValidVersion(ID id, Integer version);

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Create a common Repository interface for defining the methods to be added.
        | In the above example, common Repository interface is being created by inheriting ``JpaRepository`` of Spring Data.
    * - | (2)
      - | Annotation to prevent common Repository interface implementation class (in this example, ``MyProjectRepositoryImpl``) from being automatically registered as Repository Bean.
        | When common Repository interface implementation class is to be stored under the package specified in base-package attribute of <jpa:repositories> element, there will be an error at start-up unless this annotation is specified.
    * - | (3)
      - | Define the method to be added. In the example, method to check the version of fetched entity is being added.

- Common Repository interface implementation class

 .. code-block:: java

    // (6)
    public class MyProjectRepositoryImpl<T, ID extends Serializable>
            extends SimpleJpaRepository<T, ID>
            implements MyProjectRepository<T, ID> {

        private JpaEntityInformation<T, ID> entityInformation;
        private EntityManager entityManager;

        // (7)
        public MyProjectRepositoryImpl(
                JpaEntityInformation<T, ID> entityInformation,
                EntityManager entityManager) {
            super(entityInformation, entityManager);

            this.entityInformation = entityInformation; // (8)
            this.entityManager = entityManager; // (8)

            try {
                return domainClass.getMethod("getVersion");
            } catch (NoSuchMethodException | SecurityException e) {
                return null;
            }
        }

        // (9)
        public T findOneWithValidVersion(ID id, Integer version) {
            if (versionMethod == null) {
                throw new UnsupportedOperationException(
                        String.format(
                                "Does not found version field in entity class. class is '%s'.",
                                entityInformation.getJavaType().getName()));
            }

            T entity = findOne(id);

            if (entity != null && version != null) {
                Integer currentVersion;
                try {
                    currentVersion = (Integer) versionMethod.invoke(entity);
                } catch (IllegalAccessException | IllegalArgumentException
                        | InvocationTargetException e) {
                    throw new IllegalStateException(e);
                }
                if (!version.equals(currentVersion)) {
                    throw new ObjectOptimisticLockingFailureException(
                            entityInformation.getJavaType().getName(), id);
                }
            }

            return entity;
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (6)
      - | Create an implementation class of common Repository interface (in this example, ``MyProjectRepository``).
        | In the above example, implementation class is being created by inheriting `` SimpleJpaRepository`` of Spring Data.
    * - | (7)
      - | Create a constructor that takes ``org.springframework.data.jpa.repository.support.JpaEntityInformation`` and ``javax.persistence.EntityManager`` as arguments.
    * - | (8)
      - | Store ``JpaEntityInformation`` and ``EntityManager`` as fields.
    * - | (9)
      - | Implement the method defined in common Repository interface.

 .. note:: **Changing default implementation**

    If the behavior of default implementation ( ``SimpleJpaRepository`` ) is to be changed, the method  for which behavior is to be changed can be overridden in this class.

-  Factory class for creating instances of common Repository interface implementation class

 .. code-block:: java

    // (10)
    private static class MyProjectRepositoryFactory<T, ID extends Serializable>
            extends JpaRepositoryFactory {

        // (11)
        public MyProjectRepositoryFactory(EntityManager entityManager) {
            super(entityManager);
        }

        // (12)
        protected JpaRepository<T, ID> getTargetRepository(
                RepositoryMetadata metadata, EntityManager entityManager) {

            @SuppressWarnings("unchecked")
            JpaEntityInformation<T, ID> entityInformation = getEntityInformation((Class<T>) metadata
                    .getDomainType());

            MyProjectRepositoryImpl<T, ID> repositoryImpl = new MyProjectRepositoryImpl<T, ID>(
                    entityInformation, entityManager);
            repositoryImpl
                    .setLockMetadataProvider(LockModeRepositoryPostProcessor.INSTANCE
                            .getLockMetadataProvider()); // (13)

            return repositoryImpl;
        }

        // (14)
        protected Class<?> getRepositoryBaseClass(RepositoryMetadata metadata) {
            return MyProjectRepository.class;
        }
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (10)
      - | Create a Factory class for creating instances of common Repository interface implementation class.
        | In the example, ``org.springframework.data.jpa.repository.support.JpaRepositoryFactory`` is being inherited in order to restrict the implementation of Factory class.
    * - | (11)
      - | Define a constructor that takes ``EntityManager`` as argument and call constructor of parent class.
    * - | (12)
      - | Override the method creating instances of implementation class. Create and return instances of the created implementation class (in this example, ``MyProjectRepositoryImpl``).
    * - | (13)
      - | This code is required to specify the lock mode using ``@Lock`` annotation for ``findOne`` method and ``findAll`` methods of ``SimpleJpaRepository`` class of Spring Data JPA.
        | Make sure to implement this piece of logic, since it is implemented in the method of ``JpaRepositoryFactory``  which is being overridden.
        | This process is required when the lock mode is to be specified using ``findOne`` and ``findAll`` methods amongst the methods to be extended.
    * - | (14)
      - | Override the method that returns base interface of entity specific Repository interface, and return the created interface (in this example, ``MyProjectRepository``) type.

- FactoryBean for creating Factory class instances

 .. code-block:: java

    // (15)
    public class MyProjectRepositoryFactoryBean<R extends JpaRepository<T, ID>, T, ID extends Serializable>
            extends JpaRepositoryFactoryBean<R, T, ID> {

        // (16)
        protected RepositoryFactorySupport createRepositoryFactory(
                EntityManager entityManager) {
            return new MyProjectRepositoryFactory<T, ID>(entityManager);
        }
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (15)
      - | Create FactoryBean class for creating Factory class instances.
        | In the example, ``org.springframework.data.jpa.repository.support.JpaRepositoryFactory`` is being inherited in order to restrict the implementation of FactoryBean class.
    * - | (16)
      - | Override the method used for creating the Factory class instances. Then, create and return instances of the created Factory class.

- Entity specific Repository interface

 .. code-block:: java

    public interface OrderRepository extends MyProjectRepository<Order, Integer> { // (17)
        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (17)
      - | Inherit all entity specific Repository interfaces from the common interface (In this example: ``MyProjectRepository``).

- ``xxx-infra.xml``

 .. code-block:: xml

    <jpa:repositories base-package="x.y.z.domain.repository"
        factory-class="x.y.z.domain.repository.MyProjectRepositoryFactoryBean" /> <!-- (18) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (18)
      - | Specify class name of the created FactoryBean class (in the example, ``MyProjectRepositoryFactoryBean``) in factory-class attribute of <jpa:repositories> element.

- Service(Caller)

 .. code-block:: java

    public Order updateOrder(Order chngedOrder, Integer version) {

        Order order = orderRepository.findOneWithValidVersion(chngedOrder.getId(), version); // (19)

        // ....

        return order;
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (19)
      - | The methods of entity specific Repository interface can be called similar to other methods.
        | In the above example, when ``OrderRepository#findOneWithValidVersion(Integer, Integer)`` is called, ``MyProjectRepositoryImpl#findOneWithValidVersion(Integer, Integer)`` is executed.

|


Storing query fetch results in objects other than entity
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Query fetch results can be mapped to other objects than entities.
Use this method when records stored in persistence layer (DB) are to be handled as objects (JavaBean) other than entity.

 .. note:: **Assumed cases**

    #. When the aggregated information is to be fetched using Aggregate function in query, it is not possible to map the aggregation result to entity; hence it should be mapped with different objects.

    #. In order to refer to only a part of information in a huge entity, or of a related-entity with complex nesting, there may be cases wherein it is desirable to map the results with JavaBean containing only the necessary properties.
       This is because processing performance may get hampered due to the fact that mapping is carried out for items which are not needed in application processing or fetching of unnecessary information during the processing leading to memory exhaustion.
       It may be obtained as entity if there is no significant impact on processing performance.

|

See the example below.

- JavaBean

 .. code-block:: java

    // (1)
    public class OrderSummary implements Serializable {

        private Integer id;
        private Long totalPrice;

        // ...

        public OrderSummary(Integer id, Long totalPrice) { // (2)
            super();
            this.id = id;
            this.totalPrice = totalPrice;
        }

        // ...

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Create JavaBean for storing query results.
    * - | (2)
      - | Create a constructor for generating objects using query execution results.

- Repository interface

 .. code-block:: java

    // (3)
    @Query("SELECT NEW x.y.z.domain.model.OrderSummary(o.id, SUM(i.price*oi.quantity))"
            + " FROM Order o LEFT JOIN o.orderItems oi LEFT JOIN oi.item i"
            + " GROUP BY o.id ORDER BY o.id DESC")
    List<OrderSummary> findOrderSummaries();

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (3)
      - | Specify the constructor created in (2).
        | Specify the constructor by FQCN after "NEW" keyword.


|

Setting Audit properties
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
This section introduces a mechanism to set values to Audit properties (Created By, Created Date-Time, Last Modified By, Last Modified Date-Time) of persistence layer and method to apply the same.

Spring Data JPA provides a mechanism to set values to Audit properties for newly created entities as well as modified entities.
This mechanism facilitates separation of logic of setting values to Audit properties from application code such as Service etc.

 .. note:: **Reasons to separate the logic to set values to Audit properties from application code**

    #. Setting the values to Audit properties is generally not an application requirement, but is essential as per the audit requirements of data.
       This logic does not essentially belong to application code such as Service etc.;
       hence it is better to separate it from application code.

    #. As per JPA specifications, only when the values of the entities fetched from persistence layer change, the changes will be reflected in the persistence layer (by executing the SQL),
       thereby avoiding the unnecessary access to persistence layer.
       If the values are set unconditionally in Audit properties in the application code such as Service,
       even if only Audit properties change, the changes are reflected in persistence layer making the effective JPA functionality ineffective.
       This problem can be avoided if the values are set in Audit properties only when if they are changed;
       however it cannot be recommended as it makes the application code complex.

 .. note:: **When Audit column of the persistence layer needs to be updated irrespective of the change in values**

    If updating the Audit columns (Last Modified By, Last Modified Date-Time) even if there is no change in values is a part of application specifications,
    then it becomes necessary to set Audit properties in application code such as Service etc.
    
    However, in such a case, the data modeling or application specifications are likely to be incorrect; hence it is better to revise these specifications.

|

See the example below.

- Entity class

 .. code-block:: java

    public class XxxEntity implements Serializable {
        private static final long serialVersionUID = 1L;

        // ...

        @Column(name = "created_by")
        @CreatedBy // (1)
        private String createdBy;

        @Column(name = "created_date")
        @CreatedDate // (2)
        @Type(type = "org.jadira.usertype.dateandtime.joda.PersistentDateTime") // (3)
        private DateTime createdDate; // (4)

        @Column(name = "last_modified_by")
        @LastModifiedBy // (5)
        private String lastModifiedBy;

        @Column(name = "last_modified_date")
        @LastModifiedDate // (6)
        @Type(type = "org.jadira.usertype.dateandtime.joda.PersistentDateTime") // (3)
        private DateTime lastModifiedDate; // (4)

        // ...

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Assign ``@org.springframework.data.annotation.CreatedBy`` annotation to the field that stores "Created by".
    * - | (2)
      - | Assign ``@org.springframework.data.annotation.CreatedDate`` annotation to the field that stores "Created date-time".
    * - | (3)
      - | When ``org.joda.time.DateTime`` type is to be used, assign ``@org.hibernate.annotations.Type`` annotation to the field in order to handle in Hibernate.
        | type attribute is fixed to ``"org.jadira.usertype.dateandtime.joda.PersistentDateTime"``. This is applicable for "Last modified date-time" field also.
    * - | (4)
      - | Field type that stores the "Created date-time" supports ``org.joda.time.DateTime``, ``java.util.Date``, ``java.util.Calendar``, ``java.lang.Long``, ``long`` types as well Date and Time APIs introduced in Java 8.
        | This is applicable for "Last modified date" field also.
    * - | (5)
      - | Assign ``@org.springframework.data.annotation.LastModifiedBy`` annotation to the field type that stores "Last modified by".
    * - | (6)
      - | Assign ``@org.springframework.data.annotation.LastModifiedDate`` annotation to the field that stores the "Last modified date-time".

 .. warning::

    ``@Type`` annotation is a Hibernate specific annotation and not a standard JPA annotation.


- AuditorAware interface implementation class

 .. code-block:: java

    // (7)
    @Component // (8)
    public class SpringSecurityAuditorAware implements AuditorAware<String> {

        // (9)
        public String getCurrentAuditor() {
            Authentication authentication = SecurityContextHolder.getContext()
                    .getAuthentication();
            if (authentication == null || !authentication.isAuthenticated()) {
                return null;
            }
            return ((UserDetails) authentication.getPrincipal()).getUsername();
        }

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (7)
      - | Create ``org.springframework.data.domain.AuditorAware`` interface implementation class.
        | ``AuditorAware`` interface is used for resolving the entity operator (Created by or Last Modified by).
        | This class should be created for each project.
    * - | (8)
      - | By assigning ``@Component`` annotation, this class comes under the target of component-scan.
        | Without assigning ``@Component`` annotation, it is also ok to define a bean of this class in bean definition file.
    * - | (9)
      - | Return the object to be set in the properties of entity operator (Created by or Last Modified by).
        | In the above example, login user name (string) authenticated by SpringSecurity is returned as entity operator.

- Object/relational mapping file( ``orm.xml`` )

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>

    <entity-mappings xmlns="http://java.sun.com/xml/ns/persistence/orm"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/persistence/orm
        http://java.sun.com/xml/ns/persistence/orm_2_0.xsd"
        version="2.0">

        <persistence-unit-metadata>
            <persistence-unit-defaults>
                <entity-listeners>
                    <entity-listener
                        class="org.springframework.data.jpa.domain.support.AuditingEntityListener" /> <!-- (10) -->
                </entity-listeners>
            </persistence-unit-defaults>
        </persistence-unit-metadata>

    </entity-mappings>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (10)
      - | Specify ``org.springframework.data.jpa.domain.support.AuditingEntityListener`` class as Entity Listener.
        | Values are set in Audit properties of the methods implemented in this class.

- ``infra.xml``

 .. code-block:: xml

    <jpa:auditing auditor-aware-ref="springSecurityAuditorAware" /> <!-- (11) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (11)
      - | Specify bean of class for resolving entity operator created in (7) in auditor-aware-ref attribute of <jpa:auditing> element.
        | In the above example, component-scan is performed for ``SpringSecurityAuditorAware`` implementation class; hence bean name ``"springSecurityAuditorAware"`` is specified.

|

As default implementation, the values of ``java.util.Calendar`` instance returned by ``getNow()`` method of 
``org.springframework.data.auditing.CurrentDateTimeProvider`` are used for the values set in the fields with 
``@CreatedDate``  and ``@LastModifiedDate`` annotations.

Extended example of changing the method to create the values to be used is shown below.

 .. code-block:: java

    // (1)
    @Component // (2)
    public class AuditDateTimeProvider implements DateTimeProvider {

        @Inject
        JodaTimeDateFactory dateFactory;

        // (3)
        @Override
        public Calendar getNow() {
            DateTime currentDateTime = dateFactory.newDateTime();
            return currentDateTime.toGregorianCalendar();
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Create ``org.springframework.data.auditing.DateTimeProvider`` interface implementation class.
    * - | (2)
      - | Assign ``@Component`` annotation to  perform component-scan.
        | Bean can be defined in Bean definition file without assigning ``@Component`` annotation.
    * - | (3)
      - | Return the instances to be set in the properties of entity operation date-time (Created Date-Time or Last Modified Date-Time).
        | In the above example, the instances fetched from ``org.terasoluna.gfw.common.date.jodatime.JodaTimeDateFactory`` of common library are returned as Operation Date-Time.
        | For details on ``JodaTimeDateFactory``, refer to ":doc:`../GeneralFuncDetail/SystemDate`".

- ``infra.xml``

 .. code-block:: xml

    <jpa:auditing
        auditor-aware-ref="springSecurityAuditorAware"
        date-time-provider-ref="auditDateTimeProvider" /> <!-- (4) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (4)
      - | In date-time-provider-ref attribute of <jpa:auditing> element, specify the bean of class to return the value set in entity operation date-time created in (1).
        | In the above example, component-scan is performed for ``AuditDateTimeProvider`` implementation class; hence bean name ``"auditDateTimeProvider"`` is specified.

|


Adding common conditions to JPQL to fetch entities from persistence layer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
This section introduces a mechanism to add common conditions to JPQL for fetching the entities from persistence layer (DB).
This is an extended Hibernate function and not included in standard JPA specifications.

 .. note:: **Assumed cases**

   As per data monitoring and data storage period requirements, when an entity is to be deleted, the application should be designed such that it will delete the record logically (UPDATE of Logical Delete flag) and will not delete (DELETE) the record physically.
   In case of such applications, the records deleted logically need to be uniformly excluded from the search target; hence rather than writing the condition to exclude logically deleted records in each and every query, it is better to specify it as a common condition.

 .. warning:: **Applicable scope**

    It should be noted that the specified conditions will be added for all queries executed to operate the entities for which this mechanism is used. This mechanism cannot be used even if there is a single Query for which conditions are not to be added.

|

Adding common conditions in JPQL to fetch entities
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
The method to add common conditions for JPQL which is executed at the time of calling Repository interface methods is as follows:

- Entity

 .. code-block:: java

    @Entity
    @Table(name = "t_order")
    @Where(clause = "is_logical_delete = false") // (1)
    public class Order implements Serializable {
        // ...
        @Id
        private Integer id;
        // ...
    }

- SQL executed at the time of calling the findOne method of Repository interface

 .. code-block:: sql

    SELECT
            -- ....
        FROM
            t_order order0_
       WHERE
            order0_.id = 1
            AND (
                order0_.is_logical_delete = false -- (2)
            );


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Assign ``@org.hibernate.annotations.Where`` annotation as entity class annotation and specify the common conditions in clause attribute.
        | The WHERE clause should be specified in SQL instead of JPQL i.e. it is necessary to specify the column name instead of the property name of Java object.
    * - | (2)
      - | The condition specified with ``@Where`` annotation is added.

 .. note:: **About Dialect extension**

    If SQL specific keywords are specified in ``@Where`` annotation, Hibernate may recognize SQL specific keywords as a common string value and as a result expected SQL may not get generated.
    It is necessary to extend the ``Dialect`` in case of SQL specific keywords are used in ``@Where`` annotation.

- Extending ``Dialect`` to register standard keywords such as ``true``, ``false`` and ``unknown``.

 .. code-block:: java

    package com.example.infra.hibernate;
    
    public class ExtendedPostgreSQL9Dialect extends PostgreSQL9Dialect { // (1)
        public ExtendedPostgreSQL9Dialect() {
            super();
            // (2)
            registerKeyword("true");
            registerKeyword("false");
            registerKeyword("unknown");
        }
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1
    
    * - Sr. No.
      - Description
    * - | (1)
      - | By default Hiberanate-4.3 may not correctly process some of the SQL keywords. The BOOLEAN type keywords such as ``true``, ``false`` and ``unknown`` are not registered in PostgreSQL dialect ``org.hibernate.dialect.PostgreSQL9Dialect``. Therefore such keywords are recognized as a common string value and as a result incorrect SQL may get generated.
        | It is necessary to extend ``org.hibernate.dialect.Dialect`` dialect in order to register such keywords.
    * - | (2)
      - | Register the SQL keywords that are likely to be used in ``@Where`` annotation.

- Settings of extended Dialect

 .. code-block:: xml

    <bean id="jpaVendorAdapter"
        class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
        <property name="databasePlatform" value="com.example.infra.hibernate.ExtendedPostgreSQL9Dialect"/> // (3)
        // ...
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1
    
    * - Sr. No.
      - Description
    * - | (3)
      - | The extended ``Dialect`` is set as the value of ``databasePlatform`` property in ``JpaVendorAdapter`` of ``EntityManager``.
      
 .. note:: **Class that can be specified**

    ``@Where`` annotation is valid only in the class with ``@Entity``.
    i.e. it is not assigned to SQL even if it is assigned to the base entity class with ``@javax.persistence.MappedSuperclass``.

 .. warning::

    ``@Where`` annotation is a Hibernate specific annotation and not the standard JPA annotation.

|

Adding common conditions to JPQL to fetch the related-entities
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
The method for adding common conditions for JPQL is shown below. JPQL is used for fetching the related-entities of the parent-entity which is fetched by calling Repository interface methods.

- Entity

 .. code-block:: java

    @Entity
    @Table(name = "t_order")
    @Where(clause = "is_logical_delete = false")
    public class Order implements Serializable {
        // ...
        @Id
        private Integer id;

        @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
        @OrderBy
        @Where(clause="is_logical_delete = false") // (1)
        private Set<OrderItem> orderItems;
        // ...

    }

- SQL (Lazy Load) generated when accessing the related-entity

 .. code-block:: sql

    SELECT
            -- ...
        FROM
            t_order_item orderitems0_
        WHERE
            (orderitems0.is_logical_delete = false) -- (2)
            AND orderitems0_.order_id = 1
        ORDER BY
            orderitems0_.item_code ASC
            ,orderitems0_.order_id ASC;

- SQL(Eager Load/Join Fetch) generated at the time of fetching parent-entity and its related-entity simultaneously

 .. code-block:: sql

    SELECT
        -- ...
        FROM
            t_order order0_
                LEFT OUTER JOIN t_order_item orderitems1_
                    ON order0_.id = orderitems1_.order_id
                    AND (orderitems1_.is_logical_delete = false) -- (2)
        WHERE
            order0_.id = 1
            AND (
                order0_.is_logical_delete = false
            )
        ORDER BY
            orderitems1_.item_code ASC
            ,orderitems1_.order_id ASC;

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Assign ``@org.hibernate.annotations.Where`` annotation as a related-entity field allocation and specify common conditions in clause attribute.
        | The WHERE clause should be specified in SQL format instead of JPQL i.e. it is necessary to specify the column name instead of the property name of Java object.
    * - | (2)
      - | The condition specified with ``@Where`` annotation is added.

 .. note:: **Types of related-entities that can be specified**

    Adding common conditions to SQL generated at the time fetching the related-entities is possible only for the entities having ``@OneToMany`` and ``@ManyToMany`` relationship.

 .. warning::

    ``@Where`` annotation is a Hibernate specific annotation and not the standard JPA annotation.

|

How to use multiple PersistenceUnits
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo::

 **TBD**

    Following topic will be added later.

    * Examples for using multiple PersistenceUnits

|

How to use Native query
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo::

 **TBD**

    Following topic will be added later.

    * Examples of query that uses Native query

.. raw:: latex

   \newpage

