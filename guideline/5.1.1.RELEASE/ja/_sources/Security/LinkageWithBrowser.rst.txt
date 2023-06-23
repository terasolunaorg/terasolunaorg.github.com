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

Spring Securityがデフォルトでサポートしているレスポンスヘッダは以下の5つである。

* Cache-Control (Pragma, Expires)
* X-Frame-Options
* X-Content-Type-Options
* X-XSS-Protection
* Strict-Transport-Security

.. tip:: **ブラウザのサポート状況**

    これらのヘッダに対する処理は、一部のブラウザではサポートされていない。ブラウザの公式サイトまたは以下のページを参照されたい。

    * https://www.owasp.org/index.php/HTTP_Strict_Transport_Security (Strict-Transport-Security)
    * https://www.owasp.org/index.php/Clickjacking_Defense_Cheat_Sheet (X-Frame-Options)
    * https://www.owasp.org/index.php/List_of_useful_HTTP_headers (X-Content-Type-Options, X-XSS-Protection)


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


How to use
--------------------------------------------------------------------------------

セキュリティヘッダ出力機能の適用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

前述のセキュリティヘッダ出力機能を適用する方法をする。

セキュリティヘッダ出力機能は、Spring 3.2から追加された機能でSpring Security 4.0からデフォルトで適用されるようになっている。 
そのため、セキュリティヘッダ出力機能を有効にするための特別な定義は不要である。 
なお、セキュリティヘッダ出力機能を適用したくない場合は、明示的に無効化する必要がある。 

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


また、不要なものだけ無効化する方法も存在する。 

* spring-security.xmlの定義例
    
.. code-block:: xml 

    <sec:headers>
        <sec:cache-control disabled="true"/> <!-- disabled属性にtrueを設定して無効化 --> 
    </sec:headers>

上記の例だと、Cache-Control関連のヘッダだけが出力されなくなる。 

セキュリティヘッダの詳細については\ `公式リファレンス <http://docs.spring.io/spring-security/site/docs/4.0.3.RELEASE/reference/htmlsingle/#default-security-headers>`_\ を参照されたい。


セキュリティヘッダのオプション指定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

以下のヘッダでは、Spring Securityがデフォルトで出力する内容を変更することができる。

* X-Frame-Options
* X-XSS-Protection
* Strict-Transport-Security

Spring Securityのbean定義を変更することで、各要素の属性にオプション\ [#fSpringSecurityLinkageWithBrowser2]_\ を指定することができる。

* spring-security.xmlの定義例

.. code-block:: xml

    <sec:frame-options policy="SAMEORIGIN" />

.. [#fSpringSecurityLinkageWithBrowser2] 各要素で指定できるオプションは http://docs.spring.io/spring-security/site/docs/4.0.3.RELEASE/reference/htmlsingle/#nsa-headers を参照されたい。

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

