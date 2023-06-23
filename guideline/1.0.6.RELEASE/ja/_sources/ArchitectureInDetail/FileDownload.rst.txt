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
| Springから提供されている\ ``org.springframework.web.servlet.view.document.AbstractExcelView``\
| クラスは、modelの情報を用いてExcelファイルをレンダリングするときに、サブクラスとして利用するクラスである。
|
| Spring では上記以外にも、いろいろなViewの実装を提供している。
| Viewの技術詳細は、\ `Spring Reference View technologies <http://static.springsource.org/spring/docs/3.2.18.RELEASE/spring-framework-reference/html/view.html>`_\ を参照されたい。

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
     - | AbstractPdfViewを継承する。
   * - | (3)
     - | buildPdfDocumentメソッドを実装する。

| AbstractPdfViewは、PDFのレンダリングに、\ `iText <http://itextpdf.com/>`_\ を利用している。
| そのため、Mavenのpom.xmlに itextの定義を追加する必要がある。

.. code-block:: xml

  <dependencies>
      <!-- omitted -->
      <dependency>
          <groupId>com.lowagie</groupId>
          <artifactId>itext</artifactId>
          <version>${com.lowagie.itext.version}</version>
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
          </exclusions>
     </dependency>
  </dependencies>
  
  <properties>
      <!-- omitted -->
      <com.lowagie.itext.version>4.2.1</com.lowagie.itext.version>
  </properties>


\
    .. note::
        Spring 3.2では、itextの5系のバージョンに対応していない。

.. _viewresolver-label:

ViewResolverの定義
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| \ ``org.springframework.web.servlet.view.BeanNameViewResolver``\ とは、Springのコンテキストで管理された、bean名を用いて実行するViewを選択するクラスである。
| BeanNameViewResolverを使用する際は、\ ``order``\ プロパティを設定して、通常使用する\ ``InternalResourceViewResolver``\ より先に呼ばれるようにする必要がある。
\
    .. note::

        Spring はさまざまなView Resolverを提供している。複数のResolverをチェーンすることができるため、特定の状況では、意図しないViewが選択されてしまうことがある。
        それを防ぐために orderプロパティを指定することで、優先順位を設定できる。
        orderプロパティの指定値が低いほど、先に実行される。

**bean定義ファイル**

.. code-block:: xml
   :emphasize-lines: 6-8

    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/" />
        <property name="suffix" value=".jsp" />
    </bean>

    <bean class="org.springframework.web.servlet.view.BeanNameViewResolver">  <!-- (1) -->
        <property name="order" value="0"/>  <!-- (2) -->
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | BeanNameViewResolver を定義する。
   * - | (2)
     - | orderプロパティに0を設定する。InternalResourceViewResolverより、優先度を高くすること。

|

コントローラでのViewの指定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| BeanNameViewResolverにより、コントローラで"samplePdfView"を返却することで、
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
       | Springのコンテキストで管理された、SamplePdfViewクラスが実行される。

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
| \ ``org.springframework.web.servlet.view.document.AbstractExcelView``\ を継承したクラスを作成する必要がある。
| コントローラでEXCELファイルをダウンロードを実装するための手順は、以下で説明する。

カスタムViewの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**AbstractExcelViewを継承したクラスの実装例**

.. code-block:: java

        @Component  // (1)
        public class SampleExcelView extends AbstractExcelView {  // (2)

            @Override
            protected void buildExcelDocument(Map<String, Object> model,
                    HSSFWorkbook workbook, HttpServletRequest request,
                    HttpServletResponse response) throws Exception {  // (3)
                HSSFSheet sheet;
                HSSFCell cell;

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
     - | AbstractExcelViewを継承する。
   * - | (3)
     - | buildExcelDocumentメソッドを実装する。

| AbstractExcelViewは、EXCELのレンダリングに、\ `Apache POI <http://poi.apache.org/>`_\ を利用している。
| そのため、Mavenのpom.xmlに POIの定義を追加する必要がある。

.. code-block:: xml

  <dependencies>
      <!-- omitted -->
      <dependency>
          <groupId>org.apache.poi</groupId>
          <artifactId>poi</artifactId>
          <version>${org.apache.poi.poi.version}</version>
      </dependency>
  </dependencies>
  
  <properties>
      <!-- omitted -->
      <org.apache.poi.poi.version>3.9</org.apache.poi.poi.version>
  </properties>
        

ViewResolverの定義
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

設定は、PDFファイルをレンダリングする場合と同様である。詳しくは、\ :ref:`viewresolver-label`\ を参照されたい。

コントローラでのViewの指定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| BeanNameViewResolverにより、コントローラで"sampleExcelView"を返却することで、
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
       | Springのコンテキストで管理された、SampleExcelViewクラスが実行される。

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
| 他の形式にファイルレンダリングするために、AbstractFileDownloadViewでは、以下を実装する必要がある。

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
     - | AbstractFileDownloadViewを継承する。
   * - | (3)
     - | getInputStreamメソッドを実装する。
       | ダウンロード対象の、InputStreameを返却すること。
   * - | (4)
     - | addResponseHeaderメソッドを実装する。
       | ダウンロードするファイルに合わせた、 Content-Dispositionや、ContentTypeを設定する。

ViewResolverの定義
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

設定は、PDFファイルをレンダリングする場合と同様である。詳しくは、\ :ref:`viewresolver-label`\ を参照されたい。

コントローラでのViewの指定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| BeanNameViewResolverにより、コントローラで"textFileDownloadView"を返却することで、
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
       | Springのコンテキストで管理された、TextFileDownloadViewクラスが実行される。
\
    .. tip::

        前述してきたように、SpringはModelの情報をいろいろなViewにレンダリングすることができる。
        Springでは、Jasper Reportsのようなレンダリングエンジンをサポートし、さまざまなViewを返却することも可能である。
        詳細は、Spring の公式ドキュメント\ `Spring reference <http://static.springsource.org/spring/docs/3.2.18.RELEASE/spring-framework-reference/html/view.html#view-jasper-reports>`_\ を参照されたい。

.. raw:: latex

   \newpage

