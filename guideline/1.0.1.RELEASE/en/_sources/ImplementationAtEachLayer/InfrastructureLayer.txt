Implementation of Infrastructure Layer
================================================================================

.. only:: html

 .. contents:: Index
    :depth: 3
    :local:

\ :ref:`repository-class-label`\ is carried out in infrastructure layer.

RepositoryImpl implements the Repository interface.


.. _repository-class-label:

Implementation of RepositoryImpl
--------------------------------------------------------------------------------

The method to create Repository for relational database using JPA and MyBatis2 is introduced below.

* :ref:`repository-jpa-label`
* :ref:`repository-mybatis2-label`

|

.. _repository-jpa-label:

Implementing Repository using JPA
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| When JPA is to be used as persistence API with relational database, Repository can be very easily created if  ``org.springframework.data.jpa.repository.JpaRepository`` of Spring Data JPA is used.
| Refer to \ :doc:`../ArchitectureInDetail/DataAccessJpa`\ for details regarding usage of Spring Data JPA.

| When Spring Data JPA is used, only an interface, extending JpaRepository, is required to be created for basic CRUD operations. That means, RepositoryImpl is not required.
| However, RepositoryImpl is needed for using dynamic query (JPQL).
| Refer to \ :doc:`../ArchitectureInDetail/DataAccessJpa`\ for implementing RepositoryImpl while using Spring Data JPA

- TodoRepository.java

 .. code-block:: java
   :emphasize-lines: 1

      public interface TodoRepository extends JpaRepository<Todo, String> { // (1)
          // ...
      }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - S.No.
      - Description
    * - | (1)
      - Only by defining the interface that extends JpaRepository, basic CRUD operations for Todo entity can be performed without implementing the interface.


| Procedure to add an operation not provided by JpaRepository is explained.
| When Spring Data JPA is used, if it is a static query, the method must be added to the interface and the query (JPQL) to be executed must be specified in the annotation to the method in the interface.

- TodoRepository.java

 .. code-block:: java
    :emphasize-lines: 2

     public interface TodoRepository extends JpaRepository<Todo, String> {
         @Query("SELECT COUNT(t) FROM Todo t WHERE finished = :finished") // (1)
         long countByFinished(@Param("finished") boolean finished);
         // ...
     }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - S.No.
      - Description
    * - | (1)
      - Specify Query (JPQL) using @Query annotation.


|

.. _repository-mybatis2-label:

Implementing Repository using MyBatis2
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| When MyBatis is used as persistence API, RepositoryImpl must be created as follows 
| Refer to \ :doc:`../ArchitectureInDetail/DataAccessMybatis2`\  for details regarding usage of MyBatis2
| Furthermore, in this guideline, it is assumed that TERASOLUNA DAO which is a wrapper for MyBatis API is used instead of using MyBatis directly.

| When MyBatis is to be used, **only the required method must be defined** in the Repository interface.
| CrudRepository and PagingAndSortingRepository provided by Spring Data can also be used. However, using all the methods is very rare. Hence, 
| implementation of unnnecessary methods have to be done if these interfaces are used.

| When MyBatis is to be used, RepositoryImpl and SQL definition file should be created in addition to defining Repository interface.
| Example of implementation of PagingAndSortingRepository which is super interface of JpaRepository, is explained below.

#. Sample while implementing general purpose CRUD operation in MyBatis is shown.
#. There is a comparision to the case when Repository is implemented using mechanism of Spring Data JPA.

- TodoRepository.java

 .. code-block:: java
   :emphasize-lines: 1

     public interface TodoRepository extends PagingAndSortingRepository<Todo, String> { // (1)
         long countByFinished(boolean finished);
         // ...
     }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - S.No.
      - Description

    * - | (1)
      - The basic methods required for Repository interface is defined by inheriting ``org.springframework.data.repository.PagingAndSortingRepository`` (Sub interface of CrudRepository) 
        provided by Spring Data. In case of MyBatis, in addition to defining the interface, RepositoryImpl should also be implemented.

