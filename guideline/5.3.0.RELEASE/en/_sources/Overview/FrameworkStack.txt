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


* `Spring Framework 4.3 <http://docs.spring.io/spring/docs/4.3.5.RELEASE/spring-framework-reference/html/beans.html>`_

MVC Framework
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Spring MVC is used as Web MVC Framework.

* `Spring MVC 4.3 <http://docs.spring.io/spring/docs/4.3.5.RELEASE/spring-framework-reference/html/mvc.html>`_

O/R Mapper
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This guideline assumes the use of **any one of the below**.

* `MyBatis 3.4 <http://mybatis.github.io/mybatis-3/>`_

  * \ `MyBatis-Spring <http://mybatis.github.io/spring/>`_  is used as library for coordinating with Spring Framework.

* `JPA2.1 <http://download.oracle.com/otn-pub/jcp/persistence-2_1-fr-eval-spec/JavaPersistence.pdf>`_

  * \ `Hibernate 5.0 <http://docs.jboss.org/hibernate/orm/5.0/manual/en-US/html_single/>`_  is used as provider.

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

* `Spring Security 4.1 <http://projects.spring.io/spring-security/>`_

.. tip::

    In addition to providing a mechanism of authentication and authorization in Spring Security 3.2,
    the mechanism has been enhanced to protect a Web application from malicious attackers.

    For mechanism to protect Web applications from malicious attackers, Refer, 

    * :doc:`../Security/CSRF`
    * :doc:`../Security/LinkageWithBrowser`



Validation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* For Single item input check, \ `BeanValidation 1.1 <http://download.oracle.com/otn-pub/jcp/bean_validation-1_1-fr-eval-spec/bean-validation-specification.pdf>`_\  is used.

  * For implementation, \ `Hibernate Validator 5.2 <http://docs.jboss.org/hibernate/validator/5.2/reference/en-US/html/>`_\  is used.

* For correlated items check, \ `BeanValidation <http://download.oracle.com/otn-pub/jcp/bean_validation-1_1-fr-eval-spec/bean-validation-specification.pdf>`_\  or \ `Spring Validation <http://docs.spring.io/spring/docs/4.3.5.RELEASE/spring-framework-reference/html/validation.html>`_  is used.

  * Refer to \ :doc:`../ArchitectureInDetail/WebApplicationDetail/Validation`\  for determining the proper use. 



Logging
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* The \ `SLF4J <http://www.slf4j.org>`_\  API is used for Logger.

  * The \ `Logback <http://logback.qos.ch/>`_\  API is used for implementation of Logger.


Common Library
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* \ `https://github.com/terasolunaorg/terasoluna-gfw <https://github.com/terasolunaorg/terasoluna-gfw>`_\
* Refer to \ :ref:`frameworkstack_common_library`\  for details.

.. _frameworkstack_using_oss_version:

OSS Versions
--------------------------------------------------------------------------------

List of OSS being used in version 5.3.0.RELEASE.

