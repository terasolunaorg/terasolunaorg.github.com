Screen Layout using Tiles
================================================================================

.. only:: html

 .. contents:: Table of contents
    :depth: 3
    :local:

Overview
--------------------------------------------------------------------------------
| When developing a Web application with common layouts such as header, footer and side menu, maintaining the layouts becomes complicated if the common parts are coded in all JSPs.
| For example, if the header design needs to be modified, the same modifications must be done for all JSPs.

| In JSP development, when the same layout is used in many screens, it is recommended to use `Apache Tiles <http://tiles.apache.org/>`_\  (hereafter referred to as Tiles).
| Reasons for using Tiles are as follows:

#. To eliminate layout errors by designer
#. To reduce redundant codes
#. To change oversized layouts easily

| Tiles can combine different JSPs by defining an integrated screen layout.
| As a result, the need to describe extra code in each JSP file is eliminated, thereby facilitating developer operations.
| For example, if multiple screens have the following layout structure,

 .. figure:: ./images/screen_layout.png
    :alt: screen layout
    :width: 50%
    :align: center

    **Picture - Image of screen layout**


| By using Tiles, one can focus only on creating the body without having to include and specify the sizes of header, menu and footer, in all the screens with the same layout.
| Actual JSP file is as follows:

 .. figure:: ./images/layout_jsp.png
    :alt: layout jsp
    :width: 50%
    :align: center

    **Picture - Image of layout jsp**

Therefore, after configuring the screen layout using Tiles, only the JSP file corresponding to business process (business.jsp) may be created for each screen.

    .. note::

     In some cases, it is better to avoid using Tiles. For example, using Tiles in an error screen is not recommended due to the following reasons.

     * If an error occurs due to Tiles during error screen display, analyzing the errors becomes difficult. (In case of double failure)
     * Tiles Template is not necessarily always used to display screens in the JSP set by the <error-pages> tag of web.xml.

|

.. _TilesLayoutHowToUse:

How to use
--------------------------------------------------------------------------------

pom.xml setting
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| To use Tiles in Maven, following dependency should be added to pom.xml.

.. code-block:: xml

        <dependency>
            <groupId>org.terasoluna.gfw</groupId>
            <artifactId>terasoluna-gfw-recommended-web-dependencies</artifactId><!-- (1) -->
            <type>pom</type><!-- (2) -->
        </dependency>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Add terasoluna-gfw-recommended-web-dependencies defined for the group of web related libraries, to dependency.
   * - | (2)
     - | Dependencies such as terasoluna-gfw-recommended-web-dependencies are defined only in pom file; hence
       | ``<type>pom</type>`` needs to be specified.

|

    .. note::
        In the above setting example, since it is assumed that the dependent library version is managed by the parent project terasoluna-gfw-parent , specifying the version in pom.xml is not necessary.

|

Integration of Spring MVC and Tiles
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| It is advisable to use ``org.springframework.web.servlet.view.tiles3.TilesViewResolver`` for integrating Spring MVC and Tiles.
| Implementation of Spring MVC Controller (returning View name) need not be changed.

How to configure is shown below.

**Defining Bean (ViewResolver, TilesConfigurer)**

