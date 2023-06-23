Spring Security概要
================================================================================

.. only:: html

 .. contents:: 目次
    :local:

Overview
--------------------------------------------------------------------------------

| Spring Securityとは、アプリケーションのセキュリティを担う「認証」、「認可」の2つを
| 主な機能として提供している。
| 認証機能とは、なりすましによる不正アクセスに対抗するため、ユーザを識別する機能である。
| 認可機能とは、認証された（ログイン中の）ユーザの権限に応じて、
| システムのリソースに対するアクセス制御を行う機能である。
| また、HTTPヘッダーを付与する機能を有する。

| Spring Securityの概要図を、以下に示す。

.. figure:: ./images/spring_security_overview.png
   :alt: Spring Security Overview
   :width: 80%
   :align: center

   **Picture - Spring Security Overview**

| Spring Securityは、認証、認可のプロセスを何層にも連なる
| ServletFilter の集まりで実現している。
| また、パスワードハッシュ機能や、JSPの認可タグライブラリなども提供している。

認証
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 認証とは、正当性を確認する行為であり、ネットワークやサーバへ接続する際に
| ユーザ名とパスワードの組み合わせを使って、利用ユーザにその権利があるかどうかや、
| その人が利用ユーザ本人であるかどうかを確認することである。
| Spring Securityでの使用方法は、\ :doc:`Authentication`\ を参照されたい。

パスワードハッシュ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 平文のパスワードから、ハッシュ関数を用いて計算されたハッシュ値を、元のパスワードと置き換えることである。
| Spring Securityでの使用方法は、\ :doc:`PasswordHashing`\ を参照されたい。

認可
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 認可とは、認証された利用者がリソースにアクセスしようとしたとき、
| アクセス制御処理でその利用者がそのリソースの使用を許可されていることを調べることである。
| Spring Securityでの使用方法は、\ :doc:`Authorization`\ を参照されたい。

.. _howtouse_springsecurity:

How to use
--------------------------------------------------------------------------------

| Spring Securityを使用するために、以下の設定を定義する必要がある。

pom.xmlの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Spring Securityを使用する場合、以下のdependencyを、pom.xmlに追加する必要がある。

.. code-block:: xml

    <dependency>
        <groupId>org.terasoluna.gfw</groupId>
        <artifactId>terasoluna-gfw-security-core</artifactId>  <!-- (1) -->
    </dependency>

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
   * - | (1)
     - | terasoluna-gfw-security-coreは、webに依存しないため、ドメイン層のプロジェクトから使用する場合は、
       | terasoluna-gfw-security-coreのみをdependencyに追加すること。
   * - | (2)
     - | terasoluan-gfw-webはwebに関連する機能を提供する。terasoluna-gfw-security-coreにも依存しているため、
       | Webプロジェクトは、terasoluna-gfw-security-webのみをdependencyに追加すること。

