.. _SpringSecurityOverview:

Spring Security概要
================================================================================

.. only:: html

 .. contents:: 目次
    :local:


Spring Securityは、アプリケーションにセキュリティ対策機能を実装する際に使用するフレームワークである。 
Spring Securityはスタンドアロンなアプリケーションでも利用できるが、サーブレットコンテナにデプロイするWebアプリケーションに対してセキュリティ対策を行う際に利用するのが一般的である。
本章では、Spring Securityが提供する機能のうち、一般的なWebアプリケーションでの利用頻度が高いと思われる機能にしぼって説明する。

.. tip:: **ガイドラインで紹介していない機能**

    Spring Securityは、本ガイドラインで紹介していない機能も多く提供している。
    Spring Securityが提供するすべての機能を知りたい場合は、\ `Spring Security Reference <http://docs.spring.io/spring-security/site/docs/4.0.3.RELEASE/reference/htmlsingle/#security-filter-chain>`_\ を参照されたい。

.. note:: **Spring Securityのバージョン**

    本ガイドラインでは、Spring Securityのバージョンは4.0以上を使用することを前提としている。
    Spring Securityが4.0にバージョンアップするにあたり、様々な変更が適用されており、以降で記述されるサンプルについても、Spring Security 4を使用したサンプルとなっている。

    変更内容については\ `Migrating from Spring Security 3.x to 4.x (XML Configuration) <http://docs.spring.io/spring-security/site/migrate/current/3-to-4/html5/migrate-3-to-4-xml.html>`_\ を参照されたい。

.. _SpringSecurityFunctionalities:

Spring Securityの機能
--------------------------------------------------------------------------------

セキュリティ対策の基本機能
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Spring Securityは、セキュリティ対策の基本機能として以下の機能を提供している。

\

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: **セキュリティ対策の基本機能**
    :header-rows: 1
    :widths: 25 75

    * - 機能
      - 説明
    * - :ref:`認証機能<SpringSecurityAuthentication>` 
      - アプリケーションを利用するユーザーの正当性を確認する機能。
    * - :ref:`認可機能<SpringSecurityAuthorization>`
      - アプリケーションが提供するリソースや処理に対してアクセスを制御する機能。

|

セキュリティ対策の強化機能
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Spring Securityでは認証と認可という基本的な機能に加え、Webアプリケーションのセキュリティを強化するための機能をいくつか提供している。

\

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: **セキュリティ対策の強化機能**
    :header-rows: 1
    :widths: 25 75

    * - 機能
      - 説明
    * - :ref:`セッション管理機能<SpringSecuritySessionManagement>` 
      - セッションハイジャック攻撃やセッション固定攻撃からユーザーを守る機能、
        セッションのライフサイクル(生成、破棄、タイムアウト)を制御するための機能。
    * - :ref:`CSRF対策機能<SpringSecurityCSRF>`
      - クロスサイトリクエストフォージェリ(CSRF)攻撃からユーザーを守るための機能。
    * - :ref:`セキュリティヘッダ出力機能<SpringSecurityLinkageWithBrowser>`
      - Webブラウザのセキュリティ対策機能と連携し、ブラウザの機能を悪用した攻撃からユーザーを守るための機能。

|

.. _SpringSecurityArchitecture:

Spring Securityのアーキテクチャ
--------------------------------------------------------------------------------
各機能の詳細な説明を行う前に、Spring Securityのアーキテクチャ概要とSpring Securityを構成する主要なコンポーネントの役割を説明する。

.. note::

    ここで説明する内容は、Spring Securityが提供するデフォルトの動作をそのまま利用する場合や、
    Spring Securityのコンフィギュレーションをサポートする仕組みを利用する場合は、開発者が直接意識する必要ない。
    そのため、まず各機能の使い方を知りたい場合は、本節を読み飛ばしても問題はない。
    
    ただし、ここで説明する内容は、Spring Securityのデフォルトの動作をカスタマイズする際に必要になるので、
    アプリケーションのアーキテクトは一読しておくことを推奨する。

|

Spring Securityのモジュール
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

