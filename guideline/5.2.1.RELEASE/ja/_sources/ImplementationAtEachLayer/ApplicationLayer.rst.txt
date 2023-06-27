アプリケーション層の実装
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

本節では、HTML formを使った画面遷移型のアプリケーションにおけるアプリケーション層の実装について説明する。

.. note::

   Ajaxの開発やREST APIの開発で必要となる実装についての説明は以下のページを参照されたい。

   - :doc:`../ArchitectureInDetail/WebApplicationDetail/Ajax`

|

アプリケーション層の実装は、以下の3つにわかれる。

#. | :ref:`controller-label`
   | Controllerは、リクエストの受付、業務処理の呼び出し、モデルの更新、Viewの決定といった処理を行い、リクエストを受けてからの一連の処理フローを制御する。
   | アプリケーション層の実装において、もっとも重要な実装となる。
#. | :ref:`formobject`
   | フォームオブジェクトは、HTML formとアプリケーションの間での値の受け渡しを行う。
#. | :ref:`view`
   | View(JSP)は、モデル（フォームオブジェクトやドメインオブジェクトなど）からデータを取得し、画面(HTML)を生成する。


.. _controller-label:

Controllerの実装
--------------------------------------------------------------------------------
| まず、Controllerの実装から説明する。
| Controllerは、以下5つの役割を担う。

#. | **リクエストを受け取るためのメソッドを提供する。**
   | \ ``@RequestMapping``\ アノテーションが付与されたメソッドを実装することで、リクエストを受け取ることができる。
#. | **リクエストパラメータの入力チェックを行う。**
   | 入力チェックが必要なリクエストを受け取るメソッドでは、\ ``@Validated``\ アノテーションをフォームオブジェクトの引数に指定することで、リクエストパラメータの入力チェックを行うことができる。
   | 単項目チェックはBean Validation、相関チェックはSpring Validator又はBean Validationでチェックを行う。
#. | **業務処理の呼び出しを行う。**
   | Controllerでは業務処理の実装は行わず、Serviceのメソッドに処理を委譲する。
#. | **業務処理の処理結果をModelに反映する。**
   | Serviceのメソッドから返却されたドメインオブジェクトを\ ``Model``\ に反映することで、Viewから処理結果を参照できるようにする。