- spring-mvc.xml

 .. code-block:: xml

    <mvc:view-resolvers>
        <mvc:tiles /> <!-- (1) -->
        <mvc:jsp prefix="/WEB-INF/views/" /> <!-- (2) -->
    </mvc:view-resolvers>

    <!-- (3) -->
    <mvc:tiles-configurer>
        <mvc:definitions location="/WEB-INF/tiles/tiles-definitions.xml" />
    </mvc:tiles-configurer>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr. No.
     - Description
   * - | (1)
     - Define \ ``TilesViewResolver``\  using \ ``<mvc:tiles>``\  element added from Spring Framework 4.1.

       By defining it above \ ``<mvc:jsp>``\   element, first resolve \ ``View``\  by referring to Tiles definition file (:file:`tiles-definitions.xml`).
       If View name returned from Controller matches with \ ``name``\  attribute pattern of \ ``definition``\  element in Tiles definition file, \ ``View``\  is resolved by \ ``TilesViewResolver``\ .
   * - | (2)
     - Define \ ``InternalResourceViewResolver``\  for JSP using \ ``<mvc:jsp>``\  element added from Spring Framework 4.1.

       By defining it below  \ ``<mvc:tiles>``\  element, resolve \ ``View``\  using "\ ``InternalResourceViewResolver``\  for JSP" only for the View names that could not be resolved using \ ``TilesViewResolver``\ .
       If a JSP file corresponding to View name exists under \ ``/WEB-INF/views/``\  , \ ``View``\  is resolved by \ ``InternalResourceViewResolver``\  for JSP.
   * - | (3)
     - Read Tiles definition file using \ ``<mvc:tiles-configurer>``\  element added from Spring Framework 4.1.

       Specify Tiles definition file in \ ``location``\  attribute of \ ``<mvc:definitions>``\  element.


 .. tip::

    \ ``<mvc:view-resolvers>``\  element is an XML element added from Spring Framework 4.1.
    If \ ``<mvc:view-resolvers>`` \  element is used, it is possible to define \ ``ViewResolver`` \  in a simple way.

    Example of definition when \ ``<bean>``\  element is used in a conventional way is given below.


     .. code-block:: xml
        :emphasize-lines: 1-13

        <bean id="tilesViewResolver"
            class="org.springframework.web.servlet.view.tiles3.TilesViewResolver">
            <property name="order" value="1" />
        </bean>

        <bean id="tilesConfigurer"
            class="org.springframework.web.servlet.view.tiles3.TilesConfigurer">
            <property name="definitions">
                <list>
                    <value>/WEB-INF/tiles/tiles-definitions.xml</value>
                </list>
            </property>
        </bean>

        <bean id="viewResolver"
            class="org.springframework.web.servlet.view.InternalResourceViewResolver">
            <property name="prefix" value="/WEB-INF/views/" />
            <property name="suffix" value=".jsp" />
            <property name="order" value="2" />
        </bean>

    In \ ``order``\ property, specify a value that is lesser than \ ``InternalResourceViewResolver``\  to ensure that it gets a high priority.


**Tiles Definition**

- tiles-definitions.xml

 .. code-block:: guess

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE tiles-definitions PUBLIC
       "-//Apache Software Foundation//DTD Tiles Configuration 3.0//EN"
       "http://tiles.apache.org/dtds/tiles-config_3_0.dtd"> <!-- (1) -->

    <tiles-definitions>
        <definition name="layouts"
            template="/WEB-INF/views/layout/template.jsp"> <!-- (2) -->
            <put-attribute name="header"
                value="/WEB-INF/views/layout/header.jsp" /> <!-- (3) -->
            <put-attribute name="footer"
                value="/WEB-INF/views/layout/footer.jsp" /> <!-- (4) -->
        </definition>

        <definition name="*/*" extends="layouts"> <!-- (5) -->
            <put-attribute name="title" value="title.{1}.{2}" /> <!-- (6) -->
            <put-attribute name="body" value="/WEB-INF/views/{1}/{2}.jsp" /> <!-- (7) -->
        </definition>
    </tiles-definitions>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr. No.
     - Description
   * - | (1)
     - | Define dtd of tiles.
   * - | (2)
     - | Define the parent layout structure.
       | In 'template' attribute, specify the jsp file where layout is defined.
   * - | (3)
     - | Specify the jsp file that defines header.
   * - | (4)
     - | Specify the jsp file that defines footer.
   * - | (5)
     - | Layout definition which is called when it is same as name pattern at the time of 'create' request.
       | Extended layouts definition is also applied.
   * - | (6)
     - | Specify title.
       | Fetch the value from properties incorporated in spring-mvc. (In the following description, it is set in application-messages.properties.)
       | {1},{2} correspond to the 1st and 2nd "*" of "\*/\*" request.
   * - | (7)
     - | Design the location of jsp file that defines the body such that, request path matches with {1} and JSP name matches with {2}.
       | With this, the efforts to describe definition for each request can be saved.

 .. note::

     For the screens where Tiles is not to be applied (error screen etc.), it is necessary to set a file structure that excludes use of Tiles.
     In Blank project, /WEB-INF/views/common/error/xxxError.jsp format is used so that InternalResourceViewResolver can be used (and so that it does not change to the "\*/\*" format) on error screen.

