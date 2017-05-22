TERASOLUNA Global Frameworkのスタック
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

TERASOLUNA Global FrameworkのSoftware Framework概要
--------------------------------------------------------------------------------

TERASOLUNA Global Frameworkで使用するSoftware Frameworkは独自のフレームワークではなく、\ `Spring Framework <http://projects.spring.io/spring-framework/>`_\ を中心としたOSSの組み合わせである。

.. figure:: images/introduction-software-framework.png
   :width: 95%


Software Frameworkの主な構成要素
--------------------------------------------------------------------------------

TERASOLUNA Global Frameworkを構成するライブラリを以下に示す。

.. figure:: images/introduction-software-stack.png
   :width: 95%

DIコンテナ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
DIコンテナとしてSpring Frameworkを利用する。


* `Spring Framework 3.2 <http://docs.spring.io/spring/docs/3.2.18.RELEASE/spring-framework-reference/html/beans.html>`_

MVCフレームワーク
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Web MVCフレームワークとしてSpring MVCを利用する。

* `Spring MVC 3.2 <http://docs.spring.io/spring/docs/3.2.18.RELEASE/spring-framework-reference/html/mvc.html>`_

O/R Mapper
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

本ガイドラインでは、以下の\ **どちらか**\ を想定している。

* `JPA2.0 <http://download.oracle.com/otn-pub/jcp/persistence-2.0-fr-eval-oth-JSpec/persistence-2_0-final-spec.pdf>`_

  * プロバイダは、\ `Hibernate 4.2 <http://docs.jboss.org/hibernate/orm/4.2/manual/en-US/html/>`_\ を使用する。

* `MyBatis 2.3.5 <https://mybatis.googlecode.com/files/MyBatis-SqlMaps-2_en.pdf>`_

  * ラッパーとして、\ `TERASOLUNA Framework <http://sourceforge.jp/projects/terasoluna/releases/?package_id=6896>`_\ のDAO(TERASOLUNA DAO)を使用する。

.. note::

  MyBatisは正確には「SQL Mapper」であるが、本ガイドラインでは「O/R Mapper」に分類する。

.. warning::

  どんなプロジェクトでもJPAを採用できるわけではない。"テーブルがほとんど正規化されいない"、"テーブルのカラム数が多すぎる"というテーブル設計がされている場合に、JPAの利用は難しい。

  また、本ガイドラインではJPAの基本的な説明は行っておらず、JPA利用経験者がチーム内にいることが前提である。

View
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
ViewにはJSPを利用する。

Viewのレイアウトを共通化する場合は、

* `Apache Tiles 2.2 <http://tiles.apache.org/2.2/framework/index.html>`_

を利用する。

セキュリティ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
認証・認可のフレームワークとしてSpring Securityを利用する。

* `Spring Security 3.1 <http://docs.spring.io/spring-security/site/docs/3.1.4.RELEASE/reference/springsecurity.html>`_

.. todo::

  今後、Spring Security 3.2にupdateする予定である。

バリデーション
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* 単項目チェックには\ `BeanValidation 1.0 <http://download.oracle.com/otn-pub/jcp/bean_validation-1.0-fr-oth-JSpec/bean_validation-1_0-final-spec.pdf>`_\ を利用する。

  * 実装は、\ `Hibernate Validator 4.3 <http://docs.jboss.org/hibernate/validator/4.3/reference/en-US/html/>`_\ を利用する。

* 相関チェックには\ `BeanValidation <http://download.oracle.com/otn-pub/jcp/bean_validation-1.0-fr-oth-JSpec/bean_validation-1_0-final-spec.pdf>`_\ 、もしくは\ `Spring Validation <http://docs.spring.io/spring/docs/3.2.18.RELEASE/spring-framework-reference/html/validation.html>`_\ を利用する。

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

