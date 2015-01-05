Internationalization
================================================================================

.. only:: html

 .. contents:: Table of Contents
    :depth: 3
    :local:

Overview
--------------------------------------------------------------------------------

Internationalization is a process wherein the display of labels and messages in an application is not fixed to a specific language. It supports multiple languages. The language switching can be achieved by specifying a unit called "Locale" expressing language, country and region.

This chapter explains different methods for internationalization of messages.

| The following points should be noted to support internationalization.

* Text elements such as status, messages and labels of GUI components etc. should not be hardcoded in the program.
* These elements should be stored in external data other than program data.

For implementing internationalization, locale needs to be stored. Locale can mainly be stored in,

* Session
* Cookie



The image of changing locale is as follows:

 .. figure:: ./images_Internationalization/i18n_change_image.png
    :alt: locale change image
    :width: 40%

The location of storing locale is decided depending on the implementation method.

This chapter states various rules to use internationalization and the recommended implementation methods.

    .. note::

      The most commonly known abbreviation of internationalization is i18n.
      Internationalization is abbreviated as i18n because the number of letters between the first "i" and 
      the last "n" is 18 i.e. "nternationalizatio".

    .. tip::

      For internationalization of Codelist, refer to :doc:`Codelist`.

|

How to use

Changing locale as per user terminal (or browser) settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| By using Spring ``org.springframework.context.support.ResourceBundleMessageSource``, locale can be changed as per the settings of user terminal (or browser).
| Here, internationalization when MessageSource is being used is explained.

    .. tip::

     For MessageSource details and definition methods, refer to :doc:`MessageManagement`.

|

**bean definition file**

- applicationContext.xml

 .. code-block:: xml

    <bean id="messageSource"
        class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basenames">
            <list>
                <value>i18n/application-messages</value>  <!-- (1) -->
            </list>
        </property>
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | Sr. No.
      - | Description
    * - | (1)
      - | Specify i18n/application-messages as base name of properties file.
        | It is recommended to store message properties file under i18n directory to support internationalization.

- spring-mvc.xml

 .. code-block:: xml

    <bean id="localeResolver"
        class="org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver" /> <!-- (1) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | Sr. No.
      - | Description
    * - | (1)
      - | Specify ``org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver`` in id attribute "localeResolver" of bean tag.
        | If this localeResolver is used, HTTP header "accept-language" is added for each request and locale gets specified.

 .. note::

  When localeResolver is not set, ``org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver`` is used by default; hence localeResolver need not be set.


**File path**

 .. figure:: ./images_Internationalization/i18n_properties_filepath.png
    :alt: properties filepath
    :width: 40%

| File name should be defined in \ :file:`application-messages_XX.properties`\  format. Specify locale in XX portion.
| When locale resolved using \ ``LocaleResolver``\  is zh, \ :file:`application-messages_zh.properties`\  is used, and
| when locale resolved using \ ``LocaleResolver``\  is ja, \ :file:`application-messages_ja.properties`\  is used.
| When properties file corresponding to locale resolved using \ ``LocaleResolver``\  does not exist, \ :file:`application-messages.properties`\  is used by default. ("_XX" portion does not exist in file name)
| Note the following while using \ :file:`application-messages.properties`\ .

* The messages defined in \ :file:`application-messages.properties`\  should be created in default language.
* **Make sure you create** \ :file:`application-messages.properties`\ . If it does not exist, messages cannot be fetched from MessageSource and JspTagException occurs while setting the messages in JSP.

 .. tip::

   For description of message properties file, refer to :doc:`MessageManagement`.

 .. note::

  In locale determination, locale is verified until properties file of the corresponding locale is found in the following order.

  #. Locale specified in HTTP header "accept-language" of request
  #. Locale specified in JVM of application server (may not be set in some cases)
  #. Locale specified in OS of application server

  It is frequently misunderstood that when properties file of locale corresponding to value of HTTP header "accept-language" of request does not exist, default properties file is used.
  In actual scenario, locale specified in the application server in subsequent process is verified and even then the properties file of the corresponding locale is not found, default properties file is used.

Example of setting message definition is as follows:

**Properties file**

