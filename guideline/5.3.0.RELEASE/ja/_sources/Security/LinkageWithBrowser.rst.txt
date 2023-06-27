.. _SpringSecurityLinkageWithBrowser:

ブラウザのセキュリティ対策機能との連携
================================================================================

.. only:: html

 .. contents:: 目次
    :local:

Overview
--------------------------------------------------------------------------------

本節では、ブラウザが提供しているセキュリティ対策機能との連携方法について説明する。

主要なWebブラウザは、ブラウザが提供する機能が悪用されないようにするために、いくつかのセキュリティ対策機能を提供している。
ブラウザが提供するセキュリティ対策機能の一部は、サーバ側でHTTPのレスポンスヘッダを出力することで動作を制御することができる。

Spring Securityは、セキュリティ関連のレスポンスヘッダを出力する機能を用意することで、Webアプリケーションのセキュリティを強化する仕組みを提供している。

.. note:: **セキュリティリスク**

    セキュリティ関連のレスポンスヘッダを出力しても、セキュリティへのリスクが100%なくなるわけではない。
    あくまで、セキュリティリスクを減らすためのサポート機能と考えておくこと。

    なお、セキュリティヘッダのサポート状況はブラウザによってことなる。

.. note:: **HTTPヘッダの上書き**

    後述の設定を行ったとしても、アプリケーションにより、HTTPヘッダが上書きされる可能性は存在する。

デフォルトでサポートしているセキュリティヘッダ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityがデフォルトでサポートしているレスポンスヘッダは以下の7つである。

* Cache-Control (Pragma, Expires)
* X-Frame-Options
* X-Content-Type-Options
* X-XSS-Protection
* Strict-Transport-Security
* Content-Security-Policy(Content-Security-Policy-Report-Only)
* Public-Key-Pins(Public-Key-Pins-Report-Only)

.. tip:: **ブラウザのサポート状況**

    これらのヘッダに対する処理は、一部のブラウザではサポートされていない。ブラウザの公式サイトまたは以下のページを参照されたい。

    * https://www.owasp.org/index.php/HTTP_Strict_Transport_Security (Strict-Transport-Security)
    * https://www.owasp.org/index.php/Clickjacking_Defense_Cheat_Sheet (X-Frame-Options)
    * https://www.owasp.org/index.php/List_of_useful_HTTP_headers (X-Content-Type-Options, X-XSS-Protection, Content-Security-Policy, Public-Key-Pins)


Cache-Control
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Cache-Controlヘッダは、コンテンツのキャッシュ方法を指示するためのヘッダである。
保護されたコンテンツがブラウザにキャッシュされないようにすることで、権限のないユーザーが保護されたコンテンツを閲覧できてしまうリスクを減らすことができる。

コンテンツがキャッシュされないようにするためには、以下のようなヘッダを出力する。

* レスポンスヘッダの出力例

.. code-block:: text

    Cache-Control: no-cache, no-store, max-age=0, must-revalidate
    Pragma: no-cache
    Expires: 0

.. note:: **Cache-Controlヘッダの上書き**

    Spring MVCのControllerクラスが \ ``@SessionAttributes`` \のフォームクラスを定義している、もしくは、
    リクエストハンドラで \ ``@SessionAttributes`` \属性のModelを使用してる場合は、 Cache-Controlヘッダが上書きされる。

.. note:: **HTTP1.0互換のブラウザ**

    Spring SecurityはHTTP1.0互換のブラウザもサポートするために、PragmaヘッダとExpiresヘッダも出力する。


X-Frame-Options
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

X-Frame-Optionsヘッダは、フレーム(\ ``<frame>``\ または\ ``<iframe>``\ 要素) 内でのコンテンツの表示を許可するか否かを指示するためのヘッダである。
フレーム内でコンテンツが表示されないようすることで、クリックジャッキングと呼ばれる攻撃手法を使って機密情報を盗みとられるリスクをなくすことができる。

フレーム内での表示を拒否するためには、以下のようなヘッダを出力する。

* レスポンスヘッダの出力例(Spring Securityのデフォルト出力)

.. code-block:: text

    X-Frame-Options: DENY

なお、X-Frame-Optionsヘッダには、出力例以外のオプションを指定することができる。