まずフレームワークスタックとなっているSpring Securityの提供モジュールを紹介する。

フレームワークスタックモジュール群
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

フレームワークスタックモジュールは、以下の通りである。
本ガイドラインでもこれらのモジュールを使用してセキュリティ対策を行う方法について説明する。

\

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: **フレームワークスタックモジュール群**
    :header-rows: 1
    :widths: 25 75

    * - モジュール名
      - 説明
    * - \ ``spring-security-core``\
      - 認証と認可機能を実現するために必要となるコアなコンポーネントが格納されている。
        このモジュールに含まれるコンポーネントは、スタンドアロン環境で実行するアプリケーションでも使用することができる。
    * - \ ``spring-security-web``\
      - Webアプリケーションのセキュリティ対策を実現するために必要となるコンポーネントが格納されている。
        このモジュールに含まれるコンポーネントは、Web層(サーブレットAPIなど)に依存する処理を行う。
    * - \ ``spring-security-config``\
      - 各モジュールから提供されているコンポーネントのセットアップをサポートするためのコンポーネント(コンフィギュレーションをサポートするクラスやXMLネームスペースを解析するクラスなど)が格納されている。
        このモジュールを使用すると、Spring Securityのbean定義を簡単に行うことができる。
    * - \ ``spring-security-taglibs``\
      - 認証情報や認可機能にアクセスするためのJSPタグライブラリが格納されている。
    * - \ ``spring-security-acl``\
      - EntityなどのドメインオブジェクトをAccess Control List(ACL)を使用して認可制御するために必要となるコンポーネントが格納されている。
        本モジュールは依存関係の都合上、フレームワークスタックに含まれているモジュールであるため、本ガイドラインにおいて使用方法の説明は行わない。

要件に合わせて使用するモジュール群
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

フレームワークスタックではないが、一般的に利用される認証方法などをサポートするために、
以下のようなモジュールも提供されている。
セキュリティ要件に応じて、これらのモジュールの使用も検討されたい。

