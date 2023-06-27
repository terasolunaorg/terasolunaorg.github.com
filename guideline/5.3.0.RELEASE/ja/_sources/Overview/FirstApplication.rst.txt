はじめてのSpring MVCアプリケーション
--------------------------------------------------------------

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

Spring MVCの、詳細な使い方の解説に入る前に、実際にSpring MVCに触れることで、
Spring MVCを用いたWebアプリケーションの開発に対するイメージをつかむ。

検証環境
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

本節の説明では、次の環境で動作検証している。(他の環境で実施する際は、本書をベースに適宜読み替えて設定していくこと。)

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 25 75

    * - 種別
      - プロダクト
    * - OS
      - Windows 7
    * - JVM
      - `Java <http://www.oracle.com/technetwork/java/javase/downloads/index.html>`_ 1.8
    * - IDE
      - `Spring Tool Suite <http://spring.io/tools/sts/all>`_ 3.6.4.RELEASE (以降「STS」と呼ぶ)
    * - Build Tool
      - `Apache Maven <http://maven.apache.org/download.cgi>`_ 3.3.9 (以降「Maven」と呼ぶ)
    * - Application Server
      - `Pivotal tc Server <https://network.pivotal.io/products/pivotal-tcserver>`_ Developer Edition v3.1 (STSに同封)
    * - Web Browser
      - `Google Chrome <https://www.google.co.jp/chrome/browser/desktop/index.html>`_ 46.0.2490.80 m

.. note::

    インターネット接続するために、プロキシサーバーを介する必要がある場合、
    以下の作業を行うため、STSのProxy設定と、 `MavenのProxy設定 <http://maven.apache.org/guides/mini/guide-proxies.html>`_\ が必要である。


新規プロジェクト作成
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

インターネットから `mvn archetype:generate` を利用して、プロジェクトを作成する。

.. code-block:: console

    mvn archetype:generate -B^
     -DarchetypeGroupId=org.terasoluna.gfw.blank^
     -DarchetypeArtifactId=terasoluna-gfw-web-blank-archetype^
     -DarchetypeVersion=5.3.0.RELEASE^
     -DgroupId=com.example.helloworld^
     -DartifactId=helloworld^
     -Dversion=1.0.0-SNAPSHOT

ここではWindows上にプロジェクトの元を作成する。

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

STSのメニューから、[File] -> [Import] -> [Maven] -> [Existing Maven Projects] -> [Next]を選択し、archetypeで作成したプロジェクトを選択する。

.. figure:: images/NewMVCProjectImport.png
   :alt: New MVC Project Import
   :width: 60%

Root Directoryに \ ``C:\work\helloworld``\ を設定し、Projectsにhelloworldのpom.xmlが選択された状態で、 [Finish] を押下する。

.. figure:: images/NewMVCProjectCreate.png
   :alt: New MVC Project Import
   :width: 60%

Package Explorerに、次のようなプロジェクトが生成される。

.. figure:: images/HelloWorldWorkspace.png
   :alt: workspace

Spring MVCの設定方法を理解するために、生成されたSpring MVCの設定ファイル(src/main/resources/META-INF/spring/spring-mvc.xml)について、簡単に説明する。

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

   * - 項番
     - 説明
   * - | (1)
     - \ ``<mvc:annotation-driven>``\要素を定義することにより、Spring MVCのデフォルト設定が行われる。デフォルトの設定については、 Springの公式ページである `Enabling the MVC Java Config or the MVC XML Namespace <http://docs.spring.io/spring/docs/4.3.5.RELEASE/spring-framework-reference/html/mvc.html#mvc-config-enable>`_ を参照されたい。
   * - | (2)
     - Spring MVCで使用するコンポーネントを探すパッケージを定義する。
   * - | (3)
     - JSP用の\ ``ViewResolver``\ を指定し、JSPファイルの配置場所を定義する。

       .. tip::

           \ ``<mvc:view-resolvers>``\ 要素はSpring Framework 4.1から追加されたXML要素である。
           \ ``<mvc:view-resolvers>``\ 要素を使用すると、\ ``ViewResolver``\ をシンプルに定義することが出来る。

           従来通り\ ``<bean>``\ 要素を使用した場合の定義例を以下に示す。

            .. code-block:: xml

               <bean id="viewResolver"
                   class="org.springframework.web.servlet.view.InternalResourceViewResolver">
                   <property name="prefix" value="/WEB-INF/views/" />
                   <property name="suffix" value=".jsp" />
               </bean>

|