X-Content-Type-Options
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

X-Content-Type-Optionsヘッダは、コンテンツの種類の決定方法を指示するためのヘッダである。
一部のブラウザでは、Content-Typeヘッダの値を無視してコンテンツの内容をみて決定する。
コンテンツの種類の決定する際にコンテンツの内容を見ないようにすることで、クロスサイトスクリプティングを使った攻撃を受けるリスクを減らすことができる。

コンテンツの種類の決定する際にコンテンツの内容を見ないようにするためには、以下のヘッダを出力する。

* レスポンスヘッダの出力例

.. code-block:: text

    X-Content-Type-Options: nosniff


X-XSS-Protection
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

X-XSS-Protectionヘッダは、ブラウザのXSSフィルター機能を使って有害スクリプトを検出する方法を指示するためのヘッダである。
XSSフィルター機能を有効にして有害なスクリプトを検知するとこで、クロスサイトスクリプティングを使った攻撃を受けるリスクを減らすことができる。

XSSフィルター機能を有効にして有害なスクリプトを検知するためには、以下のようなヘッダを出力する。

* レスポンスヘッダの出力例(Spring Securityのデフォルト出力)

.. code-block:: text

    X-XSS-Protection: 1; mode=block

なお、X-XSS-Protectionヘッダには、出力例以外のオプションを指定することができる。

Strict-Transport-Security
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Strict-Transport-Securityヘッダーは、HTTPSを使ってアクセスした後にHTTPを使ってアクセスしようとした際に、HTTPSに置き換えてからアクセスすることを指示するためヘッダである。
HTTPSでアクセスした後にHTTPが使われないようにすることで、中間者攻撃と呼ばれる攻撃手法を使って悪意のあるサイトに誘導されるリスクを減らすことができる。

HTTPSでアクセスした後にHTTPが使われないようにするためには、以下のようなヘッダを出力する。

* レスポンスヘッダの出力例(Spring Securityのデフォルト出力)

.. code-block:: text

    Strict-Transport-Security: max-age=31536000 ; includeSubDomains

.. note:: **Strict-Transport-Security**

    Spring Securityのデフォルト実装では、Strict-Transport-Securityヘッダは、アプリケーションサーバに対してHTTPSを使ってアクセスがあった場合のみ出力される。
    なお、Strict-Transport-Securityヘッダ値は、オプションを指定することで変更することができる。

Content-Security-Policy
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Content-Security-Policyヘッダーはブラウザに読み込みを許可するコンテンツを指示するためのヘッダーである。
ブラウザはContent-Security-Policyヘッダーに指定したホワイトリストのコンテンツのみを読み込むため、悪意のあるコンテンツを読み込むことで実行される攻撃（クロスサイトスクリプティング攻撃など）を受けるリスクを減らすことができる。

Content-Security-Policyヘッダーを送信しない場合、ブラウザは標準の同一オリジンポリシーを適用する。

コンテンツの取得元を同一オリジンのみに制限するためには、以下のようなヘッダーを出力する。

* レスポンスヘッダの出力例

.. code-block:: text

    Content-Security-Policy: default-src 'self'

.. note:: **ポリシー違反時のレポート送信について**

    ポリシー違反時にレポートを送信したい場合、report-uriディレクティブに報告先のURIを指定する。

    同一オリジンポリシー違反があった場合にコンテンツをブロックして\ ``/csp_report``\ にレポートを送信するためには、以下のようなヘッダーを出力する。

    * レスポンスヘッダの出力例

     .. code-block:: text

        Content-Security-Policy: default-src 'self'; report-uri /csp_report;

    また、ポリシー違反があった際に、コンテンツのブロックを行わずレポートの送信のみを行いたい場合はContent-Security-Policy-Report-Onlyヘッダーを使用する。
    Content-Security-Policy-Report-Onlyヘッダーを使用してレポートを収集しながら段階的にポリシーとコンテンツを修正することで、既にサービス提供しているサイトに対してポリシーを適用した場合に正常に動作しなくなるリスクを減らすことが出来る。

    同一オリジンポリシー違反があった場合にコンテンツをブロックせず\ ``/csp_report``\ にレポートを送信するためには、以下のようなヘッダーを出力する。

    * レスポンスヘッダの出力例

     .. code-block:: text

        Content-Security-Policy-Report-Only: default-src 'self'; report-uri /csp_report;

