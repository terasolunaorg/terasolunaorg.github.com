Input Validation
================================================================================

.. only:: html

 .. contents:: Table of Contents
    :depth: 4

Overview
--------------------------------------------------------------------------------

It is mandatory to check whether the value entered by the user is correct.
Validation of input value is broadly classified into

#. Validation to determine whether the input value is valid by just looking at it irrespective of the context such as size and format.
#. Validation of whether the changes in input value are valid depending on the system status.

1. is an example of mandatory check and number of digits check and 2. is an example of check of whether EMail is registered and
whether order count is within the count of the available stock.

In this section, 1. is explained and this check is called "Input validation".
2. is called "Business logic check". For business logic check,
refer to \ :doc:`../../ImplementationAtEachLayer/DomainLayer`\ .

In this guideline, validation check should be performed in application layer
whereas business logic check should be performed in domain layer.


Input validation of Web application is performed at server side and client side (JavaScript).
It is mandatory to check at the Server Side. However, if the same check is performed at the client side also, 
usability improves since validation results can be analyzed without communicating with the server.

.. warning::

  Input validation should be performed at the server side as the process at client side may be altered using JavaScript.
  If the validation is performed only at the client side without performing at the server side, the system may be exposed to danger.


.. todo::

  Input validation at client side will be explained later. Only input validation at the server side is mentioned in the first version.


Classification of input validation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Input validation is classified into single item check and correlation item check.

