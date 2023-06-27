Webアプリケーション向け開発プロジェクトの作成
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

本節では、Webアプリケーション向けの開発プロジェクトを作成する方法について説明する。

本ガイドラインでは、マルチプロジェクト構成を採用することを推奨している。
推奨するマルチプロジェクト構成の説明については、「:ref:`application-layering_project-structure`」を参照されたい。

.. _CreateProjectFromBlankTypes:

ブランクプロジェクトの種類
--------------------------------------------------------------------------------

ブランクプロジェクトは、使用用途に応じて以下の２種類を提供している。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 70

    * - 種別
      - 使用用途
    * - | `マルチプロジェクト構成のブランクプロジェクト <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank>`_
      - 商用環境にリリースするような本格的なアプリケーションを開発する際に使用する。

        プロジェクトの雛形は、MavenのArchetypeとして、以下の3種類を用意している。

        * MyBatis3用の設定が盛り込まれた雛形
        * JPA(Spring Data JPA)用の設定が盛り込まれた雛形

        **本ガイドラインでは、マルチプロジェクト構成のプロジェクトを使用する事を推奨している。**
    * - | `シングルプロジェクト構成のブランクプロジェクト <https://github.com/terasolunaorg/terasoluna-gfw-web-blank>`_
      - POC(Proof Of Concept)、プロトタイプ、サンプルなどの簡易的なアプリケーションを作成する際に使用する。

        プロジェクトの雛形は、MavenのArchetypeとして、以下の4種類を用意している。
        (EclipseのWTP用のプロジェクトも用意しているが、本節では説明は割愛する)

        * MyBatis3用の設定が盛り込まれた雛形
        * JPA(Spring Data JPA)用の設定が盛り込まれた雛形
        * O/R Mapperに依存しない雛形

        本ガイドラインでは、各種チュートリアルをシングルプロジェクトを使用して行う手順となっている。

.. _CreateWebApplicationProject:

開発プロジェクトの作成
--------------------------------------------------------------------------------

マルチプロジェクト構成の開発プロジェクトを、
`Maven Archetype Plugin <http://maven.apache.org/archetype/maven-archetype-plugin/>`_ の `archetype:generate <http://maven.apache.org/archetype/maven-archetype-plugin/generate-mojo.html>`_ を使用して作成する。

.. note:: **前提条件**

    以降の説明では、

    * `Maven <http://maven.apache.org/>`_ (\ ``mvn``\ コマンド)が使用可能であること
    * インターネットに繋がっていること
    * インターネットにプロキシ経由で繋ぐ場合は、`Mavenのプロキシ設定 <http://maven.apache.org/guides/mini/guide-proxies.html>`_  が行われていること

    を前提としている。

    前提条件が整っていない場合は、まずこれらのセットアップを行ってほしい。

|

マルチプロジェクトを作成するためのArchetypeとして、以下の2種類を用意している。

