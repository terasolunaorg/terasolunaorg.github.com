Pagination
================================================================================

.. only:: html

 .. contents:: Table of Contents
    :depth: 3
    :local:

Overview
--------------------------------------------------------------------------------

This chapter describes the Pagination functionality wherein the data matching the search criteria is divided into pages and displayed.

| **It is recommended to use pagination functionality when a large amount of data matches the search criteria.**
| Retrieving and displaying a large amount of data at a time on screen may lead to following problems.

* | Memory exhaustion at server side
  | ``java.lang.OutOfMemoryError``  occurs when multiple requests are executed simultaneously.
* | Network load
  | Transferring unnecessary data over the network results in increased network load and thereby may affect the response time of the entire system.
* | Delay in response on screen
  | Server process, network traffic process and client rendering process take time to handle a large amount of data; hence the response on screen may get delayed.

|

Display of list screen at the time of dividing into pages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
When a page is divided using pagination, the screen looks as follows:

 .. figure:: ./images/pagination-overview_screen.png
   :alt: Screen image of Pagination.
   :width: 100%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Display the link to navigate to various pages.
        | On clicking link, send a request to display the corresponding page. JSP tag library to display this area is provided as common library.
    * - | (2)
      - | Display the information related to pagination (total records, total pages and number of displayed pages etc.).
        | Tag library to display this area does not exist; hence it should be implemented separately as JSP processing.

|

Page search
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| For implementing pagination, it is essential to first implement the server-side search processing to make page searching possible.
| It is assumed in this guideline that the mechanism provided by Spring Data is used for page search at server side.

|

.. _pagination_overview_page_springdata:

Page search functionality of Spring Data
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Page search functionality provided by Spring Data is as follows:

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - 1
      - | Extract the information required for page search (location of page to be searched, number of records to be fetched and sort condition) from request parameter and pass the extracted information as objects of ``org.springframework.data.domain.Pageable``  to the argument of Controller.
        | This functionality is provided as ``org.springframework.data.web.PageableHandlerMethodArgumentResolver``  class and is enabled by adding to ``<mvc:argument-resolvers>``  element of :file:`spring-mvc.xml` .
        | For request parameters, refer to ":ref:`Note column <pagination_overview_pagesearch_requestparameter>`".
    * - 2
      - | Save the page information (total records, data of corresponding page, location of page to be searched, number of records to be fetched and sort condition).
        | This functionality is provided as ``org.springframework.data.domain.Page``  interface and ``org.springframework.data.domain.PageImpl``  is provided as default implementation class.
        | **As per specifications, it fetches the required data from Page object in JSP tag library to output pagination link provided by common library.**
    * - 3
      - | When Spring Data JPA is used for database access, the information of corresponding page is returned as ``Page``  object by specifying ``Pageable``  object as an argument of Repository Query method.
        | All the processes such as executing SQL to fetch total records, adding sort condition and extracting data matching the corresponding page are carried out automatically.
        | When MyBatis is used for database access, the process that is automatically carried out in Spring Data JPA needs to be implemented in Java (Service) and SQL mapping file.

.. _pagination_overview_pagesearch_requestparameter:

 .. note:: **Request parameters for page search**

    Request parameters for page search provided by Spring Data are as follows:

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
     .. list-table::
         :header-rows: 1
         :widths: 10 15 75

         * - Sr. No.
           - Parameter name
           - Description
         * - 1.
           - page
           - | Request parameter to specify the location of page to be searched
             | Specify value greater than or equal to 0.
             | As per default setting, page location starts from ``0`` (zero). Hence, specify ``0`` (zero) to fetch the data of the first page and ``1``  (one) to fetch the data of the second page.
         * - 2.
           - size
           - | Request parameter to specify the count of fetched records.
             | Specify value greater than or equal to 1.
             | When value specified is greater than the value in ``maxPageSize``  of ``PageableHandlerMethodArgumentResolver``, ``maxPageSize``  value becomes ``size``  value.
         * - 3.
           - sort
           - | Parameter to specify sort condition (multiple parameters can be specified).
             | Specify the value in ``"{sort item name(,sort order)}"``  format.
             | Specify either ``"ASC"``  or ``"DESC"``  as sort order. When nothing is specified, ``"ASC"``  is used.
             | Multiple item names can be specified using ``","``  separator.
             | For example, when ``"sort=lastModifiedDate,id,DESC&sort=subId"``  is specified as query string, Order By clause ``"ORDER BY lastModifiedDate DESC, id DESC, subId ASC"``  is added to the query.

 .. warning:: **Operations at the time of specifying "size=0" in spring-data-commons 1.6.1.RELEASE**

    spring-data-commons 1.6.1.RELEASE having terasoluna-gfw-common 1.0.0.RELEASE has a bug wherein if ``"size=0"``  is specified, all the records matching the specified condition are fetched.
    As a result, ``java.lang.OutOfMemoryError`` may occur when a large amount of records are fetched.

    This problem is handled using JIRA `DATACMNS-377 <https://jira.springsource.org/browse/DATACMNS-377>`_ of Spring Data Commons and is being resolved in spring-data-commons 1.6.3.RELEASE.
    Post modification, if ``"size<=0"``  is specified, the default value when size parameter is omitted will be applied.
    
    In cases where terasoluna-gfw-common 1.0.0.RELEASE is used, the version should be upgraded to terasoluna-gfw-common 1.0.1.RELEASE or higher version.

 .. warning:: **About operations when invalid values are specified in request parameters of spring-data-commons 1.6.1.RELEASE**

    spring-data-commons 1.6.1.RELEASE having terasoluna-gfw-common 1.0.0.RELEASE has a bug wherein if an invalid value is specified in request parameters for page search (page, size, sort etc.),
    ``java.lang.IllegalArgumentException``  or ``java.lang.ArrayIndexOutOfBoundsException``  occurs and SpringMVC settings when set to default values leads to system error (HTTP status code=500).

    This problem is handled using JIRA `DATACMNS-379 <https://jira.springsource.org/browse/DATACMNS-379>`_ and `DATACMNS-408 <https://jira.springsource.org/browse/DATACMNS-408>`_ and is being resolved in spring-data-commons 1.6.3.RELEASE.
    Post modification, if invalid values are specified, the default value when parameters are omitted will be applied.

    In cases where terasoluna-gfw-common 1.0.0.RELEASE is used, the version should be upgraded to terasoluna-gfw-common 1.0.1.RELEASE or higher version.

 .. note:: **Points to be noted due to the changes in API specifications of Spring Data Commons**

    In case of "terasoluna-gfw-common 5.0.0.RELEASE or later version" dependent spring-data-commons (1.9.1 RELEASE or later),
    there is a change in API specifications of interface for page search functionality (\ ``org.springframework.data.domain.Page``\ ) and class (\ ``org.springframework.data.domain.PageImpl``\  and \ ``org.springframework.data.domain.Sort.Order``\ ).

    Specifically,

    * In \ ``Page``\  interface and \ ``PageImpl``\  class, \ ``isFirst()``\  and \ ``isLast()``\  methods are added in spring-data-commons 1.8.0.RELEASE, and \ ``isFirstPage()``\  and \ ``isLastPage()``\  methods are deleted from spring-data-commons 1.9.0.RELEASE.
    * In \ ``Sort.Order``\  class, \ ``nullHandling``\  property is added in spring-data-commons 1.8.0.RELEASE.

    If deleted API is used, there will be a compilation error and application will have to be modified.
    In addition, when using \ ``Page``\  interface (\ ``PageImpl``\  class) as resource object of REST API,
    that application may also need to be modified, as JSON and XML format get changed.

