Spring Security Overview
================================================================================

.. only:: html

 .. contents:: Table of contents
    :local:

Overview
--------------------------------------------------------------------------------

| Two main functionalities namely, "Authentication" and "Authorization" are provided by Spring Security
| for the security of applications.
| Authentication functionality identifies a user and thus prevents unauthorized access through spoofing.
| Authorization functionality controls the access to system resources
| according to the authority of the authenticated (logged-in) user.
| Moreover, it has the functionality that assigns HTTP headers.

| Spring Security overview is shown in diagram below.

.. figure:: ./images/spring_security_overview.png
   :alt: Spring Security Overview
   :width: 80%
   :align: center

   **Picture - Spring Security Overview**

| Spring Security implements authorization and authentication processes
| with help of a group of ServletFilters that interact across several levels.
| Further, it also provides password hashing functionality, JSP authorized tag library etc.

Authentication
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Authentication is the action that checks validity of a request. When connecting to the network or server, 
| through combination of user name and password, it further verifies whether the user has the required authority and
| also whether the person to be authenticated is really the user himself.
| For the details on how to use authentication in Spring Security, refer to \ :doc:`Authentication`\ .

Password hashing
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| In password hashing, the original password is replaced with a hash value that is derived from the plaintext password using hash function.
| For the details on how to use it in Spring Security, refer \ :doc:`PasswordHashing`\ .

Authorization
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Authorization is the functionality to verify whether the authenticated user is allowed to use the resource that he is trying to access,
| using access control process.
| For the details on how to use authorization in Spring Security, refer to \ :doc:`Authorization`\ .

.. _howtouse_springsecurity:

How to use
--------------------------------------------------------------------------------

| Following settings need to be defined for using Spring Security.

pom.xml settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| To use Spring Security, following dependency needs to be added to pom.xml.

.. code-block:: xml

    <dependency>
        <groupId>org.terasoluna.gfw</groupId>
        <artifactId>terasoluna-gfw-security-core</artifactId>  <!-- (1) -->
    </dependency>

    <dependency>
        <groupId>org.terasoluna.gfw</groupId>
        <artifactId>terasoluna-gfw-security-web</artifactId>  <!-- (2) -->
    </dependency>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | terasoluna-gfw-security-core is not web dependent. As a result, when using from a domain layer project,
       | only terasoluna-gfw-security-core should be added to dependency.
   * - | (2)
     - | terasoluna-gfw-web provides web related functionalities. It is dependent on terasoluna-gfw-security-core as well. Hence,
       | for Web projects, only terasoluna-gfw-security-web should be added to dependency.

