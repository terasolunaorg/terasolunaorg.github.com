Tutorial (Todo Application)
********************************************************************************

.. only:: html

 .. contents:: Index
    :depth: 3
    :local:

Introduction
================================================================================

Points to study in this tutorial
--------------------------------------------------------------------------------

* Basic application development and configuration of Eclipse project using TERASOLUNA Global Framework
* Development in accordance with application layering of this TERASOLUNA Global Framework


Target readers
--------------------------------------------------------------------------------

* Basic knowledge of DI and AOP of Spring is required
* Web application development using Servlet/JSP
* SQL knowledge


Verification environment
--------------------------------------------------------------------------------

In this tutorial, operations are verified in the following environment.

.. tabularcolumns:: |p{0.15\linewidth}|p{0.85\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 15 85

    * - Type
      - Name
    * - OS
      - Windows7 64bit
    * - JVM
      - Java 1.6
    * - IDE
      - Spring Tool Suite Version: 3.2.0.RELEASE, Build Id: 201303060821 (henceforth referred to STS) Build Maven 3.0.4 (STS attached)
    * - Application Server
      - VMWare vFabric tc Server Developer Edition v2.8 (STS attached)
    * - Web Browser
      - Google Chrome 27.0.1453.94 m

Description of application to be created
================================================================================

Overview of application
--------------------------------------------------------------------------------

Application to manage TODO list is to be developed. TODO list display, TODO registration, TODO completion and TODO deletion can be performed.


.. figure:: ./images/image001.png
   :width: 60%


.. _app-requirement:

Business requirements of application
--------------------------------------------------------------------------------

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - RuleID
      - Description
    * - B01
      - Only up to 5 incomplete TODO records can be registered
    * - B02
      - For TODOs which are already completed, "TODO Complete" processing cannot be done.

|

.. note::

  This application is for learning purpose only. It is not suitable as a real todo management application.

|

Screen transition of application
--------------------------------------------------------------------------------


.. figure:: ./images/image002.png
   :width: 60%



.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|p{0.40\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 20 15 15 40

    * - Sr.No.
      - Process name
      - HTTP method
      - URL
      - Description
    * - 1
      - Show all TODO
      - GET
      - /todo/list
      -
    * - 2
      - Create TODO
      - POST
      - /todo/create
      - Redirect to 1 after creation is completed
    * - 3
      - Finish TODO
      - POST
      - /todo/finish
      - Redirect to 1 after creation is completed
    * - 4
      - Delete TODO
      - POST
      - /todo/delete
      - Redirect to 1 after creation is completed

Show all TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* Display all records of TODO
* Provide ``Finish`` and  ``Delete`` buttons for incomplete TODO
* Strike-through the completed records of TODO
* Only record name of TODO


Create TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* Save TODO sent from the form
* Record name of TODO should be between 1 - 30 characters
* When :ref:`app-requirement` B01 is not fulfilled, business exception with error code E001 is thrown

Finish TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* For the TODOs corresponding to todoId which is received from the form object, change the status to ``completed``.
* When :ref:`app-requirement` B02 is not fulfilled, business exception with error code E002 is thrown
* When the corresponding TODO does not exist, business exception with error code E404 is thrown

Delete TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* Delete TODO corresponding to todoId sent from the form
* When the corresponding TODO does not exist, business exception with error code E404 is thrown


Error message list
--------------------------------------------------------------------------------

.. tabularcolumns:: |p{0.15\linewidth}|p{0.45\linewidth}|p{0.40\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 15 45 40

    * - Error code
      - Message
      - Parameter to be replaced
    * - E001
      - [E001] The count of un-finished Todo must not be over {0}.
      - {0}… max unfinished count
    * - E002
      - [E002] The requested Todo is already finished. (id={0})
      - {0}… todoId
    * - E404
      - [E404] The requested Todo is not found. (id={0})
      - {0}… todoId



Environment creation
================================================================================

Project creation
--------------------------------------------------------------------------------

Select ``File`` -> ``Other`` -> ``Maven`` -> ``Maven Project`` and proceed to ``Next``



.. figure:: ./images/image004.jpg
   :width: 60%

Insert check-mark to ``Create a simple project`` and proceed to ``Next``

.. figure:: ./images/image006.jpg
   :width: 60%


.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table::
    :widths: 25 75
    :stub-columns: 1

    * - Group Id:
      - org.terasoluna.tutorial
    * - Artifact Id:
      - todo
    * - Packaging:
      - war

``Finish``

.. figure:: ./images/image008.jpg
   :width: 60%

Project as shown below is created.


.. figure:: ./images/image009.png
   :width: 40%

|

.. note::

  For better visibility, Package Presentation must be changed to Hierarchical.

  .. figure:: ./images/presentation-hierarchical.png
     :width: 80%

Maven settings
--------------------------------------------------------------------------------

Change pom.xml as follows.
If basic knowledge of Maven is not there, then just copy the pom.xml and skip the below explanation. 

.. code-block:: xml
   :emphasize-lines: 9-83

    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>

        <groupId>org.terasoluna.tutorial</groupId>
        <artifactId>todo</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <packaging>war</packaging>
        <!-- (1) -->
        <parent>
            <groupId>org.terasoluna.gfw</groupId>
            <artifactId>terasoluna-gfw-parent</artifactId>
            <version>1.0.0.RELEASE</version>
        </parent>

        <!-- (2) -->
        <repositories>
            <repository>
                <releases>
                    <enabled>true</enabled>
                </releases>
                <snapshots>
                    <enabled>false</enabled>
                </snapshots>
                <id>terasoluna-gfw-releases</id>
                <url>http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases/</url>
            </repository>
            <repository>
                <releases>
                    <enabled>false</enabled>
                </releases>
                <snapshots>
                    <enabled>true</enabled>
                </snapshots>
                <id>terasoluna-gfw-snapshots</id>
                <url>http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-snapshots/</url>
            </repository>
            <repository>
                <releases>
                    <enabled>true</enabled>
                </releases>
                <snapshots>
                    <enabled>false</enabled>
                </snapshots>
                <id>terasoluna-gfw-3rdparty</id>
                <url>http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-3rdparty/</url>
            </repository>
        </repositories>

        <dependencies>
            <!-- (3) -->
            <!-- TERASOLUNA -->
            <dependency>
                <groupId>org.terasoluna.gfw</groupId>
                <artifactId>terasoluna-gfw-web</artifactId>
            </dependency>
            <!-- (4) -->
            <dependency>
                <groupId>org.terasoluna.gfw</groupId>
                <artifactId>terasoluna-gfw-security-web</artifactId>
                </dependency>
            <!-- (5) -->
            <dependency>
                <groupId>org.terasoluna.gfw</groupId>
                <artifactId>terasoluna-gfw-recommended-dependencies</artifactId>
                <type>pom</type>
            </dependency>

            <!-- (6) -->
            <!-- Servlet API/ JSP API -->
            <dependency>
                <groupId>org.apache.tomcat</groupId>
                <artifactId>tomcat-servlet-api</artifactId>
                <version>7.0.40</version>
                <scope>provided</scope>
            </dependency>
            <dependency>
                <groupId>org.apache.tomcat</groupId>
                <artifactId>tomcat-jsp-api</artifactId>
                <version>7.0.40</version>
                <scope>provided</scope>
            </dependency>
        </dependencies>
    </project>


Right click on the project name in "Package Explorer" and select [Maven] -> [Update Project]


.. figure:: ./images/update-project.png
   :width: 60%

Press "OK" button.

Confirm that the version of "JRE System Library" is "[JavaSE-1.6]". 


.. figure:: ./images/check-jre.jpg
   :width: 30%

|

    .. note::
        In order to update the version of JDK to 7, set ``<java-version>1.7</java-version>`` in ``<properties>``  of pom.xml. 
        After that, execute "Update Project"

            .. code-block:: xml
               :emphasize-lines: 4-6

                <project>
                    <!-- omitted -->

                    <properties>
                        <java-version>1.7</java-version>
                    </properties>
                </project>

If you are familiar with the Maven, and to make sure the following discussion.

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr.No.
     - Description
   * - | (1)
     - | Specify parent pom file of TERASOLUNA Global Framework.
       | In this way, even without specifying the version, the library defined in terasoluna-parent can be added to dependency.
   * - | (2)
     - | Specify URL of Maven repository for using TERASOLUNA Global Framework.
   * - | (3)
     - | Add common library of TERASOLUNA Global Framework to dependency.
   * - | (4)
     - | Add the library group recommended by TERASOLUNA Global Framework.
       | Since terasoluna-gfw-recommended-dependencies is just pom file, ``<type>pom</type>`` should be mentioned.
   * - | (5)
     - | Add the libraries which are recommended by TERASOLUNA Global Framework
       | terasoluna-gfw-recommended-dependencies is just a pom file; hence ``<type>pom</type>``  must be specified.
   * - | (6)
     - | Add Servlet/JSP API to dependency. Compatiblity with Servlet3 is necessary.
       | These API have scope=provided (provided by the original AP server), they are not included in war, but it should be added explicitly to dependency for compiling on eclipse.
       | （Further, though dependency name is tomcat-xxx, but the package of embedded class is javax.servlet, there is no dependency on tomcat）


	.. note:: When proxy server is used to access the internet,
	     perform the following settings in <HOME>/.m2/settings.xml.
	    (In case of Windows7 C:\\Users\\<YourName>\\.m2\settings.xml)

	    	.. code-block:: xml

			        <settings>
			          <proxies>
			            <proxy>
			              <active>true</active>
			              <protocol>[Proxy Server Protocol (http)]</protocol>
			              <port>[Proxy Server Port]</port>
			              <host>[Proxy Server Host]</host>
			              <username>[Username]</username>
			              <password>[Password]</password>
			            </proxy>
			          </proxies>
			        </settings>


Project configuration
--------------------------------------------------------------------------------

Below is the structure of the project to be created.

    .. code-block:: console

        src
      └main
          ├java
          │  └todo
          │    ├ app ... Application layer
          │    │   └todo ... Classes related todo management business process
          │    └domain ... Domain layer
          │        ├model ... Domain Object
          │        ├repository ... Repository
          │        │   └todo ... Repository related to Todo
          │        └service ... Services 
          │            └todo ... Service related to Todo
          ├resources
          │  └META-INF
          │      └spring ... configuration files related to Spring
          └wepapp
              └WEB-INF
                  └views ... jsp


Since above will be created in order, there is no need to provide prepare the above structure beforehand.

|

.. note::

  It had been recommended to use a multi-project structure in :ref:`"Project Structure" section of previous chapter <application-layering_project-structure>` .
  In this tutorial, a single project configuration is used because it focuses on ease of learning. However, when in a real project, multi project configuration is
  strongly recommended.

|

Creation of configuration file
--------------------------------------------------------------------------------

web.xml settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Create ``src/main/webapp/WEB-INF/web.xml`` and define servlet and filters.
New WEB-INF folder should be created by ``New`` -> ``Folder``.


.. figure:: ./images/image010.jpg
   :width: 40%

Create web.xml by ``New`` -> ``File``,


.. figure:: ./images/image011.jpg
   :width: 40%


and describe the contents as follows.

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!-- (1) -->
    <web-app xmlns="http://java.sun.com/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
        version="3.0">
        <!-- (2) -->
        <listener>
            <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
        </listener>
        <context-param>
            <param-name>contextConfigLocation</param-name>
            <!-- Root ApplicationContext -->
            <param-value>
                classpath*:META-INF/spring/applicationContext.xml
            </param-value>
        </context-param>

        <!-- (3) -->
        <filter>
            <filter-name>CharacterEncodingFilter</filter-name>
            <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
            <init-param>
                <param-name>encoding</param-name>
                <param-value>UTF-8</param-value>
            </init-param>
            <init-param>
                <param-name>forceEncoding</param-name>
                <param-value>true</param-value>
            </init-param>
        </filter>
        <filter-mapping>
            <filter-name>CharacterEncodingFilter</filter-name>
            <url-pattern>/*</url-pattern>
        </filter-mapping>

        <!-- (4) -->
        <servlet>
            <servlet-name>appServlet</servlet-name>
            <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
            <init-param>
                <param-name>contextConfigLocation</param-name>
                <!-- ApplicationContext for Spring MVC -->
                <param-value>classpath*:META-INF/spring/spring-mvc.xml</param-value>
            </init-param>
            <load-on-startup>1</load-on-startup>
        </servlet>

        <servlet-mapping>
            <servlet-name>appServlet</servlet-name>
            <url-pattern>/</url-pattern>
        </servlet-mapping>

        <!-- (5) -->
        <jsp-config>
            <jsp-property-group>
                <url-pattern>*.jsp</url-pattern>
                <el-ignored>false</el-ignored>
                <page-encoding>UTF-8</page-encoding>
                <scripting-invalid>false</scripting-invalid>
                <include-prelude>/WEB-INF/views/common/include.jsp</include-prelude>
            </jsp-property-group>
        </jsp-config>
    </web-app>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr.No.
     - Description
   * - | (1)
     - | Declaration for using Servlet3.0.
   * - | (2)
     - | Define ``ContextLoaderListener``. ``ApplicationContext`` created by this listener is the root context.
       | Path of Bean definition file is ``META-INF/spring/applicationContext.xml`` just under the classpath.
   * - | (3)
     - | Define ``CharacterEncodingFilter``. This is done for changing the character encoding of request and response to UTF-8.
   * - | (4)
     - | Define ``DispatcherServlet`` that is the entry point of Spring MVC.
       | Path of Bean definition file to be used in Spring MVC is ``META-INF/spring/spring-mvc.xml`` just under the classpath.
       | ``ApplicationContext`` created here is the child of ``ApplicationContext`` created in step (2).
   * - | (5)
     - | Define common JSP to be included. Include ``/WEB-INF/views/common/include.jsp`` for any JSP(\*.JSP)


.. figure:: ./images/image013.png
   :width: 40%

|

Settings of common JSP
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Describe the contents to be included in each common JSP in src/main/webapp/WEB-INF/views/common/include.jsp. Also define taglib in common area.
Create views/common folder and include.jsp file and describe as follows.


.. code-block:: jsp

    <%@ page session="false"%>
    <!-- (1) -->
    <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
    <%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt"%>
    <!-- (2)  -->
    <%@ taglib uri="http://www.springframework.org/tags" prefix="spring"%>
    <%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>
    <!-- (3) -->
    <%@ taglib uri="http://www.springframework.org/security/tags" prefix="sec"%>
    <!-- (4) -->
    <%@ taglib uri="http://terasoluna.org/functions" prefix="f"%>
    <%@ taglib uri="http://terasoluna.org/tags" prefix="t"%>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr.No.
     - Description
   * - | (1)
     - | Define standard tag library.
   * - | (2)
     - | Define tag library for Spring MVC.
   * - | (3)
     - | Define tag library for Spring Security.(However, it is not used in this tutorial)
   * - | (4)
     - | Define EL function and tag library provided in common library.




.. figure:: ./images/image014.png
   :width: 40%

|

Settings of Bean definition file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create 4 types of Bean definition files in the following order.

* applicationContext.xml
* todo-domain.xml
* todo-infra.xml
* spring-mvc.xml


applicationContext.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Carry out settings related to entire Todo application in src/main/resources/META-INF/spring/applicationContext.xml.


Create META-INF/spring folder and create applicationContext.xml using ``New`` -> ``Spring Bean Configuration File``.



.. figure:: ./images/image016.jpg
   :width: 40%



.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

        <!-- (1) -->
        <import resource="classpath:/META-INF/spring/todo-domain.xml" />

        <!-- (2) -->
        <context:property-placeholder
            location="classpath*:/META-INF/spring/*.properties" />

        <!-- (3) -->
        <bean class="org.dozer.spring.DozerBeanMapperFactoryBean">
            <property name="mappingFiles"
                value="classpath*:/META-INF/dozer/**/*-mapping.xml" />
        </bean>

    </beans>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr.No.
     - Description
   * - | (1)
     - | Import Bean definition file related to domain layer.
   * - | (2)
     - | Read the settings of property file.
       | Read any property file under ``src/main/resources/META-INF/spring``.
       | Using this setting, it is possible to insert property file value in ${propertyName} format in Bean definition file and to inject using @Value("${propertyName}") in Java class. 
   * - | (3)
     - | Define Mapper of library Dozer for Bean conversion.
       | (Not used in this tutorial, but while defining XML file for mapping, it should be created in ``src/main/resources/META-INF/dozer/xxx-mapping.xml`` format.
       | For the mapping file, refer to `Dozer manual <http://dozer.sourceforge.net/documentation/mappings.html>`_ .)

.. figure:: ./images/image018.png
   :width: 40%

|

.. note::
    While entering the above contents manually without copying them, open ``namespace`` tab and insert check-mark in ``beans`` and ``context`` in ``Configure Namespace``.
    It is recommended to select xsd file without version in ``Namespace Versions``.

    .. figure:: ./images/image021.jpg
       :width: 60%
       :align: center

    Thus, at the time of editing XML, it is possible to supplement the input using Ctrl+Space.

    .. figure:: ./images/image023.png
       :width: 60%
       :align: center

    Moreover, by not specifying the version, latest xsd included in the jar is used.
    
|

todo-domain.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Carry out settings related to domain layer in ``src/main/resources/META-INF/spring/todo-domain.xml``.


Create todo-domain.xml using ``New`` -> ``Spring Bean Configuration File`` under ``META-INF/spring``.


.. code-block:: xml


    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
        <!-- (1) -->
        <import resource="classpath:META-INF/spring/todo-infra.xml"/>
        <!-- (2) -->
        <context:component-scan base-package="todo.domain" />
    </beans>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr.No.
     - Description
   * - | (1)
     - | Import Bean definition file related to infrastructure layer (explained later).
   * - | (2)
     - | Components under todo.domain package are target of component scan.
       | Thus, it is possible to make DI target by attaching annotations like ``@Repository`` , ``@Service`` , ``@Controller``, ``@Component`` to the class under todo.domain package.

.. figure:: ./images/image024.png
   :width: 40%

|

todo-infra.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Define Beans related to infrastructure layer in ``src/main/resources/META-INF/spring/todo-infra.xml``.
Here, DB setting are carried out, but DB is not used in this section, hence the definition may be blank as follows. Bean will be defined in next section.


Create todo-infra.xml using ``New`` -> ``Spring Bean Configuration File`` under ``META-INF/spring``.


.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    </beans>


.. figure:: ./images/image025.png
   :width: 40%

|

.. note:: All the contents of todo-domain.xml, todo-infra.xml may likely be described in applicationContext.xml, however
    it is recommended to split the file for each layer. It will be easy to understand the definitions at various locations and to improve maintainability.
    There is no effect on a small application like the current tutorial, but larger the scope more the effect.


spring-mvc.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Define Spring MVC related definitions in ``src/main/resources/META-INF/spring/spring-mvc.xml``.


.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
        xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:util="http://www.springframework.org/schema/util"
        xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

        <!-- (1) -->
        <mvc:annotation-driven></mvc:annotation-driven>

        <!-- (2) -->
        <context:component-scan base-package="todo.app" />

        <!-- (3) -->
        <mvc:resources mapping="/resources/**"
            location="/resources/,classpath:META-INF/resources/"
            cache-period="#{60 * 60}" />

        <mvc:interceptors>
            <!-- (4) -->
            <mvc:interceptor>
                <mvc:mapping path="/**" />
                <mvc:exclude-mapping path="/resources/**" />
                <bean
                    class="org.terasoluna.gfw.web.logging.TraceLoggingInterceptor" />
            </mvc:interceptor>
        </mvc:interceptors>

        <!-- (5) -->
        <bean id="viewResolver"
            class="org.springframework.web.servlet.view.InternalResourceViewResolver">
            <property name="prefix" value="/WEB-INF/views/" />
            <property name="suffix" value=".jsp" />
        </bean>
    </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr.No.
     - Description
   * - | (1)
     - | Carry out annotation based default settings of Spring MVC.
   * - | (2)
     - | Components under todo.app package that holds classes of application layer are made target of component-scan.
   * - | (3)
     - | Carry out the settings for accessing the static resource (css, images, js etc.).
       | Set URL path to ``mapping`` attribute and physical path to ``location`` attribute.
       | In case of this setting, when there is a request for ``<contextPath>/resources/css/styles.css``, ``WEB-INF/resources/css/styles.css`` is searched. 
       | If not found, ``resources/css/style.css`` is searched in classpath (src/main/resources and jar). If not found again, 404 error is returned.
       | Cache period (3600 seconds = 60 minutes) of static resources is set in ``cache-period`` attribute.
       | Further, static resources are not used in this tutorial.
       | ``cache-period="3600"`` is also correct, however, in order to demostrate that it is 60 minutes, it is better to write as ``cache-period="#{60 * 60}"`` which uses `SpEL <http://static.springsource.org/spring/docs/3.2.x/spring-framework-reference/html/expressions.html#expressions-beandef-xml-based>`_ .
   * - | (4)
     - | Set interceptor that outputs trace log of controller processing. Set so that it excludes the path under ``/resources`` from mapping.
   * - | (5)
     - | Carry out the settings of ViewResolver. Using these settings, for example, when view name ``hello`` is returned from controller, ``/WEB-INF/views/hello.jsp`` is executed.


.. figure:: ./images/image026.png
   :width: 40%

|

	.. note:: While entering the above contents manually without copying them, in addition to the operations described in todo-domain.xml, check mark should be put also to "mvc" and "util".

	    .. figure:: ./images/image028.png
	       :width: 60%
	       :align: center

|

logback.xml settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Carry out log output settings using logback in ``src/main/resources/logback.xml``.


Create logback.xml by ``New`` -> ``File`` just under ``src/main/resources/``.

.. code-block:: xml

    <!DOCTYPE logback>
    <configuration>
        <!-- (1) -->
        <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern><![CDATA[%d{yyyy-MM-dd HH:mm:ss} [%thread] [%-5level] [%-48logger{48}] - %msg%n]]></pattern>
            </encoder>
        </appender>

        <!-- Application Loggers -->
        <!-- (2) -->
        <logger name="todo">
            <level value="debug" />
        </logger>

        <!-- TERASOLUNA -->
        <!-- (3) -->
        <logger name="org.terasoluna.gfw">
            <level value="info" />
        </logger>
        <!-- (4) -->
        <logger name="org.terasoluna.gfw.web.logging.TraceLoggingInterceptor">
            <level value="trace" />
        </logger>

        <!-- 3rdparty Loggers -->
        <!-- (5) -->
        <logger name="org.springframework">
            <level value="warn" />
        </logger>

        <!-- (6) -->
        <logger name="org.springframework.web.servlet">
            <level value="info" />
        </logger>

        <!-- (7) -->
        <root level="WARN">
            <appender-ref ref="STDOUT" />
        </root>
    </configuration>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (1)
     - | Set appender that outputs the log in standard output.
   * - | (2)
     - | Set so that log of level 'debug' and above is output under todo package.
   * - | (3)
     - | Change the log level of common library to info.
   * - | (4)
     - | Set log level to 'trace' for ``TraceLoggingInterceptor`` which is defined in spring-mvc.xml.  
   * - | (5)
     - | Set so that log of level 'warn' and above is output for Spring Framework.
   * - | (6)
     - | For log of Spring Framework, set the log level to 'info' and above for ``org.springframework.web.servlet`` so that logs valueable to development activity gets output.
   * - | (7)
     - | Set so that log level of 'warn' and above is output by default.


.. figure:: ./images/image029.png
   :width: 40%

|

Operation verification
--------------------------------------------------------------------------------
Before starting development of Todo application, create SpringMVC HelloWorld application and verify the operation.


.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table::
   :widths: 25 75
   :stub-columns: 1

   * - Package:
     - todo.app.hello
   * - Name:
     - HelloController

Create todo.app.hello.HelloController using ``New`` -> ``Class``.


.. figure:: ./images/image030.jpg
   :width: 40%


Edit HelloController as shown below.

.. code-block:: java

    package todo.app.hello;

    import java.util.Date;

    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.web.bind.annotation.RequestMapping;

    // (1)
    @Controller
    public class HelloController {
        // (2)
        private static final Logger logger = LoggerFactory
                .getLogger(HelloController.class);

        // (3)
        @RequestMapping("/")
        public String hello(Model model) {
            Date now = new Date();
            // (4)
            logger.debug("hello {}", now);
            // (5)
            model.addAttribute("now", now);
            // (6)
            return "hello";
        }
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (1)
     - | In order to make the Controller as component-scan target, attach ``@Controller`` annotation to class level.
   * - | (2)
     - | Generate logger. Since logback implements logger and API is SLF4J, ``org.slf4j.Logger`` should be used.
   * - | (3)
     - | Set mapping of methods for accessing ``/`` (root) using ``@RequestMapping``.
   * - | (4)
     - | Output debug log. ``{}`` is the placeholder.
   * - | (5)
     - | For passing date to the screen, add Date object with name ``now`` to Model.
   * - | (6)
     - | Return hello as view name. Using ViewResolver settings, WEB-INF/views/hello.jsp is output.


Next, create view(jsp). Create src/main/webapp/WEB-INF/views/hello.jsp as follows.

.. code-block:: jsp

    <!DOCTYPE html>
    <html>
    <head>
    <title>Hello World!</title>
    </head>
    <body>
        <h1>Hello World!</h1>
        <p>
            Today is
            <!-- (1) -->
            <fmt:formatDate value="${now}" pattern="yyyy-MM-dd HH:mm:ss" />
        </p>
    </body>
    </html>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (1)
     - | Display ``now`` passed from Controller. Here, ``<fmt:formatDate>`` tag is used for date formating.

Right click package project name ``todo`` and click ``Run As`` -> ``Run on Server``



.. figure:: ./images/image031.jpg
   :width: 40%

Select AP server (here, VMWare vFabric tc Server Developer Edition v2.8) to be executed
Click ``Next``

.. figure:: ./images/image032.jpg
   :width: 40%

Verify that todo is included in ``Configured``, click ``Finish`` to start the server.

.. figure:: ./images/image033.jpg
   :width: 40%


When started, log shown as below will be output. For ``/`` path, it is understood that hello method of ``todo.app.hello.HelloController`` is mapped.


.. code-block:: guess
   :emphasize-lines: 3

    2013-06-14 14:26:54 [localhost-startStop-1] [WARN ] [org.dozer.config.GlobalSettings                 ] - Dozer configuration file not found: dozer.properties.  Using defaults for all Dozer global properties.
    2013-06-14 14:26:54 [localhost-startStop-1] [INFO ] [o.springframework.web.servlet.DispatcherServlet ] - FrameworkServlet 'appServlet': initialization started
    2013-06-14 14:26:54 [localhost-startStop-1] [INFO ] [o.s.w.s.m.m.a.RequestMappingHandlerMapping      ] - Mapped "{[/],methods=[],params=[],headers=[],consumes=[],produces=[],custom=[]}" onto public java.lang.String todo.app.hello.HelloController.hello(org.springframework.ui.Model)
    2013-06-14 14:26:55 [localhost-startStop-1] [INFO ] [o.s.web.servlet.handler.SimpleUrlHandlerMapping ] - Mapped URL path [/resources/**] onto handler 'org.springframework.web.servlet.resource.ResourceHttpRequestHandler#0'
    2013-06-14 14:26:55 [localhost-startStop-1] [INFO ] [o.springframework.web.servlet.DispatcherServlet ] - FrameworkServlet 'appServlet': initialization completed in 986 ms

|

	.. note::  WARN log of the first row may be ignored. In order to prevent, a blank dozer.properties should be created in src/main/resources.


If http://localhost:8080/todo is accessed in browser,
following is displayed.


.. figure:: ./images/image034.png
   :width: 40%


If you see console, you will understand that TRACE log using ``TraceLoggingInterceptor`` and debug log implemented by Controller is output.

.. code-block:: guess

    2013-06-14 15:40:59 [tomcat-http--3] [TRACE] [o.t.gfw.web.logging.TraceLoggingInterceptor     ] - [START CONTROLLER] HelloController.hello(Model)
    2013-06-14 15:40:59 [tomcat-http--3] [DEBUG] [todo.app.hello.HelloController                  ] - hello Fri Jun 14 15:40:59 JST 2013
    2013-06-14 15:40:59 [tomcat-http--3] [TRACE] [o.t.gfw.web.logging.TraceLoggingInterceptor     ] - [END CONTROLLER  ] HelloController.hello(Model)-> view=hello, model={now=Fri Jun 14 15:40:59 JST 2013}
    2013-06-14 15:40:59 [tomcat-http--3] [TRACE] [o.t.gfw.web.logging.TraceLoggingInterceptor     ] - [HANDLING TIME   ] HelloController.hello(Model)-> 15,043,704 ns

|

	.. note:: ``TraceLoggingInterceptor`` outputs start and end of Controller in log. While ending, View and Model information and processing time are output.

After verification of log, one can delete HelloController and hello.jsp.

|

Creation of Todo application
================================================================================
| Create Todo application. Order in which it must be created is as follows


* Domain layer (+ Infrastructure layer)
 * Domain Object creation
 * Repository creation
 * Service creation
* Application layer
 * Controller creation
 * Form creation
 * View creation

Further, do not use DB for saving Todo in this section. Creation of Repository in which DB is used, is carried out in \ :ref:`tutorial-todo_infra`\ .

|

Creation of Domain layer
--------------------------------------------------------------------------------

Creation of Domain Object
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Following properties are required in domain object.

#. ID
#. Title
#. Completion flag
#. Created on



Create the following Domain objects.
FQCN should be ``todo.domain.model.Todo``. Implement as JavaBean.


.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table::
   :widths: 25 75
   :stub-columns: 1

   * - Package:
     - todo.domain.model
   * - Name:
     - Todo
   * - Interfaces:
     - java.io.Serializable


.. figure:: ./images/image057.png
   :width: 40%


.. code-block:: java

    package todo.domain.model;

    import java.io.Serializable;
    import java.util.Date;

    public class Todo implements Serializable {
        private static final long serialVersionUID = 1L;


        private String todoId;

        private String todoTitle;

        private boolean finished;

        private Date createdAt;

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
    }


.. figure:: ./images/image058.png
   :width: 40%

|

	.. note::
	    Getter/Setter can be generated automatically. After defining fields, right click ``Source`` -> ``Generate Getter and Setters…``


	.. figure:: ./images/image059.png
	   :width: 40%


	Click ``OK`` after selecting all other than serialVersionUID


	.. figure:: ./images/image060.png
	   :width: 40%



Repository creation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Following are the steps of CRUD operations pertaining to TODO object required in the current application.

* Fetch 1 record of TODO
* Fetch all records of TODO
* Delete 1 record of TODO
* Update 1 record of TODO
* Fetch record count of completed TODO

Create interface TodoRepository that defines these operations.
FQCN should be ``todo.domain.repository.todo.TodoRepository``.

.. code-block:: java

    package todo.domain.repository.todo;

    import java.util.Collection;

    import todo.domain.model.Todo;

    public interface TodoRepository {
        Todo findOne(String todoId);

        Collection<Todo> findAll();

        Todo save(Todo todo);

        void delete(Todo todo);

        long countByFinished(boolean finished);
    }


.. figure:: ./images/image061.png
   :width: 40%

|

	.. note::
	    Here, to improve versatility of TodoRepository, method is defined to fetch ``record count having x completion status`` and not ``Fetch completed record count``.


Creation of RepositoryImpl (Infrastructure layer)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| For simplification, in-memory implementation that uses Map as the implementation of Repository is used.
| Repository implementation using DB is described in \ :ref:`tutorial-todo_infra`\ .
| FQCN should be ``todo.domain.repository.todo.TodoRepositoryImpl``.
| \ ``@Repository``\ annotation must be used at class level.

.. code-block:: java

    package todo.domain.repository.todo;

    import java.util.Collection;
    import java.util.Map;
    import java.util.concurrent.ConcurrentHashMap;

    import org.springframework.stereotype.Repository;

    import todo.domain.model.Todo;

    @Repository // (1)
    public class TodoRepositoryImpl implements TodoRepository {
        private static final Map<String, Todo> TODO_MAP = new ConcurrentHashMap<String, Todo>();

        @Override
        public Todo findOne(String todoId) {
            return TODO_MAP.get(todoId);
        }

        @Override
        public Collection<Todo> findAll() {
            return TODO_MAP.values();
        }

        @Override
        public Todo save(Todo todo) {
            return TODO_MAP.put(todo.getTodoId(), todo);
        }

        @Override
        public void delete(Todo todo) {
            TODO_MAP.remove(todo.getTodoId());
        }

        @Override
        public long countByFinished(boolean finished) {
            long count = 0;
            for (Map.Entry<String, Todo> e : TODO_MAP.entrySet()) {
                Todo todo = e.getValue();
                if (finished == todo.isFinished()) {
                    count++;
                }
            }
            return count;
        }
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (1)
     - | To consider Repository as component scan target, add \ ``@Repository``\ annotation at class level.


Since the business rules must not be included in Repository, it should focus only on inserting and removing information from the persistence store (here, it is Map). 


.. figure:: ./images/image062.png
   :width: 40%
\
 .. note::
 
     If package is divided completed on the basis of layers, it is better create classes of infrastructure layer under ``todo.infrastructure``.
     However, in a normal project, infrastructure layer rarely changes.
     Hence, in order to improve the work efficiency, RepositoryImpl can be created in the layer same as the repository of domain layer.


Service creation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Implement business logic. The required processes are as follows.

* Fetch all records of Todo
* New creation of Todo
* Todo completion
* Todo deletion

First, create TodoService interface and then define the above.
FQCN shold be ``todo.domain.serivce.todo.TodoService``.

.. code-block:: java

    package todo.domain.service.todo;

    import java.util.Collection;

    import todo.domain.model.Todo;

    public interface TodoService {
        Collection<Todo> findAll();

        Todo create(Todo todo);

        Todo finish(String todoId);

        void delete(String todoId);
    }

The required processes and the corresponding implementation methods are as follows

* Fetch all records of Todo→ findAll method
* New creation of Todo→create method
* Todo completion→finish method
* Todo deletion→delete method


.. figure:: ./images/image063.png
   :width: 40%


FQCN of implementation class should be ``todo.domain.service.TodoServiceImpl``.

.. code-block:: java

    package todo.domain.service.todo;

    import java.util.Collection;
    import java.util.Date;
    import java.util.UUID;

    import javax.inject.Inject;

    import org.springframework.stereotype.Service;
    //import org.springframework.transaction.annotation.Transactional;
    import org.terasoluna.gfw.common.exception.BusinessException;
    import org.terasoluna.gfw.common.exception.ResourceNotFoundException;
    import org.terasoluna.gfw.common.message.ResultMessage;
    import org.terasoluna.gfw.common.message.ResultMessages;

    import todo.domain.model.Todo;
    import todo.domain.repository.todo.TodoRepository;

    @Service// (1)
    // @Transactional // (2)l
    public class TodoServiceImpl implements TodoService {
        @Inject// (3)
        protected TodoRepository todoRepository;

        private static final long MAX_UNFINISHED_COUNT = 5;

        // (4)
        public Todo findOne(String todoId) {
            Todo todo = todoRepository.findOne(todoId);
            if (todo == null) {
                // (5)
                ResultMessages messages = ResultMessages.error();
                messages.add(ResultMessage
                        .fromText("[E404] The requested Todo is not found. (id="
                                + todoId + ")"));
                // (6)
                throw new ResourceNotFoundException(messages);
            }
            return todo;
        }

        @Override
        public Collection<Todo> findAll() {
            return todoRepository.findAll();
        }

        @Override
        public Todo create(Todo todo) {
            long unfinishedCount = todoRepository.countByFinished(false);
            if (unfinishedCount >= MAX_UNFINISHED_COUNT) {
                ResultMessages messages = ResultMessages.error();
                messages.add(ResultMessage
                        .fromText("[E001] The count of un-finished Todo must not be over "
                                + MAX_UNFINISHED_COUNT + "."));
                // (7)
                throw new BusinessException(messages);
            }

            // (8)
            String todoId = UUID.randomUUID().toString();
            Date createdAt = new Date();

            todo.setTodoId(todoId);
            todo.setCreatedAt(createdAt);
            todo.setFinished(false);

            todoRepository.save(todo);

            return todo;
        }

        @Override
        public Todo finish(String todoId) {
            Todo todo = findOne(todoId);
            if (todo.isFinished()) {
                ResultMessages messages = ResultMessages.error();
                messages.add(ResultMessage
                        .fromText("[E002] The requested Todo is already finished. (id="
                                + todoId + ")"));
                throw new BusinessException(messages);
            }
            todo.setFinished(true);
            todoRepository.save(todo);
            return todo;
        }

        @Override
        public void delete(String todoId) {
            Todo todo = findOne(todoId);
            todoRepository.delete(todo);
        }
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (1)
     - | To consider Service as component-scan target, add ``@Service``\  at class level.
   * - | (2)
     - | DB is not used in the current implementation, hence transaction management is not required, but when DB is to be used, ``@Transactional``\  should be added at class level. 
       | It is described in \ :ref:`tutorial-todo_infra`\ .
   * - | (3)
     - | Inject TodoRepository implementation using \ ``@Inject``\ .
   * - | (4)
     - | Logic fetch a single record is used in both, delete and finish method. Hence it should be implemented in a method (OK to make it public by declaring in the interface).
   * - | (5)
     - | Use ``org.terasoluna.gfw.common.message.ResultMessage`` provided in common library, as a class that stores result messages. 
       | Currently, for throwing error message, ResultMessage is added by specifying message type using ``ResultMessages.error()``.
   * - | (6)
     - | When target data is not found, ``org.terasoluna.gfw.common.exception.ResourceNotFoundException`` provided in common library is thrown.
   * - | (7)
     - | When business error occurs, ``org.terasoluna.gfw.common.exception.BusinessException`` provided in common library is thrown.
   * - | (8)
     - | UUID is used to generate a unique value. DB sequence may be used.
\
 .. note::
 
     In this chapter, error message is hard coded for simplification, but in reality it is not preferred from maintenance viewpoint.
     Usually, it is recommended to create message externally in property file. 
     The method for creating the message externally in property file is described in \ :doc:`../ArchitectureInDetail/PropertyManagement`\ .


.. figure:: ./images/image064.png
   :width: 40%

Creation of JUnit for Service
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
TBD

Creation of application layer
--------------------------------------------------------------------------------

Since domain layer implementation is completed, use the domain layer to create application layer.

Creation of Controller
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
First create TodoController that controls screen transition.
FQCN should be ``todo.app.todo.TodoController``.
It should be noted that the higher level package is different from the domain layer.

.. code-block:: java

    package todo.app.todo;

    import org.springframework.stereotype.Controller;
    import org.springframework.web.bind.annotation.RequestMapping;

    @Controller // (1)
    @RequestMapping("todo") // (2)
    public class TodoController {

    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (1)
     - | In order to make Controller as component-scan target, add ``@Controller`` at class level.
   * - | (2)
     - | In order to bring all screen transitions handled by TodoController, under ``<contextPath>/todo``, set ``@RequestMapping(“todo”)`` at class level.


.. figure:: ./images/image065.png
   :width: 40%


Show all TODO
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Following is performed on this screen.

* Display of new form
* Display of all records of TODO



Form creation
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
Form must contain title information. It should be implemented as JavaBean as shown below. FQCN should be ``todo.app.todo.TodoForm``.

.. code-block:: java

    package todo.app.todo;

    import java.io.Serializable;

    public class TodoForm implements Serializable {
        private static final long serialVersionUID = 1L;

        private String todoTitle;

        public String getTodoTitle() {
            return todoTitle;
        }

        public void setTodoTitle(String todoTitle) {
            this.todoTitle = todoTitle;
        }

    }


.. figure:: ./images/image066.png
   :width: 40%

Implementation of Controller
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
Implement ``setUpForm`` method and ``list`` method in TodoController.

.. code-block:: java
   :emphasize-lines: 18-32

    package todo.app.todo;
    import java.util.Collection;

    import javax.inject.Inject;

    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.RequestMapping;

    import todo.domain.model.Todo;
    import todo.domain.service.todo.TodoService;

    @Controller
    @RequestMapping("todo")
    public class TodoController {
        @Inject // (3)
        protected TodoService todoService;

        @ModelAttribute // (4)
        public TodoForm setUpForm() {
            TodoForm form = new TodoForm();
            return form;
        }

        @RequestMapping(value = "list") // (5)
        public String list(Model model) {
            Collection<Todo> todos = todoService.findAll();
            model.addAttribute("todos", todos); // (6
            return "todo/list"; // (7)
        }
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (3)
     - | Add ``@Inject`` annotation for injecting ``TodoService`` using DI container. Since instance  of type ``TodoService`` managed by DI container is injected, 
       | as a result, ``TodoServicelmpl`` instance is injected.
   * - | (4)
     - | Initialize Form. Adding ``@ModelAttribute`` annotation, form object of the return value of this method is added to Model with name ``todoForm``. 
       | It is same as executing model.addAttribute(“todoForm”, form) in each method of TodoController.
   * - | (5)
     - | Map list method to ``<contextPath>/todo/list``. Since @RequestMapping("todo") is being set at class level, only @RequestMapping(value = "list") is required to be set here.
   * - | (6)
     - | Add Todo list to Model and pass to View.
   * - | (7)
     - | If ``todo/list`` is returned as View name, ``WEB-INF/views/todo/list.jsp`` will be rendered using ``InternalResourceViewResolver`` defined in spring-mvc.xml.



JSP creation
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
Display Model passed from Controller in ``WEB-INF/views/todo/list.jsp``.
First, create buttons except "Finish"`, "Delete".

.. code-block:: jsp

    <!DOCTYPE html>
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Todo List</title>
    <style type="text/css">
    .strike {
        text-decoration: line-through;
    }
    </style>
    </head>
    <body>
        <h1>Todo List</h1>
        <div id="todoForm">
            <!-- (1) -->
            <form:form
               action="${pageContext.request.contextPath}/todo/create"
                method="post" modelAttribute="todoForm">
                <!-- (2) -->
                <form:input path="todoTitle" />
                <input type="submit" value="Create Todo" />
            </form:form>
        </div>
        <hr />
        <div id="todoList">
            <ul>
                <!-- (3) -->
                <c:forEach items="${todos}" var="todo">
                    <li><c:choose>
                            <c:when test="${todo.finished}"><!-- (4) -->
                                <span class="strike">
                                <!-- (5) -->
                                ${f:h(todo.todoTitle)}
                                </span>
                            </c:when>
                            <c:otherwise>
                                ${f:h(todo.todoTitle)}
                             </c:otherwise>
                        </c:choose></li>
                </c:forEach>
            </ul>
        </div>
    </body>
    </html>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (1)
     - | Display form object using <form:form> tag. Specify name of the form object added to Model by Controller in ``modelAttribute`` attribute. 
       | ``contextPath`` to be specified in ``action`` attribute can be fetched in ``${pageContext.request.contextPath}``
   * - | (2)
     - | Bind form property using <form:input> tag. Property name of form which is specified in ``modelAttribute`` should match with the value of ``path`` attribute.
   * - | (3)
     - |  Display entire list of Todo using ``<c:forEach>`` tag.
   * - | (4)
     - | Determine whether to decorate text using strikethrough(text-decoration: line-through;) to display if it is completed (finished).
   * - | (5)
     - | **To take XSS countermeasures at the time of output of character string, HTML escape should be performed using f:h() function.**
       | Regarding XSS measures, refer to \ :doc:`../Security/XSS`\ .


| Right click ``todo`` project in STS and start Web application by ``Run As`` → ``Run on Server``.
| If ``http://localhost:8080/todo/todo/list`` is accessed in browser, the following screen gets displayed.



.. figure:: ./images/image067.png
   :width: 40%


Create TODO
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Next, implement a new business logic after clicking  ``Create TODO`` button on List display screen.

Modifications in Controller
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
Add ``create`` method to TodoController.

.. code-block:: java
   :emphasize-lines: 8,29-31,46-70

    package todo.app.todo;

    import java.util.Collection;

    import javax.inject.Inject;
    import javax.validation.Valid;

    import org.dozer.Mapper;
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.validation.BindingResult;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.servlet.mvc.support.RedirectAttributes;
    import org.terasoluna.gfw.common.exception.BusinessException;
    import org.terasoluna.gfw.common.message.ResultMessage;
    import org.terasoluna.gfw.common.message.ResultMessages;

    import todo.domain.model.Todo;
    import todo.domain.service.todo.TodoService;

    @Controller
    @RequestMapping("todo")
    public class TodoController {
        @Inject
        protected TodoService todoService;

        // (8)
        @Inject
        protected Mapper beanMapper;

        @ModelAttribute
        public TodoForm setUpForm() {
            TodoForm form = new TodoForm();
            return form;
        }

        @RequestMapping(value = "list")
        public String list(Model model) {
            Collection<Todo> todos = todoService.findAll();
            model.addAttribute("todos", todos);
            return "todo/list";
        }

        @RequestMapping(value = "create", method = RequestMethod.POST) // (9)
        public String create(@Valid TodoForm todoForm, BindingResult bindingResult, // (10)
                Model model, RedirectAttributes attributes) { // (11)

            // (12)
            if (bindingResult.hasErrors()) {
                return list(model);
            }

            // (13)
            Todo todo = beanMapper.map(todoForm, Todo.class);

            try {
                todoService.create(todo);
            } catch (BusinessException e) {
                // (14)
                model.addAttribute(e.getResultMessages());
                return list(model);
            }

            // (15)
            attributes.addFlashAttribute(ResultMessages.success().add(
                    ResultMessage.fromText("Created successfully!")));
            return "redirect:/todo/list";
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (8)
     - | At the time of converting form object into domain object. Inject useful Mapper.
   * - | (9)
     - | Set \ ``@RequestMapping``\  such that HTTP method corresponds to POST with path ``/todo/create``.
   * - | (10)
     - | For performing input validation of form, add ``@Valid`` to form argument. Input validation result is stored in the immediate next argument ``BindingResult``.
   * - | (11)
     - | Return to list screen by redirecting after it is created normally. Add ``RedirectAttributes`` to argument for storing the information to be redirected.
   * - | (12)
     - | Return to list screen in case of input error. Re-execute ``list`` method as it is necessary to fetch all records of Todo again.
   * - | (13)
     - | Create Todo object from TodoForm using Mapper. No need to set if the property name of conversion source and destination is the same. 
       | There is no merit in using Mapper to convert only todoTitle property, but it is very convenient in case of multiple properties.
   * - | (14)
     - | Execute business logic and in case of ``BusinessException``, add the result message to Model and return to list screen.
   * - | (15)
     - | Since it is created normally, add the result message to flash scope and redirect to list screen. Since redirect is used, there is no case of browser being 
       | read again and a new registration process being launched. Since this time 'Created successfully' message is displayed, ResultMessages.success() is used.



Modifications in Form
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
To define input validation rules, add annotations in form class.

.. code-block:: java
   :emphasize-lines: 3-4,8-9

    package todo.app.todo;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;

    public class TodoForm {

        @NotNull // (1)
        @Size(min = 1, max = 30) // (2)
        private String todoTitle;

        public String getTodoTitle() {
            return todoTitle;
        }

        public void setTodoTitle(String todoTitle) {
            this.todoTitle = todoTitle;
        }
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (1)
     - | Since it is a mandatory item, add ``@NotNull``.
   * - | (2)
     - | Specify the range for ``@Size`` between 1 - 30 characters.


Modifications in JSP
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
Add the tag for dislaying the result message.

.. code-block:: jsp
   :emphasize-lines: 16,22

    <!DOCTYPE html>
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Todo List</title>
    <style type="text/css">
    .strike {
        text-decoration: line-through;
    }
    </style>
    </head>
    <body>
        <h1>Todo List</h1>
        <div id="todoForm">
            <!-- (6) -->
            <t:messagesPanel />

            <form:form
               action="${pageContext.request.contextPath}/todo/create"
                method="post" modelAttribute="todoForm">
                <form:input path="todoTitle" />
                <form:errors path="todoTitle" /><!-- (7) -->
                <input type="submit" value="Create Todo" />
            </form:form>
        </div>
        <hr />
        <div id="todoList">
            <ul>
                <c:forEach items="${todos}" var="todo">
                    <li><c:choose>
                            <c:when test="${todo.finished}">
                                <span style="text-decoration: line-through;">
                                ${f:h(todo.todoTitle)}
                                </span>
                            </c:when>
                            <c:otherwise>
                                ${f:h(todo.todoTitle)}
                             </c:otherwise>
                        </c:choose></li>
                </c:forEach>
            </ul>
        </div>
    </body>
    </html>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (6)
     - | Display result message using ``<t:messagesPanel>`` tag.
   * - | (7)
     - | Display errors in case of input error using ``<form:errors>`` tag. Match value of ``path`` attribute of ``<form:errors>`` with ``path`` attribute of ``<form:input>`` tag.


If form is submitted by entering appropriate value in the form, 'Created successfully' message is displayed as given below.


.. figure:: ./images/image068.png
   :width: 40%


.. figure:: ./images/image069.png
   :width: 40%



When 6 or more records are registered and business error occurs, error message is displayed.

.. figure:: ./images/image070.png
   :width: 40%


If form is submitted by entering null character, the following error message is displayed.


.. figure:: ./images/image071.png
   :width: 40%


Customize message display
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
``<t:messagesPanel>`` result is output by default as follows.

.. code-block:: html

    <div class="alert alert-success"><ul><li>Created successfully!</li></ul></div>



With the following modifications in style sheet (in <style> tag of ``list.jsp``), customize appearance of the result message.

.. code-block:: css

    .alert {
        border: 1px solid;
    }

    .alert-error {
        background-color: #c60f13;
        border-color: #970b0e;
        color: white;
    }

    .alert-success {
        background-color: #5da423;
        border-color: #457a1a;
        color: white;
    }


The message is as follows.



.. figure:: ./images/image072.png
   :width: 40%



.. figure:: ./images/image073.png
   :width: 40%


Moreover, input error message class can be specified to ``cssClass`` attribute of <form:errors> tag. Modify JSP as follows,

.. code-block:: html

    <form:errors path="todoTitle" cssClass="text-error" />


and add the following to style sheet.

.. code-block:: css

    .text-error {
        color: #c60f13;
    }


Input error is as follows.


.. figure:: ./images/image074.png
   :width: 40%


Finish TODO
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Add ``Finish`` button to List display screen. If the form is submited, then hidden todoId target will be sent and the corresponding Todo will be completed.


Modifications in JSP
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Add form in the JSP for completion of Todo.


.. code-block:: jsp
   :emphasize-lines: 56-67

    <!DOCTYPE html>
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Todo List</title>
    </head>
    <style type="text/css">
    .strike {
        text-decoration: line-through;
    }

    .alert {
        border: 1px solid;
    }

    .alert-error {
        background-color: #c60f13;
        border-color: #970b0e;
        color: white;
    }

    .alert-success {
        background-color: #5da423;
        border-color: #457a1a;
        color: white;
    }

    .text-error {
        color: #c60f13;
    }
    </style>
    <body>
        <h1>Todo List</h1>

        <div id="todoForm">
            <t:messagesPanel />

            <form:form
                action="${pageContext.request.contextPath}/todo/create"
                method="post" modelAttribute="todoForm">
                <form:input path="todoTitle" />
                <form:errors path="todoTitle" cssClass="text-error" />
                <input type="submit" value="Create Todo" />
            </form:form>
        </div>
        <hr />
        <div id="todoList">
            <ul>
                <c:forEach items="${todos}" var="todo">
                    <li><c:choose>
                            <c:when test="${todo.finished}">
                                <span class="strike">${f:h(todo.todoTitle)}</span>
                            </c:when>
                            <c:otherwise>
                                ${f:h(todo.todoTitle)}
                                <!-- (8) -->
                                <form:form
                                    action="${pageContext.request.contextPath}/todo/finish"
                                    method="post"
                                    modelAttribute="todoForm"
                                    cssStyle="display: inline-block;">
                                    <!-- (9) -->
                                    <form:hidden path="todoId"
                                        value="${f:h(todo.todoId)}" />
                                    <input type="submit" name="finish"
                                        value="Finish" />
                                </form:form>
                            </c:otherwise>
                        </c:choose></li>
                </c:forEach>
            </ul>
        </div>
    </body>
    </html>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (8)
     - | Display the form only if there are incomplete Todo. Send todoId by POST to ``<contextPath>/todo/finish``.
   * - | (9)
     - | Pass todoId using ``<form:hidden>`` tag. Also while setting the value in ``value`` attribute, HTML escaping should always be performed using  **f:h() function.**



Modifications in Form
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
Form for completion process flow uses TodoForm.
When todoId property needs to be added to TodoForm, input validation rules for new creation are applied as it is.
For specifying separate rules for new creation and completion in a single Form, set ``group`` attribute.


.. code-block:: java
   :emphasize-lines: 8-9,11-12,15-16,19,23-29

    package todo.app.todo;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;

    public class TodoForm {
        // (3)
        public static interface TodoCreate {
        };

        public static interface TodoFinish {
        };

        // (4)
        @NotNull(groups = { TodoFinish.class })
        private String todoId;

        // (5)
        @NotNull(groups = { TodoCreate.class })
        @Size(min = 1, max = 30, groups = { TodoCreate.class })
        private String todoTitle;

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

    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (3)
     - | Create the class which will be the group name for performing group validation. Since class may be blank, define the interface here.
       | Refer to \ :doc:`../ArchitectureInDetail/Validation`\  for group validation.
   * - | (4)
     - | todoId is mandatory for completion process, hence add @NotNull. It is the rule required only at the time of completion, set TodoFinish.class in group attribute.
   * - | (5)
     - | Rule for new creation is not required for completion process, hence set TodoCreate.class in group attribute of respective ``@NotNull`` and ``@Size``.

Modifications in Controller
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Add completion processing logic to TodoController.
Take precaution of using **@Validated instead of @Valid** for executing the group validation.

.. code-block:: java
   :emphasize-lines: 6,12,50,72-94

    package todo.app.todo;

    import java.util.Collection;

    import javax.inject.Inject;
    import javax.validation.groups.Default;

    import org.dozer.Mapper;
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.validation.BindingResult;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.servlet.mvc.support.RedirectAttributes;
    import org.terasoluna.gfw.common.exception.BusinessException;
    import org.terasoluna.gfw.common.message.ResultMessage;
    import org.terasoluna.gfw.common.message.ResultMessages;

    import todo.app.todo.TodoForm.TodoCreate;
    import todo.app.todo.TodoForm.TodoFinish;
    import todo.domain.model.Todo;
    import todo.domain.service.todo.TodoService;

    @Controller
    @RequestMapping("todo")
    public class TodoController {
        @Inject
        protected TodoService todoService;

        @Inject
        protected Mapper beanMapper;

        @ModelAttribute
        public TodoForm setUpForm() {
            TodoForm form = new TodoForm();
            return form;
        }

        @RequestMapping(value = "list")
        public String list(Model model) {
            Collection<Todo> todos = todoService.findAll();
            model.addAttribute("todos", todos);
            return "todo/list";
        }

        @RequestMapping(value = "create", method = RequestMethod.POST)
        public String create(
                @Validated({ Default.class, TodoCreate.class }) TodoForm todoForm, // (16)
                BindingResult bindingResult, Model model,
                RedirectAttributes attributes) {

            if (bindingResult.hasErrors()) {
                return list(model);
            }

            Todo todo = beanMapper.map(todoForm, Todo.class);

            try {
                todoService.create(todo);
            } catch (BusinessException e) {
                model.addAttribute(e.getResultMessages());
                return list(model);
            }

            attributes.addFlashAttribute(ResultMessages.success().add(
                    ResultMessage.fromText("Created successfully!")));
            return "redirect:/todo/list";
        }

        @RequestMapping(value = "finish", method = RequestMethod.POST) // (17)
        public String finish(
                @Validated({ Default.class, TodoFinish.class }) TodoForm form, // (18)
                BindingResult bindingResult, Model model,
                RedirectAttributes attributes) {
            // (19)
            if (bindingResult.hasErrors()) {
                return list(model);
            }

            try {
                todoService.finish(form.getTodoId());
            } catch (BusinessException e) {
                // (20)
                model.addAttribute(e.getResultMessages());
                return list(model);
            }

            // (21)
            attributes.addFlashAttribute(ResultMessages.success().add(
                    ResultMessage.fromText("Finished successfully!")));
            return "redirect:/todo/list";
        }
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (16)
     - | Change @Valid to @Validated for executing group validation. Multiple group classes can be specified in value. 
       | ``Default.class`` is the group when group is not specified in validation rules. At the time of using ``@Validated``, ``Default.class`` can also be specified.
   * - | (17)
     - | Set ``@RequestMapping`` to ``/todo/finish`` and HTTP method to POST.
   * - | (18)
     - | Specify TodoFinish.class as the group for Finish.
   * - | (19)
     - | In case of input error, return to list screen.
   * - | (20)
     - | Execute business logic, add result message to Model and return to list screen in case when ``BusinessException`` occurs.
   * - | (21)
     - | Since it is created normally, add result message to flash scope and redirect to list screen.

.. note::

    Separate Form can also be created for Create and Finish. In that case, only the required parameters will be the properties of Form.
    However, as the number of classes increase, duplicate properties also increase, and when the specifications change, the modification cost will also be more.
    Moreover, if multiple Form objects in the same Controller are initialized by ``@ModelAttribute`` method,
    unnecessary instance gets generated because every time all Forms are being initialized. Therefore,
    it is recommended to basically consolidate the Form as much as possible to be used in a single Controller and carry out the group validation settings.


After creating new Todo, if submit is performed by Finish button, then strike-through is shown as below and it can be understood that the operation is completed.


.. figure:: ./images/image075.png
   :width: 40%


.. figure:: ./images/image076.png
   :width: 40%


Delete TODO
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Add ``Delete`` button to list display screen. If the form is submitted, the hidden todoId target will be sent and corresponding Todo will be deleted.

Modifications in JSP
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
Add form to the JSP for deletion business logic.


.. code-block:: jsp
   :emphasize-lines: 68-77

    <!DOCTYPE html>
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Todo List</title>
    </head>
    <style type="text/css">
    .strike {
        text-decoration: line-through;
    }

    .alert {
        border: 1px solid;
    }

    .alert-error {
        background-color: #c60f13;
        border-color: #970b0e;
        color: white;
    }

    .alert-success {
        background-color: #5da423;
        border-color: #457a1a;
        color: white;
    }

    .text-error {
        color: #c60f13;
    }
    </style>
    <body>
        <h1>Todo List</h1>

        <div id="todoForm">
            <t:messagesPanel />

            <form:form
                action="${pageContext.request.contextPath}/todo/create"
                method="post" modelAttribute="todoForm">
                <form:input path="todoTitle" />
                <form:errors path="todoTitle" cssClass="text-error" />
                <input type="submit" value="Create Todo" />
            </form:form>
        </div>
        <hr />
        <div id="todoList">
            <ul>
                <c:forEach items="${todos}" var="todo">
                    <li><c:choose>
                            <c:when test="${todo.finished}">
                                <span class="strike">${f:h(todo.todoTitle)}</span>
                            </c:when>
                            <c:otherwise>
                                ${f:h(todo.todoTitle)}
                                <form:form
                                    action="${pageContext.request.contextPath}/todo/finish"
                                    method="post"
                                    modelAttribute="todoForm"
                                    cssStyle="display: inline-block;">
                                    <form:hidden path="todoId"
                                        value="${f:h(todo.todoId)}" />
                                    <input type="submit" name="finish"
                                        value="Finish" />
                                </form:form>
                            </c:otherwise>
                        </c:choose>
                        <!-- (10) -->
                        <form:form
                            action="${pageContext.request.contextPath}/todo/delete"
                            method="post" modelAttribute="todoForm"
                            cssStyle="display: inline-block;">
                            <!-- (11) -->
                            <form:hidden path="todoId"
                                value="${f:h(todo.todoId)}" />
                            <input type="submit" value="Delete" />
                        </form:form>
                    </li>
                </c:forEach>
            </ul>
        </div>
    </body>
    </html>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (10)
     - | Display form in the JSP for deletion request. Send todoId by POST to ``<contextPath>/todo/delete``.
   * - | (11)
     - | Pass todoId using <form:hidden> tag. Also while setting the value in ``value`` attribute, HTML escaping should always be performed using **f:h() function.**



Modifications in Form
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Add group for Delete to TodoForm. Rules are almost same as for Finish.


.. code-block:: java
   :emphasize-lines: 14-15,18

    package todo.app.todo;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;

    public class TodoForm {
        public static interface TodoCreate {
        };

        public static interface TodoFinish {
        };

        // (6)
        public static interface TodoDelete {
        }

        // (7)
        @NotNull(groups = { TodoFinish.class, TodoDelete.class })
        private String todoId;

        @NotNull(groups = { TodoCreate.class })
        @Size(min = 1, max = 30, groups = { TodoCreate.class })
        private String todoTitle;

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

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (6)
     - | Define group TodoDelete for Delete.
   * - | (7)
     - | Set so that validation of TodoDelete group will be carried out for todoId property.


Modifications in Controller
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Add the logic for delete processing to TodoController. It is almost same as the completion process.

.. code-block:: java
   :emphasize-lines: 94-114

    package todo.app.todo;

    import java.util.Collection;

    import javax.inject.Inject;
    import javax.validation.groups.Default;

    import org.dozer.Mapper;
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.validation.BindingResult;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.servlet.mvc.support.RedirectAttributes;
    import org.terasoluna.gfw.common.exception.BusinessException;
    import org.terasoluna.gfw.common.message.ResultMessage;
    import org.terasoluna.gfw.common.message.ResultMessages;

    import todo.app.todo.TodoForm.TodoDelete;
    import todo.app.todo.TodoForm.TodoCreate;
    import todo.app.todo.TodoForm.TodoFinish;
    import todo.domain.model.Todo;
    import todo.domain.service.todo.TodoService;

    @Controller
    @RequestMapping("todo")
    public class TodoController {
        @Inject
        protected TodoService todoService;

        @Inject
        protected Mapper beanMapper;

        @ModelAttribute
        public TodoForm setUpForm() {
            TodoForm form = new TodoForm();
            return form;
        }

        @RequestMapping(value = "list")
        public String list(Model model) {
            Collection<Todo> todos = todoService.findAll();
            model.addAttribute("todos", todos);
            return "todo/list";
        }

        @RequestMapping(value = "create", method = RequestMethod.POST)
        public String create(
                @Validated({ Default.class, TodoCreate.class }) TodoForm todoForm,
                BindingResult bindingResult, Model model,
                RedirectAttributes attributes) {

            if (bindingResult.hasErrors()) {
                return list(model);
            }

            Todo todo = beanMapper.map(todoForm, Todo.class);

            try {
                todoService.create(todo);
            } catch (BusinessException e) {
                model.addAttribute(e.getResultMessages());
                return list(model);
            }

            attributes.addFlashAttribute(ResultMessages.success().add(
                    ResultMessage.fromText("Created successfully!")));
            return "redirect:/todo/list";
        }

        @RequestMapping(value = "finish", method = RequestMethod.POST)
        public String finish(
                @Validated({ Default.class, TodoFinish.class }) TodoForm form,
                BindingResult bindingResult, Model model,
                RedirectAttributes attributes) {
            if (bindingResult.hasErrors()) {
                return list(model);
            }

            try {
                todoService.finish(form.getTodoId());
            } catch (BusinessException e) {
                model.addAttribute(e.getResultMessages());
                return list(model);
            }

            attributes.addFlashAttribute(ResultMessages.success().add(
                    ResultMessage.fromText("Finished successfully!")));
            return "redirect:/todo/list";
        }

        @RequestMapping(value = "delete", method = RequestMethod.POST)
        public String delete(
                @Validated({ Default.class, TodoDelete.class }) TodoForm form,
                BindingResult bindingResult, Model model,
                RedirectAttributes attributes) {

            if (bindingResult.hasErrors()) {
                return list(model);
            }

            try {
                todoService.delete(form.getTodoId());
            } catch (BusinessException e) {
                model.addAttribute(e.getResultMessages());
                return list(model);
            }

            attributes.addFlashAttribute(ResultMessages.success().add(
                    ResultMessage.fromText("Deleted successfully!")));
            return "redirect:/todo/list";
        }

    }


If submit is performed for Todo using ``Delete`` button, the following target TODO gets deleted.


.. figure:: ./images/image077.png
   :width: 40%


.. figure:: ./images/image078.png
   :width: 40%

|

.. _tutorial-todo_infra:

Change of infrastructure layer
================================================================================

Till the last section, infrastructure layer is implemented using memory, but in this chapter, it is implemented using DB.
O/R Mapper is used for accessing DB, here 2 methods are described: one  using Spring Data JPA and second using TERASOLUNA DAO.



Common settings
--------------------------------------------------------------------------------

First, apply settings common to the both, Spring Data JPA version and TERASOLUNA Dao version.
Currently, H2Database is used for reducing the effort of setting up a DB.


Modifications in pom.xml
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Define dependency for using H2Database in pom.xml.


.. code-block:: xml

    <dependency>
    	<groupId>com.h2database</groupId>
	<artifactId>h2</artifactId>
        <version>1.3.172</version>
        <scope>compile</scope>
    </dependency>

.. warning::

  The above settings are \ **for easy trial of the application**\ and is not to be used in actual application development. These settings must be deleted in actual project.
  
  further,  \ ``<scope>``\  of JDBC driver must be \ ``provided``\ .


Definition of data source
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Modifications in todo-infra.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Since data source definition is related to infrastructure layer, it should be defined in todo-infra.xml, but
it is recommended to define the information depending on the environment like user name, password etc. of database in a separate Bean definition file (todo-env.xml).


Here, only import todo-env.xml.

 .. code-block:: xml
 
     <?xml version="1.0" encoding="UTF-8"?>
     <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
 
         <import resource="classpath:/META-INF/spring/todo-env.xml" />
     </beans>


 .. note::

    By saving xxx-env.xml in another file and replacing only this file with build tools like Maven, configurations values that differs with each environment 
    (development environment, test environment etc.) can be managed. Also the configuration file, in which data source of only specific environment is fetched from JNDI, 
    can be managed.



Creation of todo-env.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Create ``src/main/resources/META-INF/spring/todo-env.xml`` and perform the following settings. Include the environment dependent settings (here, DataSource) in this file.

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

        <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
            destroy-method="close">
            <property name="driverClassName" value="${database.driverClassName}" />
            <property name="url" value="${database.url}" />
            <property name="username" value="${database.username}" />
            <property name="password" value="${database.password}" />
            <property name="defaultAutoCommit" value="false" />
            <property name="maxActive" value="${cp.maxActive}" />
            <property name="maxIdle" value="${cp.maxIdle}" />
            <property name="minIdle" value="${cp.minIdle}" />
            <property name="maxWait" value="${cp.maxWait}" />
        </bean>
    </beans>


For improving maintainability define the properties in external property file.
\

.. note::

   DataSource should be fetched using JNDI depending on the environment (Application Server).
   In that case, define  ``<jee:jndi-lookup id="dataSource" jndi-name="JNDI name" />``
   env file is created in such a way that, switch-over becomes possible at the time of build, between commons-dbcp in development environment and JNDI in test environment.


todo-infra.properties
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Define property value related to infrastructure layer in ``src/main/resources/META-INF/spring/todo-infra.properties``.


.. code-block:: properties

    database=H2
    ## (1)
    database.url=jdbc:h2:mem:todo;DB_CLOSE_DELAY=-1;INIT=create table if not exists todo(todo_id varchar(36) primary key, todo_title varchar(30), finished boolean, created_at timestamp)
    database.username=sa
    database.password=
    database.driverClassName=org.h2.Driver
    # connection pool
    ## (2)
    cp.maxActive=96
    cp.maxIdle=16
    cp.minIdle=0
    cp.maxWait=60000


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (1)
     - | Perform settings related to database. Set H2 URL and driver. For simplification, in-memory DB is used here and settings are made such that initialization DDL is executed whenever AP server starts.
   * - | (2)
     - | Perform settings related to connection pool. Here sample values are being set. Take note that the actual value differ according to server performance.


Modifications in todo-domain.xml
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
In order to enable transaction management using @Transactional annotation, set ``<tx:annotation-driven>`` tag.

.. code-block:: xml
   :emphasize-lines: 5,7,11

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xmlns:tx="http://www.springframework.org/schema/tx"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
        <context:component-scan base-package="todo.domain" />
        <import resource="classpath:META-INF/spring/todo-infra.xml"/>
        <tx:annotation-driven/>
    </beans>


Modifications in TodoServiceImpl
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: java
   :emphasize-lines: 10,20,40

    package todo.domain.service.todo;

    import java.util.Collection;
    import java.util.Date;
    import java.util.UUID;

    import javax.inject.Inject;

    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;
    import org.terasoluna.gfw.common.exception.BusinessException;
    import org.terasoluna.gfw.common.exception.ResourceNotFoundException;
    import org.terasoluna.gfw.common.message.ResultMessage;
    import org.terasoluna.gfw.common.message.ResultMessages;

    import todo.domain.model.Todo;
    import todo.domain.repository.todo.TodoRepository;

    @Service
    @Transactional // (9)
    public class TodoServiceImpl implements TodoService {
        @Inject
        protected TodoRepository todoRepository;

        private static final long MAX_UNFINISHED_COUNT = 5;

        public Todo findOne(String todoId) {
            Todo todo = todoRepository.findOne(todoId);
            if (todo == null) {
                ResultMessages messages = ResultMessages.error();
                messages.add(ResultMessage
                        .fromText("[E404] The requested Todo is not found. (id="
                                + todoId + ")"));
                throw new ResourceNotFoundException(messages);
            }
            return todo;
        }

        @Override
        @Transactional(readOnly = true) // (10)
        public Collection<Todo> findAll() {
            return todoRepository.findAll();
        }

        @Override
        public Todo create(Todo todo) {
            long unfinishedCount = todoRepository.countByFinished(false);
            if (unfinishedCount >= MAX_UNFINISHED_COUNT) {
                ResultMessages messages = ResultMessages.error();
                messages.add(ResultMessage
                        .fromText("[E001] The count of un-finished Todo must not be over "
                                + MAX_UNFINISHED_COUNT + "."));

                throw new BusinessException(messages);
            }

            String todoId = UUID.randomUUID().toString();
            Date createdAt = new Date();

            todo.setTodoId(todoId);
            todo.setCreatedAt(createdAt);
            todo.setFinished(false);

            todoRepository.save(todo);

            return todo;
        }

        @Override
        public Todo finish(String todoId) {
            Todo todo = findOne(todoId);
            if (todo.isFinished()) {
                ResultMessages messages = ResultMessages.error();
                messages.add(ResultMessage
                        .fromText("[E002] The requested Todo is already finished. (id="
                                + todoId + ")"));
                throw new BusinessException(messages);
            }
            todo.setFinished(true);
            todoRepository.save(todo);
            return todo;
        }

        @Override
        public void delete(String todoId) {
            Todo todo = findOne(todoId);
            todoRepository.delete(todo);
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (9)
     - | Add ``@Transactional`` at class level and manage all public methods as transactions. This will enable to start transaction while starting the method and 
       | to commit when the method ends normally. When unchecked exception occurs in between, transaction is rolled-back.
   * - | (10)
     - | Add ``readOnly=true`` for the methods which only read data from the db. Depending on O/R Mapper, optimization is done at the time of reference queries 
       | by using this setting (not effective while using JPA).


Use Spring Data JPA
--------------------------------------------------------------------------------

In this section, configuration is explained when  `Spring Data JPA <http://www.springsource.org/spring-data/jpa>`_  is to be used in infrastructure layer.
Proceed to \ :ref:`using_terasolunaDao`\  by skipping this section when TERASOLUNA DAO is to be used.


Modifications in configuration file for using Spring Data JPA
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Modifications in pom.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Add the following to pom.xml for adding Spring Data JPA dependent library.

.. code-block:: xml

    <dependency>
        <groupId>org.terasoluna.gfw</groupId>
        <artifactId>terasoluna-gfw-jpa</artifactId>
    </dependency>


Modifications in todo-infra.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
This setting in todo-infra.xml is for using JPA and Spring Data JPA . Define EntityManagerFactory of JPA here.

.. code-block:: xml
   :emphasize-lines: 4-5,7-8,12-44

    <?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xmlns:jpa="http://www.springframework.org/schema/data/jpa"
    	xmlns:util="http://www.springframework.org/schema/util"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        	http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd
        	http://www.springframework.org/schema/data/jpa http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">

        <import resource="classpath:/META-INF/spring/todo-env.xml" />

        <!-- (1) -->
        <jpa:repositories base-package="todo.domain.repository"></jpa:repositories>

        <!-- (2) -->
        <bean id="jpaVendorAdapter"
            class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
            <property name="showSql" value="false" />
            <property name="database" value="${database}" />
        </bean>

        <!-- (3) -->
        <bean
            class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean"
            id="entityManagerFactory">
            <!-- (4) -->
            <property name="packagesToScan" value="todo.domain.model" />
            <property name="dataSource" ref="dataSource" />
            <property name="jpaVendorAdapter" ref="jpaVendorAdapter" />
            <!-- (5) -->
            <property name="jpaPropertyMap">
                <util:map>
                    <entry key="hibernate.hbm2ddl.auto" value="none" />
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

    </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (1)
     - | If Spring Data JPA is used, the class that implements ``Repository`` interface gets generated automatically. 
       | Specify the package that includes target repositories in ``base-package`` attribute of ``<jpa:repository>`` tag.
   * - | (2)
     - | Set JPA implementation vendor. Define HibernateJpaVendorAdapter for using Hibernate for JPA implementation.
   * - | (3)
     - | Define EntityManager.
   * - | (4)
     - | Specify package name of entity.
   * - | (5)
     - | Perform detailed settings related to Hibernate.


Modifications in todo-env.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Add Bean definition related to transaction manager.

.. code-block:: xml
   :emphasize-lines: 20-24

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

        <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
            destroy-method="close">
            <property name="driverClassName" value="${database.driverClassName}" />
            <property name="url" value="${database.url}" />
            <property name="username" value="${database.username}" />
            <property name="password" value="${database.password}" />
            <property name="defaultAutoCommit" value="false" />
            <property name="maxActive" value="${cp.maxActive}" />
            <property name="maxIdle" value="${cp.maxIdle}" />
            <property name="minIdle" value="${cp.minIdle}" />
            <property name="maxWait" value="${cp.maxWait}" />
        </bean>

        <!-- (6) -->
        <bean class="org.springframework.orm.jpa.JpaTransactionManager"
            id="transactionManager">
            <property name="entityManagerFactory" ref="entityManagerFactory" />
        </bean>


    </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (1)
     - | Set transaction manager. ``id`` should set to ``transactionManager``. 
       | When another name is to be specified, transaction manager name should be specified in <tx:annotation-driven> tag in todo-domain.xml and <jpa:repository> tag in todo-infra.xml.
\
 .. note::
    Transaction manager should use JtaTransactionManager in case of JavaEE container. In this case, define transaction manager using <tx:jta-transaction-manager />.
    
    These settings can be defined in  todo-infra.xml for the project (while using Tomcat) which does not change with environment.


Modifications in spring-mvc.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Add ``OpenEntityManagerInViewFilter`` to spring-mvc.xml. Start and end of EntityManager lifecycle is carried out using Interceptor.
By adding this configuration, Lazy Load support gets enabled at application layer (Controller or View class). 

.. code-block:: xml

        <mvc:interceptors>

        <!-- ... -->

        <!-- (6) -->
        <mvc:interceptor>
            <mvc:mapping path="/**" />
            <mvc:exclude-mapping path="/resources/**" />
            <bean
                class="org.springframework.orm.jpa.support.OpenEntityManagerInViewInterceptor" />
        </mvc:interceptor>

    </mvc:interceptors>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - No.
     - Details
   * - | (6)
     - | During the access (\ ``/resources/**``\ ) to static resources (css, js, image etc), database access is not going to happen for sure. Hence, intercept has been exempted.


Modifications in logback.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
.. code-block:: xml
   :emphasize-lines: 32-45

    <!DOCTYPE logback>
    <configuration>
        <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern><![CDATA[%d{yyyy-MM-dd HH:mm:ss} [%thread] [%-5level] [%-48logger{48}] - %msg%n]]></pattern>
            </encoder>
        </appender>

        <!-- Application Loggers -->
        <logger name="todo">
            <level value="debug" />
        </logger>

        <!-- TERASOLUNA -->
        <logger name="org.terasoluna.gfw">
            <level value="info" />
        </logger>

        <logger name="org.terasoluna.gfw.web.logging.TraceLoggingInterceptor">
            <level value="trace" />
        </logger>

        <!-- 3rdparty Loggers -->
        <logger name="org.springframework">
            <level value="warn" />
        </logger>

        <logger name="org.springframework.web.servlet">
            <level value="info" />
        </logger>

        <!-- (8) -->
        <logger name="org.hibernate.SQL">
            <level value="debug" />
        </logger>

        <!-- (9) -->
        <logger name="org.hibernate.type.descriptor.sql.BasicBinder">
            <level value="trace" />
        </logger>

        <!-- (10) -->
        <logger name="org.hibernate.engine.transaction">
            <level value="debug" />
        </logger>

        <root level="WARN">
            <appender-ref ref="STDOUT" />
        </root>
    </configuration>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (8)
     - | This setting is to output SQL log using Hibernate.
   * - | (9)
     - | This setting is to output SQL bind variable using Hibernate.
   * - | (10)
     - | This setting is to output transaction log using Hibernate.


Settings of Entity
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

JPA annotation must be used to map Todo class with database.

.. code-block:: java
   :emphasize-lines: 6-11,13-15,19-22,25,28,31,33

    package todo.domain.model;

    import java.io.Serializable;
    import java.util.Date;

    import javax.persistence.Column;
    import javax.persistence.Entity;
    import javax.persistence.Id;
    import javax.persistence.Table;
    import javax.persistence.Temporal;
    import javax.persistence.TemporalType;

    // (1)
    @Entity
    @Table(name = "todo")
    public class Todo implements Serializable {
        private static final long serialVersionUID = 1L;

        // (2)
        @Id
        // (3)
        @Column(name = "todo_id")
        private String todoId;

        @Column(name = "todo_title")
        private String todoTitle;

        @Column(name = "finished")
        private boolean finished;

        @Column(name = "created_at")
        // (4)
        @Temporal(TemporalType.TIMESTAMP)
        private Date createdAt;

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
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (1)
     - | Add ``@Entity`` showing that it is JPA entity and set the corresponding table name using ``@Table``.
   * - | (2)
     - | Add ``@Id`` to the field corresponding to primary key column.
   * - | (3)
     - | Set the corresponding column name by ``@Column``.
   * - | (4)
     - | It has to be clearly specified that ``Date`` type corresponds to which of ``java.sql.Date``, ``java.sql.Time``, ``java.sql.Timestamp``. Here ``Timestamp`` is specified.


Modifications in TodoRepository
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: java
   :emphasize-lines: 3-5,10-12

    package todo.domain.repository.todo;

    import org.springframework.data.jpa.repository.JpaRepository;
    import org.springframework.data.jpa.repository.Query;
    import org.springframework.data.repository.query.Param;

    import todo.domain.model.Todo;

    // (1)
    public interface TodoRepository extends JpaRepository<Todo, String> {
        @Query(value = "SELECT COUNT(x) FROM Todo x WHERE x.finished = :finished") // (2)
        long countByFinished(@Param("finished") boolean finished); // (3)
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (1)
     - | Set interface that extends JpaRepository. Specify Entity class (Todo) and primary key type (String) sequentially in Generics parameter. 
       | Since basic CRUD operations (findOne, findAll, save, delete etc.) are defined in upper level interface, only ``countByFinished`` can be defined in ``TodoRepository``.
   * - | (2)
     - | Specify JPQL executed at the time of calling ``countByFinished`` by @Query.
   * - | (3)
     - | Set bind variable of JPQL specified in (2) by ``@Param``. 
       | Here, add ``@Param("finished")`` to method argument for inserting \ ```:finished"``\  in JPQL.


Modifications in TodoRepositoryImpl
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
When Spring Data JPA is used, RepositoryImpl gets generated automatically from interface. Hence, delete the unnecessary TodoRepositoryImpl.


Usage of Spring Data JPA is completed here. 
Start AP server to output SQL log and transaction log as follows for Todo display and new creation.


.. _using_terasolunaDao:

Use TERASOLUNA DAO
--------------------------------------------------------------------------------
Configuration method using TERASOLUNA DAO in infrastructure layer, is explained in this section.


.. note::
   TERASOLUNA DAO is the library providing a simple SQL mapper which is extended from ``org.springframework.orm.ibatis.support.SqlMapClientDaoSupport`` which is a linkage class of MyBatis2.3.5 and Spring.
   DAO having the following 4 interfaces is provided.
   
      #. jp.terasoluna.fw.dao.QueryDAO
      #. jp.terasoluna.fw.dao.UpdateDAO
      #. jp.terasoluna.fw.dao.StoredProcedureDAO
      #. jp.terasoluna.fw.dao.QueryRowHandleDAO
      
   jp.terasoluna.fw.dao.ibatis.XxxDAOiBatisImpl is implemented for each interface.

Setting for using the TERASOLUNA DAO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Modifications in pom.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Add the following in pom.xml in order to add dependent library related to TERASOLUNA DAO.

.. code-block:: xml

    <dependency>
        <groupId>org.terasoluna.gfw</groupId>
        <artifactId>terasoluna-gfw-mybatis2</artifactId>
    </dependency>


Modifications in todo-infra.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
The following setting is to be done in todo-infra.xml to use TERASOLUNA DAO.

.. code-block:: xml
   :emphasize-lines: 8-37

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

        <import resource="classpath:/META-INF/spring/todo-env.xml" />

        <!-- (1) -->
        <bean id="sqlMapClient"
            class="org.springframework.orm.ibatis.SqlMapClientFactoryBean">
            <!-- (2) -->
            <property name="configLocations"
                value="classpath*:/META-INF/mybatis/config/*sqlMapConfig.xml" />
            <!-- (3) -->
            <property name="mappingLocations"
                value="classpath*:/META-INF/mybatis/sql/**/*-sqlmap.xml" />
            <property name="dataSource" ref="dataSource" />
        </bean>

        <!-- (4) -->
        <bean id="queryDAO" class="jp.terasoluna.fw.dao.ibatis.QueryDAOiBatisImpl">
            <property name="sqlMapClient" ref="sqlMapClient" />
        </bean>

        <bean id="updateDAO" class="jp.terasoluna.fw.dao.ibatis.UpdateDAOiBatisImpl">
            <property name="sqlMapClient" ref="sqlMapClient" />
        </bean>

        <bean id="spDAO"
            class="jp.terasoluna.fw.dao.ibatis.StoredProcedureDAOiBatisImpl">
            <property name="sqlMapClient" ref="sqlMapClient" />
        </bean>

        <bean id="queryRowHandleDAO"
            class="jp.terasoluna.fw.dao.ibatis.QueryRowHandleDAOiBatisImpl">
            <property name="sqlMapClient" ref="sqlMapClient" />
        </bean>
    </beans>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (1)
     - | Define ``SqlMapClient``.
   * - | (2)
     - | Set the path of SqlMap configuration file. Read *sqlMapConfig.xml under ``META-INF/mybatis/config``.
   * - | (3)
     - | Set the path of SqlMap file. Read *-sqlmap.xml of any folder under ``META-INF/mybatis/sql``.
   * - | (4)
     - | Define TERASOLUNA DAO.


Modifications in todo-env.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Add definition of transaction manager. 
Define transaction manager in environment dependent configuration file since JtaTransactionManager can also be used in JavaEE container.

.. code-block:: xml
   :emphasize-lines: 19-23

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

        <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
            destroy-method="close">
            <property name="driverClassName" value="${database.driverClassName}" />
            <property name="url" value="${database.url}" />
            <property name="username" value="${database.username}" />
            <property name="password" value="${database.password}" />
            <property name="defaultAutoCommit" value="false" />
            <property name="maxActive" value="${cp.maxActive}" />
            <property name="maxIdle" value="${cp.maxIdle}" />
            <property name="minIdle" value="${cp.minIdle}" />
            <property name="maxWait" value="${cp.maxWait}" />
        </bean>

        <!-- (1) -->
        <bean id="transactionManager"
            class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <property name="dataSource" ref="dataSource" />
        </bean>

    </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (1)
     - | Set transaction manager. ``id`` should be set to ``transactionManager``. 
       | If a separate name is to be specified, transaction manager name should be specified even in <tx:annotation-driven> tag.



Modifications in logback.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. code-block:: xml
   :emphasize-lines: 32-43

    <!DOCTYPE logback>
    <configuration>
        <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern><![CDATA[%d{yyyy-MM-dd HH:mm:ss} [%thread] [%-5level] [%-48logger{48}] - %msg%n]]></pattern>
            </encoder>
        </appender>

        <!-- Application Loggers -->
        <logger name="todo">
            <level value="debug" />
        </logger>

        <!-- TERASOLUNA -->
        <logger name="org.terasoluna.gfw">
            <level value="info" />
        </logger>

        <logger name="org.terasoluna.gfw.web.logging.TraceLoggingInterceptor">
            <level value="trace" />
        </logger>

        <!-- 3rdparty Loggers -->
        <logger name="org.springframework">
            <level value="warn" />
        </logger>

        <logger name="org.springframework.web.servlet">
            <level value="info" />
        </logger>

        <!-- (8) -->
        <logger name="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <level value="debug" />
        </logger>

        <!-- (9) -->
        <logger name="java.sql.Connection">
            <level value="trace" />
        </logger>
        <logger name="java.sql.PreparedStatement">
            <level value="debug" />
        </logger>

        <root level="WARN">
            <appender-ref ref="STDOUT" />
        </root>
    </configuration>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (8)
     - | Set to output transaction log for ``DataSourceTransactionManager``.
   * - | (9)
     - | Set to output SQL log.


Creation of sqlMapConfig
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Create ``src/main/resources/META-INF/mybatis/config/sqlMapConfig.xml`` as given below.

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE sqlMapConfig
                PUBLIC "-//ibatis.apache.org//DTD SQL Map Config 2.0//EN"
                "http://ibatis.apache.org/dtd/sql-map-config-2.dtd">
    <sqlMapConfig>
        <!-- (1) -->
        <settings useStatementNamespaces="true" />
    </sqlMapConfig>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (1)
     - | This setting is to give namespace to SQLID.


Modifications in RepositoryImpl
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: java
   :emphasize-lines: 5,7,8,11,16,17,19-21,23-24,28,30,34,36,41-47,50,52,63,65

    package todo.domain.repository.todo;

    import java.util.Collection;

    import javax.inject.Inject;

    import jp.terasoluna.fw.dao.QueryDAO;
    import jp.terasoluna.fw.dao.UpdateDAO;

    import org.springframework.stereotype.Repository;
    import org.springframework.transaction.annotation.Transactional;

    import todo.domain.model.Todo;

    @Repository
    // (1)
    @Transactional
    public class TodoRepositoryImpl implements TodoRepository {
        // (2)
        @Inject
        protected QueryDAO queryDAO;

        @Inject
        protected UpdateDAO updateDAO;

        // (3)
        @Override
        @Transactional(readOnly = true)
        public Todo findOne(String todoId) {
            return queryDAO.executeForObject("todo.findOne", todoId, Todo.class);
        }

        @Override
        @Transactional(readOnly = true)
        public Collection<Todo> findAll() {
            return queryDAO.executeForObjectList("todo.findAll", null);
        }

        @Override
        public Todo save(Todo todo) {
            // (4)
            if (exists(todo.getTodoId())) {
                updateDAO.execute("todo.update", todo);
            } else {
                updateDAO.execute("todo.create", todo);
            }
            return todo;
        }

        @Transactional(readOnly = true)
        public boolean exists(String todoId) {
            long count = queryDAO.executeForObject("todo.exists", todoId,
                    Long.class);
            return count > 0;
        }

        @Override
        public void delete(Todo todo) {
            updateDAO.execute("todo.delete", todo);
        }

        @Override
        @Transactional(readOnly = true)
        public long countByFinished(boolean finished) {
            return queryDAO.executeForObject("todo.countByFinished", finished,
                    Long.class);
        }
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr.No.
     - Description
   * - | (1)
     - | Manage all the transactions of public methods by adding ``@Transactional`` at class level. 
       | Since settings are made even at Service side that calls the Repository, the transactions can be managed even without adding ``@Transactional``. 
       | However, since the ``propagation`` attribute is ``REQUIRED`` by default, internal (Repository side) transactions take part in external (Service side) 
       | transactions when the transactions are nested.
   * - | (2)
     - | Inject QueryDAO and UpdateDAO using ``@Inject``.
   * - | (3)
     - | Implementing repository methods involves passing of ``SQLID`` and parameters to TERASOLUNA DAO. 
       | Use ``QueryDAO`` for reference and ``UpdateDAO`` for update. Set SQL corresponding to SQLID as follows.
   * - | (4)
     - | Create new and update are executed using ``save`` method. In order to determine which process is to be executed, create 'exists' method. 
       | In this method, the record count of todoId is acquired and whether the record count is greater than 0 is checked.
\
	.. note::
	   It is convenient to use save method as it can be used to create new and to update. 
	   On the other hand, the disadvantage  of using save method from performance perspective is that SQL gets executed twice. 
	   If performance is higher priority, then create ``create`` method for new creation and ``update`` method for updating the data.


Create SQLMap file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Create ``src/main/resources/META-INF/mybatis/sql/todo-sqlmap.xml`` and mention sql corresponding to ``SQLID`` used in TodoRepositoryImpl as shown below.

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE sqlMap
                PUBLIC "-//ibatis.apache.org//DTD SQL Map 2.0//EN"
                "http://ibatis.apache.org/dtd/sql-map-2.dtd">
    <sqlMap namespace="todo">
        <resultMap id="todo" class="todo.domain.model.Todo">
            <result property="todoId" column="todo_id" />
            <result property="todoTitle" column="todo_title" />
            <result property="finished" column="finished" />
            <result property="createdAt" column="created_at" />
        </resultMap>

        <select id="findOne" parameterClass="java.lang.String"
            resultMap="todo"><![CDATA[
    SELECT todo_id,
           todo_title,
           finished,
           created_at
    FROM   todo
    WHERE  todo_id = #value#
    ]]></select>

        <select id="findAll" resultMap="todo"><![CDATA[
    SELECT todo_id,
           todo_title,
           finished,
           created_at
    FROM   todo
    ]]></select>

        <insert id="create" parameterClass="todo.domain.model.Todo"><![CDATA[
    INSERT INTO todo
                (todo_id,
                 todo_title,
                 finished,
                 created_at)
    VALUES      ( #todoId#,
                 #todoTitle#,
                 #finished#,
                 #createdAt# )
    ]]></insert>

        <update id="update" parameterClass="todo.domain.model.Todo"><![CDATA[
    UPDATE todo
    SET    todo_title = #todoTitle#,
           finished = #finished#,
           created_at = #createdAt#
    WHERE  todo_id = #todoId#
    ]]></update>

        <delete id="delete" parameterClass="todo.domain.model.Todo"><![CDATA[
    DELETE FROM todo
    WHERE  todo_id = #todoId#
    ]]></delete>

        <select id="countByFinished" parameterClass="java.lang.Boolean"
            resultClass="java.lang.Long"><![CDATA[
    SELECT COUNT(*)
    FROM   todo
    WHERE  finished = #value#
    ]]></select>

        <select id="exists" parameterClass="java.lang.String"
            resultClass="java.lang.Long"><![CDATA[
    SELECT COUNT(*)
    FROM   todo
    WHERE  todo_id = #value#
    ]]></select>
    </sqlMap>

With this, usage of TERASOLUNA DAO is completed. Start AP server,
SQL log and transaction log as shown below are output for the display of Todo and for creating a new one.


.. code-block:: guess
   :emphasize-lines: 2-12

    2013-11-28 14:15:37 [tomcat-http--3] [TRACE] [o.t.gfw.web.logging.TraceLoggingInterceptor     ] - [START CONTROLLER] TodoController.list(Model)
    2013-11-28 14:15:37 [tomcat-http--3] [DEBUG] [o.s.jdbc.datasource.DataSourceTransactionManager] - Creating new transaction with name [todo.domain.serivce.todo.TodoServiceImpl.findAll]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT,readOnly; ''
    2013-11-28 14:15:37 [tomcat-http--3] [DEBUG] [o.s.jdbc.datasource.DataSourceTransactionManager] - Acquired Connection [jdbc:h2:mem:todo, UserName=SA, H2 JDBC Driver] for JDBC transaction
    2013-11-28 14:15:37 [tomcat-http--3] [DEBUG] [o.s.jdbc.datasource.DataSourceTransactionManager] - Participating in existing transaction
    2013-11-28 14:15:37 [tomcat-http--3] [DEBUG] [java.sql.Connection                             ] - {conn-100014} Connection
    2013-11-28 14:15:37 [tomcat-http--3] [DEBUG] [java.sql.Connection                             ] - {conn-100014} Preparing Statement:  SELECT todo_id,        todo_title,        finished,        created_at FROM   todo 
    2013-11-28 14:15:37 [tomcat-http--3] [DEBUG] [java.sql.PreparedStatement                      ] - {pstm-100015} Executing Statement:  SELECT todo_id,        todo_title,        finished,        created_at FROM   todo 
    2013-11-28 14:15:37 [tomcat-http--3] [DEBUG] [java.sql.PreparedStatement                      ] - {pstm-100015} Parameters: []
    2013-11-28 14:15:37 [tomcat-http--3] [DEBUG] [java.sql.PreparedStatement                      ] - {pstm-100015} Types: []
    2013-11-28 14:15:37 [tomcat-http--3] [DEBUG] [o.s.jdbc.datasource.DataSourceTransactionManager] - Initiating transaction commit
    2013-11-28 14:15:37 [tomcat-http--3] [DEBUG] [o.s.jdbc.datasource.DataSourceTransactionManager] - Committing JDBC transaction on Connection [jdbc:h2:mem:todo, UserName=SA, H2 JDBC Driver]
    2013-11-28 14:15:37 [tomcat-http--3] [DEBUG] [o.s.jdbc.datasource.DataSourceTransactionManager] - Releasing JDBC Connection [jdbc:h2:mem:todo, UserName=SA, H2 JDBC Driver] after transaction
    2013-11-28 14:15:37 [tomcat-http--3] [TRACE] [o.t.gfw.web.logging.TraceLoggingInterceptor     ] - [END CONTROLLER  ] TodoController.list(Model)-> view=todo/list, model={todoForm=todo.app.todo.TodoForm@1386751, todos=[todo.domain.model.Todo@72edc], org.springframework.validation.BindingResult.todoForm=org.springframework.validation.BeanPropertyBindingResult: 0 errors}
    2013-11-28 14:15:37 [tomcat-http--3] [TRACE] [o.t.gfw.web.logging.TraceLoggingInterceptor     ] - [HANDLING TIME   ] TodoController.list(Model)-> 2,461,131 ns

|

In the end...
================================================================================
In this tutorial, following contents have been learnt

* How to develop basic applications by TERASOLUNA Global Framework, and how to build Eclipse project
 * Using the STS
 * How to use TERASOLUNA Global Framework with Maven
* Way of development using application layering of TERASOLUNA Global Framework.
 * Implementation of domain layer with POJO(+ Spring)
 * Implementation of application layer with the use of JSP tag libraries and Sring MVC
 * Development of Infrastructure layer with the use of Spring Data JPA
 * Development of Infrastructure layer with the use of MyBatis2

The following improvement can be done in the TODO management application.

* To externalize the property → :doc:`../ArchitectureInDetail/PropertyManagement`
* To externalize the messages → :doc:`../ArchitectureInDetail/MessageManagement`
* To add paging → :doc:`../ArchitectureInDetail/Pagination`
* To add exception handling → :doc:`../ArchitectureInDetail/ExceptionHandling`
* To add a CSRF measures → :doc:`../Security/CSRF`


.. raw:: latex

   \newpage

