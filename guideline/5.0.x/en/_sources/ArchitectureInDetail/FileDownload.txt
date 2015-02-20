File Download
================================================================================

.. only:: html

 .. contents:: Table of Contents
    :depth: 3
    :local:
    
Overview
--------------------------------------------------------------------------------

| This chapter explains the functionality to download a file from server to client, using Spring.
| It is recommended to use Spring MVC View for rendering the files.
\
    .. note::
        It is not recommended to include file rendering logic in controller class.

        This is because, it deviates from the role of a controller.
        Moreover, a View can be easily changed when it is isolated from the controller.

Overview of File Download process is given below.

#. DispatchServlet sends file download request to the controller.
#. Controller fetches file display information.
#. Controller selects View.
#. File rendering is performed in View.


| In order to perform file rendering in a Spring based Web application;
| this guideline recommends implementation of custom view. 
| To implement custom view, ``org.springframework.web.servlet.View`` interface 
| is provided in Spring framework.
|
| **For PDF files**
| \ ``org.springframework.web.servlet.view.document.AbstractPdfView``\  class of Spring is used as a subclass
| when rendering PDF files using model information.
|
| **For Excel files**
| \ ``org.springframework.web.servlet.view.document.AbstractExcelView``\  class of Spring is used as a subclass
| when rendering Excel files using model information.
|
| For file formats other than those specified above, various types of View implementations are provided in Spring.
| For technical details on View, refer to \ `Spring Reference View technologies <http://docs.spring.io/spring/docs/4.1.4.RELEASE/spring-framework-reference/html/view.html>`_\ .

| \ ``org.terasoluna.gfw.web.download.AbstractFileDownloadView``\  provided by common library is the
| abstract class to download arbitrary files.
| Define this class as a subclass to render files in formats other than PDF or Excel.

|

How to use
--------------------------------------------------------------------------------

Downloading PDF files
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| For rendering PDF files, it is necessary to create a class that
| inherits \ ``org.springframework.web.servlet.view.document.AbstractPdfView``\  provided by Spring.
| The procedure to download a PDF file using controller is explained below.

Implementation of Custom View
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**Implementation of class that inherits AbstractPdfView**

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

   * - Sr. No.
     - Description
   * - | (1)
     - | In this example, this class comes under the scope of component scanning by using \ ``@Component``\  annotation.
       | It will also come under the scope of \ ``org.springframework.web.servlet.view.BeanNameViewResolver``\  which is described later.
   * - | (2)
     - | Inherit AbstractPdfView.
   * - | (3)
     - | Execute buildPdfDocument method.

| AbstractPdfView uses \ `iText <http://itextpdf.com/>`_\  for PDF rendering.
| Therefore, it is necessary to add itext definition to pom.xml of Maven.

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
        Spring 3.2 does not support itext version 5.

.. _viewresolver-label:

Definition of ViewResolver
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| \ ``org.springframework.web.servlet.view.BeanNameViewResolver``\  is the class stored in Spring context. It selects the View to be executed on the basis of bean name.
| Normally \ ``InternalResourceViewResolver``\  is used; however while using BeanNameViewResolver, it should be called before \ ``InternalResourceViewResolver``\  by setting \ ``order``\  property.
\
    .. note::

        Spring provides various types of View Resolvers and it allows chaining of multiple Resolvers; hence some unintended View may get selected under certain conditions.
        To avoid such a situation, priority can be set by specifying order property.
        Lower the specified value of order property, earlier it is executed.

**bean definition file**

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

   * - Sr. No.
     - Description
   * - | (1)
     - | Define BeanNameViewResolver.
   * - | (2)
     - | Set 0 in order property. Priority should be higher than that of InternalResourceViewResolver.

|

Specifying View in controller
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| With the help of BeanNameViewResolver, by returning "samplePDFView" in Controller,
| a view named "samplePDFView" gets used from the BeanIDs stored in Spring Context.

**Java source code**

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

   * - Sr. No.
     - Description
   * - | (1)
     - | With "samplePdfView" as the return value of method,
       | SamplePdfView class stored in Spring context is executed.