#. | **処理結果に対応するView名を返却する。**
   | Controllerでは処理結果に対する描画処理を実装せず、描画処理はJSP等のViewで実装する。
   | Controllerでは描画処理が実装されているViewのView名の返却のみ行う。
   | View名に対応するViewの解決は、Spring Frameworkより提供されている\ ``ViewResolver``\ によって行われ、処理結果に対応するView(JSPなど）が呼び出される仕組みになっている。

.. figure:: images_ApplicationLayer/application_logic-of-controller.png
   :alt: responsibility of logic
   :width: 80%
   :align: center

   **Picture - Logic of controller**

.. note::

 Controllerでは、業務処理の呼び出し、処理結果の\ ``Model``\ への反映、遷移先(View名)の決定などの **ルーティング処理の実装に徹することを推奨する。**

|

Controllerの実装について、以下4つの点に着目して説明する。

- :ref:`controller-new-label`
- :ref:`controller_mapping-label`
- :ref:`controller_method_argument-label`
- :ref:`controller_method_return-label`

|

.. _controller-new-label:

Controllerクラスの作成方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| **Controllerは、POJOクラスに @Controller アノテーションを付加したクラス (Annotation-based Controller)として作成する。**
| Spring MVCのControllerとしては、``org.springframework.web.servlet.mvc.Controller``\ インタフェースを実装する方法 (Interface-based Controller)もあるが、Spring3以降はDeprecatedになっているため、原則使用しない。

 .. code-block:: java

    @Controller
    public class SampleController {
        // ...
    }

|
|

.. _controller_mapping-label:

リクエストとハンドラメソッドのマッピング方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| リクエストを受け取るメソッドは、\ ``@RequestMapping``\ アノテーションを付与する。
| 本ガイドラインでは、\ ``@RequestMapping``\ が付加されたメソッドのことを「ハンドラメソッド」と呼ぶ。

 .. code-block:: java

    @RequestMapping(value = "hello")
    public String hello() {
        // ...
    }

|

リクエストとハンドラメソッドをマッピングするためのルールは、\ ``@RequestMapping``\ アノテーションの属性に指定する。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 10 80

   * - 項番
     - 属性名
     - 説明
   * - 1.
     - value
     - | マッピング対象にするリクエストパスを指定する(複数可)。
   * - 2.
     - method
     - | マッピング対象にするHTTPメソッド(\ ``RequestMethod``\ 型)を指定する(複数可)。
       | GET/POSTについてはHTML form向けのリクエストをマッピングする際にも使用するが、それ以外のHTTPメソッド(PUT/DELETEなど)はREST API向けのリクエストをマッピングする際に使用する。
   * - 3.
     - params
     - | マッピング対象にするリクエストパラメータを指定する(複数可)。
       | 主にHTML form向けのリクエストをマッピングする際に使用する。このマッピング方法を使用すると、HTML form上に複数のボタンが存在する場合のマッピングを簡単に実現する事ができる。
   * - 4.
     - headers
     - | マッピング対象とするリクエストヘッダを指定する(複数可)。
       | 主にREST APIやAjax向けのリクエストをマッピングする際に使用する。
   * - 5.
     - consumes
     - | リクエストのContent-Typeヘッダを使ってマッピングすることが出来る。マッピング対象とするメディアタイプを指定する(複数可)。
       | 主にREST APIやAjax向けのリクエストをマッピングする際に使用する。
   * - 6.
     - produces
     - | リクエストのAcceptヘッダを使ってマッピングすることが出来る。マッピング対象とするメディアタイプを指定する(複数可)。
       | 主にREST APIやAjax向けのリクエストをマッピングする際に使用する。

 .. note:: **マッピングの組み合わせについて**

    複数の属性を組み合わせることで複雑なマッピングを行うことも可能だが、保守性を考慮し、可能な限りシンプルな定義になるようにマッピングの設計を行うこと。
    2つの属性の組み合わせ（value属性と別の属性1つ）を目安にすることを推奨する。

|

| 以下、マッピングの具体例を6つ示す。

- :ref:`controller-mapping-path-label`
- :ref:`controller-mapping-method-label`
- :ref:`controller-mapping-params-label`
- :ref:`controller-mapping-headers-label`
- :ref:`controller-mapping-contenttype-label`
- :ref:`controller-mapping-accept-label`

| 以降の説明では、以下のControllerクラスにハンドラメソッドを定義する前提となっている。

 .. code-block:: java
    :emphasize-lines: 1-2

    @Controller // (1)
    @RequestMapping("sample") // (2)
    public class SampleController {
        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - ``@Controller``\ アノテーションを付加することでAnnotation-basedなコントローラークラスとして認識され、component scanの対象となる。
   * - | (2)
     - クラスレベルで\ ``@RequestMapping("sample")``\ アノテーションを付けることでこのクラス内のハンドラメソッドがsample配下のURLにマッピングされる。

|

.. _controller-mapping-path-label:

リクエストパスでマッピング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
下記の定義の場合、``"sample/hello"`` というURLにアクセスすると、helloメソッドが実行される。

 .. code-block:: java

    @RequestMapping(value = "hello")
    public String hello() {

| 複数指定した場合は、OR条件で扱われる。
| 下記の定義の場合、 ``"sample/hello"`` 又は ``"sample/bonjour"`` というURLにアクセスすると、helloメソッドが実行される。

 .. code-block:: java

    @RequestMapping(value = {"hello", "bonjour"})
    public String hello() {

指定するリクエストパスは、具体的な値ではなくパターンを指定することも可能である。パターン指定の詳細は、Spring FrameworkのReference Documentを参照。

- `URI Template Patterns <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-requestmapping-uri-templates>`_\
- `URI Template Patterns with Regular Expressions <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-requestmapping-uri-templates-regex>`_\
- `Path Patterns <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-requestmapping-patterns>`_\
- `Patterns with Placeholders <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-requestmapping-placeholders>`_\

|

.. _controller-mapping-method-label:

HTTPメソッドでマッピング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
下記の定義の場合、 ``"sample/hello"`` というURLにPOSTメソッドでアクセスすると、helloメソッドが実行される。
サポートしているHTTPメソッドの一覧は `RequestMethodのJavadoc <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/web/bind/annotation/RequestMethod.html>`_ を参照されたい。
指定しない場合、サポートしている全てのHTTPメソッドがマッピング対象となる。

 .. code-block:: java

    @RequestMapping(value = "hello", method = RequestMethod.POST)
    public String hello() {


| 複数指定した場合は、OR条件で扱われる。
| 下記の定義の場合、 ``"sample/hello"`` というURLにGET又はHEADメソッドでアクセスすると、helloメソッドが実行される。

 .. code-block:: java

    @RequestMapping(value = "hello", method = {RequestMethod.GET, RequestMethod.HEAD})
    public String hello() {

|

.. _controller-mapping-params-label:

リクエストパラメータでマッピング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 下記の定義の場合、 ``"sample/hello?form"`` というURLにアクセスすると、helloメソッドが実行される。
| POSTでリクエストする場合は、リクエストパラメータはURLになくてもリクエストBODYに存在していればよい。

 .. code-block:: java

    @RequestMapping(value = "hello", params = "form")
    public String hello() {


| 複数指定した場合は、AND条件で扱われる。
| 下記の定義の場合、 ``"sample/hello?form&formType=foo"`` というURLにアクセスすると、helloメソッドが実行される。

 .. code-block:: java

    @RequestMapping(value = "hello", params = {"form", "formType=foo"})
    public String hello(@RequestParam("formType") String formType) {

サポートされている指定形式は以下の通り。

 .. tabularcolumns:: |p{0.08\linewidth}|p{0.25\linewidth}|p{0.67\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 8 25 67

   * - 項番
     - 形式
     - 説明
   * - 1.
     - paramName
     - 指定したparameNameのリクエストパラメータが存在する場合にマッピングされる。
   * - 2.
     - !paramName
     - 指定したparameNameのリクエストパラメータが存在しない場合にマッピングされる。
   * - 3.
     - paramName=paramValue
     - 指定したparameNameの値がparamValueの場合にマッピングされる。
   * - 4.
     - paramName!=paramValue
     - 指定したparameNameの値がparamValueでない場合にマッピングされる。

|

.. _controller-mapping-headers-label:

リクエストヘッダでマッピング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
主にREST APIやAjax向けのリクエストをマッピングする際に使用するため、詳細は以下のページを参照されたい。

- :doc:`../ArchitectureInDetail/WebApplicationDetail/Ajax`


.. _controller-mapping-contenttype-label:

Content-Typeヘッダでマッピング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
主にREST APIやAjax向けのリクエストをマッピングする際に使用するため、詳細は以下のページを参照されたい。

- :doc:`../ArchitectureInDetail/WebApplicationDetail/Ajax`


.. _controller-mapping-accept-label:

Acceptヘッダでマッピング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
主にREST APIやAjax向けのリクエストをマッピングする際に使用するため、詳細は以下のページを参照されたい。

- :doc:`../ArchitectureInDetail/WebApplicationDetail/Ajax`

|
|

.. _controller-mapping-policy-label:

リクエストとハンドラメソッドのマッピング方針
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
以下の方針でマッピングを行うことを推奨する。

- | **業務や機能といった意味のある単位で、リクエストのURLをグループ化する。**
  | URLのグループ化とは、 \ ``@RequestMapping(value = "xxx")``\ をクラスレベルのアノテーションとして定義することを意味する。

- | **処理内の画面フローで使用するリクエストのURLは、同じURLにする。**
  | 同じURLとは \ ``@RequestMapping(value = "xxx")``\ のvalue属性の値を同じ値にすることを意味する。
  | 処理内の画面フローで使用するハンドラメソッドの切り替えは、HTTPメソッドとHTTPパラメータによって行う。

以下にベーシックな画面フローを行うサンプルアプリケーションを例にして、リクエストとハンドラメソッドの具体的なマッピング例を示す。

 * :ref:`controller-mapping-policy-sampleapp-overview-label`
 * :ref:`controller-mapping-policy-sampleapp-url-design-label`
 * :ref:`controller-mapping-policy-sampleapp-mapping-design-label`
 * :ref:`controller-mapping-policy-sampleapp-form-impl-label`
 * :ref:`controller-mapping-policy-sampleapp-confirm-impl-label`
 * :ref:`controller-mapping-policy-sampleapp-redo-impl-label`
 * :ref:`controller-mapping-policy-sampleapp-create-impl-label`

|

.. _controller-mapping-policy-sampleapp-overview-label:

サンプルアプリケーションの概要
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
サンプルアプリケーションの機能概要は以下の通り。

- | EntityのCRUD処理を行う機能を提供する。
- | 以下の5つの処理を提供する。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 70

    * - 項番
      - 処理名
      - 処理概要
    * - 1.
      - Entity一覧取得
      - 作成済みのEntityを全て取得し、一覧画面に表示する。
    * - 2.
      - Entity新規作成
      - 指定した内容で新たにEntityを作成する。処理内には、画面フロー（フォーム画面、確認画面、完了画面）が存在する。
    * - 3.
      - Entity参照
      - 指定されたIDのEntityを取得し、詳細画面に表示する。
    * - 4.
      - Entity更新
      - 指定されたIDのEntityを更新する。処理内には、画面フロー（フォーム画面、確認画面、完了画面）が存在する。
    * - 5.
      - Entity削除
      - 指定されたIDのEntityを削除する。

- | 機能全体の画面フローは以下の通り。
  | 画面フロー図には記載していないが、入力チェックエラーが発生した場合はフォーム画面を再描画するものとする。

.. figure:: images_ApplicationLayer/application_sample-screen-flow.png
   :alt: Screen flow of entity management function
   :width: 90%
   :align: center

   **Picture - Screen flow of entity management function**

|

.. _controller-mapping-policy-sampleapp-url-design-label:

リクエストURL
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
必要となるリクエストのURLの設計を行う。

- | 機能内で必要となるリクエストのリクエストURLをグループ化する。
  | ここではAbcというEntityのCRUD操作を行う機能となるので、 ``"/abc/"`` から始まるURLとする。

- 処理毎にリクエストURLを設ける。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 30 60

    * - 項番
      - 処理名
      - 処理毎のURL(パターン)
    * - 1.
      - Entity一覧取得
      - /abc/list
    * - 2.
      - Entity新規作成
      - /abc/create
    * - 3.
      - Entity参照
      - /abc/{id}
    * - 4.
      - Entity更新
      - /abc/{id}/update
    * - 5.
      - Entity削除
      - /abc/{id}/delete

 .. note::

     Entity参照、Entity更新、Entity削除処理のURL内に指定している ``"{id}"`` は、`URI Template Patterns <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-requestmapping-uri-templates>`_\ と呼ばれ、任意の値を指定する事ができる。
     サンプルアプリケーションでは、操作するEntityのIDを指定する。

 画面フロー図に各処理に割り振られたURLをマッピングすると以下のようになる。

.. figure:: images_ApplicationLayer/application_sample-screen-flow-assigned-url.png
   :alt: Screen flow of entity management function and assigned URL
   :width: 90%
   :align: center

   **Picture - Screen flow of entity management function and assigned URL**

|

.. _controller-mapping-policy-sampleapp-mapping-design-label:

リクエストとハンドラメソッドのマッピング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| リクエストとハンドラメソッドのマッピングの設計を行う。
| 以下は、マッピング方針に則って設計したマッピング定義となる。

 .. tabularcolumns:: |p{0.05\linewidth}|p{0.20\linewidth}|p{0.15\linewidth}|p{0.20\linewidth}|p{0.10\linewidth}|p{0.10\linewidth}|p{0.15\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 5 20 15 22 10 13 15
   :class: longtable

   * - | 項番
     - | 処理名
     - | URL
     - | リクエスト名
     - | HTTP
       | メソッド
     - | HTTP
       | パラメータ
     - | ハンドラメソッド
   * - 1.
     - Entity一覧取得
     - /abc/list
     - 一覧表示
     - GET
     - \-
     - list
   * - 2.
     - Entity新規作成
     - /abc/create
     - フォーム表示
     - \-
     - form
     - createForm
   * - 3.
     -
     -
     - 入力内容確認表示
     - POST
     - confirm
     - createConfirm
   * - 4.
     -
     -
     - フォーム再表示
     - POST
     - redo
     - createRedo
   * - 5.
     -
     -
     - 新規作成
     - POST
     - \-
     - create
   * - 6.
     -
     -
     - 新規作成完了表示
     - GET
     - complete
     - createComplete
   * - 7.
     - Entity参照
     - /abc/{id}
     - 詳細表示
     - GET
     - \-
     - read
   * - 8.
     - Entity更新
     - /abc/{id}/update
     - フォーム表示
     - \-
     - form
     - updateForm
   * - 9.
     -
     -
     - 入力内容確認表示
     - POST
     - confirm
     - updateConfirm
   * - 10.
     -
     -
     - フォーム再表示
     - POST
     - redo
     - updateRedo
   * - 11.
     -
     -
     - 更新
     - POST
     - \-
     - update
   * - 12.
     -
     -
     - 更新完了表示
     - GET
     - complete
     - updateComplete
   * - 13.
     - Entity削除
     - /abc/{id}/delete
     - 削除
     - POST
     - \-
     - delete
   * - 14.
     -
     -
     - 削除完了表示
     - GET
     - complete
     - deleteComplete

 .. raw:: latex

    \newpage

| Entity新規作成、Entity更新、Entity削除処理では、処理内に複数のリクエストが存在しているため、HTTPメソッドとHTTPパラメータによってハンドラメソッドを切り替えている。
| 以下に、Entity新規作成処理を例に、処理内に複数のリクエストが存在する場合のリクエストフローを示す。
| URLは全て ``"/abc/create"`` で、HTTPメソッドとHTTPパラメータの組み合わせでハンドラメソッドを切り替えている点に注目すること。

.. figure:: images_ApplicationLayer/applicationScreenflow.png
   :alt: Request flow of entity create processing
   :width: 90%
   :align: center

   **Picture - Request flow of entity create processing**

|

| 以下に、Entity新規作成処理のハンドラメソッドの実装コードを示す。
| ここではリクエストとハンドラメソッドのマッピングについて理解してもらうのが目的なので、\ ``@RequestMapping``\ の書き方に注目すること。
| ハンドラメソッドの引数や返り値(View名及びView)の詳細については、次章以降で説明する。

- :ref:`controller-mapping-policy-sampleapp-form-impl-label`
- :ref:`controller-mapping-policy-sampleapp-confirm-impl-label`
- :ref:`controller-mapping-policy-sampleapp-redo-impl-label`
- :ref:`controller-mapping-policy-sampleapp-create-impl-label`
- :ref:`controller-mapping-policy-sampleapp-complete-impl-label`
- :ref:`controller-mapping-policy-sampleapp-multi-impl-label`

|

.. _controller-mapping-policy-sampleapp-form-impl-label:

フォーム表示の実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
フォーム表示する場合は、HTTPパラメータとして ``form`` を指定させる。

 .. code-block:: java
    :emphasize-lines: 1,4

    @RequestMapping(value = "create", params = "form") // (1)
    public String createForm(AbcForm form, Model model) {
        // omitted
        return "abc/createForm"; // (2)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - params属性に ``"form"`` を指定する。
   * - | (2)
     - フォーム画面を描画するためのJSPのView名を返却する。

 .. note::
    この処理でHTTPメソッドをGETに限る必要がないのでmethod属性を指定していない。

|

以下に、ハンドラメソッド以外の部分の実装例についても説明しておく。

フォーム表示を行う場合、ハンドラメソッドの実装以外に、

- フォームオブジェクトの生成処理の実装。フォームオブジェクトの詳細は、 :ref:`formobject` を参照されたい。
- フォーム画面のViewの実装。Viewの詳細は、 :ref:`view` を参照されたい。

が必要になる。

以下のフォームオブジェクトを使用する。

 .. code-block:: java

  public class AbcForm implements Serializable {
      private static final long serialVersionUID = 1L;

      @NotEmpty
      private String input1;

      @NotNull
      @Min(1)
      @Max(10)
      private Integer input2;

      // omitted setter&getter
  }

フォームオブジェクトを生成する。

 .. code-block:: java

    @ModelAttribute
    public AbcForm setUpAbcForm() {
        return new AbcForm();
    }


フォーム画面のView(JSP)を作成する。

 .. code-block:: jsp
    :emphasize-lines: 12

    <h1>Abc Create Form</h1>
    <form:form modelAttribute="abcForm"
      action="${pageContext.request.contextPath}/abc/create">
      <form:label path="input1">Input1</form:label>
      <form:input path="input1" />
      <form:errors path="input1" />
      <br>
      <form:label path="input2">Input2</form:label>
      <form:input path="input2" />
      <form:errors path="input2" />
      <br>
      <input type="submit" name="confirm" value="Confirm" /> <!-- (1) -->
    </form:form>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - 確認画面へ遷移するためのsubmitボタンには\ ``name="confirm"``\ というパラメータを指定しておく。

|

以下に、フォーム表示の動作について説明する。

| フォーム表示処理を呼び出す。
| ``"abc/create?form"`` というURIにアクセスする。
| ``form`` というHTTPパラメータの指定があるため、ControllerのcreateFormメソッドが呼び出されフォーム画面が表示される。

 .. figure:: images_ApplicationLayer/applicationCreateFormDisplay.png
   :width: 90%

|

.. _controller-mapping-policy-sampleapp-confirm-impl-label:

入力内容確認表示の実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
フォームの入力内容を確認する場合は、POSTメソッドでデータを送信し、HTTPパラメータに ``confirm`` を指定させる。

 .. code-block:: java
    :emphasize-lines: 1,5,8

    @RequestMapping(value = "create", method = RequestMethod.POST, params = "confirm") // (1)
    public String createConfirm(@Validated AbcForm form, BindingResult result,
            Model model) {
        if (result.hasErrors()) {
            return createRedo(form, model); // return "abc/createForm"; (2)
        }
        // omitted
        return "abc/createConfirm"; // (3)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - method属性に ``RequestMethod.POST`` 、params属性に ``"confirm"`` を指定する。
   * - | (2)
     - 入力チェックエラーが発生した場合の処理は、フォーム再表示用のハンドラメソッドを呼び出すことを推奨する。フォーム画面を再表示するための処理の共通化を行うことができる。
   * - | (3)
     - 入力内容確認画面を描画するためのJSPのView名を返却する。

 .. note::
    POSTメソッドを指定させる理由は、個人情報やパスワードなどの秘密情報がブラウザのアドレスバーに現れ、他人に容易に閲覧されることを防ぐためである。
    (もちろんセキュリティ対策としては十分ではなく、SSLなどのセキュアなサイトにする必要がある)。

|

以下に、ハンドラメソッド以外の部分の実装例についても説明しておく。

入力内容確認表示を行う場合、ハンドラメソッドの実装以外に、

- 入力内容確認画面のViewの実装。Viewの詳細は、 :ref:`view` を参照されたい。

が必要になる。

入力内容確認画面のView(JSP)を作成する。

 .. code-block:: jsp
    :emphasize-lines: 6,10,12-13

    <h1>Abc Create Form</h1>
    <form:form modelAttribute="abcForm"
      action="${pageContext.request.contextPath}/abc/create">
      <form:label path="input1">Input1</form:label>
      ${f:h(abcForm.input1)}
      <form:hidden path="input1" /> <!-- (1) -->
      <br>
      <form:label path="input2">Input2</form:label>
      ${f:h(abcForm.input2)}
      <form:hidden path="input2" /> <!-- (1) -->
      <br>
      <input type="submit" name="redo" value="Back" /> <!-- (2) -->
      <input type="submit" value="Create" /> <!-- (3) -->
    </form:form>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - フォーム画面で入力された値は、Createボタン及びBackボタンが押下された際に再度サーバに送る必要があるため、HTML formのhidden項目とする。
   * - | (2)
     - フォーム画面に戻るためのsubmitボタンには\ ``name="redo"``\ というパラメータを指定しておく。
   * - | (3)
     - 新規作成を行うためのsubmitボタンにはパラメータ名の指定は不要。

 .. note::
    この例では確認項目を表示する際にHTMLエスケープするため、 ``f:h()`` 関数を使用している。
    XSS対策のため、必ず行うこと。詳細については :doc:`Cross Site Scripting <../Security/XSS>` を参照されたい。

|

以下に、入力内容確認の動作について説明する。

| 入力内容確認表示処理を呼び出す。
| フォーム画面でInput1に ``"aa"`` を、Input2に ``"5"`` を入力し、Confirmボタンを押下する。
| Confirmボタンを押下すると、 ``"abc/create?confirm"`` というURIにPOSTメソッドでアクセスする。
| ``confirm`` というHTTPパラメータがあるため、ControllerのcreateConfirmメソッドが呼び出され、入力内容確認画面が表示される。

 .. figure:: images_ApplicationLayer/applicationCreateConfirmDisplay.png
   :width: 90%

Confirmボタンを押下するとPOSTメソッドでHTTPパラメータが送信されるため、URIには現れていないが、HTTPパラメータとして ``confirm`` が含まれている。

 .. figure:: images_ApplicationLayer/applicationCreateConfirmNetwork.png
   :width: 90%

|

.. _controller-mapping-policy-sampleapp-redo-impl-label:

フォーム再表示の実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
フォームを再表示する場合は、HTTPパラメータにredoを指定させる。

 .. code-block:: java
    :emphasize-lines: 1,4

    @RequestMapping(value = "create", method = RequestMethod.POST, params = "redo") // (1)
    public String createRedo(AbcForm form, Model model) {
        // omitted
        return "abc/createForm"; // (2)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - method属性に ``RequestMethod.POST`` 、params属性に ``"redo"`` を指定する。
   * - | (2)
     - 入力内容確認画面を描画するためのJSPのView名を返却する。

|

以下に、フォーム再表示の動作について説明する。

| フォーム再表示リクエストを呼び出す。
| 入力内容確認画面で、Backボタンを押下する。
| Backボタンを押下すると、 ``abc/create?redo`` というURIにPOSTメソッドでアクセスする。
| ``redo`` というHTTPパラメータがあるため、ControllerのcreateRedoメソッドが呼び出され、フォーム画面が再表示される。

 .. figure:: images_ApplicationLayer/applicationCreateConfirmDisplay.png
   :width: 90%

Backボタンを押下するとPOSTメソッドでHTTPパラメータが送信されるため、URIには現れていないが、HTTPパラメータとして ``redo`` が含まれている。
また、フォームの入力値をhidden項目として送信されるため、フォーム画面で入力値を復元することが出来る。

 .. figure:: images_ApplicationLayer/applicationBackToCreateFormDisplay.png
   :width: 90%

 .. figure:: images_ApplicationLayer/applicationBackToCreateFormNetwork.png
   :width: 90%

.. note::

    戻るボタンの実現方法には、ボタンの属性に ``onclick="javascript:history.back()"`` を設定する方法もある。
    両者では以下が異なり、要件に応じて選択する必要がある。

    * ブラウザの戻るボタンを押した場合の挙動
    * 戻るボタンがあるページに直接アクセスして戻るボタンを押した場合の挙動
    * ブラウザの履歴

|

.. _controller-mapping-policy-sampleapp-create-impl-label:

新規作成の実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| フォームの入力内容を登録する場合は、POSTで登録対象のデータ(hiddenパラメータ)を送信させる。
| 新規作成リクエストはこの処理のメインリクエストになるので、HTTPパラメータによる振り分けは行っていない。
| この処理ではデータベースの状態を変更するので、二重送信によって新規作成処理が複数回実行されないように制御する必要がある。
| そのため、この処理が終了した後はView(画面)を直接表示するのではなく、次の画面(新規作成完了画面)へリダイレクトしている。このパターンをPOST-Redirect-GET(PRG)パターンと呼ぶ。  :abbr:`PRG (Post-Redirect-Get)` パターンの詳細については :doc:`../ArchitectureInDetail/WebApplicationDetail/DoubleSubmitProtection` を参照されたい。

 .. code-block:: java
    :emphasize-lines: 1,7

    @RequestMapping(value = "create", method = RequestMethod.POST) // (1)
    public String create(@Validated AbcForm form, BindingResult result, Model model) {
        if (result.hasErrors()) {
            return createRedo(form, model); // return "abc/createForm";
        }
        // omitted
        return "redirect:/abc/create?complete"; // (2)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - method属性に ``RequestMethod.POST`` を指定し、params属性は指定しない。
   * - | (2)
     -  :abbr:`PRG (Post-Redirect-Get)` パターンとするため、新規作成完了表示リクエストにリダイレクトするためのURLをView名として返却する。

 .. note::
    "redirect:/xxx"を返却すると"/xxx"へリダイレクトさせることができる。

.. warning::
    PRGパターンとすることで、ブラウザのF5ボタン押下時のリロードによる二重送信を防ぐ事はできるが、二重送信の対策としてはとしては十分ではない。
    二重送信の対策としては、共通部品として提供しているTransactionTokenCheckを行う必要がある。
    TransactionTokenCheckの詳細については :doc:`../ArchitectureInDetail/WebApplicationDetail/DoubleSubmitProtection` を参照されたい。

|

以下に、「新規作成」の動作について説明する。

| 新規作成処理を呼び出す。
| 入力内容確認画面で、Createボタンを押下する。
| Createボタンを押下すると、 ``"abc/create"`` というURIにPOSTメソッドでアクセスする。
| ボタンを識別するためのHTTPパラメータを送信していないので、Entity新規作成処理のメインのリクエストと判断され、Controllerのcreateメソッドが呼び出される。

| 新規作成リクエストでは、直接画面を返さず、新規作成完了表示( ``"/abc/create?complete"`` )へリダイレクトしているため、HTTPステータスが302になっている。

 .. figure:: images_ApplicationLayer/applicationCreateNetwork.png
   :width: 90%


|

.. _controller-mapping-policy-sampleapp-complete-impl-label:

新規作成完了表示の実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
新規作成処理が完了した事を通知する場合は、HTTPパラメータに ``complete`` を指定させる。

 .. code-block:: java
    :emphasize-lines: 1,4

    @RequestMapping(value = "create", params = "complete") // (1)
    public String createComplete() {
        // omitted
        return "abc/createComplete"; // (2)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - params属性に ``"complete"`` を指定する。
   * - | (2)
     - 新規作成完了画面を描画するため、JSPのView名を返却する。

 .. note::
    この処理もHTTPメソッドをGETに限る必要がないのでmethod属性を指定しなくても良い。

|

以下に、「新規作成完了表示」の動作について説明する。

| 新規作成完了後、リダイレクト先に指定されたURI( ``"/abc/create?complete"`` )にアクセスする。
| ``complete`` というHTTPパラメータがあるため、ControllerのcreateCompleteメソッドが呼び出され、新規作成完了画面が表示される。


 .. figure:: images_ApplicationLayer/applicationCreateCompleteDisplay.png
   :width: 90%

 .. figure:: images_ApplicationLayer/applicationCreateCompleteNetwork.png
   :width: 90%

 .. note::
    PRGパターンを利用しているため、ブラウザをリロードしても、新規作成処理は実行されず、新規作成完了が再度表示されるだけである。

|

.. _controller-mapping-policy-sampleapp-multi-impl-label:

HTML form上に複数のボタンを配置する場合の実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
1つのフォームに対して複数のボタンを設置したい場合、ボタンを識別するためのHTTPパラメータを送ることで、
実行するハンドラメソッドを切り替える。
ここではサンプルアプリケーションの入力内容確認画面のCreateボタンとBackボタンを例に説明する。

下図のように、入力内容確認画面のフォームには、新規作成を行うCreateボタンと新規作成フォーム画面を再表示するBackボタンが存在する。

.. figure:: images_ApplicationLayer/applicationControllerBackToForm.png
   :alt: Multiple button in the HTML form
   :width: 80%
   :align: center

   **Picture - Multiple button in the HTML form**

Backボタンを押下した場合、新規作成フォーム画面を再表示するためのリクエスト( ``"/abc/create?redo"`` )を送信する必要があるため、
HTML form内に以下のコードが必要となる。

 .. code-block:: jsp
    :emphasize-lines: 1

    <input type="submit" name="redo" value="Back" /> <!-- (1) -->
    <input type="submit" value="Create" />

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - 上記のように、入力内容確認画面( ``"abc/createConfirm.jsp"`` )のBackボタンに\ ``name="redo"``\ というパラメータを指定する。

Backボタン押下時の動作については、 :ref:`controller-mapping-policy-sampleapp-redo-impl-label` を参照されたい。

|

.. _controller-mapping-policy-sampleapp-all-impl-label:

サンプルアプリケーションのControllerのソースコード
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 以下に、サンプルアプリケーションの新規作成処理実装後のControllerの全ソースを示す。
| Entity一覧取得、Entity参照、Entity更新、Entity削除も同じ要領で実装することになるが、説明は割愛する。

 .. code-block:: java

    @Controller
    @RequestMapping("abc")
    public class AbcController {

        @ModelAttribute
        public AbcForm setUpAbcForm() {
            return new AbcForm();
        }

        // Handling request of "/abc/create?form"
        @RequestMapping(value = "create", params = "form")
        public String createForm(AbcForm form, Model model) {
            // omitted
            return "abc/createForm";
        }

        // Handling request of "POST /abc/create?confirm"
        @RequestMapping(value = "create", method = RequestMethod.POST, params = "confirm")
        public String createConfirm(@Validated AbcForm form, BindingResult result,
                Model model) {
            if (result.hasErrors()) {
                return createRedo(form, model);
            }
            // omitted
            return "abc/createConfirm";
        }

        // Handling request of "POST /abc/create?redo"
        @RequestMapping(value = "create", method = RequestMethod.POST, params = "redo")
        public String createRedo(AbcForm form, Model model) {
            // omitted
            return "abc/createForm";
        }

        // Handling request of "POST /abc/create"
        @RequestMapping(value = "create", method = RequestMethod.POST)
        public String create(@Validated AbcForm form, BindingResult result, Model model) {
            if (result.hasErrors()) {
                return createRedo(form, model);
            }
            // omitted
            return "redirect:/abc/create?complete";
        }

        // Handling request of "/abc/create?complete"
        @RequestMapping(value = "create", params = "complete")
        public String createComplete() {
            // omitted
            return "abc/createComplete";
        }

    }

|
|

.. _controller_method_argument-label:

ハンドラメソッドの引数について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`ハンドラメソッドの引数は様々な値をとることができる <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-arguments>`_ が、
基本的には次に挙げるものは原則として使用しないこと。

* ServletRequest
* HttpServletRequest
* org.springframework.web.context.request.WebRequest
* org.springframework.web.context.request.NativeWebRequest
* java.io.InputStream
* java.io.Reader
* java.io.OutputStream
* java.io.Writer
* java.util.Map
* org.springframework.ui.ModelMap

.. note::
    ``HttpServletRequest`` のgetAttribute/setAttribute
    や ``Map`` のget/putのような汎用的なメソッドの利用を許可すると自由な値の受け渡しができてしまい、
    プロジェクトの規模が大きくなると保守性を著しく低下させる可能性がある。

    同様の理由で、他で代替できる場合は ``HttpSession`` を極力使用しないことを推奨する。

    共通的なパラメータ(リクエストパラメータ)をJavaBeanに格納してControllerの引数に渡したい場合は
    後述の :ref:`methodargumentresolver` を使用することで実現できる。

|

以下に、引数の使用方法について、目的別に13例示す。

- :ref:`controller_method_argument-model-label`
- :ref:`controller_method_argument-pathvariable-label`
- :ref:`controller_method_argument-requestparam-label`
- :ref:`controller_method_argument-form-label`
- :ref:`controller_method_argument-validation-label`
- :ref:`controller_method_argument-redirectattributes-label`
- :ref:`controller_method_argument-redirectattributes-param-label`
- :ref:`controller_method_argument-redirectattributes-path-label`
- :ref:`controller_method_argument-cookievalue-label`
- :ref:`controller_method_argument-cookiewrite-label`
- :ref:`controller_method_argument-pagination-label`
- :ref:`controller_method_argument-upload-label`
- :ref:`controller_method_argument-message-label`

|

.. _controller_method_argument-model-label:

画面(View)にデータを渡す
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
画面(View)に表示するデータを渡したい場合は、``org.springframework.ui.Model``\ (以降 ``Model`` と呼ぶ) をハンドラメソッドの引数として受け取り、
\ ``Model``\ オブジェクトに渡したいデータ(オブジェクト)を追加する。

- SampleController.java

 .. code-block:: java
    :emphasize-lines: 2-4

    @RequestMapping("hello")
    public String hello(Model model) { // (1)
        model.addAttribute("hello", "Hello World!"); // (2)
        model.addAttribute(new HelloBean("Bean Hello World!")); // (3)
        return "sample/hello"; // returns view name
    }

- hello.jsp

 .. code-block:: jsp
    :emphasize-lines: 1-2

    Message : ${f:h(hello)}<br> <%-- (4) --%>
    Message : ${f:h(helloBean.message)}<br> <%-- (5) --%>

- HTML of created by View(hello.jsp)

 .. code-block:: html
    :emphasize-lines: 1-2

    Message : Hello World!<br> <!-- (6) -->
    Message : Bean Hello World!<br>　<!-- (6) -->


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - \ ``Model``\ オブジェクトを引数として受け取る。
   * - | (2)
     - | 引数で受け取った\ ``Model``\ オブジェクトの\ ``addAttribute``\ メソッドを呼び出し、渡したいデータを\ ``Model``\ オブジェクトに追加する。
       | 例では、``"hello"`` という属性名で ``"HelloWorld!"`` という文字列のデータを追加している。
   * - | (3)
     - | \ ``addAttribute``\ メソッドの第一引数を省略すると値のクラス名の先頭を小文字にした文字列が属性名になる。
       | 例では、 ``model.addAttribute("helloBean", new HelloBean());`` を行ったのと同じ結果となる。
   * - | (4)
     - | View(JSP)側では、「${属性名}」と記述することで\ ``Model``\ オブジェクトに追加したデータを取得することができる。
       | 例ではHTMLエスケープを行うEL式の関数を呼び出しているため、「${f:h(属性名)}」としている。
       | HTMLエスケープを行うEL式の関数の詳細については、 :doc:`Cross Site Scripting <../Security/XSS>` を参照されたい。
   * - | (5)
     - | 「${属性名.JavaBeanのプロパティ名}」と記述することで\ ``Model``\に格納されているJavaBeanから値を取得することができる。
   * - | (6)
     - | JSP実行後に出力されるHTML。

 .. note::
  \ ``Model``\ は使用しない場合でも引数に指定しておいてもよい。実装初期段階では必要なくても
  後で使う場合がある(後々メソッドのシグニチャを変更する必要がなくなる)。

 .. note::
  ``Model`` に ``addAttribute`` することで、 ``HttpServletRequest`` に ``setAttribute`` されるため、
  Spring MVCの管理下にないモジュール(例えばServletFilterなど)からも値を参照することが出来る。

|

.. _controller_method_argument-pathvariable-label:

URLのパスから値を取得する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| URLのパスから値を取得する場合は、引数に\ ``@PathVariable``\ アノテーションを付与する。
| \ ``@PathVariable``\ アノテーションを使用してパスから値を取得する場合、 \ ``@RequestMapping``\ アノテーションのvalue属性に取得したい部分を変数化しておく必要がある。

 .. code-block:: java
    :emphasize-lines: 1,3,4

    @RequestMapping("hello/{id}/{version}") // (1)
    public String hello(
            @PathVariable("id") String id, // (2)
            @PathVariable Integer version, // (3)
            Model model) {
        // do something
        return "sample/hello"; // returns view name
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``@RequestMapping``\ アノテーションのvalue属性に、抜き出したい箇所をパス変数として指定する。パス変数は、「{変数名}」の形式で指定する。
       | 上記例では、 ``"id"`` と ``"version"`` という二つのパス変数を指定している。
   * - | (2)
     - | \ ``@PathVariable``\ アノテーションのvalue属性には、パス変数の変数名を指定する。
       | 上記例では、 ``"sample/hello/aaaa/1"`` というURLにアクセスした場合、引数idに文字列 ``"aaaa"`` が渡る。
   * - | (3)
     - | ``@PathVariable``\ アノテーションのvalue属性は省略可能で、省略した場合は引数名がリクエストパラメータ名となる。
       | 上記例では、 ``"sample/hello/aaaa/1"`` というURLにアクセスした場合、引数versionに数値 ``"1"`` が渡る。
       | ただしこの方法は、

       * \ ``-g``\ オプション(デバッグ情報を出力するモード)
       * Java8から追加された\ ``-parameters``\ オプション(メソッド・パラメータにリフレクション用のメタデータを生成するモード)

       のどちらかを指定してコンパイルする必要がある。

 .. note::
    バインドする引数の型はString以外でも良い。型が合わない場合は\ ``org.springframework.beans.TypeMismatchException``\ がスローされ、デフォルトの動作は400(Bad Request)が応答される。
    例えば、上記例で ``"sample/hello/aaaa/v1"`` というURLでアクセスした場合、``"v1"`` をIntegerに変換できないため、例外がスローされる。

 .. warning::
    ``@PathVariable``\ アノテーションのvalue属性を省略する場合、デプロイするアプリケーションは\ ``-g``\ オプション又はJava8から追加された\ ``-parameters``\ オプションを指定してコンパイルする必要がある。
    これらのオプションを指定した場合、コンパイル後のクラスにはデバッグ時に必要となる情報や処理などが挿入されるため、メモリや処理性能に影響を与えることがあるので注意が必要である。
    基本的には、value属性を明示的に指定する方法を推奨する。

|

.. _controller_method_argument-requestparam-label:

リクエストパラメータを個別に取得する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
リクエストパラメータを1つずつ取得したい場合は、引数に\ ``@RequestParam``\ アノテーションを付与する。

 .. code-block:: java
    :emphasize-lines: 3-6

    @RequestMapping("bindRequestParams")
    public String bindRequestParams(
            @RequestParam("id") String id, // (1)
            @RequestParam String name, // (2)
            @RequestParam(value = "age", required = false) Integer age, // (3)
            @RequestParam(value = "genderCode", required = false, defaultValue = "unknown") String genderCode, // (4)
            Model model) {
        // do something
        return "sample/hello"; // returns view name
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``@RequestParam``\ アノテーションのvalue属性には、リクエストパラメータ名を指定する。
       | 上記例では、 ``"sample/hello?id=aaaa"`` というURLにアクセスした場合、引数idに文字列 ``"aaaa"`` が渡る。
   * - | (2)
     - | ``@RequestParam``\ アノテーションのvalue属性は省略可能で、省略した場合は引数名がリクエストパラメータ名となる。
       | 上記例では、 ``"sample/hello?name=bbbb&...."`` というURLにアクセスした場合、引数nameに文字列 ``"bbbb"`` が渡る。
       | ただしこの方法は、

       * \ ``-g``\ オプション(デバッグ情報を出力するモード)
       * Java8から追加された\ ``-parameters``\ オプション(メソッド・パラメータにリフレクション用のメタデータを生成するモード)

       のどちらかを指定してコンパイルする必要がある。
   * - | (3)
     - | デフォルトの動作では、指定したリクエストパラメータが存在しないとエラーとなる。リクエストパラメータが存在しないケースを許容する場合は、required属性を ``false`` に指定する。
       | 上記例では、``age`` というリクエストパラメータがない状態でアクセスした場合、引数ageに\ ``null``\ が渡る。
   * - | (4)
     - | 指定したリクエストパラメータが存在しない場合にデフォルト値を使用したい場合は、defaultValue属性にデフォルト値を指定する。
       | 上記例では、``genderCode`` というリクエストパラメータがない状態でアクセスした場合、引数genderCodeに ``"unknown"`` が渡る。


 .. note::
    必須パラメータを指定しないでアクセスした場合は、\ ``org.springframework.web.bind.MissingServletRequestParameterException``\ がスローされ、デフォルトの動作は400(Bad Request)が応答される。
    ただし、defaultValue属性を指定している場合は例外はスローされず、defaultValue属性で指定した値が渡る。

 .. note::
    バインドする引数の型はString以外でも良い。型が合わない場合は\ ``org.springframework.beans.TypeMismatchException``\ がスローされ、デフォルトの動作は400(Bad Request)が応答される。
    例えば、上記例で ``"sample/hello?age=aaaa&..."`` というURLでアクセスした場合、 ``"aaaa"`` をIntegerに変換できないため、例外がスローされる。

|

**以下の条件に当てはまる場合は、次に説明するフォームオブジェクトにバインドすること。**

- リクエストパラメータがHTML form内の項目である。
- リクエストパラメータはHTML form内の項目ではないが、リクエストパラメータに必須チェック以外の入力チェックを行う必要がある。
- リクエストパラメータの入力チェックエラーのエラー詳細をパラメータ毎に出力する必要がある。
- 3つ以上のリクエストパラメータをバインドする。(保守性、可読性の観点)

|

.. _controller_method_argument-form-label:

リクエストパラメータをまとめて取得する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| リクエストパラメータをオブジェクトにまとめて取得する場合は、フォームオブジェクトを使用する。
| フォームオブジェクトは、HTML formを表現するJavaBeanである。フォームオブジェクトの詳細は :ref:`formobject` を参照されたい。

以下は、``@RequestParam`` で個別にリクエストパラメータを受け取っていたハンドラメソッドを、フォームオブジェクトで受け取るように変更した場合の実装例である。

``@RequestParam`` を使って個別にリクエストパラメータを受け取っているハンドラメソッドは以下の通り。

 .. code-block:: java

    @RequestMapping("bindRequestParams")
    public String bindRequestParams(
            @RequestParam("id") String id,
            @RequestParam String name,
            @RequestParam(value = "age", required = false) Integer age,
            @RequestParam(value = "genderCode", required = false, defaultValue = "unknown") String genderCode,
            Model model) {
        // do something
        return "sample/hello"; // returns view name
    }

| フォームオブジェクトクラスを作成する。
| このフォームオブジェクトに対応するHTML formのjspは :ref:`formobjectjsp` を参照されたい。

 .. code-block:: java

    public class SampleForm implements Serializable{
        private static final long serialVersionUID = 1477614498217715937L;

        private String id;
        private String name;
        private Integer age;
        private String genderCode;

        // omit setters and getters

    }

 .. note::
  **リクエストパラメータ名とフォームオブジェクトのプロパティ名は一致させる必要がある。**

  上記のフォームオブジェクトに対して ``"id=aaa&name=bbbb&age=19&genderCode=men?tel=01234567"`` というパラメータが送信された場合、
  ``id`` , ``name`` , ``age`` , ``genderCode`` は名前が一致するプロパティに値が格納されるが、 ``tel`` は名前が一致するプロパティがないため、フォームオブジェクトに取り込まれない。

``@RequestParam`` を使って個別に受け取っていたリクエストパラメータをフォームオブジェクトとして受け取るようにする。

 .. code-block:: java
    :emphasize-lines: 2

    @RequestMapping("bindRequestParams")
    public String bindRequestParams(@Validated SampleForm form, // (1)
            BindingResult result,
            Model model) {
        // do something
        return "sample/hello"; // returns view name
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - \ ``SampleForm``\ オブジェクトを引数として受け取る。

 .. note::
    フォームオブジェクトを引数に用いた場合、\ ``@RequestParam``\ の場合とは異なり、
    必須チェックは行われない。\ **フォームオブジェクトを使用する場合は、次に説明する** :ref:`controller_method_argument-validation-label` **を行うこと**\ 。

.. warning::
    EntityなどDomainオブジェクトをそのままフォームオブジェクトとして使うこともできるが、
    実際には、WEBの画面上にしか存在しないパラメータ（確認用パスワードや、規約確認チェックボックス等）が存在する。
    Domainオブジェクトにそのような画面項目に依存する項目を入れるべきではないので、Domainオブジェクトとは別にフォームオブジェクト用のクラスを作成することを推奨する。
    リクエストパラメータからDomainオブジェクトを作成する場合は、一旦フォームオブジェクトにバインドしてからプロパティ値をDomainオブジェクトにコピーすること。

|

.. _controller_method_argument-validation-label:

入力チェックを行う
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
リクエストパラメータがバインドされているフォームオブジェクトに対して入力チェックを行う場合は、
フォームオブジェクト引数に\ ``@Validated``\ アノテーションを付け、
フォームオブジェクト引数の直後に\ ``org.springframework.validation.BindingResult``\ (以降\ ``BindingResult``\ と呼ぶ) を引数に指定する。

入力チェックの詳細については、 :doc:`../ArchitectureInDetail/WebApplicationDetail/Validation` を参照されたい。

フォームオブジェクトクラスのフィールドに入力チェックで必要となるアノテーションを付加する。

 .. code-block:: java

    public class SampleForm implements Serializable {
        private static final long serialVersionUID = 1477614498217715937L;

        @NotNull
        @Size(min = 10, max = 10)
        private String id;

        @NotNull
        @Size(min = 1, max = 10)
        private String name;

        @Min(1)
        @Max(100)
        private Integer age;

        @Size(min = 1, max = 10)
        private Integer genderCode;

        // omit setters and getters
    }


| フォームオブジェクト引数に\ ``@Validated``\ アノテーションを付与する。
| ``@Validated``\ アノテーションを付けた引数は、ハンドラメソッド実行前に入力チェックが行われ、チェック結果が直後の\ ``BindingResult``\ 引数に格納される。
| フォームオブジェクトにString型以外を指定した場合に発生する型変換エラーも \ ``BindingResult``\ に格納されている。

 .. code-block:: java
    :emphasize-lines: 2,3,5

    @RequestMapping("bindRequestParams")
    public String bindRequestParams(@Validated SampleForm form, // (1)
            BindingResult result, // (2)
            Model model) {
        if (result.hasErrors()) { // (3)
            return "sample/input"; // back to the input view
        }
        // do something
        return "sample/hello"; // returns view name
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - \ ``SampleForm``\ オブジェクトに\ ``@Validated``\ アノテーションを付与し、入力チェック対象のオブジェクトにする。
   * - | (2)
     - 入力チェック結果が格納される\ ``BindingResult``\ を引数に指定する。
   * - | (3)
     - 入力チェックエラーが存在するか判定する。エラーがある場合は、``true`` が返却される。

|

.. _controller_method_argument-redirectattributes-label:

リダイレクト先にデータを渡す
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
ハンドラメソッドを実行した後にリダイレクトする場合に、リダイレクト先で表示するデータを渡したい場合は、\ ``org.springframework.web.servlet.mvc.support.RedirectAttributes``\ (以降\ ``RedirectAttributes``\ と呼ぶ) をハンドラメソッドの引数として受け取り、
``RedirectAttributes``\ オブジェクトに渡したいデータを追加する。

- SampleController.java

 .. code-block:: java
    :emphasize-lines: 2-5,10

    @RequestMapping("hello")
    public String hello(RedirectAttributes redirectAttrs) { // (1)
        redirectAttrs.addFlashAttribute("hello", "Hello World!"); // (2)
        redirectAttrs.addFlashAttribute(new HelloBean("Bean Hello World!")); // (3)
        return "redirect:/sample/hello?complete"; // (4)
    }

    @RequestMapping(value = "hello", params = "complete")
    public String helloComplete() {
        return "sample/complete"; // (5)
    }

- complete.jsp

 .. code-block:: jsp
    :emphasize-lines: 1-2

    Message : ${f:h(hello)}<br> <%-- (6) --%>
    Message : ${f:h(helloBean.message)}<br> <%-- (7) --%>

- HTML of created by View(complete.jsp)

 .. code-block:: html
    :emphasize-lines: 1-2

    Message : Hello World!<br> <!-- (8) -->
    Message : Bean Hello World!<br>　<!-- (8) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90
   :class: longtable

   * - 項番
     - 説明
   * - | (1)
     - \ ``RedirectAttributes``\ オブジェクトを引数として受け取る。
   * - | (2)
     - | \ ``RedirectAttributes``\ オブジェクトの\ ``addFlashAttribute``\ メソッドを呼び出し、渡したいデータを\ ``RedirectAttributes``\ オブジェクトに追加する。
       | 例では、 ``"hello"`` という属性名で ``"HelloWorld!"`` という文字列のデータを追加している。
   * - | (3)
     - | \ ``addFlashAttribute``\ メソッドの第一引数を省略すると値に渡したオブジェクトのクラス名の先頭を小文字にした文字列が属性名になる。
       | 例では、 ``model.addFlashAttribute("helloBean", new HelloBean());`` を行ったのと同じ結果となる。
   * - | (4)
     - | 画面(View)を直接表示せず、次の画面を表示するためのリクエストにリダイレクトする。
   * - | (5)
     - | リダイレクト後のハンドラメソッドでは、(2)(3)で追加したデータを表示する画面のView名を返却する。
   * - | (6)
     - | View(JSP)側では、「${属性名}」と記述することで\ ``RedirectAttributes``\ オブジェクトに追加したデータを取得することができる。
       | 例ではHTMLエスケープを行うEL式の関数を呼び出しているため、「${f:h(属性名)}」としている。
       | HTMLエスケープを行うEL式の関数の詳細については、 :doc:`Cross Site Scripting <../Security/XSS>` を参照されたい。
   * - | (7)
     - | 「${属性名.JavaBeanのプロパティ名}」と記述することで\ ``RedirectAttributes``\に格納されているJavaBeanから値を取得することができる。
   * - | (8)
     - | HTMLの出力例。

 .. raw:: latex

    \newpage

.. warning::
    ``Model`` に追加してもリダイレクト先にデータを渡すことはできない。

.. note::

    \ ``Model``\ の\ ``addAttribute``\ メソッドに非常によく似ているが、データの生存期間が異なる。
    \ ``RedirectAttributes``\ の\ ``addFlashAttribute``\ ではflash scopeというスコープにデータが格納され、
    リダイレクト後の1リクエスト(PRGパターンのG)でのみ追加したデータを参照することができる。2回目以降のリクエストの時にはデータは消えている。

.. figure:: images_ApplicationLayer/applicationFlashscope.png
   :alt: Survival time of flush scope
   :width: 80%
   :align: center

   **Picture - Survival time of flush scope**

|

.. _controller_method_argument-redirectattributes-param-label:

リダイレクト先へリクエストパラメータを渡す
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
リダイレクト先へ動的にリクエストパラメータを設定したい場合は、引数の\ ``RedirectAttributes``\ オブジェクトに渡したい値を追加する。

 .. code-block:: java
    :emphasize-lines: 4

    @RequestMapping("hello")
    public String hello(RedirectAttributes redirectAttrs) {
        String id = "aaaa";
        redirectAttrs.addAttribute("id", id); // (1)
        // must not return "redirect:/sample/hello?complete&id=" + id;
        return "redirect:/sample/hello?complete";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 属性名にリクエストパラメータ名、属性値にリクエストパラメータの値を指定して、\ ``RedirectAttributes``\ オブジェクトの\ ``addAttribute``\ メソッドを呼び出す。
       | 上記例では、 ``"/sample/hello?complete&id=aaaa"`` にリダイレクトされる。

.. warning::
    上記例ではコメント化しているが、``return "redirect:/sample/hello?complete&id=" + id;``\ と結果は同じになる。
    ただし、 ``RedirectAttributes``\ オブジェクトの\ ``addAttribute``\ メソッドを用いるとURIエンコーディングも行われるので、
    動的に埋め込むリクエストパラメータについては、**返り値のリダイレクトURLとして組み立てるのではなく、必ずaddAttributeメソッドを使用してリクエストパラメータに設定すること。**
    動的に埋め込まないリクエストパラメータ(上記例だと"complete")については、返り値のリダイレクトURLに直接指定してよい。

|

.. _controller_method_argument-redirectattributes-path-label:

リダイレクト先URLのパスに値を埋め込む
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
リダイレクト先URLのパスに動的に値を埋め込みたい場合は、リクエストパラメータの設定と同様引数の\ ``RedirectAttributes``\ オブジェクトに埋め込みたい値を追加する。

 .. code-block:: java
    :emphasize-lines: 4,6

    @RequestMapping("hello")
    public String hello(RedirectAttributes redirectAttrs) {
        String id = "aaaa";
        redirectAttrs.addAttribute("id", id); // (1)
        // must not return "redirect:/sample/hello/" + id + "?complete";
        return "redirect:/sample/hello/{id}?complete"; // (2)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 属性名とパスに埋め込みたい値を指定して、\ ``RedirectAttributes``\ オブジェクトの\ ``addAttribute``\ メソッドを呼び出す。
   * - | (2)
     - | リダイレクトURLの埋め込みたい箇所に「{属性名}」のパス変数を指定する。
       | 上記例では、 ``"/sample/hello/aaaa?complete"`` にリダイレクトされる。

.. warning::
    上記例ではコメント化しているが、``"redirect:/sample/hello/" + id + "?complete";``\ と結果は同じになる。
    ただし、 ``RedirectAttributes``\ オブジェクトの\ ``addAttribute``\ メソッドを用いるとURLエンコーディングも行われるので、
    動的に埋め込むパス値については、**返り値のリダイレクトURLとして記述せずに、必ずaddAttributeメソッドを使用し、パス変数を使って埋め込むこと。**

|

.. _controller_method_argument-cookievalue-label:

Cookieから値を取得する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Cookieから取得したい場合は、引数に\ ``@CookieValue``\ アノテーションを付与する。

 .. code-block:: java
    :emphasize-lines: 2

    @RequestMapping("readCookie")
    public String readCookie(@CookieValue("JSESSIONID") String sessionId, Model model) { // (1)
        // do something
        return "sample/readCookie"; // returns view name
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``@CookieValue``\ アノテーションのvalue属性には、Cookie名を指定する。
       | 上記例では、Cookieから"JSESSIONID"というCookie名の値が引数sessionIdに渡る。

.. note::
    ``@RequestParam``\ 同様、required属性、defaultValue属性があり、引数の型にはString型以外の指定も可能である。
    詳細は、 :ref:`controller_method_argument-requestparam-label` を参照されたい。

|

.. _controller_method_argument-cookiewrite-label:

Cookieに値を書き込む
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Cookieに値を書き込む場合は、\ ``HttpServletResponse``\ オブジェクトの\ ``addCookie``\ メソッドを直接呼び出してCookieに追加する。
| Spring MVCからCookieに値を書き込む仕組みが提供されていないため(3.2.3時点)、**この場合に限り HttpServletResponse を引数に取っても良い。**

 .. code-block:: java
    :emphasize-lines: 3,5

    @RequestMapping("writeCookie")
    public String writeCookie(Model model,
            HttpServletResponse response) { // (1)
        Cookie cookie = new Cookie("foo", "hello world!");
        response.addCookie(cookie); // (2)
        // do something
        return "sample/writeCookie";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - Cookieを書き込むために、\ ``HttpServletResponse``\ オブジェクトを引数に指定する。
   * - | (2)
     - | \ ``Cookie``\ オブジェクトを生成し、\ ``HttpServletResponse``\ オブジェクトに追加する。
       | 上記例では、 ``"foo"`` というCookie名で ``"hello world!"`` という値を設定している。

.. tip::

    \ ``HttpServletResponse``\ を引数として受け取ることに変わりはないが、Cookieに値を書き込むためのクラスとして、
    Spring Frameworkから\ ``org.springframework.web.util.CookieGenerator``\ というクラスが提供されている。必要に応じて使用すること。

|

.. _controller_method_argument-pagination-label:

ページネーション情報を取得する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 一覧検索を行うリクエストでは、ページネーション情報が必要となる。
| ``org.springframework.data.domain.Pageable``\ (以降\ ``Pageable``\ と呼ぶ) オブジェクトをハンドラメソッドの引数に取ることで、ページネーション情報(ページ数、取得件数)を容易に扱うことができる。

 詳細については :doc:`../ArchitectureInDetail/WebApplicationDetail/Pagination` を参照すること。

|

.. _controller_method_argument-upload-label:

アップロードファイルを取得する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
アップロードされたファイルを取得する方法は大きく２つある。

- フォームオブジェクトに\ ``MultipartFile``\のプロパティを用意する。
- \ ``@RequestParam``\ アノテーションを付与して\ ``org.springframework.web.multipart.MultipartFile``\ をハンドラメソッドの引数とする。

詳細については :doc:`../ArchitectureInDetail/WebApplicationDetail/FileUpload` を参照されたい。

|

.. _controller_method_argument-message-label:

画面に結果メッセージを表示する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\ ``Model``\ オブジェクト又は\ ``RedirectAttributes``\ オブジェクトをハンドラメソッドの引数として受け取り、
\ ``ResultMessages``\ オブジェクトを追加することで処理の結果メッセージを表示できる。

詳細については :doc:`../ArchitectureInDetail/WebApplicationDetail/MessageManagement` を参照されたい。

|
|

.. _controller_method_return-label:

ハンドラメソッドの返り値について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
`ハンドラメソッドの返り値についても様々な値をとることができる <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-return-types>`_ が、
基本的には次に挙げるもののみを使用すること。

- String(View論理名)

以下に、目的別に返り値の使用方法について説明する。

- :ref:`controller_method_return-html-label`
- :ref:`controller_method_return-download-label`

|

.. _controller_method_return-html-label:

HTMLを応答する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| ハンドラメソッドの実行結果をHTMLとして応答する場合、ハンドラメソッドの返り値は、JSPのView名を返却する。
| JSPを使ってHTMLを生成する場合の\ ``ViewResolver``\ は、基本的には\ ``UrlBasedViewResolver``\ の継承クラス(\ ``InternalViewResolver``\ や \ ``TilesViewResolver``\ 等)となる。

| 以下では、JSP用の\ ``InternalViewResolver``\ を使用する場合の例を記載するが、画面レイアウトがテンプレート化されている場合は\ ``TilesViewResolver``\ を使用することを推奨する。
| \ ``TilesViewResolver``\ の使用方法については、 :doc:`../ArchitectureInDetail/WebApplicationDetail/TilesLayout` を参照されたい。

- spring-mvc.xml

 \ ``<bean>``\ 要素を使用する場合の定義例

 .. code-block:: xml

    <!-- (1) -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/" /> <!-- (2) -->
        <property name="suffix" value=".jsp" /> <!-- (3) -->
        <property name="order" value="1" /> <!-- (4) -->
    </bean>

 Spring Framework 4.1から追加された\ ``<mvc:view-resolvers>``\ 要素を使用する場合の定義例

 .. code-block:: xml

    <mvc:view-resolvers>
        <mvc:jsp prefix="/WEB-INF/views/" /> <!-- (5) -->
    </mvc:view-resolvers>


- SampleController.java

 .. code-block:: java
    :emphasize-lines: 4

    @RequestMapping("hello")
    public String hello() {
        // omitted
        return "sample/hello"; // (6)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - JSP用の\ ``InternalViewResolver``\ を定義する。
   * - | (2)
     - JSPファイルが格納されているベースディレクトリ(ファイルパスのプレフィックス)を指定する。

       プレフィックスを指定しておくことで、ControllerでView名を返却する際に、JSPの物理的な格納場所を意識する必要がなくなる。
   * - | (3)
     - JSPファイルの拡張子(ファイルパスのサフィックス)を指定する。

       サフィックスを指定しておくことで、ControllerでView名を返却する際に、JSPの拡張子を意識する必要がなくなる。
   * - | (4)
     - 複数の\ ``ViewResolver``\ を指定した場合の実行順番を指定する。

       \ ``Integer``\ の範囲で指定することが可能で、値が小さいものから順に実行される。
   * - | (5)
     - Spring Framework 4.1から追加された\ ``<mvc:jsp>``\ 要素に使用して、JSP用の\ ``InternalViewResolver``\ を定義する。

       * \ ``prefix``\ 属性には、JSPファイルが格納されているベースディレクトリ(ファイルパスのプレフィックス)を指定する。
       * \ ``suffix``\ 属性には、デフォルト値として\ ``".jsp"``\が適用されているため、明示的に指定する必要はない。

       .. note::

           \ ``<mvc:view-resolvers>``\ 要素を使用すると、\ ``ViewResolver``\ をシンプルに定義することが出来るため、
           本ガイドラインでは\ ``<mvc:view-resolvers>``\ を使用することを推奨する。

   * - | (6)
     - ハンドラメソッドの返り値として ``"sample/hello"`` というView名を返却した場合、 ``"/WEB-INF/views/sample/hello.jsp"`` が呼び出されてHTMLが応答される。

.. note::
    上記の例ではJSPを使ってHTMLを生成しているが、VelocityやFreeMarkerなど他のテンプレートエンジンを使用してHTMLを生成する場合でも、ハンドラメソッドの返り値は ``"sample/hello"`` のままでよい。
    使用するテンプレートエンジンでの差分は ``ViewResolver`` によって解決される。

|

.. _controller_method_return-download-label:

ダウンロードデータを応答する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| データベースなどに格納されているデータをダウンロードデータ(\ ``"application/octet-stream"``\ 等 )として応答する場合、
| レスポンスデータの生成(ダウンロード処理)を行うViewを作成し、処理を委譲することを推奨する。
| ハンドラメソッドでは、ダウンロード対象となるデータを \ ``Model``\ に追加し、ダウンロード処理を行うViewのView名を返却する。

| View名からViewを解決する方法としては、個別のViewResolverを作成する方法もあるが、ここではSpring Frameworkから提供されている\ ``BeanNameViewResolver``\ を使用する。
| ダウンロード処理の詳細については、 :doc:`../ArchitectureInDetail/WebApplicationDetail/FileDownload` を参照されたい。

- spring-mvc.xml

 \ ``<bean>``\ 要素を使用する場合の定義例

 .. code-block:: xml
    :emphasize-lines: 1-4

    <!-- (1) -->
    <bean class="org.springframework.web.servlet.view.BeanNameViewResolver">
        <property name="order" value="0"/> <!-- (2) -->
    </bean>

    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/" />
        <property name="suffix" value=".jsp" />
        <property name="order" value="1" />
    </bean>

 Spring Framework 4.1から追加された\ ``<mvc:view-resolvers>``\ 要素を使用する場合の定義例

 .. code-block:: xml
    :emphasize-lines: 2

    <mvc:view-resolvers>
        <mvc:bean-name /> <!-- (3) -->
        <mvc:jsp prefix="/WEB-INF/views/" />
    </mvc:view-resolvers>

- SampleController.java

 .. code-block:: java
    :emphasize-lines: 4

    @RequestMapping("report")
    public String report() {
        // omitted
        return "sample/report"; // (4)
    }


- XxxExcelView.java

 .. code-block:: java
    :emphasize-lines: 1-2

    @Component("sample/report") // (5)
    public class XxxExcelView extends AbstractExcelView { // (6)
        @Override
        protected void buildExcelDocument(Map<String, Object> model,
                HSSFWorkbook workbook, HttpServletRequest request,
                HttpServletResponse response) throws Exception {
            HSSFSheet sheet;
            HSSFCell cell;

            sheet = workbook.createSheet("Spring");
            sheet.setDefaultColumnWidth(12);

            // write a text at A1
            cell = getCell(sheet, 0, 0);
            setText(cell, "Spring-Excel test");

            cell = getCell(sheet, 2, 0);
            setText(cell, (Date) model.get("serverTime")).toString());
        }
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - \ ``BeanNameViewResolver``\ を定義する。

       \ ``BeanNameViewResolver``\ は、返却されたView名に一致するBeanをアプリケーションコンテキストから探してViewを解決するクラスとなっている。
   * - | (2)
     - JSP用の\ ``InternalViewResolver``\ や \ ``TilesViewResolver``\ と併用する場合は、これらの\ ``ViewResolver``\ より、高い優先度を指定する事を推奨する。
       上記例では、 ``"0"`` を指定することで、\ ``InternalViewResolver``\ より先に\ ``BeanNameViewResolver``\によるView解決が行われる。
   * - | (3)
     - Spring Framework 4.1から追加された\ ``<mvc:bean-name>``\ 要素を使用して、\ ``BeanNameViewResolver``\ を定義する。

       \ ``<mvc:view-resolvers>``\ 要素を使用して\ ``ViewResolver``\ を定義する場合は、子要素に指定する\ ``ViewResolver``\の定義順が優先順位となる。
       上記例では、JSP用の\ ``InternalViewResolver``\を定義するための要素(\ ``<mvc:jsp>``\)より上に定義することで、JSP用の\ ``InternalViewResolver``\ より先に\ ``BeanNameViewResolver``\によるView解決が行われる。

       .. note::

           \ ``<mvc:view-resolvers>``\ 要素を使用すると、\ ``ViewResolver``\ をシンプルに定義することが出来るため、
           本ガイドラインでは\ ``<mvc:view-resolvers>``\ を使用することを推奨する。
   * - | (4)
     - ハンドラメソッドの返り値として ``"sample/report"`` というView名を返却した場合、 (5)でBean登録されたViewインスタンスによって生成されたデータがダウンロードデータとして応答される。
   * - | (5)
     - コンポーネントの名前にView名を指定して、ViewオブジェクトをBeanとして登録する。

       上記例では、 ``"sample/report"`` というbean名(View名)で ``x.y.z.app.views.XxxExcelView`` のインスタンスがBean登録される。
   * - | (6)
     - Viewの実装例。

       上記例では、 ``org.springframework.web.servlet.view.document.AbstractExcelView`` を継承し、Excelデータを生成するViewクラスの実装となる。

|
|

処理の実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| **Controllerでは、業務処理の実装は行わない** という点がポイントとなる。
| 業務処理の実装はServiceで行い、Controllerでは業務処理が実装されているServiceのメソッドを呼び出す。
| 業務処理の実装の詳細については :doc:`DomainLayer` を参照されたい。

.. note::
    Controllerは、基本的には画面遷移の決定などの処理のルーティングと\ ``Model``\ の設定のみ実装することに徹し、可能な限りシンプルな状態に保つこと。
    この方針で統一することにより、Controllerで実装すべき処理が明確になり、開発規模が大きくなった場合でもControllerのメンテナンス性を保つことができる。

|

Controllerで実装すべき処理を以下に4つ示す。

- :ref:`controller_logic_correlationcheck-label`
- :ref:`controller_logic_businesslogic-label`
- :ref:`controller_logic_domainobject-label`
- :ref:`controller_logic_formobject-label`

|

.. _controller_logic_correlationcheck-label:

入力値の相関チェック
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 入力値に対する相関チェックは、\ ``org.springframework.validation.Validator``\ インタフェースを実装したValidationクラス、もしくは、Bean Validationで検証を行う。
| 相関チェックの実装の詳細については、:doc:`../ArchitectureInDetail/WebApplicationDetail/Validation` を参照されたい。

| 相関チェックの実装自体はControllerのハンドラメソッドで行うことはないが、相関チェックを行う\ ``Validator``\ を\ ``org.springframework.web.bind.WebDataBinder``\ に追加する必要がある。

 .. code-block:: java
    :emphasize-lines: 2,6

    @Inject
    PasswordEqualsValidator passwordEqualsValidator; // (1)

    @InitBinder
    protected void initBinder(WebDataBinder binder){
        binder.addValidators(passwordEqualsValidator); // (2)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - 相関チェックを行う\ ``Validator``\ をInjectする。
   * - | (2)
     - | Injectした\ ``Validator``\ を \ ``WebDataBinder``\ に追加する。
       | \ ``WebDataBinder``\ に追加しておくことで、ハンドラメソッド呼び出し前に行われる入力チェック処理にて、(1)で追加した\ ``Validator``\ が実行され、相関チェックを行うことが出来る。

|

.. _controller_logic_businesslogic-label:

業務処理の呼び出し
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
業務処理が実装されているServiceをInjectし、InjectしたServiceのメソッドを呼び出すことで業務処理を実行する。

 .. code-block:: java
    :emphasize-lines: 2,6

    @Inject
    SampleService sampleService; // (1)

    @RequestMapping("hello")
    public String hello(Model model){
        String message = sampleService.hello(); // (2)
        model.addAttribute("message", message);
        return "sample/hello";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 業務処理が実装されている ``Service`` をInjectする。
   * - | (2)
     - Injectした ``Service`` のメソッドを呼び出し、業務処理を実行する。

|

.. _controller_logic_domainobject-label:

ドメインオブジェクトへの値反映
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 本ガイドラインでは、HTML formから送信されたデータは直接ドメインオブジェクトにバインドするのではなく、フォームオブジェクトにバインドする方法を推奨している。
| そのため、ControllerではServiceのメソッドに渡すドメインオブジェクトにフォームオブジェクトの値を反映する処理を行う必要がある。

 .. code-block:: java
    :emphasize-lines: 4,11-12

    @RequestMapping("hello")
    public String hello(@Validated SampleForm form, BindingResult result, Model model){
        // omitted
        Sample sample = new Sample(); // (1)
        sample.setField1(form.getField1());
        sample.setField2(form.getField2());
        sample.setField3(form.getField3());
        // ...
        // and more ...
        // ...
        String message = sampleService.hello(sample); // (2)
        model.addAttribute("message", message); // (3)
        return "sample/hello";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | Serviceの引数となるドメインオブジェクトを生成し、フォームオブジェクトにバインドされている値を反映する。
   * - | (2)
     - Serviceのメソッドを呼び出し業務処理を実行する。
   * - | (3)
     - 業務処理から返却されたデータを \ ``Model``\ に追加する。

| ドメインオブジェクトへ値を反映する処理は、Controllerのハンドラメソッド内で実装してもよいが、コード量が多くなる場合はハンドラメソッドの可読性を考慮してHelperクラスのメソッドに処理を委譲することを推奨する。
| 以下にHelperメソッドに処理を委譲した場合の例を示す。

- SampleController.java

 .. code-block:: java
    :emphasize-lines: 2,7

    @Inject
    SampleHelper sampleHelper; // (1)

    @RequestMapping("hello")
    public String hello(@Validated SampleForm form, BindingResult result){
        // omitted
        String message = sampleHelper.hello(form); // (2)
        model.addAttribute("message", message);
        return "sample/hello";
    }

- SampleHelper.java

 .. code-block:: java
    :emphasize-lines: 6

    public class SampleHelper {

        @Inject
        SampleService sampleService;

        public String hello(SampleForm form){ // (3)
            Sample sample = new Sample();
            sample.setField1(form.getField1());
            sample.setField2(form.getField2());
            sample.setField3(form.getField3());
            // ...
            // and more ...
            // ...
            String message = sampleService.hello(sample);
            return message;
        }
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - ControllerにHelperクラスのオブジェクトをInjectする。
   * - | (2)
     - InjectしたHelperクラスのメソッドを呼び出すことで、ドメインオブジェクトへの値の反映を行っている。
       Helperクラスに処理を委譲することで、Controllerの実装をシンプルな状態に保つことができる。
   * - | (3)
     - ドメインオブジェクトを生成した後に、Serviceクラスのメソッド呼び出し業務処理を実行している。

 .. note::
    Helperクラスに処理を委譲する以外の方法として、Bean変換機能を使用する方法がある。
    Bean変換機能の詳細は、:doc:`../ArchitectureInDetail/GeneralFuncDetail/Dozer` を参照されたい。

|

.. _controller_logic_formobject-label:

フォームオブジェクトへの値反映
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 本ガイドラインでは、HTML formの項目にバインドするデータはドメインオブジェクトではなく、フォームオブジェクトを使用する方法を推奨している。
| そのため、ControllerではServiceのメソッドから返却されたドメインオブジェクトの値をフォームオブジェクトに反映する処理を行う必要がある。


 .. code-block:: java
    :emphasize-lines: 4,5,11

    @RequestMapping("hello")
    public String hello(SampleForm form, BindingResult result, Model model){
        // omitted
        Sample sample = sampleService.getSample(form.getId()); // (1)
        form.setField1(sample.getField1()); // (2)
        form.setField2(sample.getField2());
        form.setField3(sample.getField3());
        // ...
        // and more ...
        // ...
        model.addAttribute(sample); // (3)
        return "sample/hello";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - 業務処理が実装されているServiceのメソッドを呼び出し、ドメインオブジェクトを取得する。
   * - | (2)
     - 取得したドメインオブジェクトの値をフォームオブジェクトに反映する。
   * - | (3)
     - 表示のみ行う項目がある場合は、データを参照できるようにするために、\ ``Model``\ にドメインオブジェクトを追加する。

 .. note::
    画面に表示のみ行う項目については、フォームオブジェクトに項目をもつのではなく、Entityなどのドメインオブジェクトから直接値を参照することを推奨する。

フォームオブジェクトへの値反映処理は、Controllerのハンドラメソッド内で実装してもよいが、
コード量が多くなる場合はハンドラメソッドの可読性を考慮してHelperクラスのメソッドに委譲することを推奨する。

- SampleController.java

 .. code-block:: java
    :emphasize-lines: 5

    @RequestMapping("hello")
    public String hello(@Validated SampleForm form, BindingResult result){
        // omitted
        Sample sample = sampleService.getSample(form.getId());
        sampleHelper.applyToForm(sample, form); // (1)
        model.addAttribute(sample);
        return "sample/hello";
    }

- SampleHelper.java

 .. code-block:: java
    :emphasize-lines: 2

    public void applyToForm(SampleForm destForm, Sample srcSample){
        destForm.setField1(srcSample.getField1()); // (2)
        destForm.setField2(srcSample.getField2());
        destForm.setField3(srcSample.getField3());
        // ...
        // and more ...
        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - ドメインオブジェクトの値をフォームオブジェクトに反映するためのメソッドを呼び出す。
   * - | (2)
     - ドメインオブジェクトの値をフォームオブジェクトに反映するためのメソッドにて、ドメインオブジェクトの値をフォームオブジェクトに反映する。

 .. note::
    Helperクラスに処理を委譲する以外の方法として、Bean変換機能を使用する方法がある。
    Bean変換機能の詳細は、:doc:`../ArchitectureInDetail/GeneralFuncDetail/Dozer` を参照されたい。

|
|

.. _formobject:

フォームオブジェクトの実装
--------------------------------------------------------------------------------
フォームオブジェクトはHTML上のformを表現するオブジェクト(JavaBean)であり、以下の役割を担う。

#. **データベース等で保持している業務データを保持し、HTML(JSP) formから参照できるようにする。**
#. **HTML formから送信されたリクエストパラメータを保持し、ハンドラメソッドで参照できるようにする。**

.. figure:: ./images_ApplicationLayer/applicationFormobject.png
   :width: 80%
   :align: center

|

フォームオブジェクトの実装について、以下4点に着目して説明する。

- :ref:`formobject_new-label`
- :ref:`formobject_init-label`
- :ref:`formobject_bindhtmlform-label`
- :ref:`formobject_bindrequestparam-label`

|

.. _formobject_new-label:

フォームオブジェクトの作成方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
フォームオブジェクトはJavaBeanとして作成する。
Spring Frameworkでは、HTML formから送信されたリクエストパラメータ(文字列)を、フォームオブジェクトに定義されている型に変換してからバインドする機能を提供しているため、
フォームオブジェクトに定義するフィールドの型は、\ ``java.lang.String``\ だけではなく、任意の型で定義することができる。

 .. code-block:: java

    public class SampleForm implements Serializable {
        private String id;
        private String name;
        private Integer age;
        private String genderCode;
        private Date birthDate;
        // ommitted getter/setter
    }

 .. tip:: **Spring Frameworkから提供されている型変換を行う仕組みについて**

    Spring Frameworkは、以下の3つの仕組みを使って型変換を行っており、基本的な型への変換は標準でサポートされている。各変換機能の詳細については、リンク先のページを参照されたい。

    * `Spring Type Conversion <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/validation.html#core-convert>`_\
    * `Spring Field Formatting <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/validation.html#format>`_\
    * `java.beans.PropertyEditor implementations <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/validation.html#beans-beans-conversion>`_\

 .. warning::

    フォームオブジェクトには画面に表示のみ行う項目は保持せず、HTML formの項目のみ保持することを推奨する。
    フォームオブジェクトに画面表示のみ行う項目の値を設定した場合、フォームオブジェクトをHTTPセッションオブジェクトに格納する際にメモリを多く消費する事になり、メモリ枯渇の原因になる可能性がある。
    画面表示のみの項目は、Entityなどのドメイン層のオブジェクトをリクエストスコープに追加(\ ``Model.addAttribute``\ )することでHTML(JSP)にデータを渡すことを推奨する。

|

フィールド単位の数値型変換
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\ ``@NumberFormat``\ アノテーションを使用することでフィールド毎に数値の形式を指定することが出来る。

 .. code-block:: java
    :emphasize-lines: 2

    public class SampleForm implements Serializable {
        @NumberFormat(pattern = "#,#") // (1)
        private Integer price;
        // ommitted getter/setter
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - HTML formから送信されるリクエストパラメータの数値形式を指定する。例では、patternとして ``"#,#"`` 形式を指定しているので、「,」でフォーマットされた値をバインドすることができる。
       リクエストパラメータの値が ``"1,050"`` の場合、フォームオブジェクトのpriceには ``"1050"`` のIntegerオブジェクトがバインドされる。

``@NumberFormat``\ アノテーションで指定できる属性は以下の通り。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 10 80

   * - 項番
     - 属性名
     - 説明
   * - 1.
     - style
     - 数値のスタイルを指定する。詳細は、`NumberFormat.StyleのJavadoc <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/format/annotation/NumberFormat.Style.html>`_\ を参照されたい。
   * - 2.
     - pattern
     - Javaの数値形式を指定する。詳細は、`DecimalFormatのJavadoc <http://docs.oracle.com/javase/8/docs/api/java/text/DecimalFormat.html>`_\ を参照されたい。

|

.. _ApplicationLayer-DateTimeFormat:

フィールド単位の日時型変換
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\ ``@DateTimeFormat``\ アノテーションを使用することでフィールド毎に日時の形式を指定することが出来る。

 .. code-block:: java
    :emphasize-lines: 2

    public class SampleForm implements Serializable {
        @DateTimeFormat(pattern = "yyyyMMdd") // (1)
        private Date birthDate;
        // ommitted getter/setter
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - HTML formから送信されるリクエストパラメータの日時形式を指定する。例では、patternとして ``"yyyyMMdd"`` 形式を指定している。
       リクエストパラメータの値が ``"20131001"`` の場合、フォームオブジェクトのbirthDateには 2013年10月1日のDateオブジェクトがバインドされる。

\ ``@DateTimeFormat``\ アノテーションで指定できる属性は以下の通り。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 10 80

   * - 項番
     - 属性名
     - 説明
   * - 1.
     - iso
     - ISOの日時形式を指定する。詳細は、`DateTimeFormat.ISOのJavadoc <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/format/annotation/DateTimeFormat.ISO.html>`_\ を参照。
   * - 2.
     - pattern
     - Javaの日時形式を指定する。詳細は、`SimpleDateFormatのJavadoc <http://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html>`_\ を参照されたい。
   * - 3.
     - style
     - | 日付と時刻のスタイルを2桁の文字列として指定する。
       | 1桁目が日付のスタイル、2桁目が時刻のスタイルとなる。
       | スタイルとして指定できる値は以下の値となる。
       |
       | S : \ ``java.text.DateFormat.SHORT``\ と同じ形式となる。
       | M : \ ``java.text.DateFormat.MEDIUM``\ と同じ形式となる。
       | L : \ ``java.text.DateFormat.LONG``\ と同じ形式となる。
       | F : \ ``java.text.DateFormat.FULL``\ と同じ形式となる。
       | - : 省略を意味するスタイル。
       |
       | 指定例及び変換例)
       | MM : Dec 9, 2013 3:37:47 AM
       | M- : Dec 9, 2013
       | -M : 3:41:45 AM

.. warning::
    \ ``@DateTimeFormat`` \ の pattern でフォーマットを指定し、プロパティとして  JSR-310 Date and Time APIが提供する\ ``java.time.LocalDate`` \を使用した場合、STRICTにチェックがされない
    (\ ``"20150229"`` \を変換した場合、本来は型ミスマッチエラーとなるはずが、\ ``2015年2月28日`` \ がバインドされる)。
    Spring Framework  4.3で仕様が改善されて発生しなくなるが、TERASOLUNA Server Framework for Java (5.x)では Spring Framework 4.2 を使用しているので影響を受ける。
    本事象の詳細は「`@DateTimeFormat's JSR-310 formatter is not strict in case of pattern <https://jira.spring.io/browse/SPR-13567>`_\」を参照されたい。

|

Controller単位の型変換
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\ ``@InitBinder``\ アノテーションを使用することでController毎に型変換の定義を指定する事も出来る。

 .. code-block:: java
    :emphasize-lines: 1,5

    @InitBinder // (1)
    public void initWebDataBinder(WebDataBinder binder) {
        binder.registerCustomEditor(
                Long.class,
                new CustomNumberEditor(Long.class, new DecimalFormat("#,#"), true)); // (2)
    }

 .. code-block:: java
    :emphasize-lines: 1

    @InitBinder("sampleForm") // (3)
    public void initSampleFormWebDataBinder(WebDataBinder binder) {
        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - \ ``@InitBinder``\ アノテーション を付与したメソッド用意すると、バインド処理が行われる前にこのメソッドが呼び出され、デフォルトの動作をカスタマイズすることができる。
   * - | (2)
     - 例では、Long型のフィールドの数値形式を ``"#,#"`` に指定しているので、「,」でフォーマットされた値をバインドすることができる。
   * - | (3)
     - \ ``@InitBinder``\ アノテーションのvalue属性にフォームオブジェクトの属性名を指定することで、フォームオブジェクト毎にデフォルトの動作をカスタマイズすることもできる。
       例では、 ``"sampleForm"`` という属性名のフォームオブジェクトに対するバインド処理が行われる前にメソッドが呼び出される。

|

入力チェック用のアノテーションの指定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
フォームオブジェクトのバリデーションは、Bean Validationを使用して行うため、フィールドの制約条件を示すアノテーションを指定する必要がある。
入力チェックの詳細は、:doc:`../ArchitectureInDetail/WebApplicationDetail/Validation` を参照されたい。

|

.. _formobject_init-label:

フォームオブジェクトの初期化方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
HTMLのformにバインドするフォームオブジェクトの事をform-backing beanと呼び、\ ``@ModelAttribute``\ アノテーションを使うことで結びつけることができる。
form-backing beanの初期化は、\ ``@ModelAttribute``\ アノテーションを付与したメソッドで行う。
このようなメソッドのことを本ガイドラインではModelAttributeメソッドと呼び、\ ``setUpXxxForm``\ というメソッド名で定義することを推奨する。

 .. code-block:: java
    :emphasize-lines: 1

    @ModelAttribute // (1)
    public SampleForm setUpSampleForm() {
        SampleForm form = new SampleForm();
        // populate form
        return form;
    }

 .. code-block:: java
    :emphasize-lines: 1

    @ModelAttribute("xxx") // (2)
    public SampleForm setUpSampleForm() {
        SampleForm form = new SampleForm();
        // populate form
        return form;
    }

 .. code-block:: java
    :emphasize-lines: 3

    @ModelAttribute
    public SampleForm setUpSampleForm(
            @CookieValue(value = "name", required = false) String name, // (3)
            @CookieValue(value = "age", required = false) Integer age,
            @CookieValue(value = "birthDate", required = false) Date birthDate) {
        SampleForm form = new SampleForm();
        form.setName(name);
        form.setAge(age);
        form.setBirthDate(birthDate);
        return form;
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - \ ``Model``\ に追加するための属性名は、クラス名の先頭を小文字にした値（デフォルト値）が設定される。この例では ``"sampleForm"`` が属性名になる。
       返却したオブジェクトは、\ ``model.addAttribute(form)``\ 相当の処理が実行され\ ``Model``\ に追加される。
   * - | (2)
     -  ``Model``\ に追加するための属性名を指定したい場合は、\ ``@ModelAttribute``\ アノテーションのvalue属性に指定する。この例では ``"xxx"`` が属性名になる。
        返却したオブジェクトは、``model.addAttribute("xxx", form)``\ 相当の処理が実行され\ ``Model``\ に追加される。
        デフォルト値以外の属性名を指定した場合、ハンドラメソッドの引数としてフォームオブジェクトを受け取る時に\ ``@ModelAttribute("xxx")``\ の指定が必要になる。
   * - | (3)
     -  ModelAttributeメソッドは、ハンドラメソッドと同様に初期化に必要なパラメータを渡すこともできる。例では、\ ``@CookieValue``\ アノテーションを使用してCookieの値をフォームオブジェクトに設定している。

.. note::
    フォームオブジェクトにデフォルト値を設定したい場合はModelAttributeメソッドで値を設定すること。
    例の(3)ではCookieから値を取得しているが、定数クラスなどに定義されている固定値を直接設定してもよい。

.. note::
    ModelAttributeメソッドはController内に複数定義することができる。各メソッドはControllerのハンドラメソッドが呼び出される前に毎回実行される。

.. warning::
    ModelAttributeメソッドはリクエスト毎にメソッドが実行されるため、特定のリクエストの時のみに必要なオブジェクトをModelAttributeメソッドを使って生成すると、無駄なオブジェクトの生成及び初期化処理が行われる点に注意すること。
    特定のリクエストのみで必要なオブジェクトについては、ハンドラメソッド内で生成し\ ``Model``\ に追加する方法にすること。

|

.. _formobjectjsp:

.. _formobject_bindhtmlform-label:

HTML formへのバインディング方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| \ ``Model``\ に追加されたフォームオブジェクトは\ ``<form:xxx>``\ タグを用いて、HTML(JSP)のformにバインドすることができる。
| \ ``<form:xxx>``\ タグの詳細は、 `Using Spring's form tag library <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/view.html#view-jsp-formtaglib>`_\ を参照されたい。

 .. code-block:: jsp
    :emphasize-lines: 1

    <%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %> <!-- (1) -->

 .. code-block:: jsp
    :emphasize-lines: 2,3

    <form:form modelAttribute="sampleForm"
               action="${pageContext.request.contextPath}/sample/hello"> <!-- (2) -->
        Id         : <form:input path="id" /><form:errors path="id" /><br /> <!-- (3) -->
        Name       : <form:input path="name" /><form:errors path="name" /><br />
        Age        : <form:input path="age" /><form:errors path="age" /><br />
        Gender     : <form:input path="genderCode" /><form:errors path="genderCode" /><br />
        Birth Date : <form:input path="birthDate" /><form:errors path="birthDate" /><br />
    </form:form>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - \ ``<form:form>``\ タグを使用するためのtaglibの定義を行う。
   * - | (2)
     - \ ``<form:form>``\ タグのmodelAttribute属性には、\ ``Model``\ に格納されているフォームオブジェクトの属性名を指定する。
   * - | (3)
     - \ ``<form:input>``\ タグのpath属性には、フォームオブジェクトのプロパティ名を指定する。

|

.. _formobject_bindrequestparam-label:

リクエストパラメータのバインディング方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
HTML formから送信されたリクエストパラメータは、フォームオブジェクトにバインドし、Controllerのハンドラメソッドの引数に渡すことができる。

 .. code-block:: java
    :emphasize-lines: 3

    @RequestMapping("hello")
    public String hello(
            @Validated SampleForm form, // (1)
            BindingResult result,
            Model model) {
        if (result.hasErrors()) {
            return "sample/input";
        }
        // process form...
        return "sample/hello";
    }

 .. code-block:: java
    :emphasize-lines: 10

    @ModelAttribute("xxx")
    public SampleForm setUpSampleForm() {
        SampleForm form = new SampleForm();
        // populate form
        return form;
    }

    @RequestMapping("hello")
    public String hello(
            @ModelAttribute("xxx") @Validated SampleForm form, // (2)
            BindingResult result,
            Model model) {
        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - フォームオブジェクトにリクエストパラメータが反映された状態で、Controllerのハンドラメソッドの引数に渡される。
   * - | (2)
     - ModelAttributeメソッドにて属性名を指定した場合、\ ``@ModelAttribute("xxx")``\ といった感じで、フォームオブジェクトの属性名を明示的に指定する必要がある。

.. warning::

    ModelAttributeメソッドで指定した属性名とメソッドの引数で指定した属性名が異なる場合、ModelAttributeメソッドで生成したインスタンスとは別のインスタンスが生成されるので注意が必要。
    ハンドラメソッドで属性名の指定を省略した場合、クラス名の先頭を小文字にした値が属性名として扱われる。

|

バインディング結果の判定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
HTML formから送信されたリクエストパラメータをフォームオブジェクトにバインドする際に発生したエラー（入力チェックエラーも含む）は、 \ ``org.springframework.validation.BindingResult``\ に格納される。

 .. code-block:: java
    :emphasize-lines: 4,6

    @RequestMapping("hello")
    public String hello(
            @Validated SampleForm form,
            BindingResult result, // (1)
            Model model) {
        if (result.hasErrors()) { // (2)
            return "sample/input";
        }
        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - フォームオブジェクトの直後に\ ``BindingResult``\ を宣言すると、フォームオブジェクトへのバインド時のエラーと入力チェックエラーを参照することができる。
   * - | (2)
     - \ ``BindingResult.hasErrors()``\ を呼び出すことで、フォームオブジェクトの入力値のエラー有無を判定することができる。

フィールドエラーの有無、グローバルエラー(相関チェックエラーなどのクラスレベルのエラー)の有無を個別に判定することもできるので、要件に応じて使い分けること。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.40\linewidth}|p{0.50\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 40 50

   * - 項番
     - メソッド
     - 説明
   * - 1.
     - ``hasGlobalErrors()``
     - グローバルエラーの有無を判定するメソッド
   * - 2.
     - ``hasFieldErrors()``
     - フィールドエラーの有無を判定するメソッド
   * - 3.
     - ``hasFieldErrors(String field)``
     - 指定したフィールドのエラー有無を判定するメソッド

|

.. _view:

Viewの実装
--------------------------------------------------------------------------------
Viewは以下の役割を担う。

#. | **クライアントに応答するレスポンスデータ(HTML)を生成する。**
   | Viewはモデル（フォームオブジェクトやドメインオブジェクトなど）から必要なデータを取得し、クライアントが描画するために必要な形式でレスポンスデータを生成する。

|

.. _ApplicationLayerImplementOfJsp:

JSPの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| クライアントにHTMLを応答する場合は、JSPを使用してViewを実装する。
| JSPを呼び出すための ``ViewResolver`` は、Spring Frameworkより提供されているので、提供されているクラスを利用する。``ViewResolver`` の設定方法は、 :ref:`controller_method_return-html-label` を参照されたい。

以下に、基本的なJSPの実装方法について説明する。

- :ref:`view_jsp_include-label`
- :ref:`view_jsp_out-label`
- :ref:`view_jsp_outnumber-label`
- :ref:`view_jsp_outdate-label`
- :ref:`view_jsp_requesturl-label`
- :ref:`view_jsp_form-label`
- :ref:`view_jsp_errors-label`
- :ref:`view_jsp_resultmessages-label`
- :ref:`view_jsp_codelist-label`
- :ref:`view_jsp_message-label`
- :ref:`view_jsp_if-label`
- :ref:`view_jsp_forEach-label`
- :ref:`view_jsp_pagination-label`
- :ref:`view_jsp_authorization-label`

本章では代表的なJSPタグライブラリの使い方は説明しているが、全てのJSPタグライブラリの説明はしていないので、詳細な使い方については、それぞれのドキュメントを参照すること。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 30 60

   * - 項番
     - JSPタグライブラリ名
     - ドキュメント
   * - 1.
     - Spring's form tag library
     - - `<http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/view.html#view-jsp-formtaglib>`_\
       - `<http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/spring-form-tld.html>`_\
   * - 2.
     - Spring's tag library
     - - `<http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/spring-tld.html>`_\
   * - 3.
     - JSTL
     - - `<http://download.oracle.com/otndocs/jcp/jstl-1.2-mrel2-eval-oth-JSpec/>`_\
   * - 4.
     - Common library's tags & el functions
     - - 本ガイドラインの「:doc:`../ArchitectureInDetail/WebApplicationDetail/TagLibAndELFunctions`」

 .. warning::

    terasoluna-gfw-web 1.0.0.RELEASEを使用している場合は、Spring's form tag libraryから提供されている\ ``<form:form>``\タグを使う際は、必ず\ ``action``\属性を指定すること。

    terasoluna-gfw-web 1.0.0.RELEASEが依存しているSpring MVC(3.2.4.RELEASE)では、\ ``<form:form>``\タグの\ ``action``\属性を省略した場合、XSS(Cross-site scripting)の脆弱性が存在する。
    脆弱性に関する情報については、\ `National Vulnerability Database (NVD)のCVE-2014-1904 <http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-1904>`_\を参照されたい。

    尚、terasoluna-gfw-web 1.0.1.RELEASE以上では、XSS対策が行われているSpring MVC(3.2.10.RELEASE以上)に依存しているため、本脆弱性は存在しない。



|

.. _view_jsp_include-label:

インクルード用の共通JSPの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
全てのJSPで必要となるディレクティブの宣言などを行うためのJSPを作成する。
このJSPを ``web.xml`` の ``<jsp-config>/<jsp-property-group>/<include-prelude>`` 要素に指定することで、個々のJSPで宣言する必要がなくなる。
なお、このファイルはブランクプロジェクトで提供している。

- include.jsp

 .. code-block:: jsp
    :emphasize-lines: 1,4,8

    <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%> <%-- (1) --%>
    <%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt"%>

    <%@ taglib uri="http://www.springframework.org/tags" prefix="spring"%> <%-- (2) --%>
    <%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>
    <%@ taglib uri="http://www.springframework.org/security/tags" prefix="sec"%>

    <%@ taglib uri="http://terasoluna.org/functions" prefix="f"%> <%-- (3) --%>
    <%@ taglib uri="http://terasoluna.org/tags" prefix="t"%>

- web.xml

 .. code-block:: xml
    :emphasize-lines: 7

    <jsp-config>
        <jsp-property-group>
            <url-pattern>*.jsp</url-pattern>
            <el-ignored>false</el-ignored>
            <page-encoding>UTF-8</page-encoding>
            <scripting-invalid>false</scripting-invalid>
            <include-prelude>/WEB-INF/views/common/include.jsp</include-prelude> <!-- (4) -->
        </jsp-property-group>
    </jsp-config>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - JSTLのJSPタグライブラリを宣言している。 例では、 ``core`` と ``fmt`` を利用している。
   * - | (2)
     - Spring FrameworkのJSPタグライブラリを宣言している。 例では、 ``spring`` と ``form`` と ``sec`` を利用している。
   * - | (3)
     - 共通ライブラリから提供しているJSPタグライブラリを宣言している。
   * - | (4)
     - インクルード用のJSP(\ ``/WEB-INF/views/common/include.jsp``\ )に指定した内容が、各JSP(\ ``<url-pattern>``\ で指定されているファイル)の先頭にインクルードされる。

 .. note::

   ディレクティブの詳細は、 `JavaServer Pages Specification(Version2.2) <http://download.oracle.com/otndocs/jcp/jsp-2.2-mrel-eval-oth-JSpec/>`_\ の "JSP.1.10 Directives" を参照されたい。

 .. note::

   <jsp-property-group>要素の詳細は、 `JavaServer Pages Specification(Version2.2) <http://download.oracle.com/otndocs/jcp/jsp-2.2-mrel-eval-oth-JSpec/>`_\ の "JSP.3.3 JSP Property Groups" を参照されたい。

|

.. _view_jsp_out-label:

モデルに格納されている値を表示する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
モデル（フォームオブジェクトやドメインオブジェクトなど）に格納されている値をHTMLに表示する場合、EL式又はJSTLから提供されているJSPタグライブラリを使用する。

EL式を使用して表示する。

- SampleController.java

 .. code-block:: java
    :emphasize-lines: 3

    @RequestMapping("hello")
    public String hello(Model model) {
        model.addAttribute(new HelloBean("Bean Hello World!")); // (1)
        return "sample/hello"; // returns view name
    }

- hello.jsp

 .. code-block:: jsp
    :emphasize-lines: 1

    Message : ${f:h(helloBean.message)} <%-- (2) --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``Model``\ オブジェクトに \ ``HelloBean``\ オブジェクトを追加する。
   * - | (2)
     - | View(JSP)側では、「${属性名.JavaBeanのプロパティ名}」と記述することで\ ``Model``\ オブジェクトに追加したデータを取得することができる。
       | 例ではHTMLエスケープを行うEL式の関数を呼び出しているため、「${f:h(属性名.JavaBeanのプロパティ名)}」としている。

 .. note::
    共通部品よりEL式用のHTMLエスケープ関数( ``f:h`` )を提供しているので、EL式を使用してHTMLに値を出力する場合は、必ず使用すること。
    HTMLエスケープを行うEL式の関数の詳細については、 :doc:`Cross Site Scripting <../Security/XSS>` を参照されたい。

JSTLのJSPタグライブラリから提供されている ``<c:out>`` タグを使用して表示する。

 .. code-block:: jsp
    :emphasize-lines: 1

    Message : <c:out value="${helloBean.message}" /> <%-- (1) --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | EL式で取得した値を ``<c:out>`` タグのvalue属性に指定する。HTMLエスケープも行われる。

 .. note::
    ``<c:out>`` の詳細は、`JavaServer Pages Standard Tag Library(Version 1.2) <http://download.oracle.com/otndocs/jcp/jstl-1.2-mrel2-eval-oth-JSpec/>`_\ の "CHAPTER 4 General-Purpose Actions" を参照されたい。

|

.. _view_jsp_outnumber-label:

モデルに格納されている数値を表示する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
数値型の値をフォーマットして出力する場合、JSTLから提供されているJSPタグライブラリを使用する。

| JSTLのJSPタグライブラリから提供されている ``<fmt:formatNumber>`` タグを使用して表示する。

 .. code-block:: jsp
    :emphasize-lines: 1

    Number Item : <fmt:formatNumber value="${helloBean.numberItem}" pattern="0.00" /> <%-- (1) --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | EL式で取得した値を ``<fmt:formatNumber>`` タグのvalue属性に指定する。表示するフォーマットはpattern属性に指定する。例では、"``0.00``" を指定している。
       | 仮に ``${helloBean.numberItem}`` で取得した値が ``"1.2"`` の場合、画面には ``"1.20"`` が出力される。

.. note::
    ``<fmt:formatNumber>`` の詳細は、`JavaServer Pages Standard Tag Library(Version 1.2) <http://download.oracle.com/otndocs/jcp/jstl-1.2-mrel2-eval-oth-JSpec/>`_\ の "CHAPTER 9 Formatting Actions" を参照されたい。

|

.. _view_jsp_outdate-label:

モデルに格納されている日時を表示する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
日時型の値をフォーマットして出力する場合、JSTLから提供されているJSPタグライブラリを使用する。

JSTLのJSPタグライブラリから提供されている ``<fmt:formatDate>`` タグを使用して表示する。

 .. code-block:: jsp
    :emphasize-lines: 1

    Date Item : <fmt:formatDate value="${helloBean.dateItem}" pattern="yyyy-MM-dd" /> <%-- (1) --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | EL式で取得した値を ``<fmt:formatDate>`` タグのvalue属性に指定する。表示するフォーマットはpattern属性に指定する。例では、"``yyyy-MM-dd``" を指定している。
       | 仮に ``${helloBean.dateItem}`` で取得した値が2013年3月2日の場合、画面には ``"2013-03-02"`` が出力される。

.. note::
    ``<fmt:formatDate>`` の詳細は、`JavaServer Pages Standard Tag Library(Version 1.2) <http://download.oracle.com/otndocs/jcp/jstl-1.2-mrel2-eval-oth-JSpec/>`_\ の "CHAPTER 9 Formatting Actions" を参照されたい。

.. note::
    日時オブジェクトの型として、Joda Timeから提供されている ``org.joda.time.DateTime`` などを利用する場合は、Jada Timeから提供されているJSPタグライブラリを使用すること。
    Joda Timeの詳細は、 :doc:`../ArchitectureInDetail/GeneralFuncDetail/JodaTime` を参照されたい。


|

.. _view_jsp_requesturl-label:

リクエストURLを生成する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

HTMLの\ ``<form>``\ 要素(JSPタグライブラリの\ ``<form:form>``\ 要素)の\ ``action``\ 属性や\ ``<a>``\ 要素の\ ``href``\ 属性などに対してリクエストURL(Controllerのメソッドを呼び出すためのURL)を設定する場合は、
以下のいずれかの方法を使用してURLを生成する。

* 文字列としてリクエストURLを組み立てる
* Spring Framework 4.1から追加されたEL関数を使用してリクエストURLを組み立てる

.. note::

    どちらの方法を使用してもよいが、一つのアプリケーションの中で混在して使用することは、
    保守性を低下させる可能性があるので避けた方がよい。

|

| 以降の説明で使用するControllerのメソッドの実装サンプルを示す。
| 以降の説明では、以下に示すメソッドを呼び出すためのリクエストURLを生成するための実装方法について説明する。

 .. code-block:: java

    package com.example.app.hello;

    import org.springframework.stereotype.Controller;
    import org.springframework.web.bind.annotation.RequestMapping;

    @RequestMapping("hello")
    @Controller
    public class HelloController {

        // (1)
        @RequestMapping({"", "/"})
        public String hello() {
            return "hello/home";
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - このメソッドに割り当てられるリクエストURLは、\ "``{コンテキストパス}/hello"``\ となる。

|

**文字列としてリクエストURLを組み立てる**

まず、文字列としてリクエストURLを組み立てる方法について説明する。

 .. code-block:: jsp

    <form action="${pageContext.request.contextPath}/hello"> <!-- (2) -->
        <!-- ... -->
    </form>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (2)
      - \ ``pageContext``\ (JSPの暗黙オブジェクト)からWebアプリケーションに割り振られているコンテキスパスを取得し(\ ``${pageContext.request.contextPath}``\ )、
        コンテキストパスの後ろに呼び出すControllerのメソッドに割り振られているサーブレットパス(上記例では、\ ``/hello``\)を加える。

 .. tip::

    URLを組み立てるJSPタグライブラリとして、

    * JSTLから提供されている \ ``<c:url>``\
    * Spring Frameworkから提供されている \ ``<spring:url>``\

    が存在する。これらのJSPタグライブラリを使用して、リクエストURLを組み立ててもよい。

    リクエストURLを動的に組み立てる必要がある場合は、
    これらのJSPタグライブラリを使用してURLを組み立てた方がよいケースがある。

|

**Spring Framework 4.1から追加されたEL関数を使用してリクエストURLを組み立てる**

つぎに、Spring Framework 4.1から追加されたEL関数(\ ``spring:mvcUrl``\ )を使用してリクエストURLを組み立てる方法について説明する。

\ ``spring:mvcUrl``\ 関数を使用すると、Controllerのメソッドのメタ情報(メソッドシグネチャやアノテーションなど)と連携して、
リクエストURLを組み立てる事ができる。

 .. code-block:: jsp

    <form action="${spring:mvcUrl('HC#hello').build()}"> <!-- (3) -->
        <!-- ... -->
    </form>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (3)
      - \ ``spring:mvcUrl``\ 関数の引数には、呼び出すControllerのメソッドに割り振られているリクエストマッピング名を指定する。

        \ ``spring:mvcUrl``\ 関数からは、リクエストURLを組み立てるクラス(\ ``MvcUriComponentsBuilder.MethodArgumentBuilder``\ )のオブジェクトが返却される。
        \ ``MvcUriComponentsBuilder.MethodArgumentBuilder``\ クラスには、

        * \ ``arg``\ メソッド
        * \ ``build``\ メソッド
        * \ ``buildAndExpand``\ メソッド

        が用意されており、それぞれ、以下の役割を持つ。

        * \ ``arg``\ メソッドは、Controllerのメソッドの引数に渡す値を指定するためのメソッドである。
        * \ ``build``\ メソッドは、リクエストURLを生成するためのメソッドである。
        * \ ``buildAndExpand``\ メソッドは、Controllerのメソッドの引数として宣言されていない動的な部分(パス変数など)に埋め込む値を指定した上で、リクエストURLを生成するためのメソッドである。

        上記例では、リクエストURLが静的なURLであるため、\ ``build``\ メソッドのみを呼び出してリクエストURLを生成している。
        リクエストURLが動的なURL(パス変数やクエリ文字列が存在するURL)の場合は、
        \ ``arg``\ メソッドや\ ``buildAndExpand``\ メソッドを呼び出す必要がある。

        \ ``arg``\ メソッドと\ ``buildAndExpand``\ メソッドの具体的な使用例については、
        「\ `Spring Framework Reference Documentation(Building URIs to Controllers and methods from views) <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/mvc.html#mvc-links-to-controllers-from-views>`_\ 」を参照されたい。


 .. note:: **リクエストマッピング名について**

    リクエストマッピング名は、デフォルト実装(\ ``org.springframework.web.servlet.mvc.method.RequestMappingInfoHandlerMethodMappingNamingStrategy``\ の実装)では、
    「クラス名の大文字部分(クラスの短縮名) + \ ``"#"``\  + メソッド名」となる。

    リクエストマッピング名は重複しないようにする必要がある。
    名前が重複してしまった場合は、\ ``@RequestMapping``\ アノテーションの\ ``name``\ 属性に一意となる名前を指定する必要がある。

    Controllerのメソッドに割り当てられたリクエストマッピング名を確認したい場合は、
    \ :file:`logback.xml`\ に以下の設定を追加すればよい。

     .. code-block:: xml

        <logger name="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping">
            <level value="trace" />
        </logger>

    上記設定を行った後に再起動すると、以下のようなログが出力されるようになる。

     .. code-block:: text

        date:2014-12-09 18:34:29	thread:RMI TCP Connection(2)-127.0.0.1	X-Track:	level:TRACE	logger:o.s.w.s.m.m.a.RequestMappingHandlerMapping      	message:Mapping name=HC#hello

|

.. _view_jsp_form-label:

HTML formへフォームオブジェクトをバインドする
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
HTML formへフォームオブジェクトをバインドし、フォームオブジェクトで保持している値を表示する場合、Spring Frameworkから提供されているJSPタグライブラリを使用する。

Spring Frameworkから提供されている ``<form:form>`` タグを使用してバインドする。

 .. code-block:: jsp
    :emphasize-lines: 2-3

    <form:form action="${pageContext.request.contextPath}/sample/hello"
               modelAttribute="sampleForm"> <%-- (1) --%>
        Id : <form:input path="id" /> <%-- (2) --%>
    </form:form>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - \ ``<form:form>``\ タグのmodelAttribute属性に、\ ``Model``\ に格納されているフォームオブジェクトの属性名を指定する。
   * - | (2)
     - \ ``<form:xxx>``\ タグのpath属性に、バインドしたいプロパティのプロパティ名を指定する。  ``xxx`` の部分は、入力項目のタイプによってかわる。

.. note::
    \ ``<form:form>``\ 、\ ``<form:xxx>``\ タグの詳細は、 `Using Spring's form tag library <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/view.html#view-jsp-formtaglib>`_\ を参照されたい。

|

.. _view_jsp_errors-label:

入力チェックエラーを表示する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
入力チェックエラーの内容を表示する場合、Spring Frameworkから提供されているJSPタグライブラリを使用する。

| Spring Frameworkから提供されている ``<form:errors>`` タグを使用して表示する。
| 詳細は、 :doc:`../ArchitectureInDetail/WebApplicationDetail/Validation` を参照されたい。

 .. code-block:: jsp
    :emphasize-lines: 3

    <form:form action="${pageContext.request.contextPath}/sample/hello"
               modelAttribute="sampleForm">
        Id : <form:input path="id" /><form:errors path="id" /><%-- (1) --%>
    </form:form>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - \ ``<form:errors>``\ タグのpath属性に、エラー表示したいプロパティのプロパティ名を指定する。

|

.. _view_jsp_resultmessages-label:

処理結果のメッセージを表示する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
処理結果を通知するメッセージを表示する場合、共通部品から提供しているJSPタグライブラリを使用する。

| 共通部品から提供している ``<t:messagesPanel>`` タグを使用する。
| 詳細は、 :doc:`../ArchitectureInDetail/WebApplicationDetail/MessageManagement` を参照されたい。

 .. code-block:: jsp
    :emphasize-lines: 3

    <div class="messages">
        <h2>Message pattern</h2>
        <t:messagesPanel /> <%-- (1) --%>
    </div>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - ``"resultMessages"`` という属性名で格納されているメッセージを出力する。

|

.. _view_jsp_codelist-label:

コードリストを表示する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
共通部品から提供されているコードリストを表示する場合は、Spring Frameworkから提供されているJSPタグライブラリを使用する。

| JSPからコードリストを参照する場合は、 ``java.util.Map`` インタフェースと同じ方法で参照することができる。
| 詳細は、 :doc:`../ArchitectureInDetail/WebApplicationDetail/Codelist` を参照されたい。

コードリストをセレクトボックスに表示する。

 .. code-block:: jsp
    :emphasize-lines: 3

    <form:select path="orderStatus">
        <form:option value="" label="--Select--" />
        <form:options items="${CL_ORDERSTATUS}" /> <%-- (1) --%>
    </form:select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - コードリスト名( ``"CL_ORDERSTATUS"`` ) を属性名として、コードリスト( ``java.util.Map`` インタフェース)が格納されている。
       そのためJSPでは、EL式を使ってコードリスト( ``java.util.Map`` インタフェース)にアクセスすることができる。
       取得した ``Map`` インタフェースを ``<form:options>`` のitems属性に渡すことで、コードリストをセレクトボックスに表示することができる。

セレクトボックスで選択した値のコード名を表示する。

 .. code-block:: jsp

    Order Status : ${f:h(CL_ORDERSTATUS[orderForm.orderStatus])}

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - セレクトボックス作成時と同様に、コードリスト名( ``"CL_ORDERSTATUS"`` ) を属性名として、コードリスト( ``java.util.Map`` インタフェース)を取得する。
       取得した ``Map`` インタフェースのキー値として、セレクトボックスで選択した値を指定することで、コード名を表示することができる。

|

.. _view_jsp_message-label:

固定文言を表示する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 画面名、項目名、ガイダンス用のメッセージなどについては、国際化の必要がない場合はJSPに直接記載してもよい。
| ただし、国際化の必要がある場合はSpring Frameworkから提供されているJSPタグライブラリを使用して、プロパティファイルから取得した値を表示する。

| Spring Frameworkから提供されている ``<spring:message>`` タグを使用して表示する。
| 詳細は、 :doc:`../ArchitectureInDetail/WebApplicationDetail/Internationalization` を参照されたい。

- properties

 .. code-block:: properties
    :emphasize-lines: 1-2

    # (1)
    label.orderStatus=注文ステータス

- jsp

 .. code-block:: jsp
    :emphasize-lines: 1

    <spring:message code="label.orderStatus" text="Order Status" /> : <%-- (2) --%>
        ${f:h(CL_ORDERSTATUS[orderForm.orderStatus])}

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - プロパティファイルにラベルの値を定義する。
   * - | (2)
     - ``<spring:message>`` のcode属性にプロパティファイルのキー名を指定するとキー名に一致するプロパティ値が表示される。

.. note::
     text属性に指定した値は、プロパティ値が取得できなかった場合に表示される。

|

.. _view_jsp_if-label:

条件によって表示を切り替える
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
モデルが保持する値によって表示を切り替えたい場合は、JSTLから提供されているJSPタグライブラリを使用する。

JSTLのJSPタグライブラリから提供されている ``<c:if>`` タグ又は ``<c:choose>`` を使用して、表示の切り替えを行う。

``<c:if>`` を使用して表示を切り替える。

 .. code-block:: jsp
    :emphasize-lines: 1

    <c:if test="${orderForm.orderStatus != 'complete'}"> <%-- (1) --%>
            <%-- ... --%>
    </c:if>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - ``<c:if>`` のtest属性に分岐に入る条件を実装する。例では注文ステータスが ``'complete'`` ではない場合に分岐内の表示処理が実行される。

``<c:choose>`` を使用して表示を切り替える。

 .. code-block:: jsp
    :emphasize-lines: 2,8

    <c:choose>
        <c:when test="${customer.type == 'premium'}"> <%-- (1) --%>
            <%-- ... --%>
        </c:when>
        <c:when test="${customer.type == 'general'}">
            <%-- ... --%>
        </c:when>
        <c:otherwise> <%-- (2) --%>
            <%-- ... --%>
        </c:otherwise>
    </c:choose>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - ``<c:when>`` タグのtest属性に分岐に入る条件を実装する。例では顧客の種別が ``'premium'`` の場合に分岐内の表示処理が実行される。
       test属性で指定した条件が ``false`` の場合は、次の ``<c:when>`` タグの処理が実行される。
   * - | (2)
     - 全ての ``<c:when>`` タグのtest属性の結果が ``false`` の場合、 ``<c:otherwise>`` タグ内の表示処理が実行される。

.. note::
    詳細は、 `JavaServer Pages Standard Tag Library(Version 1.2) <http://download.oracle.com/otndocs/jcp/jstl-1.2-mrel2-eval-oth-JSpec/>`_\ の "CHAPTER 5 Conditional Actions" を参照されたい。

|

.. _view_jsp_forEach-label:

コレクションの要素に対して表示処理を繰り返す
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
モデルが保持するコレクションに対して表示処理を繰り返したい場合は、JSTLから提供されているJSPタグライブラリを使用する。

JSTLのJSPタグライブラリから提供されている ``<c:forEach>`` を使用して表示処理を繰り返す。


 .. code-block:: jsp
    :emphasize-lines: 6,8-9

    <table>
        <tr>
            <th>No</th>
            <th>Name</th>
        </tr>
        <c:forEach var="customer" items="${customers}" varStatus="status"> <%-- (1) --%>
            <tr>
                <td>${status.count}</td> <%-- (2) --%>
                <td>${f:h(customer.name)}</td> <%-- (3) --%>
            </tr>
        </c:forEach>
    </table>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - ``<c:forEach>`` タグのitems属性にコレクションを指定する事で、``<c:forEach>`` タグ内の表示処理が繰り返し実行される。
       処理対象となっている要素のオブジェクトを参照する場合は、var属性にオブジェクトを格納するための変数名を指定する。
   * - | (2)
     - ``<c:forEach>`` タグのvarStatus属性で指定した変数から現在処理を行っている要素位置(count)を取得している。
       count以外の属性については、 ``javax.servlet.jsp.jstl.core.LoopTagStatus`` の `JavaDoc <http://download.oracle.com/otndocs/jcp/jstl-1.2-mrel2-eval-oth-JSpec/>`_\ を参照されたい。
   * - | (3)
     - ``<c:forEach>`` タグのvar属性で指定した変数に格納されているオブジェクトから値を取得している。

.. note::
    詳細は、 `JavaServer Pages Standard Tag Library(Version 1.2) <http://download.oracle.com/otndocs/jcp/jstl-1.2-mrel2-eval-oth-JSpec/>`_\ の "CHAPTER 6 Iterator Actions" を参照されたい。

|

.. _view_jsp_pagination-label:

ページネーション用のリンクを表示する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
一覧表示を行う画面にてページネーション用のリンクを表示する場合は、共通部品から提供しているJSPタグライブラリを使用する。

共通部品から提供している ``<t:pagination>`` を使用してページネーション用のリンクを表示する。
詳細は、 :doc:`../ArchitectureInDetail/WebApplicationDetail/Pagination` を参照されたい。


|

.. _view_jsp_authorization-label:

権限によって表示を切り替える
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
ログインしているユーザの権限によって表示を切り替える場合は、Spring Securityから提供されているJSPタグライブラリを使用する。

Spring Securityから提供されている ``<sec:authorize>`` を使用して表示の切り替えを行う。
詳細は、 :doc:`../Security/Authorization` を参照されたい。


|
|

JavaScriptの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
画面描画後に画面項目の制御(表示/非表示、活性/非活性などの制御)を行う必要がある場合は、JavaScriptを使用して、項目の制御を行う。

.. todo::

    **TBD**

    次版以降で詳細を記載する予定である。

|

スタイルシートの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 画面のデザインに関わる属性値の指定はJSP(HTML)に直接指定するのではなく、スタイルシート(cssファイル)に指定することを推奨する。
| JSP(HTML)では、項目を一意に特定するためのid属性の指定と項目の分類を示すclass属性の指定を行い、実際の項目の配置や見た目にかかわる属性値の指定はスタイルシート(cssファイル)で指定する。
| このような構成にすることで、JSPの実装からデザインに関わる処理を減らすことができる。
| 同時にちょっとしたデザイン変更であれば、JSPを修正せずにスタイルシート(cssファイル)の修正のみで対応可能となる。

.. note::
    ``<form:xxx>`` タグを使ってフォームを生成した場合、id属性は自動で設定される。class属性については、アプリケーション開発者によって指定が必要。

|

共通処理の実装
--------------------------------------------------------------------------------

|

.. _controller-common-process:

Controllerの呼び出し前後で行う共通処理の実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
本項でいう共通処理とは、Controllerを呼び出し前後に行う必要がある共通的な処理のことを指す。

|

Servlet Filterの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Spring MVCに依存しない共通処理については、Servlet Filterで実装する。
| ただし、Controllerのハンドラメソッドにマッピングされるリクエストに対してのみ共通処理を行いたい場合は、Servlet FilterではなくHandlerInterceptorで実装すること。

| 以下に、Servlet Filterのサンプルを示す。
| サンプルコードでは、クライアントのIPアドレスをログ出力するために ``MDC`` に値を格納している。

- java

 .. code-block:: java
    :emphasize-lines: 1

    public class ClientInfoPutFilter extends OncePerRequestFilter { // (1)

        private static final String ATTRIBUTE_NAME = "X-Forwarded-For";
        protected final void doFilterInternal(HttpServletRequest request,
                HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
            String remoteIp = request.getHeader(ATTRIBUTE_NAME);
            if (remoteIp == null) {
                remoteIp = request.getRemoteAddr();
            }
            MDC.put(ATTRIBUTE_NAME, remoteIp);
            try {
                filterChain.doFilter(request, response);
            } finally {
                MDC.remove(ATTRIBUTE_NAME);
            }
        }
    }

- web.xml

 .. code-block:: xml
    :emphasize-lines: 1,5

    <filter> <!-- (2) -->
        <filter-name>clientInfoPutFilter</filter-name>
        <filter-class>x.y.z.ClientInfoPutFilter</filter-class>
    </filter>
    <filter-mapping> <!-- (3) -->
        <filter-name>clientInfoPutFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - サンプルではSpring Frameworkから提供されている ``org.springframework.web.filter.OncePerRequestFilter`` の子クラスとしてServlet Filterを作成することで、同一リクエスト内で1回だけ実行されることを保証している。
   * - | (2)
     - 作成したServlet Filterを ``web.xml`` に登録する。
   * - | (3)
     - 登録したServlet Filterを適用するURLのパターンを指定する。


Servlet FilterをSpring FrameworkのBeanとして定義することもできる。

- web.xml

 .. code-block:: xml
    :emphasize-lines: 3

    <filter>
        <filter-name>clientInfoPutFilter</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class> <!-- (1) -->
    </filter>
    <filter-mapping>
        <filter-name>clientInfoPutFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

- applicationContext.xml

 .. code-block:: xml
    :emphasize-lines: 1

    <bean id="clientInfoPutFilter" class="x.y.z.ClientInfoPutFilter" /> <!-- (2) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - サンプルではSpring Frameworkから提供されている ``org.springframework.web.filter.DelegatingFilterProxy`` をServlet Filterのクラスに指定することで、(2)で定義したServlet Filterに処理が委譲される。
   * - | (2)
     - 作成したServlet FilterのクラスをBean定義ファイル( ``applicationContext.xml`` )に追加する。
       その際に、id属性には ``web.xml`` で指定したフィルター名( ``<filter-name>`` タグで指定した値 )にすること。

|

HandlerInterceptorの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Spring MVCに依存する共通処理については、 HandlerInterceptorで実装する。
| HandlerInterceptorは、リクエストにマッピングされたハンドラメソッドが決定した後に呼び出されるので、アプリケーションが許可しているリクエストに対してのみ共通処理を行うことができる。

HandlerInterceptorでは以下の３つのポイントで処理を実行することが出来る。

- | Controllerのハンドラメソッドを実行する前
  | ``HandlerInterceptor#preHandle`` メソッドとして実装する。
- | Controllerのハンドラメソッドが正常終了した後
  | ``HandlerInterceptor#postHandle`` メソッドとして実装する。
- | Controllerのハンドラメソッドの処理が完了した後(正常/異常に関係なく実行される)
  | ``HandlerInterceptor#afterCompletion`` メソッドとして実装する。

| 以下に、HandlerInterceptorのサンプルを示す。
| サンプルコードでは、Controllerの処理が正常終了した後にinfoレベルのログを出力している。

 .. code-block:: java
    :emphasize-lines: 1

    public class SuccessLoggingInterceptor extends HandlerInterceptorAdapter { // (1)

        private static final Logger logger = LoggerFactory
                .getLogger(SuccessLoggingInterceptor.class);

        @Override
        public void postHandle(HttpServletRequest request,
                HttpServletResponse response, Object handler,
                ModelAndView modelAndView) throws Exception {
            HandlerMethod handlerMethod = (HandlerMethod) handler;
            Method m = handlerMethod.getMethod();
            logger.info("[SUCCESS CONTROLLER] {}.{}", new Object[] {
                    m.getDeclaringClass().getSimpleName(), m.getName()});
        }

    }

- spring-mvc.xml

 .. code-block:: xml
    :emphasize-lines: 4-5,7

    <mvc:interceptors>
        <!-- ... -->
        <mvc:interceptor>
            <mvc:mapping path="/**" /> <!-- (2) -->
            <mvc:exclude-mapping path="/resources/**" /> <!-- (3) -->
            <mvc:exclude-mapping path="/**/*.html" />
            <bean class="x.y.z.SuccessLoggingInterceptor" /> <!-- (4) -->
        </mvc:interceptor>
        <!-- ... -->
    </mvc:interceptors>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - サンプルではSpring Frameworkから提供されている ``org.springframework.web.servlet.handler.HandlerInterceptorAdapter`` の子クラスとしてHandlerInterceptorを作成している。
       ``HandlerInterceptorAdapter`` は ``HandlerInterceptor`` インタフェースの空実装を提供しているため、子クラスで不要なメソッドの実装をしないで済む。
   * - | (2)
     - 作成したHandlerInterceptorを適用するパスのパターンを指定する。
   * - | (3)
     - 作成したHandlerInterceptorを適用しないパスのパターンを指定する。
   * - | (4)
     - 作成したHandlerInterceptorを ``spring-mvc.xml`` の ``<mvc:interceptors>`` タグ内に追加する。

|

Controllerの共通処理の実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
ここでいう共通処理とは、すべてのControllerで共通的に実装する必要がある処理のことを指す。

.. _methodargumentresolver:

HandlerMethodArgumentResolverの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Spring FrameworkのデフォルトでサポートされていないオブジェクトをControllerの引数として渡したい場合は、
HandlerMethodArgumentResolverを実装してControllerの引数として受け取れるようにする。

| 以下に、HandlerMethodArgumentResolverのサンプルを示す。
| サンプルコードでは、 共通的なリクエストパラメータをJavaBeanに変換してControllerのメソッドで受け取れるようにしている。


- JavaBean

 .. code-block:: java
    :emphasize-lines: 1

    public class CommonParameters implements Serializable { // (1)

        private String param1;
        private String param2;
        private String param3;

        // ....

    }


- HandlerMethodArgumentResolver

 .. code-block:: java
    :emphasize-lines: 2,6,13

    public class CommonParametersMethodArgumentResolver implements
                                                       HandlerMethodArgumentResolver { // (2)

        @Override
        public boolean supportsParameter(MethodParameter parameter) {
            return CommonParameters.class.equals(parameter.getParameterType()); // (3)
        }

        @Override
        public Object resolveArgument(MethodParameter parameter,
                ModelAndViewContainer mavContainer, NativeWebRequest webRequest,
                WebDataBinderFactory binderFactory) throws Exception {
            CommonParameters params = new CommonParameters(); // (4)
            params.setParam1(webRequest.getParameter("param1"));
            params.setParam2(webRequest.getParameter("param2"));
            params.setParam3(webRequest.getParameter("param3"));
            return params;
        }


- Controller

 .. code-block:: java
    :emphasize-lines: 2

    @RequestMapping(value = "home")
    public String home(CommonParameters commonParams) { // (5)
        logger.debug("param1 : {}",commonParams.getParam1());
        logger.debug("param2 : {}",commonParams.getParam2());
        logger.debug("param3 : {}",commonParams.getParam3());
        // ...
        return "sample/home";

    }

- spring-mvc.xml

 .. code-block:: xml
    :emphasize-lines: 4

    <mvc:annotation-driven>
        <mvc:argument-resolvers>
            <!-- ... -->
            <bean class="x.y.z.CommonParametersMethodArgumentResolver" /> <!-- (6) -->
            <!-- ... -->
        </mvc:argument-resolvers>
    </mvc:annotation-driven>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - 共通パラメータを保持するJavaBean。
   * - | (2)
     - ``org.springframework.web.method.support.HandlerMethodArgumentResolver`` インタフェースを実装する。
   * - | (3)
     - 処理対象とする型を判定する。例では、共通パラメータを保持するJavaBeanの型がControllerの引数として指定されていた場合に、このクラスのresolveArgumentメソッドが呼び出される。
   * - | (4)
     - リクエストパラメータから値を取得し、共通パラメータを保持するJavaBeanに設定し返却する。
   * - | (5)
     - | Controllerのハンドラメソッドの引数に共通パラメータを保持するJavaBeanを指定する。
       | (4)で返却されるオブジェクトが渡される。
   * - | (6)
     - 作成したHandlerMethodArgumentResolverを ``spring-mvc.xml`` の ``<mvc:argument-resolvers>`` タグ内に追加する。

.. note::
    全てのControllerのハンドラメソッドで共通的に渡すパラメータがある場合は、HandlerMethodArgumentResolverを使ってJavaBeanに変換してから渡す方法が有効的である。
    ここでいうパラメータとは、リクエストパラメータに限らない。

|

.. _application_layer_controller_advice:

\ ``@ControllerAdvice``\ の実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\ ``@ControllerAdvice``\ アノテーションを付与したクラスでは、
複数のControllerで実行したい共通的な処理を実装する。

\ ``@ControllerAdvice``\ アノテーションを付与したクラスを作成すると、

- ``@InitBinder`` を付与したメソッド
- ``@ExceptionHandler`` を付与したメソッド
- ``@ModelAttribute`` を付与したメソッド

で実装した処理を、複数のControllerに適用する事ができる。

.. tip::

    \ ``@ControllerAdvice``\ アノテーションは、Spring Framework 3.2 から追加された仕組みだが、
    全てのControllerに処理が適用される仕組みになっていたため、アプリケーション全体の共通処理しか実装できなかった。

    Spring Framework 4.0 からは、共通処理を適用するControllerを柔軟に指定する事ができるように改善されている。
    この改善により、様々な粒度で共通処理を実装する事ができるようになった。

|

.. _application_layer_controller_advice_attribute:

以下に、共通処理を適用するControllerを指定する方法(属性の指定方法)について説明する。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 15 75
   :class: longtable

   * - 項番
     - 属性
     - 説明と指定例
   * - | (1)
     - ``annotations``
     - アノテーションを指定する。

       指定したアノテーションが付与されたControllerに対して共通処理が適用される。
       以下に指定例を示す。

       .. code-block:: java

           @ControllerAdvice(annotations = LoginFormModelAttributeSetter.LoginFormModelAttribute.class)
           public class LoginFormModelAttributeSetter {
               @Target(ElementType.TYPE)
               @Retention(RetentionPolicy.RUNTIME)
               public static @interface LoginFormModelAttribute {}
               // ...
           }

       .. code-block:: java

           @LoginFormModelAttribute
           @Controller
           public class WelcomeController {
               // ...
           }

       .. code-block:: java

           @LoginFormModelAttribute
           @Controller
           public class LoginController {
               // ...
           }

       上記例では、\ ``WelcomeController``\ と\ ``LoginController``\ に\ ``@LoginFormModelAttribute``\ アノテーションを付与しているため、
       \ ``WelcomeController``\ と\ ``LoginController``\ に共通処理が適用される。
   * - | (2)
     - ``assignableTypes``
     - クラス又はインタフェースを指定する。

       指定したクラス又はインタフェースに割り当て可能(キャスト可能)なControllerに対して共通処理が適用される。
       本属性を使用する場合は、共通処理を適用するControllerであることを示すためのマーカーインタフェースを属性値に指定するスタイルを採用することを推奨する。
       このスタイルを採用した場合、Controller側では、適用したい共通処理用のマーカーインタフェースを実装するだけでよい。
       以下の指定例を示す。

       .. code-block:: java

           @ControllerAdvice(assignableTypes = ISODateInitBinder.ISODateApplicable.class)
           public class ISODateInitBinder {
               public static interface ISODateApplicable {}
               // ...
           }

       .. code-block:: java

           @Controller
           public class SampleController implements ISODateApplicable {
               // ...
           }

       上記例では、\ ``SampleController``\ が\ ``@ISODateApplicable``\ インタフェース(マーカーインタフェース)を実装しているため、
       \ ``SampleController``\ に共通処理が適用される。
   * - | (3)
     - ``basePackageClasses``
     - クラス又はインタフェースを指定する。

       指定したクラス又はインタフェースのパッケージ配下のControllerに対して共通処理が適用される。

       本属性を使用する場合は、

       * \ ``@ControllerAdvice``\ を付与したクラス
       * パッケージを識別するためのマーカーインタフェース

       を属性値に指定するスタイルを採用することを推奨する。
       以下に指定例を示す。

       .. code-block:: java

           package com.example.app

           @ControllerAdvice(basePackageClasses = AppGlobalExceptionHandler.class)
           public class AppGlobalExceptionHandler {
               // ...
           }

       .. code-block:: java

           package com.example.app.sample

           @Controller
           public class SampleController {
               // ...
           }

       上記例では、\ ``SampleController``\ が\ ``@ControllerAdvice``\ を付与したクラス(\ ``AppGlobalExceptionHandler``\)が格納されているパッケージ(\ ``com.example.app``\ )配下に格納されているため、
       \ ``SampleController``\ に共通処理が適用される。

       .. code-block:: java

           package com.example.app.common

           @ControllerAdvice(basePackageClasses = AppPackage.class)
           public class AppGlobalExceptionHandler {
               // ...
           }

       .. code-block:: java

           package com.example.app

           public interface AppPackage {
           }

       \ ``@ControllerAdvice``\ が付与されているクラスとControllerが格納されているクラスのパッケージ階層が異なる場合や、複数のベースパッケージに共通処理を適用したい場合は、
       パッケージを識別するためのマーカインタフェースを用意すればよい。
   * - | (4)
     - ``basePackages``
     - パッケージ名を指定する。

       指定したパッケージ配下のControllerに対して共通処理が適用される。
       以下に指定例を示す。

       .. code-block:: java

           @ControllerAdvice(basePackages = "com.example.app")
           public class AppGlobalExceptionHandler {
               // ...
           }
   * - | (5)
     - ``value``
     - \ ``basePackages``\ へのエイリアス。

       \ ``basePackages``\ 属性を指定した際と同じ動作となる。
       以下に指定例を示す。

       .. code-block:: java

           @ControllerAdvice("com.example.app")
           public class AppGlobalExceptionHandler {
               // ...
           }

.. raw:: latex

   \newpage

.. tip::

    \ ``basePackageClasses``\ 属性 / \ ``basePackages``\ 属性 / \ ``value``\ 属性は、
    共通処理を適用したいControllerが格納されているベースパッケージを指定するための属性であるが、
    \ ``basePackageClasses``\ 属性を使用した場合、

    * 存在しないパッケージを指定してしまう事を防ぐことが出来る
    * IDE上で行ったパッケージ名変更と連動することが出来る

    ため、タイプセーフな指定方法と言える。

|

| 以下に、\ ``@InitBinder``\ メソッドの実装サンプルを示す。
| サンプルコードでは、 リクエストパラメータで指定できる日付型で形式を ``"yyyy/MM/dd"`` に設定している。

 .. code-block:: java
    :emphasize-lines: 1,2,5-6

    @ControllerAdvice // (1)
    @Order(0) // (2)
    public class SampleControllerAdvice {

        // (3)
        @InitBinder
        public void initBinder(WebDataBinder binder) {
            SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy/MM/dd");
            dateFormat.setLenient(false);
            binder.registerCustomEditor(Date.class,
                    new CustomDateEditor(dateFormat, true));
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - \ ``@ControllerAdvice``\ アノテーションを付与することで、ControllerAdviceのBeanであることを示している。
   * - | (2)
     - \ ``@Order``\ アノテーションを付与することで、共通処理が適用される優先度を指定する。複数のControllerAdviceに依存関係があるなど、ControllerAdviceに順序性を持たせたい場合は必ず指定すること。順序性を持たせる必要がなければ指定しなくてもよい。
   * - | (3)
     - \ ``@InitBinder``\ メソッドを実装する。全てのControllerに対して\ ``@InitBinder``\ メソッドが適用される。

|

| 以下に、\ ``@ExceptionHandler``\ メソッドの実装サンプルを示す。
| サンプルコードでは、  ``org.springframework.dao.PessimisticLockingFailureException`` をハンドリングしてロックエラー画面のViewを返却している。

 .. code-block:: java
    :emphasize-lines: 1-2

    // (1)
    @ExceptionHandler(PessimisticLockingFailureException.class)
    public String handlePessimisticLockingFailureException(
            PessimisticLockingFailureException e) {
        return "error/lockError";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - \ ``@ExceptionHandler``\ メソッドを実装する。全てのControllerに対して\ ``@ExceptionHandler``\ メソッドが適用される。

|

| 以下に、\ ``@ModelAttribute``\ メソッドの実装サンプルを示す。
| サンプルコードでは、 共通的なリクエストパラメータをJavaBeanに変換して ``Model`` に格納している。

- ControllerAdvice

 .. code-block:: java
    :emphasize-lines: 1-2

    // (1)
    @ModelAttribute
    public CommonParameters setUpCommonParameters(
            @RequestParam(value = "param1", defaultValue="def1") String param1,
            @RequestParam(value = "param2", defaultValue="def2") String param2,
            @RequestParam(value = "param3", defaultValue="def3") String param3) {
        CommonParameters params = new CommonParameters();
        params.setParam1(param1);
        params.setParam2(param2);
        params.setParam3(param3);
        return params;
    }

- Controller

 .. code-block:: java
    :emphasize-lines: 2

    @RequestMapping(value = "home")
    public String home(@ModelAttribute CommonParameters commonParams) { // (2)
        logger.debug("param1 : {}",commonParams.getParam1());
        logger.debug("param2 : {}",commonParams.getParam2());
        logger.debug("param3 : {}",commonParams.getParam3());
        // ...
        return "sample/home";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - \ ``@ModelAttribute``\ メソッドを実装する。全てのControllerに対して\ ``@ModelAttribute``\ メソッドが適用される。
   * - | (2)
     - \ ``@ModelAttribute``\ メソッドで生成されたオブジェクトが渡る。


|

二重送信防止について
--------------------------------------------------------------------------------
送信ボタンの複数回押下や完了画面の再読み込み(F5ボタンによる再読み込み)などで、 同じ処理が複数回実行されてしまう可能性があるため、二重送信を防止するための対策は必ず行うこと。

対策を行わない場合に発生する問題点や対策方法の詳細は、 :doc:`../ArchitectureInDetail/WebApplicationDetail/DoubleSubmitProtection` を参照されたい。

|

セッションの使用について
--------------------------------------------------------------------------------
| Spring MVCのデフォルトの動作では、モデル（フォームオブジェクトやドメインオブジェクトなど）はセッションには格納されない。
| セッションに格納したい場合は、\ ``@SessionAttributes``\ アノテーションをControllerクラスに付与する必要がある。
| 入力フォームが複数の画面にわかれている場合は、 一連の画面遷移を行うリクエストでモデル（フォームオブジェクトやドメインオブジェクトなど）を共有できるため、 \ ``@SessionAttributes``\ アノテーションの利用を検討すること。
| ただし、セッションを使用する際の注意点があるので、そちらを確認した上で\ ``@SessionAttributes``\ アノテーションの利用有無を判断すること。

セッションの利用指針及びセッション使用時の実装方法の詳細は、 :doc:`../ArchitectureInDetail/WebApplicationDetail/SessionManagement` を参照されたい。

.. raw:: latex

   \newpage
