Thymeleafにおける画面レイアウト
================================================================================

.. only:: html

.. contents:: 目次
  :depth: 4
  :local:

Overview
--------------------------------------------------------------------------------

.. _templatelayout_overview_fragment:

Thymeleafのテンプレートレイアウト機能を使用したHTMLの部品化
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Thymeleafのテンプレートレイアウト機能を使用すると共通的に使用するHTMLを部品化することが出来る。
| Thymeleafでは、共通的なHTMLを部品化して使用するための以下の機能がある。

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
  .. list-table:: \ **HTML部品の定義と参照のための機能**\
    :header-rows: 1
    :widths: 10 35 55

    * - 項番
      - 属性
      - 説明
    * - 1.
      - | \ ``th:fragment``\
      - | HTMLを部品化するための属性。部品化されたHTMLはフラグメントと呼ぶ。
    * - 2.
      - | \ ``th:insert``\
      - | フラグメントを参照するための属性。
        | 指定したタグに参照したフラグメントを挿入することができる。指定したタグ自体は残るが、そのタグの子要素は残らない。
    * - 3.
      - | \ ``th:replace``\
      - | フラグメントを参照するための属性。
        | 指定したタグを参照したフラグメントで置換することができる。指定したタグ自体も残らない。

  .. tip::

    上記以外に\ ``th:include``\ 属性が使用可能だが、Thymeleaf3.2で削除される予定のため\ ``th:include``\ 属性を使用することは推奨しない。

    \ ``th:include``\ 属性を指定すると、\ ``th:fragment``\ 属性を設定したタグの子要素のみが挿入される。

    \ ``th:include``\ 属性と同様の処理を行いたい場合は、\ ``th:fragment``\ 属性を設定したタグに\ ``th:remove="tag"``\ を設定した上で、\ ``th:insert``\ 属性を使用することで実現可能である。


| また、HTMLの部品化と関連して、フラグメント式という機能がある。
| フラグメント式は変数式などと同様にテンプレートHTML内で使用することができ、テンプレートHTMLの断片を取得することができる。
| \ ``th:insert``\ 属性、\ ``th:replace``\ 属性と組み合わせて使うことで、フラグメントやテンプレートHTMLの一部を他のテンプレートHTMLに埋め込むことができる。

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
  .. list-table:: \ **フラグメント式の記述パターン**\
    :header-rows: 1
    :widths: 10 35 55

    * - 項番
      - 式
      - 説明
    * - 1.
      - | \ ``~{Viewのパス :: セレクタ}``\
      - | \ ``Viewのパス``\ で指定されたテンプレートHTMLから、指定された\ ``セレクタ``\ に該当する要素を特定し取得する。
    * - 2.
      - | \ ``~{Viewのパス :: フラグメント名}``\
      - | \ ``Viewのパス``\ で指定されたテンプレートHTMLから、指定された\ ``フラグメント名``\ に一致する\ ``th:fragment``\ 属性が設定された要素を特定し取得する。
    * - 3.
      - | \ ``~{Viewのパス}``\
      - | \ ``Viewのパス``\ で指定されたテンプレートHTML全体を取得する。
    * - 4.
      - | \ ``~{:: セレクタ}``\
      - | フラグメント式が記載されているテンプレートHTML自体から、指定された\ ``セレクタ``\ に該当する要素を特定し取得する。
        | \ ``セレクタ``\ の代わりに\ ``フラグメント名``\ を指定することも可能である。
    * - 5.
      - | \ ``~{this :: セレクタ}``\
      - | 項番4の別の記載方法である。

  .. note::

    フラグメント式は、セレクタを使って自由に他のテンプレートHTMLの一部を取得できてしまうので、多用はHTMLの構造を複雑化させる。そのためプロジェクト全体で共通部品として何をフラグメント化するかを決めて使用することが推奨される。

