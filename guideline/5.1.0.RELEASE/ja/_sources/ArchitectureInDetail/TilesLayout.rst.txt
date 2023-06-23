Tilesによる画面レイアウト
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

Overview
--------------------------------------------------------------------------------
| ヘッダ、フッタ、サイドメニューといった共通的なレイアウトを持つWebアプリケーションを開発する場合に、全てのJSPに共通部分をコーディングすると、メンテナンスが煩雑になる。
| 例えば、ヘッダのデザインを修正する必要がある場合、全てのJSPに修正を加えなければならない。

| JSPでの開発で多くの画面で同じレイアウトを使用する場合は、 `Apache Tiles <http://tiles.apache.org/>`_\ (以下、Tiles)の利用を推奨する。
| 理由は、以下3つの通りである。

#. 設計者によるレイアウトの誤差をなくすこと
#. 冗長なコードを減らすこと
#. 大きなレイアウトの変更が容易になること

| Tilesは、統一的な画面レイアウトの際に定義を行うことで、別々のJSPを組み合わせることができる。
| その結果、各々のJSPファイルに、余計なコードを記述することがなくなるため、開発者の作業を楽にできる。
| 例えば、下記のようなレイアウト構成が複数の画面に存在する場合、

 .. figure:: ./images/screen_layout.png
    :alt: screen layout
    :width: 50%
    :align: center

    **Picture - Image of screen layout**


| Tilesを使用することにより、同じレイアウトの全ての画面でheaderやmenu、footerをincludeしてサイズを指定することなく、bodyの作成のみに集中することができる。
| 実際のJSPファイルは下記のようになる。

 .. figure:: ./images/layout_jsp.png
    :alt: layout jsp
    :width: 50%
    :align: center

    **Picture - Image of layout jsp**

よって、Tilesで画面レイアウトを設定した後は、業務に相当するJSPのみ(buisness.jsp)画面毎に作成すればよい。

    .. note::

     Tilesの適用をしない方がよい場合もある。例えば、エラー画面にTilesを使用するのは、以下の理由により推奨しない。

     * エラー画面表示中にTilesによるエラーが発生すると解析がしにくくなるため。(二重障害発生の場合)
     * web.xmlの<error-pages>タグで設定するJSPでは、必ずしも画面表示にTilesによるテンプレートが適用されないため。

|

.. _TilesLayoutHowToUse:

How to use
--------------------------------------------------------------------------------

pom.xmlの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| TilesをMavenで使用する場合、以下のdependencyをpom.xmlに追加する必要がある。

.. code-block:: xml

        <dependency>
            <groupId>org.terasoluna.gfw</groupId>
            <artifactId>terasoluna-gfw-recommended-web-dependencies</artifactId><!-- (1) -->
            <type>pom</type><!-- (2) -->
        </dependency>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | webに関連するライブラリ群が定義してあるterasoluna-gfw-recommended-web-dependenciesをdependencyに追加する。
   * - | (2)
     - | terasoluna-gfw-recommended-web-dependenciesは依存関係が定義してあるpomファイルでしかないため、
       | ``<type>pom</type>`` の指定が必要である。

|

    .. note::
        pom.xmlは、以下のようにterasoluna-gfw-parentの設定がされている前提である。

|

      .. code-block:: xml

          <parent>
              <groupId>org.terasoluna.gfw</groupId>
              <artifactId>terasoluna-gfw-parent</artifactId>
              <version>x.y.z</version>
          </parent>

    そのため、terasoluna-gfw-recommended-web-dependenciesの ``<version>`` の指定は不要である。

|

Spring MVCとTilesの連携
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| Spring MVCとTilesを連携するには ``org.springframework.web.servlet.view.tiles3.TilesViewResolver`` を利用すればよい。
| Spring MVCのControllerの実装(View名の返却)を変更する必要は無い。

設定方法について、以下に示す。

**Beanの定義(ViewResolver、TilesConfigurer)**

