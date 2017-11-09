データベースアクセス（共通編）
================================================================================

.. only:: html

 .. contents:: 目次
    :local:
    :depth: 3


.. todo::

    **TBD**

    本章では以下の内容について、現在精査中である。

    * | 複数データソースについて
      | 具体的な内容は、\ :ref:`Overviewの複数データソースについて <data-access-common_todo_multiple_datasource_overview>`\ および\ :ref:`How to extendsの複数データソースを使用するための設定 <data-access-common_todo_multiple_datasource_howtoextends>`\ を参照されたい。
    * | JPA利用時の一意制約エラー及び悲観排他エラーのハンドリング方法について
      | 具体的な内容は、\ :ref:`Overviewの例外ハンドリングについて <data-access-common_todo_exception>`\ を参照されたい。

.. _data_access_overview-label:

Overview
--------------------------------------------------------------------------------

本節では、RDBMSで管理されているデータにアクセスする方法について、説明する。

O/R Mapperに依存する部分については、

* \ :doc:`DataAccessMyBatis3`\
* \ :doc:`DataAccessJpa`\

を参照されたい。


JDBC DataSourceについて
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| RDBMSにアクセスする場合、アプリケーションからは、JDBCデータソースを参照してアクセスすることになる。
| JDBCデータソースを使用することにより、JDBCドライバーのロード、接続情報（接続URL、接続ユーザ、パスワードなど）の設定を、アプリケーションから排除することができる。
| そのため、アプリケーションからは、使用するRDBMSやデプロイする環境を、意識する必要がなくなる。

 .. figure:: images/dataaccess_common-datasource.png
    :alt: about data source
    :width: 90%
    :align: center

    **Picture - About JDBC DataSource**

| JDBCデータソースの実装は、アプリケーションサーバ、OSSライブラリ、Third-Partyライブラリ、Spring Frameworkなどから提供されているので、プロジェクト要件や、デプロイ環境にあったデータソースの選定が必要になる。
| 以下に、代表的なデータソース3種類の紹介を行う。

 * :ref:`datasource_application_server-label`
 * :ref:`datasource_oss_thirdparty-label`
 * :ref:`datasource_spring_framework-label`


.. _datasource_application_server-label:

アプリケーションサーバ提供のJDBCデータソース
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Webアプリケーションでデータソースを使用する場合、アプリケーションサーバから提供されるJDBCデータソースを使うのが一般的である。
| アプリケーションサーバから提供されるJDBCデータソースは、コネクションプーリング機能など、Webアプリケーションで使うために必要な機能が、標準で提供されている。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table:: **アプリケーションサーバから提供されているデータソース**
    :header-rows: 1
    :widths: 10 35 55

    * - 項番
      - アプリケーションサーバ
      - 参照ページ
    * - 1.
      - Apache Tomcat 8.5
      - | \ `Apache Tomcat 8.5 User Guide(The Tomcat JDBC Connection Pool) <http://tomcat.apache.org/tomcat-8.5-doc/jdbc-pool.html>`_\ を参照されたい。
        | \ `Apache Tomcat 8.5 User Guide(JNDI Datasource HOW-TO) <http://tomcat.apache.org/tomcat-8.5-doc/jndi-datasource-examples-howto.html>`_\ (Apache Commons DBCP 2)を参照されたい。
    * - 2.
      - Apache Tomcat 8.0
      - | \ `Apache Tomcat 8.0 User Guide(The Tomcat JDBC Connection Pool) <http://tomcat.apache.org/tomcat-8.0-doc/jdbc-pool.html>`_\ を参照されたい。
        | \ `Apache Tomcat 8.0 User Guide(JNDI Datasource HOW-TO) <http://tomcat.apache.org/tomcat-8.0-doc/jndi-datasource-examples-howto.html>`_\ (Apache Commons DBCP 2)を参照されたい。
    * - 3.
      - Apache Tomcat 7
      - | \ `Apache Tomcat 7 User Guide(The Tomcat JDBC Connection Pool) <http://tomcat.apache.org/tomcat-7.0-doc/jdbc-pool.html>`_\ を参照されたい。
        | \ `Apache Tomcat 7 User Guide(JNDI Datasource HOW-TO) <http://tomcat.apache.org/tomcat-7.0-doc/jndi-datasource-examples-howto.html>`_\ (Apache Commons DBCP)を参照されたい。
    * - 4.
      - Oracle WebLogic Server 12c
      - \ `Oracle WebLogic Server Product Documentation <http://docs.oracle.com/middleware/1221/wls/INTRO/jdbc.htm>`_\ を参照されたい。
    * - 5.
      - IBM WebSphere Application Server Version 9.0
      - \ `WebSphere Application Server Online information center <http://www.ibm.com/support/knowledgecenter/ja/SSEQTP_9.0.0/com.ibm.websphere.wlp.doc/ae/twlp_dep_configuring_ds.html>`_\ を参照されたい。
    * - 6.
      - JBoss Enterprise Application Platform 7.0
      - \ `JBoss Enterprise Application Platform 7.0 Product Documentation <https://access.redhat.com/documentation/en/red-hat-jboss-enterprise-application-platform/7.0/paged/configuration-guide/chapter-13-datasource-management>`_\ を参照されたい。
    * - 7.
      - JBoss Enterprise Application Platform 6.4
      - \ `JBoss Enterprise Application Platform 6.4 Product Documentation <https://access.redhat.com/documentation/en-US/JBoss_Enterprise_Application_Platform/6.4/html/Administration_and_Configuration_Guide/chap-Datasource_Management.html>`_\ を参照されたい。


.. _datasource_oss_thirdparty-label:

OSS/Third-Partyライブラリ提供のJDBCデータソース
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| アプリケーションサーバから提供されるJDBCデータソースを使わない場合は、OSS/Third-Partyライブラリから提供されているJDBCデータソースを使用する。
| 本ガイドラインでは、「Apache Commons DBCP」のみ紹介するが、他のライブラリを使ってもよい。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table:: **OSS/Third-Partyライブラリから提供されているJDBCデータソース**
    :header-rows: 1
    :widths: 10 35 55

    * - 項番
      - ライブラリ名
      - 説明
    * - 1.
      - Apache Commons DBCP
      - \ `Apache Commons DBCP <http://commons.apache.org/proper/commons-dbcp/index.html>`_\ を参照されたい。


.. _datasource_spring_framework-label:

Spring Framework提供のJDBCデータソース
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Spring Frameworkから提供されているJDBCデータソースの実装クラスは、コネクションプーリング機能がないため、Webアプリケーションのデータソースとして使用する事はない。
| Spring Frameworkでは、JDBCデータソースの実装クラスと、JDBCデータソースのアダプタクラスを提供しているが、利用するケースが限定的なので、Appendixの\ :ref:`appendix_datasource_of_spring-label`\ として紹介する。


トランザクションの管理方法について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Spring Frameworkの機能を使って、トランザクション管理を行う場合、プロジェクト要件や、デプロイ環境にあったPlatformTransactionManagerの選定が必要になる。
| 詳細は、\ :doc:`../../ImplementationAtEachLayer/DomainLayer`\ の\ :ref:`service_enable_transaction_management`\ を参照されたい。


トランザクション境界/属性の宣言について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| トランザクション境界及びトランザクション属性の宣言は、Serviceにて、\ ``@Transactional``\ アノテーションを指定することで実現する。
| 詳細は、\ :doc:`../../ImplementationAtEachLayer/DomainLayer`\ の\ :ref:`service_transaction_management`\ を参照されたい。


データの排他制御について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| データを更新する場合、データの一貫性および整合性を保障するために、排他制御を行う必要がある。
| データの排他制御については、\ :doc:`ExclusionControl`\ を参照されたい。


例外ハンドリングについて
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Spring Frameworkでは、JDBCの例外(\ ``java.sql.SQLException``\ )や、O/R Mapper固有の例外を、Spring Frameworkから提供しているデータアクセス例外(\ ``org.springframework.dao.DataAccessException``\ のサブクラス)に変換する機能がある。
| Spring Frameworkのデータアクセス例外へ変換しているクラスについては、Appendixの\ :ref:`appendix_dataaccessexception_converter_class-label`\ を参照されたい。

| 変換されたデータアクセス例外は、基本的にはアプリケーションコードでハンドリングする必要はないが、一部のエラー（一意制約違反、排他エラーなど）については、要件によっては、ハンドリングする必要がある。
| データアクセス例外をハンドリングする場合、\ ``DataAccessException``\ をcatchするのではなく、エラー内容を通知するサブクラスの例外をcatchすること。
| 以下に、アプリケーションコードでハンドリングする可能性がある代表的なサブクラスを紹介する。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table:: **ハンドリングする可能性があるDBアクセス例外のサブクラス**
    :header-rows: 1
    :widths: 10 35 55

    * - 項番
      - クラス名
      - 説明
    * - 1.
      - | org.springframework.dao.
        | DuplicateKeyException
      - | 一意制約違反が発生した場合に発生する例外。
    * - 2.
      - | org.springframework.dao.
        | OptimisticLockingFailureException
      - | 楽観ロックに成功しなかった場合に発生する例外。他の処理によって同一データが更新されていた場合に発生する。
        | 本例外は、O/R MapperとしてJPAを使用する場合に発生する例外である。MyBatisには楽観ロックを行う機能がないため、O/R Mapper本体から本例外が発生することはない。
    * - 3.
      - | org.springframework.dao.
        | PessimisticLockingFailureException
      - | 悲観ロックに成功しなかった場合に発生する例外。他の処理で同一データがロックされており、ロック解放待ちのタイムアウト時間を超えてもロックが解放されない場合に発生する。

 .. note::

    O/R MapperにMyBatisを使用して楽観ロックを実現する場合は、ServiceやRepositoryの処理として楽観ロック処理を実装する必要がある。

    本ガイドラインでは、楽観ロックに失敗したことを、Controllerに通知する方法として、\ ``OptimisticLockingFailureException``\ およびその子クラスの例外を発生させることを推奨する。

    理由は、アプリケーション層の実装(Controllerの実装)を、使用するO/R Mapperに依存させないためである。


.. _data-access-common_todo_exception:

 .. todo::

    **JPA(Hibernate)を使用すると、現状意図しないエラーとなることが発覚している。**

    * 一意制約違反が発生した場合、\ ``DuplicateKeyException``\ ではなく、\ ``org.springframework.dao.DataIntegrityViolationException``\ が発生する。


下記は、一意制約違反を、ビジネス例外として扱う実装例である。

 .. code-block:: java

     try {
         accountRepository.saveAndFlash(account);
     } catch(DuplicateKeyException e) { // (1)
         throw new BusinessException(ResultMessages.error().add("e.xx.xx.0002"), e); // (2)
     }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 一意制約違反が発生した場合に発生する例外（DuplicateKeyException）をcatchする。
    * - | (2)
      - | データが重複している旨を伝えるビジネス例外を発生させている。
        | 例外をcatchした場合は、必ず原因例外(\ ``e``\ ) をビジネス例外に指定すること。

複数データソースについて
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| アプリケーションによっては、複数のデータソースが必要になる場合がある。
| 以下に、複数のデータソースが必要になる代表的なケースを紹介する。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.30\linewidth}|p{0.30\linewidth}|
 .. list-table:: **複数のデータソースが必要になるう代表的なケース**
    :header-rows: 1
    :widths: 10 30 30 30

    * - 項番
      - ケース
      - 例
      - 特徴
    * - 1.
      - データ(テーブル)の分類毎にデータベースやスキーマがわかれている場合。
      - 顧客情報を保持するテーブル群と請求情報を保持するテーブル群が別々のデータベースやスキーマに格納されている場合など。
      - 処理で扱うデータは決まっているので、静的に使用するデータソースを決定することができる。
    * - 2.
      - 利用者（ログインユーザ）によって使用するデータベースやスキーマが分かれている場合。
      - 利用者の分類毎にデータベースやスキーマがわかれている場合など（マルチテナント等）。
      - 利用者によって使用するデータソースが異なるため、動的に使用するデータソースを決定する必要がある。

 .. _data-access-common_todo_multiple_datasource_overview:

 .. todo::

    **TBD**

    今後、以下の内容を追加する予定である。

    * 概念レベルのイメージ図


共通ライブラリから提供しているクラスについて
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 共通ライブラリから、以下の処理を行うクラスを提供している。
| 共通ライブラリの詳細はについては、以下を参照されたい。

* :ref:`data-access-common_appendix_like_escape`
* :ref:`data-access-common_appendix_sequencer`

|

How to use
--------------------------------------------------------------------------------

.. _data-access-common_howtouse_datasource:

データソースの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

アプリケーションサーバで定義したDataSourceを使用する場合の設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| アプリケーションサーバで定義したデータソースを使用する場合は、Bean定義ファイルに、JNDI経由で取得したオブジェクトを、beanとして登録するための設定を行う必要がある。
| 以下に、データベースはPostgreSQL、アプリケーションサーバはTomcat7を使用する際の、設定例を示す。

