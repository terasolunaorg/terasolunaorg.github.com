CSRF対策
================================================================================

.. only:: html

 .. contents:: 目次
    :local:

Overview
--------------------------------------------------------------------------------

| Cross site request forgeries(以下、CSRFと略す）とは、Webサイトにスクリプトや自動転送(HTTPリダイレクト)を実装することにより、
| ユーザが、ログイン済みの別のWebサイト上で、意図しない何らかの操作を行わせる攻撃手法のことである。

| サーバ側でCSRFを防ぐには、以下の方法が知られている。

* 秘密情報(トークン)の埋め込み
* パスワードの再入力
* Referのチェック

| OWASPでは、トークンパターンを使用する方法が\ `推奨されている。 <https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)_Prevention_Cheat_Sheet#General_Recommendation:_Synchronizer_Token_Pattern>`_\

.. figure:: ./images/csrf_check_other_site.png
   :alt: csrf check other site
   :width: 60%
   :align: center

   **Picture - csrf check other site**

.. note::

  **OWASPとは**

  Open Web Application Security Projectの略称であり、信頼できるアプリケーションや、セキュリティに関する
  効果的なアプローチなどを検証、提唱する、国際的な非営利団体である。

    https://www.owasp.org/index.php/Main_Page

| CSRFを回避する方法は、前述したように複数あるが、固定トークンを使用するライブラリを、Spring Securityが提供している。
| セッション毎に1つの固定トークンを用い、すべてのリクエストについて、同じ値を使用している。

| デフォルトではHTTPメソッドが、GET,HEAD,TRACE,OPTIONS以外の場合、
| リクエストに含まれるCSRFトークンをチェックし、値が一致しない場合は、エラー(HTTP Status:403[Forbidden])とする。

.. figure:: ./images/csrf_check_kind.png
   :alt: csrf check other kind
   :width: 50%
   :align: center

**Picture - csrf check other kind**

.. tip::

  CSRFトークンチェックは、別サイトからの不正な更新リクエストをチェックし、エラーとするものである。
  ユーザに順序性（一連の業務フロー）を守らせ、チェックするためには、\ :ref:`double-submit_transactiontokencheck`\ を参照されたい。

|

How to use
--------------------------------------------------------------------------------

Spring Securityの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Spring SecurityのCSRF機能を使用するための設定を説明する。
\ :ref:`Spring Security の How to use <howtouse_springsecurity>`\ で設定したweb.xmlを前提とする。

.. _csrf_spring-security-setting:

spring-security.xmlの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

追加で設定が必要な箇所を、ハイライトしている。


.. code-block:: xml
   :emphasize-lines: 3-4,8-

    <sec:http auto-config="true" use-expressions="true" >
        <!-- omitted -->
        <sec:csrf />  <!-- (1) -->
        <sec:access-denied-handler ref="accessDeniedHandler"/>  <!-- (2) -->
        <!-- omitted -->
    </sec:http>

    <bean id="accessDeniedHandler"
        class="org.springframework.security.web.access.DelegatingAccessDeniedHandler">  <!-- (3) -->
        <constructor-arg index="0">  <!-- (4) -->
            <map>
                <entry
                    key="org.springframework.security.web.csrf.InvalidCsrfTokenException">  <!-- (5) -->
                    <bean
                        class="org.springframework.security.web.access.AccessDeniedHandlerImpl">  <!-- (5) -->
                        <property name="errorPage"
                            value="/WEB-INF/views/common/error/invalidCsrfTokenError.jsp" />  <!-- (5) -->
                    </bean>
                </entry>
                <entry
                    key="org.springframework.security.web.csrf.MissingCsrfTokenException">  <!-- (6) -->
                    <bean
                        class="org.springframework.security.web.access.AccessDeniedHandlerImpl">  <!-- (6) -->
                        <property name="errorPage"
                            value="/WEB-INF/views/common/error/missingCsrfTokenError.jsp" />  <!-- (6) -->
                    </bean>
                </entry>
            </map>
        </constructor-arg>
        <constructor-arg index="1">  <!-- (7) -->
            <bean
                class="org.springframework.security.web.access.AccessDeniedHandlerImpl">  <!-- (8) -->
                <property name="errorPage"
                    value="/WEB-INF/views/common/error/accessDeniedError.jsp" />  <!-- (8) -->
            </bean>
        </constructor-arg>
    </bean>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``<sec:http>``\ 要素に\ ``<sec:csrf>``\ 要素を定義することで、Spring Security のCSRFトークンチェック機能を利用できるようになる。
       | デフォルトでチェックされるHTTPメソッドについては、\ :ref:`こちら<csrf_default-add-token-method>`\ を参照されたい。
       | 詳細については、\ `Spring Securityのレファレンスドキュメント <http://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/reference/htmlsingle/#csrf-configure>`_\ を参照されたい。
   * - | (2)
     - | \ ``AccessDeniedException``\ を継承したExceptionが発生した場合、Exceptionの種類毎に表示するviewを切り替えるためにHandlerを定義する。
       | 全て同じ画面で良い場合は ``error-page`` 属性に遷移先のjspを指定することで可能となる。
       | Spring Securityの機能でハンドリングしない場合は、\ :ref:`こちら<csrf_403-webxml-setting>`\ を参照されたい。
   * - | (3)
     - | エラーページを切り替えるためにSpring Securityで用意されているHandlerのclassに \ ``org.springframework.security.web.access.DelegatingAccessDeniedHandler``\ を指定する。
   * - | (4)
     - | コンストラクタの第1引数でデフォルト以外のException（\ ``AccessDeniedException``\ を継承したException）の種類毎に表示を変更する画面をMap形式で設定する。
   * - | (5)
     - | keyに \ ``AccessDeniedException``\ を継承したException を指定する。
       | 実装クラスとして、Spring Securityで用意されている \ ``org.springframework.security.web.access.AccessDeniedHandlerImpl`` を指定する。
       | propertyのnameにerrorPageを指定し、valueに表示するviewを指定する。
   * - | (6)
     - | (5)とExceptionの種類が違う場合に表示の変更を定義する。
   * - | (7)
     - | コンストラクタの第2引数でデフォルト（\ ``AccessDeniedException``\ とコンストラクタの第1引数で指定していない\ ``AccessDeniedException``\を継承したException）の場合のviewを指定する。
   * - | (8)
     - | 実装クラスとして、Spring Securityで用意されている \ ``org.springframework.security.web.access.AccessDeniedHandlerImpl`` を指定する。
       | propertyのnameにerrorPageを指定し、valueに表示するviewを指定する。

|

.. tabularcolumns:: |p{0.40\linewidth}|p{0.60\linewidth}|
.. list-table:: \ ``AccessDeniedException``\ を継承したCSRF対策により発生するExceptionの種類
   :header-rows: 1
   :widths: 40 60

   * - Exception
     - 発生理由
   * - | org.springframework.security.web.csrf.
       | InvalidCsrfTokenException
     - | クライアントからリクエストしたCSRFトークンとサーバで保持しているCSRFトークンが一致しない場合に発生する。
   * - | org.springframework.security.web.csrf.
       | MissingCsrfTokenException
     - | CSRFトークンがサーバに存在しない場合に発生する。
       | デフォルトの設定ではCSRFトークンをHTTPセッションに保持するため、CSRFトークンが存在しないということはHTTPセッションが破棄された(セッションタイムアウトが発生した)ことを意味する。
       |
       | \ ``<sec:csrf>``\ 要素の \ ``token-repository-ref``\ 属性でCSRFトークンの保存先をキャッシュサーバやDBなどに変更した場合は、CSRFトークンを保存先から削除した場合に\ ``MissingCsrfTokenException``\ が発生する。
       | これは、トークンの保存先をHTTPセッションにしていない場合は、本機能を使ってセッションタイムアウトの検知が出来ない事を意味している。

.. note::

    CSRFトークンの保存先としてHTTPセッションを使用する場合は、
    CSRFトークンのチェック対象のリクエストに対してセッションタイムアウトを検出することができる。

    セッションタイムアウト検知後の動作は、\ ``<session-management>``\ 要素の\ ``invalid-session-url``\ 属性の指定によって異なる。

    * \ ``invalid-session-url``\ 属性の指定がある場合は、セッションを生成した後に\ ``invalid-session-url``\ に指定したパスへリダイレクトされる。
    * \ ``invalid-session-url``\ 属性の指定がない場合は、\ ``<access-denied-handler>``\ 要素に指定した\ ``org.springframework.security.web.access.AccessDeniedHandler``\ の定義に従ったハンドリングが行われる。

    CSRFトークンのチェック対象外のリクエストに対してセッションタイムアウトを検出する必要がある場合は、
    \ ``<session-management>``\ 要素の\ ``invalid-session-url``\ 属性を指定して検出すればよい。
    詳細は、「:ref:`authentication_session-timeout`」を参照されたい。

|

.. _csrf_403-webxml-setting:

.. note::

  **<sec:access-denied-handler>の設定を省略した場合のエラーハンドリングについて**

  web.xmlに以下の設定を行うことで、任意のページに遷移させることができる。

  **web.xml**

    .. code-block:: xml

        <error-page>
            <error-code>403</error-code>  <!-- (1) -->
            <location>/WEB-INF/views/common/error/accessDeniedError.jsp</location>  <!-- (2) -->
        </error-page>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | error-code要素に、ステータスコード403を設定する。
       * - | (2)
         - | location要素に、遷移先のパスを設定する。

.. _csrf_change-httpstatus403:

.. note::

  **ステータスコード403以外を返却したい場合**

  リクエストに含まれるCSRFトークンが一致しない場合、ステータスコード403以外を返却したい場合は、\ ``org.springframework.security.web.access.AccessDeniedHandler``\ インタフェースを
  実装した、独自のAccessDeniedHandlerを作成する必要がある。

.. _csrf_spring-mvc-setting:

spring-mvc.xmlの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
CSRFトークン用の\ ``RequestDataValueProcessor``\ 実装クラスを利用し、Springのタグライブラリの\ ``<form:form>``\ タグを使うことで、自動的にCSRFトークンを、hiddenに埋め込むことができる。

.. code-block:: xml
   :emphasize-lines: 1-2,5-6

    <bean id="requestDataValueProcessor"
        class="org.terasoluna.gfw.web.mvc.support.CompositeRequestDataValueProcessor"> <!-- (1)  -->
        <constructor-arg>
            <util:list>
                <bean
                    class="org.springframework.security.web.servlet.support.csrf.CsrfRequestDataValueProcessor" /> <!-- (2)  -->
                <bean
                    class="org.terasoluna.gfw.web.token.transaction.TransactionTokenRequestDataValueProcessor" />
            </util:list>
        </constructor-arg>
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``org.terasoluna.gfw.web.mvc.support.RequestDataValueProcessor``\ を複数定義可能な
       | \ ``org.terasoluna.gfw.web.mvc.support.CompositeRequestDataValueProcessor``\ をbean定義する。
   * - | (2)
     - | コンストラクタの第1引数に、\ ``org.springframework.security.web.servlet.support.csrf.CsrfRequestDataValueProcessor``\ のbean定義を設定する。

.. note::

  CSRFトークンの生成及びチェックは \ ``<sec:csrf />``\ の設定で有効になる \ ``CsrfFilter``\ により行われるので、開発者はControllerで特にCSRF対策は意識しなくてよい。

.. _csrf_form-tag-token-send:

フォームによるCSRFトークンの送信
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

JSPで、フォームからCSRFトークンを送信するには

* \ ``<form:form>``\ タグを使用してCSRFトークンが埋め込まれた\ ``<input type="hidden">``\ タグを自動的に追加する
* \ ``<sec:csrfInput/>``\ タグを使用してCSRFトークンが埋め込まれた\ ``<input type="hidden">``\ タグを明示的に追加する

のどちらかを行う必要がある。

.. _csrf_formformtag-use:

CSRFトークンを自動で埋め込む方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ :ref:`spring-mvc.xmlの設定<csrf_spring-mvc-setting>`\ の通り、\ ``CsrfRequestDataValueProcessor``\ が定義されている場合、
\ ``<form:form>``\ タグを使うことで、CSRFトークンが埋め込まれた\ ``<input type="hidden">``\ タグが、自動的に追加される。

JSPで、CSRFトークンを意識する必要はない。

.. code-block:: jsp

    <form:form method="POST"
      action="${pageContext.request.contextPath}/csrfTokenCheckExample">
      <input type="submit" name="second" value="second" />
    </form:form>

以下のようなHTMLが、出力される。

.. code-block:: html

    <form action="/terasoluna/csrfTokenCheckExample" method="POST">
      <input type="submit" name="second" value="second" />
      <input type="hidden" name="_csrf" value="dea86ae8-58ea-4310-bde1-59805352dec7" /> <!-- (1) -->
    </form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | Spring Securityのデフォルト実装では、\ ``name``\ 属性に\ ``_csrf``\ が設定されている \ ``<input type="hidden">``\ タグが追加され、CSRFトークンが埋め込まれる。


CSRFトークンはログインのタイミングで生成される。

.. tip::

    Spring 4上で\ ``CsrfRequestDataValueProcessor``\ を使用すると、
    \ ``<form:form>``\ タグの\ ``method``\ 属性に指定した値がCSRFトークンチェック対象のHTTPメソッド(Spring Securityのデフォルト実装ではGET,HEAD,TRACE,OPTIONS以外のHTTPメソッド)と一致する場合に限り、
    CSRFトークンが埋め込まれた\ ``<input type="hidden">``\ タグが出力される。

    例えば、以下の例のように \ ``method``\ 属性にGETメソッドを指定した場合は、
    CSRFトークンが埋め込まれた\ ``<input type="hidden">``\ タグは出力されない。

        .. code-block:: jsp

            <form:form method="GET" modelAttribute="xxxForm" action="...">
                <%-- ... --%>
            </form:form>

    これは、\ `OWASP Top 10 <https://code.google.com/p/owasptop10/>`_\ で説明されている、

        The unique token can also be included in the URL itself, or a URL parameter. However, such placement runs a greater risk that the URL will be exposed to an attacker, thus compromising the secret token.

    に対応している事を意味しており、セキュアなWebアプリケーション構築の手助けとなる。

.. _csrf_formtag-use:

CSRFトークンを明示的に埋め込む方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``<form:form>``\ タグを使用しない場合は、明示的に、\ ``<sec:csrfInput/>``\ タグを追加する必要がある。

\ ``<sec:csrfInput/>``\ タグを使用すると、CSRFトークンが埋め込まれた\ ``<input type="hidden">``\ タグが出力される。

.. code-block:: jsp

    <form method="POST"
      action="${pageContext.request.contextPath}/csrfTokenCheckExample">
        <input type="submit" name="second" value="second" />
        <sec:csrfInput/>  <!-- (1) -->
    </form>

以下のようなHTMLが、出力される。

.. code-block:: html

    <form action="/terasoluna/csrfTokenCheckExample" method="POST">
      <input type="submit" name="second" value="second" />
      <input type="hidden" name="_csrf" value="dea86ae8-58ea-4310-bde1-59805352dec7"/>  <!-- (2) -->
    </form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | CSRFトークンが埋め込まれた\ ``<input type="hidden">``\ タグを出力するために、\ ``<sec:csrfInput/>``\ タグを指定する。
   * - | (2)
     - | Spring Securityのデフォルト実装では、\ ``name``\ 属性に\ ``_csrf``\ が設定されている \ ``<input type="hidden">``\ タグが追加され、CSRFトークンが埋め込まれる。

.. _csrf_default-add-token-method:

.. note::

  CSRFトークンチェック対象のリクエスト(デフォルトでは、HTTPメソッドが、GET, HEAD, TRACE, OPTIONS以外の場合)で、CSRFトークンがない、または
  サーバー上に保存されているトークン値と、送信されたトークン値が異なる場合は、\ ``AccessDeniedHandler``\ によりアクセス拒否処理が行われ、HttpStatusの403が返却される。
  \ :ref:`spring-security.xmlの設定 <csrf_spring-security-setting>`\ を記述している場合は、指定したエラーページに遷移する。


.. _csrf_ajax-token-setting:

AjaxによるCSRFトークンの送信
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| \ ``<sec:csrf />``\ の設定で有効になる \ ``CsrfFilter``\ は、前述のようにリクエストパラメータからCSRFトークンを取得するだけでなく、
| HTTPリクエストヘッダーからもCSRFトークンを取得する。
| Ajaxを利用する場合はHTTPヘッダーに、CSRFトークンを設定することを推奨する。JSON形式でリクエストを送る場合にも対応できるためである。

.. note::

  HTTPヘッダ、リクエストパラメータの両方からCSRFトークンが送信する場合は、HTTPヘッダの値が優先される。

| \ :doc:`../ArchitectureInDetail/Ajax`\ で使用した例を用いて、説明を行う。追加で設定が必要な箇所を、ハイライトしている。

**jspの実装例**

.. code-block:: jsp
   :emphasize-lines: 3-4

    <!-- omitted -->
    <head>
      <sec:csrfMetaTags />  <!-- (1) -->
      <!-- omitted -->
    </head>
    <!-- omitted -->

.. code-block:: jsp
   :emphasize-lines: 3-7

    <script type="text/javascript">
    var contextPath = "${pageContext.request.contextPath}";
    var token = $("meta[name='_csrf']").attr("content");  <!-- (2) -->
    var header = $("meta[name='_csrf_header']").attr("content");  <!-- (3) -->
    $(document).ajaxSend(function(e, xhr, options) {
        xhr.setRequestHeader(header, token);  <!-- (4) -->
    });

    $(function() {
        $('#calcButton').on('click', function() {
            var $form = $('#calcForm'),
                 $result = $('#result');
            $.ajax({
                url : contextPath + '/sample/calc',
                type : 'POST',
                data: $form.serialize(),
            }).done(function(data) {
                $result.html('add: ' + data.addResult + '<br>'
                             + 'subtract: ' + data.subtractResult + '<br>'
                             + 'multipy: ' + data.multipyResult + '<br>'
                             + 'divide: ' + data.divideResult + '<br>'); // (5)
            }).fail(function(data) {
                // error handling
                alert(data.statusText);
            });
        });
    });
    </script>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``<sec:csrfMetaTags />``\ タグを設定することにより、デフォルトでは、以下の\ ``meta``\ タグが出力される。

       * \ ``<meta name="_csrf_parameter" content="_csrf" />``\
       * \ ``<meta name="_csrf_header" content="X-CSRF-TOKEN" />``\
       * \ ``<meta name="_csrf" content="dea86ae8-58ea-4310-bde1-59805352dec7" />``\ (\ ``content``\ 属性の値はランダムなUUIDが設定される)
   * - | (2)
     - | \ ``<meta name="_csrf">``\ タグに設定されたCSRFトークンを取得する。
   * - | (3)
     - | \ ``<meta name="_csrf_header">``\ タグに設定されたCSRFヘッダ名を取得する。
   * - | (4)
     - | リクエストヘッダーに、\ ``<meta>``\ タグから取得したヘッダ名(デフォルト:X-CSRF-TOKEN)、CSRFトークンの値を設定する。
   * - | (5)
     - | この書き方はXSSの可能性があるので、実際にJavaScriptコードを書くときは気を付けること。
       | 今回の例では\ ``data.addResult``\ 、\ ``data.subtractResult``\ 、\ ``data.multipyResult``\ 、\ ``data.divideResult``\ の全てが数値型であるため、問題ない。

JSONでリクエストを送信する場合も、同様にHTTPヘッダを設定すればよい。

.. todo::

  \ :doc:`../ArchitectureInDetail/Ajax`\ 対応する例がなくなっているため、例を直す。

マルチパートリクエスト(ファイルアップロード)時の留意点
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| 一般的に、ファイルアップロードなどマルチパートリクエストを送る場合、formから送信される値を\ ``Filter``\ では取得できない。
| そのため、これまでの説明だけでは、マルチパートリクエスト時に\ ``CsrfFileter``\ がCSRFトークンを取得できず、不正なリクエストと見なされてしまう。

そのため、以下のどちらかの方法によって、対策する必要がある。

* \ ``org.springframework.web.multipart.support.MultipartFilter``\ を使用する
* クエリのパラメータでCSRFトークンを送信する

.. note::

    それぞれメリット・デメリットが存在するため、システム要件を考慮して、採用する対策方法を決めて頂きたい。

ファイルアップロードの詳細については、\ :doc:`FileUpload <../ArchitectureInDetail/FileUpload>`\ を参照されたい。


.. _csrf_use-multipart-filter:

MultipartFilterを使用する方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 通常、マルチパートリクエストの場合、formから送信された値は\ ``Filter``\ 内で取得できない。
| \ ``org.springframework.web.multipart.support.MultipartFilter``\ を使用することで、マルチパートリクエストでも、\ ``Filter``\ 内で、
| formから送信された値を取得することができる。


.. warning::

    \ ``MultipartFilter``\ を使用した場合、\ ``springSecurityFilterChain``\による認証・認可処理が行われる前にアップロード処理が行われるため、
    認証又は認可されていないユーザーからのアップロード(一時ファイル作成)を許容してしまう。


\ ``MultipartFilter``\ を使用するには、以下のように設定すればよい。

**web.xmlの設定例**

.. code-block:: xml

    <filter>
        <filter-name>MultipartFilter</filter-name>
        <filter-class>org.springframework.web.multipart.support.MultipartFilter</filter-class> <!-- (1) -->
    </filter>
    <filter>
        <filter-name>springSecurityFilterChain</filter-name> <!-- (2) -->
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>MultipartFilter</filter-name>
        <servlet-name>/*</servlet-name>
    </filter-mapping>
    <filter-mapping>
        <filter-name>springSecurityFilterChain</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``org.springframework.web.multipart.support.MultipartFilter``\ を 定義する。
   * - | (2)
     - | \ ``springSecurityFilterChain``\ より前に、\ ``MultipartFilter``\ を定義すること。

**JSPの実装例**

.. code-block:: jsp

    <form:form action="${pageContext.request.contextPath}/fileupload"
        method="post" modelAttribute="fileUploadForm" enctype="multipart/form-data">  <!-- (1) -->
        <table>
            <tr>
                <td width="65%"><form:input type="file" path="uploadFile" /></td>
            </tr>
            <tr>
                <td><input type="submit" value="Upload" /></td>
            </tr>
        </table>
    </form:form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | spring-mvc.xmlの設定の通り、\ ``CsrfRequestDataValueProcessor``\ が定義されている場合、
       | \ ``<form:form>``\ タグを使うことで、CSRFトークンが埋め込まれた\ ``<input type="hidden">``\ タグが自動的に追加されるため、
       | JSPの実装で、CSRFトークンを意識する必要はない。
       |
       | **<form> タグを使用する場合**
       | \ :ref:`csrf_formtag-use`\ でCSRFトークンを設定すること。


クエリパラメータでCSRFトークンを送る方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

認証又は認可されていないユーザーからのアップロード(一時ファイル作成)を防ぎたい場合は、
\ ``MultipartFilter``\ は使用せず、クエリパラメータでCSRFトークンを送る必要がある。

.. warning::

    この方法でCSRFトークンを送った場合、

    * ブラウザのアドレスバーにCSRFトークンが表示される
    * ブックマークした場合、ブックマークにCSRFトークンが記録される
    * WebサーバのアクセスログにCSRFトークンが記録される

    ため、\ ``MultipartFilter``\ を使用する方法と比べると、攻撃者にCSRFトークンを悪用されるリスクが高くなる。

    Spring Securityのデフォルト実装では、CSRFトークンの値としてランダムなUUIDを生成しているため、
    仮にCSRFトークンが漏洩してもセッションハイジャックされる事はないという点を補足しておく。

以下に、CSRFトークンをクエリパラメータとして送る実装例を示す。

**JSPの実装例**

.. code-block:: jsp

    <form:form action="${pageContext.request.contextPath}/fileupload?${f:h(_csrf.parameterName)}=${f:h(_csrf.token)}"
        method="post" modelAttribute="fileUploadForm" enctype="multipart/form-data"> <!-- (1) -->
        <table>
            <tr>
                <td width="65%"><form:input type="file" path="uploadFile" /></td>
            </tr>
            <tr>
                <td><input type="submit" value="Upload" /></td>
            </tr>
        </table>
    </form:form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``<form:form>``\ タグのaction属性に、以下のクエリを付与する必要がある。
       | \ ``?${f:h(_csrf.parameterName)}=${f:h(_csrf.token)}``\
       | \ ``<form>``\ タグを使用する場合も、同様の設定が必要である。

.. raw:: latex

   \newpage