- `application-messages.properties`

 .. code-block:: properties

  title.staff.createForm = Create Staff Information

 .. note::
   For details on message properties file, refer to :doc:`../WebApplicationDetail/MessageManagement`.


Following is the file structure when Tiles is set.

- tiles File Path

 .. figure:: ./images/tiles_filepath.png
   :alt: tiles file path

**Custom tag settings**


Custom tag (TLD) needs to be set to use Tiles.

- /WEB-INF/views/common/include.jsp

 .. code-block:: jsp

  <%@ page session="false"%>
  <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
  <%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt"%>
  <%@ taglib uri="http://www.springframework.org/tags" prefix="spring"%>
  <%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>
  <%@ taglib uri="http://www.springframework.org/security/tags" prefix="sec"%>
  <%@ taglib uri="http://terasoluna.org/functions" prefix="f"%>
  <%@ taglib uri="http://terasoluna.org/tags" prefix="t"%>
  <%@ taglib uri="http://tiles.apache.org/tags-tiles" prefix="tiles"%> <!-- (1) -->
  <%@ taglib uri="http://tiles.apache.org/tags-tiles-extras" prefix="tilesx"%> <!-- (2) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Add a custom tag (TLD) definition for Tiles.
   * - | (2)
     - | Add a custom tag (TLD) definition for Tiles-extras.

For details about custom tags of Tiles, refer to \ `Here <http://tiles.apache.org/framework/tiles-jsp/tagreference.html>`_\ .

.. tip::

    | Tiles version-2 had one taglib, but tiles-extras taglib is added from version-3.
    | useAttribute tag which was available in tiles taglib in version-2 is moved to tiles-extras taglib from version-3, hence should be careful while using.
    | e.g. ) `<tiles:useAttribute>` : version 2 -> `<tilesx:useAttribute>` : version 3


- web.xml

 .. code-block:: xml

    <jsp-config>
        <jsp-property-group>
            <url-pattern>*.jsp</url-pattern>
            <el-ignored>false</el-ignored>
            <page-encoding>UTF-8</page-encoding>
            <scripting-invalid>false</scripting-invalid>
            <include-prelude>/WEB-INF/views/common/include.jsp</include-prelude> <!-- (1) -->
        </jsp-property-group>
    </jsp-config>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Based on web.xml settings, when jsp file (~.jsp) is to be read, include.jsp can be read in advance.

 .. note::

     Custom tag can also be set in template.jsp. However, it is recommended to create custom tag definition in common jsp include file.
     For details, refer to :ref:`view_jsp_include-label`.

**Creating layout**


Create jsp (template) that forms frame of a layout and jsp to be embedded in the layout.

