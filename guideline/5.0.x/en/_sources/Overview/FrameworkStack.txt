Stack of TERASOLUNA Global Framework
================================================================================

.. only:: html

 .. contents:: Index
    :depth: 3
    :local:

Summary of Software Framework of TERASOLUNA Global Framework
--------------------------------------------------------------------------------

Software Framework being used in TERASOLUNA Global Framework is not a proprietory Framework but a combination of various OSS technologies around \ `Spring Framework <http://projects.spring.io/spring-framework/>`_\ .

.. figure:: images/introduction-software-framework.png
   :width: 80%


Main Structural Elements of Software Framework
--------------------------------------------------------------------------------
Libraries which constitute TERASOLUNA Global Framework are as follows:

.. figure:: images/introduction-software-stack.png
   :width: 80%

DI Container
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Spring is used as DI Container.


* `Spring Framework 4.1 <http://docs.spring.io/spring/docs/4.1.4.RELEASE/spring-framework-reference/html/beans.html>`_

MVC Framework
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Spring MVC is used as Web MVC Framework.

* `Spring MVC 4.1 <http://docs.spring.io/spring/docs/4.1.4.RELEASE/spring-framework-reference/html/mvc.html>`_

O/R Mapper
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This guideline assumes the use of **any one of the below**.

* `JPA2.0 <http://download.oracle.com/otn-pub/jcp/persistence-2.0-fr-eval-oth-JSpec/persistence-2_0-final-spec.pdf>`_

  * \ `Hibernate 4.2 <http://docs.jboss.org/hibernate/orm/4.2/manual/en-US/html/>`_\  is used as provider.

* `MyBatis 2.3.5 <https://mybatis.googlecode.com/files/MyBatis-SqlMaps-2_en.pdf>`_

  * DAO(TERASOLUNA DAO) of \ `TERASOLUNA Framework <http://sourceforge.jp/projects/terasoluna/releases/?package_id=6896>`_\  is used as wrapper.

.. todo::

  MyBatis 3 is planned to be included here. 

.. note::

  To be precise MyBatis is "SQL Mapper", but it is classified as "O/R Mapper" in this guidelines.

.. warning::

  Not every project must adopt JPA. For situations in which table design has been done and "Most of the tables are not normalized", "The number of columns in the table is too large" etc, use of JPA is difficult.

  Further, this guideline does not explain the basic usage of JPA. Hence, it is pre-requisite to have JPA experience people in the team.

View
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
JSP is used as View.

If standardize layout of view, use the following.

* `Apache Tiles 2.2 <http://tiles.apache.org/2.2/framework/index.html>`_



Security
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Spring Security is used as the framework for Authentication and Authorization.

* `Spring Security 3.2 <http://projects.spring.io/spring-security/>`_

.. todo::

  Update to Spring Security 3.2 is planned in future.

Validation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* For Single item input check, \ `BeanValidation 1.1 <http://download.oracle.com/otn-pub/jcp/bean_validation-1_1-fr-eval-spec/bean-validation-specification.pdf>`_\  is used.

  * For implementation, \ `Hibernate Validator 5.1 <http://docs.jboss.org/hibernate/validator/5.1/reference/en-US/html/>`_\  is used.

* For correlated items check, \ `BeanValidation <http://download.oracle.com/otn-pub/jcp/bean_validation-1_1-fr-eval-spec/bean-validation-specification.pdf>`_\  or \ `Spring Validation <http://docs.spring.io/spring/docs/4.1.4.RELEASE/spring-framework-reference/html/validation.html>`_

  * Refer to \ :doc:`../ArchitectureInDetail/Validation`\  for determining which of the two is to be used in which sitation. 



Logging
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* for Logger API, \ `SLF4J <http://www.slf4j.org>`_\  is used.

  * For implementation of Logger, \ `Logback <http://logback.qos.ch/>`_\  is used. 


Common Library
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* \ `https://github.com/terasolunaorg/terasoluna-gfw <https://github.com/terasolunaorg/terasoluna-gfw>`_\
* Refer to \ :ref:`frameworkstack_common_library`\  for details.

OSS Versions
--------------------------------------------------------------------------------

List of OSS being used in version 1.0.1.RELEASE.

