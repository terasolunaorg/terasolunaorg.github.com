Application Layering
********************************************************************************

.. only:: html

 .. contents:: Index
    :depth: 3
    :local:

In this guideline, the application is classified into the following 3 layers.

* Application layer
* Domain layer
* Infrastructure layer

Each layer has the following components.

.. figure:: images/ApplicationLayer.png
   :alt: application layers
   :width: 95%



Both application layer and infrastructure layer depend on domain layer, however
\ **domain layer should not depend on other layers.**
There can be changes in the application layer with the changes in domain layer,
However, there should be no changes in domain layer with the changes in application layer.

Each layer is explained.

.. note::
    Application layer, domain layer, infrastructure layer are the terms
    explained in "Domain-Driven Design (2004, Addison-Wesley)" of Eric Evans.
    However, though the terms are used, they are not necessarily as per the concept used in Domain Driven Design.

|

Layer definition
================================================================================

Flow of data from input to output is Application layer → Domain layer → Infrastructure layer,
hence explanation is given in this sequence.

Application layer
--------------------------------------------------------------------------------

Application layer does the wiring part of the application.

In this layer, it provides the following implementations:

* Provides UI(User Interface) to input and output information.
* Handles the request from clients.
* Validates the input data from a clients
* Calls components in the domain layer corresponding to the request from clients.

**This layer should be as thin as possible and should not contain any business rules.**

Controller
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Controller is mainly responsible for the below:

* Controls the screen flow (mapping the request and returning the View corresponding to the result)
* Calls services in the domain layer (executing main logic corresponding to the request)

In Spring MVC, POJO class having ``@Controller`` annotation becomes the Controller class.

.. note::

  When storing the client input data in the session, controllers also have a role to control the life cycle of data to be stored in the session.

View
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

View's responsibilities is generating output (including the UI provision) to the client.
Returns output results in various formats such as HTML/PDF/Excel/JSON.

In Spring MVC, all this is done by ``View`` class.

.. tip::

    If use the JSON or XML format output results for the REST API or Ajax request,
    \ ``HttpMessageConverter``\  class plays View's responsibility.

    Details refer to ":doc:`../ArchitectureInDetail/REST`".

|

Form
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Form's main responsibilities are the following:

* Represents the form of HTML. (sending form information to the Controller, or outputting the processing result data to form of HTML)
* Declaration of input check rules. (by adding Bean Validation annotation)

In Spring MVC, form object are the POJO class that store request parameters. It is called as form backing bean.

.. note::

    To retain the domain layer independent from the application layer,
    need to perform following processing at the application layer.

    * Conversion from Form to DomainObject(such as Entity)
    * Conversion from DomainObject to Form

    If conversion code is too many,
    recommend to perform of either or both of the following measures and keep Controller's source code in simple state.

    * Create helper class and delegate conversion logic to helper classes.
    * Use the :doc:`Dozer <../ArchitectureInDetail/Utilities/Dozer>`.


.. tip::

    If use the JSON or XML as input data for REST API or Ajax request,
    \ ``Resource``\  class plays Form's responsibility.
    Also, \ ``HttpMessageConverter``\  class plays responsibility to convert JSON or XML input data into \ ``Resource``\  class.

    Details refer to ":doc:`../ArchitectureInDetail/REST`".

|

Helper
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It plays the role of assisting the Controller.

Creating Helper is an option. Please create as POJO class if required.

.. note::

    The main duty of Controller is routing (URL mapping and sepcifying the destination).
    If there is any processing required (converting JavaBean etc),
    then that part must be cut-off from controller and must be delegated to helper classes.
  
    The existance of Helper class improves the visibility of Controller; hence, it is OK to think as a part of Controller.
    (Similar to private methods of Controller)

|

Domain layer
--------------------------------------------------------------------------------

Domain layer is the core of the application, and execute business rules.

In this layer, it provides the following implementations.

* DomainObject
* Checking business rules corresponding to the DomainObject (like whether there is sufficient balance when crediting amount into the account)
* Executing business rules for the DomainObject (reflects values corresponding to business rules)
* Executing CRUD operations for the DomainObject

Domain layer is independent from other layers and is reusable.

DomainObject
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

DomainObject is a model that represents resources required for business and items generated in the business process.

Models are broadly classified into following 3 categories.

* Resource models such as Employee, Customer, Product etc. (generally indicated by a noun)
* Event models such as Order, Payment etc.(generally indicated by a verb)
* Summary model such as YearlySales, MonthlySales etc.

Domain Object is the Entity that represents an object which indicates 1 record of database table.

