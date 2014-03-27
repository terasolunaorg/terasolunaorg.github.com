Application Layering
********************************************************************************

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
   :width: 80%



Application layer as well as infrastructure layer depends on domain layer, however
\ ** domain layer should not depend on other layers.**
There can be changes in the application layer with the changes in domain layer,
However, there should be no changes in domain layer with the changes in application layer.

Each layer is explained.

.. note::
    Application layer, domain layer, infrastructure layer are the terms
    explained in "Domain-Driven Design (2004, Addison-Wesley)" of Eric Evans.
    However, though the terms are used, they are not necessarily as per the concept used in Domain Driven Design.


Layer definition
================================================================================

Flow of data from input to output is Application layer → Domain layer → Infrastructure layer,
hence explanation is given in this sequence.

Application layer
--------------------------------------------------------------------------------

Application layer does the wiring part of the application. It provides a UI to input/output information, 
calls the domain layer by converting request information into model information and also constructs the output from the processing result.
\ **This layer should be as thin as possible and should not contain any business rules.**

Controller
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Controller is responsible for mapping the requests to the process and passing the results to the view as well as session management.
Main logic processing must be done in Service of domain layer without performing them in the Controller.

In Spring MVC, POJO class having ``@Controller`` annotation becomes the Controller class.
The output of Controller is (logical name of) the View.


View
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
View generates output to the Client. Returns output results in various formats such as JSP/PDF/Excel/JSON.
In Spring MVC, all this is done by ``View`` class. 

Form
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Represents the form of the screen. Used for sending form information to the Controller and data output from Controller to form of the screen.
Conversion from Form to DomainObject(such as Entity) and conversion from DomainObject to Form has to be done in application layer,
so that domain layer does not dependent on the application layer.


.. note::
 When conversion is done in controller, source code of controller can get too long.,
 and visibility of Controller's main responsibility (like screen transition) becomes poor.
 In that case, it is recommended to create helper class and delegate conversion logic to helper classes.

In Spring MVC, form object are the POJO class that store request parameters. It is called as form backing bean.


Helper
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It plays the role of assisting the Controller and performs the processes other than original duties of Controller such as mutual conversion of model 
between Application layer and Domain layer. It can be considered as a part of Controller.

Helper is an option, and should be created as POJO class if required.

.. note::

  Helper class exists to improve the visibility of Controller; hence, it is OK to think of it as a part of Controller.
  The main duty of Controller is routing (URL mapping and sepcifying the destination). If there is any processing required (converting JavaBean etc), 
  then that part must be cut-off from controller and must be delegated to helper classes.
  The purpose is to keep the controller class clean; hence, helper class is similar to private methods of Controller.


Domain layer
--------------------------------------------------------------------------------

Domain layer is the core of the application. Represents the business issues to be resolved and has business object and business rules 
(checks like whether there is sufficient balance when crediting amount into the account).
Domain layer is not so thick as compared to other layers and is reusable.

DomainObject
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| DomainObject is a model that represents resources required for business and items generated in the business process.
| Models are broadly classified into following 4 categories.
* Resource models such as Employee, Customer, Product etc. (generally indicated by a noun),  
* Event models such as Order, Payment (generally indicated by a verb), 
* Summary model such as YearlySales, MonthlySales.

Domain Object is the Entity that represents an object which indicates 1 record of database table.

.. note::
  Mainly \ `Models holding state only <http://martinfowler.com/bliki/AnemicDomainModel.html>`_\ are handled in this guideline.
  In "Patterns of Enterprise Application Architecture (2002, Addison-Wesley)" of Martin Fowler,
  Domain Model is defined as \ `Item having state and behavior <http://martinfowler.com/eaaCatalog/domainModel.html>`_\. 
  We will not be touching such models in detail.
  In this guideline, \ `Rich domain model <http://domaindrivendesign.org>`_\  proposed by Eric Evans is also not included.
  However, it is mentioned here for classification.

Repository
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
It can be thought of as a collection of DomainObjects and is responsible for CRUD operations such as create, read, update and delete of DomainObject.
In this layer, only interface is defined. It is implemented by RepositoryImpl of infrastructure layer. Hence, in this layer, there is no information about
how data access is implemented.

Service
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Provides the business logic. This is also the transaction boundary. 

Information related to Web such as Form and HttpRequest should not be handled in service.
This information should be converted to object of domain layer in Application layer before calling Service. 

Infrastructure layer
--------------------------------------------------------------------------------

Implementation (Repository interface) of domain layer is provided in infrastructure layer.
It is responsible for storing the data permanently (location where data is stored such as RDBMS, NoSQL etc.) as well as transmission of messages.

