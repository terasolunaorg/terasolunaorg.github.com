Exception Handling
------------------

.. only:: html

 .. contents:: Table of Contents
    :depth: 4
    :local:

This chapter describes exception handling mechanism for web applications created using this guideline.

Overview
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This section illustrates handling of exceptions occurring within the boundary of Spring MVC. The scope of description is as follows:

.. figure:: ./images/exception-handling-description-target.png
  :alt: description target
  :width: 80%
  :align: center

  **Figure - Description Targets**

#. :ref:`exception-handling-class-label`
#. :ref:`exception-handling-method-label`


.. _exception-handling-class-label:

Classification of exceptions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The exceptions that occur when an application is running, can be broadly classified into following 3 categories.

.. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.30\linewidth}|p{0.30\linewidth}|
.. list-table:: **Table - Types of exceptions that occur when an application is running**
   :header-rows: 1
   :widths: 10 30 30 30

   * - Sr. No.
     - Classification
     - Description
     - :ref:`exception-handling-exception-type-label`
   * - | (1)
     - | Exceptions wherein cause can be resolved if the user re-executes the operation (by changing input values etc.)
     - | The exceptions wherein cause can be resolved if the user re-executes the operation, are handled in application code.
     - | 1. :ref:`exception-handling-exception-type-businessexception-label`
       | 2. :ref:`exception-handling-exception-type-libraryexception-label`
   * - | (2)
     - | Exceptions wherein cause cannot be resolved even if the user re-executes the operation
     - | The exceptions wherein cause cannot be resolved even if the user re-executes the operation, are handled using the framework.
     - | 1. :ref:`exception-handling-exception-type-systemexception-label`
       | 2. :ref:`exception-handling-exception-type-unexpectedexception-label`
       | 3. :ref:`exception-handling-exception-type-error-label`
   * - | (3)
     - | Exceptions due to invalid requests from the client
     - | The exceptions which occur due to invalid requests from the client, are handled using the framework.
     - | 1. :ref:`exception-handling-exception-type-request-label`

.. note:: **Who is a target audience for exceptions?**

  - Application Developer should be aware of exception (1).
  - Application Architect should be aware of exceptions (2) and (3).


.. _exception-handling-method-label:

Exception handling methods
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| The exceptions which occur when the application is running, are handled using following 4 methods.
| For details on flow of each handling method, refer to \ :ref:`exception-handling-basic-flow-label`\ .

.. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.35\linewidth}|p{0.25\linewidth}|
.. list-table:: **Table - Exception Handling Methods**
   :header-rows: 1
   :widths: 10 30 35 25

   * - Sr. No.
     - Handling Method
     - Description
     - :ref:`exception-handling-pattern-label`
   * - | (1)
     - | \ Use ``try-catch``\  in the application code to carry out exception handling.
     - | Use this in order to handle exceptions at request (Controller method) level.
       | For details, refer to \ :ref:`exception-handling-basic-flow-catch-label`\ .
     - | 1. :ref:`exception-handling-class-from-middle-label`
   * - | (2)
     - | Use \ ``@ExceptionHandler``\  annotation to carry out exception handling in application code.
     - | Use this when exceptions are to be handled at use case (Controller) level.
       | For details, refer to \ :ref:`exception-handling-basic-flow-annotation-label`\ .
     - | 1. :ref:`exception-handling-class-from-first-label`
   * - | (3)
     - | Use HandlerExceptionResolver mechanism provided by the framework to carry out exception handling.
     - | Use this in order to handle exceptions at servlet level.
       | When \ ``<mvc:annotation-driven>``\  is specified, HandlerExceptionResolver uses \ :ref:`automatically registered class <ExceptionHandling-annotation-driven>`\ , and \ ``SystemExceptionResolver``\  provided by common library.
       | For details, refer to \ :ref:`exception-handling-basic-flow-resolver-label`\ .
     - | 1. :ref:`exception-handling-class-systemerror-label`
       |
       | 2. :ref:`exception-handling-class-requesterror-label`
   * - | (4)
     - | Use the error-page function of the servlet container to carry out exception handling.
     - | Use this in order to handle fatal errors and exceptions which are out of the boundary of Spring MVC.
       | For details, refer to \ :ref:`exception-handling-basic-flow-container-label`\ .
     - | 1. :ref:`exception-handling-class-fatalerror-label`
       |
       | 2. :ref:`exception-handling-class-viewerror-label`


.. figure:: ./images/exception-handling-method.png
  :alt: handling method
  :width: 80%
  :align: center

  **Figure - Exception Handling Methods**


.. note:: **Who will carry out exception handling?**

  - Application Developer should design and implement (1) and (2).
  - Application Architect should design and configure (3) and (4).

.. note:: **About automatically registered HandlerExceptionResolver**

  When <mvc:annotation-driven> is specified, the roles of HandlerExceptionResolver, which is registered automatically, are as follows:

  The priority order will be as given below.

  .. _ExceptionHandling-annotation-driven:

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.55\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 30 55

       * - Sr. No.
         - Class (Priority order)
         - Role
       * - | (1)
         - | ExceptionHandlerExceptionResolver
           | (order=0)
         - | This is a class for handling the exceptions by calling the methods of Controller class with \ ``@ExceptionHandler``\  annotation.
           | This class is necessary for implementing the handling method No. 2.
       * - | (2)
         - | ResponseStatusExceptionResolver
           | (order=1)
         - | This is a class for handling the exceptions wherein \ ``@ResponseStatus``\  is applied as class level annotation.
           | \ ``HttpServletResponse#sendError(int sc, String msg)``\  is called with the value specified in \ ``@ResponseStatus``\ .
       * - | (3)
         - | DefaultHandlerExceptionResolver
           | (order=2)
         - | This is a class for handling framework exceptions in Spring MVC.
           | \ ``HttpServletResponse#sendError(int sc)``\  is called using the value of HTTP response code corresponding to framework exception.
           | For details on HTTP response code to be set, refer to \ :ref:`exception-handling-appendix-defaulthandlerexceptionresolver-label`\ .

.. note:: **What is a role of SystemExceptionResolver provided by common library?**

  This is a class for handling exceptions which are not handled by HandlerExceptionResolver
  which is registered automatically when <mvc:annotation-driven> is specified.
  Therefore, the order of priority of this class should be set after DefaultHandlerExceptionResolver.

.. note:: **About @ControllerAdvice annotation added from Spring Framework 3.2**

  \ ``@ControllerAdvice``\  made exception handling possible using \ ``@ExceptionHandler``\  at servlet level.
  If methods with \ ``@ExceptionHandler``\  annotation are defined in a class with \ ``@ControllerAdvice``\  annotation, then exception handling carried out in the methods with \ ``@ExceptionHandler``\  annotation can be applied to all the Controllers in Servlet.
  When implementing the above in earlier versions, it was necessary to define methods with \ ``@ExceptionHandler``\  annotation in Controller base class
  and inherit each Controller from the base class.

 **About attribute of @ControllerAdvice annotation added from Spring Framework 4.0**

  By specifying attribute of \ ``@ControllerAdvice``\  annotation,
  it has been improved in such a way that Controller that applies a method implemented in class with \ ``@ControllerAdvice``\ , can be specified flexibly.
  For details on attribute, refer to \ :ref:`attribute of @ControllerAdvice <application_layer_controller_advice_attribute>`\ .


.. note:: **Where to use @ControllerAdvice annotation?**

  #. When logic other than determining the View name and HTTP response code is necessary for exception handling to be carried out at servlet level,
     (If only determining View name and HTTP response code is sufficient, it can be implemented using \ ``SystemExceptionResolver``\ )
  #. In case of exception handling carried out at servlet level, when response data is to be created by serializing the error model (JavaBeans) in JSON or XML formats
     without using template engines such as JSP.
     (Used for error handling at the time of creating Controller for AJAX or REST).


Detail
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. :ref:`exception-handling-exception-type-label`
#. :ref:`exception-handling-pattern-label`
#. :ref:`exception-handling-basic-flow-label`

|

.. _exception-handling-exception-type-label:

Types of exceptions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
There are 6 types of exceptions that occur in a running application.


.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.30\linewidth}|
.. list-table:: **Table - Types of Exceptions**
   :header-rows: 1
   :widths: 10 20 30

   * - Sr. No.
     - Types of Exceptions
     - Description
   * - | (1)
     - | :ref:`exception-handling-exception-type-businessexception-label`
     - | Exception to notify a violation of a business rule
   * - | (2)
     - | :ref:`exception-handling-exception-type-libraryexception-label`
     - | Amongst the exceptions that occur in framework and libraries, the exception which is likely to occur when the system is operating normally.
   * - | (3)
     - | :ref:`exception-handling-exception-type-systemexception-label`
     - | Exception to notify detection of a state which should not occur when the system is operating normally
   * - | (4)
     - | :ref:`exception-handling-exception-type-unexpectedexception-label`
     - | Unchecked exception that does not occur when the system is operating normally
   * - | (5)
     - | :ref:`exception-handling-exception-type-error-label`
     - | Error to notify an occurrence of a fatal error impacting the entire system (application)
   * - | (6)
     - | :ref:`exception-handling-exception-type-request-label`
     - | Exception to notify that the framework has detected invalid request contents


.. _exception-handling-exception-type-businessexception-label:

Business exception
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| **Exception to notify a violation of a business rule**
| This exception is generated in domain layer.
| The situations in the system such as below are pre-defined and hence need not be dealt by the system administrator.

* If the reservation date is past the deadline when making travel reservations
* If a product is out of stock when it is ordered
* etc ...

.. note:: **Corresponding Exception Class**

  - \ ``org.terasoluna.gfw.common.exception.BusinessException``\  (Class provided by common library).
  - When handling is to be carried out at detailed level, exception class that inherits BusinessException should be created.
  - If the requirements cannot be met by the  exception class provided by common library, a business exception class should be created for each project.


.. _exception-handling-exception-type-libraryexception-label:

Library exceptions that occurs during normal operation
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| Amongst the exceptions that occur in the framework and libraries, **the exception which is likely to occur when the system is operating normally.**
| The exceptions that occur in the framework and libraries cover the exception classes in Spring Framework or other libraries. 
| Situations such are below are pre-defined and hence need not be dealt by the system administrator.

* Optimistic locking exception and pessimistic locking exception that occur if multiple users attempt to update same data simultaneously.
* Unique constraint violation exception that occurs if multiple users attempt to register same data simultaneously.
* etc ...

.. note:: **Examples of corresponding Exception Classes**

  - ``org.springframework.dao.OptimisticLockingFailureException`` (Exception that occurs if there is an error due to optimistic locking).
  - ``org.springframework.dao.PessimisticLockingFailureException`` (Exception that occurs if there is an error due to pessimistic locking).
  - ``org.springframework.dao.DuplicateKeyException`` (Exception that occurs if there is a unique constraint violation).
  - etc ...