Web.xmlの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: xml
   :emphasize-lines: 5,13-20

    <context-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>  <!-- (1) -->
          classpath*:META-INF/spring/applicationContext.xml
          classpath*:META-INF/spring/spring-security.xml
      </param-value>
    </context-param>
    <listener>
      <listener-class>
        org.springframework.web.context.ContextLoaderListener
      </listener-class>
    </listener>
    <filter>
      <filter-name>springSecurityFilterChain</filter-name>  <!-- (2) -->
      <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>  <!-- (3) -->
    </filter>
    <filter-mapping>
      <filter-name>springSecurityFilterChain</filter-name>
      <url-pattern>/*</url-pattern>  <!-- (4) -->
    </filter-mapping>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | contextConfigLocationには、applicationContext.xmlに加えて、
       | クラスパスにSpring Security設定ファイルを追加する。本ガイドラインでは、「spring-security.xml」とする。
   * - | (2)
     - | filter-nameには、Spring Securityの内部で使用されるBean名、「springSecurityFilterChain」 で定義すること。
   * - | (3)
     - 各種機能を有効にするための、Spring Securityのフィルタ設定。
   * - | (4)
     - 全てのリクエストに対して設定を有効にする。

spring-security.xmlの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| web.xmlにおいて指定したパスに、spring-security.xmlを配置する。
| 通常はsrc/main/resources/META-INF/spring/spring-security.xmlに設定する。
| 以下の例は、雛形のみであるため、詳細な説明は、次章以降を参照されたい。

* spring-mvc.xml

  .. code-block:: xml

    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:sec="http://www.springframework.org/schema/security"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/security
            http://www.springframework.org/schema/security/spring-security.xsd
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd">
        <sec:http  use-expressions="true">  <!-- (1) -->
        <!-- omitted -->
        </sec:http>
    </beans>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | use-expressions="true"と記載することで、アクセス属性のSpring EL式を有効することができる。

  \

      .. note::
          use-expressions="true" で有効になるSpring EL式は、以下を参照されたい。

          \ `Expression-Based Access Control <http://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/reference/htmlsingle/#el-access>`_\

Appendix
--------------------------------------------------------------------------------

.. _SpringSecurityAppendixSecHeaders:

セキュアなHTTPヘッダー付与の設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

以下のようにspring-security.xmlの\ ``<sec:http>``\ の内の\ ``<sec:headers>``\ 要素を設定することで、HTTPレスポンスに自動でセキュリティに関するヘッダを設定することができる。
これらのHTTPレスポンスヘッダをつけることにより、Webブラウザが攻撃を検知して対処できる。
必須の設定ではないが、セキュリティ強化のために設定しておくことを推奨する。

.. code-block:: xml

    <sec:http use-expressions="true">
      <!-- omitted -->
      <sec:headers />
      <!-- omitted -->
    </sec:http>

本設定で、以下の項目に関するHTTPレスポンスヘッダが設定される。

* Cache-Control
* X-Content-Type-Options
* Strict-Transport-Security
* X-Frame-Options
* X-XSS-Protection

.. tabularcolumns:: |p{0.2\linewidth}|p{0.5\linewidth}||p{0.3\linewidth}|
.. list-table:: 
   :header-rows: 1
   :widths: 20 50 30

   * - HTTPヘッダ名
     - 設定が不適切(未設定含む)な場合の問題
     - 適切に設定した場合の挙動
   * - | \ ``Cache-Control``\ 
     - | あるユーザーがログインして閲覧できるコンテンツがキャッシュされ、ログアウト後に別ユーザーも閲覧できてしまう場合がある。
     - | コンテンツをキャッシュしないように指示をして、ブラウザがサーバの情報を常に取得するようにする。
   * - | \ ``X-Content-Type-Options``\ 
     - | ブラウザが、Content-Typeで内容を決めずにコンテンツの中身を調べて動作させる内容を判断してしまい、想定しないScriptが実行されてしまう場合がある。
     - | ブラウザが、Content-Typeで内容を決めずにコンテンツの中身を調べて動作させる内容を判断しないようにする。MIMEタイプが一致しない場合、Scriptが実行されることを制限する。
   * - | \ ``Strict-Transport-Security``\ 
     - | セキュアなページにHTTPSでアクセスされることを期待しているにも関わらず、HTTPでアクセスされた際に、HTTP由来の攻撃を受ける可能性がある。(例: 中間攻撃者がユーザーのHTTPリクエストを傍受し、悪意のあるサイトへリダイレクトさせる。)
     - | 一度正規のWebサイトへHTTPSでアクセスすれば、ブラウザは自動的にHTTPSのみを用いるよう理解して、悪意のあるサイトへ誘導されるという中間者攻撃の実行を防ぐ。
   * - | \ ``X-Frame-Options``\ 
     - | 悪意あるWebサイトAの画面を透過処理で見えなくし、代わりに\ ``<iframe>``\ タグで他の正常なサイトBを埋め込むと、攻撃者はユーザにサイトBのつもりでサイトAにアクセスさせることができる。
       | この状況において、サイトAの送信ボタンとサイトBのリンクの位置を重ねると、攻撃者はユーザーに、正常なサイトBのリンクをクリックしたつもりでサイトAによる悪意のあるリクエストを送信させることができる。(\ `Clickjacking <https://www.owasp.org/index.php/Clickjacking>`_\ )
     - | 自身の作成したWebサイト(=サイトB)が他のWebサイト(=サイトA)に\ ``<iframe>``\ タグを利用して読み込まれないようにする。
   * - | \ ``X-XSS-Protection``\ 
     - | ブラウザに実装されているXSSフィルターによる有害スクリプトの判定が無効化される。
     - | ブラウザに実装されているXSSフィルターが、有害なスクリプトとを判断して実行するかどうかをユーザに問い合わせる、または無効にする(挙動はブラウザによって異なる)。



上記設定は以下の(1)から(5)のように個別設定も可能である。必要に応じて取捨選択されたい。

.. code-block:: xml

    <sec:http use-expressions="true">
      <!-- omitted -->
      <sec:headers>
        <sec:cache-control />  <!-- (1) -->
        <sec:content-type-options />  <!-- (2) -->
        <sec:hsts />  <!-- (3) -->
        <sec:frame-options />  <!-- (4) -->
        <sec:xss-protection />  <!-- (5) -->
      </sec:headers>
      <!-- omitted -->
    </sec:http>

.. tabularcolumns:: |p{0.05\linewidth}|p{0.45\linewidth}|p{0.40\linewidth}|p{0.10\linewidth}|
.. list-table:: Spring Security によるHTTPヘッダー付与
   :header-rows: 1
   :widths: 5 45 40 10

   * - 項番
     - 説明
     - デフォルトで出力されるHTTPレスポンスヘッダ
     - 属性有無
   * - | (1)
     - | クライアントにデータをキャッシュしないように指示する。
     - | \ ``Cache-Control:no-cache, no-store, max-age=0, must-revalidate``\ 
       | \ ``Pragma: no-cache``\ 
       | \ ``Expires: 0``\ 
     - | 無し
   * - | (2)
     - | コンテントタイプを無視して、クライアント側がコンテンツ内容により、自動的に処理方法を決めないように指示する。
     - | \ ``X-Content-Type-Options:nosniff``\ 
     - | 無し
   * - | (3)
     - | HTTPSでアクセスしたサイトでは、HTTPSの接続を続けるように指示する。（HTTPでのサイトの場合、無視され、ヘッダ項目として付与されない。）
     - | \ ``Strict-Transport-Security:max-age=31536000 ; includeSubDomains``\ 
     - | 有り
   * - | (4)
     - | コンテンツをiframe内部に表示の可否を指示する。
     - | \ ``X-Frame-Options:DENY``\ 
     - | 有り
   * - | (5)
     - | \ `XSS攻撃 <https://www.owasp.org/index.php/Cross-site_Scripting_(XSS)>`_\ を検出できるフィルターが実装されているブラウザに対して、XSSフィルター機能を有効にする指示をする。
     - | \ ``X-XSS-Protection:1; mode=block``\ 
     - | 有り

|

個別設定した場合は属性を設定可能である。設定可能な属性をいくつか説明する。

.. tabularcolumns:: |p{0.05\linewidth}|p{0.20\linewidth}|p{0.30\linewidth}|p{0.20\linewidth}|p{0.25\linewidth}|
.. list-table:: 設定可能な属性
   :header-rows: 1
   :widths: 5 20 30 20 25

   * - 項番
     - オプション
     - 説明
     - 指定例
     - 出力されるHTTPレスポンスヘッダ
   * - | (3)
     - | \ ``max-age-seconds``\ 
     - | 該当サイトに対してHTTPSのみでアクセスすることを記憶する秒数（デフォルトは365日）
     - | \ ``<sec:hsts max-age-seconds="1000" />``\ 
     - | \ ``Strict-Transport-Security:max-age=1000 ; includeSubDomains``\ 
   * - | (3)
     - | \ ``include-subdomains``\ 
     - | サブドメインに対しての適用指示。デフォルト値は\ ``true``\ である。\ ``false``\ を指定すると出力されなくなる。
     - | \ ``<sec:hsts include-subdomains="false" />``\ 
     - | \ ``Strict-Transport-Security:max-age=31536000``\ 
   * - | (4)
     - | \ ``policy``\ 
     - | コンテンツをiframe内部に表示する許可方法を指示する。デフォルト値は\ ``DENY``\ （フレーム内に表示するのを全面禁止）である。\ ``SAMEORIGIN``\ (同サイト内ページのみフレームに読み込みを許可する)にも変更可能である。
     - | \ ``<sec:frame-options policy="SAMEORIGIN" />``\ 
     - | \ ``X-Frame-Options:SAMEORIGIN``\ 
   * - | (5)
     - | \ ``enabled,block``\ 
     - | \ ``false``\ を指定して、XSSフィルターを無効にすることが可能となるが、有効化を推奨する。
     - | \ ``<sec:xss-protection enabled="false" block="false" />``\ 
     - | \ ``X-XSS-Protection:0``\ 


.. note::

    これらのヘッダに対する処理は、一部のブラウザではサポートされていない。ブラウザの公式サイトまたは以下のページを参照されたい。

    * https://www.owasp.org/index.php/HTTP_Strict_Transport_Security (Strict-Transport-Security)
    * https://www.owasp.org/index.php/Clickjacking_Defense_Cheat_Sheet (X-Frame-Options)
    * https://www.owasp.org/index.php/List_of_useful_HTTP_headers (X-Content-Type-Options, X-XSS-Protection)


詳細については\ `公式リファレンス <http://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/reference/htmlsingle/#default-security-headers>`_\ を参照されたい。

    
.. raw:: latex

   \newpage

