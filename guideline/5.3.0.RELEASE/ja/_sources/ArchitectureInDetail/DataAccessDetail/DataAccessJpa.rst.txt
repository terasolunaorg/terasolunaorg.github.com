データベースアクセス（JPA編）
================================================================================

.. only:: html

 .. contents:: 目次
    :local:
    :depth: 3


.. todo::

    **TBD**

    本章では以下の内容について、現在精査中である。

    * | :file:`persistence.xml` の設定項目について。

    * | QueryDSLを使用した動的Queryの実装方法について。

    * | 関連エンティティのfetch方法を指定する設定値をデフォルト値から変更した方がよいケースの具体例について。

    * | 複数のPersistenceUnitを使用する方法について。

    * | Nativeクエリの使用方法について。

Overview
--------------------------------------------------------------------------------

| 本節では、JPAを使ってデータベースにアクセスする方法について、説明する。
| 本ガイドラインでは、JPAプロバイダとしてHibernateを、JPAのラッパーとしてSpring Data JPAを使用することを前提としている。

 .. figure:: images/dataaccess_jpa.png
    :alt: Target of description
    :width: 100%
    :align: center

    **Picture - Target of description**

.. warning:: 

  本節で説明した内容がEclipseLinkなど、Hibernate以外のJPAプロバイダでも動作することは保証しない。

JPAについて
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
JPA(Java Persistence API)は、リレーショナルデータベースで管理されているレコードを、Javaオブジェクトにマッピングする方法と、
マッピングされたJavaオブジェクトに対して行われた操作を、リレーショナルデータベースのレコードに反映するための仕組みをJavaのAPI仕様として定義したものである。

| JPAは、仕様を定義をしているだけで、実装は提供していない。
| JPAの実装は、HibernateのようなO/R Mapperを開発しているベンダーによって、参照実装として提供されている。
| このように、O/R Mapperを開発しているベンダーによって実装された参照実装のことを、JPAプロバイダと呼ぶ。

JPAのO/R Mapping
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
JPAを使用した際の、リレーショナルデータベースで管理されているレコードとJavaオブジェクトは、以下のようなイメージでマッピングされる。

 .. figure:: images/dataaccess_jpa_mapping.png
    :alt: Image of O/R Mapping
    :width: 100%
    :align: center

    **Picture - Image of O/R Mapping**

| JPAでは、「管理状態」と呼ばれる状態のEntityが保持している値を、変更(setterメソッドを呼び出して値を変更)した場合、変更内容が、リレーショナルデータベースに反映される仕組みになっている。
| これは、編集機能をもつテーブルビューアなどのクライアントソフトウェアに、よく似ている仕組みである。
| テーブルビューアなどのクライアントソフトウェアでは、ビューア上の値を変更すると、データベースに反映されるが、JPAでは、Entityと呼ばれるJavaオブジェクト(JavaBean)の値を変更すると、データベースに反映されることになる。

JPAの基本用語
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
以下に、JPAを使う上で、最低限知っていてほしい用語について、簡単に説明する。


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 70
    :class: longtable

    * - 項番
      - 用語
      - 説明
    * - 1.
      - | Entityクラス
      - | リレーショナルデータベースで管理されているレコードを表現するJavaクラス。
        | \ ``@javax.persistence.Entity``\ アノテーションを付与されたクラスが、Entityクラスとなる。
    * - 2.
      - | EntityManager
      - | Entityのライフサイクルを管理するために、必要なAPIを提供するインタフェース。
        | アプリケーションは、\ ``javax.persistence.EntityManager``\ のメソッドを使用して、リレーショナルデータベースで管理されているレコードを、Javaオブジェクトとして操作する。
        | Spring Data JPAを使う場合は、直接、使用することはないが、Spring Data JPAの仕組みでは表現できないような、Queryを発行する必要がある場合は、このインタフェースを経由してEntityを取得することになる。
    * - 3.
      - | TypedQuery
      - | Entityを検索するためのAPIを提供するインタフェース。
        | アプリケーションは、\ ``javax.persistence.TypedQuery``\ のメソッドを使用して、ID以外の条件に一致するEntityを検索する。
        | Spring Data JPAを使う場合は、直接使用することはないが、Spring Data JPAの仕組みでは、表現できないようなQueryを発行する必要がある場合は、このインタフェースを使用してEntityを検索することになる。
        | 条件に一致する永続層(DB)のEntityを直接操作(更新または削除)するためのメソッドもこのインタフェースに用意されている。
    * - 4.
      - | PersistenceContext
      - | Entityを管理するための領域。
        | ``EntityManager`` を経由して取得または作成されたEntityは、この領域に格納されてライフサイクル管理される。この領域で管理されているEntityの事を「管理状態のEntity」と呼ぶ。
        | この領域は、アプリケーションから直接アクセスすることはできない。
        | Entityの状態は「管理状態」以外に、「作成状態」「削除状態」「分離状態」が存在する。
    * - 5.
      - | findメソッド
      - | 管理状態のEntityを取得するためのメソッド。
        | IDに対応するEntityがPersistenceContextに存在しない場合は、リレーショナルデータベースに格納されているレコードを取得(SELECT)し、管理状態のEntityを生成する。
    * - 6.
      - | persistメソッド
      - | アプリケーションで作成した作成状態のEntityを、管理状態のEntityにするためのメソッド。
        | ``EntityManager`` のメソッドとして提供されており、リレーショナルデータベースにレコードのINSERTを実行するための操作がPersistenceContextに蓄積される。
    * - 7.
      - | mergeメソッド
      - | PersistenceContextで管理されていない分離状態のEntityを、管理状態のEntityにするためのメソッド。
        | \ ``EntityManager``\ のメソッドとして提供されており、基本的にはリレーショナルデータベースに格納されているレコードに対して、UPDATEを実行するための操作がPersistenceContextに蓄積される。
        | ただし、リレーショナルデータベースにIDに一致するレコードが存在しない場合は、UPDATEではなくINSERTが実行される。
    * - 8.
      - | removeメソッド
      - | 管理状態のEntityを、削除状態のEntityにするためのメソッド。
        | \ ``EntityManager``\ のメソッドとして提供されており、リレーショナルデータベースに格納されているレコードに対して、DELETEを実行するための操作がPersistenceContextに蓄積される。
    * - 9.
      - | flushメソッド
      - | PersistenceContextで管理されているEntityに対して行われた操作を、リレーショナルデータベースに強制的に反映するためのメソッド。
        | \ ``EntityManager``\ のメソッドとして提供されており、蓄積された未反映の操作をリレーショナルデータベースに対して実行する。
        | 通常、リレーショナルデータベースへの反映は、トランザクションコミット時に行われるが、コミットより前に反映する必要がある場合は、メソッドを使用する。

 .. raw:: latex

    \newpage

Entityのライフサイクル管理
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Entityのライフサイクル管理イメージは、以下の通りである。

 .. figure:: images/dataaccess_jpa_lifecycle.png
    :alt: Life cycle of entity
    :width: 100%
    :align: center

    **Picture - Life cycle of entity**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``EntityManager``\ のpersistメソッドを呼び出すと、引数に渡したEntity(作成状態のEntity)が、管理状態のEntityとして、PersistenceContextに格納される。
    * - | (2)
      - | \ ``EntityManager``\ のfindメソッドを呼び出すと、引数に渡したIDをもつ管理状態の、Entityが返却される。
        | PersistenceContextに存在しない場合は、Queryを発行して、リレーショナルデータベースよりマッピング対象のレコードを取得し、管理状態のEntityとして格納する。
    * - | (3)
      - | \ ``EntityManager``\ のmergeメソッドを呼び出すと、引数に渡したEntity(分離状態のEntity)の状態が、管理状態のEntityにマージされる。
        | PersistenceContextに存在しない場合は、Queryを発行してリレーショナルデータベースよりマッピング対象のレコードを取得し、管理状態のEntityを格納した後に、引数に渡されたEntityの状態がマージされる。
        | このメソッドを呼び出した場合、persistメソッドとは異なり、引数に渡したEntityが管理状態のEntityとして格納されるわけではないという点に注意すること。
    * - | (4)
      - | \ ``EntityManager``\ のremoveメソッドを呼び出すと、引数に渡した管理状態のEntityが削除状態のEntityとなる。
        | このメソッドを呼び出した場合、削除状態となったEntityを取得することはできなくなる。
    * - | (5)
      - | \ ``EntityManager``\ のflushメソッドを呼び出すと、persist、merge、removeメソッドによって、蓄積されたEntityへの操作が、リレーショナルデータベースに反映される。
        | このメソッドを呼び出すことで、Entityに対して行った変更内容が、リレーショナルデータベースのレコードに同期される。
        | ただし、リレーショナルデータベース側のレコードに対してのみ行われた変更は、Entityには同期されない
        |
        | \ ``EntityManager``\ のfindメソッドを使わずにQueryを発行してEntityを検索すると、検索処理を行う前に、\ ``EntityManager``\内部の処理でflushメソッドと同等の処理が実行され、蓄積されていたEntityへの操作がリレーショナルデータベースに反映される仕組みになっている。
        | Spring Data JPAを使用した際の永続操作の反映タイミングについては、 
        |  :ref:`永続操作の反映タイミングについて(その１) <how_to_create_repository_extends_springdata_flush_timing_note1>`
        |  :ref:`永続操作の反映タイミングについて(その2) <how_to_create_repository_extends_springdata_flush_timing_note2>`
        | を参照されたい。

 .. raw:: latex

    \newpage

\

 .. note:: **他のライフサイクル管理用のメソッドについて**

    \ ``EntityManager``\ には、Entityのライフサイクルを管理するためのメソッドとして、detachメソッド、refreshメソッド、clearメソッドなどがあるが、
    Spring Data JPAを使用する場合、デフォルトの機能で、これらのメソッドを呼び出す仕組みが存在しないため、メソッドの役割のみ説明しておく。

    * detachメソッドは、管理状態のEntityを分離状態のEntityにするためのメソッド。
    * refreshメソッドは、管理状態のEntityをリレーショナルデータベースの状態で最新化するためのメソッド。
    * clearメソッドは、PersistenceContextで、管理されているEntityおよび蓄積された操作をメモリ上から破棄するためのメソッド。

    clearメソッドについては、Spring Data JPAより提供されている\ ``@Modifying``\ アノテーションのclearAutomatically属性を、\ ``true``\ に設定することで、呼び出すことができる。
    詳細については、\ :ref:`data-access-jpa_howtouse_querymethod_modifying`\ を参照されたい。

\

 .. note:: **作成状態と分離状態のEntityへの操作について**

    作成状態のEntityや、分離状態のEntityに行った操作は、persistメソッドまたはmergeメソッドを呼び出さないと、リレーショナルデータベースには反映されない。

Spring Data JPAについて
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Spring Data JPAは、JPAを使ってRepositoryを作成するための、ライブラリを提供している。

| Spring Data JPAを使用すると、Queryメソッドと呼ばれるメソッドを、Repositoryインタフェースに定義するだけで、
| 指定した条件に一致するEntityを取得することが出来るため、Entityの操作を行うための実装を減らすことができる。
| ただし、Queryメソッドで定義できるのはアノテーションで表現できる静的なQueryのみなので、
| 動的Queryなどのアノテーションで表現できないQueryについては、カスタムRepositoryクラスの実装が必要となる。

Spring Data JPAを使ってデータベースにアクセスする際の基本フローを以下に示す。

 .. figure:: images/dataaccess_jpa_basic_flow.png
    :alt: Basic flow of Spring Data JPA
    :width: 100%
    :align: center

    **Picture - Basic flow of Spring Data JPA**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | Serviceから、Repositoryインタフェースのメソッドを呼び出す。
        | メソッドの呼び出しパラメータとして、Entityオブジェクト、EntityのIDなどが渡される。上記例ではEntityを渡しているが、プリミティブな値となることもある。
    * - | (2)
      - | Repositoryインタフェースを動的に実装したProxyクラスは、\ ``org.springframework.data.jpa.repository.support.SimpleJpaRepository``\ や、 カスタムRepositoryクラスに処理を委譲する。
        | Serviceから指定されたパラメータが渡される。
    * - | (3)
      - | Repositoryの実装クラスは、JPAのAPIを呼び出す。
        | Serviceから指定されたパラメータや、Repositoryの実装クラスで生成したパラメータなどが渡される。
    * - | (4)
      - | HibernateのJPA参照実装は、 HibernateのコアAPIの処理を呼び出す。
        | Repositoryの実装クラスから指定されたパラメータやHibernateのJPA参照実装のクラスで生成したパラメータなどが渡される。
    * - | (5)
      - | HibernateのコアAPIは、指定されたパラメータからSQLとバインド値を生成しJDBCドライバに渡す。
        | (実際の値のバインドは、java.sql.PreparedStatement のAPIが使われている)
    * - | (6)
      - | JDBCドライバは、渡されたSQLとバインド値をデータベースに送信することで、SQLを実行する。

 .. raw:: latex

    \newpage

| Spring Data JPAを使用してRepositoryを作成する場合、JPAのAPIを直接呼び出す必要はないが、Spring Data JPAのRepositoryインタフェースのメソッドが、
| JPAのどのメソッドを呼び出しているのかは、意識しておいた方がよい。
| 以下に、Spring Data JPAの、Repositoryインタフェースの代表的なメソッドが、JPAのどのメソッドを呼び出しているのかを示す。

 .. figure:: images/dataaccess_jpa_api-mapping.png
    :alt: API Mapping of Spring Data JPA and JPA
    :width: 90%
    :align: center

    **Picture - API Mapping of Spring Data JPA and JPA**

|

How to use
--------------------------------------------------------------------------------

pom.xmlの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
インフラストラクチャ層にJPA(Spring Data JPA)を使用する場合、以下のdependencyを、pom.xmlに追加する。

 .. code-block:: xml

    <!-- (1) -->
    <dependency>
        <groupId>org.terasoluna.gfw</groupId>
        <artifactId>terasoluna-gfw-jpa-dependencies</artifactId>
        <type>pom</type>
    </dependency>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | JPAに関連するライブラリ群が定義してある\ ``terasoluna-gfw-jpa-dependencies``\ を、dependencyに追加する。

 .. note::

    上記設定例は、依存ライブラリのバージョンを親プロジェクトである terasoluna-gfw-parent で管理する前提であるため、pom.xmlでのバージョンの指定は不要である。

アプリケーションの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

データソースの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| データベースの接続情報をデータソースに設定する。
| データソースの設定ついては、共通編の\ :ref:`data-access-common_howtouse_datasource`\ を参照されたい。

EntityManagerの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\ ``EntityManager``\ を使用するための設定を行う。

- xxx-infra.xml

 .. code-block:: xml

     <!-- (1) -->
     <bean id="jpaVendorAdapter"
         class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
         <!-- (2) -->
         <property name="showSql" value="false" />
         <!-- (3) -->
         <property name="database" value="POSTGRESQL" />
     </bean>

     <!-- (4) -->
     <bean id="entityManagerFactory"
         class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
         <!-- (5) -->
         <property name="packagesToScan" value="xxxxxx.yyyyyy.zzzzzz.domain.model" />
         <!-- (6) -->
         <property name="dataSource" ref="dataSource" />
         <!-- (7) -->
         <property name="jpaVendorAdapter" ref="jpaVendorAdapter" />
         <!-- (8) -->
         <property name="jpaPropertyMap">
             <util:map>
                 <entry key="hibernate.hbm2ddl.auto" value="" />
                 <entry key="hibernate.ejb.naming_strategy"
                     value="org.hibernate.cfg.ImprovedNamingStrategy" />
                 <entry key="hibernate.connection.charSet" value="UTF-8" />
                 <entry key="hibernate.show_sql" value="false" />
                 <entry key="hibernate.format_sql" value="false" />
                 <entry key="hibernate.use_sql_comments" value="true" />
                 <entry key="hibernate.jdbc.batch_size" value="30" />
                 <entry key="hibernate.jdbc.fetch_size" value="100" />
             </util:map>
         </property>
     </bean>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | JPAプロバイダが提供する実装クラスとのアダプタクラスを指定する。
        | JPAプロバイダとしてHibernateを使用するので、\ ``org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter``\ を指定する。
    * - | (2)
      - SQLの出力有無を指定する。設定例では、「false:出力しない 」を指定している。
    * - | (3)
      - | 使用するRDBMSに対応する値を設定する。\ ``org.springframework.orm.jpa.vendor.Database``\ 列挙型に定義されている値を、指定することができる。
        | 設定例では「PostgreSQL」を指定している。
        | **【プロジェクトで使用するデータベースに対応する値に変更が必要】**
        | 環境によって使用するデータベースがかわる場合は、プロパティファイルに値を定義すること。
    * - | (4)
      - | ``javax.persistence.EntityManagerFactory`` のインスタンスを作成するFactoryBeanのクラスを指定する。
        | ``org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean`` を指定する。
    * - | (5)
      - | Entityクラスが格納されているパッケージを指定する。
        | 指定したパッケージに格納されているEntityクラスが、\ ``javax.persistence.EntityManager``\ で管理することができるEntityクラスとなる。
        | **【プロジェクトのパッケージに変更が必要】**
    * - | (6)
      - | 永続層(DB)にアクセスする際に使用するデータソースを指定する。
        | 設定済みのデータソースのbeanを指定する。
    * - | (7)
      - | ``JpaVendorAdapter`` のbeanを指定する。
        | (1)で設定済みのbeanを指定する。
    * - | (8)
      - | Hibernateから提供されている ``EntityManager`` の動作設定を指定する。
        | 詳細については「`Hibernate Reference Documentation <http://docs.jboss.org/hibernate/orm/4.3/manual/en-US/html/ch03.html#configuration-optional>`_\」を参照されたい。

 .. raw:: latex

    \newpage

\

 .. tip::

    データベースにOracleを使う際に、テーブル結合を行うSQLにANSI標準のJOINを使用したい場合は、(8)の\ ``jpaPropertyMap``\ に、以下の設定を指定することで実現できる。

     .. code-block:: xml

         <bean id="entityManagerFactory"
             class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
             <!-- omitted -->
             <property name="jpaPropertyMap">
                 <util:map>
                     <!-- omitted -->
                     <entry key="hibernate.dialect"
                            value="org.hibernate.dialect.Oracle12cDialect" />  <!-- (9) -->
                 </util:map>
             </property>
         </bean>

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90
        :class: longtable

        * - 項番
          - 説明
        * - | (9)
          - | \ ``"hibernate.dialect"``\ に\ ``org.hibernate.dialect.Oracle12cDialect``\ を指定する。
            | \ ``Oracle12cDialect``\ を指定することで、テーブル結合を行うSQLにANSI標準のJOIN句が使用される。

| アプリケーションサーバから提供されているトランザクションマネージャ(JTA)を使用する場合は、以下の設定を行う。
| JTAを使用しない場合との差分について、説明する。
| 特に説明がない箇所については、JTAを使用しない場合と同じ設定でよい。

- xxx-infra.xml

 .. code-block:: xml

     <bean id="entityManagerFactory"
         class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">

         <!-- omitted -->

         <!-- (10) -->
         <property name="jtaDataSource" ref="dataSource" />

         <!-- omitted -->

         <property name="jpaPropertyMap">
             <util:map>

                 <!-- omitted -->

                 <!-- (11)  -->
                 <entry key="hibernate.transaction.jta.platform"
                     value="org.hibernate.service.jta.platform.internal.WeblogicJtaPlatform" />

             </util:map>
         </property>
     </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (10)
      - | 永続層(DB)にアクセスする際に使用するデータソースを指定する。
        | JTAを使用する場合は、\ ``"dataSource"``\ プロパティではなく、\ ``"jtaDataSource"``\ プロパティに、アプリケーションサーバで定義したDataSourceを指定する。
        | アプリケーションサーバで定義したDataSourceの取得方法については、共通編の\ :ref:`data-access-common_howtouse_datasource`\ を参照されたい。
    * - | (11)
      - | ``"jpaPropertyMap"`` プロパティに、JTAのプラットフォームの指定を追加する。
        | 上記は、WeblogicのJTAを使用する場合の設定例となる。
        | 設定可能な値(プラットフォーム)は、 ``org.hibernate.service.jta.platform.spi.JtaPlatform`` の実装クラスのFQCNとなる。
        | 主なアプリケーションサーバ向けの実装クラスについては、Hibernateから提供されている。

