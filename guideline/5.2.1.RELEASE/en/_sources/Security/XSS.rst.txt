XSS Countermeasures
================================================================================

.. only:: html

 .. contents:: Table of Contents
    :local:

.. _SpringSecurityXSS:

Overview
--------------------------------------------------------------------------------

It explains about Cross-site scripting (hereinafter abbreviated as XSS).
Cross Site Scripting is injection of malicious scripts across trusted web sites by deliberately using security defects in the web application.
For example, when data entered in Web Application (form input etc.) is output in HTML without appropriate escaping, the characters of tag existing in input value are interpreted as HTML as is.
If a script with malicious value is run, attacks such as session hijack occur due to cookie tampering and fetching of cookie values.

Stored & Reflected XSS Attacks
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

XSS attacks are broadly classified into two categories.

**Stored XSS Attacks**

In Stored XSS Attacks, the malicious code is permanently stored on target servers (such as database).
Upon requesting the stored information, the user retrieves the malicious script from the server and ends up running the same.

**Reflected XSS Attacks**

In Reflected attacks, the malicious code sent as a part of the request to the server is reflected back along with error messages, search results, or other different types of responses.
When a user clicks the malicious link or submits a specially crafted form, the injected code returns a result reflecting an occurrence of attack on user's browser.
The browser ends up executing the malicious code because the value came from a trusted server.

Both Stored XSS Attacks and Reflected XSS Attacks can be prevented by escaping output value.

How to use
--------------------------------------------------------------------------------

When the input from user is output as is, the system gets exposed to XSS vulnerability.
Therefore, as a countermeasure against XSS vulnerability, it is necessary to escape the characters which have specific meaning in the HTML markup language.

Escaping should be divided into 3 types if needed.

Escaping types:

 * Output Escaping
 * JavaScript Escaping
 * Event handler Escaping

.. _xss_how_to_use_ouput_escaping:

Output Escaping
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Escaping HTML special characters is a fundamental countermeasure against XSS vulnerability.
Example of HTML special characters that require escaping and example after escaping these characters are as follows:

