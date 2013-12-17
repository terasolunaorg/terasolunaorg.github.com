.. raw:: pdf

    PageBreak

CSRF対策
================================================================================

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

.. warning::

  CSRF対策機能は、Spring Security3.2から提供される機能であるが、共通ライブラリ(terasoluna-gfw-security-web)の1.0.0.RELEASE版が依存している
  Spring Securityのバージョンは、3.1.4.RELEASEである(共通ライブラリの1.0.0.RELEASE版リリース時には、Spring Securityの3.2.0.RELEASE版は未リリースであるため)。
  このため、terasoluna-gfw-security-webプロジェクト内に、1.0.0.RELEASE版リリース時のSprinng SecurityのCSRF対策機能に関する以下のクラスが同梱されている。

  * org.springframework.security.web.authentication.session.CompositeSessionAuthenticationStrategy
  * org.springframework.security.web.csrf.CsrfAuthenticationStrategy
  * org.springframework.security.web.csrf.CsrfFilter
  * org.springframework.security.web.csrf.CsrfLogoutHandler
  * org.springframework.security.web.csrf.CsrfToken
  * org.springframework.security.web.csrf.CsrfTokenRepository
  * org.springframework.security.web.csrf.DefaultCsrfToken
  * org.springframework.security.web.csrf.HttpSessionCsrfTokenRepository
  * org.springframework.security.web.csrf.InvalidCsrfTokenException
  * org.springframework.security.web.servlet.support.csrf.CsrfRequestDataValueProcessor

  共通ライブラリのバージョンアップのタイミングで、Spring Securityのバージョンアップし、上記のクラスは、terasoluna-gfw-security-webプロジェクトからは取り除かれる予定である。

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
   :emphasize-lines: 3-6,10-

    <sec:http auto-config="true" use-expressions="true" >
        <!-- omitted -->
        <sec:custom-filter ref="csrfFilter" before="LOGOUT_FILTER" />  <!-- (1) -->

        <sec:session-management
            session-authentication-strategy-ref="sessionAuthenticationStrategy" />  <!-- (2) -->
        <!-- omitted -->
    </sec:http>

    <bean id="csrfFilter" class="org.springframework.security.web.csrf.CsrfFilter">  <!-- (3) -->
        <constructor-arg index="0" ref="csrfTokenRepository" />  <!-- (4) -->
        <property name="accessDeniedHandler">
            <bean
                class="org.springframework.security.web.access.AccessDeniedHandlerImpl">  <!-- (5) -->
                <property name="errorPage"
                    value="/WEB-INF/views/common/error/csrfTokenError.jsp" />  <!-- (6) -->
            </bean>
        </property>
    </bean>

    <bean id="csrfTokenRepository"
        class="org.springframework.security.web.csrf.HttpSessionCsrfTokenRepository" />  <!-- (7) -->

    <bean id="sessionAuthenticationStrategy"
        class="org.springframework.security.web.authentication.session.CompositeSessionAuthenticationStrategy"> <!-- (8) -->
        <constructor-arg index="0">
            <list>
                <!-- omitted -->
                <bean
                    class="org.springframework.security.web.csrf.CsrfAuthenticationStrategy">  <!-- (9) -->
                    <constructor-arg index="0"
                        ref="csrfTokenRepository" />  <!-- (10) -->
                </bean>
            </list>
        </constructor-arg>
    </bean>