Public-Key-Pins
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Public-Key-Pinsヘッダはサイトの証明書の真正性を担保するために、サイトに紐付く証明書の公開鍵をブラウザに提示するヘッダである。
サイトへの再訪問時に中間者攻撃と呼ばれる攻撃手法を使って悪意のあるサイトに誘導された場合でも、
ブラウザが保持する真性のサイト証明書の公開鍵と悪意あるサイトが提示する証明書の公開鍵の不一致を検知して、
アクセスをブロックすることができる。

ブラウザが保持する情報と一致しない証明書を検出した場合にアクセスをブロックさせるためには、以下のようなヘッダを出力する。

* レスポンスヘッダの出力例

.. code-block:: text

    Public-Key-Pins: max-age=5184000 ; pin-sha256="d6qzRu9zOECb90Uez27xWltNsj0e1Md7GkYYkVoZWmM=" ; pin-sha256="E9CZ9INDbd+2eRQozYqqbQ2yXLVKB9+xcprMF+44U1g="

.. note:: **違反レポートの送信について**

    アクセスブロック時にブラウザに違反レポートを送信させるためには、Content-Security-Policyと同様にreport-uriディレクティブを指定する。

    また、ブラウザにアクセスをブロックさせずに違反レポートを送信させるためには、Public-Key-Pinsヘッダの代わりにPublic-Key-Pins-Report-Onlyヘッダを使用する。

.. note:: **Public-Key-Pinsヘッダの設定について**

    Public-Key-Pinsヘッダの設定に誤りがあった場合、ユーザが長期間サイトにアクセスできなくなる可能性があるため、
    Public-Key-Pins-Report-Onlyヘッダで十分に試験を実施した上でPublic-Key-Pinsヘッダに切り替えることを推奨する。

How to use
--------------------------------------------------------------------------------

セキュリティヘッダ出力機能の適用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

前述のセキュリティヘッダ出力機能を適用する方法を説明する。

セキュリティヘッダ出力機能は、Spring 3.2から追加された機能で以下のセキュリティヘッダ以外はデフォルトで適用されるようになっている。 

* Content-Security-Policy
* Public-Key-Pins

そのため、デフォルトで適用されるセキュリティヘッダ出力機能を有効にするための特別な定義は不要である。 
なお、デフォルトで適用されるセキュリティヘッダ出力機能を適用したくない場合は、明示的に無効化する必要がある。 

セキュリティヘッダ出力機能を無効化する場合は、以下のようなbean定義を行う。

* spring-security.xmlの定義例

.. code-block:: xml

    <sec:http>
        <!-- omitted -->
        <sec:headers disabled="true"/> <!-- disabled属性にtrueを設定して無効化 -->
        <!-- omitted -->
    </sec:http>


セキュリティヘッダの選択
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

出力するセキュリティヘッダを選択したい場合は、以下のようなbean定義を行う。
ここではSpring Securityが提供しているすべてのセキュリティヘッダを出力する例になっているが、実際には必要なものだけ指定すること。

* spring-security.xmlの定義例

