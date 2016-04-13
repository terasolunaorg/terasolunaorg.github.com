Codelist
================================================================================

.. only:: html

 .. contents:: Table of Contents
    :depth: 4
    :local:

Overview
--------------------------------------------------------------------------------

A codelist is a pair comprising of "Code values (Value) and their display names (Label)".

It is used for mapping code values with the labels to be displayed on screen such as selectbox.

Common library provides the following functionalities:

* Functionality to read and cache the codelist defined in xml file or DB at the time of launching the application.
* Functionality to refer to the codelist from JSP or Java class
* Functionality to perform input validation using codelist

Moreover, it also supports

* internationalization of codelist
* reloading of cached codelist

.. note::

    As per standard specifications, only the codelist defined in DB is reloadable.

|

Following four types of codelists are implemented in common library.

.. _listOfCodeList:

.. tabularcolumns:: |p{0.50\linewidth}|p{0.30\linewidth}|p{0.20\linewidth}|
.. list-table:: **Types of codelist**
   :header-rows: 1
   :widths: 50 30 20

   * - Types
     - Contents
     - Reloadable
   * - ``org.terasoluna.gfw.common.codelist.SimpleMapCodeList``
     - Use the hard-coded contents from xml file.
     - NO
   * - ``org.terasoluna.gfw.common.codelist.NumberRangeCodeList``
     - Use when creating number range list.
     - NO
   * - ``org.terasoluna.gfw.common.codelist.JdbcCodeList``
     - Use the codelist by fetching the code from DB using SQL.
     - YES
   * - ``org.terasoluna.gfw.common.codelist.EnumCodeList``
     - Use when creating the codelist from constant defined in \ ``Enum``\  class.
     - NO
   * - ``org.terasoluna.gfw.common.codelist.i18n.SimpleI18nCodeList``
     - Use the codelist corresponding to java.util.Locale.
     - NO

Common library provides ``org.terasoluna.gfw.common.codelist.CodeList`` for the interface of above codelists.

Codelist class diagram provided in common library is as follows:

.. figure:: ./images/codelist-class-diagram.png
   :alt: codelist class diagram
   :align: center

   **Picture - Image of codelist class diagram**

|

How to use
--------------------------------------------------------------------------------

This section describes settings for various codelists and their implementation methods.

* :ref:`codelist-simple`
* :ref:`codelist-number`
* :ref:`codelist-jdbc`
* :ref:`codelist-enum`
* :ref:`codelisti18n`
* :ref:`codelist-display-label`
* :ref:`codelist-validate`

|

.. _codelist-simple:

Using SimpleMapCodeList
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
``org.terasoluna.gfw.common.codelist.SimpleMapCodeList`` reads the code values defined in xml file at the time of launching the application and uses them as is.

**SimpleMapCodeList image**

.. figure:: ./images/codelist-simple.png
   :alt: codelist simple
   :width: 100%

|

Example of codelist settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**Definition of Bean definition file(xxx-codelist.xml)**

It is recommended to create a bean definition file for codelist.

.. code-block:: xml
   :emphasize-lines: 1,4

    <bean id="CL_ORDERSTATUS" class="org.terasoluna.gfw.common.codelist.SimpleMapCodeList"> <!-- (1) -->
        <property name="map">
            <util:map>
                <entry key="1" value="Received" /> <!-- (2) -->
                <entry key="2" value="Sent" />
                <entry key="3" value="Cancelled" />
            </util:map>
        </property>
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Define a bean of SimpleMapCodeList class.
       | beanID should have the name matching with the ID pattern of ``org.terasoluna.gfw.web.codelist.CodeListInterceptor`` described later.
   * - | (2)
     - | Define Key, Value pairs of Map.
       | When map-class attribute is omitted, it is registered in ``java.util.LinkedHashMap``; hence in the above example, "Name and value" are stored in Map in the order of registration.

|

**Definition of Bean definition file(xxx-domain.xml)**

Once the bean definition file for codelist is created, it should be imported to already existing bean definition file.

.. code-block:: xml
   :emphasize-lines: 1,4

    <import resource="classpath:META-INF/spring/projectName-codelist.xml" /> <!-- (3) -->
    <context:component-scan base-package="com.example.domain" />

    <!-- omitted -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (3)
     - | Import bean definition file for codelist.
       | Resource information of import is necessary during component-scan;
       | hence import should be set above ``<context:component-scan base-package="com.example.domain" />``.

|

.. _clientSide:

Using codelist in JSP
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

By using the interceptor of common library,
codelist can be set automatically in request scope and can be easily referred from JSP.

**Definition of Bean definition file(spring-mvc.xml)**

.. code-block:: xml
   :emphasize-lines: 3,5,6

    <mvc:interceptors>
      <mvc:interceptor>
        <mvc:mapping path="/**" /> <!-- (1) -->
        <bean
          class="org.terasoluna.gfw.web.codelist.CodeListInterceptor"> <!-- (2) -->
          <property name="codeListIdPattern" value="CL_.+" /> <!-- (3) -->
        </bean>
      </mvc:interceptor>

      <!-- omitted -->

    </mvc:interceptors>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Set the applicable path.
   * - | (2)
     - | Define a bean of CodeListInterceptor class.
   * - | (3)
     - | Set the beanID pattern of codelist which is automatically set in the request scope.
       | In pattern, regular expression used in ``java.util.regex.Pattern`` should be set.
       | In the above example, only the data in which id is defined in "CL\_XXX" format is targeted. In that case, bean definition wherein id does not start with "CL\_" should not be imported.
       | beanID defined in "CL\_" can be used in JSP since it is set in the request scope.
       |
       | \ ``codeListIdPattern``\  property can be omitted.
       | If omitting \ ``codeListIdPattern``\  property, all of \ ``CodeList``\s (all beans which implements ``org.terasoluna.gfw.common.codelist.CodeList``) are available in JSP.

|

**Example of implementing the codelist in jsp**

.. code-block:: jsp

  <form:select path="orderStatus">
    <form:option value="" label="--Select--" /> <!-- (4) -->
    <form:options items="${CL_ORDERSTATUS}" /> <!-- (5) -->
  </form:select>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (4)
     - | When setting dummy value at the top of the selectbox, null characters should be specified in the value.
   * - | (5)
     - | Specify the beanID for which codelist is defined.

**Output HTML**

.. code-block:: html

  <select id="orderStatus" name="orderStatus">
     <option value="">"--Select--</option>
     <option value="1">Received</option>
     <option value="2">Sent</option>
     <option value="3">Cancelled</option>
  </select>

