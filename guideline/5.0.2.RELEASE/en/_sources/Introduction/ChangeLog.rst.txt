Change Log
================================================================================

.. tabularcolumns:: |p{0.15\linewidth}|p{0.25\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 15 25 60

    * - Modified on
      - Modified locations
      - Modification details
    * - 2016-02-24
      - \-
      - 5.0.2 RELEASE version published

        * For details of change contents, refer \ `5.0.2 Issue List <https://github.com/terasolunaorg/guideline/issues?q=is%3Aissue+milestone%3A5.0.2+is%3Aclosed>`_\ .
    * -
      - General
      - Correction of errors in the guideline (typo mistakes and simple description errors)

        Description details modified

        * For details of modification, refer \ `5.0.2 Issue list (improvement) <https://github.com/terasolunaorg/guideline/issues?q=milestone%3A5.0.2+label%3Aimprovement+is%3Aclosed>`_\ .

    * -
      - :doc:`index`
      - Description details added

        * Description related to operation verification environment of the details described in the guideline added

    * -
      - :doc:`../Overview/FrameworkStack`
      - OSS version to be used (Spring IO Platform version) updated

        * Spring IO Platform version updated in 1.1.5.RELEASE
        * Spring Framework version updated in 4.1.9.RELEASE
        * Spring Security version updated in 3.2.9.RELEASE

        OSS version to be used along with Spring IO Platform version update is updated

        * OSS version to be used updated. For update details, refer \ `version 5.0.2 migration guide <https://github.com/terasolunaorg/terasoluna-gfw/wiki/Migration-Guide-5.0.2_ja#step-1-update-dependency-libraries>`_\ .

    * -

      - :doc:`../ImplementationAtEachLayer/CreateWebApplicationProject`
      - Description details modified

        * A method wherein SQL is executed by using \ ``SQL Maven Plugin``\  is added (\ `guideline#1428 <https://github.com/terasolunaorg/guideline/issues/1428>`_\ ) 

    * -
      - :doc:`../ArchitectureInDetail/DataAccessCommon`
      - Description details added

        *  Precautions for \ ``Log4jdbcProxyDataSource``\  overhead added (\ `guideline#1471 <https://github.com/terasolunaorg/guideline/issues/1471>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/DataAccessMyBatis3`
      - Description details corresponding to MyBatis 3.3 added

        * "Changed the default at the time of delayed reading to \ ``JAVASSIST``\ " added (\ `guideline#1384 <https://github.com/terasolunaorg/guideline/issues/1384>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/DataAccessJpa`
      - Bug correction for the guideline

        *  Utility which use Like condition modified appropriately (\ `guideline#1464 <https://github.com/terasolunaorg/guideline/issues/1464>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/Validation`
      - Description details modified

        * Added that "Index position of attribute value" for input validation error message is in alphabetical order" (\ `guideline#1296 <https://github.com/terasolunaorg/guideline/issues/1296>`_\ ) 

    * -
      - :doc:`../ArchitectureInDetail/Logging`
      - Description details modified

        * Description where \ ``ServiceLoader``\  mechanism is used in Logback setting, is added (\ `guideline#1275 <https://github.com/terasolunaorg/guideline/issues/1275>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/Internationalization`
      - Description details modified

        *  Description for appropriately reflecting locale in JSP is added (\ `guideline#1439 <https://github.com/terasolunaorg/guideline/issues/1439>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/Codelist`
      - Description details added

        *  Description which recommends a pattern wherein \ ``JdbcTemplate``\  is specified in JdbcCodeList, is added (\ `guideline#501 <https://github.com/terasolunaorg/guideline/issues/501>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/FileUpload`
      - Description details modified

        * Basic flow of uploading process and its description modified to description which use \ ``MultipartFilter``\  of Spring (\ `guideline#193 <https://github.com/terasolunaorg/guideline/issues/193>`_\ )
        * "A method which sends CSRF token by query parameter" deleted due to issues like security issues, variation in the operation according to AP server etc.
          Precaution - "when allowable size for file upload exceeds, CSRF token check is not carried out appropriately in some AP servers" added (\ `guideline#1602 <https://github.com/terasolunaorg/guideline/issues/1602>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/FileDownload`
      - Description details modified

        * Source example which use \ ``com.lowagie:itext:4.2.1``\  modified to the format which use \ ``com.lowagie:itext:2.1.7``\ for specification change in iText (\ `guideline#1310 <https://github.com/terasolunaorg/guideline/issues/1310>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/REST`  
      - Description details modified

        * Added creation of ObjectMapper which use \ ``Jackson2ObjectMapperFactoryBean``\  (\ `guideline#1022 <https://github.com/terasolunaorg/guideline/issues/1022>`_\ )  
        *  Modified to a format wherein MyBatis3 is considered as a prerequisite in the implementation of domain layer of REST API application (\ `guideline#1323 <https://github.com/terasolunaorg/guideline/issues/1323>`_\ )  

    * -
      - :doc:`../Security/Tutorial`  
      - Bug correction in the guideline

        * Corrected places in the source code where logout was not possible (\ `guideline#1300 <https://github.com/terasolunaorg/guideline/issues/1300>`_\ )  

    * -
      - :doc:`../Security/Authentication`  
      - Description details modified

        * \ ``RedirectAuthenticationHandler``\ package corrected (\ `guideline#1482 <https://github.com/terasolunaorg/guideline/issues/1482>`_\ )  

    * -
      - :doc:`../Security/PasswordHashing`  
      - Bug correction in the guideline

        * VM argument which resolves delay issues associated with SecureRandom is corrected for errors (\ `guideline#1502 <https://github.com/terasolunaorg/guideline/issues/1502>`_\ )  

    * -
      - :doc:`../Security/CSRF`  
      - Description details modified

        * Items related to multi-part request moved in :doc:`../ArchitectureInDetail/FileUpload` (\ `guideline#1602 <https://github.com/terasolunaorg/guideline/issues/1602>`_\ )  

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
      - :doc:`../ArchitectureInDetail/DataAccessCommon`
      - Added the description

        * Added the description of \ ``DataSource``\  switching functionality (\ `guideline#1071 <https://github.com/terasolunaorg/guideline/issues/1071>`_\ )
    * -
      - :doc:`../ArchitectureInDetail/DataAccessMyBatis3`
      - Fixed the guideline bug

        * Modified the description about timing of batch execution (\ `guideline#903 <https://github.com/terasolunaorg/guideline/issues/903>`_\ )
    * -
      - :doc:`../ArchitectureInDetail/Logging`
      - Improved the description

        * Added the description about \ ``additivity``\  attribute of \ ``<logger>``\  tag (\ `guideline#977 <https://github.com/terasolunaorg/guideline/issues/977>`_\ )
    * -
      - :doc:`../ArchitectureInDetail/SessionManagement`
      - Improved the description

        * Modified the description about how to define a session scope bean (\ `guideline#1082 <https://github.com/terasolunaorg/guideline/issues/1082>`_\ )
    * -
      - :doc:`../ArchitectureInDetail/DoubleSubmitProtection`
      - Added the description

        * Added the description about the transaction token check in case that response cache is disabled (\ `guideline#1260 <https://github.com/terasolunaorg/guideline/issues/1260>`_\ )
    * -
      - :doc:`../ArchitectureInDetail/Codelist`
      - Added the description

        * Added how to display a code name (\ `guideline#1109 <https://github.com/terasolunaorg/guideline/issues/1109>`_\ )
    * -
      - | :doc:`../ArchitectureInDetail/Ajax`
        | :doc:`../ArchitectureInDetail/REST`
      - Added the warning about \ `CVE-2015-3192 <http://pivotal.io/security/cve-2015-3192>`_\ (XML security vulnerability)

        * Added the warning at the time of the StAX(Streaming API for XML) use (\ `guideline#1211 <https://github.com/terasolunaorg/guideline/issues/1211>`_\ )
    * -
      - | :doc:`../ArchitectureInDetail/Pagination`
        | :doc:`../Appendix/TagLibAndELFunctions`
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
      - :doc:`../Appendix/EnvironmentIndependency`
      - Added the description

        * Added how to apply the external classpath(alternative functionality of \ ``VirtualWebappLoader``\  of Tomcat7) at the time of Tomcat8 use (\ `guideline#1081 <https://github.com/terasolunaorg/guideline/issues/1081>`_\ )
    * - 2015-06-12
      - Overall modifications
      - Released English version of "5.0.0 RELEASE"
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
        * :doc:`../ArchitectureInDetail/DataAccessMyBatis3`
        * :doc:`../Appendix/TagLibAndELFunctions`
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
      - :doc:`../TutorialTodo/index`
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
      - :doc:`../ArchitectureInDetail/DataAccessCommon`
      - Modified in accordance with bug fixes of common library

        * Added description about handling double byte wild card characters (\ ``％``\ , \ ``＿``\)\ , in accordance with bug fixes of common library (\ `terasoluna-gfw#78 <https://github.com/terasolunaorg/terasoluna-gfw/issues/78>`_\ ).
          For modification details, refer to \ `guideline#712 issue <https://github.com/terasolunaorg/guideline/issues/712>`_\ .

        Modified in accordance with Spring Framework 4.1

        * Removed the description about the problem where pessimistic locking error of JPA (Hibernate implementation) is not converted into \ ``PessimisticLockingFailureException``\  of Spring Framework.
          This problem is resolved in \ `SPR-10815 <https://jira.spring.io/browse/SPR-10815>`_\  (Spring Framework 4.0 and later versions).

        Modified in accordance with Apache Commons DBCP 2.0

        * Changed the sample code and its description to use component for Apache Commons DBCP 2.0.
    * -
      - :doc:`../ArchitectureInDetail/DataAccessMyBatis3`
      - Added new

        * Added method to implement an infrastructure layer using MyBatis3 as O/R Mapper.
    * -
      - :doc:`../ArchitectureInDetail/ExclusionControl`
      - Fixed guideline bugs

        * Modified the sample code of optimistic locking of long transactions (processing when records cannot be fetched).
          For modification details, refer to \ `guideline#450 issue <https://github.com/terasolunaorg/guideline/issues/450>`_\ .

        Modified in accordance with Spring Framework 4.1

        * Removed the description about the problem where pessimistic locking error of JPA (Hibernate implementation) is not converted into \ ``PessimisticLockingFailureException``\  of Spring Framework.
          This problem is resolved in \ `SPR-10815 <https://jira.spring.io/browse/SPR-10815>`_\  (Spring Framework 4.0 and later versions).

        Modified in accordance with MyBatis3

        * Added methods to implement exclusive control when using MyBatis3.
    * -
      - :doc:`../ArchitectureInDetail/Validation`
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
      - :doc:`../ArchitectureInDetail/ExceptionHandling`
      - Added description

        * Added a description that simple error page is likely to be displayed in Internet Explorer when an error having size lesser than 513 bytes is sent as response.
          For added contents, refer to \ `guideline#189 issue <https://github.com/terasolunaorg/guideline/issues/189>`_\ .

        Modified in accordance with Spring Framework 4.1

        * Removed the description about the problem where pessimistic locking error of JPA (Hibernate implementation) is not converted into \ ``PessimisticLockingFailureException``\  of Spring Framework.
          This problem is resolved in \ `SPR-10815 <https://jira.spring.io/browse/SPR-10815>`_\  (Spring Framework 4.0 and later versions).
    * -
      - :doc:`../ArchitectureInDetail/SessionManagement`
      - Modified in accordance with Spring Security 3.2

        * Removed the description about a problem where CSRF token error occurs (\ `SEC-2422 <https://jira.springsource.org/browse/SEC-2422>`_\  ) instead of session time out at the time of POST request.
          A mechanism to detect session time out is included in formal version of Spring Security 3.2, hence the problem is resolved.
    * -
      - :doc:`../ArchitectureInDetail/MessageManagement`
      - Reflected changes of common library

        * Added description about newly added message type (warning) and deprecated messages types (warn), in accordance with the improvement of common library (\ `terasoluna-gfw#24 <https://github.com/terasolunaorg/terasoluna-gfw/issues/24>`_\ ).
          For modification details, refer to \ `guideline#74 issue <https://github.com/terasolunaorg/guideline/issues/74>`_\ .
    * -
      - :doc:`../ArchitectureInDetail/Pagination`
      - Reflected changes of common library

        * Changed description of page link in active state, in accordance with the improvement of common library (\ `terasoluna-gfw#13 <https://github.com/terasolunaorg/terasoluna-gfw/issues/13>`_\ ).
          For modification details, refer to \ `guideline#699 issue <https://github.com/terasolunaorg/guideline/issues/699>`_\ .
        * Changed description of page link in disabled state, in accordance with the improvement of common library (\ `terasoluna-gfw#14 <https://github.com/terasolunaorg/terasoluna-gfw/issues/14>`_\ ).
          For modification details, refer to \ `guideline#700 issue <https://github.com/terasolunaorg/guideline/issues/700>`_\ .

        Modified in accordance with Spring Data Common 1.9

        * Added notes for the classes where API specifications (\ ``Page``\  interface, etc.) are changed due to version upgrade.
    * -
      - :doc:`../ArchitectureInDetail/Codelist`
      - Modified in accordance with bug fixes of common library

        * Added notes about version upgrade and changing message key of \ ``ExistInCodeList``\  in accordance with bug fixes of common library (\ `terasoluna-gfw#16 <https://github.com/terasolunaorg/terasoluna-gfw/issues/16>`_\ ).
          For modification details, refer to \ `guideline#638 issue <https://github.com/terasolunaorg/guideline/issues/638>`_\ .
        * Added notes about message definition of \ ``@ExistInCodeList``\  in accordance with bug fixes of common library (\ `terasoluna-gfw#256 <https://github.com/terasolunaorg/terasoluna-gfw/issues/256>`_\ ).
          For modification details, refer to \ `guideline#766 issue <https://github.com/terasolunaorg/guideline/issues/766>`_\ .

        Reflected changes of common library

        * Added a method to use \ ``EnumCodeList``\  class in accordance with addition of common library functions (\ `terasoluna-gfw#25 <https://github.com/terasolunaorg/terasoluna-gfw/issues/25>`_\ ).
    * -
      - :doc:`../ArchitectureInDetail/Ajax`
      - Modified in accordance with Spring Security 3.2

        * Changed the sample code for CSRF measures (method to create \ ``<meta>``\  tag for CSRF measures).

        Modified in accordance with Jackson 2.4

        * Changed the sample code and description to use components for Jackson 2.4.
    * -
      - :doc:`../ArchitectureInDetail/REST`
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
      - :doc:`../ArchitectureInDetail/FileUpload`
      - Fixed guideline bugs

        * Modified version of Apache Commons FileUpload with resolved \ `CVE-2014-0050 <http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0050>`_\  (File Upload vulnerabilities).
          For modification details, refer to \ `guideline#846 issue <https://github.com/terasolunaorg/guideline/issues/846>`_\ .

        Added description

        * File upload function of Servlet 3 has a problem of garbled characters on a part of application server. Therefore, added a method to use Apache Commons FileUpload as a measure to prevent this event.
          For added contents, refer to \ `guideline#778 issue <https://github.com/terasolunaorg/guideline/issues/778>`_\ .
    * -
      - :doc:`../ArchitectureInDetail/SystemDate`
      - Reflected changes of common library

        * Changed document structure, package name and class name in accordance with the improvement of common library (\ `terasoluna-gfw#224 <https://github.com/terasolunaorg/terasoluna-gfw/issues/224>`_\ ).
          For modification details, refer to \ `guideline#701 issue <https://github.com/terasolunaorg/guideline/issues/701>`_\ .
    * -
      - :doc:`../ArchitectureInDetail/TilesLayout`
      - Modified in accordance with Tiles 3.0

        * Changed the example of settings and description to use component for Tiles 3.0.

        Modified in accordance with Spring Framework 4.1

        * Added description about \ ``<mvc:view-resolvers>``\ , \ ``<mvc:tiles>``\ , \ ``<mvc:definitions>``\ .
          For modification details, refer to \ `guideline#609 issue <https://github.com/terasolunaorg/guideline/issues/609>`_\ .
    * -
      - :doc:`../ArchitectureInDetail/Utilities/JodaTime`
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
      - :doc:`../Security/Tutorial`
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
      - :doc:`../TutorialREST/index`
      - Improved the description

        * Changed to the contents that do not depend on specific infrastructure layer (O/R Mapper), by adding REST API in the project created in \ :doc:`../TutorialTodo/index`\ .
          For modification details, refer to \ `guideline#325 issue <https://github.com/terasolunaorg/guideline/issues/325>`_\ .

        Updated in accordance with version 5.0.0

        * Applied Spring Framework 4.1.
        * Applied Spring Security 3.2.
        * Applied Jackson 2.4.
    * -
      - :doc:`../Appendix/CreateProjectFromBlank`
      - Improved the description

        * Supported method to create a project having multi project structure.
        * Updated the method to create a project having single project structure.
    * -
      - :doc:`../Appendix/TagLibAndELFunctions`
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
        * :doc:`../ArchitectureInDetail/DataAccessCommon`
        * :doc:`../ArchitectureInDetail/DataAccessJpa`
        * :doc:`../ArchitectureInDetail/DataAccessMyBatis3`
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
        * :doc:`../Security/Authentication`
        * :doc:`../Security/PasswordHashing`
        * :doc:`../Security/Authorization`
        * :doc:`../Security/CSRF`
        * :doc:`../Appendix/CreateProjectFromBlank`
        * :doc:`../Appendix/Nexus`
        * :doc:`../Appendix/EnvironmentIndependency`
        * :doc:`../Appendix/ProjectStructureStandard`
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
      
        :doc:`../ArchitectureInDetail/MessageManagement`
      - Added description about bug fix
      
        * Fixed bugs of \ ``<t:messagesPanel>``\  tag of common library (\ `terasoluna-gfw#10 <https://github.com/terasolunaorg/terasoluna-gfw/issues/10>`_\ )
    * - 
      - Japanese version
      
        :doc:`../ArchitectureInDetail/Pagination`
      - Updated description about bug fix
      
        * Fixed bugs of \ ``<t:pagination>``\  tag of common library (\ `terasoluna-gfw#12 <https://github.com/terasolunaorg/terasoluna-gfw/issues/12>`_\  )
        * Fixed bugs of Spring Data Commons (\ `terasoluna-gfw#22 <https://github.com/terasolunaorg/terasoluna-gfw/issues/22>`_\  )
    * - 
      - Japanese version
      
        :doc:`../ArchitectureInDetail/Ajax`
      - Updated description of countermeasures against XXE Injection
    * - 
      - Japanese version
      
        :doc:`../ArchitectureInDetail/FileUpload`
      - Added a warning about `CVE-2014-0050 <http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0050>`_\  (File Upload Vulnerability)
      
        Fixed guideline bugs.
        
        * Added how to handle \ ``MultipartException``\  using error-page functionality of servlet container, because your application can't handle \ ``MultipartException``\  using \ ``SystemExceptionResolver``\  when used \ ``MultipartFilter``\  .
          Refer to \ `Issue of guideline#59 <https://github.com/terasolunaorg/guideline/issues/59>`_\  for details.
    * - 2013-12-17
      - Japanese version
      - Released "1.0.0 Public Review" version

.. raw:: latex

   \newpage