次に、Welcomeページを表示するためのController (\ ``com.example.helloworld.app.welcome.HelloController``\ ) について、簡単に説明する。

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

   * - 項番
     - 説明
   * - | (4)
     - ``@Controller`` アノテーションを付けることで、DIコンテナにより、コントローラクラスが自動で読み込まれる。前述「Spring MVCの設定ファイルの説明(2)」の設定により、component-scanの対象となっている。
   * - | (5)
     - HTTPメソッドがGETまたはPOSTで、Resource（もしくはRequest URL）が"/"で、アクセスする際に実行される。
   * - | (6)
     - Viewに渡したいオブジェクトを\ ``Model``\ に設定する。
   * - | (7)
     - View名を返却する。前述「Spring MVCの設定ファイルの説明(3)」の設定により、"WEB-INF/views/welcome/home.jsp"がレンダリングされる。

|

最後に、Welcomeページを表示するためのJSP (\ ``src/main/webapp/WEB-INF/views/welcome/home.jsp``\ ) について、簡単に説明する。

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

   * - 項番
     - 説明
   * - | (8)
     - 前述の「Controllerの説明(6)」でModelに設定したオブジェクト(serverTime)は、HttpServletRequestに格納される。
       そのため、JSPで\ ``${serverTime}``\ と記述することで、Controllerで設定した値を画面に出力することができる。

       **ただし、${XXX}の記述は、XSS対象になる可能性があるので、文字列を出力する場合はHTMLエスケープする必要がある。**

|

サーバーを起動する
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
| STSで、"helloworld"プロジェクトを右クリックして、"Run As" -> "Run On Server" -> "localhost" -> "Pivotal tc Server Developer Edition v3.0" -> "Finish"を実行し、helloworldプロジェクトを起動する。
| ブラウザに "http://localhost:8080/helloworld/" を入力し、実行すると下記の画面が表示される。

.. figure:: images/AppHelloWorldIndex.png
   :alt: Hello World

.. note::

    tc Serverは内部でTomcatを利用しており、動作検証で使用したSTSでは以下の2つのバージョンを選択する事ができる。

    * tomcat-8.0.15.A.RELEASE (デフォルトで利用されるバージョン)
    * tomcat-7-0.57.A.RELEASE

    利用するTomcatを切り替えたい場合は、ts Serverの「Edit Server Runtime Environment」ダイアログを開き「Version」フィールドを変更すればよい。
    Java(JRE)のバージョンもこのダイアログから変更する事ができる。

     .. figure:: images/EditServerRuntimeEnvironment.png
        :alt: Edit Server Runtime Environment
        :width: 80%


|

.. _first-application-create-an-echo-application:

エコーアプリケーションの作成
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
続いて、簡単なアプリケーションを作成する。作成するのは、次の図のようなテキストフィールドに、名前を入力すると
メッセージを表示する、いわゆるエコーアプリケーションである。

.. figure:: images/AppEchoIndex.png
   :alt: Form of Echo Application

.. figure:: images/AppEchoHello.png
   :alt: Output of Echo Application

|

フォームオブジェクトの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| まずは、テキストフィールドの値を受け取るための、フォームオブジェクトを作成する。
| \ ``com.example.helloworld.app.echo``\ パッケージに\ ``EchoForm``\ クラスを作成する。プロパティを1つだけ持つ、単純なJavaBeanである。

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

Controllerの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 次に、Controllerを作成する。
| 同じく ``com.example.helloworld.app.echo`` パッケージに、``EchoController`` クラスを作成する。

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

   * - 項番
     - 説明
   * - | (1)
     - | ``@ModelAttribute`` というアノテーションを、メソッドに付加する。このアノテーションがついたメソッドの返り値は、自動でModelに追加される。
       | Modelの属性名を、 ``@ModelAttribute`` で指定することもできるが、デフォルトでは、クラス名の先頭を小文字にした値が、属性名になる。この場合は、”echoForm”である。フォームの属性名は、次に説明する  ``form:form タグ`` の ``modelAttribute`` 属性の値に一致している必要がある。
   * - | (2)
     - | メソッドに付加した ``@RequestMapping`` アノテーションの ``value`` 属性に、何も指定しない場合、クラスに付加した ``@RequestMapping`` のルートに、マッピングされる。この場合、"<contextPath>/echo"にアクセスすると、 ``index`` メソッドが呼ばれる。
       | ``method`` 属性に何もしない場合は、任意のHTTPメソッドでマッピングされる。
   * - | (3)
     - | View名で"echo/index"を返すので、ViewResolverにより、 "WEB-INF/views/echo/index.jsp"がレンダリングされる。
   * - | (4)
     - | メソッドに付加した ``@RequestMapping`` アノテーションの\ ``value``\ 属性に"hello"を、\ ``method``\ 属性に\ ``RequestMethod.POST``\ を指定しているので、この場合、"<contextPath>/echo/hello"にPOSTメソッドを使用してアクセスすると ``hello`` メソッドが呼ばれる。
   * - | (5)
     - | 引数に、EchoFormには(1)によりModelに追加されたEchoFormオブジェクトが渡される。
   * - | (6)
     - | フォームで入力された ``name`` を、Viewにそのまま渡す。

