RESTful Web Service
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

.. _RESTOverview:

Overview
--------------------------------------------------------------------------------
本節では、RESTful Web Serviceの基本的な概念とSpring MVCを使った開発について説明する。

RESTful Web Serviceのアーキテクチャ、設計、実装に対する具体的な説明については、

* | 「:ref:`RESTAboutResourceOrientedArchitecture`」
  | RESTful Web Serviceの基本的なアーキテクチャについて説明している。

* | 「:ref:`RESTHowToDesign`」
  | RESTful Web Serviceの設計を行う際に考慮すべき点などを説明している。

* | 「:ref:`RESTHowToUse`」
  | RESTful Web Serviceのアプリケーション構成やAPIの実装方法について説明している。

を参照されたい。

|

.. _RESTOverviewAboutRESTfulWebService:

RESTful Web Serviceとは
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| まずRESTとは、「\ **RE**\presentational \ **S**\tate \ **T**\ransfer」の略であり、
| クライアントとサーバ間でデータをやりとりするアプリケーションを構築するための\ **アーキテクチャスタイル**\の一つである。
| RESTのアーキテクチャスタイルには、いくつかの重要な原則があり、これらの原則に従っているもの(システムなど)は \ **RESTful**\ と表現される。
| つまり、「RESTful Web Service」とは、RESTの原則に従って構築されているWeb Serviceという事になる。

| \ **RESTful Web Serviceにおいて最も重要なのは、「リソース」という概念である。**\
| RESTful Web Serviceでは、データベースなどで管理している情報の中からクライアントに提供すべき情報を「リソース」として抽出し、抽出した「リソース」に対するCRUD操作をHTTPメソッド(POST/GET/PUT/DELETE)を使ってクライアントに提供することになる。
| 「リソース」に対するCRUD操作は「REST API」や「RESTful API」と呼ばれる事があり、本ガイドラインでは「REST API」と呼ぶ。
| また、クライアントとサーバ間でリソースをやりとりする際の電文形式には、電文の視認性、及びデータ構造の表現性が高いJSONやXMLを使用するのが一般的である。

| RESTful Web Serviceを利用するアプリケーションのシステム構成は、主に以下の２パターンとなる。
| クライアントとサーバ間で「リソース」をやりとりするための具体的なアーキテクチャについては、「:ref:`RESTAboutResourceOrientedArchitecture`」で説明する。

 .. figure:: ./images_REST/RESTExampleSystemConstitution.png
   :alt: Constitution of RESTful Web Service
   :width: 100%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ユーザインタフェースを持つクライアントアプリケーションとRESTful Web Serviceの間で、直接リソースのやりとりを行う。
        | このパターンは、要件や仕様の変更頻度が多いユーザインタフェースに依存するロジックと、より普遍的で変更頻度が少ないデータモデルに対するロジックを分離する際に採用される構成である。
    * - | (2)
      - | ユーザインタフェースを持つクライアントアプリケーションと直接リソースをやり取りするのではなく、システム間でリソースをやりとりを行う。
        | このパターンは、各システムで管理しているビジネスデータを一元管理するようなシステムを構築する際に採用される構成である。


|

.. _RESTOverviewAboutRESTfulWebServiceDevelopment:

RESTful Web Serviceの開発について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
TERASOLUNA Server Framework for Java (5.x)では、Spring MVCの機能を利用してRESTful Web Serviceの開発を行う。

| Spring MVCでは、RESTful Web Serviceを開発する上で必要となる共通的な機能がデフォルトで組み込まれている。
| そのため、特別な設定の追加や実装を行うことなく、RESTful Web Serviceの開発を開始する事ができる。

| Spring MVCにデフォルトで組み込まれている主な共通機能は以下の通りである。
| これらの機能は、REST APIを提供するControllerのメソッドにアノテーションを指定だけで有効にする事ができる。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 機能概要
    * - | (1)
      - | リクエストBODYに設定されているJSONやXML形式の電文をResourceオブジェクト(JavaBean)に変換し、Controllerクラスのメソッド(REST API)に引き渡す機能
    * - | (2)
      - | 電文から変換されたResourceオブジェクト(JavaBean)に格納された値に対して入力チェックを実行する機能
    * - | (3)
      - | Controllerクラスのメソッド(REST API)から返却したResourceオブジェクト(JavaBean)を、JSONやXML形式に変換しレスポンスBODYに設定する機能

 .. note:: **例外ハンドリングについて**

    例外ハンドリングについては、Spring MVCから汎用的な機能の提供がないため、プロジェクト毎に実装が必要となる。
    例外ハンドリングの詳細については、「:ref:`RESTHowToUseExceptionHandling`」を参照されたい。

|

| Spring MVCの機能を利用してRESTful Web Serviceを開発した場合、アプリケーションは以下のような構成となり、そのうち実装が必要なのは、赤枠の部分となる。

 .. figure:: ./images_REST/RESTOverviewApplicationConstitutionOnSpringMVC.png
   :alt: Application constitution of RESTful Web Service on Spring MVC
   :width: 100%

 .. raw:: latex

    \newpage

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 70
    :class: longtable

    * - 項番
      - 処理レイヤ
      - 説明
    * - | (1)
      - | Spring MVC
        | (Framework)
      - | Spring MVCは、クライアントからのリクエストを受け取り、呼び出すREST API(Controllerのハンドラメソッド)を決定する。
    * - | (2)
      - | 
        | 
      - | Spring MVCは、\ ``HttpMessageConverter``\を使用して、リクエストBODYに指定されているJSON形式の電文をResourceオブジェクトに変換する。
    * - | (3)
      - | 
        | 
      - | Spring MVCは、\ ``Validator``\を使用して、Resourceオブジェクトに格納されて値に対して入力チェックを行う。
    * - | (4)
      - | 
        | 
      - | Spring MVCは、REST APIを呼び出す。
        | その際に、JSONから変換した入力チェック済みのResourceオブジェクトがREST APIに引き渡される。
    * - | (5)
      - | REST API
      - | REST APIは、Serviceのメソッドを呼び出し、EntityなどのDomainObjectに対する処理を行う。
    * - | (6)
      - | 
      - | Serviceのメソッドは、Repositoryのメソッドを呼び出し、EntityなどのDomainObjectのCRUD処理を行う。
    * - | (7)
      - | Spring MVC
        | (Framework)
      - | Spring MVCは、\ ``HttpMessageConverter``\を使用して、REST APIから返却されたResourceオブジェクトをJSON形式の電文に変換する。
    * - | (8)
      - | 
        | 
      - | Spring MVCは、JSON形式の電文をレスポンスBODYに設定し、クライアントにレスポンスする。

 .. raw:: latex

    \newpage

|

RESTful Web Serviceのモジュールの構成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Spring MVCから提供されている機能を使うことにより、RESTful Web Service固有の処理の多くをSpring MVCに任せることが出来る。
| そのため、開発するモジュールの構成は、HTMLを応答する従来型のWebアプリケーションの開発とほとんど同じ構成となる。
| 以下に、モジュールの構成要素について説明する。

 .. figure:: ./images_REST/RESTModuleConstitution.png
   :alt: Constitution of Modules
   :width: 100%

* **アプリケーション層のモジュール**

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 20 70
    
        * - 項番
          - モジュール名
          - 説明
        * - | (1)
          - | Controllerクラス
          - | REST APIを提供するクラス。
            | Controllerクラスはリソース単位に作成し、リソース毎のREST APIのエンドポイント(URI)の指定を行う。
            | リソースに対するCRUD処理は、ドメイン層のServiceに委譲する事で実現する。
        * - | (2)
          - | Resourceクラス
          - | REST APIの入出力となるJSON(またはXML)を表現するJava Bean。
            | このクラスには、Bean Validationのアノテーションの指定や、JSONやXMLのフォーマットを制御するためのアノテーションの指定を行う。
        * - | (3)
          - | Validatorクラス
            | (Optional)
          - | 入力値の相関チェックを実装するクラス。
            | 入力値の相関チェックが不要な場合は、本クラスを作成する必要はないため、オプションの扱いとしている。
            | 入力値の相関チェックについては、「:doc:`../WebApplicationDetail/Validation`」を参照されたい。
        * - | (4)
          - | Helperクラス
            | (Optional)
          - | Controllerで行う処理を補助するための処理を実装するクラス。
            | 本クラスは、Controllerの処理をシンプルに保つことを目的として作成するクラスである。
            | 具体的には、ResourceオブジェクトとDomainObjectのモデル変換処理などを行うメソッドを実装する。
            | モデル変換が単純な値のコピーのみで済む場合は、Helperクラスは作成せずに「:doc:`../GeneralFuncDetail/Dozer`」を使用すればよいため、オプションの扱いにしている。

|

* **ドメイン層のモジュール**

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90
    
        * - 項番
          - 説明
        * - | (5)
          - | ドメイン層で実装するモジュールは、アプリケーションの種類に依存しないため、本節での説明は割愛する。
            | 各モジュールの役割については「:doc:`../../Overview/ApplicationLayering`」を、ドメイン層の開発については「:doc:`../../ImplementationAtEachLayer/DomainLayer`」を参照されたい。

|

* **インフラストラクチャ層のモジュール**

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90
    
        * - 項番
          - 説明
        * - | (6)
          - | インフラストラクチャ層で実装するモジュールは、アプリケーションの種類に依存しないため、本節での説明は割愛する。
            | 各モジュールの役割については「:doc:`../../Overview/ApplicationLayering`」を、インフラストラクチャ層の開発については「:doc:`../../ImplementationAtEachLayer/InfrastructureLayer`」を参照されたい。


|

REST APIの実装サンプル
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 詳細な説明を行う前に、どのようなクラスをアプリケーション層に作成する事になるのかを知ってもらうために、ResourceクラスとControllerクラスの実装サンプルを以下に示す。
| 下記に示す実装サンプルは、「:doc:`../../Tutorial/TutorialREST`」で題材としているTodoリソースのREST APIである。

 .. note::

    \ **詳細な説明を読む前に、まずは**\「:doc:`../../Tutorial/TutorialREST`」\ **を実践する事を強く推奨する。**\

    チュートリアルでは”習うより慣れろ”を目的としており、 詳細な説明の前に実際に手を動かすことでTERASOLUNA Server Framework for Java (5.x)によるRESTful Web Serviceの開発を体感する事が出来る。
    RESTful Web Serviceの開発を体感した後に、詳細な説明を読むことで、RESTful Web Serviceの開発に対する理解度がより深まる事が期待できる。
    
    特にRESTful Web Serviceの開発経験がない場合は、「チュートリアルの実践」 → 「アーキテクチャ、設計、開発に関する詳細な説明(次節以降で説明)」 → 「チュートリアルの振り返り(再実践)」というプロセスを踏むことを推奨する。

|

* 実装サンプルで扱うリソース

 実装サンプルで扱うリソース(Todoリソース)は、以下のJSON形式とする。

 .. code-block:: json

    {
        "todoId" : "9aef3ee3-30d4-4a7c-be4a-bc184ca1d558",
        "todoTitle" : "Hello World!",
        "finished" : false,
        "createdAt" : "2014-02-25T02:21:48.493+0000"
    } 

|