**Output screen**

.. figure:: ./images/codelist_selectbox.png
   :alt: codelist selectbox
   :width: 40%

|

.. _serverSide:

Using codelist in Java class
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When using the codelist in Java class,
inject the codelist by setting ``javax.inject.Inject`` annotation and ``javax.inject.Named`` annotation.
Specify the codelist name in ``@Named`` annotation.

.. code-block:: java

  import javax.inject.Named;

  import org.terasoluna.gfw.common.codelist.CodeList;

  public class OrderServiceImpl implements OrderService {

      @Inject
      @Named("CL_ORDERSTATUS")
      CodeList orderStatusCodeList; // (1)

      public boolean existOrderStatus(String target) {
          return orderStatusCodeList.asMap().containsKey(target); // (2)
      }
  }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Inject the codelist with beanID "CL_ORDERSTATUS".
   * - | (2)
     - | Fetch the codelist in ``java.util.Map`` format using CodeList#asMap method.

|

.. _codelist-number:

Using NumberRangeCodeList
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``org.terasoluna.gfw.common.codelist.NumberRangeCodeList`` is a codelist that creates the list of numeric values of specified range at the time of launching the application.
It is assumed that this codelist will mainly be used in the selectboxes having only numbers i.e. selectbox for month, date etc.

**Image of NumberRangeCodeList**

.. figure:: ./images/codelist-number.png
   :alt: codelist number
   :width: 100%

.. tip::

    NumberRangeCodeList supports only Arabic numbers and does not support Chinese and Roman numbers.
    Chinese and Roman numbers can be supported by using JdbcCodeList and SimpleMapCodeList.

|

NumberRangeCodeList has the following features:

#. In order to set From value < To value, the values increased in accordance with the interval are set in From-To range in ascending order.
#. In order to set To value < From value, the values decreased in accordance with the interval are set in To-From range in descending order.
#. Increment (decrement) can be changed by setting intervals.

|

The information here describes how to configure the ascending \ ``NumberRangeCodeList``\ .
For how to create the descending \ ``NumberRangeCodeList``\  or change interval, refer to ":ref:`CodeListAppendixNumberRangeCodeListVariation`".
|

Example of codelist settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Example of setting From value < To value is shown below.

**Definition of Bean definition file(xxx-codelist.xml)**

.. code-block:: xml

    <bean id="CL_MONTH"
        class="org.terasoluna.gfw.common.codelist.NumberRangeCodeList"> <!-- (1) -->
        <property name="from" value="1" /> <!-- (2) -->
        <property name="to" value="12" /> <!-- (3) -->
        <property name="valueFormat" value="%d" /> <!-- (4) -->
        <property name="labelFormat" value="%02d" /> <!-- (5) -->
        <property name="interval" value="1" /> <!-- (6) -->
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Define a bean of NumberRangeCodeList.
   * - | (2)
     - | Specify the range start value. When omitted, "0" is set as range start value.
   * - | (3)
     - | Specify the range end value. It cannot be blank.
   * - | (4)
     - | Specify the format of the code value. Format used should be ``java.lang.String.format``.
       | When omitted, "%s" is set.
   * - | (5)
     - | Specify the format of the code name. Format used should be ``java.lang.String.format``.
       | When omitted, "%s" is set.
   * - | (6)
     - | Set the increment value. When omitted, "1" is set.

|

Using codelist in JSP
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

For details on settings shown below, refer to :ref:`Using codelist in JSP <clientSide>` described earlier.

**Example of jsp implementation**

.. code-block:: jsp

  <form:select path="depMonth" items="${CL_MONTH}" />

**Output HTML**

.. code-block:: html

  <select id="depMonth" name="depMonth">
    <option value="1">01</option>
    <option value="2">02</option>
    <option value="3">03</option>
    <option value="4">04</option>
    <option value="5">05</option>
    <option value="6">06</option>
    <option value="7">07</option>
    <option value="8">08</option>
    <option value="9">09</option>
    <option value="10">10</option>
    <option value="11">11</option>
    <option value="12">12</option>
  </select>

**Output screen**

.. figure:: ./images/codelist_numberrenge.png
   :alt: codelist numberrange
   :width: 5%

|

Using codelist in Java class
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

For details on settings shown below, refer to :ref:`Using codelist in Java class <serverSide>` described earlier.

|

.. _codelist-jdbc:

Using JdbcCodeList
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| ``org.terasoluna.gfw.common.codelist.JdbcCodeList`` is a class for creating codelist by fetching values from DB at the time of starting the application.
| Since ``JdbcCodeList`` creates a cache while starting the application, no delay occurs during DB access when you want to display a list.

| If you want to reduce the read time for the startup, it is preferable to set an upper limit on the number of acquisitions.
| ``JdbcCodeList`` consists of a field which sets the ``org.springframework.jdbc.core.JdbcTemplate``.
| If an upper limit is set for the ``fetchSize`` of ``JdbcTemplate``, the records only till the upper limit are read at the startup.
| The fetched values can be changed dynamically by reloading. For details, refer to :ref:`codeListTaskScheduler`.

**JdbcCodeList image**

.. figure:: ./images/codelist-jdbc.png
   :alt: codelist simple
   :width: 100%

|

Example of codelist settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**Definition of Table(authority)**

