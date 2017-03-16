ファイルアップロード
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

.. _FileUploadOverview:

Overview
--------------------------------------------------------------------------------

| 本節では、ファイルをアップロードする方法について、説明する。

| ファイルのアップロードは、Servlet 3.0からサポートされたファイルアップロード機能と、Spring Webから提供されているクラスを利用して行う。

 .. note::

    本節では、Servlet 3.0でサポートされたファイルアップロード機能を使用しているため、Servletのバージョンは、3.0以上であることが前提となる。

 .. note::

    一部のアプリケーションサーバ上でServlet 3.0のファイルアップロード機能を使用すると、
    リクエストパラメータやファイル名のマルチバイト文字が文字化けすることがある。

    version 5.3.0.RELEASE時点で問題の発生が確認されているアプリケーションサーバは以下の通りである。
    
    * WebLogic 12.1.3
    * JBoss EAP 7.0
    * JBoss EAP 6.4.0.GA
    
    このうちJBoss EAP 7.0では、アプリケーションサーバ独自の設定を追加することで問題を回避することができる。
    詳細は、\ `JBoss EAP 7.0を利用する際の注意点 <https://github.com/terasolunaorg/terasoluna-gfw/wiki/JBoss7_ja>`_\を参照されたい。

    その他の問題が発生するアプリケーションサーバを使用する場合は、Commons FileUploadを使用することで問題を回避することができる。
    Commons FileUploadを使用するための設定方法については、「:ref:`file-upload_usage_commons_fileupload`」を参照されたい。

 .. warning::
 
    使用するアプリケーションサーバのファイルアップロードの実装が、Apache Commons FileUploadの実装に依存している場合、\ `CVE-2014-0050 <http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0050>`_\および\ `CVE-2016-3092 <https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-3092>`_\で報告されているセキュリティの脆弱性が発生する可能性がある。
    使用するアプリケーションサーバに同様の脆弱性がない事を確認されたい。
    
    Tomcatを使用する場合、7.0系は7.0.70以上、8.0系は8.0.36以上、8.5系は8.5.3以上を使用する必要がある。

アップロード処理の基本フロー
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Servlet 3.0からサポートされたファイルアップロード機能と、Spring Webのクラスを使って、ファイルをアップロードする際の基本フローを、以下に示す。

 .. figure:: ./images/file-upload-overview_basicflow.png
   :alt: Screen image of single file upload.
   :width: 100%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90
   :class: longtable

   * - 項番
     - 説明
   * - | (1)
     - | アップロードするファイルを選択し、アップロードを実行する。
   * - | (2)
     - | サーブレットコンテナは、\ ``multipart/form-data``\ リクエストを受け取り、\ ``org.springframework.web.multipart.support.MultipartFilter``\ を呼び出す。
   * - | (3)
     - | \ ``MultipartFilter``\ は、 \ ``org.springframework.web.multipart.support.StandardServletMultipartResolver``\ のメソッドを呼び出し、Servlet 3.0のファイルアップロード機能を、Spring MVCで扱えるようにする。
       | \ ``StandardServletMultipartResolver``\ は、Servlet 3.0から導入されたAPI( \ ``javax.servlet.http.Part``\ )をラップする \ ``org.springframework.web.multipart.MultipartFile``\ のオブジェクトを生成する。
   * - | (4)
     - | \ ``MultipartFilter``\ から \ ``DispatcherServlet``\ にフィルタチェーンする。
   * - | (5)
     - | \ ``DispatcherServlet``\ は、Controllerのハンドラメソッドを呼び出す。
       | (3)で生成された \ ``MultipartFile``\ オブジェクトは、 Controllerの引数またはフォームオブジェクトに、バインドされる。
   * - | (6)
     - | Controllerは、 \ ``MultipartFile``\ オブジェクトのメソッドを呼び出し、アップロードされたファイルの中身と、メタ情報(ファイル名など)を取得する。
   * - | (7)
     - | \ ``MultipartFile``\ は、Servlet 3.0から導入された \ ``Part``\ オブジェクトのメソッドを呼び出し、アップロードされたファイルの中身と、メタ情報(ファイル名など)を取得し、Controllerに返却する。
   * - | (8)
     - | Controllerは、Serviceのメソッドを呼び出し、アップロード処理を実行する。
       | \ ``MultipartFile``\ オブジェクトより取得した、ファイルの中身と、メタ情報(ファイル名など)は、Serviceのメソッドの引数として、引き渡す。
   * - | (9)
     - | Serviceは、アップロードされたファイルの中身と、メタ情報(ファイル名など)を、ファイルまたはデータベースに格納する。
   * - | (10)
     - | \ ``MultipartFilter``\ は、 \ ``StandardServletMultipartResolver``\ を呼び出し、Servlet 3.0のファイルアップロード機能で使用される一時ファイルを削除する。
   * - | (11)
     - | \ ``StandardServletMultipartResolver``\ は、Servlet 3.0から導入された \ ``Part``\ オブジェクトのメソッドを呼び出し、ディスクに保存されている一時ファイルを削除する。

 .. raw:: latex

    \newpage

 .. note::

    Controllerでは、Spring Webから提供されている\ ``MultipartFile``\ オブジェクトに対して処理を行うため、Servlet 3.0から提供されたファイルアップロード用のAPIに依存した実装を、排除することができる。


