Criteria based mapping of guideline
================================================================================
The chapter 4 of this guideline is structured functionality wise. 
This section shows a mapping from a point of view other than functionality. It indicates which 
part of guideline contains which type of content. 

Mapping based on security measures
--------------------------------------------------------------------------------

A point of view by OWASP(Open Web Application Security Project)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Using \ `OWASP Top 10 for 2013 <https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project>`_\  as an axis, 
links to explanation of functionalities related to security have been given


.. tabularcolumns:: |p{0.10\linewidth}|p{0.40\linewidth}|p{0.50\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 40 50
   :class: longtable

   * - Sr. No.
     - Item Name
     - Corresponding Guideline
   * - A1
     - `Injection <https://www.owasp.org/index.php/Top_10_2013-A1-Injection>`_ SQL Injection
     - * \ :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessMyBatis3`\
       * \ :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessJpa`\

       (Details about using bind variable at the time of placeholders for query parameters)
   * - A1
     - `Injection <https://www.owasp.org/index.php/Top_10_2013-A1-Injection>`_ XXE(XML External Entity) Injection
     - * \ :doc:`../ArchitectureInDetail/WebApplicationDetail/Ajax`\  
   * - A1  
     - `Injection <https://www.owasp.org/index.php/Top_10_2013-A1-Injection>`_ OS Command Injection  
     - * \ :doc:`../ArchitectureInDetail/WebApplicationDetail/Validation`\  
   * - A1  
     - `Injection <https://www.owasp.org/index.php/Top_10_2013-A1-Injection>`_ Email Header Injection  
     - * \ :ref:`email-header-injection`\  
   * - A1  
     - `Injection <https://www.owasp.org/index.php/Top_10_2013-A1-Injection>`_  
     - * \ :ref:`secure-input-validation`\  

       (Shows how to validate input values)  
   * - A2
     - `Broken Authentication and Session Management <https://www.owasp.org/index.php/Top_10_2013-A2-Broken_Authentication_and_Session_Management>`_
     - * \ :doc:`../Security/SessionManagement`\ 
       * \ :doc:`../Security/Authentication`\ 
   * - A3
     - `Cross-Site Scripting (XSS) <https://www.owasp.org/index.php/Top_10_2013-A3-Cross-Site_Scripting_(XSS)>`_
     - * \ :doc:`../Security/XSS`\
       * \ :doc:`../Security/LinkageWithBrowser`\
   * - A4
     - `Insecure Direct Object References <https://www.owasp.org/index.php/Top_10_2013-A4-Insecure_Direct_Object_References>`_ Directory Traversal  
     - * \ :ref:`file-upload_security_related_warning_points_directory_traversal`\  
   * - A5
     - `Security Misconfiguration <https://www.owasp.org/index.php/Top_10_2013-A5-Security_Misconfiguration>`_
     - * \ :doc:`../ArchitectureInDetail/GeneralFuncDetail/Logging`\ (Mention about message contents of log)
       * \ :ref:`exception-handling-how-to-use-codingpoint-jsp-exceptioncode-label`\ (Mention about message output at the time of system exception)
   * - A6
     - `Sensitive Data Exposure <https://www.owasp.org/index.php/Top_10_2013-A6-Sensitive_Data_Exposure>`_
     - * \ :doc:`../ArchitectureInDetail/GeneralFuncDetail/PropertyManagement`\ 
       * \ :doc:`../Security/Encryption`\
       * \ :ref:`SpringSecurityAuthenticationPasswordHashing`\ 
   * - A7
     - `Missing Function Level Access Control <https://www.owasp.org/index.php/Top_10_2013-A7-Missing_Function_Level_Access_Control>`_
     - * \ :doc:`../Security/Authorization`\ 
   * - A8
     - `Cross-Site Request Forgery (CSRF) <https://www.owasp.org/index.php/Top_10_2013-A8-Cross-Site_Request_Forgery_(CSRF)>`_
     - * \ :doc:`../Security/CSRF`\ 
   * - A9
     - `Using Components with Known Vulnerabilities <https://www.owasp.org/index.php/Top_10_2013-A9-Using_Components_with_Known_Vulnerabilities>`_
     - No mention in particular
   * - A10
     - `Unvalidated Redirects and Forwards <https://www.owasp.org/index.php/Top_10_2013-A10-Unvalidated_Redirects_and_Forwards>`_
     - No mention in particular

A point of view by CVE(Common Vulnerabilities and Exposures)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Explain the CVE mentioned in this guideline and show the link.
About the CVE not mentioned in this guideline, refer to\ `Pivotal Product Vulnerability Reports <https://pivotal.io/security>`_\

.. tabularcolumns:: |p{0.10\linewidth}|p{0.40\linewidth}|p{0.50\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 40 50

  * - CVE
    - Outline
    - The mentioned point in this guideline
  * - \ `CVE-2014-0050 <https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0050>`_\

      \ `CVE-2016-3092 <https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-3092>`_\
    - Apache Commons FileUpload allows remote attackers to cause a denial of service via a malicious request.

    - * :ref:`FileUploadOverview`

      * :ref:`file-upload_usage_commons_fileupload`
  * - \ `CVE-2014-1904 <https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-1904>`_\
    - When \ ``action``\ parameter of \ ``<form:form>``\ tag is omitted, an attacker can use XSS attack.
    - * :ref:`ApplicationLayerImplementOfJsp`
  * - \ `CVE-2015-3192 <https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-3192>`_\
    - Using DTD allows DoS attack.
    - * :ref:`ajax_how_to_use`

      * :ref:`RESTHowToUseApplicationSettings`
  * - \ `CVE-2016-5007 <https://pivotal.io/jp/security/cve-2016-5007>`_\
    - Differences in the strictness of the pattern matching mechanisms between Spring MVC and Spring Security cause security bypass vulnerability.
    - * :ref:`access_policy_designate_web_resource`

.. raw:: latex

   \newpage

