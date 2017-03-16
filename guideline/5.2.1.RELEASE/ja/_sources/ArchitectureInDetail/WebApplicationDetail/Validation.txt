入力チェック
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 4

Overview
--------------------------------------------------------------------------------

ユーザーが入力した値が不正かどうかを検証することは必須である。
入力値の検証は大きく分けて、

#. 長さや形式など、文脈によらず入力値だけを見て、それが妥当かどうかを判定できる検証
#. システムの状態によって入力値が妥当かどうかが変わる検証

がある。

1.の例としては必須チェックや、桁数チェックがあり、2.の例としては
登録済みのEMailかどうかのチェックや、注文数が在庫数以内であるかどうかのチェックが挙げられる。

本節では、基本的には前者のことを説明し、このチェックのことを「入力チェック」を呼ぶ。
後者のチェックは「業務ロジックチェック」と呼ぶ。業務ロジックチェックについては
\ :doc:`../../ImplementationAtEachLayer/DomainLayer`\ を参照されたい。

本ガイドラインでは、基本的に入力チェックをアプリケーション層で行い、
業務ロジックチェックは、ドメイン層で行うことをポリシーとする。


Webアプリケーションの入力チェックには、サーバサイドで行うチェックと、クライアントサイド(JavaScript)で行うチェックがある。
サーバーサイドのチェックは必須であるが、クライアントサイドでも同じチェックを実施すると、
サーバー通信なしでチェック結果が分かるため、ユーザビリティが向上する。

.. warning::

  JavaScriptによるクライアントサイドの処理は、改ざん可能であるため、サーバーサイドのチェックは、必ず行うこと。
  クライアントサイドのみでチェックを行い、サーバーサイドでチェックを省略した場合は、システムが危険な状態に晒されていることになる。


.. todo::

  クライアントサイドの入力チェックについては今後追記する。初版では、サーバーサイドの入力チェックのみ言及する。


入力チェックの分類
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

入力チェックは、単項目チェック、相関項目チェックに分類される。

