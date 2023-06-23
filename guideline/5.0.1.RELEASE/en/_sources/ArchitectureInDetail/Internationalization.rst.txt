Internationalization
================================================================================

.. only:: html

 .. contents:: Table of Contents
    :depth: 3
    :local:

Overview
--------------------------------------------------------------------------------

Internationalization is a process wherein the display of labels and messages in an application is not fixed to a specific language. It supports multiple languages. The language switching can be achieved by specifying a unit called "Locale" expressing language, country and region.

This section explains how to internationalize messages to display on the screen.

In order to internationalization, following requrements should be meeted.

* Text elements on the screen(code name, messages, labels of GUI components etc.) should be retrieve from external definitions such as properties file. (should not be hard-coding in the source code)
* The mechanism to specify the locale of the clients should be provided.

Methods to specify the locale are as follows:

* Using standard request header (specify the locale by language settings of browsers)
* Saving the locale into the Cookie using request parameter
* Saving the locale into the Session using request parameter


The image of switching locale is as follows:

.. figure:: ./images_Internationalization/i18n_change_image.png
    :alt: locale change image
    :width: 90%


.. note::

    For internationalization of Codelist, refer to :doc:`Codelist`.

.. tip::

    The most commonly known abbreviation of internationalization is i18n.
    Internationalization is abbreviated as i18n because the number of letters between the first "i" and
    the last "n" is 18 i.e. "nternationalizatio".

|

How to use
--------------------------------------------------------------------------------

Configuration to define messages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To internationalize the messages on the screen, use the one of following \ ``MessageSource``\ imelementations for managing messages.

* ``org.springframework.context.support.ResourceBundleMessageSource``
* ``org.springframework.context.support.ReloadableResourceBundleMessageSource``

The information here explains an example of using the \ ``ResourceBundleMessageSource``\ .

**applicationContext.xml**

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
      - | Specify \ ``i18n/application-messages``\  as base name of properties file.
        | It is recommended to store message properties file under i18n directory to support internationalization.
        |
        | For MessageSource details and definition methods, refer to :doc:`MessageManagement`.


|

**Example of storing properties files**

.. figure:: ./images_Internationalization/i18n_properties_filepath.png
    :alt: properties filepath
    :width: 50%

Properties file should be created in accordance with the following rules.

* File name should be defined in \ :file:`application-messages_XX.properties`\  format. (Specify locale in XX portion)
* The messages defined in \ :file:`application-messages.properties`\  should be created in default language.
* **Make sure you create** \ :file:`application-messages.properties`\ . If it does not exist, messages cannot be fetched from \ ``MessageSource``\  and \ ``JspTagException``\  occurs while setting the messages in JSP.

When creating property files as above, it is determined which to use the file as follows:

* When the locale resolved by \ ``LocaleResolver``\  is zh, \ :file:`application-messages_zh.properties`\  is used.
* when the locale resolved by \ ``LocaleResolver``\  is ja, \ :file:`application-messages_ja.properties`\  is used.
* When properties file does not exist corresponding to the locale resolved by \ ``LocaleResolver``\ , \ :file:`application-messages.properties`\  is used as default. ("_XX" portion does not exist in file name)

.. note::

  The locate to use is determined in the following order until a properties file is found corresponding to the locale.

  #. Locale sent from clients
  #. Locale specified by JVM on which application server runs (it may not be set in some cases)
  #. Locale specified by OS on which application server runs

  It is frequently misunderstood that default properties file is used when properties file does not exist corresponding to the locale sent from clients .
  In this case, then it is checked whether the file is available corresponding to the locale specified by the application server.
  If not found, finally the default properties file is used.

.. tip::

   For description of message properties file, refer to :doc:`MessageManagement`.

|

Changing locale as per browser settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Settings of AcceptHeaderLocaleResolver
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

If switch the locale using browser settings, use the \ ``AcceptHeaderLocaleResolver``\ .

**spring-mvc.xml**

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
        | If this \ ``LocaleResolver``\  is used, HTTP header "accept-language" is added for each request and locale gets specified.

.. note::

  When \ ``LocaleResolver``\  is not set, ``org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver`` is used by default; hence \ ``LocaleResolver``\  need not be set.

|

Definition of messages 
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

See the example of definition of messages below.

**application-messages.properties**

.. code-block:: properties

    title.admin.top = Admin Top

**application-messages_ja.properties**

.. code-block:: properties

    title.admin.top = 管理画面 Top

|

Implementation of JSP
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

See the example of implementaion of messages below.

**include.jsp(Common jsp file to be included)**

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


|

**JSP file for screen display**

.. code-block:: jsp

  <spring:message code="title.admin.top" />  <!-- (2) -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | Sr. No.
      - | Description
    * - | (2)
      - | Output the message using ``<spring:message>``  which is a Spring tag library of JSP.
        | In code attribute, set the key specified in properties.
        | In this example, if locale is ja, "管理画面 Top" is output and for other locales, "Admin Top" is output.

|

Changing locale depending on screen operations dynamically
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The method of dynamic changing the locale depending on screen operations etc. is effective in case of selecting a specific language irrespective of user terminal (browser) settings.

Following is an example of changing locale depending on screen operations.

.. figure:: ./images_Internationalization/i18n_change_locale_on_screen.png
    :alt: i18n change locale on screen
    :align: center
    :width: 40%

To use the language selected by a user, chooose \ ``org.springframework.web.servlet.i18n.LocaleChangeInterceptor``\ .