- spring-mvc.xml

 .. code-block:: xml

    <mvc:view-resolvers>
        <mvc:tiles /> <!-- (1) -->
        <mvc:jsp prefix="/WEB-INF/views/" /> <!-- (2) -->
    </mvc:view-resolvers>

    <!-- (3) -->
    <mvc:tiles-configurer>
        <mvc:definitions location="/WEB-INF/tiles/tiles-definitions.xml" />
    </mvc:tiles-configurer>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - Spring Framework 4.1から追加された\ ``<mvc:tiles>``\ 要素を使用して、\ ``TilesViewResolver``\ を定義する。

       \ ``<mvc:jsp>``\ 要素より上に定義することで、最初にTiles定義ファイル(:file:`tiles-definitions.xml`)を参照して\ ``View``\を解決するようにする。
       Controllerから返却されたView名が、Tiles定義ファイル内の\ ``definition``\ 要素の\ ``name``\ 属性のパターンに合致する場合、\ ``TilesViewResolver``\ によって\ ``View``\が解決される。
   * - | (2)
     - Spring Framework 4.1から追加された\ ``<mvc:jsp>``\ 要素を使用して、JSP用の\ ``InternalResourceViewResolver``\ を定義する。

       \ ``<mvc:tiles>``\ 要素より下に定義することで、\ ``TilesViewResolver``\で解決できなかったView名のみ、JSP用の\ ``InternalResourceViewResolver``\を使用して\ ``View``\を解決するようにする。
       View名に対応するJSPファイルが、\ ``/WEB-INF/views/``\ 配下に存在する場合、JSP用の\ ``InternalResourceViewResolver``\ によって\ ``View``\が解決される。
   * - | (3)
     - Spring Framework 4.1から追加された\ ``<mvc:tiles-configurer>``\ 要素を使用して、Tiles定義ファイルを読み込む。

       \ ``<mvc:definitions>``\ 要素の\ ``location``\ 属性に、Tiles定義ファイルを指定する。


 .. tip::

    \ ``<mvc:view-resolvers>``\ 要素はSpring Framework 4.1から追加されたXML要素である。
    \ ``<mvc:view-resolvers>``\ 要素を使用すると、\ ``ViewResolver``\ をシンプルに定義することが出来る。

    従来通り\ ``<bean>``\ 要素を使用した場合の定義例を以下に示す。


     .. code-block:: xml
        :emphasize-lines: 1-13

        <bean id="tilesViewResolver"
            class="org.springframework.web.servlet.view.tiles3.TilesViewResolver">
            <property name="order" value="1" />
        </bean>

        <bean id="tilesConfigurer"
            class="org.springframework.web.servlet.view.tiles3.TilesConfigurer">
            <property name="definitions">
                <list>
                    <value>/WEB-INF/tiles/tiles-definitions.xml</value>
                </list>
            </property>
        </bean>

        <bean id="viewResolver"
            class="org.springframework.web.servlet.view.InternalResourceViewResolver">
            <property name="prefix" value="/WEB-INF/views/" />
            <property name="suffix" value=".jsp" />
            <property name="order" value="2" />
        </bean>

    \ ``order``\ プロパティに、\ ``InternalResourceViewResolver``\ より小さい値を指定し、優先度を高くする。


**Tilesの定義**

- tiles-definitions.xml

 .. code-block:: guess

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE tiles-definitions PUBLIC
       "-//Apache Software Foundation//DTD Tiles Configuration 3.0//EN"
       "http://tiles.apache.org/dtds/tiles-config_3_0.dtd"> <!-- (1) -->

    <tiles-definitions>
        <definition name="layouts"
            template="/WEB-INF/views/layout/template.jsp"> <!-- (2) -->
            <put-attribute name="header"
                value="/WEB-INF/views/layout/header.jsp" /> <!-- (3) -->
            <put-attribute name="footer"
                value="/WEB-INF/views/layout/footer.jsp" /> <!-- (4) -->
        </definition>

        <definition name="*/*" extends="layouts"> <!-- (5) -->
            <put-attribute name="title" value="title.{1}.{2}" /> <!-- (6) -->
            <put-attribute name="body" value="/WEB-INF/views/{1}/{2}.jsp" /> <!-- (7) -->
        </definition>
    </tiles-definitions>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | tilesのdtdを定義する。
   * - | (2)
     - | レイアウト構成の親定義。
       | template属性には、レイアウトを定義しているjspファイルを指定する。
   * - | (3)
     - | headerを定義しているjspファイルを指定する。
   * - | (4)
     - | footerを定義しているjspファイルを指定する。
   * - | (5)
     - | 描画のリクエストの際にnameのパターンと同じ場合に呼ばれるレイアウト定義。
       | extendsしている layouts定義も適用される。
   * - | (6)
     - | タイトルを指定する。
       | valueはspring-mvcに取り込まれているpropertiesの中から取得する。(以下の説明では application-messages.propertiesに設定する。)
       | {1},{2}はリクエストの"\*/\*"の「*」の1つ目、2つ目に該当する。
   * - | (7)
     - | bodyを定義しているjspファイルの置き場所について、{1}にリクエストパス、{2}にJSP名が一致するように設計する。
       | これにより、リクエストごとの定義を記述する手間を省くことができる。

 .. note::

     Tilesの適用をしたくない画面(エラー画面等)の場合、Tiles使用対象にならないようなファイル構成にする必要がある。
     ブランクプロジェクトでは、エラー画面に InternalResourceViewResolverが使われるように("\*/\*"形式にならないように)、 /WEB-INF/views/common/error/xxxError.jsp形式にしている。

