チュートリアル(Todoアプリケーション REST編)
********************************************************************************

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

はじめに
================================================================================

このチュートリアルで学ぶこと
--------------------------------------------------------------------------------

* TERASOLUNA Global Frameworkによる基本的なRESTful Webサービスの構築方法

対象読者
--------------------------------------------------------------------------------

* ﻿\ :doc:`../TutorialTodo/index`\を実施している。


検証環境
--------------------------------------------------------------------------------

このチュートリアルは以下の環境で動作確認している。他の環境で実施する際は本書をベースに適宜読み替えて設定していくこと。

ただし、REST Clientとして、Google Chromeの拡張機能である「\ `Dev HTTP Client <https://chrome.google.com/webstore/detail/dev-http-client/aejoelaoggembcahagimdiliamlcdmfm>`_\ 」を使用するため、
Web BrowserはGoogle Chromeを使用する必要がある。

 .. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 25 75

    * - 種別
      - 名前
    * - OS
      - Windows7 64bit
    * - JVM
      - Java 1.7
    * - IDE
      - Spring Tool Suite Version: 3.4.0.RELEASE, Build Id: 201310051614 (以降「STS」と呼ぶ) 
    * - Build
      - Maven 3.0.4
    * - Application Server
      - VMWare vFabric tc Server Developer Edition v2.9 (以降「ts Server」と呼ぶ)
    * - Web Browser
      - Google Chrome 33.0.1750.154 m, Language is English(United States)
    * - REST Client
      - Dev HTTP Client 0.6.9.16


環境構築
================================================================================

Java, STS, Maven, Google Chromeについては、﻿\ :doc:`../TutorialTodo/index`\を実施する事でインストール済みの状態である事を前提とする。

Dev HTTP Clientのインストール
--------------------------------------------------------------------------------
まず、RESTクライアントしてChromeの拡張機能である「Dev HTTP Client」をインストールする。

Chromeの「Tools」→「Extensions」を選択する。

 .. figure:: ./images_rest/install-dev-http-client1.png
   :width: 80%

|

「Get more extensions」のリンクを押下する。

 .. figure:: ./images_rest/install-dev-http-client2.png

|

検索フォームに「dev http clinet」を入力して検索する。

 .. figure:: ./images_rest/install-dev-http-client3.png

|

Dev HTTP Clientの「+ FREE」ボタンを押下する。

 .. figure:: ./images_rest/install-dev-http-client4.png
   :width: 80%

|

「Add」ボタンを押下する。

 .. figure:: ./images_rest/install-dev-http-client5.png

|

