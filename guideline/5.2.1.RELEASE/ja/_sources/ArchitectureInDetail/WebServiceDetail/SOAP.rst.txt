SOAP Web Service（サーバ/クライアント）
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

.. _SOAPOverview:

Overview
--------------------------------------------------------------------------------
本節では、SOAP Web Serviceの基本的な概念とJAX-WSを使用したSOAPサーバ、クライアント双方の開発について説明する。

実装に対する具体的な説明については、

* | 「:ref:`SOAPHowToUse`」
  | JAX-WSを使用したSOAP Web Serviceのアプリケーション構成やAPIの実装方法について説明している。

を参照されたい。

|

.. _SOAPOverviewAboutSOAPWebService:

SOAPとは
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| SOAPとは、XMLで記述されたメッセージをコンピュータ間で送受信を行うためのプロトコルである。
| もともとは「\ **S**\imple \ **O**\bject \ **A**\ccess \ **P**\rotocol」の略であった。
| しかし現在では、「SOAP」はなにかの略ではなく、固有名詞であるとW3Cは宣言している。
| W3CによるSOAP1.1、SOAP1.2の仕様はW3Cにより定義されている。
| 詳細は、\ `W3C -SOAP Specifications- <http://www.w3.org/TR/soap/>`_\を参照されたい。

| 本ガイドラインでは、以下の図のような構成でのSOAP Web Serviceを行う場合を想定して説明する。
| ただし、下記の構成以外でのSOAP Web Serviceの場合にも応用可能である。（例：クライアントがバッチの場合など）

.. figure:: images_SOAP/SOAPServerAndClient.png
    :alt: Server and Client for SOAP
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | クライアントは、別のSOAPサーバへの通信を行うWebアプリケーションを想定している。
        | クライアントと呼んでいるがWebアプリケーション想定なので注意が必要である。
    * - | (2)
      - | SOAPサーバは、Webサービスを公開し、クライアントからのSOAP Web ServiceによるXMLを受信して処理を行う。データベースなどにアクセスを行い、業務処理を行うことを想定している。
    * - | (3)
      - | SOAP Web ServiceではXMLを使用して情報のやり取りを行う。
        | 今回の想定では、SOAPサーバ、クライアントどちらもJavaである想定としているが、他のプラットフォームでも問題なく通信可能である。


|

.. _SOAPOverviewJaxWS:

JAX-WSとは
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| JAX-WSとは、「\ **J**\ava \ **A**\PI for \ **X**\ML-Based \ **W**\eb \ **S**\ervices」の略であり、SOAPなどを使ったWebサービスを扱うためのJava標準APIである。
| JAX-WSを用いることで、JavaのオブジェクトをSOAPの仕様に沿ったXMLに変換して送信することが可能である。
| そのため、SOAP Web Serviceとしては、XMLでやり取りが行われるものの、利用者は、XMLの構造をあまり意識せずデータを扱うことができる。
| Oracle WebLogic ServerやJBoss Enterprise Application Platformなど主要なJava EEサーバはJAX-WS実装をサーバ側で有しており、特別なライブラリを追加せずにその機能を使用して簡単にWebサービスを公開することができる。
| ただし、Tomcatは、JAX-WSを実装していないため、使用する際には別途JAX-WS実装ライブラリを追加する必要がある。
| 詳細は、「\ :ref:`SOAPAppendixTomcatWebService`\」を参照されたい。

|

.. _SOAPOverviewJaxWSSpring:

Spring FrameworkのJAX-WS連携機能について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Spring FrameworkはJAX-WSの連携機能をサポートしており、その機能を使用することでSOAPサーバ、クライアントともに簡単に実装することができる。
| 以下はその機能を用いた、推奨アクセスフローの概要である。ここではSOAPのクライアント(図左)であるWebアプリケーションがSOAPサーバ(図右)にアクセスすることを前提としている。

.. figure:: images_SOAP/SOAPProcessFlow.png
    :alt: Server and Client Projects for SOAP
    :width: 80%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | [クライアント] ControllerがServiceを呼び出す。
        | 通常の呼び出しと変更点は特にない。
    * - | (2)
      - | [クライアント] ServiceがSOAPサーバ提供側が用意したWebServiceインターフェースを呼び出す。
        | この図では、ServiceがWebServiceインターフェースを呼び出しているが、要件に応じてControllerから直接WebServiceインターフェースを呼び出してもよい。
    * - | (3)
      - | [クライアント] WebServiceインターフェースが呼び出されると実体として「動的プロキシ(Dynamic Proxy)」(以下「プロキシ」)が呼び出される。
        | このプロキシは\ ``org.springframework.remoting.jaxws.JaxWsPortProxyFactoryBean``\ が生成したWebServiceインターフェースの実装クラスである。
        | この実装クラスがServiceにインジェクションされ、ServiceはWebServiceインターフェースのメソッドを呼び出すだけで、SOAP Web Serviceを利用した処理を行うことができる。
    * - | (4)
      - | プロキシが、SOAPサーバのWebServiceインターフェースを呼び出す。
        | SOAPサーバとクライアントでの値のやり取りはDomain Objectを使用して行う。
      
        .. Note::

            厳密には、SOAPサーバとクライアントはXMLを使用して通信を行っている。
            送信時、および受信時にはJAXBを使用して、Domain ObjectとXMLの相互変換が行われているが、SOAP Web Service作成者はXMLをあまり意識せず、開発を行うことができるようになっている。
        
    * - | (5)
      - | [サーバ] WebServiceインターフェースが呼び出されると実体としてWebService実装クラスが呼び出される。
        | SOAPサーバでは、WebServiceインターフェースの実装クラスとしてWebService実装クラスを用意する。
        | このWebService実装クラスは、\ ``org.springframework.web.context.support.SpringBeanAutowiringSupport``\を継承することで、SpringのDIコンテナ上のBeanを\ ``@Inject``\などでインジェクションすることができる。
    * - | (6)
      - | [サーバ] WebService実装クラスでは、業務処理を行うServiceを呼び出す。
    * - | (7)
      - | [サーバ] Serviceでは、Repositoryなどを使用して業務処理を実行する。
        | 通常の呼び出しと変更点は特にない。

.. raw:: latex

   \newpage

.. note::

    Springでは、ドキュメントドリブンでWebサービスを開発するSpring Web Servicesをが提供されているが、ここでは扱わない。
    詳細は\ `Spring Web Services <http://projects.spring.io/spring-ws/>`_\ を参照されたい。

.. note::

    SpringでのJAX-WS実装の詳細は、\ `Spring Framework Reference Documentation -Remoting and web services using Spring(Web services)- <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/remoting.html#remoting-web-services>`_\ を参照されたい。

|

.. _SOAPOverviewAboutRESTfulWebServiceDevelopment:

JAX-WSを利用したWebサービスの開発について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| TERASOLUNA Server Framework for Java (5.x)では、APサーバのJAX-WS実装とSpringの機能を利用してWebサービスの開発を行うことを推奨する。


.. Note:: **APサーバへのデプロイについて**

    SOAPサーバ、クライアントどちらにおいても、通常のWebアプリケーション同様に、ブランクプロジェクト内のwebプロジェクトから作成したWARファイルをAPサーバにデプロイすることで、SOAP Web Serviceを実現することができる。


|

.. _SOAPOverviewAboutRESTfulModuleStructure:

JAX-WSを利用したWebサービスのモジュールの構成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

JAX-WSを利用したWebサービスを作成する場合、既存のブランクプロジェクトとは別に以下2つのプロジェクトを追加することを推奨する。

* modelプロジェクト
* webserviceプロジェクト

| modelプロジェクトは、Webサービスの引数や返り値に使用するDomain Objectを格納する。
| webserviceプロジェクトは、Webサービスを呼び出すインターフェースを格納する。
| この2つはSOAPサーバからクライアントに配布する必要があるクラスのみ格納するプロジェクトである。
| 配布する範囲を明確に識別するため、別プロジェクトにすることを推奨している。


本ガイドラインでは、マルチプロジェクトで以下のような構成を用いる。

ここでもクライアントはWebアプリケーションであることを前提とするが、デスクトップアプリケーションやコマンドラインインターフェースから呼び出す場合も基本的な考え方は同じである。

.. figure:: images_SOAP/SOAPClientAndServerProjects.png
    :alt: Server and Client Projects for SOAP
    :width: 80%


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | クライアントを作成する場合、従来のマルチプロジェクトにSOAPサーバから提供されるmodelプロジェクトとwebserviceプロジェクトを追加する。
        | ここではサーバとクライアントをともに開発することを前提としている。
        | これらのプロジェクトの詳細については「\ :ref:`SOAPHowToUseWebApplicationConstruction`\ 」で説明する。
        | 追加方法については「\ :ref:`SOAPAppendixAddProject`\ 」を参照されたい。
        |
        | サーバとクライアントの開発が別々で、modelプロジェクトとwebserviceプロジェクトが提供されない場合、もしくはJava以外でSOAPサーバが作成されている場合には、modelプロジェクト内のDomain Objectとwebserviceプロジェクト内のWebサービスインターフェースを自分で作成する必要がある。
        | wsimportを使用することで、WSDLから簡単にDomain ObjectとWebサービスインターフェースを作成することができる。
        | 詳細については「\ :ref:`SOAPAppendixWsimport`\ 」を参照されたい。
    * - | (2)
      - | SOAPサーバを作成する場合、従来のマルチプロジェクトに追加してmodelプロジェクトとwebserviceプロジェクトを追加する。
        | クライアントにこれら2つのプロジェクトを公開する。
        | クライアントへのmodelプロジェクト、webserviceプロジェクトの公開方法は、Mavenの依存関係への追加を想定している。

|

| 結果として、プロジェクトは次のような構成となる。
| 以下は、SOAPサーバのプロジェクト構成である。

.. figure:: images_SOAP/SOAPServerPackageExplorer.png
    :alt: Package explorer for SOAP server projects
    :width: 50%

以下は、クライアントのプロジェクト構成である。

.. figure:: images_SOAP/SOAPClientPackageExplorer.png
    :alt: Package explorer for SOAP client projects
    :width: 42%


|

Webサービスとして公開されるURL
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^



| SOAP Web Serviceを作成するとWSDL（\ **W**\ eb \ **S**\ ervices \ **D**\ escription \ **L**\ anguage）というWebサービスのインターフェース定義が公開され、クライアントはこの定義をもとにSOAP Web Serviceを実行する。
| WSDLの詳細は、`W3C -Web Services Description Language (WSDL)- <http://www.w3.org/TR/wsdl>`_\を参照されたい。


| WSDL内には、Webサービス実行時のアクセスURLやメソッド名、引数、戻り値などが定義される。
| 本ガイドラインの通りにSOAP Web Serviceを作成すると、以下のURLでWSDLが公開される。
| クライアントではこのURLを指定する必要がある。

* `http://AAA.BBB.CCC.DDD:XXXX/コンテキストルート/Webサービス名?wsdl`
  