- template.jsp

 .. code-block:: xml

  <!DOCTYPE html>
  <!--[if lt IE 7]> <html class="no-js lt-ie9 lt-ie8 lt-ie7"> <![endif]-->
  <!--[if IE 7]>    <html class="no-js lt-ie9 lt-ie8"> <![endif]-->
  <!--[if IE 8]>    <html class="no-js lt-ie9"> <![endif]-->
  <!--[if gt IE 8]><!-->
  <html class="no-js">
  <!--<![endif]-->
  <head>
  <meta charset="utf-8" />
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
  <meta name="viewport" content="width=device-width" />
  <link rel="stylesheet"
      href="${pageContext.request.contextPath}/resources/app/css/styles.css"
      type="text/css" media="screen, projection">
  <script type="text/javascript">

  </script> <!-- (1) -->
  <c:set var="titleKey"> <!-- (2) -->
      <tiles:insertAttribute name="title" ignore="true" />
  </c:set>
  <title><spring:message code="${titleKey}" text="Create Staff Information" /></title><!-- (3) -->
  </head>
  <body>
      <div id="header">
          <tiles:insertAttribute name="header" /> <!-- (4) -->
      </div>
      <div id="body">
          <tiles:insertAttribute name="body" /> <!-- (5) -->
      </div>
      <div id="footer">
          <tiles:insertAttribute name="footer" /> <!-- (6) -->
      </div>
  </body>
  </html>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr. No.
     - Description
   * - | (1)
     - | Mention the common contents that need to be described, above step (1).
   * - | (2)
     - | Fetch the value of ``title`` specified in step (6) of tiles-definitions.xml and set it to ``titleKey``.
   * - | (3)
     - | Set title.
       | When ``titleKey`` cannot be fetched, display the title defined in text attribute.
   * - | (4)
     - | Read the "header" defined in tiles-definitions.xml.
   * - | (5)
     - | Read the "body" defined in tiles-definitions.xml.
   * - | (6)
     - | Read the "footer" defined in tiles-definitions.xml.


- header.jsp

 .. code-block:: jsp

  <h1>
      <a href="${pageContext.request.contextPath}">Staff Management
          System</a>
  </h1>


- createForm.jsp(example of body section)

    The developer is able to focus only on the body section and describe the same without having to mention the extra source code for header and footer.

 .. code-block:: jsp

  <h2>Create Staff Information</h2>
  <table>
      <tr>
          <td>Staff First Name</td>
          <td><input type="text" /></td>
      </tr>
      <tr>
          <td>Staff Family Name</td>
          <td><input type="text" /></td>
      </tr>
      <tr>
          <td rowspan="5">Staff Authorities</td>
          <td><input type="checkbox" name="sa" value="01" /> Staff
              Management</td>
      </tr>
      <tr>
          <td><input type="checkbox" name="sa" value="02" /> Master
              Management</td>
      </tr>
      <tr>
          <td><input type="checkbox" name="sa" value="03" /> Stock
              Management</td>
      </tr>
      <tr>
          <td><input type="checkbox" name="sa" value="04" /> Order
              Management</td>
      </tr>
      <tr>
          <td><input type="checkbox" name="sa" value="05" /> Show Shopping
              Management</td>
      </tr>
  </table>

  <input type="submit" value="cancel" />
  <input type="submit" value="confirm" />


- footer.jsp

 .. code-block:: jsp

  <p style="text-align: center; background: #e5eCf9;">Copyright &copy;
      20XX CompanyName</p>

.. note::

    Refer :ref:`CreateWebApplicationProjectCustomizeCopyrightOnScreenFooter` for Copyright described in footer.


**Creating Controller**


While creating Controller, when the request is ``<contextPath>/staff/create?form``,
perform the settings such that "staff/createForm" is returned from the Controller.

- StaffCreateController.java

 .. code-block:: java

  @RequestMapping(value = "create", method = RequestMethod.GET, params = "form")
  public String createForm() {
      return "staff/createForm"; // (1)
  }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr. No.
     - Description
   * - | (1)
     - | With staff as {1} and createForm as {2}, fetch the title name from properties and identify the JSP.


**Creating screen**