- application-messages.properties

 .. code-block:: properties

    title.admin.top = Admin Top

- application-messages_jp.properties

 .. code-block:: properties

    title.admin.top = 管理画面 Top

**JSP file**

- include.jsp(Common jsp file to be included)

 .. code-block:: jsp

  <%@ page session="false"%>
  <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
  <%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt"%>
  <%@ taglib uri="http://www.springframework.org/tags" prefix="spring"%>  <!-- (1) -->
  <%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>
  <%@ taglib uri="http://www.springframework.org/security/tags" prefix="sec"%>
  <%@ taglib uri="http://terasoluna.org/functions" prefix="f"%>
  <%@ taglib uri="http://terasoluna.org/tags" prefix="t"%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | Sr. No.
      - | Description
    * - | (1)
      - | When message is to be output in JSP, it is output using Spring tag library; hence custom tag needs to be defined.
        | ``<%@taglib uri="http://www.springframework.org/tags" prefix="spring"%>``  should be defined.

 .. note::

  For details on common jsp files to be included, refer to :ref:`view_jsp_include-label`.


- JSP file for screen display

 .. code-block:: java

  <spring:message code="title.admin.top" />  <!-- (1) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | Sr. No.
      - | Description
    * - | (1)
      - | Output the message using ``<spring:message>``  which is a Spring tag library of JSP.
        | In code attribute, set the key specified in properties.
        | In this example, if locale is ja, "管理画面 Top" is output and for other locales, "Admin Top" is output.

|

For dynamically changing locale depending on screen operations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| The method of dynamically changing the locale depending on screen operations etc. is effective in case of selecting a specific language irrespective of user terminal (browser) settings.
| 
| This can be implemented using ``org.springframework.web.servlet.i18n.LocaleChangeInterceptor``.
| LocaleChangeInterceptor stores the data in ``org.springframework.web.servlet.LocaleResolver``
| using the locale value specified in request parameter.

| Select the implementation class of LocaleResolver from the following table as per the storage location  of locale.

 .. tabularcolumns:: |p{0.05\linewidth}|p{0.60\linewidth}|p{0.35\linewidth}|
 .. list-table:: **Types of LocaleResolver for using Interceptor**
    :header-rows: 1
    :widths: 5 60 35

    * - No
      - Implementation class
      - How to store locale
    * - 1.
      - ``org.springframework.web.servlet.i18n.SessionLocaleResolver``
      - | Store in server
    * - 2.
      - ``org.springframework.web.servlet.i18n.CookieLocaleResolver``
      - | Store in client

.. note::

 When ``org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver`` is used in LocaleResolver,
 locale cannot be changed dynamically using ``org.springframework.web.servlet.i18n.LocaleChangeInterceptor``.

**Bean definition file**

In case of SessionLocaleResolver

- spring-mvc.xml

 .. code-block:: xml

  <!-- omitted -->
  <mvc:interceptors>
    <mvc:interceptor>
      <mvc:mapping path="/**" />
      <mvc:exclude-mapping path="/resources/**" />
      <mvc:exclude-mapping path="/**/*.html" />
      <bean
        class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor">  <!-- (1) -->
      </bean>
	  <!-- omitted -->
    </mvc:interceptor>
  </mvc:interceptors>

  <bean id="localeResolver" class="org.springframework.web.servlet.i18n.SessionLocaleResolver">  <!-- (2) -->
      <property name="defaultLocale" value="en"/>  <!-- (3) -->
  </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | Sr. No.
      - | Description
    * - | (1)
      - | Define ``org.springframework.web.servlet.i18n.LocaleChangeInterceptor`` in interceptor of Spring MVC.
    * - | (2)
      - | Define id attribute of bean tag in "localeResolver" and specify the class wherein ``org.springframework.web.servlet.LocaleResolver`` is implemented.
        | In this example, ``org.springframework.web.servlet.i18n.SessionLocaleResolver`` that stores locale in session is specified.
        | id attribute of bean tag should be set as "localeResolver".
        | By performing these settings, LocaleResolver used in ``org.springframework.web.servlet.i18n.LocaleChangeInterceptor`` gets set in LocaleResolver of (3).
    * - | (3)
      - | When locale is not specified in request parameter, locale specified in defaultLocale is enabled. In this case, the value fetched in ``HttpServletRequest#getLocale`` is considered.

