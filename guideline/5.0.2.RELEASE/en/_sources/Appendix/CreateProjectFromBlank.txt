Create a New Project from Blank Project
================================================================================

.. only:: html

 .. contents:: Table of contents
    :depth: 3
    :local:

This chapter describes a method to create a new project from a blank project.

.. _CreateProjectFromBlankPrerequisite:

Prerequisites
--------------------------------------------------------------------------------

This chapter presumes that the following conditions are in place.
If these prerequisites are not fulfilled, first set up the same.

* Spring Tool Suite should be operational.
* Maven should be executable in command line.
* Internet connectivity.

.. _CreateProjectFromBlank_create-new-project:

.. note::

    If it is required to connect to the internet through proxy server,
    STS proxy settings and `Maven proxy settings <http://maven.apache.org/guides/mini/guide-proxies.html>`_\  are required.

|

.. _CreateProjectFromBlankVerificationEnvironment:

Verification Environment
--------------------------------------------------------------------------------

In this chapter, operations are validated in versions given below.

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - Type
      - Name
    * - OS
      - Windows 7
    * - JVM
      - `Java <http://www.oracle.com/technetwork/java/javase/downloads/index.html>`_ 1.7
    * - IDE
      - `Spring Tool Suite <http://spring.io/tools/sts/all>`_ 3.6.3.RELEASE (hereafter called as "STS")
    * - Build Tool
      - `Apache Maven <http://maven.apache.org/download.cgi>`_ 3.2.5 (hereafter called as "Maven")
    * - Application Server
      - `Pivotal tc Server <https://network.pivotal.io/products/pivotal-tcserver>`_ Developer Edition v3.0 (enclosed in STS)

|

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

|

.. _CreateProjectFromBlankGenerateMultipleProject:

Create a project of multi-project structure
--------------------------------------------------------------------------------

For method of creating a project of multi-project structure, refer to:
":doc:`../ImplementationAtEachLayer/CreateWebApplicationProject`".

The above document includes

* Method to create a project of multi-project structure
* Method to create a war file
* Description of customized locations from blank project
* Description of project structure


|

.. _CreateProjectFromBlankGenerateSingleProject:

Create a project of single-project structure
--------------------------------------------------------------------------------

This section describes about how to create a project of single-project structure.

First, go to the folder wherein a project is to be created.
  
.. code-block:: console
    
    cd C:\work

|

Create a project using `archetype:generate <http://maven.apache.org/archetype/maven-archetype-plugin/generate-mojo.html>`_  of  `Maven Archetype Plugin <http://maven.apache.org/archetype/maven-archetype-plugin/>`_.

.. code-block:: console

    mvn archetype:generate -B^
     -DarchetypeCatalog=http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases^
     -DarchetypeGroupId=org.terasoluna.gfw.blank^
     -DarchetypeArtifactId=terasoluna-gfw-web-blank-mybatis3-archetype^
     -DarchetypeVersion=5.0.2.RELEASE^
     -DgroupId=todo^
     -DartifactId=todo^
     -Dversion=1.0.0-SNAPSHOT

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 25 75
    
    * - Parameter
      - Description
    * - \-B
      - | batch mode (interaction omitted)
    * - | \-DarchetypeCatalog
      - Specify repository of TERASOLUNA Server Framework for Java (5.x). (fixed)
    * - | \-DarchetypeGroupId
      - Specify groupId of blank project. (fixed)
    * - | \-DarchetypeArtifactId
      - Specify archetypeId (ID to identify template) of blank project. **(customization is necessary)**

        Specify any of archetypeId given below.

        * | ``terasoluna-gfw-web-blank-mybatis3-archetype``
          | Template that includes settings for MyBatis3
        * | ``terasoluna-gfw-web-blank-jpa-archetype``
          | Template that includes settings for JPA (Spring Data JPA)
        * | ``terasoluna-gfw-web-blank-archetype``
          | Template that does not depend on O/R Mapper

        In above example, \ ``terasoluna-gfw-web-blank-mybatis3-archetype``\  is specified.
    * - | \-DarchetypeVersion
      - Specify the version of blank project. (fixed)
    * - | \-DgroupId
      - Specify the groupId of project to be created. **(customization is necessary)**

        In above example, \ ``"todo"``\  is specified.
    * - | \-DartifactId
      - Specify the artifactId of project to be created. **(customization is necessary)**

        In above example, \ ``"todo"``\  is specified.
    * - | \-Dversion
      - Specify the version of project to be created. **(customization is necessary)**

        In above example, \ ``"1.0.0-SNAPSHOT"``\  is specified.

