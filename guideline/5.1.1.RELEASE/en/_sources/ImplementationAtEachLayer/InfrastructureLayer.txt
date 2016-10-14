Implementation of Infrastructure Layer
================================================================================

.. only:: html

 .. contents:: Index
    :depth: 3
    :local:

\ :ref:`repository-class-label`\  is carried out in infrastructure layer.

RepositoryImpl implements the method defined in Repository interface.


.. _repository-class-label:

Implementing RepositoryImpl
--------------------------------------------------------------------------------

Methods to create a Repository for relational database using MyBatis3 and JPA are introduced below.

* :ref:`repository-mybatis3-label`
* :ref:`repository-jpa-label`


.. _repository-mybatis3-label:

Implementing Repository using MyBatis3
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When MyBatis3 is to be used as persistence API with relational database,
RepositoryImpl need not be implemented,
if Repository interface is created using ":ref:`DataAccessMyBatis3AppendixAboutMapperMechanism`" provided by MyBatis3.

This is because it is a mechanism where MyBatis3 automatically maps the method of Mapper interface and the statement (SQL) to be called.

When using MyBatis3, an application developer creates:

* Repository interface (method definition)
* Mapping file (SQL and O/R mapping definition)


| An example of creating Repository interface and mapping file is given below.
| For details on how to use MyBatis3, refer to: \ :doc:`../ArchitectureInDetail/DataAccessMyBatis3`\ .

- An example of creating Repository interface (Mapper interface)

 .. code-block:: java

    package com.example.domain.repository.todo;

    import com.example.domain.model.Todo;

    // (1)
    public interface TodoRepository {
        // (2)
        Todo findOne(String todoId);
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Create as an interface of POJO.

        It is not necessary to specify MyBatis3 interface, annotation, etc.
    * - | (2)
      - Define a method of Repository.

        Basically it is not necessary to assign MyBatis3 annotation;
        however, annotation may also be specified in some cases.


- An example of creating mapping file

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org/DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <!-- (3) -->
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <!-- (4) -->
        <select id="findOne" parameterType="string" resultMap="todoResultMap">
          SELECT
              todo_id,
              title,
              finished
          FROM
              t_todo
          WHERE
              todo_id = #{todoId}
        </select>

        <!-- (5) -->
        <resultMap id="todoResultMap" type="Todo">
            <result column="todo_id" property="todoId" />
            <result column="title" property="title" />
            <result column="finished" property="finished" />
        </resultMap>

    </mapper>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (3)
      - Create a mapping file for each Repository interface.

        Specify FQCN (Fully Qualified Class Name) of Repository interface 
        in namespace of mapping file (\ ``namespace``\  attribute of \ ``mapper``\  element).
    * - | (4)
      - Define statement (SQL) to be run for each method defined in Repository interface.

        Specify a method name of Repository interface 
        in statement ID of each statement element (\ ``id``\  attribute of \ ``select``\ /\ ``insert``\ /\ ``update``\ /\ ``delete``\  element). 
    * - | (5)
      - When a query is to be raised, define O/R mapping as required.

        Auto mapping can be used for simple O/R mapping; however,
        individual mapping definition is needed for complex O/R mapping.

        In the above example, auto mapping can also be used for mapping definition as it is simple O/R mapping.


.. _repository-jpa-label:

Implementing Repository using JPA
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| When JPA is to be used as persistence API with relational database, Repository can be very easily created if  \ ``org.springframework.data.jpa.repository.JpaRepository``\  of Spring Data JPA is used.
| For details on how to use Spring Data JPA, refer to \ :doc:`../ArchitectureInDetail/DataAccessJpa`\ .

| When Spring Data JPA is used, only an interface with inherited JpaRepository is required to be created for basic CRUD operations. In other words, RepositoryImpl is not required.
| However, RepositoryImpl is needed for using dynamic query (JPQL).
| Refer to \ :doc:`../ArchitectureInDetail/DataAccessJpa`\  for implementing RepositoryImpl when using Spring Data JPA.

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

    * - Sr. No.
      - Description
    * - | (1)
      - Only by defining the interface that inherits JpaRepository, basic CRUD operations for Todo entity can be performed without being implemented.

| Describe the case to add operations which are not provided by JpaRepository.
| When Spring Data JPA is used, if it is a static query, it is advisable to add a method to the interface and to specify the query (JPQL) to be executed when that method is called, using the annotation.

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

    * - Sr. no.
      - Description
    * - | (1)
      - Specify a query (JPQL) using \ ``@Query``\  annotation.

.. _repository-rest-label:

Implementing Repository to link with external system using RestTemplate 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo::

    **TBD**

    Details will be provided in the next version.


.. raw:: latex

   \newpage