.. tabularcolumns:: |p{0.05\linewidth}|p{0.30\linewidth}|p{0.65\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 5 30 65

    * - 項番
      - Archetype(ArtifactId)
      - 説明
    * - 1.
      - terasoluna-gfw-multi-web-blank-mybatis3-archetype
      - O/R MapperとしてMyBatis3を使用するためのプロジェクトを生成するためのArchetype。
    * - 2.
      - terasoluna-gfw-multi-web-blank-jpa-archetype
      - O/R MapperとしてJPA(with Spring Data JPA and Hibernate)を使用するためのプロジェクトを生成するためのArchetype。

|

プロジェクトを作成するフォルダに移動する。

.. code-block:: console

    cd C:\work

|

`Maven Archetype Plugin <http://maven.apache.org/archetype/maven-archetype-plugin/>`_ の `archetype:generate <http://maven.apache.org/archetype/maven-archetype-plugin/generate-mojo.html>`_ を使用して、プロジェクトを作成する。

.. code-block:: console

    mvn archetype:generate -B^
     -DarchetypeGroupId=org.terasoluna.gfw.blank^
     -DarchetypeArtifactId=terasoluna-gfw-multi-web-blank-mybatis3-archetype^
     -DarchetypeVersion=5.3.0.RELEASE^
     -DgroupId=com.example.todo^
     -DartifactId=todo^
     -Dversion=1.0.0-SNAPSHOT

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 25 75

    * - パラメータ
      - 説明
    * - | \-B
      - batch mode (対話を省略)
    * - | \-DarchetypeGroupId
      - ブランクプロジェクトのgroupIdを指定する。(固定)
    * - | \-DarchetypeArtifactId
      - ブランクプロジェクトのarchetypeId(雛形を特定するためのID)を指定する。**(カスタマイズが必要)**

        以下の何れかのarchetypeIdを指定する。

        * ``terasoluna-gfw-multi-web-blank-mybatis3-archetype``
        * ``terasoluna-gfw-multi-web-blank-jpa-archetype``

        上記例では、\ ``terasoluna-gfw-multi-web-blank-mybatis3-archetype``\ を指定している。
    * - | \-DarchetypeVersion
      - ブランクプロジェクトのバージョンを指定する。(固定)
    * - | \-DgroupId
      - 作成するプロジェクトのgroupIdを指定する。**(カスタマイズが必要)**

        上記例では、\ ``"com.example.todo"``\ を指定している。
    * - | \-DartifactId
      - 作成するプロジェクトのartifactIdを指定する。**(カスタマイズが必要)**

        上記例では、\ ``"todo"``\ を指定している。
    * - | \-Dversion
      - 作成するプロジェクトのバージョンを指定する。**(カスタマイズが必要)**

        上記例では、\ ``"1.0.0-SNAPSHOT"``\ を指定している。

|

プロジェクトの作成が成功した場合、以下のようなログが出力される。
(以下は、MyBatis3用のArchetypeを使用して作成した場合の出力例)

.. code-block:: console

    (... omit)
    [INFO] ----------------------------------------------------------------------------
    [INFO] Using following parameters for creating project from Archetype: terasoluna-gfw-multi-web-blank-mybatis3-archetype:5.3.0.RELEASE
    [INFO] ----------------------------------------------------------------------------
    [INFO] Parameter: groupId, Value: com.example.todo
    [INFO] Parameter: artifactId, Value: todo
    [INFO] Parameter: version, Value: 1.0.0-SNAPSHOT
    [INFO] Parameter: package, Value: com.example.todo
    [INFO] Parameter: packageInPathFormat, Value: com/example/todo
    [INFO] Parameter: package, Value: com.example.todo
    [INFO] Parameter: version, Value: 1.0.0-SNAPSHOT
    [INFO] Parameter: groupId, Value: com.example.todo
    [INFO] Parameter: artifactId, Value: todo
    [INFO] Parent element not overwritten in C:\work\todo\todo-env\pom.xml
    [INFO] Parent element not overwritten in C:\work\todo\todo-domain\pom.xml
    [INFO] Parent element not overwritten in C:\work\todo\todo-web\pom.xml
    [INFO] Parent element not overwritten in C:\work\todo\todo-initdb\pom.xml
    [INFO] Parent element not overwritten in C:\work\todo\todo-selenium\pom.xml
    [INFO] project created from Archetype in dir: C:\work\todo
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 9.929 s
    [INFO] Finished at: 2015-07-31T12:03:21+00:00
    [INFO] Final Memory: 10M/26M
    [INFO] ------------------------------------------------------------------------

|

プロジェクトの作成が成功した場合、Mavenのマルチプロジェクトが作成される。
Maven Archetypeで作成したプロジェクトの詳細な説明については、「:ref:`CreateWebApplicationProjectConfiguration`」を参照されたい。

.. code-block:: console

    todo
    ├── pom.xml
    ├── todo-domain
    ├── todo-env
    ├── todo-initdb
    ├── todo-selenium
    └── todo-web


|


.. _CreateWebApplicationProjectCustomize:

開発プロジェクトのカスタマイズ
--------------------------------------------------------------------------------

Maven Archetypeで作成したプロジェクトには、アプリケーション毎にカスタマイズが必要な箇所がいくつか存在する。

カスタマイズが必要な箇所を以下に示す。

- :ref:`CreateWebApplicationProjectCustomizeProjectInformation`
- :ref:`CreateWebApplicationProjectCustomizeMessageId`
- :ref:`CreateWebApplicationProjectCustomizeMessageWording`
- :ref:`CreateWebApplicationProjectCustomizeErrorScreen`
- :ref:`CreateWebApplicationProjectCustomizeCopyrightOnScreenFooter`
- :ref:`CreateWebApplicationProjectCustomizeInMemoryDatabase`
- :ref:`CreateWebApplicationProjectCustomizeDataSource`

.. note::

    上記以外のカスタマイズポイントとしては、

    * :doc:`../Security/Authentication`・:doc:`../Security/Authorization` の設定
    * :doc:`../ArchitectureInDetail/WebApplicationDetail/FileUpload` を有効化するための設定
    * :doc:`../ArchitectureInDetail/WebApplicationDetail/Internationalization` を有効化するための設定
    * :doc:`../ArchitectureInDetail/GeneralFuncDetail/Logging` の定義
    * :doc:`../ArchitectureInDetail/WebApplicationDetail/ExceptionHandling` の定義
    * :doc:`../ArchitectureInDetail/WebServiceDetail/REST` 向けの設定の適用

    などがある。

    これらのカスタマイズについては、各節のHow to useを参照し、必要に応じてカスタマイズしてほしい。


.. note::

    以降の説明で\ ``artifactId``\ と表現している部分は、
    プロジェクト作成時に指定した\ ``artifactId``\ に置き換えて読み進めてほしい。

|

.. _CreateWebApplicationProjectCustomizeProjectInformation:

POMファイルのプロジェクト情報
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Maven Archetypeで作成したプロジェクトのPOMファイルでは、

* プロジェクト名(\ ``name``\ 要素)
* プロジェクト説明(\ ``description``\ 要素)
* プロジェクトURL(\ ``url``\ 要素)
* プロジェクト創設年(\ ``inceptionYear``\ 要素)
* プロジェクトライセンス(\ ``licenses``\ 要素)
* プロジェクト組織(\ ``organization``\ 要素)

といったプロジェクト情報が、Archetype自身のプロジェクト情報が設定されている状態となっている。
実際の設定内容を以下に示す。

.. code-block:: xml

    <!-- ... -->

    <name>TERASOLUNA Server Framework for Java (5.x) Web Blank Multi Project</name>
    <description>Web Blank Multi Project using TERASOLUNA Server Framework for Java (5.x)</description>
    <url>http://terasoluna.org</url>
    <inceptionYear>2014</inceptionYear>
    <licenses>
        <license>
            <name>Apache License, Version 2.0</name>
            <url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>
            <distribution>manual</distribution>
        </license>
    </licenses>
    <organization>
        <name>TERASOLUNA Framework Team</name>
        <url>http://terasoluna.org</url>
    </organization>

    <!-- ... -->

.. note::

    **プロジェクト情報には、適切な値を設定すること。**

|

カスタマイズ対象のファイルとカスタマイズ方法を以下に示す。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.45\linewidth}|p{0.45\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 45 45

    * - 項番
      - 対象ファイル
      - カスタマイズ方法
    * - 1.
      - マルチプロジェクト全体の構成を定義するPOM(Project Object Model)ファイル

        ``artifactId/pom.xml``
      - プロジェクト情報に適切な値を指定する。

|

.. _CreateWebApplicationProjectCustomizeMessageId:

x.xx.fw.9999形式のメッセージID
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Maven Archetypeで作成したプロジェクトでは、\ ``x.xx.fw.9999``\ 形式のメッセージIDを、

* エラー画面に表示するメッセージ
* 例外発生時に出力するエラーログ

を生成する際に使用している。実際の使用箇所(サンプリング)を以下に示す。

**[application-messages.properties]**

.. code-block:: properties

    e.xx.fw.5001 = Resource not found.

**[JSP]**

.. code-block:: jsp

    <div class="error">
        <c:if test="${!empty exceptionCode}">[${f:h(exceptionCode)}]</c:if>
        <spring:message code="e.xx.fw.5001" />
    </div>

**[applicationContext.xml]**

.. code-block:: xml

    <bean id="exceptionCodeResolver"
        class="org.terasoluna.gfw.common.exception.SimpleMappingExceptionCodeResolver">
        <!-- ... -->
                <entry key="ResourceNotFoundException" value="e.xx.fw.5001" />
        <!-- ... -->
    </bean>

|

\ ``x.xx.fw.9999``\ 形式のメッセージIDは、
本ガイドラインの「:doc:`../ArchitectureInDetail/WebApplicationDetail/MessageManagement`」で紹介しているメッセージID体系であるが、
プロジェクト区分の値が暫定値「\ ``xx``\ 」の状態になっている。

.. note::

    * **本ガイドラインで紹介しているメッセージID体系を利用する場合は、プロジェクト区分に適切な値を指定すること。** 本ガイドラインで紹介しているメッセージID体系については、「:ref:`message-management_result-rule`」を参照されたい。
    * 本ガイドラインで紹介しているメッセージID体系を利用しない場合は、以下に示す修正対象ファイル内で使用しているメッセージIDを全て置き換える必要がある。

|

カスタマイズ対象のファイルとカスタマイズ方法を以下に示す。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.45\linewidth}|p{0.45\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 45 45

    * - 項番
      - 対象ファイル
      - カスタマイズ方法
    * - 1.
      - メッセージ定義ファイル

        ``artifactId/artifactId-web/src/main/resources/i18n/application-messages.properties``
      - プロパティキーに指定しているメッセージIDのプロジェクト区分の暫定値「\ ``xx``\ 」を、適切な値に修正する。
    * - 2.
      - エラー画面用のJSP

        ``artifactId/artifactId-web/src/main/webapp/WEB-INF/views/common/error/*.jsp``
      - \ ``<spring:message>``\ 要素の\ ``code``\ 属性に指定しているメッセージIDのプロジェクト区分の暫定値「\ ``xx``\ 」を、適切な値に修正する。
    * - 3.
      - Webアプリケーション用のアプリケーションコンテキストを作成するためのBean定義ファイル

        ``artifactId/artifactId-web/src/main/resources/META-INF/spring/applicationContext.xml``
      - BeanIDが\ ``"exceptionCodeResolver"``\ のBean定義内で指定している例外コード(メッセージID)のプロジェクト区分の暫定値「\ ``xx``\ 」を、適切な値に修正する。

|

.. _CreateWebApplicationProjectCustomizeMessageWording:

メッセージ文言
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Maven Archetypeで作成したプロジェクトでは、いくつかのメッセージ定義を提供しているが、
メッセージ文言は簡易的なメッセージになっている。
実際のメッセージ(サンプリング)を以下に示す。

**[application-messages.properties]**

.. code-block:: properties

    e.xx.fw.5001 = Resource not found.

    # ...

    # typemismatch
    typeMismatch="{0}" is invalid.

    # ...

.. note::

    **メッセージ文言については、アプリケーション要件(メッセージ規約など)に合わせて修正すること。**

|

カスタマイズ対象のファイルとカスタマイズ方法を以下に示す。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.45\linewidth}|p{0.45\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 45 45

    * - 項番
      - 対象ファイル
      - カスタマイズ方法
    * - 1.
      - メッセージ定義ファイル

        ``artifactId/artifactId-web/src/main/resources/i18n/application-messages.properties``
      - アプリケーション要件に応じたメッセージに修正する。

        入力チェックでエラーとなった際に表示するメッセージ(Bean Validationのメッセージ)についても、
        アプリケーション要件に応じて修正(デフォルトメッセージの上書き)が必要になる。
        デフォルトメッセージの上書き方法については、「:ref:`Validation_message_def`」を参照されたい。

|

.. _CreateWebApplicationProjectCustomizeErrorScreen:

エラー画面
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Maven Archetypeで作成したプロジェクトでは、エラーの種類毎にエラー画面を表示するためのJSP及びHTMLを提供しているが、

* 画面レイアウト
* 画面タイトル
* メッセージの文言

などが簡易的な実装になっている。実際のJSPの実装(サンプリング)を以下に示す。

**[JSP]**

.. code-block:: jsp

    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="utf-8">
    <title>Resource Not Found Error!</title>
    <link rel="stylesheet"
        href="${pageContext.request.contextPath}/resources/app/css/styles.css">
    </head>
    <body>
        <div id="wrapper">
            <h1>Resource Not Found Error!</h1>
            <div class="error">
                <c:if test="${!empty exceptionCode}">[${f:h(exceptionCode)}]</c:if>
                <spring:message code="e.xx.fw.5001" />
            </div>
            <t:messagesPanel />
        <br>
        <!-- ... -->
        <br>
        </div>
    </body>
    </html>

.. note::

    **エラー画面を表示するためのJSPとHTMLについては、アプリケーション要件(UI規約など)に合わせて修正すること。**

|

カスタマイズ対象のファイルとカスタマイズ方法を以下に示す。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.45\linewidth}|p{0.45\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 45 45

    * - 項番
      - 対象ファイル
      - カスタマイズ方法
    * - 1.
      - エラー画面用のJSP

        ``artifactId/artifactId-web/src/main/webapp/WEB-INF/views/common/error/*.jsp``
      - アプリケーション要件(UI規約など)に合わせて修正する。

        エラー画面を表示するJSPをカスタマイズする際は、「:doc:`../ArchitectureInDetail/WebApplicationDetail/ExceptionHandling` の :ref:`exception-handling-how-to-use-codingpoint-jsp-label`」を参照されたい。
    * - 2.
      - エラー画面用のHTML

        ``artifactId/artifactId-web/src/main/webapp/WEB-INF/views/common/error/unhandledSystemError.html``
      - アプリケーション要件(UI規約など)に合わせて修正する。

|

.. _CreateWebApplicationProjectCustomizeCopyrightOnScreenFooter:

画面フッターの著作権
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Maven Archetypeで作成したプロジェクトでは、Tilesを使用して画面レイアウトを構成しているが、
画面フッター部の著作権が暫定値「\ ``Copyright &copy; 20XX CompanyName``\ 」の状態になっている。
実際のJSPの実装(サンプリング)を以下に示す。

**[template.jsp]**

.. code-block:: jsp

    <div class="container">
      <tiles:insertAttribute name="header" />
      <tiles:insertAttribute name="body" />
      <hr>
      <p style="text-align: center; background: #e5eCf9;">Copyright
        &copy; 20XX CompanyName</p>
    </div>

.. note::

    **Tilesを使用して画面レイアウトを構成する場合は、著作権に適切な値を指定すること。**

|

カスタマイズ対象のファイルとカスタマイズ方法を以下に示す。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.45\linewidth}|p{0.45\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 45 45

    * - 項番
      - 対象ファイル
      - カスタマイズ方法
    * - 1.
      - Tiles用のテンプレートJSP

        ``artifactId/artifactId-web/src/main/webapp/WEB-INF/views/layout/template.jsp``
      - 著作権の暫定値「\ ``Copyright &copy; 20XX CompanyName``\ 」を適切な値に修正する。

|

.. _CreateWebApplicationProjectCustomizeInMemoryDatabase:

インメモリデータベース(H2 Database)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Maven Archetypeで作成したプロジェクトには、インメモリデータベース(H2 Database)をセットアップするための設定が行われているが、
これはちょっとした動作検証(プロトタイプ作成やPOC(Proof Of Concept))を行うための設定である。
そのため、本格的なアプリケーション開発を行う場合は、不要な設定になる。

**[artifactId-env.xml]**

.. code-block:: xml

    <jdbc:initialize-database data-source="dataSource"
        ignore-failures="ALL">
        <jdbc:script location="classpath:/database/${database}-schema.sql" encoding="UTF-8" />
        <jdbc:script location="classpath:/database/${database}-dataload.sql" encoding="UTF-8" />
    </jdbc:initialize-database>

.. code-block:: console

        └── src
            └── main
                └── resources
                    ├── META-INF
                  (...)
                    ├── database
                    │   ├── H2-dataload.sql
                    │   └── H2-schema.sql

.. note::

    **本格的なアプリケーション開発を行う場合は、インメモリデータベース(H2 Database)をセットアップするための定義とSQLを管理するためのディレクトリを削除すること。**

|

カスタマイズ対象のファイルとカスタマイズ方法を以下に示す。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.45\linewidth}|p{0.45\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 45 45

    * - 項番
      - 対象ファイル
      - カスタマイズ方法
    * - 1.
      - 環境依存するコンポーネントを定義するBean定義ファイル

        ``artifactId-env/src/main/resources/META-INF/spring/artifactId-env.xml``
      - \ ``<jdbc:initialize-database>``\ 要素を削除する。
    * - 2.
      - インメモリデータベース(H2 Database)をセットアップするためのSQLを格納するディレクトリ

        ``artifactId/artifactId-env/src/main/resources/database/``
      - ディレクトリを削除する。

|

.. _CreateWebApplicationProjectCustomizeDataSource:

データソース設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Maven Archetypeで作成したプロジェクトでは、インメモリデータベース(H2 Database)にアクセスするためのデータソース設定が行われているが、
これはちょっとした動作検証(プロトタイプ作成やPOC(Proof Of Concept))を行うための設定である。
そのため、本格的なアプリケーション開発を行う場合は、
アプリケーション稼働時に利用するデータベースにアクセスするためのデータソース設定に変更する必要がある。

**[artifactId/artifactId-domain/pom.xml]**

.. code-block:: xml

    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>

.. note::

   上記設定例は、依存ライブラリのバージョンを親プロジェクトである terasoluna-gfw-parent で管理する前提であるため、pom.xmlでのバージョンの指定は不要である。
   上記の依存ライブラリはterasoluna-gfw-parentが利用している\ `Spring IO Platform <http://platform.spring.io/platform/>`_\ で定義済みである。

**[artifactId-infra.properties]**

.. code-block:: properties

    database=H2
    database.url=jdbc:h2:mem:todo;DB_CLOSE_DELAY=-1
    database.username=sa
    database.password=
    database.driverClassName=org.h2.Driver
    # connection pool
    cp.maxActive=96
    cp.maxIdle=16
    cp.minIdle=0
    cp.maxWait=60000

**[artifactId-env.xml]**

.. code-block:: xml

    <bean id="realDataSource" class="org.apache.commons.dbcp2.BasicDataSource"
        destroy-method="close">
        <property name="driverClassName" value="${database.driverClassName}" />
        <property name="url" value="${database.url}" />
        <property name="username" value="${database.username}" />
        <property name="password" value="${database.password}" />
        <property name="defaultAutoCommit" value="false" />
        <property name="maxTotal" value="${cp.maxActive}" />
        <property name="maxIdle" value="${cp.maxIdle}" />
        <property name="minIdle" value="${cp.minIdle}" />
        <property name="maxWaitMillis" value="${cp.maxWait}" />
    </bean>

.. note::

    **本格的なアプリケーション開発を行う場合は、アプリケーション稼働時に利用するデータベースにアクセスするためのデータソース設定に変更すること。**

    Maven Archetypeで作成したプロジェクトでは、Apache Commons DBCPを使用する設定となっているが、
    アプリケーションサーバから提供されているデータソースを使用して、
    JNDI(Java Naming and Directory Interface)経由でデータソースにアクセスする方法を採用するケースも多い。

    開発環境ではApache Commons DBCPのデータソースを使用して、
    テスト環境及び商用環境ではアプリケーションサーバから提供されているデータソースを使用するといった使い分けを行うケースもある。

    データソースの設定方法については、「:doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessCommon` の :ref:`data-access-common_howtouse_datasource`」を参照されたい。

|

カスタマイズ対象のファイルとカスタマイズ方法を以下に示す。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.45\linewidth}|p{0.45\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 45 45

    * - 項番
      - 対象ファイル
      - カスタマイズ方法
    * - 1.
      - POMファイル

        * ``artifactId/pom.xml``
        * ``artifactId/artifactId-web/pom.xml``
      - インメモリデータベース(H2 Database)のJDBCドライバを依存ライブラリから削除する。

        アプリケーション稼働時に利用するデータベースにアクセスするためのJDBCドライバを依存ライブラリに追加する。

    * - 2.
      - 環境依存する設定値を定義するプロパティファイル

        ``artifactId/artifactId-env/src/main/resources/META-INF/spring/artifactId-infra.properties``
      - データソースとしてApache Commons DBCPを使用する場合は、以下のプロパティにアプリケーション稼働時に利用するデータベースにアクセスするための接続情報を指定する。

        * ``database``
        * ``database.url``
        * ``database.username``
        * ``database.password``
        * ``database.driverClassName``

        アプリケーションサーバから提供されているデータソースを使用する場合は、以下のプロパティ以外は不要なプロパティになるので削除する。

        * ``database``

    * - 3.
      - 環境依存するコンポーネントを定義するBean定義ファイル

        ``artifactId/artifactId-env/src/main/resources/META-INF/spring/artifactId-env.xml``
      - アプリケーションサーバから提供されているデータソースを使用する場合は、JNDI経由で取得したデータソースを使用するように設定を変更する。

        データソースの設定方法については、「:doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessCommon` の :ref:`data-access-common_howtouse_datasource`」を参照されたい。

.. note:: **環境依存する設定値を定義するプロパティファイルのdatabaseプロパティについて**

    O/R MapperとしてMyBatisを使用する場合は、\ ``database``\ プロパティは不要なプロパティである。
    削除してもよいが、使用しているデータベースを明示するために設定を残しておいてもよい。

.. tip:: **JDBCドライバの追加方法について**

    使用するデータベースがPostgreSQLとOracleの場合は、POMファイル内のコメントアウトを外せばよい。
    JDBCドライバのバージョンについては、使用するデータベースのバージョンに対応するバージョンに修正すること。

    ただしOracleを使用する場合は、コメントを外す前に、
    MavenのローカルリポジトリにOracleのJDBCドライバをインストールしておく必要がある。

    以下は、PostgreSQLを使用する場合の設定例である。

    * ``artifactId/pom.xml``

     .. code-block:: xml

                         <dependency>
                             <groupId>org.postgresql</groupId>
                             <artifactId>postgresql</artifactId>
                             <version>${postgresql.version}</version>
                         </dependency>
        <!--             <dependency> -->
        <!--                 <groupId>com.oracle</groupId> -->
        <!--                 <artifactId>ojdbc7</artifactId> -->
        <!--                 <version>${ojdbc.version}</version> -->
        <!--             </dependency> -->

            <!-- ... -->

            <postgresql.version>9.4-1206-jdbc41</postgresql.version>
            <ojdbc.version>12.1.0.2</ojdbc.version>

    * ``artifactId/artifactId-web/pom.xml``

     .. code-block:: xml

                     <dependency>
                         <groupId>org.postgresql</groupId>
                         <artifactId>postgresql</artifactId>
                         <scope>runtime</scope><!-- (1) -->
                     </dependency>
        <!--         <dependency> -->
        <!--             <groupId>com.oracle</groupId> -->
        <!--             <artifactId>ojdbc7</artifactId> -->
        <!--             <scope>runtime</scope> -->
        <!--         </dependency> -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - JDBCドライバはコンパイルには使用せず、アプリケーション実行時のみ使用するため、\ ``runtime``\ スコープを指定している。

        単体テストで使用する場合などは、適切なスコープに変更して使用されたい。

|

.. _CreateWebApplicationProjectConfiguration:

開発プロジェクトの構成
--------------------------------------------------------------------------------

Maven Archetypeで作成したプロジェクトの構成について説明する。

Maven Archetypeで作成したプロジェクトは、以下の構成になっている。

* 本ガイドラインで推奨しているレイヤ毎のプロジェクト構成
* 本ガイドラインで紹介している環境依存性の排除を考慮したプロジェクト構成
* CI(Continuous Integration)を意識したプロジェクト構成

また、本ガイドラインで推奨している各種設定が盛り込まれた、

* Webアプリケーションの構成定義ファイル(web.xml)
* Spring FrameworkのBean定義ファイル
* Spring MVC用のBean定義ファイル
* Spring Security用のBean定義ファイル
* O/R Mapperの設定ファイル
* Tiles用の設定ファイル
* プロパティファイル(メッセージ定義ファイルなど)

と、アプリケーション要件との依存度が低い(=どんなアプリケーションでも作成する必要がある)コンポーネントの簡易実装として、

* Welcomeページを表示するためのControllerとJSP
* エラー画面を表示するためのJSP(HTML)
* Tiles用のテンプレートJSP
* JSPタグライブラリの読み込み設定などが定義されているインクルード用JSP
* アプリケーション全体の画面スタイルを定義するCSSファイル

などが提供されている。

.. warning:: **簡易実装として提供しているコンポーネントの扱いについて**

    簡易実装として提供しているコンポーネントは、以下のいずれかの対応を行うこと。

    * アプリケーション要件にあわせて修正
    * 不要なコンポーネントは削除

.. note:: **REST API用のプロジェクトを作成する場合の手順について**

    Maven Archetypeで作成したプロジェクトは、
    伝統的なWebアプリケーション(リクエストパラメータを受け取ってHTMLを応答するアプリケーション)を構築する際に必要となる推奨設定が行われている。

    そのため、JSONやXMLを扱うREST APIを構築する際には不要な設定が存在する。
    REST APIを構築するためのプロジェクトを作成する場合は、「:doc:`../ArchitectureInDetail/WebServiceDetail/REST` の :ref:`RESTHowToUseApplicationSettings`」を参照し、
    REST API向けの設定を適用してほしい。

.. note::

    以降の説明で\ ``artifactId``\ と表現している部分は、
    プロジェクト作成時に指定した\ ``artifactId``\ に置き換えて読み進めてほしい。

|

.. _CreateWebApplicationProjectConfigurationMulti:

マルチプロジェクトの構成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

まず、マルチプロジェクト全体の構成について説明する。

.. code-block:: console

    artifactId
        ├── pom.xml  ... (1)
        ├── artifactId-web  ... (2)
        ├── artifactId-domain  ... (3)
        ├── artifactId-env  ... (4)
        ├── artifactId-initdb  ... (5)
        └── artifactId-selenium  ... (6)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - | 項番
      - | 説明
    * - | (1)
      - マルチプロジェクト全体の構成を定義するPOM(Project Object Model)ファイル。

        このファイルでは、主に以下の定義を行う。

        * 依存ライブラリのバージョン
        * ビルド用のプラグインの設定(ビルド方法の設定)

        マルチプロジェクトの階層関係については、「:ref:`CreateWebApplicationProjectAppendixProjectHierarchicalStructure`」を参照されたい。

    * - | (2)
      - アプリケーション層(Web層)のコンポーネントを管理するモジュール。

        このモジュールでは、主に以下のコンポーネントやファイルを管理する。

        * Controllerクラス
        * 相関チェック用のValidatorクラス
        * Formクラス(REST APIの場合はResourceクラス)
        * View(JSP)
        * CSSファイル
        * JavaScriptファイル
        * アプリケーション層のコンポーネント用のJUnit
        * アプリケーション層のコンポーネントを定義するためのBean定義ファイル
        * Webアプリケーションの構成定義ファイル(web.xml)
        * メッセージ定義ファイル

    * - | (3)
      - ドメイン層のコンポーネントを管理するモジュール。

        このモジュールでは、主に以下のコンポーネントやファイルを管理する。

        * Entityなどのドメインオブジェクト
        * Repository
        * Service
        * DTO
        * ドメイン層のコンポーネント用のJUnit
        * ドメイン層のコンポーネントを定義するためのBean定義ファイル

    * - | (4)
      - 環境依存性をもつ設定ファイルを管理するモジュール。

        このモジュールでは、主に以下のファイルを管理する。

        * 環境依存するコンポーネントを定義するためのBean定義ファイル
        * 環境依存するプロパティ値を定義するプロパティファイル

    * - | (5)
      - データベースを初期化するためのSQLファイルを管理するモジュール

        このモジュールでは、主に以下のファイルを管理する。

        * テーブルなどのデータベースオブジェクトを作成するためのSQLファイル
        * マスタデータなどの初期データを投入するためのSQLファイル
        * E2E(End To End)テストで使用するテストデータを投入するためのSQLファイル

    * - | (6)
      - Seleniumを使用したE2Eテスト用のコンポーネントを管理するモジュール。

        このモジュールでは、主に以下のファイルを管理する。

        * Seleniumを操作してテストを行うJUnit
        * Assert時に使用する期待値ファイル(必要に応じて)

.. raw:: latex

   \newpage

.. note:: **本ガイドラインにおける「マルチプロジェクト」の用語定義について**

    Maven Archetypeで作成したプロジェクトは、正確にはマルチモジュール構成のプロジェクトとなる。

    本ガイドラインでは、マルチモジュールとマルチプロジェクトを同じ意味で使用していることを補足しておく。


.. note:: **２つのWebアプリケーションと１つの共通ライブラリが必要となる開発プロジェクトについて**

    * bar-parent
    * bar-initdb
    * bar-common
    * bar-common-web
    * bar-domain-a
    * bar-domain-b
    * bar-web-a
    * bar-web-b
    * bar-env
    * bar-web-a-selenium
    * bar-web-b-selenium
    
    それぞれのプロジェクトの内容は下記のようになる。
    
    * bar-parent
    
      parent-pom（親POM）と呼ばれるプロジェクト。pom.xmlファイルだけを持ち、
      その他のソースコードや設定ファイルは一切持たない、シンプルなプロジェクト。
      他のプロジェクトのpom上で、このbar-parentプロジェクトを<parent>タグに指定することによって、
      親POMに指定された共通設定情報を自身に反映させることができる。
    
    * bar-initdb
    
     RDBMSのテーブル定義(DDL)と初期データをINSERTするためのSQL文を格納する。
     これもmavenプロジェクトとして管理する。pom.xmlに `sql-maven-plugin <http://www.mojohaus.org/sql-maven-plugin/>`_ 
     の設定を定義することにより、ビルドライフサイクルの過程で任意のRDBMSに対するDDL文や初期データINSERT文の実行を自動化することができる。
    
    * bar-common
    
      プロジェクト共通ライブラリを格納する。ここはweb非依存にし、webに関わるクラスはbar-common-webに配置する。
    
    * bar-common-web
    
      プロジェクト共通webライブラリを格納する
    
    * bar-domain-a
    
      aドメインに関わるドメイン層のjavaクラス、単体テストケース等を格納するプロジェクト。最終的に*.jarファイル化する。
    
    * bar-domain-b
    
      bドメインに関わるドメイン層のクラス。
    
    * bar-web-a
    
      アプリケーション層のjavaクラス、jsp、設定ファイル、単体テストケース等を格納するプロジェクト。最終的にWebアプリケーションとして*.warファイル化する。
      bar-web-aは、bar-commonとbar-envへの依存性を持つ。
    
    * bar-web-b
    
      もう一つのサブシステムとしてのWebアプリケーション。構造はbar-web-aと同じ。
    
    * bar-env
    
      環境依存性のある設定ファイルだけを集めるプロジェクト。
    
    * bar-web-a-selenium
    
      web-aプロジェクトのための、`Selenium WebDriver <http://seleniumhq.org/projects/webdriver/>`_ によるテストケースを格納するプロジェクト。
    
    * bar-web-b-selenium
    
      web-bプロジェクトのための、`Selenium WebDriver <http://seleniumhq.org/projects/webdriver/>`_ によるテストケースを格納するプロジェクト。


.. _CreateWebApplicationProjectConfigurationWeb:

webモジュールの構成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

アプリケーション層(Web層)のコンポーネントを管理するモジュールの構成について説明する。

.. code-block:: console

    artifactId-web
        ├── pom.xml  ... (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (1)
      - webモジュールの構成を定義するPOM(Project Object Model)ファイル。
        このファイルでは、以下の定義を行う。

        * 依存ライブラリとビルド用プラグインの定義
        * warファイルを作成するための定義

.. note:: **REST API用のプロジェクトを作成する際のwebモジュールのモジュール名について**

    REST APIを構築する場合は、モジュール名を\ ``artifactId-api``\といった感じの名前にしておくと、
    アプリケーションの種類が識別しやすくなる。

|

.. code-block:: console

        └── src
            ├── main
            │   ├── java
            │   │   └── com
            │   │       └── example
            │   │           └── project
            │   │               └── app  ... (2)
            │   │                   └── welcome
            │   │                       └── HelloController.java  ... (3)
            │   ├── resources
            │   │   ├── META-INF
            │   │   │   ├── dozer  ... (4)
            │   │   │   └── spring  ... (5)
            │   │   │       ├── application.properties  ... (6)
            │   │   │       ├── applicationContext.xml  ... (7)
            │   │   │       ├── spring-mvc.xml  ... (8)
            │   │   │       └── spring-security.xml  ... (9)
            │   │   └── i18n  ... (10)
            │   │       └── application-messages.properties  ... (11)

.. raw:: latex

   \newpage

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - | 項番
      - | 説明
    * - | (2)
      - アプリケーション層のクラスを格納するためのパッケージ。

        REST APIを構築する場合は、パッケージ名を\ ``api``\ といった感じの名前にしておくと、
        コンポーネントの種類が識別しやすくなる。
    * - | (3)
      - Welcomeページを表示するためのリクエストを受け取るためのControllerクラス。
    * - | (4)
      - Dozer(Bean Mapper)のマッピング定義ファイルを格納するディレクトリ。
        Dozerについては、「:doc:`../ArchitectureInDetail/GeneralFuncDetail/Dozer`」を参照されたい。

        作成時点では空のディレクトリである。
        マッピングファイルが必要になった場合(高度なマッピングが必要になった場合)は、
        このディレクトリ配下に格納すると、自動的にマッピングファイルが読み込まれる。

        .. note::

            このディレクトリには、以下のファイルを格納する。

            * アプリケーション層のJavaBeanとドメイン層のJavaBeanをマッピングするための定義ファイル
            * アプリケーション層のJavaBean同士をマッピングするための定義ファイル

            ドメイン層のJavaBean同士のマッピングはドメイン層のディレクトリに格納することを推奨している。

    * - | (5)
      - Spring FrameworkのBean定義ファイルとプロパティファイルを格納するディレクトリ。
    * - | (6)
      - アプリケーション層で使用する設定値を定義するプロパティファイル。

        作成時点では、空のファイルである。
    * - | (7)
      - Webアプリケーション用のアプリケーションコンテキストを作成するためのBean定義ファイル。

        このファイルには、以下のBeanを定義する。

        * Webアプリケーション全体で使用するコンポーネント
        * ドメイン層のコンポーネント(ドメイン層のコンポーネントが定義されているBean定義ファイルをimportする)

    * - | (8)
      - \ ``DispatcherServlet``\ 用のアプリケーションコンテキストを作成するためのBean定義ファイル。

        このファイルには、以下のBeanを定義する。

        * Spring MVCのコンポーネント
        * アプリケーション層のコンポーネント

        REST APIを構築する場合は、ファイル名を\ ``spring-mvc-api.xml``\ といった感じの名前にしておくと、 アプリケーションの種類が識別しやすくなる。

    * - | (9)
      - Spring Securityのコンポーネントを定義するためのBean定義ファイル。

        このファイルは、Webアプリケーション用のアプリケーションコンテキストを作成する際に読み込む。
    * - | (10)
      - アプリケーション層で使用するメッセージ定義ファイルを格納するディレクトリ。
    * - | (11)
      - アプリケーション層で使用するメッセージを定義するプロパティファイル。

        作成時点では、いくつかの汎用的なメッセージが定義されている。

        .. note::

            **メッセージについては、アプリケーションの要件(メッセージ規約など)にあわせて必ず修正すること。**
            メッセージ定義については、「:doc:`../ArchitectureInDetail/WebApplicationDetail/MessageManagement`」を参照されたい。

.. raw:: latex

   \newpage

.. note::

    アプリケーションコンテキストとBean定義ファイルの関連については、
    「:ref:`CreateWebApplicationProjectAppendixApplicationContext`」を参照されたい。

|

.. code-block:: console

            │   └── webapp
            │       ├── WEB-INF
            │       │   ├── tiles  ... (12)
            │       │   │   └── tiles-definitions.xml
            │       │   ├── views  ... (13)
            │       │   │   ├── common
            │       │   │   │   ├── error  ... (14)
            │       │   │   │   │   ├── accessDeniedError.jsp
            │       │   │   │   │   ├── businessError.jsp
            │       │   │   │   │   ├── dataAccessError.jsp
            │       │   │   │   │   ├── invalidCsrfTokenError.jsp
            │       │   │   │   │   ├── missingCsrfTokenError.jsp
            │       │   │   │   │   ├── resourceNotFoundError.jsp
            │       │   │   │   │   ├── systemError.jsp
            │       │   │   │   │   ├── transactionTokenError.jsp
            │       │   │   │   │   └── unhandledSystemError.html
            │       │   │   │   └── include.jsp  ... (15)
            │       │   │   ├── layout  ... (16)
            │       │   │   │   ├── header.jsp
            │       │   │   │   └── template.jsp
            │       │   │   └── welcome
            │       │   │       └── home.jsp  ... (17)
            │       │   └── web.xml  ... (18)
            │       └── resources  ... (19)
            │           └── app
            │               └── css
            │                   └── styles.css  ... (20)
            └── test
                ├── java
                └── resources

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - | 項番
      - | 説明
    * - | (12)
      - Tilesの設定ファイルを格納するディレクトリ。
        Tilesの設定ファイルについては、「:doc:`../ArchitectureInDetail/WebApplicationDetail/TilesLayout`」を参照されたい。
    * - | (13)
      - Viewを構築するテンプレートファイル(JSPなど)を格納するディレクトリ。
    * - | (14)
      - エラー画面を表示するためのJSP及びHTMLを格納するディレクトリ。

        作成時点では、アプリケーション実行時に発生する可能性があるエラーに対応するJSP(HTML)が格納されている。

        .. note::

            **エラー画面用のJSP及びHTMLについては、アプリケーションの要件(UI規約など)にあわせて必ず修正すること。**

    * - | (15)
      - インクルード用の共通JSPファイル。


        このファイルは、全てのJSPファイルの先頭にインクルードされる。
        インクルード用の共通JSPファイルについては、「:ref:`view_jsp_include-label`」を参照されたい。
    * - | (16)
      - Tilesのレイアウト用のJSPファイルを格納するディレクトリ。
        Tilesのレイアウト用のJSPファイルについては、「:doc:`../ArchitectureInDetail/WebApplicationDetail/TilesLayout`」を参照されたい。
    * - | (17)
      - Welcomeページを表示するJSPファイル。
    * - | (18)
      - Webアプリケーションの構成定義ファイル。
    * - | (19)
      - 静的なリソースファイルを格納するディレクトリ。

        このディレクトリは、リクエストの内容によって応答する内容がかわらないファイルを格納する。
        具体的には以下のファイルを格納する。

        * JavaScriptファイル
        * CSSファイル
        * 画像ファイル
        * HTMLファイル

        Spring MVCが提供する静的リソースの管理メカニズムを適用しやすくするために、
        専用のディレクトリを設ける構成を採用している。
    * - | (20)
      - アプリケーション全体に適用する画面スタイルを定義するCSSファイル。

.. raw:: latex

   \newpage

|

.. _CreateWebApplicationProjectConfigurationDomain:

domainモジュールの構成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ドメイン層のコンポーネントを管理するモジュールの構成について説明する。

.. code-block:: console

    artifactId-domain
        ├── pom.xml  ... (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (1)
      - domainモジュールの構成を定義するPOM(Project Object Model)ファイル。
        このファイルでは、以下の定義を行う。

        * 依存ライブラリとビルド用プラグインの定義
        * jarファイルを作成するための定義

|

.. code-block:: console

        └── src
            ├── main
            │   ├── java
            │   │   └── com
            │   │       └── example
            │   │           └── project
            │   │               └── domain  ... (2)
            │   │                   ├── model
            │   │                   ├── repository
            │   │                   └── service
            │   └── resources
            │       └── META-INF
            │           ├── dozer  ... (3)
            │           └── spring  ... (4)
            │               ├── artifactId-codelist.xml  ... (5)
            │               ├── artifactId-domain.xml  ... (6)
            │               └── artifactId-infra.xml  ... (7)


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (2)
      - ドメイン層のクラスを格納するためのパッケージ。
    * - | (3)
      - Dozer(Bean Mapper)のマッピング定義ファイルを格納するディレクトリ。
        Dozerについては、「:doc:`../ArchitectureInDetail/GeneralFuncDetail/Dozer`」を参照されたい。

        作成時点では空のディレクトリである。
        マッピングファイルが必要になった場合(高度なマッピングが必要になった場合)は、
        このディレクトリ配下に格納すると、自動的にマッピングファイルが読み込まれる。

        .. note::

            このディレクトリには、以下のファイルを格納する。

            * ドメイン層のJavaBean同士をマッピングするための定義ファイル

    * - | (4)
      - Spring FrameworkのBean定義ファイルとプロパティファイルを格納するディレクトリ。
    * - | (5)
      - コードリストを定義するためのBean定義ファイル。
    * - | (6)
      - ドメイン層のコンポーネントを定義するためのBean定義ファイル。

        このファイルには、以下のBeanを定義する。

        * ドメイン層のコンポーネント(Service, Repositoryなど)
        * インフラストラクチャ層のコンポーネント(インフラストラクチャ層のコンポーネントが定義されているBean定義ファイルをimportする)
        * Spring Frameworkから提供されているトランザクション管理用のコンポーネント

    * - | (7)
      - インフラストラクチャ層のコンポーネントを定義するためのBean定義ファイル。

        このファイルには、O/R MapperなどのBean定義を行う。

|

.. code-block:: console

            └── test
                ├── java
                │   └── com
                │       └── example
                │           └── project
                │               └── domain
                │                   ├── repository
                │                   └── service
                └── resources
                    └── test-context.xml  ... (8)


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (8)
      - ドメイン層のユニットテスト用のコンポーネントを定義するためのBean定義ファイル。

|

**MyBatis3用のプロジェクトを作成した場合**

.. code-block:: console

        └── src
            ├── main
            │   ├── java
           (...)
            │   └── resources
            │       ├── META-INF
            │       │   ├── dozer
            │       │   ├── mybatis  ... (9)
            │       │   │   └── mybatis-config.xml  ... (10)
            │       │   └── spring
           (...)
            │       └── com
            │           └── example
            │               └── project
            │                   └── domain
            │                       └── repository  ... (11)
            │                           └── sample
            │                               └── SampleRepository.xml  ... (12)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (9)
      - MyBatis3の設定ファイルを格納するディレクトリ。
    * - | (10)
      - MyBatis3の設定ファイル。

        作成時点では、いくつかの推奨設定が定義されている。
    * - | (11)
      - MyBatis3のMapperファイルを格納するディレクトリ。
    * - | (12)
      - MyBatis3のMapperファイルのサンプルファイル。

        作成時点では、サンプル実装がコメントアウトされた状態になっている。
        **このファイルは最終的には不要なファイルである。**

|

.. _CreateWebApplicationProjectConfigurationEnv:

envモジュールの構成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

環境依存性をもつ設定ファイルを管理するモジュールの構成について説明する。

.. code-block:: console

    artifactId-env
        ├── configs  ... (1)
        │   ├── production-server  ... (2)
        │   │   └── resources
        │   └── test-server
        │       └── resources
        ├── pom.xml  ... (3)


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (1)
      - 環境依存する設定ファイルを管理するためのディレクトリ。

        環境毎にサブディレクトリを作成し、環境依存する設定ファイルを管理する。
    * - | (2)
      - 環境毎の設定ファイルを管理するためのディレクトリ。

        作成時点では、最もシンプルな構成として、以下のディレクトリ(雛形のディレクトリ)が用意されている。

        * production-server (商用環境向けの設定ファイルを格納するディレクトリ)
        * test-server (テスト環境向けの設定ファイルを格納するディレクトリ)

    * - | (3)
      - envモジュールの構成を定義するPOM(Project Object Model)ファイル。
        このファイルでは、以下の定義を行う。

        * 依存ライブラリとビルド用プラグインの定義
        * 環境毎のjarファイルを作成するためのProfileの定義

|

.. code-block:: console

        └── src
            └── main
                └── resources  ... (4)
                    ├── META-INF
                    │   └── spring
                    │       ├── artifactId-env.xml  ... (5)
                    │       └── artifactId-infra.properties  ... (6)
                    ├── database  ... (7)
                    │   ├── H2-dataload.sql
                    │   └── H2-schema.sql
                    ├── dozer.properties  ... (8)
                    ├── log4jdbc.properties  ... (9)
                    └── logback.xml  ... (10)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (4)
      - 開発用の設定ファイルを管理するためのディレクトリ。
    * - | (5)
      - 環境依存するコンポーネントを定義するBean定義ファイル。

        このファイルには、以下のBeanを定義する。

        * データソース
        * 共通ライブラリから提供している\ ``JodaTimeDateFactory``\ (環境によって異なる実装を使用する場合)
        * Spring Frameworkから提供されているトランザクション管理用のコンポーネント (環境によって異なる実装を使用する場合)

    * - | (6)
      - 環境依存する設定値を定義するプロパティファイル。

        作成時点では、データソースの設定値(接続情報とコネクションプールの設定値)が定義されている。
    * - | (7)
      - インメモリデータベース(H2 Database)をセットアップするためのSQLを格納するディレクトリ。

        このディレクトリは、ちょっとした動作検証を行う時のために用意しているディレクトリである。
        **実際のアプリケーション開発で使用することは想定していないので、基本的にはこのディレクトリは削除すること。**
    * - | (8)
      - Dozer(Bean Mapper)のグローバル設定を行うためのプロパティファイル。
        Dozerについては、「:doc:`../ArchitectureInDetail/GeneralFuncDetail/Dozer`」を参照されたい。

        作成時点では、空のファイルである。(ファイルがないと起動時に警告ログが出力されるため、これを防ぐために空のファイルを用意している)
    * - | (9)
      - Log4jdbc-remix(JDBC関連のログ出力を行うライブラリ)のグローバル設定を行うためのプロパティファイル。
        Log4jdbc-remixについては、「:ref:`DataAccessCommonDataSourceDebug`」を参照されたい。

        作成時点では、ログに出力するSQLの改行に関する設定のみ指定されている。
    * - | (10)
      - Logback(ログ出力)の設定ファイル。
        ログ出力については、「:doc:`../ArchitectureInDetail/GeneralFuncDetail/Logging`」を参照されたい。

|

.. _CreateWebApplicationProjectConfigurationInitdb:

initdbモジュールの構成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

データベースを初期化するためのSQLファイルを管理するモジュールの構成について説明する。

.. code-block:: console

    artifactId-initdb
        ├── pom.xml  ... (1)
        └── src
            └── main
                └── sqls  ... (2)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (1)
      - initdbモジュールの構成を定義するPOM(Project Object Model)ファイル。
        このファイルでは、以下の定義を行う。

        * ビルド用プラグイン(`SQL Maven Plugin <http://www.mojohaus.org/sql-maven-plugin/>`_)の定義

        作成時点では、PostgreSQL用の雛形設定が定義されている。
    * - | (2)
      - データベースを初期化するためのSQLファイルを格納するためのディレクトリ。

        作成時点では、空のディレクトリである。
        作成例については、`サンプルアプリケーションのinitdbプロジェクト <https://github.com/terasolunaorg/terasoluna-tourreservation-mybatis3/tree/5.3.0.RELEASE/terasoluna-tourreservation-initdb/src/main/sqls>`_ を参照されたい。

.. note::

    `SQL Maven Plugin <http://www.mojohaus.org/sql-maven-plugin/>`_ の `sql:execute <http://www.mojohaus.org/sql-maven-plugin/execute-mojo.html>`_ を使用して、SQLを実行できる。

        .. code-block:: console

            mvn sql:execute

|

.. _CreateWebApplicationProjectConfigurationSelenium:

seleniumモジュールの構成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Seleniumを使用したE2E(End To End)テスト用のコンポーネントを管理するモジュールの構成について説明する。

.. code-block:: console

    artifactId-selenium
        ├── pom.xml  ... (1)
        └── src
            └── test  ... (2)
                ├── java
                │   └── com
                │       └── example
                │           └── project
                │               └── selenium
                │                   └── welcome
                │                       └── HelloTest.java  ... (3)
                └── resources
                    └── META-INF
                        └── spring
                            ├── selenium.properties  ... (4)
                            └── seleniumContext.xml  ... (5)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (1)
      - seleniumモジュールの構成を定義するPOM(Project Object Model)ファイル。

        このファイルでは、以下の定義を行う。

        * 依存ライブラリとビルド用プラグインの定義
        * jarファイルを作成するための定義

    * - | (2)
      - テスト用のコンポーネントと設定ファイルを格納するディレクトリ。

        作成例については、`サンプルアプリケーションのseleniumプロジェクト <https://github.com/terasolunaorg/terasoluna-tourreservation-mybatis3/tree/5.3.0.RELEASE/terasoluna-tourreservation-selenium>`_ を参照されたい。

    * - | (3)
      - Selenium WebDriverを使用したサンプルテストクラス。

        作成時点では、Welcomeページのタイトルを検証するテストケースが実装されている。

    * - | (4)
      - テストで使用する設定値を定義するプロパティファイル。

        作成時点では、アプリケーションサーバのURLは\ ``http://localhost:8080/``\ である。

    * - | (5)
      - テスト用のコンポーネントを定義するためのBean定義ファイル。

        作成時点では、サンプルのテストを実行するために必要な設定がされている。

|

.. _CreateWebApplicationProjectAppendix:

Appendix
--------------------------------------------------------------------------------

.. _CreateWebApplicationProjectAppendixProjectHierarchicalStructure:

プロジェクトの階層構造
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Maven Archetypeで作成したプロジェクトのプロジェクト階層の構造を以下に示す。

.. figure:: images_CreateWebApplicationProject/CreateWebApplicationProjectHierarchicalStructure.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - | 項番
      - | 説明
    * - | (1)
      - Maven Archetypeで作成したプロジェクト。

        Maven Archetypeで作成したプロジェクトはマルチモジュール構成となっており、
        親プロジェクトと各サブモジュールは相互参照の関係になっている。

        version 5.3.0.RELEASE用のMaven Archetypeで作成したプロジェクトでは、
        親プロジェクトとして「org.terasoluna.gfw:terasoluna-gfw-parent:5.3.0.RELEASE」を指定している。
    * - | (2)
      - TERASOLUNA Server Framework for Java (5.x) Parentプロジェクト。

        TERASOLUNA Server Framework for Java (5.x) Parentプロジェクトでは、

        * ビルド用のプラグインの設定
        * Spring IO Platform経由で管理されているライブラリのカスタマイズ(バージョンの調整)
        * Spring IO Platformで管理されていない推奨ライブラリのバージョン管理

        を行っている。

        なお、Spring IO Platform経由で依存ライブラリのバージョンを管理するために、本プロジェクトの\ ``<dependencyManagement>``\ に「Spring IO Platform」をインポートしている。
        
        利用しているSpring IO Platformのバージョンは\ :ref:`frameworkstack_using_oss_version`\参照のこと。
    * - | (3)
      - Spring IO Platformプロジェクト。

        親プロジェクトとして「org.springframework.boot:spring-boot-starter-parent:1.2.5.RELEASE」が指定されているため、spring-boot-starter-parentのpomファイルに定義されている\ ``<dependencyManagement>``\ の定義も、terasoluna-gfw-parentのpomファイルにインポートされる。
    * - | (4)
      - Spring Boot Starter Parentプロジェクト。

        親プロジェクトとして「org.springframework.boot:spring-boot-dependencies:1.2.5.RELEASE」が指定されているため、spring-boot-dependenciesのpomファイルに定義されている\ ``<dependencyManagement>``\の定義も、terasoluna-gfw-parentのpomファイルにインポートされる。
    * - | (5)
      - Spring Boot Dependenciesプロジェクト。

.. raw:: latex

   \newpage

.. tip::

    version 5.0.0.RELEASEより、Spring IO Platformの\ ``<dependencyManagement>``\ をインポートする構成に変更しており、
    推奨ライブラリのバージョン管理をSpring IO Platformに委譲するスタイルを採用している。

.. warning::

    version 5.0.0.RELEASEより、Spring IO Platformの\ ``<dependencyManagement>``\ をインポートする構成に変更したため、
    子プロジェクトからライブラリのバージョンを管理するためのプロパティにアクセスする事が出来なくなっている。

    そのため、子プロジェクト側でプロパティ値を参照又は上書きしている場合は、version 1.0.xからバージョンアップする際にpomファイルの修正が必要になる。

    なお、Spring IO Platformで管理していない推奨ライブラリ(TERASOLUNA Server Framework for Java (5.x)独自の推奨ライブラリ)については、
    従来通りバージョンを管理するためのプロパティにアクセスする事ができる。


|

.. _CreateWebApplicationProjectAppendixApplicationContext:

アプリケーションコンテキストの構成とBean定義ファイルの関係
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Frameworkのアプリケーションコンテキスト(DIコンテナ)の構成とBean定義ファイルの関係を以下に示す。

.. figure:: images_CreateWebApplicationProject/CreateWebApplicationProjectApplicationContext.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - | 項番
      - | 説明
    * - | (1)
      - Webアプリケーション用のアプリケーションコンテキスト。

        上記図で示す通り、

        * artifactId-web/src/main/resource/META-INF/spring/applicationContext.xml
        * artifactId-domain/src/main/resource/META-INF/spring/artifactId-domain.xml
        * artifactId-domain/src/main/resource/META-INF/spring/artifactId-infra.xml
        * artifactId-env/src/main/resource/META-INF/spring/artifactId-env.xml
        * artifactId-domain/src/main/resource/META-INF/spring/artifactId-codelist.xml
        * artifactId-web/src/main/resource/META-INF/spring/spring-security.xml

        で定義したコンポーネントがWebアプリケーション用のアプリケーションコンテキスト(DIコンテナ)に登録される。

        Webアプリケーション用のアプリケーションコンテキストに登録されているコンポーネントは、
        各\ ``DispatcherServlet``\ 用のアプリケーションコンテキストから参照する事ができる仕組みとなっている。
    * - | (2)
      - \ ``DispatcherServlet``\ 用のアプリケーションコンテキスト。

        上記図で示す通り、

        * artifactId-web/src/main/resource/META-INF/spring/spring-mvc.xml

        で定義したコンポーネントが\ ``DispatcherServlet``\ 用のアプリケーションコンテキスト(DIコンテナ)に登録される。

        \ ``DispatcherServlet``\ 用のアプリケーションコンテキストに存在しないコンポーネントは、
        Webアプリケーション用のアプリケーションコンテキスト(親コンテキスト)を参照して取得する仕組みになっているため、
        ドメイン層のコンポーネントをアプリケーション層のコンポーネントに対してインジェクションする事ができる。

.. raw:: latex

   \newpage

.. note:: **同じコンポーネントを両方のアプリケーションコンテキストに登録した時の動作について**

    Webアプリケーション用のアプリケーションコンテキストと\ ``DispatcherServlet``\ 用のアプリケーションコンテキストの両方に同じコンポーネントが登録されている場合は、
    同じアプリケーションコンテキスト(\ ``DispatcherServlet``\ 用のアプリケーションコンテキスト)内に登録されているコンポーネントがインジェクションされる点を補足しておく。

    特に、ドメイン層のコンポーネント(ServiceやRepositoryなど)を\ ``DispatcherServlet``\ 用のアプリケーションコンテキストに登録しないように注意する必要である。

    ドメイン層のコンポーネントを\ ``DispatcherServlet``\ 用のアプリケーションコンテキストに登録してしまうと、
    トランザクション制御を行うコンポーネント(AOP)が有効にならないため、データベースへの操作がコミットされない不具合が発生してしまう。

    なお、Maven Archetypeで作成したプロジェクトでは、上記のような現象は発生しないように設定が行われている。
    設定の追加又は変更を行う場合は、注意してほしい。

|

.. _CreateWebApplicationProjectAppendixDescribeConfigurationFile:

設定ファイルの解説
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo::

    各種設定が意味することの理解度を高めるために、設定ファイルの解説を追加する予定である。

    * 機能詳細に説明があるものについては、機能詳細への参照を記載する。
    * 機能詳細に記載がないものについては、ここに説明を記載する。

    具体的な対応時期は未定。

|

オフライン環境におけるアプリケーション開発
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

「:ref:`CreateWebApplicationProject`」では、
マルチプロジェクト構成の開発プロジェクトを、
`Maven Archetype Plugin <http://maven.apache.org/archetype/maven-archetype-plugin/>`_ の 
`archetype:generate <http://maven.apache.org/archetype/maven-archetype-plugin/generate-mojo.html>`_ を使用して作成する方法について述べた。
Mavenはオンライン環境での動作が前提であるが、
以下にオフライン環境でも使用できるようにする方法について述べる。

オフライン環境でプロジェクト開発を続けるためには、
開発に必要となるライブラリやプラグイン等のファイルを事前にコピーする必要がある。
以下の作業は **オンライン環境** で行うこと。

|

開発プロジェクトのルートディレクトリへ移動する。
ここでは「:ref:`CreateWebApplicationProject`」で作成したプロジェクトを例に説明をする。

.. code-block:: console

    cd C:\work\todo

|

プロジェクト開発に必要であるライブラリやプラグイン等のファイルをコピーする。
`Maven Archetype Plugin <http://maven.apache.org/archetype/maven-archetype-plugin/>`_ の 
`dependency:go-offline <https://maven.apache.org/plugins/maven-dependency-plugin/go-offline-mojo.html>`_ を実行することでコピーする。

.. code-block:: console

    mvn dependency:go-offline -Dmaven.repo.local=repository

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 25 75

    * - パラメータ
      - 説明
    * - | \--Dmaven.repo.local
      - コピー先を指定する。
        コピー先が存在しない場合は新たに作成される。
        今回はコピー先をrepositoryと指定している。

|

成果物を配布しやすくするために、warファイルまたはjarファイルを作成する。
この時、ビルドに必要となるライブラリやプラグイン等のファイルがコピーされる。

.. code-block:: console

    mvn package -Dmaven.repo.local=repository

|

ビルドが成功した場合、以下のようなログが出力される。

.. code-block:: console

	(... omit)    
	[INFO] ------------------------------------------------------------------------
	[INFO] Reactor Summary:
	[INFO]
	[INFO] TERASOLUNA Server Framework for Java (5.x) Web Blank Multi Project (MyBa
	tis3) SUCCESS [  0.006 s]
	[INFO] todo-env ........................................... SUCCESS [ 46.565 s]
	[INFO] todo-domain ........................................ SUCCESS [  0.684 s]
	[INFO] todo-web ........................................... SUCCESS [ 12.832 s]
	[INFO] todo-initdb ........................................ SUCCESS [  0.067 s]
	[INFO] todo-selenium ...................................... SUCCESS [01:13 min]
	[INFO] ------------------------------------------------------------------------
	[INFO] BUILD SUCCESS
	[INFO] ------------------------------------------------------------------------
	[INFO] Total time: 02:14 min
	[INFO] Finished at: 2015-10-01T10:32:34+09:00
	[INFO] Final Memory: 36M/206M
	[INFO] ------------------------------------------------------------------------

|

以上で、プロジェクト開発に必要なライブラリやプラグイン等のファイルをコピーした。
このrepositoryをオフライン環境マシンの${HOME}/.m2へコピーすることで、作業は完了となる。
オンライン環境で一度も実行していない処理をオフライン環境で実行すると、
必要なライブラリやプラグイン等のファイルを取得できず処理に失敗するが、
コピーを行ったことにより、オフライン環境へ移行した場合においても継続して開発を進めることが可能となる。

.. warning:: **オフライン環境での開発における注意点**

    オフライン環境では新規に依存関係をインターネットから取得することが不可能となるため、
    POM（Project Object Model）ファイルを編集しないこと。
    POMファイルに編集を加える場合は、再度オンライン環境へ戻る必要がある。

.. raw:: latex

   \newpage
