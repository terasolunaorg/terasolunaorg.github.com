Build development project
================================================================================

.. only:: html

 .. contents:: Index
    :depth: 3
    :local:

Build development project
--------------------------------------------------------------------------------

The method to create a war file to be deployed on application server and a jar file of env module (module to store the file environment dependent file) is described below.

In case of a project created using Maven Archetype, the following 2 methods are provided as methods to create a war file.

* :ref:`CreateWebApplicationProjectBuildWarExcludeEnvJar` (**recommended**)
* :ref:`CreateWebApplicationProjectBuildWarIncludeEnvJar`


.. note:: **About the recommended build method**

    This guideline recommends :ref:`CreateWebApplicationProjectBuildWarExcludeEnvJar`. 
    Other build method apart from those mentioned here can also be used.

    However, **the war file and jar file to be released in test environment and production environment should not be created using the functionality provided by IDE such as Eclipse.**
    In some of the IDE functionalities like Eclipse, class files are created using an independent compiler which has been optimized for development,
    hence there could be a risk of unexpected error during the application execution due to difference in the compiler.

.. note:: **Rule for the project structure**
    
    Consider the following project structure.

    1. Make sure to have a multi-project structure.
    2.As far as possible, consolidate the configuration files (ex. logback.xml, jdbc.properties) having environment dependency in one project.  **Hereafter, this project is expressed as \*-env.**
    
      * ex. terasoluna-tourreservation-env

    3. Projects other than \*-env never have a setting for environment dependency.

      * However, it is allowed to store the environment dependency configuration files for test under src/test/resources.

    4. Manage all software packaged binaries by storing them in package repository.

      * Deploy not only \*.jar files but also \*.war files as work products in package repository. Consequently, these jar/war files should not include environment dependencies.

    5. Configure \*-env project as follows:

      * Store the configuration values required for operations on the developer's PC in the file under src/main/resources, as default values.
      * Store the configuration files that differ with each environment such as test server, production server, in the folder other than src/main/resources (ex. configs/test-server). Then, use "profile" function of Maven that automatically replaces the configuration values depending on environment in order to build \*-env-x.y.z.jar file.

    The above structure facilitates proper development in all the scenarios of software development lifecycle.

    #. In local development environment, check out both main project and \*-env project and include env project in build path of the main project so as to do coding and testing in the local development environment.
    #. On CI server, execute the test and perform packaging using a build tool (Maven) and deploy the artifact in package repository whenever required.
    #. Application can work on test server and production server by adding \*-env project built as per the application release environment, to the main project which is stored in package repository.

    For details, refer to \ `Sample application <https://github.com/terasolunaorg/terasoluna-tourreservation>`_\ .

.. warning:: **About build environment**

    In the example below, Windows environment is used for the build. However, you can use your own environment for doing the build.
    This guideline **recommends that you should do the build using the same OS and JDK version as that of the application execution environment.**

|

| When build is done using Maven, confirm whether home directory of JDK which is used during compilation in the environment variable JAVA_HOME, has been specified.
| If the environment variable is not set or the home directory of JDK having different version has been specified, an appropriate home directory should be specified in environment variable.

**[In case of Windows]**

.. code-block:: console

    echo %JAVA_HOME%
    set JAVA_HOME={Please set home directory of JDK}


**[In case of Linux]**

.. code-block:: console

    echo $JAVA_HOME
    JAVA_HOME={Please set home directory of JDK}

.. note::

    It is advisable to set the environment variable JAVA_HOME in the user environment variable of OS user wherein build is to be done.

|

.. _CreateWebApplicationProjectBuildWarExcludeEnvJar:

Build method wherein jar file of env module is not included in war file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _CreateWebApplicationProjectBuildWarExcludeEnvJarStepWar:

Create war file
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Open the root directory of development project.

.. code-block:: console

    cd C:\work\todo

|

| Specify \ ``warpack``\  in Maven profile (\ ``-P``\  parameter) and run Maven install.


.. code-block:: console

    mvn -P warpack clean install

| If the Maven package is run successfully, a war file that does not include jar file of env module is created in the target directory of web module.
| (Example: \ ``C:\work\todo\todo-web\target\todo-web.war``\ )

.. note:: **About the goal to be specified**

    In the above example, \ ``install``\  is specified in goal and war file is installed in local repository, however it is advisable to specify

     * \ ``package``\  in goal when only creating a war file
     * \ ``deploy``\  in goal when deploying in remote repository like Nexus


|