Web.xml settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: xml
   :emphasize-lines: 5,13-20

    <context-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>  <!-- (1) -->
          classpath*:META-INF/spring/applicationContext.xml
          classpath*:META-INF/spring/spring-security.xml
      </param-value>
    </context-param>
    <listener>
      <listener-class>
        org.springframework.web.context.ContextLoaderListener
      </listener-class>
    </listener>
    <filter>
      <filter-name>springSecurityFilterChain</filter-name>  <!-- (2) -->
      <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>  <!-- (3) -->
    </filter>
    <filter-mapping>
      <filter-name>springSecurityFilterChain</filter-name>
      <url-pattern>/*</url-pattern>  <!-- (4) -->
    </filter-mapping>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | In addition to applicationContext.xml, add the Spring Security configuration file to 
       | the class path in contextConfigLocation. In this guideline, it is "spring-security.xml" file.
   * - | (2)
     - | filter-name should be defined as the Bean name to be used internally in Spring Security, namely, "springSecurityFilterChain".
   * - | (3)
     - Spring Security filter settings to enable various functionalities.
   * - | (4)
     - Enable the settings for all requests.

spring-security.xml settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| spring-security.xml is placed under the path specified in web.xml.
| Normally, it is set in src/main/resources/META-INF/spring/spring-security.xml.
| Please refer subsequent chapters for detailed explanation, as the following example is just a template.

* spring-mvc.xml

  .. code-block:: xml

    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:sec="http://www.springframework.org/schema/security"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/security
            http://www.springframework.org/schema/security/spring-security.xsd
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd">
        <sec:http  use-expressions="true">  <!-- (1) -->
        <!-- omitted -->
        </sec:http>
    </beans>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | Spring EL expressions of access attribute can be enabled by setting use-expressions="true".

  \

      .. note::
          For the Spring EL expressions enabled by use-expressions="true", please refer the following.

          \ `Expression-Based Access Control <http://static.springsource.org/spring-security/site/docs/3.1.x/reference/el-access.html>`_\

Appendix
--------------------------------------------------------------------------------

Settings to assign a secure HTTP header
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As shown below, security related headers can be set automatically in HTTP response by setting \ ``<sec:headers>``\  element in \ ``<sec:http>``\  of spring-security.xml.
By assigning these HTTP response headers, Web browser can detect an attack and cope with it.
This setting is not mandatory however, is recommended to strengthen the security.

.. code-block:: xml

    <sec:http use-expressions="true">
      <!-- omitted -->
      <sec:headers />
      <!-- omitted -->
    </sec:http>

In this setting, HTTP response headers related to following fields are set.

* Cache-Control
* X-Content-Type-Options
* Strict-Transport-Security
* X-Frame-Options
* X-XSS-Protection

.. tabularcolumns:: |p{0.2\linewidth}|p{0.5\linewidth}||p{0.3\linewidth}|
.. list-table:: 
   :header-rows: 1
   :widths: 20 50 30

   * - HTTP header name
     - Issues due to inappropriate settings (also includes cases where settings are not performed).
     - Action in case of appropriate settings
   * - | \ ``Cache-Control``\ 
     - | In some cases, the contents that can be viewed by a logged-in user are cached and can also be viewed by another user, after the first user logs-out.
     - | Instruct such that the contents are not cached and ensure that the browser always fetches server information.
   * - | \ ``X-Content-Type-Options``\ 
     - | Browser determines the contents for operation without using Content-Type, just by checking their details. This may result in execution of unexpected scripts.
     - | Ensure that, the browser does not determine the contents to be operated without using Content-Type, just by checking their details. Restrict Script execution if the MIME type does not match.
   * - | \ ``Strict-Transport-Security``\ 
     - | In spite of expecting access to a secure page by HTTPS, there is a possibility of HTTP-origin attack (Example: Man in the middle attack (MITM) intercepts a user's HTTP request and redirects it to a malicious site.), when the page is accessed using HTTP.
     - | Once a legitimate web site is accessed using HTTPS, the browser is set such that it automatically uses only HTTPS, thereby preventing Man in the middle attack of being redirected to a malicious site.
   * - | \ ``X-Frame-Options``\ 
     - | If screen of the malicious Web site is made unavailable for viewing and instead, a legitimate site B is embedded using \ ``<iframe>``\  tag, the user who thinks that he has accessed site B, is made to access site A, by the attacker.
       | In this condition, if the Send button of site A and link position of site B overlap, the attacker can make the user send a malicious request through site A, while the user thinks that the link of legitimate site B was clicked. (\ `Clickjacking <https://www.owasp.org/index.php/Clickjacking>`_\ )
     - | By using \ ``<iframe>``\  tag, ensure that the self-created Website (=site B) cannot be read in other Web site (= site A).
   * - | \ ``X-XSS-Protection``\ 
     - | Decision given by XSS filter on the harmful script, is invalidated.
     - | On realizing that the script is harmful, the XSS filter implemented in browser asks the user whether to execute it or the decision is invalidated (action differs according to each bowser).



The settings mentioned above can be performed individually as shown below in steps (1) to (5). Select them as and when required.

.. code-block:: xml

    <sec:http use-expressions="true">
      <!-- omitted -->
      <sec:headers>
        <sec:cache-control />  <!-- (1) -->
        <sec:content-type-options />  <!-- (2) -->
        <sec:hsts />  <!-- (3) -->
        <sec:frame-options />  <!-- (4) -->
        <sec:xss-protection />  <!-- (5) -->
      </sec:headers>
      <!-- omitted -->
    </sec:http>

.. tabularcolumns:: |p{0.05\linewidth}|p{0.45\linewidth}|p{0.40\linewidth}|p{0.10\linewidth}|
.. list-table:: Assigning HTTP header by Spring Security
   :header-rows: 1
   :widths: 5 45 40 10

   * - Sr. No.
     - Description
     - Default HTTP response header that is output.
     - Presence or absence of attribute
   * - | (1)
     - | Instructs the client not to cache data.
     - | \ ``Cache-Control:no-cache, no-store, max-age=0, must-revalidate``\ 
       | \ ``Pragma: no-cache``\ 
       | \ ``Expires: 0``\ 
     - | Not present
   * - | (2)
     - | Instructs the client not to decide the processing method by disregarding the content type using just the content details.
     - | \ ``X-Content-Type-Options:nosniff``\ 
     - | Not present
   * - | (3)
     - | Instructs to continue the HTTPS connection in the site accessed by HTTPS. (It is disregarded in case of HTTP site and is not assigned as header field.)
     - | \ ``Strict-Transport-Security:max-age=31536000 ; includeSubDomains``\ 
     - | Present
   * - | (4)
     - | Instructs regarding the displaying availability of contents in iframe.
     - | \ ``X-Frame-Options:DENY``\ 
     - | Present
   * - | (5)
     - | Instructs the browser implemented with a filter that can detect \ `XSS attack <https://www.owasp.org/index.php/Cross-site_Scripting_(XSS)>`_\ , to enable XSS filter functionality.
     - | \ ``X-XSS-Protection:1; mode=block``\ 
     - | Present

|

Attributes can be set when individual settings are performed. Some of the attributes that can be set are explained here.

.. tabularcolumns:: |p{0.05\linewidth}|p{0.20\linewidth}|p{0.30\linewidth}|p{0.20\linewidth}|p{0.25\linewidth}|
.. list-table:: Attributes that can be set
   :header-rows: 1
   :widths: 5 20 30 20 25

   * - Sr. No.
     - Option
     - Description
     - Specified example
     - HTTP response header that is output
   * - | (3)
     - | \ ``max-age-seconds``\ 
     - | Number of seconds for which the fact that corresponding site should be accessed only using HTTPS, is stored in memory. (Default 365 days)
     - | \ ``<sec:hsts max-age-seconds="1000" />``\ 
     - | \ ``Strict-Transport-Security:max-age=1000 ; includeSubDomains``\ 
   * - | (3)
     - | \ ``include-subdomains``\ 
     - | Application instructions for sub-domain. Default value is \ ``true``\ . Output is stopped if specified as \ ``false``\ .
     - | \ ``<sec:hsts include-subdomains="false" />``\ 
     - | \ ``Strict-Transport-Security:max-age=31536000``\ 
   * - | (4)
     - | \ ``policy``\ 
     - | Instructs regarding the permission method to display contents in iframe. Default value is \ ``DENY``\  (Display in frame is completely prohibited). It can also be changed to \ ``SAMEORIGIN``\  (allows to read only the page in the same site).
     - | \ ``<sec:frame-options policy="SAMEORIGIN" />``\ 
     - | \ ``X-Frame-Options:SAMEORIGIN``\ 
   * - | (5)
     - | \ ``enabled,block``\ 
     - | XSS filter can be disabled by specifying as \ ``false``\ . However, it is recommended to enable this filter.
     - | \ ``<sec:xss-protection enabled="false" block="false" />``\ 
     - | \ ``X-XSS-Protection:0``\ 


.. note::

    The processing for these headers is not supported in some browsers. Please refer the official site of the browser or the following pages.

    * https://www.owasp.org/index.php/HTTP_Strict_Transport_Security (Strict-Transport-Security)
    * https://www.owasp.org/index.php/Clickjacking_Defense_Cheat_Sheet (X-Frame-Options)
    * https://www.owasp.org/index.php/List_of_useful_HTTP_headers (X-Content-Type-Options, X-XSS-Protection)


For details, see \ `Official reference <http://docs.spring.io/spring-security/site/docs/3.2.4.RELEASE/reference/htmlsingle/#default-security-headers>`_\ .

    
.. raw:: latex

   \newpage