WSDL内で定義されるエンドポイントアドレスは以下のURLである。

* `http://AAA.BBB.CCC.DDD:XXXX/コンテキストルート/Webサービス名`


.. Note::

    本ガイドラインでは、マルチプロジェクト構成のwebプロジェクトをWARファイル化して、APサーバにデプロイする前提である。その場合、コンテキストルートは基本的に、[server projectName]-webとなる。ただし、APサーバによって異なるので注意すること。


.. Note::

    本ガイドラインでは、SOAPサーバ、クライアントともにWebアプリケーションとして公開する前提であるため、クライアントではWSDLのURLを指定している。URLではなく、WSDLをファイルとして用意してクライアントを作成することも可能である。
    詳細は、\ :ref:`SOAPHowToUseWebServiceClient`\ を参照されたい。


.. warning::

    本ガイドラインでは、APサーバ（Tomcatの場合は使用するライブラリ）でコンテキストルートのマッピングを切り替え以下のようなURLでアクセスするように設定している。
     
    * `http://AAA.BBB.CCC.DDD:XXXX/[server projectName]-web/ws/TodoWebService?wsdl`
       
    このコンテキストルート直下ではないURLにWebサービスをマッピングさせる方法は、APサーバごとに異なる。
    詳細は以下を参照してほしい。

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.50\linewidth}|p{0.40\linewidth}|
     .. list-table::
         :header-rows: 1
         :widths: 10 50 40
         :class: longtable

         * - 項番
           - APサーバ名
           - 説明
         * - | (1)
           - | Apache Tomcat
           - | \ :ref:`SOAPAppendixTomcatWebService`\
         * - | (2)
           - | Oracle WebLogic Server
           - | TBD
         * - | (3)
           - | JBoss Enterprise Application Platform
           - | TBD

|

.. _SOAPHowToUse:

How to use
--------------------------------------------------------------------------------
本節では、SOAP Web Serviceの具体的な作成方法について説明する。

|

.. _SOAPHowToUseWebApplicationConstruction:

SOAPサーバの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


プロジェクトの構成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**各プロジェクトの依存関係**

| 「\ :ref:`SOAPOverviewAboutRESTfulWebServiceDevelopment`\」で述べたとおり、modelプロジェクトとwebserviceプロジェクトを追加する。
| 追加方法は「\ :ref:`SOAPAppendixAddProject`\ 」を参照されたい。
| またそれに伴い、既存のプロジェクトに依存関係を追加することが必要となる。

.. figure:: images_SOAP/SOAPServerProjects.png
    :alt: Server Projects for SOAP
    :width: 80%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 30 60

    * - 項番
      - プロジェクト名
      - 説明
    * - | (1)
      - | webプロジェクト
      - | Webサービス実装クラスを配置する。
    * - | (2)
      - | domainプロジェクト
      - | WebServiceの実装クラスから呼び出されるServiceを配置する。
        | その他、Repositoryなどは従来と同じである。
    * - | (3)
      - | webserviceプロジェクト
      - | 公開するWebServiceのインターフェースをここに配置する。
        | クライアントはこのインターフェースを使用してWebサービスを実行する。
    * - | (4)
      - | modelプロジェクト
      - | ドメイン層に属するクラスのうち、SOAP Web Serviceで使用するクラスのみをここに配置する。
        | クライアントからの入力値や返却結果はこのプロジェクト内のクラスを使用する。

|

アプリケーションの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**Webサービスを公開する際の初期設定**

| APサーバとしてTomcatを使用する場合は、「\ :ref:`SOAPAppendixTomcatWebService`\」を実施する必要がある。
| その他、APサーバによってWebサービス公開の方法は違うので、詳細は各APサーバのマニュアルを参照されたい。

.. note::
    以下、参考資料として、APサーバのマニュアルを記述しておく。
    必ず、使用するバージョンとあっているか確認してから参照すること。
     
    Oracle WebLogic Server 12.2.1: \ `Oracle(R) Fusion Middleware Understanding WebLogic Web Services for Oracle WebLogic Server  Features and Standards Supported by WebLogic Web Services <https://docs.oracle.com/middleware/1221/wls/WSOVR/weblogic-web-service-stand.htm#WSOVR137>`_\
     
    JBoss Enterprise Application Platform 6.4: \ `DEVELOPMENT GUIDE JAX-WS WEB SERVICES <https://access.redhat.com/documentation/en-US/JBoss_Enterprise_Application_Platform/6.4/html/Development_Guide/chap-JAX-WS_Web_Services.html>`_\

|

**パッケージのコンポーネントスキャン設定**

Webサービスで使用するコンポーネントをスキャンするため、\ ``[server projectName]-ws.xml``\ を作成し、コンポーネントスキャンの定義を行い、Webサービスにインジェクションできるようにする。

*[server projectName]-web/src/main/resources/META-INF/spring/[server projectName]-ws.xml*

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="
             http://www.springframework.org/schema/beans
             http://www.springframework.org/schema/beans/spring-beans.xsd
             http://www.springframework.org/schema/context
             http://www.springframework.org/schema/context/spring-context.xsd">
        <!-- (1) -->
        <context:component-scan base-package="com.example.ws" />
    </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | Webサービスで使用するコンポーネントが格納されているパッケージを指定する。

|

*[server projectName]-web/src/main/webapp/WEB-INF/web.xml*

.. code-block:: xml
    :emphasize-lines: 8

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <!-- Root ApplicationContext -->
        <!-- (1) -->
        <param-value>
            classpath*:META-INF/spring/applicationContext.xml
            classpath*:META-INF/spring/spring-security.xml
            classpath*:META-INF/spring/[server projectName]-ws.xml
        </param-value>
    </context-param>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``[server projectName]-ws.xml``\ をルート\ ``ApplicationContext``\ 生成時の読み込み対象に加える。
 
|

**入力チェックを行うための定義**
 
| 入力チェックにはメソッドバリデーションを使用するため、以下の定義を追加する。
| 入力チェックの詳細は \ :ref:`SOAPHowToUseServerValidation`\を参照されたい。

*[server projectName]-web/src/main/resources/META-INF/spring/applicationContext.xml*

.. code-block:: xml

    <bean class="org.springframework.validation.beanvalidation.MethodValidationPostProcessor">
        <property name="validator" ref="validator" />
    </bean>
 
    <bean id="validator" class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean" />
      
|

.. _SOAPHowToUseWebServiceImpl:

Webサービスの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
以下の作成を行う。

- Domain Objectの作成
- WebServiceインターフェイスの作成
- WebService実装クラスの作成

.. figure:: images_SOAP/SOAPServerClass.png
   :alt: Server Projects for SOAP
   :width: 80%

|

**Domain Objectの作成**

| modelプロジェクト内に、Webサービスの引数や返り値に使用するDomain Objectを作成する。
| \ ``java.io.Serializable``\ インターフェースを実装した一般のJavaBeanと特に変わりはない。

*[server projectName]-model/src/main/java/com/example/domain/model/Todo.java*

.. code-block:: java

    package com.example.domain.model;

    import java.io.Serializable;
    import java.util.Date;

    public class Todo implements Serializable {

        private String todoId;

        private String title;

        private String description;

        private boolean finished;

        private Date createdAt;

        // omitted setter and getter

    }

|

**WebServiceインターフェイスの作成**

webserviceプロジェクト内にWebサービスを呼び出すインターフェースを作成する。

*[server projectName]-webservice/src/main/java/com/example/ws/todo/TodoWebService.java*