Spring Webから提供されているクラスについて
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Spring Webから提供されているファイルアップロード用のクラスについて、説明する。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.40\linewidth}|p{0.50\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 40 50

   * - | 項番
     - | クラス名
     - | 説明
   * - 1.
     - | org.springframework.web.multipart.
       | MultipartFile
     - | アップロードされたファイルであることを示すインタフェース。
       | 利用するファイルアップロード機能で扱うファイルオブジェクトを、抽象化する役割をもつ。
   * - 2.
     - | org.springframework.web.multipart.support.
       | StandardMultipartHttpServletRequest$
       | StandardMultipartFile
     - | Servlet 3.0から導入されたファイルアップロード機能用の\ ``MultipartFile``\ クラス。
       | Servlet 3.0から導入された\ ``Part``\ オブジェクトに、処理を委譲している。
   * - 3.
     - | org.springframework.web.multipart.
       | MultipartResolver
     - | \ ``multipart/form-data``\ リクエストの解析方法を解決するためのインタフェース。
       | ファイルアップロード機能の、実装に対応する\ ``MultipartFile``\ オブジェクトを生成する役割をもつ。
   * - 4.
     - | org.springframework.web.multipart.support.
       | StandardServletMultipartResolver
     - | Servlet 3.0から導入されたファイルアップロード機能用の\ ``MultipartResolver``\ クラス。
   * - 5.
     - | org.springframework.web.multipart.support.
       | MultipartFilter
     - | multipart/form-dataリクエストの時に、DIコンテナからMultipartResolverを実装するクラスを呼び出し、MultipartFileを生成するクラス。
       | このクラスを使用しないと、ファイルアップロードで許容する最大サイズを超えた場合に、Servlet Filterの処理内でリクエストパラメータを取得できない。
       | そのため、本ガイドラインではMultipartFilterを使用することを推奨している。

 .. tip::

    本ガイドラインでは、Servlet 3.0から導入されたファイルアップロード機能を使うことを前提としているが、Spring Webでは、\ `「Apache Commons FileUpload」用の実装クラスも提供している <http://docs.spring.io/spring/docs/4.3.5.RELEASE/spring-framework-reference/html/mvc.html#mvc-multipart-resolver-commons>`_\ 。
    アップロード処理の実装の違いは、\ ``MultipartResolver``\ と、\ ``MultipartFile``\ オブジェクトによって吸収されるため、Controllerの実装に影響を与えることはない。

|

How to use
--------------------------------------------------------------------------------

.. _file-upload_how_to_usr_application_settings:

アプリケーションの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Servlet 3.0のアップロード機能を有効化するための設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Servlet 3.0のアップロード機能を有効化するために、以下の設定を行う。

- :file:`web.xml`

 .. code-block:: xml
   :emphasize-lines: 11-15

    <web-app xmlns="http://java.sun.com/xml/ns/javaee"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
        version="3.0"> <!-- (1) (2) -->

        <servlet>
            <servlet-class>
                org.springframework.web.servlet.DispatcherServlet
            </servlet-class>
            <!-- omitted -->
            <multipart-config> <!-- (3) -->
                <max-file-size>5242880</max-file-size> <!-- (4) -->
                <max-request-size>27262976</max-request-size> <!-- (5) -->
                <file-size-threshold>0</file-size-threshold> <!-- (6) -->
            </multipart-config>
        </servlet>

        <!-- omitted -->

    </web-app>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90
   :class: longtable

   * - 項番
     - 説明
   * - | (1)
     - | \ ``<web-app>``\ 要素の\ ``xsi:schemaLocation``\ 属性に、Servlet 3.0以上のXSDファイルを指定する。
   * - | (2)
     - | \ ``<web-app>``\ 要素の\ ``version``\ 属性に、\ ``3.0``\ 以上のバージョンを指定する。
   * - | (3)
     - | ファイルアップロードを使用するServletの\ ``<servlet>``\ 要素に、\ ``<multipart-config>``\ 要素を追加する。
   * - | (4)
     - | アップロードを許可する1ファイルの最大バイト数を指定する。
       | 指定がない場合、-1 (制限なし)が設定される。
       | 指定した値を超えた場合、\ ``org.springframework.web.multipart.MultipartException``\ が発生する。
       |
       | 上記例では、 5MBを指定している。
   * - | (5)
     - | \ ``multipart/form-data``\ リクエストのContent-Lengthの最大値を指定する。
       | 指定がない場合、-1 (制限なし)が設定される。
       | 指定した値を超えた場合、\ ``org.springframework.web.multipart.MultipartException``\ が発生する。
       |
       | 本パラメータに設定する値は、以下の計算式で算出される値を設定する必要がある。
       |
       | **(「アップロードを許可する1ファイルの最大バイト数」  * 「同時にアップロードを許可するファイル数」 ) + 「その他のフォーム項目のデータサイズ」 + 「multipart/form-dataリクエストのメタ情報サイズ」**
       |
       | 上記例では、 26MBを指定している。
       | 内訳は、25MB(5MB * 5 files)と、1MB(メタ情報のバイト数 + フォーム項目のバイト数)である。
   * - | (6)
     - | アップロードされたファイルの中身を、一時ファイルとして保存するかの閾値(1ファイルのバイト数)を指定する。
       | このパラメータを明示的に指定しないと ``<max-file-size>`` 要素や ``<max-request-size>`` 要素で指定した値が有効にならないアプリケーションサーバが存在するため、デフォルト値(0)を明示的に指定している。

 .. raw:: latex

    \newpage

 .. warning::

    Dos攻撃に対する攻撃耐性を高めるため、\ ``max-file-size``\ と、\ ``max-request-size``\ は、かならず指定すること。

    Dos攻撃については、\ :ref:`file-upload_security_related_warning_points_dos`\ を参照されたい。


 .. note::

    デフォルトの設定では、アップロードされたファイルは必ず一時ファイルに出力されるが、\ ``<multipart-config>``\ の子要素である\ ``<file-size-threshold>``\ 要素の設定値によって、出力有無を制御することができる。

     .. code-block:: xml

       <!-- omitted -->

       <multipart-config>
           <!-- omitted -->
           <file-size-threshold>32768</file-size-threshold> <!-- (7) -->
       </multipart-config>

       <!-- omitted -->

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (7)
         - | アップロードされたファイルの中身を、一時ファイルとして保存するかの閾値(1ファイルのバイト数)を指定する。
           | 指定がない場合、0が設定される。
           | 指定値を超えるサイズのファイルがアップロードされた場合、アップロードされたファイルは、
           | 一時ファイルとしてディスクに出力され、リクエストが完了した時点で削除される。
           |
           | 上記例では、 32KBを指定している。

     .. warning::

        本パラメータは、以下の点でトレードオフの関係となっているため、\ **システム特性にあった設定値を指定すること。**\

        * 設定値を大きくすると、メモリ内で処理が完結するため、処理性能は向上するが、 Dos攻撃などによって\ ``OutOfMemoryError``\ が発生する可能性が高くなる。
        * 設定値を小さくすると、メモリを使用率を最小限に抑えることができるため、Dos攻撃などによって\ ``OutOfMemoryError``\ が発生する可能性を抑えることができるが、
          ディスクIOの発生頻度が高くなるため、性能劣化が発生する可能性が高くなる。


    一時ファイルの出力ディレクトリを変更したい場合は、\ ``<multipart-config>``\ の子要素である\ ``<location>``\ 要素にディレクトリパスを指定する。

     .. code-block:: xml

       <!-- omitted -->

       <multipart-config>
           <location>/tmp</location> <!-- (8) -->
           <!-- omitted -->
       </multipart-config>

       <!-- omitted -->

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (8)
         - | 一時ファイルを出力するディレクトリのパスを指定する。
           | 省略した場合、アプリケーションサーバの一時ファイルを格納するためのディレクトに出力される。
           |
           | 上記例では、\ ``/tmp``\ を指定している。

     .. warning::

        \ ``<location>``\ 要素で指定するディレクトリは、アプリケーションサーバ(サーブレットコンテナ)が利用するディレクトリであり、**アプリケーションからアクセスする場所ではない。**

        アプリケーションとしてアップロードされたファイルを一時的なファイルとして保存しておきたい場合は、\ ``<location>``\ 要素で指定するディレクトリとは、別のディレクトリに出力すること。

.. _file-upload_setting_servlet_filter:

Servlet Filterの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
multipart/form-dataリクエストの時、ファイルアップロードで許容する最大サイズを超えた場合の動作は、アプリケーションサーバによって異なる。アプリケーションサーバによっては、許容サイズを超えたアップロードの際に発生する\ ``MultipartException``\ が検知されず、後述する例外ハンドリングの設定が有効にならない場合がある。

| この動作は\ ``MiltipartFilter``\ を設定することで回避できるため、本ガイドラインでは\ ``MiltipartFilter``\ の設定を前提として説明を行う。
| 以下に、設定例を示す。

- :file:`web.xml`

 .. code-block:: xml

    <!-- (1) -->
    <filter>
        <filter-name>MultipartFilter</filter-name>
        <filter-class>org.springframework.web.multipart.support.MultipartFilter</filter-class>
    </filter>
    <!-- (2) -->
    <filter-mapping>
        <filter-name>MultipartFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | Servlet Fliterとして \ ``MultipartFilter``\ を定義する。
   * - | (2)
     - | \ ``MultipartFilter``\ を適用するURLのパターンを指定する。
     

 .. warning:: **Spring Security使用時の注意点**

    Spring Securityを使ってセキュリティ対策を行う場合は、\ ``springSecurityFilterChain``\ より前に定義すること。
    また、プロジェクト独自で作成するServlet Filterでリクエストパラメータにアクセスするものがある場合は、そのServlet Filterより前に定義すること。

    ただし、\ ``springSecurityFilterChain``\ より前に定義することで、認証又は認可されていないユーザーからのアップロード(一時ファイル作成)を許容することになる。
    この動作を回避する方法が\ `Spring Security Reference -Cross Site Request Forgery (CSRF)- <http://docs.spring.io/spring-security/site/docs/4.1.4.RELEASE/reference/htmlsingle/#csrf-include-csrf-token-in-action>`_\ の中で紹介されているが、セキュリティ上のリスクを含む回避方法になるため、本ガイドラインでは回避策の適用は推奨していない。

 .. warning:: **ファイルアップロードの許容サイズを超過した場合の注意点**

   ファイルアップロードの許容サイズを超過した場合、WebLogicなど一部のアプリケーションサーバでは、CSRFトークンを取得する前にサイズ超過のエラーが検知され、CSRFトークンチェックが行われないことがある。

 .. note:: **MultipartResolverのデフォルト呼び出し**
    
    \ ``MultipartFilter``\ を使用すると、デフォルトで
    \ ``org.springframework.web.multipart.support.StandardServletMultipartResolver``\ が呼び出される。
    \ ``StandardServletMultipartResolver``\ は、アップロードされたファイルを\ ``org.springframework.web.multipart.MultipartFile``\ として生成し、Controllerの引数およびフォームオブジェクトのプロパティとして、受け取ることができるようにする。


例外ハンドリングの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
許可されないサイズのファイルやマルチパートのリクエストが行われた際に発生する\ ``MultipartException``\ の例外ハンドリングの定義を追加する。

| \ ``MultipartException``\ は、クライアントが指定するファイルサイズに起因して発生する例外なので、クライアントエラー(HTTPレスポンスコード=4xx)として扱うことを推奨する。
| **例外ハンドリングを個別に追加しないと、システムエラー扱いとなってしまうので、かならず定義を追加すること。**

| \ ``MultipartException``\ をハンドリングするための設定は、\ ``MultipartFilter``\ を使用するか否かによって異なる。
| \ ``MultipartFilter``\ を使用する場合は、サーブレットコンテナの\ ``<error-page>``\機能を使って例外ハンドリングを行う。
| 以下に、設定例を示す。

- :file:`web.xml`

 .. code-block:: xml

    <error-page>
        <!-- (1) -->
        <exception-type>org.springframework.web.multipart.MultipartException</exception-type>
        <!-- (2) -->
        <location>/WEB-INF/views/common/error/fileUploadError.jsp</location>
    </error-page>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | ハンドリング対象の例外クラスとして、\ ``MultipartException``\を指定する。
   * - | (2)
     - | \ ``MultipartException``\ が発生した際に表示するファイルを指定する。
       |
       | 上記例では、\ ``"/WEB-INF/views/common/error/fileUploadError.jsp"``\ を指定している。

- :file:`fileUploadError.jsp`

 .. code-block:: jsp

    <%-- (3) --%>
    <% response.setStatus(HttpServletResponse.SC_BAD_REQUEST); %>
    <!DOCTYPE html>
    <html>
    
        <!-- omitted -->

    </html>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (3)
     - | HTTPステータスコードは、\ ``HttpServletResponse``\ のAPIを呼び出して設定する。
       |
       | 上記例では、\ ``"400"``\ (Bad Request) を設定している。
       | 明示的に設定しない場合、HTTPステータスコードは\ ``"500"``\ (Internal Server Error)となる。

|

| \ ``MultipartFilter``\ を使用しない場合は、\ ``SystemExceptionResolver``\を使用して例外ハンドリングを行う。
| 以下に、設定例を示す。

- :file:`spring-mvc.xml`

 .. code-block:: xml

    <bean class="org.terasoluna.gfw.web.exception.SystemExceptionResolver">
        <!-- omitted -->
        <property name="exceptionMappings">
            <map>
                <!-- omitted -->
                <!-- (4) -->
                <entry key="MultipartException"
                       value="common/error/fileUploadError" />

            </map>
        </property>
        <property name="statusCodes">
            <map>
                <!-- omitted -->
                <!-- (5) -->
                <entry key="common/error/fileUploadError" value="400" />
            </map>
        </property>
        <!-- omitted -->
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (4)
     - | \ ``SystemExceptionResolver``\ の\ ``exceptionMappings``\ に、\ ``MultipartException``\ が発生した際に表示するView(JSP)の定義を追加する。
       |
       | 上記例では、\ ``"common/error/fileUploadError"``\ を指定している。
   * - | (5)
     - | ``MultipartException`` が発生した際に応答するHTTPステータスコードの定義を追加する。
       |
       | 上記例では、\ ``"400"``\ (Bad Request) を指定している。
       | クライアントエラー(HTTPレスポンスコード = 4xx)を指定することで、
       | 共通ライブラリの例外ハンドリング機能から提供しているクラス( ``HandlerExceptionResolverLoggingInterceptor`` )によって出力されるログは、\ ``ERROR``\ レベルではなく、\ ``WARN``\ レベルとなる。

|

| \ ``MultipartException``\ に対する例外コードを設ける場合は、例外コードの設定を追加する。
| 例外コードは、共通ライブラリのログ出力機能により出力されるログに、出力される。
| 例外コードは、View(JSP)から参照することもできる。
| View(JSP)から例外コードを参照する方法については、\ :ref:`exception-handling-how-to-use-codingpoint-jsp-exceptioncode-label`\ を参照されたい。

- :file:`applicationContext.xml`

 .. code-block:: xml

    <bean id="exceptionCodeResolver"
        class="org.terasoluna.gfw.common.exception.SimpleMappingExceptionCodeResolver">
        <property name="exceptionMappings">
            <map>
                <!-- (6) -->
                <entry key="MultipartException" value="e.xx.fw.6001" />
                <!-- omitted -->
            </map>
        </property>
        <property name="defaultExceptionCode" value="e.xx.fw.9001" />
        <!-- omitted -->
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (6)
     - | \ ``SimpleMappingExceptionCodeResolver``\ の\ ``exceptionMappings``\ に、\ ``MultipartException``\ が発生した際に適用する、例外コードを追加する。
       |
       | 上記例では、\ ``"e.xx.fw.6001"``\ を指定している。
       | 個別に定義を行わない場合は、\ ``defaultExceptionCode``\ に指定した例外コードが適用される。


単一ファイルのアップロード
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
単一ファイルをアップロードする方法について、説明する。

 .. figure:: ./images/file-upload-how_to_use_single.png
   :alt: Screen image of single file upload.
   :width: 100%

| 単一ファイルの場合は、\ ``org.springframework.web.multipart.MultipartFile``\ オブジェクトを、フォームオブジェクトにバインドして受け取る方法と、Controllerの引数として直接受け取る2つの方法があるが、本ガイドラインでは、フォームオブジェクトにバインドして受け取る方法を推奨する。
| その理由は、アップロードされたファイルの単項目チェックを、Bean Validationの仕組みを使って行うことができるためである。

以下に、フォームオブジェクトにバインドして受け取る方法について、説明する。


フォームの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

 .. code-block:: java

    public class FileUploadForm implements Serializable {

        // omitted

        private MultipartFile file; // (1)

        @NotNull
        @Size(min = 0, max = 100)
        private String description;

        // omitted getter/setter methods.

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | フォームオブジェクトに、\ ``org.springframework.web.multipart.MultipartFile``\ のプロパティを定義する。


JSPの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

 .. code-block:: jsp

    <form:form
      action="${pageContext.request.contextPath}/article/upload" method="post"
      modelAttribute="fileUploadForm" enctype="multipart/form-data"> <!-- (1) (2) -->
      <table>
        <tr>
          <th width="35%">File to upload</th>
          <td width="65%">
            <form:input type="file" path="file" /> <!-- (3) -->
            <form:errors path="file" />
          </td>
        </tr>
        <tr>
          <th width="35%">Description</th>
          <td width="65%">
            <form:input path="description" />
            <form:errors  path="description" />
          </td>
        </tr>
        <tr>
          <td>&nbsp;</td>
          <td><form:button>Upload</form:button></td>
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
     - | \ ``<form:form>``\ 要素のenctype属性に、\ ``"multipart/form-data"``\ を指定する。
   * - | (2)
     - | \ ``<form:form>``\ 要素のmodelAttribute属性に、フォームオブジェクトの属性名を指定する。
       | 上記例では、\ ``"fileUploadForm"``\ を指定している。
   * - | (3)
     - | \ ``<form:input>``\ 要素type属性に、\ ``"file"``\ を指定し、path属性に、\ ``MultipartFile``\ プロパティ名を指定する。
       | 上記例では、アップロードされたファイルは、\ ``FileUploadForm``\ オブジェクトの\ ``"file"``\ プロパティに格納される。


Controllerの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

 .. code-block:: java

    @RequestMapping("article")
    @Controller
    public class ArticleController {

        @Value("${upload.allowableFileSize}")
        private int uploadAllowableFileSize;

        // omitted

        // (1)
        @ModelAttribute
        public FileUploadForm setFileUploadForm() {
            return new FileUploadForm();
        }

        // (2)
        @RequestMapping(value = "upload", method = RequestMethod.GET, params = "form")
        public String uploadForm() {
            return "article/uploadForm";
        }

        // (3)
        @RequestMapping(value = "upload", method = RequestMethod.POST)
        public String upload(@Validated FileUploadForm form,
                BindingResult result, RedirectAttributes redirectAttributes) {

            if (result.hasErrors()) {
                return "article/uploadForm";
            }

            MultipartFile uploadFile = form.getFile();

            // (4)
            if (!StringUtils.hasLength(uploadFile.getOriginalFilename())) {
                result.rejectValue(uploadFile.getName(), "e.xx.at.6002");
                return "article/uploadForm";
            }

            // (5)
            if (uploadFile.isEmpty()) {
                result.rejectValue(uploadFile.getName(), "e.xx.at.6003");
                return "article/uploadForm";
            }

            // (6)
            if (uploadAllowableFileSize < uploadFile.getSize()) {
                result.rejectValue(uploadFile.getName(), "e.xx.at.6004",
                        new Object[] { uploadAllowableFileSize }, null);
                return "article/uploadForm";
            }

            // (7)
            // omit processing of upload.

            // (8)
            redirectAttributes.addFlashAttribute(ResultMessages.success().add(
                    "i.xx.at.0001"));

            // (9)
            return "redirect:/article/upload?complete";
        }

        @RequestMapping(value = "upload", method = RequestMethod.GET, params = "complete")
        public String uploadComplete() {
            return "article/uploadComplete";
        }
    
        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90
   :class: longtable

   * - 項番
     - 説明
   * - | (1)
     - | ファイルアップロード用のフォームオブジェクトを、\ ``Model``\ に格納するためのメソッド。
       | 上記例では、\ ``Model``\ に格納するための属性名は、\ ``"fileUploadForm"``\ となる。
   * - | (2)
     - | アップロード画面を表示するためのハンドラメソッド。
   * - | (3)
     - | ファイルをアップロードするためのハンドラメソッド。
   * - | (4)
     - | アップロードファイルが選択されているかのチェックを行っている。
       | ファイルが選択されたかチェックする場合は、\ ``MultipartFile#getOriginalFilename``\ メソッドを呼び出し、ファイル名の指定有無で判断する。
       | 上記例では、ファイルが選択されていない場合は、入力チェックエラーとしている。
   * - | (5)
     - | 空のファイルが選択されているかのチェックを行っている。
       | 選択されたファイルの中身が空でないことをチェックする場合は、\ ``MultipartFile#isEmpty``\ メソッドを呼び出し、中身の存在チェックを行う。
       | 上記例では、 空のファイルが選択されている場合は、入力チェックエラーとしている。
   * - | (6)
     - | ファイルのサイズが、許容サイズ内かどうかのチェックを行っている。
       | 選択されたファイルのサイズをチェックする場合は、\ ``MultipartFile#getSize``\ メソッドを呼び出し、サイズが許容範囲内かチェックを行う。
       | 上記例では、 ファイルのサイズが許容サイズを超えている場合は、入力チェックエラーとしている。
   * - | (7)
     - | アップロード処理を実装する。
       | 上記例では、具体的な実装は省略しているが、共有ディスクやデータベースへ保存する処理を行うことになる。
   * - | (8)
     - | 要件に応じて、アップロードが成功したことを通知する、処理結果メッセージを格納する。
   * - | (9)
     - | アップロード処理完了後の画面表示は、リダイレクトして表示する。

 .. raw:: latex

    \newpage

 .. note:: **重複アップロードの防止**

    ファイルのアップロードを行う場合は、PRGパターンによる画面遷移と、トランザクショントークンチェックを行うことを推奨する。
    PRGパターンによる画面遷移と、トランザクショントークンチェックを行うことで、重複送信に伴う、同一ファイルのアップロードを防ぐことができる。

    重複送信の防止方法について、詳細は、\ :doc:`../WebApplicationDetail/DoubleSubmitProtection`\ を参照されたい。

 .. note:: **MultipartFileについて**

    MultipartFileには、アップロードされたファイルを操作するためのメソッドが用意されている。
    各メソッドの利用方法については、\ `MultipartFileクラスのJavaDoc <http://docs.spring.io/spring/docs/4.3.5.RELEASE/javadoc-api/org/springframework/web/multipart/MultipartFile.html>`_\ を参照されたい。

.. _fileupload_validator:

ファイルアップロードのBean Validation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| 上記の実装例では、アップロードファイルのバリデーションをControllerの処理として行っていたが、ここでは、Bean Validationの仕組みを使ってバリデーションする方法について説明する。
| バリデーションの詳細は、\ :doc:`Validation`\ を参照されたい。

 .. note::

    Bean Validationの仕組みでチェックすることで、Controllerの処理をシンプルに保つことができるため、Bean Validationの仕組みを使うことを推奨する。


ファイルが選択されていることを検証するためのバリデーションの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

 .. code-block:: java

    // (1)
    @Target({ ElementType.METHOD, ElementType.FIELD, ElementType.ANNOTATION_TYPE })
    @Retention(RetentionPolicy.RUNTIME)
    @Constraint(validatedBy = UploadFileRequiredValidator.class)
    public @interface UploadFileRequired {
        String message() default "{com.examples.upload.UploadFileRequired.message}";
        Class<?>[] groups() default {};
        Class<? extends Payload>[] payload() default {};

        @Target({ ElementType.METHOD, ElementType.FIELD, ElementType.ANNOTATION_TYPE })
        @Retention(RetentionPolicy.RUNTIME)
        @Documented
        @interface List {
            UploadFileRequired[] value();
        }

    }

 .. code-block:: java

    // (2)
    public class UploadFileRequiredValidator implements
        ConstraintValidator<UploadFileRequired, MultipartFile> {

        @Override
        public void initialize(UploadFileRequired constraint) {
        }

        @Override
        public boolean isValid(MultipartFile multipartFile,
            ConstraintValidatorContext context) {
            return multipartFile != null &&
                StringUtils.hasLength(multipartFile.getOriginalFilename());
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | ファイルが、選択されていることを検証するための、アノテーションを作成する。
   * - | (2)
     - | ファイルが、選択されていることを検証するための、実装を行うクラスを作成する。


ファイルが空でないことを検証するためのバリデーションの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

 .. code-block:: java

    // (3)
    @Target({ ElementType.METHOD, ElementType.FIELD, ElementType.ANNOTATION_TYPE })
    @Retention(RetentionPolicy.RUNTIME)
    @Constraint(validatedBy = UploadFileNotEmptyValidator.class)
    public @interface UploadFileNotEmpty {
        String message() default "{com.examples.upload.UploadFileNotEmpty.message}";
        Class<?>[] groups() default {};
        Class<? extends Payload>[] payload() default {};

        @Target({ ElementType.METHOD, ElementType.FIELD, ElementType.ANNOTATION_TYPE })
        @Retention(RetentionPolicy.RUNTIME)
        @Documented
        @interface List {
            UploadFileNotEmpty[] value();
        }

    }

 .. code-block:: java

    // (4)
    public class UploadFileNotEmptyValidator implements
        ConstraintValidator<UploadFileNotEmpty, MultipartFile> {

        @Override
        public void initialize(UploadFileNotEmpty constraint) {
        }

        @Override
        public boolean isValid(MultipartFile multipartFile,
            ConstraintValidatorContext context) {
            if (multipartFile == null ||
                !StringUtils.hasLength(multipartFile.getOriginalFilename())) {
                return true;
            }
            return !multipartFile.isEmpty();
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (3)
     - | ファイルが、空でないことを検証するための、アノテーションを作成する。
   * - | (4)
     - | ファイルが、空でないことを検証するための、実装を行うクラスを作成する。


ファイルのサイズが許容サイズ内であることを検証するためのバリデーションの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

 .. code-block:: java

    // (5)
    @Target({ ElementType.METHOD, ElementType.FIELD, ElementType.ANNOTATION_TYPE })
    @Retention(RetentionPolicy.RUNTIME)
    @Constraint(validatedBy = UploadFileMaxSizeValidator.class)
    public @interface UploadFileMaxSize {
        String message() default "{com.examples.upload.UploadFileMaxSize.message}";
        long value() default (1024 * 1024);
        Class<?>[] groups() default {};
        Class<? extends Payload>[] payload() default {};

        @Target({ ElementType.METHOD, ElementType.FIELD, ElementType.ANNOTATION_TYPE })
        @Retention(RetentionPolicy.RUNTIME)
        @Documented
        @interface List {
            UploadFileMaxSize[] value();
        }

    }

 .. code-block:: java

    // (6)
    public class UploadFileMaxSizeValidator implements
        ConstraintValidator<UploadFileMaxSize, MultipartFile> {

        private UploadFileMaxSize constraint;

        @Override
        public void initialize(UploadFileMaxSize constraint) {
            this.constraint = constraint;
        }

        @Override
        public boolean isValid(MultipartFile multipartFile,
            ConstraintValidatorContext context) {
            if (constraint.value() < 0 || multipartFile == null) {
                return true;
            }
            return multipartFile.getSize() <= constraint.value();
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (5)
     - | ファイルのサイズが、許容サイズ内であることを検証するための、アノテーションを作成する。
   * - | (6)
     - | ファイルのサイズが、許容サイズ内であることを検証するための、実装を行うクラスを作成する。


フォームの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

 .. code-block:: java

    public class FileUploadForm implements Serializable {

        // omitted

        // (7)
        @UploadFileRequired
        @UploadFileNotEmpty
        @UploadFileMaxSize
        private MultipartFile file;

        @NotNull
        @Size(min = 0, max = 100)
        private String description;

        // omitted getter/setter methods.

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (7)
     - | \ ``MultipartFile``\ のフィールドに、アップロードファイルのバリデーションを行うための、アノテーションを付与する。


Controllerの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

 .. code-block:: java

    @RequestMapping(value = "upload", method = RequestMethod.POST)
    public String uploadFile(@Validated FileUploadForm form,
            BindingResult result, RedirectAttributes redirectAttributes) {

        // (8)
        if (result.hasErrors()) {
            return "article/uploadForm";
        }

        MultipartFile uploadFile = form.getFile();

        // omit processing of upload.

        redirectAttributes.addFlashAttribute(ResultMessages.success().add(
                "i.xx.at.0001"));

        return "redirect:/article/upload";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (8)
     - | アップロードファイルのバリデーションの結果は、\ ``BindingResult``\ に格納される。


複数ファイルのアップロード
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
複数ファイルを同時にアップロードする方法について説明する。

 .. figure:: ./images/file-upload-how_to_use_multi.png
   :alt: Screen image of multiple file upload.
   :width: 100%

複数ファイルを同時にアップロードする場合は、\ ``org.springframework.web.multipart.MultipartFile``\ オブジェクトを、フォームオブジェクトにバインドして受け取る必要がある。

以降の説明では、単一ファイルのアップロードと重複する箇所の説明については、省略する。


フォームの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

 .. code-block:: java

    // (1)
    public class FileUploadForm implements Serializable {

        // omitted

        @UploadFileRequired
        @UploadFileNotEmpty
        @UploadFileMaxSize
        private MultipartFile file;

        @NotNull
        @Size(min = 0, max = 100)
        private String description;

        // omitted getter/setter methods.

    }

 .. code-block:: java

    public class FilesUploadForm implements Serializable {

        // omitted

        @Valid // (2)
        private List<FileUploadForm> fileUploadForms; // (3)

        // omitted getter/setter methods.

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | ファイル単位の情報(アップロードファイル自体と、関連するフォーム項目)を保持するクラス。
       | 上記例では、単一ファイルのアップロードの説明で作成したフォームオブジェクトを再利用している。
   * - | (2)
     - | リスト内で保持しているオブジェクトに対して、Bean Validationによる入力チェックを行うために、\ ``@Valid``\ アノテーションを付与する。
   * - | (3)
     - | ファイル単位の情報(アップロードファイル自体と、関連するフォーム項目)を保持するオブジェクトを、List型のプロパティとして定義する。

 .. note::

   ファイルのみアップロードする場合は、\ ``MultipartFile``\ オブジェクトを、List型のプロパティとして定義することもできるが、
   Bean Validationを使用してアップロードファイルの入力チェックを行う場合は、ファイル単位の情報を保持するオブジェクトを、List型のプロパティとして定義する方が相性がよい。


JSPの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

 .. code-block:: jsp

    <form:form
      action="${pageContext.request.contextPath}/article/uploadFiles" method="post"
      modelAttribute="filesUploadForm" enctype="multipart/form-data">
      <table>
        <tr>
          <th width="35%">File to upload</th>
          <td width="65%">
            <form:input type="file" path="fileUploadForms[0].file" /> <!-- (1) -->
            <form:errors path="fileUploadForms[0].file" />
          </td>
        </tr>
        <tr>
          <th width="35%">Description</th>
          <td width="65%">
            <form:input path="fileUploadForms[0].description" />
            <form:errors  path="fileUploadForms[0].description" />
          </td>
        </tr>
      </table>
      <table>
        <tr>
          <th width="35%">File to upload</th>
          <td width="65%">
            <form:input type="file" path="fileUploadForms[1].file" /> <!-- (1) -->
            <form:errors path="fileUploadForms[1].file" />
          </td>
        </tr>
        <tr>
          <th width="35%">Description</th>
          <td width="65%">
            <form:input path="fileUploadForms[1].description" />
            <form:errors path="fileUploadForms[1].description" />
          </td>
        </tr>
      </table>
      <div>
        <form:button>Upload</form:button>
      </div>
    </form:form>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | アップロードファイルをバインドするList内の位置を指定する。
       | バインドするリスト内の位置は、\ ``[]``\ の中に指定する。開始位置は、\ ``0``\ 開始となる。


Controllerの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

 .. code-block:: java

    @RequestMapping(value = "uploadFiles", method = RequestMethod.POST)
    public String uploadFiles(@Validated FilesUploadForm form,
            BindingResult result, RedirectAttributes redirectAttributes) {

        if (result.hasErrors()) {
            return "article/uploadForm";
        }

        // (1)
        for (FileUploadForm fileUploadForm : form.getFileUploadForms()) {

            MultipartFile uploadFile = fileUploadForm.getFile();

            // omit processing of upload.

        }

        redirectAttributes.addFlashAttribute(ResultMessages.success().add(
                "i.xx.at.0001"));

        return "redirect:/article/upload?complete";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | ファイル単位の情報(アップロードファイル自体と関連するフォーム項目)を保持するオブジェクトから ``MultipartFile`` を取得し、アップロード処理を実装する。
       | 上記例では、具体的な実装は省略しているが、共有ディスクやデータベースへ保存する処理を行うことになる。


HTML5のmultiple属性を使った複数ファイルのアップロード
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
HTML5でサポートされたinputタグのmultiple属性を使用して、複数ファイルを同時にアップロードする方法について説明する。

 .. figure:: ./images/file-upload-how_to_use_multi_html5.png
   :alt: Screen image of multiple file upload(html5).
   :width: 100%

以降の説明では、単一ファイルのアップロード及び複数ファイルのアップロードと重複する箇所の説明については、省略する。

フォームの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
HTML5のinputタグのmultiple属性を使用して、複数ファイルを同時にアップロードする場合は、\ ``org.springframework.web.multipart.MultipartFile``\ オブジェクトのコレクションを、フォームオブジェクトにバインドして受け取る必要がある。

 .. code-block:: java

    // (1)
    public class FilesUploadForm implements Serializable {
    
        // omitted
    
        // (2)
        @UploadFileNotEmpty
        private List<MultipartFile> files;
    
        // omitted getter/setter methods.
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 複数のアップロードファイルを保持するためのフォームオブジェクト。
   * - | (2)
     - | ``MultipartFile`` クラスをリストとして宣言する。
       | 上記例では、入力チェックとして、ファイルが空でないことを検証するためのアノテーションを指定している。
       | 本来は他の必須チェックやファイルのサイズチェックなども必要であるが、上記例では割愛している。

Validatorの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
コレクションに格納されている複数の ``MultipartFile`` オブジェクトに対して入力チェックを行う場合は、コレクション用のValidatorを実装する必要がある。

以下では、単一ファイル用に作成したValidatorを利用してコレクション用のValidatorを作成する方法について説明する。

 .. code-block:: java

    // (1)
    public class UploadFileNotEmptyForCollectionValidator implements
        ConstraintValidator<UploadFileNotEmpty, Collection<MultipartFile>> {
    
        // (2)
        private final UploadFileNotEmptyValidator validator = 
            new UploadFileNotEmptyValidator();

        // (3)
        @Override
        public void initialize(UploadFileNotEmpty constraintAnnotation) {
            validator.initialize(constraintAnnotation);
        }
    
        // (4)
        @Override
        public boolean isValid(Collection<MultipartFile> values,
                ConstraintValidatorContext context) {
            for (MultipartFile file : values) {
                if (!validator.isValid(file, context)) {
                    return false;
                }
            }
            return true;
        }
    
    }
    
 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 全てのファイルが空でないことを検証するための実装を行うクラス。
       | 検証対象となる値の型として、 ``Collection<MultipartFile>`` を指定する。
   * - | (2)
     - | 実際の処理は単一ファイル用のValidatorに委譲するため、単一ファイル用のValidatorのインスタンスを作成しておく。
   * - | (3)
     - | Validatorを初期化する。
       | 上記例では、実際の処理を行う単一ファイル用のValidatorの初期化を行っている。
   * - | (4)
     - | 全てのファイルが空でないことを検証する。
       | 上記例では、単一ファイル用のValidatorのメソッドを呼び出して、１ファイルずつ検証を行っている。

 .. code-block:: java

    @Target({ ElementType.METHOD, ElementType.FIELD, ElementType.ANNOTATION_TYPE })
    @Retention(RetentionPolicy.RUNTIME)
    @Constraint(validatedBy = 
        {UploadFileNotEmptyValidator.class,
         UploadFileNotEmptyForCollectionValidator.class}) // (5)
    public @interface UploadFileNotEmpty {
        
        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (5)
     - | 複数のファイルに対してチェックを行うValidatorクラスを、検証用アノテーションに追加する。
       | ``@Constraint`` アノテーションのvalidatedBy属性に、(1)で作成したクラスを指定する。
       | こうすることで、  ``@UploadFileNotEmpty`` アノテーションを付与したプロパティに対する妥当性チェックを行う際に、(1)で作成したクラスが実行される。


JSPの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

 .. code-block:: jsp

    <form:form
      action="${pageContext.request.contextPath}/article/uploadFiles" method="post"
      modelAttribute="filesUploadForm2" enctype="multipart/form-data">
      <table>
        <tr>
          <th width="35%">File to upload</th>
          <td width="65%">
            <form:input type="file" path="files" multiple="multiple" /> <!-- (1) -->
            <form:errors path="files" />
          </td>
        </tr>
      </table>
      <div>
        <form:button>Upload</form:button>
      </div>
    </form:form>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | path属性には フォームオブジェクトのプロパティ名を指定し、 multiple属性を指定する。
       | multiple属性を指定すると、HTML5をサポートしているブラウザで複数のファイルを選択しアップロードすることができる。


Controllerの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

 .. code-block:: java

    @RequestMapping(value = "uploadFiles", method = RequestMethod.POST)
    public String uploadFiles(@Validated FilesUploadForm form,
            BindingResult result, RedirectAttributes redirectAttributes) {
        if (result.hasErrors()) {
            return "article/uploadForm";
        }

        // (1)
        for (MultipartFile file : form.getFiles()) {

            // omit processing of upload.

        }

        redirectAttributes.addFlashAttribute(ResultMessages.success().add(
                "i.xx.at.0001"));

        return "redirect:/article/upload?complete";
    }
    
 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | フォームオブジェクトから ``MultipartFile`` オブジェクトが格納されているリストを取得し、アップロード処理を実装する。
       | 上記例では、具体的な実装は省略しているが、共有ディスクやデータベースへ保存する処理を行うことになる。

仮アップロード
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
アップロード結果の確認画面など、画面遷移の途中でファイルをアップロードする場合、仮アップロードという考え方が必要になる。

 .. note::

    \ ``MultipartFile``\ オブジェクトで保持しているファイルの中身は、アップロードしたリクエストが完了した時点で消滅する可能性がある。
    そのため、ファイルの中身をリクエストを跨いで扱いたい場合は、\ ``MultipartFile``\ オブジェクトで保持しているファイルの中身や、メタ情報(ファイル名など)をファイルやフォームに退避する必要がある。

    \ ``MultipartFile``\ オブジェクトで保持しているファイルの中身は、下記処理フローの(3)が完了した時点で、消滅する。

 .. figure:: ./images/file-upload-how_to_use_temporary_upload.png
   :alt: Processing flow of temporary upload.
   :width: 100%

 .. raw:: latex

    \newpage

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90
   :class: longtable

   * - 項番
     - 説明
   * - | (1)
     - | 入力画面にて、アップロードするファイルを選択し、確認画面に遷移するためのリクエストを送信する。
   * - | (2)
     - | Controllerは、アップロードされたファイルの中身を、アプリケーション用の仮ディレクトリに一時保存する。
   * - | (3)
     - | Controllerは、確認画面のView名を返却し、確認画面に遷移する。
   * - | (4)
     - | 確認画面にて、処理を実行するためのリクエストを送信する。
   * - | (5)
     - | Controllerは、Serviceのメソッドを呼び出し、処理を実行する。
   * - | (6)
     - | Serviceは、仮ディレクトリに格納されている一時ファイルを、本ディレクトリまたはデータベースに移動する。
   * - | (7)
     - | Controllerは、完了画面を表示するためのView名を返却し、完了画面に遷移する。

 .. raw:: latex

    \newpage

 .. note::

    仮アップロードの処理は、アプリケーション層の役割なので、Controller又はHelperクラスで実装することになる。


Controllerの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
以下に、アップロードされたファイルを仮ディレクトリに一時保存する実装例を示す。

 .. code-block:: java

    @Component
    public class UploadHelper {

        // (2)
        @Value("${app.upload.temporaryDirectory}")
        private File uploadTemporaryDirectory;

        // (1)
        public String saveTemporaryFile(MultipartFile multipartFile) 
            throws IOException {

            String uploadTemporaryFileId = UUID.randomUUID().toString();
            File uploadTemporaryFile =
                new File(uploadTemporaryDirectory, uploadTemporaryFileId);

            // (2)
            FileUtils.copyInputStreamToFile(multipartFile.getInputStream(),
                    uploadTemporaryFile);

            return uploadTemporaryFileId;
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 仮アップロードを行うためのメソッドをHelperクラスに作成する。
       | ファイルアップロードを行う処理が複数ある場合は、共通的なHelperメソッドを用意し、仮アップロード処理を共通化することを推奨する。
   * - | (2)
     - | アップロードしたファイルを一時ファイルとして保存する。
       | 上記例では、\ ``org.apache.commons.io.FileUtils``\ クラスの copyInputStreamToFileメソッドを呼び出し、アップロードしたファイルの中身をファイルに保存している。

 .. code-block:: java

    // omitted
    
    @Inject
    UploadHelper uploadHelper;

    @RequestMapping(value = "upload", method = RequestMethod.POST, params = "confirm")
    public String uploadConfirm(@Validated FileUploadForm form,
            BindingResult result) throws IOException {

        if (result.hasErrors()) {
            return "article/uploadForm";
        }

        // (3)
        String uploadTemporaryFileId = uploadHelper.saveTemporaryFile(form
                .getFile());

        // (4)
        form.setUploadTemporaryFileId(uploadTemporaryFileId);
        form.setFileName(form.getFile().getOriginalFilename());

        return "article/uploadConfirm";
    }
    
 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (3)
     - | アップロードファイルを一時保存するためのHelperメソッドを呼び出す。
       | 上記例では、一時保存したファイルの識別するためのIDがHelperメソッドの返り値として返却される。
   * - | (4)
     - | アップロードしたファイルのメタ情報（ファイルを識別するためのID、ファイル名など）をフォームオブジェクトに格納する。
       | 上記例では、アップロードファイルのファイル名と一時保存したファイルを識別するためのIDをフォームオブジェクトに格納している。

 .. note::

    仮ディレクトリのディレクトリは、アプリケーションをデプロイする環境によって異なる可能性があるため、外部プロパティから取得すること。
    外部プロパティの詳細については、\ :doc:`../GeneralFuncDetail/PropertyManagement`\ を参照されたい。

 .. warning::
 
    上記例では、アプリケーションサーバ上のローカルディスクに一時保存する例としているが、アプリケーションサーバがクラスタ化されている場合は、
    データベース又は共有ディスクに保存する必要がでてくるので、非機能要件も考慮して保存先を設計する必要がある。
    
    データベースに保存する場合は、トランザクション管理が必要となるため、 データベースに保存す処理をServiceのメソッドに委譲することになる。

|

How to extend
--------------------------------------------------------------------------------

.. _file-upload_how_to_use_housekeeping:

仮アップロード時の不要ファイルのHousekeeping
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 仮アップロードの仕組みを使用してファイルのアップロードを行う場合、仮ディレクトリに不要なファイルが残るケースがある。
| 具体的には、以下のようなケースである。

* 仮アップロード後の画面操作を中止した場合
* 仮アップロード後の画面操作中にシステムエラーが発生した場合
* 仮アップロード後の画面操作中にサーバが停止した場合
* etc ...

 .. warning::

    不要なファイルを残したままにすると、ディスクを圧迫する可能性があるため、必ず不要なファイルを削除する仕組みを用意すること。

本ガイドラインでは、Spring Frameworkから提供されている「Task Scheduler」機能を使用して、不要なファイルを削除する方法について説明する。
「Task Scheduler」の詳細については、\ `公式リファレンスの"Task Execution and Scheduling" <http://docs.spring.io/spring/docs/4.3.5.RELEASE/spring-framework-reference/html/scheduling.html>`_\ を参照されたい。

 .. note::

    ガイドラインとしては、Spring Frameworkから提供されている「Task Scheduler」機能を使用する方法について説明するが、使用を強制するものではない。
    実際のプロジェクトでは、インフラチームによって不要なファイルを削除するバッチアプリケーション(Shellアプリケーション)が提供されるケースがある。
    その場合は、インフラチーム作成のバッチアプリケーションを使用して不要なファイルを削除することを推奨する。


不要ファイルを削除するコンポーネントクラスの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
不要なファイルを削除するコンポーネントクラスを実装する。

 .. code-block:: java

    package com.examples.common.upload;

    import java.io.File;
    import java.util.Collection;
    import java.util.Date;
    
    import javax.inject.Inject;
    
    import org.apache.commons.io.FileUtils;
    import org.apache.commons.io.filefilter.FileFilterUtils;
    import org.apache.commons.io.filefilter.IOFileFilter;
    import org.springframework.beans.factory.annotation.Value;
    import org.terasoluna.gfw.common.date.jodatime.JodaTimeDateFactory;
    
    // (1)
    public class UnnecessaryFilesCleaner {
    
        @Inject
        JodaTimeDateFactory dateFactory;
    
        @Value("${app.upload.temporaryFileSavedPeriodMinutes}")
        private int savedPeriodMinutes;
    
        @Value("${app.upload.temporaryDirectory}")
        private File targetDirectory;
    
        // (2)
        public void cleanup() {
    
            // calculate cutoff date.
            Date cutoffDate = dateFactory.newDateTime().minusMinutes(
                    savedPeriodMinutes).toDate();
    
            // collect target files.
            IOFileFilter fileFilter = FileFilterUtils.ageFileFilter(cutoffDate);
            Collection<File> targetFiles = FileUtils.listFiles(targetDirectory,
                    fileFilter, null);
    
            if (targetFiles.isEmpty()) {
                return;
            }
    
            // delete files.
            for (File targetFile : targetFiles) {
                FileUtils.deleteQuietly(targetFile);
            }
    
        }
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 不要なファイルを削除するためのコンポーネントクラスを作成する。
   * - | (2)
     - | 不要なファイルを削除するメソッドを実装する。
       | 上記例では、ファイルの最終更新日時から、一定期間更新がないファイルを、不要ファイルとして削除している。

 .. note::

    削除対象ファイルが格納されているディレクトリのパスや、削除基準となる時間などは、アプリケーションをデプロイする環境によって異なる可能性があるため、外部プロパティから取得すること。
    外部プロパティの詳細については、\ :doc:`../GeneralFuncDetail/PropertyManagement`\ を参照されたい。


不要ファイルを削除する処理のスケジューリング設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
不要ファイルを削除するPOJOクラスを、bean登録とタスクスケジュールの設定を行う。

- :file:`applicationContext.xml`

 .. code-block:: xml

    <!-- omitted -->

    <!-- (3) -->
    <bean id="uploadTemporaryFileCleaner"
        class="com.examples.common.upload.UnnecessaryFilesCleaner" />

    <!-- (4) -->
    <task:scheduler id="fileCleanupTaskScheduler" />

    <!-- (5) -->
    <task:scheduled-tasks scheduler="fileCleanupTaskScheduler">
        <!-- (6)(7)(8) -->
        <task:scheduled ref="uploadTemporaryFileCleaner"
                        method="cleanup"
                        cron="${app.upload.temporaryFilesCleaner.cron}"/>
    </task:scheduled-tasks>

    <!-- omitted -->


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (3)
     - | 不要ファイルを削除するPOJOクラスをbean登録する。
       | 上記例では、 ``"uploadTemporaryFileCleaner"`` というIDで登録している。
   * - | (4)
     - | 不要ファイルを削除する処理を、実行するためのタスクスケジューラのbeanを、登録する。
       | 上記例では、pool-size属性を省略しているため、このタスクスケジュールは、シングルスレッドでタスクを実行する。
       | 複数のタスクを同時に実行する必要がある場合は、 pool-size属性に任意の数字を指定すること。
   * - | (5)
     - | 不要ファイルを削除するタスクスケジューラに、タスクを追加する。
       | 上記例では、(4)でbean登録したタスクスケジューラに対して、タスクを追加している。
   * - | (6)
     - | ref属性に、不要ファイルを削除する処理が実装されているbeanを、指定する。
       | 上記例では、(3)で登録したbeanを指定している。
   * - | (7)
     - | method属性に、不要ファイルを削除する処理が実装されているメソッド名を、指定する。
       | 上記例では、(3)で登録したbeanのcleanupメソッドを指定している。
   * - | (8)
     - | cron属性に、不要ファイルを削除する処理の実行タイミングを指定する。
       | 上記例では、外部プロパティよりcron定義を取得している。

 .. note::

    cron属性の設定値は、「秒 分 時 月 年 曜日」の形式で指定する。

    設定例）

     * ``0 */15 * * * *`` : 毎時 0分,15分,30分,45分に実行される。
     * ``0 0 * * * *`` : 毎時 0分に実行される。
     * ``0 0 9-17 * * MON-FRI`` : 平日9時～17時の間の毎時0分に実行される。

    cronの指定値の詳細については、\ `CronSequenceGeneratorのJavaDoc <http://docs.spring.io/spring/docs/4.3.5.RELEASE/javadoc-api/org/springframework/scheduling/support/CronSequenceGenerator.html>`_\ を参照されたい。

    実行タイミングは、アプリケーションをデプロイする環境によって異なる可能性があるため、外部プロパティから取得すること。
    外部プロパティの詳細については、\ :doc:`../GeneralFuncDetail/PropertyManagement`\ を参照されたい。

 .. tip::

    上記例では、タスクの実行トリガーとしてcronを使用しているが、cron以外に、fixed-delayとfixed-rateが、デフォルトで用意されているので、要件に応じて使い分けること。

    デフォルトで用意されているトリガーでは要件を満たせない場合は、trigger属性に\ ``org.springframework.scheduling.Trigger``\ を実装したbeanを指定することで、独自のトリガーを設けることもできる。

|

Appendix
--------------------------------------------------------------------------------
ファイルアップロードに関するセキュリティ問題への考慮
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| ファイルのアップロード機能を提供する場合、以下のようなセキュリティ問題を考慮する必要がある。

#. :ref:`file-upload_security_related_warning_points_dos`
#. :ref:`file-upload_security_related_warning_points_server_scripting`
#. :ref:`file-upload_security_related_warning_points_directory_traversal`

以下に、対策方針について説明する。


.. _file-upload_security_related_warning_points_dos:

アップロード機能に対するDos攻撃
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
アップロード機能に対するDos攻撃とは、巨大なサイズのファイルを連続してアップロードしてサーバに対して負荷を掛けることで、
サーバのダウンや、レスポンス速度の低下を狙った攻撃方法のことである。

| アップロード可能なファイルのサイズに制限がない場合や、マルチパートリクエストのサイズに制限がない場合、Dos攻撃への耐性が脆弱となる。
| Dos攻撃の耐性を高めるためには、\ :ref:`file-upload_how_to_usr_application_settings`\ で説明した\ ``<multipart-config>``\ 要素を用いて、リクエストのサイズ制限を設ける必要がある。

|

.. _file-upload_security_related_warning_points_server_scripting:

アップロードしたファイルをWebサーバ上で実行する攻撃
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| アップロードしたファイルをWebサーバ上で実行する攻撃とは、Webサーバ(アプリケーションサーバ)で実行可能なスクリプトファイル(php, asp, aspx, jspなど)をアップロードし実行することで、Webサーバ内のファイルの閲覧・改竄・削除を行う攻撃方法のことである。
| また、Webサーバを踏み台とすることで、Webサーバと同一ネットワーク上に存在する別のサーバに対して、攻撃することもできる。

この攻撃への対策方法は、以下の通りである。

* アップロードされたファイルを、Webサーバ(アプリケーションサーバ)上の公開ディレクトリに配置せず、ファイルの中身を表示するための処理を経由して、アップロードしたファイルの中身を閲覧させる。
* アップロード可能なファイルの拡張子を制限し、Webサーバ(アプリケーションサーバ)で実行可能なスクリプトファイルが、アップロードされないようにする。

いずれかの対策を行うことで攻撃を防ぐことができるが、両方とも対策しておくことを推奨する。

|

.. _file-upload_security_related_warning_points_directory_traversal:

ディレクトリトラバーサル攻撃
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| ディレクトリトラバーサル攻撃とは、"../" などの文字列が含まれる入力を用いてファイルシステムにアクセスすることにより、サーバ上の本来アクセスさせるべきでないファイルにアクセスされてしまう攻撃である。
| 例えば、ユーザからアップロードされたファイルをサーバ上の所定のディレクトリに配置するWebアプリケーションでは、実装方法によっては"../../../../somewhere/attack" というファイル名のファイルがアップロードされた際に所定外のディレクトリにファイルが配置されてしまう。
| その場合、攻撃者からアップロードされたファイルによってサーバ上のファイルが改ざんされてしまう恐れがある。
| ファイルアップロード機能を提供する場合の他、ファイルダウンロード機能を提供する際にもディレクトリトラバーサル攻撃のリスクがある。
| これは例えば、ユーザからの入力されたファイル名に従ってファイルをダウンロードするWebアプリケーションにおいて、"../../../../etc/passwd" と入力されることで攻撃者に"/etc/passwd" の内容を取得されてしまうといった攻撃が考えらえれる。

この攻撃への対策方法は、以下の通りである。

* アップロードされたファイルをサーバ上に保存する際には、オリジナルのファイル名やユーザからの入力値を使用せず、別の名前で保存する。オリジナルのファイル名についてはサーバ上のファイル名との対応関係をDB等の外部に保存するなど、実際のファイルアクセスに利用されない形で保存しておく。
* サーバ上のファイルにアクセスさせる際は、実際のファイル名ではなくリクエスト用の識別名を介してリクエストさせ、サーバ側で対応するファイル名に変換する。例えば、実際のファイル名"file_A", "file_B" に対してそれぞれ"id01", "id02" という識別子を対応させ、クライアント側から"id01" へのリクエストがあればサーバ側で対応する"file_A" というファイル名に変換してアクセスする。

.. tip::
   
   入力されたファイルパスを正規化（ "./" や "../" 等、ファイルシステム上で特別な意味を持つ文字列を含まない形式に展開すること）し、あらかじめ定めておいたパスと前方一致するかどうかをチェックすることでアクセスを許可するかどうか判断するという対策方法も考えられる。
   しかしながら、入力値のエンコーディングやOSごとのパス形式の違いを考慮すると、あらゆる場合において正しく正規化されるかどうかを確認することは困難である。
   そのため、基本的にはユーザからの入力値を使用したファイルシステムへのアクセスは回避することが望ましい。

.. _file-upload_usage_commons_fileupload:

Commons FileUpload を使用したファイルのアップロード
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
一部のアプリケーションサーバ上でServlet 3.0のファイルアップロード機能を使用すると、
リクエストパラメータやファイル名のマルチバイト文字が文字化けすることがある。

具体例としては、WebLogic 12.1.3でServlet 3.0のファイルアップロード機能を使用すると、
ファイルと一緒に送信するフィールドのマルチバイト文字が文字化けすることが確認されている。
なお、WebLogic 12.2.1では修正されている。

**この問題は、Commons FileUploadを使用することで回避できるため、
問題が発生する特定環境向けの暫定対処として、
Commons FileUploadを使用したファイルのアップロードについて説明する。
問題が発生しない環境でのCommons FileUploadの使用は推奨しない。**

Commons FileUploadを使用する場合は以下の設定を行う。

|

:file:`xxx-web/pom.xml`

.. code-block:: xml

    <!-- (1) -->
    <dependency>
        <groupId>commons-fileupload</groupId>
        <artifactId>commons-fileupload</artifactId>
    </dependency>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | 項番
     - | 説明
   * - | (1)
     - | \ ``commons-fileupload``\ への依存関係を追加する。

.. note::

   上記設定例は、依存ライブラリのバージョンを親プロジェクトである terasoluna-gfw-parent で管理する前提であるため、pom.xmlでのバージョンの指定は不要である。
   上記の依存ライブラリはterasoluna-gfw-parentが利用している\ `Spring IO Platform <http://platform.spring.io/platform/>`_\ で定義済みである。

.. warning::

    Apache Commons FileUploadを使用する場合、
    \ `CVE-2014-0050 <http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0050>`_\および\ `CVE-2016-3092 <https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-3092>`_\で報告されているセキュリティの脆弱性が発生する可能性がある。
    使用するApache Commons FileUploadのバージョンに脆弱性がない事を確認されたい。

    Apache Commons FileUploadを使用する場合、1.3.2以上を使用する必要がある。

    なお、TERASOLUNA Server Framework for Java version 5.3.0.RELEASEが準拠しているSpring IO Platform Athens-SR2.RELEASEで管理されているバージョンを使用すれば、CVE-2014-0050およびCVE-2016-3092で報告されている脆弱性は発生しない。
    意図的にApache Commons FileUploadのバージョンを変更する場合は、当該脆弱性が対処されているバージョンを指定すること。

|

:file:`xxx-web/src/main/resources/META-INF/spring/applicationContext.xml`

.. code-block:: xml

    <!-- (1) -->
    <bean id="filterMultipartResolver"
        class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <property name="maxUploadSize" value="10240000" /><!-- (2) -->
    </bean>

    <!-- ... -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | 項番
     - | 説明
   * - | (1)
     - | Commons FileUploadを使用した\ ``MultipartResolver``\ 実装である\ ``CommonsMultipartResolver``\のbean定義を行う。
       | bean IDには\ ``"filterMultipartResolver"``\ を指定する。
   * - | (2)
     - | ファイルアップロードで許容する最大サイズを設定する。
       | Commons FileUploadに場合、最大値はヘッダ含めたリクエスト全体のサイズであることに注意すること。
       | また、**デフォルト値は-1(無制限)なので、必ず値を設定すること。** その他のプロパティは\ `JavaDoc <http://docs.spring.io/spring-framework/docs/4.3.5.RELEASE/javadoc-api/org/springframework/web/multipart/commons/CommonsMultipartResolver.html>`_\ を参照されたい。

.. warning::

    Commons Fileuploadを使用する場合は、\ ``MultipartResolver``\ の定義を\ :file:`spring-mvc.xml`\ ではなく、\ :file:`applicationContext.xml`\ に行う必要がある。
    \ :file:`spring-mvc.xml`\ に定義がある場合は削除すること。


|

:file:`xxx-web/src/main/webapp/WEB-INF/web.xml`

.. code-block:: xml

    <web-app xmlns="http://java.sun.com/xml/ns/javaee"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
        version="3.0">

        <servlet>
            <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
            <!-- omitted -->
            <!-- (1) -->
            <!-- <multipart-config>...</multipart-config> -->
        </servlet>

        <!-- (2) -->
        <filter>
            <filter-name>MultipartFilter</filter-name>
            <filter-class>org.springframework.web.multipart.support.MultipartFilter</filter-class>
        </filter>
        <filter-mapping>
            <filter-name>MultipartFilter</filter-name>
            <url-pattern>/*</url-pattern>
        </filter-mapping>

        <!-- omitted -->

    </web-app>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | Commons FileUploadを使用する場合、Servlet 3.0のアップロード機能を無効にする必要がある。
       | \ ``DispatcherServlet``\ の定義の中に\ ``<multipart-config>``\ 要素がある場合は、必ず削除すること。
   * - | (2)
     - | Commons Fileuploadを使用する場合、Spring Securityを使ったセキュリティ対策を有効にするために\ ``MultipartFilter``\ を定義する必要がある。
       | \ ``MultipartFilter``\ のマッピング定義は、springSecurityFilterChain(Spring SecurityのServlet Filter)の定義より前に行うこと。

.. tip::

    \ ``MultipartFilter``\ は、DIコンテナ(:file:`applicationContext.xml`)から\ ``"filterMultipartResolver"``\ というbean IDで登録されている\ ``MultipartResolver``\ を取得して、
    ファイルアップロード処理を行う仕組みになっている。

|

.. raw:: latex

   \newpage

