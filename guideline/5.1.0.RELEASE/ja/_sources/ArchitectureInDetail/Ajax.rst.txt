Ajax
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

Overview
--------------------------------------------------------------------------------

本章では、Ajaxを利用するアプリケーションの実装方法について説明する。

 .. todo::
    
    **TBD**

        クライアント側の実装方法などについては、次版以降で詳細化する予定である。

Ajaxとは、以下の処理を非同期に行うための技術の総称である。

* ブラウザ上で行われる画面操作
* 画面操作をトリガーとしたサーバへのHTTP通信、及び通信結果のユーザインタフェースへの反映

| Ajaxを使うことで、HTTP通信中に画面の操作を継続できるため、ユーザビリティの向上を目的として使用されることが多い。
| この技術の代表的な適用例としては、検索サイトにおける検索ワードのSuggestion機能やリアルタイム検索などがあげられる。

|

.. _ajax_how_to_use:

How to use
--------------------------------------------------------------------------------

アプリケーションの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Ajax向けのアプリケーションの設定について説明する。

.. warning:: **StAX(Streaming API for XML)使用時のDOS攻撃対策について**

    XML形式のデータをStAXを使用して解析する場合は、DTDを使ったDOS攻撃を受けないように対応する必要がある。
    詳細は、\ `CVE-2015-3192 - DoS Attack with XML Input <http://pivotal.io/security/cve-2015-3192>`_\ を参照されたい。

Spring MVCのAjax関連の機能を有効化するための設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Ajax通信時で使用されるContent-Type(``"application/xml"`` や ``"application/json"`` など)を、Controllerのハンドラメソッドでハンドリングできるようにする。

- :file:`spring-mvc.xml`

 .. code-block:: xml

    <mvc:annotation-driven /> <!-- (1) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | 項番
     - | 説明
   * - | (1)
     - | ``<mvc:annotation-driven>`` 要素が指定されていると、Ajax通信時で必要となる機能が有効化されている。
       | そのため、Ajax通信用に特別な設定を行う必要はない。

 .. note::
 
    Ajax通信時で必要となる機能とは、具体的には ``org.springframework.http.converter.HttpMessageConverter`` クラスで提供される機能の事をさす。

    ``HttpMessageConverter`` は、以下の役割をもつ。

    * リクエストBodyに格納されているデータからJavaオブジェクトを生成する。
    * JavaオブジェクトからレスポンスBodyに書き込むデータを生成する。



