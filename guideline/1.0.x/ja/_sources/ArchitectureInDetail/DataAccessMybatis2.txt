データベースアクセス（Mybatis2編）
================================================================================

.. only:: html

 .. contents:: 目次
    :local:
    :depth: 3

Overview
--------------------------------------------------------------------------------

| 本節では、MyBatis2.xを使って、データベースにアクセスする方法について説明する。
| 本ガイドラインでは、TERASOLUNA DAO(MybatisのラッパーDAO)を使用することを前提としている。

 .. figure:: images/dataaccess_mybatis.png
    :alt: Target of description
    :width: 100%
    :align: center

    **Picture - Target of description**


Mybatisについて
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Mybatisは、O/R Mapperの一つだが、データベースで管理されているレコードと、オブジェクトをマッピングするという考え方ではなく、
| SQLとオブジェクトをマッピングするという考え方で開発されたO/R Mapperである。
| そのため、正規化されていないデータベースへアクセスする場合や、発行するSQLをO/R Mapperに任せずに、アプリケーション側で完全に制御したい場合に有効なO/R Mapperである。
| Mybatis2.xの詳細については、\ `Mybatis Developer Guide(PDF) <https://mybatis.googlecode.com/files/MyBatis-SqlMaps-2_en.pdf>`_\ を参照されたい。


TERASOLUNA DAOについて
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
TERASOLUNA DAOは、O/R Mapperに依存する処理を隠蔽するためのDAOインタフェースと、Mybatis2.xを使用したDAO実装クラスを提供している。

TERASOLUNA DAOから提供されているDAOインタフェースは、以下の通りである。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table:: **TERASOLUNA DAOから提供されているDAOインタフェース**
    :header-rows: 1
    :widths: 10 35 55

    * - 項番
      - クラス名
      - 説明
    * - 1.
      - | jp.terasoluna.fw.dao.
        | QueryDAO
      - 参照系SQLを実行するためのDAOインタフェース
    * - 2.
      - | jp.terasoluna.fw.dao.
        | UpdateDAO
      - 更新系SQLを実行するためのDAOインタフェース
    * - 3.
      - | jp.terasoluna.fw.dao.
        | StoredProcedureDAO
      - StoredProcedureを実行するためのDAOインタフェース
    * - 4.
      - | jp.terasoluna.fw.dao.
        | QueryRowHandleDAO
      - 参照系SQLを実行して取得されるレコードに対して、一レコードずつ処理を行うためのDAOインタフェース。

TERASOLUNA DAO(Mybatis実装)を使って、データベースにアクセスする際の基本フローを、以下に示す。

 .. figure:: images/dataaccess_mybatis_basic_flow.png
    :alt: Basic flow of TERASOLUNA DAO
    :width: 100%
    :align: center

    **Picture - Basic flow of TERASOLUNA DAO**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ServiceまたはRepositoryから、TERASOLUNA DAOから提供されているDAOインタフェースのメソッドを呼び出す。
        | メソッドの呼び出しパラメータとして、SQLIDとSQLに埋め込む値を保持しているオブジェクトを渡す。
    * - | (2)
      - | TERASOLUNA DAOは、MybatisのAPIに、処理を委譲する。
        | ServiceまたはRepositoryから指定されたSQLIDと、SQLに埋め込む値を保持しているオブジェクトもMybatisに渡される。
    * - | (3)
      - | Mybatisは、指定されたSQLIDに対応するSQLを、設定ファイル(\ ``sqlMap.xml``\ )から取得し、SQLとバインド値を、JDBCドライバに渡す。
        | (実際の値のバインドは、\ ``java.sql.PreparedStatement``\ のAPIが使われている)
    * - | (4)
      - JDBCドライバは、渡されたSQLとバインド値を、データベースに送信することで、SQLを実行する。

|

How to use
--------------------------------------------------------------------------------

pom.xmlの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
インフラストラクチャ層にMyBatis2(TERASOLUNA DAO)を使用する場合、以下のdependencyをpom.xmlに追加する。

 .. code-block:: xml

    <!-- (1) -->
    <dependency>
        <groupId>org.terasoluna.gfw</groupId>
        <artifactId>terasoluna-gfw-mybatis2</artifactId>
    </dependency>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | MyBatis2に関連するライブラリ群が定義してある ``terasoluna-gfw-mybatis2`` をdependencyに追加する。

|

アプリケーションの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

データソースの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
データソースに設定は、共通編の\ :ref:`data-access-common_howtouse_datasource`\ を参照されたい。


PlatformTransactionManagerの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| ローカルトランザクションを使用する場合は、以下の通りに設定する。

 ローカルトランザクションを使用する場合、JDBCのAPIを呼び出してトランザクション制御を行う\ ``org.springframework.jdbc.datasource.DataSourceTransactionManager``\ を使用する。

- xxx-env.xml

 .. code-block:: xml

     <bean id="transactionManager"
         class="org.springframework.jdbc.datasource.DataSourceTransactionManager"> <!-- (1) -->
         <property name="dataSource" ref="dataSource" /> <!-- (2) -->
     </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - \ ``org.springframework.jdbc.datasource.DataSourceTransactionManager``\ を指定する。
    * - | (2)
      - 設定済みのデータソースのbeanを指定する。


アプリケーションサーバから提供されているトランザクションマネージャを使用する場合は、以下の通りに設定する。

 アプリケーションサーバから提供されているトランザクションマネージャを使用する場合、JTAのAPIを呼び出してトランザクション制御を行う\ ``org.springframework.transaction.jta.JtaTransactionManager``\ を使用する。

- xxx-env.xml

 .. code-block:: xml

     <tx:jta-transaction-manager /> <!-- (1) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | アプリケーションがデプロイされているアプリケーションサーバに、最適な\ ``JtaTransactionManager``\ が、\ ``"transactionManager"``\ というidで、bean定義される。


TERASOLUNA DAOの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Spring Frameworkから提供されている\ ``SqlMapClient``\ のファクトリクラスと、TERASOLUNA DAOのbean定義を行う。

- xxx-infra.xml

 .. code-block:: xml

     <bean id="sqlMapClient"
         class="org.springframework.orm.ibatis.SqlMapClientFactoryBean"> <!-- (1) -->
         <property name="configLocations"
             value="classpath*:/META-INF/mybatis/config/*sqlMapConfig.xml" /> <!-- (2) -->
         <property name="mappingLocations"
             value="classpath*:/META-INF/mybatis/sql/**/*-sqlmap.xml" /> <!-- (3) -->
         <property name="dataSource" ref="dataSource" /> <!-- (4) -->
     </bean>

     <bean id="queryDAO"
         class="jp.terasoluna.fw.dao.ibatis.QueryDAOiBatisImpl"> <!-- (5) -->
         <property name="sqlMapClient" ref="sqlMapClient" /> <!-- (6) -->
     </bean>

     <!-- (5) (6) -->
     <bean id="updateDAO"
         class="jp.terasoluna.fw.dao.ibatis.UpdateDAOiBatisImpl">
         <property name="sqlMapClient" ref="sqlMapClient" />
     </bean>

     <!-- (5) (6) -->
     <bean id="spDAO"
         class="jp.terasoluna.fw.dao.ibatis.StoredProcedureDAOiBatisImpl">
         <property name="sqlMapClient" ref="sqlMapClient" />
     </bean>

     <!-- (5) (6) -->
     <bean id="queryRowHandleDAO"
         class="jp.terasoluna.fw.dao.ibatis.QueryRowHandleDAOiBatisImpl">
         <property name="sqlMapClient" ref="sqlMapClient" />
     </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - \ ``SqlMapClient``\ クラスのファクトリクラスとして、\ ``org.springframework.orm.ibatis.SqlMapClientFactoryBean``\ を指定する。
    * - | (2)
      - | Mybatisの設定ファイルのロケーションを指定する。
        | 例では、クラスパス上の「\ ``/META-INF/mybatis/config/``\ 」ディレクトリに格納されている「\ ``sqlMapConfig.xml``\ 」で終わるファイルが、対象ファイルとなる。
        | 設定ファイルについては、\ :ref:`sqlmapconfig-label`\ を参照されたい。
    * - | (3)
      - | MybatisのSQLマッピングファイルのロケーションを指定する。
        | 例では、クラスパス上の「\ ``/META-INF/mybatis/sql/``\ 」ディレクトリ配下（サブディレクトリも含む）に格納されている「\ ``-sqlmap.xml``\ 」で終わるファイルが、対象ファイルとなる。
        | SQLマッピングファイルについては、\ :ref:`sqlmap-label`\ を参照されたい。
    * - | (4)
      - 設定済みのデータソースのbeanを指定する。
    * - | (5)
      - TERASOLUNA DAOのMybatis実装クラスを指定して、bean定義する。
    * - | (6)
      - (1)で定義した\ ``SqlMapClient``\ クラスのファクトリクラスのbeanを指定する。


LOB型を扱う場合の設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
BLOBやCLOBなどのLarge Objectを扱う場合は、``SqlMapClient``\ クラスのファクトリクラスに、\ ``LobHandler``\ を指定する。

- xxx-infra.xml

 .. code-block:: xml

     <!-- (1) -->
    <bean id="nativeJdbcExtractor"
        class="org.springframework.jdbc.support.nativejdbc.SimpleNativeJdbcExtractor" />

    <!-- (2) -->
    <bean id="lobHandler" class="org.springframework.jdbc.support.lob.OracleLobHandler">
        <property name="nativeJdbcExtractor" ref="nativeJdbcExtractor" /> <!-- (3) -->
    </bean>

    <bean id="sqlMapClient"
        class="org.springframework.orm.ibatis.SqlMapClientFactoryBean">
        <property name="configLocations"
            value="classpath*:/META-INF/mybatis/config/*sqlMapConfig.xml" />
        <property name="mappingLocations"
            value="classpath*:/META-INF/mybatis/sql/**/*-sqlmap.xml" />
        <property name="dataSource" ref="dataSource" />
       <property name="lobHandler" ref="lobHandler" /> <!-- (4) -->
    </bean>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``org.springframework.jdbc.support.nativejdbc.NativeJdbcExtractor``\ インタフェースの実装クラスを、bean定義する。
        | 例では、\ ``org.springframework.jdbc.support.nativejdbc.SimpleNativeJdbcExtractor``\ を指定しているが、
        | Tomcat以外のAPサーバでは、nativeデータソースを取得できない場合があるので、
        | Springが提供している他のNativeJdbcExtractorを指定するか、各APサーバ用に、新たに\ ``NativeJdbcExtractor``\ を作成する必要がある。
    * - | (2)
      - | \ ``org.springframework.jdbc.support.lob.LobHandler``\ インタフェースの実装クラスをbean定義する。
        | 例では、Oracle使用時に指定する\ ``org.springframework.jdbc.support.lob.OracleLobHandler``\ を指定しているが、
        | Oracle以外の場合は、\ ``org.springframework.jdbc.support.lob.DefaultLobHandler``\ を指定する。
    * - | (3)
      - | (1)で定義した\ ``NativeJdbcExtractor``\ のbeanを指定する。
    * - | (4)
      - | (3)で定義した\ ``LobHandler``\ のbeanを指定する。


.. _sqlmapconfig-label:

Mybatisの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\ ``SqlMapClient``\ のデフォルトの動作をカスタマイズする。必要に応じてカスタマイズすること。

