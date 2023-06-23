TERASOLUNA Server Framework for Java (5.x)のスタック
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

TERASOLUNA Server Framework for Java (5.x)のSoftware Framework概要
--------------------------------------------------------------------------------

TERASOLUNA Server Framework for Java (5.x)で使用するSoftware Frameworkは独自のフレームワークではなく、\ `Spring Framework <http://projects.spring.io/spring-framework/>`_\ を中心としたOSSの組み合わせである。

.. figure:: images/introduction-software-framework.png
   :width: 95%


Software Frameworkの主な構成要素
--------------------------------------------------------------------------------

TERASOLUNA Server Framework for Java (5.x)を構成するライブラリを以下に示す。

.. figure:: images/introduction-software-stack.png
   :width: 95%

DIコンテナ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
DIコンテナとしてSpring Frameworkを利用する。


* `Spring Framework 4.1 <http://docs.spring.io/spring/docs/4.1.7.RELEASE/spring-framework-reference/html/beans.html>`_

MVCフレームワーク
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Web MVCフレームワークとしてSpring MVCを利用する。

* `Spring MVC 4.1 <http://docs.spring.io/spring/docs/4.1.7.RELEASE/spring-framework-reference/html/mvc.html>`_

O/R Mapper
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

本ガイドラインでは、以下の\ **いずれか**\ を想定している。

* `MyBatis 3.2 <http://mybatis.github.io/mybatis-3/>`_

  * Spring Frameworkとの連携ライブラリとして、\ `MyBatis-Spring <http://mybatis.github.io/spring/>`_\ を使用する。

* `JPA2.1 <http://download.oracle.com/otn-pub/jcp/persistence-2_1-fr-eval-spec/JavaPersistence.pdf>`_

  * プロバイダは、\ `Hibernate 4.3 <http://docs.jboss.org/hibernate/orm/4.3/manual/en-US/html_single/>`_\ を使用する。

.. note::

  MyBatisは正確には「SQL Mapper」であるが、本ガイドラインでは「O/R Mapper」に分類する。

.. warning::

  どんなプロジェクトでもJPAを採用できるわけではない。"テーブルがほとんど正規化されいない"、"テーブルのカラム数が多すぎる"というテーブル設計がされている場合に、JPAの利用は難しい。

  また、本ガイドラインではJPAの基本的な説明は行っておらず、JPA利用経験者がチーム内にいることが前提である。

View
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
ViewにはJSPを利用する。

Viewのレイアウトを共通化する場合は、

* `Apache Tiles 3.0 <http://tiles.apache.org/framework/index.html>`_

を利用する。

セキュリティ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
認証・認可のフレームワークとしてSpring Securityを利用する。

* `Spring Security 3.2 <http://projects.spring.io/spring-security/>`_

.. tip::

    Spring Security 3.2 から、認証・認可の仕組みの提供に加えて、
    悪意のある攻撃者からWebアプリケーションを守るための仕組みが強化されている。

    悪意のある攻撃者からWebアプリケーションを守るための仕組みについては、

    * :doc:`../Security/CSRF`
    * :ref:`SpringSecurityAppendixSecHeaders`

    を参照されたい。

バリデーション
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* 単項目チェックには\ `BeanValidation 1.1 <http://download.oracle.com/otn-pub/jcp/bean_validation-1_1-fr-eval-spec/bean-validation-specification.pdf>`_\ を利用する。

  * 実装は、\ `Hibernate Validator 5.1 <http://docs.jboss.org/hibernate/validator/5.1/reference/en-US/html/>`_\ を利用する。

* 相関チェックには\ `Bean Validation <http://download.oracle.com/otn-pub/jcp/bean_validation-1_1-fr-eval-spec/bean-validation-specification.pdf>`_\ 、もしくは\ `Spring Validation <http://docs.spring.io/spring/docs/4.1.7.RELEASE/spring-framework-reference/html/validation.html#validator>`_\ を利用する。

  * 使い分けについては\ :doc:`../ArchitectureInDetail/Validation`\ を参照されたい。



ロギング
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* ロガーのAPIは\ `SLF4J <http://www.slf4j.org>`_\ を使用する。

  * ロガーの実装は、\ `Logback <http://logback.qos.ch/>`_\ を利用する。


共通ライブラリ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* \ `https://github.com/terasolunaorg/terasoluna-gfw <https://github.com/terasolunaorg/terasoluna-gfw>`_\
* 詳細は\ :ref:`frameworkstack_common_library`\ を参照されたい。

利用するOSSのバージョン
--------------------------------------------------------------------------------

