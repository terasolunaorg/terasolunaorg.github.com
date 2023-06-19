Message Management
================================================================================

.. only:: html

 .. contents:: Table of Contents
    :depth: 4
    :local:

Overview
--------------------------------------------------------------------------------

| A message consists of fixed text displayed on screens or reports, or dynamic text displayed depending on screen operations performed by user.
| It is also recommended to define a error message in as much details as possible.

\

    .. warning::
       In following cases, there is a risk of inability to identify error cause during the production phase or during the testing just before entering into production phase (however, such risks may not surface during the development phase).

       * When only one error message is defined
       * When only two types of error messages ("Important" and "Warning") are defined

       Thus, if messages are changed when the number of developers in the team is less, the cost to modify these messages would increase as the development progresses.
       It is, therefore, recommended to define the messages in advance at a detailed level.

Types of messages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| Message that display according to the result of the user's screen operation should be classified into following 3 types depending on the message contents.
| When defining any message, it important to note its type.

.. _message-level-table-label:

.. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 20 60

   * - Message type
     - Category
     - Overview
   * - info
     - Information message
     - This message is displayed when a process is executed normally by the user.
   * - warn
     - Warning message
     - This message is displayed to indicate waring to be focused on; however the process can be continued. (Example: Notification indicating that the password is about to expire)
   * - error
     - Input error message
     - This message is displayed on input screen when value entered by the user is invalid.
   * -
     - Business error message
     - This message is displayed to indicate error in the business logic
   * -
     - System error message
     - This message is displayed when system errors (database connection failure etc.) occur and recovery is not possible by user operations.

|

Types of messages depending on patterns
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Message output patterns are shown below.

.. figure:: ./images/message-pattern.png
   :alt: message pattern
   :width: 95%

Message patterns, message display contents and the message type are shown below.