Chromeのアプリケーション一覧を開く(ブラウザのアドレスバーに「chrome://apps/」を指定して開く)と、DHC(Dev HTTP Client)が追加されている。
   
 .. figure:: ./images_rest/install-dev-http-client6.png

|

| DHC(Dev HTTP Client)をクリックする。
| 以下の画面が表示されれば、インストール完了となる。
| この画面は、ブラウザのアドレスバーに「chrome-extension://aejoelaoggembcahagimdiliamlcdmfm/HttpClient.html」を入力する事で開く事もできる。
   
 .. figure:: ./images_rest/install-dev-http-client7.png
   :width: 80%

|


プロジェクト作成
--------------------------------------------------------------------------------

| チュートリアル用のプロジェクトを作成する。
| ドメイン層は\ :doc:`../TutorialTodo/index`\で作成したTodoアプリケーションのものを利用するが、今回はプロジェクトの雛形をBlankプロジェクトから生成する。

以下のコマンドを実行し、MyBatis2版のMavenプロジェクトを作成する。

 .. code-block:: console

    mvn org.apache.maven.plugins:maven-archetype-plugin:2.4:generate -B^
     -DarchetypeCatalog=http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases^
     -DarchetypeGroupId=org.terasoluna.gfw.blank^
     -DarchetypeArtifactId=terasoluna-gfw-web-blank-mybatis2-archetype^
     -DarchetypeVersion=1.0.6.RELEASE^
     -DgroupId=todo^
     -DartifactId=todo-api^
     -Dversion=1.0-SNAPSHOT

コンソール上に以下のようなログが表示されれば、ブランクプロジェクトの作成は成功となる。

 .. code-block:: console
    
    C:\workspace>mvn org.apache.maven.plugins:maven-archetype-plugin:2.4:generate -B^
    More?  -DarchetypeCatalog=http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases^
    More?  -DarchetypeGroupId=org.terasoluna.gfw.blank^
    More?  -DarchetypeArtifactId=terasoluna-gfw-web-blank-mybatis2-archetype^
    More?  -DarchetypeVersion=1.0.6.RELEASE^
    More?  -DgroupId=todo^
    More?  -DartifactId=todo-api^
    More?  -Dversion=1.0-SNAPSHOT
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
    [INFO] Archetype repository not defined. Using the one from [org.terasoluna.gfw.blank:terasoluna-gfw-web-blank-mybatis2-archetype:1.0.0.RELEASE -> http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases] found in catalog http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases
    [INFO] ----------------------------------------------------------------------------
    [INFO] Using following parameters for creating project from Archetype: terasoluna-gfw-web-blank-mybatis2-archetype:1.0.6.RELEASE
    [INFO] ----------------------------------------------------------------------------
    [INFO] Parameter: groupId, Value: todo
    [INFO] Parameter: artifactId, Value: todo-api
    [INFO] Parameter: version, Value: 1.0-SNAPSHOT
    [INFO] Parameter: package, Value: todo
    [INFO] Parameter: packageInPathFormat, Value: todo
    [INFO] Parameter: package, Value: todo
    [INFO] Parameter: version, Value: 1.0-SNAPSHOT
    [INFO] Parameter: groupId, Value: todo
    [INFO] Parameter: artifactId, Value: todo-api
    [INFO] project created from Archetype in dir: C:\workspace\todo-api
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 2.106 s
    [INFO] Finished at: 2017-02-24T10:28:17+09:00
    [INFO] Final Memory: 12M/197M
    [INFO] ------------------------------------------------------------------------
    C:\workspace>


プロジェクトのインポート
--------------------------------------------------------------------------------
作成したMavenプロジェクトをSTSのworkspaceにインポートする。

STSで「File」→「Import」→「Maven」→「Existing Maven Projects」を選択して「Next >」をクリックする。

先ほど作成した「todo-api」フォルダを選択し、「Finish」をクリックする。

 .. figure:: ./images_rest/import-maven-project.png
   :width: 90%

 .. tip::

    「Pacakge Presentaion」は、以下のように「Hierarchical」に変更することを推奨する。

    「Package Explorer」→「View Menu」→「Pacakge Presentaion」を選択して「Hierarchical」をクリックする。

     .. figure:: ./images_rest/package-presentation-hierarchical.png
        :width: 70%

ドメイン層のファイルをコピー
--------------------------------------------------------------------------------
﻿\ :doc:`../TutorialTodo/index`\で作成したクラスのうち、

* \ ``todo.domain.model.Todo``\ 
* \ ``todo.domain.repository.todo.TodoRepository``\ 
* \ ``todo.domain.repository.todo.TodoRepositoryImpl``\ (MyBatis2版)
* \ ``todo.domain.serivce.todo.TodoService``\ 
* \ ``todo.domain.serivce.todo.TodoServiceImpl``\ (インフラストラクチャ層の変更反映版)

と設定ファイル

* \ :file:`src/main/resources/META-INF/mybatis/sql/todo-sqlmap.xml`\

を、STSにインポートしたプロジェクトにコピーする。

コピー後のSTSのworkspaceは以下のようになる。

 .. figure:: ./images_rest/after-cp-domain-layer.png

テーブル定義の追加
--------------------------------------------------------------------------------
﻿﻿todoテーブルを生成するためのDDLを追加する。

\ :doc:`../TutorialTodo/index`\で作成した、\ :file:`src/main/resources/META-INF/spring/todo-infra.properties`\の\ ``database.url``\ に設定されているテーブル定義の設定内容を、
今回作成した\ :file:`src/main/resources/META-INF/spring/todo-api-infra.properties`\の\ ``database.url``\ にコピーする。

* :file:`src/main/resources/META-INF/spring/todo-api-infra.properties`\

 INITパラメータにテーブルを作成するDDLを指定する事で、アプリケーションサーバ起動時にH2のインメモリデータベースにtodoテーブルが作成される。

 .. code-block:: properties
    :emphasize-lines: 2

    database=H2
    database.url=jdbc:h2:mem:todo-api;DB_CLOSE_DELAY=-1;INIT=create table if not exists todo(todo_id varchar(36) primary key, todo_title varchar(30), finished boolean, created_at timestamp)
    database.username=sa
    database.password=
    database.driverClassName=org.h2.Driver
    # connection pool
    cp.maxActive=96
    cp.maxIdle=16
    cp.minIdle=0
    cp.maxWait=60000


REST APIの作成
================================================================================

本チュートリアルでは、todoテーブルで管理しているデータ(以降、「Todoリソース」呼ぶ)をWeb上に公開するためのREST APIを作成する。

 .. tabularcolumns:: |p{0.20\linewidth}|p{0.10\linewidth}|p{0.30\linewidth}|p{0.15\linewidth}|p{0.25\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 20 10 30 15 25

    * - | API名
      - | HTTP
        | メソッド
      - | パス
      - | ステータス
        | コード
      - | 説明
    * - | GET Todos
      - | GET
      - | \ ``/api/v1/todos``\ 
      - | 200
        | (OK)
      - | Todoリソースを全件取得する。
    * - | POST Todos
      - | POST
      - | \ ``/api/v1/todos``\ 
      - | 201
        | (Created)
      - | Todoリソースを新規作成する。
    * - | GET Todo
      - | GET
      - | \ ``/api/v1/todos/{todoId}``\ 
      - | 200
        | (OK)
      - | Todoリソースを一件取得する。
    * - | PUT Todo
      - | PUT
      - | \ ``/api/v1/todos/{todoId}``\ 
      - | 200
        | (OK)
      - | Todoリソースを完了状態に更新する。
    * - | DELETE Todo
      - | DELETE
      - | \ ``/api/v1/todos/{todoId}``\ 
      - | 204
        | (No Content)
      - | Todoリソースを削除する。

 .. tip::

    パス内に含まれている\ ``{todoId}``\は、パス変数と呼ばれ、任意の可変値を扱う事ができる。
    パス変数を使用する事で、\ ``GET /api/v1/todos/123``\と\ ``GET /api/v1/todos/456``\を同じAPIで扱う事ができる。
   
    本チュートリアルでは、Todoを一意に識別するためのID(Todo ID)をパス変数として扱っている。



API仕様
--------------------------------------------------------------------------------

| HTTPリクエストとレスポンスの具体例を用いて、本チュートリアルで作成するREST APIのインタフェース仕様を示す。
| 本質的ではないHTTPヘッダー等は例から除いている。

| 下記に示す具体例では、Webアプリケーションのコンテキストパスは「<contextPath>」で表現している。
| 本チュートリアルで作成するWebアプリケーションのコンテキストパスは、「todo-api」となる。

GET Todos
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* リクエスト

  .. code-block:: bash
  
      > GET /<contextPath>/api/v1/todos HTTP/1.1

* レスポンス

  作成済みのTodoリソースのリストをJSON形式で返却する。

  .. code-block:: bash

      < HTTP/1.1 200 OK
      < Content-Type: application/json;charset=UTF-8
      <
      [{"todoId":"9aef3ee3-30d4-4a7c-be4a-bc184ca1d558","todoTitle":"Hello World!","finished":false,"createdAt":"2014-02-25T02:21:48.493+0000"}]

POST Todos
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* リクエスト

  新規作成するTodoリソースの内容(タイトル)をJSON形式で指定する。

  .. code-block:: bash

      > POST /<contextPath>/api/v1/todos HTTP/1.1
      > Content-Type: application/json
      > Content-Length: 29
      >
      {"todoTitle": "Study Spring"}
      

* レスポンス

  作成したTodoリソースをJSON形式で返却する。

  .. code-block:: bash

      < HTTP/1.1 201 Created
      < Content-Type: application/json;charset=UTF-8
      <
      {"todoId":"d6101d61-b22c-48ee-9110-e106af6a1404","todoTitle":"Study Spring","finished":false,"createdAt":"2014-02-25T04:05:58.752+0000"}

GET Todo
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* リクエスト

  | パス変数「\ ``todoId``\」に、取得対象のTodoリソースのIDを指定する。
  | 下記例では、パス変数「\ ``todoId``\」に\ ``9aef3ee3-30d4-4a7c-be4a-bc184ca1d558``\を指定している。

  .. code-block:: bash
  
      > GET /<contextPath>/api/v1/todos/9aef3ee3-30d4-4a7c-be4a-bc184ca1d558 HTTP/1.1


* レスポンス

  パス変数「\ ``todoId``\」に一致するTodoリソースをJSON形式で返却する。

  .. code-block:: bash

      < HTTP/1.1 200 OK
      < Content-Type: application/json;charset=UTF-8
      <
      {"todoId":"9aef3ee3-30d4-4a7c-be4a-bc184ca1d558","todoTitle":"Hello World!","finished":false,"createdAt":"2014-02-25T02:21:48.493+0000"}


PUT Todo
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* リクエスト

  | パス変数「\ ``todoId``\」に、更新対象のTodoのIDを指定する。
  | PUT Todoでは、Todoリソースを完了状態に更新するだけなので、リクエストBODYを受け取らないインタフェース仕様にしている。

  .. code-block:: bash

      > PUT /<contextPath>/api/v1/todos/9aef3ee3-30d4-4a7c-be4a-bc184ca1d558 HTTP/1.1


* レスポンス

  パス変数「\ ``todoId``\」に一致するTodoリソースを完了状態(\ ``finished``\ フィールドを\ ``true``\ )に更新し、JSON形式で返却する。

  .. code-block:: bash

      < HTTP/1.1 200 OK
      < Content-Type: application/json;charset=UTF-8
      <
      {"todoId":"9aef3ee3-30d4-4a7c-be4a-bc184ca1d558","todoTitle":"Hello World!","finished":true,"createdAt":"2014-02-25T02:21:48.493+0000"}


DELETE Todo
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* リクエスト

  パス変数「\ ``todoId``\」に、削除対象のTodoリソースのIDを指定する。

  .. code-block:: bash

      > DELETE /todo-api/api/v1/todos/9aef3ee3-30d4-4a7c-be4a-bc184ca1d558 HTTP/1.1

* レスポンス

  DELETE Todoでは、Todoリソースの削除が完了した事で返却するリソースが存在しなくなった事を示すために、レスポンスBODYを返却しないインタフェース仕様にしている。

  .. code-block:: bash

      < HTTP/1.1 204 No Content

エラー応答
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| REST APIでエラーが発生した場合は、JSON形式でエラー内容を返却する。
| 以下に代表的なエラー発生時のレスポンス仕様について記載する。
| 下記以外のエラーパターンもあるが、本チュートリアルでは説明は割愛する。

﻿\ :doc:`../TutorialTodo/index`\では、エラーメッセージはプログラムの中でハードコーディングしていたが、本チュートリアルでは、エラーメッセージはエラーコードをキーにプロパティファイルから取得する。

* 入力チェックエラー発生時のレスポンス仕様

  .. code-block:: bash

      < HTTP/1.1 400 Bad Request
      < Content-Type: application/json;charset=UTF-8
      <
      {"code":"E400","message":"[E400] The requested Todo contains invalid values.","details":[{"code":"NotNull","message":"todoTitle may not be null.",target:"todoTitle"}]}

* 業務エラー発生時のレスポンス仕様

  .. code-block:: bash

      < HTTP/1.1 409 Conflict
      < Content-Type: application/json;charset=UTF-8
      <
      {"code":"E002","message":"[E002] The requested Todo is already finished. (id=353fb5db-151a-4696-9b4a-b958358a5ab3)"}

* リソース未検出時のレスポンス仕様

  .. code-block:: bash

      < HTTP/1.1 404 Not Found
      < Content-Type: application/json;charset=UTF-8
      <
      {"code":"E404","message":"[E404] The requested Todo is not found. (id=353fb5db-151a-4696-9b4a-b958358a5ab2)"}

* システムエラー発生時のレスポンス仕様

  .. code-block:: bash

      < HTTP/1.1 500 Internal Server Error
      < Content-Type: application/json;charset=UTF-8
      <
      {"code":"E500","message":"[E500] System error occurred."}

REST API用のDispatcherServletを用意
--------------------------------------------------------------------------------

まず、REST API用のリクエストを処理するための\ ``DispatcherServlet``\の定義を追加する。

web.xmlの修正
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| REST API用の設定を追加する。
| 追加で設定する箇所をハイライトしている。

 .. code-block:: xml
    :emphasize-lines: 74-84,86-90

    <?xml version="1.0" encoding="UTF-8"?>
    <web-app xmlns="http://java.sun.com/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
        version="3.0">
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
    
        <!-- (1) -->
        <servlet>
            <servlet-name>restApiServlet</servlet-name>
            <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
            <init-param>
                <param-name>contextConfigLocation</param-name>
                <!-- ApplicationContext for Spring MVC (REST) -->
                <param-value>classpath*:META-INF/spring/spring-mvc-rest.xml</param-value>
            </init-param>
            <load-on-startup>1</load-on-startup>
        </servlet>
    
        <!-- (2) -->
        <servlet-mapping>
            <servlet-name>restApiServlet</servlet-name>
            <url-pattern>/api/v1/*</url-pattern>
        </servlet-mapping>
    
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
    
        <jsp-config>
            <jsp-property-group>
                <url-pattern>*.jsp</url-pattern>
                <el-ignored>false</el-ignored>
                <page-encoding>UTF-8</page-encoding>
                <scripting-invalid>false</scripting-invalid>
                <include-prelude>/WEB-INF/views/common/include.jsp</include-prelude>
            </jsp-property-group>
        </jsp-config>
    
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
    
        <session-config>
            <!-- 30min -->
            <session-timeout>30</session-timeout>
        </session-config>
    
    </web-app>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | 初期化パラメータ「\ ``contextConfigLocation``\」に、REST用のSpringMVC設定ファイルを指定する。
       | 本チュートリアルでは、クラスパス上にある「:file:`META-INF/spring/spring-mvc-rest.xml`」を指定している。
   * - | (2)
     - | \ ``<url-pattern>``\要素に、REST API用の\ ``DispatcherServlet``\にマッピングするURLのパターンを指定する。
       | 本チュートリアルでは、\ ``/api/v1/``\ から始まる場合はリクエストをREST APIへのリクエストとしてREST API用の\ ``DispatcherServlet``\へマッピングしている。

spring-mvc-rest.xmlの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| \ :file:`src/main/resources/META-INF/spring/spring-mvc.xml`\をコピーして、REST用のSpringMVC設定ファイルを作成する。
| REST用のSpringMVC設定ファイルは以下のような定義となる。

* :file:`src/main/resources/META-INF/spring/spring-mvc-rest.xml`

 .. code-block:: xml
    :emphasize-lines: 17, 33, 40, 70

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
    
        <context:property-placeholder
            location="classpath*:/META-INF/spring/*.properties" />

        <mvc:annotation-driven>
            <mvc:message-converters>
                <!-- (2) -->
                <bean
                    class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter">
                    <property name="objectMapper" ref="objectMapper"/>
                </bean>
            </mvc:message-converters>
            <mvc:argument-resolvers>
                <bean
                    class="org.springframework.data.web.PageableHandlerMethodArgumentResolver" />
            </mvc:argument-resolvers>
            <!-- workarround to CVE-2016-5007. -->
            <mvc:path-matching path-matcher="pathMatcher" />
        </mvc:annotation-driven>
    
        <bean id="objectMapper" class="org.codehaus.jackson.map.ObjectMapper">
            <property name="dateFormat">
                <!-- (3) -->
                <bean class="org.codehaus.jackson.map.util.StdDateFormat"/>
            </property>
        </bean>

        <mvc:default-servlet-handler />

        <!-- (1) -->
        <context:component-scan base-package="todo.api"/>

        <mvc:resources mapping="/resources/**"
            location="/resources/,classpath:META-INF/resources/"
            cache-period="#{60 * 60}" />

        <mvc:interceptors>
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
            <!-- (4) -->
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

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | REST API用のパッケージ配下に対してコンポーネントスキャンの設定をする。
       | 本チュートリアルでは、REST API用のパッケージを\ ``todo.api``\にしている。
       | 画面遷移用のContollerは、\ ``app``\ パッケージ配下に格納していたが、REST API用のContollerは、\ ``api``\ パッケージ配下に格納する事を推奨する。
   * - | (2)
     - | \ ``<mvc:message-converters>``\ に、Contrtollerの引数及び返り値で扱うJavaBeanをシリアライズ/デシリアライズするためのクラスを設定する。
       | 複数設定する事ができるが、今回のチュートリアルではJSONしか使用しないため、\ ``MappingJacksonHttpMessageConverter``\ のみ指定している。
       | \ ``MappingJacksonHttpMessageConverter``\の\ ``objectMapper``\プロパティには、Jacksonより提供されている「JSON <-> JavaBean」の変換を行うためのコンポーネント「\ ``ObjectMapper``\」を指定する。
       | 本チュートリアルでは、日時型の変換をカスタマイズした\ ``ObjectMapper``\を指定している。
   * - | (3)
     - | 本チュートリアルでは、\ ``java.util.Date``\ オブジェクトをシリアライズする際にISO-8601形式とする。
       | \ ``Date``\ オブジェクトをシリアライズする際にISO-8601形式にする場合は、\ ``ObjectMapper``\の\ ``dateFormat``\ プロパティに\ ``org.codehaus.jackson.map.util.StdDateFormat``\ を設定する事で実現する事ができる。
   * - | (4)
     - | JPAを使用する場合は\ ``OpenEntityManagerInViewInterceptor``\ を使用するが、今回のチュートリアルではMyBatis2を使用するので、コメントアウトのままにするか、この\ ``<mvc:interceptor>``\ タグを削除する。


 .. figure:: ./images_rest/add-spring-mvc-rest.png

REST API用のSpring Securityの定義追加
--------------------------------------------------------------------------------
| 本チュートリアルで作成するREST APIでは、CSRF対策を無効にする。
| REST APIを使って構築するWebアプリケーションでも、CSRF対策は必要である。ただし、本チュートリアルの目的としてCSRF対策の話題は本質的ではないため、機能を無効化し、説明も割愛する。

| CSRF対策を無効化すると、セッションを使用する必要がなくなる。
| そのため、本チュートリアルではセッションを使用しないアーキテクチャ（ステートレスなアーキテクチャ）を採用する。

| 以下の設定を追加する事で、CSRF対策の無効化及びセッションを使用しないようにする事ができる。

* :file:`src/main/resources/META-INF/spring/spring-security.xml`

 .. code-block:: xml
    :emphasize-lines: 11-16

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:sec="http://www.springframework.org/schema/security"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    
        <sec:http pattern="/resources/**" security="none"/>
        
        <!-- (1) -->
        <sec:http pattern="/api/v1/**" auto-config="true" use-expressions="true" create-session="stateless">
            <!--<sec:custom-filter ref="csrfFilter" before="LOGOUT_FILTER"/>--> <!-- (2) -->
            <sec:custom-filter ref="userIdMDCPutFilter" after="ANONYMOUS_FILTER"/>
            <!--<sec:session-management session-authentication-strategy-ref="sessionAuthenticationStrategy" />--> <!-- (3) -->
        </sec:http>

        <sec:http auto-config="true" use-expressions="true">
            <sec:custom-filter ref="csrfFilter" before="LOGOUT_FILTER"/>
            <sec:custom-filter ref="userIdMDCPutFilter" after="ANONYMOUS_FILTER"/>
            <sec:session-management session-authentication-strategy-ref="sessionAuthenticationStrategy" />
        </sec:http>
        <sec:authentication-manager></sec:authentication-manager>
    
        <!-- CSRF Protection -->
        <bean id="csrfTokenRepository"
            class="org.springframework.security.web.csrf.HttpSessionCsrfTokenRepository" />
    
        <bean id="csrfFilter" class="org.springframework.security.web.csrf.CsrfFilter">
            <constructor-arg index="0" ref="csrfTokenRepository" />
            <property name="accessDeniedHandler">
                <bean
                    class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
                    <property name="errorPage" value="/WEB-INF/views/common/error/csrfTokenError.jsp" />
                </bean>
            </property>
        </bean>
    
        <bean id="sessionAuthenticationStrategy"
            class="org.springframework.security.web.authentication.session.CompositeSessionAuthenticationStrategy">
            <constructor-arg index="0">
                <list>
                    <bean
                        class="org.springframework.security.web.authentication.session.SessionFixationProtectionStrategy" />
                    <bean
                        class="org.springframework.security.web.csrf.CsrfAuthenticationStrategy">
                        <constructor-arg index="0"
                            ref="csrfTokenRepository" />
                    </bean>
                </list>
            </constructor-arg>
        </bean>
    
        <!-- Put UserID into MDC -->
        <bean id="userIdMDCPutFilter" class="org.terasoluna.gfw.security.web.logging.UserIdMDCPutFilter">
        </bean>
    
    </beans>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | REST API用のSpring Securityの定義を追加する。
       | \ ``<sec:http>``\要素のpattern属性に、REST API用のリクエストパスのURLパターンを指定している。
       | 本チュートリアルでは\ ``/api/v1/``\で始まるリクエストパスをREST API用のリクエストパスとして扱う。
       | また、create-session属性を\ ``stateless``\とする事で、Spring Securityの処理でセッションが使用されなくなる。
   * - | (2)
     - | CSRF対策を無効化するために、CSRF対策用のServlet Filterの定義をコメントアウトしている。
   * - | (3)
     - | セッションを使用しないアーキテクチャを採用するために、セッション管理に関係するコンポーネントへの参照定義をコメントアウトしている。


REST API用パッケージの作成
--------------------------------------------------------------------------------
REST API用のクラスを格納するパッケージを作成する。

| REST API用のクラスを格納するルートパッケージのパッケージ名は\ ``api``\として、配下にリソース毎のパッケージ(リソース名の小文字)を作成する事を推奨する。
| 本チュートリアルで扱うリソースのリソース名はTodoなので、\ ``todo.api.todo``\ パッケージを作成する。

 .. figure:: ./images_rest/make-package-for-rest.png

 .. note::

    作成したパッケージに格納するクラスは、通常以下の３種類となる。
    作成するクラスのクラス名は、以下のネーミングルールとする事を推奨する。

    * \ ``[リソース名]Resource``\ 
    * \ ``[リソース名]RestController``\ 
    * \ ``[リソース名]Helper``\  (必要に応じて)

    本チュートリアルで扱うリソースのリソース名がTodoなので、

    * \ ``TodoResource``\ 
    * \ ``TodoRestController``\ 

    を作成する。
    
    本チュートリアルでは、\ ``TodoRestHelper``\は作成しない。


Resourceクラスの作成
--------------------------------------------------------------------------------
| Todoリソースを表現する\ ``TodoResource``\ クラスを作成する。
| 本ガイドラインでは、REST APIの入出力となるJSON(またはXML)を表現するJava Beanを\ **Resourceクラス**\ と呼ぶ。

* :file:`TodoResource.java`

 .. code-block:: java

    package todo.api.todo;
    
    import java.util.Date;
    
    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;
    
    public class TodoResource {

        private String todoId;
    
        @NotNull
        @Size(min = 1, max = 30)
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


 .. figure:: ./images_rest/create-todo-resource.png

 .. note::

  DomainObjectクラス(本チュートリアルでは\ ``Todo``\ クラス)があるにも関わらず、Resourceクラスを作成する理由は、
  クライアントとの入出力で使用するインタフェース上の情報と、業務処理で扱う情報は必ずしも一致しないためである。
  
  これらを混同してして使用すると、アプリケーション層の影響がドメイン層におよび、保守性を低下させる。
  DomainObjectとResourceクラスは別々に作成し、Dozer等のBeanMapperを利用してデータ変換を行うことを推奨する。
  
  ResourceクラスはFormクラスと役割が似ているが、FormクラスはHTMLの\ ``<form>`` \ タグをJavaBeanで表現したもの、
  ResourceクラスはREST APIの入出力をJavaBeanで表現したものであり、本質的には異なるものである。
  
  ただし、実体としてはBean Validationのアノテーションを付与したJavaBeanであり、Controllerクラスと同じパッケージに格納することから、
  Formクラスとほぼ同じである。


Controllerクラスの作成
--------------------------------------------------------------------------------
\ ``TodoResource``\のREST APIを提供する\ ``TodoRestController``\ クラスを作成する。

* :file:`TodoRestController.java`

 .. code-block:: java

    package todo.api.todo;
    
    import org.springframework.stereotype.Controller;
    import org.springframework.web.bind.annotation.RequestMapping;
    
    @Controller
    @RequestMapping("todos") // (1)
    public class TodoRestController {
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | リソースのパスを指定する。
       | \ ``/api/v1/``\ の部分はweb.xmlに定義しているため、この設定を行うことで\ ``/<contextPath>/api/v1/todos``\ というパスにマッピングされる。

GET Todosの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
作成済みのTodoリソースを全件取得するAPI(GET Todos)の処理を、\ ``TodoRestController``\の\ ``getTodos``\メソッドに実装する。

 .. code-block:: java
    :emphasize-lines: 3-10,13-18,23-38

    package todo.api.todo;
    
    import java.util.ArrayList;
    import java.util.Collection;
    import java.util.List;
    
    import javax.inject.Inject;
    
    import org.dozer.Mapper;
    import org.springframework.http.HttpStatus;
    import org.springframework.stereotype.Controller;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.bind.annotation.ResponseBody;
    import org.springframework.web.bind.annotation.ResponseStatus;
    
    import todo.domain.model.Todo;
    import todo.domain.service.todo.TodoService;
    
    @Controller
    @RequestMapping("todos")
    public class TodoRestController {
        @Inject
        TodoService todoService;
        @Inject
        Mapper beanMapper;
    
        @RequestMapping(method = RequestMethod.GET) // (1)
        @ResponseBody // (2)
        @ResponseStatus(HttpStatus.OK) // (3)
        public List<TodoResource> getTodos() {
            Collection<Todo> todos = todoService.findAll();
            List<TodoResource> todoResources = new ArrayList<>();
            for (Todo todo : todos) {
                todoResources.add(beanMapper.map(todo, TodoResource.class)); // (4)
            }
            return todoResources; // (5)
        }
    
    }



 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | メソッドがGETのリクエストを処理するために、\ ``method``\ 属性に\ ``RequestMethod.GET``\ を設定する。
   * - | (2)
     - | メソッドの返り値として返却したオブジェクトを、レスポンスBODYのメッセージとしてクライアントに返答することを示す\ ``@ResponseBody``\ アノテーションを付与する。
       | 本チュートリアルでは、JSON形式のデータにシリアライズする。
   * - | (3)
     - | 応答するHTTPステータスコードを\ ``@ResponseStatus``\ アノテーションに指定する。
       | HTTPステータスとして、"200 OK"を設定するため、value属性には\ ``HttpStatus.OK``\を設定する。
   * - | (4)
     - | \ ``TodoService.findAll``\ で取得した\ ``Todo``\ オブジェクトを、応答するJSONを表現する\ ``TodoResource``\ 型に変換する。
       | \ ``Todo``\と\ ``TodoResource``\ の変換処理は、Dozerの\ ``org.dozer.Mapper``\ を使うと便利である。
   * - | (5)
     - | \ ``List<TodoResource>``\ オブジェクトを返却することで、\ :file:`spring-mvc-rest.xml`\に定義した\ ``MappingJacksonHttpMessageConverter``\ により、\ ``[{"todoId" : "xxx", "todoTitle": "yyy", ...}, {"todoId" : "xxx", "todoTitle": "yyy", ...}, ...]``\ 形式のJSONにシリアライズされる。

| tc Serverを起動し、実装したAPIの動作確認を行う。
| プロジェクト名を右クリックして、「Run As」→「Run on Server」を選択する。

 .. figure:: ./images_rest/run-on-server1.png
   :width: 100%

「VMWare vFabric tc Server Developer Edition v2.9」を選択して、「Next >」をクリックする。

 .. figure:: ./images_rest/run-on-server2.png
   :width: 100%

「Configured」の欄に「todo-api」が含まれていることを確認して、「Finish」をクリックする。

 .. figure:: ./images_rest/run-on-server3.png
   :width: 100%
   
正常に起動して\ ``"localhost:8080/todo-api/"``\にアクセスすると、以下のようにデフォルトのwelcomeページが表示される。

 .. figure:: ./images_rest/run-on-server4.png
   :width: 100%

次に、REST APIにアクセスする。
Dev HTTP Clientを開いてURLに\ ``"localhost:8080/todo-api/api/v1/todos"``\を入力し、メソッドにGETを指定して、"Send"ボタンをクリックする。

 .. figure:: ./images_rest/get-todos1.png
   :width: 100%

| 以下のように「RESPONSE」の「BODY」に実行結果のJSONが表示される。
| 現時点ではデータが何も登録されていないため、空配列である\ ``[]``\ が返却される。

 .. figure:: ./images_rest/get-todos2.png
   :width: 100%
   
 Spring Securityの設定を、セッションを使用しない設定に変更しているため、「RESPONSE」の「HEADERS」に\ ``"Set-Cookie: JSESSIONID=xxxx"``\がないという点にも着目してほしい。
  

POST Todosの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Todoリソースを新規作成するAPI(POST Todos)の処理を、\ ``TodoRestController``\ の\ ``postTodos``\メソッドに実装する。


 .. code-block:: java
    :emphasize-lines: 12-13,42-49

    package todo.api.todo;

    import java.util.ArrayList;
    import java.util.Collection;
    import java.util.List;

    import javax.inject.Inject;

    import org.dozer.Mapper;
    import org.springframework.http.HttpStatus;
    import org.springframework.stereotype.Controller;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.RequestBody;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.bind.annotation.ResponseBody;
    import org.springframework.web.bind.annotation.ResponseStatus;

    import todo.domain.model.Todo;
    import todo.domain.service.todo.TodoService;

    @Controller
    @RequestMapping("todos")
    public class TodoRestController {
        @Inject
        TodoService todoService;
        @Inject
        Mapper beanMapper;

        @RequestMapping(method = RequestMethod.GET)
        @ResponseBody
        @ResponseStatus(HttpStatus.OK)
        public List<TodoResource> getTodos() {
            Collection<Todo> todos = todoService.findAll();
            List<TodoResource> todoResources = new ArrayList<>();
            for (Todo todo : todos) {
                todoResources.add(beanMapper.map(todo, TodoResource.class));
            }
            return todoResources;
        }

        @RequestMapping(method = RequestMethod.POST) // (1)
        @ResponseBody
        @ResponseStatus(HttpStatus.CREATED) // (2)
        public TodoResource postTodos(@RequestBody @Validated TodoResource todoResource) { // (3)
            Todo createdTodo = todoService.create(beanMapper.map(todoResource, Todo.class)); // (4)
            TodoResource createdTodoResponse = beanMapper.map(createdTodo, TodoResource.class); // (5)
            return createdTodoResponse; // (6)
        }

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | メソッドがPOSTのリクエストを処理するために、\ ``method``\ 属性に\ ``RequestMethod.POST``\ を設定する。
   * - | (2)
     - | 応答するHTTPステータスコードを\ ``@ResponseStatus``\ アノテーションに指定する。
       | HTTPステータスとして、"201 Created"を設定するため、value属性には\ ``HttpStatus.CREATED``\を設定する。
   * - | (3)
     - | HTTPリクエストのBody(JSON)をJavaBeanにマッピングするために、\ ``@RequestBody``\ アノテーションをマッピング対象の\ ``TodoResource``\ クラスに付与する。
       | また、入力チェックするために\ ``@Validated``\ も付与する。例外ハンドリングは別途行う必要がある。
   * - | (4)
     - | \ ``TodoResource``\を\ ``Todo``\ クラスに変換後、\ ``TodoService.create``\ を実行し、Todoリソースを新規作成する。
   * - | (5)
     - | \ ``TodoService.create``\ によって新規作成された\ ``Todo``\オブジェクトを、応答するJSONを表現する\ ``TodoResource``\ 型に変換する。
   * - | (6)
     - | \ ``TodoResource``\ オブジェクトを返却することで、\ :file:`spring-mvc-rest.xml`\に定義した\ ``MappingJacksonHttpMessageConverter``\ により、\ ``{"todoId" : "xxx", "todoTitle": "yyy", ...}``\ 形式のJSONにシリアライズされる。

| HTTP Dev Clientを使用して、実装したAPIの動作確認を行う。
| Dev HTTP Clientを開いてURLに\ ``"localhost:8080/todo-api/api/v1/todos"``\を入力し、メソッドにPOSTを指定する。
| 「REQUEST」の「BODY」に以下のJSONを入力する。

 .. code-block:: json

    {
      "todoTitle": "Hello World!"
    }

また、「REQUEST」の「HEADERS」の「+」ボタンでHTTPヘッダーを追加し、「\ ``Content-Type``\」に「\ ``application/json``\」を設定後、"Send"ボタンをクリックする。

 .. figure:: ./images_rest/post-todos1.png
   :width: 100%


"201 Created"のHTTPステータスが返却され、「RESPONSE」の「Body」に新規作成されたTodoリソースのJSONが表示される。

 .. figure:: ./images_rest/post-todos2.png
   :width: 100%

この状態で再びGET Todosを実行すると、作成したTodoリソースを含む配列が返却される。

 .. figure:: ./images_rest/get-todos3.png
   :width: 100%

GET Todoの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
﻿\ :doc:`../TutorialTodo/index`\では、\ ``TodoService``\ に一件取得用のメソッド(\ ``findOne``\ )を作成しなかったため、
\ ``TodoService``\と\ ``TodoServiceImpl``\に以下のハイライト部を追加する。

* :file:`TodoService.java`

 \ ``findOne``\メソッドの定義を追加する。

 .. code-block:: java
    :emphasize-lines: 10

    package todo.domain.service.todo;
      
    import java.util.Collection;
      
    import todo.domain.model.Todo;
      
    public interface TodoService {
        Collection<Todo> findAll();
          
        Todo findOne(String todoId);
      
        Todo create(Todo todo);
      
        Todo finish(String todoId);
      
        void delete(String todoId);
    }

* :file:`TodoServiceImpl.java`

 \ ``findOne``\メソッド呼び出し時に開始されるトランザクションを読み取り専用に設定する。

 .. code-block:: java
    :emphasize-lines: 28
  
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
    
    @Service
    @Transactional
    public class TodoServiceImpl implements TodoService {
        @Inject
        TodoRepository todoRepository;
    
        private static final long MAX_UNFINISHED_COUNT = 5;
    
        @Override
        @Transactional(readOnly = true)
        public Todo findOne(String todoId) {
            Todo todo = todoRepository.findOne(todoId);
            if (todo == null) {
                ResultMessages messages = ResultMessages.error();
                messages.add(ResultMessage
                        .fromText("[E404] The requested Todo is not found. (id="
                                + todoId + ")"));
                throw new ResourceNotFoundException(messages);
            }
            return todo;
        }
    
        @Override
        @Transactional(readOnly = true)
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
    
                throw new BusinessException(messages);
            }
    
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

Todoリソースを一件取得するAPI(GET Todo)の処理を、\ ``TodoRestController``\ の\ ``getTodo``\メソッドに実装する。

 .. code-block:: java
    :emphasize-lines: 13,52-59

    package todo.api.todo;

    import java.util.ArrayList;
    import java.util.Collection;
    import java.util.List;

    import javax.inject.Inject;

    import org.dozer.Mapper;
    import org.springframework.http.HttpStatus;
    import org.springframework.stereotype.Controller;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.PathVariable;
    import org.springframework.web.bind.annotation.RequestBody;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.bind.annotation.ResponseBody;
    import org.springframework.web.bind.annotation.ResponseStatus;

    import todo.domain.model.Todo;
    import todo.domain.service.todo.TodoService;

    @Controller
    @RequestMapping("todos")
    public class TodoRestController {
        @Inject
        TodoService todoService;
        @Inject
        Mapper beanMapper;

        @RequestMapping(method = RequestMethod.GET)
        @ResponseBody
        @ResponseStatus(HttpStatus.OK)
        public List<TodoResource> getTodos() {
            Collection<Todo> todos = todoService.findAll();
            List<TodoResource> todoResources = new ArrayList<>();
            for (Todo todo : todos) {
                todoResources.add(beanMapper.map(todo, TodoResource.class));
            }
            return todoResources;
        }

        @RequestMapping(method = RequestMethod.POST)
        @ResponseBody
        @ResponseStatus(HttpStatus.CREATED)
        public TodoResource postTodos(@RequestBody @Validated TodoResource todoResource) {
            Todo createdTodo = todoService.create(beanMapper.map(todoResource, Todo.class));
            TodoResource createdTodoResponse = beanMapper.map(createdTodo, TodoResource.class);
            return createdTodoResponse;
        }

        @RequestMapping(value="{todoId}", method = RequestMethod.GET) // (1)
        @ResponseBody
        @ResponseStatus(HttpStatus.OK)
        public TodoResource getTodo(@PathVariable("todoId") String todoId) { // (2)
            Todo todo = todoService.findOne(todoId); // (3)
            TodoResource todoResource = beanMapper.map(todo, TodoResource.class);
            return todoResource;
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | パスから\ ``todoId``\を取得するために、\ ``@RequestMapping``\アノテーションの\ ``value``\属性にパス変数を指定する。
       | メソッドがGETのリクエストを処理するために、\ ``method``\ 属性に\ ``RequestMethod.GET``\ を設定する。
   * - | (2)
     - | \ ``@PathVariable``\アノテーションの\ ``value``\属性に、\ ``todoId``\を取得するためのパス変数名を指定する。
   * - | (3)
     - | パス変数から取得した\ ``todoId``\ を使用して、Todoリソースを一件を取得する。

| HTTP Dev Clientを使用して、実装したAPIの動作確認を行う。
| Dev HTTP Clientを開いてURLに\ ``"localhost:8080/todo-api/api/v1/todos/{todoId}``\"を入力し、メソッドにGETを指定する。
| \ ``{todoId}``\の部分は実際のIDを入れる必要があるので、POST TodosまたはGET Todosを実行してResponse中の\ ``todoId``\をコピーして貼り付けてから、"Send"ボタンをクリックする。

"200 OK"のHTTPステータスが返却され、「RESPONSE」の「Body」に指定したTodoリソースのJSONが表示される。

 .. figure:: ./images_rest/get-todo1.png
   :width: 100%


PUT Todoの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Todoリソースを一件更新(完了状態へ更新)するAPI(PUT Todo)の処理を、\ ``TodoRestController``\ の\ ``putTodo``\メソッドに実装する。

 .. code-block:: java
    :emphasize-lines: 61-68

    package todo.api.todo;
    
    import java.util.ArrayList;
    import java.util.Collection;
    import java.util.List;
    
    import javax.inject.Inject;
    
    import org.dozer.Mapper;
    import org.springframework.http.HttpStatus;
    import org.springframework.stereotype.Controller;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.PathVariable;
    import org.springframework.web.bind.annotation.RequestBody;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.bind.annotation.ResponseBody;
    import org.springframework.web.bind.annotation.ResponseStatus;
    
    import todo.domain.model.Todo;
    import todo.domain.service.todo.TodoService;
    
    @Controller
    @RequestMapping("todos")
    public class TodoRestController {
        @Inject
        TodoService todoService;
        @Inject
        Mapper beanMapper;
    
        @RequestMapping(method = RequestMethod.GET)
        @ResponseBody
        @ResponseStatus(HttpStatus.OK)
        public List<TodoResource> getTodos() {
            Collection<Todo> todos = todoService.findAll();
            List<TodoResource> todoResources = new ArrayList<>();
            for (Todo todo : todos) {
                todoResources.add(beanMapper.map(todo, TodoResource.class));
            }
            return todoResources;
        }
    
        @RequestMapping(method = RequestMethod.POST)
        @ResponseBody
        @ResponseStatus(HttpStatus.CREATED)
        public TodoResource postTodos(@RequestBody @Validated TodoResource todoResource) {
            Todo createdTodo = todoService.create(beanMapper.map(todoResource, Todo.class));
            TodoResource createdTodoResponse = beanMapper.map(createdTodo, TodoResource.class);
            return createdTodoResponse;
        }
    
        @RequestMapping(value="{todoId}", method = RequestMethod.GET)
        @ResponseBody
        @ResponseStatus(HttpStatus.OK)
        public TodoResource getTodo(@PathVariable("todoId") String todoId) {
            Todo todo = todoService.findOne(todoId);
            TodoResource todoResource = beanMapper.map(todo, TodoResource.class);
            return todoResource;
        }
    
        @RequestMapping(value="{todoId}", method = RequestMethod.PUT) // (1)
        @ResponseBody
        @ResponseStatus(HttpStatus.OK)
        public TodoResource putTodo(@PathVariable("todoId") String todoId) { // (2)
            Todo finishedTodo = todoService.finish(todoId); // (3)
            TodoResource finishedTodoResource = beanMapper.map(finishedTodo, TodoResource.class);
            return finishedTodoResource;
        }
        
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | パスから\ ``todoId``\を取得するために、\ ``@RequestMapping``\アノテーションの\ ``value``\属性にパス変数を指定する。
       | メソッドがGETのリクエストを処理するために、\ ``method``\ 属性に\ ``RequestMethod.GET``\ を設定する。
   * - | (2)
     - | \ ``@PathVariable``\アノテーションの\ ``value``\属性に、\ ``todoId``\を取得するためのパス変数名を指定する。
   * - | (3)
     - | パス変数から取得した\ ``todoId``\ を使用して、Todoリソースを完了状態へ更新する。
 
| HTTP Dev Clientを使用して、実装したAPIの動作確認を行う。
| Dev HTTP Clientを開いてURLに\ ``"localhost:8080/todo-api/api/v1/todos/{todoId}"``\を入力し、メソッドにPUTを指定する。
| \ ``{todoId}``\の部分は実際のIDを入れる必要があるので、POST TodosまたはGET Todosを実行してResponse中の\ ``todoId``\をコピーして貼り付けてから、"Send"ボタンをクリックする。

 .. figure:: ./images_rest/put-todo1.png
   :width: 100%

| "200 OK"のHTTPステータスが返却され、「RESPONSE」の「Body」に更新されたTodoリソースのJSONが表示される。
| \ ``finished``\が\ ``true``\に更新されている。

 .. figure:: ./images_rest/put-todo2.png
   :width: 100%

DELETE Todoの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  
最後に、Todoリソースを一件削除するAPI(DELETE Todo)の処理を、\ ``TodoRestController``\ の\ ``deleteTodo``\メソッドに実装する。

 .. code-block:: java
    :emphasize-lines: 70-75

    package todo.api.todo;

    import java.util.ArrayList;
    import java.util.Collection;
    import java.util.List;

    import javax.inject.Inject;

    import org.dozer.Mapper;
    import org.springframework.http.HttpStatus;
    import org.springframework.stereotype.Controller;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.PathVariable;
    import org.springframework.web.bind.annotation.RequestBody;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.bind.annotation.ResponseBody;
    import org.springframework.web.bind.annotation.ResponseStatus;

    import todo.domain.model.Todo;
    import todo.domain.service.todo.TodoService;

    @Controller
    @RequestMapping("todos")
    public class TodoRestController {
        @Inject
        TodoService todoService;
        @Inject
        Mapper beanMapper;

        @RequestMapping(method = RequestMethod.GET)
        @ResponseBody
        @ResponseStatus(HttpStatus.OK)
        public List<TodoResource> getTodos() {
            Collection<Todo> todos = todoService.findAll();
            List<TodoResource> todoResources = new ArrayList<>();
            for (Todo todo : todos) {
                todoResources.add(beanMapper.map(todo, TodoResource.class));
            }
            return todoResources;
        }

        @RequestMapping(method = RequestMethod.POST)
        @ResponseBody
        @ResponseStatus(HttpStatus.CREATED)
        public TodoResource postTodos(@RequestBody @Validated TodoResource todoResource) {
            Todo createdTodo = todoService.create(beanMapper.map(todoResource, Todo.class));
            TodoResource createdTodoResponse = beanMapper.map(createdTodo, TodoResource.class);
            return createdTodoResponse;
        }

        @RequestMapping(value="{todoId}", method = RequestMethod.GET)
        @ResponseBody
        @ResponseStatus(HttpStatus.OK)
        public TodoResource getTodo(@PathVariable("todoId") String todoId) {
            Todo todo = todoService.findOne(todoId);
            TodoResource todoResource = beanMapper.map(todo, TodoResource.class);
            return todoResource;
        }

        @RequestMapping(value="{todoId}", method = RequestMethod.PUT)
        @ResponseBody
        @ResponseStatus(HttpStatus.OK)
        public TodoResource putTodo(@PathVariable("todoId") String todoId) {
            Todo finishedTodo = todoService.finish(todoId);
            TodoResource finishedTodoResource = beanMapper.map(finishedTodo, TodoResource.class);
            return finishedTodoResource;
        }
        
        @RequestMapping(value="{todoId}", method = RequestMethod.DELETE) // (1)
        @ResponseBody
        @ResponseStatus(HttpStatus.NO_CONTENT) // (2)
        public void deleteTodo(@PathVariable("todoId") String todoId) { // (3)
            todoService.delete(todoId); // (4)
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | パスから\ ``todoId``\を取得するために、\ ``@RequestMapping``\アノテーションの\ ``value``\属性にパス変数を指定する。
       | メソッドがDELETEのリクエストを処理するために、\ ``method``\ 属性に\ ``RequestMethod.DELETE``\ を設定する。
   * - | (2)
     - | 応答するHTTPステータスコードを\ ``@ResponseStatus``\ アノテーションに指定する。
       | HTTPステータスとして、"204 No Content"を設定するため、value属性には\ ``HttpStatus.NO_CONTENT``\を設定する。
   * - | (3)
     - | DELETEの場合は返却するコンテンツがないため、返り値の型を\ ``void``\ とする。
   * - | (4)
     - | パス変数から取得した\ ``todoId``\ を使用して、Todoリソースを削除する。


| HTTP Dev Clientを使用して、実装したAPIの動作確認を行う。
| Dev HTTP Clientを開いてURLに\ ``"localhost:8080/todo-api/api/v1/todos/{todoId}"``\を入力し、メソッドにDELETEを指定する。
| \ ``{todoId}``\の部分は実際のIDを入れる必要があるので、POST TodosまたはGET Todosを実行してResponse中の\ ``todoId``\をコピーして貼り付けてから、"Send"ボタンをクリックする。

 .. figure:: ./images_rest/delete-todo1.png
   :width: 100%

"204 No Content"のHTTPステータスが返却され、「RESPONSE」の「Body」は空である。

 .. figure:: ./images_rest/delete-todo2.png
   :width: 100%

| Dev HTTP ClientのURLに\ ``"localhost:8080/todo-api/api/v1/todos"``\を入力し、メソッドにGETを指定してから"Send"ボタンをクリックする。
| Todoリソースが削除されている事が確認できる。

 .. figure:: ./images_rest/delete-todo3.png
   :width: 100%

例外ハンドリングの実装
--------------------------------------------------------------------------------
| 本チュートリアルでは、例外ハンドリングの実装方法をイメージしやすくするため、本ガイドラインで推奨している実装よりシンプルな実装にしてある。
| 実際の例外ハンドリングは、\ :doc:`../ArchitectureInDetail/REST`\で\ **説明されている方法でハンドリングを行うことを強く推奨する**\ 。

ドメイン層の実装を変更
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  
| 本チュートリアルでは、エラーコードをキーにプロパティファイルからエラーメッセージを取得する。
| そのため、例外ハンドリングの実装を行う前に、\ :doc:`../TutorialTodo/index`\で作成したServiceクラスの実装を以下のように変更する。

* :file:`TodoServiceImpl.java`

 ハードコーディングされていたエラーメッセージの代わりに、エラーコードを指定するように変更する。
 
 .. code-block:: java
    :emphasize-lines: 32, 49, 70

    package todo.domain.service.todo;
    
    import java.util.Collection;
    import java.util.Date;
    import java.util.UUID;
    
    import javax.inject.Inject;
    
    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;
    import org.terasoluna.gfw.common.exception.BusinessException;
    import org.terasoluna.gfw.common.exception.ResourceNotFoundException;
    import org.terasoluna.gfw.common.message.ResultMessages;
    
    import todo.domain.model.Todo;
    import todo.domain.repository.todo.TodoRepository;
    
    @Service
    @Transactional
    public class TodoServiceImpl implements TodoService {
        @Inject
        TodoRepository todoRepository;
    
        private static final long MAX_UNFINISHED_COUNT = 5;
    
        @Override
        @Transactional(readOnly = true)
        public Todo findOne(String todoId) {
            Todo todo = todoRepository.findOne(todoId);
            if (todo == null) {
                ResultMessages messages = ResultMessages.error();
                messages.add("E404", todoId);
                throw new ResourceNotFoundException(messages);
            }
            return todo;
        }
    
        @Override
        @Transactional(readOnly = true)
        public Collection<Todo> findAll() {
            return todoRepository.findAll();
        }
    
        @Override
        public Todo create(Todo todo) {
            long unfinishedCount = todoRepository.countByFinished(false);
            if (unfinishedCount >= MAX_UNFINISHED_COUNT) {
                ResultMessages messages = ResultMessages.error();
                messages.add("E001", MAX_UNFINISHED_COUNT);
                throw new BusinessException(messages);
            }
    
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
                messages.add("E002", todoId);
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

エラーメッセージの定義
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  
| 本チュートリアルでは、エラーコードをキーにプロパティファイルからエラーメッセージを取得する。
| そのため、例外ハンドリングの実装を行う前に、エラーコードに対応するエラーメッセージを、メッセージ用のプロパティファイルに定義する。

処理結果用のエラーコードに対応するエラーメッセージを、メッセージ用のプロパティファイル(\ :file:`src/main/resources/i18n/application-messages.properties`\)に定義する。

* :file:`application-messages.properties`

 ハイライトした部分のメッセージ定義を追加する。

 .. code-block:: properties
    :emphasize-lines: 28-33

    e.xx.fw.5001 = Resource not found.
    
    e.xx.fw.7001 = Illegal screen flow detected!
    e.xx.fw.7002 = CSRF attack detected!
    
    e.xx.fw.8001 = Business error occurred!
    
    e.xx.fw.9001 = System error occurred!
    e.xx.fw.9002 = Data Access error!
    
    # typemismatch
    typeMismatch="{0}" is invalid.
    typeMismatch.int="{0}" must be an integer.
    typeMismatch.double="{0}" must be a double.
    typeMismatch.float="{0}" must be a float.
    typeMismatch.long="{0}" must be a long.
    typeMismatch.short="{0}" must be a short.
    typeMismatch.boolean="{0}" must be a boolean.
    typeMismatch.java.lang.Integer="{0}" must be an integer.
    typeMismatch.java.lang.Double="{0}" must be a double.
    typeMismatch.java.lang.Float="{0}" must be a float.
    typeMismatch.java.lang.Long="{0}" must be a long.
    typeMismatch.java.lang.Short="{0}" must be a short.
    typeMismatch.java.lang.Boolean="{0}" is not a boolean.
    typeMismatch.java.util.Date="{0}" is not a date.
    typeMismatch.java.lang.Enum="{0}" is not a valid value.

    E001 = [E001] The count of un-finished Todo must not be over {0}.
    E002 = [E002] The requested Todo is already finished. (id={0})
    E400 = [E400] The requested Todo contains invalid values.
    E404 = [E404] The requested Todo is not found. (id={0})
    E500 = [E500] System error occurred.
    E999 = [E999] Error occurred. Caused by : {0}

 .. figure:: ./images_rest/application-messages.png

|

| 入力チェック用のエラーコードに対応するエラーメッセージを、Bean Validationのメッセージ用のプロパティファイル(\ :file:`src/main/resources/ValidationMessages.properties`\)に定義する。
| 

* :file:`ValidationMessages.properties`

 | デフォルトのメッセージは、メッセージの中に項目名が含まれないため、デフォルトのメッセージ定義を変更する。
 | 本チュートリアルでは、\ ``TodoResource``\クラスで使用しているルール(\ ``@NotNull``\と\ ``@Size``\)に対応するメッセージのみ定義する。

 .. code-block:: properties
    :emphasize-lines: 1-2

    javax.validation.constraints.NotNull.message = {0} may not be null.
    javax.validation.constraints.Size.message    = {0} size must be between {min} and {max}.

 .. figure:: ./images_rest/validation-messages.png

 
エラーハンドリング用のクラスを格納するパッケージの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  
| エラーハンドリング用のクラスを格納するためのパッケージを作成する。
| 本チュートリアルでは、\ ``todo.api.common.error``\をエラーハンドリング用のクラスを格納するためのパッケージとする。

 .. figure:: ./images_rest/exception-package.png


REST APIのエラーハンドリングを行うクラスの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  

| REST APIのエラーハンドリングは、Spring MVCから提供されている\ ``org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler``\を継承したクラスを作成し、\ ``@ControllerAdvice``\アノテーションを付与する方法でハンドリングする事を推奨する。
| 以下に、\ ``ResponseEntityExceptionHandler``\を継承した\ ``todo.api.common.error.RestGlobalExceptionHandler``\ クラスを作成する。

* :file:`RestGlobalExceptionHandler.java`

 .. code-block:: java

    package todo.api.common.error;
    
    import org.springframework.web.bind.annotation.ControllerAdvice;
    import org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler;
    
    @ControllerAdvice
    public class RestGlobalExceptionHandler extends ResponseEntityExceptionHandler {
    
    }

 .. figure:: ./images_rest/exception-handlingclass.png

REST APIのエラー情報を保持するJavaBeanの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  
| REST APIで発生したエラー情報を保持するクラスとして、\ ``ApiError``\クラスを\ ``todo.api.common.error``\ パッケージに作成する。
| \ ``ApiError``\クラスがJSONに変換されて、クライアントに応答される。

* :file:`ApiError.java`

 .. code-block:: java

    package todo.api.common.error;
    
    import java.util.ArrayList;
    import java.util.List;
    
    import org.codehaus.jackson.map.annotate.JsonSerialize;
    import org.codehaus.jackson.map.annotate.JsonSerialize.Inclusion;
    
    public class ApiError {
    
        private final String code;
    
        private final String message;
    
        @JsonSerialize(include = Inclusion.NON_EMPTY)
        private final String target;
    
        @JsonSerialize(include = Inclusion.NON_EMPTY)
        private final List<ApiError> details = new ArrayList<>();
    
        public ApiError(String code, String message) {
            this(code, message, null);
        }
    
        public ApiError(String code, String message, String target) {
            this.code = code;
            this.message = message;
            this.target = target;
        }
    
        public String getCode() {
            return code;
        }
    
        public String getMessage() {
            return message;
        }
    
        public String getTarget() {
            return target;
        }
    
        public List<ApiError> getDetails() {
            return details;
        }
    
        public void addDetail(ApiError detail) {
            details.add(detail);
        }
    
    }

 .. figure:: ./images_rest/exception-apierror.png

HTTPレスポンスBODYにエラー情報を出力するための実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  
\ ``ResponseEntityExceptionHandler``\ はデフォルトではHTTPステータス(400や500など)の設定のみを行い、HTTPレスポンスのBODYは設定しない。
そのため、\ ``handleExceptionInternal``\ メソッドを以下のようにオーバーライドして、BODYを出力するように実装する。

* :file:`RestGlobalExceptionHandler.java`

 .. code-block:: java
    :emphasize-lines: 3, 5-8, 10, 16-17, 19-28, 30-34

    package todo.api.common.error;

    import javax.inject.Inject;

    import org.springframework.context.MessageSource;
    import org.springframework.http.HttpHeaders;
    import org.springframework.http.HttpStatus;
    import org.springframework.http.ResponseEntity;
    import org.springframework.web.bind.annotation.ControllerAdvice;
    import org.springframework.web.context.request.WebRequest;
    import org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler;

    @ControllerAdvice
    public class RestGlobalExceptionHandler extends ResponseEntityExceptionHandler {

        @Inject
        MessageSource messageSource;

        @Override
        protected ResponseEntity<Object> handleExceptionInternal(Exception ex,
                Object body, HttpHeaders headers, HttpStatus status,
                WebRequest request) {
            Object responseBody = body;
            if (body == null) {
                responseBody = createApiError(request, "E999", ex.getMessage());
            }
            return new ResponseEntity<Object>(responseBody, headers, status);
        }

        private ApiError createApiError(WebRequest request, String errorCode,
                Object... args) {
            return new ApiError(errorCode, messageSource.getMessage(errorCode,
                    args, request.getLocale()));
        }

    }
    
| 上記実装を行う事で、\ ``ResponseEntityExceptionHandler``\でハンドリングされる例外については、HTTPレスポンスBODYにエラー情報が出力される。
| \ ``ResponseEntityExceptionHandler``\でハンドリングされる例外については、\ :ref:`exception-handling-appendix-defaulthandlerexceptionresolver-label`\を参照されたい。

| HTTP Dev Clientを使用して、実装したエラーハンドリングの動作確認を行う。
| Dev HTTP Clientを開いてURLに\ ``"localhost:8080/todo-api/api/v1/todos"``\を入力し、メソッドにPUTを指定してから、"Send"ボタンをクリックする。

"405 Method Not Allowed"のHTTPステータスが返却され、「RESPONSE」の「Body」には、エラー情報のJSONが表示される。

 .. figure:: ./images_rest/exception-genericerror.png
   :width: 100%


入力エラーのエラーハンドリングの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  
入力エラーの種類は、

* \ ``org.springframework.web.bind.MethodArgumentNotValidException``\ 
* \ ``org.springframework.validation.BindException``\ 
* \ ``org.springframework.http.converter.HttpMessageNotReadableException``\ 
* \ ``org.springframework.beans.TypeMismatchException``\ 

となる。

| 本チュートリアルでは、\ ``MethodArgumentNotValidException``\ のエラーハンドリングの実装を行う。
| \ ``MethodArgumentNotValidException``\は、HTTPリクエストBODYに格納されているデータに入力エラーがあった場合に発生する例外である。

* :file:`RestGlobalExceptionHandler.java`

 .. code-block:: java
    :emphasize-lines: 6, 10-12, 40-54, 56-61

    package todo.api.common.error;
    
    import javax.inject.Inject;
    
    import org.springframework.context.MessageSource;
    import org.springframework.context.support.DefaultMessageSourceResolvable;
    import org.springframework.http.HttpHeaders;
    import org.springframework.http.HttpStatus;
    import org.springframework.http.ResponseEntity;
    import org.springframework.validation.FieldError;
    import org.springframework.validation.ObjectError;
    import org.springframework.web.bind.MethodArgumentNotValidException;
    import org.springframework.web.bind.annotation.ControllerAdvice;
    import org.springframework.web.context.request.WebRequest;
    import org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler;
    
    @ControllerAdvice
    public class RestGlobalExceptionHandler extends ResponseEntityExceptionHandler {
    
        @Inject
        MessageSource messageSource;
    
        @Override
        protected ResponseEntity<Object> handleExceptionInternal(Exception ex,
                Object body, HttpHeaders headers, HttpStatus status,
                WebRequest request) {
            Object responseBody = body;
            if (body == null) {
                responseBody = createApiError(request, "E999", ex.getMessage());
            }
            return new ResponseEntity<Object>(responseBody, headers, status);
        }
    
        private ApiError createApiError(WebRequest request, String errorCode,
                Object... args) {
            return new ApiError(errorCode, messageSource.getMessage(errorCode,
                    args, request.getLocale()));
        }
    
        @Override
        protected ResponseEntity<Object> handleMethodArgumentNotValid(
                MethodArgumentNotValidException ex, HttpHeaders headers,
                HttpStatus status, WebRequest request) {
            ApiError apiError = createApiError(request, "E400");
            for (FieldError fieldError : ex.getBindingResult().getFieldErrors()) {
                apiError.addDetail(createApiError(request, fieldError, fieldError
                        .getField()));
            }
            for (ObjectError objectError : ex.getBindingResult().getGlobalErrors()) {
                apiError.addDetail(createApiError(request, objectError, objectError
                        .getObjectName()));
            }
            return handleExceptionInternal(ex, apiError, headers, status, request);
        }
    
        private ApiError createApiError(WebRequest request,
                DefaultMessageSourceResolvable messageSourceResolvable,
                String target) {
            return new ApiError(messageSourceResolvable.getCode(), messageSource
                    .getMessage(messageSourceResolvable, request.getLocale()), target);
        }
    
    }


| HTTP Dev Clientを使用して、実装したエラーハンドリングの動作確認を行う。
| Dev HTTP Clientを開いてURLに\ ``"localhost:8080/todo-api/api/v1/todos"``\を入力し、メソッドにPOSTを指定する。
| 「REQUEST」の「BODY」に以下のJSONを入力する。

 .. code-block:: json

    {
      "todoTitle": null
    }

また、「REQUEST」の「HEADERS」の「+」ボタンでHTTPヘッダーを追加し、「Content-Type」に「application/json」を設定後、”Send”ボタンをクリックする。

| "400 Bad Request"のHTTPステータスが返却され、「RESPONSE」の「Body」には、エラー情報のJSONが表示される。
| \ ``todoTitle``\は必須項目なので、必須エラーが発生している。

 .. figure:: ./images_rest/exception-inputerror.png
   :width: 100%

業務例外のエラーハンドリングの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  
\ ``RestGlobalExceptionHandler``\ に\ ``org.terasoluna.gfw.common.exception.BusinessException``\ をハンドリングするメソッドを追加して、業務例外をハンドリングする。

業務例外が発生した場合は、"409 Conflict"のHTTPステータスを設定する。

* :file:`RestGlobalExceptionHandler.java`

 .. code-block:: java
    :emphasize-lines: 14, 17-19, 67-72, 74-81

    package todo.api.common.error;

    import javax.inject.Inject;

    import org.springframework.context.MessageSource;
    import org.springframework.context.support.DefaultMessageSourceResolvable;
    import org.springframework.http.HttpHeaders;
    import org.springframework.http.HttpStatus;
    import org.springframework.http.ResponseEntity;
    import org.springframework.validation.FieldError;
    import org.springframework.validation.ObjectError;
    import org.springframework.web.bind.MethodArgumentNotValidException;
    import org.springframework.web.bind.annotation.ControllerAdvice;
    import org.springframework.web.bind.annotation.ExceptionHandler;
    import org.springframework.web.context.request.WebRequest;
    import org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler;
    import org.terasoluna.gfw.common.exception.BusinessException;
    import org.terasoluna.gfw.common.exception.ResultMessagesNotificationException;
    import org.terasoluna.gfw.common.message.ResultMessage;

    @ControllerAdvice
    public class RestGlobalExceptionHandler extends ResponseEntityExceptionHandler {

        @Inject
        MessageSource messageSource;

        @Override
        protected ResponseEntity<Object> handleExceptionInternal(Exception ex,
                Object body, HttpHeaders headers, HttpStatus status,
                WebRequest request) {
            Object responseBody = body;
            if (body == null) {
                responseBody = createApiError(request, "E999", ex.getMessage());
            }
            return new ResponseEntity<Object>(responseBody, headers, status);
        }

        private ApiError createApiError(WebRequest request, String errorCode,
                Object... args) {
            return new ApiError(errorCode, messageSource.getMessage(errorCode,
                    args, request.getLocale()));
        }

        @Override
        protected ResponseEntity<Object> handleMethodArgumentNotValid(
                MethodArgumentNotValidException ex, HttpHeaders headers,
                HttpStatus status, WebRequest request) {
            ApiError apiError = createApiError(request, "E400");
            for (FieldError fieldError : ex.getBindingResult().getFieldErrors()) {
                apiError.addDetail(createApiError(request, fieldError, fieldError
                        .getField()));
            }
            for (ObjectError objectError : ex.getBindingResult().getGlobalErrors()) {
                apiError.addDetail(createApiError(request, objectError, objectError
                        .getObjectName()));
            }
            return handleExceptionInternal(ex, apiError, headers, status, request);
        }

        private ApiError createApiError(WebRequest request,
                DefaultMessageSourceResolvable messageSourceResolvable,
                String target) {
            return new ApiError(messageSourceResolvable.getCode(), messageSource
                    .getMessage(messageSourceResolvable, request.getLocale()), target);
        }

        @ExceptionHandler(BusinessException.class)
        public ResponseEntity<Object> handleBusinessException(BusinessException ex,
                WebRequest request) {
            return handleResultMessagesNotificationException(ex, null,
                    HttpStatus.CONFLICT, request);
        }

        private ResponseEntity<Object> handleResultMessagesNotificationException(
                ResultMessagesNotificationException ex, HttpHeaders headers,
                HttpStatus status, WebRequest request) {
            ResultMessage message = ex.getResultMessages().iterator().next();
            ApiError apiError = createApiError(request, message.getCode(), message
                    .getArgs());
            return handleExceptionInternal(ex, apiError, headers, status, request);
        }

    }

| HTTP Dev Clientを使用して、実装したエラーハンドリングの動作確認を行う。
| Dev HTTP Clientを開いてURLに\ ``"localhost:8080/todo-api/api/v1/todos/{todoId}"``\を入力し、メソッドにPUTを指定する。
| {todoId}の部分は実際のIDを入れる必要があるので、POST TodosまたはGET Todosを実行してResponse中のtodoIdをコピーして貼り付けてから、”Send”ボタンを2回クリックする。
| 未完了状態のTodoのtodoIdを指定すること。

2回目のリクエストに対するレスポンスとして、"409 Conflict"のHTTPステータスが返却され、「RESPONSE」の「Body」には、エラー情報のJSONが表示される。

 .. figure:: ./images_rest/exception-businesserror.png
   :width: 100%



リソース未検出例外のエラーハンドリングの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  
\ ``RestGlobalExceptionHandler``\ に\ ``org.terasoluna.gfw.common.exception.ResourceNotFoundException``\ をハンドリングするメソッドを追加して、リソース未検出例外をハンドリングする。

| リソース未検出例外が発生した場合、"404 NotFound"のHTTPステータスを設定する。

* :file:`RestGlobalExceptionHandler.java`

 .. code-block:: java
    :emphasize-lines: 18, 84-89

    package todo.api.common.error;

    import javax.inject.Inject;

    import org.springframework.context.MessageSource;
    import org.springframework.context.support.DefaultMessageSourceResolvable;
    import org.springframework.http.HttpHeaders;
    import org.springframework.http.HttpStatus;
    import org.springframework.http.ResponseEntity;
    import org.springframework.validation.FieldError;
    import org.springframework.validation.ObjectError;
    import org.springframework.web.bind.MethodArgumentNotValidException;
    import org.springframework.web.bind.annotation.ControllerAdvice;
    import org.springframework.web.bind.annotation.ExceptionHandler;
    import org.springframework.web.context.request.WebRequest;
    import org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler;
    import org.terasoluna.gfw.common.exception.BusinessException;
    import org.terasoluna.gfw.common.exception.ResourceNotFoundException;
    import org.terasoluna.gfw.common.exception.ResultMessagesNotificationException;
    import org.terasoluna.gfw.common.message.ResultMessage;

    @ControllerAdvice
    public class RestGlobalExceptionHandler extends ResponseEntityExceptionHandler {

        @Inject
        MessageSource messageSource;

        @Override
        protected ResponseEntity<Object> handleExceptionInternal(Exception ex,
                Object body, HttpHeaders headers, HttpStatus status,
                WebRequest request) {
            Object responseBody = body;
            if (body == null) {
                responseBody = createApiError(request, "E999", ex.getMessage());
            }
            return new ResponseEntity<Object>(responseBody, headers, status);
        }

        private ApiError createApiError(WebRequest request, String errorCode,
                Object... args) {
            return new ApiError(errorCode, messageSource.getMessage(errorCode,
                    args, request.getLocale()));
        }

        @Override
        protected ResponseEntity<Object> handleMethodArgumentNotValid(
                MethodArgumentNotValidException ex, HttpHeaders headers,
                HttpStatus status, WebRequest request) {
            ApiError apiError = createApiError(request, "E400");
            for (FieldError fieldError : ex.getBindingResult().getFieldErrors()) {
                apiError.addDetail(createApiError(request, fieldError, fieldError
                        .getField()));
            }
            for (ObjectError objectError : ex.getBindingResult().getGlobalErrors()) {
                apiError.addDetail(createApiError(request, objectError, objectError
                        .getObjectName()));
            }
            return handleExceptionInternal(ex, apiError, headers, status, request);
        }

        private ApiError createApiError(WebRequest request,
                DefaultMessageSourceResolvable messageSourceResolvable,
                String target) {
            return new ApiError(messageSourceResolvable.getCode(), messageSource
                    .getMessage(messageSourceResolvable, request.getLocale()), target);
        }

        @ExceptionHandler(BusinessException.class)
        public ResponseEntity<Object> handleBusinessException(BusinessException ex,
                WebRequest request) {
            return handleResultMessagesNotificationException(ex, null,
                    HttpStatus.CONFLICT, request);
        }

        private ResponseEntity<Object> handleResultMessagesNotificationException(
                ResultMessagesNotificationException ex, HttpHeaders headers,
                HttpStatus status, WebRequest request) {
            ResultMessage message = ex.getResultMessages().iterator().next();
            ApiError apiError = createApiError(request, message.getCode(), message
                    .getArgs());
            return handleExceptionInternal(ex, apiError, headers, status, request);
        }

        @ExceptionHandler(ResourceNotFoundException.class)
        public ResponseEntity<Object> handleResourceNotFoundException(
                ResourceNotFoundException ex, WebRequest request) {
            return handleResultMessagesNotificationException(ex, null,
                    HttpStatus.NOT_FOUND, request);
        }

    }


| HTTP Dev Clientを使用して、実装したエラーハンドリングの動作確認を行う。
| Dev HTTP Clientを開いてURLに\ ``"localhost:8080/todo-api/api/v1/todos/{todoId}"``\を入力し、メソッドにGETを指定する。
| {todoId}の部分には存在しないIDを指定して、”Send”ボタンをクリックする。

"404 Not Found"のHTTPステータスが返却され、「RESPONSE」の「Body」には、エラー情報のJSONが表示される。

 .. figure:: ./images_rest/exception-notfound.png
   :width: 100%


システム例外のエラーハンドリングの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
最後に、\ ``RestGlobalExceptionHandler``\ に\ ``java.lang.Exception``\ をハンドリングするメソッドを追加して、システム例外をハンドリングする。

システム例外が発生した場合、"500 InternalServerError"のHTTPステータスを設定する。

* :file:`RestGlobalExceptionHandler.java`

 .. code-block:: java
    :emphasize-lines: 91-97

    package todo.api.common.error;

    import javax.inject.Inject;

    import org.springframework.context.MessageSource;
    import org.springframework.context.support.DefaultMessageSourceResolvable;
    import org.springframework.http.HttpHeaders;
    import org.springframework.http.HttpStatus;
    import org.springframework.http.ResponseEntity;
    import org.springframework.validation.FieldError;
    import org.springframework.validation.ObjectError;
    import org.springframework.web.bind.MethodArgumentNotValidException;
    import org.springframework.web.bind.annotation.ControllerAdvice;
    import org.springframework.web.bind.annotation.ExceptionHandler;
    import org.springframework.web.context.request.WebRequest;
    import org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler;
    import org.terasoluna.gfw.common.exception.BusinessException;
    import org.terasoluna.gfw.common.exception.ResourceNotFoundException;
    import org.terasoluna.gfw.common.exception.ResultMessagesNotificationException;
    import org.terasoluna.gfw.common.message.ResultMessage;

    @ControllerAdvice
    public class RestGlobalExceptionHandler extends ResponseEntityExceptionHandler {

        @Inject
        MessageSource messageSource;

        @Override
        protected ResponseEntity<Object> handleExceptionInternal(Exception ex,
                Object body, HttpHeaders headers, HttpStatus status,
                WebRequest request) {
            Object responseBody = body;
            if (body == null) {
                responseBody = createApiError(request, "E999", ex.getMessage());
            }
            return new ResponseEntity<Object>(responseBody, headers, status);
        }

        private ApiError createApiError(WebRequest request, String errorCode,
                Object... args) {
            return new ApiError(errorCode, messageSource.getMessage(errorCode,
                    args, request.getLocale()));
        }

        @Override
        protected ResponseEntity<Object> handleMethodArgumentNotValid(
                MethodArgumentNotValidException ex, HttpHeaders headers,
                HttpStatus status, WebRequest request) {
            ApiError apiError = createApiError(request, "E400");
            for (FieldError fieldError : ex.getBindingResult().getFieldErrors()) {
                apiError.addDetail(createApiError(request, fieldError, fieldError
                        .getField()));
            }
            for (ObjectError objectError : ex.getBindingResult().getGlobalErrors()) {
                apiError.addDetail(createApiError(request, objectError, objectError
                        .getObjectName()));
            }
            return handleExceptionInternal(ex, apiError, headers, status, request);
        }

        private ApiError createApiError(WebRequest request,
                DefaultMessageSourceResolvable messageSourceResolvable,
                String target) {
            return new ApiError(messageSourceResolvable.getCode(), messageSource
                    .getMessage(messageSourceResolvable, request.getLocale()), target);
        }

        @ExceptionHandler(BusinessException.class)
        public ResponseEntity<Object> handleBusinessException(BusinessException ex,
                WebRequest request) {
            return handleResultMessagesNotificationException(ex, null,
                    HttpStatus.CONFLICT, request);
        }

        private ResponseEntity<Object> handleResultMessagesNotificationException(
                ResultMessagesNotificationException ex, HttpHeaders headers,
                HttpStatus status, WebRequest request) {
            ResultMessage message = ex.getResultMessages().iterator().next();
            ApiError apiError = createApiError(request, message.getCode(), message
                    .getArgs());
            return handleExceptionInternal(ex, apiError, headers, status, request);
        }

        @ExceptionHandler(ResourceNotFoundException.class)
        public ResponseEntity<Object> handleResourceNotFoundException(
                ResourceNotFoundException ex, WebRequest request) {
            return handleResultMessagesNotificationException(ex, null,
                    HttpStatus.NOT_FOUND, request);
        }

        @ExceptionHandler(Exception.class)
        public ResponseEntity<Object> handleSystemError(Exception ex,
                WebRequest request) {
            ApiError apiError = createApiError(request, "E500");
            return handleExceptionInternal(ex, apiError, null,
                    HttpStatus.INTERNAL_SERVER_ERROR, request);
        }

    }

    
| HTTP Dev Clientを使用して、実装したエラーハンドリングの動作確認を行う。
| システムエラーを発生させるために、テーブルを未作成の状態でアプリケーションを起動させる。

* :file:`todo-api-infra.properties`

 .. code-block:: properties
    :emphasize-lines: 3

    database=H2
    #database.url=jdbc:h2:mem:todo-api;DB_CLOSE_DELAY=-1;INIT=create table if not exists todo(todo_id varchar(36) primary key, todo_title varchar(30), finished boolean, created_at timestamp)
    database.url=jdbc:h2:mem:todo-api;DB_CLOSE_DELAY=-1
    database.username=sa
    database.password=
    database.driverClassName=org.h2.Driver
    # connection pool
    cp.maxActive=96
    cp.maxIdle=16
    cp.minIdle=0
    cp.maxWait=60000
    
 
| Dev HTTP Clientを開いてURLに\ ``"localhost:8080/todo-api/api/v1/todos"``\を入力し、メソッドにGETを指定して、”Send”ボタンをクリックする。

"500 Internal Server Error"のHTTPステータスが返却され、「RESPONSE」の「Body」には、エラー情報のJSONが表示される。

 .. figure:: ./images_rest/exception-systemerror.png
   :width: 100%

 .. note::

    システムエラーが発生した場合、クライアントへ返却するメッセージは、エラー原因が特定されないシンプルなエラーメッセージを設定することを推奨する。
    エラー原因が特定できるメッセージを設定してしまうと、システムの脆弱性をクライアントに公開する可能性があり、セキュリティー上問題がある。
    
    エラー原因は、エラー解析用にログに出力すればよい。
    Blankプロジェクトのデフォルトの設定では、共通ライブラリから提供している\ ``ExceptionLogger``\によってログが出力されるようなっているため、ログを出力するための設定や実装は不要である。

    \ ``ExceptionLogger``\によって出力されるログは以下の通りである。
    Todoテーブルが存在しない事が原因でシステムエラーが発生している事がわかる。

     .. code-block:: text
        :emphasize-lines: 5
     
        date:2014-04-04 15:15:07    thread:tomcat-http--3   X-Track:09689dcae9d04ad3b25650cba2ebbe3a    level:ERROR logger:o.t.gfw.common.exception.ExceptionLogger         message:[e.xx.fw.9001] SqlMapClient operation; bad SQL grammar []; nested exception is com.ibatis.common.jdbc.exception.NestedSQLException:   
        --- The error occurred while executing query.  
        --- Check the  SELECT todo_id,        todo_title,        finished,        created_at FROM   todo .  
        --- Check the SQL Statement (preparation failed).  
        --- Cause: org.h2.jdbc.JdbcSQLException: Table "TODO" not found; SQL statement:
         SELECT todo_id,        todo_title,        finished,        created_at FROM   todo  [42102-172]
        org.springframework.jdbc.BadSqlGrammarException: SqlMapClient operation; bad SQL grammar []; nested exception is com.ibatis.common.jdbc.exception.NestedSQLException:   
        --- The error occurred while executing query.  
        --- Check the  SELECT todo_id,        todo_title,        finished,        created_at FROM   todo .  
        --- Check the SQL Statement (preparation failed).  
        --- Cause: org.h2.jdbc.JdbcSQLException: Table "TODO" not found; SQL statement:
         SELECT todo_id,        todo_title,        finished,        created_at FROM   todo  [42102-172]
            at org.springframework.jdbc.support.SQLErrorCodeSQLExceptionTranslator.doTranslate(SQLErrorCodeSQLExceptionTranslator.java:237) ~[spring-jdbc-3.2.4.RELEASE.jar:3.2.4.RELEASE]
            at org.springframework.jdbc.support.AbstractFallbackSQLExceptionTranslator.translate(AbstractFallbackSQLExceptionTranslator.java:72) ~[spring-jdbc-3.2.4.RELEASE.jar:3.2.4.RELEASE]
            at org.springframework.orm.ibatis.SqlMapClientTemplate.execute(SqlMapClientTemplate.java:206) ~[spring-orm-3.2.4.RELEASE.jar:3.2.4.RELEASE]
            at org.springframework.orm.ibatis.SqlMapClientTemplate.queryForList(SqlMapClientTemplate.java:296) ~[spring-orm-3.2.4.RELEASE.jar:3.2.4.RELEASE]
            at jp.terasoluna.fw.dao.ibatis.QueryDAOiBatisImpl.executeForObjectList(QueryDAOiBatisImpl.java:421) ~[terasoluna-ibatis-2.0.5.0.jar:2.0.5.0]
            at todo.domain.repository.todo.TodoRepositoryImpl.findAll(TodoRepositoryImpl.java:33) ~[TodoRepositoryImpl.class:na]
            at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.7.0_51]
            ...
            
            (omitted)

おわりに
================================================================================
このチュートリアルでは、以下の内容を学習した。

* TERASOLUNA Global Frameworkによる基本的なRESTful Webサービスの構築方法
* REST API(GET, POST, PUT, DELETE)を提供するControllerクラスの実装
* JavaBeanとJSONの相互変換方法
* エラーメッセージの定義方法
* Spring MVCを使用した各種例外のハンドリング方法

ここでは、基本的なRESTful Webサービスの実装法について示した。
考え方の元となるアーキテクチャ・設計指針等について理解を深める為には、「:doc:`../ArchitectureInDetail/REST`」 を参照されたい。

.. raw:: latex

   \newpage

