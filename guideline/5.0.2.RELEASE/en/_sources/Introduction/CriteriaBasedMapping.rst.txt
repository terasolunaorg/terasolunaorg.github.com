Criteria based mapping of guideline
================================================================================
The chapter 4 of this guideline is structured functionality wise. 
This section shows a mapping from a point of view other than functionality. It indicates which 
part of guideline contains which type of content. 

Mapping based on security measures
--------------------------------------------------------------------------------

Using \ `OWASP Top 10 for 2013 <https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project>`_\  as an axis, 
links to explanation of functionalities related to security have been given


.. tabularcolumns:: |p{0.10\linewidth}|p{0.40\linewidth}|p{0.50\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 40 50

   * - Sr. No.
     - Item Name
     - Corresponding Guideline
   * - A1
     - `Injection <https://www.owasp.org/index.php/Top_10_2013-A1-Injection>`_ SQL Injection
     - * \ :doc:`../ArchitectureInDetail/DataAccessMyBatis3`\
       * \ :doc:`../ArchitectureInDetail/DataAccessJpa`\

       (Details about using bind variable at the time of placeholders for query parameters)
   * - A1
     - `Injection <https://www.owasp.org/index.php/Top_10_2013-A1-Injection>`_ XXE(XML External Entity) Injection
     - * \ :doc:`../ArchitectureInDetail/Ajax`\ 
   * - A2
     - `Broken Authentication and Session Management <https://www.owasp.org/index.php/Top_10_2013-A2-Broken_Authentication_and_Session_Management>`_
     - * \ :doc:`../Security/Authentication`\ 
     
   * - A3
     - `Cross-Site Scripting (XSS) <https://www.owasp.org/index.php/Top_10_2013-A3-Cross-Site_Scripting_(XSS)>`_
     - * \ :doc:`../Security/XSS`\ 
   * - A4
     - `Insecure Direct Object References <https://www.owasp.org/index.php/Top_10_2013-A4-Insecure_Direct_Object_References>`_
     - No mention in particular
   * - A5
     - `Security Misconfiguration <https://www.owasp.org/index.php/Top_10_2013-A5-Security_Misconfiguration>`_
     - * \ :doc:`../ArchitectureInDetail/Logging`\ (Mention about message contents of log)
       * \ :ref:`exception-handling-how-to-use-codingpoint-jsp-exceptioncode-label`\ (Mention about message output at the time of system exception)
   * - A6
     - `Sensitive Data Exposure <https://www.owasp.org/index.php/Top_10_2013-A6-Sensitive_Data_Exposure>`_
     - * \ :doc:`../ArchitectureInDetail/PropertyManagement`\ 
       * \ :doc:`../Security/PasswordHashing`\  (Mention about password hash only)
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
     - * \ :doc:`../Security/Authentication`\ (Mention about Open Redirect Vulnerability measures)


.. raw:: latex

   \newpage