When ``<contextPath>/staff/create?form`` is called in request,
Tiles construct the layout and create screen, as shown below.

 .. code-block:: xml

    <definition name="layouts"
        template="/WEB-INF/views/layout/template.jsp"> <!-- (1) -->
        <put-attribute name="header"
            value="/WEB-INF/views/layout/header.jsp" /> <!-- (2) -->
        <put-attribute name="footer"
            value="/WEB-INF/views/layout/footer.jsp" /> <!-- (3) -->
    </definition>

    <definition name="*/*" extends="layouts">
      <put-attribute name="title" value="title.{1}.{2}" /> <!-- (4) -->
      <put-attribute name="body"
        value="/WEB-INF/views/{1}/{2}.jsp" /> <!-- (5) -->
    </definition>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr. No.
     - Description
   * - | (1)
     - | In case of corresponding request, "layouts" which is a parent layout is called and template is set to /WEB-INF/views/layout/template.jsp.
   * - | (2)
     - | WEB-INF/views/layout/header.jsp is set in ``header`` within the template /WEB-INF/views/layout/template.jsp.
   * - | (3)
     - | /WEB-INF/views/layout/footer.jsp is set in ``footer`` within the template /WEB-INF/views/layout/template.jsp.
   * - | (4)
     - | With ``title.staff.createForm`` as key, fetch the value from properties incorporated in spring-mvc where staff is {1} and createForm is {2}.
   * - | (5)
     - | /WEB-INF/views/staff/createForm.jsp is set in ``body`` within template/WEB-INF/views/layout/template.jsp with staff as {1} and createForm as {2}.


As a result, it is output to the browser by combining header.jsp, createForm.jsp and footer.jsp in the above template.jsp.

 .. figure:: ./images/tiles_result.png
   :alt: tiles result
   :width: 100%
   :align: center

|

How to extend
--------------------------------------------------------------------------------

Setting multiple layouts
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| When creating actual business application, display layout may be divided depending on business process contents.
| This time, it is assumed that the staff search functionality menu is required to be displayed on left side of the screen.
| Configuration is shown below based on :ref:`TilesLayoutHowToUse`.

**Tiles Definition**

- tiles-definitions.xml

 .. code-block:: guess
   :emphasize-lines: 7-20

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE tiles-definitions PUBLIC
       "-//Apache Software Foundation//DTD Tiles Configuration 3.0//EN"
       "http://tiles.apache.org/dtds/tiles-config_3_0.dtd">

    <tiles-definitions>
        <definition name="layoutsOfSearch"
            template="/WEB-INF/views/layout/templateSearch.jsp"> <!-- (1) -->
            <put-attribute name="header"
                value="/WEB-INF/views/layout/header.jsp" />
            <put-attribute name="menu"
                value="/WEB-INF/views/layout/menu.jsp" />
            <put-attribute name="footer"
                value="/WEB-INF/views/layout/footer.jsp" />
        </definition>

        <definition name="*/search*" extends="layoutsOfSearch"> <!-- (2) -->
            <put-attribute name="title" value="title.{1}.search{2}" /> <!-- (3) -->
            <put-attribute name="body" value="/WEB-INF/views/{1}/search{2}.jsp" /> <!-- (4) -->
        </definition>

        <definition name="layouts"
            template="/WEB-INF/views/layout/template.jsp">
            <put-attribute name="header"
                value="/WEB-INF/views/layout/header.jsp" />
            <put-attribute name="footer"
                value="/WEB-INF/views/layout/footer.jsp" />
        </definition>

        <definition name="*/*" extends="layouts">
            <put-attribute name="title" value="title.{1}.{2}" />
            <put-attribute name="body" value="/WEB-INF/views/{1}/{2}.jsp" />
        </definition>
    </tiles-definitions>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr. No.
     - Description
   * - | (1)
     - | Define the parent layout structure to be added.
       | When using a different layout, ensure that name attribute of definition tag does not duplicate with the existing layout definition i.e. "layouts".
   * - | (2)
     - | Layout definition called when the layout to be added is same as the name pattern at the time of 'create' request.
       | This layout definition is read when the request corresponds to <contextPath>/\*/search\*.
       | Extended layout definition "layoutsOfSearch" is also applied.
   * - | (3)
     - | Specify the title to be used in the layout to be added.
       | Fetch the value from properties incorporated in spring-mvc. (In the following description, it is set to application-messages.properties.)
       | {1} is the 1st "*" of "\*/search\*" of request.
       | It is necessary that {2} starts with "search" as it corresponds to the  "search*" of "\*/search\*" request.
   * - | (4)
     - | Place the jsp file in which the body is defined such that, the request path matches with {1} and JSP file name beginning with "search", matches with {2}.
       | The value of 'value' attribute needs to be changed according to the configuration of JSP file location.

 .. note::

     When multiple requests correspond to name attribute patterns of definition tag, the verification is done sequentially from the top and the very first pattern that matches with the request is applied.
     In the above case, as the request for staff search screen corresponds to multiple patterns, the layout is defined at the top.

