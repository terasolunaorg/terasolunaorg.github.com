--------------------------------------------------------------------------------
Project Structure Standard
--------------------------------------------------------------------------------

.. todo::

  Rewrite.


It is recommended to split the software source code tree into multiple projects so as to have a multi-project structure.

.. note::
 
 Here, it is assumed that Maven is used as a build tool.
 When Maven is used, the standard multi-project structure consists of Hierarchical project layout.
 However, Eclipse is used as development environment and it supports only the Flat layout and not the Hierarchical layout,
 hence use Flat layout.

Simple pattern
--------------

A simplest project structure of Web application development project "foo" is as follows:

* foo-parent
* foo-initdb
* foo-domain
* foo-web
* foo-env
* foo-selenium

The details of each project are as follows:

.. _foo-parent-label: 

* foo-parent

 Project called parent-pom (parent POM). A simple project consisting of only pom.xml file.
 It never contains other source code or configuration files.
 By specifying this foo-parent project in the <parent> tag of pom of other project,
 the common configuration information specified in the parent POM itself can be reflected.

* foo-initdb

 Stores the SQL statement to INSERT the initial data and table definition (DDL) of RDBMS.
 This is also managed as Maven project. By defining `sql-maven-plugin <http://mojo.codehaus.org/sql-maven-plugin/>` setting in pom.xml,
 the execution of DDL statement of any RDBMS and INSERT statement of initial data during the build lifecycle can be automatically executed.

* foo-domain

 Stores the classes used as domain layer such as service class, repository class etc. The class of this domain layer is used to create a class of application layer in foo-web.

* foo-web

 Stores application layer classes, jsps, configuration files, unit test cases etc. Finally \*.war file is created as Web application.

* foo-env

 Consolidates only configuration files having environment dependency. foo-web has dependency on foo-env.
 Refer to :doc:`EnvironmentIndependency` for details.

* foo-selenium

 Stores test cases using `Selenium WebDriver <http://seleniumhq.org/projects/webdriver/>`_.
 
Complex pattern
---------------

The project structure of development project "bar" where 2 Web applications and 1 common library are required is as follows:

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

The details of each project are as follows:

* bar-parent

  (same as foo-parent)

* bar-initdb

  (same as foo-initdb)

* bar-common

 Stores common library in the project. This is web independent and web related classes are placed under bar-common-web.

* bar-common-web

 Stores common web library in the project.

* bar-domain

 Stores java classes and unit test cases of domain layer of Domain 'a'. Finally \*.jar file is created.

* bar-domain

 Class of domain layer of Domain 'b'.

* bar-web-a

 Stores application layer java classes, jsps, configuration files, unit test cases etc. Finally \*.war file is created as the Web application.
 bar-web-a has dependency on bar-common and bar-env.

* bar-web-b

 This is a Web application as one more subsystem. Its structure is same as the bar-web-a.

* bar-env

 Collects only the configuration files having environment dependency. Refer to :doc:`EnvironmentIndependency` for details.

* bar-web-a-selenium

 Stores test cases using `Selenium WebDriver <http://seleniumhq.org/projects/webdriver/>`_ for web-a project.

* bar-web-b-selenium

 Stores test cases using `Selenium WebDriver <http://seleniumhq.org/projects/webdriver/>`_ for web-b project.


.. todo::
    Additionally, describe about splitting a JSP.

.. raw:: latex

   \newpage