- :file:`xxx-context.xml` (Tomcatの設定ファイル)

  .. code-block:: xml

    <!-- (1) -->
    <Resource
       type="javax.sql.DataSource"
       name="jdbc/SampleDataSource"
       driverClassName="org.postgresql.Driver"
       url="jdbc:postgresql://localhost:5432/terasoluna"
       username="postgres"
       password="postgres"
       defaultAutoCommit="false"
       /> <!-- (2) -->

- :file:`xxx-env.xml`

 .. code-block:: xml

    <jee:jndi-lookup id="dataSource" jndi-name="jdbc/SampleDataSource" /> <!-- (3) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 10 80

    * - 項番
      - 属性名
      - 説明
    * - | (1)
      - \-
      - データソースを定義する。
    * - |
      - type
      - リソースの種類を指定する。\ ``javax.sql.DataSource``\ を指定する。
    * - |
      - name
      - リソース名を指定する。ここで指定した名前がJNDI名となる。
    * - |
      - driverClassName
      - JDBCドライバクラスを指定する。例では、PostgreSQLから提供されているJDBCドライバクラスを指定する。
    * - |
      - url
      - 接続URLを指定する。 【環境に合わせて変更が必要】
    * - |
      - username
      - 接続ユーザ名を指定する。【環境に合わせて変更が必要】
    * - |
      - password
      - 接続ユーザのパスワードを指定する。【環境に合わせて変更が必要】
    * - |
      - defaultAutoCommit
      - 自動コミットフラグのデフォルト値を指定する。falseを指定する。トランザクション管理下であれば強制的にfalseになる。
    * - | (2)
      - \-
      - | Tomcat7の場合、factory属性を省略するとtomcat-jdbc-poolが使用される。
        | 設定項目の詳細については、\ `Attributes of The Tomcat JDBC Connection Pool <http://tomcat.apache.org/tomcat-7.0-doc/jdbc-pool.html#Attributes>`_\ を参照されたい。
    * - | (3)
      - \-
      - データソースのJNDI名を指定する。Tomcatの場合は、データソース定義時のリソース名「(1)-name」に指定した値を指定する。


Bean定義したDataSouceを使用する場合の設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| アプリケーションサーバから提供されているデータソースを使わずに、
| OSS/Third-Partyライブラリから提供されているデータソースや、Spring Frameworkから提供されているJDBCデータソースを使用する場合は、Bean定義ファイルにDataSourceクラスのbean定義が必要となる。
| 以下に、データベースはPostgreSQL、データソースはApache Commons DBCPを使用する際の、設定例を示す。

- :file:`xxx-env.xml`

 .. code-block:: xml

    <bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource"
        destroy-method="close">                                           <!-- (1) (8) -->
        <property name="driverClassName" value="org.postgresql.Driver" /> <!-- (2) -->
        <property name="url" value="jdbc:postgresql://localhost:5432/terasoluna" /> <!-- (3) -->
        <property name="username" value="postgres" />                     <!-- (4) -->
        <property name="password" value="postgres" />                     <!-- (5) -->
        <property name="defaultAutoCommit" value="false"/>               <!-- (6) -->
        <!-- (7) -->
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - データソースの実装クラスを指定する。例では、Apache Commons DBCPから提供されているデータソースクラス(\ ``org.apache.commons.dbcp2.BasicDataSource``\ )を指定する。
    * - | (2)
      - JDBCドライバクラスを指定する。例では、PostgreSQLから提供されているJDBCドライバクラスを指定する。
    * - | (3)
      - 接続URLを指定する。 【環境に合わせて変更が必要】
    * - | (4)
      - 接続ユーザ名を指定する。【環境に合わせて変更が必要】
    * - | (5)
      - 接続ユーザのパスワードを指定する。【環境に合わせて変更が必要】
    * - | (6)
      - 自動コミットフラグのデフォルト値を指定する。falseを指定する。トランザクション管理下であれば、強制的にfalseになる。
    * - | (7)
      - | BasicDataSourceには上記以外に、JDBC共通の設定値の指定、JDBCドライバー固有のプロパティ値の指定、コネクションプーリング機能の設定値の指定を行うことができる。
        | 設定項目の詳細については、\ `DBCP Configuration <http://commons.apache.org/proper/commons-dbcp/configuration.html>`_\ を参照されたい。
    * - | (8)
      - | 設定例では値を直接指定しているが、環境によって設定値がかわる項目については、Placeholder(${...})を使用して、実際の設定値はプロパティファイルに指定すること。
        | Placeholderについては、\ `Spring Reference Document <http://docs.spring.io/spring/docs/4.3.5.RELEASE/spring-framework-reference/html/beans.html#beans-factory-extension-factory-postprocessors>`_\ の\  ``PropertyPlaceholderConfigurer``\ を参照されたい。


トランザクション管理を有効化するための設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
トランザクション管理を有効化するための基本的な設定は、\ :doc:`../../ImplementationAtEachLayer/DomainLayer`\ の\ :ref:`service_enable_transaction_management`\ を参照されたい。

PlatformTransactionManagerについては、使用するO/R Mapperによって使うクラスがかわるので、詳細設定は、

* \ :doc:`DataAccessMyBatis3`\
* \ :doc:`DataAccessJpa`\

を参照されたい。

.. _DataAccessCommonDataSourceDebug:

JDBCのDebug用ログの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| O/R Mapper(MyBatis, Hibernate)で出力されるログより、さらに細かい情報が必要な場合、log4jdbc(log4jdbc-remix)を使って出力される情報が有効である。
| log4jdbcの詳細については、\ `log4jdbc project page <https://code.google.com/p/log4jdbc/>`_\ を参照されたい。
| log4jdbc-remixの詳細については、\ `log4jdbc-remix project page <https://code.google.com/p/log4jdbc-remix/>`_\ を参照されたい。

\

 .. warning::

    **log4jdbc-remixが提供しているLog4jdbcProxyDataSourceを使用していると、ログレベルを"debug"以外に設定しても、オーバーヘッドが少なからず発生する。**
    **そのため、本設定はデバッグ用として使用し、性能試験及び商用環境にリリースする場合はLog4jdbcProxyDataSourceを経由せずにデータベースへ接続することを推奨する。**


log4jdbc提供のデータソースの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