- `application-messages.properties`

 .. code-block:: properties
   :emphasize-lines: 2

   title.staff.createForm = Create Staff Information
   title.staff.searchStaff = Search Staff Information # (1)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Message to be added.
       | "staff" is the 1st "*" of "\*/search\*" request.
       | As "searchStaff" corresponds to "search\*" part of "\*/search\*" request, it is necessary that it begins with "search".

**Creating layout**

Create the jsp (template) that forms the frame of the layout and jsp to be embedded in layout.

- templateSearch.jsp

 .. code-block:: xml

  <!DOCTYPE html>
  <!--[if lt IE 7]> <html class="no-js lt-ie9 lt-ie8 lt-ie7"> <![endif]-->
  <!--[if IE 7]>    <html class="no-js lt-ie9 lt-ie8"> <![endif]-->
  <!--[if IE 8]>    <html class="no-js lt-ie9"> <![endif]-->
  <!--[if gt IE 8]><!-->
  <html class="no-js">
  <!--<![endif]-->
  <head>
  <meta charset="utf-8" />
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
  <meta name="viewport" content="width=device-width" />
  <link rel="stylesheet"
      href="${pageContext.request.contextPath}/resources/app/css/styles.css"
      type="text/css" media="screen, projection">
  <script type="text/javascript">

  </script>
  <c:set var="titleKey">
      <tiles:insertAttribute name="title" ignore="true" />
  </c:set>
  <title><spring:message code="${titleKey}" text="Search Staff Information" /></title>
  </head>
  <body>
      <div id="header">
          <tiles:insertAttribute name="header" />
      </div>
      <div id="menu">
          <tiles:insertAttribute name="menu" /> <!-- (1) -->
      </div>
      <div id="body">
          <tiles:insertAttribute name="body" />
      </div>
      <div id="footer">
          <tiles:insertAttribute name="footer" />
      </div>
  </body>
  </html>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr. No.
     - Description
   * - | (1)
     - | Read the "menu" defined in tiles-definitions.xml.
       | Rest is same as :ref:`TilesLayoutHowToUse`.

- styles.css

 .. code-block:: css

  div#menu { /* (1) */
      float: left;
      width: 20%;
  }

  div#searchBody { /* (2) */
      float: right;
      width: 80%;
  }

  div#footer { /* (3) */
      clear: both;
  }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr. No.
     - Description
   * - | (1)
     - | Set the Menu style.
       | Here, Menu Screen is left aligned using float:left and is displayed with 20% width.
   * - | (2)
     - | Set the Body style.
       | Here, the Business Screen is right aligned using float:right and displayed with 80% width.
       | Name is specified as searchBody. This is because duplication in existing layout and name can have an impact on the existing layout style.
   * - | (3)
     - | Set the Footer style.
       | Float effect of menu and body is initialized. By this, the footer is displayed below menu and body.


- header.jsp

  Same as :ref:`TilesLayoutHowToUse`.

- menu.jsp

 .. code-block:: jsp

  <table>
      <tr>
          <td><a href="${pageContext.request.contextPath}/staff/create?form">Create Staff Information</a></td>
      </tr>
      <tr>
          <td><a href="${pageContext.request.contextPath}/staff/search">Search Staff Information</a></td>
      </tr>
  </table>

