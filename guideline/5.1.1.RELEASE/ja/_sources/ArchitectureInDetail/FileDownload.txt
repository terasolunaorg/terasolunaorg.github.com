ファイルダウンロード
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

Overview
--------------------------------------------------------------------------------

| 本節では、Springでクライアントにサーバからファイルをダウンロードする機能について説明する。
| Spring MVCのViewで、ファイルのレンダリングを行うことを推奨する。

\
    .. note::
        コントローラクラスで、ファイルレンダリングのロジックを持たせることは推奨しない。

        理由としては、コントローラの役割から逸脱するためである。
        また、コントローラから分離することで、Viewの入れ替えが、容易にできる。

ファイルのダウンロード処理の概要を、以下に示す。

#. DispatchServletは、コントローラへファイルダウンロードのリクエストを送信する。
#. コントローラは、ファイル表示の情報を取得する。
#. コントローラは、Viewを選択する。
#. ファイルレンダリングは、Viewで行われる。


| SpringベースのWebアプリケーションで、ファイルをレンダリングするため、
| 本ガイドラインでは、カスタムビューを実装することを推奨する。
| Spring フレームワークでは、カスタムビューの実装に
| ``org.springframework.web.servlet.View`` インタフェースを提供している。
|
| **PDFファイルの場合**
| Springから提供されている\ ``org.springframework.web.servlet.view.document.AbstractPdfView``\
| クラスは、modelの情報を用いてPDFファイルをレンダリングするときに、サブクラスとして利用するクラスである。
|
| **Excelファイルの場合**
| Springから提供されている\ ``org.springframework.web.servlet.view.document.AbstractXlsxView``\
| クラスは、modelの情報を用いてExcelファイルをレンダリングするときに、サブクラスとして利用するクラスである。
|
| Spring では上記以外にも、いろいろなViewの実装を提供している。
| Viewの技術詳細は、\ `Spring Reference View technologies <http://docs.spring.io/spring/docs/4.2.4.RELEASE/spring-framework-reference/html/view.html>`_\ を参照されたい。

| 共通ライブラリから提供している、\ ``org.terasoluna.gfw.web.download.AbstractFileDownloadView``\ は、
| 任意のファイルをダウンロードするために使用する抽象クラスである。
| PDFやExcel形式以外のファイルをレンダリングする際に、本クラスをサブクラスに定義する。

|

How to use
--------------------------------------------------------------------------------

PDFファイルのダウンロード
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| PDFファイルのレンダリングには、Springから提供されている、
| \ ``org.springframework.web.servlet.view.document.AbstractPdfView``\ を継承したクラスを作成する必要がある。
| コントローラでPDFダウンロードを実装するための手順は、以下で説明する。

カスタムViewの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**AbstractPdfViewを継承したクラスの実装例**

.. code-block:: java

      @Component  // (1)
      public class SamplePdfView extends AbstractPdfView {  // (2)

        @Override
        protected void buildPdfDocument(Map<String, Object> model,
                Document document, PdfWriter writer, HttpServletRequest request,
                HttpServletResponse response) throws Exception {  // (3)

            document.add(new Paragraph((Date) model.get("serverTime")).toString());
        }
      }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 本例では、\ ``@Component``\ アノテーションを使用して、component-scanの対象としている。
       | 後述する、\ ``org.springframework.web.servlet.view.BeanNameViewResolver``\ の対象とすることができる。
   * - | (2)
     - | \ ``AbstractPdfView``\ を継承する。
   * - | (3)
     - | \ ``buildPdfDocument``\ メソッドを実装する。

| \ ``AbstractPdfView``\ は、PDFのレンダリングに、\ `iText <http://itextpdf.com/>`_\ を利用している。
| そのため、Mavenのpom.xmlに itextの定義を追加する必要がある。

