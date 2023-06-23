.. _SpringSecuritySessionManagement:

セッション管理
================================================================================

.. only:: html

 .. contents:: 目次
    :local:

Overview
--------------------------------------------------------------------------------

本節では、「Webアプリケーションでセッションを扱う際に必要となるセキュリティ対策」及び「Spring Securityが提供しているセッション関連の機能」について説明する。

.. _SpringSecuritySessionManagementSecurityMeasure:

セッション利用時のセキュリティ対策
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Webアプリケーションでセッションを扱う場合、一般的には以下の攻撃に対して対策が必要となる。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - 対策
      - 説明
    * - | セッションハイジャック攻撃
      - | 通信の盗聴、規則性からの類推、クロスサイトスクリプティングなどを駆使してセッションIDを盗みとり、盗みとったセッションIDをつかっているユーザーになりすましてシステムを利用する攻撃。
    * - | セッション固定攻撃
      - | 攻撃者が事前に払い出したセッションIDを他人に使わせてシステムにログインさせ、攻撃者がログインしたユーザーになりすましてシステムを利用する攻撃。

セッションハイジャック攻撃への対策
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

セッションハイジャック攻撃への対策は、セッションIDが盗み取られないようにするしかない。
いったん盗み取られてしまうと、アプリケーションサーバは正規のユーザーからのリクエストなのか、
攻撃者からのリクエストなのかを判断することができない。

このようなセッションハイジャック攻撃からアプリケーションを守るためには、以下のような対策が必要である。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table:: **セッションハイジャック攻撃への対策**
    :header-rows: 1
    :widths: 30 70

    * - 対策
      - 説明
    * - | 推測困難なセッションIDの生成する
      - | 連番など推測できる値をセッションIDに使用せず、推測が困難な(セキュアな)ランダム値を使用する。
        | 基本的にはアプリケーションサーバが提供するセッションIDの生成機構を利用すればよい。
    * - | HTTPSを使って通信を暗号化する
      - | 盗まれると困る情報をやりとりする通信は、HTTPSプロトコルを使って暗号化する。
        | 通信の盗聴はフリーのソフトなどを使って簡単に行うことができため、盗聴されても解読されないように暗号化しておくことが重要である。
    * - | セッションIDはCookieを使って連携する
      - | クライアントとサーバーとの間でセッションIDを連携する際は、Cookieを使って連携するように設定し、URL Rewriting機能を無効化する。
    * - | Cookieの\ ``HttpOnly``\ 属性を指定する
      - | Cookieの\ ``HttpOnly``\ 属性を指定すると、JavaScriptからCookieにアクセスすることができなくため、クロスサイトスクリプティングを使ってセッションIDを盗むことができなくなる。
    * - | Cookieに\ ``Secure``\ 属性を指定する
      - | Cookieに\ ``Secure``\ 属性を指定すると、HTTPS通信の時だけCookieをサーバーに送信するため、誤ってHTTP通信を使ってしまった時にセッションIDが盗み取られるリスクを減らすことができる。

.. note:: **URL Rewriting**

    URL Rewritingは、Cookieを使用できないクライアントとセッションを維持するための仕組みである。
    具体的には、URLのリクエストパラメータの中にセッションIDを含めることでクライアントとサーバーの間でセッションIDを連携する。

    * URL Rewritingが行われたURL例

        .. code-block:: guess

            http://localhost:8080/;jsessionid=7E6EDE4D3317FC5F14FD912BEAC96646

    \ ``jsessionid=7E6EDE4D3317FC5F14FD912BEAC96646``\ の部分がURL RewritingされたセッションIDになる。
    ServletのAPI仕様では、以下のメソッドを呼び出すとURL Rewritingが行われる可能性があり、JSTLやSpringが提供しているJSPタグライブラリの中でもこれらのメソッドを呼び出している。

    * \ ``HttpServletResponse#encodeURL(String)``\
    * \ ``HttpServletResponse#encodeRedirectURL(String)``\

URL Rewritingが行われるとURL内にセッションIDが露出してしまうため、セッションIDを盗まれるリスクが高くなる。
そのため、Cookieを使うことができるクライアントのみをサポートする場合は、サーブレットコンテナのURL Rewriting機能を無効化することを推奨する。

|

セッション固定攻撃への対策
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