- sqlMapConfig.xml

 .. code-block:: xml

     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE sqlMapConfig
                 PUBLIC "-//ibatis.apache.org//DTD SQL Map Config 2.0//EN"
                 "http://ibatis.apache.org/dtd/sql-map-config-2.dtd"> <!-- (1) -->

     <sqlMapConfig>
         <settings useStatementNamespaces="true" /> <!-- (2) -->
     </sqlMapConfig>

 .. tabularcolumns:: |p{0.06\linewidth}|p{0.94\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 6 94

    * - 項番
      - 説明
    * - | (1)
      - DTDファイルを指定する。指定することで、スキーマのチェックと、IDE上でのコード補完が有効となる。
    * - | (2)
      - \ ``useStatementNamespaces="true"``\ を設定することで、SQLマッピングファイルで指定するネームスペースを、SQLIDとして使用するように設定している。


- sqlMapConfigの子要素について
 | 子要素として、\ ``properties``\ , \ ``settings``\ ,\ ``resultObjectFactory``\ , \ ``typeAlias``\ , \ ``transactionManager``\ , \ ``sqlMap``\ が存在する。
 | 必要に応じて、設定を行うこと。
 | 詳細は、Mybatis Developer Guide(PDF)の「The SQL Map XML Configuration File」(P.8-16)を参照されたい。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table:: **sqlMapConfigの子要素**
    :header-rows: 1
    :widths: 10 20 70

    * - 項番
      - 要素
      - 説明
    * - 1.
      - properties
      - | プロパティファイルを読み込むための要素。読み込んだプロパティファイルに定義されているプロパティは、
        | Mybatisの設定ファイルおよびSQLマッピングファイル内から、\ ``"${プロパティ名}"``\ の形式で参照することができる。
        | 環境に依存する値や、共通的な設定値を定義する際に使用する。
        | 詳細は、Mybatis Developer Guide(PDF)の「The SQL Map XML Configuration File」(P.9)を参照されたい。
    * - 2.
      - settings
      - | \ ``SqlMapClient``\ のデフォルト動作のカスタマイズを行うための要素。
        | 設定項目の詳細については、Mybatis Developer Guide(PDF)の「The SQL Map XML Configuration File」(P.9-11)を参照されたい。
    * - 3.
      - resultObjectFactory
      - | SQLマッピングファイルのselect要素、statement要素、procedure要素のresultClass属性、または、resultMap要素のclass属性に指定されたクラスのインスタンスを生成するファクトリクラスを指定する要素。
        | 指定しない場合は、デフォルト実装である\ ``java.lang.Class#newInstance()``\ メソッドで生成されたインスタンスが使用される。
        | 詳細については、Mybatis Developer Guide(PDF)の「The SQL Map XML Configuration File」(P.11-12)を参照されたい。
    * - 4.
      - typeAlias
      - | クラス名(FQCN)に別名（短縮名）を付けるための要素。
        | ここで定義した別名は、Mybatisの設定ファイルおよびSQLマッピングファイルのクラスを指定する箇所で使うことができる。通常、パッケージを除いたシンプルなクラス名を指定することが多い。
        | 詳細については、Mybatis Developer Guide(PDF)の「The SQL Map XML Configuration File」(P.12)を参照されたい。
    * - 5.
      - transactionManager
      - トランザクション管理は、Spring Frameworkの機能を使うため、定義は不要である。
    * - 6.
      - sqlMap
      - TERASOLUNA DAOの設定で設定済みのため、定義は不要である。


.. _sqlmap-label:

SQLマッピングの実装(基本編)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
以下に、基本的なSQLマッピングの実装例を示す。

アプリケーション内で使用するSQLを実装する。

- xxx-sqlmap.xml

 .. code-block:: xml

     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE sqlMap
                 PUBLIC "-//ibatis.apache.org//DTD SQL Map 2.0//EN"
                 "http://ibatis.apache.org/dtd/sql-map-2.dtd"> <!-- (1) -->

     <sqlMap namespace="xxx"> <!-- (2) -->

         <!-- (3) -->
         <select id="findOne">
             <!-- ... -->
         </select>

         <!-- ... -->

     </sqlMap>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - DTDファイルを指定する。指定することで、スキーマのチェックと、STS上でのコード補完が有効となる。
    * - | (2)
      - ネームスペースを指定する。
    * - | (3)
      - \ ``sqlMapConfig.xml``\ にて、ネームスペースをSQLIDとして使用するように設定しているので、このSQLを実行するために指定するSQLIDは「\ ``xxx.findOne``\ 」となる。


- sqlMapの子要素について
 子要素として、\ ``cacheModel``\ , \ ``typeAlias``\ , \ ``parameterMap``\ , \ ``resultMap``\ , \ ``select``\ , \ ``insert``\ , \ ``update``\ , \ ``delete``\ , \ ``statement``\ , \ ``sql``\ , \ ``procedure``\ が存在する。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table:: **sqlMapの子要素**
    :header-rows: 1
    :widths: 10 20 70

    * - 項番
      - 要素
      - 説明
    * - 1.
      - typeAlias
      - \ ``sqlMapConfig.xml``\ の\ ``typeAlias``\ と同じ。
    * - 2.
      - cacheModel
      - オブジェクトのキャッシュの定義を行う要素
    * - 3.
      - parameterMap
      - SQLにバインドするパラメータ（オブジェクト）のマッピングに関する定義を行う要素
    * - 4.
      - resultMap
      - SQLの実行結果として返却されるレコードとオブジェクトのマッピングに関する定義を行う要素
    * - 5.
      - select
      - SELECT文を記載する要素
    * - 6.
      - insert
      - INSERT文を記載する要素
    * - 7.
      - update
      - UPDATE文を記載する要素
    * - 8.
      - delete
      - DELETE文を記載する要素
    * - 9.
      - statement
      - \ ``select``\ , \ ``insert``\ , \ ``update``\ , \ ``delete``\ , \ ``procedure``\  要素を包含している汎用要素。個別の要素(\ ``select``\ , \ ``insert``\ , \ ``update``\ , \ ``delete``\ , \ ``procedure``\ )を使用することを推奨する。
    * - 10.
      - sql
      - \ ``select``\ , \ ``insert``\ , \ ``update``\ , \ ``delete``\ , \ ``statement``\ からインクルードするためのSQL文（SQL文の一部）を記載する要素。この要素をうまく使うことで、複数のSQLで重複している部分を、共通化することができる。
    * - 11.
      - procedure
      - PROCEDURE呼び出しを記載する要素

 .. note ::
     詳細は、Mybatis Developer Guide(PDF)の、以下の章を参照されたい。

     * | The SQL Map XML File(P.17-18)
       | SQLマッピングファイルの簡単な定義例が記載されている。
     * | Mapped Statements(P.18-26)
       | SQLを組み立てるための要素の、基本的な使い方が記載されている。
     * | Parameter Maps and Inline Parameters(P.27-31)
       | SQLにバインドするパラメータ（オブジェクト）のマッピングに関する、詳細な説明が記載されている。
     * | Substitution Strings(P.32)
       | SQLのバインド変数について、記載されている。
     * | Result Maps(P.32-41)
       | SQLの実行結果として返却されるレコードと、オブジェクトのマッピングに関する、詳細な説明が記載されている。
     * | Supported Types for Parameter Maps and Result Maps(P.42-43)
       | ParameterMapと、ResultMapでサポートされている型と、拡張方法が記載されている。
     * | Caching Mapped Statement Results(P.44-47)
       | キャッシュに関する詳細な説明が、記載されている。
     * | Dynamic Mapped Statements(P.48-53)
       | 動的SQLに関する詳細な説明が、記載されている。
     * | Simple Dynamic SQL Elements(P.53)
       | 動的SQLの簡易的な実装方法の説明が、記載されている。

 .. warning::

    \ ``statement``\ , \ ``select``\ , \ ``procedure``\ 要素を使用して、大量データを返すようなクエリを記述する場合には、fetchSize属性に適切な値を設定しておくこと。
    fetchSize属性は、JDBCドライバとデータベース間の通信において、一度の通信で取得するデータの件数を設定するパラメータである。
    fetchSize属性を省略した場合は、各JDBCドライバのデフォルト値が利用されるため、デフォルト値が全件取得するJDBCドライバの場合、メモリの枯渇の原因になる可能性があるので、注意が必要となる。


select要素の実装例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

select要素を実装する前に、検索したレコードのカラムと、JavaBeanのプロパティのマッピング定義を行う。

- xxx-sqlmap.xml

 .. code-block:: xml

     <resultMap id="resultMap_Todo"
                class="xxxxxx.yyyyyy.zzzzzz.domain.model.Todo"> <!-- (1) -->
         <result property="todoId" column="todo_id" /> <!-- (2) -->
         <result property="todoTitle" column="todo_title" />
         <result property="finished" column="finished" />
         <result property="createdAt" column="created_at" />
         <result property="version" column="version" />
     </resultMap>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 10 80

    * - 項番
      - 属性
      - 説明
    * - | (1)
      - | -
      - 検索したレコードとJavaBeanのマッピングを行う。詳細は、Developer Guideを参照されたい。
    * - |
      - | id
      - マッピングを識別するためのIDを指定する。select属性から参照される。
    * - |
      - | class
      - マッピングするJavaBeanのFQCNを指定する。
    * - | (2)
      - | -
      - JavaBeanのプロパティと、検索したレコードのカラムのマッピングを行う。
    * - |
      - | property
      - JavaBeanのプロパティ名を指定する。
    * - |
      - | column
      - property属性で指定したプロパティに、マッピングするレコードのカラム名を指定する。


select要素を実装する。

- xxx-sqlmap.xml

 .. code-block:: xml

     <select id="findOne"
             parameterClass="java.lang.String"
             resultMap="resultMap_Todo"> <!-- (3) -->
         SELECT
             *
         FROM
             todo
         WHERE
             todo_id = #todoId#   /* (4) */
     </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 10 80

    * - 項番
      - 属性
      - 説明
    * - | (3)
      - | -
      - 検索用SQLを実装する。
    * - |
      - | id
      - 検索SQLを識別するためのIDを指定する。
    * - |
      - | parameterClass
      - | バインド用オブジェクトの型を指定する。
        | 例では、\ ``java.lang.String``\ を指定しているが、複数のパラメータ(検索条件)を渡したい場合は、JavaBeanを指定することもできる。
    * - |
      - | resultMap
      - | (1)で定義したresultMapを指定する。
        | resultMapを使わずに、自動的にclass属性で指定したJavaBeanのプロパティにマッピングすることもできるが、取得レコードのカラム名と、JavaBeanのプロパティ名が一致している必要がある。
        | 取得レコードのカラム名とJavaBeanのプロパティ名を一致させる方法として、AS句を使って、カラム名に別名を付与する方法がある。
        | 例えば、SQLを\ ``"SELECT todo_title AS todoTitle, ..."``\ とすると、JavaBeanのtodoTitleプロパティに、値が設定される。
    * - | (4)
      - | -
      - | SQLにバインド値を指定する。
        | 例では、JavaBeanではなく単一オブジェクト（\ ``java.lang.String``\ ）を使用しているので、バインド変数名は任意の名前を指定することができる。
        | バインド用オブジェクトにJavaBeanを使用する場合は、バインド用の変数名は、JavaBeanのプロパティ名と一致させる必要がある。

 .. note:: **自動マッピングについて**

     resultMap属性を使わずに、resultClass属性で指定したJavaBeanのプロパティに、自動的にマッピングすることもできるが、取得レコードのカラム名と、JavaBeanのプロパティ名が一致している必要がある。
     取得レコードのカラム名と、JavaBeanのプロパティ名を一致させる方法として、AS句を使って、カラム名に別名を付与する方法がある。下記に、自動マッピングを使用した場合の実装例を示す。

      .. code-block:: xml
         :emphasize-lines: 3,5-6,8

         <select id="findOne"
                 parameterClass="java.lang.String"
                 resultClass="xxxxxx.yyyyyy.zzzzzz.domain.model.Todo">
             SELECT
                 todo_id AS todoId,
                 todo_title AS todoTitle,
                 finished,
                 created_at AS createdAt,
                 version
             FROM
                 todo
             WHERE
                 todo_id = #todoId#
         </select>

     自動マッピングは、取得したレコードとJavaBeanをマッピングする手段としては、もっとも簡単な方法である。ただし、自動マッピングを使用した場合は、以下の制約や注意点があることを考慮し、使用すること。

     * SQLで取得した値の型宣言や、変換定義などが行えない。
     * 複雑なマッピング（例えば、ネストされているJavaBeanへのマッピング）が行えない。
     * マッピングする際に、\ ``java.sql.ResultSetMetaData``\ にアクセスするため、若干のパフォーマンス劣化が発生する。


insert要素の実装例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

insert要素を実装する。

- xxx-sqlmap.xml

 .. code-block:: xml

     <insert id="insert"
             parameterClass="xxxxxx.yyyyyy.zzzzzz.domain.model.Todo"> <!-- (1) -->
         INSERT INTO todo
             (
                 todo_id
                 ,todo_title
                 ,finished
                 ,created_at
                 ,version
             )
             values(
                 #todoId#       /* (2) */
                 ,#todoTitle#
                 ,#finished#
                 ,#createdAt#
                 ,1
             )
     </insert>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 10 80

    * - 項番
      - 属性
      - 説明
    * - | (1)
      - | -
      - 挿入用SQLを実装する。
    * - |
      - | id
      - 挿入用SQLを識別するためのIDを指定する。
    * - |
      - | parameterClass
      - バインド用オブジェクトの型を指定する。JavaBeanを指定することもできる。
    * - | (2)
      - | -
      - SQLにバインド値を指定する。バインド用オブジェクトにJavaBeanを使用する場合は、バインド用の変数名は、JavaBeanのプロパティ名と一致させる必要がある。

 .. note::

     parameterMap属性や、"Inline Parameter Maps"の仕組みを使用することで、SQLにバインドする値の型の宣言や、変換定義を行うことができる。
     例えば、バインド値が\ ``null``\ の場合に、デフォルト値を設定することができる。詳細は、Mybatis Developer Guide(PDF)の「Parameter Maps and Inline Parameters」(P.27-31)を参照されたい。


update要素の実装例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

update要素を実装する。

- xxx-sqlmap.xml

 .. code-block:: xml

     <update id="update"
             parameterClass="xxxxxx.yyyyyy.zzzzzz.domain.model.Todo"> <!-- (1) -->
         UPDATE todo SET
             todo_id = #todoId#
             ,todo_title = #todoTitle#
             ,finished = #finished#
             ,version = (#version# + 1)
         WHERE
             todo_id = #todoId#
         AND version = #version#
     </update>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - 更新用SQLを実装する。


delete要素の実装例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

delete要素を実装する。

- xxx-sqlmap.xml

 .. code-block:: xml

     <delete id="delete" parameterClass="java.lang.String">  <!-- (1) -->
         DELETE FROM
             todo
         WHERE
             todo_id = #todoId#
     </delete>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - 削除用SQLを実装する。


procedure要素の実装例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

以下は、PostgreSQLに作成したファンクションを、procedure要素を使って呼び出す例となっている。

テーブルと、ファンクション(PL/pgSQL実装)を作成するSQLは、以下の通りである。

 .. code-block:: sql

    CREATE TABLE sales (
        itemno INT4 PRIMARY KEY,
        quantity INT4 NOT NULL,
        price INT4 NOT NULL
    );

 .. code-block:: guess

    CREATE
        FUNCTION sales_item(p_itemno INT4) RETURNS TABLE (
            quantity INT4
            ,total INT4
        ) AS $$ BEGIN RETURN QUERY
            SELECT
                    s.quantity
                    ,s.quantity * s.price
                FROM
                    sales s
                WHERE
                    itemno = p_itemno;
    END;
    $$ LANGUAGE plpgsql;

parameterMap要素を実装する。

 .. code-block:: xml

    <!-- (1) -->
    <parameterMap id="salesItemMap" class="xxxxxx.yyyyyy.zzzzzz.domain.model.SalesItem">
        <!-- (2) -->
        <parameter property="id" jdbcType="INTEGER" mode="IN" />
        <!-- (3) -->
        <parameter property="quantity" jdbcType="INTEGER" mode="OUT" />
        <parameter property="total" jdbcType="INTEGER" mode="OUT" />
    </parameterMap>

 .. code-block:: java

    // (4)
    public class SalesItem implements Serializable {
        private Integer id;
        private Integer quantity;
        private Integer total;
        // ...
    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - ファンクションに渡すINパラメータと、OUTパラメータのマッピングを定義する。
    * - | (2)
      - INパラメータのマッピングを定義している。INパラメータに、\ ``SalesItem#id``\ をマッピングしている。
    * - | (3)
      - OUTパラメータのマッピングを定義している。OUTパラメータの1番目を\ ``SalesItem#quantity``\ に、2番目を\ ``SalesItem#total``\ にマッピングしている。
    * - | (4)
      - マッピング対象となるJavaBean。

 .. note::

    parameterMap属性を使わずに、"Inline Parameter Maps"の仕組みでマッピングする事もできる。
    具体例は、Mybatis Developer Guide(PDF)の「Parameter Maps and Inline Parameters」(P.31)を参照されたい。


procedure要素を実装する。

 .. code-block:: xml

    <procedure id="findSalesItem" parameterMap="salesItemMap"> <!-- (1) -->
        {call sales_item(?,?,?)}
    </procedure>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 呼び出すProcedureやFunctionを、"{call Procedure/Function名(INパラメータ ...,OUTパラメータ...)}"の形式で指定する。
        | 例では、\ ``sales_item``\ というFunctionに対して、INパラメータ1つと、OUTパラメータ2つを指定している。
        | バインドされる値は、parameterMap要素で指定したマッピング定義の定義順となる。


sql要素の実装例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

sql要素の実装する。

- xxx-sqlmap.xml

 .. code-block:: xml

     <sql id="fragment_where_byFinished"> <!-- (1) -->
         WHERE
             finished = #finished#
     </sql>

     <select id="findByFinished"
             parameterClass="boolean"
             resultMap="resultMap_Todo">  <!-- (2) -->
         SELECT
             *
         FROM
             todo
         <include refid="fragment_where_byFinished" /> <!-- (3) -->
         ORDER BY
             created_at DESC
     </select>

     <select id="countByFinished"
             parameterClass="boolean"
             resultClass="long"> <!-- (4) -->
         SELECT
             count(*)
         FROM
             todo
         <include refid="fragment_where_byFinished" /> <!-- (5) -->
     </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - (2)と(4)のSQLで共有するWHERE句を定義している。includeされるSQLの定義は、includeする側のSQLより先に、定義する必要がある。
    * - | (2)
      - 条件に一致するデータを取得するためのSQL
    * - | (3)
      - (1)で定義したWHERE句が実装されているSQLを、includeする。
    * - | (4)
      - 条件に一致するデータ件数を取得するためのSQL
    * - | (5)
      - (1)で定義したWHERE句が実装されているSQLを、includeする。


.. _data-access-mybatis2_howtouse_lob_update:

LOB型更新の実装例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| BLOBおよびCLOBなどのLarge Objectを、データベースに更新する場合の実装例を、以下に示す。
| 下記は、BLOBを扱うテーブルへレコードを挿入する例となっている。

- DDL

 .. code-block:: sql

    CREATE TABLE upload_binary (
        file_id CHAR(36) NOT NULL,
        file_name VARCHAR(256) NOT NULL,
        content BLOB NOT NULL, -- (1)
        CONSTRAINT pk_upload_binary PRIMARY KEY (file_id)
    );

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | BLOB型のカラムを定義する。
        | 上記例では、データベースとしてOracleを使用する前提のDDLとなっている。


- DTO(JavaBean)

 .. code-block:: java

    public class BinaryFile implements Serializable {
        // omitted

        private String fileId;
        private String fileName;
        private InputStream content; // (2)

        // omitted setter/getter

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (2)
      - | BLOB型の値を保持するプロパティを\ ``java.io.InputStream``\ 型で定義する。
        | 上記例では、\ ``InputStream``\ に、アップロードされたファイルの入力ストリームが設定される。

 .. warning::

    BLOBを扱う プロパティの型は、原則\ ``InputStream``\ 型で定義することを推奨する。
    BLOBはバイト配列として扱うこともできるが、データの容量が大きくなると、メモリ枯渇の原因となる可能性がある。

    CLOBを扱うプロパティの型は、 原則\ ``java.io.Reader``\ 型で定義することを推奨する。
    CLOBは文字列として扱うこともできるが、データの容量が大きくなると、メモリ枯渇の原因となる可能性がある。


- xxx-sqlmap.xml

 .. code-block:: xml

    <parameterMap id="uploadBinaryParameterMap"
                  class="xxxxxx.yyyyyy.zzzzzz.domain.service.BinaryFile">
        <parameter property="fileId" />
        <parameter property="fileName" />
        <!-- (3) -->
        <parameter property="content"
                   jdbcType="BLOB"
                   typeHandler="jp.terasoluna.fw.orm.ibatis.support.BlobInputStreamTypeHandler" />
    </parameterMap>

    <!-- (4) -->
    <insert id="uploadBinary" parameterMap="uploadBinaryParameterMap">
        INSERT INTO upload_binary
        (
            file_id
            ,file_name
            ,content
        )
        VALUES
        (
            ?
            ,?
            ,?
        )
    </insert>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (3)
      - | BLOB型のカラムの登録値を保持するパラメータに対して、登録するために必要な定義を指定する。
        | jdbcType属性には\ ``"BLOB"``\ を、typeHandler属性には\ ``"jp.terasoluna.fw.orm.ibatis.support.BlobInputStreamTypeHandler"``\ を指定する。
    * - | (4)
      - | BLOB型のカラムをもつテーブルに、レコードを登録するためのSQL。

 .. note::

    CLOBを扱う場合は、jdbcType属性には\ ``"CLOB"``\ を、typeHandler属性には\ ``"jp.terasoluna.fw.orm.ibatis.support.ClobReaderTypeHandler"``\ を指定する。

 .. tip::

    FQCNで指定しているクラス名は、 typeAlias要素を使って別名を付与することで、シンプルに記載することができる。

     .. code-block:: xml

        <!-- (5) -->
        <typeAlias alias="BinaryFile"
                   type="xxxxxx.yyyyyy.zzzzzz.domain.service.BinaryFile"/>
        <typeAlias alias="BlobInputStreamTypeHandler"
                   type="jp.terasoluna.fw.orm.ibatis.support.BlobInputStreamTypeHandler"/>

        <parameterMap id="uploadBinaryParameterMap"
                      class="BinaryFile"> <!-- (6) -->

            <!-- omitted -->

            <parameter property="content" jdbcType="BLOB"
                       typeHandler="BlobInputStreamTypeHandler" /> <!-- (6) -->

        </parameterMap>

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90

        * - 項番
          - 説明
        * - | (5)
          - | typeAlias要素を使って、クラス名(FQCN)に別名を付与する。
            | 上記例では、\ ``BinaryFile``\ と\ ``BlobInputStreamTypeHandler``\ クラスに対して、別名を付与している。
            | typeAlias要素は、\ :file:`sqlMapConfig.xml`\ と\ :file:`xxx-sqlmap.xml`\ の両方で、定義することができる。
        * - | (6)
          - | (5)で付与したクラス名(FQCN)の別名を指定する。


- Service

 .. code-block:: java

    // omitted

    @Inject
    UpdateDAO updateDAO;

    // omitted

    public BinaryFile uploadBinaryFile(String fileName,
            InputStream contentInputStream) {

        // (7)
        BinaryFile inputFile = new BinaryFile();
        inputFile.setFileId(UUID.randomUUID().toString());
        inputFile.setFileName(fileName);
        inputFile.setContent(contentInputStream);

        // (8)
        updateDAO.execute("example.uploadBinary", inputFile);

        return inputFile;
    }

    // omitted

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (7)
      - | レコードを登録するために必要な情報を、DTOに設定する。
        | 上記例では、ファイルIDをUUIDとして採番し、引数で受け取ったファイル名と、ファイルの中身が格納されている\ ``InputStream``\ オブジェクトを、DTOに設定している。
    * - | (8)
      - | 登録するために必要な情報を、保持するDTOを引数に、\ ``UpdateDAO``\ を呼び出す。
        | DAOの呼び出し方法は、BLOBを扱わない場合と同じである。


- Controller

 .. code-block:: java


    @RequestMapping("uploadBinary")
    public String uploadBinaryFile(
            @RequestPart("file") MultipartFile multipartFile, Model model) throws IOException {
        // (9)
        BinaryFile uploadedFile = uploadService.uploadBinaryFile(multipartFile
                .getOriginalFilename(), multipartFile.getInputStream());
        model.addAttribute(uploadedFile);
        return "upload/form";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (9)
      - | アップロードされたファイルのファイル名と、ファイルの中身が格納されている\ ``InputStream``\ を引数に、Serviceのメソッドを呼び出す。


LOB型取得の実装例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| BLOBおよびCLOBなどのLarge Objectを、データベースから取得する場合の実装例を、以下に示す。
| 下記は、BLOBを扱うテーブルからレコードを取得する例となっている。
| 必要なテーブルを作成するDDLやDTO(JavaBean)は、\ :ref:`data-access-mybatis2_howtouse_lob_update`\ を参照されたい。

- xxx-sqlmap.xml

 .. code-block:: xml

    <resultMap id="selectBinaryResultMap" class="BinaryFile">
        <result property="fileId" column="file_id" />
        <result property="fileName" column="file_name" />
        <!-- (1) -->
        <result property="content" column="content" jdbcType="BLOB"
                typeHandler="BlobInputStreamTypeHandler" />
    </resultMap>

    <!-- (2) -->
    <select id="selectBinary" parameterClass="java.lang.String"
            resultMap="selectBinaryResultMap">
        SELECT
            *
        FROM
            upload_binary
        WHERE
            file_id = #fileId#
    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | BLOB型のカラムから取得した値を保持するプロパティに対して、値を取得するために必要な定義を指定する。
        | jdbcType属性には\ ``"BLOB"``\ を、typeHandler属性には\ ``"jp.terasoluna.fw.orm.ibatis.support.BlobInputStreamTypeHandler"``\ を指定する。
    * - | (2)
      - | BLOB型のカラムをもつテーブルから、レコードを取得するためのSQL。

 .. note::

    CLOBを扱う場合は、jdbcType属性には\ ``"CLOB"``\ を、typeHandler属性には\ ``"jp.terasoluna.fw.orm.ibatis.support.ClobReaderTypeHandler"``\ を指定する。


- Service / Repository

 .. code-block:: java

    // omitted

    @Inject
    QueryDAO queryDAO;

    // omitted

    public BinaryFile getBinaryFile(String fileId) {
        // (3)
        BinaryFile loadedFile = queryDAO.executeForObject(
                "article.selectBinary", fileId, BinaryFile.class);
        return loadedFile;
    }

    // omitted

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (3)
      - | Controllerから指定された取得条件を引数に、\ ``QueryDAO``\ を呼び出す。
        | 上記例では、ファイルIDに一致するアップロードファイルの情報を取得している。
        | DAOの呼び出し方法は、BLOBを扱わない場合と同じである。


SQLマッピングの実装例(動的SQL編)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Mybatisでは、動的にSQLを組み立てる仕組みが、デフォルトで用意されている。
| 以下に、動的にSQLを組み立てる方法について説明する。
| 詳細については、Developer Guide(PDF)の「Dynamic Mapped Statements」(P.48-53)を参照されたい。

パラメータオブジェクトの指定有無を判定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
SQLに渡されたパラメータオブジェクトが指定されているかを判定し、SQLを組み立てることができる。

判定用の要素は、以下の通りである。


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
 .. list-table::
    :widths: 10 15 75
    :header-rows: 1

    * - 項番
      - 要素
      - 説明
    * - 1.
      - isParameterPresent
      - パラメータオブジェクトが指定されている(NULLでない)時のSQLを組み立てるための要素。
    * - 2.
      - isNotParameterPresent
      - パラメータオブジェクトが指定されていない(NULLである)時のSQLを組み立てるための要素。

実装例は、以下の通りである。

 .. code-block:: xml

    <select id="findOne" parameterClass="java.lang.Integer" resultMap="...">
        SELECT
            *
        FROM
            t_order
        WHERE

        <isParameterPresent> <!-- (1) -->
            id = #id#
        </isParameterPresent>

        <isNotParameterPresent> <!-- (2) -->
            1 = 2
        </isNotParameterPresent>

        <!-- ... -->

    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | 例では、パラメータオブジェクトが指定されている時に、idカラムをWHERE句に設定している。
    * - | (2)
      - | 例では、パラメータオブジェクトが指定されていない時に、一致するレコードが0件になるように、条件「\ ``1=2``\ 」を設定している。

上記の動的SQLで生成されるSQLは、以下2パターンとなる。

 .. code-block:: sql

    -- (1) parameterObject(id)=1
    SELECT * FROM t_order WHERE id = 1

    -- (2)
    SELECT * FROM t_order WHERE 1 = 2


パラメータオブジェクト(JavaBean)のプロパティの存在有無を判定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
SQLに渡されたパラメータオブジェクト(JavaBean)に指定したプロパティが存在するか判定し、SQLを組み立てることができる。

判定用の要素は、以下の通りである。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
 .. list-table::
    :widths: 10 15 75
    :header-rows: 1

    * - 項番
      - 要素
      - 説明
    * - 1.
      - isPropertyAvailable
      - 指定したプロパティが、存在する時のSQLを組み立てるための要素。
    * - 2.
      - isNotPropertyAvailable
      - 指定したプロパティが、存在しない時のSQLを組み立てるための要素。

実装例は、以下の通りである。

 .. code-block:: xml

    <select id="findOne" parameterClass="OrderCriteria" resultMap="...">
        SELECT
            *
        FROM
            t_order
        WHERE

        <isPropertyAvailable property="statusCode"> <!-- (1) -->
            status_code = #statusCode#
        </isPropertyAvailable>

        <isNotPropertyAvailable property="statusCode"> <!-- (2) -->
            <![CDATA[
            status_code <> 'completed'
            ]]>
        </isNotPropertyAvailable>

        <!-- ... -->

    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | 例では、\ ``statusCode``\ プロパティが存在する場合に、status_codeカラムが、\ ``statusCode``\ と一致するレコードが取得されるように、WHERE句を設定している。
    * - | (2)
      - | 例では、\ ``statusCode``\ プロパティが存在しない場合に、status_codeカラムが、\ ``'completed'``\ 以外のレコードが取得されるように、WHERE句を設定している。

上記の動的SQLで生成されるSQLは、以下2パターンとなる。

 .. code-block:: sql

    -- (1) statusCode='checking'
    SELECT * FROM t_order WHERE status_code = 'checking'

    -- (2)
    SELECT * FROM t_order WHERE status_code <> 'completed'


パラメータオブジェクト(JavaBean)のプロパティ値の設定有無を判定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
SQLに渡されたパラメータオブジェクト(JavaBean)のプロパティに値が指定されているか判定し、SQLを組み立てることができる。

判定用の要素は、以下の通りである。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
 .. list-table::
    :widths: 10 15 75
    :header-rows: 1

    * - 項番
      - 要素
      - 説明
    * - 1.
      - isNull
      - プロパティの値が、\ ``null``\ 時のSQLを組み立てるための要素。
    * - 2.
      - isNotNull
      - プロパティの値が、\ ``null``\ でない時のSQLを組み立てるための要素。
    * - 3.
      - isEmpty
      - プロパティの値が、\ ``null``\ または、空の時のSQLを組み立てるための要素。
        \ ``Collection``\ および、\ ``String``\ に対して、指定することができる。
    * - 4.
      - isNotEmpty
      - プロパティの値が、\ ``null``\ および、空でない時のSQLを組み立てるための要素。
        \ ``Collection``\ および、\ ``String``\ に対して、指定することができる。

実装例は、以下の通りである。

 .. code-block:: xml

    <select id="findOne" parameterClass="OrderCriteria" resultMap="">
        SELECT
            *
        FROM
            t_order
        WHERE

        <isNull property="orderedDate"> <!-- (1) -->
            <![CDATA[
            CURRENT_DATE - '1 months'::interval <= ordered_date
            ]]>
        </isNull>

        <isNotNull property="orderedDate"> <!-- (2) -->
            ordered_date = #orderedDate#
        </isNotNull>

        <isEmpty property="statusCodes" prepend="AND"> <!-- (3) -->
            <![CDATA[
            status_code <> 'completed'
            ]]>
        </isEmpty>

        <isNotEmpty property="statusCodes" prepend="AND"> <!-- (4) -->
            status_code IN
            <iterate property="statusCodes" open="(" close=")" conjunction=",">
                #statusCodes[]#
            </iterate>
        </isNotEmpty>

    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | 例では、\ ``orderedDate``\ プロパティ(Date型)の値が\ ``null``\ の場合に、ordered_dateカラムが、1ヶ月前以降のレコードが取得されるように、WHERE句を設定している。
    * - | (2)
      - | 例では、\ ``orderedDate``\ プロパティ(Date型) の値が\ ``null``\ でない場合に、ordered_dateカラムが\ ``orderedDate``\ と一致するレコードが取得されるように、WHERE句を設定している。
    * - | (3)
      - | 例では、\ ``statusCodes``\ プロパティ(List<String>型)の値が、空の場合に、status_codeカラムが、\ ``'completed'``\ 以外のレコードが取得されるように、WHERE句を設定している。
    * - | (4)
      - | 例では、\ ``statusCodes``\ プロパティ(List<String>型)の値が、空でない場合に、status_codeカラムが、\ ``statusCodes``\ に格納されているいずれかの値と一致するレコードが取得されるように、WHERE句を設定している。
        | iterate要素の説明は、後述する。

上記の動的SQLで生成されるSQLは、以下4パターンとなる。

 .. code-block:: sql

    -- (1) orderedDate=null, statusCodes=[]
    SELECT * FROM t_order WHERE CURRENT_DATE - '1 months'::interval <= ordered_date
        AND status_code <> 'completed'

    -- (2) orderedDate=null, statusCodes=['accepted','checking']
    SELECT * FROM t_order WHERE CURRENT_DATE - '1 months'::interval <= ordered_date
        AND status_code IN ('accepted','checking')

    -- (3) orderedDate=2013/12/31, statusCodes=null
    SELECT * FROM t_order WHERE ordered_date = '2013/12/31'
        AND status_code <> 'completed'

    -- (4) orderedDate=2013/12/31, statusCodes=['accepted']
    SELECT * FROM t_order WHERE ordered_date = '2013/12/31'
        AND status_code IN ('accepted')

|

パラメータオブジェクト(JavaBean)のプロパティ値を判定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
SQLに渡されたパラメータオブジェクト(JavaBean)のプロパティに指定されている値を判定し、SQLを組み立てることができる。

判定用の要素は、以下の通りである。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
 .. list-table::
    :widths: 10 15 75
    :header-rows: 1

    * - 項番
      - 要素
      - 説明
    * - 1.
      - isEqual
      - プロパティの値が、指定した値と一致する時のSQLを組み立てるための要素。
    * - 2.
      - isNotEqual
      - プロパティの値が、指定した値と一致しない時のSQLを組み立てるための要素。
    * - 3.
      - isGreaterThan
      - プロパティの値が、指定した値より大きい時のSQLを組み立てるための要素。
    * - 4.
      - isGreaterEqual
      - プロパティの値が、指定した値以上の時のSQLを組み立てるための要素。
    * - 5.
      - isLessThan
      - プロパティの値が、指定した値より小さい時のSQLを組み立てるための要素。
    * - 6.
      - isLessEqual
      - プロパティの値が、指定した値以下の時のSQLを組み立てるための要素。

実装例は、以下の通りである。

 .. code-block:: xml

    <select id="findOne" parameterClass="OrderCriteria" resultMap="...">
        SELECT
            *
        FROM
            t_order
        WHERE
            (
            <![CDATA[
            status_code <> 'completed'
            ]]>
            <isEqual property="containCompletedOrder"
                     compareValue="true"
                     prepend="OR"> <!-- (1) -->
                status_code = 'completed'
            </isNull>
            )

    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | 例では、\ ``containCompletedOrder``\ プロパティ(Boolean型)の値が、\ ``true``\ の場合に、status_codeカラムが\ ``'completed'``\ のレコードも取得されるように、WHERE句を設定している。

 .. note::

    compareProperty属性を使用することで、JavaBean内の別のプロパティの値と、比較することもできる。

上記の動的SQLで生成されるSQLは、以下2パターンとなる。

 .. code-block:: sql

    -- (1) containCompletedOrder=false
    SELECT * FROM t_order WHERE (status_code <>  'completed')

    -- (2) containCompletedOrder=true
    SELECT * FROM t_order WHERE (status_code <>  'completed' OR status_code = 'completed')


判定要素の共通属性
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
動的SQLを組み立てるための要素には、以下の共通的な属性が存在する。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
 .. list-table::
    :widths: 10 15 75
    :header-rows: 1

    * - 項番
      - 属性
      - 説明
    * - 1.
      - prepend
      - 動的SQLを組み立てるための判定要素で、\ ``true``\ と判断され、SQLが組み立てられた際に、SQLの先頭に設定する文字列を指定する。
    * - 2.
      - open
      - 動的SQLを組み立てるための判定要素の中で、組み立てたSQLの前に追加する文字列を指定する。
    * - 3.
      - close
      - 動的SQLを組み立てるための判定要素の中で、組み立てたSQLの後に付与する文字列を指定する。

実装例は、以下の通りである。

 .. code-block:: xml

    <select id="findOne" parameterClass="OrderCriteria" resultMap="...">
        SELECT
            *
        FROM
            t_order

        <isNotEmpty property="statusCode"
                    prepend="WHERE"
                    open="("
                    close=")"> <!-- (1) -->
            status_code = #statusCode#
            <isEqual property="containCompletedOrder" compareValue="true" prepend="OR">
                status_code = 'completed'
            </isEqual>
        </isNotEmpty>

    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
 .. list-table::
    :widths: 10 15 75
    :header-rows: 1

    * - 項番
      - 属性
      - 説明
    * - | (1)
      - | -
      - | 例では、\ ``statusCode``\ プロパティに値が指定されている場合に、status_codeカラムをWHERE句にし、
        | \ ``containCompletedOrder``\ プロパティ(Boolean型)の値が\ ``true``\ の場合は、status_codeカラムが\ ``'completed'``\ のレコードも取得されるように、WHERE句を設定している。
    * - | -
      - | prepend
      - | \ ``statusCode``\ プロパティに値が指定されている場合に、SQLに\ ``"WHERE"``\ を設定している。
    * - | -
      - | open
      - | \ ``containCompletedOrder``\ プロパティ(Boolean型)の値が、\ ``true``\ の場合は、OR条件を加えるため、
        | status_codeカラムに対する条件をグループ化するための開始文^\ ``"("``\ を指定している。
    * - | -
      - | close
      - | status_codeカラムに対する条件をグループ化するための終了文字\ ``")"``\ を指定している。

上記の動的SQLで生成されるSQLは、以下3パターンとなる。

 .. code-block:: sql

    -- (1) statusCode=null, containCompletedOrder=false
    SELECT * FROM t_order

    -- (2) statusCode='accepted', containCompletedOrder=false
    SELECT * FROM t_order WHERE (status_code = 'accepted')

    -- (3) statusCode='checking', containCompletedOrder=true
    SELECT * FROM t_order WHERE (status_code = 'checking' OR status_code = 'completed')


コレクションの繰り返し
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
SQLに渡されたバインド値が、コレクションや配列の場合、コレクションおよび配列の要素分処理を繰り返して、SQLを組み立てることができる。

要素は、以下の通りである。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
 .. list-table::
    :widths: 10 15 75
    :header-rows: 1

    * - 項番
      - 要素
      - 説明
    * - 1.
      - iterate
      - コレクションおよび配列に対して、繰り返し処理を行い、SQLを組み立てるための要素。

実装例は、以下の通りである。

 .. code-block:: xml

    <select id="findOne" parameterClass="OrderCriteria" resultMap="...">
        SELECT
            *
        FROM
            t_order

        <isNotNull property="statusCodes" prepend="WHERE">
            <iterate property="statusCodes"
                     prepend="status_code IN"
                     open="("
                     conjunction=","
                     close=")" > <!-- (1) -->
                #statusCodes[]#
            </iterate>
        </isNotNull>

    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
 .. list-table::
    :widths: 10 15 75
    :header-rows: 1

    * - 項番
      - 属性
      - 説明
    * - | (1)
      - | -
      - | 例では、\ ``statusCodes``\ プロパティ(List<String>)に格納されている値を、IN句の値として設定している。
    * - | -
      - | prepend
      - | コレクションまたは配列の要素が存在する場合に、最初に設定する文字列を指定する。例では、条件に加えるカラム名と、IN句を指定している。
    * - | -
      - | open
      - | コレクションまたは配列の最初の要素を処理する前に設定する文字列を指定する。例では、IN句に指定する値の開始囲い文字\ ``"("``\ を指定している。
    * - | -
      - | conjunction
      - | 次の要素がある場合、次の要素の処理を行う前に設定する文字列を指定する。例では、IN句に指定する値の区切り文字\ ``","``\ を指定している。
    * - | -
      - | close
      - | コレクションまたは配列の最後の要素の処理を行った後に、設定する文字列を指定する。例では、IN句に指定する値の終了囲い文字\ ``")"``\ を指定している。

 .. note::

    上記例は、JavaBeanの中のプロパティがコレクションの場合の実装例であるが、パラメータオブジェクト自体をコレクションにすることもできる。
    その場合は、property属性は指定せず、\ ``#[]#``\ という形式でアクセスすることができる。

    コレクションには、JavaBeanを格納することもでき、JavaBeanにネストされているコレクションにも、アクセスすることができる。
    詳細は、Developer Guide(PDF)の「Dynamic Mapped Statements」(P.52)を参照されたい。

上記の動的SQLで生成されるSQLは、以下3パターンとなる。

 .. code-block:: sql

    -- (1) statusCodes=null
    SELECT * FROM t_order

    -- (2) statusCodes=[]
    SELECT * FROM t_order

    -- (3) statusCodes=['accepted','checking']
    SELECT * FROM t_order WHERE status_code IN ('accepted' , 'checking')


動的SQLのブロック化
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
個々の動的SQLをブロック化することで、ブロック全体として、prepend, open, close属性を制御することができる。

要素は、以下の通りである。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
 .. list-table::
    :widths: 10 15 75
    :header-rows: 1

    * - 項番
      - 要素
      - 説明
    * - 1.
      - dynamic
      - 動的SQLを組み立てる要素をブロック化するための要素。

実装例は、以下の通りである。

 .. code-block:: xml

    <select id="findOne" parameterClass="OrderCriteria" resultMap="...">
        SELECT
            *
        FROM
            t_order
        WHERE

        <dynamic prepend="WHERE"
                 open="("
                 close=")"> <!-- (1) -->

            <isNotEmpty property="id" prepend="AND"> <!-- (2) -->
                id = #id#
            </isNotEmpty>

            <isNotEmpty property="statusCode" prepend="AND"> <!-- (3) -->
                status_code = #statusCode#
            </isNotEmpty>

        </dynamic>

    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
 .. list-table::
    :widths: 10 15 75
    :header-rows: 1

    * - 項番
      - 属性
      - 説明
    * - | (1)
      - | -
      - | (2)と(3)の動的SQLを、ブロック化している。
    * - | -
      - | prepend
      - | ブロック内で組み立てたSQLの先頭に設定する文字列を指定する。ここで指定した値は、ブロック内で最初に一致した動的SQLのprepend属性の値として使用される。
        | 上記例だと、\ ``id``\ プロパティに値を指定した場合、(2)のprepend属性の値は、\ ``"AND"``\ ではなく、\ ``"WHERE"``\ となる。
    * - | -
      - | open
      - | ブロック中で組み立てたSQLの前に、追加する文字列を指定する。
    * - | -
      - | close
      - | ブロック中で組み立てたSQLの後に、追加する文字列を指定する。

上記の動的SQLで生成されるSQLは、以下4パターンとなる。

 .. code-block:: sql

    -- (1) id=null, statusCode=null
    SELECT * FROM t_order

    -- (2) id=1, statusCode=null
    SELECT * FROM t_order WHERE (id = 1)

    -- (3) id=null, statusCode='accepted'
    SELECT * FROM t_order WHERE (status_code = 'accepted')

    -- (4) id=1, statusCode='accepted'
    SELECT * FROM t_order WHERE (id = 1 AND status_code = 'accepted')


QueryDAOの使用例
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1件検索
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
検索結果が、0～1件となるクエリを発行したい場合、以下のような実装となる。

- Xxx.java

 .. code-block:: java

     String todoId = "xxxxx....";
     Todo loadedTodo = queryDAO.executeForObject( // (1)
             "todo.findOne",    // (2)
             todoId,            // (3)
             Todo.class);       // (4)
     if (loadedTodo == null) {  // (5)
         // ...                 // (6)
     }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - 検索結果を(4)で指定した型のオブジェクトとして取得するためのメソッド(\ ``QueryDAO#executeForObject``\ )を呼び出す。
    * - | (2)
      - | 検索結果が0～1件となるSQLのSQLIDを指定する。
        | 検索結果が複数件になる場合は、Mybatisが\ ``java.sql.SQLException``\ を発生させる。
    * - | (3)
      - | SQLのバインドパラメータを指定する。
        | 例では、\ ``java.lang.String``\ にしているが、複数のパラメータ(検索条件)を渡したい場合は、JavaBeanを指定することもできる。
    * - | (4)
      - SQLの取得結果をマッピングするオブジェクトの型を指定する。
    * - | (5)
      - 検索結果が0件の場合は、nullになるので、null判定が必要である。
    * - | (6)
      - 検索結果が、0件の場合の処理を実装する。


複数件検索
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
検索結果が、0～N件となるクエリを発行し、条件に一致するデータをすべて取得する場合は、以下のような実装となる。

- Xxx.java

 .. code-block:: java

     boolean finished = false;
     List<Todo> unfinishedTodoList = queryDAO.executeForObjectList( // (1)
             "todo.findByFinished",     // (2)
             finished);                 // (3)
     if(unfinishedTodoList.isEmpty()){  // (4)
         // ...                         // (5)
     }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - オブジェクトのリストを取得するための、メソッドを呼び出す。
    * - | (2)
      - 検索結果が、0～N件となるSQLのSQLIDを指定する。
    * - | (3)
      - | SQLのバインドパラメータを指定する。
        | 例では、booleanにしているが、複数のパラメータ(検索条件)を渡したい場合は、JavaBeanを指定することもできる。
    * - | (4)
      - 検索結果が0件の場合は、空のリストが返却される。nullは返却されないので、nullチェックは不要である。
    * - | (5)
      - 検索結果が、0件の場合の処理を実装する。


ページネーション検索（TERASOLUNA DAO標準機能方式）
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 検索結果が、0～N件となるクエリを発行し、条件に一致するデータの一部(指定ページ部分)を取得する場合は、以下のような実装となる。
| 以下の例では、TERASOLUNA DAOから提供されているAPIを使って、実現する実装例となっている。
\
 .. warning:: **検索条件に一致するデータ件数が非常に多くなる場合の注意点**

    TERASOLUNA DAO標準機能のページネーション検索は、\ ``java.sql.ResultSet#next``\ を使って取得するレコードの開始位置までスキップする実装となっているため、
    検索条件に一致するデータ件数が、非常に多い場合、処理性能に影響を与える可能性がある。
    検索条件に一致するデータ件数が、非常に多くなる可能性がある場合は、TERASOLUNA DAO標準機能のページネーション検索ではなく、SQL絞り込み方式の採用を検討すること。

- Xxx.java

 .. code-block:: java

     Pageable pageable = new PageRequest(0, 10); // (1)
     boolean finished = false;
     long totalCount = queryDAO.executeForObject(
             "todo.countByFinished", // (2)
             finished,
             Long.class);            // (3)

     List<Todo> unfinishedTodoList = null;
     if(0 < totalCount) {
         unfinishedTodoList = queryDAO.executeForObjectList(
             "todo.findByFinished",   // (4)
             finished,
             pageable.getOffset(),    // (5)
             pageable.getPageSize()); // (6)
     } else {
         unfinishedTodoList = new ArrayList<Todo>();
     }

     Page<Todo> page = new PageImpl<Todo>( // (7)
             unfinishedTodoList, // (8)
             pageable,           // (9)
             totalCount);        // (10)

- xxx-sqlmap.xml

 .. code-block:: xml

     <select id="findByFinished"
             parameterClass="boolean"
             resultMap="resultMap_Todo"> <!-- (11) -->
         SELECT
             *
         FROM
             todo
         WHERE
             finished = #finished#
         ORDER BY
             created_at DESC
     </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - Spring Dataより提供されているページング検索用のオブジェクト（\ ``org.springframework.data.domain.PageRequest``\ ）を生成する。
        Pageableオブジェクトは、リクエストパラメータに指定して、Controllerの引数として受け事もできる。詳細は、\ :doc:`Pagination`\ を参照されたい。
    * - | (2)
      - 条件に一致するデータの合計件数を、取得するためのSQLの、SQLIDを指定して実行する。
    * - | (3)
      - 件数の取得なので、Long.classを指定する。
    * - | (4)
      - 検索結果が、0～N件となるSQLの、SQLIDを指定して実行する。
    * - | (5)
      - | 取得開始位置を指定する。
        | 0開始。取得件数が10件のときに、10を指定すると、11～20件目が取得される。
    * - | (6)
      - | 取得件数を指定する。
        | 取得開始位置が0のときに、10を指定すると、1～10件目が取得される。
    * - | (7)
      - Spring Dataより提供されているページ用のオブジェクト（\ ``org.springframework.data.domain.PageImpl``\ ）を生成する。
    * - | (8)
      - ページネーション検索して、取得したリストを指定する。
    * - | (9)
      - ページネーション検索で使用したページング検索用のオブジェクト(Pageable)を指定する。
    * - | (10)
      - 条件に一致するデータの、合計件数を指定する。
    * - | (11)
      - SQLの実装例。SQLとしては、取得位置を意識する必要はない。


ページネーション検索（SQL絞り込み方式）
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 検索結果が0～N件となるクエリを発行し、条件に一致するデータの一部(指定ページ部分)を取得する場合は、以下のような実装となる。
| 以下の例では、TERASOLUNA DAOから提供されているAPIを使わずに、SQLを使って実現する実装例となっている。

- PageableBindParams.java (サンプルクラス)

 .. code-block:: java

     public class PageableBindParams<P> implements Serializable { // (1)
         private static final long serialVersionUID = 1L;
         private final P bindParams;
         private final Pageable pageable;
         public PageableBindParams(P bindParams, Pageable pageable) {
             this.bindParams = bindParams;
             this.pageable = pageable;
         }
         public P getBindParams() {
             return bindParams;
         }
         public Pageable getPageable() {
             return pageable;
         }
     }

- Xxx.java

 .. code-block:: java

     Pageable pageable = new PageRequest(0, 10);
     boolean finished = false;
     long totalCount = queryDAO.executeForObject(
             "todo.countByFinished",
             finished,
             Long.class); // (2)

     List<Todo> unfinishedTodoList = null;
     if(0 < totalCount) {
         PageableBindParams<Boolean> pageableBindParams =
                 new PageableBindParams<Boolean>( // (3)
                         finished,  // (4)
                         pageable); // (5)
         unfinishedTodoList = queryDAO.executeForObjectList(
                 "todo.findPageByFinished", // (6)
                 pageableBindParams);       // (7)
     } else {
         unfinishedTodoList = new ArrayList<Todo>();
     }

     Page<Todo> page = new PageImpl<Todo>(
             unfinishedTodoList,
             pageable,
             totalCount); // (8)

- xxx-sqlmap.xml

 .. code-block:: xml

     <select id="findPageByFinished"
             parameterClass="xxxxxx.yyyyyy.zzzzzz.domain.dto.PageableBindParams"
             resultMap="resultMap_Todo"> <!-- (9) -->
         SELECT
             *
         FROM
             todo
         WHERE
             finished = #bindParams#
         ORDER BY
             created_at DESC
         OFFSET
             #pageable.offset#    /* (10) */
         LIMIT
             #pageable.pageSize#  /* (11) */
    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - 検索条件となるパラメータ（バインドパラメータ）と、Spring Dataより提供されているページング検索用のオブジェクト（\ ``org.springframework.data.domain.Pageable``\ ）を保持するJavaBean。
        DAOに渡せるバインドオブジェクトは一つのみなので、本クラスのような集約オブジェクトが、必要となる。本クラスは、サンプル実装なので、各プロジェクトで必要に応じて用意すること。
    * - | (2)
      - TERASOLUNA DAO標準機能使用時と同様に、合計件数を取得する。
    * - | (3)
      - | DAOに渡すバインド用オブジェクトを生成する。
        | 例では、(1)で用意したクラスを使用する。
    * - | (4)
      - | 対象データを絞り込むための、検索条件を指定する。
        | 例では、finishedの値として、「false」を指定する。
    * - | (5)
      - | 該当ページのデータを絞り込むための、検索条件を指定する。
        | 例では、Spring Dataより提供されているページング検索用のオブジェクト（\ ``org.springframework.data.domain.PageRequest``\ ）を指定している。
        | Pageableオブジェクトは、リクエストパラメータに指定して、Controllerの引数として受けることもできる。詳細は、\ :doc:`Pagination`\ を参照されたい。
    * - | (6)
      - 該当ページのデータを抽出するSQLが実装されているSQLのSQLIDを指定する。
    * - | (7)
      - (3)で生成したバインド用オブジェクトを指定する。
    * - | (8)
      - TERASOLUNA DAO標準機能使用時と同様に、Spring Dataより提供されているページ用のオブジェクト（ ``org.springframework.data.domain.PageImpl`` ）を生成する。
    * - | (9)
      - SQLの実装例。例では、PostgreSQLから提供されている機能(OFFSET,LIMIT)を使用している。SQLとして、取得位置を意識する。
    * - | (10)
      - | 取得開始位置を指定する。
        | 0開始。取得件数が10件のときに、10を指定すると、11～20件目が取得される。(PostgreSQLの機能を使用する)
    * - | (11)
      - | 取得件数を指定する。
        | 取得開始位置が、0のときに、10を指定すると、1～10件目が取得される。(PostgreSQLの機能を使用する)


UpdateDAOの使用例
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1件挿入
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
1件のデータの挿入する場合、以下のような実装となる。

- Xxx.java

 .. code-block:: java

     // (1)
     Todo todo = new Todo();
     todo.setTodoId(todoId);
     todo.setTodoTitle(todoTitle);
     todo.setFinished(false);
     todo.setCreatedAt(now);
     int insertedCount = updateDAO.execute("todo.insert", todo); // (2)
     if(insertedCount != 1){  // (3)
         // ...               // (4)
     }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - 挿入対象のデータ(JavaBean)を生成する。
    * - | (2)
      - 挿入用SQLのSQLIDと、挿入対象のデータ(JavaBean)を指定して、DAOを実行する。
    * - | (3)
      - 必要に応じて、実際に挿入されたデータの件数を、チェックする。例では、挿入件数が1件であるかをチェックしている。
    * - | (4)
      - 必要に応じて、実際に挿入された件数が、想定件数と異なる場合の処理を行う。


複数件挿入(バッチ実行)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 複数のSQLを、バッチ実行することで、複数件のデータを挿入する場合は、以下のような実装となる。
| TERASOLUNA DAOから提供されている\ ``jp.terasoluna.fw.dao.SqlHolder``\ を使用する。

- Xxx.java

 .. code-block:: java

     // (1)
     Todo todo = new Todo();
     todo.setTodoId(todoId);
     todo.setTodoTitle(todoTitle);
     todo.setFinished(false);
     todo.setCreatedAt(now);

     // (2)
     Todo todo2 = new Todo();
     todo2.setTodoId(todoId2);
     todo2.setTodoTitle(todoTitle2);
     todo2.setFinished(false);
     todo2.setCreatedAt(now);

     List<SqlHolder> sqlHolders = new ArrayList<SqlHolder>(); // (3)
     sqlHolders.add(new SqlHolder("todo.insert", todo));      // (4)
     sqlHolders.add(new SqlHolder("todo.insert", todo2));     // (4)
     int insertedCount = updateDAO.executeBatch(sqlHolders);  // (5)
     if(insertedCount != 2){  // (6)
         // ...               // (7)
     }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - 挿入対象のデータ(JavaBean)を生成する。1件目のデータ。
    * - | (2)
      - 挿入対象のデータ(JavaBean)を生成する。2件目のデータ。
    * - | (3)
      - バッチ実行用に、TERASOLUNA DAOから提供されている\ ``jp.terasoluna.fw.dao.SqlHolder``\ のリストを生成する。
    * - | (4)
      - (1), (2)で生成したデータを、バインド用オブジェクトとして、SqlHolderのリストに追加する。例では、2件リストに追加している。
    * - | (5)
      - (1)～(4)で生成したSqlHolderのリストを指定して、バッチを実行する。
    * - | (6)
      - | 必要に応じて、実際に挿入されたデータの件数をチェックする。
        | 例では、挿入件数が2件であるかをチェックしている。
    * - | (7)
      - 必要に応じて、実際に挿入された件数が、想定件数と異なる場合の処理を行う。
\
 .. warning:: **バッチ実行における挿入件数について**

    バッチ実行した場合、JDBCドライバによっては、正確な行数が取得できないケースがある。
    正確に取得できないドライバを使用する場合に、挿入件数をチェックする必要があるケースで、バッチ実行を使用しないこと。
    (更新時の更新件数、削除時の削除件数も同様である。)


1件更新
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 1件のデータの更新する場合、以下のような実装となる。
| 1件挿入の場合と同じである。使用するSQLが、更新用のSQLになる。

- Xxx.java

 .. code-block:: java

     Todo loadedTodo = queryDAO.executeForObject("todo.findOne",
             todoId,
             Todo.class);     // (1)
     todo2.setFinished(true); // (2)
     int updatedCount = updateDAO.execute("todo.update", todo); // (3)
     if(updatedCount != 1){   // (4)
         // ...               // (5)
     }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - 更新対象のデータ(JavaBean)を、検索する。
    * - | (2)
      - データを更新する。例では、finishedを、falseからtrueに更新する。
    * - | (3)
      - 更新用SQLのSQLIDと、更新対象のデータ(JavaBean)を指定して、DAOを実行する。
    * - | (4)
      - | 必要に応じて、実際の更新されたデータの件数をチェックする。
        | 例では、更新件数が1件であるかをチェックしている。
    * - | (5)
      - 必要に応じて、実際に更新された件数が、想定件数と異なる場合の処理を行う。


複数件更新(バッチ実行)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 複数のSQLをバッチ実行することで、複数件のデータを更新する場合の実装例は、複数件挿入(バッチ実行)と同じである。
| 更新値が、レコード毎に異なる場合、バッチ実行による複数件更新が有効である。


複数件更新(WHERE句指定)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| SQLで指定した条件に一致するデータを一括で更新する場合、以下のような実装となる。
| 全レコードを同じ値に一括更新する場合は、WHERE句指定による複数件更新が有効的である。


- Xxx.java

 .. code-block:: java

     int deadlineDays = 7;
     int updatedCount = updateDAO.execute("todo.update", deadlineDays); // (1)

- xxx-sqlmap.xml

 .. code-block:: xml

     <update id="updateFinishedDeadlineByUnfinished" parameterClass="int"> <!-- (2) -->
         <![CDATA[
         UPDATE
             todo
         SET
             todo_title = '[Finished Deadline] ' || todo_title
             ,version = (version + 1)
         WHERE
             finished = false
         AND
             created_at < current_date - #deadlineDays#
         ]]>
    </update>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - 一括更新用SQLのSQLIDと、更新対象のデータを抽出するための条件を指定して、DAOを実行する。
    * - | (2)
      - 一括更新するSQLの実装例。例では、作成してから7日経過して、完了していないTODOのタイトルに\ "[Finished Deadline] "\ という文字列を先頭に付与している。


1件削除
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
1件のデータの削除する場合、以下のような実装となる。

- Xxx.java

 .. code-block:: java

     String todoId = "xxxxx....";
     int deletedCount = updateDAO.execute("todo.delete", todoId); // (1)
     if(deletedCount != 1){
         // ...               // (2)
     }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 削除用SQLのSQLIDとPKを指定して、DAOを実行する。
        | 例では、\ ``java.lang.String``\ にしているが、複合キーの場合は、JavaBeanを指定することもできる。
    * - | (2)
      - 必要に応じて、実際に削除された件数が、想定件数と異なる場合の処理を行う。


複数件削除(バッチ実行)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 複数のSQLをバッチ実行することで、複数件のデータを削除する場合の実装例は、複数件更新(バッチ実行)と同じである。
| 1件削除時の処理を共有する必要がある場合は、バッチ実行による複数件削除を使用する。ただし、削除する対象データが大量になる場合は、WHERE句指定による一括削除の方式を検討した方がよい。


複数件削除(WHERE句指定)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| SQLで指定した条件に一致するデータを一括で削除する場合の実装例は、複数件更新(WHERE句指定)と同じである。
| 削除対象のレコードが大量になる場合は、WHERE句指定による複数件削除が有効的である。


StoredProcedureDAOの使用例
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
プロシージャや、ファンクションを呼び出す場合、以下のような実装となる。

- Xxx.java

 .. code-block:: java

    SalesItem item = new SalesItem(); // (1)
    item.setId(Integer.valueOf(1));  // (2)
    storedProcedureDAO.executeForObject("todo.findSalesItem", item);  // (3)
    // (4)
    logger.debug("Quantity is {}.", item.getQuantity());
    logger.debug("Total is {}.", item.getTotal());


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - プロシージャや、ファンクションのINパラメータを、OUTパラメータを保持するバインド用オブジェクトを生成する。
    * - | (2)
      - INパラメータとして、IDをを設定する。例では、IDとして、\ ``1``\ を設定している。
    * - | (3)
      - ストアードプロシージャ呼び出し用SQLの、SQLIDとバインド用オブジェクトを引数に、\ ``StoredProcedureDAO``\ のメソッドを呼び出す。
    * - | (4)
      - | \ ``StoredProcedureDAO``\ のメソッドの呼び出しが、正常に終了した場合、
        | プロシージャや、ファンクションのOUTパラメータが、バインド用オブジェクトに設定される。
        | 例では、バインド用オブジェクトに設定されたOUTパラメータの値を、ログに出力している。


QueryRowHandleDAOの使用例
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- Xxx.java

 .. code-block:: java


     boolean finished = false;
     queryRowHandleDAO.executeWithRowHandler(
             "todo.findByFinished",            // (1)
             finished,                         // (2)
             new DataRowHandler() {            // (3)
                 public void handleRow(Object valueObject) { // (4)
                     Todo todo = (Todo) valueObject;
                     logger.info(todo.toString());  // (5)
                 }
             });

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - 検索結果が、0～N件となるSQLの、SQLIDを指定する。
    * - | (2)
      - | SQLのバインドパラメータを指定する。
        | 例では、booleanにしているが、複数のパラメータ(検索条件)を渡したい場合は、JavaBeanを指定することもできる。
    * - | (3)
      - | \ ``jp.terasoluna.fw.dao.event.DataRowHandler``\ の実装オブジェクトを指定する。
        | 例では、無名クラスを使用しているが、実際のプロジェクトでは、実装クラスを作成することを検討すること。
    * - | (4)
      - | 検索結果の1レコード毎に、handleRowメソッドが呼び出される。
        | 引数にわたってくるオブジェクトは、select要素にresultClass属性、または、resultMap要素のclass属性に指定したクラスのオブジェクトとなる。
    * - | (5)
      - 例では、ログ出力しているだけだが、実際のプロジェクトで使う場合は、値の加工、各レコード値の集計、ファイル出力などの処理を行うことになる。


.. _data-access-mybatis2_howtouse_like_escape:

LIKE検索時のエスケープについて
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| LIKE検索を行う場合は、検索条件として使用する値を、LIKE検索用にエスケープする必要がある。
| LIKE検索用のエスケープ処理は、共通ライブラリから提供している\ ``org.terasoluna.gfw.common.query.QueryEscapeUtils``\ クラスのメソッドを使用することで、実現できる。
| 共通ライブラリから提供しているエスケープ処理の仕様については、\ :doc:`DataAccessCommon`\ の\ :ref:`data-access-common_appendix_like_escape`\ を参照されたい。

| 以下に、共通ライブラリから提供しているエスケープ処理の、使用方法について説明する。


一致方法をQuery側で指定する場合の使用方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
一致方法(前方一致、後方一致、部分一致)の指定をJPQLとして指定する場合は、エスケープのみ行うメソッドを使用する。

- :file:`xxx-sqlmap.xml`

 .. code-block:: xml

    // (1) (2)
    <select id="findAllByWord" parameterClass="String" resultMap="resultMap_Article">
      SELECT
          *
      FROM
          article
      WHERE
          title LIKE '%' || #word# || '%' ESCAPE '~'
      OR
          overview LIKE '%' || #word# || '%' ESCAPE '~'
    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | SQL内に、LIKE検索用のワイルドカード(\ ``"%"``\ または\ ``"_"``\ )を指定する。
        | 上記例では、引数\ ``word``\ の前後に、ワイルドカード(\ ``"%"``\ )を指定することで、一致方法を部分一致にしている。
    * - | (2)
      - | 共通ライブラリから提供しているエスケープ処理は、エスケープ文字として\ ``"~"``\ を使用しているため、 LIKE句の後ろに\ ``"ESCAPE '~'"``\ を指定する。


- Service or Repository

 .. code-block:: java

    @Inject
    QueryDAO queryDAO;

    @Transactional(readOnly = true)
    public Page<Article> searchArticle(ArticleSearchCriteria criteria,
            Pageable pageable) {

        String escapedWord = QueryEscapeUtils.toLikeCondition(criteria.getWord()); // (3)

        long total = queryDAO.executeForObject("article.countByWord",
                escapedWord, Long.class);
        List<Article> contents = null;
        if (0 < total) {
            contents = queryDAO.executeForObjectList("article.findAllByWord",
                    escapedWord, pageable.getOffset(), pageable.getPageSize()); // (4)
        } else {
            contents = Collections.emptyList();
        }
        return new PageImpl<Article>(contents, pageable, total);
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (3)
      - | LIKE検索の一致方法をQuery側で指定する場合は、\ ``QueryEscapeUtils#toLikeCondition(String)``\ メソッドを呼び出し、LIKE検索用のエスケープのみ行う。
    * - | (4)
      - | LIKE検索用にエスケープされた値を、\ ``QueryDAO``\ のバインドパラメータに渡す。


一致方法をロジック側で指定する場合の使用方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
一致方法(前方一致、後方一致、部分一致)をロジック側で判定する場合は、エスケープされた値にワイルドカードを付与するメソッドを使用する。

- :file:`xxx-sqlmap.xml`

 .. code-block:: xml

    // (1)
    <select id="findAllByWord" parameterClass="String" resultMap="resultMap_Article">
      SELECT
          *
      FROM
          article
      WHERE
          title LIKE #word# ESCAPE '~'
      OR
          overview LIKE #word# ESCAPE '~'
    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | SQL内に、LIKE検索用のワイルドカードは、指定しない。


- Service or Repository

 .. code-block:: java

    @Inject
    QueryDAO queryDAO;

    @Transactional(readOnly = true)
    public Page<Article> searchArticle(ArticleSearchCriteria criteria,
            Pageable pageable) {

        String escapedWord  = QueryEscapeUtils
                .toContainingCondition(criteria.getWord()); // (2)

        long total = queryDAO.executeForObject("article.countByWord",
                escapedWord, Long.class);
        List<Article> contents = null;
        if (0 < total) {
            contents = queryDAO.executeForObjectList("article.findAllByWord",
                    escapedWord, pageable.getOffset(), pageable.getPageSize()); // (3)
        } else {
            contents = Collections.emptyList();
        }
        return new PageImpl<Article>(contents, pageable, total);
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
      - | LIKE検索用にエスケープ＋ワイルドカードが付与された値を、\ ``QueryDAO``\ のバインドパラメータに渡す。


SQL Injection対策について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| SQLを組み立てる際は、SQL Injectionが発生しないように、注意する必要がある。
| Mybatis2では、SQLに値を埋め込む仕組みを、2つ提供している。

* | バインド変数を使って埋め込む方法。
  | この方法を使用すると、 SQL組み立て後に\ ``java.sql.PreparedStatement``\ を使用して、値が埋め込められるため、安全に値を埋め込むことができる。
  | **ユーザからの入力値をSQLに埋め込む場合は、原則バインド変数を使用すること。**

* | 置換変数を使って埋め込む方法。
  | この方法を使用すると、SQLを組み立てるタイミングで、文字列として置換されてしまうため、安全な値の埋め込みは、保証されない。

 .. warning::

    ユーザからの入力値を置換変数を使って埋め込むと、SQL Injectionが発生する危険性が高くなることを意識すること。
    ユーザからの入力値を置換変数を使って埋め込む必要がある場合は、かならずSQL Injectionが発生しないことを保障するための、入力チェックを実施すること。

    基本的には、 **ユーザからの入力値はそのまま使わないことを強く推奨する。**


バインド変数を使って埋め込む方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| バインド変数を使用する場合は、 ParameterMapまたはInline Parametersを使用する。
| 以下に、使用例を示す。

ParameterMapの使用例を、以下に示す。

 .. code-block:: xml

    <!-- (1) -->
    <parameterMap id="uploadBinaryParameterMap" class="BinaryFile">
        <parameter property="fileId" />
        <parameter property="fileName" />
        <parameter property="content" jdbcType="BLOB" typeHandler="BlobInputStreamTypeHandler" />
    </parameterMap>

    <insert id="uploadBinary" parameterMap="uploadBinaryParameterMap">
        INSERT INTO upload_binary
        (
            file_id
            ,file_name
            ,content
        )
        VALUES
        (
            ?   /* (2) */
            ,?
            ,?
        )
    </insert>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | バインド変数として値を埋め込むプロパティを定義する。定義した順番が、(2)で指定している\ ``?``\ の位置に対応する。
    * - | (2)
      - | SQLにバインド変数を指定する。(1)で定義した順番で、\ ``?``\ の部分に、値がバインドされる。


Inline Parametersの使用例を、以下に示す。

 .. code-block:: xml

    <insert id="insert"
            parameterClass="xxxxxx.yyyyyy.zzzzzz.domain.model.Todo"> <!-- (1) -->
        INSERT INTO todo
            (
                todo_id
                ,todo_title
                ,finished
                ,created_at
                ,version
            )
            values(
                #todoId#       /* (3) */
                ,#todoTitle#
                ,#finished#
                ,#createdAt#
                ,1
            )
    </insert>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (3)
      - | バインドする値が格納されているプロパティのプロパティ名を、\ ``#``\ で囲み、バインド変数として指定する。


置換変数を使って埋め込む方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
バインド変数を使用する場合の使用例を、以下に示す。

 .. code-block:: xml

    <select id="findByFinished"
            parameterClass="..."
            resultMap="resultMap_Todo">
        SELECT
            *
        FROM
            todo
        WHERE
            finished = #finished#
        ORDER BY
            created_at $direction$  /* (4) */
    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (4)
      - | 置換する値が格納されているプロパティのプロパティ名を\ ``$``\ で囲み、置換変数として指定する。
        | 上記例では、\ ``$direction$``\ の部分は、\ ``"DESC"``\ または\ ``"ASC"``\ で置換される。

 .. warning::

    置換変数による埋め込みは、必ずアプリケーションとして安全な値であることを担保した上で、テーブル名、カラム名、ソート条件などに限定して、使用することを推奨する。
    
    例えば、以下のようにコード値と実際に使用する安全な値をペアでMapに格納し、

      .. code-block:: java
      
        Map<String, String> safeValueMap = new HashMap<String, String>();
        safeValueMap.put("1", "ASC");
        safeValueMap.put("2", "DESC");
      
    実際の入力はコード値になるようにして、SQLを実行する処理中で変換することが望ましい。

      .. code-block:: java
      
        String direction = safeValueMap.get(input.getDirection());

    \ :doc:`Codelist`\ を使用しても良い。

|

Appendix
--------------------------------------------------------------------------------

関連オブジェクトを１回のSQLでまとめて取得する実装例
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| テーブル毎にEntityのようなJavaBeanを用意して、データベースにアクセスする際に、関連オブジェクトを、1回のSQLでまとめて取得する方法について説明する。
| この方法は、N+1問題を回避する手段としても使用される。

.. warning::

  以下の点に注意して、使用すること。

  * 本例では、使い方を説明するために、すべての関連オブジェクトを、1回のSQLでまとめて取得している。
    しかしながら、実際のプロジェクトで使用する場合は、処理で必要となる関連オブジェクトのみ取得するようにすること。
    なぜなら、使用しない関連オブジェクトを、同時に取得してしまった場合、性能劣化の原因となるケースがあるからである。
  * 使用頻度の低い、1:Nの関係をもつ関連オブジェクトについては、まとめて取得しない。
    必要なときに、個別に取得する方法を採用した方がよいケースがある。
    性能要件を満たせる場合は、まとめて取得してもよい。
  * 1:Nの関係となる関連オブジェクトが、多く含まれる場合、まとめて取得すると、マッピング処理に使用されない無駄なデータの取得が行われ、性能劣化の原因となるケースがある。
    性能要件を満たせる場合は、まとめて取得してもよいが、他の方法を検討した方がよい。

.. tip::

  N+1問題の回避手段については、Mybatis Developer Guide(PDF)の「Result Maps/Avoiding N+1 Selects (1:1)」(P.37-38)及び「Result Maps/Avoiding N+1 Selects (1:M and M:N)」(P.39-40)を参照されたい。


| 以降では、注文テーブルを使って、具体的に実装例について説明する。
| 説明で使用するテーブルは、以下の通りである。

 .. figure:: images/dataaccess_er.png
    :alt: ER diagram
    :width: 90%
    :align: center

    **Picture - ER diagram**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.15\linewidth}|p{0.55\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 15 55

    * - 項番
      - カテゴリ
      - テーブル名
      - 説明
    * - | (1)
      - トランザクション系
      - t_order
      - 注文を保持するテーブル。１つの注文に対して、1レコードが格納される。
    * - | (2)
      -
      - t_order_item
      - １つの注文で購入された商品を保持するテーブル。1つの注文で、複数の商品が購入された場合は、商品数分レコードが格納される。
    * - | (3)
      -
      - t_order_coupon
      - １つの注文で使用されたクーポンを保持するテーブル。1つの注文で、複数のクーポンが使用された場合は、クーポン数分レコードが格納される。クーポンを使用しなかった場合は、レコードは格納されない。
    * - | (4)
      - マスタ系
      - m_item
      - 商品を定義するマスタテーブル。
    * - | (5)
      -
      - m_category
      - カテゴリを定義するマスタテーブル。
    * - | (6)
      -
      - m_item_category
      - 商品が所属するカテゴリを定義するマスタテーブル。商品とカテゴリのマッピングを保持している。1つの商品は、複数のカテゴリに属すことができるモデルとなっている。
    * - | (7)
      -
      - m_coupon
      - クーポンを定義するマスタテーブル。
    * - | (8)
      - コード系
      - c_order_status
      - 注文ステータスを定義するコードテーブル。


トランザクション系テーブルのレイアウトと、格納されているレコードは、以下の通りである。

 **t_order**

 .. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 20 20

    * - id(PK)
      - status_code
    * - 1
      - accepted
    * - 2
      - checking

|

 **t_order_item**

 .. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 20 20 20

    * - order_id(PK)
      - item_code(PK)
      - quantity
    * - 1
      - ITM0000001
      - 10
    * - 1
      - ITM0000002
      - 20
    * - 2
      - ITM0000001
      - 30
    * - 2
      - ITM0000002
      - 40


 **t_order_coupon**

 .. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 20 20

    * - order_id(PK)
      - coupon_code(PK)
    * - 1
      - CPN0000001
    * - 1
      - CPN0000002

|

マスタ系テーブルのレイアウトと、格納されているレコードは、以下の通りである。

 **m_item**

 .. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 20 20 20

    * - code(PK)
      - name
      - price
    * - ITM0000001
      - Orange juice
      - 100
    * - ITM0000002
      - NotePC
      - 100000

|

 **m_category**

 .. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 20 20

    * - code(PK)
      - name
    * - CTG0000001
      - Drink
    * - CTG0000002
      - PC
    * - CTG0000003
      - Hot selling

|

 **m_item_category**

 .. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 20 20

    * - item_code(PK)
      - category_code(PK)
    * - ITM0000001
      - CTG0000001
    * - ITM0000002
      - CTG0000002
    * - ITM0000002
      - CTG0000003

|

 **m_coupon**

 .. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 20 20 20

    * - code(PK)
      - name
      - price
    * - CPN0000001
      - Join coupon
      - 3000
    * - CPN0000002
      - PC coupon
      - 30000

|

コード系テーブルのレイアウトと、格納されているレコードは、以下の通りである。

 **c_order_status**

 .. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 20 20

    * - code(PK)
      - name
    * - accepted
      - Order accepted
    * - checking
      - Stock checking
    * - shipped
      - Item Shipped

|

以降で説明する実装例では、上記テーブルに格納されているデータを、以下のJavaBeanにマッピングして、取得する。

 .. figure:: images/dataaccess_entity.png
    :alt: Class(JavaBean) diagram
    :width: 90%
    :align: center

    **Picture - Class(JavaBean) diagram**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.65\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 15 65

    * - 項番
      - クラス名
      - 説明
    * - | (1)
      - Order
      - t_orderテーブルの1レコードを表現するJavaBean。 関連オブジェクトとして、\ ``OrderStatus``\ と\ ``OrderItem``\ および\ ``OrderCoupon``\ を複数保持する。
    * - | (2)
      - OrderItem
      - t_order_itemテーブルの1レコードを表現するJavaBean。 関連オブジェクトとして、\ ``Item``\ を保持する。
    * - | (3)
      - OrderCoupon
      - t_order_couponテーブルの1コードを表現するJavaBean。関連オブジェクトとして、\ ``Coupon``\ を保持する。
    * - | (4)
      - Item
      - m_itemテーブルの1コードを表現するJavaBean。 関連オブジェクトとして、所属している\ ``Category``\ を複数保持する。\ ``Item``\ と\ ``Category``\ の紐づけは、m_item_categoryテーブルによって行われる。
    * - | (5)
      - Category
      - m_categoryテーブルの1レコードを表現するJavaBean。
    * - | (6)
      - Coupon
      - m_couponテーブルの1レコードを表現するJavaBean。
    * - | (7)
      - OrderStatus
      - c_order_statusテーブルの1レコードを表現するJavaBean。


JavaBeanのプロパティ定義は、以下の通りである。

- Order.java

 .. code-block:: java

    public class Order implements Serializable {
        private int id;
        private List<OrderItem> orderItems;
        private List<OrderCoupon> orderCoupons;
        private OrderStatus status;
        // ...
    }

- OrderItem.java

 .. code-block:: java

    public class OrderItem implements Serializable {
        private int orderId;
        private String itemCode; // <!-- (1) -->
        private Item item;
        private int quantity;
        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - 保持する値が、直後の変数\ ``item``\ の\ ``code``\ プロパティと重複する。これは、後述するresultMap要素の、groupBy属性によるレコードの、グルーピングを行う際に必要になるため、定義している。

- OrderCoupon.java

 .. code-block:: java

    public class OrderCoupon implements Serializable {
        private int orderId;
        private String couponCode; // (1)
        private Coupon coupon;
        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - 保持する値が、直後の変数\ ``Coupon``\ の\ ``code``\ プロパティと重複する。これは、後述するresultMap要素の、groupBy属性によるレコードの、グルーピングを行う際に必要になるため、定義している。

- Item.java

 .. code-block:: java

    public class Item implements Serializable {
        private String code;
        private String name;
        private int price;
        private List<Category> categories;
        // ...
    }

- Category.java

 .. code-block:: java

    public class Category implements Serializable {
        private String code;
        private String name;
        // ...
    }

- Coupon.java

 .. code-block:: java

    public class Coupon implements Serializable {
        private String code;
        private String name;
        private int price;
        // ...
    }

- OrderStatus.java

 .. code-block:: java

    public class OrderStatus implements Serializable {
        private String code;
        private String name;
        // ...
    }


| SQLマッピングを実装する。
| 関連するオブジェクトを、1回のSQLでまとめて取得する場合、取得したいテーブルをJOINして、マッピングに必要なすべてのレコードを取得する。
| 取得したレコードは、resultMap要素にマッピング定義を行い、JavaBeanにマッピングする。

| 以下では、1件のOrderを取得するSQL(findOne)と、すべてのOrderを取得するSQL(findAll)の実装例となっている。
| なお、以下の実装例では、ネームスペースに\ ``"order"``\ を指定し、 マッピングするJavaBeanは、typeAliasを使って、パッケージ名を除いたクラス名が定義してある前提となっている。

- sqlMapConfig.xml

 .. code-block:: xml

    <typeAlias alias="Order" type="xxxxxx.yyyyyy.zzzzzz.domain.model.Order"/>
    <typeAlias alias="OrderStatus" type="xxxxxx.yyyyyy.zzzzzz.domain.model.OrderStatus"/>
    <typeAlias alias="OrderItem" type="xxxxxx.yyyyyy.zzzzzz.domain.model.OrderItem"/>
    <typeAlias alias="OrderCoupon" type="xxxxxx.yyyyyy.zzzzzz.domain.model.OrderCoupon"/>
    <typeAlias alias="Item" type="xxxxxx.yyyyyy.zzzzzz.domain.model.Item"/>
    <typeAlias alias="Category" type="xxxxxx.yyyyyy.zzzzzz.domain.model.Category"/>
    <typeAlias alias="Coupon" type="xxxxxx.yyyyyy.zzzzzz.domain.model.Coupon"/>

- order-sqlmap.xml

 .. code-block:: xml

    <sqlMap namespace="order">

        <!-- ... -->

    </sqlMap>

|

| まず、SQLについて説明する。
| 実際の定義は、resultMap要素の後に、定義すること。

SQLの実装

- findOne/findAll共通部分のSQL定義(sql要素)

 .. code-block:: xml

    <sql id="fragment_selectFormJoin">         <!-- (1) -->
        SELECT                                   /* (2) */
            o.id
            ,os.code AS status_code
            ,os.name AS status_name
            ,ol.quantity
            ,i.code AS item_code
            ,i.name AS item_name
            ,i.price AS item_price
            ,ct.code AS category_code
            ,ct.name AS category_name
            ,cp.code AS coupon_code
            ,cp.name AS coupon_name
            ,cp.price AS coupon_price
        FROM
            t_order o
        INNER JOIN                                /* (3) */
            c_order_status os
                ON os.code = o.status_code
        INNER JOIN
            t_orderline ol
                ON ol.order_id = o.id
        INNER JOIN
            m_item i
                ON i.code = ol.item_code
        INNER JOIN
            m_item_category ic
                ON ic.item_code = i.code
        INNER JOIN
            m_category ct
                ON ct.code = ic.category_code
        LEFT JOIN                                  /* (4) */
            t_order_coupon oc
                ON oc.order_id = o.id
        LEFT JOIN
            m_coupon cp
                ON cp.code = oc.coupon_code
    </sql>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - findOneと、findAllでSELECT句、FROM句、JOIN句を共有するためのsql要素。findOneとfindAllで、多くの共通部分があったので共通化している。
    * - | (2)
      - 関連オブジェクトを生成するために、必要なデータをすべて取得する。カラム名は、重複しないようにする必要がある。上記例では、\ ``code``\ , \ ``name``\ , \ ``price``\ が重複するため、AS句で別名を指定している。
    * - | (3)
      - 関連オブジェクトを生成するために、必要なデータが格納されているテーブルを結合する。
    * - | (4)
      - データが格納されない可能性のあるテーブルについては、外部結合とする。クーポンを使用しない場合、t_group_couponにレコードが格納されないので外部結合にする必要がある。t_group_couponと結合するt_couponも同様である。


- findOneのSQL定義

 .. code-block:: xml

    <select id="findOne" parameterClass="java.lang.Integer" resultMap="orderResultMap"> <!-- (1) -->
        <include refid="fragment_selectFormJoin"/> <!-- (2) -->
        WHERE
            o.id = #id#         /* (3) */
        ORDER BY                /* (4) */
            item_code ASC       /* (5) */
            ,category_code ASC  /* (6) */
            ,coupon_code ASC    /* (7) */
    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - 指定された注文IDの、\ ``Order``\ オブジェクトおよび関連オブジェクトを取得するためのSQL。
    * - | (2)
      - findAllと共有するSELECT句、FROM句、JOIN句が実装されたSQLを、インクルードしている。
    * - | (3)
      - バインド値で渡された注文IDを、WHERE句に指定する。
    * - | (4)
      - 1:Nの関係の関連オブジェクトがある場合は、リスト内の並び順を制御するための、ORDER BY句を指定する。並び順を意識する必要がない場合は、指定は不要である。
    * - | (5)
      - \ ``Order#orderItems``\ のリストを、t_itemテーブルのcodeカラムの昇順にするための指定。
    * - | (6)
      - \ ``Item#categories``\ のリストを、t_categoryテーブルのcodeカラムの昇順にするための指定。
    * - | (7)
      - \ ``Order#orderCoupons``\ のリストを、t_couponのcodeの昇順にするための指定。


- findAllのSQL定義

 .. code-block:: xml

    <select id="findAll" resultMap="orderResultMap"> <!-- (1) -->
        <include refid="fragment_selectFormJoin"/> <!-- (2) -->
        ORDER BY
            o.id DESC     /* (3) */
            ,i.code ASC
            ,ct.code ASC
            ,cp.code ASC
    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - すべてのOrder、および、関連オブジェクトを取得するためのSQL。
    * - | (2)
      - findOneとfindAllでSELECT句、FROM句、JOIN句を共有するためのsql要素。
    * - | (3)
      - 取得されるリストの並び順を、t_orderのidの降順にするための指定。


| 上記SQL(findAll)を実行した結果、以下のレコードが取得される。
| 注文レコードとしては2件だが、レコードが複数件格納される関連テーブルと結合しているため、合計で9レコードが取得される。
| 1～3行目は、注文IDが\ ``2``\ の \ ``Order``\ オブジェクトを生成するためのレコード、4～9行目は注文IDが\ ``1``\ の \ ``Order``\ オブジェクトを生成するためのレコードとなる。

 .. figure:: images/dataaccess_sql_result.png
    :alt: Result Set of findAll
    :width: 100%
    :align: center

    **Picture - Result Set of findAll**

|

上記レコードを、\ ``Order``\ オブジェクト、および、関連オブジェクトにマッピングする方法について説明する。

| resultMap要素の実装
| それぞれの説明については、後述する。


 .. code-block:: xml

    <resultMap id="orderResultMap" class="Order" groupBy="id">
        <result property="id" column="id" />
        <result property="status" resultMap="order.orderStatusResultMap" />
        <result property="orderItems" resultMap="order.orderItemResultMap" />
        <result property="orderCoupons" resultMap="order.orderCouponResultMap" />
    </resultMap>

    <resultMap id="orderStatusResultMap" class="OrderStatus" groupBy="code">
        <result property="code" column="status_code" />
        <result property="name" column="status_name" />
    </resultMap>

    <resultMap id="orderItemResultMap" class="OrderItem" groupBy="itemCode">
        <result property="itemCode" column="item_code" />
        <result property="item" resultMap="order.itemResultMap" />
        <result property="quantity" column="quantity" />
    </resultMap>

    <resultMap id="itemResultMap" class="Item" groupBy="code">
        <result property="code" column="item_code" />
        <result property="name" column="item_name" />
        <result property="price" column="item_price" />
        <result property="categories" resultMap="order.categoryResultMap" />
    </resultMap>

    <resultMap id="categoryResultMap" class="Category" groupBy="code">
        <result property="code" column="category_code" />
        <result property="name" column="category_name" />
    </resultMap>

    <resultMap id="orderCouponResultMap" class="OrderCoupon" groupBy="couponCode">
        <result property="couponCode" column="coupon_code" />
        <result property="coupon" resultMap="order.couponResultMap" />
    </resultMap>

    <resultMap id="couponResultMap" class="Coupon" groupBy="code">
        <result property="code" column="coupon_code" />
        <result property="name" column="coupon_name" />
        <result property="price" column="coupon_price" />
    </resultMap>

|

各resultMap要素の役割と依存関係を、以下に示す。

 .. figure:: images/dataaccess_resultmap.png
    :alt: Implementation of ResultMap
    :width: 100%
    :align: center

    **Picture - Implementation of ResultMap**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 取得したレコードを\ ``Order``\ オブジェクトにマッピングするための定義。
        | 関連オブジェクト(\ ``OrderStatus``\ , \ ``OrderItem``\ , \ ``OrderCoupon``\ )のマッピングは、別のresultMapに委譲している。
    * - | (2)
      - | 取得したレコードを、\ ``OrderStatus``\ オブジェクトにマッピングするための定義。
    * - | (3)
      - | 取得したレコードを、\ ``OrderItem``\ オブジェクトにマッピングするための定義。
        | 関連オブジェクト(\ ``Item``\ )のマッピングは別のresultMapに委譲している。
    * - | (4)
      - | 取得したレコードを、\ ``Item``\ オブジェクトにマッピングするための定義。
        | 関連オブジェクト(\ ``Category``\ )のマッピングは、別のresultMapに委譲している。
    * - | (5)
      - | 取得したレコードを、\ ``Category``\ オブジェクトにマッピングするための定義。
    * - | (6)
      - | 取得したレコードを、\ ``OrderCoupon``\ オブジェクトにマッピングするための定義。
        | 関連オブジェクト(\ ``Coupon``\ )のマッピングは、別のresultMapに委譲している。
    * - | (7)
      - | 取得したレコードを、\ ``Coupon``\ オブジェクトにマッピングするための定義。


\ ``Order``\ オブジェクトへのマッピングを行う。

 .. code-block:: xml

    <resultMap id="orderResultMap" class="Order" groupBy="id"> <!-- (1) -->
        <result property="id" column="id" /> <!-- (2) -->
        <result property="status" resultMap="order.orderStatusResultMap" /> <!-- (3) -->
        <result property="orderItems" resultMap="order.orderItemResultMap" /> <!-- (4) -->
        <result property="orderCoupons" resultMap="order.orderCouponResultMap" /> <!-- (5) -->
    </resultMap>

 .. figure:: images/dataaccess_resultmap_order.png
    :alt: ResultMap for Order
    :width: 100%
    :align: center

    **Picture - ResultMap for Order**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 取得したレコードは、注文毎にグループ化する必要があるため、注文を一意に識別するための値が格納されている\ ``id``\ プロパティを、groupBy属性に指定する。
        | 本例では、\ ``id``\ プロパティでグループ化されるため、\ ``id=1``\ と\ ``id=2``\ の２つの\ ``Order``\ オブジェクトが、生成される。
    * - | (2)
      - | 取得したレコードの\ ``id``\ カラムの値を、\ ``Order#id``\ に設定する。
    * - | (3)
      - | \ ``OrderStatus``\ オブジェクトの生成を、\ ``id="order.orderStatusResultMap"``\ のresultMapに委譲し、生成されたオブジェクトを、\ ``Order#status``\ に設定する。
    * - | (4)
      - | \ ``OrderItem``\ オブジェクトの生成を、\ ``id="order.orderItemResultMap"``\ のresultMapに委譲し、生成されたオブジェクトを、\ ``Order#orderItems``\ のリストに追加する。
    * - | (5)
      - \ ``OrderCoupon``\ オブジェクトの生成を、\ ``id="order.orderCouponResultMap"``\ のresultMapに委譲し、生成されたオブジェクトを、\ ``Order#orderCoupons``\ のリストに追加する。


以降では、\ ``id=1``\ の\ ``Order``\ オブジェクトへのマッピングに、焦点を当てて説明する。


\ ``OrderStatus``\ オブジェクトへのマッピングを行う。

 .. code-block:: xml

    <resultMap id="orderStatusResultMap" class="OrderStatus"> <!-- (1) -->
        <result property="code" column="status_code" /> <!-- (2) -->
        <result property="name" column="status_name" /> <!-- (3) -->
    </resultMap>

 .. figure:: images/dataaccess_resultmap_orderstatus.png
    :alt: ResultMap for OrderStatus
    :width: 100%
    :align: center

    **Picture - ResultMap for OrderStatus**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \  ``Order``\ と、\ ``OrderStatus``\ オブジェクトは、1:1の関係なので、groupBy属性の指定は不要である。
        | 本例では、\ ``code=accepted``\ の\ ``OrderStatus``\ オブジェクトが生成される。
    * - | (2)
      - | 取得したレコードの、\ ``status_code``\ カラムの値を、\ ``OrderStatus#code``\ に設定する。
    * - | (3)
      - | 取得したレコードの、\ ``status_name``\ カラムの値を、\ ``OrderStatus#name``\ に設定する。


\ ``OrderItem``\ オブジェクトへのマッピングを行う。

 .. code-block:: xml

    <resultMap id="orderItemResultMap" class="OrderItem" groupBy="itemCode"> <!-- (1) -->
        <result property="itemCode" column="item_code" /> <!-- (2) -->
        <result property="item" resultMap="order.itemResultMap" /> <!-- (3) -->
        <result property="quantity" column="quantity" /> <!-- (4) -->
    </resultMap>

 .. figure:: images/dataaccess_resultmap_orderitem.png
    :alt: ResultMap for OrderItem
    :width: 100%
    :align: center

    **Picture - ResultMap for OrderItem**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``Order``\ と\ ``OrderItem``\ は、1:Nの関係なので、groupBy属性の指定が必要である。
        | 注文商品は、t_order_itemのプライマリキー(order_id,item_code)でグループ化する必要があるが、order_idカラムについては、親のresultMapで指定されているため、ここでは、item_codeカラムの値を保持する\ ``itemCode``\ プロパティのみ指定する。
        | 本例では、\ ``itemCode``\ プロパティでグループ化されるため、\ ``itemCode=ITM0000001``\ と\ ``itemCode=ITM0000002``\ の、２つの\ ``OrderItem``\ オブジェクトが生成される。
    * - | (2)
      - | 取得したレコードの\ ``item_code``\ カラムの値を、\ ``OrderItem#itemCode``\ に設定する。
        | (3)で生成される\ ``Item#code``\ と重複するが、\ ``itemCode``\ プロパティは、\ ``OrderItem``\ をグループ化するために必要なプロパティとなる。
    * - | (3)
      - | \ ``Item``\ オブジェクトの生成を、\ ``id="order.itemResultMap"``\ のresultMapに委譲し、生成されたオブジェクトを\ ``OrderItem#item``\ に設定する。
    * - | (4)
      - | 取得したレコードの\ ``quantity``\ カラムの値を、\ ``OrderItem#quantity``\ に設定する。


\ ``Item``\ オブジェクトへのマッピングを行う。

 .. code-block:: xml

    <resultMap id="itemResultMap" class="Item" groupBy="code"> <!-- (1) -->
        <result property="code" column="item_code" /> <!-- (2) -->
        <result property="name" column="item_name" /> <!-- (3) -->
        <result property="price" column="item_price" /> <!-- (4) -->
        <result property="categories" resultMap="order.categoryResultMap" /> <!-- (5) -->
    </resultMap>

 .. figure:: images/dataaccess_resultmap_item.png
    :alt: ResultMap for Item
    :width: 100%
    :align: center

    **Picture - ResultMap for Item**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \  ``OrderItem``\ と\ ``Item``\ オブジェクトは、1:1の関係だが、\ ``Item``\ と\ ``Category``\は、1:Nの関係なので、groupBy属性の指定が必要である。
        | カテゴリは商品毎にグループ化する必要があるため、商品を一意に識別するための値が格納されている\ ``code``\ プロパティを、groupBy属性に指定する。
        | 本例では、\ ``OrderItem#itemCode=ITM0000001``\ 用に、\  ``code=ITM0000001``\ の\ ``Item``\ オブジェクトが、\ ``OrderItem#itemCode=ITM0000002``\ 用に、\ ``code=ITM0000002``\ の\ ``Item``\ オブジェクトが生成される。(計２つのオブジェクトが生成される。)
    * - | (2)
      - | 取得したレコードの\ ``item_code``\ カラムの値を、\ ``Item#code``\ に設定する。
    * - | (3)
      - | 取得したレコードの\ ``item_name``\ カラムの値を、\ ``Item#name``\ に設定する。
    * - | (4)
      - | 取得したレコードの\ ``item_price``\ カラムの値を、\ ``Item#price``\ に設定する。
    * - | (5)
      - | \ ``Category``\ オブジェクトの生成を、\ ``id="order.categoryResultMap"``\ のresultMapに委譲し、生成されたオブジェクトを ``Item#categories`` のリストに追加する。


\ ``Category``\ オブジェクトへのマッピングを行う。

 .. code-block:: xml

    <resultMap id="categoryResultMap" class="Category" groupBy="code"> <!-- (1) -->
        <result property="code" column="category_code" /> <!-- (1) -->
        <result property="name" column="category_name" /> <!-- (1) -->
    </resultMap>

 .. figure:: images/dataaccess_resultmap_category.png
    :alt: ResultMap for Item
    :width: 100%
    :align: center

    **Picture - ResultMap for Item**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 本例では、1:Nの関係のテーブル(t_orderとt_order_line、t_orderとt_order_coupon)を複数結合しているため、 t_order_couponに複数レコードが格納されていると \ ``Item``\ オブジェクト内に保持する\ ``Category``\ オブジェクトのリストが、重複してしまう。
        | 重複をなくすために、カテゴリを一意に識別するための値が格納されている\ ``code``\ プロパティを、groupBy属性に指定する。\ ``code``\ プロパティの値が、同じ\ ``Category``\ オブジェクトが一つにマージされ、重複をなくすことができる。
        | 本例では、\ ``Item#code=ITM0000001``\ 用に、\ ``code=CTG0000001``\ の\ ``Category``\ オブジェクトが、\ ``Item#code=ITM0000002``\ 用に、\ ``code=CTG0000002``\ と、\ ``code=CTG0000003``\ の2つの\ ``Category``\ オブジェクトが生成される。(計3つのオブジェクトが生成される。)
    * - | (2)
      - | 取得したレコードの \ ``item_code``\ カラムの値を、\ ``Item#code``\ に設定する。
    * - | (3)
      - | 取得したレコードの \ ``item_name``\ カラムの値を、\ ``Item#name``\ に設定する。


\ ``OrderCoupon``\ オブジェクトへのマッピングを行う。

 .. code-block:: xml

    <resultMap id="orderCouponResultMap" class="OrderCoupon" groupBy="couponCode"> <!-- (1) -->
        <result property="couponCode" column="coupon_code" /> <!-- (2) -->
        <result property="coupon" resultMap="order.couponResultMap" /> <!-- (3) -->
    </resultMap>

 .. figure:: images/dataaccess_resultmap_ordercoupon.png
    :alt: ResultMap for OrderCoupon
    :width: 100%
    :align: center

    **Picture - ResultMap for OrderCoupon**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \  ``Order``\ と\ ``OrderCoupon``\ は、1:Nの関係なので、groupBy属性の指定が必要である。
        | 注文クーポンは、t_order_couponのプライマリキー(order_id,coupon_code)でグループ化する必要があるが、order_idカラムについては親のresultMapで指定されているため、ここでは、coupon_codeカラムの値を保持する\ ``couponCode``\ プロパティのみ指定する。
        | 本例では、\ ``couponCode``\ プロパティでグループ化されるため、\ ``couponCode=CPN0000001``\ と\ ``couponCode=CPN0000002``\  、の2つの\  ``OrderCoupon``\ オブジェクトが生成される。
    * - | (2)
      - | 取得したレコードの\ ``coupon_code``\ カラムの値を、\ ``OrderCoupon#couponCode``\ に設定する。
        | (3)で生成される\ ``Coupon#code``\ と重複するが、\ ``couponCode``\ プロパティは、\ ``OrderCoupon``\ をグループ化するために必要なプロパティとなる。
    * - | (3)
      - | \ ``Coupon``\ オブジェクトの生成を\ ``id="order.couponResultMap"``\ のresultMapに委譲し、生成されたオブジェクトを\ ``OrderCoupon#coupon``\ に設定する。


\ ``Coupon``\ オブジェクトへのマッピングを行う。

 .. code-block:: xml

    <resultMap id="couponResultMap" class="Coupon"> <!-- (1) -->
        <result property="code" column="coupon_code" /> <!-- (2) -->
        <result property="name" column="coupon_name" /> <!-- (3) -->
        <result property="price" column="coupon_price" /> <!-- (4) -->
    </resultMap>

 .. figure:: images/dataaccess_resultmap_coupon.png
    :alt: ResultMap for Coupon
    :width: 100%
    :align: center

    **Picture - ResultMap for Coupon**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``OrderCoupon``\ と\ ``Coupon``\ オブジェクトは、1:1の関係なので、groupBy属性の指定が不要である。
        | 本例では、\ ``OrderCoupon#couponCode=CPN0000001``\ 用に、\ ``code=CPN0000001``\ の\ ``Coupon``\ オブジェクトが、\ ``OrderCoupon#couponCode=CPN0000001``\ 用に、\ ``code=CPN0000001``\ の\ ``Coupon``\ オブジェクトが生成される。(計２つのオブジェクトが生成される。)
    * - | (2)
      - | 取得したレコードの\ ``coupon_code``\ カラムの値を、\ ``Coupon#code``\ に設定する。
    * - | (3)
      - | 取得したレコードの\ ``coupon_name``\ カラムの値を、\ ``Coupon#name``\ に設定する。
    * - | (4)
      - | 取得したレコードの\ ``coupon_price``\ カラムの値を、\ ``Coupon#price``\ に設定する。


| JavaBeanにマッピングされたレコードとカラムは、以下の通りである。
| グレーアウトしている部分は、groupBy属性に指定によって、グレーアウトされていない部分にマージされる。

 .. figure:: images/dataaccess_sql_result_used.png
    :alt: Valid Result Set for result mapping
    :width: 100%
    :align: center

    **Picture - Valid Result Set for result mapping**


.. _data-access-mybatis2_warning_sqlmapping_bulk:

 .. warning::

     1:Nの関連をもつレコードをJOINしてマッピングする場合、グレーアウトされている部分のデータの取得が無駄になる点を、意識しておくこと。

     Nの部分のデータを使用しない処理で、同じSQLを使用した場合、さらに無駄なデータの取得となってしまうので、Nの部分を取得するSQLと、取得しないSQLを、別々に用意しておくなどの工夫を行うこと。


実際にマッピングされた\ ``Order``\ オブジェクトおよび関連オブジェクトの状態は、以下の通りである。

 .. figure:: images/dataaccess_object.png
    :alt: Mapped object diagram
    :width: 90%
    :align: center

    **Picture - Mapped object diagram**

 .. tip::

     関連オブジェクトを取得する別の方法として、取得したレコードの値を使って、内部で別のSQLを実行して、取得する方法がある。
     内部で別のSQLを実行する方法は、個々のSQLや、resultMap要素の定義が、非常にシンプルとなる。
     ただし、この方法で取得する場合は、N+1問題を引き起こす要因となることを、意識しておく必要がある。

     内部で別のSQLを実行する方法については、Mybatis Developer Guide(PDF)の「Result Maps/Complex Properties」(P.36-37)および「Result Maps/Composite Keys or Multiple Complex Parameters Properties」(P.40-41)を参照されたい。


 .. tip::

     内部で別のSQLを実行する方法を使う場合、関連オブジェクトは "Eager Load" されるため、関連オブジェクトを使用しない場合も、SQLが実行されてしまう。
     この動作回避する方法として、Mybatisでは、関連オブジェクトを "Lazy Load" する方法を、オプションとして提供している。

     "Lazy Load"を有効にするための設定は、以下の通りである。

     * Mybatis設定ファイルのsetting要素のenhancementEnabled属性を、\ ``true``\ に設定する。
     * CGLIB 2.xを、クラスパスに追加する。

.. raw:: latex

   \newpage

