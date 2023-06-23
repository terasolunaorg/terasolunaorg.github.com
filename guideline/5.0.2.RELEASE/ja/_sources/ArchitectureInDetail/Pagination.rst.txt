ページネーション
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

Overview
--------------------------------------------------------------------------------

本章では、検索条件に一致するデータをページ分割して表示する方法(ページネーション)について説明する。

| **検索条件に一致するデータが大量になる場合は、ページネーション機能を使用することを推奨する。**
| 一度に大量のデータを取得し画面に表示すると、以下3点の問題が発生する可能性がある。

* | サーバ側のメモリ枯渇の発生。
  | 単発のリクエストで問題が発生しなくても、同時に複数実行された場合に ``java.lang.OutOfMemoryError`` が発生する可能性がある。
* | ネットワーク負荷の発生。
  | 不要なデータがネットワークに流れることで、ネットワーク全体にかかる負荷が高くなり、システム全体のレスポンスタイムに影響を与える可能性がある。
* | 画面のレスポンス遅延の発生。
  | 大量のデータを扱う場合、サーバの処理、ネットワークのトラフィック処理、クライアントの描画処理の全てで時間がかかるため、画面のレスポンスが遅くなる可能性がある。

|

ページ分割時の一覧画面の表示について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
ページネーション機能を利用してページ分割した場合、以下のような画面になる。

 .. figure:: ./images/pagination-overview_screen.png
   :alt: Screen image of Pagination.
   :width: 100%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ページを移動するためのリンクを表示する。
        | リンク押下時には、該当ページを表示するためのリクエストを送信する。この領域を表示するためのJSPタグライブラリを共通ライブラリとして提供している。
    * - | (2)
      - | ページネーションに関連する情報(合計件数、合計ページ数、表示ページ数など)を表示する。
        | この領域を表示するためのタグライブラリは存在しないため、JSPの処理として個別に実装する必要がある。

|

ページ検索について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| ページネーションを実現する際には、まずサーバ側で行う検索処理をページ検索できるように実装する必要がある。
| 本ガイドラインでは、サーバ側のページ検索は、 Spring Data から提供されている仕組みを利用することを前提としている。

|

.. _pagination_overview_page_springdata:

Spring Data提供のページ検索機能について
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Spring Dataより提供されているページ検索用の機能は、以下の通り。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - 1
      - | リクエストパラメータよりページ検索に必要な情報(検索対象のページ位置、取得件数、ソート条件)を抽出し、抽出した情報を ``org.springframework.data.domain.Pageable`` のオブジェクトとしてControllerの引数に引き渡す。
        | この機能は、 ``org.springframework.data.web.PageableHandlerMethodArgumentResolver`` クラスとして提供されており、 :file:`spring-mvc.xml` の ``<mvc:argument-resolvers>`` 要素に追加することで有効となる。
        | リクエストパラメータについては、「 :ref:`Note欄 <pagination_overview_pagesearch_requestparameter>` 」を参照されたい。
    * - 2
      - | ページ情報(合計件数、該当ページのデータ、検索対象のページ位置、取得件数、ソート条件)を保持する。
        | この機能は、 ``org.springframework.data.domain.Page`` インタフェースとして提供されており、デフォルトの実装クラスとして ``org.springframework.data.domain.PageImpl`` が提供されている。
        | **共通ライブラリより提供しているページネーションリンクを出力するためのJSPタグライブラリでは、 Pageオブジェクトから必要なデータを取得する仕様となっている。**
    * - 3
      - | データベースアクセスとしてSpring Data JPAを使用する場合は、RepositoryのQueryメソッドの引数に ``Pageable`` オブジェクトを指定することで、該当ページの情報が ``Page`` オブジェクトとして返却される。
        | 合計件数を取得するSQLの発行、ソート条件の追加、該当ページに一致するデータの抽出などの処理が全て自動で行われる。
        | データベースアクセスとして、MyBatisを使用する場合は、Spring Data JPAが自動で行ってくれる処理を、Java(Service)及びSQLマッピングファイル内で実装する必要がある。

.. _pagination_overview_pagesearch_requestparameter:

 .. note:: **ページ検索用のリクエストパラメータについて**

    Spring Dataより提供されているページ検索用のリクエストパラメータは以下の3つとなる。

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
     .. list-table::
         :header-rows: 1
         :widths: 10 15 75

         * - 項番
           - パラメータ名
           - 説明
         * - 1.
           - page
           - | 検索対象のページ位置を指定するためのリクエストパラメータ。
             | 値には、0以上の数値を指定する。
             | デフォルトの設定では、ページ位置の値は ``0`` から開始する。そのため、1ページ目のデータを取得する場合は ``0`` を、2ページ目のデータを取得する場合は ``1`` を指定する必要がある。
         * - 2.
           - size
           - | 取得する件数を指定するためのリクエストパラメータ。
             | 値には、1以上の数値を指定する。
             | ``PageableHandlerMethodArgumentResolver`` の ``maxPageSize`` に指定された値より大きい値が指定された場合は、 ``maxPageSize`` の値が ``size`` の値となる。
         * - 3.
           - sort
           - | ソート条件を指定するためのパラメータ(複数指定可能)。
             | 値には、``"{ソート項目名(,ソート順)}"`` の形式で指定する。
             | ソート順には、``"ASC"`` 又は ``"DESC"`` のどちらかの値を指定し、省略した場合は ``"ASC"`` が適用される。
             | 項目名は ``","`` 区切りで複数指定することが可能である。
             | 例えば、クエリ文字列として ``"sort=lastModifiedDate,id,DESC&sort=subId"`` を指定した場合、 ``"ORDER BY lastModifiedDate DESC, id DESC, subId ASC"`` というOrder By句がQueryに追加される。

 .. warning:: **spring-data-commons 1.6.1.RELEASEにおける「size=0」指定時の動作について**

    terasoluna-gfw-common 1.0.0.RELEASEが依存するspring-data-commons 1.6.1.RELEASEでは、``"size=0"`` を指定すると条件に一致するレコードを全件取得するという不具合がある。
    そのため、大量のレコードが取得対象となる可能性がある場合は、``java.lang.OutOfMemoryError`` が発生する可能性が高くなる。

    この問題はSpring Data CommonsのJIRA「`DATACMNS-377 <https://jira.springsource.org/browse/DATACMNS-377>`_」で対応され、spring-data-commons 1.6.3.RELEASEで解消されている。
    改修後の動作としては、``"size<=0"`` を指定した場合は、 sizeパラメータ省略時のデフォルト値が適用される。
    
    terasoluna-gfw-common 1.0.0.RELEASEを使用している場合は、terasoluna-gfw-common 1.0.1.RELEASE以上へバージョンアップする必要がある。

 .. warning:: **spring-data-commons 1.6.1.RELEASEにおけるリクエストパラメータに不正な値を指定した際の動作について**

    terasoluna-gfw-common 1.0.0.RELEASEが依存するspring-data-commons 1.6.1.RELEASEでは、ページ検索用のリクエストパラメータ(page, size, sort)に不正な値を指定した場合、
    ``java.lang.IllegalArgumentException`` 又は ``java.lang.ArrayIndexOutOfBoundsException`` が発生し、SpringMVCのデフォルトの設定だとシステムエラー(HTTPステータスコード=500)となってしまうという不具合がある。

    この問題はSpring Data CommonsのJIRA「`DATACMNS-379 <https://jira.springsource.org/browse/DATACMNS-379>`_」と「`DATACMNS-408 <https://jira.springsource.org/browse/DATACMNS-408>`_」で対応され、spring-data-commons 1.6.3.RELEASEで解消されている。
    改修後の動作としては、不正な値を指定した場合は、 パラメータ省略時のデフォルト値が適用される。

    terasoluna-gfw-common 1.0.0.RELEASEを使用している場合は、terasoluna-gfw-common 1.0.1.RELEASE以上へバージョンアップする必要がある。

 .. note:: **Spring Data CommonsのAPI仕様の変更に伴う注意点**

    terasoluna-gfw-common 5.0.0.RELEASE以上が依存するspring-data-commons(1.9.1.RELEASE以上)では、
    ページ検索機能用のインタフェース(\ ``org.springframework.data.domain.Page``\ )とクラス(\ ``org.springframework.data.domain.PageImpl``\ と\ ``org.springframework.data.domain.Sort.Order``\ )のAPI仕様が変更になっている。

    具体的には、

    * \ ``Page``\ インタフェースと\ ``PageImpl``\ クラスでは、\ ``isFirst()``\ と\ ``isLast()``\ メソッドがspring-data-commons 1.8.0.RELEASEで追加、\ ``isFirstPage()``\ と\ ``isLastPage()``\ メソッドがspring-data-commons 1.9.0.RELEASEで削除
    * \ ``Sort.Order``\ クラスでは、 \ ``nullHandling``\ プロパティがspring-data-commons 1.8.0.RELEASEで追加

    されている。

    削除されたAPIを使用している場合はコンパイルエラーとなるので、アプリケーションの修正が必要になる。
    加えて、REST APIのリソースオブジェクトとして\ ``Page``\ インタフェース(\ ``PageImpl``\ クラス)を使用している場合は、
    JSONやXMLのフォーマットが変わってしまうため、こちらもアプリケーションの修正が必要になるケースがある。