セッション固定攻撃からアプリケーションを守るためには、以下のような対策が必要になる。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table:: **セッション固定攻撃への対策**
    :header-rows: 1
    :widths: 30 70

    * - 対策
      - 説明
    * - | URL Rewriting機能を無効化する
      - | URL Rewriting機能を無効化すると、攻撃者が事前に払い出したセッションIDが使われず、新たにセッションが開始される。
    * - | ログイン後にセッションIDを変更する
      - | ログイン後にセッションIDを変更することで、攻撃者が事前に払い出したセッションIDが使用できなくなる。

|

Spring Securityが提供するセッション管理機能
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityでは、セッションについて、主に以下の機能が提供されている。

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: **セッションに関する提供機能**
    :header-rows: 1
    :widths: 25 75

    * - 機能
      - 説明
    * - | セキュリティ対策
      - | セッションハイジャック攻撃等のセッションIDを使用した攻撃への対策機能。
    * - | ライフサイクル制御
      - | セッションの生成～破棄までのライフサイクルを制御する機能。
    * - | タイムアウト制御
      - | タイムアウトにより、セッションを破棄する機能。
    * - | 多重ログイン制御
      - | 同一ユーザーによる多重ログイン時のセッションを制御する機能。

.. _authentication(spring_security)_how_to_use_sessionmanagement:

How to use
--------------------------------------------------------------------------------

セッションハイジャック攻撃への対策
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ここではURL Rewriting機能を無効化し、Cookieを使用してセッションIDを連携する方法を説明する。、

Spring SecurityによるURL Rewriting機能の無効化
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring SecurityはURL Rewritingを無効化するための仕組みを提供しており、この機能はデフォルトで適用されている。
Cookieを使えないクライアントをサポートする必要がある場合は、URL Rewritingを許可するようにBean定義する。

* spring-security.xmlの定義例

.. code-block:: xml

    <sec:http disable-url-rewriting="false"> <!-- falseを指定してURL Rewritingを有効化 -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | Spring Securityのデフォルトでは、\ ``disable-url-rewriting``\ の値は \ ``true``\であるため、URL Rewritingは行われない。
        | URL Rewritingを有効にする際は、\ ``<sec:http>``\ 要素の \ ``disable-url-rewriting``\ 属性に\ ``false``\ を設定する。

サーブレットコンテナによるURL Rewriting機能の無効化
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Servletの標準仕様の仕組みを使ってセッションをセキュアに扱うことが可能である。

* web.xmlの定義例

.. code-block:: xml

    <session-config>
        <cookie-config>
            <http-only>true</http-only> <!-- (1)  -->
        </cookie-config>
        <tracking-mode>COOKIE</tracking-mode> <!-- (2) -->
    </session-config>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | Cookieに\ ``HttpOnly``\ 属性を付与する場合は、\ ``<http-only>``\ 要素に\ ``true``\ を指定する。
        | 使用するアプリケーションサーバによっては、デフォルト値が\ ``true``\ になっている。
    * - | (3)
      - | URL Rewriting機能を無効化する場合は、\ ``<tracking-mode>``\ 要素に\ ``COOKIE``\ を指定する。

上記の定義例からは省略しているが、\ ``<cookie-config>``\ に \ ``<secure>true</secure>``\を追加することで、 Cookieに\ ``Secure``\ 属性を付与することができる。
ただし、cookieのsecure化は、\ ``web.xml``\ で指定するのではなく、クライアントとHTTPS通信を行うミドルウェア(SSLアクセラレータやWebサーバーなど)で付与する方法を検討されたい。

実際のシステム開発の現場において、ローカルの開発環境でHTTPSを使うケースはほとんどない。
また、本番環境においても、HTTPSを使うのはSSLアクセラレータやWebサーバーとの通信までで、アプリケーションサーバへの通信はHTTPで行うケースも少なくない。
このような環境下で\ ``Secure``\ 属性の指定を\ ``web.xml``\ で行ってしまうと、実行環境毎に\ ``web.xml``\ や\ ``web-fragment.xml``\ を用意することになり、ファイルの管理が煩雑になるため推奨されない。


.. _SpringSecuritySessionManagementSetup:

セッション管理機能の適用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityのセッション管理機能を適用する方法を説明する。
Spring Securityのセッション管理機能の処理を使用する場合は、以下のようなbean定義を行う。

* spring-security.xmlの定義例