.. tabularcolumns:: |p{0.20\linewidth}|p{0.25\linewidth}|p{0.25\linewidth}|p{0.25\linewidth}|p{0.05\linewidth}|
.. list-table::
    :header-rows: 1
    :stub-columns: 1
    :widths: 20 25 25 25 5

    * - Type
      - GroupId
      - ArtifactId
      - Version
      - Remarks
    * - Spring
      - org.springframework
      - spring-aop
      - 3.2.10.RELEASE
      -
    * - Spring
      - org.springframework
      - spring-aspects
      - 3.2.10.RELEASE
      -
    * - Spring
      - org.springframework
      - spring-beans
      - 3.2.10.RELEASE
      -
    * - Spring
      - org.springframework
      - spring-context
      - 3.2.10.RELEASE
      -
    * - Spring
      - org.springframework
      - spring-context-support
      - 3.2.10.RELEASE
      -
    * - Spring
      - org.springframework
      - spring-core
      - 3.2.10.RELEASE
      -
    * - Spring
      - org.springframework
      - spring-expression
      - 3.2.10.RELEASE
      -
    * - Spring
      - org.springframework
      - spring-jdbc
      - 3.2.10.RELEASE
      -
    * - Spring
      - org.springframework
      - spring-orm
      - 3.2.10.RELEASE
      -
    * - Spring
      - org.springframework
      - spring-tx
      - 3.2.10.RELEASE
      -
    * - Spring
      - org.springframework
      - spring-web
      - 3.2.10.RELEASE
      -
    * - Spring
      - org.springframework
      - spring-webmvc
      - 3.2.10.RELEASE
      -
    * - Spring
      - org.springframework.data
      - spring-data-commons
      - 1.6.4.RELEASE
      -
    * - Spring
      - org.springframework.security
      - spring-security-acl
      - 3.1.4.RELEASE
      -
    * - Spring
      - org.springframework.security
      - spring-security-config
      - 3.1.4.RELEASE
      -
    * - Spring
      - org.springframework.security
      - spring-security-core
      - 3.1.4.RELEASE
      -
    * - Spring
      - org.springframework.security
      - spring-security-taglibs
      - 3.1.4.RELEASE
      -
    * - Spring
      - org.springframework.security
      - spring-security-web
      - 3.1.4.RELEASE
      -
    * - JPA(Hibernate)
      - antlr
      - antlr
      - 2.7.7
      - \*1
    * - JPA(Hibernate)
      - dom4j
      - dom4j
      - 1.6.1
      - \*1
    * - JPA(Hibernate)
      - org.hibernate
      - hibernate-core
      - 4.2.3.Final
      - \*1
    * - JPA(Hibernate)
      - org.hibernate
      - hibernate-entitymanager
      - 4.2.3.Final
      - \*1
    * - JPA(Hibernate)
      - org.hibernate.common
      - hibernate-commons-annotations
      - 4.0.2.Final
      - \*1
    * - JPA(Hibernate)
      - org.hibernate.javax.persistence
      - hibernate-jpa-2.0-api
      - 1.0.1.Final
      - \*1
    * - JPA(Hibernate)
      - org.javassist
      - javassist
      - 3.15.0-GA
      - \*1
    * - JPA(Hibernate)
      - org.jboss.spec.javax.transaction
      - jboss-transaction-api_1.1_spec
      - 1.0.1.Final
      - \*1
    * - JPA(Hibernate)
      - org.springframework.data
      - spring-data-jpa
      - 1.4.3.RELEASE
      - \*1
    * - MyBatis2
      - jp.terasoluna.fw
      - terasoluna-dao
      - 2.0.5.0
      - \*2
    * - MyBatis2
      - jp.terasoluna.fw
      - terasoluna-ibatis
      - 2.0.5.0
      - \*2
    * - MyBatis2
      - org.mybatis
      - mybatis
      - 2.3.5
      - \*2
    * - DI
      - javax.inject
      - javax.inject
      - 1
      -
    * - AOP
      - aopalliance
      - aopalliance
      - 1
      -
    * - AOP
      - org.aspectj
      - aspectjrt
      - 1.7.4
      -
    * - AOP
      - org.aspectj
      - aspectjweaver
      - 1.7.4
      -
    * - Log Output
      - ch.qos.logback
      - logback-classic
      - 1.0.13
      -
    * - Log Output
      - ch.qos.logback
      - logback-core
      - 1.0.13
      -
    * - Log Output
      - org.lazyluke
      - log4jdbc-remix
      - 0.2.7
      -
    * - Log Output
      - org.slf4j
      - jcl-over-slf4j
      - 1.7.5
      -
    * - Log Output
      - org.slf4j
      - slf4j-api
      - 1.7.5
      -
    * - JSON
      - org.codehaus.jackson
      - jackson-core-asl
      - 1.9.7
      -
    * - JSON
      - org.codehaus.jackson
      - jackson-mapper-asl
      - 1.9.7
      -
    * - Input check
      - javax.validation
      - validation-api
      - 1.0.0.GA
      -
    * - Input check
      - org.hibernate
      - hibernate-validator
      - 4.3.1.Final
      -
    * - Input check
      - org.jboss.logging
      - jboss-logging
      - 3.1.0.GA
      -
    * - Bean conversion
      - commons-beanutils
      - commons-beanutils
      - 1.8.3
      - \*3
    * - Bean conversion
      - net.sf.dozer
      - dozer
      - 5.4.0
      - \*3
    * - Bean conversion
      - org.apache.commons
      - commons-lang3
      - 3.1
      - \*3
    * - Date conversion
      - joda-time
      - joda-time
      - 2.2
      -
    * - Date conversion
      - joda-time
      - joda-time-jsptags
      - 1.1.1
      - \*3
    * - Date conversion
      - org.jadira.usertype
      - usertype.core
      - 3.0.0.GA
      - \*1
    * - Date conversion
      - org.jadira.usertype
      - usertype.spi
      - 3.0.0.GA
      - \*1
    * - Connection pool
      - commons-dbcp
      - commons-dbcp
      - 1.2.2.patch_DBCP264_DBCP372
      - \*3
    * - Connection pool
      - commons-pool
      - commons-pool
      - 1.6
      - \*3
    * - Tiles
      - commons-digester
      - commons-digester
      - 2
      - \*3
    * - Tiles
      - org.apache.tiles
      - tiles-api
      - 2.2.2
      - \*3
    * - Tiles
      - org.apache.tiles
      - tiles-core
      - 2.2.2
      - \*3
    * - Tiles
      - org.apache.tiles
      - tiles-jsp
      - 2.2.2
      - \*3
    * - Tiles
      - org.apache.tiles
      - tiles-servlet
      - 2.2.2
      - \*3
    * - Tiles
      - org.apache.tiles
      - tiles-template
      - 2.2.2
      - \*3
    * - Utility
      - com.google.guava
      - guava
      - 13.0.1
      -
    * - Utility
      - commons-collections
      - commons-collections
      - 3.2.1
      - \*3
    * - Utility
      - commons-io
      - commons-io
      - 2.4
      - \*3
    * - Servlet
      - javax.servlet
      - jstl
      - 1.2
      -