| \ ``th:fragment``\ 属性を利用して定義したフラグメントは、\ ``th:fragment="フラグメント名 (param1,param2)"``\ のようにパラメータを取ることが可能である。
| パラメータを取ることでフラグメントの汎用性を高めることが可能となり、HTML部品の共通化が容易となる。
| パラメータ化されたフラグメントを取得する場合は、\ ``~{Viewのパス :: フラグメント名 (${address},true)}``\ のようにフラグメント式のフラグメント名の後ろに、パラメータとして渡す値を記述する。
| パラメータとして渡す値には、変数式（\ ``${...}``\ ）やリテラル（文字列、真偽値など）のほか、フラグメント式（\ ``~{...}``\ ）を指定することも可能である。
| \ :ref:`TemplateLayoutHowToUse`\ で解説する実装例では、パラメータとしてフラグメント式を渡す方法を利用している。

  .. note::

    パラメータを渡す際に、\ ``~{Viewのパス :: フラグメント名 (param1=${address},param2=true)}``\ のようにパラメータ名を明示的に記述することも可能である。この場合、パラメータを渡す順序はフラグメントで定義したパラメータの順序と異なっていても問題ない。

    なお、本ガイドラインでは可読性と記述の簡潔さを重視し、パラメータ名を記述しない方法を推奨する。

| 上記で紹介したテンプレートレイアウト機能によるHTMLの部品化を応用して共通的な画面レイアウトを作成する方法を次節以降で説明する。
| なお、実際に業務アプリケーションを作成する場合、共通的な画面レイアウトの構成以外にも画面を構成するHTMLの一部について汎用的なHTML部品（フラグメント）を作成することが考えられる。
| その方法については、「\ :ref:`geraral_html_fragments`\ 」 で解説する。

|

共通的な画面レイアウトの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| ヘッダ、フッタ、サイドメニューといった共通的なレイアウトを持つWebアプリケーションを開発する場合に、全てのテンプレートHTMLに共通部分をコーディングすると、メンテナンスが煩雑になる。
| 例えば、ヘッダのデザインを修正する必要がある場合、全てのテンプレートHTMLに修正を加えなければならない。
| Thymeleafでは、テンプレートレイアウト機能を使うことで、Apache Tilesのように統一的なレイアウトを構成することができる。

| 多くの画面で同じレイアウトを使用する場合は、Thymeleafのテンプレートレイアウト機能を使用して、統一的な画面レイアウトを定義して、個別の画面に適用することを推奨する。
| 理由は、以下3つの通りである。

#. 設計者によるレイアウトの誤差をなくすこと
#. 冗長なコードを減らすこと
#. 大きなレイアウトの変更が容易になること

| 統一的な画面レイアウトの定義を行うことで、別々のテンプレートHTMLを組み合わせることができる。
| その結果、各々のテンプレートHTMLに、余計なコードを記述することがなくなるため、開発者の作業を楽にできる。
| 例えば、下記のようなレイアウト構成が複数の画面に存在する場合、

  .. figure:: ./images_TemplateLayout/screen_layout.png
    :alt: screen layout
    :width: 50%
    :align: center

    \ **Picture - Image of screen layout**\


| 統一的な画面レイアウトの定義を行うことで、同じレイアウトの全ての画面でheaderやmenu、footerを挿入してサイズを指定することなく、bodyの作成のみに集中することができる。
| 実際のHTMLファイルは下記のようになる。

  .. figure:: ./images_TemplateLayout/layout_html.png
    :alt: layout html
    :width: 50%
    :align: center

    \ **Picture - Image of layout html**\

よって、統一的な画面レイアウトを定義した後は、業務に相当するHTMLファイルのみ(business.html)画面毎に作成すればよい。

  .. note::

    Thymeleafのテンプレートレイアウト機能を使用した統一的な画面レイアウトを適用しない方がよい場合もある。
    
    例えば、エラー画面に統一的な画面レイアウトを使用するのは、以下の理由により推奨しない。

    * エラー画面表示中に共通的なレイアウトの部分にエラーが発生すると解析がしにくくなるため。(二重障害発生の場合)

|

.. _TemplateLayoutHowToUse:

How to use
--------------------------------------------------------------------------------

Thymeleafのテンプレートレイアウト機能を使用した画面レイアウト
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

レイアウト作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
以降、以下のファイル構成を前提に画面レイアウトの作成方法を示す。

- File Path

  .. code-block:: console

    WEB-INF
      └─views
         ├─layout
         │   ├─header.html
         │   ├─template.html
         │   └─footer.html
         │
         └─staff
             └─createForm.html

レイアウトの枠となるHTMLファイル（template.html）と、レイアウトに埋め込むHTMLファイルを作成する。

