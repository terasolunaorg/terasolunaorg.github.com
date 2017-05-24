File Upload
================================================================================

.. only:: html

 .. contents:: Table of contents
    :depth: 3
    :local:

.. _FileUploadOverview:

Overview
--------------------------------------------------------------------------------

| This chapter explains how to upload files.

| Files are uploaded using the File Upload functionality supported by Servlet 3.0 and classes provided by Spring Web.

 .. note::

    In this chapter, File Upload functionality supported by Servlet 3.0 is used; hence, Servlet version 3.0 or above is a prerequisite here.

 .. note::

    File Upload functionality of Servlet 3.0 may likely result into garbling of multi byte characters of file names or request parameters on some application server.

    Application servers wherein occurrence of issues is confirmed at the version 5.3.0.RELEASE are as shown below.

    * WebLogic 12.1.3
    * JBoss EAP 7.0
    * JBoss EAP 6.4.0.GA

    Among these, the issues can be avoided by adding application server specific settings in JBoss EAP 7.0.
    For details, refer \ `Precautions for using JBoss EAP 7.0 <https://github.com/terasolunaorg/terasoluna-gfw/wiki/JBoss7_en>`_\.

    When the application server wherein other problems occur are used, the issues can be avoided by using Commons FileUpload.
    For setup methods for using Commons FileUpload, refer ":ref:`file-upload_usage_commons_fileupload`".

 .. warning::
 
    If implementation of file upload of an application server to be used depends on implementation of Apache Commons FileUpload, security vulnerabilities reported in \ `CVE-2014-0050 <http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0050>`_\  and `CVE-2016-3092 <https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-3092>`_\  may occur.
    Hence ensure that there are no such vulnerabilities in the application server to be used.
    
    In case of using Tomcat, it is necessary to use version 7.0.52 or above for series 7.0, and version 8.0.3 or above for series 8.0.

Basic flow of upload process
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Basic flow of uploading files using File Upload functionality supported by Servlet 3.0, and classes of Spring Web, is as shown below.

 .. figure:: ./images/file-upload-overview_basicflow.png
   :alt: Screen image of single file upload.
   :width: 100%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90
   :class: longtable

   * - Sr. No.
     - Description
   * - | (1)
     - | Select and upload the target files.
   * - | (2)
     - | Servlet container receives \ ``multipart/form-data``\  request and calls \ ``org.springframework.web.multipart.support.MultipartFilter``\ .
   * - | (3)
     - | \ ``MultipartFilter``\  calls the method of \ ``org.springframework.web.multipart.support.StandardServletMultipartResolver``\  to enable File Upload functionality of Servlet 3.0 in Spring MVC.
       | \ ``StandardServletMultipartResolver``\  generates ``org.springframework.web.multipart.MultipartFile`` object that wraps the API (``javax.servlet.http.Part``) introduced through Servlet 3.0.
   * - | (4)
     - | Apply a filter chain in \ ``DispatcherServlet``\  from \ ``MultipartFilter``\ .
   * - | (5)
     - | \ ``DispatcherServlet``\  calls handler method of Controller.
       | \ ``MultipartFile``\  object generated in (3) is bound to Controller argument or form object.
   * - | (6)
     - | Controller calls a method of \ ``MultipartFile``\  object and fetch contents of uploaded file and meta information (file name etc.).
   * - | (7)
     - | \ ``MultipartFile``\  calls a method of \ ``Part``\  object introduced from Servlet 3.0, fetches contents of uploaded file and meta information (file name etc.) and returns to Controller.
   * - | (8)
     - | Controller calls the Service method and executes upload process.
       | It passes the contents and meta information (file name etc.) of the file retrieved from \ ``MultipartFile``\  object as an argument of Service method.
   * - | (9)
     - | Service stores contents of uploaded file and meta information (file name etc.) in the file or database.
   * - | (10)
     - | \ ``MultipartFilter``\  calls \ ``StandardServletMultipartResolver``\  and deletes temporary file used by file upload function of Servlet 3.0.
   * - | (11)
     - | \ ``StandardServletMultipartResolver``\  calls a method of \ ``Part``\  object introducted from Servlet 3.0 and deletes the temporary file stored in the disc.

 .. raw:: latex

    \newpage

 .. note::

    Controller performs the process for \ ``MultipartFile``\  object of Spring Web; hence implementation which is dependent on the File Upload API provided by Servlet 3.0 can be excluded.


