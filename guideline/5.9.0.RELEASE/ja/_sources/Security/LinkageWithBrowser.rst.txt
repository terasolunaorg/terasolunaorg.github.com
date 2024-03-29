.. _SpringSecurityLinkageWithBrowser:

ブラウザのセキュリティ対策機能との連携
================================================================================

.. only:: html

.. contents:: 目次
   :local:

|

Overview
--------------------------------------------------------------------------------

本節では、ブラウザが提供しているセキュリティ対策機能との連携方法について説明する。

| 主要なWebブラウザは、ブラウザが提供する機能が悪用されないようにするために、いくつかのセキュリティ対策機能を提供している。
| ブラウザが提供するセキュリティ対策機能の一部は、サーバ側でHTTPのレスポンスヘッダを出力することで動作を制御することができる。

Spring Securityは、セキュリティ関連のレスポンスヘッダを出力する機能を用意することで、Webアプリケーションのセキュリティを強化する仕組みを提供している。

.. note:: \ **セキュリティリスク**\

  セキュリティ関連のレスポンスヘッダを出力しても、セキュリティへのリスクが100%なくなるわけではない。

  あくまで、セキュリティリスクを減らすためのサポート機能と考えておくこと。

  なお、セキュリティヘッダのサポート状況はブラウザによってことなる。

.. note:: \ **HTTPヘッダの上書き**\

  後述の設定を行ったとしても、アプリケーションにより、HTTPヘッダが上書きされる可能性は存在する。

|

Springがサポートしているセキュリティヘッダ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityがサポートしているレスポンスヘッダは以下である。

* Cache-Control (Pragma, Expires)
* X-Frame-Options
* X-Content-Type-Options
* Strict-Transport-Security
* Content-Security-Policy(Content-Security-Policy-Report-Only) 
* Referrer-Policy
* Permissions-Policy
* Clear-Site-Data

また、以下のセキュリティヘッダもサポートしているが、OWASPで非推奨とされている。

* X-XSS-Protection
* Public-Key-Pins(Public-Key-Pins-Report-Only)
* Feature-Policy

詳しくは\ `Response Headers <https://owasp.org/www-project-secure-headers/#div-headers>`_\ を参照されたい。

.. tip:: \ **ブラウザのサポート状況**\

  これらのヘッダに対する処理は、一部のブラウザではサポートされていない。
  
  ブラウザの公式サイトまたは\ `Browser Support <https://owasp.org/www-project-secure-headers/#div-browsersupport>`_\ を参照されたい。

|

Cache-Control
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Cache-Controlヘッダは、コンテンツのキャッシュ方法を指示するためのヘッダである。
| 保護されたコンテンツがブラウザにキャッシュされないようにすることで、権限のないユーザーが保護されたコンテンツを閲覧できてしまうリスクを減らすことができる。

コンテンツがキャッシュされないようにするためには、以下のようなヘッダを出力する。

* レスポンスヘッダの出力例\ **(Spring Securityのデフォルト出力)**\

.. code-block:: text

  Cache-Control: no-cache, no-store, max-age=0, must-revalidate
  Pragma: no-cache
  Expires: 0

.. note:: \ **Cache-Controlヘッダの上書き**\

  Spring MVCのControllerクラスが \ ``@SessionAttributes`` \ のフォームクラスを定義している、もしくは、リクエストハンドラで \ ``@SessionAttributes`` \ 属性のModelを使用している場合は、 Cache-Controlヘッダが上書きされる。

.. note:: \ **HTTP1.0互換のブラウザ**\

  Spring SecurityはHTTP1.0互換のブラウザもサポートするために、PragmaヘッダとExpiresヘッダも出力する。

|

X-Frame-Options
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| X-Frame-Optionsヘッダは、フレーム(\ ``<frame>``\ または\ ``<iframe>``\ 要素) 内でのコンテンツの表示を許可するか否かを指示するためのヘッダである。
| フレーム内でコンテンツが表示されないようすることで、クリックジャッキングと呼ばれる攻撃手法を使って機密情報を盗みとられるリスクをなくすことができる。

フレーム内での表示を拒否するためには、以下のようなヘッダを出力する。