.. note::

    Mainly \ `Models holding state only <http://martinfowler.com/bliki/AnemicDomainModel.html>`_\  are handled in this guideline.
  
    In "Patterns of Enterprise Application Architecture (2002, Addison-Wesley)" of Martin Fowler,
    Domain Model is defined as \ `Item having state and behavior <http://martinfowler.com/eaaCatalog/domainModel.html>`_\.
    We will not be touching such models in detail.
  
    In this guideline, \ `Rich domain model <http://domaindrivendesign.org>`_\  proposed by Eric Evans is also not included.
    However, it is mentioned here for classification.

|

Repository
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
It can be thought of as a collection of DomainObjects and is responsible for CRUD operations such as create, read, update and delete of DomainObject.

In this layer, only interface is defined.

It is implemented by RepositoryImpl of infrastructure layer.
Hence, in this layer, there is no information about how data access is implemented.

|

Service
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Provides the business logic.

In this guideline, it is recommended to draw the transaction boundary at the method of Service.

.. note::

    Information related to Web such as Form and HttpRequest should not be handled in service.

    This information should be converted into domain object in Application layer before calling the Service.

|

Infrastructure layer
--------------------------------------------------------------------------------

Implementation of domain layer (Repository interface) is provided in infrastructure layer.

It is responsible for storing the data permanently (location where data is stored such as RDBMS, NoSQL etc.) as well as transmission of messages.

RepositoryImpl
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Represents implementation of Repository interface of domain layer. It covers life cycle management of DomainObject.

With the help of this structure, it is possible to hide implementation details from domain layer.

This layer also can be the transaction boundary depending on the requirements.

.. tip::

    When using MyBatis3 or Spring Data JPA, most RepositoryImpls are created automatically.

|

O/R Mapper
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
It is responsible for mapping database with Entity.

This function is provided by MyBatis, JPA and Spring JDBC.

Specifically, the following classes are the O/R Mapper.

* Mapper interface or \ ``SqlSession``\  in case of using MyBatis3
* \ ``EntityManager``\  in case of using JPA,
* \ ``JdbcTemplate``\  in case of using plain Spring JDBC

O/R Mapper used for implementation of Repository.

.. note::

  It is more correct to classify MyBatis and Spring JDBC as "SQL Mapper" and not "O/R Mapper"; however, in this guideline it is treated as "O/R Mapper".

|

Integration System Connector
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It integrates a data store other than the database( such as messaging system, Key-Value-Store, Web service, existing legacy system, external system etc.).

It is also used for implementation of Repository.

|

Dependency between layers
================================================================================
As explained earlier, domain layer is the core of the application, and application layer and infrastructure layer are dependent on it.

In this guideline, it is assumed that,

* Spring MVC is used in application layer
* MyBatis and Spring Data JPA are used in infrastructure layer

Even if there is change in implementation technology, the differences can be absorbed in respective layers and there should not be any impact on domain layer.
Coupling between layers is done by using interfaces and hence they can be made independent of implementation technology being used in each layer.

It is recommended to make loosely-coupled design by understanding the layering.

.. figure:: images/LayerDependencies.png
   :width: 95%

|

Object dependency in each layer can be resolved by DI container.

.. figure:: images/LayerDependencyInjection.png
   :width: 95%

|

Processing and data flow with Repository
--------------------------------------------------------------------------------

The flow from input to output is given in the following figure.

.. figure:: images/LayeringPattern1.png
   :alt: Data flow from request to response
   :width: 100%

Sequence is explained using the example of update operation.

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - No.
      - Description
    * - 1.
      - Controller receives the Request.
    * - 2.
      - (Optional) Controller calls Helper and converts the Form information to DomainObject or DTO.
    * - 3.
      - Controller calls the Service by using DomainObject or DTO.
    * - 4.
      - Service calls the Repository and executes business logic.
    * - 5.
      - Repository calls the O/R Mapper and persists DomainObject or DTO.
    * - 6.
      - (Implementation dependency) O/R Mapper stores DomainObject or DTO information in DB.
    * - 7.
      - Service returns DomainObject or DTO which is the result of business logic execution to Controller.
    * - 8.
      - (Optional) Controller calls the Helper and converts DomainObject or DTO to Form.
    * - 9.
      - Controller returns View name of transition destination.
    * - 10.
      - View outputs Response.

|

Please refer to the below table to determine whether it is OK to call a component from another component.

.. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|
.. list-table:: **Possibility of calling between components**
    :header-rows: 1
    :stub-columns: 1
    :widths: 20 20 20 20 20

    * - Caller/Callee
      - Controller
      - Service
      - Repository
      - O/R Mapper
    * - Controller
      - .. image:: images/cross.png
           :align: center
      - .. image:: images/tick.png
           :align: center
      - .. image:: images/cross.png
           :align: center
      - .. image:: images/cross.png
           :align: center
    * - Service
      - .. image:: images/cross.png
           :align: center
      - .. image:: images/exclamation.png
           :align: center
      - .. image:: images/tick.png
           :align: center
      - .. image:: images/cross.png
           :align: center
    * - Repository
      - .. image:: images/cross.png
           :align: center
      - .. image:: images/cross.png
           :align: center
      - .. image:: images/cross.png
           :align: center
      - .. image:: images/tick.png
           :align: center