.. tabularcolumns:: |p{0.40\linewidth}|p{0.60\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 40 60

   * - authority_id
     - authority_name
   * - | 01
     - | STAFF_MANAGEMENT
   * - | 02
     - | MASTER_MANAGEMENT
   * - | 03
     - | STOCK_MANAGEMENT
   * - | 04
     - | ORDER_MANAGEMENT
   * - | 05
     - | SHOW_SHOPPING_CENTER

|

**Definition of Bean definition file(xxx-codelist.xml)**

.. code-block:: xml

    <bean id="jdbcTemplateForCodeList" class="org.springframework.jdbc.core.JdbcTemplate" > <!-- (1) -->
        <property name="dataSource" ref="dataSource" />
        <property name="fetchSize" value="${codelist.jdbc.fetchSize:1000}" /> <!-- (2) -->
    </bean>

    <bean id="AbstractJdbcCodeList"
        class="org.terasoluna.gfw.common.codelist.JdbcCodeList" abstract="true"> <!-- (3) -->
        <property name="jdbcTemplate" ref="jdbcTemplateForCodeList" /> <!-- (4) -->
    </bean>

    <bean id="CL_AUTHORITIES" parent="AbstractJdbcCodeList" > <!-- (5) -->
        <property name="querySql"
            value="SELECT authority_id, authority_name FROM authority ORDER BY authority_id" /> <!-- (6) -->
        <property name="valueColumn" value="authority_id" /> <!-- (7) -->
        <property name="labelColumn" value="authority_name" /> <!-- (8) -->
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Define a bean for ``org.springframework.jdbc.core.JdbcTemplate`` class.
       | It is necessary for independently setting the fetchSize.
   * - | (2)
     - | Set the fetchSize.
       | An appropriate value must be set since FetchSize is set to Fetch All by default.
       | When ``fetchSize`` is set to "fetch all" and when the records that are required to be read by ``JdbcCodeList`` are large, process efficiency while fetching a list from DB is likely to reduce resulting in prolonged startup time of application.
   * - | (3)
     - | Define a common bean of JdbcCodeList.
       | Common parts of another ``JdbcCodeList`` are specified. Therefore, the bean is defined in parent class for bean definition of basic ``JdbcCodeList``.
       | An instance cannot be created for this bean by setting abstract attribute to true.
   * - | (4)
     - | Specify ``jdbcTemplate`` set in (1).
       | ``JdbcTemplate`` which specifies ``fetchSize`` is stored in ``JdbcCodeList``.
   * - | (5)
     - | Bean definition of JdbcCodeList
       | By setting Bean defined in (3) as parent class in parent attribute, ``JdbcCodeList`` which specifies ``fetchSize`` is set.
       | In this bean definition, only the query related settings are carried out and the required CodeList is created.
   * - | (6)
     - | Write an SQL to be fetched in querySql property. At that time, **always specify "ORDER BY" clause and determine the order**.
       | If "ORDER BY" is not specified, the order gets changed while fetching the records.
   * - | (7)
     - | Set the value corresponding to the Key of Map in valueColumn property. In this example, authority_id is set.
   * - | (8)
     - | Set the value corresponding to Value of Map in labelColumn property. In this example, authority_name is set.      

|

Using codelist in JSP
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| For details on settings shown below, refer to :ref:`Using codelist in JSP <clientSide>` described earlier.

**Example of jsp implementation**

.. code-block:: jsp

  <form:checkboxes items="${CL_AUTHORITIES}"/>

**Output HTML**

.. code-block:: html

  <span>
    <input id="authorities1" name="authorities" type="checkbox" value="01"/>
    <label for="authorities1">STAFF_MANAGEMENT</label>
  </span>
  <span>
    <input id="authorities2" name="authorities" type="checkbox" value="02"/>
    <label for="authorities2">MASTER_MANAGEMENT</label>
  </span>
  <span>
    <input id="authorities3" name="authorities" type="checkbox" value="03"/>
    <label for="authorities3">STOCK_MANAGEMENT</label>
  </span>
  <span>
    <input id="authorities4" name="authorities" type="checkbox" value="04"/>
    <label for="authorities4">ORDER_MANAGEMENT</label>
  </span>
  <span>
    <input id="authorities5" name="authorities" type="checkbox" value="05"/>
    <label for="authorities5">SHOW_SHOPPING_CENTER</label>
  </span>

**Output screen**

.. figure:: ./images/codelist_checkbox.png
   :alt: codelist checkbox
   :width: 50%

|

Using codelist in Java class
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

For details on settings shown below, refer to :ref:`Using codelist in Java class <serverSide>` described earlier.

|

.. _codelist-enum:

How to use EnumCodeList
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
\ ``org.terasoluna.gfw.common.codelist.EnumCodeList``\  is a class
for creating codelist from constant defined in \ ``Enum``\  class.

.. note::

    In case of handling codelist in applications that match with the following conditions,
    it should be analyzed if the codelist label can be stored in \ ``Enum``\  class using \ ``EnumCodeList``\  .
    By storing codelist label in \ ``Enum``\  class,
    the information and operations linked with code values can be aggregated in \ ``Enum``\  class.

    * It is necessary to store the code values in \ ``Enum``\  class (i.e. the process needs to be performed considering code values in Java logic)
    * Internationalization (multilingualization) of UI is not required

|

Image of using \ ``EnumCodeList``\  is shown below.

.. figure:: ./images/codelist-enum.png
   :alt: codelist enum
   :width: 100%

.. note::

    In \ ``EnumCodeList``\ , \ ``org.terasoluna.gfw.common.codelist.EnumCodeList.CodeListItem``\  interface
    is provided to fetch the information (code values and labels) required for creating codelist from \ ``Enum``\  class.

    In case of using \ ``EnumCodeList``\ , \ ``EnumCodeList.CodeListItem``\  interface should be implemented in \ ``Enum``\  class to be created.

|

Example of codelist settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**Creating Enum class**

In case of using \ ``EnumCodeList``\ ,
create \ ``Enum``\  class that implements \ ``EnumCodeList.CodeListItem``\  interface.
Example is shown below.

.. code-block:: java

    package com.example.domain.model;

    import org.terasoluna.gfw.common.codelist.EnumCodeList;

    public enum OrderStatus
        // (1)
        implements EnumCodeList.CodeListItem {

        // (2)
        RECEIVED  ("1", "Received"),
        SENT      ("2", "Sent"),
        CANCELLED ("3","Cancelled");

        // (3)
        private final String value;
        private final String label;

        // (4)
        private OrderStatus(String codeValue, String codeLabel) {
            this.value = codeValue;
            this.label = codeLabel;
        }

        // (5)
        @Override
        public String getCodeValue() {
            return value;
        }

        // (6)
        @Override
        public String getCodeLabel() {
            return label;
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - In \ ``Enum``\  class to be used as codelist,
        implement the \ ``org.terasoluna.gfw.common.codelist.EnumCodeList.CodeListItem``\  interface provided by common library.

        In \ ``EnumCodeList.CodeListItem``\  interface, following methods are defined to fetch the information (code values and labels) required for creating a codelist.

        * \ ``getCodeValue()``\  method to fetch code values
        * \ ``getCodeLabel()``\  method to fetch labels

    * - | (2)
      - Define constants.

        When creating constants, specify the information (code values and labels) required for creating a codelist.

        In above example, following 3 constants are defined. 

        * \ ``RECEIVED``\  (code value=\ ``"1"``\ , label=\ ``"Received"``\ )
        * \ ``SENT``\  (code value=\ ``"2"``\ , label=\ ``"Sent"``\ )
        * \ ``CANCELLED``\  (code value=\ ``"3"``\ , label=\ ``"Cancelled"``\ )

        .. note::

            Sorting order of codelist when using \ ``EnumCodeList``\  will be the order of defining constants.

    * - | (3)
      - Create a property to store the information (code values and labels) required for creating a codelist.
    * - | (4)
      - Create a constructor to receive the information (code values and labels) required for creating a codelist.
    * - | (5)
      - Return the code values storing constants.

        This method is defined in \ ``EnumCodeList.CodeListItem``\  interface, and
        it is called when \ ``EnumCodeList``\  fetches code value from a constant.
    * - | (6)
      - Return the label storing constants.

        This method is defined in \ ``EnumCodeList.CodeListItem``\  interface, and
        it is called when \ ``EnumCodeList``\  fetches label from a constant.


|

**Definition of bean definition file (xxx-codelist.xml)**

\ ``EnumCodeList``\  is defined in bean definition file for codelist.
Example of definition is shown below.

.. code-block:: xml

    <bean id="CL_ORDERSTATUS"
          class="org.terasoluna.gfw.common.codelist.EnumCodeList"> <!-- (7) -->
        <constructor-arg value="com.example.domain.model.OrderStatus" /> <!-- (8) -->
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (7)
      - Specify \ ``EnumCodeList``\  class as codelist implementation class.
    * - | (8)
      - Specify FQCN of \ ``Enum``\  class that implements \ ``EnumCodeList.CodeListItem``\  interface in constructor of \ ``EnumCodeList``\  class.

|

Using codelist in JSP
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

For details on how to use codelist in JSP, refer to :ref:`clientSide` described earlier.


|

Using codelist in Java class
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

For details on how to use codelist in Java class, 
refer to :ref:`serverSide` described earlier.

|

.. _codelisti18n:

How to use SimpleI18nCodeList
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``org.terasoluna.gfw.common.codelist.i18n.SimpleI18nCodeList`` is a codelist supporting internationalization.
By setting the codelist for each locale, the codelist corresponding to locale can be returned.

**SimpleI18nCodeList image**

.. figure:: ./images/codelist-i18n.png
   :alt: codelist i18n
   :width: 100%

|

Example of codelist settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

It is easier to understand if you consider \ ``SimpleI18nCodeList``\  as two dimensional table wherein row is \ ``Locale``\ , column contains code values and cell details are labels.

The table would be as follows in case of a selectbox for selecting charges.

.. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|
.. list-table::
   :header-rows: 1
   :stub-columns: 1
   :widths: 10 15 15 15 15 15 15

   * - row=Locale,column=Code
     - 0
     - 10000
     - 20000
     - 30000
     - 40000
     - 50000
   * - en
     - unlimited
     - Less than \\10,000
     - Less than \\20,000
     - Less than \\30,000
     - Less than \\40,000
     - Less than \\50,000
   * - ja
     - 上限なし
     - 10,000円以下
     - 20,000円以下
     - 30,000円以下
     - 40,000円以下
     - 50,000円以下



For creating a codelist table that supports internationalization, \ ``SimpleI18nCodeList``\  has been set in following 3 ways.

* Set \ ``CodeList``\  for each locale by rows.
* Set \ ``java.util.Map``\ (key = code value, value = label) for each locale by rows.
* Set \ ``java.util.Map``\ (key = locale, value = label) for each code value by columns.

It is recommended that you set the codelist using "Set \ ``CodeList``\  for each locale by rows." method.

The way of setting the \ ``CodeList``\  for each locale by rows considering the above example of selectbox for selecting charges, is mentioned below.
For other setting methods, refer to :ref:`afterCodelisti18n`.

|

**Definition of Bean definition file (xxx-codelist.xml)**

.. code-block:: xml
  
    <bean id="CL_I18N_PRICE"
        class="org.terasoluna.gfw.common.codelist.i18n.SimpleI18nCodeList">
        <property name="rowsByCodeList"> <!-- (1) -->
            <util:map>
                <entry key="en" value-ref="CL_PRICE_EN" />
                <entry key="ja" value-ref="CL_PRICE_JA" />
            </util:map>
        </property>
    </bean>
  
.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
  
    * - Sr. No.
      - Description
    * - | (1)
      - | Set Map wherein key is \ ``java.lang.Locale``\ , in rowsByCodeList properties.
        | In Map, specify locale in key and a reference link to codelist class corresponding to locale in value-ref.
        | For Map values, refer to codelist class corresponding to each locale.

|

**Definition of Bean definition file(xxx-codelist.xml) when creating SimpleMapCodeList for each locale**

.. code-block:: xml
  
    <bean id="CL_I18N_PRICE"
        class="org.terasoluna.gfw.common.codelist.i18n.SimpleI18nCodeList">
        <property name="rowsByCodeList">
            <util:map>
                <entry key="en" value-ref="CL_PRICE_EN" />
                <entry key="ja" value-ref="CL_PRICE_JA" />
            </util:map>
        </property>
    </bean>
  
    <bean id="CL_PRICE_EN" class="org.terasoluna.gfw.common.codelist.SimpleMapCodeList">  <!-- (2) -->
        <property name="map">
            <util:map>
                <entry key="0" value="unlimited" />
                <entry key="10000" value="Less than \\10,000" />
                <entry key="20000" value="Less than \\20,000" />
                <entry key="30000" value="Less than \\30,000" />
                <entry key="40000" value="Less than \\40,000" />
                <entry key="50000" value="Less than \\50,000" />
            </util:map>
        </property>
    </bean>
  
    <bean id="CL_PRICE_JA" class="org.terasoluna.gfw.common.codelist.SimpleMapCodeList">  <!-- (3) -->
        <property name="map">
            <util:map>
                <entry key="0" value="上限なし" />
                <entry key="10000" value="10,000円以下" />
                <entry key="20000" value="20,000円以下" />
                <entry key="30000" value="30,000円以下" />
                <entry key="40000" value="40,000円以下" />
                <entry key="50000" value="50,000円以下" />
            </util:map>
        </property>
    </bean>
  
.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
  
    * - Sr. No.
      - Description
    * - | (2)
      - | For bean definition ``CL_PRICE_EN`` where locale is "en", codelist class is set in ``SimpleMapCodeList``.
    * - | (3)
      - | For bean definition ``CL_PRICE_JA`` where locale is "ja", codelist class is set in ``SimpleMapCodeList``.

|

**Definition of Bean definition file(xxx-codelist.xml) when creating JdbcCodeList for each locale**

.. code-block:: xml
  
    <bean id="CL_I18N_PRICE"
        class="org.terasoluna.gfw.common.codelist.i18n.SimpleI18nCodeList">
        <property name="rowsByCodeList">
            <util:map>
                <entry key="en" value-ref="CL_PRICE_EN" />
                <entry key="ja" value-ref="CL_PRICE_JA" />
            </util:map>
        </property>
    </bean>
  
    <bean id="CL_PRICE_EN" parent="AbstractJdbcCodeList">  <!-- (4) -->
        <property name="querySql"
            value="SELECT code, label FROM price WHERE locale = 'en' ORDER BY code" />
        <property name="valueColumn" value="code" />
        <property name="labelColumn" value="label" />
    </bean>
  
    <bean id="CL_PRICE_JA" parent="AbstractJdbcCodeList">  <!-- (5) -->
        <property name="querySql"
            value="SELECT code, label FROM price WHERE locale = 'ja' ORDER BY code" />
        <property name="valueColumn" value="code" />
        <property name="labelColumn" value="label" />
    </bean>
  
.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
  
    * - Sr. No.
      - Description
    * - | (4)
      - | For bean definition ``CL_PRICE_EN`` where locale is "en", codelist class is set in ``JdbcCodeList``.
    * - | (5)
      - | For bean definition ``CL_PRICE_JA`` where locale is "ja", codelist class is set in ``JdbcCodeList``.
  

Insert the following data in Table Definition (price table).

.. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 20 60
  
    * - locale
      - code
      - label
    * - | en
      - | 0
      - | unlimited
    * - | en
      - | 10000
      - | Less than \\10,000
    * - | en
      - | 20000
      - | Less than \\20,000
    * - | en
      - | 30000
      - | Less than \\30,000
    * - | en
      - | 40000
      - | Less than \\40,000
    * - | en
      - | 50000
      - | Less than \\50,000
    * - | ja
      - | 0
      - | 上限なし
    * - | ja
      - | 10000
      - | 10,000円以下
    * - | ja
      - | 20000
      - | 20,000円以下
    * - | ja
      - | 30000
      - | 30,000円以下
    * - | ja
      - | 40000
      - | 40,000円以下
    * - | ja
      - | 50000
      - | 50,000円以下

.. warning::

    Currently ``SimpleI18nCodeList`` does not support reloadable functionality.
    It should be noted that even if ``JdbcCodeList`` (reloadable CodeList) referred by ``SimpleI18nCodeList`` is reloaded, it does not get reflected in ``SimpleI18nCodeList``.
    In order to make it reloadable, it should be implemented independently.
    For implementation method, refer to :ref:`originalCustomizeCodeList`.

|

Using codelist in JSP
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Description of basic settings is omitted since it is same as :ref:`Using codelist in JSP <clientSide>` described earlier.

**Definition of Bean definition file(spring-mvc.xml)**

.. code-block:: xml

    <mvc:interceptors>
      <mvc:interceptor>
        <mvc:mapping path="/**" />
        <bean
          class="org.terasoluna.gfw.web.codelist.CodeListInterceptor">
          <property name="codeListIdPattern" value="CL_.+" />
          <property name="fallbackTo" value="en" />  <!-- (1) -->
        </bean>
      </mvc:interceptor>

      <!-- omitted -->

    </mvc:interceptors>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | When request locale is not defined in codelist,
       | codelist is fetched using the locale set in fallbackTo property.
       | When fallbackTo property is not set, default JVM locale is used as fallbackTo property.
       | When codelist cannot be fetched even after using the locale set in fallbackTo property, WARN log is output and empty Map is returned.

|

**Example of jsp implementation**

.. code-block:: jsp

  <form:select path="basePrice" items="${CL_I18N_PRICE}" />

**Output HTML lang=en**

.. code-block:: html

  <select id="basePrice" name="basePrice">
    <option value="0">unlimited</option>
    <option value="1">Less than \\10,000</option>
    <option value="2">Less than \\20,000</option>
    <option value="3">Less than \\30,000</option>
    <option value="4">Less than \\40,000</option>
    <option value="5">Less than \\50,000</option>
  </select>

**Output HTML lang=ja**

.. code-block:: html

  <select id="basePrice" name="basePrice">
    <option value="0">上限なし</option>
    <option value="1">10,000円以下</option>
    <option value="2">20,000円以下</option>
    <option value="3">30,000円以下</option>
    <option value="4">40,000円以下</option>
    <option value="5">50,000円以下</option>
  </select>

**Output screen lang=en**

.. figure:: ./images/codelist_i18n_en.png
   :alt: codelist i18n en
   :width: 20%

**Output screen lang=ja**

.. figure:: ./images/codelist_i18n_ja.png
   :alt: codelist i18n ja
   :width: 20%

|

Using codelist in Java class
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Description of basic settings is omitted since it is same as :ref:`Using codelist in Java class <serverSide>` described earlier.

.. code-block:: java

    @RequestMapping("orders")
    @Controller
    public class OrderController {

        @Inject
        @Named("CL_I18N_PRICE")
        I18nCodeList priceCodeList;

        // ...

        @RequestMapping(method = RequestMethod.POST, params = "confirm")
        public String confirm(OrderForm form, Locale locale) {
            // ...
            String priceMassage = getPriceMessage(form.getPriceCode(), locale);
            // ...
        }

        private String getPriceMessage(String targetPrice, Locale locale) {
             return priceCodeList.asMap(locale).get(targetPrice);  // (1)
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Map of locale corresponding to I18nCodeList#asMap(Locale) can be fetched.

|

.. _codelist-display-label:

Display code name corresponding to code value
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When it is necessary to refer the codelist in JSP, it can be referred same as \ ``java.util.Map``\  interface.

For details, see the below example.

**Example of jsp implementation**

.. code-block:: jsp

    Order Status : ${f:h(CL_ORDERSTATUS[orderForm.orderStatus])}

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Get a codelist that has been converted to the \ ``java.util.Map``\  from the request scope (In this example, \ ``"CL_ORDERSTATUS"``\  used as codelist ). The codelist has been referred with the beanID of codelist.
       | Then specify a code value as a key of the \ ``Map``\  interface which displays a corresponding code name (In this example, \ ``orderStatus``\  value is used as a key).

|

.. _codelist-validate:

Input validation of code value using codelist
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When checking whether the input value is the code value defined in codelist,
``org.terasoluna.gfw.common.codelist.ExistInCodeList`` annotation for BeanValidation is provided in common library.

For details on BeanValidation and message output method, refer to :doc:`Validation`.

For input validation using \ ``@ExistInCodeList``\  annotation,
it is necessary to carry out ":ref:`Validation_message_def`" for \ ``@ExistInCodeList``\  .

When project is created by `Blank project <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank>`_ \ ,
the following message is defined in \ ``ValidationMessages.properties``\  file directly under \ ``xxx-web/src/main/resources``\ .
Please change the message to fit the application requirements.

.. code-block:: properties

    org.terasoluna.gfw.common.codelist.ExistInCodeList.message = Does not exist in {codeListId}

.. note::

    In the terasoluna-gfw-common 5.0.0.RELEASE or later,
    the format of message property key has been changed to standard format of Bean Validation (FQCN of annotation + \ ``.message``\ ).

     .. tabularcolumns:: |p{0.40\linewidth}|p{0.60\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 40 60
 
        * - Version
          - Property key of message
        * - | version 5.0.0.RELEASE or later
          - | ``org.terasoluna.gfw.common.codelist.ExistInCodeList.message``
        * - | version 1.0.x.RELEASE
          - | ``org.terasoluna.gfw.common.codelist.ExistInCodeList``
          
    For migrating to the version 5.0.0.RELEASE or later from the version 1.0.x.RELEASE,
    when message is changed to fit the application requirements,
    the property key should be changed.

.. note::

    From terasoluna-gfw-common 1.0.2.RELEASE, 
    \ ``ValidationMessages.properties``\ wherein \ ``@ExistInCodeList``\ message is defined, 
    is not included in jar file.
    This is to fix the "`Bug in which message is not displayed if multiple ValidationMessages.properties exist <https://github.com/terasolunaorg/terasoluna-gfw/issues/256>`_".

    For migrating to version 1.0.2.RELEASE or later from version 1.0.1.RELEASE or prior, 
    if the message defined in \ ``ValidationMessages.properties``\ included in jar of terasoluna-gfw-common, is used,
    it is necessary to define the message by creating \ ``ValidationMessages.properties``\ .

|

Example of @ExistInCodeList settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

See below the example of input validation method using codelist.

**Definition of Bean definition file(xxx-codelist.xml)**

.. code-block:: xml

    <bean id="CL_GENDER" class="org.terasoluna.gfw.common.codelist.SimpleMapCodeList">
        <property name="map">
            <map>
                <entry key="M" value="Male" />
                <entry key="F" value="Female" />
            </map>
        </property>
    </bean>

**Form object**

.. code-block:: java

    public class Person {
        @ExistInCodeList(codeListId = "CL_GENDER")  // (1)
        private String gender;

        // getter and setter omitted
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Set ``@ExistInCodeList`` annotation for the field for which input is to be validated,
       | and specify the target codelist in codeListId.

As a result of above settings, when characters other than M, F are stored in ``gender``, the system throws an error.

.. tip::

    ``@ExistInCodeList`` input validation supports only ``String`` or ``Character`` data types.
    Therefore, even if the fields with ``@ExistInCodeList`` may contain integer values, they should be defined as String data type. (such as Year/Month/Day)

|

How to extend
--------------------------------------------------------------------------------

.. _codeListTaskScheduler:

When reloading the codelist
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Codelist provided in common library is read at the time of launching the application and it is never updated subsequently.
However, in some cases, when the master data of the codelist is updated, the codelist also needs to be updated.

Example: Updating the codelist when DB master is updated using JdbcCodeList.

Common library provides ``org.terasoluna.gfw.common.codelist.ReloadableCodeList`` interface.
The class implementing the above interface, implements refresh method. Codelist can be updated by calling this refresh method.
JdbcCodeList implements ReloadableCodeList interface; hence it is possible to update the codelist.

Codelist can be updated in following two ways.

#. By using Task Scheduler
#. By calling refresh method in Controller (Service) class

This guideline recommends the method to reload the codelist periodically using \ `Spring Task Scheduler <http://docs.spring.io/spring/docs/4.1.9.RELEASE/spring-framework-reference/html/scheduling.html>`_\ .

However, when it is necessary to arbitrarily refresh the codelist, it is appropriate to call refresh method in Controller class.

.. note::

    For the codelist having ReloadableCodeList interface, refer to :ref:`List of codelist types <listOfCodeList>`.

|

Using Task Scheduler
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Example for setting the Task Scheduler is shown below.

**Definition of Bean definition file(xxx-codelist.xml)**

.. code-block:: xml

    <task:scheduler id="taskScheduler" pool-size="10"/>  <!-- (1) -->

    <task:scheduled-tasks scheduler="taskScheduler">  <!-- (2) -->
        <task:scheduled ref="CL_AUTHORITIES" method="refresh" cron="${cron.codelist.refreshTime}"/>  <!-- (3) -->
    </task:scheduled-tasks>

    <bean id="CL_AUTHORITIES" parent="AbstractJdbcCodeList">
        <property name="querySql"
            value="SELECT authority_id, authority_name FROM authority ORDER BY authority_id" />
        <property name="valueColumn" value="authority_id" />
        <property name="labelColumn" value="authority_name" />
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify the thread pool size in pool-size attribute of ``<task:scheduler>`` element.
       | When pool-size attribute is not specified, the value is set to "1".
   * - | (2)
     - | Define ``<task:scheduled-tasks>`` element and set ``<task:scheduler>`` ID in scheduler attribute.
   * - | (3)
     - | Define ``<task:scheduled>`` element. Specify refresh method in method attribute.
       | In cron attribute, the value should be mentioned in ``org.springframework.scheduling.support.CronSequenceGenerator`` supported format.
       | Reload timing for cron attribute may change with development environment and commercial environment; hence it is recommended to fetch the codelist from property file or environment variable.
       |
       | **Example of setting cron attribute**
       | Specify in "Seconds Minutes Hours Month Year Day".
       | execution every second                      "\* \* \* \* \* \*"
       | execution every hour                        "0 0 \* \* \* \*"
       | execution every hour 9:00-17:00 on weekdays "0 0 9-17 \* \* MON-FRI"
       |
       | For details, refer to JavaDoc.
       | http://docs.spring.io/spring/docs/4.1.9.RELEASE/javadoc-api/org/springframework/scheduling/support/CronSequenceGenerator.html

|

Calling refresh method in Controller (Service) class
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

See the example below for directly calling refresh method of JdbcCodeList in Service class.


**Definition of Bean definition file(xxx-codelist.xml)**

.. code-block:: xml

    <bean id="CL_AUTHORITIES" parent="AbstractJdbcCodeList">
        <property name="querySql"
            value="SELECT authority_id, authority_name FROM authority ORDER BY authority_id" />
        <property name="valueColumn" value="authority_id" />
        <property name="labelColumn" value="authority_name" />
    </bean>

**Controller class**

.. code-block:: java

    @Controller
    @RequestMapping(value = "codelist")
    public class CodeListContoller {

        @Inject
        CodeListService codeListService; // (1)

        @RequestMapping(method = RequestMethod.GET, params = "refresh")
        public String refreshJdbcCodeList() {
            codeListService.refresh(); // (2)
            return "codelist/jdbcCodeList";
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Inject the Service class that executes refresh method of ReloadableCodeList class.
   * - | (2)
     - | Execute the refresh method of Service class that executes refresh method of ReloadableCodeList class.

**Service class**

The description below is given only for the implementation class. Description for interface class has been omitted.

.. code-block:: java

  @Service
  public class CodeListServiceImpl implements CodeListService { // (3)

      @Inject
      @Named(value = "CL_AUTHORITIES") // (4)
      ReloadableCodeList codeListItem; // (5)

      @Override
      public void refresh() { // (6)
          codeListItem.refresh(); // (7)
      }
  }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr. No.
     - Description
   * - | (3)
     - | Implement ``CodeListService`` interface for ``CodeListServiceImpl`` class.
   * - | (4)
     - | Specify the corresponding codelist using ``@Named`` annotation at the time of injecting the codelist.
       | ID of the bean to be fetched should be specified in ``value`` attribute.
       | Codelist of ID attribute "CL_AUTHORITIES" of bean tag defined in Bean definition file is injected.
   * - | (5)
     - | ReloadableCodeList interface should be defined in field type.
       | ReloadableCodeList interface should be implemented for Bean specified in (4).
   * - | (6)
     - | refresh method defined in Service class
       | is called from Controller class.
   * - | (7)
     - | refresh method of codelist wherein ReloadableCodeList interface is implemented.
       | Codelist is updated by executing refresh method.

|

.. _originalCustomizeCodeList:

Customizing the codelist independently
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In order to create a codelist which does not fall under the 4 types provided by the common library, the existing codelist can be customized independently.
Refer to the table below for the implementation method and type of codelist that can be created.

.. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.30\linewidth}|p{0.45\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 15 30 45

   * - Sr. No.
     - Reloadable
     - Class to be inherited
     - Implementation location
   * - | (1)
     - | Not required
     - | ``org.terasoluna.gfw.common.codelist.AbstractCodeList``
     - | Override ``asMap``
   * - | (2)
     - | Required
     - | ``org.terasoluna.gfw.common.codelist.AbstractReloadableCodeList``
     - | Override ``retrieveMap``

The codelist can be customized by directly implementing ``org.terasoluna.gfw.common.codelist.CodeList`` and ``org.terasoluna.gfw.common.codelist.ReloadableCodeList`` interfaces; however extending the abstract class provided in common library minimizes the implementation efforts.

Actual example of independent customization is shown below.
It illustrates a codelist for creating a list of current year and the next year.
(Example: If current year is 2013, it is stored in codelist in the order of "2013, 2014".)

**Codelist class**

.. code-block:: java

    @Component("CL_YEAR") // (1)
    public class DepYearCodeList extends AbstractCodeList { // (2)

        @Inject
        JodaTimeDateFactory dateFactory; // (3)

        @Override
        public Map<String, String> asMap() {  // (4)
            DateTime dateTime = dateFactory.newDateTime();
            DateTime nextYearDateTime = dateTime.plusYears(1);

            Map<String, String> depYearMap = new LinkedHashMap<String, String>();

            String thisYear = dateTime.toString("Y");
            String nextYear = nextYearDateTime.toString("Y");
            depYearMap.put(thisYear, thisYear);
            depYearMap.put(nextYear, nextYear);

            return Collections.unmodifiableMap(depYearMap);
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr. No.
     - Description
   * - | (1)
     - | Register the codelist as a component using ``@Component`` annotation.
       | By specifying ``"CL_YEAR"`` in Value, register the codelist as a component using the codelist intercept set in bean definition.
   * - | (2)
     - | Inherit ``org.terasoluna.gfw.common.codelist.AbstractCodeList``.
       | When creating the list of current year and next year, reloading is not necessary since it is created dynamically by calculating from system date.
   * - | (3)
     - | ``org.terasoluna.gfw.common.date.jodatime.JodaTimeDateFactory`` creating the Date class of system date is injected.
       | Current year and next year can be fetched using ``JodaTimeDateFactory``.
       | Class that implements ``JodaTimeDateFactory`` interface should be set in advance in bean definition file.
   * - | (4)
     - | Override ``asMap()`` method and create the list of current year and next year.
       | Implementation differs with every created codelist.

|

**Example of jsp implementation**

.. code-block:: jsp

  <form:select path="mostRecentYear" items="${CL_YEAR}" /> <!-- (5) -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (5)
     - | ``"CL_YEAR"`` registered as component in items attribute should be specified in ``${}`` placeholder to fetch the corresponding codelist.

**Output HTML**

.. code-block:: html

  <select id="mostRecentYear" name="mostRecentYear">
     <option value="2013">2013</option>
     <option value="2014">2014</option>
  </select>

**Output screen**

.. figure:: ./images/codelist_customizeCodelist.png
   :alt: customized codelist
   :width: 10%

.. note::

    Implementation should be made thread-safe at the time of customizing the reloadable CodeList independently.

|

Appendix
--------------------------------------------------------------------------------

.. _afterCodelisti18n:

Setting SimpleI18nCodeList
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Apart from the settings mentioned in :ref:`codelisti18n`, SimpleI18nCodeList can be set in following 2 ways.
The respective setting methods are explained using the example of selectbox for selecting charges.

Set \ ``java.util.Map``\  (key = code value, value = label) for each locale by rows
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**Definition of Bean definition file(xxx-codelist.xml)**

.. code-block:: xml

    <bean id="CL_I18N_PRICE"
        class="org.terasoluna.gfw.common.codelist.i18n.SimpleI18nCodeList">
        <property name="rows"> <!-- (1) -->
            <util:map>
                <entry key="en">
                    <util:map>
                        <entry key="0" value="unlimited" />
                        <entry key="10000" value="Less than \\10,000" />
                        <entry key="20000" value="Less than \\20,000" />
                        <entry key="30000" value="Less than \\30,000" />
                        <entry key="40000" value="Less than \\40,000" />
                        <entry key="50000" value="Less than \\50,000" />
                    </util:map>
                </entry>
                <entry key="ja">
                    <util:map>
                        <entry key="0" value="unlimited" />
                        <entry key="10000" value="10,000円以下" />
                        <entry key="20000" value="20,000円以下" />
                        <entry key="30000" value="30,000円以下" />
                        <entry key="40000" value="40,000円以下" />
                        <entry key="50000" value="50,000円以下" />
                    </util:map>
                </entry>
            </util:map>
        </property>
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Set "Map of Map" for rows property. External Map key is \ ``java.lang.Locale``\ .
       | Internal Map key is a code value and value is a label corresponding to locale.

|

Set \ ``java.util.Map``\ (key = locale, value = label) for each code value by columns
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**Definition of Bean definition file(xxx-codelist.xml)**

.. code-block:: xml

    <bean id="CL_I18N_PRICE"
        class="org.terasoluna.gfw.common.codelist.i18n.SimpleI18nCodeList">
        <property name="columns"> <!-- (1) -->
            <util:map>
                <entry key="0">
                    <util:map>
                        <entry key="en" value="unlimited" />
                        <entry key="ja" value="上限なし" />
                    </util:map>
                </entry>
                <entry key="10000">
                    <util:map>
                        <entry key="en" value="Less than \\10,000" />
                        <entry key="ja" value="10,000円以下" />
                    </util:map>
                </entry>
                <entry key="20000">
                    <util:map>
                        <entry key="en" value="Less than \\20,000" />
                        <entry key="ja" value="20,000円以下" />
                    </util:map>
                </entry>
                <entry key="30000">
                    <util:map>
                        <entry key="en" value="Less than \\30,000" />
                        <entry key="ja" value="30,000円以下" />
                    </util:map>
                </entry>
                <entry key="40000">
                    <util:map>
                        <entry key="en" value="Less than \\40,000" />
                        <entry key="ja" value="40,000円以下" />
                    </util:map>
                </entry>
                <entry key="50000">
                    <util:map>
                        <entry key="en" value="Less than \\50,000" />
                        <entry key="ja" value="50,000円以下" />
                    </util:map>
                </entry>
            </util:map>
        </property>
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Set "Map of Map" for columns property. External Map key is a code value.
       | Internal Map key is \ ``java.lang.Locale``\  and value is a label corresponding to locale.


|

.. _CodeListAppendixNumberRangeCodeListVariation:

Variations of NumberRangeCodeList
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create the Descending NumberRangeCodeList
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Example of setting To value < From value is shown below.

**Definition of Bean definition file(xxx-codelist.xml)**

.. code-block:: xml

    <bean id="CL_BIRTH_YEAR"
        class="org.terasoluna.gfw.common.codelist.NumberRangeCodeList">
        <property name="from" value="2013" /> <!-- (1) -->
        <property name="to" value="2000" /> <!-- (2) -->
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify the range start value. Specify a value greater than the one specified in "value" attribute of "to" property.
        | As per this specification, display the values decreased in accordance with the interval in To-From range in descending order.
        | Since interval is not set, default value 1 is applied.
    * - | (2)
      - | Specify the range end value.
        | In this example, since 2000 is specified as range end value; the value is reduced by 1 and stored in descending order from 2013 to 2000.

|

**Example of jsp implementation**

.. code-block:: jsp

  <form:select path="birthYear" items="${CL_BIRTH_YEAR}" />

**Output HTML**

.. code-block:: html

  <select id="birthYear" name="birthYear">
    <option value="2013">2013</option>
    <option value="2012">2012</option>
    <option value="2011">2011</option>
    <option value="2010">2010</option>
    <option value="2009">2009</option>
    <option value="2008">2008</option>
    <option value="2007">2007</option>
    <option value="2006">2006</option>
    <option value="2005">2005</option>
    <option value="2004">2004</option>
    <option value="2003">2003</option>
    <option value="2002">2002</option>
    <option value="2001">2001</option>
    <option value="2000">2000</option>
  </select>

**Output screen**

.. figure:: ./images/codelist_numberrenge2.png
    :alt: codelist numberrenge2

|

Change interval of NumberRangeCodeList
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Example of setting interval value is shown below.

**Definition of Bean definition file(xxx-codelist.xml)**

.. code-block:: xml

    <bean id="CL_BULK_ORDER_QUANTITY_UNIT"
        class="org.terasoluna.gfw.common.codelist.NumberRangeCodeList">
        <property name="from" value="10" />
        <property name="to" value="50" />
        <property name="interval" value="10" /> <!-- (1) -->
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify increment (decrement) value. Then, store the values obtained upon increasing (decreasing) the interval value within From-To range as codelist.
        | In the above example, the values are stored in the order of \ ``10``\,\ ``20``\,\ ``30``\,\ ``40``\,\ ``50``\  in the codelist.

|

**Example of jsp implementation**

.. code-block:: jsp

    <form:select path="quantity" items="${CL_BULK_ORDER_QUANTITY_UNIT}" />

**Output HTML**

.. code-block:: html

    <select id="quantity" name="quantity">
        <option value="10">10</option>
        <option value="20">20</option>
        <option value="30">30</option>
        <option value="40">40</option>
        <option value="50">50</option>
    </select>

**Output screen**

.. figure:: ./images/codelist_numberrenge3.png
    :alt: codelist numberrenge3

.. note::

    If From-To value exceeds the specified range, then the value increased (decreased) in accordance with interval is not stored in the codelist.

    i.e. in case of following definition,

     .. code-block:: xml

        <bean id="CL_BULK_ORDER_QUANTITY_UNIT"
            class="org.terasoluna.gfw.common.codelist.NumberRangeCodeList">
            <property name="from" value="10" />
            <property name="to" value="55" />
            <property name="interval" value="10" />
        </bean>

    5 values of \ ``10``\,\ ``20``\,\ ``30``\,\ ``40``\,\ ``50``\  are stored in the codelist.
    The value of subsequent interval \ ``60``\  and the range threshold value \ ``55``\  are not stored in the codelist.

.. raw:: latex

   \newpage