#. Dependent libraries, when JPA is used for data access.
#. Dependent libraries, when MyBatis2 is used for data access.
#. Libraries which are not dependent on Common Library, but recommended in case of application development using TERASOLUNA Global Framework.


.. _frameworkstack_common_library:


Building blocks of Common Library
--------------------------------------------------------------------------------

\ `Common Library <https://github.com/terasolunaorg/terasoluna-gfw>`_\  includes ``+ alpha`` functionalities which are not available in Spring Ecosystem or other dependent libraries included in TERASOLUNA Global Framework. 
Basically, application development is possible using TERASOLUNA Global Framework even without this library. It is a "nice to have" kind of existence. 

.. tabularcolumns:: |p{0.05\linewidth}|p{0.30\linewidth}|p{0.35\linewidth}|p{0.30\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 5 30 35 30

    * - No.
      - Project Name
      - Summary
      - Java source-code availability
    * - | (1)
      - | terasoluna-gfw-common
      - | general-purpose functionality irrespective of Web
      - | Yes
    * - | (2)
      - | terasoluna-gfw-jodatime
      - | Functionality that depend on the Joda Time
      - | Yes
    * - | (3)
      - | terasoluna-gfw-web
      - | Group of functionalities for creating web application
      - | Yes
    * - | (4)
      - | terasoluna-gfw-jpa
      - | Dependency definition for using JPA
      - | No
    * - | (5)
      - | terasoluna-gfw-mybatis2
      - | Dependency definition for using MyBatis2
      - | No
    * - | (6)
      - | terasoluna-gfw-security-core
      - | Dependency definition for using Spring Security (other than Web).
      - | No
    * - | (7)
      - | terasoluna-gfw-security-web
      - | Dependency definition for using Spring Security (related to Web) and extended classes of Spring Security.
      - | Yes

The project which does not contain the Java source code, only defines library dependencies.



terasoluna-gfw-common
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

terasoluna-gfw-common provide following components.

.. tabularcolumns:: |p{0.30\linewidth}|p{0.35\linewidth}|p{0.30\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 30 50

    * - Classification
      - Component Name
      - Description
    * - :doc:`../ArchitectureInDetail/ExceptionHandling`
      - Exception Class
      - Provide general exception classes.
    * -
      - Exception Logger
      - Provide logger class for logging the exception that occurred in application.
    * -
      - Exception Code
      - Provide mechanism (classes) for resolving the exception code (message ID) that corresponds to the exception class.
    * -
      - Exception Logging Interceptor
      - Provide interceptor class of AOP for logging the exception that occurred in domain layer.
    * - :doc:`../ArchitectureInDetail/SystemDate`
      - System Date Time Factory
      - Provide classes for retrieving the system date time.
    * - :doc:`../ArchitectureInDetail/Codelist`
      - CodeList
      - Provide classes for generating CodeList.
    * - :doc:`../ArchitectureInDetail/DataAccessCommon`
      - Query Escape
      - Provide class for escape processing of value to bind into the SQL and JPQL.
    * -
      - Sequencer
      - Provide classes for retrieving the sequence value.

terasoluna-gfw-jodatime
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

terasoluna-gfw-jodatime provide following components.

.. tabularcolumns:: |p{0.30\linewidth}|p{0.35\linewidth}|p{0.30\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 30 50

    * - Classification
      - Component Name
      - Description
    * - :doc:`../ArchitectureInDetail/SystemDate`
      - System Date Time Factory for Joda Time
      - Provide classes for retrieving the system date time using the Joda Time API.

terasoluna-gfw-web
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

terasoluna-gfw-web provide following components.



.. tabularcolumns:: |p{0.30\linewidth}|p{0.35\linewidth}|p{0.30\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 30 50

    * - Classification
      - Component Name
      - Description
    * - :doc:`../ArchitectureInDetail/DoubleSubmitProtection`
      - Transaction Token Check
      - Provide mechanism (classes) for protecting Web Application from double submitting of request.
    * - :doc:`../ArchitectureInDetail/ExceptionHandling`
      - Exception Handler
      - Provide exception handler class(sub class of class that provided by Spring MVC) for integrating with exception handling components that provided from common library.
    * -
      - Exception Logging Interceptor
      - Provide interceptor class of AOP for logging the exception that handled by Spring MVC.
    * - :doc:`../ArchitectureInDetail/Codelist`
      - Populate CodeList interceptor
      - Provide interceptor class of Spring MVC for storing CodeList information into request scope, for the purpose of retrieving CodeList from View.
    * - :doc:`../ArchitectureInDetail/FileDownload`
      - General Download View
      - Provide abstract class for retrieving data from input stream and writing to stream for download.
    * - :doc:`../ArchitectureInDetail/Logging`
      - ServletFilter for storing Tracking ID
      - Provide Servlet Filter class for setting Tracking ID into MDC(Mapped Diagnostic Context) and request scope and response header, for the purpose of improving traceability.
        (If does not exist a Tracking ID in request header, generate a Tracking ID by this component)
    * -
      - General ServletFilter for storing to MDC
      - Provide abstract class for storing any value into Logger's MDC
    * -
      - ServletFilter for clearing MDC
      - Provide ServletFilter class for clearing information that stored in Logger's MDC.
    * - :doc:`../ArchitectureInDetail/Pagination`
      - JSP Tag for displaying Pagination Links
      - Provide JSP Tag Library for displaying Pagination Links using classes that provided by Spring Data Commons.
    * - :doc:`../ArchitectureInDetail/MessageManagement`
      - JSP Tag for displaying Result Messages
      - Provide JSP Tag Library for displaying Result Messages.
    * - :ref:`TagLibAndELFunctionsOverviewELFunctions`
      - EL Functions for XSS countermeasures
      - Provide EL Functions for XSS countermeasures.
    * -
      - EL Functions for URL
      - Provide EL Functions for URL as URL encoding.
    * -
      - EL Functions for DOM conversion
      - Provide EL Functions for DOM conversion.
    * -
      - EL Functions for Utilities
      - Provide EL Functions for general utilities processing.

terasoluna-gfw-security-web
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

terasoluna-gfw-security-web provide following components.

.. tabularcolumns:: |p{0.30\linewidth}|p{0.35\linewidth}|p{0.30\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 30 50

    * - Classification
      - Component Name
      - Description
    * - :doc:`../ArchitectureInDetail/Logging`
      - ServletFilter for storing name of authenticated user
      - Provide ServletFilter class for setting name of authenticated user into MDC, for the purpose of improving traceability.
    * - :doc:`../Security/Authentication`
      - Authentication Success Handler that can be specified redirect path
      - Provide Authentication Success Handler class that redirect to specified path in the Web Application when authentication is successful.


.. raw:: latex

   \newpage

