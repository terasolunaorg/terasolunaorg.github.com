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

The method to create Repository for relational database using JPA is introduced below.

* :ref:`repository-jpa-label`

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

.. _repository-rest-label:

Implementing Repository to link with external system using RestTemplate
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo::

   **TBD**

   Plan to provide details in the coming versions.


.. raw:: latex

   \newpage

