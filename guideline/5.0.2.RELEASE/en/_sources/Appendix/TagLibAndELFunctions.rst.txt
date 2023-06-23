JSP Tag Libraries and EL Functions offered by common library
================================================================================

.. _TagLibAndELFunctionsOverview:

Overview
--------------------------------------------------------------------------------

Below are the JSP Tag Libraries and EL Functions offered by common library
as an ability to support the JSP implementation.

.. _TagLibAndELFunctionsOverviewTagLibs:

JSP Tag Library
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Below are the JSP Tag Libraries offered by common library.

 .. tabularcolumns:: |p{0.5\linewidth}|p{0.25\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 5 25 60

    * - | Sr.No
      - | Tag name
      - | Overview
    * - 1.
      - :ref:`TaglibAndELFunctionsHowToUseTaglibPagination`
      - Outputs the pagination link.
    * - 2.
      - :ref:`TaglibAndELFunctionsHowToUseTaglibMessagesPanel`
      - Outputs the result message.
    * - 3.
      - :ref:`TaglibAndELFunctionsHowToUseTaglibTransaction`
      - Outputs the transaction token as hidden item.

.. _TagLibAndELFunctionsOverviewELFunctions:

EL Functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Below are the EL Functions offered by common library.

**XSS counter measures**

 .. tabularcolumns:: |p{0.5\linewidth}|p{0.25\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 5 25 60

    * - | Sr.No
      - | Function Name
      - | Overview
    * - 1.
      - :ref:`TaglibAndELFunctionsHowToUseELFunctionH`
      - Converts the specified object into a string and escape the HTML special characters from converted string.
    * - 2.
      - :ref:`TaglibAndELFunctionsHowToUseELFunctionJs`
      - Escape the JavaScript special characters from the specified string.
    * - 3.
      - :ref:`TaglibAndELFunctionsHowToUseELFunctionHjs`
      - After escaping the JavaScript special characters from the specified string, escape the HTML special characters. (short function of \ ``f:h(f:js())``\)

**URL related**

 .. tabularcolumns:: |p{0.5\linewidth}|p{0.25\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 5 25 60

    * - | Sr.No
      - | Function Name
      - | Overview
    * - 4.
      - :ref:`TaglibAndELFunctionsHowToUseELFunctionQuery`
      - Generates UTF-8 URL encoded query string from the specified object.
    * - 5.
      - :ref:`TaglibAndELFunctionsHowToUseELFunctionU`
      - Performs UTF-8 URL encoding on specified string.

**DOM related**

 .. tabularcolumns:: |p{0.5\linewidth}|p{0.25\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 5 25 60

    * - | Sr.No
      - | Function Name
      - | Overview
    * - 6.
      - :ref:`TaglibAndELFunctionsHowToUseELFunctionLink`
      - Generates a hyperlink (\ ``<a>`` \ tag) for jumping to specified URL.
    * - 7.
      - :ref:`TaglibAndELFunctionsHowToUseELFunctionBr`
      - Converts the new line character into \ ``<br />`` \ tag from the specified string.

**Utility**

 .. tabularcolumns:: |p{0.5\linewidth}|p{0.25\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 5 25 60

    * - | Sr.No
      - | Function Name
      - | Overview
    * - 8.
      - :ref:`TaglibAndELFunctionsHowToUseELFunctionCut`
      - Extracts specified number of characters from the specified string.

|

.. _TagLibAndELFunctionsHowToUse:

How to use
--------------------------------------------------------------------------------

The use of JSP Tag Library and EL function offered by common library explained below. 
The appropriate Hyperlink is placed at appropriate location if detail description explained in other chapters.

|

.. _TaglibAndELFunctionsHowToUseTaglibPagination:

<t:pagination>
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The \ ``<t:pagination>`` \ tag is a 
JSP Tag Library to output the pagination link
by referring the information stored in page search results (\ ``org.springframework.data.domain.Page``\).

For detail description of pagination and how to use this tag, Refer the following section [:doc:`../ArchitectureInDetail/Pagination`].

* For pagination link, [:ref:`pagination_overview_paginationlink`]
* For parameter values of this tag, [:ref:`pagination_overview_paginationlink_taglibparameters`]
* For basic implementation of the JSP using this tag, [:ref:`pagination_how_to_use_make_jsp_basic_paginationlink`]
* For the layout of how to change the pagination link, [:ref:`pagination_how_to_use_make_jsp_layout`]

|

.. _TaglibAndELFunctionsHowToUseTaglibMessagespanel:

<t:messagesPanel>
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The \ ``<t:messagesPanel>`` \ tag is a JSP Tag Library to output the processing result message, 
(Such as \ ``org.terasoluna.gfw.common.message.ResultMessage`` \ or message having exception).

Refer the following section [:doc:`../ArchitectureInDetail/MessageManagement`] for how to use this tag.

* For how to display messages using this tag, [:ref:`message-display`]
* For parameter values of this tag, [:ref:`message-management-messagepanel-attribute`]

|

.. _TaglibAndELFunctionsHowToUseTaglibTransaction:

<t:transaction>
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The \ ``<t:transaction>`` \ tag is a JSP Tag Library to output the transaction token as hidden item (\ ``<input type="hidden">"``\ ).

Refer the following section [:doc:`../ArchitectureInDetail/DoubleSubmitProtection`] for the transaction token check feature and how to use this tag.

* For transaction token check feature, [:ref:`doubleSubmit_how_to_use_transaction_token_check`]
* For how to use this tag, [:ref:`doubleSubmit_how_to_use_transaction_token_check_jsp`]

.. note::

   This tag is used for sending a transaction token to the server while using standard HTML \ ``<form>`` \ tag.

   No need to use this tag if spring framework offered \ ``<form:form>`` \ tag (JSP Tag Library) has been used because
   \ ``org.terasoluna.gfw.web.token.transaction.TransactionTokenRequestDataValueProcessor`` \ 
   offered by the common library has been already mechanized to handle a transaction token.

|

.. _TaglibAndELFunctionsHowToUseELFunctionH:

f:h()
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The \ ``f:h()`` \ is an EL Function which converts the specified object into a string and escape the HTML special characters from converted string.

Refer [:ref:`xss_how_to_use_ouput_escaping`] for the specification of HTML special characters and escaping.

f:h() function specification
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**Argument**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - Sr.No
      - Type
      - Description
    * - 1.
      - \ ``java.lang.Object``\
      - Object that contain HTML special characters

 .. note::

    Specified objects,

    * In case of array, \ ``java.util.Arrays#toString`` \ method
    * In case of not array, \ ``toString`` \ method of specified object 

    is called for the string conversions.


**Return value**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - Sr.No
      - Type
      - Description
    * - 1.
      - \ ``java.lang.String``\
      - String after HTML escaping

        If the object specified in argument is \ ``null`` \, returns the empty string(\ ``""``\ ).

f:h() how to use
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

For the information about how to use \ ``f:h()`` \ function, refer [:ref:`xss_how_to_use_h_function_example`].

|

.. _TaglibAndELFunctionsHowToUseELFunctionJs:

f:js()
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The \ ``f:js()`` \ is an EL Function which escape the JavaScript special characters from the specified string argument.

Refer [:ref:`xss_how_to_use_javascript_escaping`] for the specification of JavaScript special characters and escaping.

f:js() function specification
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**Argument**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - Sr.No
      - Type
      - Description
    * - 1.
      - \ ``java.lang.String``\
      - String that contain JavaScript special characters

**Return value**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - Sr.No
      - Type
      - Description
    * - 1.
      - \ ``java.lang.String``\
      - String after JavaScript escaping

        If the string specified in argument is \ ``null`` \, it returns the empty string(\ ``""``\ ).


f:js() how to use
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

For the information about how to use \ ``f:js()`` \ function, refer [:ref:`xss_how_to_use_js_function_example`].

|

.. _TaglibAndELFunctionsHowToUseELFunctionHjs:

f:hjs()
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The \ ``f:hjs()`` \ is an EL Function which escapes the HTML special characters after 
escaping the JavaScript special characters from the specified string argument, (short function of \ ``f:h(f:js())``\).

* For how to use, refer [:ref:`xss_how_to_use_event_handler_escaping`].
* Refer [:ref:`xss_how_to_use_javascript_escaping`] for the specification of JavaScript special characters and escaping.
* Refer [:ref:`xss_how_to_use_ouput_escaping`] for the specification of HTML special characters and escaping.


f:hjs() function specification
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**Argument**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - Sr.No
      - Type
      - Description
    * - 1.
      - \ ``java.lang.String``\
      - String that contain HTML special characters or JavaScript special characters

**Return value**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - Sr.No
      - Type
      - Description
    * - 1.
      - \ ``java.lang.String``\
      - String after JavaScript and HTML escaping.

        If the string specified in argument is \ ``null`` \, it returns the empty string(\ ``""``\ ).

f:hjs() how to use
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

For the information about how to use \ ``f:hjs()`` \ function, refer [:ref:`xss_how_to_use_hjs_function_example`].

|

.. _TaglibAndELFunctionsHowToUseELFunctionQuery:

f:query()
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The \ ``f:query()`` \ is an EL Function which generates the query string 
from java.util.Map object or JavaBean (form object) that is specified in the argument.
Parameter names and parameter values in the query string are URL encoded in UTF-8.

URL encoding specification explained below.

In this function, the parameter name and parameter value of the query string are encoded on \ `RFC 3986 <http://www.ietf.org/rfc/rfc3986.txt>`_ \ basis.
In RFC 3986, the part of query string is defined as follows.

.. figure:: ./images_TagLibAndELFunctions/TagLibAndELFunctionsRFC3986UriSyntax.png
    :width: 90%

* query = \*( pchar / \ ``"/"``\  / \ ``"?"``\ )
* pchar = unreserved / pct-encoded / sub-delims / \ ``":"``\  / \ ``"@"``\
* unreserved = ALPHA / DIGIT / \ ``"-"``\  / \ ``"."``\  / \ ``"_"``\  / \ ``"~"``\
* sub-delims = \ ``"!"``\  / \ ``"$"``\  / \ ``"&"``\  / \ ``"'"``\  / \ ``"("``\  / \ ``")"``\  / \ ``"*"``\  / \ ``"+"``\  / \ ``","``\  / \ ``";"``\  / \ ``"="``\
* pct-encoded = \ ``"%"``\  HEXDIG HEXDIG

In this function, one of the character that can be used as a query string,

* \ ``"="``\  (Separator character of the parameter name and parameter value)
* \ ``"&"``\  (Separator character when dealing with multiple parameters)
* \ ``"+"``\  (Character that represent a space when you submit HTML Form)

are encoded in the pct-encoded formatting string.

f:query() function specification
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**Argument**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - Sr.No
      - Type
      - Description
    * - 1.
      - \ ``java.lang.Object``\
      - Object from which the query string generated (JavaBean or \ ``Map``\ )

        Property name will be a request parameter name if you have specified a JavaBean and key name will be a request parameter name if you specified the \ ``Map``\.

        Supported value types of JavaBean's property and value of \ ``Map``\  are as follows:

        * Classes which implements \ ``Iterable``\  interface
        * Array
        * Classes which implements  \ ``Map``\  interface
        * JavaBean
        * Simple Types (classes that can converted to \ ``String``\  type the using \ ``DefaultFormattingConversionService``\)

        From the terasoluna-gfw-web 5.0.1.RELEASE, it has been improved to be able specify a nested structured JavaBean or \ ``Map``\ .



 .. note::

    A simple type property value of the specified object is converted into string using the convert method of \ ``org.springframework.format.support.DefaultFormattingConversionService``\.
    Refer \ `Spring Framework Reference Documentation(Spring Type Conversion) <http://docs.spring.io/spring/docs/4.1.9.RELEASE/spring-framework-reference/html/validation.html#core-convert>`_\
    for the \ ``ConversionService``\.


**Return value**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - Sr.No
      - Type
      - Description
    * - 1.
      - \ ``java.lang.String``\
      - Query string that is generated based on the specified object in the argument (URL encoded string in UTF-8)

        If the object specified in argument is other than the JavaBean or \ ``Map``\, it returns the empty string(\ ``""``\ ).

 .. note:: **Rules for conversion to the query string**

    \ ``f:query()``\  converts an object so that the Spring Web MVC can handle it at binding process provided.

    **[Request parameter name]**

     .. tabularcolumns:: |p{0.45\linewidth}|p{0.30\linewidth}|p{0.25\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 45 30 25

        * - Conditions
          - Conversion specification of parameter name
          - Conversion examples
        * - Case that property type is an instance of \ ``Iterable``\
          - Property name + \ ``[element position]``\
          - \ ``status[0]=accepting``\
        * - Case that property type is an instance of \ ``Iterable``\  or Array and the value of the element is empty
          - | Property name
            | (Does not append \ ``[element position]``\)
          -  \ ``status=``\
        * - Case that property type is an instance of \ ``Map``\
          - Property name + \ ``[Map's key name]``\
          - \ ``status[accepting]=Accepting Order``\
        * - Case that property type(including element type in \ ``Iterable``\ ,Array and \ ``Map``\) is JavaBean
          - Value that combined a property name with \ ``"."``\ (dot)
          - | \ ``mainContract.name=xxx``\
            | \ ``subContracts[0].name=xxx``\
        * - Case that property type is a simple type
          - Property name
          - \ ``userId=xxx``\
        * - Case that property value is \ ``null``\
          - \ ``_``\ (underscore) + Property name
          - | \ ``_mainContract.name=``\
            | \ ``_status[0]=``\
            | \ ``_status[accepting]=``\

    **[Request parameter value]**

     .. tabularcolumns:: |p{0.45\linewidth}|p{0.30\linewidth}|p{0.25\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 45 30 25

        * - Conditions
          - Conversion specification of parameter value
          - Conversion examples
        * - Case that property value is \ ``null``\
          - Empty string
          - \ ``_userId=``\
        * - Case that property type is an instance of \ ``Iterable``\  or Array and the value of the element is empty
          - Empty string
          - \ ``status=``\
        * - Case that property value is not \ ``null``\
          - Value that can be converted to \ ``String``\  type using the \ ``DefaultFormattingConversionService``\
          - \ ``targetDate=20150801``\

f:query() how to use
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

For the information about how to use \ ``f:query()`` \ function, refer [:ref:`pagination_how_to_use_make_jsp_basic_search_criteria`].
Here, this function is used as to carry forward the search criteria while switching the pages using the pagination link.
Further, function description and the specification also described here and that should be read.

|

.. _TaglibAndELFunctionsHowToUseELFunctionU:

f:u()
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The \ ``f:u()`` \ is an EL Function which performs UTF-8 URL encoding on specified string argument.


This function is provided for performing URL encoding on those values which are going to be set as parameter values in the query string.
For the URL encoding specification, refer [:ref:`TaglibAndELFunctionsHowToUseELFunctionQuery`]

f:u() function specification
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**Argument**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - Sr.No
      - Type
      - Description
    * - 1.
      - \ ``java.lang.String``\
      - String that contain URL encoding required characters

**Return value**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - Sr.No
      - Type
      - Description
    * - 1.
      - \ ``java.lang.String``\
      - String after URL encoding

        If the string specified in argument is \ ``null`` \, it returns the empty string(\ ``""``\ ).

f:u() how to use
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. code-block:: jsp

    <div id="url">
        <a href="https://search.yahoo.com/search?p=${f:u(bean.searchString)}">  <!-- (1) -->
            Go to Yahoo Search
        </a>
    </div>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr.No
      - Description
    * - | (1)
      - In the above example, sets the URL-encoded value to the request parameters of the search site using this function.

|

.. _TaglibAndELFunctionsHowToUseELFunctionLink:

f:link()
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The \ ``f:link()`` \ is an EL Function which generates a hyperlink (\ ``<a>`` \ tag) for jumping to specified URL which is specified in the argument.

.. warning::

    Please note that, this function is not going to escaping the special characters nor performing the URL encoding.

f:link() function specification
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**Argument**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - Sr.No
      - Type
      - Description
    * - 1.
      - \ ``java.lang.String``\
      - Link of the URL string

        URL string should be HTTP or HTTPS schema of the URL format.
        (e.g : \ ``http://hostname:80/terasoluna/global.ex?id=123``\ )

**Return value**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - Sr.No
      - Type
      - Description
    * - 1.
      - \ ``java.lang.String``\
      - Generated Hyper link (\ ``<a>`` \ tag)  based on string specified in the argument

        The string specified in arguments,

        * If the string specified in argument is \ ``null`` \, the empty string(\ ``""``\ )
        * If the string is not in the URL format of HTTP or HTTPS schema, input string without generating a hyperlink

        is return.

f:link() how to use
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**Implementation**

.. code-block:: jsp

    <div id="link">
        ${f:link(bean.httpUrl)}  <!-- (1) -->
    </div>

**Output**

.. code-block:: html

    <div id="link">
        <a href="http://terasoluna.org/">http://terasoluna.org/</a>  <!-- (2) -->
    </div>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr.No
      - Description
    * - | (1)
      - | Generated Hyper link from the URL string specified in the argument.
    * - | (2)
      - | URL string specified in the argument set in the \ ``href`` \ attribute of \ ``<a>`` \ tag and link name of the hyper link.

.. warning::

    When adding the request parameters to the URL, the value of the request parameters should be URL encoded.
    When adding the request parameters, the value of the request parameters should be URL encoded 
    using appropriate \ ``f:query()`` \ function or \ ``f:u()`` \ function.

    In addition, it has been described in the return value description, if the format of the URL string specified in the argument is not appropriate,
    it returns the input string value without generating a hyperlink.
    Therefore, if you want to use the input value from the user as a URL string in the argument, 
    similar to string output process, the escaping process of the HTML special characters (:doc:`../Security/XSS`) are required.

|

.. _TaglibAndELFunctionsHowToUseELFunctionBr:

f:br()
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The \ ``f:br()`` \ is an EL Function which converts the new line character (\ ``CRLF``\ , \ ``LF``\ , \ ``CR``\ ) specified in the argument into \ ``<br />`` \ tag.

.. tip::

    If you want to display a string containing new line code as a newline on browser, it is necessary to convert the new line code into \ `` <br /> `` \ tag.

    For example, if you want to display the string entered in the textarea (\ ``<textarea>``\ ) of the input screen as it is
    on the confirmation screen or completion screen, it is advisable to use this function.

f:br() function specification
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**Argument**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - Sr.No
      - Type
      - Description
    * - 1.
      - \ ``java.lang.String``\
      - String that contain new line code

**Return value**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - Sr.No
      - Type
      - Description
    * - 1.
      - \ ``java.lang.String``\
      - String after conversion

        If the string specified in argument is \ ``null`` \, it returns the empty string(\ ``""``\ ).

f:br() how to use
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. code-block:: jsp

    <div id="text">
        ${f:br(f:h(bean.text))}">  <!-- (1) -->
    </div>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr.No
      - Description
    * - | (1)
      - The newline displays on the browser by converting the new line character into \ ``<br />`` \ tag from the specified string argument.

.. note::

    When you display a string on the screen, there is a need to escape the HTML special character as [:doc:`../Security/XSS`].
    
    if you are converting new line code into \ ``<br />`` \ tag using \ ``f:br()`` \ function,
    as in the above example, a string that has escaped the HTML special characters need to pass as an argument to \ ``f:br()`` \ function.
    
    The string obtained by converting new line code into \ ``<br />`` \ tag using \ ``f:br()`` \ function 
    passes as an argument to the \ ``f:h()`` \ function, the letter \ ``"<br />"`` \ get displayed on the browser hence be careful in order to call the function.

|

.. _TaglibAndELFunctionsHowToUseELFunctionCut:

f:cut()
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
 The  \ ``f:cut()`` \ is an EL Function which extracts specified number of characters from the specified string.


f:cut() function specification
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**Argument**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - Sr.No
      - Type
      - Description
    * - 1.
      - \ ``java.lang.String``\
      - String from which extraction is done
    * - 2.
      - \ ``int``\
      - The number of characters that can extracted

**Return value**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 60

    * - Sr.No
      - Type
      - Description
    * - 1.
      - \ ``java.lang.String``\
      - The extracted string (String part that exceeds the specified number of characters has been destroyed)

        If the string specified in argument is \ ``null`` \, it returns the empty string(\ ``""``\ ).

f:cut() how to use
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. code-block:: jsp

    <div id="cut">
        ${f:h(f:cut(bean.originText, 5))}  <!-- (1) -->
    </div>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr.No
      - Description
    * - | (1)
      - | In the above example, you can extract the first five characters of the string that was specified in the argument and displays on the screen.

.. note::

    There is a need to escape the HTML special character as [:doc:`../Security/XSS`] while displaying the extracted string on the screen.
    In the above example, string is escaped by using \ ``f:h()`` \ function.

.. raw:: latex

  \newpage