``<mvc:annotation-driven>`` 指定時にデフォルトで有効化される ``HttpMessageConverter`` は以下の通りである。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.15\linewidth}|p{0.45\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 30 15 45

   * - | 項番
     - | クラス名
     - | 対象
       | フォーマット
     - | 説明
   * - 1.
     - | org.springframework.http.converter.json.
       | MappingJackson2HttpMessageConverter
     - | JSON
     - | リクエストBody又はレスポンスBodyとしてJSONを扱うための ``HttpMessageConverter`` 。
       | ブランクプロジェクトでは、 `Jackson <https://github.com/FasterXML/jackson/>`_ を同封しているため、デフォルトの状態で使用することができる。
   * - 2.
     - | org.springframework.http.converter.xml.
       | Jaxb2RootElementHttpMessageConverter
     - | XML
     - | リクエストBody又はレスポンスBodyとしてXMLを扱うための ``HttpMessageConverter`` 。
       | JavaSE6からJAXB2.0が標準で同封されているため、デフォルトの状態で使用することができる。

 .. note::

    **jackson version 1.x.x から jackson version 2.x.xへ変更する場合の注意点** は\ :ref:`こちら <REST_note_changed_jackson_version>`\ を参照されたい。


 .. warning:: **XXE(XML External Entity) Injection 対策について**
 
    Ajax通信でXML形式のデータを扱う場合は、\ `XXE(XML External Entity) Injection <https://www.owasp.org/index.php/XML_External_Entity_(XXE)_Processing>`_\対策を行う必要がある。
    terasoluna-gfw-web 1.0.1.RELEASE以上では、XXE Injection 対策が行われているSpring MVC(3.2.10.RELEASE以上)に依存しているため、個別に対策を行う必要はない。
    
    terasoluna-gfw-web 1.0.0.RELEASEを使用している場合は、XXE Injection対策が行われていないSpring MVC(3.2.4.RELEASE)に依存しているため、Spring-oxmから提供されているクラスを使用すること。
    
    - :file:`spring-mvc.xml`
    
     .. code-block:: xml
    
        <!-- (1) -->
        <bean id="xmlMarshaller" class="org.springframework.oxm.jaxb.Jaxb2Marshaller">
            <property name="packagesToScan" value="com.examples.app" /> <!-- (2) -->
        </bean>
    
        <!-- ... -->
    
        <mvc:annotation-driven>
    
            <mvc:message-converters>
                <!-- (3) -->
                <bean class="org.springframework.http.converter.xml.MarshallingHttpMessageConverter">
                    <property name="marshaller" ref="xmlMarshaller" /> <!-- (4) -->
                    <property name="unmarshaller" ref="xmlMarshaller" /> <!-- (5) -->
                </bean>
            </mvc:message-converters>
    
            <!-- ... -->
    
        </mvc:annotation-driven>
    
        <!-- ... -->
    
     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - | 項番
         - | 説明
       * - | (1)
         - | Spring-oxmから提供されている\ ``Jaxb2Marshaller``\のbean定義を行う。
           | \ ``Jaxb2Marshaller``\はデフォルトの状態で XXE Injection対策が行われている。
       * - | (2)
         - | ``packagesToScan`` プロパティに JAXB用のJavaBean( ``javax.xml.bind.annotation.XmlRootElement`` アノテーションなどが付与されているJavaBean)が格納されているパッケージ名を指定する。
           | 指定したパッケージ配下に格納されているJAXB用のJavaBeanがスキャンされ、marshal、unmarshal対象のJavaBeanとして登録される。
           | ``<context:component-scan>`` の base-package属性と同じ仕組みでスキャンされる。
       * - | (3)
         - | ``<mvc:annotation-driven>`` の子要素である ``<mvc:message-converters>`` 要素に、 ``MarshallingHttpMessageConverter`` のbean定義を追加する。
       * - | (4)
         - | ``marshaller`` プロパティに (1)で定義した ``Jaxb2Marshaller`` のbeanを指定する。
       * - | (5)
         - | ``unmarshaller`` プロパティに (1)で定義した ``Jaxb2Marshaller`` のbeanを指定する。
         
    |

    Spring-oxmを依存するアーティファクトとして追加する。

    - :file:`pom.xml`

     .. code-block:: xml

        <!-- omitted -->

        <!-- (1) -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-oxm</artifactId>
            <version>${org.springframework-version}</version> <!-- (2) -->
        </dependency>

        <!-- omitted -->

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - | 項番
         - | 説明
       * - | (1)
         - | Spring-oxm を依存アーティファクトとして追加する。
       * - | (2)
         - | Springのバージョンは、terasoluna-gfw-parent の :file:`pom.xml` に定義されているSpringのバージョン番号を管理するためのプレースフォルダ(${org.springframework-version})から取得すること。



|

Controllerの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
以降で説明するサンプルコードの前提は以下の通りである。

* 応答データの形式にはJSONを使用する。
* クライアント側には、JQueryを使用する。バージョンは執筆時点の1.x系の最新バージョン(1.10.2)を使用する。

.. warning:: **循環参照への対策**

    \ ``HttpMessageConverter``\ を使用してJavaBeanをJSONやXML形式にシリアライズする際に、
    相互参照関係のオブジェクトをプロパティに保持していると、
    循環参照となり\ ``StackOverflowError``\ や\ ``OutOfMemoryError``\ などが発生するので、注意が必要である。

    循環参照を回避するためには、

    * Jacksonを使用してJSON形式にシリアライズする場合は、シリアライズ対象から除外するプロパティに\ ``@com.fasterxml.jackson.annotation.JsonIgnore``\ アノテーション
    * JAXBを使用してXML形式にシリアライズする場合は、シリアライズ対象から除外するプロパティに\ ``javax.xml.bind.annotation.XmlTransient``\ アノテーション

    を付与すればよい。

    以下にJacksonを使用してJSON形式にシリアライズする際の回避例を示す。

     .. code-block:: java

         public class Order {
             private String orderId;
             private List<OrderLine> orderLines;
             // ...
         }

     .. code-block:: java

         public class OrderLine {
             @JsonIgnore
             private Order order;
             private String itemCode;
             private int quantity;
             // ...
         }

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
         :header-rows: 1
         :widths: 10 90

         * - 項番
           - 説明
         * - | (1)
           - シリアライズ対象から除外するプロパティに対して\ ``@JsonIgnore``\ アノテーションを付与する。

|

データを取得する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Ajaxを使ってデータを取得する方法について説明する。

下記例は、検索ワードに一致する情報を一覧として返却するAjax通信となっている。

- リクエストデータを受け取るためのJavaBean

 .. code-block:: java

    // (1)
    public class SearchCriteria implements Serializable {

        // omitted

        private String freeWord; // (2)

        // omitted setter/getter

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | 項番
     - | 説明
   * - | (1)
     - | リクエストデータを受け取るためのJavaBeanを作成する。
   * - | (2)
     - | プロパティ名は、リクエストパラメータのパラメータ名と一致させる。

|

- 返却するデータを格納するJavaBean

 .. code-block:: java

    // (3)
    public class SearchResult implements Serializable {

        // omitted

        private List<XxxEntity> list;

        // omitted setter/getter

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | 項番
     - | 説明
   * - | (3)
     - | 返却するデータを格納するためのJavaBeanを作成する。

|

- Controller

 .. code-block:: java

    @RequestMapping(value = "search", method = RequestMethod.GET) // (4)
    @ResponseBody // (5)
    public SearchResult search(@Validated SearchCriteria criteria) { // (6)

        SearchResult searchResult = new SearchResult(); // (7)

        // (8)
        // omitted

        return searchResult; // (9)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | 項番
     - | 説明
   * - | (4)
     - | ``@RequestMapping`` アノテーションの method属性に ``RequestMethod.GET`` を指定する。
   * - | (5)
     - | ``@org.springframework.web.bind.annotation.ResponseBody`` アノテーションを付与する。
       | このアノテーションを付与することで、返却したオブジェクトがJSON形式にmarshalされ、レスポンスBodyに設定される。
   * - | (6)
     - | リクエストデータを受け取るためのJavaBeanを引数に指定する。
       | 入力チェックが必要な場合は、 ``@Validated`` を指定する。入力チェックのエラーハンドリングについては、「 :ref:`ajax_how_to_use_input_error` 」を参照されたい。
       | 入力チェックの詳細については、「 :doc:`Validation` 」を参照されたい。
   * - | (7)
     - | 返却するデータを格納するJavaBeanのオブジェクトを生成する。
   * - | (8)
     - | データを検索し、(7)で生成したオブジェクトに検索結果を格納する。
       | 上記例では、実装は省略している。
   * - | (9)
     - | レスポンスBodyにmarshalするためのオブジェクトを返却する。

|

- HTML(JSP)

 .. code-block:: jsp

    <!-- omitted -->

    <meta name="contextPath" content="${pageContext.request.contextPath}" />

    <!-- omitted -->

    <!-- (10)  -->
    <form id="searchForm">
      <input name="freeWord" type="text">
      <button onclick="return searchByFreeWord()">Search</button>
    </form>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | 項番
     - | 説明
   * - | (10)
     - | 検索条件を入力するためのフォーム。
       | 上記例では、検索条件を入力するためのテキストボックスと検索ボタンをもっている。

 .. code-block:: jsp

    <!-- (11) -->
    <script type="text/javascript"
        src="${pageContext.request.contextPath}/resources/vendor/jquery/jquery-1.10.2.js">
    </script>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | 項番
     - | 説明
   * - | (11)
     - | JQueryのJavaScriptファイルを読み込む。
       | 上記例では、JQueryのJavaScriptファイルを読み込むために、 ``/resources/vendor/jquery/jquery-1.10.2.js`` というパスに対してリクエストが送信される。
     

 .. note::
 
    JQueryのJavaScriptファイルを読み込みための設定は、以下の通り。
    以下はブランクプロジェクトで提供されている設定値である。
    
    * :file:`spring-mvc.xml`
    
     .. code-block:: xml

        <!-- (12) -->
        <mvc:resources mapping="/resources/**"
            location="/resources/,classpath:META-INF/resources/"
            cache-period="#{60 * 60}" />
    
     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - | 項番
         - | 説明
       * - | (12)
         - | リソースファイル(JavaScriptファイル, Stylesheetファイル, 画像ファイルなど)を公開するための設定。
           | 上記設定例では、 ``/resources/`` から始まるパスに対してリクエストがあった場合に、warファイル内の ``/resources/`` ディレクトリ又はクラスパス内の ``/META-INF/resources/`` ディレクトリに格納されているファイルが応答される。

    |
           
    上記設定の場合、JQueryのJavaScriptファイルは以下の何れかのパスに配置する必要がある。
    
    * | warファイル内の ``/resources/vendor/jquery/jquery-1.10.2.js``
      | プロジェクト内のパスで表現すると、 ``src/main/webapp/resources/vendor/jquery/jquery-1.10.2.js`` となる。
    * | クラスパス内の ``/META-INF/resources/vendor/jquery/jquery-1.10.2.js``
      | プロジェクト内のパスで表現すると、 ``src/main/resources/META-INF/resources/vendor/jquery/jquery-1.10.2.js`` となる。
    
|
    
- JavaScript

 .. code-block:: javascript

    var contextPath = $("meta[name='contextPath']").attr("content");

    // (13)
    function searchByFreeWord() {
        $.ajax(contextPath + "/ajax/search", {
            type : "GET",
            data : $("#searchForm").serialize(),
            dataType : "json", // (14)

        }).done(function(json) {
            // (15)
            // render search result
            // omitted

        }).fail(function(xhr) {
            // (16)
            // render error message
            // omitted

        });
        return false;
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | 項番
     - | 説明
   * - | (13)
     - | フォームに指定された検索条件をリクエストパラメータに変換し、GETメソッドで ``/ajax/search`` に対してリクエストを送信するAjax関数。
       | 上記例では、ボタンの押下をAjax通信のトリガーとしているが、テキストボックスのキーダウンやキーアップをトリガーとすることでリアルタイム検索などを実現することができる。
   * - | (14)
     - | レスポンスとして受け取るデータ形式を指定する。
       | 上記例では ``"json"`` を指定しているため、Acceptヘッダーに ``"application/json"`` が設定される。
   * - | (15)
     - | Ajax通信が正常終了した時(Httpステータスコードが ``"200"`` の時)の処理を実装する。
       | 上記例では、実装は省略している。
   * - | (16)
     - | Ajax通信が正常終了しなかった時(Httpステータスコードが ``"4xx"`` や ``"5xx"`` の時)の処理を実装する。
       | 上記例では、実装は省略している。
       | エラー処理の実装例は、 :ref:`ajax_post_formdata` を参照されたい。

 .. tip::

    上記例ではWebアプリケーションのコンテキストパス( ``${pageContext.request.contextPath}`` ) をHTMLの ``<meta>`` 要素に設定しておくことで、
    JavaScriptのコードからJSPのコードを排除している。

|

| 上記検索フォームの「Search」ボタンを押下した際には、以下のような通信が発生する。
| ポイントとなる部分にハイライトを設けている。

- リクエストデータ

 .. code-block:: guess
    :emphasize-lines: 1,4

    GET /terasoluna-gfw-web-blank/ajax/search?freeWord= HTTP/1.1
    Host: localhost:9999
    Connection: keep-alive
    Accept: application/json, text/javascript, */*; q=0.01
    X-Requested-With: XMLHttpRequest
    User-Agent: Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/30.0.1599.101 Safari/537.36
    Referer: http://localhost:9999/terasoluna-gfw-web-blank/ajax/xxe
    Accept-Encoding: gzip,deflate,sdch
    Accept-Language: en-US,en;q=0.8,ja;q=0.6
    Cookie: JSESSIONID=3A486604D7DEE62032BA6C073FC6BE9F

|

- レスポンスデータ

 .. code-block:: guess
    :emphasize-lines: 4, 8

    HTTP/1.1 200 OK
    Server: Apache-Coyote/1.1
    X-Track: a8fb8fefaaf64ee2bffc2b0f77050226
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Fri, 25 Oct 2013 13:52:55 GMT

    {"list":[]}

|

.. _ajax_post_formdata:

フォームデータをPOSTする
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Ajaxを使ってフォームのデータをPOSTし、処理結果を取得する方法について説明する。

下記例は、2つの数値を受け取り、加算結果を返却するAjax通信となっている。

- フォームデータを受け取るためのJavaBean

 .. code-block:: java

    // (1)
    public class CalculationParameters implements Serializable {

        // omitted

        private Integer number1;

        private Integer number2;

        // omitted setter/getter

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | 項番
     - | 説明
   * - | (1)
     - | フォームデータを受け取るためのJavaBeanを作成する。

|

- 処理結果を格納するJavaBean

 .. code-block:: java

    // (2)
    public class CalculationResult implements Serializable {

        // omitted

        private int resultNumber;

        // omitted setter/getter

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | 項番
     - | 説明
   * - | (2)
     - | 処理結果を格納するためのJavaBeanを作成する。

|

- Controller

 .. code-block:: java

    @RequestMapping("xxx")
    @Controller
    public class XxxController {

        @RequestMapping(value = "plusForForm", method = RequestMethod.POST) // (3)
        @ResponseBody
        public CalculationResult plusForForm(
            @Validated CalculationParameters params) { // (4)
            CalculationResult result = new CalculationResult();
            int sum = params.getNumber1() + params.getNumber2();
            result.setResultNumber(sum); // (5)
            return result; // (6)
        }
        
        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | 項番
     - | 説明
   * - | (3)
     - | ``@RequestMapping`` アノテーションの method属性に ``RequestMethod.POST`` を指定する。
   * - | (4)
     - | フォームデータを受け取るためのJavaBeanを引数に指定する。
       | 入力チェックが必要な場合は、 ``@Validated`` を指定する。入力チェックのエラーハンドリングについては、「 :ref:`ajax_how_to_use_input_error` 」を参照されたい。
       | 入力チェックの詳細については、「 :doc:`Validation` 」を参照されたい。
   * - | (5)
     - | 処理結果を格納するオブジェクトに処理結果を格納する。
       | 上記例では、フォームオブジェクトから取得した２つの数値を加算した結果を格納している。
   * - | (6)
     - | レスポンスBodyにmarshalするためのオブジェクトを返却する。

|

- HTML(JSP)

 .. code-block:: jsp

    <!-- omitted -->

    <meta name="contextPath" content="${pageContext.request.contextPath}" />

    <sec:csrfMetaTags />

    <!-- omitted -->

    <!-- (7)  -->
    <form id="calculationForm">
        <input name="number1" type="text">+
        <input name="number2" type="text">
        <button onclick="return plus()">=</button>
        <span id="calculationResult"></span> <!-- (8) -->
    </form>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | 項番
     - | 説明
   * - | (7)
     - | 計算対象の数値を入力するためのフォーム。
   * - | (8)
     - | 計算結果を表示するための領域。
       | 上記例では、通信成功時には計算結果が表示され、通信失敗時には計算結果がクリアされる。

|

- JavaScript

 .. code-block:: javascript

    var contextPath = $("meta[name='contextPath']").attr("content");

    // (9)
    var csrfToken = $("meta[name='_csrf']").attr("content");
    var csrfHeaderName = $("meta[name='_csrf_header']").attr("content");
    $(document).ajaxSend(function(event, xhr, options) {
        xhr.setRequestHeader(csrfHeaderName, csrfToken);
    });

    // (10)
    function plus() {
        $.ajax(contextPath + "/ajax/plusForForm", {
            type : "POST",
            data : $("#calculationForm").serialize(),
            dataType : "json"
        }).done(function(json) {
            $("#calculationResult").text(json.resultNumber);

        }).fail(function(xhr) {
            // (11)
            var messages = "";
            // (12)
            if(400 <= xhr.status && xhr.status <= 499){
                // (13)
                var contentType = xhr.getResponseHeader('Content-Type');
                if (contentType != null && contentType.indexOf("json") != -1) {
                    // (14)
                    json = $.parseJSON(xhr.responseText);
                    $(json.errorResults).each(function(i, errorResult) {
                        messages += ("<div>" + errorResult.message + "</div>");
                    });
                } else {
                    // (15)
                    messages = ("<div>" + xhr.statusText + "</div>");
                }
            }else{
                // (16)
                messages = ("<div>" + "System error occurred." + "</div>");
            }
            // (17)
            $("#calculationResult").html(messages);
        });

        return false;
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | 項番
     - | 説明
   * - | (9)
     - | POSTメソッドでリクエストを行う場合、CSRFトークンをHTTPヘッダに設定して送信する必要がある。
       | 上記例では、\ ``<sec:csrfMetaTags />``\ を利用して ``<meta>`` 要素にCSRFトークンヘッダー名とCSRFトークン値を設定し、JavaScriptで値を取得するようにしている。
       | CSRF対策の詳細については、 「 :doc:`../Security/CSRF` 」を参照されたい。
   * - | (10)
     - | フォームに指定された数値をリクエストパラメータに変換し、POSTメソッドで ``/ajax/plusForForm`` に対してリクエストを送信するAjax関数。
       | 上記例では、ボタンの押下をAjax通信のトリガーとしているが、テキストボックスのロストフォーカスをトリガーとすることでリアルタイム計算を実現することができる。
   * - | (11)
     - | エラー処理の実装例を以下に示す。
       | サーバ側のエラーハンドリング処理の実装例については、 :ref:`ajax_how_to_use_input_error` を参照されたい。
   * - | (12)
     - | HTTPのステータスコードを判定し、どのようなエラーが発生したか判定する。
       | HTTPのステータスコードは、 XMLHttpRequestオブジェクトの ``status`` フィールドに格納されている。
   * - | (13)
     - | レスポンスされたデータがJSON形式か判定を行う。
       | 上記例では、レスポンスヘッダの Content-Typeに設定されている値を参照して、レスポンスされたデータの形式をチェックしている。
       | 形式をチェックしておかないと、JSON以外の形式で応答された際に、JSONオブジェクトにデシリアライズする処理でエラーが発生することになる。
       | サーバ側のエラーハンドリングを簡易的に行っていると、HTML形式のページが返却されることがある。
   * - | (14)
     - | レスポンスデータをJSONオブジェクトにデシリアライズする。
       | レスポンスデータは、 XMLHttpRequestオブジェクトの ``responseText`` フィールドに格納されている。
       | 上記例では、デシリアライズしたJSONオブジェクトからエラー情報を取得し、エラーメッセージを組み立てている。
   * - | (15)
     - | レスポンスされたデータがJSON形式以外だった場合の処理を行う。
       | 上記例では、HTTPのステータステキストをエラーメッセージに格納している。
       | HTTPのステータステキストは、 XMLHttpRequestオブジェクトの ``statusText`` フィールドに格納されている。
   * - | (16)
     - | サーバエラー時の処理を行う。
       | 上記例では、システムエラーが発生したことを通知するメッセージをエラーメッセージに格納している。
   * - | (17)
     - | エラー時の描画処理を行う。
       | 上記例では、計算結果を表示するための領域に、エラーメッセージを表示している。

 .. warning::
 
    上記例では、Ajaxの通信処理、DOM操作処理(描画処理)、エラー処理を同じfunction内で行っているが、これらの処理は分離して実装することを推奨する。

 .. todo:: **TBD**
    
    クライアント側の実装方法については、次版以降で詳細化する予定である。

 .. tip::

    上記例では\ ``<sec:csrfMetaTags />``\ を利用して、CSRFトークン値とCSRFトークンヘッダー名をHTMLの ``<meta>`` 要素に設定しておくことで、
    JavaScriptのコードからJSPのコードを排除している。\ :ref:`csrf_ajax-token-setting`\ を参照されたい。

    尚、CSRFトークン値とCSRFトークンヘッダー名はそれぞれ\ ``${_csrf.token}``\ と\ ``${_csrf.headerName}``\ を用いても取得可能である。

|

| 上記検索フォームの「=」ボタンを押下した際には、以下のような通信が発生する。
| ポイントとなる部分にハイライトを設けている。

- リクエストデータ

 .. code-block:: guess
    :emphasize-lines: 1,5,7,10,16

    POST /terasoluna-gfw-web-blank/ajax/plusForForm HTTP/1.1
    Host: localhost:9999
    Connection: keep-alive
    Content-Length: 19
    Accept: application/json, text/javascript, */*; q=0.01
    Origin: http://localhost:9999
    X-CSRF-TOKEN: a5dd1858-8a4f-4ecc-88bd-a326388ab5c9
    X-Requested-With: XMLHttpRequest
    User-Agent: Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/30.0.1599.101 Safari/537.36
    Content-Type: application/x-www-form-urlencoded; charset=UTF-8
    Referer: http://localhost:9999/terasoluna-gfw-web-blank/ajax/xxe
    Accept-Encoding: gzip,deflate,sdch
    Accept-Language: en-US,en;q=0.8,ja;q=0.6
    Cookie: JSESSIONID=3A486604D7DEE62032BA6C073FC6BE9F

    number1=1&number2=2

|

- レスポンスデータ

 .. code-block:: guess
    :emphasize-lines: 4, 8

    HTTP/1.1 200 OK
    Server: Apache-Coyote/1.1
    X-Track: c2d5066d0fa946f584536775f07d1900
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Fri, 25 Oct 2013 14:27:55 GMT

    {"resultNumber":3}

|

- エラー時のレスポンスデータ
  下記のレスポンスデータは、入力エラーが発生時のものである。

 .. code-block:: guess
    :emphasize-lines: 1, 4, 9

    HTTP/1.1 400 Bad Request
    Server: Apache-Coyote/1.1
    X-Track: cecd7b4d746249178643b7110b0eaa74
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Wed, 04 Dec 2013 15:06:01 GMT
    Connection: close
    
    {"errorResults":[{"code":"NotNull","message":"\"number2\"maynotbenull.","itemPath":"number2"},{"code":"NotNull","message":"\"number1\"maynotbenull.","itemPath":"number1"}]}

|

フォームデータをJSONとしてPOSTする
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Ajaxを使ってフォームのデータをJSON形式に変換してからPOSTし、処理結果を取得する方法について説明する。

「フォームデータをPOSTする」方法との差分部分について説明する。

- Controller

 .. code-block:: java

    @RequestMapping("xxx")
    @Controller
    public class XxxController {

        @RequestMapping(value = "plusForJson", method = RequestMethod.POST)
        @ResponseBody
        public CalculationResult plusForJson(
                @Validated @RequestBody CalculationParameters params) { // (1)
            CalculationResult result = new CalculationResult();
            int sum = params.getNumber1() + params.getNumber2();
            result.setResultNumber(sum);
            return result;
        }
        
        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | 項番
     - | 説明
   * - | (1)
     - | フォームデータを受け取るためのJavaBeanの引数アノテーションとして、 ``@org.springframework.web.bind.annotation.RequestBody`` アノテーションを付与する。
       | このアノテーションを付与することで、リクエストBodyに格納されているJSON形式のデータがunmarshalされ、オブジェクトに変換される。
       | 入力チェックが必要な場合は、 ``@Validated`` を指定する。入力チェックのエラーハンドリングについては、「 :ref:`ajax_how_to_use_input_error` 」を参照されたい。
       | 入力チェックの詳細については、「 :doc:`Validation` 」を参照されたい。

|

- JavaScript/HTML(JSP)

 .. code-block:: javascript

    // (2)
    function toJson($form) {
        var data = {};
        $($form.serializeArray()).each(function(i, v) {
            data[v.name] = v.value;
        });
        return JSON.stringify(data);
    }

    function plus() {

        $.ajax(contextPath + "/ajax/plusForJson", {
            type : "POST",
            contentType : "application/json;charset=utf-8", // (3)
            data : toJson($("#calculationForm")), // (2)
            dataType : "json",
            beforeSend : function(xhr) {
                xhr.setRequestHeader(csrfHeaderName, csrfToken);
            }

        }).done(function(json) {
            $("#calculationResult").text(json.resultNumber);

        }).fail(function(xhr) {
            $("#calculationResult").text("");

        });
        return false;
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | 項番
     - | 説明
   * - | (2)
     - | フォーム内のinput項目をJSON形式の文字列にするための関数。
   * - | (3)
     - | リクエストBodyにJSONを格納するので、Content-Typeのメディアタイプを ``"application/json"`` にする。


|

| 上記検索フォームの「=」ボタンを押下した際には、以下のような通信が発生する。
| ポイントとなる部分にハイライトを設けている。

- リクエストデータ

 .. code-block:: guess
    :emphasize-lines: 10,16

    POST /terasoluna-gfw-web-blank/ajax/plusForJson HTTP/1.1
    Host: localhost:9999
    Connection: keep-alive
    Content-Length: 31
    Accept: application/json, text/javascript, */*; q=0.01
    Origin: http://localhost:9999
    X-CSRF-TOKEN: 9d4f1e0c-c500-43f3-9125-a7a131ff88fa
    X-Requested-With: XMLHttpRequest
    User-Agent: Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/30.0.1599.101 Safari/537.36
    Content-Type: application/json;charset=UTF-8
    Referer: http://localhost:9999/terasoluna-gfw-web-blank/ajax/xxe?
    Accept-Encoding: gzip,deflate,sdch
    Accept-Language: en-US,en;q=0.8,ja;q=0.6
    Cookie: JSESSIONID=CECD7A6CB0431266B8D1173CCFA66B95

    {"number1":"34","number2":"56"}


|

.. _ajax_how_to_use_input_error:

入力エラーのハンドリング
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
入力値に不正な値が指定された場合のエラーハンドリング方法について説明する。

入力エラーのハンドリング方法は、大きく分けて以下の２つに分類される。

* 例外ハンドリング用のメソッドを用意してエラー処理を行う。

* Controllerのハンドラメソッドの引数として ``org.springframework.validation.BindingResult`` を受け取り、エラー処理を行う。


|

BindException のハンドリング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| ``org.springframework.validation.BindException`` は、 リクエストパラメータとして送信したデータをJavaBeanにバインドする際に、入力値に不正な値が指定された場合に発生する例外クラスである。
| GET時のリクエストパラメータや、フォームデータを ``"application/x-www-form-urlencoded"`` の形式として受け取る場合は、 ``BindException`` の例外ハンドリングが必要となる。

- Controller

 .. code-block:: java

    @RequestMapping("xxx")
    @Controller
    public class XxxController {
    
        // omitted
    
        @ExceptionHandler(BindException.class) // (1)
        @ResponseStatus(value = HttpStatus.BAD_REQUEST) // (2)
        @ResponseBody // (3)
        public ErrorResults handleBindException(BindException e, Locale locale) { // (4)
            // (5)
            ErrorResults errorResults = new ErrorResults();
            for (FieldError fieldError : e.getBindingResult().getFieldErrors()) {
                errorResults.add(fieldError.getCode(),
                        messageSource.getMessage(fieldError, locale),
                            fieldError.getField());
            }
            for (ObjectError objectError : e.getBindingResult().getGlobalErrors()) {
                errorResults.add(objectError.getCode(),
                        messageSource.getMessage(objectError, locale),
                            objectError.getObjectName());
            }
            return errorResults;
        }
    
        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | 項番
     - | 説明
   * - | (1)
     - | Controllerにエラーハンドリング用メソッドを定義する。
       | エラーハンドリング用のメソッドには、``@org.springframework.web.bind.annotation.ExceptionHandler`` アノテーションを付与し、 value属性にハンドリングする例外の型を指定する。
       | 上記例では、 ハンドリング対象の例外として ``BindException.class`` を指定している。
   * - | (2)
     - | 応答するHTTPステータス情報を指定する。
       | 上記例では、 ``400`` (Bad Request) を指定している。
   * - | (3)
     - | 返却したオブジェクトをレスポンスBodyに書き込むため、 ``@ResponseBody`` アノテーションを付与する。
   * - | (4)
     - | エラーハンドリング用のメソッドの引数として、ハンドリング対象の例外クラスを宣言する。
   * - | (5)
     - | エラー処理を実装する。
       | 上記例では、エラー情報を返却するためのJavaBeanを生成し、返却している。

 .. tip::

    エラー処理としてメッセージを生成する際に国際化を意識する必要がある場合は、``Locale`` オブジェクトを引数として受け取ることができる。

|

- エラー情報を保持するJavaBean

 .. code-block:: java

    // (6)
    public class ErrorResult implements Serializable {
    
        private static final long serialVersionUID = 1L;
    
        private String code;
    
        private String message;
    
        private String itemPath;
    
        public String getCode() {
            return code;
        }
    
        public void setCode(String code) {
            this.code = code;
        }
    
        public String getMessage() {
            return message;
        }
    
        public void setMessage(String message) {
            this.message = message;
        }
    
        public String getItemPath() {
            return itemPath;
        }
    
        public void setItemPath(String itemPath) {
            this.itemPath = itemPath;
        }
    
    }

 .. code-block:: java

    // (7)
    public class ErrorResults implements Serializable {
    
        private static final long serialVersionUID = 1L;
    
        private List<ErrorResult> errorResults = new ArrayList<ErrorResult>();
    
        public List<ErrorResult> getErrorResults() {
            return errorResults;
        }
    
        public void setErrorResults(List<ErrorResult> errorResults) {
            this.errorResults = errorResults;
        }
    
        public ErrorResults add(String code, String message) {
            ErrorResult errorResult = new ErrorResult();
            errorResult.setCode(code);
            errorResult.setMessage(message);
            errorResults.add(errorResult);
            return this;
        }
    
        public ErrorResults add(String code, String message, String itemPath) {
            ErrorResult errorResult = new ErrorResult();
            errorResult.setCode(code);
            errorResult.setMessage(message);
            errorResult.setItemPath(itemPath);
            errorResults.add(errorResult);
            return this;
        }
    
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | 項番
     - | 説明
   * - | (6)
     - | エラー情報を１件保持するためのJavaBean。
   * - | (7)
     - | エラー情報を１件保持するJavaBeanを複数件保持するためのJavaBean。
       | (6)のJavaBeanをリストとして保持している。

|

MethodArgumentNotValidException のハンドリング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| ``org.springframework.web.bind.MethodArgumentNotValidException`` は、 ``@RequestBody`` アノテーションを使用してリクエストBodyに格納されているデータをJavaBeanにバインドする際に、入力値に不正な値が指定された場合に発生する例外クラスである。
| ``"application/json"`` や ``"application/xml"`` などの形式として受け取る場合は、 ``MethodArgumentNotValidException`` の例外ハンドリングが必要となる。

- Controller

 .. code-block:: java

    @ExceptionHandler(MethodArgumentNotValidException.class) // (1)
    @ResponseStatus(value = HttpStatus.BAD_REQUEST)
    @ResponseBody
    public ErrorResults handleMethodArgumentNotValidException(
            MethodArgumentNotValidException e, Locale locale) { // (1)
        ErrorResults errorResults = new ErrorResults();

        // implement error handling.
        // omitted

        return errorResults;
    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | 項番
     - | 説明
   * - | (1)
     - | エラーハンドリング対象の例外として ``MethodArgumentNotValidException.class`` を指定する。
       | 上記以外は ``BindException`` と同様。

|

HttpMessageNotReadableException のハンドリング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| ``org.springframework.http.converter.HttpMessageNotReadableException`` は、 ``@RequestBody`` アノテーションを使用してリクエストBodyに格納されているデータをJavaBeanにバインドする際に、Bodyに格納されているデータからJavaBeanを生成できなかった場合に発生する例外クラスである。
| ``"application/json"`` や ``"application/xml"`` などの形式として受け取る場合は、 ``MethodArgumentNotValidException`` の例外ハンドリングが必要となる。

    .. note::

        具体的なエラー原因は、使用する ``HttpMessageConverter`` や利用するライブラリの実装によって異なる。

        JSON形式のデータをJacksonを使ってJavaBeanに変換する ``MappingJackson2HttpMessageConverter`` の実装では、Integer項目に数値以外の文字列を指定すると、 ``HttpMessageNotReadableException`` が発生する。

- Controller

 .. code-block:: java

    @ExceptionHandler(HttpMessageNotReadableException.class) // (1)
    @ResponseStatus(value = HttpStatus.BAD_REQUEST)
    @ResponseBody
    public ErrorResults handleHttpMessageNotReadableException(
            HttpMessageNotReadableException e, Locale locale) {  // (1)
        ErrorResults errorResults = new ErrorResults();

        // implement error handling.
        // omitted

        return errorResults;
    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | 項番
     - | 説明
   * - | (1)
     - | エラーハンドリング対象の例外として ``HttpMessageNotReadableException.class`` を指定する。
       | 上記以外は ``BindException`` と同様。


|

BindingResult を使用したハンドリング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 正常終了時に返却するJavaBeanと入力エラー時に返却するJavaBeanの型が同じ場合は、``BindingResult`` をハンドラメソッドの引数として受け取ることでエラーハンドリングすることができる。
| この方法は、リクエストデータの形式に関係なく使用することができる。
| ハンドラメソッドの引数として ``BindingResult`` を指定しない場合は、前述した例外をハンドリングする方法でエラー処理を実装する必要がある。

- Controller

 .. code-block:: java

    @RequestMapping(value = "plus", method = RequestMethod.POST)
    @ResponseBody
    public CalculationResult plus(
            @Validated @RequestBody CalculationParameters params,
            BindingResult bResult) { // (1)
        CalculationResult result = new CalculationResult();
        if (bResult.hasErrors()) { // (2)

            // (3)
            // implement error handling.
            // omitted

            return result; // (4)
        }
        int sum = params.getNumber1() + params.getNumber2();
        result.setResultNumber(sum);
        return result;
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | 項番
     - | 説明
   * - | (1)
     - | ハンドラメソッドの引数として ``BindingResult`` を宣言する。
       | ``BindingResult`` は入力チェック対象のJavaBeanの直後に宣言する必要がある。
   * - | (2)
     - | 入力値のエラー有無を判定する。
   * - | (3)
     - | 入力値にエラーがある場合は、入力エラー時のエラー処理を行う。
       | 上記例ではエラー処理は省略しているが、エラーメッセージの設定などが行われる想定である。
   * - | (4)
     - | 処理結果を返却する。


 .. note::

    上記例では、正常時及びエラー時共にレスポンスのHTTPステータスコードは ``200`` (OK) が返却される。
    HTTPステータスコードを処理結果によってわける必要がある場合は、 ``org.springframework.http.ResponseEntity`` を返却値とすることで実現可能である。
    別のアプローチとしては、ハンドラメソッドの引数として ``BindingResult`` を指定せず、前述した例外をハンドリングする方法でエラー処理を実装する方法がある。

      .. code-block:: java

        @RequestMapping(value = "plus", method = RequestMethod.POST)
        @ResponseBody
        public ResponseEntity<CalculationResult> plus(
                @Validated @RequestBody CalculationParameters params,
                BindingResult bResult) {
            CalculationResult result = new CalculationResult();
            if (bResult.hasErrors()) {

                // implement error handling.
                // omitted

                // (1)
                return ResponseEntity.badRequest().body(result);
            }
            // omitted

            // (2)
            return ResponseEntity.ok().body(result);
        }

      .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
      .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - | 項番
         - | 説明
       * - | (1)
         - | 入力エラー時の応答データとHTTPステータスを返却する。
       * - | (2)
         - | 正常終了時の応答データとHTTPステータスを返却する。

|

業務エラーのハンドリング
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
業務エラーのエラーハンドリング方法について説明する。

業務エラーのハンドリング方法は大きく分けて以下の２つに分類される。

* 業務例外ハンドリング用のメソッドを用意してエラー処理を行う。

* Controllerのハンドラメソッド内で業務例外をcatchしてエラー処理を行う。


例外ハンドリング用のメソッドで業務例外をハンドリング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 入力エラーと同様、例外ハンドリング用のメソッドを用意して業務例外をハンドリングする。
| 複数のハンドラメソッドに対するリクエストで同じエラー処理を実装する必要がある場合、この方法でエラーハンドリングすることを推奨する。

- Controller

 .. code-block:: java

    @ExceptionHandler(BusinessException.class) // (1)
    @ResponseStatus(value = HttpStatus.CONFLICT) // (2)
    @ResponseBody
    public ErrorResults handleHttpBusinessException(BusinessException e, // (1)
            Locale locale) {
        ErrorResults errorResults = new ErrorResults();

        // implement error handling.
        // omitted

        return errorResults;
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | 項番
     - | 説明
   * - | (1)
     - | エラーハンドリング対象の例外として ``BusinessException.class`` を指定する。
       | 上記以外は入力エラーの ``BindException`` のハンドリング方法と同様。
   * - | (2)
     - | 応答するHTTPステータス情報を指定する。
       | 上記例では、 ``409`` (Conflict) を指定している。

|

ハンドラメソッド内で業務例外をハンドリング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 業務エラーが発生する処理を try句で囲み、業務例外をcatchする。
| エラー処理がリクエスト毎に異なる場合は、この方法でエラーハンドリングすることになる。

- Controller

 .. code-block:: java

    @RequestMapping(value = "plus", method = RequestMethod.POST)
    @ResponseBody
    public ResponseEntity<CalculationResult> plusForJson(
            @Validated @RequestBody CalculationParameters params) {
        CalculationResult result = new CalculationResult();

        // omitted

        // (1)
        try {

            // call service method.
            // omitted

         // (2)
        } catch (BusinessException e) {

            // (3)
            // implement error handling.
            // omitted

            return ResponseEntity.status(HttpStatus.CONFLICT).body(result);
        }

        // omitted

        return ResponseEntity.ok().body(result);
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - | 項番
     - | 説明
   * - | (1)
     - | 業務例外が発生するメソッド呼び出しを try句で囲む。
   * - | (2)
     - | 業務例外をcatchする。
   * - | (3)
     - | 業務例外エラー時のエラー処理を行う。
       | 上記例ではエラー処理は省略しているが、エラーメッセージの設定などが行われる想定である。

.. raw:: latex

   \newpage