.. todo::

  **Currently it has been found that unexpected errors occur if JPA(Hibernate) is used.**

  * In case of unique constraint violation, \ ``org.springframework.dao.DataIntegrityViolationException``\  occurs and not \ ``DuplicateKeyException``\ .


.. _exception-handling-exception-type-systemexception-label:

System Exception
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| **Exception to notify that a state which should not occur when the system is operating normally has been detected**.
| This exception should be generated in application layer and domain layer.
| The exception needs to be dealt by the system administrator.

* If the master data, directories, files, etc. those should have pre-existed, do not exist
* Amongst the checked exceptions that occur in the framework, libraries, if exceptions classified as system errors are caught (IOException during file operations etc.).
* etc ...

.. note:: **Corresponding Exception Class**

  - \ ``org.terasoluna.gfw.common.exception.SystemException``\  (Class provided by common library).
  - When the error screen of View and HTTP response code need to be switched on the basis of the cause of each error, SystemException should be inherited for each error cause and the mapping between the inherited exception class and the error screen should be defined in the bean definition of SystemExceptionResolver.
  - If the requirements are not met by the system exception class provided in the common library, a system exception class should be created in each respective project.


.. _exception-handling-exception-type-unexpectedexception-label:

Unexpected System Exception
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| **Unchecked exception that does not occur when the system is operating normally.**
| Action by system administrator or analysis by system developer is necessary.
| **Unexpected system exceptions should not be handled (try-catch) in the application code.**

* When bugs are hidden in the application, framework or libraries
* When DB Server etc. is down
* etc ...

.. note:: **Example of Corresponding Exception Class**

  - \ ``java.lang.NullPointerException``\  (Exception caused due to bugs).
  - \ ``org.springframework.dao.DataAccessResourceFailureException``\  (Exception that occurs when DB server is down).
  - etc ...


.. _exception-handling-exception-type-error-label:

Fatal Errors
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| **Error to notify that a fatal problem impacting the entire system (application), has occurred**.
| Action/recovery by system administrator or system developer is necessary.
| **Fatal Errors (error that inherits java.lang.Error) must not be handled (try-catch) in the application code.**

* If the memory available in Java virtual machine is inadequate
* etc ...

.. note:: **Example of corresponding Error Class**

  - \ ``java.lang.OutOfMemoryError``\  (Error due to inadequate memory).
  - etc ...


.. _exception-handling-exception-type-request-label:

Framework exception in case of invalid requests
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| **Exception to notify that the framework has detected invalid request contents**.
| This exception occurs in the framework (Spring MVC).
| The cause lies at client side; hence it need not be dealt by the system administrator.

* Exception that occurs when a request path for which only POST method is permitted, is accessed using GET method.
* Exception that occurs when type-conversion fails for the values extracted from URI using @PathVariable annotation.
* etc ...

.. note:: **Example of corresponding Exception Class**

  - \ ``org.springframework.web.HttpRequestMethodNotSupportedException``\  (Exception that occurs when the access is made through a HTTP method which is not supported).
  - \ ``org.springframework.beans.TypeMismatchException``\  (Exception that occurs if the specified value cannot be converted to URI).
  - etc ...

 \ Class for which HTTP status code is "4XX" in the list given at \ :ref:`exception-handling-appendix-defaulthandlerexceptionresolver-label`\ .


.. _exception-handling-pattern-label:

Exception Handling Patterns
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| There are 6 types of exception handling patterns based on the purpose of handling.
| (1)-(2) should be handled at use case level and (3)-(6) should be handled at the entire system (application) level.

.. tabularcolumns:: |p{0.10\linewidth}|p{0.40\linewidth}|p{0.25\linewidth}|p{0.10\linewidth}|p{0.15\linewidth}|
.. list-table:: **Table - Exception Handling Patterns**
   :header-rows: 1
   :widths: 10 40 25 10 15

   * - Sr. No.
     - Purpose of handling
     - Types of exceptions
     - Handling method
     - Handling location
   * - | (1)
     - | :ref:`exception-handling-class-from-middle-label`
     - | 1. :ref:`exception-handling-exception-type-businessexception-label`
     - | Application code
       | (try-catch)
     - | Request
   * - | (2)
     - | :ref:`exception-handling-class-from-first-label`
     - | 1. :ref:`exception-handling-exception-type-businessexception-label`
       | 2. :ref:`exception-handling-exception-type-libraryexception-label`
     - | Application code
       | (@ExceptionHandler)
     - | Use case
   * - | (3)
     - | :ref:`exception-handling-class-systemerror-label`
     - | 1. :ref:`exception-handling-exception-type-systemexception-label`
       | 2. :ref:`exception-handling-exception-type-unexpectedexception-label`
     - | Framework
       | (Handling rules are specified in \ ``spring-mvc.xml``\ )
     - | Servlet
   * - | (4)
     - | :ref:`exception-handling-class-requesterror-label`
     - | 1. :ref:`exception-handling-exception-type-request-label`
     - | Framework
     - | Servlet
   * - | (5)
     - | :ref:`exception-handling-class-fatalerror-label`
     - | 1. :ref:`exception-handling-exception-type-error-label`
     - | Servlet container
       | (Handling rules are specified in \ ``web.xml``\ )
     - | Web application
   * - | (6)
     - | :ref:`exception-handling-class-viewerror-label`
     - | 1. All the exceptions and errors that occur in the presentation layer
     - | Servlet container
       | (Handling rules are specified in \ ``web.xml``\ )
     - | Web application


.. _exception-handling-class-from-middle-label:

When notifying partial redo of a use case (from middle)
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
When partial redo (from middle) of a use case is to be notified, catch (try-catch) the exception in the application code of Controller class and carry out exception handling at request level.

.. note:: **Example of notifying partial redo of a use case**

  - | When an order is placed through a shopping site, if business exception notifying stock shortage occurs.
    | In such a case, order can be placed if the no. of items to be ordered is reduced; hence display a screen on which no. of items can be changed and prompt a message asking to change the same.

.. figure:: ./images/exception-handling-class-again-from-middle.png
  :alt: class of exception handling for again from middle
  :width: 80%
  :align: center

  **Figure - Handling method when notifying partial redo of a use case (from middle)**


.. _exception-handling-class-from-first-label:

When notifying redo of a use case (from beginning)
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
When redo of a use case (from beginning) is to be notified, catch the exception using \ ``@ExceptionHandler``\ , and carry out exception handling at use case level.

.. note:: **Example when notifying redo of a use case (from beginning)**

  - | At the time of changing the product master on shopping site (site for administrator), it has been changed by other operator (in case of optimistic locking exception).
    | In such a case, the operation needs to be carried out after verifying the changes made by the other user; hence display the front screen of the use case (for example: search screen of product master) and prompt a message asking the user to perform the operation again.

.. figure:: ./images/exception-handling-class-again-from-first.png
  :alt: class of exception handling for again from first
  :width: 80%
  :align: center

  **Figure - Handling method when notifying redo of a use case (from beginning)**


.. _exception-handling-class-systemerror-label:

When notifying that the system or application is not in a normal state
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
When an exception to notify that system or application is not in a normal state is detected, catch the exception using SystemExceptionResolver and carry out exception handling at servlet level.

.. note:: **Examples for notifying that the system or application is not in a normal state**

  - | In case of a use case for connecting to an external system, if an exception occurs notifying that the external system is blocked.
    | In such a case, since use case cannot be executed until external system resumes service, display the error screen, and notify that the use case cannot be executed till the external system resumes service.
  - | When searching master information with the value specified in the application, if the corresponding master information does not exist.
    | In such a case, there is a possibility of bug in master maintenance function or of error (release error) in data input by the system administrator; hence display the system error screen and notify that a system error has occurred.
  - | When IOException occurs from the API during file operations.
    | This could be a case of disk failure etc.; hence display the system error screen and notify that system error has occurred.
  - etc ...


.. figure:: ./images/exception-handling-class-systemerror.png
  :alt: class of exception handling for system error
  :width: 80%
  :align: center

  **Figure - Handling method when an exception to notify that system or application is not in a normal state is detected**


.. _exception-handling-class-requesterror-label:

When notifying that the request contents are invalid
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
When notifying that an invalid request is detected by the framework, catch the exception using DefaultHandlerExceptionResolver, and carry out exception handling at servlet level.

.. note:: **Example when notifying that the request contents are invalid**

  - | When a URI is accessed using GET method, while only POST method is permitted.
    | In such a case, it is likely that the access has been made directly using the Favorites feature etc. of the browser; hence display the error screen and notify that the request contents are invalid.
  - | When value cannot be extracted from URI using \ ``@PathVariable``\  annotation
    | In such a case, it is likely that the access has been made directly by replacing the value of the address bar on the browser; hence display the error screen and notify that the request contents are invalid.
  - etc ...


.. figure:: ./images/exception-handling-class-requesterror.png
  :alt: class of exception handling for request error
  :width: 80%
  :align: center

  **Figure - Handling method when notifying that the request contents are invalid**


.. _exception-handling-class-fatalerror-label:

When a fatal error has been detected
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
When a fatal error has been detected, catch the exception using servlet container and carry out exception handling at web application level.

.. figure:: ./images/exception-handling-class-fatalerror.png
  :alt: class of exception handling for fatal error
  :width: 80%
  :align: center

  **Figure - Handling method when a fatal error has been detected**


.. _exception-handling-class-viewerror-label:

When notifying that an exception has occurred in the presentation layer (JSP etc.)
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
When notifying that an exception has occurred in the presentation layer (JSP etc.), catch the exception using servlet container and carry out exception handling at web application level.

.. figure:: ./images/exception-handling-class-jsperror.png
  :alt: class of exception handling for fatal error
  :width: 80%
  :align: center

  **Figure - Handling method when notifying an occurrence of exception in the presentation layer (JSP etc.)**


.. _exception-handling-basic-flow-label:

Basic Flow of Exception Handling
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| The basic flow of exception handling is shown below.
| For an overview of classes provided by common library, refer to \ :ref:`exception-handling-about-classes-of-library-label`\ .
| **The processing to be implemented in the application code is indicated in Bold.**
| The log of stack trace and exception messages is output by the classes (Filter and Interceptor class) provided by the common library.
| When any information other than exception messages and stack trace needs to be logged, it should be implemented separately in each of the logic.
| This section describes the flow of exception handling; hence the description related to flow till the calling of Service class is omitted.

#. :ref:`exception-handling-basic-flow-catch-label`
#. :ref:`exception-handling-basic-flow-annotation-label`
#. :ref:`exception-handling-basic-flow-resolver-label`
#. :ref:`exception-handling-basic-flow-container-label`


.. _exception-handling-basic-flow-catch-label:

Basic flow when the Controller class handles the exception at request level
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| In order to handle the exception at request level, catch (try-catch) the exception in the application code of the Controller class.
| Refer to the figure below:
| It illustrates the basic flow at the time of handling a business exception (\ ``org.terasoluna.gfw.common.exception.BusinessException``\ ) provided by common library.
| Log is output using interceptor (\ ``org.terasoluna.gfw.common.exception.ResultMessagesLoggingInterceptor``\ ) which records that an exception holding the result message has occurred.

.. figure:: ./images/exception-handling-flow-catch.png
  :alt: flow of exception handling using catch
  :width: 80%
  :align: center

  **Figure - Basic flow when the Controller class handles the exception at request level**

4. In Service class, BusinessException is generated and thrown.
#. ResultMessagesLoggingInterceptor calls ExceptionLogger, and outputs log of warn level (monitoring log and application log).
   ResultMessagesLoggingInterceptor class outputs logs only when sub exception (BusinessException/ResourceNotFoundException) of ResultMessagesNotificationException occurs.
#. **Controller class catches BusinessException, extracts the message information (ResultMessage) from BusinessException and sets it to Model for screen display(6').**
#. **Controller class returns the View name.**
#. DispatcherServlet calls JSP corresponding to the returned View name.
#. **JSP acquires message information (ResultMessage) using MessagesPanelTag and generates HTML code for message display.**
#. The response generated by JSP is displayed.


.. _exception-handling-basic-flow-annotation-label:

Basic flow when the Controller class handles the exception at use case level
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| In order to handle the exception at use case level, catch the exception using  \ ``@ExceptionHandler``\  of Controller class.
| Refer to the figure below.
| It illustrates the basic flow at the time of handling an arbitrary exception (\ ``XxxException``\ ).
| Log is output using interceptor (\ ``org.terasoluna.gfw.web.exception.HandlerExceptionResolverLoggingInterceptor``\ ). This interceptor records that the exception is handled using HandlerExceptionResolver.

.. figure:: ./images/exception-handling-flow-annotation.png
  :alt: flow of exception handling using annotation
  :width: 80%
  :align: center

  **Figure - Basic flow when the Controller class handles the exception at use case level**

3. Exception (XxxException) is generated in the Service class called from Controller class.
#. DispatcherServlet catches XxxException and calls ExceptionHandlerExceptionResolver.
#. ExceptionHandlerExceptionResolver calls exception handling method provided in Controller class.
#. **Controller class generates message information (ResultMessage) and sets it to the Model for screen display.**
#. **Controller class returns the View name.**
#. ExceptionHandlerExceptionResolver returns the View name returned by the Controller.
#. HandlerExceptionResolverLoggingInterceptor calls ExceptionLogger and outputs logs (monitoring log and application log) (at info, warn, error levels) corresponding to the exception code.
#. HandlerExceptionResolverLoggingInterceptor returns View name returned by ExceptionHandlerExceptionResolver.
#. DispatcherServlet calls JSP that corresponds to the returned View name.
#. **JSP acquires the message information (ResultMessage) using MessagesPanelTag and generates HTML code for message display.**
#. The response generated by JSP is displayed.


.. _exception-handling-basic-flow-resolver-label:

Basic flow when the framework handles the exception at servlet level
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| In order to handle the exception using the framework (at servlet level), catch the exception using SystemExceptionResolver.
| Refer to the figure below.
| It illustrates the basic flow at the time of handling the system exception (\ ``org.terasoluna.gfw.common.exception.SystemException``\ ) provided by common library using \ ``org.terasoluna.gfw.web.exception.SystemExceptionResolver``\  .
| Log is output using interceptor (\ ``org.terasoluna.gfw.web.exception.HandlerExceptionResolverLoggingInterceptor``\ ) which records the exception specified in the argument of exception handling method.

.. figure:: ./images/exception-handling-flow-resolver.png
  :alt: flow of exception handling using resolver
  :width: 80%
  :align: center

  **Figure - Basic flow when the framework handles the exception at servlet level**

4. A state equivalent to the system exception is detected in Service class; hence throw a SystemException.
#. DispatcherServlet catches SystemException, and calls SystemExceptionResolver.
#. SystemExceptionResolver acquires exception code from SystemException and sets it to HttpServletRequest for screen display (6').
#. SystemExceptionResolver returns the View name corresponding to SystemException.
#. HandlerExceptionResolverLoggingInterceptor calls ExceptionLogger and outputs logs (monitoring log and application log) (at info, warn, error levels) corresponding to the exception code.
#. HandlerExceptionResolverLoggingInterceptor returns the View name returned by SystemExceptionResolver.
#. DispatcherServlet calls the JSP corresponding to the returned View name.
#. **JSP acquires the exception code from HttpServletRequest and  inserts it in HTML code for message display.**
#. The response generated by JSP is displayed.


.. _exception-handling-basic-flow-container-label:

Basic flow when the servlet container handles the exception at web application level
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| In order to handle exceptions at web application level, catch the exception using servlet container.
| Fatal errors, exceptions which are not handled using the framework (exceptions in JSP etc.) and exceptions occurred in Filter are to be handled using this flow.
| Refer to the figure below.
| It illustrates the basic flow at the time of handling java.lang.Exception by "error page".
| Log is output using servlet filter (\ ``org.terasoluna.gfw.web.exception.ExceptionLoggingFilter``\ ) which records that an unhandled exception has occurred.

.. figure:: ./images/exception-handling-flow-container.png
  :alt: flow of exception handling using container
  :width: 80%
  :align: center

  **Figure - Basic flow when servlet container handles the exception at web application level**

4. DispatcherServlet catches XxxError, wraps it in ServletException and then throws it.
#. ExceptionLoggingFilter catches ServletException and calls ExceptionLogger. ExceptionLogger outputs logs (monitoring log and application log) (at info, warn, error levels) corresponding to the exception code. ExceptionLoggingFilter re-throws ServletException.
#. ServletContainer catches ServletException and outputs logs to server log. Log level varies depending on the application server.
#. ServletContainer calls the View (HTML etc.) defined in ``web.xml``.
#. The response generated by the View which is called by the ServletContainer, is displayed.

|

.. _exception-handling-how-to-use-label:

How to use
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The usage of exception handling functionality is described below.

For exception handling classes provided by common library, refer to \ :ref:`exception-handling-about-classes-of-library-label`\ .

#. :ref:`exception-handling-how-to-use-application-configuration-label`
#. :ref:`exception-handling-how-to-use-codingpoint-service-label`
#. :ref:`exception-handling-how-to-use-codingpoint-controller-label`
#. :ref:`exception-handling-how-to-use-codingpoint-jsp-label`


.. _exception-handling-how-to-use-application-configuration-label:

Application Settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| The application settings required for using exception handling are shown below.
| Further, these settings are already done in blank project. Hence, it will work only by doing changes given in \ **[Location to be customized for each project]**\ section.

#. :ref:`exception-handling-how-to-use-application-configuration-common-label`
#. :ref:`exception-handling-how-to-use-application-configuration-domain-label`
#. :ref:`exception-handling-how-to-use-application-configuration-app-label`
#. :ref:`exception-handling-how-to-use-application-configuration-container-label`


.. _exception-handling-how-to-use-application-configuration-common-label:

Common Settings
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

1. Add bean definition of logger class (\ ``ExceptionLogger``\ ) which will output exception log.

- **applicationContext.xml**

 .. code-block:: xml
    :emphasize-lines: 3,5,11,16-17

    <!-- Exception Code Resolver. -->
    <bean id="exceptionCodeResolver"
        class="org.terasoluna.gfw.common.exception.SimpleMappingExceptionCodeResolver"> <!-- (1) -->
        <!-- Setting and Customization by project. -->
        <property name="exceptionMappings"> <!-- (2) -->
            <map>
                <entry key="ResourceNotFoundException" value="e.xx.fw.5001" />
                <entry key="BusinessException" value="e.xx.fw.8001" />
            </map>
        </property>
        <property name="defaultExceptionCode" value="e.xx.fw.9001" /> <!-- (3) -->
    </bean>

    <!-- Exception Logger. -->
    <bean id="exceptionLogger"
        class="org.terasoluna.gfw.common.exception.ExceptionLogger"> <!-- (4) -->
        <property name="exceptionCodeResolver" ref="exceptionCodeResolver" /> <!-- (5) -->
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Add \ ``ExceptionCodeResolver``\  to bean definition.
    * - | (2)
      - | Specify mapping between name of the exception to be handled and applicable "Exception Code (Message ID)".
        | In the above example, if the "BusinessException" is included in the class name of exception class (or parent class), "w.xx.fw.8001" will be the "Exception code (Message ID)" and if "ResourceNotFoundException" is included in the class name of exception class (or parent class), "w.xx.fw.5001" will be the "Exception code (Message ID)".

        .. note:: **About the Exception Code (Message ID)**

             The exception code is defined here for taking into account the case wherein message ID is not specified in generated "BusinessException",
             however, it is recommended that you specify the "Exception Code (Message ID)" at the implementation side which generates the "BusinessException" (this point is explained later).
             Specification of "Exception Code (Message ID)" for "BusinessException" is an alternative measure in case it is not specified when "BusinessException" occurs.

        | **[Location to be customized for each project]**
    * - | (3)
      - | Specify default "Exception Code (Message ID)".
        | In the above example, if "BusinessException" or "ResourceNotFoundException" is not included in the class names of exception class (or parent class), " e.xx.fw.9001" will be "Exception code (Message ID)".
        | **[Location to be customized for each project]**

        .. note:: **Exception Code (Message ID)**

             Exception code is only to be output to log (It can also be fetched on screen).
             It is also possible to create an ID which need not be in the format usually used to define IDs in Properties file, but can be identified in the application.
             For example, MA7001 etc.

    * - | (4)
      - | Add \ ``ExceptionLogger``\  to bean definition.
    * - | (5)
      - | Inject \ ``ExceptionCodeResolver``\  into the bean definition of \ ``ExceptionLogger``\ .


2. Add log definition.