- `application-messages.properties`

 .. code-block:: properties

  title.staff.createForm = Create Staff Information

 .. note::
   メッセージプロパティファイルの記載方法については、 :doc:`MessageManagement` を参照されたい。


Tilesを設定したときのファイル構成を以下に示す。

- tiles File Path

 .. figure:: ./images/tiles_filepath.png
   :alt: tiles file path

**カスタムタグの設定**


Tilesを使用するためにカスタムタグ(TLD)を設定する必要がある。

- /WEB-INF/views/common/include.jsp

 .. code-block:: jsp

  <%@ page session="false"%>
  <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
  <%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt"%>
  <%@ taglib uri="http://www.springframework.org/tags" prefix="spring"%>
  <%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>
  <%@ taglib uri="http://www.springframework.org/security/tags" prefix="sec"%>
  <%@ taglib uri="http://terasoluna.org/functions" prefix="f"%>
  <%@ taglib uri="http://terasoluna.org/tags" prefix="t"%>
  <%@ taglib uri="http://tiles.apache.org/tags-tiles" prefix="tiles"%> <!-- (1) -->
  <%@ taglib uri="http://tiles.apache.org/tags-tiles-extras" prefix="tilesx"%> <!-- (2) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | Tiles用のカスタムタグ(TLD)の定義を追加する。
   * - | (2)
     - | Tiles-extras用のカスタムタグ(TLD)の定義を追加する。

Tilesのカスタムタグの詳細は、\ `こちら <http://tiles.apache.org/framework/tiles-jsp/tagreference.html>`_\ を参照されたい。

.. tip::

    | version 2系では tiles taglib は一つであったが、version 3から tiles-extras taglib が追加された。
    | version 2系では tiles taglib で利用可能であった useAttribute tag がversion 3から tiles-extras taglib へ移動されているので、利用していた場合は注意すること。
    | 変更例 ) `<tiles:useAttribute>` : version 2 -> `<tilesx:useAttribute>` : version 3


- web.xml

 .. code-block:: xml

    <jsp-config>
        <jsp-property-group>
            <url-pattern>*.jsp</url-pattern>
            <el-ignored>false</el-ignored>
            <page-encoding>UTF-8</page-encoding>
            <scripting-invalid>false</scripting-invalid>
            <include-prelude>/WEB-INF/views/common/include.jsp</include-prelude> <!-- (1) -->
        </jsp-property-group>
    </jsp-config>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | web.xmlの設定で、jspファイル(～.jsp)を読み込む場合、事前にinclude.jspを読み込ませることができる。

 .. note::

     カスタムタグはtemplate.jspに設定しても問題は無いが、カスタムタグの定義はインクルード用の共通jspファイルに作成することを推奨する。
     詳細は :ref:`view_jsp_include-label` を参照されたい。

**レイアウト作成**


レイアウトの枠となるjsp（template）と、レイアウトに埋め込むjspを作成する。