|

.. _pagination_overview_paginationlink:

ページネーションリンクの表示について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 共通ライブラリから提供しているJSPタグライブラリを使って出力されるページネーションリンクについて説明する。

| 共通ライブラリからはページネーションリンクを表示するためのスタイルシートの提供は行っていないため、各プロジェクトにて用意すること。
| 以降の説明で使用する画面は、Bootstrap v3.0.0のスタイルシートを適用している。

|

ページネーションリンクの構成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
ページネーションリンクは、以下の要素から構成される。

 .. figure:: ./images/pagination-how_to_use_jsp_pagelink_description.png
   :alt: Structure of the pagination link.
   :width: 90%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 最初のページに移動するためのリンク。
    * - | (2)
      - | 前のページに移動するためのリンク。
    * - | (3)
      - | 指定したページに移動するためのリンク。
    * - | (4)
      - | 次のページに移動するためのリンク。
    * - | (5)
      - | 最後のページに移動するためのリンク。

|

ページネーションリンクは、以下の状態をもつ。

 .. figure:: ./images/pagination-how_to_use_jsp_pagelink_description_status.png
   :alt: Status of the pagination link.
   :width: 90%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (6)
      - | 現在表示しているページで操作することができないリンクであることを示す状態。
        | 具体的には、1ページ目を表示している時の「最初のページに移動するためのリンク」「前のページに移動するためのリンク」と、最終ページを表示している時の「次のページに移動するためのリンク」「最後のページに移動するためのリンク」がこの状態となる。
        | 共通ライブラリから提供しているJSPタグライブラリでは、この状態を ``"disabled"`` と定義している。
    * - | (7)
      - | 現在表示しているページであることを示す状態。
        | 共通ライブラリから提供しているJSPタグライブラリでは、この状態を ``"active"`` と定義している。

|

| 共通ライブラリを使って出力されるHTMLは、以下の構造となる。
| 図中の番号は、上記で説明した「ページネーションリンクの構成」と「ページネーションリンクの状態」の項番に対応させている。

- JSP

 .. code-block:: jsp

    <t:pagination page="${page}" />

- 出力されるHTML

 .. figure:: ./images/pagination-overview_html.png
   :alt: html of the pagination link.
   :width: 90%
   :align: center

|

ページネーションリンクのHTML構造
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
共通ライブラリを使って出力されるページネーションリンクのHTMLは、以下の構造となる。

- HTML

 .. figure:: ./images/pagination-overview_html_basic.png
   :alt: html structure of the pagination link.
   :width: 100%
   :align: center

- 画面イメージ

 .. figure:: ./images/pagination-overview_html_basic_screen.png
   :alt: screen structure of the pagination link.
   :width: 80%
   :align: center


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.70\linewidth}|p{0.20\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 70 20

    * - 項番
      - 説明
      - デフォルト値
    * - | (1)
      - | ページネーションリンクの構成要素をまとめるための要素。
        | 共通ライブラリでは、この部分を「Outer Element」と呼び、複数の「Inner Element」を保持する。
        | 使用する要素は、JSPタグライブラリのパラメータによって変更することが出来る。
      - | ``<ul>`` 要素
    * - | (2)
      - | 「Outer Element」のスタイルクラスを指定するための属性。
        | 共通ライブラリでは、この部分を「Outer Element Class」と呼び、属性値はJSPタグライブラリのパラメータによって指定する。
      - | 指定なし
    * - | (3)
      - | ページネーションリンクを構成するための要素。
        | 共通ライブラリでは、この部分を「Inner Element」と呼び、 ページ移動するためのリクエストを送信するための ``<a>`` 要素を保持する。
        | 使用する要素は、JSPタグライブラリのパラメータによって変更することが出来る。
      - | ``<li>`` 要素
    * - | (4)
      - | 「Inner Element」のスタイルクラスを指定するための属性。
        | 共通ライブラリでは、この部分を「Inner Element Class」と呼び、属性値は表示しているページ位置によってJSPタグライブラリ内の処理で切り替わる。
      - | 「 :ref:`Note欄 <pagination_overview_paginationlink_innerelementclass>` 」を参照されたい。
    * - | (5)
      - | ページ移動するためのリクエストを送信するための要素。
        | 共通ライブラリでは、この部分を「Page Link」と呼ぶ。
      - | `<a>` 要素固定
    * - | (6)
      - | ページ移動するためのURLを指定するための属性。
        | 共通ライブラリでは、この部分を「Page Link URL」と呼ぶ。
      - | 下記の「 :ref:`Note欄 <pagination_overview_paginationlink_pagelinkurl>` 」を参照されたい。
    * - | (7)
      - | ページ移動するためのリンクの表示テキストを指定する。
        | 共通ライブラリでは、この部分を「Page Link Text」と呼ぶ。
      - | 下記の「 :ref:`Note欄 <pagination_overview_paginationlink_pagelinktext>` 」を参照されたい。


 .. note:: **「Inner Element」の数について**

    デフォルトの設定では、「Inner Element」は最大で14個となる。内訳は以下の通り。

    * 最初のページに移動するためのリンク : 1
    * 前のページに移動するためのリンク : 1
    * 指定したページに移動するためのリンク : 最大10
    * 次のページに移動するためのリンク : 1
    * 最後のページに移動するためのリンク : 1

    「Inner Element」の数は、JSPタグライブラリのパラメータの指定によって変更することができる。


.. _pagination_overview_paginationlink_innerelementclass:

 .. note:: **「Inner Element Class」の設定値について**

    デフォルトの設定では、ページ位置によって、以下3つの値となる。

    * ``"disabled"`` : 現在表示しているページでは操作することができないリンクであることを示すためのスタイルクラス。
    * ``"active"`` : 現在表示しているページのリンクであることを示すためのスタイルクラス。
    * 指定なし : 上記以外のリンクであることを示す。

    ``"disabled"`` と ``"active"`` は、JSPタグライブラリのパラメータの指定によって別の値に変更することができる。


.. _pagination_overview_paginationlink_pagelinkurl:

 .. note:: **「Page Link URL」のデフォルト値について**

    リンクの状態が\ ``"disabled"``\と\ ``"active"``\ の場合は\ ``"javascript:void(0)"``\ 、それ以外の場合は\ ``"?page={page}&size={size}"``\ となる。

    「Page Link URL」は、JSPタグライブラリのパラメータの指定によって別の値に変更することができる。

    terasoluna-gfw-web 5.0.0.RELEASEより、\ ``"active"``\ 状態のリンクのデフォルト値を\ ``"?page={page}&size={size}"``\から\ ``"javascript:void(0)"``\に変更している。
    これは、メジャーなWebサイトのページネーションリンクの実装やメジャーなCSSライブラリ(Bootstrapなど)の実装に合わせるためである。

.. _pagination_overview_paginationlink_pagelinktext:

 .. note:: **「Page Link Text」のデフォルト値について**

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.50\linewidth}|p{0.30\linewidth}|
     .. list-table::
         :header-rows: 1
         :widths: 10 50 30

         * - 項番
           - リンク名
           - デフォルト値
         * - 1.
           - 最初のページに移動するためのリンク
           - ``"<<"``
         * - 2.
           - 前のページに移動するためのリンク
           - ``"<"``
         * - 3.
           - 指定したページに移動するためのリンク
           - | 該当ページのページ番号
             | (変更不可)
         * - 4.
           - 次のページに移動するためのリンク
           - ``">"``
         * - 5.
           - 最後のページに移動するためのリンク
           - ``">>"``

    「指定したページに移動するためのリンク」以外は、JSPタグライブラリのパラメータの指定によって、別の値に変更することができる。

|

.. _pagination_overview_paginationlink_taglibparameters:

JSPタブライブラリのパラメータについて
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
JSPタグライブラリのパラメータに値を指定することで、デフォルト動作を変更することができる。

以下にパラメータの一覧を示す。