- **logback.xml**

 Add log definition for monitoring log.

 .. code-block:: xml
    :emphasize-lines: 1,13-15

    <appender name="MONITORING_LOG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender"> <!-- (1) -->
        <file>${app.log.dir:-log}/projectName-monitoring.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${app.log.dir:-log}/projectName-monitoring-%d{yyyyMMdd}.log</fileNamePattern>
            <maxHistory>7</maxHistory>
        </rollingPolicy>
        <encoder>
            <charset>UTF-8</charset>
            <pattern><![CDATA[date:%d{yyyy-MM-dd HH:mm:ss}\tX-Track:%X{X-Track}\tlevel:%-5level\tmessage:%msg%n]]></pattern>
        </encoder>
    </appender>

    <logger name="org.terasoluna.gfw.common.exception.ExceptionLogger.Monitoring" additivity="false"> <!-- (2) -->
        <level value="error" /> <!-- (3) -->
        <appender-ref ref="MONITORING_LOG_FILE" /> <!-- (4) -->
    </logger>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify appender definition used for outputting monitoring log. In the above example, an appender to be output to a file has been specified, however the appender used should be consider as per the system requirements.
        | **[Location to be customized for each project]**
    * - | (2)
      - | Specify logger definition for monitoring log. When creating ExceptionLogger, if any logger name is not specified, the above settings can be used as is.

        .. warning:: **About additivity setting value**

            Specify \ ``false``\ . If \ ``true``\  is specified, the same log will be output by upper level logger (for example, root).

    * - | (3)
      - | Specify output level. In ExceptionLogger, 3 types of logs of info, warn, error are output; however, the level specified should be as per the system requirements. The guideline recommends Error level.
        | **[Location to be customized for each project]**
    * - | (4)
      - | Specify the appender which will act as output destination.
        | **[Location to be customized for each project]**


 Add log definition for application log.

 .. code-block:: xml
    :emphasize-lines: 1,13

    <appender name="APPLICATION_LOG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender"> <!-- (1) -->
        <file>${app.log.dir:-log}/projectName-application.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${app.log.dir:-log}/projectName-application-%d{yyyyMMdd}.log</fileNamePattern>
            <maxHistory>7</maxHistory>
        </rollingPolicy>
        <encoder>
            <charset>UTF-8</charset>
            <pattern><![CDATA[date:%d{yyyy-MM-dd HH:mm:ss}\tthread:%thread\tX-Track:%X{X-Track}\tlevel:%-5level\tlogger:%-48logger{48}\tmessage:%msg%n]]></pattern>
        </encoder>
    </appender>

    <logger name="org.terasoluna.gfw.common.exception.ExceptionLogger"> <!-- (2) -->
        <level value="info" /> <!-- (3) -->
    </logger>

    <root level="warn">
        <appender-ref ref="STDOUT" />
        <appender-ref ref="APPLICATION_LOG_FILE" /> <!-- (4) -->
    </root>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify appender definition used for outputting application log. In the above example, an appender to be output to a file has been specified, however the appender used should be consider as per the system requirements.
        | **[Location to be customized for each project]**
    * - | (2)
      - | Specify logger definition for application log. When creating ExceptionLogger, if any logger name is not specified, the above settings can be used as is.

        .. note:: **Appender definition for outputting application log**

             Rather than defining a separate appender for logging exceptions, it is recommended to use the same appender which is used for logging in application code or framework.
             By using same output destination, it becomes easier to track the process until an exception occurs. 

    * - | (3)
      - | Specify the output level. In ExceptionLogger, 3 types of logs of info, warn, error are output; however, the level specified should be as per the system requirements. This guideline recommends info level.
        | **[Location to be customized for each project]**
    * - | (4)
      - | Log is transmitted to root since appender is not specified for the logger set in (2). Therefore, specify an appender which will act as output destination. Here, it will be output to "STDOUT" and "APPLICATION_LOG_FILE".
        | **[Location to be customized for each project]**


.. _exception-handling-how-to-use-application-configuration-domain-label:

Domain Layer Settings
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
When exceptions (BusinessException,ResourceNotFoundException) holding ResultMessages occur, add AOP settings and bean definition of interceptor class (\ ``ResultMessagesLoggingInterceptor``\ ) for log output.

- **xxx-domain.xml**

.. _exception-handling-how-to-use-service-pointcut-aop-label:

 .. code-block:: xml
    :emphasize-lines: 3,4,10

    <!-- interceptor bean. -->
    <bean id="resultMessagesLoggingInterceptor"
          class="org.terasoluna.gfw.common.exception.ResultMessagesLoggingInterceptor"> <!-- (1) -->
          <property name="exceptionLogger" ref="exceptionLogger" /> <!-- (2) -->
    </bean>

    <!-- setting AOP. -->
    <aop:config>
        <aop:advisor advice-ref="resultMessagesLoggingInterceptor"
                     pointcut="@within(org.springframework.stereotype.Service)" /> <!-- (3) -->
    </aop:config>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Add bean definition of \ ``ResultMessagesLoggingInterceptor``\ .
    * - | (2)
      - | Inject the logger which outputs exception log. Specify "exceptionLogger" defined in \ ``applicationContext.xml``\ .
    * - | (3)
      - | Apply ResultMessagesLoggingInterceptor for the method of the Service class (with \ ``@Service``\  annotation).


.. _exception-handling-how-to-use-application-configuration-app-label:

Application Layer Settings
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
Add to bean definition, the class (\ ``SystemExceptionResolver``\ )  used for handling the exceptions which are not handled by HandlerExceptionResolver registered automatically when <mvc:annotation-driven> is specified. .

- **spring-mvc.xml**

 .. code-block:: xml
    :emphasize-lines: 3-4,6-7,15,23-24,29

    <!-- Setting Exception Handling. -->
    <!-- Exception Resolver. -->
    <bean class="org.terasoluna.gfw.web.exception.SystemExceptionResolver"> <!-- (1) -->
        <property name="exceptionCodeResolver" ref="exceptionCodeResolver" /> <!-- (2) -->
        <!-- Setting and Customization by project. -->
        <property name="order" value="3" /> <!-- (3) -->
        <property name="exceptionMappings"> <!-- (4) -->
            <map>
                <entry key="ResourceNotFoundException" value="common/error/resourceNotFoundError" />
                <entry key="BusinessException" value="common/error/businessError" />
                <entry key="InvalidTransactionTokenException" value="common/error/transactionTokenError" />
                <entry key=".DataAccessException" value="common/error/dataAccessError" />
            </map>
        </property>
        <property name="statusCodes"> <!-- (5) -->
            <map>
                <entry key="common/error/resourceNotFoundError" value="404" />
                <entry key="common/error/businessError" value="409" />
                <entry key="common/error/transactionTokenError" value="409" />
                <entry key="common/error/dataAccessError" value="500" />
            </map>
        </property>
        <property name="defaultErrorView" value="common/error/systemError" /> <!-- (6) -->
        <property name="defaultStatusCode" value="500" /> <!-- (7) -->
    </bean>

    <!-- Settings View Resolver. -->
    <mvc:view-resolvers>
        <mvc:jsp prefix="/WEB-INF/views/" /> <!-- (8) -->
    </mvc:view-resolvers>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Add \ ``SystemExceptionResolver``\  to bean definition.
    * - | (2)
      - | Inject the object that resolves exception code (Message ID). Specify \ "exceptionCodeResolver"\  defined in \ ``applicationContext.xml``\ .
    * - | (3)
      - | Specify the order of priority for handling. The value can be "3". When \ ``<mvc:annotation-driven>``\  is specified, automatically \ :ref:`registered class <ExceptionHandling-annotation-driven>`\  is given higher priority.

        .. hint:: **Method to disable exception handling carried out by DefaultHandlerExceptionResolver**

            When exception handling is carried out by \ ``DefaultHandlerExceptionResolver``\ , HTTP response code is set; however since View is not resolved, it needs to be resolved using Error Page element of \ ``web.xml``\ .
            When it is required to resolve the View using \ ``HandlerExceptionResolver``\  and not in \ ``web.xml``\ , then priority order of \ ``SystemExceptionResolver``\  should be set to "1". By doing this, the handling process can be executed before \ ``DefaultHandlerExceptionResolver``\ .
            For mapping of HTTP response codes when handling is done by \ ``DefaultHandlerExceptionResolver``\ , refer to \ :ref:`exception-handling-appendix-defaulthandlerexceptionresolver-label`\ .

    * - | (4)
      - | Specify the mapping between name of the exception to be handled and View name.
        | In the above settings, if class name of the exception class (or parent class) includes ".DataAccessException", "common/error/dataAccessError" will be treated as View name.
        | If the exception class is "ResourceNotFoundException", "common/error/resourceNotFoundError" will be treated as View name.
        | **[Location to be customized for each project]**
    * - | (5)
      - | Specify the mapping between View name and HTTP status code.
        | In the above settings, when View name is "common/error/resourceNotFoundError", "404(Not Found)" becomes HTTP status code.
        | **[Location to be customized for each project]**
    * - | (6)
      - | Specify the default View name.
        | In the above settings, if exception class does not include "ResourceNotFoundException", "BusinessException" and "InvalidTransactionTokenException", and if exception class (or parent class) name does not include ".DataAccessException", "common/error/systemError" becomes the destination View name.
        | **[Location to be customized for each project]**
    * - | (7)
      - | Specify default value of HTTP status code to be set in response header. It is recommended that you set **"500"(Internal Server Error)**.

        .. warning:: **Behavior when nothing is specified**

            Please note that it will be handled as \ **"200"(OK)**\ .

    * - | (8)
      - Actual \ ``View``\  depends on \ ``ViewResolver``\  settings.

        In above settings, destination pages will be as given below.

        * ``/WEB-INF/views/common/error/systemError.jsp``
        * ``/WEB-INF/views/common/error/resourceNotFoundError.jsp``
        * ``/WEB-INF/views/common/error/businessError.jsp``
        * ``/WEB-INF/views/common/error/transactionTokenError.jsp``
        * ``/WEB-INF/views/common/error/dataAccessError.jsp``

        .. tip::

            \ ``<mvc:view-resolvers>``\  is an XML element added from Spring Framework 4.1.
            If \ ``<mvc:view-resolvers>``\  element is used, it is possible to define \ ``ViewResolver``\  in simple manner.

            Example of definition when \ ``<bean>``\  element is used in a conventional manner is given below.

             .. code-block:: xml

                <bean id="viewResolver"
                    class="org.springframework.web.servlet.view.InternalResourceViewResolver">
                    <property name="prefix" value="/WEB-INF/views/" />
                    <property name="suffix" value=".jsp" />
                </bean>

AOP settings and interceptor class (\ ``HandlerExceptionResolverLoggingInterceptor``\ ) in order to output the log of exceptions handled by \ ``HandlerExceptionResolver``\  should be added to bean definition.

