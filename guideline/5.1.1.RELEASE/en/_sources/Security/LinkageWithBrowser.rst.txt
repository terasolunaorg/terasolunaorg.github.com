.. _SpringSecurityLinkageWithBrowser:

Coordinating with browser security countermeasure function
================================================================================

.. only:: html

 .. contents:: Table of contents
    :local:

Overview
--------------------------------------------------------------------------------

This chapter explains how to coordinate with the security countermeasure function provided by browser.

Main Web browser provides a few security countermeasure functions so that the functions provided by the browser are not affected.
Some security countermeasure functions provided by the browser can control the operations by displaying response header of HTTP at the server side.

Spring Security provides a system to enhance security of Web application by offering function to output the security response header.

.. note:: **Security risk**

    Even if the security response header is displayed, it does not guarantee 100% elimination of security risk.
    Ultimately, user should consider it as a support function to reduce the security risk.

    Note that, the support status of security header varies depending on the browser.

.. note:: **Overwriting HTTP header**

    HTTP header may be overwritten by the application even though the following settings are done.

Security headers supported by default
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The following 5 response headers are supported by Spring Security by default.

* Cache-Control (Pragma, Expires)
* X-Frame-Options
* X-Content-Type-Options
* X-XSS-Protection
* Strict-Transport-Security

.. tip:: **Support status of browser**

    Some browsers do not support handling these headers. Refer official site of the browser or the following pages.

    * https://www.owasp.org/index.php/HTTP_Strict_Transport_Security (Strict-Transport-Security)
    * https://www.owasp.org/index.php/Clickjacking_Defense_Cheat_Sheet (X-Frame-Options)
    * https://www.owasp.org/index.php/List_of_useful_HTTP_headers (X-Content-Type-Options, X-XSS-Protection)


Cache-Control
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Cache-Control header indicates a method to cache the contents.
Risk of unauthorized users viewing the protected contents can be reduced by disabling caching for the protected contents of the browser..

The following header is output to disable caching the contents.

* Output example of response header

.. code-block:: text

    Cache-Control: no-cache, no-store, max-age=0, must-revalidate
    Pragma: no-cache
    Expires: 0

.. note:: **Overwriting Cache-Control header**

    Cache-Control header is overwritten when Controller class of Spring MVC defines form class of \ ``@SessionAttributes`` \  or
    uses Model of \ ``@SessionAttributes`` \  attribute in the request handler.

.. note:: **Browser compatible with HTTP1.0**

    Pragma header and Expires header are also output to enable Spring Security support the browser compatible with HTTP1.0 as well.


X-Frame-Options
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

X-Frame-Options header indicates whether the contents within the frame (\ ``<frame>``\  or \ ``<iframe>``\  element) are authorized.
Risk of confidential information being stolen by using the malicious practice called Clickjacking can be eliminated by disabling the display of contents within a frame.

The following header is output to deny the display within the frame.

* Output example of response header (Default output of Spring Security)

.. code-block:: text

    X-Frame-Options: DENY

Note that, options other than the output example can be specified in X-Frame-Options header.

X-Content-Type-Options
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

X-Content-Type-Options header indicates a method to determine the contents type.
Some browsers ignore the value of Content-Type header and determine the contents by looking at the content details.
Risk of attack using cross-site scripting can be reduced when you disable contents view while determining the contents type.

The following header is output in order to disable viewing the content details while determining the type of contents.

* Output example of response header

.. code-block:: text

    X-Content-Type-Options: nosniff


X-XSS-Protection
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

X-XSS-Protection header indicates the method to detect harmful script using XSS filter function of the browser.
Risk of attack using cross-site scripting can be reduced by enabling the XSS filter function and detecting harmful script.

Following header is output to enable XSS filter function and detect harmful script.

* Output example of response header (Default output of Spring Security)

.. code-block:: text

    X-XSS-Protection: 1; mode=block

Further, options other than the output example can be specified in X-XSS-Protection header.

Strict-Transport-Security
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Strict-Transport-Security header indicates that user should access the browser by replacing HTTP with HTTPS when user accesses the browser using HTTPS and then tries to access it using HTTP.
Risk of user being directed to malicious sites using malicious technique called as Man-in-the-Middle attack can be reduced by disabling HTTP use after accessing the browser using HTTPS.

Following header is output to disable the use of HTTP after accessing browser using HTTPS.

* Output example of response header (Default output of Spring Security)

.. code-block:: text

    Strict-Transport-Security: max-age=31536000 ; includeSubDomains

.. note:: **Strict-Transport-Security**

    Strict-Transport-Security header is output only when the application server is accessed using HTTPS in the default implementation of Spring Security.
    Note that, Strict-Transport-Security header value can be changed by specifying the option.


How to use
--------------------------------------------------------------------------------

Applying security header output function
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A method is executed to apply the security header output function described earlier.