- template.jsp

 .. code-block:: xml

  <!DOCTYPE html>
  <!--[if lt IE 7]> <html class="no-js lt-ie9 lt-ie8 lt-ie7"> <![endif]-->
  <!--[if IE 7]>    <html class="no-js lt-ie9 lt-ie8"> <![endif]-->
  <!--[if IE 8]>    <html class="no-js lt-ie9"> <![endif]-->
  <!--[if gt IE 8]><!-->
  <html class="no-js">
  <!--<![endif]-->
  <head>
  <meta charset="utf-8" />
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
  <meta name="viewport" content="width=device-width" />
  <link rel="stylesheet"
      href="${pageContext.request.contextPath}/resources/app/css/styles.css"
      type="text/css" media="screen, projection">
  <script type="text/javascript">

  </script> <!-- (1) -->
  <c:set var="titleKey"> <!-- (2) -->
      <tiles:insertAttribute name="title" ignore="true" />
  </c:set>
  <title><spring:message code="${titleKey}" text="Create Staff Information" /></title><!-- (3) -->
  </head>
  <body>
      <div id="header">
          <tiles:insertAttribute name="header" /> <!-- (4) -->
      </div>
      <div id="body">
          <tiles:insertAttribute name="body" /> <!-- (5) -->
      </div>
      <div id="footer">
          <tiles:insertAttribute name="footer" /> <!-- (6) -->
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
     - | 共通的に記述する必要のある内容を(1)より上に記述する。
   * - | (2)
     - | tiles-definitions.xmlの(6)で指定した ``title`` の値を取得し、 ``titleKey`` に設定する。
   * - | (3)
     - | タイトルを設定する。
       | ``titleKey`` が取得できなかった際は、text属性で定義したタイトルを表示する。
   * - | (4)
     - | tiles-definitions.xmlで定義した"header"を読み込む。
   * - | (5)
     - | tiles-definitions.xmlで定義した"body"を読み込む。
   * - | (6)
     - | tiles-definitions.xmlで定義した"footer"を読み込む。


- header.jsp

 .. code-block:: jsp

  <h1>
      <a href="${pageContext.request.contextPath}">Staff Management
          System</a>
  </h1>


- createForm.jsp(body部分の例)

    開発者は、headerやfooterの余分なソースを記述せずに、body部分のみに集中して記述できる。

 .. code-block:: jsp

  <h2>Create Staff Information</h2>
  <table>
      <tr>
          <td>Staff First Name</td>
          <td><input type="text" /></td>
      </tr>
      <tr>
          <td>Staff Family Name</td>
          <td><input type="text" /></td>
      </tr>
      <tr>
          <td rowspan="5">Staff Authorities</td>
          <td><input type="checkbox" name="sa" value="01" /> Staff
              Management</td>
      </tr>
      <tr>
          <td><input type="checkbox" name="sa" value="02" /> Master
              Management</td>
      </tr>
      <tr>
          <td><input type="checkbox" name="sa" value="03" /> Stock
              Management</td>
      </tr>
      <tr>
          <td><input type="checkbox" name="sa" value="04" /> Order
              Management</td>
      </tr>
      <tr>
          <td><input type="checkbox" name="sa" value="05" /> Show Shopping
              Management</td>
      </tr>
  </table>

  <input type="submit" value="cancel" />
  <input type="submit" value="confirm" />


- footer.jsp

 .. code-block:: jsp

  <p style="text-align: center; background: #e5eCf9;">Copyright &copy;
      20XX CompanyName</p>

**Controller作成**


Controllerを作成するとき、リクエストが ``<contextPath>/staff/create?form`` の場合、
Controllerからのリターンが"staff/createForm"となるように設定する。

- StaffCreateController.java

 .. code-block:: java

  @RequestMapping(value = "create", method = RequestMethod.GET, params = "form")
  public String createForm() {
      return "staff/createForm"; // (1)
  }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | staffが{1}、createFormが{2}となり、propertiesからタイトル名を取得し、JSPを特定する。


**画面描画**

