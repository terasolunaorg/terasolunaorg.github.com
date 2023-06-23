Stack of TERASOLUNA Server Framework for Java (5.x)
================================================================================

.. only:: html

 .. contents:: Index
    :depth: 3
    :local:

Summary of Software Framework of TERASOLUNA Server Framework for Java (5.x)
--------------------------------------------------------------------------------

Software Framework being used in TERASOLUNA Server Framework for Java (5.x) is not a proprietary Framework but a combination of various OSS technologies around \ `Spring Framework <http://projects.spring.io/spring-framework/>`_\ .

.. figure:: images/introduction-software-framework.png
   :width: 95%


Main Structural Elements of Software Framework
--------------------------------------------------------------------------------

Libraries which constitute TERASOLUNA Server Framework for Java (5.x) are as follows: 

.. figure:: images/introduction-software-stack.png
   :width: 95%

DI Container
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Spring Framework is used as DI Container.


* `Spring Framework 4.1 <http://docs.spring.io/spring/docs/4.1.7.RELEASE/spring-framework-reference/html/beans.html>`_

MVC Framework
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Spring MVC is used as Web MVC Framework.

* `Spring MVC 4.1 <http://docs.spring.io/spring/docs/4.1.7.RELEASE/spring-framework-reference/html/mvc.html>`_

O/R Mapper
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This guideline assumes the use of **any one of the below**.

* `MyBatis 3.2 <http://mybatis.github.io/mybatis-3/>`_

  * \ `MyBatis-Spring <http://mybatis.github.io/spring/>`_  is used as library for coordinating with Spring Framework.

* `JPA2.1 <http://download.oracle.com/otn-pub/jcp/persistence-2_1-fr-eval-spec/JavaPersistence.pdf>`_

  * \ `Hibernate 4.3 <http://docs.jboss.org/hibernate/orm/4.3/manual/en-US/html_single/>`_  is used as provider.

.. note::

  To be precise MyBatis is a "SQL Mapper", but it is classified as "O/R Mapper" in this guidelines.

.. warning::

  Every project must not adopt JPA. For situations in which table design has been done and "Most of the tables are not normalized", "The number of columns in the table is too large" etc, use of JPA is difficult.

  Furthermore, this guideline does not explain the basic usage of JPA. Hence, it is pre-requisite to have JPA experience people in the team.

View
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
JSP is used as View.

Use the following to standardize the view layout.

* `Apache Tiles 3.0 <http://tiles.apache.org/framework/index.html>`_



Security
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Spring Security is used as the framework for Authentication and Authorization.

* `Spring Security 3.2 <http://projects.spring.io/spring-security/>`_

.. tip::

    In addition to providing a mechanism of authentication and authorization in Spring Security 3.2,
    the mechanism has been enhanced to protect a Web application from malicious attackers.

    For mechanism to protect Web applications from malicious attackers, Refer, 

    * :doc:`../Security/CSRF`
    * :ref:`SpringSecurityAppendixSecHeaders`



Validation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* For Single item input check, \ `BeanValidation 1.1 <http://download.oracle.com/otn-pub/jcp/bean_validation-1_1-fr-eval-spec/bean-validation-specification.pdf>`_\  is used.

  * For implementation, \ `Hibernate Validator 5.1 <http://docs.jboss.org/hibernate/validator/5.1/reference/en-US/html/>`_\  is used.

* For correlated items check, \ `BeanValidation <http://download.oracle.com/otn-pub/jcp/bean_validation-1_1-fr-eval-spec/bean-validation-specification.pdf>`_\  or \ `Spring Validation <http://docs.spring.io/spring/docs/4.1.7.RELEASE/spring-framework-reference/html/validation.html>`_  is used.

  * Refer to \ :doc:`../ArchitectureInDetail/Validation`\  for determining the proper use. 



Logging
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* The \ `SLF4J <http://www.slf4j.org>`_\  API is used for Logger.

  * The \ `Logback <http://logback.qos.ch/>`_\  API is used for implementation of Logger.


Common Library
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* \ `https://github.com/terasolunaorg/terasoluna-gfw <https://github.com/terasolunaorg/terasoluna-gfw>`_\
* Refer to \ :ref:`frameworkstack_common_library`\  for details.

