インフラストラクチャ層の実装
================================================================================

.. contents:: 目次
   :depth: 3
   :local:

インフラストラクチャ層では、\ :ref:`repository-class-label`\ を行う。

RepositoryImplは、Repositoryインタフェースで定義したメソッドの実装を行う。


.. _repository-class-label:

RepositoryImplの実装
--------------------------------------------------------------------------------

以下に、JPAとMyBatis2系を使って、リレーショナルデータベース用のRepositoryを作成する方法を紹介する。

* :ref:`repository-jpa-label`
* :ref:`repository-mybatis2-label`

.. _repository-jpa-label:

JPAを使ってRepositoryを実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| リレーショナルデータベースとの永続APIとして、JPAを使う場合、Spring Data JPAの\ ``org.springframework.data.jpa.repository.JpaRepository``\ を使用すると、非常に簡単にRepositoryを作成することが出来る。
| Spring Data JPAの使用方法の詳細は、\ :doc:`../ArchitectureInDetail/DataAccessJpa`\ を参照されたい。

| Spring Data JPAを使った場合、基本的なCRUD操作は、JpaRepositoryを継承したインタフェースを作成するだけでよい。つまり、基本的には、RepositoryImplは不要である。
| ただし、動的なクエリ(JPQL)を発行する必要がある場合は、RepositoryImplが必要となる。
| Spring Data JPA使用時のRepositoryImplの実装については、\ :doc:`../ArchitectureInDetail/DataAccessJpa`\ を参照されたい。

- TodoRepository.java

 .. code-block:: java
   :emphasize-lines: 1

      public interface TodoRepository extends JpaRepository<Todo, String> { // (1)
          // ...
      }

 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - JpaRepositoryを継承したインタフェースを定義するだけで、Todoエンティティに対する基本的なCRUD操作を実装なしで実現できる。

| JpaRepositoryから提供されていない操作を追加する場合について説明する。
| Spring Data JPAを使った場合、静的なクエリであればインタフェースにメソッドを追加し、そのメソッドが呼び出された時に実行するクエリ（JPQL）をアノテーションで指定すればよい。

- TodoRepository.java

 .. code-block:: java
    :emphasize-lines: 2

     public interface TodoRepository extends JpaRepository<Todo, String> {
         @Query("SELECT COUNT(t) FROM Todo t WHERE finished = :finished") // (1)
         long countByFinished(@Param("finished") boolean finished);
         // ...
     }


 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - \ ``@Query``\ アノテーションで、クエリ（JPQL）を指定する。

.. _repository-mybatis2-label:

MyBatis2系を使ってRepositoryを実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| リレーショナルデータベースとの永続APIとしてMyBatis2系を使う場合、RepositoryImplは、以下のようになる。
| MyBatis2の使用方法の詳細は、\ :doc:`../ArchitectureInDetail/DataAccessMybatis2`\ を参照されたい。
| なお、本ガイドラインではMyBatisを直接使うのではなく、MyBatisのAPIをラップしているTERASOLUNA DAOを使うことを前提としている。

| MyBatisを使う場合、Repositoryインタフェースは、\ **必要なメソッドの定義のみ行えばよい。**\
| もちろん、Spring Dataから提供されているCrudRepositoryや、PagingAndSortingRepositoryを使ってもよいが、すべてのメソッドを使うケースは稀なので、余計な実装が必要になってしまう。

| MyBatisを使う場合、Repositoryインタフェースの定義に加え、RepositoryImplの実装と、SQL定義ファイルの実装が必要となる。
| 下記に、以下2点を目的とした、JpaRepositoryの親インタフェースであるPagingAndSortingRepositoryを実装例を示す。

#. 汎用的なCRUD操作をMyBatisで実装する際のサンプルの提示
#. Spring Data JPAの仕組みを使ってRepositoryを実装した時との実装比較

- TodoRepository.java

 .. code-block:: java
   :emphasize-lines: 1

     public interface TodoRepository extends PagingAndSortingRepository<Todo, String> { // (1)
         long countByFinished(boolean finished);
         // ...
     }

 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明

    * - | (1)
      - Spring Dataより提供されている\ ``org.springframework.data.repository.PagingAndSortingRepository``\ (CrudRepositoryの子インタフェース)を継承することで、 Repositoryインタフェースとして必要な、基本的なメソッドの定義が行われる。MyBatisの場合、インタフェースの定義に加えて、RepositoryImplの実装も必要である。

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

 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明

    * - | (1)
      - クラスアノテーションとして、\ ``@Repository``\ アノテーションを付与する。アノテーションを付与することで、component-scan対象となり、設定ファイルへのbean定義が不要となる。
    * - | (2)
      - クラスアノテーションとして、\ ``@Transactional``\ アノテーションを付与する。トランザクション境界は、Serviceで制御するが、Repositoryにも付与しておくこと。
    * - | (3)
      - 問い合わせ処理を行うための\ ``jp.terasoluna.fw.dao.QueryDAO``\ をインジェクションする。
    * - | (4)
      - 更新処理を行うための\ ``jp.terasoluna.fw.dao.UpdateDAO``\ をインジェクションする。
    * - | (5)
      - 問い合わせ系のメソッドには、\ ``@Transactional(readOnly = true)``\を付与する。
    * - | (6)
      - \ ``CrudRepository``\で定義されているメソッドを実装している。
    * - | (7)
      - \ ``PagingAndSortingRepository``\で定義されているメソッドを実装している。
    * - | (8)
      - \ ``TodoRepository``\で追加したメソッドを実装している。

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

 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明

    * - | (1)
      - namespaceを指定する。Entityを一意に特定できる名前を付与する。
    * - | (2)
      - Entityの型の指定とフィールドとカラムのマッピングを行う。
    * - | (3)
      - SQLID毎にSQLを実装する。

|

.. _repository-rest-label:

RestTemplateを使って外部システムと連携するRepositoryを実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo::

    **TBD**

    次版以降で詳細化する予定である。