.. code-block:: xml

    <sec:http>
        <!-- ommited -->
        <sec:session-management /> <!-- (1) -->
        <!-- ommited -->
    </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``<sec:http>``\ 要素の子要素として\ ``<sec:session-management>``\ 要素を指定する。
        | \ ``<sec:session-management>``\ 要素を指定すると、セッション管理機能が適用される。

|

セッション固定攻撃への対策
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityは、セッション固定攻撃対策として、ログイン成功時にセッションIDを変更するためのオプションを4つ用意している。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table:: **セッション固定攻撃への対策のオプション**
    :header-rows: 1
    :widths: 30 70

    * - オプション
      - 説明
    * - | \ ``changeSessionId``\
      - | Servlet 3.1で追加された\ ``HttpServletRequest#changeSessionId()``\ を使用してセッションIDを変更する。
        | (これはServlet 3.1以上のコンテナ上でのデフォルトの動作である)
    * - | \ ``migrateSession``\
      - | ログイン前に使用していたセッションを破棄し、新たにセッションを作成する。
        | このオプションを使用すると、ログイン前にセッションに格納されていたオブジェクトは新しいセッションに引き継がれる。
        | (Servlet 3.0以下のコンテナ上でのデフォルトの動作の動作である)
    * - | \ ``newSession``\
      - | このオプションは\ ``migrateSession``\ と同じ方法でセッションIDを変更するが、ログイン前に格納されていたオブジェクトは新しいセッションに引き継がれない。
    * - | \ ``none``\
      - | Spring Securityは、セッションIDを変更しない。

デフォルトの動作を変更したい場合は、以下のようなbean定義を行う。

* spring-security.xmlの定義例

.. code-block:: xml

    <sec:session-management
            session-fixation-protection="newSession"/> <!-- (1) -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ``<sec:session-management>``\ 要素の\ ``session-fixation-protection``\ 属性にセッション固定攻撃の対策方法を指定する。

.. _SpringSecuritySessionManagementLifecycle:

セッションのライフサイクル制御
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityは、リクエストを跨いで認証情報などのオブジェクトを共有するための手段としてHTTPセッションを使用しており、Spring Securityの処理の中でセッションのライフサイクル(セッションの作成と破棄)を制御している。

.. note:: **セッション情報の格納先**

    Spring Securityが用意しているデフォルト実装ではHTTPセッションを使用するが、HTTPセッション以外(データベースやキーバリューストアなど)にオブジェクトを格納することも可能なアーキテクチャになっている。

セッションの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityの処理の中でどのような方針でセッションを作成して利用するかは、以下のオプションから選択することができる。

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: **セッションの作成方針**
    :header-rows: 1
    :widths: 25 75

    * - オプション
      - 説明
    * - | \ ``always``\
      - | セッションが存在しない場合は、無条件に新たなセッションを生成する。
        | このオプションを指定すると、Spring Securityの処理でセッションを使わないケースでもセッションが作成される。
    * - | \ ``ifRequired``\
      - | セッションが存在しない場合は、セッションにオブジェクトを格納するタイミングで新たなセッションを作成して利用する。(デフォルトの動作)
    * - | \ ``never``\
      - | セッションが存在しない場合は、セッションの生成及び利用は行わない。
        | ただし、既にセッションが存在している場合はセッションを利用する。
    * - | \ ``stateless``\
      - | セッションの有無に関係なく、セッションの生成及び利用は行わない。

デフォルトの振る舞いを変更したい場合は、以下のようなbean定義を行う。

* spring-security.xmlの定義例

.. code-block:: xml

    <sec:http create-session="stateless"> <!-- (1) -->
        <!-- ommited -->
    </sec:http>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | \ (1)
      - | \ ``<sec:http>``\ 要素の\ ``create-session``\ 属性に、変更したいセッションの作成方針を指定する。

セッションの破棄
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityは、以下のタイミングでセッションを破棄する。

* ログアウト処理が実行されたタイミング
* 認証処理が成功したタイミング (セッション固定攻撃対策として\ ``migrateSession``\ 又は\ ``newSession``\ が適用されるとセッションが破棄される)

.. _SpringSecuritySessionManagementTimeout:

セッションタイムアウトの制御
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

セッションにオブジェクトを格納する場合、適切なセッションタイムアウト値を指定して、一定時間操作がないユーザーとのセッションを自動で破棄するようにするのが一般的である。