\ ``LocaleChangeInterceptor``\ is an interceptor to save the locale value specified by the request parameter using \ ``org.springframework.web.servlet.LocaleResolver``\ .

Select the implementation class of \ ``LocaleResolver``\  from the following table.

.. tabularcolumns:: |p{0.05\linewidth}|p{0.60\linewidth}|p{0.35\linewidth}|
.. list-table:: **Types of LocaleResolver**
    :header-rows: 1
    :widths: 5 60 35

    * - No
      - Implementation class
      - How to save locale
    * - 1.
      - ``org.springframework.web.servlet.i18n.SessionLocaleResolver``
      - | Save in server(using \ ``HttpSession``\ )
    * - 2.
      - ``org.springframework.web.servlet.i18n.CookieLocaleResolver``
      - | Save in client(using \ ``Cookie``\ )

.. note::

 When \ ``org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver``\  is used in \ ``LocaleResolver``\ ,
 locale cannot be changed dynamically using \ ``org.springframework.web.servlet.i18n.LocaleChangeInterceptor``\ .

|

How to define LocaleChangeInterceptor
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

If switching the locale using the request parameter, use the \ ``LocaleChangeInterceptor``\ .

**spring-mvc.xml**

.. code-block:: xml

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

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (1)
      - | Define ``org.springframework.web.servlet.i18n.LocaleChangeInterceptor`` in interceptor of Spring MVC.

.. note::

    **How to change the name of request parameter to specify locale**

     .. code-block:: xml

        <bean
            class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor">
            <property name="paramName" value="lang"/>  <!-- (2) -->
        </bean>

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90

        * - | Sr. No.
          - | Description
        * - | (2)
          - | In \ ``paramName``\ property, specify the name of request parameter. In this example, it is "request URL?lang=xx".
            | **When paramName property is omitted, "locale" gets set.** With "request URL?locale=xx", it becomes :ref:`enabled<i18n_set_locale_jsp>`.

|

How to define SessionLocaleResolver
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

If saving the locale in the server side, use the  \ ``SessionLocaleResolver``\ .

**spring-mvc.xml**

.. code-block:: xml

  <bean id="localeResolver" class="org.springframework.web.servlet.i18n.SessionLocaleResolver">  <!-- (1) -->
      <property name="defaultLocale" value="en"/>  <!-- (2) -->
  </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | Sr. No.
      - | Description
    * - | (1)
      - | Define id attribute of bean tag in "localeResolver" and specify the class wherein ``org.springframework.web.servlet.LocaleResolver`` is implemented.
        | In this example, ``org.springframework.web.servlet.i18n.SessionLocaleResolver`` that stores locale in session is specified.
        | id attribute of bean tag should be set as "localeResolver".
        | By performing these settings, \ ``SessionLocaleResolver``\  will be used at the \ ``LocaleChangeInterceptor``\ .
    * - | (2)
      - | When locale is not specified in request parameter, locale specified in \ ``defaultLocale``\  property is enabled. In this case, the value fetched in \ ``HttpServletRequest#getLocale``\  is considered.

|

How to define CookieLocaleResolver
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

If saving the locale in the client side, use the  \ ``CookieLocaleResolver``\ .

**spring-mvc.xml**

.. code-block:: xml

  <bean id="localeResolver" class="org.springframework.web.servlet.i18n.CookieLocaleResolver">  <!-- (1) -->
        <property name="defaultLocale" value="en"/>  <!-- (2) -->
        <property name="cookieName" value="localeCookie"/>  <!-- (3) -->
  </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | Sr. No.
      - | Description
    * - | (1)
      - | In id attribute "localeResolver" of bean tag, specify ``org.springframework.web.servlet.i18n.CookieLocaleResolver``.
        | id attribute of bean tag should be set as "localeResolver".
        | By performing these settings, \ ``CookieLocaleResolver``\  will be used at the \ ``LocaleChangeInterceptor``\ .
    * - | (2)
      - | When locale is not specified, locale specified in \ ``defaultLocale``\  property is enabled. In this case, the value fetched in \ ``HttpServletRequest#getLocale``\  is considered.
    * - | (3)
      - | The value specified in \ ``cookieName``\  property is used as cookie name. If not specified, the value of \ ``org.springframework.web.servlet.i18n.CookieLocaleResolver.LOCALE``\ is used as default. **It is recommended to change not to tell the user explicitly Spring Framework is used.**

|

Messages settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

See the example below for messages settings.

**application-messages.properties**

.. code-block:: properties

    i.xx.yy.0001 = changed locale
    i.xx.yy.0002 = Confirm change of locale at next screen

**application-messages_ja.properties**

.. code-block:: properties

    i.xx.yy.0001 = Localeを変更しました。
    i.xx.yy.0002 = 次の画面でのLocale変更を確認

|

.. _i18n_set_locale_jsp:

Implementation of JSP
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

See the example of implementation of JSP below.

**JSP file for screen display**

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
      - | Submit the request parameter to switch the locale.
        | Request parameter name is specified in \ ``paramName``\  property of \ ``LocaleChangeInterceptor``\ . (In the example above, the default parameter name is used)
        | In the above example, it is changed to English locale in English link and to Japanese locale in Japanese link.
        | Hereafter, the selected locale is enabled.
        | As "en" properties file does not exist, English locale is read from properties file by default.

.. tip::

     * Spring tag library should be defined in common jsp files to be included.
     * For details on common jsp files to be included, refer to :ref:`view_jsp_include-label`.

.. raw:: latex

   \newpage