.. warning::

    In \ ``pom.xml``\  of blank project, dependency on in-memory database (H2 database) is specified.
    This setting is used to perform minor validations (prototype creation or POC (Proof Of Concept)). It is not assumed to be used in actual development.

     .. code-block:: xml

        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>

    **When H2 Database is not to be used, this setting should be deleted.**

|

.. _CreateProjectFromBlank_STS-import-project:

Import a project in IDE (STS)
--------------------------------------------------------------------------------

This section describes about how to import a created project in STS.

.. note::

    Here, an example to import a single project is given; however, multi projects can also be imported using the same procedure.

|

From STS menu, select [File] -> [Import] -> [Maven] -> [Existing Maven Projects] -> [Next] to open a dialog for selecting the project created using archetype.

.. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlankImportSelect.png
    :alt: Open the dialog to import project
    :width: 80%

|

Set \ ``C:\work\todo``\  in Root Directory, and click [Finish] while pom.xml of todo is selected in Projects.

.. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlankImportProject.png
    :alt: Import project
    :width: 80%

|

When import is successful, a project shown below is displayed in Package Explorer.

.. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlankPackageExplorerAfterImport.png
    :alt: workspace

.. _CreateProjectFromBlank_STS-import-project-update-project:

.. note::

    If build error occurs after import, right click the project name and click "Maven" -> "Update Project...".
    Then by clicking "OK", the error may get resolved.

     .. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlankUpdateProject.png
        :width: 70%

.. tip::

    Display format of package is "Flat" by default; however, view is better if set to "Hierarchical".

    Click "View Menu" of Package Explorer (down arrow on the extreme right), and select "Package Presentation" -> "Hierarchical".

     .. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlankPresentationHierarchical.png
        :width: 80%

    When Package Presentation is set to Hierarchical, the display would be as follows:

     .. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlankPresentationHierarchicalView.png

|

.. _CreateProjectFromBlankDeployAndStartup:

Deploy and start application server (ts Server)
--------------------------------------------------------------------------------

The section describes about how to deploy and launch a project on application server on STS.

.. note::

    In case of multi projects, projects (archetypeId-web) storing the components of application layer (Web layer) will be deployment targets.

|

Right click the imported project and select "Run As" -> "Run on Server".

.. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlankRunOnServer.jpg
    :width: 80%

|

Select AP server (Pivotal tc Server Developer Edition v3.0), and click "Next".

.. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlankTcServerNext.jpg
    :width: 80%

|

Check whether the selected project is included in "Configured" and click "Finish" to start the server.

.. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlankTcServerFinish.jpg
    :width: 80%

.. note::

    The error that occurs at the time of starting the application server may get resolved if clean operations given below are performed.

    * | Clean project
      | From STS menu, select [Project] -> [Clean...], select the target on Clean dialog and click "OK".
    * | \ :ref:`Update Project <CreateProjectFromBlank_STS-import-project-update-project>`\  of Maven
    * | Clean deployed resource
      | Right click "tc Server" of "Servers" view -> [Clean...]
    * | Clean work directory of application server (tc Server)
      | Right click the "tc Server" of "Servers" view -> [Clean tc Server Work Directory...]

|

If  http://localhost:8080/todo is accessed in browser, screen shown below is displayed.

.. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlankTopPage.png
    :width: 80%


.. raw:: latex

   \newpage

