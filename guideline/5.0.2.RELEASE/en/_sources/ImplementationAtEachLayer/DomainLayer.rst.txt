Domain Layer Implementation
================================================================================

.. only:: html

 .. contents:: Index
    :local:
    :depth: 3

Roles of domain layer
--------------------------------------------------------------------------------
Domain layer \ **implements business logic**\ to be provided to the application layer.

Implementation of domain layer is classified into the following.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 30 60

   * - Sr. No.
     - Classification
     - Description
   * - | 1.
     - | :ref:`domainlayer_entity`
     - | Creation of classes (Entity class) to hold business data.
   * - | 2.
     - | :ref:`repository-label`
     - | Implementation of the methods to operate on business data. These methods are provided to Service classes. 
       | These are in particular the CRUD operations on Entity object.
   * - | 3.
     - | :ref:`service-label`
     - | Implementation of the methods for executing business logic. These methods are provided to the application layer.
       | Business data required by the business logic is fetched as the Entity object through the Repository.

This guideline recommends the structure of creating Entity classes and Repository for the following reasons.

#. \ **By splitting the overall logic into business logic (Service) and the logic to access business data, the implementation scope of business logic gets limited to the implementation of business rules**\ , 
#. \ **Access logic to business data is standardized**\  by consolidating the operations of business data in the Repository. 

 .. note::

    Though this guideline recommends a structure to create Entity classes and Repository, it is not mandatory to perform development in this structure.

    Decide a structure by taking into account the characteristics of the application as well as the project (structure of development team and development methodology).

Flow of development of domain layer
--------------------------------------------------------------------------------
| Flow of development of domain layer and allocation of roles is explained here.
| A case where application is created by multiple development teams is assumed; however, the flow itself remains same even if developed by a single team.

 .. figure:: images/service_implementation_flow.png
    :alt: implementation flow of domain layer
    :width: 100%
    :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 20 80

   * - Sr. No.
     - Team in-charge
     - Description
   * - | (1)
     - | Common development team
     - | Common development team designs and creates Entity classes.
   * - | (2)
     - | Common development team
     - | Common development team works out access pattern for the Entity classes extracted in (1) and designs methods of Repository interface.
       | Common development team should implement the methods to be shared by multiple development teams.
   * - | (3)
     - | Common development team
     - | Common development team provides Entity classes and Repository created in (1) and (2) to the business application development team.
       | At this time, it requests each business application development team to implement the Repository interface.
   * - | (4)
     - | Business application development team
     - | Business application development team takes charge of the implementation of Repository interface.
   * - | (5)
     - | Business application development team
     - | Business application development team develops Service interface and Service class using the Entity class and Repository provided by 
       | the common development team and the Repository implementation class created by the team itself.

 .. warning::

    A system having a large development scope is often developed by assigning the application to multiple teams.
    In that case, it is strongly recommended to provide a common team to design Entity classes and Repository.

    When there is no common team, O/R Mapper(MyBatis, etc.) should be called directly from Service and a method to
    access business data should be adopted without creating Entity classes and Repository .

.. _domainlayer_entity:

Implementation of Entity
--------------------------------------------------------------------------------

Policy of creating Entity class
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Create an Entity using the following method.
| Specific creation method is shown in \ :ref:`domainlayer_entity_example`\ .

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 35 55

   * - Sr. No.
     - Method
     - Supplementary
   * - | 1.
     - | Create Entity class for each table.
     - | However, Entity class is not required for mapping tables which represent the relationship between the tables.
       | Further, when the tables are not normalized, Entity class for each table rule may not be applicable. Refer to the \ :ref:`Warning as well as Note outside this table <domainlayer_entity_policy_warning_note>`\  
       | for the approach related to not-normalized tables.
   * - | 2.
     - | When there is a FK (Foreign Key) in the table, the Entity class of FK destination table must be defined as one of the properties of this Entity.
     - | When there is 1:N relationship with FK destination table, use either \ ``java.util.List<E>``\  or \ ``java.util.Set<E>``\ .
       | The Entity corresponding to the FK destination table is called as the related Entity in this guideline.
   * - | 3.
     - | Treat the code related tables as \ ``java.lang.String``\  rather than as an Entity.
     - | Code related tables are to manage the pairs of code value and name.
       | When there is a need to bifurcate the process as per code values, ``enum`` class corresponding to code value should be created and it must be defined as property.

.. _domainlayer_entity_policy_warning_note:

 .. warning::

    When table is not normalized, **check whether to use the method of creating the Entity classes and Repository** by considering the following points.
    Since the unnormalized tables do not have good compatibility with JPA, it is better not to use JPA.

    * | Creating an appropriate Entity class may often not be possible because of increased difficulty in creating entities if the tables are not normalized.
      | In addition, efforts to create an Entity classes also increases.
      | Two viewpoints must be taken into consideration here. Firstly "Can we assign an engineer who can perform normalization properly?" and secondly "Is it worth taking efforts for creating normalized Entity classes?".
    * | If the tables are not normalized, the logic to fill the gap of differences between the Entity class and structure of table is required in data access.
      | Here the viewpoint to be considered is, "Is it worth taking efforts to fill the gap of differences between the Entity class and structure of table ?".

    The method of creating Entity classes and Repository is recommended; however, the characteristics of the application as well as the project (structure of 
    development team and development methodology) must also be taken into account.
    
.. _domainlayer_entity_policy_note:

 .. note::

    If you want to operate business data as application, and as normalized Entity even if the tables are not normalized,
    it is recommended to use MyBatis as an implementation of RepositoryImpl of the infrastructure layer.

    MyBatis is the O/R Mapper developed to map the SQL with object and not to map the database table record with object. 
    Therefore, mapping to the object independent of table structure is possible depending on the implementation of SQL, .


.. _domainlayer_entity_example:

Example of creating Entity class
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| The creation of Entity class is explained using specific examples.
| Following is an example of creating the business data of Entity classes required for purchasing a product on some shopping site.