.. _CreateWebApplicationProjectBuildWarExcludeEnvJarStepEnvJar:

Create jar file of env module
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Open env module directory.

.. code-block:: console

    cd C:\work\todo\todo-env

|

Specify \ **Profile ID to identify environment**\  in Maven profile (\ ``-P``\  parameter) and run Maven package.

.. code-block:: console

    mvn -P test-server clean package

| If Maven package is run successfully, jar file for the specified environment is created in target directory of env module.
| (Example: \ ``C:\work\todo\todo-env\target\todo-env-1.0.0-SNAPSHOT-test-server.jar``\ )

.. note:: **About profile ID to identify environment**

    In case of a project created using Maven Archetype, following profile IDs are defined by default.

     * \ ``local``\ : Profile for local environment of the developer (for IDE development environment) (default profile)
     * \ ``test-server``\ : Profile for test environment
     * \ ``production-server``\ : Profile for production environment

    The above 3 profiles are provided by default; however you can add or modify them as per the environment configuration of the system to be developed.

|

.. _CreateWebApplicationProjectBuildWarIncludeEnvJar:

Build method wherein jar file of env module is included in war file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _CreateWebApplicationProjectBuildWarIncludeEnvJarWar:

Create war file
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. warning:: **Points to be noted when including a jar file of env module in war file**

    When jar file of env module is included in war file, the war file cannot be deployed in other environment; 
    hence war file should be managed so that it is not deployed to other environment (especially in production environment) by mistake.

    Moreover, when using a method in which war file is created for each environment and released in each environment, 
    it should be noted that war file released in production environment can never be the war file for which testing is complete.
    This is for the re-compilation at the time of creating war file for the production environment.
    When creating the war file and releasing the same for each environment, it is especially important to use the 
    VCS (Version Control System) functionality (Tag functionality etc.) like Git or Subversion and to establish a mechanism to create a war file
    which is to be released in production environment and various test environments, through the use of tested source files.

|

Open the root directory of development project.

.. code-block:: console

    cd C:\work\todo

|

| In Maven profile (\ ``-P``\  parameter), specify \ **Profile ID to identify environment**\ defined in env module and \ ``warpack-with-env``\ , and then run the Maven package.

.. code-block:: console

    mvn -P warpack-with-env,test-server clean package

| If Maven package is run successfully, war file which includes jar file of env module is created in target directory of web module.
| (Example: \ ``C:\work\todo\todo-web\target\todo-web.war``\ )

|


.. _CreateWebApplicationProjectBuildDeploy:

Deploy
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _CreateWebApplicationProjectBuildDeployToTomcat:

Deploy on Tomcat
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Deployment method (procedure) when Tomcat is used as an application server is given below.

1. Specify the profile of Maven as per the AP server environment in which the application is to be released and build \*-env project.
2. Place \*-env-x.y.z.jar file built above in the folder of AP server decided in advance. ex. /etc/foo/bar/abcd-env-x.y.z.jar
3. Unjar the \*.war file deployed in package repository under [CATALINA_HOME]/webapps.
4. If Tomcat 7 is used, add /etc/foo/bar/\*.jar into class path using VirtualWebappLoader function of the Tomcat.

 * The following definition should be added in [CATALINA_HOME]/conf/[contextPath].xml file.
 * For details, refer to http://tomcat.apache.org/tomcat-7.0-doc/api/org/apache/catalina/loader/VirtualWebappLoader.html and `configs folder of terasoluna-tourreservation-env <https://github.com/terasolunaorg/terasoluna-tourreservation/tree/5.3.0.RELEASE/terasoluna-tourreservation-env/configs>`_\ .
 * Example of VirtualWebappLoader function usages :
 
  .. code-block:: xml

   <Loader className="org.apache.catalina.loader.VirtualWebappLoader"
           virtualClasspath="/etc/foo/bar/*.jar" />
	 
 * In addition, VirtualWebappLoader can also be used in the Tomcat 6.

5. If Tomcat 8.x is used, add /etc/foo/bar/\*.jar into class path using Resource function of the Tomcat.

 * The following definition should be added in [CATALINA_HOME]/conf/[contextPath].xml file.
 * For details, refer to https://tomcat.apache.org/migration-8.html#Web_application_resources and `configs folder of terasoluna-tourreservation-env <https://github.com/terasolunaorg/terasoluna-tourreservation/tree/5.3.0.RELEASE/terasoluna-tourreservation-env/configs>`_\ .
 * Example of Resource function usages :
   
  .. code-block:: xml

   <Resources className="org.apache.catalina.webresources.StandardRoot">
     <PreResources className="org.apache.catalina.webresources.DirResourceSet"
                   base="/etc/foo/bar/"
                   internalPath="/"
                   webAppMount="/WEB-INF/lib" />
   </Resources>