.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``<sec:custom-filter>``\ 要素を定義し、\ ``org.springframework.security.web.authentication.logout.LogoutFilter``\ の前に CSRFのFilter定義を行う。
   * - | (2)
     - | \ ``<sec:session-management>``\ 要素の、\ ``session-authentication-strategy-ref``\ 属性で、
       | \ ``org.springframework.security.web.authentication.session.SessionAuthenticationStrategy``\ を参照する。
   * - | (3)
     - | \ ``org.springframework.security.web.csrf.CsrfFilter``\ のbean定義を行う。
   * - | (4)
     - | コンストラクタの第1引数で、トークンの作成、保持を行う\ ``org.springframework.security.web.csrf.CsrfTokenRepository``\ を参照する。
   * - | (5)
     - | \ ``accessDeniedHandler``\ プロパティに\ ``org.springframework.security.web.access.AccessDeniedHandlerImpl``\ を bean定義する。
   * - | (6)
     - | \ ``AccessDeniedHandlerImpl``\ の\ ``errorPage``\ プロパティに、リクエストに含まれるCSRFトークンが、一致しない場合の遷移先パスを設定する。
       | 設定を省略した場合、リクエストに含まれるCSRFトークンが一致しない場合、ステータスコード403でクライアントに返却する。
   * - | (7)
     - | \ ``CsrfTokenRepository``\ の実装としてHTTPセッションにCSRFトークンを保存する、\ ``org.springframework.security.web.csrf.HttpSessionCsrfTokenRepository``\ クラスを定義する。
   * - | (8)
     - | \ ``SessionAuthenticationStrategy``\ の実装として、複数の\ ``SessionAuthenticationStrategy``\ を使用できる\ ``org.springframework.security.web.authentication.session.CompositeSessionAuthenticationStrategy``\ 
   * - | (9)
     - | \ ``CompositeSessionAuthenticationStrategy``\ に、\ ``org.springframework.security.web.csrf.CsrfAuthenticationStrategy``\ を追加する。
   * - | (10)
     - | \ ``CompositeSessionAuthenticationStrategy``\ コンストラクタの第1引数で、\ ``CsrfTokenRepository``\ を参照する。

.. note::

  **AccessDeniedHandlerImplのerrorPageプロパティを省略した場合の、エラーハンドリングについて**

  web.xmlに、以下の設定を行うことで、任意のページに遷移させることができる。

  **web.xml**

    .. code-block:: xml

        <error-page>
            <error-code>403</error-code>  <!-- (1) -->
            <location>/WEB-INF/views/common/error/csrf-error.jsp</location>  <!-- (2) -->
        </error-page>

    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | error-code要素に、ステータスコード403を設定する。
       * - | (2)
         - | location要素に、遷移先のパスを設定する。

.. note::

  **ステータスコード403以外を返却したい場合**

  リクエストに含まれるCSRFトークンが一致しない場合、ステータスコード403以外を返却したい場合は、\ ``org.springframework.security.web.access.AccessDeniedHandler``\ インタフェースを
  実装した、独自のAccessDeniedHandlerを作成する必要がある。
  詳細は、\ `Spring Securityのレファレンスドキュメント <http://docs.spring.io/spring-security/site/docs/3.2.0.RELEASE/reference/html/csrf.html>`_\ を参照されたい。

.. todo::

    **Spring Security のバージョンが、 3.2.0 以降の場合の設定**

    Spring Security 3.2を使用する場合、\ ``<sec:http>``\ 要素に\ ``<sec:csrf />``\ 要素を設定することで、
    前述した設定を省略することができる。
    
    \ `Spring Securityのレファレンスドキュメント <http://docs.spring.io/spring-security/site/docs/3.2.0.RELEASE/reference/htmlsingle/#csrf-configure>`_\ を参照されたい。

.. _csrf_spring-mvc-setting:

spring-mvc.xmlの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
CSRFトークン用の\ ``RequestDataValueProcessor``\ 実装クラスを利用し、Springのタグライブラリの\ ``<form:form>``\ タグを使うことで、自動的にCSRFトークンを、hiddenに埋め込むことができる。

.. code-block:: xml
   :emphasize-lines: 5-7

    <bean id="requestDataValueProcessor"
        class="org.terasoluna.gfw.web.mvc.support.CompositeRequestDataValueProcessor"> <!-- (1)  -->
        <constructor-arg>
            <util:list>
                <bean
                    class="org.springframework.security.web.servlet.support.csrf.CsrfRequestDataValueProcessor"
                    factory-method="create" /> <!-- (2)  -->
                <bean
                    class="org.terasoluna.gfw.web.token.transaction.TransactionTokenRequestDataValueProcessor" />
            </util:list>
        </constructor-arg>
    </bean>