|

.. _pagination_overview_paginationlink:

Display of pagination link
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| This section describes the pagination link which is output using JSP tag library of common library.

| Style sheet to display pagination link is not provided from common library, hence it should be created in each project.
| Bootstrap v3.0.0 style sheet is applied for the screens used in the explanation below.

|

Structure of pagination link
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Pagination link consists of the following elements.

 .. figure:: ./images/pagination-how_to_use_jsp_pagelink_description.png
   :alt: Structure of the pagination link.
   :width: 90%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Link to navigate to the first page.
    * - | (2)
      - | Link to navigate to the previous page.
    * - | (3)
      - | Link to navigate to the specified page.
    * - | (4)
      - | Link to navigate to the next page.
    * - | (5)
      - | Link to navigate to the last page.

|

Pagination link has the following status.

 .. figure:: ./images/pagination-how_to_use_jsp_pagelink_description_status.png
   :alt: Status of the pagination link.
   :width: 90%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (6)
      - | Status indicating link where operations cannot be performed on the currently displayed page.
        | The status is specifically "Link to navigate to the first page" and "Link to navigate to the previous page" when the first page is displayed and "Link to navigate to the next page" "Link to navigate to the last page" when the last page is displayed.
        | This status is defined as ``"disabled"``  in the JSP tag library of common library.
    * - | (7)
      - | Status indicating currently displayed page.
        | This status is defined as ``"active"``  in the JSP tag library of common library.

|

| HTML to be output using common library is as follows:
| The numbers in figure correspond to serial numbers of "Structure of pagination link" and "Status of pagination link" mentioned above.

- JSP

 .. code-block:: jsp

    <t:pagination page="${page}" />

- HTML to be output

 .. figure:: ./images/pagination-overview_html.png
   :alt: html of the pagination link.
   :width: 90%
   :align: center

|

HTML of pagination link
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
HTML of pagination link to be output using common library is as follows:

- HTML

 .. figure:: ./images/pagination-overview_html_basic.png
   :alt: html structure of the pagination link.
   :width: 100%
   :align: center

- Screen image

 .. figure:: ./images/pagination-overview_html_basic_screen.png
   :alt: screen structure of the pagination link.
   :width: 80%
   :align: center


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.70\linewidth}|p{0.20\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 70 20

    * - Sr. No.
      - Description
      - Default values
    * - | (1)
      - | Elements to combine the components of pagination link.
        | In common library, this part is called "Outer Element" which stores multiple "Inner Elements".
        | The elements to be used can be changed using the parameters of JSP tag library.
      - | ``<ul>``  element
    * - | (2)
      - | Attribute to specify style class of "Outer Element".
        | In common library, this part is called "Outer Element Class". The attribute values are specified using the parameters of JSP tag library.
      - | No specification
    * - | (3)
      - | Elements to configure pagination link.
        | Call this portion as "Inner Element" and maintain ``<a>``  element to send request for navigating the page in common library.
        | The elements can be changed using parameter of JSP tag library.
      - | ``<li>``  elements
    * - | (4)
      - | Attribute to specify style class of "Inner Elements".
        | In common library, this part is called "Inner Element Class". The attribute values are switched during JSP tag library processing according to the location of the displayed page.
      - | Refer to ":ref:`Note column <pagination_overview_paginationlink_innerelementclass>`".
    * - | (5)
      - | Element to send page navigation request.
        | In common library, this part is called "Page Link".
      - | Fixed as `<a>`
    * - | (6)
      - | Attribute to specify URL for page navigation.
        | In common library, this part is called "Page Link URL".
      - | Refer to the following ":ref:`Note column <pagination_overview_paginationlink_pagelinkurl>`".
    * - | (7)
      - | Specify the text to be displayed for page navigation link
        | In common library, this part is called "Page Link Text".
      - | Refer to the following " :ref:`Note column <pagination_overview_paginationlink_pagelinktext>` ".


 .. note:: **About number of "Inner Elements"**

    As per default setting, there are maximum 14 "Inner Elements". Their division is as follows:

    * Link to navigate to the first page : 1
    * Link to navigate to the previous page : 1
    * Link to navigate to the specified page : Maximum 10
    * Link to navigate to the next page : 1
    * Link to navigate to the last page : 1

    The number of "Inner Elements" can be changed by specifying parameters of JSP tag library.


.. _pagination_overview_paginationlink_innerelementclass:

 .. note:: **About setting values of "Inner Element Class"**

    As per default setting, following are the three values depending on location of the page.

    * ``"disabled"`` : Style class indicating the link which cannot be operated on the currently displayed page.
    * ``"active"`` : Style class indicating the link of currently displayed page.
    * No specification : Indicating the link other than those mentioned above.

    ``"disabled"``  and  ``"active"``  values can be changed by specifying the parameters of JSP tag library.


.. _pagination_overview_paginationlink_pagelinkurl:

 .. note:: **About default values of "Page Link URL"**

    When link status is \ ``"disabled"``\  and \ ``"active"``\ , the default value is ``"javascript:void(0)"``  and in other cases, the default value is ``"?page={page}&size={size}"``.

    "Page Link URL" can be changed to another value by specifying parameters of JSP tag library.

    From terasoluna-gfw-web 5.0.0.RELEASE, default value of link in \ ``"active"``\ status is changed from \ ``"?page={page}&size={size}"``\  to \ ``"javascript:void(0)"``\ .
    This is to match with the implementation of pagination links of major Web sites and the implementation of major CSS libraries (Bootstrap, etc.)

.. _pagination_overview_paginationlink_pagelinktext:

 .. note:: **About default values of "Page Link Text"**

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.50\linewidth}|p{0.30\linewidth}|
     .. list-table::
         :header-rows: 1
         :widths: 10 50 30

         * - Sr. No.
           - Link name
           - Default values
         * - 1.
           - Link to navigate to the first page
           - ``"<<"``
         * - 2.
           - Link to navigate to the previous page
           - ``"<"``
         * - 3.
           - Link to navigate to the specified page
           - | Page number of the corresponding page
             | (cannot be changed)
         * - 4.
           - Link to navigate to the next page
           - ``">"``
         * - 5.
           - Link to navigate to the last page
           - ``">>"``

    Links other than "Link to navigate to the specified page" can be changed as per the specification of parameters of JSP tag library.

|

.. _pagination_overview_paginationlink_taglibparameters:

Parameters of JSP tag library
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Default operations can be changed by specifying values in parameters of JSP tag library.

List of parameters is shown below.