OSS Versions
--------------------------------------------------------------------------------

List of OSS being used in version 5.0.1.RELEASE.

.. tip::

    From version 5.0.0.RELEASE onwards, 
    adopted the mechanism of importing \ ``<dependencyManagement>`` \ of `Spring IO platform <http://platform.spring.io/platform/>`_\.

    By importing the \ ``<dependencyManagement>`` \ of Spring IO platform,

    * Spring Framework offering library
    * Spring Framework dependent OSS library
    * Spring Framework compatible OSS library

    dependencies resolved and 
    OSS version to be used in the TERASOLUNA Server Framework for Java (5.x) is following the rule of Spring IO platform definition.

    Furthermore, Spring IO platform version is `1.1.3.RELEASE <http://docs.spring.io/platform/docs/1.1.3.RELEASE/reference/htmlsingle/>`_  specified in version 5.0.1.RELEASE.

.. tabularcolumns:: |p{0.15\linewidth}|p{0.27\linewidth}|p{0.25\linewidth}|p{0.15\linewidth}|p{0.05\linewidth}|p{0.08\linewidth}|
.. list-table::
    :header-rows: 1
    :stub-columns: 1
    :widths: 15 27 25 15 5 8

    * - Type
      - GroupId
      - ArtifactId
      - Version
      - Spring IO platform
      - Remarks
    * - Spring
      - org.springframework
      - spring-aop
      - 4.1.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-aspects
      - 4.1.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-beans
      - 4.1.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-context
      - 4.1.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-context-support
      - 4.1.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-core
      - 4.1.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-expression
      - 4.1.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-jdbc
      - 4.1.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-orm
      - 4.1.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-tx
      - 4.1.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-web
      - 4.1.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-webmvc
      - 4.1.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework.data
      - spring-data-commons
      - 1.9.3.RELEASE
      - \*
      -
    * - Spring
      - org.springframework.security
      - spring-security-acl
      - 3.2.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework.security
      - spring-security-config
      - 3.2.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework.security
      - spring-security-core
      - 3.2.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework.security
      - spring-security-taglibs
      - 3.2.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework.security
      - spring-security-web
      - 3.2.7.RELEASE
      - \*
      -
    * - MyBatis3
      - org.mybatis
      - mybatis
      - 3.2.8
      -
      - \*1
    * - MyBatis3
      - org.mybatis
      - mybatis-spring
      - 1.2.2
      -
      - \*1
    * - JPA(Hibernate)
      - antlr
      - antlr
      - 2.7.7
      - \*
      - \*2
    * - JPA(Hibernate)
      - dom4j
      - dom4j
      - 1.6.1
      - \*
      - \*2
    * - JPA(Hibernate)
      - org.hibernate
      - hibernate-core
      - 4.3.10.Final
      - \*
      - \*2
    * - JPA(Hibernate)
      - org.hibernate
      - hibernate-entitymanager
      - 4.3.10.Final
      - \*
      - \*2
    * - JPA(Hibernate)
      - org.hibernate.common
      - hibernate-commons-annotations
      - 4.0.5.Final
      - \*
      - \*2 \*4
    * - JPA(Hibernate)
      - org.hibernate.javax.persistence
      - hibernate-jpa-2.1-api
      - 1.0.0.Final
      - \*
      - \*2 \*4
    * - JPA(Hibernate)
      - org.javassist
      - javassist
      - 3.18.1-GA
      - \*
      - \*2
    * - JPA(Hibernate)
      - org.jboss
      - jandex
      - 1.1.0.Final
      - \*
      - \*2 \*4
    * - JPA(Hibernate)
      - org.jboss.logging
      - jboss-logging-annotations
      - 1.2.0.Final
      - \*
      - \*2 \*4 \*5
    * - JPA(Hibernate)
      - org.jboss.spec.javax.transaction
      - jboss-transaction-api_1.2_spec
      - 1.0.0.Final
      - \*
      - \*2 \*4
    * - JPA(Hibernate)
      - org.springframework.data
      - spring-data-jpa
      - 1.7.3.RELEASE
      - \*
      - \*2
    * - DI
      - javax.inject
      - javax.inject
      - 1
      - \*
      -
    * - AOP
      - aopalliance
      - aopalliance
      - 1
      - \*
      -
    * - AOP
      - org.aspectj
      - aspectjrt
      - 1.8.6
      - \*
      -
    * - AOP
      - org.aspectj
      - aspectjweaver
      - 1.8.6
      - \*
      -
    * - Log output
      - ch.qos.logback
      - logback-classic
      - 1.1.3
      - \*
      -
    * - Log output
      - ch.qos.logback
      - logback-core
      - 1.1.3
      - \*
      - \*4
    * - Log output
      - org.lazyluke
      - log4jdbc-remix
      - 0.2.7
      -
      -
    * - Log output
      - org.slf4j
      - jcl-over-slf4j
      - 1.7.12
      - \*
      -
    * - Log output
      - org.slf4j
      - slf4j-api
      - 1.7.12
      - \*
      -
    * - JSON
      - com.fasterxml.jackson.core
      - jackson-annotations
      - 2.4.6
      - \*
      -
    * - JSON
      - com.fasterxml.jackson.core
      - jackson-core
      - 2.4.6
      - \*
      -
    * - JSON
      - com.fasterxml.jackson.core
      - jackson-databind
      - 2.4.6
      - \*
      -
    * - JSON
      - com.fasterxml.jackson.datatype
      - jackson-datatype-joda
      - 2.4.6
      - \*
      -
    * - Input check
      - javax.validation
      - validation-api
      - 1.1.0.Final
      - \*
      -
    * - Input check
      - org.hibernate
      - hibernate-validator
      - 5.1.3.Final
      - \*
      -
    * - Input check
      - org.jboss.logging
      - jboss-logging
      - 3.1.3.GA
      - \*
      - \*4
    * - Input check
      - com.fasterxml
      - classmate
      - 1.0.0
      - \*
      - \*4
    * - Bean conversion
      - commons-beanutils
      - commons-beanutils
      - 1.9.2
      - \*
      - \*3
    * - Bean conversion
      - net.sf.dozer
      - dozer
      - 5.5.1
      -
      - \*3
    * - Bean conversion
      - net.sf.dozer
      - dozer-spring
      - 5.5.1
      -
      - \*3
    * - Bean conversion
      - org.apache.commons
      - commons-lang3
      - 3.3.2
      - \*
      - \*3
    * - Date operation
      - joda-time
      - joda-time
      - 2.5
      - \*
      -
    * - Date operation
      - joda-time
      - joda-time-jsptags
      - 1.1.1
      -
      - \*3
    * - Date operation
      - org.jadira.usertype
      - usertype.core
      - 3.2.0.GA
      -
      - \*2
    * - Date operation
      - org.jadira.usertype
      - usertype.spi
      - 3.2.0.GA
      -
      - \*2
    * - Connection pool
      - org.apache.commons
      - commons-dbcp2
      - 2.0.1
      - \*
      - \*3
    * - Connection pool
      - org.apache.commons
      - commons-pool2
      - 2.2
      - \*
      - \*3
    * - Tiles
      - commons-digester
      - commons-digester
      - 2.1
      - \*
      - \*3
    * - Tiles
      - org.apache.tiles
      - tiles-api
      - 3.0.5
      - \*
      - \*3
    * - Tiles
      - org.apache.tiles
      - tiles-core
      - 3.0.5
      - \*
      - \*3
    * - Tiles
      - org.apache.tiles
      - tiles-jsp
      - 3.0.5
      - \*
      - \*3
    * - Tiles
      - org.apache.tiles
      - tiles-servlet
      - 3.0.5
      - \*
      - \*3
    * - Tiles
      - org.apache.tiles
      - tiles-template
      - 3.0.5
      - \*
      - \*3 \*4
    * - Tiles
      - org.apache.tiles
      - tiles-autotag-core-runtime
      - 1.1.0
      - \*
      - \*3 \*4
    * - Tiles
      - org.apache.tiles
      - tiles-request-servlet
      - 1.0.6
      - \*
      - \*3 \*4
    * - Tiles
      - org.apache.tiles
      - tiles-request-api
      - 1.0.6
      - \*
      - \*3
    * - Tiles
      - org.apache.tiles
      - tiles-request-jsp
      - 1.0.6
      - \*
      - \*3 \*4
    * - Utility
      - com.google.guava
      - guava
      - 17.0
      - \*
      -
    * - Utility
      - commons-collections
      - commons-collections
      - 3.2.1
      - \*
      - \*3
    * - Utility
      - commons-io
      - commons-io
      - 2.4
      - \*
      - \*3
    * - Servlet
      - org.apache.taglibs
      - taglibs-standard-jstlel
      - 1.2.5
      - \*
      -
    * - Servlet
      - org.apache.taglibs
      - taglibs-standard-spec
      - 1.2.5
      - \*
      - \*4
    * - Servlet
      - org.apache.taglibs
      - taglibs-standard-impl
      - 1.2.5
      - \*
      - \*4