.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``org.terasoluna.gfw.web.mvc.support.RequestDataValueProcessor``\ を複数定義可能な、
       | \ ``org.terasoluna.gfw.web.mvc.support.CompositeRequestDataValueProcessor``\ をbean定義する。
   * - | (2)
     - | コンストラクタの第1引数に、\ ``org.springframework.security.web.servlet.support.csrf.CsrfRequestDataValueProcessor``\ のbean定義を設定する。
       | factory-methodに、createメソッドを指定する。

.. note::

  CSRFトークンの生成、チェックは、\ ``CsrfFilter``\ が行うため、Controllerでは特に、CSRF対策は意識しなくてよい。

フォームによるCSRFトークンの送信
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

JSPで、フォームからCSRFトークンを送信するには

* \ ``<form:form>``\ タグを使用してCSRFトークンが埋め込まれた\ ``<input type="hidden">``\ タグを自動的に追加する
* \ ``<input type="hidden">``\ タグを作成し、明示的にCSRFトークンを埋め込む

のどちらかを行う必要がある。

.. _csrf_formformtag-use:

CSRFトークンを自動で埋め込む方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ :ref:`spring-mvc.xmlの設定<csrf_spring-mvc-setting>`\ の通り、\ ``CsrfRequestDataValueProcessor``\ が定義されている場合、
\ ``<form:form>``\ タグを使うことで、CSRFトークンの埋め込まれた\ ``<input type="hidden">``\ タグが、自動的に追加される。

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
      <input type="hidden" name="_csrf" value="dea86ae8-58ea-4310-bde1-59805352dec7" />
    </form>

自動で、\ ``_name``\ 属性が\ ``_csrf``\ である、\ ``<input type="hidden">``\ タグが追加され、CSRFトークンが埋め込まれているとわかる。

CSRFトークンはログインのタイミングで生成される。

.. warning::

  \ ``CsrfRequestDataValueProcessor``\ の設定をしている場合、すべてのリクエストにCSRFトークンを埋め込むため、\ ``<form:form method="GET" ...>...</form:form>``\ のように、
  \ ``<form:form>``\ でGETを指定してフォームを送信した場合、ブラウザのアドレスバーのURLに、CSRFトークンが含まれ、ブックマークバーやアクセスログに残る。
  
  これを回避するためには、

    .. code-block:: jsp
    
      <form:form method="GET" modelAttribute="xxxForm" action="...">
      ...
      </form:form>
  
  と書く代わりに、
  
    .. code-block:: jsp
    
      <form method="GET" action="...">
        <spring:nestedPath path="xxxForm">
        ...
        </spring:nestedPath>
      </form>`

  と記述すればよい。

  \ `OWASP Top 10 <https://code.google.com/p/owasptop10/>`_\ では
  
      The unique token can also be included in the URL itself, or a URL parameter. However, such placement runs a greater risk that the URL will be exposed to an attacker, thus compromising the secret token.
  
  と説明されており、必須ではないが対応することが推奨される。
  
  Spring Securityのデフォルト実装は、CSRFトークンをUUIDとして生成し、Session IDは使用しないため、CSRFトークンが漏洩してもSession IDは漏洩することはない。またログインの度にCSRFトークンは変更される。
  また、将来的にはSpring 4でこの問題は解消される(\ ``<form:form method="GET">``\ を使用してもCSRFトークンはURLに現れない)。
.. _csrf_formtag-use:

CSRFトークンを明示的に埋め込む方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``<form:form>``\ タグを使用しない場合は、明示的に、\ ``<input type="hidden">``\ タグを追加する必要がある。

\ ``CsrfFilter``\ により、\ ``org.springframework.security.web.csrf.CsrfToken``\ オブジェクトが、リクエストスコープの
\ ``_csrf``\ 属性に設定されるため、jspでは、以下のように設定すればよい

.. code-block:: jsp

    <form method="POST"
      action="${pageContext.request.contextPath}/csrfTokenCheckExample">
        <input type="submit" name="second" value="second" />
        <input type="hidden" name="${f:h(_csrf.parameterName)}" value="${f:h(_csrf.token)}"/>  <!-- (1) -->
    </form>