**レイアウトを制御するためのパラメータ**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.25\linewidth}|p{0.65\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 25 65

    * - 項番
      - パラメータ名
      - 説明
    * - 1.
      - outerElement
      - | 「Outer Element」として使用するHTML要素名を指定する。
        | 例) div
    * - 2.
      - outerElementClass
      - | 「Outer Element Class」に設定するスタイルシートのクラス名を指定する。
        | 例) pagination
    * - 3.
      - innerElement
      - | 「Inner Element」として使用するHTML要素名を指定する。
        | 例) span
    * - 4.
      - disabledClass
      - | ``"disabled"`` 状態と判断された「Inner Element」のclass属性に設定する値を指定する。
        | 例) hiddenPageLink
    * - 5.
      - activeClass
      - | ``"active"`` 状態の「Inner Element」のclass属性に設定する値を指定する。
        | 例) currentPageLink
    * - 6.
      - firstLinkText
      - | 「最初のページに移動するためのリンク」の「Page Link Text」に設定する値を指定する。
        | ``""`` を指定すると、「最初のページに移動するためのリンク」自体が出力されなくなる。
        | 例) First
    * - 7.
      - previousLinkText
      - | 「前のページに移動するためのリンク」の「Page Link Text」に設定する値を指定する。
        | ``""`` を指定すると、「前のページに移動するためのリンク」自体が出力されなくなる。
        | 例) Prev
    * - 8.
      - nextLinkText
      - | 「次のページに移動するためのリンク」の「Page Link Text」に設定する値を指定する。
        | ``""`` を指定すると、「次のページに移動するためのリンク」自体が出力されなくなる。
        | 例) Next
    * - 9.
      - lastLinkText
      - | 「最後のページに移動するためのリンク」の「Page Link Text」に設定する値を指定する。
        | ``""`` を指定すると、「次のページに移動するためのリンク」自体が出力されなくなる。
        | 例) Last
    * - 10.
      - maxDisplayCount
      - | 「指定したページに移動するためのリンク」の最大表示数を指定する。
        | ``0`` を指定すると、「指定したページに移動するためのリンク」自体が出力されなくなる。
        | 例) 5

|

 レイアウトを制御するためのパラメータを、全てデフォルトから変更した時に出力されるHTMLは以下の通り。
 図中の番号は、上記で説明したパラメータ一覧の項番に対応している。

 - JSP

  .. code-block:: jsp

    <t:pagination page="${page}"
        outerElement="div"
        outerElementClass="pagination"
        innerElement="span"
        disabledClass="hiddenPageLink"
        activeClass="currentPageLink"
        firstLinkText="First"
        previousLinkText="Prev"
        nextLinkText="Next"
        lastLinkText="Last"
        maxDisplayCount="5"
        />

 - 出力されるHTML

  .. figure:: ./images/pagination-overview_html_changed.png
   :alt: html of the pagination link(changed layout).
   :width: 100%
   :align: center

|