Note that \ **calling a Service from another Service is basically prohibited**\.

If services that can be used even from other services are required,
SharedService should be created in order to clarify such a possibility.
Refer to \ :doc:`../ImplementationAtEachLayer/DomainLayer`\  for the details.

.. note::

    It may be difficult to follow the above rules at the initial phase of application development.
    If looking at a very small application, it can be created quickly by directly calling the Repository from Controller.
    However, when rules are not followed, there will be many maintainability issues when development scope becoms larger. It may be difficult to understand the impact of modifications and to add common logic which is cross-cutting in nature.
    It is strongly recommended to pay attention to dependency from the beginning of development so that there will be no problem later on.

|

Processing and data flow without Repository
--------------------------------------------------------------------------------

Hiding the implementation details and sharing of data access logic are the merits of creating a Repository.

However, it is difficult to share data access logic depending on the project team structure (multiple companies may separately implement business processes and control on sharing may be difficult etc.)
In such cases, if abstraction of data access is not required, O/R Mapper can be called directly from Service as shown in the following diagram, without creating the repository.

.. figure:: images/LayeringPattern2.png
   :alt: Data flow from request to response (without Repository)
   :width: 100%

|

Inter-dependency between components in this case must be as shown below:

.. tabularcolumns:: |p{0.25\linewidth}|p{0.25\linewidth}|p{0.25\linewidth}|p{0.25\linewidth}|
.. list-table:: **Inter-dependency between components (without Repository)**
    :header-rows: 1
    :stub-columns: 1
    :widths: 25 25 25 25

    * - Caller/Callee
      - Controller
      - Service
      - O/R Mapper
    * - Controller
      - .. image:: images/cross.png
           :align: center
      - .. image:: images/tick.png
           :align: center
      - .. image:: images/cross.png
           :align: center
    * - Service
      - .. image:: images/cross.png
           :align: center
      - .. image:: images/exclamation.png
           :align: center
      - .. image:: images/tick.png
           :align: center

|

.. _application-layering_project-structure:

Project structure
================================================================================

Here, recommended project structure is explained when application layering is done as described earlier.

The standard maven directory structure is pre-requisite.

Basically, it is recommended to create the multiple projects with the following configuration.

|

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - Project Name
      - Description
    * - [projectName]-domain
      - Project for storing classes/configuration files related to domain layer
    * - [projectName]-web
      - Project for storing classes/configuration files related to application layer
    * - [projectName]-env
      - Project for storing files dependent on environment

([projectName] is the target project name)

.. note::

    Classes of infrastructure layer such as RepositoryImpl are also included in project-domain.
    Originally, [projectName]-infra project should be created separately; however, normally there 
    is no need to conceal the implementation details in the infra project and development becomes easy if implementation is also stored in domain project. 
    Moreover, when required, [projectName]-infra project can be created.


.. tip::

    For the example of multi-project structure, refer to \ `Sample Application <https://github.com/terasolunaorg/terasoluna-tourreservation>`_\  or \ `Test Application of Common Library <https://github.com/terasolunaorg/terasoluna-gfw-functionaltest>`_\ .

|

[projectName]-domain
--------------------------------------------------------------------------------

Recommended structure of [projectName]-domain project is as below:

.. code-block:: console

    [projectName]-domain
      └src
          └main
              ├java
              │  └com
              │      └example
              │          └domain ...(1)
              │              ├model ...(2)
              │              │  ├Xxx.java
              │              │  ├Yyy.java
              │              │  └Zzz.java
              │              ├repository ...(3)
              │              │  ├xxx
              │              │  │  └XxxRepository.java
              │              │  ├yyy
              │              │  │  └YyyRepository.java
              │              │  └zzz
              │              │      ├ZzzRepository.java
              │              │      └ZzzRepositoryImpl.java
              │              └service ...(4)
              │                  ├aaa
              │                  │  ├AaaService.java
              │                  │  └AaaServiceImpl.java
              │                  └bbb
              │                      ├BbbService.java
              │                      └BbbServiceImpl.java
              └resources
                  └META-INF
                      └spring
                          ├[projectName]-codelist.xml ...(5)
                          ├[projectName]-domain.xml ...(6)
                          └[projectName]-infra.xml ...(7)


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - No.
      - Details
    * - | (1)
      - Package to store configuration elements of domain layer.
    * - | (2)
      - Package to store DomainObjects classes.
    * - | (3)
      - Package to store Repository interfaces.

        Create a separate package for each Entity.
        If there are associated Entities to the main entity, then Repository interfaces of associated Entities must also be placed in the same package as main Entity.
        (For example, Order and OrderLine).
        If DTO(holds such as search criteria) is also required, it too must be placed in this package.

        RepositoryImpl belongs to Infrastructure layer; however, there is no problem in keeping it in this project.
        In case of using different data stores or existance of multiple persistence platforms, RepositoryImpl class must be kept in separate project or separate package so that implementation related details are concealed.
    * - | (4)
      - Package to store Service classes.

        Package must be created based on Entity Model or other functional unit. Interface and Implementation class must be kept at the same level of package.
        If input/output classes are also required, then they must be placed in this package.
    * - | (5)
      - Bean definition for CodeList.
    * - | (6)
      - Bean definition pertaining to domain layer.
    * - | (7)
      - Bean definition pertaining to infrastructure layer.