.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``_csrf.parameterName``\ でリクエストパラメータ名を、\ ``_csrf.token``\ で、CSRFトークンを設定する。

以下のようなHTMLが、出力される。


.. code-block:: html

    <form action="/terasoluna/csrfTokenCheckExample" method="POST">
      <input type="submit" name="second" value="second" />
      <input type="hidden" name="_csrf" value="dea86ae8-58ea-4310-bde1-59805352dec7"/>  <!-- (2) -->
    </form>

.. note::

  CSRFトークンチェック対象のリクエスト(デフォルトでは、HTTPメソッドが、GET, HEAD, TRACE, OPTIONS以外の場合)で、CSRFトークンがない、または
  サーバー上に保存されているトークン値と、送信されたトークン値が異なる場合は、\ ``AccessDeniedHandler``\ によりアクセス拒否処理が行われる。
  デフォルトでは403エラーとなり、\ ``AccessDeniedHandlerImpl``\ の\ ``errorPage``\ プロパティで指定したエラーページに遷移する。
  詳細は、\ :ref:`spring-security.xmlの設定 <csrf_spring-security-setting>`\ を参照されたい。


AjaxによるCSRFトークンの送信
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| ``CsrfFilter`` は、前述のようにリクエストパラメータからCSRFトークンを取得するだけでなく、
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
      <meta name="_csrf" content="${f:h(_csrf.token)}"/>  <!-- (1) -->
      <meta name="_csrf_header" content="${f:h(_csrf.headerName)}"/>  <!-- (2) -->
      <!-- omitted -->
    </head>
    <!-- omitted -->

.. code-block:: jsp
   :emphasize-lines: 3-7

    <script type="text/javascript">
    var contextPath = "${pageContext.request.contextPath}";
    var token = $("meta[name='_csrf']").attr("content");  <!-- (3) -->
    var header = $("meta[name='_csrf_header']").attr("content");  <!-- (4) -->
    $(document).ajaxSend(function(e, xhr, options) {
        xhr.setRequestHeader(header, token);  <!-- (5) -->
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
                             + 'divide: ' + data.divideResult + '<br>'); // (6)
            }).fail(function(data) {
                // error handling
                alert(data.statusText);
            });
        });
    });
    </script>

.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``<meta>``\ タグに、\ ``${f:h(_csrf.token)}``\ で取得したCSRFトークンを設定する。
   * - | (2)
     - | \ ``<meta>``\ タグに、\ ``${f:h(_csrf.headerName)}``\ で取得したヘッダ名を設定する。
   * - | (3)
     - | \ ``<meta>``\ タグに、設定したCSRFトークンを取得する。
   * - | (4)
     - | \ ``<meta>``\ タグに、設定したCSRFヘッダ名を取得する。
   * - | (5)
     - | リクエストヘッダーに、\ ``<meta>``\ タグで設定したヘッダ名(デフォルト:X-CSRF-TOKEN)、CSRFトークンの値を設定する。
   * - | (6)
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

MultipartFilterを使用する方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 通常、マルチパートリクエストの場合、formから送信された値は\ ``Filter``\ 内で取得できない。
| \ ``org.springframework.web.multipart.support.MultipartFilter``\ を使用することで、マルチパートリクエストでも、\ ``Filter``\ 内で、
| formから送信された値を取得することができる。

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

.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``<form:form>``\ タグのaction属性に、以下のクエリを付与する必要がある。
       | \ ``?${f:h(_csrf.parameterName)}=${f:h(_csrf.token)}``\
       | \ ``<form>``\ タグを使用する場合も、同様の設定が必要である。

| ファイルアップロードの詳細については、\ :doc:`FileUpload <../ArchitectureInDetail/FileUpload>`\ を参照されたい。

.. warning::

  この方法も前述したCSRFがURLに現れるという問題がある。
  \ ``MultipartFilter``\ を使用する場合は\ ``springSecurityFilterChain``\ による処理がアップロードの後になるため、
  認証されていないユーザーのアップロード(一時ファイル作成)を許容してしまう。これを防ぐ必要がある場合に、クエリパラメータでCSRFトークンを送る必要がある。
  
  
  