About classes provided by Spring Web
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Classes provided by Spring Web for uploading a file are as follows:

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.40\linewidth}|p{0.50\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 40 50

   * - | Sr. No.
     - | Class name
     - | Description
   * - 1.
     - | org.springframework.web.multipart.
       | MultipartFile
     - | Interface indicating uploaded file.
       | It plays a role in abstraction of file objects handled by the File Upload functionality to be used.
   * - 2.
     - | org.springframework.web.multipart.support.
       | StandardMultipartHttpServletRequest$
       | StandardMultipartFile
     - | \ ``MultipartFile``\  class of File Upload functionality introduced through Servlet 3.0. 
       | Process is delegated to the \ ``Part``\  object introduced through Servlet 3.0.
   * - 3.
     - | org.springframework.web.multipart.
       | MultipartResolver
     - | Interface that resolves the analysis method of \ ``multipart/form-data``\  request.
       | It plays a role in generating \ ``MultipartFile``\  object corresponding to implementation of File Upload functionality.
   * - 4.
     - | org.springframework.web.multipart.support.
       | StandardServletMultipartResolver
     - | \ ``MultipartResolver``\  class for File Upload functionality introduced through Servlet 3.0.
   * - 5.
     - | org.springframework.web.multipart.support.
       | MultipartFilter
     - | A class which generates MultipartFile by calling a class which implements MultipartResolver from DI container, at the time of multipart/form-data request.
       | If this class is not used, a request parameter cannot be fetched in Servlet Filter process when maximum size allowed in file upload exceeds the limit.
       | Therefore, it is recommended to use MultipartFilter in this guideline.

 .. tip::

    In this guideline, it is a prerequisite to use File Upload functionality implemented from Servlet 3.0. However, Spring Web also provides an \ `implementation class for "Apache Commons FileUpload" <http://docs.spring.io/spring/docs/4.3.5.RELEASE/spring-framework-reference/html/mvc.html#mvc-multipart-resolver-commons>`_\ .
    The difference in implementation of upload processes is absorbed by \ ``MultipartResolver``\  and \ ``MultipartFile``\  objects; hence it does not affect Controller implementation.

|

How to use
--------------------------------------------------------------------------------

.. _file-upload_how_to_usr_application_settings:

Application settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Settings to enable Servlet 3.0 upload functionality 
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Perform the following settings to enable upload functionality of Servlet 3.0.

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

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify the XSD file of Servlet 3.0 or above in \ ``xsi:schemaLocation``\  attribute of \ ``<web-app>``\  element.
   * - | (2)
     - | Specify version  ``3.0`` or above in the \ ``version``\  attribute of \ ``<web-app>``\  element.
   * - | (3)
     - | Add \ ``<multipart-config>``\  element to \ ``<servlet>``\  element of the Servlet using the File Upload functionality.
   * - | (4)
     - | Specify the maximum file size of 1 upload-permissible file in bytes.
       | If not specified, -1 (no limit) is set by default.
       | If it exceeds the specified value, exception, \ ``org.springframework.web.multipart.MultipartException``\  occurs.
       |
       | In the above example, a file size of 5MB is specified.
   * - | (5)
     - | Specify the maximum Content-Length value of \ ``multipart/form-data``\  request. 
       | If not specified, -1 (no limit) is set by default.
       | If it exceeds the specified value, exception \ ``org.springframework.web.multipart.MultipartException``\  occurs.
       |
       | Value to be set in this parameter should be calculated by the following formula.
       |
       | **("maximum file size of 1 file to be uploaded"  * "Number of files allowed to be uploaded simultaneously" ) + "Data size of other form fields" +  "Meta information size of multipart/form-data request"**
       |
       | In the above example, parameter value of 26MB is specified.
       | Its breakup is, 25MB (5MB * 5 files) and 1MB (number of bytes of meta information + number of bytes of form fields).
   * - | (6)
     - | Specify the threshold value (number of bytes for 1 file) if the contents of uploaded file are to be saved as a temporary file.
       | If this parameter is not specified explicitly, there are application servers wherein values specified for elements ``<max-file-size>`` and ``<max-request-size>`` are considered invalid; hence default value (0) is being specified explicitly.

 .. raw:: latex

    \newpage

 .. warning::

    In order to increase the resistance against Dos attack, \ ``max-file-size``\  and \ ``max-request-size``\  should be specified without fail.

    For Dos attack, refer to \ :ref:`file-upload_security_related_warning_points_dos`\ .


 .. note::

    Uploaded file is by default output as temporary file. However, its output can be controlled using the configuration value of \ ``<file-size-threshold>``\   element, which is the child element of \ ``<multipart-config>``\ .

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

       * - Sr. No.
         - Description
       * - | (7)
         - | Specify the threshold file size (number of bytes of 1 file) if contents of uploaded file are to be saved as a temporary file.
           | If not specified, 0 is set.
           | If uploaded file size exceeds the specified value, 
           | it is output as a temporary file to the disk and deleted when the request is completed.
           |
           | In the above example, 32KB is specified.

     .. warning::

        This parameter shows a trade-off relationship as indicated by the following points. Hence,  \ **configuration value corresponding to system characteristics should be specified.**\ .

        * Increasing the configuration value improves processing performance as, processing gets completed within available memory. However, there is a high possibility that \ ``OutOfMemoryError``\  may occur due to Dos attack.
        * If configuration value is reduced, memory utilization can be controlled to the minimum, thereby avoiding the possibility of \ ``OutOfMemoryError``\  due to Dos attack etc.
          However, there is a high possibility of performance degradation since the frequency of disk IO generation is high.


   To change output directory of temporary files, specify directory path in \ ``<location>``\  element, which is the child element of \ ``<multipart-config>``\ .

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

       * - Sr. No.
         - Description
       * - | (8)
         - | Specify the directory path for outputting temporary files.
           | When omitted, they are output to the directory that stores temporary files of application server.
           |
           | In the above example, \ ``/tmp``\  is specified.

     .. warning::

        The directory specified in \ ``<location>``\  element is the one used by the application server (servlet container) and **cannot be accessed from application.**

        When the files uploaded as application are to be saved as temporary files, they should be output to a directory other than the directory specified in \ ``<location>``\  element.

.. _file-upload_setting_servlet_filter:

Servlet Filter settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
The operation when the maximum size allowed in file upload exceeds the limit at the time of multipart/form-data request, varies depending on the application server. \ ``MultipartException``\  generated when maximum size exceeds the limit depending on the application server is likely to be not detected and exception handling settings described later will be invalid.

| Since this operation can be evaded by setting \ ``MiltipartFilter``\ , \ ``MiltipartFilter``\  setting is described as a prerequisite in this guideline.
| Setting example is given below.

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

   * - Sr. No.
     - Description
   * - | (1)
     - | Define \ ``MultipartFilter``\  as the Servlet Filter.
   * - | (2)
     - | Specify the URL pattern for applying \ ``MultipartFilter``\ .
     

 .. warning:: **Precautions while using Spring Security**

    When security countermeasures are to be carried out by using Spring Security, they should be defined prior to \ ``springSecurityFilterChain``\. 
    Further, when request parameters are accessed by a project-specific Servlet Filter, MultipartFilter should be defined before that Servlet Filter.

    However, when defined before \ ``springSecurityFilterChain``\ , unauthenticated or unauthorized users may be allowed to upload the file (create temporary file).
    Although a method to avoid this operation has been given in \ `Spring Security Reference -Cross Site Request Forgery (CSRF)- <http://docs.spring.io/spring-security/site/docs/4.1.4.RELEASE/reference/htmlsingle/#csrf-include-csrf-token-in-action>`_\ , it is not recommended to be applied in this guideline since it poses a security risk. 

 .. warning:: **Precautions when maximum size limit for file upload is exceeded**

   When allowable size limit for file upload has been exceeded, an 'Over the size limit" error may get detected before fetching a CSRF token in some of the application servers like WebLogic and CSRF token check is not performed.

 .. note:: **Default calling of MultipartResolver**
    
    If \ ``MultipartFilter``\  is used,
    \ ``org.springframework.web.multipart.support.StandardServletMultipartResolver``\  is called by default.
    \ ``StandardServletMultipartResolver``\  should be able to generates uploaded file as \ ``org.springframework.web.multipart.MultipartFile``\  and receive as property of Controller argument and form object.


Settings for exception handling
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Add the exception handling definition of \ ``MultipartException``\  which occurs when a request for file or multipart with non-permissible size is sent.

| \ ``MultipartException``\  is an exception caused due to file size specified by the client; hence it is recommended to handle it as a client error (HTTP response code=4xx).
| **If exception handling is not added for individual exception, it is eventually treated as system error; hence make sure that it is defined without fail.**

| Settings for handling \ ``MultipartException``\  differ depending upon whether  \ ``MultipartFilter``\  is used or not.
| In case of using \ ``MultipartFilter``\,  exception handling is carried out by using the \ ``<error-page>``\  functionality of servlet container.
| Example of settings is shown below.

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

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify \ ``MultipartException``\  as the exception class for handling.
   * - | (2)
     - | Specify the file displayed when \ ``MultipartException``\  occurs.
       |
       | In the above example, \ ``"/WEB-INF/views/common/error/fileUploadError.jsp"``\  is specified.

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

   * - Sr. No.
     - Description
   * - | (3)
     - | Set HTTP status code by calling the API of \ ``HttpServletResponse``\ .
       |
       | In the above request, \ ``"400"``\  (Bad Request) is set.
       | When not set explicitly, the HTTP status code is considered as \ ``"500"``\  (Internal Server Error).

|

| When not using \ ``MultipartFilter``\ , carry out exception handling by using \ ``SystemExceptionResolver``\ .
| Example of settings is shown below.

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

   * - Sr. No.
     - Description
   * - | (4)
     - | In \ ``exceptionMappings``\  of \ ``SystemExceptionResolver``\ , add the definition for View (JSP) which is displayed when \ ``MultipartException``\  occurs.
       | 
       | In the above example, \ ``"common/error/fileUploadError"``\  is specified.
   * - | (5)
     - | Add the definition of HTTP status code which is received as response when ``MultipartException`` occurs.
       |
       | In the above example, \ ``"400"``\  (Bad Request) is specified.
       | By specifying client error (HTTP response code = 4xx),
       | the level of log which is output by the class (``HandlerExceptionResolverLoggingInterceptor``) provided by the exception handling functionality of common library is  \ ``WARN``\  and not \ ``ERROR``\ .

|

| Add exception code settings when setting an exception code for \ ``MultipartException``\ .
| Exception code is output to the log which is output using log output functionality of common library.
| Exception code can also be referred from View (JSP).
| For referring to exception code from View (JSP), refer to \ :ref:`exception-handling-how-to-use-codingpoint-jsp-exceptioncode-label`\ .

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

   * - Sr. No.
     - Description
   * - | (6)
     - | In \ ``exceptionMappings``\  of \ ``SimpleMappingExceptionCodeResolver``\ , add the exception code to be applied when \ ``MultipartException``\  occurs.
       |
       | In the above example, \ ``"e.xx.fw.6001"``\  is specified.
       | When it is not defined individually, exception code specified in \ ``defaultExceptionCode``\  is applied.


Uploading a single file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The explanation about uploading a single file is given below.

 .. figure:: ./images/file-upload-how_to_use_single.png
   :alt: Screen image of single file upload.
   :width: 100%

| There are 2 methods to upload a single file. One is by binding \ ``org.springframework.web.multipart.MultipartFile``\  object to the form object and the other is by receiving it directly as Controller argument. However, this guideline recommends the first method wherein it is received after it is bound with the form object.
| The reason for this being, single field check of the uploaded file can be performed using Bean Validation.

How to receive a single file by binding it to form object is explained below.


Implementing form
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

   * - Sr. No.
     - Description
   * - | (1)
     - | Define properties of \ ``org.springframework.web.multipart.MultipartFile``\  in form object.


Implementing JSP
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

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify \ ``"multipart/form-data"``\  in the enctype attribute of \ ``<form:form>``\  element.
   * - | (2)
     - | Specify attribute name of form object in the modelAttribute of \ ``<form:form>``\  element.
       | In the above example, \ ``"fileUploadForm"``\  is specified.
   * - | (3)
     - | Specify \ ``"file"``\  in type attribute of \ ``<form:input>``\  element and specify \ ``MultipartFile``\  property name in path attribute.
       | In the above example, the uploaded file is stored in \ ``"file"``\  property of \ ``FileUploadForm``\  object.


Implementing Controller
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

   * - Sr. No.
     - Description
   * - | (1)
     - | Method of storing the form object for file upload in \ ``Model``\ .
       | In the above example, the attribute name for storing form object in \ ``Model``\  is \ ``"fileUploadForm"``\ .
   * - | (2)
     - | Handler method for displaying upload screen.
   * - | (3)
     - | Handler method for uploading files.
   * - | (4)
     - | It is checked whether the files for upload are selected.
       | To check if the files are selected, call \ ``MultipartFile#getOriginalFilename``\   method and decide on the basis of whether file name is specified or not.
       | In the above example, input validation error is thrown if the files are not selected.
   * - | (5)
     - | It is checked whether an empty file is selected.
       | To check if the selected file is not empty, call \ ``MultipartFile#isEmpty``\  method to check for presence of contents.
       | In the above example, input validation error is thrown if an empty file is selected.
   * - | (6)
     - | It is checked whether the file size is within allowable range.
       | To check the size of selected file, call \ ``MultipartFile#getSize``\  method and check whether the size is within the allowable range.
       | In the above example, input validation error is thrown if the file size exceeds the allowable range.
   * - | (7)
     - | Implement upload process.
       | The above example does not cover any specific implementation; however process to store the file on a shared disk or database is performed.
   * - | (8)
     - | As per the requirement, the processing result message notifying about successful upload is stored.
   * - | (9)
     - | Once upload is complete, redirect to upload completion screen.

 .. raw:: latex

    \newpage

 .. note:: **Preventing duplicate upload**

    When uploading files, it is recommended to perform transaction token check and screen transition based on PRG pattern.
    With this, upload of same files caused due to double submission can be prevented.

    For more details on how to prevent double submission, refer to \ :doc:`../WebApplicationDetail/DoubleSubmitProtection`\ .

 .. note:: **About MultipartFile**

    Methods to operate the uploaded file are provided in MultipartFile.
    For details on using each method, refer to \ `JavaDoc of MultipartFile class <http://docs.spring.io/spring/docs/4.3.5.RELEASE/javadoc-api/org/springframework/web/multipart/MultipartFile.html>`_\ .

.. _fileupload_validator:

Bean Validation of file upload
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| In the above implementation example, uploaded file is validated as a Controller process. However, here the uploaded file is validated using Bean Validation.
| For validation details, refer to \ :doc:`Validation`\ . 

 .. note::

    It is recommended to use Bean Validation since this makes maintenance of Controller processes easier.


Implementing validation to verify that the file is selected
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

   * - Sr. No.
     - Description
   * - | (1)
     - | Create annotation to verify that the file is selected.
   * - | (2)
     - | Create implementation class to verify that the file is selected.


Implementing validation to verify that the file is not empty
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

   * - Sr. No.
     - Description
   * - | (3)
     - | Create annotation to verify that the file is not empty.
   * - | (4)
     - | Create implementation class to verify that the file is not empty.


Implementing validation to verify that file size is within allowable range
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

   * - Sr. No.
     - Description
   * - | (5)
     - | Create annotation to verify that the file size is within allowable range.
   * - | (6)
     - | Create implementation class to verify that the file size is within allowable range.


Implementing form
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

   * - Sr. No.
     - Description
   * - | (7)
     - | Assign annotation to \ ``MultipartFile``\  field for validating uploaded file.


Implementing Controller
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

   * - Sr. No.
     - Description
   * - | (8)
     - | Validation result of uploaded file is stored in \ ``BindingResult``\ .


Uploading multiple files
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
This section explains about simultaneously uploading multiple files.

 .. figure:: ./images/file-upload-how_to_use_multi.png
   :alt: Screen image of multiple file upload.
   :width: 100%

In order to upload multiple files simultaneously, it is necessary to receive \ ``org.springframework.web.multipart.MultipartFile``\  object by binding it to the form object.

The explanation that has already been covered under single file upload has been omitted to avoid duplication. 


Implementing form
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

   * - Sr. No.
     - Description
   * - | (1)
     - | Class that maintains the information of each file (uploaded file itself and related form fields).
       | In the above example, form object that was originally created to explain about single file upload, is re-used.
   * - | (2)
     - | To carry out input validation through Bean Validation for the object maintained in list, assign \ ``@Valid``\  annotation.
   * - | (3)
     - | Define the object that maintains information of each file (uploaded file itself and related form fields) as List property.

 .. note::

   When only files are to be uploaded, \ ``MultipartFile``\  object can also be defined as List property; 
   however, for input validation of uploaded files using Bean Validation, there is better compatibility if the object that maintains information of each file, is defined as List property.


Implementing JSP
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

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify the binding position of the uploaded file in List.
       | Specify the binding position within List in \ ``[]``\ . Start position begins with \ ``0``\ .


Implementing Controller
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

   * - Sr. No.
     - Description
   * - | (1)
     - | Fetch ``MultipartFile`` from the object that maintains information of each file (uploaded file itself and related form fields) and implement upload process.
       | The above example does not cover any specific implementation; however process to store the file on a shared disk or database is performed.


Uploading multiple files using the "multiple" attribute of HTML5
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The method to simultaneously upload multiple files using "multiple" attribute of input tag supported by HTML5, is explained below.

 .. figure:: ./images/file-upload-how_to_use_multi_html5.png
   :alt: Screen image of multiple file upload(html5).
   :width: 100%

The explanation that has already been covered under single file upload and multiple file upload has been omitted.

Implementing form
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
When uploading multiple files simultaneously using "multiple" attribute of HTML5 input tag, it is necessary to receive collection of \ ``org.springframework.web.multipart.MultipartFile``\  object by binding it to form object.

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

   * - Sr. No.
     - Description
   * - | (1)
     - | Form object that maintains the multiple uploaded files.
   * - | (2)
     - | Declare ``MultipartFile`` class as list.
       | In the above example, the annotation to verify that the file is not empty, is specified as input validation.
       | Principally, a file size check or other mandatory checks are also required; however, they have been omitted in the above example.

Implementing Validator
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
When carrying out input validation for multiple  ``MultipartFile`` objects stored in collection, it is necessary to implement Validator for Collection.

The section explains about creating Validator for Collection using the Validator created for single file.

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

   * - Sr. No.
     - Description
   * - | (1)
     - | Class for performing implementation to verify that none of the files is empty.
       | Specify ``Collection<MultipartFile>`` as the type of value to be verified.
   * - | (2)
     - | In order to delegate the actual process to a Validator for single file, create an instance for that Validator.
   * - | (3)
     - | Initialize the Validator.
       | In the above example, Validator for single file that implements the actual process is initialized.
   * - | (4)
     - | Verify that none of the file is empty.
       | In the above example, each file is verified by calling the method of Validator for single file.

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

   * - Sr. No.
     - Description
   * - | (5)
     - | Add the Validator class that carries out checks with respect to multiple files, to the annotation used for verification.
       | Specify the class created in step (1) in the "validatedBy" attribute of  ``@Constraint`` annotation.
       | With this, the class created in step (1) is executed when validating the property with ``@UploadFileNotEmpty`` annotation.


Implementing JSP
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

   * - Sr. No.
     - Description
   * - | (1)
     - | In "path" attribute, specify "multiple" attribute by indicating property name of form object.
       | By specifying "multiple" attribute, multiple files can be selected and uploaded using browser supporting HTML5.


Implementing Controller
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

   * - Sr. No.
     - Description
   * - | (1)
     - | Implement upload process by fetching the list which stores ``MultipartFile`` objects from form object.
       | The above example does not cover any specific implementation; however process to store the file on a shared disk or database is performed.

Temporary upload
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Temporary upload is required when a file is to be uploaded midway through screen transitions like upload result confirmation screen etc.

 .. note::

    Contents of file stored  in \ ``MultipartFile``\  object may be deleted once the upload request is completed.
    Therefore, when the file contents are to be handled across requests, these contents and meta information (file name etc.) maintained  in \ ``MultipartFile``\  object need to be saved in a file or form.

    The contents of file stored in \ ``MultipartFile``\  object are deleted when step (3) of the following processing flow is completed.

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

   * - Sr. No.
     - Description
   * - | (1)
     - | On Input Screen, select the file to be uploaded and send a request for displaying Confirm Screen.
   * - | (2)
     - | Controller temporarily saves contents of uploaded file in the temporary directory for application.
   * - | (3)
     - | Controller returns View name of Confirm Screen and then displays the Confirm Screen.
   * - | (4)
     - | On Confirm screen, send a request for executing the process.
   * - | (5)
     - | Controller calls Service method and executes process.
   * - | (6)
     - | Service moves the temporary file saved in temporary directory to this directory or database.
   * - | (7)
     - | Controller returns the View name which is required to display Complete Screen and then displays the Complete Screen.

 .. raw:: latex

    \newpage

 .. note::

    Temporary upload process is the responsibility of application layer; hence it is executed by Controller or Helper class.


Implementing Controller
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Example for temporarily saving the uploaded file in a temporary directory, is shown below.

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

   * - Sr. No.
     - Description
   * - | (1)
     - | Create a method for executing temporary upload in Helper class.
       | When there are multiple processes that perform file upload, it is recommended to have a common temporary upload process by creating a common Helper method.
   * - | (2)
     - | Save the uploaded file as a temporary file.
       | In the above example, contents of uploaded file are saved to a file by calling copyInputStreamToFile method of \ ``org.apache.commons.io.FileUtils``\  class.

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

   * - Sr. No.
     - Description
   * - | (3)
     - | Call the Helper method to temporarily save the uploaded file.
       | In the above example, ID by which the temporarily saved file is identified, is returned as the return value of Helper method.
   * - | (4)
     - | Save the meta information of uploaded file (ID by which the file is identified, file name etc.) in form object.
       | In the above example, name of the uploaded file and ID by which the temporarily saved file is identified, are stored in form object.

 .. note::

    Directory of temporary directories should be fetched from external properties as it may differ with the environment in which the application is deployed.
    For details on external properties, refer to \ :doc:`../GeneralFuncDetail/PropertyManagement`\ .

 .. warning::
 
    In the above example, it is a file saved temporarily on the local disk of application server. However, when the application server is clustered,
    it needs to be saved in the database or on a shared disk. As a result, it is necessary to design a storage destination by considering even the non-functional requirements.
    
    Transaction management is necessary in case of saving the file to the database. As a result, the process to save it to the database will be delegated to Service method.

|

How to extend
--------------------------------------------------------------------------------

.. _file-upload_how_to_use_housekeeping:

Housekeeping of unnecessary files at the time of temporary upload
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| When uploading files using the temporary upload method, there is a possibility of unnecessary files piling up in temporary directory.
| The cases are as follows:

* When there is interruption in screen operations after temporary upload
* When system error occurs during the screen operations after temporary upload
* When server stops during the screen operations after temporary upload
  etc ...

 .. warning::

    A mechanism should be provided to delete unnecessary files as the disk may run out of space if such files are left to pile up.

This guideline explains about deleting unnecessary files using the "Task Scheduler" functionality provided by Spring Framework.
For details on "Task Scheduler", refer to the \ `official website "Task Execution and Scheduling" <http://docs.spring.io/spring/docs/4.3.5.RELEASE/spring-framework-reference/html/scheduling.html>`_\ .

 .. note::

    Although this guideline explains about how to use "Task Scheduler" functionality provided by Spring Framework; its usage is not mandatory.
    In an actual project, the infrastructure team may provide batch application (Shell application) to delete unnecessary files.
    In such cases, it is recommended to delete unnecessary files using the batch application created by infrastructure team.


Implementing component class to delete unnecessary files
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Implement a component class to delete unnecessary files.

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

   * - Sr. No.
     - Description
   * - | (1)
     - | Create component class to delete unnecessary files.
   * - | (2)
     - | Implement the method to delete unnecessary files.
       | In the above example, the files that have not been updated for a certain period of time from the last update, are treated as unnecessary files and are deleted.

 .. note::

    Directory path in which files to be deleted are stored or the time criteria for deletion etc. may differ depending upon the environment in which application is to be deployed. Hence they should be fetched from external properties.
    For details on external properties, refer to \ :doc:`../GeneralFuncDetail/PropertyManagement`\ .


Scheduling settings of the process for deleting unnecessary files
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Carry out bean registration and task schedule settings for the POJO class that deletes unnecessary files.

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

   * - Sr. No.
     - Description
   * - | (3)
     - | POJO class that deletes unnecessary files should be registered in bean.
       | In the above example, it is registered with ``"uploadTemporaryFileCleaner"`` ID.
   * - | (4)
     - | Register the bean for task scheduler that executes the process to delete unnecessary files.
       | In the above example, as pool-size attribute is omitted, this task scheduler executes the task in a single thread .
       | When multiple tasks need to be executed simultaneously, some number should be specified in ``pool-size`` attribute.
   * - | (5)
     - | Add the task to the task scheduler that deletes unnecessary files.
       | In the above example, task is added to the task scheduler for which bean is registered in step (4).
   * - | (6)
     - | In ``ref`` attribute, specify the bean that executes the process of deleting unnecessary files.
       | In the above example, the bean registered in step (3) is specified.
   * - | (7)
     - | In ``method`` attribute, specify the name of method executing the process of deleting unnecessary files.
       | In the above example, cleanup method of bean registered in step (3) is specified.
   * - | (8)
     - | In ``cron`` attribute, specify execution time of the process to delete unnecessary files.
       | In the above example, cron definition is fetched from external properties.

 .. note::

    Specify the configuration value of ``cron`` attribute in "seconds minutes hour month year day" format.

    Example:

     * ``0 */15 * * * *`` : Executed in 0 minute, 15 minutes, 30 minutes and 45 minutes every hour.
     * ``0 0 * * * *`` : Executed in 0 minute every hour.
     * ``0 0 9-17 * * MON-FRI`` : Executed in 0 minute every hour from 9:00~17:00 on weekdays.

    For details on specified value of cron, refer to \ `CronSequenceGenerator - JavaDoc <http://docs.spring.io/spring/docs/4.3.5.RELEASE/javadoc-api/org/springframework/scheduling/support/CronSequenceGenerator.html>`_\ .

    Execution time should be fetched from external properties as it may differ depending on the environment in which the application is to be deployed.
    For details on external properties, refer to \ :doc:`../GeneralFuncDetail/PropertyManagement`\ .

 .. tip::

    In the above example, cron is used as a trigger for executing tasks. However, other triggers namely fixed-delay and fixed-rate are also set by default and should be selectively used as per requirement.

    When the default triggers do not satisfy the requirements, an independent trigger can be set by specifying the bean implementing \ ``org.springframework.scheduling.Trigger``\  in trigger attribute.

|

Appendix
--------------------------------------------------------------------------------
Security issues related to file upload
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Following security issues need to be considered when providing File Upload functionality.

#. :ref:`file-upload_security_related_warning_points_dos`
#. :ref:`file-upload_security_related_warning_points_server_scripting`
#. :ref:`file-upload_security_related_warning_points_directory_traversal`

Security measures are described below.


.. _file-upload_security_related_warning_points_dos:

Dos attack with respect to upload functionality
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Dos attack with respect to upload functionality is when load on the server is increased by continuously uploading large files,
thereby crashing the server or reducing its response speed.

| When there is no limit on the size of files to be uploaded and multipart request, the resistance to Dos attack becomes weak.
| In order to enhance the resistance towards Dos attack, size limit needs to be set for a request, by using \ ``<multipart-config>``\  element explained in \ :ref:`file-upload_how_to_usr_application_settings`\ .

|

.. _file-upload_security_related_warning_points_server_scripting:

Attack by executing uploaded files on Web Server
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| In this attack, the files on Web Server can be viewed/altered/deleted by uploading and executing the script files (php, asp, aspx, jsp etc.) that are executable on Web Server (Application Server).
| With Web Server as a platform, another server present in the same network as the Web server, is also vulnerable to such attack.

Measures to be taken against this attack are as follows:

* To view the file contents through a process that displays the contents, without placing uploaded files in the public directory of Web Server (Application Server).
* To ensure that executable script file cannot be uploaded on Web server (Application Server) by restricting the extension of files that can be uploaded.

The attacks can be prevented by implementing either of the above measures; however it is always recommended to implement both the measures.

|

.. _file-upload_security_related_warning_points_directory_traversal:

Directory traversal attack
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Directory traversal attack is an attack that accesses the files on the server that originally should not been accessed, by accessing the file system using inputs which include strings like "../" etc.
| For example, in the web application wherein the file uploaded by the user is placed in the predetermined directory on the server, the file with the name "../../../../somewhere/attack" is uploaded according to implementation method and the file is placed in a directory other than predetermined directory.
| In that case, the file on the server is likely to get tampered by the file uploaded by the attacker.
| A risk of directory traversal is likely in the file download function as well, along with file upload function.
| For example, a type of attack can be considered wherein the attacker would obtain the details of "/etc/passwd" by entering "../../../../etc/passwd", in case of Web application wherein file is downloaded in accordance with the file name entered by the user.

Countermeasures for this attack are as below.

* When the uploaded file is to be stored on the server, it should be stored by using a different name, and the original file name and input value from user should not be used. It should be stored in a format which is not used for actual file access such as storing the correspondence relation with the file name on the server outside DB, for the original file name.
* When the file is to be accessed from  the server, a request should be sent using an identifier for request instead of using actual file name and then converted to the file name on the server side. For example, identifiers "id01" and "id02" are used for actual file names "file_A" and "file_B" wherein if the client requests for "id01", corresponding "file_A" on the server is accessed.

.. tip::
   
   Another countermeasure can be taken into consideration wherein file path thus entered is normalised ( "./" or "../" etc, should be developed in a format which does not include strings with specific significance in the file system) and the access is given after checking whether the path matches (begins-with match) with the path already determined.
   However, if input value encoding and the variation in the path format according to OS is taken into consideration, it is difficult to determine whether the normalization has been done appropriately.
   Hence, it is preferable to avoid accessing the file system by using the input value from user.

.. _file-upload_usage_commons_fileupload:

File upload using Commons FileUpload
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
If File Upload functionality of Servlet 3.0 is only used partially on Application Server, 
it may likely result into garbling of multi byte characters of file names or request parameters.

For example: If File Upload functionality of Servlet 3.0 is used on WebLogic 12.1.3,
it has been confirmed that multi byte characters of fields to be sent along with file are garbled.
Note that it has been corrected in WebLogic 12.2.1.

**This problem can be avoided using Commons FileUpload.
Therefore, this guideline describes about file upload using Commons FileUpload
as a temporary measure for the specific environment where problems are likely to occur.
Using Commons FileUpload is not recommended where problems are not likely.**

Perform the following settings when using Commons FileUpload.

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

   * - | Sr. No.
     - | Description
   * - | (1)
     - | Add dependency to \ ``commons-fileupload``\ .
     
.. note::  

	In the above setting example, since it is assumed that the dependent library version is managed by the parent project terasoluna-gfw-parent , specifying the version in pom.xml is not necessary.  
	The above dependent library used by terasoluna-gfw-parent is defined by \ `Spring IO Platform <http://platform.spring.io/platform/>`_\ .

.. warning::

    In case of using Apache Commons FileUpload,
    security vulnerabilities reported in \ `CVE-2014-0050 <http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0050>`_\  and `CVE-2016-3092 <https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-3092>`_\  are likely to occur.
    Confirm that there are no vulnerabilities in the version of Apache Commons FileUpload to be used.

    When using Apache Commons FileUpload, version 1.3.2 or above should be used.

    Note that, if a version managed by Spring IO Platform Athens-SR2.RELEASE which is in conformance with TERASOLUNA Server Framework for Java version 5.3.0.RELEASE is used, vulnerabilities reported in CVE-2014-0050 and CVE-2016-3092 do not occur.
    When Apache Commons FileUpload version is to be changed intentionally, a version wherein corresponding vulnerability has been addressed must be specified.

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

   * - | Sr. No.
     - | Description
   * - | (1)
     - | Perform bean definition of \ ``CommonsMultipartResolver``\  with \ ``MultipartResolver``\  implemented using Commons FileUpload.
       | Specify \ ``"filterMultipartResolver"``\  in bean ID.
   * - | (2)
     - | Set maximum size allowed in file upload.
       | In case of Commons FileUpload, it should be noted that the maximum value is the entire size of request including header.
       | Moreover, **as the default value is -1 (unlimited), make sure to set a value.** For other properties, refer to \ `JavaDoc <http://docs.spring.io/spring-framework/docs/4.3.5.RELEASE/javadoc-api/org/springframework/web/multipart/commons/CommonsMultipartResolver.html>`_\ .

.. warning::

    In case of using Commons Fileupload, \ ``MultipartResolver``\  should be defined in \ :file:`applicationContext.xml`\  and not in \ :file:`spring-mvc.xml`\ .
    It should be deleted if defined in \ :file:`spring-mvc.xml`\ .


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

   * - Sr.No.
     - Description
   * - | (1)
     - | When using Commons FileUpload, an upload function of Servlet 3.0 should be disabled.
       | If \ ``<multipart-config>``\  element is present in \ ``DispatcherServlet``\  definition, make sure to delete the same. 
   * - | (2)
     - | When using Commons Fileupload, \ ``MultipartFilter``\  must be defined to enable security countermeasures which use Spring Security.
       | \ ``MultipartFilter``\ mapping should be defined before defining springSecurityFilterChain (Servlet Filter of Spring Security).

.. tip::

    \ ``MultipartFilter``\ is a mechanism to perform the file upload process by fetching \ ``MultipartResolver``\ 
    registered with bean ID \ ``"filterMultipartResolver"``\  from DI container (:file:`applicationContext.xml`).

|

.. raw:: latex

   \newpage