.. tabularcolumns:: |p{0.15\linewidth}|p{0.30\linewidth}|p{0.25\linewidth}|p{0.30\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 15 30 25 30


   * - 種類
     - 説明
     - 例
     - 実現方法
   * - 単項目チェック
     - | 単一のフィールドで完結するチェック
     - | 入力必須チェック
       | 桁チェック
       | 型チェック
     - | Bean Validation (実装ライブラリとしてHibernate Validatorを使用)
   * - 相関項目チェック
     - | 複数のフィールドを比較するチェック
     - | パスワードと確認用パスワードの一致チェック
     - | `org.springframework.validation.Validator <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/validation.html#validator>`_\ インタフェースを実装したValidationクラス
       | または Bean Validation


Spring は、Java標準であるBean Validationをサポートしている。
単項目チェックには、このBean Validationを利用する。
相関項目チェックの場合は、Bean ValidationまたはSpringが提供している\ ``org.springframework.validation.Validator``\ インタフェースを利用する。



.. _Validation_how_to_use:

How to use
--------------------------------------------------------------------------------

.. ValidationAddDependencyLibrary:

依存ライブラリの追加
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Bean Validation 1.1(Hibernate Validator 5.x)以上を使用する場合、
Bean ValidationのAPI仕様クラス(\ ``javax.validation``\ パッケージのクラス)が格納されているjarファイルとHibernate Validatorのjarファイルに加えて、

* Expression Language 2.2以上のAPI仕様クラス (\ ``javax.el``\ パッケージのクラス)
* Expression Language 2.2以上のリファレンス実装クラス

が格納されているライブラリが必要となる。

アプリケーションサーバにデプロイして動かす場合は、
これらのライブラリはアプリケーションサーバから提供されているため、
依存ライブラリの追加は不要である。
ただし、スタンドアロン環境(JUnitなど)で動かす場合は、これらのライブラリを依存ライブラリとして追加する必要がある。

スタンドアロン環境でBean Validation 1.1以上を動かす際に必要となるライブラリの追加例を以下に示す。

.. code-block:: xml

    <!-- (1) -->
    <dependency>
        <groupId>org.apache.tomcat.embed</groupId>
        <artifactId>tomcat-embed-el</artifactId>
        <scope>test</scope> <!-- (2) -->
    </dependency>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - スタンドアロン環境で動かすプロジェクトの :file:`pom.xml` ファイルに、
        Expression Language用のクラスが格納されているライブラリを追加する。

        上記例では、組込み用のApache Tomcat向けに提供されているライブラリを指定している。
        \ ``tomcat-embed-el``\ のjarファイルには、Expression LanguageのAPI仕様クラスとリファレンス実装クラスの両方が格納されている。

    * - | (2)
      - JUnitを実行するために依存ライブラリが必要になる場合は、スコープは \ ``test``\ が適切である。

.. note::

    上記設定例は、依存ライブラリのバージョンを親プロジェクトである terasoluna-gfw-parent で管理する前提であるため、pom.xmlでのバージョンの指定は不要である。
    上記の依存ライブラリはterasoluna-gfw-parentが利用している\ `Spring IO Platform <http://platform.spring.io/platform/>`_\ で定義済みである。


.. _Validation_single_check:

単項目チェック
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

単項目チェックを実装するには、

* フォームクラスのフィールドに、Bean Validation用のアノテーションを付与する
* Controllerに、検証するための\ ``@Validated``\ アノテーションを付与する
* JSPに、検証エラーメッセージを表示するためのタグを追加する

が必要である。


.. note::

  spring-mvc.xmlに\ ``<mvc:annotation-driven>``\ の設定が行われていれば、Bean Validationは有効になる。


.. _Validation_basic_validation:

基本的な単項目チェック
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

「新規ユーザー登録」処理を例に用いて、実装方法を説明する。ここでは「新規ユーザー登録」のフォームに、以下のチェックルールを設ける。


.. tabularcolumns:: |p{0.20\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 30 50


   * - フィールド名
     - 型
     - ルール
   * - | name
     - | ``java.lang.String``
     - | 入力必須
       | 1文字以上
       | 20文字以下
   * - | email
     - | ``java.lang.String``
     - | 入力必須
       | 1文字以上
       | 50文字以下
       | Email形式
   * - | age
     - | ``java.lang.Integer``
     - | 入力必須
       | 1以上
       | 200以下

* フォームクラス

  フォームクラスの各フィールドに、Bean Validationのアノテーションを付ける。

  .. code-block:: java

      package com.example.sample.app.validation;

      import java.io.Serializable;

      import javax.validation.constraints.Max;
      import javax.validation.constraints.Min;
      import javax.validation.constraints.NotNull;
      import javax.validation.constraints.Size;

      import org.hibernate.validator.constraints.Email;

      public class UserForm implements Serializable {

        private static final long serialVersionUID = 1L;

        @NotNull // (1)
        @Size(min = 1, max = 20) // (2)
        private String name;

        @NotNull
        @Size(min = 1, max = 50)
        @Email // (3)
        private String email;

        @NotNull // (4)
        @Min(0) // (5)
        @Max(200) // (6)
        private Integer age;

        // omitted setter/getter
      }


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90


     * - 項番
       - 説明
     * - | (1)
       - | 対象のフィールドが\ ``null``\ でないことを示す\ ``javax.validation.constraints.NotNull``\ を付ける。
         |
         | Spring MVCでは、文字列の入力フィールドに未入力の状態でフォームを送信した場合、
         | デフォルトではフォームオブジェクトに\ **nullではなく、空文字がバインドされる**\ 。
         | この\ ``@NotNull``\ は、そもそもリクエストパラメータとして\ ``name``\ が存在することをチェックする。
     * - | (2)
       - | 対象のフィールドの文字列長(またはコレクションのサイズ)が指定したサイズの範囲内にあることを示す\ ``javax.validation.constraints.Size``\ を付ける。
         |
         | 上記の通り、Spring MVCではデフォルトで、未入力の文字列フィールドには、空文字がバインドされるため、
         | 1文字以上というルールが入力必須を表す。
     * - | (3)
       - | 対象のフィールドがRFC2822準拠のE-mail形式であることを示す\ ``org.hibernate.validator.constraints.Email``\ を付ける。
         | E-mail形式の要件がRFC2822準拠の制限よりも緩い場合は、\ ``@Email``\ を使用せず、\ ``javax.validation.constraints.Pattern``\ を用いて、正規表現を指定する必要がある。
     * - | (4)
       - | 数値の入力フィールドに未入力の状態でフォームを送信した場合、フォームオブジェクトに\ ``null`` \ がバインドされるため、\ ``@NotNull``\ が\ ``age``\ の入力必須条件を表す。
     * - | (5)
       - | 対象のフィールドが指定した数値の以上であることを示す\ ``javax.validation.constraints.Min``\ を付ける。
     * - | (6)
       - | 対象のフィールドが指定した数値の以下であることを示す\ ``javax.validation.constraints.Max``\ を付ける。


  .. tip::
  
    Bean Validation標準のアノテーション、Hibernate Validationが用意しているアノテーションについては、\ :ref:`Validation_jsr303_doc`\ 、\ :ref:`Validation_validator_list`\ を参照されたい。
  
  .. tip::
  
    入力フィールドが未入力の場合に、空文字ではなく\ ``null``\ にバインドする方法に関しては、\ :ref:`Validation_string_trimmer_editor`\ を参照されたい、

* Controllerクラス

  入力チェック対象のフォームクラスに、\ ``@Validated``\ を付ける。

  .. code-block:: java

      package com.example.sample.app.validation;

      import org.springframework.stereotype.Controller;
      import org.springframework.validation.BindingResult;
      import org.springframework.validation.annotation.Validated;
      import org.springframework.web.bind.annotation.ModelAttribute;
      import org.springframework.web.bind.annotation.RequestMapping;
      import org.springframework.web.bind.annotation.RequestMethod;

      @Controller
      @RequestMapping("user")
      public class UserController {

        @ModelAttribute
        public UserForm setupForm() {
          return new UserForm();
        }

        @RequestMapping(value = "create", method = RequestMethod.GET, params = "form")
        public String createForm() {
          return "user/createForm"; // (1)
        }

        @RequestMapping(value = "create", method = RequestMethod.POST, params = "confirm")
        public String createConfirm(@Validated /* (2) */ UserForm form, BindingResult /* (3) */ result) {
          if (result.hasErrors()) { // (4)
            return "user/createForm";
          }
          return "user/createConfirm";
        }

        @RequestMapping(value = "create", method = RequestMethod.POST)
        public String create(@Validated UserForm form, BindingResult result) { // (5)
          if (result.hasErrors()) {
            return "user/createForm";
          }
          // omitted business logic
          return "redirect:/user/create?complete";
        }

        @RequestMapping(value = "create", method = RequestMethod.GET, params = "complete")
        public String createComplete() {
          return "user/createComplete";
        }
      }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | 「新規ユーザー登録」フォーム画面を表示する。
     * - | (2)
       - | フォームにつけたアノテーションで入力チェックをするために、フォームの引数に \ ``org.springframework.validation.annotation.Validated``\ を付ける。
     * - | (3)
       - | (2)のチェック結果を格納する\ ``org.springframework.validation.BindingResult``\ を、引数に加える。
         | この\ ``BindingResult``\ は、フォームの直後に記述する必要がある。
         |
         | 直後に指定されていない場合は、検証後に結果をバインドできず、\ ``org.springframework.validation.BindException``\ がスローされる。
     * - | (4)
       - | (2)のチェック結果は、\ ``BindingResult.hasErrors()``\ メソッドで判定できる。
         | \ ``hasErrors()``\ の結果が\ ``true``\ の場合は、入力値に問題があるため、フォーム表示画面に戻す。
     * - | (5)
       - | 入力内容確認画面から新規作成処理にリクエストを送る際にも、\ **入力チェックを必ず再実行すること**\ 。
         | 途中でデータを改ざんすることは可能であるため、必ず業務処理の直前で入力チェックは必要である。


  .. note::
  
    \ ``@Validated``\ は、Bean Validation標準ではなく、Springの独自アノテーションである。
    Bean Validation標準の\ ``javax.validation.Valid``\ アノテーションも使用できるが、\ ``@Validated``\ は\ ``@Valid``\ に比べて、
    バリデーションのグループを指定できる点で優れているため、本ガイドラインではControllerの引数には、\ ``@Validated``\ を使用することを推奨する。


.. _Validation_jsp_impl_sample:

* JSP

  \ ``<form:errors>``\ タグで、入力エラーがある場合にエラーメッセージを表示できる。

  .. code-block:: jsp

      <!DOCTYPE html>
      <html>
      <%-- WEB-INF/views/user/createForm.jsp --%>
      <body>
          <form:form modelAttribute="userForm" method="post"
              action="${pageContext.request.contextPath}/user/create">
              <form:label path="name">Name:</form:label>
              <form:input path="name" />
              <form:errors path="name" /><%--(1) --%>
              <br>
              <form:label path="email">Email:</form:label>
              <form:input path="email" />
              <form:errors path="email" />
              <br>
              <form:label path="age">Age:</form:label>
              <form:input path="age" />
              <form:errors path="age" />
              <br>
              <form:button name="confirm">Confirm</form:button>
          </form:form>
      </body>
      </html>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | \ ``<form:errors>``\ タグの\ ``path``\ 属性に、対象のフィールド名を指定する。
         | この例では、フィールド毎に入力フィールドの横にエラーメッセージを表示する。

フォームは、以下のように表示される。

.. figure:: ./images_Validation/validations-first-sample1.png
  :width: 60%

このフォームに対して、すべての入力フィールドを未入力のまま送信すると、以下のようにエラーメッセージが表示される。

.. figure:: ./images_Validation/validations-first-sample2.png
  :width: 60%

NameとEmailが空文字であることに対するエラーメッセージと、Ageが\ ``null``\ であることに対するエラーメッセージが表示されている。

.. note::

  Bean Validationでは、通常、入力値が\ ``null``\ の場合は正常な値とみなす。ただし、
  以下のアノテーションを除く。

  * ``javax.validation.constraints.NotNull``
  * ``org.hibernate.validator.constraints.NotEmpty``
  * ``org.hibernate.validator.constraints.NotBlank``

  上記の例では、Ageの値は\ ``null``\ であるため、\ ``@Min``\ と\ ``@Max``\ によるチェックは正常とみなされ、
  エラーメッセージは出力されていない。

次に、フィールドに何らかの値を入力してフォームを送信する。

.. figure:: ./images_Validation/validations-first-sample3.png
  :width: 60%

| Nameの入力値は、チェック条件を満たすため、エラーメッセージが表示されない。
| Emailの入力値は文字列長に関する条件は満たすが、Email形式ではないため、エラーメッセージが表示される。
| Ageの入力値は最大値を超えているため、エラーメッセージが表示される。


エラー時にスタイルを変更したい場合は、前述のフォームを、以下のように変更する。

.. code-block:: jsp

    <form:form modelAttribute="userForm" method="post"
        class="form-horizontal"
        action="${pageContext.request.contextPath}/user/create">
        <form:label path="name" cssErrorClass="error-label">Name:</form:label><%-- (1) --%>
        <form:input path="name" cssErrorClass="error-input" /><%-- (2) --%>
        <form:errors path="name" cssClass="error-messages" /><%-- (3) --%>
        <br>
        <form:label path="email" cssErrorClass="error-label">Email:</form:label>
        <form:input path="email" cssErrorClass="error-input" />
        <form:errors path="email" cssClass="error-messages" />
        <br>
        <form:label path="age" cssErrorClass="error-label">Age:</form:label>
        <form:input path="age" cssErrorClass="error-input" />
        <form:errors path="age" cssClass="error-messages" />
        <br>
        <form:button name="confirm">Confirm</form:button>
    </form:form>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | エラー時に\ ``<label>``\ タグへ加えるクラス名を、\ ``cssErrorClass``\ 属性で指定する。
   * - | (2)
     - | エラー時に\ ``<input>``\ タグへ加えるクラス名を、\ ``cssErrorClass``\ 属性で指定する。
   * - | (3)
     - | エラーメッセージに加えるクラス名を、\ ``cssClass``\ 属性で指定する。

このJSPに対して、例えば以下のCSSを適用すると、

.. code-block:: css

    .form-horizontal input {
        display: block;
        float: left;
    }

    .form-horizontal label {
        display: block;
        float: left;
        text-align: right;
        float: left;
    }

    .form-horizontal br {
        clear: left;
    }

    .error-label {
        color: #b94a48;
    }

    .error-input {
        border-color: #b94a48;
        margin-left: 5px;
    }

    .error-messages {
        color: #b94a48;
        display: block;
        padding-left: 5px;
        overflow-x: auto;
    }

エラー画面は、以下のように表示される。


.. figure:: ./images_Validation/validations-has-errors1.png
  :width: 60%


画面の要件に応じてCSSをカスタマイズすればよい。


エラーメッセージを、入力フィールドの横に一件一件出力する代わりに、
まとめて出力することもできる。


.. code-block:: jsp

    <form:form modelAttribute="userForm" method="post"
        action="${pageContext.request.contextPath}/user/create">
        <form:errors path="*" element="div" cssClass="error-message-list" /><%-- (1) --%>

        <form:label path="name" cssErrorClass="error-label">Name:</form:label>
        <form:input path="name" cssErrorClass="error-input" />
        <br>
        <form:label path="email" cssErrorClass="error-label">Email:</form:label>
        <form:input path="email" cssErrorClass="error-input" />
        <br>
        <form:label path="age" cssErrorClass="error-label">Age:</form:label>
        <form:input path="age" cssErrorClass="error-input" />
        <br>
        <form:button name="confirm">Confirm</form:button>
    </form:form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``<form:form>``\ タグ内で、\ ``<form:errors>``\ の\ ``path``\ 属性に\ ``*``\ を指定することで、
       | \ ``<form:form>``\ の\ ``modelAttribute``\ 属性に指定したModelに関する全エラーメッセージを出力できる。
       | ``element``\ 属性に、これらのエラーメッセージを包含するタグ名を指定できる。デフォルトでは、\ ``span``\ であるが、
       | ここではエラーメッセージ一覧をブロック要素として出力するために、\ ``div``\ を指定する。
       | また、CSSのクラスを ``cssClass``\ 属性に指定する。



例として、以下のCSSクラスを適用した場合の、エラーメッセージ出力例を示す。

.. code-block:: css

    .form-horizontal input {
        display: block;
        float: left;
    }

    .form-horizontal label {
        display: block;
        float: left;
        text-align: right;
        float: left;
    }

    .form-horizontal br {
        clear: left;
    }

    .error-label {
        color: #b94a48;
    }

    .error-input {
        border-color: #b94a48;
        margin-left: 5px;
    }

    .error-message-list {
        color: #b94a48;
        padding:5px 10px;
        background-color: #fde9f3;
        border:1px solid #c98186;
        border-radius:5px;
        margin-bottom: 10px;
    }


.. figure:: ./images_Validation/validations-has-errors2.png
  :width: 60%


| デフォルトでは、エラーメッセージにフィールド名は含まれず、どのフィールドのエラーメッセージなのかが分かりにくい。
| そのため、エラーメッセージを一覧で表示する場合は、エラーメッセージの中にフィールド名を含めるようにメッセージを定義する必要がある。
| エラーメッセージの定義方法については、「:ref:`Validation_message_def`」を参照されたい。

.. note:: **エラーメッセージを一覧で表示する際の注意点**

   エラーメッセージの出力順序は順不同であり、標準機能で出力順序を制御することはできない。
   そのため、出力順序を制御する(一定に保つ)必要がある場合は、エラー情報をソートするなどの拡張実装が必要となる。

   「エラーメッセージを一覧で表示する」方式では、

   * フィード単位のエラーメッセージ定義
   * エラーメッセージの出力順序を制御するための拡張実装

   が必要となるため、「入力フィールドの横にエラーメッセージを表示する」方式に比べて対応コストが高くなる。
   **本ガイドラインでは、画面要件による制約がない場合は「入力フィールドの横にエラーメッセージを表示する」方式を推奨する。**

   なお、エラーメッセージの出力順序を制御するための拡張方法としては、
   Spring Frameworkから提供されている\ ``org.springframework.validation.beanvalidation.LocalValidatorFactoryBean``\ の継承クラスを作成し、
   \ ``processConstraintViolations``\ メソッドをオーバーライドしてエラー情報をソートする方法などが考えられる。

.. note:: **@GroupSequenceアノテーションについて**

   チェック順番を制御するための仕組みとして\ `@GroupSequenceアノテーション <http://docs.jboss.org/hibernate/validator/5.2/reference/en-US/html/ch05.html#section-default-group-class>`_\ が提供されているが、
   この仕組みは以下のような動作になるため、エラーメッセージの出力順序を制御するための仕組みではないという点を補足しておく。

   * エラーが発生した場合に後続のグループのチェックが実行されない。
   * 同一グループ内のチェックで複数のエラー(複数の項目でエラー)が発生するとエラーメッセージの出力順序は順不同になる。


.. note::


   エラーメッセージをまとめて表示する際に、\ ``<form:form>``\ タグの外に表示したい場合は以下のように\ ``<spring:nestedPath>``\ タグを使用する。

     .. code-block:: jsp
       :emphasize-lines: 1,4

       <spring:nestedPath path="userForm">
           <form:errors path="*" element="div"
               cssClass="error-message-list" />
       </spring:nestedPath>
       <hr>
       <form:form modelAttribute="userForm" method="post"
           action="${pageContext.request.contextPath}/user/create">
           <form:label path="name" cssErrorClass="error-label">Name:</form:label>
           <form:input path="name" cssErrorClass="error-input" />
           <br>
           <form:label path="email" cssErrorClass="error-label">Email:</form:label>
           <form:input path="email" cssErrorClass="error-input" />
           <br>
           <form:label path="age" cssErrorClass="error-label">Age:</form:label>
           <form:input path="age" cssErrorClass="error-input" />
           <br>
           <form:button name="confirm">Confirm</form:button>
       </form:form>

日時フォーマットのチェック
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 日時フォーマットのチェックを行う場合には、Bean Validationの仕組みではなく、Springが提供する日時のフォーマットを指定する\ ``@DateTimeFormat``\ アノテーションの使用を推奨する。
| \ ``@DateTimeFormat``\ アノテーションの使用方法については、\ :ref:`ApplicationLayer-DateTimeFormat`\ を参照されたい。
| Bean Validationの\ ``@Pattern``\ アノテーションを使用することでも日時フォーマットのチェックは可能である。
| しかし、\ ``@Pattern``\ アノテーションを使用すると、日時フォーマットを正規表現で記述する必要があり、存在しない日時をチェックする場合には、記述が煩雑化する。
| そのため、\ ``@Pattern``\ アノテーションよりも\ ``@DateTimeFormat``\ アノテーションのほうが実装がシンプルになる。


| \ ``@DateTimeFormat``\ アノテーションはSpringが提供する型変換の仕組みのひとつであるので、入力エラーの場合には、Bean Validationのエラーメッセージではなく、型のミスマッチが発生した時にスローされる例外(\ ``TypeMismatchException``\ )の例外メッセージがそのまま画面へ表示される。
| 例外メッセージが画面に表示されることを避けるため、型のミスマッチが発生した際のエラーメッセージを\ **プロパティファイル**\ に設定する必要がある。
| 詳細は\ :ref:`Validation_type_mismatch`\ を参照されたい。


ネストしたBeanの単項目チェック
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
ネストしたBeanをBean Validationで検証する方法を説明する。

ECサイトにおける「注文」処理の例を考える。「注文」フォームでは、以下のチェックルールを設ける。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.30\linewidth}|p{0.30\linewidth}|p{0.20\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 30 30 20


   * - フィールド名
     - 型
     - ルール
     - 説明
   * - | coupon
     - | ``java.lang.String``
     - | 5文字以下
       | 半角英数字
     - | クーポンコード
   * - | receiverAddress.name
     - | ``java.lang.String``
     - | 入力必須
       | 1文字以上
       | 50文字以下
     - | お届け先氏名
   * - | receiverAddress.postcode
     - | ``java.lang.String``
     - | 入力必須
       | 1文字以上
       | 10文字以下
     - | お届け先郵便番号
   * - | receiverAddress.address
     - | ``java.lang.String``
     - | 入力必須
       | 1文字以上
       | 100文字以下
     - | お届け先住所
   * - | senderAddress.name
     - | ``java.lang.String``
     - | 入力必須
       | 1文字以上
       | 50文字以下
     - | 請求先氏名
   * - | senderAddress.postcode
     - | ``java.lang.String``
     - | 入力必須
       | 1文字以上
       | 10文字以下
     - | 請求先郵便番号
   * - | senderAddress.address
     - | ``java.lang.String``
     - | 入力必須
       | 1文字以上
       | 100文字以下
     - | 請求先住所

\ ``receiverAddress``\ と\ ``senderAddress``\ は、同じ項目であるため、同じフォームクラスを使用する。

* フォームクラス

  .. code-block:: java

    package com.example.sample.app.validation;

    import java.io.Serializable;

    import javax.validation.Valid;
    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Pattern;
    import javax.validation.constraints.Size;

    public class OrderForm implements Serializable {
        private static final long serialVersionUID = 1L;

        @Size(max = 5)
        @Pattern(regexp = "[a-zA-Z0-9]*")
        private String coupon;

        @NotNull // (1)
        @Valid // (2)
        private AddressForm receiverAddress;

        @NotNull
        @Valid
        private AddressForm senderAddress;

        // omitted setter/getter
    }


  .. code-block:: java

    package com.example.sample.app.validation;

    import java.io.Serializable;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;

    public class AddressForm implements Serializable {
        private static final long serialVersionUID = 1L;

        @NotNull
        @Size(min = 1, max = 50)
        private String name;

        @NotNull
        @Size(min = 1, max = 10)
        private String postcode;

        @NotNull
        @Size(min = 1, max = 100)
        private String address;

        // omitted setter/getter
    }


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | 子フォーム自体が必須であることを示す。
         | この設定がない場合、\ ``receiverAddress``\ に\ ``null``\ が設定されても、正常とみなされる。
     * - | (2)
       - | ネストしたBeanのBean Validationを有効にするために、\ ``javax.validation.Valid``\ アノテーションを付与する。


* Controllerクラス

  前述のControllerと違いはない。

  .. code-block:: java

    package com.example.sample.app.validation;

    import org.springframework.stereotype.Controller;
    import org.springframework.validation.BindingResult;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;

    @RequestMapping("order")
    @Controller
    public class OrderController {

        @ModelAttribute
        public OrderForm setupForm() {
            return new OrderForm();
        }

        @RequestMapping(value = "order", method = RequestMethod.GET, params = "form")
        public String orderForm() {
            return "order/orderForm";
        }

        @RequestMapping(value = "order", method = RequestMethod.POST, params = "confirm")
        public String orderConfirm(@Validated OrderForm form, BindingResult result) {
            if (result.hasErrors()) {
                return "order/orderForm";
            }
            return "order/orderConfirm";
        }
    }

* JSP

  .. code-block:: jsp

    <!DOCTYPE html>
    <html>
    <%-- WEB-INF/views/order/orderForm.jsp --%>
    <head>
    <style type="text/css">
      /* omitted (same as previous sample) */
    </style>
    </head>
    <body>
        <form:form modelAttribute="orderForm" method="post"
            class="form-horizontal"
            action="${pageContext.request.contextPath}/order/order">
            <form:label path="coupon" cssErrorClass="error-label">Coupon Code:</form:label>
            <form:input path="coupon" cssErrorClass="error-input" />
            <form:errors path="coupon" cssClass="error-messages" />
            <br>
        <fieldset>
            <legend>Receiver</legend>
            <%-- (1) --%>
            <form:errors path="receiverAddress"
                cssClass="error-messages" />
            <%-- (2) --%>
            <form:label path="receiverAddress.name"
                cssErrorClass="error-label">Name:</form:label>
            <form:input path="receiverAddress.name"
                cssErrorClass="error-input" />
            <form:errors path="receiverAddress.name"
                cssClass="error-messages" />
            <br>
            <form:label path="receiverAddress.postcode"
                cssErrorClass="error-label">Postcode:</form:label>
            <form:input path="receiverAddress.postcode"
                cssErrorClass="error-input" />
            <form:errors path="receiverAddress.postcode"
                cssClass="error-messages" />
            <br>
            <form:label path="receiverAddress.address"
                cssErrorClass="error-label">Address:</form:label>
            <form:input path="receiverAddress.address"
                cssErrorClass="error-input" />
            <form:errors path="receiverAddress.address"
                cssClass="error-messages" />
        </fieldset>
        <br>
        <fieldset>
            <legend>Sender</legend>
            <form:errors path="senderAddress"
                cssClass="error-messages" />
            <form:label path="senderAddress.name"
                cssErrorClass="error-label">Name:</form:label>
            <form:input path="senderAddress.name"
                cssErrorClass="error-input" />
            <form:errors path="senderAddress.name"
                cssClass="error-messages" />
            <br>
            <form:label path="senderAddress.postcode"
                cssErrorClass="error-label">Postcode:</form:label>
            <form:input path="senderAddress.postcode"
                cssErrorClass="error-input" />
            <form:errors path="senderAddress.postcode"
                cssClass="error-messages" />
            <br>
            <form:label path="senderAddress.address"
                cssErrorClass="error-label">Address:</form:label>
            <form:input path="senderAddress.address"
                cssErrorClass="error-input" />
            <form:errors path="senderAddress.address"
                cssClass="error-messages" />
        </fieldset>

            <form:button name="confirm">Confirm</form:button>
        </form:form>
    </body>
    </html>



  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | 不正な操作により、\ ``receiverAddress.name``\ 、\ ``receiverAddress.postcode``\ 、\ ``receiverAddress.address``\ のすべて
         | がリクエストパラメータとして送信されない場合、\ ``receiverAddress``\ が\ ``null``\ とみなされ、この位置にエラーメッセージが表示される。
     * - | (2)
       - | 子フォームのフィールドは、\ ``親フィールド名.子フィールド名``\ で指定する。


フォームは、以下のように表示される。

.. figure:: ./images_Validation/validations-nested1.png
  :width: 60%

このフォームに対して、すべての入力フィールドを未入力のまま送信すると、以下のようにエラーメッセージが表示される。

.. figure:: ./images_Validation/validations-nested2.png
  :width: 60%


ネストしたBeanのバリデーションはコレクションに対しても有効である。

最初に説明した「ユーザー登録」フォームに住所を3件まで登録できるようにフィールドを追加する。
住所には、前述の\ ``AddressForm``\ を利用する。


* フォームクラス
  \ ``AddressForm``\ のリストを、フィールドに追加する。

  .. code-block:: java
    :emphasize-lines: 32-35

    package com.example.sample.app.validation;

    import java.io.Serializable;
    import java.util.List;

    import javax.validation.Valid;
    import javax.validation.constraints.Max;
    import javax.validation.constraints.Min;
    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;

    import org.hibernate.validator.constraints.Email;

    public class UserForm implements Serializable {

        private static final long serialVersionUID = 1L;

        @NotNull
        @Size(min = 1, max = 20)
        private String name;

        @NotNull
        @Size(min = 1, max = 50)
        @Email
        private String email;

        @NotNull
        @Min(0)
        @Max(200)
        private Integer age;

        @NotNull
        @Size(min = 1, max = 3) // (1)
        @Valid
        private List<AddressForm> addresses;

        // omitted setter/getter
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | コレクションのサイズチェックにも、\ ``@Size``\ アノテーションを使用できる。
* JSP

  .. code-block:: jsp
    :emphasize-lines: 26-58

    <!DOCTYPE html>
    <html>
    <%-- WEB-INF/views/user/createForm.jsp --%>
    <head>
    <style type="text/css">
      /* omitted (same as previous sample) */
    </style>
    </head>
    <body>

        <form:form modelAttribute="userForm" method="post"
            class="form-horizontal"
            action="${pageContext.request.contextPath}/user/create">
            <form:label path="name" cssErrorClass="error-label">Name:</form:label>
            <form:input path="name" cssErrorClass="error-input" />
            <form:errors path="name" cssClass="error-messages" />
            <br>
            <form:label path="email" cssErrorClass="error-label">Email:</form:label>
            <form:input path="email" cssErrorClass="error-input" />
            <form:errors path="email" cssClass="error-messages" />
            <br>
            <form:label path="age" cssErrorClass="error-label">Age:</form:label>
            <form:input path="age" cssErrorClass="error-input" />
            <form:errors path="age" cssClass="error-messages" />
            <br>
            <form:errors path="addresses" cssClass="error-messages" /><%-- (1) --%>
            <c:forEach items="${userForm.addresses}" varStatus="status"><%-- (2) --%>
                <fieldset class="address">
                    <legend>Address${f:h(status.index + 1)}</legend>
                    <form:label path="addresses[${status.index}].name"
                        cssErrorClass="error-label">Name:</form:label><%-- (3) --%>
                    <form:input path="addresses[${status.index}].name"
                        cssErrorClass="error-input" />
                    <form:errors path="addresses[${status.index}].name"
                        cssClass="error-messages" />
                    <br>
                    <form:label path="addresses[${status.index}].postcode"
                        cssErrorClass="error-label">Postcode:</form:label>
                    <form:input path="addresses[${status.index}].postcode"
                        cssErrorClass="error-input" />
                    <form:errors path="addresses[${status.index}].postcode"
                        cssClass="error-messages" />
                    <br>
                    <form:label path="addresses[${status.index}].address"
                        cssErrorClass="error-label">Address:</form:label>
                    <form:input path="addresses[${status.index}].address"
                        cssErrorClass="error-input" />
                    <form:errors path="addresses[${status.index}].address"
                        cssClass="error-messages" />
                    <c:if test="${status.index > 0}">
                        <br>
                        <button class="remove-address-button">Remove</button>
                    </c:if>
                </fieldset>
                <br>
            </c:forEach>
            <button id="add-address-button">Add address</button>
            <br>
            <form:button name="confirm">Confirm</form:button>
        </form:form>
        <script type="text/javascript"
            src="${pageContext.request.contextPath}/resources/vendor/js/jquery-1.10.2.min.js"></script>
        <script type="text/javascript"
            src="${pageContext.request.contextPath}/resources/app/js/AddressesView.js"></script>
    </body>
    </html>


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | \ ``address``\ フィールドに対するエラーメッセージを表示する。
     * - | (2)
       - | 子フォームのコレクションを、\ ``<c:forEach>``\ タグを使ってループで処理する。
     * - | (3)
       - | コレクション中の子フォームのフィールドは、\ ``親フィールド名[インデックス].子フィールド名``\ で指定する。


* Controllerクラス

  .. code-block:: java
    :emphasize-lines: 20-22

    package com.example.sample.app.validation;

    import java.util.ArrayList;
    import java.util.List;

    import org.springframework.stereotype.Controller;
    import org.springframework.validation.BindingResult;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;

    @Controller
    @RequestMapping("user")
    public class UserController {

        @ModelAttribute
        public UserForm setupForm() {
            UserForm form = new UserForm();
            List<AddressForm> addresses = new ArrayList<AddressForm>();
            addresses.add(new AddressForm());
            form.setAddresses(addresses); // (1)
            return form;
        }

        @RequestMapping(value = "create", method = RequestMethod.GET, params = "form")
        public String createForm() {
            return "user/createForm";
        }

        @RequestMapping(value = "create", method = RequestMethod.POST, params = "confirm")
        public String createConfirm(@Validated UserForm form, BindingResult result) {
            if (result.hasErrors()) {
                return "user/createForm";
            }
            return "user/createConfirm";
        }
    }


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | 「ユーザー登録」フォーム初期表示時に、一件の住所フォームを表示させるために、フォームオブジェクトを編集する。

* JavaScript

  動的にアドレス入力フィールドを追加するためのJavaScriptも記載するが、このコードの説明は、本質的ではないため割愛する。

  .. code-block:: javascript

    // webapp/resources/app/js/AddressesView.js

    function AddressesView() {
      this.addressSize = $('fieldset.address').size();
    };

    AddressesView.prototype.addAddress = function() {
      var $address = $('fieldset.address');
      var newHtml = addressTemplate(this.addressSize++);
      $address.last().next().after($(newHtml));
    };

    AddressesView.prototype.removeAddress = function($fieldset) {
      $fieldset.next().remove(); // remove <br>
      $fieldset.remove(); // remove <fieldset>
    };

    function addressTemplate(number) {
      return '\
    <fieldset class="address">\
        <legend>Address' + (number + 1) + '</legend>\
        <label for="addresses' + number + '.name">Name:</label>\
        <input id="addresses' + number + '.name" name="addresses[' + number + '].name" type="text" value=""/><br>\
        <label for="addresses' + number + '.postcode">Postcode:</label>\
        <input id="addresses' + number + '.postcode" name="addresses[' + number + '].postcode" type="text" value=""/><br>\
        <label for="addresses' + number + '.address">Address:</label>\
        <input id="addresses' + number + '.address" name="addresses[' + number + '].address" type="text" value=""/><br>\
        <button class="remove-address-button">Remove</button>\
    </fieldset>\
    <br>\
    ';
    }

    $(function() {
      var addressesView = new AddressesView();

      $('#add-address-button').on('click', function(e) {
        e.preventDefault();
        addressesView.addAddress();
      });

      $(document).on('click', '.remove-address-button', function(e) {
        if (this === e.target) {
          e.preventDefault();
          var $this = $(this); // this button
          var $fieldset = $this.parent(); // fieldset
          addressesView.removeAddress($fieldset);
        }
      });

    });


フォームは、以下のように表示される。

.. figure:: ./images_Validation/validations-nested-collection1.png
  :width: 60%

「Add address」ボタンを2回押して、住所フォームを2件追加する。

.. figure:: ./images_Validation/validations-nested-collection2.png
  :width: 60%

このフォームに対して、すべての入力フィールドを未入力のまま送信すると、以下のようにエラーメッセージが表示される。

.. figure:: ./images_Validation/validations-nested-collection3.png
  :width: 60%


.. _ValidationGroupValidation:

バリデーションのグループ化
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
バリデーショングループを作成し、一つのフィールドに対して、グループごとに入力チェックルールを指定することができる。

前述の「新規ユーザー登録」の例で、\ ``age``\ フィールドに「成年であること」というルールを追加する。
「成年」かどうかは国によってルールが違うため、\ ``country``\ フィールドも追加する。

Bean Validationでグループを指定する場合、アノテーションの\ ``group``\ 属性に、グループを示す任意の\ ``java.lang.Class``\ オブジェクトを設定する。

ここでは、以下の3グループ(interface)を作成する。

.. tabularcolumns:: |p{0.50\linewidth}|p{0.50\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 50 50

   * - グループ
     - 成人条件
   * - \ ``Chinese``\
     - 18歳以上
   * - \ ``Japanese``\
     - 20歳以上
   * - \ ``Singaporean``\
     - 21歳以上


このグループをつかって、バリデーションを実行する例を示す。


* フォームクラス

  .. code-block:: java
    :emphasize-lines: 18-26,38-42

    package com.example.sample.app.validation;

    import java.io.Serializable;
    import java.util.List;

    import javax.validation.Valid;
    import javax.validation.constraints.Max;
    import javax.validation.constraints.Min;
    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;

    import org.hibernate.validator.constraints.Email;

    public class UserForm implements Serializable {

        private static final long serialVersionUID = 1L;

        // (1)
        public static interface Chinese {
        };

        public static interface Japanese {
        };

        public static interface Singaporean {
        };

        @NotNull
        @Size(min = 1, max = 20)
        private String name;

        @NotNull
        @Size(min = 1, max = 50)
        @Email
        private String email;

        @NotNull
        @Min.List({ // (2)
                @Min(value = 18, groups = Chinese.class), // (3)
                @Min(value = 20, groups = Japanese.class),
                @Min(value = 21, groups = Singaporean.class)
                })
        @Max(200)
        private Integer age;

        @NotNull
        @Size(min = 2, max = 2)
        private String country; // (4)

        // omitted setter/getter
    }


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | グループクラスを指定するために、各グループをインタフェースで定義する。
     * - | (2)
       - | 一つのフィールドに同じルールを複数指定するために、\ ``@Min.List``\ アノテーションを使用する。
         | 他のアノテーションを使用する場合も同様である。
     * - | (3)
       - | 各グループごとにルールを定義し、グループを指定するために、\ ``group``\ 属性に対象のグループクラスを指定する。
         | \ ``group``\ 属性を省略した場合、\ ``javax.validation.groups.Default``\ グループが使用される。
     * - | (4)
       - | グループを振り分けるための、フィールドを追加する。


* JSP

  JSPに大きな変更はない。

  .. code-block:: jsp
      :emphasize-lines: 16-22

      <form:form modelAttribute="userForm" method="post"
          class="form-horizontal"
          action="${pageContext.request.contextPath}/user/create">
          <form:label path="name" cssErrorClass="error-label">Name:</form:label>
          <form:input path="name" cssErrorClass="error-input" />
          <form:errors path="name" cssClass="error-messages" />
          <br>
          <form:label path="email" cssErrorClass="error-label">Email:</form:label>
          <form:input path="email" cssErrorClass="error-input" />
          <form:errors path="email" cssClass="error-messages" />
          <br>
          <form:label path="age" cssErrorClass="error-label">Age:</form:label>
          <form:input path="age" cssErrorClass="error-input" />
          <form:errors path="age" cssClass="error-messages" />
          <br>
          <form:label path="country" cssErrorClass="error-label">Country:</form:label>
          <form:select path="country" cssErrorClass="error-input">
              <form:option value="cn">China</form:option>
              <form:option value="jp">Japan</form:option>
              <form:option value="sg">Singapore</form:option>
          </form:select>
          <form:errors path="country" cssClass="error-messages" />
          <br>
          <form:button name="confirm">Confirm</form:button>
      </form:form>

* Controllerクラス

  \ ``@Validated``\ に、対象のグループを設定することで、バリデーションルールを変更できる。

  .. code-block:: java
      :emphasize-lines: 46-58

      package com.example.sample.app.validation;


      import javax.validation.groups.Default;

      import org.springframework.stereotype.Controller;
      import org.springframework.validation.BindingResult;
      import org.springframework.validation.annotation.Validated;
      import org.springframework.web.bind.annotation.ModelAttribute;
      import org.springframework.web.bind.annotation.RequestMapping;
      import org.springframework.web.bind.annotation.RequestMethod;

      import com.example.sample.app.validation.UserForm.Chinese;
      import com.example.sample.app.validation.UserForm.Japanese;
      import com.example.sample.app.validation.UserForm.Singaporean;

      @Controller
      @RequestMapping("user")
      public class UserController {

          @ModelAttribute
          public UserForm setupForm() {
              UserForm form = new UserForm();
              return form;
          }

          @RequestMapping(value = "create", method = RequestMethod.GET, params = "form")
          public String createForm() {
              return "user/createForm";
          }

          String createConfirm(UserForm form, BindingResult result) {
              if (result.hasErrors()) {
                  return "user/createForm";
              }
              return "user/createConfirm";
          }

          @RequestMapping(value = "create", method = RequestMethod.POST, params = {
                  "confirm",  /* (1) */ "country=cn" })
          public String createConfirmForChinese(@Validated({ /* (2) */ Chinese.class,
                  Default.class }) UserForm form, BindingResult result) {
              return createConfirm(form, result);
          }

          @RequestMapping(value = "create", method = RequestMethod.POST, params = {
                  "confirm", "country=jp" })
          public String createConfirmForJapanese(@Validated({ Japanese.class,
                  Default.class }) UserForm form, BindingResult result) {
              return createConfirm(form, result);
          }

          @RequestMapping(value = "create", method = RequestMethod.POST, params = {
                  "confirm", "country=sg" })
          public String createConfirmForSingaporean(@Validated({ Singaporean.class,
                  Default.class }) UserForm form, BindingResult result) {
              return createConfirm(form, result);
          }
      }


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | グループを振り分けるためのパラメータの条件を、\ ``param``\ 属性に追加する。
     * - | (2)
       - | \ ``age``\ フィールドの\ ``@Min``\ 以外のアノテーションは、\ ``Default``\ グループに属しているため、\ ``Default``\ の指定も必要である。



この例では、各入力値の組み合わせに対するチェック結果は、以下の表の通りである。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|p{0.40\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 20 20 40

   * - \ ``age``\ の値
     - \ ``country``\ の値
     - 入力チェック結果
     - エラーメッセージ
   * - | 17
     - | cn
     - | NG
     - | must be greater than or equal to 18
   * - |
     - | jp
     - | NG
     - | must be greater than or equal to 20
   * - |
     - | sg
     - | NG
     - | must be greater than or equal to 21
   * - | 18
     - | cn
     - | OK
     - |
   * - |
     - | jp
     - | NG
     - | must be greater than or equal to 20
   * - |
     - | sg
     - | NG
     - | must be greater than or equal to 21
   * - | 20
     - | cn
     - | OK
     - |
   * - |
     - | jp
     - | OK
     - |
   * - |
     - | sg
     - | NG
     - | must be greater than or equal to 21
   * - | 21
     - | cn
     - | OK
     - |
   * - |
     - | jp
     - | OK
     - |
   * - |
     - | sg
     - | OK
     - |

.. warning::

   このControllerの実装は、\ ``country``\ の値が、"cn"、"jp"、"sg"のいづれでもない場合のハンドリングが行われておらず、不十分である。
   \ ``country``\ の値が、想定外の場合に、400エラーが返却される。

次にチェック対象の国が増えたため、成人条件18歳以上をデフォルトルールとしたい場合を考える。

ルールは、以下のようになる。


.. tabularcolumns:: |p{0.50\linewidth}|p{0.50\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 50 50

   * - グループ
     - 成人条件
   * - \ ``Japanese``\
     - 20歳以上
   * - \ ``Singaporean``\
     - 21歳以上
   * - 上記以外の国(\ ``Default``\ )
     - 18歳以上


* フォームクラス

  \ ``Default``\ グループに意味を持たせるため、\ ``@Min``\ 以外のアノテーションにも、明示的に全グループを指定する必要がある。

  .. code-block:: java

    package com.example.sample.app.validation;

    import java.io.Serializable;
    import java.util.List;

    import javax.validation.Valid;
    import javax.validation.constraints.Max;
    import javax.validation.constraints.Min;
    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;
    import javax.validation.groups.Default;

    import org.hibernate.validator.constraints.Email;

    public class UserForm implements Serializable {

        private static final long serialVersionUID = 1L;

        public static interface Japanese {
        };

        public static interface Singaporean {
        };

        @NotNull(groups = { Default.class, Japanese.class, Singaporean.class }) // (1)
        @Size(min = 1, max = 20, groups = { Default.class, Japanese.class,
                Singaporean.class })
        private String name;

        @NotNull(groups = { Default.class, Japanese.class, Singaporean.class })
        @Size(min = 1, max = 50, groups = { Default.class, Japanese.class,
                Singaporean.class })
        @Email(groups = { Default.class, Japanese.class, Singaporean.class })
        private String email;

        @NotNull(groups = { Default.class, Japanese.class, Singaporean.class })
        @Min.List({
                @Min(value = 18, groups = Default.class), // (2)
                @Min(value = 20, groups = Japanese.class),
                @Min(value = 21, groups = Singaporean.class) })
        @Max(200)
        private Integer age;

        @NotNull(groups = { Default.class, Japanese.class, Singaporean.class })
        @Size(min = 2, max = 2, groups = { Default.class, Japanese.class,
                Singaporean.class })
        private String country;

        // omitted setter/getter
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | \ ``age``\ フィールドの\ ``@Min``\ 以外のアノテーションにも、全グループを設定する。
     * - | (2)
       - | \ ``Default``\ グループに対するルールを設定する。

* JSP

  JSPに変更はない

* Controllerクラス

  .. code-block:: java

    package com.example.sample.app.validation;

    import org.springframework.stereotype.Controller;
    import org.springframework.validation.BindingResult;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;

    import com.example.sample.app.validation.UserForm.Japanese;
    import com.example.sample.app.validation.UserForm.Singaporean;

    @Controller
    @RequestMapping("user")
    public class UserController {

        @ModelAttribute
        public UserForm setupForm() {
            UserForm form = new UserForm();
            return form;
        }

        @RequestMapping(value = "create", method = RequestMethod.GET, params = "form")
        public String createForm() {
            return "user/createForm";
        }

        String createConfirm(UserForm form, BindingResult result) {
            if (result.hasErrors()) {
                return "user/createForm";
            }
            return "user/createConfirm";
        }

        @RequestMapping(value = "create", method = RequestMethod.POST, params = { "confirm" })
        public String createConfirmForDefault(@Validated /* (1) */ UserForm form,
                BindingResult result) {
            return createConfirm(form, result);
        }

        @RequestMapping(value = "create", method = RequestMethod.POST, params = {
                "confirm", "country=jp" })
        public String createConfirmForJapanese(
                @Validated(Japanese.class)  /* (2) */ UserForm form, BindingResult result) {
            return createConfirm(form, result);
        }

        @RequestMapping(value = "create", method = RequestMethod.POST, params = {
                "confirm", "country=sg" })
        public String createConfirmForSingaporean(
                @Validated(Singaporean.class) UserForm form, BindingResult result) {
            return createConfirm(form, result);
        }
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | \ ``country``\ フィールド指定がない場合に、\ ``Default``\ グループが使用されるように設定する。
     * - | (2)
       - | \ ``country``\ フィールド指定がある場合に、\ ``Default``\ グループが含まれないように設定する。


バリデーショングループを使用する方法について、2パターン説明した。

前者は\ ``Default``\ グループをControllerクラスで使用し、後者は\ ``Default``\ グループをフォームクラスで使用した。


.. tabularcolumns:: |p{0.25\linewidth}|p{0.25\linewidth}|p{0.25\linewidth}|p{0.25\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 25 25 25 25

   * - パターン
     - メリット
     - デメリット
     - 使用の判断ポイント
   * - \ ``Default``\ グループをControllerクラスで使用
     - グループ化する必要のないルールは、\ ``group``\ 属性を設定する必要がない。
     - グループの全パターンを定義する必要があるので、グループパターンが多いと、定義が困難になる。
     - グループパターンが、数種類の場合に使用すべき(新規作成グループ、更新グループ、削除グループ等)
   * - \ ``Default``\ グループをフォームクラスで使用
     - デフォルトに属さないグループのみ定義すればよいため、パターンが多くても対応できる。
     - グループ化する必要のないルールにも、\ ``group``\ 属性を設定する必要があり、管理が煩雑になる。
     - グループパターンにデフォルト値を設定できる(グループの大多数に共通項がある)場合に使用すべき

\ **使用の判断ポイントのどちらにも当てはまらない場合は、Bean Validationの使用が不適切であることが考えられる。**\
設計を見直したうえで、Spring Validatorの使用または業務ロジックチェックでの実装を検討すること。


.. note::

 これまでの例ではバリデーショングループの切り替えは、リクエストパラメータ等、\ ``@RequestMapping``\ アノテーションで指定できるパラメータによって行った。
 この方法では認証オブジェクトが有する権限情報など、\ ``@RequestMapping``\ アノテーションでは扱えない情報でグループを切り替えることはできない。

 この場合は、\ ``@Validated``\ アノテーションを使用せず、\ ``org.springframework.validation.SmartValidator``\ を使用し、Controllerのハンドラメソッド内でグループを指定したバリデーションを行えばよい。

   .. code-block:: java

     @Controller
     @RequestMapping("user")
     public class UserController {

         @Inject
         SmartValidator smartValidator; // (1)

         // omitted

         @RequestMapping(value = "create", method = RequestMethod.POST, params = "confirm")
         public String createConfirm(/* (2) */ UserForm form, BindingResult result) {
             // (3)
             Class<?> validationGroup = Default.class;
             // logic to determine validation group
             // if (xxx) {
             //     validationGroup = Xxx.class;
             // }
             smartValidator.validate(form, result, validationGroup); // (4)
             if (result.hasErrors()) {
                 return "user/createForm";
             }
             return "user/createConfirm";
         }

     }

   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - 項番
        - 説明
      * - | (1)
        - | \ ``SmartValidator``\ をインジェクションする。\ ``SmartValidator``\ は\ ``<mvc:annotation-driven>``\ の設定が行われていれば使用できるため、別途Bean定義不要である。
      * - | (2)
        - | \ ``@Validated``\ アノテーションは使わない。
      * - | (3)
        - | バリデーショングループを決定する。
          | バリデーショングループを決定するロジックは、Helperクラスに委譲して、Controller内のロジックをシンプルな状態に保つことを推奨する。
      * - | (4)
        - | \ ``SmartValidator``\ の\ ``validate``\ メソッドを使用して、グループを指定したバリデーションを実行する。
          | グループの指定は可変長引数になっており、複数指定できる。

 基本的には、Controllerにロジックを書くべきではないため、\ ``@RequestMapping``\ の属性でルールを切り替えられるのであれば、\ ``SmartValidator``\ は使わない方がよい。


.. _Validation_correlation_check:

相関項目チェック
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
複数フィールドにまたがる相関項目チェックには、
Spring Validator(\ ``org.springframework.validation.Validator``\ インタフェースを実装した\ ``Validator``\ )、
または、Bean Validationを用いる。

それぞれ説明するが、先にそれぞれの特徴と推奨する使い分けを述べる。


.. tabularcolumns:: |p{0.20\linewidth}|p{0.40\linewidth}|p{0.40\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 40 40


   * - 方式
     - 特徴
     - 用途
   * - | Spring Validator
     - | 特定のクラスに対する入力チェックの作成が容易である。
       | Controllerでの利用が不便。
     - | 特定のフォームに依存した業務要件固有の入力チェック実装
   * - | Bean Validation
     - | 入力チェックの作成はSpring Validatorほど容易でない。
       | Controllerでの利用が容易。
     - | 特定のフォームに依存しない、開発プロジェクト共通の入力チェック実装



Spring Validatorによる相関項目チェック実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 「パスワードリセット」処理を例に実装方法を説明する。
| 以下のルールを実装する。ここでは「パスワードリセット」のフォームに以下のチェックルールを設ける。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.30\linewidth}|p{0.30\linewidth}|p{0.20\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 30 30 20


   * - フィールド名
     - 型
     - ルール
     - 説明
   * - | password
     - | ``java.lang.String``
     - | 入力必須
       | 8文字以上
       | \ **confirmPasswordと同じ値であること**\
     - | パスワード
   * - | confirmPassword
     - | ``java.lang.String``
     - | 特になし
     - | 確認用パスワード

「confirmPasswordと同じ値であること」というルールは\ ``password``\ フィールドと\ ``confirmPassword``\ フィールドの両方の情報が必要であるため、相関項目チェックルールである。

* フォームクラス

  相関項目チェックルール以外は、これまで通りBean Validationのアノテーションで実装する。

  .. code-block:: java

    package com.example.sample.app.validation;

    import java.io.Serializable;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;

    public class PasswordResetForm implements Serializable {
        private static final long serialVersionUID = 1L;

        @NotNull
        @Size(min = 8)
        private String password;

        private String confirmPassword;

        // omitted setter/getter
    }

  .. note::

    パスワードは、通常ハッシュ化してデータベースに保存するため、最大値のチェックは行わなくても良い。

* Validatorクラス

  \ ``org.springframework.validation.Validator``\ インタフェースを実装して、相関項目チェックルールを実現する。

  .. code-block:: java

    package com.example.sample.app.validation;

    import org.springframework.stereotype.Component;
    import org.springframework.validation.Errors;
    import org.springframework.validation.Validator;

    @Component // (1)
    public class PasswordEqualsValidator implements Validator {

        @Override
        public boolean supports(Class<?> clazz) {
            return PasswordResetForm.class.isAssignableFrom(clazz); // (2)
        }

        @Override
        public void validate(Object target, Errors errors) {

            if (errors.hasFieldErrors("password")) { // (3)
                return;
            }

            PasswordResetForm form = (PasswordResetForm) target;
            String password = form.getPassword();
            String confirmPassword = form.getConfirmPassword();

            if (!password.equals(confirmPassword)) { // (4)
                errors.rejectValue(/* (5) */ "password",
                /* (6) */ "PasswordEqualsValidator.passwordResetForm.password",
                /* (7) */ "password and confirm password must be same.");
            }
        }
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | \ ``@Component``\ を付与し、Validatorをコンポーネントスキャン対象にする。
     * - | (2)
       - | このValidatorのチェック対象であるかどうかを判別する。ここでは、\ ``PasswordResetForm``\ クラスをチェック対象とする。
     * - | (3)
       - | 単項目チェック時に対象フィールドでエラーが発生している場合は、このValidatorで相関チェックは行わない。
         | 相関チェックを必ず行う必要がある場合は、この判定ロジックは不要である。
     * - | (4)
       - | チェックロジックを実装する。
     * - | (5)
       - | エラー対象のフィールド名を指定する。
     * - | (6)
       - | エラーメッセージのコード名を指定する。ここではコードを、
         | "バリデータ名.フォーム属性名.プロパティ名"
         | とする。メッセージ定義は\ :ref:`Validation_message_in_application_messages`\ を参照されたい。
     * - | (7)
       - | エラーメッセージをコードで解決できなかった場合に使用する、デフォルトメッセージを設定する。

  .. note::

    Spring Validator実装クラスは、使用するControllerと同じパッケージに配置することを推奨する。

* Controllerクラス

  .. code-block:: java

    package com.example.sample.app.validation;

    import javax.inject.Inject;

    import org.springframework.stereotype.Controller;
    import org.springframework.validation.BindingResult;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.WebDataBinder;
    import org.springframework.web.bind.annotation.InitBinder;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;

    @Controller
    @RequestMapping("password")
    public class PasswordResetController {
        @Inject
        PasswordEqualsValidator passwordEqualsValidator; // (1)

        @ModelAttribute
        public PasswordResetForm setupForm() {
            return new PasswordResetForm();
        }

        @InitBinder
        public void initBinder(WebDataBinder binder) {
            binder.addValidators(passwordEqualsValidator); // (2)
        }

        @RequestMapping(value = "reset", method = RequestMethod.GET, params = "form")
        public String resetForm() {
            return "password/resetForm";
        }

        @RequestMapping(value = "reset", method = RequestMethod.POST)
        public String reset(@Validated PasswordResetForm form, BindingResult result) { // (3)
            if (result.hasErrors()) {
                return "password/resetForm";
            }
            return "redirect:/password/reset?complete";
        }

        @RequestMapping(value = "reset", method = RequestMethod.GET, params = "complete")
        public String resetComplete() {
            return "password/resetComplete";
        }
    }


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | 使用するSpring Validatorを、インジェクションする。
     * - | (2)
       - | \ ``@InitBinder``\ アノテーションがついたメソッド内で、\ ``WebDataBinder.addValidators``\ メソッドにより、Validatorを追加する。
         | これにより、\ ``@Validated``\ アノテーションでバリデーションをする際に、追加したValidatorも呼び出される。
     * - | (3)
       - | 入力チェックの実装は、これまで通りである。

* JSP

  JSPに特筆すべき点はない。

  .. code-block:: jsp

    <!DOCTYPE html>
    <html>
    <%-- WEB-INF/views/password/resetForm.jsp --%>
    <head>
    <style type="text/css">
    /* omitted */
    </style>
    </head>
    <body>
        <form:form modelAttribute="passwordResetForm" method="post"
            class="form-horizontal"
            action="${pageContext.request.contextPath}/password/reset">
            <form:label path="password" cssErrorClass="error-label">Password:</form:label>
            <form:password path="password" cssErrorClass="error-input" />
            <form:errors path="password" cssClass="error-messages" />
            <br>
            <form:label path="confirmPassword" cssErrorClass="error-label">Password (Confirm):</form:label>
            <form:password path="confirmPassword"
                cssErrorClass="error-input" />
            <form:errors path="confirmPassword" cssClass="error-messages" />
            <br>
            <form:button>Reset</form:button>
        </form:form>
    </body>
    </html>


\ ``password``\ フィールドと、\ ``confirmPassword``\ フィールドに、別の値を入力してフォームを送信した場合は、以下のようにエラーメッセージが表示される。

.. figure:: ./images_Validation/validations-correlation-check1.png
  :width: 60%


.. note::

  \ ``<form:password>``\ タグを使用すると、再表示時に、データがクリアされる。

.. _Validation_how_to_cross-field_validation_for_multi_field_highlight:
.. note::

   相関チェック対象の複数フィールドに対してエラー情報を設定することも可能である。
   ただし、必ずエラーメッセージの表示とスタイル適用がセットで行われ、いずれか片方のみを行うことはできない。

   相関チェックエラーとなった両方のフィールドにスタイル適用したいが、エラーメッセージは1つだけ表示したいような場合は、
   エラーメッセージに空文字を設定することで実現することが可能である。
   以下に、\ ``password``\ フィールドと\ ``confirmPassword``\ フィールドにスタイルを適用し、\ ``password``\ フィールドのみにエラーメッセージを表示する例を示す。

     .. code-block:: java

       package com.example.sample.app.validation;

       import org.springframework.stereotype.Component;
       import org.springframework.validation.Errors;
       import org.springframework.validation.Validator;

       @Component
       public class PasswordEqualsValidator implements Validator {

           @Override
           public boolean supports(Class<?> clazz) {
               return PasswordResetForm.class.isAssignableFrom(clazz);
           }

           @Override
           public void validate(Object target, Errors errors) {

               // omitted
               if (!password.equals(confirmPassword)) {
                   // register a field error for password
                   errors.rejectValue("password",
                          "PasswordEqualsValidator.passwordResetForm.password",
                          "password and confirm password must be same.");

                   // register a field error for confirmPassword
                   errors.rejectValue("confirmPassword", // (1)
                             "PasswordEqualsValidator.passwordResetForm.confirmPassword", // (2)
                             ""); // (3)
               }
           }
       }

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90
        :class: longtable

        * - 項番
          - 説明
        * - | (1)
          - | \ ``confirmPassword``\ フィールドのエラーを登録する。
        * - | (2)
          - | エラーメッセージのコード名を指定する。この際、対応するエラーメッセージに空文字を指定する。
            | メッセージ定義は\ :ref:`Validation_message_in_application_messages`\ を参照されたい。
        * - | (3)
          - | エラーメッセージをコードで解決できなかった場合に使用する、デフォルトメッセージを設定する。
            | 上記の例では空文字を設定している。


.. note::

   一つのControllerで複数のフォームを扱う場合は、Validatorの対象を限定するために、\ ``@InitBinder("xxx")``\ でモデル名を指定する必要がある。

     .. code-block:: java

       @Controller
       @RequestMapping("xxx")
       public class XxxController {
           // omitted
           @ModelAttribute("aaa")
           public AaaForm() {
               return new AaaForm();
           }

           @ModelAttribute("bbb")
           public BbbForm() {
               return new BbbForm();
           }

           @InitBinder("aaa")
           public void initBinderForAaa(WebDataBinder binder) {
               // add validators for AaaForm
               binder.addValidators(aaaValidator);
           }

           @InitBinder("bbb")
           public void initBinderForBbb(WebDataBinder binder) {
               // add validators for BbbForm
               binder.addValidators(bbbValidator);
           }
           // omitted
       }

.. note::

   相関項目チェックルールのチェック内容をバリデーショングループに応じて変更したい場合（例えば、特定のバリデーショングループが指定された場合だけ相関項目チェックを実施したい場合など）は、 \ ``org.springframework.validation.Validator``\ インターフェイスを実装する代わりに、 \ ``org.springframework.validation.SmartValidator``\ インターフェイスを実装し、validateメソッド内で処理を切り替えるとよい。

     .. code-block:: java

       package com.example.sample.app.validation;

       import org.apache.commons.lang3.ArrayUtils;
       import org.springframework.stereotype.Component;
       import org.springframework.validation.Errors;
       import org.springframework.validation.SmartValidator;

       @Component
       public class PasswordEqualsValidator implements SmartValidator { // Implements SmartValidator instead of Validator interface

           @Override
           public boolean supports(Class<?> clazz) {
               return PasswordResetForm.class.isAssignableFrom(clazz);
           }

           @Override
           public void validate(Object target, Errors errors) {
               validate(target, errors, new Object[] {});
           }

           @Override
           public void validate(Object target, Errors errors, Object... validationHints) {
               // Check validationHints(groups) and apply validation logic only when 'Update.class' is specified
               if (ArrayUtils.contains(validationHints, Update.class)) {
                   PasswordResetForm form = (PasswordResetForm) target;
                   String password = form.getPassword();
                   String confirmPassword = form.getConfirmPassword();

                   // omitted...
               }
           }
       }

Bean Validationによる相関項目チェック実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Bean Validationによって、相関項目チェックの実装するためには、独自バリデーションルールの追加を行う必要がある。

:ref:`Validation_custom_constraint`\ にて説明する。


.. _Validation_message_def:

エラーメッセージの定義
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
入力チェックエラーメッセージを変更する方法を説明する。

Spring MVCによるBean Validationのエラーメッセージは、以下の順で解決される。

#. | \ ``org.springframework.context.MessageSource``\ に定義されているメッセージの中に、ルールに合致するものがあればそれをエラーメッセージとして使用する (Springのルール)。
   | Springのデフォルトのルールについては、「`DefaultMessageCodesResolverのJavaDoc <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/validation/DefaultMessageCodesResolver.html>`_」を参照されたい。
#. 1.でメッセージが見つからない場合、アノテーションの\ ``message``\ 属性に、指定されたメッセージからエラーメッセージを取得する (Bean Validationのルール)

  #. \ ``message``\ 属性に指定されたメッセージが、"{メッセージキー}"形式でない場合、そのテキストをエラーメッセージとして使用する。
  #. \ ``message``\ 属性に指定されたメッセージが、"{メッセージキー}"形式の場合、クラスパス直下のValidationMessages.propertiesから、メッセージキーに対応するメッセージを探す。

    #. メッセージキーに対応するメッセージが定義されている場合は、そのメッセージを使用する
    #. メッセージキーに対応するメッセージが定義されていない場合は、"{メッセージキー}"をそのままエラーメッセージとして使用する

基本的にエラーメッセージは、propertiesファイルに定義することを推奨する。

定義する箇所は、以下の2パターン存在する。

* \ ``org.springframework.context.MessageSource``\ が読み込むpropertiesファイル
* \ クラスパス直下のValidationMessages.properties

以下の説明では、applicationContext.xmlに次の設定があることを前提とし、前者を"application-messages.properties"、後者を"ValidationMessages.properties"と呼ぶ。

.. code-block:: xml

    <bean id="messageSource"
        class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basenames">
            <list>
                <value>i18n/application-messages</value>
            </list>
        </property>
    </bean>


.. figure:: ./images_Validation/validations-message-properties-position-image.png
  :width: 40%

.. warning::

    \ ``ValidationMessages.properties``\ ファイルは、クラスパスの直下に複数存在させてはいけない。

    クラスパスの直下に複数の\ ``ValidationMessages.properties``\ ファイルが存在する場合、
    いずれか１つのファイルが読み込まれ、他のファイルが読み込まれないため、適切なメッセージが表示されない可能性がある。

    * マルチプロジェクト構成を採用する場合は、\ ``ValidationMessages.properties``\ ファイルを複数のプロジェクトに配置しないように注意すること。
    * Bean Validation用の共通部品をjarファイルとして配布する際に、\ ``ValidationMessages.properties``\ ファイルをjarファイルの中に含めないように注意すること。

    なお、version 1.0.2.RELEASE以降の `ブランクプロジェクト <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank>`_ \ からプロジェクトを生成した場合は、
    \ ``xxx-web/src/main/resources``\ の直下に\ ``ValidationMessages.properties``\ が格納されている。

|


本ガイドラインでは、以下のように定義を分けることを推奨する。

.. tabularcolumns:: |p{0.50\linewidth}|p{0.50\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 50 50


   * - プロパティファイル名
     - 定義する内容
   * - | ValidationMessages.properties
     - | システムで定めたBean Validationのデフォルトエラーメッセージ
   * - | application-messages.properties
     - | 個別で上書きしたいBean Validationのエラーメッセージ
       | Spring Validatorで実装した入力チェックのエラーメッセージ

ValidationMessages.propertiesを用意しない場合は、\ :ref:`Hibernate Validatorが用意するデフォルトメッセージ <Validation_default_message_in_hibernate_validator>`\ が使用される。

\ ``MessageSource``\ と連携することで、日本語メッセージをNative to Asciiせずに直接扱うことができる。
詳細は、\ :ref:`Validation_without_native2ascii`\ を参照されたい。


.. _Validation_message_in_validationmessages:

ValidationMessages.propertiesに定義するメッセージ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
クラスパス直下(通常src/main/resources)のValidationMessages.properties内の、
Bean Validationのアノテーションの\ ``message``\ 属性に指定されたメッセージキーに対して、メッセージを定義する。


\ :ref:`Validation_basic_validation`\ の初めに使用した、以下のフォームを用いて説明する。


* フォームクラス(再掲)

  .. code-block:: java

    public class UserForm implements Serializable {

        @NotNull
        @Size(min = 1, max = 20)
        private String name;

        @NotNull
        @Size(min = 1, max = 50)
        @Email
        private String email;

        @NotNull
        @Min(0)
        @Max(200)
        private Integer age;

        // omitted getter/setter
    }

* ValidationMessages.properties

  \ ``@NotNull``\ , \ ``@Size``\ , \ ``@Min``\ , \ ``@Max``\ , \ ``@Email``\ のエラーメッセージを変更する。

  .. code-block:: properties

    javax.validation.constraints.NotNull.message=is required.
    # (1)
    javax.validation.constraints.Size.message=size is not in the range {min} through {max}.
    javax.validation.constraints.Min.message=can not be less than {value}.
    javax.validation.constraints.Max.message=can not be greater than {value}.
    org.hibernate.validator.constraints.Email.message=is an invalid e-mail address.

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | アノテーションに指定した属性値は、\ ``{属性名}``\ で埋め込むことができる。

この設定を加えた状態で、すべての入力フィールドを未入力のままフォームを送信すると、以下のように変更したエラーメッセージが、表示される。

.. figure:: ./images_Validation/validations-customize-message1.png
  :width: 60%

.. warning::

  Bean Validation標準のアノテーションやHibernate Validator独自のアノテーションには\ ``message``\ 属性に\ ``{アノテーションのFQCN.message}``\ という値が設定されているため、

    .. code-block:: properties

      アノテーションのFQCN.message=メッセージ

  という形式でプロパティファイルにメッセージを定義すればよいが、すべてのアノテーションが、この形式になっているわけではないので、
  対象のアノテーションのJavadocまたはソースコードを確認すること。


エラーメッセージに、フィールド名を含める場合は、以下のように、メッセージに\ ``{0}``\ を加える。

* ValidationMessages.properties

  \ ``@NotNull``\ 、\ ``@Size``\ 、\ ``@Min``\ 、\ ``@Max``\ 、\ ``@Email``\ のエラーメッセージを変更する。

  .. code-block:: properties

    javax.validation.constraints.NotNull.message="{0}" is required.
    javax.validation.constraints.Size.message=The size of "{0}" is not in the range {min} through {max}.
    javax.validation.constraints.Min.message="{0}" can not be less than {value}.
    javax.validation.constraints.Max.message="{0}" can not be greater than {value}.
    org.hibernate.validator.constraints.Email.message="{0}" is an invalid e-mail address.

エラーメッセージは、以下のように変更される。

.. figure:: ./images_Validation/validations-customize-message2.png
  :width: 60%

このままでは、フォームクラスのプロパティ名が表示されてしまい、ユーザーフレンドリではない。
適切なフィールド名を表示したい場合は、\ **application-messages.propertiesに**\

.. code-block:: properties

  フォームのプロパティ名=表示するフィールド名

形式でフィールド名を定義すればよい。

これまでの例に、以下の設定を追加する。

* application-messages.properties

  .. code-block:: properties

    name=Name
    email=Email
    age=Age

エラーメッセージは、以下のように変更される。

.. figure:: ./images_Validation/validations-customize-message3.png
  :width: 60%


.. note::

  \ ``{0}``\ でフィールド名を埋め込めむのは、Bean Validationの機能ではなく、Springの機能である。
  したがって、フィールド名変更の設定は、Spring管理下のapplication-messages.properties(\ ``ResourceBundleMessageSource``\ )に定義する必要がある。

.. tip::

    Bean Validation 1.1より、
    :file:`ValidationMessages.properties` に指定するメッセージの中にExpression Language(以降、「EL式」と呼ぶ)を使用する事ができるようになった。
    Hibernate Validator 5.xでは、Expression Language 2.2以上をサポートしている。

    実行可能なEL式のバージョンは、アプリケーションサーバのバージョンによって異なる。
    そのため、EL式を使用する場合は、**アプリケーションサーバがサポートしているEL式のバージョンを確認した上で使用すること。**

    以下に、Hibernate Validatorがデフォルトで用意している :file:`ValidationMessages.properties` に定義されているメッセージを例に、EL式の使用例を示す。

     .. code-block:: properties

        # ...
        # (1)
        javax.validation.constraints.DecimalMax.message  = must be less than ${inclusive == true ? 'or equal to ' : ''}{value}
        # ...

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90

        * - 項番
          - 説明
        * - | (1)
          - メッセージの中の 「\ ``${inclusive == true ? 'or equal to ' : ''}``\」の部分がEL式である。

            上記のメッセージ定義から実際に生成されるメッセージのパターンは、

            * must be less than or equal to {value}
            * must be less than {value}

            の2パターンとなる。(\ ``{value}``\ の部分には、\ ``@DecimalMax``\ アノテーションの \ ``value``\ 属性に指定した値が埋め込まれる)

            前者は\ ``@DecimalMax``\ アノテーションの \ ``inclusive``\ 属性に \ ``true``\ を指定した場合(又は指定しなかった場合)、
            後者は\ ``@DecimalMax``\ アノテーションの \ ``inclusive``\ 属性に \ ``false``\ を指定した場合に生成される。

            Bean ValidationにおけるEL式の扱いについては、
            \ `Hibernate Validator Reference Guide(Interpolation with message expressions) <http://docs.jboss.org/hibernate/validator/5.2/reference/en-US/html/ch04.html#section-interpolation-with-message-expressions>`_\ を参照されたい。

    また、:file:`ValidationMessages.properties` に指定するメッセージに \ ``${validatedValue}``\ を使用することで、エラーメッセージにチェック対象の値を含むことができる。

    以下に、 \ ``${validatedValue}``\ の使用例を示す。

     .. code-block:: properties

        # ...
        # (1)
        javax.validation.constraints.Pattern.message = The value entered "${validatedValue}" is invalid.
        # ...


     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90
        :class: longtable

        * - 項番
          - 説明
        * - | (1)

          - 上記のメッセージ定義から実際に生成されるメッセージは、 \ ``${validatedValue}``\ の部分にフォームに入力した値が埋め込まれる。
            入力値に機密情報を含む場合、機密情報がメッセージに表示されないようにするため、 \ ``${validatedValue}``\ を使用しないように注意すること。

            詳細については、\ `Hibernate Validator Reference Guide(Interpolation with message expressions) <http://docs.jboss.org/hibernate/validator/5.2/reference/en-US/html/ch04.html#section-interpolation-with-message-expressions>`_\ を参照されたい。


.. _Validation_message_in_application_messages:

application-messages.propertiesに定義するメッセージ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ValidationMessages.propertiesでシステムで利用するデフォルトのメッセージを定義したが、
画面によっては、デフォルトメッセージから変更したい場合が出てくる。

その場合、application-messages.propertiesに、以下の形式でメッセージを定義する。


.. code-block:: properties

  アノテーション名.フォーム属性名.プロパティ名=対象のメッセージ


\ :ref:`Validation_message_in_validationmessages`\ の設定がある前提で、以下の設定で\ ``email``\ と\ ``age``\ フィールドのメッセージを上書きする。

* application-messages.properties

  .. code-block:: properties

    # override messages
    # for email field
    Size.userForm.email=The size of "{0}" must be between {2} and {1}.
    # for age field
    NotNull.userForm.age="{0}" is compulsory.
    Min.userForm.age="{0}" must be greater than or equal to {1}.
    Max.userForm.age="{0}" must be less than or equal to {1}.

    # filed names
    name=Name
    email=Email
    age=Age

アノテーションの属性値は、\ ``{1}``\ 以降に埋め込まれる。なお、属性値のインデックス位置は、アノテーションの属性名のアルファベット順(昇順)となる。

例えば、\ ``@Size``\ のインデックス位置は、

* \ ``{0}``\  : プロパティ名 (物理名又は論理名)
* \ ``{1}``\  : \ ``max``\ 属性の値
* \ ``{2}``\  : \ ``min``\ 属性の値

となる。
仕様の詳細については \ `SpringValidatorAdapterのJavaDoc <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/validation/beanvalidation/SpringValidatorAdapter.html#getArgumentsForConstraint-java.lang.String-java.lang.String-javax.validation.metadata.ConstraintDescriptor->`_\
を参照されたい。

エラーメッセージは以下のように変更される。

.. figure:: ./images_Validation/validations-customize-message4.png
  :width: 60%


.. note::

  application-messages.propertiesのメッセージキーの形式は、\ `これ以外にも用意されている <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/validation/DefaultMessageCodesResolver.html>`_\ が、
  デフォルトメッセージを一部上書きする目的で使用するのであれば、基本的に、\ ``アノテーション名.フォーム属性名.プロパティ名``\ 形式でよい。

|

.. _Validation_custom_constraint:

How to extend
--------------------------------------------------------------------------------

Bean Validationは標準で用意されているチェックルール以外に、独自ルール用アノテーションを作成する仕組みをもつ。

作成方法は大きく分けて、以下の観点で分かれる。

* 既存ルールの組み合わせ
* 新規ルールの作成

基本的には、以下の雛形を使用して、ルール毎にアノテーションを作成する。

.. code-block:: java

  package com.example.common.validation;

  import java.lang.annotation.Documented;
  import java.lang.annotation.Retention;
  import java.lang.annotation.Target;
  import javax.validation.Constraint;
  import javax.validation.Payload;
  import static java.lang.annotation.ElementType.ANNOTATION_TYPE;
  import static java.lang.annotation.ElementType.CONSTRUCTOR;
  import static java.lang.annotation.ElementType.FIELD;
  import static java.lang.annotation.ElementType.METHOD;
  import static java.lang.annotation.ElementType.PARAMETER;
  import static java.lang.annotation.RetentionPolicy.RUNTIME;

  @Documented
  @Constraint(validatedBy = {})
  @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
  @Retention(RUNTIME)
  public @interface Xxx {
      String message() default "{com.example.common.validation.Xxx.message}";

      Class<?>[] groups() default {};

      Class<? extends Payload>[] payload() default {};

      @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
      @Retention(RUNTIME)
      @Documented
      public @interface List {
          Xxx[] value();
      }
  }



.. _Validation_convine_existing_constraint:

既存ルールを組み合わせたBean Validationアノテーションの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

システム共通で、

* 文字列は半角英数字の文字種に限定したい
* 数値は正の数に限定したい

| または、ドメイン共通で、

* 「ユーザーID」は、4文字以上20文字以下の半角英字に制限したい
* 「年齢」は、1歳以上150歳以下に制限したい

| という制約がある場合を考える。
| これらは既存ルールの\ ``@Pattern``\ 、\ ``@Size``\ 、\ ``@Min``\ 、\ ``@Max``\ 等を組み合わせることでも実現できるが、
| 同じルールを複数の箇所で使用すると、設定内容が分散してしまい、メンテナンス性が悪化する。

複数のルールを組み合わせて一つのルールを作成することができる。
独自アノテーションを作成すると、正規表現パターンや、最大値・最小値などの値を共通化できるだけでなく、エラーメッセージも共通化できるというメリットがある。
これにより、再利用性や保守性が高まる。複数のルールの組み合わせではなくても、一つのルールの属性を特定するだけでも効果的である。

以下に、実装例を示す。

* 半角英数字の文字種に限定する\ ``@Alphanumeric``\ アノテーションの実装例

  .. code-block:: java
    :emphasize-lines: 22-23,25

    package com.example.common.validation;

    import java.lang.annotation.Documented;
    import java.lang.annotation.Retention;
    import java.lang.annotation.Target;
    import javax.validation.Constraint;
    import javax.validation.Payload;
    import javax.validation.ReportAsSingleViolation;
    import javax.validation.constraints.Pattern;

    import static java.lang.annotation.ElementType.ANNOTATION_TYPE;
    import static java.lang.annotation.ElementType.CONSTRUCTOR;
    import static java.lang.annotation.ElementType.FIELD;
    import static java.lang.annotation.ElementType.METHOD;
    import static java.lang.annotation.ElementType.PARAMETER;
    import static java.lang.annotation.RetentionPolicy.RUNTIME;

    @Documented
    @Constraint(validatedBy = {})
    @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
    @Retention(RUNTIME)
    @ReportAsSingleViolation // (1)
    @Pattern(regexp = "[a-zA-Z0-9]*") // (2)
    public @interface AlphaNumeric {
        String message() default "{com.example.common.validation.AlphaNumeric.message}"; // (3)

        Class<?>[] groups() default {};

        Class<? extends Payload>[] payload() default {};

        @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
        @Retention(RUNTIME)
        @Documented
        public @interface List {
            AlphaNumeric[] value();
        }
    }


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | エラーメッセージをまとめ、エラー時はこのアノテーションによるメッセージだけを変えるようにする。
     * - | (2)
       - | このアノテーションにより使用されるルールを定義する。
     * - | (3)
       - | エラーメッセージのデフォルト値を定義する。

* 正の数に限定する\ ``@NotNegative``\ アノテーションの実装例

  .. code-block:: java
    :emphasize-lines: 22-23,25

    package com.example.common.validation;

    import java.lang.annotation.Documented;
    import java.lang.annotation.Retention;
    import java.lang.annotation.Target;
    import javax.validation.Constraint;
    import javax.validation.Payload;
    import javax.validation.ReportAsSingleViolation;
    import javax.validation.constraints.Min;

    import static java.lang.annotation.ElementType.ANNOTATION_TYPE;
    import static java.lang.annotation.ElementType.CONSTRUCTOR;
    import static java.lang.annotation.ElementType.FIELD;
    import static java.lang.annotation.ElementType.METHOD;
    import static java.lang.annotation.ElementType.PARAMETER;
    import static java.lang.annotation.RetentionPolicy.RUNTIME;

    @Documented
    @Constraint(validatedBy = {})
    @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
    @Retention(RUNTIME)
    @ReportAsSingleViolation
    @Min(value = 0)
    public @interface NotNegative {
        String message() default "{com.example.common.validation.NotNegative.message}";

        Class<?>[] groups() default {};

        Class<? extends Payload>[] payload() default {};

        @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
        @Retention(RUNTIME)
        @Documented
        public @interface List {
            NotNegative[] value();
        }
    }


* 「ユーザーID」のフォーマットを規定する\ ``@UserId``\ アノテーションの実装例

  .. code-block:: java
    :emphasize-lines: 23-25,27

    package com.example.sample.domain.validation;

    import java.lang.annotation.Documented;
    import java.lang.annotation.Retention;
    import java.lang.annotation.Target;
    import javax.validation.Constraint;
    import javax.validation.Payload;
    import javax.validation.ReportAsSingleViolation;
    import javax.validation.constraints.Pattern;
    import javax.validation.constraints.Size;

    import static java.lang.annotation.ElementType.ANNOTATION_TYPE;
    import static java.lang.annotation.ElementType.CONSTRUCTOR;
    import static java.lang.annotation.ElementType.FIELD;
    import static java.lang.annotation.ElementType.METHOD;
    import static java.lang.annotation.ElementType.PARAMETER;
    import static java.lang.annotation.RetentionPolicy.RUNTIME;

    @Documented
    @Constraint(validatedBy = {})
    @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
    @Retention(RUNTIME)
    @ReportAsSingleViolation
    @Size(min = 4, max = 20)
    @Pattern(regexp = "[a-z]*")
    public @interface UserId {
        String message() default "{com.example.sample.domain.validation.UserId.message}";

        Class<?>[] groups() default {};

        Class<? extends Payload>[] payload() default {};

        @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
        @Retention(RUNTIME)
        @Documented
        public @interface List {
            UserId[] value();
        }
    }

* 「年齢」の制限を規定する\ ``@Age``\ アノテーションの実装例

  .. code-block:: java
    :emphasize-lines: 23-25,27

    package com.example.sample.domain.validation;

    import java.lang.annotation.Documented;
    import java.lang.annotation.Retention;
    import java.lang.annotation.Target;
    import javax.validation.Constraint;
    import javax.validation.Payload;
    import javax.validation.ReportAsSingleViolation;
    import javax.validation.constraints.Max;
    import javax.validation.constraints.Min;

    import static java.lang.annotation.ElementType.ANNOTATION_TYPE;
    import static java.lang.annotation.ElementType.CONSTRUCTOR;
    import static java.lang.annotation.ElementType.FIELD;
    import static java.lang.annotation.ElementType.METHOD;
    import static java.lang.annotation.ElementType.PARAMETER;
    import static java.lang.annotation.RetentionPolicy.RUNTIME;

    @Documented
    @Constraint(validatedBy = {})
    @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
    @Retention(RUNTIME)
    @ReportAsSingleViolation
    @Min(1)
    @Max(150)
    public @interface Age {
        String message() default "{com.example.sample.domain.validation.Age.message}";

        Class<?>[] groups() default {};

        Class<? extends Payload>[] payload() default {};

        @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
        @Retention(RUNTIME)
        @Documented
        public @interface List {
            Age[] value();
        }
    }


  .. note::

    1つのアノテーションに複数のルールを設定した場合、それらのAND条件が複合ルールとなる。
    Hibernate Validatorでは、OR条件を実現するための\ ``@ConstraintComposition``\ アノテーションが用意されている。
    詳細は、\ `Hibernate Validatorのドキュメント <http://docs.jboss.org/hibernate/validator/5.2/reference/en-US/html/ch11.html#section-boolean-constraint-composition>`_\ を参照されたい。

.. _Validation_implement_new_constraint:

新規ルールを実装したBean Validationアノテーションの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ ``javax.validation.ConstraintValidator``\ インタフェースを実装し、そのValidatorを使用するアノテーションを作成することで、任意のルールを作成することができる。

用途としては、以下の3通りが挙げられる。

* 既存のルールの組み合わせでは表現できないルール
* 相関項目チェックルール
* 業務ロジックチェック

.. _Validation_cannot_expressed_existing:

既存のルールの組み合わせでは表現できないルール
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\ ``@Pattern``\ 、\ ``@Size``\ 、\ ``@Min``\ 、\ ``@Max``\ 等を組み合わせても表現できないルールは、\ ``javax.validation.ConstraintValidator``\ 実装クラスに記述する。

例として、ISBN(International Standard Book Number)-13の形式をチェックするルールを挙げる。


* アノテーション

  .. code-block:: java
    :emphasize-lines: 16

    package com.example.common.validation;

    import java.lang.annotation.Documented;
    import java.lang.annotation.Retention;
    import java.lang.annotation.Target;
    import javax.validation.Constraint;
    import javax.validation.Payload;
    import static java.lang.annotation.ElementType.ANNOTATION_TYPE;
    import static java.lang.annotation.ElementType.CONSTRUCTOR;
    import static java.lang.annotation.ElementType.FIELD;
    import static java.lang.annotation.ElementType.METHOD;
    import static java.lang.annotation.ElementType.PARAMETER;
    import static java.lang.annotation.RetentionPolicy.RUNTIME;

    @Documented
    @Constraint(validatedBy = { ISBN13Validator.class }) // (1)
    @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
    @Retention(RUNTIME)
    public @interface ISBN13 {
        String message() default "{com.example.common.validation.ISBN13.message}";

        Class<?>[] groups() default {};

        Class<? extends Payload>[] payload() default {};

        @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
        @Retention(RUNTIME)
        @Documented
        public @interface List {
            ISBN13[] value();
        }
    }


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | このアノテーションを使用したときに実行される\ ``ConstraintValidator``\ を指定する。複数指定することができる。


* Validator

  .. code-block:: java

    package com.example.common.validation;

    import javax.validation.ConstraintValidator;
    import javax.validation.ConstraintValidatorContext;

    public class ISBN13Validator implements ConstraintValidator<ISBN13, String> { // (1)

        @Override
        public void initialize(ISBN13 constraintAnnotation) { // (2)
        }

        @Override
        public boolean isValid(String value, ConstraintValidatorContext context) { // (3)
            if (value == null) {
                return true; // (4)
            }
            return isISBN13Valid(value); // (5)
        }

        // This logic is written in http://en.wikipedia.org/wiki/International_Standard_Book_Number
        static boolean isISBN13Valid(String isbn) {
            if (isbn.length() != 13) {
                return false;
            }
            int check = 0;
            try {
                for (int i = 0; i < 12; i += 2) {
                    check += Integer.parseInt(isbn.substring(i, i + 1));
                }
                for (int i = 1; i < 12; i += 2) {
                    check += Integer.parseInt(isbn.substring(i, i + 1)) * 3;
                }
                check += Integer.parseInt(isbn.substring(12));
            } catch (NumberFormatException e) {
                return false;
            }
            return check % 10 == 0;
        }
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | ジェネリクスのパラメータに、対象のアノテーションとフィールドの型を指定する。
     * - | (2)
       - | \ ``initialize``\ メソッドに、初期化処理を実装する。
     * - | (3)
       - | \ ``isValid``\ メソッドで入力チェック処理を実装する。
     * - | (4)
       - | 入力値が、\ ``null``\ の場合は、正常とみなす。
     * - | (5)
       - | ISBN-13の形式のチェックを行う。

.. tip::

  :ref:`fileupload_validator`\ の例も、ここに分類される。また共通ライブラリでは、この実装として\ :ref:`@ExistInCodeList <codelist-validate>`\ を用意している。

.. _Validation_correlation_item_check:

相関項目チェックルール
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| :ref:`Validation_correlation_check`\ で説明したように、Bean Validationによって複数のフィールドにまたがる相関項目チェックを実装できる。
| Bean Validationで相関項目チェックルールを実装する場合は、汎用的なルールを対象とすることを推奨する。

以下では、「あるフィールドとその確認用フィールドの内容が一致すること」というルールを実現する例を挙げる。

ここでは、確認用フィールドの先頭に、「confirm」を付与する規約を設ける。

* アノテーション

  相関項目チェック用のアノテーションはクラスレベルに付与できるようにする。

  .. code-block:: java
    :emphasize-lines: 14,26

    package com.example.common.validation;

    import java.lang.annotation.Documented;
    import java.lang.annotation.Retention;
    import java.lang.annotation.Target;
    import javax.validation.Constraint;
    import javax.validation.Payload;
    import static java.lang.annotation.ElementType.ANNOTATION_TYPE;
    import static java.lang.annotation.ElementType.TYPE;
    import static java.lang.annotation.RetentionPolicy.RUNTIME;

    @Documented
    @Constraint(validatedBy = { ConfirmValidator.class })
    @Target({ TYPE, ANNOTATION_TYPE }) // (1)
    @Retention(RUNTIME)
    public @interface Confirm {
        String message() default "{com.example.common.validation.Confirm.message}";

        Class<?>[] groups() default {};

        Class<? extends Payload>[] payload() default {};

        /**
         * Field name
         */
        String field(); // (2)

        @Target({ TYPE, ANNOTATION_TYPE })
        @Retention(RUNTIME)
        @Documented
        public @interface List {
            Confirm[] value();
        }
    }


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | このアノテーションが、クラスまたはアノテーションにのみ付加できるように、対象を絞る。
     * - | (2)
       - | アノテーションに渡すパラメータを定義する。

* Validator

  .. code-block:: java

    package com.example.common.validation;

    import javax.validation.ConstraintValidator;
    import javax.validation.ConstraintValidatorContext;

    import org.springframework.beans.BeanWrapper;
    import org.springframework.beans.BeanWrapperImpl;
    import org.springframework.util.ObjectUtils;
    import org.springframework.util.StringUtils;

    public class ConfirmValidator implements ConstraintValidator<Confirm, Object> {
        private String field;

        private String confirmField;

        private String message;

        public void initialize(Confirm constraintAnnotation) {
            field = constraintAnnotation.field();
            confirmField = "confirm" + StringUtils.capitalize(field);
            message = constraintAnnotation.message();
        }

        public boolean isValid(Object value, ConstraintValidatorContext context) {
            BeanWrapper beanWrapper = new BeanWrapperImpl(value); // (1)
            Object fieldValue = beanWrapper.getPropertyValue(field); // (2)
            Object confirmFieldValue = beanWrapper.getPropertyValue(confirmField);
            boolean matched = ObjectUtils.nullSafeEquals(fieldValue,
                    confirmFieldValue);
            if (matched) {
                return true;
            } else {
                context.disableDefaultConstraintViolation(); // (3)
                context.buildConstraintViolationWithTemplate(message)
                        .addPropertyNode(field).addConstraintViolation(); // (4)
                return false;
            }
        }

    }


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | JavaBeanのプロパティにアクセスする際に便利な\ ``org.springframework.beans.BeanWrapper``\ を使用する。
     * - | (2)
       - | \ ``BeanWrapper``\ 経由で、フォームオブジェクトからプロパティ値を取得する。
     * - | (3)
       - | デフォルトの\ ``ConstraintViolation``\ オブジェクトの生成を無効にする。
     * - | (4)
       - | 独自\ ``ConstraintViolation``\ オブジェクトを生成する。
         | \ ``ConstraintValidatorContext.buildConstraintViolationWithTemplate``\ で出力するメッセージを定義する。
         | \ ``ConstraintViolationBuilder.addPropertyNode``\ でエラーメッセージを出力したいフィールド名を指定する。
         | 詳細は、\ `ConstraintValidatorContextのJavaDoc <http://docs.oracle.com/javaee/7/api/javax/validation/ConstraintValidatorContext.html>`_\ を参照されたい。

 .. tip::

    \ ``ConstraintViolationBuilder.addPropertyNode``\ メソッドは、Bean Validation 1.1 から追加されたメソッドである。

    Bean Validation 1.0では \ ``ConstraintViolationBuilder.addNode``\ というメソッドを使用していたが、Bean Validation 1.1から非推奨のAPIとなっている。

    Bean Validationの非推奨APIについては、\ `Bean Validation API Document(Deprecated API) <http://docs.jboss.org/hibernate/beanvalidation/spec/1.1/api/deprecated-list.html>`_\ を参照されたい。

.. note::

   Spring Validatorによる相関項目チェックにて紹介したように、Bean Validationにおいても
   :ref:`相関チェック対象の複数フィールドに対してエラー情報を設定する<Validation_how_to_cross-field_validation_for_multi_field_highlight>` ことが可能である。

   以下に、Bean Validationにて\ ``password``\ フィールドと\ ``confirmPassword``\ フィールドにスタイルを適用し、\ ``password``\ フィールドのみにエラーメッセージを表示する例を示す。

     .. code-block:: java

       // omitted
       public class ConfirmValidator implements ConstraintValidator<Confirm, Object> {
           private String field;

           private String confirmField;

           private String message;

           public void initialize(Confirm constraintAnnotation) {
               // omitted
           }

           public boolean isValid(Object value, ConstraintValidatorContext context) {
               // omitted
               if (matched) {
                   return true;
               } else {
                   context.disableDefaultConstraintViolation();

                   //new ConstraintViolation to be generated for field
                   context.buildConstraintViolationWithTemplate(message)
                           .addPropertyNode(field).addConstraintViolation();

                   //new ConstraintViolation to be generated for confirmField
                   context.buildConstraintViolationWithTemplate("") // (1)
                           .addPropertyNode(confirmField).addConstraintViolation();

                   return false;
               }
           }

       }

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90

        * - 項番
          - 説明
        * - | (1)
          - | \ ``confirmPassword``\ フィールドのエラーを登録する。この際、エラーメッセージに空文字を設定している。


この\ ``@Confirm``\ アノテーションを使用して、前述の「パスワードリセット」処理を再実装すると、以下のようになる。


* フォームクラス

  .. code-block:: java

    package com.example.sample.app.validation;

    import java.io.Serializable;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;

    import com.example.common.validation.Confirm;

    @Confirm(field = "password") // (1)
    public class PasswordResetForm implements Serializable {
        private static final long serialVersionUID = 1L;

        @NotNull
        @Size(min = 8)
        private String password;

        private String confirmPassword;

        // omitted geter/setter
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | クラスレベルに\ ``@Confirm``\ アノテーションを付与する。
         | これにより\ ``ConstraintValidator.isValid``\ の引数にはフォームオブジェクトが渡る。

* Controllerクラス

  Validatorのインジェクションおよび\ ``@InitBinder``\ によるValidatorの追加は、不要になる。

  .. code-block:: java

    package com.example.sample.app.validation;

    import org.springframework.stereotype.Controller;
    import org.springframework.validation.BindingResult;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;

    @Controller
    @RequestMapping("password")
    public class PasswordResetController {

        @ModelAttribute
        public PasswordResetForm setupForm() {
            return new PasswordResetForm();
        }

        @RequestMapping(value = "reset", method = RequestMethod.GET, params = "form")
        public String resetForm() {
            return "password/resetForm";
        }

        @RequestMapping(value = "reset", method = RequestMethod.POST)
        public String reset(@Validated PasswordResetForm form, BindingResult result) {
            if (result.hasErrors()) {
                return "password/resetForm";
            }
            return "redirect:/password/reset?complete";
        }

        @RequestMapping(value = "reset", method = RequestMethod.GET, params = "complete")
        public String resetComplete() {
            return "password/resetComplete";
        }
    }

.. _Validation_business_logic_check:

業務ロジックチェック
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 業務ロジックチェックは、基本的には\ :ref:`ドメイン層のServiceで実装 <service-implementation-label>`\ し、結果メッセージは\ ``ResultMessages``\ オブジェクトに格納することを推奨している。
| したがって、\ :ref:`通常画面の上部などに表示されることを想定している <message-display>`\ 。

一方で、「入力されたユーザー名が既に登録済みかどうか」など、対象の入力フィールドに対する業務ロジックエラーメッセージを、フィールドの横に表示したい場合もある。
このような場合は、ValidatorクラスにServiceクラスをインジェクションして、業務ロジックチェックを実行し、その結果を、\ ``ConstraintValidator.isValid``\ の結果に使用すればよい。

「入力されたユーザー名が既に登録済みかどうか」をBean Validationで実現する例を示す。


* Serviceクラス

  実装クラス(UserServiceImpl)は割愛する。

  .. code-block:: java

    package com.example.sample.domain.service.user;

    public interface UserService {

        boolean isUnusedUserId(String userId);

        // omitted other methods
    }

* アノテーション

  .. code-block:: java

    package com.example.sample.domain.validation;

    import java.lang.annotation.Documented;
    import java.lang.annotation.Retention;
    import java.lang.annotation.Target;
    import javax.validation.Constraint;
    import javax.validation.Payload;
    import static java.lang.annotation.ElementType.ANNOTATION_TYPE;
    import static java.lang.annotation.ElementType.CONSTRUCTOR;
    import static java.lang.annotation.ElementType.FIELD;
    import static java.lang.annotation.ElementType.METHOD;
    import static java.lang.annotation.ElementType.PARAMETER;
    import static java.lang.annotation.RetentionPolicy.RUNTIME;

    @Documented
    @Constraint(validatedBy = { UnusedUserIdValidator.class })
    @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
    @Retention(RUNTIME)
    public @interface UnusedUserId {
        String message() default "{com.example.sample.domain.validation.UnusedUserId.message}";

        Class<?>[] groups() default {};

        Class<? extends Payload>[] payload() default {};

        @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
        @Retention(RUNTIME)
        @Documented
        public @interface List {
            UnusedUserId[] value();
        }
    }


* Validatorクラス

  .. code-block:: java
    :emphasize-lines: 11,15-16

    package com.example.sample.domain.validation;

    import javax.inject.Inject;
    import javax.validation.ConstraintValidator;
    import javax.validation.ConstraintValidatorContext;

    import org.springframework.stereotype.Component;

    import com.example.sample.domain.service.user.UserService;

    @Component // (1)
    public class UnusedUserIdValidator implements
                                      ConstraintValidator<UnusedUserId, String> {

        @Inject // (2)
        UserService userService;

        @Override
        public void initialize(UnusedUserId constraintAnnotation) {
        }

        @Override
        public boolean isValid(String value, ConstraintValidatorContext context) {
            if (value == null) {
                return true;
            }

            return userService.isUnusedUserId(value); // (3)
        }

    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | Validatorクラスをコンポーネントスキャンの対象にする。
         | パッケージがBean定義ファイルの\ ``<context:component-scan base-package="..." />``\ の設定に含まれている必要がある。
     * - | (2)
       - | 呼び出すServiceクラスを、インジェクションする。
     * - | (3)
       - | 業務ロジックの結果を返却する。\ ``isValid``\ メソッド名で業務ロジックを記述せず、かならずServiceに処理を委譲すること。

|

.. _MethodValidation:

Method Validation 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Bean Validationによってメソッドの実引数と返り値の妥当性を確認する方法を説明する。
説明のために、本節ではこの方法をMethod Validationと呼ぶ。
防衛的プログラミングを行う場合などでは、Controller以外のクラスでメソッドの入出力を確認する必要がある。
このとき、Bean Validationライブラリを利用すれば、Controllerで使用したBean Validationの制約アノテーションを再利用できる。

.. _MethodValidationOnSpringFrameworkHowToUseSettings:

アプリケーションの設定
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Frameworkが提供するMethod Validationを使用する場合は、
Spring Frameworkから提供されている\ ``org.springframework.validation.beanvalidation.MethodValidationPostProcessor``\ クラスをBean定義する必要がある。

\ ``MethodValidationPostProcessor``\ を定義するBean定義ファイルは、Method Validationを使用する箇所によって異なる。

ここでは、本ガイドラインが推奨するマルチプロジェクト環境でMethod Validationを使用するための設定例を示す。

* アプリケーション層用のプロジェクト(\ ``projectName-web``\ )
* ドメイン層用のプロジェクト(\ ``projectName-domain``\ )

の両プロジェクトの設定を変更する必要がある。

* :file:`projectName-domain/src/main/resources/META-INF/spring/projectName-domain.xml`

 .. code-block:: xml

    <!-- (1) -->
    <bean id="validator"
          class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean"/>

    <!-- (2) -->
    <bean class="org.springframework.validation.beanvalidation.MethodValidationPostProcessor">
        <property name="validator" ref="validator" />
    </bean>

* :file:`projectName-web/src/main/resources/META-INF/spring/spring-mvc.xml`

 .. code-block:: xml

    <!-- (3) -->
    <mvc:annotation-driven validator="validator">
        <!-- ... -->
    </mvc:annotation-driven>

    <!-- (4) -->
    <bean class="org.springframework.validation.beanvalidation.MethodValidationPostProcessor">
        <property name="validator" ref="validator" />
    </bean>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - \ ``LocalValidatorFactoryBean``\ をBean定義する。
    * - | (2)
      - \ ``MethodValidationPostProcessor``\ をBean定義し、
        ドメイン層のクラスのメソッドに対してMethod Validationが実行されるようにする。

        \ ``validator``\ プロパティには、(1)で定義したBeanを指定する。
    * - | (3)
      - \ ``<mvc:annotation-driven>``\ 要素の\ ``validator``\ 属性に、(1)で定義したBeanを指定する。

        この設定がない場合は(1)で作成したものとは異なる\ ``Validator``\ インスタンスが生成されてしまう。
    * - | (4)
      - \ ``MethodValidationPostProcessor``\ をBean定義し、
        アプリケーション層のクラスのメソッドに対してMethod Validationが実行されるようにする。

        \ ``validator``\ プロパティには、(1)で定義したBeanを指定する。

.. tip::

    \ ``LocalValidatorFactoryBean``\ は、
    Bean Validation(Hibernate Validator)が提供する\ ``Validator``\ クラスとSpring Frameworkを連携するためのラッパー\ ``Validator``\ オブジェクトを生成するためのクラスである。

    このクラスによって生成されたラッパー\ ``Validator``\を使用することで、
    Spring Frameworkが提供するメッセージ管理機能(\ ``MessageSource``\ )やDIコンテナなどとの連携が行えるようになる。

.. tip::

    Spring Frameworkでは、DIコンテナで管理されているBeanのメソッド呼び出しに対するMethod Validationの実行を、
    AOPの仕組みを利用して行っている。

    \ ``MethodValidationPostProcessor``\ は、Method Validationを実行するためのAOPを適用するためのクラスである。

.. note::

    上記例では、各Beanの\ ``validator``\ プロパティに対して、同じ\ ``Validator``\ オブジェクト(インスタンス)を設定しているが、
    これは必ずしも必須ではない。
    ただし、特に理由がない場合は、同じオブジェクト(インスタンス)を設定しておくことを推奨する。

|

.. _MethodValidationOnSpringFrameworkHowToUseApplyTarget:

Method Validation対象のメソッドにするための定義方法
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

メソッドにMethod Validationを適用するには、
対象のメソッドを含むことを示したアノテーションをクラスレベルに、
Bean Validationの制約アノテーションをメソッドと仮引数にそれぞれ指定する必要がある。

「:ref:`MethodValidationOnSpringFrameworkHowToUseSettings`」を行っただけでは、Method Validationを実行するAOPは適用されない。
Method Validationを実行するAOPを適用するためには、
インタフェース又はクラスに\ ``@ org.springframework.validation.annotation.Validated``\ アノテーションを付与する必要がある。

ここでは、インタフェースに対してアノテーションを指定する方法を紹介する。

.. code-block:: java

    package com.example.domain.service;

    import org.springframework.validation.annotation.Validated;

    @Validated // (1)
    public interface HelloService {
        // ...
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - Method Validationの対象となるインタフェースに、\ ``Validated``\ アノテーションを指定する。

        上記例では、\ ``HelloService``\ インタフェースの実装メソッドに対して、
        Method Validationを実行するAOPが適用される。

.. tip::

    \ ``@Validated``\ アノテーションの\ ``value``\ 属性にグループインタフェースを指定することで、
    指定したグループに属するValidationのみ実行する事も可能である。

    また、メソッドレベルに\ ``Validated``\ アノテーションを付与することで、
    メソッド毎にバリデーショングループを切り替える事も可能な仕組みとなっている。

    バリデーショングループについては、「:ref:`ValidationGroupValidation`」を参照されたい。

|

次に、Bean Validationの制約アノテーションをメソッドや仮引数へ指定する方法を説明する。
具体的には、

* メソッドの引数
* メソッドの引数に指定されたJavaBeanのフィールド

に対してBean Validationの制約アノテーションを、

* メソッドの返り値
* メソッドの返り値として返却するJavaBeanのフィールド

に対してBean Validationの制約アノテーションを指定する。

以下に、具体的な指定方法について説明する。
以降の説明では、インタフェースにアノテーションを指定する方法を紹介する。

まず、メソッドのシグネチャとして基本型(プリミティブやプリミティブラッパ型など)を使用するメソッドに対して、
制約アノテーションを指定する方法について説明する。

.. code-block:: java

    package com.example.domain.service;

    import org.springframework.validation.annotation.Validated;

    import javax.validation.constraints.NotNull;

    @Validated
    public interface HelloService {

        // (2)
        @NotNull
        String hello(@NotNull /* (1) */ String message);

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - Bean Validationの制約アノテーションをメソッドの引数アノテーションとして指定する。

        \ ``@NotNull``\は\ ``message``\ という引数がNull値を許可しないことを意味する制約である。
        引数にNull値が指定された場合、\ ``javax.validation.ConstraintViolationException``\ が発生する。
    * - | (2)
      - Bean Validationの制約アノテーションをメソッドアノテーションとして指定する。

        上記例では、返り値がNull値にならないことを示しており、
        返り値としてNull値が返却された場合、\ ``javax.validation.ConstraintViolationException``\ が発生する。

|

次に、メソッドのシグネチャとしてJavaBeanを使用するメソッドに対して、
Bean Validationの制約アノテーションを指定する方法について説明する。

ここでは、インタフェースに対してアノテーションを指定する方法を紹介する。

.. note::

    ポイントは、\ ``@javax.validation.Valid``\ アノテーションを指定するという点である。
    以下に、サンプルコード使って指定方法を詳しく説明する。

**Serviceインタフェース**

.. code-block:: java

    package com.example.domain.service;

    import org.springframework.validation.annotation.Validated;

    import javax.validation.constraints.NotNull;

    @Validated
    public interface HelloService {

        @NotNull // (3)
        @Valid   // (4)
        HelloOutput hello(@NotNull /* (1) */ @Valid /* (2) */ HelloInput input);

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - Bean Validationの制約アノテーションをメソッドの引数アノテーションとして指定する。

        \ ``input``\ という引数(JavaBean)がNull値を許可しない事を示しており、
        引数にNull値が指定された場合は、\ ``javax.validation.ConstraintViolationException``\ が発生する。
    * - | (2)
      - \ ``@javax.validation.Valid``\ アノテーションをメソッドの引数アノテーションとして指定する。

        \ ``@Valid``\ アノテーションを付与する事で、引数のJavaBeanのフィールドに指定したBean Validationの制約アノテーションが有効となる。
        JavaBeanに指定された制約を満たさない場合は\ ``javax.validation.ConstraintViolationException``\ が発生する。
    * - | (3)
      - Bean Validationの制約アノテーションをメソッドアノテーションとして指定する。

        返り値のJavaBeanがNull値にならないことを示しており、
        返り値としてNull値が返却された場合は例外が発生する。
    * - | (4)
      - \ ``@Valid``\ アノテーションをメソッドアノテーションとして指定する。

        \ ``@Valid``\ アノテーションを付与する事で、返り値のJavaBeanのフィールドに指定したBean Validationの制約アノテーションが有効となる。
        JavaBeanに指定された制約を満たさない場合は\ ``javax.validation.ConstraintViolationException``\ が発生する。

|

| 以下にJavaBeanの実装サンプルを紹介する。
| 基本的には、Bean Validationの制約アノテーションを指定するだけだが、JavaBeanが更にJavaBeanをネストしている場合は注意が必要になる。

**Input用のJavaBean**

.. code-block:: java

    package com.example.domain.service;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Past;
    import java.util.Date;

    public class HelloInput {

        @NotNull
        @Past
        private Date visitDate;

        @NotNull
        private String visitMessage;

        private String userId;

        // ...

    }

**Output用のJavaBean**

.. code-block:: java

    package com.example.domain.service;

    import com.example.domain.model.User;

    import java.util.Date;

    import javax.validation.Valid;
    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Past;

    public class HelloOutput {

        @NotNull
        @Past
        private Date acceptDate;

        @NotNull
        private String acceptMessage;

        @Valid // (5)
        private User user;

        // ...

    }

**Output用のJavaBean内でネストしているJavaBean**

.. code-block:: java

    package com.example.domain.model;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Past;
    import java.util.Date;

    public class User {

        @NotNull
        private String userId;

        @NotNull
        private String userName;

        @Past
        private Date dateOfBirth;

        // ...

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (5)
      - ネストしたJavaBeanに指定しているBean Validationの制約アノテーションを有効にする場合は、
        \ ``@Valid``\ アノテーションをフィールドアノテーションとして指定する。

        \ ``@Valid``\ アノテーションを付与する事で、ネストしたJavaBeanのフィールドに指定したBean Validationの制約アノテーションが有効となる。
        ネストしたJavaBeanに指定された制約を満たさない場合は\ ``javax.validation.ConstraintViolationException``\ が発生する。

|

.. _MethodValidationOnSpringFrameworkHowToUseExceptionHandling:

制約違反時の例外ハンドリング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

制約に違反した場合、\ ``javax.validation.ConstraintViolationException``\ が発生する。

\ ``ConstraintViolationException``\ が発生した場合、スタックトレースから発生したメソッドは特定できるが、
具体的な違反内容が特定できない。

違反内容を特定するためには、\ ``ConstraintViolationException``\ をハンドリングしてログ出力を行う例外ハンドリングクラスを作成するとよい。

以下の例外ハンドリングクラスの作成例を示す。

.. code-block:: java

    package com.example.app;

    import javax.validation.ConstraintViolationException;

    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.web.bind.annotation.ControllerAdvice;
    import org.springframework.web.bind.annotation.ExceptionHandler;

    @ControllerAdvice
    public class ConstraintViolationExceptionHandler {

        private static final Logger log = LoggerFactory.getLogger(ConstraintViolationExceptionHandler.class);

        // (1)
        @ExceptionHandler
        public String handleConstraintViolationException(ConstraintViolationException e){
            // (2)
            if (log.isErrorEnabled()) {
                log.error("ConstraintViolations[\n{}\n]", e.getConstraintViolations());
            }
            return "common/error/systemError";
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - \ ``ConstraintViolationException``\ をハンドリングするための\ ``@ExceptionHandler``\ メソッドを作成する。

        メソッドの引数として、\ ``ConstraintViolationException``\ を受け取るようにする。
    * - | (2)
      - メソッドの引数で受け取った\ ``ConstraintViolationException``\ が保持している違反内容(\ ``ConstraintViolation``\ の\ ``Set``\ )をログに出力する。

.. note::

    \ ``@ControllerAdvice``\ アノテーションの詳細については「:ref:`application_layer_controller_advice`」を参照されたい。

.. warning::

    \ ``ConstraintViolation#getMessage``\ メソッドを使用することでエラーメッセージを取得することができるが、Springの機能によるメッセージ補完は行われないため、エラーメッセージに \ ``{0}``\ でフィールド名を埋め込むことはできない。
    
    代わりに、フィールド名は\ ``ConstraintViolation#getPropertyPath``\メソッドで取得することが可能である。
    
    Springの機能によるメッセージ補完については、:ref:`Validation_message_in_validationmessages` のNoteを参照されたい。
    
    \ ``ConstraintViolation``\ の詳細については、\ `Hibernate Validatorのリファレンス <http://docs.jboss.org/hibernate/validator/5.2/reference/en-US/html_single/#section-constraint-violation-methods>`_\ を参照されたい。
    

Appendix
--------------------------------------------------------------------------------

Hibernate Validatorが用意する入力チェックルール
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Hibernate ValidatorはBean Validationで定義されたアノテーションに加え、独自の検証用アノテーションを提供している。
| 検証に使用することができるアノテーションのリストは、\ `こちら <http://docs.jboss.org/hibernate/validator/5.2/reference/en-US/html/ch02.html#section-builtin-constraints>`_\ を参照されたい。

.. _Validation_jsr303_doc:

Bean Validationのチェックルール
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Bean Validationの標準アノテーション(\ ``javax.validation.*``\ )を以下に示す。

詳細は、\ `Bean Validation specification <http://download.oracle.com/otn-pub/jcp/bean_validation-1_1-fr-eval-spec/bean-validation-specification.pdf>`_\ の7章を参照されたい。

.. tabularcolumns:: |p{0.15\linewidth}|p{0.30\linewidth}|p{0.30\linewidth}|p{0.25\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 15 30 30 25
   :class: longtable

   * - アノテーション
     - 対象の型
     - 説明
     - 使用例
   * - \ ``@NotNull``\
     - 任意
     - 対象のフィールドが、\ ``null``\ でないことを検証する。
     - .. code-block:: java

            @NotNull
            private String id;
            
   * - \ ``@Null``\
     - 任意
     - | 対象のフィールドが、\ ``null``\ であることを検証する。
       | (例：グループ検証での使用)
     - .. code-block:: java

            @Null(groups={Update.class})
            private String id;
            
   * - \ ``@Pattern``\
     - \ ``String``\
     - | 対象のフィールドが正規表現にマッチするかどうか
       | (Hibernate Validator実装では、任意の\ ``CharSequence``\ インタフェースの実装クラスにも適用可能)
     - .. code-block:: java

            @Pattern(regexp = "[0-9]+")
            private String tel;
            
   * - \ ``@Min``\
     - | \ ``BigDecimal``\ , \ ``BigInteger``\ , \ ``byte``\ , \ ``short``\ , \ ``int``\ , \ ``long``\ およびラッパー
       | (Hibernate Validator実装では、任意の\ ``Number``\ の継承クラス,\ ``CharSequence``\ インタフェースの実装クラスにも適用可能。ただし、文字列が数値表現の場合に限る。)
     - 値が、最小値以上であるかどうかを検証する。
     - @Max参照
   * - \ ``@Max``\
     - | \ ``BigDecimal``\ , \ ``BigInteger``\ , \ ``byte``\ , \ ``short``\ , \ ``int``\ , \ ``long``\ およびラッパー
       | (Hibernate Validator実装では任意の\ ``Number``\ の継承クラス,\ ``CharSequence``\ インタフェースの実装クラスにも適用可能。ただし、文字列が数値表現の場合に限る。)
     - 値が、最大値以下であるかどうかを検証する。
     - .. code-block:: java

            @Min(1)
            @Max(100)
            private int quantity;
            
   * - \ ``@DecimalMin``\
     - \ ``BigDecimal``\ , \ ``BigInteger``\ , \ ``String``\ , \ ``byte``\ , \ ``short``\ , \ ``int``\ , \ ``long``\ およびラッパー
       (Hibernate Validator実装では任意の\ ``Number``\ の継承クラス,\ ``CharSequence``\ インタフェースの実装クラスにも適用可能)
     - | \ ``Decimal``\ 型の値が、最小値以上であるかどうかを検証する。
       | \ ``inclusive = false``\ を指定する事で、最小値より大きいかどうかを検証するように動作を変更する事ができる。
     - \ ``@DecimalMax``\ 参照
   * - \ ``@DecimalMax``\
     - \ ``BigDecimal``\ , \ ``BigInteger``\ , \ ``String``\ , \ ``byte``\ , \ ``short``\ , \ ``int``\ , \ ``long``\ およびラッパー
       (Hibernate Validator実装では任意の\ ``Number``\ の継承クラス,\ ``CharSequence``\ インタフェースの実装クラスにも適用可能)
     - | \ ``Decimal``\ 型の値が、最大値以下であるかどうかを検証する。
       | \ ``inclusive = false``\ を指定する事で、最大値より小さいかどうかを検証するように動作を変更する事ができる。
     - .. code-block:: java

            @DecimalMin("0.0")
            @DecimalMax("99999.99")
            private BigDecimal price;
            
   * - \ ``@Size``\
     - \ ``String``\ (文字列の長さ), \ ``Collection``\ (要素のサイズ), \ ``Map``\ (要素のサイズ), Array(配列の長さ)
       (Hibernate Validator実装では、任意の\ ``CharSequence``\ インタフェースの実装クラスにも適用可能)
     - | 要素の長さ(要素のサイズ)が\ ``min``\ と\ ``max``\ の間のサイズか検証する。
       | \ ``min``\ と\ ``max``\ は省略可能であるが、デフォルトは\ ``min=0``\ ,\ ``max= Integer.MAX_VALUE``\ となる。
     - .. code-block:: java

            @Size(min=4, max=64)
            private String password;
            
   * - \ ``@Digits``\
     - \ ``BigDecimal``\ , \ ``BigInteger``\ , \ ``String``\ , \ ``byte``\ , \ ``short``\ , \ ``int``\ , \ ``long``\ およびラッパー
     - | 値が指定された範囲内の数値であるかチェックする。
       | \ ``integer``\ に最大整数の桁を指定し、\ ``fraction``\ に最大小数桁を指定する。
     - .. code-block:: java

            @Digits(integer=6, fraction=2)
            private BigDecimal price;
            
   * - \ ``@AssertTrue``\
     - \ ``boolean``\ ,\ ``Boolean``\
     - 対象のフィールドが\ ``true``\ であることを検証する(例：規約に同意したかどうか）
     - .. code-block:: java

            @AssertTrue
            private boolean checked;
            
   * - \ ``@AssertFalse``\
     - \ ``boolean``\ ,\ ``Boolean``\
     - 対象のフィールドが\ ``false``\ であることを検証する
     - .. code-block:: java

            @AssertFalse
            private boolean checked;
            
   * - \ ``@Future``\
     - \ ``Date``\ , \ ``Calendar``\
       (Hibernate Validator実装ではJoda-Timeのクラスにも適用可能)
     - 未来日付であるか検証する。
     - .. code-block:: java

            @Future
            private Date eventDate;
            
   * - \ ``@Past``\
     - \ ``Date``\ , \ ``Calendar``\
       (Hibernate Validator実装ではJoda-Timeのクラスにも適用可能)
     - 過去日付であるか検証する。
     - .. code-block:: java

            @Past
            private Date eventDate;
            
   * - \ ``@Valid``\
     - 任意の非プリミティブ型
     - 関連付けられているオブジェクトについて、再帰的に検証を行う。
     - .. code-block:: java

            @Valid
            private List<Employer> employers;
            
            @Valid
            private Dept dept;

.. raw:: latex

   \newpage

.. tip::

     \ ``@DecimalMin``\  と \ ``@DecimalMax``\ アノテーションの \ ``inclusive``\ 属性は、
     Bean Validation 1.1 から追加された属性である。

     \ ``inclusive``\ 属性のデフォルト値には \ ``true``\ (指定した閾値と同じ値を許容する)が指定されており、
     Bean Validation 1.0 との互換性が保たれている。

.. warning::

     \ ``@Size``\ アノテーションでは、サロゲートペアと呼ばれるchar型2つ（32ビット）で表される文字に対する考慮がされていない。

     サロゲートペアを含む文字列をチェック対象とした場合、カウントした文字数が実際の入力文字数より多くカウントされる可能性があるため注意すること。

     サロゲートペアを含む文字列の文字列長については、 :ref:`StringProcessingHowToGetSurrogatePairStringLength` を参照されたい。


.. _Validation_validator_list:

Hibernate Validatorのチェックルール
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Hibernate Validatorの代表的なアノテーション(\ ``org.hibernate.validator.constraints.*``\ )を以下に示す。

詳細は、\ `Hibernate Validator仕様 <http://docs.jboss.org/hibernate/validator/5.2/reference/en-US/html/ch02.html#validator-defineconstraints-hv-constraints>`_\ を参照されたい。

.. tabularcolumns:: |p{0.15\linewidth}|p{0.30\linewidth}|p{0.30\linewidth}|p{0.25\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 15 30 30 25

   * - アノテーション
     - 対象の型
     - 説明
     - 使用例
   * - \ ``@CreditCardNumber``\
     - 任意の\ ``CharSequence``\ インタフェースの実装クラスに適用可能
     - | Luhnアルゴリズムでクレジットカード番号が妥当かどうかを検証する。使用可能な番号かどうかをチェックするわけではない。
       | \ ``ignoreNonDigitCharacters = true``\ を指定する事で、数字以外の文字を無視して検証する事ができる。
     - .. code-block:: java

            @CreditCardNumber
            private String cardNumber;
            
   * - \ ``@Email``\
     - 任意の\ ``CharSequence``\ インタフェースの実装クラスに適用可能
     - RFC2822に準拠したEmailアドレスかどうか検証する。
     - .. code-block:: java

            @Email
            private String email;
            
   * - \ ``@URL``\
     - 任意の\ ``CharSequence``\ インタフェースの実装クラスに適用可能
     - URLとして妥当であること検証する。\ ``java.net.URL``\ のコンストラクタを使用して文字列検証を行っており、
       URLとして妥当とされるプロトコルはJVMがサポートするプロトコル(\ ``http``\ ,\ ``https``\ ,\ ``file``\ ,\ ``jar``\ など)に依存する。
     - .. code-block:: java

            @URL
            private String url;
            
   * - \ ``@NotBlank``\
     - 任意の\ ``CharSequence``\ インタフェースの実装クラスに適用可能
     - \ ``null``\ 、空文字(\ ``""``\ )、空白のみでないことを検証する。
     - .. code-block:: java

            @NotBlank
            private String userId;
            
   * - \ ``@NotEmpty``\
     - \ ``Collection``\ 、\ ``Map``\ 、Array、任意の\ ``CharSequence``\ インタフェースの実装クラスに適用可能
     - | \ ``null``\ 、または空でないことを検証する。
       | \ ``@NotNull``\  + \ ``@Min(1)``\ の組み合わせでチェックする場合は、\ ``@NotEmpty``\ を使用すること。
     - .. code-block:: java

            @NotEmpty
            private String password;

.. tip::

     \ ``@URL``\ にて、JVMがサポートしていないプロトコルについても妥当として検証したい場合、Hibernateから提供されている\ ``org.hibernate.validator.constraintvalidators.RegexpURLValidator``\ を使用する。
     当該クラスは\ ``@URL``\ アノテーションに対応するValidatorクラスで、URL形式であるかを正規表現で検証しており、JVMがサポートしていないプロトコルについても妥当として検証可能である。

     * アプリケーション全体の\ ``@URL``\ のチェックルールを変更してもよい場合には、\ `JavaDoc <https://docs.jboss.org/hibernate/validator/5.2/api/org/hibernate/validator/constraints/URL.html>`_\ に記載されているように、
       XMLにてValidatorクラスを\ ``RegexpURLValidator``\ に変更する。
     * 一部の項目だけに正規表現による検証を適用し、\ ``@URL``\ はデフォルトのルールを使用したい場合には、新規アノテーション、および\ ``RegexpURLValidator``\ と同様の検証を行う\ ``javax.validation.ConstraintValidator``\ 実装クラスを作成し、
       必要な項目に作成したアノテーションによる検証を適用する。

     など、用途に応じた適用を行えばよい。

     XMLによるチェックルール変更の詳細については\ `Hibernateのリファレンス <https://docs.jboss.org/hibernate/validator/5.2/reference/en-US/html/ch07.html#section-configuration-validation-xml>`_\ を、
     新規アノテーションの作成方法については、\ :ref:`Validation_implement_new_constraint`\ をそれぞれ参照されたい。

.. _Validation_default_message_in_hibernate_validator:

Hibernate Validatorが用意するデフォルトメッセージ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

hibernate-validator-<version>.jar内のorg/hibernate/validatorに、ValidationMessages.propertiesのデフォルト値が定義されている。

.. code-block:: properties

  javax.validation.constraints.AssertFalse.message = must be false
  javax.validation.constraints.AssertTrue.message  = must be true
  javax.validation.constraints.DecimalMax.message  = must be less than ${inclusive == true ? 'or equal to ' : ''}{value}
  javax.validation.constraints.DecimalMin.message  = must be greater than ${inclusive == true ? 'or equal to ' : ''}{value}
  javax.validation.constraints.Digits.message      = numeric value out of bounds (<{integer} digits>.<{fraction} digits> expected)
  javax.validation.constraints.Future.message      = must be in the future
  javax.validation.constraints.Max.message         = must be less than or equal to {value}
  javax.validation.constraints.Min.message         = must be greater than or equal to {value}
  javax.validation.constraints.NotNull.message     = may not be null
  javax.validation.constraints.Null.message        = must be null
  javax.validation.constraints.Past.message        = must be in the past
  javax.validation.constraints.Pattern.message     = must match "{regexp}"
  javax.validation.constraints.Size.message        = size must be between {min} and {max}

  org.hibernate.validator.constraints.CreditCardNumber.message        = invalid credit card number
  org.hibernate.validator.constraints.EAN.message                     = invalid {type} barcode
  org.hibernate.validator.constraints.Email.message                   = not a well-formed email address
  org.hibernate.validator.constraints.Length.message                  = length must be between {min} and {max}
  org.hibernate.validator.constraints.LuhnCheck.message               = The check digit for ${validatedValue} is invalid, Luhn Modulo 10 checksum failed
  org.hibernate.validator.constraints.Mod10Check.message              = The check digit for ${validatedValue} is invalid, Modulo 10 checksum failed
  org.hibernate.validator.constraints.Mod11Check.message              = The check digit for ${validatedValue} is invalid, Modulo 11 checksum failed
  org.hibernate.validator.constraints.ModCheck.message                = The check digit for ${validatedValue} is invalid, ${modType} checksum failed
  org.hibernate.validator.constraints.NotBlank.message                = may not be empty
  org.hibernate.validator.constraints.NotEmpty.message                = may not be empty
  org.hibernate.validator.constraints.ParametersScriptAssert.message  = script expression "{script}" didn't evaluate to true
  org.hibernate.validator.constraints.Range.message                   = must be between {min} and {max}
  org.hibernate.validator.constraints.SafeHtml.message                = may have unsafe html content
  org.hibernate.validator.constraints.ScriptAssert.message            = script expression "{script}" didn't evaluate to true
  org.hibernate.validator.constraints.URL.message                     = must be a valid URL

  org.hibernate.validator.constraints.br.CNPJ.message                 = invalid Brazilian corporate taxpayer registry number (CNPJ)
  org.hibernate.validator.constraints.br.CPF.message                  = invalid Brazilian individual taxpayer registry number (CPF)
  org.hibernate.validator.constraints.br.TituloEleitoral.message      = invalid Brazilian Voter ID card number


.. _Validation_terasoluna_gfw:

共通ライブラリが用意する入力チェックルール
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

共通ライブラリでは、独自の検証用アノテーションを提供している。
ここでは、共通ライブラリで提供しているアノテーションを使用した入力チェックルールの指定方法について説明する。

.. _Validation_terasoluna_gfw_list:

terasoluna-gfw-commonのチェックルール
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ `terasoluna-gfw-common <https://github.com/terasolunaorg/terasoluna-gfw/tree/5.2.0.RELEASE/terasoluna-gfw-common-libraries/terasoluna-gfw-common>`_\ が提供するアノテーション(\ ``org.terasoluna.gfw.common.codelist.*``\ )を以下に示す。

.. tabularcolumns:: |p{0.15\linewidth}|p{0.30\linewidth}|p{0.30\linewidth}|p{0.25\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 15 30 30 25

    * - アノテーション
      - 対象の型
      - 説明
      - 使用例
    * - \ ``@ExistInCodeList``\
      - | \ ``Character``\
        | \ ``CharSequence``\ の実装クラス
        | (\ ``String``\, \ ``StringBuilder``\ など)
      - 値がコードリストに含まれているかどうかを検証する。
      - \ :ref:`@ExistInCodeList <codelist-validate>`\ 参照


terasoluna-gfw-codepointsのチェックルール
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ `terasoluna-gfw-codepoints <https://github.com/terasolunaorg/terasoluna-gfw/tree/5.2.0.RELEASE/terasoluna-gfw-common-libraries/terasoluna-gfw-codepoints>`_\ が提供するアノテーション(\ ``org.terasoluna.gfw.common.codepoints.*``\ )を以下に示す。なお、\ ``terasoluna-gfw-codepoints``\ はバージョン5.1.0.RELEASE以上で利用することができる。

.. tabularcolumns:: |p{0.15\linewidth}|p{0.30\linewidth}|p{0.30\linewidth}|p{0.25\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 15 30 30 25

    * - アノテーション
      - 対象の型
      - 説明
      - 使用例
    * - \ ``@ConsistOf``\
      - | \ ``CharSequence``\ の実装クラス
        | (\ ``String``\, \ ``StringBuilder``\ など)
      - チェック対象の文字列が指定したコードポイント集合に全て含まれるかどうかを検証する。
      - \ :ref:`@ConsistOf <StringProcessingHowToUseCodePointsValidator>`\ 参照


terasoluna-gfw-validatorのチェックルール
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ `terasoluna-gfw-validator <https://github.com/terasolunaorg/terasoluna-gfw/tree/5.2.0.RELEASE/terasoluna-gfw-common-libraries/terasoluna-gfw-validator>`_\ が提供するアノテーション(\ ``org.terasoluna.gfw.common.validator.constraints.*``\ )を以下に示す。なお、\ ``terasoluna-gfw-validator``\ はバージョン5.1.0.RELEASE以上で利用することができる。

.. tabularcolumns:: |p{0.15\linewidth}|p{0.30\linewidth}|p{0.30\linewidth}|p{0.25\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 15 30 30 25
    :class: longtable

    * - アノテーション
      - 対象の型
      - 説明
      - 使用例
    * - \ ``@ByteMin``\
      - | \ ``CharSequence``\ の実装クラス
        | (\ ``String``\, \ ``StringBuilder``\ など)
      - | 値のバイト長が最小値以上であることを検証する。
        |
        | **[アノテーションの属性]**
        | \ ``long value``\  - バイト長の最小値を指定する。
        | \ ``String charset``\  - 値をバイトシーケンスに符号化する際に使用する文字セットを指定する。デフォルト値は\ ``UTF-8``\ 。
      - \ ``@ByteMax``\ 参照
    * - \ ``@ByteMax``\
      - | \ ``CharSequence``\ の実装クラス
        | (\ ``String``\, \ ``StringBuilder``\ など)
      - | 値のバイト長が最大値以下であることを検証する。
        |
        | **[アノテーションの属性]**
        | \ ``long value``\  - バイト長の最大値を指定する。
        | \ ``String charset``\  - 値をバイトシーケンスに符号化する際に使用する文字セットを指定する。デフォルト値は\ ``UTF-8``\ 。
      - .. code-block:: java

             @ByteMin(1)
             @ByteMax(value = 100,
                     charset = "Shift_JIS")
             private String id;

    * - \ ``@Compare``\
      - | \ ``Comparable``\ インタフェースの実装クラスをプロパティにもつ任意のJavaBeanに適用可能
      - | 指定したプロパティの値の大小関係が正しいことを検証する。
        |
        | **[アノテーションの属性]**
        | \ ``String left``\  - オブジェクト内の比較元としたいプロパティ名を指定する。検証エラーとなった場合は、このプロパティにメッセージを表示される。
        | \ ``String right``\  - オブジェクト内の比較先としたいプロパティ名を指定する。
        | \ ``org.terasoluna.gfw.common.validator.constraints.Compare.Operator operator``\  - 期待する大小関係を示す列挙型\ ``Operator``\ の値を指定する。指定可能な値は以下の通り。

        * \ ``EQUAL``\  : \ ``left = right``\ である
        * \ ``GREATER_THAN``\  : \ ``left > right``\ である
        * \ ``GREATER_THAN_OR_EQUAL``\  : \ ``left >= right``\ である
        * \ ``LESS_THAN``\  : \ ``left < right``\ である
        * \ ``LESS_THAN_OR_EQUAL``\  : \ ``left <= right``\ である

        | \ ``boolean requireBoth``\  - \ ``left``\ 属性と\ ``right``\ 属性で指定したフィールドの両方が入力されている（\ ``null``\ でない）必要があるかどうかを指定する。

        * \ ``true``\  : どちらか一方だけ入力されている場合は検証エラーとする。ただし、両方とも未入力の場合は検証成功とする
        * \ ``false``\  : どちらか一方でも入力されている場合は検証成功とする（デフォルト）

        | \ ``org.terasoluna.gfw.common.validator.constraints.Compare.Node node``\  - エラーメッセージを出力するパスを示す列挙型\ ``Node``\ の値を指定する。指定可能な値は以下の通り。

        * \ ``PROPERTY``\  : \ ``left``\ 属性で指定したフィールドのエラーとして出力する（デフォルト）
        * \ ``ROOT_BEAN``\  : チェックを実施したオブジェクトのエラーとして出力する

      - メールアドレスと確認用に入力したメールアドレスが一致することをチェックし、フォーム全体のエラーメッセージとして表示する場合、以下のように実装する。

        .. code-block:: java

             @Compare(left = "email",
                     right = "confirmEmail",
                     operator = Compare.Operator.EQUAL,
                     requireBoth = true,
                     node = Compare.Node.ROOT_BEAN)
             public class UserRegisterForm {
                 private String email;
                 private String confirmEmail;
             }

        |

        期間の開始日と終了日が両方入力された場合は、開始日が終了日以前であることをチェックし、期間の開始日にエラーメッセージを表示する場合、以下のように実装する。

        .. code-block:: java

             @Compare(left = "form",
                     right = "to",
                     operator = Compare.Operator.LESS_THAN_OR_EQUAL)
             public class Period {
                 private Date from;
                 private Date to;
             }

.. raw:: latex

   \newpage

.. note:: **相関項目チェックにおける入力必須について**
  
  単項目チェックにおいては、入力フィールドが入力されている（ \ ``null``\ でない）かどうかは \ ``@NotNull``\ を併用してチェックすればよい。しかし、相関項目チェックにおいては、「どちらか一方でも入力した場合は、もう一方の入力を強制する」といった、 \ ``@NotNull``\ の併用だけでは実現できない場合がある。このため、\ ``@Compare``\ では、チェック対象の入力必須を制御する \ ``requireBoth``\ 属性を提供しており、これを併用して要件に応じたチェックを実装することができる。

  なお、入力フィールドが未入力の場合に \ ``null``\ がバインドされる場合のみ、 \ ``requireBoth``\ 属性が利用できる。Spring MVCでは文字列の入力フィールドに未入力の状態でフォームを送信した場合、デフォルトでは、フォームオブジェクトに\ ``null``\ ではなく、空文字がバインドされることに注意しなければならない。  文字列フィールドが未入力の場合に、空文字ではなく、\ ``null``\ をフォームオブジェクトにバインドするには、\ :ref:`Validation_string_trimmer_editor`\ を参照されたい。
  
  期間の開始日が終了日以前であることのチェックを例に、想定されるチェック要件と設定の例を以下に示す。
  
    .. tabularcolumns:: |p{0.50\linewidth}|p{0.50\linewidth}|
    .. list-table::
        :header-rows: 1
        :widths: 50 50
        :class: longtable

        * - チェック要件
          - 設定例
        * - \ ``from``\ と \ ``to``\ がともに必須で、\ ``from``\ と \ ``to``\ の比較チェックを行う。
          - \ ``from``\ と \ ``to``\ に \ ``@NotNull``\ を付与し、 \ ``requireBoth``\ 属性はデフォルト値（ \ ``false``\ ）を使用する。

            .. code-block:: java

                @Compare(left = "from", right = "to", operator = Compare.Operator.LESS_THAN_OR_EQUAL)
                public class Period {
                  @NotNull
                  LocalDate from;
                  @NotNull
                  LocalDate to;
                }

        * - \ ``from``\ だけ必須だが、 \ ``to``\ も入力された時は比較チェックする。
          - \ ``from``\ にだけ \ ``@NotNull``\ を付与し、 \ ``requireBoth``\ 属性はデフォルト値（ \ ``false``\ ）を使用する。

            .. code-block:: java

                @Compare(left = "from", right = "to", operator = Compare.Operator.LESS_THAN_OR_EQUAL)
                public class Period {
                  @NotNull
                  LocalDate from;
                  LocalDate to;
                }

        * - \ ``from``\ と \ ``to``\ がともに必須ではなく、 \ ``from``\ と \ ``to``\ が両方入力された時だけ比較チェックする。どちらか一方だけが入力された場合は比較チェックを行わない。
          - \ ``@NotNull``\ は付与せず、 \ ``requireBoth``\ 属性はデフォルト値（ \ ``false``\ ）を使用する。

            .. code-block:: java

                @Compare(left = "from", right = "to", operator = Compare.Operator.LESS_THAN_OR_EQUAL)
                public class Period {
                  LocalDate from;
                  LocalDate to;
                }

        * - \ ``from``\ と \ ``to``\ がともに必須ではないが、 \ ``from``\ か \ ``to``\ のどちら一方でも入力した場合は、必ず両方入力して比較チェックを行う。
          - \ ``@NotNull``\ は付与せず、 \ ``requireBoth``\ 属性に \ ``true``\ を設定する。

            .. code-block:: java

                @Compare(left = "from", right = "to", operator = Compare.Operator.LESS_THAN_OR_EQUAL, requireBoth = true)
                public class Period {
                  LocalDate from;
                  LocalDate to;
                }

|

.. _Validation_terasoluna_gfw_how_to_use:

.. raw:: latex

   \newpage

共通ライブラリのチェックルールの適用方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

以下の手順で、共通ライブラリのチェックルールを適用する。

使用したいチェックルールに応じて、依存ライブラリを追加する。\ ``terasoluna-gfw-validator``\ を追加する例を以下に示す。

.. code-block:: xml

    <dependencies>
        <dependency>
            <groupId>org.terasoluna.gfw</groupId>
            <artifactId>terasoluna-gfw-validator</artifactId>
        </dependency>
    </dependencies>

.. note::

    上記設定例は、依存ライブラリのバージョンを親プロジェクトである terasoluna-gfw-parent で管理する前提であるため、pom.xmlでのバージョンの指定は不要である。

次に、\ :ref:`Validation_message_in_validationmessages`\ で説明したように :file:`ValidationMessages.properties` に、アノテーションに対応する任意のメッセージ定義を追加する。

.. code-block:: properties

  # (1)
  org.terasoluna.gfw.common.validator.constraints.ByteMin.message = must be greater than or equal to {value} bytes
  org.terasoluna.gfw.common.validator.constraints.ByteMax.message = must be less than or equal to {value} bytes
  org.terasoluna.gfw.common.validator.constraints.Compare.message = invalid combination of {left} and {right}

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - アノテーションごとにメッセージ定義を追加する。アノテーションの属性値は、プレースフォルダー（\ ``{属性名}``\ の形式）を使用してメッセージの中に埋め込むことができる。

最後に、\ :ref:`Validation_basic_validation`\ で説明したように、JavaBeanのプロパティにアノテーションを付与する。

.. note::

    Bean Validationでは、アノテーションの属性値の不正により検証が実行できない場合、\ ``javax.validation.ValidationException``\ がスローされる。スタックトレースに出力される原因を参照し、属性値を適切な値に修正すること。
    
    詳細は、\ `Bean Validation specification <http://download.oracle.com/otn-pub/jcp/bean_validation-1_1-fr-eval-spec/bean-validation-specification.pdf>`_\ の9章を参照されたい。


.. _Validation_terasoluna_gfw_how_to_extend:

共通ライブラリのチェックルールの拡張方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

共通ライブラリで提供しているチェックルールを利用して、任意のルールを作成することができる。

以下では、\ :ref:`Validation_correlation_item_check`\ で独自に実装した\ ``@Confirm``\ アノテーションを、共通ライブラリで提供しているチェックルールを利用して作成する例を紹介する。

\ :ref:`Validation_convine_existing_constraint`\ で説明したように、\ ``@Compare``\ を利用して\ ``@Confirm``\ アノテーションを作成する。

.. code-block:: java

    package com.example.sample.domain.validation;

    import static java.lang.annotation.ElementType.ANNOTATION_TYPE;
    import static java.lang.annotation.ElementType.TYPE;
    import static java.lang.annotation.RetentionPolicy.RUNTIME;
    
    import java.lang.annotation.Documented;
    import java.lang.annotation.Retention;
    import java.lang.annotation.Target;
    
    import javax.validation.Constraint;
    import javax.validation.OverridesAttribute;
    import javax.validation.Payload;
    import javax.validation.ReportAsSingleViolation;
    
    import org.terasoluna.gfw.common.validator.constraints.Compare;
    
    @Documented
    @Constraint(validatedBy = {})
    @Target({ TYPE, ANNOTATION_TYPE }) // (1)
    @Retention(RUNTIME)
    @ReportAsSingleViolation // (2)
    @Compare(left = "", right = "", operator = Compare.Operator.EQUAL, requireBoth = true) // (3)
    public @interface Confirm {
    
        String message() default "{com.example.sample.domain.validation.Confirm.message}"; // (4)
    
        Class<?>[] groups() default {};
    
        Class<? extends Payload>[] payload() default {};
    
        @OverridesAttribute(constraint = Compare.class, name = "left") // (5)
        String field();
    
        @OverridesAttribute(constraint = Compare.class, name = "right") // (6)
        String confirmField();
    
        @Documented
        @Target({ TYPE, ANNOTATION_TYPE })
        @Retention(RUNTIME)
        public @interface List {
            Confirm[] value();
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | このアノテーションを付与できる場所を、クラスまたはアノテーションに限定する。
    * - | (2)
      - | エラー時にこのアノテーションの\ ``message``\ 属性に指定したメッセージが使用されるようにする。
    * - | (3)
      - | \ ``@Compare``\ アノテーションの\ ``operator``\ 属性に\ ``Compare.Operator.EQUAL``\ (同値であること)を指定する。どちらか一方が未入力の場合はエラーとするため、\ ``requireBoth``\ 属性に\ ``true``\ を指定する。
    * - | (4)
      - | エラーメッセージのデフォルト値を定義する。
    * - | (5)
      - | \ ``@Compare``\ アノテーションの\ ``left``\ 属性をオーバーライドし、属性名を\ ``field``\ に変更する。
    * - | (6)
      - | 同様に\ ``right``\ 属性をオーバーライドし、属性名を\ ``confirmField``\ に変更する。

\ :ref:`Validation_correlation_item_check`\ で実装したアノテーションの代わりに、上記で作成したアノテーションを使用する。

.. code-block:: java

    package com.example.sample.app.validation;

    import java.io.Serializable;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;

    import com.example.common.validation.Confirm;

    @Confirm(field = "password", confirmField = "confirmPassword") // (1)
    public class PasswordResetForm implements Serializable {
        private static final long serialVersionUID = 1L;

        @NotNull // (2)
        @Size(min = 8)
        private String password;

        @NotNull // (3)
        private String confirmPassword;

        // omitted geter/setter
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | クラスレベルに\ ``@Confirm``\ アノテーションを付与する。
    * - | (2)
      - | \ ``password``\ フィールドが\ ``null``\ の場合は\ ``@Confirm``\ の検証はパスするため、\ ``null``\ チェックは\ ``@NotNull``\ アノテーションを付与して行う。
    * - | (3)
      - | 同様に\ ``confirmPassword``\ フィールドにも、\ ``@NotNull``\ アノテーションを付与する。

.. _Validation_type_mismatch:

型のミスマッチ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

フォームオブジェクトの\ ``String``\ 以外のフィールドに対して、変換不可能な値を送信した場合は\ ``org.springframework.beans.TypeMismatchException``\ がスローされる。

「新規ユーザー登録」処理の例では「Age」フィールドは\ ``Integer``\ で定義されているが、このフィールドに対して整数に変換できない値を入力すると、以下のようなエラーメッセージが表示される。

.. figure:: ./images_Validation/validations-typemismatch1.png
  :width: 60%

例外の原因がそのまま表示されてしまい、エラーメッセージとしては不適切である。
型がミスマッチの場合のエラーメッセージは、\ ``org.springframework.context.MessageSource``\ が読み込むpropertiesファイル(application-messages.properties)に定義できる。

以下のルールで、エラーメッセージを定義すればよい。

.. tabularcolumns:: |p{0.35\linewidth}|p{0.40\linewidth}|p{0.25\linewidth}|
.. list-table::
    :widths: 35 40 25
    :header-rows: 1

    * - メッセージキー
      - メッセージ内容
      - 用途
    * - \ ``typeMismatch``\
      - 型ミスマッチエラーのデフォルトメッセージ
      - システム全体のデフォルト値
    * - \ ``typeMismatch.対象のFQCN``\
      - 特定の型ミスマッチエラーのデフォルトメッセージ
      - システム全体のデフォルト値
    * - \ ``typeMismatch.フォーム属性名.プロパティ名``\
      - 特定のフォームのフィールドに対する型ミスマッチエラーのメッセージ
      - 画面毎に変更したいメッセージ

application-messages.propertiesに以下の定義を行った場合、

.. code-block:: properties

  # typemismatch
  typeMismatch="{0}" is invalid.
  typeMismatch.int="{0}" must be an integer.
  typeMismatch.double="{0}" must be a double.
  typeMismatch.float="{0}" must be a float.
  typeMismatch.long="{0}" must be a long.
  typeMismatch.short="{0}" must be a short.
  typeMismatch.java.lang.Integer="{0}" must be an integer.
  typeMismatch.java.lang.Double="{0}" must be a double.
  typeMismatch.java.lang.Float="{0}" must be a float.
  typeMismatch.java.lang.Long="{0}" must be a long.
  typeMismatch.java.lang.Short="{0}" must be a short.
  typeMismatch.java.util.Date="{0}" is not a date.

  # filed names
  name=Name
  email=Email
  age=Age

エラーメッセージは、次のように変更される。

.. figure:: ./images_Validation/validations-typemismatch2.png
  :width: 60%


| \ :ref:`Validation_message_in_application_messages`\ で説明したように、\ ``{0}``\ でフィールド名を埋めることができる。
| \ **基本的にデフォルトメッセージは定義しておくこと**\ 。

.. tip::

  メッセージキーのルールの詳細は、\ `DefaultMessageCodesResolverのJavadoc <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/validation/DefaultMessageCodesResolver.html>`_\ を参照されたい。


.. _Validation_string_trimmer_editor:

文字列フィールドが未入力の場合にnullをバインドする
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
これまで説明してきたように、Spring MVCでは文字列の入力フィールドに未入力の状態でフォームを送信した場合、
デフォルトでは、フォームオブジェクトに\ ``null``\ ではなく、空文字がバインドされる。

この場合、「未入力は許容するが、入力された場合は6文字以上であること」という要件を、既存のアノテーションで満たすことができない。

| 文字列フィールドが未入力の場合に、空文字ではなく、\ ``null``\ をフォームオブジェクトにバインドするには、
| 以下のように\ ``org.springframework.beans.propertyeditors.StringTrimmerEditor``\ を使用すればよい。

.. code-block:: java

  @Controller
  @RequestMapping("xxx")
  public class XxxController {

      @InitBinder
      public void initBinder(WebDataBinder binder) {
          // bind empty strings as null
          binder.registerCustomEditor(String.class, new StringTrimmerEditor(true));
      }

      // omitted ...
  }

この設定により、Controller毎に空文字を\ ``null``\ とみなすかどうかを設定できる。

プロジェクト全体で空文字を\ ``null``\ にしたい場合は、プロジェクト共通設定として\ :ref:`@ControllerAdvice <application_layer_controller_advice>`\ で設定すればよい。

.. tip:: **Spring Framework 4.0 より追加された@ControllerAdviceアノテーションの属性について**

    \ ``@ControllerAdvice``\ アノテーションの属性を指定することで、
    \ ``@ControllerAdvice``\ が付与されたクラスで実装したメソッドを適用するControllerを柔軟に指定できるように改善されている。
    属性の詳細については、\ :ref:`@ControllerAdviceの属性 <application_layer_controller_advice_attribute>`\ を参照されたい。

.. code-block:: java

  @ControllerAdvice
  public class XxxControllerAdvice {

      @InitBinder
      public void initBinder(WebDataBinder binder) {
          // bind empty strings as null
          binder.registerCustomEditor(String.class, new StringTrimmerEditor(true));
      }

      // omitted ...
  }

| この設定を行った場合は、フォームオブジェクトの文字列フィールドに設定される空文字がすべて\ ``null``\ になる。
| したがって、必須チェックに、かならず\ ``@NotNull``\ が必要であることに注意しないといけない。


.. _Validation_without_native2ascii:

Native to Asciiを行わないメッセージの読み込み
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Native to Asciiを行わずにBean Validationのメッセージ（\ ``ValidationMessage.properties``\ ）を読み込む方法紹介する。

日本語メッセージをNative to Asciiせずに直接扱いたい場合、Springの \ ``MessageSource``\ と連携すると簡単に実装することができる。


以下のように定義すると、\ ``MessageSource``\ の機能で読み込まれたメッセージが\ ``Hibernate Validator``\ の中で
使用されるようになる。

* Bean定義

  \ ``*-domain.xml``\ 

 .. code-block:: xml

     <!-- (1) -->
     <bean id="validator" class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean">
         <property name="validationMessageSource">
             <!-- (2) -->
             <bean class="org.springframework.context.support.ResourceBundleMessageSource">
                 <property name="basenames">
                     <list>
                         <value>ValidationMessages</value> <!-- (3) -->
                     </list>
                 </property>
                 <property name="defaultEncoding" value="UTF-8" />
             </bean>
         </property>
     </bean>

     <!-- (4) -->
     <bean class="org.springframework.validation.beanvalidation.MethodValidationPostProcessor">
         <property name="validator" ref="validator" />
     </bean>

 \ ``spring-mvc.xml``\ 

 .. code-block:: xml

     <!-- (5) -->
     <mvc:annotation-driven validator="validator">
         <!-- ommited -->
     </mvc:annotation-driven>

     <!-- (6) -->
     <bean class="org.springframework.validation.beanvalidation.MethodValidationPostProcessor">
         <property name="validator" ref="validator" />
     </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - \ ``LocalValidatorFactoryBean``\ をBean定義する。
     * - | (2)
       - \ ``MessageSource``\ の定義。ここでは\ ``ResourceBundleMessageSource``\ を使用する。
     * - | (3)
       - \ ``ApplicationContext``\ に読み込ませるリソースバンドルを指定する。
     * - | (4)
       - | \ :ref:`MethodValidation`\ を利用する際には、\ ``MethodValidationPostProcessor``\ の\ ``validator``\ プロパティに(1)で定義したBeanを指定する。  
         | Method Validationを利用しない場合、このBean定義は不要である。
     * - | (5)
       - \ ``<mvc:annotation-driven>``\ 要素の\ ``validator``\ 属性に、(1)で定義したBeanを指定する。
     * - | (6)
       - \ (4)と同様である。

 .. note::

     \ ``MessageSource``\ の機能を利用することで、
     プロパティファイルの配置先がクラスパス直下に制限されなくなる。また、複数のプロパティファイルを指定することもできるようになる。

.. _Validation_os_command_injection:

OSコマンドインジェクション対策
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
ここでは、セキュリティ脆弱性の一種であるOSコマンドインジェクションとその対策について説明する。

OSコマンドインジェクションとは
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
OSコマンドインジェクションとは、アプリケーション内でユーザー入力文字列からコマンド実行文字列を組み立てている箇所がある場合に、
ユーザー入力文字列の中に悪意のあるコマンドが送り込まれると、コンピュータを不正に操られてしまう問題である。

.. tip::

    詳細は、OWASPの\ `解説ページ <https://www.owasp.org/index.php/OS_Command_Injection>`_\ などを参照されたい。

Javaでは\ ``ProcessBuilder``\ クラスや、\ ``Runtime``\ クラスの\ ``exec``\ メソッドを用いてコマンドを実行する際に、実行するコマンドとして以下のものを利用する場合に、
OSコマンドインジェクションが発生する可能性がある。

* \ ``/bin/sh``\ （Unix系の場合）や\ ``cmd.exe``\（Windowsの場合）
* ユーザーが入力した文字列

以下では、\ ``/bin/sh``\ を利用する場合にOSコマンドインジェクションが発生する例を示す。

.. code-block:: java

  ProcessBuilder pb = new ProcessBuilder("/bin/sh", "-c", script); // (1)
  Process p = pb.start();


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 例えば、\ ``script``\ に"exec.sh ; cat /etc/passwd" が入ると、文字列中のセミコロンが\ ``/bin/sh``\ により区切り文字として解釈され、"cat /etc/passwd"が実行される。
        | そのため、標準出力の扱い方によっては\ ``/etc/passwd``\ が出力される可能性がある。

.. warning:: **ScriptEngineやScriptTemplateViewResolverの利用について**

    Java SE 6より追加された\ ``ScriptEngine``\ や、Spring Framework 4.2より追加された\ ``ScriptTemplateViewResolver``\ では、
    JVM上で別言語（\ ``Ruby``\ や\ ``Python``\ など）を使用することができる。

    これらの機能を利用して別言語のコードを実行する場合、コードの書き方によってはOSコマンドインジェクションが発生する可能性があるため、
    利用には十分注意すること。

対策方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
OSコマンドインジェクションを起こさないためには、可能な限り外部プロセスの実行を避ける。ただし諸般の事情により外部プロセスの実行がどうしても必要な場合、
以下の対策を行った上で外部プロセス実行を実装すること。

* 極力、\ ``/bin/sh``\ （Unix系の場合）や\ ``cmd.exe``\ （Windowsの場合）を使用したコマンド実行を行わない
* ユーザーにより入力された文字が、アプリケーションとして許可されたものであるかをホワイトリスト方式を用いてチェックする

以下では、ユーザーが入力したコマンドと引数が指定された文字列で構成されているかをホワイトリスト方式でチェックするルールの例を示す。

.. code-block:: java

    @Pattern(regexp = "batch0\\d\\.sh") // (1)
    private String cmdStr;

    @Pattern(regexp = "[\\w=_]+")  // (2)
    private String arg;


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | コマンドとして \ ``batch0X.sh``\ （Xは0から9までの半角数字）のみ許可するルールを指定する。
    * - | (2)
      - | 引数として、無害な文字である半角英数字（\\w）、\ ``=``\ 、\ ``_``\ から構成された文字列のみ許可するルールを指定する。

.. note::

    この例では、コマンドや引数にパスが含まれないようなルールとすることで、ディレクトリトラバーサルを起こさないようにしている。
      
\ ``@Pattern``\ を利用する場合、\ ``@Pattern``\ に指定された正規表現がそのままエラーメッセージとして出力され、
以下の点でメッセージとしては不適切である。

* エラーの意味が不明確となり、ユーザに優しくない
* 脆弱性への対策のためのロジックが利用者に露呈してしまう

.. figure:: ./images_Validation/validations-os-command-injection.png
  :width: 60%

エラーの意味を明確にし、かつ、ロジックを隠蔽するために、application-messages.propertiesに適切なメッセージを定義する。
メッセージの定義方法については、\ :ref:`Validation_message_in_application_messages`\ を参照されたい。

.. code-block:: properties

  Pattern.cmdForm.cmdStr = permit command name: batch00.sh - batch09.sh
  Pattern.cmdForm.arg = permit parameter characters and symbols: alphanumeric, =, _

.. figure:: ./images_Validation/validations-os-command-injection2.png
  :width: 60%

.. raw:: latex

   \newpage

