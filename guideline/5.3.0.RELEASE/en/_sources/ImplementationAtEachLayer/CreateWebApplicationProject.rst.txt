Create Web application development project
================================================================================

.. only:: html

 .. contents:: Index
    :depth: 3
    :local:

In this section, how to create a web application development project is described.

In this guideline, it is recommended to adopt multi-project configuration.
For description of the recommended multi-project configuration, refer [:ref:`application-layering_project-structure`].

.. _CreateProjectFromBlankTypes:

Types of Blank Project
--------------------------------------------------------------------------------

Following two types of blank projects are provided depending on usage.

.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 70

    * - Type
      - Usage
    * - | `Blank project of multi-project structure <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank>`_
      - This should be used when developing a real application that is to be released in commercial environment.

        Following types of project templates are provided as Archetype of Maven.

        * Template that includes settings for MyBatis3
        * Template that includes settings for JPA (Spring Data JPA)

        **This guideline recommends using a project having multi-project structure.**
    * - | `Blank project of single-project structure <https://github.com/terasolunaorg/terasoluna-gfw-web-blank>`_
      - This should be used when creating simple applications such as POC (Proof Of Concept), prototype, sample, etc.

        Following types of project templates are provided as Archetype of Maven.
        (Projects for Eclipse WTP are also provided; however, their description is omitted in this chapter)

        * Template that includes settings for MyBatis3
        * Template that includes settings for JPA (Spring Data JPA)
        * Template that does not depend on O/R Mapper

        This guideline follows a procedure wherein various tutorials are performed using a single project.
        
.. _CreateWebApplicationProject:

Create development project
--------------------------------------------------------------------------------

The multi-project structured development project will be created using the 
`archetype:generate <http://maven.apache.org/archetype/maven-archetype-plugin/generate-mojo.html>`_ of the `Maven Archetype Plugin <http://maven.apache.org/archetype/maven-archetype-plugin/>`_ .

.. note:: **Prerequisite**

    In the following description,

    * `Maven <http://maven.apache.org/>`_ (\ ``mvn`` \ command) is used
    * Internet connection is used
    * If internet is connected via proxy, `Maven proxy setting <http://maven.apache.org/guides/mini/guide-proxies.html>`_  needs to be done

    are prerequisites.

    If prerequisite conditions are not satisfied, it is necessary to perform these setups first.

|

As an Archetype following two types are provided for creating multi-project.

