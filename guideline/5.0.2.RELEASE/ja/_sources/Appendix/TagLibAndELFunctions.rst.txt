共通ライブラリが提供するJSP Tag Library と EL Functions
================================================================================

.. _TagLibAndELFunctionsOverview:

Overview
--------------------------------------------------------------------------------

共通ライブラリでは、JSPの実装をサポートする機能として、
以下に示すJSP Tag Library と EL Functionsを提供している。

.. _TagLibAndELFunctionsOverviewTagLibs:

JSP Tag Library
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
共通ライブラリから提供しているJSP Tag Libraryを以下に示す。

 .. tabularcolumns:: |p{0.5\linewidth}|p{0.25\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 5 25 60

    * - | 項番
      - | タグ名
      - | 概要
    * - 1.
      - :ref:`TaglibAndELFunctionsHowToUseTaglibPagination`
      - ページネーションリンクを出力する。
    * - 2.
      - :ref:`TaglibAndELFunctionsHowToUseTaglibMessagesPanel`
      - 処理結果メッセージを出力する。
    * - 3.
      - :ref:`TaglibAndELFunctionsHowToUseTaglibTransaction`
      - トランザクショントークンをhidden項目として出力する。

.. _TagLibAndELFunctionsOverviewELFunctions:

EL Functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
共通ライブラリから提供しているEL Functionsを以下に示す。

**XSS対策関連**

 .. tabularcolumns:: |p{0.5\linewidth}|p{0.25\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 5 25 60

    * - | 項番
      - | 関数名
      - | 概要
    * - 1.
      - :ref:`TaglibAndELFunctionsHowToUseELFunctionH`
      - 指定されたオブジェクトを文字列に変換し、変換した文字列内のHTML特殊文字をエスケープする。
    * - 2.
      - :ref:`TaglibAndELFunctionsHowToUseELFunctionJs`
      - 指定された文字列内のJavaScript特殊文字をエスケープする。
    * - 3.
      - :ref:`TaglibAndELFunctionsHowToUseELFunctionHjs`
      - 指定された文字列内のJavaScript特殊文字をエスケープ後、HTML特殊文字をエスケープする。(\ ``f:h(f:js())``\ のショートカット関数)

**URL関連**

 .. tabularcolumns:: |p{0.5\linewidth}|p{0.25\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 5 25 60

    * - | 項番
      - | 関数名
      - | 概要
    * - 4.
      - :ref:`TaglibAndELFunctionsHowToUseELFunctionQuery`
      - 指定されたオブジェクトから、UTF-8でURLエンコーディングされたクエリ文字列を生成する。
    * - 5.
      - :ref:`TaglibAndELFunctionsHowToUseELFunctionU`
      - 指定された文字列をUTF-8でURLエンコーディングする。

**DOM関連**

 .. tabularcolumns:: |p{0.5\linewidth}|p{0.25\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 5 25 60

    * - | 項番
      - | 関数名
      - | 概要
    * - 6.
      - :ref:`TaglibAndELFunctionsHowToUseELFunctionLink`
      - 指定されたURLにジャンプするハイパーリンク(\ ``<a>``\ タグ)を生成する。
    * - 7.
      - :ref:`TaglibAndELFunctionsHowToUseELFunctionBr`
      - 指定された文字列内の改行コードを\ ``<br />``\ タグに変換する。

**ユーティリティ**

 .. tabularcolumns:: |p{0.5\linewidth}|p{0.25\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 5 25 60

    * - | 項番
      - | 関数名
      - | 概要
    * - 8.
      - :ref:`TaglibAndELFunctionsHowToUseELFunctionCut`
      - 指定された文字列から、指定された文字数を抜き出す。

|

.. _TagLibAndELFunctionsHowToUse:

How to use
--------------------------------------------------------------------------------

共通ライブラリから提供している JSP Tag Library と EL関数の使用方法を以下に示す。
なお、他の章で使用方法の説明があるものについては、該当箇所へのハイパーリンクを貼っている。

|

.. _TaglibAndELFunctionsHowToUseTaglibPagination:

<t:pagination>
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
\ ``<t:pagination>``\ タグは、
ページ検索の結果(\ ``org.springframework.data.domain.Page``\ )に格納されている情報を参照して、
ページネーションリンクを出力するJSP Tag Libraryである。

ページネーション機能の説明及び本タグの使用方法は、「:doc:`../ArchitectureInDetail/Pagination`」の以下の節を参照されたい。

* ページネーションリンクについては、「:ref:`pagination_overview_paginationlink`」
* 本タグのパラメータ値については、「:ref:`pagination_overview_paginationlink_taglibparameters`」
* 本タグを使用したJSPの基本的な実装方法については、「:ref:`pagination_how_to_use_make_jsp_basic_paginationlink`」
* ページネーションリンクのレイアウトの変更方法については、「:ref:`pagination_how_to_use_make_jsp_layout`」

|

.. _TaglibAndELFunctionsHowToUseTaglibMessagespanel:

<t:messagesPanel>
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
\ ``<t:messagesPanel>``\ タグは、
処理結果メッセージ(\ ``org.terasoluna.gfw.common.message.ResultMessage``\ や例外が保持するメッセージなど)を出力するJSP Tag Libraryである。

本タグの使用方法は、「:doc:`../ArchitectureInDetail/MessageManagement`」の以下の節を参照されたい。

* 本タグを使用したメッセージの表示方法については、「:ref:`message-display`」
* 本タグのパラメータ値については、「:ref:`message-management-messagepanel-attribute`」

|

.. _TaglibAndELFunctionsHowToUseTaglibTransaction:

<t:transaction>
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
\ ``<t:transaction>``\ タグは、トランザクショントークンをhidden項目(\ ``<input type="hidden">"``\ )として出力するJSP Tag Libraryである。

トランザクショントークンチェック機能の説明及び本タグの使用方法は、「:doc:`../ArchitectureInDetail/DoubleSubmitProtection`」の以下の節を参照されたい。

* トランザクショントークンチェック機能については、「:ref:`doubleSubmit_how_to_use_transaction_token_check`」
* 本タグの使用方法については、「:ref:`doubleSubmit_how_to_use_transaction_token_check_jsp`」

.. note::

   本タグは、HTML標準の\ ``<form>``\ タグを使用する際にトランザクショントークンをサーバに送信するために使用する。

   Spring Framework提供の\ ``<form:form>``\ タグ(JSP Tag Library)を使用する際は、
   共通ライブラリから提供している\ ``org.terasoluna.gfw.web.token.transaction.TransactionTokenRequestDataValueProcessor``\ が自動でトランザクショントークンを埋め込む仕組みになっているため、
   本タグを使用する必要はない。

|

.. _TaglibAndELFunctionsHowToUseELFunctionH:

f:h()
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
\ ``f:h()``\ は、引数に指定されたオブジェクトを文字列に変換し、変換した文字列内のHTML特殊文字をエスケープするEL Functionである。

HTML特殊文字とエスケープ仕様については、「:ref:`xss_how_to_use_ouput_escaping`」を参照されたい。

f:h() 関数仕様
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**引数**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - 項番
      - 型
      - 説明
    * - 1.
      - \ ``java.lang.Object``\
      - HTML特殊文字が含まれる可能性があるオブジェクト

 .. note::

    指定されたオブジェクトは、

    * 配列の場合は、\ ``java.util.Arrays#toString``\ メソッド
    * 配列以外の場合は、指定されたオブジェクトの \ ``toString``\ メソッド

    を使用して文字列に変換される。


**戻り値**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - 項番
      - 型
      - 説明
    * - 1.
      - \ ``java.lang.String``\
      - HTMLエスケープ後の文字列

        引数で指定さらたオブジェクトが\ ``null``\ の場合は、空文字(\ ``""``\ )を返却する。

f:h() 使用方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``f:h()``\ の使用方法については、「:ref:`xss_how_to_use_h_function_example`」を参照されたい。

|

.. _TaglibAndELFunctionsHowToUseELFunctionJs:

f:js()
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
\ ``f:js()``\ は、引数に指定された文字列内のJavaScript特殊文字をエスケープするEL Functionである。

JavaScript特殊文字とエスケープ仕様については、「:ref:`xss_how_to_use_javascript_escaping`」を参照されたい。

f:js() 関数仕様
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**引数**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - 項番
      - 型
      - 説明
    * - 1.
      - \ ``java.lang.String``\
      - JavaScript特殊文字が含まれる可能性がある文字列

**戻り値**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - 項番
      - 型
      - 説明
    * - 1.
      - \ ``java.lang.String``\
      - JavaScriptエスケープ後の文字列

        引数で指定さらた文字列が\ ``null``\ の場合は、空文字(\ ``""``\ )を返却する。


f:js() 使用方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``f:js()``\ の使用方法については、「:ref:`xss_how_to_use_js_function_example`」を参照されたい。

|

.. _TaglibAndELFunctionsHowToUseELFunctionHjs:

f:hjs()
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
\ ``f:hjs()``\ は、引数に指定された文字列内のJavaScript特殊文字をエスケープした後に、
HTML特殊文字をエスケープするEL Function(\ ``f:h(f:js())``\ のショートカット関数)である。

* 本関数の用途については、「:ref:`xss_how_to_use_event_handler_escaping`」を参照されたい。
* JavaScript特殊文字とエスケープ仕様については、「:ref:`xss_how_to_use_javascript_escaping`」を参照されたい。
* HTML特殊文字とエスケープ仕様については、「:ref:`xss_how_to_use_ouput_escaping`」を参照されたい。


f:hjs() 関数仕様
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**引数**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - 項番
      - 型
      - 説明
    * - 1.
      - \ ``java.lang.String``\
      - JavaScript特殊文字又はHTML特殊文字が含まれる可能性がある文字列

**戻り値**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - 項番
      - 型
      - 説明
    * - 1.
      - \ ``java.lang.String``\
      - JavaScript及びHTMLエスケープ後の文字列

        引数で指定さらた文字列が\ ``null``\ の場合は、空文字(\ ``""``\ )を返却する。

f:hjs() 使用方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``f:hjs()``\ の使用方法については、「:ref:`xss_how_to_use_hjs_function_example`」を参照されたい。

|

.. _TaglibAndELFunctionsHowToUseELFunctionQuery:

f:query()
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
\ ``f:query()``\ は、引数に指定されたJavaBean(フォームオブジェクト)又は\ ``java.util.Map``\ オブジェクトから、
クエリ文字列を生成するEL Functionである。
クエリ文字列内のパラメータ名とパラメータ値は、UTF-8でURLエンコーディングされる。

URLエンコーディング仕様を以下に示す。

本関数では、クエリ文字列のパラメータ名とパラメータ値に対して、\ `RFC 3986 <http://www.ietf.org/rfc/rfc3986.txt>`_\ ベースのURLエンコーディングを行う。
RFC 3986では、クエリ文字列のパート以下のように定義している。

.. figure:: ./images_TagLibAndELFunctions/TagLibAndELFunctionsRFC3986UriSyntax.png
    :width: 90%

* query = \*( pchar / \ ``"/"``\  / \ ``"?"``\ )
* pchar = unreserved / pct-encoded / sub-delims / \ ``":"``\  / \ ``"@"``\
* unreserved = ALPHA / DIGIT / \ ``"-"``\  / \ ``"."``\  / \ ``"_"``\  / \ ``"~"``\
* sub-delims = \ ``"!"``\  / \ ``"$"``\  / \ ``"&"``\  / \ ``"'"``\  / \ ``"("``\  / \ ``")"``\  / \ ``"*"``\  / \ ``"+"``\  / \ ``","``\  / \ ``";"``\  / \ ``"="``\
* pct-encoded = \ ``"%"``\  HEXDIG HEXDIG

本関数では、クエリ文字列として使用できる文字のうち、

* \ ``"="``\  (パラメータ名とパラメータ値のセパレータ文字)
* \ ``"&"``\  (複数のパラメータを扱う場合のセパレータ文字)
* \ ``"+"``\  (HTMLのformからサブミットした時に半角スペースを表す文字)

をpct-encoded形式の文字列にエンコーディングする。

f:query() 関数仕様
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**引数**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - 項番
      - 型
      - 説明
    * - 1.
      - \ ``java.lang.Object``\
      - クエリ文字列の生成元となるオブジェクト(JavaBean又は\ ``Map``\ )

        JavaBeanを指定した場合はプロパティ名がリクエストパラメータ名となり、\ ``Map``\ を指定した場合はキー名がリクエストパラメータとなる。

        JavaBeanのプロパティ及び\ ``Map``\ の値としてサポートしている型は以下の通りである。

        * \ ``Iterable``\ インタフェースの実装クラス
        * 配列
        * \ ``Map``\ インタフェースの実装クラス
        * JavaBean
        * シンプル型 (\ ``DefaultFormattingConversionService``\ を使って\ ``String``\ 型へ変換可能なクラス)

        terasoluna-gfw-web 5.0.1.RELEASEより、ネスト構造をもつJavaBean及び\ ``Map``\ を指定できるように改善されている。


 .. note::

    指定されたオブジェクトのシンプル型のプロパティ値は、
    \ ``org.springframework.format.support.DefaultFormattingConversionService``\ の \ ``convert``\ メソッドを使用して文字列に変換される。
    \ ``ConversionService``\ については、
    \ `Spring Framework Reference Documentation(Spring Type Conversion) <http://docs.spring.io/spring/docs/4.1.9.RELEASE/spring-framework-reference/html/validation.html#core-convert>`_\ を参照されたい。


**戻り値**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - 項番
      - 型
      - 説明
    * - 1.
      - \ ``java.lang.String``\
      - 引数で指定されたオブジェクトを元に生成したクエリ文字列(UTF-8でURLエンコーディング済みの文字列)

        引数で指定されたオブジェクトが、JavaBean又は\ ``Map``\ 以外の場合は、空文字(\ ``""``\ )を返却する。

 .. note:: **クエリ文字列への変換ルール**

    \ ``f:query()``\ は、Spring Web MVCのバインディング処理で扱うことができる形式に変換している。
    具体的には以下のルールでクエリ文字列に変換している。

    **[リクエストパラメータ名]**

     .. tabularcolumns:: |p{0.45\linewidth}|p{0.30\linewidth}|p{0.25\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 45 30 25

        * - 条件
          - パラメータ名の変換仕様
          - 変換例
        * - プロパティの型が\ ``Iterable``\ の実装クラス又は配列の場合
          - プロパティ名 + \ ``[要素位置]``\
          - \ ``status[0]=accepting``\
        * - プロパティの型が\ ``Iterable``\ の実装クラス又は配列で値の要素が空の場合
          - | プロパティ名
            | (\ ``[要素位置]``\ は付与しない)
          -  \ ``status=``\
        * - プロパティの型が\ ``Map``\ の実装クラスの場合
          - プロパティ名 + \ ``[Mapのキー名]``\
          - \ ``status[accepting]=Accepting Order``\
        * - プロパティの型(\ ``Iterable``\、配列、\ ``Map``\ の要素型)がJavaBeanの場合
          - プロパティ名を\ ``"."``\ (ドット)でつなげた値
          - | \ ``mainContract.name=xxx``\
            | \ ``subContracts[0].name=xxx``\
        * - プロパティの型がシンプル型の場合
          - プロパティ名
          - \ ``userId=xxx``\
        * - プロパティの値が\ ``null``\ の場合
          - \ ``_``\ (アンダースコア) + プロパティ名
          - | \ ``_mainContract.name=``\
            | \ ``_status[0]=``\
            | \ ``_status[accepting]=``\

    **[リクエストパラメータ値]**

     .. tabularcolumns:: |p{0.45\linewidth}|p{0.30\linewidth}|p{0.25\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 45 30 25

        * - 条件
          - パラメータ値の変換仕様
          - 変換例
        * - プロパティの値が\ ``null``\ の場合
          - ブランク文字列
          - \ ``_userId=``\
        * - プロパティの型が\ ``Iterable``\ の実装クラス又は配列で値の要素が空の場合
          - ブランク文字列
          - \ ``status=``\
        * - プロパティの値が\ ``null``\ でない場合
          - \ ``DefaultFormattingConversionService``\ を使って\ ``String``\ 型へ変換した値
          - \ ``targetDate=20150801``\

f:query() 使用方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``f:query()``\ の使用方法については、「:ref:`pagination_how_to_use_make_jsp_basic_search_criteria`」を参照されたい。
ここでは、ページネーションリンクを使用してページを切り替える際に、検索条件を引き継ぐ際の手段として、本関数を使用している。
また、関数の仕様と注意点についても記載しているので、これについても一読されたい。

|

.. _TaglibAndELFunctionsHowToUseELFunctionU:

f:u()
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
\ ``f:u()``\ は、引数に指定された文字列をUTF-8でURLエンコーディングするEL Functionである。


本関数は、クエリ文字列内のパラメータ値に設定する値をURLエンコーディングするために用意している。
URLエンコーディング仕様は、「:ref:`TaglibAndELFunctionsHowToUseELFunctionQuery`」を参照されたい。

f:u() 関数仕様
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**引数**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - 項番
      - 型
      - 説明
    * - 1.
      - \ ``java.lang.String``\
      - URLエンコードが必要な文字が含まれる可能性がある文字列

**戻り値**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - 項番
      - 型
      - 説明
    * - 1.
      - \ ``java.lang.String``\
      - URLエンコード後の文字列

        引数で指定さらた文字列が\ ``null``\ の場合は、空文字(\ ``""``\ )を返却する。

f:u() 使用方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. code-block:: jsp

    <div id="url">
        <a href="https://search.yahoo.com/search?p=${f:u(bean.searchString)}">  <!-- (1) -->
            Go to Yahoo Search
        </a>
    </div>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - 上記例では、本関数を使用してURLエンコードした値を検索サイトのリクエストパラメータに設定している。

|

.. _TaglibAndELFunctionsHowToUseELFunctionLink:

f:link()
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
\ ``f:link()``\ は、引数に指定されたURLにジャンプするためのハイパーリンク(\ ``<a>``\ タグ)を出力するEL Functionである。

.. warning::

    本関数では、URLエンコーディングや特殊文字のエスケープ処理は行われない点に注意すること。

f:link() 関数仕様
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**引数**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - 項番
      - 型
      - 説明
    * - 1.
      - \ ``java.lang.String``\
      - リンク先のURL文字列

        URL文字列は、HTTP又はHTTPSスキーマのURL形式である必要がある。
        （e.g : \ ``http://hostname:80/terasoluna/global.ex?id=123``\ ）

**戻り値**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - 項番
      - 型
      - 説明
    * - 1.
      - \ ``java.lang.String``\
      - 引数に指定された文字列を元に生成したハイパーリンク(\ ``<a>``\ タグ)

        引数に指定された文字列が、

        * 引数で指定さらた文字列が\ ``null``\ の場合は、空文字(\ ``""``\ )
        * HTTP又はHTTPSスキーマのURL形式でない場合は、ハイパーリンクを生成せず入力値の文字列

        を返却する。

f:link() 使用方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**実装例**

.. code-block:: jsp

    <div id="link">
        ${f:link(bean.httpUrl)}  <!-- (1) -->
    </div>

**出力例**

.. code-block:: html

    <div id="link">
        <a href="http://terasoluna.org/">http://terasoluna.org/</a>  <!-- (2) -->
    </div>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 引数に指定されたURL文字列からハイパーリンクを生成する。
    * - | (2)
      - | 引数で指定したURL文字列が、\ ``<a>``\ タグの \ ``href``\ 属性と、ハイパーリンクのリンク名に設定される。

.. warning::

    URLにリクエストパラメータを付加する場合は、リクエストパラメータの値はURLエンコーディングする必要がある。
    リクエストパラメータを付加する場合は、\ ``f:query()``\ 関数や\ ``f:u()``\ 関数を使用して、
    リクエストパラメータの値を適切にURLエンコーディングすること。

    また、戻り値の説明でも記載しているが、引数のURL文字列の形式が適切でない場合は、
    ハイパーリンクを生成せず入力値の文字列を返却する仕様としている。
    そのため、引数に指定するURL文字列としてユーザからの入力値を使用する場合は、
    文字列出力処理と同様のHTML特殊文字のエスケープ処理(:doc:`../Security/XSS`)が必要になるケースがある。

|

.. _TaglibAndELFunctionsHowToUseELFunctionBr:

f:br()
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
\ ``f:br()``\ は、引数に指定された文字列内の改行コード（\ ``CRLF``\ , \ ``LF``\ , \ ``CR``\ ）を\ ``<br />``\ タグに変換するEL Functionである。

.. tip::

    改行コードを含む文字列をブラウザ上の表示として改行する場合は、改行コードを\ ``<br />``\ タグに変換する必要がある。

    例えば、入力画面のテキストアリア(\ ``<textarea>``\ )で入力された文字列を、
    確認画面や完了画面などで入力さらた状態のまま表示する際に、本関数を使用するとよい。

f:br() 関数仕様
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**引数**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - 項番
      - 型
      - 説明
    * - 1.
      - \ ``java.lang.String``\
      - 改行コードが含まれる可能性がある文字列

**戻り値**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - 項番
      - 型
      - 説明
    * - 1.
      - \ ``java.lang.String``\
      - 変換後の文字列

        引数で指定さらた文字列が\ ``null``\ の場合は、空文字(\ ``""``\ )を返却する。

f:br() 使用方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. code-block:: jsp

    <div id="text">
        ${f:br(f:h(bean.text))}">  <!-- (1) -->
    </div>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - 引数で指定された文字列内の改行コードを\ ``<br />``\ タグに変換することで、ブラウザ上の表示を改行する。

.. note::

    文字列を画面上に表示する際は、「:doc:`../Security/XSS`」としてHTML特殊文字をエスケープする必要がある。

    \ ``f:br()``\ 関数を使用して改行コードを\ ``<br />``\ タグに変換する場合は、
    上記例のように、HTML特殊文字をエスケープした文字列を\ ``f:br()``\ の引数として渡す必要がある。

    \ ``f:br()``\ を使用して改行コードを\ ``<br />``\ タグに変換した文字列を、
    \ ``f:h()``\ 関数の引数に渡すと、\ ``"<br />"``\ という文字がブラウザ上に表示されてしまうため、
    関数を呼び出す順番に注意すること。

|

.. _TaglibAndELFunctionsHowToUseELFunctionCut:

f:cut()
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
\ ``f:cut()``\ は、引数に指定された文字列の先頭から、引数で指定された文字数までの文字列を切り出すEL Functionである。


f:cut() 関数仕様
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**引数**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - 項番
      - 型
      - 説明
    * - 1.
      - \ ``java.lang.String``\
      - 切り出し元となる文字列
    * - 2.
      - \ ``int``\
      - 切り出す文字数

**戻り値**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - 項番
      - 型
      - 説明
    * - 1.
      - \ ``java.lang.String``\
      - 切り出した文字列(指定された文字数を超えている部分が破棄された文字列)

        引数で指定さらた文字列が\ ``null``\ の場合は、空文字(\ ``""``\ )を返却する。

f:cut() 使用方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. code-block:: jsp

    <div id="cut">
        ${f:h(f:cut(bean.originText, 5))}  <!-- (1) -->
    </div>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 上記例では、引数に指定した文字列の先頭5文字を切り出して、画面上に表示している。

.. note::

    切り出した文字列を画面上に表示する際は、「:doc:`../Security/XSS`」としてHTML特殊文字をエスケープする必要がある。
    上記例では、\ ``f:h()``\ 関数を使用してエスケープしている。

.. raw:: latex

  \newpage
