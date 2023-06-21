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


.. raw:: latex

   \newpage
