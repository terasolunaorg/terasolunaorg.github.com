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

* TERASOLUNA Server Framework for Java (5.x)による基本的なアプリケーションの開発方法
* MavenおよびSTS(Eclipse)プロジェクトの構築方法
* TERASOLUNA Server Framework for Java (5.x)の :doc:`../Overview/ApplicationLayering` に従った開発方法


対象読者
--------------------------------------------------------------------------------

* SpringのDIやAOPに関する基礎的な知識がある
* Servlet/JSPを使用してWebアプリケーションを開発したことがある
* SQLに関する知識がある


検証環境
--------------------------------------------------------------------------------

このチュートリアルは以下の環境で動作確認している。他の環境で実施する際は本書をベースに適宜読み替えて設定していくこと。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - 種別
      - 名前
    * - OS
      - Windows 7
    * - JVM
      - `Java <http://www.oracle.com/technetwork/java/javase/downloads/index.html>`_ 1.7
    * - IDE
      - `Spring Tool Suite <http://spring.io/tools/sts/all>`_ 3.6.3.RELEASE (以降「STS」と呼ぶ)
    * - Build Tool
      - `Apache Maven <http://maven.apache.org/download.cgi>`_ 3.2.5 (以降「Maven」と呼ぶ)
    * - Application Server
      - `Pivotal tc Server <https://network.pivotal.io/products/pivotal-tcserver>`_ Developer Edition v3.0 (STSに同封)
    * - Web Browser
      - `Google Chrome <https://www.google.co.jp/chrome/browser/desktop/index.html>`_ 39.0.2171.99 m

|

作成するアプリケーションの説明
================================================================================

アプリケーションの概要
--------------------------------------------------------------------------------

TODOを管理するアプリケーションを作成する。TODOの一覧表示、TODOの登録、TODOの完了、TODOの削除を行える。

.. figure:: ./images/image001.png
    :width: 50%

.. _app-requirement:

アプリケーションの業務要件
--------------------------------------------------------------------------------
アプリケーションの業務要件は、以下の通りとする。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 80

    * - ルールID
      - 説明
    * - B01
      - 未完了のTODOは5件までしか登録できない
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

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.10\linewidth}|p{0.15\linewidth}|p{0.45\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 20 10 15 45

    * - 項番
      - プロセス名
      - HTTPメソッド
      - URL
      - 備考
    * - 1
      - Show all TODO
      - \-
      - /todo/list
      -
    * - 2
      - Create TODO
      - POST
      - /todo/create
      - 作成処理終了後、Show all TODOへリダイレクト
    * - 3
      - Finish TODO
      - POST
      - /todo/finish
      - 完了処理終了後、Show all TODOへリダイレクト
    * - 4
      - Delete TODO
      - POST
      - /todo/delete
      - 削除処理終了後、Show all TODOへリダイレクト

Show all TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* TODOを全件表示する
* 未完了のTODOに対しては「Finish」と「Delete」用のボタンが付く
* 完了のTODOは打ち消し線で装飾する
* TODOの件名のみ表示する


Create TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* フォームから送信されたTODOを保存する
* TODOの件名は1文字以上30文字以下であること
* :ref:`app-requirement` のB01を満たさない場合はエラーコードE001でビジネス例外をスローする
* 処理が成功した場合は、遷移先の画面で「Created successfully!」を表示する

Finish TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* フォームから送信された\ ``todoId``\ に対応するTODOを完了済みにする
* 該当するTODOが存在しない場合はエラーコードE404でリソース未検出例外をスローする
* :ref:`app-requirement` のB02を満たさない場合はエラーコードE002でビジネス例外をスローする
* 処理が成功した場合は、遷移先の画面で「Finished successfully!」を表示する

Delete TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* フォームから送信された\ ``todoId``\ に対応するTODOを削除する
* 該当するTODOが存在しない場合はエラーコードE404でリソース未検出例外をスローする
* 処理が成功した場合は、遷移先の画面で「Deleted successfully!」を表示する

|

エラーメッセージ一覧
--------------------------------------------------------------------------------

エラーメッセージとして、以下の3つを定義する。

.. tabularcolumns:: |p{0.15\linewidth}|p{0.50\linewidth}|p{0.35\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 15 50 35

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

本チュートリアルでは、インフラストラクチャ層のRepositoryImplの実装として、

* データベースを使用せず\ ``java.util.Map``\ を使ったインメモリ実装のRepositoryImpl
* MyBatis3を使用してデータベースにアクセスするRepositoryImpl
* Spring Data JPAを使用してデータベースにアクセスするRepositoryImpl

の3種類を用意している。用途に応じていずれかを選択する。

チュートリアルの進行上、まずはインメモリ実装を試し、その後MyBatis3またはSpring Data JPAを選ぶのが円滑である。

プロジェクトの作成
--------------------------------------------------------------------------------

まず、\ ``mvn archetype:generate``\ を利用して、実装するインフラストラクチャ層向けのブランクプロジェクトを作成する。
ここでは、Windowsのコマンドプロンプトを使用してブランクプロジェクトを作成する手順となっている。

.. note::

    インターネット接続するために、プロキシサーバーを介する必要がある場合、
    以下の作業を行うため、STSのProxy設定と、 `MavenのProxy設定 <http://maven.apache.org/guides/mini/guide-proxies.html>`_\ が必要である。

.. tip::

    Bash上で\ ``mvn archetype:generate``\ を実行する場合は、以下のように\ ``^``\ を\ ``\``\ に置き換えて実行すればよい。

     .. code-block:: bash

        mvn archetype:generate -B\
         -DarchetypeCatalog=http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases\
         -DarchetypeGroupId=org.terasoluna.gfw.blank\
         -DarchetypeArtifactId=terasoluna-gfw-web-blank-archetype\
         -DarchetypeVersion=5.0.1.RELEASE\
         -DgroupId=todo\
         -DartifactId=todo\
         -Dversion=1.0.0-SNAPSHOT

|

.. _TutorialCreatePlainBlankProject:

O/R Mapperに依存しないブランクプロジェクトの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

データベースを使用せず\ ``java.util.Map``\ を使ったインメモリ実装のRepositoryImpl用のプロジェクトを作成する場合は、
以下のコマンドを実行してO/R Mapperに依存しないブランクプロジェクトを作成する。\ **本チュートリアルを順序通り読み進める場合は、まずはこの方法でプロジェクトを作成すること**\ 。

.. code-block:: console

    mvn archetype:generate -B^
     -DarchetypeCatalog=http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases^
     -DarchetypeGroupId=org.terasoluna.gfw.blank^
     -DarchetypeArtifactId=terasoluna-gfw-web-blank-archetype^
     -DarchetypeVersion=5.0.1.RELEASE^
     -DgroupId=todo^
     -DartifactId=todo^
     -Dversion=1.0.0-SNAPSHOT

.. _TutorialCreateMyBatis3BlankProject:

MyBatis3用のブランクプロジェクトの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

MyBatis3を使用してデータベースにアクセスするRepositoryImpl用のプロジェクトを作成する場合は、
以下のコマンドを実行してMyBatis3用のブランクプロジェクトを作成する。このプロジェクト作成方法は\ :ref:`using_MyBatis3`\ で使用する。

.. code-block:: console

    mvn archetype:generate -B^
     -DarchetypeCatalog=http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases^
     -DarchetypeGroupId=org.terasoluna.gfw.blank^
     -DarchetypeArtifactId=terasoluna-gfw-web-blank-mybatis3-archetype^
     -DarchetypeVersion=5.0.1.RELEASE^
     -DgroupId=todo^
     -DartifactId=todo^
     -Dversion=1.0.0-SNAPSHOT

.. _TutorialCreateJPABlankProject:

JPA用のブランクプロジェクトの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Data JPAの使用してデータベースにアクセスするRepositoryImpl用のプロジェクトを作成する場合は、
以下のコマンドを実行してJPA用のブランクプロジェクトを作成する。このプロジェクト作成方法は\ :ref:`using_SpringDataJPA`\ で使用する。

.. code-block:: console

    mvn archetype:generate -B^
     -DarchetypeCatalog=http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases^
     -DarchetypeGroupId=org.terasoluna.gfw.blank^
     -DarchetypeArtifactId=terasoluna-gfw-web-blank-jpa-archetype^
     -DarchetypeVersion=5.0.1.RELEASE^
     -DgroupId=todo^
     -DartifactId=todo^
     -Dversion=1.0.0-SNAPSHOT

|

プロジェクトのインポート
--------------------------------------------------------------------------------

作成したブランクプロジェクトをSTSへインポートする。

STSのメニューから、[File] -> [Import] -> [Maven] -> [Existing Maven Projects] -> [Next]を選択し、archetypeで作成したプロジェクトを選択する。

.. figure:: images/NewMVCProjectImport.png
   :alt: New MVC Project Import
   :width: 60%

|

Root Directoryに \ ``C:\work\todo``\ を設定し、Projectsにtodoのpom.xmlが選択された状態で、 [Finish] を押下する。

.. figure:: images/NewMVCProjectCreate.png
   :alt: New MVC Project Import
   :width: 60%

|

インポートが完了すると、Package Explorerに次のようなプロジェクトが表示される。

.. figure:: images/image004.png
   :alt: workspace

.. note::

    インポート後にビルドエラーが発生する場合は、プロジェクト名を右クリックし、「Maven」->「Update Project...」をクリックし、
    「OK」ボタンをクリックすることでエラーが解消されるケースがある。

     .. figure:: ./images/update-project.png
        :width: 70%

.. tip::

    パッケージの表示形式は、デフォルトは「Flat」だが、「Hierarchical」にしたほうが見通しがよい。

    Package Explorerの「View Menu」 (右端の下矢印)をクリックし、「Package Presentation」->「Hierarchical」を選択する。

     .. figure:: ./images/presentation-hierarchical.png
        :width: 80%

    Package PresentationをHierarchicalにすると、以下の様な表示になる。

     .. figure:: ./images/presentation-hierarchical-view.png

.. warning::
 
    O/R Mapperを使用するブランクプロジェクトの場合、H2 Databaseがdependencyとして定義されているが、
    この設定は簡易的なアプリケーションを簡単に作成するためのものであり、実際のアプリケーション開発で使用されることは想定していない。
    
    以下の定義は、実際のアプリケーション開発を行う際は削除すること。
    
     .. code-block:: xml

        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>

|

プロジェクトの構成
--------------------------------------------------------------------------------

本チュートリアルで作成するプロジェクトの構成を以下に示す。

.. note::

    :ref:`前節の「プロジェクト構成」 <application-layering_project-structure>` ではマルチプロジェクトにすることを推奨していたが、
    本チュートリアルでは、学習容易性を重視しているためシングルプロジェクト構成にしている。

    **ただし、実プロジェクトで適用する場合は、マルチプロジェクト構成を強く推奨する。**

    マルチプロジェクトの作成方法は、「:doc:`../ImplementationAtEachLayer/CreateWebApplicationProject`」を参照されたい。

|

**[MyBatis3用のブランクプロジェクトを作成した場合の構成]**

.. code-block:: console

    src
      └main
          ├java
          │  └todo
          │    ├ app
          │    │   └todo
          │    └domain
          │        ├model
          │        ├repository
          │        │   └todo
          │        └service
          │            └todo
          ├resources
          │  ├META-INF
          │  │  ├mybatis ... (8)
          │  │  └spring
          │  └todo
          │    └domain
          │        └repository ... (9)
          │             └todo
          └wepapp
              └WEB-INF
                  └views


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (8)
      - MyBatis関連の設定ファイルを格納するディレクトリ。
    * - | (9)
      - SQLを記述するMyBatisのMapperファイルを格納するディレクトリ。

        本チュートリアルでは、Todoオブジェクト用のRepositoryのMapperファイルを格納するためのディレクトリを作成する。

|

**[O/R Mapperに依存しないブランクプロジェクト、JPA用のブランクプロジェクト用を作成した場合の構成]**

.. code-block:: console

    src
      └main
          ├java
          │  └todo
          │    ├ app ... (1)
          │    │   └todo
          │    └domain ... (2)
          │        ├model ... (3)
          │        ├repository ... (4)
          │        │   └todo
          │        └service ... (5)
          │            └todo
          ├resources
          │  └META-INF
          │      └spring ... (6)
          └wepapp
              └WEB-INF
                  └views ... (7)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - アプリケーション層のクラスを格納するパッケージ。

        本チュートリアルでは、Todo管理業務用のクラスを格納するためのパッケージを作成する。
    * - | (2)
      - ドメイン層のクラスを格納するパッケージ。
    * - | (3)
      - Domain Objectを格納するパッケージ。
    * - | (4)
      - Repositoryを格納するパッケージ。

        本チュートリアルでは、Todoオブジェクト(Domain Object)用のRepositoryを格納するためのパッケージを作成する
    * - | (5)
      - Serviceを格納するパッケージ。

        本チュートリアルでは、Todo管理業務用のServiceを格納するためのパッケージを作成する。
    * - | (6)
      - spring関連の設定ファイルを格納するディレクトリ。
    * - | (7)
      - jspを格納するディレクトリ。

|

設定ファイルの確認
--------------------------------------------------------------------------------
チュートリアルを進める上で必要となる設定の多くは、作成したブランクプロジェクトに既に設定済みの状態である。

チュートリアルを実施するだけであれば、これらの設定の理解は必須ではないが、
アプリケーションを動かすためにどのような設定が必要なのかを理解しておくことを推奨する。

アプリケーションを動かすために必要な設定(設定ファイル)の解説については、「:ref:`TutorialTodoAppendixExpoundConfigurations`」を参照されたい。

.. note::
 
    まず、手を動かしてTodoアプリケーションを作成したい場合は、設定ファイルの確認は読み飛ばしてもよいが、
    Todoアプリケーションを作成した後に一読して頂きたい。

|

プロジェクトの動作確認
--------------------------------------------------------------------------------
Todoアプリケーションの開発を始める前に、プロジェクトの動作確認を行う。

ブランクプロジェクトでは、トップページを表示するためのControllerとJSPの実装が用意されているため、
トップページを表示する事で動作確認を行う事ができる。

ブランクプロジェクトから提供されているController(\ :file:`src/main/java/todo/app/welcome/HomeController.java`\ )は、
以下のような実装となっている。

.. code-block:: java
    :emphasize-lines: 17, 21, 28, 31, 40, 43

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
    public class HomeController {

        // (2)
        private static final Logger logger = LoggerFactory
                .getLogger(HomeController.class);

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
     - | Controllerとしてcomponent-scanの対象とするため、クラスレベルに\ ``@Controller``\ アノテーションが付与している。
   * - | (2)
     - | (4)でログ出力するためのロガーを生成している。
       | ロガーの実装はlogbackのものであるが、APIはSLF4Jの\ ``org.slf4j.Logger``\ を使用している。
   * - | (3)
     - | ``@RequestMapping`` アノテーションを使用して、\ ``"/"``\ (ルート)へのアクセスに対するメソッドとしてマッピングを行っている。
   * - | (4)
     - | メソッドが呼ばれたことを通知するためのログをinfoレベルで出力している。
   * - | (5)
     - | 画面に表示するための日付文字列を、\ ``"serverTime"``\ という属性名でModelに設定している。
   * - | (6)
     - | view名として\ ``"welcome/home"``\ を返す。\ ``ViewResolver``\ の設定により、\ ``WEB-INF/views/welcome/home.jsp``\ が呼び出される。

|

ブランクプロジェクトから提供されているJSP(\ :file:`src/main/webapp/WEB-INF/views/welcome/home.jsp`\ )は、
以下のような実装となっている。

.. code-block:: jsp
    :emphasize-lines: 12

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
            <!-- (7) -->
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
   * - | (7)
     - | ControllerでModelに設定した\ ``"serverTime"``\ を表示する。
       | ここでは、XSS対策を行っていないが、ユーザの入力値を表示する場合は、\ ``f:h()``\ 関数を用いて、必ずXSS対策を行うこと。

|

プロジェクトを右クリックして「Run As」->「Run on Server」を選択する。

.. figure:: ./images/image031.jpg
   :width: 70%

|

APサーバー(Pivotal tc Server Developer Edition v3.0)を選択し、「Next」をクリックする。

.. figure:: ./images/image032.jpg
   :width: 70%

|

todoが「Configured」に含まれていることを確認して「Finish」をクリックしてサーバーを起動する。

.. figure:: ./images/image033.jpg
   :width: 70%

|

起動すると以下のようなログが出力される。
\ ``"/"``\ というパスに対して\ ``todo.app.welcome.HomeController``\ のhelloメソッドがマッピングされていることが分かる。


.. code-block:: console
   :emphasize-lines: 3

    date:2015-01-16 21:32:05	thread:localhost-startStop-1	X-Track:	level:INFO 	logger:o.springframework.web.servlet.DispatcherServlet 	message:FrameworkServlet 'appServlet': initialization started
    date:2015-01-16 21:32:07	thread:localhost-startStop-1	X-Track:	level:DEBUG	logger:o.t.gfw.web.codelist.CodeListInterceptor        	message:registered codeList : []
    date:2015-01-16 21:32:07	thread:localhost-startStop-1	X-Track:	level:INFO 	logger:o.s.w.s.m.m.a.RequestMappingHandlerMapping      	message:Mapped "{[/],methods=[GET || POST],params=[],headers=[],consumes=[],produces=[],custom=[]}" onto public java.lang.String todo.app.welcome.HomeController.home(java.util.Locale,org.springframework.ui.Model)
    date:2015-01-16 21:32:11	thread:localhost-startStop-1	X-Track:	level:INFO 	logger:o.s.w.s.m.m.a.RequestMappingHandlerAdapter      	message:Looking for @ControllerAdvice: WebApplicationContext for namespace 'appServlet-servlet': startup date [Fri Jan 16 21:32:05 JST 2015]; parent: Root WebApplicationContext
    date:2015-01-16 21:32:12	thread:localhost-startStop-1	X-Track:	level:INFO 	logger:o.s.w.s.m.m.a.RequestMappingHandlerAdapter      	message:Looking for @ControllerAdvice: WebApplicationContext for namespace 'appServlet-servlet': startup date [Fri Jan 16 21:32:05 JST 2015]; parent: Root WebApplicationContext
    date:2015-01-16 21:32:12	thread:localhost-startStop-1	X-Track:	level:INFO 	logger:o.s.web.servlet.handler.SimpleUrlHandlerMapping 	message:Mapped URL path [/**] onto handler 'org.springframework.web.servlet.resource.DefaultServletHttpRequestHandler#0'
    date:2015-01-16 21:32:12	thread:localhost-startStop-1	X-Track:	level:INFO 	logger:o.s.web.servlet.handler.SimpleUrlHandlerMapping 	message:Mapped URL path [/resources/**] onto handler 'org.springframework.web.servlet.resource.ResourceHttpRequestHandler#0'
    date:2015-01-16 21:32:12	thread:localhost-startStop-1	X-Track:	level:INFO 	logger:o.springframework.web.servlet.DispatcherServlet 	message:FrameworkServlet 'appServlet': initialization completed in 6957 ms

|

ブラウザで http://localhost:8080/todo にアクセスすると、以下のように表示される。

.. figure:: ./images/image034.png
   :width: 60%


コンソールを見ると、

* 共通ライブラリから提供している\ ``TraceLoggingInterceptor``\ のTRACEログ
* Controllerで実装したINFOログ

が出力されていることがわかる。

.. code-block:: console
   :emphasize-lines: 1-2,4-5

    date:2015-01-16 21:36:36	thread:tomcat-http--11	X-Track:2c4902f4fe5a477b8ad8aefb10973c04	level:TRACE	logger:o.t.gfw.web.logging.TraceLoggingInterceptor     	message:[START CONTROLLER] HomeController.home(Locale,Model)
    date:2015-01-16 21:36:36	thread:tomcat-http--11	X-Track:2c4902f4fe5a477b8ad8aefb10973c04	level:INFO 	logger:todo.app.welcome.HomeController                 	message:Welcome home! The client locale is en_US.
    date:2015-01-16 21:36:36	thread:tomcat-http--11	X-Track:2c4902f4fe5a477b8ad8aefb10973c04	level:DEBUG	logger:o.t.gfw.web.codelist.CodeListInterceptor        	message:locale for I18nCodelist is 'en_US'.
    date:2015-01-16 21:36:36	thread:tomcat-http--11	X-Track:2c4902f4fe5a477b8ad8aefb10973c04	level:TRACE	logger:o.t.gfw.web.logging.TraceLoggingInterceptor     	message:[END CONTROLLER  ] HomeController.home(Locale,Model)-> view=welcome/home, model={serverTime=January 16, 2015 9:36:36 PM JST}
    date:2015-01-16 21:36:36	thread:tomcat-http--11	X-Track:2c4902f4fe5a477b8ad8aefb10973c04	level:TRACE	logger:o.t.gfw.web.logging.TraceLoggingInterceptor     	message:[HANDLING TIME   ] HomeController.home(Locale,Model)-> 983,574 ns

.. note::
 
    \ ``TraceLoggingInterceptor``\ はControllerの開始、終了でログを出力する。終了時には\ ``View``\ と\ ``Model``\ の情報および処理時間が出力される。

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

ここでは、データベースを使用せず\ ``java.util.Map``\ を使ったインメモリ実装のRepositoryImplを作成する方法について説明を行う。
データベースを使用する場合は、「:ref:`tutorial-todo_infra`」に記載されている内容で読み替えて、Todoアプリケーションを作成して頂きたい。

|

ドメイン層の作成
--------------------------------------------------------------------------------

Domain Objectの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Domainオブジェクトを作成する。

Package Explorer上で右クリック -> New -> Class を選択し、「New Java Class」ダイアログを表示し、

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - 項番
      - 項目
      - 入力値
    * - 1
      - Package
      - ``todo.domain.model``
    * - 2
      - Name
      - ``Todo``
    * - 3
      - Interfaces
      - ``java.io.Serializable``

を入力して「Finish」する。

.. figure:: ./images/image057.png
   :width: 70%

作成したクラスは以下のディレクトリに格納される。

.. figure:: ./images/image058.png

|

作成したクラスに以下のプロパティを追加する。

* ID → todoId
* タイトル → todoTitle
* 完了フラグ → finished
* 作成日 → createdAt

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

.. tip::

    Getter/SetterメソッドはSTSの機能を使って自動生成することができる。
    フィールドを定義した後、エディタ上で右クリックし、「Source」->「Generate Getter and Setters…」を選択する。

        .. figure:: ./images/image059.png
           :width: 90%

    serialVersionUID以外を選択して「OK」

        .. figure:: ./images/image060.png
           :width: 60%

|

.. _TutorialTodoCreateRepository:

Repositoryの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ ``TodoRepository``\ インタフェースを作成する。
データベースを使用する場合は、「:ref:`tutorial-todo_infra`」に記載されている内容で読み替えて、Repositoryを作成する。

Package Explorer上で右クリック -> New -> Interface を選択し、「New Java Interface」ダイアログを表示し、

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - 項番
      - 項目
      - 入力値
    * - 1
      - Package
      - ``todo.domain.repository.todo``
    * - 2
      - Name
      - ``TodoRepository``

を入力して「Finish」する。

作成したインタフェースは以下のディレクトリに格納される。

.. figure:: ./images/image061.png

作成したインタフェースに、今回のアプリケーションで必要となる以下のCRUD操作を行うメソッドを定義する。

* TODOの1件取得 → findOne
* TODOの全件取得 → findAll
* TODOの1件作成 → create
* TODOの1件更新 → update
* TODOの1件削除 → delete
* 完了済みTODO件数の取得 → countByFinished

.. code-block:: java

    package todo.domain.repository.todo;

    import java.util.Collection;

    import todo.domain.model.Todo;

    public interface TodoRepository {
        Todo findOne(String todoId);

        Collection<Todo> findAll();

        void create(Todo todo);

        boolean update(Todo todo);

        void delete(Todo todo);

        long countByFinished(boolean finished);
    }

.. note::

    ここでは、\ ``TodoRepository``\ の汎用性を上げるため、「完了済み件数を取得する」メソッド(\ ``long countFinished()``\)ではなく、
    「完了状態がxxである件数を取得する」メソッド(\ ``long countByFinished(boolean)``\)として定義している。
    
    \ ``long countByFinished(boolean)``\の引数として\ ``true``\を渡すと「完了済みの件数」、
    \ ``false``\を渡すと「未完了の件数」が取得できる仕様としている。

|

RepositoryImplの作成(インフラストラクチャ層)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ここでは、説明を単純化するため、\ ``java.util.Map``\ を使ったインメモリ実装のRepositoryImplを作成する。
データベースを使用する場合は、「:ref:`tutorial-todo_infra`」に記載されている内容で読み替えて、RepositoryImplを作成する。

Package Explorer上で右クリック -> New -> Class を選択し、「New Java Class」ダイアログを表示し、

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - 項番
      - 項目
      - 入力値
    * - 1
      - Package
      - ``todo.domain.repository.todo``
    * - 2
      - Name
      - ``TodoRepositoryImpl``
    * - 3
      - Interfaces
      - ``todo.domain.repository.todo.TodoRepository``

を入力して「Finish」する。

作成したクラスは以下のディレクトリに格納される。

.. figure:: ./images/image062.png

作成したクラスにCRUD操作を実装する。

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
        public void create(Todo todo) {
            TODO_MAP.put(todo.getTodoId(), todo);
        }

        @Override
        public boolean update(Todo todo) {
            TODO_MAP.put(todo.getTodoId(), todo);
            return true;
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

.. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 80

   * - 項番
     - 説明
   * - | (1)
     - | Repositoryとしてcomponent-scan対象とするため、クラスレベルに\ ``@Repository``\ アノテーションをつける。

.. note::
 
    本チュートリアルでは、インフラストラクチャ層に属するクラス(RepositoryImpl)をドメイン層のパッケージ(\ ``todo.domain``\)に格納しているが、
    完全に層別にパッケージを分けるのであれば、インフラストラクチャ層のクラスは、\ ``todo.infra``\以下に作成した方が良い。

    ただし、通常のプロジェクトでは、インフラストラクチャ層が変更されることを前提としていない(そのような前提で進めるプロジェクトは、少ない)。
    そこで、作業効率向上のために、ドメイン層のRepositoryインタフェースと同じ階層に、RepositoryImplを作成しても良い。

|

Serviceの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

まず、\ ``TodoService``\ インタフェースを作成する。

Package Explorer上で右クリック -> New -> Interface を選択し、「New Java Interface」ダイアログを表示し、

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - 項番
      - 項目
      - 入力値
    * - 1
      - Package
      - ``todo.domain.service.todo``
    * - 2
      - Name
      - ``TodoService``

を入力して「Finish」する。

作成したインタフェースは以下のディレクトリに格納される。

.. figure:: ./images/image063.png

作成したインタフェースに以下の業務処理を行うメソッドを定義する。

* Todoの全件取得 → findAll
* Todoの新規作成 → create
* Todoの完了 → finish
* Todoの削除 → delete

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

|

次に、\ ``TodoService``\ インタフェースに定義したメソッドを実装する\ ``TodoServiceImpl``\ クラスを作成する。

Package Explorer上で右クリック -> New -> Class を選択し、「New Java Class」ダイアログを表示し、

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - 項番
      - 項目
      - 入力値
    * - 1
      - Package
      - ``todo.domain.service.todo``
    * - 2
      - Name
      - ``TodoServiceImpl``
    * - 3
      - Interfaces
      - ``todo.domain.service.todo.TodoService``

を入力して「Finish」する。

作成したインタフェースは以下のディレクトリに格納される。

.. figure:: ./images/image064.png

.. code-block:: java
    :emphasize-lines: 19, 20, 25-26, 28-29, 32-33, 37-38, 44, 57-58, 61-62, 71, 90

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

            todoRepository.create(todo);
            /* REMOVE THIS LINE IF YOU USE JPA
                todoRepository.save(todo); // 10
               REMOVE THIS LINE IF YOU USE JPA */

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
            todoRepository.update(todo);
            /* REMOVE THIS LINE IF YOU USE JPA
                todoRepository.save(todo); // (11)
               REMOVE THIS LINE IF YOU USE JPA */
            return todo;
        }

        @Override
        public void delete(String todoId) {
            Todo todo = findOne(todoId);
            todoRepository.delete(todo);
        }
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | Serviceとしてcomponent-scanの対象とするため、クラスレベルに\ ``@Service``\ アノテーションをつける。
   * - | (2)
     - | クラスレベルに、\ ``@Transactional``\ アノテーションをつけることで、公開メソッドをすべてトランザクション管理する。
       | アノテーションを付与することで、メソッド開始時にトランザクションを開始、メソッド正常終了時にトランザクションのコミットが行われる。
       | また、途中で非検査例外が発生した場合は、トランザクションがロールバックされる。
       |
       | データベースを使用しない場合は、\ ``@Transactional``\ アノテーションは不要である。
   * - | (3)
     - | \ ``@Inject``\ アノテーションで、\ ``TodoRepository``\ の実装をインジェクションする。
   * - | (4)
     - | 1件取得は、\ ``finish``\ メソッドでも\ ``delete``\ メソッドでも使用するため、メソッドとして用意しておく(interfaceに公開しても良い)。
   * - | (5)
     - | 結果メッセージを格納するクラスとして、共通ライブラリで用意されている\ ``org.terasoluna.gfw.common.message.ResultMessage``\ を用いる。
       | 今回は、エラーメッセージを例外に追加する際に、\ ``ResultMessages.error()``\ でメッセージ種別を指定して、\ ``ResultMessage``\ を追加している。
   * - | (6)
     - | 対象のデータが存在しない場合、共通ライブラリで用意されている\ ``org.terasoluna.gfw.common.exception.ResourceNotFoundException``\ をスローする。
   * - | (7)
     - | 参照のみ行う処理に関しては、\ ``readOnly=true``\ をつける。
       | O/R Mapperによっては、この設定により、参照時のトランザクション制御の最適化が行われる(JPAを使用する場合、効果はない)。
       |
       | データベースを使用しない場合は、\ ``@Transactional``\ アノテーションは不要である。
   * - | (8)
     - | 業務エラーが発生した場合、共通ライブラリで用意されている\ ``org.terasoluna.gfw.common.exception.BusinessException``\ をスローする。
   * - | (9)
     - | 一意性のある値を生成するために、UUIDを使用している。データベースのシーケンスを用いてもよい。
   * - | (10)
     - | Spring Data JPAを使用してデータベースにアクセスする場合は、\ ``create``\ メソッドではなく、\ ``save``\ メソッドを呼び出す。
   * - | (11)
     - | Spring Data JPAを使用してデータベースにアクセスする場合は、\ ``update``\ メソッドではなく、\ ``save``\ メソッドを呼び出す。

.. note::

    本節では、説明を単純化するため、エラーメッセージをハードコードしているが、メンテナンスの観点で本来は好ましくない。
    通常、メッセージは、プロパティファイルに外部化することが推奨される。
    プロパティファイルに外部化する方法は、\ :doc:`../ArchitectureInDetail/PropertyManagement`\ を参照されたい。

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

まずは、todo管理業務にかかわる画面遷移を、制御するControllerを作成する。

Package Explorer上で右クリック -> New -> Class を選択し、「New Java Class」ダイアログを表示し、

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - 項番
      - 項目
      - 入力値
    * - 1
      - Package
      - ``todo.app.todo``
    * - 2
      - Name
      - ``TodoController``

を入力して「Finish」する。

.. note::

    **上位パッケージがドメイン層と異なるので注意すること。**

作成したインタフェースは以下のディレクトリに格納される。

.. figure:: ./images/image065.png

.. code-block:: java
    :emphasize-lines: 6, 7

    package todo.app.todo;

    import org.springframework.stereotype.Controller;
    import org.springframework.web.bind.annotation.RequestMapping;

    @Controller // (1)
    @RequestMapping("todo") // (2)
    public class TodoController {

    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | Controllerとしてcomponent-scanの対象とするため、クラスレベルに、\ ``@Controller``\ アノテーションをつける。
   * - | (2)
     - | \ ``TodoController``\ が扱う画面遷移のパスを、すべて\ ``<contextPath>/todo``\ 配下にするため、クラスレベルに\ ``@RequestMapping(“todo”)``\ を設定する。

|

Show all TODOの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
本チュートリアルで作成する画面では、

* 新規作成フォームの表示
* TODOの全件表示

を行う。

Formの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Formクラス(JavaBean)を作成する。

Package Explorer上で右クリック -> New -> Class を選択し、「New Java Class」ダイアログを表示し、

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - 項番
      - 項目
      - 入力値
    * - 1
      - Package
      - ``todo.app.todo``
    * - 2
      - Name
      - ``TodoForm``
    * - 3
      - Interfaces
      - ``java.io.Serializable``

を入力して「Finish」する。

作成したインタフェースは以下のディレクトリに格納される。

.. figure:: ./images/image066.png

作成したクラスに以下のプロパティを追加する。

* タイトル → todoTitle

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

Controllerの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

一覧画面表示処理を\ ``TodoController``\ に追加する。

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
        @Inject // (1)
        TodoService todoService;

        @ModelAttribute // (2)
        public TodoForm setUpForm() {
            TodoForm form = new TodoForm();
            return form;
        }

        @RequestMapping(value = "list") // (3)
        public String list(Model model) {
            Collection<Todo> todos = todoService.findAll();
            model.addAttribute("todos", todos); // (4)
            return "todo/list"; // (5)
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``TodoService``\ を、DIコンテナによってインジェクションさせるために、\ ``@Inject``\ アノテーションをつける。
       |
       | DIコンテナの管理する\ ``TodoService``\ 型のインスタンス(\ ``TodoServiceImpl``\ のインスタンス)がインジェクションされる。
   * - | (2)
     - | Formを初期化する。
       |
       | \ ``@ModelAttribute``\ アノテーションをつけることで、このメソッドの返り値のformオブジェクトが、\ ``"todoForm"``\ という名前で\ ``Model``\ に追加される。
       | これは、\ ``TodoController``\ の各処理で、\ ``model.addAttribute("todoForm", form)``\ を実装するのと同義である。
   * - | (3)
     - | \ ``/todo/list``\ というパスにリクエストされた際に、一覧画面表示処理用のメソッド(\ ``list``\ メソッド)が実行されるように\ ``@RequestMapping``\ アノテーションを設定する。
       |
       | クラスレベルに\ ``@RequestMapping(“todo”)``\ が設定されているため、ここでは\ ``@RequestMapping("list")``\ のみで良い。
   * - | (4)
     - | \ ``Model``\ にTodoのリストを追加して、Viewに渡す。
   * - | (5)
     - | View名として\ ``"todo/list"``\ を返すと、spring-mvc.xmlに定義した\ ``ViewResolver``\ によって、\ :file:`WEB-INF/views/todo/list.jsp`\がレンダリングされることになる。

JSPの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

JSPを作成し、Controllerから渡されたModelを表示する。

Package Explorer上で右クリック -> New -> File を選択し、「New File」ダイアログを表示し、

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - 項番
      - 項目
      - 入力値
    * - 1
      - Enter or select the parent folder
      - ``todo/src/main/webapp/WEB-INF/views/todo``
    * - 2
      - File name
      - ``list.jsp``

を入力して「Finish」する。

作成したファイルは以下のディレクトリに格納される。

.. figure:: ./images/create-list-jsp.png

まず、以下を表示するために必要なJSPの実装を行う。

* TODOの入力フォーム
* 「Create Todo」ボタン
* TODOの一覧表示エリア

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
                <form:button>Create Todo</form:button>
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

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 新規作成処理用のformを表示する。
       | formを表示するために、\ ``<form:form>``\ タグを使用する。
       | \ ``modelAttribute``\ 属性には、Controllerで\ ``Model``\ に追加したFormの名前を指定する。
       | \ ``action``\ 属性には新規作成処理を実行するためのURL(\ ``<contextPath>/todo/create``\ )を指定する。
       | 新規作成処理は更新系の処理なので、\ ``method``\属性には\ ``POST``\ メソッドを指定する。
       |
       | \ ``action``\ 属性に指定する<contextPath>は、\ ``${pageContext.request.contextPath}``\ で取得することができる。
   * - | (2)
     - | \ ``<form:input>``\ タグでフォームのプロパティをバインドする。
       | \ ``modelAttribute``\ 属性に指定したFormのプロパティ名と、\ ``path``\ 属性の値が一致している必要がある。
   * - | (3)
     - | \ ``<c:forEach>``\ タグを用いて、Todoのリストを全て表示する。
   * - | (4)
     - | 完了かどうか(\ ``finished``\ )で、打ち消し線(\ ``text-decoration: line-through;``\ )を装飾するかどうかを判断する。
   * - | (5)
     - | **文字列値を出力する際は、XSS対策のため、必ずf:h()関数を使用してHTMLエスケープを行うこと。**
       | XSS対策についての詳細は、\ :doc:`../Security/XSS`\ を参照されたい。


|

STSで「todo」プロジェクトを右クリックし、「Run As」→「Run on Server」でWebアプリケーションを起動する。
ブラウザで http://localhost:8080/todo/todo/list にアクセスすると、以下のような画面が表示される。

.. figure:: ./images/image067.png
   :width: 50%

|

Create TODOの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

次に、一覧表示画面から「Create TODO」ボタンを押した後の、新規作成処理を実装する。

Controllerの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

新規作成処理を\ ``TodoController``\ に追加する。

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

        // (1)
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

        @RequestMapping(value = "create", method = RequestMethod.POST) // (2)
        public String create(@Valid TodoForm todoForm, BindingResult bindingResult, // (3)
                Model model, RedirectAttributes attributes) { // (4)

            // (5)
            if (bindingResult.hasErrors()) {
                return list(model);
            }

            // (6)
            Todo todo = beanMapper.map(todoForm, Todo.class);

            try {
                todoService.create(todo);
            } catch (BusinessException e) {
                // (7)
                model.addAttribute(e.getResultMessages());
                return list(model);
            }

            // (8)
            attributes.addFlashAttribute(ResultMessages.success().add(
                    ResultMessage.fromText("Created successfully!")));
            return "redirect:/todo/list";
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | FormオブジェクトをDomainObjectに変換するために、Dozerの\ ``Mapper``\ インタフェースをインジェクションする。
   * - | (2)
     - | \ ``/todo/create``\ というパスに\ ``POST``\ メソッドを使用してリクエストされた際に、新規作成処理用のメソッド(\ ``create``\ メソッド)が実行されるように\ ``@RequestMapping``\ アノテーションを設定する。
   * - | (3)
     - | フォームの入力チェックを行うため、Formの引数に\ ``@Valid``\ アノテーションをつける。入力チェック結果は、その直後の引数\ ``BindingResult``\ に格納される。
   * - | (4)
     - | 正常に作成が完了した後にリダイレクトし、一覧画面を表示する。
       | リダイレクト先への情報を格納するために、引数に\ ``RedirectAttributes``\ を加える。
   * - | (5)
     - | 入力エラーがあった場合、一覧画面に戻る。
       | Todo全件取得を再度行う必要があるので、\ ``list``\ メソッドを再実行する。
   * - | (6)
     - | Dozerの\ ``Mapper``\ インタフェースを用いて、\ ``TodoForm``\ オブジェクトから\ ``Todo``\ オブジェクトを作成する。
       | 変換元と変換先のプロパティ名が同じ場合は、設定不要である。
       | 今回は、\ ``todoTitle``\ プロパティのみ変換するため、Dozerの\ ``Mapper``\ インタフェースを使用するメリットはほとんどない。プロパティの数が多い場合には、非常に便利である。
   * - | (7)
     - | 業務処理を実行して、\ ``BusinessException``\ が発生した場合、結果メッセージを\ ``Model``\ に追加して、一覧画面に戻る。
   * - | (8)
     - | 正常に作成が完了したので、結果メッセージをflashスコープに追加して、一覧画面でリダイレクトする。
       | リダイレクトすることにより、ブラウザを再読み込みして、再び新規登録処理が\ ``POST``\ されることがなくなる。（詳しくは、「:ref:`DoubleSubmitProtectionAboutPRG`」を参照されたい）
       | なお、今回は成功メッセージであるため、\ ``ResultMessages.success()``\ を使用している。


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
     - | \ ``@NotNull``\ アノテーションを使用して必須チェックを有効化する。
   * - | (2)
     - | \ ``@Size``\ アノテーションを使用して文字数チェックを有効化する。

JSPの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

結果メッセージと入力チェックエラーを表示するエリアを追加する。

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
            <!-- (1) -->
            <t:messagesPanel />

            <form:form
               action="${pageContext.request.contextPath}/todo/create"
                method="post" modelAttribute="todoForm">
                <form:input path="todoTitle" />
                <form:errors path="todoTitle" /><!-- (2) -->
                <form:button>Create Todo</form:button>
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
   * - | (1)
     - | \ ``<t:messagesPanel>``\ タグで、結果メッセージを表示する。
   * - | (2)
     - | \ ``<form:errors>``\ タグで、入力エラーがあった場合に表示する。\ ``path``\ 属性の値は、\ ``<form:input>``\ タグと合わせる。

|

フォームに適切な値を入力してsubmitすると、以下のように、成功メッセージが表示される。

.. figure:: ./images/image068.png
   :width: 40%

.. figure:: ./images/image069.png
   :width: 40%


未完了のTODOが5件登録済みの場合は、業務エラーとなり、エラーメッセージが表示される。

.. figure:: ./images/image070.png
   :width: 60%


入力フォームを、空文字にしてsubmitすると、以下のように、エラーメッセージが表示される。

.. figure:: ./images/image071.png
   :width: 65%

メッセージ表示のカスタマイズ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``<t:messagesPanel>``\ を使用した場合、以下のようなHTMLが出力される。

.. code-block:: html

    <div class="alert alert-success"><ul><li>Created successfully!</li></ul></div>

スタイルシート(\ ``list.jsp``\ の\ ``<style>``\ タグ内)に、以下の修正を加えて、結果メッセージの見た目をカスタマイズする。

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

|

メッセージは、以下のように装飾される。

.. figure:: ./images/image072.png
   :width: 40%

.. figure:: ./images/image073.png
   :width: 60%

|

また、\ ``<form:errors>``\ タグの\ ``cssClass``\ 属性で、入力エラーメッセージのclassを指定できる。

JSPを次のように修正し、

.. code-block:: jsp

    <form:errors path="todoTitle" cssClass="text-error" />

スタイルシートに、以下を追加する。

.. code-block:: css

    .text-error {
        color: #c60f13;
    }

入力エラー時のメッセージは、以下のように装飾される。

.. figure:: ./images/image074.png
   :width: 65%

|

Finish TODOの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

一覧画面に「Finish」ボタンを追加し、TODOを完了させるための処理を追加する。

Formの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

完了処理用のFormについても、\ ``TodoForm``\ を使用する。

\ ``TodoForm``\ に\ ``todoId``\ プロパティを追加する必要があるが、単純に追加してしまうと、新規作成処理でも\ ``todoId``\プロパティのチェックが実行されてしまう。
一つのFormクラスを使用して複数のformから送信されるリクエストパラメータをバインドする場合は、\ ``groups``\ 属性を使用して、入力チェッックルールをグループ化する。

Formクラスに以下のプロパティを追加する。

* ID → todoId

.. code-block:: java
    :emphasize-lines: 9-11,13-14,18-20,22-24,27-29,31-33

    package todo.app.todo;

    import java.io.Serializable;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;

    public class TodoForm implements Serializable {
        // (1)
        public static interface TodoCreate {
        };

        public static interface TodoFinish {
        };

        private static final long serialVersionUID = 1L;

        // (2)
        @NotNull(groups = { TodoFinish.class })
        private String todoId;

        // (3)
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


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | 入力チェックルールをグループ化するためのインタフェースを作成する。
       | 入力チェックルールのグループ化ついては、\ :doc:`../ArchitectureInDetail/Validation`\ を参照されたい。
       |
       | ここでは、新規作成処理用のインタフェースとして\ ``TodoCreate``\を、完了処理用のインタフェースとして\ ``TodoFinish``\ を作成している。
   * - | (2)
     - | \ ``todoId``\ は完了処理で使用するプロパティである。
       | そのため、\ ``@NotNull``\ アノテーションの\ ``groups``\ 属性には、完了処理用の入力チェックルールである事を示す\ ``TodoFinish``\ インタフェースを指定する。
   * - | (3)
     - | \ ``todoTitle``\ は新規作成処理で使用するプロパティである。
       | そのため、\ ``@NotNull``\ アノテーションと\ ``@Size``\ アノテーションの\ ``groups``\ 属性には、新規作成処理用の入力チェックルールである事を示す\ ``TodoCreate``\ インタフェースを指定する。

Controllerの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

完了処理を\ ``TodoController``\ に追加する。

グループ化した入力チェックルールを適用するためには、\ **@Valid アノテーションの代わりに、@Validated アノテーションを使用すること**\ に注意する。

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
                @Validated({ Default.class, TodoCreate.class }) TodoForm todoForm, // (1)
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

        @RequestMapping(value = "finish", method = RequestMethod.POST) // (2)
        public String finish(
                @Validated({ Default.class, TodoFinish.class }) TodoForm form, // (3)
                BindingResult bindingResult, Model model,
                RedirectAttributes attributes) {
            // (4)
            if (bindingResult.hasErrors()) {
                return list(model);
            }

            try {
                todoService.finish(form.getTodoId());
            } catch (BusinessException e) {
                // (5)
                model.addAttribute(e.getResultMessages());
                return list(model);
            }

            // (6)
            attributes.addFlashAttribute(ResultMessages.success().add(
                    ResultMessage.fromText("Finished successfully!")));
            return "redirect:/todo/list";
        }
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | グループ化した入力チェックルールを適用するために、\ ``@Valid``\ アノテーションを\ ``@Validated``\ アノテーションに変更する。

       | \ ``value``\ 属性には、適用する入力チェックルールのグループ(グループインタフェース)を指定する。
       | \ ``Default.class``\ は、グループ化されていない入力チェックルールを適用するために用意されているグループインタフェースである。
   * - | (2)
     - | \ ``/todo/finish``\というパスに\ ``POST``\ メソッドを使用してリクエストされた際に、完了処理用のメソッド(\ ``finish``\ メソッド)が実行されるように\ ``@RequestMapping``\ アノテーションを設定する。
   * - | (3)
     - | 適用する入力チェックのグループとして、完了処理用のグループインタフェース(\ ``TodoFinish``\ インタフェース)を指定する。
   * - | (4)
     - | 入力エラーがあった場合、一覧画面に戻る。
   * - | (5)
     - | 業務処理を実行して、\ ``BusinessException``\ が発生した場合は、結果メッセージを\ ``Model``\ に追加して、一覧画面に戻る。
   * - | (6)
     - | 正常に作成が完了した場合は、結果メッセージをflashスコープに追加して、一覧画面でリダイレクトする。

.. note::

    新規作成処理用と完了処理用を別々のFormクラスとして作成しても良い。
    別々のFormクラスにした場合、入力チェックルールをグループ化する必要がないため、入力チェックルールの定義はシンプルになる。

    ただし、処理毎にFormクラスを作成した場合、

    * クラス数が増える
    * プロパティが重複するため入力チェックルールを一元管理できない

    ため、仕様変更が発生した場合に修正コストが高くなる可能性があるという点に注意してほしい。

    また、\ ``@ModelAttribute``\ メソッドを使用して複数のFormを初期化した場合、
    毎回すべてのFormが初期化されるため、不要なインスタンスが生成されることになる。

JSPの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

完了処理用のformを追加する。

.. code-block:: jsp
    :emphasize-lines: 56-66

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
                <form:button>Create Todo</form:button>
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
                                <!-- (1) -->
                                <form:form
                                    action="${pageContext.request.contextPath}/todo/finish"
                                    method="post"
                                    modelAttribute="todoForm"
                                    cssStyle="display: inline-block;">
                                    <!-- (2) -->
                                    <form:hidden path="todoId"
                                        value="${f:h(todo.todoId)}" />
                                    <form:button>Finish</form:button>
                                </form:form>
                            </c:otherwise>
                        </c:choose></li>
                </c:forEach>
            </ul>
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
     - | TODOが未完了の場合は、TODOを完了させるためのリクエストを送信するformを表示する。
       | \ ``action``\ 属性には完了処理を実行するためのURL(\ ``<contextPath>/todo/finish``\ )を指定する。
       | 完了処理は更新系の処理なので、\ ``method``\属性には\ ``POST``\ メソッドを指定する。
   * - | (2)
     - | \ ``<form:hidden>``\ タグを使用して、リクエストパラメータとして\ ``todoId``\ を送信する。
       | \ ``value``\ 属性に値を設定する場合も、 **必ずf:h()関数でHTMLエスケープすること。**

|

Todoを新規作成した後に、「Finish」ボタン押下すると、以下のように打ち消し線が入り、完了したことがわかる。


.. figure:: ./images/image075.png
   :width: 40%


.. figure:: ./images/image076.png
   :width: 40%


Delete TODOの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

一覧表示画面に「Delete」ボタンを追加して、TODOを削除するための処理を追加する。

Formの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

削除処理用のFormについても、\ ``TodoForm``\ を使用する。

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

        // (1)
        public static interface TodoDelete {
        }

        private static final long serialVersionUID = 1L;

        // (2)
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

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 削除処理用の入力チェックルールをグループ化するためのインタフェースとして\ ``TodoDelete``\ を作成する。
   * - | (2)
     - | 削除処理では\ ``todoId``\ プロパティを使用する。
       | そのため、\ ``todoId``\ の\ ``@NotNull``\ アノテーションの\ ``groups``\ 属性には、削除処理用の入力チェックルールである事を示す\ ``TodoDelete``\ インタフェースを指定する。

Controllerの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

削除処理を\ ``TodoController``\ に追加する。完了処理とほぼ同じである。

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

        @RequestMapping(value = "delete", method = RequestMethod.POST) // (1)
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

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - \ ``/todo/delete``\ というパスに\ ``POST``\ メソッドを使用してリクエストされた際に、
       削除処理用のメソッド(\ ``delete``\ メソッド)が実行されるように\ ``@RequestMapping``\ アノテーションを設定する。

JSPの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
削除処理用のformを追加する。

.. code-block:: jsp
    :emphasize-lines: 67-76

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
                <form:button>Create Todo</form:button>
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
                                    <form:button>Finish</form:button>
                                </form:form>
                            </c:otherwise>
                        </c:choose>
                        <!-- (1) -->
                        <form:form
                            action="${pageContext.request.contextPath}/todo/delete"
                            method="post" modelAttribute="todoForm"
                            cssStyle="display: inline-block;">
                            <!-- (2) -->
                            <form:hidden path="todoId"
                                value="${f:h(todo.todoId)}" />
                            <form:button>Delete</form:button>
                        </form:form>
                    </li>
                </c:forEach>
            </ul>
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
     - | 削除処理用のformを表示する。
       | \ ``action``\ 属性には削除処理を実行するためのURL(\ ``<contextPath>/todo/delete``\ )を指定する。
       | 削除処理は更新系の処理なので、\ ``method``\属性には\ ``POST``\ メソッドを指定する。
   * - | (2)
     - | \ ``<form:hidden>``\ タグを使用して、リクエストパラメータとして\ ``todoId``\ を送信する。
       | \ ``value``\ 属性に値を設定する場合も、\ **必ずf:h()関数でHTMLエスケープすること。**\

|

未完了状態のTODOの「Delete」ボタンを押下すると、以下のようにTODOが削除される。

.. figure:: ./images/image077.png
   :width: 40%

.. figure:: ./images/image078.png
   :width: 40%

CSSファイルの使用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

これまでスタイルシートをJSPファイルの中で直接定義していたが、
実際のアプリケーションを開発する場合は、CSSファイルに定義するのが一般的である。

ここでは、スタイルシートをCSSファイルに定義する方法について説明する。

ブランクプロジェクトから提供しているCSSファイル(\ ``src/main/webapp/resources/app/css/styles.css``\ )にスタイルシートの定義を追加する。

.. code-block:: css

    /* ... */

    .strike {
        text-decoration: line-through;
    }

    .alert {
        border: 1px solid;
        margin-bottom: 5px;
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

    .alert ul {
        margin: 15px 0px 15px 0px;
    }

    #todoList li {
        margin-top: 5px;
    }

|

JSPからCSSファイルを読み込む。

.. code-block:: jsp
    :emphasize-lines: 6-7

    <!DOCTYPE html>
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Todo List</title>
    <!-- (1) -->
    <link rel="stylesheet" href="${pageContext.request.contextPath}/resources/app/css/styles.css" type="text/css">
    </head>
    <body>
        <h1>Todo List</h1>

        <div id="todoForm">
            <t:messagesPanel />

            <form:form
                action="${pageContext.request.contextPath}/todo/create"
                method="post" modelAttribute="todoForm">
                <form:input path="todoTitle" />
                <form:errors path="todoTitle" cssClass="text-error" />
                <form:button>Create Todo</form:button>
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
                                    <form:button>Finish</form:button>
                                </form:form>
                            </c:otherwise>
                        </c:choose>
                        <form:form
                            action="${pageContext.request.contextPath}/todo/delete"
                            method="post" modelAttribute="todoForm"
                            cssStyle="display: inline-block;">
                            <form:hidden path="todoId"
                                value="${f:h(todo.todoId)}" />
                            <form:button>Delete</form:button>
                        </form:form>
                    </li>
                </c:forEach>
            </ul>
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
     - | JSPファイルからスタイルシートの定義を削除し、代わりにスタイルシートを定義したCSSファイルを読み込む。

|

CSSファイルを適用すると、以下のようなレイアウトになる。

.. figure:: ./images/list-screen-css.png
    :width: 40%

|

.. _tutorial-todo_infra:

データベースアクセスを伴うインフラストラクチャ層の作成
================================================================================

ここでは、Domainオブジェクトをデータベースに永続化するためのインフラストラクチャ層の実装方法について説明する。

本チュートリアルでは、以下の2つのO/R Mapperを使用したインフラストラクチャ層の実装方法について説明する。

* MyBatis3
* Spring Data JPA

|

.. _Tutorial_Setup_Database:

データベースのセットアップ
--------------------------------------------------------------------------------

まず、データベースのセットアップを行う。

本チュートリアルでは、データベースのセットアップの手間を省くため、H2 Databaseを使用する。

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
 
    INITパラメータに設定しているDDL文をフォーマットすると、以下の様なSQLとなる。
    
     .. code-block:: sql

        create table if not exists todo (
            todo_id varchar(36) primary key,
            todo_title varchar(30),
            finished boolean,
            created_at timestamp
        )

|

.. _using_MyBatis3:

MyBatis3を使用したインフラストラクチャ層の作成
--------------------------------------------------------------------------------

ここでは、MyBatis3を使用してインフラストラクチャ層のRepositoryImplを作成する方法について説明する。

まずは\ :ref:`TutorialCreateMyBatis3BlankProject`\ でプロジェクト作成し直し、\ :ref:`Tutorial_Setup_Database`\ までで作成した\ :file:`src`\ フォルダ以下のうち、
\ **TodoRepositoryImplクラス以外のファイルを新規作成したプロジェクトにコピーすること**\ 。

Spring Data JPAを使用する場合は、本節を読み飛ばして、\ :ref:`using_SpringDataJPA`\ に進んでよい。

TodoRepositoryの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ ``TodoRepository``\ は、O/R Mapperを使用しない場合と同じ方法で作成する。
作成方法は、「:ref:`TutorialTodoCreateRepository`」を参照されたい。

TodoRepositoryImplの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

MyBatis3を使用する場合、RepositoryImplはRepositoryインタフェース(Mapperインタフェース)から自動生成される。
そのため、\ ``TodoRepositoryImpl``\ の作成は不要である。作成した場合は削除すること。

Mapperファイルの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ ``TodoRepository``\ インタフェースのメソッドが呼び出された際に実行するSQLを定義するためのMapperファイルを作成する。

Package Explorer上で右クリック -> New -> File を選択し、「New File」ダイアログを表示し、

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - 項番
      - 項目
      - 入力値
    * - 1
      - Enter or select the parent folder
      - ``todo/src/main/resources/todo/domain/repository/todo``
    * - 2
      - File name
      - ``TodoRepository.xml``

を入力して「Finish」する。

作成したファイルは以下のディレクトリに格納される。

.. figure:: ./images/create-mapper-for-mybatis3.png


\ ``TodoRepository``\ インタフェースに定義したメソッドが呼び出された際に実行するSQLを記述する。

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

    <!-- (1) -->
    <mapper namespace="todo.domain.repository.todo.TodoRepository">

        <!-- (2) -->
        <resultMap id="todoResultMap" type="Todo">
            <id property="todoId" column="todo_id" />
            <result property="todoTitle" column="todo_title" />
            <result property="finished" column="finished" />
            <result property="createdAt" column="created_at" />
        </resultMap>

        <!-- (3) -->
        <select id="findOne" parameterType="String" resultMap="todoResultMap">
        <![CDATA[
            SELECT
                todo_id,
                todo_title,
                finished,
                created_at
            FROM
                todo
            WHERE
                todo_id = #{todoId}
        ]]>
        </select>

        <!-- (4) -->
        <select id="findAll" resultMap="todoResultMap">
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

        <!-- (5) -->
        <insert id="create" parameterType="Todo">
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
                #{todoId},
                #{todoTitle},
                #{finished},
                #{createdAt}
            )
        ]]>
        </insert>

        <!-- (6) -->
        <update id="update" parameterType="Todo">
        <![CDATA[
            UPDATE todo
            SET
                todo_title = #{todoTitle},
                finished = #{finished},
                created_at = #{createdAt}
            WHERE
                todo_id = #{todoId}
        ]]>
        </update>

        <!-- (7) -->
        <delete id="delete" parameterType="Todo">
        <![CDATA[
            DELETE FROM
                todo
            WHERE
                todo_id = #{todoId}
        ]]>
        </delete>

        <!-- (8) -->
        <select id="countByFinished" parameterType="Boolean"
            resultType="Long">
        <![CDATA[
            SELECT
                COUNT(*)
            FROM
                todo
            WHERE
                finished = #{finished}
        ]]>
        </select>

    </mapper>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``mapper``\ 要素の\ ``namespace``\ 属性に、Repositoryインタフェースの完全修飾クラス名(FQCN)を指定する。
   * - | (2)
     - | \ ``<resultMap>``\ 要素に、検索結果(\ ``ResultSet``\ )とJavaBeanのマッピング定義を行う。
       | マッピングファイルの詳細は\ :doc:`../ArchitectureInDetail/DataAccessMyBatis3`\ を参照されたい。
   * - | (3)
     - | \ ``todoId``\ (PK)が一致するレコードを1件取得するSQLを実装する。
       | \ ``<select>``\ 要素の\ ``resultMap``\ 属性には、適用するマッピング定義のIDを指定する。
   * - | (4)
     - | 全レコードを取得するSQLを実装している。
       | \ ``<select>``\ 要素の\ ``resultMap``\ 属性に、適用するマッピング定義のIDを指定する。
       | アプリケーションの要件には記載がないが、最新のTODOが先頭に表示されるようにレコードを並び替えている。
   * - | (5)
     - | 引数に指定されたTodoオブジェクトを挿入するSQLを実装する。
       | \ ``<insert>``\ 要素の\ ``parameterType``\ 属性に、パラメータのクラス名(FQCN又はエイリアス名)を指定する。
   * - | (6)
     - | 引数に指定されたTodoオブジェクトを更新するSQLを実装する。
       | \ ``<update>``\ 要素の\ ``parameterType``\ 属性に、パラメータのクラス名(FQCN又はエイリアス名)を指定する。
   * - | (7)
     - | 引数に指定されたTodoオブジェクトを削除するSQLを実装する。
       | \ ``<delete>``\ 要素の\ ``parameterType``\ 属性に、パラメータのクラス名(FQCN又はエイリアス名)を指定する。
   * - | (8)
     - | 引数に指定された\ ``finished``\ に一致するTodoの件数を取得するSQLを実装する。

|

以上で、MyBatis3を使用したインフラストラクチャ層の作成が完了したので、Service及びアプリケーション層の作成を行う。

Service及びアプリケーション層を作成後にAPサーバーを起動し、Todoの表示を行うと、以下のようなSQLログやトランザクションログが出力される。

.. code-block:: console
   :emphasize-lines: 2-3,6-15,17-19

    date:2015-01-17 14:59:06	thread:tomcat-http--7	X-Track:6a624a51b4f64a528c16c87ad6e9e2ea	level:TRACE	logger:o.t.gfw.web.logging.TraceLoggingInterceptor     	message:[START CONTROLLER] TodoController.list(Model)
    date:2015-01-17 14:59:06	thread:tomcat-http--7	X-Track:6a624a51b4f64a528c16c87ad6e9e2ea	level:DEBUG	logger:o.s.jdbc.datasource.DataSourceTransactionManager	message:Creating new transaction with name [todo.domain.service.todo.TodoServiceImpl.findAll]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT,readOnly; ''
    date:2015-01-17 14:59:06	thread:tomcat-http--7	X-Track:6a624a51b4f64a528c16c87ad6e9e2ea	level:DEBUG	logger:o.s.jdbc.datasource.DataSourceTransactionManager	message:Acquired Connection [net.sf.log4jdbc.ConnectionSpy@20c7885b] for JDBC transaction
    date:2015-01-17 14:59:06	thread:tomcat-http--7	X-Track:6a624a51b4f64a528c16c87ad6e9e2ea	level:DEBUG	logger:t.domain.repository.todo.TodoRepository.findAll 	message:==>  Preparing: SELECT todo_id, todo_title, finished, created_at FROM todo
    date:2015-01-17 14:59:06	thread:tomcat-http--7	X-Track:6a624a51b4f64a528c16c87ad6e9e2ea	level:DEBUG	logger:t.domain.repository.todo.TodoRepository.findAll 	message:==> Parameters:
    date:2015-01-17 14:59:06	thread:tomcat-http--7	X-Track:6a624a51b4f64a528c16c87ad6e9e2ea	level:DEBUG	logger:jdbc.sqltiming                                  	message: sun.reflect.NativeMethodAccessorImpl.invoke0(NativeMethodAccessorImpl.java:-2)
    2. SELECT
                todo_id,
                todo_title,
                finished,
                created_at
            FROM
                todo {executed in 0 msec}
    date:2015-01-17 14:59:06	thread:tomcat-http--7	X-Track:6a624a51b4f64a528c16c87ad6e9e2ea	level:DEBUG	logger:t.domain.repository.todo.TodoRepository.findAll 	message:<==      Total: 0
    date:2015-01-17 14:59:06	thread:tomcat-http--7	X-Track:6a624a51b4f64a528c16c87ad6e9e2ea	level:DEBUG	logger:o.s.jdbc.datasource.DataSourceTransactionManager	message:Initiating transaction commit
    date:2015-01-17 14:59:06	thread:tomcat-http--7	X-Track:6a624a51b4f64a528c16c87ad6e9e2ea	level:DEBUG	logger:o.s.jdbc.datasource.DataSourceTransactionManager	message:Committing JDBC transaction on Connection [net.sf.log4jdbc.ConnectionSpy@20c7885b]
    date:2015-01-17 14:59:06	thread:tomcat-http--7	X-Track:6a624a51b4f64a528c16c87ad6e9e2ea	level:DEBUG	logger:o.s.jdbc.datasource.DataSourceTransactionManager	message:Releasing JDBC Connection [net.sf.log4jdbc.ConnectionSpy@20c7885b] after transaction
    date:2015-01-17 14:59:06	thread:tomcat-http--7	X-Track:6a624a51b4f64a528c16c87ad6e9e2ea	level:DEBUG	logger:o.t.gfw.web.codelist.CodeListInterceptor        	message:locale for I18nCodelist is 'ja_JP'.
    date:2015-01-17 14:59:06	thread:tomcat-http--7	X-Track:6a624a51b4f64a528c16c87ad6e9e2ea	level:TRACE	logger:o.t.gfw.web.logging.TraceLoggingInterceptor     	message:[END CONTROLLER  ] TodoController.list(Model)-> view=todo/list, model={todoForm=todo.app.todo.TodoForm@5e98d549, todos=[], org.springframework.validation.BindingResult.todoForm=org.springframework.validation.BeanPropertyBindingResult: 0 errors}
    date:2015-01-17 14:59:06	thread:tomcat-http--7	X-Track:6a624a51b4f64a528c16c87ad6e9e2ea	level:TRACE	logger:o.t.gfw.web.logging.TraceLoggingInterceptor     	message:[HANDLING TIME   ] TodoController.list(Model)-> 4,324,544 ns

|

.. _using_SpringDataJPA:

Spring Data JPAを使用したインフラストラクチャ層の作成
--------------------------------------------------------------------------------

ここでは、\ `Spring Data JPA <http://www.springsource.org/spring-data/jpa>`_\ を使用してインフラストラクチャ層のRepositoryImplを作成する方法について説明する。

まずは\ :ref:`TutorialCreateJPABlankProject`\ でプロジェクト作成し直し、\ :ref:`Tutorial_Setup_Database`\ までで作成した\ :file:`src`\ フォルダ以下のうち、
\ **TodoRepositoryImplクラス以外のファイルを新規作成したプロジェクトにコピーすること**\ 。

Entityの修正
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

TodoクラスとデータベースのTODOテーブルをマッピングするために、JPAのアノテーションを設定する。

.. code-block:: java
    :emphasize-lines: 6-10,12-14,18-19,26-27

    package todo.domain.model;

    import java.io.Serializable;
    import java.util.Date;

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
        private String todoId;

        private String todoTitle;

        private boolean finished;

        // (3)
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
     - | \ ``java.util.Date``\ 型は、\ ``java.sql.Date``\ , \ ``java.sql.Time``\ , \ ``java.sql.Timestamp``\ のインスタンスを格納できるため、明示的にどの型のインスタンスを設定するか指定する必要がある。
       | \ ``createdAt``\ プロパティには、\ ``Timestamp``\ を指定する。


TodoRepositoryの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Data JPAのRepository機能を使用して\ ``TodoRepository``\ の作成を行う。

Package Explorer上で右クリック -> New -> Interface を選択し、「New Java Interface」ダイアログを表示し、

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 50

    * - 項番
      - 項目
      - 入力値
    * - 1
      - Package
      - ``todo.domain.repository.todo``
    * - 2
      - Name
      - ``TodoRepository``
    * - 3
      - Extended interfaces
      - ``org.springframework.data.jpa.repository.JpaRepository<T, ID>``

を入力して「Finish」する。



.. code-block:: java
    :emphasize-lines: 3-5,9-10,12,13

    package todo.domain.repository.todo;

    import org.springframework.data.jpa.repository.JpaRepository;
    import org.springframework.data.jpa.repository.Query;
    import org.springframework.data.repository.query.Param;

    import todo.domain.model.Todo;

    // (1)
    public interface TodoRepository extends JpaRepository<Todo, String> {

        @Query("SELECT COUNT(t) FROM Todo t WHERE t.finished = :finished") // (2)
        long countByFinished(@Param("finished") boolean finished); // (3)

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``JpaRepository``\のGenericsのパラメータを指定する。
       | 左から順に、Entityのクラス(\ ``Todo``\)、主キーのクラス(\ ``String``\)を指定する。
       | 基本的なCRUD操作(\ ``findOne``\ , \ ``findAll``\ , \ ``save``\ , \ ``delete``\ など)は、\ ``JpaRepository``\ インタフェースに定義済みであるため、\ ``TodoRepository``\ には\ ``countByFinished``\ メソッドのみ定義すればよい。
   * - | (2)
     - | \ ``countByFinished``\ メソッドを呼び出した際に実行するJPQLを、\ ``@Query``\ アノテーションで指定する。
   * - | (3)
     - | (2)で指定したJPQL内のバインド変数に対応するメソッド引数に、\ ``@Param``\ アノテーションを指定する。
       | ここでは、JPQL中の\ ``”:finished”``\ に値を埋め込むために、メソッド引数の\ ``finished``\に\ ``@Param(“finished”)``\ を付けている。


TodoRepositoryImplの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Data JPAを使用する場合、RepositoryImplはRepositoryインタフェースから自動生成される。
そのため、\ ``TodoRepositoryImpl``\ の作成は不要である。作成した場合は削除すること。

|

以上で、Spring Data JPAを使用したインフラストラクチャ層の作成が完了したので、Service及びアプリケーション層の作成を行う。

Service及びアプリケーション層を作成後にAPサーバーを起動し、Todoの表示を行うと、以下のようなSQLログや、トランザクションログが出力される。

.. code-block:: console
   :emphasize-lines: 2-7

    date:2015-01-17 15:45:55	thread:tomcat-http--4	X-Track:5fcebe300ab844f49a1bac35b68184c8	level:TRACE	logger:o.t.gfw.web.logging.TraceLoggingInterceptor     	message:[START CONTROLLER] TodoController.list(Model)
    date:2015-01-17 15:45:55	thread:tomcat-http--4	X-Track:5fcebe300ab844f49a1bac35b68184c8	level:DEBUG	logger:o.h.e.transaction.spi.AbstractTransactionImpl   	message:begin
    date:2015-01-17 15:45:55	thread:tomcat-http--4	X-Track:5fcebe300ab844f49a1bac35b68184c8	level:DEBUG	logger:o.h.e.transaction.internal.jdbc.JdbcTransaction 	message:initial autocommit status: false
    date:2015-01-17 15:45:55	thread:tomcat-http--4	X-Track:5fcebe300ab844f49a1bac35b68184c8	level:DEBUG	logger:jdbc.sqltiming                                  	message: org.hibernate.engine.jdbc.internal.ResultSetReturnImpl.extract(ResultSetReturnImpl.java:82)
    6. /* select generatedAlias0 from Todo as generatedAlias0 */ select todo0_.todo_id as todo_id1_0_, todo0_.created_at as created_2_0_, todo0_.finished as finished3_0_, todo0_.todo_title as todo_tit4_0_ from todo todo0_ {executed in 0 msec}
    date:2015-01-17 15:45:55	thread:tomcat-http--4	X-Track:5fcebe300ab844f49a1bac35b68184c8	level:DEBUG	logger:o.h.e.transaction.spi.AbstractTransactionImpl   	message:committing
    date:2015-01-17 15:45:55	thread:tomcat-http--4	X-Track:5fcebe300ab844f49a1bac35b68184c8	level:DEBUG	logger:o.h.e.transaction.internal.jdbc.JdbcTransaction 	message:committed JDBC Connection
    date:2015-01-17 15:45:55	thread:tomcat-http--4	X-Track:5fcebe300ab844f49a1bac35b68184c8	level:DEBUG	logger:o.t.gfw.web.codelist.CodeListInterceptor        	message:locale for I18nCodelist is 'ja_JP'.
    date:2015-01-17 15:45:55	thread:tomcat-http--4	X-Track:5fcebe300ab844f49a1bac35b68184c8	level:TRACE	logger:o.t.gfw.web.logging.TraceLoggingInterceptor     	message:[END CONTROLLER  ] TodoController.list(Model)-> view=todo/list, model={todoForm=todo.app.todo.TodoForm@38574f7e, todos=[], org.springframework.validation.BindingResult.todoForm=org.springframework.validation.BeanPropertyBindingResult: 0 errors}
    date:2015-01-17 15:45:55	thread:tomcat-http--4	X-Track:5fcebe300ab844f49a1bac35b68184c8	level:TRACE	logger:o.t.gfw.web.logging.TraceLoggingInterceptor     	message:[HANDLING TIME   ] TodoController.list(Model)-> 5,288,781 ns

|

おわりに
================================================================================
このチュートリアルでは、以下の内容を学習した。


* TERASOLUNA Server Framework for Java (5.x)による基本的なアプリケーションの開発方法

* MavenおよびSTS(Eclipse)プロジェクトの構築方法

* TERASOLUNA Server Framework for Java (5.x)のアプリケーションのレイヤ化に従った開発方法

 * POJO(+ Spring)を使用したドメイン層の実装
 * POJO(+ Spring MVC)とJSPタグライブラリを使用したアプリケーション層の実装
 * MyBatis3を使用したインフラストラクチャ層の実装
 * Spring Data JPAを使用したインフラストラクチャ層の実装
 * O/R Mapperを使用しないインフラストラクチャ層の実装

本チュートリアルで作成したTODO管理アプリケーションには、以下の改善点がある。
アプリケーションの修正を学習課題として、ガイドライン中の該当する説明を参照されたい。

* プロパティ(未完了TODOの上限数)を外部化する → :doc:`../ArchitectureInDetail/PropertyManagement`
* メッセージを外部化する → :doc:`../ArchitectureInDetail/MessageManagement`
* ページング処理を追加する → :doc:`../ArchitectureInDetail/Pagination`
* 例外ハンドリングを加える → :doc:`../ArchitectureInDetail/ExceptionHandling`
* 二重送信を防止する(トランザクショントークンチェックを追加する) → :doc:`../ArchitectureInDetail/DoubleSubmitProtection`
* システム日時の取得元を変更する → :doc:`../ArchitectureInDetail/SystemDate`

|

Appendix
================================================================================

.. _TutorialTodoAppendixExpoundConfigurations:

設定ファイルの解説
--------------------------------------------------------------------------------

アプリケーションを動かすためにどのような設定が必要なのかを理解するために、設定ファイルの解説を行う。
ここでは、チュートリアルで作成するTodoアプリケーションで使用しない設定については、解説を割愛している箇所がある。

web.xml
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ :file:`web.xml`\ には、WebアプリケーションとしてTodoアプリをデプロイするための設定を行う。

作成したブランクプロジェクトの\ :file:`src/main/webapp/WEB-INF/web.xml`\ は、以下のような設定となっている。

.. code-block:: xml
    :emphasize-lines: 2, 6, 22, 78, 95, 106, 120


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

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

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
       | DispatcherServletの中で使用する\ ``ApplicationContext``\を、(2)で作成した\ ``ApplicationContext``\の子として作成する。
       | (2)で作成した\ ``ApplicationContext``\を親にすることで、(2)で読み込まれたコンポーネントも使用することができる。
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

インクルードJSP
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
インクルードJSPには、全てのJSPに適用するJSPの設定や、タグライブラリの設定を行う。

作成したブランクプロジェクトの\ :file:`src/main/webapp/WEB-INF/views/common/include.jsp`\ は、以下のような設定となっている。

.. code-block:: jsp
    :emphasize-lines: 1, 3, 6, 9, 11

    <!-- (1) -->
    <%@ page session="false"%>
    <!-- (2) -->
    <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
    <%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt"%>
    <!-- (3) -->
    <%@ taglib uri="http://www.springframework.org/tags" prefix="spring"%>
    <%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>
    <!-- (4) -->
    <%@ taglib uri="http://www.springframework.org/security/tags" prefix="sec"%>
    <!-- (5) -->
    <%@ taglib uri="http://terasoluna.org/functions" prefix="f"%>
    <%@ taglib uri="http://terasoluna.org/tags" prefix="t"%>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

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

Bean定義ファイル
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

作成したブランクプロジェクトには、以下のBean定義ファイルとプロパティファイルが作成される。

* :file:`src/main/resources/META-INF/spring/applicationContext.xml`
* :file:`src/main/resources/META-INF/spring/todo-domain.xml`
* :file:`src/main/resources/META-INF/spring/todo-infra.xml`
* :file:`src/main/resources/META-INF/spring/todo-infra.properties`
* :file:`src/main/resources/META-INF/spring/todo-env.xml`
* :file:`src/main/resources/META-INF/spring/spring-mvc.xml`
* :file:`src/main/resources/META-INF/spring/spring-security.xml`

.. note::

    O/R Mapperに依存しないブランクプロジェクトを作成した場合は、\ ``todo-infra.properties``\ と\ ``todo-env.xml``\ は作成されない。

.. note::

    本ガイドラインでは、Bean定義ファイルを役割(層)ごとにファイルを分割することを推奨している。

    これは、どこに何が定義されているか想像しやすく、メンテナンス性が向上するからである。
    今回のチュートリアルのような小さなアプリケーションでは効果はないが、アプリケーションの規模が大きくなるにつれ、効果が大きくなる。

|

applicationContext.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ :file:`applicationContext.xml`\ には、Todoアプリ全体に関わる設定を行う。

| 作成したブランクプロジェクトの\ :file:`src/main/resources/META-INF/spring/applicationContext.xml`\  は、以下のような設定となっている。
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
       | \ ``src/main/resources/META-INF/spring``\ 直下の任意のプロパティファイルを読み込む。
       | この設定により、プロパティファイルの値をBean定義ファイル内で\ ``${propertyName}``\ 形式で埋め込んだり、Javaクラスに\ ``@Value("${propertyName}")``\ でインジェクションすることができる。
   * - | (3)
     - | Bean変換用ライブラリDozerのMapperを定義する。
       | (マッピングファイルに関しては `Dozerマニュアル <http://dozer.sourceforge.net/documentation/mappings.html>`_ を参照されたい。)

.. tip::

    エディタの「Configure Namespaces」タブにて、以下のようにチェックを入れると、
    チェックしたXMLスキーマが有効になり、XML編集時にCtrl+Spaceを使用して入力を補完することができる。

    「Namespace Versions」にはバージョンなしのxsdファイルを選択することを推奨する。
    バージョンなしのxsdファイルを選択することで、常にjarに含まれる最新のxsdが使用されるため、
    Springのバージョンアップを意識する必要がなくなる。

     .. figure:: ./images/image021.jpg
        :width: 90%

     .. figure:: ./images/image023.png
        :width: 60%

|

todo-domain.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ :file:`todo-domain.xml`\ には、Todoアプリのドメイン層に関わる設定を行う。

| 作成したブランクプロジェクトの\ :file:`src/main/resources/META-INF/spring/todo-domain.xml`\ は、以下のような設定となっている。
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

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

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
    \ ``<tx:annotation-driven>``\タグが設定されている。

     .. code-block:: xml
        :emphasize-lines: 9-10, 12-13

         <tx:annotation-driven />

|

todo-infra.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ :file:`todo-infra.xml`\ には、Todoアプリのインフラストラクチャ層に関わる設定を行う。

作成したブランクプロジェクトの\ :file:`src/main/resources/META-INF/spring/todo-infra.xml`\ は、
以下のような設定となっている。

\ :file:`todo-infra.xml`\は、インフラストラクチャ層によって設定が大きく異なるため、
ブランクプロジェクト毎に説明を行う。
作成したブランクプロジェクト以外の説明は読み飛ばしてもよい。


O/R Mapperに依存しないブランクプロジェクトを作成した場合のtodo-infra.xml
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

O/R Mapperに依存しないブランクプロジェクトを作成した場合、以下のように空定義のファイルが作成される。

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">


    </beans>

MyBatis3用のブランクプロジェクトを作成した場合のtodo-infra.xml
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

MyBatis3用のブランクプロジェクトを作成した場合、以下のような設定となっている。

.. code-block:: xml
   :emphasize-lines: 11-12, 14-16, 17-18, 19-20, 23-25

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:mybatis="http://mybatis.org/schema/mybatis-spring"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://mybatis.org/schema/mybatis-spring
            http://mybatis.org/schema/mybatis-spring.xsd">

         <!-- (1) -->
        <import resource="classpath:/META-INF/spring/todo-env.xml" />

         <!-- (2) -->
        <!-- define the SqlSessionFactory -->
        <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
             <!-- (3) -->
            <property name="dataSource" ref="dataSource" />
             <!-- (4) -->
            <property name="configLocation" value="classpath:/META-INF/mybatis/mybatis-config.xml" />
        </bean>

         <!-- (5) -->
        <!-- scan for Mappers -->
        <mybatis:scan base-package="todo.domain.repository" />

    </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 環境依存するコンポーネント(データソースやトランザクションマネージャなど)を定義するBean定義ファイルをimportする。
   * - | (2)
     - | \ ``SqlSessionFactory``\ を生成するためのコンポーネントとして、\ ``SqlSessionFactoryBean``\ をbean定義する。
   * - | (3)
     - | \ ``dataSource``\ プロパティに、設定済みのデータソースのbeanを指定する。
       |
       | MyBatis3の処理の中でSQLを発行する際は、ここで指定したデータソースからコネクションが取得される。
   * - | (4)
     - | \ ``configLocation``\ プロパティに、MyBatis設定ファイルのパスを指定する。
       |
       | ここで指定したファイルは\ ``SqlSessionFactory``\ を生成する時に読み込まれる。
   * - | (5)
     - | Mapperインタフェースをスキャンするために\ ``<mybatis:scan>``\ を定義し、\ ``base-package``\ 属性には、
       | Mapperインタフェースが格納されている基底パッケージを指定する。
       |
       | 指定されたパッケージ配下に格納されている Mapperインタフェースがスキャンされ、
       | スレッドセーフなMapperオブジェクト(MapperインタフェースのProxyオブジェクト)が自動的に生成される。

.. note::

    \ :file:`mybatis-config.xml`\ は、MyBatis3自体の動作設定を行う設定ファイルである。

    ブランクプロジェクトでは、デフォルトで以下の設定が行われている。

     .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>
        <!DOCTYPE configuration
          PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-config.dtd">
        <configuration>

            <!-- See http://mybatis.github.io/mybatis-3/configuration.html#settings -->
            <settings>
                <setting name="mapUnderscoreToCamelCase" value="true" />
                <setting name="lazyLoadingEnabled" value="true" />
                <setting name="aggressiveLazyLoading" value="false" />
        <!--
                <setting name="defaultExecutorType" value="REUSE" />
                <setting name="jdbcTypeForNull" value="NULL" />
                <setting name="proxyFactory" value="JAVASSIST" />
                <setting name="localCacheScope" value="STATEMENT" />
        -->
            </settings>

            <typeAliases>
                <package name="todo.domain.model" />
                <package name="todo.domain.repository" />
        <!--
                <package name="todo.infra.mybatis.typehandler" />
        -->
            </typeAliases>

            <typeHandlers>
        <!--
                <package name="todo.infra.mybatis.typehandler" />
        -->
            </typeHandlers>

        </configuration>

JPA用のブランクプロジェクトを作成した場合のtodo-infra.xml
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

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 環境依存するコンポーネント(データソースやトランザクションマネージャなど)を定義するBean定義ファイルをimportする。
   * - | (2)
     - | Spring Data JPAを使用して、Repositoryインタフェースから実装クラスを自動生成する。
       | \ ``<jpa:repository>``\ タグの\ ``base-package``\ 属性に、Repositoryを格納するパッケージを指定する。
   * - | (3)
     - | JPAの実装ベンダの設定を行う。
       | JPA実装として、Hibernateを使うため、\ ``HibernateJpaVendorAdapter``\ を定義している。
   * - | (4)
     - | \ ``EntityManager``\ の定義を行う。
   * - | (5)
     - | JPAのエンティティとして扱うクラスが格納されているパッケージ名を指定する。
   * - | (6)
     - | Hibernateに関する詳細な設定を行う。

|

todo-infra.properties
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ :file:`todo-infra.properties`\ には、Todoアプリのインフラストラクチャ層の環境依存値の設定を行う。

O/R Mapperに依存しないブランクプロジェクトを作成した際は、\ :file:`todo-infra.properties`\ は作成されない。

作成したブランクプロジェクトの\ :file:`src/main/resources/META-INF/spring/todo-infra.properties`\ は、
以下のような設定となっている。

.. code-block:: properties
    :emphasize-lines: 1, 7

    # (1)
    database=H2
    database.url=jdbc:h2:mem:todo;DB_CLOSE_DELAY=-1;
    database.username=sa
    database.password=
    database.driverClassName=org.h2.Driver
    # (2)
    # connection pool
    cp.maxActive=96
    cp.maxIdle=16
    cp.minIdle=0
    cp.maxWait=60000

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


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

todo-env.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ :file:`todo-env.xml`\ には、デプロイする環境によって設定が異なるコンポーネントの設定を行う。

作成したブランクプロジェクトの\ ``src/main/resources/META-INF/spring/todo-env.xml``\ は、以下のような設定となっている。

ここでは、MyBatis3用のブランクプロジェクトに格納されるファイルを例に説明する。
なお、データベースにアクセスしないブランクプロジェクトを作成した際は、\ :file:`todo-env.xml`\ は作成されない。

.. code-block:: xml
    :emphasize-lines: 8, 22, 39

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

        <bean id="dateFactory" class="org.terasoluna.gfw.common.date.jodatime.DefaultJodaTimeDateFactory" />

        <!-- (1) -->
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

        <!-- (2) -->
        <bean id="dataSource" class="net.sf.log4jdbc.Log4jdbcProxyDataSource">
            <constructor-arg index="0" ref="realDataSource" />
        </bean>

        <!--  REMOVE THIS LINE IF YOU USE JPA
        <bean id="transactionManager"
            class="org.springframework.orm.jpa.JpaTransactionManager">
            <property name="entityManagerFactory" ref="entityManagerFactory" />
        </bean>
              REMOVE THIS LINE IF YOU USE JPA  -->
        <!--  REMOVE THIS LINE IF YOU USE MyBatis2
        <bean id="transactionManager"
            class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <property name="dataSource" ref="dataSource" />
        </bean>
              REMOVE THIS LINE IF YOU USE MyBatis2  -->
        <!-- (3) -->
        <bean id="transactionManager"
            class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <property name="dataSource" ref="dataSource" />
        </bean>
    </beans>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 実データソースの設定。
   * - | (2)
     - | データソースの設定。
       | JDBC関連のログを出力する機能をもったデータソースを指定している。
       | \ ``net.sf.log4jdbc.Log4jdbcProxyDataSource``\ を使用すると、SQLなどのJDBC関連のログを出力できるため、デバッグに役立つ情報を出力することができる。
   * - | (3)
     - | トランザクションマネージャの設定。
       | id属性には、\ ``transactionManager``\ を指定する。
       | 別の名前を指定する場合は、\ ``<tx:annotation-driven>``\ タグにもトランザクションマネージャ名を指定する必要がある。
       |
       | ブランクプロジェクトでは、JDBCのAPIを使用してトランザクションを制御するクラス(\ ``org.springframework.jdbc.datasource.DataSourceTransactionManager``\)が設定されている。

.. note::

    JPA用のブランクプロジェクトを作成した場合は、トランザクションマネージャには、
    JPAのAPIを使用してトランザクションを制御するクラス(\ ``org.springframework.orm.jpa.JpaTransactionManager``\)が設定されている。

     .. code-block:: xml

        <bean id="transactionManager"
            class="org.springframework.orm.jpa.JpaTransactionManager">
            <property name="entityManagerFactory" ref="entityManagerFactory" />
        </bean>

|

spring-mvc.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ :file:`spring-mvc.xml`\ には、Spring MVCに関する定義を行う。

| 作成したブランクプロジェクトの\ :file:`src/main/resources/META-INF/spring/spring-mvc.xml`\ は、以下のような設定となっている。
| なお、チュートリアルで使用しないコンポーネントについての説明は割愛する。

.. code-block:: xml
    :emphasize-lines: 12, 16, 28, 31, 37, 71

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
                <bean
                    class="org.springframework.security.web.bind.support.AuthenticationPrincipalArgumentResolver" />
            </mvc:argument-resolvers>
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
        <mvc:view-resolvers>
            <mvc:jsp prefix="/WEB-INF/views/" />
        </mvc:view-resolvers>

        <bean id="requestDataValueProcessor"
            class="org.terasoluna.gfw.web.mvc.support.CompositeRequestDataValueProcessor">
            <constructor-arg>
                <util:list>
                    <bean class="org.springframework.security.web.servlet.support.csrf.CsrfRequestDataValueProcessor" />
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

    </beans>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | プロパティファイルの読み込み設定を行う。

       | src/main/resources/META-INF/spring直下の任意のプロパティファイルを読み込む。
       | この設定により、プロパティファイルの値をBean定義ファイル内で\ ``${propertyName}``\ 形式で埋め込んだり、Javaクラスに\ ``@Value("${propertyName}")``\ でインジェクションすることができる。
   * - | (2)
     - | Spring MVCのアノテーションベースのデフォルト設定を行う。
   * - | (3)
     - | アプリケーション層のクラスを管理するtodo.appパッケージ配下をcomponent-scan対象とする。
   * - | (4)
     - | 静的リソース(css, images, jsなど)アクセスのための設定を行う。

       | \ ``mapping``\ 属性にURLのパスを、\ ``location``\ 属性に物理的なパスの設定を行う。
       | この設定の場合\ ``<contextPath>/resources/app/css/styles.css``\ に対してリクエストが来た場合、\ ``WEB-INF/resources/app/css/styles.css``\ を探し、見つからなければクラスパス上(\ ``src/main/resources``\ やjar内)の\ ``META-INF/resources/app/css/styles.css``\ を探す。
       | どこにも\ ``styles.css``\ が格納されていない場合は、404エラーを返す。

       | ここでは\ ``cache-period``\ 属性で静的リソースのキャッシュ時間(3600秒=60分)も設定している。
       | \ ``cache-period="3600"``\ と設定しても良いが、60分であることを明示するために `SpEL <http://docs.spring.io/spring/docs/4.1.7.RELEASE/spring-framework-reference/html/expressions.html#expressions-beandef-xml-based>`_ を使用して \ ``cache-period="#{60 * 60}"``\  と書く方が分かりやすい。
   * - | (5)
     - | コントローラ処理のTraceログを出力するインターセプタを設定する。
       | \ ``/resources``\ 配下を除く任意のパスに適用されるように設定する。
   * - | (6)
     - | \ ``ViewResolver``\ の設定を行う。
       | この設定により、例えばコントローラからview名として\ ``"hello"``\が返却された場合には\ ``/WEB-INF/views/hello.jsp``\ が実行される。

       .. tip::

           \ ``<mvc:view-resolvers>``\ 要素はSpring Framework 4.1から追加されたXML要素である。
           \ ``<mvc:view-resolvers>``\ 要素を使用すると、\ ``ViewResolver``\ をシンプルに定義することが出来る。

           従来通り\ ``<bean>``\ 要素を使用した場合の定義例を以下に示す。

            .. code-block:: xml

               <bean id="viewResolver"
                   class="org.springframework.web.servlet.view.InternalResourceViewResolver">
                   <property name="prefix" value="/WEB-INF/views/" />
                   <property name="suffix" value=".jsp" />
               </bean>

.. note::

    JPA用のブランクプロジェクトを作成した場合は、\ ``<mvc:interceptors>``\ の定義として、
    \ ``OpenEntityManagerInViewInterceptor``\ の定義が有効な状態となっている。

     .. code-block:: xml

        <mvc:interceptor>
            <mvc:mapping path="/**" />
            <mvc:exclude-mapping path="/resources/**" />
            <mvc:exclude-mapping path="/**/*.html" />
            <bean
                class="org.springframework.orm.jpa.support.OpenEntityManagerInViewInterceptor" />
        </mvc:interceptor>

    \ ``OpenEntityManagerInViewInterceptor``\ は、\ ``EntityManager``\ のライフサイクルの開始と終了を行う\ ``Interceptor``\ である。
    この設定を追加することで、アプリケーション層(Controllerや、Viewクラス)でのLazy Loadが、サポートされる。

|

spring-security.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ :file:`spring-security.xml`\ には、Spring Securityに関する定義を行う。

| 作成したブランクプロジェクトの\ :file:`src/main/resources/META-INF/spring/spring-security.xml`\ は、以下のような設定となっている。
| なお、本チュートリアルではSpring Securityの設定ファイルの説明は割愛する。Spring Securityの設定ファイルについては、「:doc:`../Security/Tutorial`」を参照されたい。

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:sec="http://www.springframework.org/schema/security"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

        <sec:http pattern="/resources/**" security="none"/>
        <sec:http auto-config="true" use-expressions="true">
            <sec:headers>
                <sec:cache-control />
                <sec:content-type-options />
                <sec:hsts />
                <sec:frame-options />
                <sec:xss-protection />
            </sec:headers>
            <sec:csrf />
            <sec:access-denied-handler ref="accessDeniedHandler"/>
            <sec:custom-filter ref="userIdMDCPutFilter" after="ANONYMOUS_FILTER"/>
            <sec:session-management />
        </sec:http>

        <sec:authentication-manager></sec:authentication-manager>

        <!-- Change View for CSRF or AccessDenied -->
        <bean id="accessDeniedHandler"
            class="org.springframework.security.web.access.DelegatingAccessDeniedHandler">
            <constructor-arg index="0">
                <map>
                    <entry
                        key="org.springframework.security.web.csrf.InvalidCsrfTokenException">
                        <bean
                            class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
                            <property name="errorPage"
                                value="/WEB-INF/views/common/error/invalidCsrfTokenError.jsp" />
                        </bean>
                    </entry>
                    <entry
                        key="org.springframework.security.web.csrf.MissingCsrfTokenException">
                        <bean
                            class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
                            <property name="errorPage"
                                value="/WEB-INF/views/common/error/missingCsrfTokenError.jsp" />
                        </bean>
                    </entry>
                </map>
            </constructor-arg>
            <constructor-arg index="1">
                <bean
                    class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
                    <property name="errorPage"
                        value="/WEB-INF/views/common/error/accessDeniedError.jsp" />
                </bean>
            </constructor-arg>
        </bean>

        <!-- Put UserID into MDC -->
        <bean id="userIdMDCPutFilter" class="org.terasoluna.gfw.security.web.logging.UserIdMDCPutFilter">
        </bean>

    </beans>

|

logback.xml
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ :file:`logback.xml`\ には、ログ出力に関する定義を行う。

| 作成したブランクプロジェクトの\ :file:`src/main/resources/logback.xml`\ は、以下のような設定となっている。
| なお、チュートリアルで使用しないログ設定についての説明は割愛する。

.. code-block:: xml
    :emphasize-lines: 4, 36, 45

    <?xml version="1.0" encoding="UTF-8"?>
    <configuration>

        <!-- (1) -->
        <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern><![CDATA[date:%d{yyyy-MM-dd HH:mm:ss}\tthread:%thread\tX-Track:%X{X-Track}\tlevel:%-5level\tlogger:%-48logger{48}\tmessage:%msg%n]]></pattern>
            </encoder>
        </appender>

        <appender name="APPLICATION_LOG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <file>${app.log.dir:-log}/todo-application.log</file>
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>${app.log.dir:-log}/todo-application-%d{yyyyMMdd}.log</fileNamePattern>
                <maxHistory>7</maxHistory>
            </rollingPolicy>
            <encoder>
                <charset>UTF-8</charset>
                <pattern><![CDATA[date:%d{yyyy-MM-dd HH:mm:ss}\tthread:%thread\tX-Track:%X{X-Track}\tlevel:%-5level\tlogger:%-48logger{48}\tmessage:%msg%n]]></pattern>
            </encoder>
        </appender>

        <appender name="MONITORING_LOG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <file>${app.log.dir:-log}/todo-monitoring.log</file>
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>${app.log.dir:-log}/todo-monitoring-%d{yyyyMMdd}.log</fileNamePattern>
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
        <!--  REMOVE THIS LINE IF YOU USE MyBatis3
        <logger name="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <level value="debug" />
        </logger>
              REMOVE THIS LINE IF YOU USE MyBatis3  -->

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
     - | spring-mvc.xmlに設定した\ ``TraceLoggingInterceptor``\ に出力されるようにtraceレベルで設定。


.. note::

    O/R Mapperを使用するブランクプロジェクトを作成した場合は、トランザクション制御関連のログを出力するロガーが有効な状態となっている。

    * JPA用のブランクプロジェクト

     .. code-block:: xml

        <logger name="org.hibernate.engine.transaction">
            <level value="debug" />
        </logger>

    * MyBatis3用のブランクプロジェクト

     .. code-block:: xml

        <logger name="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <level value="debug" />
        </logger>

.. raw:: latex

   \newpage