.. code-block:: xml

  <dependencies>
      <!-- omitted -->
      <dependency>
          <groupId>com.lowagie</groupId>
          <artifactId>itext</artifactId>
          <exclusions>
              <exclusion>
                  <artifactId>xml-apis</artifactId>
                  <groupId>xml-apis</groupId>
              </exclusion>
              <exclusion>
                  <artifactId>bctsp-jdk14</artifactId>
                  <groupId>org.bouncycastle</groupId>
              </exclusion>
              <exclusion>
                  <artifactId>jfreechart</artifactId>
                  <groupId>jfree</groupId>
              </exclusion>
              <exclusion>
                  <artifactId>dom4j</artifactId>
                  <groupId>dom4j</groupId>
              </exclusion>
              <exclusion>
                  <groupId>org.swinglabs</groupId>
                  <artifactId>pdf-renderer</artifactId>
              </exclusion>
              <exclusion>
                    <groupId>org.bouncycastle</groupId>
                    <artifactId>bcprov-jdk14</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.bouncycastle</groupId>
            <artifactId>bcprov-jdk14</artifactId>
            <version>1.38</version>
        </dependency>
  </dependencies>
  

\
    .. note::
        itextのバージョンはSpring IO Platformにて定義されている。

.. _viewresolver-label:

ViewResolverの定義
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\ ``org.springframework.web.servlet.view.BeanNameViewResolver``\ とは、
Springのコンテキストで管理されたbean名を用いて実行するViewを選択するクラスである。

\ ``BeanNameViewResolver``\ を使用する際は、通常使用する、

* JSP用の\ ``ViewResolver``\(\ ``InternalResourceViewResolver``\)
* Tiles用の\ ``ViewResolver``\(\ ``TilesViewResolver``\)

より先に\ ``BeanNameViewResolver``\が実行されるように定義する事を推奨する。

.. note::

    Spring Frameworkはさまざまな\ ``ViewResolver``\ を提供しており、複数の\ ``ViewResolver``\をチェーンすることができる。
    そのため、特定の状況では、意図しないViewが選択されてしまうことがある。

    この動作は、\ ``ViewResolver``\に適切な優先順位を設定する事で防ぐことができる。
    優先順位の設定方法は、\ ``ViewResolver``\ の定義方法によって異なる。

    * Spring Framework 4.1から追加された\ ``<mvc:view-resolvers>``\ 要素を使用して\ ``ViewResolver``\ を定義する場合は、子要素に指定する\ ``ViewResolver``\の定義順が優先順位となる。(上から順に実行される)

    * 従来通り\ ``<bean>``\ 要素を使用して\ ``ViewResolver``\ を指定する場合は、\ ``order``\ プロパティに優先順位を設定する。(設定値が小さいものから実行される)

|

**bean定義ファイル**

.. code-block:: xml
   :emphasize-lines: 2

    <mvc:view-resolvers>
        <mvc:bean-name /> <!-- (1) (2) -->
        <mvc:jsp prefix="/WEB-INF/views/" />
    </mvc:view-resolvers>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | Spring Framework 4.1から追加された\ ``<mvc:bean-name>``\ 要素を使用して、\ ``BeanNameViewResolver``\ を定義する。
   * - | (2)
     - | \ ``<mvc:bean-name>``\ 要素を先頭に定義し、通常使用する\ ``ViewResolver``\ (JSP用の\ ``ViewResolver``\ )より優先度を高くする。


.. tip::

    \ ``<mvc:view-resolvers>``\ 要素はSpring Framework 4.1から追加されたXML要素である。
    \ ``<mvc:view-resolvers>``\ 要素を使用すると、\ ``ViewResolver``\ をシンプルに定義することが出来る。

    従来通り\ ``<bean>``\ 要素を使用した場合の定義例を以下に示す。


     .. code-block:: xml
        :emphasize-lines: 1-3

        <bean class="org.springframework.web.servlet.view.BeanNameViewResolver">
            <property name="order" value="0"/>
        </bean>

        <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
            <property name="prefix" value="/WEB-INF/views/" />
            <property name="suffix" value=".jsp" />
            <property name="order" value="1" />
        </bean>

    \ ``order``\ プロパティに、\ ``InternalResourceViewResolver``\ より小さい値を指定し、優先度を高くする。

|

コントローラでのViewの指定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| \ ``BeanNameViewResolver``\ により、コントローラで"samplePdfView"を返却することで、
| Springのコンテキストで管理されたBeanIDにより、"samplePdfView"であるViewが使用される。

