.. _SpringSecurityAuthorization:

認可
================================================================================

.. only:: html

.. contents:: 目次
  :local:

|

Overview
--------------------------------------------------------------------------------
本節では、Spring Securityが提供している認可機能について説明する。

| 認可処理は、アプリケーションの利用者がアクセスできるリソースを制御するための処理である。
| 利用者がアクセスできるリソースを制御するためのもっとも標準的な方法は、リソース(又はリソースの集合)毎にアクセスポリシーを定義しておき、利用者がリソースにアクセスしようとした時にアクセスポリシーを調べて制御する方法である。

| アクセスポリシーには、どのリソースにどのユーザーからのアクセスを許可するかを定義する。
| Spring Securityでは、以下の3つのリソースに対してアクセスポリシーを定義することができる。

* Webリソース
* Javaメソッド
* ドメインオブジェクト \ [#fSpringSecurityAuthorization1]_\
* 画面項目

本節では、「Webリソース」「Javaメソッド」「画面項目」のアクセスに対して認可処理を適用するための実装例(定義例)を紹介しながら、Spring Securityの認可機能について説明する。

.. [#fSpringSecurityAuthorization1] ドメインオブジェクトのアクセスに対する認可処理については、 \ `Spring Security Reference -Domain Object Security (ACLs)- <https://docs.spring.io/spring-security/reference/servlet/authorization/acls.html>`_\ を参照されたい。

|

認可処理のアーキテクチャ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityは、以下のような流れで認可処理を行う。

.. figure:: ./images_Authorization/AuthorizationArchitecture.png
  :width: 100%

  \ **認可処理のアーキテクチャ**\

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | クライアントは、任意のリソースにアクセスする。
  * - | (2)
    - | \ ``AuthorizationFilter``\ クラスは、\ ``AuthorizationManager``\ インタフェースのメソッドを呼び出し、リソースへのアクセス権の有無をチェックする。
  * - | (3)
    - | \ ``AuthorizationManager``\ の実装クラスである\ ``RequestMatcherDelegatingAuthorizationManager``\ が、受け取ったリクエストを適切な\ ``AuthorizationManager``\ に振り分けてアクセス権の有無をチェックする。
  * - | (4)
    - | \ ``AuthorizationFilter``\ は、\ ``AuthorizationManager``\ によってアクセス権が付与された場合に限り、リソースへアクセスする。

|

ExceptionTranslationFilter
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| \ ``ExceptionTranslationFilter``\ は、認可処理(\ ``AuthorizationManager``\ )で発生した例外をハンドリングし、クライアントへ適切なレスポンスを行うためのSecurity Filterである。
| デフォルトの実装では、未認証ユーザーからのアクセスの場合は認証を促すレスポンス、認証済みのユーザーからのアクセスの場合は認可エラーを通知するレスポンスを返却する。
|

AuthorizationFilter
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| \ ``AuthorizationFilter``\ は、HTTPリクエストに対して認可処理を適用するためのSecurity Filterで、実際の認可処理は\ ``AuthorizationManager``\ に委譲する。
| \ ``AuthorizationManager``\ インタフェースのメソッドを呼び出す際には、クライアントがアクセスしようとしたリソースに指定されているアクセスポリシーを連携する。
| アクセスが許可されると、\ ``AuthorizationFilter``\ は\ ``FilterChain``\ を続行する。
|

AuthorizationManager
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| \ ``AuthorizationManager``\ は、アクセスしようとしたリソースに対してアクセス権があるかチェックを行うためのインタフェースである。
| アクセス権がないと判断した場合は、\ ``AccessDeniedException``\ を発生させアクセスを拒否する。
| Spring Securityでは以下の実装クラスを提供している。
|

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: \ **Spring Securityが提供するAuthorizationManagerの実装クラス**\
  :header-rows: 1
  :widths: 25 75

  * - クラス名
    - 説明
  * - | \ ``RequestMAtcherDelegatingAuthorizationManager``\
    - | リクエストに一致する\ ``RequestMatcher``\ を基に、認可処理を特定の\ ``AuthorizationManager``\ に移譲する。
  * - | \ ``AuthorityAuthorizationManager``\
    - | Spring Securityが提供する一般的な\ ``AuthorizationManager``\ 。
      | 認証情報(\ ``Authentication``\ )に指定された権限が含まれているかどうかを評価し、現在のユーザーが認可されているかどうかを判別する。
  * - | \ ``AuthenticatedAuthorizationManager``\
    - | 匿名ユーザー、完全認証ユーザー、リメンバー認証ユーザーを区別するために使用される。
  * - | \ ``JSR250AuthorizationManager``\
    - | 認証情報(\ ``Authentication``\ )がJSR-250セキュリティアノテーションから指定された権限を含んでいるかどうかを評価する。
  * - | \ ``SecuredAuthorizationManager``\
    - | 認証情報(\ ``Authentication``\ )がSpring Securityの\ ``Secured``\ アノテーションから指定された権限を含んでいるかどうかを評価する。
  * - | \ ``PreAuthorizeAuthorizationManager``\
    - | 認証情報(\ ``Authentication``\ )が\ ``PreAuthorize``\ アノテーションから指定された権限を含んでいるかどうかを評価する。
  * - | \ ``PreAuthorizaAuthorizationMAnager``\
    - | 認証情報(\ ``Authentication``\ )が\ ``PostAuthorize``\ アノテーションから指定された権限を含んでいるかどうかを評価する。

| Spring Securityが提供する\ ``AuthorizationManager``\ 以外に、独自に構築した\ ``RequestMatcherDelegatingAuthorizationManager``\ を使用することも可能である。 
| 詳しくは、\ `Configure RequestMatcherDelegatingAuthorizationManager <https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html#_tabs_17>`_\ を参照されたい。
|

How to use
--------------------------------------------------------------------------------

| 認可機能を使用するために必要となるbean定義例(アクセスポリシーの指定方法)や実装方法について説明する。
|

.. _SpringSecurityAuthorizationPolicy:

アクセスポリシーの記述方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

アクセスポリシーの記述方法を説明する。

| Spring Securityは、アクセスポリシーを指定する記述方法として\ ``HttpSecurity``\ に対するメソッドチェーンをサポートしている。
| また、\ ``<intercept-url>``\ JSP Taglibなど、式が必要な場合に備えSpring Expression Language(SpEL)をサポートしている。

.. tip:: 

  SpELの使い方については本節でも紹介するが、より詳しい使い方を知りたい場合は\ `Spring Framework Documentation -Spring Expression Language (SpEL)- <https://docs.spring.io/spring-framework/docs/6.1.3/reference/html/core.html#expressions>`_\ を参照されたい。

|

代表的な認可メソッド
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| \ ``HttpSecurity#authorizeHttpRequests(Customizer<AuthorizeHttpRequestsConfigurer<HttpSecurity>.AuthorizationManagerRequestMatcherRegistry> authorizeHttpRequestsCustomizer)``\ のCustomizerを実装し、URLパターン経由でHttpServletRequestに基づいてアクセスを制限することができる。
| 代表的な認可メソッドは以下の通り。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table:: \ **Spring Securityが提供している認可メソッド**\
  :header-rows: 1
  :widths: 30 70
  :class: longtable

  * - メソッド名
    - 説明
  * - | \ ``hasRole(String role)``\
    - | ログインユーザーが、引数に指定したロールを保持している場合に\ ``true``\ を返却する。
      | ロールの\ ``ROLE_`` \ プレフィックスは省略可能である。
  * - | \ ``hasAnyRole(String... roles)``\
    - | ログインユーザーが、引数に指定したロールのいずれかを保持している場合に\ ``true``\ を返却する。
      | ロールの\ ``ROLE_`` \ プレフィックスは省略可能である。
  * - | \ ``anonymous()``\
    - | ログインしていない匿名ユーザーの場合に\ ``true``\ を返却する。
  * - | \ ``rememberMe()``\
    - | Remember Me認証によってログインしたユーザーの場合に\ ``true``\ を返却する。
  * - | \ ``authenticated()``\
    - | ログイン中の場合に\ ``true``\ を返却する。
  * - | \ ``fullyAuthenticated()``\
    - | Remember Me認証ではなく通常の認証プロセスによってログインしたユーザーの場合に\ ``true``\ を返却する。
  * - | \ ``permitAll()``\
    - | 常に\ ``true``\ を返却する。
  * - | \ ``denyAll()``\
    - | 常に\ ``false``\ を返却する。(\ **デフォルト値**\ )
  * - | \ ``access(AuthorizationManager<RequestAuthorizationContext> manager)``\
    - | カスタム\ ``AuthorizationManager``\ を使用して認可処理を実施する。

.. note:: 

  Spring Securityの認可処理のデフォルト値は\ ``denyAll``\ であるため、業務要件に応じ適切に認可する範囲を指定する必要がある。

|

Built-InのCommon Expressions
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityが用意している共通的なExpressionは以下の通り。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table:: \ **Spring Securityが提供している共通的なExpression**\
  :header-rows: 1
  :widths: 30 70
  :class: longtable

  * - Expression
    - 説明
  * - | \ ``hasRole(String role)``\
    - | ログインユーザーが、引数に指定したロールを保持している場合に\ ``true``\ を返却する。
      | ロールの\ ``ROLE_`` \ プレフィックスは省略可能である。
  * - | \ ``hasAnyRole(String... roles)``\
    - | ログインユーザーが、引数に指定したロールのいずれかを保持している場合に\ ``true``\ を返却する。
      | ロールの\ ``ROLE_`` \ プレフィックスは省略可能である。
  * - | \ ``isAnonymous()``\
    - | ログインしていない匿名ユーザーの場合に\ ``true``\ を返却する。
  * - | \ ``isRememberMe()``\
    - | Remember Me認証によってログインしたユーザーの場合に\ ``true``\ を返却する。
  * - | \ ``isAuthenticated()``\
    - | ログイン中の場合に\ ``true``\ を返却する。
  * - | \ ``isFullyAuthenticated()``\
    - | Remember Me認証ではなく通常の認証プロセスによってログインしたユーザーの場合に\ ``true``\ を返却する。
  * - | \ ``permitAll``\
    - | 常に\ ``true``\ を返却する。
  * - | \ ``denyAll``\
    - | 常に\ ``false``\ を返却する。(\ **デフォルト値**\ )
  * - | \ ``principal``\
    - | 認証されたユーザーのユーザー情報(\ ``UserDetails``\ インタフェースを実装したクラスのオブジェクト)を返却する。
  * - | \ ``authentication``\
    - | 認証されたユーザーの認証情報(\ ``Authentication``\ インタフェースを実装したクラスのオブジェクト)を返却する。

.. note:: \ **Expressionを使用した認証情報へのアクセス**\

  Expressionとして\ ``principal``\ や\ ``authentication``\ を使用すると、ログインユーザーのユーザー情報や認証情報を参照することができるため、ロール以外の属性を使ってアクセスポリシーを設定することが可能になる。

.. note:: \ **Spring Secuirtyが提供するその他のExpression**\

  上記に記載した以外にも、Spring Securityではログインユーザーが保持する権限を確認するExpressionとして、\ ``hasAuthority(String authority)``\ 、\ ``hasAnyAuthority(String... authorities)``\ 、\ ``hasPermission(Object target, Object permission)``\ 、\ ``hasPermission(Object targetId, String targetType, Object permission)``\ を提供している。

  ユーザの属性により権限をグループ化したものがロールであり、一般的には個々の権限による認可ではなくロールによる認可が推奨される。

  Spring Securityの認可においてはいずれもログインユーザが「指定した権限（ロール）を保持しているか」を確認するため利用方法に違いはないが、権限名はロール名と異なり\ ``ROLE_``\ のようなプレフィックスがないため、権限の定義と認可で名称を完全一致させる必要がある。

.. note:: 

  Spring Securityの認可処理のデフォルト値は\ ``denyAll``\ であるため、業務要件に応じ適切に認可する範囲を指定する必要がある。

|

.. _built-incommon-expressions:

Built-InのWeb Expressions
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityが用意しているWebアプリケーション向けExpressionは以下の通り。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table:: \ **Spring Securityが提供するWebアプリケーション向けExpression**\
  :header-rows: 1
  :widths: 30 70

  * - Expression
    - 説明
  * - | \ ``hasIpAddress(String ipAddress)``\
    - | リクエスト元のIPアドレスが、引数に指定したIPアドレス体系に一致する場合に\ ``true``\ を返却する。

|

演算子の使用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 演算子を使用した判定も行うことができる。
| 以下の例では、ロールと、リクエストされたIPアドレス両方に合致した場合、アクセス可能となる。

.. tabs::
  .. group-tab:: Java Config

    * SpringSecurityConfig.javaの定義例
    
      .. code-block:: java

        @Bean
        public SecurityFilterChain filterChain(HttpSecurity http) {
            // omitted
            http.authorizeHttpRequests(authz -> authz
                    // omitted
                    .access(new WebExpressionAuthorizationManager("hasRole('ADMIN') and hasIpAddress('192.168.10.1')"))
                    // omitted
                    );
            return http.build();
        }

  .. group-tab:: XML Config

    * spring-security.xmlの定義例
    
      .. code-block:: xml
    
        <sec:http request-matcher="ant">
            <sec:intercept-url pattern="/admin/**" access="hasRole('ADMIN') and hasIpAddress('192.168.10.1')"/>
            <!-- omitted -->
        </sec:http>

\ **使用可能な演算子一覧**\

.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 20 80

  * - 演算子
    - 説明
  * - | \ ``[式1] and [式2]``\
    - | 式1、式2が、どちらも真の場合に、真を返す。
  * - | \ ``[式1] or [式2]``\
    - | いずれかの式が、真の場合に、真を返す。
  * - | \ ``![式]``\
    - | 式が真の場合は偽を、偽の場合は真を返す。

|

.. _AuthorizationToWebResources:

Webリソースへの認可
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityは、サーブレットフィルタの仕組みを利用してWebリソース(HTTPリクエスト)に対して認可処理を行う。

|

認可処理の適用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Webリソースに対して認可処理を適用する場合は、以下のようなbean定義を行う。

.. tabs::
  .. group-tab:: Java Config

    * SpringSecurityConfig.javaの定義例
    
    .. code-block:: java

      @Bean
      public SecurityFilterChain filterChain(HttpSecurity http) {
          // omitted  
          http.authorizeHttpRequests(authz -> authz
                  .requestMatchers(new AntPathRequestMatcher("/**")).authenticated() // (1)
                  );
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
        - | \ ``HttpSecurity``\ のメソッドチェーンにより、HTTPリクエストに対してアクセスポリシーを定義する。
          | ここでは、\ ``authenticated()``\ メソッドを呼び出し「Webアプリケーション配下の全てのリクエストに対して認証済みのユーザーのみアクセスを許可する」というアクセスポリシーを定義している。

  .. group-tab:: XML Config

    * spring-security.xmlの定義例
    
    .. code-block:: xml
    
      <sec:http request-matcher="ant">
          <!-- omitted -->
          <sec:intercept-url pattern="/**" access="isAuthenticated()" /> <!-- (1) -->
          <!-- omitted -->
      </sec:http>
    
    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90
    
      * - 項番
        - 説明
      * - | (1)
        - | \ ``<sec:intercept-url>``\ タグに、HTTPリクエストに対してアクセスポリシーを定義する。
          | ここでは、SpELを使用して「Webアプリケーション配下の全てのリクエストに対して認証済みのユーザーのみアクセスを許可する」というアクセスポリシーを定義している。

|

.. _access_policy_designate_web_resource:

アクセスポリシーの定義
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

bean定義ファイルを使用して、Webリソースに対してアクセスポリシーを定義する方法について説明する。

|

アクセスポリシーを適用するWebリソースの指定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

.. tabs::
  .. group-tab:: Java Config

    \ ``HttpSecurity#authorizeHttpRequests(Customizer<AuthorizeHttpRequestsConfigurer<HttpSecurity>.AuthorizationManagerRequestMatcherRegistry> authorizeHttpRequestsCustomizer)``\ のCustomizerを実装し、アクセスポリシーを設定する。
    
    * SpringSecurityConfig.javaの定義例
        
      .. code-block:: java
    
        @Bean
        public SecurityFilterChain filterChain(HttpSecurity http) {
            // omitted
            http.authorizeHttpRequests(authz -> authz
                    .requestMatchers(new AntPathRequestMatcher("/admin/accounts/**", HttpMethod.GET.name())).hasRole("ACCOUNT_MANAGER") // (1)(4)
                    .requestMatchers(new AntPathRequestMatcher("/admin/configurations/**"))
                            .access(new WebExpressionAuthorizationManager("hasIpAddress('127.0.0.1') and hasRole('CONFIGURATION_MANAGER')")) // (2)
                    .requestMatchers(new AntPathRequestMatcher("/admin/**")).hasAnyRole("USER", "ADMIN")
                    .requestMatchers(new AntPathRequestMatcher("/**")).denyAll()
                    );
            http.requiresChannel(channel -> channel.anyRequest().requiresSecure()); // (3)
            // omitted
            return http.build();
        }
    
      .. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
      .. list-table:: \ **メソッドチェーンによるアクセスポリシーを適用する設定**\
        :header-rows: 1
        :widths: 20 80
        
        * - 項番
          - 説明
        * - | (1)
          - | パスパターンに一致するリソースを適用対象とするため、\ ``AuthorizationManagerRequestMatcherRegistry.requestMatchers``\ に\ ``RequestMatcher``\ オブジェクトを設定する。
            | 上記設定例では\ ``AntPathRequestMatcher``\ を指定している。設定できる項目は以下となる。
    
              .. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
              .. list-table:: \ **AntPathRequestMatcherの設定項目**\
                :header-rows: 1
                :widths: 20 80
    
                * - 変数名
                  - 説明
                * - | \ ``pattern``\
                  - | Ant形式又は正規表現で指定したパスパターンに一致するリソースを適用対象にする。
                * - | \ ``httpMethod``\
                  - | 指定したHTTPメソッド(GET,POSTなど)を使ってアクセスがあった場合に適用対象にする。
    
        * - | (2)
          - | Expressionsを使用する場合は\ ``AuthorizedUrl#access``\ を使用し、\ ``WebExpressionAuthorizationManager``\ を設定する。
            | \ ``WebExpressionAuthorizationManager``\ のコンストラクタ引数に認可制御用のExpressionsを設定する。
        * - | (3)
          - | \ ``HttpSecurity#requiresChannel(Customizer<ChannelSecurityConfigurer<HttpSecurity>.ChannelRequestMatcherRegistry> requiresChannelCustomizer)``\ のCustomizerを実装し、指定したプロトコルを強制することができる。
            | 上記設定例では\ ``http``\ でリクエストされた際に\ ``https``\ へリダイレクトする処理となる。
        * - | (4)
          - | \ ``requestMatchers``\ が返却する    

    
  .. group-tab:: XML Config

    \ ``<sec:intercept-url>``\ タグに、HTTPリクエストに対してアクセスポリシーを定義する。

    * spring-security.xmlの定義例
    
    .. code-block:: xml

      <sec:http request-matcher="ant">
          <sec:intercept-url pattern="/admin/accounts/**" method="GET" requires-channel="https" access="hasRole('ACCOUNT_MANAGER')"/> <!-- (1) -->
          <sec:intercept-url pattern="/admin/configurations/**" access="hasIpAddress('127.0.0.1') and hasRole('CONFIGURATION_MANAGER')" />
          <sec:intercept-url pattern="/admin/**" access="hasRole('ADMIN')" />
          <!-- omitted -->
      </sec:http>

    .. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
    .. list-table:: \ **アクセスポリシーを適用する設定**\
      :header-rows: 1
      :widths: 20 80
    
      * - 項番
        - 説明
      * - | (1)
        - | パスパターンに一致するリソースを適用対象とするため、\ ``<sec:intercept-url>``\ にリソースを設定する。
          | 設定できる属性は以下となる。

              .. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
              .. list-table:: \ **<sec:intercept-url>の属性項目**\
                :header-rows: 1
                :widths: 20 80
    
                * - 属性名
                  - 説明
                * - | \ ``pattern``\
                  - | Ant形式又は正規表現で指定したパスパターンに一致するリソースを適用対象にする。
                * - | \ ``httpMethod``\
                  - | 指定したHTTPメソッド(GET,POSTなど)を使ってアクセスがあった場合に適用対象にする。
                * - | \ ``requires-channel``\
                  - | 「http」もしくは「https」を指定する。指定したプロトコルでのアクセスを強制するための属性。
                    | 指定しない場合、どちらでもアクセス可能である。
                * - | \ ``access``\
                  - | SpELでのアクセス制御式や、アクセス可能なロールを指定する。

| Spring Securityは定義した順番でリクエストとのマッチング処理を行い、最初にマッチした定義を適用する。
| そのため、bean定義ファイルを使用してアクセスポリシーを指定する場合も定義順番には注意が必要である。

.. warning::

    Spring Security 4.1以降、Spring Securityがデフォルトで使用している\ `AntPathRequestMatcher` \ のパスマッチングの仕様が大文字・小文字を区別する様になった。

    例えば以下に示すように、\ ``/Todo/List``\ というパスが割り当てられたSpring MVCのエンドポイントに対してアクセスポリシーを定義する場合は、パスパターンについて\ ``/Todo/List``\ や \ ``/Todo/*``\ のように大文字・小文字をそろえる必要がある。

    誤って\ ``/todo/list``\ や\ ``/todo/**``\ など大文字・小文字がそろっていない値を指定してしまうと、意図した認可制御が行われなくなるので注意されたい。

    * Spring MVCのエンドポイントの実装例

      .. code-block:: java

        @RequestMapping(value="/Todo/List")
        public String viewTodoList(){
            // omitted
        }

    * アクセスポリシーの定義例

      .. tabs::
        .. group-tab:: Java Config

          .. code-block:: java
    
            @Bean
            public SecurityFilterChain filterChain(HttpSecurity http) {
                // omitted
                http.authorizeHttpRequests(authz -> authz
                        .requestMatchers(new AntPathRequestMatcher("/Todo/List")).authenticated()
                        // omitted
                        );
                // omitted
                return http.build();
            }

        .. group-tab:: XML Config

          .. code-block:: xml
    
            <sec:http request-matcher="ant">
                <sec:intercept-url pattern="/Todo/List" access="isAuthenticated()" />
                <!-- omitted -->
            </sec:http>

.. note::

  Spring MVCとSpring Securityでは、リクエストとのマッチングの仕組みが厳密には異なっており、この差異を利用してSpring Securityの認可機能を突破し、ハンドラメソッドにアクセスできる脆弱性が存在する。
    
  本事象の詳細は「\ `CVE-2016-5007 Spring Security / MVC Path Matching Inconsistency <https://tanzu.vmware.com/security/cve-2016-5007>`_\ 」を参照されたい。

  \ ``trimTokens``\ プロパティに\ ``true``\ を設定した\ ``org.springframework.util.AntPathMatcher``\ のBeanがSpring MVCに適用されている場合に、本事象が発生する。
    
  デフォルト値は\ ``false``\ であるため、意図的に変更しない限り本事象は発生しない。

|

アクセスポリシーの設定例
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| ログインユーザーに「ROLE_USER」「ROLE_ADMIN」というロールがある場合を例に、設定例を示す。

.. tabs::
  .. group-tab:: Java Config

    * \ ``SpringSecurityConfig.java``\
    
      .. code-block:: java

        @Bean
        public SecurityFilterChain filterChain(HttpSecurity http) {
            // omitted
            http.authorizeHttpRequests(authz -> authz
                    .requestMatchers(new AntPathRequestMatcher("/reserve/**")).hasAnyRole("USER", "ADMIN") // (1)
                    .requestMatchers(new AntPathRequestMatcher("/admin/**")).hasRole("ADMIN") // (2)
                    .requestMatchers(new AntPathRequestMatcher("/**")).denyAll() // (3)
                    );
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
          - | 「\ ``/reserve/**``\ 」にアクセスするためには、「ROLE_USER」もしくは「ROLE_ADMIN」ロールが必要である。
            | ログインユーザーが指定したロールを保持していれば真を返す。
        * - | (2)
          - | 「\ ``/admin/**``\ 」にアクセスするためには、「ROLE_ADMIN」ロールが必要である。
            | ログインユーザーが指定したロールを保持していれば真を返す。
        * - | (3)
          - | \ ``denyAll``\ を全てのパターンに設定し、
            | 権限設定が記述されていないURLに対してはどのユーザーもアクセス出来ない設定としている。

  .. group-tab:: XML Config

    * \ ``spring-security.xml``\
    
      .. code-block:: xml
    
        <sec:http request-matcher="ant">
            <sec:intercept-url pattern="/reserve/**" access="hasAnyRole('USER','ADMIN')" /> <!-- (1) -->
            <sec:intercept-url pattern="/admin/**" access="hasRole('ADMIN')" /> <!-- (2) -->
            <sec:intercept-url pattern="/**" access="denyAll" /> <!-- (3) -->
            <!-- omitted -->
        </sec:http>
    
      .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
      .. list-table::
        :header-rows: 1
        :widths: 10 90
    
        * - 項番
          - 説明
        * - | (1)
          - | 「\ ``/reserve/**``\ 」にアクセスするためには、「ROLE_USER」もしくは「ROLE_ADMIN」ロールが必要である。
            | ログインユーザーが指定したロールを保持していれば真を返す。
        * - | (2)
          - | 「\ ``/admin/**``\ 」にアクセスするためには、「ROLE_ADMIN」ロールが必要である。
            | ログインユーザーが指定したロールを保持していれば真を返す。
        * - | (3)
          - | \ ``denyAll``\ を全てのパターンに設定し、
            | 権限設定が記述されていないURLに対してはどのユーザーもアクセス出来ない設定としている。

.. note:: \ **URLパターンの記述順序について**\

  クライアントからのリクエストに対して、intercept-urlで記述されているパターンに、上から順にマッチさせ、マッチしたパターンに対してアクセス認可を行う。

  そのため、パターンの記述は、必ず、より限定されたパターンから記述すること。

|

パス変数の参照
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Spring Security 4.1以降では、アクセスポリシーを適用するリソースを指定する際にパス変数\ [#fPathVariableDescription]_\ を使用することができ、アクセスポリシーの定義内で\ ``#パス変数名``\ と指定することで参照できる。

ただし、拡張子を付けてアクセス可能なパスに対してパス変数を使用するアクセスポリシーを定義する場合は、パス変数値に拡張子部分が格納されない様に定義する必要がある。

例えば、パターンに\ ``/users/{userName}``\ と定義し、\ ``/users/personName.json``\ というリクエストパスを送信した際、アクセスポリシーの定義内で参照しているパス変数\ ``#userName``\ には\ ``personName``\ ではなく\ ``personName.json``\ が格納され、意図しない認可制御が行われてしまう。

この事象を防ぐためには、「拡張子を付けたパスに対するアクセスポリシー」を定義した後に、「拡張子を付けないパスに対するアクセスポリシー」を定義する必要がある。

以下の例は、ログインユーザが自身のユーザ情報のみアクセスできる様にアクセスポリシーを定義している。

* ワイルドカードを使用する場合

  .. tabs::
    .. group-tab:: Java Config

      * SpringSecurityConfig.javaの定義例

      .. code-block:: java

        @Bean
        public SecurityFilterChain filterChain(HttpSecurity http) {
            // omitted
            http.authorizeHttpRequests(authz -> authz
                    .requestMatchers(new AntPathRequestMatcher("/users/{userName}.*"))
                            .access(new WebExpressionAuthorizationManager("isAuthenticated() and #userName == principal.username")) // (1)
                    .requestMatchers(new AntPathRequestMatcher("/users/{userName}/**"))
                            .access(new WebExpressionAuthorizationManager("isAuthenticated() and #userName == principal.username")) // (2)
                    // omitted
                    );
            // omitted
            return http.build();
        }
    
      .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
      .. list-table::
        :header-rows: 1
        :widths: 10 90
        :class: longtable
    
        * - 項番
          - 説明
        * - | (1)
          - | 「拡張子を付けたパスに対するアクセスポリシー」を定義する。
        * - | (2)
          - | 「拡張子を付けないパスに対するアクセスポリシー」を定義する。
            | ワイルドカードを使用して\ ``/users/{userName}``\ で始まるパスに対するアクセスポリシーを定義する。

    .. group-tab:: XML Config

      .. code-block:: xml
    
        <sec:http request-matcher="ant">
            <!-- (1) -->
            <sec:intercept-url pattern="/users/{userName}.*"  access="isAuthenticated() and #userName == principal.username"/>
            <!-- (2) -->
            <sec:intercept-url pattern="/users/{userName}/**" access="isAuthenticated() and #userName == principal.username"/>
            <!-- omitted -->
        </sec:http>
    
      .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
      .. list-table::
        :header-rows: 1
        :widths: 10 90
        :class: longtable
    
        * - 項番
          - 説明
        * - | (1)
          - | 「拡張子を付けたパスに対するアクセスポリシー」を定義する。
        * - | (2)
          - | 「拡張子を付けないパスに対するアクセスポリシー」を定義する。
            | ワイルドカードを使用して\ ``/users/{userName}``\ で始まるパスに対するアクセスポリシーを定義する。

|

* ワイルドカードを使用しない場合

  .. tabs::
    .. group-tab:: Java Config

      * SpringSecurityConfig.javaの定義例

      .. code-block:: java

        @Bean
        public SecurityFilterChain filterChain(HttpSecurity http) {
            // omitted
            http.authorizeHttpRequests(authz -> authz
                    .requestMatchers(new AntPathRequestMatcher("/users/{userName}.*"))
                            .access(new WebExpressionAuthorizationManager("isAuthenticated() and #userName == principal.username")) // (1)
                    .requestMatchers(new AntPathRequestMatcher("/users/{userName}/"))
                            .access(new WebExpressionAuthorizationManager("isAuthenticated() and #userName == principal.username")) // (2)
                    .requestMatchers(new AntPathRequestMatcher("/users/{userName}"))
                            .access(new WebExpressionAuthorizationManager("isAuthenticated() and #userName == principal.username")) // (2)
                    // omitted
                    );
            // omitted
            return http.build();
        }
    
      .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
      .. list-table::
        :header-rows: 1
        :widths: 10 90
        :class: longtable
    
        * - 項番
          - 説明
        * - | (1)
          - | 「拡張子を付けたパスに対するアクセスポリシー」を定義する。
        * - | (2)
          - | 「拡張子を付けないパスに対するアクセスポリシー」を定義する。
            | ワイルドカードを使用しない場合、Spring MVCとSpring Securityのパスマッチングの差を吸収するために
            | 末尾が"\ ``/``\ "で終わるパスに対するアクセスポリシーも定義する。

    .. group-tab:: XML Config

      * spring-security.xmlの定義例

      .. code-block:: xml
    
        <sec:http request-matcher="ant">
            <!-- (1) -->
            <sec:intercept-url pattern="/users/{userName}.*" access="isAuthenticated() and #userName == principal.username"/>
            <!-- (2) -->
            <sec:intercept-url pattern="/users/{userName}/"  access="isAuthenticated() and #userName == principal.username"/>
            <sec:intercept-url pattern="/users/{userName}"   access="isAuthenticated() and #userName == principal.username"/>
            <!-- omitted -->
        </sec:http>
    
      .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
      .. list-table::
        :header-rows: 1
        :widths: 10 90
        :class: longtable
    
        * - 項番
          - 説明
        * - | (1)
          - | 「拡張子を付けたパスに対するアクセスポリシー」を定義する。
        * - | (2)
          - | 「拡張子を付けないパスに対するアクセスポリシー」を定義する。
            | ワイルドカードを使用しない場合、Spring MVCとSpring Securityのパスマッチングの差を吸収するために
            | 末尾が"\ ``/``\ "で終わるパスに対するアクセスポリシーも定義する。

.. [#fPathVariableDescription] パス変数の説明は\ :doc:`../ImplementationAtEachLayer/ApplicationLayer`\ の\ :ref:`controller_method_argument-pathvariable-label`\ を参照されたい。

|

.. _AuthorizationToMethod:

メソッドへの認可
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityは、Spring AOPの仕組みを利用してDIコンテナで管理しているBeanのメソッド呼び出しに対して認可処理を行う。

| メソッドに対する認可処理は、ドメイン層(サービス層)のメソッド呼び出しに対して行うことを想定して提供されている。
| メソッドに対する認可処理を使用すると、ドメインオブジェクトのプロパティを参照することができるため、きめの細かいアクセスポリシーの定義を行うことが可能になる。

|

AOPの有効化
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| メソッドへの認可処理を使用する場合は、メソッド呼び出しに対して認可処理を行うためのコンポーネント(AOP)を有効化する必要がある。
| AOPを有効化すると、アクセスポリシーをメソッドのアノテーションに定義できるようになる。

Spring Securityは、以下のアノテーションをサポートしている。

* \ ``@PreAuthorize``\ 、\ ``@PostAuthorize``\ 、\ ``@PreFilter``\ 、\ ``@PostFilter``\
* JSR-250 (\ ``jakarta.annotation.security``\ パッケージ)のアノテーション(\ ``@RolesAllowed``\ など)
* \ ``@Secured``\

本ガイドラインでは、アクセスポリシーをExpressionで使用することができる\ ``@PreAuthorize``\ 、\ ``@PostAuthorize``\ を使用する方法を説明する。

.. tabs::
  .. group-tab:: Java Config

    * SpringSecurityConfig.javaの定義例
    
    .. code-block:: java

      @Configuration
      @EnableMethodSecurity // (1)
      public class SpringSecurityConfig {
    
    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90
    
      * - 項番
        - 説明
      * - | (1)
        - | \ ``@EnableMethodSecurity``\ アノテーションをコンフィグレーションクラスに付与すると、メソッド呼び出しに対する認可処理を行うAOPが有効になる。
          | なお、\ ``prePostEnabled``\ プロパティはデフォルトで\ ``true``\ となっており、Expressionを指定してアクセスポリシーを定義できるアノテーションが有効となっている。

  .. group-tab:: XML Config

    * spring-security.xmlの定義例
    
    .. code-block:: xml
    
      <sec:method-security /> <!-- (1) -->
    
    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90
    
      * - 項番
        - 説明
      * - | (1)
        - | \ ``<sec:method-security>``\ タグを付与すると、メソッド呼び出しに対する認可処理を行うAOPが有効になる。
          | なお、\ ``pre-post-annotations``\ 属性はデフォルトで\ ``true``\ となっており、Expressionを指定してアクセスポリシーを定義できるアノテーションが有効となっている。

|

認可処理の適用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

メソッドに対して認可処理を適用する際は、アクセスポリシーを指定するアノテーションを使用して、メソッド毎にアクセスポリシーを定義する。

|

アクセスポリシーの定義
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

メソッド実行前に適用するアクセスポリシーの指定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

メソッドの実行前に適用するアクセスポリシーを指定する場合は、\ ``@PreAuthorize``\ を使用する。

| \ ``@PreAuthorize``\ の\ ``value``\ 属性に指定したExpressionの結果が\ ``true``\ になるとメソッドの実行が許可される。
| 下記例では、管理者以外は、他人のアカウント情報にアクセスできないように定義している。

* \ ``@PreAuthorize``\ の定義例

.. code-block:: java

  // (1) (2)
  @PreAuthorize("hasRole('ADMIN') or (#username == principal.username)")
  public Account findOne(String username) {
      return accountRepository.findByUsername(username);
  }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | 認可処理を適用したいメソッドに、\ ``@PreAuthorize``\ を付与する。
  * - | (2)
    - | \ ``value``\ 属性に、メソッドに対してアクセスポリシーを定義する。
      | ここでは、「管理者の場合は全てのアカウントへのアクセスを許可する」「管理者以外の場合は自身のアカウントへのアクセスのみ許可する」というアクセスポリシーを定義している。

| ここでポイントになるのは、Expressionの中からメソッドの引数にアクセスしている部分である。
| 具体的には、「\ ``#username``\ 」の部分が引数にアクセスしている部分である。
| Expression内で「# + 引数名」形式のExpressionを指定することで、メソッドの引数にアクセスすることができる。

.. tip:: \ **引数名を指定するアノテーション**\

  Spring Securityは、クラスに出力されているデバッグ情報から引数名を解決する仕組みになっているが、アノテーション(\ ``@org.springframework.security.core.parameters.P``\ )を使用して明示的に引数名を指定することもできる。

  以下のケースにあてはまる場合は、アノテーションを使用して明示的に変数名を指定する。

  * クラスに変数のデバッグ情報を出力しない
  * Expressionの中から実際の変数名とは別の名前を使ってアクセスしたい (例えば短縮した名前)

    .. code-block:: java

      @PreAuthorize("hasRole('ADMIN') or (#username == principal.username)")
      public Account findOne(@P("username") String username) {
          return accountRepository.findByUsername(username);
      }

  なお、\ ``#username``\ と、メソッドの引数である \ ``username``\ の名称が一致している場合は \ ``@P``\ を省略することが可能である。

  ただし、Spring Securityは引数名の解決を、実装クラスの引数名を使用して行っているため\ ``@PreAuthorize``\ アノテーションをインターフェースに定義している場合には、\ **実装クラスの引数名を、 @PreAuthorize 内で指定した #username と一致させる必要がある**\ ので、注意されたい。

  JDK 8 から追加されたコンパイルオプション(\ ``-parameters``\ )を使用すると、メソッドパラメータにリフレクション用のメタデータが生成されるため、アノテーションを指定しなくても引数名が解決される。

.. warning::

  Spring 5から、SpringのコアAPIに\ `null-safety <https://docs.spring.io/spring-framework/docs/6.1.3/reference/html/core.html#null-safety>`_\ の機能が取り入れられており、SpELが解釈される際の\ ``null``\ に対する動作も変更(\ `SPR-15540 <https://jira.spring.io/browse/SPR-15540?redirect=false>`_\ )されている。

  例えば\ ``@PreAuthorize``\ の引数(\ ``#xxx``\ )や、\ ``@PostAuthorize``\ の戻り値（\ ``resultObject``\ ）が\ ``Map``\ を含む場合、\ ``Map``\ から値を取得するSpELでキー値に\ ``null``\ となる値を入力すると、Spring 4以前ではそのまま\ ``Map``\ に\ ``null``\ が渡され該当する値がないため\ ``null``\ が返却されていたが、Spring 5以降ではキーとなるSpELを評価した結果に対する\ ``null``\ チェックが追加されており、\ ``null``\ の場合は\ ``IllegalStateException``\ が発生する。

  そのため、キーとする値に対して事前に\ ``null``\ チェックを行うなど、\ ``null``\ を考慮した実装が必要となる。

|

メソッド実行後に適用するアクセスポリシーの指定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

メソッドの実行後に適用するアクセスポリシーを指定する場合は、\ ``@PostAuthorize``\ を使用する。

| \ ``@PostAuthorize``\ の\ ``value``\ 属性に指定したExpressionの結果が\ ``true``\ になるとメソッドの実行結果が呼び出し元に返却される。
| 下記例では、所属する部署が違うユーザーのアカウント情報にアクセスできないように定義している。

* \ ``@PostAuthorize``\ の定義例

.. code-block:: java

  @PreAuthorize("...")
  @PostAuthorize("(returnObject == null) " +
          "or (returnObject.departmentCode == principal.account.departmentCode)")
  public Account findOne(String username) {
      return accountRepository.findByUsername(username);
  }

| ここでポイントになるのは、Expressionの中からメソッドの返り値にアクセスしている部分である。
| 具体的には、「\ ``returnObject.departmentCode``\ 」の部分が返り値にアクセスしている部分である。
| Expression内で「\ ``returnObject``\ 」を指定すると、メソッドの返り値にアクセスすることができる。

|

画面項目への認可
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| Spring Securityは、JSPタグライブラリを使用してJSPの画面項目に対して認可処理を適用することができる。
| また、Spring Security Dialectは、Spring Securityが提供するJSPタグライブラリと同等の認可処理をThymeleafに適用することができる。

ここでは最もシンプルな定義を例に、画面項目のアクセスに対して認可処理を適用する方法について説明する。

|

アクセスポリシーの定義
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. tabs::
  .. group-tab:: JSP

    JSPタグライブラリを使用してJSPの画面項目に対してアクセスポリシーを定義する際は、表示を許可する条件(アクセスポリシー)をJSPに定義する。

    * アクセスポリシー定義例

    .. code-block:: jsp

      <%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags" %>

      <!-- (1) -->
      <sec:authorize access="hasRole('ADMIN')"> <!-- (2) -->
          <h2>Admin Menu</h2>
          <!-- omitted -->
      </sec:authorize>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - 項番
        - 説明
      * - | (1)
        - | アクセスポリシーを適用したい部分を\ ``<sec:authorize>``\ タグで囲む。
      * - | (2)
        - | \ ``access``\ 属性にアクセスポリシーを定義する。ここでは、「管理者の場合は表示を許可する」というアクセスポリシーを定義している。

  .. group-tab:: Thymeleaf

    Spring Security Dialectを使用して画面項目に対してアクセスポリシーを定義する際は、表示を許可する条件(アクセスポリシー)をHTMLに定義する。

    * アクセスポリシー定義例

    .. code-block:: html

      <html xmlns:th="http://www.thymeleaf.org"
          xmlns:sec="http://www.thymeleaf.org/extras/spring-security">

      <!--/* (1) */-->
      <div sec:authorize="hasRole('ADMIN')"> <!--/* (2) */-->
          <h2>Admin Menu</h2>
          <!--/* omitted */-->
      </div>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - 項番
        - 説明
      * - | (1)
        - | アクセスポリシーを適用したい部分を\ ``sec:authorize``\ 属性を記述したタグで囲む。
      * - | (2)
        - | 属性値にアクセスポリシーを定義する。ここでは、「管理者の場合は表示を許可する」というアクセスポリシーを定義している。

|

Webリソースに指定したアクセスポリシーとの連動
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. tabs::
  .. group-tab:: JSP

    | ボタンやリンクなど(サーバーへのリクエストを伴う画面項目)に対してアクセスポリシーを定義する際は、リクエスト先のWebリソースに定義されているアクセスポリシーと連動させる。
    | Webリソースに指定したアクセスポリシーと連動させる場合は、\ ``<sec:authorize>``\ タグの\ ``url``\ 属性を使用する。

    \ ``url``\ 属性に指定したWebリソースにアクセスできる場合に限り\ ``<sec:authorize>``\ タグの中に実装したJSPの処理が実行される。

    * Webリソースに定義されているアクセスポリシーとの連携例

    .. code-block:: jsp

      <ul>
          <!-- (1) -->
          <sec:authorize url="/admin/accounts"> <!-- (2) -->
              <li>
                  <a href="<c:url value='/admin/accounts' />">Account Management</a>
              </li>
          </sec:authorize>
      </ul>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - 項番
        - 説明
      * - | (1)
        - | ボタンやリンクを出力する部分を\ ``<sec:authorize>``\ タグで囲む。
      * - | (2)
        - | \ ``<sec:authorize>``\ タグの\ ``url``\ 属性にWebリソースへアクセスするためのURLを指定する。
          | ここでは、「\ ``/admin/accounts``\ というURLが割り振られているWebリソースにアクセス可能な場合は表示を許可する」というアクセスポリシーを定義しており、Webリソースに定義されているアクセスポリシーを直接意識する必要がない。

    .. note:: \ **HTTPメソッドによるポリシーの指定**\

      Webリソースのアクセスポリシーの定義をする際に、HTTPメソッドによって異なるアクセスポリシーを指定している場合は、\ ``<sec:authorize>``\ タグの\ ``method``\ 属性を指定して、連動させる定義を特定すること。

    .. warning:: \ **表示制御に関する留意点**\

      ボタンやリンクなどの表示制御を行う場合は、必ずWebリソースに定義されているアクセスポリシーと連動させること。

      ボタンやリンクに対して直接アクセスポリシーの指定を行い、Webリソース自体にアクセスポリシーを定義していないと、URLを直接してアクセスするような不正なアクセスを防ぐことができない。

  .. group-tab:: Thymeleaf

    | ボタンやリンクなど(サーバーへのリクエストを伴う画面項目)に対してアクセスポリシーを定義する際は、リクエスト先のWebリソースに定義されているアクセスポリシーと連動させる。
    | Webリソースに指定したアクセスポリシーと連動させる場合は、\ ``sec:authorize-url``\ 属性を使用する。

    \ ``sec:authorize-url``\ 属性に指定したWebリソースにアクセスできる場合に限り\ ``sec:authorize-url``\ 属性を付与したタグの中に実装したThymeleafの処理が実行される。

    * Webリソースに定義されているアクセスポリシーとの連携例

    .. code-block:: html

      <ul>
          <!--/* (1) */-->
          <li sec:authorize-url="/admin/accounts"> <!--/* (2) */-->
              <a th:href="@{/admin/accounts}">Account Management</a>
          </li>
      </ul>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - 項番
        - 説明
      * - | (1)
        - | ボタンやリンクを出力する部分を\ ``sec:authorize-url``\ 属性を記述したタグで囲む。
      * - | (2)
        - | \ ``sec:authorize-url``\ 属性にWebリソースへアクセスするためのURLを指定する。
          | ここでは、「\ ``/admin/accounts``\ というURLが割り振られているWebリソースにアクセス可能な場合は表示を許可する」というアクセスポリシーを定義しており、Webリソースに定義されているアクセスポリシーを直接意識する必要がない。

    .. note:: \ **HTTPメソッドによるポリシーの指定**\

      Webリソースのアクセスポリシーの定義をする際に、HTTPメソッドによって異なるアクセスポリシーを指定している場合は、\ ``sec:authorize-url``\ 属性の前半にmethodを指定して、スペースで区切りURLを記載することによって連動させる定義を特定すること。

    .. warning:: \ **表示制御に関する留意点**\

      ボタンやリンクなどの表示制御を行う場合は、必ずWebリソースに定義されているアクセスポリシーと連動させること。

      ボタンやリンクに対して直接アクセスポリシーの指定を行い、Webリソース自体にアクセスポリシーを定義していないと、URLを直接してアクセスするような不正なアクセスを防ぐことができない。

    |

    .. tip:: \ **#authorizationの紹介**\

        ここでは、\ ``sec:authorize``\ 属性や\ ``sec:authorize-url``\ 属性を用いて、画面項目に対してアクセスポリシーを定義する実装例を説明したが、
        \ ``#authorization``\ を用いても、ThymeleafのテンプレートHTMLから認可情報にアクセスする事が可能である。
        \ ``#authorization``\ は、変数式 ``${}`` にて使用できるため、条件判定やリテラル置換等\ ``sec:authorize``\ 属性や\ ``sec:authorize-url``\ 属性より複雑な使い方が可能である。

        上記の例は、以下のように記述できる

          .. code-block:: HTML

            <html xmlns:th="http://www.thymeleaf.org" xmlns:sec="http://www.thymeleaf.org/extras/spring-security"><!--/* (1) */-->
            <!--/* omitted */-->

            <div th:if="${#authorization.expr('isAuthenticated()')}"> <!--/* (2) */-->
                <!--/* omitted */-->
            </div>

            <div th:if="${#authorization.url('/admin/accounts')}"> <!--/* (3) */-->
                <!--/* omitted */-->
            </div>

        .. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
        .. list-table::
            :header-rows: 1
            :widths: 25 75

            * - 項番
              - 説明
            * - | (1)
              - | \ ``sec:authorize``\ 属性や\ ``sec:authorize-url``\ 属性を使用する際には\ ``<html>``\ タグに\ ``xmlns:sec``\ 属性を定義していたが、
                | \ ``#authorization``\ を使用する際には、\ ``xmlns:sec``\ 属性の定義は不要である。
            * - | (2)
              - | \ ``#authorization.expr``\ の引数には、\ ``sec:authorize``\ 属性と同様にアクセスポリシーを指定する。
            * - | (3)
              - | \ ``#authorization.url``\ の引数には、\ ``sec:authorize-url``\ 属性と同様にWebリソースへアクセスするためのURLを指定する。

|

認可処理の判定結果を変数に格納
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

JSPにおいて、\ ``<sec:authorize>``\ タグを使って呼び出した認可処理の判定結果は、変数に格納して使いまわすことができる。

.. tabs::
  .. group-tab:: JSP

    * JSPの実装例

    .. code-block:: jsp

      <sec:authorize url="/admin/accounts"
                      var="hasAccountsAuthority"/> <!-- (1) -->

      <c:if test="${hasAccountsAuthority}"> <!-- (2) -->
          <!-- omitted -->
      </c:if>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - 項番
        - 説明
      * - |  (1)
        - | \ ``var``\ 属性に判定結果を格納するための変数名を指定する。
          | アクセスが許可された場合は、変数に\ ``true``\ が設定される。
      * - | (2)
        - | 変数の値を参照して表示処理を実装する。

|

.. _AuthorizationErrorResponse:

認可エラー時のレスポンス
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityは、リソースへのアクセスを拒否した場合、以下のような流れでエラーをハンドリングしてレスポンスの制御を行う。

.. figure:: ./images_Authorization/AuthorizationAccessDeniedHandling.png
  :width: 100%

  \ **認可エラーのハンドリングの仕組み**\

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | Spring Securityは、リソースやメソッドへのアクセスを拒否するために、\ ``AccessDeniedException``\ を発生させる。
  * - | (2)
    - | \ ``ExceptionTranslationFilter``\ クラスは、\ ``AccessDeniedException``\ をキャッチし、\ ``AccessDeniedHandler``\ または\ ``AuthenticationEntryPoint``\ インタフェースのメソッドを呼び出してエラー応答を行う。
  * - | (3)
    - | 認証済みのユーザーからのアクセスの場合は、\ ``AccessDeniedHandler``\ インタフェースのメソッドを呼び出してエラー応答を行う。
  * - | (4)
    - | 未認証のユーザーからのアクセスの場合は、\ ``AuthenticationEntryPoint``\ インタフェースのメソッドを呼び出してエラー応答を行う。

|

AccessDeniedHandler
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| \ ``AccessDeniedHandler``\ インタフェースは、認証済みのユーザーからのアクセスを拒否した際のエラー応答を行うためのインタフェースである。
| Spring Securityは、\ ``AccessDeniedHandler``\ インタフェースの実装クラスとして以下のクラスを提供している。

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: \ **Spring Securityが提供するAccessDeniedHandlerの実装クラス**\
  :header-rows: 1
  :widths: 25 75

  * - クラス名
    - 説明
  * - | \ ``AccessDeniedHandlerImpl``\
    - | HTTPレスポンスコードに403(Forbidden)を設定し、指定されたエラーページに遷移する。
      | エラーページの指定がない場合は、HTTPレスポンスコードに403(Forbidden)を設定してエラー応答(\ ``HttpServletResponse#sendError``\ )を行う。
  * - | \ ``InvalidSessionAccessDeniedHandler``\
    - | \ ``InvalidSessionStrategy``\ インタフェースの実装クラスに処理を委譲する。
      | このクラスは、CSRF対策とセッション管理機能を使用してセッションタイムアウトを検知する設定を有効にした際に、CSRFトークンがセッションに存在しない(つまりセッションタイムアウトが発生している)場合に使用される。
  * - | \ ``DelegatingAccessDeniedHandler``\
    - | \ ``AccessDeniedException``\ と\ ``AccessDeniedHandler``\ インタフェースの実装クラスのマッピングを行い、発生した\ ``AccessDeniedException``\ に対応する\ ``AccessDeniedHandler``\ インタフェースの実装クラスに処理を委譲する。
      | \ ``InvalidSessionAccessDeniedHandler``\ はこの仕組みを利用して呼び出されている。
  * - | \ ``RequestMatcherDelegatingAccessDeniedHandler``\
    - \ ``RequestMatcher``\ インタフェースの仕組みを利用して、指定されたリクエストのパターンに対応する\ ``AccessDeniedHandler``\ インタフェースの実装クラスに処理を委譲する。

      .. note::

        \ ``RequestMatcherDelegatingAccessDeniedHandler``\ の設定方法については、\ :ref:`LinkageWithBrowserEachRequestPattern`\ の\ ``DelegatingRequestMatcherHeaderWriter``\ と同様にリクエストパターンの判定を行う\ ``RequestMatcher``\ と処理を委譲する\ ``AccessDeniedHandler``\ を設定すれば良い。

        なお、\ ``<sec:intercept-url>``\ と\ ``RequestMatcherDelegatingAccessDeniedHandler``\ がパスマッチングを行う間にはリクエストのパスが変わる可能性がある処理が挟まれないため、Warning「指定したパスが意図した通りに認識されない問題」に記載されているような事象は発生しない。

Spring Securityのデフォルトの設定では、エラーページの指定がない\ ``AccessDeniedHandlerImpl``\ が使用される。

|

AuthenticationEntryPoint
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| \ ``AuthenticationEntryPoint``\ インタフェースは、未認証のユーザーからのアクセスを拒否した際のエラー応答を行うためのインタフェースである。
| Spring Securityは、\ ``AuthenticationEntryPoint``\ インタフェースの実装クラスとして以下のクラスを提供している。

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: \ **Spring Securityが提供する主なAuthenticationEntryPointの実装クラス**\
  :header-rows: 1
  :widths: 25 75

  * - クラス名
    - 説明
  * - | \ ``LoginUrlAuthenticationEntryPoint``\
    - | フォーム認証用のログインフォームを表示する。
  * - | \ ``BasicAuthenticationEntryPoint``\
    - | Basic認証用のエラー応答を行う。
      | 具体的には、HTTPレスポンスコードに401(Unauthorized)を、レスポンスヘッダとしてBasic認証用の「\ ``WWW-Authenticate``\ 」ヘッダを設定してエラー応答(\ ``HttpServletResponse#sendError``\ )を行う。
  * - | \ ``DigestAuthenticationEntryPoint``\
    - | Digest認証用のエラー応答を行う。
      | 具体的には、HTTPレスポンスコードに401(Unauthorized)を、レスポンスヘッダとしてDigest認証用の「\ ``WWW-Authenticate``\ 」ヘッダを設定してエラー応答(\ ``HttpServletResponse#sendError``\ )を行う。
  * - | \ ``Http403ForbiddenEntryPoint``\
    - | HTTPレスポンスコードに403(Forbidden)を設定してエラー応答(\ ``HttpServletResponse#sendError``\ )を行う。
  * - | \ ``HttpStatusEntryPoint``\
    - | 任意のHTTPレスポンスコードを設定して正常応答(\ ``HttpServletResponse#setStatus``\ )を行う。
  * - | \ ``DelegatingAuthenticationEntryPoint``\
    - | \ ``RequestMatcher``\ インタフェースの仕組みを利用して、指定されたリクエストのパターンに対応する\ ``AuthenticationEntryPoint``\ インタフェースの実装クラスに処理を委譲する。

Spring Securityのデフォルトの設定では、認証方式に対応する\ ``AuthenticationEntryPoint``\ インタフェースの実装クラスが使用される。

|

.. _SpringSecurityAuthorizationOnError:

認可エラー時の遷移先
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Spring Securityのデフォルトの設定だと、認証済みのユーザーからのアクセスを拒否した際は、アプリケーションサーバのエラーページが表示される。
| アプリケーションサーバーのエラーページを表示してしまうと、システムのセキュリティを低下させる要因になるため、適切なエラー画面を表示することを推奨する。
| エラーページの指定は、以下のようなbean定義を行うことで可能である。

.. tabs::
  .. group-tab:: Java Config

    .. tabs::
      .. group-tab:: JSP
    
        * SpringSecurityConfig.javaの定義例
    
        .. code-block:: java

          @Bean("accessDeniedHandler")
          public AccessDeniedHandler accessDeniedHandler() {
              LinkedHashMap<Class<? extends AccessDeniedException>, AccessDeniedHandler> errorHandlers = new LinkedHashMap<>();
      
              // omitted

              AccessDeniedHandlerImpl defaultErrorHandler = new AccessDeniedHandlerImpl();
              defaultErrorHandler.setErrorPage("/WEB-INF/views/common/error/accessDeniedError.jsp"); // (1)
      
              return new DelegatingAccessDeniedHandler(errorHandlers, defaultErrorHandler);
          }

        .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
        .. list-table::
          :header-rows: 1
          :widths: 10 90
    
          * - 項番
            - 説明
          * - | (1)
            - | \ ``AccessDeniedHandler``\ をBean定義し、defaultHandlerに認可エラー用のエラーページを指定する。
    
      .. group-tab:: Thymeleaf
    
        * SpringSecurityConfig.javaの定義例
    
        .. code-block:: java

          @Bean("accessDeniedHandler")
          public AccessDeniedHandler accessDeniedHandler() {
              LinkedHashMap<Class<? extends AccessDeniedException>, AccessDeniedHandler> errorHandlers = new LinkedHashMap<>();
      
              // omitted

              AccessDeniedHandlerImpl defaultErrorHandler = new AccessDeniedHandlerImpl();
              defaultErrorHandler.setErrorPage("/common/error/accessDeniedError"); // (1)
      
              return new DelegatingAccessDeniedHandler(errorHandlers, defaultErrorHandler);
          }

        .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
        .. list-table::
          :header-rows: 1
          :widths: 10 90
    
          * - 項番
            - 説明
          * - | (1)
            - | \ ``AccessDeniedHandler``\ をBean定義し、defaultHandlerに認可エラー用のエラーページを指定する。

  .. group-tab:: XML Config

    .. tabs::
      .. group-tab:: JSP
    
        * spring-security.xmlの定義例
    
        .. code-block:: xml
    
          <sec:http request-matcher="ant">
              <!-- omitted -->
              <sec:access-denied-handler
                  error-page="/WEB-INF/views/common/error/accessDeniedError.jsp" /> <!-- (1) -->
              <!-- omitted -->
          </sec:http>
    
        .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
        .. list-table::
          :header-rows: 1
          :widths: 10 90
    
          * - 項番
            - 説明
          * - | (1)
            - | \ ``<sec:access-denied-handler>``\ タグの\ ``error-page``\ 属性に認可エラー用のエラーページを指定する。
    
      .. group-tab:: Thymeleaf
    
        * spring-security.xmlの定義例
    
        .. code-block:: xml
    
          <sec:http request-matcher="ant">
              <!-- omitted -->
              <sec:access-denied-handler
                  error-page="/common/error/accessDeniedError" /> <!-- (1) -->
              <!-- omitted -->
          </sec:http>
    
        .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
        .. list-table::
          :header-rows: 1
          :widths: 10 90
    
          * - 項番
            - 説明
          * - | (1)
            - | \ ``<sec:access-denied-handler>``\ タグの\ ``error-page``\ 属性に認可エラー用のエラーページを指定する。

.. tip:: \ **サーブレットコンテナのエラーページ機能の利用**\

  認可エラーのエラーページは、サーブレットコンテナのエラーページ機能を使って指定することもできる。

  サーブレットコンテナのエラーページ機能を使う場合は、\ ``web.xml``\ の\ \ ``<error-page>``\ タグを使用してエラーページを指定する。

    .. tabs::
      .. group-tab:: JSP

        .. code-block:: xml

          <error-page>
              <error-code>403</error-code>
              <location>/WEB-INF/views/common/error/accessDeniedError.jsp</location>
          </error-page>

      .. group-tab:: Thymeleaf

        .. code-block:: xml

          <error-page>
              <error-code>403</error-code>
              <location>/common/error/accessDeniedError</location>
          </error-page>

|

How to extend
--------------------------------------------------------------------------------

本節では、Spring Securityが用意しているカスタマイズポイントや拡張方法について説明する。

| Spring Securityは、多くのカスタマイズポイントを提供しているため、すべてのカスタマイズポイントは紹介しない。
| 本節では代表的なカスタマイズポイントに絞って説明を行う。

|

認可エラー時のレスポンス (認証済みユーザー編)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ここでは、認証済みユーザーからのアクセスを拒否した際の動作をカスタマイズする方法を説明する。

|

.. _SpringSecurityAuthorizationAccessDeniedHandler:

AccessDeniedHandlerの適用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityが提供しているデフォルトの動作をカスタマイズする仕組みだけでは要件をみたせない場合は、\ ``AccessDeniedHandler``\ インタフェースの実装クラスを直接適用することができる。

| 例えば、Ajaxのリクエスト(REST APIなど)で認可エラーが発生した場合は、エラーページ(HTML)ではなくJSON形式でエラー情報を応答することが求められるケースがある。
| そのような場合は、\ ``AccessDeniedHandler``\ インタフェースの実装クラスを作成してSpring Securityに適用することで実現することができる。

* AccessDeniedHandlerインタフェースの実装クラスの作成例

.. code-block:: java

  public class JsonDelegatingAccessDeniedHandler implements AccessDeniedHandler {

      private final RequestMatcher jsonRequestMatcher;
      private final AccessDeniedHandler delegateHandler;

      public JsonDelegatingAccessDeniedHandler(
      public JsonDelegatingAccessDeniedHandler(
              RequestMatcher jsonRequestMatcher, AccessDeniedHandler delegateHandler) {
          this.jsonRequestMatcher = jsonRequestMatcher;
          this.delegateHandler = delegateHandler;
      }

      public void handle(HttpServletRequest request, HttpServletResponse response,
                        AccessDeniedException accessDeniedException)
             throws IOException, ServletException {
         if (jsonRequestMatcher.matches(request)) {
             // response error information of JSON format
             response.setStatus(HttpServletResponse.SC_FORBIDDEN);
             // omitted
         } else {
             // response error page of HTML format
             delegateHandler.handle(
                     request, response, accessDeniedException);
         }
     }

  }

.. tabs::
  .. group-tab:: Java Config

    .. tabs::
      .. group-tab:: JSP
    
        * SpringSecurityConfig.javaの定義例
    
        .. code-block:: java

          @Bean("accessDeniedHandler")
          public JsonDelegatingAccessDeniedHandler accessDeniedHandler() {
              return new JsonDelegatingAccessDeniedHandler(accessAntPathRequestMatcher(), accessDeniedHandler()); // (1)
          }

          @Bean("accessAntPathRequestMatcher")
          public AntPathRequestMatcher accessAntPathRequestMatcher() {
              return new AntPathRequestMatcher("/api/**");
          }

          @Bean("accessDeniedHandler")
          public AccessDeniedHandler accessDeniedHandler() {
              AccessDeniedHandlerImpl bean = new AccessDeniedHandlerImpl();
              bean.setErrorPage("/WEB-INF/views/common/error/accessDeniedError.jsp");
              return bean;
          }

          @Bean
          public SecurityFilterChain filterChain1(HttpSecurity http) {
              // omitted
              http.exceptionHandling(exceptionHandling -> exceptionHandling
                      .accessDeniedHandler(accessDeniedHandler())); // (2)
              // omitted
              return http.build();
          }
    
        .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
        .. list-table::
          :header-rows: 1
          :widths: 10 90
    
          * - 項番
            - 説明
          * - \ (1)
            - \ ``AccessDeniedHandler``\ インタフェースの実装クラスをbean定義してDIコンテナに登録する。
          * - \ (2)
            - \ ``HttpSecurity#exceptionHandling``\ に\ ``AccessDeniedHandler``\ のbeanを指定する。
    
      .. group-tab:: Thymeleaf
    
        * SpringSecurityConfig.javaの定義例
    
        .. code-block:: java

          @Bean("accessDeniedHandler")
          public JsonDelegatingAccessDeniedHandler accessDeniedHandler() {
              return new JsonDelegatingAccessDeniedHandler(accessAntPathRequestMatcher(), accessDeniedHandler()); // (1)
          }

          @Bean("accessAntPathRequestMatcher")
          public AntPathRequestMatcher accessAntPathRequestMatcher() {
              return new AntPathRequestMatcher("/api/**");
          }

          @Bean("accessDeniedHandler")
          public AccessDeniedHandler accessDeniedHandler() {
              AccessDeniedHandlerImpl bean = new AccessDeniedHandlerImpl();
              bean.setErrorPage("/common/error/accessDeniedError");
              return bean;
          }

          @Bean
          public SecurityFilterChain filterChain1(HttpSecurity http) {
              // omitted
              http.exceptionHandling(exceptionHandling -> exceptionHandling
                      .accessDeniedHandler(accessDeniedHandler())); // (2)
              // omitted
              return http.build();
          }
    
        .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
        .. list-table::
          :header-rows: 1
          :widths: 10 90
    
          * - 項番
            - 説明
          * - \ (1)
            - \ ``AccessDeniedHandler``\ インタフェースの実装クラスをbean定義してDIコンテナに登録する。
          * - \ (2)
            - \ ``HttpSecurity#exceptionHandling``\ に\ ``AccessDeniedHandler``\ のbeanを指定する。

  .. group-tab:: XML Config

    .. tabs::
      .. group-tab:: JSP
    
        * spring-security.xmlの定義例
    
        .. code-block:: xml
    
          <!-- (1) -->
          <bean id="accessDeniedHandler"
                class="com.example.web.security.JsonDelegatingAccessDeniedHandler">
              <constructor-arg>
                  <bean class="org.springframework.security.web.util.matcher.AntPathRequestMatcher">
                      <constructor-arg value="/api/**"/>
                  </bean>
              </constructor-arg>
              <constructor-arg>
                  <bean class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
                      <property name="errorPage"
                                value="/WEB-INF/views/common/error/accessDeniedError.jsp"/>
                  </bean>
              </constructor-arg>
          </bean>
    
          <sec:http request-matcher="ant">
              <!-- omitted -->
              <sec:access-denied-handler ref="accessDeniedHandler" />  <!-- (2) -->
              <!-- omitted -->
          </sec:http>
    
        .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
        .. list-table::
          :header-rows: 1
          :widths: 10 90
    
          * - 項番
            - 説明
          * - \ (1)
            - \ ``AccessDeniedHandler``\ インタフェースの実装クラスをbean定義してDIコンテナに登録する。
          * - \ (2)
            - \ ``<sec:access-denied-handler>``\ タグの\ ``ref``\ 属性に\ ``AccessDeniedHandler``\ のbeanを指定する。
    
      .. group-tab:: Thymeleaf
    
        * spring-security.xmlの定義例
    
        .. code-block:: xml
    
          <!-- (1) -->
          <bean id="accessDeniedHandler"
                class="com.example.web.security.JsonDelegatingAccessDeniedHandler">
              <constructor-arg>
                  <bean class="org.springframework.security.web.util.matcher.AntPathRequestMatcher">
                      <constructor-arg value="/api/**"/>
                  </bean>
              </constructor-arg>
              <constructor-arg>
                  <bean class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
                      <property name="errorPage"
                                value="/common/error/accessDeniedError"/>
                  </bean>
              </constructor-arg>
          </bean>
    
          <sec:http request-matcher="ant">
              <!-- omitted -->
              <sec:access-denied-handler ref="accessDeniedHandler" />  <!-- (2) -->
              <!-- omitted -->
          </sec:http>
    
        .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
        .. list-table::
          :header-rows: 1
          :widths: 10 90
    
          * - 項番
            - 説明
          * - \ (1)
            - \ ``AccessDeniedHandler``\ インタフェースの実装クラスをbean定義してDIコンテナに登録する。
          * - \ (2)
            - \ ``<sec:access-denied-handler>``\ タグの\ ``ref``\ 属性に\ ``AccessDeniedHandler``\ のbeanを指定する。

|

認可エラー時のレスポンス (未認証ユーザー編)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ここでは、未認証ユーザーからのアクセスを拒否した際の動作をカスタマイズする方法を説明する。

|

リクエスト毎にAuthenticationEntryPointを適用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 認証済みユーザーと同様に、Ajaxのリクエスト(REST APIなど)で認可エラーが発生した場合は、ログインページ(HTML)ではなくJSON形式でエラー情報を応答することが求められるケースがある。
| そのような場合は、リクエストのパターン毎に\ ``AuthenticationEntryPoint``\ インタフェースの実装クラスをSpring Securityに適用することで実現することができる。

.. tabs::
  .. group-tab:: Java Config

    * SpringSecurityConfig.javaの定義例
    
    .. code-block:: java

      @Bean("authenticationEntryPoint")
      public DelegatingAuthenticationEntryPoint authenticationEntryPoint() {
          LinkedHashMap<RequestMatcher, AuthenticationEntryPoint> map = new LinkedHashMap<>();
          map.put(antPathRequestMatcher(), entryPoint());
          DelegatingAuthenticationEntryPoint bean = new DelegatingAuthenticationEntryPoint(map); // (1)
          bean.setDefaultEntryPoint(defaultEntryPoint());
          return bean;
      }

      @Bean("antPathRequestMatcher")
      public AntPathRequestMatcher antPathRequestMatcher() {
          return new AntPathRequestMatcher("/api/**");
      }

      @Bean("entryPoint")
      public JsonAuthenticationEntryPoint entryPoint() {
          return new JsonAuthenticationEntryPoint();
      }

      @Bean("defaultEntryPoint")
      public LoginUrlAuthenticationEntryPoint defaultEntryPoint() {
          return new LoginUrlAuthenticationEntryPoint("/login");
      }

      @Bean
      public SecurityFilterChain filterChain(HttpSecurity http) {
          // omitted
          http.exceptionHandling(exceptionHandling -> exceptionHandling
                  .authenticationEntryPoint(authenticationEntryPoint())); // (2)
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
        - | \ ``AuthenticationEntryPoint``\ インタフェースの実装クラスをbean定義してDIコンテナに登録する。
          | ここでは、Spring Securityが提供している\ ``DelegatingAuthenticationEntryPoint``\ クラスを利用して、リクエストのパターン毎に\ ``AuthenticationEntryPoint``\ インタフェースの実装クラスを適用している。
      * - | (2)
        - | \ ``HttpSecurity#exceptionHandling``\ に\ ``AuthenticationEntryPoint``\ のbeanを指定する。

  .. group-tab:: XML Config

    * spring-security.xmlの定義例
    
    .. code-block:: xml
    
      <!-- (1) -->
      <bean id="authenticationEntryPoint"
            class="org.springframework.security.web.authentication.DelegatingAuthenticationEntryPoint">
          <constructor-arg>
              <map>
                  <entry>
                      <key>
                          <bean class="org.springframework.security.web.util.matcher.AntPathRequestMatcher">
                              <constructor-arg value="/api/**"/>
                          </bean>
                      </key>
                      <bean class="com.example.web.security.JsonAuthenticationEntryPoint"/>
                  </entry>
              </map>
          </constructor-arg>
          <property name="defaultEntryPoint">
              <bean class="org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint">
                  <constructor-arg value="/login"/>
              </bean>
          </property>
      </bean>
    
      <sec:http request-matcher="ant" entry-point-ref="authenticationEntryPoint"> <!-- (2) -->
          <!-- omitted -->
      </sec:http>
    
    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90
    
      * - 項番
        - 説明
      * - | (1)
        - | \ ``AuthenticationEntryPoint``\ インタフェースの実装クラスをbean定義してDIコンテナに登録する。
          | ここでは、Spring Securityが提供している\ ``DelegatingAuthenticationEntryPoint``\ クラスを利用して、リクエストのパターン毎に\ ``AuthenticationEntryPoint``\ インタフェースの実装クラスを適用している。
      * - | (2)
        - | \ ``<sec:http>``\ タグの\ ``entry-point-ref``\ 属性に\ ``AuthenticationEntryPoint``\ のbeanを指定する。

.. note:: \ **デフォルトで適用されるAuthenticationEntryPoint**\

  リクエストに対応する\ \ ``AuthenticationEntryPoint``\ インタフェースの実装クラスの指定がない場合は、Spring Securityがデフォルトで定義する\ ``AuthenticationEntryPoint``\ インタフェースの実装クラスが使用される仕組みになっている。

  認証方式としてフォーム認証を使用する場合は、\ ``LoginUrlAuthenticationEntryPoint``\ クラスが使用されログインフォームが表示される。

|

ロールの階層化
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
認可処理では、ロールに階層関係を設けることができる。

| 上位に指定したロールは、下位のロールにアクセスが許可されているリソースにもアクセスすることができる。
| ロールの関係が複雑な場合は、階層関係も設けることも検討されたい。

| 例えば、「ROLE_ADMIN」が上位ロール、「ROLE_USER」が下位ロールという階層関係を設けた場合、下記のようアクセスポリシーを設定すると、「ROLE_ADMIN」権限を持つユーザーは、\ ``/user``\ 配下のパス(「ROLE_USER」権限を持つユーザーがアクセスできるパス)にアクセスすることができる。

.. tabs::
  .. group-tab:: Java Config

    * SpringSecurityConfig.javaの定義例
    
    .. code-block:: java

      @Bean
      public SecurityFilterChain filterChain(HttpSecurity http) {
          // omitted
          http.authorizeHttpRequests(authz -> authz
                  .requestMatchers(new AntPathRequestMatcher("/user/**")).hasAnyRole("USER")
                  // omitted
                  );
          // omitted  
          return http.build();
      }

  .. group-tab:: XML Config

    * spring-security.xmlの定義例
    
    .. code-block:: xml
    
      <sec:http request-matcher="ant">
          <sec:intercept-url pattern="/user/**" access="hasAnyRole('USER')" />
          <!-- omitted -->
      </sec:http>

|

階層関係の設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ロールの階層関係は、\ ``org.springframework.security.access.hierarchicalroles.RoleHierarchy``\ インタフェースの実装クラスで解決する。

.. tabs::
  .. group-tab:: Java Config

    * SpringSecurityConfig.javaの定義例
    
    .. code-block:: java

      @Bean("roleHierarchy")
      public RoleHierarchy roleHierarchy() {
          RoleHierarchyImpl bean = new RoleHierarchyImpl(); // (1)
          bean.setHierarchy("""
                  ROLE_ADMIN > ROLE_STAFF
                  ROLE_STAFF > ROLE_USER
                  """); // (2)
          return bean;
      }
    
    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90
    
      * - 項番
        - 説明
      * - | (1)
        - | \ ``org.springframework.security.access.hierarchicalroles.RoleHierarchyImpl``\ クラスを指定する。
          | \ ``RoleHierarchyImpl``\ は、Spring Securityが提供するデフォルトの実装クラスである。
      * - | (2)
        - | \ ``hierarchy``\ プロパティに階層関係を定義する。
          |
          | 書式: [上位ロール] > [下位ロール]
          |
          | 上記例では、
          | STAFFは、USERに認可されたリソースにもアクセス可能である。
          | ADMINは、USERとSTAFFに認可されたリソースにもアクセス可能である。

  .. group-tab:: XML Config

    * spring-security.xmlの定義例
    
    .. code-block:: xml
    
      <bean id="roleHierarchy"
          class="org.springframework.security.access.hierarchicalroles.RoleHierarchyImpl"> <!-- (1) -->
          <property name="hierarchy"> <!-- (2) -->
              <value>
                  ROLE_ADMIN > ROLE_STAFF
                  ROLE_STAFF > ROLE_USER
              </value>
          </property>
      </bean>
    
    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90
    
      * - 項番
        - 説明
      * - | (1)
        - | \ ``org.springframework.security.access.hierarchicalroles.RoleHierarchyImpl``\ クラスを指定する。
          | \ ``RoleHierarchyImpl``\ は、Spring Securityが提供するデフォルトの実装クラスである。
      * - | (2)
        - | \ ``hierarchy``\ プロパティに階層関係を定義する。
          |
          | 書式: [上位ロール] > [下位ロール]
          |
          | 上記例では、
          | STAFFは、USERに認可されたリソースにもアクセス可能である。
          | ADMINは、USERとSTAFFに認可されたリソースにもアクセス可能である。

|

Webリソースの認可処理への適用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ロールの階層化を、Webリソースと画面項目に対する認可処理に適用する方法を説明する。

.. tabs::
  .. group-tab:: Java Config

    * SpringSecurityConfig.javaの定義例
    
    .. code-block:: java

      @Bean
      @Order(90)
      public SecurityFilterChain filterChain(HttpSecurity http) {
  
          AuthorityAuthorizationManager<RequestAuthorizationContext> authManager = AuthorityAuthorizationManager.hasRole("STAFF");
          authManager.setRoleHierarchy(roleHierarchy()); // (1)
          // omitted
          http.authorizeHttpRequests(authz -> authz
                  .requestMatchers(new AntPathRequestMatcher("/user/**"))
                  .access(authManager) //(2)
                  // omitted
                  );
  
          return http.build();
      }
   
    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90
    
      * - | 項番
        - | 説明
      * - | (1)
        - | \ ``org.springframework.security.authorization.AuthorityAuthorizationManager``\ のインスタンスを生成し、\ ``roleHierarchy``\ プロパティに\ ``RoleHierarchy``\ インタフェースの実装クラスのBeanを指定する。
      * - | (2)
        - | (1)で生成した\ ``AuthorizationManager``\ を\ ``access``\ メソッドで設定する。

  .. group-tab:: XML Config

    * spring-security.xmlの定義例
    
    .. code-block:: xml
    
      <!-- (1) -->
      <bean id="webExpressionHandler"
          class="org.springframework.security.web.access.expression.DefaultWebSecurityExpressionHandler">
          <property name="roleHierarchy" ref="roleHierarchy"/>  <!-- (2) -->
      </bean>
    
      <sec:http request-matcher="ant">
          <!-- omitted -->
          <sec:expression-handler ref="webExpressionHandler" />  <!-- (3) -->
      </sec:http>
    
    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90
    
      * - | 項番
        - | 説明
      * - | (1)
        - | \ ``org.springframework.security.web.access.expression.DefaultWebSecurityExpressionHandler``\ のBeanを定義する。
      * - | (2)
        - | \ ``roleHierarchy``\ プロパティに\ ``RoleHierarchy``\ インタフェースの実装クラスのBeanを指定する。
      * - | (3)
        - | \ ``<sec:expression-handler>``\ タグの\ ``ref``\ 属性に、\ ``org.springframework.security.access.expression.SecurityExpressionHandler``\ インタフェースの実装クラスのBeanを指定する。

|

メソッドの認可処理への適用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ロールの階層化を、Javaメソッドに対する認可処理に適用する方法を説明する。

.. tabs::
  .. group-tab:: Java Config

    * SpringSecurityConfig.javaの定義例
    
    .. code-block:: java

      @EnableMethodSecurity // (3)
      @Configuration
      public class SpringSecurityConfig {

          // (3)
          @Bean("methodExpressionHandler")
          public static MethodSecurityExpressionHandler methodExpressionHandler(
                  RoleHierarchy roleHierarchy) {
              DefaultMethodSecurityExpressionHandler bean = new DefaultMethodSecurityExpressionHandler();  // (1)
              bean.setRoleHierarchy(roleHierarchy); // (2) 
              return bean;
          }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90
    
      * - 項番
        - 説明
      * - | (1)
        - | \ ``org.springframework.security.access.expression.method.DefaultMethodSecurityExpressionHandler``\ のBeanをstatic定義する。
      * - | (2)
        - | \ ``roleHierarchy``\ プロパティに\ ``RoleHierarchy``\ インタフェースの実装クラスのBeanを指定する。
      * - | (3)
        - | \ ``@EnableMethodSecurity``\ アノテーションを設定する。

  .. group-tab:: XML Config

    * spring-security.xmlの定義例
    
    .. code-block:: xml
    
      <bean id="methodExpressionHandler"
          class="org.springframework.security.access.expression.method.DefaultMethodSecurityExpressionHandler"> <!-- (1) -->
          <property name="roleHierarchy" ref="roleHierarchy"/> <!-- (2) -->
      </bean>

      <sec:method-security>
        <sec:expression-handler ref="methodExpressionHandler" /> <!-- (3) -->
      </sec:method-security>
    
    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90
    
      * - 項番
        - 説明
      * - | (1)
        - | \ ``org.springframework.security.access.expression.method.DefaultMethodSecurityExpressionHandler``\ のBeanを定義する。
      * - | (2)
        - | \ ``roleHierarchy``\ プロパティに\ ``RoleHierarchy``\ インタフェースの実装クラスのBeanを指定する。
      * - | (3)
        - | \ ``<sec:method-security>``\ タグの\ ``ref``\ 属性に、\ ``org.springframework.security.access.expression.SecurityExpressionHandler``\ インタフェースの実装クラスのBeanを指定する。

|

Appendix
--------------------------------------------------------------------------------

.. _Authoziation_expansion_of_path_pattern_match:

Spring Securityのパスパターンマッチングにおける拡張子および末尾\ ``/``\ の考慮
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. tabs::
  .. group-tab:: Java Config

    | \ :ref:`ApplicationLayer_expansion_of_path_pattern_match`\ に記載している方法でパスパターンマッチを拡張している場合、\ ``RequestMatcher``\ オブジェクトの設定が不足していると認可処理を行わずにアクセス出来てしまう。
    | パスパターンマッチの拡張箇所をカバーできるように、\ ``RequestMatcher``\ の引数に"\ ``*``\ "や\ ``**``\ などのワイルドカード指定を含めている場合は問題とはならないが、そうでない場合は\ ``RequestMatcher``\ の引数に対して個別に考慮する必要がある。
    
    | 下記の設定例は、\ ``/restrict``\ に対して「ROLE_ADMIN」ロールを持つユーザからのアクセスのみを許可している。

    .. code-block:: java

      @Bean
      public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
          // omitted  
          http.authorizeHttpRequests(authz -> authz
                  .requestMatchers(new AntPathRequestMatcher("/restrict.*")).hasRole("ADMIN") // (1)
                  .requestMatchers(new AntPathRequestMatcher("/restrict/")).hasRole("ADMIN") // (2)
                  .requestMatchers(new AntPathRequestMatcher("/restrict")).hasRole("ADMIN") // (3)
          // omitted
          return http.build();
      }
  
    .. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 20 80
      :class: longtable
  
      * - 項番
        - 説明
      * - | (1)
        - | \ ``/restrict``\ に拡張子を付けたパターン(\ ``/restrict.json``\ など)のアクセスポリシーを定義する。
          | 個別に拡張子を許容している場合または\ ``PathMatchConfigurer#setUseSuffixPatternMatch(true)``\ を設定している場合は必須。
      * - | (2)
        - | \ ``/restrict``\ の末尾に"\ ``/``\ "を付けたパターン(\ ``/restrict/``\ )のアクセスポリシーを定義する。
          | 個別に末尾の"\ ``/``\ を許容している場合または\ ``PathMatchConfigurer#setUseSuffixPatternMatch(true)``\ を設定している場合は必須。
      * - | (3)
        - | \ ``/restrict``\ に対するアクセスポリシーを定義する。

  .. group-tab:: XML Config

    | \ :ref:`ApplicationLayer_expansion_of_path_pattern_match`\ に記載している方法でパスパターンマッチを拡張している場合、\ ``sec:intercept-url``\ の\ ``pattern``\ 属性の設定が不足していると認可処理を行わずにアクセス出来てしまう。
    | パスパターンマッチの拡張箇所をカバーできるように、\ ``sec:intercept-url``\ の\ ``pattern``\ 属性に"\ ``*``\ "や\ ``**``\ などのワイルドカード指定を含めている場合は問題とはならないが、そうでない場合は\ ``sec:intercept-url``\ の\ ``pattern``\ 属性に対して個別に考慮する必要がある。
    
    | 下記の設定例は、\ ``/restrict``\ に対して「ROLE_ADMIN」ロールを持つユーザからのアクセスのみを許可している。

    .. code-block:: xml
  
      <sec:http request-matcher="ant">
          <sec:intercept-url pattern="/restrict.*" access="hasRole('ADMIN')" /> <!-- (1) -->
          <sec:intercept-url pattern="/restrict/" access="hasRole('ADMIN')" /> <!-- (2) -->
          <sec:intercept-url pattern="/restrict" access="hasRole('ADMIN')" /> <!-- (3) -->
          <!-- omitted -->
      </sec:http>
  
    .. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 20 80
      :class: longtable
  
      * - 項番
        - 説明
      * - | (1)
        - | \ ``/restrict``\ に拡張子を付けたパターン(\ ``/restrict.json``\ など)のアクセスポリシーを定義する。
          | 個別に拡張子を許容している場合または\ ``suffix-pattern="true"``\ を設定している場合は必須。
      * - | (2)
        - | \ ``/restrict``\ の末尾に"\ ``/``\ "を付けたパターン(\ ``/restrict/``\ )のアクセスポリシーを定義する。
          | 個別に末尾の"\ ``/``\ を許容している場合または\ ``trailing-slash="true"``\ を設定している場合は必須。
      * - | (3)
        - | \ ``/restrict``\ に対するアクセスポリシーを定義する。

.. raw:: latex

  \ newpage