- template.html

  .. code-block:: html

    <!DOCTYPE html>
    <!--/* (1) */-->
    <html class="no-js" xmlns:th="http://www.thymeleaf.org" th:fragment="layout (title,body)">
    <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
    <meta name="viewport" content="width=device-width">
    <link rel="stylesheet" th:href="@{/resources/app/css/styles.css}" type="text/css"
        media="screen, projection">
    <script type="text/javascript">
      
    </script>
    <!--/* (2) */-->
    <title th:replace="${title}">Staff Management System</title>
    </head>
    <body>
        <!--/* (3) */-->
        <div id="header" th:replace="~{layout/header :: header}"></div>
        <!--/* (4) */-->
        <div id="body" th:replace="${body}"></div>
        <!--/* (5) */-->
        <div id="footer" th:replace="~{layout/footer :: footer}"></div>
    </body>
    </html>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | htmlタグ以下全体を\ ``layout``\ という名前でフラグメント化し、個別のテンプレートHTMLと合成できるようにしている。
        | また、個別画面（createForm.html）の\ ``title``\ タグと\ ``body``\ タグの内容をパラメータとして受け取っている。
    * - | (2)
      - | \ ``th:replace``\ 属性を使用して(1)でパラメータとして受け取った個別画面（createForm.html）の\ ``title``\ タグの内容で置換している。
    * - | (3)
      - | \ ``th:replace``\ 属性を使用して後述するlayout/headar.htmlの\ ``header``\ フラグメントで置換している。
    * - | (4)
      - | \ ``th:replace``\ 属性を使用して(1)でパラメータとして受け取った個別画面（createForm.html）の\ ``body``\ タグ内のコンテンツで置換している。
    * - | (5)
      - | \ ``th:replace``\ 属性を使用し、後述するlayout/footer.htmlの\ ``footer``\ フラグメントで置換している。

  .. note::

    \ ``th:replace="~{layout/header :: header}"``\ は、\ ``~{}``\ を省略して、\ ``th:replace="layout/header :: header"``\ と書くこともできるが、本ガイドラインでは可読性を重視し\ ``~{}``\ を省略しないことを推奨する。

- header.html

  .. code-block:: html

    <!DOCTYPE html>
    <html xmlns:th="http://www.thymeleaf.org">
    <head>
    <title>header</title>
    </head>
    <body>
        <!--/* (1) */-->
        <div th:fragment="header" th:remove="tag">
            <h1>
                <a th:href="@{/}">Staff Management System</a>
            </h1>
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
      - | \ ``th:fragment``\ 属性を使用し、\ ``header``\ という名前でフラグメント化している。
        | 画面レイアウトを合成するなかで使用されるのはフラグメント化された部分のみ。
        | ここでは、単独でHTMLとして表示できるように\ ``body``\ タグ以上のタグも記述し仮のタイトルを記述している。
        | また、\ ``th:replace``\ 属性を使用して合成したときに\ ``div``\ タグを残さないため、\ ``th:remove``\ 属性の値に\ ``tag``\ を設定している。