* レスポンスヘッダの出力例\ **(Spring Securityのデフォルト出力)**\

.. code-block:: text

  X-Frame-Options: DENY

出力例以外の設定については\ `Response Headers <https://owasp.org/www-project-secure-headers/#div-headers>`_\ のX-Frame-Optionsを参照されたい。

|

.. _LinkageWithBrowserXContentTypeOtions:

X-Content-Type-Options
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| X-Content-Type-Optionsヘッダは、コンテンツの種類の決定方法を指示するためのヘッダである。
| 一部のブラウザでは、Content-Typeヘッダの値を無視してコンテンツの内容をみて決定する。
| コンテンツの種類の決定する際にコンテンツの内容を見ないようにすることで、クロスサイトスクリプティングを使った攻撃を受けるリスクを減らすことができる。

コンテンツの種類の決定する際にコンテンツの内容を見ないようにするためには、以下のヘッダを出力する。

* レスポンスヘッダの出力例\ **(Spring Securityのデフォルト出力)**\

.. code-block:: text

  X-Content-Type-Options: nosniff

|

Strict-Transport-Security
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Strict-Transport-Securityヘッダは、HTTPSを使ってアクセスした後にHTTPを使ってアクセスしようとした際に、HTTPSに置き換えてからアクセスすることを指示するためヘッダである。
| HTTPSでアクセスした後にHTTPが使われないようにすることで、中間者攻撃と呼ばれる攻撃手法を使って悪意のあるサイトに誘導されるリスクを減らすことができる。

HTTPSでアクセスした後にHTTPが使われないようにするためには、以下のようなヘッダを出力する。

* レスポンスヘッダの出力例\ **(Spring Securityのデフォルト出力)**\

.. code-block:: text

  Strict-Transport-Security: max-age=31536000 ; includeSubDomains

出力例以外の設定については\ `Response Headers <https://owasp.org/www-project-secure-headers/#div-headers>`_\ のStrict-Transport-Securityを参照されたい。

.. note:: \ **Strict-Transport-Security**\

  Spring Securityのデフォルト実装では、Strict-Transport-Securityヘッダは、アプリケーションサーバに対してHTTPSを使ってアクセスがあった場合のみ出力される。

  なお、Strict-Transport-Securityヘッダ値は、オプションを指定することで変更することができる。

.. note:: \ **HTTP Strict Transport Security (HSTS) preload list**\

  Strict-Transport-Securityヘッダーを設定していても、一度HTTPSアクセスが行われるまでの間や有効期限切れ後のアクセスでは中間者攻撃を受けるリスクがある。

  Googleはこのリスクを回避出来るようにHSTS preload listを運営している。このリストにドメインを登録すると、ブラウザからのアクセスで自動的にHTTPSが使用される。

  \ `主要なブラウザ <https://caniuse.com/stricttransportsecurity>`__\ は全て、HSTS preload listに対応している。

  HSTS preload listへのドメインの登録方法は\ `HSTS Preload List Submission <https://hstspreload.org/>`_\ を参照されたい。

  Spring SecurityではHSTS preload listへの登録に必要となるpreloadディレクティブをサポートしており、オプションを指定することで出力することが出来る。

|

.. _LinkageWithBrowserContentSecurityPolicy:

Content-Security-Policy
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Content-Security-Policyヘッダは、ブラウザに読み込みを許可するコンテンツを指示するためのヘッダである。
| ブラウザはContent-Security-Policyヘッダに指定したホワイトリストのコンテンツのみを読み込むため、悪意のあるコンテンツを読み込むことで実行される攻撃（クロスサイトスクリプティング攻撃など）を受けるリスクを減らすことができる。

Content-Security-Policyヘッダを送信しない場合、ブラウザは標準の同一オリジンポリシーを適用する。

コンテンツの取得元を同一オリジンのみに制限するためには、以下のようなヘッダを出力する。

* レスポンスヘッダの出力例

.. code-block:: text

  Content-Security-Policy: default-src 'self'

出力例以外の設定については\ `Response Headers <https://owasp.org/www-project-secure-headers/#div-headers>`_\ のContent-Security-Policyを参照されたい。