セッションタイムアウトの指定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

セッションタイムアウトは、サーブレットコンテナに対して指定する。
アプリケーションサーバーによっては、サーバー独自の指定方法を用意しているケースもあるが、ここでは、Servlet標準仕様で定められた指定方法を説明する。

* web.xmlの定義例

.. code-block:: xml

    <session-config>
        <session-timeout>60</session-timeout> <!-- (1) -->
        <!-- ommited -->
    </session-config>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``<session-timeout>``\ 要素に適切なタイムアウト値(分単位)を指定する。
        |  タイムアウト値を指定しない場合は、サーブレットコンテナが用意しているデフォルト値が適用される。
        | また、0以下の値を指定するとサーブレットコンテナのセッションタイム機能が無効化される。

.. _SpringSecuritySessionDetectInvalidSession:

無効なセッションを使ったリクエストの検知
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityは、無効なセッションを使ったリクエストを検知する機能を提供している。
無効なセッションとして扱われるリクエストの大部分は、セッションタイムアウト後のリクエストである。
デフォルトではこの機能は無効になっているが、以下のようなbean定義を行うことで有効化することができる。

* spring-security.xmlの定義例

.. code-block:: xml

    <sec:session-management
            invalid-session-url="/error/invalidSession"/>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``<sec:session-management>``\ 要素の\ ``invalid-session-url``\ 属性に、無効なセッションを使ったリクエストを検知した際のリダイレクト先のパスを指定する。

除外パスの指定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

無効なセッションを使ったリクエストを検知する機能を有効にすると、Spring Securityのサーブレットフィルタを通過するすべてのリクエストに対してチェックが行われる。
そのため、セッションが無効な状態でアクセスしても問題がないページにアクセスした場合もチェックが行われる。

この動作を変更したい場合は、チェック対象から除外したいパスに対して個別にbean定義を行うことで実現することが可能である。
例として、トップページを開くためのパス(\ ``"/"``\ )を除外パスに指定したい場合は、以下のようなbean定義を行う。

* spring-security.xmlの定義例

.. code-block:: xml

    <!-- (1) -->
    <sec:http pattern="/"> <!-- (2) -->
        <sec:session-management />
    </sec:http>

    <!-- (3) -->
    <sec:http>
        <!-- ommited -->
        <sec:session-management
                invalid-session-url="/error/invalidSession"/>
        <!-- ommited -->
    </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | トップページを開くためのパス(\ ``"/"``\ )に適用する\ ``SecurityFilterChain``\ を作成するための\ ``<sec:http>``\ 要素を新たに追加する。
    * - | (2)
      - | (1)の\ ``<sec:http>``\ 要素を使って生成した\ ``SecurityFilterChain``\ を適用するパスパターンを指定する。
        | 指定可能なパスパターンはAnt形式のパス表記と正規表現の２つの形式であり、デフォルトではAnt形式のパスとして扱われる。
        | また、パスパターンではなく\ ``RequestMatcher``\ オブジェクトを直接指定することも可能である。
    * - | (3)
      - | 個別定義していないパスに適用する\ ``SecurityFilterChain``\ を作成するための\ ``<sec:http>``\ 要素を定義する。
        | この定義は、個別定義用の\ ``<sec:http>``\ 要素より下に定義すること。
        | これは\ ``<sec:http>``\ 要素の定義順番が\ ``SecurityFilterChain``\ の優先順位となるためである。

|

.. _SpringSecuritySessionManagementConcurrency:

多重ログインの制御
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityは、同じユーザー名(ログインID)を使った多重ログインを制御する機能を提供している。
デフォルトではこの機能は無効になってるが、:ref:`SpringSecurityHowToUseSessionManagementConcurrency` を行うことで有効化することができる。