**Javaソースコード**

.. code-block:: java

        @RequestMapping(value = "home", params= "pdf", method = RequestMethod.GET)
        public String homePdf(Model model) {
        	model.addAttribute("serverTime", new Date());
        	return "samplePdfView";   // (1)
        }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | "samplePdfView" をメソッドの戻り値として返却することで、
       | Springのコンテキストで管理された、\ ``SamplePdfView``\ クラスが実行される。

| 上記の手順を実行した後、以下に示すようなPDFを開くことができる。

.. figure:: ./images/file-download-pdf.png
   :alt: FILEDOWNLOAD PDF
   :width: 60%
   :align: center

   **Picture - FileDownload PDF**

|

Excelファイルのダウンロード
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| EXCELファイルのレンダリングには、Springから提供されている、
| \ ``org.springframework.web.servlet.view.document.AbstractXlsxView``\ を継承したクラスを作成する必要がある。
| コントローラでEXCELファイルをダウンロードを実装するための手順は、以下で説明する。

カスタムViewの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**AbstractXlsxViewを継承したクラスの実装例**

.. code-block:: java

        @Component  // (1)
        public class SampleExcelView extends AbstractXlsxView {  // (2)

            @Override
            protected void buildExcelDocument(Map<String, Object> model,
                    Workbook workbook, HttpServletRequest request,
                    HttpServletResponse response) throws Exception {  // (3)
                Sheet sheet;
                Cell cell;
 
                sheet = workbook.createSheet("Spring");
                sheet.setDefaultColumnWidth(12);

                // write a text at A1
                cell = getCell(sheet, 0, 0);
                setText(cell, "Spring-Excel test");

                cell = getCell(sheet, 2, 0);
                setText(cell, (Date) model.get("serverTime")).toString());
            }
        }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 本例では、\ ``@Component``\ アノテーションを使用して、component-scanの対象としている。
       | 前述した、\ ``org.springframework.web.servlet.view.BeanNameViewResolver``\ の対象とすることができる。
   * - | (2)
     - | \ ``AbstractXlsxView``\ を継承する。
   * - | (3)
     - | \ ``buildExcelDocument``\ メソッドを実装する。

| \ ``AbstractXlsxView``\ は、EXCELのレンダリングに、\ `Apache POI <http://poi.apache.org/>`_\ を利用している。
| そのため、Mavenのpom.xmlに POIの定義を追加する必要がある。

.. code-block:: xml

  <dependencies>
      <!-- omitted -->
      <dependency>
          <groupId>org.apache.poi</groupId>
          <artifactId>poi-ooxml</artifactId>
      </dependency>
      <exclusions>
          <exclusion>
              <groupId>stax</groupId>
              <artifactId>stax-api</artifactId>
          </exclusion>
      </exclusions>
  </dependencies>
  
\
    .. note::
        poi-ooxmlが依存しているstax-apiはJava SEより標準で提供されるようになったため、不要なライブラリである。また、ライブラリの競合が発生する可能性があることから、 \ ``<exclusions>``\ 要素を追加し、当該ライブラリがアプリケーションに含まれないよう考慮する必要がある。

\
    .. note::
        poi-ooxmlのバージョンはSpring IO Platformにて定義されているものを利用するため、設定例では <version> を省略している。

        また、\ ``AbstractExcelView``\ はSpring Framework 4.2から@Deprecatedとなった。そのため、xlsファイルを使用したい場合も同様に\ ``AbstractXlsxView``\ の使用を推奨する。
        詳細は、\ `AbstractExcelViewのJavaDoc <https://docs.spring.io/spring/docs/4.2.4.RELEASE/javadoc-api/org/springframework/web/servlet/view/document/AbstractExcelView.html>`_\ を参照されたい。
          

ViewResolverの定義
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

設定は、PDFファイルをレンダリングする場合と同様である。詳しくは、\ :ref:`viewresolver-label`\ を参照されたい。

コントローラでのViewの指定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| \ ``BeanNameViewResolver``\ により、コントローラで"sampleExcelView"を返却することで、
| Springのコンテキストで管理されたBeanIDにより、”sampleExcelView”であるViewが使用される。

