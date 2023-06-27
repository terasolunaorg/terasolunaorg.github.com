Change Log
================================================================================

.. tabularcolumns:: |p{0.15\linewidth}|p{0.25\linewidth}|p{ 0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 15 25 60

    * - Modified on
      - Modified locations
      - Modification details

    * - 2017-03-17
      - \-
      - 5.2.1 RELEASE version published

        * For details of update, refer \ `Issue list of 5.2.1 <https://github.com/terasolunaorg/guideline/issues?utf8=%E2%9C%93&q=label%3A5.3.0%20is%3Aissue%20is%3Aclosed%20>`_\.

    * -
      - General
      - Clerical errors of guideline(typing mistake, simple descriptive error etc ).
      
        * For details of update,please refer,\ `5.2.1's Issue list(clerical error) <https://github.com/terasolunaorg/guideline/issues?utf8=%E2%9C%93&q=%20label%3A5.2.1%20is%3Aclosed%20label%3A%22clerical%20error%22%20>`_\.
        
        Description details modified

        * For details of update、please refer \ `5.2.1's Issue list(improvement) <https://github.com/terasolunaorg/guideline/issues?utf8=%E2%9C%93&q=label%3A5.2.1%20label%3Aimprovement%20is%3Aclosed%20>`_\.

        Change to fix the version of \ ``maven-archetype-plugin``\ used while creating following project to 2.4(\ `guideline#2523 <https://github.com/terasolunaorg/guideline/issues/2523>`_\ )

        * :doc:`../Overview/FirstApplication`  
        * :doc:`../ImplementationAtEachLayer/CreateWebApplicationProject`  
        * :doc:`../Tutorial/TutorialTodo`  
        * :doc:`../Tutorial/TutorialSecurity`

    * -
      - :doc:`../Introduction/CriteriaBasedMapping`
      - Description details added

        * A table listing a point of view by CVE is added in Mapping based on security measures (\ `guideline#2439 <https://github.com/terasolunaorg/guideline/issues/2439>`_\ )


    * -
      - :doc:`../ImplementationAtEachLayer/DomainLayer`
      - Description details modified

        * Modified signature-limiting interface and base class implementation sample(\ `guideline#2219 <https://github.com/terasolunaorg/guideline/issues/2219>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Validation`
      - Description details modified

        * Add explanation on how to include input check target in message(\ `guideline#2002 <https://github.com/terasolunaorg/guideline/issues/2002>`_\ )

        * Modified check contents description  of input check by @URL(\ `guideline#2260 <https://github.com/terasolunaorg/guideline/issues/2260>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/SessionManagement`
      - Description details added

        * Added how to prevent binding of request parameters when receiving object stored in session scope(\ `guideline#1293 <https://github.com/terasolunaorg/guideline/issues/1293>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Internationalization`
      - Description details added

        * Example when internationalization is not applied and its countermeasures added (\ `guideline#2427 <https://github.com/terasolunaorg/guideline/issues/2427>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessMyBatis3`
      - Description details added

        * (\ `guideline#2375 <https://github.com/terasolunaorg/guideline/issues/2375>`_\ ) Added description of setting related to calling rollback processing when an error occurs at the time of commit.

    * -
      - :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessJpa`
      - Description details added

        * Added warning and measures(\ `guideline#2439 <https://github.com/terasolunaorg/guideline/issues/2439>`_\ )  related to \ `CVE-2016-6652 <https://pivotal.io/security/cve-2016-6652>`_\ (Vulnerability that may be subject to blind SQL injection attack).

    * -
      - :doc:`../ArchitectureInDetail/MessagingDetail/Email`
      - Description details added

        * Issues occuring in JavaMail and the methods to avoid the same added (\ `guideline#2190 <https://github.com/terasolunaorg/guideline/issues/2190>`_\ )

    * -
      - :doc:`../Security/Authentication`
      - Description details added
      
        * Description added for value attribute of checkbox used in Remember Me authentication (\ `guideline#785 <https://github.com/terasolunaorg/guideline/issues/785>`_\ )

        Description details modified

        * Description for use of SecureRandom modified (\ `guideline#2177 <https://github.com/terasolunaorg/guideline/issues/2177>`_\ )

    * -
      - :doc:`../Security/Authorization`
      - Description details added

        * Warning related to access control for specific URL added (\ `guideline#2399 <https://github.com/terasolunaorg/guideline/issues/2399>`_\ )

        * Added warning and measures(\ `guideline#2439 <https://github.com/terasolunaorg/guideline/issues/2439>`_\ )  related to \ `CVE-2016-9879 <https://pivotal.io/security/cve-2016-9879>`_\ 

    * - 2016-08-31
      - \-
      - 5.2.0 RELEASE version published

        * For details of update, refer \ `Issue list of 5.2.0 <https://github.com/terasolunaorg/guideline/issues?utf8=%E2%9C%93&q=label%3A5.2.0%20is%3Aissue%20is%3Aclosed%20>`_\.

    * -
      - General
      - Correction of errors in the guideline (typos or simple description errors)

        * For details of modifications, refer \ `Issue list of 5.2.0 (clerical error) <https://github.com/terasolunaorg/guideline/issues?utf8=%E2%9C%93&q=%20label%3A5.2.0%20is%3Aclosed%20label%3A%22clerical%20error%22%20>`_\.

        Description details modified

        * For details of modification, refer \ `Issue list of 5.2.0 (improvement) <https://github.com/terasolunaorg/guideline/issues?utf8=%E2%9C%93&q=label%3A5.2.0%20label%3Aimprovement%20is%3Aclosed%20>`_\.

        Review of all the chapters

        * For details of update, refer \ `Optimize the order of chapters and sections #1683 <https://github.com/terasolunaorg/guideline/issues/1683>`_\.

        Update common library version to 5.2.0.

        * For details of update, refer \ `Check Version  #2076 <https://github.com/terasolunaorg/guideline/issues/2076>`_\.

        Description details modified 

        * Added regarding pom dependency of common library (\ `guideline#1982 <https://github.com/terasolunaorg/guideline/issues/1982>`_\ )

    * -
      - :doc:`../Overview/FrameworkStack`
      - Description details added

        * Embedding status of common library standards of blank project added (\ `guideline#1700 <https://github.com/terasolunaorg/guideline/issues/1700>`_\ )
        * mybatis-typehandlers-jsr310, jackson-datatype-jsr310 added to OSS stack (\ `guideline#1966 <https://github.com/terasolunaorg/guideline/issues/1966>`_\ )
        * spring-jms and its dependent libraries added to OSS stack (\ `guideline#1992 <https://github.com/terasolunaorg/guideline/issues/1992>`_\ )

        Version of OSS used (Spring IO Platform version) updated)

        * Spring IO Platform  version updated to 2.0.6.RELEASE
        * Spring Framework version updated to 4.2.7.
        * Spring Security version updated to 4.0.4.RELEASE

        OSS version used in accordance with Spring IO Platform version update is updated

    * -
      - :doc:`../ImplementationAtEachLayer/DomainLayer`
      - Description details added

        * For MyBatis 3.3 + MyBatis-Spring 1.2, "value specified in timeout attribute of @Transactinal is not used" is added (\ `guideline#1777 <https://github.com/terasolunaorg/guideline/issues/1777>`_\ )

    * -
      - :doc:`../ImplementationAtEachLayer/ApplicationLayer`
      - Description details added

        * HttpSession should not be used as an argument for handler method (\ `guideline#1313 <https://github.com/terasolunaorg/guideline/issues/1313>`_\ )
        * Precautions for using JSR-310 Date and Time API are described (\ `guideline#1991 <https://github.com/terasolunaorg/guideline/issues/1991>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Validation`
      - Description details modified

        * A method to directly handle a message property file without conversion from  Native to Ascii is added (\ `guideline#994 <https://github.com/terasolunaorg/guideline/issues/994>`_\ )
        * Description for cross-field validation added (\ `guideline#1561 <https://github.com/terasolunaorg/guideline/issues/1561>`_\ )
        * @DateTimeFormat description added (\ `guideline#1873 <https://github.com/terasolunaorg/guideline/issues/1873>`_\ )
        * Description for ValidationMessages.properties modified (\ `guideline#1948 <https://github.com/terasolunaorg/guideline/issues/1948>`_\ )
        * Precautions for input check which use Method Validation added (\ `guideline#1998 <https://github.com/terasolunaorg/guideline/issues/1998>`_\ )

        Description details added

        * Description for OS command injection added (\ `guideline#1957 <https://github.com/terasolunaorg/guideline/issues/1957>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/ExceptionHandling`
      - Modification associated with Spring Framework 4.2.7
      
        * Description details for HTTP response header output modified (\ `guideline#1965 <https://github.com/terasolunaorg/guideline/issues/1965>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/DoubleSubmitProtection`
      - Description details added
      
        * Description for specifications and implementation methods of \ ``TransactionTokenType.CHECK``\  which was newly added in type attribute of \ ``@TransactionTokenCheck``\  annotation
          (\ `guideline#2071 <https://github.com/terasolunaorg/guideline/issues/2071>`_\ )

        "How to manage transaction token life cycle in How To Extend programmatic" deleted.

        * When API for application offered by \ ``TransactionTokenContext``\ is used,
          it impacts the behaviour of internal framework like inability to maintain \ ``TransactionToken``\  in the appropriate state
          Current API is deprecated. Description for how to use function in accordance with deprecation, deleted. 

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Internationalization`
      - Description details modified

        *   Position of request parameter (default parameter name) description modified (\ `guideline#1354 <https://github.com/terasolunaorg/guideline/issues/1354>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/FileUpload`
      - Description details added

        * \ `CVE-2016-3092 <https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-3092>`_\ Precautions for (File Upload vulnerability) added (\ `guideline#1973 <https://github.com/terasolunaorg/guideline/issues/1973>`_\ )
        * Description for directory traversal attack added (\ `guideline#2010 <https://github.com/terasolunaorg/guideline/issues/2010>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/HealthCheck`
      - Added new

        * Health check added (\ `guideline#1698 <https://github.com/terasolunaorg/guideline/issues/1698>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebServiceDetail/REST`
      - Description details changed / added

        * Description for the configuration while using JSR-310 Date and Time API / Joda Time changed (\ `guideline#1966 <https://github.com/terasolunaorg/guideline/issues/1966>`_\ )
        * Precautions while using Jackson in Java SE 7 environment described (\ `guideline#1966 <https://github.com/terasolunaorg/guideline/issues/1966>`_\ )
        * Configuration while using JSR-310 Date and Time API in JSON described (\ `guideline#1966 <https://github.com/terasolunaorg/guideline/issues/1966>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebServiceDetail/RestClient`
      - Description details modified

        * HTTP Proxy server configuration for RestClient added (\ `guideline#1856 <https://github.com/terasolunaorg/guideline/issues/1856>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebServiceDetail/SOAP`
      - Description details added

        * Added an option "Do not connect to SOAP server at the time of SOAP client start (\ `guideline#1871 <https://github.com/terasolunaorg/guideline/issues/1871>`_\ )
        * Description for env project of SOAP client modified (\ `guideline#1901 <https://github.com/terasolunaorg/guideline/issues/1901>`_\ )
        * How to fetch status code at the time of SOAP Web service exception occurrence added (\ `guideline#2007 <https://github.com/terasolunaorg/guideline/issues/2007>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessMyBatis3`
      - Description details added

        * "How to avoid tentative WARN log output" deleted (\ `guideline#1292 <https://github.com/terasolunaorg/guideline/issues/1292>`_\ )
        * "How to configure for using JSR-310 Date and Time API in Mybatis3.3" described (\ `guideline#1966 <https://github.com/terasolunaorg/guideline/issues/1966>`_\ )
        * Precautions while using MyBatis in Java SE 7 environment described (\ `guideline#1966 <https://github.com/terasolunaorg/guideline/issues/1966>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/DataAccessDetail/ExclusionControl`
      - Description details added

        *  warning message added to ExclusionControl (\ `guideline#1694 <https://github.com/terasolunaorg/guideline/issues/1694>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/GeneralFuncDetail/Logging`
      - Description details added
        
        * "How to extend in order to output log message with ID" described (\ `guideline#1928 <https://github.com/terasolunaorg/guideline/issues/1928>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/GeneralFuncDetail/StringProcessing`
      - Description details added

        * An example to add terasoluna-gfw-string to dependency is added (\ `guideline#1699 <https://github.com/terasolunaorg/guideline/issues/1699>`_\ )
        * Precautions for surrogate pair added to description of @Size annotation (\ `guideline#1874 <https://github.com/terasolunaorg/guideline/issues/1874>`_\ )
        * Description for JIS characters \ ``U+2014``\(EM DASH) UCS(Unicode) characters added (\ `guideline#1914 <https://github.com/terasolunaorg/guideline/issues/1914>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/GeneralFuncDetail/Dozer`
      - Description details added

        * Precautions while using JSR-310 Date and Time API described (\ `guideline#1966 <https://github.com/terasolunaorg/guideline/issues/1966>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/MessagingDetail/JMS`
      - Added new

        * JMS added (\ `guideline#1407 <https://github.com/terasolunaorg/guideline/issues/1407>`_\ )

    * -
      - :doc:`../Security/Authentication`
      - Modifications for Spring Security 4.0.4

        * Code example modified to include modification of specifications of authentication-failure-url in Spring 4.0.4 and Note deleted (\ `guideline#1963 <https://github.com/terasolunaorg/guideline/issues/1963>`_\ )

    * -
      - :doc:`../Security/Authorization`
      - Description details added

        * How to handle \ `CVE-2016-5007 Spring Security / MVC Path Matching Inconsistency <https://pivotal.io/security/cve-2016-5007>`_\ added (\ `guideline#1976 <https://github.com/terasolunaorg/guideline/issues/1976>`_\ )

    * -
      - :doc:`../Security/SecureLoginDemo`
      - Description details added

        * "Input value check for security" added
        * "Audit log output" added

    * -
      - :doc:`../Appendix/ReferenceBooks`
      - Description details added

        * Spring thorough introduction" added as a a reference material (\ `guideline#2043 <https://github.com/terasolunaorg/guideline/issues/2043>`_\ )

    * - 2016-02-24
      - \-
      - 5.1.0 RELEASE version published

        * For details of change contents, refer \ `5.1.0 Issue List <https://github.com/terasolunaorg/guideline/issues?q=is%3Aissue+milestone%3A5.1.0+is%3Aclosed>`_\ .
    * -
      - General
      - Correction of errors in the guideline (typo mistakes and simple description errors)

        Description details modified

        * For details of modification, refer \ `5.1.0 Issue list (improvement) <https://github.com/terasolunaorg/guideline/issues?q=milestone%3A5.1.0+label%3Aimprovement+is%3Aclosed>`_\ .

    * -
      - :doc:`index`
      - Description details added

        * Description related to operation verification environment of the details described in the guideline added

    * -
      - :doc:`../Overview/FrameworkStack`
      - OSS version to be used (Spring IO Platform version) updated

        * Spring IO Platform version updated in 2.0.1.RELEASE
        * Spring Framework version updated in 4.2.4.RELEASE
        * Spring Security version updated in 4.0.3.RELEASE

        OSS version to be used along with Spring IO Platform version update is updated

        * OSS version to be used updated. For update details, refer \ `version 5.1.0 migration guide <https://github.com/terasolunaorg/terasoluna-gfw/wiki/Migration-Guide-5.1.0_ja#step-1-update-dependency-libraries>`_\ .

        New project added

        * Descriptions for \ ``terasoluna-gfw-string``\ , \ ``terasoluna-gfw-codepoints``\ , \ ``terasoluna-gfw-validator``\ , \ ``terasoluna-gfw-web-jsp``\  projects added.

        New function of common library added

        \ ``terasoluna-gfw-string``\ 
         * Half width to full width conversion

        \ ``terasoluna-gfw-codepoints``\
         * Codepoint check
         * Bean Validation constraint annotation for code point check

        \ ``terasoluna-gfw-validator``\
         * Bean Validation constraint annotation for byte length check
         * Bean Validation constraint annotation for field value comparison correlation check

    * -
      - :doc:`../Overview/FirstApplication`
      - Description details modified

        *  Modification of sample source corresponding to Spring Security 4 (\ `guideline#1519 <https://github.com/terasolunaorg/guideline/issues/1519>`_\ )

         * \ ``AuthenticationPrincipalArgumentResolver``\  package changed

    * -
      - :doc:`../Tutorial/TutorialTodo`
      - Modifications corresponding to Spring Security 4

        *  Modification of source corresponding to Spring Security 4 (\ `guideline#1519 <https://github.com/terasolunaorg/guideline/issues/1519>`_\ )

         * \ ``AuthenticationPrincipalArgumentResolver``\  package changed
         * Since the specification is true by default, \ ``<use-expressions="true">``\  deleted from sample source

    * -
      - :doc:`../ImplementationAtEachLayer/CreateWebApplicationProject`
      - Modification of description details

        *  A method wherein mvn command is used in the offline environment is added (\ `guideline#1197 <https://github.com/terasolunaorg/guideline/issues/1197>`_\ )

    * -
      - :doc:`../ImplementationAtEachLayer/ApplicationLayer`
      - Description details modified

        *  A method to create a request URL using EL function is added (\ `guideline#632 <https://github.com/terasolunaorg/guideline/issues/632>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessCommon`
      - Description details added

        *  Precautions for \ ``Log4jdbcProxyDataSource``\  overhead added (\ `guideline#1471 <https://github.com/terasolunaorg/guideline/issues/1471>`_\ )
    * -
      - :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessMyBatis3`
      - Description details corresponding to MyBatis 3.3 added

        * Setup method of \ ``defaultFetchSize``\  added (\ `guideline#965 <https://github.com/terasolunaorg/guideline/issues/965>`_\ )
        * "Changed the default at the time of delayed reading to \ ``JAVASSIST``\" added (\ `guideline#1384 <https://github.com/terasolunaorg/guideline/issues/1384>`_\ )
        * Sample code which assigns Genrics to \ ``ResultHandler``\  modified (\ `guideline#1384 <https://github.com/terasolunaorg/guideline/issues/1384>`_\ )
        * Source example which use newly added \ ``@Flush``\  annotation, and precautions added (\ `guideline#915 <https://github.com/terasolunaorg/guideline/issues/915>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessJpa`
      - Bug correction for the guideline

        *  Utility which use Like condition modified appropriately (\ `guideline#1464 <https://github.com/terasolunaorg/guideline/issues/1464>`_\ )
        *  Incorrect implementation of true value in JPQL corrected (\ `guideline#1525 <https://github.com/terasolunaorg/guideline/issues/1525>`_\ )
        *  Incorrect implementation of pagination corrected (\ `guideline#1463 <https://github.com/terasolunaorg/guideline/issues/1463>`_\ )
        *  Incorrect implementation of sample code corrected which implements \ ``DateTimeProvider``\  (\ `guideline#1327 <https://github.com/terasolunaorg/guideline/issues/1327>`_\ )
        *  Incorrect implementation in Factory class for generating an instance of implementation class for common Repository interface corrected (\ `guideline#1327 <https://github.com/terasolunaorg/guideline/issues/1327>`_\ )

        Description details modified

        *  Default value of \ ``hibernate.hbm2ddl.auto``\  corrected (\ `guideline#1282 <https://github.com/terasolunaorg/guideline/issues/1282>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Validation`
      - Description details modified

        *  Description for MethodValidation added (\ `guideline#708 <https://github.com/terasolunaorg/guideline/issues/708>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/GeneralFuncDetail/Logging`
      - Description details modified

        * Description where \ ``ServiceLoader``\  mechanism is used in Logback setting, is added (\ `guideline#1275 <https://github.com/terasolunaorg/guideline/issues/1275>`_\ )
        * Sample source corresponding to Spring Security 4 modified (\ `guideline#1519 <https://github.com/terasolunaorg/guideline/issues/1519>`_\ )

         * Since the specification is true by default, \ ``<use-expressions="true">``\  deleted from the sample source

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/SessionManagement`
      - Description details modified

        *  Description of session scope reference which use SpEL expression is added (\ `guideline#1306 <https://github.com/terasolunaorg/guideline/issues/1306>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Internationalization`
      - Description details modified

        *  Description for appropriately reflecting locale in JSP is added (\ `guideline#1439 <https://github.com/terasolunaorg/guideline/issues/1439>`_\ )
        *  Description of \ ``defaultLocale``\  of \ ``SessionLocalResolver``\  corrected (\ `guideline#686 <https://github.com/terasolunaorg/guideline/issues/686>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/FileUpload`
      - Description details added

        * Description for directory traversal attack added (\ `guideline#2010 <https://github.com/terasolunaorg/guideline/issues/2010>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Codelist`
      - Description details added

        *  Description which recommends a pattern wherein \ ``JdbcTemplate``\  is specified in JdbcCodeList, is added (\ `guideline#501 <https://github.com/terasolunaorg/guideline/issues/501>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/HealthCheck`
      - New

        * Health check added (\ `guideline#1698 <https://github.com/terasolunaorg/guideline/issues/1698>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebServiceDetail/REST`
      - Description details modified

        *  Creation of ObjectMapper which use \ ``Jackson2ObjectMapperFactoryBean``\  added (\ `guideline#1022 <https://github.com/terasolunaorg/guideline/issues/1022>`_\ )
        *  Modified to a format where MyBatis3 is used as a prerequisite in the implementation of domain layer of REST API application (\ `guideline#1323 <https://github.com/terasolunaorg/guideline/issues/1323>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebServiceDetail/RestClient`
      - Added new

        *  REST client (HTTP client) added (\ `guideline#1307 <https://github.com/terasolunaorg/guideline/issues/1307>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebServiceDetail/SOAP`
      - Added new

        *  SOAP Web Service (Server / Client) added (\ `guideline#1340 <https://github.com/terasolunaorg/guideline/issues/1340>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/FileUpload`
      - Description details modified

        * Basic flow of uploading process and its description modified to description which use \ ``MultipartFilter``\  of Spring (\ `guideline#193 <https://github.com/terasolunaorg/guideline/issues/193>`_\ )
        * "A method which sends CSRF token by query parameter" deleted due to issues like security issues, variation in the operation according to AP server etc.
          Precaution - "when allowable size for file upload exceeds, CSRF token check is not carried out appropriately in some AP servers" added (\ `guideline#1602 <https://github.com/terasolunaorg/guideline/issues/1602>`_\ )


    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/FileDownload`
      - Description details corresponding to Spring Framework4.2 added

        *  \ ``AbstractXlsxView``\  which manages xlsx format, is added (\ `guideline#996 <https://github.com/terasolunaorg/guideline/issues/996>`_\ )

        Description details modified

        * Source example which use \ ``com.lowagie:itext:4.2.1``\  modified to a format which uses \ ``com.lowagie:itext:2.1.7``\  for the specification change of the iText

    * -
      - :doc:`../ArchitectureInDetail/MessagingDetail/Email`
      - Added new

        *  E-mail sending (SMTP) added (\ `guideline#1165 <https://github.com/terasolunaorg/guideline/issues/1165>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/GeneralFuncDetail/DateAndTime`
      - Added new

        *  Date and time operation (JSR-310 Date and Time API) added (\ `guideline#1450 <https://github.com/terasolunaorg/guideline/issues/1450>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/GeneralFuncDetail/JodaTime`
      - Description details added and modified

        *  The object of sample code which handles the date that does not use Timezone modified to \ ``LocalDate``\  (\ `guideline#1283 <https://github.com/terasolunaorg/guideline/issues/1283>`_\ )
        *  A method to handle Japanese calendar in Java8 and earlier versions is added (\ `guideline#1450 <https://github.com/terasolunaorg/guideline/issues/1450>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/GeneralFuncDetail/Logging`
      - Description details added
        
        * Extension method to output log message with ID is described (\ `guideline#1928 <https://github.com/terasolunaorg/guideline/issues/1928>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/GeneralFuncDetail/StringProcessing`
      - Added new

        *  String processing added (\ `guideline#1451 <https://github.com/terasolunaorg/guideline/issues/1451>`_\ )
        
    * -
      - :doc:`../ArchitectureInDetail/MessagingDetail/JMS`
      - Added new

        * JMS added (\ `guideline#1407 <https://github.com/terasolunaorg/guideline/issues/1407>`_\ )
        

    * -
      - :doc:`../Security/index`
      - Configuration review

        * \ ``Password hashing``\ moved in :doc:`../Security/Authentication`
        * Session management items are separated as :doc:`../Security/SessionManagement` from :doc:`../Security/Authentication`

    * -
      - :doc:`../Security/SpringSecurity`
      - Modify corresponding to Spring Security 4

        * Restructuring overall description

         *  \ ``spring-security-test``\  introduction
         *  Since the specification is true by default, \ ``<use-expressions="true">``\  deleted from sample source
         * Description related to \ ``RedirectAuthenticationHandler``\ deprecation deleted

    * -
      - :doc:`../Tutorial/TutorialSecurity`
      - Modified corresponding to Spring Security 4

        * Modified tutorial source to a format corresponding to Spring Security 4 (\ `guideline#1519 <https://github.com/terasolunaorg/guideline/issues/1519>`_\ )

    * -
      - :doc:`../Security/Authentication`
      - Modified corresponding to Spring Security 4 (\ `guideline#1519 <https://github.com/terasolunaorg/guideline/issues/1519>`_\ )

        * Restructuring of overall description

         * Deleted \ ``auto-config="true"``\
         * Authentication event listener modified to \ ``@org.springframework.context.event.EventListener``\
         * Modified \ ``AuthenticationPrincipal``\  package
         * Since prefix is assigned by default, \ ``ROLE_``\  prefix deleted from sample source

    * -
      - :doc:`../Security/Authorization`
      - Modified corresponding to Spring Security 4 (\ `guideline#1519 <https://github.com/terasolunaorg/guideline/issues/1519>`_\ )

        * Restructuring of overall description

         *  Since the prefix is assigned by default, \ ``ROLE_``\  prefix deleted from sample source
         *  Since the specification is true by default, \ ``<use-expressions="true">``\  deleted from sample source
         *  Definition example of \ ``@PreAuthorize``\  added

    * -
      - :doc:`../Security/CSRF`
      - Modified corresponding to Spring Security 4

        * Restructuring of overall description

         * CSRF invalidation settings modified \ ``<sec:csrf disabled="true"/>``\

        * Description details modified

         * Items related to multi-part request moved to :doc:`../ArchitectureInDetail/WebApplicationDetail/FileUpload` (\ `guideline#1602 <https://github.com/terasolunaorg/guideline/issues/1602>`_\ )

    * -
      - :doc:`../Security/Encryption`
      - Added new

        * Encryption guidelines added (\ `guideline#1106 <https://github.com/terasolunaorg/guideline/issues/1106>`_\ )

    * -
      - :doc:`../Security/SecureLoginDemo`
      - Added new

    * -
      - :doc:`../Security/SecureLoginDemo`
      - Description details added

        * "Input check for security" added
        * "Audit log output" added

        *  Typical implementation example of security requirements added (\ `guideline#1604 <https://github.com/terasolunaorg/guideline/issues/1604>`_\ )

    * -
      - :doc:`../Tutorial/TutorialSession`
      - Added new

        *  Session tutorial added (\ `guideline#1599 <https://github.com/terasolunaorg/guideline/issues/1599>`_\ )

    * -
      - :doc:`../Tutorial/TutorialREST`
      - Modified corresponding to Spring Security 4

        *  Modified source corresponding to Spring Security 4 (\ `guideline#1519 <https://github.com/terasolunaorg/guideline/issues/1519>`_\ )

         * CSRF invalidation settings modified \ ``<sec:csrf disabled="true"/>``\
         * Since the specification is true by default, \ ``<use-expressions="true">``\  deleted from sample source

    * - 2015-08-05
      - \-
      - Released "5.0.1 RELEASE" version

        * For update details, refer to \ `Issue list of 5.0.1 <https://github.com/terasolunaorg/guideline/issues?q=is%3Aissue+milestone%3A5.0.1+is%3Aclosed>`_\
    * -
      - Overall modifications
      - Fixed guideline errors (corrected typos, mistakes in description, etc.)

        * For modification details, refer to \ `Issue list of 5.0.1 (clerical error) <https://github.com/terasolunaorg/guideline/issues?q=is%3Aclosed+milestone%3A5.0.1+label%3A%22clerical+error%22>`_\

        Improved the description

        * For improvement details, \ `Issue list of 5.0.1 (improvement) <https://github.com/terasolunaorg/guideline/issues?q=milestone%3A5.0.1+label%3Aimprovement+is%3Aclosed>`_\

        Fixed the description about application server

        * Removed the description for the Resin
        * Updated the link of reference page
    * -
      - :doc:`index`
      - Added the description

        * Added description about tested environments for contents described in this guideline
    * -
      - :doc:`../Overview/FrameworkStack`
      - Updated the OSS version(Spring IO Platform version) to protect security vulnerability

        * Spring IO Platform version updated to 1.1.3.RELEASE
        * Spring Framework version updated to 4.1.7.RELEASE (\ `CVE-2015-3192 <http://pivotal.io/security/cve-2015-3192>`_\ )
        * JSTL version updated to 1.2.5 (\ `CVE-2015-0254 <http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-0254>`_\ )

        Updated the OSS version by the Spring IO Platform version update

        * Updated the OSS version to be used. For update details, refer to \ `Migration guide of version 5.0.1 <https://github.com/terasolunaorg/terasoluna-gfw/wiki/Migration-Guide-5.0.1#step-1-update-dependency-libraries>`_\

        Improved the description (\ `guideline#1148 <https://github.com/terasolunaorg/guideline/issues/1148>`_\ )

        * Added the description of \ ``terasoluna-gfw-recommended-dependencies``\ ,\ ``terasoluna-gfw-recommended-web-dependencies``\  and \ ``terasoluna-gfw-parent``\
        * Modified the description for some project
        * Added the illustration to indicate project dependencies
    * -
      - :doc:`../ImplementationAtEachLayer/CreateWebApplicationProject`
      - Added the description

        * Added how to build a war file (\ `guideline#1146 <https://github.com/terasolunaorg/guideline/issues/1146>`_\ )
    * -
      - :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessCommon`
      - Added the description

        * Added the description of \ ``DataSource``\  switching functionality (\ `guideline#1071 <https://github.com/terasolunaorg/guideline/issues/1071>`_\ )
    * -
      - :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessMyBatis3`
      - Fixed the guideline bug

        * Modified the description about timing of batch execution (\ `guideline#903 <https://github.com/terasolunaorg/guideline/issues/903>`_\ )
    * -
      - :doc:`../ArchitectureInDetail/GeneralFuncDetail/Logging`
      - Improved the description

        * Added the description about \ ``additivity``\  attribute of \ ``<logger>``\  tag (\ `guideline#977 <https://github.com/terasolunaorg/guideline/issues/977>`_\ )
    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/SessionManagement`
      - Improved the description

        * Modified the description about how to define a session scope bean (\ `guideline#1082 <https://github.com/terasolunaorg/guideline/issues/1082>`_\ )
    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/DoubleSubmitProtection`
      - Added the description

        * Added the description about the transaction token check in case that response cache is disabled (\ `guideline#1260 <https://github.com/terasolunaorg/guideline/issues/1260>`_\ )
    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Codelist`
      - Added the description

        * Added how to display a code name (\ `guideline#1109 <https://github.com/terasolunaorg/guideline/issues/1109>`_\ )
    * -
      - | :doc:`../ArchitectureInDetail/WebApplicationDetail/Ajax`
        | :doc:`../ArchitectureInDetail/WebServiceDetail/REST`
      - Added the warning about \ `CVE-2015-3192 <http://pivotal.io/security/cve-2015-3192>`_\ (XML security vulnerability)

        * Added the warning at the time of the StAX(Streaming API for XML) use (\ `guideline#1211 <https://github.com/terasolunaorg/guideline/issues/1211>`_\ )
    * -
      - | :doc:`../ArchitectureInDetail/WebApplicationDetail/Pagination`
        | :doc:`../ArchitectureInDetail/WebApplicationDetail/TagLibAndELFunctions`
      - Modified in accordance with bug fixes of common library

        * Modified the description about \ ``f:query``\  specification , in accordance with bug fixes of common library (\ `terasoluna-gfw#297 <https://github.com/terasolunaorg/terasoluna-gfw/issues/297>`_\ ) (\ `guideline#1244 <https://github.com/terasolunaorg/guideline/issues/1244>`_\ )
    * -
      - :doc:`../Security/Authentication`
      - Improved the description

        * Added the notes about handling with some properties of parent class of \ ``ExceptionMappingAuthenticationFailureHandler``\  (\ `guideline#812 <https://github.com/terasolunaorg/guideline/issues/812>`_\ )
        * Modified the setting example for the \ ``requiresAuthenticationRequestMatcher``\  property of \ ``AbstractAuthenticationProcessingFilter``\  (\ `guideline#1110 <https://github.com/terasolunaorg/guideline/issues/1110>`_\ )
    * -
      - :doc:`../Security/Authorization`
      - Fixed the guideline bug

        * Modified the setting example for the \ ``access``\  attribute of \ ``<sec:authorize>``\  tag (JSP tag library) (\ `guideline#1003 <https://github.com/terasolunaorg/guideline/issues/1003>`_\ )
    * -
      - Elimination of environmental dependency
      - Added the description

        * Added how to apply the external classpath(alternative functionality of \ ``VirtualWebappLoader``\  of Tomcat7) at the time of Tomcat8 use (\ `guideline#1081 <https://github.com/terasolunaorg/guideline/issues/1081>`_\ )
    * - 2015-06-12
      - Overall modifications
      - Released English version of "5.0.0 RELEASE"
    * - 2015-03-06
      - :doc:`../ArchitectureInDetail/WebServiceDetail/REST`
      - Guideline bug modification

        * Modification of sample code for exception handling (the code that contains the issue of generating \ ``NullPointerException``\ ).
          For improvement details, refer to \ `guideline#918 <https://github.com/terasolunaorg/guideline/issues/918>`_\ .
    * -
      - :doc:`../Tutorial/TutorialREST`
      - Guideline bug modification

        * Fixed a problem that generates \ `` NullPointerException`` \ in the processing of exception handling.
          For improvement details, refer to \ `guideline#918 <https://github.com/terasolunaorg/guideline/issues/918>`_\ .
    * - 2015-02-23
      - \-
      - Released "5.0.0 RELEASE" version

        * For update details, refer to \ `Issue list of 5.0.0 <https://github.com/terasolunaorg/guideline/issues?q=is%3Aissue+milestone%3A5.0.0+is%3Aclosed>`_\  and \ `Backport issue list of 1.0.2  <https://github.com/terasolunaorg/guideline/issues?q=is%3Aclosed+milestone%3A1.0.2+label%3Abackport>`_\ .
    * -
      - Overall modifications
      - Fixed guideline errors (corrected typos, mistakes in description, etc.)

        * For modification details, refer to \ `Backport issue list of 1.0.2 (clerical error) <https://github.com/terasolunaorg/guideline/issues?q=is%3Aclosed+milestone%3A1.0.2+label%3Abackport+label%3A%22clerical+error%22>`_\ .

        Improved the description

        * For improvement details, refer to \ `Issue list of 5.0.0 (improvement) <https://github.com/terasolunaorg/guideline/issues?q=milestone%3A5.0.0+label%3Aimprovement+is%3Aclosed>`_\  and \ `Backport issue list of 1.0.2 (improvement) <https://github.com/terasolunaorg/guideline/issues?q=is%3Aclosed+milestone%3A1.0.2+label%3Aimprovement+label%3Abackport>`_\ .

        Added new

        * :doc:`../ImplementationAtEachLayer/CreateWebApplicationProject`
        * :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessMyBatis3`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/TagLibAndELFunctions`
        * :doc:`../Appendix/Lombok`

        Updated in accordance with version 5.0.0 

        * Deleted MyBatis2 
    * -
      - :doc:`../Overview/FrameworkStack`
      - Spring IO Platform compatible

        * Added a point that except for some libraries, the management of recommended libraries is changed to a structure delegating it to Spring IO Platform.

        Updated the OSS version

        * Updated the OSS version to be used. For update details, refer to \ `Migration guide of version 5.0.0 <https://github.com/terasolunaorg/terasoluna-gfw/wiki/Migration-Guide-5.0.0#step-1-update-dependency-libraries>`_\ .
    * -
      - :doc:`../Overview/FirstApplication`
      - Updated in accordance with version 5.0.0

        * Used Spring Framework 4.1
        * Reviewed structure of document.
    * -
      - :doc:`../Overview/ApplicationLayering`
      - Fixed bugs in English translation.

        * Fixed translation bugs related to domain layer and other layers.
          For modification details, refer to \ `guideline#364 issue <https://github.com/terasolunaorg/guideline/issues/364>`_\ .
    * -
      - :doc:`../Tutorial/TutorialTodo`
      - Updated in accordance with version 5.0.0

        * Use of Spring Framework 4.1.
        * MyBatis3 support as infrastructure layer.
        * Revised document structure.
    * -
      - :doc:`../ImplementationAtEachLayer/CreateWebApplicationProject`
      - Added new

        * Added a method to create a project having multi project structure
    * -
      - :doc:`../ImplementationAtEachLayer/DomainLayer`
      - Modified in accordance with Spring Framework 4.1

        * Added description about handling \ ``@Transactional``\  of JTA 1.2.
          For modification details, refer to \ `guideline#562 issue <https://github.com/terasolunaorg/guideline/issues/562>`_\ .
        * Modified description about handling \ ``@Transactional(readOnly = true)``\  when using JPA (Hibernate implementation).
          With \ `SPR-8959 <https://jira.spring.io/browse/SPR-8959>`_\  (Spring Framework 4.1 and later versions) support,
          it has been improved so that instruction can be given so as to handle as "Read-only transactions" for JDBC driver.

        Added description

        * Added notes regarding the cases where "Read-only transactions" are not enabled.
          For added contents, refer to \ `guideline#861 issue <https://github.com/terasolunaorg/guideline/issues/861>`_\ .
    * -
      - :doc:`../ImplementationAtEachLayer/InfrastructureLayer`
      - Modified in accordance with MyBatis3

        * Added a method to use MyBatis3 mechanism as implementation of RepositoryImpl.
    * -
      - :doc:`../ImplementationAtEachLayer/ApplicationLayer`
      - Modified in accordance with Spring Framework 4.1

        * Added description about the attribute (attribute to filter the Controllers to be used) added in \ ``@ControllerAdvice``\ .
          For modification details, refer to \ `guideline#549 issue <https://github.com/terasolunaorg/guideline/issues/549>`_\ .
        * Added description about \ ``<mvc:view-resolvers>``\ .
          For modification details, refer to \ `guideline#609 issue <https://github.com/terasolunaorg/guideline/issues/609>`_\ .
    * -
      - :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessCommon`
      - Modified in accordance with bug fixes of common library

        * Added description about handling double byte wild card characters (\ ``％``\ , \ ``＿``\)\ , in accordance with bug fixes of common library (\ `terasoluna-gfw#78 <https://github.com/terasolunaorg/terasoluna-gfw/issues/78>`_\ ).
          For modification details, refer to \ `guideline#712 issue <https://github.com/terasolunaorg/guideline/issues/712>`_\ .

        Modified in accordance with Spring Framework 4.1

        * Removed the description about the problem where pessimistic locking error of JPA (Hibernate implementation) is not converted into \ ``PessimisticLockingFailureException``\  of Spring Framework.
          This problem is resolved in \ `SPR-10815 <https://jira.spring.io/browse/SPR-10815>`_\  (Spring Framework 4.0 and later versions).

        Modified in accordance with Apache Commons DBCP 2.0

        * Changed the sample code and its description to use component for Apache Commons DBCP 2.0.
    * -
      - :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessMyBatis3`
      - Added new

        * Added method to implement an infrastructure layer using MyBatis3 as O/R Mapper.
    * -
      - :doc:`../ArchitectureInDetail/DataAccessDetail/ExclusionControl`
      - Fixed guideline bugs

        * Modified the sample code of optimistic locking of long transactions (processing when records cannot be fetched).
          For modification details, refer to \ `guideline#450 issue <https://github.com/terasolunaorg/guideline/issues/450>`_\ .

        Modified in accordance with Spring Framework 4.1

        * Removed the description about the problem where pessimistic locking error of JPA (Hibernate implementation) is not converted into \ ``PessimisticLockingFailureException``\  of Spring Framework.
          This problem is resolved in \ `SPR-10815 <https://jira.spring.io/browse/SPR-10815>`_\  (Spring Framework 4.0 and later versions).

        Modified in accordance with MyBatis3

        * Added methods to implement exclusive control when using MyBatis3.
    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Validation`
      - Fixed guideline bugs

        * Modified the description of \ ``@GroupSequence``\ .
          For modification details, refer to \ `guideline#296 issue <https://github.com/terasolunaorg/guideline/issues/296>`_\ .

        Modified in accordance with bug fixes of common library

        * Added notes about \ ``ValidationMessages.properties``\ , in accordance with bug fixes of common library (\ `terasoluna-gfw#256 <https://github.com/terasolunaorg/terasoluna-gfw/issues/256>`_\ ).
          For modification details, refer to \ `guideline#766 issue <https://github.com/terasolunaorg/guideline/issues/766>`_\ .

        Added description

        * Added a method to link with the mechanism of Group Validation of Bean Validation at the time of correlated item check using Spring Validator.
          For added contents, refer to \ `guideline#320 issue <https://github.com/terasolunaorg/guideline/issues/320>`_\ .

        Modified in accordance with Bean Validation 1.1 (Hibernate Validator 5.1)

        * Added description about \ ``inclusive``\  attribute of \ ``@DecimalMin``\  and \ ``@DecimalMax``\ .
        * Added description about Expression Language.
        * Described about deprecated API from Bean Validation 1.1.
        * Added description about a bug related to \ ``ValidationMessages.properties``\  of Hibernate Validator 5.1.x (\ `HV-881 <https://hibernate.atlassian.net/browse/HV-881>`_\ ) and methods to prevent the same.
    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/ExceptionHandling`
      - Added description

        * Added a description that simple error page is likely to be displayed in Internet Explorer when an error having size lesser than 513 bytes is sent as response.
          For added contents, refer to \ `guideline#189 issue <https://github.com/terasolunaorg/guideline/issues/189>`_\ .

        Modified in accordance with Spring Framework 4.1

        * Removed the description about the problem where pessimistic locking error of JPA (Hibernate implementation) is not converted into \ ``PessimisticLockingFailureException``\  of Spring Framework.
          This problem is resolved in \ `SPR-10815 <https://jira.spring.io/browse/SPR-10815>`_\  (Spring Framework 4.0 and later versions).
    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/SessionManagement`
      - Modified in accordance with Spring Security 3.2

        * Removed the description about a problem where CSRF token error occurs (\ `SEC-2422 <https://jira.springsource.org/browse/SEC-2422>`_\  ) instead of session time out at the time of POST request.
          A mechanism to detect session time out is included in formal version of Spring Security 3.2, hence the problem is resolved.
    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/MessageManagement`
      - Reflected changes of common library

        * Added description about newly added message type (warning) and deprecated messages types (warn), in accordance with the improvement of common library (\ `terasoluna-gfw#24 <https://github.com/terasolunaorg/terasoluna-gfw/issues/24>`_\ ).
          For modification details, refer to \ `guideline#74 issue <https://github.com/terasolunaorg/guideline/issues/74>`_\ .
    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Pagination`
      - Reflected changes of common library

        * Changed description of page link in active state, in accordance with the improvement of common library (\ `terasoluna-gfw#13 <https://github.com/terasolunaorg/terasoluna-gfw/issues/13>`_\ ).
          For modification details, refer to \ `guideline#699 issue <https://github.com/terasolunaorg/guideline/issues/699>`_\ .
        * Changed description of page link in disabled state, in accordance with the improvement of common library (\ `terasoluna-gfw#14 <https://github.com/terasolunaorg/terasoluna-gfw/issues/14>`_\ ).
          For modification details, refer to \ `guideline#700 issue <https://github.com/terasolunaorg/guideline/issues/700>`_\ .

        Modified in accordance with Spring Data Common 1.9

        * Added notes for the classes where API specifications (\ ``Page``\  interface, etc.) are changed due to version upgrade.
    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Codelist`
      - Modified in accordance with bug fixes of common library

        * Added notes about version upgrade and changing message key of \ ``ExistInCodeList``\  in accordance with bug fixes of common library (\ `terasoluna-gfw#16 <https://github.com/terasolunaorg/terasoluna-gfw/issues/16>`_\ ).
          For modification details, refer to \ `guideline#638 issue <https://github.com/terasolunaorg/guideline/issues/638>`_\ .
        * Added notes about message definition of \ ``@ExistInCodeList``\  in accordance with bug fixes of common library (\ `terasoluna-gfw#256 <https://github.com/terasolunaorg/terasoluna-gfw/issues/256>`_\ ).
          For modification details, refer to \ `guideline#766 issue <https://github.com/terasolunaorg/guideline/issues/766>`_\ .

        Reflected changes of common library

        * Added a method to use \ ``EnumCodeList``\  class in accordance with addition of common library functions (\ `terasoluna-gfw#25 <https://github.com/terasolunaorg/terasoluna-gfw/issues/25>`_\ ).
    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Ajax`
      - Modified in accordance with Spring Security 3.2

        * Changed the sample code for CSRF measures (method to create \ ``<meta>``\  tag for CSRF measures).

        Modified in accordance with Jackson 2.4

        * Changed the sample code and description to use components for Jackson 2.4.
    * -
      - :doc:`../ArchitectureInDetail/WebServiceDetail/REST`
      - Improvement in description

        * Improve the method to build an URL to be set in location header and hypermedia link.
          For improvement details, refer to \ `guideline#374 issue <https://github.com/terasolunaorg/guideline/issues/374>`_\ .

        Modified in accordance with Spring Framework 4.1

        * Added a description about \ ``@RestController``\ .
          For modification details, refer to \ `guideline#560 issue <https://github.com/terasolunaorg/guideline/issues/560>`_\ .
        * Changed the sample code to create \ ``ResponseEntity``\  using builder style API.

        Modified in accordance with Jackson 2.4

        * Changed the sample code and description to use components for Jackson 2.4.

        Modified in accordance with Spring Data Common 1.9

        * Added notes for the classes where API specifications (\ ``Page``\   interface, etc.) are changed due to version upgrade.
    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/FileUpload`
      - Fixed guideline bugs

        * Modified version of Apache Commons FileUpload with resolved \ `CVE-2014-0050 <http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0050>`_\  (File Upload vulnerabilities).
          For modification details, refer to \ `guideline#846 issue <https://github.com/terasolunaorg/guideline/issues/846>`_\ .

        Added description

        * File upload function of Servlet 3 has a problem of garbled characters on a part of application server. Therefore, added a method to use Apache Commons FileUpload as a measure to prevent this event.
          For added contents, refer to \ `guideline#778 issue <https://github.com/terasolunaorg/guideline/issues/778>`_\ .
    * -
      - :doc:`../ArchitectureInDetail/GeneralFuncDetail/SystemDate`
      - Reflected changes of common library

        * Changed document structure, package name and class name in accordance with the improvement of common library (\ `terasoluna-gfw#224 <https://github.com/terasolunaorg/terasoluna-gfw/issues/224>`_\ ).
          For modification details, refer to \ `guideline#701 issue <https://github.com/terasolunaorg/guideline/issues/701>`_\ .
    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/TilesLayout`
      - Modified in accordance with Tiles 3.0

        * Changed the example of settings and description to use component for Tiles 3.0.

        Modified in accordance with Spring Framework 4.1

        * Added description about \ ``<mvc:view-resolvers>``\ , \ ``<mvc:tiles>``\ , \ ``<mvc:definitions>``\ .
          For modification details, refer to \ `guideline#609 issue <https://github.com/terasolunaorg/guideline/issues/609>`_\ .
    * -
      - :doc:`../ArchitectureInDetail/GeneralFuncDetail/JodaTime`
      - Added description

        * Added method to use \ ``LocalDateTime``\ .
          For added contents, refer to \ `guideline#584 issue <https://github.com/terasolunaorg/guideline/issues/584>`_\ .

        Modified in accordance with Joda Time 2.5

        * Since \ ``DateMidnight``\  class is deprecated in accordance with version upgrade, changed the method to fetch start time of specific date (0:00:00.000).
    * -
      - :doc:`../Security/SpringSecurity`
      - Modified in accordance with Spring Security 3.2

        * Added "Settings to create secure HTTP header" in appendix.
    * -
      - :doc:`../Tutorial/TutorialSecurity`
      - Updated in accordance with version 5.0.0

        * Made changes so as to use MyBatis3 as infrastructure layer.
        * Applied Spring Framework 4.1
        * Applied Spring Security 3.2
        * Revised document structure.
    * -
      - :doc:`../Security/Authentication`
      - Fixed guideline bugs

        * Modified the erroneous and inadequate description of \ ``<form-login>``\ , \ ``<logout>``\ , \ ``<session-management>``\  tag.
          For modification details, refer to \ `guideline#754 issue <https://github.com/terasolunaorg/guideline/issues/754>`_\ .
        * Modified the sample code that indicates extension method of AuthenticationFilter (added settings to validate CSRF measures and session fixation attack measures).
          For details, refer to \ `guideline#765 issue <https://github.com/terasolunaorg/guideline/issues/765>`_\ .

        Modified in accordance with Spring Security 3.2

        * Added notes about logout method when CSRF measures are validated.
        * Added description of \ ``@AuthenticationPrincipal``\ , as a method to access \ ``UserDetails``\  (authentication user information class) from Controller.
        * Added description of \ ``changeSessionId``\ , as parameters of \ ``session-fixation-protection``\  attribute of \ ``<sec:session-management>``\ .
        * Added methods to detect session time-out and notes for same.
        * Changed setting method to validate concurrent session control of identical users (made changes so as to use \ ``<sec:concurrency-control>``\ ).
        * Added a point that a class of concurrent session control of identical users is deprecated and other class is provided.
    * -
      - :doc:`../Security/CSRF`
      - Modified in accordance with Spring Security 3.2

        * Removed description about the component for CSRF measures of Spring Security 3.2.0 (provisional version before formal release) included in common library of version 1.0.x.
        * Changed setting method to validate CSRF measures by a proper method of Spring Security 3.2 (method using \ ``<sec:csrf>``\ ).
        * Added description about JSP tag library (\ ``<sec:csrfInput>``\  and \ ``<sec:csrfMetaTags>``\ ) for CSRF measures.
        * Added methods to detect session time-out and precautions when CSRF measures are validated.

        Modified in accordance with Spring Framework 4.1

        * Changed description about the condition where CSRF token is output as hidden, when \ ``<form:form>``\  is used.
    * -
      - :doc:`../Tutorial/TutorialREST`
      - Improved the description

        * Changed to the contents that do not depend on specific infrastructure layer (O/R Mapper), by adding REST API in the project created in \ :doc:`../Tutorial/TutorialTodo`\ .
          For modification details, refer to \ `guideline#325 issue <https://github.com/terasolunaorg/guideline/issues/325>`_\ .

        Updated in accordance with version 5.0.0

        * Applied Spring Framework 4.1.
        * Applied Spring Security 3.2.
        * Applied Jackson 2.4.
    * -
      - Create a new project from a blank project
      - Improved the description

        * Supported method to create a project having multi project structure.
        * Updated the method to create a project having single project structure.
    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/TagLibAndELFunctions`
      - Added new

        * Added description about EL functions and JSP tag libraries provided by common libraries.
    * -
      - :doc:`../Appendix/Lombok`
      - Added new

        * Added description about how to remove a boilerplate code where Lombok is used.
    * -
      - English version
      - Added English version of the following.

        * :doc:`../ImplementationAtEachLayer/CreateWebApplicationProject`
        * :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessCommon`
        * :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessJpa`
        * :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessMyBatis3`
        * :doc:`../ArchitectureInDetail/DataAccessDetail/ExclusionControl`
        * :doc:`../ArchitectureInDetail/GeneralFuncDetail/Logging`
        * :doc:`../ArchitectureInDetail/GeneralFuncDetail/PropertyManagement`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/Pagination`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/DoubleSubmitProtection`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/Internationalization`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/Codelist`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/Ajax`
        * :doc:`../ArchitectureInDetail/WebServiceDetail/REST`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/FileUpload`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/FileDownload`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/TilesLayout`
        * :doc:`../ArchitectureInDetail/GeneralFuncDetail/SystemDate`
        * :doc:`../ArchitectureInDetail/GeneralFuncDetail/Dozer`
        * :doc:`../Security/SpringSecurity`
        * :doc:`../Security/Authentication`
        * :doc:`../Security/Authorization`
        * :doc:`../Security/CSRF`
        * Create a new project from a blank project
        * :doc:`../Appendix/Nexus`
        * Elimination of environmental dependency
        * Project Structure Standard
        * :doc:`../Appendix/Lombok`
        * :doc:`../Appendix/SpringComprehensionCheck`
    * - 2014-08-27
      - \-
      - Released "1.0.1 RELEASE" version

        Refer to \ `Issue list of 1.0.1 <https://github.com/terasolunaorg/guideline/issues?labels=&milestone=1&state=closed>`_\  for details.
    * -
      - Overall modifications
      - Fixed guideline bugs (corrected typos, mistakes in description etc.)

        Refer to \ `Issue list of 1.0.1 (bug & clerical error) <https://github.com/terasolunaorg/guideline/issues?labels=bug&milestone=1&state=closed>`_\  for details.
    * -
      - Japanese version
      - Added Japanese version of the following.

        * :doc:`CriteriaBasedMapping`
        * :doc:`../ArchitectureInDetail/WebServiceDetail/REST`
        * :doc:`../Tutorial/TutorialREST`
    * -
      - English version
      - Added English version of the following.

        * :doc:`index`
        * :doc:`../Overview/index`
        * :doc:`../Tutorial/TutorialTodo`
        * :doc:`../ImplementationAtEachLayer/index`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/Validation`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/ExceptionHandling`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/MessageManagement`
        * :doc:`../ArchitectureInDetail/GeneralFuncDetail/JodaTime`
        * :doc:`../Security/XSS`
        * :doc:`../Appendix/ReferenceBooks`
    * -
      - :doc:`../Overview/FrameworkStack`
      - Updated the OSS version in accordance with bug fixes.

        * GroupId (\ ``org.springframework``\  ) updated to 3.2.10.RELEASE from 3.2.4.RELEASE
        * GroupId (\ ``org.springframework.data``\  )/ArtifactId(\ ``spring-data-commons``\  ) updated to 1.6.4.RELEASE from 1.6.1.RELEASE
        * GroupId (\ ``org.springframework.data``\  )/ArtifactId(\ ``spring-data-jpa``\  ) updated to 1.4.3.RELEASE from 1.4.1.RELEASE
        * GroupId (\ ``org.aspectj``\  ) updated to 1.7.4 from 1.7.3
        * Deleted GroupId (\ ``javax.transaction``\  )/ArtifactId(\ ``jta``\  )
    * -
      - :doc:`../ImplementationAtEachLayer/ApplicationLayer`
      - Added a warning about `CVE-2014-1904 <http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-1904>`_\  (XSS Vulnerability of \ ``action``\  attribute in \ ``<form:form>``\  tag)
    * -
      - Japanese version

        :doc:`../ArchitectureInDetail/WebApplicationDetail/MessageManagement`
      - Added description about bug fix

        * Fixed bugs of \ ``<t:messagesPanel>``\  tag of common library (\ `terasoluna-gfw#10 <https://github.com/terasolunaorg/terasoluna-gfw/issues/10>`_\ )
    * -
      - Japanese version

        :doc:`../ArchitectureInDetail/WebApplicationDetail/Pagination`
      - Updated description about bug fix

        * Fixed bugs of \ ``<t:pagination>``\  tag of common library (\ `terasoluna-gfw#12 <https://github.com/terasolunaorg/terasoluna-gfw/issues/12>`_\  )
        * Fixed bugs of Spring Data Commons (\ `terasoluna-gfw#22 <https://github.com/terasolunaorg/terasoluna-gfw/issues/22>`_\  )
    * -
      - Japanese version

        :doc:`../ArchitectureInDetail/WebApplicationDetail/Ajax`
      - Updated description of countermeasures against XXE Injection
    * -
      - Japanese version

        :doc:`../ArchitectureInDetail/WebApplicationDetail/FileUpload`
      - Added a warning about `CVE-2014-0050 <http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0050>`_\  (File Upload Vulnerability)

        Fixed guideline bugs.

        * Added how to handle \ ``MultipartException``\  using error-page functionality of servlet container, because your application can't handle \ ``MultipartException``\  using \ ``SystemExceptionResolver``\  when used \ ``MultipartFilter``\ . Refer to \ `Issue of guideline#59 <https://github.com/terasolunaorg/guideline/issues/59>`_\  for details.
    * -
      - Japanese version
      - Change how to create following projects to be carried out from \ ``mvn archetype:generate``\

        * :doc:`../Overview/FirstApplication`
        * :doc:`../Tutorial/TutorialTodo`
        * :doc:`../Tutorial/TutorialTodo`
    * -
      - Japanese version
      - Minor modifications in how to create following Maven archetype

        * :doc:`../Tutorial/TutorialSecurity`
        * Create a new project from a blank project
    * - 2013-12-17
      - Japanese version
      - Released "1.0.0 Public Review" version

.. raw:: latex

   \newpage