- searchStaff.jsp (example of body section)

 .. code-block:: jsp

  <h2>Search Staff Information</h2>
  <table>
      <tr>
          <td>Staff First Name</td>
          <td><input type="text" /></td>
      </tr>
      <tr>
          <td>Staff Family Name</td>
          <td><input type="text" /></td>
      </tr>
      <tr>
          <td rowspan="5">Staff Authorities</td>
          <td><input type="checkbox" name="sa" value="01" /> Staff
              Management</td>
      </tr>
      <tr>
          <td><input type="checkbox" name="sa" value="02" /> Master
              Management</td>
      </tr>
      <tr>
          <td><input type="checkbox" name="sa" value="03" /> Stock
              Management</td>
      </tr>
      <tr>
          <td><input type="checkbox" name="sa" value="04" /> Order
              Management</td>
      </tr>
      <tr>
          <td><input type="checkbox" name="sa" value="05" /> Show Shopping
              Management</td>
      </tr>
  </table>

  <input type="submit" value="Search" />

- footer.jsp

  Same as :ref:`TilesLayoutHowToUse`.

**Creating Controller**


While creating Controller, if the request is ``<contextPath>/staff/search``, set such that 
"staff/searchStaff" is returned from the Controller.

- StaffSearchController.java 

 .. code-block:: java

  @RequestMapping(value = "search", method = RequestMethod.GET)
  public String createForm() {
      return "staff/searchStaff"; // (1)
  }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr. No.
     - Description
   * - | (1)
     - | With staff as {1} and searchStaff as {2}, fetch the title name from properties and identify the JSP.


**Creating screen**

When ``<contextPath>/staff/search`` is called in request,
screen is generated through another layout as shown below.


 .. code-block:: xml

    <definition name="layoutsOfSearch"
        template="/WEB-INF/views/layout/templateSearch.jsp"> <!-- (1) -->
        <put-attribute name="header"
            value="/WEB-INF/views/layout/header.jsp" /> <!-- (2) -->
        <put-attribute name="menu"
            value="/WEB-INF/views/layout/menu.jsp" /> <!-- (3) -->
        <put-attribute name="footer"
            value="/WEB-INF/views/layout/footer.jsp" /> <!-- (4) -->
    </definition>

    <definition name="*/search*" extends="layoutsOfSearch"> <!-- (5) -->
        <put-attribute name="title" value="title.{1}.search{2}" /> <!-- (6) -->
        <put-attribute name="body" value="/WEB-INF/views/{1}/search{2}.jsp" /> <!-- (7) -->
    </definition>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr. No.
     - Description
   * - | (1)
     - | In case of a corresponding request, "layoutsOfSearch" which is a parent layout is called and template is set in /WEB-INF/views/layout/templateSearch.jsp.
   * - | (2)
     - | WEB-INF/views/layout/header.jsp is set in ``header`` within the template /WEB-INF/views/layout/templateSearch.jsp.
   * - | (3)
     - | /WEB-INF/views/layout/menu.jsp is set in ``menu`` within the template /WEB-INF/views/layout/templateSearch.jsp.
   * - | (4)
     - | /WEB-INF/views/layout/footer.jsp is set in ``footer`` within the template /WEB-INF/views/layout/templateSearch.jsp.
   * - | (5)
     - | This layout definition is read when the request corresponds to <contextPath>/\*/search\*.
       | In that case, "layoutsOfSearch"  which is a parent layout is also read.
   * - | (6)
     - | With ``title.staff.searchStaff`` as key, fetch the value from properties incorporated in spring-mvc, where staff is {1} and searchStaff is "search{2}".
   * - | (7)
     - | /WEB-INF/views/staff/searchStaff.jsp is set in ``body`` within the template/WEB-INF/views/layout/templateSearch.jsp where staff is {1} and searchStaff is "search{2}".


As a result, it is output to the browser by combining header.jsp, menu.jsp, searchStaff.jsp and footer.jsp in the above templateSearch.jsp file.

 .. figure:: ./images/tiles_result2.png
   :alt: tiles result another template
   :width: 100%
   :align: center
   
.. raw:: latex

   \newpage

