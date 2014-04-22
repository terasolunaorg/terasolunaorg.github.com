XSS(Cross Site Scripting) Countermeasures
================================================================================

Overview
--------------------------------------------------------------------------------
Cross Site Scripting is injection of malicious scripts across trusted web sites
used intentionally making use of the security flaws of the web application.
For example, when data (form input) entered in web application is output in HTML where appropriate escaping is not performed,
the characters of tag existing in input value is interpreted as HTML.
Session hijack occurs when script with a malicious value acquires the cookie value and
tampers cookie information.

Stored, Reflected XSS Attacks
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
XSS attacks are broadly classified into two categories.

**Stored XSS Attacks**

Stored XSS Attacks are those where the malicious code is permanently stored on target servers
such as in a database.
The victim then retrieves the malicious script from the server when it requests the stored information.

**Reflected XSS Attacks**

Reflected attacks are those where the malicious code is reflected off the web server
such as in an error message, search result, or any other response that includes some or all of the input sent to the server as part of the request.
When a user is tricked into clicking on a malicious link, submitting a specially crafted form, or even just browsing to a malicious site,
the injected code travels to the vulnerable web site, which reflects the attack back to the user's browser.
The browser then executes the code because it came from a "trusted" server.

Both Stored XSS Attacks and Reflected XSS Attacks can be prevented by escaping output value.

How to use
--------------------------------------------------------------------------------
When the input from user is output as it is, XSS vulnerability occurs.
Therefore, as a countermeasure against XSS vulnerability, escaping of characters having specific meaning
should be performed in HTML markup language.

Escaping is of 3 types depending on the requirements.

Escaping types:

 * Output Escaping
 * JavaScript Escaping
 * Event handler Escaping

Output Escaping
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
HTML escaping for special characters is performed as a countermeasure against XSS vulnerability.
Examples of before and after escaping for special characters in HTML are shown below:

.. tabularcolumns:: |p{0.50\linewidth}|p{0.50\linewidth}|
.. list-table:: 
   :header-rows: 1
   :widths: 50 50

   * - Before escaping
     - After escaping
   * - ``&``
     - ``&amp;``
   * - ``<``
     - ``&lt;``
   * - ``>``
     - ``&gt;``
   * - ``"``
     - ``&quot;``
   * - ``'``
     - ``&#39;``

| ``f:h()`` should be used in all display items to be output as string to prevent XSS.
| An example of application where input value is to be redisplayed on another screen is described.

Example of vulnerability where output value is not escaped
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Since the example is only described as reference example, the implementation as follows should not be performed.

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

   * - Sr.No.
     - Description
   * - | (1)
     - | It is customerForm field and job is output without escaping.

| Enter <script> tag in Job field on input screen.

.. figure:: ./images/xss_screen_input_html_tag.png 
   :alt: input_html_tag
   :width: 80%
   :align: center
 
   **Picture - Input HTML Tag**

|It is recognized as <script> tag and dialog box is displayed.

.. figure:: ./images/xss_screen_no_escape_result.png 
   :alt: no_escape_result
   :width: 60%
   :align: center
 
   **Picture - No Escape Result**

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

   * - Sr.No.
     - Description
   * - | (1)
     - | Value is output after escaping by ``f:h()``  of EL expressions.

| Enter <script> tag in Job field on input screen.

.. figure:: ./images/xss_screen_input_html_tag.png 
   :alt: input_html_tag
   :width: 80%
   :align: center
 
   **Picture - Input HTML Tag**

| By escaping special characters,
| input value is output as it is without considering as the <script> tag.

.. figure:: ./images/xss_screen_escape_result.png 
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

.. tip::

    **java.util.Date inheritance class format**

    It is recommended to use ``<fmt:formatDate>`` of JSTL to format and display java.util.Date inheritance class.
    Example of settings is described below.

        .. code-block:: jsp

            <fmt:formatDate value="${form.date}" pattern="yyyyMMdd" />

    Value is set to String using ``f:h()`` described earlier in 'value' and since ``javax.el.ELException``  is thrown, ``${form.date}`` is used.
    However, it is safe from XSS attack since the value is in yyyyMMdd format.

.. tip::

    **String that can be passed to java.lang.Number inheritance class or java.lang.Number**

    It is recommended to use ``<fmt:formatNumber>`` to format and display the string that can be passed to java.lang.Number inheritance class or java.lang.Number.
    Example of settings is described below.

        .. code-block:: jsp

            <fmt:formatNumber value="${f:h(form.price)}" pattern="###,###" />

    Since there is no problem even if the above is String, ``f:h()`` is explicitly used to prevent forgetting to add ``f:h()``  when ``<fmt:formatNumber>`` tag is no longer used.