.. note:: \ **ポリシー違反時のレポート送信について**\

  ポリシー違反時にレポートを送信したい場合、report-uriディレクティブに報告先のURIを指定する。

  同一オリジンポリシー違反があった場合にコンテンツをブロックして\ ``/csp_report``\ にレポートを送信するためには、以下のようなヘッダを出力する。

  * レスポンスヘッダの出力例

    .. code-block:: text

      Content-Security-Policy: default-src 'self'; report-uri /csp_report;

  また、ポリシー違反があった際に、コンテンツのブロックを行わずレポートの送信のみを行いたい場合はContent-Security-Policy-Report-Onlyヘッダを使用する。

  Content-Security-Policy-Report-Onlyヘッダを使用してレポートを収集しながら段階的にポリシーとコンテンツを修正することで、既にサービス提供しているサイトに対してポリシーを適用した場合に正常に動作しなくなるリスクを減らすことが出来る。

  同一オリジンポリシー違反があった場合にコンテンツをブロックせず\ ``/csp_report``\ にレポートを送信するためには、以下のようなヘッダを出力する。

  * レスポンスヘッダの出力例

    .. code-block:: text

      Content-Security-Policy-Report-Only: default-src 'self'; report-uri /csp_report;

.. note:: \ **混在コンテンツについて**\

  HTTPSのページの中にHTTPで送られてくるコンテンツ（画像、動画、スタイルシート、スクリプト等）が含まれる場合、混在コンテンツと呼ばれる。混在コンテンツが存在する場合、中間者攻撃を受けるリスクが発生する

  Google Chrome 81以降では混在コンテンツに対してHTTPSアクセスを強制し、HTTPSでアクセスできない場合はブロックを行う。

  upgrade-insecure-requestsディレクティブを指定することでChromeと同等の動作をブラウザに指示することが出来る。

  * レスポンスヘッダの出力例

    .. code-block:: text

      Content-Security-Policy: upgrade-insecure-requests; default-src 'self';

|

Referrer-Policy
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Referrer-Policyヘッダは、Refererヘッダーで送信されるリファラー情報をリクエストに含めるかどうかを指定するためのヘッダである。
| リファラー情報の送信範囲や送信内容を制限することができる。

| リファラー情報を送信しないようにするには、以下のようなヘッダを出力する。

* レスポンスヘッダの出力例

.. code-block:: text

  Referrer-Policy: no-referrer

出力例以外の設定については\ `Response Headers <https://owasp.org/www-project-secure-headers/#div-headers>`_\ のReferrer-Policyを参照されたい。

|

.. _LinkageWithBrowserPermissionsPolicy:

Permissions-Policy
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Permissions-Policyヘッダは、権限の委譲や強力な機能の制御を行うためのヘッダである。
| ブラウザ内の特定のAPIやWeb機能を部分的に有効、無効、および変更することができる。

| 同じオリジンのフレーム(同じプロトコル、ホスト、ポート番号を持つフレーム)でのみGeolocationのAPIを使用できるようにするには、以下のようなヘッダを出力する。


* レスポンスヘッダの出力例

.. code-block:: text

  Permissions-Policy: geolocation=(self)

出力例以外の設定については\ `Response Headers <https://owasp.org/www-project-secure-headers/#div-headers>`_\ のPermissions-Policyを参照されたい。

|

Clear-Site-Data
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Clear-Site-Dataヘッダは、要求元の Webサイトに関連付けられた閲覧データ(Cookie、ストレージ、キャッシュ)をクリアするためのヘッダである。
| ログアウトやセキュリティ上の理由でWebサイトに関するデータをクリアしたい場合に使用する。

| ローカルにキャッシュされたデータを削除するには、以下のようなヘッダを出力する。

* レスポンスヘッダの出力例

.. code-block:: text

  Clear-Site-Data: "cache"

出力例以外の設定については\ `Response Headers <https://owasp.org/www-project-secure-headers/#div-headers>`_\ のClear-Site-Dataを参照されたい。

.. note::

  本機能はLogoutHandlerの仕組みを用いて適用されるが、自動的には適用されない。

  詳しくは \ :ref:`SpringSecurityAuthenticationLogout`\ を参照されたい。

|

