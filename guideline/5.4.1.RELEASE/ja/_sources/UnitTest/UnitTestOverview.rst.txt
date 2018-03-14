単体テスト概要
================================================================================

.. only:: html

 .. contents:: 目次
    :local:

はじめに
--------------------------------------------------------------------------------

本章では、TERASOLUNA Server Framework (5.x)を使用したシステムにおける、JUnitを用いた単体テストについて提示する。

ここでの単体テストのスコープはレイヤまたはレイヤ間結合とし、テストに関するアクティビティのうち、テスト実装と
テスト実施に関して解説する。
なお、解説するテスト実装は参考例でありシステムの品質を保証するためのテスト方針については、別途検討いただきたい。

単体テストガイドラインが示すこと
--------------------------------------------------------------------------------

本章では、以下のOSSライブラリを使用したレイヤまたはレイヤ間のテスト実装方法について説明する。

単体テストで利用するOSSライブラリ構成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
テスティングフレームワーク
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Javaのテスティングフレームワークとして、\ `JUnit <http://www.junit.org/>`_\ を使用する。

アサーション
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

アサーションに使用するライブラリとして\ `Hamcrest <http://hamcrest.org/JavaHamcrest/>`_\ を使用する。
JUnit4が標準でサポートしているアサーションライブラリである。

モック化
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

テスト対象のメソッドが依存するクラスをモック化するためのライブラリとして\ `Mockito <http://mockito.org/>`_\
を使用する。

DIコンテナ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

テスト用のDIコンテナとして
\ `Spring TestのDI機能 <https://docs.spring.io/spring/docs/4.3.14.RELEASE/spring-framework-reference/html/integration-testing.html#testing-fixture-di>`_\を使用する。

MVCフレームワーク
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

テスト用のMVCフレームワークとして
\ `Spring MVC Test Framework <https://docs.spring.io/spring/docs/4.3.14.RELEASE/spring-framework-reference/html/integration-testing.html#spring-mvc-test-framework>`_\を使用する。


トランザクション管理
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

テスト用のトランザクション管理として
\ `Spring Testのトランザクション管理機能 <https://docs.spring.io/spring/docs/4.3.14.RELEASE/spring-framework-reference/html/integration-testing.html#testing-tx>`_\を使用する。


データアクセス
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

テスト用のデータアクセスとして、Spring TestまたはDBUnitとSpring Test DBUnitを使用することを想定している。

* \ `Spring Test <https://docs.spring.io/spring/docs/4.3.14.RELEASE/spring-framework-reference/html/testing-introduction.html>`_\

  * Spring Testは\ ``@Sql``\ アノテーションや\ ``JdbcTemplate``\ などを使用してSQLを発行する機能を提供している。

* \ `DBUnit <http://dbunit.sourceforge.net/>`_\  と\ `Spring Test DBUnit <https://springtestdbunit.github.io/spring-test-dbunit/>`_\

  * Spring Test DBUnitは、Spring Framework上でDBUnitを利用する際の支援ライブラリのため、DBUnitと組み合わせて使用する。
    DBUnitの提供するデータベースのセットアップ、状態の検証などの機能をアノテーションベースで実装する機能を提供している。

単体テストで利用するOSSライブラリのバージョン
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

単体テストで利用するOSSライブラリの一覧を以下に示す。

なお、以下のOSSライブラリはあくまで一例であり、実際は業務要件に合わせたライブラリを検討いただきたい。
また、本章内のアプリケーション自体を動作させるために利用するOSSライブラリ一覧については、
\ :ref:`frameworkstack_using_oss_version`\を参照されたい。

.. tip::

    version 5.0.0.RELEASEより、\ `Spring IO platform <http://platform.spring.io/platform/>`_\ の
    \ ``<dependencyManagement>``\ をインポートする構成を採用している。

    Spring IO platformの\ ``<dependencyManagement>``\ をインポートすることで、

    * Spring Frameworkが提供しているライブラリ
    * Spring Frameworkが依存しているOSSライブラリ
    * Spring Frameworkと相性のよいOSSライブラリ

    への依存関係を解決しており、TERASOLUNA Server Framework (5.x)で使用するOSSのバージョンは、原則として、
    Spring IO platformの定義に準じている。

    なお、version 5.4.1.RELEASEで指定しているSpring IO platformのバージョンは、
    \ `Brussels-SR5 <https://docs.spring.io/platform/docs/Brussels-SR5/reference/htmlsingle/>`_\ である。

|

以下のOSSライブラリの中で、特にSpring Test（MockMvc）、Mockitoの使い方については、\ :ref:`UsageOfLibraryForTest`\
で詳細を説明する。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.25\linewidth}|p{0.20\linewidth}|p{0.15\linewidth}|p{0.20\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 25 20 15 20

    * - Type
      - GroupId
      - ArtifactId
      - Version
      - Spring IO platform
    * - JUnit
      - junit
      - junit
      - 4.12
      - \*
    * - Hamcrest
      - org.hamcrest
      - hamcrest-core
      - 1.3
      - \*
    * - Hamcrest
      - org.hamcrest
      - hamcrest-library
      - 1.3
      - \*
    * - Mockito
      - org.mockito
      - mockito-core
      - 1.10.19
      - \*
    * - Spring Test
      - org.springframework
      - spring-test
      - 4.3.14.RELEASE
      -
    * - DBUnit
      - org.dbunit
      - dbunit
      - 2.5.4
      - \
    * - Spring Test DBUnit
      - com.github.springtestdbunit
      - spring-test-dbunit
      - 1.3.0
      - \

|

単体テストの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

単体テストは\ :ref:`ApplicationLayering`\ に沿った以下のレイヤ単位で実装している。レイヤまたはレイヤ間のテスト方法を
\ :ref:`ImplementsOfTestByLayer`\ で説明する。
レイヤ単位に当てはめられない共通機能や、機能特有のテスト方法は、\ :ref:`ImplementsOfTestByFunction`\ で説明する。

.. figure:: ./images/UnitTestOverviewApplicationLayer.png
   :width: 85%

|

対象読者
--------------------------------------------------------------------------------

本章は、\ :ref:`TargetReadersOfThisDocument`\ に加えて以下の知識・経験があることを前提としている。

* JUnitを使用した単体テストを行ったことがある

単体テストの動作検証環境
--------------------------------------------------------------------------------

本章は、以下の環境で動作検証をしている。
他の環境で実施する際は、本章をベースに適宜読み替えること。

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 25 75

    * - 種別
      - 名前
    * - OS
      - Windows 7
    * - JVM
      - `Java <http://www.oracle.com/technetwork/java/javase/downloads/index.html>`_ 1.8.0_144
    * - IDE
      - `Spring Tool Suite <http://spring.io/tools/sts/all>`_ 3.9.1.RELEASE (以降「STS」と呼ぶ)
    * - Build Tool
      - `Apache Maven <http://maven.apache.org/download.cgi>`_ 3.3.9 (以降「Maven」と呼ぶ)
    * - RDBMS
      - `PostgreSQL <http://www.postgresql.org/docs/9.6/static/sql-insert.html>`_ 9.6.5