リクエストに ``<contextPath>/staff/create?form`` が呼ばれると、
以下のようにTilesがレイアウトを構築して画面描画を行う。

 .. code-block:: xml

    <definition name="layouts"
        template="/WEB-INF/views/layout/template.jsp"> <!-- (1) -->
        <put-attribute name="header"
            value="/WEB-INF/views/layout/header.jsp" /> <!-- (2) -->
        <put-attribute name="footer"
            value="/WEB-INF/views/layout/footer.jsp" /> <!-- (3) -->
    </definition>

    <definition name="*/*" extends="layouts">
      <put-attribute name="title" value="title.{1}.{2}" /> <!-- (4) -->
      <put-attribute name="body"
        value="/WEB-INF/views/{1}/{2}.jsp" /> <!-- (5) -->
    </definition>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | リクエストの時、親レイアウトである layouts が呼ばれ、テンプレートが /WEB-INF/views/layout/template.jspに設定される。
   * - | (2)
     - | テンプレート /WEB-INF/views/layout/template.jsp内に存在する ``header`` に WEB-INF/views/layout/header.jspが設定される。
   * - | (3)
     - | テンプレート /WEB-INF/views/layout/template.jsp内に存在する ``footer`` に /WEB-INF/views/layout/footer.jspが設定される。
   * - | (4)
     - | staffが{1}、createFormが{2}となり、spring-mvcに取り込まれているpropertiesから ``title.staff.createForm`` をkeyにvalueを取得する。
   * - | (5)
     - | staffが{1}、createFormが{2}となり、テンプレート/WEB-INF/views/layout/template.jsp内に存在する ``body`` に/WEB-INF/views/staff/createForm.jspが設定される。


結果として上記のtemplate.jspに、header.jsp、createForm.jsp、footer.jspが組み合わされた方法でブラウザに出力される。

 .. figure:: ./images/tiles_result.png
   :alt: tiles result
   :width: 100%
   :align: center

|

How to extend
--------------------------------------------------------------------------------

複数レイアウトを設定する場合
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| 実際に業務アプリケーションを作成する場合、業務内容によって表示レイアウトを分けたい場合がある。
| 今回は、スタッフ検索機能の場合、メニューを画面の左側に出す要望があると想定する。
| その設定方法について、 :ref:`TilesLayoutHowToUse` をベースに以下に示す。

**Tilesの定義**

- tiles-definitions.xml

 .. code-block:: guess
   :emphasize-lines: 7-20

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE tiles-definitions PUBLIC
       "-//Apache Software Foundation//DTD Tiles Configuration 3.0//EN"
       "http://tiles.apache.org/dtds/tiles-config_3_0.dtd">

    <tiles-definitions>
        <definition name="layoutsOfSearch"
            template="/WEB-INF/views/layout/templateSearch.jsp"> <!-- (1) -->
            <put-attribute name="header"
                value="/WEB-INF/views/layout/header.jsp" />
            <put-attribute name="menu"
                value="/WEB-INF/views/layout/menu.jsp" />
            <put-attribute name="footer"
                value="/WEB-INF/views/layout/footer.jsp" />
        </definition>

        <definition name="*/search*" extends="layoutsOfSearch"> <!-- (2) -->
            <put-attribute name="title" value="title.{1}.search{2}" /> <!-- (3) -->
            <put-attribute name="body" value="/WEB-INF/views/{1}/search{2}.jsp" /> <!-- (4) -->
        </definition>

        <definition name="layouts"
            template="/WEB-INF/views/layout/template.jsp">
            <put-attribute name="header"
                value="/WEB-INF/views/layout/header.jsp" />
            <put-attribute name="footer"
                value="/WEB-INF/views/layout/footer.jsp" />
        </definition>

        <definition name="*/*" extends="layouts">
            <put-attribute name="title" value="title.{1}.{2}" />
            <put-attribute name="body" value="/WEB-INF/views/{1}/{2}.jsp" />
        </definition>
    </tiles-definitions>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | 今回追加するレイアウト構成の親定義。
       | 別のレイアウトを使用する場合、difinitionタグのname属性について、既存のレイアウト定義"layouts"と重複しないようにする。
   * - | (2)
     - | 今回追加するレイアウトについて、描画のリクエストの際にnameのパターンと同じ場合に呼ばれるレイアウト定義。
       | リクエストが<contextPath>/\*/search\*に該当する場合、このレイアウト定義が読み込まれる。
       | extendsしている レイアウト定義"layoutsOfSearch"も適用される。
   * - | (3)
     - | 今回追加するレイアウトで使用するタイトルを指定する。
       | valueはspring-mvcに取り込まれているpropertiesの中から取得する。(以下の説明では application-messages.propertiesに設定する。)
       | {1}はリクエストの"\*/search\*"の「*」の1つ目。
       | {2}はリクエストの"\*/search\*"の"search*"に該当する為、先頭が"search"で始まる必要がある。
   * - | (4)
     - | bodyを定義しているjspファイルの置き場所について、{1}にリクエストパス、{2}に先頭に"search"を含んだJSPファイル名が一致するように設計する。
       | JSPファイルの置き場所の構成によってvalue属性の値を変更する必要がある。

 .. note::

     リクエストがdefinitionタグのname属性のパターンに複数該当する場合、上から順に確認し、1番最初に該当するパターンが採用される。
     上記の場合、スタッフ検索画面へのリクエストが複数パターンに該当するため、1番上にレイアウト定義している。