.. tip::

    From version 5.0.0.RELEASE onwards, 
    adopted the mechanism of importing \ ``<dependencyManagement>`` \ of `Spring IO platform <http://platform.spring.io/platform/>`_\.

    By importing the \ ``<dependencyManagement>`` \ of Spring IO platform,

    * Spring Framework offering library
    * Spring Framework dependent OSS library
    * Spring Framework compatible OSS library

    dependencies resolved and 
    OSS version to be used in the TERASOLUNA Server Framework for Java (5.x) is following the rule of Spring IO platform definition.

    Furthermore, Spring IO platform version is `Athens-SR2 <http://docs.spring.io/platform/docs/Athens-SR2/reference/htmlsingle/>`_  specified in version 5.3.0.RELEASE.

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
      - 4.3.5.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-aspects
      - 4.3.5.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-beans
      - 4.3.5.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-context
      - 4.3.5.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-context-support
      - 4.3.5.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-core
      - 4.3.5.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-expression
      - 4.3.5.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-jdbc
      - 4.3.5.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-orm
      - 4.3.5.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-tx
      - 4.3.5.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-web
      - 4.3.5.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-webmvc
      - 4.3.5.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-jms
      - 4.3.5.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-messaging
      - 4.3.5.RELEASE
      - \*
      -
    * - Spring
      - org.springframework.data
      - spring-data-commons
      - 1.12.6.RELEASE
      - \*
      -
    * - Spring
      - org.springframework.security
      - spring-security-acl
      - 4.1.4.RELEASE
      - \*
      -
    * - Spring
      - org.springframework.security
      - spring-security-config
      - 4.1.4.RELEASE
      - \*
      -
    * - Spring
      - org.springframework.security
      - spring-security-core
      - 4.1.4.RELEASE
      - \*
      -
    * - Spring
      - org.springframework.security
      - spring-security-taglibs
      - 4.1.4.RELEASE
      - \*
      -
    * - Spring
      - org.springframework.security
      - spring-security-web
      - 4.1.4.RELEASE
      - \*
      -
    * - Spring  
      - org.springframework.security.oauth  
      - spring-security-oauth2  
      - 2.0.12.RELEASE  
      - \*  
      -
    * - MyBatis3
      - org.mybatis
      - mybatis
      - 3.4.2
      -
      - \*1
    * - MyBatis3
      - org.mybatis
      - mybatis-spring
      - 1.3.1
      -
      - \*1
    * - MyBatis3
      - org.mybatis
      - mybatis-typehandlers-jsr310
      - 1.0.2
      -
      - \*1*6
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
      - 5.0.11.Final
      - \*
      - \*2
    * - JPA(Hibernate)
      - org.hibernate
      - hibernate-entitymanager
      - 5.0.11.Final
      - \*
      - \*2
    * - JPA(Hibernate)
      - org.hibernate.common
      - hibernate-commons-annotations
      - 5.0.1.Final
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
      - 3.20.0-GA
      - \*
      - \*2
    * - JPA(Hibernate)
      - org.jboss
      - jandex
      - 2.0.0.Final
      - \*
      - \*2 \*4
    * - JPA(Hibernate)
      - org.springframework.data
      - spring-data-jpa
      - 1.10.6.RELEASE
      - \*
      - \*2
    * - JPA(Hibernate)
      - org.apache.geronimo.specs
      - geronimo-jta_1.1_spec
      - 1.1.1
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
      - 1.8.9
      - \*
      -
    * - AOP
      - org.aspectj
      - aspectjweaver
      - 1.8.9
      - \*
      -
    * - Log output
      - ch.qos.logback
      - logback-classic
      - 1.1.8
      - \*
      -
    * - Log output
      - ch.qos.logback
      - logback-core
      - 1.1.8
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
      - 1.7.22
      - \*
      -
    * - Log output
      - org.slf4j
      - slf4j-api
      - 1.7.22
      - \*
      -
    * - JSON
      - com.fasterxml.jackson.core
      - jackson-annotations
      - 2.8.5
      - \*
      -
    * - JSON
      - com.fasterxml.jackson.core
      - jackson-core
      - 2.8.5
      - \*
      -
    * - JSON
      - com.fasterxml.jackson.core
      - jackson-databind
      - 2.8.5
      - \*
      -
    * - JSON
      - com.fasterxml.jackson.datatype
      - jackson-datatype-joda
      - 2.8.5
      - \*
      -
    * - JSON
      - com.fasterxml.jackson.datatype
      - jackson-datatype-jsr310
      - 2.8.5
      - \*
      - \*6
    * - Input check
      - javax.validation
      - validation-api
      - 1.1.0.Final
      - \*
      -
    * - Input check
      - org.hibernate
      - hibernate-validator
      - 5.2.4.Final
      - \*
      -
    * - Input check
      - org.jboss.logging
      - jboss-logging
      - 3.3.0.Final
      - \*
      - \*4
    * - Input check
      - com.fasterxml
      - classmate
      - 1.3.3
      - \*
      - \*4
    * - Bean conversion
      - commons-beanutils
      - commons-beanutils
      - 1.9.3
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
      - 3.4
      - \*
      - \*3
    * - Date operation
      - joda-time
      - joda-time
      - 2.9.6
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
      - 5.0.0.GA
      -
      - \*2
    * - Date operation
      - org.jadira.usertype
      - usertype.spi
      - 5.0.0.GA
      -
      - \*2
    * - Connection pool
      - org.apache.commons
      - commons-dbcp2
      - 2.1.1
      - \*
      - \*3
    * - Connection pool
      - org.apache.commons
      - commons-pool2
      - 2.4.2
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
      - 3.0.7
      - \*
      - \*3
    * - Tiles
      - org.apache.tiles
      - tiles-core
      - 3.0.7
      - \*
      - \*3
    * - Tiles
      - org.apache.tiles
      - tiles-jsp
      - 3.0.7
      - \*
      - \*3
    * - Tiles
      - org.apache.tiles
      - tiles-servlet
      - 3.0.7
      - \*
      - \*3
    * - Tiles
      - org.apache.tiles
      - tiles-template
      - 3.0.7
      - \*
      - \*3 \*4
    * - Tiles
      - org.apache.tiles
      - tiles-autotag-core-runtime
      - 1.2
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
      - 3.2.2
      - \*
      - \*3
    * - Utility
      - commons-io
      - commons-io
      - 2.5
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
#. | A library which considers API of Java SE 8 and further versions,as a prerequisite.
   | Dependency relation to the library must be explicitly removed in case of versions earlier than Java SE 7