version 5.0.1.RELEASEで利用するOSSの一覧を以下に示す。

.. tip::

    version 5.0.0.RELEASEより、
    `Spring IO platform <http://platform.spring.io/platform/>`_\ の\ ``<dependencyManagement>``\ をインポートする構成を採用している。

    Spring IO platformの\ ``<dependencyManagement>``\ をインポートすることで、

    * Spring Frameworkが提供しているライブラリ
    * Spring Frameworkが依存しているOSSライブラリ
    * Spring Frameworkと相性のよいOSSライブラリ

    への依存関係を解決しており、
    TERASOLUNA Server Framework for Java (5.x)で使用するOSSのバージョンは、原則として、Spring IO platformの定義に準じている。

    なお、version 5.0.1.RELEASEで指定しているSpring IO platformのバージョンは、`1.1.3.RELEASE <http://docs.spring.io/platform/docs/1.1.3.RELEASE/reference/htmlsingle/>`_\ である。

.. tabularcolumns:: |p{0.15\linewidth}|p{0.27\linewidth}|p{0.25\linewidth}|p{0.15\linewidth}|p{0.05\linewidth}|p{0.08\linewidth}|
.. list-table::
    :header-rows: 1
    :stub-columns: 1
    :widths: 15 27 25 15 5 8

    * - Type
      - GroupId
      - ArtifactId
      - Version
      - Spring IO platform
      - Remarks
    * - Spring
      - org.springframework
      - spring-aop
      - 4.1.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-aspects
      - 4.1.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-beans
      - 4.1.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-context
      - 4.1.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-context-support
      - 4.1.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-core
      - 4.1.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-expression
      - 4.1.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-jdbc
      - 4.1.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-orm
      - 4.1.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-tx
      - 4.1.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-web
      - 4.1.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-webmvc
      - 4.1.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework.data
      - spring-data-commons
      - 1.9.3.RELEASE
      - \*
      -
    * - Spring
      - org.springframework.security
      - spring-security-acl
      - 3.2.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework.security
      - spring-security-config
      - 3.2.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework.security
      - spring-security-core
      - 3.2.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework.security
      - spring-security-taglibs
      - 3.2.7.RELEASE
      - \*
      -
    * - Spring
      - org.springframework.security
      - spring-security-web
      - 3.2.7.RELEASE
      - \*
      -
    * - MyBatis3
      - org.mybatis
      - mybatis
      - 3.2.8
      -
      - \*1
    * - MyBatis3
      - org.mybatis
      - mybatis-spring
      - 1.2.2
      -
      - \*1
    * - JPA(Hibernate)
      - antlr
      - antlr
      - 2.7.7
      - \*
      - \*2
    * - JPA(Hibernate)
      - dom4j
      - dom4j
      - 1.6.1
      - \*
      - \*2
    * - JPA(Hibernate)
      - org.hibernate
      - hibernate-core
      - 4.3.10.Final
      - \*
      - \*2
    * - JPA(Hibernate)
      - org.hibernate
      - hibernate-entitymanager
      - 4.3.10.Final
      - \*
      - \*2
    * - JPA(Hibernate)
      - org.hibernate.common
      - hibernate-commons-annotations
      - 4.0.5.Final
      - \*
      - \*2 \*4
    * - JPA(Hibernate)
      - org.hibernate.javax.persistence
      - hibernate-jpa-2.1-api
      - 1.0.0.Final
      - \*
      - \*2 \*4
    * - JPA(Hibernate)
      - org.javassist
      - javassist
      - 3.18.1-GA
      - \*
      - \*2
    * - JPA(Hibernate)
      - org.jboss
      - jandex
      - 1.1.0.Final
      - \*
      - \*2 \*4
    * - JPA(Hibernate)
      - org.jboss.logging
      - jboss-logging-annotations
      - 1.2.0.Final
      - \*
      - \*2 \*4 \*5
    * - JPA(Hibernate)
      - org.jboss.spec.javax.transaction
      - jboss-transaction-api_1.2_spec
      - 1.0.0.Final
      - \*
      - \*2 \*4
    * - JPA(Hibernate)
      - org.springframework.data
      - spring-data-jpa
      - 1.7.3.RELEASE
      - \*
      - \*2
    * - DI
      - javax.inject
      - javax.inject
      - 1
      - \*
      -
    * - AOP
      - aopalliance
      - aopalliance
      - 1
      - \*
      -
    * - AOP
      - org.aspectj
      - aspectjrt
      - 1.8.6
      - \*
      -
    * - AOP
      - org.aspectj
      - aspectjweaver
      - 1.8.6
      - \*
      -
    * - ログ出力
      - ch.qos.logback
      - logback-classic
      - 1.1.3
      - \*
      -
    * - ログ出力
      - ch.qos.logback
      - logback-core
      - 1.1.3
      - \*
      - \*4
    * - ログ出力
      - org.lazyluke
      - log4jdbc-remix
      - 0.2.7
      -
      -
    * - ログ出力
      - org.slf4j
      - jcl-over-slf4j
      - 1.7.12
      - \*
      -
    * - ログ出力
      - org.slf4j
      - slf4j-api
      - 1.7.12
      - \*
      -
    * - JSON
      - com.fasterxml.jackson.core
      - jackson-annotations
      - 2.4.6
      - \*
      -
    * - JSON
      - com.fasterxml.jackson.core
      - jackson-core
      - 2.4.6
      - \*
      -
    * - JSON
      - com.fasterxml.jackson.core
      - jackson-databind
      - 2.4.6
      - \*
      -
    * - JSON
      - com.fasterxml.jackson.datatype
      - jackson-datatype-joda
      - 2.4.6
      - \*
      -
    * - 入力チェック
      - javax.validation
      - validation-api
      - 1.1.0.Final
      - \*
      -
    * - 入力チェック
      - org.hibernate
      - hibernate-validator
      - 5.1.3.Final
      - \*
      -
    * - 入力チェック
      - org.jboss.logging
      - jboss-logging
      - 3.1.3.GA
      - \*
      - \*4
    * - 入力チェック
      - com.fasterxml
      - classmate
      - 1.0.0
      - \*
      - \*4
    * - Bean変換
      - commons-beanutils
      - commons-beanutils
      - 1.9.2
      - \*
      - \*3
    * - Bean変換
      - net.sf.dozer
      - dozer
      - 5.5.1
      -
      - \*3
    * - Bean変換
      - net.sf.dozer
      - dozer-spring
      - 5.5.1
      -
      - \*3
    * - Bean変換
      - org.apache.commons
      - commons-lang3
      - 3.3.2
      - \*
      - \*3
    * - 日付操作
      - joda-time
      - joda-time
      - 2.5
      - \*
      -
    * - 日付操作
      - joda-time
      - joda-time-jsptags
      - 1.1.1
      -
      - \*3
    * - 日付操作
      - org.jadira.usertype
      - usertype.core
      - 3.2.0.GA
      -
      - \*2
    * - 日付操作
      - org.jadira.usertype
      - usertype.spi
      - 3.2.0.GA
      -
      - \*2
    * - コネクションプール
      - org.apache.commons
      - commons-dbcp2
      - 2.0.1
      - \*
      - \*3
    * - コネクションプール
      - org.apache.commons
      - commons-pool2
      - 2.2
      - \*
      - \*3
    * - Tiles
      - commons-digester
      - commons-digester
      - 2.1
      - \*
      - \*3
    * - Tiles
      - org.apache.tiles
      - tiles-api
      - 3.0.5
      - \*
      - \*3
    * - Tiles
      - org.apache.tiles
      - tiles-core
      - 3.0.5
      - \*
      - \*3
    * - Tiles
      - org.apache.tiles
      - tiles-jsp
      - 3.0.5
      - \*
      - \*3
    * - Tiles
      - org.apache.tiles
      - tiles-servlet
      - 3.0.5
      - \*
      - \*3
    * - Tiles
      - org.apache.tiles
      - tiles-template
      - 3.0.5
      - \*
      - \*3 \*4
    * - Tiles
      - org.apache.tiles
      - tiles-autotag-core-runtime
      - 1.1.0
      - \*
      - \*3 \*4
    * - Tiles
      - org.apache.tiles
      - tiles-request-servlet
      - 1.0.6
      - \*
      - \*3 \*4
    * - Tiles
      - org.apache.tiles
      - tiles-request-api
      - 1.0.6
      - \*
      - \*3
    * - Tiles
      - org.apache.tiles
      - tiles-request-jsp
      - 1.0.6
      - \*
      - \*3 \*4
    * - ユーティリティ
      - com.google.guava
      - guava
      - 17.0
      - \*
      -
    * - ユーティリティ
      - commons-collections
      - commons-collections
      - 3.2.1
      - \*
      - \*3
    * - ユーティリティ
      - commons-io
      - commons-io
      - 2.4
      - \*
      - \*3
    * - サーブレット
      - org.apache.taglibs
      - taglibs-standard-jstlel
      - 1.2.5
      - \*
      -
    * - サーブレット
      - org.apache.taglibs
      - taglibs-standard-spec
      - 1.2.5
      - \*
      - \*4
    * - サーブレット
      - org.apache.taglibs
      - taglibs-standard-impl
      - 1.2.5
      - \*
      - \*4

