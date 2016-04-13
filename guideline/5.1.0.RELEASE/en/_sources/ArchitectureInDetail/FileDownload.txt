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
| \ ``org.springframework.web.servlet.view.document.AbstractXlsxView``\  class of Spring is used as a subclass
| when rendering Excel files using model information.
|
| For file formats other than those specified above, various types of View implementations are provided in Spring.
| For technical details on View, refer to \ `Spring Reference View technologies <http://docs.spring.io/spring/docs/4.2.4.RELEASE/spring-framework-reference/html/view.html>`_\ .

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
     - | Inherit \ ``AbstractPdfView``\ .
   * - | (3)
     - | Execute \ ``buildPdfDocument``\  method.

| \ ``AbstractPdfView``\  uses \ `iText <http://itextpdf.com/>`_\  for PDF rendering.
| Therefore, it is necessary to add itext definition to pom.xml of Maven.

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
        Spring IO Platform defines itext version.

.. _viewresolver-label:

Definition of ViewResolver
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\ ``org.springframework.web.servlet.view.BeanNameViewResolver``\  is a class,
that selects View to be executed using bean name stored in Spring context.

When using \ ``BeanNameViewResolver``\ , it is recommended to define such that \ ``BeanNameViewResolver``\  is executed before

* \ ``ViewResolver``\  for JSP (\ ``InternalResourceViewResolver``\ )
* \ ``ViewResolver``\  for Tiles (\ ``TilesViewResolver``\ )

which are generally used.

.. note::

    Spring Framework provides various types of \ ``ViewResolver``\  and it allows chaining of multiple \ ``ViewResolver``\ .
    Therefore, some unintended View may get selected under certain conditions.

    It is possible to avoid such a situation by setting appropriate priority order in \ ``ViewResolver``\ .
    Method to set priority order differs depending on definition method of \ ``ViewResolver``\ .

    * When defining \ ``ViewResolver``\  using \ ``<mvc:view-resolvers>``\  element added from Spring Framework 4.1, definition order of  \ ``ViewResolver``\  specified in child element will be the priority order. (executed sequentially from top)

    * When specifying \ ``ViewResolver``\ using \ ``<bean>``\ element in a conventional way, set priority order in \ ``order``\ property. (It is executed starting from smallest setting value).

|

**bean definition file**

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

   * - Sr. No.
     - Description
   * - | (1)
     - | Define \ ``BeanNameViewResolver``\  using \ ``<mvc:bean-name>``\  element added from Spring Framework 4.1.
   * - | (2)
     - | Define \ ``<mvc:bean-name>``\ element right at the top so that it has a higher priority than the generally used \ ``ViewResolver``\ (\ ``ViewResolver``\ for JSP).


.. tip::

    \ ``<mvc:view-resolvers>``\  element is an XML element added from Spring Framework 4.1.
    If \ ``<mvc:view-resolvers>``\  element is used, it is possible to define \ ``ViewResolver`` \  in a simple way.

    Example of definition when \ ``<bean>``\  element used in a conventional way is given below.


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

    In \ ``order``\ property, specify a value that is lesser than \ ``InternalResourceViewResolver``\  to ensure that it gets a high priority.

|

Specifying View in controller
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| With the help of \ ``BeanNameViewResolver``\ , by returning "samplePDFView" in Controller,
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
       | \ ``SamplePdfView``\  class stored in Spring context is executed.

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
| inherits \ ``org.springframework.web.servlet.view.document.AbstractXlsxView``\  provided by Spring.
| The procedure to download an EXCEL file using controller is explained below.

Implementation of Custom View
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**Implementation of class that inherits AbstractXlsxView**

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

   * - Sr. No.
     - Description
   * - | (1)
     - | In this example, this class comes under the scope of component scanning by using \ ``@Component``\  annotation.
       | It will also come under the scope of \ ``org.springframework.web.servlet.view.BeanNameViewResolver``\  which is described earlier.
   * - | (2)
     - | Inherit \ ``AbstractXlsxView``\ .
   * - | (3)
     - | Execute \ ``buildExcelDocument``\  method.

| \ ``AbstractXlsxView``\ uses \ `Apache POI <http://poi.apache.org/>`_\  to render EXCEL file.
| Therefore, it is necessary to add POI definition to the pom.xml file of Maven.

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
        Since stax-api on which poi-ooxml is dependent, is provided as a standard from SE, the library is not required. Also, since a conflict is likely in the library, \ ``<exclusions>``\  element should be added and the relevant library should not be added in the application.

\
    .. note::
        <version> is omitted in the configuration example since poi-ooxml version uses details defined in Spring IO Platform.

        Also, \ ``AbstractExcelView``\  uses @Deprecated annotation from Spring Framework 4.2. Hence, it is recommended to use \ ``AbstractXlsxView``\ in the same way even if you want to use a xls file.
        For details, refer \ `AbstractExcelView - JavaDoc <https://docs.spring.io/spring/docs/4.2.4.RELEASE/javadoc-api/org/springframework/web/servlet/view/document/AbstractExcelView.html>`_\ .
          

Definition of ViewResolver
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Settings are same as that for PDF file rendering. For details, refer to \ :ref:`viewresolver-label`\ .

Specifying View in controller
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| With the help of \ ``BeanNameViewResolver``\ , by returning "sampleExcelView" in Controller, 
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
       | \ ``SampleExcelView``\  class stored in Spring context is executed.

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
| Following steps should be implemented in \ ``AbstractFileDownloadView``\  to render files in other format.

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
     - | Inherit \ ``AbstractFileDownloadView``\ .
   * - | (3)
     - | Execute \ ``getInputStream``\  method.
       | \ ``InputStream``\  to be downloaded should be returned.
   * - | (4)
     - | Execute \ ``addResponseHeader method``\ .
       | Set Content-Disposition or ContentType as per the file to be downloaded.

Definition of ViewResolver
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Settings are same as that of PDF file rendering. For details, refer to \ :ref:`viewresolver-label`\ .

Specifying View in controller
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| With the help of \ ``BeanNameViewResolver``\ , by returning "textFileDownloadView" in Controller, 
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
       | \ ``TextFileDownloadView``\  class stored in Spring context is executed.

\

    .. tip::

            As described above, Model information can be rendered in various types of Views using Spring.
            Spring supports rendering engine such as Jasper Reports and returns various types of views.
            For details, refer to the official Spring website  \ `Spring reference <http://docs.spring.io/spring/docs/4.2.4.RELEASE/spring-framework-reference/html/view.html#view-jasper-reports>`_\ .

.. raw:: latex

   \newpage

