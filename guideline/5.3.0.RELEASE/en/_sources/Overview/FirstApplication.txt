First application based on Spring MVC 
--------------------------------------------------------------

.. only:: html

 .. contents:: Index
    :depth: 3
    :local:

Before entering into the advanced usage of Spring MVC, it is better to understand Spring MVC by actually trying hands-on web application development using Spring MVC.

Prerequisites
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The description of this chapter has been verified on the following environment. (For other environments, replace the contents mentioned here appropriately)

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 25 75

    * - Classification
      - Product
    * - OS
      - Windows 7
    * - JVM
      - `Java <http://www.oracle.com/technetwork/java/javase/downloads/index.html>`_ 1.8
    * - IDE
      - `Spring Tool Suite <http://spring.io/tools/sts/all>`_ 3.6.4.RELEASE (Hereafter referred as [STS])
    * - Build Tool
      - `Apache Maven <http://maven.apache.org/download.cgi>`_ 3.3.9 (Hereafter referred as [Maven])
    * - Application Server
      - `Pivotal tc Server <https://network.pivotal.io/products/pivotal-tcserver>`_ Developer Edition v3.1 (enclosed in STS)
    * - Web Browser
      - `Google Chrome <https://www.google.co.jp/chrome/browser/desktop/index.html>`_ 46.0.2490.80 m

.. note::

  To connect the internet via proxy server, 
  STS Proxy settings and \ `Maven Proxy settings <http://maven.apache.org/guides/mini/guide-proxies.html>`_\  are required for the following operations.


Create a New Project
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Create project using `mvn archetype:generate` on internet.

.. code-block:: console

    mvn archetype:generate -B^
     -DarchetypeGroupId=org.terasoluna.gfw.blank^
     -DarchetypeArtifactId=terasoluna-gfw-web-blank-archetype^
     -DarchetypeVersion=5.3.0.RELEASE^
     -DgroupId=com.example.helloworld^
     -DartifactId=helloworld^
     -Dversion=1.0.0-SNAPSHOT

Here, created a project on Windows environment.

.. code-block:: console

    C:\work>mvn archetype:generate -B^
    More?  -DarchetypeGroupId=org.terasoluna.gfw.blank^
    More?  -DarchetypeArtifactId=terasoluna-gfw-web-blank-archetype^
    More?  -DarchetypeVersion=5.3.0.RELEASE^
    More?  -DgroupId=com.example.helloworld^
    More?  -DartifactId=helloworld^
    More?  -Dversion=1.0.0-SNAPSHOT
    [INFO] Scanning for projects...
    [INFO]
    [INFO] ------------------------------------------------------------------------
    [INFO] Building Maven Stub Project (No POM) 1
    [INFO] ------------------------------------------------------------------------
    [INFO]
    [INFO] >>> maven-archetype-plugin:2.4:generate (default-cli) > generate-sources @ standalone-pom >>>
    [INFO]
    [INFO] <<< maven-archetype-plugin:2.4:generate (default-cli) < generate-sources @ standalone-pom <<<
    [INFO]
    [INFO] --- maven-archetype-plugin:2.4:generate (default-cli) @ standalone-pom ---
    [INFO] Generating project in Batch mode
    [INFO] Archetype repository not defined. Using the one from [org.terasoluna.gfw.blank:terasoluna-gfw-web-blank-archetype:5.3.0.RELEASE] found in catalog remote
    [INFO] ----------------------------------------------------------------------------
    [INFO] Using following parameters for creating project from Archetype: terasoluna-gfw-web-blank-archetype:5.3.0.RELEASE
    [INFO] ----------------------------------------------------------------------------
    [INFO] Parameter: groupId, Value: com.example.helloworld
    [INFO] Parameter: artifactId, Value: helloworld
    [INFO] Parameter: version, Value: 1.0.0-SNAPSHOT
    [INFO] Parameter: package, Value: com.example.helloworld
    [INFO] Parameter: packageInPathFormat, Value: com/example/helloworld
    [INFO] Parameter: package, Value: com.example.helloworld
    [INFO] Parameter: version, Value: 1.0.0-SNAPSHOT
    [INFO] Parameter: groupId, Value: com.example.helloworld
    [INFO] Parameter: artifactId, Value: helloworld
    [INFO] project created from Archetype in dir: C:\work\helloworld
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 9.266 s
    [INFO] Finished at: 2017-02-10T11:46:01+09:00
    [INFO] Final Memory: 13M/188M
    [INFO] ------------------------------------------------------------------------
    C:\work>