.. note::

    \ ``@RequestMapping``\ アノテーションの\ ``method``\ 属性に指定する値は、
    クライアントから送信されたデータの扱い方によって変えるのが一般的である。

    * データをサーバに保存する場合(更新系の処理の場合)は、POSTメソッド。
    * データをサーバに保存しない場合(参照系の処理の場合)は、GETメソッド又は未指定(任意のメソッド)。

    エコーアプリケーションでは、

    * \ ``index``\ メソッドはデータをサーバに保存しない処理なので未指定(任意のメソッド)
    * \ ``hello``\ メソッドはデータを\ ``Model``\ オブジェクトに保存する処理なのでPOSTメソッド

    を指定している。

|

JSPの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
最後に、入力画面と、出力画面のJSPを作成する。それぞれのファイルパスは、View名に合わせて、次のようになる。

入力画面 (src/main/webapp/WEB-INF/views/echo/index.jsp) を作成する。

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

   * - 項番
     - 説明
   * - | (1)
     - | タグライブラリを利用し、HTMLフォームを構築している。 ``modelAttribute`` 属性に、Controllerで用意したフォームオブジェクトの名前を指定する。
       | タグライブラリは `こちら <http://docs.spring.io/spring/docs/4.3.5.RELEASE/spring-framework-reference/html/view.html#view-jsp-formtaglib-formtag>`_\を参照されたい。

.. note::

    \ ``<form:form>``\ タグの\ ``method``\ 属性を省略した場合は、POSTメソッドが使用される。

出力されるHTMLは、

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

となる。

|

出力画面 (src/main/webapp/WEB-INF/views/echo/hello.jsp) を作成する。

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

   * - 項番
     - 説明
   * - | (2)
     - | Controllerから渡された"name"を出力する。 ``c:out`` タグにより、XSS対策を行っている。

.. note::

    ここではXSS対策を標準タグの ``c:out`` で実現したが、より容易に使用できる ``f:h()`` 関数を共通ライブラリで用意している。
    詳細は、  :doc:`../Security/XSS` を参照されたい。

|

| これでエコーアプリケーションの実装は完了である。
| サーバーを起動し、 "http://localhost:8080/helloworld/echo"にアクセスするとフォームが表示される。

|

入力チェックの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
ここまでのアプリケーションでは、入力チェックを行っていない。
Spring MVCでは、 `Bean Validation <http://jcp.org/en/jsr/detail?id=349>`_\ をサポートしており、アノテーションベースな入力チェックを、簡単に
実装することができる。例として、エコーアプリケーションで名前の入力チェックを行う。


\ ``EchoForm``\ の\ ``name``\ フィールドに、入力チェックルールを指定するアノーテションを付与する。

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

   * - 項番
     - 説明
   * - | (1)
     - | ``@NotNull`` アノテーションをつけることで、HTTPリクエスト中に ``name`` パラメータがあることを確認する。
   * - | (2)
     - | ``@Size(min = 1, max = 5)`` をつけることで、``name`` のサイズが、1以上5以下であることを確認する。

|

入力チェックが実行されるように修正し、入力チェックでエラーが発生した場合の処理を実装する。

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

   * - 項番
     - 説明
   * - | (1)
     - | コントローラー側には、Validation対象の引数に ``@Validated`` アノテーションを付加し、 ``BindingResult`` オブジェクトを引数に追加する。
       | Bean Validationによる入力チェックは、自動で行われる。結果は、 ``BindingResult`` オブジェクトに渡される。
   * - | (2)
     - | ``hasErrors`` メソッドを実行して、エラーがあるかどうかを確認する。入力エラーがある場合は、入力画面を表示するためのView名を返却する。

|

入力画面 (src/main/webapp/WEB-INF/views/echo/index.jsp) に、入力エラーのメッセージを表示するための実装を追加する。


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

   * - 項番
     - 説明
   * - | (1)
     - | 入力画面には、エラーがあった場合に、エラーメッセージを表示するため、 ``form:errors`` タグを追加する。

|

| 以上で、入力チェックの実装は完了である。
| 実際に、次のような場合、エラーメッセージが表示される。

* 名前を空にして送信した場合
* 5文字より大きいサイズで送信した場合

.. figure:: images/AppValidationEmpty.png
   :alt: Validation Error (name is empty)

.. figure:: images/AppValidationSizeOver.png
   :alt: Validation Error (name's size is over 5)


出力されるHTMLは、

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

となる。

|

まとめ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

この章では、

#. \ ``mvn archetype:generate``\を利用したブランクプロジェクトの作成方法
#. SpringMVCの基本的な設定方法
#. 最も簡易な、画面遷移方法
#. 画面間での値の引き渡し方法
#. シンプルな入力チェック方法

を学んだ。

上記の内容が理解できていない場合は、もう一度、本節を読み、環境構築から始めて、進めていくことで理解が深まる。

.. raw:: latex

   \newpage

