チュートリアル(Todoアプリケーション)
********************************************************************************

.. contents:: 目次
   :depth: 3
   :local:

はじめに
================================================================================

このチュートリアルで学ぶこと
--------------------------------------------------------------------------------

* TERASOLUNA Global Frameworkによる基本的なアプリケーションの開発方法およびEclipseプロジェクトの構築方法
* TERASOLUNA Global Frameworkの :doc:`../Overview/ApplicationLayering` に従った開発方法


対象読者
--------------------------------------------------------------------------------

* SpringのDIやAOPに関する基礎的な知識がある
* Servlet/JSPを使用してWebアプリケーションを開発したことがある
* SQLに関する知識がある


検証環境
--------------------------------------------------------------------------------

このチュートリアルは以下の環境で動作確認している。他の環境で実施する際は本書をベースに適宜読み替えて設定していくこと。

.. list-table::
    :header-rows: 1
    :widths: 15 85

    * - 種別
      - 名前
    * - OS
      - Windows7 64bit
    * - JVM
      - Java 1.6
    * - IDE
      - Spring Tool Suite Version: 3.2.0.RELEASE, Build Id: 201303060821 (以下STS) Build   Maven 3.0.4 (STS付属)
    * - Application Server
      - VMWare vFabric tc Server Developer Edition v2.8 (STS付属)
    * - Web Browser
      - Google Chrome 27.0.1453.94 m

|

作成するアプリケーションの説明
================================================================================

アプリケーションの概要
--------------------------------------------------------------------------------

TODOを管理するアプリケーションを作成する。TODOの一覧表示、TODOの登録、TODOの完了、TODOの削除を行える。


.. figure:: ./images/image001.png
   :width: 60%


.. _app-requirement:

アプリケーションの業務要件
--------------------------------------------------------------------------------

.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - ルールID
      - 説明
    * - B01
      - 未完のTODOは5件までしか登録できない
    * - B02
      - 完了済みのTODOは完了できない

|

    .. note::

        本要件は学習のためのもので、現実的なTODO管理アプリケーションとしては適切ではない。

|

アプリケーションの画面遷移
--------------------------------------------------------------------------------


.. figure:: ./images/image002.png
   :width: 60%



.. list-table::
    :header-rows: 1
    :widths: 10 20 15 15 40

    * - 項番
      - プロセス名
      - HTTPメソッド
      - URL
      - 説明
    * - 1
      - Show all TODO
      - GET
      - /todo/list
      -
    * - 2
      - Create TODO
      - POST
      - /todo/create
      - 作成完了後1へリダイレクト
    * - 3
      - Finish TODO
      - POST
      - /todo/finish
      - 作成完了後1へリダイレクト
    * - 4
      - Delete TODO
      - POST
      - /todo/delete
      - 作成完了後1へリダイレクト

Show all TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* TODOを全件表示する
* 未完了のTODOに対しては”Finish”と”Delete”用のボタンが付く
* 完了のTODOは打ち消し線で装飾する
* TODOの件名のみ


Create TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* フォームから送信されたTODOを保存する
* TODOの件名は1文字以上30文字以下であること
* :ref:`app-requirement` のB01を満たさない場合はエラーコードE001でビジネス例外をスローする

Finish TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* フォームから送信されたtodoIdに対応するTODOを完了済みにする
* :ref:`app-requirement` のB02を満たさない場合はエラーコードE002でビジネス例外をスローする
* 該当するTODOが存在しない場合はエラーコードE404でビジネス例外をスローする

Delete TODO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* フォームから送信されたtodoIdに対応するTODOを削除する
* 該当するTODOが存在しない場合はエラーコードE404でビジネス例外をスローする


エラーメッセージ一覧
--------------------------------------------------------------------------------

.. list-table::
    :header-rows: 1
    :widths: 15 45 40

    * - エラーコード
      - メッセージ
      - 置換パラメータ
    * - E001
      - [E001] The count of un-finished Todo must not be over {0}.
      - {0}… max unfinished count
    * - E002
      - [E002] The requested Todo is already finished. (id={0})
      - {0}… todoId
    * - E404
      - [E404] The requested Todo is not found. (id={0})
      - {0}… todoId

|

環境構築
================================================================================

プロジェクトの作成
--------------------------------------------------------------------------------

「File」->「Other」->「Maven」->「Maven Project」を選択して「Next」。



.. figure:: ./images/image004.jpg
   :width: 60%

「Create a simple project」にチェックを入れて「Next」。

.. figure:: ./images/image006.jpg
   :width: 60%


.. list-table::
    :widths: 25 75
    :stub-columns: 1

    * - Group Id:
      - org.terasoluna.tutorial
    * - Artifact Id:
      - todo
    * - Packaging:
      - war

で「Finish」

.. figure:: ./images/image008.jpg
   :width: 60%

以下のようなプロジェクトが作成される。


.. figure:: ./images/image009.png
   :width: 40%

|

.. note::

  パッケージ構成上、Package PresentaionをHierarchicalにしたほうが見通しがよい。

  .. figure:: ./images/presentation-hierarchical.png
     :width: 80%

Mavenの設定
--------------------------------------------------------------------------------

pom.xmlを以下のように変更する。
Mavenの知識がない場合は、pom.xmlをコピーするだけで、解説は読み飛ばしてよい。

.. code-block:: xml
   :emphasize-lines: 9-83


    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>

        <groupId>org.terasoluna.tutorial</groupId>
        <artifactId>todo</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <packaging>war</packaging>
        <!-- (1) -->
        <parent>
            <groupId>org.terasoluna.gfw</groupId>
            <artifactId>terasoluna-gfw-parent</artifactId>
            <version>1.0.0.RELEASE</version>
        </parent>

        <!-- (2) -->
        <repositories>
            <repository>
                <releases>
                    <enabled>true</enabled>
                </releases>
                <snapshots>
                    <enabled>false</enabled>
                </snapshots>
                <id>terasoluna-gfw-releases</id>
                <url>http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases/</url>
            </repository>
            <repository>
                <releases>
                    <enabled>false</enabled>
                </releases>
                <snapshots>
                    <enabled>true</enabled>
                </snapshots>
                <id>terasoluna-gfw-snapshots</id>
                <url>http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-snapshots/</url>
            </repository>
            <repository>
                <releases>
                    <enabled>true</enabled>
                </releases>
                <snapshots>
                    <enabled>false</enabled>
                </snapshots>
                <id>terasoluna-gfw-3rdparty</id>
                <url>http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-3rdparty/</url>
            </repository>
        </repositories>

        <dependencies>
            <!-- (3) -->
            <!-- TERASOLUNA -->
            <dependency>
                <groupId>org.terasoluna.gfw</groupId>
                <artifactId>terasoluna-gfw-web</artifactId>
            </dependency>
            <!-- (4) -->
            <dependency>
                <groupId>org.terasoluna.gfw</groupId>
                <artifactId>terasoluna-gfw-security-web</artifactId>
            </dependency>
            <!-- (5) -->
            <dependency>
                <groupId>org.terasoluna.gfw</groupId>
                <artifactId>terasoluna-gfw-recommended-dependencies</artifactId>
                <type>pom</type>
            </dependency>

            <!-- (6) -->
            <!-- Servlet API/ JSP API -->
            <dependency>
                <groupId>org.apache.tomcat</groupId>
                <artifactId>tomcat-servlet-api</artifactId>
                <version>7.0.40</version>
                <scope>provided</scope>
            </dependency>
            <dependency>
                <groupId>org.apache.tomcat</groupId>
                <artifactId>tomcat-jsp-api</artifactId>
                <version>7.0.40</version>
                <scope>provided</scope>
            </dependency>
        </dependencies>
    </project>


pom.xmlを編集した後、プロジェクト名を右クリックし、「Maven」->「Update Project」をクリックし、


.. figure:: ./images/update-project.png
   :width: 60%

「OK」ボタンをクリックする。

以下のように"JRE System Library"のバージョンが"[JavaSE-1.6]"になっていることを確認する。

.. figure:: ./images/check-jre.jpg
   :width: 30%

|

    .. note::
        JDKのバージョンを7に変更したい場合は、pom.xmlの ``<properties>`` に ``<java-version>1.7</java-version>`` を設定した後、
        「Update Project」を実施すること。

            .. code-block:: xml
               :emphasize-lines: 4-6

                <project>
                    <!-- omitted -->

                    <properties>
                        <java-version>1.7</java-version>
                    </properties>
                </project>

Mavenの知識がある場合は、以下の解説を確認すること。

.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | TERASOLUNA Global Frameworkの親pomファイルを指定する。
       | これにより、terasoluna-parentで定義されているライブラリは、versionを指定しなくても、dependencyに追加することができる。
   * - | (2)
     - | TERASOLUNA Global Frameworkを使うためのMavenレポジトリのURLを指定する。
   * - | (3)
     - | TERASOLUNA Global Frameworkの共通ライブラリ(Web用)をdependencyに追加する。
   * - | (4)
     - | TERASOLUNA Global Frameworkの共通ライブラリ(セキュリティWeb用)をdependencyに追加する。
   * - | (5)
     - | TERASOLUNA Global Frameworkで推奨されるライブラリ群を追加する。
       | terasoluna-gfw-recommended-dependenciesはただのpomファイルであるため ``<type>pom</type>`` を記述する必要がある。
   * - | (6)
     - | Servlet/JSP APIをdependencyに追加する。Servlet3に対応する必要がある。
       | これらはscope=provided(本来APサーバーから提供される)であり、warには含まれないが、eclipse上でコンパイルするためには明示的にdependencyに追加する必要がある。
       | （尚、dependency名がtomcat-xxxとなっているが、内包するクラスのパッケージはjavax.servletであるためtomcatに依存しているわけではない。）

|

    .. note:: Proxyサーバーを介してインターネットアクセスする必要がある場合は、
        <HOME>/.m2/settings.xmlに以下のような設定を行う。
        (Windows7の場合C:\\Users\\<YourName>\\.m2\settings.xml)

            .. code-block:: xml

                    <settings>
                      <proxies>
                        <proxy>
                          <active>true</active>
                          <protocol>[Proxy Server Protocol (http)]</protocol>
                          <port>[Proxy Server Port]</port>
                          <host>[Proxy Server Host]</host>
                          <username>[Username]</username>
                          <password>[Password]</password>
                        </proxy>
                      </proxies>
                    </settings>

|

プロジェクト構成
--------------------------------------------------------------------------------

今後作成していくプロジェクトの構成について、以下に示す。

.. code-block:: console

    src
      └main
          ├java
          │  └todo
          │    ├ app ... アプリケーション層を格納
          │    │   └todo ... todo管理業務に関わるクラスを格納
          │    └domain ... ドメイン層を格納
          │        ├model ... Domain Objectを格納
          │        ├repository ... Repositoryを格納
          │        │   └todo ... Todo用Repository
          │        └service ... Serviceを格納
          │            └todo ... TODO業務Service
          ├resources
          │  └META-INF
          │      └spring ... spring関連の設定ファイルを格納
          └wepapp
              └WEB-INF
                  └views ... jspを格納


順番に作成していくので、最初に上記構成を用意する必要はない。

|

    .. note::

         :ref:`前節の「プロジェクト構成」 <application-layering_project-structure>` ではマルチプロジェクトにすることを推奨していたが、
         本チュートリアルでは、学習容易性を重視しているためシングルプロジェクト構成にしている。ただし、実プロジェクトで適用する場合は、
         マルチプロジェクト構成を強く推奨する。

|

設定ファイルの作成
--------------------------------------------------------------------------------

web.xmlの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
src/main/webapp/WEB-INF/web.xmlを作成して、サーブレットやフィルタの定義を行う。
WEB-INFフォルダは「New」->「Folder」で新規作成すること。


.. figure:: ./images/image010.jpg
   :width: 40%

「New」->「File」でweb.xmlを作成し、


.. figure:: ./images/image011.jpg
   :width: 40%