.. code-block:: xml

    <sec:headers defaults-disabled="true"> <!-- (1) -->
        <sec:cache-control/> <!-- (2) -->
        <sec:frame-options/> <!-- (3) -->
        <sec:content-type-options/> <!-- (4) -->
        <sec:xss-protection/> <!-- (5) -->
        <sec:hsts/> <!-- (6) -->
        <sec:content-security-policy policy-directives="default-src 'self'" /> <!-- (7) -->
        <sec:hpkp report-uri="https://www.example.net/hpkp-report"> <!-- (8) -->
            <sec:pins>
                <sec:pin algorithm="sha256">d6qzRu9zOECb90Uez27xWltNsj0e1Md7GkYYkVoZWmM=</sec:pin>
                <sec:pin algorithm="sha256">E9CZ9INDbd+2eRQozYqqbQ2yXLVKB9+xcprMF+44U1g=</sec:pin>
            </sec:pins>
        </sec:hpkp>
    </sec:headers>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | まずデフォルトで適用されるヘッダ出力を行うコンポーネント登録を無効化する。
    * - | (2)
      - | Cache-Control(Pragma, Expires)ヘッダを出力するコンポーネントを登録する。
    * - | (3)
      - | Frame-Optionsヘッダを出力するコンポーネントを登録する。
    * - | (4)
      - | X-Content-Type-Optionsヘッダを出力するコンポーネントを登録する。
    * - | (5)
      - | X-XSS-Protectionヘッダを出力するコンポーネントを登録する。
    * - | (6)
      - | Strict-Transport-Securityヘッダを出力するコンポーネントを登録する。
    * - | (7)
      - | Content-Security-PolicyヘッダまたはContent-Security-Policy-Report-Onlyヘッダを出力するコンポーネントを登録する。
    * - | (8)
      - | Public-Key-PinsヘッダまたはPublic-Key-Pins-Report-Onlyヘッダを出力するコンポーネントを登録する。

        * サイトの提示する証明書の公開鍵が一致しなかった場合、アクセスをブロックせず\ ``https://www.example.net/hpkp-report``\ に違反レポートの送信を行う。
        * 証明書の危殆化や期限切れなどの理由で証明書を更新した際に公開鍵の不一致が発生しないようにするために、バックアップ用の公開鍵の情報も設定している。


.. note:: **Public-Key-Pinsヘッダの出力について**

    Spring Securityのデフォルトの設定では、Public-Key-Pinsヘッダではなく、Public-Key-Pins-Report-Onlyヘッダが出力される。

    また、Spring Securityのデフォルト実装では、Public-Key-Pinsヘッダは、アプリケーションサーバに対してHTTPSを使ってアクセスがあった場合のみ出力される。


また、不要なものだけ無効化する方法も存在する。 

* spring-security.xmlの定義例
    
.. code-block:: xml 

    <sec:headers>
        <sec:cache-control disabled="true"/> <!-- disabled属性にtrueを設定して無効化 --> 
    </sec:headers>

上記の例だと、Cache-Control関連のヘッダだけが出力されなくなる。 

セキュリティヘッダの詳細については\ `公式リファレンス <http://docs.spring.io/spring-security/site/docs/4.1.4.RELEASE/reference/htmlsingle/#default-security-headers>`_\ を参照されたい。


セキュリティヘッダのオプション指定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

以下のヘッダでは、Spring Securityがデフォルトで出力する内容を変更することができる。

* X-Frame-Options
* X-XSS-Protection
* Strict-Transport-Security
* Content-Security-Policy(Content-Security-Policy-Report-Only)
* Public-Key-Pins(Public-Key-Pins-Report-Only)

Spring Securityのbean定義を変更することで、各要素の属性にオプション\ [#fSpringSecurityLinkageWithBrowser2]_\ を指定することができる。

* spring-security.xmlの定義例

.. code-block:: xml

    <sec:frame-options policy="SAMEORIGIN" />

.. [#fSpringSecurityLinkageWithBrowser2] 各要素で指定できるオプションは http://docs.spring.io/spring-security/site/docs/4.1.4.RELEASE/reference/htmlsingle/#nsa-headers を参照されたい。

カスタムヘッダの出力
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityがデフォルトで用意していないヘッダを出力することもできる。

以下のヘッダを出力するケースの例を説明する。

.. code-block:: text

    X-WebKit-CSP: default-src 'self'

上記のヘッダを出力する場合は、以下のようなbean定義を行う。

* spring-security.xmlの定義例

.. code-block:: xml

      <sec:headers>
          <sec:header name="X-WebKit-CSP" value="default-src 'self'"/>
      </sec:headers>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``<sec:headers>``\ 要素の子要素として\ ``<sec:header>`` を追加し、\ ``name``\ 属性にヘッダ名を\ ``value``\ 属性にヘッダ値を指定する。

リクエストパターン毎のセキュリティヘッダの出力
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityは、\ ``RequestMatcher``\ インタフェースの仕組みを利用して、リクエストのパターン毎にセキュリティヘッダの出力を制御することも可能である。

例えば、保護対象のコンテンツが\ ``/secure/``\ というパスの配下に格納されていて、保護対象のコンテンツへアクセスした時だけCache-Controlヘッダを出力する場合は、以下のようなbean定義を行う。

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

    <sec:http>
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