|

[projectName]-web
--------------------------------------------------------------------------------

Recommended structure of [projectName]-web

.. code-block:: console

    [projectName]-web
      └src
          └main
              ├java
              │  └com
              │      └example
              │          └app ...(1)
              │              ├abc
              │              │  ├AbcController.java
              │              │  ├AbcForm.java
              │              │  └AbcHelper.java
              │              └def
              │                  ├DefController.java
              │                  ├DefForm.java
              │                  └DefOutput.java
              ├resources
              │  ├META-INF
              │  │  └spring
              │  │      ├applicationContext.xml ...(2)
              │  │      ├application.properties ...(3)
              │  │      ├spring-mvc.xml ...(4)
              │  │      └spring-security.xml ...(5)
              │  └i18n
              │      └application-messages.properties ...(6)
              └webapp
                  ├resources ...(7)
                  └WEB-INF
                      ├views ...(8)
                      │  ├abc
                      │  │ ├list.jsp
                      │  │ └createForm.jsp
                      │  └def
                      │     ├list.jsp
                      │     └createForm.jsp
                      └web.xml ...(9)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - No,
      - Details
    * - | (1)
      - Package to store configuration elements of application layer.
    * - | (2)
      - Bean definited related to the entire application.
    * - | (3)
      - Define the properties to be used in the application.
    * - | (4)
      - Bean definitions related to Spring MVC.
    * - | (5)
      - Bean definitions related to Spring Security
    * - | (6)
      - Define the messages and other contents to be used for screen display (internationalization).
    * - | (7)
      - Store static resources(css、js、image, etc)
    * - | (8)
      - Store View(jsp) files.
    * - | (9)
      - Servlet deployment definitions

|

[projectName]-env
--------------------------------------------------------------------------------

The recommended structure of [projectName]-env project is below:

.. code-block:: console

    [projectName]-env
      ├configs ...(1)
      │   └[envName] ...(2)
      │       └resources ...(3)
      └src
          └main
              └resources ...(4)
                 ├META-INF
                 │  └spring
                 │      ├[projectName]-env.xml ...(5)
                 │      └[projectName]-infra.properties ...(6)
                 ├dozer.properties
                 ├log4jdbc.properties
                 └logback.xml ...(7)


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - No.
      - Details
    * - | (1)
      - Directory to define configurations depends on the environment for all environments.
    * - | (2)
      - Directory to define configurations depeands on each environment.

        The directory name is used as the name to identify the environment.
    * - | (3)
      - Directory to define configurations depeands on each environment.

        The sub directory structure and files are same as (4).
    * - | (4)
      - Directory to define configurations depeands on the local development environment.
    * - | (5)
      - Bean definitions that depend on the local development environment (like dataSource etc).
    * - | (6)
      - Property definitions which depend on the local development environment.
    * - | (7)
      - Log output definitions which depend on the local development environment.


.. note::

    The purpose of separating [projectName]-domain and [projectName]-web into different projects is to prevent reverse dependency among them.
  
    It is natural that [projectName]-web uses [projectName]-domain; however, [projectName]-domain must not refer projectname]-web.
  
    If configruation elements of both, [projectName]-web and [projectName]-domain, are kept in a single project, it may lead to an incorrect reference by mistake.
    It is strongly recommended to prevent reference to [projectName]-web from [projectName]-domain by separating the project and setting order of reference.

.. note::

    The reason for creating [projectName]-env is to separate the information depending on the environment and thereby enable switching of environment.
  
    For example, set local development environment by default and at the time of building the application, create war file without including [projectName]-env.
    By creating a separate jar for integration test environment or system test environment, deployment becomes possible just by replacing the jar of corresponding environment.
  
    Further, it is also possible to minimize the changes in case of project where RDBMS being used changes.
    
    If the above point is not considered, contents of configuration files have to changed according to the target environment and the project has to be re-build.
  
    For further details regarding significance of creating a separate project for environment dependent files, refer to \ :doc:`../Appendix/EnvironmentIndependency`\ .


.. raw:: latex

   \newpage