.. _i18n_change_locale_key:

* When changing key of locale to be set in request parameter

- spring-mvc.xml

 .. code-block:: xml

      <bean
        class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor">
        <property name="paramName" value="lang"/>  <!-- (1) -->
      </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | Sr. No.
      - | Description
    * - | (1)
      - | In paramName, specify the key of locale set in request parameter. In this example, it is "request URL?lang=xx".
        | **When key is not specified, "locale" gets set.** With "request URL?locale=xx", it becomes :ref:`enabled<i18n_set_locale_jsp>`.


In case of CookieLocaleResolver

- spring-mvc.xml

 .. code-block:: xml

  <!-- omitted -->
  <mvc:interceptors>
    <mvc:interceptor>
      <mvc:mapping path="/**" />
      <mvc:exclude-mapping path="/resources/**" />
      <mvc:exclude-mapping path="/**/*.html" />
      <bean
        class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor">  <!-- (1) -->
      </bean>
	  <!-- omitted -->
    </mvc:interceptor>
  </mvc:interceptors>

  <bean id="localeResolver" class="org.springframework.web.servlet.i18n.CookieLocaleResolver">  <!-- (2) -->
        <property name="defaultLocale" value="en"/>  <!-- (3) -->
        <property name="cookieName" value="localeCookie"/>  <!-- (4) -->
  </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | Sr. No.
      - | Description
    * - | (1)
      - | Define ``org.springframework.web.servlet.i18n.LocaleChangeInterceptor`` in Spring MVC interceptor.
    * - | (2)
      - | In id attribute "localeResolver" of bean tag, specify ``org.springframework.web.servlet.i18n.CookieLocaleResolver``.
        | id attribute of bean tag should be set as "localeResolver".
        | By performing these settings, LocaleResolver used in ``org.springframework.web.servlet.i18n.LocaleChangeInterceptor`` is set to LocaleResolver of (3).
    * - | (3)
      - | When locale is not specified, locale specified in defaultLocale is enabled. In this case, the value fetched in ``HttpServletRequest#getLocale`` is considered.
    * - | (4)
      - | The value specified in cookieName properties is considered cookie name. If not specified, it is considered as ``org.springframework.web.servlet.i18n.CookieLocaleResolver.LOCALE``. It is recommended to change the same since use of springframework is explicit.

* When changing the key of locale to be set in request parameter

:ref:`settings<i18n_change_locale_key>` are similar to SessionLocaleResolver.

See the example below for properties settings.


**Properties file**

- application-messages.properties

 .. code-block:: properties

    i.xx.yy.0001 = changed locale
    i.xx.yy.0002 = Confirm change of locale at next screen

- application-messages_ja.properties

 .. code-block:: properties

    i.xx.yy.0001 = Localeを変更しました。
    i.xx.yy.0002 = 次の画面でのLocale変更を確認

.. _i18n_set_locale_jsp:

**JSP file**

- JSP file for screen display

 .. code-block:: jsp

    <a href='${pageContext.request.contextPath}?locale=en'>English</a>  <!-- (1) -->
    <a href='${pageContext.request.contextPath}?locale=ja'>Japanese</a>
    <spring:message code="i.xx.yy.0001" />

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | Sr. No.
      - | Description
    * - | (1)
      - | Submit the request parameter key specified in paramName of LocaleChangeInterceptor.
        | In the above example, it is changed to English locale in English link and to Japanese locale in Japanese link.
        | Hereafter, the selected locale is enabled.
        | As "en" properties file does not exist, English locale is read from properties file by default.

 .. tip::

     * Spring tag library should be defined in common jsp files to be included.
     * For details on common jsp files to be included, refer to :ref:`view_jsp_include-label`.

Following is an example of changing locale depending on screen operations.

.. figure:: ./images_Internationalization/i18n_change_locale_on_screen.png
   :alt: i18n change locale on screen
   :width: 30%
   :align: center

.. raw:: latex

   \newpage