Select the archetype created project from STS menu [File] -> [Import] -> [Maven] -> [Existing Maven Projects] -> [Next].

.. figure:: images/NewMVCProjectImport.png
   :alt: New MVC Project Import
   :width: 60%

Click on [Finish] by selecting \ ``C:\work\helloworld`` \ in Root Directory and selecting pom.xml of helloworld in Projects.

.. figure:: images/NewMVCProjectCreate.png
   :alt: New MVC Project Import
   :width: 60%

Following project is generated in the Package Explorer.

.. figure:: images/HelloWorldWorkspace.png
   :alt: workspace

To understand the configuration of Spring MVC, the generated Spring MVC configuration file (src/main/resources/META-INF/spring/spring-mvc.xml) is described briefly.

.. code-block:: xml
    :emphasize-lines: 18-19, 32-33, 73-77

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xmlns:mvc="http://www.springframework.org/schema/mvc"
        xmlns:util="http://www.springframework.org/schema/util"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
        ">

        <context:property-placeholder
            location="classpath*:/META-INF/spring/*.properties" />

        <!-- (1) Enables the Spring MVC @Controller programming model -->
        <mvc:annotation-driven>
            <mvc:argument-resolvers>
                <bean
                    class="org.springframework.data.web.PageableHandlerMethodArgumentResolver" />
                <bean
                    class="org.springframework.security.web.method.annotation.AuthenticationPrincipalArgumentResolver" />
            </mvc:argument-resolvers>
            <!-- workaround to CVE-2016-5007. -->
            <mvc:path-matching path-matcher="pathMatcher" />
        </mvc:annotation-driven>

        <mvc:default-servlet-handler />

        <!-- (2) -->
        <context:component-scan base-package="com.example.helloworld.app" />

        <mvc:resources mapping="/resources/**"
            location="/resources/,classpath:META-INF/resources/"
            cache-period="#{60 * 60}" />

        <mvc:interceptors>
            <mvc:interceptor>
                <mvc:mapping path="/**" />
                <mvc:exclude-mapping path="/resources/**" />
                <mvc:exclude-mapping path="/**/*.html" />
                <bean
                    class="org.terasoluna.gfw.web.logging.TraceLoggingInterceptor" />
            </mvc:interceptor>
            <mvc:interceptor>
                <mvc:mapping path="/**" />
                <mvc:exclude-mapping path="/resources/**" />
                <mvc:exclude-mapping path="/**/*.html" />
                <bean
                    class="org.terasoluna.gfw.web.token.transaction.TransactionTokenInterceptor" />
            </mvc:interceptor>
            <mvc:interceptor>
                <mvc:mapping path="/**" />
                <mvc:exclude-mapping path="/resources/**" />
                <mvc:exclude-mapping path="/**/*.html" />
                <bean class="org.terasoluna.gfw.web.codelist.CodeListInterceptor">
                    <property name="codeListIdPattern" value="CL_.+" />
                </bean>
            </mvc:interceptor>
            <!--  REMOVE THIS LINE IF YOU USE JPA
            <mvc:interceptor>
                <mvc:mapping path="/**" />
                <mvc:exclude-mapping path="/resources/**" />
                <mvc:exclude-mapping path="/**/*.html" />
                <bean
                    class="org.springframework.orm.jpa.support.OpenEntityManagerInViewInterceptor" />
            </mvc:interceptor>
                REMOVE THIS LINE IF YOU USE JPA  -->
        </mvc:interceptors>

        <!-- (3) Resolves views selected for rendering by @Controllers to .jsp resources in the /WEB-INF/views directory -->
        <!-- Settings View Resolver. -->
        <mvc:view-resolvers>
            <mvc:jsp prefix="/WEB-INF/views/" />
        </mvc:view-resolvers>

        <bean id="requestDataValueProcessor"
            class="org.terasoluna.gfw.web.mvc.support.CompositeRequestDataValueProcessor">
            <constructor-arg>
                <util:list>
                    <bean
                        class="org.springframework.security.web.servlet.support.csrf.CsrfRequestDataValueProcessor" />
                    <bean
                        class="org.terasoluna.gfw.web.token.transaction.TransactionTokenRequestDataValueProcessor" />
                </util:list>
            </constructor-arg>
        </bean>

        <!-- Setting Exception Handling. -->
        <!-- Exception Resolver. -->
        <bean id="systemExceptionResolver"
            class="org.terasoluna.gfw.web.exception.SystemExceptionResolver">
            <property name="exceptionCodeResolver" ref="exceptionCodeResolver" />
            <!-- Setting and Customization by project. -->
            <property name="order" value="3" />
            <property name="exceptionMappings">
                <map>
                    <entry key="ResourceNotFoundException" value="common/error/resourceNotFoundError" />
                    <entry key="BusinessException" value="common/error/businessError" />
                    <entry key="InvalidTransactionTokenException" value="common/error/transactionTokenError" />
                    <entry key=".DataAccessException" value="common/error/dataAccessError" />
                </map>
            </property>
            <property name="statusCodes">
                <map>
                    <entry key="common/error/resourceNotFoundError" value="404" />
                    <entry key="common/error/businessError" value="409" />
                    <entry key="common/error/transactionTokenError" value="409" />
                    <entry key="common/error/dataAccessError" value="500" />
                </map>
            </property>
            <property name="defaultErrorView" value="common/error/systemError" />
            <property name="defaultStatusCode" value="500" />
        </bean>
        <!-- Setting AOP. -->
        <bean id="handlerExceptionResolverLoggingInterceptor"
            class="org.terasoluna.gfw.web.exception.HandlerExceptionResolverLoggingInterceptor">
            <property name="exceptionLogger" ref="exceptionLogger" />
        </bean>
        <aop:config>
            <aop:advisor advice-ref="handlerExceptionResolverLoggingInterceptor"
                pointcut="execution(* org.springframework.web.servlet.HandlerExceptionResolver.resolveException(..))" />
        </aop:config>

        <!-- Setting PathMatcher. -->
        <bean id="pathMatcher" class="org.springframework.util.AntPathMatcher">
            <property name="trimTokens" value="false" />
        </bean>

    </beans>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - Default settings of Spring MVC are configured by defining \ ``<mvc:annotation-driven>``\. Refer to the official website `Enabling the MVC Java Config or the MVC XML Namespace <http://docs.spring.io/spring/docs/4.3.5.RELEASE/spring-framework-reference/html/mvc.html#mvc-config-enable>`_ for default configuration of Spring framework.
   * - | (2)
     - Define the package which will be target of searching components used in Spring MVC.
   * - | (3)
     - Define the location of the JSP by specifying \ ``ViewResolver`` \ for JSP.

       .. tip::

           \ ``<mvc:view-resolvers>`` \ element is a XML element that added from Spring Framework 4.1.
           By using \ ``<mvc:view-resolvers>`` \ element, it is possible to define \ ``ViewResolver`` \ simply.

           The definition example of using the conventional \ ``<bean>`` \ element is shown below.

            .. code-block:: xml

               <bean id="viewResolver"
                   class="org.springframework.web.servlet.view.InternalResourceViewResolver">
                   <property name="prefix" value="/WEB-INF/views/" />
                   <property name="suffix" value=".jsp" />
               </bean>