- **spring-mvc.xml**

 .. code-block:: xml
    :emphasize-lines: 3,4,8

    <!-- Setting AOP. -->
    <bean id="handlerExceptionResolverLoggingInterceptor"
        class="org.terasoluna.gfw.web.exception.HandlerExceptionResolverLoggingInterceptor"> <!-- (1) -->
        <property name="exceptionLogger" ref="exceptionLogger" /> <!-- (2) -->
    </bean>
    <aop:config>
        <aop:advisor advice-ref="handlerExceptionResolverLoggingInterceptor"
            pointcut="execution(* org.springframework.web.servlet.HandlerExceptionResolver.resolveException(..))" /> <!-- (3) -->
    </aop:config>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Add \ ``HandlerExceptionResolverLoggingInterceptor``\  to bean definition.
    * - | (2)
      - | Inject the logger object which outputs exception log. Specify the "exceptionLogger" defined in \ ``applicationContext.xml``\ .
    * - | (3)
      - | Apply \ ``HandlerExceptionResolverLoggingInterceptor``\  to resolveException method of \ ``HandlerExceptionResolver``\  interface.
        |
        | As per default settings, this class will not output the log for common library provided ``org.terasoluna.gfw.common.exception.ResultMessagesNotificationException`` class and its subclasses.
        | The reason the exceptions of the sub class of ``ResultMessagesNotificationException`` are excluded from the log output is because their log output is carried out by ``org.terasoluna.gfw.common.exception.ResultMessagesLoggingInterceptor``.
        | If default settings need to be changed, refer to :ref:`exception-handling-about-handlerexceptionresolverlogginginterceptor`.

 Filter class (\ ``ExceptionLoggingFilter``\ ) used to output log of fatal errors and exceptions that are out of the boundary of Spring MVC should be added to bean definition and \ ``web.xml``\ .

- **applicationContext.xml**

 .. code-block:: xml
    :emphasize-lines: 3,4

    <!-- Filter. -->
    <bean id="exceptionLoggingFilter"
        class="org.terasoluna.gfw.web.exception.ExceptionLoggingFilter" > <!-- (1) -->
        <property name="exceptionLogger" ref="exceptionLogger" /> <!-- (2) -->
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Add \ ``ExceptionLoggingFilter``\  to bean definition.
    * - | (2)
      - | Inject the logger object which outputs exception log. Specify the "exceptionLogger" defined in \ ``applicationContext.xml``\ .


