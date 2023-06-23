XSS対策
================================================================================

.. only:: html

 .. contents:: 目次
    :local:

.. _SpringSecurityXSS:

Overview
--------------------------------------------------------------------------------

クロスサイトスクリプティング(以下、XSSと略す)について説明する。
クロスサイトスクリプティングとは、アプリケーションのセキュリティ上の不備を意図的に利用し、サイト間を横断して悪意のあるスクリプトを混入させることである。
例えば、ウェブアプリケーションが入力したデータ（フォーム入力など）を、適切にエスケープしないまま、HTML上に出力することにより、入力値に存在するタグなどの文字が、そのままHTMLとして解釈される。
悪意のある値が入力された状態で、スクリプトを起動させることにより、クッキーの改ざんや、クッキーの値を取得することによる、セッションハイジャックなどの攻撃が行えてしまう。

Stored, Reflected XSS Attacks
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

XSS攻撃は、大きく二つのカテゴリに分けられる。

**Stored XSS Attacks**

Stored XSS Attacksとは、悪意のあるコードが、永久的にターゲットサーバ上(データベース等)に格納されていることである。
ユーザーは、格納されている情報を要求するときに、サーバから悪意のあるスクリプトを取得し、実行してしまう。

**Reflected XSS Attacks**

Reflected attacksとは、リクエストの一部としてサーバに送信された悪意のあるコードが、エラーメッセージ、検索結果、その他いろいろなレスポンスからリフレクションされることである。
ユーザーが、悪意のあるリンクをクリックするか、特別に細工されたフォームを送信すると、挿入されたコードは、ユーザーのブラウザに、攻撃を反映した結果を返却する。
その結果、信頼できるサーバからきた値のため、ブラウザは悪意のあるコードを実行してしまう。

Stored XSS Attacks、Reflected XSS Attacksともに、出力値をエスケープすることで防ぐことができる。

How to use
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ユーザーの入力を、そのまま出力している場合、XSSの脆弱性にさらされている。
したがって、XSSの脆弱性に対する対抗措置として、HTMLのマークアップ言語で、特定の意味を持つ文字をエスケープする必要がある。

必要に応じて、3種類のエスケープを使い分けること。

エスケープの種類:

 * Output Escaping
 * JavaScript Escaping
 * Event handler Escaping

.. _xss_how_to_use_ouput_escaping:

Output Escaping
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

XSSの脆弱性への対応としては、HTML特殊文字をエスケープすることが基本である。
エスケープが必要なHTML上の特殊文字の例と、エスケープ後の例は、以下の通りである。