| Following PDF file can be opened after executing the above procedure.

.. figure:: ./images/file-download-pdf.png
   :alt: FILEDOWNLOAD PDF
   :width: 60%
   :align: center

   **Picture - FileDownload PDF**

|

Downloading Excel files
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| For rendering EXCEL files, it is necessary to create a class that
| inherits \ ``org.springframework.web.servlet.view.document.AbstractExcelView``\  provided by Spring.
| The procedure to download an EXCEL file using controller is explained below.

Implementation of Custom View
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**Implementation of class that inherits AbstractExcelView**

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

   * - Sr. No.
     - Description
   * - | (1)
     - | In this example, this class comes under the scope of component scanning by using \ ``@Component``\  annotation.
       | It will also come under the scope of \ ``org.springframework.web.servlet.view.BeanNameViewResolver``\  which is described earlier.
   * - | (2)
     - | Inherit AbstractExcelView.
   * - | (3)
     - | Execute buildExcelDocument method.

| AbstractExcelView uses \ `Apache POI <http://poi.apache.org/>`_\  to render EXCEL file.
| Therefore, it is necessary to add POI definition to the pom.xml file of Maven.

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
        

Definition of ViewResolver
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Settings are same as that for PDF file rendering. For details, refer to \ :ref:`viewresolver-label`\ .

Specifying View in controller
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| With the help of BeanNameViewResolver, by returning "sampleExcelView" in Controller, 
| a view named "sampleExcelView" gets used from the BeanIDs stored in Spring Context.

**Java source**

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

   * - Sr. No.
     - Description
   * - | (1)
     - | With "sampleExcelView"as the return value of method,
       | SampleExcelView class stored in Spring context is executed.

| EXCEL file can be opened as shown below after executing the above procedures.

.. figure:: ./images/file-download-excel.png
   :alt: FILEDOWNLOAD EXCEL
   :width: 60%
   :align: center

   **Picture - FileDownload EXCEL**

Downloading arbitrary files
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| To download files in formats other than PDF or EXCEL,
| class that inherits \ ``org.terasoluna.gfw.web.download.AbstractFileDownloadView``\  provided by common library can be implemented.
| Following steps should be implemented in AbstractFileDownloadView to render files in other format.

1. Fetch InputStream in order to write to the response body.
2. Set information in HTTP header.

| How to implement file download using controller is explained below.

Implementation of Custom View
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| The example of text file download is given below.

**Implementation of class that inherits AbstractFileDownloadView**

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

   * - Sr. No.
     - Description
   * - | (1)
     - | In this example, this class comes under the scope of component scanning by using \ ``@Component``\  annotation.
       | It will also come under the scope of \ ``org.springframework.web.servlet.view.BeanNameViewResolver``\  which is described earlier.
   * - | (2)
     - | Inherit AbstractFileDownloadView.
   * - | (3)
     - | Execute getInputStream method.
       | InputStream to be downloaded should be returned.
   * - | (4)
     - | Execute addResponseHeader method.
       | Set Content-Disposition or ContentType as per the file to be downloaded.

Definition of ViewResolver
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Setting is same as that of PDF file rendering. For details, refer to \ :ref:`viewresolver-label`\ .

Specifying View in controller
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| With the help of BeanNameViewResolver, by returning "textFileDownloadView" in Controller, 
| a view named "textFileDownloadView" gets used from the BeanIDs stored in Spring Context. 

**Java source**

.. code-block:: java

        @RequestMapping(value = "download", method = RequestMethod.GET)
        public String download() {
            return "textFileDownloadView"; // (1)
        }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | With "textFileDownloadView"as the return value of method, 
       | TextFileDownloadView class stored in Spring context is executed.
\
    .. tip::

            As described above, Model information can be rendered in various types of Views using Spring.
            Spring supports rendering engine such as Jasper Reports and returns various types of views.
            For details, refer to the official Spring website  \ `Spring reference <http://docs.spring.io/spring/docs/4.1.4.RELEASE/spring-framework-reference/html/view.html#view-jasper-reports>`_\ .

.. raw:: latex

   \newpage