.. _frameworkstack_common_library:


Building blocks of Common Library
--------------------------------------------------------------------------------

\ `Common Library <https://github.com/terasolunaorg/terasoluna-gfw>`_\  includes ``+ alpha`` functionalities which are not available in Spring Ecosystem or other dependent libraries included in TERASOLUNA Server Framework for Java (5.x).
Basically, application development is possible using TERASOLUNA Server Framework for Java (5.x) without this library but "convenient to have" kind of existence.
With the default settings, provided two blank projects, \ `Blank project of multi-project <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank>`_\  and \ `Blank project of single-project <https://github.com/terasolunaorg/terasoluna-gfw-web-blank>`_\ , contains built-in Common Library as shown in the following listing. 

.. tabularcolumns:: |p{0.05\linewidth}|p{0.15\linewidth}|p{0.40\linewidth}|p{0.10\linewidth}|p{0.10\linewidth}|p{0.10\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 5 15 40 20 10 10
    :class: longtable

    * - No.
      - Project Name
      - Summary
      - Java source-code availability
      - Blank project of multi-project built-in
      - Blank project of single-project built-in
    * - \ (1)
      - terasoluna-gfw-parent
      - Provide recommended settings for dependent library management and build plug-in.
      - No
      - Yes*1
      - Yes*1
    * - \ (2)
      - terasoluna-gfw-common-libraries
      - Define structure of the project which includes Java source code, among the common libraries. It is not necessary to add to pom.xml as a dependency.
      - No
      - Yes*1
      - Yes*1
    * - \ (3)
      - terasoluna-gfw-dependencies
      - Define structure of the project which provides only dependency definition (other than terasoluna-gfw-parent), among the common libraries. It is not necessary to add pom.xml as a dependency.
      - No
      - L*1
      - L*1
    * - \ (4)
      - terasoluna-gfw-common
      - Provide a general purpose function that does not depend on Web. When this library is to be used, terasoluna-gfw-common-dependencies is added to pom.xml as a dependency.
      - Yes
      - Yes*2
      - Yes*2
    * - \ (5)
      - terasoluna-gfw-common-dependencies
      - Provide definition for dependency while using the function offered by terasoluna-gfw-common project.
      - No
      - Yes
      - Yes
    * - \ (6)
      - terasoluna-gfw-jodatime
      - Provide a function that depends on Joda Time. When the library is to be used, terasoluna-gfw-jodatime-dependencies is added to pom.xml as a dependency. (add from 5.0.0)
      - Yes
      - Yes*2
      - Yes*2
    * - \ (7)
      - terasoluna-gfw-jodatime-dependencies
      - Provide dependency definition while using a function offered by terasoluna-gfw-jodatime project.
      - No
      - Yes
      - Yes
    * - \ (8)
      - terasoluna-gfw-web
      - Provide a function used while creating a Web application. Functions that do not depend on View are consolidated. When the library is used, terasoluna-gfw-web-dependencies is added to pom.xml as a dependency.
      - Yes
      - Yes*2
      - Yes*2
    * - \ (9)
      - terasoluna-gfw-web-dependencies
      - Provide dependency definition while using a function offered by terasoluna-gfw-web project.
      - No
      - Yes
      - Yes
    * - \ (10)
      - terasoluna-gfw-web-jsp
      - Provide a function used while creating a Web application which adopts JSP in View. When this library is used, terasoluna-gfw-web-jsp-dependencies is added to pom.xml as a dependency.
      - Yes
      - Yes*2
      - Yes*2
    * - \ (11)
      - terasoluna-gfw-web-jsp-dependencies
      - Provide dependency definition while using a function offered by terasoluna-gfw-web-jsp project.
      - No
      - Yes
      - Yes
    * - \ (12)
      - terasoluna-gfw-security-web
      - Provide extended parts of Spring Security. When the library is used, terasoluna-gfw-security-web-dependencies is added to pom.xml as a dependency.
      - Yes
      - Yes*2
      - Yes*2
    * - \ (13)
      - terasoluna-gfw-security-web-dependencies
      - Provide dependency definition while using Spring Security (Web related) and dependency definition while using a function offered by terasoluna-gfw-security-web project.
      - No
      - Yes
      - Yes
    * - \ (14)
      - terasoluna-gfw-string
      - Provide a function related to string processing. (added from 5.1.0)
      - Yes
      - No
      - No
    * - \ (15)
      - terasoluna-gfw-codepoints
      - Provide a function to check whether the code points that constitute the targeted string are included in the code point set. (added from 5.1.0)
      - Yes
      - No
      - No
    * - \ (16)
      - terasoluna-gfw-validator
      - Provide by adding a constraint annotation of general-purpose Bean Validation. (added from 5.1.0)
      - Yes
      - No
      - No
    * - \ (17)
      - terasoluna-gfw-security-core-dependencies
      - Provide a dependency definition (other than Web) while using Spring Security.
      - No
      - Yes
      - Yes
    * - \ (18)
      - terasoluna-gfw-mybatis3-dependencies
      - Provide a dependency definition while using MyBatis3.
      - No
      - Yes*3
      - Yes*3
    * - \ (19)
      - terasoluna-gfw-jpa-dependencies
      - Provide a dependency definition while using JPA.
      - No
      - Yes*4
      - Yes*4
    * - \ (20)
      - terasoluna-gfw-recommended-dependencies
      - Provide a dependency definition for recommended libraries that do not depend on web.
      - No
      - Yes
      - Yes
    * - \ (21)
      - terasoluna-gfw-recommended-web-dependencies
      - Provide a dependency definition for web-dependent recommended libraries.
      - No
      - Yes
      - Yes

.. raw:: latex

   \newpage

#. | Incorporated as \ ``<parent>``\  element of each project and not as a \ ``<dependency>``\ element.
#. | Incorporated as a transition dependency from  \ ``<dependency>``\  element and not as a \ ``<dependency>``\ element.
#. | With the default settings of Common Library, when MyBatis3 is used for data access.
#. | With the default settings of Common Library, when JPA is used for data access.


The project which does not contain the Java source code, only defines library dependencies.

In addition, project dependencies are as follows:

.. figure:: images_FrameworkStack/FrameworkStackProjectDependencies.png
    :width: 75%

.. note::

  Excluding exceptions, a project assigned with "dependencies" at the end of project name exists in common library.
  (For example, terasoluna-gfw-common-dependencies etc corresponding to terasoluna-gfw-common)

  In these projects, since a dependency definition for OSS library recommended for use is provided along with dependency definition for common library,
  it is recommended to add it to pom.xml as a dependency, for the project assigned with "dependencies" while using a common library.
  

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
    * - :doc:`../ArchitectureInDetail/WebApplicationDetail/ExceptionHandling`
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
    * - :doc:`../ArchitectureInDetail/GeneralFuncDetail/SystemDate`
      - System Date Time Factory
      - Provide classes for retrieving the system date time.
    * - :doc:`../ArchitectureInDetail/WebApplicationDetail/Codelist`
      - CodeList
      - Provide classes for generating CodeList.
    * - :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessCommon`
      - Query Escape
      - Provide class for escape processing of value to bind into the SQL and JPQL.
    * -
      - Sequencer
      - Provide classes for retrieving the sequence value.

terasoluna-gfw-string
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

terasoluna-gfw-string provides following components.

.. tabularcolumns:: |p{0.20\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 30 50

    * - Classification
      - Component Name
      - Description
    * - :doc:`../ArchitectureInDetail/GeneralFuncDetail/StringProcessing`
      - Half width to full width conversion
      - Provide a class which carries out a process wherein half width character of input string is converted to full width and a process wherein full width character is converted to half width based on mapping table of half width string and full width string.


terasoluna-gfw-codepoints
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

terasoluna-gfw-codepoints provides following components.

.. tabularcolumns:: |p{0.20\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 30 50

    * - Classification
      - Component Name
      - Description
    * - :doc:`../ArchitectureInDetail/GeneralFuncDetail/StringProcessing`
      - Codepoint check
      - Provide a class which checks whether the codepoint constituting target string is incorporated in the codepoint set.
    * - :doc:`../ArchitectureInDetail/WebApplicationDetail/Validation`
      - Bean validation constraint annotation for codepoint check
      - Provide constraint annotation to carry out Bean validation for codepoint check.


terasoluna-gfw-validator
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

terasoluna-gfw-validator provides following components.

.. tabularcolumns:: |p{0.20\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 30 50

    * - Classification
      - Component name
      - Description
    * - :doc:`../ArchitectureInDetail/WebApplicationDetail/Validation`
      - Bean Validation constraint annotation for byte length check
      - Provide a constraint annotation to check whether the byte length in the character code of input value is less than or equal to specified maximum value, and greater than or equal to specified minimum value, using Bean Validation.
    * -
      - Bean Validation constraint annotation for field value comparison correlation check
      - Provide a constraint annotation to check magnitude correlation of two fields using Bean Validation.

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
    * - :doc:`../ArchitectureInDetail/GeneralFuncDetail/SystemDate`
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
    * - :doc:`../ArchitectureInDetail/WebApplicationDetail/DoubleSubmitProtection`
      - Transaction Token Check
      - Provide mechanism (classes) for protecting Web Application from double submitting of request.
    * - :doc:`../ArchitectureInDetail/WebApplicationDetail/ExceptionHandling`
      - Exception Handler
      - Provide exception handler class(sub class of class that provided by Spring MVC) for integrating with exception handling components that provided from common library.
    * -
      - Exception Logging Interceptor
      - Provide interceptor class of AOP for logging the exception that handled by Spring MVC.
    * - :doc:`../ArchitectureInDetail/WebApplicationDetail/Codelist`
      - Populate CodeList interceptor
      - Provide interceptor class of Spring MVC for storing CodeList information into request scope, for the purpose of retrieving CodeList from View.
    * - :doc:`../ArchitectureInDetail/WebApplicationDetail/FileDownload`
      - General Download View
      - Provide abstract class for retrieving data from input stream and writing to stream for download.
    * - :doc:`../ArchitectureInDetail/GeneralFuncDetail/Logging`
      - ServletFilter for storing Tracking ID
      - Provide Servlet Filter class for setting Tracking ID into MDC(Mapped Diagnostic Context) and request scope and response header, for the purpose of improving traceability.
        (If does not exist a Tracking ID in request header, generate a Tracking ID by this component)
    * -
      - General ServletFilter for storing MDC
      - Provide abstract class for storing any value into Logger's MDC
    * -
      - ServletFilter for clearing MDC
      - Provide ServletFilter class for clearing information that stored in Logger's MDC.

terasoluna-gfw-web-jsp
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

terasoluna-gfw-web-jsp provides following components.

.. tabularcolumns:: |p{0.20\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 30 50

    * - Classification
      - Component name
      - Description
    * - :doc:`../ArchitectureInDetail/WebApplicationDetail/DoubleSubmitProtection`
      - JSP tag for transaction token output
      - Provide a JSP tag library to output transaction tokens as hidden items.
    * - :doc:`../ArchitectureInDetail/WebApplicationDetail/Pagination`
      - JSP Tag for displaying Pagination Links
      - Provide JSP Tag Library for displaying Pagination Links using classes that provided by Spring Data Commons.
    * - :doc:`../ArchitectureInDetail/WebApplicationDetail/MessageManagement`
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
    * - :doc:`../ArchitectureInDetail/GeneralFuncDetail/Logging`
      - ServletFilter for storing name of authenticated user
      - Provide ServletFilter class for setting name of authenticated user into MDC, for the purpose of improving traceability.


.. raw:: latex

   \newpage