- **web.xml**

 .. code-block:: xml
    :emphasize-lines: 2,3,6,7

    <filter>
        <filter-name>exceptionLoggingFilter</filter-name> <!-- (1) -->
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class> <!-- (2) -->
    </filter>
    <filter-mapping>
        <filter-name>exceptionLoggingFilter</filter-name> <!-- (3) -->
        <url-pattern>/*</url-pattern> <!-- (4) -->
    </filter-mapping>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify the filter name. Match it with the bean name of \ ``ExceptionLoggingFilter``\  defined in \ ``applicationContext.xml``\ .
    * - | (2)
      - | Specify the filter class. The value should be fixed to \ ``org.springframework.web.filter.DelegatingFilterProxy``\ .
    * - | (3)
      - | Specify the name of the filter to be mapped. The value specified in (1).
    * - | (4)
      - | Specify the URL pattern to which the filter must be applied. It is recommended that you use \ ``/*``\  for outputting log of fatal errors and exceptions that are out of the boundary of Spring MVC.

- Output Log

 .. code-block:: guess

    date:2013-09-25 19:51:52	thread:tomcat-http--3	X-Track:f94de92148f1489b9ceeac3b2f17c969	level:ERROR	logger:o.t.gfw.common.exception.ExceptionLogger        	message:[e.xx.fw.9001] An exception occurred processing JSP page /WEB-INF/views/exampleJSPException.jsp at line 13


.. _exception-handling-how-to-use-application-configuration-container-label:

Servlet Container Settings
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
Add error-page definition for Servlet Container in order to handle error response (HttpServletResponse#sendError) received through default exception handling functionality of Spring MVC, fatal errors and the exceptions that are out of the boundary of Spring MVC.

- **web.xml**

 Add definitions in order to handle error response (HttpServletResponse#sendError) received through default exception handling functionality of Spring MVC.

 .. code-block:: xml

   <error-page>
       <!-- (1) -->
       <error-code>404</error-code>
       <!-- (2) -->
       <location>/WEB-INF/views/common/error/resourceNotFoundError.jsp</location>
   </error-page>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify the **HTTP Response Code** to be handled.
       | **[Location to be customized for each project]**
       | For HTTP response code for which response is sent using the default exception handling function of Spring MVC, refer to \ :ref:`exception-handling-appendix-defaulthandlerexceptionresolver-label`\ .
   * - | (2)
     - | Specify the file name. It should be specified with the path from Web application root. In the above settings, "${WebAppRoot}/WEB-INF/views/common/error/resourceNotFoundError.jsp" will be the View file.
       | **[Location to be customized for each project]**


 Add definitions in order to handle fatal errors and exceptions that are out of the boundary of Spring MVC.

 .. code-block:: xml

    <error-page>
        <!-- (3) -->
        <location>/WEB-INF/views/common/error/unhandledSystemError.html</location>
    </error-page>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (3)
     - | Specify the file name. Specify with a path from web application root. In the above settings, "${WebAppRoot}/WEB-INF/views/common/error/unhandledSystemError.html" will be the View file.
       | **[Location to be customized for each project]**

.. note:: **About the path specified in location tag**

  If a fatal error occurs, there is high possibility of getting another error if the path of dynamic contents is specified.; hence
  in location tag, **it is recommended that you specify a path of static contents such as HTML** and not dynamic contents such as JSP.

.. note:: **If untraceable error occurs during development**

  If an unexpected error response (HttpServletResponse#sendError) occurs after carrying out the above settings, there may be cases wherein it cannot be determined what kind of error response occurred.
  
  Error screen specified in location tag is displayed; however if the cause of error cannot be identified from logs,
  it is possible to verify the error response (HTTP response code) on screen by commenting out the above settings.


  When it is necessary to individually handle the exceptions that are out of the boundary of Spring MVC, definition of each exception should be added.

    .. code-block:: xml

      <error-page>
          <!-- (4) -->
          <exception-type>java.io.IOException</exception-type>
          <!-- (5) -->
          <location>/WEB-INF/views/common/error/systemError.jsp</location>
      </error-page>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - Sr. No.
        - Description
      * - | (4)
        - | Specify the **Exception Class Name (FQCN)** to be handled.
      * - | (5)
        - | Specify the file name. Specify it using a path from web application root. In above case, "${WebAppRoot}/WEB-INF/views/common/error/systemError.jsp" will be the View file.
          | **[Location to be customized for each project]**


.. _exception-handling-how-to-use-codingpoint-service-label:

Coding Points (Service)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The coding points in Service when handling the exceptions are given below.

#. :ref:`exception-handling-how-to-use-codingpoint-service-business-label`
#. :ref:`exception-handling-how-to-use-codingpoint-service-system-label`
#. :ref:`exception-handling-how-to-use-codingpoint-service-continue-label`


.. _exception-handling-how-to-use-codingpoint-service-business-label:

Generating Business Exception
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
The method of generating Business Exception is given below.

.. note:: **Notes related to the method of generating business exception**

  - | It is recommended that you generate the business exception by detecting violation of business rules in logic.
  - | When it is required by the API specification of underlying base framework or existing layer of application, that violation of business rule be notified through an exception, then it is OK to catch the exception and throw it as business exception.
    | Use of an exception to control the processing flow lowers the readability of overall logic and thus may reduce maintainability.

| Generate business exceptions by detecting violation of business rules in logic.

.. warning::

  - It is assumed that business exception is generated in Service by default. In \ :ref:`AOP settings <exception-handling-how-to-use-service-pointcut-aop-label>`\ ,
    log of business exception occurred in class with \ ``@Service``\  annotation, is being output.
    Business exceptions should not be logged in Controller etc. This rule can be changed if needed in the project.

- xxxService.java

 .. code-block:: java

    ...
    @Service
    public class ExampleExceptionServiceImpl implements ExampleExceptionService {
        @Override
        public String throwBusinessException(String test) {
            ...
            // int stockQuantity = 5;
            // int orderQuantity = 6;

            if (stockQuantity < orderQuantity) {                  // (1)
                ResultMessages messages = ResultMessages.error(); // (2)
                messages.add("e.ad.od.5001", stockQuantity);      // (3)
                throw new BusinessException(messages);            // (4)
            }
            ...

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Check whether there is any violation of a business rule.
    * - | (2)
      - | If there is a violation, generate ResultMessages. In the above example, ResultMessages of error level are being generated.
        | For the details on method of generating ResultMessages, refer to \ :doc:`MessageManagement`\ .
    * - | (3)
      - | Add ResultMessage to ResultMessages. Specify message ID as 1st argument (mandatory) and value to be inserted in message as 2nd argument (optional).
        | The value to be inserted in message is a variable-length parameter; hence multiple values can be specified.
    * - | (4)
      - | Specify ResultMessages and generate BusinessException.


 .. tip::

    For the purpose of explanation, ``xxxService.java`` logic is written in steps (2)-(4) as shown above, however it can also be written in a single step.

     .. code-block:: java

        throw new BusinessException(ResultMessages.error().add(
             "e.ad.od.5001", stockQuantity));


- xxx.properties

  Add the below settings to the properties file.

  .. code-block:: properties

    e.ad.od.5001 = Order number is higher than the stock quantity={0}. Change the order number.

An application log as shown below is output.

 .. code-block:: console

    date:2013-09-17 22:25:55	thread:tomcat-http--8	X-Track:6cfb0b378c124b918e40ac0c32a1fac7	level:WARN 	logger:o.t.gfw.common.exception.ExceptionLogger        	message:[e.xx.fw.8001] ResultMessages [type=error, list=[ResultMessage [code=e.ad.od.5001, args=[5], text=null]]]
    org.terasoluna.gfw.common.exception.BusinessException: ResultMessages [type=error, list=[ResultMessage [code=e.ad.od.5001, args=[5], text=null]]]

    // stackTrace omitted
    ...

    date:2013-09-17 22:25:55	thread:tomcat-http--8	X-Track:6cfb0b378c124b918e40ac0c32a1fac7	level:DEBUG	logger:o.t.gfw.web.exception.SystemExceptionResolver   	message:Resolving exception from handler [public java.lang.String org.terasoluna.exception.app.example.ExampleExceptionController.home(java.util.Locale,org.springframework.ui.Model)]: org.terasoluna.gfw.common.exception.BusinessException: ResultMessages [type=error, list=[ResultMessage [code=e.ad.od.5001, args=[5], text=null]]]
    date:2013-09-17 22:25:55	thread:tomcat-http--8	X-Track:6cfb0b378c124b918e40ac0c32a1fac7	level:DEBUG	logger:o.t.gfw.web.exception.SystemExceptionResolver   	message:Resolving to view 'common/error/businessError' for exception of type [org.terasoluna.gfw.common.exception.BusinessException], based on exception mapping [BusinessException]
    date:2013-09-17 22:25:55	thread:tomcat-http--8	X-Track:6cfb0b378c124b918e40ac0c32a1fac7	level:DEBUG	logger:o.t.gfw.web.exception.SystemExceptionResolver   	message:Applying HTTP status code 409
    date:2013-09-17 22:25:55	thread:tomcat-http--8	X-Track:6cfb0b378c124b918e40ac0c32a1fac7	level:DEBUG	logger:o.t.gfw.web.exception.SystemExceptionResolver   	message:Exposing Exception as model attribute 'exception'

Displayed screen

 .. figure:: ./images/exception-handling-screen-businessexception.png
    :alt: screen business exception
    :width: 50%

 .. warning::
    It is recommended that you handle business exception in Controller and display a message on each business screen.
    The above example illustrates a screen which is displayed when the exception is not handled in Controller.


Catch an exception to generate a business exception

 .. code-block:: java

    try {
        order(orderQuantity, itemId );
    } catch (StockNotEnoughException e) {                  // (1)
        throw new BusinessException(ResultMessages.error().add(
                "e.ad.od.5001", e.getStockQuantity()), e); // (2)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Catch the exception that occurs when a business rule is violated.
    * - | (2)
      - | Specify ResultMessages and \ **Cause Exception (e)**\  to generate BusinessException.


.. _exception-handling-how-to-use-codingpoint-service-system-label:

Generating System Exception
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
The method of generating SystemException is given below.


Generate a system exception by detecting a system error in logic.

 .. code-block:: java

    if (itemEntity == null) {                                      // (1)
        throw new SystemException("e.ad.od.9012",
            "not found item entity. item code [" + itemId + "]."); // (2)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Check whether the system is in normal state.
        | In this example, it is checked whether the requested product code (itemId) exists in the product master (Item Master).
        | If it does not exist in the product master, it would be considered that a resource which should have been available in the system, does not exist and hence it is treated as system error.
    * - | (2)
      - | If the system is in an abnormal state, specify the exception code (message ID) as 1st argument. Specify exception message as 2nd argument to generate SystemException.
        | In the above example, the value of variable "itemId" is inserted in message text.

Application log as shown below is output.

 .. code-block:: console

  date:2013-09-19 21:03:06	thread:tomcat-http--3	X-Track:c19eec546b054d54a13658f94292b24f	level:DEBUG	logger:o.t.gfw.web.exception.SystemExceptionResolver   	message:Resolving exception from handler [public java.lang.String org.terasoluna.exception.app.example.ExampleExceptionController.home(java.util.Locale,org.springframework.ui.Model)]: org.terasoluna.gfw.common.exception.SystemException: not found item entity. item code [10-123456].
  date:2013-09-19 21:03:06	thread:tomcat-http--3	X-Track:c19eec546b054d54a13658f94292b24f	level:DEBUG	logger:o.t.gfw.web.exception.SystemExceptionResolver   	message:Resolving to default view 'common/error/systemError' for exception of type [org.terasoluna.gfw.common.exception.SystemException]
  date:2013-09-19 21:03:06	thread:tomcat-http--3	X-Track:c19eec546b054d54a13658f94292b24f	level:DEBUG	logger:o.t.gfw.web.exception.SystemExceptionResolver   	message:Applying HTTP status code 500
  date:2013-09-19 21:03:06	thread:tomcat-http--3	X-Track:c19eec546b054d54a13658f94292b24f	level:DEBUG	logger:o.t.gfw.web.exception.SystemExceptionResolver   	message:Exposing Exception as model attribute 'exception'
  date:2013-09-19 21:03:06	thread:tomcat-http--3	X-Track:c19eec546b054d54a13658f94292b24f	level:ERROR	logger:o.t.gfw.common.exception.ExceptionLogger        	message:[e.ad.od.9012] not found item entity. item code [10-123456].
  org.terasoluna.gfw.common.exception.SystemException: not found item entity. item code [10-123456].
  	at org.terasoluna.exception.domain.service.ExampleExceptionServiceImpl.throwSystemException(ExampleExceptionServiceImpl.java:14) ~[ExampleExceptionServiceImpl.class:na]
  ...
  // stackTrace omitted

Displayed screen

 .. figure:: ./images/exception-handling-screen-systemexception.png
   :alt: screen system exception
   :width: 60%

 .. note::

    It is recommended to have a common system error screen rather than creating multiple screens for system errors.

    The screen mentioned in this guideline displays a (business-wise) message ID for system errors and has a fixed message.
    This is because there is no need to inform the details of error to the operator and it is sufficient to only convey that the system error has occurred.
    Therefore, in order to enhance the response for inquiry against system errors, the Message ID which acts as a key for the message text is displayed on the screen,
    in order to make the analysis easier for the development side.
    Displayed error screens should be designed in accordance with the UI standards of each project.


Catch an exception to generate system exception.

 .. code-block:: java

    try {
        return new File(preUploadDir.getFile(), key);
    } catch (FileNotFoundException e) { // (1)
        throw new SystemException("e.ad.od.9007",
            "not found upload file. file is [" + preUploadDir.getDescription() + "]."
            e); // (2)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Catch the checked exception classified as system error.
    * - | (2)
      - | Specify the exception code (message ID), message, \ **Cause Exception (e)**\  to generate SystemException.


.. _exception-handling-how-to-use-codingpoint-service-continue-label:

Catch the exception to continue the execution
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| When it is necessary to catch the exception to continue the execution, the exception should be logged before continuing the execution.

| When fetching customer interaction history from external system fails, the process of fetching information other than customer history can still be continued. This is illustrated in the following example.
| In this example, although fetching of customer history fails, business process does not have to be stopped and hence the execution continues.

 .. code-block:: java

    @Inject
    ExceptionLogger exceptionLogger; // (1)

    // ...

 .. code-block:: java

    InteractionHistory interactionHistory = null;
    try {
        interactionHistory = restTemplete.getForObject(uri, InteractionHistory.class, customerId);
    } catch (RestClientException e) { // (2)
        exceptionLogger.log(e); // (3)
    }

    // (4)
    Customer customer = customerRepository.findOne(customerId);

    // ...

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Inject the object of \ ``org.terasoluna.gfw.common.exception.ExceptionLogger``\  provided by the common library for log output.
    * - | (2)
      - | Catch the exception to be handled.
    * - | (3)
      - | Output the handled exception to log. In the example, log method is being called; however if the output level is known in advance 
        | and if there is no possibility of any change in output level, it is ok to call info, warn, error methods directly.
    * - | (4)
      - | Continue the execution just by outputting the log in (3).


An application log as shown below is output.

 .. code-block:: console

  date:2013-09-19 21:31:47	thread:tomcat-http--3	X-Track:df5271ece2304b12a2c59ff494806397	level:ERROR	logger:o.t.gfw.common.exception.ExceptionLogger        	message:[e.xx.fw.9001] Test example exception
  org.springframework.web.client.RestClientException: Test example exception
  ...
  // stackTrace omitted

 .. warning::

   When log() is used in \ ``exceptionLogger``\ , since it will be output at error level; by default, it will be output in monitoring log also.

 .. code-block:: console

      date:2013-09-19 21:31:47	X-Track:df5271ece2304b12a2c59ff494806397	level:ERROR	message:[e.xx.fw.9001] Test example exception

As shown in following example, if there is no problem in continuing the execution, and if monitoring log is being monitored through application monitoring, it should be set to a level such that it will not get monitored at log output level or defined such that it does not get monitored from the log content (log message).

 .. code-block:: java

      } catch (RestClientException e) {
          exceptionLogger.info(e);
      }

 | As per default settings, monitoring log other than error level will not be output. It is output in application log as follows:

 .. code-block:: console

    date:2013-09-19 22:17:53	thread:tomcat-http--3	X-Track:999725b111b4445b8d10b4ea44639c61	level:INFO 	logger:o.t.gfw.common.exception.ExceptionLogger        	message:[e.xx.fw.9001] Test example exception
    org.springframework.web.client.RestClientException: Test example exception


.. _exception-handling-how-to-use-codingpoint-controller-label:

Coding Points (Controller)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The coding points in Controller while handling the exceptions are given below.

#. :ref:`exception-handling-how-to-use-codingpoint-controller-request-label`
#. :ref:`exception-handling-how-to-use-codingpoint-controller-usecase-label`


.. _exception-handling-how-to-use-codingpoint-controller-request-label:

Method to handle exceptions at request level
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| Handle the exception at request level and set the message related information to Model.
| Then, by calling the method for displaying the View, generate the model required by the View and determine the View name.

 .. code-block:: java

    @RequestMapping(value = "change", method = RequestMethod.POST)
    public String change(@Validated UserForm userForm,
                         BindingResult result,
                         RedirectAttributes redirectAttributes,
                         Model model) {         // (1)

        // omitted

        User user = userHelper.convertToUser(userForm);
        try {
            userService.change(user);
        } catch (BusinessException e) {                                   // (2)
            model.addAttribute(e.getResultMessages());                    // (3)
            return viewChangeForm(user.getUserId(), model);               // (4)
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
      - | As of the parameters of the method, define Model in argument as an object which will be used to link the error information with View.
    * - | (2)
      - | Exceptions which need to be handled should be caught in application code.
    * - | (3)
      - | Add ResultMessages object to Model.
    * - | (4)
      - | Call the method for displaying the View at the time of error. Fetch the model and View name required for View display and then return the View name to be displayed.


.. _exception-handling-how-to-use-codingpoint-controller-usecase-label:

Method to handle exception at use case level
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| Handle the exception at use case level and generate ModelMap (ExtendedModelMap) wherein message related information etc. is stored.
| Then, by calling the method for displaying the View, generate the model required by the View and determine the View name.

 .. code-block:: java

    @ExceptionHandler(BusinessException.class) // (1)
    @ResponseStatus(HttpStatus.CONFLICT) // (2)
    public ModelAndView handleBusinessException(BusinessException e) {
        ExtendedModelMap modelMap = new ExtendedModelMap();                 // (3)
        modelMap.addAttribute(e.getResultMessages());                       // (4)
        String viewName = top(modelMap);                                    // (5)
        return new ModelAndView(viewName, modelMap);                        // (6)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | The exception class which need to be handled should be specified in the value attribute of \ ``@ExceptionHandler``\  annotation. You can also specify multiple exceptions which are in the scope of handling.
    * - | (2)
      - | Specify the HTTP status code to be returned to value attribute of \ ``@ResponseStatus``\  annotation. In the example, "409: Conflict" is specified.
    * - | (3)
      - | Generate ExtendedModelMap as an object to link the error information and model information with View.
    * - | (4)
      - | Add ResultMessages object to ExtendedModelMap.
    * - | (5)
      - | Call the method to display the View at the time of error and fetch model and View name necessary for View display.
    * - | (6)
      - | Generate ModelAndView wherein View name and Model acquired in steps (3)-(5) are stored and then return the same.


.. _exception-handling-how-to-use-codingpoint-jsp-label:

Coding points (JSP)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The coding points in JSP while handling the exceptions are given below.

#. :ref:`exception-handling-how-to-use-codingpoint-jsp-panel-label`
#. :ref:`exception-handling-how-to-use-codingpoint-jsp-exceptioncode-label`

.. tip::

    When Internet Explorer is a support browser, 
    implementation should be such that size of the HTML responded as error screen should be 513 bytes or more.

    This is because in case of Internet Explorer, 
    if the following three conditions such as
    
    * Response status code is Error (4xx and 5xx)
    * Response HTML is 512 bytes or less
    * Browser setting "Show Friendly HTTP Error Messages" is valid
    
    are satisfied, the mechanism is such that friendly messages created by Internet Explorer will be displayed.

.. _exception-handling-how-to-use-codingpoint-jsp-panel-label:

Method to display messages on screen using MessagesPanelTag
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
The example below illustrates implementation at the time of outputting ResultMessages to an arbitrary location.

 .. code-block:: xml

    <t:messagesPanel /> <!-- (1) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      -  <t:messagesPanel> tag should be specified at a location where the message is to be output. For details on usage of <t:messagesPanel> tag, refer to \ :doc:`MessageManagement`\ .


.. _exception-handling-how-to-use-codingpoint-jsp-exceptioncode-label:

Method to display system exception code on screen
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
The example below illustrates implementation at the time of displaying exception code (message ID) and fixed message at an arbitrary location.

 .. code-block:: xml

    <p>
        <c:if test="${!empty exceptionCode}">  <!-- (1) -->
            [${f:h(exceptionCode)}]            <!-- (2) -->
        </c:if>
        <spring:message code="e.cm.fw.9999" /> <!-- (3) -->
    </p>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Check whether exception code (message ID) exists. Perform existence check when the exception code (message ID) is enclosed with symbols as shown in the above example.
    * - | (2)
      - | Output exception code (message ID).
    * - | (3)
      - | Output fixed message acquired from message definition.

- Output Screen (With exceptionCode)

 .. figure:: ./images/exception-handling-screen-systemexception-messagecode.png
   :alt: screen system exception messagecode
   :width: 40%

- Output Screen (Without exceptionCode)

 .. figure:: ./images/exception-handling-screen-systemexceptionbyjspexception.png
   :alt: screen system exception no messagecode
   :width: 40%

 .. note:: **Messages to be output at the time of system exception**

    - When a system exception occurs, it is recommended that you display a message which would only convey that a system exception has occurred, without outputting a detailed message from which cause of error can be identified or guessed.
    - On displaying a detailed message from which cause of error can be identified or guessed, the vulnerabilities of system are likely to get exposed.

 .. note:: **Exception code (message ID)**

    - When a system exception occurs, it is desirable to output an exception code (message ID) instead of a detailed message.
    - By outputting the exception code (message ID), it is possible for development team to respond quickly to the inquiries from system user.
    - Only a system administrator can identify the cause of error from exception code (message ID); hence the risk of exposing the vulnerabilities of system is lowered.

|

How to use (Ajax)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For the exception handling of Ajax, refer to \ :doc:`Ajax`\ .

|

Appendix
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. :ref:`exception-handling-about-classes-of-library-label`
#. :ref:`exception-handling-about-systemexceptionresolver-label`
#. :ref:`exception-handling-appendix-defaulthandlerexceptionresolver-label`

|

.. _exception-handling-about-classes-of-library-label:

Exception handling classes provided by the common library
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| In addition to the classes provided by Spring MVC, the classes for carrying out exception handling are being provided by the common library.
| The roles of classes are as follows:

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.65\linewidth}|
.. list-table:: **Table - Classes under org.terasoluna.gfw.common.exception package**
   :header-rows: 1
   :widths: 10 20 65

   * - Sr. No.
     - Class
     - Role
   * - | (1)
     - | ExceptionCode
       | Resolver
     - | An interface for resolving exception code (message ID) of exception class.
       | An exception code is used for identifying the exception type and is expected to be output to system error screen and log.
       | It is referenced from \ ``ExceptionLogger``\ , \ ``SystemExceptionResolver``\ . 
   * - | (2)
     - | SimpleMapping
       | ExceptionCode
       | Resolver
     - | Implementation class of \ ``ExceptionCodeResolver``\  contains the mapping of exception class name and exception code, through which the exception code is resolved.
       | The name of exception class need not be FQCN, it can be a part of FQCN or parent class name.

        .. warning::

            - Note that when a part of FQCN is specified, it may be matched with classes which were not assumed.
            - Note that when the name of parent class is specified, all the child classes will also be matched.

   * - | (3)
     - | enums.
       | ExceptionLevel
     - | enum indicating exception level corresponding to exception class.
       | INFO, WARN, ERROR are defined.
   * - | (4)
     - | ExceptionLevel
       | Resolver
     - | Interface for resolving exception level (log level) of exception class.
       | Exception level is a code for identifying the level of exception and is used to switch the output level of log.
       | It is referenced from ``ExceptionLogger``.
   * - | (5)
     - | DefaultException
       | LevelResolver
     - | This is an implementation class of ``ExceptionLevelResolver`` which resolves the exception level by first character of exception code.
       | If the first character (case insensitive) is,
       |   1. "i", treat it as ``ExceptionLevel.INFO``
       |   2. "w", treat it as ``ExceptionLevel.WARN``
       |   3. "e", treat it as ``ExceptionLevel.ERROR``
       |   4. a character other than the above, treat as ``ExceptionLevel.ERROR`` level.
       | This class is implemented in accordance with the rules of the message ID described in \ :doc:`Message <MessageManagement>`\  guidelines.
   * - | (6)
     - | ExceptionLogger
     - | A class to output log of exceptions.
       | Monitoring log (only messages) and application log (both messages and stack trace) can be output.
       | This class is used from Filter or Interceptor classes provided by the framework.
       | When the execution is to be continued by handling the exceptions in application code, log should be output by using this class.
   * - | (7)
     - | ResultMessages
       | LoggingInterceptor
     - | This is an interceptor class for logging an occurrence of exception which stores \ ``ResultMessages``\  (sub exception of \ ``ResultMessagesNotificationException``\ ).
       | Output all the logs at WARN level.
       | It is assumed that this Interceptor will be applied to the methods of class with ``@Service`` annotation.
       | Log is output using \ ``ExceptionLogger``\ .
   * - | (8)
     - | BusinessException
     - | This is an exception class to notify violation of business rules. This exception should be generated in the logic of domain layer.
       | It inherits \ ``java.lang.RuntimeException``\ ; hence the transactions are rolled back as per default behavior.
       | If you want to commit the transactions, this exception class needs to be specified in noRollbackFor or noRollbackForClassName of \ ``@Transactional``\  annotation.
   * - | (9)
     - | Resource
       | NotFoundException
     - | This is an exception class to notify that the specified resource (data) does not exist in the system. This exception should mainly be generated in the logic of domain layer.
       | It inherits \ ``java.lang.RuntimeException``\ ; hence the transactions are rolled back as per default behavior.
   * - | (10)
     - | ResultMessages
       | Notification
       | Exception
     - | This is an abstract exception class to notify that the exception holds the result messages (\ ``ResultMessages``\ ), and \ ``BusinessException``\  and \ ``ResourceNotFoundException``\  inherit it in the common library.
       | It inherits \ ``java.lang.RuntimeException``\ ; hence the transactions are rolled back as per default behavior.
       | If this exception class is inherited, log of warn level will be output by \ ``ResultMessagesLoggingInterceptor``\ .
   * - | (11)
     - | SystemException
     - | This is an exception class to notify a system or application error. This exception should be generated in the logic of application layer or domain layer.
       | It inherits \ ``java.lang.RuntimeException``\ ; hence the transactions are rolled back as per default behavior.
   * - | (12)
     - | ExceptionCodeProvider
     - | This is an interface indicating that it has a role to hold the exception code; it is implemented by \ ``SystemException``\  in the common library.
       | If an exception class that implements this interface is created, it can be used with the exception handling mechanism of ExceptionCodeResolver, to use the exception code of this exception class as it is.


.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.65\linewidth}|
.. list-table:: **Table - Classes under org.terasoluna.gfw.web.exception package**
   :header-rows: 1
   :widths: 10 20 65

   * - Sr. No.
     - Class
     - Role
   * - | (13)
     - | SystemException
       | Resolver
     - | This is a class for handling the exceptions that will not be handled by \ ``HandlerExceptionResolver``\ , which is registered automatically when \ ``<mvc:annotation-driven>``\  is specified.
       | It inherits \ ``SimpleMappingExceptionResolver``\  provided by Spring MVC and adds the functionality of referencing ResultMessages of exception code from View.
   * - | (14)
     - | HandlerException
       | ResolverLogging
       | Interceptor
     - | This is an Interceptor class to output the log of exceptions handled by \ ``HandlerExceptionResolver``\ .
       | In this Interceptor class, the output level of log is switched based on the classification of HTTP response code resolved by \ ``HandlerExceptionResolver``\ .
       |   1. When it is "100-399", log is output at INFO level.
       |   2. When it is "400-499", log is output at WARN level.
       |   3. When it is "500-", log is output at ERROR level.
       |   4. When it is "-99", log is not output.
       | By using this Interceptor, it is possible to output the log of all exceptions which are within the boundary of Spring MVC.
       | Log is output using \ ``ExceptionLogger``\ .
   * - | (15)
     - | ExceptionLogging
       | Filter
     - | This is a Filter class to output the log of fatal errors and exceptions that are out of the boundary of Spring MVC.
       | All logs are output at ERROR level.
       | When this Filter is used, fatal errors and all exceptions that are out of the boundary of Spring MVC can be output to log.
       | Log is output using \ ``ExceptionLogger``\ .


.. _exception-handling-about-systemexceptionresolver-label:

About SystemExceptionResolver settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
This section describes the settings which are not explained above.
The settings should be performed depending on the requirements.

.. tabularcolumns:: |p{0.05\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|p{0.45\linewidth}|p{0.20\linewidth}|
.. list-table:: **List of settings not explained above**
   :header-rows: 1
   :widths: 5 15 15 45 20

   * - Sr. No.
     - Field Name
     - Property Name
     - Description
     - Default Value
   * - | (1)
     - | Attribute name of result message
     - | resultMessagesAttribute
     - | Specify the attribute name (String) used for setting message of businessException to Model.
       | This attribute name is used for accessing result message in View(JSP).
     - resultMessages
   * - | (2)
     - | Attribute name of exception code (message ID)
     - | exceptionCode
       | Attribute
     - | Specify the attribute name (String) used for setting the exception code (message ID) to HttpServletRequest.
       | This attribute name is used for accessing the exception code (message ID) in View(JSP).
     - exceptionCode
   * - | (3)
     - | Header name of exception code (message ID)
     - | exceptionCode
       | Header
     - | Specify the header name (String) used for setting the exception code (message ID) to response header of HttpServletResponse.
     - X-Exception-Code
   * - | (4)
     - | Attribute name of exception object
     - | exceptionAttribute
     - | Specify the attribute name (String) used for setting the handled exception object to Model.
       | This attribute name is used for accessing the exception object in View(JSP).
     - exception
   * - | (5)
     - | List of handler (Controller) objects to be used as this ExceptionResolver
     - | mappedHandlers
     - | Specify the object list (Set) of handlers that use this ExceptionResolver.
       | Only the exceptions in the specified handler objects will be handled.
       | **This setting should not be specified.**
     - | Not specified
       |
       | **When specified, the operation is not guaranteed.**
   * - | (6)
     - | List of handler (Controller) classes that use this ExceptionResolver
     - | mappedHandlerClasses
     - | Specify the class list (Class[]) of handlers that use this ExceptionResolver.
       | Only the exceptions that occur in the specified handler classes are handled.
       | **This setting should not be specified**
     - | Not specified
       |
       | **When specified, the operation is not guaranteed.**
   * - | (7)
     - | Cache control flag of HTTP response
     - | preventResponseCaching
     - | Set the flag (true: Yes, false: No) for cache control at the time of HTTP response.
       | If true: Yes is specified, HTTP response header to disable cache is added.
     - | false: No

| (1)-(3) are the settings of \ ``org.terasoluna.gfw.web.exception.SystemExceptionResolver``\ .
| (4) is the setting of \ ``org.springframework.web.servlet.handler.SimpleMappingExceptionResolver``\ .
| (5)-(7) are the settings of \ ``org.springframework.web.servlet.handler.AbstractHandlerExceptionResolver``\ .


Attribute name of result message
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| If the message set by handling the exception using SystemExceptionResolver and the message set by handling the exception in application code, both are to be output in separate messagesPanel tags in View(JSP), then specify an attribute name exclusive to SystemExceptionResolver.
| The example below illustrates settings and implementation when changing the default value to "resultMessagesForExceptionResolver".

- **spring-mvc.xml**

  .. code-block:: xml

    <bean class="org.terasoluna.gfw.web.exception.SystemExceptionResolver">

        <!-- omitted -->

        <property name="resultMessagesAttribute" value="resultMessagesForExceptionResolver" /> <!-- (1) -->

        <!-- omitted -->
    </bean>

- **jsp**

  .. code-block:: xml

    <t:messagesPanel messagesAttributeName="resultMessagesForExceptionResolver"/> <!-- (2) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify "resultMessagesForExceptionResolver" in result message attribute name (resultMessagesAttribute).
    * - | (2)
      - | Specify attribute name that was set in SystemExceptionResolver, in message attribute name (messagesAttributeName).


Attribute name of exception code (message ID)
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| When default attribute name is being used in the application code, a different value should be set in order to avoid duplication. When there is no duplication, there is no need to change the default value.
| The example below illustrates settings and implementation when changing the default value to "exceptionCodeForExceptionResolver".

- **spring-mvc.xml**

  .. code-block:: xml

    <bean class="org.terasoluna.gfw.web.exception.SystemExceptionResolver">

        <!-- omitted -->

        <property name="exceptionCodeAttribute" value="exceptionCodeForExceptionResolver" /> <!-- (1) -->

        <!-- omitted -->
    </bean>

- **jsp**

  .. code-block:: xml

    <p>
        <c:if test="${!empty exceptionCodeForExceptionResolver}">  <!-- (2) -->
            [${f:h(exceptionCodeForExceptionResolver)}]            <!-- (3) -->
        </c:if>
        <spring:message code="e.cm.fw.9999" />
    </p>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify "exceptionCodeForExceptionResolver" in attribute name (exceptionCodeAttribute) of exception code (message ID).
    * - | (2)
      - | Specify the value (exceptionCodeForExceptionResolver) set in SystemExceptionResolver as a variable name (for Empty check) to be tested.
    * - | (3)
      - | Specify the value (exceptionCodeForExceptionResolver) set in SystemExceptionResolver as a variable name to be output.


Header name of exception code (message ID)
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| When default header name is being used, set a different value in order to avoid duplication. When there is no duplication, there is no need to change the default value.
| The example below illustrates settings and implementation when changing the default value to "X-Exception-Code-ForExceptionResolver".

- **spring-mvc.xml**

  .. code-block:: xml

    <bean class="org.terasoluna.gfw.web.exception.SystemExceptionResolver">

        <!-- omitted -->

        <property name="exceptionCodeHeader" value="X-Exception-Code-ForExceptionResolver" /> <!-- (1) -->

        <!-- omitted -->
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify "X-Exception-Code-ForExceptionResolver" in the header name (exceptionCodeHeader) of the exception code (message ID).


Attribute name of exception object
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| When default attribute name is being used in the application code, set a different value in order to avoid duplication. When there is no duplication, there is no need to change the default value.
| The example below illustrates settings and implementation when changing the default value to "exceptionForExceptionResolver".

- **spring-mvc.xml**

  .. code-block:: xml

    <bean class="org.terasoluna.gfw.web.exception.SystemExceptionResolver">

        <!-- omitted -->

        <property name="exceptionAttribute" value="exceptionForExceptionResolver" /> <!-- (1) -->

        <!-- omitted -->
    </bean>

- **jsp**

  .. code-block:: xml

    <p>[Exception Message]</p>
    <p>${f:h(exceptionForExceptionResolver.message)}</p> <!-- (2) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify "exceptionForExceptionResolver" in attribute name (exceptionAttribute) of exception object.
    * - | (2)
      - | Specify the value (exceptionForExceptionResolver) set in SystemExceptionResolver as a variable name for fetching message from exception object.


Cache control flag of HTTP response
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| When header for cache control is to be added to HTTP response, specify true: Yes.

- **spring-mvc.xml**

  .. code-block:: xml

    <bean class="org.terasoluna.gfw.web.exception.SystemExceptionResolver">

        <!-- omitted -->

        <property name="preventResponseCaching" value="true" /> <!-- (1) -->

        <!-- omitted -->
    </bean>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Set the cache control flag (preventResponseCaching) of HTTP response to true: Yes.

 .. note:: **HTTP response header when Yes is specified**

    If cache control flag of HTTP response is set to Yes, the following HTTP response header is output.

     | Cache-Control:no-store
     | Cache-Control:no-cache
     | Expires:Thu, 01 Jan 1970 00:00:00 GMT
     | Pragma:no-cache


.. _exception-handling-about-handlerexceptionresolverlogginginterceptor:

About HandlerExceptionResolverLoggingInterceptor settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
This section describes the settings which are not explained above.
The settings should be performed depending on the requirements.

.. tabularcolumns:: |p{0.05\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|p{0.45\linewidth}|p{0.20\linewidth}|
.. list-table:: **List of settings not described above**
   :header-rows: 1
   :widths: 5 15 15 45 20

   * - Sr. No.
     - Field Name
     - Property Name
     - Description
     - Default Value
   * - | (1)
     - | List of exception classes to be excluded from scope of logging
     - | ignoreExceptions
     - | Amongst the exceptions handled by ``HandlerExceptionResolver``, the exception classes which are not to be logged should be specified in list format.
       | When an exception of the specified exception class and sub class occurs, the same is not logged in this class.
       | Only the exceptions which are logged at a different location (using different mechanism) should be specified here.
     - | ``ResultMessagesNotificationException.class``
       |
       | Exceptions of ``ResultMessagesNotificationException.class`` and its sub classes are logged by ``ResultMessagesLoggingInterceptor``; hence, as default settings, they are being excluded.

List of exception classes to be excluded from scope of logging
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
The settings when exception classes provided in the project are to be excluded from the scope of logging, are as follows:

- **spring-mvc.xml**

 .. code-block:: xml

    <bean id="handlerExceptionResolverLoggingInterceptor"
        class="org.terasoluna.gfw.web.exception.HandlerExceptionResolverLoggingInterceptor">
        <property name="exceptionLogger" ref="exceptionLogger" />
        <property name="ignoreExceptions">
            <set>
                <!-- (1) -->
                <value>org.terasoluna.gfw.common.exception.ResultMessagesNotificationException</value>
                <!-- (2) -->
                <value>com.example.common.XxxException</value>
            </set>
        </property>
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | ``ResultMessagesNotificationException`` specified in the default settings of common library, should be specified under exception to be excluded.
    * - | (2)
      - |  Specify the exception classes created in the project.

|

The settings when all the exception classes are to be logged are as follows:

- **spring-mvc.xml**

 .. code-block:: xml

    <bean id="handlerExceptionResolverLoggingInterceptor"
        class="org.terasoluna.gfw.web.exception.HandlerExceptionResolverLoggingInterceptor">
        <property name="exceptionLogger" ref="exceptionLogger" />
        <!-- (3) -->
        <property name="ignoreExceptions"><null /></property>
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (3)
      - | Specify ``null`` in ignoreExceptions properties.
        | If ``null`` is specified, all the exception classes will become target for logging.

.. _exception-handling-appendix-defaulthandlerexceptionresolver-label:

HTTP response code set by DefaultHandlerExceptionResolver
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The mapping between framework exceptions handled using DefaultHandlerExceptionResolver and HTTP status code is shown below.

.. tabularcolumns:: |p{0.10\linewidth}|p{0.60\linewidth}|p{0.20\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 60 20

   * - Sr. No.
     - Handled framework exceptions
     - HTTP Status Code
   * - | (1)
     - | org.springframework.web.servlet.mvc.multiaction.NoSuchRequestHandlingMethodException
     - | 404
   * - | (2)
     - | org.springframework.web.HttpRequestMethodNotSupportedException
     - | 405
   * - | (3)
     - | org.springframework.web.HttpMediaTypeNotSupportedException
     - | 415
   * - | (4)
     - | org.springframework.web.HttpMediaTypeNotAcceptableException
     - | 406
   * - | (5)
     - | org.springframework.web.bind.MissingServletRequestParameterException
     - | 400
   * - | (6)
     - | org.springframework.web.bind.ServletRequestBindingException
     - | 400
   * - | (7)
     - | org.springframework.beans.ConversionNotSupportedException
     - | 500
   * - | (8)
     - | org.springframework.beans.TypeMismatchException
     - | 400
   * - | (9)
     - | org.springframework.http.converter.HttpMessageNotReadableException
     - | 400
   * - | (10)
     - | org.springframework.http.converter.HttpMessageNotWritableException
     - | 500
   * - | (11).
     - | org.springframework.web.bind.MethodArgumentNotValidException
     - | 400
   * - | (12)
     - | org.springframework.web.multipart.support.MissingServletRequestPartException
     - | 400
   * - | (13)
     - | org.springframework.validation.BindException
     - | 400

.. raw:: latex

   \newpage