- createForm.html(body部分の例)

  開発者は、個別画面のコンテンツ（主にbody部分）のみに集中して記述できる。

  .. code-block:: html

    <!DOCTYPE html>
    <!--/* (1) */-->
    <html xmlns:th="http://www.thymeleaf.org"
        th:replace="~{layout/template :: layout(~{::title},~{::body/content()})}">
    <head>
    <!--/* (2) */-->
    <title>Create Staff Information</title>
    </head>
    <body>
        <h2>Create Staff Information</h2>
        <form th:object="${staffInfoForm}" method="post" th:action="@{/staff/create}">
            <table>
                <tr>
                    <td>Staff First Name</td>
                    <td><input type="text" th:field="*{firstName}"></td>
                </tr>
                <tr>
                    <td>Staff Family Name</td>
                    <td><input type="text" th:field="*{familyName}"></td>
                </tr>
                <tr>
                    <td rowspan="5">Staff Authorities</td>
                    <td><input type="checkbox" value="01" th:field="*{authorities}"> Staff Management</td>
                </tr>
                <tr>
                    <td><input type="checkbox" value="02" th:field="*{authorities}"> Master Management</td>
                </tr>
                <tr>
                    <td><input type="checkbox" value="03" th:field="*{authorities}"> Stock Management</td>
                </tr>
                <tr>
                    <td><input type="checkbox" value="04" th:field="*{authorities}"> Order Management</td>
                </tr>
                <tr>
                    <td><input type="checkbox" value="05" th:field="*{authorities}"> Show Shopping Management</td>
                </tr>
            </table>
  
            <input type="submit" value="cancel">
            <input type="submit" value="confirm">
        </form>
    </body>
    </html>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``th:replace``\ 属性を使用して、テンプレートであるlayout/template.htmlの\ ``layout``\ フラグメントの内容で\ ``html``\ タグ以下の内容を置換している。
        | \ ``~{::title}``\ は自身のテンプレートHTMLの\ ``title``\ タグを、\ ``~{::body/content()}``\ は自身のテンプレートHTMLの\ ``body``\ タグ内のコンテンツを取得している。
        | そして、自身のHTMLの\ ``title``\ タグ、\ ``body``\ タグをlayout/template.htmlの\ ``layout``\ フラグメントにパラメータとして渡している。
        | そのため、 画面レイアウトを合成するなかで使用されるのはパラメータとして渡した\ ``title``\ タグ、\ ``body``\ タグの内容のみである。
        | ここでは、単独でHTMLとして表示できるように\ ``body``\ タグ以上のタグも記述し仮のタイトルを記述している。
    * - | (2)
      - | template.htmlの\ ``title``\ タグを置き換えるタイトルメッセージを定義している。

  .. note::

    上記の実装例に示した通り、適用するテンプレートを画面ごとに指定する形になっているので、指定するテンプレートを変えることで別のレイアウトを適用することができる。

    複数のテンプレートレイアウトを使い分けることで、同一アプリケーション内で異なる画面レイアウトに対応することが可能となっている。

  .. note::

    個別画面で\ ``title``\ タグを定義せず、テンプレートの\ ``title``\ タグの内容をそのまま使用する場合は、no-operation token("\ ``_``\ ") を使用して、\ ``th:replace="~{layout/template :: layout(_,~{::body/content()})}"``\ と記述する。
    
    no-operation tokenについては、\ :ref:`view_thymeleaf_conditional-label`\ のNoteも参照されたい。

- footer.html

  .. code-block:: html

    <!DOCTYPE html>
    <html xmlns:th="http://www.thymeleaf.org">
    <head>
    <title>footer</title>
    </head>
    <body>
        <div th:fragment="footer" th:remove="tag">
            <!-- (1) -->
            <p style="text-align: center; background: #e5eCf9;">Copyright &copy; 20XX CompanyName</p>
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
      - | \ ``th:fragment``\ 属性を使用し、\ ``footer``\ という名前でフラグメント化している。
        | 画面レイアウトを合成するなかで使用されるのはフラグメント化された部分のみである。
        | ここでは、単独でHTMLとして表示できるように\ ``body``\ タグ以上のタグも記述し仮のタイトルを記述している。
        | また、合成後に\ ``div``\ タグを残さないため、\ ``th:remove``\ 属性の値に\ ``tag``\ を設定している。

  .. note::

    フッターに記載する著作権に関しては\ :ref:`CreateWebApplicationProjectCustomizeCopyrightOnScreenFooter`\ を参照すること。

結果として上記のtemplate.htmlに、header.html、createForm.html、footer.htmlが組み合わされた方法でブラウザに出力される。

  .. figure:: ./images_TemplateLayout/thymeleaf_result.png
    :alt: thymeleaf result
    :width: 100%
    :align: center