X-XSS-Protection
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. warning::

  X-XSS-Protectionヘッダは\ `OWASP <https://owasp.org/www-project-secure-headers/#div-headers>`_\ で非推奨となっている。

  Spring Securityのデフォルト設定ではX-XSS-Protectionヘッダを出力しているが、XSS Auditorを無効にし、レスポンスを処理するブラウザのデフォルトの挙動を取らせないようにするために、ヘッダの値を"\ ``0``\ "としている。

  \ `OWASP <https://owasp.org/www-project-secure-headers/#div-headers>`_\ ではX-XSS-Protectionヘッダの代わりに\ :ref:`Content-Security-Policyヘッダ<LinkageWithBrowserContentSecurityPolicy>`\ を使用することを推奨している。

| X-XSS-Protectionヘッダは、ブラウザのXSSフィルター機能を使って有害スクリプトを検出する方法を指示するためのヘッダである。
| XSSフィルター機能を有効にして有害なスクリプトを検知するとこで、クロスサイトスクリプティングを使った攻撃を受けるリスクを減らすことができる。
| XSSフィルター機能を有効にして有害なスクリプトを検知するためには、以下のようなヘッダを出力する。

* レスポンスヘッダの出力例\ **(Spring Securityのデフォルト出力)**\

.. code-block:: text

  X-XSS-Protection: 0

|

Public-Key-Pins
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. warning::

  Public-Key-Pinsヘッダは\ `OWASP <https://owasp.org/www-project-secure-headers/#div-headers>`_\ で非推奨となっている。

| Public-Key-Pinsヘッダは、サイトの証明書の真正性を担保するために、サイトに紐づく証明書の公開鍵をブラウザに提示するヘッダである。
| サイトへの再訪問時に中間者攻撃と呼ばれる攻撃手法を使って悪意のあるサイトに誘導された場合でも、
| ブラウザが保持する真性のサイト証明書の公開鍵と悪意あるサイトが提示する証明書の公開鍵の不一致を検知して、アクセスをブロックすることができる。

ブラウザが保持する情報と一致しない証明書を検出した場合にアクセスをブロックさせるためには、以下のようなヘッダを出力する。

* レスポンスヘッダの出力例

.. code-block:: text

  Public-Key-Pins: max-age=5184000 ; pin-sha256="d6qzRu9zOECb90Uez27xWltNsj0e1Md7GkYYkVoZWmM=" ; pin-sha256="E9CZ9INDbd+2eRQozYqqbQ2yXLVKB9+xcprMF+44U1g="

.. note:: \ **違反レポートの送信について**\

  アクセスブロック時にブラウザに違反レポートを送信させるためには、Content-Security-Policyと同様にreport-uriディレクティブを指定する。

  また、ブラウザにアクセスをブロックさせずに違反レポートを送信させるためには、Public-Key-Pinsヘッダの代わりにPublic-Key-Pins-Report-Onlyヘッダを使用する。

.. note:: \ **Public-Key-Pinsヘッダの設定について**\

  Public-Key-Pinsヘッダの設定に誤りがあった場合、ユーザが長期間サイトにアクセスできなくなる可能性がある。

|

Feature-Policy
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. warning::

  Feature-Policyヘッダは\ `OWASP <https://owasp.org/www-project-secure-headers/#div-headers>`_\ で非推奨となっている。

  \ `OWASP <https://owasp.org/www-project-secure-headers/#div-headers>`_\ ではFeature-Policyヘッダの代わりに\ :ref:`Permissions-Policyヘッダ<LinkageWithBrowserPermissionsPolicy>`\ を使用することを推奨している。

| Feature-Policyヘッダは、機能の制御を行うためのヘッダである。
| ブラウザ内の特定のAPIやWeb機能を部分的に有効、無効、および変更することができる。
| ブラウザが保持する真性のサイト証明書の公開鍵と悪意あるサイトが提示する証明書の公開鍵の不一致を検知して、アクセスをブロックすることができる。

ブラウザが保持する情報と一致しない証明書を検出した場合にアクセスをブロックさせるためには、以下のようなヘッダを出力する。

* レスポンスヘッダの出力例

.. code-block:: text

  Feature-Policy: geolocation 'self'

