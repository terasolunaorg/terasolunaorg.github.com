Change Log
================================================================================

.. tabularcolumns:: |p{0.15\linewidth}|p{0.25\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 15 25 60

    * - Modified on
      - Modified locations
      - Modification details
    * - 2014-08-xx
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
        * :doc:`../ArchitectureInDetail/utilities/JodaTime`
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