.. tabularcolumns:: |p{0.05\linewidth}|p{0.15\linewidth}|p{0.20\linewidth}|p{0.10\linewidth}|p{0.50\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 5 15 20 10 50

   * - Symbol
     - Pattern
     - Display contents
     - Message type
     - Example
   * - | (A)
     - | Title
     - | Screen title
     - | -
     - * Employee Registration screen
   * - |
     - | Label
     - | Screen field name
       | Report field name
       | Comment
       | Guidance
     - | -
     - * User name
       * Password
   * - | (B)
     - | Dialog
     - | Confirmation message
     - | info
     - * Are you sure you want to register?
       * Are you sure you want to delete?
   * - | (C)
     - | Result message
     - | Successful completion
     - | info
     - * Registered.
       * Deleted.
   * - | (D)
     - |
     - | Warning
     - | warn
     - * Password is about to expire. Please change the password.
       * Server is busy. Please try again later.
   * - | (E)
     - |
     - | Single field validation error
     - | error
     - * "User name" is mandatory.
       * Please enter "Name" within 20 characters.
       * Please enter the "Amount" in number.
   * - | (F)
     - |
     - | Correlation check error
     - | error
     - * "Password" and "Confirm Password" do not match.
   * - | (G)
     - |
     - | Business error
     - | error
     - * Failed to cancel the reservation as cancellation period has elapsed.
       * Failed to register as number of allowed registrations exceeded.
   * - | (H)
     - |
     - | System error
     - | error
     - * XXXSystem is blocked, please try again later.
       * Timeout has occurred.
       * System error.

Message ID
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| For effective message management, adding an ID to the message is recommended.
| The advantages of adding an ID are as follows:

* To change the message without modifying the source code.
* To be able to identify the message output location easily
* To support internationalization

From maintenance perspective, it is strongly recommended that you define the message IDs by creating and standardizing the rules.

| See the example below for Message ID rules for each message pattern.
| Refer to these rules when message ID rules are not defined in a development project.

Title
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| The method of defining message ID to be used in screen title is described below.


* Format

    .. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 20 20 20 20 20

       * - Prefix
         - Delimiter
         - Business process name
         - Delimiter
         - Screen name
       * - | title
         - | .
         - | nnn*
         - | .
         - | nnn*

* Description

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.25\linewidth}|p{0.35\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 10 25 35

       * - Field
         - Position
         - Contents
         - Remarks
       * - | Prefix
         - | 1st - 5th digit (5 digits)
         - | "title" (fixed)
         - |
       * - | Business process name
         - | Variable length：Optional
         - | Directory under prefix of viewResolver defined in spring-mvc.xml (parent directory of JSP)
         - |
       * - | Screen name
         - | Variable length：Optional
         - | JSP name
         - | "aaa" when file name is "aaa.jsp"

* Example

    .. code-block:: properties

        # In case of "/WEB-INF/views/admin/top.jsp"
        title.admin.top=Admin Top
        # In case of "/WEB-INF/views/staff/createForm.jsp"
        title.staff.createForm=Staff Register Input

    .. tip::

       This example is valid when using Tiles. For details, refer to :doc:`TilesLayout`.
       When not using Tiles, follow the \ :ref:`message-management_label-rule`\  rules explained later.

|

.. _message-management_label-rule:

Labels
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The method of defining message ID to be used in screen label and fixed text of reports is described below.


* Format

    .. tabularcolumns:: |p{0.14\linewidth}|p{0.14\linewidth}|p{0.16\linewidth}|p{0.14\linewidth}|p{0.14\linewidth}|p{0.14\linewidth}|p{0.14\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 14 14 16 14 14 14 14

       * - Prefix
         - Delimiter
         - Project code
         - Delimiter
         - Business process name
         - Delimiter
         - Field name
       * - | label
         - | .
         - | xx
         - | .
         - | nnn*
         - | .
         - | nnn*


* Description

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.25\linewidth}|p{0.35\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 10 25 35

       * - Field
         - Position
         - Contents
         - Remarks
       * - | Prefix
         - | 1st - 5th digit (5 digits)
         - | "label" (Fixed)
         - |
       * - | Project code
         - | 7th - 8th digit (2 digits)
         - | Enter 2 alphabets of project name
         - |
       * - | Business process name
         - | Variable length：Optional
         - |
         - |
       * - | Field name
         - | Variable length：Optional
         - | Label name, Caption
         - |

    .. note::

        When including the field name into validation error message, define messages as follows.

        * model attribute name of form + "." + field name

         .. code-block:: properties

            staffForm.staffName = Staff name

        * filed name

         .. code-block:: properties

            staffName = Staff name

* Example

    .. code-block:: properties

        # Form field name on Staff Registration screen
        # Project code=em (Event Management System)
        label.em.staff.staffName=Staff name
        # In case of a caption to be displayed on Tour Search screen
        # Project code=tr (Tour Reservation System)
        label.tr.tourSearch.tourSearchMessage=You can search tours with the specified conditions.

    .. note::

        In case of multiple projects, define a project code to avoid duplication of message ID.
        Even if there is a single project, it is recommended to define a project code for future enhancements.

.. _message-management_result-rule:

Result messages
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Messages commonly used in business processes
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

To avoid duplication of messages, the messages which are common in multiple business processes are explained below.

* Format

    .. tabularcolumns:: |p{0.12\linewidth}|p{0.12\linewidth}|p{0.14\linewidth}|p{0.12\linewidth}|p{0.14\linewidth}|p{0.12\linewidth}|p{0.12\linewidth}|p{0.12\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 12 12 14 12 14 12 12 12

       * - Message type
         - Delimiter
         - Project code
         - Delimiter
         - Common message code
         - Delimiter
         - Error level
         - Sr. No.
       * - | x
         - | .
         - | xx
         - | .
         - | fw
         - | .
         - | 9
         - | 999

* Description

    .. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.40\linewidth}|p{0.10\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 20 20 40 10

       * - Field
         - Position
         - Contents
         - Remarks
       * - | Message type
         - | 1st digit (1 digit)
         - | info  : i
           | warn  : w
           | error : e
         - |
       * - | Project code
         - | 3rd - 4th digit (2 digits)
         - | Enter 2 alphabets of project name
         - |
       * - | Common message code
         - | 6th - 7th digit (2 digits)
         - | "fw" (fixed)
         - |
       * - | Error level
         - | 9th digit (1 digit)
         - | 0-1 : Normal message
           | 2-4 : Business error (semi-normal message)
           | 5-7 : Input validation error
           | 8 : Business error (error)
           | 9 : System error
         - |
       * - | Sr. No.
         - | 10th -12th digit (3 digits)
         - | Use as per serial number (000-999)
         - | Even if the message is deleted, serial number field should be blank and it should not be deleted.

* Example

    .. code-block:: properties

        # When registration is successful (Normal message)
        i.ex.fw.0001=Registered successfully.
        # Insufficient server resources
        w.ex.fw.9002=Server busy. Please, try again.
        # When system error occurs (System error)
        e.ex.fw.9001=A system error has occurred.

.. _message-properties-example:

Messages used individually in each business process
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

The messages used individually in each business process are explained below.

* Format

    .. tabularcolumns:: |p{0.12\linewidth}|p{0.12\linewidth}|p{0.14\linewidth}|p{0.12\linewidth}|p{0.14\linewidth}|p{0.12\linewidth}|p{0.12\linewidth}|p{0.12\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 12 12 14 12 14 12 12 12

       * - Message type
         - Delimiter
         - Project code
         - Delimiter
         - Business process message code
         - Delimiter
         - Error level
         - Sr. No.
       * - | x
         - | .
         - | xx
         - | .
         - | xx
         - | .
         - | 9
         - | 999

* Description

    .. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.40\linewidth}|p{0.10\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 20 20 40 10

       * - Field
         - Position
         - Contents
         - Remarks
       * - | Message type
         - | 1st digit (1 digit)
         - | info  : i
           | warn  : w
           | error : e
         - |
       * - | Project code
         - | 3rd -4th digit (2 digits)
         - | Enter 2 alphabets of project name
         - |
       * - | Business process message code
         - | 6th -7th digit (2 digits)
         - | 2 characters defined for each business process such as Business ID
         - |
       * - | Error level
         - | 9th digit (1 digit)
         - | 0-1 : Normal message
           | 2-4 : Business error (semi-normal message)
           | 5-7 : Input validation error
           | 8 : Business error (error)
           | 9 : System error
         - |
       * - | Sr. No.
         - | 10th -12th digit (3 digits)
         - | Use as per serial number (000-999)
         - | Even if the message is deleted, serial number field should be blank and it should not be deleted.


* Example

    .. code-block:: properties

        # When file upload is successful.
        i.ex.an.0001={0} upload completed.
        # When the recommended password change interval has passed.
        w.ex.an.2001=The recommended change interval has passed password. Please change your password.
        # When file size exceeds the limit.
        e.ex.an.8001=Cannot upload, Because the file size must be less than {0}MB.
        # When there is inconsistency in data.
        e.ex.an.9001=There are inconsistencies in the data.

|

Input validation error message
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

For the messages to be displayed in case of input validation error, refer to \ :ref:`Validation_message_def`\ .


    .. note::

        Basic policies related to output location of input validation error are as follows:

        * | Single field input validation error messages should be displayed next to the target field so that it can be identified easily.
        * | Correlation input validation error messages should be displayed collectively on the top of the page .
        * | When it is difficult to display the single field validation message next to the target field, it should be displayed on the top of the page.
          | In that case, field name should be included in the message.

|

How to use
--------------------------------------------------------------------------------

Display of messages set in properties file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Settings at the time of using properties
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Define implementation class of \ ``org.springframework.context.MessageSource``\  which is used for performing message management.

* applicationContext.xml

    .. code-block:: xml

        <!-- Message -->
        <bean id="messageSource"
            class="org.springframework.context.support.ResourceBundleMessageSource"> <!-- (1) -->
            <property name="basenames"> <!-- (2) -->
                <list>
                    <value>i18n/application-messages</value>
                </list>
            </property>
        </bean>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - Sr. No.
         - Description
       * - | (1)
         - | Definition of ``MessageSource``\. Here, use \ ``ResourceBundleMessageSource``\  .
       * - | (2)
         - | Define the base name of message property to be used. Specify it with relative class path.
           | In this example, read "src/main/resources/i18n/application-messages.properties".

Display of messages set in properties
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* application-messages.properties

    See the example below for defining the messages in \ :file:`application-messages.properties`\  .

    .. code-block:: properties

        label.aa.bb.year=Year
        label.aa.bb.month=Month
        label.aa.bb.day=Day


    .. note::

        Earlier, it was necessary to convert the characters (such as Japanese characters etc.) that cannot be expressed into "ISO-8859-1"
        with the help of \ ``native2ascii``\  command. However, from JDK version 6 onwards, it has become possible to specify the character encoding;
        hence character conversion is no longer needed. By setting the character encoding to UTF-8, Japanese characters etc. can be used directly in properties file.

        * application-messages.properties

            .. code-block:: properties

                label.aa.bb.year= Year
                label.aa.bb.month= Month
                label.aa.bb.day= Day

        In such a case, it is necessary to specify the character encoding that can also be read in \ ``ResourceBundleMessageSource``\  .

        * applicationContext.xml

            .. code-block:: java
                :emphasize-lines: 8

                <bean id="messageSource"
                    class="org.springframework.context.support.ResourceBundleMessageSource">
                    <property name="basenames">
                        <list>
                            <value>i18n/application-messages</value>
                        </list>
                    </property>
                    <property name="defaultEncoding" value="UTF-8" />
                </bean>

        ISO-8859-1 is used by default; hence when describing the Japanese characters directly in properties file,
        make sure that the character encoding is set as value of \ ``defaultEncoding``\  property.

* JSP

    Messages set above can be displayed using \ ``<spring:message>``\  tag in JSP.
    For using it, settings mentioned in \ :ref:`view_jsp_include-label`\  must be done.

    .. code-block:: jsp

        <spring:message code="label.aa.bb.year" />
        <spring:message code="label.aa.bb.month" />
        <spring:message code="label.aa.bb.day" />

    When used with form label, it can be used as follows:

    .. code-block:: jsp
        :emphasize-lines: 3,7,11

        <form:form modelAttribute="sampleForm">
            <form:label path="year">
                <spring:message code="label.aa.bb.year" />
            </form:label>: <form:input path="year" />
            <br>
            <form:label path="month">
                <spring:message code="label.aa.bb.month" />
            </form:label>: <form:input path="month" />
            <br>
            <form:label path="day">
                <spring:message code="label.aa.bb.day" />
            </form:label>: <form:input path="day" />
        </form:form>


    It is displayed in browser as follows:

    .. figure:: ./images_MessageManagement/message-management-ymd.png
        :width: 40%

    .. tip::

        When supporting internationalization,

        .. code-block:: properties

            src/main/resources/i18n
                                ├ application-messages.properties (English message)
                                ├ application-messages_fr.properties (French message)
                                ├ ...
                                └ application-messages_ja.properties (Japanese message)

        properties file should be created for each language as shown above.
        For details, refer to \ :doc:`./Internationalization`\  .


.. _message-display:

Display of result messages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ ``org.terasoluna.gfw.common.message.ResultMessages``\  and \ ``org.terasoluna.gfw.common.message.ResultMessage``\  are provided in common library,
as classes storing the result messages which indicate success or failure of process at server side.

.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 20 80

  * - Class name
    - Description
  * - | ``ResultMessages``
    - | Class having list of result messages and message type.
      | List of Result messages is expressed in terms of \ ``List<ResultMessage>``\  and message type is expressed in terms of \ ``org.terasoluna.gfw.common.message.ResultMessageType``\  interface.
  * - | ``ResultMessage``
    - | Class having result message ID or message text.

| \ ``<t:messagesPanel>``\  tag is also provided as JSP tag library for displaying this result message in JSP.

Using basic result messages
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
The way of creating \ ``ResultMessages``\  in Controller, passing them to screen and displaying
the result messages using \ ``<t:messagesPanel>``\  tag in JSP, is displayed below.

* Controller class

    The methods of creating \ ``ResultMessages``\  object and passing the messages to screen are given below.
    An example of \ :ref:`message-properties-example`\  should be defined in application-messages.properties.

    .. code-block:: java

        package com.example.sample.app.message;

        import org.springframework.stereotype.Controller;
        import org.springframework.ui.Model;
        import org.springframework.web.bind.annotation.RequestMapping;
        import org.springframework.web.bind.annotation.RequestMethod;
        import org.terasoluna.gfw.common.message.ResultMessages;

        @Controller
        @RequestMapping("message")
        public class MessageController {

          @RequestMapping(method = RequestMethod.GET)
          public String hello(Model model) {
            ResultMessages messages = ResultMessages.error().add("e.ex.an.9001"); // (1)
            model.addAttribute(messages); // (2)
            return "message/index";
          }
        }


    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - Sr. No.
        - Description
      * - | (1)
        - | Create \ ``ResultMessages``\  wherein message type is "error" and
          | set result messages wherein message ID is "e.ex.an.9001".
          | This process is same as follows:
          | ``ResultMessages.error().add(ResultMessage.fromCode("e.ex.an.9001"));``
          | Since it is possible to skip the creation of  \ ``ResultMessage``\  object if message ID is specified, it is recommended to skip the same.
      * - | (2)
        - | Add \ ``ResultMessages``\  to Model.
          | It is ok even if the attribute is not specified. (Attribute name is "resultMessages")



* JSP

    Write WEB-INF/views/message/index.jsp as follows:

    .. code-block:: jsp

        <!DOCTYPE HTML>
        <html>
        <head>
        <meta charset="utf-8">
        <title>Result Message Example</title>
        </head>
        <body>
            <h1>Result Message</h1>
            <t:messagesPanel /><!-- (1) -->
        </body>
        </html>


    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - Sr. No.
        - Description
      * - | (1)
        - | ``<t:messagesPanel>`` tag is used with default settings.
          | By default, "resultMessages" object is displayed.
          | Therefore, attribute name need not be specified when \ ``ResultMessages``\  is set in Model from Controller with default settings.

    It is displayed in browser as follows:


    .. figure:: ./images_MessageManagement/message-management-resultmessage-basic.png
        :width: 40%


    HTML output by \ ``<t:messagesPanel>``\  is shown below. (The format makes the explanation easier).

    .. code-block:: html

        <div class="alert alert-error"><!-- (1) -->
          <ul><!-- (2) -->
            <li>There are inconsistencies in the data.</li><!-- (3) -->
          </ul>
        </div>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - Sr. No.
        - Description
      * - | (1)
        - | "alert-error"class is assigned in accordance with the message type. "error error-[Message type]" is assigned to \ ``<div>``\  tag class by default.
      * - | (2)
        - | Result message list is output using \ ``<ul>``\  tag.
      * - | (3)
        - | The message corresponding to message ID is resolved from \ ``MessageSource``\ .


    \ ``<t:messagesPanel>``\  outputs only HTML with class; hence it is necessary to customize the look and feel using CSS as per the output class (explained later).

    .. note::

        Message text can be hard-coded such as \ ``ResultMessages.error().add(ResultMessage.fromText("There are inconsistencies in the data."));``\ ;
        however, to enhance maintainability, it is recommended to create \ ``ResultMessage``\  object using message key,
        and to fetch the message text from properties file.

|

For inserting a value in message placeholder, set second or subsequent arguments of \ ``add``\  method as follows:

.. code-block:: java

    ResultMessages messages = ResultMessages.error().add("e.ex.an.8001", 1024);
    model.addAttribute(messages);

In such a case, the HTML shown below is output using \ ``<t:messagesPanel />``\  tag.

.. code-block:: html

    <div class="alert alert-error">
      <ul>
        <li>Cannot upload, Because the file size must be less than 1,024MB.</li>
      </ul>
    </div>

\

 .. warning:: **Points to be noted when inserting values in placeholder using terasoluna-gfw-web 1.0.0.RELEASE**

        When using terasoluna-gfw-web 1.0.0.RELEASE, \ **if the user entered value is inserted in the placeholder, there is a risk of XSS vulnerability.**\ 
        If the user entered value is likely to include XSS vulnerable characters, then the value should not be inserted in the placeholder.
        
        When using terasoluna-gfw-web 1.0.1.RELEASE or higher version, XSS vulnerability does not occur even after inserting the user entered value in the placeholder.
        
 .. note::
 
    \ ``ResourceBundleMessageSource``\  uses \ ``java.text.MessageFormat``\  at the time of creating a message; hence \ ``1024``\  is displayed as
    \ ``1,024``\  with comma. When comma is not required, perform settings in properties file as shown below.
 
        .. code-block:: properties

            e.ex.an.8001=Cannot upload, Because the file size must be less than {0,number,#}MB.

    For details, refer to \ `Javadoc <http://docs.oracle.com/javase/8/docs/api/java/text/MessageFormat.html>`_\ .

|

It is also possible to set multiple result messages as shown below.

.. code-block:: java

    ResultMessages messages = ResultMessages.error()
        .add("e.ex.an.9001")
        .add("e.ex.an.8001", 1024);
    model.addAttribute(messages);

In such a case, HTML is output as follows (no need to change JSP).

.. code-block:: html

    <div class="alert alert-error">
      <ul>
        <li>There are inconsistencies in the data.</li>
        <li>Cannot upload, Because the file size must be less than 1,024MB.</li>
      </ul>
    </div>

In order to display info message, it is desirable to create \ ``ResultMessages``\  object using \ ``ResultMessages.info()``\  method as shown below.

.. code-block:: java

    ResultMessages messages = ResultMessages.info().add("i.ex.an.0001", "XXXX");
    model.addAttribute(messages);

HTML shown below is output.

.. code-block:: html

  <div class="alert alert-info"><!-- (1) -->
    <ul>
      <li>XXXX upload completed.</li>
    </ul>
  </div>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - Sr. No.
    - Description
  * - | (1)
    - | The output class name has changed to "alert alert-**info**" in accordance with the message type.

Fundamentally the following message types are created.


.. tabularcolumns:: |p{0.15\linewidth}|p{0.30\linewidth}|p{0.25\linewidth}|p{0.30\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 15 30 25 30

  * - Message type
    - Creation of \ ``ResultMessages``\  object
    - Default class name
    - Remarks
  * - | success
    - | ``ResultMessages.success()``\
    - | alert alert-success
    - | \-
  * - | info
    - | \ ``ResultMessages.info()``\
    - | alert alert-info
    - | \-
  * - | warn
    - | \ ``ResultMessages.warn()``\
    - | alert alert-warn
    - | This type is deprecated because added "warning" as new message type from terasoluna-gfw-common 5.0.0.RELEASE.
      | **This message type might be removed in the future.**
  * - | warning
    - | \ ``ResultMessages.warning()``\
    - | alert alert-warning
    - | Added this message type to support \ `Alerts component <http://getbootstrap.com/components/#alerts>`_\  of \ `Bootstrap(CSS Framework) <http://getbootstrap.com/>`_\  from the terasoluna-gfw-common 5.0.0.RELEASE.
  * - | error
    - | \ ``ResultMessages.error()``\
    - | alert alert-error
    - | \-
  * - | danger
    - | \ ``ResultMessages.danger()``\
    - | alert alert-danger
    - | \-

CSS should be defined according to the message type. Example of applying CSS is given below.

.. code-block:: css

    .alert {
      margin-bottom: 15px;
      padding: 10px;
      border: 1px solid;
      border-radius: 4px;
      text-shadow: 0 1px 0 #ffffff;
    }
    .alert-info {
      background: #ebf7fd;
      color: #2d7091;
      border-color: rgba(45, 112, 145, 0.3);
    }
    .alert-warning {
      background: #fffceb;
      color: #e28327;
      border-color: rgba(226, 131, 39, 0.3);
    }
    .alert-error {
      background: #fff1f0;
      color: #d85030;
      border-color: rgba(216, 80, 48, 0.3);
    }

* Example wherein \ ``ResultMessages.error().add("e.ex.an.9001")``\  is output using \ ``<t:messagesPanel />``\


    .. figure:: ./images_MessageManagement/message-management-resultmessage-error.jpg
        :width: 100%


* Example wherein \ ``ResultMessages.warning().add("w.ex.an.2001")``\  is output using \ ``<t:messagesPanel />``\


    .. figure:: ./images_MessageManagement/message-management-resultmessage-warn.jpg
        :width: 100%


* Example wherein \ ``ResultMessages.info().add("i.ex.an.0001", "XXXX")``\  is output using \ ``<t:messagesPanel />``\


    .. figure:: ./images_MessageManagement/message-management-resultmessage-info.jpg
        :width: 100%

    .. note::

        "success" and "danger" are provided to have diversity in style. In this guideline, success is synonymous with info and error is synonymous with danger.

    .. tip::

        \ `Alerts component <http://getbootstrap.com/components/#alerts>`_\  of \ `Bootstrap <http://getbootstrap.com/>`_ 3.0.0 which is a CSS framework can be used with default settings of \ ``<t:messagePanel />``\ .

    .. warning::

        In this example, message keys are hardcoded. However, in order to improve maintainability, it is recommended to define message keys in constant class.

        Refer to :ref:`message-management-messagekeysgen`\.

Specifying attribute name of result messages
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Attribute name can be omitted when adding \ ``ResultMessages``\  to Model.
| However, \ ``ResultMessages``\  cannot represent more than one message type.
| In order to \ **simultaneously**\  display the \ ``ResultMessages``\  of different message types on 1 screen, it is necessary to specify the attribute name explicitly and set it in Model.

* Controller (Add to MessageController)

    .. code-block:: java

        @RequestMapping(value = "showMessages", method = RequestMethod.GET)
        public String showMessages(Model model) {

            model.addAttribute("messages1",
                        ResultMessages.warning().add("w.ex.an.2001")); // (1)
            model.addAttribute("messages2",
                        ResultMessages.error().add("e.ex.an.9001")); // (2)

            return "message/showMessages";
        }



    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - Sr. No.
        - Description
      * - | (1)
        - | Add \ ``ResultMessages``\  of "warning" message type to Model with attribute name "messages1".
      * - | (2)
        - | Add \ ``ResultMessages``\  of "info" message type to Model with attribute name "messages2".


* JSP (WEB-INF/views/message/showMessages.jsp)

    .. code-block:: jsp

        <!DOCTYPE HTML>
        <html>
        <head>
        <meta charset="utf-8">
        <title>Result Message Example</title>
        <style type="text/css">
        .alert {
            margin-bottom: 15px;
            padding: 10px;
            border: 1px solid;
            border-radius: 4px;
            text-shadow: 0 1px 0 #ffffff;
        }

        .alert-info {
            background: #ebf7fd;
            color: #2d7091;
            border-color: rgba(45, 112, 145, 0.3);
        }

        .alert-warning {
            background: #fffceb;
            color: #e28327;
            border-color: rgba(226, 131, 39, 0.3);
        }

        .alert-error {
            background: #fff1f0;
            color: #d85030;
            border-color: rgba(216, 80, 48, 0.3);
        }
        </style>
        </head>
        <body>
            <h1>Result Message</h1>
            <h2>Messages1</h2>
            <t:messagesPanel messagesAttributeName="messages1" /><!-- (1) -->
            <h2>Messages2</h2>
            <t:messagesPanel messagesAttributeName="messages2" /><!-- (2) -->
        </body>
        </html>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - Sr. No.
        - Description
      * - | (1)
        - | Display \ ``ResultMessages``\  having attribute name "messages1".
      * - | (2)
        - | Display \ ``ResultMessages``\  having attribute name "messages2".

    It is displayed in browser as follows:

    .. figure:: ./images_MessageManagement/message-management-multiple-messages.jpg
        :width: 80%

Displaying business exception messages
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| \ ``org.terasoluna.gfw.common.exception.BusinessException``\  and \ ``org.terasoluna.gfw.common.exception.ResourceNotFoundException``\  stores
| \ ``ResultMessages``\  internally.

| When displaying the business exception message, \ ``BusinessException``\  wherein \ ``ResultMessages``\  is set should be thrown in Service class.
| Catch \ ``BusinessException``\  in Controller class and add the result message fetched from the caught exception to Model.

* Service class

    .. code-block:: java

        @Service
        @Transactional
        public class UserServiceImpl implements UserService {
            // omitted

            public void create(...) {

                // omitted...

                if (...) {
                    // illegal state!
                    ResultMessages messages = ResultMessages.error()
                                                            .add("e.ex.an.9001"); // (1)
                    throw new BusinessException(messages);
                }
            }

        }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - Sr. No.
        - Description
      * - | (1)
        - | Create error message using \ ``ResultMessages``\  and set in \ ``BusinessException``\ .

* Controller class

    .. code-block:: java

        @RequestMapping(value = "create", method = RequestMethod.POST)
        public String create(@Validated UserForm form, BindingResult result, Model model) {
            // omitted

            try {
                userService.create(user);
            } catch (BusinessException e) {
                ResultMessages messages = e.getResultMessages(); // (1)
                model.addAttribute(messages);

                return "user/createForm";
            }

            // omitted
        }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - Sr. No.
        - Description
      * - | (1)
        - | Fetch \ ``ResultMessages``\  held by \ ``BusinessException``\  and add to Model.


Normally, this method should be used to display error message instead of creating
\ ``ResultMessages``\  object in Controller.

|

How to extend
--------------------------------------------------------------------------------

Creating independent message types
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| The method of creating independent message type is given below.
| Normally, the available message types are sufficient. However, new message type may need to be added
| depending upon the CSS library. See the example below for adding the message type "notice". 


First, create independent message type class wherein \ ``org.terasoluna.gfw.common.message.ResultMessageType``\  interface is implemented
as follows:

.. code-block:: java

    import org.terasoluna.gfw.common.message.ResultMessageType;

    public enum ResultMessageTypes implements ResultMessageType { // (1)
        NOTICE("notice");

        private ResultMessageTypes(String type) {
            this.type = type;
        }

        private final String type;

        @Override
        public String getType() { // (2)
            return this.type;
        }

        @Override
        public String toString() {
            return this.type;
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - Sr. No.
    - Description
  * - | (1)
    - | Define Enum wherein \ ``ResultMessageType``\  interface is implemented. A new message type can be created using constant class; however it is recommended to create it using Enum.
  * - | (2)
    - | Return value of \ ``getType``\  corresponds to class name of CSS which is output.

| Create \ ``ResultMessages``\  using this message type as mentioned below.

.. code-block:: java

    ResultMessages messages = new ResultMessages(ResultMessageTypes.NOTICE) // (1)
            .add("w.ex.an.2001");
    model.addAttribute(messages);

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - Sr. No.
    - Description
  * - | (1)
    - | Specify \ ``ResultMessageType``\  in constructor of \ ``ResultMessages``\ .

In such a case, HTML shown below is output in \ ``<t:messagesPanel />`` \ .

.. code-block:: html

    <div class="alert alert-notice">
      <ul>
        <li>The recommended change interval has passed password. Please change your password.</li>
      </ul>
    </div>

\

    .. tip::

        For extension method, refer to \ ``org.terasoluna.gfw.common.message.StandardResultMessageType``\ .

|

Appendix
--------------------------------------------------------------------------------

.. _message-management-messagepanel-attribute:

Changing attribute of <t:messagesPanel> tag
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ ``<t:messagesPanel>``\  tag contains various attributes for changing the display format.

.. tabularcolumns:: |p{0.25\linewidth}|p{0.55\linewidth}|p{0.20\linewidth}|
.. list-table:: \ ``<t:messagesPanel>``\  Tag attribute list
   :header-rows: 1
   :widths: 25 50 25

   * - Option
     - Contents
     - Default setting value
   * - panelElement
     - Result message display panel elements
     - div
   * - panelClassName
     - CSS class name of result message display panel.
     - alert
   * - panelTypeClassPrefix
     - Prefix of CSS class name
     - alert-
   * - messagesType
     - Message type. When this attribute is set, the set message type is given preference over the message type of \ ``ResultMessages``\  object.
     -
   * - outerElement
     - Outer tag of HTML configuring result messages list
     - ul
   * - innerElement
     - Inner tag of HTML configuring result messages list
     - li
   * - disableHtmlEscape
     - | Flag for disabling HTML escaping.
       | By setting the flag to \ ``true``\ , HTML escaping is no longer performed for the message to be output.
       | This attribute is used to create different message styles by inserting HTML into the message to be output.
       | **When the flag is set to true, XSS vulnerable characters should not be included in the message.**
       |
       | This attribute can be used with terasoluna-gfw-web 1.0.1.RELEASE or higher version. 
     - ``false``


For example, following CSS is provided in CSS framework "\ `BlueTrip <http://www.bluetrip.org/>`_\ ".

.. code-block:: css

    .error,.notice,.success {
        padding: .8em;
        margin-bottom: 1.6em;
        border: 2px solid #ddd;
    }

    .error {
        background: #FBE3E4;
        color: #8a1f11;
        border-color: #FBC2C4;
    }

    .notice {
        background: #FFF6BF;
        color: #514721;
        border-color: #FFD324;
    }

    .success {
        background: #E6EFC2;
        color: #264409;
        border-color: #C6D880;
    }

| To use this CSS, the message \ ``<div class="error">...</div>``\  should be output.
| In this case, \ ``<t:messagesPanel>``\  tag can be used as follows (no need to modify the Controller):

.. code-block:: jsp

    <t:messagesPanel panelClassName="" panelTypeClassPrefix="" />

HTML shown below is output.

.. code-block:: html

    <div class="error">
      <ul>
        <li>There are inconsistencies in the data.</li>
      </ul>
    </div>

It is displayed in browser as follows:

.. figure:: ./images_MessageManagement/message-management-bluetrip-error.jpg
    :width: 80%

When you do not want to use \ ``<ul>``\  tag to display the message list,
it can be customized using \ ``outerElement``\  attribute and \ ``innerElement``\  attribute.

When the attributes are set as follows:

.. code-block:: jsp

    <t:messagesPanel outerElement="" innerElement="span" />


HTML shown below is output.


.. code-block:: html

    <div class="alert alert-error">
        <span>There are inconsistencies in the data.</span>
        <span>Cannot upload, Because the file size must be less than 1,024MB.</span>
    </div>

Set the CSS as follows:

.. code-block:: css

    .alert > span {
        display: block; /* (1) */
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - Sr. No.
    - Description
  * - | (1)
    - | Set \ ``<span>``\  tag which is a child element of "alert" class to Block-level element.

It is displayed in browser as follows:


.. figure:: ./images_MessageManagement/message-management-messagespanel-span.jpg
    :width: 60%


| When disableHtmlEscape attribute is set to \ ``true``\ , the output will be as follows:
| In the example below, font of a part of the message has been set to 16px Red.

- jsp

 .. code-block:: jsp
    :emphasize-lines: 4

    <spring:message var="informationMessage" code="i.ex.od.0001" />
    <t:messagesPanel messagesAttributeName="informationMessage"
        messagesType="alert alert-info"
        disableHtmlEscape="true" />

- properties

 .. code-block:: properties

    i.ex.od.0001 = Please confirm order content. <font style="color: red; font-size: 16px;">If this orders submitted, cannot cancel.</font>

- Output image

 .. figure:: ./images_MessageManagement/message-management-disableHtmlEscape-true.png
    :width: 100%
    
 When disableHtmlEscape attribute is \ ``false``\ (default), the output will be as follows after HTML escaping.

 .. figure:: ./images_MessageManagement/message-management-disableHtmlEscape-false.png
    :width: 100%


Display of result message wherein ResultMessages is not used
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Apart from \ ``ResultMessages``\  object, \ ``<t:messagesPanel>``\  tag can also output the following objects.

* ``java.lang.String``
* ``java.lang.Exception``
* ``java.util.List``



| Normally \ ``<t:messagesPanel>``\  tag is used to output the \ ``ResultMessages``\  object; however
| it can also be used to display strings (error messages) set in the request scope by the framework.

| For example, at the time of authentication error, Spring Security sets the exception class with attribute name "SPRING_SECURITY_LAST_EXCEPTION"
| in the request scope.

| Perform the following settings if you want to output this exception message in \ ``<t:messagesPanel>``\  tag similar to the result messages.


.. code-block:: jsp

    <!DOCTYPE HTML>
    <html>
    <head>
    <meta charset="utf-8">
    <title>Login</title>
    <style type="text/css">
    /* (1) */
    .alert {
        margin-bottom: 15px;
        padding: 10px;
        border: 1px solid;
        border-radius: 4px;
        text-shadow: 0 1px 0 #ffffff;
    }

    .alert-error {
        background: #fff1f0;
        color: #d85030;
        border-color: rgba(216, 80, 48, 0.3);
    }
    </style>
    </head>
    <body>
        <c:if test="${param.error}">
            <t:messagesPanel messagesType="error"
                messagesAttributeName="SPRING_SECURITY_LAST_EXCEPTION" /><!-- (2) -->
        </c:if>
        <form:form
            action="${pageContext.request.contextPath}/authentication"
            method="post">
            <fieldset>
                <legend>Login Form</legend>
                <div>
                    <label for="username">Username: </label><input
                        type="text" id="username" name="j_username">
                </div>
                <div>
                    <label for="username">Password:</label><input
                        type="password" id="password" name="j_password">
                </div>
                <div>
                    <input type="submit" value="Login" />
                </div>
            </fieldset>
        </form:form>
    </body>
    </html>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - Sr. No.
    - Description
  * - | (1)
    - | Re-define the CSS. It is strongly recommended to mention it in CSS file.
  * - | (2)
    - | In \ ``messagesAttributeName``\  attribute, specify the attribute name wherein \ ``Exception``\   object is stored.
      | Unlike the \ ``ResultMessages``\  object, it does not contain the information of message type; hence
      | it is necessary to explicitly specify the message type in \ ``messagesType``\  attribute.

The HTML output in case of an authentication error will be,

.. code-block:: html

    <div class="alert alert-error"><ul><li>Bad credentials</li></ul></div>

and it will be displayed in the browser as follows:

.. figure:: ./images_MessageManagement/message-management-login-error.jpg
    :width: 60%

\

    .. tip::

        For details on JSP for login, refer to \ :doc:`../Security/Authentication`\ .

.. _message-management-messagekeysgen:

Auto-generation tool of message key constant class
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
In all earlier examples, message keys were hard-coded strings; however
it is recommended that you define the message keys in constant class.

This section introduces the program that auto-generates message key constant class from properties file
and the corresponding usage method. You can customize and use them based on the requirements.

#. Creation of message key constant class

    First, create an empty message key constant class. Here, it is \ ``com.example.common.message.MessageKeys``\ .

    .. code-block:: java


        package com.example.common.message;

        public class MessageKeys {

        }

#. Creation of auto-generation class

    Next, create \ ``MessageKeysGen``\  class in the same package as \ ``MessageKeys``\  class and write the logic as follows:

    .. code-block:: java

        package com.example.common.message;

        import java.io.BufferedReader;
        import java.io.File;
        import java.io.FileInputStream;
        import java.io.IOException;
        import java.io.InputStream;
        import java.io.InputStreamReader;
        import java.io.PrintWriter;
        import java.util.regex.Pattern;

        import org.apache.commons.io.FileUtils;
        import org.apache.commons.io.IOUtils;

        public class MessageKeysGen {
            public static void main(String[] args) throws IOException {
                // message properties file
                InputStream inputStream = new FileInputStream("src/main/resources/i18n/application-messages.properties");
                BufferedReader br = new BufferedReader(new InputStreamReader(inputStream));
                Class<?> targetClazz = MessageKeys.class;
                File output = new File("src/main/java/"
                        + targetClazz.getName().replaceAll(Pattern.quote("."), "/")
                        + ".java");
                System.out.println("write " + output.getAbsolutePath());
                PrintWriter pw = new PrintWriter(FileUtils.openOutputStream(output));

                try {
                    pw.println("package " + targetClazz.getPackage().getName() + ";");
                    pw.println("/**");
                    pw.println(" * Message Id");
                    pw.println(" */");
                    pw.println("public class " + targetClazz.getSimpleName() + " {");

                    String line;
                    while ((line = br.readLine()) != null) {
                        String[] vals = line.split("=", 2);
                        if (vals.length > 1) {
                            String key = vals[0].trim();
                            String value = vals[1].trim();
                            pw.println("    /** " + key + "=" + value + " */");
                            pw.println("    public static final String "
                                    + key.toUpperCase().replaceAll(Pattern.quote("."),
                                            "_").replaceAll(Pattern.quote("-"), "_")
                                    + " = \"" + key + "\";");
                        }
                    }
                    pw.println("}");
                    pw.flush();
                } finally {
                    IOUtils.closeQuietly(br);
                    IOUtils.closeQuietly(pw);
                }
            }
        }

#. Provision of message properties file

    Define the messages in src/main/resource/i18m/application-messages.properties. The settings are carried out as follows:


    .. code-block:: properties

        i.ex.an.0001={0} upload completed.
        w.ex.an.2001=The recommended change interval has passed password. Please change your password.
        e.ex.an.8001=Cannot upload, Because the file size must be less than {0}MB.
        e.ex.an.9001=There are inconsistencies in the data.

#. Execution of auto-generation class


    .. figure:: ./images_MessageManagement/message-management-messagekeysgen.png
        :width: 60%

    \ ``MessageKeys``\  class is overwritten as follows:


    .. code-block:: java

        package com.example.common.message;
        /**
         * Message Id
         */
        public class MessageKeys {
            /** i.ex.an.0001={0} upload completed. */
            public static final String I_EX_AN_0001 = "i.ex.an.0001";
            /** w.ex.an.2001=The recommended change interval has passed password. Please change your password. */
            public static final String W_EX_AN_2001 = "w.ex.an.2001";
            /** e.ex.an.8001=Cannot upload, Because the file size must be less than {0}MB. */
            public static final String E_EX_AN_8001 = "e.ex.an.8001";
            /** e.ex.an.9001=There are inconsistencies in the data. */
            public static final String E_EX_AN_9001 = "e.ex.an.9001";
        }

\

.. raw:: latex

   \newpage