* Resourceクラスの実装サンプル

 上記で示したTodoリソースを表現するJavaBeanとして、Resourceクラスを作成する。
 

 .. code-block:: java

    package todo.api.todo;

    import java.io.Serializable;
    import java.util.Date;
    
    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;
    
    public class TodoResource implements Serializable {

        private static final long serialVersionUID = 1L;

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

|

* Controllerクラス(REST API)の実装サンプル

 Todoリソースに対して、以下の5つのREST API(Controllerのハンドラメソッド)を作成する。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.10\linewidth}|p{0.25\linewidth}|p{0.15\linewidth}|p{0.20\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 15 10 30 15 20

    * - | 項番
      - | API名
      - | HTTP
        | メソッド
      - | パス
      - | ステータス
        | コード
      - | 説明
    * - | (1)
      - | GET Todos
      - | GET
      - | \ ``/api/v1/todos``\ 
      - | 200
        | (OK)
      - | Todoリソースを全件取得する。
    * - | (2)
      - | POST Todos
      - | POST
      - | \ ``/api/v1/todos``\ 
      - | 201
        | (Created)
      - | Todoリソースを新規作成する。
    * - | (3)
      - | GET Todo
      - | GET
      - | \ ``/api/v1/todos/{todoId}``\ 
      - | 200
        | (OK)
      - | Todoリソースを一件取得する。
    * - | (4)
      - | PUT Todo
      - | PUT
      - | \ ``/api/v1/todos/{todoId}``\ 
      - | 200
        | (OK)
      - | Todoリソースを完了状態に更新する。
    * - | (5)
      - | DELETE Todo
      - | DELETE
      - | \ ``/api/v1/todos/{todoId}``\ 
      - | 204
        | (No Content)
      - | Todoリソースを削除する。

 .. code-block:: java
    :emphasize-lines: 30-34, 42-45, 51-55, 59-63, 68-72

    package todo.api.todo;

    import java.util.ArrayList;
    import java.util.Collection;
    import java.util.List;

    import javax.inject.Inject;

    import org.dozer.Mapper;
    import org.springframework.http.HttpStatus;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.PathVariable;
    import org.springframework.web.bind.annotation.RequestBody;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.bind.annotation.ResponseStatus;
    import org.springframework.web.bind.annotation.RestController;

    import todo.domain.model.Todo;
    import todo.domain.service.todo.TodoService;

    @RestController
    @RequestMapping("todos")
    public class TodoRestController {
        @Inject
        TodoService todoService;
        @Inject
        Mapper beanMapper;

        // (1)
        @RequestMapping(method = RequestMethod.GET)
        @ResponseStatus(HttpStatus.OK)
        public List<TodoResource> getTodos() {
            Collection<Todo> todos = todoService.findAll();
            List<TodoResource> todoResources = new ArrayList<>();
            for (Todo todo : todos) {
                todoResources.add(beanMapper.map(todo, TodoResource.class));
            }
            return todoResources;
        }

        // (2)
        @RequestMapping(method = RequestMethod.POST)
        @ResponseStatus(HttpStatus.CREATED)
        public TodoResource postTodos(@RequestBody @Validated TodoResource todoResource) {
            Todo createdTodo = todoService.create(beanMapper.map(todoResource, Todo.class));
            TodoResource createdTodoResponse = beanMapper.map(createdTodo, TodoResource.class);
            return createdTodoResponse;
        }

        // (3)
        @RequestMapping(value="{todoId}", method = RequestMethod.GET)
        @ResponseStatus(HttpStatus.OK)
        public TodoResource getTodo(@PathVariable("todoId") String todoId) {
            Todo todo = todoService.findOne(todoId);
            TodoResource todoResource = beanMapper.map(todo, TodoResource.class);
            return todoResource;
        }

        // (4)
        @RequestMapping(value="{todoId}", method = RequestMethod.PUT)
        @ResponseStatus(HttpStatus.OK)
        public TodoResource putTodo(@PathVariable("todoId") String todoId) {
            Todo finishedTodo = todoService.finish(todoId);
            TodoResource finishedTodoResource = beanMapper.map(finishedTodo, TodoResource.class);
            return finishedTodoResource;
        }
        
        // (5)
        @RequestMapping(value="{todoId}", method = RequestMethod.DELETE)
        @ResponseStatus(HttpStatus.NO_CONTENT)
        public void deleteTodo(@PathVariable("todoId") String todoId) {
            todoService.delete(todoId);
        }

    }



|

.. _RESTAboutResourceOrientedArchitecture:

Architecture
--------------------------------------------------------------------------------
| 本節では、RESTful Web Serviceを構築するためのアーキテクチャについて説明する。

| RESTful Web Serviceを構築するためのアーキテクチャとして、リソース指向アーキテクチャ(ROA)というものが存在する。
| ROAは、「\ **R**\esource \ **O**\riented \ **A**\rchitecture」の略であり、\ **RESTのアーキテクチャスタイル(原則)に従ったWeb Serviceを構築するための具体的なアーキテクチャ**\を定義している。
| RESTful Web Serviceを作る際は、まずROAのアーキテクチャの理解を深めてほしい。

| 本節では、ROAのアーキテクチャとして、以下の7つについて説明する。
| これらは、RESTful Web Serviceを構築する上で重要なアーキテクチャ要素であるが、必ず全てを適用しなくてはいけないという事ではない。
| 開発するアプリケーションの特性を考慮し、必要なものを適用してほしい。

以下の5つのアーキテクチャは、アプリケーションの特性に関係なく適用すべきアーキテクチャである。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 35 55

    * - 項番
      - アーキテクチャ
      - アーキテクチャの概要
    * - | (1)
      - | :ref:`RESTOverviewProvideResourceOnWeb`
      - | システム上で管理する情報をクライアントに提供する手段として、Web上のリソースとして公開する。
    * - | (2)
      - | :ref:`RESTOverviewAssignURI`
      - | クライアントに公開するリソースには、Web上のリソースとして一意に識別できるURI(Universal Resource Identifier)を割り当てる。
    * - | (3)
      - | :ref:`RESTOverviewOperatedByHttpMethod`
      - | リソースに対する操作は、HTTPメソッド(GET,POST,PUT,DELETE)を使い分けることで実現する。
    * - | (4)
      - | :ref:`RESTOverviewResourceRepresentationFormat`
      - | リソースのフォーマットは、JSON又はXMLなどのデータ構造を示すためのフォーマットを使用する。
    * - | (5)
      - | :ref:`RESTOverviewHttpStatusCode`
      - | クライアントへ返却するレスポンスには、適切なHTTPステータスコードを設定する。

|

以下の2つのアーキテクチャは、アプリケーションの特性に応じて、適用するアーキテクチャである。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 35 55

    * - 項番
      - アーキテクチャ
      - アーキテクチャの概要
    * - | (6)
      - | :ref:`RESTOverviewClientServerCommunicateOnStateless`
      - | サーバ上でアプリケーションの状態は保持せずに、クライアントからリクエストされてきた情報のみで処理を行うようにする。
    * - | (7)
      - | :ref:`RESTOverviewHyperMediaLinksToRelatedResources`
      - | リソースの中には、指定されたリソースと関連をもつ他のリソースへのリンク(URI)を含める。


|

.. _RESTOverviewProvideResourceOnWeb:

Web上のリソースとして公開
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| **システム上で管理する情報をクライアントに提供する手段として、Web上のリソースとして公開する。**
| これは、HTTPプロトコルを使ってリソースにアクセスできるようにする事を意味しており、その際にリソースを識別する方法とし、URIが使用される。

例えば、ショッピングサイトを提供するWebシステムであれば、以下のような情報がWeb上のリソースとして公開する事になる。

* 商品の情報
* 在庫の情報
* 注文の情報
* 会員の情報
* 会員毎の認証の情報(ログインIDとパスワードなど)
* 会員毎の注文履歴の情報
* 会員毎の認証履歴の情報
* and more ...

|

.. _RESTOverviewAssignURI:

URIによるリソースの識別
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| **クライアントに公開するリソースには、Web上のリソースとして一意に識別できるURI(Universal Resource Identifier)を割り当てる。**
| 実際に使用されるのは、URIのサブセットであるURL(Uniform Resource Locator)となる。

| ROAでは、URIを使用してWeb上のリソースにアクセスできる状態になっていることを「アドレス可能性」と呼んでいる。
| これは同じURIを使用すれば、どこからでも同じリソースにアクセスできる状態になっている事を意味している。

| RESTful Web Serviceに割り当てるURIは、「\ **リソースの種類を表す名詞**\」と「\ **リソースを一意に識別するための値(IDなど)**\」の組み合わせとする。
| 例えば、ショッピングサイトを提供するWebシステムで扱う商品情報のURIは、以下のようになる。

* | \ `http://example.com/api/v1/items`\
  | 「**items**」の部分が「リソースの種類を表す名詞」となり、リソースの数が複数になる場合は、複数系の名詞を使用する。
  | 上記例では、商品情報である事を表す名詞の複数系を指定しており、商品情報を一括で操作する際に使用するURIとなる。これは、ファイルシステムに置き換えると、ディレクトリに相当する。

* | \ `http://example.com/api/v1/items/I312-535-01216`\
  | 「**I312-535-01216**」の部分が「リソースを識別するための値」となり、リソース毎に異なる値となる。
  | 上記例では、商品情報を一意に識別するための値として商品IDを指定しており、特定の商品情報を操作する際に使用するURIとなる。これは、ファイルシステムに置き換えると、ディレクトリの中に格納されているファイルに相当する。

|

.. warning::
 
    RESTful Web Serviceに割り当てるURIには、下記で示すような\ **操作を表す動詞を含んではいけない。**\
    
    * \ `http://example.com/api/v1/items?get&itemId=I312-535-01216`\
    * \ `http://example.com/api/v1/items?delete&itemId=I312-535-01216`\
    
    上記例では、 URIの中に\ **get**\や\ **delete**\という動詞を含んでいるため、RESTful Web Serviceに割り当てるURIとして適切ではない。
    
    RESTful Web Serviceでは、\ **リソースに対する操作はHTTPメソッド(GET,POST,PUT,DELETE)を使用して表現する。**\

|

.. _RESTOverviewOperatedByHttpMethod:

HTTPメソッドによるリソースの操作
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| **リソースに対する操作は、HTTPメソッド(GET,POST,PUT,DELETE)を使い分けることで実現する。**

| ROAでは、HTTPメソッドの事を「統一インタフェース」と呼んでいる。
| これは、HTTPメソッドがWeb上で公開される全てのリソースに対して実行する事ができ、且つリソース毎にHTTPメソッドの意味が変わらない事を意味している。

以下に、HTTPメソッドに割り当てられるリソースに対する操作の対応付けと、それぞれの操作が保証すべき事後条件について説明する。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.35\linewidth}|p{0.35\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 35 35

    * - 項番
      - HTTPメソッド
      - リソースに対する操作
      - 操作が保証すべき事後条件
    * - | (1)
      - | GET
      - | リソースを取得する。
      - | 安全性、べき等性。
    * - | (2)
      - | POST
      - | リソースの作成する。
      - | 作成したリソースのURIの割り当てはサーバが行い、割り当てたURIはレスポンスのLocationヘッダに設定してクライアントに返却する。
    * - | (4)
      - | PUT
      - | リソースを作成又は更新する。
      - | べき等性。
    * - | (5)
      - | PATCH
      - | リソースを差分更新する。
      - | べき等性。
    * - | (6)
      - | DELETE
      - | リソースを削除する。
      - | べき等性。
    * - | (7)
      - | HEAD
      - | リソースのメタ情報を取得する。
        | GETと同じ処理を行いヘッダのみ応答する。
      - | 安全性、べき等性。
    * - | (8)
      - | OPTIONS
      - | リソースに対して使用できるHTTPメソッドの一覧を応答する。
      - | 安全性、べき等性。

 .. note:: **安全性とべき等性の保証について**
 
    HTTPメソッドを使ってリソースの操作を行う場合、事後条件として、「安全性」と「べき等性」の保証を行う事が求められる。

    **【安全性とは】**

            ある数字に1を何回掛けても、数字がかわらない事(10に1を何回掛けても結果は10のままである事)を保証する。
            これは、同じ操作を何回行ってもリソースの状態が変わらない事を保証する事である。

    **【べき等性とは】**

            数字に0を何回掛けても0になる事(10に0を1回掛けても何回掛けても結果は共に0になる事)を保証する。
            これは、一度操作を行えば、その後で同じ操作を何回行ってもリソースの状態が変わらない事を保証する事である。
            ただし、別のクライアントが同じリソースの状態を変更している場合は、べき等性を保障する必要はなく、事前条件に対するエラーとして扱ってもよい。
    

 .. tip:: **クライアントがリソースに割り当てるURIを指定してリソースを作成する場合**
 
    リソースを作成する際に、クライアントによってリソースに割り当てるURIを指定する場合は、\ **作成するリソースに割り当てるURIに対して、PUTメソッドを呼び出すことで実現する。**\

    PUTメソッドを使用してリソースを作成する場合、
    
     * 指定されたURIにリソースが存在しない場合はリソースを作成
     * 既にリソースが存在する場合はリソースの状態を更新
    
    するのが一般的な動作である。
    
    以下に、PUTとPOSTメソッドを使ってリソースを作成する際の処理イメージの違いについて説明する。
    
    **【PUTメソッドを使用してリソースを作成する際の処理イメージ】**

         .. figure:: ./images_REST/RESTCreatedNewResourceUsingByPutMethod.png
           :alt: Image of processing for create new resource using by PUT method
           :width: 70%

         .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
         .. list-table::
            :header-rows: 1
            :widths: 10 90
        
            * - 項番
              - 説明
            * - | (1)
              - | URIに作成するリソースのURI(ID)を指定して、PUTメソッドを呼び出す。
            * - | (2)
              - | URIで指定されたIDのEntityを作成する。
                | 既に同じIDで作成済みの場合は、内容を更新する。
            * - | (3)
              - | 作成又は更新したリソースを応答する。

    **【POSTメソッドを使用してリソースを作成する際の処理イメージ】**

         .. figure:: ./images_REST/RESTCreatedNewResourceUsingByPostMethod.png
           :alt: Image of processing for create new resource using by POST method
           :width: 70%


         .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
         .. list-table::
            :header-rows: 1
            :widths: 10 90

            * - 項番
              - 説明
            * - | (1)
              - | POSTメソッドを呼び出す。
            * - | (2)
              - | リクエストされリソースを識別するためのIDを生成する。
            * - | (3)
              - | (2)で生成したIDのEntityを作成する。
            * - | (4)
              - | 作成したリソースを応答する。
                | レスポンスのLocationヘッダに生成したリソースにアクセスするためのURIを設定する。
  
|

.. _RESTOverviewResourceRepresentationFormat:

適切なフォーマットの使用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
**リソースのフォーマットは、JSON又はXMLなどのデータ構造を示すためのフォーマットを使用する。**

| ただし、リソースの種類によっては、JSONやXML以外のフォーマットを使ってもよい。
| 例えば、統計情報に分類される様なリソースでは、折れ線グラフを画像フォーマット(バイナリデータ)としてリソースを公開する事も考えられる。

| リソースのフォーマットとして、複数のフォーマットをサポートする場合は、以下のどちらかの方法で切り替えを行う。


* **拡張子によって切り替えを行う。**

  | レスポンスのフォーマットは、拡張子を指定する事で切り替える事ができる。
  | **本ガイドラインでは、拡張子による切り替えを推奨する。**
  | 推奨する理由は、レスポンスするフォーマット指定が簡単であるという点と、レスポンスするフォーマットがURIに含まれ、直感的なURIになるという点である。

 .. note:: **拡張子で切り替える場合のURI例**
    
    * \ `http://example.com/api/v1/items.json`\
    * \ `http://example.com/api/v1/items.xml`\
    * \ `http://example.com/api/v1/items/I312-535-01216.json`\
    * \ `http://example.com/api/v1/items/I312-535-01216.xml`\

|

* **リクエストのAcceptヘッダのMIMEタイプによって切り替えを行う。**

  RESTful Web Serviceで使用される代表的なMIMEタイプを以下に示す。

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 30 60
    
        * - 項番
          - フォーマット
          - MIMEタイプ
        * - | (1)
          - | JSON
          - | application/json
        * - | (2)
          - | XML
          - | application/xml

|


.. _RESTOverviewHttpStatusCode:

適切なHTTPステータスコードの使用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
\ **クライアントへ返却するレスポンスには、適切なHTTPステータスコードを設定する。**\

| HTTPステータスコードには、クライアントから受け取ったリクエストをサーバがどのように処理したかを示す値を設定する。
| \ **これはHTTPの仕様であり、HTTPの仕様に可能な限り準拠することを推奨する。**\

 .. tip:: **HTTPの仕様について**
 
    `RFC 2616(Hypertext Transfer Protocol -- HTTP/1.1)の6.1.1 Status Code and Reason Phrase <http://tools.ietf.org/search/rfc2616#section-6.1.1>`_ を参照されたい。

|

| ブラウザにHTMLを返却するような伝統的なWebシステムでは、処理結果に関係なく\ ``"200 OK"``\を応答し、処理結果はエンティティボディ(HTML)の中で表現するという事が一般的であった。
| HTMLを返却するような伝統的なWebアプリケーションでは、処理結果を判断するのはオペレータ(人間)のため、この仕組みでも問題が発生する事はなかった。
| しかし、この仕組みでRESTful Web Serviceを構築した場合、以下のような問題が潜在的に存在することになるため、適切なHTTPステータスコードを設定することを推奨する。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 潜在的な問題点
    * - | (1)
      - | 処理結果(成功と失敗)のみを判断すればよい場合でも、エンティティボディを解析処理が必須になるため、無駄な処理が必要になる。
    * - | (2)
      - | エラーハンドリングを行う際に、システム独自に定義されたエラーコードを意識する事が必須になるため、クライアント側のアーキテクチャ(設計及び実装)に悪影響を与える可能性がある。
    * - | (3)
      - | クライアント側でエラー原因を解析する際に、システム独自に定義されたエラーコードの意味を理解しておく必要があるため、直感的なエラー解析の妨げになる可能性がある。

|

.. _RESTOverviewClientServerCommunicateOnStateless:

ステートレスなクライアント/サーバ間の通信
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| **サーバ上でアプリケーションの状態は保持せずに、クライアントからリクエストされてきた情報のみで処理を行うようにする。**

| ROAでは、サーバ上でアプリケーションの状態を保持しない事を「ステートレス性」と呼んでいる。
| これは、アプリケーションサーバのメモリ(HTTPセッションなど)にアプリケーションの状態を保持しない事を意味し、リクエストされた情報のみでリソースに対する操作が完結できる状態にしておく事を意味している。
| 本ガイドラインでは、\ **可能な限り「ステートレス性」を保つことを推奨する。**\

 .. note:: **アプリケーションの状態とは**
 
    Webページの遷移状態、入力値、プルダウン/チェックボックス/ラジオボタンなどの選択状態、認証状態などの事である。

 .. note:: **CSRF対策との関連**
 
    本ガイドラインに記載されているCSRF対策をRESTful Web Serviceに対して行った場合、CSRF対策用のトークン値がHTTPセッションに保存されるため、クライアントとサーバ間の「ステートレス性」を保つ事が出来ないという点を補足しておく。

    そのため、CSRF対策を行う場合は、システムの可用性を考慮する必要がある。
    
    高い可用性が求められるシステムでは、
    
    * APサーバをクラスタ化し、セッションをレプリケーションする。
    * セッションの保存先をAPサーバのメモリ以外にする。
    
    等の対策が必要となる。
    ただし、上記対策は性能への影響があるため、性能要件も考慮する必要がある。
    
    CSRF対策については、\ :doc:`../../Security/CSRF`\を参照されたい。

 .. todo:: **TBD**

    高い可用性が求められる場合は、「CSRF対策用のトークン値をAPサーバのメモリ(HTTPセッション)以外に保存する」アーキテクチャを検討した方がよい。
    
    具体的なアーキテクチャについては、現在検討中であり、次版以降に記載する予定である。
    
|

.. _RESTOverviewHyperMediaLinksToRelatedResources:

関連のあるリソースへのリンク
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| \ **リソースの中には、指定されたリソースと関連をもつ他のリソースへのハイパーメディアリンク(URI)を含める。**\

| ROAでは、リソース状態の表現の中に、他のリソースへのハイパーメディアリンクを含めることを「接続性」と呼んでいる。
| これは、関連をもつリソース同士が相互にリンクを保持し、そのリンクをたどる事で関連する全てのリソースにアクセスできる状態にしておく事を意味している。

下記に、ショッピングサイトの会員情報のリソースを例に、リソースの接続性について説明する。

 .. figure:: ./images_REST/RESTConnectivity.png
   :alt: Image of resource connectivity
   :width: 100%

|

 .. raw:: latex

    \newpage

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | 会員情報のリソースを取得(\ ``GET http://example.com/api/v1/memebers/M000000001``\)を行うと、以下のJSONが返却される。

         .. code-block:: json
            :emphasize-lines: 13-14,17-18

            {
                "memberId" : "M000000001",
                "memberName" : "John Smith",
                "address" : {
                    "address1" : "45 West 36th Street",
                    "address2" : "7th Floor",
                    "city" : "New York",
                    "state" : "NY",
                    "zipCode" : "10018"
                },
                "links" : [
                    {
                        "rel" : "orders",
                        "href" : "http://example.com/api/v1/memebers/M000000001/orders"
                    },
                    {
                        "rel" : "authentications",
                        "href" : "http://example.com/api/v1/memebers/M000000001/authentications"
                    }
                ]
            }

        | ハイライトした部分が、関連をもつ他のリソースへのハイパーメディアリンク(URI)となる。
        | 上記例では、会員毎の注文履歴と認証履歴のリソースに対して接続性を保持している。
    * - | (2)
      - | 返却されたJSONに設定されているハイパーメディアリンク(URI)を使用して、注文履歴のリソースを取得(\ ``GET http://example.com/api/v1/memebers/M000000001/orders``\)を行うと、以下のJSONが返却される。

         .. code-block:: json
            :emphasize-lines: 10-11,22-23,30-31
        
            {
                "orders" : [
                    {
                        "orderId" : "029b49d7-0efa-411b-bc5a-6570ce40ead8",
                        "orderDatetime" : "2013-12-27T20:34:50.897Z", 
                        "orderName" : "Note PC",
                        "shopName" : "Global PC Shop",
                        "links" : [
                            {
                                "rel" : "order",
                                "href" : "http://example.com/api/v1/memebers/M000000001/orders/029b49d7-0efa-411b-bc5a-6570ce40ead8"
                            }
                        ]
                    },
                    {
                        "orderId" : "79bf991d-d42d-4546-9265-c5d4d59a80eb",
                        "orderDatetime" : "2013-12-03T19:01:44.109Z", 
                        "orderName" : "Orange Juice 100%",
                        "shopName" : "Global Food Shop",
                        "links" : [
                            {
                                "rel" : "order",
                                "href" : "http://example.com/api/v1/memebers/M000000001/orders/79bf991d-d42d-4546-9265-c5d4d59a80eb"
                            }
                        ]
                    }
                ],
                "links" : [
                    {
                        "rel" : "ownerMember",
                        "href" : "http://example.com/api/v1/memebers/M000000001"
                    }
                ]
            }

        | ハイライトした部分が、関連をもつ他のリソースへのハイパーメディアリンク(URI)となる。
        | 上記例では、注文履歴のオーナの会員情報のリソース及び注文履歴のリソースに対する接続性を保持している。
    * - | (3)
      - | 注文履歴のオーナとなる会員情報のリソースを再度取得(\ ``GET http://example.com/api/v1/memebers/M000000001``\)し、返却されたJSONに設定されているハイパーメディアリンク(URI)を使用して、認証履歴のリソースを取得(\ ``GET http://example.com/api/v1/memebers/M000000001/authentications/``\)を行うと、以下のJSONが返却される。
        
         .. code-block:: json
            :emphasize-lines: 18-19
        
            {
                "authentications" : [
                    {
                        "authenticationId" : "6ae9613b-85b6-4dd1-83da-b53c43994433",
                        "authenticationDatetime" : "2013-12-27T20:34:50.897Z", 
                        "clientIpaddress" : "230.210.3.124",
                        "authenticationResult" : true
                    },
                    {
                        "authenticationId" : "103bf3c5-7707-46eb-b2d8-c00ce6243d5f",
                        "authenticationDatetime" : "2013-12-26T10:03:45.001Z", 
                        "clientIpaddress" : "230.210.3.124",
                        "authenticationResult" : false
                    }
                ],
                "links" : [
                    {
                        "rel" : "ownerMember",
                        "href" : "http://example.com/api/v1/memebers/M000000001"
                    }
                ]
            }
        
        | ハイライトした部分が、関連をもつ他のリソースへのハイパーメディアリンク(URI)となる。
        | 上記例では、認証履歴のオーナとなる会員情報のリソースに対して接続性を保持している。

 .. raw:: latex

    \newpage

|

| リソースの中に他のリソースへのハイパーメディアリンク(URI)を含めることは、必須ではない。
| 事前に全てのREST APIのエンドポイント(URI)を公開している場合、リソースの中に関連リソースへのリンクを設けても、リンクが使用されない可能性が高い。
| 特に、システム間でリソースのやりとりを行うためのREST APIの場合は、事前に公開されているREST APIのエンドポイントに対して直接アクセスするような実装になる事が多いため、リンクを設ける意味がない事がある。
| リンクを設ける意味がない場合は、無理にリンクを設ける必要はない。

| 逆に、ユーザインタフェースを持つクライアントアプリケーションとRESTful Web Serviceの間で直接リソースのやりとりを行う場合は、リンクを設けることで、クライアントとサーバ間の疎結合性を高めることが出来る。
| クライアントとサーバ間の疎結合性を高めることが出来る理由は以下の通りである。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 疎結合性を高めることが出来る理由
    * - | (1)
      - | クライアントアプリケーションは、リンクの論理名のみ事前に知っていればよいため、REST APIを呼び出すための具体的なURIを意識する必要がなくなる。
    * - | (2)
      - | クライアントアプリケーションが具体的なURIを意識する必要がなくなるため、サーバ側のURIを変更する際に与える影響度を最小限に抑える事ができる。

上記にあげた点を考慮し、他のリソースへのハイパーメディアリンク(URI)を設けるか否かを判断すること。

.. tip:: **HATEOASとの関係**

    HATEOASは、「\ **H**\ypermedia \ **A**\s \ **T**\he \ **E**\ngine \ **O**\f \ **A**\pplication \ **S**\tate」の略であり、RESTfulなWebアプリケーションを作成するためのアーキテクチャの一つである。

    HATEOASのアーキテクチャでは、
    
    * サーバは、クライアントとサーバ間でやり取りするリソース(JSONやXML)の中に、アクセス可能なリソースへのハイパーメディアリンク(URI)を含める。
    * クライアントは、リソース表現(JSONやXML)の中に含まれるハイパーメディアリンクを介して、サーバから必要なリソースを取得し、アプリケーションの状態(画面の状態など)を変化させる。
    
    ことになるため、関連のあるリソースへのリンクを設ける事は、HATEOASのアーキテクチャと一致する。
    
    サーバとクライアントとの疎結合性を高めたい場合は、HATEOASのアーキテクチャを採用する事を検討されたい。
    
    

|

.. _RESTHowToDesign:

How to design
--------------------------------------------------------------------------------
本説では、RESTful Web Serviceの設計について説明する。

.. _RESTHowToDesignExtractResource:

リソースの抽出
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
まず、Web上に公開するリソースを抽出する。

リソースを抽出する際の注意点を以下に示す。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - リソース抽出時の注意点
    * - | (1)
      - | Web上に公開するリソースは、データベースなどで管理されている情報になるが、\ **安易にデータベースのデータモデルをそのままリソースとして公開してはいけない。**\
        | データベースに格納されている項目の中には、クライアントに公開すべきでない項目もあるので、精査が必要である。
    * - | (2)
      - | \ **データベースの同じテーブルで管理されている情報であっても、情報の種類が異なる場合は、別のリソースとして公開する事を検討する。**\
        | 本質的には別の情報だが、データ構造が同じという理由で同じテーブルで管理されているケースがあるので、精査が必要である。
    * - | (3)
      - | RESTful Web Serviceでは、イベントで操作する情報をリソースとして抽出する。
        | \ **イベント自体をリソースとして抽出してはいけない。**\
        |
        | 例えば、ワークフロー機能で発生するイベント(承認、否認、差し戻しなど)から呼び出されるRESTful Web Serviceを作成する場合、ワークフロー自体やワークフローの状態を管理するための情報をリソースとして抽出する。

|

.. _RESTHowToDesignAssignURI:

URIの割り当て
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
抽出したリソースに対して、リソースを識別するためのURIを割り当てる。

URIは、以下の形式を推奨する。

* ``http(s)://{ドメイン名(:ポート番号)}/{REST APIであることを示す値}/{APIバージョン}/{リソースを識別するためのパス}``\

* ``http(s)://{REST APIであることを示すドメイン名(:ポート番号)}/{APIバージョン}/{リソースを識別するためのパス}``\

具体例は以下の通り。

* ``http://example.com/api/v1/members/M000000001``\

* ``http://api.example.com/v1/members/M000000001``\

|

.. _RESTHowToDesignAssignUriForPublishAPI:

REST APIであることを示すためのURIの割り当て
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
RESTful Web Service(REST API)向けのURIであること明確にするために、URI内のドメイン又はパスに \ ``api``\ を含めることを推奨する。

具体的には、以下のようなURIとする。

* ``http://example.com/api/...``\
* ``http://api.example.com/...``\

|

.. _RESTHowToDesignAssignUriForApiVersion:

APIバージョンを識別するためのURIの割り当て
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
RESTful Web Serviceは、複数のバージョンで稼働が必要になる可能性があるため、クライアントに公開するURIには、APIバージョンを識別するための値を含めるようにする事を推奨する。

具体的には、以下のような形式のURIとする。

* ``http://example.com/api/{APIバージョン}/{リソースを識別するためのパス}``\
* ``http://api.example.com/{APIバージョン}/{リソースを識別するためのパス}``\

.. todo:: **TBD**
 
    URIの中にAPIバージョンを含めるべきかは、現在検討中である。

|

.. _RESTHowToDesignAssignUriForResource:

リソースを識別するためのパスの割り当て
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Web上に公開するリソースに対して、以下の２つのURIを割り当てる。
| 下記の例では、会員情報をWeb上に公開する場合のURI例を記載している。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.25\linewidth}|p{0.30\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 35 25 30

    * - 項番
      - URIの形式
      - URIの具体例
      - 説明
    * - | (1)
      - | /{リソースのコレクションを表す名詞}
      - | /api/v1/members
      - | リソースを一括で操作する際に使用するURIとなる。
    * - | (2)
      - | /{リソースのコレクションを表す名詞/リソースの識別子(IDなど)}
      - | /api/v1/members/M0001
      - | 特定のリソースを操作する際に使用するURIとなる。

|

| Web上に公開する関連リソースへのURIは、ネストさせて表現する。
| 下記の例では、会員毎の注文情報をWeb上に公開する場合のURI例を記載している。
    
 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.25\linewidth}|p{0.30\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 35 25 30
    
    * - 項番
      - URIの形式
      - URIの具体例
      - 説明
    * - | (3)
      - | {リソースのURI}/{関連リソースのコレクションを表す名詞}
      - | /api/v1/members/M0001/orders
      - | 関連リソースを一括で操作する際に使用するURIとなる。
    * - | (4)
      - | {リソースのURI}/{関連リソースのコレクションを表す名詞}/{関連リソースの識別子(IDなど)}
      - | /api/v1/members/M0001/orders/O0001
      - | 特定の関連リソースを操作する際に使用するURIとなる。

|

| Web上に公開する関連リソースの要素が1件の場合は、関連リソースを示す名詞は複数系ではなく単数形とする。
| 下記の例では、会員毎の資格情報をWeb上に公開する場合のURI例を記載している。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.25\linewidth}|p{0.30\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 35 25 30

    * - 項番
      - URIの形式
      - URIの具体例
      - 説明
    * - | (5)
      - | {リソースのURI}/{関連リソースを表す名詞}
      - | /api/v1/members/M0001/credential
      - | 要素が1件の関連リソースを操作する際に使用するURI。

|

.. _RESTHowToDesignAssignHttpMethod:

HTTPメソッドの割り当て
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
リソース毎に割り当てたURIに対して、以下のHTTPメソッドを割り当て、リソースに対するCRUD操作をREST APIとして公開する。

 .. note:: **HEADとOPTIONSメソッドについて**
 
    以降の説明では、HEADとOPTIONSメソッドについても触れているが、REST APIとしての提供は任意とする。
    
    HTTPの仕様に準拠したREST APIを作成する場合は、HEAD及びOPTIONSメソッドの提供も必要だが、実際に使われるケースは稀であり、必要ない事が多いためである。

|

.. _RESTHowToDesignAssignHttpMethodForCollectionResource:

リソースコレクションのURIに対するHTTPメソッドの割り当て
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 70

    * - 項番
      - HTTPメソッド
      - 実装するREST APIの概要
    * - | (1)
      - | GET
      - | URIで指定されたリソースのコレクションを取得するREST APIを実装する。
    * - | (2)
      - | POST
      - | 指定されたリソースを作成しコレクションに追加するREST APIを実装する。
    * - | (3)
      - | PUT
      - | URIで指定されたリソースの一括更新を行うREST APIを実装する。
    * - | (4)
      - | DELETE
      - | URIで指定されたリソースの一括削除を行うREST APIを実装する。
    * - | (5)
      - | HEAD
      - | URIで指定されたリソースコレクションのメタ情報を取得するREST APIを実装する。
        | GETと同じ処理を行いヘッダのみ応答する。
    * - | (6)
      - | OPTIONS
      - | URIで指定されたリソースコレクションでサポートされているHTTPメソッド(API)のリストを応答するREST APIを実装する。

|

.. _RESTHowToDesignAssignHttpMethodForSpecifiedResource:

特定リソースのURIに対するHTTPメソッドの割り当て
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 70

    * - 項番
      - HTTPメソッド
      - 実装するREST APIの概要
    * - | (1)
      - | GET
      - | URIで指定されたリソースを取得するREST APIを実装する。
    * - | (2)
      - | PUT
      - | URIで指定されたリソースの作成又は更新を行うREST APIを実装する。
    * - | (3)
      - | DELETE
      - | URIで指定されたリソースの削除を行うREST APIを実装する。
    * - | (4)
      - | HEAD
      - | URIで指定されたリソースのメタ情報を取得するREST APIを実装する。
        | GETと同じ処理を行いヘッダのみ応答する。
    * - | (5)
      - | OPTIONS
      - | URIで指定されたリソースでサポートされているHTTPメソッド(API)のリストを応答するREST APIを実装する。

|

.. _RESTHowToDesignResourceRepresentationFormat:

リソースのフォーマット
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| リソースを表現するフォーマットとしては、\ **JSONを使用する事を推奨する。**\
| 以降の説明では、リソースを表現するフォーマットとしてJSONを使用する前提で説明を記載する。


JSONのフィールド名
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| JSONのフィールド名は、\ **「lower camel case」にすることを推奨する。**\
| これはクライアントアプリケーションの一つとして想定されるJavaScriptとの相性を考慮した結果である。

| フィールド名を「lower camel case」にした場合のJSONのサンプルは以下の通り。
| 「lower camel case」は、先頭文字を小文字にし、単語の先頭文字を大文字にする。

 .. code-block:: json

    {
        "memberId" : "M000000001"
    }

|

NULLとブランク文字
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| JSONの値として、\ **NULLとブランク文字は区別する事を推奨する。**\
| アプリケーションの処理としてNULLとブランク文字を同一視する事はよくあるが、JSONに設定する値としては、NULLとブランク文字は区別しておいた方がよい。

| NULLとブランク文字を区別した場合のJSONのサンプルは以下の通り。

 .. code-block:: json

    {
        "dateOfBirth" : null,
        "address1" : ""
    }


|

日時のフォーマット
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| JSONの日時フィールドの形式は、\ **ISO-8601の拡張形式とする事を推奨する。**\
| ISO-8601の拡張形式以外でもよいが、特に理由がない場合は、ISO-8601の拡張形式にすればよい。
| ISO-8601には基本形式と拡張形式があるが、拡張形式の方が視認性が高い表記方法である。

具体的には、以下の３つの形式となる。

1. yyyy-MM-dd

 .. code-block:: json

    {
        "dateOfBirth" : "1977-03-12"
    }

2. yyyy-MM-dd'T'HH:mm:ss.SSSZ

 .. code-block:: json

    {
        "lastModifiedAt" : "2014-03-12T22:22:36.637+09:00"
    }

3. yyyy-MM-dd'T'HH:mm:ss.SSS'Z' (UTC用の形式)

 .. code-block:: json

    {
        "lastModifiedAt" : "2014-03-12T13:11:27.356Z"
    }

|

パイパーメディアリンクの形式
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| パイパーメディアリンクを設ける場合は、以下に示す形式とすることを推奨する。
| 推奨する形式のサンプルは以下の通り。

 .. code-block:: json

    {
        "links" : [
            {
                "rel" : "ownerMember",
                "href" : "http://example.com/api/v1/memebers/M000000001"
            }
        ]
    }

 * \ ``"rel"``\と\ ``"href"``\という2つのフィールドを持ったLinkオブジェクトをコレクション形式で保持する。
 * \ ``"rel"``\には、なんのリンクか識別するためのリンク名を指定する。
 * \ ``"href"``\には、リソースにアクセスするためのURIを指定する。
 * Linkオブジェクトをコレクション形式で保持するフィールドは、\ ``"links"``\とする。

|

エラー応答時のフォーマット
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| エラーを検知した場合、どのようなエラーが発生したのか保持できるフォーマットにする事を推奨する。
| 特に、クライアントが再操作する事でエラーが解消できる可能性がある場合は、より詳細なエラー情報を含めた方がよい。
| 逆に、システムの脆弱性をさらすような事象が発生した場合は、詳細なエラー情報は含めるべきではない。この場合、詳細なエラー情報はログに出力すべきである。

エラーを検知した際に応答するフォーマット例を以下に示す。

 .. code-block:: json
    :emphasize-lines: 10, 20, 23

    {
      "code" : "e.ex.fw.7001",
      "message" : "Validation error occurred on item in the request body.",
      "details" : [ {
        "code" : "ExistInCodeList",
        "message" : "\"genderCode\" must exist in code list of CL_GENDER.",
        "target" : "genderCode"
      } ]
    }

上記のフォーマット例では、

* エラーコード(code)
* エラーメッセージ(message)
* エラー詳細リスト(details)

| をエラー応答時のフォーマットとして用意している。
| エラー詳細リストは、入力チェックエラー発生時に利用する事を想定しており、どのフィールドで、どのようなエラーが発生したのかを保持できるフォーマットとしている。

|

.. _RESTHowToDesignHttpStatusCode:

HTTPステータスコード
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
HTTPステータスコードは、以下の指針に則って応答する。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 方針
    * - | (1)
      - | リクエストが成功した場合は、成功又は転送を示すHTTPステータスコード(2xx又は3xx系)を応答する。
    * - | (2)
      - | リクエストが失敗した原因がクライアント側にある場合は、クライアントエラーを示すHTTPステータスコード(4xx系)を応答する。
        | リクエストが失敗した原因はクライアントにはないが、クライアントの再操作によってリクエストが成功する可能性がある場合も、クライアントエラーとする。
    * - | (3)
      - | リクエストが失敗した原因がサーバ側にある場合は、サーバエラーを示すHTTPステータスコード(5xx系)を応答する。

|

.. _RESTHowToDesignHttpStatusCodeForSuccess:

リクエストが成功した場合のHTTPステータスコード
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
リクエストが成功した場合は、状況に応じて以下のHTTPステータスコードを応答する。
 
 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.30\linewidth}|p{0.40\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 30 40

    * - | 項番
      - | HTTP
        | ステータスコード
      - | 説明
      - | 適用条件
    * - | (1)
      - | 200
        | OK
      - | リクエストが成功した事を通知するHTTPステータスコード。
      - | リクエストが成功した結果として、レスポンスのエンティティボディに、リクエストに対応するリソースの情報を出力する際に応答する。
    * - | (2)
      - | 201
        | Created
      - | 新しいリソースを作成した事を通知するHTTPステータスコード。
      - | POSTメソッドを使用して、新しいリソースを作成した際に使用する。
        | レスポンスのLocationヘッダに、作成したリソースのURIを設定する。
    * - | (3)
      - | 204
        | No Content
      - | リクエストが成功した事を通知するHTTPステータスコード。
      - | リクエストが成功した結果として、レスポンスのエンティティボディに、リクエストに対応するリソースの情報を出力しない時に応答する。

 .. tip::
 
    \ ``"200 OK``\ と \ ``"204 No Content"``\の違いは、レスポンスボディにリソースの情報を出力する/しないの違いとなる。

|

.. _RESTHowToDesignHttpStatusCodeForClientError:

リクエストが失敗した原因がクライアント側にある場合のHTTPステータスコード
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
リクエストが失敗した原因がクライアント側にある場合は、状況に応じて以下のHTTPステータスコードを応答する。

リソースを扱う個々のREST APIで意識する必要があるステータスコードは以下の通り。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.30\linewidth}|p{0.40\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 30 40

    * - | 項番
      - | HTTP
        | ステータスコード
      - | 説明
      - | 適用条件
    * - | (1)
      - | 400
        | Bad Request
      - | リクエストの構文やリクエストされた値が間違っている事を通知するHTTPステータスコード。
      - | エンティティボディに指定されたJSONやXMLの形式不備を検出した場合や、JSONやXML又はリクエストパラメータに指定された入力値の不備を検出した場合に応答する。
    * - | (2)
      - | 404
        | Not Found
      - | 指定されたリソースが存在しない事を通知するHTTPステータスコード。
      - | 指定されたURIに対応するリソースがシステム内に存在しない場合に応答する。
    * - | (3)
      - | 409
        | Conflict
      - | リクエストされた内容でリソースの状態を変更すると、リソースの状態に矛盾が発生ため処理を中止した事を通知するHTTPステータスコード。
      - | 排他エラーが発生した場合や業務エラーを検知した場合に応答する。
        | エンティティボディには矛盾の内容や矛盾を解決するために必要なエラー内容を出力する必要がある。

|

| リソースを扱う個々のREST APIで意識する必要がないステータスコードは以下の通り。
| 以下のステータスコードは、フレームワークや共通処理として意識する必要がある。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.30\linewidth}|p{0.40\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 30 40

    * - | 項番
      - | HTTP
        | ステータスコード
      - | 説明
      - | 適用条件
    * - | (4)
      - | 405
        | Method Not Allowed
      - | 使用されたHTTPメソッドが、指定されたリソースでサポートしていない事を通知するHTTPステータスコード。
      - | サポートされていないHTTPメソッドが使用された事を検知した場合に応答する。
        | レスポンスのAllowヘッダに、許可されているメソッドの列挙を設定する。
    * - | (5)
      - | 406
        | Not Acceptable
      - | 指定された形式でリソースの状態を応答する事が出来ないため、リクエストを受理できない事を通知するHTTPステータスコード。
      - | レスポンス形式として、拡張子又はAcceptヘッダで指定された形式をサポートしていない場合に応答する。
    * - | (6)
      - | 415
        | Unsupported Media Type
      - | エンティティボディに指定された形式をサポートしていないため、リクエストが受け取れない事を通知するHTTPステータスコード。
      - | リクエスト形式として、Content-Typeヘッダで指定された形式をサポートしていない場合に応答する。

|

.. _RESTHowToDesignHttpStatusCodeForServerError:

リクエストが失敗した原因がサーバ側にある場合のHTTPステータスコード
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
リクエストが失敗した原因がサーバ側にある場合は、状況に応じて以下のHTTPステータスコードを応答する。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.30\linewidth}|p{0.40\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 30 40

    * - | 項番
      - | HTTP
        | ステータスコード
      - | 説明
      - | 適用条件
    * - | (1)
      - | 500
        | Internal Server Error
      - | サーバ内部でエラーが発生した事を通知するHTTPステータスコード。
      - | サーバ内で予期しないエラーが発生した場合や、正常稼働時には発生してはいけない状態を検知した場合に応答する。

|

認証・認可
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo:: **TBD**

    認証及び認可制御をどのような指針で行うかについて記載する。
    
    OAuth2の仕組みを使って認証・認可を行う仕組みについて、次版以降に記載する予定である。

|

リソースの条件付き更新の制御
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo:: **TBD**

    HTTPヘッダを使ったリソースの条件付き更新(排他制御)をどのように行うか記載する。
    
    Etag/Last-Modified-Sinceなどのヘッダを使って条件付き更新の仕組みについて、次版以降に記載する予定である。

|

リソースの条件付き取得の制御
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo:: **TBD**

    HTTPヘッダを使ったリソースの条件付き取得(304 Not Modified制御)をどのように行うか記載する。

    Etag/Last-Modifiedなどのヘッダを使ったリソースの条件付き取得の仕組みについて、次版以降に記載する予定である。

|

リソースのキャッシュ制御
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo:: **TBD**

    HTTPヘッダを使ったリソースのキャッシュ制御をどのように行うか記載する。
    
    Cache-Control/Pragma/Expiresなどのヘッダを使ったリソースのキャッシュ制御の仕組みについて、次版以降に記載する予定である。

|

バージョニング
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo:: **TBD**

    RESTful Web Service自体のバージョン管理及び複数バージョンの並行稼働をどのように行うかについて、次版以降に記載する予定である。
    
|

.. _RESTHowToUse:

How to use
--------------------------------------------------------------------------------
本節では、RESTful Web Serviceの具体的な作成方法について説明する。

.. _RESTHowToUseWebApplicationConstruction:

Webアプリケーションの構成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| RESTful Web Serviceを構築する場合は、以下のいずれかの構成でWebアプリケーション(war)を構築する。
| **特に理由がない場合は、RESTful Web Service専用のWebアプリケーションとして構築する事を推奨する。**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 60

    * - 項番
      - 構成
      - 説明
    * - | (1)
      - | RESTful Web Service専用のWebアプリケーションとして構築する。
      - | RESTful Web Serviceを利用するクライアントアプリケーション(UI層のアプリケーション)との独立性を確保したい(する必要がある)場合は、RESTful Web Service専用のWebアプリケーション(war)として構築することを推奨する。
        |
        | RESTful Web Serviceを利用するクライアントアプリケーションが複数になる場合は、この方法でRESTful Web Serviceを生成することになる。
    * - | (2)
      - | RESTful Web Service用の\ ``DispatcherServlet``\を設けて構築する。
      - | RESTful Web Serviceを利用するクライアントアプリケーション(UI層のアプリケーション)との独立性を確保する必要がない場合は、RESTful Web Serviceとクライアントアプリケーションを一つのWebアプリケーション(war)として構築してもよい。
        |
        | ただし、RESTful Web Service用のリクエストを受ける\ ``DispatcherServlet``\と、クライアントアプリケーション用のリクエストを受け取る\ ``DispatcherServlet``\は分割して構築することを強く推奨する。

 .. note:: **クライアントアプリケーション(UI層のアプリケーション)とは**

    ここで言うクライアントアプリケーション(UI層のアプリケーション)とは、HTML, JavaScriptなどのスクリプト, CSS(Cascading Style Sheets)といったクライアント層(UI層)のコンポーネントを応答するアプリケーションの事をさす。
    JSPなどのテンプレートエンジンによって生成されるHTMLも対象となる。

 .. note:: **DispatcherServletを分割する事を推奨する理由**

    Spring MVCでは、\ ``DispatcherServlet``\毎にアプリケーションの動作設定を定義することになる。
    そのため、RESTful Web Serviceとクライアントアプリケーション(UI層のアプリケーション)のリクエストを同じ\ ``DispatcherServlet``\で受ける構成にしてしまうと、RESTful Web Service又はクライアントアプリケーション固有の動作設定を定義する事ができなくなったり、設定が煩雑又は複雑になることがある。
    
    本ガイドラインでは、上記の様な問題が起こらないようにするために、RESTful Web Serviceをクライアントアプリケーションを同じWebアプリケーション(war)として構築する場合は、\ ``DispatcherServlet``\を分割することを推奨している。

|

RESTful Web Service専用のWebアプリケーションとして構築する際の構成イメージは以下の通り。

 .. figure:: ./images_REST/RESTWebAppsConstitutionDivideWar.png
   :alt: Constitution of RESTful Web Service
   :width: 100%

|

RESTful Web Serviceとクライアントアプリケーションを一つのWebアプリケーションとして構築する際の構成イメージは以下の通り。

 .. figure:: ./images_REST/RESTWebAppsConstitutionDivideServlet.png
   :alt: Constitution of RESTful Web Service
   :width: 100%

|


pom.xmlの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
terasoluna-gfw-common-dependenciesを使用していれば、依存関係の設定は不要である。

.. Warning:: **Java SE 7環境にて使用する場合の設定**

   terasoluna-gfw-common-dependenciesはJava SE 8を前提とした依存関係を設定している。Java SE 7環境にて使用する場合は下記のようにJava SE 8依存ライブラリをexclusionすること。
   java SE 8依存ライブラリについてはアーキテクチャ概要の「\ :ref:`frameworkstack_using_oss_version` \」を参照

    .. code-block:: xml

       <dependency>
           <groupId>org.terasoluna.gfw</groupId>
           <artifactId>terasoluna-gfw-common-dependencies</artifactId>
           <exclusions>
               <exclusion>
                   <groupId>com.fasterxml.jackson.datatype</groupId>
                   <artifactId>jackson-datatype-jsr310</artifactId>
               </exclusion>
           </exclusions>
       </dependency>


.. _RESTHowToUseApplicationSettings:

アプリケーションの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
RESTful Web Service向けのアプリケーションの設定について説明する。

.. warning:: **StAX(Streaming API for XML)使用時のDOS攻撃対策について**

    XML形式のデータをStAXを使用して解析する場合は、DTDを使ったDOS攻撃を受けないように対応する必要がある。
    詳細は、\ `CVE-2015-3192 - DoS Attack with XML Input <http://pivotal.io/security/cve-2015-3192>`_\ を参照されたい。


.. _RESTHowToUseApplicationSettingsOfSpringMVC:

RESTful Web Serviceで必要となるSpring MVCのコンポーネントを有効化するための設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| RESTful Web Service用のbean定義ファイルを作成する。
| 以降の説明で示すサンプルを動かす際に必要となる定義を、以下に示す。

- :file:`spring-mvc-rest.xml`

 .. code-block:: xml
    :emphasize-lines: 22, 32-34, 39-41, 44-47, 51, 61, 65

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans" 
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xmlns:mvc="http://www.springframework.org/schema/mvc"
        xmlns:util="http://www.springframework.org/schema/util"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xsi:schemaLocation="
            http://www.springframework.org/schema/mvc
            http://www.springframework.org/schema/mvc/spring-mvc.xsd
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/util
            http://www.springframework.org/schema/util/spring-util.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd
            http://www.springframework.org/schema/aop
            http://www.springframework.org/schema/aop/spring-aop.xsd
    ">

        <!-- Load properties files for placeholder. -->
        <!-- (1) -->
        <context:property-placeholder 
            location="classpath*:/META-INF/spring/*.properties" />
    
        <bean id="jsonMessageConverter"
            class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
            <property name="objectMapper" ref="objectMapper" />
        </bean>
    
        <bean id="objectMapper" class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">
            <!-- (2) -->
            <property name="dateFormat">
                <bean class="com.fasterxml.jackson.databind.util.StdDateFormat" />
            </property>
        </bean>

        <!-- Register components of Spring MVC. -->
        <!-- (3) -->
         <mvc:annotation-driven>
            <mvc:message-converters register-defaults="false">
                <ref bean="jsonMessageConverter" />
            </mvc:message-converters>
            <!-- (4) -->
            <mvc:argument-resolvers>
                <bean class="org.springframework.data.web.PageableHandlerMethodArgumentResolver" />
            </mvc:argument-resolvers>
        </mvc:annotation-driven>
        
        <!-- Register components of interceptor. -->
        <!-- (5) -->
        <mvc:interceptors>
            <mvc:interceptor>
                <mvc:mapping path="/**" />
                <bean class="org.terasoluna.gfw.web.logging.TraceLoggingInterceptor" />
            </mvc:interceptor>
            <!-- omitted -->
        </mvc:interceptors>
    
        <!-- Scan & register components of RESTful Web Service. -->
        <!-- (6) -->
        <context:component-scan base-package="com.example.project.api" />

        <!-- Register components of AOP. -->
        <!-- (7) -->
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
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | アプリケーション層のコンポーネントでプロパティファイルに定義されている値を参照する必要がある場合は、\ ``<context:property-placeholder>``\要素を使用してプロパティファイルを読み込む必要がある。
        | プロパティファイルから値を取得する方法の詳細ついては、「:doc:`../GeneralFuncDetail/PropertyManagement`」を参照されたい。
    * - | (2)
      - | JSONの日付フィールドの形式をISO-8601の拡張形式として扱うための設定を追加する。
        | なお、リソースを表現するJavaBean(Resourceクラス)のプロパティとしてJoda Timeのクラスを使用する場合は、「\ :ref:`RESTAppendixUsingJSR310_JodaTime`\ 」を行う必要がある。
    * - | (3)
      - | RESTful Web Serviceを提供するために必要となるSpring MVCのフレームワークコンポーネントをbean登録する。
        | 本設定を行うことで、リソースのフォーマットとしてJSONを使用する事ができる。
        | 上記例では、\ ``<mvc:message-converters``>\要素のregister-defaults属性を\ ``false``\にしているので、リソースの形式はJSONに限定される。
        |
        | リソースのフォーマットとしてXMLを使用する場合は、XXE Injection対策が行われているXML用の\ ``MessageConverter``\を指定すること。指定方法は、「\ :ref:`RESTAppendixEnabledXXEInjectProtection`\」を参照されたい。
    * - | (4)
      - | ページ検索機能を有効にするための設定を追加する。
        | ページ検索の詳細については、「:doc:`../WebApplicationDetail/Pagination`」を参照されたい。
        | ページ検索が必要ない場合は、本設定は不要であるが、定義があっても問題はない。
    * - | (5)
      - | Spring MVCのインターセプタをbean登録する。
        | 上記例では、共通ライブラリから提供されている\ ``TraceLoggingInterceptor``\のみを定義しているが、データアクセスとしてJPAを使う場合は、別途\ ``OpenEntityManagerInViewInterceptor``\の設定を追加する必要がある。
        | \ ``OpenEntityManagerInViewInterceptor``\については、「\ :doc:`../DataAccessDetail/DataAccessJpa`\」を参照されたい。
    * - | (6)
      - | RESTful Web Service用のアプリケーション層のコンポーネント(ControllerやHelperクラスなど)をスキャンしてbean登録する。
        | \ ``"com.example.project.api"``\ の部分は\ **プロジェクト毎のパッケージ名となる。**\
    * - | (7)
      - | Spring MVCのフレームワークでハンドリングされた例外を、ログ出力するためのAOP定義を指定する。
        | \ ``HandlerExceptionResolverLoggingInterceptor``\については、「\ :doc:`../WebApplicationDetail/ExceptionHandling`\」を参照されたい。

 .. raw:: latex

    \newpage

.. note:: **ObjectMapperのBean定義方法について**

    Jacksonの\ ``com.fasterxml.jackson.databind.ObjectMapper``\ のBean定義を行う場合は、
    Springが提供している\ ``Jackson2ObjectMapperFactoryBean``\ を使用するとよい。
    \ ``Jackson2ObjectMapperFactoryBean``\ を使用すると、JSR-310 Date and Time APIやJoda Time用の拡張モジュールを自動登録することができ、
    さらにXMLのBean定義ファイル上で表現が難しかった\ ``ObjectMapper``\ のコンフィギュレーションも簡単に行うことができる。

    なお、\ ``ObjectMapper``\ を直接Bean定義するスタイルから\ ``Jackson2ObjectMapperFactoryBean``\ を使用するスタイルに変更する場合は、
    以下のコンフィギュレーションに対するデフォルト値がJacksonのデフォルト値と異なる(無効化されている)点に注意すること。

    * `MapperFeature#DEFAULT_VIEW_INCLUSION <http://fasterxml.github.io/jackson-databind/javadoc/2.6/com/fasterxml/jackson/databind/MapperFeature.html?is-external=true#DEFAULT_VIEW_INCLUSION>`_\
    * `DeserializationFeature#FAIL_ON_UNKNOWN_PROPERTIES <http://fasterxml.github.io/jackson-databind/javadoc/2.6/com/fasterxml/jackson/databind/DeserializationFeature.html?is-external=true#FAIL_ON_UNKNOWN_PROPERTIES>`_\

    \ ``ObjectMapper``\の動作をJacksonのデフォルト動作にあわせたい場合は、\ ``featuresToEnable``\ プロパティを使用して上記のコンフィギュレーションを有効化する。

     .. code-block:: xml

        <bean id="objectMapper" class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">
            <!-- ... -->
            <property name="featuresToEnable">
                <array>
                    <util:constant static-field="com.fasterxml.jackson.databind.MapperFeature.DEFAULT_VIEW_INCLUSION"/>
                    <util:constant static-field="com.fasterxml.jackson.databind.DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES"/>
                </array>
            </property>
        </bean>

    \ ``Jackson2ObjectMapperFactoryBean``\ の詳細については、 `Jackson2ObjectMapperFactoryBeanのJavaDoc <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/http/converter/json/Jackson2ObjectMapperFactoryBean.html>`_\ を参照されたい。


.. _REST_note_changed_jackson_version:

.. note::

    **jackson version 1.x.x から jackson version 2.x.xへ変更する場合の注意点**
    
    * パッケージの変更

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90

        * - verision
          - package
        * - | 1.x.x
          - | `org.codehaus.jackson`
        * - | 2.x.x
          - | `com.fasterxml.jackson`

     * 注意事項として、配下のパッケージ構成も変更されている。

    * Deprecated一覧

     * http://fasterxml.github.io/jackson-core/javadoc/2.6/deprecated-list.html
     * http://fasterxml.github.io/jackson-databind/javadoc/2.6/deprecated-list.html
     * http://fasterxml.github.io/jackson-annotations/javadoc/2.6/deprecated-list.html

|

.. _RESTHowToUseApplicationSettingsOfDispatcherServlet:

RESTful Web Service用のサーブレットの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 下記の設定は、RESTful Web Serviceとクライアントアプリケーションを別のWebアプリケーションとして構築する場合の設定例となっている。
| RESTful Web Serviceとクライアントアプリケーションを同じWebアプリケーションとして構築する場合は、 「\ :ref:`RESTAppendixSettingsOfDeployInSameWarFileRestAndClientApplication`\」を行う必要がある。

- :file:`web.xml`

 .. code-block:: xml
    :emphasize-lines: 4-5,9-10,14-18

    <!-- omitted -->

    <servlet>
        <!-- (1) -->
        <servlet-name>restAppServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <!-- (2) -->
            <param-value>classpath*:META-INF/spring/spring-mvc-rest.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <!-- (3) -->
    <servlet-mapping>
        <servlet-name>restAppServlet</servlet-name>
        <url-pattern>/api/v1/*</url-pattern>
    </servlet-mapping>

    <!-- omitted -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``<servlet-name>``\要素に、RESTful Web Service用のサーブレットであることを示す名前を指定する。
        | 上記例では、サーブレット名として\ ``"restAppServlet"``\を指定している。
    * - | (2)
      - | RESTful Web Service用の\ ``DispatcherServlet``\を構築する際に使用するSpring MVCのbean定義ファイルを指定する。
        | 上記例では、Spring MVCのbean定義ファイルとして、クラスパス上にある\ :file:`META-INF/spring/spring-mvc-rest.xml`\を指定している。
    * - | (3)
      - | RESTful Web Service用の\ ``DispatcherServlet``\へマッピングするサーブレットパスのパターンの指定を行う。
        | 上記例では、\ ``"/api/v1/"``\配下のサーブレットパスをRESTful Web Service用の\ ``DispatcherServlet``\にマッピングしている。
        | 具体的には、
        |   \ ``"/api/v1/"``\
        |   \ ``"/api/v1/members"``\
        |   \ ``"/api/v1/members/xxxxx"``\
        | といったサーブレットパスが、RESTful Web Service用の\ ``DispatcherServlet``\(\ ``"restAppServlet"``\)にマッピングされる。

 .. tip:: **@RequestMappingアノテーションのvalue属性に指定する値について**

   \ ``@RequestMapping``\アノテーションのvalue属性に指定する値は、\ ``<url-pattern>``\要素で指定したワイルドカード(\ ``*``\)の部分の値を指定する。
   
   例えば、\ ``@RequestMapping(value = "members")``\と指定した場合、\ ``"/api/v1/members"``\といパスに対する処理を行うメソッドとしてデプロイされる。
   そのため、\ ``@RequestMapping``\アノテーションのvalue属性には、分割したサーブレットへマッピングするためパス(\ ``"api/v1"``\)を指定する必要はない。
   
   \ ``@RequestMapping(value = "api/v1/members")``\と指定すると、\ ``"/api/v1/api/v1/members"``\というパスに対する処理を行うメソッドとしてデプロイされてしまうので、注意すること。

|

.. _RESTHowToUseApiImplementation:

REST APIの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| REST APIの実装方法について説明する。
| 以降の説明では、ショッピングサイトの会員情報(Memberリソース)に対するREST APIの実装例を使用して、説明を行う。

 .. note::

    本節では、ドメイン層の実装の説明は行わないが、「\ :ref:`RESTAppendixSoruceCodesOfDomainLayer`\」として、添付しておく。

    必要に応じて、参照されたい。

|

まず、説明で使用するREST APIの仕様を以下に示す。

|

**リソースの形式**

 | 会員情報のリソースの形式は、以下のようなJSON形式とする。
 | 下記の例では、全フィールドを表示しているが、全てのAPIのリクエストとレスポンスで使用するわけではない。
 | 例えば、\ ``"password"``\はリクエストのみで使用、\ ``"createdAt"``\や\ ``"lastModifiedAt"``\はレスポンスのみ使用などの違いがある。

 .. code-block:: json

    {
        "memberId" : "M000000001",
        "firstName" : "Firstname",
        "lastName" : "Lastname",
        "genderCode" : "1",
        "dateOfBirth" : "1977-03-13",
        "emailAddress" : "user1@test.com",
        "telephoneNumber" : "09012345678",
        "zipCode" : "1710051",
        "address" : "Tokyo",
        "credential" : {
            "signId" : "user1@test.com",
            "password" : "zaq12wsx",
            "passwordLastChangedAt" : "2014-03-13T04:39:14.831Z",
            "lastModifiedAt" : "2014-03-13T04:39:14.831Z"
        },
        "createdAt" : "2014-03-13T04:39:14.831Z",
        "lastModifiedAt" : "2014-03-13T04:39:14.831Z"
    }

 .. note::
 
     本節では、関連リソースへのハイパーメディアリンクは設けない例となっている。
     ハイパーメディアリンクを設ける場合の実装例は、「:ref:`RESTAppendixHyperMediaLink`」を参照されたい。

|

**リソースの項目仕様**
 
 リソース(JSON)の項目毎の仕様は以下の通りとする。
 
 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.10\linewidth}|p{0.10\linewidth}|p{0.15\linewidth}|p{0.25\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 10 10 15 25
    :class: longtable

    * - 項番
      - 項目名
      - 型
      - I/O仕様
      - 桁数
        (min-max)
      - その他の仕様
    * - | (1)
      - memberId
      - String
      - I/O
      - 10-10
      - POST Membersのリクエスト時は未指定(NULL)であること。
    * - | (2)
      - firstName
      - String
      - I/O
      - 1-128
      - \-
    * - | (3)
      - lastName
      - String
      - I/O
      - 1-128
      - \-
    * - | (4)
      - genderCode
      - | String
        | (Code)
      - I/O
      - 1-1
      - | ``"0"`` : UNKNOWN
        | ``"1"`` : MEN
        | ``"2"`` : WOMEN
    * - | (5)
      - dateOfBirth
      - Date
      - I/O
      - \-
      - | yyyy-MM-dd形式
        | (ISO-8601拡張形式)
    * - | (6)
      - emailAddress
      - | String
        | (E-mail)
      - I/O
      - 1-256
      - \-
    * - | (7)
      - telephoneNumber
      - String
      - I/O
      - 0-20
      - \-
    * - | (8)
      - zipCode
      - String
      - I/O
      - 0-20
      - \-
    * - | (9)
      - address
      - String
      - I/O
      - 0-256
      - \-
    * - | (10)
      - credential
      - | Object
        | (MemberCredential)
      - I/O
      - \-
      - POST Membersのリクエスト時は指定されていること。
    * - | (11)
      - credential/signId
      - | String
        | (E-mail)
      - I/O
      - 0-256
      - 指定がない場合は、emailAddressの値を適用する。
    * - | (12)
      - | credential/
        | password
      - String
      - I
      - 8-32
      - \-
    * - | (13)
      - | credential/
        | passwordLastChangedAt
      - | Timestamp
        | with time-zone
      - O
      - \-
      - | yyyy-MM-dd'T'HH:mm:ss.SSS'Z'形式
        | (ISO-8601拡張形式)
    * - | (14)
      - | credential/
        | lastModifiedAt
      - | Timestamp
        | with time-zone
      - O
      - \-
      - | yyyy-MM-dd'T'HH:mm:ss.SSS'Z'形式
        | (ISO-8601拡張形式)
    * - | (15)
      - createdAt
      - | Timestamp
        | with time-zone
      - O
      - \-
      - | yyyy-MM-dd'T'HH:mm:ss.SSS'Z'形式
        | (ISO-8601拡張形式)
    * - | (16)
      - lastModifiedAt
      - | Timestamp
        | with time-zone
      - O
      - \-
      - | yyyy-MM-dd'T'HH:mm:ss.SSS'Z'形式
        | (ISO-8601拡張形式)

 .. raw:: latex

    \newpage

|

**REST API一覧**

 実装するREST APIは以下の5つのAPIとする。
 
 .. tabularcolumns:: |p{0.05\linewidth}|p{0.15\linewidth}|p{0.10\linewidth}|p{0.25\linewidth}|p{0.15\linewidth}|p{0.25\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 15 10 25 15 25

    * - | 項番
      - | API名
      - | HTTP
        | メソッド
      - | リソースパス
      - | ステータス
        | コード
      - | API概要
    * - | (1)
      - :ref:`GET Members <RESTHowToUseApiImplementationOfGetCollection>`
      - GET
      - \ ``/api/v1/members``\ 
      - | 200
        | (OK)
      - | 条件に一致するMemberリソースをページ検索する。
    * - | (2)
      - :ref:`POST Members <RESTHowToUseApiImplementationOfPostCollection>`
      - POST
      - \ ``/api/v1/members``\ 
      - | 201
        | (Created)
      - Memberリソースを一件作成する。
    * - | (3)
      - :ref:`GET Member <RESTHowToUseApiImplementationOfGetSpecifiedResource>`
      - GET
      - \ ``/api/v1/members/{memberId}``\ 
      - | 200
        | (OK)
      - Memberリソースの一件取得する。
    * - | (4)
      - :ref:`PUT Member <RESTHowToUseApiImplementationOfPutSpecifiedResource>`
      - PUT
      - \ ``/api/v1/members/{memberId}``\ 
      - | 200
        | (OK)
      - Memberリソースを一件更新する。
    * - | (5)
      - :ref:`DELETE Member <RESTHowToUseApiImplementationOfDeleteSpecifiedResource>`
      - DELETE
      - \ ``/api/v1/members/{memberId}``\ 
      - | 204
        | (No Content)
      - Memberリソースを一件削除する。

 .. note::
 
     本節では、リソースのCRUD操作の説明に注力するため、HEADとOPTIONSメソッドの説明は行わない。
     HTTPの仕様に準拠したRESTful Web Serviceを作成する場合は、「:ref:`RESTAppendixRestApiOfHTTPCompliance`」を参照されたい。

|

.. _RESTHowToUsePackage:

REST API用パッケージの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
REST API用のクラスを格納するパッケージを作成する。

| REST API用のクラスを格納するルートパッケージのパッケージ名は\ ``api``\として、配下にリソース毎のパッケージ(リソース名の小文字)を作成する事を推奨する。
| 説明で扱うリソース名は\ ``Member``\なので、\ ``org.terasoluna.examples.rest.api.member``\ というパッケージとする。

 .. note::

    作成したパッケージに格納するクラスは、通常以下の4種類となる。
    作成するクラスのクラス名は、以下のネーミングルールとする事を推奨する。

     * \ ``[リソース名]Resource``\ 
     * \ ``[リソース名]RestController``\ 
     * \ ``[リソース名]Validator``\  (必要に応じて作成する)
     * \ ``[リソース名]Helper``\  (必要に応じて作成する)

    説明で扱うリソースのリソース名は\ ``Member``\なので、

     * \ ``MemberResource``\ 
     * \ ``MemberRestController``\ 
     * \ ``MemberValidator``\ 
     * \ ``MemberHelper``\ 

    となる。

    関連リソースを扱う場合、関連リソース用のクラスも同じパッケージに配置すればよい。

|

| REST API用の共通部品を格納するパッケージは、REST API用のクラスを格納するルートパッケージ直下に\ ``common``\という名前で作成し、サブパッケージは機能単位に作成する事を推奨する。
| 例えば、エラーハンドリングを行う共通部品を格納するサブパッケージの場合、\ ``error``\という名前でサブパッケージを作成する。
| 以降の説明で作成する例外ハンドリング用のクラスは、\ ``org.terasoluna.examples.rest.api.common.error``\というパッケージに格納している。

 .. note::

    共通部品が格納されているパッケージという事がわかれば、パッケージ名は\ ``common``\以外でもよい。

| 


.. _RESTHowToUseResourceClass:

Resourceクラスの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 本ガイドラインでは、Web上に公開するリソースを表現(JSONやXMLを表現)するクラスとして、Resourceクラスを設けることを推奨する。

 .. note:: **Resourceクラスを作成する理由**

  DomainObjectクラス(例えばEntityクラス)があるにも関わらず、Resourceクラスを作成する理由は、
  クライアントとの入出力で使用するユーザーインタフェース(UI)上の情報と業務処理で扱う情報は必ずしも一致しないためである。
  
  これらを混同して使用すると、アプリケーション層の影響がドメイン層におよび、保守性を低下させる原因となる。
  DomainObjectとResourceクラスは別々に作成し、Dozer等のBeanMapperを利用してデータ変換を行うことを推奨する。

|

Resourceクラスの役割は以下の通りである。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 60

    * - 項番
      - 役割
      - 説明
    * - | (1)
      - | リソースのデータ構造の定義を行う。
      - | Web上に公開するリソースのデータ構造を定義する。
        | データベースなどの永続層で管理しているデータの構造のままWeb上のリソースとして公開する事は、一般的には稀である。
    * - | (2)
      - | フォーマットに関する定義を行う。
      - | リソースのフォーマットに関する定義を、アノテーションを使って指定する。
        | 使用するアノテーションは、リソースの形式(JSON/XMLなど)よって異なり、JSON形式の場合はJacksonのアノテーション、XML形式の場合はJAXBのアノテーションを使用する事になる。
    * - | (3)
      - | 入力チェックルールの定義を行う。
      - | 項目毎の単項目の入力チェックルールを、Bean Validationのアノテーションを使って指定する。
        | 入力チェックの詳細については、「\ :doc:`../WebApplicationDetail/Validation`\」を参照されたい。


 .. warning:: **循環参照への対策**

     Resourceクラス(JavaBean)をJSONやXML形式にシリアライズする際に、相互参照関係のオブジェクトをプロパティに保持していると、
     循環参照となり\ ``StackOverflowError``\ や\ ``OutOfMemoryError``\ などが発生するので、注意が必要である。

     循環参照を回避するためには、

     * Jacksonを使用してJSON形式にシリアライズする場合は、シリアライズ対象から除外するプロパティに\ ``@com.fasterxml.jackson.annotation.JsonIgnore``\ アノテーション
     * JAXBを使用してXML形式にシリアライズする場合は、シリアライズ対象から除外するプロパティに\ ``javax.xml.bind.annotation.XmlTransient``\ アノテーション

     を付与すればよい。

     以下にJacksonを使用してJSON形式にシリアライズする際の回避例を示す。

      .. code-block:: java

          public class Order {
              private String orderId;
              private List<OrderLine> orderLines;
              // ...
          }

      .. code-block:: java

          public class OrderLine {
              @JsonIgnore
              private Order order;
              private String itemCode;
              private int quantity;
              // ...
          }

      .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
      .. list-table::
          :header-rows: 1
          :widths: 10 90
          :class: longtable

          * - 項番
            - 説明
          * - | (1)
            - シリアライズ対象から除外するプロパティに対して\ ``@JsonIgnore``\ アノテーションを付与する。

|

以下にResourceクラスの作成例を示す。

* :file:`MemberResource.java`

 .. code-block:: java
    :emphasize-lines: 18, 23-28, 68

    package org.terasoluna.examples.rest.api.member;
    
    import java.io.Serializable;
    
    import javax.validation.Valid;
    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Null;
    import javax.validation.constraints.Past;
    import javax.validation.constraints.Size;
    
    import org.hibernate.validator.constraints.Email;
    import org.hibernate.validator.constraints.NotEmpty;
    import org.joda.time.DateTime;
    import org.joda.time.LocalDate;
    import org.springframework.format.annotation.DateTimeFormat;
    import org.terasoluna.gfw.common.codelist.ExistInCodeList;
    
    // (1)
    public class MemberResource implements Serializable {
    
        private static final long serialVersionUID = 1L;
    
        // (2)
        interface PostMembers {
        }
    
        interface PutMember {
        }
    
        @Null(groups = PostMembers.class)
        @NotEmpty(groups = PutMember.class)
        @Size(min = 10, max = 10, groups = PutMember.class)
        private String memberId;
    
        @NotEmpty
        @Size(max = 128)
        private String firstName;
    
        @NotEmpty
        @Size(max = 128)
        private String lastName;
    
        @NotEmpty
        @ExistInCodeList(codeListId = "CL_GENDER")
        private String genderCode;
    
        @NotNull
        @Past
        private LocalDate dateOfBirth;
    
        @NotEmpty
        @Size(max = 256)
        @Email
        private String emailAddress;
    
        @Size(max = 20)
        private String telephoneNumber;
    
        @Size(max = 20)
        private String zipCode;
    
        @Size(max = 256)
        private String address;
    
        @NotNull(groups = PostMembers.class)
        @Null(groups = PutMember.class)
        @Valid
        // (3)
        private MemberCredentialResource credential;
    
        @Null
        private DateTime createdAt;
    
        @Null
        private DateTime lastModifiedAt;
    
        // omitted setter and getter
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | Memberリソースを表現するJavaBean。
    * - | (2)
      - | Bean Validationのバリデーショングループを指定するためのインタフェースを定義している。
        | 実装例では、POSTとPUTで異なる入力チェックを行うため、バリデーションをグループ化して入力チェックを行っている。
        | バリデーションのグループ化については、「:doc:`../WebApplicationDetail/Validation`」を参照されたい。
    * - | (3)
      - | 関連リソースをネストしたJavaBeanをフィールドに定義している。
        | 実装例では、会員の資格情報(サインIDとパスワード)を関連リソースとして扱っている。
        | これは、サインIDの変更やパスワードの変更のみ行うという操作を考慮して、関連リソースとして切り出している。
        | ただし、関連リソースに対するREST APIの実装例については割愛している。

* :file:`MemberCredentialResource.java`

 .. code-block:: java
    :emphasize-lines: 13, 22

    package org.terasoluna.examples.rest.api.member;
    
    import java.io.Serializable;
    
    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Null;
    import javax.validation.constraints.Size;
    
    import com.fasterxml.jackson.annotation.JsonInclude;
    import org.hibernate.validator.constraints.Email;
    import org.joda.time.DateTime;

    // (4)
    public class MemberCredentialResource implements Serializable {
    
        private static final long serialVersionUID = 1L;
    
        @Size(max = 256)
        @Email
        private String signId;

        // (5)
        @JsonInclude(JsonInclude.Include.NON_NULL)
        @NotNull
        @Size(min = 8, max = 32)
        private String password;
    
        @Null
        private DateTime passwordLastChangedAt;
    
        @Null
        private DateTime lastModifiedAt;
    
        // omitted setter and getter
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (4)
      - | Memberリソースの関連リソースとなるCredentialリソースを表現するJavaBean。
    * - | (5)
      - | 値が\ ``null``\の時に、JSONにフィールド自体を出力しないようにするためのアノテーションを指定している。
        | これは、レスポンスするJSONの中にパスワードのフィールド出力しないようにするために行っている。
        | 上記例ではNULLの場合(\ ``Inclusion.NON_NULL``\)に限っているが、値が空の場合(\ ``Inclusion.NON_EMPTY``\)という指定も可能である。

|

* | Beanのマッピング定義の追加
  | これから説明する実装例では、EntityクラスとResourceクラスのコピーは、「\ :doc:`../GeneralFuncDetail/Dozer`\」を使って行う。
  | 上記に示したJavaBeanには、Joda-Timeのクラスである\ ``org.joda.time.DateTime``\と\ ``org.joda.time.LocalDate``\が含まれているが、「\ :doc:`../GeneralFuncDetail/Dozer`\」を使ってコピーするとJoda-Timeのオブジェクトは正しくコピーされない。
  | そのため、正しくコピーされるようにするためには、「:ref:`RESTAppendixCopyJodaObjectByBeanConvert`」を適用する必要がある。

|

.. _RESTHowToUseControllerClass:

Controllerクラスの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Controllerクラスはリソース毎に作成する。
| 全てのAPIの実装が完了した際のソースコードについては、\ :ref:`Appendix <RESTAppendixSoruceCodesOfMemberRestController>`\を参照されたい。

 .. code-block:: java
    :emphasize-lines: 7-8

    package org.terasoluna.examples.rest.api.member;
    
    // omitted
    import org.springframework.web.bind.annotation.RestController;
    // omitted

    @RequestMapping("members") // (1)
    @RestController // (2)
    public class MemberRestController {

        // omitted ...

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | Controllerに対して、リソースのコレクション用のURI(サーブレットパス)をマッピングする。
        | 具体的には、\ ``@RequestMapping``\アノテーションのvalue属性に、リソースのコレクションを表すサーブレットパスを指定する。
        | 上記例では、 \ ``/api/v1/members``\ というサーブレットパスをマッピングしている。
    * - | (2)
      - Controllerに対して、\ ``@RestController``\ アノテーションを付与する。

        \ ``@RestController``\ アノテーションを付与することで、

        * クラスに\ ``org.springframework.stereotype.Controller``\ アノテーションを付与
        * 以降で説明するControllerのメソッドに\ ``@org.springframework.web.bind.annotation.ResponseBody``\ アノテーションを付与

        したのと同じ意味となる。

        Controllerのメソッドに\ ``@ResponseBody``\ を付与することで、返却したResourceオブジェクトがJSONやXMLにmarshalされ、レスポンスBODYに設定される。

 .. tip::

    \ ``@RestController``\ アノテーションは、Spring Framework 4.0 から追加されたアノテーションである。

    \ ``@RestController``\ アノテーションの登場により、Controllerの各メソッドに\ ``@ResponseBody``\ アノテーションを付与する必要がなくなったため、
    REST API用のControllerをよりシンプルに作成出来るようになった。
    \ ``@RestController``\ アノテーションの詳細については、\ `こちら <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/web/bind/annotation/RestController.html>`_\ を参照されたい。

    従来通り\ ``@Controller``\ アノテーションと\ ``@ResponseBody``\ アノテーションを組み合わせてREST API用のControllerを作成する例を以下に示す。

     .. code-block:: java

        @RequestMapping("members")
        @Controller
        public class MemberRestController {

            @RequestMapping(method = RequestMethod.GET)
            @ResponseStatus(HttpStatus.OK)
            @ResponseBody
            public Page<MemberResource> getMembers() {
                // ...
            }

            // ...

        }

|

.. _RESTHowToUseApiImplementationOfGetCollection:

リソースのコレクションを取得するREST APIの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
URIで指定されたMemberリソースのコレクションをページ検索するREST APIの実装例を、以下に示す。

- | 検索条件を受け取るためのJavaBeanの作成
  | リソースのコレクションを取得する際に、検索条件が必要な場合は、検索条件を受け取るためのJavaBeanの作成する。

 .. code-block:: java
    :emphasize-lines: 1, 5

    // (1)
    public class MembersSearchQuery implements Serializable {
        private static final long serialVersionUID = 1L;
    
        // (2)
        @NotEmpty
        private String name;
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 検索条件を受け取るためのJavaBeanを作成する
        | 検索条件が不要な場合は、JavaBeanの作成は不要である。
    * - | (2)
      - | プロパティ名は、リクエストパラメータのパラメータ名と一致させる。
        | 上記例では、\ ``/api/v1/members?name=John``\ というリクエストの場合、JavaBeanのnameプロパティに \ ``"John"``\ という値が設定される。

|

* | REST APIの実装
  | Memberリソースのコレクションをページ検索する処理を実装する。
  
 .. code-block:: java
    :emphasize-lines: 13, 15, 18, 20, 23, 26, 34

    @RequestMapping("members")
    @RestController
    public class MemberRestController {
    
        // omitted

        @Inject
        MemberService memberService;
    
        @Inject
        Mapper beanMapper;
    
        // (3)
        @RequestMapping(method = RequestMethod.GET)
        // (4)
        @ResponseStatus(HttpStatus.OK)
        public Page<MemberResource> getMembers(
                // (5)
                @Validated MembersSearchQuery query,
                // (6)
                Pageable pageable) {
    
            // (7)
            Page<Member> page = memberService.searchMembers(query.getName(), pageable);
    
            // (8)
            List<MemberResource> memberResources = new ArrayList<>();
            for (Member member : page.getContent()) {
                memberResources.add(beanMapper.map(member, MemberResource.class));
            }
            Page<MemberResource> responseResource = new PageImpl<>(memberResources, 
                    pageable, page.getTotalElements());
    
            // (9)
            return responseResource;
        }

        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (3)
      - | \ ``@RequestMapping``\アノテーションのmethod属性に、\ ``RequestMethod.GET``\を指定する。
    * - | (4)
      - | メソッドアノテーションとして、\ ``@org.springframework.web.bind.annotation.ResponseStatus``\アノテーションを付与し、応答するステータスコードを指定する。
        | \ ``@ResponseStatus``\アノテーションのvalue属性には、200(OK)を設定する。
        
        .. tip:: **ステータスコードの指定方法について**
        
            本例では、\ ``@ResponseStatus``\アノテーションを使って応答するステータスコードを固定で指定しているが、Controllerのロジック内で指定する事もできる。

             .. code-block:: java
            
                public ResponseEntity<Page<MemberResource>> getMembers(
                        @Validated MembersSearchQuery query,
                        Pageable pageable) {
                
                    // omitted
                    
                    return ResponseEntity.ok().body(responseResource);
                }

            応答するステータスコードを処理内容や処理結果に応じて変える必要がある場合は、上記実装例の様に、\ ``org.springframework.http.ResponseEntity``\を使用する事になる。


    * - | (5)
      - | 検索条件を受け取るためのJavaBeanを引数に指定する。
        | 入力チェックが必要な場合は、引数アノテーションとして、\ ``@Validated``\アノテーションを付与する。入力チェックの詳細については、「\ :doc:`../WebApplicationDetail/Validation`\」を参照されたい。
    * - | (6)
      - | ページ検索が必要な場合は、\ ``org.springframework.data.domain.Pageable``\を引数に指定する。
        | ページ検索の詳細については、「:doc:`../WebApplicationDetail/Pagination`」を参照されたい。
    * - | (7)
      - | ドメイン層のServiceのメソッドを呼び出し、条件に一致するリソースの情報(Entityなど)を取得する。
        | ドメイン層の実装については、「:doc:`../../ImplementationAtEachLayer/DomainLayer`」を参照されたい。
    * - | (8)
      - | 条件に一致したリソースの情報(Entityなど)をもとに、Web上に公開する情報を保持するResourceオブジェクトを生成する。
        | ページ検索の結果を応答する際は、 \ ``org.springframework.data.domain.PageImpl``\クラスを使用することで、ページ検索時の応答として必要な項目をクライアントに返却する事ができる。
        |
        | 上記例では、Beanマッピングライブラリを使用してEntityからResourceオブジェクトを生成している。Beanマッピングライブラリについては、「\ :doc:`../GeneralFuncDetail/Dozer`\」を参照されたい。
        | **Resourceオブジェクトを生成するためのコード量が多くなる場合は、HelperクラスにResourceオブジェクトを生成するためのメソッドを作成することを推奨する。**
    * - | (9)
      - | (8)で生成したResourceオブジェクトを返却する。
        | ここで返却したオブジェクトがJSONやXMLにmarshalされ、レスポンスBODYに設定される。

 .. raw:: latex

    \newpage

 | \ ``PageImpl``\クラスを使用した時のレスポンスは以下の様になる。
 | ハイライトしている部分が、ページ検索固有の項目となる。
 
 .. code-block:: json
    :emphasize-lines: 37-50
    
    {
      "content" : [ {
        "memberId" : "M000000001",
        "firstName" : "John",
        "lastName" : "Smith",
        "genderCode" : "1",
        "dateOfBirth" : "1977-03-07",
        "emailAddress" : "john.smith@test.com",
        "telephoneNumber" : "09012345678",
        "zipCode" : "1710051",
        "address" : "Tokyo",
        "credential" : {
          "signId" : "john.smit@test.com",
          "passwordLastChangedAt" : "2014-03-13T10:18:08.003Z",
          "lastModifiedAt" : "2014-03-13T10:18:08.003Z"
        },
        "createdAt" : "2014-03-13T10:18:08.003Z",
        "lastModifiedAt" : "2014-03-13T10:18:08.003Z"
      }, {
        "memberId" : "M000000002",
        "firstName" : "Sophia",
        "lastName" : "Smith",
        "genderCode" : "2",
        "dateOfBirth" : "1977-03-07",
        "emailAddress" : "sophia.smith@test.com",
        "telephoneNumber" : "09012345678",
        "zipCode" : "1710051",
        "address" : "Tokyo",
        "credential" : {
          "signId" : "sophia.smith@test.com",
          "passwordLastChangedAt" : "2014-03-13T10:18:08.003Z",
          "lastModifiedAt" : "2014-03-13T10:18:08.003Z"
        },
        "createdAt" : "2014-03-13T10:18:08.003Z",
        "lastModifiedAt" : "2014-03-13T10:18:08.003Z"
      } ],
      "last" : false,
      "totalPages" : 13,
      "totalElements" : 25,
      "size" : 2,
      "number" : 1,
      "sort" : [ {
        "direction" : "DESC",
        "property" : "lastModifiedAt",
        "ignoreCase" : false,
        "nullHandling": "NATIVE",
        "ascending" : false
      } ],
      "numberOfElements" : 2,
      "first" : false
    }

 .. note:: **Spring Data CommonsのAPI仕様の変更に伴う注意点**

    terasoluna-gfw-common 5.0.0.RELEASE以上が依存するspring-data-commons(1.9.1.RELEASE以上)では、
    ページ検索機能用のインタフェース(\ ``org.springframework.data.domain.Page``\ )とクラス(\ ``org.springframework.data.domain.PageImpl``\ と\ ``org.springframework.data.domain.Sort.Order``\ )のAPI仕様が変更になっている。

    具体的には、

    * \ ``Page``\ インタフェースと\ ``PageImpl``\ クラスでは、\ ``isFirst()``\ と\ ``isLast()``\ メソッドがspring-data-commons 1.8.0.RELEASEで追加、\ ``isFirstPage()``\ と\ ``isLastPage()``\ メソッドがspring-data-commons 1.9.0.RELEASEで削除
    * \ ``Sort.Order``\ クラスでは、 \ ``nullHandling``\ プロパティがspring-data-commons 1.8.0.RELEASEで追加

    されている。

    REST APIのリソースオブジェクトとして\ ``Page``\ インタフェース(\ ``PageImpl``\ クラス)を使用している場合は、
    JSONやXMLのフォーマットが変わってしまうため、アプリケーションの修正が必要になるケースがある。

|

* | Beanのマッピング定義の追加
  | 上記実装例では、\ ``Member``\オブジェクトと\ ``MemberResource``\オブジェクトのコピーは、「\ :doc:`../GeneralFuncDetail/Dozer`\」を使って行っている。
  | 単純なフィールド値のコピーのみでよい場合は、Beanのマッピング定義の追加は不要だが、上記実装例では、\ ``Member``\オブジェクトの内容を\ ``MemberResource``\オブジェクトにコピーする際に、\ ``credential.password``\ をコピー対象外にする必要がある。
  | 特定のフィールドをコピー対象外にするためには、Beanのマッピング定義の追加が必要となる。

 .. code-block:: xml
    :emphasize-lines: 1, 10-14
  
    <!-- (11) -->
    <?xml version="1.0" encoding="UTF-8"?>
    <mappings xmlns="http://dozer.sourceforge.net" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://dozer.sourceforge.net
              http://dozer.sourceforge.net/schema/beanmapping.xsd">
    
        <mapping type="one-way">
            <class-a>org.terasoluna.examples.rest.domain.model.MemberCredential</class-a>
            <class-b>org.terasoluna.examples.rest.api.member.MemberCredentialResource</class-b>
            <!-- (12) -->
            <field-exclude>
                <a>password</a>
                <b>password</b>
            </field-exclude>
        </mapping>
    
    </mappings>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (11)
      - | \ ``Member``\オブジェクトと\ ``MemberResource``\オブジェクトのマッピングルールを定義するファイルを作成する。
        | Dozerのマッピング定義ファイルは、リソース毎に作成する事を推奨する。
        | 
        | 今回の実装例では、\ :file:`/xxx-web/src/main/resources/META-INF/dozer/memberResource-mapping.xml`\に格納する。
    * - | (12)
      - | 上記例では、\ ``Member``\の関連エンティティである\ ``MemberCredential``\の内容を、\ ``MemberResource``\の関連リソースである\ ``MemberCredentialResource``\にコピーする際に、\ ``password``\フィールドをコピー対象外に指定している。
        | Beanマッピングの定義方法の詳細については、「\ :doc:`../GeneralFuncDetail/Dozer`\」を参照されたい。

|

* リクエスト例

 .. code-block:: guess
    :emphasize-lines: 1

    GET /rest-api-web/api/v1/members?name=Smith&page=0&size=2 HTTP/1.1
    Accept: text/plain, application/json, application/*+json, */*
    User-Agent: Java/1.7.0_51
    Host: localhost:8080
    Connection: keep-alive

|

* レスポンス例

 .. code-block:: guess
    :emphasize-lines: 1

    HTTP/1.1 200 OK
    Server: Apache-Coyote/1.1
    X-Track: fb63a6d446f849feb8ccaa4c9a794333
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Thu, 13 Mar 2014 11:10:43 GMT
    
    {"content":[{"memberId":"M000000001","firstName":"John","lastName":"Smith","genderCode":"1","dateOfBirth":"2013-03-13","emailAddress":"user1394709042120@test.com","telephoneNumber":"09012345678","zipCode":"1710051","address":"Tokyo","credential":{"signId":"user1394709042120@test.com","passwordLastChangedAt":"2014-03-13T11:10:43.066Z","lastModifiedAt":"2014-03-13T11:10:43.066Z"},"createdAt":"2014-03-13T11:10:43.066Z","lastModifiedAt":"2014-03-13T11:10:43.066Z"},{"memberId":"M000000002","firstName":"Sophia","lastName":"Smith","genderCode":"2","dateOfBirth":"2013-03-13","emailAddress":"user1394709043663@test.com","telephoneNumber":"09012345678","zipCode":"1710051","address":"Tokyo","credential":{"signId":"user1394709043663@test.com","passwordLastChangedAt":"2014-03-13T11:10:43.678Z","lastModifiedAt":"2014-03-13T11:10:43.678Z"},"createdAt":"2014-03-13T11:10:43.678Z","lastModifiedAt":"2014-03-13T11:10:43.678Z"}],"last":true,"totalPages":1,"totalElements":2,"size":2,"number":0,"sort":null,"numberOfElements":2,"first":true}

|

 .. tip::
 
    ページ検索が不要な場合は、Resourceクラスのリストを直接扱えばよい。

    Resourceクラスのリストを直接扱う場合のControllerのメソッドは以下のような定義となる。

     .. code-block:: java
        :emphasize-lines: 3

        @RequestMapping(method = RequestMethod.GET)
        @ResponseStatus(HttpStatus.OK)
        public List<MemberResource> getMembers(
                @Validated MembersSearchQuery query) {
            // omitted
        }

    Resourceクラスのリストを直接扱った場合、以下のようなJSONとなる。
    
     .. code-block:: json

        [ {
            "memberId" : "M000000001",
            "firstName" : "John",
            "lastName" : "Smith",
            "genderCode" : "1",
            "dateOfBirth" : "1977-03-07",
            "emailAddress" : "john.smith@test.com",
            "telephoneNumber" : "09012345678",
            "zipCode" : "1710051",
            "address" : "Tokyo",
            "credential" : {
              "signId" : "john.smit@test.com",
              "passwordLastChangedAt" : "2014-03-13T10:18:08.003Z",
              "lastModifiedAt" : "2014-03-13T10:18:08.003Z"
            },
            "createdAt" : "2014-03-13T10:18:08.003Z",
            "lastModifiedAt" : "2014-03-13T10:18:08.003Z"
        }, {
            "memberId" : "M000000002",
            "firstName" : "Sophia",
            "lastName" : "Smith",
            "genderCode" : "2",
            "dateOfBirth" : "1977-03-07",
            "emailAddress" : "sophia.smith@test.com",
            "telephoneNumber" : "09012345678",
            "zipCode" : "1710051",
            "address" : "Tokyo",
            "credential" : {
              "signId" : "sophia.smith@test.com",
              "passwordLastChangedAt" : "2014-03-13T10:18:08.003Z",
              "lastModifiedAt" : "2014-03-13T10:18:08.003Z"
            },
            "createdAt" : "2014-03-13T10:18:08.003Z",
            "lastModifiedAt" : "2014-03-13T10:18:08.003Z"
        } ]


|

.. _RESTHowToUseApiImplementationOfPostCollection:

リソースをコレクションに追加するAPI RESTの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
指定されたMemberリソースを作成し、Memberリソースをコレクションに追加するREST APIの実装例を、以下に示す。

* | REST APIの実装
  | 指定されたMemberリソースを作成し、Memberリソースをコレクションに追加する処理を実装する。

 .. code-block:: java
    :emphasize-lines: 7, 9, 12, 16

    @RequestMapping("members")
    @RestController
    public class MemberRestController {
    
        // omitted

        // (1)
        @RequestMapping(method = RequestMethod.POST)
        // (2)
        @ResponseStatus(HttpStatus.CREATED)
        public MemberResource postMember(
                // (3)
                @RequestBody @Validated({ PostMembers.class, Default.class }) 
                MemberResource requestedResource) {

            // (4)
            Member inputMember = beanMapper.map(requestedResource, Member.class);
            Member createdMember = memberService.createMember(inputMember);

            MemberResource responseResource = beanMapper.map(createdMember,
                    MemberResource.class);

            return responseResource;
        }

        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@RequestMapping``\アノテーションのmethod属性に、\ ``RequestMethod.POST``\を指定する。
    * - | (2)
      - | メソッドアノテーションとして、\ ``@ResponseStatus``\アノテーションを付与し、応答するステータスコードを指定する。
        | \ ``@ResponseStatus``\アノテーションのvalue属性には、\ **201(Created)**\を設定する。
    * - | (3)
      - | 新規に作成するリソースの情報を受け取るためのJavaBean(Resourceクラス)を引数に指定する。
        | 引数アノテーションとして、``@org.springframework.web.bind.annotation.RequestBody``\アノテーションを付与する。
        | \ ``@RequestBody``\アノテーションを付与することで、リクエストBodyに設定されているJSONやXMLのデータがResourceオブジェクトにunmarshalされる。
        |
        | 入力チェックを有効化するために、引数アノテーションとして、\ ``@Validated``\アノテーションを付与する。入力チェックの詳細については、「\ :doc:`../WebApplicationDetail/Validation`\」を参照されたい。
    * - | (4)
      - | ドメイン層のServiceのメソッドを呼び出し、新規にリソースを作成する。
        | ドメイン層の実装については、「:doc:`../../ImplementationAtEachLayer/DomainLayer`」を参照されたい。

|

* リクエスト例

 .. code-block:: guess
    :emphasize-lines: 1

    POST /rest-api-web/api/v1/members HTTP/1.1
    Accept: text/plain, application/json, application/*+json, */*
    Content-Type: application/json;charset=UTF-8
    User-Agent: Java/1.7.0_51
    Host: localhost:8080
    Connection: keep-alive
    Content-Length: 248
    
    {"firstName":"John","lastName":"Smith","genderCode":"1","dateOfBirth":"2013-03-13","emailAddress":"user1394708306056@test.com","telephoneNumber":"09012345678","zipCode":"1710051","address":"Tokyo","credential":{"signId":null,"password":"zaq12wsx"}}

|

* レスポンス例

 .. code-block:: guess
    :emphasize-lines: 1

    HTTP/1.1 201 Created
    Server: Apache-Coyote/1.1
    X-Track: c7e9c8a9aa4f40ff87f3acdb77baccdf
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Thu, 13 Mar 2014 10:58:26 GMT
    
    {"memberId":"M000000023","firstName":"John","lastName":"Smith","genderCode":"1","dateOfBirth":"2013-03-13","emailAddress":"user1394708306056@test.com","telephoneNumber":"09012345678","zipCode":"1710051","address":"Tokyo","credential":{"signId":"user1394708306056@test.com","passwordLastChangedAt":"2014-03-13T10:58:26.324Z","lastModifiedAt":"2014-03-13T10:58:26.324Z"},"createdAt":"2014-03-13T10:58:26.324Z","lastModifiedAt":"2014-03-13T10:58:26.324Z"}

|

.. _RESTHowToUseApiImplementationOfGetSpecifiedResource:

指定されたリソースを取得するREST APIの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
URIで指定されたMemberリソースを取得するREST APIの実装例を、以下に示す。

* | REST APIの実装
  | URIで指定されたMemberリソースを取得する処理を実装する。

 .. code-block:: java
    :emphasize-lines: 7, 9, 12, 15

    @RequestMapping("members")
    @RestController
    public class MemberRestController {

        // omitted

        // (1)
        @RequestMapping(value = "{memberId}", method = RequestMethod.GET)
        // (2)
        @ResponseStatus(HttpStatus.OK)
        public MemberResource getMember(
                // (3)
                @PathVariable("memberId") String memberId) {
    
            // (4)
            Member member = memberService.getMember(memberId);
    
            MemberResource responseResource = beanMapper.map(member,
                    MemberResource.class);
    
            return responseResource;
        }

        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@RequestMapping``\アノテーションのvalue属性にパス変数(上記例では\ ``{memberId}``\)を、method属性に\ ``RequestMethod.GET``\を指定する。
        | \ ``{memberId}``\には、リソースを一意に識別するための値が指定される。
    * - | (2)
      - | メソッドアノテーションとして、\ ``@ResponseStatus``\アノテーションを付与し、応答するステータスコードを指定する。
        | \ ``@ResponseStatus``\アノテーションのvalue属性には、200(OK)を設定する。
    * - | (3)
      - | リソースを一意に識別するための値を、パス変数から取得する。
        | 引数アノテーションとして、\ ``@PathVariable("memberId")``\を指定することで、パス変数(\ ``{memberId}``\)に指定された値をメソッドの引数として受け取ることが出来る。
        | パス変数の詳細については、 「:ref:`controller_method_argument-pathvariable-label`」を参照されたい。
        | 上記例だと、URIが\ ``/api/v1/members/M12345``\の場合、引数の\ ``memberId``\に\ ``"M12345"``\が格納される。
    * - | (4)
      - | ドメイン層のServiceのメソッドを呼び出し、パス変数から取得したIDに一致するリソースの情報(Entityなど)を取得する。
        | ドメイン層の実装については、「:doc:`../../ImplementationAtEachLayer/DomainLayer`」を参照されたい。

|

* リクエスト例

 .. code-block:: guess
    :emphasize-lines: 1

    GET /rest-api-web/api/v1/members/M000000003 HTTP/1.1
    Accept: text/plain, application/json, application/*+json, */*
    User-Agent: Java/1.7.0_51
    Host: localhost:8080
    Connection: keep-alive

|

* レスポンス例

 .. code-block:: guess
    :emphasize-lines: 1

    HTTP/1.1 200 OK
    Server: Apache-Coyote/1.1
    X-Track: 275b4e7a61f946eea47672f272315ac2
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Thu, 13 Mar 2014 11:25:13 GMT
    
    {"memberId":"M000000003","firstName":"John","lastName":"Smith","genderCode":"1","dateOfBirth":"2013-03-13","emailAddress":"user1394709913496@test.com","telephoneNumber":"09012345678","zipCode":"1710051","address":"Tokyo","credential":{"signId":"user1394709913496@test.com","passwordLastChangedAt":"2014-03-13T11:25:13.762Z","lastModifiedAt":"2014-03-13T11:25:13.762Z"},"createdAt":"2014-03-13T11:25:13.762Z","lastModifiedAt":"2014-03-13T11:25:13.762Z"}

|

.. _RESTHowToUseApiImplementationOfPutSpecifiedResource:

指定されたリソースを更新するREST APIの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
URIで指定されたMemberリソースを更新するREST APIの実装例を、以下に示す。

* | REST APIの実装
  | URIで指定されたMemberリソースを更新する処理を実装する。

 .. code-block:: java
    :emphasize-lines: 7, 9, 13, 17

    @RequestMapping("members")
    @RestController
    public class MemberRestController {

        // omitted

        // (1)
        @RequestMapping(value = "{memberId}", method = RequestMethod.PUT)
        // (2)
        @ResponseStatus(HttpStatus.OK)
        public MemberResource putMember(
                @PathVariable("memberId") String memberId,
                // (3)
                @RequestBody @Validated({ PutMember.class, Default.class })
                MemberResource requestedResource) {
    
            // (4)
            Member inputMember = beanMapper.map(
                requestedResource, Member.class);
            Member updatedMember = memberService.updateMember(
                memberId, inputMember);
    
            MemberResource responseResource = beanMapper.map(updatedMember,
                    MemberResource.class);
    
            return responseResource;
        }

        // omitted
        
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@RequestMapping``\アノテーションのvalue属性にパス変数(上記例では\ ``{memberId}``\)を、method属性に\ ``RequestMethod.PUT``\を指定する。
        | \ ``{memberId}``\には、リソースを一意に識別するための値が指定される。
    * - | (2)
      - | メソッドアノテーションとして、\ ``@ResponseStatus``\アノテーションを付与し、応答するステータスコードを指定する。
        | \ ``@ResponseStatus``\アノテーションのvalue属性には、200(OK)を設定する。
    * - | (3)
      - | リソースの更新内容を受け取るためのJavaBean(Resourceクラス)を引数に指定する。
        | 引数アノテーションとして、\ ``@RequestBody``\アノテーションを付与することで、リクエストBodyに設定されているJSONやXMLのデータがResourceオブジェクトにunmarshalされる。
        |
        | 入力チェックを有効化するために、引数アノテーションとして、\ ``@Validated``\アノテーションを付与する。
        | 入力チェックの詳細については、「\ :doc:`../WebApplicationDetail/Validation`\」を参照されたい。
    * - | (4)
      - | ドメイン層のServiceのメソッドを呼び出し、パス変数から取得したIDに一致するリソースの情報(Entityなど)を更新する。
        | ドメイン層の実装については、「:doc:`../../ImplementationAtEachLayer/DomainLayer`」を参照されたい。

|

* リクエスト例

 .. code-block:: guess
    :emphasize-lines: 1

    PUT /rest-api-web/api/v1/members/M000000004 HTTP/1.1
    Accept: text/plain, application/json, application/*+json, */*
    Content-Type: application/json;charset=UTF-8
    User-Agent: Java/1.7.0_51
    Host: localhost:8080
    Connection: keep-alive
    Content-Length: 221
    
    {"memberId":"M000000004","firstName":"John","lastName":"Smith","genderCode":"1","dateOfBirth":"2013-03-08","emailAddress":"user1394710559584@test.com","telephoneNumber":"09012345678","zipCode":"1710051","address":"Tokyo"}

|

* レスポンス例

 .. code-block:: guess
    :emphasize-lines: 1

    HTTP/1.1 200 OK
    Server: Apache-Coyote/1.1
    X-Track: 5e8fea3aae044e94bf20a293e155af57
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Thu, 13 Mar 2014 11:35:59 GMT
    
    {"memberId":"M000000004","firstName":"John","lastName":"Smith","genderCode":"1","dateOfBirth":"2013-03-08","emailAddress":"user1394710559584@test.com","telephoneNumber":"09012345678","zipCode":"1710051","address":"Tokyo","credential":{"signId":"user1394710559584@test.com","passwordLastChangedAt":"2014-03-13T11:35:59.847Z","lastModifiedAt":"2014-03-13T11:35:59.847Z"},"createdAt":"2014-03-13T11:35:59.847Z","lastModifiedAt":"2014-03-13T11:36:00.122Z"}

|


|

.. _RESTHowToUseApiImplementationOfDeleteSpecifiedResource:

指定されたリソースを削除するREST APIの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
URIで指定されたMemberリソースを削除するREST APIの実装例を、以下に示す。

* | REST APIの実装
  | URIで指定されたMemberリソースを削除する処理を実装する。

 .. code-block:: java
    :emphasize-lines: 7, 9, 14

    @RequestMapping("members")
    @RestController
    public class MemberRestController {

        // omitted

        // (1)
        @RequestMapping(value = "{memberId}", method = RequestMethod.DELETE)
        // (2)
        @ResponseStatus(HttpStatus.NO_CONTENT)
        public void deleteMember(
                @PathVariable("memberId") String memberId) {
    
            // (3)
            memberService.deleteMember(memberId);
            
        }

        // omitted
        
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@RequestMapping``\アノテーションのvalue属性にパス変数(上記例では\ ``{memberId}``\)を、method属性に\ ``RequestMethod.DELETE``\を指定する。
    * - | (2)
      - | メソッドアノテーションとして、\ ``@ResponseStatus``\アノテーションを付与し、応答するステータスコードを指定する。
        | \ ``@ResponseStatus``\アノテーションのvalue属性には、\ **204(NO_CONTENT)**\を設定する。
    * - | (3)
      - | ドメイン層のServiceのメソッドを呼び出し、パス変数から取得したIDに一致するリソースの情報(Entityなど)を削除する。
        | ドメイン層の実装については、「:doc:`../../ImplementationAtEachLayer/DomainLayer`」を参照されたい。

 .. note::
 
    削除したリソースの情報をレスポンスBODYに設定する場合は、ステータスコードには200(OK)を設定する。

|

* リクエスト例

 .. code-block:: guess
    :emphasize-lines: 1

    DELETE /rest-api-web/api/v1/members/M000000005 HTTP/1.1
    Accept: text/plain, application/json, application/*+json, */*
    User-Agent: Java/1.7.0_51
    Host: localhost:8080
    Connection: keep-alive

|

* レスポンス例

 .. code-block:: guess
    :emphasize-lines: 1

    HTTP/1.1 204 No Content
    Server: Apache-Coyote/1.1
    X-Track: e06c5bd40c864a299c48d9be3f12b2c0
    Date: Thu, 13 Mar 2014 11:40:05 GMT

|


.. _RESTHowToUseExceptionHandling:

例外のハンドリングの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
RESTful Web Serviceで発生した例外のハンドリング方法について説明する。

| Spring MVCでは、RESTful Web Service向けの汎用的な例外ハンドリングの仕組みは用意されていない。
| 代わりに、RESTful Web Service向けの例外ハンドリングの実装を補助してくれるクラスとして、(\ ``org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler``\)が提供されている。
| 本ガイドラインでは、Spring MVCから提供されているクラスを継承した例外ハンドリング用のクラスを作成し、作成した例外ハンドリング用のクラスに\ ``@ControllerAdvice``\アノテーションを付与する事で、例外ハンドリングを共通的に行う方法を推奨する。

| \ ``ResponseEntityExceptionHandler``\では、Spring MVCのフレームワーク内で発生する例外を\ ``@ExceptionHandler``\アノテーションを使ってハンドリングするメソッドが予め実装されている。
| そのため、Spring MVCのフレームワーク内で発生する例外のハンドリングを個別に実装する必要がない。
| また、\ ``ResponseEntityExceptionHandler``\でハンドリングされる例外に対応するHTTPステータスコードは、\ ``DefaultHandlerExceptionResolver``\と同様の仕様で設定される。
| ハンドリングされる例外と設定されるHTTPステータスコードについては、「\ :ref:`exception-handling-appendix-defaulthandlerexceptionresolver-label`\」を参照されたい。

| \ ``ResponseEntityExceptionHandler``\のデフォルトの実装ではレスポンスBodyは空で返却されるが、レスポンスBodyにエラー情報を出力する様に拡張する事ができる。
| 本ガイドラインでは、レスポンスBodyに適切なエラー情報を出力する事を推奨する。

| 具体的な実装例を説明する前に、\ ``ResponseEntityExceptionHandler``\を継承した例外ハンドリング用のクラスを作成し、例外ハンドリングを共通的に行う際の処理フローについて説明する。
| なお、個別に実装が必要になるのは、赤枠の部分となる。

 .. figure:: ./images_REST/RESTHowToUseExceptionHandling.png
   :alt: Image of exception handling by Spring MVC
   :width: 100%


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10  20 70

    * - 項番
      - 処理レイヤ
      - 説明
    * - | (1)
        | (2)
      - | Spring MVC
        | (Framework)
      - | Spring MVCはクライアントからのリクエストを受け付け、REST APIを呼び出す。
    * - | (3)
      - | 
        | 
      - | REST APIの処理中に例外が発生する。
        | 発生した例外は、Spring MVCによって捕捉される。
    * - | (4)
      - | 
        | 
      - | Spring MVCは、例外ハンドリング用のクラスに処理を委譲する。
    * - | (5)
      - | Custom Exception Handler
        | (Common Component)
      - | 例外ハンドリング用のクラスでは、エラー情報を保持するエラーオブジェクトを生成し、Spring MVCに返却する。
    * - | (6)
      - | Spring MVC
        | (Framework)
      - | Spring MVCは、\ ``HttpMessageConverter``\を利用して、エラーオブジェクトをJSON形式の電文に変換する。
    * - | (7)
      - | 
        | 
      - | Spring MVCは、JSON形式のエラー電文をレスポンスBODYに設定し、クライアントにレスポンスする。

|

.. _RESTHowToUseExceptionHandlingForErrorContentInResponseBody:

レスポンスBodyにエラー情報を出力するための実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
* エラー情報は以下のJSON形式とする。

 .. code-block:: json
    :emphasize-lines: 10, 20, 23

    {
      "code" : "e.ex.fw.7001",
      "message" : "Validation error occurred on item in the request body.",
      "details" : [ {
        "code" : "ExistInCodeList",
        "message" : "\"genderCode\" must exist in code list of CL_GENDER.",
        "target" : "genderCode"
      } ]
    }

|

* エラー情報を保持するJavaBeanを作成する。

 .. code-block:: java
    :emphasize-lines: 9, 19, 22

    package org.terasoluna.examples.rest.api.common.error;
    
    import java.io.Serializable;
    import java.util.ArrayList;
    import java.util.List;
    
    import com.fasterxml.jackson.annotation.JsonInclude;

    // (1)
    public class ApiError implements Serializable {
    
        private static final long serialVersionUID = 1L;
    
        private final String code;
    
        private final String message;
    
        @JsonInclude(JsonInclude.Include.NON_EMPTY)
        private final String target; // (2)
    
        @JsonInclude(JsonInclude.Include.NON_EMPTY)
        private final List<ApiError> details = new ArrayList<>(); // (3)
    
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

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | エラー情報を保持するためのクラスを作成する。
        | 上記例では、エラーコード、エラーメッセージ、エラー対象、エラーの詳細情報のリストを保持するクラスとなっている。
    * - | (2)
      - | エラーが発生した対象を識別するための値を保持するフィールド。
        | 入力チェックでエラーが発生した場合、どの項目でエラーが発生したのかを識別できる値をクライアントに返却する事が求められるケースがある。
        | そのような場合は、エラーが発生した項目名を保持するフィールドが必要になる。
    * - | (3)
      - | エラーの詳細情報のリストを保持するためのフィールド。
        | 入力チェックでエラーが発生した場合、エラー原因が複数存在する場合があるため、すべてのエラー情報をクライアントに返却する事が求められるケースがある。
        | そのような場合は、エラーの詳細情報をリストで保持するフィールドが必要になる。

 .. tip::   
 
    フィールドに\ ``@JsonInclude(JsonInclude.Include.NON_EMPTY)``\を指定することで、値が\ ``null``\や空の場合にJSONに項目が出力されないようにする事が出来る。
    項目を出力させないための条件を\ ``null``\に限定したい場合は、\ ``@JsonInclude(JsonInclude.Include.NON_NULL)``\を指定すればよい。

|

* エラー情報を保持するJavaBeanを生成するためのクラスを作成する。

 全ての例外ハンドリングの実装が完了した際のソースコードについては、\ :ref:`Appendix <RESTAppendixSoruceCodesOfApiErrorCreator>`\を参照されたい。

 .. code-block:: java
    :emphasize-lines: 1, 10

    // (4)
    @Component
    public class ApiErrorCreator {
    
        @Inject
        MessageSource messageSource;

        public ApiError createApiError(WebRequest request, String errorCode,
                String defaultErrorMessage, Object... arguments) {
            // (5)
            String localizedMessage = messageSource.getMessage(errorCode,
                    arguments, defaultErrorMessage, request.getLocale()); 
            return new ApiError(errorCode, localizedMessage);
        }
    
        // omitted
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (4)
      - | 必要に応じて、エラー情報を生成するためのメソッドを提供するクラスを作成する。
        | このクラスの作成は必須ではないが、役割を明確に分担するために作成する事を推奨する。
    * - | (5)
      - | エラーメッセージは、\ ``MessageSource``\より取得する。
        | メッセージの管理方法については、「\ :doc:`../WebApplicationDetail/MessageManagement`\」を参照されたい。

 .. tip::

    上記例では、メッセージのローカライズをサポートするために\ ``org.springframework.web.context.request.WebRequest``\を引数として受け取っている。
    メッセージのローカライズが必要ない場合は、\ ``WebRequest``\は不要である。
    
    \ ``java.util.Locale``\ではなく\ ``WebRequest``\を引数にしている理由は、エラーメッセージの中にHTTPリクエストの内容を埋め込むといった要件が追加される事を考慮したためである。
    エラーメッセージの中にHTTPリクエストの内容を埋め込む要件がない場合は、\ ``Locale``\でもよい。

|

* \ ``ResponseEntityExceptionHandler``\のメソッドを拡張し、レスポンスBodyにエラー情報を出力するための実装を行う。

 全ての例外ハンドリングの実装が完了した際のソースコードについては、\ :ref:`Appendix <RESTAppendixSoruceCodesOfApiGlobalExceptionHandler>`\を参照されたい。

 .. code-block:: java
    :emphasize-lines: 1-2, 10-12, 16, 24

    @ControllerAdvice // (6)
    public class ApiGlobalExceptionHandler extends ResponseEntityExceptionHandler {
    
        @Inject
        ApiErrorCreator apiErrorCreator;
    
        @Inject
        ExceptionCodeResolver exceptionCodeResolver;
    
        // (7)
        @Override
        protected ResponseEntity<Object> handleExceptionInternal(Exception ex,
                Object body, HttpHeaders headers, HttpStatus status,
                WebRequest request) {
            final Object apiError;
            // (8)
            if (body == null) {
                String errorCode = exceptionCodeResolver.resolveExceptionCode(ex);
                apiError = apiErrorCreator.createApiError(request, errorCode, ex
                        .getLocalizedMessage());
            } else {
                apiError = body;
            }
            // (9)
            return ResponseEntity.status(status).headers(headers).body(apiError);
        }
        
        // omitted

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (6)
      - | Spring MVCから提供されている\ ``ResponseEntityExceptionHandler``\を継承したクラスを作成し、\ ``@ControllerAdvice``\アノテーションを付与する。
    * - | (7)
      - | \ ``ResponseEntityExceptionHandler``\のhandleExceptionInternalメソッドをオーバライドする。
    * - | (8)
      - | レスポンスBodyに出力するJavaBeanの指定がない場合は、エラー情報を保持するJavaBeanオブジェクトを生成する。
        | 上記例では、共通ライブラリから提供している\ ``ExceptionCodeResolver``\を使用して、例外クラスをエラーコードを変換している。
        | \ ``ExceptionCodeResolver``\の設定例については、「\ :ref:`RESTHowToUseExceptionHandlingSettingsOfExceptionCodeResolver`\」を参照されたい。
        |
        | レスポンスBodyに出力するJavaBeanの指定がある場合は、指定されたJavaBeanをそのまま使用する。
        | この処理は、例外毎のエラーハンドリング処理にて、個別にエラー情報が生成される事を考慮した実装となっている。
    * - | (9)
      - | レスポンス用のHTTP EntityのBody部分に、(8)で生成したエラー情報を設定し返却する。
        | 返却したエラー情報は、フレームワークによってJSONに変換されレスポンスされる。
        |
        | ステータスコードには、Spring MVCから提供されている\ ``ResponseEntityExceptionHandler``\によって適切な値が設定される。
        | 設定されるステータスコードについては、「\ :ref:`exception-handling-appendix-defaulthandlerexceptionresolver-label`\」を参照されたい。

 .. tip:: **Spring Framework 4.0 より追加された@ControllerAdviceアノテーションの属性について**

    \ ``@ControllerAdvice``\ アノテーションの属性を指定することで、
    \ ``@ControllerAdvice``\ が付与されたクラスで実装したメソッドを適用するControllerを柔軟に指定できるように改善されている。
    属性の詳細については、\ :ref:`@ControllerAdviceの属性 <application_layer_controller_advice_attribute>`\ を参照されたい。

 .. note:: **@ControllerAdviceアノテーションの属性使用時の注意点**

    \ ``@ControllerAdvice``\ アノテーションの属性を使用することで、さまざまな粒度で例外ハンドリングを共通化することができるようになるが、
    アプリケーション共通の例外ハンドラクラス(上記例の\ ``ApiGlobalExceptionHandler``\ クラスに相当するクラス)に対しては、\ ``@ControllerAdvice``\ アノテーションの属性を指定しない方がよい。

    \ ``ApiGlobalExceptionHandler``\ に付与する\ ``@ControllerAdvice``\ アノテーションに属性を指定した場合、Spring MVCが提供するフレームワーク処理の中で発生する一部の例外をハンドリングできないケースがある。

    具体的には、リクエストに対応するREST API(Controllerのハンドラメソッド)が見つからない時に発生する例外を\ ``ApiGlobalExceptionHandler``\ クラスでハンドリングする事ができないため、
    「405 Method Not Allowed」などのエラーを正しく応答する事が出来なくなってしまう。

|

* レスポンス例

 .. code-block:: guess
    :emphasize-lines: 1, 9

    HTTP/1.1 400 Bad Request
    Server: Apache-Coyote/1.1
    X-Track: e60b3b6468194e22852c8bfc7618e625
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Thu, 13 Mar 2014 12:16:55 GMT
    Connection: close
    
    {"code":"e.ex.fw.7001","message":"Validation error occurred on item in the request body.","details":[{"code":"ExistInCodeList","message":"\"genderCode\" must exist in code list of CL_GENDER.","target":"genderCode"}]}

|

.. _RESTHowToUseExceptionHandlingForValidationError:

入力エラー例外のハンドリング実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
入力エラー（電文不正、単項目チェックエラー、相関項目チェックエラー）を応答するための実装例について説明する。

入力エラーを応答するためには、以下の３つの例外をハンドリングする必要がある。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 35 55

    * - 項番
      - 例外
      - 説明
    * - | (1)
      - | org.springframework.web.bind.
        | MethodArgumentNotValidException
      - | リクエストBODYに指定されたJSONやXMLに対する入力チェックでエラーが発生した場合、本例外が発生する。
        | 具体的には、リソースのPOST又はPUT時に指定するリソースに不正な値が指定されている場合に発生する。
    * - | (2)
      - | org.springframework.validation.
        | BindException
      - | リクエストパラメータ(key=value形式のクエリ文字列)に対する入力チェックでエラーが発生した場合、本例外が発生する。
        | 具体的には、リソースコレクションのGET時に指定する検索条件に不正な値が指定されている場合に発生する。
    * - | (3)
      - | org.springframework.http.converter.
        | HttpMessageNotReadableException
      - | JSONやXMLからResourceオブジェクトを生成する際にエラーが発生した場合は、本例外が発生する。
        | 具体的には、JSONやXMLの構文不正やスキーマ定義に違反などがあった場合に発生する。

 .. note::
 
    Spring Frameworkから提供されているアノテーションを使用してリクエストパラメータ、リクエストヘッダ、パス変数から値を取得する際に、値の型変換エラーが発生した場合、\ ``org.springframework.beans.TypeMismatchException``\が発生する。
    
    Controllerのハンドラメソッドの引数(\ ``String``\以外の引数)に、以下のアノテーションを指定した場合、\ ``TypeMismatchException``\が発生する可能性がある。
    
     * \ ``@org.springframework.web.bind.annotation.RequestParam``\
     * \ ``@org.springframework.web.bind.annotation.RequestHeader``\
     * \ ``@org.springframework.web.bind.annotation.Pathvariable``\
     * \ ``@org.springframework.web.bind.annotation.MatrixVariable``\
     
    \ ``TypeMismatchException``\は、\ ``ResponseEntityExceptionHandler``\によって例外がハンドリングされ、400(Bad Request)となるので個別にハンドリングしなくてもよい。
    
    エラー情報に設定するエラーコードとエラーメッセージの解決方法については、「\ :ref:`RESTHowToUseExceptionHandlingSettingsOfExceptionCodeResolver`\」を参照されたい。

|

* 入力チェックエラー用のエラー情報を生成するためのメソッドを作成する。

 .. code-block:: java
    :emphasize-lines: 9-10, 26-27

    @Component
    public class ApiErrorCreator {
    
        @Inject
        MessageSource messageSource;

        // omitted

        // (1)
        public ApiError createBindingResultApiError(WebRequest request,
                String errorCode, BindingResult bindingResult,
                String defaultErrorMessage) {
            ApiError apiError = createApiError(request, errorCode,
                    defaultErrorMessage);
            for (FieldError fieldError : bindingResult.getFieldErrors()) {
                apiError.addDetail(createApiError(request, fieldError, fieldError
                        .getField()));
            }
            for (ObjectError objectError : bindingResult.getGlobalErrors()) {
                apiError.addDetail(createApiError(request, objectError, objectError
                        .getObjectName()));
            }
            return apiError;
        }
    
        // (2)
        private ApiError createApiError(WebRequest request,
                DefaultMessageSourceResolvable messageResolvable, String target) {
            String localizedMessage = messageSource.getMessage(messageResolvable,
                    request.getLocale());
            return new ApiError(messageResolvable.getCode(), localizedMessage, target);
        }

        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 入力チェック用のエラー情報を生成するためのメソッドを作成する。
        | 上記例では、単項目チェックエラー(\ ``FieldError``\)と相関項目チェックエラー(\ ``ObjectError``\)を、エラーの詳細情報に追加している。
        | 項目毎のエラー情報を出力する必要がない場合は、本メソッドを用意する必要はない。
    * - | (2)
      - | 単項目チェックエラー(\ ``FieldError``\)と相関項目チェックエラー(\ ``ObjectError``\)で同じ処理を実装する事になるので、共通メソッドとして本メソッドを作成している。

|

* \ ``ResponseEntityExceptionHandler``\のメソッドを拡張し、レスポンスBodyに入力チェック用のエラー情報を出力するための実装を行う。

 .. code-block:: java
    :emphasize-lines: 12-14, 21-23, 29-31, 34-36, 44-45

    @ControllerAdvice
    public class ApiGlobalExceptionHandler extends ResponseEntityExceptionHandler {
    
        @Inject
        ApiErrorCreator apiErrorCreator;
    
        @Inject
        ExceptionCodeResolver exceptionCodeResolver;
    
        // omitted

        // (3)
        @Override
        protected ResponseEntity<Object> handleMethodArgumentNotValid(
                MethodArgumentNotValidException ex, HttpHeaders headers,
                HttpStatus status, WebRequest request) {
            return handleBindingResult(ex, ex.getBindingResult(), headers, status,
                    request);
        }
    
        // (4)
        @Override
        protected ResponseEntity<Object> handleBindException(BindException ex,
                HttpHeaders headers, HttpStatus status, WebRequest request) {
            return handleBindingResult(ex, ex.getBindingResult(), headers, status,
                    request);
        }
    
        // (5)
        @Override
        protected ResponseEntity<Object> handleHttpMessageNotReadable(
                HttpMessageNotReadableException ex, HttpHeaders headers,
                HttpStatus status, WebRequest request) {
            if (ex.getCause() instanceof Exception) {
                return handleExceptionInternal((Exception) ex.getCause(), null,
                        headers, status, request);
            } else {
                return handleExceptionInternal(ex, null, headers, status, request);
            }
        }

        // omitted

        // (6)
        protected ResponseEntity<Object> handleBindingResult(Exception ex,
                BindingResult bindingResult, HttpHeaders headers,
                HttpStatus status, WebRequest request) {
            String code = exceptionCodeResolver.resolveExceptionCode(ex);
            String errorCode = exceptionCodeResolver.resolveExceptionCode(ex);
            ApiError apiError = apiErrorCreator.createBindingResultApiError(
                    request, errorCode, bindingResult, ex.getMessage());
            return handleExceptionInternal(ex, apiError, headers, status, request);
        }

        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (3)
      - | \ ``ResponseEntityExceptionHandler``\のhandleMethodArgumentNotValidメソッドをオーバライドし、\ ``MethodArgumentNotValidException``\のエラーハンドリングを拡張する。
        | 上記例では、入力チェックエラーをハンドリングするための共通メソッド(6)に処理を委譲している。
        | 項目毎のエラー情報を出力する必要がない場合は、オーバライドする必要はない。
        | 
        | ステータスコードには\ **400(Bad Request)**\が設定され、指定されたリソースの項目値に不備がある事を通知する。
    * - | (4)
      - | \ ``ResponseEntityExceptionHandler``\のhandleBindExceptionメソッドをオーバライドし、\ ``BindException``\のエラーハンドリングを拡張する。
        | 上記例では、入力チェックエラーをハンドリングするための共通メソッド(6)に処理を委譲している。
        | 項目毎のエラー情報を出力する必要がない場合は、オーバライドする必要はない。
        | 
        | ステータスコードには\ **400(Bad Request)**\が設定され、指定されたリクエストパラメータに不備がある事を通知する。
    * - | (5)
      - | \ ``ResponseEntityExceptionHandler``\のhandleHttpMessageNotReadableメソッドをオーバライドし、\ ``HttpMessageNotReadableException``\のエラーハンドリングを拡張する。
        | 上記例では、細かくエラーハンドリングを行うために、原因例外を使ってエラーハンドリングしている。
        | 細かくエラーハンドリングをしなくてもよい場合は、オーバライドする必要はない。
        | 
        | ステータスコードには\ **400(Bad Request)**\が設定され、指定されたリソースのフォーマットなどに不備がある事を通知する。
    * - | (6)
      - | 入力チェックエラーのエラー情報を保持するJavaBeanオブジェクトを生成する。
        | 上記例では、handleMethodArgumentNotValidとhandleBindExceptionで同じ処理を実装する事になるので、共通メソッドとして本メソッドを作成している。

 .. tip:: **JSON使用時のエラーハンドリングについて**

    リソースのフォーマットとしてJSONを使用する場合、以下の例外が\ ``HttpMessageNotReadableException``\の原因例外として格納される。

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 35 55
        :class: longtable

        * - 項番
          - 例外クラス
          - 説明
        * - | (1)
          - | com.fasterxml.jackson.core.
            | JsonParseException
          - | JSONとして不正な構文が含まれる場合に発生する。
        * - | (2)
          - | com.fasterxml.jackson.databind.exc.
            | UnrecognizedPropertyException
          - | Resourceオブジェクトに存在しないフィールドがJSONに指定されている場合に発生する。
        * - | (3)
          - | com.fasterxml.jackson.databind.
            | JsonMappingException
          - | JSONからResourceオブジェクトへ変換する際に、値の型変換エラーが発生した場合に発生する。

|

* 入力チェックエラー(単項目チェック、相関項目チェックエラー)が発生した場合、以下のようなエラー応答が行われる。

 .. code-block:: guess
    :emphasize-lines: 1, 9

    HTTP/1.1 400 Bad Request
    Server: Apache-Coyote/1.1
    X-Track: 13522b3badf2432ba4cad0dc7aeaee80
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Wed, 19 Feb 2014 05:08:28 GMT
    Connection: close
    
    {"code":"e.ex.fw.7002","message":"Validation error occurred on item in the request parameters.","details":[{"code":"NotEmpty","message":"\"{0}\" may not be empty.","target":"name"}]}

|

* JSONエラー(フォーマットエラーなど)が発生した場合、以下のようなエラー応答が行われる。

 .. code-block:: guess
    :emphasize-lines: 1, 9

    HTTP/1.1 400 Bad Request
    Server: Apache-Coyote/1.1
    X-Track: ca4c742a6bfd49e5bc01cd0b124738a1
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Wed, 19 Feb 2014 13:32:24 GMT
    Connection: close
    
    {"code":"e.ex.fw.7003","message":"Request body format error occurred."}

|

.. _RESTHowToUseExceptionHandlingForNotFound:

リソース未検出エラー例外のハンドリング実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
リソースが存在しない場合に、リソース未検出エラーを応答するための実装例について説明する。

| パス変数から取得したIDに一致するリソースが見つからない場合は、リソースが見つからない事を通知する例外を発生させる。
| リソースが見つからなかった事を通知する例外として、共通ライブラリより\ ``org.terasoluna.gfw.common.exception.ResourceNotFoundException``\を用意している。
| 以下に実装例を示す。

|

* パス変数から取得したIDに一致するリソースが見つからない場合は、\ ``ResourceNotFoundException``\を発生させる。

 .. code-block:: java
    :emphasize-lines: 4-5

    public Member getMember(String memberId) {
        Member member = memberRepository.findOne(memberId);
        if (member == null) {
            throw new ResourceNotFoundException(ResultMessages.error().add(
                    "e.ex.mm.5001", memberId));
        }
        return member;
    }

|

* \ ``ResultMessages``\用のエラー情報を生成するためのメソッドを作成する。

 .. code-block:: java
    :emphasize-lines: 6-7

    @Component
    public class ApiErrorCreator {

        // omitted

        // (1)
        public ApiError createResultMessagesApiError(WebRequest request,
                String rootErrorCode, ResultMessages resultMessages,
                String defaultErrorMessage) {
            ApiError apiError;
            if (resultMessages.getList().size() == 1) {
                ResultMessage resultMessage = resultMessages.iterator().next();
                String errorCode = resultMessage.getCode();
                String errorText = resultMessage.getText();
                if (errorCode == null && errorText == null) {
                    errorCode = rootErrorCode;
                }
                apiError = createApiError(request, errorCode, errorText,
                        resultMessage.getArgs());
            } else {
                apiError = createApiError(request, rootErrorCode,
                        defaultErrorMessage);
                for (ResultMessage resultMessage : resultMessages.getList()) {
                    apiError.addDetail(createApiError(request, resultMessage
                            .getCode(), resultMessage.getText(), resultMessage
                            .getArgs()));
                }
            }
            return apiError;
        }
    
        // omitted
        
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 処理結果からエラー情報を生成するためのメソッドを作成する。
        | 上記例では、\ ``ResultMessages``\が保持しているメッセージ情報を、エラー情報に設定している。
        | 
        
 .. note::
 
    上記例では、\ ``ResultMessages``\が複数のメッセージを保持する事ができるため、格納されているメッセージが1件の時と複数件の時で処理をわけている。

    複数件のメッセージをサポートする必要がない場合は、先頭の1件をエラー情報として生成する処理にすればよい。


|

* エラーハンドリングを行うクラスに、リソースが見つからない事を通知する例外をハンドリングするためのメソッドを作成する。

 .. code-block:: java
    :emphasize-lines: 12-13, 17, 22-23

    @ControllerAdvice
    public class ApiGlobalExceptionHandler extends ResponseEntityExceptionHandler {
    
        @Inject
        ApiErrorCreator apiErrorCreator;
    
        @Inject
        ExceptionCodeResolver exceptionCodeResolver;

        // omitted

        // (2)
        @ExceptionHandler(ResourceNotFoundException.class)
        public ResponseEntity<Object> handleResourceNotFoundException(
                ResourceNotFoundException ex, WebRequest request) {
            return handleResultMessagesNotificationException(ex, new HttpHeaders(),
                    HttpStatus.NOT_FOUND, request);
        }

        // omitted

        // (3)
        private ResponseEntity<Object> handleResultMessagesNotificationException(
                ResultMessagesNotificationException ex, HttpHeaders headers,
                HttpStatus status, WebRequest request) {
            String errorCode = exceptionCodeResolver.resolveExceptionCode(ex);
            ApiError apiError = apiErrorCreator.createResultMessagesApiError(
                    request, errorCode, ex.getResultMessages(), ex.getMessage());
            return handleExceptionInternal(ex, apiError, headers, status, request);
        }
        
        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (2)
      - | \ ``ResourceNotFoundException``\をハンドリングするためのメソッドを追加する。
        | メソッドアノテーションとして\ ``@ExceptionHandler(ResourceNotFoundException.class)``\を指定すると、\ ``ResourceNotFoundException``\の例外をハンドリングする事ができる。
        | 上記例では、\ ``ResourceNotFoundException``\の親クラス(\ ``ResultMessagesNotificationException``\)の例外をハンドリングするメソッドに処理を委譲している。
        |
        | ステータスコードには\ **404(Not Found)**\を設定し、指定されたリソースがサーバに存在しない事を通知する。
    * - | (3)
      - | リソース未検出エラー及び業務エラーのエラー情報を保持するJavaBeanオブジェクトを生成する。
        | 上記例では、以降で説明する業務エラーのハンドリング処理と同じ処理となるので、共通メソッドとして本メソッドを作成している。

|

* リソースが見つからない場合、以下のようなエラー応答が行われる。

 .. code-block:: guess
    :emphasize-lines: 1, 8

    HTTP/1.1 404 Not Found
    Server: Apache-Coyote/1.1
    X-Track: 5ee563877f3140fd904d0acf52eba398
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Wed, 19 Feb 2014 08:46:18 GMT
    
    {"code":"e.ex.mm.5001","message":"Specified member not found. member id : M000000001"}

|

.. _RESTHowToUseExceptionHandlingForBusinessError:

業務エラー例外のハンドリング実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
ビジネスルールの違反を検知した場合に、業務エラーを応答するための実装例について説明する。

ビジネスルールのチェックはServiceの処理として行い、ビジネスルールの違反を検知した場合は、業務例外を発生させる。
業務エラーの検知方法については、「\ :ref:`service-return-businesserrormessage-label`\」を参照されたい。

* エラーハンドリングを行うクラスに、業務例外をハンドリングするためのメソッドを作成する。

 .. code-block:: java
    :emphasize-lines: 6-7, 11

    @ControllerAdvice
    public class ApiGlobalExceptionHandler extends ResponseEntityExceptionHandler {

        // omitted

        // (1)
        @ExceptionHandler(BusinessException.class)
        public ResponseEntity<Object> handleBusinessException(BusinessException ex,
                WebRequest request) {
            return handleResultMessagesNotificationException(ex, new HttpHeaders(),
                    HttpStatus.CONFLICT, request);
        }

        // omitted

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``BusinessException``\をハンドリングするためのメソッドを追加する。
        | メソッドアノテーションとして\ ``@ExceptionHandler(BusinessException.class)``\を指定すると、\ ``BusinessException``\の例外をハンドリングする事ができる。
        | 上記例では、\ ``BusinessException``\の親クラス(\ ``ResultMessagesNotificationException``\)の例外をハンドリングするメソッドに処理を委譲している。
        |
        | ステータスコードには\ **409(Conflict)**\を設定し、クライアントから指定されたリソース自体には不備はないが、サーバで保持しているリソースを操作するための条件が全て整っていない事を通知する。

|

* 業務エラーが発生した場合、以下のようなエラー応答が行われる。

 .. code-block:: guess
    :emphasize-lines: 1, 8

    HTTP/1.1 409 Conflict
    Server: Apache-Coyote/1.1
    X-Track: 37c1a899d5f74e7a9c24662292837ef7
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Wed, 19 Feb 2014 09:03:26 GMT
    
    {"code":"e.ex.mm.8001","message":"Cannot use specified sign id. sign id : user1@test.com"}

|

.. _RESTHowToUseExceptionHandlingForExcusionError:

排他エラー例外のハンドリング実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 排他エラーが発生した場合に、排他エラーを応答するための実装例について説明する。
| 排他制御を行う場合は、排他エラーのハンドリングが必要となる。
| 排他制御の詳細については、「\ :doc:`../DataAccessDetail/ExclusionControl`\」を参照されたい。

* エラーハンドリングを行うクラスに、排他エラーをハンドリングするためのメソッドを作成する。

 .. code-block:: java
    :emphasize-lines: 6-8, 12

    @ControllerAdvice
    public class ApiGlobalExceptionHandler extends ResponseEntityExceptionHandler {
    
        // omitted

        // (1)
        @ExceptionHandler({ OptimisticLockingFailureException.class,
                PessimisticLockingFailureException.class })
        public ResponseEntity<Object> handleLockingFailureException(Exception ex,
                WebRequest request) {
            return handleExceptionInternal(ex, null, new HttpHeaders(),
                    HttpStatus.CONFLICT, request);
        }
    
        // omitted
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 排他エラー(\ ``OptimisticLockingFailureException``\と\ ``PessimisticLockingFailureException``\)をハンドリングするためのメソッドを追加する。
        | メソッドアノテーションとして\ ``@ExceptionHandler({ OptimisticLockingFailureException.class, PessimisticLockingFailureException.class })``\を指定すると、排他エラー(\ ``OptimisticLockingFailureException``\と\ ``PessimisticLockingFailureException``\)の例外をハンドリングする事ができる。
        |
        | ステータスコードには\ **409(Conflict)**\を設定し、クライアントから指定されたリソース自体には不備はないが、処理が競合したためリソースを操作するための条件を満たすことが出来なかった事を通知する。

|

* 排他エラーが発生した場合、以下のようなエラー応答が行われる。

 .. code-block:: guess
    :emphasize-lines: 1, 8

    HTTP/1.1 409 Conflict
    Server: Apache-Coyote/1.1
    X-Track: 85200b5a51be42b29840e482ee35087f
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Wed, 19 Feb 2014 16:32:45 GMT
    
    {"code":"e.ex.fw.8002","message":"Conflict with other processing occurred."}

|

.. _RESTHowToUseExceptionHandlingForSystemError:

システムエラー例外のハンドリング実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
システム異常を検知した場合に、システムエラーを応答するための実装例について説明する。

システム異常の検知した場合は、システム例外を発生させる。
システムエラーの検知方法については、「\ :ref:`service-return-systemerrormessage-label`\」を参照されたい。

* エラーハンドリングを行うクラスに、システム例外をハンドリングするためのメソッドを作成する。

 .. code-block:: java
    :emphasize-lines: 6-7, 11

    @ControllerAdvice
    public class ApiGlobalExceptionHandler extends ResponseEntityExceptionHandler {
    
        // omitted

        // (1)
        @ExceptionHandler(Exception.class)
        public ResponseEntity<Object> handleSystemError(Exception ex,
                WebRequest request) {
            return handleExceptionInternal(ex, null, new HttpHeaders(),
                    HttpStatus.INTERNAL_SERVER_ERROR, request);
        }
    
        // omitted
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``Exception``\をハンドリングするためのメソッドを追加する。
        | メソッドアノテーションとして\ ``@ExceptionHandler(Exception.class)``\を指定すると、\ ``Exception``\の例外をハンドリングする事ができる。
        | 上記例では、使用している依存ライブラリから発生するシステム例外もハンドリング対象としている。
        |
        | ステータスコードには\ **500(Internal Server Error)**\を設定する。

|

* システムエラーが発生した場合、以下のようなエラー応答が行われる。

 .. code-block:: guess
    :emphasize-lines: 1, 9

    HTTP/1.1 500 Internal Server Error
    Server: Apache-Coyote/1.1
    X-Track: 3625d5a040a744e49b0a9b3763a24e9c
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Wed, 19 Feb 2014 12:22:33 GMT
    Connection: close
    
    {"code":"e.ex.fw.9003","message":"System error occurred."}


 .. warning:: **システムエラー時のエラーメッセージについて**

    システムエラーが発生した場合、クライアントへ返却するメッセージは、エラー原因が特定されないシンプルなエラーメッセージを設定することを推奨する。
    エラー原因が特定できるメッセージを設定してしまうと、システムの脆弱性をクライアントに公開する可能性があり、セキュリティー上問題がある。
    
    エラー原因は、エラー解析用にログに出力する。
    Blankプロジェクトのデフォルトの設定では、共通ライブラリから提供している\ ``ExceptionLogger``\によってログが出力されるようなっているため、ログを出力するための設定や実装は不要である。

|

.. _RESTHowToUseExceptionHandlingSettingsOfExceptionCodeResolver:

ExceptionCodeResolverを使ったエラーコードとメッセージの解決
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 共通ライブラリより提供している\ ``ExceptionCodeResolver``\を使用すると、例外クラスからエラーコードを解決する事ができる。
| 特に、エラー原因がクライアント側にある場合は、エラー原因に応じたエラーメッセージを設定する事が求められるケースがあるため、そのような場合に便利な機能である。

- | :file:`applicationContext.xml`
  | 例外クラスとエラーコード(例外コード)のマッピングを行う。

 .. code-block:: xml
    
    <!-- omitted -->

    <bean id="exceptionCodeResolver"
        class="org.terasoluna.gfw.common.exception.SimpleMappingExceptionCodeResolver">
        <property name="exceptionMappings">
            <map>
                <!-- omitted -->
                <entry key="ResourceNotFoundException"              value="e.ex.fw.5001" />
                <entry key="HttpRequestMethodNotSupportedException" value="e.ex.fw.6001" />
                <entry key="MediaTypeNotAcceptableException"        value="e.ex.fw.6002" />
                <entry key="HttpMediaTypeNotSupportedException"     value="e.ex.fw.6003" />
                <entry key="MethodArgumentNotValidException"        value="e.ex.fw.7001" />
                <entry key="BindException"                          value="e.ex.fw.7002" />
                <entry key="JsonParseException"                     value="e.ex.fw.7003" />
                <entry key="UnrecognizedPropertyException"          value="e.ex.fw.7004" />
                <entry key="JsonMappingException"                   value="e.ex.fw.7005" />
                <entry key="TypeMismatchException"                  value="e.ex.fw.7006" />
                <entry key="BusinessException"                      value="e.ex.fw.8001" />
                <entry key="OptimisticLockingFailureException"      value="e.ex.fw.8002" />
                <entry key="PessimisticLockingFailureException"     value="e.ex.fw.8002" />
                <entry key="DataAccessException"                    value="e.ex.fw.9002" />
                <!-- omitted -->
            </map>
        </property>
        <property name="defaultExceptionCode" value="e.ex.fw.9001" />
    </bean>

    <!-- omitted -->

|

| エラーコードに対応するメッセージの設定例を以下に示す。
| メッセージの管理方法については、「\ :doc:`../WebApplicationDetail/MessageManagement`\」を参照されたい。

- | :file:`xxx-web/src/main/resources/i18n/application-messages.properties`
  | アプリケーション層で発生するエラーに対して、エラーコード(例外コード)に対応するメッセージの設定を行う。

 .. code-block:: properties

    # ---
    # Application common messages
    # ---
    e.ex.fw.5001 = Resource not found.
    
    e.ex.fw.6001 = Request method not supported.
    e.ex.fw.6002 = Specified representation format not supported.
    e.ex.fw.6003 = Specified media type in the request body not supported.
    
    e.ex.fw.7001 = Validation error occurred on item in the request body.
    e.ex.fw.7002 = Validation error occurred on item in the request parameters.
    e.ex.fw.7003 = Request body format error occurred.
    e.ex.fw.7004 = Unknown field exists in JSON.
    e.ex.fw.7005 = Type mismatch error occurred in JSON field.
    e.ex.fw.7006 = Type mismatch error occurred in request parameter or header or path variable.
    
    e.ex.fw.8001 = Business error occurred.
    e.ex.fw.8002 = Conflict with other processing occurred.
    
    e.ex.fw.9001 = System error occurred.
    e.ex.fw.9002 = System error occurred.
    e.ex.fw.9003 = System error occurred.

    # omitted

- | :file:`xxx-web/src/main/resources/ValidationMessages.properties`
  | Bean Validationを使った入力チェックで発生するエラーに対して、エラーコードに対応するメッセージの設定を行う。

 .. code-block:: properties

    # ---
    # Bean Validation common messages
    # ---
    
    # for bean validation of standard
    javax.validation.constraints.AssertFalse.message = "{0}" must be false.
    javax.validation.constraints.AssertTrue.message  = "{0}" must be true.
    javax.validation.constraints.DecimalMax.message  = "{0}" must be less than ${inclusive == true ? 'or equal to ' : ''}{value}.
    javax.validation.constraints.DecimalMin.message  = "{0}" must be greater than ${inclusive == true ? 'or equal to ' : ''}{value}.
    javax.validation.constraints.Digits.message      = "{0}" numeric value out of bounds (<{integer} digits>.<{fraction} digits> expected).
    javax.validation.constraints.Future.message      = "{0}" must be in the future.
    javax.validation.constraints.Max.message         = "{0}" must be less than or equal to {value}.
    javax.validation.constraints.Min.message         = "{0}" must be greater than or equal to {value}.
    javax.validation.constraints.NotNull.message     = "{0}" may not be null.
    javax.validation.constraints.Null.message        = "{0}" must be null.
    javax.validation.constraints.Past.message        = "{0}" must be in the past.
    javax.validation.constraints.Pattern.message     = "{0}" must match "{regexp}".
    javax.validation.constraints.Size.message        = "{0}" size must be between {min} and {max}.
    
    # for bean validation of hibernate
    org.hibernate.validator.constraints.CreditCardNumber.message        = "{0}" invalid credit card number.
    org.hibernate.validator.constraints.EAN.message                     = "{0}" invalid {type} barcode.
    org.hibernate.validator.constraints.Email.message                   = "{0}" not a well-formed email address.
    org.hibernate.validator.constraints.Length.message                  = "{0}" length must be between {min} and {max}.
    org.hibernate.validator.constraints.LuhnCheck.message               = "{0}" The check digit for ${validatedValue} is invalid, Luhn Modulo 10 checksum failed.
    org.hibernate.validator.constraints.Mod10Check.message              = "{0}" The check digit for ${validatedValue} is invalid, Modulo 10 checksum failed.
    org.hibernate.validator.constraints.Mod11Check.message              = "{0}" The check digit for ${validatedValue} is invalid, Modulo 11 checksum failed.
    org.hibernate.validator.constraints.ModCheck.message                = "{0}" The check digit for ${validatedValue} is invalid, ${modType} checksum failed.
    org.hibernate.validator.constraints.NotBlank.message                = "{0}" may not be empty.
    org.hibernate.validator.constraints.NotEmpty.message                = "{0}" may not be empty.
    org.hibernate.validator.constraints.ParametersScriptAssert.message  = "{0}" script expression "{script}" didn't evaluate to true.
    org.hibernate.validator.constraints.Range.message                   = "{0}" must be between {min} and {max}.
    org.hibernate.validator.constraints.SafeHtml.message                = "{0}" may have unsafe html content.
    org.hibernate.validator.constraints.ScriptAssert.message            = "{0}" script expression "{script}" didn't evaluate to true.
    org.hibernate.validator.constraints.URL.message                     = "{0}" must be a valid URL.

    org.hibernate.validator.constraints.br.CNPJ.message                 = "{0}" invalid Brazilian corporate taxpayer registry number (CNPJ).
    org.hibernate.validator.constraints.br.CPF.message                  = "{0}" invalid Brazilian individual taxpayer registry number (CPF).
    org.hibernate.validator.constraints.br.TituloEleitoral.message      = "{0}" invalid Brazilian Voter ID card number.

    # for common library
    org.terasoluna.gfw.common.codelist.ExistInCodeList.message   = "{0}" must exist in code list of {codeListId}.

- | :file:`xxx-domain/src/main/resources/i18n/domain-messages.properties`
  | ドメイン層で発生するエラーに対して、エラーコード(例外コード)に対応するメッセージの設定を行う。

 .. code-block:: properties

    # omitted

    e.ex.mm.5001 = Specified member not found. member id : {0}
    e.ex.mm.8001 = Cannot use specified sign id. sign id : {0}

    # omitted

|


.. _RESTHowToUseExceptionHandlingForServletContainer:

サーブレットコンテナに通知されたエラーのハンドリングの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Filterでエラーが発生した場合や\ ``HttpServletResponse#sendError``\を使ってエラーレスポンスが行われた場合は、Spring MVCの例外ハンドリングの仕組みを使ってハンドリングできないため、
これらのエラーはサーブレットコンテナに通知される。

本節では、サーブレットコンテナに通知されたエラーをハンドリングする方法について説明する。

 .. figure:: ./images_REST/RESTHowToUseErrorHandlingByServletContainer.png
   :alt: Image of error handling processing by servlet container
   :width: 100%


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 70

    * - 項番
      - 処理レイヤ
      - 説明
    * - | (1)
      - | Servlet Container
        | (AP Server)
      - | Servlet Containerはクライアントからのリクエストを受け付け、処理を行う。
        | Servlet Containerは処理中にエラーを検知する。
    * - | (2)
      - | 
      - | Servlet Containerは\ ``web.xml``\のerror-pageの定義に従って、エラー処理を行う。
        | 致命的なエラーでない場合は、エラーハンドリングを行うControllerを呼び出し、エラー処理を行う。
    * - | (2')
      - | 
      - | 致命的なエラーの場合は、予め用意してある静的なJSONファイルを取得し、クライアントへ応答する。
    * - | (3)
      - | Spring MVC
        | (Framework)
      - | Spring MVCは、エラーハンドリングを行うControllerを呼び出す。
    * - | (4)
      - | Controller
        | (Common Component)
      - | エラーハンドリングを行うControllerでは、エラー情報を保持するエラーオブジェクトを生成し、Spring MVCに返却する。
    * - | (5)
      - | Spring MVC
        | (Framework)
      - | Spring MVCは、\ ``HttpMessageConverter``\を利用して、エラーオブジェクトをJSON形式の電文に変換する。
    * - | (6)
      - | 
      - | Spring MVCは、JSON形式のエラー電文をレスポンスBODYに設定し、クライアントにレスポンスする。

|

エラー応答を行うためのControllerの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
サーブレットコンテナに通知されたエラーのエラー応答を行うControllerを作成する。

 .. code-block:: java
    :emphasize-lines: 17-20, 22-23, 25, 28, 33-35, 36, 40, 42, 45

    package org.terasoluna.examples.rest.api.common.error;
    
    import java.util.HashMap;
    import java.util.Map;
    
    import javax.inject.Inject;
    import javax.servlet.RequestDispatcher;
    
    import org.springframework.http.HttpStatus;
    import org.springframework.http.MediaType;
    import org.springframework.http.ResponseEntity;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RestController;
    import org.springframework.web.context.request.RequestAttributes;
    import org.springframework.web.context.request.WebRequest;
    
    // (1)
    @RequestMapping("error")
    @RestController
    public class ApiErrorPageController {
    
        @Inject
        ApiErrorCreator apiErrorCreator; // (2)
    
        // (3)
        private final Map<HttpStatus, String> errorCodeMap = new HashMap<HttpStatus, String>();
    
        // (4)
        public ApiErrorPageController() {
            errorCodeMap.put(HttpStatus.NOT_FOUND, "e.ex.fw.5001");
        }
    
        // (5)
        @RequestMapping
        public ResponseEntity<ApiError> handleErrorPage(WebRequest request) {
            // (6)
            HttpStatus httpStatus = HttpStatus.valueOf((Integer) request
                    .getAttribute(RequestDispatcher.ERROR_STATUS_CODE,
                            RequestAttributes.SCOPE_REQUEST));
            // (7)
            String errorCode = errorCodeMap.get(httpStatus);
            // (8)
            ApiError apiError = apiErrorCreator.createApiError(request, errorCode,
                    httpStatus.getReasonPhrase());
            // (9)
            return ResponseEntity.status(httpStatus).body(apiError);
        }
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | エラー応答を行うためのControllerクラスを作成する。
        | 上記例では、「\ ``/api/v1/error``\」というサーブレットパスにマッピングしている。
    * - | (2)
      - | エラー情報を作成するクラスをInjectする。
    * - | (3)
      - | HTTPステータスコードとエラーコードをマッピングするための \ ``Map``\ を作成する。
    * - | (4)
      - | HTTPステータスコードとエラーコードとのマッピングを登録する。
    * - | (5)
      - | エラー応答を行うハンドラメソッドを作成する。
        | 上記例では、レスポンスコード(\ ``<error-code>``\)を使ってエラーページのハンドリングを行うケースのみを考慮した実装になっている。
        | したがって、例外タイプ(\ ``<exception-type>``\)を使ってハンドリングしたエラーページの処理を本メソッドを使って行う場合は、別途考慮が必要である。
    * - | (6)
      - | リクエストスコープに格納されているステータスコードを取得する。
    * - | (7)
      - | 取得したステータスコードに対応するエラーコードを取得する。
    * - | (8)
      - | 取得したエラーコードに対応するエラー情報を生成する。
    * - | (9)
      - | (8)で生成したエラー情報を応答する。

| 

致命的なエラーが発生した際に応答する静的なJSONファイルの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
致命的なエラーが発生した際に応答する静的なJSONファイルを作成する。

- :file:`unhandledSystemError.json`

 .. code-block:: json
    
    {"code":"e.ex.fw.9999","message":"Unhandled system error occurred."}

|

サーブレットコンテナに通知されたエラーをハンドリングするための設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
ここでは、サーブレットコンテナに通知されたエラーをハンドリングするための設定について説明する。

- :file:`web.xml`

 .. code-block:: xml
    :emphasize-lines: 3, 9, 15

    <!-- omitted -->

    <!-- (1) -->
    <error-page>
        <error-code>404</error-code>
        <location>/api/v1/error</location>
    </error-page>

    <!-- (2) -->
    <error-page>
        <exception-type>java.lang.Exception</exception-type>
        <location>/WEB-INF/views/common/error/unhandledSystemError.json</location>
    </error-page>

    <!-- (3) -->
    <mime-mapping>
        <extension>json</extension>
        <mime-type>application/json;charset=UTF-8</mime-type>
    </mime-mapping>

    <!-- omitted -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 必要に応じてレスポンスコードによるエラーページの定義を追加する。
        | 上記例では、\ ``"404 Not Found"``\が発生した際に、「\ ``/api/v1/error``\」というリクエストにマッピングされているController(\ ``ApiErrorPageController``\)を呼び出してエラー応答を行っている。
    * - | (2)
      - | 致命的なエラーをハンドリングするための定義を追加する。
        | 致命的なエラーが発生していた場合、レスポンス情報を作成する処理で二重障害が発生する可能性があるため、予め用意している静的なJSONを応答する事を推奨する。
        | 上記例では、「\ ``/WEB-INF/views/common/error/unhandledSystemError.json``\」に定義されている固定のJSONを応答している。
    * - | (3)
      - | jsonのMIMEタイプを指定する。
        | (2)で作成するJSONファイルの中にマルチバイト文字を含める場合は、\ ``charset=UTF-8``\を指定しないと、クライアント側で文字化けする可能性がある。
        | JSONファイルにマルチバイト文字を含めない場合は、この設定は必須ではないが、設定しておいた方が無難である。

 .. note::
    
    `Servletの仕様 <http://download.oracle.com/otn-pub/jcp/servlet-3_1-fr-spec/servlet-3_1-final.pdf#page=128>`_\ では、 \ ``<error-page>``\ の \ ``<location>``\ にクエリパラメータを付与したパスを指定した場合の挙動について、定義していない。そのため、APサーバによって挙動が異なる可能性がある。
    よって、クエリパラメータを使用してエラー時の遷移先に情報を渡すことは推奨しない。

|

* 存在しないパスへリクエストを送った場合、以下のようなエラー応答が行われる。

 .. code-block:: guess
    :emphasize-lines: 1, 8

    HTTP/1.1 404 Not Found
    Server: Apache-Coyote/1.1
    X-Track: 2ad50fb5ba2441699c91a5b01edef83f
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Wed, 19 Feb 2014 23:24:20 GMT
    
    {"code":"e.ex.fw.5001","message":"Resource not found."}

|

* 致命的なエラーが発生した場合、以下のようなエラー応答が行われる。

 .. code-block:: guess
    :emphasize-lines: 1, 9

    HTTP/1.1 500 Internal Server Error
    Server: Apache-Coyote/1.1
    X-Track: 69db3854a19f439781584321d9ce8336
    Content-Type: application/json
    Content-Length: 68
    Date: Thu, 20 Feb 2014 00:13:43 GMT
    Connection: close
    
    {"code":"e.ex.fw.9999","message":"Unhandled system error occurred."}

|

.. _RESTHowToUseSecurity:

セキュリティ対策
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| RESTful Web Serviceに対するセキュリティ対策の実現方法について説明する。
| 本ガイドラインでは、セキュリティ対策の実現方法として、Spring Securityを使用する事を推奨している。

.. _RESTHowToUseSecurityAuth:

認証・認可
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. todo:: **TBD**

    OAuth2(Spring Security OAuth2)を使用して認証・認可を実現する方法について、次版以降に記載する予定である。

|

.. _RESTHowToUseSecurityCsrf:

CSRF対策
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* RESTful Web Serviceに対してCSRF対策を行う場合の設定方法については、\ :ref:`CSRF対策 <csrf_spring-security-setting>`\ を参照されたい。

* RESTful Web Serviceに対してCSRF対策を行わない場合の設定方法については、:ref:`RESTAppendixDisabledCSRFProtection`\ を参照されたい。


|

.. _RESTHowToUseConditionalOperations:

リソースの条件付き操作
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo:: **TBD**

    Etagなどのヘッダを使った条件付き処理の制御の実現方法について、次版以降に記載する予定である。


|

.. _RESTHowToUseCacheControl:

リソースのキャッシュ制御
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo:: **TBD**

    Cache-Control/Expires/Pragmaなどのヘッダを使ったキャッシュ制御の実現方法について、次版以降に記載する予定である。

|


.. _RESTHowToExtend:

How to extend
--------------------------------------------------------------------------------



.. _RESTAppendixJsonview:

@JsonViewを使用したレスポンスの出力制御
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| \ ``@JsonView``\ を使用することによって、Resourceオブジェクト内のプロパティーをグループ分けすることができる。
| この機能はSpring FrameworkがJacksonの機能をサポートすることにより実現している。
| 詳細は、\ `JacksonJsonViews <http://wiki.fasterxml.com/JacksonJsonViews>`_\ を参照されたい。

| Controllerにてグループを指定することで、指定したグループに所属するプロパティーのみ出力することができる。
| 1つのプロパティーは、複数のグループに所属することも可能である。
|
| 以下は、Memberリソースを「概要」と「詳細」の２つのフォーマットで扱う際の実装例である。
| 「概要フォーマット」はMemberリソースの主要項目を、「詳細フォーマット」はMemberリソースの全項目を出力する。

* :file:`MemberResource.java`

 .. code-block:: java

    package org.terasoluna.examples.rest.api.member;
    
    import java.io.Serializable;
    
    import org.joda.time.DateTime;
    import org.joda.time.LocalDate;

    import com.fasterxml.jackson.annotation.JsonView;
    
    public class MemberResource implements Serializable {
    
        private static final long serialVersionUID = 1L;

        // (1)
        interface Summary {
        }

        // (2)
        interface Detail {
        }

        // (3)
        @JsonView({Summary.class, Detail.class})
        private String memberId;

        @JsonView({Summary.class, Detail.class})
        private String firstName;

        @JsonView({Summary.class, Detail.class})
        private String lastName;

        // (4)
        @JsonView(Detail.class)
        private String genderCode;

        @JsonView(Detail.class)
        private LocalDate dateOfBirth;

        @JsonView(Detail.class)
        private String emailAddress;

        @JsonView(Detail.class)
        private String telephoneNumber;

        @JsonView(Detail.class)
        private String zipCode;

        @JsonView(Detail.class)
        private String address;

        // (5)
        private DateTime createdAt;

        private DateTime lastModifiedAt;

        // omitted setter and getter
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 出力制御するグループを指定するためのマーカーインターフェースを定義している。
        | 上記例では、概要出力時に指定するグループを定義している。
    * - | (2)
      - | 出力制御するグループを指定するためのマーカーインターフェースを定義している。
        | 上記例では、詳細出力時に指定するグループを定義している。
    * - | (3)
      - | 複数のグループで出力したい項目には、引数を配列にして複数のマーカーインターフェースを渡すことで、複数のグループに所属させることができる。
        | 上記例の場合、概要と詳細の両方のグループに所属させたい項目であるため、2つのマーカーインターフェースを引数にしている。
    * - | (4)
      - | 単一のグループで出力したい項目には、マーカーインターフェースを引数にすることで、
        | 該当のグループに所属させることができる。
        | この場合は要素が1つため、配列にする必要はない。
        | 上記例の場合、詳細のみのグループに所属させたい項目であるため、1つのマーカーインターフェースを引数にしている。
    * - | (5)
      - | グループに所属しない項目には\ ``@JsonView``\ を設定しない。
        | グループに所属しない項目を出力するかどうかは設定によって変えることができる。
        | 設定方法については後述する。

|


* :file:`MemberRestController.java`

 .. code-block:: java

    package org.terasoluna.examples.rest.api.member;
    
    import java.util.ArrayList;
    import java.util.List;
    
    import javax.inject.Inject;
    
    import org.dozer.Mapper;
    import org.springframework.http.HttpStatus;
    import org.springframework.web.bind.annotation.PathVariable;
    import org.springframework.web.bind.annotation.RequestBody;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.bind.annotation.ResponseStatus;
    import org.springframework.web.bind.annotation.RestController;
    import org.terasoluna.examples.rest.domain.model.Member;
    import org.terasoluna.examples.rest.domain.service.member.MemberService;
    
    import com.fasterxml.jackson.annotation.JsonView;
    
    @RequestMapping("members")
    @RestController
    public class MemberRestController {
    
        @Inject
        MemberService memberService;
    
        @Inject
        Mapper beanMapper;
        
        // (1)
        @JsonView(Summary.class)
        @RequestMapping(value = "{memberId}", params = "format=summary", method = RequestMethod.GET)
        @ResponseStatus(HttpStatus.OK)
        public MemberResource getMemberSummary(@PathVariable("memberId") String memberId) {

            Member member = memberService.getMember(memberId);

            MemberResource responseResource = beanMapper.map(member,
                    MemberResource.class);

            return responseResource;
        }
        
        // (2)
        @JsonView(Detail.class)
        @RequestMapping(value = "{memberId}", params = "format=detail", method = RequestMethod.GET)
        @ResponseStatus(HttpStatus.OK)
        public MemberResource getMemberDetail(@PathVariable("memberId") String memberId) {

            Member member = memberService.getMember(memberId);

            MemberResource responseResource = beanMapper.map(member,
                    MemberResource.class);

            return responseResource;
        }
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@JsonView``\ を付けて、出力したいグループのマーカーインターフェースを設定する。
        | 概要を出力するメソッドに\ ``Summary``\ マーカーインターフェースを設定する。
    * - | (2)
      - | 詳細を出力するメソッドに\ ``Detail``\ マーカーインターフェースを設定する。


|

| 出力されるボディは、Controllerで指定したグループに所属するプロパティーのみ出力される。
| 出力例は以下の通りとなる。

* Summary

 .. code-block:: java

    {
      "memberId" : "M000000001",
      "firstName" : "John",
      "lastName" : "Smith",
      "createdAt" : "2014-03-14T11:02:41.477Z",
      "lastModifiedAt" : "2014-03-14T11:02:41.477Z"
    }


|

* Detail

 .. code-block:: java

    {
      "memberId" : "M000000001",
      "firstName" : "John",
      "lastName" : "Smith",
      "genderCode" : "1",
      "dateOfBirth" : "2013-03-14",
      "emailAddress" : "user1394794959984@test.com",
      "telephoneNumber" : "09012345678",
      "zipCode" : "1710051",
      "address" : "Tokyo",
      "createdAt" : "2014-03-14T11:02:41.477Z",
      "lastModifiedAt" : "2014-03-14T11:02:41.477Z"
    }


|

| \ ``@JsonView``\ を付けなかったプロパティーは、\ ``MapperFeature.DEFAULT_VIEW_INCLUSION``\ の設定を有効にすれば出力され、無効にすれば出力されない。
| 上記の出力例は、\ ``MapperFeature.DEFAULT_VIEW_INCLUSION``\ を有効にした場合の出力例である。
| \ ``MapperFeature.DEFAULT_VIEW_INCLUSION``\ を有効にする場合は、以下のように設定する。

 .. code-block:: xml
 
   <bean id="objectMapper" class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">
       <!-- ... -->
       
       <!-- (1) -->
       <property name="featuresToEnable">
           <array>
               <util:constant static-field="com.fasterxml.jackson.databind.MapperFeature.DEFAULT_VIEW_INCLUSION"/>
           </array>
       </property>
   </bean>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``featuresToEnable``\ 要素に\ ``MapperFeature.DEFAULT_VIEW_INCLUSION``\ を定義することで設定が有効となる。
    

|


| \ ``MapperFeature.DEFAULT_VIEW_INCLUSION``\ を無効にする場合は、以下のように設定する。

 .. code-block:: xml
 
   <bean id="objectMapper" class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">
       <!-- ... -->
       
       <!-- (1) -->
       <property name="featuresToDisable">
           <array>
               <util:constant static-field="com.fasterxml.jackson.databind.MapperFeature.DEFAULT_VIEW_INCLUSION"/>
           </array>
       </property>
   </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``featuresToDisable``\ 要素に\ ``MapperFeature.DEFAULT_VIEW_INCLUSION``\ を定義することで設定が無効となる。
    

| \ ``MapperFeature.DEFAULT_VIEW_INCLUSION``\ が無効の場合、先ほどの出力例は、以下のように出力内容が変更される。

* Summary

 .. code-block:: java

    {
      "memberId" : "M000000001",
      "firstName" : "John",
      "lastName" : "Smith"
    }


|

* Detail

 .. code-block:: java

    {
      "memberId" : "M000000001",
      "firstName" : "John",
      "lastName" : "Smith",
      "genderCode" : "1",
      "dateOfBirth" : "2013-03-14",
      "emailAddress" : "user1394794959984@test.com",
      "telephoneNumber" : "09012345678",
      "zipCode" : "1710051",
      "address" : "Tokyo"
    }

|
    
    
.. warning::
 
    \ ``MapperFeature.DEFAULT_VIEW_INCLUSION``\ を指定しない場合のデフォルト値は、\ ``ObjectMapper``\ の設定方法によって異なるデフォルト値となるため注意が必要である。
    \ :ref:`RESTHowToUseApplicationSettingsOfSpringMVC`\ でも記述しているが、\ ``ObjectMapper``\ のBean定義方法を\ ``ObjectMapper``\ を直接Bean定義するスタイルにすると、デフォルト値が有効になる。\ ``Jackson2ObjectMapperFactoryBean``\ を利用すると、デフォルト値は無効になる。設定を明示するため、どちらのスタイルで設定する場合においても、\ ``MapperFeature.DEFAULT_VIEW_INCLUSION``\ の指定を記述することを推奨する。
    


    
.. note::
  \ ``@JsonView``\ は以下の2つの機能を使用して作成されている。これらは、Controller内の\ ``@RequestMapping``\ が付けられた処理メソッドで、Objectとのマッピング前後に共通的な処理を実装したい場合に、使用することができる機能である。
  
  * \ ``org.springframework.web.servlet.mvc.method.annotation.RequestBodyAdvice``\ 
  * \ ``org.springframework.web.servlet.mvc.method.annotation.ResponseBodyAdvice``\

  \ ``@ControllerAdvice``\ をこれらのインタフェースの実装クラスにつけることで適用することができる。\ ``@ControllerAdvice``\ の詳細は、\ :ref:`application_layer_controller_advice`\を参照されたい。
  
  \ ``RequestBodyAdvice``\ は下記のメソッドを実装することができる。


  * :file:`org.springframework.web.servlet.mvc.method.annotation.RequestBodyAdvice`

   .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 20 70
      :class: longtable

      * - 項番
        - メソッド名
        - 概要
      * - | (1)
        - | supports
        - | このAdviceが送信されたリクエストに対して適用されるかどうか決定する。\ ``true``\ だと適用される。
      * - | (2)
        - | handleEmptyBody
        - | リクエストボディの内容をControllerで使用するオブジェクトに反映する前かつ、ボディが空の場合に呼び出される。
      * - | (3)
        - | beforeBodyRead
        - | リクエストボディの内容をControllerで使用するオブジェクトに反映する前に呼び出される。
      * - | (4)
        - | afterBodyRead
        - | リクエストボディの内容をControllerで使用するオブジェクトに反映した後に呼び出される。

  上記すべてのタイミングで処理を記述する必要がない場合は、上記のsupports以外のメソッドが何もしない状態で実装された\ ``org.springframework.web.servlet.mvc.method.annotation.RequestBodyAdviceAdapter``\ を継承し、必要な部分だけオーバーライドすることで、簡単に実装することができる。




  \ ``ResponseBodyAdvice``\ は下記のメソッドを実装することができる。

  * :file:`org.springframework.web.servlet.mvc.method.annotation.ResponseBodyAdvice`

   .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 20 70
      :class: longtable

      * - 項番
        - メソッド名
        - 概要
      * - | (1)
        - | supports
        - | このAdviceが送信されたリクエストに対して適用されるかどうか決定する。\ ``true``\ だと適用される。
      * - | (2)
        - | beforeBodyWrite
        - | Contorollerでの処理終了後、レスポンスに返り値を反映する前に呼び出される。

|


.. _RESTAppendix:

Appendix
--------------------------------------------------------------------------------

.. _RESTAppendixUsingJSR310_JodaTime:

JSR-310 Date and Time API / Joda Timeを使う場合の設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

リソースを表現するJavaBean(Resourceクラス)のプロパティとしてJSR-310 Date and Time APIを使用する場合は、
terasoluna-gfw-common-dependenciesにて依存関係が定義されているため依存関係の追加は不要である。
一方、Joda Timeのクラスを使用する場合は、
\ ``pom.xml``\ にJacksonから提供されている拡張モジュールを依存ライブラリに追加する。


**Joda Timeのクラスを使用する場合**

.. code-block:: xml

    <dependency>
        <groupId>org.terasoluna.gfw</groupId>
        <artifactId>terasoluna-gfw-jodatime-dependencies</artifactId>
        <type>pom</type>
    </dependency>

or

.. code-block:: xml

    <dependency>
        <groupId>com.fasterxml.jackson.datatype</groupId>
        <artifactId>jackson-datatype-joda</artifactId>
    </dependency>

.. note::
    上記設定例は、依存ライブラリのバージョンを親プロジェクトである terasoluna-gfw-parent で管理する前提であるため、pom.xmlでのバージョンの指定は不要である。
    上記のjackson-datatype-jodaはterasoluna-gfw-parentが利用している\ `Spring IO Platform <http://platform.spring.io/platform/>`_\ で定義済みである。



    上記以外にも、

    * Java SE 7から追加された\ ``java.nio.file.Path``\
    * Java SE 8から追加された\ ``java.util.Optional``\
    * Hibernate ORMのLazy Load機能によってProxy化されたオブジェクト

    などを扱うための拡張モジュール(\ ``jackson-datatype-xxx``\ )が、別途Jacksonから提供されている。

|

.. _RESTAppendixSettingsOfDeployInSameWarFileRestAndClientApplication:

RESTful Web Serviceとクライアントアプリケーションを同じWebアプリケーションとして動かす際の設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _RESTAppendixDivideDispatcherServlet:

RESTful Web Service用の\ ``DispatcherServlet``\を設ける方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| RESTful Web Serviceとクライアントアプリケーションを同じWebアプリケーションとして構築する場合、RESTful Web Service用のリクエストを受ける\ ``DispatcherServlet``\と、クライアントアプリケーション用のリクエストを受け取る\ ``DispatcherServlet``\を分割する事を推奨する。
| \ ``DispatcherServlet``\を分割する方法について、以下に説明する。

- :file:`web.xml`

 .. code-block:: xml
    :emphasize-lines: 3,17-19,19-20,24-25,29-33

    <!-- omitted -->

    <!-- (1) -->
    <servlet>
        <servlet-name>appServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath*:META-INF/spring/spring-mvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>appServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!-- (2) -->
    <servlet>
        <servlet-name>restAppServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <!-- (3) -->
            <param-value>classpath*:META-INF/spring/spring-mvc-rest.xml</param-value>
        </init-param>
        <load-on-startup>2</load-on-startup>
    </servlet>
    <!-- (4) -->
    <servlet-mapping>
        <servlet-name>restAppServlet</servlet-name>
        <url-pattern>/api/v1/*</url-pattern>
    </servlet-mapping>

    <!-- omitted -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | クライアントアプリケーション用のリクエストを受け取る\ ``DispatcherServlet``\とリクエストマッピング。
    * - | (2)
      - | RESTful Web Service用のリクエストを受けるServlet(\ ``DispatcherServlet``\)を追加する。
        | \ ``<servlet-name>``\要素に、RESTful Web Service用サーブレットであることを示す名前を指定する。
        | 上記例では、サーブレット名として\ ``"restAppServlet"``\を指定している。
    * - | (3)
      - | RESTful Web Service用の\ ``DispatcherServlet``\を構築する際に使用するSpring MVCのbean定義ファイルを指定する。
        | 上記例では、Spring MVCのbean定義ファイルとして、クラスパス上にある\ :file:`META-INF/spring/spring-mvc-rest.xml`\を指定している。
    * - | (4)
      - | RESTful Web Service用の\ ``DispatcherServlet``\へマッピングするサーブレットパスのパターンの指定を行う。
        | 上記例では、\ ``"/api/v1/"``\配下のサーブレットパスをRESTful Web Service用の\ ``DispatcherServlet``\にマッピングしている。
        | 具体的には、
        |   \ ``"/api/v1/"``\
        |   \ ``"/api/v1/members"``\
        |   \ ``"/api/v1/members/xxxxx"``\
        | といったサーブレットパスが、RESTful Web Service用の\ ``DispatcherServlet``\(\ ``"restAppServlet"``\)にマッピングされる。

 .. tip:: **@RequestMappingアノテーションのvalue属性に指定する値について**

   \ ``@RequestMapping``\アノテーションのvalue属性に指定する値は、\ ``<url-pattern>``\要素で指定したワイルドカード(\ ``*``\)の部分の値を指定する。
   
   例えば、\ ``@RequestMapping(value = "members")``\と指定した場合、\ ``"/api/v1/members"``\といパスに対する処理を行うメソッドとしてデプロイされる。
   そのため、\ ``@RequestMapping``\アノテーションのvalue属性には、分割したサーブレットへマッピングするためパス(\ ``"api/v1"``\)を指定する必要はない。
   
   \ ``@RequestMapping(value = "api/v1/members")``\と指定すると、\ ``"/api/v1/api/v1/members"``\というパスに対する処理を行うメソッドとしてデプロイされてしまうので、注意すること。

|

.. _RESTAppendixHyperMediaLink:

ハイパーメディアリンクの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
JSONの中に関連リソースへのハイパーメディアリンクを含める場合の実装について説明する。

共通部品の実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* リンク情報を保持するJavaBeanを作成する。

 .. code-block:: java
    :emphasize-lines: 5
 
    package org.terasoluna.examples.rest.api.common.resource;
    
    import java.io.Serializable;
    
    // (1)
    public class Link implements Serializable {
    
        private static final long serialVersionUID = 1L;

        private final String rel;

        private final String href;
    
        public Link(String rel, String href) {
            this.rel = rel;
            this.href = href;
        }
    
        public String getRel() {
            return rel;
        }
    
        public String getHref() {
            return href;
        }
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | リンク名とURLを保持するリンク情報用のJavaBeanを作成する。

|

* リンク情報のコレクションを保持するResourceの抽象クラスを作成する。

 .. code-block:: java
    :emphasize-lines: 9, 12, 20, 26, 31

    package org.terasoluna.examples.rest.api.common.resource;
    
    import java.net.URI;
    import java.util.LinkedHashSet;
    import java.util.Set;
    
    import com.fasterxml.jackson.annotation.JsonInclude;
    
    // (2)
    public abstract class AbstractLinksSupportedResource {

        // (3)
        @JsonInclude(JsonInclude.Include.NON_EMPTY)
        private final Set<Link> links = new LinkedHashSet<>();
    
        public Set<Link> getLinks() {
            return links;
        }
    
        // (4)
        public AbstractLinksSupportedResource addLink(String rel, URI href) {
            links.add(new Link(rel, href.toString()));
            return this;
        }
    
        // (5)
        public AbstractLinksSupportedResource addSelf(URI href) {
            return addLink("self", href);
        }
    
        // (5)
        public AbstractLinksSupportedResource addParent(URI href) {
            return addLink("parent", href);
        }
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (2)
      - | リンク情報のコレクションを保持するResourceの抽象クラス(JavaBean)を作成する。
        | 本クラスは、パイパーメディアリンクをサポートするResourceクラスによって、継承される事を想定したクラスである。
    * - | (3)
      - | リンク情報を複数保持するフィールドを定義する。
        | 上記例では、リンクの指定がない時にJSONに出力しないようにするために、\ ``@JsonInclude(JsonInclude.Include.NON_EMPTY)``\を指定している。
    * - | (4)
      - | リンク情報を追加するためのメソッドを用意する。
    * - | (5)
      - | 必要に応じて共通的なリンク情報を追加するためのメソッドを用意する。
        | 上記例では、自身のリソースにアクセスするためのリンク情報(\ ``"self"``\)と、親のリソースにアクセスするためのリンク情報(\ ``"parent"``\)を追加するためのメソッドを用意している。

|


リソース毎の実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* Resourceクラスにて、リンク情報のコレクションを保持するResourceの抽象クラスを継承する。

 .. code-block:: java
    :emphasize-lines: 3

    package org.terasoluna.examples.rest.api.member;
    
    // (1)
    public class MemberResource extends 
        AbstractLinksSupportedResource implements Serializable {

        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | リンク情報のコレクションを保持するResourceの抽象クラスを継承する。
        | 継承することで、リンク情報のコレクションを保持するフィールド(\ ``links``\)が取り込まれ、ハイパーメディアリンクをサポートするResourceクラスとなる。


* REST APIの処理で、ハイパーメディアリンクを追加する。

 .. code-block:: java
    :emphasize-lines: 11, 19

    @RequestMapping("members")
    @RestController
    public class MemberRestController {

        // omitted

        @RequestMapping(value = "{memberId}", method = RequestMethod.GET)
        @ResponseStatus(HttpStatus.OK)
        public MemberResource getMember(
                @PathVariable("memberId") String memberId
                // (2)
                UriComponentsBuilder uriBuilder) {
    
            Member member = memberService.getMember(memberId);
    
            MemberResource responseResource = beanMapper.map(member,
                    MemberResource.class);
    
            // (3)
            responseResource.addSelf(uriBuilder.path("/members").pathSegment(memberId)
                    .build().toUri());
    
            return responseResource;
        }

        // omitted

    }
    
 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (2)
      - | リンク情報に設定するURIを組み立てるため、Spring MVCから提供されている\ ``org.springframework.web.util.UriComponentsBuilder``\クラスをメソッドの引数に指定する。
        | \ ``UriComponentsBuilder``\ クラスをControllerのメソッドの引数に指定すると、メソッド実行時に、Spring MVCにより\ ``UriComponentsBuilder``\ クラスを継承した
        | \ ``org.springframework.web.servlet.support.ServletUriComponentsBuilder``\クラスのインスタンスが渡される。
    * - | (3)
      - | リソースにリンク情報を追加する。
        | 上記例では、リンク情報に設定するURIを組み立てるため \ ``UriComponentsBuilder``\ クラスのメソッドを呼び出し、自身のリソースにアクセスするためのURIをリソースに追加している。
        |
        | Controllerのメソッドの引数として渡された\ ``ServletUriComponentsBuilder``\ のインスタンスは、web.xmlに記載の\ ``<servlet-mapping>``\要素の情報を元に初期化されており、リソースには依存しない。
        | そのため、Spring Frameworkから提供される `URI Template Patterns <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-requestmapping-uri-templates>`_\ 等を利用し、
        | リクエスト情報をベースにURIを組み立てる事により、リソースに依存しない汎用的な組み立て処理を実装することが可能となる。
        | 
        | 例えば、上記例において\ ``http://example.com/api/v1/members/M000000001``\に対してGETした場合、組み立てられるURIは、リクエストされたURIと同じ値\ ``（http://example.com/api/v1/members/M000000001）``\になる。
        | 
        | 必要に応じてリンク情報に設定するURIを組み立てるためのメソッドを実装すること。

 .. tip::

    \ ``ServletUriComponentsBuilder``\では、URIを組み立てる際に「\ ``X-Forwarded-Host``\」ヘッダを参照することで、クライアントとアプリケーションサーバの間にロードバランサやWebサーバがある構成を考慮している。 
    ただし、パスの構成を合わせておかないと期待通りのURIにならないので注意が必要である。


* | レスポンス例
  | 実際に動かすと、以下のようなレスポンスとなる。

 .. code-block:: guess

    GET /rest-api-web/api/v1/members/M000000001 HTTP/1.1
    Accept: text/plain, application/json, application/*+json, */*
    User-Agent: Java/1.7.0_51
    Host: localhost:8080
    Connection: keep-alive

 .. code-block:: java
    :emphasize-lines: 2-5

    {
      "links" : [ {
        "rel" : "self",
        "href" : "http://localhost:8080/rest-api-web/api/v1/members/M000000001"
      } ],
      "memberId" : "M000000001",
      "firstName" : "John",
      "lastName" : "Smith",
      "genderCode" : "1",
      "dateOfBirth" : "2013-03-14",
      "emailAddress" : "user1394794959984@test.com",
      "telephoneNumber" : "09012345678",
      "zipCode" : "1710051",
      "address" : "Tokyo",
      "credential" : {
        "signId" : "user1394794959984@test.com",
        "passwordLastChangedAt" : "2014-03-14T11:02:41.477Z",
        "lastModifiedAt" : "2014-03-14T11:02:41.477Z"
      },
      "createdAt" : "2014-03-14T11:02:41.477Z",
      "lastModifiedAt" : "2014-03-14T11:02:41.477Z"
    }

|

.. _RESTAppendixRestApiOfHTTPCompliance:

HTTPの仕様に準拠したRESTful Web Serviceの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 本編で説明したREST APIの実装では、HTTPの仕様に準拠していない箇所がある。
| 本節では、HTTPの仕様に準拠したRESTful Web Serviceにするための実装例について説明する。

POST時のLocationヘッダの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| HTTPの仕様に準拠する場合、POST時のレスポンスヘッダー(「Locationヘッダ」)には、作成したリソースのURIを設定する必要がある。
| POST時のレスポンスヘッダ(「Locationヘッダ」)に、作成したリソースのURIを設定するための実装方法について説明する。



リソース毎の実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* REST APIの処理で、作成したリソースのURIをLocationヘッダに設定する。

 .. code-block:: java
    :emphasize-lines: 11, 21, 25

    @RequestMapping("members")
    @RestController
    public class MemberRestController {

        // omitted

        @RequestMapping(method = RequestMethod.POST)
        public ResponseEntity<MemberResource> postMembers(
                @RequestBody @Validated({ PostMembers.class, Default.class })
                MemberResource requestedResource,
                // (1)
                UriComponentsBuilder uriBuilder) {
    
            Member creatingMember = beanMapper.map(requestedResource, Member.class);
    
            Member createdMember = memberService.createMember(creatingMember);
    
            MemberResource responseResource = beanMapper.map(createdMember,
                    MemberResource.class);

            // (2)
            URI createdUri = uriBuilder.path("/members/{memberId}")
                    .buildAndExpand(responseResource.getMemberId()).toUri();
    
            // (3)
            return ResponseEntity.created(createdUri).body(responseResource);
        }

        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | 作成したリソースのURIを組み立てるため、Spring MVCから提供されている\ ``org.springframework.web.util.UriComponentsBuilder``\クラスをメソッドの引数に指定する。
        | \ ``UriComponentsBuilder``\ クラスをControllerのメソッドの引数に指定すると、メソッド実行時に、Spring MVCにより\ ``UriComponentsBuilder``\ クラスを継承した
        | \ ``org.springframework.web.servlet.support.ServletUriComponentsBuilder``\クラスのインスタンスが渡される。
    * - | (2)
      - | 作成したリソースのURIを組み立てる。
        | 上記例では、引数として渡された\ ``ServletUriComponentsBuilder``\ のインスタンスに\ ``path``\ メソッドで、URI Template Patternsを用いたパスを追加し、
        | \ ``buildAndExpand``\ メソッドを呼び出して、作成したリソースのIDをバインドすることで、作成したリソースのURIを組み立てている。
        | 
        | Controllerのメソッドの引数として渡された\ ``ServletUriComponentsBuilder``\ のインスタンスは、web.xmlに記載の\ ``<servlet-mapping>``\要素の情報を元に初期化されており、リソースには依存しない。
        | そのため、Spring Frameworkから提供される `URI\ Template\ Patterns <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-requestmapping-uri-templates>`_\ 等を利用し、
        | リクエスト情報をベースにURIを組み立てる事により、リソースに依存しない汎用的な組み立て処理を実装することが可能となる。
        | 
        | 例えば、上記例において\ ``http://example.com/api/v1/members``\に対してPOSTした場合、組み立てられるURIは、「リクエストされたURI + \ ``"/"``\ + 作成したリソースのID」となる。
        | 具体的には、IDに\ ``"M000000001"``\を指定した場合、\ ``http://example.com/api/v1/members/M000000001``\となる。
        | 
        | 必要に応じてリンク情報に設定するURIを組み立てるためのメソッドを実装すること。
    * - | (3)
      - | 以下のパラメータを使用して\ ``org.springframework.http.ResponseEntity``\ を生成し返却する。

        * ステータスコード : 201(Created)
        * Locationヘッダ : 作成したリソースのURI
        * レスポンスBODY : 作成したResourceオブジェクト

 .. raw:: latex

    \newpage

 .. tip::

    \ ``ServletUriComponentsBuilder``\では、URIを組み立てる際に「\ ``X-Forwarded-Host``\」ヘッダを参照することで、クライアントとアプリケーションサーバの間にロードバランサやWebサーバがある構成を考慮している。 
    ただし、パスの構成を合わせておかないと期待通りのURIにならないので注意が必要である。


* | レスポンス例
  | 実際に動かすと、以下のようなレスポンスヘッダとなる。

 .. code-block:: guess
    :emphasize-lines: 4

    HTTP/1.1 201 Created
    Server: Apache-Coyote/1.1
    X-Track: 693e132312d64998a7d8d6cabf3d13ef
    Location: http://localhost:8080/rest-api-web/api/v1/members/M000000001
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Fri, 14 Mar 2014 12:34:31 GMT

|

.. _RESTAppendixDispatchOptionsMethod:

OPTIONSメソッドのリクエストをControllerにディスパッチするための設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| HTTPの仕様に準拠する場合は、、リソース毎に呼び出しが許可されているHTTPメソッドの一覧を返却する必要がある。そのため、OPTIONSメソッドのリクエストをControllerへディスパッチするための設定を追加する必要となる。
| \ ``DispatcherServlet``\のデフォルトの設定では、OPTIONSメソッドのリクエストはControllerにディスパッチされずに、\ ``DispatcherServlet``\が許可しているメソッドのリストがAllowヘッダに設定されてしまう。

- :file:`web.xml`

 .. code-block:: xml
    :emphasize-lines: 10-14

    <!-- omitted -->

    <servlet>
        <servlet-name>appServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath*:META-INF/spring/spring-mvc-rest.xml</param-value>
        </init-param>
        <!-- (1) -->
        <init-param>
            <param-name>dispatchOptionsRequest</param-name>
            <param-value>true</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <!-- omitted -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | 項番
     - | 説明
   * - | (1)
     - | RESTful Web Serviceのリクエストを受け付ける\ ``DispatcherServlet``\の初期化パラメータ(dispatchOptionsRequest)の値を、\ ``true``\に設定する。

|

.. _RESTAppendixRestApiOfHTTPComplianceImplementationOfOptionsSpecifiedResource:

OPTIONSメソッドの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| HTTPの仕様に準拠する場合、リソース毎に呼び出しが許可されているHTTPメソッドの一覧を返却する必要がある。
| URIで指定されたリソースでサポートされているHTTPメソッド(REST API)のリストを応答するAPIの実装例を、以下に示す。

* | REST APIの実装
  | URIで指定されたリソースでサポートされているHTTPメソッド(REST API)のリストを応答する処理を実装する。

 .. code-block:: java
    :emphasize-lines: 11, 14

    @RequestMapping("members")
    @RestController
    public class MembersRestController {

        // omitted

        @RequestMapping(value = "{memberId}", method = RequestMethod.OPTIONS)
        public ResponseEntity<Void> optionsMember(
            @PathVariable("memberId") String memberId) {

            // (1)
            memberService.getMember(memberId);

            // (2)
            return ResponseEntity
                    .ok()
                    .allow(HttpMethod.GET, HttpMethod.HEAD, HttpMethod.PUT,
                            HttpMethod.DELETE, HttpMethod.OPTIONS).build();
        }
    
        // omitted

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ドメイン層のServiceのメソッドを呼び出し、パス変数から取得したIDに一致するリソースが存在するかチェックを行う。
    * - | (2)
      - | **URIで指定されたリソースでサポートされているHTTPメソッドを、Allowヘッダに設定する。**

|

* リクエスト例

 .. code-block:: guess
    :emphasize-lines: 1

    OPTIONS /rest-api-web/api/v1/members/M000000004 HTTP/1.1
    Accept: text/plain, application/json, application/*+json, */*
    User-Agent: Java/1.7.0_51
    Host: localhost:8080
    Connection: keep-alive


|

* レスポンス例

 .. code-block:: guess
    :emphasize-lines: 4

    HTTP/1.1 200 OK
    Server: Apache-Coyote/1.1
    X-Track: 6d7bbc818c7f44e7942c54bc0ddc15bb
    Allow: GET,HEAD,PUT,DELETE,OPTIONS
    Content-Length: 0
    Date: Mon, 17 Mar 2014 01:54:27 GMT

|

HEADメソッドの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| HTTPの仕様に準拠する場合、GETメソッドを実装する場合、HEADメソッドも実装する必要がある。
| URIで指定されたリソースのメタ情報を応答するAPIの実装例を、以下に示す。


* | REST APIの実装
  | URIで指定されたリソースのメタ情報を取得する処理を実装する。

 .. code-block:: java
    :emphasize-lines: 9

    @RequestMapping("members")
    @RestController
    public class MemberRestController {

        // omitted

        @RequestMapping(value = "{memberId}",
                        method = { RequestMethod.GET,
                                   RequestMethod.HEAD }) // (1)
        @ResponseStatus(HttpStatus.OK)
        public MemberResource getMember(
                @PathVariable("memberId") String memberId) {
            // omitted
        }

        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | GETメソッドの処理を行うREST APIの\ ``@RequestMapping``\アノテーションのmethod属性に\ ``RequestMethod.HEAD``\を追加する。
        | HEADメソッドは、GETメソッドと同じ処理を行いヘッダ情報のみレスポンスする必要があるため、\ ``@RequestMapping``\アノテーションのmethod属性に、\ ``RequestMethod.HEAD``\も指定する。
        | レスポンスBODYを空にする処理は、Servlet APIの標準機能で行われるため、Controllerの処理としてはGETメソッドと同じ処理を行えばよい。

 |
 
* リクエスト例
 
 .. code-block:: guess
    :emphasize-lines: 1
 
    HEAD /rest-api-web/api/v1/members/M000000001 HTTP/1.1
    Accept: text/plain, application/json, application/*+json, */*
    User-Agent: Java/1.7.0_51
    Host: localhost:8080
    Connection: keep-alive

 |
 
* レスポンス例

 .. code-block:: guess
    :emphasize-lines: 1, 4, 5

    HTTP/1.1 200 OK
    Server: Apache-Coyote/1.1
    X-Track: 71093a551e624c149867b6bfec486d2c
    Content-Type: application/json;charset=UTF-8
    Content-Length: 452
    Date: Thu, 13 Mar 2014 13:25:23 GMT
 

|



.. _RESTAppendixDisabledCSRFProtection:

CSRF対策の無効化
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
RESTful Web Service向けのリクエストに対して、CSRF対策を行わないようにするための設定方法について説明する。

 .. tip::

    CSRF対策を行わない場合は、セッションを利用する必要がなくなる。
    
    下記設定例では、Spring Securityの処理でセッションが使用されなくなる様にしている。

|

Blankプロジェクトのデフォルトの設定では、CSRF対策が有効化されているため、以下の設定を追加し、
RESTful Web Service向けのリクエストに対して、CSRF対策の処理が行われないようにする。

* :file:`spring-security.xml`

 .. code-block:: xml
    :emphasize-lines: 3-10

    <!-- omitted -->

    <!-- (1) -->
    <sec:http
        pattern="/api/v1/**"
        create-session="stateless">
        <sec:http-basic/>
        <sec:csrf disabled="true"/>
    </sec:http>

    <sec:http>
        <sec:access-denied-handler ref="accessDeniedHandler"/>
        <sec:custom-filter ref="userIdMDCPutFilter" after="ANONYMOUS_FILTER"/>
        <sec:form-login/>
        <sec:logout/>
        <sec:session-management />
    </sec:http>

    <!-- omitted -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | REST API用のSpring Securityの定義を追加する。
       | \ ``<sec:http>``\要素の\ ``pattern``\属性に、REST API用のリクエストパスのURLパターンを指定している。
       | 上記例では、\ ``/api/v1/``\で始まるリクエストパスをREST API用のリクエストパスとして扱う。
       | また、\ ``create-session``\属性を\ ``stateless``\とする事で、Spring Securityの処理でセッションが使用されなくなる。
       |
       | CSRF対策を無効化するために、\ ``<sec:csrf>``\ 要素に \ ``disabled="true"``\ を指定している。

|

.. _RESTAppendixEnabledXXEInjectProtection:

XXE Injection対策の有効化
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| RESTful Web ServiceでXML形式のデータを扱う場合は、\ `XXE(XML External Entity) Injection <https://www.owasp.org/index.php/XML_External_Entity_(XXE)_Processing>`_\対策を行う必要がある。
| terasoluna-gfw-web 1.0.1.RELEASE以上では、XXE Injection 対策が行われているSpring MVC(3.2.10.RELEASE以上)に依存しているため、個別に対策を行う必要はない。

 .. warning:: **XXE(XML External Entity) Injection 対策について**

    terasoluna-gfw-web 1.0.0.RELEASEを使用している場合は、XXE Injection対策が行われていないSpring MVC(3.2.4.RELEASE)に依存しているため、Spring-oxmから提供されているクラスを使用すること。

|

Spring-oxmを依存アーティファクトとして追加する。

- :file:`pom.xml`

 .. code-block:: xml

    <!-- omitted -->

    <!-- (1) -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-oxm</artifactId>
        <version>${org.springframework-version}</version> <!-- (2) -->
    </dependency>

    <!-- omitted -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90
   :class: longtable

   * - | 項番
     - | 説明
   * - | (1)
     - | Spring-oxm を依存アーティファクトとして追加する。
   * - | (2)
     - | Springのバージョンは、terasoluna-gfw-parent の :file:`pom.xml` に定義されているSpringのバージョン番号を管理するためのプレースフォルダ(${org.springframework-version})から取得すること。

|

Spring-oxmから提供されているクラスを使用してXMLとオブジェクトの相互変換を行うためのbean定義を行う。

- :file:`spring-mvc-rest.xml`

 .. code-block:: xml

    <!-- omitted -->

    <!-- (1) -->
    <bean id="xmlMarshaller" class="org.springframework.oxm.jaxb.Jaxb2Marshaller">
        <property name="packagesToScan" value="com.examples.app" /> <!-- (2) -->
    </bean>

    <!-- omitted -->

    <mvc:annotation-driven>

        <mvc:message-converters>
            <!-- (3) -->
            <bean class="org.springframework.http.converter.xml.MarshallingHttpMessageConverter">
                <property name="marshaller" ref="xmlMarshaller" /> <!-- (4) -->
                <property name="unmarshaller" ref="xmlMarshaller" /> <!-- (5) -->
            </bean>
        </mvc:message-converters>

        <!-- omitted -->

    </mvc:annotation-driven>

    <!-- omitted -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | 項番
     - | 説明
   * - | (1)
     - | Spring-oxmから提供されている\ ``Jaxb2Marshaller``\のbean定義を行う。
       | \ ``Jaxb2Marshaller``\はデフォルトの状態で XXE Injection対策が行われている。
   * - | (2)
     - | ``packagesToScan`` プロパティに JAXB用のJavaBean( ``javax.xml.bind.annotation.XmlRootElement`` アノテーションなどが付与されているJavaBean)が格納されているパッケージ名を指定する。
       | 指定したパッケージ配下に格納されているJAXB用のJavaBeanがスキャンされ、marshal、unmarshal対象のJavaBeanとして登録される。
       | ``<context:component-scan>`` の base-package属性と同じ仕組みでスキャンされる。
   * - | (3)
     - | ``<mvc:annotation-driven>`` の子要素である ``<mvc:message-converters>`` 要素に、 ``MarshallingHttpMessageConverter`` のbean定義を追加する。
   * - | (4)
     - | ``marshaller`` プロパティに (1)で定義した ``Jaxb2Marshaller`` のbeanを指定する。
   * - | (5)
     - | ``unmarshaller`` プロパティに (1)で定義した ``Jaxb2Marshaller`` のbeanを指定する。

|

.. _RESTAppendixCopyJodaObjectByBeanConvert:

Dozerを使ってJoda-Timeのクラスをコピーする方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Dozerを使用して、Joda-Timeのクラス(\ ``org.joda.time.DateTime``\、\ ``org.joda.time.LocalDate``\など)をコピーする方法について説明する。

| Joda-Timeのクラスを変換するためのカスタムコンバータを作成する。
| カスタムコンバータの詳細については、「:doc:`../GeneralFuncDetail/Dozer`」を参照されたい。

* :file:`JodaDateTimeConverter.java`

 .. code-block:: java
  
    package org.terasoluna.examples.rest.infra.dozer.converter;
    
    import org.dozer.DozerConverter;
    import org.joda.time.DateTime;
    
    public class JodaDateTimeConverter extends DozerConverter<DateTime, DateTime> {
    
        public JodaDateTimeConverter() {
            super(DateTime.class, DateTime.class);
        }
    
        @Override
        public DateTime convertTo(DateTime source, DateTime destination) {
            // This method not called, because type of from/to is same.
            return convertFrom(source, destination);
        }
    
        @Override
        public DateTime convertFrom(DateTime source, DateTime destination) {
            return source;
        }
    
    }


* :file:`JodaLocalDateConverter.java`

 .. code-block:: java

    package org.terasoluna.examples.rest.infra.dozer.converter;
    
    import org.dozer.DozerConverter;
    import org.joda.time.LocalDate;
    
    public class JodaLocalDateConverter extends
                                       DozerConverter<LocalDate, LocalDate> {
    
        public JodaLocalDateConverter() {
            super(LocalDate.class, LocalDate.class);
        }
    
        @Override
        public LocalDate convertTo(LocalDate source, LocalDate destination) {
            // This method not called, because type of from/to is same.
            return convertFrom(source, destination);
        }
    
        @Override
        public LocalDate convertFrom(LocalDate source, LocalDate destination) {
            return source;
        }
    
    }

| 作成したカスタムコンバータをDozerに適用する。
| カスタムコンバータの詳細については、「:doc:`../GeneralFuncDetail/Dozer`」を参照されたい。

 .. code-block:: xml
    :emphasize-lines: 1, 10-18
  
    <!-- (1) -->
    <?xml version="1.0" encoding="UTF-8"?>
    <mappings xmlns="http://dozer.sourceforge.net" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="
            http://dozer.sourceforge.net http://dozer.sourceforge.net/schema/beanmapping.xsd
        ">
    
        <configuration>
            <custom-converters>
                <!-- (2) -->
                <converter type="org.terasoluna.examples.rest.infra.dozer.converter.JodaDateTimeConverter">
                    <class-a>org.joda.time.DateTime</class-a>
                    <class-b>org.joda.time.DateTime</class-b>
                </converter>
                <converter type="org.terasoluna.examples.rest.infra.dozer.converter.JodaLocalDateConverter">
                    <class-a>org.joda.time.LocalDate</class-a>
                    <class-b>org.joda.time.LocalDate</class-b>
                </converter>
            </custom-converters>
        </configuration>
    
    </mappings>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | Dozerの動作設定を定義するファイルを作成する。
        | 
        | 今回の実装例では、\ :file:`/xxx-domain/src/main/resources/META-INF/dozer/dozer-configration-mapping.xml`\に格納する。
    * - | (2)
      - | 上記例では、Joda-Timeのクラス(\ ``org.joda.time.DateTime``\と\ ``org.joda.time.LocalDate``\)に対するカスタムコンバータの定義を追加している。


 .. note::
 
    ドメイン層でもDozerを使用する場合は、Dozerの動作設定を定義するファイルは、ドメイン層用のプロジェクト(\ ``xxx-domain``\)に格納する事を推奨する。
    
    アプリケーション層のみでDozerを使う場合は、アプリケーション層用のプロジェクト(\ ``xxx-web``\)に格納してもよい。

|


.. _RESTAppendixSoruceCodesOfApplicationLayer:

アプリケーション層のソースコード
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| \ :ref:`RESTHowToUse`\の説明で使用したアプリケーション層のソースコードのうち、断片的に貼りつけていたソースコードの完全版を添付しておく。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.45\linewidth}|p{0.45\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 45 45

   * - | 項番
     - | セクション
     - | ファイル名
   * - | (1)
     - | :ref:`RESTHowToUseApiImplementation`
     - | :ref:`MemberRestController.java <RESTAppendixSoruceCodesOfMemberRestController>`
   * - | (2)
     - | :ref:`RESTHowToUseExceptionHandling`
     - | :ref:`ApiErrorCreator.java <RESTAppendixSoruceCodesOfApiErrorCreator>`
   * - | (3)
     - | 
     - | :ref:`ApiGlobalExceptionHandler.java <RESTAppendixSoruceCodesOfApiGlobalExceptionHandler>`

 以下のファイルは、除外している。

 * JavaBean
 * 設定ファイル


|

.. _RESTAppendixSoruceCodesOfMemberRestController:

MemberRestController.java
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

:file:`java/org/terasoluna/examples/rest/api/member/MemberRestController.java`

.. code-block:: java

    package org.terasoluna.examples.rest.api.member;
    
    import java.util.ArrayList;
    import java.util.List;
    
    import javax.inject.Inject;
    import javax.validation.groups.Default;
    
    import org.dozer.Mapper;
    import org.springframework.data.domain.Page;
    import org.springframework.data.domain.PageImpl;
    import org.springframework.data.domain.Pageable;
    import org.springframework.http.HttpStatus;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.PathVariable;
    import org.springframework.web.bind.annotation.RequestBody;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.bind.annotation.ResponseStatus;
    import org.springframework.web.bind.annotation.RestController;
    import org.terasoluna.examples.rest.api.member.MemberResource.PostMembers;
    import org.terasoluna.examples.rest.api.member.MemberResource.PutMember;
    import org.terasoluna.examples.rest.domain.model.Member;
    import org.terasoluna.examples.rest.domain.service.member.MemberService;
    
    @RequestMapping("members")
    @RestController
    public class MemberRestController {
    
        @Inject
        MemberService memberService;
    
        @Inject
        Mapper beanMapper;
    
        @RequestMapping(method = RequestMethod.GET)
        @ResponseStatus(HttpStatus.OK)
        public Page<MemberResource> getMembers(@Validated MembersSearchQuery query,
                Pageable pageable) {
    
            Page<Member> page = memberService.searchMembers(query.getName(), pageable);
    
            List<MemberResource> memberResources = new ArrayList<>();
            for (Member member : page.getContent()) {
                memberResources.add(beanMapper.map(member, MemberResource.class));
            }
            Page<MemberResource> responseResource =
                new PageImpl<>(memberResources, pageable, page.getTotalElements());
    
            return responseResource;
        }
    
        @RequestMapping(method = RequestMethod.GET)
        @ResponseStatus(HttpStatus.OK)
        public List<MemberResource> getMembers() {
            List<Member> members = memberService.findAll();
            
            List<MemberResource> memberResources = new ArrayList<>();
            for (Member member : members) {
                memberResources.add(beanMapper.map(member, MemberResource.class));
            }
            return memberResources;
        }
    
        @RequestMapping(method = RequestMethod.POST)
        @ResponseStatus(HttpStatus.CREATED)
        public MemberResource postMembers(@RequestBody @Validated({
                PostMembers.class, Default.class }) MemberResource requestedResource) {
    
            Member creatingMember = beanMapper.map(requestedResource, Member.class);
    
            Member createdMember = memberService.createMember(creatingMember);
    
            MemberResource responseResource = beanMapper.map(createdMember,
                    MemberResource.class);
    
            return responseResource;
        }
    
        @RequestMapping(value = "{memberId}", method = RequestMethod.GET)
        @ResponseStatus(HttpStatus.OK)
        public MemberResource getMember(@PathVariable("memberId") String memberId) {
    
            Member member = memberService.getMember(memberId);
    
            MemberResource responseResource = beanMapper.map(member,
                    MemberResource.class);
    
            return responseResource;
        }
    
        @RequestMapping(value = "{memberId}", method = RequestMethod.PUT)
        @ResponseStatus(HttpStatus.OK)
        public MemberResource putMember(
                @PathVariable("memberId") String memberId,
                @RequestBody @Validated({
                PutMember.class, Default.class }) MemberResource requestedResource) {
    
            Member updatingMember = beanMapper.map(requestedResource, Member.class);
    
            Member updatedMember = memberService.updateMember(memberId,
                    updatingMember);
    
            MemberResource responseResource = beanMapper.map(updatedMember,
                    MemberResource.class);
    
            return responseResource;
        }
    
        @RequestMapping(value = "{memberId}", method = RequestMethod.DELETE)
        @ResponseStatus(HttpStatus.NO_CONTENT)
        public void deleteMember(@PathVariable("memberId") String memberId) {
    
            memberService.deleteMember(memberId);
    
        }
    
    }

|

.. _RESTAppendixSoruceCodesOfApiErrorCreator:

ApiErrorCreator.java
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

:file:`java/org/terasoluna/examples/rest/api/common/error/ApiErrorCreator.java`

.. code-block:: java

    package org.terasoluna.examples.rest.api.common.error;
    
    import javax.inject.Inject;
    
    import org.springframework.context.MessageSource;
    import org.springframework.context.support.DefaultMessageSourceResolvable;
    import org.springframework.stereotype.Component;
    import org.springframework.validation.BindingResult;
    import org.springframework.validation.FieldError;
    import org.springframework.validation.ObjectError;
    import org.springframework.web.context.request.WebRequest;
    import org.terasoluna.gfw.common.message.ResultMessage;
    import org.terasoluna.gfw.common.message.ResultMessages;
    
    @Component
    public class ApiErrorCreator {
    
        @Inject
        MessageSource messageSource;
    
        public ApiError createApiError(WebRequest request, String errorCode,
                String defaultErrorMessage, Object... arguments) {
            String localizedMessage = messageSource.getMessage(errorCode,
                    arguments, defaultErrorMessage, request.getLocale());
            return new ApiError(errorCode, localizedMessage);
        }
    
        public ApiError createBindingResultApiError(WebRequest request,
                String errorCode, BindingResult bindingResult,
                String defaultErrorMessage) {
            ApiError apiError = createApiError(request, errorCode,
                    defaultErrorMessage);
            for (FieldError fieldError : bindingResult.getFieldErrors()) {
                apiError.addDetail(createApiError(request, fieldError, fieldError
                        .getField()));
            }
            for (ObjectError objectError : bindingResult.getGlobalErrors()) {
                apiError.addDetail(createApiError(request, objectError, objectError
                        .getObjectName()));
            }
            return apiError;
        }
    
        private ApiError createApiError(WebRequest request,
                DefaultMessageSourceResolvable messageResolvable, String target) {
            String localizedMessage = messageSource.getMessage(messageResolvable,
                    request.getLocale());
            return new ApiError(messageResolvable.getCode(), localizedMessage, target);
        }
    
        public ApiError createResultMessagesApiError(WebRequest request,
                String rootErrorCode, ResultMessages resultMessages,
                String defaultErrorMessage) {
            ApiError apiError;
            if (resultMessages.getList().size() == 1) {
                ResultMessage resultMessage = resultMessages.iterator().next();
                String errorCode = resultMessage.getCode();
                String errorText = resultMessage.getText();
                if (errorCode == null && errorText == null) {
                    errorCode = rootErrorCode;
                }
                apiError = createApiError(request, errorCode, errorText,
                        resultMessage.getArgs());
            } else {
                apiError = createApiError(request, rootErrorCode,
                        defaultErrorMessage);
                for (ResultMessage resultMessage : resultMessages.getList()) {
                    apiError.addDetail(createApiError(request, resultMessage
                            .getCode(), resultMessage.getText(), resultMessage
                            .getArgs()));
                }
            }
            return apiError;
        }
    
    }

|

.. _RESTAppendixSoruceCodesOfApiGlobalExceptionHandler:

ApiGlobalExceptionHandler.java
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

:file:`java/org/terasoluna/examples/rest/api/common/error/ApiGlobalExceptionHandler.java`

.. code-block:: java

    package org.terasoluna.examples.rest.api.common.error;
    
    import javax.inject.Inject;
    
    import org.springframework.dao.OptimisticLockingFailureException;
    import org.springframework.dao.PessimisticLockingFailureException;
    import org.springframework.http.HttpHeaders;
    import org.springframework.http.HttpStatus;
    import org.springframework.http.ResponseEntity;
    import org.springframework.http.converter.HttpMessageNotReadableException;
    import org.springframework.validation.BindException;
    import org.springframework.validation.BindingResult;
    import org.springframework.web.bind.MethodArgumentNotValidException;
    import org.springframework.web.bind.annotation.ControllerAdvice;
    import org.springframework.web.bind.annotation.ExceptionHandler;
    import org.springframework.web.context.request.WebRequest;
    import org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler;
    import org.terasoluna.gfw.common.exception.BusinessException;
    import org.terasoluna.gfw.common.exception.ExceptionCodeResolver;
    import org.terasoluna.gfw.common.exception.ResourceNotFoundException;
    import org.terasoluna.gfw.common.exception.ResultMessagesNotificationException;
    
    @ControllerAdvice
    public class ApiGlobalExceptionHandler extends ResponseEntityExceptionHandler {
    
        @Inject
        ApiErrorCreator apiErrorCreator;
    
        @Inject
        ExceptionCodeResolver exceptionCodeResolver;
    
        @Override
        protected ResponseEntity<Object> handleExceptionInternal(Exception ex,
                Object body, HttpHeaders headers, HttpStatus status,
                WebRequest request) {
            final Object apiError;
            if (body == null) {
                String errorCode = exceptionCodeResolver.resolveExceptionCode(ex);
                apiError = apiErrorCreator.createApiError(request, errorCode, ex
                        .getLocalizedMessage());
            } else {
                apiError = body;
            }
            return ResponseEntity.status(status).headers(headers).body(apiError);
        }
    
        @Override
        protected ResponseEntity<Object> handleMethodArgumentNotValid(
                MethodArgumentNotValidException ex, HttpHeaders headers,
                HttpStatus status, WebRequest request) {
            return handleBindingResult(ex, ex.getBindingResult(), headers, status,
                    request);
        }
    
        @Override
        protected ResponseEntity<Object> handleBindException(BindException ex,
                HttpHeaders headers, HttpStatus status, WebRequest request) {
            return handleBindingResult(ex, ex.getBindingResult(), headers, status,
                    request);
        }
    
        private ResponseEntity<Object> handleBindingResult(Exception ex,
                BindingResult bindingResult, HttpHeaders headers,
                HttpStatus status, WebRequest request) {
            String errorCode = exceptionCodeResolver.resolveExceptionCode(ex);
            ApiError apiError = apiErrorCreator.createBindingResultApiError(
                    request, errorCode, bindingResult, ex.getMessage());
            return handleExceptionInternal(ex, apiError, headers, status, request);
        }
    
        @Override
        protected ResponseEntity<Object> handleHttpMessageNotReadable(
                HttpMessageNotReadableException ex, HttpHeaders headers,
                HttpStatus status, WebRequest request) {
            if (ex.getCause() instanceof Exception) {
                return handleExceptionInternal((Exception) ex.getCause(), null,
                        headers, status, request);
            } else {
                return handleExceptionInternal(ex, null, headers, status, request);
            }
        }
    
        @ExceptionHandler(ResourceNotFoundException.class)
        public ResponseEntity<Object> handleResourceNotFoundException(
                ResourceNotFoundException ex, WebRequest request) {
            return handleResultMessagesNotificationException(ex, new HttpHeaders(),
                    HttpStatus.NOT_FOUND, request);
        }
    
        @ExceptionHandler(BusinessException.class)
        public ResponseEntity<Object> handleBusinessException(BusinessException ex,
                WebRequest request) {
            return handleResultMessagesNotificationException(ex, new HttpHeaders(),
                    HttpStatus.CONFLICT, request);
        }
    
        private ResponseEntity<Object> handleResultMessagesNotificationException(
                ResultMessagesNotificationException ex, HttpHeaders headers,
                HttpStatus status, WebRequest request) {
            String errorCode = exceptionCodeResolver.resolveExceptionCode(ex);
            ApiError apiError = apiErrorCreator.createResultMessagesApiError(
                    request, errorCode, ex.getResultMessages(), ex.getMessage());
            return handleExceptionInternal(ex, apiError, headers, status, request);
        }
    
        @ExceptionHandler({ OptimisticLockingFailureException.class,
                PessimisticLockingFailureException.class })
        public ResponseEntity<Object> handleLockingFailureException(Exception ex,
                WebRequest request) {
            return handleExceptionInternal(ex, null, new HttpHeaders(),
                    HttpStatus.CONFLICT, request);
        }
    
        @ExceptionHandler(Exception.class)
        public ResponseEntity<Object> handleSystemError(Exception ex,
                WebRequest request) {
            return handleExceptionInternal(ex, null, new HttpHeaders(),
                    HttpStatus.INTERNAL_SERVER_ERROR, request);
        }
    
    }

|

.. _RESTAppendixSoruceCodesOfDomainLayer:

REST API実装時に作成したドメイン層のクラスのソースコード
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| \ :ref:`RESTHowToUse`\で説明したREST APIから呼び出しているドメイン層のクラスのソースコードを添付しておく。
| なお、インフラストラクチャ層は、MyBatis3を使って実装している。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 35 55

   * - | 項番
     - | 分類
     - | ファイル名
   * - | (1)
     - | model
     - | :ref:`Member.java <RESTAppendixSoruceCodesOfMember>`
   * - | (2)
     - | 
     - | :ref:`MemberCredentia.java <RESTAppendixSoruceCodesOfMemberCredentia>`
   * - | (3)
     - | 
     - | :ref:`Gender.java <RESTAppendixSoruceCodesOfGender>`
   * - | (4)
     - | repository
     - | :ref:`MemberRepository.java <RESTAppendixSoruceCodesOfMemberRepository>`
   * - | (5)
     - | service
     - | :ref:`MemberService.java <RESTAppendixSoruceCodesOfMemberService>`
   * - | (6)
     - | 
     - | :ref:`MemberServiceImpl.java <RESTAppendixSoruceCodesOfMemberServiceImpl>`
   * - | (7)
     - | other
     - | :ref:`DomainMessageCodes.java <RESTAppendixSoruceCodesOfDomainMessageCodes>`
   * - | (8)
     - | 
     - | :ref:`GenderTypeHandler.java <RESTAppendixSoruceCodesOfGenderTypeHandler>`
   * - | (9)
     - | 
     - | :ref:`member-mapping.xml <RESTAppendixSoruceCodesOfMemberMappingXml>`
   * - | (10)
     - | 
     - | :ref:`mybatis-config.xml <RESTAppendixSoruceCodesOfMybatisConfig>`
   * - | (11)
     - | 
     - | :ref:`MemberRepository.xml <RESTAppendixSoruceCodesOfMemberRepositoryxml>`




 以下のファイルは、除外している。

 * Entity以外のJavaBean
 * Dozer以外の設定ファイル

|

.. _RESTAppendixSoruceCodesOfMember:

Member.java
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

:file:`java/org/terasoluna/examples/rest/domain/model/Member.java`

.. code-block:: java

    package org.terasoluna.examples.rest.domain.model;
    
    import java.io.Serializable;    
    import org.joda.time.DateTime;
    import org.joda.time.LocalDate;
    
    public class Member implements Serializable {
    
        private static final long serialVersionUID = 1L;
    
        private String memberId;
    
        private String firstName;
    
        private String lastName;
    
        private Gender gender;
    
        private LocalDate dateOfBirth;
    
        private String emailAddress;
    
        private String telephoneNumber;
    
        private String zipCode;
    
        private String address;
    
        private DateTime createdAt;
    
        private DateTime lastModifiedAt;
    
        private long version;
    
        private MemberCredential credential;
    
        public String getMemberId() {
            return memberId;
        }
    
        public void setMemberId(String memberId) {
            this.memberId = memberId;
        }
    
        public String getFirstName() {
            return firstName;
        }
    
        public void setFirstName(String firstName) {
            this.firstName = firstName;
        }
    
        public String getLastName() {
            return lastName;
        }
    
        public void setLastName(String lastName) {
            this.lastName = lastName;
        }
    
        public Gender getGender() {
            return gender;
        }
    
        public void setGender(Gender gender) {
            this.gender = gender;
        }
    
        public String getGenderCode() {
            if (gender == null) {
                return null;
            } else {
                return gender.getCode();
            }
        }
    
        public void setGenderCode(String genderCode) {
            this.gender = Gender.getByCode(genderCode);
        }
    
        public LocalDate getDateOfBirth() {
            return dateOfBirth;
        }
    
        public void setDateOfBirth(LocalDate dateOfBirth) {
            this.dateOfBirth = dateOfBirth;
        }
    
        public String getEmailAddress() {
            return emailAddress;
        }
    
        public void setEmailAddress(String emailAddress) {
            this.emailAddress = emailAddress;
        }
    
        public String getTelephoneNumber() {
            return telephoneNumber;
        }
    
        public void setTelephoneNumber(String telephoneNumber) {
            this.telephoneNumber = telephoneNumber;
        }
    
        public String getZipCode() {
            return zipCode;
        }
    
        public void setZipCode(String zipCode) {
            this.zipCode = zipCode;
        }
    
        public String getAddress() {
            return address;
        }
    
        public void setAddress(String address) {
            this.address = address;
        }
    
        public DateTime getCreatedAt() {
            return createdAt;
        }
    
        public void setCreatedAt(DateTime createdAt) {
            this.createdAt = createdAt;
        }
    
        public DateTime getLastModifiedAt() {
            return lastModifiedAt;
        }
    
        public void setLastModifiedAt(DateTime lastModifiedAt) {
            this.lastModifiedAt = lastModifiedAt;
        }
    
        public long getVersion() {
            return version;
        }
    
        public void setVersion(long version) {
            this.version = version;
        }
    
        public MemberCredential getCredential() {
            return credential;
        }
    
        public void setCredential(MemberCredential credential) {
            this.credential = credential;
        }
    
    }


|

.. _RESTAppendixSoruceCodesOfMemberCredentia:

MemberCredentia.java
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

:file:`java/org/terasoluna/examples/rest/domain/model/MemberCredential.java`

.. code-block:: java

    package org.terasoluna.examples.rest.domain.model;
    
    import java.io.Serializable;    
    import org.joda.time.DateTime;
    
    public class MemberCredential implements Serializable {
    
        private static final long serialVersionUID = 1L;
    
        private String memberId;
    
        private String signId;
    
        private String password;
    
        private String previousPassword;
    
        private DateTime passwordLastChangedAt;
    
        private DateTime lastModifiedAt;
    
        private long version;
    
        public String getMemberId() {
            return memberId;
        }
    
        public void setMemberId(String memberId) {
            this.memberId = memberId;
        }
    
        public String getSignId() {
            return signId;
        }
    
        public void setSignId(String signId) {
            this.signId = signId;
        }
    
        public String getPassword() {
            return password;
        }
    
        public void setPassword(String password) {
            this.password = password;
        }
    
        public String getPreviousPassword() {
            return previousPassword;
        }
    
        public void setPreviousPassword(String previousPassword) {
            this.previousPassword = previousPassword;
        }
    
        public DateTime getPasswordLastChangedAt() {
            return passwordLastChangedAt;
        }
    
        public void setPasswordLastChangedAt(DateTime passwordLastChangedAt) {
            this.passwordLastChangedAt = passwordLastChangedAt;
        }
    
        public DateTime getLastModifiedAt() {
            return lastModifiedAt;
        }
    
        public void setLastModifiedAt(DateTime lastModifiedAt) {
            this.lastModifiedAt = lastModifiedAt;
        }
    
        public long getVersion() {
            return version;
        }
    
        public void setVersion(long version) {
            this.version = version;
        }
    
    }


|

.. _RESTAppendixSoruceCodesOfGender:

Gender.java
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

:file:`java/org/terasoluna/examples/rest/domain/model/Gender.java`

.. code-block:: java

    package org.terasoluna.examples.rest.domain.model;
    
    import java.util.Collections;
    import java.util.HashMap;
    import java.util.Map;
    
    import org.springframework.util.Assert;
    
    public enum Gender {
    
        UNKNOWN("0"), MEN("1"), WOMEN("2");
    
        private static final Map<String, Gender> genderMap;
    
        static {
            Map<String, Gender> map = new HashMap<>();
            for (Gender gender : values()) {
                map.put(gender.code, gender);
            }
            genderMap = Collections.unmodifiableMap(map);
        }
    
        private final String code;
    
        private Gender(String code) {
            this.code = code;
        }
    
        public static Gender getByCode(String code) {
            Gender gender = genderMap.get(code);
            Assert.notNull(gender, "gender code is invalid. code : " + code);
            return gender;
        }
    
        public String getCode() {
            return code;
        }
    
    }
    

|

.. _RESTAppendixSoruceCodesOfMemberRepository:

MemberRepository.java
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

:file:`java/org/terasoluna/examples/rest/domain/repository/member/MemberRepository.java`

.. code-block:: java

    package org.terasoluna.examples.rest.domain.repository.member;
    
    import java.util.List;    
    import org.apache.ibatis.session.RowBounds;
    
    import org.terasoluna.examples.rest.domain.model.Member;
    
    public interface MemberRepository {
    
        Member findOne(String memberId);
        
        List<Member> findAll();

        long countByContainsName(String name);
        List<Member> findPageByContainsName(String name, RowBounds rowBounds);

        void createMember(Member creatingMember);
        void createCredential(Member creatingMember);

        boolean updateMember(Member updatingMember);

        void deleteMember(String memberId); 
        void deleteCredential(String memberId);
    
    }


|

.. _RESTAppendixSoruceCodesOfMemberService:

MemberService.java
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

:file:`java/org/terasoluna/examples/rest/domain/service/member/MemberService.java`

.. code-block:: java

    package org.terasoluna.examples.rest.domain.service.member;
    
    import java.util.List;
    import org.springframework.data.domain.Page;
    import org.springframework.data.domain.Pageable;
    import org.terasoluna.examples.rest.domain.model.Member;
    
    public interface MemberService {
    
        List<Member> findAll();
        
        Page<Member> searchMembers(String name, Pageable pageable);
    
        Member getMember(String memberId);
    
        Member createMember(Member creatingMember);
    
        Member updateMember(String memberId, Member updatingMember);
    
        void deleteMember(String memberId);
    
    }

|

.. _RESTAppendixSoruceCodesOfMemberServiceImpl:

MemberServiceImpl.java
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

:file:`java/org/terasoluna/examples/rest/domain/service/member/MemberServiceImpl.java`

.. code-block:: java

    package org.terasoluna.examples.rest.domain.service.member;
    
    import java.util.ArrayList;
    import java.util.List;
    import javax.inject.Inject;
    import org.apache.ibatis.session.RowBounds;
    import org.dozer.Mapper;
    import org.joda.time.DateTime;
    import org.springframework.dao.DuplicateKeyException;
    import org.springframework.data.domain.Page;
    import org.springframework.data.domain.PageImpl;
    import org.springframework.data.domain.Pageable;
    import org.springframework.orm.ObjectOptimisticLockingFailureException;
    import org.springframework.security.crypto.password.PasswordEncoder;
    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;
    import org.springframework.util.StringUtils;
    import org.terasoluna.examples.rest.domain.message.DomainMessageCodes;
    import org.terasoluna.examples.rest.domain.model.Member;
    import org.terasoluna.examples.rest.domain.model.MemberCredential;
    import org.terasoluna.examples.rest.domain.repository.member.MemberRepository;
    import org.terasoluna.gfw.common.date.jodatime.JodaTimeDateFactory;
    import org.terasoluna.gfw.common.exception.BusinessException;
    import org.terasoluna.gfw.common.exception.ResourceNotFoundException;
    import org.terasoluna.gfw.common.message.ResultMessages;

    @Transactional
    @Service
    public class MemberServiceImpl implements MemberService {
    
        @Inject
        MemberRepository memberRepository;
    
        @Inject
        JodaTimeDateFactory dateFactory;
    
        @Inject
        PasswordEncoder passwordEncoder;
    
        @Inject
        Mapper beanMapper;
        
        @Override
        @Transactional(readOnly = true)
        public List<RestMember> findAll() {
            return restMemberRepository.findAll();
        }
    
        @Override
        @Transactional(readOnly = true)
        public Page<Member> searchMembers(String name, Pageable pageable) {
            List<Member> members = null;
            // Count Members by search criteria
            long total = memberRepository.countByContainsName(name);
            if (0 < total) {
                 RowBounds rowBounds = new RowBounds(pageable.getOffset(), pageable.getPageSize());
                 members = memberRepository.findPageByContainsName(name, rowBounds);
            } else {
                members = new ArrayList<Member>();
            }
            return new PageImpl<Member>(members, pageable, total);
        }

        @Override
        @Transactional(readOnly = true)
        public Member getMember(String memberId) {
            // find member
            Member member = memberRepository.findOne(memberId);
            if (member == null) {
                // If member is not exists
                throw new ResourceNotFoundException(ResultMessages.error().add(
                                DomainMessageCodes.E_EX_MM_5001, memberId));
            }
            return member;
        }

        @Override
        public Member createMember(Member creatingMember) {
            MemberCredential creatingCredential = creatingMember
                                .getCredential();

            // get processing current date time
            DateTime currentDateTime = dateFactory.newDateTime();

            creatingMember.setCreatedAt(currentDateTime);
            creatingMember.setLastModifiedAt(currentDateTime);

            // decide sign id(email-address)
            String signId = creatingCredential.getSignId();
            if (!StringUtils.hasLength(signId)) {
                signId = creatingMember.getEmailAddress();
                creatingCredential.setSignId(signId.toLowerCase());
            }

            // encrypt password
            String rawPassword = creatingCredential.getPassword();
            creatingCredential.setPassword(passwordEncoder.encode(rawPassword));
            creatingCredential.setPasswordLastChangedAt(currentDateTime);
            creatingCredential.setLastModifiedAt(currentDateTime);

            // save member & member credential
            try {

                // Registering member details
                memberRepository.createMember(creatingMember);
                // //Registering credential details
                memberRepository.createCredential(creatingMember);
                return creatingMember;
            } catch (DuplicateKeyException e) {
                // If sign id is already used
                throw new BusinessException(ResultMessages.error().add(
                                DomainMessageCodes.E_EX_MM_8001,
                                creatingCredential.getSignId()), e);
            }
        }

        @Override
        public Member updateMember(String memberId, Member updatingMember) {
            // get member
            Member member = getMember(memberId);

            // override updating member attributes
            beanMapper.map(updatingMember, member, "member.update");

            // get processing current date time
            DateTime currentDateTime = dateFactory.newDateTime();
            member.setLastModifiedAt(currentDateTime);

            // save updating member
            boolean updated = memberRepository.updateMember(member);
            if (!updated) {
                    throw new ObjectOptimisticLockingFailureException(Member.class,
                                    member.getMemberId());
            }
            return member;
        }

        @Override
        public void deleteMember(String memberId) {

            // First Delete from credential (Child)
            memberRepository.deleteCredential(memberId);
            // Delete member
            memberRepository.deleteMember(memberId);
        }
    
    }

|

.. _RESTAppendixSoruceCodesOfDomainMessageCodes:

DomainMessageCodes.java
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

:file:`java/org/terasoluna/examples/rest/domain/message/DomainMessageCodes.java`

.. code-block:: java

    package org.terasoluna.examples.rest.domain.message;
    
    /**
     * Message codes of domain layer message.
     * @author DomainMessageCodesGenerator
     */
    public class DomainMessageCodes {
    
        private DomainMessageCodes() {
            // NOP
        }
    
        /** e.ex.mm.5001=Specified member not found. member id : {0} */
        public static final String E_EX_MM_5001 = "e.ex.mm.5001";
    
        /** e.ex.mm.8001=Cannot use specified sign id. sign id : {0} */
        public static final String E_EX_MM_8001 = "e.ex.mm.8001";
    }

|

.. _RESTAppendixSoruceCodesOfGenderTypeHandler:

GenderTypeHandler.java
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Enum型のコード値をマッピングするためのタイプハンドラーとなります。

:file:`java/org/terasoluna/examples/infra/mybatis/typehandler/GenderTypeHandler.java`

.. code-block:: java

    package org.terasoluna.examples.infra.mybatis.typehandler;

    import java.sql.CallableStatement;
    import java.sql.PreparedStatement;
    import java.sql.ResultSet;
    import java.sql.SQLException;
    import org.terasoluna.examples.domain.model.Gender;
    import org.apache.ibatis.type.JdbcType;
    import org.apache.ibatis.type.BaseTypeHandler;

    public class GenderTypeHandler extends BaseTypeHandler<Gender> {

        @Override
        public Gender getNullableResult(ResultSet rs, String columnName) throws SQLException {
                return getByCode(rs.getString(columnName));
        }

        @Override
        public Gender getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
                return getByCode(rs.getString(columnIndex));
        }

        @Override
        public Gender getNullableResult(CallableStatement cs, int columnIndex)
                        throws SQLException {
                return getByCode(cs.getString(columnIndex));
        }

        @Override
        public void setNonNullParameter(PreparedStatement ps, int i,
                        Gender parameter, JdbcType jdbcType) throws SQLException {
                ps.setString(i, parameter.getCode());
        }
        
        private Gender getByCode(String byCode) {
                if (byCode == null) {
                    return null;
                } else {
                    return Gender.getByCode(byCode);
                }
        }
    }

|

.. _RESTAppendixSoruceCodesOfMemberMappingXml:

member-mapping.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 実装したServiceクラスでは、クライアントから指定された値を\ ``Member``\オブジェクトにコピーする際に、「\ :doc:`../GeneralFuncDetail/Dozer`\」を使って行っている。
| 単純なフィールド値のコピーのみでよい場合は、Beanのマッピング定義の追加は不要だが、実装例では、更新対象外の項目(\ ``memberId``\、\ ``credential``\、\ ``createdAt``\、\ ``version``\)をコピー対象外にする必要がある。
| 特定のフィールドをコピー対象外にするためには、Beanのマッピング定義の追加が必要となる。

:file:`resources/META-INF/dozer/member-mapping.xml`

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <mappings xmlns="http://dozer.sourceforge.net" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://dozer.sourceforge.net
              http://dozer.sourceforge.net/schema/beanmapping.xsd">
    
        <mapping map-id="member.update">
            <class-a>org.terasoluna.examples.rest.domain.model.Member</class-a>
            <class-b>org.terasoluna.examples.rest.domain.model.Member</class-b>
            <field-exclude>
                <a>memberId</a>
                <b>memberId</b>
            </field-exclude>
            <field-exclude>
                <a>credential</a>
                <b>credential</b>
            </field-exclude>
            <field-exclude>
                <a>createdAt</a>
                <b>createdAt</b>
            </field-exclude>
            <field-exclude>
                <a>lastModifiedAt</a>
                <b>lastModifiedAt</b>
            </field-exclude>
            <field-exclude>
                <a>version</a>
                <b>version</b>
            </field-exclude>
        </mapping>
    
    </mappings>

|

.. _RESTAppendixSoruceCodesOfMybatisConfig:

mybatis-config.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| MyBatis3の動作をカスタマイズする場合は、MyBatis設定ファイルに設定値を追加する。MyBatis3では、Joda-Timeのクラス(org.joda.time.DateTime、org.joda.time.LocalDateTime、org.joda.time.LocalDateなど)はサポートされていない。
| そのため、EntityクラスのフィールドにJoda-Timeのクラスを使用する場合は、Joda-Time用の\ ``TypeHandler`` \を用意する必要がある。
| org.joda.time.DateTimeとjava.sql.Timestampをマッピングするための\ ``TypeHandler`` \の実装例、「\ :ref:`DataAccessMyBatis3HowToExtendTypeHandlerJoda`\」を使って行っている。

:file:`resources/META-INF/mybatis/mybatis-config.xml`

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration PUBLIC "-//mybatis.org/DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>

        <settings>
            <setting name="jdbcTypeForNull" value="NULL" />
            <setting name="mapUnderscoreToCamelCase" value="true" />
        </settings>

        <typeAliases>
            <package name="org.terasoluna.examples.infra.mybatis.typehandler" />
        </typeAliases>

        <typeHandlers>
           <package name="org.terasoluna.examples.infra.mybatis.typehandler" />
        </typeHandlers>
    
    </configuration>

|

.. _RESTAppendixSoruceCodesOfMemberRepositoryxml:

MemberRepository.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

:file:`resources/org/terasoluna/examples/rest/domain/repository/member/MemberRepository.xml`

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper
        namespace="org.terasoluna.examples.rest.domain.repository.member.MemberRepository">

        <resultMap id="MemberResultMap" type="Member">
            <id property="memberId" column="member_id" />
            <result property="firstName" column="first_name" />
            <result property="lastName" column="last_name" />
            <result property="gender" column="gender" />
            <result property="dateOfBirth" column="date_of_birth" />
            <result property="emailAddress" column="email_address" />
            <result property="telephoneNumber" column="telephone_number" />
            <result property="zipCode" column="zip_code" />
            <result property="address" column="address" />
            <result property="createdAt" column="created_at" />
            <result property="lastModifiedAt" column="last_modified_at" />
            <result property="version" column="version" />
            <result property="credential.memberId" column="member_id" />
            <result property="credential.signId" column="sign_id" />
            <result property="credential.password" column="password" />
            <result property="credential.previousPassword" column="previous_password" />
            <result property="credential.passwordLastChangedAt" column="password_last_changed_at" />
            <result property="credential.lastModifiedAt" column="credential_last_modified_at" />
            <result property="credential.version" column="credential_version" />
        </resultMap>

        <sql id="selectMember">
            SELECT
             member.member_id as member_id
             ,member.first_name as first_name
             ,member.last_name as last_name
             ,member.gender as gender
             ,member.date_of_birth as date_of_birth
             ,member.email_address as email_address
             ,member.telephone_number as telephone_number
             ,member.zip_code as zip_code
             ,member.address as address
             ,member.created_at as created_at
             ,member.last_modified_at as last_modified_at
             ,member.version as version
             ,credential.sign_id as sign_id
             ,credential.password as password
             ,credential.previous_password as previous_password
             ,credential.password_last_changed_at as password_last_changed_at
             ,credential.last_modified_at as credential_last_modified_at
             ,credential.version as credential_version
            FROM
             t_member member
             INNER JOIN t_member_credential credential ON credential.member_id = member.member_id
        </sql>

        <sql id="whereMember">
            WHERE
                member.first_name LIKE #{nameContainingCondition} ESCAPE '~'
                OR member.last_name LIKE #{nameContainingCondition} ESCAPE '~'
        </sql>

        <select id="findAll" resultMap="RestMemberResultMap">
            <include refid="selectRestMember" />
            ORDER BY member_id ASC
        </select>
	
	    <select id="findOne" parameterType="string" resultMap="MemberResultMap">
            <include refid="selectMember" />
            WHERE
            member.member_id = #{memberId}
        </select>

        <select id="countByContainsName" parameterType="string" resultType="_long">
            <bind name="nameContainingCondition"
            value="@org.terasoluna.gfw.common.query.QueryEscapeUtils@toStartingWithCondition(_parameter)" />
            SELECT
            COUNT(*)
            FROM
            t_member member
            <include refid="whereMember" />
        </select>

        <select id="findPageByContainsName" parameterType="string"
            resultMap="MemberResultMap">
            <bind name="nameContainingCondition"
            value="@org.terasoluna.gfw.common.query.QueryEscapeUtils@toStartingWithCondition(_parameter)" />
            <include refid="selectMember" />
            <include refid="whereMember" />
            ORDER BY member_id ASC
        </select>

        <insert id="createMember" parameterType="Member">
            <selectKey keyProperty="memberId" resultType="string" order="BEFORE">
                SELECT 'M'||TO_CHAR(NEXTVAL('s_member'),'FM000000000')
            </selectKey>            
            INSERT INTO
            t_member
            (
            member_id
            ,first_name
            ,last_name
            ,gender
            ,date_of_birth
            ,email_address
            ,telephone_number
            ,zip_code
            ,address
            ,created_at
            ,last_modified_at
            ,version
            )
            VALUES
            (
            #{memberId}
            ,#{firstName}
            ,#{lastName}
            ,#{gender}
            ,#{dateOfBirth}
            ,#{emailAddress}
            ,#{telephoneNumber}
            ,#{zipCode}
            ,#{address}
            ,#{createdAt}
            ,#{lastModifiedAt}
            ,1
            )
        </insert>

        <insert id="createCredential" parameterType="Member">
            INSERT INTO
            t_member_credential
            (
            member_id
            ,sign_id
            ,password
            ,previous_password
            ,password_last_changed_at
            ,last_modified_at
            ,version
            )
            VALUES
            (
            #{memberId}
            ,#{credential.signId}
            ,#{credential.password}
            ,#{credential.previousPassword}
            ,#{credential.passwordLastChangedAt}
            ,#{credential.lastModifiedAt}
            ,1
            )
        </insert>

        <update id="updateMember" parameterType="Member">
            UPDATE
                t_member
            SET
                first_name = #{firstName}
                ,last_name = #{lastName}
                ,gender = #{gender}
                ,date_of_birth = #{dateOfBirth}
                ,email_address = #{emailAddress}
                ,telephone_number = #{telephoneNumber}
                ,zip_code = #{zipCode}
                ,address = #{address}
                ,created_at = #{createdAt}
                ,last_modified_at = #{lastModifiedAt}
                ,version = version + 1
            WHERE
                member_id = #{memberId}
                AND version = #{version}
        </update>

        <delete id="deleteCredential" parameterType="string">
            DELETE FROM t_member_credential
            WHERE
            member_id = #{memberId}
        </delete>

        <delete id="deleteMember" parameterType="string">
            DELETE FROM t_member
            WHERE
            member_id = #{memberId}
        </delete>
        
    </mapper>


.. raw:: latex

   \newpage