RepositoryImpl
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Represents implementation of Repository interface of domain layer. IT covers life cycle management of DomainObject.
With the help of this structure, it is possible to hide implementation details from domain layer.
When using Spring Data JPA, a few RepositoryImpls are created by Spring Data JPA automatically.

O/R Mapper
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
It is responsible for mapping database with Entity.
This function is provided by JPA, MyBatis and Spring JDBC.
When using JPA specifically, EntityManager can be used. In case of MyBatis2 (TERASOLUNA DAO), QueryDAO and UpdateDAO are applicable are used.

.. note::

  It is more correct to classify MyBatis and Spring JDBC as "SQL Mapper" and not "O/R Mapper"; however, in this guideline it is treated as "O/R Mapper".


Integration System Connector
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
It links a data store other than the database; such as messaging system, Key-Value-Store, Web service, existing legacy system etc. 
It also links with some external systems. Used for implementation of Repository.


Dependency between layers
================================================================================
| As explained earlier, domain layer is the core of the application, and application layer and infrastructure layer are dependent on it.

| In this guideline, it is assumed that, 
* Spring MVC is used in application layer 
* Spring Data JPA and MyBatis are used in infrastructure layer
| Even if there is change in implementation technology, the differences can be absorbed in respective layers and there should not be any impact on domain layer.
| Coupling between layers is done by using interfaces and hence they can be made independent of implementation technology being used in each layer.

It is recommended to make loosely-coupled design by understanding the layering.

.. figure:: images/LayerDependencies.png
   :width: 80%


Object dependency in each layer can be resolved by DI container.

.. figure:: images/LayerDependencyInjection.png
   :width: 90%


The flow from input to output is given in the following figure.

.. figure:: images/LayeringPattern1.png
   :alt: Data flow from request to response
   :width: 100%

Sequence is explained using the example of update operation.

#. Controller receives the Request.
#. (Optional) Controller calls Helper and converts the Form information to DomainObject or DTO.
#. Controller calls the Service by using DomainObject or DTO.
#. Service calls the Repository and executes business logic.
#. Repository calls the O/R Mapper and persists DomainObject or DTO.
#. (Implementation dependency) O/R Mapper stores DomainObject or DTO information in DB.
#. Service returns DomainObject or DTO which is the result of business logic execution to Controller.
#. (Optional) Controller calls the Helper and converts DomainObject or DTO to Form.
#. Controller returns View name of transition destination.
#. View outputs Response.


Please refer to the below table to determine whether it is OK to call a component from another component.

.. list-table:: Possibility of calling between components
    :header-rows: 1
    :stub-columns: 1

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


Note that \**Calling a Service from another Service is basically prohibited**\.
If services that can be used even from other services are required,
SharedService should be created in order to clarify such a possibility.
Refer to \ :doc:`../ImplementationAtEachLayer/DomainLayer`\  for the details.

.. note::
  It may be difficult to follow the above rules at the initial phase of application development.
  If looking at a very small application, it can be created quickly by directly calling the Repository from Controller.
  However, when rules are not followed, there will be many maintainability issues when development scope becoms larger. It may be difficult to understand the impact of modifications and to add common logic which is cross-cutting in nature.
  It is strongly recommended to pay attention to dependency from the beginning of development so that there will be no problem later on.


Hiding the implementation details and sharing of data access logic are the merits of creating a Repository.
However, it is difficult to share data access logic depending on the project team structure (multiple companies may separately implement business processes and control on sharing may be difficult etc.)
In such cases, if abstraction of data access is not required, O/R Mapper can be called directly from Service as shown in the following diagram, without creating the repository.

.. figure:: images/LayeringPattern2.png
   :alt: Data flow from request to response (without Repository)
   :width: 100%


Inter-dependency between components in this case must be as shown below:

.. list-table:: Inter-dependency between components (without Repository)
    :header-rows: 1
    :stub-columns: 1

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


Project structure
================================================================================

Here, recommended project structure is explained when application layering is done as described earlier.

The standard maven directory structure is pre-requisite.

Basically, it is recommended to create the multiple projects with the following configuration.

* [projectname]-domain ... Project for storing classes/configuration files related to domain layer
* [projectname]-web ... Project for storing classes/configuration files related to application layer
* [projectname]-env ... Project for storing files dependent on environment

([projectname] is the target project name)

.. note::
    Classes of infrastructure layer such as RepositoryImpl are also included in project-domain.
    Originally, [projectname]-infra project should be created separately; however, normally there 
    is no need to conceal the implementation details in the infra project and development becomes easy if implementation is also stored in domain project, . 
    Moreover, when required, [projectname]-infra project can be created.


