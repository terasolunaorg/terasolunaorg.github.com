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

* Basic application development using TERASOLUNA Server Framework for Java (5.x)
* Configuration of Maven and STS(Eclipse) project
* Way of development using application layering of TERASOLUNA Server Framework for Java (5.x).


Target readers
--------------------------------------------------------------------------------

* Basic knowledge of Spring DI and AOP required
* Web application development using Servlet/JSP
* SQL knowledge


Verification environment
--------------------------------------------------------------------------------

In this tutorial, operations are verified on following environment. In case of implementation on other environment, perform appropriate setting based on this guideline.

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - Type
      - Name
    * - OS
      - Windows 7
    * - JVM
      - `Java <http://www.oracle.com/technetwork/java/javase/downloads/index.html>`_ 1.8
    * - IDE
      - `Spring Tool Suite <http://spring.io/tools/sts/all>`_ 3.6.4.RELEASE (Onwards referred as [STS])
    * - Build Tool
      - `Apache Maven <http://maven.apache.org/download.cgi>`_ 3.3.9 (Onwards referred as [Maven])
    * - Application Server
      - `Pivotal tc Server <https://network.pivotal.io/products/pivotal-tcserver>`_ Developer Edition v3.1 (Enclosed in STS)
    * - Web Browser
      - `Google Chrome <https://www.google.co.jp/chrome/browser/desktop/index.html>`_ 46.0.2490.80 m

|

Description of application to be created
================================================================================

Overview of the application
--------------------------------------------------------------------------------

Application to manage TODO list is to be developed. TODO list display, TODO registration, TODO completion and TODO deletion can be performed.

.. figure:: ./images/image001.png
    :width: 50%

.. _app-requirement:

Business requirements of application
--------------------------------------------------------------------------------
Business requirement of the application is as follows.

.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 80

    * - RuleID
      - Description
    * - B01
      - Only up to 5 incomplete TODO records can be registered
    * - B02
      - For TODOs which are already completed, "TODO Complete" processing cannot be done.

.. note::

     This application is for learning purpose only. It is not suitable as a real todo management application.

|

Screen transition of application
--------------------------------------------------------------------------------
Processing specification and screen transition of an application is as follows.