#. | データアクセスに、MyBatis3を使用する場合に依存するライブラリ
#. | データアクセスに、JPAを使用する場合に依存するライブラリ
#. | 共通ライブラリに依存しないが、TERASOLUNA Server Framework for Java (5.x)でアプリケーションを開発する場合に、利用することを推奨しているライブラリ
#. | Spring IO platformでサポートしているライブラリが個別に依存しているライブラリ
   | (Spring IO platformとしては依存関係の管理は行っていないライブラリ)
#. | Spring IO platformで適用されるバージョンが、BetaやRC(Release Candidate)であるライブラリ
   | (TERASOLUNA Server Framework for Java (5.x)側でGAのバージョンを明示的に指定しているライブラリ)

.. _frameworkstack_common_library:


共通ライブラリの構成要素
--------------------------------------------------------------------------------

\ `共通ライブラリ <https://github.com/terasolunaorg/terasoluna-gfw>`_\ は、TERASOLUNA Server Framework for Java (5.x)が含むSpring Ecosystem や、その他依存ライブラリでは足りない+αな機能を提供するライブラリである。
基本的には、このライブラリがなくてもTERASOLUNA Server Framework for Java (5.x)によるアプリケーション開発は可能であるが、"あると便利"な存在である。

.. tabularcolumns:: |p{0.05\linewidth}|p{0.30\linewidth}|p{0.45\linewidth}|p{0.20\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 5 30 45 20

    * - 項番
      - プロジェクト名
      - 概要
      - Javaソースコード有無
    * - \ (1)
      - terasoluna-gfw-common
      - Webに依存しない汎用的に使用できる機能と依存関係定義を提供する。
      - 有
    * - \ (2)
      - terasoluna-gfw-jodatime
      - Joda Timeに依存する機能と依存関係定義を提供する。
      - 有
    * - \ (3)
      - terasoluna-gfw-web
      - Webアプリケーションを作成する場合に使用する機能と依存関係定義を提供する。
      - 有
    * - \ (4)
      - terasoluna-gfw-mybatis3
      - MyBatis3を使用する場合の依存関係定義を提供する。
      - 無
    * - \ (5)
      - terasoluna-gfw-jpa
      - JPAを使用する場合の依存関係定義を提供する。
      - 無
    * - \ (6)
      - terasoluna-gfw-security-core
      - Spring Securityを使用する場合の依存関係定義(Web以外)を提供する。
      - 無
    * - \ (7)
      - terasoluna-gfw-security-web
      - Spring Securityを使用する場合の依存関係定義(Web関連)とSpring Securityの拡張部品を提供する。
      - 有
    * - \ (8)
      - terasoluna-gfw-recommended-dependencies
      - Webに依存しない推奨ライブラリへの依存関係定義を提供する。
      - 無
    * - \ (9)
      - terasoluna-gfw-recommended-web-dependencies
      - Webに依存する推奨ライブラリへの依存関係定義を提供する。
      - 無
    * - \ (10)
      - terasoluna-gfw-parent
      - 依存ライブラリの管理とビルド用プラグインの推奨設定を提供する。
      - 無

Javaソースコードを含まないものは、ライブラリの依存関係のみ定義しているプロジェクトである。

なお、プロジェクトの依存関係は以下の通りである。

.. figure:: images_FrameworkStack/FrameworkStackProjectDependencies.png
    :width: 75%


terasoluna-gfw-common
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

terasoluna-gfw-commonは以下の部品を提供している。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 30 50

    * - 分類
      - 部品名
      - 説明
    * - :doc:`../ArchitectureInDetail/ExceptionHandling`
      - 例外クラス
      - 汎用的に使用できる例外クラスを提供する。
    * -
      - 例外ロガー
      - プリケーション内で発生した例外をログに出力するためのロガークラスを提供する。
    * -
      - 例外コード
      - 例外クラスに対応する例外コード(メッセージID)を解決するための仕組み(クラス)を提供する。
    * -
      - 例外ログ出力インターセプタ
      - ドメイン層で発生した例外をログ出力するためのインターセプタクラス(AOP)を提供する。
    * - :doc:`../ArchitectureInDetail/SystemDate`
      - システム時刻ファクトリ
      - システム時刻を取得するためのクラスを提供する。
    * - :doc:`../ArchitectureInDetail/Codelist`
      - コードリスト
      - コードリストを生成するためのクラスを提供する。
    * - :doc:`../ArchitectureInDetail/DataAccessCommon`
      - クエリエスケープ
      - SQL及びJPQLにバインドする値のエスケープ処理を行うクラスを提供する。
    * -
      - シーケンサ
      - シーケンス値を取得するためのクラスを提供する。

terasoluna-gfw-jodatime
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

terasoluna-gfw-jodatimeは以下の部品を提供している。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 30 50

    * - 分類
      - 部品名
      - 説明
    * - :doc:`../ArchitectureInDetail/SystemDate`
      - Joda Time用システム時刻ファクトリ
      - Joda TimeのAPIを利用してシステム時刻を取得するためのクラスを提供する。


terasoluna-gfw-web
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

terasoluna-gfw-webは以下の部品を提供している。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 30 50

    * - 分類
      - 部品名
      - 説明
    * - :doc:`../ArchitectureInDetail/DoubleSubmitProtection`
      - トランザクショントークンチェック
      - リクエストの二重送信からWebアプリケーションを守るための仕組み(クラス)を提供する。
    * - :doc:`../ArchitectureInDetail/ExceptionHandling`
      - 例外ハンドラ
      - 共通ライブラリが提供する例外ハンドリングの部品と連携するための例外ハンドラクラス(Spring MVC提供のクラスのサブクラス)を提供する。
    * -
      - 例外ログ出力インターセプタ
      - Spring MVCの例外ハンドラがハンドリングした例外をログ出力するためのインターセプタクラス(AOP)を提供する。
    * - :doc:`../ArchitectureInDetail/Codelist`
      - コードリスト埋込インターセプタ
      - Viewからコードリストを取得できるようにするために、コードリストの情報をリクエストスコープに格納するためのインターセプタクラス(Spring MVC Interceptor)を提供する。
    * - :doc:`../ArchitectureInDetail/FileDownload`
      - 汎用ダウンロードView
      - ストリームから取得したデータを、ダウンロード用のストリームに出力するための抽象クラスを提供する。
    * - :doc:`../ArchitectureInDetail/Logging`
      - トラッキングID格納用サーブレットフィルタ
      - トレーサビリティを向上させるために、
        クライアントから指定されたトラッキングIDを、ロガーのMDC(Mapped Diagnostic Context)、リクエストスコープ、レスポンスヘッダに設定するためのサーブレットフィルタクラスを提供する。
        (クライアントからトラッキングIDの指定がない場合は、本クラスでトラッキングIDを生成する)
    * -
      - 汎用MDC格納用サーブレットフィルタ
      - ロガーのMDCに任意の値を設定するための抽象クラスを提供する。
    * -
      - MDCクリア用サーブレットフィルタ
      - ロガーのMDCに格納されている情報をクリアするためのサーブレットフィルタクラスを提供する。
    * - :doc:`../ArchitectureInDetail/Pagination`
      - ページネーションリンク表示用のJSPタグ
      - Spring Data Commons提供のクラスと連携してページネーションリンクを表示するためのJSPタグライブラリを提供する。
    * - :doc:`../ArchitectureInDetail/MessageManagement`
      - 結果メッセージ表示用のJSPタグ
      - 処理結果を表示するためのJSPタグライブラリを提供する。
    * - :ref:`TagLibAndELFunctionsOverviewELFunctions`
      - XSS対策用EL関数
      - XSS対策用のEL関数を提供する。
    * -
      - URL用EL関数
      - URLエンコーディングなどのURL用のEL関数を提供する。
    * -
      - DOM変換用EL関数
      - DOM文字列に変換するためのEL関数を提供する。
    * -
      - ユーティリティEL関数
      - 汎用的なユーティリティ処理を行うためのEL関数を提供する。

terasoluna-gfw-security-web
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

terasoluna-gfw-security-webは以下の部品を提供している。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 30 50

    * - 分類
      - 部品名
      - 説明
    * - :doc:`../ArchitectureInDetail/Logging`
      - 認証ユーザ名格納用サーブレットフィルタ
      - トレーサビリティを向上させるために、
        認証ユーザ名をロガーのMDCに設定するためのサーブレットフィルタクラスを提供する。
    * - :doc:`../Security/Authentication`
      - リダイレクト先の指定が可能な認証成功ハンドラ
      - 認証が成功した際に、Webアプリケーション内の任意のパスにリダイレクトするためのハンドラクラスを提供する。


.. raw:: latex

   \newpage