.. code-block:: java

    package com.example.ws.todo;

    import java.util.List;

    import javax.jws.WebMethod;
    import javax.jws.WebParam;
    import javax.jws.WebResult;
    import javax.jws.WebService;

    import com.example.domain.model.Todo;
    import com.example.ws.webfault.WebFaultException;

    @WebService(targetNamespace = "http://example.com/todo") // (1)
    public interface TodoWebService {

        @WebMethod // (2)
        @WebResult(name = "todo") // (3)
        Todo getTodo(@WebParam(name = "todoId") /* (4) */ String todoId) throws WebFaultException;

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@WebService``\ を付けることで、WebServiceインターフェースであることを宣言する。
        | \ ``targetNamespace``\ 属性には、名前空間を定義するが、これは作成するWebサービスのパッケージ名と合わせることを推奨する。
          
        .. warning::
            \ ``targetNamespace``\ 属性の値は一意にする必要がある。そのため、ガイドライン上のソースを流用する場合は必ず変更すること。

        .. Note::
            \ ``targetNamespace``\ 属性の値はWSDL上に定義され、このWebサービスの名前空間を決定し、一意に特定するために使用される。
              
    * - | (2)
      - | Webサービスのメソッドとして公開するメソッドに\ ``@WebMethod``\ を付ける。
        | このアノテーションを付けることにより、WSDL上にメソッドが公開され、外部から使用することが可能になる。
    * - | (3)
      - | 返り値に\ ``@WebResult``\ を付け、名前を\ ``name``\ 属性に指定する。返り値がない場合は不要。
        | このアノテーションを付けることにより、WSDL上に返り値として公開される。
    * - | (4)
      - | 引数に\ ``@WebParam``\ を付け、名前を\ ``name``\ 属性に指定する。
        | このアノテーションを付けることにより、WSDL上に引数が公開され、外部から呼び出すときの必要なパラメータとして定義される。
        | \ ``WebFaultException``\ の詳細は「\ :ref:`SOAPHowToUseExceptionHandler`\ 」を参照されたい。


.. note:: **パッケージ名および、ネームスペースの付け方について**

    パッケージ名が以下のような形式になっている場合

      * 【ドメイン】.【アプリケーション名(システム名)】.ws.【ユースケース名】

    本ガイドラインでは、以下のようなネームスペースにすることを推奨する。

      * http://【ドメイン】/【アプリケーション名(システム名)】
      
      
.. note:: **ネームスペースとパッケージ名の関係**

    ドメインをcom.example、アプリケーション名をtodoとした場合、Namespaceは以下のようなJavaのパッケージと紐づけられる。

    .. figure:: images_SOAP/SOAPURL.png
        :alt: Server and Client Projects for SOAP
        :width: 50%

    仕様ではないが、Namespaceとパッケージの命名について、\ `XML Namespace Mapping(Red Hat JBoss Fuse) <https://access.redhat.com/documentation/en-US/Red_Hat_JBoss_Fuse/6.0/html/Developing_Applications_Using_JAX-WS/files/JAXWSDataNamespaceMapping.html>`_\ にまとまっている。

|

    
      
**WebService実装クラスの作成**

webプロジェクト内にWebServiceインターフェースの実装クラスを作成する。

*[server projectName]-web/src/main/java/com/example/ws/todo/TodoWebServiceImpl.java*

.. code-block:: java

    package com.example.ws.todo;

    import java.util.List;

    import javax.inject.Inject;
    import javax.jws.HandlerChain;
    import javax.jws.WebService;
    import javax.xml.ws.BindingType;
    import javax.xml.ws.soap.SOAPBinding;

    import org.springframework.web.context.support.SpringBeanAutowiringSupport;

    import com.example.domain.model.Todo;
    import com.example.domain.service.TodoService;
    import com.example.ws.webfault.WebFaultException;
    import com.example.ws.exception.WsExceptionHandler;
    import com.example.ws.todo.TodoWebService;


    @WebService(
            portName = "TodoWebPort",
            serviceName = "TodoWebService",
            targetNamespace = "http://example.com/todo",
            endpointInterface = "com.example.ws.todo.TodoWebService") // (1)
    @BindingType(SOAPBinding.SOAP12HTTP_BINDING) // (2)
    public class TodoWebServiceImpl extends SpringBeanAutowiringSupport implements TodoWebService { // (3)

        @Inject // (4)
        TodoService todoService;

        @Override // (5)
        public Todo getTodo(String todoId) throws WebFaultException {
            return todoService.getTodo(todoId);
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@WebService``\ を付けることで、WebServiceの実装クラスであることを宣言する。
        | \ ``portName``\ 属性は、WSDL上のポート名として公開される。
        | \ ``serviceName``\ 属性は、WSDL上のサービス名として公開される。
        | \ ``targetNamespace``\ 属性は、WSDL上で使用されるネームスペース。
        | \ ``endpointInterface``\ 属性は、このクラスが実装しているWebサービスのインターフェース名を定義する。

        .. note::
          \ ``TodoWebService``\ インターフェースでは、\ ``@WebService``\ の属性として\ ``portName``\ 属性, \ ``serviceName``\ 属性, \ ``endpointInterface``\ 属性を設定してはいけない。これは、このインターフェースはWSDL上の\ ``portType``\ 要素に対応しており、Webサービスの内容を記述する要素ではないためである。

    * - | (2)
      - | \ ``@BindingType``\ を付けることで、バインディングの方式を設定する。
        | \ ``SOAPBinding.SOAP12HTTP_BINDING``\ を定義するとSOAP1.2でのバインディングとなる。
        | 何もつけない場合は、SOAP1.1でのバインディングとなる。
    * - | (3)
      - | 先ほど作成した\ ``TodoWebService``\ インターフェースを実装する。
        | \ ``org.springframework.web.context.support.SpringBeanAutowiringSupport``\ を継承することで、SpringのBeanをDIできるようにする。
    * - | (4)
      - | Serviceをインジェクションする。
        | 通常のControllerでServiceを呼び出す場合と変わりはない。
    * - | (5)
      - | Serviceを呼び出して業務処理を実行する。
        | 通常のControllerでServiceを呼び出す場合と変わりはない。

.. note::
    Webサービス関連のクラスはwsパッケージ配下にまとめることを推奨する。これは、アプリケーション層のクラスはappパッケージ配下に配置することを推奨しており、それらと区別をしやすくするためである。

|

.. _SOAPHowToUseServerValidation:

入力チェックの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| SOAP Web Serviceにより送信されたパラメータの入力チェックには、Springから提供されているメソッドバリデーションを使用する。
| メソッドバリデーションの詳細については\ :ref:`MethodValidationOnSpringFrameworkHowToUseApplyTarget`\ を参照されたい。
| 以下のように、Serviceのインターフェースに入力チェック内容を定義する。

*[server projectName]-domain/src/main/java/com/example/domain/service/todo/TodoService.java*

.. code-block:: java

    package com.example.domain.service.todo;

    import java.util.List;

    import javax.validation.Valid;
    import javax.validation.constraints.NotNull;
    import javax.validation.groups.Default;

    import org.springframework.validation.annotation.Validated;

    import com.example.domain.model.Todo;

    @Validated // (1)
    public interface TodoService {

        Todo getTodo(@NotNull String todoId); // (2)

        Todo createTodo(@Valid Todo todo); // (3)

        @Validated({ Default.class, Todo.Update.class }) // (4)
        Todo updateTodo(@Valid Todo todo);

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@Validated``\ を付けることで、このインターフェースの実装クラスが入力チェック対象であることを宣言する。
    * - | (2)
      - | 引数をチェックする場合には、引数自体にアノテーションを付ける。
    * - | (3)
      - | JavaBeanの入力チェックを行う場合も、引数に\ ``@Valid``\ を付ける。
    * - | (4)
      - | \ ``@Validated``\ にグループを指定し、特定の条件を絞って入力チェックすることも可能である。
        | グループの詳細は次のJavaBeanの説明で記述する。

|

*[server projectName]-model/src/main/java/com/example/domain/model/Todo.java*

.. code-block:: java

    package com.example.domain.model;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Null;
    import java.io.Serializable;
    import java.util.Date;

    // (1)
    public class Todo implements Serializable {

        // (2)
        public interface Create {
        }

        public interface Update {
        }

        @Null(groups = Create.class)
        @NotNull(groups = Update.class)
        private String todoId;

        @NotNull
        private String title;

        private String description;

        private boolean finished;

        @Null(groups = Create.class)
        private Date createdAt;

        // omitted setter and getter
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | Bean ValidationでJavaBeanの入力チェックを定義する。
        | 詳細は「\ :doc:`../WebApplicationDetail/Validation`\ 」を参照されたい。
    * - | (2)
      - | バリデーションのグループ化を行うために使用するインターフェースを定義する。

|

セキュリティ対策
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**認証処理**

| SOAPの認証・認可方式に関して、本ガイドラインではSpring SecurityでBasic認証を行う方法とServiceでの認可の方法のみ紹介する。
| WS-Securityは扱わない。
| 詳細な利用方法は、「\ :doc:`../../Security/Authentication`\ 」と「\ :doc:`../../Security/Authorization`\ 」を参照されたい。

以下にSOAP Web Serviceに対して、Basic認証を行うSpring Securityの設定例を示す。

*[server projectName]-web/src/main/resources/META-INF/spring/spring-security.xml*

.. code-block:: xml

    <!-- (1) -->
    <sec:http pattern="/ws/**"
              create-session="stateless">
       <sec:csrf disabled="true" />
       <sec:http-basic />
    </sec:http>

    <!-- (2) -->
    <sec:authentication-manager>
       <sec:authentication-provider
           user-service-ref="sampleUserDetailsService">
           <sec:password-encoder ref="passwordEncoder" />
       </sec:authentication-provider>
    </sec:authentication-manager>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``sec:http-basic``\タグを記述するとBasic認証を行うことができる。
        | \ ``pattern``\属性を使用して、Webサービスを実行する部分のみ認証を行う。
    * - | (2)
      - | \ ``authentication-provider``\を利用して、認証方式を定義する。
        | 実際の認証およびユーザ情報取得は\ ``UserDetailsService``\を作成して実施する必要がある。
        | 詳細は「\ :doc:`../../Security/Authentication`\」を参照されたい。

|

**認可処理**

| 認可はServiceごとにアノテーションを付けて行う。
| 詳細は「\ :doc:`../../Security/Authorization`\ 」のアクセス認可(Method)を参照されたい。

*[server projectName]-web/src/main/resources/META-INF/spring/spring-security.xml*

.. code-block:: xml

    <sec:global-method-security pre-post-annotations="enabled" /> <!-- (1) -->

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``<sec:global-method-security>``\ 要素の\ ``pre-post-annotations``\ 属性を\ ``enabled``\ に指定する。

|

*[server projectName]-domain/src/main/java/com/example/domain/service/todo/TodoServiceImpl.java*

.. code-block:: java

    public class TodoServiceImpl implements TodoService {

        // omitted

        // (1)
        @PreAuthorize("isAuthenticated()")
        public List<Todo> getTodos() {
            // omitted
        }

        @PreAuthorize("hasRole('ROLE_ADMIN')")
        public Todo createTodo(Todo todo) {
            // omitted
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 認可処理を行うメソッドに\ ``org.springframework.security.access.prepost.PreAuthorize``\ アノテーションを設定する。

|

**CSRF対策**

| SOAP Web Serviceはセッションを利用せず、ステートレスな通信にすべきである。
| そのため、セッションを利用するCSRF対策を行わないようにするための設定方法について以下に記述する。
| CSRFの詳細は「\ :doc:`../../Security/CSRF`\」を参照されたい。
| ブランクプロジェクトのデフォルトの設定では、CSRF対策が有効化されている。
| そのため、以下の設定を追加し、SOAP Web Serviceのリクエストに対して、CSRF対策の処理が行われないようにする。

*[server projectName]-web/src/main/resources/META-INF/spring/spring-security.xml*

.. code-block:: xml

    <!-- (1) -->
    <sec:http pattern="/ws/**"
        create-session="stateless">
        <sec:http-basic />
        <sec:csrf disabled="true" />
    </sec:http>

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明

    * - | (1)
      - | SOAP Web Service用のSpring Securityの定義を追加する。
        | \ ``<sec:http>``\ 要素の\ ``pattern``\ 属性にSOAP Web Service用のリクエストパスのURLパターンを指定する。
        | このコード例では、\ ``/ws/``\ で始まるリクエストパスをSOAP Web Service用のリクエストパスとしている。
        | また、\ ``create-session``\ 属性を\ ``stateless``\ とする事で、Spring Securityの処理でセッションが使用されなくなる。
        |
        | CSRF対策を無効化するために、\ ``<sec:csrf>``\ 要素の\ ``disabled``\ 属性を\ ``true``\ に指定する。

|

例外ハンドリングの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| SOAPサーバで例外が発生した場合にクライアントへ伝えるためには専用の例外クラスをスローする必要がある。
| その実装を以下に記述する。


**SOAPサーバで発生する例外**

SOAPサーバで発生した例外はこれから記述する例外を実装したクラス（SOAPFault）を使用することで、クライアントへの通知メッセージを決定することができる。
  
具体的には以下のクラスを作成する。
  
.. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 30 60

    * - 項番
      - クラス名
      - 概要
    * - | (1)
      - | \ ``ErrorBean``\
      - | 発生した例外のコードとメッセージなどを保持するクラス。
    * - | (2)
      - | \ ``WebFaultType``\
      - | 例外の種類を判別するために使用する列挙型。
    * - | (3)
      - | \ ``WebFaultBean``\
      - | \ ``ErrorBean``\ と\ ``WebFaultType``\ を保持するクラス。\ ``ErrorBean``\ を\ ``List``\ で保持して例外情報を複数保持できる。
    * - | (4)
      - | \ ``WebFaultException``\
      - | \ ``WebFaultBean``\ を保持する例外クラス。
  
これらの例外はSOAPサーバ、クライアントで共用するため、[server projectName]-webserviceに配置する。

|

*[server projectName]-webservice/src/main/java/com/example/ws/webfault/ErrorBean.java*

.. code-block:: java

    package com.example.ws.webfault;

    public class ErrorBean { // (1)
        private String code;
        private String message;
        private String path;

        // omitted setter and getter
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 例外のメッセージなどを保持するクラスを作成する。

|

*[server projectName]-webservice/src/main/java/com/example/ws/webfault/WebFaultType.java*

.. code-block:: java

    package com.example.ws.webfault;

    public enum WebFaultType { // (2)
        AccessDeniedFault,
        BusinessFault,
        ResourceNotFoundFault,
        ValidationFault,
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 例外の種類を判別するために使用する列挙型を定義する。

|

*[server projectName]-webservice/src/main/java/com/example/ws/webfault/WebFaultBean.java*

.. code-block:: java

    package com.example.ws.webfault;

    import java.util.ArrayList;
    import java.util.List;

    public class WebFaultBean { // (3)

        private WebFaultType type;

        private List<ErrorBean> errors = new ArrayList<ErrorBean>();

        public WebFaultBean(WebFaultType type) {
            this.type = type;
        }

        public void addError(String code, String message) {
            addError(code, message, null);
        }

        public void addError(String code, String message, String path) {
            errors.add(new ErrorBean(code, message, path));
        }

        // omitted setter and getter
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``ErrorBean``\ と\ ``WebFaultType``\ を保持するクラスを作成する。

|

*[server projectName]-webservice/src/main/java/com/example/ws/webfault/WebFaultException.java*

.. code-block:: java

    package com.example.ws.webfault;

    import java.util.List;

    import javax.xml.ws.WebFault;

    @WebFault(name = "WebFault", targetNamespace = "http://example.com/todo") // (1)
    public class WebFaultException extends Exception {
        private WebFaultBean faultInfo; // (2)

        public WebFaultException() {
        }

        public WebFaultException(String message, WebFaultBean faultInfo) {
            super(message);
            this.faultInfo = faultInfo;
        }

        public WebFaultException(String message, WebFaultBean faultInfo, Throwable e) {
            super(message, e);
            this.faultInfo = faultInfo;
        }

        public List<ErrorBean> getErrors() {
            return this.faultInfo.getErrors();
        }

        public WebFaultType getType() {
            return this.faultInfo.getType();
        }
        // omitted setter and getter
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | Exception継承クラスに\ ``@WebFault``\を付けて、SOAPFaultであることを宣言する。
        | \ ``name``\属性には、クライアントに送信するSOAPFaultの\ ``name``\属性を設定する。
        | \ ``targetNamespace``\属性には、使用するネームスペースを設定する。Webサービスと同じにする必要がある。
    * - | (2)
      - | faultInfoをフィールドに保持させるとともに、コード例のように以下のようなコンストラクタとメソッドを持たせる。

        - メッセージ文字列とfaultInfoを引数とするコンストラクタ
        - メッセージ文字列とfaultInfoと原因例外を引数とするコンストラクタ
        - getFaultInfoメソッド

.. Note:: **WebFaultExceptionにRuntimeExceptionではなく、Exceptionを継承させている理由**

    \ ``WebFaultException``\ の親クラスを\ ``RuntimeException``\ にすれば、例外の処理をもっと簡略化することができそうに見える。しかし、親クラスを\ ``RuntimeException``\ にしてはいけない。\ `JSR 224: JavaTM API for XML-Based Web Services <https://jcp.org/en/jsr/detail?id=224>`_\ でも明確にしてはいけないと宣言されている。実際に試してみても、APサーバのJAX-WS実装次第ではあるが、クライアントで\ ``@WebFault``\ を付けた例外クラス（\ ``WebFaultException``\ ）を取得することができず、エラーの種類やメッセージを取得することができなくなる。AOPを使用して例外処理を実施していないのも\ ``Exception``\ を継承しているためである。

.. warning:: **WebFaultExceptionのコンストラクタとフィールドについて**

    \ ``WebFaultException``\ には、デフォルトコンストラクタと各フィールドに対応するsetterが必須となる。これは、クライアントの内部処理で、\ ``WebFaultException``\ を作成する際に使用するためである。そのため、各フィールドをfinalにすることも不可能である。
  
  
|

**発生する例外をSOAPFaultでラップする例外ハンドラー**


Serviceから発生する実行時例外をSOAPFaultでラップするために例外ハンドラークラスを作成する。
本ガイドラインではWebService実装クラスがこのハンドラーを用いて例外を変換してスローする方針とする。

Serviceからスローされる例外は以下を想定している。必要に応じて追加されたい。

.. tabularcolumns:: |p{0.60\linewidth}|p{0.40\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 60 40

    * - 例外名
      - 内容
    * - | \ ``org.springframework.security.access.AccessDeniedException``\		
      - | 認可エラー時の例外
    * - | \ ``javax.validation.ConstraintViolationException``\
      - | 入力チェックエラー時の例外
    * - | \ ``org.terasoluna.gfw.common.exception.ResourceNotFoundException``\
      - | リソースが見つからない場合の例外
    * - | \ ``org.terasoluna.gfw.common.exception.BusinessException``\
      - | 業務例外


*[server projectName]-web/src/main/java/com/example/ws/exception/WsExceptionHandler.java*

.. code-block:: java

    package com.example.ws.exception;

    import java.util.Iterator;
    import java.util.Locale;
    import java.util.Set;

    import javax.inject.Inject;
    import javax.validation.ConstraintViolation;
    import javax.validation.ConstraintViolationException;
    import javax.validation.Path;

    import org.springframework.context.MessageSource;
    import org.springframework.security.access.AccessDeniedException;
    import org.springframework.stereotype.Component;
    import org.terasoluna.gfw.common.exception.BusinessException;
    import org.terasoluna.gfw.common.exception.ExceptionCodeResolver;
    import org.terasoluna.gfw.common.exception.ExceptionLogger;
    import org.terasoluna.gfw.common.exception.ResourceNotFoundException;
    import org.terasoluna.gfw.common.exception.SystemException;
    import org.terasoluna.gfw.common.message.ResultMessage;
    import org.terasoluna.gfw.common.message.ResultMessages;

    import com.example.ws.webfault.WebFaultBean;
    import com.example.ws.webfault.WebFaultException;
    import com.example.ws.webfault.WebFaultType;

    @Component  // (1)
    public class WsExceptionHandler {

        @Inject
        MessageSource messageSource; // (2)

        @Inject
        ExceptionCodeResolver exceptionCodeResolver; // (3)

        @Inject
        ExceptionLogger exceptionLogger; // (4)

        // (5)
        public void translateException(Exception e) throws WebFaultException {
            loggingException(e);
            WebFaultBean faultInfo = null;

            if (e instanceof AccessDeniedException) {
                faultInfo = new WebFaultBean(WebFaultType.AccessDeniedFault);
                faultInfo.addError(e.getClass().getName(), e.getMessage());
            } else if (e instanceof ConstraintViolationException) {
                faultInfo = new WebFaultBean(WebFaultType.ValidationFault);
                this.addErrors(faultInfo, ((ConstraintViolationException) e).getConstraintViolations());
            } else if (e instanceof ResourceNotFoundException) {
                faultInfo = new WebFaultBean(WebFaultType.ResourceNotFoundFault);
                this.addErrors(faultInfo, ((ResourceNotFoundException) e).getResultMessages());
            } else if (e instanceof BusinessException) {
                faultInfo = new WebFaultBean(WebFaultType.BusinessFault);
                this.addErrors(faultInfo, ((BusinessException) e).getResultMessages());
            } else {
                // not translate.
                throw new SystemException("e.ex.fw.9001", e);
            }

            throw new WebFaultException(e.getMessage(), faultInfo, e.getCause());
        }

        private void loggingException(Exception e) {
            exceptionLogger.log(e);
        }

        private void addErrors(WebFaultBean faultInfo, Set<ConstraintViolation<?>> constraintViolations) {
            for (ConstraintViolation<?> v : constraintViolations) {
                Iterator<Path.Node> pathIt = v.getPropertyPath().iterator();
                pathIt.next(); // method name node (skip)
                Path.Node methodArgumentNameNode = pathIt.next();
                faultInfo.addError(
                    v.getConstraintDescriptor().getAnnotation().annotationType().getSimpleName(),
                    v.getMessage(),
                    pathIt.hasNext() ? pathIt.next().toString() : methodArgumentNameNode.toString());
            }

        }

        private void addErrors(WebFaultBean faultInfo, ResultMessages resultMessages) {
            Locale locale = Locale.getDefault();
            for (ResultMessage message : resultMessages) {
                faultInfo.addError(
                    message.getCode(),
                    messageSource.getMessage(message.getCode(), message.getArgs(), message.getText(), locale));
            }
        }

    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 本クラスをDIコンテナに管理をさせるため、\ ``@Component``\ を付ける。
    * - | (2)
      - | 出力するメッセージを取得するために\ ``MessageSource``\ を使用する。
    * - | (3)
      - | 共通ライブラリが提供する\ ``ExceptionCodeResolverMessageSource``\ を使用して例外の種類と例外コードをマッピングする。
        | 詳細は、「\ :doc:`../WebApplicationDetail/ExceptionHandling`\」を参照されたい。
    * - | (4)
      - | 共通ライブラリが提供する\ ``ExceptionLogger``\ を使用して例外情報を例外に出力する。
        | 詳細は、「\ :doc:`../WebApplicationDetail/ExceptionHandling`\ 」を参照されたい。
    * - | (5)
      - | Serviceから発生しうる各例外について、\ ``SOAPFault``\へのラップを行う。
        | 例外のマッピングは冒頭の表を参考されたい。

.. note:: **その他の例外の扱いについて**

    その他の例外発生時（上記の\ ``translateException``\ メソッドのelse部分）では、クライアントでは詳細な例外の内容は通知されず、\ ``com.sun.xml.internal.ws.fault.ServerSOAPFaultException``\ が発生するのみとなる。他の例外同様にラップしてクライアント側に通知することも可能である。

|

**Serviceで発生した例外をWebサービス内から例外ハンドラーを呼び出し、ラップする**

Webサービスクラスにて、例外ハンドラーを呼び出す。以下はその例である。

*[server projectName]-web/src/main/java/com/example/ws/todo/TodoWebServiceImpl.java*

.. code-block:: java


    @WebService(
            portName = "TodoWebPort",
            serviceName = "TodoWebService",
            targetNamespace = "http://example.com/todo",
            endpointInterface = "com.example.ws.todo.TodoWebService")
    @BindingType(SOAPBinding.SOAP12HTTP_BINDING)
    public class TodoWebServiceImpl extends SpringBeanAutowiringSupport implements TodoWebService {
        @Inject
        TodoService todoService;
        @Inject
        WsExceptionHandler handler; // (1)

        @Override
        public Todo getTodo(String todoId) throws WebFaultException /* (2) */ {
            try {
                return todoService.getTodo(todoId);
            } catch (RuntimeException e) {
                handler.translateException(e); // (3)
            }
        }
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 例外ハンドラーをインジェクションする。
    * - | (2)
      - | \ ``WebFaultException``\ にラップしてスローするため、throws句を付ける。
    * - | (3)
      - | 実行時例外が発生した場合は、例外ハンドラークラスに処理を委譲する。

|

MTOMを利用した大容量のバイナリデータを扱う方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| SOAPでは、バイナリデータを扱う場合、Byte配列にマッピングすることで、送受信を行うことができる。
| ただし、大容量のバイナリデータを扱う場合、ヒープが枯渇するなどの問題が発生することがある。
| そこで、MTOM（Message Transmission Optimization Mechanism）に準拠した実装を行うことで、最適化した状態で添付ファイルとしてバイナリデータを扱うことができる。
| 詳細な定義は `W3C -SOAP Message Transmission Optimization Mechanism- <http://www.w3.org/TR/soap12-mtom/>`_\ を参照されたい。
| 以下にその方法を記述する。

*[server projectName]-webservice/src/main/java/com/example/ws/todo/TodoWebService.java*

.. code-block:: java

    package com.example.ws.todo;

    import java.util.List;

    import javax.activation.DataHandler;
    import javax.jws.WebMethod;
    import javax.jws.WebParam;
    import javax.jws.WebResult;
    import javax.jws.WebService;
    import javax.xml.bind.annotation.XmlMimeType;

    import com.example.domain.model.Todo;
    import com.example.ws.webfault.WebFaultException;

    @WebService(targetNamespace = "http://example.com/todo")
    public interface TodoWebService {

        // omitted

        @WebMethod
        void uploadFile(@XmlMimeType("application/octet-stream") /* (1) */ DataHandler dataHandler) throws WebFaultException;

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | バイナリデータを処理する\ ``javax.activation.DataHandler``\ に対して\ ``@XmlMimeType``\ を付ける。

|

*[server projectName]-web/src/main/java/com/example/ws/todo/TodoWebServiceImpl.java*

.. code-block:: java

    package com.example.ws.todo;

    import java.io.IOException;
    import java.io.InputStream;
    import java.util.List;

    import javax.activation.DataHandler;
    import javax.inject.Inject;
    import javax.jws.HandlerChain;
    import javax.jws.WebService;
    import javax.xml.ws.BindingType;
    import javax.xml.ws.soap.MTOM;
    import javax.xml.ws.soap.SOAPBinding;

    import org.springframework.web.context.support.SpringBeanAutowiringSupport;
    import org.terasoluna.gfw.common.exception.SystemException;

    import com.example.domain.model.Todo;
    import com.example.domain.service.TodoService;
    import com.example.ws.webfault.WebFaultException;
    import com.example.ws.exception.WsExceptionHandler;

    // (1)
    @MTOM
    @WebService(
            portName = "TodoWebPort",
            serviceName = "TodoWebService",
            targetNamespace = "http://example.com/todo",
            endpointInterface = "com.example.ws.todo.TodoWebService")
    @BindingType(SOAPBinding.SOAP12HTTP_BINDING)
    public class TodoWebServiceImpl extends SpringBeanAutowiringSupport implements TodoWebService {

        @Inject
        TodoService todoService;

        // omitted

        @Override
        public void uploadFile(DataHandler dataHandler) throws WebFaultException {

            try (InputStream inputStream = dataHandler.getInputStream()){ // (2)
                todoService.uploadFile(inputStream);
            } catch (Exception e) {
                handler.translateException(e);
            }
        }

    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@MTOM``\を付けて、MTOMに準拠した実装を使用することを宣言する。
    * - | (2)
      - | \ ``javax.activation.DataHandler``\から\ ``java.io.InputStream``\を取得してファイルを扱う。

|

クライアントの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


プロジェクトの構成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

「\ :ref:`SOAPOverviewAboutRESTfulWebServiceDevelopment`\」で述べたとおり、modelプロジェクトとwebserviceプロジェクトをSOAPサーバから受領する前提である。

.. figure:: images_SOAP/SOAPClientProjects.png
    :alt: Client Projects for SOAP
    :width: 80%


.. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 30 60
    :class: longtable

    * - 項番
      - プロジェクト名
      - 説明
    * - | (1)
      - | webプロジェクト
      - | Controllerを作成する。
        | 通常の画面遷移時のControllerと特に変更点はない。
    * - | (2)
      - | domainプロジェクト
      - | Serviceクラスからwebserviceプロジェクトで用意されたWebServeインターフェースを使用してWebサービスを呼び出す。
        | SOAPサーバと通信する際に使用するWebServiceインターフェースを実装したプロキシを定義する。
    * - | (3)
      - | webserviceプロジェクト
      - | SOAPサーバと同じ資材を配置する。
        | クライアントはこのインターフェースを使用してWebサービスを実行する。
    * - | (4)
      - | modelプロジェクト
      - | SOAPサーバと同じ資材を配置する。
        | SOAPサーバに渡す入力値や返却結果はこのプロジェクト内のクラスを使用する。
    * - | (5)
      - | envプロジェクト
      - | domainプロジェクトで定義したプロキシの環境依存する値を定義する。
        | プロキシの定義から環境依存する値をプロパティファイルに集約し、プロパティファイルのみenvプロジェクトに配置する。

.. raw:: latex

   \newpage

.. note:: **プロキシの定義ついて**

   試験用SOAPサーバ、本番用SOAPサーバ等、複数環境向けのプロキシを定義する際に発生する重複部分を排除し、管理を容易にするために、当ガイドラインではプロキシの定義はdomainプロジェクトで行い、環境依存する値はプロパティファイルに集約、プロパティファイルのみenvプロジェクトに配置することを推奨する。
    
   ユニットテストでプロキシのスタブやモックを使用する場合は、ユニットテスト用のコンポーネントを定義するためのBean定義ファイル(test-context.xml)にBeanを定義する。
   
|

.. _SOAPHowToUseWebServiceClient:

Webサービス クライアントの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
以下のクラスの実装を行う。

- WebServiceインターフェースを実装したプロキシの定義
- ServiceクラスからWebServiceインターフェース経由でWebサービスを呼び出す。

.. figure:: images_SOAP/SOAPClientClass.png
    :alt: Server Projects for SOAP
    :width: 80%


**WebServiceインターフェースを実装したプロキシの作成**

WebServiceインターフェースを実装したプロキシを生成する\ ``org.springframework.remoting.jaxws.JaxWsPortProxyFactoryBean``\の定義を行う。

*[client projectName]-domain/src/main/resources/META-INF/spring/[client projectName]-domain.xml*

.. code-block:: xml

    <bean id="todoWebService"
        class="org.springframework.remoting.jaxws.JaxWsPortProxyFactoryBean"><!-- (1) -->
        <property name="serviceInterface" value="com.example.ws.todo.TodoWebService" /><!-- (2) -->
        <!-- (3) -->
        <property name="serviceName" value="TodoWebService" />
        <property name="portName" value="TodoWebPort" />
        <property name="namespaceUri" value="http://example.com/todo" />
        <property name="wsdlDocumentResource" value="${webservice.todoWebService.wsdlDocumentResource}" /><!-- (4) -->
        <property name="lookupServiceOnStartup" value="false" /><!-- (5) -->
    </bean>

*[client projectName]-env/src/main/resources/META-INF/spring/[client projectName]-infra.properties*

.. code-block:: properties

    # (6)
    webservice.todoWebService.wsdlDocumentResource=http://AAA.BBB.CCC.DDD:XXXX/[server projectName]-web/ws/TodoWebService?wsdl

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``org.springframework.remoting.jaxws.JaxWsPortProxyFactoryBean``\ を定義する。このクラスが生成するプロキシを経由してSOAPサーバにアクセスできる。
    * - | (2)
      - | \ ``serviceInterface``\ プロパティに本来このWebサービスが実装すべきインターフェースを定義する。
    * - | (3)
      - | \ ``serviceName``\ 、\ ``portName``\ 、\ ``namespaceUri``\ プロパティにそれぞれSOAPサーバ側で定義している同じ内容を定義する必要がある。
    * - | (4)
      - | \ ``wsdlDocumentResource``\ プロパティに公開されているWDSLのURLを設定する。
        | ここでは後述するプロパティファイルにURLを記述するため、プロパティのキーを指定している。
    * - | (5)
      - | \ ``lookupServiceOnStartup``\ プロパティにBean生成する時、SOAPサーバからWSDLファイルを取得するかどうかを設定する。falseの場合はBeanが初めて使用されるタイミングでWSDLファイルの取得が行われる。
        | SOAPサーバからWSDLファイルの取得が不可能な場合でもWebサービス クライアントを起動させるために、\ ``lookupServiceOnStartup``\ プロパティに\ ``false``\を指定することを推奨する。ただし、WSDLファイルをWebサービス クライアントで保持している場合は設定不要である。
    * - | (6)
      - | \ ``[client projectName]-domain.xml``\ で定義したプロパティのキーの値を設定する。WSDLのURLを記述する。

        .. Note:: **wsdlDocumentResourceへのWSDLファイルのURL以外の指定**

            上記の例では、SOAPサーバがWSDLファイルを公開している前提である。\ ``classpath:``\ や\ ``file:``\ プレフィックスを使用して指定することで静的ファイルを指定することもできる。指定できる文字列は、\ `Spring Framework Reference Documentation -Resources(The ResourceLoader)- <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/resources.html#resources-resourceloader>`_\ を参照されたい。


.. Note:: **エンドポイントアドレスの上書き指定**

    WSDLファイルには、Webサービス実行時のアクセスURL（エンドポイントアドレス）が記述されているため、クライアントではアクセスURLの設定は不要である。
    ただし、WSDLファイルに記述されているURLではないURLにアクセスする場合、\ ``org.springframework.remoting.jaxws.JaxWsPortProxyFactoryBean``\の\ ``endpointAddress``\ プロパティを設定することで上書きすることができる。
    テストなどで、環境を切り替える場合に使用するとよい。
    以下はその設定例である。

    *[client projectName]-domain/src/main/resources/META-INF/spring/[client projectName]-domain.xml*

     .. code-block:: xml

         <bean id="todoWebService"
             class="org.springframework.remoting.jaxws.JaxWsPortProxyFactoryBean">
             <property name="serviceInterface" value="com.example.ws.todo.TodoWebService" />
             <property name="serviceName" value="TodoWebService" />
             <property name="portName" value="TodoWebPort" />
             <property name="namespaceUri" value="http://example.com/todo" />
             <property name="wsdlDocumentResource" value="${webservice.todoWebService.wsdlDocumentResource}" />
             <property name="endpointAddress" value="${webservice.todoWebService.endpointAddress}" /><!-- (1) -->
             <property name="lookupServiceOnStartup" value="false" />
         </bean>

    *[client projectName]-env/src/main/resources/META-INF/spring/[client projectName]-infra.properties*

     .. code-block:: properties

         # (2)
         webservice.todoWebService.endpointAddress=http://AAA.BBB.CCC.DDD:XXXX/[server projectName]-web/ws/TodoWebService


     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
         :header-rows: 1
         :widths: 10 90
         :class: longtable

         * - 項番
           - 説明
         * - | (1)
           - | エンドポイントアドレスを設定する。
             | ここでは後述するプロパティファイルにURLを記述するため、プロパティのキーを指定している。
         * - | (2)
           - | \ ``[client projectName]-domain.xml``\ で定義したプロパティのキーの値を設定する。エンドポイントアドレスを記述する。

|

**ServiceからWebサービスを呼び出す**

上記で作成したWebサービスをServiceでインジェクションして実行する。


*[client projectName]-domain/src/main/java/com/example/domain/service/todo/TodoServiceImpl.java*

.. code-block:: java

    package com.example.soap.domain.service.todo;

    import java.util.List;

    import javax.inject.Inject;

    import org.springframework.stereotype.Service;

    import com.example.domain.model.Todo;
    import com.example.ws.webfault.WebFaultException;
    import com.example.ws.todo.TodoWebService;

    @Service
    public class TodoServiceImpl implements TodoService {

        @Inject
        TodoWebService todoWebService;

        @Override
        public void createTodo(Todo todo) {
            // (1)
            try {
                todoWebService.createTodo(todo);
            } catch (WebFaultException e) {
                // (2)
                // handle exception…
            }
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明

    * - | (1)
      - | \ ``TodoWebService``\ をインジェクションして、実行対象のServiceを呼び出す。
    * - | (2)
      - | サーバ側で、例外が発生した場合は、\ ``WebFaultException``\ にラップされて送信される。
        | 内容に応じて処理を行う。
        | 例外処理の詳細は「:ref:`SOAPHowToUseExceptionHandler`」を参照されたい。

.. note:: **レスポンスの情報取得**

    リトライを考慮するなど、レスポンスの情報をクライアントで取得したい場合、以下のように\ ``javax.xml.ws.BindingProvider``\クラスにキャストすることで取得できる。

     .. code-block:: java

         BindingProvider provider = (BindingProvider) todoWebService;
         int status = (int) provider.getResponseContext().get(MessageContext.HTTP_RESPONSE_CODE);

    \ ``BindingProvider``\ の詳細については \ `The Java API for XML-Based Web Services(JAX-WS) 2.2 -4.2 javax.xml.ws.BindingProvider- <http://download.oracle.com/otn-pub/jcp/jaxws-2.2-mrel3-evalu-oth-JSpec/jaxws-2_2-mrel3-spec.pdf>`_\ を参照されたい。
    
    ただし、クライアントの依存関係にApatch CXFライブラリが含まれる場合、通信エラー時に上記の方法でレスポンスの情報を取得することができない。
    これは、依存関係にApatch CXFライブラリが含まれる場合は自動的にApatch CXFのプロキシが使用されるため、およびApache CXFのプロキシは通信エラーが発生した場合にレスポンスの情報をレスポンスコンテキストに保持しないためである。
    Apache CXFのエラー処理については\ `Apache CXF Software Architecture Guide -Fault Handling- <http://cxf.apache.org/docs/cxf-architecture.html#CXFArchitecture-FaultHandling>`_\を参照されたい。

    Webサービスと別のWebサービスへのクライアントを持つ中継サービスのように、どうしてもクライアントにApache CXFライブラリの依存関係を含んでしまう場合は制限事項として注意されたい。

|

セキュリティ対策
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
**認証処理**

\ ``org.springframework.remoting.jaxws.JaxWsPortProxyFactoryBean``\を使用している場合でBasic認証を使用しているSOAPサーバと通信をする場合には、bean定義にユーザ名とパスワードを追加するだけで認証を行うことができる。

*[client projectName]-domain/src/main/resources/META-INF/spring/[client projectName]-domain.xml*

.. code-block:: xml
    :emphasize-lines: 8-10

    <bean id="todoWebService"
        class="org.springframework.remoting.jaxws.JaxWsPortProxyFactoryBean">
        <property name="serviceInterface" value="com.example.ws.todo.TodoWebService" />
        <property name="serviceName" value="TodoWebService" />
        <property name="portName" value="TodoWebPort" />
        <property name="namespaceUri" value="http://example.com/todo" />
        <property name="wsdlDocumentResource" value="${webservice.todoWebService.wsdlDocumentResource}" />
        <!-- (1) -->
        <property name="username" value="${webservice.todoWebService.username}" />
        <property name="password" value="${webservice.todoWebService.password}" />
    </bean>

*[client projectName]-env/src/main/resources/META-INF/spring/[client projectName]-infra.properties*

.. code-block:: properties

    # (2)
    webservice.todoWebService.username=testuser
    webservice.todoWebService.password=password

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``org.springframework.remoting.jaxws.JaxWsPortProxyFactoryBean``\のbean定義にusernameとpasswordを加えることでBasic認証における、認証情報を送信することができる。
        | ユーザ名とパスワードをプロパティファイルに切り出した場合のサンプルである。
    * - | (2)
      - | \ ``[client projectName]-domain.xml``\ で定義したプロパティのキーの値を設定する。認証に使用するユーザ名とパスワードを記述する。

|

.. _SOAPHowToUseExceptionHandler:

例外ハンドリングの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| SOAPサーバでは、\ ``WebFaultException``\ に例外をラップして、スローすることを推奨している。
| クライアントは\ ``WebFaultException``\ をキャッチして、その原因例外を判定してそれぞれの処理を行う。

.. code-block:: java
    :emphasize-lines: 8-19

    @Override
    public void createTodo(Todo todo) {

        try {
            // (1)
            todoWebService.createTodo(todo);
        } catch (WebFaultException e) {
            // (2)
            switch (e.getFaultInfo().getType()) {
            case ValidationFault:
                // handle exception…
                break;
            case BusinessFault:
                // handle exception…
                break;
            default:
                // handle exception…
                break;
            }
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | Webサービスを呼び出す。throwsがついているため、\ ``WebFaultException``\ をキャッチする必要がある。
    * - | (2)
      - | \ ``faultInfo``\ の種別で例外を判定し、それぞれの処理を記述する（画面にメッセージを出す、例外をスローするなど）

|

タイムアウトの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
クライアントで指定できるタイムアウトは大きく以下の2つがある。

- SOAPサーバとのコネクションタイムアウト
- SOAPサーバへのリクエストタイムアウト

| どちらの設定も、\ ``org.springframework.remoting.jaxws.JaxWsPortProxyFactoryBean``\ のカスタムプロパティに指定する必要がある。
| 設定の方法は以下の通りである。

*[client projectName]-domain/src/main/resources/META-INF/spring/[client projectName]-domain.xml*

.. code-block:: xml
    :emphasize-lines: 9-16

    <bean id="todoWebService"
        class="org.springframework.remoting.jaxws.JaxWsPortProxyFactoryBean">
        <property name="serviceInterface" value="com.example.ws.todo.TodoWebService" />
        <property name="serviceName" value="TodoWebService" />
        <property name="portName" value="TodoWebPort" />
        <property name="namespaceUri" value="http://example.com/todo" />
        <property name="wsdlDocumentResource" value="${webservice.todoWebService.wsdlDocumentResource}" />
        <!-- (1) -->
        <property name="customProperties">
            <map>
                <!-- (2) -->
                <entry key="com.sun.xml.internal.ws.connect.timeout" value-type="java.lang.Integer" value="${webservice.connect.timeout}"/>
                <entry key="com.sun.xml.internal.ws.request.timeout" value-type="java.lang.Integer" value="${webservice.request.timeout}"/>
            </map>
        </property>
    </bean>

*[client projectName]-env/src/main/resources/META-INF/spring/[client projectName]-infra.properties*

.. code-block:: properties

    # (3)
    webservice.request.timeout=3000
    webservice.connect.timeout=3000

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``customProperties``\ プロパティにMapを指定することでカスタムプロパティを定義する。
    * - | (2)
      - | コネクションタイムアウトとリクエストタイムアウトを定義する。
        | それぞれの値をプロパティファイルに切り出した場合のサンプルである。

        .. warning:: **タイムアウト定義に使用するキーについて**

            それぞれのタイムアウトを定義するキーはJAX-WSの実装により異なる値を設定する必要がある。
            詳細は\ `JAX_WS-1166 Standardize timeout settings <https://java.net/jira/browse/JAX_WS-1166>`_\を参照されたい。

    * - | (3)
      - | \ ``[client projectName]-domain.xml``\ で定義したプロパティのキーの値を設定する。コネクションタイムアウトとリクエストタイムアウトを記述する。


|

Appendix
--------------------------------------------------------------------------------

.. _SOAPAppendixAddProject:

SOAPサーバ用にプロジェクトの設定を変更する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| SOAPサーバを作成する場合、ブランクプロジェクトにmodelプロジェクトとwebserviceプロジェクトを追加することを推奨する。
| 以下にその方法を記述する。

| ブランクプロジェクトは初期状態は以下の構成になっている。
| なお、artifactIdにはブランクプロジェクト作成時に指定したartifactIdが設定される。

.. code-block:: console

    artifactId
    ├── pom.xml
    ├── artifactId-domain
    ├── artifactId-env
    ├── artifactId-initdb
    ├── artifactId-selenium
    └── artifactId-web

以下のようなプロジェクト構成にする。

.. code-block:: console

    artifactId
    ├── pom.xml
    ├── artifactId-domain
    ├── artifactId-env
    ├── artifactId-initdb
    ├── artifactId-selenium
    ├── artifactId-web
    ├── artifactId-model
    └── artifactId-webservice

|

既存プロジェクトの変更
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| ブランクプロジェクトの初期状態では、ControllerなどWebアプリケーションの簡易実装が含まれている。
| そのままにしてもSOAP Web Serviceは実現可能だが、不要であるため、削除することを推奨する。
| 削除対象は、「:doc:`../../ImplementationAtEachLayer/CreateWebApplicationProject` の :ref:`CreateWebApplicationProjectConfigurationMulti`」を参照されたい。

|

modelプロジェクトの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

modelプロジェクトの構成について説明する。

.. code-block:: console

    artifactId-model
        ├── pom.xml  ... (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (1)
      - modelモジュールの構成を定義するPOM(Project Object Model)ファイル。
        このファイルでは、以下の定義を行う。

        * 依存ライブラリとビルド用プラグインの定義
        * jarファイルを作成するための定義

| \ ``pom.xml``\ は以下のようなイメージになる。必要に応じて編集する必要がある。
| 実際には、「artifactId」と「groupId」はブランクプロジェクト作成時に指定した値を設定する必要がある。

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

        <modelVersion>4.0.0</modelVersion>
        <artifactId>artifactId-model</artifactId>
        <packaging>jar</packaging>
        <parent>
            <groupId>groupId</groupId>
            <artifactId>artifactId</artifactId>
            <version>1.0.0-SNAPSHOT</version>
            <relativePath>../pom.xml</relativePath>
        </parent>
        <dependencies>
            <!-- == Begin TERASOLUNA == -->
            <dependency>
                <groupId>org.terasoluna.gfw</groupId>
                <artifactId>terasoluna-gfw-common-dependencies</artifactId>
                <type>pom</type>
            </dependency>
            <dependency>
                <groupId>org.terasoluna.gfw</groupId>
                <artifactId>terasoluna-gfw-jodatime-dependencies</artifactId>
                <type>pom</type>
            </dependency>
            <dependency>
                <groupId>org.terasoluna.gfw</groupId>
                <artifactId>terasoluna-gfw-security-core-dependencies</artifactId>
                <type>pom</type>
            </dependency>

            <dependency>
                <groupId>org.terasoluna.gfw</groupId>
                <artifactId>terasoluna-gfw-recommended-dependencies</artifactId>
                <type>pom</type>
            </dependency>
            <!-- == End TERASOLUNA == -->
        </dependencies>
    </project>

|

webserviceプロジェクトの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

webserviceプロジェクトの構成について説明する。

.. code-block:: console

    artifactId-webservice
        ├── pom.xml  ... (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (1)
      - webserviceモジュールの構成を定義するPOM(Project Object Model)ファイル。
        このファイルでは、以下の定義を行う。

        * 依存ライブラリとビルド用プラグインの定義
        * jarファイルを作成するための定義

| \ ``pom.xml``\ は以下のようなイメージになる。必要に応じて編集する必要がある。
| 実際には、「artifactId」と「groupId」はブランクプロジェクト作成時に指定した値を設定する必要がある。

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

        <modelVersion>4.0.0</modelVersion>
        <artifactId>artifactId-webservice</artifactId>
        <packaging>jar</packaging>
        <parent>
            <groupId>groupId</groupId>
            <artifactId>artifactId</artifactId>
            <version>1.0.0-SNAPSHOT</version>
            <relativePath>../pom.xml</relativePath>
        </parent>
        <dependencies>
            <dependency>
                <groupId>${project.groupId}</groupId>
                <artifactId>artifactId-model</artifactId>
            </dependency>
            <!-- == Begin TERASOLUNA == -->
            <dependency>
                <groupId>org.terasoluna.gfw</groupId>
                <artifactId>terasoluna-gfw-common-dependencies</artifactId>
                <type>pom</type>
            </dependency>
            <dependency>
                <groupId>org.terasoluna.gfw</groupId>
                <artifactId>terasoluna-gfw-jodatime-dependencies</artifactId>
                <type>pom</type>
            </dependency>
            <dependency>
                <groupId>org.terasoluna.gfw</groupId>
                <artifactId>terasoluna-gfw-security-core-dependencies</artifactId>
                <type>pom</type>
            </dependency>

            <dependency>
                <groupId>org.terasoluna.gfw</groupId>
                <artifactId>terasoluna-gfw-recommended-dependencies</artifactId>
                <type>pom</type>
            </dependency>
            <!-- == End TERASOLUNA == -->
        </dependencies>
    </project>

|

.. _SOAPAppendixPackageServer:

SOAPサーバのパッケージ構成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| SOAPサーバを作成するときの推奨する構成について、説明する。
| ガイドラインに従いプロジェクトを追加すると以下の構成となる。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - プロジェクト名
      - 説明
    * - | [server projectName]-domain
      - | SOAPサーバのドメイン層に関するクラス・設定ファイルを格納するプロジェクト
    * - | [server projectName]-web
      - | SOAPサーバのアプリケーション層に関するクラス・設定ファイルを格納するプロジェクト
    * - | [server projectName]-env
      - | SOAPサーバの環境に依存するファイル等を格納するプロジェクト
    * - | [server projectName]-model
      - | SOAPサーバのドメイン層に関するクラスの中で、Webサービス実行時に使用し、クライアントと共有するクラスを格納するプロジェクト
    * - | [server projectName]-webservice
      - | SOAPサーバが提供するWebサービスのインターフェースを格納するプロジェクト

|


[server projectName]-domain
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

[server projectName]-modelの依存関係を追加するため、\ ``pom.xml``\ に以下を追加する。

.. code-block:: xml
      
    <dependency>
        <groupId>${project.groupId}</groupId>
        <artifactId>artifactId-model</artifactId>
    </dependency>

その他のパッケージ構成は、通常のdomainプロジェクトと変わらないため、「:doc:`../../Overview/ApplicationLayering` の :ref:`application-layering_project-structure`」を参照されたい。

|

[server projectName]-web
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

[server projectName]-webserviceの依存関係を追加するため、\ ``pom.xml``\ に以下を追加する。

.. code-block:: xml

    <dependency>
        <groupId>${project.groupId}</groupId>
        <artifactId>artifactId-webservice</artifactId>
    </dependency>

.. note:: **依存性の解決について**

    [server projectName]-modelの依存関係の定義は不要である。これは[server projectName]-webserviceから[server projectName]-modelへの依存関係が定義されているため、推移的に依存関係が追加されるためである。
      
|

[server projectName]-webのプロジェクト推奨構成を、以下に示す。

.. code-block:: console

    [server projectName]-web
      └src
          └main
              ├java
              │  └com
              │      └example
              │          ├app...(1)
              │          └ws...(2)
              │            ├exception...(3)
              │            │  └WsExceptionHandler.java
              │            ├abc
              │            │  └AbcWebServiceImpl.java
              │            └def
              │                └DefWebServiceImpl.java
              ├resources
              │  ├META-INF
              │  │  └spring
              │  │      ├applicationContext.xml...(4)
              │  │      ├application.properties...(5)
              │  │      ├spring-mvc.xml ...(6)
              │  │      ├spring-security.xml...(7)
              │  │      └[server projectName]-ws.xml...(8)
              │  └i18n
              │      └application-messages.properties...(9)
              └webapp
                  ├resources...(10)
                  └WEB-INF
                      ├views ...(11)
                      └web.xml...(12)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | アプリケーション層の構成要素を格納するパッケージ。
        | Webサービスのみ作成する場合は削除してもよい。
    * - | (2)
      - | Webサービスの関連クラスを格納するパッケージ。
    * - | (3)
      - | Webサービスの例外ハンドラーなどを格納するパッケージ。
    * - | (4)
      - | アプリケーション全体に関するBean定義を行う。
    * - | (5)
      - | アプリケーションで使用するプロパティを定義する。
    * - | (6)
      - | Spring MVCの設定を行うBean定義を行う。
        | Webサービスのみ作成する場合は削除してもよい。
    * - | (7)
      - | Spring Securityの設定を行うBean定義を行う。
    * - | (8)
      - | Webサービスに関するBean定義を行う。
    * - | (9)
      - | 画面表示用のメッセージ(国際化対応)定義を行う。
    * - | (10)
      - | 静的リソース(css、js、画像など)を格納する。
        | Webサービスのみ作成する場合は削除してもよい。
    * - | (11)
      - | View(jsp)を格納する。
        | Webサービスのみ作成する場合は削除してもよい。
    * - | (12)
      - | Servletのデプロイメント定義を行う。

.. raw:: latex

   \newpage

.. Note:: **SOAPサーバの不要なファイル**

    SOAPサーバで、Webサービスのみを作成する場合、ブランクプロジェクトに存在するSpring MVCの設定ファイルなどは不要となるため、削除したほうが望ましい。


|

[server projectName]-env
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

[server projectName]-envについては、通常のenvプロジェクトと変わらないため、「:doc:`../../Overview/ApplicationLayering` の :ref:`application-layering_project-structure`」を参照されたい。

|

[server projectName]-model
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

[server projectName]-modelのプロジェクト推奨構成を、以下に示す。

.. code-block:: console

    [server projectName]-model
      └src
          └main
              └java
                  └com
                      └example
                          └domain ...(1)
                              └model ...(2)
                                  ├Xxx.java
                                  ├Yyy.java
                                  └Zzz.java


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ドメイン層の構成要素を格納するパッケージ。
    * - | (2)
      - | Domain Objectの中でWebサービス実行時に使用するクラスを格納するパッケージ。

|

[server projectName]-webservice
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

[server projectName]-webserviceのプロジェクト推奨構成を、以下に示す。
  
  
.. code-block:: console

    [server projectName]-webservice
      └src
          └main
              └java
                  └com
                      └example
                          └ws...(1)
                            ├webfault...(2)
                            ├abc
                            │  └AbcWebService.java
                            └def
                                └DefWebService.java

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | Webサービスのインターフェースを格納するパッケージ。
    * - | (2)
      - | Webサービスのwebfaultを格納するパッケージ。

|

.. _SOAPAppendixPackageClient:

クライアントのパッケージ構成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| クライアントを作成するときの推奨する構成について、説明する。
| ガイドラインに従いプロジェクトをSOAPサーバから提供されると以下の構成となる。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - プロジェクト名
      - 説明
    * - | [client projectName]-domain
      - | クライアントのドメイン層に関するクラス・設定ファイルを格納するプロジェクト
    * - | [client projectName]-web
      - | クライアントのアプリケーション層に関するクラス・設定ファイルを格納するプロジェクト
    * - | [client projectName]-env
      - | クライアントの環境に依存するファイル等を格納するプロジェクト

.. note::

    [server projectName]-modelと[server projectName]-webserviceについては、前述の「 :ref:`SOAPAppendixPackageServer`」を参照されたい。

|

[client projectName]-domain
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

SOAPサーバから提供される[server projectName]-webserviceの依存関係を追加するため、\ ``pom.xml``\ に以下を追加する。

.. code-block:: xml
      
    <dependency>
        <groupId>${project.groupId}</groupId>
        <artifactId>artifactId-webservice</artifactId>
    </dependency>

.. note:: **依存性の解決について**

    [server projectName]-webと同様に、この\ ``pom.xml``\ には、[server projectName]-modelの依存関係の定義は不要である。これは[server projectName]-webserviceから[server projectName]-modelへの依存関係が定義されているため、推移的に依存関係が追加されるためである。

その他のパッケージ構成は、通常のdomainプロジェクトと変わらないため、「:doc:`../../Overview/ApplicationLayering` の :ref:`application-layering_project-structure`」を参照されたい。

|

[client projectName]-web
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

[client projectName]-webについては、通常のwebプロジェクトと変わらないため、「:doc:`../../Overview/ApplicationLayering` の :ref:`application-layering_project-structure`」を参照されたい。
  
  

[client projectName]-env
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

[client projectName]-envのプロジェクト推奨構成を、以下に示す。
  
  
.. code-block:: console

    [projectName]-env
      ├configs ...(1)
      │   └[envName] ...(2)
      │       └resources ...(3)
      └src
          └main
              └resources ...(4)
                 ├META-INF
                 │  └spring
                 │      ├[projectName]-env.xml ...(5)
                 │      └[projectName]-infra.properties ...(6)
                 ├dozer.properties
                 ├log4jdbc.properties
                 └logback.xml ...(7)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 全環境の環境依存ファイルを管理するためのディレクトリ。
    * - | (2)
      - | 環境毎の環境依存ファイルを管理するためのディレクトリ。
        | ディレクトリ名は、環境を識別する名前を指定する。
    * - | (3)
      - | 環境毎の設定ファイルを管理するためのディレクトリ。
        | サブディレクトリの構成や管理する設定ファイルは、(4)と同様。
    * - | (4)
      - | ローカル開発環境用の設定ファイルを管理するためのディレクトリ。
    * - | (5)
      - | ローカル開発環境用のBean定義を行う。
    * - | (6)
      - | ローカル開発環境用のプロパティを定義する。
        | WSDLのURLなど環境ごとに変更の可能性がある値を設定する。
    * - | (7)
      - | ローカル開発環境用のログ出力定義を行う。

|

.. _SOAPAppendixWsimport:

wsimportについて
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| wsimportは Java SEに同梱されるコマンドライン・ツールである。
| WSDLファイルを読み取り、Webサービスを呼び出すことが可能なJavaクラス（オプションによってはソースも）を出力するツールである。

wsimportの使い道
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 本ガイドラインでは、wsimportは以下の図のような場合に使用することを推奨している。
| クライアント作成時に、SOAPサーバで使用されるDomain ObjectやWebサービスインターフェースが使用できない場合でも、wsimportを使用することでWebサービスの実行ができるようになる。

.. figure:: images_SOAP/SOAPModelNoProvide.png
    :alt: Server and Client Projects for SOAP
    :width: 80%

|

wsimportの使い方
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| JDKのbinフォルダに格納されており、パスを通すだけで使用可能になる。
| コマンドライン上で以下のようにコマンドを実行すると、ソースファイルがカレントディレクトリに作成される。


.. code-block:: bat

    # (1)
    wsimport -keep -p [出力するソースのパッケージ名] -s [出力するソースを格納する場所] [wsdlのURL]


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | wsimportの引数としてWSDLのURLを指定する。
        | オプションとして以下を使用をする。
        
          * -keep ソースも出力する。
          * -p 出力するソースのパッケージを指定する。
          * -s 出力するソースを格納する場所を指定する。
          
        | その他オプションについては、\ `Java Platform, Standard Edition Tools Reference -Web Services(wsimport)- <http://docs.oracle.com/javase/8/docs/technotes/tools/windows/wsimport.html>`_\ を参照されたい。

.. note::

    wsimportはデフォルトの挙動としてclassファイルのみが出力される。動かすだけなら問題はないが、デバッグなどを実行したい場合に備えkeepオプションを付けてソースも保存することを推奨する。


|

例えば、以下のようなコマンドとなる。

.. code-block:: bat

    wsimport -keep -p com.example.ws.todo -s c:/tmp http://AAA.BBB.CCC.DDD:XXXX/soap-web/ws/TodoWebService?wsdl

作成されるソースは公開されているWebサービスに依存するが、本ガイドラインで使用している以下のJavaクラスが出力される。
    
* Webサービスインターフェース（ソース例では\ ``TodoWebService.java``\ ）
* Domain Object（ソース例では\ ``Todo.java``\ ）

| wsimportで生成したクラスを1つのクライアントプロジェクトのみでしか使用しない場合は、これらをdomainプロジェクトへ配置すればよい。
| 生成したクラスはインフラストラクチャ層(\ :ref:`application-layering_Integration-System-Connector`\ )に所属するが、\ :ref:`application-layering_project-structure`\ のNoteで示したように通常はdomainプロジェクトに含めても問題ない。
| 生成したクラスを複数のクライアントで使用する場合は、\ :ref:`SOAPAppendixAddProject`\ をもとに、modelプロジェクトとwebserviceプロジェクトを作成し、それぞれのクライアントから参照して使用することが望ましい。

.. note::

    出力されるJavaクラスは上記以外にも出力される。出力されたソースのみでクライアントを作成可能なソースである。ただし、本ガイドラインではクライアントは、\ ``org.springframework.remoting.jaxws.JaxWsPortProxyFactoryBean``\ を使用する方針であるため、その他のJavaクラスは使用しないことを推奨する。

|

.. _SOAPAppendixTomcatWebService:

Tomcat上でのWebサービス開発
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 本ガイドラインでは、Java EEサーバ上のJAX-WSを使う前提で記述されているが、Tomcatの場合、JAX-WS実装が存在しない。
| そのため、ここではSOAPサーバがTomcatの場合、JAX-WSの実装プロダクトとして\ `Apache CXF <https://cxf.apache.org/>`_\ を使用する。設定を変更して\ ``CXFServlet``\ を使用する必要がある。
| Apache CXFを使用する場合は、WebServiceクラスの実装方式は以下の2つが存在する。

#. POJOでWebサービス実装クラスを記述する方式
#. \ ``SpringBeanAutowiringSupport``\を継承してWebサービス実装クラスを作成する方式 (これまで説明してきた方法)

| 1の場合、Webサービス実装クラスがPOJOになるため、単体試験などをしやすくなる。ただし、この方式はTomcat以外のAPサーバでは、うまく動作しないことがある。そのため、ガイドライン本体では、この方式ではなく2の方式での実現を記述しているが、Tomcatのみを使用する場合、この1の方式を使用したほうがメリットが多いのでこちらを推奨する。
| 2の場合、他のAPサーバ同様に実装をすることができる。運用はJava EEサーバであるが、開発中はTomcatを使用せざるをえないケースではこちらの方式を利用されたい。

|

CXFServletを使用する場合の設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``CXFServlet``\ を使用するため、\ ``pom.xml``\ にライブラリの設定を記述する。

.. code-block:: xml

    <!-- (1) -->
    <dependency>
        <groupId>org.apache.cxf</groupId>
        <artifactId>cxf-rt-frontend-jaxws</artifactId>
        <version>3.1.4</version>
    </dependency>
    <dependency>
        <groupId>org.apache.cxf</groupId>
        <artifactId>cxf-rt-transports-http</artifactId>
        <version>3.1.4</version>
    </dependency>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``CXFServlet``\ を使用するため、Apache CXFライブラリへの依存関係を追加する。

|

次に\ ``web.xml``\ にSOAP Web Serviceを受け付ける\ ``CXFServlet``\ を定義する。

.. code-block:: xml
      
    <!-- (1) -->
    <servlet>
        <servlet-name>cxfServlet</servlet-name>
        <servlet-class>org.apache.cxf.transport.servlet.CXFServlet</servlet-class>
        <init-param>
            <param-name>config-location</param-name>
            <param-value>classpath:/META-INF/spring/cxf-servlet.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <!-- (2) -->
    <servlet-mapping>
        <servlet-name>cxfServlet</servlet-name>
        <url-pattern>/ws/*</url-pattern>
    </servlet-mapping>
      
      
.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``org.apache.cxf.transport.servlet.CXFServlet``\ のサーブレット定義を行う。
        | \ ``config-location``\ には、後述する\ ``cxf-servlet.xml``\ のパスを指定する。
    * - | (2)
      - | 定義したサーブレットへのマッピングを定義する。この場合、コンテキスト名/ws配下にWebサービスが作成される。

|

POJO方式で必要な設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Webサービス実装クラスをエンドポイントとして設定する。


*[server projectName]-web/src/main/resources/META-INF/spring/cxf-servlet.xml*

.. code-block:: xml

    <beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xmlns:jaxws="http://cxf.apache.org/jaxws" xmlns:soap="http://cxf.apache.org/bindings/soap"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
             http://www.springframework.org/schema/beans/spring-beans.xsd
             http://www.springframework.org/schema/context
             http://www.springframework.org/schema/context/spring-context.xsd
             http://cxf.apache.org/jaxws
             http://cxf.apache.org/schemas/jaxws.xsd
             http://cxf.apache.org/bindings/soap
             http://cxf.apache.org/schemas/configuration/soap.xsd">

        <!-- (1) -->
        <jaxws:endpoint id="todoWebEndpoint" implementor="#todoWebServiceImpl"
            address="/TodoWebService" />

    </beans>
      
.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 公開するエンドポイントを定義する。
        | \ ``implementor``\ 属性に、DIコンテナに登録済みのWebサービスクラスのbean名(「#bean名」形式)を指定する。
        | \ ``address``\ 属性にWebサービスを公開するアドレスを指定する。
        | アドレスは、公開するエンドポイントのパス部分のみ記述する。
        | 属性の詳細については\ `Apache CXF JAX-WS Configuration <https://cwiki.apache.org/confluence/display/CXF20DOC/JAX-WS+Configuration>`_\ を参照されたい。


|

\ ``TodoWebServiceImpl``\ をPOJOとして作成する。

*[server projectName]-web/src/main/java/com/example/ws/todo/TodoWebServiceImpl.java*

.. code-block:: java

    package com.example.ws.todo;

    import java.util.List;

    import javax.inject.Inject;
    import javax.jws.HandlerChain;
    import javax.jws.WebService;
    import javax.xml.ws.BindingType;
    import javax.xml.ws.soap.SOAPBinding;

    import org.springframework.web.context.support.SpringBeanAutowiringSupport;

    import org.springframework.stereotype.Component;

    import com.example.domain.model.Todo;
    import com.example.domain.service.TodoService;
    import com.example.ws.webfault.WebFaultException;
    import com.example.ws.exception.WsExceptionHandler;
    import com.example.ws.todo.TodoWebService;

    // (1)
    @Component
    @WebService(
          portName = "TodoWebPort",
          serviceName = "TodoWebService",
          targetNamespace = "http://example.com/todo",
          endpointInterface = "com.example.ws.todo.TodoWebService")
    @BindingType(SOAPBinding.SOAP12HTTP_BINDING)
    // (2)
    public class TodoWebServiceImpl implements TodoWebService {

        // omitted

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@Component`` \を付けて、DIコンテナへの登録を行う。
        |
    * - | (2)
      - | コンポーネントスキャンにてDIコンテナへの登録が可能であるため、POJOとして作成する。つまり、\ ``org.springframework.web.context.support.SpringBeanAutowiringSupport``\を継承する必要がなくなる。
        |

|

SpringBeanAutowiringSupportを継承する方式で必要な設定
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

CXFServlet用のBean定義ファイルに、SOAPのエンドポイントとなるクラス名およびアドレスを定義する。

*[server projectName]-web/src/main/resources/META-INF/spring/cxf-servlet.xml*

.. code-block:: xml

    <beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xmlns:jaxws="http://cxf.apache.org/jaxws" xmlns:soap="http://cxf.apache.org/bindings/soap"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
             http://www.springframework.org/schema/beans/spring-beans.xsd
             http://www.springframework.org/schema/context
             http://www.springframework.org/schema/context/spring-context.xsd
             http://cxf.apache.org/jaxws
             http://cxf.apache.org/schemas/jaxws.xsd
             http://cxf.apache.org/bindings/soap
             http://cxf.apache.org/schemas/configuration/soap.xsd">
        <!-- (1) -->
        <jaxws:endpoint id="todoWebEndpoint" implementor="com.example.ws.todo.TodoWebServiceImpl"
            address="/TodoWebService" />

    </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 公開するエンドポイントを定義する。
        | \ ``implementor``\ 属性に公開するWebサービスの実装クラスを指定する。
        | \ ``address``\ 属性にWebサービスを公開するアドレスを指定する。
        | アドレスは、公開するエンドポイントのパス部分のみ記述する。
        | 属性の詳細については\ `Apache CXF JAX-WS Configuration`_\ を参照されたい。

.. raw:: latex

   \newpage