内容は以下のように記述する。

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!-- (1) -->
    <web-app xmlns="http://java.sun.com/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
        version="3.0">
        <!-- (2) -->
        <listener>
            <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
        </listener>
        <context-param>
            <param-name>contextConfigLocation</param-name>
            <!-- Root ApplicationContext -->
            <param-value>
                classpath*:META-INF/spring/applicationContext.xml
            </param-value>
        </context-param>

        <!-- (3) -->
        <filter>
            <filter-name>CharacterEncodingFilter</filter-name>
            <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
            <init-param>
                <param-name>encoding</param-name>
                <param-value>UTF-8</param-value>
            </init-param>
            <init-param>
                <param-name>forceEncoding</param-name>
                <param-value>true</param-value>
            </init-param>
        </filter>
        <filter-mapping>
            <filter-name>CharacterEncodingFilter</filter-name>
            <url-pattern>/*</url-pattern>
        </filter-mapping>

        <!-- (4) -->
        <servlet>
            <servlet-name>appServlet</servlet-name>
            <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
            <init-param>
                <param-name>contextConfigLocation</param-name>
                <!-- ApplicationContext for Spring MVC -->
                <param-value>classpath*:META-INF/spring/spring-mvc.xml</param-value>
            </init-param>
            <load-on-startup>1</load-on-startup>
        </servlet>

        <servlet-mapping>
            <servlet-name>appServlet</servlet-name>
            <url-pattern>/</url-pattern>
        </servlet-mapping>

        <!-- (5) -->
        <jsp-config>
            <jsp-property-group>
                <url-pattern>*.jsp</url-pattern>
                <el-ignored>false</el-ignored>
                <page-encoding>UTF-8</page-encoding>
                <scripting-invalid>false</scripting-invalid>
                <include-prelude>/WEB-INF/views/common/include.jsp</include-prelude>
            </jsp-property-group>
        </jsp-config>
    </web-app>


.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | Servlet3.0を使用するための宣言。
   * - | (2)
     - | ``ContextLoaderListener`` の定義。このリスナーが作成する ``ApplicationContext`` がルートコンテキストとなる。
       | Bean定義ファイルのパスをclasspath直下のMETA-INF/spring/applicationContext.xmlとする。
   * - | (3)
     - | ``CharacterEncodingFilter`` の定義。リクエストとレスポンスの文字コードをUTF-8にするための設定。
   * - | (4)
     - | Spring MVCのエントリポイントとなるDispatcherServletの定義。
       | Spring MVCで使用するBean定義ファイルのパスをclasspath直下のMETA-INF/spring/spring-mvc.xmlとする。
       | ここで作成される ``ApplicationContext`` は(2)で作成される ``ApplicatnionContext`` の子となる。
   * - | (5)
     - | JSP共通でincludeするJSPの定義。任意のJSP(\*.jsp)に対して、/WEB-INF/views/common/include.jspをincludeする。


.. figure:: ./images/image013.png
   :width: 40%

|

共通JSPの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

src/main/webapp/WEB-INF/views/common/include.jspに各JSP共通でincludeする内容を記述する。taglibの定義を共通的に行う。
views/commonフォルダ、include.jspファイルを作成し、以下のように記述する。


.. code-block:: jsp

    <%@ page session="false"%>
    <!-- (1) -->
    <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
    <%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt"%>
    <!-- (2)  -->
    <%@ taglib uri="http://www.springframework.org/tags" prefix="spring"%>
    <%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>
    <!-- (3) -->
    <%@ taglib uri="http://www.springframework.org/security/tags" prefix="sec"%>
    <!-- (4) -->
    <%@ taglib uri="http://terasoluna.org/functions" prefix="f"%>
    <%@ taglib uri="http://terasoluna.org/tags" prefix="t"%>


.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 標準タグライブラリを定義する。
   * - | (2)
     - | Spring MVC用タグライブラリを定義する。
   * - | (3)
     - | Spring Security用タグライブラリを定義する。(ただし本チュートリアルでは使用しない。)
   * - | (4)
     - | 共通ライブラリで提供されている、EL関数、タグライブラリを定義する。




.. figure:: ./images/image014.png
   :width: 40%

|

Bean定義ファイルの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Bean定義ファイルは、以下4種類のファイルを作成する。

* applicationContext.xml
* todo-domain.xml
* todo-infra.xml
* spring-mvc.xml

上から順に説明する。


applicationContext.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
src/main/resources/META-INF/spring/applicationContext.xmlに、Todoアプリ全体に関わる設定を行う。


META-INF/springフォルダを作成し、「New」->「Spring Bean Configuration File」でapplicationContext.xmlを作成する。



.. figure:: ./images/image016.jpg
   :width: 40%



.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

        <!-- (1) -->
        <import resource="classpath:/META-INF/spring/todo-domain.xml" />

        <!-- (2) -->
        <context:property-placeholder
            location="classpath*:/META-INF/spring/*.properties" />

        <!-- (3) -->
        <bean class="org.dozer.spring.DozerBeanMapperFactoryBean">
            <property name="mappingFiles"
                value="classpath*:/META-INF/dozer/**/*-mapping.xml" />
        </bean>

    </beans>


.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 次に説明する、ドメイン層に関するBean定義ファイルをimportする。
   * - | (2)
     - | プロパティファイルの読み込み設定を行う。
       | src/main/resources/META-INF/spring直下の任意のプロパティファイルを読み込む。
       | この設定により、プロパティファイルの値をBean定義ファイル内で${propertyName}形式で埋め込んだり、Javaクラスに@Value("${propertyName}")でインジェクションすることができる。
   * - | (3)
     - | Bean変換用ライブラリDozerのMapperを定義する。
       | (今回は使用しないが、マッピング用XMLファイルを定義する場合はsrc/main/resources/META-INF/dozer/xxx-mapping.xmlという形式でマッピングファイルを作成すること。
       | マッピングファイルに関して `Dozerマニュアル <http://dozer.sourceforge.net/documentation/mappings.html>`_ を参照されたい。)

.. figure:: ./images/image018.png
   :width: 40%

|

    .. note::
        上記内容をコピーせず手入力を行う場合は、「namespace」タブを開き、「Configure Namspecse」で「beans」と「context」にチェックを入れること。
        また「Namespace Versions」でバージョンなしのxsdファイルを選択することを推奨する。

        .. figure:: ./images/image021.jpg
           :width: 60%
           :align: center

        これにより、XML編集時にCtrl+Spaceを使用して入力を補完することができる。

        .. figure:: ./images/image023.png
           :width: 60%
           :align: center

        またバージョンを指定しないことにより、常にjarに含まれる最新のxsdが使用される。

|

todo-domain.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
src/main/resources/META-INF/spring/todo-domain.xmlに、ドメイン層に関わる設定を行う。


META-INF/spring直下において、「New」->「Spring Bean Configuration File」でtodo-domain.xmlを作成する。


.. code-block:: xml


    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
        <!-- (1) -->
        <import resource="classpath:META-INF/spring/todo-infra.xml"/>
        <!-- (2) -->
        <context:component-scan base-package="todo.domain" />
    </beans>


.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 次に説明する、インフラストラクチャ層に関するBean定義ファイルをimportする。
   * - | (2)
     - | ドメイン層のクラスを管理するtodo.domainパッケージ配下をcomponent-scan対象とする。
       | これにより、todo.domainパッケージ配下のクラスに ``@Repository`` , ``@Service`` , ``@Component`` などのアノテーションを付けることで、Spring Framerowkが管理するBeanとして登録される。
       | 登録されたクラス(Bean)は、ControllerやServiceクラスにDIする事で、利用する事が出来る。

.. figure:: ./images/image024.png
   :width: 40%

|

todo-infra.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
src/main/resources/META-INF/spring/todo-infra.xmlに、インフラストラクチャ層に関するBean定義を行う。
ここではDBの設定などを行うが、本節ではDBを使用しないため、以下のように空定義で良い。次節でBean定義を行う。


META-INF/spring直下において、「New」->「Spring Bean Configuration File」でtodo-infra.xmlを作成する。


.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    </beans>


.. figure:: ./images/image025.png
   :width: 40%

|

    .. note:: todo-domain.xml, todo-infra.xmlの内容もすべてapplicationContext.xmlに記述すればよいように思えるかもしれないが、
        役割(層)ごとにファイルを分割することを推奨する。どこに何が定義されているか想像しやすく、メンテナンス性が向上するからである。
        今回のチュートリアルのような小さなアプリケーションでは効果がない。しかし、アプリケーションの規模が大きくなるにつれ、効果が大きくなる。

|

spring-mvc.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
src/main/resources/META-INF/spring/spring-mvc.xmlに、Spring MVCに関する定義を行う。


.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
        xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:util="http://www.springframework.org/schema/util"
        xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

        <!-- (1) -->
        <mvc:annotation-driven></mvc:annotation-driven>

        <!-- (2) -->
        <context:component-scan base-package="todo.app" />

        <!-- (3) -->
        <mvc:resources mapping="/resources/**"
            location="/resources/,classpath:META-INF/resources/"
            cache-period="#{60 * 60}" />

        <mvc:interceptors>
            <!-- (4) -->
            <mvc:interceptor>
                <mvc:mapping path="/**" />
                <mvc:exclude-mapping path="/resources/**" />
                <bean
                    class="org.terasoluna.gfw.web.logging.TraceLoggingInterceptor" />
            </mvc:interceptor>
        </mvc:interceptors>

        <!-- (5) -->
        <bean id="viewResolver"
            class="org.springframework.web.servlet.view.InternalResourceViewResolver">
            <property name="prefix" value="/WEB-INF/views/" />
            <property name="suffix" value=".jsp" />
        </bean>
    </beans>

.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | Spring MVCのアノテーションベースのデフォルト設定を行う。
   * - | (2)
     - | アプリケーション層のクラスを管理するtodo.appパッケージ配下をcomponent-scan対象とする。
   * - | (3)
     - | 静的リソース(css, images, jsなど)アクセスのための設定を行う。
       | mapping属性にURLのパスを、location属性に物理的なパスの設定を行う。
       | この設定の場合<contextPath>/rerources/css/styles.cssに対してリクエストが来た場合、WEB-INF/resources/css/styles.cssを探し、見つからなければクラスパス上(src/main/resourcesやjar内)のresources/css/style.cssを探す。
       | WEB-INF/resources/css/styles.cssが見つからなければ、404エラーを返す。
       | ここではcache-period属性で静的リソースのキャッシュ時間(3600秒=60分)も設定している。
       | ``cache-period="3600"`` と設定しても良いが、60分であることを明示するために `SpEL <http://static.springsource.org/spring/docs/3.2.x/spring-framework-reference/html/expressions.html#expressions-beandef-xml-based>`_ を使用して ``cache-period="#{60 * 60}"`` と書く方が分かりやすい。
       | 尚、本チュートリアルでは静的リソースは使用しない。
   * - | (4)
     - | コントローラ処理のTraceログを出力するインターセプタを設定する。/resources以下を除く任意のパスに適用されるように設定する。
   * - | (5)
     - | ViewResolverの設定を行う。この設定により、例えばコントローラからview名”hello”が返却された場合には/WEB-INF/views/hello.jspが実行される。


.. figure:: ./images/image026.png
   :width: 40%

|

    .. note:: 上記内容をコピーせず手入力を行う場合は、todo-domain.xmlで説明した操作に加え、「mvc」と「util」にもチェックを入れること。

        .. figure:: ./images/image028.png
           :width: 60%
           :align: center

|

logback.xmlの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
src/main/resources/logback.xmlに、logbackによるログの出力設定を行う。


src/main/resources/直下において、「New」->「File」でlogback.xmlを作成する。

.. code-block:: xml

    <!DOCTYPE logback>
    <configuration>
        <!-- (1) -->
        <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern><![CDATA[%d{yyyy-MM-dd HH:mm:ss} [%thread] [%-5level] [%-48logger{48}] - %msg%n]]></pattern>
            </encoder>
        </appender>

        <!-- Application Loggers -->
        <!-- (2) -->
        <logger name="todo">
            <level value="debug" />
        </logger>

        <!-- TERASOLUNA -->
        <!-- (3) -->
        <logger name="org.terasoluna.gfw">
            <level value="info" />
        </logger>
        <!-- (4) -->
        <logger name="org.terasoluna.gfw.web.logging.TraceLoggingInterceptor">
            <level value="trace" />
        </logger>

        <!-- 3rdparty Loggers -->
        <!-- (5) -->
        <logger name="org.springframework">
            <level value="warn" />
        </logger>

        <!-- (6) -->
        <logger name="org.springframework.web.servlet">
            <level value="info" />
        </logger>

        <!-- (7) -->
        <root level="WARN">
            <appender-ref ref="STDOUT" />
        </root>
    </configuration>

.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | 標準出力でログを出力するアペンダを設定する。
   * - | (2)
     - | todoパッケージ以下はdebugレベル以上を出力するように設定する。
   * - | (3)
     - | 共通ライブラリのログレベルをinfoにする。
   * - | (4)
     - | spring-mvc.xmlに設定した ``TraceLoggingInterceptor`` に出力されるようにtraceレベルで設定する。
   * - | (5)
     - | Springframeworkのログはwarnレベル以上を出力するように設定する。
   * - | (6)
     - | Springframeworkのログの中でもorg.springframework.web.servlet以下は開発中に有益なログを出力するためinfoレベル以上で設定する。
   * - | (7)
     - | デフォルトはwarnレベル以上を出力するように設定する。


.. figure:: ./images/image029.png
   :width: 40%

|

動作確認
--------------------------------------------------------------------------------
Todoアプリケーションの開発を始める前に、SpringMVCのHelloWorldアプリケーションを作成して、動作確認を行う。「New」->「Class」で


.. list-table::
   :widths: 25 75
   :stub-columns: 1

   * - Package:
     - todo.app.hello
   * - Name:
     - HelloController

でtodo.app.hello.HelloControllerを作成する。


.. figure:: ./images/image030.jpg
   :width: 40%


HelloControllerを以下のように編集する。

.. code-block:: java

    package todo.app.hello;

    import java.util.Date;

    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.web.bind.annotation.RequestMapping;

    // (1)
    @Controller
    public class HelloController {
        // (2)
        private static final Logger logger = LoggerFactory
                .getLogger(HelloController.class);

        // (3)
        @RequestMapping("/")
        public String hello(Model model) {
            Date now = new Date();
            // (4)
            logger.debug("hello {}", now);
            // (5)
            model.addAttribute("now", now);
            // (6)
            return "hello";
        }
    }


.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | Controllerとしてcomponent-scanの対象とするため、クラスレベルに ``@Controller`` アノテーションをつける。
   * - | (2)
     - | ロガーの生成を行う。ロガーの実装はlogbackのものであるが、APIはSLF4Jのものであるため、``org.slf4j.Logger`` を使用すること。
   * - | (3)
     - | ``@RequestMapping`` で”/”(ルート)へのアクセスに対するメソッドのマッピングを設定する。
   * - | (4)
     - | debugログを出力する。”{}”はプレースホルダである。
   * - | (5)
     - | 画面へ日付を渡すためにModelに”now”という名前でDateオブジェクトを追加する。
   * - | (6)
     - | view名としてhelloを返す。ViewResolverの設定により、WEB-INF/views/hello.jspが出力される。


次にview(jsp)を作成する。src/main/webapp/WEB-INF/views/hello.jspを作成して、以下のように記述する。