\

 .. note::

    環境によって使用するトランザクションマネージャを切り替える必要がある場合は、 ``"entityManagerFactory"`` のbean定義は :file:`xxx-infra.xml` ではなく、 :file:`xxx-env.xml` に行うことを推奨する。

    トランザクションマネージャを環境によって切り替える必要がある具体例としては、ローカルの開発環境ではTomcatなどJTAの機能を持たないアプリケーションサーバを使用し、
    本番および各試験環境では、WeblogicなどのJTAの機能をもつアプリケーションサーバを使用するといったケースがあげられる。

PlatformTransactionManagerの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ローカルトランザクションを使用する場合は、以下の設定を行う。

- xxx-env.xml

 .. code-block:: xml

     <bean id="transactionManager"
         class="org.springframework.orm.jpa.JpaTransactionManager"> <!-- (1) -->
         <property name="entityManagerFactory" ref="entityManagerFactory" /> <!-- (2) -->
     </bean>

 .. tabularcolumns:: |p{0.06\linewidth}|p{0.94\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 6 94

    * - 項番
      - 説明
    * - | (1)
      - ``org.springframework.orm.jpa.JpaTransactionManager`` を指定する。 このクラスは、JPAのAPIを呼び出してトランザクション制御を行う。
    * - | (2)
      - | トランザクション内で使用する\ ``EntityManager``\ の、Factoryを指定する。
        | 設定済みのEntityManagerFactoryのbeanを指定する。

アプリケーションサーバから提供されているトランザクションマネージャ(JTA)を使用する場合は、以下の設定を行う。

- xxx-env.xml

 .. code-block:: xml

     <tx:jta-transaction-manager /> <!-- (1) -->

 .. tabularcolumns:: |p{0.06\linewidth}|p{0.94\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 6 94

    * - 項番
      - 説明
    * - | (1)
      - アプリケーションがデプロイされているアプリケーションサーバに最適な\ ``org.springframework.transaction.jta.JtaTransactionManager``\ が、"transactionManager"というidで、bean定義される。
        bean定義されたクラスは、JTAのAPIを呼び出して、トランザクション制御を行う。

persistence.xmlの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| ``LocalContainerEntityManagerFactoryBean`` を使用する場合は、 :file:`persistence.xml` に必ず設定しなくてはいけない設定項目はない。

\

 .. todo::

     **TBD**

     現時点では、:file:`persistence.xml` に必ず設定しなくてはいけない設定項目はないが、今後増える可能性はある。

     また、Java EEのアプリケーションサーバー上のEntityManagerFactoryを使用する場合は、
     \ :file:`persistence.xml`\ に、設定が必要となると思われるので、Java EEのアプリケーションサーバー上のEntityManagerFactoryを使用する場合の設定については、今後整備する予定である。

Spring Data JPAを有効化するための設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

- xxx-infra.xml

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:jpa="http://www.springframework.org/schema/data/jpa"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation=".....
        http://www.springframework.org/schema/data/jpa
        http://www.springframework.org/schema/data/jpa/spring-jpa.xsd"> <!-- (1) -->

        <!-- ... -->

    </beans>

 .. code-block:: xml

     <jpa:repositories base-package="xxxxxx.yyyyyy.zzzzzz.domain.repository" /> <!-- (2) -->

 .. tabularcolumns:: |p{0.06\linewidth}|p{0.94\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 6 94

    * - 項番
      - 説明
    * - | (1)
      - | Spring Data JPAのコンフィギュレーション用のスキーマ定義を取り込み、ネームスペースとして(\ ``"jpa"``\ )を付与する。
    * - | (2)
      - | RepositoryインタフェースおよびカスタムRepositoryクラスが格納されているベースパッケージを指定する。
        | ``org.springframework.data.repository.Repository`` を継承しているインタフェースと、\ ``org.springframework.data.repository.RepositoryDefinition``\ アノテーションが付与されているインタフェースが、 Spring Data JPAのRepositoryクラスとして自動的にbean定義される。


- <jpa:repositories>要素の属性について
    | 属性として、entity-manager-factory-ref、transaction-manager-ref、named-queries-location、query-lookup-strategy、factory-class、repository-impl-postfixが存在する。

 .. tabularcolumns:: |p{0.06\linewidth}|p{0.20\linewidth}|p{0.74\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 6 20 74
    :class: longtable

    * - 項番
      - 要素
      - 説明
    * - 1.
      - entity-manager-factory-ref
      - | Repositoryで使用する ``EntityManager`` を生成するためのFactoryを指定する。
        | 通常指定する必要はないが、 ``EntityManager`` のFactoryを複数用意する場合は、使用するbeanを指定する必要がある。
    * - 2.
      - transaction-manager-ref
      - | Repositoryのメソッドが呼び出された際に使用する ``PlatformTransactionManager`` を指定する。
        | デフォルトは  ``"transactionManager"`` というbean名で登録されているbeanが使用される。
        | 使用する ``PlatformTransactionManager`` のbean名が ``"transactionManager"`` でない場合は指定が必要である。
    * - 3.
      - named-queries-location
      - | Named Queryが指定されているSpring Data JPAのプロパティファイルのロケーションを指定する。
        | デフォルトは「classpath:META-INF/jpa-named-queries.properties」が使用される。
    * - 4.
      - query-lookup-strategy
      - | Queryメソッドが呼び出された特に実行するQueryをLookupする方法を指定する。
        | デフォルトは ``"CREATE_IF_NOT_FOUND"`` となっている。詳細は、`Spring Data Commons - Reference Documentationの "Query lookup strategies" <http://docs.spring.io/spring-data/commons/docs/1.12.6.RELEASE/reference/html/#repositories.query-methods.query-lookup-strategies>`_\ を参照されたい。 特に理由がない場合は、デフォルトのままでよい。
    * - 5.
      - factory-class
      - | Repositoryインタフェースのメソッドが呼び出された際の処理を実装するクラスを生成するためのFactoryを指定する。
        | デフォルトでは、 ``org.springframework.data.jpa.repository.support.JpaRepositoryFactory`` が使用される。Spring Data JPAのデフォルト実装を変更する場合や、新しいメソッドを追加する場合に作成したFactoryを指定する。
        | 新しいメソッドを追加する方法については、\ :ref:`custommethod_all-label`\ を参照されたい。
    * - 6.
      - repository-impl-postfix
      - | カスタムRepositoryの実装クラスをであることを表す接尾辞を指定する。
        | デフォルトは ``"Impl"`` となっている。例えば、Repositoryインタフェースの名前が ``OrderRepository`` の場合は、 ``OrderRepositoryImpl`` がカスタムRepositoryの実装クラスとなる。特に理由がない場合は、デフォルトのままでよい。
        | カスタムRepositoryについては、「:ref:`custommethod_individual-label`」を参照されたい。

 .. raw:: latex

    \newpage

JPAのアノテーションを使用するための設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| JPAから提供されているアノテーション（\ ``javax.persistence.PersistenceContext``\ と、\ ``javax.persistence.PersistenceUnit``\ ）を使用して、 ``javax.persistence.EntityManagerFactory`` と ``javax.persistence.EntityManager`` をInjectするためには、 ``org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor`` をbean定義する必要がある。
| ``<jpa:repositories>`` 要素を指定した場合、デフォルトでbeanが定義されるため、bean定義の必要はない。

JPAの例外を DataAccessExceptionに変換するための設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| JPAの例外をSpring Frameworkから提供されている ``DataAccessException`` に変換するためには、``org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor`` をbean定義する必要がある。
| ``<jpa:repositories>`` 要素を指定した場合、デフォルトでbeanが定義されるため、bean定義の必要はない。

OpenEntityManagerInViewInterceptorの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
ControllerやJSP等のアプリケーション層でEntityのLazy Fetchを行うためには、\ ``org.springframework.orm.jpa.support.OpenEntityManagerInViewInterceptor``\ を使用して、
``EntityManager`` の生存期間をアプリケーション層まで伸ばす必要がある。

 .. figure:: images/dataaccess_jpa_entitymanager-lifetime-interceptor.png
    :alt: Lifetime of EntityManager on OpenEntityManagerInViewInterceptor
    :width: 80%
    :align: center

    **Picture - Lifetime of EntityManager on OpenEntityManagerInViewInterceptor**

``OpenEntityManagerInViewInterceptor`` を使用しない場合は、 ``EntityManager`` の生存期間はトランザクションと同じになるため、
アプリケーション層で必要となるデータをServiceクラスの処理としてFetchするか、Lazy Fetchを使わずにEager Fetchを使用する必要がある。

 .. figure:: images/dataaccess_jpa_entitymanager-lifetime-default.png
    :alt: Default lifetime of EntityManager
    :width: 80%
    :align: center

    **Picture - Default Life time of EntityManager**


下記の点から、基本的にはFetch方法はLazy Fetchとして、\ ``OpenEntityManagerInViewInterceptor``\ を使用することを推奨する。

* Serviceクラスの処理としてFetchした場合、getterメソッドを呼び出すだけの処理やgetterメソッドへアクセスしたコレクションへのアクセスなど、一見意味のない処理を実装することになってしまう。
* Eager Fetchにした場合、アプリケーション層で使用しないデータへのFetchも行われる可能性があるため、性能に影響を与える可能性がある。

以下に ``OpenEntityManagerInViewInterceptor`` の設定例を示す。

- spring-mvc.xml

 .. code-block:: xml

    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/**" /> <!-- (1) -->
            <mvc:exclude-mapping path="/resources/**" /> <!-- (1) -->
            <mvc:exclude-mapping path="/**/*.html" /> <!-- (1) -->
            <!-- (2) -->
            <bean
                class="org.springframework.orm.jpa.support.OpenEntityManagerInViewInterceptor" />
        </mvc:interceptor>
    </mvc:interceptors>

 .. tabularcolumns:: |p{0.06\linewidth}|p{0.94\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 6 94

    * - 項番
      - 説明
    * - | (1)
      - | Interceptorを適用するパスと、除外パスを指定する。
        | 例では、リソースファイル(js、css、imageなど)のパスと、静的Webページ(HTML)のパス以外のリクエストに対してInterceptorを適用している。
    * - | (2)
      - | ``org.springframework.orm.jpa.support.OpenEntityManagerInViewInterceptor`` を指定する。


 .. note:: **静的リソースへのパスを適用対象外とする**

    静的リソース(js、css、image、htmlなど)のパスについては、データアクセスが発生しないため、Interceptorの適用外とすることを推奨する。
    適用対象にしてしまうと、 ``EntityManager`` に対して無駄な処理（インスタンス生成とクローズ処理）が実行されることになる。

|

| Servlet FilterでLazy Fetchが必要な場合は、 ``org.springframework.orm.jpa.support.OpenEntityManagerInViewFilter`` を使用して、EntityManagerの生存期間をServlet Filter層まで伸ばす必要がある。
| 例えば、 SpringSecurityの ``org.springframework.security.core.userdetails.UserDetailsService`` を拡張実装し、拡張した処理の中でEntityオブジェクトにアクセスする場合、このケースにあてはまる。
| ただし、Lazy Fetchの必要がないのであれば、 ``EntityManager`` の生存期間をServlet Filter層まで伸ばす必要はない。

 .. note:: **Servlet Filter層でのLazy Fetchについて**

    **Servlet Filter層でLazy Fetchが発生しないように設計および実装することを推奨する。**
    ``OpenEntityManagerInViewInterceptor`` を使用した方が、 適用パターンと除外パターンを柔軟に指定できるため、 ``EntityManager`` の生存期間をアプリケーション層まで伸ばす対象のパスを指定しやすくなる。
    Servlet Filterで必要となるデータへのアクセスについては、Serviceクラスの処理として事前にFetchしておくか、またはEager Fetchを使用して事前にロードしておくことで、Lazy Fetchが発生しないようにする。

 .. figure:: images/dataaccess_jpa_entitymanager-lifetime-filter.png
    :alt: Lifetime of EntityManager on OpenEntityManagerInViewFilter
    :width: 80%
    :align: center

    **Picture - Lifetime of EntityManager on OpenEntityManagerInViewFilter**

|

以下に ``OpenEntityManagerInViewFilter`` の設定例を示す。

- web.xml

 .. code-block:: xml

     <!-- (1) -->
     <filter>
         <filter-name>Spring OpenEntityManagerInViewFilter</filter-name>
         <filter-class>org.springframework.orm.jpa.support.OpenEntityManagerInViewFilter</filter-class>
     </filter>
     <!-- (2) -->
     <filter-mapping>
         <filter-name>Spring OpenEntityManagerInViewFilter</filter-name>
         <url-pattern>/*</url-pattern>
     </filter-mapping>

 .. tabularcolumns:: |p{0.06\linewidth}|p{0.94\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 6 94

    * - 項番
      - 説明
    * - | (1)
      - | ``org.springframework.orm.jpa.support.OpenEntityManagerInViewFilter`` を指定する。
        | このServlet Filterは、 **Lazy Fetchが発生するServlet Filterより前に定義する必要がある。**
    * - | (2)
      - | フィルタを適用するURLのパターンを指定する。可能な限り必要なパスのみに適用することを推奨するが、設定が煩雑になってしまうのであれば、「/\*」（すべてのリクエスト）としてもよい。

 .. note::

     ``OpenEntityManagerInViewFilter`` を適用するURLのパターンに「/\*」（すべてのリクエスト）を指定した場合は、 ``OpenEntityManagerInViewInterceptor`` の設定は不要となる。

|

Repositoryインタフェースの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Spring DataではEntity毎のRepositoryインタフェースを作成する方法として、以下3つの方法を提供している。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table::
    :widths: 10 35 55
    :header-rows: 1

    * - 項番
      - 作成方法
      - 説明
    * - 1.
      - :ref:`how_to_create_repository_extends_springdata-label`
      - Spring Dataから提供されているインタフェースを継承することで、Entity毎のRepositoryインタフェースを作成する。
        **特に理由がない場合は、この方法で、Entity毎のRepositoryインタフェースを作成することを推奨する。**
    * - 2.
      - :ref:`how_to_create_repository_extends_myinterface-label`
      - Spring Dataから提供されているRepositoryインタフェースのメソッドの中から、必要なメソッドのみ定義したプロジェクト用の共通インタフェースを作成し、作成した共通インタフェースを継承することでEntity毎のRepositoryインタフェースを作成する。
    * - 3.
      - :ref:`how_to_create_repository_notextends-label`
      - Spring Dataから提供されているインタフェースやプロジェクト用の共通インタフェースの継承は行わずに、Entity毎にRepositoryインタフェースを作成する。

|

.. _how_to_create_repository_extends_springdata-label:

Spring Data提供のインタフェースを継承する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Spring Dataから提供されているインタフェースを継承してEntity毎のRepositoryインタフェースを作成する方法について説明する。

継承することができるインタフェースは以下の通り。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table::
    :widths: 10 35 55
    :header-rows: 1

    * - 項番
      - インタフェース
      - 説明
    * - 1.
      - | org.springframework.data.repository
        | CrudRepository
      - 汎用的なCRUD操作を行うメソッドを提供しているRepositoryインタフェース。
    * - 2.
      - | org.springframework.data.repository
        | PagingAndSortingRepository
      - ``CrudRepository`` のfindAllメソッドにページネーション機能とソート機能を追加したRepositoryインタフェース。
    * - 3.
      - org.springframework.data.jpa.repository
        JpaRepository
      - | JPAの仕様に依存するメソッドを提供しているRepositoryインタフェース。
        | ``PagingAndSortingRepository`` を継承しているため、 ``PagingAndSortingRepository`` および ``CrudRepository`` のメソッドも使用する事ができる。
        | **特に理由がない場合は、本インタフェースを継承してEntity毎のRepositoryインタフェースを作成することを推奨する。**

 .. note:: **Spring Data提供のRepositoryインタフェースのデフォルト実装について**

    上記インタフェースで定義されているメソッドの実装は、Spring Data JPAより提供されている
    ``org.springframework.data.jpa.repository.support.SimpleJpaRepository`` で行われている。

|

以下に、作成例を示す。

 .. code-block:: java

    public interface OrderRepository extends JpaRepository<Order, Integer> { // (1)

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | ``JpaRepository`` を継承し、ジェネリック型 ``<T>`` にEntityの型、ジェネリック型 ``<ID extends Serializable>`` にEntityのIDの型を指定する。
        | 上記例では、Entityに\ ``Order``\ 型、EntityのIDに\ ``Integer``\ 型を指定している。

|

``JpaRepository`` を継承してEntity毎のRepositoryインタフェースを作成すると、以下のメソッドに対する実装を得ることが出来る。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table::
    :widths: 10 35 55
    :header-rows: 1
    :class: longtable

    * - 項番
      - メソッド
      - 説明
    * - 1.
      - <S extends T> S save(S entity)
      - | 指定されたEntityに対する永続操作(INSERT/UPDATE)を ``javax.persistence.EntityManger`` に蓄積するためのメソッド。
        | IDプロパティ(``@javax.persistence.Id`` アノテーションまたは ``@javax.persistence.EmbeddedId`` アノテーションが付与されているプロパティ)に値が設定されていない場合は ``EntityManager`` の ``persist`` メソッドが呼ばれ、値が設定されている場合は ``merge`` メソッドが呼び出される。
        | mergeメソッドが呼び出された場合、返却されるEntityオブジェクトは、引数で渡されたEntityとは別のオブジェクトとなるので注意すること。
    * - 2.
      - <S extends T> List<S> save(Iterable<S> entities)
      - | 指定された複数のEntityに対する永続操作を ``EntityManger`` に蓄積するためのメソッド。
        | ``<S extends T> S save(S entity)`` メソッドを繰り返し呼び出す事で実現している。
    * - 3.
      - T saveAndFlush(T entity)
      - | 指定されたEntityに対する永続操作を ``EntityManger`` に蓄積した後に、蓄積されている永続操作(INSERT/UPDATE/DELETE)を永続層(DB)に反映するためのメソッド。
    * - 4.
      - void flush()
      - ``EntityManager`` に蓄積されたEntityへの永続操作(INSERT/UPDATE/DELETE)を永続層(DB)に実行するためのメソッド。
    * - 5.
      - void delete(ID id)
      - | 指定されたIDのEntityに対する削除操作を ``EntityManger`` に蓄積するためのメソッド。
        | このメソッドは ``T findOne(ID)`` メソッドを呼び出してEntityオブジェクトを ``EntityManger`` の管理下にしてから削除している。
        | ``T findOne(ID)`` メソッドの呼び出し時にEntityが存在しない場合は、 ``org.springframework.dao.EmptyResultDataAccessException`` が発生する。
    * - 6.
      - void delete(T entity)
      - | 指定されたEntityに対する削除操作を ``EntityManger`` に蓄積するためのメソッド。
    * - 7.
      - void delete(Iterable<? extends T> entities)
      - | 指定された複数のEntityに対する削除操作を ``EntityManger`` に蓄積するためのメソッド。
        | ``void delete(T entity)``  メソッドを繰り返し呼び出す事で実現している。削除対象のEntityが大量になる場合は、 ``void deleteInBatch(Iterable<T> entities)`` メソッドを使用した方が効率的に削除することができる。
    * - 8.
      - void deleteAll()
      - | すべてのEntityに対する削除操作を ``EntityManger`` に蓄積するためのメソッド。
        | ``List<T> findAll()`` メソッドで取得したEntityに対して ``void delete(T entity)`` メソッドを繰り返し呼び出す事で実現している。
        | 削除対象のEntityが大量になる場合は、 ``void deleteAllInBatch()`` メソッドを使用すること。このメソッドは削除するEntityをすべてアプリケーション上に読み込むので、メモリ枯渇の原因になる。
    * - 9.
      - void deleteInBatch(Iterable<T> entities)
      - | 指定された複数のEntityを直接永続層(DB)から削除するためのメソッド。
        | このメソッドを使って削除した場合、 ``EntityManager`` 上で管理(キャッシュ)されているEntityは削除されないため、 本メソッドで削除した後に ``T findOne(ID id)`` メソッドを呼び出すと ``EntityManager`` 上で管理されているEntityが返却されるので注意が必要。
        | 後続処理で削除したEntityに対して ``T findOne(ID id)`` メソッドなどの ``EntityManager`` 上で管理(キャッシュ)されているEntityオブジェクトを返却するメソッドを呼び出す可能性がある場合は、 ``void delete(Iterable<? extends T> entities)`` メソッドを使用して削除すること。
    * - 10.
      - void deleteAllInBatch()
      - | すべてのEntityを直接永続層(DB)から削除するためのメソッド。
        | ``void deleteInBatch(Iterable<T> entities)`` メソッドと同様、``EntityManager`` 上で管理(キャッシュ)されているEntityは削除されない点に注意すること。
    * - 11.
      - T findOne(ID id)
      - | 指定されたIDのEntityを永続層(DB)から取得するためのメソッド。
        | 永続層から取得されたEntityは ``EntityManager`` によって管理(キャッシュ)されるため、2回目以降のアクセスでは永続層へのアクセスは発生せず、キャッシュされているEntityが返却される。
    * - 12.
      - List<T> findAll()
      - | すべてのEntityを永続層(DB)から取得するためのメソッド。
        | 永続層から取得されたEntityは ``EntityManager`` によって管理(キャッシュ)される。
    * - 13.
      - Iterable<T> findAll(Iterable<ID> ids)
      - | 指定された複数のIDのEntityを永続層(DB)から取得するためのメソッド。
        | 永続層から取得されたEntityは ``EntityManager`` によって管理(キャッシュ)される。 指定されたIDはIN句を使って検索されるため、OracleなどIN句に指定できる値の数に制限があるDBを使う場合は注意すること。
    * - 14.
      - List<T> findAll(Sort sort)
      - | 指定された並び順ですべてのEntityを永続層(DB)から取得するためのメソッド。
        | 永続層から取得されたEntityは ``EntityManager`` によって管理(キャッシュ)される。
    * - 15.
      - Page<T> findAll(Pageable pageable)
      - | 指定されたページ(並び順、ページ数、ページ内に表示する件数)に一致するEntityを永続層(DB)から取得するためのメソッド。
        | 永続層から取得されたEntityは ``EntityManager`` によって管理(キャッシュ)される。
    * - 16.
      - boolean exists(ID id)
      - | 指定されたIDのEntityが存在するかチェックするためのメソッド。
    * - 17.
      - long count()
      - | 永続化対象のEntityの件数を取得するためのメソッド。

 .. raw:: latex

    \newpage

 .. warning:: **JPAの楽観ロック(@javax.persistence.Version)使用時の動作について**

     JPAの楽観ロック( ``@Version`` )使用時に更新対象のEntityが更新または削除された場合は、\ ``org.springframework.dao.OptimisticLockingFailureException``\ が発生する。
     ``OptimisticLockingFailureException`` が発生する可能性があるメソッドは、以下の通りである。

      * <S extends T> S save(S entity)
      * <S extends T> List<S> save(Iterable<S> entities)
      * T saveAndFlush(T entity)
      * void delete(ID id)
      * void delete(T entity)
      * void delete(Iterable<? extends T> entities)
      * void deleteAll()
      * void flush()

     JPAの楽観ロックの詳細については :doc:`ExclusionControl` を参照されたい。

.. _how_to_create_repository_extends_springdata_flush_timing_note1:

 .. note:: **永続操作の反映タイミングについて(その１)**

    \ ``EntityManager``\ に蓄積されたEntityへの永続操作は、トランザクションをコミットする直前に実行され永続層(DB)に反映される。
    そのため、一意制約違反などのエラーをトランザクション管理内の処理(Serviceの処理)でハンドリングしたい場合は、 ``saveAndFlush`` メソッドまたは ``flush`` メソッドを呼び出して ``EntityManager`` 内に蓄積されているEntityへの永続操作を強制的に実行する必要がある。
    単にエラーをクライアントに通知するだけでよければ、Controllerで例外ハンドリングを行い適切なメッセージを設定すればよい。

    \ ``saveAndFlush``\ メソッドおよび\ ``flush``\ メソッドはJPA依存のメソッドなので、意図なく使用しないように注意すること。

 - 通常のフロー

  .. figure:: images/dataaccess_jpa_persistence_flow_normal.png
    :alt: Normal sequence of persistence processing
    :width: 100%
    :align: center

    **Picture - Normal sequence of persistence processing**

 - flush時のフロー

  .. figure:: images/dataaccess_jpa_persistence_flow_flush.png
    :alt: Sequence of persistence processing when using flush method
    :width: 100%
    :align: center

    **Picture - Sequence of persistence processing when using flush method**

.. _how_to_create_repository_extends_springdata_flush_timing_note2:

 .. note:: **永続操作の反映タイミングについて(その２)**

    以下のメソッドを呼び出した場合、\ ``EntityManager``\ と、永続層(DB)で管理しているデータの不整合が発生しないようにするために、
    メインの処理が行われる前に ``EntityManager`` に蓄積されているEntityへの永続操作が永続層(DB)に反映される。

     * ``List<T> findAll`` 系メソッド
     * ``boolean exists(ID id)``
     * ``long count()``

    上記のメソッドは、永続層(DB)に直接Queryを発行するため、 メインの処理が行われる前に永続層(DB)に反映されないとデータの不整合が発生することになる。
    後述するQueryメソッドを呼び出した際も、 ``EntityManager`` に蓄積されていたEntityへの永続操作が、永続層(DB)に反映されるトリガーとなる。

 - Query発行時のフロー

  .. figure:: images/dataaccess_jpa_persistence_flow_query.png
    :alt: Sequence of persistence processing when using query method
    :width: 100%
    :align: center

    **Picture - Sequence of persistence processing when using query method**

.. _how_to_create_repository_extends_myinterface-label:

必要なメソッドのみ定義したインタフェースを継承する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Spring Dataから提供されているインタフェースに定義されているメソッドの中から、必要なメソッドのみ定義した共通インタフェースを
作成し継承する事でEntity毎のRepositoryインタフェースを作成する方法について説明する。

メソッドのシグネチャをSpring Dataから提供されているRepositoryインタフェースのメソッドと一致させる必要があるが、Spring Data提供の
Repositoryインタフェースを継承して作成した際と同様、メソッドの実装は不要である。

 .. note:: **想定される適用ケース**

    Spring Dataから提供されているRepositoryインタフェースのメソッドの中には、実際のアプリケーションでは使用しないまたは使用しない方がよいメソッドもある。
    そのようなメソッドをRepositoryインタフェースから削除したい場合は、この方法で作成する。
    インタフェースに定義したメソッドの実装は、Spring Data JPAより提供されている\ ``org.springframework.data.jpa.repository.support.SimpleJpaRepository``\ で行われている。


以下に、作成例を示す。

 .. code-block:: java

    @NoRepositoryBean // (1)
    public interface MyProjectRepository<T, ID extends Serializable> extends
            Repository<T, ID> { // (2)

        T findOne(ID id); // (3)

        T save(T entity); // (3)

        // ...

    }

 .. code-block:: java

    public interface OrderRepository extends MyProjectRepository<Order, Integer> { // (4)

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | ``@NoRepositoryBean`` を指定し、Spring Dataによる ``Repository`` インターフェースのインスタンス化対象から外す。
    * - | (2)
      - | ``org.springframework.data.repository.Repository`` を継承しプロジェクト用の汎用インタフェースを作成する。
        | 汎用インタフェースなので、ジェネリック型を使用する。
    * - | (3)
      - Spring Dataから提供されているRepositoryインタフェースのメソッドの中から必要なメソッドを選んで定義する。
    * - | (4)
      - プロジェクト用の汎用インタフェースを継承し、ジェネリック型 ``<T>`` にEntityの型、ジェネリック型 ``<ID extends Serializable>`` にEntityのIDの型を指定する。例では、Entityに\ ``Order``\ 型、EntityのIDに\ ``Integer``\ 型を指定している。

.. _how_to_create_repository_notextends-label:

インタフェースの継承は行わない
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Spring Dataより提供されているインタフェースや共通インタフェースを継承しないで、Entity毎のRepositoryインタフェースを作成する方法について、説明する。

| クラスアノテーションとして\ ``@org.springframework.data.repository.RepositoryDefinition``\ アノテーションを指定し、 domainClass属性にEntityの型を、idClass属性にEntityのIDの型を指定する。
| Spring Dataから提供されているRepositoryインタフェースに定義されているメソッドと同じシグネチャのメソッドについては、Spring Data提供のRepositoryインタフェースを継承して作成した際と同様、メソッドの実装は不要である。

\

 .. note:: **想定される適用ケース**

    共通的なEntityの操作が必要ない場合は、この方法で作成してもよい。
    Spring Dataから提供されているRepositoryインタフェースに定義されているメソッドと同じシグネチャのメソッドの実装は、Spring Data JPAより提供されている ``org.springframework.data.jpa.repository.support.SimpleJpaRepository`` で行われている。


以下に、作成例を示す。

 .. code-block:: java

    @RepositoryDefinition(domainClass = Order.class, idClass = Integer.class) // (1)
    public interface OrderRepository { //(2)

        Order findOne(Integer id); // (3)

        Order save(Order entity); // (3)

        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | ``@RepositoryDefinition`` アノテーションを指定する。
        | 例では、domainClass属性(Entityの型)に\ ``Order``\ 型、idClass属性(EntityのIDの型)に\ ``Integer``\ 型を指定している。
    * - | (2)
      - Spring Dataから提供されているインタフェース(\ ``org.springframework.data.repository.Repository``\ )の継承は不要である。
    * - | (3)
      - Entity毎に必要なメソッドを定義する。

.. _data-access-jpa_how_to_use_querymethod:

Queryメソッドの追加
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Spring Dataより提供されている汎用的なCRUD操作を行うためのインタフェースだけでは、実際のアプリケーションを構築する事は難しい。
| そのためSpring Dataでは、Entity毎のRepositoryインタフェースに対して任意の永続操作(SELECT/UPDATE/DELETE)を行うためのQueryメソッドを追加できる仕組みを提供している。
| 追加したQueryメソッドでは、Query言語(JPQLまたはNativeなSQL)を使用してEntityの操作を行う。

 .. note:: **JPQLとは**

   JPQLとは"Java Persistence Query Language"の略で、永続層(DB)のレコードに対応するEntityを操作(SELECT/UPDATE/DELETE)するためのQuery言語である。
   文法はSQLに似ているが、永続層(DB)のレコードを直接操作するのではなく、永続層のレコードにマッピングされているEntityを操作することになる。
   Entityに対して行った操作の永続層(DB)への反映は、JPAプロバイダ(Hibernate)によって行われる。

   JPQLの詳細については、`JSR 338: Java Persistence API, Version 2.1のSpecification(PDF)「Chapter 4 Query Language」 <http://download.oracle.com/otn-pub/jcp/persistence-2_1-fr-eval-spec/JavaPersistence.pdf>`_\ を参照されたい。

Queryメソッドを定義する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Queryメソッドは、Entity毎のRepositoryインタフェースのメソッドとして定義する。

 .. code-block:: java

    public interface OrderRepository extends JpaRepository<Order, Integer> {
        List<Order> findByStatusCode(String statusCode);
    }

実行するQueryを指定する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Queryメソッド呼び出し時に実行するQueryを指定する必要がある。
| 指定方法は以下の通り。詳細は、\ :ref:`how_to_specify_query-label`\ を参照されたい。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :widths: 10 30 60
    :header-rows: 1

    * - 項番
      - Queryの指定方法
      - 説明
    * - 1.
      - | :ref:`@Query アノテーション <how_to_specify_query_annotation-label>`
        | (Spring Dataの機能)
      - | Entity毎のRepositoryインタフェースに追加したメソッドに ``@org.springframework.data.jpa.repository.Query`` アノテーションを指定し、実行するQueryを指定する。
        | **特に理由がない場合は、この方法で指定することを推奨する。**
    * - 2.
      - | :ref:`命名規約ベースのメソッド名 <how_to_specify_query_mathodname-label>`
        | (Spring Dataの機能)
      - | Spring Dataが定めた命名規約に則りメソッド名を付与することで実行するQueryを指定する。
        | Spring Data JPAの機能によってメソッド名から実行するQuery(JPQL)が生成される。生成できるJPQLはSELECTのみとなっている。
        | **条件が少なくシンプルなQueryの場合は、 @Query アノテーションを使わずにこの方法を使ってもよい。** ただし、条件が多く複雑なQueryの場合は、メソッド名は振る舞いを表すシンプルな名前にして ``@Query`` アノテーションでQueryを指定すること。
    * - 3.
      - | :ref:`プロパティファイルのNamed query <how_to_specify_query_namedquery_properties-label>`
        | (Spring Dataの機能)
      - | Spring Data JPAから提供されているプロパティファイルに実行するQueryを指定する。
        | **メソッド定義とQuery指定を行う箇所が分離してしまうので、基本的にはこの方法での指定は推奨しない。**
        | **ただし、QueryとしてNativeなSQLを使用する場合は、データベースに依存するSQLをプロパティファイルに定義する必要があるか確認すること。**
        | 使用するデータベースを任意に選択できるアプリケーションや、実行環境によって用意できるデータベースが変わる(変わる可能性がある)場合には、この方法でQueryを指定し、プロパティファイルを環境依存資材として管理する必要がある。

 .. note:: **Query指定方法の併用について**

    複数のQueryの指定方法を併用することに対して、特に制限は設けない。プロジェクトで使用する指定方法や併用の制限については、プロジェクト毎に判断すること。

 .. note:: **QueryのLookup方法について**

    Spring Dataデフォルトの動作は、 ``CREATE_IF_NOT_FOUND`` に設定されているため、以下の動作となる。

    #. ``@Query`` アノテーションに指定されているQueryを取得し、指定があればそのQueryを使用する。
    #. Named queryから対応するQueryを取得し、対応するQueryが見つかった場合はそのQueryを使用する。
    #. メソッド名からQuery(JPQL)を作成して使用する。
    #. メソッド名からQuery(JPQL)が作成できない場合は、エラーとなる。

    QueryのLookup方法の詳細については、 `Spring Data Commons - Reference Documentation「Defining query methods」の
    「Query lookup strategies」 <http://docs.spring.io/spring-data/commons/docs/1.12.6.RELEASE/reference/html/#repositories.query-methods.query-lookup-strategies>`_\ を参照されたい。

Entityのロックを取得する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Entityのロックを取得する必要がある場合は、Queryメソッドに ``@org.springframework.data.jpa.repository.Lock`` アノテーションを追加し、ロックモードを指定する。
| 詳細については、 :doc:`ExclusionControl` を参照されたい。

 .. code-block:: java

    @Query(value = "SELECT o FROM Order o WHERE o.status.code = :statusCode ORDER BY o.id DESC")
    @Lock(LockModeType.PESSIMISTIC_WRITE) // (1)
    List<Order> findByStatusCode(@Param("statusCode") String statusCode);

 .. code-block:: sql

    -- (2) statusCode='accepted'
    SELECT
            order0_.id AS id1_5_
            ,order0_.status_code AS status2_5_
        FROM
            t_order order0_
        WHERE
            order0_.status_code = 'accepted'
        ORDER BY
            order0_.id DESC
        FOR UPDATE


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | ``@Lock`` アノテーションのvalue属性にロックモードを指定する。
        | 指定可能なロックモードについては、`Java Platform, Enterprise Edition API Specification <http://docs.oracle.com/javaee/7/api/javax/persistence/LockModeType.html>`_\ を参照されたい。
    * - | (2)
      - | JPQLから変換されたNativeなSQL。(使用DBはPostgreSQL)
        | 例では、``LockModeType.PESSIMISTIC_WRITE`` を指定しているので、SQLに"FOR UPDATE"句が追加される。


.. _data-access-jpa_howtouse_querymethod_modifying:

永続層のEntityを直接操作する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Entityの更新および削除の操作は、原則 ``EntityManager`` 上で管理されているEntityオブジェクトに対して行うことを推奨する。
| ただし、Entityを一括で更新または削除する必要がある場合は、Queryメソッドを使って永続層(DB)のEntityを直接操作することを検討すること。

 .. note:: **性能劣化の要因軽減**

    永続層のEntityを直接操作することで、Entityの操作を行うためのSQLの発行回数を減らすことができる。
    そのため、高い性能要件があるアプリケーションの場合は、この方法でEntityの一括操作を行うことで、性能劣化の要因を減らす事ができる。
    減らすことが出来るSQLは以下の通り。

    * Entityオブジェクトを\ ``EntityManager``\ 上に読み込むためのSQL。発行が不要となる。
    * Entityを更新および削除するためのSQL。n回の発行が必要だったものが1回の発行で済む。

 .. note::  **永続層のEntityを直接操作するかの判断基準について**

    永続層のEntityを直接操作する場合、機能的な注意点がいくつかあるため、 **性能要件が高くないアプリケーションの場合は、
    一括操作についても EntityManager 上で管理されているEntityオブジェクトに対して行うことを推奨する。**
    具体的な注意点については、実装例を参照されたい。

以下に、Queryメソッドを使って、永続層のEntityを直接操作する実装例を示す。

 .. code-block:: java

    @Modifying // (1)
    @Query("UPDATE OrderItem oi SET oi.logicalDelete = true WHERE oi.id.orderId = :orderId ") // (2)
    int updateToLogicalDelete(@Param("orderId") Integer orderId); // (3)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | 更新系のQueryメソッドであることを示す ``@org.springframework.data.jpa.repository.Modifying`` アノテーションを指定する。
        | 指定しないと実行時にエラーとなる。
    * - | (2)
      - | 更新系(UPDATEまたはDELETE)用のQueryを指定する。
    * - | (3)
      - | 更新件数や削除件数が必要な場合は、``int`` または ``java.lang.Integer`` を戻り値の型として指定し、件数が必要ない場合は、 ``void`` を指定する。

 .. warning:: **EntityManager上で管理しているEntityとの整合性について**

    Queryメソッドを使って永続層のEntityを直接操作した場合、Spring Data JPAのデフォルト動作ではEntityManager上で管理されているEntityに反映されない。
    そのため、直後に ``JpaRepository#findOne(ID)`` メソッドを呼び出して取得されるEntityオブジェクトは、操作前の状態である点に注意すること。

    この動作を回避する方法として、 ``@Modifying`` アノテーションのclearAutomatically属性を ``true`` に指定する方法がある。
    clearAutomatically属性に ``true`` を指定した場合、永続層のEntityを直接操作した後に、 ``EntityManager`` の ``clear()`` メソッドが呼び出され、 ``EntityManager`` 上で管理されていたEntityオブジェクトと蓄積されていた永続操作が ``EntityManager`` 上から破棄される。
    そのため、直後に ``JpaRepository#findOne(ID)`` メソッドを呼び出した場合、永続層から最新状態のEntityが取得され、永続層と ``EntityManager`` の状態が同期される仕組みになっている。

 .. warning:: **@Modifying(clearAutomatically = true) 使用時の注意点**

    ``@Modifying(clearAutomatically = true)`` とすることで、蓄積されていた永続操作(INSERT/UPDATE/DELETE)も ``EntityManager`` 上から破棄されてしまうという点に注意が必要となる。
    これは、必要な永続操作が永続層に反映されない可能性がある事を意味するため、バグを引き起こす要因となりうる。

    この問題を回避するためには、 永続層のEntityを直接操作する前に ``JpaRepository#saveAndFlush(T entity)`` または ``JpaRepository#flush()`` メソッドを呼び出し、蓄積されている永続操作を永続層に反映しておく必要がある。

Queryヒントを設定する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Queryにヒントを設定する必要がある場合は、Queryメソッドに ``@org.springframework.data.jpa.repository.QueryHints`` アノテーションを追加し、
value属性にQueryヒント( ``@javax.persistence.QueryHint`` )を指定する。

 .. code-block:: java

    @Query(value = "SELECT o FROM Order o WHERE o.status.code = :statusCode ORDER BY o.id DESC")
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @QueryHints(value = { @QueryHint(name = "javax.persistence.lock.timeout", value = "0") }) // (1)
    List<Order> findByStatusCode(@Param("statusCode") String statusCode);

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | ``@QueryHint`` アノテーションのname属性にヒント名、value属性にヒント値を設定する。
        | 指定できるヒントは、JPAの仕様で決められているものに加え、プロバイダ固有のものがある。
        | 上記例では、ロックタイムアウトを ``0`` に設定している(使用DBはOracle)。SQLに"FOR UPDATE NOWAIT"句が追加される。

 .. note:: **Hibernateで指定できるQueryヒントについて**

    JPA仕様で決められているQueryヒントは以下の通り。
    詳細は、`JSR 338: Java Persistence API, Version 2.1のSpecification(PDF) <http://download.oracle.com/otn-pub/jcp/persistence-2_1-fr-eval-spec/JavaPersistence.pdf>`_\ を参照されたい。

    * ``javax.persistence.query.timeout``
    * ``javax.persistence.lock.timeout``
    * ``javax.persistence.cache.retrieveMode``
    * ``javax.persistence.cache.storeMode``

    Hibernate固有のQueryヒントについては、 `Hibernate EntityManager User guide <http://docs.jboss.org/hibernate/entitymanager/3.6/reference/en/html/objectstate.html#d0e1109>`_\ の「3.4.1.8. Query hints」を参照されたい。


.. _how_to_specify_query-label:

QueryメソッドのQuery指定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Queryメソッド呼び出し時に実行するQueryの指定方法について説明する。

* :ref:`how_to_specify_query_annotation-label`
* :ref:`how_to_specify_query_mathodname-label`
* :ref:`how_to_specify_query_namedquery_properties-label`

.. _how_to_specify_query_annotation-label:

@Query アノテーションで指定する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
``@Query`` アノテーションのvalue属性に実行するQuery(JPQL)を指定する。

 .. code-block:: java

    @Query(value = "SELECT o FROM Order o WHERE o.status.code = :statusCode ORDER BY o.id DESC") // (1)
    List<Order> findByStatusCode(@Param("statusCode") String statusCode);

 .. code-block:: sql

    -- (2) statusCode='accepted'
    SELECT
            order0_.id AS id1_5_
            ,order0_.status_code AS status2_5_
        FROM
            t_order order0_
        WHERE
            order0_.status_code = 'accepted'
        ORDER BY
            order0_.id DESC

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | ``@Query`` アノテーションのvalue属性に実行するQuery(JPQL)を指定する。
        | 上記例では、``Order`` オブジェクトで保持している ``status`` プロパティ( ``OrderStatus`` 型)の ``code`` プロパティ( ``String`` 型) の値が指定したパラメータ値( ``statusCode`` )と一致する ``Order`` オブジェクトを ``id`` プロパティの降順に並べて取得するためのQueryを指定している。
    * - | (2)
      - JPQLから変換されたNativeなSQL。``@Query`` アノテーションのvalue属性に指定したQuery(JPQL)は、使用するデータベースのNativeなSQLに変換され実行される。

 .. note:: **JPQLではなくNativeなSQLを直接指定する方法**

    QueryとしてJPQLではなくNativeなSQLを直接指定したい場合は、 nativeQuery属性を ``true`` に設定することで指定可能となる。
    **基本的にはJPQLを使用する事を推奨するが、JPQLで表現できないQueryを発行する必要がある場合はNativeなSQLを直接指定してもよい。**
    データベースに依存するSQLを指定する場合は、SQLをプロパティファイルに定義することを検討すること。

    SQLをプロパティファイルに定義する方法については、「:ref:`how_to_specify_query_namedquery_properties-label`」を参照されたい。

 .. note:: **Named Parametersについて**

    Queryにバインドするパラメータに対して名前を付与し、Query内からはパラメータ名を指定することで値をバインドすることができる。
    Named Parameterを使用する場合は、 ``@org.springframework.data.repository.query.Param`` アノテーションを対象とする引数に追加し、value属性にパラメータ名を指定する。
    Queryでは、バインドしたい位置に「:パラメータ名」の形式で指定する。

    **特に理由がない場合は、 Qyeryのメンテナンス性と可読性を考慮し、Named Parametersを使用することを推奨する。**

|

| LIKE検索の一致方法(前方一致、後方一致、部分一致)が固定の場合は、JPQL内に ``"%"`` を指定することが出来る。
| ただし、これはJPQLの標準形式ではなくSpring Data JPAの拡張形式になるので、``@Query`` アノテーションで指定するJPQLでのみ指定することが出来る。
| Named queryとして指定するJPQL内に ``"%"`` を指定するとエラーになるので注意すること。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|p{0.50\linewidth}|
 .. list-table::
    :widths: 10 20 20 50
    :header-rows: 1

    * - 項番
      - 一致方法
      - 形式
      - 具体例
    * - 1.
      - 前方一致
      - | ``:parameterName%``
        | or
        | ``?n%``
      - | ``SELECT a FROM Account WHERE a.firstName LIKE :firstName%``
        | ``SELECT a FROM Account WHERE a.firstName LIKE ?1%``
    * - 2.
      - 後方一致
      - | ``%:parameterName``
        | or
        | ``%?n``
      - | ``SELECT a FROM Account WHERE a.firstName LIKE %:firstName``
        | ``SELECT a FROM Account WHERE a.firstName LIKE %?1``
    * - 3.
      - 部分一致
      - | ``%:parameterName%``
        | or
        | ``%?n%``
      - | ``SELECT a FROM Account WHERE a.firstName LIKE %:firstName%``
        | ``SELECT a FROM Account WHERE a.firstName LIKE %?1%``

 .. note:: **LIKE検索時のエスケープについて**

    LIKE検索を行う場合は、検索条件となる値をLIKE検索用にエスケープする必要がある。

    ``org.terasoluna.gfw.common.query.QueryEscapeUtils`` クラスにエスケープするためのメソッドが用意されているため、要件を充たせる場合は使用を検討すること。
    ``QueryEscapeUtils`` クラスの詳細については、「:doc:`DataAccessCommon`」の「:ref:`data-access-common_appendix_like_escape`」を参照されたい。

 .. note:: **一致方法を動的に変化させる必要がある場合**

    一致方法(前方一致、後方一致、部分一致)を動的に変化させる必要がある場合は、
    JPQL内に ``%`` を指定するのではなく、従来通りバインドするパラメータ値の前後に ``"%"`` を追加すること。

    ``org.terasoluna.gfw.common.query.QueryEscapeUtils`` クラスに一致方法に対応する検索条件値に変換するメソッドが用意されているため、
    要件を充たせる場合は、使用を検討すること。
    ``QueryEscapeUtils`` クラスの詳細については、「:doc:`DataAccessCommon`」の「:ref:`data-access-common_appendix_like_escape`」を参照されたい。

| ソート条件は、Query内に直接指定することができる。
| 以下に、実装例を示す。

 .. code-block:: sql

    // (1)
    @Query(value = "SELECT o FROM Order o WHERE o.status.code = :statusCode ORDER BY o.id DESC")
    Page<Order> findByStatusCode(@Param("statusCode") String statusCode, Pageable pageable);

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - Queryに ``"ORDER BY"`` を指定する。降順にする場合は ``DESC`` を、昇順にする場合は ``ASC`` を指定する。DESC/ASCを省略した場合は、 ``ASC`` が適用される。

|

| ソート条件はQuery内に直接指定する以外に、 ``Pageable`` オブジェクト内に保持している ``org.springframework.data.domain.Sort`` オブジェクトに指定することが出来る。
| この方法でソート条件を指定する場合は、countQuery属性の指定は不要。
| 以下に、``Pageable`` オブジェクト内に保持している ``Sort`` オブジェクトを使用してソートする実装例を示す。

- Controller

 .. code-block:: java

    @RequestMapping("list")
    public String list(@PageableDefault(
                            size=5,
                            sort = "id", // (1)
                            direction = Direction.DESC // (1)
                            ) Pageable pageable,
                              Model model) {
        Page<Order> orderPage = orderService.getOrders(pageable); // (2)
        model.addAttribute("orderPage", orderPage);
        return "order/list";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | ソート条件を指定する。 ``Pageable#getSort()`` メソッドで取得できる ``Sort`` オブジェクトにソート条件が設定される。
        | 上記例では、 idフィールドの降順をソート条件として指定している。
    * - | (2)
      - ``Pageable`` オブジェクトを指定してServiceのメソッドを呼び出す。

|

- Service (Caller)

 .. code-block:: java

    public String getOrders(Pageable pageable){
        return orderRepository.findByStatusCode("accepted", pageable); // (3)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (3)
      - Controllerから渡された ``Pageable`` オブジェクトを指定してRepositoryのメソッドを呼び出す。

|

- Repositoryインタフェース

 .. code-block:: java

    @Query(value = "SELECT o FROM Order o WHERE o.status.code = :statusCode") // (4)
    Page<Order> findByStatusCode(@Param("statusCode") String statusCode, Pageable pageable);

 .. code-block:: sql

    -- (5) statusCode='accepted'
    SELECT
            COUNT(order0_.id) AS col_0_0_
        FROM
            t_order order0_
        WHERE
            order0_.status_code = 'accepted'

    -- (6) statusCode='accepted'
    SELECT
            order0_.id AS id1_5_
            ,order0_.status_code AS status2_5_
        FROM
            t_order order0_
        WHERE
            order0_.status_code = 'accepted'
        ORDER BY
            order0_.id DESC
        LIMIT 5

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (4)
      - Queryに"ORDER BY"句の指定は行わない。countQuery属性の指定も不要。
    * - | (5)
      - JPQLから変換された件数カウント用のNativeなSQL。
    * - | (6)
      - | JPQLから変換された指定されたページ位置のEntityを取得するためのNativeなSQL。
        | Queryに指定はしていないが、\ ``Pageable``\ オブジェクト内に保持している ``Sort`` オブジェクトに指定した条件で"ORDER BY"句が追加される。例では、PostgreSQL用のSQLになっている。

.. _how_to_specify_query_mathodname-label:

命名規約ベースのメソッド名で指定する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Spring Dataが定めた命名規約に則ったメソッド名にすることで実行するQuery(JPQL)を指定する。
| Spring Data JPAの機能によってメソッド名からJPQLが生成される。
| ただし、メソッド名からJPQLを作成できるのはSELECTのみで、UPDATEおよびDELETEのJPQLは生成できない。

メソッド名からJPQLを生成するための命名規約などのルールについては、以下のページを参照されたい。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.45\linewidth}|p{0.45\linewidth}|
 .. list-table::
    :widths: 10 45 45
    :header-rows: 1

    * - 項番
      - 参照ページ
      - 説明
    * - 1.
      - `Spring Data Commons - Reference Documentation「Defining query methods」の「Query creation」 <http://docs.spring.io/spring-data/commons/docs/1.12.6.RELEASE/reference/html/#repositories.query-methods.query-creation>`_\
      - Distinct、ORDER BY、Case insensitiveの指定方法などが記載されている。
    * - 2.
      - `Spring Data Commons - Reference Documentation「Defining query methods」の「Property expressions」 <http://docs.spring.io/spring-data/commons/docs/1.12.6.RELEASE/reference/html/#repositories.query-methods.query-property-expressions>`_\
      - ネストされたEntityのプロパティを条件に指定する方法などが記載されている。
    * - 3.
      - `Spring Data Commons - Reference Documentation「Defining query methods」の「Special parameter handling」 <http://docs.spring.io/spring-data/commons/docs/1.12.6.RELEASE/reference/html/#repositories.special-parameters>`_\
      - 特別なメソッド引数(``Pageable`` 、 ``Sort``)についての説明が記載されている。
    * - 4.
      - `Spring Data JPA - Reference Documentation「Query methods」の「Query creation」 <http://docs.spring.io/spring-data/jpa/docs/1.10.6.RELEASE/reference/html/#jpa.query-methods.query-creation>`_\
      - JPQLを組み立てるための命名規約(キーワード)に関する説明が記載されている。
    * - 5.
      - `Spring Data Commons - Reference Documentation「Appendix C. Repository query keywords」 <http://docs.spring.io/spring-data/commons/docs/1.12.6.RELEASE/reference/html/#repository-query-keywords>`_\
      - JPQLを組み立てるための命名規約(キーワード)に関する説明が記載されている。

以下に、実装例を示す。

- OrderRepositry.java

 .. code-block:: java

    Page<Order> findByStatusCode(String statusCode, Pageable pageable); // (1)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | メソッド名が ``^(find|read|get).*By(.+)`` のパターンに一致する場合、メソッド名からJPQLを生成する対象のメソッドとなる。
        | ``(.+)`` の部分に条件となるEntityのプロパティや操作を示すキーワードを指定する。
        | 例では、``Order`` オブジェクトで保持している ``status`` プロパティ( ``OrderStatus`` 型)の ``code`` プロパティ( ``String`` 型) の値が指定したパラメータ値( ``statusCode`` )と一致する ``Order`` オブジェクトをページ形式で取得している。

- 件数カウント用Query

 .. code-block:: sql

    -- (2) JPQL
    SELECT
            COUNT(*)
        FROM
            ORDER AS generatedAlias0
                LEFT JOIN generatedAlias0.status AS generatedAlias1
            WHERE
                generatedAlias1.code = ?1

    -- (3) SQL statusCode='accepted'
    SELECT
            COUNT(*) AS col_0_0_
        FROM
            t_order order0_
                LEFT OUTER JOIN c_order_status orderstatu1_
                    ON order0_.status_code = orderstatu1_.code
        WHERE
            orderstatu1_.code = 'accepted'

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (2)
      - メソッド名から生成された件数カウント用のJPQLのQuery。
    * - | (3)
      - (2)のJPQLから変換された件数カウント用のNativeなSQL。

- Entity取得用Query

 .. code-block:: sql

    -- (4) JPQL
    SELECT
            generatedAlias0
        FROM
            ORDER AS generatedAlias0
                LEFT JOIN generatedAlias0.status AS generatedAlias1
            WHERE
                generatedAlias1.code = ?1
            ORDER BY
                generatedAlias0.id DESC;

    -- (5) statusCode='accepted'
    SELECT
            order0_.id AS id1_5_
            ,order0_.status_code AS status2_5_
        FROM
            t_order order0_
                LEFT OUTER JOIN c_order_status orderstatu1_
                    ON order0_.status_code = orderstatu1_.code
        WHERE
            orderstatu1_.code = 'accepted'
        ORDER BY
            order0_.id DESC
        LIMIT 5

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (4)
      - メソッド名から生成されたEntity取得用のJPQLのQuery。
    * - | (5)
      - (4)のJPQLから変換されたEntity取得用のNativeなSQL。

.. _how_to_specify_query_namedquery_properties-label:

プロパティファイルにNamed queryとして指定する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Spring Data JPAから提供されているプロパティファイル(classpath:META-INF/jpa-named-queries.properties)に、実行するQueryを指定する。

| **この方法は、QueryとしてNativeQueryを使用する際に、データベース固有のSQLを記載する必要が出た場合に使用するか検討すること。**
| **データベース固有のSQLであっても、実行環境に依存しないのであれば @Queryアノテーションに直接指定する方法を推奨する。**

- OrderRepositry.java

 .. code-block:: java

    @Query(nativeQuery = true)
    List<Order> findAllByStatusCode(@Param("statusCode") String statusCode); // (1)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - Named queryのLookup名は、Entityのクラス名とメソッド名を ``"."`` (dot) で連結したものが使用される。
        上記例だと、 ``"Order.findAllByStatusCode"`` がLookup名となる。

 .. tip:: **Named queryのLookup名を指定する方法**

    デフォルトの動作では、Entityのクラス名とメソッド名を ``"."`` (dot) で連結したものがLookup名として使用されるが、任意のクエリ名を指定することも出来る。

    * Entity取得用のLookup名に任意のクエリ名を指定したい場合は、 ``@Query`` アノテーションのname属性にクエリ名を指定する。
    * ページ検索時の件数カウント用のLookup名に任意のクエリ名を指定したい場合は、 ``@Query`` アノテーションのcountName属性にクエリ名を指定する。

 .. code-block:: java

    @Query(name = "OrderRepository.findAllByStatusCode", nativeQuery = true) // (2)
    List<Order> findAllByStatusCode(@Param("statusCode") String statusCode);

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (2)
      - 上記例では、``"OrderRepository.findAllByStatusCode"`` をLookup用のクエリ名として指定している。

|

- :file:`jpa-named-queries.properties`

 .. code-block:: properties

    # (3)
    Order.findAllByStatusCode=SELECT * FROM order WHERE status_code = :statusCode

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (3)
      - | クエリー名をキーとして、実行するSQLを指定する。
        | 上記例では、``"Order.findAllByStatusCode"`` をキーに、実行するSQLを指定している。

 .. tip::

    Spring Data JPAから提供されているプロパティファイルではなく、任意のプロパティファイルにNamed Queryを指定する方法を以下に紹介する。

    - ``xxx-infra.xml``

     .. code-block:: xml

         <!-- (4) -->
         <jpa:repositories base-package="xxxxxx.yyyyyy.zzzzzz.domain.repository"
             named-queries-location="classpath:META-INF/jpa/jpa-named-queries.properties" />

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
         :widths: 10 90
         :header-rows: 1
         :class: longtable

         * - 項番
           - 説明
         * - | (4)
           - | <jpa:repositories>要素のnamed-queries-location属性に、任意のプロパティファイルを指定する。
             | 上記例では、クラスパス上にある\ :file:`META-INF/jpa/jpa-named-queries.properties`\ が使用される。


Entityの検索処理の実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Entityの検索方法について、目的別に説明する。

条件に一致するEntityを全件検索
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
条件に一致するEntityを全件取得するQueryメソッドを呼び出す。

- Repositroyインタフェース

 .. code-block:: java

    public interface AccountRepository extends JpaRepository<Account, String> {

        // (1)
        @Query("SELECT a FROM Account a WHERE :createdDateFrom <= a.createdDate AND a.createdDate < :createdDateTo ORDER BY a.createdDate DESC")
        List<Account> findByCreatedDate(
                @Param("createdDateFrom") Date createdDateFrom,
                @Param("createdDateTo") Date createdDateTo);

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | ``java.util.List`` インタフェースを返却するQueryメソッドを定義する。

- Service

 .. code-block:: java

    public List<Account> getAccounts(Date targetDate) {
        LocalDate targetLocalDate = new LocalDate(targetDate);
        Date fromDate = targetLocalDate.toDate();
        Date toDate = targetLocalDate.dayOfYear().addToCopy(1).toDate();

        // (2)
        List<Account> accounts = accountRepository.findByCreatedDate(fromDate,
                toDate);
        if (accounts.isEmpty()) { // (3)
            // ...
        }
        return accounts;
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (2)
      - | Repositryインタフェースに実装したQueryメソッドを呼び出す。
    * - | (3)
      - | 検索結果が0件の場合は、空のリストが返却される。nullは返却されないのでnullチェックは不要。
        | 必要に応じて、検索結果が0件の場合の処理を実装する。

|

.. _DataAccessJpaHowToUseFindPage:

条件に一致するEntityのページ検索
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
条件に一致するEntityの中から指定ページに該当するEntityを取得するQueryメソッドを呼び出す。

- Repositroyインタフェース

 .. code-block:: java

    public interface AccountRepository extends JpaRepository<Account, String> {

        // (1)
        @Query("SELECT a FROM Account a WHERE :createdDateFrom <= a.createdDate AND a.createdDate < :createdDateTo")
        Page<Account> findByCreatedDate(
                @Param("createdDateFrom") Date createdDateFrom,
                @Param("createdDateTo") Date createdDateTo, Pageable pageable);

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | 引数として ``org.springframework.data.domain.Pageable`` インタフェースを受け取り、 ``org.springframework.data.domain.Page`` インタフェースを返却するQueryメソッドを定義する。

- Controller

 .. code-block:: java

    @RequestMapping("list")
    public String list(@RequestParam("targetDate") Date targetDate,
                       @PageableDefault(
                           page = 0,
                           value = 5,
                           sort = { "createdDate" },
                           direction = Direction.DESC)
                           Pageable pageable, // (2)
                       Model model) {
        Page<Order> accountPage = accountService.getAccounts(targetDate, pageable);
        model.addAttribute("accountPage", accountPage);
        return "account/list";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (2)
      - | Spring Dataより提供されているページング検索用のオブジェクト（  ``org.springframework.data.domain.Pageable`` ）を生成する。
        | 詳細は「:doc:`../WebApplicationDetail/Pagination`」を参照されたい。


- Service

 .. code-block:: java

    public Page<Account> getAccounts(Date targetDate ,Pageable pageable) {

        LocalDate targetLocalDate = new LocalDate(targetDate);
        Date fromDate = targetLocalDate.toDate();
        Date toDate = targetLocalDate.dayOfYear().addToCopy(1).toDate();

        // (3)
        Page<Account> page = accountRepository.findByCreatedDate(fromDate,
                toDate, pageable);
        if (!page.hasContent()) { // (4)
            // ...
        }
        return page;
    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (3)
      - | Repositryインタフェースに実装したQueryメソッドを呼び出す。
    * - | (4)
      - | 検索結果が0件の場合は、``Page`` オブジェクトに空のリストが設定され、 ``Page#hasContent()`` メソッドの返り値が ``false`` になる。
        | 必要に応じて、検索結果が0件の場合の処理を実装する。


|

Entityの動的条件による検索処理の実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
動的条件によるEntityの検索を行うQueryメソッドをRepositoryメソッドに追加する場合は、 Entity毎のRepositoryインタフェースに対して、
カスタムRepositoryインタフェースとカスタムRepositoryインタフェースの実装クラスを用意する方法で実装する。
カスタムRepositoryインタフェースとカスタムRepositoryクラスの作成方法については、「:ref:`custommethod_individual-label`」を参照されたい。

以降では、動的条件を適用してEntityを検索する方法について、目的別に説明する。

.. todo::

    **TBD**

    今後、以下の内容を追加する予定である。

    * QueryDSLを使用した動的Queryの実装例。

|

動的条件に一致するEntityを全件検索
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
動的条件に一致するEntityを全件取得するQueryメソッドを実装し、呼び出す。

以下の実装例を示す。

実装例では、動的条件として、

* 注文ID
* 商品名
* 注文状態(複数指定可能)

を指定可能とし、指定された条件に一致する注文をAND条件で絞り込む検索とする。
なお、条件の指定がない場合は、検索は行わず空のリストを返却する。

- Criteria (JavaBean)

 .. code-block:: java

    public class OrderCriteria implements Serializable { // (1)

        private Integer id;

        private String itemName;

        private List<String> statusCodes;

        // ...

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | 検索条件を保持するCriteriaオブジェクト(JavaBean)を作成する。

- カスタムRepositoryインタフェース

 .. code-block:: java

    public interface OrderRepositoryCustom {

        Page<Order> findAllByCriteria(OrderCriteria criteria); // (2)

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (2)
      - | カスタムRepositoryインタフェースに、Criteriaオブジェクトを引数にとり、Listを返却するメソッドを定義する。

- カスタムRepositoryクラス

 .. code-block:: java

    public class OrderRepositoryImpl implements OrderRepositoryCustom { // (3)

        @PersistenceContext
        EntityManager entityManager; // (4)

        public List<Order> findAllByCriteria(OrderCriteria criteria) { // (5)

            // Collect dynamic conditions.
            // (6)
            final List<String> andConditions = new ArrayList<String>();
            final List<String> joinConditions = new ArrayList<String>();
            final Map<String, Object> bindParameters = new HashMap<String, Object>();

            // (7)
            if (criteria.getId() != null) {
                andConditions.add("o.id = :id");
                bindParameters.put("id", criteria.getId());
            }
            if (!CollectionUtils.isEmpty(criteria.getStatusCodes())) {
                andConditions.add("o.status.code IN :statusCodes");
                bindParameters.put("statusCodes", criteria.getStatusCodes());
            }
            if (StringUtils.hasLength(criteria.getItemName())) {
                joinConditions.add("o.orderItems oi");
                joinConditions.add("oi.item i");
                andConditions.add("i.name LIKE :itemName ESCAPE '~'");
                bindParameters.put("itemName", QueryEscapeUtils
                        .toLikeCondition(criteria.getItemName()));
            }

            // (8)
            if (andConditions.isEmpty()) {
                return Collections.emptyList();
            }

            // (9)
            // Create dynamic query.
            final StringBuilder queryString = new StringBuilder();

            // (10)
            queryString.append("SELECT o FROM Order o");

            // (11)
            // add join conditions.
            for (String joinCondition : joinConditions) {
                queryString.append(" LEFT JOIN ").append(joinCondition);
            }
            // add conditions.
            Iterator<String> andConditionsIt = andConditions.iterator();
            if (andConditionsIt.hasNext()) {
                queryString.append(" WHERE ").append(andConditionsIt.next());
            }
            while (andConditionsIt.hasNext()) {
                queryString.append(" AND ").append(andConditionsIt.next());
            }

            // (12)
            // add order by condition.
            queryString.append(" ORDER BY o.id");

            // (13)
            // Create typed query.
            final TypedQuery<Order> findQuery = entityManager.createQuery(
                    queryString.toString(), Order.class);
            // Bind parameters.
            for (Map.Entry<String, Object> bindParameter : bindParameters
                    .entrySet()) {
                findQuery.setParameter(bindParameter.getKey(), bindParameter
                        .getValue());
            }

            // (14)
            // Execute query.
            return findQuery.getResultList();

        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1
    :class: longtable

    * - 項番
      - 説明
    * - | (3)
      - | カスタムRepositoryインタフェースの実装クラスを作成する。
    * - | (4)
      - | ``EntityManager`` をInjectする。
        | ``@javax.persistence.PersistenceContext`` アノテーションを使用してInjectすること。
    * - | (5)
      - | 動的条件に一致するEntityを全件取得するQueryメソッドを実装する。
        | 上記実装例では説明のためにメソッド分割はしていないが、必要に応じてメソッド分割すること。
    * - | (6)
      - | 動的クエリを組み立てるための変数（AND条件用リスト、結合条件用リスト、バインドパラメータ用マップ）を定義している。
        | OrderCriteriaオブジェクトに条件の指定があるものについて、これらの変数に必要な情報を設定する。
    * - | (7)
      - | OrderCriteriaオブジェクトに条件の指定があるか判定し、動的クエリを組み立てるために必要な情報を設定していく。
        | 上記例では、``id`` は指定した値と完全一致するもの、 ``statusCodes`` は指定したリストに含まれるもの、 ``itemName`` は指定した値と前方一致するものを取得対象とするための情報を設定している。
        | ``itemName`` については、比較対象の値を保持する関連Entityが複雑なネスト関係になっているため、関連EntityをJOINする必要がある。
    * - | (8)
      - | 上記実装例では条件が指定されていない場合は、検索する必要がないため、空のリストを返却する。
    * - | (9)
      - | 条件が指定されている場合は、Entityを検索するためのQueryを組み立てる。
        | 上記例では実行するQueryを ``java.lang.StringBuilder`` クラスを使って組み立てる実装例となっている。
    * - | (10)
      - | 静的なQuery要素を組み立てる。
        | 上記例では、SELECT句とFROM句は静的なQueryの構成要素として組み立てている。
    * - | (11)
      - | 動的なQuery要素を組み立てる。
        | 上記例では、結合条件用リスト(JOIN句)とAND条件用リスト(WHERE句)に設定する条件を動的なQuery要素として組み立てている。
    * - | (12)
      - | 静的なQuery要素を組み立てる。
        | 上記例では、ORDER BY句は静的なQuery要素として組み立てている。
    * - | (13)
      - | 動的に組み立てたQuery文字列を ``javax.persistence.TypedQuery`` に変換し、Query実行に必要なバインド用のパラメータを設定する。
    * - | (14)
      - | 動的に組み立てたQueryを実行し、条件に一致するEntityを全件取得する。

 .. raw:: latex

    \newpage

- Entity毎のRepositoryインタフェース

 .. code-block:: java

    public interface OrderRepository extends JpaRepository<Order, Integer>,
                                    OrderRepositoryCustom { // (15)
        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (15)
      - | Entity毎のRepositoryインタフェースに、カスタムRepositoryインタフェースを継承する。


- Service (Caller)

 .. code-block:: java

    // condition values for sample.
    Integer conditionValueOfId = 4;
    List<String> conditionValueOfStatusCodes = Arrays.asList("accepted");
    String conditionValueOfItemName = "Wat";

    // implementation of sample.
    // (16)
    OrderCriteria criteria = new OrderCriteria();
    criteria.setId(conditionValueOfId);
    criteria.setStatusCodes(conditionValueOfStatusCodes);
    criteria.setItemName(conditionValueOfItemName);
    List<Order> orders = orderRepository.findAllByCriteria(criteria); // (17)
    if (orders.isEmpty()) { // (18)
        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (16)
      - | OrderCriteriaオブジェクトに検索条件を指定する。
    * - | (17)
      - | OrderCriteriaオブジェクトを引数として、動的条件に一致するEntityを全件取得するQueryメソッドを呼び出す。
    * - | (18)
      - | 必要に応じて検索結果を判定し、0件の場合の処理を行う。

- 実行されるJPQL(SQL)

 .. code-block:: sql

    -- (19)
    --   conditionValueOfId=4
    --   conditionValueOfStatusCodes = ["accepted"]
    --   conditionValueOfItemName = "Wat"

    -- JPQL
    SELECT
            o
        FROM
            ORDER o
                JOIN o.orderItems oi
                    JOIN oi.item i
                WHERE
                    o.id = :id
                    AND o.status.code IN :statusCodes
                    AND i.name LIKE :itemName ESCAPE '~'
                ORDER BY
                    o.id

    -- SQL
    SELECT
            order0_.id AS id1_6_
            ,order0_.created_by AS created2_6_
            ,order0_.created_date AS created3_6_
            ,order0_.last_modified_by AS last4_6_
            ,order0_.last_modified_date AS last5_6_
            ,order0_.status_code AS status6_6_
        FROM
            t_order order0_ INNER JOIN t_order_item orderitems1_
                ON order0_.id = orderitems1_.order_id INNER JOIN m_item item2_
                ON orderitems1_.item_code = item2_.code
    WHERE
        order0_.id = 4
        AND (
            order0_.status_code IN ('accepted')
        )
        AND (
            item2_.name LIKE 'Wat%' ESCAPE '~'
        )
    ORDER BY
        order0_.id

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (19)
      - | すべての条件を指定した場合に発行されるJPQLとSQLの例。

 .. code-block:: sql

    -- (20)
    --   conditionValueOfId=4
    --   conditionValueOfStatusCodes = ["accepted"]
    --   conditionValueOfItemName = ""
    -- JPQL
    SELECT
            o
        FROM
            ORDER o
        WHERE
            o.id = :id
            AND o.status.code IN :statusCodes
        ORDER BY
            o.id

    -- SQL
    SELECT
            order0_.id AS id1_6_
            ,order0_.created_by AS created2_6_
            ,order0_.created_date AS created3_6_
            ,order0_.last_modified_by AS last4_6_
            ,order0_.last_modified_date AS last5_6_
            ,order0_.status_code AS status6_6_
        FROM
            t_order order0_
        WHERE
            order0_.id = 4
            AND (
                order0_.status_code IN ('accepted')
            )
        ORDER BY
            order0_.id;

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (20)
      - | ``iteName`` 以外の条件を指定した場合に発行されるJPQLとSQLの例。

 .. code-block:: sql

    -- (21)
    --   conditionValueOfId=4
    --   conditionValueOfStatusCodes = []
    --   conditionValueOfItemName = ""
    -- JPQL
    SELECT
            o
        FROM
            ORDER o
        WHERE
            o.id = :id
        ORDER BY
            o.id

    -- SQL
    SELECT
            order0_.id AS id1_6_
            ,order0_.created_by AS created2_6_
            ,order0_.created_date AS created3_6_
            ,order0_.last_modified_by AS last4_6_
            ,order0_.last_modified_date AS last5_6_
            ,order0_.status_code AS status6_6_
        FROM
            t_order order0_
        WHERE
            order0_.id = 4
        ORDER BY
            order0_.id;

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (21)
      - | ``id`` のみを指定した場合に発行されるJPQLとSQLの例。


|

動的条件に一致するEntityをページ検索
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
動的条件に一致するEntityの中から指定ページに該当するEntityを取得するQueryメソッドを実装し、呼び出す。

以下に実装例を示すが、該当ページを取得する箇所以外は、全件取得する場合と同じ仕様とする。
なお、全件取得で説明した箇所の説明は省略する。

- カスタムRepositoryインタフェース

 .. code-block:: java

    public interface OrderRepositoryCustom {

        Page<Order> findPageByCriteria(OrderCriteria criteria, Pageable pageable); // (1)

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | 動的条件に一致するEntityの中から指定ページに該当するEntityを取得するQueryメソッドを定義する。


- カスタムRepositoryクラス

 .. code-block:: java

    public class OrderRepositoryCustomImpl implements OrderRepositoryCustom {

        @PersistenceContext
        EntityManager entityManager;

        public Page<Order> findPageByCriteria(OrderCriteria criteria,
                Pageable pageable) { // (2)

            // collect dynamic conditions.
            final List<String> andConditions = new ArrayList<String>();
            final List<String> joinConditions = new ArrayList<String>();
            final Map<String, Object> bindParameters = new HashMap<String, Object>();

            if (criteria.getId() != null) {
                andConditions.add("o.id = :id");
                bindParameters.put("id", criteria.getId());
            }
            if (!CollectionUtils.isEmpty(criteria.getStatusCodes())) {
                andConditions.add("o.status.code IN :statusCodes");
                bindParameters.put("statusCodes", criteria.getStatusCodes());
            }
            if (StringUtils.hasLength(criteria.getItemName())) {
                joinConditions.add("o.orderItems oi");
                joinConditions.add("oi.item i");
                andConditions.add("i.name LIKE :itemName ESCAPE '~'");
                bindParameters.put("itemName", QueryEscapeUtils.toLikeCondition(criteria
                        .getItemName()));
            }

            if (andConditions.isEmpty()) {
                List<Order> orders = Collections.emptyList();
                return new PageImpl<Order>(orders, pageable, 0); // (3)
            }

            // create dynamic query.
            final StringBuilder queryString = new StringBuilder();
            final StringBuilder countQueryString = new StringBuilder(); // (4)
            final StringBuilder conditionsString = new StringBuilder(); // (4)

            queryString.append("SELECT o FROM Order o");
            countQueryString.append("SELECT COUNT(o) FROM Order o"); // (5)

            // add join conditions.
            for (String joinCondition : joinConditions) {
                conditionsString.append(" JOIN ").append(joinCondition);
            }

            // add conditions.
            Iterator<String> andConditionsIt = andConditions.iterator();
            if (andConditionsIt.hasNext()) {
                conditionsString.append(" WHERE ").append(andConditionsIt.next());
            }
            while (andConditionsIt.hasNext()) {
                conditionsString.append(" AND ").append(andConditionsIt.next());
            }
            queryString.append(conditionsString); // (6)
            countQueryString.append(conditionsString); // (6)

            // add order by condition.
            // (7)
            String orderByString = QueryUtils.applySorting("", pageable.getSort(), "o");
            queryString.append(orderByString);

            // create typed query.
            final TypedQuery<Long> countQuery = entityManager.createQuery(
                    countQueryString.toString(), Long.class); // (8)

            final TypedQuery<Order> findQuery = entityManager.createQuery(
                    queryString.toString(), Order.class);

            // bind parameters.
            for (Map.Entry<String, Object> bindParameter : bindParameters
                    .entrySet()) {
                countQuery.setParameter(bindParameter.getKey(), bindParameter
                        .getValue()); // (8)
                findQuery.setParameter(bindParameter.getKey(), bindParameter
                        .getValue());
            }

            long total = countQuery.getSingleResult().longValue(); // (9)
            List<Order> orders = null;
            if (total != 0) { // (10)
                findQuery.setFirstResult(pageable.getOffset());
                findQuery.setMaxResults(pageable.getPageSize());
                // execute query.
                orders = findQuery.getResultList();
            } else { // (11)
                orders = Collections.emptyList();
            }

            return new PageImpl<Order>(orders, pageable, total); // (12)
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1
    :class: longtable

    * - 項番
      - 説明
    * - | (2)
      - | 動的条件に一致するEntityの中から指定ページに該当するEntityを取得するQueryメソッドを実装する。
        | 上記実装例では説明のためにメソッド分割はしていないが、必要に応じてメソッド分割すること。
    * - | (3)
      - | 上記実装例では条件が指定されていない場合は、検索する必要がないため、空のページ情報を返却する。
    * - | (4)
      - | 件数取得用のQueryを組み立てるための変数と、条件(結合条件とAND条件)を組み立てるための変数を用意する。
        | Entity取得用のQueryと件数取得用のQueryで同じ条件を使用する必要があるため、条件(結合条件とAND条件)を組み立てるための変数を用意している。
    * - | (5)
      - | 件数取得用Queryの静的なQuery要素を組み立てる。
        | 上記例では、SELECT句とFROM句は静的なQueryの構成要素として組み立てている。
    * - | (6)
      - | Entity取得用Queryと件数取得用Queryの動的なQuery要素を組み立てる。
    * - | (7)
      - | Entity取得用Queryに対してソート条件(ORDER BY句)を動的なQuery要素として組み立てている。
        | ORDER BY句の組み立ては、 Spring Data JPAから提供されているユーティリティ部品( ``org.springframework.data.jpa.repository.query.QueryUtils`` )を使用している。
    * - | (8)
      - | 動的に組み立てた件数取得用のQuery文字列を ``javax.persistence.TypedQuery`` に変換し、Query実行に必要なバインド用のパラメータを設定する。
    * - | (9)
      - | 件数取得用のQueryを実行し、条件に一致する合計件数を取得する。
    * - | (10)
      - | 条件に一致するEntityが存在する場合は、Entity取得用Queryを実行し、該当ページの情報を取得する。
        | Entity取得用Queryを実行する際は、取得開始位置( ``TypedQuery#setFirstResult`` )と取得件数( ``TypedQuery#setMaxResults`` )を指定する。
    * - | (11)
      - | 条件に一致するEntityが存在しない場合は、空のリストを取得結果とする。
    * - | (12)
      - | 該当ページのEntityのリスト、ページ情報、条件に条件した合計件数を引数に指定して、 ``Page`` オブジェクトを生成し返却する。

 .. raw:: latex

    \newpage

- Service (Caller)

 .. code-block:: java

    // condition values for sample.
    Integer conditionValueOfId = 4;
    List<String> conditionValueOfStatusCodes = Arrays.asList("accepted");
    String conditionValueOfItemName = "Wat";

    // implementation of sample.
    OrderCriteria criteria = new OrderCriteria();
    criteria.setId(conditionValueOfId);
    criteria.setStatusCodes(conditionValueOfStatusCodes);
    criteria.setItemName(conditionValueOfItemName);
    Page<Order> orderPage = orderRepository.findPageByCriteria(criteria,
            pageable); // (13)
    if (!orderPage.hasContent()) {
        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (13)
      - | 動的条件に一致するEntityの中から指定ページに該当するEntityを取得するQueryメソッドを呼び出す。


|

Entityの取得処理の実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Entityの取得方法について、目的別に説明する。

|

IDを指定してEntityを1件取得
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
ID(Primary Key)がわかっている場合は、RepositryインタフェースのfindOneメソッドを呼び出してEntityオブジェクトを取得する。

 .. code-block:: java

    public Account getAccount(String accountUuid) {
        Account account = accountRepository.findOne(accountUuid); // (1)
        if (account == null) { // (2)
            // ...
        }
        return account;
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | EntityのID(Primary Key)を指定して、RepositryインタフェースのfindOne(ID)メソッドを呼び出す。
    * - | (2)
      - | 指定したIDのEntityが存在しない場合は、返却値がnullになるので、null判定が必要となる。
        | 必要に応じて、指定したIDのEntityが存在しない場合の処理を実装する。

 .. note :: **返却されるEntityオブジェクトについて**

    指定したIDのEntityオブジェクトが、既に ``EntityManager`` 上で管理されている場合は、永続層(DB)へのアクセスは行われずに、
    ``EntityManager`` 上で管理されているEntityオブジェクトが返却される。
    そのため、 findOneメソッドを使用すると、永続層への無駄なアクセスを抑えることが出来る。

 .. note :: **関連Entityのロードタイミングについて**

    Query実行時の関連Entityのロードは、Entityの関連付けアノテーション( ``@javax.persistence.OneToOne`` 、 ``@javax.persistence.OneToMany`` 、 ``@javax.persistence.ManyToOne`` 、 ``@javax.persistence.ManyToMany`` )の
    fetch属性に指定されている値によって決定される。

    * ``javax.persistence.FetchType#LAZY`` の場合は、JOIN FETCH対象からはずれるため、、関連Entityのロードは初回アクセス時に行われる。
    * ``javax.persistence.FetchType#EAGER`` の場合は、 JOIN FETCHされるため、関連Entityがロードされる。

    fetch属性のデフォルトはアノテーションによって異なる。デフォルト値は以下の通り。

    * ``@OneToOne`` アノテーション : ``EAGER``
    * ``@ManyToOne`` アノテーション : ``EAGER``
    * ``@OneToMany`` アノテーション : ``LAZY``
    * ``@ManyToMany`` アノテーション : ``LAZY``

 .. note:: **1:N(N:M)の関連をもつ関連Entityの並び順について**

    関連Entityのプロパティのアノテーションに、 ``@javax.persistence.OrderBy`` アノテーションを指定して制御する。

 例を以下に示す。

  .. code-block:: java

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    @OrderBy // (1)
    private Set<OrderItem> orderItems;

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | value属性を指定しない場合は、 IDの昇順となる。
        | 詳細は、 `JSR 338: Java Persistence API, Version 2.1のSpecification(PDF)「11.1.42 OrderBy Annotation」 <http://download.oracle.com/otn-pub/jcp/persistence-2_1-fr-eval-spec/JavaPersistence.pdf>`_\

 .. todo::

    **TBD**

    今後、以下の内容を追加する予定である。

    * fetch属性をデフォルト値から変更した方がよいケースの例。

|

ID以外の条件を指定してEntityを1件取得
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
IDがわからない場合は、ID以外の条件でEntityを検索するQueryメソッドを呼び出す。

- Repositroyインタフェース

 .. code-block:: java

    public interface AccountRepository extends JpaRepository<Account, String> {

        // (1)
        @Query("SELECT a FROM Account a WHERE a.accountId = :accountId")
        Account findByAccountId(@Param("accountId") String accountId);

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | ID以外の項目を条件として、Entityを1件検索するQueryメソッドを定義する。

- Service

 .. code-block:: java

    public Account getAccount(String accountId) {
        Account account = accountRepository.findByAccountId(accountId); // (2)
        if (account == null) { // (3)
            // ...
        }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (2)
      - | Repositryインタフェースに実装したQueryメソッドを呼び出す。
    * - | (3)
      - | RepositryインタフェースのfindOneメソッドと同様に、条件に一致するEntityが存在しない場合は、返却値がnullになるので、null判定が必要となる。
        | 必要に応じて、指定した条件に一致するEntityが存在しない場合の処理を実装する。
        | 条件に一致するEntityが複数存在した場合は、 ``org.springframework.dao.IncorrectResultSizeDataAccessException`` が発生する。

 .. note :: **返却されるEntityオブジェクトについて**

    Queryメソッドを呼び出した場合、必ず永続層(DB)に対してQueryが実行される。
    ただし、Queryを実行して取得されたEntityが既に ``EntityManager`` 上で管理されている場合は、 Queryを実行して取得されたEntityオブジェクトは破棄され、 ``EntityManager``
    上で管理されているEntityオブジェクトが返却される。

 .. note :: **ID + α を条件とするQueryメソッドについて**

    ID + α を条件とするQueryメソッドは、原則作成しないようにすることを推奨する。
    このケースは、 findOneメソッドを呼び出して取得したEntityオブジェクトのプロパティ値を比較するロジックを組むことで、同じことが実現できる。

    原則作成しないことを推奨する理由は、Queryを実行して取得されたEntityオブジェクトが破棄される可能性があり、無駄なQueryの実行となってしまうためである。
    IDがわかっているのであれば、 無駄なQueryの実行が行われないfindOneメソッドを使用した方がよい。
    特に、性能要件が高いアプリケーションの場合は、意識して実装すること。

    ただし、以下の条件に当てはまる場合は、Queryメソッドを使用した方がQueryの実行回数が少なくなる事もあるので、その場合はQueryメソッドを使用してもよい。

    * \+ α の条件の部分に関連オブジェクトのプロパティ(関連テーブルのカラム)が含まれる。
    * 条件となる関連EntityへのFetchTypeに ``LAZY`` が含まれる。


 .. note :: **関連Entityのロードタイミングについて**

    JOIN FETCHに指定された関連Entityは、Query実行直後にロードされる。

    JOIN FETCHに指定がない関連Entityは、関連付けアノテーション( ``@OneToOne`` 、 ``@OneToMany`` 、 ``@ManyToOne`` 、 ``@ManyToMany`` )の
    fetch属性に指定されている値によって以下の動作になる。

    * ``javax.persistence.FetchType#LAZY`` の場合は、Lazy Load対象となり、関連Entityのロードは初回アクセス時に行われる。
    * ``javax.persistence.FetchType#EAGER`` の場合は、 関連EntityをロードするためのQueryが実行され、関連Entityのオブジェクトがロードされる。

 .. note:: 1:N(N:M)の関連をもつ関連Entityの並び順について

   * JOIN FETCHに指定された関連Entityの並び順は、JPQLに"ORDER BY"句を指定して制御する。
   * Query実行後にロードされる関連Entityの並び順は、  関連Entityのプロパティのアノテーションに、 ``@javax.persistence.OrderBy`` アノテーションを指定して制御する。

|


Entityの追加処理の実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Entityの追加方法について、目的別に実装例を説明する。

.. _data-access-jpa_how_to_use_way_to_add_entity:

Entityの追加
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Entityを追加したい場合は、Entityオブジェクトを生成し、Repositryインタフェースのsaveメソッドを呼び出す。

- Service

 .. code-block:: java

    Order order = new Order("accepted"); // (1)
    order = orderRepository.save(order); // (2)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | Entityオブジェクトのインスタンスを生成し、必要なプロパティに値を設定する。
        | 上記例では、IDの設定はJPAのID採番機能を使用している。JPAのID採番機能を使用する場合は、アプリケーションコードでIDの設定は行ってはいけない。
        | アプリケーションコードでIDを設定してしまうと、 ``EntityManager`` のmergeメソッドが呼び出されるため、不要な処理が実行されてしまう。
    * - | (2)
      - | Repositoryインタフェースのsaveメソッドを呼び出し、(1)で生成したEntityオブジェクトを ``EntityManager`` の管理対象にする。
        | ``EntityManager`` によって管理されるEntityオブジェクトは、saveメソッドの引数に渡したEntityオブジェクトではなく、saveメソッドから返却されたEntityオブジェクトになるので注意すること。
        | IDは、このタイミングでJPAのID採番機能によって設定される。

 .. note:: **mergeメソッドが呼び出されるデメリット**

    ``EntityManager`` のmergeメソッドは、Entityを ``EntityManager`` の管理対象にする際に、永続層(DB)から同じIDをもつEntityを取得する仕組みになっている。
    Entityの追加処理としては、Entityの取得処理は無駄な処理となる。高い性能要件があるアプリケーションの場合は、IDの採番タイミングを意識すること。

 .. note:: **制約エラーのハンドリング**

    saveメソッドを呼び出したタイミングでは、永続層(DB)にQuery(INSERT)は実行されない。
    そのため、一意制約違反などの制約系のエラーをハンドリングする必要がある場合は、Repositoryインタフェースのsaveメソッドではなく、
    saveAndFlushメソッドまたはflushメソッドを呼び出す必要がある。


- Entity

 .. code-block:: java

    @Entity // (3)
    @Table(name = "t_order") // (4)
    public class Order implements Serializable {

        // (5)
        @SequenceGenerator(name = "GEN_ORDER_ID", sequenceName = "s_order_id",
                           allocationSize = 1)
        @GeneratedValue(strategy = GenerationType.SEQUENCE,
                        generator = "GEN_ORDER_ID")
        @Id // (6)
        private int id;

        // (7)
        @ManyToOne
        @JoinColumn(name = "status_code")
        private OrderStatus status;

        // ...

        public Order(String statusCode) {
            this.status = new OrderStatus(statusCode);
        }

        // ...

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (3)
      - | Entityクラスには ``@javax.persistence.Entity`` アノテーションを付与する。
    * - | (4)
      - | Entityクラスとマッピングするテーブル名は ``@javax.persistence.Table`` アノテーションのname属性に指定する。
        | エンティティ名からテーブル名が解決できる場合は指定しなくてもよいが、解決できない場合は指定すること。
    * - | (5)
      - | JPAのID採番機能を使用するために必要なアノテーションを指定している。
        | JPAのID採番機能を使用する場合は、 ``@javax.persistence.GeneratedValue`` アノテーションを指定する。
        | シーケンスオブジェクトを使用する場合は、 ``@javax.persistence.SequenceGenerator`` アノテーション、採番テーブルを使用する場合は、 ``@javax.persistence.TableGenerator`` アノテーションの指定も必要となる。
        | 上記例では、 ``"s_order_id"`` という名前のシーケンスオブジェクトを使ってIDを採番している。
    * - | (6)
      - | 主キーを保持するプロパティに、 ``@javax.persistence.Id`` アノテーションを付与する。
        | 複合キーの場合は、 ``@javax.persistence.EmbeddedId`` アノテーションを付与する。
    * - | (7)
      - | 別のEntityとの関連を保持するプロパティに、 関連付けアノテーション( ``@OneToOne`` 、 ``@OneToMany`` 、 ``@ManyToOne`` 、 ``@ManyToMany`` )を付与する。

 .. note :: **ID採番用のアノテーションについて**

    各アノテーションの詳細は、`JSR 338: Java Persistence API, Version 2.1のSpecification(PDF) <http://download.oracle.com/otn-pub/jcp/persistence-2_1-fr-eval-spec/JavaPersistence.pdf>`_\ を参照されたい。

    * ``@GeneratedValue`` : 11.1.20 GeneratedValue Annotation
    * ``@SequenceGenerator`` : 11.1.48 SequenceGenerator Annotation
    * ``@TableGenerator`` : 11.1.50 TableGenerator Annotation

 .. note :: **IDの採番方法について**

    採番方法は、 ``@GeneratedValue`` アノテーションのstrategy属性に ``javax.persistence.GenerationType`` 列挙の値を指定する。
    指定可能な値は以下の通り。

    * ``TABLE`` : 永続層(DB)のテーブルを使用してIDを採番する。
    * ``SEQUENCE`` : 永続層(DB)のシーケンスオブジェクトを使用してIDを採番する。
    * ``IDENTITY`` : 永続層(DB)のidentity列を使用してIDを採番する。
    * ``AUTO`` : 永続層(DB)に最も適切な採番方法を選択してIDを採番する。

    原則 ``AUTO`` は使用せず、明示的に使用するタイプを指定することを推奨する。


|

Entityと関連Entityの追加
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Entityと関連Entityを一緒に追加したい場合は、Repositryインタフェースのsaveメソッドを呼び出してEntityオブジェクトを ``EntityManager`` の管理対象にした後に、
関連Entityのオブジェクトを生成し、Entityオブジェクトに関連づける。

この方法を使用するためには、関連EntityのCascade対象の操作に、``persist`` が含まれている必要がある。

 .. note:: **Cascade対象となった場合の挙動について**

    関連EntityがCascade対象に指定されている場合、Entityに対するJPAの操作( ``persist`` , ``merge`` , ``remove`` , ``refresh`` , ``detach`` )
    が関連Entityに対して、連鎖して行われる。

    Spring Data JPAのRepositoryインタフェースの操作とのマッピングは以下の通り。

    * saveメソッド : ``persist`` or ``merge``
    * deleteメソッド : ``remove``

    ``refresh`` , ``detach`` は Spring Data JPAのデフォルト実装で実行されることはない。


- Service

 .. code-block:: java

    String itemCode = "ITM0000001";
    int itemQuantity = 10;
    String wayToPay = "card";

    Order order = new Order("accepted");
    order = orderRepository.save(order); // (1)

    OrderItem orderItem = new OrderItem(order.getId(), itemCode,
            itemQuantity);
    order.setOrderItems(Collections.singleton(orderItem));    // (2)
    order.setOrderPay(new OrderPay(order.getId(), wayToPay)); // (2)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | Repositoryインタフェースのsaveメソッドを呼び出し、まずEntityオブジェクトを ``EntityManager`` の管理対象にする。
        | 上記例では、Entityオブジェクトに対して、関連Entityのオブジェクトを設定する前に、saveメソッドを呼び出している。これは、関連EntityのIDの一部として使用されるEntityのID(orderId)を採番する必要があるためである。
    * - | (2)
      - | ``EntityManager`` の管理対象となっているEntityオブジェクトに対して、関連Entityのオブジェクトを設定する。
        | トランザクションのコミット時に、Entityオブジェクトへのpersist操作(INSERT)が、関連Entityのオブジェクトに対して連鎖される。

- Entity

 .. code-block:: java

    @Entity
    @Table(name = "t_order")
    public class Order implements Serializable {

        private static final long serialVersionUID = 1L;

        @Id
        @SequenceGenerator(name = "GEN_ORDER_ID", sequenceName = "s_order_id",
                           allocationSize = 1)
        @GeneratedValue(strategy = GenerationType.SEQUENCE,
                        generator = "GEN_ORDER_ID")
        private int id;

        @OneToMany(mappedBy = "order", cascade = CascadeType.ALL,  // (3)
                   orphanRemoval = true)
        @OrderBy
        private Set<OrderItem> orderItems;

        @OneToOne(mappedBy = "order", cascade = CascadeType.ALL,
                  orphanRemoval = true)
        private OrderPay orderPay;

        @ManyToOne
        @JoinColumn(name = "status_code")
        private OrderStatus status;

        // ...

        public Order(String statusCode) {
            this.status = new OrderStatus(statusCode);
        }

        // ...

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (3)
      - | 関連アノテーションのcascade属性に、Cascade対象にする操作のタイプ( ``javax.persistence.CascadeType`` )を指定する。
        | 上記例では、すべての操作を関連EntityのCascade対象としている。
        | 特に理由がない場合は、すべての操作をCascade対象にしておくことを推奨する。

 .. warning :: **cascade属性を指定してはいけない関連Entityについて**

    トランザクション系のEntityからコード系およびマスタ系のEntityに関連をもつ場合は、cascade属性は指定してはいけない。
    上記例だと、``OrderStatus`` はコード系のEntityになるので ``Order`` の ``status`` プロパティの ``@ManyToOne`` アノテーションにcascade属性は指定してはいけない。


- 関連Entity

 .. code-block:: java

    @Entity
    @Table(name = "t_order_item")
    public class OrderItem implements Serializable {

        @EmbeddedId
        private OrderItemPK id;

        private int quantity;

        @ManyToOne
        @JoinColumn(name = "order_id", insertable = false, updatable = false)
        private Order order;

        // ...

        public OrderItem(Integer orderId, String itemCode, int quantity) {
            this.id = new OrderItemPK(orderId, itemCode);
            this.quantity = quantity;
        }

        // ...

    }

    @Entity
    @Table(name = "t_order_pay")
    public class OrderPay implements Serializable {

        @Id
        @Column(name = "order_id")
        private Integer orderId;

        @Column(name = "way_to_pay")
        private String wayToPay;

        @OneToOne
        @JoinColumn(name = "order_id")
        private Order order;

        // ...

        public OrderPay(int orderId, String wayToPay) {
            this.orderId = orderId;
            this.wayToPay = wayToPay;
        }

        // ...

    }

|

関連Entityの追加
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
関連Entityを追加したい場合は、Repositoryインタフェース経由で取得したEntityオブジェクトに対して、生成した関連Entityのオブジェクトを関連付る。

この方法を使用するためには、関連EntityのCascade対象の操作に、``persist`` と ``merge`` が含まれている必要がある。

 .. code-block:: java

    String itemCode = "ITM0000003";
    int quantity = 30;

    Order order = orderRepository.findOne(orderId); // (1)

    OrderItem orderItem = new OrderItem(order.getId(), itemCode, quantity);
    order.getOrderItems().add(orderItem); // (2)

    OrderPay orderPay = order.getOrderPay();
    if (orderPay == null) {
        order.setOrderPay(new OrderPay(order.getId(), "cash")); // (3)
    } else {
        orderPay.setWayToPay("cash");
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | Repositoryインタフェースのメソッドを使用して、Entityオブジェクトを取得する。
    * - | (2)
      - | 1:Nの関連Entityの場合、生成した関連Entityのオブジェクトを、Entityオブジェクトから取得したコレクションに追加する。
    * - | (3)
      - | 1:1の関連Entityの場合、生成した関連EntityのオブジェクトをEntityオブジェクトに設定する。

|

関連Entityの直接追加
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
関連Entityを、親のEntityオブジェクトに関連付けせずに直接追加したい場合は、関連Entity用のRepositoryインタフェースを使用して保存する。

 .. code-block:: java

    String itemCode = "ITM0000003";
    int quantity = 40;

    OrderItem orderItem = new OrderItem(orderId, itemCode, quantity); // (1)

    orderItemRepository.save(orderItem); // (2)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | 関連Entityのオブジェクトを生成する。
    * - | (2)
      - | 関連Entity用のRepositoryインタフェースのsaveメソッドを呼び出す。

 .. note:: **関連Entityを直接保存するメリット**

     生成されるオブジェクトの数が少なくなる。 親のEntityオブジェクトを取得する場合、処理に不要な関連Entityのオブジェクトが生成される可能性がある。

 .. note:: **関連Entityを直接保存するデメリット**

    IDの一部に親EntityのIDを使用している場合、saveメソッドを呼び出す前にIDを設定しておく必要がある。
    IDを設定してしまうとSpring Data JPAのデフォルト実装は ``EntityManager`` のmergeメソッドを呼び出す。
    そのため、必ず永続層(DB)からIDが同じEntityを取得する処理が実行されてしまう。

 .. warning:: **追加後に親のEntityオブジェクトを利用する際の注意点**

    関連Entity用Repositoryのsaveメソッドを使って関連Entity追加した場合は、
    親にあたるEntityオブジェクトには関連付けられていないため、親のEntityオブジェクトを経由して取得することができない。

    回避方法は、

    #. 関連Entityのオブジェクトを直接追加するのではなく、親のEntityオブジェクトに関連付けて追加するようにする。
    #. saveAndFlushメソッドを使用することで、永続層(DB)と同期を親のEntityを取得する前に行う。

    このケースの場合、特に理由がないのであれば、前者の方法で関連Entityのオブジェクトを追加すること。
    後者は、親にあたるEntityオブジェクトが既に管理状態のEntityになっていた場合に、問題を回避することができない。

|

Entityの更新処理の実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Entityの更新方法について、目的別に実装例を説明する。

|

Entityの更新
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Entityを更新したい場合は、Repositoryインタフェースのメソッドを使用して取得したEntityオブジェクトに対して、変更したい値を設定する。

 .. code-block:: java

    Order order = orderRepository.findOne(orderId); // (1)
    order.setStatus(new OrderStatus("checking")); // (2)


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | Repositoryインタフェースのメソッドを使用して、Entityオブジェクトを取得する。
    * - | (2)
      - | Entityオブジェクトの状態を、setterメソッドを呼び出して更新する。

 .. note:: **Repositoryのsaveメソッドの呼び出しについて**

    Repositoryインタフェースのメソッドを使用して取得したEntityオブジェクトは、 ``EntityManager`` 上で管理されている。
    ``EntityManager`` 上で管理されているEntityオブジェクトは、setterメソッドを使ってオブジェクトの状態を変更するだけで、
    トランザクションコミット時に変更内容が永続層(DB)に反映される仕組みになっている。
    そのため、Repositoryインタフェースのsaveメソッドを明示的に呼び出す必要はない。

    ただし、Entityオブジェクトが ``EntityManager`` 上で管理されていない場合は、 saveメソッドを呼び出す必要がある。
    例えば、画面から送信されたリクエストパラメータをもとにEntityオブジェクトを生成した場合などが、このケースに当てはまる。

|

関連Entityの更新
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
関連Entityを更新したい場合は、Repositoryインタフェースのメソッドを使用して取得したEntityオブジェクトから取得した関連Entityのオブジェクトに対して、変更したい値を設定する。

この方法を使用するためには、関連EntityのCascade対象の操作に、``merge`` が含まれている必要がある。

 .. code-block:: java

    Order order = orderRepository.findOne(orderId); // (1)

    for (OrderItem orderItem : order.getOrderItems()) {
        int newQuantity = quantityMap.get(orderItem.getId().getItemCode());
        orderItem.setQuantity(newQuantity); // (2)
    }

    order.getOrderPay().setWayToPay("cash"); // (3)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | Repositoryインタフェースのメソッドを使用して、Entityオブジェクトを取得する。
    * - | (2)
      - | 1:Nの関連Entityの場合、Entityオブジェクトから取得したコレクションに格納されている関連Entityのオブジェクトの状態を、setterメソッドを呼び出して更新する。
    * - | (3)
      - | 1:1の関連Entityの場合、Entityオブジェクトから取得した関連Entityのオブジェクトの状態を、setterメソッドを呼び出して更新する。

|


関連Entityの直接更新
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
関連Entityを、親のEntityを使用せず直接更新したい場合は、関連Entity用のRepositoryインタフェースのメソッドを使用して取得したEntityオブジェクトに対して、変更したい値を設定する。

 .. code-block:: java

    int quantity = 43;

    OrderItem orderItem = orderItemRepository.findOne(new OrderItemPK(
            orderId, itemCode)); // (1)

    orderItem.setQuantity(quantity); // (2)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | 関連EntityのRepositoryインタフェースのfindOneメソッドを呼び出して、関連Entityのオブジェクトを取得する。
    * - | (2)
      - | 関連Entityのオブジェクトの状態を、setterメソッドを使って更新する。

 .. note:: **更新後に親のEntityを利用した場合の動作について**

    関連Entity用Repositoryのsaveメソッドを使って関連Entityを更新した場合は、
    追加時とは異なり、親のEntityオブジェクトで保持している関連Entityにも反映される
    これは、``EntityManager`` 上で管理されている同じインスタンスの参照を保持しているためである。

|

Queryメソッドを使用して更新
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 永続層(DB)のEntityを直接更新したい場合は、Queryメソッドを使用して更新する。
| 詳細は、「:ref:`data-access-jpa_howtouse_querymethod_modifying`」を参照されたい。

|

Entityの削除処理の実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Entityと関連Entityの削除
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Entityと関連Entityを一緒に削除したい場合は、Repositryインタフェースのdeleteメソッドを呼び出す。

この方法を使用するためには、関連EntityのCascade対象の操作に ``remove`` が含まれているか、
または関連Entityを削除するための設定を有効(関連付けアノテーションのorphanRemoval属性を ``true`` )にする必要がある。

- Service

 .. code-block:: java

    orderRepository.delete(orderId); // (1)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | IDまたはEntityオブジェクトを指定してRepositryインタフェースのdeleteメソッドを呼び出す。

- Entity

 .. code-block:: java

    @Entity
    @Table(name = "t_order")
    public class Order implements Serializable {

        // ...

        @OneToMany(mappedBy = "order",
                   cascade = CascadeType.ALL, orphanRemoval = true) // (2)
        @OrderBy
        private Set<OrderItem> orderItems;

        // ...

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (2)
      - | Cascade対象の操作に ``remove`` を含め、 関連Entityを削除するための設定を有効 (関連付けアノテーションのorphanRemoval属性を ``true`` )にする。

 .. note:: **関連Entityの削除について**

    関連Entityのオブジェクトを一緒に削除したくない場合は、 Cascade対象の操作に ``remove`` が含まれないようにしておく必要がある。
    また、関連付けアノテーションのorphanRemoval属性も ``false`` に指定しておくこと。

|

.. _daba-access-jpa_howtouse_remove_relationship:

関連Entityの削除
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
関連Entityを削除したい場合は、Repositoryインタフェース経由で取得したEntityオブジェクトから、関連Entityのオブジェクトを削除する。

この方法を使用するためには、関連Entityを削除するための設定を有効(関連付けアノテーションのorphanRemoval属性を ``true`` )にする必要がある。

 .. note:: **関連Entityを削除する設定を有効にした場合の挙動について**

    関連Entityを削除する設定を有効にした場合、以下の動作になる。

    * 1:Nの関連Entityの場合は、関連Entityのオブジェクトをコレクションの中から削除すると、トランザクションコミット時に永続層(DB)からも削除される。
    * 1:1の関連Entityの場合は、関連Entityを ``null`` に設定すると、トランザクションコミット時に永続層(DB)からも削除される。

    orphanRemoval属性のデフォルト値となっている ``false`` が指定されている場合は、メモリ上からは消えるが、
    削除した関連Entityのオブジェクトに対する永続処理(UPDATE/DELETE)は行われない。


- Service

 .. code-block:: java

    Order order = orderRepository.findOne(orderId); // (1)

    // (2)
    Set<OrderItem> orderItemsOfRemoveTarget = new LinkedHashSet<OrderItem>(); // (3)
    for (OrderItem orderItem : order.getOrderItems()) {
        String itemCode = orderItem.getId().getItemCode();
        if (quantityMap.containsKey(itemCode)) {
            int newQuantity = quantityMap.get(itemCode);
            orderItem.setQuantity(newQuantity);
        } else {
            orderItemsOfRemoveTarget.add(orderItem); // (4)
        }
    }
    order.getOrderItems().removeAll(orderItemsOfRemoveTarget); // (5)

    order.setOrderPay(null); // (6)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | Repositoryインタフェースのメソッドを使用して、Entityオブジェクトを取得する。
    * - | (2)
      - | 1:Nの関連Entityを削除する実装例について説明する。
    * - | (3)
      - | 削除する関連Entityのオブジェクトを格納しておくコレクションを用意する。
    * - | (4)
      - | 削除対象の関連Entityのオブジェクトを、(3)のコレクションに追加する。
    * - | (5)
      - | 削除対象の関連Entityのオブジェクトから取得したコレクションの中から削除する。
    * - | (6)
      - | 1:1の関連Entityの場合、削除したい関連Entityのオブジェクトを保持するプロパティを ``null`` に設定する。

- Entity

 .. code-block:: java

    @Entity
    @Table(name = "t_order")
    public class Order implements Serializable {

        @Id
        @SequenceGenerator(name = "GEN_ORDER_ID", sequenceName = "s_order_id", allocationSize = 1)
        @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "GEN_ORDER_ID")
        private int id;

        @OneToMany(mappedBy = "order", cascade = CascadeType.ALL,
                   orphanRemoval = true) // (7)
        @OrderBy
        private Set<OrderItem> orderItems;

        @OneToOne(mappedBy = "order", cascade = CascadeType.ALL,
                  orphanRemoval = true) // (7)
        private OrderPay orderPay;

        @ManyToOne
        @JoinColumn(name = "status_code")
        private OrderStatus status;

        // ...

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (7)
      - | 関連Entityを削除するための設定を有効(関連付けアノテーションのorphanRemoval属性を ``true`` )にする。

 .. note :: **orphanRemoval属性が指定できる関連付けアノテーションについて**

    orphanRemoval属性が指定できる関連付けアノテーションは、 ``@OneToOne`` と ``@OneToMany`` の2つとなっている。

|

関連Entityの直接削除
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
関連Entityを、親のEntityを使用せず直接削除したい場合は、関連EntityのRepositoryインタフェースのdeleteメソッドを呼び出す。

 .. code-block:: java

    int quantity = 43;

    orderItemRepository.delete(new OrderItemPK(orderId, itemCode)); // (1)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | IDまたはEntityオブジェクトを指定して関連Entity用Repositryインタフェースのdeleteメソッドを呼び出す。

 .. warning:: **直接削除の注意点**

    関連Entity用Repositoryのdeleteメソッドを呼び出した場合、永続層(DB)から削除されないケースがあるので注意すること。

    削除されないケースは以下の通り。

    * deleteメソッド内の中でfindOneメソッドを呼び出しているため、親のEntityとの関連が ``@OneToOne`` の場合は、親のEntityが ``EntityManager`` 上で管理されてしまう。
      親のEntityが管理対象になった際に、deleteメソッドで削除しようとしていた関連Entityが親のEntityオブジェクト内にロードされる可能性がある。
      親のEntityオブジェクト内にロードされてしまうと、永続層(DB)から削除されない。
    * deleteメソッド呼び出し後に、親のEntityオブジェクトを取得した場合、deleteメソッドで削除した関連Entityが親のEntityオブジェクト内にロードされる可能性がある。
      親のEntityオブジェクト内にロードされてしまうと、永続層(DB)から削除されない。

    回避方法は、

    #. 関連Entityのオブジェクトを直接削除するのではなく、親のEntityとの関連付けを削除するようにする。

    関連付けアノテーションの見直しを行うことで回避出来る可能性はあるが、確実な回避方法は、直接削除を行わないようにする方法である。

|

Queryメソッドを使用して削除
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 永続層(DB)からEntityを直接削除したい場合は、Queryメソッドを使用して削除する。
| 詳細は、「:ref:`data-access-jpa_howtouse_querymethod_modifying`」を参照されたい。

 .. warning:: **関連オブジェクトの扱いについて**

    Queryメソッドを使用して永続層(DB)から直接Entityを削除した場合、関連付けアノテーションの指定に関係なく、
    関連Entityのオブジェクトは永続層から削除されない。

    Repositoryインタフェースの ``void deleteInBatch(Iterable<T> entities)`` と ``void deleteAllInBatch()`` も同様の動作となる。

|

.. _data-access-jpa_howtouse_like_escape:

LIKE検索時のエスケープについて
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| LIKE検索を行う場合は、検索条件として使用する値をLIKE検索用にエスケープする必要がある。
| LIKE検索用のエスケープ処理は、共通ライブラリから提供している ``org.terasoluna.gfw.common.query.QueryEscapeUtils`` クラスのメソッドを使用する事で実現することができる。
| 共通ライブラリから提供しているエスケープ処理の仕様については、「:doc:`DataAccessCommon`」の「:ref:`data-access-common_appendix_like_escape`」を参照されたい。

以下に、共通ライブラリから提供しているエスケープ処理の使用方法について説明する。

|

一致方法をQuery側で指定する場合の使用方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
一致方法(前方一致、後方一致、部分一致)の指定をJPQLとして指定する場合は、エスケープのみ行うメソッドを使用する。

- Repository

 .. code-block:: java

    // (1) (2)
    @Query("SELECT a FROM Article a WHERE"
            + " (a.title LIKE %:word% ESCAPE '~' OR a.overview LIKE %:word% ESCAPE '~')")
    Page<Article> findPageBy(@Param("word") String word, Pageable pageable);

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | ``@Query`` アノテーションに指定するJPQL内にLIKE検索用のワイルドカード( ``"%"`` または ``"_"`` )を指定する。
        | 上記例では、引数 ``word`` の前後にワイルドカード( ``"%"`` )を指定することで、一致方法を部分一致にしている。
    * - | (2)
      - | 共通ライブラリから提供しているエスケープ処理は、エスケープ文字として ``"~"`` を使用しているため、 LIKE句の後ろに ``"ESCAPE '~'"`` を指定する。

 .. note :: **ワイルドカード文字 "_" について**

    ワイルドカード文字 ``"%"`` は、「:ref:`how_to_specify_query_annotation-label`」で説明しているように、 ``@Query`` アノテーションを利用した場合に限り直接JPQL内で使用することができる。
    ワイルドカード文字 ``"_"`` は直接JPQL内のLIKE検索に利用することはできないため、下記2方法で利用すること。
    
    #. ワイルドカード文字 ``"_"`` をバインド変数内に含める。
    #. ワイルドカード文字 ``"_"`` を ``CONCAT`` などのデータベース製品が持つ文字列結合機能を利用してバインド変数に結合する。

|

- Service

 .. code-block:: java

    @Transactional(readOnly = true)
    public Page<Article> searchArticle(ArticleSearchCriteria criteria,
            Pageable pageable) {

        String escapedWord = QueryEscapeUtils.toLikeCondition(criteria.getWord()); // (3)

        Page<Article> page = articleRepository.findPageBy(escapedWord, pageable); // (4)

        return page;
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (3)
      - | LIKE検索の一致方法をQuery側で指定する場合は、``QueryEscapeUtils#toLikeCondition(String)`` メソッドを呼び出し、LIKE検索用のエスケープのみ行う。
    * - | (4)
      - | LIKE検索用にエスケープされた値をQueryメソッドの引数に渡す。

|

一致方法をロジック側で指定する場合の使用方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
一致方法(前方一致、後方一致、部分一致)をロジック側で判定する場合は、エスケープされた値にワイルドカードを付与するメソッドを使用する。

- Repository

 .. code-block:: java

    // (1)
    @Query("SELECT a FROM Article a WHERE"
            + " (a.title LIKE :word ESCAPE '~' OR a.overview LIKE :word ESCAPE '~')")
    Page<Article> findPageBy(@Param("word") String word, Pageable pageable);

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | ``@Query`` アノテーションに指定するJPQL内にLIKE検索用のワイルドカードは指定しない。

|

- Service

 .. code-block:: java

    @Transactional(readOnly = true)
    public Page<Article> searchArticle(ArticleSearchCriteria criteria,
            Pageable pageable) {

        String word = QueryEscapeUtils
                .toContainingCondition(criteria.getWord()); // (2)

        Page<Article> page = articleRepository.findPageBy(word, pageable); // (3)

        return page;
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (2)
      - | ロジック側で一致方法を指定する場合は、  以下の何れかのメソッドを呼び出し、LIKE検索用のエスケープとLIKE検索用のワイルドカードを付与する。
        |   ``QueryEscapeUtils#toStartingWithCondition(String)``
        |   ``QueryEscapeUtils#toEndingWithCondition(String)``
        |   ``QueryEscapeUtils#toContainingCondition(String)``
    * - | (3)
      - | LIKE検索用にエスケープ＋ワイルドカードが付与された値をQueryメソッドの引数に渡す。

|

.. _data-access-jpa_howtouse_join_fetch:

JOIN FETCHについて
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| JOIN FETCHは、関連するEntityをJOINして一括で取得することで、Entityを取得するために発行するQueryの数を減らすための仕組みである。

| 任意のQueryを実行してEntityを取得する場合、関連付けアノテーションのfetch属性が ``EAGER`` となっているプロパティは必ず JOIN FETCH すること。
| JOIN FETCHしないと、関連Entityを取得するためのQueryが別途発行されるため、性能に影響を与える可能性がある。

| 関連付けアノテーションのfetch属性が ``LAZY`` となっているプロパティについては、使用頻度を考慮して JOIN FETCH するか判断すること。
| 必ず参照される関連Entityの場合は JOIN FETCHすべきであるが、参照されるケースが少ないのであれば JOIN FETCH せず、初回アクセス時にQueryを発行して取得する方がよい。

- Repository

 .. code-block:: java

    @Query("SELECT a FROM Article a"
            + " INNER JOIN FETCH a.articleClass"   // (1)
            + " WHERE a.publishedDate = :publishedDate"
            + " ORDER BY a.articleId DESC")
    List<Article> findAllByPublishedDate(
            @Param("publishedDate") Date publishedDate);

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | ``"[LEFT [OUTER] | INNER] JOIN FETCH Join対象のプロパティ"`` の形式で指定する。
        | 具体的には以下のパターンとなる。
        |
        | 1. ``LEFT JOIN FETCH``
        |    関連Entityが存在しない場合でもEntityは取得される。
        |    SQLとしては、 ``"left outer join RelatedTable r on m.fkColumn = r.fkColumn"`` となる。
        | 2. ``LEFT OUTER JOIN FETCH``
        |    1と同様。
        | 3. ``INNER JOIN FETCH``
        |    関連Entityが存在しない場合は、Entityは取得されない。
        |    SQLとしては、 ``"inner join RelatedTable r on m.fkColumn = r.fkColumn"`` となる。
        | 4. ``JOIN FETCH``
        |    3と同様。
        |
        | 上記例では、ArticleクラスのarticleClassプロパティをJOIN FETCH対象として指定している。

 .. note::

    JOIN FETCHはN+1問題の解決策の一つとして使用される。
    N+1問題については、「:ref:`data-access-common_howtosolve_n_plus_1`」を参照されたい。


|

How to extend
--------------------------------------------------------------------------------

.. _data-access-jpa_how_to_extends_custommethod:

カスタムメソッドの追加方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Spring Dataでは、Repositoryインタフェースに対して、任意のカスタムメソッドを追加する仕組みを提供している。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table::
    :widths: 10 35 55
    :header-rows: 1

    * - 項番
      - 追加方法
      - 使用ケース
    * - 1.
      - :ref:`custommethod_individual-label`
      - | Spring Dataより提供されているQueryメソッドの仕組みで表現できないQueryを実装する必要がある場合に使用する。
        | 動的Queryを実装する場合は、この方法を使用してメソッドを追加する。
    * - 2.
      - :ref:`custommethod_all-label`
      - | すべてのRepositoryインタフェースで使用できるような汎用的なQueryを実装する必要がある場合に使用する。
        | プロジェクト固有の汎用Queryを実装する場合は、この方法を使用してメソッドを追加する。
        | Spring Data JPAから提供されているデフォルト実装( ``SimpleJpaRepository`` )の振る舞いを変更する必要がある場合、この仕組みを使用してメソッドをオーバーライドすることで対応することが出来る。

|

.. _custommethod_individual-label:

Entity毎のRepositoryインタフェースに個別に追加する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Entity毎のRepositoryインタフェースに個別にカスタムメソッドを追加する方法について説明する。

- Entity毎のカスタムRepositoryインタフェース

 .. code-block:: java

    // (1)
    public interface OrderRepositoryCustom {

        // (2)
        Page<Order> findByCriteria(OrderCriteria criteria, Pageable pageable);

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | カスタムメソッドを定義するカスタムインタフェースを作成する。
        | インタフェース名に特に制約はないが、Entity毎のRepositoryインタフェース名 + ``"Custom"`` とすることを推奨する。
    * - | (2)
      - | カスタムメソッドを定義する。

- Entity毎のカスタムRepositoryクラス

 .. code-block:: java

    // (3)
    public class OrderRepositoryImpl implements OrderRepositoryCustom {

        @PersistenceContext
        EntityManager entityManager; // (4)

        // (5)
        public Page<Order> findByCriteria(OrderCriteria criteria, Pageable pageable) {
            // ...
            return new PageImpl<Order>(orders, pageable, totalCount);
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (3)
      - | カスタムインタフェースの実装クラスを作成する。
        | クラス名は、Entity毎のRepositoryインタフェース名 + ``"Impl"`` とすること。
    * - | (4)
      - | Queryを実行するために必要となる ``javax.persistence.EntityManager`` は ``@javax.persistence.PersistenceContext`` アノテーションを使ってインジェクションする。
    * - | (5)
      - | カスタムインタフェースに定義したメソッドを実装する。

- Entity毎のRepositoryインタフェース

 .. code-block:: java

    public interface OrderRepository extends JpaRepository<Order, Integer>,
            OrderRepositoryCustom { // (6)
        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (6)
      - | Entity毎のRepositoryインタフェースでカスタムインタフェースを継承する。
        | Repositoryインタフェースを継承することで、実行時にカスタムインタフェースの実装クラスのメソッドが呼び出される。

- Service(Caller)

 .. code-block:: java

    public Page<Order> search(OrderCriteria criteria, Pageable pageable) {
        return orderRepository.findByCriteria(criteria, pageable); // (7)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (7)
      - | 呼び出し側は、他のメソッドと同様にEntity毎のRepositoryインタフェースのメソッドを呼び出せばよい。
        | 上記例では、``OrderRepository#findByCriteria(OrderCriteria, Pageable)`` を呼び出すと ``OrderRepositoryImpl#findByCriteria(OrderCriteria, Pageable)`` が実行される。

|

.. _custommethod_all-label:

すべてのRepositoryインタフェースに一括で追加する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
すべてのRepositoryインタフェースに一括でカスタムメソッドを追加する方法について説明する。

下記の実装例では、取得したEntityとのバージョンチェックを行い、バージョンが異なる場合に楽観排他エラーとするメソッドを追加している。

- 共通のRepositoryインタフェース

 .. code-block:: java

    // (1)
    @NoRepositoryBean // (2)
    public interface MyProjectRepository<T, ID extends Serializable> extends
            JpaRepository<T, ID> {

        // (3)
        T findOneWithValidVersion(ID id, Integer version);

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | 追加するメソッドを定義するための共通Repositoryインタフェースを作成する。
        | 上記例では、Spring Dataから提供されている ``JpaRepository`` を継承して共通Repositoryインタフェースを作成している。
    * - | (2)
      - | 共通Repositoryインタフェースの実装クラス( 例だと ``MyProjectRepositoryImpl`` )が、RepositoryのBeanとして自動登録されないようにするためのアノテーション。
        | 共通Repositoryインタフェースの実装クラスを<jpa:repositories>要素のbase-package属性で指定したパッケージ配下に格納する場合は、このアノテーションを指定しないと起動時にエラーとなる。
    * - | (3)
      - | 追加するメソッドを定義する。例では、取得したEntityのバージョンチェックを行うメソッドを追加している。

- 共通Repositoryインタフェースの実装クラス

 .. code-block:: java

    // (6)
    public class MyProjectRepositoryImpl<T, ID extends Serializable>
            extends SimpleJpaRepository<T, ID>
            implements MyProjectRepository<T, ID> {

        private JpaEntityInformation<T, ID> entityInformation;
        private EntityManager entityManager;
        Method versionMethod;

        // (7)
        public MyProjectRepositoryImpl(
                JpaEntityInformation<T, ID> entityInformation,
                EntityManager entityManager) {
            super(entityInformation, entityManager);

            this.entityInformation = entityInformation; // (8)
            this.entityManager = entityManager; // (8)
            
            try {
                versionMethod = entityInformation.getJavaType().getMethod("getVersion");
            } catch (NoSuchMethodException | SecurityException e) { }

        }

        // (9)
        public T findOneWithValidVersion(ID id, Integer version) {

            if (versionMethod == null) {
                throw new UnsupportedOperationException(
                        String.format(
                                "Does not found version field in entity class. class is '%s'.",
                                entityInformation.getJavaType().getName()));
            }

            T entity = findOne(id);

            if (entity != null && version != null) {
                Integer currentVersion;
                try {
                    currentVersion = (Integer) versionMethod.invoke(entity);
                } catch (IllegalAccessException | IllegalArgumentException
                        | InvocationTargetException e) {
                    throw new IllegalStateException(e);
                }
                if (!version.equals(currentVersion)) {
                    throw new ObjectOptimisticLockingFailureException(
                            entityInformation.getJavaType().getName(), id);
                }
            }

            return entity;
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (6)
      - | 共通Repositoryインタフェース(例だと、 ``MyProjectRepository`` )の実装クラスを作成する。
        | 上記例では、Spring Dataから提供されている ``SimpleJpaRepository`` を継承して実装クラスを作成している。
    * - | (7)
      - | ``org.springframework.data.jpa.repository.support.JpaEntityInformation`` と ``javax.persistence.EntityManager`` を引数にとるコンストラクタを作成する。
    * - | (8)
      - | 追加するメソッドの処理で必要な場合は、 ``JpaEntityInformation`` および ``EntityManager`` をフィールドに保持しておく。
    * - | (9)
      - | 共通Repositoryインタフェースに定義したメソッドの実装を行う。

 .. note:: **デフォルト実装の変更**

    デフォルト実装( ``SimpleJpaRepository`` )の振る舞いを変更する場合は、このクラスで振る舞いを変更したいメソッドをオーバーライドすればよい。

- Entity毎のRepositoryインタフェース

 .. code-block:: java

    public interface OrderRepository extends MyProjectRepository<Order, Integer> { // (10)
        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (10)
      - | Entity毎のRepositoryインタフェースでは、作成した共通インタフェース(例だと ``MyProjectRepository`` )を継承先として指定する。

- ``xxx-infra.xml``

 .. code-block:: xml

    <jpa:repositories base-package="x.y.z.domain.repository"
        base-class="x.y.z.domain.repository.MyProjectRepositoryImpl" /> <!-- (11) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (11)
      - | <jpa:repositories>要素のbase-class属性に、作成した共通Repositoryインタフェースの実装クラス(例だと ``MyProjectRepositoryImpl``) のクラス名を指定する。

- Service(Caller)

 .. code-block:: java

    public Order updateOrder(Order chngedOrder, Integer version) {

        Order order = orderRepository.findOneWithValidVersion(chngedOrder.getId(), version); // (12)

        // ....

        return order;
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (12)
      - | 呼び出し側は、他のメソッドと同様にEntity毎のRepositoryインタフェースのメソッドを呼び出せばよい。
        | 上記例では、``OrderRepository#findOneWithValidVersion(Integer, Integer)`` を呼び出すと ``MyProjectRepositoryImpl#findOneWithValidVersion(Integer, Integer)`` が実行される。

|


Entity以外のオブジェクトにQueryの取得結果を格納する方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Queryの取得結果をEntityにマッピングするのではなく、別のオブジェクトにマッピングすることができる。
これは、永続層(DB)に格納されているレコードをEntity以外のオブジェクト(JavaBean)として扱いたい時に使用する。

 .. note:: **想定される適用ケース**

    #. Query内で集計関数を使用して集計済みの情報を取得したい場合は、集計結果をEntityにマッピングすることはできたいため、別のオブジェクトにマッピングする必要がある。

    #. 巨大なEntityの中の一部の情報のみ参照したい場合や、ネストが深い関連Entityの一部の情報のみ参照したい場合は、必要なプロパティのみ定義したJavaBeanにマッピングして取得する事を検討した方がよい場合がある。
       Entityとして取得した場合、アプリケーションの処理に不要な項目に対するマッピング処理が行われる点や、処理に不要な情報を取得することでメモリを無駄に使用する点などにより、処理性能に影響を与える場合があるためである。
       処理性能に大きな影響を与えない場合は、Entityとして取得してよい。

|

以下の実装例を示す。

- JavaBean

 .. code-block:: java

    // (1)
    public class OrderSummary implements Serializable {

        private Integer id;
        private Long totalPrice;

        // ...

        public OrderSummary(Integer id, Long totalPrice) { // (2)
            super();
            this.id = id;
            this.totalPrice = totalPrice;
        }

        // ...

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | Query結果を格納するJavaBeanを作成する。
    * - | (2)
      - | Queryを実行した結果でオブジェクト生成するためのコンストラクタを用意する。

- Repositoryインタフェース

 .. code-block:: java

    // (3)
    @Query("SELECT NEW x.y.z.domain.model.OrderSummary(o.id, SUM(i.price*oi.quantity))"
            + " FROM Order o LEFT JOIN o.orderItems oi LEFT JOIN oi.item i"
            + " GROUP BY o.id ORDER BY o.id DESC")
    List<OrderSummary> findOrderSummaries();

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (3)
      - | (2)で作成したコンストラクタを指定する。
        | "NEW"キーワードの後に FQCNでコンストラクタを指定する。


|

Audit用プロパティの設定方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Spring Data JPAより提供されている、永続層のAudit用プロパティ(作成者、作成日時、最終更新者、最終更新日時)に値を設定する仕組みと適用方法について説明する。

Spring Data JPAでは、新たに作成されたEntityと更新されたEntityに対して、Audit用プロパティに値を設定する仕組みを提供している。
この仕組みを使用すると、ServiceなどのアプリケーションのコードからAudit用プロパティに値を設定する処理を切り離すことができる。

 .. note:: **アプリケーションのコードからAudit用プロパティに値を設定する処理を切り離す理由**

    #. Audit用プロパティへの値の設定は、一般的にアプリケーション要件ではなく、データの監査要件によって必要になる処理である。
       本質的にはServiceなどのアプリケーション内のコードで意識すべき処理ではないため、
       アプリケーションのコードから切り離した方がよい。

    #. JPAでは、永続層から取得したEntityに対して値の変更があった場合のみ、永続層へ反映(SQLを発行)する仕様になっており、
       永続層へ無駄なアクセスが行われないように制御されている。
       ServiceなどのアプリケーションのコードでAudit用プロパティに無条件に値を設定してしまうと、
       Audit用プロパティ以外に変更がない場合でも、永続層への反映が行われることになるため、JPAの有効な機能を無駄にしてしまう。
       値の変更があった場合のみAudit用プロパティに値を設定するように制御すればこの問題は回避できるが、
       アプリケーションのコードが煩雑になるため推奨できない。

 .. note:: **値の変更有無に関係なく永続層のAudit用カラムを更新する必要がある場合**

    値に変更がなくても永続層のAudit用カラム(最終更新者、最終更新日時)を更新するのが、アプリケーションの仕様となっている場合は、
    ServiceなどのアプリケーションのコードでAudit用プロパティを設定する必要がある。

    ただし、このケースではデータモデリングまたはアプリケーション仕様が間違っている可能性があるため、データモデリングおよびアプリケーション仕様の見直しを行った方がよい。

|

以下に、適用時の実装例を示す。

- Entityクラス

 .. code-block:: java

    public class XxxEntity implements Serializable {
        private static final long serialVersionUID = 1L;

        // ...

        @Column(name = "created_by")
        @CreatedBy // (1)
        private String createdBy;

        @Column(name = "created_date")
        @CreatedDate // (2)
        @Type(type = "org.jadira.usertype.dateandtime.joda.PersistentDateTime") // (3)
        private DateTime createdDate; // (4)

        @Column(name = "last_modified_by")
        @LastModifiedBy // (5)
        private String lastModifiedBy;

        @Column(name = "last_modified_date")
        @LastModifiedDate // (6)
        @Type(type = "org.jadira.usertype.dateandtime.joda.PersistentDateTime") // (3)
        private DateTime lastModifiedDate; // (4)

        // ...

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | 作成者を保持するフィールドに ``@org.springframework.data.annotation.CreatedBy`` アノテーションを付与する。
    * - | (2)
      - | 作成日時を保持するフィールドに ``@org.springframework.data.annotation.CreatedDate`` アノテーションを付与する。
    * - | (3)
      - | ``org.joda.time.DateTime`` 型を使用する場合は、Hibernateで扱えるようにするために、 フィールドに ``@org.hibernate.annotations.Type`` アノテーションを付与する。
        | type属性は、 ``"org.jadira.usertype.dateandtime.joda.PersistentDateTime"`` 固定。最終更新日時のフィールドも同様。
    * - | (4)
      - | 作成日時を保持するフィールドの型は、 ``org.joda.time.DateTime`` 、``java.util.Date`` 、``java.util.Calendar`` 、 ``java.lang.Long`` 、 ``long`` 型 、Java 8から追加されたDate and Time APIなどをサポートしている。
        | 最終更新日時のフィールドも同様。
    * - | (5)
      - | 最終更新者を保持するフィールドの型に ``@org.springframework.data.annotation.LastModifiedBy`` アノテーションを付与する。
    * - | (6)
      - | 最終更新日時を保持するフィールドに ``@org.springframework.data.annotation.LastModifiedDate`` アノテーションを付与する。

 .. warning::

    ``@Type`` アノテーションは、JPAの標準アノテーションではなく、Hibernate独自のアノテーションである。


- AuditorAwareインタフェースの実装クラス

 .. code-block:: java

    // (7)
    @Component // (8)
    public class SpringSecurityAuditorAware implements AuditorAware<String> {

        // (9)
        public String getCurrentAuditor() {
            Authentication authentication = SecurityContextHolder.getContext()
                    .getAuthentication();
            if (authentication == null || !authentication.isAuthenticated()) {
                return null;
            }
            return ((UserDetails) authentication.getPrincipal()).getUsername();
        }

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (7)
      - | ``org.springframework.data.domain.AuditorAware`` インタフェースの実装クラスを作成する。
        | ``AuditorAware`` インタフェースは、Entityの操作者(作成者または最終更新更新者)を解決するためのインタフェースとなっている。
        | このクラスはプロジェクト毎に作成する必要がある。
    * - | (8)
      - | ``@Component`` アノテーションを付与することで、component-scan対象になるようにしている。
        | ``@Component`` アノテーションを付けずに、Bean定義ファイルにbean定義してもよい。
    * - | (9)
      - | Entityの操作者(作成者または最終更新者)のプロパティに設定するオブジェクトを返却する。
        | 上記例では、SpringSecurityによって認証されたログインユーザのユーザ名(文字列)をEntityの操作者として返却している。

- Object/relational mapping file( ``orm.xml`` )

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>

    <entity-mappings xmlns="http://java.sun.com/xml/ns/persistence/orm"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/persistence/orm
        http://java.sun.com/xml/ns/persistence/orm_2_0.xsd"
        version="2.0">

        <persistence-unit-metadata>
            <persistence-unit-defaults>
                <entity-listeners>
                    <entity-listener
                        class="org.springframework.data.jpa.domain.support.AuditingEntityListener" /> <!-- (10) -->
                </entity-listeners>
            </persistence-unit-defaults>
        </persistence-unit-metadata>

    </entity-mappings>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (10)
      - | Entity Listenerとして、 ``org.springframework.data.jpa.domain.support.AuditingEntityListener`` クラスを指定する。
        | このクラスで実装しているメソッドにて、Audit用プロパティに値を設定している。

- ``infra.xml``

 .. code-block:: xml

    <jpa:auditing auditor-aware-ref="springSecurityAuditorAware" /> <!-- (11) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (11)
      - | <jpa:auditing>要素のauditor-aware-ref属性に、(7)で作成したEntityの操作者を解決するためのクラスのbeanを指定する。
        | 上記例では、 ``SpringSecurityAuditorAware`` という実装クラスをcomponent-scanしているので、 ``"springSecurityAuditorAware"`` というbean名を指定している。

|

``@CreatedDate`` および ``@LastModifiedDate`` アノテーションが付与されたフィールドに設定される値は、
デフォルト実装だと ``org.springframework.data.auditing.CurrentDateTimeProvider`` の ``getNow()`` メソッド
から返却される ``java.util.Calendar`` のインスタンスの値が使用される。

以下に、使用する値の生成方法を変更する場合の拡張例を示す。

 .. code-block:: java

    // (1)
    @Component // (2)
    public class AuditDateTimeProvider implements DateTimeProvider {

        @Inject
        JodaTimeDateFactory dateFactory;

        // (3)
        @Override
        public Calendar getNow() {
            DateTime currentDateTime = dateFactory.newDateTime();
            return currentDateTime.toGregorianCalendar();
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | ``org.springframework.data.auditing.DateTimeProvider`` インタフェースの実装クラスを作成する。
    * - | (2)
      - | ``@Component`` アノテーションを付与することで、component-scan対象になるようにしている。
        | ``@Component`` アノテーションを付けずに、Bean定義ファイルにbean定義してもよい。
    * - | (3)
      - | Entityの操作日時(作成日時、最終更新日時)のプロパティに設定するインスタンスを返却する。
        | 上記例では、共通ライブラリから提供している ``org.terasoluna.gfw.common.date.jodatime.JodaTimeDateFactory`` から取得したインスタンスを操作日時として返却している。
        | ``JodaTimeDateFactory`` の詳細については、「:doc:`../GeneralFuncDetail/SystemDate`」を参照されたい。

- ``infra.xml``

 .. code-block:: xml

    <jpa:auditing
        auditor-aware-ref="springSecurityAuditorAware"
        date-time-provider-ref="auditDateTimeProvider" /> <!-- (4) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (4)
      - | <jpa:auditing>要素のdate-time-provider-ref属性に、(1)で作成したEntityの操作日時に設定する値を返却するクラスのbeanを指定する。
        | 上記例では、 ``AuditDateTimeProvider`` という実装クラスをcomponent-scanしているので、 ``"auditDateTimeProvider"`` というbean名を指定している。

|


永続層からEntityを取得するJPQLに共通条件を加える方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
永続層(DB)からEntityを取得するためのJPQLに共通条件を加える仕組みを紹介する。
これはJPAの標準仕様ではなく、Hibernateが拡張した仕組み機能である。

 .. note:: **想定される適用ケース**

    データの監査やデータの保存期間などの要件により、Entityを削除する際に、レコードの物理削除(DELETE)ではなく、論理削除(論理削除フラグのUPDATE)を行うようなアプリケーションが有効である。
    このケースでは、アプリケーションとしては論理削除されたレコードは一律検索対象から除外する必要があるため、Queryひとつひとつに除外するための条件を加えるより、共通条件として指定した方がよい。

 .. warning:: **適用範囲について**

    この仕組みを適用したEntityを操作するすべてのQueryに対して、指定した条件が追加される点に注意すること。条件を加えたくないQueryが1つでもある場合は、この仕組みは使用できない。

|

Entityを取得するJPQLに共通条件を加える
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Repositoryインタフェースのメソッド呼び出し時に実行されるJPQLに対して、共通の条件を加える方法を以下に示す。

- Entity

 .. code-block:: java

    @Entity
    @Table(name = "t_order")
    @Where(clause = "is_logical_delete = false") // (1)
    public class Order implements Serializable {
        // ...
        @Id
        private Integer id;
        // ...
    }

- RepositoryインタフェースのfindOneメソッドを呼び出した時に発行されるSQL

 .. code-block:: sql

    SELECT
            -- ....
        FROM
            t_order order0_
       WHERE
            order0_.id = 1
            AND (
                order0_.is_logical_delete = false -- (2)
            );


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | Entityのクラスアノテーションとして、 ``@org.hibernate.annotations.Where`` アノテーションを付与し、 clause属性に共通の条件を指定する。
        | ここで指定するWHERE句の書式は、JPQLではなく、SQLで指定する必要がある。つまりJavaオブジェクトのプロパティ名ではなくカラム名を指定する必要がある。
    * - | (2)
      - | ``@Where`` アノテーションで指定した条件が追加されている。

 .. note:: **Dialect 拡張について**

    ``@Where`` アノテーション内でSQL固有のキーワードを指定する場合、HibernateがSQL固有のキーワードを一般的な文字列として認識されてしまい、期待したSQLへ変換されないケースがある。
    SQL固有のキーワードを利用する場合に、 ``Dialect`` を拡張して使用する必要がある。

- 標準的なキーワード ``true`` 、``false`` 、``unknown`` などを登録するための ``Dialect`` を拡張する

 .. code-block:: java

    package com.example.infra.hibernate;
    
    public class ExtendedPostgreSQL9Dialect extends PostgreSQL9Dialect { // (1)
        public ExtendedPostgreSQL9Dialect() {
            super();
            // (2)
            registerKeyword("true");
            registerKeyword("false");
            registerKeyword("unknown");
        }
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1
    
    * - 項番
      - 説明
    * - | (1)
      - | Hibernate 4.3のデフォルトの状態では、SQLのキーワードを正しく認識できない場合がある。例えば、PostgreSQL向けのキーワードを管理する ``org.hibernate.dialect.PostgreSQL9Dialect`` にはキーワードとしてBOOLEAN型の ``true`` 、``false`` 、``unknown`` などが登録されていないため、一般的な文字列として認識されてしまい、正しいSQLへ変換されない。
        | そのため、必要に応じて ``org.hibernate.dialect.Dialect`` を拡張し、キーワードを登録する必要がある。
    * - | (2)
      - | ``@Where`` で利用する可能性のあるSQLキーワードを登録する。

- 拡張したDialectを設定する

 .. code-block:: xml

    <bean id="jpaVendorAdapter"
        class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
        <property name="databasePlatform" value="com.example.infra.hibernate.ExtendedPostgreSQL9Dialect"/> // (3)
        // ...
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1
    
    * - 項番
      - 説明
    * - | (3)
      - | 拡張した ``Dialect`` を ``EntityManager`` である ``JpaVendorAdapter`` の ``databasePlatform`` プロパティの値に設定する。

 .. note:: **指定可能なクラスについて**

    ``@Where`` アノテーションは、 ``@Entity`` が付与されているクラスでのみ有効である。
    つまり、 ``@javax.persistence.MappedSuperclass`` を付与した基底Entityクラスに付与してもSQLに付与されない。

 .. warning::

    ``@Where`` アノテーションは、JPAの標準アノテーションではなく、Hibernate独自のアノテーションである。

|

関連Entityを取得するJPQLに共通条件を加える
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Repositoryインタフェースのメソッド呼び出して取得したEntityの関連Entityを取得するためのJPQLに対して、共通の条件を加える方法を以下に示す。

- Entity

 .. code-block:: java

    @Entity
    @Table(name = "t_order")
    @Where(clause = "is_logical_delete = false")
    public class Order implements Serializable {
        // ...
        @Id
        private Integer id;

        @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
        @OrderBy
        @Where(clause="is_logical_delete = false") // (1)
        private Set<OrderItem> orderItems;
        // ...

    }

- 関連Entityにアクセスした際に発行されるSQL(Lazy Load)

 .. code-block:: sql

    SELECT
            -- ...
        FROM
            t_order_item orderitems0_
        WHERE
            (orderitems0.is_logical_delete = false) -- (2)
            AND orderitems0_.order_id = 1
        ORDER BY
            orderitems0_.item_code ASC
            ,orderitems0_.order_id ASC;

- Entityと関連Entityを同時に取得する際に発行されるSQL(Eager Load / Join Fetch)

 .. code-block:: sql

    SELECT
        -- ...
        FROM
            t_order order0_
                LEFT OUTER JOIN t_order_item orderitems1_
                    ON order0_.id = orderitems1_.order_id
                    AND (orderitems1_.is_logical_delete = false) -- (2)
        WHERE
            order0_.id = 1
            AND (
                order0_.is_logical_delete = false
            )
        ORDER BY
            orderitems1_.item_code ASC
            ,orderitems1_.order_id ASC;

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | 関連Entityのフィールドアノテーションとして、 ``@org.hibernate.annotations.Where`` アノテーションを付与し、 clause属性に共通の条件を指定する。
        | ここで指定するWHERE句の書式は、JPQLではなく、SQLで指定する必要がある。つまりJavaオブジェクトのプロパティ名ではなくカラム名を指定する必要がある。
    * - | (2)
      - | ``@Where`` アノテーションで指定した条件が追加されている。

 .. note:: **指定可能な関連の種類について**

    関連Entityを取得する際のSQLに共通の条件を加えることができるのは、 ``@OneToMany`` および ``@ManyToMany`` の関連をもつEntityに対してのみとなる。

 .. warning::

    ``@Where`` アノテーションは、JPAの標準アノテーションではなく、Hibernate独自のアノテーションである。

|

複数のPersistenceUnitを使用する方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo::

 **TBD**

    今後、以下の内容を追加する予定である。

    * 複数のPersistenceUnitの使用例。

|

Nativeクエリの使用方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo::

 **TBD**

    今後、以下の内容を追加する予定である。

    * Nativeクエリを使用したQueryの実装例。

.. raw:: latex

   \newpage

