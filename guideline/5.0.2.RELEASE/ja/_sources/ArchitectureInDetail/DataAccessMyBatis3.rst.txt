データベースアクセス（MyBatis3編）
================================================================================

.. only:: html

 .. contents:: 目次
    :local:
    :depth: 3


.. _DataAccessMyBatis3Overview:

Overview
--------------------------------------------------------------------------------

本節では、\ `MyBatis3 <http://mybatis.org>`_\ を使用してデータベースにアクセスする方法について説明する。

本ガイドラインでは、MyBatis3のMapperインタフェースをRepositoryインタフェースとして使用することを前提としている。
Repositoryインタフェースについては、「:ref:`repository-label`」を参照されたい。

| Overviewでは、MyBatis3とMyBatis-Springを使用してデータベースアクセスする際のアーキテクチャについて説明を行う。
| 実際の使用方法については、「:ref:`DataAccessMyBatis3HowToUse`」を参照されたい。

 .. figure:: images_DataAccessMyBatis3/DataAccessMyBatis3Scope.png
    :alt: Scope of description
    :width: 100%
    :align: center

    **Picture - Scope of description**

|

.. _DataAccessMyBatis3OverviewAboutMyBatis3:

MyBatis3について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| MyBatis3は、O/R Mapperの一つだが、データベースで管理されているレコードとオブジェクトをマッピングするという考え方ではなく、
 SQLとオブジェクトをマッピングするという考え方で開発されたO/R Mapperである。
| そのため、正規化されていないデータベースへアクセスする場合や、発行するSQLをO/R Mapperに任せずに、
 アプリケーション側で完全に制御したい場合に有効なO/R Mapperである。

本ガイドラインでは、MyBatis3から追加されたMapperインタフェースを使用して、EntityのCRUD操作を行う。
Mapperインタフェースの詳細については、「:ref:`DataAccessMyBatis3AppendixAboutMapperMechanism`」を参照されたい。

本ガイドラインでは、MyBatis3の全ての機能の使用方法について説明を行うわけではないため、
「\ `MyBatis 3 REFERENCE DOCUMENTATION <http://mybatis.github.io/mybatis-3/>`_ \」も合わせて参照して頂きたい。

|

.. _DataAccessMyBatis3OverviewAboutComponentConstitutionOfMyBatis3:

MyBatis3のコンポーネント構成について
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| MyBatis3の主要なコンポーネント(設定ファイル)について説明する。
| MyBatis3では、設定ファイルの定義に基づき、以下のコンポーネントが互いに連携する事によって、SQLの実行及びO/Rマッピングを実現している。

 .. tabularcolumns:: |p{0.1\linewidth}|p{0.2\linewidth}|p{0.6\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - 項番
      - コンポーネント/設定ファイル
      - 説明
    * - (1)
      - MyBatis設定ファイル
      - MyBatis3の動作設定を記載するXMLファイル。

        データベースの接続先、マッピングファイルのパス、MyBatisの動作設定などを記載するファイルである。
        Springと連携して使用する場合は、データベースの接続先やマッピングファイルのパスの設定を本設定ファイルに指定する必要がないため、
        MyBatis3のデフォルトの動作を変更又は拡張する際に、設定を行う事になる。
    * - (2)
      - ``org.apache.ibatis.session.``
        ``SqlSessionFactoryBuilder``
      - MyBatis設定ファイルを読込み、\ ``SqlSessionFactory`` \を生成するためのコンポーネント。

        Springと連携して使用する場合は、アプリケーションのクラスから本コンポーネントを直接扱うことはない。
    * - (3)
      - ``org.apache.ibatis.session.``
        ``SqlSessionFactory``
      - \ ``SqlSession`` \を生成するためのコンポーネント。

        Springと連携して使用する場合は、アプリケーションのクラスから本コンポーネントを直接扱うことはない。
    * - (4)
      - ``org.apache.ibatis.session.``
        ``SqlSession``
      - SQLの発行やトランザクション制御のAPIを提供するコンポーネント。

        MyBatis3を使ってデータベースにアクセスする際に、もっとも重要な役割を果たすコンポーネントである。
        
        Springと連携して使用する場合は、アプリケーションのクラスから本コンポーネントを直接扱うことは、基本的にはない。
    * - (5)
      - Mapperインタフェース
      - マッピングファイルに定義したSQLをタイプセーフに呼び出すためのインタフェース。

        Mapperインターフェースに対する実装クラスは、MyBatis3が自動で生成するため、開発者はインターフェースのみ作成すればよい。
    * - (6)
      - マッピングファイル

      - SQLとO/Rマッピングの設定を記載するXMLファイル。

|

| 以下に、MyBatis3の主要コンポーネントが、どのような流れでデータベースにアクセスしているのかを説明する。
| データベースにアクセスするための処理は、大きく２つにわける事ができる。

* アプリケーションの起動時に行う処理。下記(1)～(3)の処理が、これに該当する。
* クライアントからのリクエスト毎に行う処理。下記(4)～(10)の処理が、これに該当する。

 .. figure:: images_DataAccessMyBatis3/DataAccessMyBatis3RelationshipOfComponents.png
    :alt: Relationship of MyBatis3 components
    :width: 100%
    :align: center

    **Picture - Relationship of MyBatis3 components**

| アプリケーションの起動時に行う処理は、以下の流れで実行する。
| Springと連携時の流れについては、「:ref:`DataAccessMyBatis3OverviewAboutComponentConstitutionOfMyBatisSpring`」を参照されたい。

 .. tabularcolumns:: |p{0.1\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80


    * - 項番
      - 説明
    * - (1)
      - アプリケーションは、\ ``SqlSessionFactoryBuilder`` \ に対して \ ``SqlSessionFactory`` \ の構築を依頼する。
    * - (2)
      - \ ``SqlSessionFactoryBuilder`` \は、 \ ``SqlSessionFactory`` \を生成するためにMyBatis設定ファイルを読込む。
    * - (3)
      - \ ``SqlSessionFactoryBuilder`` \ は、MyBatis設定ファイルの定義に基づき \ ``SqlSessionFactory`` \を生成する。

|

| クライアントからのリクエスト毎に行う処理は、以下の流れで実行する。
| Springと連携時の流れについては、「:ref:`DataAccessMyBatis3OverviewAboutComponentConstitutionOfMyBatisSpring`」を参照されたい。

 .. tabularcolumns:: |p{0.1\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80


    * - 項番
      - 説明
    * - (4)
      - クライアントは、アプリケーションに対して処理を依頼する。
    * - (5)
      - アプリケーションは、 \ ``SqlSessionFactoryBuilder`` \によって構築された \ ``SqlSessionFactory`` \から \ ``SqlSession`` \を取得する。
    * - (6)
      - \ ``SqlSessionFactory`` \は、\ ``SqlSession`` \を生成しアプリケーションに返却する。
    * - (7)
      - アプリケーションは、 \ ``SqlSession`` \からMapperインタフェースの実装オブジェクトを取得する。
    * - (8)
      - アプリケーションは、Mapperインタフェースのメソッドを呼び出す。
      
        Mapperインタフェースの仕組みについては、「:ref:`DataAccessMyBatis3AppendixAboutMapperMechanism`」を参照されたい。
    * - (9)
      - Mapperインタフェースの実装オブジェクトは、 \ ``SqlSession`` \のメソッドを呼び出して、SQLの実行を依頼する。
    * - (10)
      - \ ``SqlSession`` \は、マッピングファイルから実行するSQLを取得し、SQLを実行する。

 .. tip:: **トランザクション制御について**
 
    上記フローには記載していないが、トランザクションのコミット及びロールバックは、
    アプリケーションのコードから\ ``SqlSession`` \のAPIを直接呼び出して行う。
    
    ただし、Springと連携する場合は、Springのトランザクション管理機能がコミット及びロールバックを行うため、
    アプリケーションのクラスから\ ``SqlSession`` \のトランザクションを制御するためのAPIを直接呼び出すことはない。


|

.. _DataAccessMyBatis3OverviewAboutMyBatisSpring:

MyBatis3とSpringの連携について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| MyBatis3とSpringを連携させるライブラリとして、MyBatisから\ `MyBatis-Spring <http://mybatis.github.io/spring/>`_ \というライブラリが提供されている。
| このライブラリを使用することで、MyBatis3のコンポーネントをSpringのDIコンテナ上で管理する事ができる。

MyBatis-Springを使用すると、

* MyBatis3のSQLの実行をSpringが管理しているトランザクション内で行う事ができるため、MyBatis3のAPIに依存したトランザクション制御を行う必要がない。

* MyBatis3の例外は、Springが用意している汎用的な例外(\ ``org.springframework.dao.DataAccessException`` \)へ変換されるため、MyBatis3のAPIに依存しない例外処理を実装する事ができる。

* MyBatis3を使用するための初期化処理は、すべてMyBatis-SpringのAPIが行ってくれるため、基本的にはMyBatis3のAPIを直接使用する必要がない。

* スレッドセーフなMapperオブジェクトの生成が行えるため、シングルトンのServiceクラスにMapperオブジェクトを注入する事ができる。

等のメリットがある。
本ガイドラインでは、MyBatis-Springを使用することを前提とする。

本ガイドラインでは、MyBatis-Springの全ての機能の使用方法について説明を行うわけではないため、
「\ `Mybatis-Spring REFERENCE DOCUMENTATION <http://mybatis.github.io/spring/>`_ \」も合わせて参照して頂きたい。

|

.. _DataAccessMyBatis3OverviewAboutComponentConstitutionOfMyBatisSpring:

MyBatis-Springのコンポーネント構成について
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| MyBatis-Springの主要なコンポーネントについて説明する。
| MyBatis-Springでは、以下のコンポーネントが連携する事によって、MyBatis3とSpringの連携を実現している。

 .. tabularcolumns:: |p{0.1\linewidth}|p{0.2\linewidth}|p{0.6\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - 項番
      - コンポーネント/設定ファイル
      - 説明
    * - (1)
      - ``org.mybatis.spring.``
        ``SqlSessionFactoryBean``
      - \ ``SqlSessionFactory`` \を構築し、SpringのDIコンテナ上にオブジェクトを格納するためのコンポーネント。

        MyBatis3標準では、MyBatis設定ファイルに定義されている情報を基に\ ``SqlSessionFactory`` \を構築するが、
        \ ``SqlSessionFactoryBean`` \を使用すると、MyBatis設定ファイルがなくても\ ``SqlSessionFactory`` \を構築することができる。
        もちろん、併用することも可能である。
    * - (2)
      - ``org.mybatis.spring.mapper.``
        ``MapperFactoryBean``
      - シングルトンのMapperオブジェクトを構築し、SpringのDIコンテナ上にオブジェクトを格納するためのコンポーネント。

        MyBatis3標準の仕組みで生成されるMapperオブジェクトはスレッドセーフではないため、
        スレッド毎にインスタンスを割り当てる必要があった。
        MyBatis-Springのコンポーネントで作成されたMapperオブジェクトは、
        スレッドセーフなMapperオブジェクトを生成する事ができるため、ServiceなどのシングルトンのコンポーネントにDIすることが可能となる。
    * - (3)
      - ``org.mybatis.spring.``
        ``SqlSessionTemplate``
      - \ ``SqlSession`` \インターフェースを実装したシングルトン版の\ ``SqlSession`` \コンポーネント。

        MyBatis3標準の仕組みで生成される\ ``SqlSession`` \オブジェクトはスレッドセーフではないため、
        スレッド毎にインスタンスを割り当てる必要があった。
        MyBatis-Springのコンポーネントで作成された\ ``SqlSession`` \オブジェクトは、
        スレッドセーフな\ ``SqlSession`` \オブジェクトが生成されるため、ServiceなどのシングルトンのコンポーネントにDIすることが可能になる。

        ただし、本ガイドラインでは、\ ``SqlSession`` \を直接扱う事は想定していない。

|

以下に、MyBatis-Springの主要コンポーネントが、どのような流れでデータベースにアクセスしているのかを説明する。
データベースにアクセスするための処理は、大きく２つにわける事ができる。

* アプリケーションの起動時に行う処理。下記(1)～(4)の処理が、これに該当する。
* クライアントからのリクエスト毎に行う処理。下記(5)～(11)の処理が、これに該当する。


 .. figure:: images_DataAccessMyBatis3/DataAccessMyBatisSpringRelationshipOfComponents.png
    :alt: Relationship of MyBatis-Spring components
    :width: 100%
    :align: center

    **Picture - Relationship of MyBatis-Spring components**


アプリケーションの起動時に行う処理は、以下の流れで実行される。

 .. tabularcolumns:: |p{0.1\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80


    * - 項番
      - 説明
    * - (1)
      - \ ``SqlSessionFactoryBean`` \は、\ ``SqlSessionFactoryBuilder`` \に対して \ ``SqlSessionFactory`` \の構築を依頼する。
    * - (2)
      - \ ``SqlSessionFactoryBuilder`` \は、 \ ``SqlSessionFactory`` \を生成するためにMyBatis設定ファイルを読込む。
    * - (3)
      - \ ``SqlSessionFactoryBuilder`` \は、MyBatis設定ファイルの定義に基づき \ ``SqlSessionFactory`` \を生成する。

        生成された\ ``SqlSessionFactory`` \は、SpringのDIコンテナによって管理される。
    * - (4)
      - \ ``MapperFactoryBean`` \は、スレッドセーフな\ ``SqlSession`` \(\ ``SqlSessionTemplate`` \)と、
        スレッドセーフなMapperオブジェクト(MapperインタフェースのProxyオブジェクト)を生成する。

        生成されたMapperオブジェクトは、SpringのDIコンテナによって管理され、ServiceクラスなどにDIされる。
        Mapperオブジェクトは、スレッドセーフな\ ``SqlSession`` \(\ ``SqlSessionTemplate`` \)を利用することで、スレッドセーフな実装を提供している。

|

クライアントからのリクエスト毎に行う処理は、以下の流れで実行される。

 .. tabularcolumns:: |p{0.1\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80


    * - 項番
      - 説明
    * - (5)
      - クライアントは、アプリケーションに対して処理を依頼する。
    * - (6)
      - アプリケーション(Service)は、 DIコンテナによって注入されたMapperオブジェクト(Mapperインターフェースを実装したProxyオブジェクト)のメソッドを呼び出す。

        Mapperインタフェースの仕組みについては、 「:ref:`DataAccessMyBatis3AppendixAboutMapperMechanism`」を参照されたい。
    * - (7)
      - Mapperオブジェクトは、呼び出されたメソッドに対応する\ ``SqlSession`` \(\ ``SqlSessionTemplate`` \)のメソッドを呼び出す。
    * - (8)
      - \ ``SqlSession`` \(\ ``SqlSessionTemplate`` \)は、Proxy化されたスレッドセーフな\ ``SqlSession`` \のメソッドを呼び出す。
    * - (9)
      - Proxy化されたスレッドセーフな\ ``SqlSession`` \は、トランザクションに割り当てられているMyBatis3標準の\ ``SqlSession`` \を使用する。

        トランザクションに割り当てられている\ ``SqlSession`` \が存在しない場合は、MyBatis3標準の\ ``SqlSession`` \を取得するために、
        \ ``SqlSessionFactory`` \ のメソッドを呼び出す。
    * - (10)
      - \ ``SqlSessionFactory`` \は、MyBatis3標準の\ ``SqlSession`` \を返却する。

        返却されたMyBatis3標準の\ ``SqlSession`` \はトランザクションに割り当てられるため、同一トランザクション内であれば、新たに生成されることはなく、
        同じ\ ``SqlSession`` \が使用される仕組みになっている。
    * - (11)
      - MyBatis3標準の\ ``SqlSession`` \は、マッピングファイルから実行するSQLを取得し、SQLを実行する。

 .. tip:: **トランザクション制御について**

    上記フローには記載していないが、トランザクションのコミット及びロールバックは、Springのトランザクション管理機能が行う。
    
    Springのトランザクション管理機能を使用したトランザクション管理方法については、
    「:ref:`service_transaction_management`」を参照されたい。

|


.. _DataAccessMyBatis3HowToUse:

How to use
--------------------------------------------------------------------------------

ここからは、実際にMyBatis3を使用して、データベースにアクセスするための設定及び実装方法について、説明する。

以降の説明は、大きく以下に分類する事ができる。


 .. tabularcolumns:: |p{0.1\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60


    * - 項番
      - 分類
      - 説明
    * - (1)
      - アプリケーション全体の設定
      - MyBatis3をアプリケーションで使用するための設定方法や、
        MyBatis3の動作を変更するための設定方法について記載している。

        ここに記載している内容は、\ **プロジェクト立ち上げ時にアプリケーションアーキテクトが設定を行う時に必要となる。**\
        そのため、基本的にはアプリケーション開発者が個々に意識する必要はない部分である。
        
        以下のセクションが、この分類に該当する。
        
        * :ref:`DataAccessMyBatis3HowToUseSettingsPomXml`
        * :ref:`DataAccessMyBatis3HowToUseSettingsCooperateWithMyBatis3AndSpring`
        * :ref:`DataAccessMyBatis3HowToUseSettingsMyBatis3`
        
        `MyBatis3用のブランクプロジェクト <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank#multi-blank-project-with-mybatis3>`_ \ からプロジェクトを生成した場合は、
        上記で説明している設定の多くが既に設定済みの状態となっているため、アプリケーションアーキテクトは、
        プロジェクト特性を判断し、必要に応じて設定の追加及び変更を行うことになる。

    * - (2)
      - データアクセス処理の実装方法
      - MyBatis3を使った基本的なデータアクセス処理の実装方法について記載している。
      
        ここに記載している内容は、\ **アプリケーション開発者が実装時に必要となる。**\
        
        以下のセクションが、この分類に該当する。
        
        * :ref:`DataAccessMyBatis3HowToDababaseAccess`
        * :ref:`DataAccessMyBatis3HowToUseResultSetMapping`
        * :ref:`DataAccessMyBatis3HowToUseFind`
        * :ref:`DataAccessMyBatis3HowToUseCreate`
        * :ref:`DataAccessMyBatis3HowToUseUpdate`
        * :ref:`DataAccessMyBatis3HowToUseDelete`
        * :ref:`DataAccessMyBatis3HowToUseDynamicSql`
        * :ref:`DataAccessMyBatis3HowToUseLikeEscape`
        * :ref:`DataAccessMyBatis3HowToUseSqlInjectionCountermeasure`

|

.. _DataAccessMyBatis3HowToUseSettingsPomXml:

pom.xmlの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| インフラストラクチャ層にMyBatis3を使用する場合は、\ :file:`pom.xml`\にterasoluna-gfw-mybatis3への依存関係を追加する。
| マルチプロジェクト構成の場合は、domainプロジェクトの\ :file:`pom.xml`\(:file:`projectName-domain/pom.xml`)に追加する。

`MyBatis3用のブランクプロジェクト <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank#multi-blank-project-with-mybatis3>`_ \ からプロジェクトを生成した場合は、terasoluna-gfw-mybatis3への依存関係は、設定済の状態である。

 .. code-block:: xml
    :emphasize-lines: 22-26

    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
            http://maven.apache.org/maven-v4_0_0.xsd">

        <modelVersion>4.0.0</modelVersion>
        <artifactId>projectName-domain</artifactId>
        <packaging>jar</packaging>

        <parent>
            <groupId>com.example</groupId>
            <artifactId>mybatis3-example-app</artifactId>
            <version>1.0.0-SNAPSHOT</version>
            <relativePath>../pom.xml</relativePath>
        </parent>

        <dependencies>
        
            <!-- omitted -->

            <!-- (1) -->
            <dependency>
                <groupId>org.terasoluna.gfw</groupId>
                <artifactId>terasoluna-gfw-mybatis3</artifactId>
            </dependency>

            <!-- omitted -->

        </dependencies>

        <!-- omitted -->

    </project>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - terasoluna-gfw-mybatis3をdependenciesに追加する。
        terasoluna-gfw-mybatis3には、MyBatis3及びMyBatis-Springへの依存関係が定義されている。
        
 .. tip:: **terasoluna-gfw-parentをParentプロジェクトとして使用しない場合の設定方法について**
 
    親プロジェクトとしてterasoluna-gfw-parentプロジェクトを指定していない場合は、バージョンの指定も個別に必要となる。

     .. code-block:: xml
        :emphasize-lines: 4
 
        <dependency>
            <groupId>org.terasoluna.gfw</groupId>
            <artifactId>terasoluna-gfw-mybatis3</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
        
    上記例では5.0.2.RELEASEを指定しているが、実際に指定するバージョンは、プロジェクトで利用するバージョンを指定すること。

|

.. _DataAccessMyBatis3HowToUseSettingsCooperateWithMyBatis3AndSpring:

MyBatis3とSpringを連携するための設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _DataAccessMyBatis3HowToUseSettingsDataSource:

データソースの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

MyBatis3とSpringを連携する場合、データソースはSpringのDIコンテナで管理しているデータソースを使用する必要がある。

`MyBatis3用のブランクプロジェクト <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank#multi-blank-project-with-mybatis3>`_ \ からプロジェクトを生成した場合は、Apache Commons DBCPのデータソースが設定済の状態であるため、
プロジェクトの要件に合わせて設定を変更すること。

データソースの設定方法については、共通編の「\ :ref:`data-access-common_howtouse_datasource` \」を参照されたい。

|

.. _DataAccessMyBatis3HowToUseSettingsTransactionManager:

トランザクション管理の設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| MyBatis3とSpringを連携する場合、
 トランザクション管理はSpringのDIコンテナで管理している\ ``PlatformTransactionManager`` \を使用する必要がある。

ローカルトランザクションを使用する場合は、JDBCのAPIを呼び出してトランザクション制御を行う\ ``DataSourceTransactionManager`` \を使用する。

`MyBatis3用のブランクプロジェクト <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank#multi-blank-project-with-mybatis3>`_ \ からプロジェクトを生成した場合は、\ ``DataSourceTransactionManager`` \が設定済みの状態である。

設定例は以下の通り。

- :file:`projectName-env/src/main/resources/META-INF/spring/projectName-env.xml`

 .. code-block:: xml
    :emphasize-lines: 15-20

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:jee="http://www.springframework.org/schema/jee"
        xmlns:jdbc="http://www.springframework.org/schema/jdbc"
        xsi:schemaLocation="http://www.springframework.org/schema/jdbc
            http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
            http://www.springframework.org/schema/jee
            http://www.springframework.org/schema/jee/spring-jee.xsd
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd">

        <!-- omitted -->

        <!-- (1) -->
        <bean id="transactionManager"
            class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <!-- (2) -->
            <property name="dataSource" ref="dataSource" />
        </bean>

        <!-- omitted -->

    </beans>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - \ ``PlatformTransactionManager`` \として、\ ``org.springframework.jdbc.datasource.DataSourceTransactionManager`` \を指定する。
    * - (2)
      - \ ``dataSource`` \プロパティに、設定済みのデータソースのbeanを指定する。

        トランザクション内でSQLを実行する際は、ここで指定したデータソースからコネクションが取得される。

 .. note:: **PlatformTransactionManagerのbean IDについて**
 
    id属性には、\ ``transactionManager`` \を指定することを推奨する。
    
    \ ``transactionManager`` \以外の値を指定すると、
    \ ``<tx:annotation-driven>`` \タグのtransaction-manager属性に同じ値を設定する必要がある。
    

|

アプリケーションサーバから提供されているトランザクションマネージャを使用する場合は、JTAのAPIを呼び出してトランザクション制御を行う
\ ``org.springframework.transaction.jta.JtaTransactionManager`` \を使用する。

設定例は以下の通り。

- :file:`projectName-env/src/main/resources/META-INF/spring/projectName-env.xml`

 .. code-block:: xml
    :emphasize-lines: 6,13-14,18-19

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:jee="http://www.springframework.org/schema/jee"
        xmlns:jdbc="http://www.springframework.org/schema/jdbc"
        xmlns:tx="http://www.springframework.org/schema/tx"
        xsi:schemaLocation="http://www.springframework.org/schema/jdbc
            http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
            http://www.springframework.org/schema/jee
            http://www.springframework.org/schema/jee/spring-jee.xsd
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/tx
            http://www.springframework.org/schema/tx/spring-tx.xsd">

        <!-- omitted -->

        <!-- (1) -->
        <tx:jta-transaction-manager />

        <!-- omitted -->

    </beans>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - \ ``<tx:jta-transaction-manager />`` \を指定すると、
        アプリケーションサーバに対して最適な \ ``JtaTransactionManager`` \がbean定義される。

|

.. _DataAccessMyBatis3HowToUseSettingsMyBatis-Spring:

MyBatis-Springの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

MyBatis3とSpringを連携する場合、MyBatis-Springのコンポーネントを使用して、

* MyBatis3とSpringを連携するために必要となる処理がカスタマイズされた\ ``SqlSessionFactory``\ の生成
* スレッドセーフなMapperオブジェクト(MapperインタフェースのProxyオブジェクト)の生成

を行う必要がある。

`MyBatis3用のブランクプロジェクト <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank#multi-blank-project-with-mybatis3>`_ \ からプロジェクトを生成した場合は、MyBatis3とSpringを連携するための設定は、
設定済みの状態である。

設定例は以下の通り。

- :file:`projectName-domain/src/main/resources/META-INF/spring/projectName-infra.xml`

 .. code-block:: xml
    :emphasize-lines: 4,7-8,12-20,22-23

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:mybatis="http://mybatis.org/schema/mybatis-spring"
        xsi:schemaLocation="http://www.springframework.org/schema/beans 
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://mybatis.org/schema/mybatis-spring
            http://mybatis.org/schema/mybatis-spring.xsd">

        <import resource="classpath:/META-INF/spring/projectName-env.xml" />

        <!-- (1) -->
        <bean id="sqlSessionFactory"
            class="org.mybatis.spring.SqlSessionFactoryBean">
            <!-- (2) -->
            <property name="dataSource" ref="dataSource" />
            <!-- (3) -->
            <property name="configLocation"
                value="classpath:/META-INF/mybatis/mybatis-config.xml" />
        </bean>

        <!-- (4) -->
        <mybatis:scan base-package="com.example.domain.repository" />

    </beans>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80

   * - 項番
     - 説明
   * - (1)
     - \ ``SqlSessionFactory`` \を生成するためのコンポーネントとして、\ ``SqlSessionFactoryBean`` \をbean定義する。
   * - (2)
     - \ ``dataSource`` \プロパティに、設定済みのデータソースのbeanを指定する。

       MyBatis3の処理の中でSQLを発行する際は、ここで指定したデータソースからコネクションが取得される。
   * - (3)
     - \ ``configLocation`` \プロパティに、MyBatis設定ファイルのパスを指定する。

       ここで指定したファイルが\ ``SqlSessionFactory`` \を生成する時に読み込まれる。
   * - (4)
     - Mapperインタフェースをスキャンするために\ ``<mybatis:scan>`` \を定義し、\ ``base-package`` \属性には、
       Mapperインタフェースが格納されている基底パッケージを指定する。

       指定されたパッケージ配下に格納されている Mapperインタフェースがスキャンされ、
       スレッドセーフなMapperオブジェクト(MapperインタフェースのProxyオブジェクト)が自動的に生成される。

       **【指定するパッケージは、各プロジェクトで決められたパッケージにすること】**

 .. note:: **MyBatis3の設定方法について**

    \ ``SqlSessionFactoryBean`` \を使用する場合、MyBatis3の設定は、
    MyBatis設定ファイルではなくbeanのプロパティに直接指定することもできるが、
    本ガイドラインでは、MyBatis3自体の設定はMyBatis標準の設定ファイルに指定する方法を推奨する。

|

.. _DataAccessMyBatis3HowToUseSettingsMyBatis3:

MyBatis3の設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| MyBatis3では、MyBatis3の動作をカスタマイズするための仕組みが用意されている。
| MyBatis3の動作をカスタマイズする場合は、MyBatis設定ファイルに設定値を追加する事で実現可能である。

| ここでは、アプリケーションの特性に依存しない設定項目についてのみ、説明を行う。
| その他の設定項目に関しては、
 「\ `MyBatis 3 REFERENCE DOCUMENTATION(Configuration XML) <http://mybatis.github.io/mybatis-3/configuration.html>`_ \」を参照し、
 アプリケーションの特性にあった設定を行うこと。
| 基本的にはデフォルト値のままでも問題ないが、アプリケーションの特性を考慮し、必要に応じて設定を変更すること。

 .. note:: **MyBatis設定ファイルの格納場所について**
 
    本ガイドラインでは、MyBatis設定ファイルは、
    \ :file:`projectName-domain/src/main/resources/META-INF/mybatis/mybatis-config.xml`\ に格納することを推奨している。

    `MyBatis3用のブランクプロジェクト <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank#multi-blank-project-with-mybatis3>`_ \ からプロジェクトを生成した場合は、上記ファイルは格納済みの状態である。

|

.. _DataAccessMyBatis3HowToUseSettingsExecutorType:

SQL実行モードの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

MyBatis3では、SQLを実行するモードとして以下の3種類を用意している。

| どのモードを使用するかは、各モードの特性と制約、及び性能要件を考慮して決定して頂きたい。
| 実行モードの設定方法などについては、「:ref:`DataAccessMyBatis3HowToExtendExecutorType`」を参照されたい。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.65\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 15 65

    * - 項番
      - モード
      - 説明
    * - (1)
      - SIMPLE
      - SQL実行毎に新しい\ ``java.sql.PreparedStatement``\を作成する。

        MyBatisのデフォルトの動作であり、`MyBatis3用のブランクプロジェクト <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank#multi-blank-project-with-mybatis3>`_ \ も\ ``SIMPLE``\モードとなっている。
    * - (2)
      - REUSE
      - ``PreparedStatement``\をキャッシュし再利用する。

        同一トランザクション内で同じSQLを複数回実行する場合は、
        \ ``REUSE``\モードで実行すると、\ ``SIMPLE``\モードと比較して性能向上が期待できる。

        これは、SQLを解析して\ ``PreparedStatement``\を生成する処理の実行回数を減らす事ができるためである。
    * - (3)
      - BATCH
      - 更新系のSQLをバッチ実行する。(\ ``java.sql.Statement#executeBatch()``\を使ってSQLを実行する)。

        同一トランザクション内で更新系のSQLを連続して大量に実行する場合は、\ ``BATCH``\モードで実行すると、
        \ ``SIMPLE``\モードや\ ``REUSE``\モードと比較して性能向上が期待できる。

        これは、

        * SQLを解析して\ ``PreparedStatement``\を生成する処理の実行回数
        * サーバと通信する回数

        を減らす事ができるためである。

        ただし、\ ``BATCH``\モードを使用する場合は、MyBatisの動きが\ ``SIMPLE``\モードや\ ``SIMPLE``\モードと異なる部分がある。
        具体的な違いと注意点については、「:ref:`DataAccessMyBatis3HowToExtendExecutorTypeBatchNotes`」を参照されたい。

|

.. _DataAccessMyBatis3HowToUseSettingsTypeAlias:

TypeAliasの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

TypeAliasを使用すると、マッピングファイルで指定するJavaクラスに対して、エイリアス名(短縮名)を割り当てる事ができる。

TypeAliasを使用しない場合、マッピングファイルで指定する\ ``type`` \属性、\ ``parameterType`` \属性、\ ``resultType`` \属性などには、
Javaクラスの完全修飾クラス名(FQCN)を指定する必要があるため、マッピングファイルの記述効率の低下、記述ミスの増加などが懸念される。

本ガイドラインでは、記述効率の向上、記述ミスの削減、マッピングファイルの可読性向上などを目的として、TypeAliasを使用することを推奨する。

`MyBatis3用のブランクプロジェクト <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank#multi-blank-project-with-mybatis3>`_ \ からプロジェクトを生成した場合は、
Entityを格納するパッケージ(\ ``${projectPackage}.domain.model``\)配下に格納されるクラスがTypeAliasの対象となっている。
必要に応じて、設定を追加されたい。

TypeAliasの設定方法は以下の通り。

- :file:`projectName-domain/src/main/resources/META-INF/mybatis/mybatis-config.xml`

 .. code-block:: xml
    :emphasize-lines: 7-8

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
      PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
        <typeAliases>
            <!-- (1) -->
            <package name="com.example.domain.model" />
        </typeAliases>
    </configuration>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80

   * - 項番
     - 説明
   * - (1)
     - \ ``package`` \要素の\ ``name`` \属性に、エイリアスを設定するクラスが格納されているパッケージ名を指定する。
     
       指定したパッケージ配下に格納されているクラスは、パッケージの部分が除去された部分が、エイリアス名となる。
       上記例だと、``com.example.domain.model.Account`` \クラスのエイリアス名は、\ ``Account`` \となる。

       **【指定するパッケージは、各プロジェクトで決められたパッケージにすること】**


 .. tip:: **クラス単位にType Aliasを設定する方法について**
 
    Type Aliasの設定には、クラス単位に設定する方法やエイリアス名を明示的に指定する方法が用意されている。
    詳細は、Appendixの「:ref:`DataAccessMyBatis3AppendixSettingsTypeAlias`」を参照されたい。

|

TypeAliasを使用した際の、マッピングファイルの記述例は以下の通り。

 .. code-block:: xml
    :emphasize-lines: 8,13,19

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

    <mapper namespace="com.example.domain.repository.account.AccountRepository">

        <resultMap id="accountResultMap"
            type="Account">
            <!-- omitted -->
        </resultMap>

        <select id="findOne"
            parameterType="string"
            resultMap="accountResultMap">
            <!-- omitted -->
        </select>

        <select id="findByCriteria"
            parameterType="AccountSearchCriteria"
            resultMap="accountResultMap">
            <!-- omitted -->
        </select>

    </mapper>

 .. tip:: **MyBatis3標準のエイリアス名について**
 
    プリミティブ型やプリミティブラッパ型などの一般的なJavaクラスについては、予めエイリアス名が設定されている。

    予め設定されるエイリアス名については、
    「\ `MyBatis 3 REFERENCE DOCUMENTATION(Configuration XML-typeAliases-) <http://mybatis.github.io/mybatis-3/configuration.html#typeAliases>`_ \」を参照されたい。

|

.. _DataAccessMyBatis3HowToUseSettingsMappingNullAndJdbcType:

NULL値とJDBC型のマッピング設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 使用しているデータベース(JDBCドライバ)によっては、カラム値をnullに設定する際に、エラーが発生する場合がある。
| この事象は、JDBCドライバが\ ``null``\値の設定と認識できるJDBC型を指定する事で、解決する事ができる。

| \ ``null``\値を設定した際に、以下の様なスタックトレースを伴うエラーが発生した場合は、\ ``null``\値とJDBC型のマッピングが必要となる。
| MyBatis3のデフォルトでは、\ ``OTHER``\と呼ばれる汎用的なJDBC型が指定されるが、\ ``OTHER``\だとエラーとなるJDBCドライバもある。

 .. code-block:: guess
    :emphasize-lines: 1

    java.sql.SQLException: Invalid column type: 1111
        at oracle.jdbc.driver.OracleStatement.getInternalType(OracleStatement.java:3916) ~[ojdbc6-11.2.0.2.0.jar:11.2.0.2.0]
        at oracle.jdbc.driver.OraclePreparedStatement.setNullCritical(OraclePreparedStatement.java:4541) ~[ojdbc6-11.2.0.2.0.jar:11.2.0.2.0]
        at oracle.jdbc.driver.OraclePreparedStatement.setNull(OraclePreparedStatement.java:4523) ~[ojdbc6-11.2.0.2.0.jar:11.2.0.2.0]
        ...

 .. note:: **Oracle使用時の動作について**
 
    データベースにOracleを使用する場合は、デフォルトの設定のままだとエラーが発生する事が確認されている。
    バージョンによって動作がかわる可能性はあるが、Oracleを使う場合は、設定の変更が必要になる可能性がある事を記載しておく。

    エラーが発生する事が確認されているバージョンは、Oracle 11g R1で、JDBC型の\ ``NULL`` \型をマッピングするように設定を変更することで、
    エラーを解決する事できる。

|

以下に、MyBatis3のデフォルトの動作を変更する方法を示す。

- :file:`projectName-domain/src/main/resources/META-INF/mybatis/mybatis-config.xml`

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration PUBLIC "-//mybatis.org/DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>

        <settings>
            <!-- (1) -->
            <setting name="jdbcTypeForNull" value="NULL" />
        </settings>

    </configuration>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80

   * - 項番
     - 説明
   * - (1)
     - jdbcTypeForNullに、JDBC型を指定する。

       上記例では、\ ``null``\値のJDBC型として\ ``NULL``\型を指定している。



 .. tip:: **項目単位で解決する方法について**

    別の解決方法として、\ ``null``\値が設定される可能性があるプロパティのインラインパラメータに、
    Java型に対応する適切なJDBC型を個別に指定する方法もある。

    ただし、インラインパラメータで個別に指定した場合、マッピングファイルの記述量及び指定ミスが発生する可能性が増えることが予想されるため、
    本ガイドラインとしては、全体の設定でエラーを解決することを推奨している。
    全体の設定を変更してもエラーが解決しない場合は、エラーが発生するプロパティについてのみ、個別に設定を行えばよい。


|


.. _DataAccessMyBatis3HowToUseSettingsTypeHandler:

TypeHandlerの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

TypeHandlerは、JavaクラスとJDBC型をマッピングする時に使用される。

具体的には、

* SQLを発行する際に、Javaクラスのオブジェクトを\ ``java.sql.PreparedStatement`` \のバインドパラメータとして設定する
* SQLの発行結果として取得した\ ``java.sql.ResultSet`` \から値を取得する

際に、使用される。

プリミティブ型やプリミティブラッパ型などの一般的なJavaクラスについては、MyBatis3からTypeHandlerが提供されており、
特別な設定を行う必要はない。

 .. tip::

    MyBatis3から提供されているTypeHandlerについては、
    「\ `MyBatis 3 REFERENCE DOCUMENTATION(Configuration XML-typeHandlers-) <http://mybatis.github.io/mybatis-3/configuration.html#typeHandlers>`_ \」を参照されたい。

 .. tip:: **Enum型のマッピングについて**

    MyBatis3のデフォルトの動作では、Enum型はEnumの定数名(文字列)とマッピングされる。

    下記のようなEnum型の場合は、
    \ ``"WAITING_FOR_ACTIVE"`` \, \ ``"ACTIVE"`` \, \ ``"EXPIRED"`` \, \ ``"LOCKED"`` \
    という文字列とマッピングされてテーブルに格納される。

     .. code-block:: java

        package com.example.domain.model;

        public enum AccountStatus {
            WAITING_FOR_ACTIVE, ACTIVE, EXPIRED, LOCKED
        }

    MyBatisでは、Enum型を数値(定数の定義順)とマッピングする事もできる。数値とマッピングする方法については、
    「\ `MyBatis 3 REFERENCE DOCUMENTATION(Configuration XML-Handling Enums-) <http://mybatis.github.io/mybatis-3/configuration.html#Handling_Enums>`_ \」を参照されたい。


|

TypeHandlerの作成が必要になるケースは、MyBatis3でサポートしていないJavaクラスとJDBC型をマッピングする場合である。

具体的には、

* 容量の大きいファイルデータ(バイナリデータ)を\ ``java.io.InputStream`` \型で保持し、JDBC型の\ ``BLOB`` \型にマッピングする
* 容量の大きいテキストデータを\ ``java.io.Reader`` \型として保持し、JDBC型の\ ``CLOB`` \型にマッピングする
* 本ガイドラインで利用を推奨している「:doc:`Utilities/JodaTime`」の\ ``org.joda.time.DateTime`` \型と、JDBC型の\ ``TIMESTAMP`` \型をマッピングする
* etc ...

場合に、TypeHandlerの作成が必要となる。

上記にあげた3つのTypeHandlerの作成例については、
「:ref:`DataAccessMyBatis3HowToExtendTypeHandler`」を参照されたい。

|

ここでは、作成したTypeHandlerをMyBatisに適用する方法について説明を行う。

- :file:`projectName-domain/src/main/resources/META-INF/mybatis/mybatis-config.xml`

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration PUBLIC "-//mybatis.org/DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>

        <typeHandlers>
            <!-- (1) -->
            <package name="com.example.infra.mybatis.typehandler" />
        </typeHandlers>

    </configuration>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80

   * - 項番
     - 説明
   * - (1)
     - MyBatis設定ファイルにTypeHandlerの設定を行う。

       \ ``package``\要素のname 属性に、作成したTypeHandlerが格納されているパッケージ名を指定する。
       指定したパッケージ配下に格納されているTypeHandlerが、MyBatisによって自動検出される。

 .. tip::

    上記例では、指定したパッケージ配下に格納されているTypeHandlerをMyBatisによって自動検出させているが、
    クラス単位に設定する事もできる。

    クラス単位にTypeHandlerを設定する場合は、\ ``typeHandler``\要素を使用する。

    - :file:`projectName-domain/src/main/resources/META-INF/mybatis/mybatis-config.xml`

     .. code-block:: xml
        :emphasize-lines: 2

        <typeHandlers>
            <typeHandler handler="xxx.yyy.zzz.CustomTypeHandler" />
            <package name="com.example.infra.mybatis.typehandler" />
        </typeHandlers>

    |

    更に、TypeHandlerの中でDIコンテナで管理されているbeanを使用したい場合は、
    bean定義ファイル内でTypeHandlerを指定すればよい。

    - :file:`projectName-domain/src/main/resources/META-INF/spring/projectName-infra.xml`

     .. code-block:: xml
        :emphasize-lines: 16-20

        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
               xmlns:tx="http://www.springframework.org/schema/tx" xmlns:mybatis="http://mybatis.org/schema/mybatis-spring"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/tx
            http://www.springframework.org/schema/tx/spring-tx.xsd
            http://mybatis.org/schema/mybatis-spring
            http://mybatis.org/schema/mybatis-spring.xsd">

            <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
                <property name="dataSource" ref="oracleDataSource" />
                <property name="configLocation"
                    value="classpath:/META-INF/mybatis/mybatis-config.xml" />
                <property name="typeHandlers">
                    <list>
                        <bean class="xxx.yyy.zzz.CustomTypeHandler" />
                    </list>
                </property>
            </bean>

        </beans>

    |

    TypeHandlerを適用するJavaクラスとJDBC型のマッピングの指定は、

    * MyBatis設定ファイル内の\ ``typeHandler``\要素の属性値として指定
    * ``@org.apache.ibatis.type.MappedTypes``\アノテーションと\ ``@org.apache.ibatis.type.MappedJdbcTypes``\アノテーションに指定
    * MyBatis3から提供されているTypeHandlerの基底クラス(\ ``org.apache.ibatis.type.BaseTypeHandler``\)を継承することで指定

    する方法がある。

    詳しくは、「\ `MyBatis 3 REFERENCE DOCUMENTATION(Configuration XML-typeHandlers-) <http://mybatis.github.io/mybatis-3/configuration.html#typeHandlers>`_ \」を参照されたい。


 .. tip::

    上記の設定例は、いずれもアプリケーション全体に適用するための設定方法であったが、
    フィールド毎に個別のTypeHandlerを指定する事も可能である。
    これは、アプリケーション全体に適用しているTypeHandlerを上書きする際に使用する。

     .. code-block:: xml
        :emphasize-lines: 6-7,31-32

        <?xml version="1.0" encoding="UTF-8" ?>
        <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
        <mapper namespace="com.example.domain.repository.image.ImageRepository">
            <resultMap id="resultMapImage" type="Image">
                <id property="id" column="id" />
                <!-- (2) -->
                <result property="imageData" column="image_data" typeHandler="XxxBlobInputStreamTypeHandler" />
                <result property="createdAt" column="created_at"  />
            </resultMap>
            <select id="findOne" parameterType="string" resultMap="resultMapImage">
                SELECT
                    id
                    ,image_data
                    ,created_at
                FROM
                    t_image
                WHERE
                    id = #{id}
            </select>
            <insert id="create" parameterType="Image">
                INSERT INTO
                    t_image
                (
                    id
                    ,image_data
                    ,created_at
                )
                VALUES
                (
                    #{id}
                    /* (3) */
                    ,#{imageData,typeHandler=XxxBlobInputStreamTypeHandler}
                    ,#{createdAt}
                )
            </insert>
        </mapper>

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 80

        * - 項番
          - 説明
        * - (2)
          - 検索結果(\ ``ResultSet``\)から値を取得する際は、
            \ ``id``\又は\ ``result``\要素の\ ``typeHandler``\属性に適用するTypeHandlerを指定する。
        * - (3)
          - \ ``PreparedStatement``\に値を設定する際は、
            インラインパラメータの\ ``typeHandler``\属性に適用するTypeHandlerを指定する。

    TypeHandlerをフィールド毎に個別に指定する場合は、TypeHandlerのクラスにTypeAliasを設けることを推奨する。
    TypeAliasの設定方法については、「:ref:`DataAccessMyBatis3HowToUseSettingsTypeAlias`」を参照されたい。



|

.. _DataAccessMyBatis3HowToDababaseAccess:

データベースアクセス処理の実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

MyBatis3の機能を使用してデータベースにアクセスするための、具体的な実装方法について説明する。

.. _DataAccessMyBatis3HowToDababaseAccessCreateRepository:

Repositoryインタフェースの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Entity毎にRepositoryインタフェースを作成する。

 .. code-block:: java

    package com.example.domain.repository.todo;

    // (1)
    public interface TodoRepository {
    }
    
 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - JavaのインタフェースとしてRepositoryインタフェースを作成する。
      
        上記例では、\ ``Todo``\というEntityに対するRepositoryインタエースを作成している。

|

.. _DataAccessMyBatis3HowToDababaseAccessCreateMappingFile:

マッピングファイルの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Repositoryインタフェース毎にマッピングファイルを作成する。

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <!-- (1)  -->
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">
    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - \ ``mapper``\要素の\ ``namespace``\属性に、Repositoryインタフェースの完全修飾クラス名(FQCN)を指定する。

 .. note:: **マッピングファイルの格納先について**
 
    マッピングファイルの格納先は、
    
    * MyBatis3が自動的にマッピングファイルを読み込むために定めたルールに則ったディレクトリ
    * 任意のディレクトリ
    
    のどちらかを選択することができる。
    
    \ **本ガイドラインでは、MyBatis3が定めたルールに則ったディレクトリに格納し、マッピングファイルを自動的に読み込む仕組みを利用することを推奨する。**\
    
    マッピングファイルを自動的に読み込ませるためには、
    Repositoryインタフェースのパッケージ階層と同じ階層で、マッピングファイルをクラスパス上に格納する必要がある。
    
    具体的には、
    \ ``com.example.domain.repository.todo.TodoRepository``\というRepositoryインターフェースに対するマッピングファイル(\ :file:`TodoRepository.xml`\)は、
    \ ``projectName-domain/src/main/resources/com/example/domain/repository/todo``\ディレクトリに格納すればいよい。

|

.. _DataAccessMyBatis3HowToDababaseAccessCrud:

CRUD処理の実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
ここからは、基本的なCRUD処理の実装方法と、SQL実装時の考慮点について説明を行う。

基本的なCRUD処理として、以下の処理の実装方法について説明を行う。

* :ref:`DataAccessMyBatis3HowToUseResultSetMapping`
* :ref:`DataAccessMyBatis3HowToUseFind`
* :ref:`DataAccessMyBatis3HowToUseCreate`
* :ref:`DataAccessMyBatis3HowToUseUpdate`
* :ref:`DataAccessMyBatis3HowToUseDelete`
* :ref:`DataAccessMyBatis3HowToUseDynamicSql`

 .. note::

    MyBatis3を使用してCRUD処理を実装する際は、
    検索したEntityがローカルキャッシュと呼ばれる領域にキャッシュされる仕組みになっている点を意識しておく必要がある。

    MyBatis3が提供するローカルキャッシュのデフォルトの動作は以下の通りである。

    * ローカルキャッシュは、トランザクション単位で管理する。
    * Entityのキャッシュは、「ステートメントID + 組み立てられたSQLのパターン + 組み立てられたSQLにバインドするパラメータ値 + ページ位置(取得範囲)」毎に行う。

    つまり、同一トランザクション内の処理において、
    MyBatisが提供している検索APIを全て同じパラメータで呼び出すと、
    2回目以降はSQLを発行せずに、キャッシュされているEntityのインスタンスが返却される。

    ここでは、 **MyBatisのAPIが返却するEntityとローカルキャッシュで管理しているEntityが同じインスタンス** という点を意識しておいてほしい。

 .. tip::

    ローカルキャッシュは、ステートメント単位で管理するように変更する事もできる。
    ローカルキャッシュをステートメント単位で管理する場合、MyBatisは毎回SQLを実行して最新のEntityを取得する。

|

SQL実装時の考慮点として、以下の点について説明を行う。

* :ref:`DataAccessMyBatis3HowToUseLikeEscape`
* :ref:`DataAccessMyBatis3HowToUseSqlInjectionCountermeasure`

|

具体的な実装方法の説明を行う前に、以降の説明で登場するコンポーネントについて、簡単に説明しておく。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.25\linewidth}|p{0.55\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 25 55

    * - 項番
      - コンポーネント
      - 説明
    * - (1)
      - Entity
      - アプリケーションで扱う業務データを保持するJavaBeanクラス。
      
        Entityの詳細については、「:ref:`domainlayer_entity`」を参照されたい。
    * - (2)
      - Repositoryインタフェース
      - EntityのCRUD操作を行うためのメソッドを定義するインタフェース。
      
        Repositoryの詳細については、「:ref:`repository-label`」を参照されたい。
    * - (3)
      - Serviceクラス
      - 業務ロジックを実行するためのクラス。
      
        Serviceの詳細については、「:ref:`service-label`」を参照されたい。

 .. note::

    本ガイドラインでは、アーキテクチャ上の用語を統一するために、
    MyBatis3のMapperインタフェースの事をRepositoryインタフェースと呼んでいる。

以降の説明では、「:ref:`domainlayer_entity`」「:ref:`repository-label`」「:ref:`service-label`」を読んでいる前提で説明を行う。

|

.. _DataAccessMyBatis3HowToUseResultSetMapping:

検索結果とJavaBeanのマッピング方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Entityの検索処理の説明を行う前に、検索結果とJavaBeanのマッピング方法について説明を行う。

MyBatis3では、検索結果(\ ``ResultSet``\)をJavaBean(Entity)にマッピングする方法として、
自動マッピング と手動マッピングの2つの方法が用意されている。
それぞれ特徴があるので、\ **プロジェクトの特性やアプリケーションで実行するSQLの特性などを考慮して、使用するマッピング方法を決めて頂きたい。**\

 .. note:: **使用するマッピング方法について**

    本ガイドラインでは、

    * シンプルなマッピング(単一オブジェクトへのマッピング)の場合は自動マッピングを使用し、高度なマッピング(関連オブジェクトへのマッピング)が必要な場合は手動マッピングを使用する。
    * 一律手動マッピングを使用する

    の、２つの案を提示する。これは、上記2案のどちらかを選択する事を強制するものではなく、あくまで選択肢のひとつと考えて頂きたい。

    \ **アーキテクトは、自動マッピングと手動マッピングを使うケースの判断基準をプログラマに対して明確に示すことで、
    アプリケーション全体として統一されたマッピング方法が使用されるように心がけてほしい。**\

以下に、自動マッピングと手動マッピングに対して、それぞれの特徴と使用例を説明する。

|

.. _DataAccessMyBatis3HowToUseResultMappingByAuto:

検索結果の自動マッピング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

MyBatis3では、検索結果(\ ``ResultSet``\)のカラムとJavaBeanのプロパティをマッピングする方法として、
カラム名とプロパティ名を一致させることで、自動的に解決する仕組みを提供している。

 .. note:: **自動マッピングの特徴について**

    自動マッピングを使用すると、マッピングファイルには実行するSQLのみ記述すればよいため、
    マッピングファイルの記述量を減らすことができる点が特徴である。
    
    記述量が減ることで、単純ミスの削減や、カラム名やプロパティ名変更時の修正箇所の削減といった効果も期待できる。
    
    ただし、自動マッピングが行えるのは、単一オブジェクトに対するマッピングのみである。
    ネストした関連オブジェクトに対してマッピングを行いたい場合は、手動マッピングを使用する必要がある。

 .. tip:: **カラム名について**
 
     ここで言うカラム名とは、テーブルの物理的なカラム名ではなく、
     SQLを発行して取得した検索結果(\ ``ResultSet``\)がもつカラム名の事である。
     そのため、AS句を使うことで、物理的なカラム名とJavaBeanのプロパティ名を一致させることは、
     比較的容易に行うことができる。

|

以下に、自動マッピングを使用して検索結果をJavaBeanにマッピングする実装例を示す。

- :file:`projectName-domain/src/main/resources/com/example/domain/repository/todo/TodoRepository.xml`

 .. code-block:: xml
    :emphasize-lines: 8, 10

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">
    
        <select id="findOne" parameterType="string" resultType="Todo">
            SELECT
                todo_id AS "todoId", /* (1) */
                todo_title AS "todoTitle",
                finished, /* (2) */
                created_at AS "createdAt",
                version
            FROM
                t_todo
            WHERE
                todo_id = #{todoId}
        </select>
    
    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80

   * - 項番
     - 説明
   * - (1)
     - テーブルの物理カラム名とJavaBeanのプロパティ名が異なる場合は、AS句を使用して一致させることで、自動マッピング対象にすることができる。
   * - (2)
     - テーブルの物理カラム名とJavaBeanのプロパティ名が一致している場合は、AS句を指定する必要はない。

- JavaBean

 .. code-block:: java

    package com.example.domain.model;
    
    import java.io.Serializable;
    import java.util.Date;
    
    public class Todo implements Serializable {
    
        private static final long serialVersionUID = 1L;
    
        private String todoId;
    
        private String todoTitle;
    
        private boolean finished;
    
        private Date createdAt;
    
        private long version;
    
        public String getTodoId() {
            return todoId;
        }
    
        public void setTodoId(String todoId) {
            this.todoId = todoId;
        }
    
        public String getTodoTitle() {
            return todoTitle;
        }
    
        public void setTodoTitle(String todoTitle) {
            this.todoTitle = todoTitle;
        }
    
        public boolean isFinished() {
            return finished;
        }
    
        public void setFinished(boolean finished) {
            this.finished = finished;
        }
    
        public Date getCreatedAt() {
            return createdAt;
        }
    
        public void setCreatedAt(Date createdAt) {
            this.createdAt = createdAt;
        }
    
        public long getVersion() {
            return version;
        }
    
        public void setVersion(long version) {
            this.version = version;
        }
    
    }

 .. tip:: **アンダースコア区切りのカラム名とキャメルケース形式のプロパティ名のマッピング方法について**
 
         上記例では、アンダースコア区切りのカラム名とキャメルケース形式のプロパティ名の違いをAS句を使って吸収しているが、
         アンダースコア区切りのカラム名とキャメルケース形式のプロパティ名の違いを吸収するだけならば、
         MyBatis3の設定を変更する事で実現可能である。

|

テーブルの物理カラム名をアンダースコア区切りにしている場合は、
MyBatis設定ファイル(\ :file:`mybatis-config.xml`\)に以下の設定を追加することで、
キャメルケースのJavaBeanのプロパティに自動マッピングする事ができる。

- :file:`projectName-domain/src/main/resources/META-INF/mybatis/mybatis-config.xml`

 .. code-block:: xml
    :emphasize-lines: 8-9

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
      PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>

        <settings>
            <!-- (3) -->
            <setting name="mapUnderscoreToCamelCase" value="true" />
        </settings>

    </configuration>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (3)
      - \ ``mapUnderscoreToCamelCase`` \を\ `true`\にする設定を追加する。
      
        設定を\ `true`\にすると、アンダースコア区切りのカラム名がキャメルケース形式に自動変換される。
        具体例としては、カラム名が\ ``"todo_id"``\の場合、\ ``"todoId"``\に変換されてマッピングが行われる。

- :file:`projectName-domain/src/main/resources/com/example/domain/repository/todo/TodoRepository.xml`

 .. code-block:: xml
    :emphasize-lines: 8-12

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">
    
        <select id="findOne" parameterType="string" resultType="Todo">
            SELECT
                todo_id, /* (4) */
                todo_title,
                finished,
                created_at,
                version
            FROM
                t_todo
            WHERE
                todo_id = #{todoId}
        </select>
    
    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80

   * - 項番
     - 説明
   * - (4)
     - アンダースコア区切りのカラム名とキャメルケース形式のプロパティ名の違いを吸収するために、AS句の指定が不要になるため、よりシンプルなSQLとなる。

|

.. _DataAccessMyBatis3HowToUseResultMappingByManual:

検索結果の手動マッピング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

MyBatis3では、検索結果(\ ``ResultSet``\)のカラムとJavaBeanのプロパティの対応付けを、
マッピングファイルに定義する事で、手動で解決する仕組みを用意している。

 .. note:: **手動マッピングの特徴について**

    手動マッピングを使用すると、検索結果(\ ``ResultSet``\)のカラムとJavaBeanのプロパティの対応付けを、
    マッピングファイルに１項目ずつ定義することになる。
    そのため、マッピングの柔軟性が非常に高く、より複雑なマッピングを実現する事ができる点が特徴である。

    手動マッピングは、
    
     * アプリケーションが扱うデータモデル(JavaBean)と物理テーブルのレイアウトが一致しない
     * JavaBeanがネスト構造になっている(別のJavaBeanをネストしている)

    といったケースにおいて、検索結果(\ ``ResultSet``\)のカラムとJavaBeanのプロパティをマッピングする際に力を発揮するマッピング方法である。

    また、自動マッピングに比べて効率的にマッピングを行う事ができる。
    処理の効率性を優先するアプリケーションの場合は、自動マッピングの代わりに手動マッピングを使用した方がよい。

|

| 以下に、手動マッピングを使用して検索結果をJavaBeanにマッピングする実装例を示す。
| ここでは、手動マッピングの使用方法を示す事が目的なので、自動マッピングでもマッピング可能なもっともシンプルなパターンを例に、説明を行う。

実践的なマッピングの実装例については、

* 「\ `MyBatis 3 REFERENCE DOCUMENTATION(Mapper XML Files-Advanced Result Maps-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#Advanced_Result_Maps>`_ \」
* 「:ref:`DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnce`」
* 「:ref:`DataAccessMyBatis3AppendixNestedSelect`」

を参照されたい。

- :file:`projectName-domain/src/main/resources/com/example/domain/repository/todo/TodoRepository.xml`

 .. code-block:: xml
    :emphasize-lines: 6-7, 8-9, 10-14, 17-18

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <!-- (1) -->
        <resultMap id="todoResultMap" type="Todo">
            <!-- (2) -->
            <id column="todo_id" property="todoId" />
            <!-- (3) -->
            <result column="todo_title" property="todoTitle" />
            <result column="finished" property="finished" />
            <result column="created_at" property="createdAt" />
            <result column="version" property="version" />
        </resultMap>

        <!-- (4) -->
        <select id="findOne" parameterType="string" resultMap="todoResultMap">
            SELECT
                todo_id,
                todo_title,
                finished,
                created_at,
                version
            FROM
                t_todo
            WHERE
                todo_id = #{todoId}
        </select>

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - \ ``<resultMap>``\要素に、検索結果(\ ``ResultSet``\)とJavaBeanのマッピング定義を行う。
      
        \ ``id``\属性にマッピングを識別するためのIDを、\ ``type``\属性にマッピングするJavaBeanのクラス名(又はエイリアス名)を指定する。
        
        \ ``<resultMap>``\要素の詳細は、「\ `MyBatis 3 REFERENCE DOCUMENTATION(Mapper XML Files-resultMap-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#resultMap>`_ \」を参照されたい。
    * - (2)
      - 検索結果(\ ``ResultSet``\)のID(PK)のカラムとJavaBeanのプロパティのマッピングを行う。
      
        ID(PK)のマッピングは、\ ``<id>``\要素を使って指定する。
        \ ``column``\属性には検索結果(\ ``ResultSet``\)のカラム名、\ ``property``\属性にはJavaBeanのプロパティ名を指定する。
        
        \ ``<id>``\要素の詳細は、「\ `MyBatis 3 REFERENCE DOCUMENTATION(Mapper XML Files-id & result-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#id__result>`_ \」を参照されたい。
    * - (3)
      - 検索結果(\ ``ResultSet``\)のID(PK)以外のカラムとJavaBeanのプロパティのマッピングを行う。
      
        ID(PK)以外のマッピングは、\ ``<result>``\要素を使って指定する。
        \ ``column``\属性には検索結果(\ ``ResultSet``\)のカラム名、\ ``property``\属性にはJavaBeanのプロパティ名を指定する。

        \ ``<result>``\要素の詳細は、「\ `MyBatis 3 REFERENCE DOCUMENTATION(Mapper XML Files-id & result-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#id__result>`_ \」を参照されたい。
    * - (4)
      - \ ``<select>``\要素の\ ``resultMap``\属性に、適用するマッピング定義のIDを指定する。

 .. note:: **id要素とresult要素の使い分けについて**
 
    \ ``<id>``\要素と\ ``<result>``\要素は、
    どちらも検索結果(\ ``ResultSet``\)のカラムとJavaBeanのプロパティをマッピングするための要素であるが、
    ID(PK)カラムに対してマッピングは、\ ``<id>``\要素を使うことを推奨する。
    
    理由は、ID(PK)カラムに対して\ ``<id>``\要素を使用してマッピングを行うと、MyBatis3が提供しているオブジェクトのキャッシュ制御の処理や、
    関連オブジェクトへのマッピングの処理のパフォーマンスを、全体的に向上させることが出来るためである。

|

.. _DataAccessMyBatis3HowToUseFind:

Entityの検索処理
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Entityの検索処理の実装方法について、目的別に説明を行う。

Entityの検索処理の実装方法の説明を読む前に、「:ref:`DataAccessMyBatis3HowToUseResultSetMapping`」を一読して頂きたい。

以降の説明では、アンダースコア区切りのカラム名をキャメルケース形式のプロパティ名に自動でマッピングする設定を有効にした場合の実装例となる。

- :file:`projectName-domain/src/main/resources/META-INF/mybatis/mybatis-config.xml`

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
      PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>

        <settings>
            <setting name="mapUnderscoreToCamelCase" value="true" />
        </settings>

    </configuration>

|

.. _DataAccessMyBatis3HowToUseFindOne:

単一キーのEntityの取得
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
PKが単一カラムで構成されるテーブルより、PKを指定してEntityを1件取得する際の実装例を以下に示す。

* Repositoryインタフェースにメソッドを定義する。

 .. code-block:: java

    package com.example.domain.repository.todo;

    import com.example.domain.model.Todo;

    public interface TodoRepository {

        // (1)
        Todo findOne(String todoId);

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - 上記例では、引数に指定された\ ``todoId``\(PK)に一致するTodoオブジェクトを1件取得するためのメソッドとして、
        \ ``findOne``\メソッドを定義している。

|

* マッピングファイルにSQLを定義する。

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <!-- (2) -->
        <select id="findOne" parameterType="string" resultType="Todo">
            /* (3) */
            SELECT
                todo_id,
                todo_title,
                finished,
                created_at,
                version
            FROM
                t_todo
            /* (4) */
            WHERE
                todo_id = #{todoId}
        </select>

    </mapper>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 10 70

    * - 項番
      - 属性
      - 説明
    * - (2)
      - \-
      - \ ``select``\要素の中に、検索結果が0～1件となるSQLを実装する。
      
        上記例では、ID(PK)が一致するレコードを取得するSQLを実装している。

        \ ``select``\要素の詳細については、
        「`MyBatis3 REFERENCE DOCUMENTATION (Mapper XML Files-select-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#select>`_\」を参照されたい。

    * -
      - id
      - Repositoryインタフェースに定義したメソッドのメソッド名を指定する。
    * -
      - parameterType
      - パラメータ完全修飾クラス名(又はエイリアス名)を指定する。
    * -
      - resultType
      - 検索結果(\ ``ResultSet``\)をマッピングするJavaBeanの完全修飾クラス名(又はエイリアス名)を指定する。
      
        手動マッピングを使用する場合は、\ ``resultType``\属性の代わりに\ ``resultMap``\属性を使用して、
        適用するマッピング定義を指定する。
        手動マッピングについては、「:ref:`DataAccessMyBatis3HowToUseResultMappingByManual`」を参照されたい。
    * - (3)
      - \-
      - 取得対象のカラムを指定する。
      
        上記例では、検索結果(\ ``ResultSet``\)をJavaBeanへマッピングする方法として、自動マッピングを使用している。
        自動マッピングについては、「:ref:`DataAccessMyBatis3HowToUseResultMappingByAuto`」を参照されたい。
    * - (4)
      - \-
      - WHERE句に検索条件を指定する。
      
        検索条件にバインドする値は、\ ``#{variableName}``\形式のバインド変数として指定する。上記例では、
        \ ``#{todoId}``\がバインド変数となる。
        
        Repositoryインタフェースの引数の型が\ ``String``\のような単純型の場合は、
        バインド変数名は任意の名前でよいが、引数の型がJavaBeanの場合は、
        バインド変数名にはJavaBeanのプロパティ名を指定する必要がある。

 .. note:: **単純型のバインド変数名について**
 
    \ ``String``\のような単純型の場合は、バインド変数名に制約はないが、メソッドの引数名と同じ値にしておくことを推奨する。

|

* ServiceクラスにRepositoryをDIし、Repositoryインターフェースのメソッドを呼び出す。

 .. code-block:: java

    package com.example.domain.service.todo;

    import javax.inject.Inject;

    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;

    import com.example.domain.model.Todo;
    import com.example.domain.repository.todo.TodoRepository;

    @Transactional
    @Service
    public class TodoServiceImpl implements TodoService {

        // (5)
        @Inject
        TodoRepository todoRepository;

        @Transactional(readOnly = true)
        @Override
        public Todo getTodo(String todoId) {
            // (6)
            Todo todo = todoRepository.findOne(todoId);
            if (todo == null) { // (7)
                throw new ResourceNotFoundException(ResultMessages.error().add(
                        "e.ex.td.5001", todoId));
            }
            return todo;
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (5)
      - ServiceクラスにRepositoryインターフェースをDIする。
    * - (6)
      - Repositoryインターフェースのメソッドを呼び出し、Entityを1件取得する。
    * - (7)
      - 検索結果が0件の場合は\ ``null``\が返却されるため、
        必要に応じてEntityが取得できなかった時の処理を実装する。

        上記例では、Entityが取得できなかった場合は、リソース未検出エラーを発生させている。

|

複合キーのEntityの取得
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| PKが複数カラムで構成されるテーブルより、PKを指定してEntityを1件取得する際の実装例を以下に示す。
| 基本的な構成は、PKが単一カラムで構成される場合と同じであるが、Repositoryインタフェースのメソッド引数の指定方法が異なる。

* Repositoryインタフェースにメソッドを定義する。

 .. code-block:: java

    package com.example.domain.repository.order;
    
    import org.apache.ibatis.annotations.Param;
    
    import com.example.domain.model.OrderHistory;
    
    public interface OrderHistoryRepository {
    
       // (1)
       OrderHistory findOne(@Param("orderId") String orderId,
               @Param("historyId") int historyId);
    
    }
   
 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - PKを構成するカラムに対応する引数を、メソッドに定義する。

        上記例では、受注の変更履歴を管理するテーブルのPKとして、\ ``orderId``\と\ ``historyId``\を引数に定義している。

 .. tip:: **メソッド引数を複数指定する場合のバインド変数名について**
 
    Repositoryインタフェースのメソッド引数を複数指定する場合は、引数に\ ``@org.apache.ibatis.annotations.Param``\アノテーションを指定することを推奨する。
    \ ``@Param``\アノテーションの\ ``value``\属性には、マッピングファイルから値を参照する際に指定する「バインド変数名」を指定する。
     
    上記例だと、マッピングファイルから\ ``#{orderId}``\及び\ ``#{historyId}``\と指定することで、引数に指定された値をSQLにバインドする事ができる。

     .. code-block:: xml
    
        <?xml version="1.0" encoding="UTF-8"?>
        <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
        <mapper namespace="com.example.domain.repository.order.OrderHistoryRepository">
    
            <select id="findOne" resultType="OrderHistory">
                SELECT
                    order_id,
                    history_id,
                    order_name,
                    operation_type,
                    created_at"
                FROM
                    t_order_history
                WHERE
                    order_id = #{orderId}
                AND
                    history_id = #{historyId}
            </select>
            
        </mapper>

    \ ``@Param``\アノテーションの指定は必須ではないが、
    指定しないと以下に示すような機械的なバインド変数名を指定する必要がある。
    \ ``@Param``\アノテーションの指定しない場合のバインド変数名は、「"param" + 引数の宣言位置(1から開始)」という名前になるため、
    ソースコードのメンテナンス性及び可読性を損なう要因となる。
    
     .. code-block:: xml
    
        <!-- omitted -->
    
        WHERE
            order_id = #{param1}
        AND
            history_id = #{param2}

        <!-- omitted -->

|

.. _DataAccessMyBatis3HowToUseFindMultiple:

Entityの検索
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
検索結果が0～N件となるSQLを発行し、Entityを複数件取得する際の実装例を以下に示す。

 .. warning::

    検索結果が大量のデータになる可能性がある場合は、「:ref:`DataAccessMyBatis3HowToExtendResultHandler`」の利用を検討すること。

|

* Entityを複数件取得するためのメソッドを定義する。

 .. code-block:: java

    package com.example.domain.repository.todo;

    import java.util.List;

    import com.example.domain.model.Todo;

    public interface TodoRepository {

        // (1)
        List<Todo> findAllByCriteria(TodoCriteria criteria);

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - 上記例では、検索条件を保持するJavaBean(\ ``TodoCriteria``\)に一致するTodoオブジェクトをリスト形式で複数件取得するためのメソッドとして、
        \ ``findAllByCriteria``\メソッドを定義している。

 .. tip::

    上記例では、メソッドの返り値に\ ``java.util.List``\を指定しているが、
    検索結果を\ ``java.util.Map``\として受け取る事も出来る。

    \ ``Map``\で受け取る場合は、

    * \ ``Map``\の\ ``key``\にはPKの値
    * \ ``Map``\の\ ``value``\にはEntityオブジェクト

    を格納する事になる。

    検索結果を\ ``Map``\で受け取る場合、\ ``java.util.HashMap``\のインスタンスが返却されるため、
    \ ``Map``\の並び順は保証されないという点に注意すること。

    以下に、実装例を示す。

     .. code-block:: java

        package com.example.domain.repository.todo;

        import java.util.Map;

        import com.example.domain.model.Todo;
        import org.apache.ibatis.annotations.MapKey;

        public interface TodoRepository {

            @MapKey("todoId")
            Map<String, Todo> findAllByCriteria(TodoCriteria criteria);

        }

    検索結果を\ ``Map``\で受け取る場合は、\ ``@org.apache.ibatis.annotations.MapKey``\アノテーションをメソッドに指定する。
    アノテーションの\ ``value``\属性には、\ ``Map``\の\ ``key``\として扱うプロパティ名を指定する。
    上記例では、TodoオブジェクトのPK(\ ``todoId``\)を指定している。



|

* 検索条件を保持するJavaBeanを作成する。

 .. code-block:: java

    package com.example.domain.repository.todo;

    import java.io.Serializable;
    import java.util.Date;

    public class TodoCriteria implements Serializable {

        private static final long serialVersionUID = 1L;

        private String title;

        private Date createdAt;

        public String getTitle() {
            return title;
        }

        public void setTitle(String title) {
            this.title = title;
        }

        public Date getCreatedAt() {
            return createdAt;
        }

        public void setCreatedAt(Date createdAt) {
            this.createdAt = createdAt;
        }

    }

 .. note:: **検索条件を保持するためのJavaBeanの作成について**

    検索条件を保持するためのJavaBeanの作成は必須ではないが、格納されている値の役割が明確になるため、
    JavaBeanを作成することを推奨する。ただし、JavaBeanを作成しない方法で実装してもよい。
    
    \ **アーキテクトは、JavaBeanを作成するケースと作成しないケースの判断基準をプログラマに対して明確に示すことで、
    アプリケーション全体として統一された作りになるようにすること。**\

    JavaBeanを作成しない場合の実装例を以下に示す。

     .. code-block:: java

        package com.example.domain.repository.todo;

        import java.util.List;

        import com.example.domain.model.Todo;

        public interface TodoRepository {

            List<Todo> findAllByCriteria(@Param("title") String title,
                    @Param("createdAt") Date createdAt);

        }

    JavaBeanを作成しない場合は、検索条件を1項目ずつ引数に宣言し、
    \ ``@Param``\アノテーションの\ ``value``\属性に「バインド変数名」を指定する。
    上記のようなメソッドを定義することで、複数の検索条件をSQLに引き渡すことができる。

|

* マッピングファイルにSQLを定義する。

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <!-- (2) -->
        <select id="findAllByCriteria" parameterType="TodoCriteria" resultType="Todo">
            <![CDATA[
            SELECT
                todo_id,
                todo_title,
                finished,
                created_at,
                version
            FROM
                t_todo
            WHERE
                todo_title LIKE #{title} || '%' ESCAPE '~'
            AND
                created_at < #{createdAt}
            /* (3) */
            ORDER BY
                todo_id
            ]]>
        </select>

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (2)
      - \ ``select``\要素の中に、検索結果が0～N件となるSQLを実装する。
      
        上記例では、\ ``todo_title``\と\ ``created_at``\が指定した条件に一致するTodoレコードを取得する実装している。
    * - (3)
      - ソート条件を指定する。
      
        複数件のレコードを取得する場合は、ソート条件を指定する。
        特に画面に表示するレコードを取得するSQLでは、ソート条件の指定は必須である。

 .. tip:: **CDATAセクションの活用方法について**
 
    SQL内にXMLのエスケープが必要な文字(\ ``"<"``\や\ ``">"``\など)を指定する場合は、
    CDATAセクションを使用すると、SQLの可読性を保つことができる。
    CDATAセクションを使用しない場合は、\ ``"&lt;"``\や\ ``"&gt;"``\といったエンティティ参照文字を指定する必要があり、
    SQLの可読性を損なう要因となる。
    
    上記例では、\ ``created_at``\に対する条件として\ ``"<"``\を使用しているため、CDATAセクションを指定している。

|

.. _DataAccessMyBatis3HowToUseCount:

Entityの件数の取得
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
検索条件に一致するEntityの件数を取得する際の実装例を以下に示す。

* 検索条件に一致するEntityの件数を取得するためのメソッドを定義する。

 .. code-block:: java

    package com.example.domain.repository.todo;

    public interface TodoRepository {

        // (1)
        long countByFinished(boolean finished);

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - 件数を取得ためのメソッドの返り値は、数値型(\ ``int``\や\ ``long``\など)を指定する。
      
        上記例では、\ ``long``\を指定している。

|

* マッピングファイルにSQLを定義する。

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <!-- (2) -->
        <select id="countByFinished" parameterType="_boolean" resultType="_long">
            SELECT
                COUNT(*)
            FROM
                t_todo
            WHERE
                finished = #{finished}
        </select>

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (2)
      - 件数を取得するSQLを実行する。
      
        \ ``resultType``\属性には、返り値の型を指定する。
      
        上記例では、プリミティブ型の\ ``long``\を指定するためのエイリアス名を指定している。

 .. tip:: **プリミティブ型のエイリアス名について**
 
    プリミティブ型のエイリアス名は、先頭に\ ``"_"``\(アンダースコア)を指定する必要がある。
    \ ``"_"``\(アンダースコア)を指定しない場合は、プリミティブのラッパ型(\ ``java.lang.Long``\など)として扱われる。

|

.. _DataAccessMyBatis3HowToUseFindPageUsingMyBatisFunction:

Entityのページネーション検索(MyBatis3標準方式)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
MyBatis3の取得範囲指定機能を使用してEntityを検索する際の実装例を以下に示す。

MyBatisでは取得範囲を指定するクラスとして\ ``org.apache.ibatis.session.RowBounds``\クラスを用意されており、
SQLに取得範囲の条件を記述する必要がない。

 .. warning:: **検索条件に一致するデータ件数が多くなる場合の注意点について**
 
    MyBatis3標準の方式は、検索結果(\ ``ResultSet``\)のカーソルを移動することで、取得範囲外のデータをスキップする方式である。
    そのため、検索条件に一致するデータ件数に比例して、メモリ枯渇やカーソル移動処理の性能劣化が発生する可能性が高くなる。

    カーソルの移動処理は、JDBCの結果セット型に応じて以下の２種類がサポートされており、デフォルトの動作は、
    JDBCドライバのデフォルトの結果セット型に依存する。

    * 結果セット型が\ ``FORWARD_ONLY``\の場合は、\ ``ResultSet#next()``\を繰返し呼び出して取得範囲外のデータをスキップする。
    * 結果セット型が\ ``SCROLL_SENSITIVE``\又は\ ``SCROLL_INSENSITIVE``\の場合は、\ ``ResultSet#absolute(int)``\を呼び出して取得範囲外のデータをスキップする。

    \ ``ResultSet#absolute(int)``\を使用することで、性能劣化を最小限に抑える事ができる可能性はあるが、
    JDBCドライバの実装次第であり、内部で\ ``ResultSet#next()``\と同等の処理が行われている場合は、
    メモリ枯渇や性能劣化が発生する可能性を抑える事はできない。

    \ **検索条件に一致するデータ件数が多くなる可能性がある場合は、MyBatis3標準方式のページネーション検索ではなく、
    SQL絞り込み方式の採用を検討した方がよい。**\

|

* Entityのページネーション検索を行うためのメソッドを定義する。

 .. code-block:: java

    ackage com.example.domain.repository.todo;

    import java.util.List;

    import org.apache.ibatis.session.RowBounds;

    import com.example.domain.model.Todo;

    public interface TodoRepository {

        // (1)
        long countByCriteria(TodoCriteria criteria);

        // (2)
        List<Todo> findPageByCriteria(TodoCriteria criteria,
            RowBounds rowBounds);

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - 検索条件に一致するEntityの総件数を取得するメソッドを定義する。
    * - (2)
      - 検索条件に一致するEntityの中から、取得範囲のEntityを抽出メソッドを定義する。
      
        定義したメソッドの引数として、取得範囲の情報(offsetとlimit)を保持する\ ``RowBounds``\を指定する。

|

* マッピングファイルにSQLを定義する。

  検索結果から該当範囲のレコードを抽出する処理は、MyBatis3が行うため、SQLで取得範囲のレコードを絞り込む必要がない。

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <select id="countByCriteria" parameterType="TodoCriteria" resultType="_long">
            <![CDATA[
            SELECT
                COUNT(*)
            FROM
                t_todo
            WHERE
                todo_title LIKE #{title} || '%' ESCAPE '~'
            AND
                created_at < #{createdAt}
            ]]>
        </select>

        <select id="findPageByCriteria" parameterType="TodoCriteria" resultType="Todo">
            <![CDATA[
            SELECT
                todo_id,
                todo_title,
                finished,
                created_at,
                version
            FROM
                t_todo
            WHERE
                todo_title LIKE #{title} || '%' ESCAPE '~'
            AND
                created_at < #{createdAt}
            ORDER BY
                todo_id
            ]]>
        </select>

    </mapper>

 .. note:: **WHERE句の共通化について**
 
    ページネーション検索を実現する場合、「検索条件に一致するEntityの総件数を取得するSQL」と
    「 検索条件に一致するEntityのリストを取得するSQL」で指定するWHERE句は、
    MyBatis3のinclude機能を使って共通化することを推奨する。
    
    上記SQLのWHERE句を共通化した場合、以下のような定義となる。
    詳細は、「:ref:`DataAccessMyBatis3HowToExtendSqlShare`」を参照されたい。

     .. code-block:: xml
        :emphasize-lines: 1, 15, 27

        <sql id="findPageByCriteriaWherePhrase">
            <![CDATA[
            WHERE
                todo_title LIKE #{title} || '%' ESCAPE '~'
            AND
                created_at < #{createdAt}
            ]]>
        </sql>

        <select id="countByCriteria" parameterType="TodoCriteria" resultType="_long">
            SELECT
                COUNT(*)
            FROM
                t_todo
            <include refid="findPageByCriteriaWherePhrase"/>
        </select>

        <select id="findPageByCriteria" parameterType="TodoCriteria" resultType="Todo">
            SELECT
                todo_id,
                todo_title,
                finished,
                created_at,
                version
            FROM
                t_todo
            <include refid="findPageByCriteriaWherePhrase"/>
            ORDER BY
                todo_id
        </select>

 .. note:: **結果セット型を明示的に指定する方法について**

    結果セット型を明示的に指定する場合は、\ ``resultType``\属性に結果セット型を指定する。
    JDBCドライバのデフォルトの結果セット型が、\ ``FORWARD_ONLY``\の場合は、\ ``SCROLL_INSENSITIVE``\を指定することを推奨する。

     .. code-block:: xml
        :emphasize-lines: 2

        <select id="findPageByCriteria" parameterType="TodoCriteria" resultType="Todo"
            resultSetType="SCROLL_INSENSITIVE">
            <!-- omitted -->
        </select>

|

* Serviceクラスにページネーション検索処理を実装する。

 .. code-block:: java

    // omitted

    @Transactional
    @Service
    public class TodoServiceImpl implements TodoService {
    
        @Inject
        TodoRepository todoRepository;
        
        // omitted

        @Transactional(readOnly = true)
        @Override
        public Page<Todo> searchTodos(TodoCriteria criteria, Pageable pageable) {
            // (3)
            long total = todoRepository.countByCriteria(criteria);
            List<Todo> todos;
            if (0 < total) {
                // (4)
                RowBounds rowBounds = new RowBounds(pageable.getOffset(), 
                    pageable.getPageSize());
                // (5)
                todos = todoRepository.findPageByCriteria(criteria, rowBounds);
            } else {
                // (6)
                todos = Collections.emptyList();
            }
            // (7)
            return new PageImpl<>(todos, pageable, total);
        }

        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (3)
      - まず、検索条件に一致するEntityの総件数を取得する。
    * - (4)
      - 検索条件に一致するEntityが存在する場合は、ページネーション検索の取得範囲を指定する\ ``RowBounds``\オブジェクトを生成する。

        \ ``RowBounds``\の第1引数(\ ``offset``\)には「スキップ件数」、
        第２引数(\ ``limit``\)には「最大取得件数」を指定する。
        引数に指定する値、Spring Data Commonsから提供されている\ ``Pageable``\オブジェクトの
        \ ``getOffset``\メソッドと\ ``getPageSize``\メソッドを呼び出して取得した値を指定すればよい。

        具体的には、

        * offsetに\ ``0``\、limitに\ ``20``\を指定した場合、1～20件目
        * offsetに\ ``20``\、limitに\ ``20``\を指定した場合、21～40件目

        が取得範囲となる。

    * - (5)
      - Repositoryのメソッドを呼び出し、検索条件に一致した取得範囲のEntityを取得する。
    * - (6)
      - 検索条件に一致するEntityが存在しない場合は、空のリストを検索結果に設定する。
    * - (7)
      - ページ情報(\ ``org.springframework.data.domain.PageImpl``\)を作成し返却する。

|

.. _DataAccessMyBatis3HowToUseFindPageUsingSqlFilter:

Entityのページネーション検索(SQL絞り込み方式)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
データベースから提供されている範囲検索の仕組みを使用してEntityを検索する際の実装例を以下に示す。

SQL絞り込み方式は、データベースから提供されている範囲検索の仕組みを使用するため、
MyBatis3標準方式に比べて効率的に取得範囲のEntityを取得することができる。

 .. note::

    \ **検索条件に一致するデータ件数が大量にある場合は、SQL絞り込み方式を採用する事を推奨する。**\ 

|

* Entityのページネーション検索を行うためのメソッドを定義する。

 .. code-block:: java

    package com.example.domain.repository.todo;
    
    import java.util.List;
    
    import org.apache.ibatis.annotations.Param;
    import org.springframework.data.domain.Pageable;
    
    import com.example.domain.model.Todo;
    
    public interface TodoRepository {
    
        // (1)
        long countByCriteria(
                @Param("criteria") TodoCriteria criteria);

        // (2)
        List<Todo> findPageByCriteria(
                @Param("criteria") TodoCriteria criteria,
                @Param("pageable") Pageable pageable);
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - 検索条件に一致するEntityの総件数を取得するメソッドを定義する。
    * - (2)
      - 検索条件に一致するEntityの中から、取得範囲のEntityを抽出メソッドを定義する。

        定義したメソッドの引数として、取得範囲の情報(offsetとlimit)を保持する\ ``org.springframework.data.domain.Pageable``\を指定する。

 .. note:: **引数が1つのメソッドに@Paramアノテーションを指定する理由について**
 
     上記例では、引数が1つのメソッド(\ ``countByCriteria``\)に対して\ ``@Param``\アノテーションを指定している。
     これは、\ ``findPageByCriteria``\メソッド呼び出し時に実行されるSQLとWHERE句を共通化するためである。
     
     \ ``@Param``\アノテーションを使用して引数にバインド変数名を指定することで、
     SQL内で指定するバインド変数名のネスト構造を合わせている。
     
     具体的なSQLの実装例については、次に示す。

|

* マッピングファイルにSQLを定義する。

  SQLで取得範囲のレコードを絞り込む。

 .. code-block:: xml
    :emphasize-lines: 8, 36-37, 38-39

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <sql id="findPageByCriteriaWherePhrase">
            <![CDATA[
            /* (3) */
            WHERE
                todo_title LIKE #{criteria.title} || '%' ESCAPE '~'
            AND
                created_at < #{criteria.createdAt}
            ]]>
        </sql>
    
        <select id="countByCriteria" resultType="_long">
            SELECT
                COUNT(*)
            FROM
                t_todo
            <include refid="findPageByCriteriaWherePhrase" />
        </select>
    
        <select id="findPageByCriteria" resultType="Todo">
            SELECT
                todo_id,
                todo_title,
                finished,
                created_at,
                version
            FROM
                t_todo
            <include refid="findPageByCriteriaWherePhrase" />
            ORDER BY
                todo_id
            LIMIT
                #{pageable.pageSize} /* (4) */
            OFFSET
                #{pageable.offset}  /* (4) */
        </select>

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (3)
      - \ ``countByCriteria``\と\ ``findPageByCriteria``\メソッドの引数に\ ``@Param("criteria")``\を指定しているため、
        SQL内で指定するバインド変数名は\ ``criteria.フィールド名``\の形式となる。
    * - (4)
      - データベースから提供されている範囲検索の仕組みを使用して、必要なレコードのみ抽出する。
      
        \ ``Pageable``\オブジェクトの\ ``offset``\には「スキップ件数」、
        \ ``pageSize``\には「最大取得件数」が格納されている。

        上記例は、データベースとしてH2 Databaseを使用した際の実装例である。

|

* Serviceクラスにページネーション検索処理を実装する。

 .. code-block:: java

    // omitted

    @Transactional
    @Service
    public class TodoServiceImpl implements TodoService {
    
        @Inject
        TodoRepository todoRepository;
        
        // omitted

        @Transactional(readOnly = true)
        @Override
        public Page<Todo> searchTodos(TodoCriteria criteria,
                Pageable pageable) {
            long total = todoRepository.countByCriteria(criteria);
            List<Todo> todos;
            if (0 < total) {
                // (5)
                todos = todoRepository.findPageByCriteria(criteria,
                        pageable);
            } else {
                todos = Collections.emptyList();
            }
            return new PageImpl<>(todos, pageable, total);
        }

        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (5)
      - Repositoryのメソッドを呼び出し、検索条件に一致した取得範囲のEntityを取得する。
      
        Repositoryのメソッドを呼び出す際は、引数で受け取った\ ``Pageable``\オブジェクトをそのまま渡せばよい。

|

.. _DataAccessMyBatis3HowToUseCreate:

Entityの登録処理
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Entityの登録方法について、目的別に実装例を説明する。

.. _DataAccessMyBatis3HowToUseCreateOne:

Entityの1件登録
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Entityを1件登録する際の実装例を以下に示す。

* Repositoryインタフェースにメソッドを定義する。

 .. code-block:: java

    package com.example.domain.repository.todo;
    
    import com.example.domain.model.Todo;
    
    public interface TodoRepository {
    
        // (1)
        void create(Todo todo);
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - 上記例では、引数に指定されたTodoオブジェクトを1件登録するためのメソッドとして、
        \ ``create``\メソッドを定義している。

\

 .. note:: **Entityを登録するメソッドの返り値について**
 
    Entityを登録するメソッドの返り値は、基本的には\ ``void``\でよい。

    ただし、SELECTした結果をINSERTするようなSQLを発行する場合は、
    アプリケーション要件に応じて\ ``boolean``\や数値型(\ ``int``\又は\ ``long``\)を返り値とすること。

    * 返り値として\ ``boolean``\を指定した場合は、登録件数が0件の際は\ ``false``\、登録件数が1件以上の際は\ ``true``\が返却される。
    * 返り値として数値型を指定した場合は、登録件数が返却される。

|

* マッピングファイルにSQLを定義する。

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <!-- (2) -->
        <insert id="create" parameterType="Todo">
            INSERT INTO
                t_todo
            (
                todo_id,
                todo_title,
                finished,
                created_at,
                version
            )
            /* (3) */
            VALUES
            (
                #{todoId},
                #{todoTitle},
                #{finished},
                #{createdAt},
                #{version}
            )
        </insert>

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (2)
      - insert要素の中に、INSERTするSQLを実装する。
      
        \ ``id``\属性には、Repositoryインタフェースに定義したメソッドのメソッド名を指定する。

        \ ``insert``\要素の詳細については、
        「`MyBatis3 REFERENCE DOCUMENTATION (Mapper XML Files-insert, update and delete-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#insert_update_and_delete>`_\」を参照されたい。

    * - (3)
      - VALUE句にレコード登録時の設定値を指定する。
      
        VALUE句にバインドする値は、#{variableName}形式のバインド変数として指定する。
        上記例では、Repositoryインタフェースの引数としてJavaBean(\ ``Todo``\)を指定しているため、
        バインド変数名にはJavaBeanのプロパティ名を指定する。

|

* ServiceクラスにRepositoryをDIし、Repositoryインターフェースのメソッドを呼び出す。

 .. code-block:: java


    package com.example.domain.service.todo;

    import java.util.UUID;

    import javax.inject.Inject;

    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;
    import org.terasoluna.gfw.common.date.jodatime.JodaTimeDateFactory;

    import com.example.domain.model.Todo;
    import com.example.domain.repository.todo.TodoRepository;

    @Transactional
    @Service
    public class TodoServiceImpl implements TodoService {

        // (4)
        @Inject
        TodoRepository todoRepository;

        @Inject
        JodaTimeDateFactory dateFactory;

        @Override
        public Todo create(Todo todo) {
            // (5)
            todo.setTodoId(UUID.randomUUID().toString());
            todo.setCreatedAt(dateFactory.newDate());
            todo.setFinished(false);
            todo.setVersion(1);
            // (6)
            todoRepository.create(todo);
            // (7)
            return todo;
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (4)
      - ServiceクラスにRepositoryインターフェースをDIする。
    * - (5)
      - 引数で渡されたEntityオブジェクトに対して、アプリケーション要件に応じて値を設定する。

        上記例では、

        * IDとして「UUID」
        * 登録日時として「システム日時」
        * 完了フラグに「\ ``false``\ : 未完了」
        * バージョンに「\ ``1``\」
        
        を設定している。
    * - (6)
      - Repositoryインターフェースのメソッドを呼び出し、Entityを1件登録する。
    * - (7)
      - 登録したEntityを返却する。
      
        Serviceクラスの処理で登録値を設定する場合は、登録したEntityオブジェクトを返り値として返却する事を推奨する。

|


.. _DataAccessMyBatis3HowToUseGenId:

キーの生成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

「:ref:`DataAccessMyBatis3HowToUseCreateOne`」では、
Serviceクラスでキー(ID)の生成をする実装例になっているが、
MyBatis3では、マッピングファイル内でキーを生成する仕組みが用意されている。

 .. note:: **MyBatis3のキー生成機能の使用ケースについて**
 
    キーを生成するために、データベースの機能(関数やID列など)を使用する場合は、
    MyBatis3のキー生成機能の仕組みを使用する事を推奨する。

|

キーの生成方法は、2種類用意されている。

* データベースから用意されている関数などを呼び出した結果をキーとして扱う方法
* データベースから用意されているID列(IDENTITY型、AUTO_INCREMENT型など) + JDBC3.0から追加された\ ``Statement#getGeneratedKeys()``\を呼び出した結果をキーとして扱う方法

|

まず、データベースから用意されている関数などを呼び出した結果をキーとして扱う方法について説明する。
下記例は、データベースとしてH2 Databaseを使用している。

 .. code-block:: xml
    :emphasize-lines: 7-11

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <insert id="create" parameterType="Todo">
            <!-- (1) -->
            <selectKey keyProperty="todoId" resultType="string" order="BEFORE">
                /* (2) */
                SELECT RANDOM_UUID()
            </selectKey>
            INSERT INTO
                t_todo
            (
                todo_id,
                todo_title,
                finished,
                created_at,
                version
            )
            VALUES
            (
                #{todoId},
                #{todoTitle},
                #{finished},
                #{createdAt},
                #{version}
            )
        </insert>

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 10 70

    * - 項番
      - 属性
      - 説明
    * - (1)
      - \-
      - \ ``selectKey``\要素の中に、キーを生成するためのSQLを実装する。

        上記例では、データベースから提供されている関数を使用してUUIDを取得している。

        \ ``selectKey``\の詳細については、
        「`MyBatis3 REFERENCE DOCUMENTATION (Mapper XML Files-insert, update and delete-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#insert_update_and_delete>`_\」を参照されたい。
    * -
      - keyProperty
      - 取得したキー値を格納するEntityのプロパティ名を指定する。

        上記例では、Entityの\ ``todoId``\プロパティに生成したキーが設定される。
    * -
      - resultType
      - SQLを発行して取得するキー値の型を指定する。
    * -
      - order
      - キー生成用SQLを実行するタイミング(\ ``BEFORE``\又は\ ``AFTER``\)を指定する。

        * \ ``BEFORE``\を指定した場合、\ ``selectKey``\要素で指定したSQLを実行した結果をEntityに反映した後にINSERT文が実行される。
        * \ ``AFTER``\を指定した場合、INSERT文を実行した後に\ ``selectKey``\要素で指定したSQLを実行され、取得した値がEntityに反映される。
    * - (2)
      - \-
      - \ キーを生成するためのSQLを実装する。

        上記例では、H2 DatabaseのUUIDを生成する関数を呼び出して、キーを生成している。
        キー生成の代表的な実装としては、シーケンスオブジェクトから取得した値を文字列にフォーマットする実装があげられる。

|

次に、データベースから用意されているID列 + JDBC3.0から追加された\ ``Statement#getGeneratedKeys()``\を呼び出した結果をキーとして扱う方法について説明する。
下記例は、データベースとしてH2 Databaseを使用している。

 .. code-block:: xml
    :emphasize-lines: 6-7

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.audit.AuditLogRepository">

        <!-- (3) -->
        <insert id="create" parameterType="Todo" useGeneratedKeys="true" keyProperty="logId">
            INSERT INTO
                t_audit_log
            (
                level,
                message,
                created_at,
            )
            VALUES
            (
                #{level},
                #{message},
                #{createdAt},
            )
        </insert>
        
    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 10 70

    * - 項番
      - 属性
      - 説明
    * - (3)
      - useGeneratedKeys
      - \ ``true``\を指定すると、ID列+\ ``Statement#getGeneratedKeys()``\を呼び出してキーを取得する機能が利用可能となる。

        \ ``useGeneratedKeys``\の詳細については、
        「`MyBatis3 REFERENCE DOCUMENTATION (Mapper XML Files-insert, update and delete-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#insert_update_and_delete>`_\」を参照されたい。
    * -
      - keyProperty
      - データベース上で自動でインクリメントされたキー値を格納するEntityのプロパティ名を指定する。

        上記例では、INSERT文実行後に、Entityの\ ``logId``\プロパティに\ ``Statement#getGeneratedKeys()``\で取得したキー値が設定される。


|

.. _DataAccessMyBatis3HowToUseCreateMultiple:

Entityの一括登録
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Entityを一括で登録する際の実装例を以下に示す。

Entityを一括で登録する場合は、

* 複数のレコードを同時に登録するINSERT文を発行する

* JDBCのバッチ更新機能を使用する

方法がある。

JDBCのバッチ更新機能を使用する方法については、「:ref:`DataAccessMyBatis3HowToExtendExecutorTypeBatch`」を参照されたい。

ここでは、複数のレコードを同時に登録するINSERT文を発行するする方法について説明する。
下記例は、データベースとしてH2 Databaseを使用している。


* Repositoryインタフェースにメソッドを定義する。

 .. code-block:: java

    package com.example.domain.repository.todo;
    
    import java.util.List;

    import com.example.domain.model.Todo;
    
    public interface TodoRepository {
    
        // (1)
        void createAll(List<Todo> todos);
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - 上記例では、引数に指定されたTodoオブジェクトのリストを一括登録するためのメソッドとして、
        \ ``createAll``\メソッドを定義している。

|

* マッピングファイルにSQLを定義する。

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <insert id="createAll" parameterType="list">
            INSERT INTO
                t_todo
            (
                todo_id,
                todo_title,
                finished,
                created_at,
                version
            )
            /* (2) */
            VALUES
            /* (3) */
            <foreach collection="list" item="todo" separator=",">
            (
                #{todo.todoId},
                #{todo.todoTitle},
                #{todo.finished},
                #{todo.createdAt},
                #{todo.version}
            )
            </foreach>
        </insert>

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 10 70

    * - 項番
      - 属性
      - 説明
    * - (2)
      - \-
      - VALUE句にレコード登録時の設定値を指定する。
    * - (3)
      - \-
      - \ ``foreach``\要素を使用して、引数で渡されたTodoオブジェクトのリストに対して繰り返し処理を行う。

        \ ``foreach``\の詳細については、
        「`MyBatis3 REFERENCE DOCUMENTATION (Dynamic SQL-foreach-) <http://mybatis.github.io/mybatis-3/dynamic-sql.html#foreach>`_\」を参照されたい。
    * -
      - collection
      - 処理対象のコレクションを指定する。
      
        上記例では、Repositoryのメソッド引数のリストに対して繰り返し処理を行っている。
        Repositoryメソッドの引数に\ ``@Param``\を指定していない場合は、\ ``"list"``\を指定する。
        \ ``@Param``\を指定した場合は、\ ``@Param``\の\ ``value``\属性に指定した値を指定する。
    * -
      - item
      - リストの中の1要素を保持するローカル変数名を指定する。
      
        \ ``foreach``\要素内のSQLからは、#{ローカル変数名.プロパティ名}の形式でJavaBeanのプロパティにアクセスする事ができる。
    * -
      - separator
      - リスト内の要素間を区切るための文字列を指定する。
      
        上記例では、\ ``","``\を指定することで、要素毎のVALUE句を\ ``","``\で区切っている。

\

 .. note:: **複数のレコードを同時に登録するSQLを使用する際の注意点**

    複数のレコードを同時に登録するSQLを実行する場合は、前述の「:ref:`DataAccessMyBatis3HowToUseGenId`」を使用することが出来ない。

|

* 以下のようなSQLが生成され、実行される。

 .. code-block:: sql

    INSERT INTO
        t_todo
    (
        todo_id,
        todo_title,
        finished,
        created_at,
        version
    )
    VALUES 
    (
        '99243507-1b02-45b6-bfb6-d9b89f044e2d',
        'todo title 1',
        false,
        '09/17/2014 23:59:59.999',
        1
    )
    , 
    (
        '66b096f1-791f-412f-9a0a-ee4a3a9186c2',
        'todo title 2',
        0,
        '09/17/2014 23:59:59.999',
        1
    ) 

 .. tip::

    一括登録するためのSQLは、データベースやバージョンによりサポート状況や文法が異なる。
    以下に主要なデータベースのリファレンスページへのリンクを記載しておく。

    * `Oracle 12c <http://docs.oracle.com/database/121/SQLRF/statements_9014.htm>`_
    * `DB2 10.5 <http://www-01.ibm.com/support/knowledgecenter/SSEPGG_10.5.0/com.ibm.db2.luw.sql.ref.doc/doc/r0000970.html>`_
    * `PostgreSQL 9.3 <http://www.postgresql.org/docs/9.3/static/sql-insert.html>`_
    * `MySQL 5.7 <http://dev.mysql.com/doc/refman/5.7/en/insert.html>`_

|

.. _DataAccessMyBatis3HowToUseUpdate:

Entityの更新処理
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Entityの更新方法について、目的別に実装例を説明する。


.. _DataAccessMyBatis3HowToUseUpdateOne:

Entityの1件更新
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Entityを1件更新する際の実装例を以下に示す。

 .. note::

     以降の説明では、バージョンカラムを使用して楽観ロックを行う実装例となっているが、
     楽観ロックの必要がない場合は、楽観ロック関連の処理を行う必要はない。

     排他制御の詳細については、「:doc:`ExclusionControl`」を参照されたい。

|

* Repositoryインタフェースにメソッドを定義する。

 .. code-block:: java

    package com.example.domain.repository.todo;
    
    import com.example.domain.model.Todo;
    
    public interface TodoRepository {
    
        // (1)
        boolean update(Todo todo);
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - 上記例では、引数に指定されたTodoオブジェクトを1件更新するためのメソッドとして、
        \ ``update``\メソッドを定義している。

\

 .. note:: **Entityを1件更新するメソッドの返り値について**
 
    Entityを1件更新するメソッドの返り値は、基本的には\ ``boolean``\でよい。

    ただし、更新結果が複数件になった場合にデータ不整合エラーとして扱う必要がある場合は、
    数値型(\ ``int``\又は\ ``long``\)を返り値にし、更新件数が1件であることをチェックする必要がある。
    主キーが更新条件となっている場合は、更新結果が複数件になる事はないので、\ ``boolean``\でよい。

    * 返り値として\ ``boolean``\を指定した場合は、更新件数が0件の際は\ ``false``\、更新件数が1件以上の際は\ ``true``\が返却される。
    * 返り値として数値型を指定した場合は、更新件数が返却される。


|

* マッピングファイルにSQLを定義する。

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <!-- (2) -->
        <update id="update" parameterType="Todo">
            UPDATE
                t_todo
            SET
                todo_title = #{todoTitle},
                finished = #{finished},
                version = version + 1
            WHERE
                todo_id = #{todoId}
            AND
                version = #{version}
        </update>

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (2)
      - \ ``update``\要素の中に、UPDATEするSQLを実装する。
      
        \ ``id``\属性には、Repositoryインタフェースに定義したメソッドのメソッド名を指定する。

        \ ``update``\要素の詳細については、
        「`MyBatis3 REFERENCE DOCUMENTATION (Mapper XML Files-insert, update and delete-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#insert_update_and_delete>`_\」を参照されたい。

        SET句及びWHERE句にバインドする値は、#{variableName}形式のバインド変数として指定する。
        上記例では、Repositoryインタフェースの引数としてJavaBean(\ ``Todo``\)を指定しているため、
        バインド変数名にはJavaBeanのプロパティ名を指定する。

|

* ServiceクラスにRepositoryをDIし、Repositoryインターフェースのメソッドを呼び出す。

 .. code-block:: java


    package com.example.domain.service.todo;

    import javax.inject.Inject;

    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;

    import com.example.domain.model.Todo;
    import com.example.domain.repository.todo.TodoRepository;

    @Transactional
    @Service
    public class TodoServiceImpl implements TodoService {

        // (3)
        @Inject
        TodoRepository todoRepository;

        @Override
        public Todo update(Todo todo) {

            // (4)
            Todo currentTodo = todoRepository.findOne(todo.getTodoId());
            if (currentTodo == null || currentTodo.getVersion() != todo.getVersion()) {
                throw new ObjectOptimisticLockingFailureException(Todo.class, todo
                        .getTodoId());
            }

            // (5)
            currentTodo.setTodoTitle(todo.getTodoTitle());
            currentTodo.setFinished(todo.isFinished());

            // (6)
            boolean updated = todoRepository.update(currentTodo);
            // (7)
            if (!updated) {
                throw new ObjectOptimisticLockingFailureException(Todo.class,
                        currentTodo.getTodoId());
            }
            currentTodo.setVersion(todo.getVersion() + 1);

            return currentTodo;
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (3)
      - ServiceクラスにRepositoryインターフェースをDIする。
    * - (4)
      - 更新対象のEntityをデータベースより取得する。
      
        上記例では、Entityが更新されている場合(レコードが削除されている場合又はバージョンが更新されている場合)は、
        Spring Frameworkから提供されている楽観ロック例外(\ ``org.springframework.orm.ObjectOptimisticLockingFailureException``\)を発生させている。
    * - (5)
      - 更新対象のEntityに対して、更新内容を反映する。

        上記例では、「タイトル」「完了フラグ」を反映している。更新項目が少ない場合は上記実装例のままでもよいが、
        更新項目が多い場合は、「:doc:`Utilities/Dozer`」を使用することを推奨する。
    * - (6)
      - Repositoryインターフェースのメソッドを呼び出し、Entityを1件更新する。
    * - (7)
      - Entityの更新結果を判定する。
      
        上記例では、Entityが更新されなかった場合(レコードが削除されている場合又はバージョンが更新されている場合)は、
        Spring Frameworkから提供されている楽観ロック例外(\ ``org.springframework.orm.ObjectOptimisticLockingFailureException``\)を発生させている。


 .. tip::

    上記例では、更新処理が成功した後に、

     .. code-block:: java

        currentTodo.setVersion(todo.getVersion() + 1);

    としている。

    これはデータベースに更新したバージョンと、Entityが保持するバージョンを合わせるための処理である。
    
    呼び出し元(ControllerやJSPなど)の処理でバージョンを参照する場合は、
    データベースの状態とEntityの状態を一致させておかないと、
    データ不整合が発生し、アプリケーションが期待通りの動作しない事になる。

|

.. _DataAccessMyBatis3HowToUseUpdateMultiple:

Entityの一括更新
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Entityを一括で更新する際の実装例を以下に示す。

Entityを一括で更新する場合は、

* 複数のレコードを同時に更新するUPDATE文を発行する

* JDBCのバッチ更新機能を使用する

方法がある。

JDBCのバッチ更新機能を使用する方法については、「:ref:`DataAccessMyBatis3HowToExtendExecutorTypeBatch`」を参照されたい。

|

ここでは、複数のレコードを同時に更新するUPDATE文を発行する方法について説明する。

* Repositoryインタフェースにメソッドを定義する。

 .. code-block:: java

    package com.example.domain.repository.todo;

    import com.example.domain.model.Todo;
    import org.apache.ibatis.annotations.Param;

    import java.util.List;

    public interface TodoRepository {

        // (1)
        int updateFinishedByTodIds(@Param("finished") boolean finished,
                                   @Param("todoIds") List<String> todoIds);

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - 上記例では、引数に指定されたIDのリストに該当するレコードの\ ``finished``\カラムを更新するためのメソッドとして、
        \ ``updateFinishedByTodIds``\メソッドを定義している。

 .. note:: **Entityを一括更新するメソッドの返り値について**

    Entityを一括更新するメソッドの返り値は、数値型(\ ``int``\又は\ ``long``\)でよい。
    数値型にすると、更新されたレコード数を取得する事ができる。

|

* マッピングファイルにSQLを定義する。

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <update id="updateFinishedByTodIds">
            UPDATE
                t_todo
            SET
                finished = #{finished},
                /* (2) */
                version = version + 1
            WHERE
                /* (3) */
                <foreach item="todoId" collection="todoIds"
                         open="todo_id IN (" separator="," close=")">
                    #{todoId}
                </foreach>
        </update>

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 10 70

    * - 項番
      - 属性
      - 説明
    * - (2)
      - \-
      - バージョンカラムを使用して楽観ロックを行う場合は、バージョンカラムを更新する。

        更新しないと、楽観ロック制御が正しく動作しなくなる。
        排他制御の詳細については、「:doc:`ExclusionControl`」を参照されたい。
    * - (3)
      - \-
      - WHERE句に複数レコードを更新するための更新条件を指定する。
    * -
      - \-
      - \ ``foreach``\要素を使用して、引数で渡されたIDのリストに対して繰り返し処理を行う。

        上記例では、引数で渡されたIDのリストより、IN句を生成している。

        \ ``foreach``\の詳細については、
        「`MyBatis3 REFERENCE DOCUMENTATION (Dynamic SQL-foreach-) <http://mybatis.github.io/mybatis-3/dynamic-sql.html#foreach>`_\」を参照されたい。
    * -
      - collection
      - 処理対象のコレクションを指定する。

        上記例では、Repositoryのメソッド引数のIDのリスト(\ ``todoIds``\)に対して繰り返し処理を行っている。
    * -
      - item
      - リストの中の1要素を保持するローカル変数名を指定する。
    * -
      - separator
      - リスト内の要素間を区切るための文字列を指定する。

        上記例では、IN句の区切り文字である\ ``","``\を指定している。

|


.. _DataAccessMyBatis3HowToUseDelete:

Entityの削除処理
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _DataAccessMyBatis3HowToUseDeleteOne:


Entityの1件削除
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Entityを1件削除する際の実装例を以下に示す。

 .. note::

     以降の説明では、バージョンカラムを使用した楽観ロックを行う実装例となっているが、
     楽観ロックの必要がない場合は、楽観ロック関連の処理を行う必要はない。

     排他制御の詳細については、「:doc:`ExclusionControl`」を参照されたい。

|

* Repositoryインタフェースにメソッドを定義する。

 .. code-block:: java

    package com.example.domain.repository.todo;

    import com.example.domain.model.Todo;

    public interface TodoRepository {

        // (1)
        boolean delete(Todo todo);

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - 上記例では、引数に指定されたTodoオブジェクトを1件削除するためのメソッドとして、
        \ ``delete``\メソッドを定義している。


 .. note:: **Entityを1件削除するメソッドの返り値について**

    Entityを1件削除するメソッドの返り値は、基本的には\ ``boolean``\でよい。

    ただし、削除結果が複数件になった場合にデータ不整合エラーとして扱う必要がある場合は、
    数値型(\ ``int``\又は\ ``long``\)を返り値にし、削除件数が1件であることをチェックする必要がある。
    主キーが削除条件となっている場合は、削除結果が複数件になる事はないので、\ ``boolean``\でよい。

    * 返り値として\ ``boolean``\を指定した場合は、削除件数が0件の際は\ ``false``\、削除件数が1件以上の際は\ ``true``\が返却される。
    * 返り値として数値型を指定した場合は、削除件数が返却される。


|

* マッピングファイルにSQLを定義する。


 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <!-- (2) -->
        <delete id="delete" parameterType="Todo">
            DELETE FROM
                t_todo
            WHERE
                todo_id = #{todoId}
            AND
                version = #{version}
        </delete>

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (2)
      - \ ``delete``\要素の中に、DELETEするSQLを実装する。

        \ ``id``\属性には、Repositoryインタフェースに定義したメソッドのメソッド名を指定する。

        \ ``delete``\要素の詳細については、
        「`MyBatis3 REFERENCE DOCUMENTATION (Mapper XML Files-insert, update and delete-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#insert_update_and_delete>`_\」を参照されたい。

        WHERE句にバインドする値は、#{variableName}形式のバインド変数として指定する。
        上記例では、Repositoryインタフェースの引数としてJavaBean(\ ``Todo``\)を指定しているため、
        バインド変数名にはJavaBeanのプロパティ名を指定する。

|

* ServiceクラスにRepositoryをDIし、Repositoryインターフェースのメソッドを呼び出す。


 .. code-block:: java

    package com.example.domain.service.todo;

    import javax.inject.Inject;

    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;

    import com.example.domain.model.Todo;
    import com.example.domain.repository.todo.TodoRepository;

    @Transactional
    @Service
    public class TodoServiceImpl implements TodoService {

        // (3)
        @Inject
        TodoRepository todoRepository;

        @Override
        public Todo delete(String todoId, long version) {

            // (4)
            Todo currentTodo = todoRepository.findOne(todoId);
            if (currentTodo == null || currentTodo.getVersion() != version) {
                throw new ObjectOptimisticLockingFailureException(Todo.class, todoId);
            }

            // (5)
            boolean deleted = todoRepository.delete(currentTodo);
            // (6)
            if (!deleted) {
                throw new ObjectOptimisticLockingFailureException(Todo.class,
                        currentTodo.getTodoId());
            }

            return currentTodo;
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (3)
      - ServiceクラスにRepositoryインターフェースをDIする。
    * - (4)
      - 削除対象のEntityをデータベースより取得する。

        上記例では、Entityが更新されている場合(レコードが削除されている場合又はバージョンが更新されている場合)は、
        Spring Frameworkから提供されている楽観ロック例外(\ ``org.springframework.orm.ObjectOptimisticLockingFailureException``\)を発生させている。
    * - (5)
      - Repositoryインターフェースのメソッドを呼び出し、Entityを1件削除する。
    * - (6)
      - Entityの削除結果を判定する。

        上記例では、Entityが削除されなかった場合(レコードが削除されている場合又はバージョンが更新されている場合)は、
        Spring Frameworkから提供されている楽観ロック例外(\ ``org.springframework.orm.ObjectOptimisticLockingFailureException``\)を発生させている。

|

.. _DataAccessMyBatis3HowToUseDeleteMultiple:


Entityの一括削除
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""


Entityを一括で削除する際の実装例を以下に示す。

Entityを一括で削除する場合は、

* 複数のレコードを同時に削除するDELETE文を発行する

* JDBCのバッチ更新機能を使用する

方法がある。

JDBCのバッチ更新機能を使用する方法については、「:ref:`DataAccessMyBatis3HowToExtendExecutorTypeBatch`」を参照されたい。

|

ここでは、複数のレコードを同時に削除するDELETE文を発行する方法について説明する。

* Repositoryインタフェースにメソッドを定義する。

 .. code-block:: java

    package com.example.domain.repository.todo;

    public interface TodoRepository {

        // (1)
        int deleteOlderFinishedTodo(Date criteriaDate);

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - 上記例では、基準日より前に作成され完了済みのレコードを削除するためのメソッドとして、
        \ ``deleteOlderFinishedTodo``\メソッドを定義している。

 .. note:: **Entityを一括削除するメソッドの返り値について**

    Entityを一括削除するメソッドの返り値は、数値型(\ ``int``\又は\ ``long``\)でよい。
    数値型にすると、削除されたレコード数を取得する事ができる。

|

* マッピングファイルにSQLを定義する。

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <delete id="deleteOlderFinishedTodo" parameterType="date">
            <![CDATA[
            DELETE FROM
                t_todo
            /* (2) */
            WHERE
                finished = TRUE
            AND
                created_at  < #{criteriaDate}
            ]]>
        </delete>

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (3)
      - WHERE句に複数レコードを更新するための削除条件を指定する。

        上記例では、

        * 完了済み(\ ``finished``\が\ ``TRUE``\)
        * 基準日より前に作成された(\ ``created_at``\が基準日より前)

        を削除条件として指定している。

|

.. _DataAccessMyBatis3HowToUseDynamicSql:

動的SQLの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

動的SQLを組み立てる実装例を以下に示す。

MyBatis3では、動的にSQLを組み立てるためのXML要素と、OGNLベースの式（Expression言語）を使用することで、
動的SQLを組み立てる仕組みを提供している。

動的SQLの詳細については、
「`MyBatis3 REFERENCE DOCUMENTATION (Dynamic SQL) <http://mybatis.github.io/mybatis-3/dynamic-sql.html>`_\」を参照されたい。

MyBatis3では、動的にSQLを組み立てるために、以下のXML要素を提供している。


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 10 70

    * - 項番
      - 要素名
      - 説明
    * - 1.
      - \ ``if``
      - 条件に一致した場合のみ、SQLの組み立てを行うための要素。
    * - 2.
      - \ ``choose``\
      - 複数の選択肢の中から条件に一致する1つを選んで、SQLの組み立てを行うための要素。
    * - 3.
      - \ ``where``\
      - 組み立てたWHERE句に対して、接頭語及び末尾の付与や除去など行うための要素。
    * - 4.
      - \ ``set``\
      - 組み立てたSET句用に対して、接頭語及び末尾の付与や除去など行うための要素。
    * - 5.
      - \ ``foreach``\
      - コレクションや配列に対して繰り返し処理を行うための要素
    * - 6.
      - \ ``bind``\
      - OGNL式の結果を変数に格納するための要素。

        \ ``bind``\要素を使用して格納した変数は、SQL内で参照する事ができる。

 .. tip::

    一覧には記載していないが、動的SQLを組み立てるためのXML要素として\ ``trim``\要素が提供されている。

    \ ``trim``\要素は、\ ``where``\要素\と\ ``set``\要素をより汎用的にしたXML要素である。

    ほとんどの場合は、\ ``where``\要素と\ ``set``\要素で要件を充たせるため、本ガイドラインでは\ ``trim``\要素の説明は割愛する。
    \ ``trim``\要素が必要になる場合は、
    「`MyBatis3 REFERENCE DOCUMENTATION (Dynamic SQL-trim, where, set-) <http://mybatis.github.io/mybatis-3/dynamic-sql.html#trim_where_set>`_\」
    を参照されたい。

|

if要素の実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\ ``if``\要素は、指定した条件に一致した場合のみ、SQLの組み立てを行うためのXML要素である。

 .. code-block:: xml

    <select id="findAllByCriteria" parameterType="TodoCriteria" resultType="Todo">
        SELECT
            todo_id,
            todo_title,
            finished,
            created_at,
            version
        FROM
            t_todo
        WHERE
            todo_title LIKE #{todoTitle} || '%' ESCAPE '~'
        <!-- (1) -->
        <if test="finished != null">
            AND
                finished = #{finished}
        </if>
        ORDER BY
            todo_id
    </select>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - \ ``if``\要素の\ ``test``\属性に、条件を指定する。

        上記例では、検索条件として\ ``finished``\が指定されている場合に、\ ``finished``\カラムに対する条件をSQLに加えている。

上記の動的SQLで生成されるSQL(WHERE句)は、以下2パターンとなる。

 .. code-block:: sql

    -- (1) finished == null
    ...
    WHERE
        todo_title LIKE ? || '%' ESCAPE '~'
    ORDER BY
        todo_id

 .. code-block:: sql

    -- (2) finished != null
    ...
    WHERE
        todo_title LIKE ? || '%' ESCAPE '~'
    AND
        finished = ?
    ORDER BY
        todo_id

|

choose要素の実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``choose``\要素は、複数の選択肢の中から条件に一致する1つを選んで、SQLの組み立てを行うためのXML要素である。

 .. code-block:: xml

    <select id="findAllByCriteria" parameterType="TodoCriteria" resultType="Todo">
        SELECT
            todo_id,
            todo_title,
            finished,
            created_at,
            version
        FROM
            t_todo
        WHERE
            todo_title LIKE #{todoTitle} || '%' ESCAPE '~'
        <!-- (1) -->
        <choose>
            <!-- (2) -->
            <when test="createdAt != null">
                AND
                    created_at <![CDATA[ > ]]> #{createdAt}
            </when>
            <!-- (3) -->
            <otherwise>
                AND
                    created_at <![CDATA[ > ]]> CURRENT_DATE
            </otherwise>
        </choose>
        ORDER BY
            todo_id
    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - \ ``choose``\要素に中に、\ ``when``\要素と\ ``otherwise``\要素を指定して、SQLを組み立てる条件を指定する。
    * - (2)
      - \ ``when``\要素の\ ``test``\属性に、条件を指定する。

        上記例では、検索条件として\ ``createdAt``\が指定されている場合に、
        \ ``create_at``\カラムの値が指定日以降のレコードを抽出するための条件をSQLに加えている。
    * - (3)
      - \ ``otherwise``\要素に、全ての\ ``when``\要素に一致しない場合時に組み立てるSQLを指定する。

        上記例では、\ ``create_at``\カラムの値が現在日以降のレコード(当日作成されたレコード)を抽出するための条件をSQLに加えている。


上記の動的SQLで生成されるSQL(WHERE句)は、以下2パターンとなる。

 .. code-block:: sql

    -- (1) createdAt!=null
    ...
    WHERE
        todo_title LIKE ? || '%' ESCAPE '~'
    AND
        created_at   >   ?
    ORDER BY
        todo_id

 .. code-block:: sql

    -- (2) createdAt==null
    ...
    WHERE
        todo_title LIKE ? || '%' ESCAPE '~'
    AND
        created_at > CURRENT_DATE
    ORDER BY
        todo_id

|

where要素の実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``where``\要素は、WHERE句を動的に生成するためのXML要素である。

\ ``where``\要素を使用すると、

* WHERE句の付与
* AND句、OR句の除去

などが行われるため、シンプルにWHERE句を組み立てる事ができる。

 .. code-block:: xml

    <select id="findAllByCriteria2" parameterType="TodoCriteria" resultType="Todo">
        SELECT
            todo_id,
            todo_title,
            finished,
            created_at,
            version
        FROM
            t_todo
        <!-- (1) -->
        <where>
            <!-- (2) -->
            <if test="finished != null">
                AND
                    finished = #{finished}
            </if>
            <!-- (3) -->
            <if test="createdAt != null">
                AND
                    created_at <![CDATA[ > ]]> #{createdAt}
            </if>
        </where>
        ORDER BY
            todo_id
    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - \ ``where``\要素に中で、WHERE句を組み立てるための動的SQLを実装する。

        \ ``where``\要素内で組み立てたSQLに応じて、WHERE句の付与や、AND句及びORの除去などが行われる。
    * - (2)
      - 動的SQLを組み立てる。

        上記例では、検索条件として\ ``finished``\が指定されている場合に、
        \ ``finished``\カラムに対する条件をSQLに加えている。
    * - (3)
      - 動的SQLを組み立てる。

        上記例では、検索条件として\ ``createdAt``\が指定されている場合に、
        \ ``created_at``\カラムに対する条件をSQLに加えている。


上記の動的SQLで生成されるSQL(WHERE句)は、以下4パターンとなる。

 .. code-block:: sql

    -- (1) finished != null && createdAt != null
    ...
    FROM
        t_todo
    WHERE
        finished = ?
    AND
        created_at  >  ?
    ORDER BY
        todo_id

 .. code-block:: sql

    -- (2) finished != null && createdAt == null
    ...
    FROM
        t_todo
    WHERE
        finished = ?
    ORDER BY
        todo_id

 .. code-block:: sql

    -- (3) finished == null && createdAt != null
    ...
    FROM
        t_todo
    WHERE
        created_at  >  ?
    ORDER BY
        todo_id

 .. code-block:: sql

    -- (4) finished == null && createdAt == null
    ...
    FROM
        t_todo
    ORDER BY
        todo_id

|

set要素の実装例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``set``\要素は、SET句を動的に生成するためのXML要素である。

\ ``set``\要素を使用すると、

* SET句の付与
* 末尾のカンマの除去

などが行われるため、シンプルにSET句を組み立てる事ができる。

 .. code-block:: xml

    <update id="update" parameterType="Todo">
        UPDATE
            t_todo
        <!-- (1)  -->
        <set>
            version = version + 1,
            <!-- (2) -->
            <if test="todoTitle != null">
                todo_title = #{todoTitle}
            </if>
        </set>
        WHERE
            todo_id = #{todoId}
    </update>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - \ ``set``\要素に中で、SET句を組み立てるための動的SQLを実装する。

        \ ``set``\要素内で組み立てたSQLに応じて、SET句の付与や、末尾のカンマの除去などが行われる。
    * - (2)
      - 動的SQLを組み立てる。

        上記例では、更新項目として\ ``todoTitle``\が指定されている場合に、
        \ ``todo_title``\カラムを更新カラムとしてSQLに加えている。

上記の動的SQLで生成されるSQLは、以下2パターンとなる。

 .. code-block:: sql

    -- (1) todoTitle != null
    UPDATE
        t_todo
    SET
        version = version + 1,
        todo_title = ?
    WHERE
        todo_id = ?

 .. code-block:: sql

    -- (2) todoTitle == null
    UPDATE
        t_todo
    SET
       version = version + 1
    WHERE
        todo_id = ?

|

foreach要素の実装例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``foreach``\要素は、コレクションや配列に対して繰り返し処理を行うためのXML要素である。

 .. code-block:: xml

    <select id="findAllByCreatedAtList" parameterType="list" resultType="Todo">
        SELECT
            todo_id,
            todo_title,
            finished,
            created_at,
            version
        FROM
            t_todo
        <where>
            <!-- (1) -->
            <if test="list != null">
                <!-- (2) -->
                <foreach collection="list" item="date" separator="OR">
                <![CDATA[
                    (created_at >= #{date} AND created_at < DATEADD('DAY', 1, #{date}))
                ]]>
                </foreach>
            </if>
        </where>
        ORDER BY
            todo_id
    </select>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.65\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 15 65

    * - 項番
      - 属性
      - 説明
    * - (1)
      - \-
      - 繰返し処理を行う対象のコレクション又は配列に対して、\ ``null``\チェックを行う。

        \ ``null``\にならない事がない場合は、このチェックは実装しなくてもよい。
    * - (2)
      - \-
      - \ ``foreach``\要素を使用して、コレクションや配列に対して繰返し処理を行い、動的SQLを組み立てる。

        上記例では、レコードの作成日付が、指定された日付(日付リスト)の何れかと一致するレコードを検索するためのWHERE句を組み立てている。
    * -
      - collection
      - \ ``collection``\属性に、繰返し処理を行うコレクションや配列を指定する。

        上記例では、Repositoryメソッドの引数に指定されたコレクションを指定している。

    * -
      - item
      - \ ``item``\属性に、リストの中の1要素を保持するローカル変数名を指定する。

        上記例では、\ ``collection``\属性に日付リストを指定しているので、
        \ ``date``\という変数名を指定している。
    * -
      - separator
      - \ ``separator``\属性に、要素間の区切り文字列を指定する。

        上記例では、OR条件のWHERE句を組み立てている。

 .. tip::

    上記例では使用していないが、 \ ``foreach``\要素には、以下の属性が存在する。

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.65\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 15 65

        * - 項番
          - 属性
          - 説明
        * - (1)
          - open
          - コレクションの先頭要素を処理する前に設定する文字列を指定する。
        * - (2)
          - close
          - コレクションの末尾要素を処理した後に設定する文字列を指定する。
        * - (3)
          - index
          - ループ番号を格納する変数名を指定する。

    \ ``index``\属性を使用するケースはあまりないが、\ ``open``\属性と \ ``close``\属性は、
    IN句などを動的に生成する際に使用される。

    以下に、IN句を作成する際の\ ``foreach``\要素の使用例を記載しておく。

     .. code-block:: xml

        <foreach collection="list" item="statusCode"
                open="AND order_status IN ("
                separator=","
                close=")">
            #{statusCode}
        </foreach>

    以下の様なSQLが組み立てられる。

     .. code-block:: sql

        -- list=['accepted','checking']
        ...
        AND order_status IN (?,?)


| 上記の動的SQLで生成されるSQL(WHERE句)は、以下3パターンとなる。

 .. code-block:: sql

    -- (1) list=null or statusCodes=[]
    ...
    FROM
        t_todo
    ORDER BY
        todo_id

 .. code-block:: sql

    -- (2) list=['2014-01-01']
    ...
    FROM
        t_todo
    WHERE
        (created_at >= ? AND created_at < DATEADD('DAY', 1, ?))
    ORDER BY
        todo_id

 .. code-block:: sql

    -- (3) list=['2014-01-01','2014-01-02']
    ...
    FROM
        t_todo
    WHERE
        (created_at >= ? AND created_at < DATEADD('DAY', 1, ?))
    OR
        (created_at >= ? AND created_at < DATEADD('DAY', 1, ?))
    ORDER BY
        todo_id

|

bind要素の実装例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``bind``\要素は、OGNL式の結果を変数に格納するためのXML要素である。

 .. code-block:: xml

    <select id="findAllByCriteria" parameterType="TodoCriteria" resultType="Todo">
        <!-- (1) -->
        <bind name="escapedTodoTitle"
              value="@org.terasoluna.gfw.common.query.QueryEscapeUtils@toLikeCondition(todoTitle)" />
        SELECT
            todo_id,
            todo_title,
            finished,
            created_at,
            version
        FROM
            t_todo
        WHERE
            /* (2) */
            todo_title LIKE #{escapedTodoTitle} || '%' ESCAPE '~'
        ORDER BY
            todo_id
    </select>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.65\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 15 65

    * - 項番
      - 属性
      - 説明
    * - (1)
      - \-
      - \ ``bind``\要素を使用して、OGNL式の結果を変数に格納する

        上記例では、OGNL式を使ってメソッドを呼び出した結果を、変数に格納している。
    * -
      - name
      - \ ``name``\属性には、変数名を指定する。

        ここで指定した変数名は、SQLのバインド変数として使用する事ができる。
    * -
      - value
      - \ ``value``\属性には、OGNL式を指定する。

        OGNL式を実行した結果が、\ ``name``\属性で指定した変数に格納される。

        上記例では、共通ライブラリから提供しているメソッド(\ ``QueryEscapeUtils#toLikeCondition(String)``\)を呼び出した結果を、
        \ ``escapedTodoTitle``\という変数に格納している。
    * - (2)
      - \-
      - \ ``bind``\要素を使用して作成した変数を、バインド変数として指定する。

        上記例では、\ ``bind``\要素を使用して作成した変数(\ ``escapedTodoTitle``\)を、バインド変数として指定している。

 .. tip::

    上記例では、\ ``bind``\要素を使用して作成した変数をバインド変数として指定しているが、
    置換変数として使用する事もできる。

    バインド変数と置換変数については、
    「:ref:`DataAccessMyBatis3HowToUseSqlInjectionCountermeasure`」を参照されたい。

|

.. _DataAccessMyBatis3HowToUseLikeEscape:

LIKE検索時のエスケープ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

LIKE検索を行う場合は、検索条件として使用する値をLIKE検索用にエスケープする必要がある。

LIKE検索用のエスケープ処理は、
共通ライブラリから提供している\ ``org.terasoluna.gfw.common.query.QueryEscapeUtils`` \クラスのメソッドを使用することで実現する事ができる。

共通ライブラリから提供しているエスケープ処理の仕様については、
「:ref:`data-access-common_appendix_like_escape`」を参照されたい。

 .. code-block:: xml

    <select id="findAllByCriteria" parameterType="TodoCriteria" resultType="Todo">
        <!-- (1) -->
        <bind name="todoTitleContainingCondition"
              value="@org.terasoluna.gfw.common.query.QueryEscapeUtils@toContainingCondition(todoTitle)" />
        SELECT
            todo_id,
            todo_title,
            finished,
            created_at,
            version
        FROM
            t_todo
        WHERE
            /* (2) (3) */
            todo_title LIKE #{todoTitleContainingCondition} ESCAPE '~'
        ORDER BY
            todo_id
    </select>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - \ ``bind``\要素(OGNL式)を使用して、共通ライブラリから提供しているLIKE検索用のエスケープ処理メソッドを呼び出す。

        上記例では、部分一致用のエスケープ処理を行い\ ``todoTitleContainingCondition``\という変数に格納している。
        \ ``QueryEscapeUtils@toContainingCondition(String)``\メソッドは、エスケープした文字列の前後に\ "``%``"\を付与するメソッドである。
    * - (2)
      - 部分一致用のエスケープを行った文字列を、LIKE句のバインド変数として指定する。
    * - (3)
      - ESCAPE句にエスケープ文字を指定する。

        共通ライブラリから提供しているエスケープ処理では、
        エスケープ文字として\ ``"~"``\を使用しているため、ESCAPE句に\ ``'~'``\を指定している。

 .. tip::

     上記例では、部分一致用のエスケープ処理を行うメソッドを呼び出しているが、

     * 前方一致用のエスケープ(\ ``QueryEscapeUtils@toStartingWithCondition(String)``\)
     * 後方一致用のエスケープ(\ ``QueryEscapeUtils@toEndingWithCondition(String)``\)
     * エスケープのみ(\ ``QueryEscapeUtils@toLikeCondition(String)``\)

     を行うメソッドも用意されている。

     詳細は「:ref:`data-access-common_appendix_like_escape`」を参照されたい。

 .. note::

     上記例では、マッピングファイル内でエスケープ処理を行うメソッドを呼び出しているが、
     Repositoryのメソッドを呼び出す前に、Serviceの処理としてエスケープ処理を行う方法もある。

     コンポーネントの役割としては、マッピングファイルでエスケープ処理を行う方が適切なため、
     本ガイドラインとしては、マッピングファイル内でエスケープ処理を行う事を推奨する。

|

.. _DataAccessMyBatis3HowToUseSqlInjectionCountermeasure:

SQL Injection対策
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

SQLを組み立てる際は、SQL Injectionが発生しないように注意する必要がある。

MyBatis3では、SQLに値を埋め込む仕組みとして、以下の2つの方法を提供している。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
     :header-rows: 1
     :widths: 10 20 60

     * - 項番
       - 方法
       - 説明
     * - (1)
       -  バインド変数を使用して埋め込む
       - この方法を使用すると、 SQL組み立て後に\ ``java.sql.PreparedStatement`` \を使用して値が埋め込められるため、
         **安全に値を埋め込むことができる。**

         **ユーザからの入力値をSQLに埋め込む場合は、原則バインド変数を使用すること。**
     * - (2)
       -  置換変数を使用して埋め込む
       - この方法を使用すると、SQLを組み立てるタイミングで文字列として置換されてしまうため、
         **安全な値の埋め込みは保証されない。**

 .. warning::

    ユーザからの入力値を置換変数を使って埋め込むと、
    SQL Injectionが発生する危険性が高くなることを意識すること。

    ユーザからの入力値を置換変数を使って埋め込む必要がある場合は、
    SQL Injectionが発生しないことを保障するために、かならず入力チェックを行うこと。

    基本的には、 **ユーザからの入力値はそのまま使わないことを強く推奨する。**

|

バインド変数を使って埋め込む方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

バインド変数の使用例を以下に示す。

 .. code-block:: xml

    <insert id="create" parameterType="Todo">
        INSERT INTO
            t_todo
        (
            todo_id,
            todo_title,
            finished,
            created_at,
            version
        )
        VALUES
        (
            /* (1) */
            #{todoId},
            #{todoTitle},
            #{finished},
            #{createdAt},
            #{version}
        )
    </insert>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :widths: 10 80
    :header-rows: 1

    * - 項番
      - 説明
    * - (1)
      - バインドする値が格納されているプロパティのプロパティ名を、
        \ ``#{`` \と\ ``}`` \で囲み、バインド変数として指定する。

 .. tip::

     バインド変数には、いくつかの属性を指定する事が出来る。

     指定できる属性としては、

     * javaType
     * jdbcType
     * typeHandler
     * numericScale
     * mode
     * resultMap
     * jdbcTypeName

     がある。

     基本的には、単純にプロパティ名を指定するだけで、MyBatisが適切な振る舞いを選択してくれる。
     上記属性は、MyBatisが適切な振る舞いを選択してくれない時に指定すればよい。

     属性の使い方については、
     「\ `MyBatis3 REFERENCE DOCUMENTATION(Mapper XML Files-Parameters-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#Parameters>`_ 」を参照されたい。



|

置換変数を使って埋め込む方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

置換変数の使用例を以下に示す。


* Repositoryインタフェースにメソッドを定義する。

 .. code-block:: java

    public interface TodoRepository {
        List<Todo> findAllByCriteria(@Param("criteria") TodoCriteria criteria,
                                     @Param("direction") String direction);
    }

* マッピングファイルにSQLを実装する。

 .. code-block:: xml

    <select id="findAllByCriteria" parameterType="TodoCriteria" resultType="Todo">
        <bind name="todoTitleContainingCondition"
              value="@org.terasoluna.gfw.common.query.QueryEscapeUtils@toContainingCondition(criteria.todoTitle)" />
        SELECT
            todo_id,
            todo_title,
            finished,
            created_at,
            version
        FROM
            t_todo
        WHERE
            todo_title LIKE #{todoTitleContainingCondition} ESCAPE '~'
        ORDER BY
            /* (1) */
            todo_id ${direction}
    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :widths: 10 80
    :header-rows: 1

    * - 項番
      - 説明
    * - (1)
      - 置換する値が格納されているプロパティのプロパティ名を\ ``${`` \と
        \ ``}`` \で囲み、置換変数として指定する。上記例では、\ ``${direction}`` \
        の部分は、\ ``"DESC"`` \または\ ``"ASC"`` \で置換される。

 .. warning::

    **置換変数による埋め込みは、必ずアプリケーションとして安全な値であることを担保した上で、**
    **テーブル名、カラム名、ソート条件などに限定して使用することを推奨する。**

    例えば以下のように、コード値とSQLに埋め込むための値のペアを\ ``Map``\に格納しておき、

      .. code-block:: java

        Map<String, String> directionMap = new HashMap<String, String>();
        directionMap.put("1", "ASC");
        directionMap.put("2", "DESC");

    入力値はコード値として扱い、SQLを実行する処理の中で安全な値に変換することが望ましい。

      .. code-block:: java

        String direction = directionMap.get(directionCode);
        todoRepository.findAllByCriteria(criteria, direction);

    上記例では\ ``Map``\を使用しているが、
    共通ライブラリから提供している「\ :doc:`Codelist` \」を使用しても良い。
    「\ :doc:`Codelist` \」を使用すると、入力チェックと連動する事ができるため、
    より安全に値の埋め込みを行う事ができる。

    - :file:`projectName-domain/src/main/resources/META-INF/spring/projectName-codelist.xml`

      .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd">

            <bean id="CL_DIRECTION" class="org.terasoluna.gfw.common.codelist.SimpleMapCodeList">
                <property name="map">
                    <map>
                        <entry key="1" value="ASC" />
                        <entry key="2" value="DESC" />
                    </map>
                </property>
            </bean>
        </beans>

    - Serviceクラス

      .. code-block:: java

        @Inject
        @Named("CL_DIRECTION")
        CodeList directionCodeList;

        // ...

        public List<Todo> searchTodos(TodoCriteria criteria, String directionCode){
            String direction = directionCodeList.asMap().get(directionCode);
            List<Todo> todos = todoRepository.findAllByCriteria(criteria, direction);
            return todos;
        }

|

.. _DataAccessMyBatis3HowToExtend:

How to extend
--------------------------------------------------------------------------------


.. _DataAccessMyBatis3HowToExtendSqlShare:

SQL文の共有
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

SQL文を複数のSQLで共有する方法について、説明を行う。

MyBatis3では、 \ ``sql``\要素と\ ``include``\要素を使用することで、
SQL文(又はSQL文の一部)を共有する事ができる。

 .. note:: **SQL文の共有化の使用例**

    ページネーション検索を実現する場合は、「検索条件に一致するEntityの総件数を取得するSQL」と
    「 検索条件に一致するEntityのリストを取得するSQL」のWHERE句は共有した方がよい。

|

マッピングファイルの実装例は以下の通り。

 .. code-block:: xml
    :emphasize-lines: 1-2, 16-17, 29-30

    <!-- (1)  -->
    <sql id="findPageByCriteriaWherePhrase">
        <![CDATA[
        WHERE
            todo_title LIKE #{title} || '%' ESCAPE '~'
        AND
            created_at < #{createdAt}
        ]]>
    </sql>

    <select id="countByCriteria" resultType="_long">
        SELECT
            COUNT(*)
        FROM
            t_todo
        <!-- (2)  -->
        <include refid="findPageByCriteriaWherePhrase"/>
    </select>

    <select id="findPageByCriteria" resultType="Todo">
        SELECT
            todo_id,
            todo_title,
            finished,
            created_at,
            version
        FROM
            t_todo
        <!-- (2)  -->
        <include refid="findPageByCriteriaWherePhrase"/>
        ORDER BY
            todo_id
    </select>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :widths: 10 80
    :header-rows: 1

    * - 項番
      - 説明
    * - (1)
      - \ ``sql``\要素の中に、複数のSQLで共有するSQL文を実装する。

        \ ``id``\属性には、マッピングファイル内でユニークとなるIDを指定する。
    * - (2)
      - \ ``include``\要素を使用して、インクルードするSQLを指定する。

        \ ``refid``\属性には、インクルードするSQLのID(\ ``sql``\要素の\ ``id``\属性に指定した値)を指定する。


|

.. _DataAccessMyBatis3HowToExtendTypeHandler:


TypeHandlerの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

MyBatis3の標準でサポートされていないJavaクラスとのマッピングが必要だったり、
MyBatis3標準の振る舞いを変更する必要がある場合は、独自のTypeHandlerの作成が必要となる。

以下に、

* :ref:`DataAccessMyBatis3HowToExtendTypeHandlerBlob`
* :ref:`DataAccessMyBatis3HowToExtendTypeHandlerClob`
* :ref:`DataAccessMyBatis3HowToExtendTypeHandlerJoda`

を例に、TypeHandlerの実装方法について説明する。

作成したTypeHandlerをアプリケーションに適用する方法については、
「:ref:`DataAccessMyBatis3HowToUseSettingsTypeHandler`」を参照されたい。

 .. note:: **BLOB用とCLOB用の実装例の前提条件について**

    BLOBとCLOBの実装例では、JDBC 4.0から追加されたメソッドを使用している。

    JDBC 4.0との互換性のないJDBCドライバや3rdパーティのラッパクラスなどを使用する場合は、
    以下に説明する実装例では動作しない可能性がある点を補足しておく。
    JDBC 4.0との互換性がない環境で動作させる場合は、
    利用するJDBCドライバの互換バージョンを意識した実装に変更する必要がある。

    例えば、PostgreSQL9.3用のJDBCドライバ(\ ``postgresql-9.3-1102-jdbc41.jar``\)では、
    JDBC 4.0から追加された多くのメソッドが、未実装の状態である。

|

.. _DataAccessMyBatis3HowToExtendTypeHandlerBlob:

BLOB用のTypeHandlerの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

MyBatis3では、BLOBを\ ``byte[]``\にマッピングするためのTypeHandlerを提供している。
ただし、扱うデータの容量が大きい場合は、\ ``java.io.InputStream``\とマッピングが必要なケースがある。

以下に、BLOBと\ ``java.io.InputStream``\をマッピングするためのTypeHandlerの実装例を示す。

 .. code-block:: java

    package com.example.infra.mybatis.typehandler;

    import org.apache.ibatis.type.BaseTypeHandler;
    import org.apache.ibatis.type.JdbcType;
    import org.apache.ibatis.type.MappedTypes;

    import java.io.InputStream;
    import java.sql.*;

    // (1)
    public class BlobInputStreamTypeHandler extends BaseTypeHandler<InputStream> {

        // (2)
        @Override
        public void setNonNullParameter(PreparedStatement ps, int i, InputStream parameter,
                                        JdbcType jdbcType) throws SQLException {
            ps.setBlob(i, parameter);
        }

        // (3)
        @Override
        public InputStream getNullableResult(ResultSet rs, String columnName)
                throws SQLException {
            return toInputStream(rs.getBlob(columnName));
        }

        // (3)
        @Override
        public InputStream getNullableResult(ResultSet rs, int columnIndex)
                throws SQLException {
            return toInputStream(rs.getBlob(columnIndex));
        }

        // (3)
        @Override
        public InputStream getNullableResult(CallableStatement cs, int columnIndex)
                throws SQLException {
            return toInputStream(cs.getBlob(columnIndex));
        }

        private InputStream toInputStream(Blob blob) throws SQLException {
            // (4)
            if (blob == null) {
                return null;
            } else {
                return blob.getBinaryStream();
            }
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - MyBatis3から提供されている\ ``BaseTypeHandler``\を親クラスに指定する。

        その際、\ ``BaseTypeHandler``\のジェネリック型には、\ ``InputStream``\を指定する。
    * - (2)
      - \ ``InputStream``\を\ ``PreparedStatement``\に設定する処理を実装する。
    * - (3)
      - \ ``ResultSet``\又は\ ``CallableStatement``\から取得した\ ``Blob``\から\ ``InputStream``\を取得し、返り値として返却する。
    * - (4)
      - \ ``null``\を許可するカラムの場合、取得した\ ``Blob``\が\ ``null``\になる可能性があるため、
        \ ``null``\チェックを行ってから\ ``InputStream``\を取得する必要がある。

        上記実装例では、3つのメソッドで同じ処理が必要になるため、privateメソッドを作成している。

|

.. _DataAccessMyBatis3HowToExtendTypeHandlerClob:

CLOB用のTypeHandlerの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

MyBatis3では、CLOBを\ ``java.lang.String``\にマッピングするためのTypeHandlerを提供している。
ただし、扱うデータの容量が大きい場合は、\ ``java.io.Reader``\とマッピングが必要なケースがある。

以下に、CLOBと\ ``java.io.Reader``\をマッピングするためのTypeHandlerの実装例を示す。

 .. code-block:: java

    package com.example.infra.mybatis.typehandler;

    import org.apache.ibatis.type.BaseTypeHandler;
    import org.apache.ibatis.type.JdbcType;

    import java.io.Reader;
    import java.sql.*;

    // (1)
    public class ClobReaderTypeHandler extends BaseTypeHandler<Reader> {

        // (2)
        @Override
        public void setNonNullParameter(PreparedStatement ps, int i, Reader parameter,
                                        JdbcType jdbcType) throws SQLException {
            ps.setClob(i, parameter);
        }

        // (3)
        @Override
        public Reader getNullableResult(ResultSet rs, String columnName)
            throws SQLException {
            return toReader(rs.getClob(columnName));
        }

        // (3)
        @Override
        public Reader getNullableResult(ResultSet rs, int columnIndex)
            throws SQLException {
            return toReader(rs.getClob(columnIndex));
        }

        // (3)
        @Override
        public Reader getNullableResult(CallableStatement cs, int columnIndex)
            throws SQLException {
            return toReader(cs.getClob(columnIndex));
        }

        private Reader toReader(Clob clob) throws SQLException {
            // (4)
            if (clob == null) {
                return null;
            } else {
                return clob.getCharacterStream();
            }
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - MyBatis3から提供されている\ ``BaseTypeHandler``\を親クラスに指定する。

        その際、\ ``BaseTypeHandler``\のジェネリック型には、\ ``Reader``\を指定する。
    * - (2)
      - \ ``Reader``\を\ ``PreparedStatement``\に設定する処理を実装する。
    * - (3)
      - \ ``ResultSet``\又は\ ``CallableStatement``\から取得した\ ``Clob``\から\ ``Reader``\を取得し、返り値として返却する。
    * - (4)
      - \ ``null``\を許可するカラムの場合、取得した\ ``Clob``\が\ ``null``\になる可能性があるため、
        \ ``null``\チェックを行ってから\ ``Reader``\を取得する必要がある。

        上記実装例では、3つのメソッドで同じ処理が必要になるため、privateメソッドを作成している。

|

.. _DataAccessMyBatis3HowToExtendTypeHandlerJoda:

Joda-Time用のTypeHandlerの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| MyBatis3では、Joda-Timeのクラス(\ ``org.joda.time.DateTime``\ 、\ ``org.joda.time.LocalDateTime``\、\ ``org.joda.time.LocalDate``\など)はサポートされていない。
| そのため、EntityクラスのフィールドにJoda-Timeのクラスを使用する場合は、Joda-Time用のTypeHandlerを用意する必要がある。

``org.joda.time.DateTime``\と\ ``java.sql.Timestamp``\をマッピングするためのTypeHandlerの実装例を、以下に示す。

 .. note::

    Jada-Timeから提供されている他のクラス(\ ``LocalDateTime``\、\ ``LocalDate``\、\ ``LocalTime``\など)も同じ要領で実装すればよい。


 .. code-block:: java

    package com.example.infra.mybatis.typehandler;

    import java.sql.CallableStatement;
    import java.sql.PreparedStatement;
    import java.sql.ResultSet;
    import java.sql.SQLException;
    import java.sql.Timestamp;

    import org.apache.ibatis.type.BaseTypeHandler;
    import org.apache.ibatis.type.JdbcType;
    import org.joda.time.DateTime;

    // (1)
    public class DateTimeTypeHandler extends BaseTypeHandler<DateTime> {

        // (2)
        @Override
        public void setNonNullParameter(PreparedStatement ps, int i,
                DateTime parameter, JdbcType jdbcType) throws SQLException {
            ps.setTimestamp(i, new Timestamp(parameter.getMillis()));
        }

        // (3)
        @Override
        public DateTime getNullableResult(ResultSet rs, String columnName)
                throws SQLException {
            return toDateTime(rs.getTimestamp(columnName));
        }

        // (3)
        @Override
        public DateTime getNullableResult(ResultSet rs, int columnIndex)
                throws SQLException {
            return toDateTime(rs.getTimestamp(columnIndex));
        }

        // (3)
        @Override
        public DateTime getNullableResult(CallableStatement cs, int columnIndex)
                throws SQLException {
            return toDateTime(cs.getTimestamp(columnIndex));
        }

        private DateTime toDateTime(Timestamp timestamp) {
            // (4)
            if (timestamp == null) {
                return null;
            } else {
                return new DateTime(timestamp.getTime());
            }
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - MyBatis3から提供されている\ ``BaseTypeHandler``\を親クラスに指定する。

        その際、\ ``BaseTypeHandler``\のジェネリック型には、\ ``DateTime``\を指定する。
    * - (2)
      - \ ``DateTime``\を\ ``Timestamp``\に変換し、\ ``PreparedStatement``\に設定する処理を実装する。
    * - (3)
      - \ ``ResultSet``\又は\ ``CallableStatement``\から取得した\ ``Timestamp``\を\ ``DateTime``\に変換し、返り値として返却する。
    * - (4)
      - \ ``null``\を許可するカラムの場合、\ ``Timestamp``\が\ ``null``\になる可能性があるため、
        \ ``null``\チェックを行ってから\ ``DateTime``\に変換する必要がある。

        上記実装例では、3つのメソッドで同じ処理が必要になるため、privateメソッドを作成している。


|

.. _DataAccessMyBatis3HowToExtendResultHandler:

ResultHandlerの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

MyBatis3では、検索結果を1件単位で処理する仕組みを提供している。

この仕組みを利用すると、

* DBより取得した値をJavaの処理で加工する
* DBより取得した値などをJavaの処理として集計する

といった処理を行う際に、同時に消費するメモリの容量を最小限に抑える事ができる。

例えば、検索結果をCSV形式のデータとしてダウンロードするような処理を実装する場合は、
検索結果を1件単位で処理する仕組みを使用するとよい。

 .. note::

    **検索結果が大量になる可能性があり、且つJavaの処理で検索結果を1件ずつ処理する必要がある場合は、**
    **この仕組みを使用することを強く推奨する。**

    検索結果を1件単位で処理する仕組みを使用しない場合、
    検索結果の全データ「1データのサイズ * 検索結果件数」をメモリ上に同時に確保することになり、
    全てのデータに対して処理が終了するまでGC候補になることはない。

    一方、検索結果を1件単位で処理する仕組みを使用した場合、
    基本的には「1データのサイズ」をメモリ上に確保するだけであり、
    1データの処理を終えた時点でGC候補となる。

    例えば「1データのサイズ」が\ ``2KB``\で「検索結果件数」が\ ``10,000``\件だった場合、

    * まとめて処理を行う場合は、\ ``20MB``\のメモリ
    * 1件単位で処理を行う場合は、\ ``2KB``\のメモリ

    が同時に消費される。シングルスレッドで動くアプリケーションであれば問題になる事はないが、
    Webアプリケーションの様なマルチスレッドで動くアプリケーションの場合は、問題になる事がある。

    仮に100スレッドで同時に処理を行った場合、

    * まとめて処理を行う場合は、\ ``2GB``\のメモリ
    * 1件単位で処理を行う場合は、\ ``200KB``\のメモリ

    が同時に消費される。

    結果として、

    * まとめて処理を行う場合は、ヒープの最大サイズの指定によっては、メモリ枯渇によるシステムダウンやフルGCの頻発による性能劣化などが起こる可能性が高まる。
    * 1件単位で処理を行う場合は、メモリ枯渇やコストの高いGC処理が発生する可能性を抑える事ができる。

    上記に挙げた数字は目安であり、実際の計測値ではないという点を補足しておく。

|

以下に、検索結果をCSV形式のデータとしてダウンロードする処理の実装例を示す。

* Repositoryインタフェースにメソッドを定義する。

 .. code-block:: java

    public interface TodoRepository {

        // (1) (2)
        void collectAllByCriteria(TodoCriteria criteria, ResultHandler<Todo> resultHandler);

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - メソッドの引数として、  \ ``org.apache.ibatis.session.ResultHandler``\を指定する。
    * - (2)
      - メソッドの返り値は、\ ``void``\型を指定する。

        \ ``void``\以外を指定すると、\ ``ResultHandler``\が呼び出されなくなるので、注意すること。

|

* マッピングファイルにSQLを定義する。

 .. code-block:: xml

    <!-- (3) -->
    <select id="collectAllByCriteria" parameterType="TodoCriteria" resultType="Todo">
        SELECT
            todo_id,
            todo_title,
            finished,
            created_at,
            version
        FROM
            t_todo
        <where>
            <if test="title != null">
                <bind name="titleContainingCondition"
                      value="@org.terasoluna.gfw.common.query.QueryEscapeUtils@toContainingCondition()" />
                todo_titile LIKE #{titleContainingCondition} ESCAPE '~'
            </if>
            <if test="createdAt != null">
                <![CDATA[
                AND created_at < #{createdAt}
                ]]>
            </if>
        </where>
    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (3)
      - マッピングファイルの実装は、通常の検索処理と同じである。

 .. warning:: **fetchSize属性の指定について**

    大量のデータを返すようなクエリを記述する場合には、\ ``fetchSize``\属性に適切な値を設定すること。
    \ ``fetchSize``\属性は、JDBCドライバとデータベース間の通信において、
    一度の通信で取得するデータの件数を設定するパラメータである。

    \ ``fetchSize``\属性を省略した場合は、JDBCドライバのデフォルト値が利用されるため、
    デフォルト値が全件取得するJDBCドライバの場合、メモリの枯渇の原因になる可能性があるので、
    注意が必要となる。


|

* ServiceクラスにRepositoryをDIし、Repositoryインターフェースのメソッドを呼び出す。

 .. code-block:: java

    public class TodoServiceImpl implements TodoService {

        private static final DateTimeFormatter DATE_FORMATTER =
            DateTimeFormat.forPattern("yyyy/MM/dd");

        @Inject
        TodoRepository todoRepository;

        public void downloadTodos(TodoCriteria criteria,
            final BufferedWriter downloadWriter) {

            // (4)
            ResultHandler<Todo> handler = new ResultHandler<Todo>() {
                @Override
                public void handleResult(ResultContext<? extends Todo> context) {
                    Todo todo = context.getResultObject();
                    StringBuilder sb = new StringBuilder();
                    try {
                        sb.append(todo.getTodoId());
                        sb.append(",");
                        sb.append(todo.getTodoTitle());
                        sb.append(",");
                        sb.append(todo.isFinished());
                        sb.append(",");
                        sb.append(DATE_FORMATTER.print(todo.getCreatedAt().getTime()));
                        downloadWriter.write(sb.toString());
                        downloadWriter.newLine();
                    } catch (IOException e) {
                        throw new SystemException("e.xx.fw.9001", e);
                    }
                }
            };

            // (5)
            todoRepository.collectAllByCriteria(criteria, handler);

        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (4)
      - \ ``ResultHandler``\のインスタンスを生成する。

        \ ``ResultHandler``\の\ ``handleResult``\メソッドの中に、1件毎に行う処理を実装する。

        上記例では、\ ``ResultHandler``\の実装クラスは作らず、無名オブジェクトとして\ ``ResultHandler``\の実装を行っている。
        実装クラスを作成してもよいが、複数の処理で共有する必要がない場合は、無理に実装クラスを作成する必要はない。
    * - (5)
      - Repositoryインタフェースのメソッドを呼び出す。

        メソッドを呼び出す際に、(4)で生成した\ ``ResultHandler``\のインスタンスを引数に指定する。

        \ ``ResultHandler``\を使用した場合、MyBatisは以下の処理を検索結果の件数分繰り返す。

        * 検索結果からレコードを取得し、JavaBeanにマッピングを行う。
        * \ ``ResultHandler``\インスタンスの\ ``handleResult(ResultContext)``\メソッドを呼び出す。

 .. warning:: **ResultHandler使用時の注意点**

    \ ``ResultHandler``\を使用する場合、以下の２点に注意すること。

    * MyBatis3では、検索処理の性能向上させる仕組みとして、検索結果をローカルキャッシュ及びグローバルな2次キャッシュに保存する仕組みを提供しているが、\ ``ResultHandler``\を引数に取るメソッドから返されるデータはキャッシュされない。
    * 手動マッピングを使用して複数行のデータを一つのJavaオブジェクトにマッピングするステートメントに対して\ ``ResultHandler``\を使用した場合、不完全な状態(関連Entityのオブジェクトがマッピングされる前の状態)のオブジェクトが渡されるケースがある。

 .. tip:: **ResultContextのメソッドについて**

    \ ``ResultHandler#handleResult``\メソッドの引数である\ ``ResultContext``\には、以下のメソッドが用意がされている。

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.65\linewidth}|
     .. list-table::
         :header-rows: 1
         :widths: 10 15 65

         * - 項番
           - メソッド
           - 説明
         * - (1)
           - getResultObject
           - 検索結果がマッピングされたオブジェクトを取得するためのメソッド。
         * - (2)
           - getResultCount
           - \ ``ResultHandler#handleResult``\メソッドの呼び出し回数を取得するためのメソッド。
         * - (3)
           - stop
           - 以降のレコードに対する処理を中止するようにMyBatis側に通知するためのメソッド。
             このメソッドは、以降のレコードを全て破棄したい場合に使用するとよい。

    \ ``ResultContext``\には\ ``isStopped``\というメソッドもあるが、これはMyBatis側が使用するメソッドなので、説明は割愛する。

|

.. _DataAccessMyBatis3HowToExtendExecutorType:

SQL実行モードの利用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

MyBatis3では、SQLを実行するモードとして以下の3種類を用意しており、デフォルトは\ ``SIMPLE``\である。

ここでは、

* 実行モードの使用方法
* バッチモードのRepository利用時の注意点

| について説明を行う。
| 実行モードの説明については、「:ref:`DataAccessMyBatis3HowToUseSettingsExecutorType`」を参照されたい。

.. _DataAccessMyBatis3HowToExtendExecutorTypeReuse:

PreparedStatement再利用モードの利用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

実行モードを\ ``SIMPLE``\から\ ``REUSE``\に変更した場合、MyBatis内部の\ ``PreparedStatement``\の扱い方は変わるが、
MyBatisの動作(使い方)は変わらない。

実行モードをデフォルト(\ ``SIMPLE``\)から\ ``REUSE``\に変更する方法を、以下に示す。

* :file:`projectName-domain/src/main/resources/META-INF/mybatis/mybatis-config.xml` に設定を追加する。

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
            PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
        <settings>
            <!-- (1) -->
            <setting name="defaultExecutorType" value="REUSE"/>
        </settings>
    </configuration>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - \ ``defaultExecutorType``\に\ ``REUSE``\に変更する。

        上記設定を行うと、デフォルト動作がPreparedStatement再利用モードになる。

|

.. _DataAccessMyBatis3HowToExtendExecutorTypeBatch:

バッチモードの利用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Mapperインタフェースの更新系メソッドの呼び出しを、全てバッチモードで実行する場合は、
「:ref:`DataAccessMyBatis3HowToExtendExecutorTypeReuse`」と同じ方法で、
実行モードを\ ``BATCH``\モードに変更すればよい。

ただし、バッチモードはいくつかの制約事項があるため、
実際のアプリケーション開発では\ ``SIMPLE``\又は\ ``REUSE``\モードと共存して使用するケースが想定される。

例えば、

* 大量のデータ更新を伴い性能要件を充たす事が最優先される処理では、バッチモードを使用する。
* 楽観ロックの制御などデータの一貫性を保つために更新結果の判定が必要な処理では、\ ``SIMPLE``\又は\ ``REUSE``\モードを使用する。

等の使い分けを行う場合である。

 .. warning:: **実行モードを共存して使用する際の注意点**

    アプリケーション内で複数の実行モードを使用する場合は、
    **同一トランザクション内で実行モードを切り替える事が出来ないという点に注意すること。**

    仮に同一トランザクション内で複数の実行モードを使用した場合は、MyBatisが矛盾を検知しエラーとなる。

    これは、同一トランザクション内の処理において、

    * XxxRepositoryのメソッド呼び出しは\ ``BATCH``\モードで実行する
    * YyyRepositoryのメソッド呼び出しは\ ``REUSE``\モードで実行する

    といった事が出来ないという事を意味する。

    本ガイドラインをベースに作成するアプリケーションのトランザクション境界は、
    Service又はRepositoryとなる。
    そのため、**アプリケーション内で複数の実行モードを使用する場合は、**
    **ServiceやRepositoryの設計を行う際に、実行モードを意識する必要がある。**

    トランザクションを分離させたい場合は、ServiceやRepositoryのメソッドアノテーションとして、
    \ ``@Transactional(propagation = Propagation.REQUIRES_NEW)``\を指定する事で実現する事ができる。
    トランザクション管理の詳細については、「:ref:`service_transaction_management`」を参照されたい。

|

以降では、

* 複数の実行モードを共存させるための設定方法
* アプリケーションの実装例

について説明を行う。

|

.. _DataAccessMyBatis3HowToExtendExecutorTypeBatchIndividualSetting:

個別にバッチモードのRepositoryを作成するための設定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

特定のRepositoryに対してバッチモードのRepositoryを作成したい場合は、
MyBatis-Springから提供されている\ ``org.mybatis.spring.mapper.MapperFactoryBean``\を使用して、
RepositoryのBean定義を行えばよい。

下記の設定例では、

* 通常使用するRepositoryとして\ ``REUSE``\モードのRepository
* 特定のRepositoryに対して\ ``BATCH``\モードのRepository

をBean登録している。

- :file:`projectName-domain/src/main/resources/META-INF/spring/projectName-infra.xml` にBean定義を追加する。

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:context="http://www.springframework.org/schema/context"
           xmlns:mybatis="http://mybatis.org/schema/mybatis-spring"
           xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd
           http://mybatis.org/schema/mybatis-spring
           http://mybatis.org/schema/mybatis-spring.xsd">

        <bean id="sqlSessionFactory"
              class="org.mybatis.spring.SqlSessionFactoryBean">
            <property name="dataSource" ref="dataSource"/>
            <property name="configLocation"
                      value="classpath:META-INF/mybatis/mybatis-config.xml"/>
        </bean>

        <!-- (1) -->
        <bean id="sqlSessionTemplate"
              class="org.mybatis.spring.SqlSessionTemplate">
            <constructor-arg index="0" ref="sqlSessionFactory"/>
            <constructor-arg index="1" value="REUSE"/>
        </bean>

        <mybatis:scan base-package="com.example.domain.repository"
                      template-ref="sqlSessionTemplate"/> <!-- (2) -->

        <!-- (3) -->
        <bean id="batchSqlSessionTemplate"
              class="org.mybatis.spring.SqlSessionTemplate">
            <constructor-arg index="0" ref="sqlSessionFactory"/>
            <constructor-arg index="1" value="BATCH"/>
        </bean>

        <!-- (4) -->
        <bean id="todoBatchRepository"
              class="org.mybatis.spring.mapper.MapperFactoryBean">
            <!-- (5) -->
            <property name="mapperInterface"
                      value="com.example.domain.repository.todo.TodoRepository"/>
            <!-- (6) -->
            <property name="sqlSessionTemplate" ref="batchSqlSessionTemplate"/>
        </bean>

    </beans>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - 通常使用するRepositoryで利用するための\ ``SqlSessionTemplate``\をBean定義する。
    * - (2)
      - 通常使用するRepositoryをスキャンしBean登録する。

        \ ``template-ref``\属性に、(1)で定義した\ ``SqlSessionTemplate``\を指定する。
    * - (3)
      - バッチモードのRepositoryで利用するための\ ``SqlSessionTemplate``\をBean定義する。
    * - (4)
      - バッチモード用のRepositoryをBean定義する。

        \ ``id``\属性には、(2)でスキャンしたRepositoryのBean名と重複しない値を指定する。
        (2)でスキャンされたRepositoryのBean名は、インタフェース名を「lowerCamelCase」にした値にとなる。

        上記例では、バッチモード用の\ ``TodoRepository``\が\ ``todoBatchRepository``\という名前のBeanでBean登録される。
    * - (5)
      - \ ``mapperInterface``\プロパティには、
        バッチモードを利用するRepositoryのインタフェース名(FQCN)を指定する。
    * - (6)
      - \ ``sqlSessionTemplate``\プロパティには、
        (3)で定義したバッチモード用の\ ``SqlSessionTemplate``\を指定する。

 .. note::

    ``SqlSessionTemplate``\をBean定義すると、アプリケーション終了時に以下の様なWARNログが出力される。

    これは、\ ``SqlSession``\インタフェースが\ ``java.io.Closeable``\を継承しているため、
    SpringのApplicationContextの終了処理時に\ ``close``\メソッドが呼び出されている事が原因である。

     .. code-block:: text

        21:12:35.999 [Thread-2] WARN  o.s.b.f.s.DisposableBeanAdapter - Invocation of destroy method 'close' failed on bean with name 'sqlSessionTemplate'
        java.lang.UnsupportedOperationException: Manual close is not allowed over a Spring managed SqlSession
            at org.mybatis.spring.SqlSessionTemplate.close(SqlSessionTemplate.java:310) ~[mybatis-spring-1.2.2.jar:1.2.2]
            at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_20]
            at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_20]
            at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_20]
            at java.lang.reflect.Method.invoke(Method.java:483) ~[na:1.8.0_20]

    上記ログが出力されても、アプリケーションの動作に影響はないため、システム運用上問題がなければ対策は不要である。

    ただし、ログ監視などシステム運用上問題がある場合は、
    SpringのApplicationContextの終了処理時に呼び出されるメソッド(\ ``destroy-method``\属性)を指定する事で、
    ログ出力を抑止する事ができる。

    下記例では、\ ``getExecutorType``\メソッドを呼び出すように指定している。
    \ ``getExecutorType``\メソッドは、コンストラクタ引数で指定した実行モードを返却するだけのメソッドであり、
    このメソッドを呼び出しても他への副作用はない。

     .. code-block:: xml
        :emphasize-lines: 3

        <bean id="batchSqlSessionTemplate"
              class="org.mybatis.spring.SqlSessionTemplate"
              destroy-method="getExecutorType">
            <constructor-arg index="0" ref="sqlSessionFactory"/>
            <constructor-arg index="1" value="BATCH"/>
        </bean>

|

.. _DataAccessMyBatis3HowToExtendExecutorTypeBatchBulkSetting:

一括でバッチモードのRepositoryを作成するための設定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

一括でバッチモードのRepositoryを作成したい場合は、
MyBatis-Springから提供されているスキャン機能(\ ``mybatis:scan``\要素)を使用して、
RepositoryのBean定義を行えばよい。

下記の設定例では、全てのRepositoryに対して、\ ``REUSE``\モードと\ ``BATCH``\モードのRepositoryをBean登録している。

* \ ``BeanNameGenerator``\を作成する。

 .. code-block:: java

    package com.example.domain.repository;

    import org.springframework.beans.factory.config.BeanDefinition;
    import org.springframework.beans.factory.support.BeanDefinitionRegistry;
    import org.springframework.beans.factory.support.BeanNameGenerator;
    import org.springframework.util.ClassUtils;

    import java.beans.Introspector;

    // (1)
    public class BachRepositoryBeanNameGenerator implements BeanNameGenerator {
        // (2)
        @Override
        public String generateBeanName(BeanDefinition definition, BeanDefinitionRegistry registry) {
            String defaultBeanName = Introspector.decapitalize(ClassUtils.getShortName(definition
                    .getBeanClassName()));
            return defaultBeanName.replaceAll("Repository", "BatchRepository");
        }
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - SpringのApplicationContextに登録するBean名を生成するクラスを作成する。

        このクラスは、通常使用する\ ``REUSE``\モードのRepositoryのBean名と、
        \ ``BATCH``\モードのBean名が重複しないようにするために必要なクラスである。
    * - (2)
      - Bean名を生成するためのメソッドを実装する。

        上記例では、Bean名のsuffixを\ ``BatchRepository``\とする事で、
        通常使用される\ ``REUSE``\モードのRepositoryのBean名と重複しないようにしている。

|

* :file:`projectName-domain/src/main/resources/META-INF/spring/projectName-infra.xml` にBean定義を追加する。

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:context="http://www.springframework.org/schema/context"
           xmlns:mybatis="http://mybatis.org/schema/mybatis-spring"
           xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd
           http://mybatis.org/schema/mybatis-spring
           http://mybatis.org/schema/mybatis-spring.xsd">

        <bean id="sqlSessionFactory"
              class="org.mybatis.spring.SqlSessionFactoryBean">
            <property name="dataSource" ref="dataSource"/>
            <property name="configLocation"
                      value="classpath:META-INF/mybatis/mybatis-config.xml"/>
        </bean>

        <!-- ... -->

        <bean id="batchSqlSessionTemplate"
              class="org.mybatis.spring.SqlSessionTemplate">
            <constructor-arg index="0" ref="sqlSessionFactory"/>
            <constructor-arg index="1" value="BATCH"/>
        </bean>

        <!-- (3) -->
        <mybatis:scan base-package="com.example.domain.repository"
            template-ref="batchSqlSessionTemplate"
            name-generator="com.example.domain.repository.BatchRepositoryBeanNameGenerator"/>

    </beans>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.65\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 15 65

    * - 項番
      - 属性
      - 説明
    * - (3)
      - \-
      - \ ``mybatis:scan``\要素を使用して、バッチモードのRepositoryをBean登録する。
    * -
      - base-package
      - Repositoryをスキャンするベースパッケージを指定する。

        指定パッケージの配下に存在するRepositoryインタフェースがスキャンされ、
        SpringのApplicationContextにBean登録される。
    * -
      - template-ref
      - バッチモード用の\ ``SqlSessionTemplate``\のBeanを指定する。
    * -
      - name-generator
      - スキャンしたRepositoryのBean名を生成するためのクラスを指定する。

        具体的には、(1)で作成したクラスのクラス名(FQCN)を指定する。

        この指定を省略した場合、Bean名が重複するため、
        バッチモードのRepositoryはSpringのApplicationContextに登録されない。

|

.. _DataAccessMyBatis3HowToExtendExecutorTypeBatchUsage:

バッチモードのRepositoryの使用例
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

以下に、バッチモードのRepositoryを使用してデータベースにアクセスするための実装例を示す。

 .. code-block:: java

    @Transactional
    @Service
    public class TodoServiceImpl implements TodoService {

        // (1)
        @Inject
        @Named("todoBatchRepository")
        TodoRepository todoBatchRepository;

        @Override
        public void updateTodos(List<Todo> todos) {
            for (Todo todo : todos) {
                // (2)
                todoBatchRepository.update(todo);
            }
        }

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - バッチモードのRepositoryをインジェクションする。
    * - (2)
      - バッチモードのRepositoryのメソッドを呼び出し、Entityの更新を行う。

        バッチモードのRepositoryの場合は、メソッドを呼び出したタイミングでSQLが実行されないため、
        メソッドから返却される更新結果は無視する必要がある。

        Entityを更新するためのSQLは、トランザクションがコミットされる直前にバッチ実行され、
        エラーがなければコミットされる。

 .. note:: **バッチ実行のタイミングについて**

    SQLがバッチ実行されるタイミングは、基本的には以下の場合である。

    * トランザクションがコミットされる直前
    * クエリ(SELECT)を実行する直前

    Repositoryのメソッドの呼び出し順番に関する注意点は、「:ref:`DataAccessMyBatis3HowToExtendExecutorTypeBatchNotesMethodCallOrder`」を参照されたい。

|

.. _DataAccessMyBatis3HowToExtendExecutorTypeBatchNotes:

バッチモードのRepository利用時の注意点
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

バッチモードのRepositoryを利用する場合、Serviceクラスの実装として、
以下の点に注意する必要がある。

* :ref:`DataAccessMyBatis3HowToExtendExecutorTypeBatchNotesUpdateResult`
* :ref:`DataAccessMyBatis3HowToExtendExecutorTypeBatchNotesDuplicate`
* :ref:`DataAccessMyBatis3HowToExtendExecutorTypeBatchNotesMethodCallOrder`

.. _DataAccessMyBatis3HowToExtendExecutorTypeBatchNotesUpdateResult:

更新結果の判定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

バッチモードのRepositoryを使用した場合、更新結果の妥当性をチェックする事ができない。

バッチモードを使用する場合、Mapperインタフェースのメソッドから返却される更新結果は、

* 返り値が数値(\ ``int``\や\ ``long``\)の場合は、\ `固定値 <http://mybatis.github.io/mybatis-3/apidocs/reference/org/apache/ibatis/executor/BatchExecutor.html#BATCH_UPDATE_RETURN_VALUE>`_\ (\ ``org.apache.ibatis.executor.BatchExecutor#BATCH_UPDATE_RETURN_VALUE``\ )
* 返り値が\ ``boolean``\の場合は、\ ``false``\

が返却される。

これは、Mapperインタフェースのメソッドを呼び出したタイミングではSQLが発行されず、
バッチ実行用にキューイング(\ ``java.sql.Statement#addBatch()``\)される仕組みになっているためである。

**つまり、更新結果の妥当性をチェックする必要がある場合(楽観ロックによる排他制御処理など)では、**
**バッチモードを使用する事はできない。**

 .. tip::

    MyBatis3自体の機能としては、
    \ ``org.apache.ibatis.session.SqlSession``\インタフェースのメソッド(\ ``flushStatements``\)を使用すると、
    バッチ実行用にキューイングされているSQLを実行し、更新結果を受け取る事ができる。
    ただし、本ガイドラインでは\ ``SqlSession``\を直接使用する前提ではないため、
    可能な限り\ ``SqlSession``\を直接使用しないようにする事を推奨する。

    どうしても\ ``SqlSession``\インタフェースのメソッドを直接呼び出す必要がある場合は、
    「`MyBatis-Spring REFERENCE DOCUMENTATION(Using an SqlSession) <http://mybatis.github.io/spring/sqlsession.html>`_\」
    を参照されたい。
    MyBatis-Springが提供している\ ``org.mybatis.spring.SqlSessionTemplate``\を使用するという点がポイントである。

 .. warning:: **バッチモード使用時のJDBCドライバの動作について**

    \ ``SqlSession``\インタフェースを使用するとバッチ実行時の更新結果を受け取る事ができると前述したが、
    JDBCドライバから返却される更新結果が「処理したレコード数」になる保証はない。

    これは、使用するJDBCドライバの実装にも依存する部分なので、
    使用するJDBCドライバの仕様を確認しておく必要がある。

|

これは、以下の様な実装が出来ないことを意味している。

 .. code-block:: java

    @Transactional
    @Service
    public class TodoServiceImpl implements TodoService {

        @Inject
        @Named("todoBatchRepository")
        TodoRepository todoBatchRepository;

        @Override
        public void updateTodos(List<Todo> todos) {
            for (Todo todo : todos) {
                boolean updateSuccess = todoBatchRepository.update(todo);
                // (1)
                if (!updateSuccess) {
                    // ...
                }
            }
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - 上記例のように実装した場合、更新結果は常に\ ``false``\になるため、
        必ず更新失敗時の処理が実行されてしまう。

|

.. _DataAccessMyBatis3HowToExtendExecutorTypeBatchNotesDuplicate:

一意制約違反の検知方法
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

バッチモードのRepositoryを使用した場合、
一意制約違反などのデータベースエラーをServiceの処理として検知する事が出来ないケースがある。

これは、Mapperインタフェースのメソッドを呼び出したタイミングではSQLが発行されず、
バッチ実行用にキューイング(\ ``java.sql.Statement#addBatch()``\)される仕組みになっているためであり、
以下の様な実装が出来ないことを意味している。

 .. code-block:: java

    @Transactional
    @Service
    public class TodoServiceImpl implements TodoService {

        @Inject
        @Named("todoBatchRepository")
        TodoRepository todoBatchRepository;

        @Override
        public void storeTodos(List<Todo> todos) {
            for (Todo todo : todos) {
                try {
                    todoBatchRepository.create(todo);
                // (1)
                } catch (DuplicateKeyException e) {
                    // ....
                }
            }
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - 上記例のように実装した場合、
        このタイミングで\ ``org.springframework.dao.DuplicateKeyException``\が発生することはないため、
        \ ``DuplicateKeyException``\補足後の処理が実行される事はない。

        これは、SQLがバッチ実行されるタイミングが、
        Serviceの処理が終わった後(トランザクションがコミットされる直前)に行われるためである。

|

.. _DataAccessMyBatis3HowToExtendExecutorTypeBatchNotesMethodCallOrder:

Repositoryのメソッドの呼び出し順番
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
バッチモードを使用する目的は更新処理の性能向上であるが、
Repositoryのメソッドの呼び出し順番を間違えると、性能向上につながらないケースがある。

バッチモードを使用して性能向上させるためには、以下のMyBatisの仕様を理解しておく必要がある。

* クエリ(SELECT)を実行すると、それまでキューイングされていたSQLがバッチ実行される。
* 連続して呼び出された更新処理(Repositoryのメソッド)毎に\ ``PreparedStatement``\が生成され、SQLをキューイングする。

これは、以下の様な実装をすると、バッチモードを利用するメリットがない事を意味している。

* 例１

 .. code-block:: java

    @Transactional
    @Service
    public class TodoServiceImpl implements TodoService {

        @Inject
        @Named("todoBatchRepository")
        TodoRepository todoBatchRepository;

        @Override
        public void storeTodos(List<Todo> todos) {
            for (Todo todo : todos) {
                // (1)
                Todo currentTodo = todoBatchRepository.findOne(todo.getTodoId());
                if (currentTodo == null) {
                    todoBatchRepository.create(todo);
                } else{
                    todoBatchRepository.update(todo);
                }
            }
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - 上記例のように実装した場合、繰返し処理の先頭にクエリを発行しているため、
        1件毎にSQLがバッチ実行される事になってしまう。
        これはほぼ、シンプルモード(\ ``SIMPLE``\)で実行しているのと同義である。

        上記のような処理が必要な場合は、
        PreparedStatement再利用モード(\ ``REUSE``\)のRepositoryを使用した方が効率的である。

* 例２

 .. code-block:: java

    @Transactional
    @Service
    public class TodoServiceImpl implements TodoService {

        @Inject
        @Named("todoBatchRepository")
        TodoRepository todoBatchRepository;

        @Override
        public void storeTodos(List<Todo> todos) {
            for (Todo todo : todos) {
                // (2)
                todoBatchRepository.create(todo);
                todoBatchRepository.createHistory(todo);
            }
        }

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (2)
      - 上記のような処理が必要な場合は、Repositoryのメソッドが交互に呼び出されているため、
        1件毎に\ ``PreparedStatement``\が生成されてしまう。
        これはほぼ、シンプルモード(\ ``SIMPLE``\)で実行しているのと同義である。

        上記のような処理が必要な場合は、
        PreparedStatement再利用モード(\ ``REUSE``\)のRepositoryを使用した方が効率的である。

|

.. _DataAccessMyBatis3HowToExtendStoredProcedure:

ストアドプロシージャの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

データベースに登録されているストアドプロシージャやファンクションを、
MyBatis3から呼び出す方法について説明を行う。

以下で説明する実装例では、PostgreSQLに登録されているファンクションを呼び出している。

* ストアードプロシージャ（ファンクション）を登録する。

 .. code-block:: guess

    /* (1) */
    CREATE FUNCTION findTodo(pTodoId CHAR)
    RETURNS TABLE(
        todo_id CHAR,
        todo_title VARCHAR,
        finished BOOLEAN,
        created_at TIMESTAMP,
        version BIGINT
    ) AS $$ BEGIN RETURN QUERY
    SELECT
        t.todo_id,
        t.todo_title,
        t.finished,
        t.created_at,
        t.version
    FROM
        t_todo t
    WHERE
        t.todo_id = pTodoId;
    END;
    $$ LANGUAGE plpgsql;

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - このファンクションは、指定されたIDのレコードを取得するファンクションである。

|

* Repositoryインタフェースにメソッドを定義する。

 .. code-block:: java

    // (2)
    public interface TodoRepository extends Repository {
        Todo findOne(String todoId);
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (2)
      - SQLを発行する際と同じインタフェースでよい。

|

* マッピングファイルにストアドプロシージャの呼び出し処理を実装する。

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <!-- (3) -->
        <select id="findOne" parameterType="string" resultType="Todo"
                statementType="CALLABLE">
            <!-- (4) -->
            {call findTodo(#{todoId})}
        </select>

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (3)
      - ストアドプロシージャを呼び出すステートメントを実装する。

        ストアドプロシージャを呼び出す場合は、\ ``statementType``\属性に\ ``CALLABLE``\を指定する。
        \ ``CALLABLE``\を指定すると、
        \ ``java.sql.CallableStatement``\を使用してストアドプロシージャが呼び出される。

        OUTパラメータをJavaBeanにマッピングするために、
        \ ``resultType``\属性又は\ ``resultMap``\属性を指定する。
    * - (4)
      - ストアドプロシージャを呼び出す。

        ストアドプロシージャ（ファクション）を呼び出す場合は、

        * \ ``{call Procedure or Function名(INパラメータ...)}``\

        形式で指定する。

        上記例では、\ ``findTodo``\という名前のファンクションに対して、
        INパラメータにIDを指定して呼び出している。

|

.. _DataAccessMyBatis3Appendix:

Appendix
--------------------------------------------------------------------------------

.. _DataAccessMyBatis3AppendixAboutMapperMechanism:

Mapperインタフェースの仕組みについて
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Mapperインタフェースを使用する場合、開発者はMapperインタフェースとマッピングファイルを作成するだけで、SQLを実行する事ができる。
| Mapperインタフェースの実装クラスは、MyBatis3がJDKのProxy機能を使用してアプリケーション実行時に生成されるため、
 開発者がMapperインタフェースの実装クラスを作成する必要はない。

| Mapperインタフェースは、MyBatis3から提供されているインタフェースの継承やアノテーションなどの定義は不要であり、
 単にJavaのインタフェースとして作成すればよい。
| 以下に、Mapperインタフェースとマッピングファイルの作成例、及びアプリケーション(Service)での利用例を示す。
| ここでは、開発者が作成する成果物をイメージしてもらう事が目的なので、コードに対する説明はポイントとなる点に絞って行っている。

- Mapperインタフェースの作成例

  本ガイドラインでは、MyBatis3のMapperインタフェースをRepositoryインタフェースとして使用することを前提としているため、
  インタフェース名は、「Entity名」 + \ ``"Repository"`` \というネーミングにしている。

 .. code-block:: java

    package com.example.domain.repository.todo;

    import com.example.domain.model.Todo;

    public interface TodoRepository {
        Todo findOne(String todoId);
    }

- マッピングファイルの作成例

  マッピングファイルでは、ネームスペースとしてMapperインタフェースのFQCN(Fully Qualified Class Name)を指定し、
  Mapperインタフェースに定義したメソッドの呼び出し時に実行するSQLとの紐づけは、
  各種ステートメントタグ(insert/update/delete/selectタグ)のid属性に、メソッド名を指定する事で行う事ができる。

 .. code-block:: xml
    :emphasize-lines: 4, 12

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org/DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.domain.repository.todo.TodoRepository">

        <resultMap id="todoResultMap" type="Todo">
            <result column="todo_id" property="todoId" />
            <result column="title" property="title" />
            <result column="finished" property="finished" />
        </resultMap>

        <select id="findOne" parameterType="String" resultMap="todoResultMap">
          SELECT
            todo_id,
            title,
            finished
          FROM
            t_todo
          WHERE
            todo_id = #{todoId}
        </select>

    </mapper>

- アプリケーション(Service)でのMapperインタフェースの使用例

  アプリケーション(Service)からMapperインタフェースのメソッドを呼び出す場合は、Spring(DIコンテナ)によって注入されたMapperオブジェクトのメソッドを呼び出す。
  アプリケーション(Service)は、Mapperオブジェクトのメソッドを呼び出すことで、透過的にSQLが実行され、SQLの実行結果を得ることができる。

 .. code-block:: java
    :emphasize-lines: 12

    package com.example.domain.service.todo;

    import com.example.domain.model.Todo;
    import com.example.domain.repository.todo.TodoRepository;

    public class TodoServiceImpl implements TodoService {

        @Inject
        TodoRepository todoRepository;

        public Todo getTodo(String todoId){
            Todo todo = todoRepository.findOne(todoId);
            if(todo == null){
                throw new ResourceNotFoundException(
                    ResultMessages.error().add("e.ex.td.5001" ,todoId));
            }
            return todo;
        }

    }

|

以下に、Mapperインタフェースのメソッドを呼び出した際に、SQLが実行されるまでの処理フローについて説明を行う。


 .. figure:: images_DataAccessMyBatis3/DataAccessMyBatis3MapperMechanism.png
    :alt: Mapper mechanism
    :width: 100%
    :align: center

    **Picture - Mapper mechanism**

 .. tabularcolumns:: |p{0.1\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80


    * - 項番
      - 説明
    * - (1)
      - アプリケーションは、Mapperインタフェースに定義されているメソッドを呼び出す。

        Mapperインタフェースの実装クラス(MapperインタフェースのProxyオブジェクト)は、アプリケーション起動時にMyBatis3のコンポーネントによって生成される。
    * - (2)
      - MapperインタフェースのProxyオブジェクトは、\ ``MapperProxy`` \のinvokeメソッドを呼び出す。

        \ ``MapperProxy`` \は、Mapperインタフェースのメソッド呼び出しをハンドリングする役割をもつ。
    * - (3)
      - \ ``MapperProxy`` \は、呼び出されたMapperインタフェースのメソッドに対応する \ ``MapperMethod`` \を生成し、executeメソッドを呼び出す。

        \ ``MapperMethod`` \は、 呼び出されたMapperインタフェースのメソッドに対応する\ ``SqlSession`` \のメソッドを呼び出す役割をもつ。
    * - (4)
      - \ ``MapperMethod`` \は、 \ ``SqlSession`` \のメソッドを呼び出す。

        \ ``SqlSession`` \のメソッドを呼び出す際は、実行するSQLステートメントを特定するためのキー(以降、「ステートメントID」と呼ぶ)を引き渡している。
    * - (5)
      - \ ``SqlSession`` \は、指定されたステートメントIDをキーに、マッピングファイルよりSQLステートメントを取得する。
    * - (6)
      - \ ``SqlSession`` \は、マッピングファイルより取得したSQLステートメントに指定されているバインド変数に値を設定し、SQLを実行する。
    * - (7)
      - Mapperインタフェース(\ ``SqlSession`` \)は、SQLの実行結果をJavaBeanなどに変換して、アプリケーションに返却する。

        件数のカウントや、更新件数などを取得する場合は、プリミティブ型やプリミティブラッパ型などが返却値となるケースもある。


 .. tip:: **ステートメントIDとは**

    ステートメントIDは、実行するSQLステートメントを特定するためのキーであり、
    \ **「MapperインタフェースのFQCN + "." + 呼び出されたMapperインタフェースのメソッド名」** \というルールで生成される。

    \ ``MapperMethod`` \によって生成されたステートメントIDに対応するSQLステートメントをマッピングファイルに定義するためには、
    マッピングファイルのネームスペースに「MapperインタフェースのFQCN」、
    各種ステートメントタグのid属性に「Mapperインタフェースのメソッド名」を指定する必要がある。

|

.. _DataAccessMyBatis3AppendixSettingsTypeAlias:

TypeAliasの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
TypeAliasの設定は、基本的には\ ``package`` \要素を使用してパッケージ単位で設定すればよいが、

* クラス単位でエイリアス名を設定する方法
* デフォルトで付与されるエイリアス名を上書きする方法(任意のエイリアス名を指定する方法)

も用意されている。

TypeAliasをクラス単位に設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
TypeAliasの設定は、クラス単位で設定する事もできる。

- :file:`projectName-domain/src/main/resources/META-INF/mybatis/mybatis-config.xml`

 .. code-block:: xml
    :emphasize-lines: 2-4

    <typeAliases>
        <!-- (1) -->
        <typeAlias
            type="com.example.domain.repository.account.AccountSearchCriteria" />
        <package name="com.example.domain.model" />
    </typeAliases>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80

   * - 項番
     - 説明
   * - (1)
     - \ ``typeAlias`` \要素の\ ``type`` \属性に、エイリアスを設定するクラスの完全修飾クラス名(FQCN)を指定する。

       上記例だと、\ ``com.example.domain.repository.account.AccountSearchCriteria`` \クラスのエイリアス名は、
       \ ``AccountSearchCriteria`` \(パッケージの部分が除去された部分)となる。
       
       エイリアス名に任意の値を指定したい場合は、\ ``typeAlias`` \要素の\ ``alias`` \属性に任意のエイリアス名を指定することができる。


デフォルトで付与されるエイリアス名の上書き
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\ ``package`` \要素を使用してエイリアスを設定した場合や、
\ ``typeAlias`` \要素の\ ``alias`` \属性を省略してエイリアスを設定した場合は、
TypeAliasのエイリアス名は、完全修飾クラス名(FQCN)からパッケージの部分が除去された部分となる。

デフォルトで付与されるエイリアス名ではなく、任意のエイリアス名にしたい場合は、
TypeAliasを設定したいクラスに\ ``@org.apache.ibatis.type.Alias`` \アノテーションを指定する事で、
任意のエイリアス名を指定する事ができる。

- エイリアス設定対象のJavaクラス

 .. code-block:: java
    :emphasize-lines: 3

    package com.example.domain.model.book;

    @Alias("BookAuthor") // (1)
    public class Author {
       // ...
    }
    
 .. code-block:: java
    :emphasize-lines: 3

    package com.example.domain.model.article;

    @Alias("ArticleAuthor") // (1)
    public class Author {
       // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80

   * - 項番
     - 説明
   * - (1)
     - \ ``@Alias`` \アノテーションの\ ``value`` \属性に、エイリアス名を指定する。

       上記例だと、\ ``com.example.domain.model.book.Author`` \クラスのエイリアス名は、
       \ ``BookAuthor`` \となる。
       
       異なるパッケージの中に同じクラス名のクラスが格納されている場合は、この方法を使用することで、それぞれ異なるエイリアス名を設定する事ができる。
       ただし、本ガイドラインでは、クラス名は重複しないように設計する事を推奨する。
       上記例であれば、クラス名自体を\ ``BookAuthor`` \と\ ``ArticleAuthor`` \にすることを検討して頂きたい。

 .. tip::
 
    TypeAliasの エイリアス名は、
    
     * \ ``typeAlias`` \要素の\ ``alias`` \属性の指定値
     * ``@Alias`` \アノテーションの\ ``value`` \属性の指定値
     * デフォルトで付与されるエイリアス名(完全修飾クラス名からパッケージの部分が除去された部分)
    
    の優先順で適用される。


|

.. _DataAccessMyBatis3AppendixSwitchingSqlByDatabase:

データベースによるSQL切替について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

MyBatis3では、JDBCドライバから接続しているデータベースのベンダー情報を取得して、
使用するSQLを切り替える仕組み(\ ``org.apache.ibatis.mapping.VendorDatabaseIdProvider``\)を提供している。

この仕組みは、動作環境として複数のデータベースをサポートするようなアプリケーションを構築する際に有効である。

 .. note::

    本ガイドラインでは、環境依存するコンポーネントや設定ファイルについては、
    [projectName]-envというサブプロジェクトで管理し、
    ビルド時に実行環境にあったコンポーネントや設定ファイル作成を選択するスタイルを推奨している。

    [projectName]-envは、

    * 開発環境(ローカルのPC環境)
    * 各種試験環境
    * 商用環境

    毎の差分を吸収するためのサブプロジェクトであり、
    複数のデータベースをサポートするアプリケーションの開発でも利用する事ができる。

    基本的には、環境依存するコンポーネントや設定ファイルは、
    [projectName]-envというサブプロジェクトで管理する事を推奨するが、
    SQLのちょっとした違いを吸収したい場合は、本仕組みを使用してもよい。

    **アーキテクトは、データベースの違いによるSQLの環境依存をどのように実装するかの指針を明確に示すことで、**
    **アプリケーション全体として統一された実装となるように心がけてほしい。**

|

- :file:`projectName-domain/src/main/resources/META-INF/spring/projectName-infra.xml` にBean定義を追加する。

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:mybatis="http://mybatis.org/schema/mybatis-spring"
        xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://mybatis.org/schema/mybatis-spring
            http://mybatis.org/schema/mybatis-spring.xsd
        ">

        <import resource="classpath:/META-INF/spring/projectName-env.xml" />

        <!-- (1) -->
        <bean id="databaseIdProvider"
              class="org.apache.ibatis.mapping.VendorDatabaseIdProvider">
            <!-- (2) -->
            <property name="properties">
                <props>
                    <prop key="H2">h2</prop>
                    <prop key="PostgreSQL">postgresql</prop>
                </props>
            </property>
        </bean>

        <bean id="sqlSessionFactory"
            class="org.mybatis.spring.SqlSessionFactoryBean">
            <property name="dataSource" ref="dataSource" />
            <!-- (3) -->
            <property name="databaseIdProvider" ref="databaseIdProvider"/>
            <property name="configLocation"
                value="classpath:/META-INF/mybatis/mybatis-config.xml" />
        </bean>

        <mybatis:scan base-package="com.example.domain.repository" />

    </beans>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - MyBatis3から提供されている\ ``VendorDatabaseIdProvider``\をBean定義する。

        \ ``VendorDatabaseIdProvider``\は、
        JDBCドライバから取得したデータベースのプロダクト名(\ ``java.sql.DatabaseMetaData#getDatabaseProductName()``\)をデータベースIDとして扱うためのクラスである。
    * - (2)
      - \ ``properties``\プロパティには、JDBCドライバから取得したデータベースのプロダクト名とデータベースIDのマッピングを指定する。

        マッピング仕様ついては、「`MyBatis3 REFERENCE DOCUMENTATION(Configuration-databaseIdProvider-) <http://mybatis.github.io/mybatis-3/configuration.html#databaseIdProvider>`_\」を参照されたい。
    * - (3)
      - データベースIDを使用する\ ``SqlSessionFactoryBean``\の\ ``databaseIdProvider``\プロパティ対して、
        (1)で定義した\ ``DatabaseIdProvider``\を指定する。

        この指定を行うと、マッッピングファイルからデータベースIDを参照する事が可能となる。

 .. note::

    本ガイドラインでは、\ ``properties``\プロパティを指定して、
    データベースのプロダクト名とデータベースIDをマッピングする方式を推奨する。

    理由は、JDBCドライバから取得したデータベースのプロダクト名は、
    JDBCのバージョンによって変わる可能性があるためである。
    \ ``properties``\プロパティを使用すると、使用するバージョン毎のプロダクト名の違いを、
    一箇所で管理する事ができる。

|

- マッピングファイルの実装を行う。

 .. code-block:: xml

    <insert id="create" parameterType="Todo">
        <!-- (1) -->
        <selectKey keyProperty="todoId" resultType="string" order="BEFORE"
                   databaseId="h2">
            SELECT RANDOM_UUID()
        </selectKey>
        <selectKey keyProperty="todoId" resultType="string" order="BEFORE"
                   databaseId="postgresql">
            SELECT UUID_GENERATE_V4()
        </selectKey>

        INSERT INTO
          t_todo
        (
            todo_id
            ,todo_title
            ,finished
            ,created_at
            ,version
        )
        VALUES
        (
            #{todoId}
            ,#{todoTitle}
            ,#{finished}
            ,#{createdAt}
            ,#{version}
        )
    </insert>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - ステートメント要素(\ ``select``\要素、\ ``update``\要素、\ ``sql``\要素など)をデータベース毎に切り替えたい場合は、
        各要素の\ ``databaseId``\属性にデータベースIDを指定する。

        \ ``databaseId``\属性を指定すると、データベースIDが一致するステートメント要素が使用される。

        上記例では、データベース固有のUUID生成関数を呼び出して、IDを生成している。

 .. tip::

    上記例では、PostgreSQLのUUID生成関数として\ ``UUID_GENERATE_V4()``\を呼び出しているが、
    この関数は、`uuid-ossp <http://www.postgresql.org/docs/9.3/static/uuid-ossp.html>`_\と呼ばれるサブモジュールの関数である。

    この関数を使用したい場合は、uuid-osspモジュールを有効にする必要がある。

 .. tip::

    データベースIDは、OGNLベースの式（Expression言語）内でも参照する事ができる。

    これは、データベースIDを動的SQLの条件として使用できる事を意味している。
    以下に実装例を紹介する。

     .. code-block:: xml

        <select id="findAllByCreatedAtBefore" parameterType="_int" resultType="Todo">
            SELECT
                todo_id,
                todo_title,
                finished,
                created_at,
                version
            FROM
                t_todo
            WHERE
                <choose>
                    <!-- (2) -->
                    <when test="_databaseId == 'h2'">
                        <bind name="criteriaDate"
                              value="'DATEADD(\'DAY\',#{days} * -1,#{currentDate})'"/>
                    </when>
                    <when test="_databaseId == 'postgresql'">
                        <bind name="criteriaDate"
                              value="'#{currentDate}::DATE - (#{days} * INTERVAL \'1 DAY\')'"/>
                    </when>
                </choose>
                <![CDATA[
                    created_at < ${criteriaDate}
                ]]>
        </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (2)
      - OGNLベースの式（Expression言語）内では、
        \ ``_databaseId``\という特別な変数にデータベースIDが格納されている。

        上記例では、「システム日付 - 指定日」より前に作成されたレコードを抽出するための条件を、
        データベースの関数を利用して指定している。


|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnce:

関連Entityを１回のSQLで取得する方法について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

主Entityと関連Entityを1回のSQLでまとめて取得する方法について説明する。

主Entityと関連Entityをまとめて取得する仕組みを使用すると、
ServiceクラスでEntity(JavaBean)の組み立て処理を行う必要がなくなり、
Serviceクラスは業務ロジック(ビジネスルール)の実装に集中する事ができる。

| また、この方法は、N+1問題を回避する手段としても使用される。
| N+1問題については、「:ref:`data-access-common_howtosolve_n_plus_1`」を参照されたい。

 .. warning::

    主Entityと関連Entityをまとめて取得する場合は、以下の点に注意して使用すること。

    * 以下の説明では全ての関連Entityを1回のSQLでまとめて取得しているが、
      実際のプロジェクトで使用する場合は、
      処理で必要となる関連Entityのみ取得するようにした方がよいケースがある。
      使用しない関連Entityを同時に取得すると、
      無駄なオブジェクト生成やマッピング処理が行われるため性能劣化の要因となる事がある。
      **特に、一覧検索を行うSQLでは、必要な関連Entityのみ取得するようにした方がよいケースが多い。**

    \

    * 使用頻度の低い関連Entityについては、
      まとめて取得せず必要なときに個別に取得する方法を採用した方がよいケースがある。
      使用頻度の低い関連Entityを同時に取得すると、
      無駄なオブジェクト生成やマッピング処理が行われるため性能劣化の要因となる事がある。

    \

    * 1:Nの関係となる関連Entityが複数含まれる場合、
      主Entityと関連Entityを別々に取得する方法を採用した方がよいケースがある。
      1:Nの関係となる関連Entityが複数ある場合、
      無駄なデータをDBから取得する必要があるため、性能劣化の要因となる事がある。
      主Entityと関連Entityを別々に取得する方法の一例については、
      「:ref:`data-access-common_howtosolve_n_plus_1`」を参照されたい。

 .. tip::

    使用頻度の低い関連Entityを必要になった時に個別に取得する方法としては、

    * Serviceクラスの処理で関連Entityを取得するメソッド(SQL)を呼び出して取得する。
    * 関連Entityを"Lazy Load"対象にし、Getterメソッドが呼び出された際にSQLを透過的に実行して取得する。

    方法がある。

    "Lazy Load"の仕組みを使用すると、
    ServiceクラスでEntity(JavaBean)の組み立て処理を行う必要がなくなり、
    Serviceクラスは業務ロジック(ビジネスルール)の実装に集中する事ができる。

    **一覧検索を行うSQLで"Lazy Load"を使用するとN+1問題を引き起こすので、使用する際は注意すること。**

    "Lazy Load"の使用方法については、「:ref:`DataAccessMyBatis3AppendixNestedSelectLazySetting`」を参照されたい。


|

ここからは、ショッピングサイトで扱う注文データを、
1回のSQLでまとめて取得し、主Entity及び関連Entityにマッピングする実装例について説明を行う。

ここで説明する実装方法は、あくまで一例である。
MyBatis3では、本節で説明していない機能も多く提供しており、より高度なマッピングを行う事も可能である。

MyBatis3のマッピング機能の詳細については、
「\ `MyBatis3 REFERENCE DOCUMENTATION(Mapper XML Files-Result Maps-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#Result_Maps>`_ \」を参照されたい。

|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceTable:

テーブルレイアウトとデータ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

説明で使用するテーブルは、以下の通り。

 .. figure:: ../ImplementationAtEachLayer/images/service_entity_table_layout.png
    :alt: ER diagram
    :width: 100%
    :align: center

    **Picture - ER diagram**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|p{0.50\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 15 15 50

    * - 項番
      - カテゴリ
      - テーブル名
      - 説明
    * - (1)
      - トランザクション系
      - t_order
      - 注文データを保持するテーブル。

        １つの注文に対して、1レコードが格納される。
    * - (2)
      -
      - t_order_item
      - １つの注文で購入された商品データを保持するテーブル。

        1つの注文で複数の商品が購入された場合は、商品数分レコードが格納される。
    * - (3)
      -
      - t_order_coupon
      - １つの注文で使用されたクーポンのデータを保持するテーブル。

        1つの注文で、複数のクーポンが使用された場合は、クーポン数分レコードが格納される。
        クーポンを使用しなかった場合は、レコードは格納されない。
    * - (4)
      - マスタ系
      - m_item
      - 商品を定義するマスタテーブル。
    * - (5)
      -
      - m_category
      - 商品のカテゴリを定義するマスタテーブル。
    * - (6)
      -
      - m_item_category
      - 商品が所属するカテゴリを定義するマスタテーブル。

        商品とカテゴリのマッピングを保持している。
        1つの商品は、複数のカテゴリに属すことができるモデルとなっている。
    * - (7)
      -
      - m_coupon
      - クーポンを定義するマスタテーブル。
    * - (8)
      - コード系
      - c_order_status
      - 注文ステータスを定義するコードテーブル。

|

説明で使用するテーブルレイアウトと格納データを作成するためのSQL(DDLとDML)を以下に示す。
(SQLはH2 Database用である)

* マスタ系テーブル作成用のDDL

 .. code-block:: sql

    CREATE TABLE m_item (
        code CHAR(10),
        name NVARCHAR(256),
        price INTEGER,
        CONSTRAINT m_item_pk PRIMARY KEY(code)
    );

    CREATE TABLE m_category (
        code CHAR(10),
        name NVARCHAR(256),
        CONSTRAINT m_category_pk PRIMARY KEY(code)
    );

    CREATE TABLE m_item_category (
        item_code CHAR(10),
        category_code CHAR(10),
        CONSTRAINT m_item_category_pk PRIMARY KEY(item_code, category_code),
        CONSTRAINT m_item_category_fk1 FOREIGN KEY(item_code) REFERENCES m_item(code),
        CONSTRAINT m_item_category_fk2 FOREIGN KEY(category_code) REFERENCES m_category(code)
    );

    CREATE TABLE m_coupon (
        code CHAR(10),
        name NVARCHAR(256),
        price INTEGER,
        CONSTRAINT m_coupon_pk PRIMARY KEY(code)
    );

* コード系テーブル作成用のDDL

 .. code-block:: sql

    CREATE TABLE c_order_status (
        code VARCHAR(10),
        name NVARCHAR(256),
        CONSTRAINT c_order_status_pk PRIMARY KEY(code)
    );

* トランザクション系テーブル作成用のDDL

 .. code-block:: sql

    CREATE TABLE t_order (
        id INTEGER,
        status_code VARCHAR(10),
        CONSTRAINT t_order_pk PRIMARY KEY(id),
        CONSTRAINT t_order_fk FOREIGN KEY(status_code) REFERENCES c_order_status(code)
    );

    CREATE TABLE t_order_item (
        order_id INTEGER,
        item_code CHAR(10),
        quantity INTEGER,
        CONSTRAINT t_order_item_pk PRIMARY KEY(order_id, item_code),
        CONSTRAINT t_order_item_fk1 FOREIGN KEY(order_id) REFERENCES t_order(id),
        CONSTRAINT t_order_item_fk2 FOREIGN KEY(item_code) REFERENCES m_item(code)
    );

    CREATE TABLE t_order_coupon (
        order_id INTEGER,
        coupon_code CHAR(10),
        CONSTRAINT t_order_coupon_pk PRIMARY KEY(order_id, coupon_code),
        CONSTRAINT t_order_coupon_fk1 FOREIGN KEY(order_id) REFERENCES t_order(id),
        CONSTRAINT t_order_coupon_fk2 FOREIGN KEY(coupon_code) REFERENCES m_coupon(code)
    );

* データ投入用のDML

 .. code-block:: sql

    -- Setup master tables
    INSERT INTO m_item VALUES ('ITM0000001','Orange juice',100);
    INSERT INTO m_item VALUES ('ITM0000002','NotePC',100000);

    INSERT INTO m_category VALUES ('CTG0000001','Drink');
    INSERT INTO m_category VALUES ('CTG0000002','PC');
    INSERT INTO m_category VALUES ('CTG0000003','Hot selling');

    INSERT INTO m_item_category VALUES ('ITM0000001','CTG0000001');
    INSERT INTO m_item_category VALUES ('ITM0000002','CTG0000002');
    INSERT INTO m_item_category VALUES ('ITM0000002','CTG0000003');

    INSERT INTO m_coupon VALUES ('CPN0000001','Join coupon',3000);
    INSERT INTO m_coupon VALUES ('CPN0000002','PC coupon',30000);

    -- Setup code tables
    INSERT  INTO  c_order_status VALUES ('accepted','Order accepted');
    INSERT  INTO  c_order_status VALUES ('checking','Stock checking');
    INSERT  INTO  c_order_status VALUES ('shipped','Item Shipped');

    -- Setup transaction tables
    INSERT INTO t_order VALUES (1,'accepted');
    INSERT INTO t_order VALUES (2,'checking');

    INSERT INTO t_order_item VALUES (1,'ITM0000001',1);
    INSERT INTO t_order_item VALUES (1,'ITM0000002',2);
    INSERT INTO t_order_item VALUES (2,'ITM0000001',3);
    INSERT INTO t_order_item VALUES (2,'ITM0000002',4);

    INSERT INTO t_order_coupon VALUES (1,'CPN0000001');
    INSERT INTO t_order_coupon VALUES (1,'CPN0000002');

    COMMIT;

|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceEntity:

Entityのクラス図
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

実装例では、上記テーブルに格納されているレコードを、以下のEntity(JavaBean)にマッピングする。

 .. figure:: images/dataaccess_entity.png
    :alt: Class(JavaBean) diagram
    :width: 100%
    :align: center

    **Picture - Class(JavaBean) diagram**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.65\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 15 65

    * - 項番
      - クラス名
      - 説明
    * - (1)
      - Order
      - t_orderテーブルの1レコードを表現するJavaBean。

        関連Entityとして、\ ``OrderStatus``\を1件、\ ``OrderItem``\および\ ``OrderCoupon``\を複数保持する。

         .. code-block:: java

            public class Order implements Serializable {
                private static final long serialVersionUID = 1L;
                private int id;
                private OrderStatus orderStatus;
                List<OrderItem> orderItems;
                List<OrderCoupon> orderCoupons;
                // ...
            }

    * - (2)
      - OrderItem
      - t_order_itemテーブルの1レコードを表現するJavaBean。

        関連Entityとして、\ ``Item``\を保持する。

         .. code-block:: java

            public class OrderItem implements Serializable {
                private static final long serialVersionUID = 1L;
                private int orderId;
                private Item item;
                private int quantity;
                // ...
            }

    * - (3)
      - OrderCoupon
      - t_order_couponテーブルの1コードを表現するJavaBean。

        関連Entityとして、\ ``Coupon``\を保持する。

         .. code-block:: java

            public class OrderCoupon implements Serializable {
                private static final long serialVersionUID = 1L;
                private int orderId;
                private Coupon coupon;
                // ...
            }

    * - (4)
      - Item
      - m_itemテーブルの1コードを表現するJavaBean。

        関連オブジェクトとして、所属している\ ``Category``\を複数保持する。
        \ ``Category``\との紐づけは、m_item_categoryテーブルによって行われる。

         .. code-block:: java

            public class Item implements Serializable {
                private static final long serialVersionUID = 1L;
                private String code;
                private String name;
                private int price;
                private List<Category> categories;
                // ...
            }

    * - (5)
      - Category
      - m_categoryテーブルの1レコードを表現するJavaBean。

         .. code-block:: java

            public class Category implements Serializable {
                private static final long serialVersionUID = 1L;
                private String code;
                private String name;
                // ...
            }

    * - (6)
      - Coupon
      - m_couponテーブルの1レコードを表現するJavaBean。

         .. code-block:: java

            public class Coupon implements Serializable {
                private static final long serialVersionUID = 1L;
                private String code;
                private String name;
                private int price;
                // ...
            }

    * - (7)
      - OrderStatus
      - c_order_statusテーブルの1レコードを表現するJavaBean。

         .. code-block:: java

            public class OrderStatus implements Serializable {
                private static final long serialVersionUID = 1L;
                private String code;
                private String name;
                // ...
            }


|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceRepository:

Repositoryインタフェースの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

実装例では、

* Orderオブジェクトを1件取得するメソッド(\ ``findOne``\)
* 該当ページのOrderオブジェクトを取得するメソッド(\ ``findPage``\)

を実装する。

 .. code-block:: java

    package com.example.domain.repository.order;

    import com.example.domain.model.Order;

    import java.util.List;

    public interface OrderRepository {

        Order findOne(int id);

        List<Order> findPage(@Param("pageable") Pageable pageable);

    }

|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceSql:

SQLの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

関連Entityを1回のSQLでまとめて取得する場合は、
取得対象のテーブルをJOINしてマッピングに必要な全てのレコードを取得する。

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
    <mapper namespace="com.example.domain.repository.order.OrderRepository">

        <!-- (1) -->
        <sql id="selectFromJoin">
            SELECT
                /* (2) */
                o.id,
                /* (3) */
                o.status_code,
                os.name AS status_name,
                /* (4) */
                oi.quantity,
                i.code AS item_code,
                i.name AS item_name,
                i.price AS item_price,
                /* (5) */
                ct.code AS category_code,
                ct.name AS category_name,
                /* (6) */
                cp.code AS coupon_code,
                cp.name AS coupon_name,
                cp.price AS coupon_price
            FROM
                ${orderTable} o
            /* (7) */
            INNER JOIN c_order_status os ON os.code = o.status_code
            INNER JOIN t_order_item oi ON oi.order_id = o.id
            INNER JOIN m_item i ON i.code = oi.item_code
            INNER JOIN m_item_category ic ON ic.item_code = i.code
            INNER JOIN m_category ct ON ct.code = ic.category_code
            /* (8) */
            LEFT JOIN t_order_coupon oc ON oc.order_id = o.id
            LEFT JOIN m_coupon cp ON cp.code = oc.coupon_code
        </sql>

        <!-- (9) -->
        <select id="findOne" parameterType="_int" resultMap="orderResultMap">
            <bind name="orderTable" value="'t_order'" />
            <include refid="selectFromJoin"/>
            WHERE
                o.id = #{id}
            ORDER BY
                item_code ASC,
                category_code ASC,
                coupon_code ASC
        </select>

        <!-- (10) -->
        <select id="findPage" resultMap="orderResultMap">
            <bind name="orderTable" value="
                '(
                  SELECT
                      *
                  FROM
                      t_order
                  ORDER BY
                      id DESC
                  LIMIT #{pageable.pageSize}
                  OFFSET #{pageable.offset}
                  )'" />
            <include refid="selectFromJoin"/>
            ORDER BY
                id DESC,
                item_code ASC,
                category_code ASC,
                coupon_code ASC
        </select>

        <!-- ... -->

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - \ ``findOne``\メソッドと\ ``findPage``\メソッド用のSELECT句、FROM句、JOIN句を実装する。

        上記例では、\ ``findOne``\メソッドと\ ``findPage``\メソッドの共通箇所を共通化している。
    * - (2)
      - Orderオブジェクトを生成するために必要なデータを取得する。
    * - (3)
      - OrderStatusオブジェクトを生成するために必要なデータを取得する。

        取得するカラム名は重複しないようにする必要がある。
        上記例では、\ ``name``\カラムが重複するため、
        AS句を使用して別名(\ ``status_``\プレフィックス)を指定している。
    * - (4)
      - OrderItemオブジェクトとItemオブジェクトを生成するために必要なデータを取得する。

        取得するカラム名は重複しないようにする必要がある。
        上記例では、\ ``code``\,\ ``name``\, \ ``price``\が重複するため、
        AS句を使用して別名(\ ``item_``\プレフィックス)を指定している。
    * - (5)
      - Categoryオブジェクトを生成するために必要なデータを取得する。

        取得するカラム名は重複しないようにする必要がある。
        上記例では、\ ``code``\,\ ``name``\が重複するため、
        AS句を使用して別名(\ ``category_``\プレフィックス)を指定している。
    * - (6)
      - OrderCouponオブジェクトとCouponオブジェクトを生成するために必要なデータを取得する。

        取得するカラム名は重複しないようにする必要がある。
        上記例では、\ ``code``\,\ ``name``\, \ ``price``\が重複するため、
        AS句を使用して別名(\ ``coupon_``\プレフィックス)を指定している。
    * - (7)
      - 関連オブジェクトを生成するために必要なデータが格納されているテーブルを結合する。
    * - (8)
      - レコードが格納されない可能性のあるテーブルについては、外部結合とする。
        クーポンを使用しない場合、t_order_couponにレコードが格納されないので外部結合にする必要がある。
        t_order_couponと結合するt_couponも同様である。
    * - (9)
      - \ ``findOne``\メソッド用のSQLを実装する。

        ORDER BY句には、1:Nの関連をもつEntityの並び順を指定する。
        上記例では、PKの昇順で並べ替えている。
    * - (10)
      - \ ``findPage``\メソッド用のSQLを実装する。
        ORDER BY句には、Orderと1:Nの関連をもつEntityの並び順を指定する。
        上記例では、OrderはPKの降順(新しい順)、関連EntityはPKの昇順で並べ替えている。

 .. tip::

    1:Nの関連を持つ関連Entityを1回のSQLでまとめて取得する際にページネーション検索が必要な場合は、
    MyBatis3から提供されている\ ``RowBounds``\を使用することが出来ない。

    代替案としては、

    * まず主Entityのみを検索するメソッドを呼び出し、関連Entityは別途のメソッドを呼び出して取得する
    * SQLでページ範囲内の主Entityのみ格納されている仮想テーブルを作成し、仮想テーブルのレコードとJOINする事で、
      マッピングに必要な全てのレコードを取得する(上記例の \ ``findPage``\は、このパターンで実装している)

    等の方法が考えられる。

|

上記SQL(findPage)を実行すると以下のレコードが取得される。
注文レコードとしては2件だが、レコードが複数件格納される関連テーブルと結合しているため、
合計で9レコードが取得される。

内訳は、

* 1～3行目は、注文IDが\ ``2``\の\ ``Order``\オブジェクトを生成するためのレコード
* 4～9行目は、注文IDが\ ``1``\の\ ``Order``\オブジェクトを生成するためレコード

となる。

以降の説明では、注文IDが\ ``1``\のレコードを例に、
どのように検索結果(\ ``ResultSet``\)をJavaBeanにマッピングするかを説明していく。


 .. figure:: images/dataaccess_sql_result.png
    :alt: Result Set of findPage
    :width: 100%
    :align: center

    **Picture - Result Set of findPage**

|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceMapping:

マッピングの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

上記レコードを、\ ``Order``\オブジェクトと関連Entityにマッピングするための定義を以下に示す。

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
    <mapper namespace="com.example.domain.repository.order.OrderRepository">

        <!-- ... -->

        <!-- (1) -->
        <resultMap id="orderResultMap" type="Order">
            <id property="id" column="id"/>
            <!-- (2) -->
            <result property="orderStatus.code" column="status_code" />
            <result property="orderStatus.name" column="status_name" />
            <!-- (3) -->
            <collection property="orderItems" ofType="OrderItem">
                <id property="orderId" column="id"/>
                <id property="item.code" column="item_code"/>
                <result property="quantity" column="quantity"/>
                <association property="item" resultMap="itemResultMap"/>
            </collection>
            <!-- (4) -->
            <collection property="orderCoupons" ofType="OrderCoupon"
                        notNullColumn="coupon_code">
                <id property="orderId" column="id"/>
                <!-- (5) -->
                <id property="coupon.code" column="coupon_code"/>
                <result property="coupon.name" column="coupon_name"/>
                <result property="coupon.price" column="coupon_price"/>
            </collection>
        </resultMap>

        <!-- (6) -->
        <resultMap id="itemResultMap" type="Item">
            <id property="code" column="item_code"/>
            <result property="name" column="item_name"/>
            <result property="price" column="item_price"/>
            <!-- (7) -->
            <collection property="categories" ofType="Category">
                <id property="code" column="category_code"/>
                <result property="name" column="category_name"/>
            </collection>
        </resultMap>

    </mapper>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - 取得したレコードを\ ``Order``\オブジェクトにマッピングするための定義。
        関連Entity(\ ``OrderStatus``\, \ ``OrderItem``\,\ ``OrderCoupon``\)のマッピングを行う。
    * - (2)
      - 取得したレコードを\ ``OrderStatus``\オブジェクトにマッピングするための定義。
    * - (3)
      - 取得したレコードを\ ``OrderItem``\オブジェクトにマッピングするための定義。
        関連Entity(\ ``Item``\)へのマッピングは、別の\ ``resultMap``\(6)に委譲している。
    * - (4)
      - 取得したレコードを\ ``OrderCoupon``\オブジェクトにマッピングするための定義。
    * - (5)
      - 取得したレコードを\ ``Coupon``\オブジェクトにマッピングするための定義。
    * - (6)
      - 取得したレコードを\ ``Item``\オブジェクトにマッピングするための定義。
    * - (7)
      - 取得したレコードを\ ``Category``\オブジェクトにマッピングするための定義。

|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceMappingOrder:

Orderオブジェクトへのマッピングの実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

\ ``Order``\オブジェクトへのマッピングを行う。

 .. code-block:: xml
    :emphasize-lines: 1-4

    <!-- (1) -->
    <resultMap id="orderResultMap" type="Order">
        <!-- (2) -->
        <id property="id" column="id"/>
        <result property="orderStatus.code" column="status_code" />
        <result property="orderStatus.name" column="status_name" />
        <collection property="orderItems" ofType="OrderItem">
            <id property="orderId" column="id"/>
            <id property="item.code" column="item_code"/>
            <result property="quantity" column="quantity"/>
            <association property="item" resultMap="itemResultMap"/>
        </collection>
        <collection property="orderCoupons" ofType="OrderCoupon"
                    notNullColumn="coupon_code">
            <id property="orderId" column="id"/>
            <id property="coupon.code" column="coupon_code"/>
            <result property="coupon.name" column="coupon_name"/>
            <result property="coupon.price" column="coupon_price"/>
        </collection>
    </resultMap>

 .. figure:: images/dataaccess_resultmap_order.png
    :alt: ResultMap for Order
    :width: 100%
    :align: center

    **Picture - ResultMap for Order**


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - 検索結果を\ ``Order``\オブジェクトにマッピングする。

        \ ``type``\属性にマッピングするクラスを指定する。
    * - (2)
      - 取得したレコードの\ ``id``\カラムの値を、\ ``Order#id``\プロパティに設定する。

        \ ``id``\カラムはPKなので、\ ``id``\要素を使用してマッピングを行う。
        \ ``id``\要素を使用すると、指定したプロパティの値でレコードがグループ化される。
        具体的には、\ ``id=1``\と\ ``id=2``\の2つにグループ化され、
        2つの\ ``Order``\オブジェクトが生成される。


|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceMappingOrderStatus:

OrderStatusオブジェクトへのマッピングの実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

\ ``OrderStatus``\オブジェクトへのマッピングを行う。

 .. note::

    「:ref:`domainlayer_entity`」のEntityクラスの作成方針では、
    「コード系テーブルは、Entityとして扱うのではなく、java.lang.Stringなどの基本型で扱う。」としている。
    これは、コード系テーブルで保持しているデータは、
    「:doc:`Codelist`」などの別の仕組みを使用するケースが多いためである。

    本節では、関連Entity(JavaBean)へのマッピング方法を説明する事が目的なので、
    コード系テーブルもEntityとして扱っている点を補足しておく。

    実際のプロジェクトでは、Entityクラスの作成方針を参考にEntityを作成することを推奨する。


 .. code-block:: xml
    :emphasize-lines: 3-6

    <resultMap id="orderResultMap" type="Order">
        <id property="id" column="id"/>
        <!-- (1) -->
        <result property="orderStatus.code" column="status_code" />
        <!-- (2) -->
        <result property="orderStatus.name" column="status_name" />
        <collection property="orderItems" ofType="OrderItem">
            <id property="orderId" column="id"/>
            <id property="item.code" column="item_code"/>
            <result property="quantity" column="quantity"/>
            <association property="item" resultMap="itemResultMap"/>
        </collection>
        <collection property="orderCoupons" ofType="OrderCoupon"
                    notNullColumn="coupon_code">
            <id property="orderId" column="id"/>
            <id property="coupon.code" column="coupon_code"/>
            <result property="coupon.name" column="coupon_name"/>
            <result property="coupon.price" column="coupon_price"/>
        </collection>
    </resultMap>

 .. figure:: images/dataaccess_resultmap_orderstatus.png
    :alt: ResultMap for OrderStatus
    :width: 100%
    :align: center

    **Picture - ResultMap for OrderStatus**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - 取得したレコードの\ ``status_code``\カラムの値を、
        \ ``OrderStatus#code``\プロパティに設定する。
    * - (2)
      - 取得したレコードの\ ``status_name``\カラムの値を、
        \ ``OrderStatus#name``\プロパティに設定する。

 .. note::

    \ ``OrderStatus``\オブジェクトには、\ ``id``\カラムでグループ化されたレコードの値が設定される。

|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceMappingOrderItem:

OrderItemオブジェクトへのマッピングの実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

``OrderItem``\オブジェクトへのマッピングを行う。

 .. code-block:: xml
    :emphasize-lines: 5-15

    <resultMap id="orderResultMap" type="Order">
        <id property="id" column="id"/>
        <result property="orderStatus.code" column="status_code" />
        <result property="orderStatus.name" column="status_name" />
        <!-- (1) -->
        <collection property="orderItems" ofType="OrderItem">
            <!-- (2) -->
            <id property="orderId" column="id"/>
            <!-- (3) -->
            <id property="item.code" column="item_code"/>
            <!-- (4) -->
            <result property="quantity" column="quantity"/>
            <!-- (5) -->
            <association property="item" resultMap="itemResultMap"/>
        </collection>
        <collection property="orderCoupons" ofType="OrderCoupon"
                    notNullColumn="coupon_code">
            <id property="orderId" column="id"/>
            <id property="coupon.code" column="coupon_code"/>
            <result property="coupon.name" column="coupon_name"/>
            <result property="coupon.price" column="coupon_price"/>
        </collection>
    </resultMap>

 .. figure:: images/dataaccess_resultmap_orderitem.png
    :alt: ResultMap for OrderItem
    :width: 100%
    :align: center

    **Picture - ResultMap for OrderItem**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - 検索結果を\ ``OrderItem``\オブジェクトにマッピングし、
        \ ``Order#orderItems``\プロパティに追加する。

        1:Nの関係の関連Entityにマッピングする場合は、\ ``collection``\要素を使用する。
        \ ``collection``\要素の詳細は、
        「`MyBatis3 REFERENCE DOCUMENTATION(Mapper XML Files-collection-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#collection>`_\」を参照されたい。
    * - (2)
      - 取得したレコードの\ ``id``\カラムの値を、\ ``OrderItem#orderId``\プロパティに設定する。

        \ ``id``\カラムはPKなので、\ ``id``\要素を使用してマッピングを行う。
    * - (3)
      - 取得したレコードの\ ``item_code``\カラムの値を、\ ``Item#code``\プロパティに設定する。


        \ ``item_code``\カラムはPKなので、\ ``id``\要素を使用してマッピングを行う。
        \ ``id``\要素を使用すると、指定したプロパティの値でレコードがグループ化される。
        具体的には、\ ``Item#code=ITM0000001``\と\ ``Item#code=ITM0000002``\の2つにグループ化され、
        2つの\ ``OrderItem``\オブジェクトが生成される。
    * - (4)
      - 取得したレコードの\ ``quantity``\カラムの値を、\ ``OrderItem#quantity``\プロパティに設定する。
    * - (5)
      - \ ``Item``\オブジェクトの生成を、別の\ ``resultMap``\に委譲し、
        生成されたオブジェクトを\ ``OrderItem#item``\プロパティに設定する。
        実際のマッピングは、
        「:ref:`DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceMappingItem`」を参照されたい。

        1:1の関係の関連Entityにマッピングする場合は、\ ``association``\要素を使用する。
        \ ``association``\要素の詳細は、
        「`MyBatis3 REFERENCE DOCUMENTATION(Mapper XML Files-association-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#association>`_\」を参照されたい。

 .. note::

    \ ``OrderItem``\オブジェクトには、
    \ ``id``\カラムと\ ``item_code``\カラムでグループ化されたレコードの値が設定される。

|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceMappingItem:

Itemオブジェクトへのマッピングの実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

\ ``Item``\オブジェクトへのマッピングを行う。

 .. code-block:: xml
    :emphasize-lines: 1-8

    <!-- (1) -->
    <resultMap id="itemResultMap" type="Item">
        <!-- (2) -->
        <id property="code" column="item_code"/>
        <!-- (3) -->
        <result property="name" column="item_name"/>
        <!-- (4) -->
        <result property="price" column="item_price"/>
        <collection property="categories" ofType="Category">
            <id property="code" column="category_code"/>
            <result property="name" column="category_name"/>
        </collection>
    </resultMap>

 .. figure:: images/dataaccess_resultmap_item.png
    :alt: ResultMap for Item
    :width: 100%
    :align: center

    **Picture - ResultMap for Item**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - 検索結果を\ ``Item``\オブジェクトにマッピングする。

        \ ``type``\属性にマッピングするクラスを指定する。
    * - (2)
      - 取得したレコードの\ ``item_code``\カラムの値を、\ ``Item#code``\に設定する。

        \ ``item_code``\カラムはPKなので、\ ``id``\要素を使用してマッピングを行う。
    * - (3)
      - 取得したレコードの\ ``item_name``\カラムの値を、\ ``Item#name``\に設定する。
    * - (4)
      - 取得したレコードの\ ``item_price``\カラムの値を、\ ``Item#price``\に設定する。

 .. note::

    \ ``Item``\オブジェクトには、
    \ ``id``\カラムと\ ``item_code``\カラムでグループ化されたレコードの値が設定される。

|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceMappingCategory:

Categoryオブジェクトへのマッピングの実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

\ ``Category``\オブジェクトへのマッピングを行う。

 .. code-block:: xml
    :emphasize-lines: 5-11

    <resultMap id="itemResultMap" type="Item">
        <id property="code" column="item_code"/>
        <result property="name" column="item_name"/>
        <result property="price" column="item_price"/>
        <!-- (1) -->
        <collection property="categories" ofType="Category">
            <!-- (2) -->
            <id property="code" column="category_code"/>
            <!-- (3) -->
            <result property="name" column="category_name"/>
        </collection>
    </resultMap>


 .. figure:: images/dataaccess_resultmap_category.png
    :alt: ResultMap for Category
    :width: 100%
    :align: center

    **Picture - ResultMap for Category**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - 検索結果を\ ``Category``\オブジェクトにマッピングし、\ ``Item#categories``\プロパティに追加する。

        1:Nの関係の関連Entityにマッピングする場合は、\ ``collection``\要素を使用する。
        \ ``collection``\要素の詳細は、
        「`MyBatis3 REFERENCE DOCUMENTATION(Mapper XML Files-collection-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#collection>`_\」を参照されたい。
    * - (2)
      - 取得したレコードの\ ``category_code``\カラムの値を、\ ``Category#code``\に設定する。

        \ ``category_code``\カラムはPKなので、\ ``id``\要素を使用してマッピングを行う。
        \ ``id``\要素を使用すると、指定したプロパティの値でレコードがグループ化される。

        具体的には、

        * \ ``Item#code=ITM0000001``\のカテゴリとして、\ ``Category#code=CTG0000001``\の\ ``Category``\オブジェクト

        * \ ``Item#code=ITM0000002``\のカテゴリとして、\ ``Category#code=CTG0000002``\と\ ``Category#code=CTG0000003``\の2つの\ ``Category``\オブジェクト

        が生成される。
    * - (3)
      - 取得したレコードの\ ``category_name``\カラムの値を、\ ``Category#name`` \に設定する。

 .. note::

    \ ``Category``\オブジェクトには、
    \ ``id``\カラムと\ ``item_code``\カラムと\ ``category_code``\カラムでグループ化されたレコードの値が設定される。

|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceMappingOrderCoupon:

OrderCouponオブジェクトへのマッピングの実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

\ ``OrderCoupon``\オブジェクトへのマッピングを行う。

 .. code-block:: xml
    :emphasize-lines: 11-16

    <resultMap id="orderResultMap" type="Order">
        <id property="id" column="id"/>
        <result property="orderStatus.code" column="status_code" />
        <result property="orderStatus.name" column="status_name" />
        <collection property="orderItems" ofType="OrderItem">
            <id property="orderId" column="id"/>
            <id property="item.code" column="item_code"/>
            <result property="quantity" column="quantity"/>
            <association property="item" resultMap="itemResultMap"/>
        </collection>
        <!-- (1) -->
        <collection property="orderCoupons" ofType="OrderCoupon" notNullColumn="coupon_code">
            <!-- (2) -->
            <id property="orderId" column="id"/>
            <!-- (3) -->
            <id property="coupon.code" column="coupon_code"/>
            <result property="coupon.name" column="coupon_name"/>
            <result property="coupon.price" column="coupon_price"/>
        </collection>
    </resultMap>

 .. figure:: images/dataaccess_resultmap_ordercoupon.png
    :alt: ResultMap for OrderCoupon
    :width: 100%
    :align: center

    **Picture - ResultMap for OrderCoupon**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - 検索結果を\ ``OrderCoupon``\オブジェクトにマッピングし、\ ``Order#orderCoupons``\プロパティに追加する。

        1:Nの関係の関連Entityにマッピングする場合は、\ ``collection``\要素を使用する。
        \ ``collection``\要素の詳細は、
        「`MyBatis3 REFERENCE DOCUMENTATION(Mapper XML Files-collection-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#collection>`_\」を参照されたい。

        上記例にて、\ ``notNullColumn``\属性を指定している点に注目してほしい。

        これは\ ``t_coupon``\テーブルにレコードが存在しない時に、
        \ ``OrderCoupon``\オブジェクトを生成させないための設定である。
        本実装例では、\ ``id``\カラムが\ ``2``\のデータには\ ``t_coupon``\テーブルのレコードを格納していないため、
        検索結果をみると、\ ``coupon_code``\と\ ``coupon_name``\と\ ``coupon_price``\の値が\ ``null``\になっているのがわかる。

        \ ``OrderCoupon``\オブジェクトにマッピングするカラムがこの3つだけであれば、
        \ ``notNullColumn``\属性を指定する必要はないが、
        実装例では\ ``id``\カラムの値を\ ``OrderCoupon#orderId``\プロパティにマッピングする設定を行っているため、
        \ ``notNullColumn``\属性の指定が必要となる。
        これは、マッピング対象のカラムの中に\ ``null``\でない値がセットされていた場合に、
        MyBatisがオブジェクトを生成するためである。

        上記例のように、\ ``notNullColumn``\属性に\ ``coupon_code``\カラムを指定しておくと、
        \ ``coupon_code``\カラムが\ ``null``\でない場合(つまり、レコードが存在する場合)にのみ、
        オブジェクトが生成される。
        \ ``notNullColumn``\属性には、複数のカラムを指定する事もできる。
    * - (2)
      - 取得したレコードの\ ``id``\カラムの値を、\ ``OrderCoupon#orderId``\プロパティに設定する。

        \ ``orderId``\はPKなので、\ ``id``\要素を使用する。
    * - (3)
      - 取得したレコードの\ ``coupon_code``\カラムの値を\ ``Coupon#code``\に設定する。

        \ ``coupon_code``\カラムはPKなので、\ ``id``\要素を使用してマッピングを行う。
        \ ``id``\要素を使用すると、指定したプロパティの値でレコードがグループ化される。

        具体的には、\ ``Coupon#code=CPN0000001``\と\ ``Coupon#code=CPN0000002``\の2つにグループ化され、
        2つの\ ``OrderCoupon``\オブジェクトが生成される。

|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceMappingCoupon:

Couponオブジェクトへのマッピングの実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''


\ ``Coupon``\オブジェクトへのマッピングを行う。

 .. code-block:: xml
    :emphasize-lines: 13-18

    <resultMap id="orderResultMap" type="Order">
        <id property="id" column="id"/>
        <result property="orderStatus.code" column="status_code" />
        <result property="orderStatus.name" column="status_name" />
        <collection property="orderItems" ofType="OrderItem">
            <id property="orderId" column="id"/>
            <id property="item.code" column="item_code"/>
            <result property="quantity" column="quantity"/>
            <association property="item" resultMap="itemResultMap"/>
        </collection>
        <collection property="orderCoupons" ofType="OrderCoupon" notNullColumn="coupon_code">
            <id property="orderId" column="id"/>
            <!-- (1) -->
            <id property="coupon.code" column="coupon_code"/>
            <!-- (2) -->
            <result property="coupon.name" column="coupon_name"/>
            <!-- (3) -->
            <result property="coupon.price" column="coupon_price"/>
        </collection>
    </resultMap>


 .. figure:: images/dataaccess_resultmap_coupon.png
    :alt: ResultMap for Coupon
    :width: 100%
    :align: center

    **Picture - ResultMap for Coupon**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - 取得したレコードの\ ``coupon_code``\カラムの値を、\ ``Coupon#code``\に設定する。
    * - (2)
      - 取得したレコードの\ ``coupon_name``\カラムの値を、\ ``Coupon#name``\に設定する。
    * - (3)
      - 取得したレコードの\ ``coupon_price``\ カラムの値を、\ ``Coupon#price``\に設定する。

 .. note::

    \ ``Coupon``\オブジェクトには、
    \ ``id``\カラムと\ ``coupon_code``\カラムでグループ化されたレコードの値が設定される。


|

.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsAtOnceObject:

マッピング後のオブジェクト図
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

実際にマッピングされた\ ``Order``\オブジェクトおよび関連Entityの状態は、以下の通りである。

 .. figure:: images/dataaccess_object.png
    :alt: Mapped object diagram
    :width: 90%
    :align: center

    **Picture - Mapped object diagram**


| \ ``Order``\オブジェクトにマッピングされたレコードとカラムは、以下の通りである。
| グレーアウトしている部分は、グループ化によって、グレーアウトされていない部分にマージされる。

 .. figure:: images/dataaccess_sql_result_used.png
    :alt: Valid Result Set
    :width: 100%
    :align: center

    **Picture - Valid Result Set**


.. _DataAccessMyBatis3AppendixAcquireRelatedObjectsWarningSqlMapping:

 .. warning::

     1:Nの関連をもつレコードをJOINしてマッピングする場合、グレーアウトされている部分のデータの取得が無駄になる点を、意識しておくこと。

     Nの部分のデータを使用しない処理で、同じSQLを使用した場合、さらに無駄なデータの取得となってしまうので、Nの部分を取得するSQLと、取得しないSQLを、別々に用意しておくなどの工夫を行うこと。

|

.. _DataAccessMyBatis3AppendixNestedSelect:

関連EntityをネストしたSQLを使用して取得する方法について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

MyBatis3では、マッピング時に別のSQL(ネストしたSQL)を使用して関連Entityを取得する方法を提供している。

ネストしたSQLを使用して関連Entityを取得する仕組みを使用すると、

* 個々のSQL定義
* \ ``resultMap``\要素のマッピング定義

をシンプルにする事ができる。

 .. warning::

     各種定義がシンプルになる一方で、ネストしたSQLを多用すると、
     N+1問題を引き起こす要因になるという事を意識する必要がある。

     ネストしたSQLを使用する場合のMyBatisのデフォルトの動作は、"Eager Load"となる。
     これは、関連Entityの使用有無に関係なくSQLが発行される事を意味しており、

     * 無駄なSQLの実行とデータの取得
     * N+1問題

     などが発生する危険性が高まる。

 .. tip::

    MyBatis3では、ネストしたSQLを使用して関連Entityを取得する際の動作を、
    "Lazy Load"に変更するためのオプションを提供している。

    "Lazy Load"の使用方法については、「:ref:`DataAccessMyBatis3AppendixNestedSelectLazySetting`」を参照されたい。

|

.. _DataAccessMyBatis3AppendixNestedSelectMapping:

関連EntityをネストしたSQLを使用して取得する実装例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ネストしたSQLを使用して関連Entityを取得する際の実装例を以下に示す。

 .. code-block:: xml
    :emphasize-lines: 5-7

    <resultMap id="itemResultMap" type="Item">
        <id property="code" column="item_code"/>
        <result property="name" column="item_name"/>
        <result property="price" column="item_price"/>
        <!-- (1) -->
        <collection property="categories" column="item_code"
            select="findAllCategoryByItemCode" />
    </resultMap>

    <select id="findAllCategoryByItemCode"
        parameterType="string" resultType="Category">
        SELECT
            ct.code,
            ct.name
        FROM
            m_item_category ic
        INNER JOIN m_category ct ON ct.code = ic.category_code
        WHERE
            ic.item_code = #{itemCode}
        ORDER BY
            code
    </select>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - \ ``association``\要素又は\ ``collection``\要素の\ ``select``\属性に、
        呼び出すSQLのステートメントIDを指定する。

        \ ``column``\属性には、SQLに渡すパラメータ値が格納されているカラム名を指定する。
        上記例では、
        \ ``findAllCategoryByItemCode``\のパラメータとして\ ``item_code``\カラムの値を渡している。

        指定可能な属性の詳細は、
        「`MyBatis3 REFERENCE DOCUMENTATION(Mapper XML Files-Nested Select for Association-) <http://mybatis.github.io/mybatis-3/sqlmap-xml.html#Nested_Select_for_Association>`_\」を参照されたい。

 .. note::

    上記例では、\ ``fetchType``\属性を指定していないため、
    "Lazy Load"と"Eager Load"のどちらで実行されるかは、
    アプリケーション全体の設定に依存する。

    アプリケーション全体の設定については、
    「:ref:`DataAccessMyBatis3AppendixNestedSelectLazySettingMyBatis`」を参照されたい。

|

.. _DataAccessMyBatis3AppendixNestedSelectLazySetting:

関連EntityをLazy Loadするための設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ネストしたSQLを使用して関連Entityを取得する際のMyBatis3のデフォルト動作は、
"Eager Load"であるが、"Lazy Load"を使用する事も可能である。

以下に、"Lazy Load"を使用するために最低限必要な設定及び使用方法について説明を行う。

説明していない設定値については、
「`MyBatis3 REFERENCE DOCUMENTATION(Mapper XML Files-settings-) <http://mybatis.github.io/mybatis-3/configuration.html#settings>`_\」を参照されたい。

|

.. _DataAccessMyBatis3AppendixNestedSelectLazySettingLibrary:

バイトコード操作ライブラリの追加
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

"Lazy Load"を使用する場合は、"Lazy Load"を実現するためのProxyオブジェクトを生成するために、

* JAVASSIST
* CGLIB

のいずれか一方のライブラリが必要となる。

MyBatis 3.2系まではCGLIBがデフォルトで使用されるライブラリであったが、
terasoluna-gfw-mybatis3 5.0.2.RELEASEでサポートしたMyBatis 3.3.0以降のバージョンではJAVASSISTがデフォルトで使用される。
さらに、MyBatis 3.3.0からJAVASSISTがMyBatis本体に内包されているため、ライブラリを追加しなくても"Lazy Load"を使用する事ができる。

 .. note::

    MyBatis 3.3.0以降のバージョンでCGLIBを使用する場合は、

    * \ :file:`pom.xml`\にCGLIBのアーティファクトを追加
    * MyBatis設定ファイル(:file:`projectName-domain/src/main/resources/META-INF/mybatis/mybatis-config.xml`)に「\ ``proxyFactory=CGLIB``\」を追加

    すればよい。

    CGLIBのアーティファクト情報については、
    「`MyBatis3 PROJECT DOCUMENTATION(Project Dependencies-compile-) <http://mybatis.github.io/mybatis-3/dependencies.html#compile>`_\」を参照されたい。

|

.. _DataAccessMyBatis3AppendixNestedSelectLazySettingMyBatis:

Lazy Loadを使用するためのMyBatisの設定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

MyBatis3では、"Lazy Load"の使用有無を、

* アプリケーションの全体設定(MyBatis設定ファイル)
* 個別設定(マッピングファイル)

の2箇所で指定する事ができる。

* アプリケーションの全体設定は、
  MyBatis設定ファイル(:file:`projectName-domain/src/main/resources/META-INF/mybatis/mybatis-config.xml`)に指定する。

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
            PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
        <settings>
            <!-- (1) -->
            <setting name="lazyLoadingEnabled" value="true"/>
        </settings>
    </configuration>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - アプリケーションのデフォルト動作を\ ``lazyLoadingEnabled``\に指定する。

        * \ ``true``\ : "Lazy Load"
        * \ ``false``\ : "Eager Load" (デフォルト)

        \ ``association``\要素と\ ``collection``\要素の\ ``fetchType``\属性を指定した場合は、
        \ ``fetchType``\属性の指定値が優先される。

 .. warning::

    「\ ``false``\ : "Eager Load"」の状態で\ ``association``\要素又は\ ``collection``\要素の\ ``select``\属性を使用すると、
    マッピング時にSQLが実行されるので、注意が必要である。

    特に理由がない場合は、\ ``lazyLoadingEnabled``\は\ ``true``\にする事を推奨する。

|

* 個別設定は、
  マッピングファイルの\ ``association``\要素と\ ``collection``\要素の\ ``fetchType``\属性で指定する。

 .. code-block:: xml
    :emphasize-lines: 7

        <resultMap id="itemResultMap" type="Item">
            <id property="code" column="item_code"/>
            <result property="name" column="item_name"/>
            <result property="price" column="item_price"/>
            <!-- (2) -->
            <collection property="categories" column="item_code"
                fetchType="lazy"
                select="findAllCategoryByItemCode" />
        </resultMap>

        <select id="findAllCategoryByItemCode"
            parameterType="string" resultType="Category">
            SELECT
                ct.code,
                ct.name
            FROM
                m_item_category ic
            INNER JOIN m_category ct ON ct.code = ic.category_code
            WHERE
                ic.item_code = #{itemCode}
            ORDER BY
                code
        </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (2)
      - \ ``association``\要素又は\ ``collection``\要素の\ ``fetchType``\属性に、
        \ ``lazy``\又は\ ``eager``\を指定する。

        \ ``fetchType``\属性を指定すると、アプリケーション全体の設定を上書きする事ができる。

|

.. _DataAccessMyBatis3AppendixNestedSelectLazySettingTiming:

Lazy Loadの実行タイミングを制御するための設定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

MyBatis3では、"Lazy Load"を実行するタイミングを制御するためのオプションを提供している。

"Lazy Load"を実行するタイミングを制御するための設定は、
MyBatis設定ファイル(:file:`projectName-domain/src/main/resources/META-INF/mybatis/mybatis-config.xml`)に指定する。

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
            PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
        <settings>
            <!-- (1) -->
            <setting name="aggressiveLazyLoading" value="false"/>
        </settings>
    </configuration>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - "Lazy Load"を実行するタイミングを\ ``aggressiveLazyLoading``\に指定する。

        * \ ``true``\ : "Lazy Load"対象となっているプロパティを保持するオブジェクトのgetterメソッドが呼び出されたタイミングで実行する（デフォルト）
        * \ ``false``\ : "Lazy Load"対象となっているプロパティのgetterメソッドが呼び出されたタイミングで実行する


 .. warning::

    「\ ``true``\」（デフォルト）の場合、使用されないデータを取得するためにSQLが実行される可能性があるので、注意が必要である。

    具体的には、以下のようなマッピングを行い、
    "Lazy Load"対象になっていないプロパティだけにアクセスするケースである。
    「\ ``true``\」（デフォルト）の場合、"Lazy Load"対象のプロパティに対して直接アクセスしなくても、
    "Lazy Load"が実行されてしまう。

    特に理由がない場合は、\ ``aggressiveLazyLoading``\は\ ``false``\にする事を推奨する。

    * Entity

      .. code-block:: java

        public class Item implements Serializable {
            private static final long serialVersionUID = 1L;
            private String code;
            private String name;
            private int price;
            private List<Category> categories;
            // ...
        }

    * マッピングファイル

      .. code-block:: xml

        <resultMap id="itemResultMap" type="Item">
            <id property="code" column="item_code"/>
            <result property="name" column="item_name"/>
            <result property="price" column="item_price"/>
            <collection property="categories" column="item_code"
                fetchType="lazy" select="findByItemCode" />
        </resultMap>

    * アプリケーションコード(Service)

      .. code-block:: java
        :emphasize-lines: 2-3

            Item item = itemRepository.findOne(itemCode);
            // (2)
            String code = item.getCode();
            String name = item.getName();
            String price = item.getPrice();
            // ...
        }

      .. tabularcolumns:: |p{0.15\linewidth}|p{0.75\linewidth}|
      .. list-table::
        :header-rows: 1
        :widths: 15 75

        * - 項番
          - 説明
        * - (2)
          - 上記例では、"Lazy Load"対象のプロパティである\ ``categories``\プロパティにアクセスしていないが、
            \ ``Item#code``\プロパティにアクセスした際に、"Lazy Load"が実行される。

            「\ ``false``\」（デフォルト）の場合、上記のケースでは"Lazy Load"は実行されない。


.. raw:: latex

   \newpage