version 1.0.6.RELEASEで利用するOSSの一覧を以下に示す。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.25\linewidth}|p{0.25\linewidth}|p{0.25\linewidth}|p{0.5\linewidth}|
.. list-table::
    :header-rows: 1
    :stub-columns: 1
    :widths: 20 25 25 25 5

    * - Type
      - GroupId
      - ArtifactId
      - Version
      - Remarks
    * - Spring
      - org.springframework
      - spring-aop
      - 3.2.18.RELEASE
      -
    * - Spring
      - org.springframework
      - spring-aspects
      - 3.2.18.RELEASE
      -
    * - Spring
      - org.springframework
      - spring-beans
      - 3.2.18.RELEASE
      -
    * - Spring
      - org.springframework
      - spring-context
      - 3.2.18.RELEASE
      -
    * - Spring
      - org.springframework
      - spring-context-support
      - 3.2.18.RELEASE
      -
    * - Spring
      - org.springframework
      - spring-core
      - 3.2.18.RELEASE
      -
    * - Spring
      - org.springframework
      - spring-expression
      - 3.2.18.RELEASE
      -
    * - Spring
      - org.springframework
      - spring-jdbc
      - 3.2.18.RELEASE
      -
    * - Spring
      - org.springframework
      - spring-orm
      - 3.2.18.RELEASE
      -
    * - Spring
      - org.springframework
      - spring-tx
      - 3.2.18.RELEASE
      -
    * - Spring
      - org.springframework
      - spring-web
      - 3.2.18.RELEASE
      -
    * - Spring
      - org.springframework
      - spring-webmvc
      - 3.2.18.RELEASE
      -
    * - Spring
      - org.springframework.data
      - spring-data-commons
      - 1.6.4.RELEASE
      -
    * - Spring
      - org.springframework.security
      - spring-security-acl
      - 3.1.4.RELEASE
      -
    * - Spring
      - org.springframework.security
      - spring-security-config
      - 3.1.4.RELEASE
      -
    * - Spring
      - org.springframework.security
      - spring-security-core
      - 3.1.4.RELEASE
      -
    * - Spring
      - org.springframework.security
      - spring-security-taglibs
      - 3.1.4.RELEASE
      -
    * - Spring
      - org.springframework.security
      - spring-security-web
      - 3.1.4.RELEASE
      -
    * - JPA(Hibernate)
      - antlr
      - antlr
      - 2.7.7
      - \*1
    * - JPA(Hibernate)
      - dom4j
      - dom4j
      - 1.6.1
      - \*1
    * - JPA(Hibernate)
      - org.hibernate
      - hibernate-core
      - 4.2.3.Final
      - \*1
    * - JPA(Hibernate)
      - org.hibernate
      - hibernate-entitymanager
      - 4.2.3.Final
      - \*1
    * - JPA(Hibernate)
      - org.hibernate.common
      - hibernate-commons-annotations
      - 4.0.2.Final
      - \*1
    * - JPA(Hibernate)
      - org.hibernate.javax.persistence
      - hibernate-jpa-2.0-api
      - 1.0.1.Final
      - \*1
    * - JPA(Hibernate)
      - org.javassist
      - javassist
      - 3.15.0-GA
      - \*1
    * - JPA(Hibernate)
      - org.jboss.spec.javax.transaction
      - jboss-transaction-api_1.1_spec
      - 1.0.1.Final
      - \*1
    * - JPA(Hibernate)
      - org.springframework.data
      - spring-data-jpa
      - 1.4.3.RELEASE
      - \*1
    * - MyBatis2
      - jp.terasoluna.fw
      - terasoluna-dao
      - 2.0.5.0
      - \*2
    * - MyBatis2
      - jp.terasoluna.fw
      - terasoluna-ibatis
      - 2.0.5.0
      - \*2
    * - MyBatis2
      - org.mybatis
      - mybatis
      - 2.3.5
      - \*2
    * - DI
      - javax.inject
      - javax.inject
      - 1
      -
    * - AOP
      - aopalliance
      - aopalliance
      - 1
      -
    * - AOP
      - org.aspectj
      - aspectjrt
      - 1.7.4
      -
    * - AOP
      - org.aspectj
      - aspectjweaver
      - 1.7.4
      -
    * - ログ出力
      - ch.qos.logback
      - logback-classic
      - 1.0.13
      -
    * - ログ出力
      - ch.qos.logback
      - logback-core
      - 1.0.13
      -
    * - ログ出力
      - org.lazyluke
      - log4jdbc-remix
      - 0.2.7
      -
    * - ログ出力
      - org.slf4j
      - jcl-over-slf4j
      - 1.7.5
      -
    * - ログ出力
      - org.slf4j
      - slf4j-api
      - 1.7.5
      -
    * - JSON
      - org.codehaus.jackson
      - jackson-core-asl
      - 1.9.7
      -
    * - JSON
      - org.codehaus.jackson
      - jackson-mapper-asl
      - 1.9.7
      -
    * - 入力チェック
      - javax.validation
      - validation-api
      - 1.0.0.GA
      -
    * - 入力チェック
      - org.hibernate
      - hibernate-validator
      - 4.3.1.Final
      -
    * - 入力チェック
      - org.jboss.logging
      - jboss-logging
      - 3.1.0.GA
      -
    * - Bean変換
      - commons-beanutils
      - commons-beanutils
      - 1.8.3
      - \*3
    * - Bean変換
      - net.sf.dozer
      - dozer
      - 5.4.0
      - \*3
    * - Bean変換
      - org.apache.commons
      - commons-lang3
      - 3.1
      - \*3
    * - 日付操作
      - joda-time
      - joda-time
      - 2.2
      -
    * - 日付操作
      - joda-time
      - joda-time-jsptags
      - 1.1.1
      - \*3
    * - 日付操作
      - org.jadira.usertype
      - usertype.core
      - 3.0.0.GA
      - \*1
    * - 日付操作
      - org.jadira.usertype
      - usertype.spi
      - 3.0.0.GA
      - \*1
    * - コネクションプール
      - commons-dbcp
      - commons-dbcp
      - 1.2.2.patch_DBCP264_DBCP372
      - \*3
    * - コネクションプール
      - commons-pool
      - commons-pool
      - 1.6
      - \*3
    * - Tiles
      - commons-digester
      - commons-digester
      - 2
      - \*3
    * - Tiles
      - org.apache.tiles
      - tiles-api
      - 2.2.2
      - \*3
    * - Tiles
      - org.apache.tiles
      - tiles-core
      - 2.2.2
      - \*3
    * - Tiles
      - org.apache.tiles
      - tiles-jsp
      - 2.2.2
      - \*3
    * - Tiles
      - org.apache.tiles
      - tiles-servlet
      - 2.2.2
      - \*3
    * - Tiles
      - org.apache.tiles
      - tiles-template
      - 2.2.2
      - \*3
    * - ユーティリティ
      - com.google.guava
      - guava
      - 13.0.1
      -
    * - ユーティリティ
      - commons-collections
      - commons-collections
      - 3.2.2
      - \*3
    * - ユーティリティ
      - commons-io
      - commons-io
      - 2.4
      - \*3
    * - サーブレット
      - javax.servlet
      - jstl
      - 1.2
      -