.. figure:: ./images/image002.png
   :width: 60%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.10\linewidth}|p{0.15\linewidth}|p{0.45\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 20 10 15 45

    * - Sr.No.
      - Process name
      - HTTP method
      - URL
      - Remark
    * - 1
      - Show all TODO
      - \-
      - /todo/list
      -
    * - 2
      - Create TODO
      - POST
      - /todo/create
      - Redirect to Show all TODO after creation is completed
    * - 3
      - Finish TODO
      - POST
      - /todo/finish
      - Redirect to Show all TODO after completion process
    * - 4
      - Delete TODO
      - POST
      - /todo/delete
      - Redirect to Show all TODO after deletion is completed

Show all TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* Display all records of TODO
* Provide ``Finish`` and  ``Delete`` buttons for incomplete TODO
* Strike-through the completed records of TODO
* Display only record name of TODO


Create TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* Save TODO sent from the form
* Record name of TODO should be between 1 - 30 characters
* When :ref:`app-requirement` B01 is not fulfilled, business exception with error code E001 is thrown
* Display "Created successfully!" at the transited screen when the creation process is successful.

Finish TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* For the TODOs corresponding to \ ``todoId`` \ which is received from the form object, change the status to ``completed``.
* When the corresponding TODO does not exist, resource not found exception with error code E404 is thrown
* When :ref:`app-requirement` B02 is not fulfilled, business exception with error code E002 is thrown
* Display "Finished successfully!" at the transited screen when the finishing process is successful.

Delete TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* Delete TODO corresponding to \ ``todoId`` \ sent from the form
* When the corresponding TODO does not exist, resources undetected exception with error code E404 is thrown
* Display "Deleted successfully!" at the transited screen when the deletion process is successful.

|

Error message list
--------------------------------------------------------------------------------

Define below 3 error messages.

.. tabularcolumns:: |p{0.15\linewidth}|p{0.50\linewidth}|p{0.35\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 15 50 35

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

|

Environment creation
================================================================================

In this tutorial, as RepositoryImpl implementation of the infrastructure layer,

* In-memory implementation of RepositoryImpl using \ ``java.util.Map`` \ without using the database
* RepositoryImpl access the database using MyBatis3
* RepositoryImpl access the database using Spring Data JPA

3 types are prepared. Select anyone depending upon the use.

Under this tutorial, First, try the in-memory implementation followed by select myBatis3 or Spring Data JPA.

Project creation
--------------------------------------------------------------------------------

First, create a blank project for implementation of infrastructure layer using \ ``mvn archetype:generate``\.
This is a procedure to create a blank project using the Windows command prompt.

.. note::

    If internet connection is accessed through proxy server,
    In order to perform the following tasks, necessary STS proxy settings and `Maven proxy setting <http://maven.apache.org/guides/mini/guide-proxies.html>`_ \ needs to be done.

.. tip::

    If \ ``mvn archetype:generate`` \ executes on Bash, it can be executed by replacing the \ ``^`` \ with \ ``\``\ .

     .. code-block:: bash

        mvn org.apache.maven.plugins:maven-archetype-plugin:2.4:generate -B\
         -DarchetypeCatalog=http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases\
         -DarchetypeGroupId=org.terasoluna.gfw.blank\
         -DarchetypeArtifactId=terasoluna-gfw-web-blank-archetype\
         -DarchetypeVersion=5.2.1.RELEASE\
         -DgroupId=todo\
         -DartifactId=todo\
         -Dversion=1.0.0-SNAPSHOT

|

.. _TutorialCreatePlainBlankProject:

Creating O/R Mapper independent blank project
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you want to create a project for RepositoryImpl using \ ``java.util.Map``\ (without accessing the database), 
run the following command to create O/R Mapper independent blank project in command prompt.  \ **If you read through this tutorial in consecutive order, first of all, create a project in this way**\ .

.. code-block:: console

    mvn org.apache.maven.plugins:maven-archetype-plugin:2.4:generate -B^
     -DarchetypeCatalog=http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases^
     -DarchetypeGroupId=org.terasoluna.gfw.blank^
     -DarchetypeArtifactId=terasoluna-gfw-web-blank-archetype^
     -DarchetypeVersion=5.2.1.RELEASE^
     -DgroupId=todo^
     -DartifactId=todo^
     -Dversion=1.0.0-SNAPSHOT

.. _TutorialCreateMyBatis3BlankProject:

Creating blank project for MyBatis3
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you want to create a project for RepositoryImpl to access the database using MyBatis3, 
run the following command to create a blank project for the MyBatis3. This way to create a project is to be done in \ :ref:`using_MyBatis3`\ .

.. code-block:: console

    mvn org.apache.maven.plugins:maven-archetype-plugin:2.4:generate -B^
     -DarchetypeCatalog=http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases^
     -DarchetypeGroupId=org.terasoluna.gfw.blank^
     -DarchetypeArtifactId=terasoluna-gfw-web-blank-mybatis3-archetype^
     -DarchetypeVersion=5.2.1.RELEASE^
     -DgroupId=todo^
     -DartifactId=todo^
     -Dversion=1.0.0-SNAPSHOT

.. _TutorialCreateJPABlankProject:

Creating blank project for JPA
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you want to create a project for RepositoryImpl to access the database using Spring Data JPA, 
run the following command to create a blank project for the JPA. This way to create a project is to be done in \ :ref:`using_SpringDataJPA`\ .

.. code-block:: console

    mvn org.apache.maven.plugins:maven-archetype-plugin:2.4:generate -B^
     -DarchetypeCatalog=http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases^
     -DarchetypeGroupId=org.terasoluna.gfw.blank^
     -DarchetypeArtifactId=terasoluna-gfw-web-blank-jpa-archetype^
     -DarchetypeVersion=5.2.1.RELEASE^
     -DgroupId=todo^
     -DartifactId=todo^
     -Dversion=1.0.0-SNAPSHOT

|

Project import
--------------------------------------------------------------------------------

Import created blank project into STS.

Select the archetype created project from STS menu [File] -> [Import] -> [Maven] -> [Existing Maven Projects] -> [Next].

.. figure:: images/NewMVCProjectImport.png
   :alt: New MVC Project Import
   :width: 60%

|

Click on [Finish] by selecting \ ``C:\work\todo`` \ in Root Directory and selecting pom.xml of todo in Projects.

.. figure:: images/NewMVCProjectCreate.png
   :alt: New MVC Project Import
   :width: 60%

|

When the import is completed, project is displayed in the Package Explorer as follows.

.. figure:: images/image004.png
   :alt: workspace

.. note::

    If the build error occurs after the import, it can be removed by 
    Right click on the project name in Package Explorer and select [Maven] -> [Update Project...] -> [OK] .

     .. figure:: images/update-project.png
        :width: 70%

.. tip::

    For better visibility, Package Presentation must be changed to [Hierarchical] from default [Flat].

    Click [View Menu] (The right edge of the down arrow) of the Package Explorer and select [Package Presentation] -> [Hierarchical].

     .. figure:: ./images/presentation-hierarchical.png
        :width: 80%

    It can be displayed as follows if Package Presentation changed into [Hierarchical].

     .. figure:: ./images/presentation-hierarchical-view.png

.. warning::
 
    H2 Database has been defined as a dependency in O/R Mapper type blank project but,
    this setting is done to create a simple application easily therefore it is not intended to use in the actual application development.
    
    The following definitions shall be removed while performing the actual application development.
    
     .. code-block:: xml

        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        
note::  
 
	In the above setting example, since it is assumed that the dependent library version is managed by the parent project terasoluna-gfw-parent , specifying the version in pom.xml is not necessary.
	The above dependent library used by terasoluna-gfw-parent is defined by \ `Spring IO Platform <http://platform.spring.io/platform/>`_\ .

|

Project configuration
--------------------------------------------------------------------------------

Below is the structure of the project to be created in this tutorial.

.. note::

    It had been recommended to use a multi-project structure in :ref:`[Project Structure] section of previous chapter <application-layering_project-structure>` but,
    in this tutorial, a single project configuration is used because it focuses on ease of learning.
    However, when in a real project, multi project configuration is strongly recommended.

    **However, when in a real project, multi project configuration is strongly recommended.**

    For creating multi-project, Refer [:doc:`../ImplementationAtEachLayer/CreateWebApplicationProject`].

|

**[Configuration of blank project created for MyBatis3]**

.. code-block:: console

    src
      └main
          ├java
          │  └todo
          │    ├ app
          │    │   └todo
          │    └domain
          │        ├model
          │        ├repository
          │        │   └todo
          │        └service
          │            └todo
          ├resources
          │  ├META-INF
          │  │  ├mybatis ... (8)
          │  │  └spring
          │  └todo
          │    └domain
          │        └repository ... (9)
          │             └todo
          └wepapp
              └WEB-INF
                  └views


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (8)
      - Directory that contains the MyBatis configuration files.
    * - | (9)
      - Directory that contains the Mapper of MyBatis describing the SQL.

        In this tutorial, create a directory for storing the Mapper file of Repository for Todo object.

|

**[Configuration of blank project created for JPA, O/R Mapper independent blank project]**

.. code-block:: console

    src
      └main
          ├java
          │  └todo
          │    ├ app ... (1)
          │    │   └todo
          │    └domain ... (2)
          │        ├model ... (3)
          │        ├repository ... (4)
          │        │   └todo
          │        └service ... (5)
          │            └todo
          ├resources
          │  └META-INF
          │      └spring ... (6)
          └wepapp
              └WEB-INF
                  └views ... (7)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Packages for storing application layer classes.

        In this tutorial, creating package for storing the Todo business classes.
    * - | (2)
      - Packages for storing domain layer class.
    * - | (3)
      - Packages for storing Domain Object.
    * - | (4)
      - Packages for storing Repository

        In this tutorial, creating package for storing the Todo object (Domain Object) Repository.
    * - | (5)
      - Packages for storing Service.

        In this tutorial, creating package for storing the Todo business services.
    * - | (6)
      - Directory that contains the Spring configuration files.
    * - | (7)
      - Directory for storing jsp.

|

Confirmation of configuration file
--------------------------------------------------------------------------------
Many settings those are required in advanced tutorial already done in the created blank project.

If only the implementation of the tutorial is concern, understanding of these settings are not required but
it is recommended that you understand what settings are necessary to run an application.

For description of the required configuration (settings file) to run an application, Refer [:ref:`TutorialTodoAppendixExpoundConfigurations`].

.. note::
 
    If you are creating a Todo application for familiarizing with the system, you may skip the confirmation of configuration file but
    suggest to read this after creating the Todo application.

|

Operation verification of the project
--------------------------------------------------------------------------------
Before starting the development of Todo application, verify the project operation.

Since the implementation of the Controller and JSP for displaying the top page are provided in the blank project,
it is possible to check the operation by displaying the top page.

The following implementation has been done in the Controller(\ :file:`src/main/java/todo/app/welcome/HelloController.java`\ ),
provided in the blank project.

.. code-block:: java
    :emphasize-lines: 17, 21, 28, 31, 40, 43

    package todo.app.welcome;

    import java.text.DateFormat;
    import java.util.Date;
    import java.util.Locale;

    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;

    /**
     * Handles requests for the application home page.
     */
    // (1)
    @Controller
    public class HelloController {

        // (2)
        private static final Logger logger = LoggerFactory
                .getLogger(HelloController.class);

        /**
         * Simply selects the home view to render by returning its name.
         */
        // (3)
        @RequestMapping(value = "/", method = {RequestMethod.GET, RequestMethod.POST})
        public String home(Locale locale, Model model) {
            // (4)
            logger.info("Welcome home! The client locale is {}.", locale);
    
            Date date = new Date();
            DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG,
                    DateFormat.LONG, locale);

            String formattedDate = dateFormat.format(date);

            // (5)
            model.addAttribute("serverTime", formattedDate);

            // (6)
            return "welcome/home";
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | In order to make the Controller as component-scan target, attach \ ``@Controller`` \ annotation to class level.
   * - | (2)
     - | Generating logger for outputting the log at (4).
       | Logger impmements the logback but, the API \ ``org.slf4j.Logger`` \ of SLF4J is used.
   * - | (3)
     - | Set mapping methods for accessing the \ ``"/"`` \ (root) using ``@RequestMapping`` annotation.
   * - | (4)
     - | Outputting info level log for notifying that the method is called.
   * - | (5)
     - | For displaying date on screen set date as \ ``"serverTime"`` \ attribute name to the Model.
   * - | (6)
     - | Return \ ``"welcome/home"`` \ as view name. Using \ ``ViewResolver`` \ settings, \ ``WEB-INF/views/welcome/home.jsp`` \ is called.

|

The following implementation has been done in the JSP(\ :file:`src/main/webapp/WEB-INF/views/welcome/home.jsp`\ ),
provided in the blank project.

.. code-block:: jsp
    :emphasize-lines: 12

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
            <h1>Hello world!</h1>
            <!-- (7) -->
            <p>The time on the server is ${serverTime}.</p>
        </div>
    </body>
    </html>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr. No.
     - Description
   * - | (7)
     - | Display \ ``"serverTime"`` \ passed from Controller in Model.
       | Here, XSS measures are done but always performs the XSS measures for displaying the user input values by \ ``f:h()`` \ function.

|

Right click on project and select [Run As] -> [Run on Server].

.. figure:: ./images/image031.jpg
   :width: 70%

|

Select [Next] after selecting AP server (Pivotal tc Server Developer Edition v3.1).

.. figure:: ./images/image032.jpg
   :width: 70%

|

Verify that todo is included in [Configured] and click [Finish] to start the server.

.. figure:: ./images/image033.jpg
   :width: 70%

|

When started, log shown as below will be output.
For \ ``"/"`` \ path, it is understood that hello method of \ ``todo.app.welcome.HelloController`` \ is mapped.


.. code-block:: console
   :emphasize-lines: 3

    date:2016-02-17 11:25:30	thread:localhost-startStop-1	X-Track:	level:INFO 	logger:o.springframework.web.servlet.DispatcherServlet 	message:FrameworkServlet 'appServlet': initialization started
    date:2016-02-17 11:25:31	thread:localhost-startStop-1	X-Track:	level:DEBUG	logger:o.t.gfw.web.codelist.CodeListInterceptor        	message:registered codeList : []
    date:2016-02-17 11:25:31	thread:localhost-startStop-1	X-Track:	level:INFO 	logger:o.s.w.s.m.m.a.RequestMappingHandlerMapping      	message:Mapped "{[/],methods=[GET || POST],params=[],headers=[],consumes=[],produces=[],custom=[]}" onto public java.lang.String todo.app.welcome.HelloController.home(java.util.Locale,org.springframework.ui.Model)
    date:2016-02-17 11:25:31	thread:localhost-startStop-1	X-Track:	level:INFO 	logger:o.s.w.s.m.m.a.RequestMappingHandlerAdapter      	message:Looking for @ControllerAdvice: WebApplicationContext for namespace 'appServlet-servlet': startup date [Wed Feb 17 11:25:30 JST 2016]; parent: Root WebApplicationContext
    date:2016-02-17 11:25:32	thread:localhost-startStop-1	X-Track:	level:INFO 	logger:o.s.w.s.m.m.a.RequestMappingHandlerAdapter      	message:Looking for @ControllerAdvice: WebApplicationContext for namespace 'appServlet-servlet': startup date [Wed Feb 17 11:25:30 JST 2016]; parent: Root WebApplicationContext
    date:2016-02-17 11:25:32	thread:localhost-startStop-1	X-Track:	level:INFO 	logger:o.s.web.servlet.handler.SimpleUrlHandlerMapping 	message:Mapped URL path [/**] onto handler 'org.springframework.web.servlet.resource.DefaultServletHttpRequestHandler#0'
    date:2016-02-17 11:25:32	thread:localhost-startStop-1	X-Track:	level:INFO 	logger:o.s.web.servlet.handler.SimpleUrlHandlerMapping 	message:Mapped URL path [/resources/**] onto handler 'org.springframework.web.servlet.resource.ResourceHttpRequestHandler#0'
    date:2016-02-17 11:25:33	thread:localhost-startStop-1	X-Track:	level:INFO 	logger:o.springframework.web.servlet.DispatcherServlet 	message:FrameworkServlet 'appServlet': initialization completed in 2826 ms

|

If http://localhost:8080/todo is accessed in browser, following is displayed.

.. figure:: ./images/image034.png
   :width: 60%


If you confirm the console,

* TRACE log of \ ``TraceLoggingInterceptor`` \ provided by the common library
* INFO log implemented in the Controller

are the log output.

.. code-block:: console
   :emphasize-lines: 1-4

    date:2016-02-17 11:25:35	thread:tomcat-http--11	X-Track:b49b630274974bffbcd9e8d13261f6a7	level:TRACE	logger:o.t.gfw.web.logging.TraceLoggingInterceptor     	message:[START CONTROLLER] HelloController.home(Locale,Model)
    date:2016-02-17 11:25:35	thread:tomcat-http--11	X-Track:b49b630274974bffbcd9e8d13261f6a7	level:INFO 	logger:todo.app.welcome.HelloController                 	message:Welcome home! The client locale is ja_JP.
    date:2016-02-17 11:25:35	thread:tomcat-http--11	X-Track:b49b630274974bffbcd9e8d13261f6a7	level:TRACE	logger:o.t.gfw.web.logging.TraceLoggingInterceptor     	message:[END CONTROLLER  ] HelloController.home(Locale,Model)-> view=welcome/home, model={serverTime=2016/02/17 11:25:35 JST}
    date:2016-02-17 11:25:35	thread:tomcat-http--11	X-Track:b49b630274974bffbcd9e8d13261f6a7	level:TRACE	logger:o.t.gfw.web.logging.TraceLoggingInterceptor     	message:[HANDLING TIME   ] HelloController.home(Locale,Model)-> 97,346,576 ns

.. note::
 
    The \ ``TraceLoggingInterceptor`` \ outputs start and end log of the Controller. While ending, the information of \ ``View`` \ and \ ``Model`` \ as well as processing time are output.

|

Creation of Todo application
================================================================================
| Create Todo application. Order in which it must be created is as follows

* Domain layer (+ Infrastructure layer)

 * Domain Object creation
 * Repository creation
 * RepositoryImpl creation
 * Service creation

* Application layer

 * Controller creation
 * Form creation
 * View creation

|

About the creation of RepositoryImpl, implementation is differ depending on the type of the selected infrastructure layer.

Here, In-memory implemented RepositoryImpl is created using \ ``java.util.Map`` \ without using the database is explained.
If you want to use database, create Todo application by referring the [:ref:`tutorial-todo_infra`] content.

|

Creation of Domain layer
--------------------------------------------------------------------------------

Creation of Domain Object
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create Domain object.

Right click on the Package Explorer, select -> [New] -> [Class] -> [New Java Class] dialog box appears, 

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - Sr. No.
      - Item
      - Input value
    * - 1
      - Package
      - ``todo.domain.model``
    * - 2
      - Name
      - ``Todo``
    * - 3
      - Interfaces
      - ``java.io.Serializable``

and click [Finish] after entering above information.

.. figure:: ./images/image057.png
   :width: 70%

Created class stored in the following directory.

.. figure:: ./images/image058.png

|

Add following properties in created class.

* ID = todoId
* Title = todoTitle
* Completion flag = finished
* Created on = createdAt

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

.. tip::

    Getter/Setter methods can be generated automatically by using STS feature.
    After defining fields, right click on editor and select [Source] -> [Generate Getter and Setters…]

        .. figure:: ./images/image059.png
           :width: 90%

    Click [OK] after selecting all other than serialVersionUID.

        .. figure:: ./images/image060.png
           :width: 60%

|

.. _TutorialTodoCreateRepository:

Repository creation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create \ ``TodoRepository`` \ interface.
If you want use database, create Repository by referring the [:ref:`tutorial-todo_infra`] content.

Right click on the Package Explorer, select -> [New] -> [Interface] -> [New Java Interface] dialog box appears, 

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - Sr. No.
      - Item
      - Input value
    * - 1
      - Package
      - ``todo.domain.repository.todo``
    * - 2
      - Name
      - ``TodoRepository``

and click [Finish] after entering above information.

Created Interface stored in the following directory.

.. figure:: ./images/image061.png

Define following CRUD operation methods pertaining to this application in created interface.

* Fetch 1 record of TODO = findOne
* Fetch all records of TODO = findAll
* Create 1 record of TODO = create
* Update 1 record of TODO = update
* Delete 1 record of TODO = delete
* Fetch record count of completed TODO = countByFinished

.. code-block:: java

    package todo.domain.repository.todo;

    import java.util.Collection;

    import todo.domain.model.Todo;

    public interface TodoRepository {
        Todo findOne(String todoId);

        Collection<Todo> findAll();

        void create(Todo todo);

        boolean update(Todo todo);

        void delete(Todo todo);

        long countByFinished(boolean finished);
    }

.. note::

    Here, to improve versatility of \ ``TodoRepository``\, instead of (\ ``long countFinished()``\) method for [Fetch completed record count],
    (\ ``long countByFinished(boolean)``\) method for [record count having xx completion status ] is defined.
    
    If pass \ ``true`` \ as an argument to \ ``long countByFinished(boolean)``\, [completed record count] and 
    if pass \ ``false`` \ as an argument, [not completed record count] can be fetched by specification.

|

Creation of RepositoryImpl (Infrastructure layer)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Here, In-memory implemented RepositoryImpl is created using \ ``java.util.Map`` \ for simplification.
If you want use database, create RepositoryImpl by referring the [:ref:`tutorial-todo_infra`] content.

Right click on the Package Explorer, select -> [New] -> [Class] -> [New Java Class] dialog box appears, 

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - Sr. No.
      - Item
      - Input value
    * - 1
      - Package
      - ``todo.domain.repository.todo``
    * - 2
      - Name
      - ``TodoRepositoryImpl``
    * - 3
      - Interfaces
      - ``todo.domain.repository.todo.TodoRepository``

and click [Finish] after entering above information.

Created class stored in the following directory.

.. figure:: ./images/image062.png

Implement the CRUD operations in created class.

.. note::
 
    Business logic must not be included in RepositoryImpl,  it should focus only on inserting and removing (CRUD operation) from the persistence store.

.. code-block:: java
    :emphasize-lines: 11

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
        public void create(Todo todo) {
            TODO_MAP.put(todo.getTodoId(), todo);
        }

        @Override
        public boolean update(Todo todo) {
            TODO_MAP.put(todo.getTodoId(), todo);
            return true;
        }

        @Override
        public void delete(Todo todo) {
            TODO_MAP.remove(todo.getTodoId());
        }

        @Override
        public long countByFinished(boolean finished) {
            long count = 0;
            for (Todo todo : TODO_MAP.values()) {
                if (finished == todo.isFinished()) {
                    count++;
                }
            }
            return count;
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 80

   * - Sr. No.
     - Description
   * - | (1)
     - | To consider Repository as component scan target, add \ ``@Repository`` \ annotation at class level. 

.. note::
 
    In this tutorial, although infrastructure layer belonging classes (RepositoryImpl) are stored under domain layer package (\ ``todo.domain``\),
    if package is divided completed on the basis of layers, it is better to create classes of infrastructure layer under \ ``todo.infra``\.

    However, in a normal project, infrastructure layer rarely changes (such projects are less).
    Hence, in order to improve the work efficiency, RepositoryImpl can be created in the layer same as the repository of domain layer.

|

Service creation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

First create \ ``TodoService`` \ interface.

Right click on the Package Explorer, select -> [New] -> [Interface] -> [New Java Interface] dialog box appears, 

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - Sr. No.
      - Item
      - Input value
    * - 1
      - Package
      - ``todo.domain.service.todo``
    * - 2
      - Name
      - ``TodoService``

and click [Finish] after entering above information.

Created Interface stored in the following directory.

.. figure:: ./images/image063.png

Define method to perform the following business processing in created class.

* Fetch all records of Todo = findAll
* New creation of Todo = create
* Completed of Todo = finish
* Delete of Todo = delete

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

|

Next, create \ ``TodoServiceImpl`` \ class that implements the methods defined in \ ``TodoService`` \ interface.

Right click on the Package Explorer, select -> [New] -> [Class] -> [New Java Class] dialog box appears, 

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - Sr. No.
      - Item
      - Input value
    * - 1
      - Package
      - ``todo.domain.service.todo``
    * - 2
      - Name
      - ``TodoServiceImpl``
    * - 3
      - Interfaces
      - ``todo.domain.service.todo.TodoService``

and click [Finish] after entering above information.

Created class stored in the following directory.

.. figure:: ./images/image064.png

.. code-block:: java
    :emphasize-lines: 19, 20, 25-26, 28-29, 32-33, 37-38, 44, 57-58, 61-62, 71, 90

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

    @Service// (1)
    @Transactional // (2)
    public class TodoServiceImpl implements TodoService {

        private static final long MAX_UNFINISHED_COUNT = 5;

        @Inject// (3)
        TodoRepository todoRepository;

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
        @Transactional(readOnly = true) // (7)
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
                // (8)
                throw new BusinessException(messages);
            }

            // (9)
            String todoId = UUID.randomUUID().toString();
            Date createdAt = new Date();

            todo.setTodoId(todoId);
            todo.setCreatedAt(createdAt);
            todo.setFinished(false);

            todoRepository.create(todo);
            /* REMOVE THIS LINE IF YOU USE JPA
                todoRepository.save(todo); // 10
               REMOVE THIS LINE IF YOU USE JPA */

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
            todoRepository.update(todo);
            /* REMOVE THIS LINE IF YOU USE JPA
                todoRepository.save(todo); // (11)
               REMOVE THIS LINE IF YOU USE JPA */
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
   :class: longtable


   * - Sr. No.
     - Description
   * - | (1)
     - | To consider Service as component-scan target, add \ ``@Service`` \ annotation at class level.
   * - | (2)
     - | All public methods will be treated as transaction management by attaching the \ ``@Transactional`` \ annotation at class level.
       | By applying annotation, the transaction starts at the timing of method execution starts and transaction commits at the time of method execution successful completion.
       | However, if unexpected exception occurs in between, the transaction roll-backs.
       |
       | If database is not used, \ ``@Transactional`` \ annotation is not required.
   * - | (3)
     - | Inject \ ``TodoRepository`` \ implementation using \ ``@Inject`` \ annotation.
   * - | (4)
     - | Logic of fetching single record is used in both delete and finish methods. Hence it should be implemented in a method (OK to make it public by declaring in the interface).
   * - | (5)
     - | Use \ ``org.terasoluna.gfw.common.message.ResultMessage`` \ provided in common library, as a class that stores result messages.
       | Currently, for throwing error message, \ ``ResultMessage`` \ is added by specifying message type using \ ``ResultMessages.error()`` \.
   * - | (6)
     - | When targeted data not found, \ ``org.terasoluna.gfw.common.exception.ResourceNotFoundException`` \ provided in common library is thrown.
   * - | (7)
     - | Regarding the reading process only, put \ ``readOnly=true`` \.
       | By this settings, the optimization of transaction control is done while reading process depending upon the O/R Mapper (Not effective in case of JPA used).
       |
       | If database is not used, \ ``@Transactional`` \ annotation is not required.
   * - | (8)
     - | When business error occurs, \ ``org.terasoluna.gfw.common.exception.BusinessException`` \ provided in common library is thrown.
   * - | (9)
     - | UUID is used to generate a unique value. DB sequence may be used.
   * - | (10)
     - | If database is accessed using Spring Data JPA, instead of \ ``create`` \ method, call \ ``save`` \ method.
   * - | (11)
     - | If database is accessed using Spring Data JPA, instead of \ ``update`` \ method, call \ ``save`` \ method.

.. raw:: latex

   \newpage

.. note::

    In this chapter, error messages are hard coded for simplification, but in reality it is not preferred from maintenance viewpoint.
    Usually, it is recommended to create message externally in property file.
    The method for creating the external property file is described in \ :doc:`../ArchitectureInDetail/GeneralFuncDetail/PropertyManagement`\.

|

Creation of JUnit for Service
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo:: **TBD**
 
    For information about how to Unit test the service, planned to be described in next version later.

|

Creation of application layer
--------------------------------------------------------------------------------

Since domain layer implementation is completed, use the domain layer to create application layer.

Creation of Controller
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

First create Controller that controls screen transition of todo business application.

Right click on the Package Explorer, select -> [New] -> [Class] -> [New Java Class] dialog box appears, 

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - Sr. No.
      - Item
      - Input value
    * - 1
      - Package
      - ``todo.app.todo``
    * - 2
      - Name
      - ``TodoController``

and click [Finish] after entering above information.

.. note::

    **It should be noted that the higher level package is different from the domain layer.**

Created class stored in the following directory.

.. figure:: ./images/image065.png

.. code-block:: java
    :emphasize-lines: 6, 7

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

   * - Sr. No.
     - Description
   * - | (1)
     - | In order to make Controller as component-scan target, add \ ``@Controller`` \ annotation at class level.
   * - | (2)
     - | In order to bring all screen transitions handled by \ ``TodoController``\, under \ ``<contextPath>/todo``\, set \ ``@RequestMapping(“todo”)`` \ at class level.

|

Show all TODO implementation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Perform below in screen that are created in this tutorial.

* Display new form
* Display all records of TODO


Form creation
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Create Form class (JavaBean).

Right click on the Package Explorer, select -> [New] -> [Class] -> [New Java Class] dialog box appears, 

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - Sr. No.
      - Item
      - Input value
    * - 1
      - Package
      - ``todo.app.todo``
    * - 2
      - Name
      - ``TodoForm``
    * - 3
      - Interfaces
      - ``java.io.Serializable``

and click [Finish] after entering above information.

Created class stored in the following directory.

.. figure:: ./images/image066.png

Add following property in the created class.

* Title = todoTitle

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

Implementation of Controller
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Add list screen display into \ ``TodoController``\.

.. code-block:: java
    :emphasize-lines: 18-19, 21-22, 27, 30, 31

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
        @Inject // (1)
        TodoService todoService;

        @ModelAttribute // (2)
        public TodoForm setUpForm() {
            TodoForm form = new TodoForm();
            return form;
        }

        @RequestMapping(value = "list") // (3)
        public String list(Model model) {
            Collection<Todo> todos = todoService.findAll();
            model.addAttribute("todos", todos); // (4)
            return "todo/list"; // (5)
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Add \ ``@Inject`` \ annotation for injecting \ ``TodoService`` \ using DI container.
       |
       | Instance of type \ ``TodoService`` \(instance  of \ ``TodoServiceImpl``\) managed by DI container is injected.
   * - | (2)
     - | Initialize Form
       |
       | By adding \ ``@ModelAttribute`` \ annotation, form object of the return value of this method is added to \ ``Model`` \ with name \ ``"todoForm"``\.
       | It is same as executing \ ``model.addAttribute("todoForm", form)`` \ in each method of \ ``TodoController``\.
   * - | (3)
     - | set \ ``@RequestMapping`` \ annotation such a way that method of list screen display (\ ``list`` \ method) gets executed when it is requested to \ ``/todo/list`` \ path.
       |
       | Since \ ``@RequestMapping(“todo”)`` \ is being set at class level, only \ ``@RequestMapping("list")`` \ is fine here.
   * - | (4)
     - | Add Todo list to \ ``Model`` \ and pass to View.
   * - | (5)
     - | If \ ``"todo/list"`` \ is returned as View name, \ :file:`WEB-INF/views/todo/list.jsp` \ will be rendered using \ ``ViewResolver`` \ defined in spring-mvc.xml.

JSP creation
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Display Model passed from Controller by creating JSP.

Right click on the Package Explorer, select -> [New] -> [File] -> [New File] dialog box appears, 

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - Sr. No.
      - Item
      - Input value
    * - 1
      - Enter or select the parent folder
      - ``todo/src/main/webapp/WEB-INF/views/todo``
    * - 2
      - File name
      - ``list.jsp``

and click [Finish] after entering above information.

Created file stored in the following directory.

.. figure:: ./images/create-list-jsp.png

First, implement necessary JSP for following display.

* Input form of TODO
* [Create Todo] button
* List display area of TODO

.. code-block:: jsp
    :emphasize-lines: 15, 19-20, 27-28, 30, 32-33

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
                <form:button>Create Todo</form:button>
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

   * - Sr. No.
     - Description
   * - | (1)
     - | Display the form of new creation process.
       | The \ ``<form:form>`` \ tag is used for displaying form.
       | Specify name of the form object added to \ ``Model`` \ by Controller in \ ``modelAttribute`` \ attribute. 
       | Specify URL(\ ``<contextPath>/todo/create``\ ) into \ ``action`` \ attribute for running the new creation process.
       | Since the new creation process is a process of updation, specify the \ ``POST`` \ method into the \ ``method`` \ attribute.
       |
       | <contextPath> to be specified in \ ``action`` \ attribute can be fetched by \ ``${pageContext.request.contextPath}``\.
   * - | (2)
     - | Bind form property using \ ``<form:input>`` \ tag.
       | Property name of form which is specified in \ ``modelAttribute`` \ attribute should match with the value of \ ``path`` \ attribute.
   * - | (3)
     - | Display entire list of Todo using \ ``<c:forEach>`` \ tag.
   * - | (4)
     - | Determine whether to decorate text using strike through(\ ``text-decoration: line-through;``\ ) to display, if it is completed (\ ``finished``\ ).
   * - | (5)
     - | **To take XSS countermeasures at the time of output of character string, HTML escape should be performed using f:h() function.**
       | Regarding XSS measures, refer to \ :doc:`../Security/XSS`\.


|

Right click [todo] project in STS and start Web application by [Run As] -> [Run on Server].
If http://localhost:8080/todo/todo/list is accessed in browser, the following screen gets displayed.

.. figure:: ./images/image067.png
   :width: 50%

|

Create TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Next, implement a new creation logic after clicking  [Create TODO] button on List display screen.

Modifications in Controller
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Add new creation process into \ ``TodoController``\.

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
        TodoService todoService;

        // (1)
        @Inject
        Mapper beanMapper;

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

        @RequestMapping(value = "create", method = RequestMethod.POST) // (2)
        public String create(@Valid TodoForm todoForm, BindingResult bindingResult, // (3)
                Model model, RedirectAttributes attributes) { // (4)

            // (5)
            if (bindingResult.hasErrors()) {
                return list(model);
            }

            // (6)
            Todo todo = beanMapper.map(todoForm, Todo.class);

            try {
                todoService.create(todo);
            } catch (BusinessException e) {
                // (7)
                model.addAttribute(e.getResultMessages());
                return list(model);
            }

            // (8)
            attributes.addFlashAttribute(ResultMessages.success().add(
                    ResultMessage.fromText("Created successfully!")));
            return "redirect:/todo/list";
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90
   :class: longtable

   * - Sr. No.
     - Description
   * - | (1)
     - | At the time of converting form object into domain object. Inject the \ ``Mapper`` \ interface of Dozer.
   * - | (2)
     - | Set \ ``@RequestMapping`` \ annotation such a way that method of new creation process (\ ``create`` \ method) gets executed when it is requested to \ ``/todo/create`` \ path using \ ``POST`` \ method.
   * - | (3)
     - | For performing input validation of form, add \ ``@Valid`` \ annotation to form argument. Input validation result is stored in the immediate next argument \ ``BindingResult``\.
   * - | (4)
     - | Display the list screen by redirecting after successful completion.
       | Add \ ``RedirectAttributes`` \ into argument for storing the information to be redirected.
   * - | (5)
     - | Return to list screen in case of input error.
       | Re-execute \ ``list`` \ method as it is necessary to fetch all records of Todo again.
   * - | (6)
     - | Create \ ``Todo`` \ object from \ ``TodoForm`` \ object using \ ``Mapper`` \ interface of Dozer.
       | No need to set if the property name of conversion source and destination is the same.
       | There is no merit in using \ ``Mapper`` \ interface of Dozer to convert only \ ``todoTitle`` \ property, but it is very convenient in case of multiple properties.
   * - | (7)
     - | In case of \ ``BusinessException`` \  while executing business logic, add the result message to \ ``Model`` \ and return to list screen.
   * - | (8)
     - | Since it is created successfully, add the result message to flash scope and redirect to list screen.
       | Since redirect is used, there is no case of browser being read again and a new registration process being \ ``POST``\. (For details, refer to ":ref:`DoubleSubmitProtectionAboutPRG`")
       | Since this time Created successfully message is displayed, \ ``ResultMessages.success()`` \ is used.

.. raw:: latex

   \newpage

Modifications in Form
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

To define input validation rules, add annotation to the form object.

.. code-block:: java
    :emphasize-lines: 5-6,11-12

    package todo.app.todo;

    import java.io.Serializable;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;

    public class TodoForm implements Serializable {
        private static final long serialVersionUID = 1L;

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

.. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 80


   * - Sr. No.
     - Description
   * - | (1)
     - | Enable mandatory check using \ ``@NotNull`` \ annotation.
   * - | (2)
     - | Enable string length check using \ ``@Size`` \ annotation.

Modifications in JSP
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Add the tag for displaying the result message and input check error.

.. code-block:: jsp
    :emphasize-lines: 15-16,22

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
            <t:messagesPanel />

            <form:form
               action="${pageContext.request.contextPath}/todo/create"
                method="post" modelAttribute="todoForm">
                <form:input path="todoTitle" />
                <form:errors path="todoTitle" /><!-- (2) -->
                <form:button>Create Todo</form:button>
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


.. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 80


   * - Sr. No.
     - Description
   * - | (1)
     - | Display result message using \ ``<t:messagesPanel>`` \ tag.
   * - | (2)
     - | Display errors in case of input error using \ ``<form:errors>`` \ tag. Match the value of \ ``path`` \ attribute with \ ``<form:input>`` \ tag.

|

If form is submitted by entering appropriate value in the form, success message is displayed as given below.

.. figure:: ./images/image068.png
   :width: 40%

.. figure:: ./images/image069.png
   :width: 40%


When 6 or more records are registered and business error occurs, error message is displayed.

.. figure:: ./images/image070.png
   :width: 60%


If form is submitted by entering null character, the following error message is displayed.

.. figure:: ./images/image071.png
   :width: 65%

Customize message display
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

If \ ``<t:messagesPanel>`` \ is used, following is the HTML output.

.. code-block:: html

    <div class="alert alert-success"><ul><li>Created successfully!</li></ul></div>

With the following modifications in style sheet (in \ ``<style>`` \ tag of \ ``list.jsp``\), customize appearance of the result message.

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

|

The message is as follows.

.. figure:: ./images/image072.png
   :width: 40%

.. figure:: ./images/image073.png
   :width: 60%

|

Moreover, input error message class can be specified to \ ``cssClass`` \ attribute of \ ``<form:errors>`` \ tag. 

Modify JSP as follows,

.. code-block:: jsp

    <form:errors path="todoTitle" cssClass="text-error" />

and add the following to style sheet.

.. code-block:: css

    .text-error {
        color: #c60f13;
    }

Input error message is as follows.

.. figure:: ./images/image074.png
   :width: 65%

|

Finish TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Add [Finish] button to List display screen and add completion process of TODO.

Modifications in Form
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

About the completion Form, same \ ``TodoForm`` \ is used.

The \ ``todoId`` \ property needs to be added to \ ``TodoForm`` \ but if simply added, \ ``todoId`` \ property check rules for new creation are applied as it is.
For specifying separate rules for new creation and completion in a single Form, set \ ``groups`` \ attribute and perform input check rule group.

Add below property in Form class.

* ID → todoId

.. code-block:: java
    :emphasize-lines: 9-11,13-14,18-20,22-24,27-29,31-33

    package todo.app.todo;

    import java.io.Serializable;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;

    public class TodoForm implements Serializable {
        // (1)
        public static interface TodoCreate {
        };

        public static interface TodoFinish {
        };

        private static final long serialVersionUID = 1L;

        // (2)
        @NotNull(groups = { TodoFinish.class })
        private String todoId;

        // (3)
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


   * - Sr. No.
     - Description
   * - | (1)
     - | Create an interface for grouping the input check rules.
       | For grouping the input check rules, Refer \ :doc:`../ArchitectureInDetail/WebApplicationDetail/Validation`\.
       |
       | However as a new creation prcess \ ``TodoCreate`` \ interface and as a completion process \ ``TodoFinish`` \ interface is created.
   * - | (2)
     - | \ ``todoId`` \ is a property for completion process.
       | Therefore in \ ``groups`` \ attribute of \ ``@NotNull`` \ annotation, the \ ``TodoFinish`` \ interface is specified for indicating input check rule of completion process.
   * - | (3)
     - | \ ``todoTitle`` \ is a property for new creation process.
       | Therefore in \ ``groups`` \ attribute of \ ``@NotNull`` \ annotation and \ ``@Size`` \ annotation, the \ ``TodoCreate`` \ interface is specified for indicating input check rule of new creation process.

Modifications in Controller
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Add completion process logic to \ ``TodoController``\.

Take precaution of \ **using @Validated instead of @Valid** \ for executing the group validation.

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
        TodoService todoService;

        @Inject
        Mapper beanMapper;

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
                @Validated({ Default.class, TodoCreate.class }) TodoForm todoForm, // (1)
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

        @RequestMapping(value = "finish", method = RequestMethod.POST) // (2)
        public String finish(
                @Validated({ Default.class, TodoFinish.class }) TodoForm form, // (3)
                BindingResult bindingResult, Model model,
                RedirectAttributes attributes) {
            // (4)
            if (bindingResult.hasErrors()) {
                return list(model);
            }

            try {
                todoService.finish(form.getTodoId());
            } catch (BusinessException e) {
                // (5)
                model.addAttribute(e.getResultMessages());
                return list(model);
            }

            // (6)
            attributes.addFlashAttribute(ResultMessages.success().add(
                    ResultMessage.fromText("Finished successfully!")));
            return "redirect:/todo/list";
        }
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr. No.
     - Description
   * - | (1)
     - | Change \ ``@Valid`` \ to \ ``@Validated`` \ for executing group validation.

       | Group of input check rules (group interface) can be specified in \ ``value`` \ attribute.
       | \ ``Default.class`` \ is a group interface provided to apply an input validation rules when group is not specified.
   * - | (2)
     - | Set \ ``@RequestMapping`` \ annotation such a way that method of completion process (\ ``finish`` \ method) gets executed when it is requested to \ ``/todo/finish`` \ path using \ ``POST`` \ method.
   * - | (3)
     - | Specify the group interface (\ ``TodoFinish`` \ interface) for Finish processing as group of input check.
   * - | (4)
     - | In case of input error, return to list screen.
   * - | (5)
     - | In case of \ ``BusinessException`` \  while executing business logic, add the result message to \ ``Model`` \ and return to list screen.
   * - | (6)
     - | Since it is created successfully, add the result message to flash scope and redirect to list screen.

.. note::

    Separate Form can also be created for Create and Finish.
    In case of separate Form class, there is no need to group the input check rules therefore definition of input check rules will be simple.

    However, as the number of Form class increase,

    * The number of classes increases
    * Not possible to centrally manage the input check rules due to duplicate properties increases

    Therefore, please note that when the specifications changes, the modification cost will also be more.

    Moreover, if multiple Form objects are initialized by \ ``@ModelAttribute`` \ method,
    unnecessary instance gets generated because every time all Forms are being initialized.

Modifications in JSP
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Add completion process form.

.. code-block:: jsp
    :emphasize-lines: 56-66

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
                <form:button>Create Todo</form:button>
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
                                <!-- (1) -->
                                <form:form
                                    action="${pageContext.request.contextPath}/todo/finish"
                                    method="post"
                                    modelAttribute="todoForm"
                                    cssStyle="display: inline-block;">
                                    <!-- (2) -->
                                    <form:hidden path="todoId"
                                        value="${f:h(todo.todoId)}" />
                                    <form:button>Finish</form:button>
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

   * - Sr. No.
     - Description
   * - | (1)
     - | Display the form for sending the request to complete the TODO if there are incomplete Todo.
       | Specify URL(\ ``<contextPath>/todo/finish``\ ) into \ ``action`` \ attribute for running the completion process.
       | Since the completion process is a process of updating, specify the \ ``POST`` \ method into the \ ``method`` \ attribute.
   * - | (2)
     - | Pass \ ``todoId`` \ as request parameter using \ ``<form:hidden>`` \ tag.
       | Also while setting the value in \ ``value`` \ attribute, **HTML escaping should always be performed using f:h() function.**

|

When pressing the [Finish] button after newly creating Todo, strike-through is shown as below and it can be understood that the operation is completed.


.. figure:: ./images/image075.png
   :width: 40%


.. figure:: ./images/image076.png
   :width: 40%


Delete TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Add [Delete] button on the list display screen and add the deletion process for TODO removal.

Modification in Form
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Create \ ``TodoForm`` \ for deletion form.

.. code-block:: java
    :emphasize-lines: 15-17,21-22

    package todo.app.todo;

    import java.io.Serializable;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;

    public class TodoForm implements Serializable {
        public static interface TodoCreate {
        };

        public static interface TodoFinish {
        };

        // (1)
        public static interface TodoDelete {
        }

        private static final long serialVersionUID = 1L;

        // (2)
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

   * - Sr. No.
     - Description
   * - | (1)
     - | Create the \ ``TodoDelete`` \ interface for deletion processing as group of input check rule.
   * - | (2)
     - | \ ``todoId`` \ property is used in deletion process.
       | Therefore, in the \ ``groups`` \ attribute of \ ``@NotNull`` \ annotation of \ ``todoId``\, specify the \ ``TodoDelete`` \ interface indicating that it is an input validation rules for the deletion process.

Modifications in Controller
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Add the logic for delete processing to \ ``TodoController``\. It is almost same as the completion process.

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
        TodoService todoService;

        @Inject
        Mapper beanMapper;

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

        @RequestMapping(value = "delete", method = RequestMethod.POST) // (1)
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

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr. No.
     - Description
   * - | (1)
     - Set \ ``@RequestMapping`` \ annotation such a way that method of deletion process (\ ``delete`` \ method) gets executed
       when it is requested to \ ``/todo/delete`` \ path using \ ``POST`` \ method.

Modifications in JSP
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Add deletion process form.

.. code-block:: jsp
    :emphasize-lines: 67-76

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
                <form:button>Create Todo</form:button>
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
                                    <form:button>Finish</form:button>
                                </form:form>
                            </c:otherwise>
                        </c:choose>
                        <!-- (1) -->
                        <form:form
                            action="${pageContext.request.contextPath}/todo/delete"
                            method="post" modelAttribute="todoForm"
                            cssStyle="display: inline-block;">
                            <!-- (2) -->
                            <form:hidden path="todoId"
                                value="${f:h(todo.todoId)}" />
                            <form:button>Delete</form:button>
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

   * - Sr. No.
     - Description
   * - | (1)
     - | Display deletion process form.
       | Specify URL(\ ``<contextPath>/todo/delete``\ ) into \ ``action`` \ attribute for running the deletion process.
       | Since the deletion process is a process of updating, specify the \ ``POST`` \ method into the \ ``method`` \ attribute.
   * - | (2)
     - | Pass \ ``todoId`` \ as request parameter using \ ``<form:hidden>`` \ tag.
       | Also while setting the value in \ ``value`` \ attribute, **HTML escaping should always be performed using f:h() function.**

|

When pressing the [Delete] button in an uncompleted TODO state, TODO is deleted as follows.

.. figure:: ./images/image077.png
   :width: 40%

.. figure:: ./images/image078.png
   :width: 40%

Use of CSS file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Although style sheets are directly defined in a JSP file,
generally it is defined in the CSS file while developing the actual application.

Here, how to define style sheet into the CSS file are described.

Add the style sheet definitions in the CSS file (\ ``src/main/webapp/resources/app/css/styles.css``\) that is provided in the blank project.

.. code-block:: css

    /* ... */

    .strike {
        text-decoration: line-through;
    }

    .alert {
        border: 1px solid;
        margin-bottom: 5px;
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

    .alert ul {
        margin: 15px 0px 15px 0px;
    }

    #todoList li {
        margin-top: 5px;
    }

|

Loading CSS file within JSP.

.. code-block:: jsp
    :emphasize-lines: 6-7

    <!DOCTYPE html>
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Todo List</title>
    <!-- (1) -->
    <link rel="stylesheet" href="${pageContext.request.contextPath}/resources/app/css/styles.css" type="text/css">
    </head>
    <body>
        <h1>Todo List</h1>

        <div id="todoForm">
            <t:messagesPanel />

            <form:form
                action="${pageContext.request.contextPath}/todo/create"
                method="post" modelAttribute="todoForm">
                <form:input path="todoTitle" />
                <form:errors path="todoTitle" cssClass="text-error" />
                <form:button>Create Todo</form:button>
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
                                    <form:button>Finish</form:button>
                                </form:form>
                            </c:otherwise>
                        </c:choose>
                        <form:form
                            action="${pageContext.request.contextPath}/todo/delete"
                            method="post" modelAttribute="todoForm"
                            cssStyle="display: inline-block;">
                            <form:hidden path="todoId"
                                value="${f:h(todo.todoId)}" />
                            <form:button>Delete</form:button>
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

   * - Sr. No.
     - Description
   * - | (1)
     - | Delete the style sheet definitions from the JSP file and load the CSS file in which the style sheets are defined.

|

Following would be the layout if CSS file is applied.

.. figure:: ./images/list-screen-css.png
    :width: 40%

|

.. _tutorial-todo_infra:

Creating infrastructure layer with a Database access
================================================================================

In this section, infrastructure layer for persisting Domain objects in the database is explained.

In this tutorial, it explains how to implement the infrastructure layer using following two O/R Mapper.

* MyBatis3
* Spring Data JPA

|

.. _TutorialCreateORMapperBlankProject:

Creating a blank project dependent on O/R Mapper
--------------------------------------------------------------------------------

Here, a blank project dependent on O/R Mapper is created.

At first, recreate the project corresponding to O/R Mapper being used.

* \ :ref:`TutorialCreateMyBatis3BlankProject`\
* \ :ref:`TutorialCreateJPABlankProject`\

Next, \ **copy a file other than **TodoRepositoryImpl class to a newly created project**\ under \ :file:`src`\ folder created upto \ :ref:`tutorial-todo_infra`\.


\ **However, file to be copied must be a newly created file or file with added changes. A file without modifications must not be copied**\.

|

.. _Tutorial_Setup_Database:

Database set-up
--------------------------------------------------------------------------------

Here, perform the Database set-up.

In this tutorial, the H2 Database is used to save the database setup time.

Modification in todo-infra.properties
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Modify the \ :file:`src/main/resources/META-INF/spring/todo-infra.properties` \ settings
for creating tables into H2 Database while booting the AP server.

.. code-block:: properties
    :emphasize-lines: 2-3

    database=H2
    # (1)
    database.url=jdbc:h2:mem:todo;DB_CLOSE_DELAY=-1;INIT=create table if not exists todo(todo_id varchar(36) primary key, todo_title varchar(30), finished boolean, created_at timestamp)
    database.username=sa
    database.password=
    database.driverClassName=org.h2.Driver
    # connection pool
    cp.maxActive=96
    cp.maxIdle=16
    cp.minIdle=0
    cp.maxWait=60000

.. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 80


   * - Sr. No.
     - Description
   * - | (1)
     - | Specify the DDL statements to create tables into INIT parameter of the URL connection.
     
.. note::
 
    If you format the DDL statement that is set to INIT parameter, it will looks like follows.
    
     .. code-block:: sql

        create table if not exists todo (
            todo_id varchar(36) primary key,
            todo_title varchar(30),
            finished boolean,
            created_at timestamp
        )

|

.. _using_MyBatis3:

Creating infrastructure layer with MyBatis3
--------------------------------------------------------------------------------

Here, How to create RepositoryImpl of infrastructure layer using MyBatis3 is explained.

If you want to use the Spring Data JPA, you can skip this section and may proceed to the \ :ref:`using_SpringDataJPA`\.

Create TodoRepository
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ ``TodoRepository`` \ is created by the same way as it is created for without O/R Mapper.
For creation method, refer [:ref:`TutorialTodoCreateRepository`].

Create TodoRepositoryImpl
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If MyBatis3 is used, RepositoryImpl is automatically generated from the Repository interface (Mapper interface).
Therefore, the creation of \ ``TodoRepositoryImpl`` \ is not required. Remove if it is created.

Create Mapper file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create a Mapper file for defining SQL to be executed when the \ ``TodoRepository`` \ interface methods are called.

Right click on the Package Explorer, select -> [New] -> [File] -> [New File] dialog box appears, 

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - Sr. No.
      - Item
      - Input value
    * - 1
      - Enter or select the parent folder
      - ``todo/src/main/resources/todo/domain/repository/todo``
    * - 2
      - File name
      - ``TodoRepository.xml``

and click [Finish] after entering above information.

Created file stored in the following directory.

.. figure:: ./images/create-mapper-for-mybatis3.png


Describe the SQL to be executed when the \ ``TodoRepository`` \ methods defined in the interfaces are called.

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

    <!-- (1) -->
    <mapper namespace="todo.domain.repository.todo.TodoRepository">

        <!-- (2) -->
        <resultMap id="todoResultMap" type="Todo">
            <id property="todoId" column="todo_id" />
            <result property="todoTitle" column="todo_title" />
            <result property="finished" column="finished" />
            <result property="createdAt" column="created_at" />
        </resultMap>

        <!-- (3) -->
        <select id="findOne" parameterType="String" resultMap="todoResultMap">
        <![CDATA[
            SELECT
                todo_id,
                todo_title,
                finished,
                created_at
            FROM
                todo
            WHERE
                todo_id = #{todoId}
        ]]>
        </select>

        <!-- (4) -->
        <select id="findAll" resultMap="todoResultMap">
        <![CDATA[
            SELECT
                todo_id,
                todo_title,
                finished,
                created_at
            FROM
                todo
        ]]>
        </select>

        <!-- (5) -->
        <insert id="create" parameterType="Todo">
        <![CDATA[
            INSERT INTO todo
            (
                todo_id,
                todo_title,
                finished,
                created_at
            )
            VALUES
            (
                #{todoId},
                #{todoTitle},
                #{finished},
                #{createdAt}
            )
        ]]>
        </insert>

        <!-- (6) -->
        <update id="update" parameterType="Todo">
        <![CDATA[
            UPDATE todo
            SET
                todo_title = #{todoTitle},
                finished = #{finished},
                created_at = #{createdAt}
            WHERE
                todo_id = #{todoId}
        ]]>
        </update>

        <!-- (7) -->
        <delete id="delete" parameterType="Todo">
        <![CDATA[
            DELETE FROM
                todo
            WHERE
                todo_id = #{todoId}
        ]]>
        </delete>

        <!-- (8) -->
        <select id="countByFinished" parameterType="Boolean"
            resultType="Long">
        <![CDATA[
            SELECT
                COUNT(*)
            FROM
                todo
            WHERE
                finished = #{finished}
        ]]>
        </select>

    </mapper>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify the fully qualified class name of the Repository interfaces (FQCN) in the \ ``namespace`` \ attribute of \ ``mapper`` \ element.
   * - | (2)
     - | Define JavaBean mapping with search result (\ ``ResultSet``\ ) in the \ ``<resultMap>`` \ element.
       | For the mapping file details, Refer \ :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessMyBatis3`\.
   * - | (3)
     - | Implement the SQL to get one record which matches with the \ ``todoId`` \(PK).
       | Specify the ID of applicable mapping definition in the \ ``resultMap`` \ attribute of \ ``<select>`` \ element.
   * - | (4)
     - | Implement the SQL to retrieve all records.
       | Specify the ID of applicable mapping definition in the \ ``resultMap`` \ attribute of \ ``<select>`` \ element.
       | Although it is not described in the application requirement, records are rearranged as the most recent TODO is displayed at the top.
   * - | (5)
     - | Implement the SQL to insert the Todo object specified in the argument.
       | Specify the parameter of the class name (FQCN or alias name) in the \ ``parameterType`` \ attribute of \ ``<insert>`` \ element.
   * - | (6)
     - | Implement the SQL to update the Todo object specified in the argument.
       | Specify the parameter of the class name (FQCN or alias name) in the \ ``parameterType`` \ attribute of \ ``<update>`` \ element.
   * - | (7)
     - | Implement the SQL to delete the Todo object specified in the argument.
       | Specify the parameter of the class name (FQCN or alias name) in the \ ``parameterType`` \ attribute of \ ``<delete>`` \ element.
   * - | (8)
     - | Implement the SQL to get all records matches with the \ ``finished`` \ status specified in the argument.

|

Since the creation of infrastructure layer using MyBatis3 has been completed, create the application layer components and services.

Once creation of the Services and application layer are completed, execute the Todo application after starting the AP server, SQL and transaction log output is as following.

.. code-block:: console
   :emphasize-lines: 2-3,6-18,20-22

    date:2016-02-17 13:18:54	thread:tomcat-http--5	X-Track:390066c43aa94b6588e5bac6a54812b2	level:TRACE	logger:o.t.gfw.web.logging.TraceLoggingInterceptor     	message:[START CONTROLLER] TodoController.list(Model)
    date:2016-02-17 13:18:54	thread:tomcat-http--5	X-Track:390066c43aa94b6588e5bac6a54812b2	level:DEBUG	logger:o.s.jdbc.datasource.DataSourceTransactionManager	message:Creating new transaction with name [todo.domain.service.todo.TodoServiceImpl.findAll]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT,readOnly; ''
    date:2016-02-17 13:18:55	thread:tomcat-http--5	X-Track:390066c43aa94b6588e5bac6a54812b2	level:DEBUG	logger:o.s.jdbc.datasource.DataSourceTransactionManager	message:Acquired Connection [net.sf.log4jdbc.ConnectionSpy@4e53de7c] for JDBC transaction
    date:2016-02-17 13:18:55	thread:tomcat-http--5	X-Track:390066c43aa94b6588e5bac6a54812b2	level:DEBUG	logger:t.domain.repository.todo.TodoRepository.findAll 	message:==>  Preparing: SELECT todo_id, todo_title, finished, created_at FROM todo 
    date:2016-02-17 13:18:55	thread:tomcat-http--5	X-Track:390066c43aa94b6588e5bac6a54812b2	level:DEBUG	logger:t.domain.repository.todo.TodoRepository.findAll 	message:==> Parameters: 
    date:2016-02-17 13:18:55	thread:tomcat-http--5	X-Track:390066c43aa94b6588e5bac6a54812b2	level:DEBUG	logger:jdbc.sqltiming                                  	message: sun.reflect.NativeMethodAccessorImpl.invoke0(NativeMethodAccessorImpl.java:-2)
    1. SELECT
                todo_id,
                todo_title,
                finished,
                created_at
            FROM
                todo {executed in 0 msec}
    date:2016-02-17 13:18:55	thread:tomcat-http--5	X-Track:390066c43aa94b6588e5bac6a54812b2	level:INFO 	logger:jdbc.resultsettable                             	message:|--------|-----------|---------|-----------|
    date:2016-02-17 13:18:55	thread:tomcat-http--5	X-Track:390066c43aa94b6588e5bac6a54812b2	level:INFO 	logger:jdbc.resultsettable                             	message:|TODO_ID |TODO_TITLE |FINISHED |CREATED_AT |
    date:2016-02-17 13:18:55	thread:tomcat-http--5	X-Track:390066c43aa94b6588e5bac6a54812b2	level:INFO 	logger:jdbc.resultsettable                             	message:|--------|-----------|---------|-----------|
    date:2016-02-17 13:18:55	thread:tomcat-http--5	X-Track:390066c43aa94b6588e5bac6a54812b2	level:INFO 	logger:jdbc.resultsettable                             	message:|--------|-----------|---------|-----------|
    date:2016-02-17 13:18:55	thread:tomcat-http--5	X-Track:390066c43aa94b6588e5bac6a54812b2	level:DEBUG	logger:t.domain.repository.todo.TodoRepository.findAll 	message:<==      Total: 0
    date:2016-02-17 13:18:55	thread:tomcat-http--5	X-Track:390066c43aa94b6588e5bac6a54812b2	level:DEBUG	logger:o.s.jdbc.datasource.DataSourceTransactionManager	message:Initiating transaction commit
    date:2016-02-17 13:18:55	thread:tomcat-http--5	X-Track:390066c43aa94b6588e5bac6a54812b2	level:DEBUG	logger:o.s.jdbc.datasource.DataSourceTransactionManager	message:Committing JDBC transaction on Connection [net.sf.log4jdbc.ConnectionSpy@4e53de7c]
    date:2016-02-17 13:18:55	thread:tomcat-http--5	X-Track:390066c43aa94b6588e5bac6a54812b2	level:DEBUG	logger:o.s.jdbc.datasource.DataSourceTransactionManager	message:Releasing JDBC Connection [net.sf.log4jdbc.ConnectionSpy@4e53de7c] after transaction
    date:2016-02-17 13:18:55	thread:tomcat-http--5	X-Track:390066c43aa94b6588e5bac6a54812b2	level:TRACE	logger:o.t.gfw.web.logging.TraceLoggingInterceptor     	message:[END CONTROLLER  ] TodoController.list(Model)-> view=todo/list, model={todoForm=todo.app.todo.TodoForm@2a075f1d, todos=[], org.springframework.validation.BindingResult.todoForm=org.springframework.validation.BeanPropertyBindingResult: 0 errors}
    date:2016-02-17 13:18:55	thread:tomcat-http--5	X-Track:390066c43aa94b6588e5bac6a54812b2	level:TRACE	logger:o.t.gfw.web.logging.TraceLoggingInterceptor     	message:[HANDLING TIME   ] TodoController.list(Model)-> 756,709,153 ns

|

.. _using_SpringDataJPA:

Creating infrastructure layer with Spring Data JPA
--------------------------------------------------------------------------------

Here, How to create RepositoryImpl of infrastructure layer using \ `Spring Data JPA <http://www.springsource.org/spring-data/jpa>`_ \ is explained.

Modification in Entity
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Configure the JPA annotation for mapping the Todo class with TODO table of Database.

.. code-block:: java
    :emphasize-lines: 6-10,12-14,18-19,26-27

    package todo.domain.model;

    import java.io.Serializable;
    import java.util.Date;

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
        private String todoId;

        private String todoTitle;

        private boolean finished;

        // (3)
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


   * - Sr. No.
     - Description
   * - | (1)
     - | Add \ ``@Entity`` \ showing that it is JPA entity and set the corresponding table name using \ ``@Table``\.
   * - | (2)
     - | Add \ ``@Id`` \ to the field corresponding to primary key column.
   * - | (3)
     - | It should be clearly specified that \ ``java.util.Date`` \ type corresponds to which of \ ``java.sql.Date``\ , \ ``java.sql.Time``\ , \ ``java.sql.Timestamp``\.
       | Here \ ``Timestamp`` \ is specified to \ ``createdAt`` \ property.


Creat TodoRepository
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create \ ``TodoRepository`` \ using Repository feature of Spring Data JPA.

Right click on the Package Explorer, select -> [New] -> [Interface] -> [New Java Interface] dialog box appears, 

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - Sr. No.
      - Item
      - Input value
    * - 1
      - Package
      - ``todo.domain.repository.todo``
    * - 2
      - Name
      - ``TodoRepository``
    * - 3
      - Extended interfaces
      - ``org.springframework.data.jpa.repository.JpaRepository<T, ID>``

and click [Finish] after entering above information.



.. code-block:: java
    :emphasize-lines: 3-5,9-10,12,13

    package todo.domain.repository.todo;

    import org.springframework.data.jpa.repository.JpaRepository;
    import org.springframework.data.jpa.repository.Query;
    import org.springframework.data.repository.query.Param;

    import todo.domain.model.Todo;

    // (1)
    public interface TodoRepository extends JpaRepository<Todo, String> {

        @Query("SELECT COUNT(t) FROM Todo t WHERE t.finished = :finished") // (2)
        long countByFinished(@Param("finished") boolean finished); // (3)

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify generics parameter of \ ``JpaRepository``\.
       | From left to right, Entity Classes (\ ``Todo``\), primary key class (\ ``String``\) is specified.
       | Since basic CRUD operations (\ ``findOne``\ , \ ``findAll``\ , \ ``save``\ , \ ``delete`` \ etc.) are defined in \ ``JpaRepository`` \ interface,
       | only \ ``countByFinished`` \ can be defined in \ ``TodoRepository``\.
   * - | (2)
     - | Specify \ ``@Query`` \ annotation to the JPQL executed at the time of calling \ ``countByFinished`` \ method
   * - | (3)
     - | Set bind variable of JPQL specified in (2) by \ ``@Param`` \ annotation.
       | Here, \ ``@Param(“finished”)`` \ is added before the method argument \ ``finished`` \ to embed the value with \ ``”:finished”`` \ of JPQL.       


Create TodoRepositoryImpl
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If Spring Data JPA is used, RepositoryImpl is automatically generated from the Repository interface.
Therefore, the creation of \ ``TodoRepositoryImpl`` \ is not required. Remove if it is created.

|

Since the creation of infrastructure layer using Spring Data JPA has been completed, create the application layer components and services.

Once creation of the Services and application layer are completed, execute the Todo application after starting the AP server, SQL and transaction log output is as following.

.. code-block:: console
   :emphasize-lines: 2-11

    date:2016-02-17 13:32:44	thread:tomcat-http--5	X-Track:7c34263e0a2143639f3ffd191b35c135	level:TRACE	logger:o.t.gfw.web.logging.TraceLoggingInterceptor     	message:[START CONTROLLER] TodoController.list(Model)
    date:2016-02-17 13:32:44	thread:tomcat-http--5	X-Track:7c34263e0a2143639f3ffd191b35c135	level:DEBUG	logger:o.h.e.transaction.spi.AbstractTransactionImpl   	message:begin
    date:2016-02-17 13:32:44	thread:tomcat-http--5	X-Track:7c34263e0a2143639f3ffd191b35c135	level:DEBUG	logger:o.h.e.transaction.internal.jdbc.JdbcTransaction 	message:initial autocommit status: false
    date:2016-02-17 13:32:45	thread:tomcat-http--5	X-Track:7c34263e0a2143639f3ffd191b35c135	level:DEBUG	logger:jdbc.sqltiming                                  	message: org.hibernate.engine.jdbc.internal.ResultSetReturnImpl.extract(ResultSetReturnImpl.java:82)
    5. /* select generatedAlias0 from Todo as generatedAlias0 */ select todo0_.todo_id as todo_id1_0_, todo0_.created_at as created_2_0_, todo0_.finished as finished3_0_, todo0_.todo_title as todo_tit4_0_ from todo todo0_ {executed in 1 msec}
    date:2016-02-17 13:32:45	thread:tomcat-http--5	X-Track:7c34263e0a2143639f3ffd191b35c135	level:INFO 	logger:jdbc.resultsettable                             	message:|--------|-----------|---------|-----------|
    date:2016-02-17 13:32:45	thread:tomcat-http--5	X-Track:7c34263e0a2143639f3ffd191b35c135	level:INFO 	logger:jdbc.resultsettable                             	message:|TODO_ID |CREATED_AT |FINISHED |TODO_TITLE |
    date:2016-02-17 13:32:45	thread:tomcat-http--5	X-Track:7c34263e0a2143639f3ffd191b35c135	level:INFO 	logger:jdbc.resultsettable                             	message:|--------|-----------|---------|-----------|
    date:2016-02-17 13:32:45	thread:tomcat-http--5	X-Track:7c34263e0a2143639f3ffd191b35c135	level:INFO 	logger:jdbc.resultsettable                             	message:|--------|-----------|---------|-----------|
    date:2016-02-17 13:32:45	thread:tomcat-http--5	X-Track:7c34263e0a2143639f3ffd191b35c135	level:DEBUG	logger:o.h.e.transaction.spi.AbstractTransactionImpl   	message:committing
    date:2016-02-17 13:32:45	thread:tomcat-http--5	X-Track:7c34263e0a2143639f3ffd191b35c135	level:DEBUG	logger:o.h.e.transaction.internal.jdbc.JdbcTransaction 	message:committed JDBC Connection
    date:2016-02-17 13:32:45	thread:tomcat-http--5	X-Track:7c34263e0a2143639f3ffd191b35c135	level:TRACE	logger:o.t.gfw.web.logging.TraceLoggingInterceptor     	message:[END CONTROLLER  ] TodoController.list(Model)-> view=todo/list, model={todoForm=todo.app.todo.TodoForm@5dac2c75, todos=[], org.springframework.validation.BindingResult.todoForm=org.springframework.validation.BeanPropertyBindingResult: 0 errors}
    date:2016-02-17 13:32:45	thread:tomcat-http--5	X-Track:7c34263e0a2143639f3ffd191b35c135	level:TRACE	logger:o.t.gfw.web.logging.TraceLoggingInterceptor     	message:[HANDLING TIME   ] TodoController.list(Model)-> 320,129,237 ns

|

In the end...
================================================================================
In this tutorial, following contents have been learnt.


* How to develop basic applications by TERASOLUNA Server Framework for Java (5.x)

* How to build Maven as well as STS (Eclipse) project

* Way of development using application layering of TERASOLUNA Server Framework for Java (5.x).

 * Implementation of domain layer with POJO(+ Spring)
 * Implementation of application layer with the use of JSP tag libraries and POJO(+ Spring MVC)
 * Development of Infrastructure layer with the use of MyBatis3
 * Development of Infrastructure layer with the use of Spring Data JPA
 * Development of Infrastructure layer without use of O/R Mapper

The following improvement can be done in the TODO management application.
 As a learning challenge of the application improvement refer the appropriate description of the guidelines.

* To externalize the property (Maximum number of uncompleted TODO) -> :doc:`../ArchitectureInDetail/GeneralFuncDetail/PropertyManagement`
* To externalize the messages -> :doc:`../ArchitectureInDetail/WebApplicationDetail/MessageManagement`
* To add pagination function -> :doc:`../ArchitectureInDetail/WebApplicationDetail/Pagination`
* To add exception handling -> :doc:`../ArchitectureInDetail/WebApplicationDetail/ExceptionHandling`
* To add double submit protection (Support the transaction token check) -> :doc:`../ArchitectureInDetail/WebApplicationDetail/DoubleSubmitProtection`
* To change how to get the system date time -> :doc:`../ArchitectureInDetail/GeneralFuncDetail/SystemDate`

|

Appendix
================================================================================

.. _TutorialTodoAppendixExpoundConfigurations:

Description of the configuration file
--------------------------------------------------------------------------------

Description of the configuration files are done for giving an understanding of what type of settings are required to run an application.
Settings that are not used in created Todo tutorial application are ignored here.

web.xml
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In \ :file:`web.xml`\, the settings are done for deploying the Todo application as a Web application.

Following settings are done in created blank project :file:`src/main/webapp/WEB-INF/web.xml`\.

.. code-block:: xml
    :emphasize-lines: 2, 8, 25, 79, 95, 106, 122


    <?xml version="1.0" encoding="UTF-8"?>
    <!-- (1) -->
    <web-app xmlns="http://java.sun.com/xml/ns/javaee"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
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
                classpath*:META-INF/spring/spring-security.xml
            </param-value>
        </context-param>

        <listener>
            <listener-class>org.terasoluna.gfw.web.logging.HttpSessionEventLoggingListener</listener-class>
        </listener>

        <!-- (3) -->
        <filter>
            <filter-name>MDCClearFilter</filter-name>
            <filter-class>org.terasoluna.gfw.web.logging.mdc.MDCClearFilter</filter-class>
        </filter>
        <filter-mapping>
            <filter-name>MDCClearFilter</filter-name>
            <url-pattern>/*</url-pattern>
        </filter-mapping>

        <filter>
            <filter-name>exceptionLoggingFilter</filter-name>
            <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
        </filter>
        <filter-mapping>
            <filter-name>exceptionLoggingFilter</filter-name>
            <url-pattern>/*</url-pattern>
        </filter-mapping>

        <filter>
            <filter-name>XTrackMDCPutFilter</filter-name>
            <filter-class>org.terasoluna.gfw.web.logging.mdc.XTrackMDCPutFilter</filter-class>
        </filter>
        <filter-mapping>
            <filter-name>XTrackMDCPutFilter</filter-name>
            <url-pattern>/*</url-pattern>
        </filter-mapping>

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

        <filter>
            <filter-name>springSecurityFilterChain</filter-name>
            <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
        </filter>
        <filter-mapping>
            <filter-name>springSecurityFilterChain</filter-name>
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

        <!-- (6) -->
        <error-page>
            <error-code>500</error-code>
            <location>/WEB-INF/views/common/error/systemError.jsp</location>
        </error-page>

        <error-page>
            <error-code>404</error-code>
            <location>/WEB-INF/views/common/error/resourceNotFoundError.jsp</location>
        </error-page>

        <error-page>
            <exception-type>java.lang.Exception</exception-type>
            <location>/WEB-INF/views/common/error/unhandledSystemError.html</location>
        </error-page>

        <!-- (7) -->
        <session-config>
            <!-- 30min -->
            <session-timeout>30</session-timeout>
            <cookie-config>
                <http-only>true</http-only>
                <!-- <secure>true</secure> -->
            </cookie-config>
            <tracking-mode>COOKIE</tracking-mode>
        </session-config>

    </web-app>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90
   :class: longtable

   * - Sr. No.
     - Description
   * - | (1)
     - | Declaration to use Servlet3.0.
   * - | (2)
     - | Definition of servlet context listener.

       In the blank project,

       * \ ``ContextLoaderListener`` \ is used to create an \ ``ApplicationContext`` \  for entire application
       * \ ``HttpSessionEventLoggingListener`` \ is used to log output the HTTPSession operations

       are configured.
   * - | (3)
     - | Definition of servlet filter.

       In the blank project,

       * Servlet filter provided by the common library
       * \ ``CharacterEncodingFilter`` \ for specifing the character encoding provided by the Spring Framework
       * Servlet filter for authentication and authorization provided by the Spring Security

       are configured.
   * - | (4)
     - | Definition of DispatcherServlet that is the entry point of Spring MVC.
       |
       | \ ``ApplicationContext`` \ used in the DispatcherServlet is the child of \ ``ApplicationContext`` \ created in step (2).
       | It is also possible to use components loaded in (2) by setting \ ``ApplicationContext`` \ created at step (2) as a parent.
   * - | (5)
     - | Common definition of JSP.

       In the blank project,

       * EL expression can be used in JSP
       * UTF-8 as the JSP page encoding
       * Scripting can be used in JSP
       * Included \ :file:`/WEB-INF/views/common/include.jsp` \ at the beginning of each JSP

       are configured.
   * - | (6)
     - | Definition of error page.

       In the blank project,

       * Respond HTTP status code \ ``404`` \ or \ ``500`` \ to the servlet container
       * Exception notification to the servlet container

       related transition destination are defined.
   * - | (7)
     - | Definition of session management.

       In the blank project,

       * 30 minutes as a session time-out

       are defined.

.. raw:: latex

   \newpage

|

Common JSP
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Tag library settings applied to all JSP are done in Include JSP.

Following settings are done in created blank project \ :file:`src/main/webapp/WEB-INF/views/common/include.jsp`\.

.. code-block:: jsp
    :emphasize-lines: 1, 3, 6, 9, 11

    <!-- (1) -->
    <%@ page session="false"%>
    <!-- (2) -->
    <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
    <%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt"%>
    <!-- (3) -->
    <%@ taglib uri="http://www.springframework.org/tags" prefix="spring"%>
    <%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>
    <!-- (4) -->
    <%@ taglib uri="http://www.springframework.org/security/tags" prefix="sec"%>
    <!-- (5) -->
    <%@ taglib uri="http://terasoluna.org/tags" prefix="t"%>
    <%@ taglib uri="http://terasoluna.org/functions" prefix="f"%>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Definition to avoid session creation while JSP execution.
   * - | (2)
     - | Definition standard tag library.
   * - | (3)
     - | Definition tag library for Spring MVC.
   * - | (4)
     - | Definition tag library for Spring Security.(However, it is not used in this tutorial)
   * - | (5)
     - | Definition EL function and tag library provided in common library.

|

Bean definition file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Following Bean definition files and property files are generated in created blank project.

* :file:`src/main/resources/META-INF/spring/applicationContext.xml`
* :file:`src/main/resources/META-INF/spring/todo-domain.xml`
* :file:`src/main/resources/META-INF/spring/todo-infra.xml`
* :file:`src/main/resources/META-INF/spring/todo-infra.properties`
* :file:`src/main/resources/META-INF/spring/todo-env.xml`
* :file:`src/main/resources/META-INF/spring/spring-mvc.xml`
* :file:`src/main/resources/META-INF/spring/spring-security.xml`

.. note::

    The \ ``todo-infra.properties`` \ and \ ``todo-env.xml`` \ are not created while creating blank project that does not dependent on the O/R Mapper.

.. note::

    In this guideline, it is recommended to split Bean definition file for each role (layer).

    By this way, what is defined in which file can be easily understood and improves the ease of maintenance.
    It is not effective in case of small application like tutorial but  more effective in case of large scale application.

|

applicationContext.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Perform entire Todo application related settings in the \ :file:`applicationContext.xml`\.

| Following settings are done in created blank project \ :file:`src/main/resources/META-INF/spring/applicationContext.xml`\.
| In addition, a description of the components that are not used in the tutorial are omitted.

.. code-block:: xml
    :emphasize-lines: 10-11, 15-17, 19-20

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
        ">

        <!-- (1) -->
        <import resource="classpath:/META-INF/spring/todo-domain.xml" />

        <bean id="passwordEncoder" class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder" />

        <!-- (2) -->
        <context:property-placeholder
            location="classpath*:/META-INF/spring/*.properties" />

        <!-- (3) -->
        <bean id="beanMapper" class="org.dozer.spring.DozerBeanMapperFactoryBean">
            <property name="mappingFiles"
                value="classpath*:/META-INF/dozer/**/*-mapping.xml" />
        </bean>

        <!-- Message -->
        <bean id="messageSource"
            class="org.springframework.context.support.ResourceBundleMessageSource">
            <property name="basenames">
                <list>
                    <value>i18n/application-messages</value>
                </list>
            </property>
        </bean>

        <!-- Exception Code Resolver. -->
        <bean id="exceptionCodeResolver"
            class="org.terasoluna.gfw.common.exception.SimpleMappingExceptionCodeResolver">
            <!-- Setting and Customization by project. -->
            <property name="exceptionMappings">
                <map>
                    <entry key="ResourceNotFoundException" value="e.xx.fw.5001" />
                    <entry key="InvalidTransactionTokenException" value="e.xx.fw.7001" />
                    <entry key="BusinessException" value="e.xx.fw.8001" />
                    <entry key=".DataAccessException" value="e.xx.fw.9002" />
                </map>
            </property>
            <property name="defaultExceptionCode" value="e.xx.fw.9001" />
        </bean>

        <!-- Exception Logger. -->
        <bean id="exceptionLogger"
            class="org.terasoluna.gfw.common.exception.ExceptionLogger">
            <property name="exceptionCodeResolver" ref="exceptionCodeResolver" />
        </bean>

        <!-- Filter. -->
        <bean id="exceptionLoggingFilter"
            class="org.terasoluna.gfw.web.exception.ExceptionLoggingFilter" >
            <property name="exceptionLogger" ref="exceptionLogger" />
        </bean>

    </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Import Bean definition file related to domain layer.
   * - | (2)
     - | Loading configuration of the property file.
       | Load all property files stored under the \ ``src/main/resources/META-INF/spring``\.
       | Using this setting, it is possible to insert property file value in  \ ``${propertyName}`` \ format in Bean definition file and to inject using \ ``@Value("${propertyName}")`` \ in Java class.
   * - | (3)
     - | Define Dozer Mapper of Bean conversion library.
       | For the mapping file, refer to `Dozer manual <http://dozer.sourceforge.net/documentation/mappings.html>`_.

.. tip::

    By inserting following type of check-mark in [Configure Namespace] tab of the editor,
    selected XML schema gets enabled and possible to supplement the input using Ctrl+Space at the time of XML editing.

    It is recommended to select xsd file without version in [Namespace Versions].
    By selecting the xsd file without version, always the latest xsd is used included in the jar
    therefore no need to concern about the Spring version up.

     .. figure:: ./images/image021.jpg
        :width: 90%

     .. figure:: ./images/image023.png
        :width: 60%

|

todo-domain.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Perform the domain layer related settings of the Todo application in \ :file:`todo-domain.xml`\.

| Following settings are done in created blank project \ :file:`src/main/resources/META-INF/spring/todo-domain.xml`\.
| In addition, a description of the components that are not used in the tutorial are omitted.

.. code-block:: xml
    :emphasize-lines: 12-13, 16-17

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xsi:schemaLocation="
            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
        ">

        <!-- (1) -->
        <import resource="classpath:META-INF/spring/todo-infra.xml" />
        <import resource="classpath*:META-INF/spring/**/*-codelist.xml" />

        <!-- (2) -->
        <context:component-scan base-package="todo.domain" />

        <!-- AOP. -->
        <bean id="resultMessagesLoggingInterceptor"
            class="org.terasoluna.gfw.common.exception.ResultMessagesLoggingInterceptor">
            <property name="exceptionLogger" ref="exceptionLogger" />
        </bean>
        <aop:config>
            <aop:advisor advice-ref="resultMessagesLoggingInterceptor"
                pointcut="@within(org.springframework.stereotype.Service)" />
        </aop:config>

    </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Import Bean definition file related to infrastructure layer.
   * - | (2)
     - | Components under todo.domain package are target of component scan.
       | Thus, by attaching annotations like ``@Repository`` , ``@Service`` etc to the class under todo.domain package, it get registered as Bean of Spring Framework.
       | It is possible to use registered classes (Bean) by doing DI in Controller and Service class.

.. note::

    The \ ``<tx:annotation-driven>`` \ tag is set in order to enable the transaction management by \ ``@Transactional`` \ annotation 
    while creating O/R Mapper dependent blank project.

     .. code-block:: xml
        :emphasize-lines: 9-10, 12-13

         <tx:annotation-driven />

|

todo-infra.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Perform infrastructure layer related settings of the TODO application in \ :file:`todo-infra.xml`\.

Following settings are done 
in created blank project \ :file:`src/main/resources/META-INF/spring/todo-infra.xml`\.

Since configuration of \ :file:`todo-infra.xml` \ is significantly different by the infrastructure layer,
description of each blank project done.
You may skip the explanation other than the created blank project.


todo-infra.xml if creating blank project that does not dependent on the O/R Mapper
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Empty definition file is created as follows while creating blank project that does not dependent on the O/R Mapper.

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        ">

    </beans>

todo-infra.xml of blank project created for MyBatis3
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

The following settings are done in created MyBatis3 project.

.. code-block:: xml
   :emphasize-lines: 10-11, 13-15, 16-17, 18-19, 22-24

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:mybatis="http://mybatis.org/schema/mybatis-spring"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://mybatis.org/schema/mybatis-spring http://mybatis.org/schema/mybatis-spring.xsd
        ">

         <!-- (1) -->
        <import resource="classpath:/META-INF/spring/todo-env.xml" />

         <!-- (2) -->
        <!-- define the SqlSessionFactory -->
        <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
             <!-- (3) -->
            <property name="dataSource" ref="dataSource" />
             <!-- (4) -->
            <property name="configLocation" value="classpath:/META-INF/mybatis/mybatis-config.xml" />
        </bean>

         <!-- (5) -->
        <!-- scan for Mappers -->
        <mybatis:scan base-package="todo.domain.repository" />

    </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Import the Bean definition file that defines the environment dependent components(Such as DataSource and transaction manager).
   * - | (2)
     - | The bean definition of \ ``SqlSessionFactoryBean`` \ as a component for generating a \ ``SqlSessionFactory``\.
   * - | (3)
     - | Specify the bean configured DataSource in the \ ``dataSource`` \ property.
       |
       | The connection is retrieved from the DataSource specified here by executing the SQL in MyBatis3 process.
   * - | (4)
     - | Specify the path of MyBatis configuration file in the \ ``configLocation`` \ property.
       |
       | The specified file loaded while generating a \ ``SqlSessionFactory``\
   * - | (5)
     - | Define \ ``<mybatis:scan>`` \ for scanning the Mapper interface,
       | Specify the base package in the \ ``base-package`` \ attribute that contains the Mapper interface.
       |
       | Scanned the Mapper interface that stored under the specified package and 
       | automatically generated the thread-safe Mapper object (Proxy object of Mapper interface).

.. note::

    \ :file:`mybatis-config.xml` \ is a configuration file that performs operational setting of MyBatis3.

    The following settings are done by default in the blank project.

     .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>
        <!DOCTYPE configuration
            PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-config.dtd">
        <configuration>

            <!-- See http://mybatis.github.io/mybatis-3/configuration.html#settings -->
            <settings>
                <setting name="mapUnderscoreToCamelCase" value="true" />
                <setting name="lazyLoadingEnabled" value="true" />
                <setting name="aggressiveLazyLoading" value="false" />
                <setting name="defaultFetchSize" value="100" />
        <!--
                <setting name="defaultExecutorType" value="REUSE" />
                <setting name="jdbcTypeForNull" value="NULL" />
                <setting name="localCacheScope" value="STATEMENT" />
        -->
            </settings>

            <typeAliases>
                <package name="todo.domain.model" />
                <package name="todo.domain.repository" />
        <!--
                <package name="todo.infra.mybatis.typehandler" />
        -->
            </typeAliases>

            <typeHandlers>
        <!--
                <package name="todo.infra.mybatis.typehandler" />
        -->
            </typeHandlers>

        </configuration>

todo-infra.xml of blank project created for JPA
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

The following settings are done in created JPA blank project.

.. code-block:: xml
    :emphasize-lines: 12-13, 15-16, 18-20, 25-27, 28-29, 32-33

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:jpa="http://www.springframework.org/schema/data/jpa"
        xmlns:util="http://www.springframework.org/schema/util"
        xsi:schemaLocation="
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd
            http://www.springframework.org/schema/data/jpa http://www.springframework.org/schema/data/jpa/spring-jpa.xsd
        ">

        <!-- (1) -->
        <import resource="classpath:/META-INF/spring/todo-env.xml" />

        <!-- (2) -->
        <jpa:repositories base-package="todo.domain.repository"></jpa:repositories>

        <!-- (3) -->
        <bean id="jpaVendorAdapter"
            class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
            <property name="showSql" value="false" />
            <property name="database" value="${database}" />
        </bean>

        <!-- (4) -->
        <bean id="entityManagerFactory"
            class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
            <!-- (5) -->
            <property name="packagesToScan" value="todo.domain.model" />
            <property name="dataSource" ref="dataSource" />
            <property name="jpaVendorAdapter" ref="jpaVendorAdapter" />
            <!-- (6) -->
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

    </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Import the Bean definition file that defines the environment dependent components(Such as DataSource and transaction manager).
   * - | (2)
     - | Automatically generate the implementation class from Repository interface using the Spring Data JPA.
       | Specify the package in \ ``base-package`` \ attribute of \ ``<jpa:repository>`` \ tag that contains the Repository.
   * - | (3)
     - | Configure JPA implementation vendor.
       | In order to use Hibernate, \ ``HibernateJpaVendorAdapter`` \ is defined as a JPA implementation.
   * - | (4)
     - | Define the \ ``EntityManager``\.
   * - | (5)
     - | Specify the package name that treated as a JPA entity classes.
   * - | (6)
     - | Perform the Hibernate related detail settings.

|

todo-infra.properties
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Perform environment dependent infrastructure layer configuration of Todo application in the \ :file:`todo-infra.properties`\.

The \ :file:`todo-infra.properties` \ is not created while creating blank project that does not dependent on the O/R Mapper.

Following settings are done in created blank project \ :file:`src/main/resources/META-INF/spring/todo-infra.properties`\.

.. code-block:: properties
    :emphasize-lines: 1, 7

    # (1)
    database=H2
    database.url=jdbc:h2:mem:todo;DB_CLOSE_DELAY=-1
    database.username=sa
    database.password=
    database.driverClassName=org.h2.Driver
    # (2)
    # connection pool
    cp.maxActive=96
    cp.maxIdle=16
    cp.minIdle=0
    cp.maxWait=60000

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr. No.
     - Description
   * - | (1)
     - | Database related settings.
       | In this tutorial, H2 Database is used to save the setup time of the database.
   * - | (2)
     - | Connection pool related settings.


.. note::

    The configuration value of this is referred from \ :file:`todo-env.xml`\.

|

todo-env.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

In the \ :file:`todo-env.xml` \ file, define such components which are differing depending upon the environment.

Following settings are done in created blank project \ ``src/main/resources/META-INF/spring/todo-env.xml``\.

Here, the file stored in the blank project for MyBatis3 is described as an example.
Furthermore, the \ :file:`todo-env.xml` \ is not created while creating blank project that does not access the database.

.. code-block:: xml
    :emphasize-lines: 12, 27, 44

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:jdbc="http://www.springframework.org/schema/jdbc"
        xsi:schemaLocation="
            http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        ">

        <bean id="dateFactory" class="org.terasoluna.gfw.common.date.jodatime.DefaultJodaTimeDateFactory" />

        <!-- (1) -->
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


        <!-- (2) -->
        <bean id="dataSource" class="net.sf.log4jdbc.Log4jdbcProxyDataSource">
            <constructor-arg index="0" ref="realDataSource" />
        </bean>

        <jdbc:initialize-database data-source="dataSource"
            ignore-failures="ALL">
            <jdbc:script location="classpath:/database/${database}-schema.sql" encoding="UTF-8" />
            <jdbc:script location="classpath:/database/${database}-dataload.sql" encoding="UTF-8" />
        </jdbc:initialize-database>

        <!--  REMOVE THIS LINE IF YOU USE JPA
        <bean id="transactionManager"
            class="org.springframework.orm.jpa.JpaTransactionManager">
            <property name="entityManagerFactory" ref="entityManagerFactory" />
        </bean>
              REMOVE THIS LINE IF YOU USE JPA  -->
        <!-- (3) -->
        <bean id="transactionManager"
            class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <property name="dataSource" ref="dataSource" />
            <property name="rollbackOnCommitFailure" value="true" />
        </bean>
    </beans>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Setting of the actual DataSource.
   * - | (2)
     - | Setting of the DataSource.
       | Specify the DataSource that having JDBC-related logging feature.
       | By use of \ ``net.sf.log4jdbc.Log4jdbcProxyDataSource`` \, you can output JDBC related log such as SQL log which could be very useful for debugging.
   * - | (3)
     - | Setting of the Transaction Manager.
       | Specify the \ ``transactionManager`` \ in id attribute.
       | In case of specifing different name, it is necessary to specify the transaction manager name in \ ``<tx:annotation-driven>`` \ tag also.
       |
       | In the blank project, transaction controlling class (\ ``org.springframework.jdbc.datasource.DataSourceTransactionManager``\) is configured using JDBC API.

.. note::

    If blank project created for JPA, Transaction Manager, 
    transaction controlling class (\ ``org.springframework.orm.jpa.JpaTransactionManager``\) is configured using JPA API.

     .. code-block:: xml

        <bean id="transactionManager"
            class="org.springframework.orm.jpa.JpaTransactionManager">
            <property name="entityManagerFactory" ref="entityManagerFactory" />
        </bean>

|

spring-mvc.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The Spring MVC related definitions are done in \ :file:`spring-mvc.xml`\.

| Following settings are done in created blank project \ :file:`src/main/resources/META-INF/spring/spring-mvc.xml`\.
| In addition, a description of the components that are not used in the tutorial are omitted.

.. code-block:: xml
    :emphasize-lines: 15, 19, 33, 36, 42, 76

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

        <!-- (1) -->
        <context:property-placeholder
            location="classpath*:/META-INF/spring/*.properties" />

        <!-- (2) -->
        <mvc:annotation-driven>
            <mvc:argument-resolvers>
                <bean
                    class="org.springframework.data.web.PageableHandlerMethodArgumentResolver" />
                <bean
                    class="org.springframework.security.web.method.annotation.AuthenticationPrincipalArgumentResolver" />
            </mvc:argument-resolvers>
            <!-- workaround to CVE-2016-5007. -->
            <mvc:path-matching path-matcher="pathMatcher" />
        </mvc:annotation-driven>

        <mvc:default-servlet-handler />

        <!-- (3) -->
        <context:component-scan base-package="todo.app" />

        <!-- (4) -->
        <mvc:resources mapping="/resources/**"
            location="/resources/,classpath:META-INF/resources/"
            cache-period="#{60 * 60}" />

        <mvc:interceptors>
            <!-- (5) -->
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

        <!-- (6) -->
        <!-- Settings View Resolver. -->
        <mvc:view-resolvers>
            <mvc:jsp prefix="/WEB-INF/views/" />
        </mvc:view-resolvers>

        <bean id="requestDataValueProcessor"
            class="org.terasoluna.gfw.web.mvc.support.CompositeRequestDataValueProcessor">
            <constructor-arg>
                <util:list>
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
   :class: longtable

   * - Sr. No.
     - Description
   * - | (1)
     - | Configuration of reading property file done

       | Load all property files stored under the src/main/resources/META-INF/spring.
       | Using this setting, it is possible to insert property file value in  \ ``${propertyName}`` \ format in Bean definition file and to inject using \ ``@Value("${propertyName}")`` \ in Java class.
   * - | (2)
     - | Carry out annotation based default settings of Spring MVC.
   * - | (3)
     - | Components under todo.app package that holds classes of application layer are made target of component-scan.
   * - | (4)
     - | Carry out the settings for accessing the static resource (css, images, js etc.).

       | Set URL path to \ ``mapping`` \ attribute and physical path to \ ``location`` \ attribute.
       | In case of this setting, when there is a request for \ ``<contextPath>/rerources/app/css/styles.css``\, the \ ``WEB-INF/resources/app/css/styles.css`` \ is searched. If not found, \ ``META-INF/resources/app/css/styles.css`` \ is searched in classpath (\ ``src/main/resources`` \ and jar). 
       | If \ ``styles.css`` \ is not stored anywhere, 404 error is returned.

       | Here, cache period (3600 seconds = 60 minutes) of static resources is set in \ ``cache-period`` \ attribute.
       | \ ``cache-period="3600"`` \ is also correct, however, in order to demonstrate that it is 60 minutes, it is better to write as \ ``cache-period="#{60 * 60}"`` \ which uses `SpEL <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/expressions.html#expressions-beandef-xml-based>`_.
   * - | (5)
     - | Set interceptor that outputs trace log of controller processing. 
       | Set so that it excludes the path under\ ``/resources`` \ from mapping.
   * - | (6)
     - | Carry out the settings of \ ``ViewResolver``\. 
       | Using these settings, for example, when view name \ ``"hello"`` \ is returned from controller, \ ``/WEB-INF/views/hello.jsp`` \ is executed.

       .. tip::

           \ ``<mvc:view-resolvers>`` \ element is a XML element that added from Spring Framework 4.1
           By using \ ``<mvc:view-resolvers>`` \ element, it is possible to define \ ``ViewResolver`` \ simply.

           The definition example of using the conventional \ ``<bean>`` \ element is shown below.

            .. code-block:: xml

               <bean id="viewResolver"
                   class="org.springframework.web.servlet.view.InternalResourceViewResolver">
                   <property name="prefix" value="/WEB-INF/views/" />
                   <property name="suffix" value=".jsp" />
               </bean>

.. raw:: latex

   \newpage

.. note::

    In blank project created for JPA, as a \ ``<mvc:interceptors>`` \ definition,
    \ ``OpenEntityManagerInViewInterceptor`` \ definition is in enable state.

     .. code-block:: xml

        <mvc:interceptor>
            <mvc:mapping path="/**" />
            <mvc:exclude-mapping path="/resources/**" />
            <mvc:exclude-mapping path="/**/*.html" />
            <bean
                class="org.springframework.orm.jpa.support.OpenEntityManagerInViewInterceptor" />
        </mvc:interceptor>

    \ ``OpenEntityManagerInViewInterceptor`` \ is a Interceptor that performs start and end of the life cycle of the \ ``EntityManager``\.
    By adding this setting, the application layer (Controller or View class) Lazy Load is supported.

|

spring-security.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The Spring Security related definitions are done in \ :file:`spring-security.xml`\.

| Following settings are done in created blank project \ :file:`src/main/resources/META-INF/spring/spring-security.xml`\.
| In addition, a description of the Spring Security configuration file is omitted in the tutorial. For the Spring Security configuration file, Refer [:doc:`./TutorialSecurity`].

.. code-block:: xml

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
            <sec:form-login/>
            <sec:logout/>
            <sec:access-denied-handler ref="accessDeniedHandler"/>
            <sec:custom-filter ref="userIdMDCPutFilter" after="ANONYMOUS_FILTER"/>
            <sec:session-management />
        </sec:http>

        <sec:authentication-manager />

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

|

logback.xml
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The log output related definitions are done in \ :file:`logback.xml`\.

| Following settings are done in created blank project \ :file:`src/main/resources/logback.xml`\.
| In addition, a description of the log settings that are not used in the tutorial are omitted.

.. code-block:: xml
    :emphasize-lines: 4, 36, 45

    <?xml version="1.0" encoding="UTF-8"?>
    <configuration>

        <!-- (1) -->
        <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern><![CDATA[date:%d{yyyy-MM-dd HH:mm:ss}\tthread:%thread\tX-Track:%X{X-Track}\tlevel:%-5level\tlogger:%-48logger{48}\tmessage:%msg%n]]></pattern>
            </encoder>
        </appender>

        <appender name="APPLICATION_LOG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <file>${app.log.dir:-log}/todo-application.log</file>
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>${app.log.dir:-log}/todo-application-%d{yyyyMMdd}.log</fileNamePattern>
                <maxHistory>7</maxHistory>
            </rollingPolicy>
            <encoder>
                <charset>UTF-8</charset>
                <pattern><![CDATA[date:%d{yyyy-MM-dd HH:mm:ss}\tthread:%thread\tX-Track:%X{X-Track}\tlevel:%-5level\tlogger:%-48logger{48}\tmessage:%msg%n]]></pattern>
            </encoder>
        </appender>

        <appender name="MONITORING_LOG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <file>${app.log.dir:-log}/todo-monitoring.log</file>
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>${app.log.dir:-log}/todo-monitoring-%d{yyyyMMdd}.log</fileNamePattern>
                <maxHistory>7</maxHistory>
            </rollingPolicy>
            <encoder>
                <charset>UTF-8</charset>
                <pattern><![CDATA[date:%d{yyyy-MM-dd HH:mm:ss}\tX-Track:%X{X-Track}\tlevel:%-5level\tmessage:%msg%n]]></pattern>
            </encoder>
        </appender>

        <!-- Application Loggers -->
        <!-- (2) -->
        <logger name="todo">
            <level value="debug" />
        </logger>

        <!-- TERASOLUNA -->
        <logger name="org.terasoluna.gfw">
            <level value="info" />
        </logger>
        <!-- (3) -->
        <logger name="org.terasoluna.gfw.web.logging.TraceLoggingInterceptor">
            <level value="trace" />
        </logger>
        <logger name="org.terasoluna.gfw.common.exception.ExceptionLogger">
            <level value="info" />
        </logger>
        <logger name="org.terasoluna.gfw.common.exception.ExceptionLogger.Monitoring" additivity="false">
            <level value="error" />
            <appender-ref ref="MONITORING_LOG_FILE" />
        </logger>

        <!-- 3rdparty Loggers -->
        <logger name="org.springframework">
            <level value="warn" />
        </logger>

        <logger name="org.springframework.web.servlet">
            <level value="info" />
        </logger>

        <!--  REMOVE THIS LINE IF YOU USE JPA
        <logger name="org.hibernate.engine.transaction">
            <level value="debug" />
        </logger>
              REMOVE THIS LINE IF YOU USE JPA  -->
        <!--  REMOVE THIS LINE IF YOU USE MyBatis3
        <logger name="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <level value="debug" />
        </logger>
              REMOVE THIS LINE IF YOU USE MyBatis3  -->

        <logger name="jdbc.sqltiming">
            <level value="debug" />
        </logger>

        <!-- only for development -->
        <logger name="jdbc.resultsettable">
            <level value="debug" />
        </logger>

        <root level="warn">
            <appender-ref ref="STDOUT" />
            <appender-ref ref="APPLICATION_LOG_FILE" />
        </root>

    </configuration>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr. No.
     - Description
   * - | (1)
     - | Set appender that outputs the log in standard output.
   * - | (2)
     - | Set so that log of debug level and above is output under todo package.
   * - | (3)
     - | Set log level to ‘trace’ for TraceLoggingInterceptor which is defined in spring-mvc.xml.


.. note::

    If you create a blank project that uses the O/R Mapper, the logger to output transaction control-related log has been in enable state.

    * Blank project for the JPA

     .. code-block:: xml

        <logger name="org.hibernate.engine.transaction">
            <level value="debug" />
        </logger>

    * Blank project for the MyBatis3

     .. code-block:: xml

        <logger name="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <level value="debug" />
        </logger>

.. raw:: latex

   \newpage