**Parameters to control layout**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.25\linewidth}|p{0.65\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 25 65

    * - Sr. No.
      - Parameter name
      - Description
    * - 1.
      - outerElement
      - | Specify HTML element name to be used as "Outer Element".
        | Example: div
    * - 2.
      - outerElementClass
      - | Specify class name of style sheet to be set in "Outer Element Class".
        | Example: pagination
    * - 3.
      - innerElement
      - | Specify HTML element name to be used as "Inner Element".
        | Example: span
    * - 4.
      - disabledClass
      - | Specify the value to be set in the class attribute of "Inner Element" with ``"disabled"``  status.
        | Example: hiddenPageLink
    * - 5.
      - activeClass
      - | Specify the value to be set in the class attribute of "Inner Element" with ``"active"``  status.
        | Example: currentPageLink
    * - 6.
      - firstLinkText
      - | Specify the value to be set in "Page Link Text" of "Link to navigate to the first page".
        | If ``""``  is specified, "Link to navigate to the first page" itself is not output.
        | Example: First
    * - 7.
      - previousLinkText
      - | Specify the value to be set in "Page Link Text" of "Link to navigate to the previous page".
        | If ``""``  is specified, "Link to navigate to the previous page" itself is not output.
        | Example: Prev
    * - 8.
      - nextLinkText
      - | Specify the value to be set in "Page Link Text" of "Link to navigate to the next page".
        | If ``""``  is specified, "Link to navigate to the next page" itself is not output.
        | Example: Next
    * - 9.
      - lastLinkText
      - | Specify the value to be set in "Page Link Text" of "Link to navigate to the last page".
        | If ``""``  is specified, "Link to navigate to the next page" itself is not output.
        | Example: Last
    * - 10.
      - maxDisplayCount
      - | Specify maximum display count for "Link to navigate to the specified page".
        | If ``0``  is specified, "Link to navigate to the specified page" itself is not output.
        | Example: 5

|

 When default values of all parameters to control the layout are changed, the following HTML is output.
 The numbers in figure correspond to serial numbers in the parameter list mentioned above.

 - JSP

  .. code-block:: jsp

    <t:pagination page="${page}"
        outerElement="div"
        outerElementClass="pagination"
        innerElement="span"
        disabledClass="hiddenPageLink"
        activeClass="currentPageLink"
        firstLinkText="First"
        previousLinkText="Prev"
        nextLinkText="Next"
        lastLinkText="Last"
        maxDisplayCount="5"
        />

 - Output HTML

  .. figure:: ./images/pagination-overview_html_changed.png
   :alt: html of the pagination link(changed layout).
   :width: 100%
   :align: center

|

**Parameters to control operations**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.25\linewidth}|p{0.65\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 25 65

    * - Sr. No.
      - Parameter name
      - Description
    * - 1.
      - disabledHref
      - | Specify the value to be set in "Page Link URL" having ``"disabled"`` state.
    * - 2.
      - pathTmpl
      - | Specify the template of request path to be set in "Page Link URL".
        | When request path at the time of page display and the request path for page navigation are different, request path for page navigation needs to be specified in this parameter.
        | In the template of request path to be specified, location of page (page) and number of records to be fetched (size) can be specified as path variables (placeholders).
        | The specified value of URL is encoded in UTF-8.
    * - 3.
      - queryTmpl
      - | Specify the template of query string of "Page Link URL".
        | Specify the template for generating pagination related query string (page, size, sort parameters) required at the time of page navigation.
        | When setting request parameter name for location of page or the number of records to be fetched to values other than default values, the query string needs to be specified in this parameter.
        | In the template of query string to be specified, location of page (page) and number of records to be fetched (size) can be specified as path variables (placeholders).
        | The specified value of URL is encoded in UTF-8.
        |
        | This attribute is used to generate pagination related query string (page, size, sort parameters); hence query string for adding search conditions should be specified in criteriaQuery attribute.
    * - 4.
      - criteriaQuery
      - | Specify the query string for search conditions to be added to "Page Link URL".
        | **Specify the query string for search conditions in this parameter when adding search conditions to "Page Link URL".**
        | **The specified value of URL is not encoded; hence encoded URL query string needs to be specified.**
        |
        | If EL function (\ ``f:query(Object)``\ ) of common library is used when converting the search conditions stored in form object into encoded URL query string, the search conditions can be added easily.
        |
        | This parameter can be used in terasoluna-gfw-web 1.0.1.RELEASE or higher version.
    * - 5.
      - disableHtmlEscapeOfCriteriaQuery
      - | Flag to disable HTML escaping for the values specified in \ ``criteriaQuery``\  parameter.
        | When the flag is set to \ ``true``\ , HTML escaping is no longer possible for the values specified in \ ``criteriaQuery``\  parameter. (Default value is \ ``false``\ ).
        | **When specifying true, it should be ensured that the characters vulnerable to XSS are not included in the query string.**
        |
        | This parameter can be used in terasoluna-gfw-web 1.0.1.RELEASE or higher version.
    * - 6.
      - enableLinkOfCurrentPage
      - | Flag to send request for redisplaying the corresponding page on clicking page link in ``"active"``\  status.
        | When it is set to \ ``true``\ , URL (default value is \ ``"?page={page}&size={size}"``\ ) to redisplay the corresponding page is set to "Page Link URL". (If default value is \ ``false``\ , the value of \ ``disabledHref``\  attribute is set to "Page Link URL")
        |
        | This parameter can be used in terasoluna-gfw-web 5.0.0.RELEASE or higher version. 
    
 .. note:: **About setting values of disabledHref**

    \ ``"javascript:void(0)"``\ is set in \ ``disabledHref``\  attribute by default.
    It may remain as default in order to disable only the operation of page link click.

    However, if the focus is moved or on mouseover to page link in default state,
    \ ``"javascript:void(0)"``\  may be displayed on browser status bar.
    To change this behavior, it is necessary to disable the operation of page link click by using JavaScript.
    Refer to ":ref:`PaginationHowToUseDisablePageLinkUsingJavaScript`" for implementation example.

    From terasoluna-gfw-web 5.0.0.RELEASE, default value of \ ``disabledHref``\  attribute is changed from \ ``"#"``\  to \ ``"javascript:void(0)"``\ .
    By doing so, the focus does not move to top of the page on clicking page link in \ ``"disabled"``\  state.


 .. note:: **Path variables (placeholders)**

   Path variables that can be specified in  ``pathTmpl``  and ``queryTmpl`` are as follows:

        .. tabularcolumns:: |p{0.10\linewidth}|p{0.25\linewidth}|p{0.75\linewidth}|
        .. list-table::
            :header-rows: 1
            :widths: 10 25 75
    
            * - Sr. No.
              - Path variable name
              - Description
            * - 1.
              - page
              - Path variable for inserting page location.
            * - 2.
              - size
              - Path variable for inserting number of records to be fetched.
            * - 3.
              - sortOrderProperty
              - Path variable for inserting sort field of sort condition.
            * - 4.
              - sortOrderDirection
              - Path variable for inserting sort order of sort condition.

    Specify path variables in ``"{Path variable name}"``  format.

 .. warning:: **Constraints related to sort condition**

    Only one sort condition can be set as a sort condition path variable.
    Therefore, when the search result obtained by specifying multiple sort condition needs to be displayed using pagination,
    it is necessary to extend the JSP tag library of common library.

|

 When parameters to control the operations are changed, the following HTML is output.
 The numbers in figure correspond to serial numbers of parameter list mentioned above.

 - JSP

  .. code-block:: jsp

    <t:pagination page="${page}"
        disabledHref="#"
        pathTmpl="${pageContext.request.contextPath}/article/list/{page}/{size}"
        queryTmpl="sort={sortOrderProperty},{sortOrderDirection}"
        criteriaQuery="${f:query(articleSearchCriteriaForm)}"
        enableLinkOfCurrentPage="true" />

 - HTML to be output

  .. figure:: ./images/pagination-overview_html_changed2.png
   :alt: html of the pagination link(changed behavior).
   :width: 100%
   :align: center

|

Process flow when pagination is used
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Process flow when using pagination functionality of Spring Data and JSP tag library of common library is as follows:

 .. figure:: ./images/pagination-overview_flow.png
   :alt: processing flow of pagination
   :width: 100%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Apart from search conditions, specify the location of page to be searched (page) and number of records to be fetched (size) as request parameters and send the request.
    * - | (2)
      - | ``PageableHandlerMethodArgumentResolver``  fetches location of page to be searched (page) and number of records to be fetched (size) specified in request parameter and creates ``Pageable``  object.
        | The created ``Pageable``  object is set as an argument of Controller handler method.
    * - | (3)
      - | Controller passes the ``Pageable``  object received as an argument to Service method.
    * - | (4)
      - | Service passes the ``Pageable``  object received as an argument to Query method of Repository.
    * - | (5)
      - | Repository fetches total records (totalElements) of data matching the search conditions. It also fetches the data falling in the range of page location (page) and number of records to be fetched (size) specified in ``Pageable``  object received as an argument, from the database.
    * - | (6)
      - | Repository creates ``Page``  object based on total records fetched (totalElements), fetched data (content) and ``Pageable``  object received as an argument, and returns it to Service and Controller.
    * - | (7)
      - | Controller stores the returned ``Page``  object in ``Model``  object and displays JSP.
    * - | (8)
      - | JSP fetches the ``Page`` object stored in ``Model``  object and calls JSP tag library (``<t:pagination>``) for pagination provided by common library.
        | JSP tag library for pagination refers to ``Page``  object and creates pagination link.
    * - | (9)
      - | The HTML created in JSP is returned to client (browser).
    * - | (10)
      - | On clicking pagination link, the request to display the corresponding page is sent.

 .. note:: **Implementation of Repository**

   The implementation method of (5) & (6) are different depending on the O/R Mapper to be used.

   * When using MyBatis3, implementation of Java (Service) and SQL mapping file is necessary.
   * When using Spring Data JPA, implementation is not necessary because it is carried out automatically through the use of Spring Data JPA functionality.

   For implementation example refer to:

   * :doc:`DataAccessMyBatis3`
   * :doc:`DataAccessJpa`


|

How to use
--------------------------------------------------------------------------------

The method of using pagination functionality is as follows:

Application settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Settings for enabling pagination functionality of Spring Data
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Location of page to be searched (page), number of records to be fetched (size) and sort condition (sort) are specified in the request parameter. The functionality to set these properties in the argument of Controller as ``Pageable`` object should be enabled.
| The settings mentioned below are preset in a blank project.

:file:`spring-mvc.xml`

 .. code-block:: xml

    <mvc:annotation-driven>
        <mvc:argument-resolvers>
            <!-- (1) -->
            <bean
                class="org.springframework.data.web.PageableHandlerMethodArgumentResolver" />
        </mvc:argument-resolvers>
    </mvc:annotation-driven>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify ``org.springframework.data.web.PageableHandlerMethodArgumentResolver``  in ``<mvc:argument-resolvers>``.
        | For the properties that can be specified in ``PageableHandlerMethodArgumentResolver``, refer to ":ref:`paginatin_appendix_pageableHandlerMethodArgumentResolver`".

|

Page search
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The method of implementing page search is as follows:

Implementation of application layer
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
The information required for page search (such as location of page to be searched, number of records to be fetched and sort condition) is received as an argument and passed to Service method.

- Controller

 .. code-block:: java

    @RequestMapping("list")
    public String list(@Validated ArticleSearchCriteriaForm form,
            BindingResult result,
            Pageable pageable, // (1)
            Model model) {

        ArticleSearchCriteria criteria = beanMapper.map(form,
                ArticleSearchCriteria.class);

        Page<Article> page = articleService.searchArticle(criteria, pageable); // (2)

        model.addAttribute("page", page); // (3)

        return "article/list";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify ``Pageable``  as an argument of handler method.
        | ``Pageable``  object stores the information required for page search (such as location of page to be searched, number of records to be fetched and sort condition).
    * - | (2)
      - | Specify the ``Pageable``  object as an argument of Service method and then call the same.
    * - | (3)
      - | Add the search result (``Page`` object) returned by Service to ``Model``. It can be referred from View (JSP) after it is added to ``Model``.

 .. note:: **Operations when the information required for page search is not specified in request parameter**

    Default values are applied when the information required for page search (such as location of page to be searched, number of records to be fetched and sort condition) is not specified in request parameter.
    Default values are as follows:

    * Location of page to be searched: `0` (first page)
    * number of records to be fetched: `20`
    * Sort condition: `null` (no sort condition)

    Default values can be changed using the following two methods.

    * Define the default values by specifying ``@org.springframework.data.web.PageableDefault``  annotation as an argument of ``Pageable``  of handler method.
    * Specify ``Pageable``  object wherein default values are defined in ``fallbackPageable``  property of ``PageableHandlerMethodArgumentResolver``.

|

| See the method below for specifying default values using ``@PageableDefault``  annotation.
| Use ``@PageableDefault``  annotation to change the default values for each page search.

 .. code-block:: java

    @RequestMapping("list")
    public String list(@Validated ArticleSearchCriteriaForm form,
            BindingResult result,
            @PageableDefault( // (1)
                    page = 0,    // (2)
                    size = 50,   // (3)
                    direction = Direction.DESC,  // (4)
                    sort = {     // (5)
                        "publishedDate",
                        "articleId"
                        }
                    ) Pageable pageable,
            Model model) {
        // ...
        return "article/list";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.70\linewidth}|p{0.20\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 70 20

    * - Sr. No.
      - Description
      - Default values
    * - | (1)
      - | Specify ``@PageableDefault``  annotation as an argument of ``Pageable``.
      - | -
    * - | (2)
      - | To change the default value of location of page, specify the value in page attribute of ``@PageableDefault``  annotation.
        | Normally it need not be changed.
      - | ``0``
        | (first page)
    * - | (3)
      - | To change the default value of number of records to be fetched, specify the value in size or value attribute of ``@PageableDefault``  annotation.
      - | ``10``
    * - | (4)
      - | To change the default value of sort condition, specify the value in direction attribute of ``@PageableDefault``  annotation.
      - | ``Direction.ASC``
        | (Ascending order)
    * - | (5)
      - | Specify the sort fields of sort condition in sort attribute of ``@PageableDefault``  annotation.
        | When sorting the records using multiple sort fields, specify the property name to be sorted in array.
        | In the above example, sort condition ``"ORDER BY publishedDate DESC, articleId DESC"``  is added to Query.
      - | Empty array
        | (No sort field)

 .. note:: **About sort order that can be specified using @PageableDefault annotation**

    Sort order that can be specified using ``@PageableDefault``  annotation is either ascending or descending; hence when you want to specify different sort order for each field, it is necessary to use ``@org.springframework.data.web.SortDefaults``  annotation.
    For example, when sorting the fields using ``"ORDER BY publishedDate DESC, articleId ASC"``  sort order.

 .. tip:: **Specifying annotation when only the default value of number of records to be fetched is to be changed**

    In order to change only the default value of number of records to be fetched, the annotation can also be specified as ``@PageableDefault(50)``. This operation is same as ``@PageableDefault(size = 50)``.

|

| See the method below to specify default values using ``@SortDefaults``  annotation.
| ``@SortDefaults``  annotation is used when sorting needs to be done on multiple fields and in order to have different sort order for each field.

 .. code-block:: java

    @RequestMapping("list")
    public String list(
            @Validated ArticleSearchCriteriaForm form,
            BindingResult result,
            @PageableDefault(size = 50)
            @SortDefaults(  // (1)
                    {
                        @SortDefault(  // (2)
                                     sort = "publishedDate",    // (3)
                                     direction = Direction.DESC // (4)
                                    ),
                        @SortDefault(
                                     sort = "articleId"
                                    )
                    }) Pageable pageable,
            Model model) {
        // ...
        return "article/list";
    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.70\linewidth}|p{0.20\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 70 20

    * - Sr. No.
      - Description
      - Default values
    * - | (1)
      - | Specify ``@SortDefaults``  annotation as an argument of ``Pageable``.
        | Multiple ``@org.springframework.data.web.SortDefault``  annotations can be specified as arrays in ``@SortDefaults``  annotation.
      - | -
    * - | (2)
      - | Specify ``@SortDefault``  annotation as value attribute of ``@SortDefaults``  annotation.
        | Specify as array when specifying multiple annotations.
      - | -
    * - | (3)
      - | Specify sort fields in  sort or value attribute of ``@PageableDefault``.
        | Specify as array when specifying multiple fields.
      - | Empty array
        | (No sort field)
    * - | (4)
      - | Specify value in direction attribute of ``@PageableDefault``  to change default sort condition.
      - | ``Direction.ASC``
        | (Ascending)

 In the above example, sort condition called ``"ORDER BY publishedDate DESC, articleId ASC"``  is added to query.

 .. tip:: **Specifying annotation when only the default value of sort fields is to be specified**

    In order to specify only the records to be fetched, the annotation can also be specified as ``@PageableDefault("articleId")``.
    This operation is same as ``@PageableDefault(sort = "articleId")``  and ``@PageableDefault(sort = "articleId", direction = Direction.ASC)``.

|

When it is necessary to change the default values of entire application, specify ``Pageable``  object wherein default values are defined in ``fallbackPageable``  property
of ``PageableHandlerMethodArgumentResolver``  that is defined in :file:`spring-mvc.xml`.

For description of ``fallbackPageable``  and example of settings, refer to ":ref:`paginatin_appendix_pageableHandlerMethodArgumentResolver`".


|

Implementation of domain layer (MyBatis3)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| When accessing the database using MyBatis3, extract the necessary information from ``Pageable`` object received from Controller and pass it to the Repository.
| Sort conditions and SQL for extracting the corresponding data need to be implemented in SQL mapping.

For details on page search process to be implemented at domain layer, refer to:

* :ref:`DataAccessMyBatis3HowToUseFindPageUsingMyBatisFunction`
* :ref:`DataAccessMyBatis3HowToUseFindPageUsingSqlFilter`

|

Implementation of domain layer (JPA)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
When accessing the database using JPA (Spring Data JPA), pass the ``Pageable``  object received from Controller to Repository.

For details on page search process to be implemented at domain layer, refer to:

* :ref:`DataAccessJpaHowToUseFindPage`

|

Implementation of JSP (Base version)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The method to display pagination link and pagination information (total records, total pages, number of displayed pages etc.) by displaying the ``Page``  object fetched during page search on list screen, is described below.

Display of fetched data
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
An example to display the data fetched during page search is shown below.

- Controller

 .. code-block:: java

    @RequestMapping("list")
    public String list(@Validated ArticleSearchCriteriaForm form, BindingResult result,
            Pageable pageable, Model model) {

        if (!StringUtils.hasLength(form.getWord())) {
            return "article/list";
        }

        ArticleSearchCriteria criteria = beanMapper.map(form,
                ArticleSearchCriteria.class);

        Page<Article> page = articleService.searchArticle(criteria, pageable);

        model.addAttribute("page", page); // (1)

        return "article/list";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Store ``Page``  object with the attribute name ``"page"`` in ``Model``.
        | In JSP,  ``Page``  object can be accessed by specifying attribute name ``"page"``.


- JSP

 .. code-block:: jsp

    <%-- ... --%>

    <%-- (2) --%>
    <c:when test="${page != null && page.totalPages != 0}">

      <table class="maintable">
        <thead>
          <tr>
            <th class="no">No</th>
            <th class="articleClass">Class</th>
            <th class="title">Title</th>
            <th class="overview">Overview</th>
            <th class="date">Published Date</th>
          </tr>
        </thead>

        <%-- (3) --%>
        <c:forEach var="article" items="${page.content}" varStatus="rowStatus">
          <tr>
            <td class="no">
              ${(page.number * page.size) + rowStatus.count}
            </td>
            <td class="articleClass">
              ${f:h(article.articleClass.name)}
            </td>
            <td class="title">
              ${f:h(article.title)}
            </td>
            <td class="overview">
              ${f:h(article.overview)}
            </td>
            <td class="date">
              ${f:h(article.publishedDate)}
            </td>
          </tr>
        </c:forEach>

      </table>

      <div class="paginationPart">

        <%-- ... --%>

      </div>
    </c:when>

    <%-- ... --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (2)
      - | In the above example, it is checked whether data matching the specified conditions exists. If there is no such data, the header row is also not displayed.
        | When it is necessary to display the header row even if there is no matching data, this branching is no longer required.
    * - | (3)
      - | Display the list of data fetched using ``<c:forEach>``  tag of JSTL.
        | The fetched data is stored in a list in ``content``  property of ``Page``  object.

- Example of screen output in JSP

 .. figure:: ./images/pagination-how_to_use_view_list_screen.png
   :alt: Screen image of content table
   :width: 100%
   :align: center


|

.. _pagination_how_to_use_make_jsp_basic_paginationlink:

Display of Pagination link
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
See the example below to display the link for page navigation (pagination link).

Pagination link is output using JSP tag library of common library.

- :file:`include.jsp`

 Declare the JSP tag library of common library. The settings below are carried out in a blank project.

 .. code-block:: jsp

    <%@ taglib uri="http://terasoluna.org/tags" prefix="t"%>       <%-- (1) --%>
    <%@ taglib uri="http://terasoluna.org/functions" prefix="f"%>  <%-- (2) --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | JSP tag to display pagination link is stored.
    * - | (2)
      - | EL function of JSP used at the time of using pagination link, is stored.

- JSP

 .. code-block:: jsp

    <t:pagination page="${page}" /> <%-- (3) --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (3)
      - | Use ``<t:pagination>``  tag. In page attribute, specify ``Page``  object stored in ``Model``  of Controller.

|

- Output HTML

 The example below shows the search results obtained upon specifying ``"?page=0&size=6"``.

 .. code-block:: html

     <ul>
        <li class="disabled"><a href="javascript:void(0)">&lt;&lt;</a></li>
        <li class="disabled"><a href="javascript:void(0)">&lt;</a></li>
        <li class="active"><a href="javascript:void(0)">1</a></li>
        <li><a href="?page=1&size=6">2</a></li>
        <li><a href="?page=2&size=6">3</a></li>
        <li><a href="?page=3&size=6">4</a></li>
        <li><a href="?page=4&size=6">5</a></li>
        <li><a href="?page=5&size=6">6</a></li>
        <li><a href="?page=6&size=6">7</a></li>
        <li><a href="?page=7&size=6">8</a></li>
        <li><a href="?page=8&size=6">9</a></li>
        <li><a href="?page=9&size=6">10</a></li>
        <li><a href="?page=1&size=6">&gt;</a></li>
        <li><a href="?page=9&size=6">&gt;&gt;</a></li>
    </ul>

|

| If style sheet for pagination link is not created, the display will be as follows:
| As it is visible below, the pagination link is not established.

 .. figure:: ./images/pagination-how_to_use_jsp_not_applied_css.png
   :alt: Screen image that style sheet is not applied.
   :width: 120px
   :height: 290px

|

| The display will be as follows if minimal changes are carried out like adding a definition of style sheet for pagination link and changing the JSP.

- Screen image

 .. figure:: ./images/pagination-how_to_use_jsp_applied_simple_css.png
   :alt: Screen image that simple style sheet applied.
   :width: 290px
   :height: 40px

- JSP

 .. code-block:: jsp

    <%-- ... --%>

    <t:pagination page="${page}"
        outerElementClass="pagination" /> <%-- (4) --%>

    <%-- ... --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (4)
      - | Specify the class name indicating that it is a pagination link.
        | By specifying the class name, the applicable range of styles to be specified in style sheet can be restricted to pagination link.

- Style sheet

 .. code-block:: css

    .pagination li {
        display: inline;
    }

    .pagination li>a {
        margin-left: 10px;
    }

|

Even after the pagination link is established, the following two problems still persist.

* clickable and non-clickable links cannot be distinguished.
* The location of currently displayed page cannot be identified.

|

When Bootstrap v3.0.0 style sheet is applied to resolve the above problems, the display is as follows: 

- Screen image

 .. figure:: ./images/pagination-how_to_use_jsp_applied_bootstrap_v3_0_0_css.png
   :alt: Screen image that v3.0.0 of bootstrap is applied.
   :width: 520px
   :height: 70px

- Style sheet

 | Place the css file of bootstrap v3.0.0 under ``$WEB_APP_ROOT/resources/vendor/bootstrap-3.0.0/css/bootstrap.css``.
 | Abstract of pagination related style definition.


 .. code-block:: css

    .pagination {
      display: inline-block;
      padding-left: 0;
      margin: 20px 0;
      border-radius: 4px;
    }

    .pagination > li {
      display: inline;
    }

    .pagination > li > a,
    .pagination > li > span {
      position: relative;
      float: left;
      padding: 6px 12px;
      margin-left: -1px;
      line-height: 1.428571429;
      text-decoration: none;
      background-color: #ffffff;
      border: 1px solid #dddddd;
    }

    .pagination > li:first-child > a,
    .pagination > li:first-child > span {
      margin-left: 0;
      border-bottom-left-radius: 4px;
      border-top-left-radius: 4px;
    }

    .pagination > li:last-child > a,
    .pagination > li:last-child > span {
      border-top-right-radius: 4px;
      border-bottom-right-radius: 4px;
    }

    .pagination > li > a:hover,
    .pagination > li > span:hover,
    .pagination > li > a:focus,
    .pagination > li > span:focus {
      background-color: #eeeeee;
    }

    .pagination > .active > a,
    .pagination > .active > span,
    .pagination > .active > a:hover,
    .pagination > .active > span:hover,
    .pagination > .active > a:focus,
    .pagination > .active > span:focus {
      z-index: 2;
      color: #ffffff;
      cursor: default;
      background-color: #428bca;
      border-color: #428bca;
    }

    .pagination > .disabled > span,
    .pagination > .disabled > a,
    .pagination > .disabled > a:hover,
    .pagination > .disabled > a:focus {
      color: #999999;
      cursor: not-allowed;
      background-color: #ffffff;
      border-color: #dddddd;
    }


- JSP

 Add a definition to read the css file placed under JSP.

 .. code-block:: jsp

    <link rel="stylesheet"
        href="${pageContext.request.contextPath}/resources/vendor/bootstrap-3.0.0/css/bootstrap.css"
        type="text/css" media="screen, projection">

|

Display of pagination information
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
An example to display the information related to pagination (such as total records, total pages and total displayed pages) is as follows:

- Screen example

 .. figure:: ./images/pagination-how_to_use_view_pagination_info1.png
   :alt: Screen image of pagination information(total results, current pages, total pages)
   :width: 400px
   :height: 250px

- JSP

 .. code-block:: jsp

    <div>
        <fmt:formatNumber value="${page.totalElements}" /> results <%-- (1) --%>
    </div>
    <div>
        ${f:h(page.number + 1) } /       <%-- (2) --%>
        ${f:h(page.totalPages)} Pages    <%-- (3) --%>
    </div>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | To display total number of data records matching the search conditions, fetch value from ``totalElements``  property of ``Page``  object.
    * - | (2)
      - | To display number of displayed pages, fetch value from ``number`` property of ``Page`` object and increment the value by 1.
        | ``number`` property of ``Page`` object starts with ``0``; hence value should be incremented by 1 at the time of displaying the page number.
    * - | (3)
      - | To display total pages of data matching the search conditions, fetch the value from ``totalPages``  property of ``Page``  object.

|

Example to display the display data range of the corresponding page is shown below.

- Example of screen

 .. figure:: ./images/pagination-how_to_use_view_pagination_info2.png
   :alt: Screen image of pagination information(begin position, end position)
   :width: 400px
   :height: 250px

- JSP

 .. code-block:: jsp

    <div>
        <%-- (4) --%>
        <fmt:formatNumber value="${(page.number * page.size) + 1}" /> -
        <%-- (5) --%>
        <fmt:formatNumber value="${(page.number * page.size) + page.numberOfElements}" />
    </div>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (4)
      - | To display start location, calculate the value using ``number``  property and ``size``  property of ``Page`` object.
        | ``number``  property of ``Page``  object starts with ``0``; hence the value needs to be incremented by 1 at the time of displaying data start location.
    * - | (5)
      - | To display end location, calculate the value using ``number`` property, ``size``  property and ``numberOfElements`` property of ``Page`` object.
        | ``numberOfElements`` needs to be calculated since the last page is likely to be a fraction.

 .. tip:: **About format of numeric value**

    When the numeric value to be displayed needs to be formatted, use tag library ( ``<fmt:formatNumber>`` ) provided by JSTL.


|

.. _pagination_how_to_use_make_jsp_basic_search_criteria:

Carrying forward search conditions using page link
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
The method of carrying forward the search conditions to the page navigation request is shown below.

 .. figure:: ./images/pagination-how_to_use_view_take_over_search_criteria.png
   :alt: Processing image of take over search criteria
   :width: 100%
   :align: center

- JSP

 .. code-block:: jsp

    <%-- (1) --%>
    <div id="criteriaPart">
      <form:form action="${pageContext.request.contextPath}/article/list" method="get"
                 modelAttribute="articleSearchCriteriaForm">
        <form:input path="word" />
        <form:button>Search</form:button>
        <br>
      </form:form>
    </div>

    <%-- ... --%>

    <t:pagination page="${page}"
        outerElementClass="pagination"
        criteriaQuery="${f:query(articleSearchCriteriaForm)}" /> <%-- (2) --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Form to specify search conditions.
        | ``word``  is specified as a search condition.
    * - | (2)
      - | When carrying forward the search conditions to page navigation request, specify the \ **encoded URL query string**\  in \ ``criteriaQuery``\  attribute.
        | When storing the search conditions in form object, conditions can be carried forward easily if EL function ( ``f:query(Object)`` ) provided by common library is used.
        | In the above example, query string of \ ``"?page=page location&size=number of records to be fetched&word=input value"``\  format is generated.
        |
        | \ ``criteriaQuery``\ attribute can be used in terasoluna-gfw-web 1.0.1.RELEASE or higher version.

 .. note:: **Specifications of f:query(Object)**

    JavaBean of form object and ``Map`` object can be specified as an argument of ``f:query``.
    In case of JavaBean, property name is treated as request parameter name and in case of ``Map`` object, map key name is treated as request parameter.
    URL of the generated query string is encoded in UTF-8.

    From the terasoluna-gfw-web 5.0.1.RELEASE,  ``f:query`` has been supporting a nested structured JavaBean or \ ``Map``\.

    Refer to :ref:`TagLibAndELFunctionsHowToUseELFunctionQuery` for detail specification of the \ ``f:query``\  (URL encoding specification etc).

 .. warning:: **Operations when Query string created using f:query is specified in queryTmpl attribute**

    It has been found that specifying the query string generated using \ ``f:query``\ , in queryTmpl attribute leads to duplication of URL encoding. Thus, special characters are not carried forward correctly.
    
    This URL encoding duplication can be avoided by using \ ``criteriaQuery``\  attribute which can be used in terasoluna-gfw-web 1.0.1.RELEASE or higher version. 
    
|

Carrying forward the sort condition using page link
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
The method to carry forward the sort condition to page navigation request is as follows:

- JSP

 .. code-block:: jsp

    <t:pagination page="${page}"
        outerElementClass="pagination"
        queryTmpl="page={page}&size={size}&sort={sortOrderProperty},{sortOrderDirection}" />  <%-- (1) --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | To carry forward the sort condition to page navigation request, specify ``queryTmpl`` and add sort condition to query string.
        | For parameter specifications to specify sort condition, refer to ":ref:`Request parameters for page search <pagination_overview_pagesearch_requestparameter>` " 
        | In the above example, ``"?page=0&size=20&sort=sort item, sort order(ASC or DESC)"`` is a query string.

|

.. _pagination_how_to_use_make_jsp_layout:

Implementation of JSP (layout change)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Removal of link to navigate to the first page and the last page
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Example to remove "Link to navigate to the first page" and "Link to navigate to the last page" is shown below.

- Screen example

 .. figure:: ./images/pagination-how_to_use_view_remove_link1.png
   :alt: Remove page link that move to first & last page
   :width: 510px
   :height: 140px

- JSP

 .. code-block:: jsp

    <t:pagination page="${page}"
        outerElementClass="pagination"
        firstLinkText=""
        lastLinkText="" /> <%-- (1) (2) --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify ``""`` as firstLinkText attribute of ``<t:pagination>`` tag to hide "Link to navigate to the first page".
    * - | (2)
      - | Specify ``""`` as lastLinkText attribute of ``<t:pagination>`` tag  to hide "Link to navigate to the last page".

|


Removal of link to navigate to previous page and next page
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Example to remove "Link to navigate to the first page" and "Link to navigate to the last page" is shown below.

- Screen example

 .. figure:: ./images/pagination-how_to_use_view_remove_link2.png
   :alt: Remove page link that move to previous & next page
   :width: 470
   :height: 220px

- JSP

 .. code-block:: jsp

    <t:pagination page="${page}"
        outerElementClass="pagination"
        previousLinkText=""
        nextLinkText="" /> <%-- (1) (2) --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify ``""`` as previousLinkText attribute of ``<t:pagination>`` tag to hide "Link to navigate to the previous page".
    * - | (2)
      - | Specify ``""`` as nextLinkText attribute of ``<t:pagination>`` tag to hide "Link to navigate to the next page".

|

Removal of disabled link
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Example to remove the link in ``"disabled"`` state is shown below.
| Add the following definition to style sheet when the status is ``"disabled"``.

- Screen example

 .. figure:: ./images/pagination-how_to_use_view_remove_link3.png
   :alt: Remove page link that move to previous & next page
   :width: 530
   :height: 200px

- Style sheet

 .. code-block:: css

    .pagination .disabled {
        display: none;  /* (1) */
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify ``"display: none;"`` as an attribute value of ``"disabled"`` class.

|

Change in maximum number of display links to navigate to the specified page
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Example to change maximum number of display links to navigate to the specified page is shown below.

- Screen example

 .. figure:: ./images/pagination-how_to_use_view_change_maxsize.png
   :alt: change max display count of page link that move to specified page
   :width: 450
   :height: 220px

- JSP

 .. code-block:: jsp

    <t:pagination page="${page}"
        outerElementClass="pagination"
        maxDisplayCount="5" /> <%-- (1) --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | In order to change maximum number of display links to navigate to the specified page, specify value in maxDisplayCount attribute of ``<t:pagination>`` tag.

|

Removal of link to navigate to the specified page
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Example to remove link to navigate to the specified page is shown below.

- Screen example

 .. figure:: ./images/pagination-how_to_use_view_remove_link4.png
   :alt: Remove page link that move to specified page
   :width: 410
   :height: 220px

- JSP

 .. code-block:: jsp

    <t:pagination page="${page}"
        outerElementClass="pagination"
        maxDisplayCount="0" /> <%-- (1) --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | In order to hide the link to navigate to the specified page, specify ``"0"`` as maxDisplayCount attribute of ``<t:pagination>`` tag.


|

Implementation of JSP (Operation)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Specifying sort condition
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Example to specify sort condition from client is shown below.

- Screen example

 .. figure:: ./images/pagination-how_to_use_view_sort.png
   :alt: specify the sort condition
   :width: 100%

- JSP

 .. code-block:: jsp

    <div id="criteriaPart">
      <form:form
        action="${pageContext.request.contextPath}/article/search"
        method="get" modelAttribute="articleSearchCriteriaForm">
        <form:input path="word" />
        <%-- (1) --%>
        <form:select path="sort">
            <form:option value="publishedDate,DESC">Newest</form:option>
            <form:option value="publishedDate,ASC">Oldest</form:option>
        </form:select>
        <form:button>Search</form:button>
        <br>
      </form:form>
    </div>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | For specifying the sort condition from client, add the corresponding parameters for specifying the sort condition.
        | For parameter specifications to specify sort condition, refer to ":ref:`Request parameters for page search <pagination_overview_pagesearch_requestparameter>` " .
        | In the above example, publishedDate can be selected in ascending order or descending order from pull-down.

|

.. _PaginationHowToUseDisablePageLinkUsingJavaScript:

To disable page link using JavaScript
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
By default \ ``"javascript:void(0)"``\  is set in \ ``disabledHref``\  attribute of \ ``<t:pagination>``\  tag to disable the operation on clicking page link in \ ``"disabled"``\  state and \ ``"active"``\  state.
In such a state, if focus is moved or on mouseover to page link, \ ``"javascript:void(0)"``\  is displayed on browser status bar.
To change this behavior, it is necessary to disable the operation of page link click by using JavaScript.

Implementation example is shown below.

**JSP**

.. code-block:: jsp

    <%-- (1) --%>
    <script type="text/javascript"
            src="${pageContext.request.contextPath}/resources/vendor/js/jquery.js"></script>

    <%-- (2) --%>
    <script type="text/javascript">
        $(function(){
            $(document).on("click", ".disabled a, .active a", function(){
                return false;
            });
        });
    </script>

    <%-- ... --%>

    <%-- (3) --%>
    <t:pagination page="${page}" disabledHref="#" />

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Read js file of jQuery.

        In the above example, jQuery API is used to disable the operation of page link click using JavaScript.
    * - | (2)
      - Disable click event of page link of \ ``"disabled"``\  and \ ``"active"``\  states by using API of jQuery.

        However, when \ ``enableLinkOfCurrentPage``\  attribute of \ ``<t:pagination>``\  tag is set to \ ``"true"``\ , the click event of page link in \ ``"active"``\  state should not be disabled.
    * - | (3)
      - Set \ ``"#"``\  in \ ``disabledHref``\  attribute.

|

.. _paginatin_appendix:

Appendix
--------------------------------------------------------------------------------

.. _paginatin_appendix_pageableHandlerMethodArgumentResolver:

About property values of ``PageableHandlerMethodArgumentResolver``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Properties that can be specified in ``PageableHandlerMethodArgumentResolver`` are as follows:
| Values should be changed as required in the application.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.55\linewidth}|p{0.15\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 55 15

    * - Sr. No.
      - Property name
      - Description
      - Default value
    * - 1.
      - maxPageSize
      - | Specify maximum permissible value for the number of records to be fetched.
        | When the specified number of records to be fetched exceeds ``maxPageSize``, only the number of records specified as ``maxPageSize`` will be fetched.
      - |  `2000`
    * - 2.
      - fallbackPageable
      - | Specify default values for page location, number of records to be fetched and sort condition of the entire application.
        | When page location, number of records to be fetched and sort condition are not specified, the values set in fallbackPageable are used.
      - | Page location : `0`
        | Number of records to be fetched : `20`
        | Sort condition : `null`
    * - 3.
      - oneIndexedParameters
      - | Specify start value of page location.
        | When `false`  is specified, start value of page location becomes `0`  and when `true` is specified, it becomes `1`.
      - | `false`
    * - 4.
      - pageParameterName
      - | Specify request parameter name to specify page location.
      - | ``"page"``
    * - 5.
      - sizeParameterName
      - | Specify request parameter name to specify number of records to be fetched.
      - | ``"size"``
    * - 6.
      - prefix
      - | Specify prefix (namespace) of request parameter to specify page location and number of records to be fetched.
        | When there is a conflict between default parameter name and the parameter to be used in the application, it is recommended to specify namespace to avoid this issue.
        | If prefix is specified, request parameter name to specify page location will be ``prefix + pageParameterName`` and request parameter name to specify number of records to be fetched will be ``prefix + sizeParameterName``.
      - | ``""``
        | (No namespace)
    * - 7.
      - qualifierDelimiter
      - | To search multiple pages in the same request, specify request parameter name in ``qualifier + delimiter + standard parameter name`` format to distinguish the information required for page search (such as location of page to be searched, number of records to be fetched).
        | For this property, set ``delimiter`` value of the above format.
        | To change this setting, it is necessary to change the setting of ``qualifierDelimiter`` of ``SortHandlerMethodArgumentResolver``.
      - | ``"_"``

 .. note:: **Setting value of maxPageSize**

    Default value is ``2000``; however it is recommended to change the setting to maximum permissible value for the application.
    If maximum permissible value for the application is `100`, maxPageSize should also be set to `100`.

 .. note:: **Setting fallbackPageable**

    To change default values used in the entire application, set ``Pageable`` ( ``org.springframework.data.domain.PageRequest`` ) object wherein default value is defined in ``fallbackPageable`` property .
    To change the default sort condition, ``org.springframework.data.domain.Sort`` object where default value is defined in ``fallbackSort`` property  of ``SortHandlerMethodArgumentResolver``.

|

It is assumed that the fields given below will normally be changed in each application to be developed. The example to change the default values of such fields is given below.

* Maximum permissible value for number of records to be fetched ( ``maxPageSize`` )
* Default values ( ``fallbackPageable`` ) of page location and number of records to be fetched in the entire application
* Default sort condition ( ``fallbackSort`` )

 .. code-block:: xml

    <mvc:annotation-driven>
        <mvc:argument-resolvers>
            <bean
                class="org.springframework.data.web.PageableHandlerMethodArgumentResolver">
                <!-- (1) -->
                <property name="maxPageSize" value="100" />
                <!-- (2) -->
                <property name="fallbackPageable">
                    <bean class="org.springframework.data.domain.PageRequest">
                        <!-- (3) -->
                        <constructor-arg index="0" value="0" />
                        <!-- (4) -->
                        <constructor-arg index="1" value="50" />
                    </bean>
                </property>
                <!-- (5) -->
                <constructor-arg index="0">
                    <bean class="org.springframework.data.web.SortHandlerMethodArgumentResolver">
                        <!-- (6) -->
                        <property name="fallbackSort">
                            <bean class="org.springframework.data.domain.Sort">
                                <!-- (7) -->
                                <constructor-arg index="0">
                                    <list>
                                        <!-- (8) -->
                                        <bean class="org.springframework.data.domain.Sort.Order">
                                            <!-- (9) -->
                                            <constructor-arg index="0" value="DESC" />
                                            <!-- (10) -->
                                            <constructor-arg index="1" value="lastModifiedDate" />
                                        </bean>
                                        <!-- (8) -->
                                        <bean class="org.springframework.data.domain.Sort.Order">
                                            <constructor-arg index="0" value="ASC" />
                                            <constructor-arg index="1" value="id" />
                                        </bean>
                                    </list>
                                </constructor-arg>
                            </bean>
                        </property>
                    </bean>
                </constructor-arg>
            </bean>
        </mvc:argument-resolvers>
    </mvc:annotation-driven>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | In the above example, maximum value of number of records to be fetched is set to `100`. When value specified in number of records to be fetched (size) is `101` or more, search is performed for `100` records only.
    * - | (2)
      - | Create an instance of ``org.springframework.data.domain.PageRequest`` and set to ``fallbackPageable``.
    * - | (3)
      - | Specify default value of page location as the first argument of constructor of  ``PageRequest``.
        | In the above example, `0`  is specified, hence the default value is not changed.
    * - | (4)
      - | Specify default value of number of records to be fetched as the second argument of constructor of ``PageRequest``.
        | In the above example, the value will be considered `50` when number of records to be fetched is not specified in request parameter.
    * - | (5)
      - | Set an instance of ``SortHandlerMethodArgumentResolver`` as constructor of ``PageableHandlerMethodArgumentResolver`` .
    * - | (6)
      - | Create an instance of ``Sort`` and set to ``fallbackSort``.
    * - | (7)
      - | Set the list of ``Order`` objects to be used as default value as the first argument of ``Sort`` constructor.
    * - | (8)
      - | Create an instance of ``Order`` and add to the list of ``Order`` objects to be used as default value.
        | In the above example, sort condition of ``"ORDER BY x.lastModifiedDate DESC, x.id ASC"`` is added to query when sort condition is not specified in request parameter.
    * - | (9)
      - | Specify sort order (ASC/DESC) as the first argument of ``Order``  constructor.
    * - | (10)
      - | Specify sort item as the second argument of ``Order`` constructor.

|

.. _paginatin_appendix_sortHandlerMethodArgumentResolver:

Property value of ``SortHandlerMethodArgumentResolver``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Properties that can be specified in ``SortHandlerMethodArgumentResolver`` are as follows:
| Values should be changed as required in the application.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.55\linewidth}|p{0.15\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 55 15

    * - Sr. No.
      - Property name
      - Description
      - Default value
    * - 1.
      - fallbackSort
      - | Specify default sort condition for the entire application.
        | When sort condition is not specified, the value set in fallbackSort is used.
      - | `null`
        | (No sort condition)
    * - 2.
      - sortParameter
      - | Specify request parameter name to specify the sort condition.
        | When there is a conflict between default parameter name and the parameter to be used in the application, it is recommended to change the request parameter name to avoid this issue.
      - | ``"sort"``
    * - 3.
      - propertyDelimiter
      - | Specify delimiter of sort items and sort order (ASC,DESC).
      - | ``","``
    * - 4.
      - qualifierDelimiter
      - | To search multiple pages in the same request, specify request parameter name in `` qualifier + delimiter + sortParameter `` format to distinguish the information required for page search (such as sort condition).
        | For this property, set ``delimiter`` value of the above format.
      - | ``"_"``

.. raw:: latex

   \newpage