.. note::

 * autoDeploy attribute of Host tag of [CATALINA_HOME]/conf/server.xml should be set to false. Otherwise [CATALINA_HOME]/conf/[contextPath].xml gets deleted each time web application is restarted.
 * When autoDeploy is disabled, Web application does not start by just placing the war file in [CATALINA_HOME]/webapps. war file should always be unjarred (unzipped).

|

.. _CreateWebApplicationProjectBuildDeployToOtherServer:

Deployment to other application server
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Pprocedure to deployment on other (not tomcat) application server is explained.

When releasing the Web application on application servers (Example: WebSphere, WebLogic, JBoss) where a mechanism for
adding a class path for each web application (which is provided in VirtualWebappLoader of Tomcat) is not provided,
the method to release it after adding \*-env-x.y.z.jar file under WEB-INF/lib of war file is the easiest.

1. Specify profile of Maven as per the AP server environment in which application is to be released and build \*-env project.
2. Copy \*.war file deployed in the package repository to the working directory.
3. Add it under WEB-INF/lib of war file using add option of jar command as follows.
4. Release foo-x.y.z.war on AP server.

.. note::

    For a method to deploy a war file on application server, refer to the manual of application server to be used.

|

Here, a method to embed the jar file of env module in war file using jar command is given.

| Open the working directory.
| Here the in the example below, work is performed in env project.

.. code-block:: console

    cd C:\work\todo\todo-env

|

| Copy the created war file to the working directory.
| Here in the example below, war file is fetched from Maven repository. (war file is required to be \ ``installed``\  or \ ``deployed``\ .)

.. code-block:: console

    mvn org.apache.maven.plugins:maven-dependency-plugin:2.5:get^
     -DgroupId=com.example.todo^
     -DartifactId=todo-web^
     -Dversion=1.0.0-SNAPSHOT^
     -Dpackaging=war^
     -Ddest=target/todo-web.war

| If the command is run successfully, the specified war file is copied to the target directory of env module.
| (Example: \ ``C:\work\todo\todo-env\target\todo-web.war``\ )

.. note::

    * An appropriate value should be specified in \ ``-DgroupId``\ , \ ``-DartifactId``\ , \ ``-Dversion``\ , \ ``-Ddest``\ . 
    * When run on Linux, \ ``^``\  at the end of the line should be read as \ ``\``\  . 

|

Copy the created jar file to working directory (\ ``target\WEB-INF\lib``\ ) once and add it to the war file.

**[In case of Windows]**

.. code-block:: console

    mkdir target\WEB-INF\lib
    copy target\todo-env-1.0.0-SNAPSHOT-test-server.jar target\WEB-INF\lib\.
    cd target
    jar -uvf todo-web.war WEB-INF\lib

**[In case of Linux]**

.. code-block:: console

    mkdir -p target/WEB-INF/lib
    cp target/todo-env-1.0.0-SNAPSHOT-test-server.jar target/WEB-INF/lib/.
    cd target
    jar -uvf todo-web.war WEB-INF/lib

.. note:: **Measures to be taken when jar command is not found**

    The problem when jar command is not found can be resolved using either of the following measures.

    * Add \ ``JAVA_HOME/bin``\  to environment variable "PATH". 
    * Specify the jar command with full path. In case of Windows, \ ``%JAVA_HOME%\bin\jar``\  and in case of Linux, \ ``${JAVA_HOME}/bin/jar``\  can be specified.


.. _CreateWebApplicationProjectBuildDeployContinuedDeployment:

Continuous deployment
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Continuous deployment is constantly releasing the target software through continuous looping of project (source code tree) structure, version control, inspection, build operations and lifecycle management.

During development, release the software of SNAPSHOT version in the package repository and development AP server and execute the test.
To release the software officially, tagging to source code tree in VCS needs be performed after assigning a version number.
In this way, the flow of build and deployment slightly differs in the snapshot release and official release.

To deploy the application on AP server that provides Web service, irrespective of snapshot version or official release version, a group of
environment dependency configuration files and \*.war file should be deployed in a set as per the target release AP server environment.

Separating the operation of registering libraries (jar, war) without environment dependency settings, in Maven repository and
the operation of actually deploying them on AP server facilitates deployment.