Table structure
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
The table structure is as given below:

 .. figure:: images/service_entity_table_layout.png
    :alt: Example of table layout
    :width: 100%
    :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.15\linewidth}|p{0.55\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 15 55

    * - Sr. No.
      - Classification
      - Table name
      - Description
    * - | (1)
      - | Transaction related
      - | t_order
      - | Table to store orders. 1 record is stored for 1 order.
    * - | (2)
      - |
      - | t_order_item
      - | Table to store the products purchased in 1 order. Record of each product is stored when multiple products are purchased in 1 order.
    * - | (3)
      - |
      - | t_order_coupon
      - | Table to store the coupon used in a single order. Record of each coupon is stored when multiple coupons are used in 1 order. No record is stored when coupon is not used.
    * - | (4)
      - | Master related
      - | m_item
      - | Master table to define products.
    * - | (5)
      - |
      - | m_category
      - | Master table to define product category.
    * - | (6)
      - |
      - | m_item_category
      - | Master table to define the category of the product. Mapping between product and category is maintained. Model where 1 product belongs to multiple categories.
    * - | (7)
      - |
      - | m_coupon
      - | Master table to define coupons.
    * - | (8)
      - | Code related
      - | c_order_status
      - | Code table to define order status.


Entity structure
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
If Entity classes are created with the help of policy defined by the above table, it results into the following structure.

 .. figure:: images/service_entity_entity_layout.png
    :alt: Example of entity layout
    :width: 100%
    :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.65\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 15 65

    * - Sr. No.
      - Class name
      - Description
    * - | (1)
      - | Order
      - | Entity class indicating 1 record of t_order table.
        | Multiple \ ``OrderItem``\  and \ ``OrderCoupon``\  are stored as the related Entity.
    * - | (2)
      - | OrderItem
      - | Entity class indicating 1 record of t_order_item table.
        | ``Item`` is stored as the related Entity.
    * - | (3)
      - | OrderCoupon
      - | Entity class indicating 1 record of t_order_coupon table.
        | \ ``Coupon``\  is stored as the related Entity.
    * - | (4)
      - | Item
      - | Entity class indicating 1 record of m_item table.
        | Multiple \ ``Category``\  are stored as the related Entity. The association between \ ``Item``\  and \ ``Category``\  is done using m_item_category table.
    * - | (5)
      - | Category
      - | Entity class indicating 1 record of m_category table.
    * - | (6)
      - | ItemCategory
      - | Entity class is not created since m_item_category table is the mapping table to store the relationship between m_item table and m_category table.
    * - | (7)
      - | Coupon
      - | Entity class indicating 1 record of m_coupon table.
    * - | (8)
      - | OrderStatus
      - | Entity class is not created since c_order_status table is code table.


As it can be observed from the above entity diagram, it might first seem that Order class is the only main entity class
in the shopping site application; however, there are other main entity class as well other than Order class.

Below is the classification of main Entity classes as well as Entity class which are not main. 

 .. figure:: images/service_entity_entity_class_layout.png
    :alt: Example of entity layout
    :width: 100%
    :align: center

|

The following 4 Entities are treated as the main Entity for creating shopping site application.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 30 60

   * - Sr. No.
     - Entity class
     - Reasons for treating as the main Entity.
   * - | (1)
     - | Order class
     - | It is one of the most important Entity class in the shopping site.
       | Order class is the Entity indicating the order itself and a shopping site cannot be created without the Order class.
   * - | (2)
     - | Item class
     - | It is one of the most important Entity class in the shopping site.
       | Item class is the Entity indicating the products handled in the shopping site and a shopping site cannot be created without Item class.
   * - | (3)
     - | Category class
     - | Product categories are displayed usually on the top page or as a common menu in shopping sites. In such shopping sites, Category becomes a main entity.
         Usually operations like 'search category list' can be expected.
   * - | (4)
     - | Coupon class
     - | Often discounts through coupons are offered in the shopping sites as a measure of promoting sales of the products. 
       | In such shopping sites, Coupon becomes a main entity. Usually operations like 'search coupon list' can be expected.


The following are not main Entities for creating shopping site application.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 30 60

   * - Sr. No.
     - Entity class
     - Reason of not treating Entity as main Entity
   * - | (5)
     - | OrderItem class
     - | This class indicates 1 product purchased in 1 order and exists only as the related Entity of Order class.
       | So OrderItem class should not be considered as main Entity.
   * - | (6)
     - | OrderCoupon
     - | This class indicates 1 coupon used in 1 order and exists only as the related Entity of Order class.
       | So, OrderCoupon class should not be considered as main Entity.


.. _repository-label:

Implementation of Repository
--------------------------------------------------------------------------------

Roles of Repository
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Repository has following 2 roles.

1. | **To provide to Service, the operations necessary to control Entity lifecycle (Repository interface).**
   | The operations for controlling Entity lifecycle are CRUD operations.

 .. figure:: images/repository_responsibility_1.png
    :alt: provide access operations to entity
    :width: 100%
    :align: center


2. | **To provide persistence logic for Entity (implementation class of Repository interface).**
   | Entity object should persist irrespective of the lifecycle (start and stop of server) of application.
   | Mostly relational database is the permanent destination of Entity. However, NoSQL database, cache server, external system and file (shared disk) can also be the permanent destination.
   | The actual persistence processing is done using O/R Mapper API.
   | This role is implemented in the RepositoryImpl of the infrastructure layer. Refer to \ :doc:`InfrastructureLayer`\  for details.

 .. figure:: images/repository_responsibility_2.png
    :alt: persist entity
    :width: 100%
    :align: center


Structure of Repository
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Repository consists of Repository interface and RepositoryImpl and performs the following roles.

 .. figure:: images/repository_classes_responsibility.png
   :alt: persist entity
   :width: 100%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.30\linewidth}|p{0.40\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 20 30 40

   * - Sr. No.
     - Class(Interface)
     - Role
     - Description

   * - | (1)
     - | Repository interface
     - | Defines methods to control Entity lifecycle required for implementing business logic (Service).
     - | Defines methods for CRUD operations of the Entity and is not dependent on persistence layer.
       | Repository interface belongs to the domain layer since it plays the roles of defining the operations on Entity required for implementing business logic (Service).

   * - | (2)
     - | RepositoryImpl
     - | Implements the methods defined in Repository interface.
     - | Implements CRUD operations of the Entity and is dependent on persistence layer. Performs actual CRUD processes using API that performs persistence provided by Spring Framework, O/R Mapper and middleware.
       | RepositoryImpl belongs to infrastructure layer since it plays the role of implementing the operations defined in Repository interface.
       | Refer to \ :doc:`InfrastructureLayer`\  for the implementation of RepositoryImpl.


| In case of multiple destinations in persistence layer, the resulting configuration as follows.
| due to this, the logic depending on persistence platform of Entity is hidden from business logic (Service).

 .. figure:: images/repository_not_depends_on.png
   :alt: persist entity
   :width: 100%
   :align: center

 .. note:: **Is it possible to hide 100% of persistence platform dependent logic from the Service class ?**

    In some cases it cannot be hidden completely due to constraints of persistence platform and the libraries used to access the platform.
    As much as possible, platform dependent logic should be implemented in RepositoryImpl instead of Service class.
    When it is difficult to exclude the platform dependent logic and merits of doing so are less,
    persistence platform dependent logic can be implemented as a part of business logic (Service) process.

    A specific example of this is given here. There are cases when unique constraints violation error is needed to be handled when save method
    of ``org.springframework.data.jpa.repository.JpaRepository`` interface provided by Spring Data JPA is called.
    In case of JPA, there is a mechanism of cache entity operations and SQL is executed when transactions are committed.
    Therefore, since SQL is not executed even if save method of JpaRepository is called, unique constraints violation error cannot be handled in logic.
    There is a method (flush method) to reflect cached operations as means to explicitly issue SQLs in JPA.
    saveAndFlush and flush methods are also provided in JpaRepository for the same purpose.
    Therefore, when unique constraints violation error needs to be handled using JpaRepository of Spring Data JPA,
    JPA dependent method (saveAndFlush or flush) must be called.

 .. warning::

    The most important purpose of creating Repository is not to exclude the persistence platform dependent logic from business logic.
    The most important purpose is to limit the implementation scope of business logic (Service) to the implementation of business rules. 
    This is done by separating the operations to access business data in Repository. As an outcome of this, persistence platform dependent logic gets
    implemented in Repository instead of business logic (Service).


Creation of Repository
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Repository must be created using the following policy only.


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 35 55

   * - Sr. No.
     - Method
     - Supplementary
   * - | 1.
     - | Create Repository for the main Entity only.
     - | This means separate Repository for operations of related Entity is not required.
       | However, there are case when it is better to provide Repository for the related Entity in specific applications (for example, 
       | application having high performance requirements etc).
   * - | 2.
     - | Place Repository interface and RepositoryImpl in the same package of domain layer.
     - | Repository interface belongs to domain layer and RepositoryImpl belongs to infrastructure layer. However,
       | Java package of RepositoryImpl can be same as the Repository interface of domain layer.
   * - | 3.
     - | Place DTO used in Repository in the same package as Repository interface.
     - | For example, DTO to store search criteria or summary DTO for that defines only a few items of Entity.


Example of creating Repository
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| An example of creating Repository is explained here.
| An example of creating Repository of Entity class used in the explanation of \ :ref:`domainlayer_entity_example`\  is as follows.


Structure of Repository 
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Entity class used in the explanation of \ :ref:`domainlayer_entity_example`\  is used as an example, the resulting configuration is as follows:

 .. figure:: images/domainlayer_repository_layout.png
   :alt: Example of repository layout
   :width: 100%
   :align: center


| Repository is created for the main Entity class.
| Refer to \ :ref:`application-layering_project-structure`\  for the recommended package structure.


.. _repository-interface-label:

Definition of Repository interface
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Creation of Repository interface
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

An example of creating Repository interface is introduced below.

- :file:`SimpleCrudRepository.java`

 | This interface provides only simple CRUD operations.
 | Method signature is created by referring to \ ``CrudRepository``\  interface and \ ``PagingAndSortingRepository``\  provided by Spring Data.

 .. code-block:: java

     public interface SimpleCrudRepository<T, ID extends Serializable> {
         // (1)
         T findOne(ID id);
         // (2)
         boolean exists(ID id);
         // (3)
         List<T> findAll();
         // (4)
         Page<T> findAll(Pageable pageable);
         // (5)
         long count();
         // (6)
         T save(T entity);
         // (7)
         void delete(T entity);
     }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Method to fetch the Entity object of specified ID.
    * - | (2)
      - | Method to determine if the Entity of specified ID exists or not.
    * - | (3)
      - | Method to retrieve the list of all Entities. In Spring Data, it was \ ``java.util.Iterable``\ . Here as a sample, it is set to \ ``java.util.List``\ .
    * - | (4)
      - | Method to fetch collection of Entity objects corresponding to the specified pagination information (start position, record count, sort information).
        | ``Pageable`` and \ ``Page``\  are the interfaces provided by Spring Data.
    * - | (5)
      - | Method to fetch total number of Entity objects.
    * - | (6)
      - | Method to save (create, update) the specified Entity collection.
    * - | (7)
      - | Method to delete the specified Entity.


- :file:`TodoRepository.java`

 An example of creating Repository of Todo Entity, which was created in tutorial, on the basis of \ ``SimpleCrudRepository``\  interface created above is shown below.

 .. code-block:: java

     // (1)
     public interface TodoRepository extends SimpleCrudRepository<Todo, String> {
         // (2)
         long countByFinished(boolean finished);
     }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description

    * - | (1)
      - | TodoRepository interface is created by specifying Todo entity in the generic type parameter "T" and
        | String class in the generic type parameter "ID".
    * - | (2)
      - | Methods not provided by \ ``SimpleCrudRepository``\  interface are added in this interface.
        | In this case, "Method for acquiring count of Todo entity objects for which specified tasks have been finished" is added.


Method definition of Repository interface
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| It is recommended to have the same signature as \ ``CrudRepository``\  and \ ``PagingAndSortingRepository``\  provided by Spring Data for the methods performing general CRUD operations.
| However, in case of returning collection, (\ ``java.util.Collection``\  or \ ``java.util.List``\ ) interfaces which can be handled in a better way in logic are better than \ ``java.lang.Iterable``\ .
| In real development environment, it is difficult to develop an application using only general CRUD operations. Hence additional methods are required.
| It is recommended to add the methods as per the following rules.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 70

    * - Sr. No.
      - Types of methods
      - Rules
    * - 1.
      - Method for searching a single record
      - #. Method name beginning with \ **findOneBy**\ to indicate that this method fetches a single record that matches with the condition.
        #. In the method name after "findOneBy", physical or logical name of the field used as search condition must be specified. Hence, the method name must be such that it becomes possible to estimate "the kind of entity that can be fetched using this method".
        #. There must be an argument for each search condition. However, when there are many conditions, DTO containing all search conditions can be provided.
        #. Return value must be Entity class.
    * - 2.
      - Method for searching multiple records
      - #. Method name beginning with \ **findAllBy**\ to indicate that this method fetches all the records that matches with the condition.
        #. In the method name after "findAllBy", physical or logical name of the field used as search condition must be specified. Hence, the method name must be such that it becomes possible to estimate "the kind of entity that can be fetched using this method".
        #. There must be an argument for each search condition. However, when there are many conditions, DTO containing all search conditions can be provided.
        #. Return value must be collection of Entity class.
    * - 3.
      - Method for searching multiple records with pagination
      - #. Method name beginning with \ **findPageBy**\ to indicate that this method fetches pages that matches with the condition.
        #. In the method name after "findPageBy", physical or logical name of the field used as search condition must be specified. Hence, the method name must be such that it becomes possible to estimate "the kind of entity that can be fetched using this method".
        #. There must be an argument for each search condition. However, when there are many conditions, DTO containing all search conditions can be provided. ``Pageable`` provided by Spring Data should be the interface for pagination information (start position, record count, sort information).
        #. Return value should be ``Page`` interface provided by Spring Data.
    * - 4.
      - Count related method
      - #. Method name beginning with **countBy** to indicate that this method fetches count of Entities which matches with the condition.
        #. Return value must be long type.
        #. In the method name after "countBy", physical or logical name of the field used as search condition must be specified. Hence, the method name must be such that it becomes possible to estimate "the kind of entity that can be fetched using this method".
        #. There must be an argument for each search condition. However, when there are many conditions, DTO containing all search conditions can be provided.
    * - 5.
      - Method for existence check
      - #. Method name beginning with **existsBy** to indicate that this method checks the existence of Entity which matches with the condition.
        #. In the method name after "existsBy"physical or logical name of the field used as search condition must be specified. Hence, the method name must be such that it becomes possible to estimate "the kind of entity that can be fetched using this method".
        #. There must be an argument for each search condition. However, when there are many conditions, DTO containing all search conditions can be provided.
        #. Return value must be boolean type.

 .. note::

     In case of methods related to update processing, it is recommended to construct methods in the same way as shown above.
     "find" in the method name above can be replaced by "update" or "delete".


- :file:`Todo.java` (Entity)

 .. code-block:: java

     public class Todo implements Serializable {
         private String todoId;
         private String todoTitle;
         private boolean finished;
         private Date createdAt;
         // ...
      }

|

- :file:`TodoRepository.java`

 .. code-block:: java

      public interface TodoRepository extends SimpleCrudRepository<Todo, String> {
          // (1)
          Todo findOneByTodoTitle(String todoTitle);
          // (2)
          List<Todo> findAllByUnfinished();
          // (3)
          Page<Todo> findPageByUnfinished();
          // (4)
          long countByExpired(int validDays);
          // (5)
          boolean existsByCreateAt(Date date);
      }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Example of method that fetches TODO objects whose title matches with specified value (TODO in which todoTitle=[argument value]).
        | Physical name(todoTitle) of condition field is specified after findOneBy.
    * - | (2)
      - | Example of method that fetches unfinished TODO objects (TODO objects where finished=false).
        | Logical condition name is specified after findAllBy.
    * - | (3)
      - | Example of method that fetches pages of unfinished TODOs (TODO objects where finished=false).
        | Logical condition name is specified after findPageBy.
    * - | (4)
      - | Example of method that fetches count of TODO objects for which the finish deadline has already passed (TODO for which createdAt < sysdate - [finish deadline in days] && finished=false).
        | Logical condition name is specified after countBy.
    * - | (5)
      - | Example of method that checks whether a TODO is created on a specific date (createdAt=specified date).
        | Physical name (createdAt) is specified after existsBy.


Creation of RepositoryImpl
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Refer to \ :doc:`InfrastructureLayer`\  for the implementation of RepositoryImpl.


.. _service-label:

Implementation of Service
--------------------------------------------------------------------------------

Roles of Service
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Service plays the following 2 roles.

1. | **Provides business logic to Controller.**
   | Business logic consists of create, update, consistency check etc of business data as well as all the processes related to business logic.
   | Create and update process of business data should be delegated to Repository(or O/R Mapper) and \ **service should be limited to implementation of business rules.**\

 .. note:: **Regarding distribution of logic between Controller and Service**

    In this guideline, the logic to be implemented by Controller and Service should be as per the rules given below.

    1. For the data requested from the client, single item check and correlated item check is to be performed in Controller (Bean Validation or Spring Validator).

    2. Conversion processes (Bean conversion, Type conversion and Format conversion) for the data to be passed to Service, must be performed in Controller instead of Service.

    3. \ **Business rules should be implemented in Service.**\ Access to business data is to be delegated to Repository or O/R Mapper.

    4. Conversion processes (Type conversion and Format conversion) for the data received from Service (data to respond to the client), must be performed in Controller (View class etc).


 .. figure:: images/service_responsibility-of-logic.png
    :alt: responsibility of logic
    :width: 90%
    :align: center


2. | **Declare transaction boundary.**
   | Declare transaction boundary when business logic is performing any operation which requires ensuring data consistency (mainly data update process).
   | Even in case of logic that just read the data, often there are cases where transaction management is required due to the nature of business requirements. In such cases, declare transaction boundary.
   | **Transaction boundary must be set in Service layer as a principle rule.** If it is found to be set in application layer (Web layer), there is a possibility that the extraction of business logic has 
   | not been performed correctly.

 .. figure:: images/service_transaction-boundary.png
    :alt: transaction boundary
    :width: 90%
    :align: center

 Refer to \ :ref:`service_transaction_management`\  for details.


.. _service-constitution-role-label:

Structure of Service class
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| Service consists of Service classes and SharedService classes and plays the following role.
| In this guideline, POJO (Plain Old Java Object) having \ ``@Service``\  annotation is defined as Service or SharedService class. 
| We are not preventing the creation of interface and base classes that limit the signature of methods.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.30\linewidth}|p{0.45\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 15 30 45

   * - Sr. No.
     - Class
     - Role
     - Notes related to dependency relationship

   * - 1.
     - Service class
     - | **Provides business logic to the specific Controller.**
       | Service class methods **must not implement logic that need to be reused.**\
     - #. \ **It is prohibited to call a method of Service class from another Service class method (Figure 1-1).**\ For shared logic, create SharedService class.
       #. Method of Service class can be called from multiple Controllers (Figure 1-2). However, \ **it must be created for each controller when processing is to be branched based on the calling controller.**\ In such a scenario, create a method in SharedService class and call that method from the individual Service class methods.
   * - 2
     - SharedService class
     - | \ **Provides shared (reusable) logic**\ for multiple Controllers and Service classes.
     - #. Methods of other SharedService classes can be called from a SharedService (Figure 2-1). However, **Calling hierarchy should not become complicated.** If calling hierarchy becomes complicated, there is a risk of reduction in maintainability.
       #. Methods of SharedService classes can be called from Controller (Figure 2-2). However, \ **it can be only be done if there is no problem from transaction management perspective.**\ If there is a problem from transaction management perspective, first create a method in Service class and implement transaction management in this method.
       #. \ **It is prohibited to call methods of Service class from SharedService (Figure 2-3).**\


| Dependency relationship of Service class and SharedService class is shown below.
| The numbers inside the diagram are related to the numbering in "Notes related to dependency relationship" column of the above table.

 .. figure:: images/service_class-dependency.png
   :alt: class dependency
   :width: 100%
   :align: center


Reason for separating Service and SharedService
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Logic that cannot be (should not be) reused and logic that can be (should be) reused exist in the business logic.
| To implement these 2 logics in the same class, it is difficult to decide whether a method can be re-used or not.
| To avoid this problem, \ **it is strongly recommended to implement the method to be re-used in the SharedService class**\ in this guideline.


Reason for prohibiting the calling of other Service classes from Service class
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| In this guideline, calling methods of other Service classes from a Service class is prohibited.
| Service provides business logic to a specific controller and is not created with the assumption of using it from other services.
| If it is called directly from other Service classes, the following situations can easily occur \ **and there is a risk of reduced maintainability.**\

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Situations that can occur
   * - 1.
     - | The logic that must be implemented in the calling service class, gets implemented in the called service class for reasons like "having the logic at a single location" etc.
       | As a result, \ **arguments for identifying the caller, get added to the method easily; Ultimately, the logic is incorrectly abstracted out as shared logic (like utilities). It results into a modular structure without much insight.**\
   * - 2.
     - | If the stack patterns or stack of services calling each other is large in number, \ **understanding the impact of modifications in source-code due to change in specifications or bug fixes, becomes difficult.**\


Regarding interface and base classes to limit signature of method
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| In order to bring consistency in development of business logic, interfaces and base classes are created which limit the signature of the methods.
| The purpose is also to prevent the injection of differences due to development style of each developer by limiting the signature through interfaces and base classes.

 .. note::

    In large scale development, there are situations where not every single developer is highly skilled or situations like having consistency in development of business logic considering maintainability after servicing.
    In such situations, limiting the signature through interfaces can be an appropriate decision.
    
    In this guideline, we do not specifically recommended to create interface to limit signature; however, type of architecture must be selected on the basis of characteristics of the project.
    decide the type of architecture taking into account the project properties.

\

| Appendix has a sample of creating interface and base classes to limit the signature.
| Refer to \ :ref:`domainlayer_appendix_blogic`\  for details.


.. _service-creation-unit-label:

Patterns of creating service class
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There are mainly 3 patterns for creating Service.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.25\linewidth}|p{0.50\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 15 25 50

   * - Sr. No.
     - Unit
     - Creation method
     - Description

   * - 1.
     - | For each Entity
     - | Create Service paired with the main Entity.
     - | Main Entity is in other words, business data. **If the application is to be designed and implemented with focus on business data, then Service classes should be created in this way.**
       |
       | If service is created in this way, business logic will also be created for each entity and it will become to extract shared logic.
       | However, if Service is created using this pattern, its affinity is not so good with the type of application which has to be developed 
       | by introducing a large number of developers at the same time. 
       | It can be said that the pattern is suitable when for the developing small or medium sized application.       
   * - 2.
     - | For each use-case
     - | Create Service paired with the use-case.
     - | **If the application is to be designed and implemented with focus on events on the screen, Service should be created in this way.**
       |
       | If the Service is created using this pattern, it is possible to assign a person to each use case; hence, its affinity is good with 
       | the type of application which has to be developed by introducing a large number of developer at the same time.
       | 
       | On the other hand, if Service is created using this pattern, shared logic within use case can be extracted to a single location; 
       | however, shared logic which spans across multiple use-case might not get extracted to a single location.
       | When it is very important to have extract shared logic out to a single location, it becomes necessary devise measures like having 
       | a separate team to look after designing shared components of business logic that span across multiple use cases. 
   * - 3.
     - | For each event
     - | Create Service paired with the events generated from screen.
     - | **If the application is to be designed and implemented with focus on events on the screen and BLogic class is auto-generated using TERASOLUNA ViSC, Service should be created in this way.** 
       | In this guideline, the Service class created using this pattern is called \ ``BLogic``\ .
       |
       | The characteristics of the application if creating Service using this pattern are basically same as those when creating Service for each use case.
       | 
       | However, in this case extracting shared logic out of business logic might get more difficult compared to creation of Service for each use-case (Pattern No. 2).
       | In this guideline, pattern of creating Service for each event is not specifically recommended. However, in large scale development, creating Service using this 
       | pattern can be considered as one of the options with the view of having consistency in development style of business logic from maintainability point of view.

 .. warning::

    **The pattern of Service creation must be decided by taking into account the features of application to be developed and the structure of development team.**

    **It is not necessary to narrow down to any one pattern** out of the 3 indicated patterns.
    Creating Services using different patterns randomly should be avoided for sure; however, **patterns can be used in combinations, if policy of usage of patterns in certain 
    specific conditions has been well-thought decision and has been directed by the architect.**
    For example, the following combinations are possible.

    [Example of usage of patterns in combination]

    * For the business logic very important to the whole application, create as SharedService class for each Entity.
    * For the business logic to be processed for the events from the screen, create as Service class for each Controller.
    * In the Service class for each controller, implement business logic by calling the sharedService as and when required.

 .. tip::

    BLogic is generated directly from design documents when using "TERASOLUNA ViSC".

|

Image of Application development - Creating Service for each Entity
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Following is the image of application development when creating a Service for each Entity.

 .. note::

    An example of a typical application in which a Service is created for each Entity is a REST application. REST application provides CRUD operations (POST, GET, PUT, DELETE of HTTP) 
    for published resources on HTTP. Most of the times, the resources published on HTTP are business data (Entity) or part of business data (Entity), they have good compatibility
    with the pattern of creating Service for each Entity.

    In case of REST application, most of the times, use-cases are also extracted on a "per Entity" basis. Hence, the structure is similar to the case when Service is created for on a "per use-case" basis.

|

 .. figure:: images/service_unit_resource.png
   :alt: multiple controller unit
   :width: 100%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Implement Service by assigning a person for each Entity.
       | If there is no specific reason, it is desirable that Controller must also be created for each Entity and must be developed by the same developer who created the Service class.
   * - | (2)
     - | Implement SharedService if there is shared logic between multiple business logics.
       | In the above figure, different person is assigned as the in-charge. However, he may be the same person as (1) depending per the project structure.

|


Image of Application development - Creating Service for each use case
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Following is the image of application development when creating a Service for each use-case.
| In case of use-case which performs CRUD operations on the Entity, structure is same as in case of creating Service for each Entity.


 .. figure:: images/service_unit_controller.png
   :alt: controller unit
   :width: 100%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Implement Service by assigning a person for each use-case.
       | If there is no specific reason, it is desirable that Controller must also be created for each use-case and must be developed by the same developer who created the Service class.
   * - | (2)
     - | Implement SharedService if there is shared logic between multiple business logics.
       | In the above figure, different person is assigned as the in-charge. However, he may be the same person as (1) depending per the project structure.

 .. note::

    With an increase in the size of the use-cases, the development scope of a person increases. At such a point of time, it becomes difficult to divide the work of this use-case
    with other developers. In case of application which has to be developed by introducing a large number of developer at the same time, the use-case can be further split into finer
    use-cases and which can then be allocated to more number of developers.  

|

| Below is the image of application development when the use-case is further split.
| Splitting a use-case has no impact on SharedService. Hence, the explanation is omitted here.

 .. figure:: images/service_unit_controller2.png
   :alt: multiple controller unit
   :width: 100%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Divide the use-case into finer processes which make-up the complete use-case. Assign each fine process to a developer. Each developer creates the Service for assigned process.
       | Note that the processes here are operations like search, create, update, delete etc. and these processes do not have a direct mapping to the processing required to be done for 
       | each event generated on screen. 
       | For example, if it the event generated on screen is "Update", it includes multiple finer processes such as "Fetching the data to be updated", "Compatibility check of update contents" etc.
       | If there is no specific reason, it is desirable that Controller must also be created for each of these finer processes and must be developed by the same developer who creates the Service class.

 .. tip::

    In some projects, "group of use-cases" and "use-cases" are used in place of "use-case" and "processes" used in this guideline.

|

Image of Application development - Creating Service for each use event
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Following is the image of application development when creating a Service(BLogic) for each event.

 .. figure:: images/service_unit_business-ligic.png
   :alt: constitution image of business logic unit
   :width: 100%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Implement Service(BLogic) by assigning a person for each event.
       | Above example is an extreme case where separate developer is assigned for each service(BLogic).
       | In reality, a single person must be assigned for a use-case.
   * - | (2)
     - | If there is no specific reason, controller also should be created on "per use-case" basis.
   * - | (3)
     - | Even if the separate Service(BLogic) is created for each event, it is recommended that same person is the in-charge of the complete use-case.
   * - | (4)
     - | Implement in SharedService to share the logic with multiple business logics.
       | In the above figure, different person is assigned as the in-charge. However, he may be the same person as (1) depending per the project structure.

 .. note::

    With an increase in the size of the use-cases, the development scope of a person increases. At such a point of time, it becomes difficult to divide the work of this use-case
    with other developers. In case of application which has to be developed by introducing a large number of developer at the same time, the use-case can be further split into finer
    use-cases and which can then be allocated to more number of developers.  

|

| Below is the image of application development when the use-case is further split.
| Splitting a use-case has no impact on SharedService. Hence, the explanation is omitted here.

 .. figure:: images/service_unit_business-ligic2.png
   :alt: multiple controller unit
   :width: 100%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Divide the use-case into finer processes which make-up the complete use-case. Assign each fine process to a developer. Each developer creates the Service for assigned process.
       | Note that the processes here are operations like search, create, update, delete etc. and these processes do not have a direct mapping to the processing required to be done for 
       | each event generated on screen. 
       | For example, if it the event generated on screen is "Update", it includes multiple finer processes such as "Fetching the data to be updated", "Compatibility check of update contents" etc.
       | If there is no specific reason, it is desirable that Controller must also be created for each of these finer processes and must be developed by the same developer who creates the Service class.

.. _service-class-label:

Creation of Service class
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _service-class-creation-label:

Methods of creating Service class
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Below are the points to be taken care of while creating Service class.

- Creation of Service interface

 .. code-block:: java

    public interface CartService { // (1)
        // omitted
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | **It is recommended to create Service interface.**
       | By providing an interface, it is possible to execute the method published as Service explicitly.

\

 .. note:: **Merits from architecture perspective**

    #. If interface is there, When using AOP, Dynamic proxies functionality of standard JDK is used.
       In case of no interface, CGLIB included in Spring Framework is used. In case of CGLIB there are certain restrictions like "Advice cannot be applied on final methods" etc.
       Refer to \ `Spring Reference Document -Aspect Oriented Programming with Spring(Proxying mechanisms)- <http://docs.spring.io/spring/docs/4.1.9.RELEASE/spring-framework-reference/html/aop.html#aop-proxying>`_\ for details.
    #. It becomes easier to create a stub of business logic. When application layer and domain layer are developed in parallel using different development teams, stubs of Service
       are required. When there is a need to create stubs, it is recommended to have interface .

- Creation of Service class

 .. code-block:: java

    @Service // (1)
    @Transactional // (2)
    public class CartServiceImpl implements CartService { // (3) (4)
        // omitted
    }

 .. code-block:: xml

    <context:component-scan base-package="xxx.yyy.zzz.domain" /> <!-- (1) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | **Add @Service annotation to class.**
       | By adding the above annotation, bean definition in configuration file is not required.
       | Specify package for component scanning in ``base-package`` attribute of <context:component-scan> element.
       | In case of this example, all the classes in "xxx.yyy.zzz.domain" is registered in container.
   * - | (2)
     - | **Add @Transactional annotation to class.**
       | By adding the above annotation, transaction boundary is set for to the all the methods of the Service class.
       | ``value`` attribute should be specified as required.
       | Refer to \ :ref:`transaction-management-declare-transaction-info-label`\  for details.

       | Moreover, to understand the points to be noted when using \ ``@Transactional``\  annotation, it is advisable to confirm with ":ref:`DomainLayerAppendixTransactionManagement`".
   * - | (3)
     - | **Consider interface name as XxxService and class name as XxxServiceImpl.**
       | Any naming conventions can be used. However, it is recommended to use distinguishable naming conventions for Service class and SharedService class.
   * - | (4)
     - | **Service class must not maintain state. Register it in container as bean of singleton scope .**
       | Objects (POJO such as Entity/DTO/VO) and values (primitive type, primitive wrapper class) where state changes in each thread should not be maintained in class level fields.
       | Setting scope to any value other than singleton (prototype, request, session)  using ``@Scope``  annotation is also prohibited.

\

 .. note:: **Reason for adding @Transactional annotation to class**

    Transaction boundary is required only for the business logic that updates the database. However, it is recommended to apply the annotation at class level to prevent bugs due to skipped annotation.
    However, defining \ ``@Transactional``\  annotation only at required places (methods which update the database) is also fine.

 .. note:: **Reason to prohibit non-singleton scopes**

    #. prototype, request, session are the scopes for registering bean that maintains state. Hence they must not be used in Service class.
    #. When scope is set to request or prototype, performance is affected as the bean generation frequency is high in DI container.
    #. When scope is set to request or session, it cannot be used in non Web applications (for example, Batch application).

.. _service-class-method-creation-label:

Creation of methods of Service class
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Below are the points to be taken care of while writing methods of Service class.

- Creation of method of Service interface

 .. code-block:: java

    public interface CartService {
        Cart createCart(); // (1) (2)
        Cart findCart(String cartId); // (1) (2)
    }

- Creation of methods of Service class

 .. code-block:: java

    @Service
    @Transactional
    public class CartServiceImpl implements CartService {

        @Inject
        CartRepository cartRepository;

        public Cart createCart() { // (1) (2)
            Cart cart = new Cart();
            // ...
            cartRepository.save(cart);
            return cart;
        }

        @Transactional(readOnly = true) // (3)
        public Cart findCart(String cartId) { // (1) (2)
            Cart cart = cartRepository.findByCartId(cartId);
            // ...
            return cart;
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | **Create a method of Service class for each business logic.**
   * - | (2)
     - | **Define methods in Service interface and implement business logic in its implementation class.**
   * - | (3)
     - | **Add @Transactional annotation for changes to default transaction definition (class level annotation).**
       | Attributes should be specified as per the requirement.
       | Refer to \ :ref:`transaction-management-declare-transaction-info-label` for details.

       | Moreover, to understand the points to be noted when using \ ``@Transactional``\  annotation, it is advisable to confirm with ":ref:`DomainLayerAppendixTransactionManagement`".

\

 .. tip:: **Transaction definition of business logic for reference**

    When business logic for reference is to be implemented, by specifying \ ``@Transactional(readOnly = true)``\ ,
    instruction can be given to run the SQL under "Read-only transactions" for JDBC driver.

    The way of handling read-only transactions depends on the implementation of JDBC driver; hence, confirm the specifications of JDBC driver to be used.


 .. note:: **Points to be noted when using "Read-only transactions"**

    If it is set to "perform health check" when retrieving a connection from connection pool, "Read-only transactions" may not be enabled.
    For details on this event and to avoid the same, refer to :ref:`About cases where "Read-only transactions" are not enabled <DomainLayerTransactionManagementWarningDisableCase>`.

 .. note:: **Transaction definition when a new transaction is required to be started**

    Set \ ``@Transactional(propagation = Propagation.REQUIRES_NEW)``\  to start a new transaction
    without participating in the transaction of the caller method.

.. _service-class-method-args-return-label:

Regarding arguments and return values of methods of Service class 
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
The below points must be considered for arguments and return values of methods of Service class.

| Serializable classes (class implementing \ ``java.io.Serializable``\ ) must be used for arguments and return values of Service class.
| Since there is possibility of Service class getting deployed as distributed application, it is recommended to allow only Serializable class.

**Typical arguments and return values of the methods are as follows.**

 * Primitive types (\ ``int``\ , \ ``long``\ )
 * Primitive wrapper classes (\ ``java.lang.Integer``\ , \ ``java.lang.Long``\ )
 * java standard classes (\ ``java.lang.String``\ , \ ``java.util.Date``\ )
 * Domain objects (Entity, DTO)
 * Input/output objects (DTO)
 * Collection (implementation class of \ ``java.util.Collection``\ ) of above types
 * void
 * etc ...

\

 .. note:: **Input/Output objects**

     #. Input object indicates the object that has all the input values required for executing Service method.
     #. Output object indicates the object that has all the execution results (output values) of Service method.

      If business logic(BLogic class) is generated using "TERASOLUNA ViSC" then, input and output objects are used as argument and return value of the of BLogic class.

**Values that are forbidden as arguments and return values are as follows.**

 * Objects (``javax.servlet.http.HttpServletRequest`` , ``javax.servlet.http.HttpServletResponse`` , ``javax.servlet.http.HttpSession`` , ``org.springframework.http.server.ServletServerHttpRequest``) which are dependent on implementation architecture of application layer (Servlet API or web layer API of Spring).
 * Model(Form, DTO) classes of application layer
 * Implementation classes of ``java.util.Map``

 .. note:: **Reason for prohibition**

    #. If objects depending on implementation architecture of application layer are allowed, then application layer and domain layer get tightly coupled.
    #. \ ``java.util.Map``\  is too generalized. Using it for method arguments and return values makes it difficult to understand what type of object is stored inside it. Further, since the values are managed using keys, the following problems may occur.

     * Values are mapped to a unique key and hence cannot be retrieved by specifying a key name which is different from the one specified at the time of inserting the value.
     * When key name has to be changed, it becomes difficult to determine the impacted area.


**How to sharing the same DTO between the application layer and domain layer is shown below.**

* DTO belonging to the package of domain layer can be used in application layer.

\

 .. warning::

   Form and DTO of application layer should not be used in domain layer.

.. _shared-service-class-label:

Implementation of SharedService class
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _shared-service-class-creation-label:

Creation of SharedService class
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Below are the points to be taken care of while creating SharedService class.
| Only the points which are different from Service class are explained here.

#. | **Add @Transactional annotation to class as and when required.**
   | \ ``@Transactional``\  annotation is not required when data access is not involved.

#. | **Interface name should be XxxSharedService and class name should be XxxSharedServiceImpl.**
   | Any other naming conventions can also be used. However, it is recommended to use distinguishable naming conventions for Service class and SharedService class.

.. _shared-service-class-method-creation-label:

Creation of SharedService class method
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Below are the points to be taken care of while writing methods of SharedService class.
| Only the points which are different from Service class are explained here.

#. **Methods in SharedService class must be created for each logic which is shared between multiple business logics.**

#. | **Add @Transactional annotation to class as and when required.**
   | Annotation is not required when data access is not involved.

.. _shared-service-class-method-args-return-label:

Regarding arguments and return values of SharedService class method
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Points are same as :ref:`service-class-method-args-return-label`.

.. _service-implementation-label:

Implementation of logic
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Implementation in Service and SharedService is explained here.

Service and SharedService has implementation of logic related to operations such as data fetch, update, consistency check  of business data 
and implementation related to business rules.

Example of a typical logic is explained below.

Operate on business data
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Refer to the following for the examples of data (Entity) fetch and update.

* When using MyBatis3, \ :doc:`../ArchitectureInDetail/DataAccessMyBatis3`\ 
* When using JPA, \ :doc:`../ArchitectureInDetail/DataAccessJpa`\ 


.. _service-return-message-label:

Returning messages
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Warning message and business error message are the two type of messages which must be resolved in Service (refer to the figure in red broken line below).
| Other messages should be resolved in application layer.
| Refer to \ :doc:`../ArchitectureInDetail/MessageManagement`\  for message types and message pattern.

 .. figure:: images/service_target-resolving-message.png
   :alt: target of resolving message
   :width: 100%
   :align: center

\

 .. note:: **Regarding resolving message**

    In service, instead of the actual message \ **the information required for building the message  (message code, message insert value) is resolved**\.

Refer to the following for detailed implementation method.

* :ref:`service-return-warnmessage-label`
* :ref:`service-return-businesserrormessage-label`



.. _service-return-warnmessage-label:

Returning warning message
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Message object must be returned for warning message. If domain object such as Entity needs to be returned with it,
| message object and domain object should inserted into output object (DTO) and this output object must be returned.

| Message object ( ``org.terasoluna.gfw.common.message.ResultMessages`` ) is provided as common library. When the class provided in common library
| does not fulfill the requirements, message object should be created for each project.

- Creation of DTO

 .. code-block:: java

    public class OrderResult implements Serializable {
        private ResultMessages warnMessages;
        private Order order;

        // omitted

    }

|

- Implementation of method of Service class

  Following is an example of implementation of displaying a warning message.
  The message is "Products may not be delivered together since the order includes products which are not available right now".

 .. code-block:: java

    public OrderResult submitOrder(Order order) {

        // omitted

        boolean hasOrderProduct = orderRepository.existsByOrderProduct(order); // (1)

        // omitted

        Order order = orderRepository.save(order);

        // omitted

        ResultMessages warnMessages = null;
        // (2)
        if(hasOrderProduct) {
            warnMessages = ResultMessages.warn().add("w.xx.xx.0001");
        }
        // (3)
        OrderResult orderResult = new OrderResult();
        orderResult.setOrder(order);
        orderResult.setWarnMessages(warnMessages);
        return orderResult;
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | When the order includes products which are not available right now, set  \ ``hasOrderProduct``\  to \ ``true``\ .
   * - | (2)
     - | In the above example, when the order includes products which are not available right now, a warning message occurs.
   * - | (3)
     - | In the above example, the registered \ ``Order``\  object and warning message are returned by storing objects in a DTO called \ ``OrderResult``\ .

.. _service-return-businesserrormessage-label:

Notifying business error
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Business exception is thrown when business rules are violated while executing business logic.
| The following can be the cases.

-  When reservation date exceeds deadline while making tour reservation
-  When the product is out of stock at the time of placing an order
-  etc ...

| Business exception ( ``org.terasoluna.gfw.common.exception.BusinessException`` ) is provided as common library.
| When business exception class provided in common library does not fulfill the requirements, business exception class should be created in the project.
| **It is recommended to create business exception class as subclass of java.lang.RuntimeException**.

\

 .. note:: **Reason for considering business exception as an unchecked exception**

   Since business exceptions need to be handled in controller class, they can be configured as checked exception.
   However in this guideline, it is recommended that business exception be subclass of unchecked exception (``java.lang.RuntimeException``). By default, 
   if there is a RuntimeException, transaction will be rolled back. Hence, doing this will prevent leaving a bug in the source-code due to inadequate settings of @Transactional annotation. 
   Obviously, if settings are changed such that transaction rollbacks even in case checked exceptions, business exception can be configured as subclass of checked exceptions.

| Example of throwing business exception.
| Below example notifies that reservation is past the deadline and a business error.

 .. code-block:: java

    // omitted

    if(currentDate.after(reservationLimitDate)) { // (1)
        throw new BusinessException(ResultMessages.error().add("e.xx.xx.0001"));
    }

    // omitted

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description

   * - | (1)
     - Business exception is thrown since reservation date is past the deadline at the time of making reservation.

Refer to \ :doc:`../ArchitectureInDetail/ExceptionHandling`\  for details of entire exception handling.

.. _service-return-systemerrormessage-label:

Notifying system error
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| System exception is thrown when error occurs in system while executing business logic.
| The following can be the cases.

-  When master data, directories and files that should already exist, do not exist
-  When a checked exception generated by a library method is caught and this exception indicates abnormal system state.
-  etc ...

| System exception (\ ``org.terasoluna.gfw.common.exception.SystemException``\ ) is provided as common library.
| When system exception class provided in common library does not fulfill the requirements, system exception class should be created in the project.
| **It is recommended to create system exception class as subclass of java.lang.RuntimeException**.
| The reason is system exception should not be handled by application code and rollback target of \ ``@Transactinal``\  annotation is set to \ ``java.lang.RuntimeException``\  by default.

| Example of throwing system exception.
| Example notifying the non-existence of the specific product in product master as system error is shown below.

 .. code-block:: java

    ItemMaster itemMaster = itemMasterRepository.findOne(itemCode);
    if(itemMaster == null) { // (1)
        throw new SystemException("e.xx.fw.0001",
            "Item master data is not found. item code is " + itemCode + ".");
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description

   * - | (1)
     - System exception is thrown since master data that should already exist does not exist. Example of case when system error is detected in logic）

Example that throws system exception while catching IO exception while copying the file is shown below.

 .. code-block:: java

    // ...

    try {
        FileUtils.copy(srcFile, destFile);
    } catch(IOException e) { // (1)
        throw new SystemException("e.xx.fw.0002",
            "Failed file copy. src file '" + srcFile + "' dest file '" + destFile + "'.", e);
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | System exception that is classified into invalid system state is thrown by the library method.
       | **The exception generated by library must be passed to system exception class as cause exception.**
       | If cause exception is lost, error occurrence location and basic error cause cannot be traced from the stack trace.

\

 .. note:: **Regarding handling of data access error**

    When data access error occurs in Repository and O/R Mapper while executing business logic, it is converted to subclass of 
    \ ``org.springframework.dao.DataAccessException``\  and thrown.
    Error can be handled in application layer instead of catching in business logic.
    However, some errors like unique constraints violation error should be handled in business logic as per business requirements.
    Refer to \ :doc:`../ArchitectureInDetail/DataAccessCommon`\  for details.

.. _service_transaction_management:

Regarding transaction management
--------------------------------------------------------------------------------
Transaction management is required in the logic where data consistency must be ensured.

Method of transaction management
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
There are various transaction management methods. However, in this guideline, \ **it is recommended to use "Declarative Transaction Management" provided by Spring Framework.**\

Declarative transaction management
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
In "Declarative transaction management", the information required for transaction management can be declared by the following 2 methods.

* Declaration in XML(bean definition file).
* **Declaration using annotation (@Transactional) (Recommended).**

Refer to \ `Spring Reference Document -Transaction Management(Declarative transaction management)- <http://docs.spring.io/spring/docs/4.1.9.RELEASE/spring-framework-reference/html/transaction.html#transaction-declarative>`_\  for
the details on "Declarative type transaction management" provided by Spring Framework.
\

 .. note:: **Reason for recommending annotation method**

    #. The transaction management to be performed can be understood by just looking at the source code.
    #. AOP settings for transaction management is not required if annotations are used and so XML becomes simple.

.. _transaction-management-declare-transaction-info-label:

Information required for "Declarative transaction management"
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Specify \ ``@Transactional``\  annotation for at class level or method level which are considered as target of transaction management and specify the information required for
| transaction control in attributes of \ ``@Transactional``\  annotation.

 .. note::

    In this guideline, it is a prerequisite to use \ ``@org.springframework.transaction.annotation.Transactional`` \  annotation provided by Spring Framework.

 .. tip::

    From Spring 4, it is possible to use \ ``@javax.transaction.Transactional`` \  annotation added from JTA 1.2.

    However, in this guideline, it is recommended to use an annotation of Spring Framework that can specify the information required for "Declarative transaction management" in a much more detailed way.

    Following attributes can be specified if Spring Framework annotation is used.

    * \ ``NESTED``\ (JDBC savepoint) as an attribute value of the propagation method of transaction (\ ``propagation``\  attribute).
    * Isolation level of transaction (\ ``isolation``\  attribute)
    * Timeout period of transaction (\ ``timeout``\  attribute)
    * Read-only flag of transaction (\ ``readOnly``\  attribute)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 10 80

    * - Sr. No.
      - Attribute name
      - Description

    * - 1
      - propagation
      - | Specify transaction propagation method.
        |
        | **[REQUIRED]**
        | Starts transaction if not started. (default when omitted)
        | **[REQUIRES_NEW]**
        | Always starts a new transaction.
        | **[SUPPORTS]**
        | Uses transaction if started. Does not use if not started.
        | **[NOT_SUPPORTED]**
        | Does not use transaction.
        | **[MANDATORY]**
        | Transaction should start. An exception occurs if not started.
        | **[NEVER]**
        | Does not use transaction (never start). An exception occurs if started.
        | **[NESTED]**
        | save points are set. They are valid only in JDBC.
    * - 2
      - isolation
      - | Specify isolation level of transaction.
        | Since this setting depends on DB specifications, settings should be decided by checking DB specifications.
        |
        | **[DEFAULT]**
        | Isolation level provided by DB by default.(default when omitted)
        | **[READ_UNCOMMITTED]**
        | Reads (uncommitted) data modified in other transactions.
        | **[READ_COMMITTED]**
        | Does not read (uncommitted) data modified in other transactions.
        | **[REPEATABLE_READ]**
        | Data read by other transactions cannot be updated.
        | **[SERIALIZABLE]**
        | Isolates transactions completely.
        |
        | Isolation level of transaction is considered as the parameter related to exclusion control.
        | Refer to \ :doc:`./../ArchitectureInDetail/ExclusionControl`\  for exclusion control.
    * - 3
      - timeout
      - | Specify timeout of transaction (seconds).
        | -1 by default (Depends on specifications and settings of DB to be used)
    * - 4
      - readOnly
      - | Specify Read-only flag of transaction.
        | false by default (Not read-only)
    * - 5
      - rollbackFor
      - | Specify list of exception classes to rollback transactions.
        | Blank by default (Not specified)
    * - 6
      - rollbackForClassName
      - | Specify list of exception class names to rollback transactions.
        | Blank by default (Not specified)
    * - 7
      - noRollbackFor
      - | Specify list of exception classes to commit transactions.
        | Blank by default (Not specified)
    * - 8
      - noRollbackForClassName
      - | Specify list of exception classes to commit transactions.
        | Blank by default (Not specified)

\

 .. note:: **Location to specify the @Transactional annotation**

    **It is recommended to specify the annotation at the class level or method level of the class.**
    Must be noted that it should not interface or method of interface.
    For the reason, refer to 2nd Tips of \ `Spring Reference Document -Transaction Management(Using @Transactional)- <http://docs.spring.io/spring/docs/4.1.9.RELEASE/spring-framework-reference/html/transaction.html#transaction-declarative-annotations>`_\ .

 .. warning:: **Default operations of rollback and commit when exception occurs**

    When rollbackFor and noRollbackFor is not specified, Spring Framework performs the following operations.

    * Rollback when unchecked exception of (java.lang.RuntimeException and java.lang.Error) class or its subclass occurs.
    * Commit when checked exception of (java.lang.Exception) class or its subclass occurs. \ **(Necessary to note)**\

 .. note:: **Regarding value attributes of @Transactional annotation**

    There is a value attribute in \ ``@Transactional``\  annotation. However, this attribute specifies which Transaction Manager to be used when multiple Transaction Managers are declared.
    It is not required to specify when there is only one Transaction Manager.
    When multiple Transaction Managers need to be used, refer to \ `Spring Reference Document -Transaction Management(Multiple Transaction Managers with @Transactional)- <http://docs.spring.io/spring/docs/4.1.9.RELEASE/spring-framework-reference/html/transaction.html#tx-multiple-tx-mgrs-with-attransactional>`_\ .

 .. note:: **Default isolation levels of main DB are given below.**

    Default isolation levels of main DB are given below.

    * Oracle : READ_COMMITTED
    * DB2 : READ_COMMITTED
    * PostgreSQL : READ_COMMITTED
    * SQL Server : READ_COMMITTED
    * MySQL : REPEATABLE_READ

.. _DomainLayerTransactionManagementWarningDisableCase:

 .. note:: **Cases where "Read-only transactions" are not enabled**

    A mechanism is provided to run SQL under "Read-only transactions" by specifying \ ``readOnly = true``\ ; however,
    when all of the following conditions are satisfied, there will be a JDBC driver where "Read-only transactions" are not enabled.

    **[Conditions to generate this event]**

    * Perform health check when retrieving a connection from connection pool.
    * Disable 'Auto commit' of connection retrieved from connection pool.
    * Use \ ``DataSourceTransactionManager``\  or \ ``JpaTransactionManager``\  as \ ``PlatformTransactionManager``\ . (This event does not occur when using \ ``JtaTransactionManager``\ ).

    **[JDBC driver where occurrence of this event is confirmed]**

    * ``org.postgresql:postgresql:9.3-1102-jdbc41`` (JDBC 4.1 compatible JDBC driver for PostgreSQL 9.3)

    **[Method to avoid this event]**

    In a case where "Read-only transactions" are not enabled,
    if \ ``readOnly = true``\  is specified, it ends up carrying out the unnecessary processes.
    Therefore, it is recommended to execute the SQL under "Updatable transactions" even for the reference processes.

    Other methods to avoid this event are as follows:

    * Do not perform health check when retrieving a connection from connection pool.
    * Enable 'Auto commit' of the connection retrieved from connection pool. (Disable 'Auto commit' only when transaction management is required)

    However, However, do not change the design for 'health check' and 'auto commit' to avoid this event.

    **[Remarks]**

    * Reproduction of this event is confirmed on PostgreSQL 9.3 and Oracle 12c. It is not performed on any other database and versions.
    * In PostgreSQL 9.3, \ ``SQLException``\  occurs when \ ``java.sql.Connection#setReadOnly(boolean)``\  method is called.
    * When SQL or API call of JDBC is logged in using \ :ref:`log4jdbc <DataAccessCommonDataSourceDebug>`\ , \ ``SQLException``\  occurred from JDBC driver is output to log with ERROR level.
    * **SQL Exception occurred from JDBC driver is ignored by exception handling of Spring Framework. Hence, even though it is not an error as application behavior, the "Read-only transactions" is not enabled.**
    * Occurrence of this event is not confirmed in Oracle 12c.

    **[Reference]**

    When following log is output using \ :ref:`log4jdbc <DataAccessCommonDataSourceDebug>`\ , it will be treated as a case corresponding to this event.

     .. code-block:: text

        date:2015-02-20 16:11:56	thread:main	user:	X-Track:	level:ERROR	logger:jdbc.audit                                      	message:3. Connection.setReadOnly(true)
        org.postgresql.util.PSQLException: Cannot change transaction read-only property in the middle of a transaction.
            at org.postgresql.jdbc2.AbstractJdbc2Connection.setReadOnly(AbstractJdbc2Connection.java:741) ~[postgresql-9.3-1102-jdbc41.jar:na]
            ...


Propagation of transaction
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| In most of the cases, Propagation method of transaction is "REQUIRED".
| However, since  **"REQUIRES_NEW" is also used according to the requirements of Application**, transaction control flow in case of "REQUIRED" and "REQUIRES_NEW" is explained below.
| The explanation of other propagation methods is omitted in this guideline since their usage frequency is very low.

| **Transaction control flow when propagation method of transaction is set to "REQUIRED"**
| When propagation method of transaction is set to "REQUIRED", all sequential processes called from controller are processed in the same transaction.

 .. figure:: images/service_transaction-propagation-required.png
    :alt: transaction management flow of REQUIRED
    :width: 100%
    :align: center

#. Controller calls a method of Service class.
   At this time, since started transaction does not exist, transaction is started using \ ``TransactionInterceptor``\ .
#. \ ``TransactionInterceptor``\  calls the method of service class after starting the transaction.
#. Service class calls a method of \ ``SharedService``\ . This method is also under transaction control.
   At this time, though started transaction exists, \ ``TransactionInterceptor``\  participates in the started transaction without starting a new transaction.
#. \ ``TransactionInterceptor``\  calls the method under transaction control after participating in the started transaction.
#. \ ``TransactionInterceptor``\  performs commit or rollback according to result of processing and ends the transaction.


.. note:: **Reason for occurrence of org.springframework.transaction.UnexpectedRollbackException**

  When propagation method of transaction is set to "REQUIRED", though there is only one physical transaction, internally Spring Framework creates transaction boundaries.
  In case of above example, when a method of SharedService is called, a TransactionInterceptor is started which internally provides transaction control boundary at SharedService level.
  Therefore, when an exception (which is set as target of rollback) occurs in \ ``SharedService``\  method, status of transaction is set to rollback (rollback-only) by \ ``TransactionInterceptor``\ .
  This transaction now cannot be committed.
  Going further, if the Service method tries to commit this transaction due to conflicting settings of rollback target exception between Service method and SharedShared method,
  \ ``UnexpectedRollbackException``\  is generated by Spring Framework notifying that there is inconsistency in transaction control settings. When UnexpectedRollbackException is generated, it should be checked that there is no inconsistency in rollbackFor and noRollbackFor settings.

| **Transaction management flow when propagation method of transaction is set to "REQUIRES_NEW"**
| When propagation method of transaction is set to "REQUIRES_NEW", a part of the sequence of processing (Processing done in SharedService) are processed in another transaction when called from Controller.

 .. figure:: images/service_transaction-propagation-requires_new.png
    :alt: transaction management flow of REQUIRES_NEW
    :width: 100%
    :align: center

#. Controller calls a method of Service class. This method is under transaction control. At this time, since started transaction does not exist, transaction is started by ``TransactionInterceptor`` (Hereafter, the started transaction is referred as "Transaction A").
#. ``TransactionInterceptor`` calls the method of service class after transaction (Transaction A) is started.
#. Service class calls a method of ``SharedService`` class. At this time, though started transaction (Transaction A) exists, since propagation method of transaction is "REQUIRES_NEW", new transaction is started by \ ``TransactionInterceptor``\. (Hereafter, started transaction is referred as "Transaction B"). At this time, "Transaction A" is interrupted and the status changes to 'Awaiting to resume'.
#. \ ``TransactionInterceptor``\  calls the method of SharedService class after Transaction B is started.
#. \ ``TransactionInterceptor``\  performs commit or rollback according to the process result and ends the Transaction B.
   At this time, "Transaction A" is resumed and status is changed to Active.
#. \ ``TransactionInterceptor``\  performs commit or rollback according to the process result and ends the Transaction A.

Way of calling the method which is under transaction control
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Since "Declarative transaction management" provided by Spring Framework is implemented using AOP, transaction management is applied only for method calls for which AOP is enabled.
| Since the default mode of AOP is **"proxy" mode, transaction control will be applied only when public method is called from another class.**\
| Note that \ **transaction control is not applied if the target method is called from an internal method even if the target method is a public method**\.

- **Way of calling the method which is under transaction control**

 .. figure:: images/service_transaction-valid-call.png
   :alt: enabled method calls of transaction management
   :width: 100%
   :align: center

- **Way of calling the method which is not under transaction control**

 .. figure:: images/service_transaction-invalid-call.png
   :alt: not enabled method calls of transaction management
   :width: 100%
   :align: center

 .. note:: **In order to bring internal method calls under transaction control**

   It is possible to enable transaction control for internal method calls as well by setting the AOP mode to \ ``"aspectj"``\ .
   However, if internal method call of transaction management is enabled, the route of transaction management may become complicated;
   hence it is recommended to use the default "proxy" for AOP mode.

.. _service_enable_transaction_management:

Settings for using transaction management
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The settings required for using transaction management are explained.

PlatformTransactionManager settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| In order to have transaction management done, it is necessary to define the bean of \ ``PlatformTransactionManager``\ .
| Spring Framework has provided  couple of classes based on the purpose; any class can be specified that meets the requirement of the application.

- :file:`xxx-env.xml`

 Example of settings for managing the transaction using JDBC connection which is fetched from DataSource is given below.

 .. code-block:: xml

     <!-- (1) -->
     <bean id="transactionManager"
           class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
         <property name="dataSource" ref="dataSource" />
     </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description

    * - | (1)
      - | Specify the implementation class of PlatformTransactionManager.
        | It is recommended to set id as "transactionManager".

\

 .. note:: **When transaction management (Global transaction management) is required for multiple DBs (Multiple resources)**

     * It is necessary to use \ ``org.springframework.transaction.jta.JtaTransactionManager``\  and manage transactions by using JTA functionality provided by application server.
     * When JTA is to be used in WebSphere and Oracle WebLogic Server, a \ ``JtaTransactionManager``\  which is extended for the application server is automatically set
       by specifying <tx:jta-transaction-manager/>.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table:: **Implementation class of PlatformTransactionManager provided by Spring Framework**
    :header-rows: 1
    :widths: 10 35 55

    * - Sr. No.
      - Class name
      - Description
    * - 1.
      - | org.springframework.jdbc.datasource.
        | DataSourceTransactionManager
      - | Implementation class for managing the transaction by calling API of JDBC(\ ``java.sql.Connection``\ ).
        | Use this class when MyBatis and \ ``JdbcTemplate``\  is to be used.
    * - 2.
      - | org.springframework.orm.jpa.
        | JpaTransactionManager
      - | Implementation class for managing the transaction by calling API of JPA(\ ``javax.persistence.EntityTransaction``\ ).
        | Use this class when JPA is to be used.
    * - 3.
      - | org.springframework.transaction.jta.
        | JtaTransactionManager
      - | Implementation class for managing the transaction by calling API of JTA(\ ``javax.transaction.UserTransaction``\ ).
        | Use this class to manage transaction with resources (Database/Messaging service/General-purpose EIS(Enterprise Information System) etc.) using JTS (Java Transaction Service) provided by application server.
        | When it is necessary to execute the operations with multiple resources in a single transaction, it is necessary to  use JTA for managing transactions.

Settings for enabling @Transactional
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| In this guideline, it is recommended to manage transaction by using "Declarative transaction management" where \ ``@Transactional``\  annotation is used.
| Here, the settings required for using \ ``@Transactional``\  annotation are explained.

- :file:`xxx-domain.xml`

 .. code-block:: xml

     <tx:annotation-driven /> <!-- (1) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description

    * - | (1)
      - With use of <tx:annotation-driven> element in XML (bean definition file), transaction control gets enabled at the locations where \ ``@Transactional``\  annotation is used.

Regarding attributes of <tx:annotation-driven> element
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Various attributes can be specified in <tx:annotation-driven> and default behavior can be customized.

- :file:`xxx-domain.xml`

 .. code-block:: xml

     <tx:annotation-driven
          transaction-manager="txManager"
          mode="aspectj"
          proxy-target-class="true"
          order="0" />

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 15 75

    * - Sr. No.
      - Attribute
      - Description

    * - 1
      - transaction-manager
      - Specify \ ``PlatformTransactionManager``\  bean. When omitted, bean registered with the name "transactionManager" is used.

    * - 2
      - mode
      - Specify AOP mode. When omitted, \ ``"proxy"``\  is the default value. \ ``"aspectj"``\  can be also be specified. It is recommended to use \ ``"proxy"``\ .

    * - 3
      - proxy-target-class
      - Flag to specify whether proxy target is limited to class (this is valid only in case of mode="proxy"). When omitted, it will be "false".

        * In case of false, if the target class has an interface, proxy is done using dynamic proxies functionality of standard JDK.
          When there is no interface, proxy is done using GCLIB functionality.
        * In case of true, proxy is done using GCLIB function irrespective of whether interface is available or not.

    * - 4
      - order
      - Order of Advice of AOP (Priority). When omitted, it will be "Last (Lowest priority)".

|

Appendix
--------------------------------------------------------------------------------

.. _DomainLayerAppendixTransactionManagement:

Regarding drawbacks of transaction management
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| There is a description regarding "Understanding drawbacks of transaction" in IBM DeveloperWorks.
| Read this article about pitfalls of transaction management and pitfalls while using @Transactional of Spring Framework.
  Refer to \ `Article on IBM DeveloperWorks <http://www.ibm.com/developerworks/java/library/j-ts1/index.html>`_\ for details.

.. note::

    Since the article of IBM DeveloperWorks is an old article (year 2009), some of the content is different than the behavior when using Spring Framework 4.1.

    Specifically, the contents of "Listing 7. Using read-only with REQUIRED propagation mode - JPA".

    From Spring Framework 4.1, when Hibernate ORM 4.2 or higher version is used as JPA provider,
    it has been improved so that instruction can be given to run the SQL under "Read-only transactions" for JDBC driver (\ `SPR-8959 <https://jira.spring.io/browse/SPR-8959>`_\ ).

    The way of handling read-only transactions depends on the implementation of JDBC driver; hence, confirm the specifications of JDBC driver to be used.


Programmatic transaction management
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In this guideline, "Declarative transaction management" is recommended. However, programmatic transaction management is also possible.
Refer to \ `Spring Reference Document -Transaction Management(Programmatic transaction management)- <http://docs.spring.io/spring/docs/4.1.9.RELEASE/spring-framework-reference/html/transaction.html#transaction-programmatic>`_\  for details.

.. _domainlayer_appendix_blogic:


Sample of implementation of interface and base classes to limit signature
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- Interface to limit signature

 .. code-block:: java

    // (1)
    public interface BLogic<I, O> {
      O execute(I input);
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Interface to limit signature of implementation method of business logic.
        | In the above example, it is defined as generic type of input (I) and output (O) information having one method (execute) for executing business logic.
        | In this guideline, the above interface is called BLogic interface.

- Controller

 .. code-block:: java

    // (2)
    @Inject
    XxxBLogic<XxxInput, XxxOutput> xxxBLogic;

    public String reserve(XxxForm form, RedirectAttributes redirectAttributes) {

        XxxInput input = new XxxInput();
        // omitted

        // (3)
        XxxOutput output = xxxBlogic.execute(input);

        // omitted

        redirectAttributes.addFlashAttribute(output.getTourReservation());
        return "redirect:/xxx?complete";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (2)
      - | Controller injects calling BLogic interface.
    * - | (3)
      - | Controller calls execute method of BLogic interface and executes business logic.

To standardize process flow of business logic when a fixed common process is included in Service, base classes are created to limit signature of method.

- Base classes to limit signature

 .. code-block:: java


    public abstract class AbstractBLogic<I, O> implements BLogic<I, O> {

        public O execute(I input){
          try{

              // omitted

              // (4)
              preExecute(input);

              // (5)
              O output = doExecute(input);

              // omitted

              return output;
          } finally {
              // omitted
          }

        }

        protected abstract void preExecute(I input);

        protected abstract O doExecute(I input);

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (4)
      - | Call the method to perform pre-processing before executing business logic from base classes.
        | In the preExecute method, business rules are checked.
    * - | (5)
      - | Call the method executing business logic from the base classes.


Sample of extending base classes to limit signature is shown below.


- BLogic class (Service)

 .. code-block:: java

    public class XxxBLogic extends AbstractBLogic<XxxInput, XxxOutput> {

        // (6)
        protected void preExecute(XxxInput input) {

            // omitted
            Tour tour = tourRepository.findOne(input.getTourId());
            Date reservationLimitDate = tour.reservationLimitDate();
            if(input.getReservationDate().after(reservationLimitDate)){
                throw new BusinessException(ResultMessages.error().add("e.xx.xx.0001"));
            }

        }

        // (7)
        protected XxxOutput doExecute(XxxInput input) {
            TourReservation tourReservation = new TourReservation();

            // omitted

            tourReservationRepository.save(tourReservation);
            XxxOutput output = new XxxOutput();
            output.setTourReservation(tourReservation);

            // omitted
            return output;
        }

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (6)
      - | Implement pre-process before executing business logic.
        | Business rules are checked.
    * - | (7)
      - | Implement business logic.
        | Logic is implemented to satisfy business rules.

|

Tips
--------------------------------------------------------------------------------

.. _tips_business_error-label:

Method of dealing with violation of business rules as field error
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| When it is necessary to output the error of business rules for each field, the mechanism of (Bean Validation or Spring Validator) on the Controller side should be used.
| In this case, It is recommended to implement check logic as Service class and then to call the method of Service class from Bean Validation or Spring Validator.
| Refer to \ :doc:`../ArchitectureInDetail/Validation`\  business logic approach for details.


.. raw:: latex

   \newpage