- :file:`xxx-env.xml`

 .. code-block:: xml

    <jee:jndi-lookup id="dataSourceSpied" jndi-name="jdbc/SampleDataSource" /> <!-- (1) -->

    <bean id="dataSource" class="net.sf.log4jdbc.Log4jdbcProxyDataSource"> <!-- (2) -->
        <constructor-arg ref="dataSourceSpied" /> <!-- (3) -->
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - データソースの実体を定義する。例では、アプリケーションサーバからJNDI経由で取得したデータソースを使用している。
    * - | (2)
      - log4jdbcより提供されている\ ``net.sf.log4jdbc.Log4jdbcProxyDataSource``\ を指定する。
    * - | (3)
      - データソースの実体となるbeanを、コンストラクタに指定する。

 .. warning::

    **性能試験及び商用環境にリリースする場合、データソースとしてLog4jdbcProxyDataSourceは使用しないこと。**

    具体的には、(2)と(3)の設定を外し、\ ``"dataSourceSpied"``\ のbean名を\ ``"dataSource"``\ に変更する。


log4jdbc用ロガーの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

- :file:`logback.xml`

 .. code-block:: xml

    <!-- (1) -->
    <logger name="jdbc.sqltiming">
        <level value="debug" />
    </logger>

    <!-- (2) -->
    <logger name="jdbc.sqlonly">
        <level value="warn" />
    </logger>

    <!-- (3) -->
    <logger name="jdbc.audit">
        <level value="warn" />
    </logger>

    <!-- (4) -->
    <logger name="jdbc.connection">
        <level value="warn" />
    </logger>

    <!-- (5) -->
    <logger name="jdbc.resultset">
        <level value="warn" />
    </logger>

    <!-- (6) -->
    <logger name="jdbc.resultsettable">
        <level value="debug" />
    </logger>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | バインド変数に値が設定された状態のSQL文と、SQLの実行時間を出力するためのロガー。値がバインドされた形式のSQLが出力されるので、DBアクセスツールに貼りつけて実行する事ができる。
    * - | (2)
      - | バインド変数に値が設定された状態のSQL文を、出力するためのロガー。(1)との違いは、実行時間が出力されない。
    * - | (3)
      - | ResultSetインタフェースを除く、JDBCインタフェースのメソッド呼び出し（引数と、返り値）を出力するためのロガー。JDBC関連で問題が発生した時の解析に有効なログであるが、出力されるログの量が多い。
    * - | (4)
      - | Connectionの接続/切断イベントと使用中の接続数を出力するためのロガー。接続リークが発生時の解析に有効なログであるが、接続リークの問題がなければ、出力する必要はない。
    * - | (5)
      - | ResultSetインタフェースに対するメソッド呼び出し（引数と、返り値）を出力するためのロガー。取得結果が、想定と異なった時の解析に有効なログであるが、出力されるログの量が多い。
    * - | (6)
      - | ResultSetの中身を確認しやすい形式にフォーマットして出力するためのロガー。取得結果が、想定と異なった時の解析に有効なログであるが、出力されるログの量が多い。

 .. warning::

    **ロガーによっては大量にログが出力されるので、必要なロガーのみ定義、または出力対象にすること。**

    上記サンプルでは、開発中の非常に有効なログを出力するロガーについて、ログレベルを\ ``"debug"``\ に設定している。
    その他のロガーについては、必要に応じて\ ``"debug"``\ に設定する必要がある。

    **性能試験及び商用環境にリリースする場合、正常終了時にlog4jdbc用のロガーによってログが出力されないようにすること。**

    具体的には、ログレベルを\ ``"warn"``\ に設定する。


log4jdbcのオプションの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
クラスパス直下に、\ :file:`log4jdbc.properties`\ というプロパティファイルを配置することで、log4jdbcのデフォルトの動作をカスタマイズすることができる。

- :file:`log4jdbc.properties`

 .. code-block:: properties

     # (1)
     log4jdbc.dump.sql.maxlinelength=0
     # (2)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - SQL分の折り返し文字数を指定する。0を指定すると、折り返しはされない。
    * - | (2)
      - オプションの詳細については、\ `log4jdbc project page -Options- <https://code.google.com/p/log4jdbc/#Options>`_\ を参照されたい。

|

How to extend
--------------------------------------------------------------------------------

.. _data-access-common_todo_multiple_datasource_howtoextends:

複数データソースを使用するための設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

 .. todo::

    **TBD**

    今後、以下の内容を追加する予定である。

    * 処理パターン（複数のデータソースに対して更新あり、更新は１つのデータソース、参照のみ、同時アクセスはなしなど）によってトランザクション管理の方法がかわると思うので、その辺りを中心にブレークダウンする予定である。


動的にデータソースを切り替えるための設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 複数のデータソースを定義し、動的に切り替えを行うには、\ ``org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource``\ を継承したクラスを作成し、どのような条件でデータソースを切り替えるかを実装する必要がある。
| 具体的には\ ``determineCurrentLookupKey``\ メソッドの戻り値となるキーとデータソースをマッピングさせることによって、これを実現する。キーの選択には通常、認証ユーザー情報、時間、ロケール等のコンテキスト情報を使用する。

AbstractRoutingDataSourceの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| \ ``AbstractRoutingDataSource``\ を拡張して作成した\ ``DataSource``\ を、通常のデータソースと同じように使用することでデータソースの動的な切り替えが実現できる。
| 以下に、時間によってデータソースを切り替える例を示す。