|

How to use
--------------------------------------------------------------------------------

セキュリティヘッダ出力機能の適用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

前述のセキュリティヘッダ出力機能を適用する方法を説明する。

セキュリティヘッダ出力機能は、Spring 3.2から追加された機能であり、以下のセキュリティヘッダがデフォルトで適用されるようになっている。

* Cache-Control (Pragma, Expires)
* X-Frame-Options
* X-Content-Type-Options
* X-XSS-Protection
* Strict-Transport-Security

| そのため、デフォルトで適用されるセキュリティヘッダ出力機能を有効にするための特別な定義は不要である。 
| なお、デフォルトで適用されるセキュリティヘッダ出力機能を適用したくない場合は、明示的に無効化する必要がある。 

セキュリティヘッダ出力機能を無効化する場合は、以下のようなbean定義を行う。

.. tabs::
  .. group-tab:: Java Config

    * SpringSecurityConfig.javaの定義例
    
    .. code-block:: java
    
      @Bean
      public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
          // omitted
          http.headers(headers -> headers.disable()); // headers.disable()を呼び出して無効化
          // omitted
  
          return http.build();
      }

  .. group-tab:: XML Config

    * spring-security.xmlの定義例
    
    .. code-block:: xml
    
      <sec:http request-matcher="ant">
          <!-- omitted -->
          <sec:headers disabled="true"/> <!-- disabled属性にtrueを設定して無効化 -->
          <!-- omitted -->
      </sec:http>

|

セキュリティヘッダの選択
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| 出力するセキュリティヘッダを選択したい場合は、以下のようなbean定義を行う。
| ここでは、Spring Securityが提供している非推奨以外のすべてのセキュリティヘッダを出力する例になっているが、実際には必要なものだけ指定すること。

.. tabs::
  .. group-tab:: Java Config

    * SpringSecurityConfig.javaの定義例
    
    .. code-block:: java

      @Bean
      public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
          // omitted
          http.headers(headers -> headers.defaultsDisabled() // (1)
                  .cacheControl(Customizer.withDefaults()) // (2)
                  .frameOptions(Customizer.withDefaults()) // (3)
                  .contentTypeOptions(Customizer.withDefaults()) // (4)
                  .httpStrictTransportSecurity(Customizer.withDefaults()) // (5)
                  .contentSecurityPolicy(csp -> csp.policyDirectives("default-src 'self'")) // (6)
                  .referrerPolicy(Customizer.withDefaults()) // (7)
                  .permissionsPolicy(permissions -> permissions.policy("geolocation=(self)")) // (8)
          ); 
          // omitted
  
          return http.build();
      }

      @Bean
      public ClearSiteDataHeaderWriter clearSiteDataHeaderWriter() {
          return new ClearSiteDataHeaderWriter(Directive.CACHE);
      }
    
    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90
    
      * - 項番
        - 説明
      * - | (1)
        - | デフォルトで適用されるヘッダ出力を行うコンポーネント登録を無効化する。
      * - | (2)
        - | Cache-Control(Pragma, Expires)ヘッダを出力するコンポーネントを登録する。
      * - | (3)
        - | Frame-Optionsヘッダを出力するコンポーネントを登録する。
      * - | (4)
        - | X-Content-Type-Optionsヘッダを出力するコンポーネントを登録する。
      * - | (5)
        - | Strict-Transport-Securityヘッダを出力するコンポーネントを登録する。
      * - | (6)
        - | Content-Security-PolicyヘッダまたはContent-Security-Policy-Report-Onlyヘッダを出力するコンポーネントを登録する。
      * - | (7)
        - | Referrer-Policyヘッダを出力するコンポーネントを登録する。
      * - | (8)
        - | Permissions-Policyヘッダを出力するコンポーネントを登録する。

  .. group-tab:: XML Config

    * spring-security.xmlの定義例
    
    .. code-block:: xml
    
      <sec:http request-matcher="ant">
          <!-- omitted -->
          <sec:headers defaults-disabled="true"> <!-- (1) -->
              <sec:cache-control /> <!-- (2) -->
              <sec:frame-options /> <!-- (3) -->
              <sec:content-type-options /> <!-- (4) -->
              <sec:hsts /> <!-- (5) -->
              <sec:content-security-policy policy-directives="default-src 'self'"/> <!-- (6) -->
              <sec:referrer-policy /> <!-- (7) -->
              <sec:permissions-policy policy="geolocation=(self)"/> <!-- (8) -->
          </sec:headers>
          <!-- omitted -->
      </sec:http>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90
    
      * - 項番
        - 説明
      * - | (1)
        - | デフォルトで適用されるヘッダ出力を行うコンポーネント登録を無効化する。
      * - | (2)
        - | Cache-Control(Pragma, Expires)ヘッダを出力するコンポーネントを登録する。
      * - | (3)
        - | Frame-Optionsヘッダを出力するコンポーネントを登録する。
      * - | (4)
        - | X-Content-Type-Optionsヘッダを出力するコンポーネントを登録する。
      * - | (5)
        - | Strict-Transport-Securityヘッダを出力するコンポーネントを登録する。
      * - | (6)
        - | Content-Security-PolicyヘッダまたはContent-Security-Policy-Report-Onlyヘッダを出力するコンポーネントを登録する。
      * - | (7)
        - | Referrer-Policyヘッダを出力するコンポーネントを登録する。
      * - | (8)
        - | Permissions-Policyヘッダを出力するコンポーネントを登録する。