- `application-messages.properties`

 .. code-block:: properties
   :emphasize-lines: 2

   title.staff.createForm = Create Staff Information
   title.staff.searchStaff = Search Staff Information # (1)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 今回追加するメッセージ。
       | "staff"はリクエストの"\*/search\*"の「*」の1つ目。
       | "searchStaff"はリクエストの"\*/search\*"の"search\*"に該当する為、先頭が"search"で始まる必要がある。

**レイアウト作成**

レイアウトの枠となるjsp（template）と、レイアウトに埋め込むjspを作成する。

- templateSearch.jsp

 .. code-block:: xml

  <!DOCTYPE html>
  <!--[if lt IE 7]> <html class="no-js lt-ie9 lt-ie8 lt-ie7"> <![endif]-->
  <!--[if IE 7]>    <html class="no-js lt-ie9 lt-ie8"> <![endif]-->
  <!--[if IE 8]>    <html class="no-js lt-ie9"> <![endif]-->
  <!--[if gt IE 8]><!-->
  <html class="no-js">
  <!--<![endif]-->
  <head>
  <meta charset="utf-8" />
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
  <meta name="viewport" content="width=device-width" />
  <link rel="stylesheet"
      href="${pageContext.request.contextPath}/resources/app/css/styles.css"
      type="text/css" media="screen, projection">
  <script type="text/javascript">

  </script>
  <c:set var="titleKey">
      <tiles:insertAttribute name="title" ignore="true" />
  </c:set>
  <title><spring:message code="${titleKey}" text="Search Staff Information" /></title>
  </head>
  <body>
      <div id="header">
          <tiles:insertAttribute name="header" />
      </div>
      <div id="menu">
          <tiles:insertAttribute name="menu" /> <!-- (1) -->
      </div>
      <div id="body">
          <tiles:insertAttribute name="body" />
      </div>
      <div id="footer">
          <tiles:insertAttribute name="footer" />
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
     - | tiles-definitions.xmlで定義した"menu"を読み込む。
       | それ以外は :ref:`TilesLayoutHowToUse` と同じ

- styles.css

 .. code-block:: css

  div#menu { /* (1) */
      float: left;
      width: 20%;
  }

  div#searchBody { /* (2) */
      float: right;
      width: 80%;
  }

  div#footer { /* (3) */
      clear: both;
  }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | menu部分のstyleを設定する。
       | ここでは、float:leftでメニュー画面を左側に寄せて、width:20%で横幅2割で表示をするようにしている。
   * - | (2)
     - | body部分のstyleを設定する。
       | ここでは、float:rightで業務画面を右側に寄せて、width:80%で横幅8割で表示をするようにしている。
       | 名前をsearchBodyにしているのは、既存のレイアウトと名前が重複することにより、既存のレイアウトのstyleに影響を与えないためである。
   * - | (3)
     - | footer部分のstyleを設定する。
       | 上記menu部分とbody部分のfloatの効果を初期化している。これにより、menu部分とbody部分の下に表示するようにしている。


- header.jsp

  :ref:`TilesLayoutHowToUse` と同じ

- menu.jsp

 .. code-block:: jsp

  <table>
      <tr>
          <td><a href="${pageContext.request.contextPath}/staff/create?form">Create Staff Information</a></td>
      </tr>
      <tr>
          <td><a href="${pageContext.request.contextPath}/staff/search">Search Staff Information</a></td>
      </tr>
  </table>

