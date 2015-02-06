Create New Project From Blank
================================================================================

Method to create a new project from blank is explained below.

* Prerequisites

  * Operational Spring Tool Suite.
  * Maven should be executable in command line.
  * Internet connectivity.

The contents in this chapter have been validated on the following versions.

.. tabularcolumns:: |p{0.75\linewidth}|p{0.25\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 75 25

   * - Product
     - Version
   * - JDK
     - 1.7.0\_40
   * - Spring Tool Suite (STS)
     - 3.4.0
   * - Maven
     - 3.0.4

.. _CreateProjectFromBlank_create-new-project:


.. note::

  If proxy server is required to connect to the internet,
  STS proxy settings and `Maven proxy settings <http://maven.apache.org/guides/mini/guide-proxies.html>`_\  are required.

Creating a new project
--------------------------------------------------------------------------------

#. Move to the target folder using command prompt.

    .. code-block:: text
    
        cd C:\work

#. Create a project template using Maven archetype.
    
    .. code-block:: console
    
      mvn archetype:generate -B^
       -DarchetypeCatalog=http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases^
       -DarchetypeGroupId=org.terasoluna.gfw.blank^
       -DarchetypeArtifactId=terasoluna-gfw-web-blank-archetype^
       -DarchetypeVersion=1.0.1.RELEASE^
       -DgroupId=example.first.application^
       -DartifactId=example-first-application^
       -Dversion=1.0.0-SNAPSHOT


    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 20 80
    
       * - Parameter
         - Description
       * - | \-B
         - | batch mode (skips interaction)
       * - | \-DarchetypeCatalog
         - | Creates project using Maven archetype. Specifies TERASOLUNA Server Framework for Java (5.x) repository.
       * - | \-DarchetypeGroupId
         - | terasoluna.gfw blank project GroupID is fixed to org.terasoluna.gfw.blank
       * - | \-DarchetypeArtifactId
         - | terasoluna-gfw-web-blank-archetype = when creating project without using JPA, MyBatis2
           | terasoluna-gfw-web-blank-jpa-archetype = when creating project using JPA
           | terasoluna-gfw-web-blank-mybatis2-archetype = when creating project using MyBatis2
       * - | \-DarchetypeVersion
         - | Specifies terasoluna.gfw version.
       * - | \-DgroupId
         - | Specifies the groupId of the created project.
       * - | \-DartifactId
         - | Specifies the artifactId of the created project.
       * - | \-Dversion
         - | Specifies the version of the created project.
    

.. _CreateProjectFromBlank_STS-import-project:

3. Import the project in Spring Tool Suite.

    Specify the project created using Maven archetype in [STS] -> [File] -> [Import] -> [Maven] -> [Existing Maven Projects] ->[ Browse...]. -> Confirm that the check box of one pom.xml is checked and click [Finish]
  
    The status will be as given below.
  
    .. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlank_import_blank_project.png
       :alt: import blank project
       :width: 100%


#. Add the created project to the Application Server.

    Here, VMware vFabric tc Server Developer Edition v2.9 which is provided with STS is used by default.
  
    Right click on [VMware vFabric tc Server Developer Edition v2.9] -> [Add and Remove] -> select created project and [Add] -> [Finish]
  
    The status will be as given below.
  
    .. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlank_add_server_blank_project.png
       :alt: add server blank project
       :width: 100%


#. Launch the Application Server.

    Launch by clicking Start button of server. If the following is displayed on Console, it can be considered that server is launched successfully.
    
    .. code-block:: console
    
      FrameworkServlet 'appServlet': initialization completed
  
    As for the following example, log is output on Console; however "\ ``FrameworkServlet 'appServlet': initialization completed``\ " is displayed on the line above the line of red characters (It is not displayed on screen capture).
  
    .. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlank_server_start_blank_project.png
       :alt: server start blank project
       :width: 100%


#. Access the launched application.

    Access http://localhost:8080/example-first-application/ on browser.
  
    The screen given below is displayed.
  
    .. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlank_access_blank_project.png
       :alt: access blank project
       :width: 50%
  
    If "Hello world!" is displayed, it can be considered that a new project is created successfully.
    Necessary functions should be added as per the guidelines.


Creating simple Echo project
--------------------------------------------------------------------------------

The steps are basically same as \ :ref:`first-application-create-an-echo-application`\  explained in \ :doc:`../Overview/FirstApplication`\, so source is skipped.

In \ :doc:`../Overview/FirstApplication`\, \ ``<context:component-scan base-package="com.example.helloworld" />``\  is set in spring-mvc.xml; however,
when created from Blank Project, it is set as \ ``<context:component-scan base-package="example.first.application.app" />``\ .

\ ``EchoController``\  should be created in \ ``example.first.application.app.echo``\  package.

.. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlank_echo_input_blank_project.png
   :alt: echo input blank project
   :width: 50%

.. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlank_echo_output_blank_project.png
   :alt: echo output blank project
   :width: 50%

.. todo::

  **TBD**

   Currently, way of creating a single project structure is explained; however it is assumed to be mainly used for verification purpose. 
   Actually, it is necessary to build the project using \ :ref:`multi-project structure <application-layering_project-structure>`\ .
   How to create a multi-project structure will be explained later.

.. warning::

  The following settings defined in pom.xml of Blank project are simply for testing the sample application. Use of these settings in actual development is not assumed.
  In the actual project, these settings should be deleted.
  
    .. code-block:: xml
    
      <dependency>
          <groupId>com.h2database</groupId>
          <artifactId>h2</artifactId>
          <version>1.3.172</version>
          <scope>compile</scope>
      </dependency>

.. raw:: latex

   \newpage