.. note::

 In Maven, it is automatically distinguished whether it is a SNAPSHOT version or RELEASE version according to the contents of <version> tag of pom.xml.

 * It is considered as SNAPSHOT if it ends with -SNAPSHOT. Example: <version>1.0-SNAPSHOT</version>
 * It is considered as RELEASE if it does not end with -SNAPSHOT. Example: <version>1.0</version>

 Please note that there are 2 types of repositories in Maven package repository i.e. snapshot repository and release repository with a few limitations.

 * Software of SNAPSHOT version cannot be registered in release repository. release repository also cannot be registered in snapshot repository.
 * In release repository, artifact having the same GAV information can be registered only once. (GAV=groupId, artifactId, version)
 * In snapshot repository, artifact having the same GAV information can be re-registered many times.

Operations of SNAPSHOT version
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

A simple delivery flow of SNAPSHOT version software is as shown in the following figure.

.. figure:: ./images/ContinuousDelivery-snapshot.png
   :alt: Continuous delivery for SNAPSHOT version.
   :width: 600px

1. Check out the source code from development trunk.
2. Compile, measure the code metrics and execute test.

 * In case of compilation error, certain violations of code metrics or in case the test fails, the subsequent operations should be stopped.

3. Upload (mvn deploy) the artifact (jar, war file) on Maven package repository server.
|

Operations of RELEASE version
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

In case of official release, since it is necessary to assign the version number, the flow becomes slightly more complex than the SNAPSHOT release.

.. figure:: ./images/ContinuousDelivery-release.png
   :alt: Continuous delivery for RELEASE version.
   :width: 600px

1. Decide the version number to be assigned for release. (Example：1.0.1)
2. Check out the source code from development trunk (or release branch).
3. Change <version> tag of pom.xml. (Example：<version>1.0.1</version>)
4. Assign tag to VCS. (Example： tags/1.0.1)
5. Compile, measure the code metrics and execute test.

 * In case of compilation error, certain violations of code metrics or in case the test fails, the subsequent operations should be stopped.
 * If the test fails, delete the tag of VCS.

6. Upload (mvn deploy) the artifact (jar, war file) on Maven package repository server.

.. todo:: 
 
 Here, should the version tag of pom.xml of trunk source tree be written at the end till it is
 replaced by the next SNAPSHOT version and committed?

.. note::

 <version> tag of pom.xml file can be changed in `versions-maven-plugin <http://www.mojohaus.org/versions-maven-plugin/>`_ .
 
 .. code-block:: bash
 
  mvn versions:set -DnewVersion=1.0.0
 
 Version tag in pom.xml can be edited as <version>1.0.0</version> by the above commands.

Release on Application Server
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

To release the application on AP server that provides Web service,
release the \*.war file registered in Maven package repository and the group of environment dependency
configuration files in a set according to the target release AP server environment.
This has same flow irrespective of snapshot release or official release.

.. figure:: ./images/ContinuousDelivery-apserver.png
   :alt: Continuous delivery for webapp to application server.
   :width: 600px

1. Download war file of the version to be released from Maven package repository.
2. Check out \*-resources project (that consolidates environment dependency configuration files) from VCS.
3. Using "profile" function of Maven, replace the contents with group of configuration files according to the target release environment, package the resources project and create \*-resources-x.y.z.jar.
4. Add the created \*-resources-x.y.z.jar file under WEB-INF/lib folder of war file.

 * In case of Tomcat, instead of adding \*-resources-x.y.z.jar to war file, copy it to any path of Tomcat server and specify that path in the extended class path of VirtualWebappLoader.

5. Deploy the war file on application server.

.. note::

 War file can be downloaded from Maven package repository with "get goal" of maven-dependency-plugin.

 .. code-block:: bash

  mvn org.apache.maven.plugins:maven-dependency-plugin:2.5:get \
   -DgroupId=com.example \
   -DartifactId=mywebapp \
   -Dversion=0.0.1-SNAPSHOT \
   -Dpackaging=war \
   -Ddest=${WORKSPACE}/target/mywebapp.war

 With this, mywebapp.war file is downloaded under the target directory.
 
 Package of environment dependency configuration files can be added to mywebapp.war file using the following commands.

 .. code-block:: bash

  mkdir -p $WORKSPACE/target/WEB-INF/lib
  cd $WORKSPACE/target
  cp ./mywebapp-resources*.jar WEB-INF/lib
  jar -ufv mywebapp.war WEB-INF/lib


.. raw:: latex

   \newpage