- \ ``AbstractRoutingDataSource``\ を継承したクラスの実装例`

 .. code-block:: java

    package com.examples.infra.datasource;

    import javax.inject.Inject;

    import org.joda.time.DateTime;
    import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;
    import org.terasoluna.gfw.common.date.jodatime.JodaTimeDateFactory;

    public class RoutingDataSource extends AbstractRoutingDataSource { // (1)

        @Inject
        JodaTimeDateFactory dateFactory; // (2)

        @Override
        protected Object determineCurrentLookupKey() { // (3)

            DateTime dateTime = dateFactory.newDateTime();
            int hour = dateTime.getHourOfDay();

            if (7 <= hour && hour <= 23) { // (4)
                return "OPEN"; // (5)
            } else {
                return "CLOSE";
            }
        }
    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - \ ``AbstractRoutingDataSource``\ を継承する。
    * - | (2)
      - 時刻を取得するため、\ ``JodaTimeDateFactory``\ を使用する。詳細は、\ :doc:`../GeneralFuncDetail/SystemDate`\ を参照のこと。
    * - | (3)
      - \ ``determineCurrentLookupKey``\ メソッドを実装する。このメソッドの返り値と後述するbean定義ファイル内の\ ``targetDataSources``\ に定義した\ ``key``\ をマッピングすることにより使用するデータソースが決定される。
    * - | (4)
      - メソッド内で、コンテキスト情報（ここでは時間）を参照し、キーの切り替えを行う。ここは業務用件に合わせて実装する必要がある。このサンプルは、時刻が「7:00から23:59まで」と「0:00から6:59まで」で違うキーを返すように実装されている。
    * - | (5)
      - 後述するbean定義ファイル内の\ ``targetDataSources``\ とマッピングさせる\ ``key``\ を返す。

.. note

    認証ユーザー情報(IDや権限)によってデータソースを切り替えたい場合には、\ ``determineCurrentLookupKey``\ メソッド内で、\ ``org.springframework.security.core.context.SecurityContext``\ を使用して取得すれば良い。
    \ ``org.springframework.security.core.context.SecurityContext``\ クラスの詳細は\ :doc:`../Security/Authentication`\ を参照のこと。

データソースの定義
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

作成した\ ``AbstractRoutingDataSource``\ 拡張クラスをbean定義ファイルに定義する。

- :file:`xxx-env.xml`

 .. code-block:: xml

    <bean id="dataSource"
        class="com.examples.infra.datasource.RoutingDataSource">  <!-- (1) -->
        <property name="targetDataSources">  <!-- (2) -->
            <map>
                <entry key="OPEN" value-ref="dataSourceOpen" />
                <entry key="CLOSE" value-ref="dataSourceClose" />
            </map>
        </property>
        <property name="defaultTargetDataSource" ref="dataSourceDefault" />  <!-- (3) -->
    </bean>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - 先ほど作成した\ ``AbstractRoutingDataSource``\ を継承したクラスを定義する。
    * - | (2)
      - 使用するデータソースを定義する。\ ``key``\ は\ ``determineCurrentLookupKey``\ メソッドで返却しうる値を定義する。\ ``value-ref``\ には\ ``key``\ ごとに使用するデータソースを指定する。\ :ref:`データソースの設定 <data-access-common_howtouse_datasource>`\ をもとに切り替えるデータソースの個数分、定義を行う必要がある。
    * - | (3)
      - \ ``determineCurrentLookupKey``\ メソッドで指定した\ ``key``\ が\ ``targetDataSources``\ に存在しない場合は、このデータソースが使用される。実装例の場合、デフォルトが使用されることはないが、今回は説明のため、\ ``defaultTargetDataSource``\ を定義している。


|

how to solve the problem
--------------------------------------------------------------------------------
|

.. _data-access-common_howtosolve_n_plus_1:

N+1問題の対策方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
N+1問題とは、データベースから取得するレコード数に比例して実行されるSQLの数が増えることにより、データベースへの負荷およびレスポンスタイムの劣化を引き起こす問題のことである。

以下に、具体的をあげる。

 .. figure:: images/dataaccess_common-n_plus_1.png
    :alt: about N+1 Problem
    :width: 90%
    :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 検索条件に一致するレコードを、メインとなるテーブルから検索する。
        | 上記例では、 MainTableテーブルのcol1カラムが、\ ``'Foo'``\ のレコードを取得しており、20件のレコードが取得されている。
    * - | (2)
      - | (1)で検索した各レコードに対して、関連レコードを関連テーブルから取得する。
        | 上記例では、SubTableテーブルのidカラムが、(1)で取得したレコードのidカラムと同じレコードを取得している。
        | **このSQLは、(1)で取得されたレコード件数分、実行される。**

 | 上記例では、\ **合計で21回のSQLが発行されることになる。**\
 | 仮に関連テーブルが3テーブルあると、\ **合計で61回のSQLが発行されることになるため、対策が必要となる。**\


N+1問題の解決方法の代表例を、以下に示す。


JOIN(Join Fetch)を使用して解決する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 関連テーブルをJOINすることで、1回のSQLでメインのテーブルと関連テーブルのレコードを取得する。
| 関連テーブルとの関係が、1:1の場合は、この方法によって解決することを検討すること。

 .. figure:: images/dataaccess_common-n_plus_1_solve_join.png
    :alt: about solve N+1 Problem using JOIN
    :width: 90%
    :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 検索条件に一致するレコードを検索する際に、関連テーブルをJOINすることで、メインとなるテーブルと関連テーブルから、レコードを一括で取得する。
        | 上記例では、 MainTableテーブルのcol1カラムが\ ``'Foo'``\ のレコードと、検索条件に一致したレコードのidが一致するSubTableのレコードを一括で取得している。
        | カラム名が重複する場合は、別名を付与してどちらのテーブルのカラムなのか識別する必要がある。

 | JOIN(Join Fetch)を使用すると、\ **1回のSQLの発行で必要なデータを全て取得することができる。**\

 .. note:: **JPQLでJOINする場合**

     JPQLでJOINする場合の実装例については、\ :ref:`data-access-jpa_howtouse_join_fetch`\ を参照されたい。

 .. warning::

    関連テーブルとの関連が、1:Nの場合は、JOIN(Join Fetch)による解決も可能だが、以下の点に注意すること。

    * 1:Nの関連をもつレコードをJOINする場合、関連テーブルのレコード数に比例して、無駄なデータを取得することになる。
      詳細については、\ :ref:`一括取得時の注意事項 <DataAccessMyBatis3AppendixAcquireRelatedObjectsWarningSqlMapping>`\ を参照されたい。

    * JPA(Hibernate)使用する際に、1:NのNの部分が、複数ある場合は、Nの部分を格納するコレクション型は、\ ``java.util.List``\ ではなく、\ ``java.util.Set``\ を使用する必要がある。


関連レコードを一括で取得する事で解決する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 1:Nの関係が複数あるパターンなどは、関連レコードを一括で取得し、その後プログラミングによって振り分ける方法をとった方がよいケースがある。
| 関連テーブルとの関係が1:Nの場合は、この方法によって解決することを検討すること。

 .. figure:: images/dataaccess_common-n_plus_1_solve_programing.png
    :alt: about solve N+1 Problem using programing
    :width: 90%
    :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 検索条件に一致するレコードを、メインとなるテーブルから検索する。
        | 上記例では、 MainTableテーブルのcol1カラムが、\ ``'Foo'``\ のレコードを取得しており、20件のレコードが取得されている。
    * - | (2)
      - | (1)で検索した各レコードに対して、関連レコードを関連テーブルから取得する。
        | 1レコード毎に取得するのではなく、(1)で取得した各レコードの外部キーに一致するレコードを、一括で取得する。
        | 上記例では、SubTableテーブルのidカラムが、(1)で取得したレコードのidカラムと同じレコードを、IN句を使用して一括取得している。
    * - | (3)
      - | (2)で取得したSubTableのレコードを、(1)で取得したレコードに振り分けマージする。

 | 上記例では、\ **合計で2回のSQLの発行で、必要なデータを取得することができる。**\
 | 仮に、関連テーブルが、3テーブルあっても、\ **合計で4回のSQLの発行で済むことになる。**\

 .. note::

     この方法は、SQLの発行を最小限におさえつつ、必要なデータのみ取得することができるという特徴をもつ。
     関連テーブルのレコードをプログラミングによって振り分ける必要があるが、関連テーブルの数が多い場合や、1:NのNのレコード数が多い場合は、この方法で解決する方がよいケースがある。

|

Appendix
--------------------------------------------------------------------------------

.. _data-access-common_appendix_like_escape:

LIKE検索時のエスケープについて
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
LIKE検索を行う場合は、検索条件として使用する値を、LIKE検索用にエスケープする必要がある。

共通ライブラリでは、LIKE検索用のエスケープ処理を行うためのコンポーネントとして、以下のクラスを提供している。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.40\linewidth}|p{0.50\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 40 50

    * - 項番
      - クラス
      - 説明
    * - 1.
      - | org.terasoluna.gfw.common.query.
        | QueryEscapeUtils
      - SQL及びJPQLのエスケープ処理を行うメソッドを提供するユーティリティクラス。

        本クラスでは、

        * LIKE検索用のエスケープ処理を行うメソッド

        を提供している。

    * - 2.
      - | org.terasoluna.gfw.common.query.
        | LikeConditionEscape
      - LIKE検索用のエスケープ処理を行うクラス。

.. note::

    \ ``LikeConditionEscape``\ クラスは、「`LIKE検索用のワイルドカード文字の扱いに関するバグ <https://github.com/terasolunaorg/terasoluna-gfw/issues/78>`_」を修正するために、
    terasoluna-gfw-common 1.0.2.RELEASEから追加したクラスである。

    \ ``LikeConditionEscape``\ クラスは、データベース及びデータベースのバージョンの違いによるワイルドカード文字の違いを吸収する役割を持つ。