**動作を制御するためのパラメータ**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.25\linewidth}|p{0.65\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 25 65

    * - 項番
      - パラメータ名
      - 説明
    * - 1.
      - disabledHref
      - | ``"disabled"`` 状態のリンクの「Page Link URL」に設定する値を指定する。
    * - 2.
      - pathTmpl
      - | 「Page Link URL」に設定するリクエストパスのテンプレートを指定する。
        | ページ表示時のリクエストパスとページ移動するためのリクエストパスが異なる場合は、このパラメータにページ移動用のリクエストパスを指定する必要がある。
        | 指定するリクエストパスのテンプレートには、ページ位置(page)や取得件数(size)などをパス変数(プレースホルダ)として指定することができる。
        | 指定した値はUTF-8でURLエンコーディングされる。
    * - 3.
      - queryTmpl
      - | 「Page Link URL」のクエリ文字列のテンプレートを指定する。
        | ページ移動する際に必要となるページネーション用のクエリ文字列(page,size,sortパラメータ)を生成するためのテンプレートを指定する。
        | ページ位置や取得件数のリクエストパラメータ名をデフォルト以外の値にする場合は、このパラメータにクエリ文字列を指定する必要がある。
        | 指定するクエリ文字列のテンプレートには、ページ位置(page)や取得件数(size)などをパス変数(プレースホルダ)として指定することができる。
        | 指定した値はUTF-8でURLエンコーディングされる。
        |
        | この属性は、ページネーション用のクエリ文字列(page,size,sortパラメータ)を生成するための属性であるため、検索条件を引き継ぐためのクエリ文字列はcriteriaQuery属性に指定すること。
    * - 4.
      - criteriaQuery
      - | 「Page Link URL」に追加する検索条件用のクエリ文字列を指定する。
        | **「Page Link URL」に検索条件を引き継ぐ場合は、このパラメータに検索条件用のクエリ文字列を指定すること。**
        | **指定した値はURLエンコーディングされないため、URLエンコーディング済みのクエリ文字列を指定する必要がある。**
        |
        | フォームオブジェクトに格納されている検索条件をURLエンコーディング済みのクエリ文字列に変換する場合は、共通ライブラリから提供しているELファクション(\ ``f:query(Object)``\)を使用すると、簡単に条件を引き継ぐことができる。
        |
        | terasoluna-gfw-web 1.0.1.RELEASE以上で利用可能なパラメータである。
    * - 5.
      - disableHtmlEscapeOfCriteriaQuery
      - | \ ``criteriaQuery``\パラメータに指定された値に対するHTMLエスケープ処理を無効化するためのフラグ。
        | \ ``true``\ を指定する事で、\ ``criteriaQuery``\パラメータに指定された値に対してHTMLエスケープ処理が行われなくなる。(デフォルト値は\ ``false``\)
        | **trueを指定する場合は、XSS対策が必要な文字がクエリ文字列内に含まれない事が保証されていること。**
        |
        | terasoluna-gfw-web 1.0.1.RELEASE以上で利用可能なパラメータである。
    * - 6.
      - enableLinkOfCurrentPage
      - | \ ``"active"``\ 状態のページリンクを押下した際に、該当ページを再表示するためのリクエストを送信するためのフラグ。
        | \ ``true``\ を指定する事で、「Page Link URL」に該当ページを再表示するためのURL(デフォルト値は\ ``"?page={page}&size={size}"``\ )が設定される。(デフォルト値は\ ``false``\で、「Page Link URL」には\ ``disabledHref``\ 属性の値が設定される)
        |
        | terasoluna-gfw-web 5.0.0.RELEASE以上で利用可能なパラメータである。

 .. note:: **disabledHrefの設定値について**

    デフォルトでは、\ ``disabledHref``\ 属性には\ ``"javascript:void(0)"``\ が設定されている。
    ページリンク押下時の動作を無効化するだけであれば、デフォルトのままでよい。

    ただし、デフォルトの状態でページリンクにフォーカスを移動又はマウスオーバーした場合、
    ブラウザのステータスバーに\ ``"javascript:void(0)"``\ が表示されることがある。
    この挙動を変えたい場合は、JavaScriptを使用してページリンク押下時の動作を無効化する必要がある。
    実装例については、「:ref:`PaginationHowToUseDisablePageLinkUsingJavaScript`」を参照されたい。

    terasoluna-gfw-web 5.0.0.RELEASEより、\ ``disabledHref``\ 属性のデフォルト値を\ ``"#"``\から\ ``"javascript:void(0)"``\ に変更している。
    この変更を行うことで、\ ``"disabled"``\ 状態のページリンクを押下した際に、フォーカスがページのトップへ移動しないようになっている。


 .. note:: **パス変数(プレースホルダ)について**

    ``pathTmpl`` 及び ``queryTmpl`` に指定できるパス変数は、以下の通り。

        .. tabularcolumns:: |p{0.10\linewidth}|p{0.25\linewidth}|p{0.75\linewidth}|
        .. list-table::
            :header-rows: 1
            :widths: 10 25 75
    
            * - 項番
              - パス変数名
              - 説明
            * - 1.
              - page
              - ページ位置を埋め込むためのパス変数。
            * - 2.
              - size
              - 取得件数を埋め込むためのパス変数。
            * - 3.
              - sortOrderProperty
              - ソート条件のソート項目を埋め込むためのパス変数。
            * - 4.
              - sortOrderDirection
              - ソート条件のソート順を埋め込むためのパス変数。

    パス変数は、``"{パス変数名}"`` の形式で指定する。

 .. warning:: **ソート条件の制約事項**

    ソート条件のパス変数に設定される値は、ひとつのソート条件のみとなっている。
    そのため、複数のソート条件を指定して検索した結果を、ページネーション表示する必要がある場合は、
    共通ライブラリから提供しているJSPタグライブラリを拡張する必要がある。

|

 動作を制御するためのパラメータを変更した時に出力されるHTMLは、以下の通り。
 図中の番号は、上記で説明したパラメータ一覧の項番に対応している。

 - JSP

  .. code-block:: jsp

    <t:pagination page="${page}"
        disabledHref="#"
        pathTmpl="${pageContext.request.contextPath}/article/list/{page}/{size}"
        queryTmpl="sort={sortOrderProperty},{sortOrderDirection}"
        criteriaQuery="${f:query(articleSearchCriteriaForm)}"
        enableLinkOfCurrentPage="true" />

 - 出力されるHTML

  .. figure:: ./images/pagination-overview_html_changed2.png
   :alt: html of the pagination link(changed behavior).
   :width: 100%
   :align: center

|

ページネーション機能使用時の処理フロー
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Spring Dataより提供されているページネーション機能と、共通ライブラリから提供してるJSPタグライブラリを利用した際の処理フローは、以下の通り。

 .. figure:: ./images/pagination-overview_flow.png
   :alt: processing flow of pagination
   :width: 100%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 検索条件と共に、リクエストパラメータとして検索対象のページ位置(page)と取得件数(size)を指定してリクエストを送信する。
    * - | (2)
      - | ``PageableHandlerMethodArgumentResolver`` は、リクエストパラメータに指定されている検索対象のページ位置(page)と取得件数(size)を取得し、 ``Pageable`` オブジェクトを生成する。
        | 生成された ``Pageable`` オブジェクトは、Controllerの処理メソッドの引数に設定される。
    * - | (3)
      - | Controllerは、引数で受け取った ``Pageable`` オブジェクトを、Serviceのメソッドに引き渡す。
    * - | (4)
      - | Serviceは、引数で受け取った ``Pageable`` オブジェクトを、 RepositoryのQueryメソッドに引き渡す。
    * - | (5)
      - | Repositoryは、検索条件に一致するデータの合計件数(totalElements)と、引数で受け取った ``Pageable`` オブジェクトに指定されているページ位置(page)と取得件数(size)の範囲に存在するデータを、データベースより取得する。
    * - | (6)
      - | Repositoryは、取得した合計件数(totalElements)、取得データ(content)、引数で受け取った ``Pageable`` オブジェクトより ``Page`` オブジェクトを作成し、Service及びControllerへ返却する。
    * - | (7)
      - | Controllerは、返却された ``Page`` オブジェクトを、 ``Model`` オブジェクトに格納後、JSPに遷移する。
    * - | (8)
      - | JSPは、 ``Model`` オブジェクトに格納されている ``Page`` オブジェクトを取得し、共通ライブラリから提供されているページネーション用のJSPタグライブラリ( ``<t:pagination>`` )を呼び出す。
        | ページネーション用のJSPタグライブラリは ``Page`` オブジェクトを参照し、ページネーションリンクを生成する。
    * - | (9)
      - | JSPで生成したHTMLを、クライアント(ブラウザ)に返却する。
    * - | (10)
      - | ページネーションリンクを押下すると、該当ページを表示するためリクエストが送信される。

 .. note:: **Repositoryの実装について**

    上記フローの(5)と(6)の処理は、使用するO/R Mapperによって実装方法が異なる。

    * MyBatis3を使用する場合は、Java(Service)及びSQLマッピングファイルの実装が必要がある。
    * Spring Data JPAを使用する場合は、Spring Data JPAの機能で自動的で行われるため実装は不要である。

    具体的な実装例については、

    * :doc:`DataAccessMyBatis3`
    * :doc:`DataAccessJpa`

    を参照されたい。


|

How to use
--------------------------------------------------------------------------------

ページネーション機能の具体的な使用方法を以下に示す。

アプリケーションの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Dataのページネーション機能を有効化するための設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| リクエストパラメータに指定された検索対象のページ位置(page)、取得件数(size)、ソート条件(sort)を、 ``Pageable`` オブジェクトとしてControllerの引数に設定するための機能を有効化する。
| 下記の設定は、ブランクプロジェクトでは設定済みの状態になっている。

:file:`spring-mvc.xml`

 .. code-block:: xml

    <mvc:annotation-driven>
        <mvc:argument-resolvers>
            <!-- (1) -->
            <bean
                class="org.springframework.data.web.PageableHandlerMethodArgumentResolver" />
        </mvc:argument-resolvers>
    </mvc:annotation-driven>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ``<mvc:argument-resolvers>`` に ``org.springframework.data.web.PageableHandlerMethodArgumentResolver`` を指定する。
        | ``PageableHandlerMethodArgumentResolver`` で指定できるプロパティについては、「 :ref:`paginatin_appendix_pageableHandlerMethodArgumentResolver` 」を参照されたい。

|

ページ検索の実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
ページ検索を実現するための実装方法を以下に示す。

アプリケーション層の実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
ページ検索に必要な情報(検索対象のページ位置、取得件数、ソート条件)を、Controllerの引数として受け取り、Serviceのメソッドに引き渡す。

- Controller

 .. code-block:: java

    @RequestMapping("list")
    public String list(@Validated ArticleSearchCriteriaForm form,
            BindingResult result,
            Pageable pageable, // (1)
            Model model) {

        ArticleSearchCriteria criteria = beanMapper.map(form,
                ArticleSearchCriteria.class);

        Page<Article> page = articleService.searchArticle(criteria, pageable); // (2)

        model.addAttribute("page", page); // (3)

        return "article/list";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 処理メソッドの引数として ``Pageable`` を指定する。
        | ``Pageable`` オブジェクトには、ページ検索に必要な情報(検索対象のページ位置、取得件数、ソート条件)が格納されている。
    * - | (2)
      - | Serviceのメソッドの引数に ``Pageable`` オブジェクトを指定して呼び出す。
    * - | (3)
      - | Serviceから返却された検索結果( ``Page`` オブジェクト )を ``Model`` に追加する。 ``Model`` に追加することで、View(JSP)から参照できるようになる。

 .. note:: **リクエストパラメータにページ検索に必要な情報の指定がない場合の動作について**

    ページ検索に必要な情報(検索対象のページ位置、取得件数、ソート条件)がリクエストパラメータに指定されていない場合は、デフォルト値が適用される。
    デフォルト値は、以下の通り。

    * 検索対象のページ位置 : `0` (1ページ目)
    * 取得件数 : `20`
    * ソート条件 : `null` (ソート条件なし)

    デフォルト値は、以下の２つの方法で変更することができる。

    * 処理メソッドの ``Pageable`` の引数に、 ``@org.springframework.data.web.PageableDefault`` アノテーションを指定してデフォルト値を定義する。
    * ``PageableHandlerMethodArgumentResolver`` の ``fallbackPageable`` プロパティにデフォルト値を定義した ``Pageable`` オブジェクトを指定する。

|

| ``@PageableDefault`` アノテーションを使用してデフォルト値を指定する方法について説明する。
| ページ検索処理毎にデフォルト値を変更する必要がある場合は、``@PageableDefault`` アノテーションを使ってデフォルト値を指定する。

 .. code-block:: java

    @RequestMapping("list")
    public String list(@Validated ArticleSearchCriteriaForm form,
            BindingResult result,
            @PageableDefault( // (1)
                    page = 0,    // (2)
                    size = 50,   // (3)
                    direction = Direction.DESC,  // (4)
                    sort = {     // (5)
                        "publishedDate",
                        "articleId"
                        }
                    ) Pageable pageable,
            Model model) {
        // ...
        return "article/list";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.70\linewidth}|p{0.20\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 70 20

    * - 項番
      - 説明
      - デフォルト値
    * - | (1)
      - | ``Pageable`` の引数に ``@PageableDefault`` アノテーションを指定する。
      - | -
    * - | (2)
      - | ページ位置のデフォルト値を変更する場合は、 ``@PageableDefault`` のpage属性に値を指定する。
        | 通常変更する必要はない。
      - | ``0``
        | (1ページ目)
    * - | (3)
      - | 取得件数のデフォルト値を変更する場合は、 ``@PageableDefault`` のsize又はvalue属性に値を指定する。
      - | ``10``
    * - | (4)
      - | ソート条件のデフォルト値を変更する場合は、 ``@PageableDefault`` のdirection属性に値を指定する。
      - | ``Direction.ASC``
        | (昇順)
    * - | (5)
      - | ソート条件のソート項目を指定する場合は、 ``@PageableDefault`` のsort属性にソート項目を指定する。
        | 複数の項目でソートする場合は、ソートするプロパティ名を配列で指定する。
        | 上記例では、 ``"ORDER BY publishedDate DESC, articleId DESC"`` というソート条件がQueryに追加される。
      - | 空の配列
        | (ソート項目なし)

 .. note:: **@PageableDefaultアノテーションで指定できるソート順について**

    ``@PageableDefault`` アノテーションで指定できるソート順は昇順か降順のどちらか一つなので、項目ごとに異なるソート順を指定したい場合は ``@org.springframework.data.web.SortDefaults`` アノテーションを使用する必要がある。
    具体的には、 ``"ORDER BY publishedDate DESC, articleId ASC"`` というソート順にしたい場合である。

 .. tip:: **取得件数のデフォルト値のみ変更する場合の指定方法**

    取得件数のデフォルト値のみ変更する場合は、 ``@PageableDefault(50)`` と指定することもできる。これは ``@PageableDefault(size = 50)`` と同じ動作となる。

|

| ``@SortDefaults`` アノテーションを使用してデフォルト値を指定する方法について説明する。
| ``@SortDefaults`` アノテーションは、ソート項目が複数あり、項目ごとに異なるソート順を指定したい場合に使用する。

 .. code-block:: java

    @RequestMapping("list")
    public String list(
            @Validated ArticleSearchCriteriaForm form,
            BindingResult result,
            @PageableDefault(size = 50)
            @SortDefaults(  // (1)
                    {
                        @SortDefault(  // (2)
                                     sort = "publishedDate",    // (3)
                                     direction = Direction.DESC // (4)
                                    ),
                        @SortDefault(
                                     sort = "articleId"
                                    )
                    }) Pageable pageable,
            Model model) {
        // ...
        return "article/list";
    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.70\linewidth}|p{0.20\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 70 20

    * - 項番
      - 説明
      - デフォルト値
    * - | (1)
      - | ``Pageable`` の引数に ``@SortDefaults`` アノテーションを指定する。
        | ``@SortDefaults`` アノテーションには、複数の ``@org.springframework.data.web.SortDefault`` アノテーションを配列として指定することができる。
      - | -
    * - | (2)
      - | ``@SortDefaults`` アノテーションの value属性に、 ``@SortDefault`` アノテーションを指定する。
        | 複数指定する場合は配列として指定する。
      - | -
    * - | (3)
      - | ``@PageableDefault`` のsort又はvalue属性にソート項目を指定する。
        | 複数の項目を指定する場合は配列として指定する。
      - | 空の配列
        | (ソート項目なし)
    * - | (4)
      - | ソート条件のデフォルト値を変更する場合は、``@PageableDefault`` のdirection属性に値を指定する。
      - | ``Direction.ASC``
        | (昇順)

 上記例では、 ``"ORDER BY publishedDate DESC, articleId ASC"`` というソート条件がQueryに追加される。

 .. tip:: **ソート項目のデフォルト値のみ指定する場合の指定方法**

    取得項目のみ指定する場合は、 ``@PageableDefault("articleId")`` と指定することもできる。
    これは ``@PageableDefault(sort = "articleId")`` や ``@PageableDefault(sort = "articleId", direction = Direction.ASC)`` と同じ動作となる。

|

アプリケーション全体のデフォルト値を変更する必要がある場合は、 :file:`spring-mvc.xml` に定義した ``PageableHandlerMethodArgumentResolver``
の ``fallbackPageable`` プロパティにデフォルト値を定義した ``Pageable`` オブジェクトを指定する。

``fallbackPageable`` の説明や具体的な設定例については、「 :ref:`paginatin_appendix_pageableHandlerMethodArgumentResolver` 」を参照されたい。

|

ドメイン層の実装(MyBatis3編)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| MyBatisを使用してデータベースにアクセスする場合は、Controllerから受け取った ``Pageable`` オブジェクトより、必要な情報を抜き出してRepositoryに引き渡す。
| 該当データを抽出するためのSQLやソート条件については、SQLマッピングで実装する必要がある。

ドメイン層で実装するページ検索処理の詳細については、

* :ref:`DataAccessMyBatis3HowToUseFindPageUsingMyBatisFunction`
* :ref:`DataAccessMyBatis3HowToUseFindPageUsingSqlFilter`

を参照されたい。

|

ドメイン層の実装(JPA編)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
JPA(Spring Data JPA)を使用してデータベースにアクセスする場合は、Controllerから受け取った ``Pageable`` オブジェクトをRepositoryに引き渡す。

ドメイン層で実装するページ検索処理の詳細については、

* :ref:`DataAccessJpaHowToUseFindPage`


を参照されたい。

|

JSPの実装(基本編)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
ページ検索処理で取得した ``Page`` オブジェクトを一覧画面に表示し、ページネーションリンク及びページネーション情報(合計件数、合計ページ数、表示ページ数など)を表示する方法を以下に示す。

取得データの表示
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
ページ検索処理で取得したデータを表示するための実装例を以下に示す。

- Controller

 .. code-block:: java

    @RequestMapping("list")
    public String list(@Validated ArticleSearchCriteriaForm form, BindingResult result,
            Pageable pageable, Model model) {

        if (!StringUtils.hasLength(form.getWord())) {
            return "article/list";
        }

        ArticleSearchCriteria criteria = beanMapper.map(form,
                ArticleSearchCriteria.class);

        Page<Article> page = articleService.searchArticle(criteria, pageable);

        model.addAttribute("page", page); // (1)

        return "article/list";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ``"page"`` という属性名で ``Page`` オブジェクトを ``Model`` に格納する。
        | JSPでは ``"page"`` という属性名を指定して ``Page`` オブジェクトにアクセスすることになる。


- JSP

 .. code-block:: jsp

    <%-- ... --%>

    <%-- (2) --%>
    <c:when test="${page != null && page.totalPages != 0}">

      <table class="maintable">
        <thead>
          <tr>
            <th class="no">No</th>
            <th class="articleClass">Class</th>
            <th class="title">Title</th>
            <th class="overview">Overview</th>
            <th class="date">Published Date</th>
          </tr>
        </thead>

        <%-- (3) --%>
        <c:forEach var="article" items="${page.content}" varStatus="rowStatus">
          <tr>
            <td class="no">
              ${(page.number * page.size) + rowStatus.count}
            </td>
            <td class="articleClass">
              ${f:h(article.articleClass.name)}
            </td>
            <td class="title">
              ${f:h(article.title)}
            </td>
            <td class="overview">
              ${f:h(article.overview)}
            </td>
            <td class="date">
              ${f:h(article.publishedDate)}
            </td>
          </tr>
        </c:forEach>

      </table>

      <div class="paginationPart">

        <%-- ... --%>

      </div>
    </c:when>

    <%-- ... --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (2)
      - | 上記例では、条件に一致するデータが存在するかチェックを行い、一致するデータがない場合はヘッダ行も含めて表示していない。
        | 一致するデータがない場合でもヘッダ行は表示させる必要がある場合は、この分岐は不要となる。
    * - | (3)
      - | JSTLの ``<c:forEach>`` タグを使用して、取得したデータの一覧を表示する。
        | 取得したデータは、 ``Page`` オブジェクトの ``content`` プロパティにリスト形式で格納されている。

- 上記JSPで出力される画面例

 .. figure:: ./images/pagination-how_to_use_view_list_screen.png
   :alt: Screen image of content table
   :width: 100%
   :align: center


|

.. _pagination_how_to_use_make_jsp_basic_paginationlink:

ページネーションリンクの表示
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
ページ移動するためのリンク(ページネーションリンク)を表示するための実装例を以下に示す。

共通ライブラリより提供しているJSPタグライブラリを使用して、ページネーションリンクを出力する。

- :file:`include.jsp`

 共通ライブラリより提供しているJSPタグライブラリの宣言を行う。ブランクプロジェクトでは設定済みの状態となっている。

 .. code-block:: jsp

    <%@ taglib uri="http://terasoluna.org/tags" prefix="t"%>       <%-- (1) --%>
    <%@ taglib uri="http://terasoluna.org/functions" prefix="f"%>  <%-- (2) --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ページネーションリンクを表示するためのJSPタグが格納されている。
    * - | (2)
      - | ページネーションリンクを使う際に利用するJSPのELファンクションが格納されている。

- JSP

 .. code-block:: jsp

    <t:pagination page="${page}" /> <%-- (3) --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (3)
      - | ``<t:pagination>`` タグを使用する。 page属性には、 Controllerで ``Model`` に格納した ``Page`` オブジェクトを指定する。

|

- 出力されるHTML

 下記の出力例は、``"?page=0&size=6"`` を指定して検索した際の結果である。

 .. code-block:: html

     <ul>
        <li class="disabled"><a href="javascript:void(0)">&lt;&lt;</a></li>
        <li class="disabled"><a href="javascript:void(0)">&lt;</a></li>
        <li class="active"><a href="javascript:void(0)">1</a></li>
        <li><a href="?page=1&size=6">2</a></li>
        <li><a href="?page=2&size=6">3</a></li>
        <li><a href="?page=3&size=6">4</a></li>
        <li><a href="?page=4&size=6">5</a></li>
        <li><a href="?page=5&size=6">6</a></li>
        <li><a href="?page=6&size=6">7</a></li>
        <li><a href="?page=7&size=6">8</a></li>
        <li><a href="?page=8&size=6">9</a></li>
        <li><a href="?page=9&size=6">10</a></li>
        <li><a href="?page=1&size=6">&gt;</a></li>
        <li><a href="?page=9&size=6">&gt;&gt;</a></li>
    </ul>

|

| ページネーションリンク用のスタイルシートを用意しないと以下のような表示となる。
| 見てわかる通り、ページネーションリンクとして成立していない。

 .. figure:: ./images/pagination-how_to_use_jsp_not_applied_css.png
   :alt: Screen image that style sheet is not applied.
   :width: 120px
   :height: 290px

|

| ページネーションリンクとして成立する最低限のスタイルシートの定義の追加と、JSPの変更を行うと、以下のような表示となる。

- 画面イメージ

 .. figure:: ./images/pagination-how_to_use_jsp_applied_simple_css.png
   :alt: Screen image that simple style sheet applied.
   :width: 290px
   :height: 40px

- JSP

 .. code-block:: jsp

    <%-- ... --%>

    <t:pagination page="${page}"
        outerElementClass="pagination" /> <%-- (4) --%>

    <%-- ... --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (4)
      - | ページネーションリンクであることを示すクラス名を指定する。
        | クラス名を指定することでスタイルシートで指定するスタイルの適用範囲をページネーションリンクに限定することができる。

- スタイルシート

 .. code-block:: css

    .pagination li {
        display: inline;
    }

    .pagination li>a {
        margin-left: 10px;
    }

|

ページネーションリンクとして成立したが、以下2つの問題が残る。

* 押下できるリンクと押下できないリンクの区別ができない。
* 現在表示しているページ位置がわからない。

|

上記の問題を解決する手段として、Bootstrap v3.0.0のスタイルシートと適用すると、以下のような表示となる。

- 画面イメージ

 .. figure:: ./images/pagination-how_to_use_jsp_applied_bootstrap_v3_0_0_css.png
   :alt: Screen image that v3.0.0 of bootstrap is applied.
   :width: 520px
   :height: 70px

- スタイルシート

 | bootstrap v3.0.0 の cssファイルを ``$WEB_APP_ROOT/resources/vendor/bootstrap-3.0.0/css/bootstrap.css`` に配置する。
 | 以下、ページネーション関連のスタイル定義の抜粋。


 .. code-block:: css

    .pagination {
      display: inline-block;
      padding-left: 0;
      margin: 20px 0;
      border-radius: 4px;
    }

    .pagination > li {
      display: inline;
    }

    .pagination > li > a,
    .pagination > li > span {
      position: relative;
      float: left;
      padding: 6px 12px;
      margin-left: -1px;
      line-height: 1.428571429;
      text-decoration: none;
      background-color: #ffffff;
      border: 1px solid #dddddd;
    }

    .pagination > li:first-child > a,
    .pagination > li:first-child > span {
      margin-left: 0;
      border-bottom-left-radius: 4px;
      border-top-left-radius: 4px;
    }

    .pagination > li:last-child > a,
    .pagination > li:last-child > span {
      border-top-right-radius: 4px;
      border-bottom-right-radius: 4px;
    }

    .pagination > li > a:hover,
    .pagination > li > span:hover,
    .pagination > li > a:focus,
    .pagination > li > span:focus {
      background-color: #eeeeee;
    }

    .pagination > .active > a,
    .pagination > .active > span,
    .pagination > .active > a:hover,
    .pagination > .active > span:hover,
    .pagination > .active > a:focus,
    .pagination > .active > span:focus {
      z-index: 2;
      color: #ffffff;
      cursor: default;
      background-color: #428bca;
      border-color: #428bca;
    }

    .pagination > .disabled > span,
    .pagination > .disabled > a,
    .pagination > .disabled > a:hover,
    .pagination > .disabled > a:focus {
      color: #999999;
      cursor: not-allowed;
      background-color: #ffffff;
      border-color: #dddddd;
    }


- JSP

 JSPでは配置したcssファイルを読み込む定義を追加する。

 .. code-block:: jsp

    <link rel="stylesheet"
        href="${pageContext.request.contextPath}/resources/vendor/bootstrap-3.0.0/css/bootstrap.css"
        type="text/css" media="screen, projection">

|

ページネーション情報の表示
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
ページネーションに関連する情報(合計件数、合計ページ数、表示ページ数など)を表示するための実装例を以下に示す。

- 画面例

 .. figure:: ./images/pagination-how_to_use_view_pagination_info1.png
   :alt: Screen image of pagination information(total results, current pages, total pages)
   :width: 400px
   :height: 250px

- JSP

 .. code-block:: jsp

    <div>
        <fmt:formatNumber value="${page.totalElements}" /> results <%-- (1) --%>
    </div>
    <div>
        ${f:h(page.number + 1) } /       <%-- (2) --%>
        ${f:h(page.totalPages)} Pages    <%-- (3) --%>
    </div>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 検索条件に一致するデータの合計件数を表示する場合は、 ``Page`` オブジェクトの ``totalElements`` プロパティから値を取得する。
    * - | (2)
      - | 表示しているページのページ数を表示する場合は、 ``Page`` オブジェクトの ``number`` プロパティから値を取得し、``+1`` する。
        | ``Page`` オブジェクトの ``number`` プロパティは ``0`` 開始のため、 ページ番号を表示する際は ``+1`` が必要となる。
    * - | (3)
      - | 検索条件に一致するデータの合計ページ数を表示する場合は、 ``Page`` オブジェクトの ``totalPages`` プロパティから値をする。

|

該当ページの表示データ範囲を表示するための実装例を以下に示す。

- 画面例

 .. figure:: ./images/pagination-how_to_use_view_pagination_info2.png
   :alt: Screen image of pagination information(begin position, end position)
   :width: 400px
   :height: 250px

- JSP

 .. code-block:: jsp

    <div>
        <%-- (4) --%>
        <fmt:formatNumber value="${(page.number * page.size) + 1}" /> -
        <%-- (5) --%>
        <fmt:formatNumber value="${(page.number * page.size) + page.numberOfElements}" />
    </div>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (4)
      - | 開始位置を表示する場合は、 ``Page`` オブジェクトの ``number`` プロパティと ``size`` プロパティを使って計算する。
        | ``Page`` オブジェクトの ``number`` プロパティは ``0`` 開始のため、データ開始位置を表示する際は ``+1`` が必要となる。
    * - | (5)
      - | 終了位置を表示する場合は、 ``Page`` オブジェクトの ``number`` プロパティ、 ``size`` プロパティ、 ``numberOfElements`` プロパティ を使って計算する。
        | 最終ページは端数となる可能性があるので、 ``numberOfElements`` を加算する必要がある。

 .. tip:: **数値のフォーマットについて**

    表示する数値をフォーマットする必要がある場合は、 JSTLから提供されているタグライブラリ( ``<fmt:formatNumber>`` )を使用する。


|

.. _pagination_how_to_use_make_jsp_basic_search_criteria:

ページリンクで検索条件を引き継ぐ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
検索条件をページ移動時のリクエストに引き継ぐ方法を、以下に示す。

 .. figure:: ./images/pagination-how_to_use_view_take_over_search_criteria.png
   :alt: Processing image of take over search criteria
   :width: 100%
   :align: center

- JSP

 .. code-block:: jsp

    <%-- (1) --%>
    <div id="criteriaPart">
      <form:form action="${pageContext.request.contextPath}/article/list" method="get"
                 modelAttribute="articleSearchCriteriaForm">
        <form:input path="word" />
        <form:button>Search</form:button>
        <br>
      </form:form>
    </div>

    <%-- ... --%>

    <t:pagination page="${page}"
        outerElementClass="pagination"
        criteriaQuery="${f:query(articleSearchCriteriaForm)}" /> <%-- (2) --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 検索条件を指定するフォーム。
        | 検索条件として ``word`` が存在する。
    * - | (2)
      - | ページ移動時のリクエストに検索条件を引き継ぐ場合は、 \ ``criteriaQuery``\属性に\ **URLエンコーディング済みのクエリ文字列**\を指定する。
        | 検索条件をフォームオブジェクトに格納する場合は、共通ライブラリから提供しているELファクション( ``f:query(Object)`` ) を使用すると、簡単に条件を引き継ぐことができる。
        | 上記例の場合、 \ ``"?page=ページ位置&size=取得件数&word=入力値"``\という形式のクエリ文字列が生成される。
        |
        | \ ``criteriaQuery``\属性は、terasoluna-gfw-web 1.0.1.RELEASE以上で利用可能な属性である。

 .. note:: **f:query(Object) の仕様について**

    ``f:query`` の引数には、 フォームオブジェクトなどのJavaBeanと ``Map`` オブジェクトを指定することができる。
    JavaBeanの場合はプロパティ名がリクエストパラメータ名となり、 ``Map`` オブジェクトの場合はマップのキー名がリクエストパラメータとなる。
    生成されるクエリ文字列は、UTF-8のURLエンコーディングが行われる。

    terasoluna-gfw-web 5.0.1.RELEASEより、ネスト構造をもつJavaBeanと\ ``Map``\ オブジェクトを指定できるように改善されている。

    \ ``f:query``\ の詳細な仕様(URLエンコーディングの仕様など)については、「:ref:`TagLibAndELFunctionsHowToUseELFunctionQuery`」を参照されたい。

 .. warning:: **f:queryを使用して生成したクエリ文字列をqueryTmpl属性に指定した際の動作について**

    \ ``f:query``\を使用して生成したクエリ文字列をqueryTmpl属性に指定すると、URLエンコーディングが重複してしまい、特殊文字の引き継ぎが正しく行われないことが判明している。
    
    URLエンコーディングが重複してしまう事象については、terasoluna-gfw-web 1.0.1.RELEASE以上で利用可能な\ ``criteriaQuery``\属性を使用することで回避する事が出来る。
    
|

ページリンクでソート条件を引き継ぐ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
ソート条件をページ移動時のリクエストに引き継ぐ方法を、以下に示す。

- JSP

 .. code-block:: jsp

    <t:pagination page="${page}"
        outerElementClass="pagination"
        queryTmpl="page={page}&size={size}&sort={sortOrderProperty},{sortOrderDirection}" />  <%-- (1) --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ページ移動時のリクエストにソート条件を引き継ぐ場合は、 ``queryTmpl`` を指定し、クエリ文字列にソート条件を追加する。
        | ソート条件を指定するためのパラメータの仕様については、「 :ref:`ページ検索用のリクエストパラメータについて <pagination_overview_pagesearch_requestparameter>` 」を参照されたい。
        | 上記例の場合、 ``"?page=0&size=20&sort=ソート項目,ソート順(ASC or DESC)"`` がクエリ文字列となる。

|

.. _pagination_how_to_use_make_jsp_layout:

JSPの実装(レイアウト変更編)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

先頭ページと最終ページに移動するリンクの削除
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
「最初のページに移動するためのリンク」と「最後のページに移動するためのリンク」を削除するための実装例を、以下に示す。

- 画面例

 .. figure:: ./images/pagination-how_to_use_view_remove_link1.png
   :alt: Remove page link that move to first & last page
   :width: 510px
   :height: 140px

- JSP

 .. code-block:: jsp

    <t:pagination page="${page}"
        outerElementClass="pagination"
        firstLinkText=""
        lastLinkText="" /> <%-- (1) (2) --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 「最初のページに移動するためのリンク」を非表示にする場合は、 ``<t:pagination>`` タグの firstLinkText属性に ``""`` を指定する。
    * - | (2)
      - | 「最後のページに移動するためのリンク」を非表示にする場合は、 ``<t:pagination>`` タグの lastLinkText属性に ``""`` を指定する。

|


前ページと次ページに移動するリンクの削除
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
「最初のページに移動するためのリンク」と「最後のページに移動するためのリンク」を削除するための実装例を、以下に示す。

- 画面例

 .. figure:: ./images/pagination-how_to_use_view_remove_link2.png
   :alt: Remove page link that move to previous & next page
   :width: 470
   :height: 220px

- JSP

 .. code-block:: jsp

    <t:pagination page="${page}"
        outerElementClass="pagination"
        previousLinkText=""
        nextLinkText="" /> <%-- (1) (2) --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 「前のページに移動するためのリンク」を非表示にする場合は、 ``<t:pagination>`` タグの previousLinkText属性に ``""`` を指定する。
    * - | (2)
      - | 「次のページに移動するためのリンク」を非表示にする場合は、 ``<t:pagination>`` タグの nextLinkText属性に ``""`` を指定する。

|

disabled状態のリンクの削除
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| ``"disabled"`` 状態のリンクを削除するための実装例を、以下に示す。
| ``"disabled"`` 時のスタイルシートに、以下の定義を追加する。

- 画面例

 .. figure:: ./images/pagination-how_to_use_view_remove_link3.png
   :alt: Remove page link that move to previous & next page
   :width: 530
   :height: 200px

- スタイルシート

 .. code-block:: css

    .pagination .disabled {
        display: none;  /* (1) */
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ``"disabled"`` クラスの属性値として、 ``"display: none;"`` を指定する。

|

指定ページへ移動するリンクの最大表示数の変更
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
指定したページに移動するためのリンクの最大表示数を変更するための実装例を、以下に示す。

- 画面例

 .. figure:: ./images/pagination-how_to_use_view_change_maxsize.png
   :alt: change max display count of page link that move to specified page
   :width: 450
   :height: 220px

- JSP

 .. code-block:: jsp

    <t:pagination page="${page}"
        outerElementClass="pagination"
        maxDisplayCount="5" /> <%-- (1) --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 指定したページに移動するためのリンクの最大表示数を変更する場合は、 ``<t:pagination>`` タグの maxDisplayCount属性に値を指定する。

|

指定ページへ移動するリンクの削除
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 指定したページに移動するためのリンクを削除するための実装例を、以下に示す。

- 画面例

 .. figure:: ./images/pagination-how_to_use_view_remove_link4.png
   :alt: Remove page link that move to specified page
   :width: 410
   :height: 220px

- JSP

 .. code-block:: jsp

    <t:pagination page="${page}"
        outerElementClass="pagination"
        maxDisplayCount="0" /> <%-- (1) --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 指定したページに移動するためのリンクを非表示にする場合は、 ``<t:pagination>`` タグの maxDisplayCount属性に ``"0"`` を指定する。


|

JSPの実装(動作編)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ソート条件の指定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
クライアントからソート条件を指定するための実装例を、以下に示す。

- 画面例

 .. figure:: ./images/pagination-how_to_use_view_sort.png
   :alt: specify the sort condition
   :width: 100%

- JSP

 .. code-block:: jsp

    <div id="criteriaPart">
      <form:form
        action="${pageContext.request.contextPath}/article/search"
        method="get" modelAttribute="articleSearchCriteriaForm">
        <form:input path="word" />
        <%-- (1) --%>
        <form:select path="sort">
            <form:option value="publishedDate,DESC">Newest</form:option>
            <form:option value="publishedDate,ASC">Oldest</form:option>
        </form:select>
        <form:button>Search</form:button>
        <br>
      </form:form>
    </div>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | クライアントからソート条件を指定する場合は、 ソート条件を指定するためのパラメータを追加する。
        | ソート条件を指定するためのパラメータの仕様については、「 :ref:`ページ検索用のリクエストパラメータについて <pagination_overview_pagesearch_requestparameter>` 」を参照されたい。
        | 上記例では、publishedDateの昇順と降順をプルダウンで選択できるようにしている。

|

.. _PaginationHowToUseDisablePageLinkUsingJavaScript:

JavaScriptを使用したページリンクの無効化
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
デフォルトでは、\ ``"disabled"``\ 状態と\ ``"active"``\ 状態のページリンク押下時の動作を無効化するために、\ ``<t:pagination>``\ タグの\ ``disabledHref``\ 属性に\ ``"javascript:void(0)"``\ を設定している。
この状態でページリンクにフォーカスを移動又はマウスオーバーすると、 ブラウザのステータスバーに\ ``"javascript:void(0)"``\ が表示されることがある。
この挙動を変えたい場合は、JavaScriptを使用してページリンク押下時の動作を無効化する必要がある。

以下に実装例を示す。

**JSP**

.. code-block:: jsp

    <%-- (1) --%>
    <script type="text/javascript"
            src="${pageContext.request.contextPath}/resources/vendor/js/jquery.js"></script>

    <%-- (2) --%>
    <script type="text/javascript">
        $(function(){
            $(document).on("click", ".disabled a, .active a", function(){
                return false;
            });
        });
    </script>

    <%-- ... --%>

    <%-- (3) --%>
    <t:pagination page="${page}" disabledHref="#" />

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - jQueryのjsファイルを読み込む。

        上記例では、JavaScriptを使用してページリンク押下時の動作を無効化するためにjQueryのAPIを利用する。
    * - | (2)
      - jQueryのAPIを使用して、\ ``"disabled"``\ 状態と\ ``"active"``\ 状態のページリンクのクリックイベントを無効化する。

        ただし、\ ``<t:pagination>``\ タグの\ ``enableLinkOfCurrentPage``\ 属性に\ ``"true"``\ を指定している場合は、\ ``"active"``\ 状態のページリンクのクリックイベントを無効化してはいけない。
    * - | (3)
      - \ ``disabledHref``\ 属性に\ ``"#"``\ を指定する。

|

.. _paginatin_appendix:

Appendix
--------------------------------------------------------------------------------

.. _paginatin_appendix_pageableHandlerMethodArgumentResolver:

``PageableHandlerMethodArgumentResolver`` のプロパティ値について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| ``PageableHandlerMethodArgumentResolver`` で指定できるプロパティは以下の通り。
| アプリケーションの要件に応じて、値を変更すること。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.55\linewidth}|p{0.15\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 55 15

    * - 項番
      - プロパティ名
      - 説明
      - デフォルト値
    * - 1.
      - maxPageSize
      - | 取得件数として許可する最大値を指定する。
        | 指定された取得件数が ``maxPageSize`` を超えていた場合は、 ``maxPageSize`` が取得件数となる。
      - |  `2000`
    * - 2.
      - fallbackPageable
      - | アプリケーション全体のページ位置、取得件数、ソート条件のデフォルト値を指定する。
        | ページ位置、取得件数、ソート条件が指定されていない場合は、fallbackPageableに設定されている値が適用される。
      - | ページ位置 : `0`
        | 取得件数 : `20`
        | ソート条件 : `null`
    * - 3.
      - oneIndexedParameters
      - | ページ位置の開始値を指定する。
        | `false` を指定した場合はページ位置の開始値は `0` となり、 `true` を指定した場合はページ位置の開始値は `1` となる。
      - | `false`
    * - 4.
      - pageParameterName
      - | ページ位置を指定するためのリクエストパラメータ名を指定する。
      - | ``"page"``
    * - 5.
      - sizeParameterName
      - | 取得件数を指定するためのリクエストパラメータ名を指定する。
      - | ``"size"``
    * - 6.
      - prefix
      - | ページ位置と取得件数を指定するためのリクエストパラメータの接頭辞(ネームスペース)を指定する。
        | デフォルトのパラメータ名がアプリケーションで使用するパラメータと衝突する場合は、ネームスペースを指定することでリクエストパラメータ名の衝突を防ぐことが出来る。
        | prefixを指定すると、ページ位置を指定するためのリクエストパラメータ名は ``prefix + pageParameterName`` 、取得件数を指定するためのリクエストパラメータ名は ``prefix + sizeParameterName`` となる。
      - | ``""``
        | (ネームスペースなし)
    * - 7.
      - qualifierDelimiter
      - | 同一リクエストで複数のページ検索が必要になる場合、ページ検索に必要な情報(検索対象のページ位置、取得件数など)を区別するために、リクエストパラメータ名は ``qualifier + delimiter + 標準パラメータ名`` の形式で指定する。
        | 本プロパティは、上記形式の中の ``delimiter`` の値を設定する。
        | この設定を変更する場合は、 ``SortHandlerMethodArgumentResolver`` の ``qualifierDelimiter`` 設定も合わせて変更する必要がある。
      - | ``"_"``

 .. note:: **maxPageSizeの設定値について**

    デフォルト値は ``2000`` であるが、アプリケーションが許容する最大値に設定を変更することを推奨する。
    アプリケーションが許可する最大値が `100` ならば、maxPageSizeも `100` に設定する。

 .. note:: **fallbackPageableの設定方法について**

    アプリケーション全体に適用するデフォルト値を変更する場合は、 ``fallbackPageable`` プロパティにデフォルト値が定義されている ``Pageable`` ( ``org.springframework.data.domain.PageRequest`` ) オブジェクトを設定する。
    ソート条件のデフォルト値を変更する場合は、 ``SortHandlerMethodArgumentResolver`` の ``fallbackSort`` プロパティにデフォルト値が定義されている  ``org.springframework.data.domain.Sort`` オブジェクトを設定する。

|

開発するアプリケーション毎に変更が想定される以下の項目について、デフォルト値を変更する際の設定例を以下に示す。

* 取得件数として許可する最大値( ``maxPageSize`` )
* アプリケーション全体のページ位置、取得件数のデフォルト値( ``fallbackPageable`` )
* ソート条件のデフォルト値( ``fallbackSort`` )

 .. code-block:: xml

    <mvc:annotation-driven>
        <mvc:argument-resolvers>
            <bean
                class="org.springframework.data.web.PageableHandlerMethodArgumentResolver">
                <!-- (1) -->
                <property name="maxPageSize" value="100" />
                <!-- (2) -->
                <property name="fallbackPageable">
                    <bean class="org.springframework.data.domain.PageRequest">
                        <!-- (3) -->
                        <constructor-arg index="0" value="0" />
                        <!-- (4) -->
                        <constructor-arg index="1" value="50" />
                    </bean>
                </property>
                <!-- (5) -->
                <constructor-arg index="0">
                    <bean class="org.springframework.data.web.SortHandlerMethodArgumentResolver">
                        <!-- (6) -->
                        <property name="fallbackSort">
                            <bean class="org.springframework.data.domain.Sort">
                                <!-- (7) -->
                                <constructor-arg index="0">
                                    <list>
                                        <!-- (8) -->
                                        <bean class="org.springframework.data.domain.Sort.Order">
                                            <!-- (9) -->
                                            <constructor-arg index="0" value="DESC" />
                                            <!-- (10) -->
                                            <constructor-arg index="1" value="lastModifiedDate" />
                                        </bean>
                                        <!-- (8) -->
                                        <bean class="org.springframework.data.domain.Sort.Order">
                                            <constructor-arg index="0" value="ASC" />
                                            <constructor-arg index="1" value="id" />
                                        </bean>
                                    </list>
                                </constructor-arg>
                            </bean>
                        </property>
                    </bean>
                </constructor-arg>
            </bean>
        </mvc:argument-resolvers>
    </mvc:annotation-driven>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 上記例では取得件数の最大値を `100` に設定している。 取得件数(size)に `101` 以上が指定された場合は、 `100` に切り捨てて検索が行われる。
    * - | (2)
      - | ``org.springframework.data.domain.PageRequest`` のインスタンスを生成し、 ``fallbackPageable`` に設定する。
    * - | (3)
      - | ``PageRequest`` のコンストラクタの第1引数に、ページ位置のデフォルト値を指定する。
        |  上記例では `0` を指定しているため、デフォルト値は変更していない。
    * - | (4)
      - | ``PageRequest`` のコンストラクタの第2引数に、取得件数のデフォルト値を指定する。
        | 上記例ではリクエストパラメータに取得件数の指定がない場合の取得件数は `50` となる。
    * - | (5)
      - | ``PageableHandlerMethodArgumentResolver`` のコンストラクタとして、 ``SortHandlerMethodArgumentResolver`` のインスタンスを設定する。
    * - | (6)
      - | ``Sort`` のインスタンスを生成し、 ``fallbackSort`` に設定する。
    * - | (7)
      - | ``Sort`` のコンストラクタの第1引数に、 デフォルト値として使用する ``Order`` オブジェクトのリストを設定する。
    * - | (8)
      - | ``Order`` のインスタンスを生成し、 デフォルト値として使用する ``Order`` オブジェクトのリストに追加する。
        | 上記例ではリクエストパラメータにソート条件の指定がない場合は ``"ORDER BY x.lastModifiedDate DESC, x.id ASC"`` というソート条件がQueryに追加される。
    * - | (9)
      - | ``Order`` のコンストラクタの第1引数に、ソート順(ASC/DESC)を指定する。
    * - | (10)
      - | ``Order`` のコンストラクタの第2引数に、ソート項目を指定する。

|

.. _paginatin_appendix_sortHandlerMethodArgumentResolver:

``SortHandlerMethodArgumentResolver`` のプロパティ値について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| ``SortHandlerMethodArgumentResolver`` で指定できるプロパティは以下の通り。
| アプリケーションの要件に応じて、値を変更すること。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.55\linewidth}|p{0.15\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 55 15

    * - 項番
      - プロパティ名
      - 説明
      - デフォルト値
    * - 1.
      - fallbackSort
      - | アプリケーション全体のソート条件のデフォルト値を指定する。
        | ソート条件が指定されていない場合は、fallbackSortに設定されている値が適用される。
      - | `null`
        | (ソート条件なし)
    * - 2.
      - sortParameter
      - | ソート条件を指定するためのリクエストパラメータ名を指定する。
        | デフォルトのパラメータ名がアプリケーションで使用するパラメータと衝突する場合は、リクエストパラメータ名を変更することで衝突を防ぐことができる。
      - | ``"sort"``
    * - 3.
      - propertyDelimiter
      - | ソート項目及びソート順(ASC,DESC)の区切り文字を指定する。
      - | ``","``
    * - 4.
      - qualifierDelimiter
      - | 同一リクエストで複数のページ検索が必要になる場合、ページ検索に必要な情報(ソート条件)を区別するために、リクエストパラメータ名は ``qualifier + delimiter + sortParameter`` の形式で指定する。
        | 本プロパティは、上記形式の中の ``delimiter`` の値を設定する。
      - | ``"_"``

.. raw:: latex

   \newpage

