Change Log
================================================================================

.. tabularcolumns:: |p{0.15\linewidth}|p{0.25\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 15 25 60

    * - Modified on
      - Modified locations
      - Modification details
    * - 2015-02-xx
      - \-
      - Released "1.0.2 RELEASE" version

        For details, refer to \ `issue list of 1.0.2 <https://github.com/terasolunaorg/guideline/issues?q=is%3Aissue+milestone%3A1.0.2+is%3Aclosed>`_\ .
    * -
      - Overall
      - Fixed clerical error(corrected typos, mistakes in description etc.) of guideline

        * For details, refer to \ `issue list(clerical error) of 1.0.2 <https://github.com/terasolunaorg/guideline/issues?q=is%3Aissue+milestone%3A1.0.2+is%3Aclosed+label%3A%22clerical+error%22>`_\ .

        Improved the description

        * For details, refer to \ `issue list(improvement) of 1.0.2 <https://github.com/terasolunaorg/guideline/issues?q=is%3Aissue+milestone%3A1.0.2+label%3Aimprovement+is%3Aclosed>`_\ .

        Added new contents

        * :doc:`../Appendix/TagLibAndELFunctions`
    * -
      - :doc:`../Overview/FrameworkStack`
      - Updated the OSS version in accordance with Spring Framework bug fix(security vulnerability)

        * GroupId (\ ``org.springframework``\ ) updated to 3.2.13.RELEASE from 3.2.10.RELEASE.
    * -
      - :doc:`../Overview/ApplicationLayering`
      - Fixed error of english translation

        * Fixed a english translation error about relationship with domain layer and other layer.
          For detail, refer to \ `Issue of guideline#364 <https://github.com/terasolunaorg/guideline/issues/364>`_\ .
    * -
      - :doc:`../ArchitectureInDetail/DataAccessCommon`
      - Updated description about common library bug fix

        * Added a description about handling full width wildcard character(\ ``％``\ , \ ``＿``\ ) in accordance with common library bug fix(\ `terasoluna-gfw#78 <https://github.com/terasolunaorg/terasoluna-gfw/issues/78>`_\).
          For detail, refer to \ `Issue of guideline#712 <https://github.com/terasolunaorg/guideline/issues/712>`_\ .
    * -
      - :doc:`../ArchitectureInDetail/DataAccessMybatis2`
      - Fixed error of guideline

        * Fixed a setting and description for handling the LOB type.
          For detail, refer to \ `Issue of guideline#402 <https://github.com/terasolunaorg/guideline/issues/402>`_\ .
    * -
      - :doc:`../ArchitectureInDetail/ExclusionControl`
      - Fixed error of guideline

        * Fixed a sample code (processing when the record can not be obtained) for perform optimistic lock of long transaction.
          For detail, refer to \ `Issue of guideline#450 <https://github.com/terasolunaorg/guideline/issues/450>`_\ .
    * -
      - :doc:`../ArchitectureInDetail/Validation`
      - Fixed error of guideline

        * Fixed a description of \ ``@GroupSequence``\ .
          For detail, refer to \ `Issue of guideline#296 <https://github.com/terasolunaorg/guideline/issues/296>`_\ .

        Updated description about common library bug fix

        * Added a note about \ ``ValidationMessages.properties``\  in accordance with common library bug fix(\ `terasoluna-gfw#256 <https://github.com/terasolunaorg/terasoluna-gfw/issues/256>`_\).
          For detail, refer to \ `Issue of guideline#766 <https://github.com/terasolunaorg/guideline/issues/766>`_\ .

        Added description

        * Added a way that combine with the Group Validation mechanism of Bean Validation when perform the correlation item check using the Spring Validator.
          For detail, refer to \ `Issue of guideline#320 <https://github.com/terasolunaorg/guideline/issues/320>`_\ .
    * -
      - :doc:`../ArchitectureInDetail/ExceptionHandling`
      - Added description

        * Added a description that there is a possibility that as error page is displayed the Internet Explorer simple error page if response data size is less than 513 bytes.
          For detail, refer to \ `Issue of guideline#189 <https://github.com/terasolunaorg/guideline/issues/189>`_\ .
    * -
      - :doc:`../ArchitectureInDetail/Codelist`
      - Updated description about common library bug fix

        * Added a note about message definition of \ ``@ExistInCodeList``\  in accordance with common library bug fix(\ `terasoluna-gfw#256 <https://github.com/terasolunaorg/terasoluna-gfw/issues/256>`_\).
          For detail, refer to \ `Issue of guideline#766 <https://github.com/terasolunaorg/guideline/issues/766>`_\ .
    * -
      - :doc:`../ArchitectureInDetail/Ajax`
      - Improve description

        * Fixed a sample code of CSRF countermeasure (setting value of \ ``name``\  attribute of \ ``<meta>``\  tag) to hold the compatibility with Spring Security 3.2(used by version 5.0.0).
          For detail, refer to \ `Issue of guideline#680 <https://github.com/terasolunaorg/guideline/issues/680>`_\ .
    * -
      - :doc:`../ArchitectureInDetail/REST`
      - Improve description

        * Improved the way to build the URL for set to the Location header or hypermedia links.
          For detail, refer to \ `Issue of guideline#374 <https://github.com/terasolunaorg/guideline/issues/374>`_\ .
    * -
      - :doc:`../ArchitectureInDetail/FileUpload`
      - Fixed error of guideline

        * Fixed the version of Apache Commons FileUpload where \ `CVE-2014-0050 <http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0050>`_\ (File Upload Vulnerability) has been resolved.
          For detail, refer to \ `Issue of guideline#846 <https://github.com/terasolunaorg/guideline/issues/846>`_\ .

        Added description

        * Added a way to use the Apache Commons Fileupload as workaround, because request data is garbled in parts of application server when use the servlet 3's file upload functionality.
          For detail, refer to \ `Issue of guideline#778 <https://github.com/terasolunaorg/guideline/issues/778>`_\ .
    * -
      - :doc:`../ArchitectureInDetail/Utilities/JodaTime`
      - Added description

        * Added a usage of \ ``LocalDateTime``\ .
          For detail, refer to \ `Issue of guideline#584 <https://github.com/terasolunaorg/guideline/issues/584>`_\ .
    * -
      - :doc:`../Security/Authentication`
      - Fixed error of guideline

        * Fixed a wrong description of \ ``<form-login>``\ , \ ``<logout>``\  and \ ``<session-management>``\ .
          For details, refer to \ `Issue of guideline#754 <https://github.com/terasolunaorg/guideline/issues/754>`_\ .
        * Fixed a sample code for indicate how to extend the \ ``AuthenticationFilter``\ (settings of Session fixation protection countermeasure and CSRF countermeasure).
          For detail, refer to \ `Issue of guideline#765 <https://github.com/terasolunaorg/guideline/issues/765>`_\ .
    * -
      - :doc:`../Appendix/TagLibAndELFunctions`
      - Added new contents

        * Added a description of JSP tag library and EL functions provided by common library.
    * -
      - English version
      - Added English version the follows:

        * :doc:`../ArchitectureInDetail/DataAccessCommon`
        * :doc:`../ArchitectureInDetail/DataAccessJpa`
        * :doc:`../ArchitectureInDetail/DataAccessMybatis2`
        * :doc:`../ArchitectureInDetail/ExclusionControl`
        * :doc:`../ArchitectureInDetail/Logging`
        * :doc:`../ArchitectureInDetail/PropertyManagement`
        * :doc:`../ArchitectureInDetail/Pagination`
        * :doc:`../ArchitectureInDetail/DoubleSubmitProtection`
        * :doc:`../ArchitectureInDetail/Internationalization`
        * :doc:`../ArchitectureInDetail/Codelist`
        * :doc:`../ArchitectureInDetail/Ajax`
        * :doc:`../ArchitectureInDetail/REST`
        * :doc:`../ArchitectureInDetail/FileUpload`
        * :doc:`../ArchitectureInDetail/FileDownload`
        * :doc:`../ArchitectureInDetail/TilesLayout`
        * :doc:`../ArchitectureInDetail/SystemDate`
        * :doc:`../ArchitectureInDetail/Utilities/Dozer`
        * :doc:`../Security/SpringSecurity`
        * :doc:`../Security/PasswordHashing`
        * :doc:`../Security/Authorization`
        * :doc:`../Appendix/CreateProjectFromBlank`
        * :doc:`../Appendix/Nexus`
        * :doc:`../Appendix/EnvironmentIndependency`
        * :doc:`../Appendix/ProjectStructureStandard`
        * :doc:`../Appendix/SpringComprehensionCheck`
    * - 2014-08-27
      - \-
      - Released "1.0.1 RELEASE" version
        
        For details, refer to \ `Issue list of 1.0.1 <https://github.com/terasolunaorg/guideline/issues?labels=&milestone=1&state=closed>`_\ .
    * - 
      - Overall modifications
      - Fixed guideline errors (corrected typos, mistakes in description etc.)

        For details, refer to \ `Issue list of 1.0.1 <https://github.com/terasolunaorg/guideline/issues?labels=bug&milestone=1&state=closed>`_\ .
    * - 
      - Japanese version
      - Added Japanese version of the following.
      
        * :doc:`CriteriaBasedMapping`
        * :doc:`../ArchitectureInDetail/REST`
        * :doc:`Tutorial (Todo Application for REST)<../TutorialREST/index>`
    * - 
      - English version
      - Added English version of the following.
      
        * :doc:`index`
        * :doc:`../Overview/index`
        * :doc:`../TutorialTodo/index`
        * :doc:`../ImplementationAtEachLayer/index`
        * :doc:`../ArchitectureInDetail/Validation`
        * :doc:`../ArchitectureInDetail/ExceptionHandling`
        * :doc:`../ArchitectureInDetail/MessageManagement`
        * :doc:`../ArchitectureInDetail/Utilities/JodaTime`
        * :doc:`../Security/XSS`
        * :doc:`../Appendix/ReferenceBooks`
    * - 
      - :doc:`../Overview/FrameworkStack`
      - Updated the OSS version in accordance with bug fixes.
      
        * GroupId (\ ``org.springframework``\ ) updated to 3.2.10.RELEASE from 3.2.4.RELEASE
        * GroupId (\ ``org.springframework.data``\ )/ArtifactId(\ ``spring-data-commons``\ ) updated to 1.6.4.RELEASE from 1.6.1.RELEASE
        * GroupId (\ ``org.springframework.data``\ )/ArtifactId(\ ``spring-data-jpa``\ ) updated to 1.4.3.RELEASE from 1.4.1.RELEASE
        * GroupId (\ ``org.aspectj``\ ) updated to 1.7.4 from 1.7.3
        * Deleted GroupId (\ ``javax.transaction``\ )/ArtifactId(\ ``jta``\ )
    * - 
      - :doc:`../ImplementationAtEachLayer/ApplicationLayer`
      - Added a warning about `CVE-2014-1904 <http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-1904>`_\ (XSS Vulnerability of \ ``action``\  attribute in \ ``<form:form>``\  tag)
    * - 
      - Japanese version
      
        :doc:`../ArchitectureInDetail/MessageManagement`
      - Added description about bug fix
      
        * Fixed bugs of \ ``<t:messagesPanel>``\  tag of common library (\ `terasoluna-gfw#10 <https://github.com/terasolunaorg/terasoluna-gfw/issues/10>`_\ )
    * - 
      - Japanese version
      
        :doc:`../ArchitectureInDetail/Pagination`
      - Updated description about bug fix
      
        * Fixed bugs of \ ``<t:pagination>``\  tag of common library (\ `terasoluna-gfw#12 <https://github.com/terasolunaorg/terasoluna-gfw/issues/12>`_\ )
        * Fixed bugs of Spring Data Commons (\ `terasoluna-gfw#22 <https://github.com/terasolunaorg/terasoluna-gfw/issues/22>`_\ )
    * - 
      - Japanese version
      
        :doc:`../ArchitectureInDetail/Ajax`
      - Updated description of countermeasures against XXE Injection
    * - 
      - Japanese version
      
        :doc:`../ArchitectureInDetail/FileUpload`
      - Added a warning about `CVE-2014-0050 <http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0050>`_\ (File Upload Vulnerability)
      
        Fixed guideline errors.
        
        * Added how to handle \ ``MultipartException``\  using error-page functionality of servlet container, because your application can't handle \ ``MultipartException``\  using \ ``SystemExceptionResolver``\  when used \ ``MultipartFilter``\.
          For details, refer to \ `Issue of guideline#59 <https://github.com/terasolunaorg/guideline/issues/59>`_\ .
    * - 2013-12-17
      - Japanese version
      - Released "1.0.0 Public Review" version

.. raw:: latex

   \newpage