|

共通ライブラリのエスケープ仕様について
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
共通ライブラリから提供しているエスケープ処理の仕様は、以下の通りである。

* エスケープ文字は「 ``"~"`` 」。
* エスケープ対象文字は、デフォルトでは「 ``"%"`` , ``"_"``」の2文字。

.. note::

    エスケープ対象文字は、terasoluna-gfw-common 1.0.1.RELEASEまでは「 ``"%"`` , ``"_"`` , ``"％"`` , ``"＿"``」の4文字であったが、
    「`LIKE検索用のワイルドカード文字の扱いに関するバグ <https://github.com/terasolunaorg/terasoluna-gfw/issues/78>`_」を修正するために、
    terasoluna-gfw-common 1.0.2.RELEASEより「 ``"%"`` , ``"_"``」の2文字に変更している。

    なお、エスケープ対象文字として全角文字「``"％"`` , ``"＿"``」を含めてエスケープする方法も提供している。

|

具体的なエスケープ例を以下に示す。

**[デフォルト仕様のエスケープ例]**

エスケープ対象文字としてデフォルト値を使用する場合のエスケープ例を以下に示す。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.20\linewidth}|p{0.10\linewidth}|p{0.45\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 15 20 10 45

    * - | 項番
      - | 対象
        | 文字列
      - | エスケープ後
        | 文字列
      - | エスケープ
        | 有無
      - | 解説
    * - 1.
      - ``"a"``
      - ``"a"``
      - 無
      - エスケープ対象文字が含まれていないため、エスケープされない。
    * - 2.
      - ``"a~"``
      - ``"a~~"``
      - 有
      - エスケープ文字が含まれているため、エスケープされる。
    * - 3.
      - ``"a%"``
      - ``"a~%"``
      - 有
      - エスケープ対象文字が含まれているため、エスケープされる。
    * - 4.
      - ``"a_"``
      - ``"a~_"``
      - 有
      - No.3と同様。
    * - 5.
      - ``"_a%"``
      - ``"~_a~%"``
      - 有
      - エスケープ対象文字が含まれているため、エスケープされる。エスケープ対象文字が複数存在する場合はすべてエスケープされる。
    * - 6.
      - ``"a％"``
      - ``"a％"``
      - 無
      - No.1と同様。

        terasoluna-gfw-common 1.0.2.RELEASEより、デフォルト仕様では「``"％"``」はエスケープ対象外の文字として扱う。
    * - 7.
      - ``"a＿"``
      - ``"a＿"``
      - 無
      - No.1と同様。

        terasoluna-gfw-common 1.0.2.RELEASEより、デフォルト仕様では「``"＿"``」はエスケープ対象外の文字として扱う。
    * - 8.
      - ``" "``
      - ``" "``
      - 無
      - No.1と同様。
    * - 9.
      - ``""``
      - ``""``
      - 無
      - No.1と同様。
    * - 10.
      - ``null``
      - ``null``
      - 無
      - No.1と同様。

|

**[全角文字を含める場合のエスケープ例]**

エスケープ対象文字として全角文字を含める場合のエスケープ例を以下に示す。
項番6と7以外は、デフォルト仕様のエスケープ例を参照されたい。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.20\linewidth}|p{0.10\linewidth}|p{0.45\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 15 20 10 45

    * - | 項番
      - | 対象
        | 文字列
      - | エスケープ後
        | 文字列
      - | エスケープ
        | 有無
      - | 解説
    * - 6.
      - ``"a％"``
      - ``"a~％"``
      - 有
      - エスケープ対象文字が含まれているため、エスケープされる。
    * - 7.
      - ``"a＿"``
      - ``"a~＿"``
      - 有
      - No.6と同様。

|

共通ライブラリから提供しているエスケープ用のメソッドについて
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
共通ライブラリから提供している\ ``QueryEscapeUtils``\ クラスと\ ``LikeConditionEscape``\ クラスのLIKE検索用のエスケープメソッドの一覧を、以下に示す。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 35 55

    * - 項番
      - メソッド名
      - 説明
    * - 1.
      - toLikeCondition(String)
      - | 引数で渡された文字列をLIKE検索用にエスケープする。
        | SQLやJPQL側で一致方法(前方一致、後方一致、部分一致)を指定する場合は、本メソッドを使用してエスケープのみ行う。
    * - 2.
      - toStartingWithCondition(String)
      - | 引数で渡された文字列をLIKE検索用にエスケープした上で、エスケープ後の文字列の最後尾に ``"%"`` を付与する。
        | 前方一致検索用の値に変換する場合に使用するメソッドである。
    * - 3.
      - toEndingWithCondition(String)
      - | 引数で渡された文字列をLIKE検索用にエスケープした上で、エスケープ後の文字列の先頭に ``"%"`` を付与する。
        | 後方一致検索用の値に変換する場合に使用するメソッドである。
    * - 4.
      - toContainingCondition(String)
      - | 引数で渡された文字列をLIKE検索用にエスケープした上で、エスケープ後の文字列の先頭と最後尾に ``"%"`` を付与する。
        | 部分一致検索用の値に変換する場合に使用するメソッドである。

 .. note::

    No.2, 3, 4 については、SQLやJPQL側で一致方法(前方一致、後方一致、部分一致)を指定するのではなく、プログラム側で指定する時に使用するメソッドである。

|

共通ライブラリの使用方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
LIKE検索時のエスケープ処理の実装例については、使用するO/R Mapper向けのドキュメントを参照されたい。

* MyBatis3を使用する場合は、\ :doc:`DataAccessMyBatis3`\ の\ :ref:`DataAccessMyBatis3HowToUseLikeEscape`\ を参照されたい。
* JPA(Spring Data JPA)を使用する場合は、\ :doc:`DataAccessJpa`\ の\ :ref:`data-access-jpa_howtouse_like_escape`\ を参照されたい。

.. note::

    エスケープ処理を行うために使用するAPIは、使用するデータベースがサポートしているワイルドカード文字によって使い分ける必要がある。

    **[ワイルドカードとして「 "%" , "_"」(半角文字)のみをサポートしているデータベースの場合]**

     .. code-block:: java

        String escapedWord = QueryEscapeUtils.toLikeCondition(word);

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90

        * - | 項番
          - | 説明
        * - | (1)
          -  \ ``QueryEscapeUtils``\ クラスのメソッドを直接使用して、エスケープ処理を行う。

    **[ワイルドカードとして「"％" , "＿"」(全角文字)もサポートしているデータベースの場合]**

     .. code-block:: java

        String escapedWord = QueryEscapeUtils.withFullWidth()  // (2)
                                .toLikeCondition(word);        // (3)


     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90
        :class: longtable

        * - | 項番
          - | 説明
        * - | (2)
          -  \ ``QueryEscapeUtils``\ メソッドの\ ``withFullWidth()``\ メソッドを呼び出して、\ ``LikeConditionEscape``\ クラスのインスタンスを取得する。
        * - | (3)
          -  (2)で取得した\ ``LikeConditionEscape``\ クラスのインスタンスのメソッドを使用して、エスケープ処理を行う。

|

.. _data-access-common_appendix_sequencer:

Sequencerについて
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Sequencerは、シーケンス値を取得するための共通ライブラリである。
| Sequencerから取得したシーケンス値は、データベースのプライマリキーカラムの設定値などとして使用する。

 .. note:: **共通ライブラリとしてSequencerを用意した理由**

    Sequencerを用意した理由は、JPAの機能として提供されているID採番機能において、シーケンス値を文字列としてフォーマットする仕組みがないためである。
    実際のアプリケーション開発では、フォーマットされた文字列をプライマリキーに設定するケースもあるため、共通ライブラリとしてSequencerを提供している。

    プライマリキーに設定する値が数値の場合は、JPAの機能として提供されているID採番機能を使用することを推奨する。JPAのID採番機能については、\ :doc:`DataAccessJpa`\ の\ :ref:`data-access-jpa_how_to_use_way_to_add_entity`\ を参照されたい。

    Sequencerを用意した主な目的は、JPAでサポートされていない機能の補完であるが、JPAと関係ない処理で、シーケンス値が必要な場合に、使用することもできる。

共通ライブラリから提供しているクラスについて
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 共通ライブラリから提供しているSequencer機能のクラス一覧を以下に示す。
| 具体的な使用例については、How to useの\ :ref:`data-access-common_howtouse_sequencer`\ を参照されたい。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 60

    * - 項番
      - クラス名
      - 説明
    * - 1.
      - | org.terasoluna.gfw.common.sequencer.
        | Sequencer
      - | 次のシーケンス値を取得するメソッド(getNext)とシーケンスの現在値を返却するメソッド(getCurrent)を定義しているインタフェース。
    * - 2.
      - | org.terasoluna.gfw.common.sequencer.
        | JdbcSequencer
      - | ``Sequencer`` インタフェースのJDBC用の実装クラス。
        | データベースにSQLを発行してシーケンス値を取得するためのクラスである。
        | データベースのシーケンスオブジェクトから値を取得することを想定したクラスではあるが、データベースにストアードされているファンクションを呼び出すことで、シーケンスオブジェクト以外から値を取得することもできる。

.. _data-access-common_howtouse_sequencer:

共通ライブラリの利用方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Sequencerをbean定義する。

- :file:`xxx-infra.xml`

 .. code-block:: xml

    <!-- (1) -->
    <bean id="articleIdSequencer" class="org.terasoluna.gfw.common.sequencer.JdbcSequencer">
         <!-- (2) -->
        <property name="dataSource" ref="dataSource" />
         <!-- (3) -->
        <property name="sequenceClass" value="java.lang.String" />
        <!-- (4) -->
        <property name="nextValueQuery"
            value="SELECT TO_CHAR(NEXTVAL('seq_article'),'AFM0000000000')" />
        <!-- (5) -->
        <property name="currentValueQuery"
            value="SELECT TO_CHAR(CURRVAL('seq_article'),'AFM0000000000')" />
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``org.terasoluna.gfw.common.sequencer.Sequencer``\ インタフェースを実装したクラスを、bean定義する。
        | 上記例では、SQLを発行してシーケンス値を取得するためのクラス(\ ``JdbcSequencer``\ )を指定している。
    * - | (2)
      - | シーケンス値を取得するSQLを、実行するデータソースを指定する。
    * - | (3)
      - | 取得するシーケンス値の型を指定する。
        | 上記例では、SQLで文字列へ変換しているので、\ ``java.lang.String``\ 型を指定している。
    * - | (4)
      - | 次のシーケンス値を取得するためのSQLを指定する。
        | 上記例では、データベース(PostgreSQL)のシーケンスオブジェクトから取得したシーケンス値を、文字列としてフォーマットしている。
        | データベースのから取得したシーケンス値が、\ ``1``\ の場合、\ ``"A0000000001"``\ が\ ``Sequencer#getNext()``\ メソッドの返り値として返却される。
    * - | (5)
      - | 現在のシーケンス値を取得するためのSQLを指定する。
        | データベースのから取得したシーケンス値が、\ ``2``\ の場合、\ ``"A0000000002"``\ が\ ``Sequencer#getCurrent()``\ メソッドの返り値として返却される。


bean定義したSequencerからシーケンス値を取得する。

- Service

 .. code-block:: java

    // omitted

    // (1)
    @Inject
    @Named("articleIdSequencer") // (2)
    Sequencer<String> articleIdSequencer;

    // omitted

    @Transactional
    public Article createArticle(Article inputArticle) {

        String articleId = articleIdSequencer.getNext(); // (3)
        inputArticle.setArticleId(articleId);

        Article savedArticle = articleRepository.save(inputArticle);

        return savedArticle;
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | bean定義した\ ``Sequencer``\ オブジェクトをInjectする。
        | 上記例では、シーケンス値は、フォーマットされた文字列として取得するため、\ ``Sequencer``\ のジェネリックス型には、\ ``java.lang.String``\ 型を指定している。
    * - | (2)
      - | Injectするbeanのbean名を\ ``@javax.inject.Named``\ アノテーションのvalue属性に指定する。
        | 上記例では、\ :file:`xxx-infra.xml`\ に定義したbean名(\ ``"articleIdSequencer"``\ )を指定している。
    * - | (3)
      - | \ ``Sequencer#getNext()``\ メソッドを呼び出し、次のシーケンス値を取得する。
        | 上記例では、取得したシーケンス値を、EntityのIDとして使用している。
        | 現在のシーケンス値を取得する場合は、\ ``Sequencer#getCurrent()``\ メソッドを呼び出す。

 .. tip::

    bean定義する\ ``Sequencer``\ が一つの場合は、\ ``@Named``\ アノテーションが省略できる。複数指定する場合は、\ ``@Named``\ アノテーションを使用して、bean名の指定が必要となる。

.. _appendix_dataaccessexception_converter_class-label:

Spring Frameworkから提供されているデータアクセス例外へ変換するクラス
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Spring Frameworkのデータアクセス例外へ変換する役割を持つクラスを、以下に示す。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
 .. list-table:: **Spring Frameworkのデータアクセス例外への変換クラス**
    :header-rows: 1
    :widths: 10 30 60

    * - 項番
      - クラス名
      - 説明
    * - 1.
      - | org.springframework.jdbc.support.
        | SQLErrorCodeSQLExceptionTranslator
      - MyBatisや、\ ``JdbcTemplate``\ を使った場合、本クラスによって、JDBC例外が、Spring Frameworkのデータアクセス例外に変換される。変換ルールは、XMLファイルに記載されており、デフォルトで使用されるXMLファイルは、\ ``spring-jdbc.jar``\ 内の\ ``org/springframework/jdbc/support/sql-error-codes.xml``\ となる。
        クラスパス直下に、XMLファイル（\ ``sql-error-codes.xml``\ ）を配置することで、デフォルトの動作を変更することもできる。
    * - 2.
      - | org.springframework.orm.jpa.vendor.
        | HibernateJpaDialect
      - JPA(Hibernateの実装)を使った場合、本クラスによって、O/R Mapper例外(Hibernateの例外)がSpring Frameworkのデータアクセス例外に変換される。
    * - 3.
      - | org.springframework.orm.jpa.
        | EntityManagerFactoryUtils
      - \ ``HibernateJpaDialect``\ で変換できない例外が発生した場合は、本クラスによって、JPA例外がSpring Frameworkのデータアクセス例外に変換される。
    * - 4.
      - | org.hibernate.dialect.Dialect
        | のサブクラス
      - JPA(Hibernateの実装)を使った場合、本クラスによって、JDBC例外とO/R Mapper例外に変換される。

.. _appendix_datasource_of_spring-label:

Spring Frameworkから提供されているJDBCデータソースクラス
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Spring Frameworkでは、JDBCデータソースの実装を提供しているが、非常にシンプルなクラスなので、商用環境で使われることは少ない。
| 主に単体試験時に使用されるクラスである。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table:: **Spring Frameworkから提供されているJDBCデータソース**
    :header-rows: 1
    :widths: 10 35 55

    * - 項番
      - クラス名
      - 説明
    * - 1.
      - | org.springframework.jdbc.datasource.
        | DriverManagerDataSource
      - アプリケーションからコネクションの取得依頼があったタイミングで、\ ``java.sql.DriverManager#getConnection``\ を呼び出し、新しいコネクションを生成するデータソースクラス。
        コネクションのプーリングが必要な場合は、アプリケーションサーバのデータソース、または、OSS/Third-Partyライブラリから提供されているデータソースを使用すること。
    * - 2.
      - | org.springframework.jdbc.datasource.
        | SingleConnectionDataSource
      - \ ``DriverManagerDataSource``\ の子クラスで、一つのコネクションを使いまわす実装になっており、シングルスレッドで動くユニットテスト向けのデータソースクラスである。
        ユニットテストでも、マルチスレッドでデータソースにアクセスする場合は、本クラスを使用すると、期待した動作にならないことがあるので、注意が必要である。
    * - 3.
      - | org.springframework.jdbc.datasource.
        | SimpleDriverDataSource
      - アプリケーションからコネクションの取得依頼があったタイミングで、\ ``java.sql.Driver#getConnection``\ を呼び出し、新しいコネクションを生成するデータソースクラス。
        コネクションのプーリングが必要な場合は、アプリケーションサーバのデータソース、または、OSS/Third-Partyライブラリから提供されているデータソースを使用すること。


| Spring Frameworkでは、JDBCデータソースの動作を拡張したアダプタークラスを提供している。
| 以下に、代表的なアダプタークラスを紹介する。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table:: **Spring Frameworkから提供されているJDBCデータソースのアダプター**
    :header-rows: 1
    :widths: 10 35 55

    * - 項番
      - クラス名
      - 説明
    * - 1.
      - | org.springframework.jdbc.datasource.
        | TransactionAwareDataSourceProxy
      - トランザクション管理されていないデータソースを、Spring Frameworkのトランザクション管理対象にするためのアダプタークラス。
    * - 2.
      - | org.springframework.jdbc.datasource.lookup.
        | IsolationLevelDataSourceRoute
      - 実行中のトランザクションの独立性レベルによって、使用するデータソースを切り替えるためのアダプタークラス。

.. raw:: latex

   \newpage