\

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: **要件に合わせて使用するモジュール群**
    :header-rows: 1
    :widths: 25 75

    * - モジュール名
      - 説明
    * - \ ``spring-security-remoting``\
      - JNDI経由でDNSにアクセス、Basic認証が必要なWebサイトにアクセス、Spring Securityを使用してセキュリティ対策しているメソッドにRMI経由でアクセスする際に必要となるコンポーネントが格納されている。
    * - \ ``spring-security-aspects``\
      - Javaのメソッドに対して認可機能を適用する際にAspectJの機能を使用する際に必要となるコンポーネントが格納されています。
        このモジュールは、AOPとしてSpring AOPを使う場合は不要である。
    * - \ ``spring-security-messaging``\ \ [#fSpringSecurityArchitecture5]_\ 
      - SpringのWeb Socket機能に対してセキュリティ対策を追加するためのコンポーネントが格納されている。 
    * - \ ``spring-security-data``\ \ [#fSpringSecurityArchitecture5]_\ 
      - Spring Dataの機能から認証情報にアクセスできるようにするためのコンポーネントが格納されている。 
    * - \ ``spring-security-ldap``\
      - Lightweight Directory Access Protocol(LDAP)を使用した認証を実現するために必要となるコンポーネントが格納されている。
    * - \ ``spring-security-openid``\
      - OpenID\ [#fSpringSecurityArchitecture1]_\ を使用した認証を実現するために必要となるコンポーネントが格納されている。
    * - \ ``spring-security-cas``\
      - Central Authentication Service(CAS)\ [#fSpringSecurityArchitecture2]_\ と連携するために必要となるコンポーネントが格納されている。
    * - \ ``spring-security-crypto``\
      - 暗号化、キーの生成、ハッシュアルゴリズムを利用したパスワードエンコーディングを行うためのコンポーネントが格納されている。
        このモジュールに含まれるクラスは、フレームワークスタックモジュールである\ ``spring-security-core``\にも含まれている。

テスト用のモジュール
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Security 4.0からはテストを支援するためのモジュールが追加されている。

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}| 
.. list-table:: **テスト用のモジュール** 
    :header-rows: 1 
    :widths: 25 75 
  
    * - モジュール名 
      - 説明 
    * - \ ``spring-security-test``\ \ [#fSpringSecurityArchitecture5]_\ 
      - Spring Securityに依存しているクラスのテストを支援するためのコンポーネントが格納されている。 
        このモジュールを使用すると、JUnitテスト時に必要となる認証情報を簡単にセットアップすることができる。 
        また、Spring MVCのテスト用コンポーネント(\ ``MockMvc``\ )と連携して使用するコンポーネントも含まれている。 

要件に合わせて利用する関連モジュール群
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

また、いくつかの関連モジュールも提供されている。

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: **要件に合わせて利用する主な関連モジュール群**
    :header-rows: 1
    :widths: 25 75

    * - モジュール名
      - 説明
    * - \ ``spring-security-oauth2``\ \ [#fSpringSecurityArchitecture3]_\
      - OAuth 2.0\ [#fSpringSecurityArchitecture4]_\ の仕組みを使用してAPIの認可を実現するために必要となるコンポーネントが格納されている。
    * - \ ``spring-security-oauth``\ \ [#fSpringSecurityArchitecture3]_\
      - OAuth 1.0の仕組みを使用してAPIの認可を実現するために必要となるコンポーネントが格納されている。

|

.. [#fSpringSecurityArchitecture1] OpenIDは、簡単に言うと「1つのIDで複数のサイトにログインできるようする」ための仕組みである。
.. [#fSpringSecurityArchitecture2] CASは、OSSとして提供されているシングルサインオン用のサーバーコンポーネントである。詳細は https://www.apereo.org/cas を参照されたい。
.. [#fSpringSecurityArchitecture3] 詳細は http://projects.spring.io/spring-security-oauth/ を参照されたい。
.. [#fSpringSecurityArchitecture4] OAuth 2.0は、OAuth 1.0が抱えていた課題(署名と認証フローの複雑さ、モバイルやデスクトップのクライアントアプリの未対応など)を改善したバージョンで、OAuth 1.0との後方互換性はない。
.. [#fSpringSecurityArchitecture5] Spring Security 4.0から追加されたモジュールである。

|

.. _SpringSecurityProcess:

フレームワーク処理
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityは、サーブレットフィルタの仕組みを使用してWebアプリケーションのセキュリティ対策を行うアーキテクチャを採用しており、以下のような流れで処理を実行している。

.. figure:: ./images_SpringSecurity/Architecture.png
    :width: 100%

    **Spring Securityのフレームワークアーキテクチャ**

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - \ (1)
      - クライアントは、Webアプリケーションに対してリクエストを送る。
    * - \ (2)
      - Spring Securityの\ ``FilterChainProxy``\ クラス(サーブレットフィルタ)がリクエストを受け取り、
        \ ``HttpFirewall``\ インタフェースのメソッドを呼び出して\ ``HttpServletRequest``\ と\ ``HttpServletResponse``\ に対してファイアフォール機能を組み込む。
    * - \ (3)
      - \ ``FilterChainProxy``\ クラスは、Spring Securityが提供しているセキュリティ対策用のSecurity Filter(サーブレットフィルタ)クラスに処理を委譲する。
    * - \ (4)
      - Security Filterは複数のクラスで構成されており、サーブレットフィルタの処理が正常に終了すると後続のサーブレットフィルタが呼び出される。
    * - \ (5)
      - 最後のSecurity Filterの処理が正常に終了した場合、後続処理(サーブレットフィルタやサーブレットなど)を呼びだし、Webアプリケーション内のリソースへアクセスする。
    * - \ (6)
      - \ ``FilterChainProxy``\ クラスは、Webアプリケーションから返却されたリソースをクライアントへレスポンスする。

|

Webアプリケーション向けのフレームワーク処理を構成する主要なコンポーネントは以下の通りである。
詳細は \ `Spring Security Reference -The Security Filter Chain- <http://docs.spring.io/spring-security/site/docs/4.0.3.RELEASE/reference/htmlsingle/#security-filter-chain>`_\ を参照されたい。


FilterChainProxy
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``FilterChainProxy``\ クラスは、Webアプリケーション向けのフレームワーク処理のエントリーポイントとなるサーブレットフィルタクラスである。
このクラスはフレームワーク処理の全体の流れを制御するクラスであり、具体的なセキュリティ対策処理はSecurity Filterに委譲している。

HttpFirewall
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``HttpFirewall``\ インタフェースは、\ ``HttpServletRequest``\ と\ ``HttpServletResponse``\ に対してファイアフォール機能を組み込むためのインタフェースである。
デフォルトでは、\ ``DefaultHttpFirewall``\ クラスが使用され、ディレクトリトラバーサル攻撃やHTTPレスポンス分割攻撃に対するチェックなどが実装されている。

SecurityFilterChain
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``SecurityFilterChain``\ インタフェースは、\ ``FilterChainProxy``\ が受け取ったリクエストに対して、適用するSecurity Filterのリストを管理するためのインタフェースである。
デフォルトでは\ ``DefaultSecurityFilterChain``\ クラスが使用され、適用するSecurity Filterのリストを、リクエストURLのパターン毎に管理する。

たとえば、以下のようなbean定義を行うと、URLに応じて異なる内容のセキュリティ対策を適用することができる。

* xxx-web/src/main/resources/META-INF/spring/spring-security.xmlの定義例

.. code-block:: xml

    <sec:http pattern="/api/**">
        <!-- ... -->
    </sec:http>

    <sec:http pattern="/ui/**">
        <!-- ... -->
    </sec:http>

Security Filter
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Security Filterクラスは、フレームワーク機能やセキュリティ対策機能を実現する上で必要となる処理を提供するサーブレットフィルタクラスである。

Spring Securityは、複数のSecurity Filterを連鎖させることでWebアプリケーションのセキュリティ対策を行う仕組みになっている。
ここでは、認証と認可機能を実現するために必要となるコアなクラスを紹介する。
詳細は \ `Spring Security Reference -Core Security Filters- <http://docs.spring.io/spring-security/site/docs/4.0.3.RELEASE/reference/htmlsingle/#core-web-filters>`_\ を参照されたい。

.. _SpringSecurityTableSecurityFilter:

.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table:: **コアなSecurity Filter**
    :header-rows: 1
    :widths: 35 65

    * - クラス名
      - 説明
    * - \ ``SecurityContextPersistenceFilter``\
      - 認証情報をリクエストを跨いで共有するための処理を提供するクラス。
        デフォルトの実装では、\ ``HttpSession``\ に認証情報を格納することで、リクエストをまたいで認証情報を共有している。
    * - \ ``UsernamePasswordAuthenticationFilter``\
      - リクエストパラメータで指定されたユーザー名とパスワードを使用して認証処理を行うクラス。
        フォーム認証を行う際に使用する。
    * - \ ``LogoutFilter``\
      - ログアウト処理を行うクラス。
    * - \ ``FilterSecurityInterceptor``\
      - HTTPリクエスト(\ ``HttpServletRequest``\ )に対して認可処理を実行するためのクラス。
    * - \ ``ExceptionTranslationFilter``\
      - \ ``FilterSecurityInterceptor``\ で発生した例外をハンドリングし、クライアントへ返却するレスポンスを制御するクラス。
        デフォルトの実装では、未認証ユーザーからのアクセスの場合は認証を促すレスポンス、
        認証済みのユーザーからのアクセスの場合は認可エラーを通知するレスポンスを返却する。

|

.. _SpringSecuritySetup:


Spring Securityのセットアップ
--------------------------------------------------------------------------------

WebアプリケーションにSpring Securityを適用するためのセットアップ方法について説明する。

ここでは、WebアプリケーションにSpring Securityを適用し、Spring Securityが提供しているデフォルトのログイン画面を表示させる最もシンプルなセットアップ方法を説明する。
実際のアプリケーション開発で必要となるカスタマイズ方法や拡張方法については、次節以降で順次説明する。

.. note::

    開発プロジェクトを\ `ブランンクプロジェクト <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank>`_\ から作成すると、ここで説明する各設定はセットアップ済みの状態になっている。
    開発プロジェクトの作成方法については、「:doc:`../ImplementationAtEachLayer/CreateWebApplicationProject`」を参照されたい。

|

.. _SpringSecuritySetupDependency:

依存ライブラリの適用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

まず、Spring Securityを依存関係として使用している共通ライブラリを適用する。
Spring Securityと共通ライブラリの関連については、:ref:`frameworkstack_common_library` を参照されたい。

本ガイドラインでは、Mavenを使って開発プロジェクトを作成していることを前提とする。

* xxx-domain/pom.xmlの設定例

.. code-block:: xml

    <dependency>
        <groupId>org.terasoluna.gfw</groupId>
        <artifactId>terasoluna-gfw-security-core</artifactId>  <!-- (1) -->
    </dependency>

* xxx-web/pom.xmlの設定例

.. code-block:: xml

    <dependency>
        <groupId>org.terasoluna.gfw</groupId>
        <artifactId>terasoluna-gfw-security-web</artifactId>  <!-- (2) -->
    </dependency>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - \ (1)
      - ドメイン層のプロジェクトでSpring Securityの機能を使用する場合は、terasoluna-gfw-security-coreをdependencyに追加する。
    * - \ (2)
      - アプリケーション層のプロジェクトでSpring Securityの機能を使用する場合は、terasoluna-gfw-security-webをdependencyに追加する。


.. note::

    本ガイドラインでは、Spring IO Platformを使用してライブラリのバージョンを管理する前提で記載しているため、\ ``<version>``\ 要素は省略している。

|

bean定義ファイルの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Spring Securityのコンポーネントをbean定義するため、以下のようなXMLファイルを作成する。（`ブランクプロジェクト <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank>`_\より抜粋）

* xxx-web/src/main/resources/META-INF/spring/spring-security.xmlの定義例

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:sec="http://www.springframework.org/schema/security"
           xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/security
            http://www.springframework.org/schema/security/spring-security.xsd
           "> <!-- (1) -->

        <sec:http> <!-- (2) -->
            <sec:form-login /> <!-- (3) -->
            <sec:logout /> <!-- (4) -->
            <sec:access-denied-handler ref="accessDeniedHandler"/> <!-- (5) -->
            <sec:custom-filter ref="userIdMDCPutFilter" after="ANONYMOUS_FILTER"/> <!-- (6) -->
            <sec:session-management /> <!-- (7) -->
        </sec:http>

        <sec:authentication-manager /> <!-- (8) -->

        <bean id="accessDeniedHandler" class="org.springframework.security.web.access.DelegatingAccessDeniedHandler"> <!-- (9) -->
            <!-- omitted -->
        </bean>

        <bean id="userIdMDCPutFilter" class="org.terasoluna.gfw.security.web.logging.UserIdMDCPutFilter">  <!-- (10) -->
            <!-- omitted -->
        </bean>

    </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90


    * - 項番
      - 説明
    * - \ (1)
      - Spring Securityから提供されているXMLネームスペースを有効する。
        上記例では、\ ``sec``\ という名前を割り当てている。
        XMLネームスペースを使用すると、Spring Securityのコンポーネントのbean定義を簡単に行うことができる。
    * - \ (2)
      - \ ``<sec:http>``\ タグを定義する。
        \ ``<sec:http>``\ タグを定義すると、Spring Securityを利用するために必要となるコンポーネントのbean定義が自動的に行われる。
    * - \ (3)
      - \ ``<sec:form-login>``\ タグを定義し、フォーム認証を使用したログインに関する設定行う。
        \ 詳細は :ref:`form-login` を参照されたい
    * - \ (4)
      - \ ``<sec:logout>``\ タグ を定義し、ログアウトに関する設定を行う。
        \ 詳細は :ref:`SpringSecurityAuthenticationLogout` を参照されたい。
    * - \ (5)
      - \ ``<sec:access-denied-handler>``\ タグを定義し、アクセスエラー時の制御を行うための設定を定義する。
        \ 詳細は :ref:`SpringSecurityAuthorizationAccessDeniedHandler` 、 :ref:`SpringSecurityAuthorizationOnError` を参照されたい。
    * - \ (6)
      - ログ出力するユーザ情報をMDCに格納するための共通ライブラリのフィルタを定義する。
    * - \ (7)
      - \ ``<sec:session-management>``\ タグ を定義し、セッション管理に関する設定を行う。
        \ 詳細は :ref:`SpringSecuritySessionManagement` を参照されたい
    * - \ (8)
      - \ ``<sec:authentication-manager>``\ タグを定義して、認証機能用のコンポーネントをbean定義する。
        このタグを定義しておかないとサーバ起動時にエラーが発生する。
    * - \ (9)
      - \ アクセスエラー時のエラーハンドリングを行うコンポーネントをbean定義する。
    * - \ (10)
      - \ ログ出力するユーザ情報をMDCにする共通ライブラリのコンポーネントをbean定義する。


.. note:: **静的リソースへのアクセス**

    JSPでCSS等の静的リソースを使用している場合は、それらを格納するフォルダにアクセス権を付与する必要がある。
    詳細は、:ref:`SpringSecurityNotApply` を参照されたい。 

|

作成したbean定義ファイルを使用してSpringのDIコンテナを生成するように定義する。

* xxx-web/src/main/webapp/WEB-INF/web.xmlの設定例

.. code-block:: xml

    <!-- (1) -->
    <listener>
        <listener-class>
            org.springframework.web.context.ContextLoaderListener
        </listener-class>
    </listener>
    <!-- (2) -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            classpath*:META-INF/spring/applicationContext.xml
            classpath*:META-INF/spring/spring-security.xml
        </param-value>
    </context-param>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - \(1)
     -  サーブレットコンテナのリスナクラスとして、\ ``ContextLoaderListener``\ クラスを指定する。
   * - \(2)
     -  サーブレットコンテナの\ ``contextClass``\ パラメータに、\ ``applicationContext.xml``\ に加えて、Spring Security用のbean定義ファイルを追加する。

|

サーブレットフィルタの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
最後に、Spring Securityが提供しているサーブレットフィルタクラス(\ ``FilterChainProxy``\) をサーブレットコンテナに登録する。

* xxx-web/src/main/webapp/WEB-INF/web.xmlの設定例

.. code-block:: xml

    <!-- (1) -->
    <filter>
        <filter-name>springSecurityFilterChain</filter-name>
        <filter-class>
            org.springframework.web.filter.DelegatingFilterProxy
        </filter-class>
    </filter>
    <!-- (2) -->
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
   * - \ (1)
     - Spring Frameworkから提供されている\ ``DelegatingFilterProxy``\ を使用して、
       SpringのDIコンテナで管理されているbean(\ ``FilterChainProxy``\ )をサーブレットコンテナに登録する。
       サーブレットフィルタの名前には、SpringのDIコンテナで管理されているbeanのbean名(\ ``springSecurityFilterChain``\ )を指定する。
   * - \ (2)
     -  Spring Securityを適用するURLのパターンを指定する。
        上記例では、すべてのリクエストに対してSpring Securityを適用する。

|

.. _SpringSecurityNotApply:

セキュリティ対策を適用しないため設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

セキュリティ対策が不要なリソースのパス(cssファイルやimageファイルにアクセスするためのパスなど)に対しては、
\ ``<sec:http>``\ タグを使用して、Spring Securityのセキュリティ機能(Security Filter)が適用されないように制御することができる。

* xxx-web/src/main/resources/META-INF/spring/spring-security.xmlの定義例

.. code-block:: xml
  
    <sec:http pattern="/resources/**" security="none"/>  <!-- (1) (2) -->
    <sec:http>
        <!-- omitted -->
    </sec:http>
  
.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
  
    * - 項番
      - 説明
    * - | (1)
      - | \ ``pattern``\ 属性にセキュリティ機能を適用しないパスのパターンを指定する。
    * - | (2)
      - | \ ``security``\ 属性に\ ``none``\ を指定する。
        | \ ``none``\ を指定すると、Spring Securityのセキュリティ機能(Security Filter)が適用されない。

.. raw:: latex

   \newpage