JavaScript Escaping
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| JavaScript escaping for special characters is performed as a countermeasure against XSS vulnerability.
| Escaping should be performed to dynamically generate string literal of JavaScript based on the input from outside.

| Example of before and after JavaScript escaping for special characters is shown below.

.. tabularcolumns:: |p{0.50\linewidth}|p{0.50\linewidth}|
.. list-table:: 
   :header-rows: 1
   :widths: 50 50

   * - Before escaping
     - After escaping
   * - ``'``
     - ``\'``
   * - ``"``
     - ``\"``
   * - ``\``
     - ``\\``
   * - ``/``
     - ``\/``
   * - ``<``
     - ``\x3c``
   * - ``>``
     - ``\x3e``
   * - ``0x0D(Return)``
     - ``\r``
   * - ``0x0A(Linefeed)``
     - ``\n``

Example of vulnerability where escaping of output value is not performed
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Example of occurrence of XSS problem is described below.

.. code-block:: jsp

  <html>
    <script  type="text/javascript">
        var aaa = '${warnCode}';
        document.write(aaa);
    </script>
  <html>

.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table:: 
   :header-rows: 1
   :widths: 20 80

   * - Attribute name
     - Value
   * - warnCode
     - ``"foo"; <script>alert("XSS Attack!");</script> dummy=``

| To output code derived from input provided by the user
| and to dynamically generate JavaScript elements as shown in the above example, string literal is closed unintentionally and XSS vulnerability occurs.

.. figure:: ./images/javascript_xss_screen_no_escape_result.png 
   :alt: javascript_xss_screen_no_escape_result
   :width: 30%
   :align: center
 
   **Picture - No Escape Result**

**Output result**

.. code-block:: html

    <script  type="text/javascript">
        var aaa = "\"foo\"; <script>alert(\"XSS!\");<\/script> dummy=";
        document.write(aaa);
    </script>

.. tip::

    As long as there is no business requirement, JavaScript elements are dynamically generated depending on the input from outside
    since there is a possibility of an arbitrary script being inserted, consider another method or avoid it to the extent possible.

Example of escaping output value using f:js() function
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| To prevent XSS, it is recommended to use EL expression function and ``f:js()`` for the value entered by the user.

Usage example is shown below.

.. code-block:: jsp

  <script type="text/javascript">
    var message = '${f:js(message)}';  // (1)
    <!-- omitted -->
  </script>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr.No.
     - Description
   * - | (1)
     - | It is an argument after escaping by ``f:js()`` of EL expressions.

**Output result**

.. code-block:: jsp

    <script  type="text/javascript">
        var aaa = '\x3cscript\x3ealert(XSS Attack!);\x3c\/script\x3e';
        document.write(aaa);
    </script>

Event handler Escaping
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| To escape event handler of javascript, ``f:hjs()``  should be used instead of ``f:h()`` or ``f:js()`` 
| It is same as  ``${f:h(f:js())}``.

| When ``"');alert("XSS Atack");// "`` is specified as event handler value such as ``<input type="submit" onclick="callback('xxxx');">``,
| since another script can be inserted, ``"');alert("XSS Atack");// "``,
| HTML escaping should be performed after performing escaping in character reference format.

Example of inability to escape vulnerability of output value
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Examples of XSS occurrence are described below.

.. code-block:: jsp

    <input type="text" onmouseover="alert('output is ${warnCode}') . ">

.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table:: 
   :header-rows: 1
   :widths: 20 80

   * - Attribute name
     - Value
   * - warnCode
     - | ``'); alert('XSS Attack!'); //``
       | When the above values are set, string literal is closed unintentionally and XSS attack occurs.

| XSS dialog box is displayed on mouseover.

.. figure:: ./images/eventhandler_xss_screen_no_escape_result.png 
   :alt: eventhandler_xss_screen_no_escape_result
   :width: 50%
   :align: center
 
   **Picture - No Escape Result**


**Output result**

.. code-block:: jsp

    <!-- omitted -->
    <input type="text" onmouseover="alert('output is'); alert('XSS Attack!'); // .') ">
    <!-- omitted -->


Example of escaping output value using f:hjs() function
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Usage example is shown below:

.. code-block:: jsp

    <input type="text" onmouseover="alert('output is ${f:hjs(warnCode)}') . ">  // (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: 
   :header-rows: 1
   :widths: 10 90

   * - Sr.No.
     - Description
   * - | (1)
     - | It is an argument after escaping by ``f:hjs()`` of EL expressions.

| XSS dialog is not output on mouseover.

.. figure:: ./images/eventhandler_xss_screen_escape_result.png 
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