.. code-block:: jsp

    <!DOCTYPE html>
    <html>
    <head>
    <title>Hello World!</title>
    </head>
    <body>
        <h1>Hello World!</h1>
        <p>
            Today is
            <!-- (1) -->
            <fmt:formatDate value="${now}" pattern="yyyy-MM-dd HH:mm:ss" />
        </p>
    </body>
    </html>


.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | Controllerから渡された”now”を表示する。ここでは ``<fmt:formatDate>`` タグを用いて日付フォーマットを行っている。

パッケージプロジェクト名”todo”を右クリックして「Run As」->「Run on Server」



.. figure:: ./images/image031.jpg
   :width: 40%

実行したいAPサーバー(ここではVMWare vFabric tc Server Developer Edition v2.8)を選び
「Next」をクリック

.. figure:: ./images/image032.jpg
   :width: 40%

todoが「Configured」に含まれていることを確認して「Finish」をクリックしてサーバーを起動する。


.. figure:: ./images/image033.jpg
   :width: 40%


起動すると以下のようなログが出力される。”/”というパスに対して ``todo.app.hello.HelloController`` のhelloメソッドがマッピングされていることが分かる。


.. code-block:: guess
   :emphasize-lines: 3

    2013-06-14 14:26:54 [localhost-startStop-1] [WARN ] [org.dozer.config.GlobalSettings                 ] - Dozer configuration file not found: dozer.properties.  Using defaults for all Dozer global properties.
    2013-06-14 14:26:54 [localhost-startStop-1] [INFO ] [o.springframework.web.servlet.DispatcherServlet ] - FrameworkServlet 'appServlet': initialization started
    2013-06-14 14:26:54 [localhost-startStop-1] [INFO ] [o.s.w.s.m.m.a.RequestMappingHandlerMapping      ] - Mapped "{[/],methods=[],params=[],headers=[],consumes=[],produces=[],custom=[]}" onto public java.lang.String todo.app.hello.HelloController.hello(org.springframework.ui.Model)
    2013-06-14 14:26:55 [localhost-startStop-1] [INFO ] [o.s.web.servlet.handler.SimpleUrlHandlerMapping ] - Mapped URL path [/resources/**] onto handler 'org.springframework.web.servlet.resource.ResourceHttpRequestHandler#0'
    2013-06-14 14:26:55 [localhost-startStop-1] [INFO ] [o.springframework.web.servlet.DispatcherServlet ] - FrameworkServlet 'appServlet': initialization completed in 986 ms

|

    .. note:: 一行目のWARNログは無視しても良い。抑止したい場合はsrc/main/resourcesに空のdozer.propertiesを作成すること。


ブラウザでhttp://localhost:8080/todo
にアクセスすると、以下のように表示される。


.. figure:: ./images/image034.png
   :width: 40%


コンソールを見ると ``TraceLoggingInterceptor`` によるTRACEログとControllerで実装したdebugログが出力されていることがわかる。

.. code-block:: guess

    2013-06-14 15:40:59 [tomcat-http--3] [TRACE] [o.t.gfw.web.logging.TraceLoggingInterceptor     ] - [START CONTROLLER] HelloController.hello(Model)
    2013-06-14 15:40:59 [tomcat-http--3] [DEBUG] [todo.app.hello.HelloController                  ] - hello Fri Jun 14 15:40:59 JST 2013
    2013-06-14 15:40:59 [tomcat-http--3] [TRACE] [o.t.gfw.web.logging.TraceLoggingInterceptor     ] - [END CONTROLLER  ] HelloController.hello(Model)-> view=hello, model={now=Fri Jun 14 15:40:59 JST 2013}
    2013-06-14 15:40:59 [tomcat-http--3] [TRACE] [o.t.gfw.web.logging.TraceLoggingInterceptor     ] - [HANDLING TIME   ] HelloController.hello(Model)-> 15,043,704 ns

|

    .. note:: ``TraceLoggingInterceptor`` はControllerの開始、終了でログを出力する。終了時にはViewとModelの情報および処理時間を出力する。

ログの確認後は、HelloController, hello.jspの2ファイルを削除しても構わない。

|

Todoアプリケーションの作成
================================================================================
| Todoアプリケーションを作成する。作成する順は、以下の通りである。

* ドメイン層(+ インフラストラクチャ層)

 * Domain Object作成
 * Repository作成
 * Service作成

* アプリケーション層

 * Controller作成
 * Form作成
 * View作成

なお、本節では、Todoの保存にDBを使用しない。DBを使用するRepositoryの作成は、\ :ref:`tutorial-todo_infra`\ で行う。

|

ドメイン層の作成
--------------------------------------------------------------------------------

Domain Objectの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ドメインオブジェクトに必要なプロパティは、

#. ID
#. タイトル
#. 完了フラグ
#. 作成日

である。

以下の、Domainオブジェクトを作成する。
FQCNは、todo.domain.model.Todoとする。JavaBeanとして実装すればよい。


.. list-table::
   :widths: 25 75
   :stub-columns: 1

   * - Package:
     - todo.domain.model
   * - Name:
     - Todo
   * - Interfaces:
     - java.io.Serializable


.. figure:: ./images/image057.png
   :width: 40%


.. code-block:: java

    package todo.domain.model;

    import java.io.Serializable;
    import java.util.Date;

    public class Todo implements Serializable {
        private static final long serialVersionUID = 1L;


        private String todoId;

        private String todoTitle;

        private boolean finished;

        private Date createdAt;

        public String getTodoId() {
            return todoId;
        }

        public void setTodoId(String todoId) {
            this.todoId = todoId;
        }

        public String getTodoTitle() {
            return todoTitle;
        }

        public void setTodoTitle(String todoTitle) {
            this.todoTitle = todoTitle;
        }

        public boolean isFinished() {
            return finished;
        }

        public void setFinished(boolean finished) {
            this.finished = finished;
        }

        public Date getCreatedAt() {
            return createdAt;
        }

        public void setCreatedAt(Date createdAt) {
            this.createdAt = createdAt;
        }
    }


.. figure:: ./images/image058.png
   :width: 40%

|

    .. note::
        Getter/Setterは自動生成できる。フィールドを定義した後、右クリックで「Source」->「Generate Getter and Setters…」


            .. figure:: ./images/image059.png
               :width: 40%


        serialVersionUID以外を選択して「OK」


            .. figure:: ./images/image060.png
               :width: 40%

Repositoryの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
今回のアプリケーションで、必要なTODOオブジェクトに対するCRUD系操作は、

* TODOの1件取得
* TODOの全件取得
* TODOの1件削除
* TODOの1件更新
* 完了済みTODO件数の取得

である。これらの操作を定義するインタフェースTodoRepositoryを作成する。
FQCNは、todo.domain.repository.todo.TodoRepositoryとする。

.. code-block:: java

    package todo.domain.repository.todo;

    import java.util.Collection;

    import todo.domain.model.Todo;

    public interface TodoRepository {
        Todo findOne(String todoId);

        Collection<Todo> findAll();

        Todo save(Todo todo);

        void delete(Todo todo);

        long countByFinished(boolean finished);
    }


.. figure:: ./images/image061.png
   :width: 40%

.. note::
   ここで、TodoRepositoryの汎用性を上げるため、「完了済み件数の取得」ではなく、「完了状態がxである件数」を取得するメソッドとして定義した。

RepositoryImplの作成(インフラストラクチャ層)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 説明を単純化するため、Repositotyの実装は、Mapを使ったインメモリ実装とする。
| DBを使用したRepositoryの実装は、\ :ref:`tutorial-todo_infra`\ で説明する。
| FQCNはtodo.domain.repository.todo.TodoRepositoryImplとする。クラスレベルに、\ ``@Repository``\ アノテーションをつけること。

.. code-block:: java

    package todo.domain.repository.todo;

    import java.util.Collection;
    import java.util.Map;
    import java.util.concurrent.ConcurrentHashMap;

    import org.springframework.stereotype.Repository;

    import todo.domain.model.Todo;

    @Repository // (1)
    public class TodoRepositoryImpl implements TodoRepository {
        private static final Map<String, Todo> TODO_MAP = new ConcurrentHashMap<String, Todo>();

        @Override
        public Todo findOne(String todoId) {
            return TODO_MAP.get(todoId);
        }

        @Override
        public Collection<Todo> findAll() {
            return TODO_MAP.values();
        }

        @Override
        public Todo save(Todo todo) {
            return TODO_MAP.put(todo.getTodoId(), todo);
        }

        @Override
        public void delete(Todo todo) {
            TODO_MAP.remove(todo.getTodoId());
        }

        @Override
        public long countByFinished(boolean finished) {
            long count = 0;
            for (Map.Entry<String, Todo> e : TODO_MAP.entrySet()) {
                Todo todo = e.getValue();
                if (finished == todo.isFinished()) {
                    count++;
                }
            }
            return count;
        }
    }


.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | Repositoryとして、component-scan対象とするため、クラスレベルに\ ``@Repository``\ アノテーションをつける。


Repositoryは、業務ルールを含まないので、保存先(この場合は、Map)への出し入れに終始することに注意する。


.. figure:: ./images/image062.png
   :width: 40%

.. note::

  完全に層別パッケージを分けるのであれば、インフラストラクチャ層のクラスは、todo.infrastructure以下に作成した方が良い。

  ただし、通常のプロジェクトでは、インフラストラクチャ層が変更されることを前提としていない(そのような前提で進めるプロジェクトは、少ない)。
  そこで、作業効率向上のために、ドメイン層のrepositotyと同じ階層に、RepositoryImplを作成しても良い。

Serviceの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
業務処理を実装する。必要な処理は、

* Todoの全件取得
* Todoの新規作成
* Todoの完了
* Todoの削除

である。まずは、TodoServiceインタフェースを作成して、これらを定義する。
FQCNは、todo.domain.serivce.todo.TodoServiceとする。

.. code-block:: java

    package todo.domain.service.todo;

    import java.util.Collection;

    import todo.domain.model.Todo;

    public interface TodoService {
        Collection<Todo> findAll();

        Todo create(Todo todo);

        Todo finish(String todoId);

        void delete(String todoId);
    }

必要な処理と、実装するメソッドの対応は、以下の通りである。

* Todoの全件取得→findAllメソッド
* Todoの新規作成→createメソッド
* Todoの完了→finishメソッド
* Todoの削除→deleteメソッド


.. figure:: ./images/image063.png
   :width: 40%


実装クラスのFQCNを、todo.domain.service.TodoServiceImplとする。

.. code-block:: java

    package todo.domain.service.todo;

    import java.util.Collection;
    import java.util.Date;
    import java.util.UUID;

    import javax.inject.Inject;

    import org.springframework.stereotype.Service;
    //import org.springframework.transaction.annotation.Transactional;
    import org.terasoluna.gfw.common.exception.BusinessException;
    import org.terasoluna.gfw.common.exception.ResourceNotFoundException;
    import org.terasoluna.gfw.common.message.ResultMessage;
    import org.terasoluna.gfw.common.message.ResultMessages;

    import todo.domain.model.Todo;
    import todo.domain.repository.todo.TodoRepository;

    @Service// (1)
    // @Transactional // (2)l
    public class TodoServiceImpl implements TodoService {
        @Inject// (3)
        protected TodoRepository todoRepository;

        private static final long MAX_UNFINISHED_COUNT = 5;

        // (4)
        public Todo findOne(String todoId) {
            Todo todo = todoRepository.findOne(todoId);
            if (todo == null) {
                // (5)
                ResultMessages messages = ResultMessages.error();
                messages.add(ResultMessage
                        .fromText("[E404] The requested Todo is not found. (id="
                                + todoId + ")"));
                // (6)
                throw new ResourceNotFoundException(messages);
            }
            return todo;
        }

        @Override
        public Collection<Todo> findAll() {
            return todoRepository.findAll();
        }

        @Override
        public Todo create(Todo todo) {
            long unfinishedCount = todoRepository.countByFinished(false);
            if (unfinishedCount >= MAX_UNFINISHED_COUNT) {
                ResultMessages messages = ResultMessages.error();
                messages.add(ResultMessage
                        .fromText("[E001] The count of un-finished Todo must not be over "
                                + MAX_UNFINISHED_COUNT + "."));
                // (7)
                throw new BusinessException(messages);
            }

            // (8)
            String todoId = UUID.randomUUID().toString();
            Date createdAt = new Date();

            todo.setTodoId(todoId);
            todo.setCreatedAt(createdAt);
            todo.setFinished(false);

            todoRepository.save(todo);

            return todo;
        }

        @Override
        public Todo finish(String todoId) {
            Todo todo = findOne(todoId);
            if (todo.isFinished()) {
                ResultMessages messages = ResultMessages.error();
                messages.add(ResultMessage
                        .fromText("[E002] The requested Todo is already finished. (id="
                                + todoId + ")"));
                throw new BusinessException(messages);
            }
            todo.setFinished(true);
            todoRepository.save(todo);
            return todo;
        }

        @Override
        public void delete(String todoId) {
            Todo todo = findOne(todoId);
            todoRepository.delete(todo);
        }
    }


.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | Serviceとしてcomponent-scanの対象とするため、クラスレベルに\ ``@Service``\ アノテーションをつける。
   * - | (2)
     - | 今回の実装では、DBを使用しないため、トランザクション管理は不要であるが、DBを使用する場合は、クラスレベルに\ ``@Transactional``\ アノテーションをつけること。
       | 詳しくは、\ :ref:`tutorial-todo_infra`\ で説明する。
   * - | (3)
     - | \ ``@Inject``\ アノテーションで、TodoRepositoryの実装をインジェクションする。
   * - | (4)
     - | 1件取得は、finishメソッドでもdeleteメソッドでも使用するため、メソッドとして用意しておく(interfaceに公開しても良い)。
   * - | (5)
     - | 結果メッセージを格納するクラスとして、共通ライブラリで用意されているorg.terasoluna.gfw.common.message.ResultMessageを用いる。
       | 今回は、Errorメッセージをスローするために、ResultMessages.error()でメッセージ種別を指定して、ResultMessageを追加している。
   * - | (6)
     - | 対象のデータが存在しない場合、共通ライブラリで用意されているorg.terasoluna.gfw.common.exception.ResourceNotFoundExceptionをスローする。
   * - | (7)
     - | 業務エラーが発生した場合、共通ライブラリで用意されているorg.terasoluna.gfw.common.exception.BusinessExceptionをスローする。
   * - | (8)
     - | 一意性のある値を生成するために、UUIDを使用している。DBのシーケンスを用いてもよい。

.. note::

  本節では、説明を単純化するため、エラーメッセージをハードコードしているが、メンテナンスの観点で本来は好ましくない。
  通常、メッセージは、プロパティファイルに外部化することが推奨される。
  プロパティファイルに外部化する方法は、\ :doc:`../ArchitectureInDetail/PropertyManagement`\ を参照されたい。

.. figure:: ./images/image064.png
   :width: 40%

ServiceのJUnit作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
TBD

アプリケーション層の作成
--------------------------------------------------------------------------------

ドメイン層の実装が完了したので、次はドメイン層を利用して、アプリケーション層の作成に取り掛かる。

Controllerの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| まずは、todo管理業務にかかわる画面遷移を、制御するTodoControllerを作成する。
| FQCNはtodo.app.todo.TodoControllerとする。上位パッケージがドメイン層とは異なるので注意すること。

.. code-block:: java

    package todo.app.todo;

    import org.springframework.stereotype.Controller;
    import org.springframework.web.bind.annotation.RequestMapping;

    @Controller // (1)
    @RequestMapping("todo") // (2)
    public class TodoController {

    }


.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | Controllerとしてcomponent-scanの対象とするため、クラスレベルに、\ ``@Controller``\ アノテーションをつける。
   * - | (2)
     - | TodoControllerが扱う画面遷移のパスを、すべて<contextPath>/todo配下にするため、クラスレベルに@RequestMapping(“todo”)を設定する。


.. figure:: ./images/image065.png
   :width: 40%


Show all TODO
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
この画面では、

* 新規作成フォームの表示
* TODOの全件表示

を行う。

Formの作成
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
Formには、タイトル情報があればよいので、次のようなJavaBeanになる。FQCNは、todo.app.todo.TodoFormとする。

.. code-block:: java

    package todo.app.todo;

    import java.io.Serializable;

    public class TodoForm implements Serializable {
        private static final long serialVersionUID = 1L;

        private String todoTitle;

        public String getTodoTitle() {
            return todoTitle;
        }

        public void setTodoTitle(String todoTitle) {
            this.todoTitle = todoTitle;
        }

    }


.. figure:: ./images/image066.png
   :width: 40%

Controllerの実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
TodoControllerに、setUpFormメソッドと、listメソッドを実装する。

.. code-block:: java
   :emphasize-lines: 18-32

    package todo.app.todo;

    import java.util.Collection;

    import javax.inject.Inject;

    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.RequestMapping;

    import todo.domain.model.Todo;
    import todo.domain.service.todo.TodoService;

    @Controller
    @RequestMapping("todo")
    public class TodoController {
        @Inject // (3)
        protected TodoService todoService;

        @ModelAttribute // (4)
        public TodoForm setUpForm() {
            TodoForm form = new TodoForm();
            return form;
        }

        @RequestMapping(value = "list") // (5)
        public String list(Model model) {
            Collection<Todo> todos = todoService.findAll();
            model.addAttribute("todos", todos); // (6
            return "todo/list"; // (7)
        }
    }


.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (3)
     - | TodoServiceを、DIコンテナによってインジェクションさせるために、\ ``@Inject``\ アノテーションをつける。
       | DIコンテナの管理するTodoSerivce型インスタンスがインジェクションされるため、結果として、TodoServiceImplインスタンスがインジェクションされる。
   * - | (4)
     - | Formを初期化する。\ ``@ModelAttribute``\ アノテーションをつけることで、このメソッドの返り値のformオブジェクトが、”todoForm”という名前でModelに追加される。
       | TodoControllerの各処理で、model.addAttribute(“todoForm”, form)が実行されるのと同義。
   * - | (5)
     - | listメソッドを”<contextPath>/todo/list”にマッピングされるための設定。クラスレベルで@RequestMapping(“todo”)が設定されているため、ここでは@RequestMapping(value = “list”)だけで良い。
   * - | (6)
     - | ModelにTodoのリストを追加して、Viewに渡す。
   * - | (7)
     - | View名として”todo/list”を返すと、spring-mvc.xmlに定義したInternalResourceViewResolverによって、WEB-INF/views/todo/list.jspがレンダリングされることになる。

JSPの作成
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
WEB-INF/views/todo/list.jspで、Controllerから渡されたModelを表示する。
まずは、”Finish”,”Delete”ボタン以外を作成する。

.. code-block:: jsp

    <!DOCTYPE html>
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Todo List</title>
    <style type="text/css">
    .strike {
        text-decoration: line-through;
    }
    </style>
    </head>
    <body>
        <h1>Todo List</h1>
        <div id="todoForm">
            <!-- (1) -->
            <form:form
               action="${pageContext.request.contextPath}/todo/create"
                method="post" modelAttribute="todoForm">
                <!-- (2) -->
                <form:input path="todoTitle" />
                <input type="submit" value="Create Todo" />
            </form:form>
        </div>
        <hr />
        <div id="todoList">
            <ul>
                <!-- (3) -->
                <c:forEach items="${todos}" var="todo">
                    <li><c:choose>
                            <c:when test="${todo.finished}"><!-- (4) -->
                                <span class="strike">
                                <!-- (5) -->
                                ${f:h(todo.todoTitle)}
                                </span>
                            </c:when>
                            <c:otherwise>
                                ${f:h(todo.todoTitle)}
                             </c:otherwise>
                        </c:choose></li>
                </c:forEach>
            </ul>
        </div>
    </body>
    </html>


.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | <form:form>タグでフォームを表示する。modelAttribute属性に、ControllerでModelに追加したformの名前を指定する。
       | action属性に指定するcontextPathは、${pageContext.request.contextPath}で取得できる。
   * - | (2)
     - | <form:input>タグでフォームのプロパティをバインドする。modelAttribute属性に指定したformのプロパティ名と、path属性の値が一致している必要がある。
   * - | (3)
     - | <c:forEach>タグを用いて、Todoのリストを全て表示する。
   * - | (4)
     - | 完了かどうか(finished)で、打ち消し線(text-decoration: line-through;)を装飾するかどうかを判断する。
   * - | (5)
     - | **文字列値を出力する際は、XSS対策のため、必ずf:h()関数を使用してHTMLエスケープを行うこと。**
       | XSS対策についての詳細は、\ :doc:`../Security/XSS`\ を参照されたい。

| STSで「todo」プロジェクトを右クリックし、「Run As」→「Run on Server」でWebアプリケーションを起動する。
| ブラウザで”http://localhost:8080/todo/todo/list”にアクセスすると、以下のような画面が表示される。


.. figure:: ./images/image067.png
   :width: 40%


Create TODO
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
次に、一覧表示画面から”Create TODO”ボタンを押した後の、新規作成処理を実装する。

Controllerの修正
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
TodoControllerに、createメソッドを追加する。

.. code-block:: java
   :emphasize-lines: 8,29-31,46-70

    package todo.app.todo;

    import java.util.Collection;

    import javax.inject.Inject;
    import javax.validation.Valid;

    import org.dozer.Mapper;
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.validation.BindingResult;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.servlet.mvc.support.RedirectAttributes;
    import org.terasoluna.gfw.common.exception.BusinessException;
    import org.terasoluna.gfw.common.message.ResultMessage;
    import org.terasoluna.gfw.common.message.ResultMessages;

    import todo.domain.model.Todo;
    import todo.domain.service.todo.TodoService;

    @Controller
    @RequestMapping("todo")
    public class TodoController {
        @Inject
        protected TodoService todoService;

        // (8)
        @Inject
        protected Mapper beanMapper;

        @ModelAttribute
        public TodoForm setUpForm() {
            TodoForm form = new TodoForm();
            return form;
        }

        @RequestMapping(value = "list")
        public String list(Model model) {
            Collection<Todo> todos = todoService.findAll();
            model.addAttribute("todos", todos);
            return "todo/list";
        }

        @RequestMapping(value = "create", method = RequestMethod.POST) // (9)
        public String create(@Valid TodoForm todoForm, BindingResult bindingResult, // (10)
                Model model, RedirectAttributes attributes) { // (11)

            // (12)
            if (bindingResult.hasErrors()) {
                return list(model);
            }

            // (13)
            Todo todo = beanMapper.map(todoForm, Todo.class);

            try {
                todoService.create(todo);
            } catch (BusinessException e) {
                // (14)
                model.addAttribute(e.getResultMessages());
                return list(model);
            }

            // (15)
            attributes.addFlashAttribute(ResultMessages.success().add(
                    ResultMessage.fromText("Created successfully!")));
            return "redirect:/todo/list";
        }

    }

.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (8)
     - | Formオブジェクトを、DomainObjectに変換する際に、有用なMapperをインジェクションする。
   * - | (9)
     - | パスが/todo/createで、HTTPメソッドがPOSTに対応するように、\ ``@RequestMapping``\ アノテーションを設定する。
   * - | (10)
     - | フォームの入力チェックを行うため、Formの引数に\ ``@Valid``\ アノテーションをつける。入力チェック結果は、その直後の引数BindingResultに格納される。
   * - | (11)
     - | 正常に作成が完了した後、リダイレクトし、一覧画面を表示する。リダイレクト先への情報を格納するために、引数にRedirectAttributesを加える。
   * - | (12)
     - | 入力エラーがあった場合、一覧画面に戻る。Todo全件取得を再度行う必要があるので、listメソッドを再実行する。
   * - | (13)
     - | Mapperを用いて、TodoFormからTodoオブジェクトを作成する。変換元と変換先のプロパティ名が同じ場合は、設定不要である。
       | 今回は、todoTitleプロパティのみ変換するため、Mapperを使用するメリットはほとんどない。プロパティの数が多い場合には、非常に便利である。
   * - | (14)
     - | 業務処理を実行して、BusinessExceptionが発生した場合、結果メッセージをModelに追加して、一覧画面に戻る。
   * - | (15)
     - | 正常に作成が完了したので、結果メッセージをflashスコープに追加して、一覧画面でリダイレクトする。
       | リダイレクトすることにより、ブラウザを再読み込みして、再び新規登録処理がPOSTされることがなくなる。なお、今回は成功メッセージであるため、ResultMessages.success()を使用している。


Formの修正
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
入力チェックのルールを定義するため、Formオブジェクトにアノテーションを追加する。

.. code-block:: java
   :emphasize-lines: 3-4,8-9

    package todo.app.todo;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;

    public class TodoForm {

        @NotNull // (1)
        @Size(min = 1, max = 30) // (2)
        private String todoTitle;

        public String getTodoTitle() {
            return todoTitle;
        }

        public void setTodoTitle(String todoTitle) {
            this.todoTitle = todoTitle;
        }
    }


.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | 必須項目であるので、\ ``@NotNull``\ アノテーションを付ける。
   * - | (2)
     - | 1文字以上30文字以下であるので、\ ``@Size``\ アノテーションで、範囲を指定する。

JSPの修正
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
結果メッセージ表示用のタグを追加する。

.. code-block:: jsp
   :emphasize-lines: 16,22

    <!DOCTYPE html>
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Todo List</title>
    <style type="text/css">
    .strike {
        text-decoration: line-through;
    }
    </style>
    </head>
    <body>
        <h1>Todo List</h1>
        <div id="todoForm">
            <!-- (6) -->
            <t:messagesPanel />

            <form:form
               action="${pageContext.request.contextPath}/todo/create"
                method="post" modelAttribute="todoForm">
                <form:input path="todoTitle" />
                <form:errors path="todoTitle" /><!-- (7) -->
                <input type="submit" value="Create Todo" />
            </form:form>
        </div>
        <hr />
        <div id="todoList">
            <ul>
                <c:forEach items="${todos}" var="todo">
                    <li><c:choose>
                            <c:when test="${todo.finished}">
                                <span style="text-decoration: line-through;">
                                ${f:h(todo.todoTitle)}
                                </span>
                            </c:when>
                            <c:otherwise>
                                ${f:h(todo.todoTitle)}
                             </c:otherwise>
                        </c:choose></li>
                </c:forEach>
            </ul>
        </div>
    </body>
    </html>


.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (6)
     - | <t:messagesPanel>タグで、結果メッセージを表示する。
   * - | (7)
     - | <form:errors>タグで、入力エラーがあった場合に表示する。path属性の値は、<form:input>タグと合わせる。


フォームに適切な値を入力してsubmitすると、以下のように、成功メッセージが表示される。


.. figure:: ./images/image068.png
   :width: 40%


.. figure:: ./images/image069.png
   :width: 40%



6件以上登録した場合は、業務エラーとなり、エラーメッセージが表示される。

.. figure:: ./images/image070.png
   :width: 40%


入力フォームを、空文字にしてsubmitすると、以下のように、エラーメッセージが表示される。


.. figure:: ./images/image071.png
   :width: 40%

メッセージ表示のカスタマイズ
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
<t:messagesPanel>の結果はデフォルトで、

.. code-block:: html

    <div class="alert alert-success"><ul><li>Created successfully!</li></ul></div>


と出力される。
スタイルシート(list.jspの<style>タグ内)に、以下の修正を加えて、結果メッセージの見た目をカスタマイズする。

.. code-block:: css

    .alert {
        border: 1px solid;
    }

    .alert-error {
        background-color: #c60f13;
        border-color: #970b0e;
        color: white;
    }

    .alert-success {
        background-color: #5da423;
        border-color: #457a1a;
        color: white;
    }


メッセージは、以下のように装飾される。



.. figure:: ./images/image072.png
   :width: 40%



.. figure:: ./images/image073.png
   :width: 40%


また、<form:errors>タグのcssClass属性で、入力エラーメッセージのclassを指定できる。JSPを次のように修正し、

.. code-block:: html

    <form:errors path="todoTitle" cssClass="text-error" />


スタイルシートに、以下を追加する。

.. code-block:: css

    .text-error {
        color: #c60f13;
    }


入力エラーは、以下のように装飾される。


.. figure:: ./images/image074.png
   :width: 40%

Finish TODO
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

一覧表示画面に”Finish”ボタンを追加して、ボタンをsubmitすると、hiddenで対象のtodoIdが送られ、Todoを完了するように実装する。


JSPの修正
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

完了用のformを追加する。


.. code-block:: jsp
   :emphasize-lines: 56-67

    <!DOCTYPE html>
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Todo List</title>
    </head>
    <style type="text/css">
    .strike {
        text-decoration: line-through;
    }

    .alert {
        border: 1px solid;
    }

    .alert-error {
        background-color: #c60f13;
        border-color: #970b0e;
        color: white;
    }

    .alert-success {
        background-color: #5da423;
        border-color: #457a1a;
        color: white;
    }

    .text-error {
        color: #c60f13;
    }
    </style>
    <body>
        <h1>Todo List</h1>

        <div id="todoForm">
            <t:messagesPanel />

            <form:form
                action="${pageContext.request.contextPath}/todo/create"
                method="post" modelAttribute="todoForm">
                <form:input path="todoTitle" />
                <form:errors path="todoTitle" cssClass="text-error" />
                <input type="submit" value="Create Todo" />
            </form:form>
        </div>
        <hr />
        <div id="todoList">
            <ul>
                <c:forEach items="${todos}" var="todo">
                    <li><c:choose>
                            <c:when test="${todo.finished}">
                                <span class="strike">${f:h(todo.todoTitle)}</span>
                            </c:when>
                            <c:otherwise>
                                ${f:h(todo.todoTitle)}
                                <!-- (8) -->
                                <form:form
                                    action="${pageContext.request.contextPath}/todo/finish"
                                    method="post"
                                    modelAttribute="todoForm"
                                    cssStyle="display: inline-block;">
                                    <!-- (9) -->
                                    <form:hidden path="todoId"
                                        value="${f:h(todo.todoId)}" />
                                    <input type="submit" name="finish"
                                        value="Finish" />
                                </form:form>
                            </c:otherwise>
                        </c:choose></li>
                </c:forEach>
            </ul>
        </div>
    </body>
    </html>


.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (8)
     - | 未完了の場合に、完了用のformを表示する。<contextPath>/todo/finishに対して、POSTでtodoIdを送信する。
   * - | (9)
     - | <form:hidden>タグでtodoIdを渡す。value属性に値を設定する場合も、 **必ずf:h()関数でHTMLエスケープすること。**

Formの修正
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
完了用のフォームも、TodoFormを用いる。
TodoFormに、todoIdプロパティを追加する必要があるが、そのままだと、新規作成用の入力チェックルールが適用されてしまう。
一つのFormに、新規作成用と完了用で、別々のルールを指定するために、group属性を設定する。

.. code-block:: java
   :emphasize-lines: 8-9,11-12,15-16,19,23-29

    package todo.app.todo;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;

    public class TodoForm {
        // (3)
        public static interface TodoCreate {
        };

        public static interface TodoFinish {
        };

        // (4)
        @NotNull(groups = { TodoFinish.class })
        private String todoId;

        // (5)
        @NotNull(groups = { TodoCreate.class })
        @Size(min = 1, max = 30, groups = { TodoCreate.class })
        private String todoTitle;

        public String getTodoId() {
            return todoId;
        }

        public void setTodoId(String todoId) {
            this.todoId = todoId;
        }

        public String getTodoTitle() {
            return todoTitle;
        }

        public void setTodoTitle(String todoTitle) {
            this.todoTitle = todoTitle;
        }

    }


.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (3)
     - | グループ化したバリデーションを行うためのグループ名となるクラスを作成する。クラスは空でよいため、ここでは、インタフェースを定義する。
       | グループ化バリデーションについては、\ :doc:`../ArchitectureInDetail/Validation`\ を参照されたい。
   * - | (4)
     - | todoIdは、完了処理には必須であるため、\ ``@NotNull``\ アノテーションをつける。完了時にのみ必要なルールであるので、group属性にTodoFinish.classを設定する。
   * - | (5)
     - | 新規作成用のルールは、完了処理には不要であるので、\ ``@NotNull``\ アノテーション、\ ``@Size``\ アノテーション、それぞれのgroup属性にTodoCreate.classを設定する。

Controllerの修正
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

完了処理をTodoControllerに追加する。
グループ化したバリデーションを実行するために、\ **@Valid アノテーションの代わりに、@Validated アノテーションを使用すること**\ に注意する。

.. code-block:: java
   :emphasize-lines: 6,12,50,72-94

    package todo.app.todo;

    import java.util.Collection;

    import javax.inject.Inject;
    import javax.validation.groups.Default;

    import org.dozer.Mapper;
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.validation.BindingResult;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.servlet.mvc.support.RedirectAttributes;
    import org.terasoluna.gfw.common.exception.BusinessException;
    import org.terasoluna.gfw.common.message.ResultMessage;
    import org.terasoluna.gfw.common.message.ResultMessages;

    import todo.app.todo.TodoForm.TodoCreate;
    import todo.app.todo.TodoForm.TodoFinish;
    import todo.domain.model.Todo;
    import todo.domain.service.todo.TodoService;

    @Controller
    @RequestMapping("todo")
    public class TodoController {
        @Inject
        protected TodoService todoService;

        @Inject
        protected Mapper beanMapper;

        @ModelAttribute
        public TodoForm setUpForm() {
            TodoForm form = new TodoForm();
            return form;
        }

        @RequestMapping(value = "list")
        public String list(Model model) {
            Collection<Todo> todos = todoService.findAll();
            model.addAttribute("todos", todos);
            return "todo/list";
        }

        @RequestMapping(value = "create", method = RequestMethod.POST)
        public String create(
                @Validated({ Default.class, TodoCreate.class }) TodoForm todoForm, // (16)
                BindingResult bindingResult, Model model,
                RedirectAttributes attributes) {

            if (bindingResult.hasErrors()) {
                return list(model);
            }

            Todo todo = beanMapper.map(todoForm, Todo.class);

            try {
                todoService.create(todo);
            } catch (BusinessException e) {
                model.addAttribute(e.getResultMessages());
                return list(model);
            }

            attributes.addFlashAttribute(ResultMessages.success().add(
                    ResultMessage.fromText("Created successfully!")));
            return "redirect:/todo/list";
        }

        @RequestMapping(value = "finish", method = RequestMethod.POST) // (17)
        public String finish(
                @Validated({ Default.class, TodoFinish.class }) TodoForm form, // (18)
                BindingResult bindingResult, Model model,
                RedirectAttributes attributes) {
            // (19)
            if (bindingResult.hasErrors()) {
                return list(model);
            }

            try {
                todoService.finish(form.getTodoId());
            } catch (BusinessException e) {
                // (20)
                model.addAttribute(e.getResultMessages());
                return list(model);
            }

            // (21)
            attributes.addFlashAttribute(ResultMessages.success().add(
                    ResultMessage.fromText("Finished successfully!")));
            return "redirect:/todo/list";
        }
    }


.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (16)
     - | グループ化したバリデーションを実施するために、\ ``@Valid``\ アノテーションから\ ``@Validated``\ アノテーションに変更する。
       | valueには、対象のグループクラスを複数指定できる。Default.classはバリデーションルールにgroupが指定されていない場合のグループである。
       | \ ``@Validated``\ アノテーションを使用する際は、Default.classも指定しておくのがよい。
   * - | (17)
     - | パスが、/todo/finishで、HTTPメソッドがPOSTに対応するように、\ ``@RequestMapping``\ アノテーションを設定する。
   * - | (18)
     - | Finish用のグループとして、TodoFinish.classを指定する。
   * - | (19)
     - | 入力エラーがあった場合、一覧画面に戻る。
   * - | (20)
     - | 業務処理を実行して、BusinessExceptionが発生した場合は、結果メッセージをModelに追加して、一覧画面に戻る。
   * - | (21)
     - | 正常に作成が完了したので、結果メッセージをflashスコープに追加して、一覧画面でリダイレクトする。

.. note::

  Create用、Finish用に、別々のFormを作成しても良い。その場合は、必要なパラメータだけが、Formのプロパティになる。
  ただし、クラス数が増え、プロパティも重複することが多いので、仕様変更が発生した場合に、修正コストが高くなる。
  また、同一のController内で、複数のFormオブジェクトを、 ``@ModelAttribute`` メソッドによって初期化すると、
  毎回すべてのFormが初期化されてしまうので、不要なインスタンスが生成されてしまう。そのため、
  基本的に、一つのControllerで利用するFormは、できるだけ集約し、グループ化したバリデーションの設定を行うことを推奨する。


Todoを新規作成した後に、FinishボタンをSubmitすると、以下のように打ち消し線が入り、完了したことがわかる。


.. figure:: ./images/image075.png
   :width: 40%


.. figure:: ./images/image076.png
   :width: 40%

Delete TODO
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
一覧表示画面に"Delete"ボタンを追加して、ボタンをsubmitすると、hiddenで対象のtodoIdが送られ、Todoを完了するように実装する。

JSPの修正
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
削除用のformを追加する。


.. code-block:: jsp
   :emphasize-lines: 68-77

    <!DOCTYPE html>
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Todo List</title>
    </head>
    <style type="text/css">
    .strike {
        text-decoration: line-through;
    }

    .alert {
        border: 1px solid;
    }

    .alert-error {
        background-color: #c60f13;
        border-color: #970b0e;
        color: white;
    }

    .alert-success {
        background-color: #5da423;
        border-color: #457a1a;
        color: white;
    }

    .text-error {
        color: #c60f13;
    }
    </style>
    <body>
        <h1>Todo List</h1>

        <div id="todoForm">
            <t:messagesPanel />

            <form:form
                action="${pageContext.request.contextPath}/todo/create"
                method="post" modelAttribute="todoForm">
                <form:input path="todoTitle" />
                <form:errors path="todoTitle" cssClass="text-error" />
                <input type="submit" value="Create Todo" />
            </form:form>
        </div>
        <hr />
        <div id="todoList">
            <ul>
                <c:forEach items="${todos}" var="todo">
                    <li><c:choose>
                            <c:when test="${todo.finished}">
                                <span class="strike">${f:h(todo.todoTitle)}</span>
                            </c:when>
                            <c:otherwise>
                                ${f:h(todo.todoTitle)}
                                <form:form
                                    action="${pageContext.request.contextPath}/todo/finish"
                                    method="post"
                                    modelAttribute="todoForm"
                                    cssStyle="display: inline-block;">
                                    <form:hidden path="todoId"
                                        value="${f:h(todo.todoId)}" />
                                    <input type="submit" name="finish"
                                        value="Finish" />
                                </form:form>
                            </c:otherwise>
                        </c:choose>
                        <!-- (10) -->
                        <form:form
                            action="${pageContext.request.contextPath}/todo/delete"
                            method="post" modelAttribute="todoForm"
                            cssStyle="display: inline-block;">
                            <!-- (11) -->
                            <form:hidden path="todoId"
                                value="${f:h(todo.todoId)}" />
                            <input type="submit" value="Delete" />
                        </form:form>
                    </li>
                </c:forEach>
            </ul>
        </div>
    </body>
    </html>

.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (10)
     - | 削除用のformを表示する。<contextPath>/todo/deleteに対して、POSTでtodoIdを送信する。
   * - | (11)
     - | <form:hidden>タグで、todoIdを渡す。value属性に値を設定する場合も、\ **必ずf:h()関数でHTMLエスケープすること。**\

Formの修正
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Delete用のグループを、TodoFormに追加する。ルールは、Finish用と同じである。


.. code-block:: java
   :emphasize-lines: 14-15,18

    package todo.app.todo;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;

    public class TodoForm {
        public static interface TodoCreate {
        };

        public static interface TodoFinish {
        };

        // (6)
        public static interface TodoDelete {
        }

        // (7)
        @NotNull(groups = { TodoFinish.class, TodoDelete.class })
        private String todoId;

        @NotNull(groups = { TodoCreate.class })
        @Size(min = 1, max = 30, groups = { TodoCreate.class })
        private String todoTitle;

        public String getTodoId() {
            return todoId;
        }

        public void setTodoId(String todoId) {
            this.todoId = todoId;
        }

        public String getTodoTitle() {
            return todoTitle;
        }

        public void setTodoTitle(String todoTitle) {
            this.todoTitle = todoTitle;
        }

    }

.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (6)
     - | Delete用のグループTodoDeleteを定義する。
   * - | (7)
     - | todoIdプロパティに対して、TodoDeleteグループのバリデーションを行うように設定する。

Controllerの修正
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

削除処理を、TodoControllerに追加する。完了処理とほぼ同じである。

.. code-block:: java
   :emphasize-lines: 94-114

    package todo.app.todo;

    import java.util.Collection;

    import javax.inject.Inject;
    import javax.validation.groups.Default;

    import org.dozer.Mapper;
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.validation.BindingResult;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.servlet.mvc.support.RedirectAttributes;
    import org.terasoluna.gfw.common.exception.BusinessException;
    import org.terasoluna.gfw.common.message.ResultMessage;
    import org.terasoluna.gfw.common.message.ResultMessages;

    import todo.app.todo.TodoForm.TodoDelete;
    import todo.app.todo.TodoForm.TodoCreate;
    import todo.app.todo.TodoForm.TodoFinish;
    import todo.domain.model.Todo;
    import todo.domain.service.todo.TodoService;

    @Controller
    @RequestMapping("todo")
    public class TodoController {
        @Inject
        protected TodoService todoService;

        @Inject
        protected Mapper beanMapper;

        @ModelAttribute
        public TodoForm setUpForm() {
            TodoForm form = new TodoForm();
            return form;
        }

        @RequestMapping(value = "list")
        public String list(Model model) {
            Collection<Todo> todos = todoService.findAll();
            model.addAttribute("todos", todos);
            return "todo/list";
        }

        @RequestMapping(value = "create", method = RequestMethod.POST)
        public String create(
                @Validated({ Default.class, TodoCreate.class }) TodoForm todoForm,
                BindingResult bindingResult, Model model,
                RedirectAttributes attributes) {

            if (bindingResult.hasErrors()) {
                return list(model);
            }

            Todo todo = beanMapper.map(todoForm, Todo.class);

            try {
                todoService.create(todo);
            } catch (BusinessException e) {
                model.addAttribute(e.getResultMessages());
                return list(model);
            }

            attributes.addFlashAttribute(ResultMessages.success().add(
                    ResultMessage.fromText("Created successfully!")));
            return "redirect:/todo/list";
        }

        @RequestMapping(value = "finish", method = RequestMethod.POST)
        public String finish(
                @Validated({ Default.class, TodoFinish.class }) TodoForm form,
                BindingResult bindingResult, Model model,
                RedirectAttributes attributes) {
            if (bindingResult.hasErrors()) {
                return list(model);
            }

            try {
                todoService.finish(form.getTodoId());
            } catch (BusinessException e) {
                model.addAttribute(e.getResultMessages());
                return list(model);
            }

            attributes.addFlashAttribute(ResultMessages.success().add(
                    ResultMessage.fromText("Finished successfully!")));
            return "redirect:/todo/list";
        }

        @RequestMapping(value = "delete", method = RequestMethod.POST)
        public String delete(
                @Validated({ Default.class, TodoDelete.class }) TodoForm form,
                BindingResult bindingResult, Model model,
                RedirectAttributes attributes) {

            if (bindingResult.hasErrors()) {
                return list(model);
            }

            try {
                todoService.delete(form.getTodoId());
            } catch (BusinessException e) {
                model.addAttribute(e.getResultMessages());
                return list(model);
            }

            attributes.addFlashAttribute(ResultMessages.success().add(
                    ResultMessage.fromText("Deleted successfully!")));
            return "redirect:/todo/list";
        }

    }


Todoに対して、”Delete”ボタンをsubmitすると、以下のように、対象のTODOが削除される。


.. figure:: ./images/image077.png
   :width: 40%


.. figure:: ./images/image078.png
   :width: 40%

|

.. _tutorial-todo_infra:

インフラストラクチャ層の変更
================================================================================

| 前節までは、インフラストラクチャ層はメモリによる実装であった。
| 本節では、DBに永続化する実装を行う。DBアクセスするためにO/R Mapperを使用するが、
| ここで、Spring Data JPAによる方法と、TERASOLUNA DAOによる方法の2通りについて、説明する。


共通設定
--------------------------------------------------------------------------------

| まずは、Spring Data JPA版、TERASOLUNA Dao版の両方に共通して適用する設定を行う。
| 今回は、DBセットアップの手間を省くため、H2Databaseを使用する。


pom.xmlの修正
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
pom.xmlに、H2Databaseを使用するためのdependencyを定義する。


.. code-block:: xml

    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <version>1.3.172</version>
        <scope>compile</scope>
    </dependency>

.. warning::

  この設定用は\ **サンプルアプリケーションを簡単試すためのもの**\ であり、実際の開発で使用されることを想定していない。実際のプロジェクトでは削除すること。
  
  また、JDBCドライバの\ ``<scope>``\ は\ ``provided``\ にすべきである。


データソースの定義
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

todo-infra.xmlの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

データソースの定義は、インフラストラクチャ層に関わるので、todo-infra.xmlに定義すべきであるが、
データベースのユーザー名や、パスワードなど、環境に依存する情報を含む定義は、別のBean定義ファイル(todo-env.xml)に定義することを推奨する。

ここでは、todo-env.xmlのインポートのみ行う。

 .. code-block:: xml

     <?xml version="1.0" encoding="UTF-8"?>
     <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

         <import resource="classpath:/META-INF/spring/todo-env.xml" />
     </beans>


 .. note::

    xxx-env.xmlを別ファイルにし、Mavenなどのビルドツールでこのファイルだけ差し替えることにより、環境ごと(開発環境、テスト環境など)で異なる設定値を管理できる。
    また、特定の環境だけに対して、データソースをJNDIから取得するような設定ファイルの管理もできる。


todo-env.xmlの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| src/main/resources/META-INF/spring/todo-env.xmlを作成し、以下のように設定する。
| このファイルに対して、環境に依存する設定(ここでは、DataSource)を含む定義をする。

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

        <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
            destroy-method="close">
            <property name="driverClassName" value="${database.driverClassName}" />
            <property name="url" value="${database.url}" />
            <property name="username" value="${database.username}" />
            <property name="password" value="${database.password}" />
            <property name="defaultAutoCommit" value="false" />
            <property name="maxActive" value="${cp.maxActive}" />
            <property name="maxIdle" value="${cp.maxIdle}" />
            <property name="minIdle" value="${cp.minIdle}" />
            <property name="maxWait" value="${cp.maxWait}" />
        </bean>
    </beans>


メンテナンス性向上のため、プロパティ値は外部化し、プロパティファイルに定義する。
\

 .. note::

    環境(Application Server)によっては、DataSourceをJNDIで取得したほうがよい。
    その場合は<jee:jndi-lookup id="dataSource" jndi-name="JNDI名" />という定義を行う。
    ビルド時に開発環境ではcommons-dbcpを使用し、テスト環境ではJNDIを使用する、というような切り替えができるように、envファイルを作成している。


todo-infra.properties
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
src/main/resources/META-INF/spring/todo-infra.propertiesに、インフラストラクチャ層に関するプロパティ値を定義する。

.. code-block:: properties

    database=H2
    ## (1)
    database.url=jdbc:h2:mem:todo;DB_CLOSE_DELAY=-1;INIT=create table if not exists todo(todo_id varchar(36) primary key, todo_title varchar(30), finished boolean, created_at timestamp)
    database.username=sa
    database.password=
    database.driverClassName=org.h2.Driver
    # connection pool
    ## (2)
    cp.maxActive=96
    cp.maxIdle=16
    cp.minIdle=0
    cp.maxWait=60000


.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | データベースに関する設定を行う。H2のURL,ドライバを設定する。
       | ここでは、説明を単純化するため、インメモリDBを使用して、APサーバーが起動するたびに初期化DDLが実行されるように設定している。
   * - | (2)
     - | コネクションプールに関する設定。ここでは、サンプルの値を設定している。実際の値は、サーバーの性能によって異なることに注意する。


todo-domain.xmlの修正
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
\ ``@Transactional``\ アノテーションによるトランザクション管理を有効にするために、<tx:annotation-driven>タグを設定する。

.. code-block:: xml
   :emphasize-lines: 5,7,11

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xmlns:tx="http://www.springframework.org/schema/tx"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
        <context:component-scan base-package="todo.domain" />
        <import resource="classpath:META-INF/spring/todo-infra.xml"/>
        <tx:annotation-driven/>
    </beans>


TodoServiceImplの修正
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: java
   :emphasize-lines: 10,20,40

    package todo.domain.service.todo;

    import java.util.Collection;
    import java.util.Date;
    import java.util.UUID;

    import javax.inject.Inject;

    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;
    import org.terasoluna.gfw.common.exception.BusinessException;
    import org.terasoluna.gfw.common.exception.ResourceNotFoundException;
    import org.terasoluna.gfw.common.message.ResultMessage;
    import org.terasoluna.gfw.common.message.ResultMessages;

    import todo.domain.model.Todo;
    import todo.domain.repository.todo.TodoRepository;

    @Service
    @Transactional // (9)
    public class TodoServiceImpl implements TodoService {
        @Inject
        protected TodoRepository todoRepository;

        private static final long MAX_UNFINISHED_COUNT = 5;

        public Todo findOne(String todoId) {
            Todo todo = todoRepository.findOne(todoId);
            if (todo == null) {
                ResultMessages messages = ResultMessages.error();
                messages.add(ResultMessage
                        .fromText("[E404] The requested Todo is not found. (id="
                                + todoId + ")"));
                throw new ResourceNotFoundException(messages);
            }
            return todo;
        }

        @Override
        @Transactional(readOnly = true) // (10)
        public Collection<Todo> findAll() {
            return todoRepository.findAll();
        }

        @Override
        public Todo create(Todo todo) {
            long unfinishedCount = todoRepository.countByFinished(false);
            if (unfinishedCount >= MAX_UNFINISHED_COUNT) {
                ResultMessages messages = ResultMessages.error();
                messages.add(ResultMessage
                        .fromText("[E001] The count of un-finished Todo must not be over "
                                + MAX_UNFINISHED_COUNT + "."));

                throw new BusinessException(messages);
            }

            String todoId = UUID.randomUUID().toString();
            Date createdAt = new Date();

            todo.setTodoId(todoId);
            todo.setCreatedAt(createdAt);
            todo.setFinished(false);

            todoRepository.save(todo);

            return todo;
        }

        @Override
        public Todo finish(String todoId) {
            Todo todo = findOne(todoId);
            if (todo.isFinished()) {
                ResultMessages messages = ResultMessages.error();
                messages.add(ResultMessage
                        .fromText("[E002] The requested Todo is already finished. (id="
                                + todoId + ")"));
                throw new BusinessException(messages);
            }
            todo.setFinished(true);
            todoRepository.save(todo);
            return todo;
        }

        @Override
        public void delete(String todoId) {
            Todo todo = findOne(todoId);
            todoRepository.delete(todo);
        }
    }

.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (9)
     - | クラスレベルに、\ ``@Transactional``\ アノテーションをつけることで、公開メソッドをすべてトランザクション管理する。
       | これにより、メソッド開始時にトランザクションを開始し、メソッド正常終了時に、トランザクションをコミットする。
       | 途中で非検査例外が発生した場合は、トランザクションをロールバックする。
   * - | (10)
     - | 参照のみ行う処理に関しては、readOnly=trueをつける。
       | O/R Mapperによっては、この設定により、参照時に最適化が行われる(JPAを使用する場合、効果はない)。


Spring Data JPAを使用する
--------------------------------------------------------------------------------

本節では、インフラストラクチャ層において、 `Spring Data JPA <http://www.springsource.org/spring-data/jpa>`_ を使用する場合の設定方法について、説明する。
TERASOLUNA DAOを使用する場合は、本節を読み飛ばして、\ :ref:`using_terasolunaDao`\ に進んでよい。

Spring Data JPAを使用するための設定ファイルの修正
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

pom.xmlの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Spring Data JPAに関する依存ライブラリを追加するために、pom.xmlに、以下を追加する。

.. code-block:: xml

    <dependency>
        <groupId>org.terasoluna.gfw</groupId>
        <artifactId>terasoluna-gfw-jpa</artifactId>
    </dependency>


todo-infra.xmlの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
todo-infra.xmlに、JPA、およびSpring Data JPAを使用するための設定を行う。JPAのEntityManagerFactoryは、ここで定義する。

.. code-block:: xml
   :emphasize-lines: 4-5,7-8,12-44

    <?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xmlns:jpa="http://www.springframework.org/schema/data/jpa"
    	xmlns:util="http://www.springframework.org/schema/util"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        	http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd
        	http://www.springframework.org/schema/data/jpa http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">

        <import resource="classpath:/META-INF/spring/todo-env.xml" />

        <!-- (1) -->
        <jpa:repositories base-package="todo.domain.repository"></jpa:repositories>

        <!-- (2) -->
        <bean id="jpaVendorAdapter"
            class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
            <property name="showSql" value="false" />
            <property name="database" value="${database}" />
        </bean>

        <!-- (3) -->
        <bean
            class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean"
            id="entityManagerFactory">
            <!-- (4) -->
            <property name="packagesToScan" value="todo.domain.model" />
            <property name="dataSource" ref="dataSource" />
            <property name="jpaVendorAdapter" ref="jpaVendorAdapter" />
            <!-- (5) -->
            <property name="jpaPropertyMap">
                <util:map>
                    <entry key="hibernate.hbm2ddl.auto" value="none" />
                    <entry key="hibernate.ejb.naming_strategy"
                        value="org.hibernate.cfg.ImprovedNamingStrategy" />
                    <entry key="hibernate.connection.charSet" value="UTF-8" />
                    <entry key="hibernate.show_sql" value="false" />
                    <entry key="hibernate.format_sql" value="false" />
                    <entry key="hibernate.use_sql_comments" value="true" />
                    <entry key="hibernate.jdbc.batch_size" value="30" />
                    <entry key="hibernate.jdbc.fetch_size" value="100" />
                </util:map>
            </property>
        </bean>

    </beans>

.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | Spring Data JPAを使用すると、Repositoryインタフェースから実装クラスを自動生成する。
       | <jpa:repository>タグのbase-package属性で、対象のRepositoryを含むパッケージを指定する。
   * - | (2)
     - | JPAの実装ベンダの設定を行う。JPA実装として、Hibernateを使うため、HibernateJpaVendorAdapterを定義する。
   * - | (3)
     - | EntityManagerの定義を行う。
   * - | (4)
     - | エンティティのパッケージ名を指定する。
   * - | (5)
     - | Hibernateに関する詳細な設定を行う。


todo-env.xmlの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
トランザクションマネージャに関連するBean定義を追加する。

.. code-block:: xml
   :emphasize-lines: 20-24

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

        <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
            destroy-method="close">
            <property name="driverClassName" value="${database.driverClassName}" />
            <property name="url" value="${database.url}" />
            <property name="username" value="${database.username}" />
            <property name="password" value="${database.password}" />
            <property name="defaultAutoCommit" value="false" />
            <property name="maxActive" value="${cp.maxActive}" />
            <property name="maxIdle" value="${cp.maxIdle}" />
            <property name="minIdle" value="${cp.minIdle}" />
            <property name="maxWait" value="${cp.maxWait}" />
        </bean>

        <!-- (6) -->
        <bean class="org.springframework.orm.jpa.JpaTransactionManager"
            id="transactionManager">
            <property name="entityManagerFactory" ref="entityManagerFactory" />
        </bean>


    </beans>

.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (6)
     - | トランザクションマネージャの設定。idは、transactionManagerにすること。
       | 別の名前を指定する場合は、todo-domain.xmlの<tx:annotation-driven>タグと、todo-infra.xmlの<jpa:repository>タグにもトランザクションマネージャ名を指定する必要がある。

.. note::
    JavaEEコンテナ上で、トランザクションマネージャは、JtaTransactionManagerを使用したほうがよい。この場合、<tx:jta-transaction-manager />でトランザクションマネージャの定義を行う。

    これらの設定が、環境によって変わらないプロジェクト(例えば、Tomcatを使用する場合など)は、todo-infra.xmlに定義してもよい。


spring-mvc.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
spring-mvc.xmlにOpenEntityManagerInViewInterceptorを追加し、Interceptorで、EntityManagerのライフサイクルの開始と終了を行う。
この設定を追加することで、アプリケーション層(Contollerや、Viewクラス)でのLazy Loadが、サポートされる。

.. code-block:: xml

    <mvc:interceptors>

        <!-- ... -->

        <!-- (6) -->
        <mvc:interceptor>
            <mvc:mapping path="/**" />
            <mvc:exclude-mapping path="/resources/**" />
            <bean
                class="org.springframework.orm.jpa.support.OpenEntityManagerInViewInterceptor" />
        </mvc:interceptor>

    </mvc:interceptors>

.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (6)
     - | 静的リソース(css, js, imageなど)へのアクセス(\ ``/resources/**``\ )の場合は、データアクセスが確実に発生しないため、Interceptorの適用対象外としている。


logback.xmlの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
.. code-block:: xml
   :emphasize-lines: 32-45

    <!DOCTYPE logback>
    <configuration>
        <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern><![CDATA[%d{yyyy-MM-dd HH:mm:ss} [%thread] [%-5level] [%-48logger{48}] - %msg%n]]></pattern>
            </encoder>
        </appender>

        <!-- Application Loggers -->
        <logger name="todo">
            <level value="debug" />
        </logger>

        <!-- TERASOLUNA -->
        <logger name="org.terasoluna.gfw">
            <level value="info" />
        </logger>

        <logger name="org.terasoluna.gfw.web.logging.TraceLoggingInterceptor">
            <level value="trace" />
        </logger>

        <!-- 3rdparty Loggers -->
        <logger name="org.springframework">
            <level value="warn" />
        </logger>

        <logger name="org.springframework.web.servlet">
            <level value="info" />
        </logger>

        <!-- (8) -->
        <logger name="org.hibernate.SQL">
            <level value="debug" />
        </logger>

        <!-- (9) -->
        <logger name="org.hibernate.type.descriptor.sql.BasicBinder">
            <level value="trace" />
        </logger>

        <!-- (10) -->
        <logger name="org.hibernate.engine.transaction">
            <level value="debug" />
        </logger>

        <root level="WARN">
            <appender-ref ref="STDOUT" />
        </root>
    </configuration>

.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (8)
     - | HibernateによるSQLログを、出力するための設定。
   * - | (9)
     - | HibernateによるSQLのバインド変数を、出力するための設定。
   * - | (10)
     - | Hibernateによるトランザクションのログを、出力するための設定。


Entityの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Todoクラスを、データベースとマッピングするために、JPAのアノテーションを設定する。

.. code-block:: java
   :emphasize-lines: 6-11,13-15,19-22,25,28,31,33

    package todo.domain.model;

    import java.io.Serializable;
    import java.util.Date;

    import javax.persistence.Column;
    import javax.persistence.Entity;
    import javax.persistence.Id;
    import javax.persistence.Table;
    import javax.persistence.Temporal;
    import javax.persistence.TemporalType;

    // (1)
    @Entity
    @Table(name = "todo")
    public class Todo implements Serializable {
        private static final long serialVersionUID = 1L;

        // (2)
        @Id
        // (3)
        @Column(name = "todo_id")
        private String todoId;

        @Column(name = "todo_title")
        private String todoTitle;

        @Column(name = "finished")
        private boolean finished;

        @Column(name = "created_at")
        // (4)
        @Temporal(TemporalType.TIMESTAMP)
        private Date createdAt;

        public String getTodoId() {
            return todoId;
        }

        public void setTodoId(String todoId) {
            this.todoId = todoId;
        }

        public String getTodoTitle() {
            return todoTitle;
        }

        public void setTodoTitle(String todoTitle) {
            this.todoTitle = todoTitle;
        }

        public boolean isFinished() {
            return finished;
        }

        public void setFinished(boolean finished) {
            this.finished = finished;
        }

        public Date getCreatedAt() {
            return createdAt;
        }

        public void setCreatedAt(Date createdAt) {
            this.createdAt = createdAt;
        }
    }

.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | JPAのエンティティであることを示す\ ``@Entity``\ アノテーションを付け、対応するテーブル名を\ ``@Table``\ アノテーションで設定する。
   * - | (2)
     - | 主キーとなるカラムに対応するフィールドに、\ ``@Id``\ アノテーションをつける。
   * - | (3)
     - | \ ``@Column``\ アノテーションで、対応するカラム名を設定する。
   * - | (4)
     - | Date型は、java.sql.Date, java.sql.Time, java.sql.Timestampのどれに対応するか、明示的に指定する必要がある。ここでは、Timestampを指定する。


TodoRepositoryの修正
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: java
   :emphasize-lines: 3-5,10-12

    package todo.domain.repository.todo;

    import org.springframework.data.jpa.repository.JpaRepository;
    import org.springframework.data.jpa.repository.Query;
    import org.springframework.data.repository.query.Param;

    import todo.domain.model.Todo;

    // (1)
    public interface TodoRepository extends JpaRepository<Todo, String> {
        @Query(value = "SELECT COUNT(x) FROM Todo x WHERE x.finished = :finished") // (2)
        long countByFinished(@Param("finished") boolean finished); // (3)
    }


.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | JpaRepositoryを拡張したインタフェースにする。Genericsのパラメータには、順にEntityのクラス(Todo)、主キーのクラス(String)を指定する。
       | 基本的なCRUD操作(findOne, findAll, save, deleteなど)は、上位のインタフェースに定義済みであるため、TodoRepositoryでは、countByFinishedのみ定義すればよい。
   * - | (2)
     - | countByFinishedを呼び出した際に、実行されるJPQLを、\ ``@Query``\ アノテーションで指定する。
   * - | (3)
     - | (2)で指定したJPQLのバインド変数を\ ``@Param``\ アノテーションで設定する。
       | ここでは、JPQL中の\ ``”:finished”``\ を埋めるためのメソッド引数に、@Param(“finished”)を付けている。


TodoRepositoryImplの修正
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Spring Data JPAを使用した場合、RepositoryImplは、インタフェースから自動生成される。
| TodoRepositoryImplは、不要であるため、削除する。

| 以上で、Spring Data JPAを使う対応が完了した。
| APサーバーを起動し、Todoの表示を行うと、以下のようなSQLログや、トランザクションログが出力される。

.. code-block:: guess
   :emphasize-lines: 2-6

    2014-03-27 11:31:13 [tomcat-http--50] [TRACE] [o.t.gfw.web.logging.TraceLoggingInterceptor     ] - [START CONTROLLER] TodoController.list(Model)
    2014-03-27 11:31:13 [tomcat-http--50] [DEBUG] [o.h.e.transaction.spi.AbstractTransactionImpl   ] - begin
    2014-03-27 11:31:13 [tomcat-http--50] [DEBUG] [o.h.e.transaction.internal.jdbc.JdbcTransaction ] - initial autocommit status: false
    2014-03-27 11:31:13 [tomcat-http--50] [DEBUG] [org.hibernate.SQL                               ] - /* select generatedAlias0 from Todo as generatedAlias0 */ select todo0_.todo_id as todo1_0_, todo0_.created_at as created2_0_, todo0_.finished as finished3_0_, todo0_.todo_title as todo4_0_ from todo todo0_
    2014-03-27 11:31:13 [tomcat-http--50] [DEBUG] [o.h.e.transaction.spi.AbstractTransactionImpl   ] - committing
    2014-03-27 11:31:13 [tomcat-http--50] [DEBUG] [o.h.e.transaction.internal.jdbc.JdbcTransaction ] - committed JDBC Connection
    2014-03-27 11:31:13 [tomcat-http--50] [TRACE] [o.t.gfw.web.logging.TraceLoggingInterceptor     ] - [END CONTROLLER  ] TodoController.list(Model)-> view=todo/list, model={todoForm=todo.app.todo.TodoForm@211dcc, todos=[], org.springframework.validation.BindingResult.todoForm=org.springframework.validation.BeanPropertyBindingResult: 0 errors}
    2014-03-27 11:31:13 [tomcat-http--50] [TRACE] [o.t.gfw.web.logging.TraceLoggingInterceptor     ] - [HANDLING TIME   ] TodoController.list(Model)-> 2,959,644 ns


.. _using_terasolunaDao:

TERASOLUNA DAOを使用する
--------------------------------------------------------------------------------
本節では、インフラストラクチャ層において、TERASOLUNA DAOを使用する場合の設定方法について説明する。

 .. note::
    TERASOLUNA DAOとは、MyBatis2.3.5とSpringの連携クラスであるorg.springframework.orm.ibatis.support.SqlMapClientDaoSupportを、用途別に拡張した簡易SQLマッパーを提供するライブラリである。
    以下4つのインタフェースをもつDAOが、提供されている。

       #. jp.terasoluna.fw.dao.QueryDAO
       #. jp.terasoluna.fw.dao.UpdateDAO
       #. jp.terasoluna.fw.dao.StoredProcedureDAO
       #. jp.terasoluna.fw.dao.QueryRowHandleDAO

    それぞれのインタフェースに対して、\ ``jp.terasoluna.fw.dao.ibatis.XxxDAOiBatisImpl``\ という実装を持つ。


TERASOLUNA DAOを使用するための設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

pom.xmlの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
TERASOLUNA DAOに関する依存ライブラリを追加するために、pom.xmlに、以下の内容を追加する。

.. code-block:: xml

    <dependency>
        <groupId>org.terasoluna.gfw</groupId>
        <artifactId>terasoluna-gfw-mybatis2</artifactId>
    </dependency>


todo-infra.xmlの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
todo-infra.xmlに、TERASOLUNA DAOを使用するための設定を行う。

.. code-block:: xml
   :emphasize-lines: 8-37

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

        <import resource="classpath:/META-INF/spring/todo-env.xml" />

        <!-- (1) -->
        <bean id="sqlMapClient"
            class="org.springframework.orm.ibatis.SqlMapClientFactoryBean">
            <!-- (2) -->
            <property name="configLocations"
                value="classpath*:/META-INF/mybatis/config/*sqlMapConfig.xml" />
            <!-- (3) -->
            <property name="mappingLocations"
                value="classpath*:/META-INF/mybatis/sql/**/*-sqlmap.xml" />
            <property name="dataSource" ref="dataSource" />
        </bean>

        <!-- (4) -->
        <bean id="queryDAO" class="jp.terasoluna.fw.dao.ibatis.QueryDAOiBatisImpl">
            <property name="sqlMapClient" ref="sqlMapClient" />
        </bean>

        <bean id="updateDAO" class="jp.terasoluna.fw.dao.ibatis.UpdateDAOiBatisImpl">
            <property name="sqlMapClient" ref="sqlMapClient" />
        </bean>

        <bean id="spDAO"
            class="jp.terasoluna.fw.dao.ibatis.StoredProcedureDAOiBatisImpl">
            <property name="sqlMapClient" ref="sqlMapClient" />
        </bean>

        <bean id="queryRowHandleDAO"
            class="jp.terasoluna.fw.dao.ibatis.QueryRowHandleDAOiBatisImpl">
            <property name="sqlMapClient" ref="sqlMapClient" />
        </bean>
    </beans>


.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | SqlMapClientの定義を行う。
   * - | (2)
     - | SqlMap設定ファイルのパスを設定する。ここでは、META-INF/mybatis/config以下の、\*sqlMapConfig.xmlを読み込む。
   * - | (3)
     - | SqlMapファイルのパスを設定する。ここでは、META-INF/mybatis/sql以下の、任意のフォルダの*-sqlmap.xmlを読み込む。
   * - | (4)
     - | TERASOLUNA DAOの定義を行う。


todo-env.xmlの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| トランザクションマネージャの定義を追加する。
| JavaEEコンテナ上で、トランザクションマネージャは、JtaTransactionManagerを使用することもあるため、環境依存設定ファイルに定義する。

.. code-block:: xml
   :emphasize-lines: 19-23

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

        <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
            destroy-method="close">
            <property name="driverClassName" value="${database.driverClassName}" />
            <property name="url" value="${database.url}" />
            <property name="username" value="${database.username}" />
            <property name="password" value="${database.password}" />
            <property name="defaultAutoCommit" value="false" />
            <property name="maxActive" value="${cp.maxActive}" />
            <property name="maxIdle" value="${cp.maxIdle}" />
            <property name="minIdle" value="${cp.minIdle}" />
            <property name="maxWait" value="${cp.maxWait}" />
        </bean>

        <!-- (1) -->
        <bean id="transactionManager"
            class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <property name="dataSource" ref="dataSource" />
        </bean>

    </beans>

.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | トランザクションマネージャの設定。idは、transactionManagerにすること。
       | 別の名前を指定する場合は、<tx:annotation-driven>タグにも、トランザクションマネージャ名を指定する必要がある。


logback.xmlの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. code-block:: xml
   :emphasize-lines: 32-43

    <!DOCTYPE logback>
    <configuration>
        <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern><![CDATA[%d{yyyy-MM-dd HH:mm:ss} [%thread] [%-5level] [%-48logger{48}] - %msg%n]]></pattern>
            </encoder>
        </appender>

        <!-- Application Loggers -->
        <logger name="todo">
            <level value="debug" />
        </logger>

        <!-- TERASOLUNA -->
        <logger name="org.terasoluna.gfw">
            <level value="info" />
        </logger>

        <logger name="org.terasoluna.gfw.web.logging.TraceLoggingInterceptor">
            <level value="trace" />
        </logger>

        <!-- 3rdparty Loggers -->
        <logger name="org.springframework">
            <level value="warn" />
        </logger>

        <logger name="org.springframework.web.servlet">
            <level value="info" />
        </logger>

        <!-- (8) -->
        <logger name="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <level value="debug" />
        </logger>

        <!-- (9) -->
        <logger name="java.sql.Connection">
            <level value="trace" />
        </logger>
        <logger name="java.sql.PreparedStatement">
            <level value="debug" />
        </logger>

        <root level="WARN">
            <appender-ref ref="STDOUT" />
        </root>
    </configuration>


.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (8)
     - | DataSourceTransactionManagerによる、トランザクションのログを出力するための設定。
   * - | (9)
     - | SQLログを出力するための設定。


sqlMapConfigの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
src/main/resources/META-INF/mybatis/config/sqlMapConfig.xmlを作成し、以下のように記述する。

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE sqlMapConfig
                PUBLIC "-//ibatis.apache.org//DTD SQL Map Config 2.0//EN"
                "http://ibatis.apache.org/dtd/sql-map-config-2.dtd">
    <sqlMapConfig>
        <!-- (1) -->
        <settings useStatementNamespaces="true" />
    </sqlMapConfig>

.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | SQLIDに、名前空間を与える設定を行う。


RepositoryImplの修正
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: java
   :emphasize-lines: 5,7,8,11,16,17,19-21,23-24,28,30,34,36,41-47,50,52,63,65

    package todo.domain.repository.todo;

    import java.util.Collection;

    import javax.inject.Inject;

    import jp.terasoluna.fw.dao.QueryDAO;
    import jp.terasoluna.fw.dao.UpdateDAO;

    import org.springframework.stereotype.Repository;
    import org.springframework.transaction.annotation.Transactional;

    import todo.domain.model.Todo;

    @Repository
    // (1)
    @Transactional
    public class TodoRepositoryImpl implements TodoRepository {
        // (2)
        @Inject
        protected QueryDAO queryDAO;

        @Inject
        protected UpdateDAO updateDAO;

        // (3)
        @Override
        @Transactional(readOnly = true)
        public Todo findOne(String todoId) {
            return queryDAO.executeForObject("todo.findOne", todoId, Todo.class);
        }

        @Override
        @Transactional(readOnly = true)
        public Collection<Todo> findAll() {
            return queryDAO.executeForObjectList("todo.findAll", null);
        }

        @Override
        public Todo save(Todo todo) {
            // (4)
            if (exists(todo.getTodoId())) {
                updateDAO.execute("todo.update", todo);
            } else {
                updateDAO.execute("todo.create", todo);
            }
            return todo;
        }

        @Transactional(readOnly = true)
        public boolean exists(String todoId) {
            long count = queryDAO.executeForObject("todo.exists", todoId,
                    Long.class);
            return count > 0;
        }

        @Override
        public void delete(Todo todo) {
            updateDAO.execute("todo.delete", todo);
        }

        @Override
        @Transactional(readOnly = true)
        public long countByFinished(boolean finished) {
            return queryDAO.executeForObject("todo.countByFinished", finished,
                    Long.class);
        }
    }


.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | クラスレベルに、\ ``@Transactional``\ アノテーションをつけることで、公開メソッドをすべてトランザクション管理する。
       | Repositoryを呼び出すService側でも設定しているため、\ ``@Transactional``\ をつけなくともトランザクション管理になるが、propagation属性は、デフォルトのREQUIREDであるため、トランザクションがネストした場合、内側(Repository側)のトランザクションは、外側(Service側)のトランザクションに参加する。
   * - | (2)
     - | \ ``@Inject``\ アノテーションで、QueryDAO, UpdateDAOをインジェクションする。
   * - | (3)
     - | Repositoryのメソッド実装は、基本的には、TERASOLUNA DAOにSQLIDと、パラメータを渡すことになる。
       | 参照系の場合はQueryDAO、更新系の場合はUpdateDAOを使用する。SQLIDに対応するSQLの設定は、次に行う。
   * - | (4)
     - | saveメソッドで新規作成と、更新の両方を実装している。
       | どちらの処理を行うか判断するために、existsメソッドを作成する。
       | このメソッドでは、対象のtodoIdの件数を取得し、件数が0より大きいかどうかで存在を確認する。

.. note::

    saveメソッドは、新規作成でも更新でも利用できるメリットがある。
    しかしながら、2回SQLが実行されるという性能面でのデメリットもある。
    性能を重視する場合は、新規作成用にcreateメソッドを、更新用にupdateメソッドを作成すること。


SQLMapファイルの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
src/main/resources/META-INF/mybatis/sql/todo-sqlmap.xmlを作成し、TodoRepositoryImplで使用したSQLIDに対応するsqlを、以下のように記述する。

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE sqlMap
                PUBLIC "-//ibatis.apache.org//DTD SQL Map 2.0//EN"
                "http://ibatis.apache.org/dtd/sql-map-2.dtd">
    <sqlMap namespace="todo">
        <resultMap id="todo" class="todo.domain.model.Todo">
            <result property="todoId" column="todo_id" />
            <result property="todoTitle" column="todo_title" />
            <result property="finished" column="finished" />
            <result property="createdAt" column="created_at" />
        </resultMap>

        <select id="findOne" parameterClass="java.lang.String"
            resultMap="todo"><![CDATA[
    SELECT todo_id,
           todo_title,
           finished,
           created_at
    FROM   todo
    WHERE  todo_id = #value#
    ]]></select>

        <select id="findAll" resultMap="todo"><![CDATA[
    SELECT todo_id,
           todo_title,
           finished,
           created_at
    FROM   todo
    ]]></select>

        <insert id="create" parameterClass="todo.domain.model.Todo"><![CDATA[
    INSERT INTO todo
                (todo_id,
                 todo_title,
                 finished,
                 created_at)
    VALUES      ( #todoId#,
                 #todoTitle#,
                 #finished#,
                 #createdAt# )
    ]]></insert>

        <update id="update" parameterClass="todo.domain.model.Todo"><![CDATA[
    UPDATE todo
    SET    todo_title = #todoTitle#,
           finished = #finished#,
           created_at = #createdAt#
    WHERE  todo_id = #todoId#
    ]]></update>

        <delete id="delete" parameterClass="todo.domain.model.Todo"><![CDATA[
    DELETE FROM todo
    WHERE  todo_id = #todoId#
    ]]></delete>

        <select id="countByFinished" parameterClass="java.lang.Boolean"
            resultClass="java.lang.Long"><![CDATA[
    SELECT COUNT(*)
    FROM   todo
    WHERE  finished = #value#
    ]]></select>

        <select id="exists" parameterClass="java.lang.String"
            resultClass="java.lang.Long"><![CDATA[
    SELECT COUNT(*)
    FROM   todo
    WHERE  todo_id = #value#
    ]]></select>
    </sqlMap>

| 以上で、TERASOLUNA DAOを使う対応が完了した。
| APサーバーを起動し、Todoの表示を行うと、以下のようなSQLログやトランザクションログが出力される。

.. code-block:: guess
   :emphasize-lines: 2-12

    2013-11-28 14:15:37 [tomcat-http--3] [TRACE] [o.t.gfw.web.logging.TraceLoggingInterceptor     ] - [START CONTROLLER] TodoController.list(Model)
    2013-11-28 14:15:37 [tomcat-http--3] [DEBUG] [o.s.jdbc.datasource.DataSourceTransactionManager] - Creating new transaction with name [todo.domain.serivce.todo.TodoServiceImpl.findAll]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT,readOnly; ''
    2013-11-28 14:15:37 [tomcat-http--3] [DEBUG] [o.s.jdbc.datasource.DataSourceTransactionManager] - Acquired Connection [jdbc:h2:mem:todo, UserName=SA, H2 JDBC Driver] for JDBC transaction
    2013-11-28 14:15:37 [tomcat-http--3] [DEBUG] [o.s.jdbc.datasource.DataSourceTransactionManager] - Participating in existing transaction
    2013-11-28 14:15:37 [tomcat-http--3] [DEBUG] [java.sql.Connection                             ] - {conn-100014} Connection
    2013-11-28 14:15:37 [tomcat-http--3] [DEBUG] [java.sql.Connection                             ] - {conn-100014} Preparing Statement:  SELECT todo_id,        todo_title,        finished,        created_at FROM   todo 
    2013-11-28 14:15:37 [tomcat-http--3] [DEBUG] [java.sql.PreparedStatement                      ] - {pstm-100015} Executing Statement:  SELECT todo_id,        todo_title,        finished,        created_at FROM   todo 
    2013-11-28 14:15:37 [tomcat-http--3] [DEBUG] [java.sql.PreparedStatement                      ] - {pstm-100015} Parameters: []
    2013-11-28 14:15:37 [tomcat-http--3] [DEBUG] [java.sql.PreparedStatement                      ] - {pstm-100015} Types: []
    2013-11-28 14:15:37 [tomcat-http--3] [DEBUG] [o.s.jdbc.datasource.DataSourceTransactionManager] - Initiating transaction commit
    2013-11-28 14:15:37 [tomcat-http--3] [DEBUG] [o.s.jdbc.datasource.DataSourceTransactionManager] - Committing JDBC transaction on Connection [jdbc:h2:mem:todo, UserName=SA, H2 JDBC Driver]
    2013-11-28 14:15:37 [tomcat-http--3] [DEBUG] [o.s.jdbc.datasource.DataSourceTransactionManager] - Releasing JDBC Connection [jdbc:h2:mem:todo, UserName=SA, H2 JDBC Driver] after transaction
    2013-11-28 14:15:37 [tomcat-http--3] [TRACE] [o.t.gfw.web.logging.TraceLoggingInterceptor     ] - [END CONTROLLER  ] TodoController.list(Model)-> view=todo/list, model={todoForm=todo.app.todo.TodoForm@1386751, todos=[todo.domain.model.Todo@72edc], org.springframework.validation.BindingResult.todoForm=org.springframework.validation.BeanPropertyBindingResult: 0 errors}
    2013-11-28 14:15:37 [tomcat-http--3] [TRACE] [o.t.gfw.web.logging.TraceLoggingInterceptor     ] - [HANDLING TIME   ] TodoController.list(Model)-> 2,461,131 ns

|

おわりに
================================================================================
このチュートリアルでは、以下の内容を学習した。

* TERASOLUNA Global Frameworkによる基本的なアプリケーションの開発方法、およびEclipseプロジェクトの構築方法

 * STSの使用方法
 * MavenでTERASOLUNA Global Frameworkを使用する方法

* TERASOLUNA Global Frameworkのアプリケーションのレイヤ化に従った開発方法

 * POJO(+ Spring)によるドメイン層の実装
 * Spring MVCとJSPタグライブラリを使用したアプリケーション層の実装
 * Spring Data JPAによるインフラストラクチャ層の実装
 * MyBatis2によるインフラストラクチャ層の実装

ここで作成したTODO管理アプリケーションには、以下の改善点がある。
アプリケーションの修正を学習課題として、ガイドライン中の該当する説明を参照されたい。

* プロパティを外部化する → :doc:`../ArchitectureInDetail/PropertyManagement`
* メッセージを外部化する → :doc:`../ArchitectureInDetail/MessageManagement`
* ページング処理を追加する → :doc:`../ArchitectureInDetail/Pagination`
* 例外ハンドリングを加える → :doc:`../ArchitectureInDetail/ExceptionHandling`
* CSRF対策を追加する → :doc:`../Security/CSRF`



