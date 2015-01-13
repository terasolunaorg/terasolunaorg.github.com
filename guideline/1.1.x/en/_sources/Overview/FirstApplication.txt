First application based on Spring MVC 
--------------------------------------------------------------

.. only:: html

 .. contents:: Index
    :depth: 3
    :local:

Before entering into the advanced usage of Spring MVC, it is better to understand Spring MVC by actually trying handson web application development using Spring MVC.
This purpose of this chapter to understand the overall picture. **Note that it does not follow the recommendations given from the next chapter onwards. (for the ease of understanding)**.

Prerequisites
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The description of this chapter has been verified on the following environment. (For other environments, replace the contents mentioned here appropriately)

.. tabularcolumns:: |p{0.75\linewidth}|p{0.25\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 75 25

   * - Product
     - Version
   * - JDK
     - 1.6.0\_33
   * - Spring Tool Suite (STS)
     - 3.2.0
   * - VMware vFabric tc Server Developer Edition
     - 2.8
   * - Fire Fox
     - 21.0

.. note::

  To connect to the internet via a proxy server, 
  STS Proxy settings and \ `Maven Proxy settings <http://maven.apache.org/guides/mini/guide-proxies.html>`_\  are required for the following operations.


.. warning::

  "Spring Template Project" used in this chapter has been removed from Spiring Tool Suite 3.4; hence, the contents of this chapter can only be confirmed on "Spring Tool Suite 3.3" or before. 
  
  Refer to \ :doc:`../Appendix/CreateProjectFromBlank`\  in case using Spring Tool Suite 3.4.
  
  We plan to update the contents of this chapter in future.

Create a New Project
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
From the menu of SpringSource Tool Suite, select [File] -> [New] -> [Spring Template Project] -> [Spring MVC Project] -> [Next],
and create Spring MVC project.

.. note::

    If proxy server is being used, there is a possibility of not being able to select Spring Template Project.
    In order to resolve this, do the following.

    * First select [window] -> [Preferences] -> [Spring] -> [Template Projects] and remove all items except "spring-defaults".
    * Then click Apply.
    * After this, when [File] -> [New] -> [Spring Project] is clicked, [Spring MVC Project] can be selected from Templetes.

Enter "helloworld" in [Project Name], "com.example.helloworld" in [Please specify the top-level package] and click [Finish] on the next screen.

.. figure:: images/NewSpringMVCProject.png
   :alt: New Spring MVC Project
   :width: 60%

Following project is generated in the Package Explorer (**Internet access is required**)

.. figure:: images/HelloWorldWorkspace.png
   :alt: workspace

The generated configuration file (src/main/webapp/WEB-INF/spring/appServlet/servlet-context.xml) of Spring MVC is described briefly to understand the configuration of Spring MVC.

.. literalinclude:: ../../resources/helloworld/src/main/webapp/WEB-INF/spring/appServlet/servlet-context.xml
   :language: xml


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Item number
     - Description
   * - | (1)
     - Default settings of Spring MVC are configured by defining <annotation-driven />. Refer to the official website `Enabling the MVC Java Config or the MVC XML Namespace <http://docs.spring.io/spring/docs/4.1.3.RELEASE/spring-framework-reference/html/mvc.html#mvc-config-enable>`_  of Spring framework for default configuration.
   * - | (2)
     - Define the location of View by specifying Resolver of View.
   * - | (3)
     - Define the package which will be target of searching components used in Spring MVC.

Following is the ``com.example.helloworld.HomeController`` (However, it is modified to simple form for explanation).

.. literalinclude:: ../../resources/helloworld/src/main/java/com/example/helloworld/HomeController.java
   :language: java
   :emphasize-lines: 15,24,25,31

It is explained briefly. 

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Item number
     - Description
   * - | (1)
     - It can be read automatically by DI container if ``@Controller`` annotation is used. As stated earlier in "explanation of Spring MVC configuration files (3)", it is the target of component-scan.
   * - | (2)
     - It gets executed when the HTTP method is GET and when the Resource is (or request URL) is "/".
   * - | (3)
     - Set objects to be delivered to View.
   * - | (4)
     - Return View name. Rendering is performed by ``WEB-INF/views/home.jsp`` as per the configuration in "Explanation of Spring MVC configuration files (2)".

The object set in Model is set in HttpServletRequest.
The value passed from Controller can be output by mentioning ``${serverTime}`` in home.jsp as shown below.
( **However, as described below, it is necessary to perform HTML escaping since there is possibility of ${XXX} becoming a target of XSS** )

.. literalinclude:: ../../resources/helloworld/src/main/webapp/WEB-INF/views/home.jsp
   :language: jsp

|

Run on Server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Right click "helloworld" project in STS, and start helloworld project by executing "Run As" -> "Run On Server" -> "localhost" -> "VMware vFabric tc Server Developer Edition v2.8" -> "Finish".
Enter "http://localhost:8080/helloworld/" in browser to display the following screen.

.. figure:: images/AppHelloWorldIndex.png
   :alt: Hello World

|

.. _first-application-create-an-echo-application:

Create an Echo Application
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Lets go ahead and reate a simple application. It is a typical eco application in which message will be displayed if name is entered in the text field as given below.

.. figure:: images/AppEchoIndex.png
   :alt: Form of Echo Application

.. figure:: images/AppEchoHello.png
   :alt: Output of Echo Application

|

Creating a form object
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
First create a form object to accept the value of text field. Create ``EchoForm`` class in ``com.example.helloworld.echo`` package.
It is a simple JavaBean that has only 1 property.

.. literalinclude:: ../../resources/helloworld/src/main/java/com/example/helloworld/echo/EchoForm.java
   :language: java

|

Create a Controller
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Next, create the Controller class. create the ``EchoController`` class in "com.example.helloworld.echo" package.

.. literalinclude:: ../../resources/helloworld/src/main/java/com/example/helloworld/echo/EchoController.java
   :language: java
   :emphasize-lines: 12,18,20,23-25

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Item number
     - Description
   * - | (1)
     - | Add ``@ModelAttribute`` annotation to the method. Return value of such a method is automatically added to the Model.
       | Attribute name can be specified in ``@ModelAttribute``, but the class name with the first letter in lower case is the default attribute name. 
       | In this case it will be ``echoForm``. This attribute name must match with the value of ``modelAttribute`` of ``form:form`` tag.
   * - | (2)
     - | When nothing is specified in ``value`` attribute of ``@RequestMapping`` annotation at the method level, it is mapped to ``@RequestMapping`` added at class level. 
       | In this case, ``index`` method is called, if ``<contextPath>/echo`` is accessed. When nothing is set in ``method`` attribute, mapping is done for any HTTP method.
   * - | (3)
     - | EchoForm object added to the model in (1) is passed as argument.
   * - | (4)
     - | Since ``echo/index`` is returned as View name, ``WEB-INF/views/echo/index.jsp`` is rendered by ViewResolver.
   * - | (5)
     - | Since ``hello`` is specified in ``@RequestMapping`` annotation at method level, if ``<contextPath>/echo/hello`` is accessed, ``hello`` method is called.
   * - | (6)
     - | ``name`` entered in form is passed as it is to the View.


Create JSP Files
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Finally create JSP of input screen and output screen. Each file path must match with View name as follows.

* Input Screen src/main/webapp/WEB-INF/views/echo/index.jsp

.. literalinclude:: ../../resources/helloworld/src/main/webapp/WEB-INF/views/echo/index.jsp
   :language: jsp

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Item number
     - Description
   * - | (1)
     - | HTML form is constructed by using tag library. Specify the name of form object created by Controller in ``modelAttribute``.
       | Refer \ `here <http://docs.spring.io/spring/docs/4.1.3.RELEASE/spring-framework-reference/html/view.html#view-jsp-formtaglib-formtag>`_\ for tag library.

The generated HTML is as follows

.. code-block:: html

    <body>

      <form id="echoForm" action="/helloworld/echo/hello" method="post">
        <label for="name">Input Your Name:</label>
        <input id="name" name="name" type="text" value=""/>
        <input type="submit" />
      </form>
    </body>
    </html>

* Output Screen src/main/webapp/WEB-INF/views/echo/hello.jsp

.. literalinclude:: ../../resources/helloworld/src/main/webapp/WEB-INF/views/echo/hello.jsp
   :language: jsp

Output ``name`` passed from Controller. Take countermeasures for XSS  using ``c:out`` tag.  

|

.. note::

    Countermeasure for XSS are taken using ``c:out`` standard tag here. However, ``f:h()`` function that can be used easily is provided in common library.
    Refer to :doc:`../Security/XSS` for details.


Implementation of Eco application is completed here. Start the server, access the link ``http://localhost:8080/helloworld/echo``  to access the application.

|

Implement Input Validation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Input validation is not performed in this application till this point.
In Spring MVC, `Bean Validation <http://jcp.org/en/jsr/detail?id=349>`_\  and annotation based input validation can be easily implemented. For example input validation of name is performed in Eco Application.

Following dependency is added to pom.xml for using Bean Validation. 

.. literalinclude:: ../../resources/helloworld2/pom.xml
   :language: xml
   :lines: 112-117

In ``EchoForm``, add ``@NotNull`` annotation and ``@Size`` annotation to ``name`` property as follows (can be added to getter method).


.. literalinclude:: ../../resources/helloworld2/src/main/java/com/example/helloworld/echo/EchoForm.java
   :language: java
   :emphasize-lines: 5,6,11,12

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Item number
     - Description
   * - | (1)
     - | By adding ``@NotNull`` annotation, whether ``name`` parameter exists in HTTP request is checked.
   * - | (2)
     - | By adding ``@Size(min=1, max=5)`` annotation, whether the size of ``name`` is more than or equal to 1 and less than or equal to 5 is checked. 
     
In ``EchoController`` class, add ``@Valid`` annotation as shown below and also use of ``hasErrors`` method.

.. literalinclude:: ../../resources/helloworld2/src/main/java/com/example/helloworld/echo/EchoController.java
   :language: java
   :emphasize-lines: 3,7,27-30

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Item number
     - Description
   * - | (1)
     - | In Controller, add ``@Valid`` annotation to the argument on which validation needs to be executed. Also add ``BindingResult`` object to arguments.
       | Input validation is automatically performed using Bean Validation and the result is stored in ``BindingResult`` object. 
   * - | (2)
     - | It can be checked whether there is error by using ``hasErrors`` method.

.. literalinclude:: ../../resources/helloworld2/src/main/webapp/WEB-INF/views/echo/index.jsp
   :language: jsp
   :emphasize-lines: 15

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Item number
     - Description
   * - | (1)
     - | Add ``form:errors`` tag for displaying error message when an error occurs on input screen.


| Implementation of input validation is completed. 
| Error message is displayed in the following conditions:
* When an empty name is sent 
* Size is more than 5 characters.

.. figure:: images/AppValidationEmpty.png
   :alt: Validation Error (name is empty)

.. figure:: images/AppValidationSizeOver.png
   :alt: Validation Error (name's size is over 5)


The generated HTML is as follows 

.. code-block:: html

    <body>

      <form id="echoForm" action="/helloworld/echo/hello" method="post">
        <label for="name">Input Your Name:</label>
        <input id="name" name="name" type="text" value=""/>
        <span id="name.errors" style="color:red">size must be between 1 and 5</span>
        <input type="submit" />
      </form>
    </body>
    </html>



Summary
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The following are the learnings from this chapter.

#. Setting up configuration file of Spring MVC (simplified level)
#. How to do Screen Transition (simplified level)
#. Way to pass values between screens.
#. Simple input validation

If above points are still not understood, it is recommended to read this chapter again and start again from building the environment. 
This will imporve the understanding of above concepts.


.. raw:: latex

   \newpage