#. データアクセスに、JPAを使用する場合に依存するライブラリ
#. データアクセスに、MyBatis2を使用する場合に依存するライブラリ
#. 共通ライブラリに依存しないが、TERASOLUNA Global Frameworkでアプリケーションを開発する場合に、利用することを推奨しているライブラリ


.. _frameworkstack_common_library:


共通ライブラリの構成要素
--------------------------------------------------------------------------------

\ `共通ライブラリ <https://github.com/terasolunaorg/terasoluna-gfw>`_\ は、TERASOLUNA Global Frameworkが含むSpring Ecosystem や、その他依存ライブラリでは足りない+αな機能を提供するライブラリである。
基本的には、このライブラリがなくてもTERASOLUNA Global Frameworkによるアプリケーション開発は可能であるが、"あると便利"な存在である。

.. tabularcolumns:: |p{0.05\linewidth}|p{0.30\linewidth}|p{0.35\linewidth}|p{0.30\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 5 30 35 30

    * - 項番
      - プロジェクト名
      - 概要
      - Javaソースコード有無
    * - | (1)
      - | terasoluna-gfw-common
      - | Webに限らず、汎用的に使用できる機能
      - | 有
    * - | (2)
      - | terasoluna-gfw-web
      - | Webアプリケーションを作成する場合に使用する機能群
      - | 有
    * - | (3)
      - | terasoluna-gfw-jpa
      - | JPAを使用する場合の、依存関係定義
      - | 無
    * - | (4)
      - | terasoluna-gfw-mybatis2
      - | MyBatis2を使用する場合の、依存関係定義
      - | 無
    * - | (5)
      - | terasoluna-gfw-security-core
      - | Spring Securityを使用する場合の、依存関係定義(Web以外)
      - | 無
    * - | (6)
      - | terasoluna-gfw-security-web
      - | Spring Securityを使用する場合の依存関係定義(Web関連)、およびSpring Securityの拡張
      - | 有

Javaソースコードを含まないものは、ライブラリの依存関係のみ定義しているプロジェクトである。



terasoluna-gfw-common
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

terasoluna-gfw-commonは以下の部品を提供している。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.35\linewidth}|p{0.30\linewidth}|
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

terasoluna-gfw-web
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

terasoluna-gfw-webは以下の部品を提供している。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.35\linewidth}|p{0.30\linewidth}|
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

.. tabularcolumns:: |p{0.30\linewidth}|p{0.35\linewidth}|p{0.30\linewidth}|
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
    * - :doc:`../Security/CSRF`
      - CSRF(Cross site request forgeries)チェック
      - CSRFを使用した攻撃からWebアプリケーションを守るための仕組み(クラス)を提供する。
        (Spring Security 3.2を導入するまでの暫定措置)

.. raw:: latex

   \newpage

