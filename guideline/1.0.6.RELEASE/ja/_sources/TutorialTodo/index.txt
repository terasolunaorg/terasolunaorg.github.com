チュートリアル(Todoアプリケーション)
********************************************************************************

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

はじめに
================================================================================

このチュートリアルで学ぶこと
--------------------------------------------------------------------------------

* TERASOLUNA Global Frameworkによる基本的なアプリケーションの開発方法およびEclipseプロジェクトの構築方法
* TERASOLUNA Global Frameworkの :doc:`../Overview/ApplicationLayering` に従った開発方法


対象読者
--------------------------------------------------------------------------------

* SpringのDIやAOPに関する基礎的な知識がある
* Servlet/JSPを使用してWebアプリケーションを開発したことがある
* SQLに関する知識がある


検証環境
--------------------------------------------------------------------------------

このチュートリアルは以下の環境で動作確認している。他の環境で実施する際は本書をベースに適宜読み替えて設定していくこと。

 .. tabularcolumns:: |p{0.15\linewidth}|p{0.75\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 15 75

    * - 種別
      - 名前
    * - OS
      - Windows7 64bit
    * - JVM
      - Java 1.7
    * - IDE
      - Spring Tool Suite Version: 3.5.0.RELEASE Build Id: 201404011654 (以下STS) Build Maven 3.0.4 (STS付属)
    * - Application Server
      - VMWare vFabric tc Server Developer Edition v2.9 (STS付属)
    * - Web Browser
      - Google Chrome 27.0.1453.94 m

|

作成するアプリケーションの説明
================================================================================

アプリケーションの概要
--------------------------------------------------------------------------------

TODOを管理するアプリケーションを作成する。TODOの一覧表示、TODOの登録、TODOの完了、TODOの削除を行える。


 .. figure:: ./images/image001.png
   :width: 60%


.. _app-requirement:

アプリケーションの業務要件
--------------------------------------------------------------------------------
アプリケーションの業務要件は、以下の通りとする。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 80

    * - ルールID
      - 説明
    * - B01
      - 未完のTODOは5件までしか登録できない
    * - B02
      - 完了済みのTODOは完了できない

 .. note::

     本要件は学習のためのもので、現実的なTODO管理アプリケーションとしては適切ではない。

|

アプリケーションの処理仕様
--------------------------------------------------------------------------------
アプリケーションの処理仕様と画面遷移は、以下の通りとする。

 .. figure:: ./images/image002.png
   :width: 60%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|p{0.40\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 15 15 40

    * - 項番
      - プロセス名
      - HTTPメソッド
      - URL
      - 説明
    * - 1
      - Show all TODO
      - GET
      - /todo/list
      -
    * - 2
      - Create TODO
      - POST
      - /todo/create
      - 作成完了後1へリダイレクト
    * - 3
      - Finish TODO
      - POST
      - /todo/finish
      - 作成完了後1へリダイレクト
    * - 4
      - Delete TODO
      - POST
      - /todo/delete
      - 作成完了後1へリダイレクト

Show all TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* TODOを全件表示する
* 未完了のTODOに対しては”Finish”と”Delete”用のボタンが付く
* 完了のTODOは打ち消し線で装飾する
* TODOの件名のみ


Create TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* フォームから送信されたTODOを保存する
* TODOの件名は1文字以上30文字以下であること
* :ref:`app-requirement` のB01を満たさない場合はエラーコードE001でビジネス例外をスローする

Finish TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* フォームから送信されたtodoIdに対応するTODOを完了済みにする
* :ref:`app-requirement` のB02を満たさない場合はエラーコードE002でビジネス例外をスローする
* 該当するTODOが存在しない場合はエラーコードE404でビジネス例外をスローする

Delete TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* フォームから送信されたtodoIdに対応するTODOを削除する
* 該当するTODOが存在しない場合はエラーコードE404でビジネス例外をスローする

|

エラーメッセージ一覧
--------------------------------------------------------------------------------

エラーメッセージとして、以下の3つを定義する。

 .. tabularcolumns:: |p{0.15\linewidth}|p{0.45\linewidth}|p{0.30\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 15 45 30

    * - エラーコード
      - メッセージ
      - 置換パラメータ
    * - E001
      - [E001] The count of un-finished Todo must not be over {0}.
      - {0}… max unfinished count
    * - E002
      - [E002] The requested Todo is already finished. (id={0})
      - {0}… todoId
    * - E404
      - [E404] The requested Todo is not found. (id={0})
      - {0}… todoId

|

環境構築
================================================================================

プロジェクトの作成
--------------------------------------------------------------------------------

本チュートリアルでは、インフラストラクチャ層のRepositoryImplの実装として、

* データベースを使用せずMapを使ったインメモリ実装のRepositoryImpl
* Spring Data JPAの使用してデータベースにアクセスするRepositoryImpl
* TERASOLUNA DAO(MyBatis2)を使用してデータベースにアクセスするRepositoryImpl

の3種類を用意している。

まず、\ ``mvn archetype:generate``\ を利用して、実装するインフラストラクチャ層向けのブランクプロジェクトを作成する。

ここでは、Windows上にブランクプロジェクトを作成する手順となっている。


* データベースを使用せずMapを使ったインメモリ実装のRepositoryImpl用のプロジェクトを作成する場合は、O/R Mapperに依存しないブランクプロジェクトを作成するために、コマンドプロンプトで以下のコマンドを実行する。

 .. code-block:: console

    mvn org.apache.maven.plugins:maven-archetype-plugin:2.4:generate -B^
     -DarchetypeCatalog=http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases^
     -DarchetypeGroupId=org.terasoluna.gfw.blank^
     -DarchetypeArtifactId=terasoluna-gfw-web-blank-archetype^
     -DarchetypeVersion=1.0.6.RELEASE^
     -DgroupId=todo^
     -DartifactId=todo^
     -Dversion=1.0.0-SNAPSHOT

* Spring Data JPAの使用してデータベースにアクセスするRepositoryImpl用のプロジェクトを作成する場合は、JPA用のブランクプロジェクトを作成するために、コマンドプロンプトで以下のコマンドを実行する。

 .. code-block:: console

    mvn org.apache.maven.plugins:maven-archetype-plugin:2.4:generate -B^
     -DarchetypeCatalog=http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases^
     -DarchetypeGroupId=org.terasoluna.gfw.blank^
     -DarchetypeArtifactId=terasoluna-gfw-web-blank-jpa-archetype^
     -DarchetypeVersion=1.0.6.RELEASE^
     -DgroupId=todo^
     -DartifactId=todo^
     -Dversion=1.0.0-SNAPSHOT

* TERASOLUNA DAO(MyBatis2)を使用してデータベースにアクセスするRepositoryImpl用のプロジェクトを作成する場合は、TERASOLUNA DAO(MyBatis2)用のブランクプロジェクトを作成するために、コマンドプロンプトで以下のコマンドを実行する。

 .. code-block:: console

    mvn org.apache.maven.plugins:maven-archetype-plugin:2.4:generate -B^
     -DarchetypeCatalog=http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases^
     -DarchetypeGroupId=org.terasoluna.gfw.blank^
     -DarchetypeArtifactId=terasoluna-gfw-web-blank-mybatis2-archetype^
     -DarchetypeVersion=1.0.6.RELEASE^
     -DgroupId=todo^
     -DartifactId=todo^
     -Dversion=1.0.0-SNAPSHOT

コンソール上に以下のようなログが表示されれば、ブランクプロジェクトの作成は成功となる。

 .. code-block:: console

    C:\work>mvn org.apache.maven.plugins:maven-archetype-plugin:2.4:generate -B^
    More?  -DarchetypeCatalog=http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases^
    More?  -DarchetypeGroupId=org.terasoluna.gfw.blank^
    More?  -DarchetypeArtifactId=terasoluna-gfw-web-blank-archetype^
    More?  -DarchetypeVersion=1.0.6.RELEASE^
    More?  -DgroupId=todo^
    More?  -DartifactId=todo^
    More?  -Dversion=1.0.0-SNAPSHOT
    [INFO] Scanning for projects...
    [INFO]
    [INFO] ------------------------------------------------------------------------
    [INFO] Building Maven Stub Project (No POM) 1
    [INFO] ------------------------------------------------------------------------
    [INFO]
    [INFO] >>> maven-archetype-plugin:2.4:generate (default-cli) > generate-sources @ standalone-pom >>>
    [INFO]
    [INFO] <<< maven-archetype-plugin:2.4:generate (default-cli) < generate-sources @ standalone-pom <<<
    [INFO]
    [INFO] --- maven-archetype-plugin:2.4:generate (default-cli) @ standalone-pom ---
    [INFO] Generating project in Batch mode
    [INFO] Archetype repository not defined. Using the one from [org.terasoluna.gfw.blank:terasoluna-gfw-web-blank-archetype:1.0.0.RELEASE -> http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases] found in catalog http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases
    [INFO] ----------------------------------------------------------------------------
    [INFO] Using following parameters for creating project from Archetype: terasoluna-gfw-web-blank-archetype:1.0.6.RELEASE
    [INFO] ----------------------------------------------------------------------------
    [INFO] Parameter: groupId, Value: todo
    [INFO] Parameter: artifactId, Value: todo
    [INFO] Parameter: version, Value: 1.0.0-SNAPSHOT
    [INFO] Parameter: package, Value: todo
    [INFO] Parameter: packageInPathFormat, Value: todo
    [INFO] Parameter: package, Value: todo
    [INFO] Parameter: version, Value: 1.0.0-SNAPSHOT
    [INFO] Parameter: groupId, Value: todo
    [INFO] Parameter: artifactId, Value: todo
    [INFO] project created from Archetype in dir: C:\work\todo
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 2.730 s
    [INFO] Finished at: 2017-02-24T10:30:02+09:00
    [INFO] Final Memory: 12M/232M
    [INFO] ------------------------------------------------------------------------
    C:\work>

STSのメニューから、[File] -> [Import] -> [Maven] -> [Existing Maven Projects] -> [Next]を選択し、archetypeで作成したプロジェクトを選択する。

 .. figure:: images/NewMVCProjectImport.png
   :alt: New MVC Project Import
   :width: 60%

Root Directoryに \ ``C:\work\todo``\ を設定し、Projectsにtodoのpom.xmlが選択された状態で、 [Finish] を押下する。

 .. figure:: images/NewMVCProjectCreate.png
   :alt: New MVC Project Import
   :width: 60%

Package Explorerに、次のようなプロジェクトが生成される( **要インターネット接続** )。

 .. figure:: images/image004.png
   :alt: workspace
   :width: 30%

 .. note::

    パッケージ構成上、Package PresentaionをHierarchicalにしたほうが見通しがよい。

      .. figure:: ./images/presentation-hierarchical.png
         :width: 80%



 .. note::

    Bash上で\ ``mvn archetype:generate``\ を実行する場合は以下のように\ ``^``\ を\ ``\``\ に置き換えて実行する必要がある。
    
      .. code-block:: bash
      
        mvn org.apache.maven.plugins:maven-archetype-plugin:2.4:generate -B\
         -DarchetypeCatalog=http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases\
         -DarchetypeGroupId=org.terasoluna.gfw.blank\
         -DarchetypeArtifactId=terasoluna-gfw-web-blank-archetype\
         -DarchetypeVersion=1.0.6.RELEASE\
         -DgroupId=todo\
         -DartifactId=todo\
         -Dversion=1.0.0-SNAPSHOT



Mavenの設定
--------------------------------------------------------------------------------

| チュートリアルで必要となるライブラリへの依存関係の設定などは、作成したブランクプロジェクトに既に設定済みの状態である。
| そのため、設定の追加や変更は不要である。

 .. note::
 
    Proxyサーバーを介してインターネットアクセスする必要がある場合は、
    \ :file:`<HOME>/.m2/settings.xml`\に以下のような設定を行う必要がある。
    (Windows7の場合C:\\Users\\<YourName>\\.m2\\settings.xml)

        .. code-block:: xml

            <settings>
              <proxies>
                <proxy>
                  <active>true</active>
                  <protocol>[Proxy Server Protocol (http)]</protocol>
                  <port>[Proxy Server Port]</port>
                  <host>[Proxy Server Host]</host>
                  <username>[Username]</username>
                  <password>[Password]</password>
                </proxy>
              </proxies>
            </settings>

 .. note::

    インポート後にビルドエラーが発生する場合は、プロジェクト名を右クリックし、「Maven」->「Update Project」をクリックし、
    「OK」ボタンをクリックすることでエラーが解消されるケースがある。

     .. figure:: ./images/update-project.png
        :width: 60%


 .. warning::
 
    O/R Mapperを使用するブランクプロジェクトの場合、H2 Databaseがdependencyとして定義されているが、
    この設定は簡易的なアプリケーションを簡単に作成するためのものであり、実際のアプリケーション開発で使用されることは想定していない。
    
    以下の定義は、実際のアプリケーション開発を行う際は削除すること。
    
     .. code-block:: xml

        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>${com.h2database.version}</version>
            <scope>runtime</scope>
        </dependency>

|

プロジェクト構成
--------------------------------------------------------------------------------

チュートリアルで作成していくプロジェクトの構成について、以下に示す。

 .. code-block:: console

    src
      └main
          ├java
          │  └todo
          │    ├ app ... アプリケーション層を格納
          │    │   └todo ... todo管理業務に関わるクラスを格納
          │    └domain ... ドメイン層を格納
          │        ├model ... Domain Objectを格納
          │        ├repository ... Repositoryを格納
          │        │   └todo ... Todo用Repository
          │        └service ... Serviceを格納
          │            └todo ... TODO業務Service
          ├resources
          │  └META-INF
          │      └spring ... spring関連の設定ファイルを格納
          └wepapp
              └WEB-INF
                  └views ... jspを格納

 .. note::

    :ref:`前節の「プロジェクト構成」 <application-layering_project-structure>` ではマルチプロジェクトにすることを推奨していたが、
    本チュートリアルでは、学習容易性を重視しているためシングルプロジェクト構成にしている。
    
    **ただし、実プロジェクトで適用する場合は、マルチプロジェクト構成を強く推奨する。**

|

設定ファイルの確認
--------------------------------------------------------------------------------
チュートリアルを進める上で必要となる設定の多くは、作成したブランクプロジェクトに既に設定済みの状態である。

本節では、アプリケーションを動かすためにどのような設定ファイルが必要なのかを理解するために、 設定ファイルの確認をしていく。

 .. note::
 
    まず、手を動かしてTodoアプリを作成したい場合は、本節を読み飛ばしてもよいが、
    Todoアプリを作成した後に必ず一読して頂きたい。


web.xmlの確認
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
\ :file:`web.xml`\には、WebアプリケーションとしてTodoアプリをデプロイするための設定を行う。

作成したブランクプロジェクトの\ :file:`src/main/webapp/WEB-INF/web.xml`\は、以下のような設定となっている。

 .. code-block:: xml
    :emphasize-lines: 2, 6, 22, 77, 94, 105, 119

    <?xml version="1.0" encoding="UTF-8"?>
    <!-- (1) -->
    <web-app xmlns="http://java.sun.com/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
        version="3.0">
        <!-- (2) -->
        <listener>
            <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
        </listener>
        <listener>
            <listener-class>org.terasoluna.gfw.web.logging.HttpSessionEventLoggingListener</listener-class>
        </listener>
        <context-param>
            <param-name>contextConfigLocation</param-name>
            <!-- Root ApplicationContext -->
            <param-value>
                classpath*:META-INF/spring/applicationContext.xml
                classpath*:META-INF/spring/spring-security.xml
            </param-value>
        </context-param>

        <!-- (3) -->
        <filter>
            <filter-name>MDCClearFilter</filter-name>
            <filter-class>org.terasoluna.gfw.web.logging.mdc.MDCClearFilter</filter-class>
        </filter>
        <filter-mapping>
            <filter-name>MDCClearFilter</filter-name>
            <url-pattern>/*</url-pattern>
        </filter-mapping>

        <filter>
            <filter-name>exceptionLoggingFilter</filter-name>
            <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
        </filter>
        <filter-mapping>
            <filter-name>exceptionLoggingFilter</filter-name>
            <url-pattern>/*</url-pattern>
        </filter-mapping>

        <filter>
            <filter-name>XTrackMDCPutFilter</filter-name>
            <filter-class>org.terasoluna.gfw.web.logging.mdc.XTrackMDCPutFilter</filter-class>
        </filter>
        <filter-mapping>
            <filter-name>XTrackMDCPutFilter</filter-name>
            <url-pattern>/*</url-pattern>
        </filter-mapping>

        <filter>
            <filter-name>CharacterEncodingFilter</filter-name>
            <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
            <init-param>
                <param-name>encoding</param-name>
                <param-value>UTF-8</param-value>
            </init-param>
            <init-param>
                <param-name>forceEncoding</param-name>
                <param-value>true</param-value>
            </init-param>
        </filter>
        <filter-mapping>
            <filter-name>CharacterEncodingFilter</filter-name>
            <url-pattern>/*</url-pattern>
        </filter-mapping>

        <filter>
            <filter-name>springSecurityFilterChain</filter-name>
            <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
        </filter>

        <filter-mapping>
            <filter-name>springSecurityFilterChain</filter-name>
            <url-pattern>/*</url-pattern>
        </filter-mapping>


        <!-- (4) -->
        <servlet>
            <servlet-name>appServlet</servlet-name>
            <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
            <init-param>
                <param-name>contextConfigLocation</param-name>
                <!-- ApplicationContext for Spring MVC -->
                <param-value>classpath*:META-INF/spring/spring-mvc.xml</param-value>
            </init-param>
            <load-on-startup>1</load-on-startup>
        </servlet>

        <servlet-mapping>
            <servlet-name>appServlet</servlet-name>
            <url-pattern>/</url-pattern>
        </servlet-mapping>

        <!-- (5) -->
        <jsp-config>
            <jsp-property-group>
                <url-pattern>*.jsp</url-pattern>
                <el-ignored>false</el-ignored>
                <page-encoding>UTF-8</page-encoding>
                <scripting-invalid>false</scripting-invalid>
                <include-prelude>/WEB-INF/views/common/include.jsp</include-prelude>
            </jsp-property-group>
        </jsp-config>

        <!-- (6) -->
        <error-page>
            <error-code>500</error-code>
            <location>/WEB-INF/views/common/error/systemError.jsp</location>
        </error-page>
        <error-page>
            <error-code>404</error-code>
            <location>/WEB-INF/views/common/error/resourceNotFoundError.jsp</location>
        </error-page>
        <error-page>
            <exception-type>java.lang.Exception</exception-type>
            <location>/WEB-INF/views/common/error/unhandledSystemError.html</location>
        </error-page>

        <!-- (7) -->
        <session-config>
            <!-- 30min -->
            <session-timeout>30</session-timeout>
        </session-config>
    
    </web-app>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80

   * - 項番
     - 説明
   * - | (1)
     - | Servlet3.0を使用するための宣言。
   * - | (2)
     - | サーブレットコンテキストリスナーの定義。

       ブランクプロジェクトでは、
       
       * アプリケーション全体で使用される\ ``ApplicationContext``\を作成するための\ ``ContextLoaderListener``\
       * HttpSessionに対する操作をログ出力するための \ ``HttpSessionEventLoggingListener``\

       が設定済みである。
   * - | (3)
     - | サーブレットフィルタの定義。

       ブランクプロジェクトでは、
       
       * 共通ライブラリから提供しているサーブレットフィルタ
       * Spring Frameworkから提供されている文字エンコーディングを指定するための\ ``CharacterEncodingFilter``\
       * Spring Securityから提供されている認証・認可用のサーブレットフィルタ

       が設定済みである。
   * - | (4)
     - | Spring MVCのエントリポイントとなるDispatcherServletの定義。
       |
       | DispatcherServletの中で使用する\ ``ApplicationContext``\を、(2)で作成した\ ``ApplicatnionContext``\の子として作成する。
       | (2)で作成した\ ``ApplicatnionContext``\を親にすることで、(2)で読み込まれたコンポーネントも使用することができる。
   * - | (5)
     - | JSPの共通定義。
     
       ブランクプロジェクトでは、
       
       * JSP内でEL式が使用可能な状態
       * JSPのページエンコーディングとしてUTF-8
       * JSP内でスクリプティングが使用可能な状態
       * 各JSPの先頭でインクルードするJSPとして、\ :file:`/WEB-INF/views/common/include.jsp`\

       が設定済みである。
   * - | (6)
     - | エラーページの定義。
     
       ブランクプロジェクトでは、
       
       * サーブレットコンテナにHTTPステータスコードとして、\ ``404``\又は\ ``500``\が応答
       * サーブレットコンテナに例外が通知

       された際の遷移先が定義済みである。
   * - | (7)
     - | セッション管理の定義。
     
       ブランクプロジェクトでは、
       
       * セッションタイムアウトとして、30分

       が定義済みである。


|

インクルードJSPの確認
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
インクルードJSPには、全てのJSPに適用するJSPの設定や、タグライブラリの設定を行う。

作成したブランクプロジェクトの\ :file:`src/main/webapp/WEB-INF/views/common/include.jsp`\は、以下のような設定となっている。

 .. code-block:: jsp
    :emphasize-lines: 1, 3, 6, 9, 11 

    <!-- (1) -->
    <%@ page session="false"%>
    <!-- (2) -->
    <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
    <%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt"%>
    <!-- (3)  -->
    <%@ taglib uri="http://www.springframework.org/tags" prefix="spring"%>
    <%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>
    <!-- (4) -->
    <%@ taglib uri="http://www.springframework.org/security/tags" prefix="sec"%>
    <!-- (5) -->
    <%@ taglib uri="http://terasoluna.org/tags" prefix="t"%>
    <%@ taglib uri="http://terasoluna.org/functions" prefix="f"%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80

   * - 項番
     - 説明
   * - | (1)
     - | JSP実行時にセッションを作成しないようにするための定義。
   * - | (2)
     - | 標準タグライブラリの定義。
   * - | (3)
     - | Spring MVC用タグライブラリの定義。
   * - | (4)
     - | Spring Security用タグライブラリの定義(本チュートリアルでは使用しない。)
   * - | (5)
     - | 共通ライブラリで提供されている、EL関数、タグライブラリの定義。

|

Bean定義ファイルの確認
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

作成したブランクプロジェクトには、以下のBean定義ファイルとプロパティファイルが作成される。

* :file:`src/main/resources/META-INF/spring/applicationContext.xml`
* :file:`src/main/resources/META-INF/spring/todo-domain.xml`
* :file:`src/main/resources/META-INF/spring/todo-infra.xml`
* :file:`src/main/resources/META-INF/spring/todo-infra.properties`
* :file:`src/main/resources/META-INF/spring/todo-env.xml`
* :file:`src/main/resources/META-INF/spring/spring-mvc.xml`

 .. note::
 
    O/R Mapperに依存しないブランクプロジェクトを作成した場合は、\ ``todo-infra.properties``\と\ ``todo-env.xml``\は作成されない。

 .. note::

    本ガイドラインでは、Bean定義ファイルを役割(層)ごとにファイルを分割することを推奨している。
    
    これは、どこに何が定義されているか想像しやすく、メンテナンス性が向上するからである。
    今回のチュートリアルのような小さなアプリケーションでは効果はないが、アプリケーションの規模が大きくなるにつれ、効果が大きくなる。

|

applicationContext.xmlの確認
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\ :file:`applicationContext.xml`\には、Todoアプリ全体に関わる設定を行う。

| 作成したブランクプロジェクトの\ :file:`src/main/resources/META-INF/spring/applicationContext.xml`\は、以下のような設定となっている。
| なお、チュートリアルで使用しないコンポーネントについての説明は割愛する。

 .. code-block:: xml
    :emphasize-lines: 9-10, 14-16, 18-19

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

        <!-- (1) -->
        <import resource="classpath:/META-INF/spring/todo-domain.xml" />

        <bean id="passwordEncoder" class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder" />

        <!-- (2) -->
        <context:property-placeholder
            location="classpath*:/META-INF/spring/*.properties" />

        <!-- (3) -->
        <bean class="org.dozer.spring.DozerBeanMapperFactoryBean">
            <property name="mappingFiles"
                value="classpath*:/META-INF/dozer/**/*-mapping.xml" />
        </bean>

        <!-- Message -->
        <bean id="messageSource"
            class="org.springframework.context.support.ResourceBundleMessageSource">
            <property name="basenames">
                <list>
                    <value>i18n/application-messages</value>
                </list>
            </property>
        </bean>

        <!-- Exception Code Resolver. -->
        <bean id="exceptionCodeResolver"
            class="org.terasoluna.gfw.common.exception.SimpleMappingExceptionCodeResolver">
            <!-- Setting and Customization by project. -->
            <property name="exceptionMappings">
                <map>
                    <entry key="ResourceNotFoundException" value="e.xx.fw.5001" />
                    <entry key="InvalidTransactionTokenException" value="e.xx.fw.7001" />
                    <entry key="BusinessException" value="e.xx.fw.8001" />
                    <entry key=".DataAccessException" value="e.xx.fw.9002" />
                </map>
            </property>
            <property name="defaultExceptionCode" value="e.xx.fw.9001" />
        </bean>

        <!-- Exception Logger. -->
        <bean id="exceptionLogger"
            class="org.terasoluna.gfw.common.exception.ExceptionLogger">
            <property name="exceptionCodeResolver" ref="exceptionCodeResolver" />
        </bean>

        <!-- Filter. -->
        <bean id="exceptionLoggingFilter"
            class="org.terasoluna.gfw.web.exception.ExceptionLoggingFilter" >
            <property name="exceptionLogger" ref="exceptionLogger" />
        </bean>

    </beans>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | ドメイン層に関するBean定義ファイルをimportする。
   * - | (2)
     - | プロパティファイルの読み込み設定を行う。
       | src/main/resources/META-INF/spring直下の任意のプロパティファイルを読み込む。
       | この設定により、プロパティファイルの値をBean定義ファイル内で${propertyName}形式で埋め込んだり、Javaクラスに@Value("${propertyName}")でインジェクションすることができる。
   * - | (3)
     - | Bean変換用ライブラリDozerのMapperを定義する。
       | マッピングファイルに関して `Dozerマニュアル <http://dozer.sourceforge.net/documentation/mappings.html>`_ を参照されたい。)

 .. note::

        エディタの「Configure Namspecse」タブにて、以下のように「beans」と「context」にチェックを入れると、
        XML編集時にCtrl+Spaceを使用して入力を補完することができる。

        .. figure:: ./images/image021.jpg
           :width: 60%
           :align: center

        「Namespace Versions」にはバージョンなしのxsdファイルを選択することを推奨する。
        バージョンなしのxsdファイルを選択することで、常にjarに含まれる最新のxsdが使用されるため、
        Springのバージョンアップを意識する必要がなくなる。

        .. figure:: ./images/image023.png
           :width: 60%
           :align: center

|

todo-domain.xmlの確認
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ :file:`todo-domain.xml`\には、Todoアプリのドメイン層に関わる設定を行う。

| 作成したブランクプロジェクトの\ :file:`src/main/resources/META-INF/spring/todo-domain.xml`\は、以下のような設定となっている。
| なお、チュートリアルで使用しないコンポーネントについての説明は割愛する。

 .. code-block:: xml
    :emphasize-lines: 9-10, 13-14

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

        <!-- (1) -->
        <import resource="classpath:META-INF/spring/todo-infra.xml" />
        <import resource="classpath*:META-INF/spring/**/*-codelist.xml" />

        <!-- (2) -->
        <context:component-scan base-package="todo.domain" />

        <!-- AOP. -->
        <bean id="resultMessagesLoggingInterceptor"
            class="org.terasoluna.gfw.common.exception.ResultMessagesLoggingInterceptor">
            <property name="exceptionLogger" ref="exceptionLogger" />
        </bean>
        <aop:config>
            <aop:advisor advice-ref="resultMessagesLoggingInterceptor"
                pointcut="@within(org.springframework.stereotype.Service)" />
        </aop:config>

    </beans>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80

   * - 項番
     - 説明
   * - | (1)
     - | インフラストラクチャ層に関するBean定義ファイルをimportする。
   * - | (2)
     - | ドメイン層のクラスを管理するtodo.domainパッケージ配下をcomponent-scan対象とする。
       | これにより、todo.domainパッケージ配下のクラスに ``@Repository`` , ``@Service`` などのアノテーションを付けることで、Spring Framerowkが管理するBeanとして登録される。
       | 登録されたクラス(Bean)は、ControllerやServiceクラスにDIする事で、利用する事が出来る。


 .. note::
 
    O/R Mapperに依存するブランクプロジェクトを作成した場合は、\ ``@Transactional``\アノテーションによるトランザクション管理を有効にするために、
    \ ``<tx:annotation-driven>``\タグを設定されている。

     .. code-block:: xml
        :emphasize-lines: 9-10, 12-13

         <tx:annotation-driven />

|

todo-infra.xmlの確認
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ :file:`todo-infra.xml`\には、Todoアプリのインフラストラクチャ層に関わる設定を行う。

作成したブランクプロジェクトの\ :file:`src/main/resources/META-INF/spring/todo-infra.xml`\は、
以下のような設定となっている。

\ :file:`todo-infra.xml`\は、インフラストラクチャ層によって設定が大きく異なるため、
ブランクプロジェクト毎に説明を行う。
作成したブランクプロジェクト以外の説明は読み飛ばしてもよい。


O/R Mapperに依存しないブランクプロジェクトを作成した場合
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

O/R Mapperに依存しないブランクプロジェクトを作成した場合、以下のように空定義のファイルが作成される。

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">


    </beans>

JPA用のブランクプロジェクトを作成した場合
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
JPA用のブランクプロジェクトを作成した場合、以下のような設定となっている。

 .. code-block:: xml
    :emphasize-lines: 9-10, 12-13, 15-17, 22-24, 26-27, 30-31

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jpa="http://www.springframework.org/schema/data/jpa"
        xmlns:util="http://www.springframework.org/schema/util"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd
            http://www.springframework.org/schema/data/jpa http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">

        <!-- (1) -->
        <import resource="classpath:/META-INF/spring/todo-env.xml" />

        <!-- (2) -->
        <jpa:repositories base-package="todo.domain.repository"></jpa:repositories>

        <!-- (3) -->
        <bean id="jpaVendorAdapter"
            class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
            <property name="showSql" value="false" />
            <property name="database" value="${database}" />
        </bean>

        <!-- (4) -->
        <bean
            class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean"
            id="entityManagerFactory">
            <!-- (5) -->
            <property name="packagesToScan" value="todo.domain.model" />
            <property name="dataSource" ref="dataSource" />
            <property name="jpaVendorAdapter" ref="jpaVendorAdapter" />
            <!-- (6) -->
            <property name="jpaPropertyMap">
                <util:map>
                    <entry key="hibernate.hbm2ddl.auto" value="none" />
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
    
    </beans>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80

   * - 項番
     - 説明
   * - | (1)
     - | 環境依存するコンポーネント(データソースやトランザクションマネージャなど)を定義するBean定義ファイルをimportする。
   * - | (2)
     - | Spring Data JPAを使用して、Repositoryインタフェースから実装クラスを自動生成する。
       | <jpa:repository>タグのbase-package属性で、対象のRepositoryを含むパッケージを指定する。
   * - | (3)
     - | JPAの実装ベンダの設定を行う。
       | JPA実装として、Hibernateを使うため、HibernateJpaVendorAdapterを定義している。
   * - | (4)
     - | EntityManagerの定義を行う。
   * - | (5)
     - | JPAのエンティティとして扱うクラスが格納されているパッケージ名を指定する。
   * - | (6)
     - | Hibernateに関する詳細な設定を行う。


TERASOLUNA DAO(MyBatis2)用のブランクプロジェクトを作成した場合
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
TERASOLUNA DAO(MyBatis2)用のブランクプロジェクトを作成した場合、以下のような設定となっている。

 .. code-block:: xml
   :emphasize-lines: 6-7, 9-11, 12-14, 15-17, 21

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    
        <!-- (1) -->
        <import resource="classpath:/META-INF/spring/todo-env.xml" />
    
        <!-- (2) -->
        <bean id="sqlMapClient"
            class="org.springframework.orm.ibatis.SqlMapClientFactoryBean">
            <!-- (3) -->
            <property name="configLocations"
                value="classpath*:/META-INF/mybatis/config/*sqlMapConfig.xml" />
            <!-- (4) -->
            <property name="mappingLocations"
                value="classpath*:/META-INF/mybatis/sql/**/*-sqlmap.xml" />
            <property name="dataSource" ref="dataSource" />
        </bean>
    
        <!-- (5) -->
        <bean id="queryDAO" class="jp.terasoluna.fw.dao.ibatis.QueryDAOiBatisImpl">
            <property name="sqlMapClient" ref="sqlMapClient" />
        </bean>
    
        <bean id="updateDAO" class="jp.terasoluna.fw.dao.ibatis.UpdateDAOiBatisImpl">
            <property name="sqlMapClient" ref="sqlMapClient" />
        </bean>
    
        <bean id="spDAO"
            class="jp.terasoluna.fw.dao.ibatis.StoredProcedureDAOiBatisImpl">
            <property name="sqlMapClient" ref="sqlMapClient" />
        </bean>
    
        <bean id="queryRowHandleDAO"
            class="jp.terasoluna.fw.dao.ibatis.QueryRowHandleDAOiBatisImpl">
            <property name="sqlMapClient" ref="sqlMapClient" />
        </bean>
    </beans>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80


   * - 項番
     - 説明
   * - | (1)
     - | 環境依存するコンポーネント(データソースやトランザクションマネージャなど)を定義するBean定義ファイルをimportする。
   * - | (2)
     - | SqlMapClientの定義を行う。
   * - | (3)
     - | SqlMap設定ファイルのパスを設定する。
       | ここでは、META-INF/mybatis/config以下の、\*sqlMapConfig.xmlを読み込む。
   * - | (4)
     - | SqlMapファイルのパスを設定する。
       | ここでは、META-INF/mybatis/sql以下の、任意のフォルダの*-sqlmap.xmlを読み込む。
   * - | (5)
     - | TERASOLUNA DAOの定義を行う。

 .. note::
 
    \ :file:`sqlMapConfig.xml`\は、MyBatis2自体の動作設定を行う設定ファイルである。
    
    ブランクプロジェクトでは、デフォルトで以下の設定が行われている。

     .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>
        <!DOCTYPE sqlMapConfig 
                    PUBLIC "-//ibatis.apache.org//DTD SQL Map Config 2.0//EN"
                    "http://ibatis.apache.org/dtd/sql-map-config-2.dtd">
        <sqlMapConfig>
            <settings useStatementNamespaces="true" />
        </sqlMapConfig>
        
    \ ``useStatementNamespaces``\を\ ``true``\にすることで、SQLIDにネームスペースを指定することが出来る。
 

|

todo-infra.propertiesの確認
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ :file:`todo-infra.properties`\には、Todoアプリのインフラストラクチャ層の環境依存値の設定を行う。

O/R Mapperに依存しないブランクプロジェクトを作成した際は、\ :file:`todo-infra.properties`\は作成されない。

作成したブランクプロジェクトの\ :file:`src/main/resources/META-INF/spring/todo-infra.properties`\は、
以下のような設定となっている。

 .. code-block:: properties
    :emphasize-lines: 1, 7

    # (1)
    database=H2
    database.url=jdbc:h2:mem:todo;DB_CLOSE_DELAY=-1
    database.username=sa
    database.password=
    database.driverClassName=org.h2.Driver
    # (2)
    # connection pool
    cp.maxActive=96
    cp.maxIdle=16
    cp.minIdle=0
    cp.maxWait=60000

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80


   * - 項番
     - 説明
   * - | (1)
     - | データベースに関する設定を行う。
       | 本チュートリアルでは、データベースのセットアップの手間を省くため、H2 Databaseを使用する。
   * - | (2)
     - | コネクションプールに関する設定。


 .. note::
 
    これらの設定値は、\ :file:`todo-env.xml`\から参照されている。

|

todo-env.xmlの確認
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ :file:`todo-env.xml`\には、デプロイする環境によって設定が異なるコンポーネントの設定を行う。

データベースにアクセスしないブランクプロジェクトを作成した際は、\ :file:`todo-env.xml`\は作成されない。

作成したブランクプロジェクトの\ ``src/main/resources/META-INF/spring/todo-env.xml``\は、以下のような設定となっている。

 .. code-block:: xml
    :emphasize-lines: 11-13, 25-28, 36-40

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jee="http://www.springframework.org/schema/jee"
        xmlns:jdbc="http://www.springframework.org/schema/jdbc"
        xsi:schemaLocation="http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
            http://www.springframework.org/schema/jee http://www.springframework.org/schema/jee/spring-jee.xsd
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

        <bean id="dateFactory" class="org.terasoluna.gfw.common.date.DefaultDateFactory" />

        <!-- (1) -->
        <bean id="realDataSource" class="org.apache.commons.dbcp.BasicDataSource"
            destroy-method="close">
            <property name="driverClassName" value="${database.driverClassName}" />
            <property name="url" value="${database.url}" />
            <property name="username" value="${database.username}" />
            <property name="password" value="${database.password}" />
            <property name="defaultAutoCommit" value="false" />
            <property name="maxActive" value="${cp.maxActive}" />
            <property name="maxIdle" value="${cp.maxIdle}" />
            <property name="minIdle" value="${cp.minIdle}" />
            <property name="maxWait" value="${cp.maxWait}" />
        </bean>

        <!-- (2) -->
        <bean id="dataSource" class="net.sf.log4jdbc.Log4jdbcProxyDataSource">
            <constructor-arg index="0" ref="realDataSource" />
        </bean>

        <jdbc:initialize-database data-source="dataSource"
            ignore-failures="ALL">
            <jdbc:script location="classpath:/database/${database}-schema.sql" />
            <jdbc:script location="classpath:/database/${database}-dataload.sql" />
        </jdbc:initialize-database>

        <!-- (3) -->
        <bean id="transactionManager"
            class="org.springframework.orm.jpa.JpaTransactionManager">
            <property name="entityManagerFactory" ref="entityManagerFactory" />
        </bean>
        <!--  REMOVE THIS LINE IF YOU USE MyBatis2
        <bean id="transactionManager"
            class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <property name="dataSource" ref="dataSource" />
        </bean>
              REMOVE THIS LINE IF YOU USE MyBatis2  -->
    </beans>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80


   * - 項番
     - 説明
   * - | (1)
     - | 実データソースの設定。
   * - | (2)
     - | データソースの設定。
       | JDBC関連のログを出力する機能をもったデータソースを指定している。
       | \ ``net.sf.log4jdbc.Log4jdbcProxyDataSource``\を使用すると、SQLなどのJDBC関連のログを出力できるため、デバッグに役立つ情報を出力することができる。
   * - | (3)
     - | トランザクションマネージャの設定。idは、transactionManagerにすること。
       | 別の名前を指定する場合は、<tx:annotation-driven>タグにも、トランザクションマネージャ名を指定する必要がある。

 .. note::

    使用するトランザクションマネージャは、使用するO/R Mapperによって切り替える必要がある。
    
    作成したブランクプロジェクトが、
    
    * JPA用のブランクプロジェクトを作成した場合は、
      JPAのAPIを使用してトランザクションを制御するクラス(\ ``org.springframework.orm.jpa.JpaTransactionManager``\)
    
    * TERASOLUNA DAO(MyBatis2)用のブランクプロジェクトを作成した場合は、
      JDBCのAPIを使用してトランザクションを制御するクラス(\ ``org.springframework.jdbc.datasource.DataSourceTransactionManager``\)
    
    が設定されている。

 .. note::
 
    JavaEEコンテナ上にアプリケーションをデプロイする場合は、
    JTAのAPIを利用してトランザクションを制御するクラス(\ ``org.springframework.transaction.jta.JtaTransactionManager``\ )を使用したほうがよい。
    \ ``JtaTransactionManager``\ を使う場合は、
    \ ``<tx:jta-transaction-manager />``\ を指定してトランザクションマネージャの定義を行う。
    
    トランザクションマネージャの設定が、アプリケーションをデプロイする環境によって変わらないプロジェクト(例えば、Tomcatを使用する場合など)は、
    \ :file:`todo-infra.xml`に定義してもよい。

|

spring-mvc.xmlの確認
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ :file:`spring-mvc.xml`\には、Spring MVCに関する定義を行う。

| 作成したブランクプロジェクトの\ :file:`src/main/resources/META-INF/spring/spring-mvc.xml`\は、以下のような設定となっている。
| なお、チュートリアルで使用しないコンポーネントについての説明は割愛する。

 .. code-block:: xml
    :emphasize-lines: 12-14, 16-21, 28-29, 31-34, 37-44, 71-77

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
        xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:util="http://www.springframework.org/schema/util"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

        <!-- (1) -->
        <context:property-placeholder
            location="classpath*:/META-INF/spring/*.properties" />

        <!-- (2) -->
        <mvc:annotation-driven>
            <mvc:argument-resolvers>
                <bean
                    class="org.springframework.data.web.PageableHandlerMethodArgumentResolver" />
            </mvc:argument-resolvers>
            <!-- workarround to CVE-2016-5007. -->
            <mvc:path-matching path-matcher="pathMatcher" />
        </mvc:annotation-driven>

        <mvc:default-servlet-handler />

        <!-- (3) -->
        <context:component-scan base-package="todo.app" />

        <!-- (4) -->
        <mvc:resources mapping="/resources/**"
            location="/resources/,classpath:META-INF/resources/"
            cache-period="#{60 * 60}" />

        <mvc:interceptors>
            <!-- (5) -->
            <mvc:interceptor>
                <mvc:mapping path="/**" />
                <mvc:exclude-mapping path="/resources/**" />
                <mvc:exclude-mapping path="/**/*.html" />
                <bean
                    class="org.terasoluna.gfw.web.logging.TraceLoggingInterceptor" />
            </mvc:interceptor>
            <mvc:interceptor>
                <mvc:mapping path="/**" />
                <mvc:exclude-mapping path="/resources/**" />
                <mvc:exclude-mapping path="/**/*.html" />
                <bean
                    class="org.terasoluna.gfw.web.token.transaction.TransactionTokenInterceptor" />
            </mvc:interceptor>
            <mvc:interceptor>
                <mvc:mapping path="/**" />
                <mvc:exclude-mapping path="/resources/**" />
                <mvc:exclude-mapping path="/**/*.html" />
                <bean class="org.terasoluna.gfw.web.codelist.CodeListInterceptor">
                    <property name="codeListIdPattern" value="CL_.+" />
                </bean>
            </mvc:interceptor>
            <!--  REMOVE THIS LINE IF YOU USE JPA
            <mvc:interceptor>
                <mvc:mapping path="/**" />
                <mvc:exclude-mapping path="/resources/**" />
                <mvc:exclude-mapping path="/**/*.html" />
                <bean
                    class="org.springframework.orm.jpa.support.OpenEntityManagerInViewInterceptor" />
            </mvc:interceptor>
                REMOVE THIS LINE IF YOU USE JPA  -->
        </mvc:interceptors>

        <!-- (6) -->
        <!-- Settings View Resolver. -->
        <bean id="viewResolver"
            class="org.springframework.web.servlet.view.InternalResourceViewResolver">
            <property name="prefix" value="/WEB-INF/views/" />
            <property name="suffix" value=".jsp" />
        </bean>

        <bean id="requestDataValueProcessor"
            class="org.terasoluna.gfw.web.mvc.support.CompositeRequestDataValueProcessor">
            <constructor-arg>
                <util:list>
                    <bean
                        class="org.springframework.security.web.servlet.support.csrf.CsrfRequestDataValueProcessor" factory-method="create" />
                    <bean
                        class="org.terasoluna.gfw.web.token.transaction.TransactionTokenRequestDataValueProcessor" />
                </util:list>
            </constructor-arg>
        </bean>

        <!-- Setting Exception Handling. -->
        <!-- Exception Resolver. -->
        <bean class="org.terasoluna.gfw.web.exception.SystemExceptionResolver">
            <property name="exceptionCodeResolver" ref="exceptionCodeResolver" />
            <!-- Setting and Customization by project. -->
            <property name="order" value="3" />
            <property name="exceptionMappings">
                <map>
                    <entry key="ResourceNotFoundException" value="common/error/resourceNotFoundError" />
                    <entry key="BusinessException" value="common/error/businessError" />
                    <entry key="InvalidTransactionTokenException" value="common/error/transactionTokenError" />
                    <entry key=".DataAccessException" value="common/error/dataAccessError" />
                </map>
            </property>
            <property name="statusCodes">
                <map>
                    <entry key="common/error/resourceNotFoundError" value="404" />
                    <entry key="common/error/businessError" value="409" />
                    <entry key="common/error/transactionTokenError" value="409" />
                    <entry key="common/error/dataAccessError" value="500" />
                </map>
            </property>
            <property name="defaultErrorView" value="common/error/systemError" />
            <property name="defaultStatusCode" value="500" />
        </bean>
        <!-- Setting AOP. -->
        <bean id="handlerExceptionResolverLoggingInterceptor"
            class="org.terasoluna.gfw.web.exception.HandlerExceptionResolverLoggingInterceptor">
            <property name="exceptionLogger" ref="exceptionLogger" />
        </bean>
        <aop:config>
            <aop:advisor advice-ref="handlerExceptionResolverLoggingInterceptor"
                pointcut="execution(* org.springframework.web.servlet.HandlerExceptionResolver.resolveException(..))" />
        </aop:config>

        <!-- Setting PathMatcher. -->
        <bean id="pathMatcher" class="org.springframework.util.AntPathMatcher">
            <property name="trimTokens" value="false" />
        </bean>

    </beans>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80

   * - 項番
     - 説明
   * - | (1)
     - | プロパティファイルの読み込み設定を行う。
       | src/main/resources/META-INF/spring直下の任意のプロパティファイルを読み込む。
       | この設定により、プロパティファイルの値をBean定義ファイル内で${propertyName}形式で埋め込んだり、Javaクラスに@Value("${propertyName}")でインジェクションすることができる。
   * - | (2)
     - | Spring MVCのアノテーションベースのデフォルト設定を行う。
   * - | (3)
     - | アプリケーション層のクラスを管理するtodo.appパッケージ配下をcomponent-scan対象とする。
   * - | (4)
     - | 静的リソース(css, images, jsなど)アクセスのための設定を行う。
       | mapping属性にURLのパスを、location属性に物理的なパスの設定を行う。
       | この設定の場合<contextPath>/rerources/css/styles.cssに対してリクエストが来た場合、WEB-INF/resources/css/styles.cssを探し、見つからなければクラスパス上(src/main/resourcesやjar内)のresources/css/style.cssを探す。
       | WEB-INF/resources/css/styles.cssが見つからなければ、404エラーを返す。
       | ここではcache-period属性で静的リソースのキャッシュ時間(3600秒=60分)も設定している。
       | ``cache-period="3600"`` と設定しても良いが、60分であることを明示するために `SpEL <http://static.springsource.org/spring/docs/3.2.18.RELEASE/spring-framework-reference/html/expressions.html#expressions-beandef-xml-based>`_ を使用して ``cache-period="#{60 * 60}"`` と書く方が分かりやすい。
   * - | (5)
     - | コントローラ処理のTraceログを出力するインターセプタを設定する。/resources以下を除く任意のパスに適用されるように設定する。
   * - | (6)
     - | ViewResolverの設定を行う。この設定により、例えばコントローラからview名”hello”が返却された場合には/WEB-INF/views/hello.jspが実行される。

 .. note::
 
    JPA用のブランクプロジェクトを作成した場合は、\ ``<mvc:interceptors>``\の定義として、
    \ ``OpenEntityManagerInViewInterceptor``\の定義が有効な状態となっている。
    
     .. code-block:: xml
    
        <mvc:interceptor>
            <mvc:mapping path="/**" />
            <mvc:exclude-mapping path="/resources/**" />
            <mvc:exclude-mapping path="/**/*.html" />
            <bean
                class="org.springframework.orm.jpa.support.OpenEntityManagerInViewInterceptor" />
        </mvc:interceptor>    


    \ ``OpenEntityManagerInViewInterceptor``\は、EntityManagerのライフサイクルの開始と終了を行うInterceptorである。
    この設定を追加することで、アプリケーション層(Contollerや、Viewクラス)でのLazy Loadが、サポートされる。

 .. note:: 

    エディタの「Configure Namspecse」タブにて、\ :file:`todo-domain.xml`\で説明した操作に加え、「mvc」と「util」にもチェックを入れると、
    「mvc」と「util」ネームスペースについても、XML編集時にCtrl+Spaceを使用して入力を補完することができる。

    .. figure:: ./images/image028.png
       :width: 60%
       :align: center

|

logback.xmlの確認
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ :file:`logback.xml`\には、ログ出力に関する定義を行う。

| 作成したブランクプロジェクトの\ :file:`src/main/resources/logback.xml`\は、以下のような設定となっている。
| なお、チュートリアルで使用しないログ設定についての説明は割愛する。

 .. code-block:: xml
    :emphasize-lines: 4-9, 36-39, 45-48

    <?xml version="1.0" encoding="UTF-8"?>
    <configuration>
    
        <!-- (1) -->
        <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern><![CDATA[date:%d{yyyy-MM-dd HH:mm:ss}\tthread:%thread\tX-Track:%X{X-Track}\tlevel:%-5level\tlogger:%-48logger{48}\tmessage:%msg%n]]></pattern>
            </encoder>
        </appender>
    
        <appender name="APPLICATION_LOG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <file>log/todo-application.log</file>
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>log/todo-application-%d{yyyyMMdd}.log</fileNamePattern>
                <maxHistory>7</maxHistory>
            </rollingPolicy>
            <encoder>
                <charset>UTF-8</charset>
                <pattern><![CDATA[date:%d{yyyy-MM-dd HH:mm:ss}\tthread:%thread\tX-Track:%X{X-Track}\tlevel:%-5level\tlogger:%-48logger{48}\tmessage:%msg%n]]></pattern>
            </encoder>
        </appender>
    
        <appender name="MONITORING_LOG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <file>log/todo-monitoring.log</file>
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>log/todo-monitoring-%d{yyyyMMdd}.log</fileNamePattern>
                <maxHistory>7</maxHistory>
            </rollingPolicy>
            <encoder>
                <charset>UTF-8</charset>
                <pattern><![CDATA[date:%d{yyyy-MM-dd HH:mm:ss}\tX-Track:%X{X-Track}\tlevel:%-5level\tmessage:%msg%n]]></pattern>
            </encoder>
        </appender>
    
        <!-- Application Loggers -->
        <!-- (2) -->
        <logger name="todo">
            <level value="debug" />
        </logger>
    
        <!-- TERASOLUNA -->
        <logger name="org.terasoluna.gfw">
            <level value="info" />
        </logger>
        <!-- (3) -->
        <logger name="org.terasoluna.gfw.web.logging.TraceLoggingInterceptor">
            <level value="trace" />
        </logger>
        <logger name="org.terasoluna.gfw.common.exception.ExceptionLogger">
            <level value="info" />
        </logger>
        <logger name="org.terasoluna.gfw.common.exception.ExceptionLogger.Monitoring" additivity="false">
            <level value="error" />
            <appender-ref ref="MONITORING_LOG_FILE" />
        </logger>
    
        <!-- 3rdparty Loggers -->
        <logger name="org.springframework">
            <level value="warn" />
        </logger>
    
        <logger name="org.springframework.web.servlet">
            <level value="info" />
        </logger>
    
        <!--  REMOVE THIS LINE IF YOU USE JPA
        <logger name="org.hibernate.engine.transaction">
            <level value="debug" />
        </logger>
              REMOVE THIS LINE IF YOU USE JPA  -->
        <!--  REMOVE THIS LINE IF YOU USE MyBatis2
        <logger name="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <level value="debug" />
        </logger>
              REMOVE THIS LINE IF YOU USE MyBatis2  -->
    
        <logger name="jdbc.sqltiming">
            <level value="debug" />
        </logger>

        <!-- only for development -->
        <logger name="jdbc.resultsettable">
            <level value="debug" />
        </logger>

        <root level="warn">
            <appender-ref ref="STDOUT" />
            <appender-ref ref="APPLICATION_LOG_FILE" />
        </root>
    
    </configuration>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | 標準出力でログを出力するアペンダを設定。
   * - | (2)
     - | todoパッケージ以下はdebugレベル以上を出力するように設定。
   * - | (3)
     - | spring-mvc.xmlに設定した ``TraceLoggingInterceptor`` に出力されるようにtraceレベルで設定。


 .. note::
 
    O/R Mapperを使用するブランクプロジェクトを作成した場合は、トランザクション制御関連のログを出力するロガーが有効な状態となっている。
    
    * JPA用のブランクプロジェクト
    
     .. code-block:: xml

        <logger name="org.hibernate.engine.transaction">
            <level value="debug" />
        </logger>

    * TERASOLUNA DAO(MyBatis2)用のブランクプロジェクト

     .. code-block:: xml

        <logger name="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <level value="debug" />
        </logger>

|

ブランクプロジェクトの動作確認
--------------------------------------------------------------------------------
Todoアプリケーションの開発を始める前に、ブランクプロジェクトの動作確認を行う。

ブランクプロジェクトでは、トップページを表示するためのControllerとJSPの実装が用意されているため、
トップページを表示する事で動作確認を行う事ができる。

ブランクプロジェクトから提供されているController(\ :file:`src/main/java/todo/app/welcome/HelloControllerr.java`\)は、
以下のような実装となっている。

 .. code-block:: java
    :emphasize-lines: 17-18, 21-23, 28-29, 31-32, 40-41, 43-44

    package todo.app.welcome;

    import java.text.DateFormat;
    import java.util.Date;
    import java.util.Locale;

    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;

    /**
     * Handles requests for the application home page.
     */
    // (1)
    @Controller
    public class HelloControllerr {

        // (2)
        private static final Logger logger = LoggerFactory
                .getLogger(HelloControllerr.class);

        /**
         * Simply selects the home view to render by returning its name.
         */
        // (3)
        @RequestMapping(value = "/", method = {RequestMethod.GET, RequestMethod.POST})
        public String home(Locale locale, Model model) {
            // (4)
            logger.info("Welcome home! The client locale is {}.", locale);
    
            Date date = new Date();
            DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG,
                    DateFormat.LONG, locale);

            String formattedDate = dateFormat.format(date);

            // (5)
            model.addAttribute("serverTime", formattedDate);

            // (6)
            return "welcome/home";
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | Controllerとしてcomponent-scanの対象とするため、クラスレベルに ``@Controller`` アノテーションが付与している。
   * - | (2)
     - | (4)でログ出力するためのロガーの生成している。
       | ロガーの実装はlogbackのものであるが、APIはSLF4Jの ``org.slf4j.Logger`` を使用している。
   * - | (3)
     - | ``@RequestMapping`` アノテーションを使用して、”/”(ルート)へのアクセスに対するメソッドとしてマッピングを行っている。
   * - | (4)
     - | メソッドが呼ばれたことを通知するためのログをinfoレベルで出力している。
   * - | (5)
     - | 画面に表示するための日付文字列を、"serverTime"という属性名でModelに設定している。
   * - | (6)
     - | view名として"welcome/home"を返す。ViewResolverの設定により、WEB-INF/views/welcome/home.jspが呼び出される。

|

ブランクプロジェクトから提供されているJSP(\ :file:`src/main/webapp/WEB-INF/views/welcome/home.jsp`\)は、
以下のような実装となっている。

 .. code-block:: jsp
    :emphasize-lines: 12-13

    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="utf-8">
    <title>Home</title>
    <link rel="stylesheet"
        href="${pageContext.request.contextPath}/resources/app/css/styles.css">
    </head>
    <body>
        <div id="wrapper">
            <h1>Hello world!</h1>
            <!-- (1) -->
            <p>The time on the server is ${serverTime}.</p>
        </div>
    </body>
    </html>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | ControllerでModelに設定した”serverTime”を表示する。
       | ここでは、XSS対策を行っていないが、ユーザの入力値を表示する場合は、\ ``f:h()``\ 関数を用いて、必ずXSS対策を行うこと。

|

パッケージプロジェクト名”todo”を右クリックして「Run As」->「Run on Server」



 .. figure:: ./images/image031.jpg
   :width: 60%

|

実行したいAPサーバー(ここではVMWare vFabric tc Server Developer Edition v2.9)を選び
「Next」をクリック

 .. figure:: ./images/image032.jpg
   :width: 60%

|

todoが「Configured」に含まれていることを確認して「Finish」をクリックしてサーバーを起動する。


 .. figure:: ./images/image033.jpg
   :width: 60%


起動すると以下のようなログが出力される。
”/”というパスに対して ``todo.app.welcome.HelloControllerr`` のhelloメソッドがマッピングされていることが分かる。


 .. code-block:: text
   :emphasize-lines: 2

    date:2014-08-25 12:35:34    thread:localhost-startStop-1    X-Track:    level:INFO  logger:o.springframework.web.servlet.DispatcherServlet  message:FrameworkServlet 'appServlet': initialization started
    date:2014-08-25 12:35:35    thread:localhost-startStop-1    X-Track:    level:INFO  logger:o.s.w.s.m.m.a.RequestMappingHandlerMapping       message:Mapped "{[/],methods=[GET || POST],params=[],headers=[],consumes=[],produces=[],custom=[]}" onto public java.lang.String todo.app.welcome.HelloControllerr.home(java.util.Locale,org.springframework.ui.Model)
    date:2014-08-25 12:35:36    thread:localhost-startStop-1    X-Track:    level:INFO  logger:o.s.web.servlet.handler.SimpleUrlHandlerMapping  message:Mapped URL path [/**] onto handler 'org.springframework.web.servlet.resource.DefaultServletHttpRequestHandler#0'
    date:2014-08-25 12:35:36    thread:localhost-startStop-1    X-Track:    level:INFO  logger:o.s.web.servlet.handler.SimpleUrlHandlerMapping  message:Mapped URL path [/resources/**] onto handler 'org.springframework.web.servlet.resource.ResourceHttpRequestHandler#0'
    date:2014-08-25 12:35:36    thread:localhost-startStop-1    X-Track:    level:INFO  logger:o.springframework.web.servlet.DispatcherServlet  message:FrameworkServlet 'appServlet': initialization completed in 1862 ms

|

ブラウザでhttp://localhost:8080/todo
にアクセスすると、以下のように表示される。


 .. figure:: ./images/image034.png
   :width: 60%


コンソールを見ると ``TraceLoggingInterceptor`` によるTRACEログとControllerで実装したinfoログが出力されていることがわかる。

 .. code-block:: text


    date:2014-08-25 12:35:41    thread:tomcat-http--3   X-Track:94bc5651406e4333a4b47b2276707b5c    level:TRACE logger:o.t.gfw.web.logging.TraceLoggingInterceptor      message:[START CONTROLLER] HelloControllerr.home(Locale,Model)
    date:2014-08-25 12:35:41    thread:tomcat-http--3   X-Track:94bc5651406e4333a4b47b2276707b5c    level:INFO  logger:todo.app.welcome.HelloControllerr                  message:Welcome home! The client locale is en.
    date:2014-08-25 12:35:41    thread:tomcat-http--3   X-Track:94bc5651406e4333a4b47b2276707b5c    level:TRACE logger:o.t.gfw.web.logging.TraceLoggingInterceptor      message:[END CONTROLLER  ] HelloControllerr.home(Locale,Model)-> view=welcome/home, model={serverTime=August 25, 2014 12:35:41 PM UTC}
    date:2014-08-25 12:35:41    thread:tomcat-http--3   X-Track:94bc5651406e4333a4b47b2276707b5c    level:TRACE logger:o.t.gfw.web.logging.TraceLoggingInterceptor      message:[HANDLING TIME   ] HelloControllerr.home(Locale,Model)-> 41,090,439 ns

 .. note::
 
    ``TraceLoggingInterceptor`` はControllerの開始、終了でログを出力する。終了時にはViewとModelの情報および処理時間が出力される。

|

Todoアプリケーションの作成
================================================================================
| Todoアプリケーションを作成する。作成する順は、以下の通りである。

* ドメイン層(+ インフラストラクチャ層)

 * Domain Object作成
 * Repository作成
 * RepositoryImpl作成
 * Service作成

* アプリケーション層

 * Controller作成
 * Form作成
 * View作成

|

RepositoryImplの作成は、選択したインフラストラクチャ層の種類に応じて実装方法が異なる。

本節では、データベースを使用せずMapを使ったインメモリ実装のRepositoryImplを作成する方法について説明を行っているので、
データベースを使用する場合は、「:ref:`tutorial-todo_infra`」に記載されている内容で読み替えて、Todoアプリケーションを作成して頂きたい。

|

ドメイン層の作成
--------------------------------------------------------------------------------

Domain Objectの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ドメインオブジェクトに必要なプロパティは、

#. ID
#. タイトル
#. 完了フラグ
#. 作成日

である。

| Domainオブジェクトを作成する。
| FQCNは、\ ``todo.domain.model.Todo``\とする。

 .. figure:: ./images/image057.png
   :width: 60%


 .. code-block:: java

    package todo.domain.model;

    import java.io.Serializable;
    import java.util.Date;

    public class Todo implements Serializable {
        private static final long serialVersionUID = 1L;


        private String todoId;

        private String todoTitle;

        private boolean finished;

        private Date createdAt;

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
    }


 .. figure:: ./images/image058.png
   :width: 40%

 .. note::
    Getter/Setterは自動生成できる。フィールドを定義した後、右クリックで「Source」->「Generate Getter and Setters…」

        .. figure:: ./images/image059.png
           :width: 60%

    serialVersionUID以外を選択して「OK」

        .. figure:: ./images/image060.png
           :width: 60%

|

Repositoryの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
今回のアプリケーションで、必要なTODOオブジェクトに対するCRUD系操作は、

* TODOの1件取得
* TODOの全件取得
* TODOの1件削除
* TODOの1件更新
* 完了済みTODO件数の取得

である。

| これらの操作を定義するインタフェースTodoRepositoryを作成する。
| FQCNは、\ ``todo.domain.repository.todo.TodoRepository``\とする。

 .. code-block:: java

    package todo.domain.repository.todo;

    import java.util.Collection;

    import todo.domain.model.Todo;

    public interface TodoRepository {
        Todo findOne(String todoId);

        Collection<Todo> findAll();

        Todo save(Todo todo);

        void delete(Todo todo);

        long countByFinished(boolean finished);
    }

 .. figure:: ./images/image061.png
   :width: 40%

 .. note::

    ここでは、TodoRepositoryの汎用性を上げるため、「完了済み件数の取得する」メソッド(\ ``long countFinished()``\)ではなく、
    「完了状態がxである件数を取得する」メソッド(\ ``long countByFinished(boolean)``\)として定義している。
    
    \ ``long countByFinished(boolean)``\の引数として\ ``true``\を渡すと「完了済みの件数」、
    \ ``false``\を渡すと「未完了の件数」が取得できる仕様としている。

|

RepositoryImplの作成(インフラストラクチャ層)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| 本節では、説明を単純化するため、Mapを使ったインメモリ実装のRepositotyImplを作成する。
| データベースを使用する場合は、「:ref:`tutorial-todo_infra`」に記載されている内容で読み替えて、RepositoryImplを作成する。
| FQCNは、\ ``todo.domain.repository.todo.TodoRepositoryImpl``\とする。

 .. note::
 
    RepositoryImplには、業務ロジックは含めず、Domainオブジェクトの保存先への出し入れ(CRUD操作)に終始することが実装ポイントである。

 .. code-block:: java
    :emphasize-lines: 11

    package todo.domain.repository.todo;

    import java.util.Collection;
    import java.util.Map;
    import java.util.concurrent.ConcurrentHashMap;

    import org.springframework.stereotype.Repository;

    import todo.domain.model.Todo;

    @Repository // (1)
    public class TodoRepositoryImpl implements TodoRepository {
        private static final Map<String, Todo> TODO_MAP = new ConcurrentHashMap<String, Todo>();

        @Override
        public Todo findOne(String todoId) {
            return TODO_MAP.get(todoId);
        }

        @Override
        public Collection<Todo> findAll() {
            return TODO_MAP.values();
        }

        @Override
        public Todo save(Todo todo) {
            return TODO_MAP.put(todo.getTodoId(), todo);
        }

        @Override
        public void delete(Todo todo) {
            TODO_MAP.remove(todo.getTodoId());
        }

        @Override
        public long countByFinished(boolean finished) {
            long count = 0;
            for (Todo todo : TODO_MAP.values()) {
                if (finished == todo.isFinished()) {
                    count++;
                }
            }
            return count;
        }
    }

 .. figure:: ./images/image062.png
   :width: 40%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80

   * - 項番
     - 説明
   * - | (1)
     - | Repositoryとして、component-scan対象とするため、クラスレベルに\ ``@Repository``\ アノテーションをつける。

 .. note::
 
    本チュートリアルでは、インフラストラクチャ層に属するクラス(RepositoryImpl)をドメイン層のパッケージ(\ ``todo.domain``\)に格納しているが、
    完全に層別にパッケージを分けるのであれば、インフラストラクチャ層のクラスは、\ ``todo.infra``\以下に作成した方が良い。

    ただし、通常のプロジェクトでは、インフラストラクチャ層が変更されることを前提としていない(そのような前提で進めるプロジェクトは、少ない)。
    そこで、作業効率向上のために、ドメイン層のRepositotyインタフェースと同じ階層に、RepositoryImplを作成しても良い。

|

Serviceの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
業務処理を実装する。必要な処理は、

* Todoの全件取得
* Todoの新規作成
* Todoの完了
* Todoの削除

である。

| まずは、TodoServiceインタフェースを作成して、これらの処理を行うメソッドを定義する。
| FQCNは、\ ``todo.domain.serivce.todo.TodoService``\とする。

 .. code-block:: java

    package todo.domain.service.todo;

    import java.util.Collection;

    import todo.domain.model.Todo;

    public interface TodoService {
        Collection<Todo> findAll();

        Todo create(Todo todo);

        Todo finish(String todoId);

        void delete(String todoId);
    }

 .. figure:: ./images/image063.png
   :width: 40%

必要な処理と、実装するメソッドの対応は、以下の通りである。

* Todoの全件取得→findAllメソッド
* Todoの新規作成→createメソッド
* Todoの完了→finishメソッド
* Todoの削除→deleteメソッド


実装クラスのFQCNは、\ ``todo.domain.service.todo.TodoServiceImpl``\とする。

 .. code-block:: java
    :emphasize-lines: 19, 20, 25-26, 28-29, 32-33, 37-38, 44, 57-58, 61-62

    package todo.domain.service.todo;

    import java.util.Collection;
    import java.util.Date;
    import java.util.UUID;

    import javax.inject.Inject;

    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;
    import org.terasoluna.gfw.common.exception.BusinessException;
    import org.terasoluna.gfw.common.exception.ResourceNotFoundException;
    import org.terasoluna.gfw.common.message.ResultMessage;
    import org.terasoluna.gfw.common.message.ResultMessages;

    import todo.domain.model.Todo;
    import todo.domain.repository.todo.TodoRepository;

    @Service// (1)
    @Transactional // (2)
    public class TodoServiceImpl implements TodoService {

        private static final long MAX_UNFINISHED_COUNT = 5;

        @Inject// (3)
        TodoRepository todoRepository;

        // (4)
        public Todo findOne(String todoId) {
            Todo todo = todoRepository.findOne(todoId);
            if (todo == null) {
                // (5)
                ResultMessages messages = ResultMessages.error();
                messages.add(ResultMessage
                        .fromText("[E404] The requested Todo is not found. (id="
                                + todoId + ")"));
                // (6)
                throw new ResourceNotFoundException(messages);
            }
            return todo;
        }

        @Override
        @Transactional(readOnly = true) // (7)
        public Collection<Todo> findAll() {
            return todoRepository.findAll();
        }

        @Override
        public Todo create(Todo todo) {
            long unfinishedCount = todoRepository.countByFinished(false);
            if (unfinishedCount >= MAX_UNFINISHED_COUNT) {
                ResultMessages messages = ResultMessages.error();
                messages.add(ResultMessage
                        .fromText("[E001] The count of un-finished Todo must not be over "
                                + MAX_UNFINISHED_COUNT + "."));
                // (8)
                throw new BusinessException(messages);
            }

            // (9)
            String todoId = UUID.randomUUID().toString();
            Date createdAt = new Date();

            todo.setTodoId(todoId);
            todo.setCreatedAt(createdAt);
            todo.setFinished(false);

            todoRepository.save(todo);

            return todo;
        }

        @Override
        public Todo finish(String todoId) {
            Todo todo = findOne(todoId);
            if (todo.isFinished()) {
                ResultMessages messages = ResultMessages.error();
                messages.add(ResultMessage
                        .fromText("[E002] The requested Todo is already finished. (id="
                                + todoId + ")"));
                throw new BusinessException(messages);
            }
            todo.setFinished(true);
            todoRepository.save(todo);
            return todo;
        }

        @Override
        public void delete(String todoId) {
            Todo todo = findOne(todoId);
            todoRepository.delete(todo);
        }
    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80


   * - 項番
     - 説明
   * - | (1)
     - | Serviceとしてcomponent-scanの対象とするため、クラスレベルに\ ``@Service``\ アノテーションをつける。
   * - | (2)
     - | クラスレベルに、\ ``@Transactional``\ アノテーションをつけることで、公開メソッドをすべてトランザクション管理する。
       | アノテーションを付与することで、メソッド開始時にトランザクションを開始、メソッド正常終了時にトランザクションをコミットが行われる。
       | また、途中で非検査例外が発生した場合は、トランザクションをロールバックされる。
       |
       | データベースを使用しない場合は、\ ``@Transactional``\ アノテーションは不要である。
   * - | (3)
     - | \ ``@Inject``\ アノテーションで、TodoRepositoryの実装をインジェクションする。
   * - | (4)
     - | 1件取得は、finishメソッドでもdeleteメソッドでも使用するため、メソッドとして用意しておく(interfaceに公開しても良い)。
   * - | (5)
     - | 結果メッセージを格納するクラスとして、共通ライブラリで用意されているorg.terasoluna.gfw.common.message.ResultMessageを用いる。
       | 今回は、Errorメッセージをスローするために、ResultMessages.error()でメッセージ種別を指定して、ResultMessageを追加している。
   * - | (6)
     - | 対象のデータが存在しない場合、共通ライブラリで用意されているorg.terasoluna.gfw.common.exception.ResourceNotFoundExceptionをスローする。
   * - | (7)
     - | 参照のみ行う処理に関しては、readOnly=trueをつける。
       | O/R Mapperによっては、この設定により、参照時のトランザクション制御の最適化が行われる(JPAを使用する場合、効果はない)。
       |
       | データベースを使用しない場合は、\ ``@Transactional``\ アノテーションは不要である。
   * - | (8)
     - | 業務エラーが発生した場合、共通ライブラリで用意されているorg.terasoluna.gfw.common.exception.BusinessExceptionをスローする。
   * - | (9)
     - | 一意性のある値を生成するために、UUIDを使用している。データベースのシーケンスを用いてもよい。

 .. note::

    本節では、説明を単純化するため、エラーメッセージをハードコードしているが、メンテナンスの観点で本来は好ましくない。
    通常、メッセージは、プロパティファイルに外部化することが推奨される。
    プロパティファイルに外部化する方法は、\ :doc:`../ArchitectureInDetail/PropertyManagement`\ を参照されたい。

 .. figure:: ./images/image064.png
   :width: 40%

|

ServiceのJUnit作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

 .. todo:: **TBD**
 
    ServiceのUnitテストの方法については、次版以降で記載する予定である。

|

アプリケーション層の作成
--------------------------------------------------------------------------------

ドメイン層の実装が完了したので、次はドメイン層を利用して、アプリケーション層の作成に取り掛かる。

Controllerの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| まずは、todo管理業務にかかわる画面遷移を、制御するControllerを作成する。
| FQCNは、\ ``todo.app.todo.TodoController``\とする。
| **上位パッケージがドメイン層とは異なるので注意すること。**

 .. code-block:: java
    :emphasize-lines: 6, 7

    package todo.app.todo;

    import org.springframework.stereotype.Controller;
    import org.springframework.web.bind.annotation.RequestMapping;

    @Controller // (1)
    @RequestMapping("todo") // (2)
    public class TodoController {

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80


   * - 項番
     - 説明
   * - | (1)
     - | Controllerとしてcomponent-scanの対象とするため、クラスレベルに、\ ``@Controller``\ アノテーションをつける。
   * - | (2)
     - | TodoControllerが扱う画面遷移のパスを、すべて<contextPath>/todo配下にするため、クラスレベルに@RequestMapping(“todo”)を設定する。


 .. figure:: ./images/image065.png
   :width: 40%

|

Show all TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
この画面では、

* 新規作成フォームの表示
* TODOの全件表示

を行う。

Formの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Formには、タイトル情報があればよいので、次のようなJavaBeanになる。
| FQCNは、\ ``todo.app.todo.TodoForm``\とする。

 .. code-block:: java

    package todo.app.todo;

    import java.io.Serializable;

    public class TodoForm implements Serializable {
        private static final long serialVersionUID = 1L;

        private String todoTitle;

        public String getTodoTitle() {
            return todoTitle;
        }

        public void setTodoTitle(String todoTitle) {
            this.todoTitle = todoTitle;
        }

    }

 .. figure:: ./images/image066.png
   :width: 40%

Controllerの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
TodoControllerに、setUpFormメソッドと、listメソッドを実装する。

 .. code-block:: java
    :emphasize-lines: 18-19, 21-22, 27, 30, 31

    package todo.app.todo;

    import java.util.Collection;

    import javax.inject.Inject;

    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.RequestMapping;

    import todo.domain.model.Todo;
    import todo.domain.service.todo.TodoService;

    @Controller
    @RequestMapping("todo")
    public class TodoController {
        @Inject // (3)
        TodoService todoService;

        @ModelAttribute // (4)
        public TodoForm setUpForm() {
            TodoForm form = new TodoForm();
            return form;
        }

        @RequestMapping(value = "list") // (5)
        public String list(Model model) {
            Collection<Todo> todos = todoService.findAll();
            model.addAttribute("todos", todos); // (6)
            return "todo/list"; // (7)
        }
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80

   * - 項番
     - 説明
   * - | (3)
     - | TodoServiceを、DIコンテナによってインジェクションさせるために、\ ``@Inject``\ アノテーションをつける。
       | DIコンテナの管理するTodoSerivce型インスタンスがインジェクションされるため、結果として、TodoServiceImplインスタンスがインジェクションされる。
   * - | (4)
     - | Formを初期化する。\ ``@ModelAttribute``\ アノテーションをつけることで、このメソッドの返り値のformオブジェクトが、”todoForm”という名前でModelに追加される。
       | TodoControllerの各処理で、model.addAttribute(“todoForm”, form)が実行されるのと同義。
   * - | (5)
     - | listメソッドを”<contextPath>/todo/list”にマッピングされるための設定。クラスレベルで@RequestMapping(“todo”)が設定されているため、ここでは@RequestMapping(value = “list”)だけで良い。
   * - | (6)
     - | ModelにTodoのリストを追加して、Viewに渡す。
   * - | (7)
     - | View名として”todo/list”を返すと、spring-mvc.xmlに定義したInternalResourceViewResolverによって、\ :file:`WEB-INF/views/todo/list.jsp`\がレンダリングされることになる。

JSPの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\ :file:`src/main/webapp/WEB-INF/views/todo/list.jsp`\を作成し、Controllerから渡されたModelを表示する。

まずは、”Finish”,”Delete”ボタン以外を作成する。

 .. code-block:: jsp
    :emphasize-lines: 15, 19-20, 27-28, 30, 32-33

    <!DOCTYPE html>
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Todo List</title>
    <style type="text/css">
    .strike {
        text-decoration: line-through;
    }
    </style>
    </head>
    <body>
        <h1>Todo List</h1>
        <div id="todoForm">
            <!-- (1) -->
            <form:form
               action="${pageContext.request.contextPath}/todo/create"
                method="post" modelAttribute="todoForm">
                <!-- (2) -->
                <form:input path="todoTitle" />
                <input type="submit" value="Create Todo" />
            </form:form>
        </div>
        <hr />
        <div id="todoList">
            <ul>
                <!-- (3) -->
                <c:forEach items="${todos}" var="todo">
                    <li><c:choose>
                            <c:when test="${todo.finished}"><!-- (4) -->
                                <span class="strike">
                                <!-- (5) -->
                                ${f:h(todo.todoTitle)}
                                </span>
                            </c:when>
                            <c:otherwise>
                                ${f:h(todo.todoTitle)}
                             </c:otherwise>
                        </c:choose></li>
                </c:forEach>
            </ul>
        </div>
    </body>
    </html>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80

   * - 項番
     - 説明
   * - | (1)
     - | <form:form>タグでフォームを表示する。modelAttribute属性に、ControllerでModelに追加したformの名前を指定する。
       | action属性に指定するcontextPathは、${pageContext.request.contextPath}で取得できる。
   * - | (2)
     - | <form:input>タグでフォームのプロパティをバインドする。modelAttribute属性に指定したformのプロパティ名と、path属性の値が一致している必要がある。
   * - | (3)
     - | <c:forEach>タグを用いて、Todoのリストを全て表示する。
   * - | (4)
     - | 完了かどうか(finished)で、打ち消し線(text-decoration: line-through;)を装飾するかどうかを判断する。
   * - | (5)
     - | **文字列値を出力する際は、XSS対策のため、必ずf:h()関数を使用してHTMLエスケープを行うこと。**
       | XSS対策についての詳細は、\ :doc:`../Security/XSS`\ を参照されたい。

| STSで「todo」プロジェクトを右クリックし、「Run As」→「Run on Server」でWebアプリケーションを起動する。
| ブラウザで”http://localhost:8080/todo/todo/list”にアクセスすると、以下のような画面が表示される。


 .. figure:: ./images/image067.png
   :width: 40%


Create TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
次に、一覧表示画面から”Create TODO”ボタンを押した後の、新規作成処理を実装する。

Controllerの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\ ``TodoController``\に、createメソッドを追加する。

 .. code-block:: java
    :emphasize-lines: 8,29-31,46-70

    package todo.app.todo;

    import java.util.Collection;

    import javax.inject.Inject;
    import javax.validation.Valid;

    import org.dozer.Mapper;
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.validation.BindingResult;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.servlet.mvc.support.RedirectAttributes;
    import org.terasoluna.gfw.common.exception.BusinessException;
    import org.terasoluna.gfw.common.message.ResultMessage;
    import org.terasoluna.gfw.common.message.ResultMessages;

    import todo.domain.model.Todo;
    import todo.domain.service.todo.TodoService;

    @Controller
    @RequestMapping("todo")
    public class TodoController {
        @Inject
        TodoService todoService;

        // (8)
        @Inject
        Mapper beanMapper;

        @ModelAttribute
        public TodoForm setUpForm() {
            TodoForm form = new TodoForm();
            return form;
        }

        @RequestMapping(value = "list")
        public String list(Model model) {
            Collection<Todo> todos = todoService.findAll();
            model.addAttribute("todos", todos);
            return "todo/list";
        }

        @RequestMapping(value = "create", method = RequestMethod.POST) // (9)
        public String create(@Valid TodoForm todoForm, BindingResult bindingResult, // (10)
                Model model, RedirectAttributes attributes) { // (11)

            // (12)
            if (bindingResult.hasErrors()) {
                return list(model);
            }

            // (13)
            Todo todo = beanMapper.map(todoForm, Todo.class);

            try {
                todoService.create(todo);
            } catch (BusinessException e) {
                // (14)
                model.addAttribute(e.getResultMessages());
                return list(model);
            }

            // (15)
            attributes.addFlashAttribute(ResultMessages.success().add(
                    ResultMessage.fromText("Created successfully!")));
            return "redirect:/todo/list";
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80


   * - 項番
     - 説明
   * - | (8)
     - | Formオブジェクトを、DomainObjectに変換する際に、有用なMapperをインジェクションする。
   * - | (9)
     - | パスが/todo/createで、HTTPメソッドがPOSTに対応するように、\ ``@RequestMapping``\ アノテーションを設定する。
   * - | (10)
     - | フォームの入力チェックを行うため、Formの引数に\ ``@Valid``\ アノテーションをつける。入力チェック結果は、その直後の引数BindingResultに格納される。
   * - | (11)
     - | 正常に作成が完了した後、リダイレクトし、一覧画面を表示する。リダイレクト先への情報を格納するために、引数にRedirectAttributesを加える。
   * - | (12)
     - | 入力エラーがあった場合、一覧画面に戻る。Todo全件取得を再度行う必要があるので、listメソッドを再実行する。
   * - | (13)
     - | Mapperを用いて、TodoFormからTodoオブジェクトを作成する。変換元と変換先のプロパティ名が同じ場合は、設定不要である。
       | 今回は、todoTitleプロパティのみ変換するため、Mapperを使用するメリットはほとんどない。プロパティの数が多い場合には、非常に便利である。
   * - | (14)
     - | 業務処理を実行して、BusinessExceptionが発生した場合、結果メッセージをModelに追加して、一覧画面に戻る。
   * - | (15)
     - | 正常に作成が完了したので、結果メッセージをflashスコープに追加して、一覧画面でリダイレクトする。
       | リダイレクトすることにより、ブラウザを再読み込みして、再び新規登録処理がPOSTされることがなくなる。なお、今回は成功メッセージであるため、ResultMessages.success()を使用している。


Formの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
入力チェックのルールを定義するため、Formオブジェクトにアノテーションを追加する。

 .. code-block:: java
    :emphasize-lines: 5-6,11-12

    package todo.app.todo;

    import java.io.Serializable;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;

    public class TodoForm implements Serializable {
        private static final long serialVersionUID = 1L;

        @NotNull // (1)
        @Size(min = 1, max = 30) // (2)
        private String todoTitle;

        public String getTodoTitle() {
            return todoTitle;
        }

        public void setTodoTitle(String todoTitle) {
            this.todoTitle = todoTitle;
        }
    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80


   * - 項番
     - 説明
   * - | (1)
     - | 必須項目であるので、\ ``@NotNull``\ アノテーションを付ける。
   * - | (2)
     - | 1文字以上30文字以下であるので、\ ``@Size``\ アノテーションで、範囲を指定する。

JSPの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
結果メッセージ表示用のタグを追加する。

 .. code-block:: jsp
    :emphasize-lines: 15-16,22

    <!DOCTYPE html>
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Todo List</title>
    <style type="text/css">
    .strike {
        text-decoration: line-through;
    }
    </style>
    </head>
    <body>
        <h1>Todo List</h1>
        <div id="todoForm">
            <!-- (6) -->
            <t:messagesPanel />

            <form:form
               action="${pageContext.request.contextPath}/todo/create"
                method="post" modelAttribute="todoForm">
                <form:input path="todoTitle" />
                <form:errors path="todoTitle" /><!-- (7) -->
                <input type="submit" value="Create Todo" />
            </form:form>
        </div>
        <hr />
        <div id="todoList">
            <ul>
                <c:forEach items="${todos}" var="todo">
                    <li><c:choose>
                            <c:when test="${todo.finished}">
                                <span style="text-decoration: line-through;">
                                ${f:h(todo.todoTitle)}
                                </span>
                            </c:when>
                            <c:otherwise>
                                ${f:h(todo.todoTitle)}
                             </c:otherwise>
                        </c:choose></li>
                </c:forEach>
            </ul>
        </div>
    </body>
    </html>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80


   * - 項番
     - 説明
   * - | (6)
     - | <t:messagesPanel>タグで、結果メッセージを表示する。
   * - | (7)
     - | <form:errors>タグで、入力エラーがあった場合に表示する。path属性の値は、<form:input>タグと合わせる。


フォームに適切な値を入力してsubmitすると、以下のように、成功メッセージが表示される。


 .. figure:: ./images/image068.png
   :width: 40%


 .. figure:: ./images/image069.png
   :width: 40%



6件以上登録した場合は、業務エラーとなり、エラーメッセージが表示される。

 .. figure:: ./images/image070.png
   :width: 40%


入力フォームを、空文字にしてsubmitすると、以下のように、エラーメッセージが表示される。


 .. figure:: ./images/image071.png
   :width: 40%

メッセージ表示のカスタマイズ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
<t:messagesPanel>の結果はデフォルトで、

 .. code-block:: html

    <div class="alert alert-success"><ul><li>Created successfully!</li></ul></div>


と出力される。
スタイルシート(list.jspの<style>タグ内)に、以下の修正を加えて、結果メッセージの見た目をカスタマイズする。

 .. code-block:: css

    .alert {
        border: 1px solid;
    }

    .alert-error {
        background-color: #c60f13;
        border-color: #970b0e;
        color: white;
    }

    .alert-success {
        background-color: #5da423;
        border-color: #457a1a;
        color: white;
    }


メッセージは、以下のように装飾される。



 .. figure:: ./images/image072.png
   :width: 40%



 .. figure:: ./images/image073.png
   :width: 40%


また、<form:errors>タグのcssClass属性で、入力エラーメッセージのclassを指定できる。JSPを次のように修正し、

 .. code-block:: html

    <form:errors path="todoTitle" cssClass="text-error" />


スタイルシートに、以下を追加する。

 .. code-block:: css

    .text-error {
        color: #c60f13;
    }


入力エラーは、以下のように装飾される。


 .. figure:: ./images/image074.png
   :width: 40%

|

Finish TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

一覧表示画面に”Finish”ボタンを追加して、ボタンをsubmitすると、hiddenで対象のtodoIdが送られ、Todoを完了するように実装する。


JSPの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

完了用のformを追加する。


 .. code-block:: jsp
    :emphasize-lines: 56-67

    <!DOCTYPE html>
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Todo List</title>
    </head>
    <style type="text/css">
    .strike {
        text-decoration: line-through;
    }

    .alert {
        border: 1px solid;
    }

    .alert-error {
        background-color: #c60f13;
        border-color: #970b0e;
        color: white;
    }

    .alert-success {
        background-color: #5da423;
        border-color: #457a1a;
        color: white;
    }

    .text-error {
        color: #c60f13;
    }
    </style>
    <body>
        <h1>Todo List</h1>

        <div id="todoForm">
            <t:messagesPanel />

            <form:form
                action="${pageContext.request.contextPath}/todo/create"
                method="post" modelAttribute="todoForm">
                <form:input path="todoTitle" />
                <form:errors path="todoTitle" cssClass="text-error" />
                <input type="submit" value="Create Todo" />
            </form:form>
        </div>
        <hr />
        <div id="todoList">
            <ul>
                <c:forEach items="${todos}" var="todo">
                    <li><c:choose>
                            <c:when test="${todo.finished}">
                                <span class="strike">${f:h(todo.todoTitle)}</span>
                            </c:when>
                            <c:otherwise>
                                ${f:h(todo.todoTitle)}
                                <!-- (8) -->
                                <form:form
                                    action="${pageContext.request.contextPath}/todo/finish"
                                    method="post"
                                    modelAttribute="todoForm"
                                    cssStyle="display: inline-block;">
                                    <!-- (9) -->
                                    <form:hidden path="todoId"
                                        value="${f:h(todo.todoId)}" />
                                    <input type="submit" name="finish"
                                        value="Finish" />
                                </form:form>
                            </c:otherwise>
                        </c:choose></li>
                </c:forEach>
            </ul>
        </div>
    </body>
    </html>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80


   * - 項番
     - 説明
   * - | (8)
     - | 未完了の場合に、完了用のformを表示する。<contextPath>/todo/finishに対して、POSTでtodoIdを送信する。
   * - | (9)
     - | <form:hidden>タグでtodoIdを渡す。value属性に値を設定する場合も、 **必ずf:h()関数でHTMLエスケープすること。**

Formの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
完了用のフォームも、TodoFormを用いる。
TodoFormに、todoIdプロパティを追加する必要があるが、そのままだと、新規作成用の入力チェックルールが適用されてしまう。
一つのFormに、新規作成用と完了用で、別々のルールを指定するために、group属性を設定する。

 .. code-block:: java
    :emphasize-lines: 9-11,13-14,18-20,22-23,27-33

    package todo.app.todo;

    import java.io.Serializable;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;

    public class TodoForm implements Serializable {
        // (3)
        public static interface TodoCreate {
        };

        public static interface TodoFinish {
        };

        private static final long serialVersionUID = 1L;

        // (4)
        @NotNull(groups = { TodoFinish.class })
        private String todoId;

        // (5)
        @NotNull(groups = { TodoCreate.class })
        @Size(min = 1, max = 30, groups = { TodoCreate.class })
        private String todoTitle;

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

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80


   * - 項番
     - 説明
   * - | (3)
     - | グループ化したバリデーションを行うためのグループ名となるクラスを作成する。クラスは空でよいため、ここでは、インタフェースを定義する。
       | グループ化バリデーションについては、\ :doc:`../ArchitectureInDetail/Validation`\ を参照されたい。
   * - | (4)
     - | todoIdは、完了処理には必須であるため、\ ``@NotNull``\ アノテーションをつける。完了時にのみ必要なルールであるので、group属性にTodoFinish.classを設定する。
   * - | (5)
     - | 新規作成用のルールは、完了処理には不要であるので、\ ``@NotNull``\ アノテーション、\ ``@Size``\ アノテーション、それぞれのgroup属性にTodoCreate.classを設定する。

Controllerの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

完了処理をTodoControllerに追加する。
グループ化したバリデーションを実行するために、\ **@Valid アノテーションの代わりに、@Validated アノテーションを使用すること**\ に注意する。

 .. code-block:: java
    :emphasize-lines: 6,12,50,72-94

    package todo.app.todo;

    import java.util.Collection;

    import javax.inject.Inject;
    import javax.validation.groups.Default;

    import org.dozer.Mapper;
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.validation.BindingResult;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.servlet.mvc.support.RedirectAttributes;
    import org.terasoluna.gfw.common.exception.BusinessException;
    import org.terasoluna.gfw.common.message.ResultMessage;
    import org.terasoluna.gfw.common.message.ResultMessages;

    import todo.app.todo.TodoForm.TodoCreate;
    import todo.app.todo.TodoForm.TodoFinish;
    import todo.domain.model.Todo;
    import todo.domain.service.todo.TodoService;

    @Controller
    @RequestMapping("todo")
    public class TodoController {
        @Inject
        TodoService todoService;

        @Inject
        Mapper beanMapper;

        @ModelAttribute
        public TodoForm setUpForm() {
            TodoForm form = new TodoForm();
            return form;
        }

        @RequestMapping(value = "list")
        public String list(Model model) {
            Collection<Todo> todos = todoService.findAll();
            model.addAttribute("todos", todos);
            return "todo/list";
        }

        @RequestMapping(value = "create", method = RequestMethod.POST)
        public String create(
                @Validated({ Default.class, TodoCreate.class }) TodoForm todoForm, // (16)
                BindingResult bindingResult, Model model,
                RedirectAttributes attributes) {

            if (bindingResult.hasErrors()) {
                return list(model);
            }

            Todo todo = beanMapper.map(todoForm, Todo.class);

            try {
                todoService.create(todo);
            } catch (BusinessException e) {
                model.addAttribute(e.getResultMessages());
                return list(model);
            }

            attributes.addFlashAttribute(ResultMessages.success().add(
                    ResultMessage.fromText("Created successfully!")));
            return "redirect:/todo/list";
        }

        @RequestMapping(value = "finish", method = RequestMethod.POST) // (17)
        public String finish(
                @Validated({ Default.class, TodoFinish.class }) TodoForm form, // (18)
                BindingResult bindingResult, Model model,
                RedirectAttributes attributes) {
            // (19)
            if (bindingResult.hasErrors()) {
                return list(model);
            }

            try {
                todoService.finish(form.getTodoId());
            } catch (BusinessException e) {
                // (20)
                model.addAttribute(e.getResultMessages());
                return list(model);
            }

            // (21)
            attributes.addFlashAttribute(ResultMessages.success().add(
                    ResultMessage.fromText("Finished successfully!")));
            return "redirect:/todo/list";
        }
    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80


   * - 項番
     - 説明
   * - | (16)
     - | グループ化したバリデーションを実施するために、\ ``@Valid``\ アノテーションから\ ``@Validated``\ アノテーションに変更する。
       | valueには、対象のグループクラスを複数指定できる。Default.classはバリデーションルールにgroupが指定されていない場合のグループである。
       | \ ``@Validated``\ アノテーションを使用する際は、Default.classも指定しておくのがよい。
   * - | (17)
     - | パスが、/todo/finishで、HTTPメソッドがPOSTに対応するように、\ ``@RequestMapping``\ アノテーションを設定する。
   * - | (18)
     - | Finish用のグループとして、TodoFinish.classを指定する。
   * - | (19)
     - | 入力エラーがあった場合、一覧画面に戻る。
   * - | (20)
     - | 業務処理を実行して、BusinessExceptionが発生した場合は、結果メッセージをModelに追加して、一覧画面に戻る。
   * - | (21)
     - | 正常に作成が完了したので、結果メッセージをflashスコープに追加して、一覧画面でリダイレクトする。

 .. note::

    Create用、Finish用に、別々のFormを作成しても良い。その場合は、必要なパラメータだけが、Formのプロパティになる。
    ただし、クラス数が増え、プロパティも重複することが多いので、仕様変更が発生した場合に、修正コストが高くなる。
    また、同一のController内で、複数のFormオブジェクトを、 ``@ModelAttribute`` メソッドによって初期化すると、
    毎回すべてのFormが初期化されてしまうので、不要なインスタンスが生成されてしまう。そのため、
    基本的に、一つのControllerで利用するFormは、できるだけ集約し、グループ化したバリデーションの設定を行うことを推奨する。


Todoを新規作成した後に、FinishボタンをSubmitすると、以下のように打ち消し線が入り、完了したことがわかる。


 .. figure:: ./images/image075.png
   :width: 40%


 .. figure:: ./images/image076.png
   :width: 40%

|

Delete TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
一覧表示画面に"Delete"ボタンを追加して、ボタンをsubmitすると、hiddenで対象のtodoIdが送られ、Todoを完了するように実装する。

JSPの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
削除用のformを追加する。


 .. code-block:: jsp
    :emphasize-lines: 68-77

    <!DOCTYPE html>
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Todo List</title>
    </head>
    <style type="text/css">
    .strike {
        text-decoration: line-through;
    }

    .alert {
        border: 1px solid;
    }

    .alert-error {
        background-color: #c60f13;
        border-color: #970b0e;
        color: white;
    }

    .alert-success {
        background-color: #5da423;
        border-color: #457a1a;
        color: white;
    }

    .text-error {
        color: #c60f13;
    }
    </style>
    <body>
        <h1>Todo List</h1>

        <div id="todoForm">
            <t:messagesPanel />

            <form:form
                action="${pageContext.request.contextPath}/todo/create"
                method="post" modelAttribute="todoForm">
                <form:input path="todoTitle" />
                <form:errors path="todoTitle" cssClass="text-error" />
                <input type="submit" value="Create Todo" />
            </form:form>
        </div>
        <hr />
        <div id="todoList">
            <ul>
                <c:forEach items="${todos}" var="todo">
                    <li><c:choose>
                            <c:when test="${todo.finished}">
                                <span class="strike">${f:h(todo.todoTitle)}</span>
                            </c:when>
                            <c:otherwise>
                                ${f:h(todo.todoTitle)}
                                <form:form
                                    action="${pageContext.request.contextPath}/todo/finish"
                                    method="post"
                                    modelAttribute="todoForm"
                                    cssStyle="display: inline-block;">
                                    <form:hidden path="todoId"
                                        value="${f:h(todo.todoId)}" />
                                    <input type="submit" name="finish"
                                        value="Finish" />
                                </form:form>
                            </c:otherwise>
                        </c:choose>
                        <!-- (10) -->
                        <form:form
                            action="${pageContext.request.contextPath}/todo/delete"
                            method="post" modelAttribute="todoForm"
                            cssStyle="display: inline-block;">
                            <!-- (11) -->
                            <form:hidden path="todoId"
                                value="${f:h(todo.todoId)}" />
                            <input type="submit" value="Delete" />
                        </form:form>
                    </li>
                </c:forEach>
            </ul>
        </div>
    </body>
    </html>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80


   * - 項番
     - 説明
   * - | (10)
     - | 削除用のformを表示する。<contextPath>/todo/deleteに対して、POSTでtodoIdを送信する。
   * - | (11)
     - | <form:hidden>タグで、todoIdを渡す。value属性に値を設定する場合も、\ **必ずf:h()関数でHTMLエスケープすること。**\

Formの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Delete用のグループを、TodoFormに追加する。ルールは、Finish用と同じである。


 .. code-block:: java
    :emphasize-lines: 15-17,21-22

    package todo.app.todo;

    import java.io.Serializable;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;

    public class TodoForm implements Serializable {
        public static interface TodoCreate {
        };

        public static interface TodoFinish {
        };

        // (6)
        public static interface TodoDelete {
        }

        private static final long serialVersionUID = 1L;

        // (7)
        @NotNull(groups = { TodoFinish.class, TodoDelete.class })
        private String todoId;

        @NotNull(groups = { TodoCreate.class })
        @Size(min = 1, max = 30, groups = { TodoCreate.class })
        private String todoTitle;

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

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80


   * - 項番
     - 説明
   * - | (6)
     - | Delete用のグループTodoDeleteを定義する。
   * - | (7)
     - | todoIdプロパティに対して、TodoDeleteグループのバリデーションを行うように設定する。

Controllerの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

削除処理を、TodoControllerに追加する。完了処理とほぼ同じである。

 .. code-block:: java
    :emphasize-lines: 94-114

    package todo.app.todo;

    import java.util.Collection;

    import javax.inject.Inject;
    import javax.validation.groups.Default;

    import org.dozer.Mapper;
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.validation.BindingResult;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.servlet.mvc.support.RedirectAttributes;
    import org.terasoluna.gfw.common.exception.BusinessException;
    import org.terasoluna.gfw.common.message.ResultMessage;
    import org.terasoluna.gfw.common.message.ResultMessages;

    import todo.app.todo.TodoForm.TodoDelete;
    import todo.app.todo.TodoForm.TodoCreate;
    import todo.app.todo.TodoForm.TodoFinish;
    import todo.domain.model.Todo;
    import todo.domain.service.todo.TodoService;

    @Controller
    @RequestMapping("todo")
    public class TodoController {
        @Inject
        TodoService todoService;

        @Inject
        Mapper beanMapper;

        @ModelAttribute
        public TodoForm setUpForm() {
            TodoForm form = new TodoForm();
            return form;
        }

        @RequestMapping(value = "list")
        public String list(Model model) {
            Collection<Todo> todos = todoService.findAll();
            model.addAttribute("todos", todos);
            return "todo/list";
        }

        @RequestMapping(value = "create", method = RequestMethod.POST)
        public String create(
                @Validated({ Default.class, TodoCreate.class }) TodoForm todoForm,
                BindingResult bindingResult, Model model,
                RedirectAttributes attributes) {

            if (bindingResult.hasErrors()) {
                return list(model);
            }

            Todo todo = beanMapper.map(todoForm, Todo.class);

            try {
                todoService.create(todo);
            } catch (BusinessException e) {
                model.addAttribute(e.getResultMessages());
                return list(model);
            }

            attributes.addFlashAttribute(ResultMessages.success().add(
                    ResultMessage.fromText("Created successfully!")));
            return "redirect:/todo/list";
        }

        @RequestMapping(value = "finish", method = RequestMethod.POST)
        public String finish(
                @Validated({ Default.class, TodoFinish.class }) TodoForm form,
                BindingResult bindingResult, Model model,
                RedirectAttributes attributes) {
            if (bindingResult.hasErrors()) {
                return list(model);
            }

            try {
                todoService.finish(form.getTodoId());
            } catch (BusinessException e) {
                model.addAttribute(e.getResultMessages());
                return list(model);
            }

            attributes.addFlashAttribute(ResultMessages.success().add(
                    ResultMessage.fromText("Finished successfully!")));
            return "redirect:/todo/list";
        }

        @RequestMapping(value = "delete", method = RequestMethod.POST) // (22)
        public String delete(
                @Validated({ Default.class, TodoDelete.class }) TodoForm form,
                BindingResult bindingResult, Model model,
                RedirectAttributes attributes) {

            if (bindingResult.hasErrors()) {
                return list(model);
            }

            try {
                todoService.delete(form.getTodoId());
            } catch (BusinessException e) {
                model.addAttribute(e.getResultMessages());
                return list(model);
            }

            attributes.addFlashAttribute(ResultMessages.success().add(
                    ResultMessage.fromText("Deleted successfully!")));
            return "redirect:/todo/list";
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80


   * - 項番
     - 説明
   * - | (22)
     - | Todoに対して、”Delete”ボタンをsubmitすると、以下のように、対象のTODOが削除される。


 .. figure:: ./images/image077.png
   :width: 40%


 .. figure:: ./images/image078.png
   :width: 40%

|

.. _tutorial-todo_infra:

データベースアクセスを伴うインフラストラクチャ層の作成
================================================================================

本節では、Domainオブジェクトをデータベースに永続化するためのインフラストラクチャ層の作成方法について、説明する。

本チュートリアルでは、以下の2つのO/R Mapperを使用したインフラストラクチャ層の作成について、説明する。

* Spring Data JPA
* TERASOLUNA DAO(MyBatis2)


共通設定
--------------------------------------------------------------------------------

| まずは、Spring Data JPA版、TERASOLUNA Dao版の両方に共通して適用する設定を行う。
| 今回は、データベースのセットアップの手間を省くため、H2 Databaseを使用する。

todo-infra.propertiesの修正
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
APサーバ起動時にH2 Database上にテーブルが作成されるようにするために、
\ :file:`src/main/resources/META-INF/spring/todo-infra.properties`\の設定を変更する。

 .. code-block:: properties
    :emphasize-lines: 2-3

    database=H2
    # (1)
    database.url=jdbc:h2:mem:todo;DB_CLOSE_DELAY=-1;INIT=create table if not exists todo(todo_id varchar(36) primary key, todo_title varchar(30), finished boolean, created_at timestamp)
    database.username=sa
    database.password=
    database.driverClassName=org.h2.Driver
    # connection pool
    cp.maxActive=96
    cp.maxIdle=16
    cp.minIdle=0
    cp.maxWait=60000

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80


   * - 項番
     - 説明
   * - | (1)
     - | 接続URLのINITパラメータに、テーブルを作成するDDL文を指定する。
     
 .. note::
 
    INITパラメータに設定しているDDL文は以下の通り。
    
     .. code-block:: sql

        create table if not exists todo(
            todo_id varchar(36) primary key,
            todo_title varchar(30),
            finished boolean,
            created_at timestamp
        )

|

Spring Data JPAを使用したインフラストラクチャ層の作成
--------------------------------------------------------------------------------

本節では、インフラストラクチャ層において、 `Spring Data JPA <http://www.springsource.org/spring-data/jpa>`_ を使用する方法について、説明する。
TERASOLUNA DAOを使用する場合は、本節を読み飛ばして、\ :ref:`using_terasolunaDao`\ に進んでよい。

Entityの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Todoクラスを、データベースとマッピングするために、JPAのアノテーションを設定する。

 .. code-block:: java
    :emphasize-lines: 6-11,13-15,19-22,25,28,31,33

    package todo.domain.model;

    import java.io.Serializable;
    import java.util.Date;

    import javax.persistence.Column;
    import javax.persistence.Entity;
    import javax.persistence.Id;
    import javax.persistence.Table;
    import javax.persistence.Temporal;
    import javax.persistence.TemporalType;

    // (1)
    @Entity
    @Table(name = "todo")
    public class Todo implements Serializable {
        private static final long serialVersionUID = 1L;

        // (2)
        @Id
        // (3)
        @Column(name = "todo_id")
        private String todoId;

        @Column(name = "todo_title")
        private String todoTitle;

        @Column(name = "finished")
        private boolean finished;

        @Column(name = "created_at")
        // (4)
        @Temporal(TemporalType.TIMESTAMP)
        private Date createdAt;

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
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | JPAのエンティティであることを示す\ ``@Entity``\ アノテーションを付け、対応するテーブル名を\ ``@Table``\ アノテーションで設定する。
   * - | (2)
     - | 主キーとなるカラムに対応するフィールドに、\ ``@Id``\ アノテーションをつける。
   * - | (3)
     - | \ ``@Column``\ アノテーションで、対応するカラム名を設定する。
   * - | (4)
     - | Date型は、java.sql.Date, java.sql.Time, java.sql.Timestampのどれに対応するか、明示的に指定する必要がある。ここでは、Timestampを指定する。


TodoRepositoryの修正
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Data JPAのRepository機能を使用するための修正を行う。

 .. code-block:: java
    :emphasize-lines: 3-5,9-10,12,13

    package todo.domain.repository.todo;

    import org.springframework.data.jpa.repository.JpaRepository;
    import org.springframework.data.jpa.repository.Query;
    import org.springframework.data.repository.query.Param;

    import todo.domain.model.Todo;

    // (1)
    public interface TodoRepository extends JpaRepository<Todo, String> {

        @Query(value = "SELECT COUNT(x) FROM Todo x WHERE x.finished = :finished") // (2)
        long countByFinished(@Param("finished") boolean finished); // (3)

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | JpaRepositoryを拡張したインタフェースにする。Genericsのパラメータには、順にEntityのクラス(Todo)、主キーのクラス(String)を指定する。
       | 基本的なCRUD操作(findOne, findAll, save, deleteなど)は、上位のインタフェースに定義済みであるため、TodoRepositoryでは、countByFinishedのみ定義すればよい。
   * - | (2)
     - | countByFinishedを呼び出した際に、実行されるJPQLを、\ ``@Query``\ アノテーションで指定する。
   * - | (3)
     - | (2)で指定したJPQLのバインド変数を\ ``@Param``\ アノテーションで設定する。
       | ここでは、JPQL中の\ ``”:finished”``\ を埋めるためのメソッド引数に、@Param(“finished”)を付けている。


TodoRepositoryImplの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Spring Data JPAを使用した場合、RepositoryImplは、インタフェースから自動生成される。
| そのため、TodoRepositoryImplの作成は、不要である。

|

以上で、Spring Data JPAを使用したインフラストラクチャ層の作成が完了したので、Service及びアプリケーション層の作成を行う。

Service及びアプリケーション層を作成後にAPサーバーを起動し、Todoの表示を行うと、以下のようなSQLログや、トランザクションログが出力される。

 .. code-block:: text
   :emphasize-lines: 2-6

    date:2014-08-26 12:57:48    thread:tomcat-http--3   X-Track:2705c06e0eae416591bc1f27efc84cd6    level:TRACE logger:o.t.gfw.web.logging.TraceLoggingInterceptor      message:[START CONTROLLER] TodoController.list(Model)
    date:2014-08-26 12:57:48    thread:tomcat-http--3   X-Track:2705c06e0eae416591bc1f27efc84cd6    level:DEBUG logger:o.h.e.transaction.spi.AbstractTransactionImpl    message:begin
    date:2014-08-26 12:57:48    thread:tomcat-http--3   X-Track:2705c06e0eae416591bc1f27efc84cd6    level:DEBUG logger:o.h.e.transaction.internal.jdbc.JdbcTransaction  message:initial autocommit status: false
    date:2014-08-26 12:57:48    thread:tomcat-http--3   X-Track:2705c06e0eae416591bc1f27efc84cd6    level:DEBUG logger:jdbc.sqltiming                                   message: org.hibernate.engine.jdbc.internal.ResultSetReturnImpl.extract(ResultSetReturnImpl.java:56)
    2. /* select generatedAlias0 from Todo as generatedAlias0 */ select todo0_.todo_id as todo1_0_, todo0_.created_at as created2_0_, todo0_.finished as finished3_0_, todo0_.todo_title as todo4_0_ from todo todo0_ {executed in 5 msec}
    date:2014-08-26 12:57:48    thread:tomcat-http--3   X-Track:2705c06e0eae416591bc1f27efc84cd6    level:DEBUG logger:o.h.e.transaction.spi.AbstractTransactionImpl    message:committing
    date:2014-08-26 12:57:48    thread:tomcat-http--3   X-Track:2705c06e0eae416591bc1f27efc84cd6    level:DEBUG logger:o.h.e.transaction.internal.jdbc.JdbcTransaction  message:committed JDBC Connection
    date:2014-08-26 12:57:48    thread:tomcat-http--3   X-Track:2705c06e0eae416591bc1f27efc84cd6    level:TRACE logger:o.t.gfw.web.logging.TraceLoggingInterceptor      message:[END CONTROLLER  ] TodoController.list(Model)-> view=todo/list, model={todoForm=todo.app.todo.TodoForm@11d3160, todos=[], org.springframework.validation.BindingResult.todoForm=org.springframework.validation.BeanPropertyBindingResult: 0 errors}
    date:2014-08-26 12:57:48    thread:tomcat-http--3   X-Track:2705c06e0eae416591bc1f27efc84cd6    level:TRACE logger:o.t.gfw.web.logging.TraceLoggingInterceptor      message:[HANDLING TIME   ] TodoController.list(Model)-> 239,725,570 ns

|

.. _using_terasolunaDao:

TERASOLUNA DAO(MyBatis2)を使用したインフラストラクチャ層の作成
--------------------------------------------------------------------------------
本節では、インフラストラクチャ層において、TERASOLUNA DAO(MyBatis2)を使用する場合の設定方法について説明する。

 .. note::
    TERASOLUNA DAOとは、MyBatis2.3.5とSpringの連携クラスであるorg.springframework.orm.ibatis.support.SqlMapClientDaoSupportを、用途別に拡張した簡易SQLマッパーを提供するライブラリである。
    以下4つのインタフェースをもつDAOが、提供されている。

       #. jp.terasoluna.fw.dao.QueryDAO
       #. jp.terasoluna.fw.dao.UpdateDAO
       #. jp.terasoluna.fw.dao.StoredProcedureDAO
       #. jp.terasoluna.fw.dao.QueryRowHandleDAO

    それぞれのインタフェースに対して、\ ``jp.terasoluna.fw.dao.ibatis.XxxDAOiBatisImpl``\ という実装を持つ。

RepositoryImplの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

TERASOLUNA DAO(MyBatis2)を使用した場合、RepositoryImplは、TERASOLUNA DAOから提供されているクラスを呼び出すことで、
データベースにアクセスを行うように実装する。

 .. code-block:: java
    :emphasize-lines: 5,7,8,11,15,16,19-21,23-24,28,30,34,36,41-47,50,52,63,65

    package todo.domain.repository.todo;

    import java.util.Collection;

    import javax.inject.Inject;

    import jp.terasoluna.fw.dao.QueryDAO;
    import jp.terasoluna.fw.dao.UpdateDAO;

    import org.springframework.stereotype.Repository;
    import org.springframework.transaction.annotation.Transactional;

    import todo.domain.model.Todo;

    @Repository // (1)
    @Transactional // (2)
    public class TodoRepositoryImpl implements TodoRepository {

        // (3)
        @Inject
        QueryDAO queryDAO;

        @Inject
        UpdateDAO updateDAO;

        // (4)
        @Override
        @Transactional(readOnly = true)
        public Todo findOne(String todoId) {
            return queryDAO.executeForObject("todo.findOne", todoId, Todo.class);
        }

        @Override
        @Transactional(readOnly = true)
        public Collection<Todo> findAll() {
            return queryDAO.executeForObjectList("todo.findAll", null);
        }

        @Override
        public Todo save(Todo todo) {
            // (5)
            if (exists(todo.getTodoId())) {
                updateDAO.execute("todo.update", todo);
            } else {
                updateDAO.execute("todo.create", todo);
            }
            return todo;
        }

        @Transactional(readOnly = true)
        public boolean exists(String todoId) {
            long count = queryDAO.executeForObject("todo.exists", todoId,
                    Long.class);
            return count > 0;
        }

        @Override
        public void delete(Todo todo) {
            updateDAO.execute("todo.delete", todo);
        }

        @Override
        @Transactional(readOnly = true)
        public long countByFinished(boolean finished) {
            return queryDAO.executeForObject("todo.countByFinished", finished,
                    Long.class);
        }
    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 80


   * - 項番
     - 説明
   * - | (1)
     - | Repositoryとして、component-scan対象とするため、クラスレベルに@Repositoryアノテーションをつける。
   * - | (2)
     - | クラスレベルに、\ ``@Transactional``\ アノテーションをつけることで、公開メソッドをすべてトランザクション管理する。
       | Repositoryを呼び出すService側でも設定しているため、\ ``@Transactional``\ をつけなくともトランザクション管理になるが、propagation属性は、デフォルトのREQUIREDであるため、トランザクションがネストした場合、内側(Repository側)のトランザクションは、外側(Service側)のトランザクションに参加する。
   * - | (3)
     - | \ ``@Inject``\ アノテーションで、QueryDAO, UpdateDAOをインジェクションする。
   * - | (4)
     - | Repositoryのメソッド実装は、基本的には、TERASOLUNA DAOにSQLIDと、パラメータを渡すことになる。
       | 参照系の場合はQueryDAO、更新系の場合はUpdateDAOを使用する。SQLIDに対応するSQLの設定は、次に行う。
   * - | (5)
     - | saveメソッドで新規作成と、更新の両方を実装している。
       | どちらの処理を行うか判断するために、existsメソッドを作成する。
       | このメソッドでは、対象のtodoIdの件数を取得し、件数が0より大きいかどうかで存在を確認する。

 .. note::

    saveメソッドは、新規作成でも更新でも利用できるメリットがある。
    しかしながら、2回SQLが実行されるという性能面でのデメリットもある。
    性能を重視する場合は、新規作成用にcreateメソッドを、更新用にupdateメソッドを作成すること。


SQLMapファイルの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
\ :file:`src/main/resources/META-INF/mybatis/sql/todo-sqlmap.xml`\を作成し、
\ ``TodoRepositoryImpl``\で使用したSQLIDに対応するsqlを、以下のように記述する。

 .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE sqlMap
                PUBLIC "-//ibatis.apache.org//DTD SQL Map 2.0//EN"
                "http://ibatis.apache.org/dtd/sql-map-2.dtd">
    <sqlMap namespace="todo">
        <resultMap id="todo" class="todo.domain.model.Todo">
            <result property="todoId" column="todo_id" />
            <result property="todoTitle" column="todo_title" />
            <result property="finished" column="finished" />
            <result property="createdAt" column="created_at" />
        </resultMap>

        <select id="findOne" parameterClass="java.lang.String"
            resultMap="todo">
        <![CDATA[
            SELECT
                todo_id,
                todo_title,
                finished,
                created_at
            FROM
                todo
            WHERE
                todo_id = #value#
        ]]>
        </select>

        <select id="findAll" resultMap="todo">
        <![CDATA[
            SELECT
                todo_id,
                todo_title,
                finished,
                created_at
            FROM
                todo
        ]]>
        </select>

        <insert id="create" parameterClass="todo.domain.model.Todo">
        <![CDATA[
            INSERT INTO todo
            (
                todo_id,
                todo_title,
                finished,
                created_at
            )
            VALUES
            (
                #todoId#,
                #todoTitle#,
                #finished#,
                #createdAt#
            )
        ]]>
        </insert>

        <update id="update" parameterClass="todo.domain.model.Todo">
        <![CDATA[
            UPDATE todo
            SET
                todo_title = #todoTitle#,
                finished = #finished#,
                created_at = #createdAt#
            WHERE
                todo_id = #todoId#
        ]]>
        </update>

        <delete id="delete" parameterClass="todo.domain.model.Todo">
        <![CDATA[
            DELETE FROM
                todo
            WHERE
                todo_id = #todoId#
        ]]>
        </delete>

        <select id="countByFinished" parameterClass="java.lang.Boolean"
            resultClass="java.lang.Long">
        <![CDATA[
            SELECT
                COUNT(*)
            FROM
                todo
            WHERE
                finished = #value#
        ]]>
        </select>

        <select id="exists" parameterClass="java.lang.String"
            resultClass="java.lang.Long">
        <![CDATA[
            SELECT
                COUNT(*)
            FROM
                todo
            WHERE
                todo_id = #value#
        ]]>
        </select>
    </sqlMap>

|

以上で、TERASOLUNA DAOを使用したインフラストラクチャ層の作成が完了したので、Service及びアプリケーション層の作成を行う。

Service及びアプリケーション層を作成後にAPサーバーを起動し、Todoの表示を行うと、以下のようなSQLログや、トランザクションログが出力される。

 .. code-block:: text
   :emphasize-lines: 2-8

    date:2014-08-26 13:39:08    thread:tomcat-http--9   X-Track:49fbb5426f574375b8169d39c330ec7f    level:TRACE logger:o.t.gfw.web.logging.TraceLoggingInterceptor      message:[START CONTROLLER] TodoController.list(Model)
    date:2014-08-26 13:39:08    thread:tomcat-http--9   X-Track:49fbb5426f574375b8169d39c330ec7f    level:DEBUG logger:o.s.jdbc.datasource.DataSourceTransactionManager message:Creating new transaction with name [todo.domain.repository.todo.TodoRepositoryImpl.findAll]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT,readOnly; ''
    date:2014-08-26 13:39:08    thread:tomcat-http--9   X-Track:49fbb5426f574375b8169d39c330ec7f    level:DEBUG logger:o.s.jdbc.datasource.DataSourceTransactionManager message:Acquired Connection [net.sf.log4jdbc.ConnectionSpy@1d7541c] for JDBC transaction
    date:2014-08-26 13:39:08    thread:tomcat-http--9   X-Track:49fbb5426f574375b8169d39c330ec7f    level:DEBUG logger:jdbc.sqltiming                                   message: com.ibatis.sqlmap.engine.execution.DefaultSqlExecutor.executeQuery(DefaultSqlExecutor.java:183)
    4. SELECT             todo_id,             todo_title,             finished,             created_at         FROM             todo {executed in 0 msec}
    date:2014-08-26 13:39:08    thread:tomcat-http--9   X-Track:49fbb5426f574375b8169d39c330ec7f    level:DEBUG logger:o.s.jdbc.datasource.DataSourceTransactionManager message:Initiating transaction commit
    date:2014-08-26 13:39:08    thread:tomcat-http--9   X-Track:49fbb5426f574375b8169d39c330ec7f    level:DEBUG logger:o.s.jdbc.datasource.DataSourceTransactionManager message:Committing JDBC transaction on Connection [net.sf.log4jdbc.ConnectionSpy@1d7541c]
    date:2014-08-26 13:39:08    thread:tomcat-http--9   X-Track:49fbb5426f574375b8169d39c330ec7f    level:DEBUG logger:o.s.jdbc.datasource.DataSourceTransactionManager message:Releasing JDBC Connection [net.sf.log4jdbc.ConnectionSpy@1d7541c] after transaction
    date:2014-08-26 13:39:08    thread:tomcat-http--9   X-Track:49fbb5426f574375b8169d39c330ec7f    level:TRACE logger:o.t.gfw.web.logging.TraceLoggingInterceptor      message:[END CONTROLLER  ] TodoController.list(Model)-> view=todo/list, model={todoForm=todo.app.todo.TodoForm@c7582d, todos=[], org.springframework.validation.BindingResult.todoForm=org.springframework.validation.BeanPropertyBindingResult: 0 errors}
    date:2014-08-26 13:39:08    thread:tomcat-http--9   X-Track:49fbb5426f574375b8169d39c330ec7f    level:TRACE logger:o.t.gfw.web.logging.TraceLoggingInterceptor      message:[HANDLING TIME   ] TodoController.list(Model)-> 2,521,308 ns

|

おわりに
================================================================================
このチュートリアルでは、以下の内容を学習した。

* TERASOLUNA Global Frameworkによる基本的なアプリケーションの開発方法、およびEclipseプロジェクトの構築方法

 * STSの使用方法
 * MavenでTERASOLUNA Global Frameworkを使用する方法

* TERASOLUNA Global Frameworkのアプリケーションのレイヤ化に従った開発方法

 * POJO(+ Spring)によるドメイン層の実装
 * Spring MVCとJSPタグライブラリを使用したアプリケーション層の実装
 * Spring Data JPAによるインフラストラクチャ層の実装
 * MyBatis2によるインフラストラクチャ層の実装

ここで作成したTODO管理アプリケーションには、以下の改善点がある。
アプリケーションの修正を学習課題として、ガイドライン中の該当する説明を参照されたい。

* プロパティを外部化する → :doc:`../ArchitectureInDetail/PropertyManagement`
* メッセージを外部化する → :doc:`../ArchitectureInDetail/MessageManagement`
* ページング処理を追加する → :doc:`../ArchitectureInDetail/Pagination`
* 例外ハンドリングを加える → :doc:`../ArchitectureInDetail/ExceptionHandling`
* CSRF対策を追加する → :doc:`../Security/CSRF`


.. raw:: latex

   \newpage