また、不要なものだけ無効化する方法も存在する。 

.. tabs::
  .. group-tab:: Java Config

    * SpringSecurityConfig.javaの定義例
        
    .. code-block:: java

      @Bean
      public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
          // omitted
          http.headers(headers -> headers.cacheControl(cacheControl -> cacheControl.disable())); // cacheControl.disable()を呼び出して無効化
          // omitted
  
          return http.build();
      }

  .. group-tab:: XML Config

    * spring-security.xmlの定義例
        
    .. code-block:: xml 
    
      <sec:http request-matcher="ant">
          <!-- omitted -->
          <sec:headers>
              <sec:cache-control disabled="true"/> <!-- disabled属性にtrueを設定して無効化 --> 
          </sec:headers>
          <!-- omitted -->
      </sec:http>

上記の例だと、Cache-Control関連のヘッダだけが出力されなくなる。 

セキュリティヘッダの詳細については\ `Spring Security Reference -Default Security Headers- <https://docs.spring.io/spring-security/reference/servlet/exploits/headers.html#servlet-headers-default>`_\ を参照されたい。

|

セキュリティヘッダのオプション指定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityのbean定義を変更することで、各要素の属性にオプション\ [#fSpringSecurityLinkageWithBrowser2]_\ を指定することができる。

.. tabs::
  .. group-tab:: Java Config

    * SpringSecurityConfig.javaの定義例

    .. code-block:: java

      @Bean
      public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
          // omitted
          http.headers(headers -> headers.addHeaderWriter(
                new XFrameOptionsHeaderWriter(XFrameOptionsMode.SAMEORIGIN)));
          return http.build();
      }

  .. group-tab:: XML Config

    * spring-security.xmlの定義例
    
    .. code-block:: xml
    
      <sec:frame-options policy="SAMEORIGIN" />

.. [#fSpringSecurityLinkageWithBrowser2] 各要素で指定できるオプションは\ `Spring Security Reference -The Security Namespace (<headers>)- <https://docs.spring.io/spring-security/reference/servlet/appendix/namespace/http.html#nsa-headers>`_\ を参照されたい。

|

カスタムヘッダの出力
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityがデフォルトで用意していないヘッダを出力することもできる。

以下のヘッダを出力するケースの例を説明する。

.. code-block:: text

  X-WebKit-CSP: default-src 'self'

上記のヘッダを出力する場合は、以下のようなbean定義を行う。

.. tabs::
  .. group-tab:: Java Config

    * SpringSecurityConfig.javaの定義例

    .. code-block:: java

      @Bean
      public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
          // omitted
          http.headers(headers -> headers.addHeaderWriter(
                  new StaticHeadersWriter("X-WebKit-CSP", "default-src 'self'")));
          // omitted
  
          return http.build();
      }
    
    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90
    
      * - 項番
        - 説明
      * - | (1)
        - | \ ``HeadersConfigurer#addHeaderWriter``\ の引数に\ ``StaticHeadersWriter``\ を設定し、第一引数にヘッダ名を、第二引数にヘッダ値を指定する。

  .. group-tab:: XML Config

    * spring-security.xmlの定義例

    .. code-block:: xml
    
      <sec:http request-matcher="ant">
          <!-- omitted -->
          <sec:headers>
              <sec:header name="X-WebKit-CSP" value="default-src 'self'"/>
          </sec:headers>
          <!-- omitted -->
      </sec:http>
    
    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90
    
      * - 項番
        - 説明
      * - | (1)
        - | \ ``<sec:headers>``\ 要素の子要素として\ ``<sec:header>``\ を追加し、\ ``name``\ 属性にヘッダ名を\ ``value``\ 属性にヘッダ値を指定する。