.. tabularcolumns:: |p{0.50\linewidth}|p{0.50\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 50 50

   * - | Before escaping
     - | After escaping
   * - | ``&``
     - | ``&amp;``
   * - | ``<``
     - | ``&lt;``
   * - | ``>``
     - | ``&gt;``
   * - | ``"``
     - | ``&quot;``
   * - | ``'``
     - | ``&#39;``

To prevent XSS, \ ``f:h()``\  should be used in all display items that are to be output as strings.
An example of application where input value is to be re-output on different screen is given below.

Example of vulnerability when output values are not escaped
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

This example below is given only for reference; it should never be implemented.

**Implementation of output screen**

.. code-block:: jsp

    <!-- omitted -->
    <tr>
        <td>Job</td>
        <td>${customerForm.job}</td>  <!-- (1) -->
    </tr>
    <!-- omitted -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Job, which is a customerForm field, is output without escaping.

Enter <script> tag in Job field on input screen.

.. figure:: ./images_XSS/xss_screen_input_html_tag.png
   :alt: input_html_tag
   :width: 80%
   :align: center

   **Picture - Input HTML Tag**

| It is recognized as <script> tag and dialog box is displayed.

.. figure:: ./images_XSS/xss_screen_no_escape_result.png
   :alt: no_escape_result
   :width: 60%
   :align: center

   **Picture - No Escape Result**

.. _xss_how_to_use_h_function_example:

Example of escaping output value using f:h() function
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""


**Implementation of output screen**

.. code-block:: jsp

    <!-- omitted -->
    <tr>
        <td>Job</td>
        <td>${f:h(customerForm.job)}</td>  <!-- (1) -->
    </tr>
    .<!-- omitted -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | EL function \ ``f:h()``\  is used for escaping.

| Enter <script> tag in Job field on input screen.

.. figure:: ./images_XSS/xss_screen_input_html_tag.png
   :alt: input_html_tag
   :width: 80%
   :align: center

   **Picture - Input HTML Tag**

| By escaping special characters, input value is output as is without being recognized as <script> tag.

.. figure:: ./images_XSS/xss_screen_escape_result.png
   :alt: escape_result
   :width: 60%
   :align: center

   **Picture - Escape Result**

**Output result**

.. code-block:: jsp

    <!-- omitted -->
    <tr>
        <td>Job</td>
        <td>&lt;script&gt;alert(&quot;XSS Attack&quot;)&lt;/script&gt;</td>
    </tr>
    <!-- omitted -->

.. tip:: **java.util.Date subclass format**

    It is recommended that you use \ ``<fmt:formatDate>``\  of JSTL to format and display java.util.Date subclasses.
    See the example below.

        .. code-block:: jsp

            <fmt:formatDate value="${form.date}" pattern="yyyyMMdd" />

    If \ ``f:h()``\  is used for setting the value of "value" attribute, it gets converted into String and \ ``javax.el.ELException``\  is thrown; hence \ ``${form.date}``\  is used as is.
    However, it is safe from XSS attack since the value is in yyyyMMdd format.

.. tip::

        **String that can be parsed into java.lang.Number or subclass of java.lang.Number**

        It is recommended that you use \ ``<fmt:formatNumber>``\  to format and display the string that can be parsed to java.lang.Number subclasses or java.lang.Number.
        See the example below.

            .. code-block:: jsp

                <fmt:formatNumber value="${f:h(form.price)}" pattern="###,###" />

        There is no problem even if the above is a String; hence when \ ``<fmt:formatNumber>``\  tag is no longer used, \ ``f:h()``\ is being used explicitly so that no one forgets to use ``f:h()``.

.. _xss_how_to_use_javascript_escaping:

JavaScript Escaping
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Escaping JavaScript special characters is a fundamental countermeasure against XSS vulnerability.
Escaping is must if it is required to dynamically generate JavaScript based on the outside input.

Example of JavaScript special characters that require escaping and example after escaping these characters are as follows:

.. tabularcolumns:: |p{0.50\linewidth}|p{0.50\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 50 50

   * - | Before escaping
     - | After escaping
   * - | ``'``
     - | ``\'``
   * - | ``"``
     - | ``\"``
   * - | ``\``
     - | ``\\``
   * - | ``/``
     - | ``\/``
   * - | ``<``
     - | ``\x3c``
   * - | ``>``
     - | ``\x3e``
   * - | ``0x0D(Return)``
     - | ``\r``
   * - | ``0x0A(Linefeed)``
     - | ``\n``

Example of vulnerability when output values are not escaped
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Example of occurrence of XSS problem is given below.

This example below is given only for reference; it should never be implemented.

.. code-block:: html

  <html>
    <script  type="text/javascript">
        var aaa = '<script>${warnCode}<\/script>';
        document.write(aaa);
    </script>
  <html>

.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 80

   * - Attribute name
     - Value
   * - | warnCode
     - | ``<script></script><script>alert('XSS Attack!');</script><\/script>``

As shown in the above example, in order to dynamically generate JavaScript elements such as generating the code based on the user input, string literal gets terminated unintentionally leading to XSS vulnerability.

.. figure:: ./images_XSS/javascript_xss_screen_no_escape_result.png
   :alt: javascript_xss_screen_no_escape_result
   :width: 30%
   :align: center

   **Picture - No Escape Result**

**Output result**

.. code-block:: html

    <script type="text/javascript">
        var aaa = '<script><\/script><script>alert('XSS Attack!');<\/script><\/script>';
        document.write(aaa);
    </script>

    .. tip::

    Dynamically generated JavaScript code depending on user input carries a risk of any script being inserted; hence an alternate way should be considered or it should be avoided as much as possible unless there is a specific business requirement.

.. _xss_how_to_use_js_function_example:

Example of escaping output value using f:js() function
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

To prevent XSS, it is recommended that you use EL function \ ``f:js()``\  for the value entered by user.

Usage example is shown below.

.. code-block:: html

    <script type="text/javascript">
      var message = '<script>${f:js(message)}<\/script>';  // (1)
      <!-- omitted -->
    </script>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | By using \ ``f:js()``\  of EL function, the value is set as variable after escaping the value entered by user.

**Output result**

.. code-block:: html

    <script  type="text/javascript">
        var aaa = '<script>\x3c\/script\x3e\x3cscript\x3ealert(\'XSS Attack!\');\x3c\/script\x3e<\/script>';
        document.write(aaa);
    </script>

.. _xss_how_to_use_event_handler_escaping:

Event handler Escaping
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To escape the value of event handler of javascript, \ ``f:hjs()``\  should be used instead of \ ``f:h()``\  or \ ``f:js()``\ . It is equivalent to  \ ``${f:h(f:js())}``\ .

This is because, when \ ``"');alert("XSS Attack");// "``\  is specified as event handler value such as \ ``<input type="submit" onclick="callback('xxxx');">``\ , different script gets inserted, after escaping the value in character reference format, escaping in HTML needs to be done.

Example of vulnerability when output values are not escaped
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Example of occurrence of XSS problem is given below.

.. code-block:: jsp

    <input type="text" onmouseover="alert('output is ${warnCode}') . ">

.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 80

   * - Attribute name
     - Value
   * - | warnCode
     - | ``'); alert('XSS Attack!'); //``
       | When the above values are set, string literal is terminated unintentionally leading to XSS attack.

XSS dialog box is displayed on mouse over.

.. figure:: ./images_XSS/eventhandler_xss_screen_no_escape_result.png
   :alt: eventhandler_xss_screen_no_escape_result
   :width: 50%
   :align: center

   **Picture - No Escape Result**


**Output result**

.. code-block:: jsp

    <!-- omitted -->
    <input type="text" onmouseover="alert('output is'); alert('XSS Attack!'); // .') ">
    <!-- omitted -->

.. _xss_how_to_use_hjs_function_example:

Example of escaping output value using f:hjs() function
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Example is shown below:

.. code-block:: jsp

    <input type="text" onmouseover="alert('output is ${f:hjs(warnCode)}') . ">  // (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Value after escaping by EL function \ ``f:hjs()``\  is set as an argument of javascript event handler.

| XSS dialog is not output on mouse over.

.. figure:: ./images_XSS/eventhandler_xss_screen_escape_result.png
   :alt: eventhandler_xss_screen_escape_result
   :width: 50%
   :align: center

   **Picture - Escape Result**

**Output result**

.. code-block:: jsp

    <!-- omitted -->
    <input type="text" onmouseover="alert('output is \&#39;); alert(\&#39;XSS Attack!\&#39;);\&quot; \/\/ .') ">
    <!-- omitted -->

.. raw:: latex

   \newpage