.. warning:: **多重ログイン制御における制約**

    Spring Securityが提供しているデフォルト実装では、ユーザー毎のセッション情報をアプリケーションサーバーのメモリ内で管理しているため、以下の2つの制約がある。

    ひとつめの制約として、複数のアプリケーションサーバーを同時に起動するシステムでは、デフォルト実装を利用することができないことが挙げられる。
    複数のアプリケーションサーバーを同時に使用する場合は、ユーザー毎のセッション情報をデータベースやキーバリューストア(キャッシュサーバー)などの共有領域で管理する実装クラスの作成が必要になる。

    ふたつめの制約は、アプリケーションサーバーを停止または再起動時した際に、セッション情報が復元されると、正常動作しない可能性があるという点である。
    使用するアプリケーションサーバーによっては、停止または再起動時のセッション状態を復元する機能をもっているため、実際のセッション状態とSpring Securityが管理しているセッション情報に不整合が生じることになる。
    このような不整合が生まれる可能性がある場合は、以下のいずれかの対応が必要になる。

    * アプリケーションサーバ側のセッション状態が復元されないようにする。
    * Spring Security側のセッション情報を復元する仕組みを実装する。
    * HTTPセッション以外(データベースやキーバリューストアなど)にオブジェクトを格納する。

本節では、Spring Securityのデフォルト実装を使用する方法を紹介する。
Spring Securityが用意しているデフォルト実装ではHTTPセッションを使用するが、HTTPセッション以外(データベースやキーバリューストアなど)にオブジェクトを格納することも可能なアーキテクチャになっている。
ただし、ここで紹介する方法は **上記Warningの制約が残っている実装方法であるため** 、適用する際は注意されたい。

.. Todo::
   インメモリを使用しない実装方法に関しては、今後追加予定である。

.. _SpringSecurityHowToUseSessionManagementConcurrency:

セッションのライフサイクル検知の有効化
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

多重ログインを制御する機能は、:ref:`セッションのライフサイクル(セッションの生成と破棄)を検知する仕組み<SpringSecuritySessionManagementLifecycle>` を利用してユーザー毎のセッション状態を管理している。
このため、多重ログインの制御機能を使用する際は、Spring Securityから提供されている\ ``HttpSessionEventPublisher``\ クラスをサーブレットコンテナに登録する必要がある。

* web.xmlの定義例

.. code-block:: xml

    <listener>
        <!-- (1) -->
        <listener-class>
            org.springframework.security.web.session.HttpSessionEventPublisher
        </listener-class>
    </listener>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | サーブレットリスナとして\ ``HttpSessionEventPublisher``\ を登録する。

多重ログインの禁止(先勝ち)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

同じユーザー名(ログインID)を使って既にログインしているユーザーがいる場合に、認証エラーを発生させて多重ログインを防ぐ場合は、以下のようなbean定義を行う。

* bean定義ファイルの定義例

.. code-block:: xml

    <sec:session-management>
        <sec:concurrency-control
                max-sessions="1"
                error-if-maximum-exceeded="true"/> <!-- (1) (2) -->
    </sec:session-management>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - \ (1)
      - \ ``<sec:concurrency-control>``\ 要素の\ ``max-sessions``\ 属性に、同時にログイン
        を許可するセッション数を指定する。
        多重ログインを防ぎたい場合は、通常\ ``1``\ を指定する。
    * - \ (2)
      - \ ``<sec:concurrency-control>``\ 要素の\ ``error-if-maximum-exceeded``\ 属性に、
        同時にログインできるセッション数を超えた時の動作を指定する。
        既にログインしているユーザーを有効なユーザーとして扱う場合は、\ ``true``\
        を指定する。

多重ログインの禁止(後勝ち)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

同じユーザー名(ログインID)を使って既にログインしているユーザーがいる場合に、
既にログインしているユーザーを無効化することで多重ログインを防ぐ場合は、
以下のようなbean定義を行う。

* spring-security.xmlの定義例

.. code-block:: xml

    <sec:session-management>
        <sec:concurrency-control
                max-sessions="1"
                error-if-maximum-exceeded="false"
                expired-url="/error/expire"/> <!-- (1) (2) -->
    </sec:session-management>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``<sec:concurrency-control>``\ 要素の\ ``error-if-maximum-exceeded``\ 属性に、同時にログインできるセッション数を超えた時の動作を指定する。
        | 新たにログインしたユーザーを有効なユーザーとして扱う場合は、\ ``false``\ を指定する。
    * - | (2)
      - | \ ``<sec:concurrency-control>``\ 要素の\ ``expired-url``\ 属性に、無効化されたユーザーからのリクエストを検知した際のリダイレクト先のパスを指定する。
        | これは\ ``<sec:http>``\ 要素の定義順番が\ ``SecurityFilterChain``\ の優先順位となるためである。

.. raw:: latex

   \newpage