**Javaソース**

.. code-block:: java

        @RequestMapping(value = "home", params= "excel", method = RequestMethod.GET)
        public String homeExcel(Model model) {
        	model.addAttribute("serverTime", new Date());
        	return "sampleExcelView";  // (1)
        }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | "sampleExcelView" をメソッドの戻り値として返却することで、
       | Springのコンテキストで管理された、\ ``SampleExcelView``\ クラスが実行される。

| 上記の手順を実行した後、以下に示すようなEXCELを開くことができる。

.. figure:: ./images/file-download-excel.png
   :alt: FILEDOWNLOAD EXCEL
   :width: 60%
   :align: center

   **Picture - FileDownload EXCEL**

任意のファイルのダウンロード
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 前述した、PDFやEXCELファイル以外のファイルのダウンロードを行う場合、
| 共通ライブラリが提供している、\ ``org.terasoluna.gfw.web.download.AbstractFileDownloadView``\ を継承したクラスを実装すればよい。
| 他の形式にファイルレンダリングするために、\ ``AbstractFileDownloadView``\では、以下を実装する必要がある。

1. レスポンスボディへの書き込むためのInputStreamを取得する。
2. HTTPヘッダに情報を設定する。

| コントローラでファイルダウンロードを実装するための手順は、以下で説明する。

カスタムViewの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| テキストファイルをダウンロードする例を用いて、説明を行う。

**AbstractFileDownloadViewを継承したクラスの実装例**

.. code-block:: java

        @Component  // (1)
        public class TextFileDownloadView extends AbstractFileDownloadView {  // (2)

           @Override
           protected InputStream getInputStream(Map<String, Object> model,
                   HttpServletRequest request) throws IOException {  // (3)
               Resource resource = new ClassPathResource("abc.txt");
               return resource.getInputStream();
           }

           @Override
           protected void addResponseHeader(Map<String, Object> model,
                   HttpServletRequest request, HttpServletResponse response) {  // (4)
               response.setHeader("Content-Disposition",
                       "attachment; filename=abc.txt");
               response.setContentType("text/plain");

           }
        }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 本例では、\ ``@Component``\ アノテーションを使用して、component-scanの対象としている。
       | 前述した、\ ``org.springframework.web.servlet.view.BeanNameViewResolver``\ の対象とすることができる。
   * - | (2)
     - | \ ``AbstractFileDownloadView``\ を継承する。
   * - | (3)
     - | \ ``getInputStream``\ メソッドを実装する。
       | ダウンロード対象の、\ ``InputStream``\ を返却すること。
   * - | (4)
     - | \ ``addResponseHeaderメソッド``\ を実装する。
       | ダウンロードするファイルに合わせた、 Content-Dispositionや、ContentTypeを設定する。

ViewResolverの定義
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

設定は、PDFファイルをレンダリングする場合と同様である。詳しくは、\ :ref:`viewresolver-label`\ を参照されたい。

コントローラでのViewの指定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| \ ``BeanNameViewResolver``\ により、コントローラで"textFileDownloadView"を返却することで、
| Springのコンテキストで管理されたBeanIDにより、”textFileDownloadView”であるViewが使用される。

**Javaソース**

.. code-block:: java

        @RequestMapping(value = "download", method = RequestMethod.GET)
        public String download() {
            return "textFileDownloadView"; // (1)
        }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | "textFileDownloadView" をメソッドの戻り値として返却することで、
       | Springのコンテキストで管理された、\ ``TextFileDownloadView``\ クラスが実行される。

\

    .. tip::

        前述してきたように、SpringはModelの情報をいろいろなViewにレンダリングすることができる。
        Springでは、Jasper Reportsのようなレンダリングエンジンをサポートし、さまざまなViewを返却することも可能である。
        詳細は、Spring の公式ドキュメント\ `Spring reference <http://docs.spring.io/spring/docs/4.2.4.RELEASE/spring-framework-reference/html/view.html#view-jasper-reports>`_\ を参照されたい。

.. raw:: latex

   \newpage