出力されるHTMLは以下の通り。

  .. code-block:: html

    <!DOCTYPE html>

    <html class="no-js">
    <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
    <meta name="viewport" content="width=device-width">
    <link rel="stylesheet" href="/staff-management-web/resources/app/css/styles.css" type="text/css"
        media="screen, projection">
    <script type="text/javascript">
      
    </script>

    <title>Create Staff Information</title>
    </head>
    <body>
    
        <h1>
            <a href="/staff-management-web/">Staff Management System</a>
        </h1>
    
        <h2>Create Staff Information</h2>
        <form method="post" action="/staff-management-web/staff/create"><input type="hidden" name="_csrf" value="2557dc95-6f36-4c2c-9900-9e0efd411ad7">
            <table>
                <tr>
                    <td>Staff First Name</td>
                    <td><input type="text" id="firstName" name="firstName" value=""></td>
                </tr>
                <tr>
                    <td>Staff Family Name</td>
                    <td><input type="text" id="familyName" name="familyName" value=""></td>
                </tr>
                <tr>
                    <td rowspan="5">Staff Authorities</td>
                    <td><input type="checkbox" value="01" id="authorities1" name="authorities"><input type="hidden" name="_authorities" value="on"> Staff Management</td>
                </tr>
                <tr>
                    <td><input type="checkbox" value="02" id="authorities2" name="authorities"><input type="hidden" name="_authorities" value="on"> Master Management</td>
                </tr>
                <tr>
                    <td><input type="checkbox" value="03" id="authorities3" name="authorities"><input type="hidden" name="_authorities" value="on"> Stock Management</td>
                </tr>
                <tr>
                    <td><input type="checkbox" value="04" id="authorities4" name="authorities"><input type="hidden" name="_authorities" value="on"> Order Management</td>
                </tr>
                <tr>
                    <td><input type="checkbox" value="05" id="authorities5" name="authorities"><input type="hidden" name="_authorities" value="on"> Show Shopping Management</td>
                </tr>
            </table>
  
            <input type="submit" value="cancel">
            <input type="submit" value="confirm">
        </form>

        <p style="text-align: center; background: #e5eCf9;">Copyright &copy; 20XX CompanyName</p>
    
    </body>
    </html>

.. _TemplateLayoutNoteFragment:

  .. note::

    上記の例では、\ ``body``\ タグ内のコンテンツ以外に\ ``tilte``\ タグを渡しているが、フラグメントに引数を追加することで任意のパラメータを個別画面からテンプレートに渡すことができる。

    以下は\ ``script``\ タグの例である。

    - template.html

      .. code-block:: html

        <!DOCTYPE html>
        <!--/* (1) */-->
        <html class="no-js" xmlns:th="http://www.thymeleaf.org" th:fragment="layout (title,script,body)">
        <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
        <meta name="viewport" content="width=device-width">
        <link rel="stylesheet" th:href="@{/resources/app/css/styles.css}" type="text/css"
            media="screen, projection">
        <!--/* (2) */-->
        <script type="text/javascript" th:replace="${script}">    
        </script>
        <title th:replace="${title}">Staff Management System</title>
        </head>
        <!-- omitted -->
        </html>

      .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
      .. list-table::
        :header-rows: 1
        :widths: 10 90

        * - 項番
          - 説明
        * - | (1)
          - | \ ``layout``\ フラグメントに、個別画面の\ ``script``\ タグの内容をパラメータとして受け取るための引数\ ``script``\ を追加する。
        * - | (2)
          - | \ ``th:replace``\ 属性を使用して、(1)でパラメータとして受け取った\ ``script``\ タグの内容で置換する。
            | なお、タグごと置換されるので\ ``script``\ タグ以外のダミーのタグに同様の設定をしても問題なく動作する。

    - createForm.html(body部分の例)

      .. code-block:: html

        <!DOCTYPE html>
        <!--/* (1) */-->
        <html xmlns:th="http://www.thymeleaf.org"
            th:replace="~{layout/template :: layout(~{::title},~{::script},~{::body/content()})}">
        <head>
        <title>Create Staff Information</title>
        <!--/* (2) */-->
        <script type="text/javascript" th:src="@{/resources/app/js/sample.js}"></script>
        </head>
        <body>
            <h2>Create Staff Information</h2>
            <!-- omitted -->
        </html>

      .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
      .. list-table::
        :header-rows: 1
        :widths: 10 90

        * - 項番
          - 説明
        * - | (1)
          - | \ ``th:replace``\ 属性の\ ``layout``\ フラグメントを指定している箇所にパラメータとしてフラグメント式\ ``~{::script}``\ を追加する。
        * - | (2)
          - | 個別画面固有の\ ``script``\ タグを定義し、使用するJavaScriptファイルを読み込んでいる。

  .. warning::

    テンプレートレイアウトにより部品化する際に、フラグメント（部品）に引数として式を与える場合には、式を\ ``th:object``\ と選択変数式\ ``*{}``\ ではなく、変数式\ ``${}``\ で組み立てなければならない。これは、部品内で\ ``th:object``\ と選択変数式\ ``*{}``\ を使用していても、部品を呼び出す側が与えた式の意味(参照するもの)が変わらないようにするための(裏を返せば、部品内での\ ``th:object``\ と選択変数式\ ``*{}``\ の使用を制限せずに済むための)ルールである。

    こちらの詳細については、\ :ref:`view_thymeleaf_object-label`\ のWarningを参照されたい。