- TodoRepositoryImpl.java

 .. code-block:: java
    :emphasize-lines: 1,2,5,8,11,12,17,18,25,26,31,32,37,38,43,44,58,59,65,75,83,88,93,99

      @Repository // (1)
      @Transactional // (2)
      public class TodoRepositoryImpl implements TodoRepository {
          @Inject
          protected QueryDAO queryDAO; // (3)

          @Inject
          protected UpdateDAO updateDAO; // (4)

          @Override
          @Transactional(readOnly = true) // (5)
          public Todo findOne(String id) { // (6)
              return queryDAO.executeForObject("todo.findOne", todoId, Todo.class);
          }

          @Override
          @Transactional(readOnly = true) // (5)
          public boolean exists(String id) { // (6)
              Long count = queryDAO.executeForObject("todo.exists", todoId,
                  Long.class);
              return 0 < count.longValue();
          }

          @Override
          @Transactional(readOnly = true) // (5)
          public Iterable<Todo> findAll() { // (6)
              return findAll((Sort) null);
          }

          @Override
          @Transactional(readOnly = true) // (5)
          public Iterable<Todo> findAll(Iterable<String> ids) { // (6)
              return queryDAO.executeForObjectList("todo.findAll", ids);
          }

          @Override
          @Transactional(readOnly = true) // (5)
          public Iterable<Todo> findAll(Sort sort) { // (7)
              return queryDAO.executeForObjectList("todo.findAllSort", sort);
          }

          @Override
          @Transactional(readOnly = true) // (5)
          Page<Todo> findAll(Pageable pageable) { // (7)
              long count = count();
              List<Todo> todos = new ArrayList<Todo>();
              if(0 < count){
                  todos = queryDAO.executeForObjectList("todo.findAllSort",
                      pageable.getSort(),pageable.getOffset(),pageable.getPageSize());
              } else {
                  todos = new ArrayList<Todo>();
              }
              Page page = new PageImpl(todos,pageable,count);
              return page;
          }

          @Override
          @Transactional(readOnly = true) // (5)
          public long count() { // (6)
              Long count = queryDAO.executeForObject("todo.count", null, Long.class);
              return count.longValue();
          }

          @Override
          public <S extends Todo> S save(S todo) { // (6)
              if(exists(todo.getTodoId())){
                  updateDAO.execute("todo.update", todo);
              } else {
                  updateDAO.execute("todo.insert", todo);
              }
              return todo;
          }

          @Override
          public <S extends Todo> Iterable<S> save(Iterable<S> todos) { // (6)
              for(Todo todo : todos){
                  save(todo);
              }
              return todos;
          }

          @Override
          public void delete(String id) { // (6)
              updateDAO.execute("todo.delete", id);
          }

          @Override
          public void delete(Todo todo) { // (6)
              delete(todo.getTodoId());
          }

          @Override
          public void delete(Iterable<? extends Todo> todos) { // (6)
              for(Todo todo : todos){
                  delete(todo);
              }
          }

          public long countByFinished(boolean finished) { // (8)
              Long count = queryDAO.executeForObject("todo.countByFinished", finished, Long.class);
              return count.longValue();
          }

      }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - S.No.
      - Description

    * - | (1)
      - Assign \ ``@Repository``\  as class annotation. By assigning annotation, it becomes target of component-scan and bean definition in the configuration file is not required.
    * - | (2)
      - Assign \ ``@Transactional``\  as class annotation. Transaction boundary is controlled by Service, but this annotation should also be assigned to Repository as well.
    * - | (3)
      - Inject ``jp.terasoluna.fw.dao.QueryDAO`` for executing query processing.
    * - | (4)
      - Inject  ``jp.terasoluna.fw.dao.UpdateDAO`` for executing update processing.
    * - | (5)
      - Assign \ ``@Transactional(readOnly = true)``\  to query method.
    * - | (6)
      - The method defined in \ ``CrudRepository``\  is implemented.
    * - | (7)
      - The method defined in \ ``PagingAndSortingRepository``\  is implemented.
    * - | (8)
      - The method added in \ ``TodoRepository``\  is implemented.

- sqlMap.xml


 .. code-block:: xml
    :emphasize-lines: 5,7,14,15

     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE sqlMap
                 PUBLIC "-//ibatis.apache.org//DTD SQL Map 2.0//EN"
                 "http://ibatis.apache.org/dtd/sql-map-2.dtd">
     <sqlMap namespace="todo"> <!-- (1) -->

         <resultMap id="todo" class="todo.domain.model.Todo"> <!-- (2) -->
             <result property="todoId" column="todo_id" />
             <result property="todoTitle" column="todo_title" />
             <result property="finished" column="finished" />
             <result property="createdAt" column="created_at" />
         </resultMap>

         <!-- (3) -->
         <select id="findOne" parameterClass="java.lang.String" resultMap="todo">
             <!-- ... -->
         </select>

         <select id="exists" parameterClass="java.lang.String" resultClass="java.lang.Long">
             <!-- ... -->
         </select>

         <select id="findAll" resultMap="todo">
             <!-- ... -->
         </select>

         <select id="findAllSort" parameterClass="org.springframework.data.domain.Sort"
                 resultMap="todo">
             <!-- ... -->
         </select>

         <select id="count" resultClass="java.lang.Long">
             <!-- ... -->
         </select>

         <insert id="insert" parameterClass="todo.domain.model.Todo">
             <!-- ... -->
         </insert>

         <update id="update" parameterClass="todo.domain.model.Todo">
             <!-- ... -->
         </update>

         <delete id="delete" parameterClass="todo.domain.model.Todo">
             <!-- ... -->
         </delete>

         <select id="countByFinished" parameterClass="java.lang.Boolean" resultClass="java.lang.Long">
             <!-- ... -->
         </select>

     </sqlMap>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - S.No.
      - Description

    * - | (1)
      - Specify namespace. Assign name that can uniquely identify Entity.
    * - | (2)
      - Specify the type of Entity and execute mapping of field with column.
    * - | (3)
      - Implement SQL for each SQLID.

|

.. _repository-rest-label:

Implementing Repository to link with external system using RestTemplate
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo::

   **TBD**

   Plan to provide details in the coming versions.


.. raw:: latex

   \newpage