.. tabularcolumns:: |p{0.05\linewidth}|p{0.30\linewidth}|p{0.65\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 5 30 65

    * - Sr. No.
      - Archetype(ArtifactId)
      - Description
    * - 1.
      - terasoluna-gfw-multi-web-blank-mybatis3-archetype
      - Archetype for generating a project to use MyBatis3 as O/R Mapper.
    * - 2.
      - terasoluna-gfw-multi-web-blank-jpa-archetype
      - Archetype for generating a project to use JPA(with Spring Data JPA and Hibernate) as O/R Mapper.

|

Move to the folder where you want to create project.

.. code-block:: console

    cd C:\work

|

Create project using `archetype:generate <http://maven.apache.org/archetype/maven-archetype-plugin/generate-mojo.html>`_ of `Maven Archetype Plugin <http://maven.apache.org/archetype/maven-archetype-plugin/>`_.

.. code-block:: console

    mvn archetype:generate -B^
     -DarchetypeGroupId=org.terasoluna.gfw.blank^
     -DarchetypeArtifactId=terasoluna-gfw-multi-web-blank-mybatis3-archetype^
     -DarchetypeVersion=5.3.0.RELEASE^
     -DgroupId=com.example.todo^
     -DartifactId=todo^
     -Dversion=1.0.0-SNAPSHOT

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 25 75

    * - Parameter
      - Description
    * - | \-B
      - batch mode (skips interaction)
    * - | \-DarchetypeGroupId
      - Specify groupId of the blank project.(Fixed)
    * - | \-DarchetypeArtifactId
      - Specify archetypeId (ID to identify the template) of the blank project. **(Customization required)**

        specify one of the following archetypeId.

        * ``terasoluna-gfw-multi-web-blank-mybatis3-archetype``
        * ``terasoluna-gfw-multi-web-blank-jpa-archetype``

        In above example, \ ``terasoluna-gfw-multi-web-blank-mybatis3-archetype`` \ is specified.
    * - | \-DarchetypeVersion
      - Specify version of the blank project.(Fixed)
    * - | \-DgroupId
      - Specify groupId of the project that you want to create. **(Customization required)**

        In above example, \ ``"com.example.todo"`` \ is specified.
    * - | \-DartifactId
      - Specify artifactId of the project that you want to create. **(Customization required)**

        In above example, \ ``"todo"`` \ is specified.
    * - | \-Dversion
      - Specify version of the project that you want to create. **(Customization required)**

        In above example, \ ``"1.0.0-SNAPSHOT"`` \ is specified.

|

If the project creation successes, following type of log will be printed.
(The following output is an example when project is created using the MyBatis3 Archetype)

.. code-block:: console

    (... omit)
    [INFO] ----------------------------------------------------------------------------
    [INFO] Using following parameters for creating project from Archetype: terasoluna-gfw-multi-web-blank-mybatis3-archetype:5.3.0.RELEASE
    [INFO] ----------------------------------------------------------------------------
    [INFO] Parameter: groupId, Value: com.example.todo
    [INFO] Parameter: artifactId, Value: todo
    [INFO] Parameter: version, Value: 1.0.0-SNAPSHOT
    [INFO] Parameter: package, Value: com.example.todo
    [INFO] Parameter: packageInPathFormat, Value: com/example/todo
    [INFO] Parameter: package, Value: com.example.todo
    [INFO] Parameter: version, Value: 1.0.0-SNAPSHOT
    [INFO] Parameter: groupId, Value: com.example.todo
    [INFO] Parameter: artifactId, Value: todo
    [INFO] Parent element not overwritten in C:\work\todo\todo-env\pom.xml
    [INFO] Parent element not overwritten in C:\work\todo\todo-domain\pom.xml
    [INFO] Parent element not overwritten in C:\work\todo\todo-web\pom.xml
    [INFO] Parent element not overwritten in C:\work\todo\todo-initdb\pom.xml
    [INFO] Parent element not overwritten in C:\work\todo\todo-selenium\pom.xml
    [INFO] project created from Archetype in dir: C:\work\todo
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 9.929 s
    [INFO] Finished at: 2015-07-31T12:03:21+00:00
    [INFO] Final Memory: 10M/26M
    [INFO] ------------------------------------------------------------------------

|

If the project creation successes, Maven multi-project gets created.
For detail description of the project that you have created in the Maven Archetype, Refer [:ref:`CreateWebApplicationProjectConfiguration`].

.. code-block:: console

    todo
    ├── pom.xml
    ├── todo-domain
    ├── todo-env
    ├── todo-initdb
    ├── todo-selenium
    └── todo-web

|

.. _CreateWebApplicationProjectCustomize:

Customization of development project
--------------------------------------------------------------------------------

Depending upon the application, there are several locations where customization is required in the Maven Archetype created project.

The customization required locations are described below.

- :ref:`CreateWebApplicationProjectCustomizeProjectInformation`
- :ref:`CreateWebApplicationProjectCustomizeMessageId`
- :ref:`CreateWebApplicationProjectCustomizeMessageWording`
- :ref:`CreateWebApplicationProjectCustomizeErrorScreen`
- :ref:`CreateWebApplicationProjectCustomizeCopyrightOnScreenFooter`
- :ref:`CreateWebApplicationProjectCustomizeInMemoryDatabase`
- :ref:`CreateWebApplicationProjectCustomizeDataSource`

.. note::

    The customization points other than the above are,

    * Settings of :doc:`../Security/Authentication`・:doc:`../Security/Authorization`
    * Settings to enable :doc:`../ArchitectureInDetail/WebApplicationDetail/FileUpload`
    * Setting to activate :doc:`../ArchitectureInDetail/WebApplicationDetail/Internationalization`
    * Definition of :doc:`../ArchitectureInDetail/GeneralFuncDetail/Logging`
    * Definition of :doc:`../ArchitectureInDetail/WebApplicationDetail/ExceptionHandling`
    * Apply settings of :doc:`../ArchitectureInDetail/WebServiceDetail/REST`

    For these customizations, Refer to "How to use" of each section and customize if required.


.. note::

    Part that is expressed as \ ``artifactId`` \ in the following description 
    needs to be read by replacing the \ ``artifactId`` \ which is specified at the time of creating a project.

|

.. _CreateWebApplicationProjectCustomizeProjectInformation:

POM file project information
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In the POM file of Maven Archetype created project,

* Project name (\ ``name`` \ element)
* Project description (\ ``description`` \ element)
* Project URL (\ ``url`` \ element)
* Project inception year (\ ``inceptionYear`` \ element)
* Project license (\ ``licenses`` \ element)
* Project organization (\ ``organization`` \ element)

such information set in Archetype projects.
The actual settings contents indicated below.

.. code-block:: xml

    <!-- ... -->

    <name>TERASOLUNA Server Framework for Java (5.x) Web Blank Multi Project</name>
    <description>Web Blank Multi Project using TERASOLUNA Server Framework for Java (5.x)</description>
    <url>http://terasoluna.org</url>
    <inceptionYear>2014</inceptionYear>
    <licenses>
        <license>
            <name>Apache License, Version 2.0</name>
            <url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>
            <distribution>manual</distribution>
        </license>
    </licenses>
    <organization>
        <name>TERASOLUNA Framework Team</name>
        <url>http://terasoluna.org</url>
    </organization>

    <!-- ... -->

.. note::

    **Set the appropriate values in the project information.**

|

Customization method and customization targeted files are indicated below.

.. tabularcolumns:: |p{0.10\linewidth}|p{0.45\linewidth}|p{0.45\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 45 45

    * - Sr. No.
      - Targeted File
      - Customization method
    * - 1.
      - POM (Project Object Model) file that defines the overall configuration of multi-project

        ``artifactId/pom.xml``
      - Set the appropriate values in the project information.

|

.. _CreateWebApplicationProjectCustomizeMessageId:

x.xx.fw.9999 format message ID
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In the Maven Archetype created project, the \ ``x.xx.fw.9999`` \ format message ID used at the time of,

* Message to be displayed on the error screen
* Error log to be output when an exception occurs

Actual point-of-use (sampling) indicated below.

**[application-messages.properties]**

.. code-block:: properties

    e.xx.fw.5001 = Resource not found.

**[JSP]**

.. code-block:: jsp

    <div class="error">
        <c:if test="${!empty exceptionCode}">[${f:h(exceptionCode)}]</c:if>
        <spring:message code="e.xx.fw.5001" />
    </div>

**[applicationContext.xml]**

.. code-block:: xml

    <bean id="exceptionCodeResolver"
        class="org.terasoluna.gfw.common.exception.SimpleMappingExceptionCodeResolver">
        <!-- ... -->
                <entry key="ResourceNotFoundException" value="e.xx.fw.5001" />
        <!-- ... -->
    </bean>

|

The \ ``x.xx.fw.9999`` \ format message ID is
a message ID system that is introduced in [:doc:`../ArchitectureInDetail/WebApplicationDetail/MessageManagement`] of this guideline but,
the value of the project division is in the state of provisional value [\ ``xx``\].

.. note::

    * **If the message ID system introduced in this guideline is used, specify the appropriate values to the project classification.** For the message ID system introduced in this guideline, Refer [:ref:`message-management_result-rule`].
    * If the message ID system introduced in this guideline is not used, it is necessary to replace all the message IDs those are used in the customization targeted file indicated below.

|

Customization method and customization targeted files are indicated below.

.. tabularcolumns:: |p{0.10\linewidth}|p{0.45\linewidth}|p{0.45\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 45 45

    * - Sr. No.
      - Targeted File
      - Customization method
    * - 1.
      - Message definition file

        ``artifactId/artifactId-web/src/main/resources/i18n/application-messages.properties``
      - The provisional value [\ ``xx``\] of project classification message ID specified in the property key needs to be modified by appropriate value.
    * - 2.
      - Error screen JSP

        ``artifactId/artifactId-web/src/main/webapp/WEB-INF/views/common/error/*.jsp``
      - The provisional value [\ ``xx``\] of project classification message ID specified in the \ ``code`` \ attribute of the element \ ``<spring:message>`` \ needs to be modified by appropriate value.
    * - 3.
      - Bean definition file to create an application context for Web applications

        ``artifactId/artifactId-web/src/main/resources/META-INF/spring/applicationContext.xml``
      - The provisional value [\ ``xx``\] of project classification exception code (message ID) specified in the Bean definition of \ ``"exceptionCodeResolver"`` \ needs to be modified by appropriate value.

|

.. _CreateWebApplicationProjectCustomizeMessageWording:

Message wording
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In the Maven Archetype created project, number of message definitions are provided but, 
message wordings are simple messages.
Actual messages (sampling) are indicated below.

**[application-messages.properties]**

.. code-block:: properties

    e.xx.fw.5001 = Resource not found.

    # ...

    # typemismatch
    typeMismatch="{0}" is invalid.

    # ...

.. note::

    **Modify the message wording depending upon the application requirements (such as message terms)**

|

Customization method and customization targeted files are indicated below.

.. tabularcolumns:: |p{0.10\linewidth}|p{0.45\linewidth}|p{0.45\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 45 45

    * - Sr. No.
      - Targeted File
      - Customization method
    * - 1.
      - Message definition file

        ``artifactId/artifactId-web/src/main/resources/i18n/application-messages.properties``
      - Modify the messages in accordance with the application requirements.

        The message to be displayed (Bean Validation messages) when there is an error in input check
        needs to be modified (override default messages) depending upon the application requirement.
        For overriding the default messages, Refer [:ref:`Validation_message_def`].

|

.. _CreateWebApplicationProjectCustomizeErrorScreen:

Error screen
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In the Maven Archetype created project, JSP and HTML are provided for displaying an error screen for every kind of errors but,

* screen layout
* screen title
* wording of the message

etc are simple implementation. Actual JSP implementation (sampling) is indicated below.

**[JSP]**

.. code-block:: jsp

    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="utf-8">
    <title>Resource Not Found Error!</title>
    <link rel="stylesheet"
        href="${pageContext.request.contextPath}/resources/app/css/styles.css">
    </head>
    <body>
        <div id="wrapper">
            <h1>Resource Not Found Error!</h1>
            <div class="error">
                <c:if test="${!empty exceptionCode}">[${f:h(exceptionCode)}]</c:if>
                <spring:message code="e.xx.fw.5001" />
            </div>
            <t:messagesPanel />
        <br>
        <!-- ... -->
        <br>
        </div>
    </body>
    </html>

.. note::

    **Modify the JSP and HTML depending upon the application requirements (such as UI terms) used for displaying an error screen.**

|

Customization method and customization targeted files are indicated below.

.. tabularcolumns:: |p{0.10\linewidth}|p{0.45\linewidth}|p{0.45\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 45 45

    * - Sr. No.
      - Targeted File
      - Customization method
    * - 1.
      - JSP for the error screen

        ``artifactId/artifactId-web/src/main/webapp/WEB-INF/views/common/error/*.jsp``
      - Modify depending upon the application requirements (such as UI terms).

        Refer [:ref:`exception-handling-how-to-use-codingpoint-jsp-label` of :doc:`../ArchitectureInDetail/WebApplicationDetail/ExceptionHandling`] for customizing the JSP to display an error screen.
    * - 2.
      - HTML for the error screen

        ``artifactId/artifactId-web/src/main/webapp/WEB-INF/views/common/error/unhandledSystemError.html``
      - Modify depending upon the application requirements (such as UI terms).

|

.. _CreateWebApplicationProjectCustomizeCopyrightOnScreenFooter:

Screen footer copyright
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In the Maven Archetype created project, screen layouts are configured using Tiles but,
the copyright of the screen footer portion is in a state of provisional value [\ ``Copyright &copy; 20XX CompanyName``\]. 
Actual JSP implementation (sampling) is indicated below.

**[template.jsp]**

.. code-block:: jsp

    <div class="container">
      <tiles:insertAttribute name="header" />
      <tiles:insertAttribute name="body" />
      <hr>
      <p style="text-align: center; background: #e5eCf9;">Copyright
        &copy; 20XX CompanyName</p>
    </div>

.. note::

    **If screen layouts are configured using Tiles, specify appropriate value to the copyright.**

|

Customization method and customization targeted files are indicated below.

.. tabularcolumns:: |p{0.10\linewidth}|p{0.45\linewidth}|p{0.45\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 45 45

    * - Sr. No.
      - Targeted File
      - Customization method
    * - 1.
      - Template JSP for Tiles

        ``artifactId/artifactId-web/src/main/webapp/WEB-INF/views/layout/template.jsp``
      - Modify the provisional value [\ ``Copyright &copy; 20XX CompanyName`` \ ] of the copyright with an appropriate value.

|

.. _CreateWebApplicationProjectCustomizeInMemoryDatabase:

In-memory database (H2 Database)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In the Maven Archetype created project, in-memory database (H2 Database) setting is configured but,
these settings are done for the small operation (Prototyping and POC (Proof Of Concept)) verification.
Therefore, these could be unnecessary settings while having regular application development.

**[artifactId-env.xml]**

.. code-block:: xml

    <jdbc:initialize-database data-source="dataSource"
        ignore-failures="ALL">
        <jdbc:script location="classpath:/database/${database}-schema.sql" encoding="UTF-8" />
        <jdbc:script location="classpath:/database/${database}-dataload.sql" encoding="UTF-8" />
    </jdbc:initialize-database>

.. code-block:: console

        └── src
            └── main
                └── resources
                    ├── META-INF
                  (...)
                    ├── database
                    │   ├── H2-dataload.sql
                    │   └── H2-schema.sql

.. note::

    **While having regular application development, remove the directory which is maintained for definition and SQL files for setting up a In-memory database (H2 Database)**

|

Customization method and customization targeted files are indicated below.

.. tabularcolumns:: |p{0.10\linewidth}|p{0.45\linewidth}|p{0.45\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 45 45

    * - Sr. No.
      - Targeted File
      - Customization method
    * - 1.
      - Bean definition file for defining environment dependent components

        ``artifactId-env/src/main/resources/META-INF/spring/artifactId-env.xml``
      - Remove the \ ``<jdbc:initialize-database>`` \ element.
    * - 2.
      - Directory that contains the SQL for configuring In-memory database (H2 Database)

        ``artifactId/artifactId-env/src/main/resources/database/``
      - Remove the directory.

|

.. _CreateWebApplicationProjectCustomizeDataSource:

DataSource configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In the Maven Archetype created project, DataSource setting is done for accessing in-memory database (H2 Database) but,
these settings are done for the small operation (Prototyping and POC (Proof Of Concept)) verification.
Therefore it is necessary to change the DataSource settings for accessing the actual running database application while having regular application development.

**[artifactId/artifactId-domain/pom.xml]**

.. code-block:: xml

    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>

.. note::  
 
	In the above setting example, since it is assumed that the dependent library version is managed by the parent project terasoluna-gfw-parent , specifying the version in pom.xml is not necessary.
	The above dependent library used by terasoluna-gfw-parent is defined by \ `Spring IO Platform <http://platform.spring.io/platform/>`_\ .

**[artifactId-infra.properties]**

.. code-block:: properties

    database=H2
    database.url=jdbc:h2:mem:todo;DB_CLOSE_DELAY=-1
    database.username=sa
    database.password=
    database.driverClassName=org.h2.Driver
    # connection pool
    cp.maxActive=96
    cp.maxIdle=16
    cp.minIdle=0
    cp.maxWait=60000

**[artifactId-env.xml]**

.. code-block:: xml

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

.. note::

    **Change the DataSource settings for accessing the actual running database application while having regular application development.**

    In the Maven Archetype created project, the use of Apache Commons DBCP is configured but,
    there are many cases that adopting a method of accessing a DataSource via JNDI (Java Naming and Directory Interface) 
    by use of DataSource provided by the application server.

    Again there are some cases where Apache Commons DBCP is used on development environment and 
    DataSource provided by the application server is used on test as well as production environment.

    For how to set-up the DataSource, Refer [:ref:`data-access-common_howtouse_datasource` of :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessCommon`].

|

Customization method and customization targeted files are indicated below.

.. tabularcolumns:: |p{0.10\linewidth}|p{0.45\linewidth}|p{0.45\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 45 45

    * - Sr. No.
      - Targeted File
      - Customization method
    * - 1.
      - POM file

        * ``artifactId/pom.xml``
        * ``artifactId/artifactId-web/pom.xml``
      - Remove in-memory database (H2 Database) JDBC driver from the dependency library.

        Add the JDBC driver in dependency library for accessing the actual running application database.

    * - 2.
      - Property file for defining environment dependent setting

        ``artifactId/artifactId-env/src/main/resources/META-INF/spring/artifactId-infra.properties``
      - If Apache Commons DBCP is used as a DataSource, specify the connection information for accessing the actual running application database in below property.

        * ``database``
        * ``database.url``
        * ``database.username``
        * ``database.password``
        * ``database.driverClassName``

        Remove unnecessary property except the following property if DataSource provided by the application server is used.

        * ``database``

    * - 3.
      - Bean definition file for defining environment dependent components

        ``artifactId/artifactId-env/src/main/resources/META-INF/spring/artifactId-env.xml``
      - If DataSource provided by the application server is used, change the configuration to use the DataSource that is obtained via JNDI.

        For how to set-up the DataSource, Refer [:ref:`data-access-common_howtouse_datasource` of :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessCommon`].

.. note:: **About the database property of the property file for defining environment dependent setting**


    The \ ``database`` \ property is unnecessary property if MyBatis is used as O/R Mapper.
    You may remove this but you may leave the settings in order to specify the database being used.

.. tip:: **How to add the JDBC driver**

    It is fine to remove the comment out of POM file in case of PostgreSQL or Oracle database is used.. 
    Modify the JDBC driver version by actual use of the corresponding database version.

    However, if Oracle is used, 
    it is necessary to install the Oracle JDBC driver in the local repository of Maven before removing the comment.

    The following is an example of setting in case of PostgreSQL is used.

    * ``artifactId/pom.xml``

     .. code-block:: xml

                         <dependency>
                             <groupId>org.postgresql</groupId>
                             <artifactId>postgresql</artifactId>
                             <version>${postgresql.version}</version>
                         </dependency>
        <!--             <dependency> -->
        <!--                 <groupId>com.oracle</groupId> -->
        <!--                 <artifactId>ojdbc7</artifactId> -->
        <!--                 <version>${ojdbc.version}</version> -->
        <!--             </dependency> -->

            <!-- ... -->

            <postgresql.version>9.4-1206-jdbc41</postgresql.version>
            <ojdbc.version>12.1.0.2</ojdbc.version>

    * ``artifactId/artifactId-web/pom.xml``

     .. code-block:: xml

                     <dependency>
                         <groupId>org.postgresql</groupId>
                         <artifactId>postgresql</artifactId>
                         <scope>runtime</scope><!-- (1) -->
                     </dependency>
        <!--         <dependency> -->
        <!--             <groupId>com.oracle</groupId> -->
        <!--             <artifactId>ojdbc7</artifactId> -->
        <!--             <scope>runtime</scope> -->
        <!--         </dependency> -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | Sr. No.
      - | Description
    * - | (1)
      - JDBC driver is not need when compile, it is only need when run time, so the \ ``runtime`` \ scope is specified.

        In case of using JDBC driver in unit test, change to appropriate scope and use it.

|

.. _CreateWebApplicationProjectConfiguration:

Structure of the development project
--------------------------------------------------------------------------------

Explained the structure of the project created in Maven Archetype.

Below is the structure of the project created in Maven Archetype.

* Project structure of each layer that is recommended in this guideline
* Project structure that takes into account the exclusion of environmental dependency introduced in this guideline
* Project structure that conscious the CI (Continuous Integration)

In addition, various settings have been included that is recommended in this guideline

* Web application configuration file (web.xml)
* Spring Framework Bean definition file
* Bean definition file for the Spring MVC
* Bean definition file for the Spring Security
* O/R Mapper configuration file
* Tiles configuration file
* Property file (such as message definition file)

and, as a simple component implementation of low (=necessary to develop every kind of application) dependency on the application requirements,

* Controller and JSP for displaying Welcome page
* JSP to display an error screen (HTML)
* Template JSP for Tiles
* Include JSP for reading configuration such as JSP tag library
* CSS file that defines the screen style of entire application

etc are provided.

.. warning:: **Components provided as a simple implementation**

    Components provided as a simple implementation can be corresponding to one of the following.

    * Modification to meet the application requirements
    * Removal of unnecessary components

.. note:: **Procedure to create the REST API project**

    In the Maven Archetype created project,
    the recommended settings are done which are required for building a traditional Web application (application that receives the request parameters and respond the HTML).

    Therefore, unnecessary setting exists in building a REST API for handling JSON or XML.
    If you want to create a project for building REST API, 
    need to apply the REST API related settings by referring to the [:ref:`RESTHowToUseApplicationSettings` of :doc:`../ArchitectureInDetail/WebServiceDetail/REST`].

.. note::

    Part that is expressed as \ ``artifactId`` \ in the following description 
    needs to be read by replacing the \ ``artifactId`` \ which is specified at the time of creating a project.

|

.. _CreateWebApplicationProjectConfigurationMulti:

Multi-project structure
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Initially entire multi-project structure is explained.

.. code-block:: console

    artifactId
        ├── pom.xml  ... (1)
        ├── artifactId-web  ... (2)
        ├── artifactId-domain  ... (3)
        ├── artifactId-env  ... (4)
        ├── artifactId-initdb  ... (5)
        └── artifactId-selenium  ... (6)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - | Sr. No.
      - | Description
    * - | (1)
      - The entire multi-project configuration is defined in POM (Project Object Model) file.

        Mainly following definitions are done in this file.

        * Version of the dependent libraries
        * Build plug-ins settings (setting of how to build)

        Refer [:ref:`CreateWebApplicationProjectAppendixProjectHierarchicalStructure`] for the hierarchical relationship of multi-project.

    * - | (2)
      - Module that manages the application layer (Web layer) components.

        Mainly following components and files are managed in this module.

        * Controller class
        * Validator class for relational check
        * Form class (the Resource class in case of REST API)
        * View (JSP)
        * CSS file
        * JavaScript file
        * JUnit for the application layer components
        * Bean definition file for defining the application layer components
        * Web application configuration file (web.xml)
        * Message definition file
         

    * - | (3)
      - Module that manages the domain layer components.

        Mainly following components and files are managed in this module.

        * Domain object such as Entity
        * Repository
        * Service
        * DTO
        * JUnit for the domain layer components
        * Bean definition file for defining the domain layer components

    * - | (4)
      - Module that manages the environmental dependency settings files.

        Mainly following files are managed in this module.

        * Bean definition file for defining the environment dependent components
        * Property file for defining the environment dependent properties value

    * - | (5)
      - Module that manages the database initialization SQL files.

        Mainly following files are managed in this module.

        * SQL file to create the database objects such as tables
        * SQL file to insert the initial data such as master data
        * SQL file to insert the test data used for E2E (End To End) test

    * - | (6)
      - Module that manages the Selenium used E2E testing components

        Mainly following files are managed in this module.

        * JUnit testing using Selenium operation
        * Expected value file used while Assert (if necessary)

.. raw:: latex

   \newpage

.. note:: **About a terminology definition of [multi-project] in this guideline**

    The project created in Maven Archetype is the exact multi-module structured project.

    This is supplement that the multi-module and multi-project is being used as the same meaning in this guideline.


.. note:: **Development projects required for two Web applications and one common library**

    * bar-parent
    * bar-initdb
    * bar-common
    * bar-common-web
    * bar-domain-a
    * bar-domain-b
    * bar-web-a
    * bar-web-b
    * bar-env
    * bar-web-a-selenium
    * bar-web-b-selenium
    
    The contents of each project are as follows.
    
    * bar-parent
    
      Project called as a parent-pom (parent POM). A simple project consisting of only pom.xml file.
      It never contains other source code or configuration files.
      Common setting information specified in the parent POM can be reflected in other project
      by specifying this bar-parent project into <parent> tag.
    
    * bar-initdb
    
     Stores RDBMS table definitions (DDL) and SQL statements for INSERT the initial data.
     This also managed as a maven project.
     By defining `sql-maven-plugin <http://www.mojohaus.org/sql-maven-plugin/>`_ in pom.xml,
     it is possible to automate the execution of DDL statements and initial data INSERT statements for any RDBMS in the course of the build lifecycle.
    
    * bar-common
    
      Stores common library in the project. Web related classes are placed in the bar-common-web by making it as a web-independent.
    
    * bar-common-web
    
      Stores common web library in the project.
    
    * bar-domain-a
    
      Stores unit test cases and domain layer java classes related to "a" domain. Finally \*.jar file is created.
    
    * bar-domain-b
    
      Stores domain layer java classes related to "b" domain.
    
    * bar-web-a
    
      Stores application layer java classes, jsps, configuration files, unit test cases. Finally created \*.war file is created as the Web application.
      bar-web-a having dependency on bar-common and bar-env.
    
    * bar-web-b
    
      This is a Web application as one more subsystem. Structure is the same as the bar-web-a.
    
    * bar-env
    
      Collects only the configuration files having environment dependency.
    
    * bar-web-a-selenium
    
      Project that stores test cases using `Selenium WebDriver <http://seleniumhq.org/projects/webdriver/>`_ for web-a project.
    
    * bar-web-b-selenium
    
      Project that stores test cases using `Selenium WebDriver <http://seleniumhq.org/projects/webdriver/>`_ for web-b project.


.. _CreateWebApplicationProjectConfigurationWeb:

Structure of Web module
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Module that manages the application layer (Web layer) components are explained. 

.. code-block:: console

    artifactId-web
        ├── pom.xml  ... (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | Sr. No.
      - | Description
    * - | (1)
      - The web module configuration is defined in POM (Project Object Model) file.
        Following definitions are done in this file.

        * Definition of dependent libraries and build plug-ins
        * Definition to create a war file

.. note:: **About the module name of the web module while creating a project for REST API**

    The application type can be easily distinguished,
    if the module name is assigned the name of \ ``artifactId-api`` \ while building a REST API.

|

.. code-block:: console

        └── src
            ├── main
            │   ├── java
            │   │   └── com
            │   │       └── example
            │   │           └── project
            │   │               └── app  ... (2)
            │   │                   └── welcome
            │   │                       └── HelloController.java  ... (3)
            │   ├── resources
            │   │   ├── META-INF
            │   │   │   ├── dozer  ... (4)
            │   │   │   └── spring  ... (5)
            │   │   │       ├── application.properties  ... (6)
            │   │   │       ├── applicationContext.xml  ... (7)
            │   │   │       ├── spring-mvc.xml  ... (8)
            │   │   │       └── spring-security.xml  ... (9)
            │   │   └── i18n  ... (10)
            │   │       └── application-messages.properties  ... (11)

.. raw:: latex

   \newpage

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - | Sr. No.
      - | Description
    * - | (2)
      - Package for storing the  application layer classes.

        The component type can be easily distinguished, 
        if the package name is assigned the name of \ ``api`` \ while building a REST API.
    * - | (3)
      - The controller class for receiving a request to display the Welcome page.
    * - | (4)
      - The directory in which a mapping definition file of Dozer (Bean Mapper) is stored, 
        Refer to [:doc:`../ArchitectureInDetail/GeneralFuncDetail/Dozer`] for Dozer.

        It is an empty directory at the time of creation. 
        If the mapping file is required (if high mapping is required), 
        it gets automatically read in case of stored under this directory.

        .. note::

            Following files are stored under this directory.

            * Definition file for mapping the application layer JavaBean with domain layer JavaBean
            * Definition file for each other mapping of application layer JavaBean

            It is recommended to store each other mapping of domain layer JavaBean in domain layer directory.

    * - | (5)
      - Directory contains the property file and Spring Framework bean definition file.
    * - | (6)
      - Properties file that defines the settings to be used in the application layer.

        It is an empty file at the time of creation. 
    * - | (7)
      - Bean definition file to create an application context for Web applications.

        Following beans are defined in this file.

        * Components to be used in the entire Web application
        * Domain layer components (Import the bean definition file in which domain layer components are defined)

    * - | (8)
      - Bean definition file to create an application context for the \ ``DispatcherServlet``\.

        Following beans are defined in this file.

        * Spring MVC components
        * application layer components

        The application type can be easily distinguished, if the file name is assigned the name of \ ``spring-mvc-api.xml`` \ while building a REST API.

    * - | (9)
      - Bean definition file for defining the Spring Security components.

        This file is read when you create an application context for the Web application.
    * - | (10)
      - Directory that contains the message definition file to be used in the application layer.
    * - | (11)
      - Property file that defines the messages to be used in the application layer.

        Some of the generic messages are defined at the time of creation.

        .. note::

            **Messages should be modified according to the application requirements (Such as message Terms).**
            For the message definition, Refer [:doc:`../ArchitectureInDetail/WebApplicationDetail/MessageManagement`].

.. raw:: latex

   \newpage

.. note::

    Refer [:ref:`CreateWebApplicationProjectAppendixApplicationContext`] for the application context and bean definition file related.

|

.. code-block:: console

            │   └── webapp
            │       ├── WEB-INF
            │       │   ├── tiles  ... (12)
            │       │   │   └── tiles-definitions.xml
            │       │   ├── views  ... (13)
            │       │   │   ├── common
            │       │   │   │   ├── error  ... (14)
            │       │   │   │   │   ├── accessDeniedError.jsp
            │       │   │   │   │   ├── businessError.jsp
            │       │   │   │   │   ├── dataAccessError.jsp
            │       │   │   │   │   ├── invalidCsrfTokenError.jsp
            │       │   │   │   │   ├── missingCsrfTokenError.jsp
            │       │   │   │   │   ├── resourceNotFoundError.jsp
            │       │   │   │   │   ├── systemError.jsp
            │       │   │   │   │   ├── transactionTokenError.jsp
            │       │   │   │   │   └── unhandledSystemError.html
            │       │   │   │   └── include.jsp  ... (15)
            │       │   │   ├── layout  ... (16)
            │       │   │   │   ├── header.jsp
            │       │   │   │   └── template.jsp
            │       │   │   └── welcome
            │       │   │       └── home.jsp  ... (17)
            │       │   └── web.xml  ... (18)
            │       └── resources  ... (19)
            │           └── app
            │               └── css
            │                   └── styles.css  ... (20)
            └── test
                ├── java
                └── resources

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - | Sr. No.
      - | Description
    * - | (12)
      - Directory that contains the Tiles configuration files.
        Refer [:doc:`../ArchitectureInDetail/WebApplicationDetail/TilesLayout`] for the Tiles configuration files.
    * - | (13)
      - Directory that contains the View generation templates (jsp etc).
    * - | (14)
      - Directory that contains the JSP and HTML for displaying error screens.

        At the time of creation, JSPs (HTMLs) are stored corresponding to the errors that may occur during application execution.

        .. note::

            **Error screen JSP and HTML should be modified according to the application requirements (Such as UI Terms).**

    * - | (15)
      - Common JSP files for include.


        This file is included at the beginning of all JSP files.
        Refer [:ref:`view_jsp_include-label`] for common JSP files for include.
    * - | (16)
      - Directory that contains the JSP files for the Tiles layout.
        Refer [:doc:`../ArchitectureInDetail/WebApplicationDetail/TilesLayout`] for JSP files for the Tiles layout.
    * - | (17)
      - JSP file that displays the Welcome page.
    * - | (18)
      - Configuration definition file for the Web application.
    * - | (19)
      - Directory that contains the static resource files.

        This directory contains such files whose response contents are not going to change depending upon the request contents.
        Specifically following files are stored.

        * JavaScript files
        * CSS files
        * Image files
        * HTML files

        Here adopted a dedicated directory mechanism for managing static resources offered by Spring MVC.
    * - | (20)
      - CSS file that defines the screen style applied to the entire application.

.. raw:: latex

   \newpage

|

.. _CreateWebApplicationProjectConfigurationDomain:

Structure of Domain module
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Module that manages the domain layer components are explained.

.. code-block:: console

    artifactId-domain
        ├── pom.xml  ... (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | Sr. No.
      - | Description
    * - | (1)
      - The domain module configuration is defined in POM (Project Object Model) file.
        Following definitions are done in this file.

        * Definition of dependent libraries and build plug-ins
        * Definition to create a jar file

|

.. code-block:: console

        └── src
            ├── main
            │   ├── java
            │   │   └── com
            │   │       └── example
            │   │           └── project
            │   │               └── domain  ... (2)
            │   │                   ├── model
            │   │                   ├── repository
            │   │                   └── service
            │   └── resources
            │       └── META-INF
            │           ├── dozer  ... (3)
            │           └── spring  ... (4)
            │               ├── artifactId-codelist.xml  ... (5)
            │               ├── artifactId-domain.xml  ... (6)
            │               └── artifactId-infra.xml  ... (7)


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | Sr. No.
      - | Description
    * - | (2)
      - Package for storing the  domain layer classes.
    * - | (3)
      - The directory in which a mapping definition file of Dozer (Bean Mapper) is stored, 
        Refer to [:doc:`../ArchitectureInDetail/GeneralFuncDetail/Dozer`] for Dozer.

        It is an empty directory at the time of creation. 
        If the mapping file is required (if high mapping is required), 
        it gets automatically read in case of stored under this directory.

        .. note::

            Following files are stored under this directory.

            * Definition file for each other mapping of domain layer JavaBean

    * - | (4)
      - Directory contains the property file and Spring Framework bean definition file.
    * - | (5)
      - Bean definition file for defining the code list.
    * - | (6)
      - Bean definition file for defining the domain layer components.

        Following beans are defined in this file.

        * Domain layer components (Service, Repository etc)
        * Infrastructure layer components (Import the bean definition file that the component has been defined in the Infrastructure layer)
        * Components for transaction management provided from Spring Framework.

    * - | (7)
      - Bean definition file for defining the Infrastructure layer components.

        O/R Mapper etc beans are defined in this file.

|

.. code-block:: console

            └── test
                ├── java
                │   └── com
                │       └── example
                │           └── project
                │               └── domain
                │                   ├── repository
                │                   └── service
                └── resources
                    └── test-context.xml  ... (8)


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | Sr. No.
      - | Description
    * - | (8)
      - Bean definition file for defining the domain layer unit test components.

|

**In case of project created for MyBatis3**

.. code-block:: console

        └── src
            ├── main
            │   ├── java
           (...)
            │   └── resources
            │       ├── META-INF
            │       │   ├── dozer
            │       │   ├── mybatis  ... (9)
            │       │   │   └── mybatis-config.xml  ... (10)
            │       │   └── spring
           (...)
            │       └── com
            │           └── example
            │               └── project
            │                   └── domain
            │                       └── repository  ... (11)
            │                           └── sample
            │                               └── SampleRepository.xml  ... (12)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | Sr. No.
      - | Description
    * - | (9)
      - Directory that contains the MyBatis3 configuration files
    * - | (10)
      - MyBatis3 configuration files.

        Some of the recommended settings are defined at the time of creation.
    * - | (11)
      - Directory that contains the MyBatis3 Mapper files.
    * - | (12)
      - Sample file of MyBatis3 Mapper file.

        Sample implementation is in commented out state at the time of creation
        **Lastly, these files will not required.**

|

.. _CreateWebApplicationProjectConfigurationEnv:

Structure of Env module
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Module that manages the environment dependent configuration files are explained.

.. code-block:: console

    artifactId-env
        ├── configs  ... (1)
        │   ├── production-server  ... (2)
        │   │   └── resources
        │   └── test-server
        │       └── resources
        ├── pom.xml  ... (3)


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | Sr. No.
      - | Description
    * - | (1)
      - Directory for managing the environment dependent configuration files.

        Manage environment dependent configuration file by creating subdirectories of each environment.
    * - | (2)
      - Directory for managing the each environment configuration file.

        At the time of creation, following directories (directory template) are provided as most simple configuration.

        * production-server (Directory that contains the production environment configuration files)
        * test-server (Directory that contains the test environment configuration files)

    * - | (3)
      - The env module configuration is defined in POM (Project Object Model) file.
        Following definitions are done in this file.

        * Definition of dependent libraries and build plug-ins
        * Definition of Profile to create a jar file for each environment

|

.. code-block:: console

        └── src
            └── main
                └── resources  ... (4)
                    ├── META-INF
                    │   └── spring
                    │       ├── artifactId-env.xml  ... (5)
                    │       └── artifactId-infra.properties  ... (6)
                    ├── database  ... (7)
                    │   ├── H2-dataload.sql
                    │   └── H2-schema.sql
                    ├── dozer.properties  ... (8)
                    ├── log4jdbc.properties  ... (9)
                    └── logback.xml  ... (10)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | Sr. No.
      - | Description
    * - | (4)
      - Directory for managing configuration files of the development.
    * - | (5)
      - Bean definition file that defines the environment dependent components.

        Following beans are defined in this file.

        * Datasource
        * \ ``JodaTimeDateFactory`` \ offered by common library (In case of different implementations depending on the environment)
        * Components for transaction management provided by Spring Framework (In case of different implementations depending on the environment)

    * - | (6)
      - Property file that defines the environment dependent settings.

        At the time of creation, the DataSource settings are defined (Setting of the connection and connection pool)
    * - | (7)
      - Directory that contains the SQL to set up an in-memory database (H2 Database).

        This directory is prepared while performing small operation verification .
        **Basically remove this directory because this directory is not intended to use in the actual application development.**
    * - | (8)
      - Property file for carrying out the Dozer (Bean Mapper) global settings. For Dozer refer [:doc:`../ArchitectureInDetail/GeneralFuncDetail/Dozer`].

        It is an empty file at the time of creation. (The warning log appears at the start-up time if file is not exist, the empty file is prepared in order to prevent it)
    * - | (9)
      - Property file for carrying out the Log4jdbc-remix (library to perform the JDBC-related log output) global settings. For Log4jdbc-remix, refer [:ref:`DataAccessCommonDataSourceDebug`].

        At the time of creation, new line character related setting are specified for those SQLs which are going to be printed in log.
    * - | (10)
      - Configuration file of the Logback (log output).
        For the log output refer [:doc:`../ArchitectureInDetail/GeneralFuncDetail/Logging`].

|

.. _CreateWebApplicationProjectConfigurationInitdb:

Structure of Initdb module
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Module that manages the SQL file to initialize the database is explained.

.. code-block:: console

    artifactId-initdb
        ├── pom.xml  ... (1)
        └── src
            └── main
                └── sqls  ... (2)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | Sr. No.
      - | Description
    * - | (1)
      - The initdb module configuration is defined in POM (Project Object Model) file.
        Following definitions are done in this file.

        * Definition of build plug-ins (`SQL Maven Plugin <http://www.mojohaus.org/sql-maven-plugin/>`_)

        Simple configuration for PostgreSQL is defined at the time of creation.
    * - | (2)
      - Directory for storing the database initialization SQL files.

        It is an empty directory at the time of creation.
        For how to create, Refer `Sample application of initdb project <https://github.com/terasolunaorg/terasoluna-tourreservation-mybatis3/tree/5.3.0.RELEASE/terasoluna-tourreservation-initdb/src/main/sqls>`_.

.. note::

    Can be executed SQL using `sql:execute <http://www.mojohaus.org/sql-maven-plugin/execute-mojo.html>`_ of `SQL Maven Plugin <http://www.mojohaus.org/sql-maven-plugin/>`_.

        .. code-block:: console

            mvn sql:execute

|

.. _CreateWebApplicationProjectConfigurationSelenium:

Structure of Selenium module
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Module that manages the E2E (End To End) testing components used in Selenium explained.

.. code-block:: console

    artifactId-selenium
        ├── pom.xml  ... (1)
        └── src
            └── test  ... (2)
                ├── java
                │   └── com
                │       └── example
                │           └── project
                │               └── selenium
                │                   └── welcome
                │                       └── HelloTest.java  ... (3)
                └── resources
                    └── META-INF
                        └── spring
                            ├── selenium.properties  ... (4)
                            └── seleniumContext.xml  ... (5)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | Sr. No.
      - | Description
    * - | (1)
      - The selenium module configuration is defined in POM (Project Object Model) file.

        Following definitions are done in this file.

        * Definition of dependent libraries and build plug-ins
        * Definition to create a war file

    * - | (2)
      - Directory that contains the configuration files and testing components.

        For how to create, Refer `Sample application of selenium project <https://github.com/terasolunaorg/terasoluna-tourreservation-mybatis3/tree/5.3.0.RELEASE/terasoluna-tourreservation-selenium>`_.

    * - | (3)
      - Sample test class using Selenium WebDriver.

        At the time of creation, it has the test method for asserting a title of the Welcome page.

    * - | (4)
      - Properties file that defines the settings to be used in the test.

        The URL of the application server is \ ``http://localhost:8080/``\ at the time of creation.

    * - | (5)
      - Bean definition file for defining the test components.

        At the time of creation, it defines required settings for executing the sample test.

|

.. _CreateWebApplicationProjectAppendix:

Appendix
--------------------------------------------------------------------------------

.. _CreateWebApplicationProjectAppendixProjectHierarchicalStructure:

Hierarchical structure of the project
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The hierarchical structure of the project indicated below which is created in Maven Archetype.

.. figure:: images_CreateWebApplicationProject/CreateWebApplicationProjectHierarchicalStructure.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - | Sr. No.
      - | Description
    * - | (1)
      - Project created in Maven Archetype.

        The project created in Maven Archetype has become a multi-module configuration,
        parent project and each sub-module have a cross-reference relationship.

        In the project created in version 5.3.0.RELEASE Maven Archetype,
        [org.terasoluna.gfw:terasoluna-gfw-parent:5.3.0.RELEASE] is specified as a parent project.
    * - | (2)
      - TERASOLUNA Server Framework for Java (5.x) Parent project.

        In the TERASOLUNA Server Framework for Java (5.x) Parent project,

        * Plug-ins settings for build
        * Customization of libraries that is managed through Spring IO Platform (adjusted version)
        * Version management of recommended libraries that is not managed by Spring IO Platform

        are performed.

        Furthermore, in order to version management of the dependent libraries via Spring IO Platform, imported the [Spring IO Platform] into \ ``<dependencyManagement>`` \ of the project.
        
        Spring IO Platform Version (the current project using) is described in \ :ref:`frameworkstack_using_oss_version`\.
    * - | (3)
      - Spring IO Platform project.

        Since [org.springframework.boot:spring-boot-starter-parent:1.2.5.RELEASE] is specified as a parent project, the definition of \ ``<dependencyManagement>`` \ defined into pom file of the spring-boot-starter-parent also imported into pom file of the terasoluna-gfw-parent.
    * - | (4)
      - Spring Boot Starter Parent project.

        Since [org.springframework.boot:spring-boot-dependencies:1.2.5.RELEASE] is specified as a parent project,  the definition of \ ``<dependencyManagement>`` \ defined into pom file of the spring-boot-dependencies also imported into pom file of the terasoluna-gfw-parent.
    * - | (5)
      - Spring Boot Dependencies project.

.. raw:: latex

   \newpage

.. tip::

    The configuration has been changed like \ ``<dependencyManagement>`` \ of Spring IO Platform is imported from version 5.0.0.RELEASE,
    we have adopted a style that version management of recommended libraries are done in Spring IO Platform.


.. warning::

    Since the configuration has been changed like \ ``<dependencyManagement>`` \ of Spring IO Platform is imported from version 5.0.0.RELEASE,
    You are no longer able to access the version management properties from the child project.
    
    Therefore, if property values are referring or overwriting at the child project, pom file should be modified while upgrading from version 1.0.x.
    
    Furthermore, it is possible to access the conventional version management properties for recommended libraries (TERASOLUNA Server Framework for Java (5.x) recommended library)
    which are not managed by the Spring IO Platform.


|


.. _CreateWebApplicationProjectAppendixApplicationContext:

Relationship of bean definition file and application context structure
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Relationship of bean definition file and structure of the Spring Framework application context (DI container) indicated below.

.. figure:: images_CreateWebApplicationProject/CreateWebApplicationProjectApplicationContext.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | Sr. No.
      - | Description
    * - | (1)
      - Application context for the Web application.

        As shown in above diagram, Components defined in 

        * artifactId-web/src/main/resource/META-INF/spring/applicationContext.xml
        * artifactId-domain/src/main/resource/META-INF/spring/artifactId-domain.xml
        * artifactId-domain/src/main/resource/META-INF/spring/artifactId-infra.xml
        * artifactId-env/src/main/resource/META-INF/spring/artifactId-env.xml
        * artifactId-domain/src/main/resource/META-INF/spring/artifactId-codelist.xml
        * artifactId-web/src/main/resource/META-INF/spring/spring-security.xml

        are registered in the application context (DI container) for the Web application..

        Components registered in the application context for the Web application are
        mechanized such a way that it can be referred by the application context of each \ ``DispatcherServlet``\.
    * - | (2)
      - Application context for \ ``DispatcherServlet``\.

        As shown in above diagram, Components defined in 

        * artifactId-web/src/main/resource/META-INF/spring/spring-mvc.xml

        are registered in the application context (DI container) for the \ ``DispatcherServlet``\.

        Components not stored in the application context for the \ ``DispatcherServlet`` \ are
        mechanized such a way that it can be obtained by referring the application context of the Web application (parent context),
        hence it is possible to inject domain layer components for the application layer component.

.. raw:: latex

   \newpage

.. note:: **About the operation when registered the same components in both application contexts.**

    If same components are registered in both application context for web application and application context for \ ``DispatcherServlet``\, 
    injected component will be the registered component in the same application context(Application context for \ ``DispatcherServlet``\) and this point is supplemented here.

    In particular, it is necessary to be careful that do not register the domain layer component (such as Service and Repository) to application context for the \ ``DispatcherServlet``\.

    If domain layer components are registered to the application context for the \ ``DispatcherServlet``\, 
    trouble like the database operations are not committed occurs due to component that performs the transaction control (AOP) is not enabled.

    Furthermore, the settings are done in the project created using Maven Archetype so that the above events don't occur.
    It is necessary to be careful while performing modification or addition of the settings.

|

.. _CreateWebApplicationProjectAppendixDescribeConfigurationFile:

Description of the configuration file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo::

    In order to increase the understanding of various settings, planning to add explanation of a configuration file.

    * If functional description is explained somewhere, Reference to the functional description will be noted down.
    * If functional description is explained anywhere, description will be done here.

    Specific time-line is not decided yet.

|

Application development in offline environment
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In ":ref:`CreateWebApplicationProject`",
a method to create a development project of multi-project configuration
by using `archetype:generate <http://maven.apache.org/archetype/maven-archetype-plugin/generate-mojo.html>`_ of
`Maven Archetype Plugin <http://maven.apache.org/archetype/maven-archetype-plugin/>`_ is described.
Although Maven is used for the operations in the online environment,
a method is described below for how to use it in offline environment as well.

To continue project development in the offline environment,
the files like libraries and plugins necessary for development must be copied in advance.
The operation below should be performed in **online environment**.

|

Move to root directory of development project.
Here, the project created using ":ref:`CreateWebApplicationProject`" is used for the explanation.

.. code-block:: console

    cd C:\work\todo

|

Copy the files like libraries and plugins necessary for project development.
Files are copied by executing `dependency:go-offline <https://maven.apache.org/plugins/maven-dependency-plugin/go-offline-mojo.html>`_ of
`Maven Archetype Plugin <http://maven.apache.org/archetype/maven-archetype-plugin/>`_.

.. code-block:: console

    mvn dependency:go-offline -Dmaven.repo.local=repository

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 25 75

    * - Parameter
      - Description
    * - | \--Dmaven.repo.local
      - Specify copy destination.
        A new destination is created if a copy destination does not exist.
        At present, copy destination is specified as a repository.

|

Create a war file or a jar file in order to facilitate the distribution of deliverables.
At that time, files like libraries and plugins necessary for build are copied.

.. code-block:: console

    mvn package -Dmaven.repo.local=repository

|

When build is successful, the log shown below is output.

.. code-block:: console

	(... omit)    
	[INFO] ------------------------------------------------------------------------
	[INFO] Reactor Summary:
	[INFO]
	[INFO] TERASOLUNA Server Framework for Java (5.x) Web Blank Multi Project (MyBa
	tis3) SUCCESS [  0.006 s]
	[INFO] todo-env ........................................... SUCCESS [ 46.565 s]
	[INFO] todo-domain ........................................ SUCCESS [  0.684 s]
	[INFO] todo-web ........................................... SUCCESS [ 12.832 s]
	[INFO] todo-initdb ........................................ SUCCESS [  0.067 s]
	[INFO] todo-selenium ...................................... SUCCESS [01:13 min]
	[INFO] ------------------------------------------------------------------------
	[INFO] BUILD SUCCESS
	[INFO] ------------------------------------------------------------------------
	[INFO] Total time: 02:14 min
	[INFO] Finished at: 2015-10-01T10:32:34+09:00
	[INFO] Final Memory: 36M/206M
	[INFO] ------------------------------------------------------------------------

|

Above, files like libraries and plugins necessary for project development are copied.
Operation is completed when the repository is copied to ${HOME}/.m2 of offline environment machine.
If a process which has not been executed even once in online environment is executed in offline environment,
necessary files like libraries and plugins cannot be fetched resulting in the process failure.
However, by copying the files, the development can be continued uninterrupted even after moving to offline environment.

.. warning:: **Precautions for the development in offline environment**

    Since it is not possible to fetch a new dependency relation from internet in the offline environment,
    POM (Project Object Model) file should not be edited.
    It is necessary to return to online environment again for editing POM file.

.. raw:: latex

   \newpage