|

How to extend
--------------------------------------------------------------------------------

.. _geraral_html_fragments:

汎用的なHTML部品の作成方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| \ ``th:fragment``\ 属性を使用して汎用的なHTML部品（フラグメント）を作成することができる。
| 本節では、結果メッセージを表示するHTMLを再利用するため、HTML部品（フラグメント）化する例を示す。

|
| \ ``Model``\ に格納された\ ``org.terasoluna.gfw.common.message.ResultMessages``\ を参照し、結果メッセージを出力するフラグメントを作成する。

- common-parts.html

  .. code-block:: html

    <!DOCTYPE html>
    <html xmlns:th="http://www.thymeleaf.org">
    <head>
    <title>common-parts</title>
    </head>
    <body>
        <!--/* (1) */-->
        <div th:fragment="messagesPanel" th:if="${resultMessages} != null"
            th:class="|alert alert-${resultMessages.type}|">
            <ul>
                <!--/* (2) */-->
                <li th:each="message : ${resultMessages}"
                    th:text="${message.code} != null ? ${#messages.msgWithParams(message.code, message.args)} : ${message.text}">blank
                    messages</li>
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
      - | \ ``th:fragment``\ 属性を使用し、\ ``messagesPanel``\ という名前でフラグメント化する。
        | \ ``th:if``\ 属性は、条件に応じて、タグを生成するかどうか制御するための属性である。
        | ここでは、\ ``resultMessages``\ が\ ``null``\ の場合に、結果メッセージを表示するHTMLを生成しないようにしている。
        | また、\ ``th:class``\ 属性を使用し、\ ``ResultMessages``\ に設定されたメッセージタイプ（例:\ ``info``\ , \ ``error``\ ）に応じた\ ``class``\ 属性を設定する。
    * - | (2)
      - | \ ``th:each``\ 属性は、コレクションや配列に対して繰り返し処理を行うための属性である。
        | \ ``ResultMessages``\ には、複数件の結果メッセージを格納できるので、それを1件づつ取得し、\ ``li``\ タグを生成している。
        | メッセージの取得はThymeleafの\ ``#messages``\ を使用する。
        | その\ ``#messages.msgWithParams({メッセージID},{置換文字列})``\ メソッドを使用することで、プロパティファイルからメッセージを取得することができる。

|
| フラグメント化した\ ``messagesPanel``\ を各テンプレートHTMLにおいて使用する。

- businessError.html

  .. code-block:: html

    <!DOCTYPE html>
    <html xmlns:th="http://www.thymeleaf.org">
    <head>
    <meta charset="utf-8">
    <title>Business Error!</title>
    <link rel="stylesheet"
        href="../../../../resources/app/css/styles.css" th:href="@{/resources/app/css/styles.css}">
    </head>
    <body>
        <div id="wrapper">
            <h1>Business Error!</h1>
            <div class="error">
                <span th:text="${#strings.isEmpty(exceptionCode)} ? #{e.xx.fw.8001} : |[${exceptionCode}] #{${exceptionCode}}|"></span>
                <!--/* (1) */-->
                <span th:replace="~{common/common-parts :: messagesPanel}"></span>
            </div>
            <br>
            <!-- omitted -->
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
      - | \ ``th:replace``\ 属性を使用して、"messagesPanel"フラグメントの内容で置換している。

\ ``org.terasoluna.gfw.common.message.ResultMessages``\ に結果メッセージが格納されている場合、結果として、businessError.htmlの一部にmessagesPanelの内容が埋め込まれた形でブラウザに出力される。

  .. figure:: ./images_TemplateLayout/thymeleaf_result_messagesPanel.png
    :alt: thymeleaf result
    :width: 70%
    :align: center

出力されるHTMLは以下の通り。

  .. code-block:: html

    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="utf-8">
    <title>Business Error!</title>
    <link rel="stylesheet"
        href="/staff-management-web/resources/app/css/styles.css" type="text/css">
    </head>
    <body>
        <div id="wrapper">
            <h1>Business Error!</h1>
            <div class="error">
                <span>[e.xx.fw.8001] Business error occurred!</span>
                <div class="alert alert-error">
            <ul>
              
                <li>Create Staff error occurred!</li>
            </ul>
        </div>
            </div>
            <br>
            <!-- omitted -->
        </div>
    </body>
    </html>

.. raw:: latex

  \newpage
