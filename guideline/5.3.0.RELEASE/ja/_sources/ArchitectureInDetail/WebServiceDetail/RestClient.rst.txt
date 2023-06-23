RESTクライアント（HTTPクライアント）
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

.. _RestClientOverview:

Overview
--------------------------------------------------------------------------------

本節では、Spring Frameworkが提供する\ ``org.springframework.web.client.RestTemplate``\ を使用してRESTful Web Service(REST API)を呼び出す実装方法について説明する。

.. _RestClientOverviewRestTemplate:

``RestTemplate`` とは
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``RestTemplate``\ は、REST API(Web API)を呼び出すためのメソッドを提供するクラスであり、
Spring Frameworkが提供するHTTPクライアントである。

具体的な実装方法の説明を行う前に、\ ``RestTemplate``\ がどのようにREST API(Web API)にアクセスしているかを説明する。

.. figure:: ./images_RestClient/RestClientOverview.png
    :alt: Constitution of RestTemplate
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - 項番
      - コンポーネント
      - 説明
    * - | (1)
      - | アプリケーション
      - | \ ``RestTemplate``\ のメソッドを呼び出して、REST API(Web API)の呼び出し依頼を行う。
    * - | (2)
      - | \ ``RestTemplate``\
      - | \ ``HttpMessageConverter``\ を使用して、Javaオブジェクトをリクエストボディに設定する電文(JSON等)に変換する。
    * - | (3)
      - |
      - | \ ``ClientHttpRequestFactory``\ から\ ``ClientHttpRequest``\ を取得して、電文(JSON等)の送信依頼を行う。
    * - | (4)
      - | \ ``ClientHttpRequest``\
      - | 電文(JSON等)をリクエストボディに設定して、REST API(Web API)にHTTP経由でリクエストを行う。
    * - | (5)
      - | \ ``RestTemplate``\
      - | \ ``ResponseErrorHandler``\ を使用して、HTTP通信のエラー判定及びエラー処理を行う。
    * - | (6)
      - | \ ``ResponseErrorHandler``\
      - | \ ``ClientHttpResponse``\ からレスポンスデータを取得して、エラー判定及びエラー処理を行う。
    * - | (7)
      - | \ ``RestTemplate``\
      - | \ ``HttpMessageConverter``\ を使用して、レスポンスボディに設定されている電文(JSON等)をJavaオブジェクトに変換する。
    * - | (8)
      - |
      - | REST API(Web API)の呼び出し結果(Javaオブジェクト)をアプリケーションへ返却する。

.. note:: **非同期処理への対応**

    REST APIからの応答を別スレッドで処理したい場合(非同期で処理したい場合)は、
    \ ``RestTemplate``\ の代わりに\ ``org.springframework.web.client.AsyncRestTemplate``\ を使用すればよい。
    非同期処理の実装例については、:ref:`RestClientAsync` を参照されたい。


.. _RestClientOverviewHttpMessageConverter:

``HttpMessageConverter``
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``org.springframework.http.converter.HttpMessageConverter``\は、アプリケーションで扱うJavaオブジェクトとサーバと通信するための電文(JSON等)を相互に変換するためのインタフェースである。

\ ``RestTemplate``\ を使用した場合、デフォルトで以下の\ ``HttpMessageConverter``\ の実装クラスが登録される。

.. raw:: latex

   \newpage