.. tabularcolumns:: |p{0.50\linewidth}|p{0.50\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 50 50

   * - | エスケープ前
     - | エスケープ後
   * - | ``&``
     - | ``&amp;``
   * - | ``<``
     - | ``&lt;``
   * - | ``>``
     - | ``&gt;``
   * - | ``"``
     - | ``&quot;``
   * - | ``'``
     - | ``&#39;``

XSSを防ぐために、文字列として出力するすべての表示項目に、\ ``f:h()``\ を使用すること。
入力値を、別画面に再出力するアプリケーションを例に、説明する。

出力値をエスケープしない脆弱性のある例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

本例は、あくまで参考例として載せているだけなので、以下のような実装は、決して行わないこと。

**出力画面の実装**

.. code-block:: jsp

    <!-- omitted -->
    <tr>
        <td>Job</td>
        <td>${customerForm.job}</td>  <!-- (1) -->
    </tr>
    <!-- omitted -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | customerFormのフィールドである、jobをエスケープせず出力している。

入力画面のJobフィールドに、<script>タグを入力する。

.. figure:: ./images_XSS/xss_screen_input_html_tag.png
   :alt: input_html_tag
   :width: 80%
   :align: center

   **Picture - Input HTML Tag**

| <script>タグとして認識され、ダイアログボックスが表示されてしまう。

.. figure:: ./images_XSS/xss_screen_no_escape_result.png
   :alt: no_escape_result
   :width: 60%
   :align: center

   **Picture - No Escape Result**

.. _xss_how_to_use_h_function_example:

出力値をf:h()関数でエスケープする例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""


**出力画面の実装**

.. code-block:: jsp

    <!-- omitted -->
    <tr>
        <td>Job</td>
        <td>${f:h(customerForm.job)}</td>  <!-- (1) -->
    </tr>
    .<!-- omitted -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | EL式の\ ``f:h()``\ を使用することにより、エスケープして出力している。

入力画面のJobフィールドに<script>タグを入力する。

.. figure:: ./images_XSS/xss_screen_input_html_tag.png
   :alt: input_html_tag
   :width: 80%
   :align: center

   **Picture - Input HTML Tag**

| 特殊文字がエスケープされることにより、 <script>タグとして認識されず、入力値がそのまま出力される。

.. figure:: ./images_XSS/xss_screen_escape_result.png
   :alt: escape_result
   :width: 60%
   :align: center

   **Picture - Escape Result**

**出力結果**

.. code-block:: jsp

    <!-- omitted -->
    <tr>
        <td>Job</td>
        <td>&lt;script&gt;alert(&quot;XSS Attack&quot;)&lt;/script&gt;</td>
    </tr>
    <!-- omitted -->

.. tip:: **java.util.Date継承クラスのフォーマット**

    java.util.Date継承クラスをフォーマットして表示する場合は、JSTLの\ ``<fmt:formatDate>``\ を用いることを推奨する。
    以下に、設定例を示す。

        .. code-block:: jsp

            <fmt:formatDate value="${form.date}" pattern="yyyyMMdd" />

    valueの値に前述した \ ``f:h()``\ を使用して値を設定すると、Stringになってしまい、\ ``javax.el.ELException``\ がスローされるため、そのまま\ ``${form.date}``\ を使用している。
    しかし、yyyyMMddにフォーマットするため、XSSの心配はない。

.. tip::

        **java.lang.Number継承クラス、またはjava.lang.Numberにパースできる文字列**

        java.lang.Number継承クラスまたはjava.lang.Numberにパースできる文字列をフォーマットして表示する場合は、\ ``<fmt:formatNumber>``\ を用いることを推奨する。
        以下に、設定例を示す。

            .. code-block:: jsp

                <fmt:formatNumber value="${f:h(form.price)}" pattern="###,###" />

        上記は、Stringでも問題ないので、\ ``<fmt:formatNumber>``\ タグを使わなくなった場合に ``f:h()`` を付け忘れることを予防するため、\ ``f:h()``\ を明示的に使用している。

.. _xss_how_to_use_javascript_escaping:

JavaScript Escaping
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

XSSの脆弱性への対応としては、JavaScript特殊文字をエスケープすることが基本である。
ユーザーからの入力をもとに、JavaScriptの文字列リテラルを動的に生成する場合に、エスケープが必要となる。

エスケープが必要なJavaScriptの特殊文字の例と、エスケープ後の例は、以下のとおりである。

.. tabularcolumns:: |p{0.50\linewidth}|p{0.50\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 50 50

   * - | エスケープ前
     - | エスケープ後
   * - | ``'``
     - | ``\'``
   * - | ``"``
     - | ``\"``
   * - | ``\``
     - | ``\\``
   * - | ``/``
     - | ``\/``
   * - | ``<``
     - | ``\x3c``
   * - | ``>``
     - | ``\x3e``
   * - | ``0x0D(復帰)``
     - | ``\r``
   * - | ``0x0A(改行)``
     - | ``\n``

出力値をエスケープしない脆弱性のある例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

XSS問題が発生する例を、以下に示す。

本例は、あくまで参考例として載せているだけなので、以下のような実装は、決して行わないこと。

.. code-block:: html

  <html>
    <script  type="text/javascript">
        var aaa = '<script>${warnCode}<\/script>';
        document.write(aaa);
    </script>
  <html>

.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 80

   * - 属性名
     - 値
   * - | warnCode
     - | ``<script></script><script>alert('XSS Attack!');</script><\/script>``

上記例のように、ユーザーの入力を導出元としてコードを出力するなど、JavaScriptの要素を動的に生成する場合、意図せず文字列リテラルが閉じられ、XSSの脆弱性が生じる。

.. figure:: ./images_XSS/javascript_xss_screen_no_escape_result.png
   :alt: javascript_xss_screen_no_escape_result
   :width: 30%
   :align: center

   **Picture - No Escape Result**

**出力結果**

.. code-block:: html

    <script type="text/javascript">
        var aaa = '<script><\/script><script>alert('XSS Attack!');<\/script><\/script>';
        document.write(aaa);
    </script>

.. tip::

    業務要件上必要でない限り、JavaScriptの要素をユーザーからの入力値に依存して動的に生成する仕様は、任意のスクリプトが埋め込まれてしまう可能性があるため、別の方式を検討する、または、極力避けるべきである。

.. _xss_how_to_use_js_function_example:

出力値をf:js()関数でエスケープする例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

XSSを防ぐために、ユーザーの入力値、が設定される値にEL式の関数、\ ``f:js()``\ の使用を推奨する。

使用例を、下記に示す。

.. code-block:: html

    <script type="text/javascript">
      var message = '<script>${f:js(message)}<\/script>';  // (1)
      <!-- omitted -->
    </script>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | EL式の\ ``f:js()``\ を使用することにより、エスケープして変数に設定している。

**出力結果**

.. code-block:: html

    <script  type="text/javascript">
        var aaa = '<script>\x3c\/script\x3e\x3cscript\x3ealert(\'XSS Attack!\');\x3c\/script\x3e<\/script>';
        document.write(aaa);
    </script>

.. _xss_how_to_use_event_handler_escaping:

Event handler Escaping
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

javascript のイベントハンドラの値をエスケープする場合、\ ``f:h()``\ や、\ ``f:js()``\ を使用するのではなく、\ ``f:hjs()``\ を使用すること。\ ``${f:h(f:js())}``\ と同義である。

理由としては、 \ ``<input type="submit" onclick="callback('xxxx');">``\ のようなイベントハンドラの値に\ ``"');alert("XSS Attack");// "``\ を指定された場合、別のスクリプトを挿入できてしまうため、文字参照形式にエスケープ後、HTMLエスケープを行う必要がある。

出力値をエスケープしない脆弱性のある例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
XSS問題が発生する例を、以下に示す。

.. code-block:: jsp

    <input type="text" onmouseover="alert('output is ${warnCode}') . ">

.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 80

   * - 属性名
     - 値
   * - | warnCode
     - | ``'); alert('XSS Attack!'); //``
       | 上記の値が設定されてしまうことで、意図せず文字列リテラルが閉じられ、XSSの脆弱性が生じる。

マウスオーバ時、XSSのダイアログボックスが表示されてしまう。

.. figure:: ./images_XSS/eventhandler_xss_screen_no_escape_result.png
   :alt: eventhandler_xss_screen_no_escape_result
   :width: 50%
   :align: center

   **Picture - No Escape Result**


**出力結果**

.. code-block:: jsp

    <!-- omitted -->
    <input type="text" onmouseover="alert('output is'); alert('XSS Attack!'); // .') ">
    <!-- omitted -->

.. _xss_how_to_use_hjs_function_example:

出力値をf:hjs()関数でエスケープする例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

使用例を、下記に示す。

.. code-block:: jsp

    <input type="text" onmouseover="alert('output is ${f:hjs(warnCode)}') . ">  // (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | EL式の\ ``f:hjs()``\ を使用することにより、エスケープして引数としている。

マウスオーバ時、XSSのダイアログは出力されない。

.. figure:: ./images_XSS/eventhandler_xss_screen_escape_result.png
   :alt: eventhandler_xss_screen_escape_result
   :width: 50%
   :align: center

   **Picture - Escape Result**

**出力結果**

.. code-block:: jsp

    <!-- omitted -->
    <input type="text" onmouseover="alert('output is \&#39;); alert(\&#39;XSS Attack!\&#39;);\&quot; \/\/ .') ">
    <!-- omitted -->

.. raw:: latex

   \newpage