.. tip::
    For the example of multi-project structure, refer to \ `Sample Application <https://github.com/terasolunaorg/terasoluna-tourreservation>`_\  or \ `Test Application of Common Library <https://github.com/terasolunaorg/terasoluna-gfw-functionaltest>`_\ .


[projectname]-domain
--------------------------------------------------------------------------------

Recommended structure of [projectname]-domain project is as below:

.. code-block:: console

    [projectName]-domain
      └src
          └main
              ├java
              │  └com
              │      └example
              │          └domain ...(1)
              │              ├model
              │              │  ├Xxx.java
              │              │  ├Yyy.java
              │              │  └Zzz.java
              │              ├repository ...(2)
              │              │  ├xxx
              │              │  │  └XxxRepository.java
              │              │  ├yyy
              │              │  │  └YyyRepository.java
              │              │  └zzz
              │              │      ├ZzzRepository.java
              │              │      └ZzzRepositoryImpl.java
              │              └service ...(3)
              │                  ├aaa
              │                  │  ├AaaService.java
              │                  │  └AaaServiceImpl.java
              │                  └bbb
              │                      ├BbbService.java
              │                      └BbbServiceImpl.java
              └resources
                  └META-INF
                      └spring
                          ├[projectname]-domain.xml ...(4)
                          └[projectname]-infra.xml ...(5)


.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - No.
      - Details
    * - | (1)
      - | Store DomainObjects
    * - | (2)
      - | Store Repository interfaces. Create a separate package for each Entity.
        | If there are associated Entities to the main entity, then Repository interfaces of associated Entities must also be placed in the same package as main Entity.
        | (For example, Order and OrderLine). If DTO is also required, it too must be placed in this package.
        | RepositoryImpl belongs to Infrastructure layer; however, there is no problem in keeping it in this project. 
        | In case of using different data stores or existance of multiple persistence platforms, RepositoryImpl class must be kept in separate project or separate package so that 
        | implementation related details are concealed.
    * - | (3)
      - | Service classes are kept here. Package must be created based on Entity Model or other functional unit. Interface and Implementation class must be kept at the same level of package.
        | If input/output classes are also required, then they must be placed in this package.
    * - | (4)
      - | Bean definition pertaining to domain layer.
    * - | (5)
      - | Bean definition pertaining to infrastructure layer.


[projectname]-web
--------------------------------------------------------------------------------

Recommended structure of [projectname]-web

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
                  └WEB-INF
                      ├views ...(7)
                      │  ├abc
                      │  │ ├list.jsp
                      │  │ └createForm.jsp
                      │  └def
                      │     ├list.jsp
                      │     └createForm.jsp
                      └web.xml

.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - No,
      - Details
    * - | (1)
      - | Package to store configuration elements of application layer.
    * - | (2)
      - | Bean definited related to the entire application.
    * - | (3)
      - | Define the properties to be used in the application.
    * - | (4)
      - | Bean definitions related to Spring MVC.
    * - | (5)
      - | Bean definitions related to Spring Security
    * - | (6)
      - | Define the key-values to be used for screen display (internationalization).
    * - | (7)
      - | Store View(jsp) files.

[projectname]-env
--------------------------------------------------------------------------------

The recommended structure of [projectname]-env project is below:

.. code-block:: console

    [projectName]-env
      └src
          └main
              └resources
                  └META-INF
                      └spring
                          ├[projectname]-env.xml ...(1)
                          └[projectname]-infra.properties ...(2)


.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - No.
      - Details
    * - | (1)
      - | Bean definitions that depend on the environment (like dataSource etc).
    * - | (2)
      - | Property definitions which depend on the environment.


.. note::

  The purpose of separating [projectname]-domain and [projectname]-web into different projects is to prevent reverse dependency among them. 
  
  It is natural that [projectname]-web uses [projectname]-domain; however, [projectname]-domain must not refer projectname]-web.
  
  If configruation elements of both, [projectname]-web and [projectname]-domain, are kept in a single project, it may lead to an incorrect reference by mistake. 
  It is strongly recommended to prevent reference to [projectname]-web from [projectname]-domain by separating the project and setting order of reference.

.. note::

  The reason for creating [projectname]-env is to separate the information depending on the environment and thereby enable switching of environment.
  
  For example, set local development environment by default and at the time of building the application, create war file without including [projectname]-env.  
  By creating a separate jar for integration test environment or system test environment, deployment becomes possible just by replacing the jar of corresponding environment. 
  
  Further, it is also possible to minimize the changes in case of project where RDBMS being used changes.
    
  If the above point is not considered, contents of configuration files have to changed according to the target environment and the project has to be re-build.
  
  For further details regarding significance of creating a separate project for environment dependent files, refer to \ :doc:`../Appendix/EnvironmentIndependency`\ .