.. tabularcolumns:: |p{0.05\linewidth}|p{0.25\linewidth}|p{0.55\linewidth}|p{0.15\linewidth}|
.. list-table:: **デフォルトで登録されるHttpMessageConverter**
    :header-rows: 1
    :widths: 5 25 55 15
    :class: longtable

    * - 項番
      - クラス名
      - 説明
      - サポート型
    * - | (1)
      - | ``org.springframework.http.converter.``
        | ``ByteArrayHttpMessageConverter``
      - | 「HTTPボディ(テキスト or バイナリデータ)⇔バイト配列」変換用のクラス。
        | デフォルトですべてのメディアタイプ(\ ``*/*``\)をサポートする。
      - | ``byte[]``
    * - | (2)
      - | ``org.springframework.http.converter.``
        | ``StringHttpMessageConverter``
      - | 「HTTPボディ(テキスト)⇔文字列」変換用のクラス。
        | デフォルトですべてのテキストメディアタイプ(\ ``text/*``\ )をサポートする。
      - | ``String``
    * - | (3)
      - | ``org.springframework.http.converter.``
        | ``ResourceHttpMessageConverter``
      - | 「HTTPボディ(バイナリデータ)⇔Springのリソースオブジェクト」変換用のクラス。
        | デフォルトですべてのメディアタイプ(\ ``*/*``\ )をサポートする。
      - | ``Resource`` [#p1]_
    * - | (4)
      - | ``org.springframework.http.converter.xml.``
        | ``SourceHttpMessageConverter``
      - | 「HTTPボディ(XML)⇔XMLソースオブジェクト」変換用のクラス。
        | デフォルトでXML用のメディアタイプ(\ ``text/xml``\ ,\ ``application/xml``\ ,\ ``application/*-xml``\ )をサポートする。
      - | ``Source`` [#p2]_
    * - | (5)
      - | ``org.springframework.http.converter.support.``
        | ``AllEncompassingFormHttpMessageConverter``
      - | 「HTTPボディ⇔\ ``MultiValueMap``\ オブジェクト」変換用のクラス。
        | \ ``FormHttpMessageConverter``\ の拡張クラスで、multipartのパートデータとしてXMLとJSONへの変換がサポートされている。
        | デフォルトでフォームデータ用のメディアタイプ(\ ``application/x-www-form-urlencoded``\ ,\ ``multipart/form-data``\ )をサポートする。

        * メディアタイプが\ ``application/x-www-form-urlencoded``\ の場合、\ ``MultiValueMap<String, String>``\ として読込/書込される。
        * メディアタイプが\ ``multipart/form-data``\ の場合、\ ``MultiValueMap<String, Object>``\ として書込され、\ ``Object``\ は\ ``AllEncompassingFormHttpMessageConverter``\ 内に別途設定される\ ``HttpMessageConveter``\ で変換される。
          （注意： Note 参照）

        | デフォルトで登録されるパートデータ変換用の\ ``HttpMessageConveter``\ は、`AllEncompassingFormHttpMessageConverter <https://github.com/spring-projects/spring-framework/blob/v4.3.5.RELEASE/spring-web/src/main/java/org/springframework/http/converter/support/AllEncompassingFormHttpMessageConverter.java>`_\
          と `FormHttpMessageConverter <https://github.com/spring-projects/spring-framework/blob/v4.3.5.RELEASE/spring-web/src/main/java/org/springframework/http/converter/FormHttpMessageConverter.java>`_\ のソースを参照されたい。なお、任意の\ ``HttpMessageConverter``\ を登録することもできる。
      - | ``MultiValueMap`` [#p3]_

.. raw:: latex

   \newpage

.. note:: **AllEncompassingFormHttpMessageConverterのメディアタイプがmultipart/form-dataの場合について**

    メディアタイプが\ ``multipart/form-data``\ の場合、「\ ``MultiValueMap``\ オブジェクト から HTTPボディ」への変換は可能だが、
    「HTTPボディ から \ ``MultiValueMap``\ オブジェクト」への変換は現状サポートされていない。
    よって、「HTTPボディ から \ ``MultiValueMap``\ オブジェクト」への変換を行いたい場合は、独自に実装する必要がある。

\

.. raw:: latex

   \newpage

.. tabularcolumns:: |p{0.05\linewidth}|p{0.25\linewidth}|p{0.55\linewidth}|p{0.15\linewidth}|
.. list-table:: **依存ライブラリがクラスパス上に存在する場合に登録されるHttpMessageConverter**
    :header-rows: 1
    :widths: 5 25 55 15
    :class: longtable

    * - 項番
      - クラス名
      - 説明
      - サポート型
    * - | (6)
      - | ``org.springframework.http.converter.feed.``
        | ``AtomFeedHttpMessageConverter``
      - | 「HTTPボディ(Atom)⇔Atomフィードオブジェクト」変換用のクラス。
        | デフォルトでATOM用のメディアタイプ(\ ``application/atom+xml``\ )をサポートする。
        | (ROMEがクラスパスに存在する場合に登録される)
      - | ``Feed`` [#p4]_
    * - | (7)
      - | ``org.springframework.http.converter.feed.``
        | ``RssChannelHttpMessageConverter``
      - | 「HTTPボディ(RSS)⇔Rssチャネルオブジェクト」変換用のクラス。
        | デフォルトでRSS用のメディアタイプ(\ ``application/rss+xml``\ )をサポートする。
        | (ROMEがクラスパスに存在する場合に登録される)
      - | ``Channel`` [#p5]_
    * - | (8)
      - | ``org.springframework.http.converter.json.``
        | ``MappingJackson2HttpMessageConverter``
      - | 「HTTPボディ(JSON)⇔JavaBean」変換用のクラス。
        | デフォルトでJSON用のメディアタイプ(\ ``application/json``\ ,\ ``application/*+json``\ )をサポートする。
        | (Jackson2がクラスパスに存在する場合に登録される)
      - | ``Object`` (JavaBean)
        | ``Map``
    * - | (9)
      - | ``org.springframework.http.converter.xml.``
        | ``MappingJackson2XmlHttpMessageConverter``
      - | 「HTTPボディ(XML)⇔JavaBean」変換用のクラス。
        | デフォルトでXML用のメディアタイプ(\ ``text/xml``\ ,\ ``application/xml``\ ,\ ``application/*-xml``\ )をサポートする。
        | (Jackson-dataformat-xmlがクラスパスに存在する場合に登録される)
      - | ``Object`` (JavaBean)
        | ``Map``
    * - | (10)
      - | ``org.springframework.http.converter.xml.``
        | ``Jaxb2RootElementHttpMessageConverter``
      - | 「HTTPボディ(XML)⇔JavaBean」変換用のクラス。
        | デフォルトでXML用のメディアタイプ(\ ``text/xml``\ ,\ ``application/xml``\ ,\ ``application/*-xml``\ )をサポートする。
        | (JAXBがクラスパスに存在する場合に登録される)
      - | ``Object`` (JavaBean)
    * - | (11)
      - | ``org.springframework.http.converter.json.``
        | ``GsonHttpMessageConverter``
      - | 「HTTPボディ(JSON)⇔JavaBean」変換用のクラス。
        | デフォルトでJSON用のメディアタイプ(\ ``application/json``\ ,\ ``application/*+json``\ )をサポートする。
        | (Gsonがクラスパスに存在する場合に登録される)
      - | ``Object`` (JavaBean)
        | ``Map``

.. raw:: latex

   \newpage

\

.. [#p1] \ ``org.springframework.core.io``\ パッケージのクラス
.. [#p2] \ ``javax.xml.transform``\ パッケージのクラス
.. [#p3] \ ``org.springframework.util``\ パッケージのクラス
.. [#p4] \ ``com.rometools.rome.feed.atom``\ パッケージのクラス
.. [#p5] \ ``com.rometools.rome.feed.rss``\ パッケージのクラス


.. _RestClientOverviewClientHttpRequestFactory:

``ClientHttpRequestFactory``
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``RestTemplate``\ は、サーバとの通信処理を以下の３つのインタフェースの実装クラスに委譲することで実現している。

* ``org.springframework.http.client.ClientHttpRequestFactory``
* ``org.springframework.http.client.ClientHttpRequest``
* ``org.springframework.http.client.ClientHttpResponse``

この３つのインタフェースのうち、開発者が意識するのは\ ``ClientHttpRequestFactory``\ である。
\ ``ClientHttpRequestFactory``\ は、サーバとの通信処理を行うクラス(\ ``ClientHttpRequest``\ と \ ``ClientHttpResponse``\ インタフェースの実装クラス)を解決する役割を担っている。

なお、Spring Frameworkが提供している主な\ ``ClientHttpRequestFactory``\ の実装クラスは以下の通りである。

.. tabularcolumns:: |p{0.05\linewidth}|p{0.25\linewidth}|p{0.70\linewidth}|
.. list-table:: **Spring Frameworkが提供している主なClientHttpRequestFactoryの実装クラス**
   :header-rows: 1
   :widths: 5 25 70

   * - 項番
     - クラス名
     - 説明
   * - | (1)
     - | ``org.springframework.http.client.``
       | ``SimpleClientHttpRequestFactory``
     - | Java SE標準の `HttpURLConnection <https://docs.oracle.com/javase/8/docs/api/java/net/HttpURLConnection.html>`_\ のAPIを使用して通信処理(同期、非同期)を行うための実装クラス。(デフォルトで使用される実装クラス)
   * - | (2)
     - | ``org.springframework.http.client.``
       | ``Netty4ClientHttpRequestFactory``
     - | `Netty 4 <http://netty.io/>`_\ のAPIを使用して通信処理(同期、非同期)を行うための実装クラス。
   * - | (3)
     - | ``org.springframework.http.client.``
       | ``HttpComponentsClientHttpRequestFactory``
     - | `Apache HttpComponents HttpClient <http://hc.apache.org/httpcomponents-client-ga/>`_\ のAPIを使用して同期型の通信処理を行うための実装クラス。(HttpClient 4.3以上が必要)
   * - | (4)
     - | ``org.springframework.http.client.``
       | ``HttpComponentsAsyncClientHttpRequestFactory``
     - | `Apache HttpComponents HttpAsyncClient <http://hc.apache.org/httpcomponents-asyncclient-dev/>`_\ のAPIを使用して非同期型の通信処理を行うための実装クラス。(HttpAsyncClient 4.0以上が必要)
   * - | (5)
     - | ``org.springframework.http.client.``
       | ``OkHttpClientHttpRequestFactory``
     - | `Square OkHttp <http://square.github.io/okhttp/>`_\ のAPIを使用して通信処理（同期、非同期）を行うための実装クラス。

.. note:: **使用するClientHttpRequestFactoryの実装クラスについて**

    \ ``RestTemplate``\ が使用するデフォルト実装は\ ``SimpleClientHttpRequestFactory``\ であり、本ガイドラインでも\ ``SimpleClientHttpRequestFactory``\ を使用した際の実装例となっている。
    Java SEの\ ``HttpURLConnection``\ で要件が満たせない場合は、Netty、Apache Http Componentsなどのライブラリの利用を検討されたい。


.. _RestClientOverviewResponseErrorHandler:

``ResponseErrorHandler``
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``RestTemplate``\ は、サーバとの通信エラーのハンドリングを\ ``org.springframework.web.client.ResponseErrorHandler``\ インタフェースに委譲することで実現している。

\ ``ResponseErrorHandler``\ には、

* エラー判定を行うメソッド(\ ``hasError``\ )
* エラー処理を行うメソッド(\ ``handleError``\)

が定義されており、Spring Frameworkはデフォルト実装として\ ``org.springframework.web.client.DefaultResponseErrorHandler``\ を提供している。

\ ``DefaultResponseErrorHandler``\ は、サーバから応答されたHTTPのステータスコードの値によって以下のようなエラー処理を行う。

* レスポンスコードが正常系(2xx)の場合は、エラー処理は行わない。
* レスポンスコードがクライアントエラー系(4xx)の場合は、\ ``org.springframework.web.client.HttpClientErrorException``\ を発生させる。
* レスポンスコードがサーバエラー系(5xx)の場合は、\ ``org.springframework.web.client.HttpServerErrorException``\ を発生させる。
* レスポンスコードが未定義のコード(ユーザ定義のカスタムコード)の場合は、\ ``org.springframework.web.client.UnknownHttpStatusCodeException``\ を発生させる。

.. note:: **エラー時のレスポンスデータの取得方法**

    エラー時のレスポンスデータ(HTTPステータスコード、レスポンスヘッダ、レスポンスボディなど)は、例外クラスのgetterメソッドを呼び出すことで取得することができる。


.. _RestClientOverviewClientHttpRequestInterceptor:

``ClientHttpRequestInterceptor``
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``org.springframework.http.client.ClientHttpRequestInterceptor``\ は、サーバとの通信の前後に共通的な処理を実装するためのインタフェースである。

\ ``ClientHttpRequestInterceptor``\ を使用すると、

* サーバとの通信ログ
* 認証ヘッダの設定

といった共通的な処理を\ ``RestTemplate``\ に適用することができる。

.. note:: **ClientHttpRequestInterceptorの動作仕様**

    \ ``ClientHttpRequestInterceptor``\ は複数適用することができ、指定した順番でチェーン実行される。
    これはサーブレットフィルタの動作によく似ており、最後に実行されるチェーン先として\ ``ClientHttpRequest``\ によるHTTP通信処理が登録されている。
    例えば、ある条件に一致した際にサーバとの通信処理をキャンセルしたいという要件があった場合は、チェーン先を呼びださなければよい。

    この仕組みを活用すると、

    * サーバとの通信の閉塞
    * 通信処理のリトライ

    といった処理を適用することもできる。


.. _RestClientHowToUse:

How to use
--------------------------------------------------------------------------------

本節では、\ ``RestTemplate``\ を使用したクライアント処理の実装方法について説明する。

.. note:: **RestTemplateがサポートするHTTPメソッドについて**

    本ガイドラインでは、GETメソッドとPOSTメソッドを使用したクライアント処理の実装例のみを紹介するが、
    \ ``RestTemplate``\ は他のHTTPメソッド(PUT, PATCH, DELETE, HEAD, OPTIONSなど)もサポートしており、同じような要領で使用することができる。
    詳細は\ `RestTemplate <http://docs.spring.io/spring/docs/4.3.5.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html>`_\ のJavadocを参照されたい。

.. _RestClientHowToUseSetup:

\ ``RestTemplate``\ のセットアップ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ ``RestTemplate``\ を使用する場合は、\ ``RestTemplate``\ をDIコンテナに登録し、\ ``RestTemplate``\ を利用するコンポーネントにインジェクションする。


依存ライブラリ設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| \ ``RestTemplate``\ を使用するために\ ``pom.xml``\ に、Spring Frameworkのspring-webライブラリを追加する。
| マルチプロジェクト構成の場合は、domainプロジェクトの\ ``pom.xml``\ に追加する。

.. code-block:: xml

    <dependencies>

        <!-- (1) -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
        </dependency>

    </dependencies>

.. note::
    上記設定例は、依存ライブラリのバージョンを親プロジェクトである terasoluna-gfw-parent で管理する前提であるため、pom.xmlでのバージョンの指定は不要である。
    上記の依存ライブラリはterasoluna-gfw-parentが利用している\ `Spring IO Platform <http://platform.spring.io/platform/>`_\ で定義済みである。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | Spring Frameworkの\ ``spring-web``\ ライブラリをdependenciesに追加する。


\ ``RestTemplate``\ のbean定義
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``RestTemplate``\ のbean定義を行い、DIコンテナに登録する。

**bean定義ファイル(applicationContext.xml)の定義例**

.. code-block:: xml

    <bean id="restTemplate" class="org.springframework.web.client.RestTemplate" /> <!-- (1) -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``RestTemplate``\ をデフォルト設定のまま利用する場合は、デフォルトコンストラクタを使用してbeanを登録する。


.. note:: **RestTemplateのカスタマイズ方法**

    HTTP通信処理をカスタマイズする場合は、以下のようなbean定義となる。

     .. code-block:: xml

        <bean id="clientHttpRequestFactory"
              class="org.springframework.http.client.SimpleClientHttpRequestFactory"> <!-- (1) -->
            <!-- Set properties for customize a http communication (omit on this sample) -->
        </bean>

        <bean id="restTemplate" class="org.springframework.web.client.RestTemplate">
            <constructor-arg ref="clientHttpRequestFactory" /> <!-- (2) -->
        </bean>

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90

        * - 項番
          - 説明
        * - | (1)
          - | \ ``ClientHttpRequestFactory``\ のbean定義を行う。
            | 本ガイドラインではタイムアウトの設定をカスタマイズする方法を紹介している。詳細は :ref:`RestClientHowToUseTimeoutSettings` を参照されたい。
        * - | (2)
          - | \ ``ClientHttpRequestFactory``\ を引数に指定するコンストラクタを使用してbeanを登録する。

    なお、\ ``HttpMessageConverter``\ 、\ ``ResponseErrorHandler``\ 、\ ``ClientHttpRequestInterceptor``\ のカスタマイズ方法については、

    * :ref:`RestClientHowToExtendHttpMessageConverter`
    * :ref:`RestClientHowToUseErrorHandlingResponseEntity`
    * :ref:`RestClientHowToExtendClientHttpRequestInterceptor`

    を参照されたい。


\ ``RestTemplate``\ の利用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``RestTemplate``\ を利用する場合は、DIコンテナに登録されている\ ``RestTemplate``\ をインジェクションする。

**RestTemplateのインジェクション例**

.. code-block:: java

    @Service
    public class AccountServiceImpl implements AccountService {

        @Inject
        RestTemplate restTemplate;

        // ...

    }


.. _RestClientHowToUseGet:

GETリクエストの送信
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``RestTemplate``\ は、GETリクエストを行うためのメソッドを複数提供している。

* 通常は\ ``getForObject``\ メソッド又は\ ``getForEntity``\ メソッドを使用する。
* 任意のヘッダを設定するなどリクエストに細かい設定を行いたい場合は、\ ``org.springframework.http.RequestEntity``\ と\ ``exchange``\ メソッドを使用する。

\ ``getForObject``\ メソッドを使用した実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

レスポンスボディのみ取得できればよい場合は、\ ``getForObject``\ メソッドを使用する。

**getForObjectメソッドの使用例**

フィールド宣言部

.. code-block:: java

    @Value("${api.url:http://localhost:8080/api}")
    URI uri;


メソッド内部

.. code-block:: java

    User user = restTemplate.getForObject(uri, User.class); // (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ``getForObject``\ メソッドを使用した場合は、戻り値はレスポンスボディの値になる。
        | レスポンスボディのデータは\ ``HttpMessageConverter``\ によって第2引数に指定したJavaクラスへ変換された後、返却される。


\ ``getForEntity``\ メソッドを使用した実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

HTTPステータスコード、レスポンスヘッダ、レスポンスボディを取得する必要がある場合は、\ ``getForEntity``\ メソッドを使用する。

**getForEntityメソッドの使用例**

.. code-block:: java

    ResponseEntity<User> responseEntity =
            restTemplate.getForEntity(uri, User.class); // (1)
    HttpStatus statusCode = responseEntity.getStatusCode(); // (2)
    HttpHeaders header = responseEntity.getHeaders(); // (3)
    User user = responseEntity.getBody(); // (4)


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ``getForEntity``\ メソッドを使用した場合は、戻り値は\ ``org.springframework.http.ResponseEntity``\ となる。
    * - | (2)
      - | HTTPステータスコードは\ ``getStatusCode``\ メソッドを用いて取得する。
    * - | (3)
      - | レスポンスヘッダは\ ``getHeaders``\ メソッドを用いて取得する。
    * - | (4)
      - | レスポンスボディは\ ``getBody``\ メソッドを用いて取得する。

.. note:: **ResponseEntityとは**

    ``ResponseEntity``\ はHTTPレスポンスを表すクラスで、HTTPステータスコード、レスポンスヘッダ、レスポンスボティの情報を取得することができる。
    詳細は\ `ResponseEntity <http://docs.spring.io/spring/docs/4.3.5.RELEASE/javadoc-api/org/springframework/http/ResponseEntity.html>`_\ のJavadocを参照されたい。



\ ``exchange``\ メソッドを使用した実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

リクエストヘッダを指定する必要がある場合は、\ ``org.springframework.http.RequestEntity``\ を生成し\ ``exchange``\ メソッドを使用する。

**exchangeメソッドの使用例**

import部

.. code-block:: java

    import org.springframework.http.RequestEntity;
    import org.springframework.http.ResponseEntity;


フィールド宣言部

.. code-block:: java

    @Value("${api.url:http://localhost:8080/api}")
    URI uri;


メソッド内部

.. code-block:: java

    RequestEntity requestEntity = RequestEntity
            .get(uri)//(1)
            .build();//(2)

    ResponseEntity<User> responseEntity =
            restTemplate.exchange(requestEntity, User.class);//(3)

    User user = responseEntity.getBody();//(4)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ``RequestEntity``\ の\ ``get``\メソッドを使用し、GETリクエスト用のリクエストビルダを生成する。
        | パラメータにURIを設定する。
    * - | (2)
      - | ``RequestEntity.HeadersBuilder``\ の\ ``build``\メソッドを使用し、\ ``RequestEntity``\ オブジェクトを作成する。
    * - | (3)
      - | ``exchange``\ メソッドを使用し、リクエストを送信する。第二引数に、レスポンスデータの型を指定する。
        | レスポンスは、\ ``ResponseEntity<T>``\ になる。型パラメータに、レスポンスデータの型を設定する。
    * - | (4)
      - | ``getBody``\ メソッドを使用し、レスポンスボディのデータを取得する。

.. note:: **RequestEntityとは**

    ``RequestEntity``\ はHTTPリクエストを表すクラスで、接続URI、HTTPメソッド、リクエストヘッダ、リクエストボディを設定することができる。
    詳細は\ `RequestEntity <http://docs.spring.io/spring/docs/4.3.5.RELEASE/javadoc-api/org/springframework/http/RequestEntity.html>`_\ のJavadocを参照されたい。

    なお、リクエストヘッダの設定方法については、:ref:`RestClientHowToUseRequestHeader` を参照されたい。


.. _RestClientHowToUsePost:

POSTリクエストの送信
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``RestTemplate``\ は、POSTリクエストを行うためのメソッドを複数提供している。

* 通常は、\ ``postForObject``\ 、\ ``postForEntity``\ を使用する。
* 任意のヘッダを設定するなどリクエストに細かい設定を行いたい場合は、\ ``RequestEntity``\ と \ ``exchange``\ メソッドを使用する。

\ ``postForObject``\ メソッドを使用した実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

POSTした結果としてレスポンスボディのみ取得できればよい場合は、\ ``postForObject``\ メソッドを使用する。

**postForObjectメソッドの使用例**

.. code-block:: java


    User user = new User();

    //...

    User user = restTemplate.postForObject(uri, user, User.class); // (1)


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ``postForObject``\ メソッドは、簡易にPOSTリクエストを実装できる。
        | 第二引数には、``HttpMessageConverter``\ によってリクエストボディに変換されるJavaオブジェクトを設定する。
        | ``postForObject``\ メソッドを使用した場合は、戻り値はレスポンスボディの値になる。

\ ``postForEntity``\ メソッドを使用した実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

POSTした結果としてHTTPステータスコード、レスポンスヘッダ、レスポンスボディを取得する必要がある場合は、\ ``postForEntity``\ メソッドを使用する。

**postForEntityメソッドの使用例**

.. code-block:: java

    User user = new User();

    //...

    ResponseEntity<User> responseEntity =
            restTemplate.postForEntity(uri, user, User.class); // (1)


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ``postForEntity``\ メソッドも\ ``getForObject``\ メソッドと同様に簡易にPOSTリクエストを実装できる。
        | ``postForEntity``\ メソッドを使用した場合は、戻り値は\ ``ResponseEntity``\ となる。
        | レスポンスボディの値は、\ ``ResponseEntity``\ から取得する。



\ ``exchange``\ メソッドを使用した実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

リクエストヘッダを指定する必要がある場合は、\ ``RequestEntity``\ を生成し\ ``exchange``\ メソッドを使用する。

**exchangeメソッドの使用例**

import部

.. code-block:: java

    import org.springframework.http.RequestEntity;
    import org.springframework.http.ResponseEntity;


フィールド宣言部

.. code-block:: java

    @Value("${api.url:http://localhost:8080/api}")
    URI uri;


メソッド内部

.. code-block:: java

    User user = new User();

    //...

    RequestEntity<User> requestEntity = RequestEntity//(1)
            .post(uri)//(2)
            .body(user);//(3)

    ResponseEntity<User> responseEntity =
            restTemplate.exchange(requestEntity, User.class);//(4)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ``RequestEntity``\ を使用して、リクエストを生成する。型パラメータに、リクエストボディに設定するデータの型を指定する。
    * - | (2)
      - | ``post``\ メソッドを使用し、POSTリクエスト用のリクエストビルダを生成する。パラメータにURIを設定する。
    * - | (3)
      - | ``RequestEntity.BodyBuilder``\ の\ ``body``\ メソッドを使用し、\ ``RequestEntity``\ オブジェクトを作成する。
        | パラメータにリクエストボディに変換するJavaオブジェクトを設定する。
    * - | (4)
      - | ``exchange``\ メソッドを使用し、リクエストを送信する。

.. note:: **リクエストヘッダの設定方法**

    リクエストヘッダの設定方法については、:ref:`RestClientHowToUseRequestHeader` を参照されたい。


.. _RestClientHowToUseGetCollection:

コレクション形式のデータ取得
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

サーバから応答されるレスポンスボディの電文(JSON等)がコレクション形式の場合は、以下のような実装となる。

**コレクション形式のデータの取得例**

.. code-block:: java

    ResponseEntity<List<User>> responseEntity = //(1)
        restTemplate.exchange(requestEntity, new ParameterizedTypeReference<List<User>>(){}); //(2)

    List<User> userList = responseEntity.getBody();//(3)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ``ResponseEntity``\ の型パラメータに\ ``List``\<レスポンスデータの型>を指定する。
    * - | (2)
      - | ``exchange``\ メソッドの第二引数に\ ``org.springframework.core.ParameterizedTypeReference``\ のインスタンスを指定し、型パラメータに\ ``List``\<レスポンスデータの型>を指定する。
    * - | (2)
      - | ``getBody``\ メソッドで、レスポンスボディのデータを取得する。

.. _RestClientHowToUseRequestHeader:

リクエストヘッダの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ ``RequestEntity``\ と\ ``exchange``\ メソッドを使用すると、\ ``RequestEntity``\ のメソッドを使用して特定のヘッダ及び任意のヘッダを設定することができる。
詳細は\ `RequestEntity <http://docs.spring.io/spring/docs/4.3.5.RELEASE/javadoc-api/org/springframework/http/RequestEntity.html>`_\ のJavadocを参照されたい。

本ガイドラインでは、

* :ref:`RestClientHowToUseRequestHeaderContentType`
* :ref:`RestClientHowToUseRequestHeaderAccept`
* :ref:`RestClientHowToUseRequestHeaderAnyHeader`

について説明する。

.. _RestClientHowToUseRequestHeaderContentType:

Content-Typeヘッダの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

サーバへデータを送信する場合は、通常Content-Typeヘッダの指定が必要となる。

**Content-Typeヘッダの設定例**

.. code-block:: java

    User user = new User();

    //...

    RequestEntity<User> requestEntity = RequestEntity
            .post(uri)
            .contentType(MediaType.APPLICATION_JSON) // (1)
            .body(user);



.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ``RequestEntity.BodyBuilder``\ の\ ``contentType``\ メソッドを使用し、Context-Typeヘッダの値を指定する。
        | 上記の実装例では、データ形式がJSONであることを示す「\ ``application/json``\」を設定している。


.. _RestClientHowToUseRequestHeaderAccept:

Acceptヘッダの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

サーバから取得するデータの形式を指定する場合は、Acceptヘッダの指定が必要となる。
サーバが複数のデータ形式のレスポンスをサポートしていない場合は、Acceptヘッダを明示的に指定しなくてもよいケースもある。

**Acceptヘッダの設定例**

.. code-block:: java

    User user = new User();

    //...

    RequestEntity<User> requestEntity = RequestEntity
            .post(uri)
            .accept(MediaType.APPLICATION_JSON) // (1)
            .body(user);



.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ``RequestEntity.HeadersBuilder``\ の\ ``accept``\ メソッドを使用して、Acceptヘッダの値を設定する。
        | 上記の実装例では、取得可能なデータ形式がJSONであることを示す「\ ``application/json``\」を設定している。


.. _RestClientHowToUseRequestHeaderAnyHeader:

任意のリクエストヘッダの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

サーバへアクセスするために、リクエストヘッダの設定が必要になるケースがある。

**任意ヘッダの設定例**

.. code-block:: java

    User user = new User();

    //...

    RequestEntity<User> requestEntity = RequestEntity
            .post(uri)
            .header("Authorization", "Basic " + base64Credentials) // (1)
            .body(user);



.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ``RequestEntity.HeadersBuilder``\ の\ ``header``\ メソッドを使用してリクエストヘッダの名前と値を設定する。
        | 上記の実装例では、AuthorizationヘッダにBasic認証に必要な資格情報を設定している。


.. _RestClientHowToUseErrorHandling:

エラーハンドリング
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _RestClientHowToUseErrorHandlingHandleException:

例外ハンドリング(デフォルトの動作)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``RestTemplate``\ のデフォルト実装(\ ``DefaultResponseErrorHandler``\ )では、

* レスポンスコードがクライアントエラー系(4xx)の場合は、\ ``HttpClientErrorException``\
* レスポンスコードがサーバエラー系(5xx)の場合は、\ ``HttpServerErrorException``\
* レスポンスコードが未定義のコード(ユーザ定義のカスタムコード)の場合は、\ ``UnknownHttpStatusCodeException``\

が発生するため、必要に応じてこれらの例外をハンドリングする必要がある。

**例外ハンドリングの実装例**

.. note::

    以下の実装例は、サーバエラーが発生した際の例外ハンドリングの一例である。

    アプリケーションの要件に応じて\ **適切な例外ハンドリングを行うこと。**\

フィールド宣言部

.. code-block:: java

    @Value("${api.retry.maxCount}")
    int retryMaxCount;

    @Value("${api.retry.retryWaitTimeCoefficient}")
    int retryWaitTimeCoefficient;


メソッド内部

.. code-block:: java

    int retryCount = 0;
    while (true) {
        try {

            responseEntity = restTemplate.exchange(requestEntity, String.class);

            if (log.isInfoEnabled()) {
                log.info("Success({}) ", responseEntity.getStatusCode());
            }

            break;

        } catch (HttpServerErrorException e) { // (1)

            if (retryCount == retryMaxCount) {
                throw e;
            }

            retryCount++;

            if (log.isWarnEnabled()) {
                log.warn("An error ({}) occurred on the server. (The number of retries：{} Times)", e.getStatusCode(),
                    retryCount);
            }

            try {
                Thread.sleep(retryWaitTimeCoefficient * retryCount);
            } catch (InterruptedException ie) {
                Thread.currentThread().interrupt();
            }

            //...
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 例外をキャッチしてエラー処理を行う。サーバエラー（500系）の場合、\ ``HttpServerErrorException``\ をキャッチする。


.. _RestClientHowToUseErrorHandlingResponseEntity:

\ ``ResponseEntity``\ の返却（エラーハンドラの拡張）
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``org.springframework.web.client.ResponseErrorHandler``\ インタフェースの実装クラスを\ ``RestTemplate``\ に設定することで、独自のエラー処理を行うことができる。

以下の例では、サーバエラー及びクライアントエラーが発生した場合でも\ ``ResponseEntity``\ を返すようにエラーハンドラを拡張している。

**エラーハンドラの実装クラスの作成例**

.. code-block:: java

    import org.springframework.http.client.ClientHttpResponse;
    import org.springframework.web.client.DefaultResponseErrorHandler;

    public class CustomErrorHandler extends DefaultResponseErrorHandler { // (1)

        @Override
        public void handleError(ClientHttpResponse response) throws IOException {
            //Don't throw Exception.
        }

    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ``ResponseErrorHandler``\ インタフェースの実装クラスを作成する。
        | 上記の作成例では、デフォルトのエラーハンドラの実装クラスである\ ``DefaultResponseErrorHandler``\ を拡張し、
        | サーバエラー及びクライアントエラーが発生した際に例外を発生させずに\ ``ResponseEntity``\ が返るようにしている。

**bean定義ファイル(applicationContext.xml)の定義例**

.. code-block:: xml

    <bean id="customErrorHandler" class="com.example.restclient.CustomErrorHandler" /> <!-- (1) -->

    <bean id="restTemplate" class="org.springframework.web.client.RestTemplate">
        <property name="errorHandler" ref="customErrorHandler" /><!-- (2) -->
    </bean>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``ResponseErrorHandler``\ の実装クラスのbean定義を行う。
    * - | (2)
      - | \ ``errorHandler``\ プロパティに\ ``ResponseErrorHandler``\ のbeanをインジェクションする。

**クライアント処理の実装例**

.. code-block:: java

    int retryCount = 0;
    while (true) {

        responseEntity = restTemplate.exchange(requestEntity, User.class);

        if (responseEntity.getStatusCode() == HttpStatus.OK) { // (1)

            break;

        } else if (responseEntity.getStatusCode() == HttpStatus.SERVICE_UNAVAILABLE) { // (2)

            if (retryCount == retryMaxCount) {
                break;
            }

            retryCount++;

            if (log.isWarnEnabled()) {
                log.warn("An error ({}) occurred on the server. (The number of retries：{} Times)",
                    responseEntity.getStatusCode(), retryCount);
            }

            try {
                Thread.sleep(retryWaitTimeCoefficient * retryCount);
            } catch (InterruptedException ie) {
                Thread.currentThread().interrupt();
            }

            //...
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 上記の実装例では、エラー時にも\ ``ResponseEntity``\ を返すようにエラーハンドラを拡張しているので、返却された\ ``ResponseEntity``\ からHTTPステータスコードを取得して、処理結果が正常であったか確認する必要がある。
    * - | (2)
      - | エラー発生時も返却された\ ``ResponseEntity``\ からHTTPステータスコードを取得して、その値に応じて処理を制御することができる。

.. _RestClientHowToUseTimeoutSettings:

通信タイムアウトの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

サーバとの通信に対してタイムアウト時間を指定したい場合は、以下のようなbean定義を行う。

**bean定義ファイル(applicationContext.xml)の定義例**

.. code-block:: xml

    <bean id="clientHttpRequestFactory"
          class="org.springframework.http.client.SimpleClientHttpRequestFactory">
        <property name="connectTimeout" value="${api.connectTimeout: 2000}" /><!-- (1) -->
        <property name="readTimeout" value="${api.readTimeout: 2000}" /><!-- (2) -->
    </bean>

    <bean id="restTemplate" class="org.springframework.web.client.RestTemplate">
        <constructor-arg ref="clientHttpRequestFactory" />
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``connectTimeout``\ プロパティにサーバとの接続タイムアウト時間(ミリ秒)を設定する。
        | タイムアウト発生時は\ ``org.springframework.web.client.ResourceAccessException``\ が発生する。
    * - | (2)
      - | \ ``readTimeout``\ プロパティにレスポンスデータの読み込みタイムアウト時間(ミリ秒)を設定する。
        | タイムアウト発生時は\ ``ResourceAccessException``\ が発生する。

.. note:: **タイムアウト発生時の起因例外**

    \ ``ResourceAccessException``\ は起因例外をラップしており、接続タイムアウト及び読み込みタイムアウト発生時の起因例外は共に\ ``java.net.SocketTimeoutException``\ である。
    デフォルト実装(\ ``SimpleClientHttpRequestFactory``\)を使用した場合は、どちらのタイムアウトが発生したかを例外クラスの種類で区別できないという点を補足しておく。

    なお、他の\ ``HttpRequestFactory``\ を使用した場合の動作は未検証であるため、起因例外が上記と異なる可能性がある。
    他の\ ``HttpRequestFactory``\ を使用する場合は、タイムアウト時に発生する例外を把握した上で適切な例外ハンドリングを行うこと。


.. _RestClientHowToUseHttps:

SSL自己署名証明書の使用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

テスト環境などでSSL自己署名証明書を使用する場合は、以下のように実装する。

**FactoryBeanの実装例**

\ ``RestTemplate``\ のBean定義で、コンストラクタの引数に渡す \ ``org.springframework.http.client.ClientHttpRequestFactory``\ を作成するための \ ``org.springframework.beans.factory.FactoryBean``\ を実装する。

.. code-block:: java

    import java.security.KeyStore;

    import javax.net.ssl.KeyManagerFactory;
    import javax.net.ssl.SSLContext;
    import javax.net.ssl.TrustManagerFactory;

    import org.apache.http.client.HttpClient;
    import org.apache.http.impl.client.HttpClientBuilder;
    import org.springframework.beans.factory.FactoryBean;
    import org.springframework.http.client.ClientHttpRequestFactory;
    import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;

    public class RequestFactoryBean implements
            FactoryBean<ClientHttpRequestFactory> {

        private String keyStoreFileName;

        private char[] keyStorePassword;

        @Override
        public ClientHttpRequestFactory getObject() throws Exception {

            // (1)
            SSLContext sslContext = SSLContext.getInstance("TLS");

            KeyStore ks = KeyStore.getInstance(KeyStore.getDefaultType());
            ks.load(this.getClass().getClassLoader()
                    .getResourceAsStream(this.keyStoreFileName),
                    this.keyStorePassword);

            KeyManagerFactory kmf = KeyManagerFactory.getInstance(KeyManagerFactory
                    .getDefaultAlgorithm());
            kmf.init(ks, this.keyStorePassword);

            TrustManagerFactory tmf = TrustManagerFactory
                    .getInstance(TrustManagerFactory.getDefaultAlgorithm());
            tmf.init(ks);

            sslContext.init(kmf.getKeyManagers(), tmf.getTrustManagers(), null);

            // (2)
            HttpClient httpClient = HttpClientBuilder.create()
                    .setSSLContext(sslContext).build();

            // (3)
            ClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory(
                    httpClient);

            return factory;
        }

        @Override
        public Class<?> getObjectType() {
            return ClientHttpRequestFactory.class;
        }

        @Override
        public boolean isSingleton() {
            return true;
        }

        public void setKeyStoreFileName(String keyStoreFileName) {
            this.keyStoreFileName = keyStoreFileName;
        }

        public void setKeyStorePassword(char[] keyStorePassword) {
            this.keyStorePassword = keyStorePassword;
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 後述のbean定義で指定されたキーストアファイルのファイル名とパスワードを元に、SSLコンテキストを作成する。
        | 使用するSSL自己署名証明書のキーストアファイルは、クラスパス上に配置する。
    * - | (2)
      - | 作成したSSLコンテキストを利用する \ ``org.apache.http.client.HttpClient``\ を作成する。
    * - | (3)
      - | 作成した\ ``HttpClient``\ を利用する \ ``ClientHttpRequestFactory``\ を作成する。


\ ``HttpClient``\ および  \ ``HttpClientBuilder``\ を使用するためには、Apache HttpComponents HttpClient のライブラリが必要となる。
以下を \ :file:`pom.xml`\ に追加し、Apache HttpComponents HttpClient を依存ライブラリに追加する。

* :file:`pom.xml`

 .. code-block:: xml

    <dependency>
        <groupId>org.apache.httpcomponents</groupId>
        <artifactId>httpclient</artifactId>
    </dependency>

.. note::
    上記設定例は、依存ライブラリのバージョンを親プロジェクトである terasoluna-gfw-parent で管理する前提であるため、pom.xmlでのバージョンの指定は不要である。
    上記の依存ライブラリはterasoluna-gfw-parentが利用している\ `Spring IO Platform <http://platform.spring.io/platform/>`_\ で定義済みである。

**bean定義ファイル(applicationContext.xml)の定義例**

SSL自己署名証明書を使用したSSL通信を行う \ ``RestTemplate``\ を定義する。

.. code-block:: xml

    <bean id="httpsRestTemplate" class="org.springframework.web.client.RestTemplate">
        <constructor-arg>
            <bean class="com.example.restclient.RequestFactoryBean"><!-- (1) -->
                <property name="keyStoreFileName" value="${rscl.keystore.filename}" />
                <property name="keyStorePassword" value="${rscl.keystore.password}" />
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
      - | 作成した \ ``RequestFactoryBean``\ を \ ``RestTemplate``\ のコンストラクタに指定する。
        | \ ``RequestFactoryBean``\ には、キーストアファイルのファイル名とパスワードを渡す。

**RestTemplateの使用方法**

\ ``RestTemplate``\ の使い方については、SSL自己署名証明書を使用しない場合と同じである。



.. _RestClientHowToUseAuthentication:

Basic認証
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

サーバがBasic認証を要求する場合は、以下のように実装する。

**Basic認証の実装例**

フィールド宣言部

.. code-block:: java


    @Value("${api.auth.userid}")
    String userid;

    @Value("${api.auth.password}")
    String password;


メソッド内部

.. code-block:: java

    String plainCredentials = userid + ":" + password; // (1)
    String base64Credentials = Base64.getEncoder()
            .encodeToString(plainCredentials.getBytes(StandardCharsets.UTF_8)); // (2)

    RequestEntity requestEntity = RequestEntity
          .get(uri)
          .header("Authorization", "Basic " + base64Credentials) // (3)
          .build();

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ユーザIDとパスワードを「\ ``":"``\ 」でつなげる。
    * - | (2)
      - | （1）をバイト配列に変換して、Base64エンコードする。
    * - | (3)
      - | AuthorizationヘッダをBasic認証の資格情報を設定する。

.. note::

  Java SE8以降の場合は、Java標準の\ ``java.util.Base64``\ を使用する。それ以前の場合は、Spring Securityの\ ``org.springframework.security.crypto.codec.Base64``\ を使用する。


.. _RestClientHowToUseFileUpload:

ファイルアップロード(マルチパートリクエスト)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``RestTemplate``\ を使用してファイルアップロード(マルチパートリクエスト)を行う場合は、以下のように実装する。

**ファイルアップロードの実装例**

.. code-block:: java

  MultiValueMap<String, Object> multiPartBody = new LinkedMultiValueMap<>();//(1)
  multiPartBody.add("file", new ClassPathResource("/uploadFiles/User.txt"));//(2)

  RequestEntity<MultiValueMap<String, Object>> requestEntity = RequestEntity
          .post(uri)
          .contentType(MediaType.MULTIPART_FORM_DATA)//(3)
          .body(multiPartBody);//(4)


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | マルチパートリクエストとして送信するデータを格納するために\ ``MultiValueMap``\ を生成する。
    * - | (2)
      - | パラメータ名をキーに指定して、アップロードするファイルを\ ``MultiValueMap``\ に追加する。
        | 上記例では、\ ``file``\ というパラメータ名を指定して、クラスパス上に配置されているファイルをアップロードファイルとして追加している。
    * - | (3)
      - | Content-Typeヘッダのメディアタイプを\ ``multipart/form-data``\ に設定する。
    * - | (4)
      - | アップロードするファイルが格納されている\ ``MultiValueMap``\ をリクエストボディに設定する。

.. note:: **Spring Frameworkが提供するResourceクラスについて**

    Spring Frameworkはリソースを表現するインタフェースとして\ ``org.springframework.core.io.Resource``\ を提供しており、
    ファイルをアップロードする際に使用することができる。

    \ ``Resource``\ インタフェースの主な実装クラスは以下の通りである。

    * ``org.springframework.core.io.PathResource``
    * ``org.springframework.core.io.FileSystemResource``
    * ``org.springframework.core.io.ClassPathResource``
    * ``org.springframework.core.io.UrlResource``
    * ``org.springframework.core.io.InputStreamResource``  (ファイル名をサーバに連携できない)
    * ``org.springframework.core.io.ByteArrayResource``  (ファイル名をサーバに連携できない)


.. _RestClientHowToUseFileDownload:

ファイルダウンロード
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``RestTeamplate``\を使用してファイルダウンロードを行う場合は、以下のように実装する。

**ファイルダウンロードの実装例（ファイルサイズが小さい場合）**

.. code-block:: java

    RequestEntity requestEntity = RequestEntity
            .get(uri)
            .build();

    ResponseEntity<byte[]> responseEntity =
            restTemplate.exchange(requestEntity, byte[].class);//(1)

    byte[] downloadContent = responseEntity.getBody();//(2)


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ダウンロードファイルを指定したデータ型で扱う。ここでは、バイト配列を指定。
    * - | (2)
      - | レスポンスボディからダウンロードしたファイルのデータを取得する。

.. warning:: **サイズの大きいファイルをダウンロードする際の注意点**

    サイズの大きなファイルをデフォルトで登録されている\ ``HttpMessageConverter``\ を使用して \ ``byte``\ 配列で取得すると、 \ ``java.lang.OutOfMemoryError``\ が発生する可能性がある。
    そのため、サイズの大きなファイルをダウンロードしたい場合は、レスポンスから \ ``InputStream``\ を取得して、ダウンロードデータを少しずつファイルに書き出す必要がある。


.. _RestClientHowToUseBigFileDownload:

**ファイルダウンロードの実装例（ファイルサイズが大きい場合）**

.. code-block:: java

    // (1)
    final ResponseExtractor<ResponseEntity<File>> responseExtractor = 
            new ResponseExtractor<ResponseEntity<File>>() {

        // (2)
        @Override
        public ResponseEntity<File> extractData(ClientHttpResponse response)
                throws IOException {
            
            File rcvFile = File.createTempFile("rcvFile", "zip");

            FileCopyUtils.copy(response.getBody(), new FileOutputStream(rcvFile));
            
            return ResponseEntity.status(response.getStatusCode())
                    .headers(response.getHeaders()).body(rcvFile);
        }

    };

    // (3)
    ResponseEntity<File> responseEntity = this.restTemplate.execute(targetUri,
            HttpMethod.GET, null, responseExtractor);
    if (HttpStatus.OK.equals(responseEntity.getStatusCode())) {
        File getFile = responseEntity.getBody();
        
        .....
        
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``RestTemplate#execute``\ で取得されたレスポンスから、\ ``RestTemplate#execute``\ の戻り値を作成するための処理を作成する。
    * - | (2)
      - | レスポンスボディ(\ ``InputStream``\ )からデータを読込み、ファイルを作成する。
        | 作成したファイルとHTTPヘッダ、ステータスコードを \ ``ResponseEntity<File>``\ に格納して返却する。
    * - | (3)
      - | \ ``RestTemplate#execute``\ を使用して、ファイルのダウンロードを行う。


**ファイルダウンロードの実装例（ファイルサイズが大きい場合（ResponseEntityを使わない例））**
  
ステータスコードの判定やHTTPヘッダの参照等が不要な場合は、 以下のように\ ``ResponseEntity``\ ではなく \ ``File``\ を返却すればよい。
  
.. code-block:: java

    final ResponseExtractor<File> responseExtractor = new ResponseExtractor<File>() {

        @Override
        public File extractData(ClientHttpResponse response)
                throws IOException {

            File rcvFile = File.createTempFile("rcvFile", "zip");

            FileCopyUtils.copy(response.getBody(), new FileOutputStream(
                    rcvFile));

            return rcvFile;
        }

    };

    File getFile = this.restTemplate.execute(targetUri, HttpMethod.GET,
            null, responseExtractor);
    .....


.. _RestClientHowToUseRestFull:

RESTfulなURL(URIテンプレート)を扱う方法と実装例
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

RESTfulなURLを扱うには、URIテンプレートを使用して実装を行えばよい。

**getForObjectメソッドでの使用例**

フィールド宣言部

.. code-block:: java

    @Value("${api.serverUrl}/api/users/{userId}") // (1)
    String uriStr;


メソッド内部

.. code-block:: java

    User user = restTemplate.getForObject(uriStr, User.class, "0001"); // (2)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | URIテンプレートの変数{userId}は、``RestTeamplate``\の使用時に指定の値に変換される。
    * - | (2)
      - | URIテンプレートの変数1つ目が ``getForObject``\ メソッドの第3引数に指定した値で置換され、『http://localhost:8080/api/users/0001』として処理される。


**exchangeメソッドでの使用例**

.. code-block:: java

    @Value("${api.serverUrl}/api/users/{action}") // (1)
    String uriStr;


メソッド内部

.. code-block:: java

    URI targetUri = UriComponentsBuilder.fromUriString(uriStr).
            buildAndExpand("create").toUri(); //(2)

    User user = new User();

    //...

    RequestEntity<User> requestEntity = RequestEntity
            .post(targetUri)
            .body(user);

    ResponseEntity<User> responseEntity = restTemplate.exchange(requestEntity, User.class);


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | URIテンプレートの変数{action}は、``RestTeamplate``\の使用時に指定の値に変換される。
    * - | (2)
      - | ``UriComponentsBuilder``\ を使用することで、URIテンプレートの変数1つ目が ``buildAndExpand``\ の引数で指定した値に置換され、『http://localhost:8080/api/users/create』のURIが作成される。
        | 詳細は\ `UriComponentsBuilder <http://docs.spring.io/spring/docs/4.3.5.RELEASE/javadoc-api/org/springframework/web/util/UriComponentsBuilder.html>`_\ のJavadocを参照されたい。





.. _RestClientHowToExtend:

How to extend
--------------------------------------------------------------------------------

本節では、\ ``RestTemplate``\ の拡張方法について説明する。

.. _RestClientHowToExtendHttpMessageConverter:

任意の\ ``HttpMessageConverter``\ を登録する方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

デフォルトで登録されている \ ``HttpMessageConverter``\ で電文変換の要件を満たせない場合は、任意の\ ``HttpMessageConverter``\ を登録することができる。
ただし、デフォルトで登録されていた\ ``HttpMessageConverter``\ は削除されるので、必要な\ ``HttpMessageConverter``\ をすべて個別に登録する必要がある。

**bean定義ファイル(applicationContext.xml)の定義例**

.. code-block:: xml

    <bean id="jaxb2CollectionHttpMessageConverter"
          class="org.springframework.http.converter.xml.Jaxb2CollectionHttpMessageConverter" /> <!-- (1) -->

    <bean id="restTemplate" class="org.springframework.web.client.RestTemplate">
        <property name="messageConverters"> <!-- (2) -->
            <list>
                <ref bean="jaxb2CollectionHttpMessageConverter" />
            </list>
        </property>
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 登録する\ ``HttpMessageConverter``\ の実装クラスをbean定義する。
    * - | (2)
      - | \ ``messageConverters``\ プロパティに登録する\ ``HttpMessageConverter``\ のbeanをインジェクションする。


.. _RestClientHowToExtendClientHttpRequestInterceptor:

共通処理の適用（\ ``ClientHttpRequestInterceptor``\）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``ClientHttpRequestInterceptor``\ を使用することで、サーバとの通信処理の前後に任意の処理を実行させることができる。

ここでは、

* :ref:`RestClientHowToExtendClientHttpRequestInterceptorLogging`
* :ref:`RestClientHowToExtendClientHttpRequestInterceptorBasicAuthentication`

の実装例を紹介する。

.. _RestClientHowToExtendClientHttpRequestInterceptorLogging:

ロギング処理
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

サーバとの通信ログを出力したい場合は、以下のような実装を行う。

**通信ログ出力の実装例**

.. code-block:: java

    package com.example.restclient;

    import org.springframework.http.HttpRequest;
    import org.springframework.http.client.ClientHttpRequestExecution;
    import org.springframework.http.client.ClientHttpRequestInterceptor;
    import org.springframework.http.client.ClientHttpResponse;

    public class LoggingInterceptor implements ClientHttpRequestInterceptor { //(1)

        private static final Logger log = LoggerFactory.getLogger(LoggingInterceptor.class);

        @Override
        public ClientHttpResponse intercept(HttpRequest request, byte[] body,
                ClientHttpRequestExecution execution) throws IOException {

            if (log.isInfoEnabled()) {
                String requestBody = new String(body, StandardCharsets.UTF_8);

                log.info("Request Header {}", request.getHeaders()); //(2)
                log.info("Request Body {}", requestBody);
            }

            ClientHttpResponse response = execution.execute(request, body); //(3)
          
            if (log.isInfoEnabled()) {
                log.info("Response Header {}", response.getHeaders()); // (4)
                log.info("Response Status Code {}", response.getStatusCode()); // (5)
            }

            return response; // (6)
        }

    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``ClientHttpRequestInterceptor``\ インタフェースを実装する。
    * - | (2)
      - | リクエストする前に行いたい共通処理を実装する。
        | 上記の実装例では、リクエストヘッとリクエストボディの内容をログに出力している。
    * - | (3)
      - | \ ``intercept``\ メソッドの引数として受け取った\ ``ClientHttpRequestExecution``\ の\ ``execute``\ メソッドを実行し、リクエストの送信を実行する。
    * - | (4)
      - | レスポンスを受け取った後に行いたい共通処理を実装する。
        | 上記の実装例では、レスポンスヘッダの内容をログに出力している。
    * - | (5)
      - | (4)と同様に、ステータスコードの内容をログに出力している。
    * - | (6)
      - | (3)で受信したレスポンスをリターンする。


.. _RestClientHowToExtendClientHttpRequestInterceptorBasicAuthentication:

Basic認証用のリクエストヘッダ設定処理
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

サーバにアクセスするためにBasic認証用のリクエストヘッダを設定する必要がある場合は、以下のような実装を行う。

**Basic認証用のリクエストヘッダ設定処理の実装例**

.. code-block:: java

    package com.example.restclient;

    import org.springframework.http.HttpRequest;
    import org.springframework.http.client.ClientHttpRequestExecution;
    import org.springframework.http.client.ClientHttpRequestInterceptor;
    import org.springframework.http.client.ClientHttpResponse;

    public class BasicAuthInterceptor implements ClientHttpRequestInterceptor { //(1)

        private static final Logger log = LoggerFactory.getLogger(BasicAuthInterceptor.class);

        @Value("${api.auth.userid}")
        String userid;

        @Value("${api.auth.password}")
        String password;

        @Override
        public ClientHttpResponse intercept(HttpRequest request, byte[] body,
                ClientHttpRequestExecution execution) throws IOException {
          
            String plainCredentials = userid + ":" + password;
            String base64Credentials = Base64.getEncoder()
                    .encodeToString(plainCredentials.getBytes(StandardCharsets.UTF_8));
            request.getHeaders().add("Authorization", "Basic " + base64Credentials); // (1)

            ClientHttpResponse response = execution.execute(request, body);
          
            return response;
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ``intercept``\ メソッド内で、Basic認証のリクエストヘッダを追加する。


\ ``ClientHttpRequestInterceptor``\ の適用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``RestTemplate``\ に作成した\ ``ClientHttpRequestInterceptor``\ を適用する場合は、以下のようなbean定義を行う。

**bean定義ファイル(applicationContext.xml)の定義例**

.. code-block:: xml

    <!-- (1) -->
    <bean id="basicAuthInterceptor" class="com.example.restclient.BasicAuthInterceptor" />
    <bean id="loggingInterceptor" class="com.example.restclient.LoggingInterceptor" />

    <bean id="restTemplate" class="org.springframework.web.client.RestTemplate">
        <property name="interceptors"><!-- (2) -->
            <list>
                <ref bean="basicAuthInterceptor" />
                <ref bean="loggingInterceptor" />
            </list>
        </property>
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``ClientHttpRequestInterceptor``\ の実装クラスのbean定義を行う。
    * - | (2)
      - | ``interceptors``\ プロパティに\ ``ClientHttpRequestInterceptor``\ のbeanをインジェクションする。
        | 複数のbeanをインジェクションした場合は、リストの先頭から順にチェーン実行される。
        | 上記の例だと、\ ``BasicAuthInterceptor``\  -> \ ``LoggingInterceptor``\  -> \ ``ClientHttpRequest``\  の順番でリクエスト前の処理が実行される。(レスポンス後の処理は順番が逆転する)


.. _RestClientAsync:

非同期リクエスト
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

非同期リクエストを行う場合は、\ ``org.springframework.web.client.AsyncRestTemplate``\ を使用する。

.. _RestClientAsyncBeanDefinition:

\ ``AsyncRestTemplate``\ のbean定義
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

``AsyncRestTemplate``\ のbean定義を行う。

**bean定義ファイル(applicationContext.xml)の定義例**

.. code-block:: xml

    <bean id="asyncRestTemplate" class="org.springframework.web.client.AsyncRestTemplate" /> <!-- (1) -->


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``AsyncRestTemplate``\ をデフォルト設定のまま利用する場合は、デフォルトコンストラクタを使用してbeanを登録する。
        | デフォルト設定の場合、\ ``AsyncRestTemplate``\ の\ ``org.springframework.http.client.AsyncClientHttpRequestFactory``\ には、\ ``org.springframework.core.task.AsyncListenableTaskExecutor``\ として\ ``org.springframework.core.task.SimpleAsyncTaskExecutor``\ が設定された \ ``SimpleClientHttpRequestFactory``\ が設定される。


.. note:: **AsyncRestTemplateのカスタマイズ方法**

    デフォルトで設定される\ ``SimpleAsyncTaskExecutor``\ は、スレッドプールを使わずにスレッドを生成しており、
    スレッドの同時実行数に制限は無い。
    そのため、同時に使用するスレッド数が非常に大きい場合はOutOfMemoryErrorが発生する可能性がある。
    
    \ ``AsyncRestTemplate``\のコンストラクタに\ ``org.springframework.core.task.AsyncListenableTaskExecutor``\ インターフェースのBeanを設定することで、スレッドプール数の上限などを指定できる。
    下記は\ ``org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor``\ を設定する例である。

     .. code-block:: xml

        <!-- (1) -->
        <bean id="asyncTaskExecutor" class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor">
            <property name="maxPoolSize" value="100" />
        </bean>

        <!-- (2) -->
        <bean id="asyncRestTemplate" class="org.springframework.web.client.AsyncRestTemplate" >
            <constructor-arg index="0" ref="asyncTaskExecutor" />
        </bean>


     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90

        * - 項番
          - 説明
        * - | (1)
          - | \ ``AsyncTaskExecutor``\ のbean定義を行う。
            | \ ``ThreadPoolTaskExecutor``\ を使うことで、スレッドプールを使ったスレッド運用が行われる。
            | また、\ ``maxPoolSize``\ プロパティを設定することで、スレッド数の制御が行える。
        * - | (2)
          - | \ ``AsyncRestTemplate``\ のbean定義を行う。
            | \ ``ThreadPoolTaskExecutor``\ を引数に指定するコンストラクタを使用してbeanを登録する。

    本ガイドラインでは、タスク実行処理をカスタマイズする実装例のみを紹介するが、
    \ ``AsyncRestTemplate``\は、HTTP通信処理もカスタマイズ出来る。
    詳細は\ `AsyncRestTemplate <http://docs.spring.io/spring/docs/4.3.5.RELEASE/javadoc-api/org/springframework/web/client/AsyncRestTemplate.html>`_\ のJavadocを参照されたい。
    
    また、\ ``ThreadPoolTaskExecutor``\ についても、スレッドプールサイズ以外のカスタマイズが出来る。
    詳細は\ `ThreadPoolTaskExecutor <http://docs.spring.io/spring/docs/4.3.5.RELEASE/javadoc-api/org/springframework/scheduling/concurrent/ThreadPoolTaskExecutor.html>`_\ のJavadocを参照されたい。



.. _RestClientAsyncImplementation:

非同期リクエストの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**非同期リクエストの実装例**

フィールド宣言部

.. code-block:: java

    @Inject
    AsyncRestTemplate asyncRestTemplate;


メソッド内部

.. code-block:: java

    ListenableFuture<ResponseEntity<User>> responseEntity =
            asyncRestTemplate.getForEntity(uri, User.class); // (1)

    responseEntity.addCallback(new ListenableFutureCallback<ResponseEntity<User>>() { // (2)
        @Override
        public void onSuccess(ResponseEntity<User> entity) {
            //...
        }

        @Override
        public void onFailure(Throwable t) {
          //...
        }
    });


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ``AsyncRestTemplate``\ の各メソッドを使用して、非同期リクエストを送信する。
        | 上記の実装例では、\ ``getForEntity``\ メソッドを使用している。
        | 戻り値は、\ ``org.springframework.util.concurrent.ListenableFuture``\ にラップされた、``ResponseEntity``\ となっている。
        | 各メソッドの使用方法は、\ ``RestTemplate``\ と似たものとなっている。
    * - | (2)
      - | ``ListenableFuture``\ に\ ``org.springframework.util.concurrent.ListenableFutureCallback``\ を登録して、レスポンスが返ってきた際の処理を実装する。
        | 成功のレスポンスが返ってきた場合の処理は\ ``onSuccess``\ メソッドに、エラーが発生した場合の処理は\ ``onFailure``\ メソッドに実装する。


非同期リクエストの共通処理の実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\ ``org.springframework.http.client.AsyncClientHttpRequestInterceptor``\ を使用することで、サーバとの通信処理の前後に任意の処理を実行させることができる。

ここでは、ロギング処理の実装例を紹介する。

**通信ログ出力の実装例**

.. code-block:: java

    package com.example.restclient;

    import java.io.IOException;
    import java.nio.charset.StandardCharsets;

    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.http.HttpRequest;
    import org.springframework.http.client.AsyncClientHttpRequestExecution;
    import org.springframework.http.client.AsyncClientHttpRequestInterceptor;
    import org.springframework.http.client.ClientHttpResponse;
    import org.springframework.util.concurrent.ListenableFuture;
    import org.springframework.util.concurrent.ListenableFutureCallback;

    public class AsyncLoggingInterceptor implements
                                         AsyncClientHttpRequestInterceptor { // (1)
        private static final Logger log = LoggerFactory.getLogger(
                AsyncLoggingInterceptor.class);

        @Override
        public ListenableFuture<ClientHttpResponse> intercept(HttpRequest request,
                byte[] body,
                AsyncClientHttpRequestExecution execution) throws IOException {
            // (2)
            if (log.isInfoEnabled()) {
                String requestBody = new String(body, StandardCharsets.UTF_8);

                log.info("Request Header {}", request.getHeaders());
                log.info("Request Body {}", requestBody);
            }

            // (3)
            ListenableFuture<ClientHttpResponse> future = execution.executeAsync(
                    request, body);
            if (log.isInfoEnabled()) {
                // (4)
                future.addCallback(new ListenableFutureCallback<ClientHttpResponse>() {

                    @Override
                    public void onSuccess(ClientHttpResponse response) {
                        try {
                            log.info("Response Header {}", response
                                    .getHeaders());
                            log.info("Response Status Code {}", response
                                    .getStatusCode());
                        } catch (IOException e) {
                            log.warn("I/O Error", e);
                        }
                    }

                    @Override
                    public void onFailure(Throwable e) {
                        log.info("Communication Error", e);
                    }
                });
            }

            return future; // (5)
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``AsyncClientHttpRequestInterceptor``\ インタフェースを実装する。
    * - | (2)
      - | 非同期リクエストを送信する前に実行する処理を実装する。
        | 上記の実装例では、リクエストヘッダとリクエストボディの内容をログに出力している。
    * - | (3)
      - | \ ``intercept``\ メソッドの引数として受け取った\ ``AsyncClientHttpRequestExecution``\ の\ ``executeAsync``\ メソッドを使用して、非同期リクエストを送信する。
    * - | (4)
      - | (3)で受け取った\ ``ListenableFuture``\ に\ ``org.springframework.util.concurrent.ListenableFutureCallback``\ を登録して、レスポンスが返ってきた際の処理を実装する。
        | レスポンスが返却された場合、\ ``onSuccess``\ メソッドが呼び出される。
        | また、非同期リクエスト時に例外が発生した場合、\ ``onFailure``\ メソッドが呼び出される。以下に具体例を示す。

        * 指定したホストに接続できない（\ ``ConnectException``\ ）
        * レスポンスデータの読み込みタイムアウトが発生した（\ ``SocketTimeoutException``\ ）

    * - | (5)
      - | (3)で受け取った\ ``ListenableFuture``\ をリターンする。


**bean定義ファイル(applicationContext.xml)の定義例**

.. code-block:: xml

    <!-- (1) -->
    <bean id="asyncLoggingInterceptor" class="com.example.restclient.AsyncLoggingInterceptor" />

    <bean id="asyncRestTemplate" class="org.springframework.web.client.AsyncRestTemplate">
        <property name="interceptors"><!-- (2) -->
            <list>
                <ref bean="asyncLoggingInterceptor" />
            </list>
        </property>
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``AsyncClientHttpRequestInterceptor``\ の実装クラスのbean定義を行う。
    * - | (2)
      - | \ ``interceptors``\ プロパティに\ ``AsyncClientHttpRequestInterceptor``\ のbeanをインジェクションする。
        | 複数のbeanをインジェクションした場合は、\ ``RestTemplate``\ と同様にリストの先頭から順にチェーン実行される。



.. _RestClientAppendix:

Appendix
--------------------------------------------------------------------------------

.. _RestClientProxySettings:

HTTP Proxyサーバの設定方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

サーバへアクセスする際にHTTP Proxyサーバを経由する必要がある場合は、システムプロパティやJVM起動引数、または\ ``RestTemplate``\ のBean定義にてHTTP Proxyサーバの設定が必要となる。
システムプロパティやJVM起動引数に設定した場合、アプリケーション全体に影響を与えてしまうため、\ ``RestTemplate``\ 毎にHTTP Proxyサーバの設定を行う例を紹介する。

\ ``RestTemplate``\ 毎のHTTP Proxyサーバの設定は、\ ``ClientHttpRequestFactory``\ インタフェースのデフォルト実装である\ ``SimpleClientHttpRequestFactory``\ に付与することが可能である。
ただし\ ``SimpleClientHttpRequestFactory``\ では資格情報を設定することはできないため、Proxy認証を行う場合は\ ``HttpComponentsClientHttpRequestFactory``\ を使用する。
\ ``HttpComponentsClientHttpRequestFactory``\ は、\ ``Apache HttpComponents HttpClient``\ を用いてリクエストを生成する\ ``ClientHttpRequestFactory``\ インタフェースの実装クラスである。

SimpleClientHttpRequestFactoryを使用したHTTP Proxyサーバの設定方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

資格情報が不要なHTTP Proxyサーバの接続先の指定は、\ ``RestTemplate``\ でデフォルトで使用されている\ ``SimpleClientHttpRequestFactory``\ に指定することが可能である。

**Bean定義ファイル**

.. code-block:: xml

    <!-- (1) -->
    <bean id="inetSocketAddress" class="java.net.InetSocketAddress" >
        <constructor-arg index="0" value="${rscl.http.proxyHost}" />    <!-- (2) -->
        <constructor-arg index="1" value="${rscl.http.proxyPort}" />    <!-- (3) -->
    </bean>

    <!-- (4) -->
    <bean id="simpleClientRestTemplate" class="org.springframework.web.client.RestTemplate" >
        <constructor-arg>
            <!-- (5) -->
            <bean class="org.springframework.http.client.SimpleClientHttpRequestFactory">
                <!-- (6) -->
                <property name="proxy">
                    <bean class="java.net.Proxy" >
                        <!-- (7) -->
                        <constructor-arg index="0">
                            <util:constant static-field="java.net.Proxy.Type.HTTP"/>
                        </constructor-arg>
                        <constructor-arg index="1" ref="inetSocketAddress"/>
                    </bean>
                </property>
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
      - | \ ``java.net.InetSocketAddress``\ にHTTP Proxyサーバの設定を行う。
    * - | (2)
      - | \ ``InetSocketAddress``\ のコンストラクタの第一引数に、プロパティファイルに設定されたキー\ ``rscl.http.proxyHost``\ の値をHTTP Proxyサーバのホスト名として設定する。
    * - | (3)
      - | \ ``InetSocketAddress``\ のコンストラクタの第二引数に、プロパティファイルに設定されたキー\ ``rscl.http.proxyPort``\ の値をHTTP Proxyサーバのポート番号として設定する。
    * - | (4)
      - | \ ``RestTemplate``\ のBean定義を行う。
    * - | (5)
      - | \ ``RestTemplate``\ のコンストラクタの引数に、\ ``SimpleClientHttpRequestFactory``\ を設定する。
    * - | (6)
      - | \ ``SimpleClientHttpRequestFactory``\ の\ ``proxy``\ プロパティに\ ``java.net.Proxy``\ を設定する。
    * - | (7)
      - | \ ``Proxy``\ のコンストラクタの引数に、\ ``java.net.Proxy.Type.HTTP``\ と生成した\ ``InetSocketAddress``\ を設定する。


HttpComponentsClientHttpRequestFactoryを使用したHTTP Proxyサーバの設定方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

HTTP Proxyサーバの指定方法
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

資格情報が必要なHTTP Proxyサーバの接続先の指定は、\ ``RestTemplate``\ に対して、\ ``HttpComponentsClientHttpRequestFactory``\ を使用し指定する。

**pom.xml**

.. code-block:: xml

    <!-- (1) -->
    <dependency>
        <groupId>org.apache.httpcomponents</groupId>
        <artifactId>httpclient</artifactId>
    </dependency>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``HttpComponentsClientHttpRequestFactory``\ 内で使用する\ ``Apache HTTP Client``\ を使用するために、\ ``Apache HttpComponents Client``\ を :file:`pom.xml` の依存ライブラリに追加する。

.. note::
    上記設定例は、依存ライブラリのバージョンを親プロジェクトである terasoluna-gfw-parent で管理する前提であるため、pom.xmlでのバージョンの指定は不要である。
    上記の依存ライブラリはterasoluna-gfw-parentが利用している\ `Spring IO Platform <http://platform.spring.io/platform/>`_\ で定義済みである。


**Bean定義ファイル**

.. code-block:: xml

    <!-- (1) -->
    <bean id="proxyHttpClientBuilder" class="org.apache.http.impl.client.HttpClientBuilder" factory-method="create" >
        <!-- (2) -->
        <property name="proxy">
            <bean class="org.apache.http.HttpHost" >
                <constructor-arg index="0" value="${rscl.http.proxyHost}" />    <!-- (3) -->
                <constructor-arg index="1" value="${rscl.http.proxyPort}" />    <!-- (4) -->
            </bean>
        </property>
    </bean>

    <!-- (5) -->
    <bean id="proxyRestTemplate" class="org.springframework.web.client.RestTemplate" >
        <constructor-arg>
            <!-- (6) -->
            <bean class="org.springframework.http.client.HttpComponentsClientHttpRequestFactory">
                <!-- (7) -->
                <constructor-arg>
                    <bean factory-bean="proxyHttpClientBuilder" factory-method="build" />
                </constructor-arg>
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
      - | \ ``org.apache.http.impl.client.HttpClientBuilder``\ を使用し、\ ``org.apache.http.client.HttpClient``\ の設定を行う。
    * - | (2)
      - | \ ``HttpClientBuilder``\ の\ ``proxy``\ プロパティに、\ HTTP Proxyサーバの設定を行った\ ``org.apache.http.HttpHost``\ を設定する。
    * - | (3)
      - | \ ``HttpHost``\ のコンストラクタの第一引数に、プロパティファイルに設定されたキー\ ``rscl.http.proxyHost``\ の値をHTTP Proxyサーバのホスト名として設定する。
    * - | (4)
      - | \ ``HttpHost``\ のコンストラクタの第二引数に、プロパティファイルに設定されたキー\ ``rscl.http.proxyPort``\ の値をHTTP Proxyサーバのポート番号として設定する。
    * - | (5)
      - | \ ``RestTemplate``\ のBean定義を行う。
    * - | (6)
      - | \ ``RestTemplate``\ のコンストラクタの引数に、\ ``HttpComponentsClientHttpRequestFactory``\ を設定する。
    * - | (7)
      - | \ ``HttpComponentsClientHttpRequestFactory``\ のコンストラクタの引数に、\ ``HttpClientBuilder``\ から生成した\ ``HttpClient``\ を設定する。


HTTP Proxyサーバの資格情報の指定方法
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

HTTP Proxyサーバにアクセスする際に資格情報(ユーザ名とパスワード)が必要な場合は、\ ``org.apache.http.impl.client.BasicCredentialsProvider``\ を使用し資格情報を設定する。

\ ``BasicCredentialsProvider``\ の\ ``setCredentials``\ メソッドが引数を2つ取るため、セッターインジェクションを利用してBeanを生成することができない。このため、\ ``org.springframework.beans.factory.FactoryBean``\ を利用してBeanを生成する。

**FactoryBeanクラス**

.. code-block:: java

    import org.apache.http.auth.AuthScope;
    import org.apache.http.auth.UsernamePasswordCredentials;
    import org.apache.http.impl.client.BasicCredentialsProvider;
    import org.springframework.beans.factory.FactoryBean;
    import org.springframework.beans.factory.annotation.Value;

    // (1)
    public class BasicCredentialsProviderFactoryBean implements FactoryBean<BasicCredentialsProvider> {

        // (2)
        @Value("${rscl.http.proxyHost}")
        String host;

        // (3)
        @Value("${rscl.http.proxyPort}")
        int port;

        // (4)
        @Value("${rscl.http.proxyUserName}")
        String userName;

        // (5)
        @Value("${rscl.http.proxyPassword}")
        String password;

        @Override
        public BasicCredentialsProvider getObject() throws Exception {

            // (6)
            AuthScope authScope = new AuthScope(this.host, this.port);

            // (7)
            UsernamePasswordCredentials usernamePasswordCredentials =
                    new UsernamePasswordCredentials(this.userName, this.password);

            // (8)
            BasicCredentialsProvider credentialsProvider = new BasicCredentialsProvider();
            credentialsProvider.setCredentials(authScope, usernamePasswordCredentials);

            return credentialsProvider;
        }

        @Override
        public Class<?> getObjectType() {
            return BasicCredentialsProvider.class;
        }

        @Override
        public boolean isSingleton() {
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
      - | \ ``org.springframework.beans.factory.FactoryBean``\ を実装した\ ``BasicCredentialsProviderFactoryBean``\ クラスを定義する。
        | Beanの型に\ ``BasicCredentialsProvider``\ を設定する。
    * - | (2)
      - | プロパティファイルに設定されたキー\ ``rscl.http.proxyHost``\ の値をHTTP Proxyサーバのホスト名として、インスタンス変数に設定する。
    * - | (3)
      - | プロパティファイルに設定されたキー\ ``rscl.http.proxyPort``\ の値をHTTP Proxyサーバのポート番号として、インスタンス変数に設定する。
    * - | (4)
      - | プロパティファイルに設定されたキー\ ``rscl.http.proxyUserName``\ の値をHTTP Proxyサーバのユーザ名として、インスタンス変数に設定する。
    * - | (5)
      - | プロパティファイルに設定されたキー\ ``rscl.http.proxyPassword``\ の値をHTTP Proxyサーバのパスワードとして、インスタンス変数に設定する。
    * - | (6)
      - | \ ``org.apache.http.auth.AuthScope`` \ を作成し資格情報のスコープを設定する。この例は、HTTP Proxyサーバのホスト名とポート番号を指定したものである。その他の設定方法については、\ `AuthScope (Apache HttpClient API) <https://hc.apache.org/httpcomponents-client-ga/httpclient/apidocs/org/apache/http/auth/AuthScope.html>`_\ を参照されたい。
    * - | (7)
      - | \ ``org.apache.http.auth.UsernamePasswordCredentials`` \ を作成し資格情報を設定する。
    * - | (8)
      - | \ ``org.apache.http.impl.client.BasicCredentialsProvider``\ を作成し、\ ``setCredentials``\ メソッドを使用し、資格情報のスコープと資格情報を設定する。


**Bean定義ファイル**

.. code-block:: xml

    <bean id="proxyHttpClientBuilder" class="org.apache.http.impl.client.HttpClientBuilder" factory-method="create">
        <!-- (1) -->
        <property name="defaultCredentialsProvider">
            <bean class="com.example.restclient.BasicCredentialsProviderFactoryBean" />
        </property>
        <property name="proxy">
            <bean id="proxyHost" class="org.apache.http.HttpHost">
                <constructor-arg index="0" value="${rscl.http.proxyHost}" />
                <constructor-arg index="1" value="${rscl.http.proxyPort}" />
            </bean>
        </property>
    </bean>

    <bean id="proxyRestTemplate" class="org.springframework.web.client.RestTemplate">
        <constructor-arg>
            <bean class="org.springframework.http.client.HttpComponentsClientHttpRequestFactory">
                <constructor-arg>
                    <bean factory-bean="proxyHttpClientBuilder" factory-method="build" />
                </constructor-arg>
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
      - | \ ``HttpClientBuilder``\ の\ ``defaultCredentialsProvider``\ プロパティに、\ ``BasicCredentialsProvider``\ を設定する。
        | \ ``BasicCredentialsProvider``\ は、\ ``FactoryBean``\ を実装した\ ``BasicCredentialsProviderFactoryBean``\ を使用しBeanを作成する。


JSONでJSR-310 Date and Time APIを使う場合の設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

リソースを表現するJavaBean(Resourceクラス)のプロパティとしてJSR-310 Date and Time APIを使用する場合の設定は
「\ :ref:`RESTAppendixUsingJSR310_JodaTime` \」を参照されたい。