- searchStaff.jsp(body部分の例)

 .. code-block:: jsp

  <h2>Search Staff Information</h2>
  <table>
      <tr>
          <td>Staff First Name</td>
          <td><input type="text" /></td>
      </tr>
      <tr>
          <td>Staff Family Name</td>
          <td><input type="text" /></td>
      </tr>
      <tr>
          <td rowspan="5">Staff Authorities</td>
          <td><input type="checkbox" name="sa" value="01" /> Staff
              Management</td>
      </tr>
      <tr>
          <td><input type="checkbox" name="sa" value="02" /> Master
              Management</td>
      </tr>
      <tr>
          <td><input type="checkbox" name="sa" value="03" /> Stock
              Management</td>
      </tr>
      <tr>
          <td><input type="checkbox" name="sa" value="04" /> Order
              Management</td>
      </tr>
      <tr>
          <td><input type="checkbox" name="sa" value="05" /> Show Shopping
              Management</td>
      </tr>
  </table>

  <input type="submit" value="Search" />

- footer.jsp

  :ref:`TilesLayoutHowToUse` と同じ

**Controller作成**


Controllerを作成するとき、リクエストが ``<contextPath>/staff/search`` の場合、
Controllerからのリターンが"staff/searchStaff"となるように設定する。

- StaffSearchController.java

 .. code-block:: java

  @RequestMapping(value = "search", method = RequestMethod.GET)
  public String createForm() {
      return "staff/searchStaff"; // (1)
  }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | staffが{1}、searchStaffが{2}となり、propertiesからタイトル名を取得し、JSPを特定する。


**画面描画**

リクエストに ``<contextPath>/staff/search`` が呼ばれると、
以下のように別のレイアウトを構築して画面描画を行う。


 .. code-block:: xml

    <definition name="layoutsOfSearch"
        template="/WEB-INF/views/layout/templateSearch.jsp"> <!-- (1) -->
        <put-attribute name="header"
            value="/WEB-INF/views/layout/header.jsp" /> <!-- (2) -->
        <put-attribute name="menu"
            value="/WEB-INF/views/layout/menu.jsp" /> <!-- (3) -->
        <put-attribute name="footer"
            value="/WEB-INF/views/layout/footer.jsp" /> <!-- (4) -->
    </definition>

    <definition name="*/search*" extends="layoutsOfSearch"> <!-- (5) -->
        <put-attribute name="title" value="title.{1}.search{2}" /> <!-- (6) -->
        <put-attribute name="body" value="/WEB-INF/views/{1}/search{2}.jsp" /> <!-- (7) -->
    </definition>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | 該当するリクエストの時、親レイアウトであるlayoutsOfSearchが呼ばれ、テンプレートが /WEB-INF/views/layout/templateSearch.jspに設定される。
   * - | (2)
     - | テンプレート /WEB-INF/views/layout/templateSearch.jsp内に存在する ``header`` に WEB-INF/views/layout/header.jspが設定される。
   * - | (3)
     - | テンプレート /WEB-INF/views/layout/templateSearch.jsp内に存在する ``menu`` に /WEB-INF/views/layout/menu.jspが設定される。
   * - | (4)
     - | テンプレート /WEB-INF/views/layout/templateSearch.jsp内に存在する ``footer`` に /WEB-INF/views/layout/footer.jspが設定される。
   * - | (5)
     - | リクエストが<contextPath>/\*/search\*に該当する場合、このレイアウト定義が読み込まれる。
       | その時、親レイアウトである"layoutsOfSearch"も読み込まれる。
   * - | (6)
     - | staffが{1}、searchStaffが"search{2}"となり、spring-mvcに取り込まれているpropertiesから ``title.staff.searchStaff`` をkeyにvalueを取得する。
   * - | (7)
     - | staffが{1}、searchStaffが"search{2}"となり、テンプレート/WEB-INF/views/layout/templateSearch.jsp内に存在する ``body`` に/WEB-INF/views/staff/searchStaff.jspが設定される。


結果として上記のtemplateSearch.jspに、header.jsp、menu.jsp、searchStaff.jsp、footer.jspが組み合わされた方法でブラウザに出力される。

 .. figure:: ./images/tiles_result2.png
   :alt: tiles result another template
   :width: 100%
   :align: center

.. raw:: latex

   \newpage