.. tabularcolumns:: |p{0.15\linewidth}|p{0.30\linewidth}|p{0.25\linewidth}|p{0.30\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 15 30 25 30


   * - Type
     - Description
     - Example
     - Implementation method
   * - Single item check
     - | Check completed in single field
     - | Input mandatory check
       | Digit check
       | Type check
     - | Bean Validation (Hibernate Validator is used as implementation library)
   * - Correlation item check
     - | Check comparing multiple fields
     - | Password and confirm password check
     - | Bean Validation or Validation class implementing `org.springframework.validation.Validator <http://docs.spring.io/spring/docs/4.3.5.RELEASE/spring-framework-reference/html/validation.html#validator>`_\
       | interface


Spring supports Bean Validation which is a Java standard.
This Bean Validation is used for single item check.
Bean Validation or \ ``org.springframework.validation.Validator``\  interface provided by Spring is used for correlation item check.



.. _Validation_how_to_use:

How to use
--------------------------------------------------------------------------------
.. ValidationAddDependencyLibrary:

Adding dependent libraries
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
When Bean validation 1.1 (Hibernate Validator 5.x) or higher version is to be used,
in addition to jar file of Hibernate Validator and jar file storing API specifications class (\ ``javax.validation``\  package class) of Bean Validation,
libraries that store the following classes are required.

* API specifications class of Expression Language 2.2 or higher version (\ ``javax.el``\  package class)
* Reference implementation class of Expression Language 2.2 or higher version


If the file is run by deploying on application server,
dependent libraries need not be added; 
since these libraries are provided by application server.
However, when it is run in standalone environment (JUnit etc.), these libraries need to be added as dependent libraries.

An example of adding libraries which are required when running Bean Validation 1.1 or higher version in standalone environment is given below.

.. code-block:: xml

    <!-- (1) -->
    <dependency>
        <groupId>org.apache.tomcat.embed</groupId>
        <artifactId>tomcat-embed-el</artifactId>
        <scope>test</scope> <!-- (2) -->
    </dependency>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Add a library wherein a class for Expression Language is stored,
        in :file:`pom.xml` file of the project to be run in standalone environment.

        In the above example, libraries provided for Apache Tomcat to be embedded are specified.
        API specifications classes of Expression Language and reference implementation classes are both stored in jar file of \ ``tomcat-embed-el``\ .

    * - | (2)
      - When dependent libraries are required to execute JUnit, an appropriate scope is \ ``test``\ .

.. note::

    In the above setting example, since it is assumed that the dependent library version is managed by the parent project terasoluna-gfw-parent , specifying the version in pom.xml is not necessary.
    The above dependent library used by terasoluna-gfw-parent is defined by \ `Spring IO Platform <http://platform.spring.io/platform/>`_\ .


.. _Validation_single_check:

Single item check
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For the implementation of single item check following points are necessary,

* Bean Validation annotation should be assigned to the field of form class
* \ ``@Validated``\  annotation should be assigned in Controller for validation
* Tag for displaying validation error message should be added to JSP

.. note::

  \ ``<mvc:annotation-driven>``\  settings are carried out in spring-mvc.xml, Bean Validation is enabled.


.. _Validation_basic_validation:

Basic single item check
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Implementation method is explained using "New user registration" process as an example. Rules for checking "New user registration" form are provided below.


.. tabularcolumns:: |p{0.20\linewidth}|p{0.30\linewidth}|p{0.50\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 30 50


   * - Field name
     - Type
     - Rules
   * - | name
     - | ``java.lang.String``
     - | Mandatory input
       | Between 1 and
       | 20 characters
   * - | email
     - | ``java.lang.String``
     - | Mandatory input
       | Between 1 and
       | 50 characters
       | Email format
   * - | age
     - | ``java.lang.Integer``
     - | Mandatory input
       | Between 1 and
       | 200

* Form class

  Assign Bean Validation annotation to each field of form class.

  .. code-block:: java

      package com.example.sample.app.validation;

      import java.io.Serializable;

      import javax.validation.constraints.Max;
      import javax.validation.constraints.Min;
      import javax.validation.constraints.NotNull;
      import javax.validation.constraints.Size;

      import org.hibernate.validator.constraints.Email;

      public class UserForm implements Serializable {

        private static final long serialVersionUID = 1L;

        @NotNull // (1)
        @Size(min = 1, max = 20) // (2)
        private String name;

        @NotNull
        @Size(min = 1, max = 50)
        @Email // (3)
        private String email;

        @NotNull // (4)
        @Min(0) // (5)
        @Max(200) // (6)
        private Integer age;

        // omitted setter/getter
      }


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90


     * - Sr. No.
       - Description
     * - | (1)
       - | Assign \ ``javax.validation.constraints.NotNull``\  indicating that the target field is not \ ``null``\ .
         |
         | In Spring MVC, when form is sent with input fields left blank,
         | \ **empty string instead of null binds**\ to form object by default.
         | This \ ``@NotNull``\  checks that \ ``name``\  exists as request parameter.
     * - | (2)
       - | Assign \ ``javax.validation.constraints.Size``\  indicating that the string length (or collection size) of the target field is within the specified size range.
         |
         | Since empty string binds to the field where string is left blank by default in Spring MVC,
         | '1 or more character' rule indicates Mandatory input.
     * - | (3)
       - | Assign \ ``org.hibernate.validator.constraints.Email``\  indicating that the target field is in RFC2822-compliant E-mail format.
         | When E-mail format requirements are flexible than RFC2822-compliant constraints, regular expression should be specified using  \ ``javax.validation.constraints.Pattern``\  instead of \ ``@Email``\ .
     * - | (4)
       - | When form is sent without entering any number in input field, \ ``null`` \ binds to form object so \ ``@NotNull``\  indicates mandatory input of \ ``age``\.
     * - | (5)
       - | Assign \ ``javax.validation.constraints.Min``\  indicating that the target field value must be greater than the specified value.
     * - | (6)
       - | Assign \ ``javax.validation.constraints.Max``\  indicating that the target field value must be less than the specified value.


  .. tip::
  
    Refer to \ :ref:`Validation_jsr303_doc`\  and \ :ref:`Validation_validator_list`\  for standard annotations of Bean Validation and annotations provided by Hibernate.
  
  .. tip::
  
    Refer to \ :ref:`Validation_string_trimmer_editor`\  for the method of binding \ ``null``\  when input field is left blank.

* Controller class

  Assign \ ``@Validated``\  to form class for input validation.

  .. code-block:: java

      package com.example.sample.app.validation;

      import org.springframework.stereotype.Controller;
      import org.springframework.validation.BindingResult;
      import org.springframework.validation.annotation.Validated;
      import org.springframework.web.bind.annotation.ModelAttribute;
      import org.springframework.web.bind.annotation.RequestMapping;
      import org.springframework.web.bind.annotation.RequestMethod;

      @Controller
      @RequestMapping("user")
      public class UserController {

        @ModelAttribute
        public UserForm setupForm() {
          return new UserForm();
        }

        @RequestMapping(value = "create", method = RequestMethod.GET, params = "form")
        public String createForm() {
          return "user/createForm"; // (1)
        }

        @RequestMapping(value = "create", method = RequestMethod.POST, params = "confirm")
        public String createConfirm(@Validated /* (2) */ UserForm form, BindingResult /* (3) */ result) {
          if (result.hasErrors()) { // (4)
            return "user/createForm";
          }
          return "user/createConfirm";
        }

        @RequestMapping(value = "create", method = RequestMethod.POST)
        public String create(@Validated UserForm form, BindingResult result) { // (5)
          if (result.hasErrors()) {
            return "user/createForm";
          }
          // omitted business logic
          return "redirect:/user/create?complete";
        }

        @RequestMapping(value = "create", method = RequestMethod.GET, params = "complete")
        public String createComplete() {
          return "user/createComplete";
        }
      }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | Display "New user registration" form screen.
     * - | (2)
       - | Assign \ ``org.springframework.validation.annotation.Validated``\  to the form class argument to perform input validation.
     * - | (3)
       - | Add \ ``org.springframework.validation.BindingResult``\  that stores check result of input validation performed in step (2).
         | This \ ``BindingResult``\  must be specified immediately after the form argument.
         |
         | When not specified immediately, \ ``org.springframework.validation.BindException``\  is thrown.
     * - | (4)
       - | Use \ ``BindingResult.hasErrors()``\  method to determine the check result of step (2).
         | When the result of  \ ``hasErrors()``\  is \ ``true``\ , return to form display screen as there is an error in input value.
     * - | (5)
       - | \ **Input validation should be executed again**\ even at the time of submission on the confirmation screen.
         | There is a possibility of data tempering and hence Input validation must be performed just before entering business logic.


  .. note::
  
    \ ``@Validated``\  is not a standard Bean Validation annotation. It is an independent annotation provided by Spring.
    Bean Validation standard \ ``javax.validation.Valid``\  annotation can also be used. However, \ ``@Validated``\  is better as compared to \ ``@Valid``\  annotation.
    Validation group can be specified in case of \ ``@Validated``\  and hence \ ``@Validated``\  is recommended in this guideline.


.. _Validation_jsp_impl_sample:

* JSP

 When there is input error, it can be displayed in \ ``<form:errors>``\  tag.

  .. code-block:: jsp

      <!DOCTYPE html>
      <html>
      <%-- WEB-INF/views/user/createForm.jsp --%>
      <body>
          <form:form modelAttribute="userForm" method="post"
              action="${pageContext.request.contextPath}/user/create">
              <form:label path="name">Name:</form:label>
              <form:input path="name" />
              <form:errors path="name" /><%--(1) --%>
              <br>
              <form:label path="email">Email:</form:label>
              <form:input path="email" />
              <form:errors path="email" />
              <br>
              <form:label path="age">Age:</form:label>
              <form:input path="age" />
              <form:errors path="age" />
              <br>
              <form:button name="confirm">Confirm</form:button>
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
       - | Specify target field name in \ ``path``\  attribute of \ ``<form:errors>``\  tag.
         | Error message is displayed next to input field of each field.

Form is displayed as follows.

.. figure:: ./images_Validation/validations-first-sample1.png
  :width: 60%

Error message is displayed as follows if this form is sent with all the input fields left blank.

.. figure:: ./images_Validation/validations-first-sample2.png
  :width: 60%

Error messages state that Name and Email are blank and Age is \ ``null``\ .

.. note::

  In Bean Validation, ``null``\  is a valid input value except the following annotations.

  * ``javax.validation.constraints.NotNull``
  * ``org.hibernate.validator.constraints.NotEmpty``
  * ``org.hibernate.validator.constraints.NotBlank``

  In the above example, error messages related to ``@Min``\  and \ ``@Max``\  annotations are not displayed. This is because \ ``null``\  is a valid value for \ ``@Min``\  and \ ``@Max``\  annotations.

Next, send the form by entering any value in the field.

.. figure:: ./images_Validation/validations-first-sample3.png
  :width: 60%

| Error message is not displayed since input value of Name fulfills validation conditions.
| Error message is displayed since input value of Email is not in Email format though it fulfills the conditions related to string length.
| Error message is displayed since input value of Age exceeds maximum value.


Change the form as follows to change the style at the time of error.

.. code-block:: jsp

    <form:form modelAttribute="userForm" method="post"
        class="form-horizontal"
        action="${pageContext.request.contextPath}/user/create">
        <form:label path="name" cssErrorClass="error-label">Name:</form:label><%-- (1) --%>
        <form:input path="name" cssErrorClass="error-input" /><%-- (2) --%>
        <form:errors path="name" cssClass="error-messages" /><%-- (3) --%>
        <br>
        <form:label path="email" cssErrorClass="error-label">Email:</form:label>
        <form:input path="email" cssErrorClass="error-input" />
        <form:errors path="email" cssClass="error-messages" />
        <br>
        <form:label path="age" cssErrorClass="error-label">Age:</form:label>
        <form:input path="age" cssErrorClass="error-input" />
        <form:errors path="age" cssClass="error-messages" />
        <br>
        <form:button name="confirm">Confirm</form:button>
    </form:form>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify class name for \ ``<label>``\  tag in \ ``cssErrorClass``\  attribute at the time of error.
   * - | (2)
     - | Specify class name for \ ``<input>``\  tag in \ ``cssErrorClass``\  attribute at the time of error.
   * - | (3)
     - | Specify class name for  error messages in \ ``cssClass``\  attribute.

For example, if the following CSS is applied to this JSP, error screen is displayed as follows.

.. code-block:: css

    .form-horizontal input {
        display: block;
        float: left;
    }

    .form-horizontal label {
        display: block;
        float: left;
        text-align: right;
        float: left;
    }

    .form-horizontal br {
        clear: left;
    }

    .error-label {
        color: #b94a48;
    }

    .error-input {
        border-color: #b94a48;
        margin-left: 5px;
    }

    .error-messages {
        color: #b94a48;
        display: block;
        padding-left: 5px;
        overflow-x: auto;
    }




.. figure:: ./images_Validation/validations-has-errors1.png
  :width: 60%


CSS can be customized as per the requirements of screen.


Instead of displaying the error messages next to each input field,
output them collectively.


.. code-block:: jsp

    <form:form modelAttribute="userForm" method="post"
        action="${pageContext.request.contextPath}/user/create">
        <form:errors path="*" element="div" cssClass="error-message-list" /><%-- (1) --%>

        <form:label path="name" cssErrorClass="error-label">Name:</form:label>
        <form:input path="name" cssErrorClass="error-input" />
        <br>
        <form:label path="email" cssErrorClass="error-label">Email:</form:label>
        <form:input path="email" cssErrorClass="error-input" />
        <br>
        <form:label path="age" cssErrorClass="error-label">Age:</form:label>
        <form:input path="age" cssErrorClass="error-input" />
        <br>
        <form:button name="confirm">Confirm</form:button>
    </form:form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | By specifying \ ``*``\  in \ ``path``\  attribute of \ ``<form:errors>``\  in \ ``<form:form>``\  tag,
       | all error messages related to Model specified in \ ``modelAttribute``\  attribute of \ ``<form:form>``\  can be output.
       | Tag name including these error messages can be specified in ``element``\  attribute. By default, it is \ ``span``\ . However,
       | specify \ ``div``\  to output error message list as block element.
       | Specify CSS class in ``cssClass``\  attribute.



An example of error message is shown when the following CSS class is applied.

.. code-block:: css

    .form-horizontal input {
        display: block;
        float: left;
    }

    .form-horizontal label {
        display: block;
        float: left;
        text-align: right;
        float: left;
    }

    .form-horizontal br {
        clear: left;
    }

    .error-label {
        color: #b94a48;
    }

    .error-input {
        border-color: #b94a48;
        margin-left: 5px;
    }

    .error-message-list {
        color: #b94a48;
        padding:5px 10px;
        background-color: #fde9f3;
        border:1px solid #c98186;
        border-radius:5px;
        margin-bottom: 10px;
    }


.. figure:: ./images_Validation/validations-has-errors2.png
  :width: 60%


| By default, field name is not included in error message, hence it is difficult to understand which error message corresponds to which field.
| Therefore, when an error message is to be displayed in a list, it is necessary to define the message such that field name is included in the error message.
| For method of defining error messages, refer to ":ref:`Validation_message_def`".

.. note:: **Points to be noted when displaying error messages in a list**

   Error messages are output in a random order and the output order cannot be controlled by standard function.
   Therefore, when an output order needs to be controlled (to be kept constant), an extended implementation such as sorting the error information, etc. is required.

   In the method of "Displaying error messages in a list",

   * Error message definition in feed unit
   * Extended implementation to control the output order of error messages

   are required. Therefore, the cost is higher as compared to "displaying error messages next to input field".
   **This guideline recommends the method of "displaying error messages next to input field" when there are no constraints due to screen requirements.**

   Further, following method can be considered as extended methods to control output order of error message.
   Creating inherited class of \ ``org.springframework.validation.beanvalidation.LocalValidatorFactoryBean``\  provided by Spring Framework,
   and sorting error information by overriding \ ``processConstraintViolations``\  method, etc.

.. note:: **About @GroupSequence annotation**

   A mechanism of \ `@GroupSequence annotation <http://docs.jboss.org/hibernate/validator/5.2/reference/en-US/html/ch05.html#section-default-group-class>`_\  is provided to control the check sequence;
   however, add a note that this mechanism is not to control the output order of error message as operations given below are performed.

   * When an error occurs, checking for subsequent groups is not executed.
   * If multiple errors (errors in multiple fields) occur in the check of identical groups, then the output order of error messages would be random. 


.. note::


   Use \ ``<spring:nestedPath>``\  tag to display error messages collectively outside the \ ``<form:form>``\  tag.

     .. code-block:: jsp
       :emphasize-lines: 1,4

       <spring:nestedPath path="userForm">
           <form:errors path="*" element="div"
               cssClass="error-message-list" />
       </spring:nestedPath>
       <hr>
       <form:form modelAttribute="userForm" method="post"
           action="${pageContext.request.contextPath}/user/create">
           <form:label path="name" cssErrorClass="error-label">Name:</form:label>
           <form:input path="name" cssErrorClass="error-input" />
           <br>
           <form:label path="email" cssErrorClass="error-label">Email:</form:label>
           <form:input path="email" cssErrorClass="error-input" />
           <br>
           <form:label path="age" cssErrorClass="error-label">Age:</form:label>
           <form:input path="age" cssErrorClass="error-input" />
           <br>
           <form:button name="confirm">Confirm</form:button>
       </form:form>

Date and time format check
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| In case of performing the date and time format check, it is recommend the use of \ ``@DateTimeFormat`` \ annotation offered by Spring rather than the use of Bean Validation mechanism.
| About how to use the \ ``@DateTimeFormat`` \ annotation, refer \ :ref:`ApplicationLayer-DateTimeFormat`\.
| It is also possible to check the date and time format using \ ``@Pattern`` \ annotation of the Bean Validation.
| However, it is necessary to write the date and time format in regular expressions while using \ ``@Pattern`` \ annotation. If you want to check the date and time that does not exist, the description is complicated.
| Therefore \ ``@DateTimeFormat`` \ annotation is more simpler than the \ ``@Pattern`` \ annotation.


| Since \ ``@DateTimeFormat`` \ annotation is one of the type conversion mechanism provided by Spring, instead of the error messages of Bean Validation, the error message of the type mismatch exception (\ ``TypeMismatchException``\ ) has been displayed on screen at the time of input error.
| In order to avoid the actual exception message gets displayed on the screen, it is necessary to configure the error message in the \ **property file** \ which will be displayed at the time of type mismatch is occurred.
| For more detail, refer \ :ref:`Validation_type_mismatch`\ .


Single item check of nested Bean
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
The method to validate nested Bean using Bean Validation is explained below.

"Ordering" process of an EC site is considered as an example. Rules for checking "Order" form are provided below.

.. tabularcolumns:: |p{0.20\linewidth}|p{0.30\linewidth}|p{0.30\linewidth}|p{0.20\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 30 30 20


   * - Field name
     - Type
     - Rules
     - Description
   * - | coupon
     - | ``java.lang.String``
     - | 5 or less characters
       | Single byte alphanumeric characters
     - | Coupon code
   * - | receiverAddress.name
     - | ``java.lang.String``
     - | Mandatory input
       | Between 1 and
       | 50 characters
     - | Receiver name
   * - | receiverAddress.postcode
     - | ``java.lang.String``
     - | Mandatory input
       | Between 1 and
       | 10 characters
     - | Receiver postal code
   * - | receiverAddress.address
     - | ``java.lang.String``
     - | Mandatory input
       | Between 1 and
       | 100 characters
     - | Receiver address
   * - | senderAddress.name
     - | ``java.lang.String``
     - | Mandatory input
       | Between 1 and
       | 50 characters
     - | Sender name
   * - | senderAddress.postcode
     - | ``java.lang.String``
     - | Mandatory input
       | Between 1 and
       | 10 characters
     - | Sender postal code
   * - | senderAddress.address
     - | ``java.lang.String``
     - | Mandatory input
       | Between 1 and
       | 100 characters
     - | Sender address

Use the same form class since \ ``receiverAddress``\  and \ ``senderAddress``\  are objects of the same class.

* Form class

  .. code-block:: java

    package com.example.sample.app.validation;

    import java.io.Serializable;

    import javax.validation.Valid;
    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Pattern;
    import javax.validation.constraints.Size;

    public class OrderForm implements Serializable {
        private static final long serialVersionUID = 1L;

        @Size(max = 5)
        @Pattern(regexp = "[a-zA-Z0-9]*")
        private String coupon;

        @NotNull // (1)
        @Valid // (2)
        private AddressForm receiverAddress;

        @NotNull
        @Valid
        private AddressForm senderAddress;

        // omitted setter/getter
    }


  .. code-block:: java

    package com.example.sample.app.validation;

    import java.io.Serializable;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;

    public class AddressForm implements Serializable {
        private static final long serialVersionUID = 1L;

        @NotNull
        @Size(min = 1, max = 50)
        private String name;

        @NotNull
        @Size(min = 1, max = 10)
        private String postcode;

        @NotNull
        @Size(min = 1, max = 100)
        private String address;

        // omitted setter/getter
    }


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | This indicates that the child form is mandatory.
         | When not set, it will be considered as valid even if \ ``null``\  is set in \ ``receiverAddress``\ .
     * - | (2)
       - | Assign \ ``javax.validation.Valid``\  annotation to enable Bean Validation of the nested Bean.


* Controller class

  It is not different from the Controller described earlier.

  .. code-block:: java

    package com.example.sample.app.validation;

    import org.springframework.stereotype.Controller;
    import org.springframework.validation.BindingResult;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;

    @RequestMapping("order")
    @Controller
    public class OrderController {

        @ModelAttribute
        public OrderForm setupForm() {
            return new OrderForm();
        }

        @RequestMapping(value = "order", method = RequestMethod.GET, params = "form")
        public String orderForm() {
            return "order/orderForm";
        }

        @RequestMapping(value = "order", method = RequestMethod.POST, params = "confirm")
        public String orderConfirm(@Validated OrderForm form, BindingResult result) {
            if (result.hasErrors()) {
                return "order/orderForm";
            }
            return "order/orderConfirm";
        }
    }

* JSP

  .. code-block:: jsp

    <!DOCTYPE html>
    <html>
    <%-- WEB-INF/views/order/orderForm.jsp --%>
    <head>
    <style type="text/css">
      /* omitted (same as previous sample) */
    </style>
    </head>
    <body>
        <form:form modelAttribute="orderForm" method="post"
            class="form-horizontal"
            action="${pageContext.request.contextPath}/order/order">
            <form:label path="coupon" cssErrorClass="error-label">Coupon Code:</form:label>
            <form:input path="coupon" cssErrorClass="error-input" />
            <form:errors path="coupon" cssClass="error-messages" />
            <br>
        <fieldset>
            <legend>Receiver</legend>
            <%-- (1) --%>
            <form:errors path="receiverAddress"
                cssClass="error-messages" />
            <%-- (2) --%>
            <form:label path="receiverAddress.name"
                cssErrorClass="error-label">Name:</form:label>
            <form:input path="receiverAddress.name"
                cssErrorClass="error-input" />
            <form:errors path="receiverAddress.name"
                cssClass="error-messages" />
            <br>
            <form:label path="receiverAddress.postcode"
                cssErrorClass="error-label">Postcode:</form:label>
            <form:input path="receiverAddress.postcode"
                cssErrorClass="error-input" />
            <form:errors path="receiverAddress.postcode"
                cssClass="error-messages" />
            <br>
            <form:label path="receiverAddress.address"
                cssErrorClass="error-label">Address:</form:label>
            <form:input path="receiverAddress.address"
                cssErrorClass="error-input" />
            <form:errors path="receiverAddress.address"
                cssClass="error-messages" />
        </fieldset>
        <br>
        <fieldset>
            <legend>Sender</legend>
            <form:errors path="senderAddress"
                cssClass="error-messages" />
            <form:label path="senderAddress.name"
                cssErrorClass="error-label">Name:</form:label>
            <form:input path="senderAddress.name"
                cssErrorClass="error-input" />
            <form:errors path="senderAddress.name"
                cssClass="error-messages" />
            <br>
            <form:label path="senderAddress.postcode"
                cssErrorClass="error-label">Postcode:</form:label>
            <form:input path="senderAddress.postcode"
                cssErrorClass="error-input" />
            <form:errors path="senderAddress.postcode"
                cssClass="error-messages" />
            <br>
            <form:label path="senderAddress.address"
                cssErrorClass="error-label">Address:</form:label>
            <form:input path="senderAddress.address"
                cssErrorClass="error-input" />
            <form:errors path="senderAddress.address"
                cssClass="error-messages" />
        </fieldset>

            <form:button name="confirm">Confirm</form:button>
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
       - | When \ ``receiverAddress.name``\ , \ ``receiverAddress.postcode``\ , \ ``receiverAddress.address``\  are not sent as
         | request parameters due to invalid operation, \ ``receiverAddress``\  is considered as \ ``null``\  and error message is displayed.
     * - | (2)
       - | Fields of nested bean are specified as \ ``[parent field name].[child field name]``\ .


Form is displayed as follows.

.. figure:: ./images_Validation/validations-nested1.png
  :width: 60%

Error message is displayed as follows if this form is sent with all the input fields left blank.

.. figure:: ./images_Validation/validations-nested2.png
  :width: 60%


Validation of nested bean is enabled for collections also.

Add a field such that up to 3 addresses can be registered in "user registration" form explained at the beginning.

* Add list of \ ``AddressForm``\  as a field in the form class.

  .. code-block:: java
    :emphasize-lines: 32-35

    package com.example.sample.app.validation;

    import java.io.Serializable;
    import java.util.List;

    import javax.validation.Valid;
    import javax.validation.constraints.Max;
    import javax.validation.constraints.Min;
    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;

    import org.hibernate.validator.constraints.Email;

    public class UserForm implements Serializable {

        private static final long serialVersionUID = 1L;

        @NotNull
        @Size(min = 1, max = 20)
        private String name;

        @NotNull
        @Size(min = 1, max = 50)
        @Email
        private String email;

        @NotNull
        @Min(0)
        @Max(200)
        private Integer age;

        @NotNull
        @Size(min = 1, max = 3) // (1)
        @Valid
        private List<AddressForm> addresses;

        // omitted setter/getter
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | It is possible to use \ ``@Size``\  annotation for checking size of collection as well.
* JSP

  .. code-block:: jsp
    :emphasize-lines: 26-58

    <!DOCTYPE html>
    <html>
    <%-- WEB-INF/views/user/createForm.jsp --%>
    <head>
    <style type="text/css">
      /* omitted (same as previous sample) */
    </style>
    </head>
    <body>

        <form:form modelAttribute="userForm" method="post"
            class="form-horizontal"
            action="${pageContext.request.contextPath}/user/create">
            <form:label path="name" cssErrorClass="error-label">Name:</form:label>
            <form:input path="name" cssErrorClass="error-input" />
            <form:errors path="name" cssClass="error-messages" />
            <br>
            <form:label path="email" cssErrorClass="error-label">Email:</form:label>
            <form:input path="email" cssErrorClass="error-input" />
            <form:errors path="email" cssClass="error-messages" />
            <br>
            <form:label path="age" cssErrorClass="error-label">Age:</form:label>
            <form:input path="age" cssErrorClass="error-input" />
            <form:errors path="age" cssClass="error-messages" />
            <br>
            <form:errors path="addresses" cssClass="error-messages" /><%-- (1) --%>
            <c:forEach items="${userForm.addresses}" varStatus="status"><%-- (2) --%>
                <fieldset class="address">
                    <legend>Address${f:h(status.index + 1)}</legend>
                    <form:label path="addresses[${status.index}].name"
                        cssErrorClass="error-label">Name:</form:label><%-- (3) --%>
                    <form:input path="addresses[${status.index}].name"
                        cssErrorClass="error-input" />
                    <form:errors path="addresses[${status.index}].name"
                        cssClass="error-messages" />
                    <br>
                    <form:label path="addresses[${status.index}].postcode"
                        cssErrorClass="error-label">Postcode:</form:label>
                    <form:input path="addresses[${status.index}].postcode"
                        cssErrorClass="error-input" />
                    <form:errors path="addresses[${status.index}].postcode"
                        cssClass="error-messages" />
                    <br>
                    <form:label path="addresses[${status.index}].address"
                        cssErrorClass="error-label">Address:</form:label>
                    <form:input path="addresses[${status.index}].address"
                        cssErrorClass="error-input" />
                    <form:errors path="addresses[${status.index}].address"
                        cssClass="error-messages" />
                    <c:if test="${status.index > 0}">
                        <br>
                        <button class="remove-address-button">Remove</button>
                    </c:if>
                </fieldset>
                <br>
            </c:forEach>
            <button id="add-address-button">Add address</button>
            <br>
            <form:button name="confirm">Confirm</form:button>
        </form:form>
        <script type="text/javascript"
            src="${pageContext.request.contextPath}/resources/vendor/js/jquery-1.10.2.min.js"></script>
        <script type="text/javascript"
            src="${pageContext.request.contextPath}/resources/app/js/AddressesView.js"></script>
    </body>
    </html>


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | Display error message related to \ ``address``\  field.
     * - | (2)
       - | Process the collection of child forms in a loop using \ ``<c:forEach>``\  tag.
     * - | (3)
       - | Inside the loop, Specify the field of child form using \ ``[parent field name][Index].[child field name]``\ .


* Controller class

  .. code-block:: java
    :emphasize-lines: 20-22

    package com.example.sample.app.validation;

    import java.util.ArrayList;
    import java.util.List;

    import org.springframework.stereotype.Controller;
    import org.springframework.validation.BindingResult;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;

    @Controller
    @RequestMapping("user")
    public class UserController {

        @ModelAttribute
        public UserForm setupForm() {
            UserForm form = new UserForm();
            List<AddressForm> addresses = new ArrayList<AddressForm>();
            addresses.add(new AddressForm());
            form.setAddresses(addresses); // (1)
            return form;
        }

        @RequestMapping(value = "create", method = RequestMethod.GET, params = "form")
        public String createForm() {
            return "user/createForm";
        }

        @RequestMapping(value = "create", method = RequestMethod.POST, params = "confirm")
        public String createConfirm(@Validated UserForm form, BindingResult result) {
            if (result.hasErrors()) {
                return "user/createForm";
            }
            return "user/createConfirm";
        }
    }


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | Edit the form object to display a single address form at the time of initial display of "user registration" form.

* JavaScript

  Below is the JavaScript to dynamically add address input field. However, the explanation of this code is omitted as it is not required.

  .. code-block:: javascript

    // webapp/resources/app/js/AddressesView.js

    function AddressesView() {
      this.addressSize = $('fieldset.address').size();
    };

    AddressesView.prototype.addAddress = function() {
      var $address = $('fieldset.address');
      var newHtml = addressTemplate(this.addressSize++);
      $address.last().next().after($(newHtml));
    };

    AddressesView.prototype.removeAddress = function($fieldset) {
      $fieldset.next().remove(); // remove <br>
      $fieldset.remove(); // remove <fieldset>
    };

    function addressTemplate(number) {
      return '\
    <fieldset class="address">\
        <legend>Address' + (number + 1) + '</legend>\
        <label for="addresses' + number + '.name">Name:</label>\
        <input id="addresses' + number + '.name" name="addresses[' + number + '].name" type="text" value=""/><br>\
        <label for="addresses' + number + '.postcode">Postcode:</label>\
        <input id="addresses' + number + '.postcode" name="addresses[' + number + '].postcode" type="text" value=""/><br>\
        <label for="addresses' + number + '.address">Address:</label>\
        <input id="addresses' + number + '.address" name="addresses[' + number + '].address" type="text" value=""/><br>\
        <button class="remove-address-button">Remove</button>\
    </fieldset>\
    <br>\
    ';
    }

    $(function() {
      var addressesView = new AddressesView();

      $('#add-address-button').on('click', function(e) {
        e.preventDefault();
        addressesView.addAddress();
      });

      $(document).on('click', '.remove-address-button', function(e) {
        if (this === e.target) {
          e.preventDefault();
          var $this = $(this); // this button
          var $fieldset = $this.parent(); // fieldset
          addressesView.removeAddress($fieldset);
        }
      });

    });


Form is displayed as follows.

.. figure:: ./images_Validation/validations-nested-collection1.png
  :width: 60%

Add 2 address forms by clicking "Add address" button twice.

.. figure:: ./images_Validation/validations-nested-collection2.png
  :width: 60%

Error message is displayed as follows if this form is sent with all the input fields left blank.

.. figure:: ./images_Validation/validations-nested-collection3.png
  :width: 60%


.. _ValidationGroupValidation:

Grouped validation
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
By creating validation group, input validation rules for a field can be specified for each group.

In the "new user registration" example, add "Must be an adult" rule for the \ ``age``\  field.
Add \ ``country``\  field also as "Adult" rules differ with country.

To specify group in Bean Validation, set any \ ``java.lang.Class``\  object representing a group in \ ``group``\  attribute of the annotation.

Create the following 3 groups (interface) here.

.. tabularcolumns:: |p{0.50\linewidth}|p{0.50\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 50 50

   * - Group
     - Adult condition
   * - \ ``Chinese``\
     - 18 years or more
   * - \ ``Japanese``\
     - 20 years or more
   * - \ ``Singaporean``\
     - 21 years or more


An example of executing validation using these groups is shown here.


* Form class

  .. code-block:: java
    :emphasize-lines: 18-26,38-42

    package com.example.sample.app.validation;

    import java.io.Serializable;
    import java.util.List;

    import javax.validation.Valid;
    import javax.validation.constraints.Max;
    import javax.validation.constraints.Min;
    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;

    import org.hibernate.validator.constraints.Email;

    public class UserForm implements Serializable {

        private static final long serialVersionUID = 1L;

        // (1)
        public static interface Chinese {
        };

        public static interface Japanese {
        };

        public static interface Singaporean {
        };

        @NotNull
        @Size(min = 1, max = 20)
        private String name;

        @NotNull
        @Size(min = 1, max = 50)
        @Email
        private String email;

        @NotNull
        @Min.List({ // (2)
                @Min(value = 18, groups = Chinese.class), // (3)
                @Min(value = 20, groups = Japanese.class),
                @Min(value = 21, groups = Singaporean.class)
                })
        @Max(200)
        private Integer age;

        @NotNull
        @Size(min = 2, max = 2)
        private String country; // (4)

        // omitted setter/getter
    }


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | Define each group as an interface.
     * - | (2)
       - | \ ``@Min.List``\  annotation is used to specify multiple ``@Min``\  rules on a single field.
         | It is same even while using other annotations.
     * - | (3)
       - | Specify corresponding group class in the \ ``group``\  attribute, in order to define rules for each group.
         | When \ ``group``\  attribute is not specified, \ ``javax.validation.groups.Default``\  group is used.
     * - | (4)
       - | Add a field which will be used to determine which group is to be applied.


* JSP

  There are no major changes in JSP.

  .. code-block:: jsp
      :emphasize-lines: 16-22

      <form:form modelAttribute="userForm" method="post"
          class="form-horizontal"
          action="${pageContext.request.contextPath}/user/create">
          <form:label path="name" cssErrorClass="error-label">Name:</form:label>
          <form:input path="name" cssErrorClass="error-input" />
          <form:errors path="name" cssClass="error-messages" />
          <br>
          <form:label path="email" cssErrorClass="error-label">Email:</form:label>
          <form:input path="email" cssErrorClass="error-input" />
          <form:errors path="email" cssClass="error-messages" />
          <br>
          <form:label path="age" cssErrorClass="error-label">Age:</form:label>
          <form:input path="age" cssErrorClass="error-input" />
          <form:errors path="age" cssClass="error-messages" />
          <br>
          <form:label path="country" cssErrorClass="error-label">Country:</form:label>
          <form:select path="country" cssErrorClass="error-input">
              <form:option value="cn">China</form:option>
              <form:option value="jp">Japan</form:option>
              <form:option value="sg">Singapore</form:option>
          </form:select>
          <form:errors path="country" cssClass="error-messages" />
          <br>
          <form:button name="confirm">Confirm</form:button>
      </form:form>

* Controller class

 By giving a group name to \ ``@Validated``\  annotation, the rules defined for that group will be applied.

  .. code-block:: java
      :emphasize-lines: 46-58

      package com.example.sample.app.validation;


      import javax.validation.groups.Default;

      import org.springframework.stereotype.Controller;
      import org.springframework.validation.BindingResult;
      import org.springframework.validation.annotation.Validated;
      import org.springframework.web.bind.annotation.ModelAttribute;
      import org.springframework.web.bind.annotation.RequestMapping;
      import org.springframework.web.bind.annotation.RequestMethod;

      import com.example.sample.app.validation.UserForm.Chinese;
      import com.example.sample.app.validation.UserForm.Japanese;
      import com.example.sample.app.validation.UserForm.Singaporean;

      @Controller
      @RequestMapping("user")
      public class UserController {

          @ModelAttribute
          public UserForm setupForm() {
              UserForm form = new UserForm();
              return form;
          }

          @RequestMapping(value = "create", method = RequestMethod.GET, params = "form")
          public String createForm() {
              return "user/createForm";
          }

          String createConfirm(UserForm form, BindingResult result) {
              if (result.hasErrors()) {
                  return "user/createForm";
              }
              return "user/createConfirm";
          }

          @RequestMapping(value = "create", method = RequestMethod.POST, params = {
                  "confirm",  /* (1) */ "country=cn" })
          public String createConfirmForChinese(@Validated({ /* (2) */ Chinese.class,
                  Default.class }) UserForm form, BindingResult result) {
              return createConfirm(form, result);
          }

          @RequestMapping(value = "create", method = RequestMethod.POST, params = {
                  "confirm", "country=jp" })
          public String createConfirmForJapanese(@Validated({ Japanese.class,
                  Default.class }) UserForm form, BindingResult result) {
              return createConfirm(form, result);
          }

          @RequestMapping(value = "create", method = RequestMethod.POST, params = {
                  "confirm", "country=sg" })
          public String createConfirmForSingaporean(@Validated({ Singaporean.class,
                  Default.class }) UserForm form, BindingResult result) {
              return createConfirm(form, result);
          }
      }


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | The parameter which is used as condition to dividing between the groups must be set to \ ``param``\  attribute.
     * - | (2)
       - | All the annotations, except \ ``@Min``\  of \ ``age``\  field, belongs to \ ``Default``\  group; hence, specifying \ ``Default``\  is mandatory.


In this example, the check result of the combination of each input value is as follows.

.. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|p{0.40\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 20 20 40

   * - \ ``age``\  value
     - \ ``country``\  value
     - Input validation result
     - Error message
   * - | 17
     - | cn
     - | NG
     - | must be greater than or equal to 18
   * - |
     - | jp
     - | NG
     - | must be greater than or equal to 20
   * - |
     - | sg
     - | NG
     - | must be greater than or equal to 21
   * - | 18
     - | cn
     - | OK
     - |
   * - |
     - | jp
     - | NG
     - | must be greater than or equal to 20
   * - |
     - | sg
     - | NG
     - | must be greater than or equal to 21
   * - | 20
     - | cn
     - | OK
     - |
   * - |
     - | jp
     - | OK
     - |
   * - |
     - | sg
     - | NG
     - | must be greater than or equal to 21
   * - | 21
     - | cn
     - | OK
     - |
   * - |
     - | jp
     - | OK
     - |
   * - |
     - | sg
     - | OK
     - |

.. warning::

   Implementation of this Controller is inadequate; there is no handling when \ ``country``\  value is neither "cn", "jp"  nor "sg".
   400 error is returned when unexpected \ ``country``\  value is encountered.

Next, we can think of a condition where the number of countries increase and adult condition of 18 years or more is be set as a default rule.

Rules are as follows.


.. tabularcolumns:: |p{0.50\linewidth}|p{0.50\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 50 50

   * - Group
     - Adult condition
   * - \ ``Japanese``\
     - 20 years or more
   * - \ ``Singaporean``\
     - 21 years or more
   * - Country other than the above-mentioned(\ ``Default``\ )
     - 18 years or more


* Form class

  In order to specify a value to \ ``Default``\  group (18 years or more), all groups should be specified explicitly in other annotations as well.

  .. code-block:: java

    package com.example.sample.app.validation;

    import java.io.Serializable;
    import java.util.List;

    import javax.validation.Valid;
    import javax.validation.constraints.Max;
    import javax.validation.constraints.Min;
    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;
    import javax.validation.groups.Default;

    import org.hibernate.validator.constraints.Email;

    public class UserForm implements Serializable {

        private static final long serialVersionUID = 1L;

        public static interface Japanese {
        };

        public static interface Singaporean {
        };

        @NotNull(groups = { Default.class, Japanese.class, Singaporean.class }) // (1)
        @Size(min = 1, max = 20, groups = { Default.class, Japanese.class,
                Singaporean.class })
        private String name;

        @NotNull(groups = { Default.class, Japanese.class, Singaporean.class })
        @Size(min = 1, max = 50, groups = { Default.class, Japanese.class,
                Singaporean.class })
        @Email(groups = { Default.class, Japanese.class, Singaporean.class })
        private String email;

        @NotNull(groups = { Default.class, Japanese.class, Singaporean.class })
        @Min.List({
                @Min(value = 18, groups = Default.class), // (2)
                @Min(value = 20, groups = Japanese.class),
                @Min(value = 21, groups = Singaporean.class) })
        @Max(200)
        private Integer age;

        @NotNull(groups = { Default.class, Japanese.class, Singaporean.class })
        @Size(min = 2, max = 2, groups = { Default.class, Japanese.class,
                Singaporean.class })
        private String country;

        // omitted setter/getter
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | Set all groups to annotations other than \ ``@Min``\  of \ ``age``\  field as well.
     * - | (2)
       - | Set the rule for \ ``Default``\  group.

* JSP

  No change in JSP

* Controller class

  .. code-block:: java

    package com.example.sample.app.validation;

    import org.springframework.stereotype.Controller;
    import org.springframework.validation.BindingResult;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;

    import com.example.sample.app.validation.UserForm.Japanese;
    import com.example.sample.app.validation.UserForm.Singaporean;

    @Controller
    @RequestMapping("user")
    public class UserController {

        @ModelAttribute
        public UserForm setupForm() {
            UserForm form = new UserForm();
            return form;
        }

        @RequestMapping(value = "create", method = RequestMethod.GET, params = "form")
        public String createForm() {
            return "user/createForm";
        }

        String createConfirm(UserForm form, BindingResult result) {
            if (result.hasErrors()) {
                return "user/createForm";
            }
            return "user/createConfirm";
        }

        @RequestMapping(value = "create", method = RequestMethod.POST, params = { "confirm" })
        public String createConfirmForDefault(@Validated /* (1) */ UserForm form,
                BindingResult result) {
            return createConfirm(form, result);
        }

        @RequestMapping(value = "create", method = RequestMethod.POST, params = {
                "confirm", "country=jp" })
        public String createConfirmForJapanese(
                @Validated(Japanese.class)  /* (2) */ UserForm form, BindingResult result) {
            return createConfirm(form, result);
        }

        @RequestMapping(value = "create", method = RequestMethod.POST, params = {
                "confirm", "country=sg" })
        public String createConfirmForSingaporean(
                @Validated(Singaporean.class) UserForm form, BindingResult result) {
            return createConfirm(form, result);
        }
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | When the field \ ``country``\  does not have a value, the request is mapped to a method in which \ ``Default``\  group is specified in \ ``@Validated``\  annotation.
     * - | (2)
       - | When the field \ ``country``\  has a value, the request is mapped to a method in which \ ``Default``\  group is not included in \ ``@Validated``\  annotation.


Till now, 2 patterns of using grouped validation have been explained.

\ In the previous pattern, ``Default``\  group is used in Controller class and in the later one,  \ ``Default``\  group is used in form class.


.. tabularcolumns:: |p{0.25\linewidth}|p{0.25\linewidth}|p{0.25\linewidth}|p{0.25\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 25 25 25 25

   * - Pattern
     - Advantages
     - Disadvantages
     - Decision points
   * - Using \ ``Default``\  group in Controller class
     - \ ``group``\  attribute need not be set for the rules that need not be grouped.
     - Since all patterns of group should be defined, it is difficult to define when there are many group patterns.
     - Should be used when there are only a limited number of group patterns (New create group, Update group and Delete group)
   * - Using \ ``Default``\  group in form class
     - Since only the groups that do not belong to the default group need to be defined, it can be handled even if there are many patterns.
     - \ ``group``\  attribute should be set for the rules that need not be grouped making the process complicated.
     - Should be used when there are many group patterns and majority of patterns have a common value.

\ **If none of the above decision points are applicable, then using Bean Validation itself might not be a good idea.**\
After reviewing the design, usage of Spring Validator or implementation of validation in business logic should be considered.


.. note::

 In the examples explained so far, the switching of group validation is carried out using request parameter and parameter that can be specified in \ ``@RequestMapping``\  annotation.
 It is not possible to switch between groups, if switching is to be performed based on permissions in authentication object or any information which cannot be handled by \ ``@RequestMapping``\  annotation.

 In such a case, \ ``@Validated``\  annotation must not be used but \ ``org.springframework.validation.SmartValidator``\  must be used. Group validation can be performed inside the handler method of controller.

   .. code-block:: java

     @Controller
     @RequestMapping("user")
     public class UserController {

         @Inject
         SmartValidator smartValidator; // (1)

         // omitted

         @RequestMapping(value = "create", method = RequestMethod.POST, params = "confirm")
         public String createConfirm(/* (2) */ UserForm form, BindingResult result) {
             // (3)
             Class<?> validationGroup = Default.class;
             // logic to determine validation group
             // if (xxx) {
             //     validationGroup = Xxx.class;
             // }
             smartValidator.validate(form, result, validationGroup); // (4)
             if (result.hasErrors()) {
                 return "user/createForm";
             }
             return "user/createConfirm";
         }

     }

   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - Sr. No.
        - Description
      * - | (1)
        - | Inject \ ``SmartValidator``\ . Since \ ``SmartValidator``\  can be used if \ ``<mvc:annotation-driven>``\  setting is carried out so there is no need to define separately.
      * - | (2)
        - | Do not use \ ``@Validated``\  annotation.
      * - | (3)
        - | Determine a validation group.
          | Logic to determine a validation group recommends delegating to Helper class and keeping logic in Controller in simple state.
      * - | (4)
        - | Execute grouped validation using \ ``validate``\  method of \ ``SmartValidator``\ .
          | Multiple groups can be specified in \ ``validate``\  method.

 Since logic should not be written in Controller, if switching is possible using request parameters in \ ``@RequestMapping``\ annotation, \ ``SmartValidator``\  must not be used.


.. _Validation_correlation_check:

Correlation item check
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
For the validation of correlated items,
Spring Validator(\ ``Validator``\  implementing \ ``org.springframework.validation.Validator``\  interface)
or Bean Validation must be used.

Each one of above has been explained below. However, before that, their features and usage have been explained.


.. tabularcolumns:: |p{0.20\linewidth}|p{0.40\linewidth}|p{0.40\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 40 40


   * - Format
     - Features
     - Usage
   * - | Spring Validator
     - | It is easy to create input validation for a particular class.
       | It is inconvenient to use in Controller.
     - | Input validation implementation of unique business requirements depending on specific form
   * - | Bean Validation
     - | Creation of input validation is not as easy as Spring Validator.
       | It is easy to use in Controller.
     - | Common input validation implementation of development project not depending on specific form



Correlation item check implementation using Spring Validator
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Implementation method is explained with the help of "reset password" process as an example.
| Implement the following rules. Following rules are provided in the "reset password" form.

.. tabularcolumns:: |p{0.20\linewidth}|p{0.30\linewidth}|p{0.30\linewidth}|p{0.20\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 30 30 20


   * - Field name
     - Type
     - Rules
     - Description
   * - | password
     - | ``java.lang.String``
     - | Mandatory input
       | 8 or more characters
       | \ **Must be same as confirmPassword**\
     - | Password
   * - | confirmPassword
     - | ``java.lang.String``
     - | Nothing in particular
     - | Confirm password

Check rule "Must be same as confirmPassword" is validation of correlated items as \ ``password``\  field and \ ``confirmPassword``\  field should have the same value.

* Form class

  other than validation of correlated items, implement using Bean Validation annotation.

  .. code-block:: java

    package com.example.sample.app.validation;

    import java.io.Serializable;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;

    public class PasswordResetForm implements Serializable {
        private static final long serialVersionUID = 1L;

        @NotNull
        @Size(min = 8)
        private String password;

        private String confirmPassword;

        // omitted setter/getter
    }

  .. note::

    Password is normally saved in database after hashing it, hence there is no need to check the maximum number of characters.

* Validator class

  Implement validation of correlated items using \ ``org.springframework.validation.Validator``\  interface.

  .. code-block:: java

    package com.example.sample.app.validation;

    import org.springframework.stereotype.Component;
    import org.springframework.validation.Errors;
    import org.springframework.validation.Validator;

    @Component // (1)
    public class PasswordEqualsValidator implements Validator {

        @Override
        public boolean supports(Class<?> clazz) {
            return PasswordResetForm.class.isAssignableFrom(clazz); // (2)
        }

        @Override
        public void validate(Object target, Errors errors) {

            if (errors.hasFieldErrors("password")) { // (3)
                return;
            }

            PasswordResetForm form = (PasswordResetForm) target;
            String password = form.getPassword();
            String confirmPassword = form.getConfirmPassword();

            if (!password.equals(confirmPassword)) { // (4)
                errors.rejectValue(/* (5) */ "password",
                /* (6) */ "PasswordEqualsValidator.passwordResetForm.password",
                /* (7) */ "password and confirm password must be same.");
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
       - | Assign \ ``@Component``\  to make Validator the target of component scan.
     * - | (2)
       - | Decide the argument is check target of this validator or not. Here \ ``PasswordResetForm``\  class is the target to be checked.
     * - | (3)
       - | If an error occurs at the target fields during a single item check, do not perform correlation check in this Validator.
         | If it is necessary to perform the correlation check, this determination logic is not required.
     * - | (4)
       - | Implement check logic.
     * - | (5)
       - | Specify field name where there is error.
     * - | (6)
       - | Specify code name of error message. Here, code is
         | "[validator name].[form attribute name].[property name]"
         | Refer to \ :ref:`Validation_message_in_application_messages`\  for mesaage definition.
     * - | (7)
       - | Set default message to be used when error message does not get resolved using code.

  .. note::

    Spring Validator implementation class should be placed in the same package as the Controller.

* Controller class

  .. code-block:: java

    package com.example.sample.app.validation;

    import javax.inject.Inject;

    import org.springframework.stereotype.Controller;
    import org.springframework.validation.BindingResult;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.WebDataBinder;
    import org.springframework.web.bind.annotation.InitBinder;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;

    @Controller
    @RequestMapping("password")
    public class PasswordResetController {
        @Inject
        PasswordEqualsValidator passwordEqualsValidator; // (1)

        @ModelAttribute
        public PasswordResetForm setupForm() {
            return new PasswordResetForm();
        }

        @InitBinder
        public void initBinder(WebDataBinder binder) {
            binder.addValidators(passwordEqualsValidator); // (2)
        }

        @RequestMapping(value = "reset", method = RequestMethod.GET, params = "form")
        public String resetForm() {
            return "password/resetForm";
        }

        @RequestMapping(value = "reset", method = RequestMethod.POST)
        public String reset(@Validated PasswordResetForm form, BindingResult result) { // (3)
            if (result.hasErrors()) {
                return "password/resetForm";
            }
            return "redirect:/password/reset?complete";
        }

        @RequestMapping(value = "reset", method = RequestMethod.GET, params = "complete")
        public String resetComplete() {
            return "password/resetComplete";
        }
    }


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | Inject Spring Validator to be used.
     * - | (2)
       - | In the method having \ ``@InitBinder``\  annotation, add Validators using \ ``WebDataBinder.addValidators``\  method. 
         | By this, the added Validator is called when validation is executed with the use of \ ``@Validated``\  annotation.
     * - | (3)
       - | Implement input validation as per the process performed so far.

* JSP

  There are no points to mention for JSP.

  .. code-block:: jsp

    <!DOCTYPE html>
    <html>
    <%-- WEB-INF/views/password/resetForm.jsp --%>
    <head>
    <style type="text/css">
    /* omitted */
    </style>
    </head>
    <body>
        <form:form modelAttribute="passwordResetForm" method="post"
            class="form-horizontal"
            action="${pageContext.request.contextPath}/password/reset">
            <form:label path="password" cssErrorClass="error-label">Password:</form:label>
            <form:password path="password" cssErrorClass="error-input" />
            <form:errors path="password" cssClass="error-messages" />
            <br>
            <form:label path="confirmPassword" cssErrorClass="error-label">Password (Confirm):</form:label>
            <form:password path="confirmPassword"
                cssErrorClass="error-input" />
            <form:errors path="confirmPassword" cssClass="error-messages" />
            <br>
            <form:button>Reset</form:button>
        </form:form>
    </body>
    </html>


Error message as shown below is displayed when form is sent by entering different values in \ ``password``\  field and \ ``confirmPassword``\  fields.

.. figure:: ./images_Validation/validations-correlation-check1.png
  :width: 60%


.. note::

  When \ ``<form:password>``\  tag is used, data gets cleared at the time of redisplay.

.. _Validation_how_to_cross-field_validation_for_multi_field_highlight:
.. note::

   Error information can be set for multiple fields for correlation check.
   However, displaying error messages and applying style must always be performed in a set and only one part of the tasks cannot not be performed.

   When you want to apply style to both the fields wherein correlation check error has occurred however you want to display only one error message,
   it can be done by setting a null string in the error message.
   An example is given below wherein style is applied to \ ``password``\  field and \ ``confirmPassword``\  field and error message is displayed only in \ ``password``\  field.

     .. code-block:: java

       package com.example.sample.app.validation;

       import org.springframework.stereotype.Component;
       import org.springframework.validation.Errors;
       import org.springframework.validation.Validator;

       @Component
       public class PasswordEqualsValidator implements Validator {

           @Override
           public boolean supports(Class<?> clazz) {
               return PasswordResetForm.class.isAssignableFrom(clazz);
           }

           @Override
           public void validate(Object target, Errors errors) {

               // omitted
               if (!password.equals(confirmPassword)) {
                   // register a field error for password
                   errors.rejectValue("password",
                          "PasswordEqualsValidator.passwordResetForm.password",
                          "password and confirm password must be same.");

                   // register a field error for confirmPassword
                   errors.rejectValue("confirmPassword", // (1)
                             "PasswordEqualsValidator.passwordResetForm.confirmPassword", // (2)
                             ""); // (3)
               }
           }
       }

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90
        :class: longtable

        * - Sr. No.
          - Description
        * - | (1)
          - | Register error in \ ``confirmPassword``\ field.
        * - | (2)
          - | Specify code name of error message. Specify a null string in the corresponding error message at that time.
            | For message definition, refer \ :ref:`Validation_message_in_application_messages`\.
        * - | (3)
          - | Set a default message to be used when error message could not be resolved in the code.
            | A null string is set in the example above.


.. note::

   When multiple forms are used in a single controller, model name should be specified in \ ``@InitBinder("xxx")``\  in order to limit the target of Validator.

     .. code-block:: java

       @Controller
       @RequestMapping("xxx")
       public class XxxController {
           // omitted
           @ModelAttribute("aaa")
           public AaaForm() {
               return new AaaForm();
           }

           @ModelAttribute("bbb")
           public BbbForm() {
               return new BbbForm();
           }

           @InitBinder("aaa")
           public void initBinderForAaa(WebDataBinder binder) {
               // add validators for AaaForm
               binder.addValidators(aaaValidator);
           }

           @InitBinder("bbb")
           public void initBinderForBbb(WebDataBinder binder) {
               // add validators for BbbForm
               binder.addValidators(bbbValidator);
           }
           // omitted
       }

.. note::

   To change the check contents of correlated items check rules in accordance with a validation group (for example: To implement correlated items check only when specific validation group is specified, etc.), it is better to switch the process within validate method by implementing \ ``org.springframework.validation.SmartValidator``\  interface instead of implementing \ ``org.springframework.validation.Validator``\  interface.

     .. code-block:: java

       package com.example.sample.app.validation;

       import org.apache.commons.lang3.ArrayUtils;
       import org.springframework.stereotype.Component;
       import org.springframework.validation.Errors;
       import org.springframework.validation.SmartValidator;

       @Component
       public class PasswordEqualsValidator implements SmartValidator { // Implements SmartValidator instead of Validator interface

           @Override
           public boolean supports(Class<?> clazz) {
               return PasswordResetForm.class.isAssignableFrom(clazz);
           }

           @Override
           public void validate(Object target, Errors errors) {
               validate(target, errors, new Object[] {});
           }

           @Override
           public void validate(Object target, Errors errors, Object... validationHints) {
               // Check validationHints(groups) and apply validation logic only when 'Update.class' is specified
               if (ArrayUtils.contains(validationHints, Update.class)) {
                   PasswordResetForm form = (PasswordResetForm) target;
                   String password = form.getPassword();
                   String confirmPassword = form.getConfirmPassword();

                   // omitted...
               }
           }
       }

Implementation of input check of correlated items using Bean Validation
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Independent validation rules should be added to implement validation of correlated items using Bean Validation.

It is explained in :ref:`Validation_custom_constraint`\.


.. _Validation_message_def:

Definition of error messages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Method to change error messages of input validation is explained.

Error messages of Bean Validation in Spring MVC are resolved in the following order.

#. | If there is any message which matches with the rule, among the messages defined in \ ``org.springframework.context.MessageSource``\ , then it is to be used as error message (Spring rule).
   | For default rules of Spring, refer to "`JavaDoc of DefaultMessageCodesResolver <http://docs.spring.io/spring/docs/4.3.5.RELEASE/javadoc-api/org/springframework/validation/DefaultMessageCodesResolver.html>`_ of DefaultMessageCodesResolver".
#. If message cannot be found as mentioned in step 1, then error message is acquired from the \ ``message``\  attribute of the annotation. (Bean Validation rule)

  #. When the value of \ ``message``\  attribute is not in "{message key}" format, use that text as error message.
  #. When the value of \ ``message``\  attribute is in "{message key}" format, search messages corresponding to message key from ValidationMessages.properties under classpath.

    #. When message corresponding to message key is defined, use that message
    #. When message corresponding to message key is not defined, use "{message key}" as error message

Basically, it is recommended to define error messages in properties file.

Messages should be defined at following places.

* properties file read by \ ``org.springframework.context.MessageSource``\ 
* \ ValidationMessages.properties under classpath

Considering that the following settings are done in applicationContext.xml, former is called as "application-messages.properties" and latter is called "ValidationMessages.properties".

.. code-block:: xml

    <bean id="messageSource"
        class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basenames">
            <list>
                <value>i18n/application-messages</value>
            </list>
        </property>
    </bean>


.. figure:: ./images_Validation/validations-message-properties-position-image.png
  :width: 40%

.. warning::

    Multiple \ ``ValidationMessages.properties``\  files should not exist directly under class path. 

    If multiple \ ``ValidationMessages.properties``\  files exist directly under class path,
    an appropriate message may not be displayed, as either one file of them is read leaving rest of the files unread.

    * When adopting multi project structure, please take care so that \ ``ValidationMessages.properties``\  file is not placed in multiple projects.
    * When distributing common parts for Bean Validation as jar file, please take care so that \ ``ValidationMessages.properties``\  file is not included in jar file.

    Further, when a project is created from `Blank project <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank>`_ \  of version 1.0.2.RELEASE or higher, 
    \ ``ValidationMessages.properties``\  is stored directly under \ ``xxx-web/src/main/resources``\ .

|


This guideline classifies the definition as follows.

.. tabularcolumns:: |p{0.50\linewidth}|p{0.50\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 50 50


   * - Properties file name
     - Contents to be defined
   * - | ValidationMessages.properties
     - | Default error messages of Bean Validation specified by the system
   * - | application-messages.properties
     - | Error message of Bean Validation to be overwritten separately
       | Error message of input validation implemented in Spring Validator

When ValidationMessages.properties is not provided, \ :ref:`Default messages provided by Hibernate Validator<Validation_default_message_in_hibernate_validator>`\  is used.

By working with \ ``MessageSource``\, you can handle Japanese messages directly without Native to Ascii.
For details,please refer \ :ref:`Validation_without_native2ascii`

.. _Validation_message_in_validationmessages:

Messages to be defined in ValidationMessages.properties
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Define messages for message key specified in \ ``message``\  attribute of Bean Validation annotation of
ValidationMessages.properties under class path (normal src/main/resources).


It is explained below using the following form used at the beginning of \ :ref:`Validation_basic_validation`\.


* Form class (re-displayed)

  .. code-block:: java

    public class UserForm implements Serializable {

        @NotNull
        @Size(min = 1, max = 20)
        private String name;

        @NotNull
        @Size(min = 1, max = 50)
        @Email
        private String email;

        @NotNull
        @Min(0)
        @Max(200)
        private Integer age;

        // omitted getter/setter
    }

* ValidationMessages.properties

  Change error messages of \ ``@NotNull``\ , \ ``@Size``\ , \ ``@Min``\ , \ ``@Max``\ , \ ``@Email``\ .

  .. code-block:: properties

    javax.validation.constraints.NotNull.message=is required.
    # (1)
    javax.validation.constraints.Size.message=size is not in the range {min} through {max}.
    javax.validation.constraints.Min.message=cannot be less than {value}.
    javax.validation.constraints.Max.message=cannot be greater than {value}.
    org.hibernate.validator.constraints.Email.message=is an invalid e-mail address.

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | It is possible to embed the value of attributes specified in the annotation using \ ``{Attribute name}``\ .

When the form is sent with input fields left blank after adding the above settings, changed error messages are displayed as shown below.

.. figure:: ./images_Validation/validations-customize-message1.png
  :width: 60%

.. warning::

  Since \ ``{FQCN of annotation.message}``\  is set in \ ``message``\  attribute in Bean Validation standard annotations and independent Hibernate Validator annotations,
  messages can be defined in properties file in the above format. However, since all annotations may not be in this format, Javadoc or source code of the target annotation should be checked.
    
.. code-block:: properties

      FQCN of annotation.message = Message


Add \ ``{0}``\  to message as shown below when field name is to be included in error message.

* ValidationMessages.properties

 Change error message of \ ``@NotNull``\ , \ ``@Size``\ , \ ``@Min``\ , \ ``@Max``\  and \ ``@Email``\ .

  .. code-block:: properties

    javax.validation.constraints.NotNull.message="{0}" is required.
    javax.validation.constraints.Size.message=The size of "{0}" is not in the range {min} through {max}.
    javax.validation.constraints.Min.message="{0}" cannot be less than {value}.
    javax.validation.constraints.Max.message="{0}" cannot be greater than {value}.
    org.hibernate.validator.constraints.Email.message="{0}" is an invalid e-mail address.

Error message is changed as follows.

.. figure:: ./images_Validation/validations-customize-message2.png
  :width: 60%

In this way, property name of form class gets displayed on the screen and so it is not user friendly.
To display an appropriate field name, it should be defined in \ **application-messages.properties**\  in the following format.

.. code-block:: properties

  form property name=field name to be displayed
  
It is good to define field names in the form.

Adding the same to our example.

* application-messages.properties

  .. code-block:: properties

    name=Name
    email=Email
    age=Age

Error messages are changed as follows.

.. figure:: ./images_Validation/validations-customize-message3.png
  :width: 60%


.. note::

  Inserting field name in place of \ ``{0}``\  is the functionality of Spring and not of Bean Validation.
  Therefore, the settings for changing field name should be defined in application-messages.properties(\ ``ResourceBundleMessageSource``\ ) which is directly under Spring management.

.. tip::

    In Bean Validation 1.1,
    it is possible to use Expression Language (hereafter referred to as "EL expression") in a message specified in :file:`ValidationMessages.properties`.
    Hibernate Validator 5.x supports Expression Language 2.2 or higher version.

    Executable EL expression version differs depending on the version of application server.
    Therefore when EL expression is to be used, **it should be used after confirming the version of EL expression supported by application server.**

    Following is an example of using EL expression in a message which is defined in :file:`ValidationMessages.properties` provided by Hibernate Validator by default.

     .. code-block:: properties

        # ...
        # (1)
        javax.validation.constraints.DecimalMax.message  = must be less than ${inclusive == true ? 'or equal to ' : ''}{value}
        # ...

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90

        * - Sr. No.
          - Description
        * - | (1)
          - An EL expression is a part of "\ ``${inclusive == true ? 'or equal to ' : ''}``\  " in a message.

            From the above mentioned definition of message, 2 patterns of messages are created as given below.

            * must be less than or equal to {value}
            * must be less than {value}

            (A value specified in \ ``value``\  attribute of \ ``@DecimalMax``\  annotation is embedded in \ ``{value}``\  part)

            Former is created when \ ``true``\  is specified (or when not specified) in \ ``inclusive``\  attribute of \ ``@DecimalMax``\  annotation,
            Latter is created when \ ``false``\  is specified in \ ``inclusive``\  attribute of \ ``@DecimalMax``\  annotation.

            For handling of EL expressions in Bean Validation refer to:
            \ `Hibernate Validator Reference Guide(Interpolation with message expressions) <http://docs.jboss.org/hibernate/validator/5.2/reference/en-US/html/ch04.html#section-interpolation-with-message-expressions>`_\ .

    Further, the value to be checked can be included in the error message by using \ ``${validatedValue}``\  in the message which is specified in :file:`ValidationMessages.properties`.

    How to use \ ``${validatedValue}``\  is shown below.

     .. code-block:: properties

        # ...
        # (1)
        javax.validation.constraints.Pattern.message = The value entered "${validatedValue}" is invalid.
        # ...


     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90
        :class: longtable

        * - Sr. No.
          - Description
        * - | (1)

          - Messages that are actually generated from the message definition above are embedded with the values input in the form, in \ ``${validatedValue}``\  part.
            If confidential information is included in the input value, care must be taken not to use \ ``${validatedValue}``\  so that confidential information is not displayed.

            For details, refer \ `Hibernate Validator Reference Guide(Interpolation with message expressions) <http://docs.jboss.org/hibernate/validator/5.2/reference/en-US/html/ch04.html#section-interpolation-with-message-expressions>`_\.


.. _Validation_message_in_application_messages:

Messages to be defined in application-messages.properties
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Default messages to be used in system are defined in ValidationMessages.properties
however, depending on the screen, they may have to be changed from the default value.

In this case, define messages in the following format in application-messages.properties.


.. code-block:: properties

  [annotation name].[form attribute name].[property name] = [target message]


Apply "\ :ref:`Validation_message_in_validationmessages`\ " and override the message for  \ ``email``\  and \ ``age``\  field using the below settings.

* application-messages.properties

  .. code-block:: properties

    # override messages
    # for email field
    Size.userForm.email=The size of "{0}" must be between {2} and {1}.
    # for age field
    NotNull.userForm.age="{0}" is compulsory.
    Min.userForm.age="{0}" must be greater than or equal to {1}.
    Max.userForm.age="{0}" must be less than or equal to {1}.

    # filed names
    name=Name
    email=Email
    age=Age

Value of attributes of the annotation gets inserted after \ ``{1}``\  onwards.
Incidentally, index position of attribute values are alphabetical ordering(ascending order) of attribute names.

For example, index positions of \ ``@Size``\  are as follow:

* \ ``{0}``\  : property name (physical name or logical name)
* \ ``{1}``\  : value of \ ``max``\  attribute
* \ ``{2}``\  : value of \ ``min``\  attribute

For specification details, refer to \ `JavaDoc of SpringValidatorAdapter <http://docs.spring.io/spring/docs/4.3.5.RELEASE/javadoc-api/org/springframework/validation/beanvalidation/SpringValidatorAdapter.html#getArgumentsForConstraint-java.lang.String-java.lang.String-javax.validation.metadata.ConstraintDescriptor->`_\.

Error messages are changed as follows.

.. figure:: ./images_Validation/validations-customize-message4.png
  :width: 60%


.. note::

  \ `There are other formats <http://docs.spring.io/spring/docs/4.3.5.RELEASE/javadoc-api/org/springframework/validation/DefaultMessageCodesResolver.html>`_\  as well for the message key format of application-messages.properties;
  however, if it is used with the purpose of overwriting some default messages, it should be in \ ``[annotation name].[form attribute name].[property name]``\  format.

|

.. _Validation_custom_constraint:

How to extend
--------------------------------------------------------------------------------

Other than standard check rules, bean validation has a mechanism to develop annotations for independent rules .

The method of creating independent rules can be widely classified into the following two broader criteria.

* Combination of existing rules
* Creation of new rules

Basically, the below template can be used to create annotation for each rule.

.. code-block:: java

  package com.example.common.validation;

  import java.lang.annotation.Documented;
  import java.lang.annotation.Retention;
  import java.lang.annotation.Target;
  import javax.validation.Constraint;
  import javax.validation.Payload;
  import static java.lang.annotation.ElementType.ANNOTATION_TYPE;
  import static java.lang.annotation.ElementType.CONSTRUCTOR;
  import static java.lang.annotation.ElementType.FIELD;
  import static java.lang.annotation.ElementType.METHOD;
  import static java.lang.annotation.ElementType.PARAMETER;
  import static java.lang.annotation.RetentionPolicy.RUNTIME;

  @Documented
  @Constraint(validatedBy = {})
  @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
  @Retention(RUNTIME)
  public @interface Xxx {
      String message() default "{com.example.common.validation.Xxx.message}";

      Class<?>[] groups() default {};

      Class<? extends Payload>[] payload() default {};

      @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
      @Retention(RUNTIME)
      @Documented
      public @interface List {
          Xxx[] value();
      }
  }


.. _Validation_convine_existing_constraint:

Creation of Bean Validation annotation by combining existing rules
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Consider the following restrictions at the system level and domain level respectively.

| At the system level, 

* String must be single byte alphanumeric characters
* Numbers must be positive

| Or at the domain level,

* "User ID" must be between 4 and 20 single byte characters
* "Age" must be between 1 year and 150 years

| These can be implemented by combining \ ``@Pattern``\ , \ ``@Size``\ , \ ``@Min``\ , \ ``@Max``\  of the existing rules.
| However, if the same rules are to be used at multiple places, settings get distributed and maintainability worsens.

One rule can be created by combining multiple rules.
There is an advantage to be able to have not only common regular expression pattern and maximum/minimum values but also error message when an independent annotation is created.
By this, reusability and maintainability increases. Even if multiple rules are not combined, it also proves beneficial if used only to give specific value to an attribute.

Implementation example is shown below.

* Implementation example of \ ``@Alphanumeric``\  annotation which is restricted to single byte alphanumeric characters

  .. code-block:: java
    :emphasize-lines: 22-23,25

    package com.example.common.validation;

    import java.lang.annotation.Documented;
    import java.lang.annotation.Retention;
    import java.lang.annotation.Target;
    import javax.validation.Constraint;
    import javax.validation.Payload;
    import javax.validation.ReportAsSingleViolation;
    import javax.validation.constraints.Pattern;

    import static java.lang.annotation.ElementType.ANNOTATION_TYPE;
    import static java.lang.annotation.ElementType.CONSTRUCTOR;
    import static java.lang.annotation.ElementType.FIELD;
    import static java.lang.annotation.ElementType.METHOD;
    import static java.lang.annotation.ElementType.PARAMETER;
    import static java.lang.annotation.RetentionPolicy.RUNTIME;

    @Documented
    @Constraint(validatedBy = {})// (1)
    @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
    @Retention(RUNTIME)
    @ReportAsSingleViolation // (2)
    @Pattern(regexp = "[a-zA-Z0-9]*") // (3)
    public @interface AlphaNumeric {
        String message() default "{com.example.common.validation.AlphaNumeric.message}"; // (4)

        Class<?>[] groups() default {};

        Class<? extends Payload>[] payload() default {};

        @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
        @Retention(RUNTIME)
        @Documented
        public @interface List {
            AlphaNumeric[] value();
        }
    }


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | When implementing using existing annotations, \ ``validatedBy``\ needs to be empty
     * - | (2)
       - | This will consolidate error messages and return only the message of this annotation at the time of error.
     * - | (3)
       - | Define rules used by this annotation.
     * - | (4)
       - | Define default value of error message.

* Implementation example of \ ``@NotNegative``\  annotation which is restricted to positive number

  .. code-block:: java
    :emphasize-lines: 22-23,25

    package com.example.common.validation;

    import java.lang.annotation.Documented;
    import java.lang.annotation.Retention;
    import java.lang.annotation.Target;
    import javax.validation.Constraint;
    import javax.validation.Payload;
    import javax.validation.ReportAsSingleViolation;
    import javax.validation.constraints.Min;

    import static java.lang.annotation.ElementType.ANNOTATION_TYPE;
    import static java.lang.annotation.ElementType.CONSTRUCTOR;
    import static java.lang.annotation.ElementType.FIELD;
    import static java.lang.annotation.ElementType.METHOD;
    import static java.lang.annotation.ElementType.PARAMETER;
    import static java.lang.annotation.RetentionPolicy.RUNTIME;

    @Documented
    @Constraint(validatedBy = {})
    @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
    @Retention(RUNTIME)
    @ReportAsSingleViolation
    @Min(value = 0)
    public @interface NotNegative {
        String message() default "{com.example.common.validation.NotNegative.message}";

        Class<?>[] groups() default {};

        Class<? extends Payload>[] payload() default {};

        @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
        @Retention(RUNTIME)
        @Documented
        public @interface List {
            NotNegative[] value();
        }
    }


* Implementation example of \ ``@UserId``\  annotation which regulates the format of "User ID".

  .. code-block:: java
    :emphasize-lines: 23-25,27

    package com.example.sample.domain.validation;

    import java.lang.annotation.Documented;
    import java.lang.annotation.Retention;
    import java.lang.annotation.Target;
    import javax.validation.Constraint;
    import javax.validation.Payload;
    import javax.validation.ReportAsSingleViolation;
    import javax.validation.constraints.Pattern;
    import javax.validation.constraints.Size;

    import static java.lang.annotation.ElementType.ANNOTATION_TYPE;
    import static java.lang.annotation.ElementType.CONSTRUCTOR;
    import static java.lang.annotation.ElementType.FIELD;
    import static java.lang.annotation.ElementType.METHOD;
    import static java.lang.annotation.ElementType.PARAMETER;
    import static java.lang.annotation.RetentionPolicy.RUNTIME;

    @Documented
    @Constraint(validatedBy = {})
    @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
    @Retention(RUNTIME)
    @ReportAsSingleViolation
    @Size(min = 4, max = 20)
    @Pattern(regexp = "[a-z]*")
    public @interface UserId {
        String message() default "{com.example.sample.domain.validation.UserId.message}";

        Class<?>[] groups() default {};

        Class<? extends Payload>[] payload() default {};

        @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
        @Retention(RUNTIME)
        @Documented
        public @interface List {
            UserId[] value();
        }
    }

* Implementation example of \ ``@Age``\  annotation which regulates the constraints on "Age"

  .. code-block:: java
    :emphasize-lines: 23-25,27

    package com.example.sample.domain.validation;

    import java.lang.annotation.Documented;
    import java.lang.annotation.Retention;
    import java.lang.annotation.Target;
    import javax.validation.Constraint;
    import javax.validation.Payload;
    import javax.validation.ReportAsSingleViolation;
    import javax.validation.constraints.Max;
    import javax.validation.constraints.Min;

    import static java.lang.annotation.ElementType.ANNOTATION_TYPE;
    import static java.lang.annotation.ElementType.CONSTRUCTOR;
    import static java.lang.annotation.ElementType.FIELD;
    import static java.lang.annotation.ElementType.METHOD;
    import static java.lang.annotation.ElementType.PARAMETER;
    import static java.lang.annotation.RetentionPolicy.RUNTIME;

    @Documented
    @Constraint(validatedBy = {})
    @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
    @Retention(RUNTIME)
    @ReportAsSingleViolation
    @Min(1)
    @Max(150)
    public @interface Age {
        String message() default "{com.example.sample.domain.validation.Age.message}";

        Class<?>[] groups() default {};

        Class<? extends Payload>[] payload() default {};

        @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
        @Retention(RUNTIME)
        @Documented
        public @interface List {
            Age[] value();
        }
    }


  .. note::

    If multiple rules are set in a single annotation, their AND condition forms the composite annotation.
    In Hibernate Validator, \ ``@ConstraintComposition``\  annotation is provided to implement OR condition.
    Refer to \ `Hibernate Validator document <http://docs.jboss.org/hibernate/validator/5.2/reference/en-US/html/ch11.html#section-boolean-constraint-composition>`_\  for details.
    
    |
    
.. _Validation_for_parameter_object_in_collection_corresponding_annotation:

How to check the values in the collection using Bean Validation's annotation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When dealing with multiple selectable screen items (such as checkboxes and multiple selection dropdown), it is common to treat screen items as collections of basic types such as \ ``String``\ in form classes.
In the standard specification of Bean Validation, each value in the collection can not be checked by using Bean Validation annotation, but by using original features of Java SE 8 and Hibernate Validator, or if original functions of Java SE 8 and Hibernate Validator are not used, a wrapper class for the value of the screen item can be created and it can be treated as a collection so that the value in the collection using Bean Validation can be checked.


Here, annotations to check whether the input value provided by the common library is a code value defined in the code list,
input checking for Strings in the collection using \ ``org.terasoluna.gfw.common.codelist.ExistInCodeList``\ is explained as an example.

Implementation method for matching \ ``@ExistInCodeList``\ annotation with multiple selectable screen items (check boxes, multiple selection drop down, etc.) is shown below.

* :ref:`Validation_exist_in_codelist_javase8`\
    Method of implementing original annotations that can be added to \ ``List``\ of \ ``String``\ by using the original functions of Java SE 8 and Hibernate Validator.
    **It is easier and simpler to implement than the latter, so we recommend this method in environments where the original features of Java SE 8 and Hibernate Validator can be used.** In addition, this method will be standardized in the future version of Bean Validation in future.
    

* :ref:`Validation_exist_in_codelist_formatter`\
    Method of setting the \ ``@ExistInCodeList``\ annotation for the property wrapped in the Java bean class.
    For users who do not use Java SE 8 and Hibernate Validator's original features. In environments where the original features of Java SE 8 and Hibernate Validator can be used, :ref:`Validation_exist_in_codelist_javase8`\ is recommended.


.. _Validation_exist_in_codelist_javase8:

Implementation with Java SE 8 and Hibernate Validator 5.2+
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Here,by creating original annotation \ ``@ExistInCodeList``\ provided by the common library is wrapped,
and how to make it correspond to collection is shown.

\ ``java.lang.annotation.ElementType.TYPE_USE``\ has been added in Java SE 8.
This makes it possible to add annotations to not only declarations of conventional classes and methods, but also general types (types of local variables, etc.),
Hibernate Validator 5.2+ for Java SE 8 supports constraint annotations attached to parameterized types such as \ ``Collection``\ , \ ``Map``\ , \ ``Optional``\.By reading it, it is possible to check the values in the collection.

By combining Java SE 8 with Hibernate Validator 5.2+, like \ ``List<@NotNullForTypeArgument String>``\,
Annotations that can be added to the type specification part of the list can be created, and the values in the collection can be checked.
For details,please refer Hibernate Validator document(\ `Type argument constraints <http://docs.jboss.org/hibernate/validator/5.2/reference/en-US/html_single/#type-arguments-constraints>`_\ ).

\ ``@ExistInCodeList``\ provided by the common library does not correspond to \ ``TYPE_USE``\ because it is Java SE 7 compatible,
By creating original annotation that can be added to the type specifying part of the list as described above, it is possible to check the value in the collection.

The main steps are as follows.

* Use "TYPE_USE" and implement annotations extending \ ``@ExistInCodeList``\ which can be added to the type usage place.

* Set the annotation to be checked.

Input check for Role (\ ``List``\ of \ ``String``\) that can set multiple items is used as an example to explain.

Multiple item configurable Role

Implementation example of \ ``@ExistInCodeListForTypeArgument``\ which can be added to type usage.

  .. code-block:: java

    package com.example.common.validation;

    import static java.lang.annotation.ElementType.TYPE_USE;
    import static java.lang.annotation.RetentionPolicy.RUNTIME;
    import java.lang.annotation.Documented;
    import java.lang.annotation.Retention;
    import java.lang.annotation.Target;
    import javax.validation.Constraint;
    import javax.validation.OverridesAttribute;
    import javax.validation.Payload;
    import javax.validation.ReportAsSingleViolation;
    import org.terasoluna.gfw.common.codelist.ExistInCodeList;

    @Documented
    @Constraint(validatedBy = {})
    @Target(TYPE_USE) // (1)
    @Retention(RUNTIME)
    @ReportAsSingleViolation
    @ExistInCodeList(codeListId = "") // (2)
    public @interface ExistInCodeListForTypeArgument {
        String message() default "{com.example.common.validation.ExistInCodeListForTypeArgument.message}"; // (3)
        
        @OverridesAttribute(constraint = ExistInCodeList.class, name = "codeListId") // (4)
        String codeListId();

        
        Class<?>[] groups() default {};

        Class<? extends Payload>[] payload() default {};

        @Target(TYPE_USE) // (1)
        @Retention(RUNTIME)
        @Documented
        @interface List {
            ExistInCodeListForTypeArgument[] value();
        }
    }


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr.No.
       - Description
     * - | (1)
       - | Set \ ``TYPE_USE``\, so that this annotation can be added at the type usage place.
     * - | (2)
       - | Define the rules (\ ``@ExistInCodeList``\) used by this annotation.
     * - | (3)
       - | Define the default value of the error message. Also define any error message in ValidationMessages.properties.
     * - | (4)
       - | Override the \ ``codeListId``\  attribute of the \ ``@ExistInCodeList``\ annotation.


|

* Form class

  .. code-block:: java

    package com.example.sample.app.validation;

    import java.util.List;
    
    import javax.validation.constraints.NotNull;
    
    import com.example.common.validation.ExistInCodeListForTypeArgument;

    public class SampleForm {
        @NotNull
        @Valid // (1)
        private List<@ExistInCodeListForTypeArgument(codeListId = "CL_ROLE") String> roles; // (2)

        public List<String> getRoles() {
            return roles;
        }

        public void setRoles(List<String> roles) {
            this.roles = roles;
        }
    }


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr.No.
       - Description
     * - | (1)
       - | Give \ ``javax.validation.Valid``\ annotation to enable input checking of element values in \ ``List``\
     * - | (2)
       - | Set the \ ``@ExistInCodeListForTypeArgument``\ annotation for the type specification part of the collection to be checked for input.
         | Specify the code list as the check source in the \ ``codeListId``\  parameter of the annotation.


|

* JSP

  .. code-block:: jsp

    <form:form modelAttribute="sampleForm">
        <!-- (1) -->
        <form:checkboxes path="roles" items="${CL_ROLE}"/>
        <form:errors path="roles*"/>
        <form:button>Submit</form:button>
    </form:form>


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr.No.
       - Description
     * - | (1)
       - |  Implement \ ``<form:checkboxes>``\.


|


.. _Validation_exist_in_codelist_formatter:

Implementation of wrapper class of String using Java bean
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The implementation introduced here is for users who do not use Java SE 8 and Hibernate Validator's original functions. In environments where the original features of Java SE 8 and Hibernate Validator can be used, :ref:`Validation_exist_in_codelist_javase8`\ is recommended.

Since you can not use Bean Validation annotation for elements in the collection as described above when Java SE 8 and Hibernate Validator's original functions are not used,
\ ``String``\ is wrapped with a Java bean and input check is done by appending \ ``@ExistInCodeList``\ to the property of the nested bean.

When using a Spring-supplied tag library for a wrapper class, it is necessary to perform type conversion between a character string and a wrapper class in order to bind to a form. This is a `type conversion mechanism provided by Spring (Formatter) <http://docs.spring.io/spring/docs/4.3.5.RELEASE/spring-framework-reference/htmlsingle/#format>` _
is used for implementation.

By adding a type conversion from \ ``String``\ to \ ``Role``\, \ ``Role``\ to \ ``String``\ in Formatter,as with \ ``List<String>``\,
it can be implemented using  \ ``<form:checkboxes>``\ without complicated implementation.

The main steps are as follows.

* Set the \ ``@ExistInCodeList``\ annotation for checking.

* Create a class that implements the \ ``Formatter``\ class which is the interface for conversion.

* Using \ ``ConversionServiceFactoryBean``\ register the created \ ``Formatter``\ in Spring.


Also, in order to display items that have been successfully selected with \ ``<form:checkboxes>``\, you need to override the \ ``toString``\ method of the wrapper class in addition to the Formatter implementation .

\ ``<form:checkboxes>``\ displays the elements of the collection specified by the \ ``items``\ attribute as selected items, if the value of the selected item specified by the \ ``path``\ attribute matches with the value of the property, it is displayed as the selected item.
To determine this consistency, Formatter is used when the property is a single item, and in the case of an array or collection, the result of the \ ``toString``\ method of the specified property is used.

In order to display it as a selected item successfully, you need to override the \ ``toString``\ method in a wrapper class and return the wrapped value string as in the example below.

.. note::

    For details, please refer the actual determination method of selected \ ``org.springframework.web.servlet.tags.form.SelectedValueComparator``\ class's `javadoc <https://github.com/spring-projects/spring-framework/blob/v4.3.5.RELEASE/spring-webmvc/src/main/java/org/springframework/web/servlet/tags/form/SelectedValueComparator.java>`__


Input check for \ ``Role``\ (Java Bean's \ ``List``\) that can set multiple items is used as an example for explaination.

|

* Form class

  .. code-block:: java
    
    package com.example.sample.app.validation;

    import java.util.List;

    import javax.validation.Valid;
    import javax.validation.constraints.NotNull;

    import com.example.sample.domain.model.Role;

    public class SampleForm {
        @NotNull
        @Valid // (1)
        private List<Role> roles; // (2)

        public List<Role> getRoles() {
            return roles;
        }

        public void setRoles(List<Role> roles) {
            this.roles = roles;
        }
    }


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr.No.
       - Description
     * - | (1)
       - | \ ``javax.validation.Valid``\ annotation to enable bean validation of nested beans.
     * - | (2)
       - | You cannot add \ ``@ExistInCodeList``\ to \ ``List``\ of  \ ``String``\ but 
           by wrapping \ ``String``\ with a Java bean you can add \ ``@ExistInCodeList``\ to the nested bean's \ ``String``\ property.

|

* JavaBean class

  .. code-block:: java

    package com.example.sample.domain.model

    import org.terasoluna.gfw.common.codelist.ExistInCodeList;

    public class Role {
        @ExistInCodeList(codeListId = "CL_ROLE") // (1)
        private String value;

        public String getValue() {
            return value;
        }

        public void setValue(String value) {
            this.value = value;
        }

        @Override // (2)
        public String toString() {
            return getValue();
        }
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr.No.
       - Description
     * - | (1)
       - | Set the \ ``@ExistInCodeList``\ annotation for the property wrapped in the \ ``Role``\ class to perform input check, and specify the code list to check \ ``codeListId``\ .
     * - | (2)
       - | Override in order to draw the screen properly when multiple selection is made and return the character string of the wrapped value.

|

Implement the Formatter class that performs type conversion and register it in Spring.

As mentioned above, it is necessary to wrap \ ``String``\ with \ ``Role``\ (Java bean) to perform input checking.
Implement type conversion by \ ``Formatter``\ to convert from screen input (\ ``String``\) to wrapped \ ``Role``\ and vice versa.

By adding type conversion, mutual conversion between \ ``String``\ and \ ``Role``\ is done automatically.
On the Controller side, it is treated as \ ``List``\ of \ ``Role``\, on JSP side  \ ``List``\ of \ ``String``\.

* \ ``Formatter``\ class

  Implementation of \ ``Formatter``\ which performs mutual conversion between \ ``String``\ and \ ``Role``\.

  .. code-block:: java

    package com.example.sample.app.validation.formatter;

    import java.text.ParseException;
    import java.util.Locale;

    import org.springframework.format.Formatter;

    import com.example.usermanagement.domain.model.Role;

    public class RoleFormatter implements Formatter<Role> { //(1)

        @Override
        public String print(Role source, Locale locale) {
            return source.getValue();
        }

        @Override
        public Role parse(String source, Locale locale) throws ParseException {
            Role role = new Role();
            role.setValue(source);
            return role;
        }

    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr.No.
       - Description
     * - | (1)
       - | Implement the interface \ ``org.springframework.format.Formatter<T>``\.

|

* Bean definition for applying original \ ``Formatter``\

  .. code-block:: xml

    <!-- (1) -->
    <bean id="conversionService"
        class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="formatters">
            <list>
                <!-- (2) -->
                <bean class="com.example.sample.app.validation.formatter.RoleFormatter" />
            </list>
        </property>
    </bean>


  .. code-block:: xml

    <!-- (3) -->
    <mvc:annotation-driven conversion-service="conversionService">
        <!-- omitted -->
    </mvc:annotation-driven>


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr.No.
       - Description
     * - | (1)
       - | Add bean definition of \ ``FormattingConversionServiceFactoryBean``\.
     * - | (2)
       - | Set the created Formatter class(\ ``RoleFormatter``\).
     * - | (3)
       - |  To use the customized type conversion, set the bean defined in (1) to the \ ``conversion-service``\ attribute of \ ``mvc:annotation-driven``\.

|


* JSP

  .. code-block:: jsp

    <form:form modelAttribute="sampleForm">
        <!-- (1) -->
        <form:checkboxes path="roles" items="${CL_ROLE}"/>
        <form:errors path="roles*"/>
        <form:button>Submit</form:button>
    </form:form>


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr.No.
       - Description
     * - | (1)
       - |  \ ``<form:checkboxes>``\ can be used same as \ ``List<String>``\.


|

.. _Validation_implement_new_constraint:

Creation of Bean Validation annotation by implementing new rules
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Any rule can be created by implementing \ ``javax.validation.ConstraintValidator``\  interface and creating annotation that uses this Validator.

The following 3 ways can be cited as applications.

* Rules that cannot be implemented by combining the existing rules
* check rule for correlated items
* Business logic check

.. _Validation_cannot_expressed_existing:

Rules that cannot be implemented by combining the existing rules
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
For the rules that cannot be implemented by combining \ ``@Pattern``\ , \ ``@Size``\ , \ ``@Min``\ , \ ``@Max``\ , implement \ ``javax.validation.ConstraintValidator``\ .

For example, rules that check ISBN (International Standard Book Number)-13 format are given.

* Annotation

  .. code-block:: java
    :emphasize-lines: 16

    package com.example.common.validation;

    import java.lang.annotation.Documented;
    import java.lang.annotation.Retention;
    import java.lang.annotation.Target;
    import javax.validation.Constraint;
    import javax.validation.Payload;
    import static java.lang.annotation.ElementType.ANNOTATION_TYPE;
    import static java.lang.annotation.ElementType.CONSTRUCTOR;
    import static java.lang.annotation.ElementType.FIELD;
    import static java.lang.annotation.ElementType.METHOD;
    import static java.lang.annotation.ElementType.PARAMETER;
    import static java.lang.annotation.RetentionPolicy.RUNTIME;

    @Documented
    @Constraint(validatedBy = { ISBN13Validator.class }) // (1)
    @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
    @Retention(RUNTIME)
    public @interface ISBN13 {
        String message() default "{com.example.common.validation.ISBN13.message}";

        Class<?>[] groups() default {};

        Class<? extends Payload>[] payload() default {};

        @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
        @Retention(RUNTIME)
        @Documented
        public @interface List {
            ISBN13[] value();
        }
    }


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | Specify the \ ``ConstraintValidator``\  implementation class which will get executed when this annotation is used. Multiple constraints can also be specified.


* Validator

  .. code-block:: java

    package com.example.common.validation;

    import javax.validation.ConstraintValidator;
    import javax.validation.ConstraintValidatorContext;

    public class ISBN13Validator implements ConstraintValidator<ISBN13, String> { // (1)

        @Override
        public void initialize(ISBN13 constraintAnnotation) { // (2)
        }

        @Override
        public boolean isValid(String value, ConstraintValidatorContext context) { // (3)
            if (value == null) {
                return true; // (4)
            }
            return isISBN13Valid(value); // (5)
        }

        // This logic is written in http://en.wikipedia.org/wiki/International_Standard_Book_Number
        static boolean isISBN13Valid(String isbn) {
            if (isbn.length() != 13) {
                return false;
            }
            int check = 0;
            try {
                for (int i = 0; i < 12; i += 2) {
                    check += Integer.parseInt(isbn.substring(i, i + 1));
                }
                for (int i = 1; i < 12; i += 2) {
                    check += Integer.parseInt(isbn.substring(i, i + 1)) * 3;
                }
                check += Integer.parseInt(isbn.substring(12));
            } catch (NumberFormatException e) {
                return false;
            }
            return check % 10 == 0;
        }
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | Specify target annotation and field type in generics parameter.
     * - | (2)
       - | Implement initialization process in \ ``initialize``\  method.
     * - | (3)
       - | Implement input validation in \ ``isValid``\  method.
     * - | (4)
       - | Consider input value as correct in case of \ ``null``\ .
     * - | (5)
       - | Check ISBN-13 format.

.. tip::

  Example of :ref:`fileupload_validator`\  is classified in this category. Further, in common library as well, \ :ref:`@ExistInCodeList <codelist-validate>`\  is implemented in this way.

.. _Validation_correlation_item_check:

Check rules for correlated items
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Check of correlated items, which span over multiple fields, can be done using Bean Validation as explained in  :ref:`Validation_correlation_check`\.
| It is recommended to target generalized rules, in case of deciding to use Bean Validation for check of correlated items.

An example of implementing the rule, "Contents of a field should match with its confirmation field" is given below.

Set constraint of assigning "confirm" as the prefix of confirmation field.

* Annotation

  Define such that annotation for validation of correlated items can be used at class level as well.

  .. code-block:: java
    :emphasize-lines: 14,26

    package com.example.common.validation;

    import java.lang.annotation.Documented;
    import java.lang.annotation.Retention;
    import java.lang.annotation.Target;
    import javax.validation.Constraint;
    import javax.validation.Payload;
    import static java.lang.annotation.ElementType.ANNOTATION_TYPE;
    import static java.lang.annotation.ElementType.TYPE;
    import static java.lang.annotation.RetentionPolicy.RUNTIME;

    @Documented
    @Constraint(validatedBy = { ConfirmValidator.class })
    @Target({ TYPE, ANNOTATION_TYPE }) // (1)
    @Retention(RUNTIME)
    public @interface Confirm {
        String message() default "{com.example.common.validation.Confirm.message}";

        Class<?>[] groups() default {};

        Class<? extends Payload>[] payload() default {};

        /**
         * Field name
         */
        String field(); // (2)

        @Target({ TYPE, ANNOTATION_TYPE })
        @Retention(RUNTIME)
        @Documented
        public @interface List {
            Confirm[] value();
        }
    }


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | Narrow down the target of using this annotation, such that it can be added only to class or annotation.
     * - | (2)
       - | Define parameter to pass to annotation.

* Validator

  .. code-block:: java

    package com.example.common.validation;

    import javax.validation.ConstraintValidator;
    import javax.validation.ConstraintValidatorContext;

    import org.springframework.beans.BeanWrapper;
    import org.springframework.beans.BeanWrapperImpl;
    import org.springframework.util.ObjectUtils;
    import org.springframework.util.StringUtils;

    public class ConfirmValidator implements ConstraintValidator<Confirm, Object> {
        private String field;

        private String confirmField;

        private String message;

        public void initialize(Confirm constraintAnnotation) {
            field = constraintAnnotation.field();
            confirmField = "confirm" + StringUtils.capitalize(field);
            message = constraintAnnotation.message();
        }

        public boolean isValid(Object value, ConstraintValidatorContext context) {
            BeanWrapper beanWrapper = new BeanWrapperImpl(value); // (1)
            Object fieldValue = beanWrapper.getPropertyValue(field); // (2)
            Object confirmFieldValue = beanWrapper.getPropertyValue(confirmField);
            boolean matched = ObjectUtils.nullSafeEquals(fieldValue,
                    confirmFieldValue);
            if (matched) {
                return true;
            } else {
                context.disableDefaultConstraintViolation(); // (3)
                context.buildConstraintViolationWithTemplate(message)
                        .addPropertyNode(field).addConstraintViolation(); // (4)
                return false;
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
       - | Use \ ``org.springframework.beans.BeanWrapper``\  which is very convenient to access JavaBean properties.
     * - | (2)
       - | Acquire property value from the form object through \ ``BeanWrapper``\ .
     * - | (3)
       - | Disable the creation of default \ ``ConstraintViolation``\  object.
     * - | (4)
       - | Create an independent \ ``ConstraintViolation``\  object.
         | Define message to be output in \ ``ConstraintValidatorContext.buildConstraintViolationWithTemplate``\ .
         | Specify field name to output error message in \ ``ConstraintViolationBuilder.addPropertyNode``\ .
         | Refer to \ `JavaDoc of ConstraintValidatorContext <http://docs.oracle.com/javaee/7/api/javax/validation/ConstraintValidatorContext.html>`_\  for details.

 .. tip::

    \ ``ConstraintViolationBuilder.addPropertyNode``\  method has been added from the Bean Validation 1.1.

    \ ``ConstraintViolationBuilder.addNode``\  method had been used in Bean Validation 1.0; however, it is a deprecated API from Bean Validation 1.1.

    For deprecated API of Bean Validation, refer to \ `Bean Validation API Document (Deprecated API) <http://docs.jboss.org/hibernate/beanvalidation/spec/1.1/api/deprecated-list.html>`_\ .

.. note::

   As introduced in correlation item  check using Spring Validator, even in Bean Validation,
   :ref:`Set error information for multiple fields for correlation check<Validation_how_to_cross-field_validation_for_multi_field_highlight>` can be performed.

   An example is shown below wherein styles are applied to \ ``password``\ field and \ ``confirmPassword``\ field in Bean Validation and error message is displayed only for \ ``password``\ field.

     .. code-block:: java

       // omitted
       public class ConfirmValidator implements ConstraintValidator<Confirm, Object> {
           private String field;

           private String confirmField;

           private String message;

           public void initialize(Confirm constraintAnnotation) {
               // omitted
           }

           public boolean isValid(Object value, ConstraintValidatorContext context) {
               // omitted
               if (matched) {
                   return true;
               } else {
                   context.disableDefaultConstraintViolation();

                   //new ConstraintViolation to be generated for field
                   context.buildConstraintViolationWithTemplate(message)
                           .addPropertyNode(field).addConstraintViolation();

                   //new ConstraintViolation to be generated for confirmField
                   context.buildConstraintViolationWithTemplate("") // (1)
                           .addPropertyNode(confirmField).addConstraintViolation();

                   return false;
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
          - | Register error of \ ``confirmPassword``\ field. A null string is set in error message at that time.


Check below for the changes, if the "Reset password" is re-implemented using \ ``@Confirm``\  annotation.


* Form class

  .. code-block:: java

    package com.example.sample.app.validation;

    import java.io.Serializable;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;

    import com.example.common.validation.Confirm;

    @Confirm(field = "password") // (1)
    public class PasswordResetForm implements Serializable {
        private static final long serialVersionUID = 1L;

        @NotNull
        @Size(min = 8)
        private String password;

        private String confirmPassword;

        // omitted getter/setter
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | Assign \ ``@Confirm``\  annotation at class level.
         | By this, form object is passed as an argument to \ ``ConstraintValidator.isValid``\ .

* Controller class

  There is no need to inject Validator and add Validator using \ ``@InitBinder``\ .

  .. code-block:: java

    package com.example.sample.app.validation;

    import org.springframework.stereotype.Controller;
    import org.springframework.validation.BindingResult;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;

    @Controller
    @RequestMapping("password")
    public class PasswordResetController {

        @ModelAttribute
        public PasswordResetForm setupForm() {
            return new PasswordResetForm();
        }

        @RequestMapping(value = "reset", method = RequestMethod.GET, params = "form")
        public String resetForm() {
            return "password/resetForm";
        }

        @RequestMapping(value = "reset", method = RequestMethod.POST)
        public String reset(@Validated PasswordResetForm form, BindingResult result) {
            if (result.hasErrors()) {
                return "password/resetForm";
            }
            return "redirect:/password/reset?complete";
        }

        @RequestMapping(value = "reset", method = RequestMethod.GET, params = "complete")
        public String resetComplete() {
            return "password/resetComplete";
        }
    }

.. _Validation_business_logic_check:

Business logic check
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Business logic check should implement \ :ref:`Implementation in Service of domain layer<service-implementation-label>`\  and store result message in \ ``ResultMessages``\  object.
| Accordingly, it is expected that, \ :ref:`they will be normally displayed at the top of the screen <message-display>`\ .

However, there are cases, where business logic error message (such as, "whether the entered user name is already registered") of target input field is to be displayed next to the field.
In such a case, service class is injected in Validator class and business logic check is executed in \ ``ConstraintValidator.isValid``\ . 

An example of implementing, "whether the entered user name is already registered" in Bean Validation is shown below.


* Service class

  Implementation class (UserServiceImpl) is omitted.

  .. code-block:: java

    package com.example.sample.domain.service.user;

    public interface UserService {

        boolean isUnusedUserId(String userId);

        // omitted other methods
    }

* Annotation

  .. code-block:: java

    package com.example.sample.domain.validation;

    import java.lang.annotation.Documented;
    import java.lang.annotation.Retention;
    import java.lang.annotation.Target;
    import javax.validation.Constraint;
    import javax.validation.Payload;
    import static java.lang.annotation.ElementType.ANNOTATION_TYPE;
    import static java.lang.annotation.ElementType.CONSTRUCTOR;
    import static java.lang.annotation.ElementType.FIELD;
    import static java.lang.annotation.ElementType.METHOD;
    import static java.lang.annotation.ElementType.PARAMETER;
    import static java.lang.annotation.RetentionPolicy.RUNTIME;

    @Documented
    @Constraint(validatedBy = { UnusedUserIdValidator.class })
    @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
    @Retention(RUNTIME)
    public @interface UnusedUserId {
        String message() default "{com.example.sample.domain.validation.UnusedUserId.message}";

        Class<?>[] groups() default {};

        Class<? extends Payload>[] payload() default {};

        @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
        @Retention(RUNTIME)
        @Documented
        public @interface List {
            UnusedUserId[] value();
        }
    }


* Validator class

  .. code-block:: java
    :emphasize-lines: 11,15-16

    package com.example.sample.domain.validation;

    import javax.inject.Inject;
    import javax.validation.ConstraintValidator;
    import javax.validation.ConstraintValidatorContext;

    import org.springframework.stereotype.Component;

    import com.example.sample.domain.service.user.UserService;

    @Component // (1)
    public class UnusedUserIdValidator implements
                                      ConstraintValidator<UnusedUserId, String> {

        @Inject // (2)
        UserService userService;

        @Override
        public void initialize(UnusedUserId constraintAnnotation) {
        }

        @Override
        public boolean isValid(String value, ConstraintValidatorContext context) {
            if (value == null) {
                return true;
            }

            return userService.isUnusedUserId(value); // (3)
        }

    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | Settings to enable component scan of the Validator class.
         | Target package must be included in \ ``<context:component-scan base-package="..." />``\  of Bean definition file.
     * - | (2)
       - | Inject the service class to be called.
     * - | (3)
       - | Return the result of business logic error check. Process should be delegated to service class. Logic must not be written directly in the \ ``isValid``\  method.

|

.. _MethodValidation:

Method Validation 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A method to check validity of actual argument and return value of method using Bean Validation, is described.
In order to explain, the method is called Method Validation in this chapter.
While performing defensive programming etc, method I/O should be checked in the class other than Controller.
If Bean Validation library is used at that time, constraint annotation of Bean Validation used in Controller can be reused.

.. _MethodValidationOnSpringFrameworkHowToUseSettings:

Application settings
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When Method Validation offered by Spring Framework is used,
a bean must be defined for \ ``org.springframework.validation.beanvalidation.MethodValidationPostProcessor``\  class offered by Spring Framework.

Bean definition file which defines \ ``MethodValidationPostProcessor``\  differs depending on where you use the Method Validation.

Here, a setup example is given wherein Method Validation is used in the multi-project environment recommended in this guideline.

Setup for both the projects below must be changed

* Project (\ ``projectName-web``\ ) for application layer
* Project (\ ``projectName-domain``\ ) for domain layer

* :file:`projectName-domain/src/main/resources/META-INF/spring/projectName-domain.xml`

 .. code-block:: xml

    <!-- (1) -->
    <bean id="validator"
          class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean"/>

    <!-- (2) -->
    <bean class="org.springframework.validation.beanvalidation.MethodValidationPostProcessor">
        <property name="validator" ref="validator" />
    </bean>

* :file:`projectName-web/src/main/resources/META-INF/spring/spring-mvc.xml`

 .. code-block:: xml

    <!-- (3) -->
    <mvc:annotation-driven validator="validator">
        <!-- ... -->
    </mvc:annotation-driven>

    <!-- (4) -->
    <bean class="org.springframework.validation.beanvalidation.MethodValidationPostProcessor">
        <property name="validator" ref="validator" />
    </bean>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Define a bean for \ ``LocalValidatorFactoryBean``\ .
    * - | (2)
      - Define a bean for \ ``MethodValidationPostProcessor``\  and ensure that
        Method Validation is executed for a domain layer class method.

        Specify a bean defined in (1), in \ ``validator``\  property.
    * - | (3)
      - Specify a bean defined in (1), in the \ ``validator``\  attribute of \ ``<mvc:annotation-driven>``\  element.

        A \ ``Validator``\  instance is generated that is different from the instance created in (1) in the absence of this setup.
    * - | (4)
      - Define a bean for \ ``MethodValidationPostProcessor``\  and ensure that
        Method Validation is executed for an application layer class method.

        Specify Bean defined in (1), in \ ``validator``\  property.

.. tip::

    \ ``LocalValidatorFactoryBean``\  is a class to generate a wrapper \ ``Validator``\  object
    in order to link \ ``Validator``\  class provided by Bean Validation(Hibernate Validator) and Spring Framework.

    By using the wrapper \ ``Validator``\  generated by this class,
    message management function (\ ``MessageSource``\ ) offered by Spring Framework and DI container can be linked.

.. tip::

    In Spring Framework, Method Validation for calling the method of Bean which is managed by DI container is executed
    by using AOP system.

    \ ``MethodValidationPostProcessor``\  is a class to apply AOP in order to execute Method Validation.

.. note::

    In the example above, an identical \ ``Validator``\  object (instance) is set for \ ``validator``\  property of each Bean, but
    this is not necessarily required.
    However, it is recommended to set an identical object (instance) unless there is a reason to do otherwise.

|

.. _MethodValidationOnSpringFrameworkHowToUseApplyTarget:

How to define for the method for Method Validation target
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When Method Validation is applied to the method,
annotation which indicates inclusion of target method and the constraint annotation of Bean Validation should be specified in class level
and, method and dummy argument respectively.

AOP which executes Method Validation is not applicable for ":ref:`MethodValidationOnSpringFrameworkHowToUseSettings`".
It is necessary to assign \ ``@ org.springframework.validation.annotation.Validated``\  annotation to interface or class in order to apply AOP which executes Method Validation.


Here, a method to specify an annotation for interface is introduced.

.. code-block:: java

    package com.example.domain.service;

    import org.springframework.validation.annotation.Validated;

    @Validated // (1)
    public interface HelloService {
        // ...
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Specify \ ``Validated``\  annotation in the interface acting as a target for Method Validation.

        In the example above, AOP which executes Method Validation is applied to implementation method of \ ``HelloService``\  interface.
        

.. tip::

    By specifying a group interface in \ ``value``\  attribute of \ ``@Validated``\  annotation,
    a validation belonging to a specified group can alone be executed as well.

    Further, the validation groups can be changed for each method by assigning \ ``Validated``\  annotation in method level.
    

    Refer ":ref:`ValidationGroupValidation`" for validation group.

|

Next, a method is described wherein constraint annotation of Bean Validation is specified in the method and dummy argument.
Basically,

Specify constraint annotation of Bean Validation for

* Method arguments
* JavaBean field specified in method argument

and constraint annotation of Bean Validation for

* Return value of method
* JavaBean field returned as return value of method.

A basic specification method is described below.
A method to specify annotation in the interface is introduced in the description hereafter.

First, a method that specifies constraint annotation is described for the method using basic types (primitive type or primitive wrapper type etc) as a signature of the method


.. code-block:: java

    package com.example.domain.service;

    import org.springframework.validation.annotation.Validated;

    import javax.validation.constraints.NotNull;

    @Validated
    public interface HelloService {

        // (2)
        @NotNull
        String hello(@NotNull /* (1) */ String message);

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Specify a constraint annotation of Bean Validation as an argument annotation of method.

        \ ``@NotNull``\  is a constraint that signifies that \ ``message``\  argument does not allow Null value.
        When Null value is specified in the argument, \ ``javax.validation.ConstraintViolationException``\  is thrown.
    * - | (2)
      - Specify constraint annotation of Bean Validatio as a method annotation.

        In the above example, it is shown that the return value is not a Null value,
        when Null value is returned as a return value, \ ``javax.validation.ConstraintViolationException``\  is thrown.

|

Next, a method that specifies constraint annotation of Bean Validation is described  for the method using JavaBean as a signature of method.


Here, a method which species annotation for interface is introduced.

.. note::

    The main point is to specify \ ``@javax.validation.Valid``\  annotation.
    The specification method is described below in detail using sample code.

**Service interface**

.. code-block:: java

    package com.example.domain.service;

    import org.springframework.validation.annotation.Validated;

    import javax.validation.constraints.NotNull;

    @Validated
    public interface HelloService {

        @NotNull // (3)
        @Valid   // (4)
        HelloOutput hello(@NotNull /* (1) */ @Valid /* (2) */ HelloInput input);

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Specify constraint annotation of Bean Validation as an argument annotation.

        It indicates that \ ``input``\  argument (JavaBean) does not allow Null value.
        When Null value is specified in the argument, \ ``javax.validation.ConstraintViolationException``\  is thrown.
    * - | (2)
      - Specify \ ``@javax.validation.Valid``\  annotation as an argument annotation of method.

        By assigning \ ``@Valid``\  annotation, constraint annotation of Bean Validation specified in JavaBean field of argument becomes valid.
        When the constraint specified in JavaBean is not fulfilled, \ ``javax.validation.ConstraintViolationException``\  is thrown.
    * - | (3)
      - Specify constraint annotation of Bean Validation as a method annotation.

        It indicates that JavaBean of return value is not Null value.
        When Null value is returned as a return value, an exception is thrown.
    * - | (4)
      - Specify \ ``@Valid``\  annotation as a method annotation.

        By assigning \ ``@Valid``\  annotation, constraint annotation of Bean Validation specified in JavaBean field of return value becomes valid.
        When constraint specified in JavaBean is not fulfilled, \ ``javax.validation.ConstraintViolationException``\  is thrown.

|

| Implementation sample of JavaBean is introduced below.
| Basically, only constraint annotation of Bean Validation is specified, however care must be taken while performing further nesting of JavaBean in JavaBean.

**JavaBean for Input**

.. code-block:: java

    package com.example.domain.service;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Past;
    import java.util.Date;

    public class HelloInput {

        @NotNull
        @Past
        private Date visitDate;

        @NotNull
        private String visitMessage;

        private String userId;

        // ...

    }

**JavaBean for Output**

.. code-block:: java

    package com.example.domain.service;

    import com.example.domain.model.User;

    import java.util.Date;

    import javax.validation.Valid;
    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Past;

    public class HelloOutput {

        @NotNull
        @Past
        private Date acceptDate;

        @NotNull
        private String acceptMessage;

        @Valid // (5)
        private User user;

        // ...

    }

**JavaBean nested in JavaBean for Output**

.. code-block:: java

    package com.example.domain.model;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Past;
    import java.util.Date;

    public class User {

        @NotNull
        private String userId;

        @NotNull
        private String userName;

        @Past
        private Date dateOfBirth;

        // ...

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (5)
      - When constraint annotation of Bean Validation specified in nested JavaBean is to be enabled,
        specify \ ``@Valid``\  annotation as field annotation.

        By assigning \ ``@Valid``\  annotation, constraint annotation of Bean Validation specified in nested JavaBean field becomes valid.
        When constraint specified in nested JavaBean is not fulfilled, \ ``javax.validation.ConstraintViolationException``\  is thrown.

|

.. _MethodValidationOnSpringFrameworkHowToUseExceptionHandling:

Exception handling at the time of violation of constraint
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When a constraint violation occurs, \ ``javax.validation.ConstraintViolationException``\  is thrown.

When \ ``ConstraintViolationException``\  is thrown, the method generated from stack trace can be identified,  however,
basic violation details cannot be identified.

An exception handling class should be created which outputs log by handling \ ``ConstraintViolationException``\  exception in order to identify violation details.

An example to show how to create an exception handling class is given below.

.. code-block:: java

    package com.example.app;

    import javax.validation.ConstraintViolationException;

    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.web.bind.annotation.ControllerAdvice;
    import org.springframework.web.bind.annotation.ExceptionHandler;

    @ControllerAdvice
    public class ConstraintViolationExceptionHandler {

        private static final Logger log = LoggerFactory.getLogger(ConstraintViolationExceptionHandler.class);

        // (1)
        @ExceptionHandler
        public String handleConstraintViolationException(ConstraintViolationException e){
            // (2)
            if (log.isErrorEnabled()) {
                log.error("ConstraintViolations[\n{}\n]", e.getConstraintViolations());
            }
            return "common/error/systemError";
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Create a \ ``@ExceptionHandler``\  method to handle \ ``ConstraintViolationException``\ .

        Receive \ ``ConstraintViolationException``\  as an argument of method.
    * - | (2)
      - Output violation details  (\ ``Set``\  of \ ``ConstraintViolation``\ ) in a log which are retained by \ ``ConstraintViolationException``\ received by method argument.

.. note::

    Refer to ":ref:`application_layer_controller_advice`" for details of \ ``@ControllerAdvice``\  annotation.

.. warning::

    An error message can occur by using the  \ ``ConstraintViolation#getMessage``\ method, but since the message completion by Spring's function is not done, field names in error message at \ ``{0}``\ cannot be embedded.
    
    Instead, field names can be obtained with the \ ``ConstraintViolation#getPropertyPath``\ method.
    
    For message completion by Spring's function, please see the note of:ref:`Validation_message_in_validationmessages`.
    
    For details about \ ``ConstraintViolation``\,please refer \ `Hibernate Validator reference <http://docs.jboss.org/hibernate/validator/5.2/reference/en-US/html_single/#section-constraint-violation-methods>`_\.

Appendix
--------------------------------------------------------------------------------

Input validation rules provided by Hibernate Validator
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Hibernate Validator provides additional validation annotations, in addition to the annotations defined in Bean Validation.
| Refer to \ `Here <http://docs.jboss.org/hibernate/validator/5.2/reference/en-US/html/ch02.html#section-builtin-constraints>`_\ for the annotation list that can be used for validation.

.. _Validation_jsr303_doc:

Bean Validation check rules
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Bean Validation standard annotations(\ ``javax.validation.*``\ ) are shown below.

Refer to Chapter 7 \ `Bean Validation specification <http://download.oracle.com/otn-pub/jcp/bean_validation-1_1-fr-eval-spec/bean-validation-specification.pdf>`_\  for details.

.. tabularcolumns:: |p{0.15\linewidth}|p{0.30\linewidth}|p{0.30\linewidth}|p{0.25\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 15 30 30 25
   :class: longtable

   * - Annotation
     - Target type
     - Description
     - Usage example
   * - \ ``@NotNull``\
     - Arbitrary
     - Validates that the target field is not \ ``null``\ .
     - .. code-block:: java

            @NotNull
            private String id;

   * - \ ``@Null``\
     - Arbitrary
     - | Validates that the target field is \ ``null``\ .
       | (Example: usage in group validation)
     - .. code-block:: java

            @Null(groups={Update.class})
            private String id;

   * - \ ``@Pattern``\
     - \ ``String``\
     - | Whether the target field matches with the regular expression
       | (In case of Hibernate Validator implementation, it is also possible to use it with arbitrary implementation class of \ ``CharSequence``\  interface)
     - .. code-block:: java

            @Pattern(regexp = "[0-9]+")
            private String tel;

   * - \ ``@Min``\
     - | \ ``BigDecimal``\ , \ ``BigInteger``\ , \ ``byte``\ , \ ``short``\ , \ ``int``\ , \ ``long``\  and wrapper
       | (In case of Hibernate Validator implementation, it is also possible to use it with arbitrary implementation class of \ ``CharSequence``\  interface, \ ``Number``\  inherited class. However, only in cases where string can be converted to a number.)
     - Validate whether the value is greater than the minimum value.
     - Refer \ ``@Max``\
   * - \ ``@Max``\
     - | \ ``BigDecimal``\ , \ ``BigInteger``\ , \ ``byte``\ , \ ``short``\ , \ ``int``\ , \ ``long``\  and wrapper
       | (In case of Hibernate Validator implementation, it is also possible to use it with arbitrary implementation class of \ ``CharSequence``\  interface, \ ``Number``\  inherited class. However, only in cases where string can be converted to a number.)
     - Validate whether the value is less than the maximum value.
     - .. code-block:: java

            @Min(1)
            @Max(100)
            private int quantity;

   * - \ ``@DecimalMin``\
     - \ ``BigDecimal``\ , \ ``BigInteger``\ , \ ``String``\ , \ ``byte``\ , \ ``short``\ , \ ``int``\ , \ ``long``\  and wrapper
       (In case of Hibernate Validator implementation, it is also possible to use it with arbitrary implementation class of \ ``CharSequence``\  interface, \ ``Number``\  inherited class.)
     - | Validate whether the Decimal value is greater than or equal to minimum value.
       | By specifying \ ``inclusive = false``\ , it is possible to change to an operation so as to validate whether the value is greater than the minimum value.
     - Refer \ ``@DecimalMax``\
   * - \ ``@DecimalMax``\
     - \ ``BigDecimal``\ , \ ``BigInteger``\ , \ ``String``\ , \ ``byte``\ , \ ``short``\ , \ ``int``\ , \ ``long``\  and wrapper
       (In case of Hibernate Validator implementation, it is also possible to use it with arbitrary implementation class of \ ``CharSequence``\  interface, \ ``Number``\  inherited class.)
     - | Validate whether the decimal value is less than or equal to the maximum value.
       | By specifying \ ``inclusive = false``\ , it is possible to change to an operation so as to validate whether the value is less than the maximum value.
     - .. code-block:: java

            @DecimalMin("0.0")
            @DecimalMax("99999.99")
            private BigDecimal price;

   * - \ ``@Size``\
     - \ ``String``\ (string length), \ ``Collection``\ (elements size), \ ``Map``\ (elements size), Array(array length)
       (In case of Hibernate Validator implementation, it is also possible to use it with arbitrary implementation class of \ ``CharSequence``\  interface.)
     - | Validate whether elements length or size is between \ ``min``\  and \ ``max``\ .
       | It is possible to omit \ ``min``\  and \ ``max``\ . In that case, \ ``min=0``\  and \ ``max=Integer.MAX_VALUE``\  by default.
     - .. code-block:: java

            @Size(min=4, max=64)
            private String password;

   * - \ ``@Digits``\
     - \ ``BigDecimal``\ , \ ``BigInteger``\ , \ ``String``\ , \ ``byte``\ , \ ``short``\ , \ ``int``\ , \ ``long``\  and wrapper
     - | Check whether the value is a numerical value within the specified range.
       | Specify the maximum number of integer digits permitted in the \ ``integer``\  and the maximum value of decimal digits permitted in \ ``fraction``\ .
     - .. code-block:: java

            @Digits(integer=6, fraction=2)
            private BigDecimal price;

   * - \ ``@AssertTrue``\
     - \ ``boolean``\ ,\ ``Boolean``\
     - Validate that the target field is \ ``true``\ (Example: Whether it agrees to the terms）
     - .. code-block:: java

            @AssertTrue
            private boolean checked;

   * - \ ``@AssertFalse``\
     - \ ``boolean``\ ,\ ``Boolean``\
     - Validate whether the target field is \ ``false``\
     - .. code-block:: java

            @AssertFalse
            private boolean checked;

   * - \ ``@Future``\
     - \ ``Date``\ , \ ``Calendar``\
       (In case of Hibernate Validator implementation, it is also applicable to Joda-Time class)
     - Validate whether it is a future date.
     - .. code-block:: java

            @Future
            private Date eventDate;

   * - \ ``@Past``\
     - \ ``Date``\ , \ ``Calendar``\
       (In case of Hibernate Validator implementation, it is also applicable to Joda-Time class)
     - Validate whether it is a past date.
     - .. code-block:: java

            @Past
            private Date eventDate;

   * - \ ``@Valid``\
     - Optional non-primitive type
     - Validate recursively for related objects.
     - .. code-block:: java

            @Valid
            private List<Employer> employers;

            @Valid
            private Dept dept;

.. raw:: latex

   \newpage

.. tip::

     \ ``inclusive``\  attribute of \ ``@DecimalMin``\  and \ ``@DecimalMax``\  annotation is,
     an attribute added from Bean Validation 1.1.

     By specifying \ ``true``\  (to allow same value of specified threshold) to the default value of \ ``inclusive``\  attribute,
     compatibility with Bean Validation 1.0 is maintained.

.. warning::

     In \ ``@Size``\  annotation, characters represented by char type 2 (32 bits) called as surrogate pair are not considered.

     When a string consisiting of a surrogate pair is excluded from the check, adequate care must be taken since number of characters that are counted are likely to be more than the actual number of characters.

     For length of the string including surrogate pair, refer :ref:`StringProcessingHowToGetSurrogatePairStringLength`.


.. _Validation_validator_list:

Hibernate Validator check rules
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

All the major annotations(\ ``org.hibernate.validator.constraints.*``\ ) of Hibernate Validator are shown below.

Refer to \ `Hibernate Validator specifications <http://docs.jboss.org/hibernate/validator/5.2/reference/en-US/html/ch02.html#validator-defineconstraints-hv-constraints>`_\  for details.

.. tabularcolumns:: |p{0.15\linewidth}|p{0.30\linewidth}|p{0.30\linewidth}|p{0.25\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 15 30 30 25

   * - Annotation
     - Target type
     - Description
     - Usage example
   * - \ ``@CreditCardNumber``\
     - It is applicable to any implementation class of \ ``CharSequence``\  interface
     - | Validate whether the credit card number is valid as per Luhn algorithm. It does not necessarily check whether the credit card number is available.
       | By specifying \ ``ignoreNonDigitCharacters = true``\ , it is possible to validate by ignoring non-numeric characters.
     - .. code-block:: java

            @CreditCardNumber
            private String cardNumber;

   * - \ ``@Email``\
     - It is applicable to any implementation class of \ ``CharSequence``\  interface
     - Validate whether the Email address is complaint with RFC2822.
     - .. code-block:: java

            @Email
            private String email;

   * - \ ``@URL``\
     - It is applicable to any implementation class of \ ``CharSequence``\  interface
     - Verify that it is valid as URL. Character string is validated by using \ ``java.net.URL``\  constructor
       and the protocol that is validated as a URL is dependent on the protocols supported by JVM (\ ``http``\ ,\ ``https``\ ,\ ``file``\ ,\ ``jar``\  etc).
     - .. code-block:: java

            @URL
            private String url;

   * - \ ``@NotBlank``\
     - It is applicable to any implementation class of \ ``CharSequence``\  interface
     - Validate that it is not \ ``null``\ , empty string (\ ``""``\ ) and space only.
     - .. code-block:: java

            @NotBlank
            private String userId;

   * - \ ``@NotEmpty``\
     - It is applicable to \ ``Collection``\ , \ ``Map``\ , Array, and any implementation class of \ ``CharSequence``\  interface
     - | Validate that it is not \ ``null``\  or empty.
       | \ ``@NotEmpty``\  should be used when check is to be done in \ ``@NotNull``\  + \ ``@Min(1)``\  combination.
     - .. code-block:: java

            @NotEmpty
            private String password;

.. tip::

     In \ ``@URL``\ , when the verification is to be performed for validating the protocol not supported by JVM, \ ``org.hibernate.validator.constraintvalidators.RegexpURLValidator``\  provided by Hibernate is used.
     This class is a Validator class corresponding to \ ``@URL``\  annotation and it can be checked whether it is a URL format by a regular expression. It can also verify the validation for the protocols not supported by JVM.

     * If the check rules of \ ``@URL``\  of overall application can be changed, Validator class is changed to \ ``RegexpURLValidator``\  in XML,
       as described in \ `JavaDoc <https://docs.jboss.org/hibernate/validator/5.2/api/org/hibernate/validator/constraints/URL.html>`_\ .
     * When only some of the items are to be validated by using regular expressions and default rules are to be used for \ ``@URL``\ , a new annotation and an implementation class \ ``javax.validation.ConstraintValidator``\  that performs validation similar to \ ``RegexpURLValidator``\  are created
       and the validation by the annotation thus created is applied to the required items.

     These can be applied as per the intended use.

     For details of check rule change by XML, refer to \ `Hibernate reference <https://docs.jboss.org/hibernate/validator/5.2/reference/en-US/html/ch07.html#section-configuration-validation-xml>`_\ 
     and for how to create a new annotation, refer to \ :ref:`Validation_implement_new_constraint`\  respectively..
     
.. _Validation_default_message_in_hibernate_validator:

Default messages provided by Hibernate Validator
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In hibernate-validator-<version>.jar, there is a ValidationMessages.properties file at org/hibernate/validator location, which contains the default messages of all Hibernate provided annotations.

.. code-block:: properties

  javax.validation.constraints.AssertFalse.message = must be false
  javax.validation.constraints.AssertTrue.message  = must be true
  javax.validation.constraints.DecimalMax.message  = must be less than ${inclusive == true ? 'or equal to ' : ''}{value}
  javax.validation.constraints.DecimalMin.message  = must be greater than ${inclusive == true ? 'or equal to ' : ''}{value}
  javax.validation.constraints.Digits.message      = numeric value out of bounds (<{integer} digits>.<{fraction} digits> expected)
  javax.validation.constraints.Future.message      = must be in the future
  javax.validation.constraints.Max.message         = must be less than or equal to {value}
  javax.validation.constraints.Min.message         = must be greater than or equal to {value}
  javax.validation.constraints.NotNull.message     = may not be null
  javax.validation.constraints.Null.message        = must be null
  javax.validation.constraints.Past.message        = must be in the past
  javax.validation.constraints.Pattern.message     = must match "{regexp}"
  javax.validation.constraints.Size.message        = size must be between {min} and {max}

  org.hibernate.validator.constraints.CreditCardNumber.message        = invalid credit card number
  org.hibernate.validator.constraints.EAN.message                     = invalid {type} barcode
  org.hibernate.validator.constraints.Email.message                   = not a well-formed email address
  org.hibernate.validator.constraints.Length.message                  = length must be between {min} and {max}
  org.hibernate.validator.constraints.LuhnCheck.message               = The check digit for ${validatedValue} is invalid, Luhn Modulo 10 checksum failed
  org.hibernate.validator.constraints.Mod10Check.message              = The check digit for ${validatedValue} is invalid, Modulo 10 checksum failed
  org.hibernate.validator.constraints.Mod11Check.message              = The check digit for ${validatedValue} is invalid, Modulo 11 checksum failed
  org.hibernate.validator.constraints.ModCheck.message                = The check digit for ${validatedValue} is invalid, ${modType} checksum failed
  org.hibernate.validator.constraints.NotBlank.message                = may not be empty
  org.hibernate.validator.constraints.NotEmpty.message                = may not be empty
  org.hibernate.validator.constraints.ParametersScriptAssert.message  = script expression "{script}" didn't evaluate to true
  org.hibernate.validator.constraints.Range.message                   = must be between {min} and {max}
  org.hibernate.validator.constraints.SafeHtml.message                = may have unsafe html content
  org.hibernate.validator.constraints.ScriptAssert.message            = script expression "{script}" didn't evaluate to true
  org.hibernate.validator.constraints.URL.message                     = must be a valid URL

  org.hibernate.validator.constraints.br.CNPJ.message                 = invalid Brazilian corporate taxpayer registry number (CNPJ)
  org.hibernate.validator.constraints.br.CPF.message                  = invalid Brazilian individual taxpayer registry number (CPF)
  org.hibernate.validator.constraints.br.TituloEleitoral.message      = invalid Brazilian Voter ID card number


.. _Validation_terasoluna_gfw:

Input check rules provided by a common library
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A common library provides an independent annotation for verification.
Here, how to specify input check rules which use annotation provided by common library is explained.

.. _Validation_terasoluna_gfw_list:

terasoluna-gfw-common check rules
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Annotation provided by \ `terasoluna-gfw-common <https://github.com/terasolunaorg/terasoluna-gfw/tree/master/terasoluna-gfw-common>`_\  (\ ``org.terasoluna.gfw.common.codelist.*``\ ) is shown below.

.. tabularcolumns:: |p{0.15\linewidth}|p{0.30\linewidth}|p{0.30\linewidth}|p{0.25\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 15 30 30 25

    * - Annotation
      - Target type
      - Description
      - Usage example
    * - \ ``@ExistInCodeList``\
      - | \ ``Character``\
        | Implementation class of \ ``CharSequence``\ 
        | (\ ``String``\, \ ``StringBuilder``\ etc)
      - Verify whether value is incorporated in the code list.
      - Refer \ :ref:`@ExistInCodeList <codelist-validate>`\ 


terasoluna-gfw-codepoints check rules
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Annotation (\ ``org.terasoluna.gfw.common.codepoints.*``\ ) offered by \ `terasoluna-gfw-codepoints <https://github.com/terasolunaorg/terasoluna-gfw/tree/master/terasoluna-gfw-codepoints>`_\  is shown below. Further, \ ``terasoluna-gfw-codepoints``\  can be used in 5.1.0.RELEASE and subsequent versions.

.. tabularcolumns:: |p{0.15\linewidth}|p{0.30\linewidth}|p{0.30\linewidth}|p{0.25\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 15 30 30 25

    * - Annotation
      - Target type
      - Description
      - Usage example
    * - \ ``@ConsistOf``\
      - | Implementation class of \ ``CharSequence``\ 
        | (\ ``String``\, \ ``StringBuilder``\  etc)
      - Verify whether all the character strings to be checked are included in the specified code point set.
      - Refer \ :ref:`@ConsistOf <StringProcessingHowToUseCodePointsValidator>`\ 


terasoluna-gfw-validator check rules
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Annotation (\ ``org.terasoluna.gfw.common.validator.constraints.*``\ ) offered by \ `terasoluna-gfw-validator <https://github.com/terasolunaorg/terasoluna-gfw/tree/master/terasoluna-gfw-validator>`_\  is shown below. Further, \ ``terasoluna-gfw-validator``\ can be used in 5.1.0.RELEASE and subsequent versions.

.. tabularcolumns:: |p{0.15\linewidth}|p{0.30\linewidth}|p{0.30\linewidth}|p{0.25\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 15 30 30 25
    :class: longtable

    * - Annotation
      - Target type
      - Description
      - Usage example
    * - \ ``@ByteMin``\
      - | Implementation class of \ ``CharSequence``\ 
        | (\ ``String``\, \ ``StringBuilder``\  etc)
      - | Verify whether byte length value is greater than or equal to minimum value.
        |
        | **[Annotation attributes]**
        | \ ``long value``\  - Specify minimum value of byte length.
        | \ ``String charset``\  - Specify string character set used while encoding the value in byte sequence. Default value is \ ``UTF-8``\ .
      - Refer \ ``@ByteMax``\ 
    * - \ ``@ByteMax``\
      - | Implementation class of \ ``CharSequence``\ 
        | (\ ``String``\, \ ``StringBuilder``\  etc)
      - | Verify whether byte length value is less than or equal to maximum value.
        |
        | **[Annotation attribute]**
        | \ ``long value``\  - Specify maximum value for byte length.
        | \ ``String charset``\  - Specify string character set used while encoding the value in byte sequence. Default value is \ ``UTF-8``\ .
      - .. code-block:: java

             @ByteMin(1)
             @ByteMax(value = 100,
                     charset = "Shift_JIS")
             private String id;

    * - \ ``@Compare``\
      - | Implementation class of \ ``Comparable``\  interface can be applied to any JavaBean with a property
      - | Verify that the magnitude relation of specified property value is correct.
        |
        | **[Annotation attribute]**
        | \ ``String left``\  - Specify the property name to be used as a comparison source in the object. A message is displayed in the property in case of a validation error.
        | \ ``String right``\  - Specify property name to be used as a comparison destination in the object.
        | \ ``org.terasoluna.gfw.common.validator.constraints.Compare.Operator operator``\  - Specify Enum type \ ``Operator``\  value which shows expected magnitude relation. Values that can be specified are as below.

        * \ ``EQUAL``\  : \ ``left = right``\ 
        * \ ``GREATER_THAN``\  : \ ``left > right``\ 
        * \ ``GREATER_THAN_OR_EQUAL``\  : \ ``left >= right``\ 
        * \ ``LESS_THAN``\  : \ ``left < right``\ 
        * \ ``LESS_THAN_OR_EQUAL``\  : \ ``left <= right``\ 

        | \ ``boolean requireBoth``\  - Specify whether values must be entered in the fields specified by both \ ``left``\  attribute and \ ``right``\  attribute (should not be \ ``null``\ ).

        * \ ``true``\  : Considered as a validation error if value is entered in only one of the fields. However, considered as "validation success" when values are not entered in both the fields.
        * \ ``false``\  : Considered as "validation success" when only one of the fields contain a value. (Default)

        | \ ``org.terasoluna.gfw.common.validator.constraints.Compare.Node node``\  - Specify Enum type \ ``Node``\  value which indicates path to output error message. Values that can be specified are as below.

        * \ ``PROPERTY``\  : Output as an error for the field specified in \ ``left``\  attribute. (Default)
        * \ ``ROOT_BEAN``\  : Output as an error of object which has been checked.

      - Check whether the mail address matches with the mail address entered for checking. Implementation is as below when entire form is to be displayed as an error message.

        .. code-block:: java

             @Compare(left = "email",
                     right = "confirmEmail",
                     operator = Compare.Operator.EQUAL,
                     requireBoth = true,
                     node = Compare.Node.ROOT_BEAN)
             public class UserRegisterForm {
                 private String email;
                 private String confirmEmail;
             }

        |

        When period start date and period end date are both entered, check that start date is earlier than the end date. Implementation is as below when error message is displayed for period start date.

        .. code-block:: java

             @Compare(left = "form",
                     right = "to",
                     operator = Compare.Operator.LESS_THAN_OR_EQUAL)
             public class Period {
                 private Date from;
                 private Date to;
             }

.. raw:: latex

   \newpage

.. note:: **For mandatory input during correlated item check**
  
  For unit item check, whether a value is entered in the input field（ should not be \ ``null``\ ) can be checked by using \ ``@NotNull``\  in combination. However, in correlated item check, the check like "if value is entered in one field, value is entered forcefully in another field" cannot be implemented by using \ ``@NotNull``\  alone. Hence, \ ``@Compare``\  provides \ ``requireBoth``\  attribute which controls mandatory input for checking which can then be used for implementing the check whenever required.

  Also, when a value is not entered in the input field, \ ``requireBoth``\  attribute can be used only when \ ``null``\  is bound. If a form is sent in Spring MVC when a value is not entered in input field of string, it must be noted that empty string is bound in the form object by default instead of \ ``null``\ . When a value is not entered in the string field, refer \ :ref:`Validation_string_trimmer_editor`\  to bind \ ``null``\  in form object instead of empty string.
  
  Expected check requirements and configuration example are shown below using "checking whether period start date is earlier than end date" as an example.
  
    .. tabularcolumns:: |p{0.50\linewidth}|p{0.50\linewidth}|
    .. list-table::
        :header-rows: 1
        :widths: 50 50
        :class: longtable

        * - Check requirements
          - Configuration example
        * - \ ``from``\  and \ ``to``\  both are mandatory, compare \ ``from``\  and \ ``to``\ .
          - Assign \ ``@NotNull``\ in \ ``from``\  and \ ``to``\  and use default value （ \ ``false``\ ）in \ ``requireBoth``\  attribute.

            .. code-block:: java

                @Compare(left = "from", right = "to", operator = Compare.Operator.LESS_THAN_OR_EQUAL)
                public class Period {
                  @NotNull
                  LocalDate from;
                  @NotNull
                  LocalDate to;
                }

        * - Only \ ``from``\  is mandatory, however carry out comparison check when a value is entered in \ ``to``\  as well.
          - Assign \ ``@NotNull``\ only in  \ ``from``\  and use default value （ \ ``false``\ ）in \ ``requireBoth``\  attribute.

            .. code-block:: java

                @Compare(left = "from", right = "to", operator = Compare.Operator.LESS_THAN_OR_EQUAL)
                public class Period {
                  @NotNull
                  LocalDate from;
                  LocalDate to;
                }

        * - \ ``from``\  and \ ``to``\  are not both required, carry out comparison check only when values are entered in both \ ``from``\  and \ ``to``\ . When the value is entered in only one of the fields, comparison check is not carried out.
          - Do not assign \ ``@NotNull``\ and use default value （ \ ``false``\ ）in \ ``requireBoth``\  attribute.

            .. code-block:: java

                @Compare(left = "from", right = "to", operator = Compare.Operator.LESS_THAN_OR_EQUAL)
                public class Period {
                  LocalDate from;
                  LocalDate to;
                }

        * - \ ``from``\  and \ ``to``\  are not both required, when values are entered in either of the \ ``from``\ or \ ``to``\ , carry out comparison check by always entering values in both fields.
          - Do not assign \ ``@NotNull``\ and specify \ ``true``\  in \ ``requireBoth``\  attribute.

            .. code-block:: java

                @Compare(left = "from", right = "to", operator = Compare.Operator.LESS_THAN_OR_EQUAL, requireBoth = true)
                public class Period {
                  LocalDate from;
                  LocalDate to;
                }


|

.. _Validation_terasoluna_gfw_how_to_use:

.. raw:: latex

   \newpage

How to apply check rules of common library
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Apply check rules of common library using the procedure below.

Add a dependent library for the rules that are to be used. An example for how to add \ ``terasoluna-gfw-validator``\  is shown below.

.. code-block:: xml

    <dependencies>
        <dependency>
            <groupId>org.terasoluna.gfw</groupId>
            <artifactId>terasoluna-gfw-validator</artifactId>
        </dependency>
    </dependencies>

.. note::

    In the above setting example, since it is assumed that the dependent library version is managed by the parent project terasoluna-gfw-parent , specifying the version in pom.xml is not necessary.

Next, a message definition corresponding to annotation is added to :file:`ValidationMessages.properties`  as explained in \ :ref:`Validation_message_in_validationmessages`\ .

.. code-block:: properties

  # (1)
  org.terasoluna.gfw.common.validator.constraints.ByteMin.message = must be greater than or equal to {value} Bytes
  org.terasoluna.gfw.common.validator.constraints.ByteMax.message = must be less than or equal to {value} Bytes
  org.terasoluna.gfw.common.validator.constraints.Compare.message = not match '{left}' and '{right}'

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Add a message definition for each annotation. Annotation attribute value can be incorporated in the message by using placeholder (\ ``{attribute name}``\  format).

In the end, assign an annotation to JavaBean property as explained in \ :ref:`Validation_basic_validation`\ .

.. note::

    When validation cannot be implemented in Bean Validation due to invalid attribute value of annotation, \ ``javax.validation.ValidationException``\  is thrown. Modify the attribute value to a valid value by referring the reason that is output in stack trace.
    
    Refer Chapter 9 of \ `Bean Validation specification <http://download.oracle.com/otn-pub/jcp/bean_validation-1_1-fr-eval-spec/bean-validation-specification.pdf>`_\  for details.


.. _Validation_terasoluna_gfw_how_to_extend:

How to extend check rules of common library
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Any rule can be created by using check rules provided by common library.

An example is introduced below wherein \ ``@Confirm``\  annotation independently implemented by \ :ref:`Validation_correlation_item_check`\  is created by using check rules provided by common library.

Create \ ``@Confirm``\  annotation by using \ ``@Compare``\  as described in \ :ref:`Validation_convine_existing_constraint`\ .

.. code-block:: java

    package com.example.sample.domain.validation;

    import static java.lang.annotation.ElementType.ANNOTATION_TYPE;
    import static java.lang.annotation.ElementType.TYPE;
    import static java.lang.annotation.RetentionPolicy.RUNTIME;
    
    import java.lang.annotation.Documented;
    import java.lang.annotation.Retention;
    import java.lang.annotation.Target;
    
    import javax.validation.Constraint;
    import javax.validation.OverridesAttribute;
    import javax.validation.Payload;
    import javax.validation.ReportAsSingleViolation;
    
    import org.terasoluna.gfw.common.validator.constraints.Compare;
    
    @Documented
    @Constraint(validatedBy = {})
    @Target({ TYPE, ANNOTATION_TYPE }) // (1)
    @Retention(RUNTIME)
    @ReportAsSingleViolation // (2)
    @Compare(left = "", right = "", operator = Compare.Operator.EQUAL, requireBoth = true) // (3)
    public @interface Confirm {
    
        String message() default "{com.example.sample.domain.validation.Confirm.message}"; // (4)
    
        Class<?>[] groups() default {};
    
        Class<? extends Payload>[] payload() default {};
    
        @OverridesAttribute(constraint = Compare.class, name = "left") // (5)
        String field();
    
        @OverridesAttribute(constraint = Compare.class, name = "right") // (6)
        String confirmField();
    
        @Documented
        @Target({ TYPE, ANNOTATION_TYPE })
        @Retention(RUNTIME)
        public @interface List {
            Confirm[] value();
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Restrict the location where the annotation can be assigned to class or annotation.
    * - | (2)
      - | Message specified in \ ``message``\  attribute of this annotation should be used at the time of error.
    * - | (3)
      - | Specify \ ``Compare.Operator.EQUAL``\ (should be equivalent value) in \ ``operator``\  attribute of \ ``@Compare``\  annotation. Specify \ ``true``\  in \ ``requireBoth``\ attribute since an error occurs if a value is not entered in either of the fields.
    * - | (4)
      - | Define default value of error message.
    * - | (5)
      - | Override \ ``left``\  attribute of \ ``@Compare``\  annotation and change attribute name to \ ``field``\ .
    * - | (6)
      - | Similarly, override \ ``right``\  attribute and change attribute name to \ ``confirmField``\ .

Use annotation created above instead of annotation implemented in \ :ref:`Validation_correlation_item_check`\ .

.. code-block:: java

    package com.example.sample.app.validation;

    import java.io.Serializable;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;

    import com.example.common.validation.Confirm;

    @Confirm(field = "password", confirmField = "confirmPassword") // (1)
    public class PasswordResetForm implements Serializable {
        private static final long serialVersionUID = 1L;

        @NotNull // (2)
        @Size(min = 8)
        private String password;

        @NotNull // (3)
        private String confirmPassword;

        // omitted geter/setter
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Assign \ ``@Confirm``\  annotation to class level.
    * - | (2)
      - | Since \ ``@Confirm``\  verification is passed when \ ``password``\  field is \ ``null``\ , perform \ ``null``\  check by assigning \ ``@NotNull``\  annotation.
    * - | (3)
      - | Similarly assign \ ``@NotNull``\  annotation in \ ``confirmPassword``\  field as well.

.. _Validation_type_mismatch:

Type mismatch
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Pertaining the non-String fields of the form object, if values which cannot be converted to the corresponding data-type have been submitted, \ ``org.springframework.beans.TypeMismatchException``\  is thrown.

In the "New user registration" example, The data-type of "Age" is \ ``Integer``\ . However, if the value entered in this field cannot be converted to an integer, error message is displayed as follows.

.. figure:: ./images_Validation/validations-typemismatch1.png
  :width: 60%

Cause of exception is displayed as it is and is not appropriate as an error message.
It is possible to define the error message for type mismatch in the properties file (application-messages.properties) which is read by \ ``org.springframework.context.MessageSource``\ .

Error messages can be defined with the following rules.

.. tabularcolumns:: |p{0.35\linewidth}|p{0.40\linewidth}|p{0.25\linewidth}|
.. list-table::
    :widths: 35 40 25
    :header-rows: 1

    * - Message key
      - Message contents
      - Application
    * - \ ``typeMismatch``\
      - Default message for all type mismatch errors
      - Default value for entire system
    * - \ ``typeMismatch.[target FQCN]``\
      - Default message for specific type mismatch error
      - Default value for entire system
    * - \ ``typeMismatch.[form attribute name].[property name]``\
      - Type mismatch error message for specific form field
      - Customized message for each screen

When the following messages are added to application-messages.properties,

.. code-block:: properties

  # typemismatch
  typeMismatch="{0}" is invalid.
  typeMismatch.int="{0}" must be an integer.
  typeMismatch.double="{0}" must be a double.
  typeMismatch.float="{0}" must be a float.
  typeMismatch.long="{0}" must be a long.
  typeMismatch.short="{0}" must be a short.
  typeMismatch.java.lang.Integer="{0}" must be an integer.
  typeMismatch.java.lang.Double="{0}" must be a double.
  typeMismatch.java.lang.Float="{0}" must be a float.
  typeMismatch.java.lang.Long="{0}" must be a long.
  typeMismatch.java.lang.Short="{0}" must be a short.
  typeMismatch.java.util.Date="{0}" is not a date.

  # field names
  name=Name
  email=Email
  age=Age

Error message gets changed as shown below.

.. figure:: ./images_Validation/validations-typemismatch2.png
  :width: 60%


| Field name can be inserted in the message by specifying \ ``{0}``\  in the message; This is as per the description in \ :ref:`Validation_message_in_application_messages`\ .
| \ **Default messages should be defined**\.

.. tip::

  Refer to \ `Javadoc of DefaultMessageCodesResolver <http://docs.spring.io/spring/docs/4.3.5.RELEASE/javadoc-api/org/springframework/validation/DefaultMessageCodesResolver.html>`_\  for the details of message key rules.

.. _Validation_string_trimmer_editor:

Binding null to blank string field
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
When the form is sent with input fields left blank in Spring MVC,
empty string instead of \ ``null``\  binds to form object by default.

In this case, the conditions like "blank is allowed but if specified, it must have at least 6 characters" is not satisfied by the use of existing annotations.

| To bind \ ``null``\  instead of empty string to the properties without any input,
| \ ``org.springframework.beans.propertyeditors.StringTrimmerEditor``\  can be used as follows.

.. code-block:: java

  @Controller
  @RequestMapping("xxx")
  public class XxxController {

      @InitBinder
      public void initBinder(WebDataBinder binder) {
          // bind empty strings as null
          binder.registerCustomEditor(String.class, new StringTrimmerEditor(true));
      }

      // omitted ...
  }

With the help of this configuration, it can be specified in the controller whether empty strings which are to be considered as \ ``null``\  or not.

\ :ref:`@ControllerAdvice <application_layer_controller_advice>`\  can be set as a configuration common to the entire project when you want to set reply empty string to \ ``null``\  in the entire project.


.. tip:: **About attribute of @ControllerAdvice annotation added from Spring Framework 4.0**

    By specifying attribute of \ ``@ControllerAdvice``\  annotation,
    it has been improved to allow flexibility in specifying Controller to apply a method implemented in a class wherein \ ``@ControllerAdvice``\  is assigned.
    For details about attribute refer to \ :ref:`Attribute of @ControllerAdvice <application_layer_controller_advice_attribute>`\ .

.. code-block:: java

  @ControllerAdvice
  public class XxxControllerAdvice {

      @InitBinder
      public void initBinder(WebDataBinder binder) {
          // bind empty strings as null
          binder.registerCustomEditor(String.class, new StringTrimmerEditor(true));
      }

      // omitted ...
  }

| When this setting is made, all empty String values set to String fields of form object become \ ``null``\ .
| Therefore it must be noted that, it becomes necessary to specify \ ``@NotNull``\  in case of mandatory check.


.. _Validation_without_native2ascii:

Reading messages which are not converted from Native to Ascii
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A method to read Bean Validation message （\ ``ValidationMessage.properties``\ ） is introduced without converting the message from  Native to Ascii.

When the Japanese message is to be handled directly without converting from  Native to Ascii, it can be easily implemented if it is linked with \ ``MessageSource``\  of Spring.


If it is defined as below,, message read by \ ``MessageSource``\  function can be used in \ ``Hibernate Validator``\ .


* Bean definition

  \ ``*-domain.xml``\ 

 .. code-block:: xml

     <!-- (1) -->
     <bean id="validator" class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean">
         <property name="validationMessageSource">
             <!-- (2) -->
             <bean class="org.springframework.context.support.ResourceBundleMessageSource">
                 <property name="basenames">
                     <list>
                         <value>ValidationMessages</value> <!-- (3) -->
                     </list>
                 </property>
                 <property name="defaultEncoding" value="UTF-8" />
             </bean>
         </property>
     </bean>

     <!-- (4) -->
     <bean class="org.springframework.validation.beanvalidation.MethodValidationPostProcessor">
         <property name="validator" ref="validator" />
     </bean>

 \ ``spring-mvc.xml``\ 

 .. code-block:: xml

     <!-- (5) -->
     <mvc:annotation-driven validator="validator">
         <!-- ommited -->
     </mvc:annotation-driven>

     <!-- (6) -->
     <bean class="org.springframework.validation.beanvalidation.MethodValidationPostProcessor">
         <property name="validator" ref="validator" />
     </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - Define a Bean for \ ``LocalValidatorFactoryBean``\ .
     * - | (2)
       - Define \ ``MessageSource``\ . Here, \ ``ResourceBundleMessageSource``\  is used.
     * - | (3)
       - Specify a resource bundle to be read in \ ``ApplicationContext``\ .
     * - | (4)
       - | Specify a Bean defined by (1) in \ ``validator``\  property of \ ``MethodValidationPostProcessor``\  while using \ :ref:`MethodValidation`\ .
         | When Method Validation is not used, the Bean definition is not required.
     * - | (5)
       - Specify a Bean defined in (1), in  \ ``validator``\ attribute of \ ``<mvc:annotation-driven>``\ element.
     * - | (6)
       - \ Same as (4).

 .. note::

     By using \ ``MessageSource``\ function,
     property file is not necessarily restricted to be placed just under class path. Further, multiple property files can also be specified.
     

.. _Validation_os_command_injection:

OS Command Injection Countermeasures
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
OS command injection, a type of security vulnerability and its countermeasures are explained here.

What is OS command injection -
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
OS command injection is an issue which occurs when malicious commands are supplied to the user input string resulting in illegal manipulation of the computer,
when command execution strings are assembled from user input strings in the application.

.. tip::

    For details, refer \ `Description page <https://www.owasp.org/index.php/OS_Command_Injection>`_\  of OWASP.

In Java, OS command injection can occur when following are used as the commands to be executed when commands are executed
using ``exec``\  method of \ ``ProcessBuilder``\ class  and \ ``Runtime``\  class.

* \ ``/bin/sh``\  (in case of Unix system) or \ ``cmd.exe``\  (In case of Windows)
* String input by user

An example wherein OS command injection occurs when \ ``/bin/sh``\  is used, is shown below.

.. code-block:: java

  ProcessBuilder pb = new ProcessBuilder("/bin/sh", "-c", script); // (1)
  Process p = pb.start();


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | For example, when "exec.sh ; cat /etc/passwd" is input in the \ ``script``\ , semicolon in the string is interpreted as a delimiter by \ ``/bin/sh``\  and "cat /etc/passwd" is executed.
        | Hence, \ ``/etc/passwd``\  is likely to be output depending on how the standard output is handled.

.. warning:: **How to use ScriptEngine and ScriptTemplateViewResolver**

    Another language  (\ ``Ruby``\ or \ ``Python``\ etc) can be used on JVM , in \ ``ScriptEngine``\  added from  Java SE 6
    or \ ``ScriptTemplateViewResolver``\  added from  Spring Framework 4.2.

    When code of another language is to be executed using this function, care must be taken while using the function
    since OS command injection is likely to occur during writing the code.

Countermeasures
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Running external processes should be avoided as much as possible in order to prevent occurrence of OS command injection. However, if it is necessary to execute the external process due to certain circumstances,
it must be executed after implementing the countermeasures below.

* Commands which use \ ``/bin/sh``\  (in case of Unix system) and \ ``cmd.exe``\  (in case of Windows) should not be executed as far as possible.
* Check whether the characters entered by user are allowed by the application using the whitelist system

The rules to check whether commands and arguments entered by the user are configured  by the specified string are shown below.

.. code-block:: java

    @Pattern(regexp = "batch0\\d\\.sh") // (1)
    private String cmdStr;

    @Pattern(regexp = "[\\w=_]+")  // (2)
    private String arg;


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify rule to allow only \ ``batch0X.sh``\ (X is a single byte numeral from 0 to 9).
    * - | (2)
      - | Specify rule to allow only the strings configured from  innocuous alphanumeric characters (\\w）、\ ``=``\ 、\ ``_``\) as arguments.

.. note::

    In this example, directory traversal is prevented by establishing a rule wherein the path is not included in the commands or arguments.
      
When \ ``@Pattern``\  is used, regular expressions specified in \ ``@Pattern``\  are output as error messages and
are considered invalid as messages due to following points.

* Significance of error is not clear, not user-friendly
* logic for the countermeasures to the vulnerability is exposed to the user

.. figure:: ./images_Validation/validations-os-command-injection.png
  :width: 60%

A valid message is defined in application-messages.properties to conceal the logic by clarifying the significance of error.
For how to define a message, refer \ :ref:`Validation_message_in_application_messages`\.

.. code-block:: properties

  Pattern.cmdForm.cmdStr = permit command name: batch00.sh - batch09.sh
  Pattern.cmdForm.arg = permit parameter characters and symbols: alphanumeric, =, _

.. figure:: ./images_Validation/validations-os-command-injection2.png
  :width: 60%

.. raw:: latex

   \newpage