Security header output function is added by Spring 3.2 and applied by default from Spring Security 4.0.
Therefore, a specific definition is not required to enable the security header output function.
Further, when the security header output function is not to be applied, it must be disabled explicitly.

Define a bean as given below when the security header output function is to be disabled.

* Definition example for spring-security.xml

.. code-block:: xml

    <sec:http>
        <!-- omitted -->
        <sec:headers disabled="true"/> <!-- Disable by setting true in "disabled" attribute -->
        <!-- omitted -->
    </sec:http>


Selecting security header
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Define a bean as given below for selecting the security header to be output.
Here, the example denotes output of all security headers provided by Spring Security, but only required headers should be specified in practice.

* Definition example for spring-security.xml

.. code-block:: xml

    <sec:headers defaults-disabled="true"> <!-- (1) -->
        <sec:cache-control/> <!-- (2) -->
        <sec:frame-options/> <!-- (3) -->
        <sec:content-type-options/> <!-- (4) -->
        <sec:xss-protection/> <!-- (5) -->
        <sec:hsts/> <!-- (6) -->
    </sec:headers>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | First disable the registration of the component which outputs the header applied by default.
    * - | (2)
      - | Register the component which outputs Cache-Control(Pragma, Expires) header.
    * - | (3)
      - | Register the component which outputs Frame-Options header.
    * - | (4)
      - | Register the component which outputs X-Content-Type-Options header.
    * - | (5)
      - | Register the component which outputs X-XSS-Protection header.
    * - | (6)
      - | Register the component which outputs Strict-Transport-Security header.


Further, a method is also provided which disables security headers which are not required.

* Definition example for spring-security.xml
    
.. code-block:: xml 

    <sec:headers>
        <sec:cache-control disabled="true"/> <!-- Disable by setting true in "disabled" attribute --> 
    </sec:headers>

In the above example, only Cache-Control header is not output. 

For details of security header, refer \ `Official reference <http://docs.spring.io/spring-security/site/docs/4.0.3.RELEASE/reference/htmlsingle/#default-security-headers>`_\ .


Specifying options of security header
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Contents which are output by Spring Security by default, can be changed in the following header.

* X-Frame-Options
* X-XSS-Protection
* Strict-Transport-Security

An option \ [#fSpringSecurityLinkageWithBrowser2]_\  can be specified in the attribute of each element by changing the bean definition of Spring Security.

* Definition example for spring-security.xml

.. code-block:: xml

    <sec:frame-options policy="SAMEORIGIN" />

.. [#fSpringSecurityLinkageWithBrowser2] Refer http://docs.spring.io/spring-security/site/docs/4.0.3.RELEASE/reference/htmlsingle/#nsa-headers for the options which can be specified in each element.

Output of custom header
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Security can also output the headers which are not provided by default.

A case study wherein following header is output, is explained.

.. code-block:: text

    X-WebKit-CSP: default-src 'self'

Define a bean as follows when the header described above is to be output.

* Definition example for spring-security.xml

.. code-block:: xml

      <sec:headers>
          <sec:header name="X-WebKit-CSP" value="default-src 'self'"/>
      </sec:headers>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Add \ ``<sec:header>`` as child element of \ ``<sec:headers>``\  element and specify the header name in \ ``name``\  attribute and header value in \ ``value``\  attribute.

Displaying security header for each request pattern
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Security can control the output of security header for each request pattern by using \ ``RequestMatcher``\  interface system.

For example, when the contents to be protected are stored under the path \ ``/secure/``\  and Cache-Control header is to be output only when the contents to be protected are accessed, define a bean as follows.

* Definition example for spring-security.xml

.. code-block:: xml

    <!-- (1) -->
    <bean id="secureCacheControlHeadersWriter"
          class="org.springframework.security.web.header.writers.DelegatingRequestMatcherHeaderWriter">
        <constructor-arg>
            <bean class="org.springframework.security.web.util.matcher.AntPathRequestMatcher">
                <constructor-arg value="/secure/**"/>
            </bean>
        </constructor-arg>
        <constructor-arg>
            <bean class="org.springframework.security.web.header.writers.CacheControlHeadersWriter"/>
        </constructor-arg>
    </bean>

    <sec:http>
        <!-- omitted -->
        <sec:headers>
            <sec:header ref="secureCacheControlHeadersWriter"/> <!-- (2) -->
        </sec:headers>
        <!-- omitted -->
    </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify implementation class of \ ``RequestMatcher``\  and \ ``HeadersWriter``\  interface and define a bean for \ ``DelegatingRequestMatcherHeaderWriter``\  class.
    * - | (2)
      - | Add \ ``<sec:header>`` as child element of \ ``<sec:headers>``\  element and specify a bean for \ ``HeaderWriter``\  defined in (1) in \ ``ref``\  attribute.


.. raw:: latex

   \newpage