#. | Dependent libraries, when MyBatis3 is used for data access.
#. | Dependent libraries, when JPA is used for data access.
#. | Libraries which are not dependent on Common Library, but recommended in case of application development using TERASOLUNA Server Framework for Java (5.x).
#. | Libraries that are supported by Spring IO platform, but library that relies individually.
   | (Library is not managed as dependencies in Spring IO platform)
#. | Library versions that are applied in the Spring IO platform is a Beta or RC (Release Candidate)
   | (Library that explicitly specify the GA version at TERASOLUNA Server Framework for Java (5.x))

.. _frameworkstack_common_library:


Building blocks of Common Library
--------------------------------------------------------------------------------

\ `Common Library <https://github.com/terasolunaorg/terasoluna-gfw>`_\  includes ``+ alpha`` functionalities which are not available in Spring Ecosystem or other dependent libraries included in TERASOLUNA Server Framework for Java (5.x).
Basically, application development is possible using TERASOLUNA Server Framework for Java (5.x) without this library but "convenient to have" kind of existence.

.. tabularcolumns:: |p{0.05\linewidth}|p{0.30\linewidth}|p{0.45\linewidth}|p{0.20\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 5 30 45 20

    * - No.
      - Project Name
      - Summary
      - Java source-code availability
    * - \ (1)
      - terasoluna-gfw-common
      - Provide general-purpose functionalities and dependency definitions irrespective of Web.
      - Yes
    * - \ (2)
      - terasoluna-gfw-jodatime
      - Provide functionalities and dependency definitions that depend on the Joda Time.
      - Yes
    * - \ (3)
      - terasoluna-gfw-web
      - Provide group of functionalities and dependency definitions for creating web application.
      - Yes
    * - \ (4)
      - terasoluna-gfw-mybatis3
      - Provide dependency definitions for using MyBatis3.
      - No
    * - \ (5)
      - terasoluna-gfw-jpa
      - Provide dependency definitions for using JPA.
      - No
    * - \ (6)
      - terasoluna-gfw-security-core
      - Provide dependency definitions for using Spring Security (other than Web).
      - No
    * - \ (7)
      - terasoluna-gfw-security-web
      - Provide dependency definitions for using Spring Security (related to Web) and extended classes of Spring Security.
      - Yes
    * - \ (8)
      - terasoluna-gfw-recommended-dependencies
      - Provide dependency definitions of recommended libraries that doesn't depends on web.
      - No
    * - \ (9)
      - terasoluna-gfw-recommended-web-dependencies
      - Provide dependency definitions of recommended libraries that depends on web.
      - No
    * - \ (10)
      - terasoluna-gfw-parent
      - Provide dependency libraries management and recommended settings of build plugins.
      - No

The project which does not contain the Java source code, only defines library dependencies.

In addition, project dependencies are as follows:

.. figure:: images_FrameworkStack/FrameworkStackProjectDependencies.png
    :width: 75%



terasoluna-gfw-common
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

terasoluna-gfw-common provide following components.

.. tabularcolumns:: |p{0.20\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
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

.. tabularcolumns:: |p{0.20\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
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



.. tabularcolumns:: |p{0.20\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
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
      - General ServletFilter for storing MDC
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

.. tabularcolumns:: |p{0.20\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
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