|

.. _LinkageWithBrowserEachRequestPattern:

リクエストパターン毎のセキュリティヘッダの出力
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityは、\ ``RequestMatcher``\ インタフェースの仕組みを利用して、リクエストのパターン毎にセキュリティヘッダの出力を制御することも可能である。

例えば、保護対象のコンテンツが\ ``/secure/``\ というパスの配下に格納されていて、保護対象のコンテンツへアクセスした時だけCache-Controlヘッダを出力する場合は、以下のようなbean定義を行う。

.. tabs::
  .. group-tab:: Java Config

    * SpringSecurityConfig.javaの定義例
    
    .. code-block:: java

      // (1)
      @Bean("secureCacheControlHeadersWriter")
      public DelegatingRequestMatcherHeaderWriter secureCacheControlHeadersWriter() {
  
          AntPathRequestMatcher antPathRequestMatcher = new AntPathRequestMatcher("/secure/**");
          CacheControlHeadersWriter cacheControlHeadersWriter = new CacheControlHeadersWriter();
  
          return new DelegatingRequestMatcherHeaderWriter(antPathRequestMatcher, cacheControlHeadersWriter);
      }
  
      @Bean
      public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
          // omitted
          http.headers(headers -> headers.defaultsDisabled().addHeaderWriter(
                  secureCacheControlHeadersWriter())); // (2)
          // omitted
  
          return http.build();
      }
    
    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90
    
      * - 項番
        - 説明
      * - | (1)
        - | \ ``RequestMatcher``\ と\ ``HeadersWriter``\ インタフェースの実装クラスを指定して\ ``DelegatingRequestMatcherHeaderWriter``\ クラスのbeanを定義する。
      * - | (2)
        - | \ ``HeadersConfigurer#addHeaderWriter``\ の引数に(1)で定義した\ ``HeaderWriter``\ のbeanを指定する。

  .. group-tab:: XML Config

    * spring-security.xmlの定義例
    
    .. code-block:: xml
    
      <!-- (1) -->
      <bean id="secureCacheControlHeadersWriter"
            class="org.springframework.security.web.header.writers.DelegatingRequestMatcherHeaderWriter">
          <constructor-arg>
              <bean class="org.springframework.security.web.util.matcher.AntPathRequestMatcher">
                  <constructor-arg value="/secure/**"/>
              </bean>
          </constructor-arg>
          <constructor-arg>
              <bean class="org.springframework.security.web.header.writers.CacheControlHeadersWriter"/>
          </constructor-arg>
      </bean>
    
      <sec:http request-matcher="ant">
          <!-- omitted -->
          <sec:headers>
              <sec:header ref="secureCacheControlHeadersWriter"/> <!-- (2) -->
          </sec:headers>
          <!-- omitted -->
      </sec:http>
    
    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90
    
      * - 項番
        - 説明
      * - | (1)
        - | \ ``RequestMatcher``\ と\ ``HeadersWriter``\ インタフェースの実装クラスを指定して\ ``DelegatingRequestMatcherHeaderWriter``\ クラスのbeanを定義する。
      * - | (2)
        - | \ ``<sec:headers>``\ 要素の子要素として\ ``<sec:header>`` を追加し、\ ``ref``\ 属性に(1)で定義した\ ``HeaderWriter``\ のbeanを指定する。

.. raw:: latex

  \newpage