|

Next, Controller (\ ``com.example.helloworld.app.welcome.HelloController``\ ) for displaying the Welcome page is described briefly.

.. code-block:: java
   :emphasize-lines: 17,26,36,38

    package com.example.helloworld.app.welcome;

    import java.text.DateFormat;
    import java.util.Date;
    import java.util.Locale;

    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;

    /**
     * Handles requests for the application home page.
     */
    @Controller // (4)
    public class HelloController {

        private static final Logger logger = LoggerFactory
                .getLogger(HelloController.class);

        /**
         * Simply selects the home view to render by returning its name.
         */
        @RequestMapping(value = "/", method = {RequestMethod.GET, RequestMethod.POST}) // (5)
        public String home(Locale locale, Model model) {
            logger.info("Welcome home! The client locale is {}.", locale);

            Date date = new Date();
            DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG,
                    DateFormat.LONG, locale);

            String formattedDate = dateFormat.format(date);

            model.addAttribute("serverTime", formattedDate); // (6)

            return "welcome/home"; // (7)
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (4)
     - It can be read automatically by DI container if ``@Controller`` annotation is used. As stated earlier in [explanation of Spring MVC configuration files (2)], it is the target of component-scan.
   * - | (5)
     - It gets executed when the HTTP method is GET or POST and the Resource is (or request URL) is "/".
   * - | (6)
     - Set \ ``Model`` \ object to be delivered to View.
   * - | (7)
     - Return View name. "WEB-INF/views/welcome/home.jsp" is rendered as per the configuration [Explanation of Spring MVC configuration files (3)] .

|

Lastly, JSP (\ ``src/main/webapp/WEB-INF/views/welcome/home.jsp``\ ) for displaying the Welcome page is described briefly.

.. code-block:: jsp
    :emphasize-lines: 12

    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="utf-8">
    <title>Home</title>
    <link rel="stylesheet"
        href="${pageContext.request.contextPath}/resources/app/css/styles.css">
    </head>
    <body>
        <div id="wrapper">
            <h1>Hello world!</h1>
            <p>The time on the server is ${serverTime}.</p> <%-- (8) --%>
        </div>
    </body>
    </html>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (8)
     - As stated earlier in [explanation of Controller (6)] the object (serverTime) set in Model is stored in HttpServletRequest.
       Therefore the value passed from Controller can be output by mentioning \ ``${serverTime}`` \ in JSP.

       **However about ${XXX}, it is necessary to perform HTML escaping since there is possibility of XSS attack.**

|

Run on Server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
| Right click "helloworld" project in STS, and start helloworld project by executing "Run As" -> "Run On Server" -> "localhost" -> "Pivotal tc Server Developer Edition v3.0" -> "Finish".
| Enter "http://localhost:8080/helloworld/" in browser to display the following screen.

.. figure:: images/AppHelloWorldIndex.png
   :alt: Hello World

.. note::

    Since tc Server internally uses the Tomcat, it is possible to choose below two versions in the STS.

    * tomcat-8.0.15.A.RELEASE (version used by default)
    * tomcat-7-0.57.A.RELEASE

    If you want to switch the Tomcat to be used, change the [Version] field of ts Server by opening the [Edit Server Runtime Environment] dialog box. 
    Java (JRE) version can also be changed from this dialog box.

     .. figure:: images/EditServerRuntimeEnvironment.png
        :alt: Edit Server Runtime Environment
        :width: 80%


|

.. _first-application-create-an-echo-application:

Create an Echo Application
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Lets go ahead and create a simple application. It is a typical eco application in which message will be displayed 
if name is entered in the text field as given below.

.. figure:: images/AppEchoIndex.png
   :alt: Form of Echo Application

.. figure:: images/AppEchoHello.png
   :alt: Output of Echo Application

|

Creating a form object
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| First create a form object to accept the value of text field. 
| Create \ ``EchoForm`` \ class under \ ``com.example.helloworld.app.echo`` \ package. It is a simple JavaBean that has only one1 property.

.. code-block:: java

    package com.example.helloworld.app.echo;

    import java.io.Serializable;

    public class EchoForm implements Serializable {
        private static final long serialVersionUID = 2557725707095364445L;

        private String name;

        public void setName(String name) {
            this.name = name;
        }

        public String getName() {
            return name;
        }
    }

|

Create a Controller
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Next, create the Controller class.
| create the ``EchoController`` class under ``com.example.helloworld.app.echo`` package.

.. code-block:: java
    :emphasize-lines: 10,13,19,21,24-26

    package com.example.helloworld.app.echo;

    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;

    @Controller
    @RequestMapping("echo")
    public class EchoController {

        @ModelAttribute // (1)
        public EchoForm setUpEchoForm() {
            EchoForm form = new EchoForm();
            return form;
        }

        @RequestMapping // (2)
        public String index(Model model) {
            return "echo/index"; // (3)
        }

        @RequestMapping(value = "hello", method = RequestMethod.POST) // (4)
        public String hello(EchoForm form, Model model) {// (5)
            model.addAttribute("name", form.getName()); // (6)
            return "echo/hello";
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Add ``@ModelAttribute`` annotation to the method. Return value of such a method is automatically added to the Model.
       | Attribute name of the Model can be specified in ``@ModelAttribute``, but the class name with the first letter in lower case is the default attribute name. In this case it will be "echoForm". This attribute name must match with the value of ``modelAttribute`` of ``form:form tag``.
   * - | (2)
     - | When nothing is specified in ``value`` attribute of ``@RequestMapping`` annotation at the method level, it is mapped to ``@RequestMapping`` added at class level. In this case, ``index`` method is called, if "<contextPath>/echo" is accessed. 
       | When nothing is set in ``method`` attribute, mapping is done for any HTTP method.
   * - | (3)
     - | Since "echo/index" is returned as View name, "WEB-INF/views/echo/index.jsp" is rendered by the ViewResolver.
   * - | (4)
     - | Since "hello" is specified in \ ``value`` \ attribute and \ ``RequestMethod.POST`` \ is specified in \ ``method`` \ attribute of the ``@RequestMapping`` annotation method, if "<contextPath>/echo/hello" is accessed with POST method, ``hello`` method is called.
   * - | (5)
     - | EchoForm object added to the model in (1) is passed as argument.
   * - | (6)
     - | ``name`` entered in form is passed as it is to the View.

.. note::

    The value specified in the \ ``method`` \ attribute of \ ``@RequestMapping`` \ annotation is
    generally varied by how the data transmitted from the client.

    * POST method in case of storing data to the server (in case of updating process).
    * GET method or unspecified (any method) in case of not storing data to the server (in case of referring process).

    In echo application,

    * \ ``index`` \ method is not going to save data to the server, it is unspecified (any method)
    * \ ``hello`` \ method is going to save data into \ ``Model`` \ object, it is POST method

    etc are specified.

|

Create JSP Files
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Finally create JSP for input screen and output screen. Each file path must match with View name as follows.

Create input screen (src/main/webapp/WEB-INF/views/echo/index.jsp).

.. code-block:: jsp
    :emphasize-lines: 7-8

    <!DOCTYPE html>
    <html>
    <head>
    <title>Echo Application</title>
    </head>
    <body>
      <%-- (1) --%>
      <form:form modelAttribute="echoForm" action="${pageContext.request.contextPath}/echo/hello">
        <form:label path="name">Input Your Name:</form:label>
        <form:input path="name" />
        <input type="submit" />
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
     - | HTML form is constructed using tag library. Specify the name of form object created by Controller in ``modelAttribute`` attribute.
       | Refer `here <http://docs.spring.io/spring/docs/4.3.5.RELEASE/spring-framework-reference/html/view.html#view-jsp-formtaglib-formtag>`_  for tag library.

.. note::

    POST method is used if \ ``method`` \ attribute of \ ``<form:form>`` \ tag is omitted.

The generated HTML is as follows

.. code-block:: html
    :emphasize-lines: 7

    <!DOCTYPE html>
    <html>
    <head>
    <title>Echo Application</title>
    </head>
    <body>
      <form id="echoForm" action="/helloworld/echo/hello" method="post">
        <label for="name">Input Your Name:</label>
        <input id="name" name="name" type="text" value=""/>
        <input type="submit" />
      <input type="hidden" name="_csrf" value="43595f38-3edd-4c08-843b-3c31a00d2b15" />
    </form>
    </body>
    </html>


|

Create output screen (src/main/webapp/WEB-INF/views/echo/hello.jsp).

.. code-block:: jsp
    :emphasize-lines: 8

    <!DOCTYPE html>
    <html>
    <head>
    <title>Echo Application</title>
    </head>
    <body>
      <p>
        Hello <c:out value="${name}" /> <%-- (2) --%>
      </p>
    </body>
    </html>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (2)
     - | Output "name" passed from Controller. Take countermeasures for XSS using ``c:out`` tag.

.. note::

    Here, XSS countermeasure taken using ``c:out`` standard tag, however, ``f:h()`` function provided in common library can be used easily.
    Refer to :doc:`../Security/XSS` for details.

|

| Implementation of Eco application is completed here.
| Start the server and form will be displayed by accessing "http://localhost:8080/helloworld/echo".

|

Implement Input Validation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Till this point Input validation is not implemented in this application.
In Spring MVC, `Bean Validation <http://jcp.org/en/jsr/detail?id=349>`_ \ and annotation based input validation can be easily implemented.
For example name input validation is performed in Eco Application.


Add annotation for Input check rule to the \ ``name`` \ of \ ``EchoForm``\.

.. code-block:: java
    :emphasize-lines: 5,6,11,12

    package com.example.helloworld.app.echo;

    import java.io.Serializable;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;

    public class EchoForm implements Serializable {
        private static final long serialVersionUID = 2557725707095364445L;

        @NotNull // (1)
        @Size(min = 1, max = 5) // (2)
        private String name;

        public void setName(String name) {
            this.name = name;
        }

        public String getName() {
            return name;
        }
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Whether ``name`` parameter exists in HTTP request or not can be checked by ``@NotNull`` annotation.
   * - | (2)
     - | Whether the size of ``name`` is more than or equal to 1 and less than or equal to 5 can be checked by ``@Size(min = 1, max = 5)``.

|

Implement the input check and the error handling when an error occurs in the input check.

.. code-block:: java
    :emphasize-lines: 5,6,27-30

    package com.example.helloworld.app.echo;

    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.validation.BindingResult;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;

    @Controller
    @RequestMapping("echo")
    public class EchoController {

        @ModelAttribute
        public EchoForm setUpEchoForm() {
            EchoForm form = new EchoForm();
            return form;
        }

        @RequestMapping
        public String index(Model model) {
            return "echo/index";
        }

        @RequestMapping(value = "hello", method = RequestMethod.POST)
        public String hello(@Validated EchoForm form, BindingResult result, Model model) { // (1)
            if (result.hasErrors()) { // (2)
                return "echo/index";
            }
            model.addAttribute("name", form.getName());
            return "echo/hello";
        }
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | In Controller, add ``@Validated`` annotation to the argument on which validation needs to be executed. Also add ``BindingResult`` object to the arguments.
       | Input validation is automatically performed using Bean Validation and the result is stored in ``BindingResult`` object.
   * - | (2)
     - | Error can be checked by executing ``hasErrors`` method. If there is an input error, it returns View name to display the input screen.

|

Add the implementation of displaying input error message on input screen (src/main/webapp/WEB-INF/views/echo/index.jsp).


.. code-block:: jsp
    :emphasize-lines: 10

    <!DOCTYPE html>
    <html>
    <head>
    <title>Echo Application</title>
    </head>
    <body>
      <form:form modelAttribute="echoForm" action="${pageContext.request.contextPath}/echo/hello">
        <form:label path="name">Input Your Name:</form:label>
        <form:input path="name" />
        <form:errors path="name" cssStyle="color:red" /><%-- (1) --%>
        <input type="submit" />
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
     - | Add ``form:errors`` tag for displaying error message when an error occurs on input screen.

|

| Implementation of input validation is completed.
| Error message is displayed in the following conditions:

* When an empty name is sent
* Size is more than 5 characters.

.. figure:: images/AppValidationEmpty.png
   :alt: Validation Error (name is empty)

.. figure:: images/AppValidationSizeOver.png
   :alt: Validation Error (name's size is over 5)


The generated HTML is as follows.

.. code-block:: html
    :emphasize-lines: 10

    <!DOCTYPE html>
    <html>
    <head>
    <title>Echo Application</title>
    </head>
    <body>
      <form id="echoForm" action="/helloworld/echo/hello" method="post">
        <label for="name">Input Your Name:</label>
        <input id="name" name="name" type="text" value=""/>
        <span id="name.errors" style="color:red">size must be between 1 and 5</span>
        <input type="submit" />
      <input type="hidden" name="_csrf" value="6e94a78d-4a2c-4a41-a514-0a60f0dbedaf" />
    </form>
    </body>
    </html>


|

Summary
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The following are the learnings from this chapter.

#. How to create blank project using \ ``mvn archetype:generate``\.
#. Basic Spring MVC configuration set-up
#. Simplified level screen transition 
#. Way to pass values between screens
#. Simple input validation


If above points are still not understood, it is recommended to read this chapter again and start again from building the environment.

.. raw:: latex

   \newpage

