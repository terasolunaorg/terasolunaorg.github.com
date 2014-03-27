はじめてのSpring MVCアプリケーション
--------------------------------------------------------------

.. contents:: 目次
   :depth: 3
   :local:

Spring MVCの、詳細な使い方の解説に入る前に、実際にSpring MVCに触れることで、
Spring MVCを用いたWebアプリケーションの開発に対するイメージをつかむ。

本節は、全体イメージをつかむことを目的としており、 **次章以降で説明する推奨方式に従っていないことに注意する。**

検証環境
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

本節の説明では、次の環境で動作検証している。(他の環境で実施する際は、本書をベースに適宜読み替えて設定していくこと。)

.. list-table::
   :header-rows: 1
   :widths: 75 25

   * - Product
     - Version
   * - JDK
     - 1.6.0\_33
   * - SpringSource Tool Suite (STS)
     - 3.2.0
   * - VMware vFabric tc Server Developer Edition
     - 2.8
   * - Fire Fox
     - 21.0

.. note::

  インターネット接続するために、プロキシサーバーを介する必要がある場合、
  以下の作業を行うため、STSのProxy設定と、 `MavenのProxy設定 <http://maven.apache.org/guides/mini/guide-proxies.html>`_\ が必要である。


.. warning::

  この節で使用するSpring Tool Suiteの「Spring Template Project」はSpiring Tool Suite 3.4から廃止されたため、この節の内容はSpiring Tool Suite 3.3以前でしか確認できない。
  
  Spiring Tool Suite 3.4を使用する場合は\ :doc:`../Appendix/CreateProjectFromBlank`\ を参照されたい。
  
  今後、検証環境を更新する予定である。

新規プロジェクト作成
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

SpringSource Tool Suiteのメニューから、[File] -> [New] -> [Spring Template Project] -> [Spring MVC Project] -> [Next]を選択し、
Spring MVCプロジェクトを作成する。

.. note::

    Proxyサーバーを経由している場合、Spring Template Projectが選択できないことがある。
    これを解決するには、以下のように、実施すること。

    * まず、[window] -> [Preferences] -> [Spring] -> [Template Projects] を選択し、"spring-defaults"を残して、他項目をRemoveする。
    * 次に、Preferences右下のApplyをクリックする。
    * 最後に、[File] -> [New] -> [Spring Project]をクリックすると、Templetesから[Spring MVC Project]が、選択できるようになる。



次の画面で、[Project Name]に"helloworld"、[Please specify the top-level package]に"com.example.helloworld"を入力し、[Finish]をクリックする。

.. figure:: images/NewSpringMVCProject.png
   :alt: New Spring MVC Project
   :width: 60%

Package Explorerに、次のようなプロジェクトが生成される( **要インターネット接続** )。

.. figure:: images/HelloWorldWorkspace.png
   :alt: workspace

Spring MVCの設定方法を理解するために、生成されたSpring MVCの設定ファイル(src/main/webapp/WEB-INF/spring/appServlet/servlet-context.xml)について、簡単に説明する。

.. literalinclude:: ../../resources/helloworld/src/main/webapp/WEB-INF/spring/appServlet/servlet-context.xml
   :language: xml


.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - <annotation-driven /> を定義することにより、Spring MVCのデフォルト設定が行われる。デフォルトの設定については、 Springの公式ページである `Enabling the MVC Java Config or the MVC XML Namespace <http://docs.spring.io/spring/docs/3.2.x/spring-framework-reference/html/mvc.html#mvc-config-enable>`_ を参照されたい。
   * - | (2)
     - ViewのResolverを指定し、Viewの配置場所を定義する。
   * - | (3)
     - Spring MVCで使用するコンポーネントを探すパッケージを定義する。

``com.example.helloworld.HomeController`` を以下に示す(ただし、説明用に、シンプルな形に修正している)。

.. literalinclude:: ../../resources/helloworld/src/main/java/com/example/helloworld/HomeController.java
   :language: java
   :emphasize-lines: 15,24,25,31

簡単に、解説を行う。


.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - ``@Controller`` アノテーションを付けることで、DIコンテナにより、コントローラクラスが自動で読み込まれる。前述「Spring MVCの設定ファイルの説明(3)」の設定により、component-scanの対象となっている。
   * - | (2)
     - HTTPメソッドがGETで、Resource（もしくはRequest URL）が"/"で、アクセスする際に実行される。
   * - | (3)
     - Viewに渡したいオブジェクトを設定する。
   * - | (4)
     - View名を返却する。前述「Spring MVCの設定ファイルの説明(2)」の設定により、"WEB-INF/views/home.jsp"がレンダリングされる。

Modelに設定したオブジェクトが、HttpServletRequestに設定される。
home.jspで以下のように ``${serverTime}`` を記述することで、Controllerから渡された値を出力することができる。
( **ただし、${XXX}の記述は、XSS対象になる可能性があるので、文字列を出力する場合は、後述のようにHTMLエスケープする必要があることに注意する。** )

.. literalinclude:: ../../resources/helloworld/src/main/webapp/WEB-INF/views/home.jsp
   :language: jsp

|

サーバーを起動する
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
| STSで、"helloworld"プロジェクトを右クリックして、"Run As" -> "Run On Server" -> "localhost" -> "VMware vFabric tc Server Developer Edition v2.8" -> "Finish"を実行し、helloworldプロジェクトを起動する。
| ブラウザに "http://localhost:8080/helloworld/" を入力し、実行すると下記の画面が表示される。

.. figure:: images/AppHelloWorldIndex.png
   :alt: Hello World

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
| \ ``com.example.helloworld.echo``\ パッケージに\ ``EchoForm``\ クラスを作成する。プロパティを1つだけ持つ、単純なJavaBeanである。

.. literalinclude:: ../../resources/helloworld/src/main/java/com/example/helloworld/echo/EchoForm.java
   :language: java

|

Controllerの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 次に、Controllerを作成する。
| 同じく ``com.example.helloworld.echo`` パッケージに、``EchoController`` クラスを作成する。

.. literalinclude:: ../../resources/helloworld/src/main/java/com/example/helloworld/echo/EchoController.java
   :language: java
   :emphasize-lines: 12,18,20,23-25

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
     - | 引数に、EchoFormには(1)によりModelに追加されたEchoFormオブジェクトが渡される。
   * - | (4)
     - | View名で"echo/index"を返すので、ViewResolverにより、 "WEB-INF/views/echo/index.jsp"がレンダリングされる。
   * - | (5)
     - | メソッドに付加した ``@RequestMapping`` アノテーションに"hello"を指定しているので、この場合、"<contextPath>/echo/hello"にアクセスすると ``hello`` メソッドが呼ばれる。
   * - | (6)
     - | フォームで入力された ``name`` を、Viewにそのまま渡す。

|

JSPの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
最後に、入力画面と、出力画面のJSPを作成する。それぞれのファイルパスは、View名に合わせて、次のようになる。

* 入力画面 src/main/webapp/WEB-INF/views/echo/index.jsp

.. literalinclude:: ../../resources/helloworld/src/main/webapp/WEB-INF/views/echo/index.jsp
   :language: jsp

.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | タグライブラリを利用し、HTMLフォームを構築している。 ``modelAttribute`` 属性に、Controllerで用意したフォームオブジェクトの名前を指定する。
       | タグライブラリは `こちら <http://docs.spring.io/spring/docs/3.2.x/spring-framework-reference/html/view.html#view-jsp-formtaglib-formtag>`_\を参照されたい。

出力されるHTMLは、

.. code-block:: html

    <body>

      <form id="echoForm" action="/helloworld/echo/hello" method="post">
        <label for="name">Input Your Name:</label>
        <input id="name" name="name" type="text" value=""/>
        <input type="submit" />
      </form>
    </body>
    </html>

となる。

* 出力画面 src/main/webapp/WEB-INF/views/echo/hello.jsp

.. literalinclude:: ../../resources/helloworld/src/main/webapp/WEB-INF/views/echo/hello.jsp
   :language: jsp

Controllerから渡された"name"を出力する。 ``c:out`` タグにより、XSS対策を行っている。

|

.. note::

    ここではXSS対策を標準タグの ``c:out`` で実現したが、より容易に使用できる ``f:h()`` 関数を共通ライブラリで用意している。
    詳細は、  :doc:`../Security/XSS` を参照されたい。


| これでエコーアプリケーションの実装は完了である。
| サーバーを起動し、 "http://localhost:8080/helloworld/echo"にアクセスするとフォームが表示される。

|

入力チェックの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
ここまでのアプリケーションでは、入力チェックを行っていない。
Spring MVCでは、Bean Validation (`JSR-303 <http://jcp.org/en/jsr/detail?id=303>`_\)をサポートしており、アノテーションベースな入力チェックを、簡単に
実装することができる。例として、エコーアプリケーションで名前の入力チェックを行う。

Bean Validationを利用するために、pom.xmlのdependenciesの中に、以下のdependencyを追加する。

.. literalinclude:: ../../resources/helloworld2/pom.xml
   :language: xml
   :lines: 112-117

``EchoForm`` を以下の通り、 ``name`` プロパティに ``@NotNull`` アノテーションおよび、 ``@Size`` アノテーションを付加する(getterメソッドに付加してもよい)。


.. literalinclude:: ../../resources/helloworld2/src/main/java/com/example/helloworld/echo/EchoForm.java
   :language: java
   :emphasize-lines: 5,6,11,12

.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | ``@NotNull`` アノテーションをつけることで、HTTPリクエスト中に ``name`` パラメータがあることを確認する。
   * - | (2)
     - | ``@Size(min = 1, max = 5)`` をつけることで、``name`` のサイズが、1以上5以下であることを確認する。

``EchoController`` を以下の通り、 ``@Valid`` アノテーションを追加し、 ``hasErrors`` メソッドを追加する。


.. literalinclude:: ../../resources/helloworld2/src/main/java/com/example/helloworld/echo/EchoController.java
   :language: java
   :emphasize-lines: 3,7,27-30

.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | コントローラー側には、Validation対象の引数に ``@Valid`` アノテーションを付加し、 ``BindingResult`` オブジェクトを引数に追加する。
       | Bean Validationによる入力チェックは、自動で行われる。結果は、 ``BindingResult`` オブジェクトに渡される。
   * - | (2)
     - | ``hasErrors`` メソッドを実行して、エラーがあるかどうかを確認できる。

入力画面 src/main/webapp/WEB-INF/views/echo/index.jsp に、 ``form:errors`` タグを追加する。


.. literalinclude:: ../../resources/helloworld2/src/main/webapp/WEB-INF/views/echo/index.jsp
   :language: jsp
   :emphasize-lines: 15

.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 入力画面には、エラーがあった場合に、エラーメッセージを表示するため、 ``form:errors`` タグを追加する。


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

    <body>

      <form id="echoForm" action="/helloworld/echo/hello" method="post">
        <label for="name">Input Your Name:</label>
        <input id="name" name="name" type="text" value=""/>
        <span id="name.errors" style="color:red">size must be between 1 and 5</span>
        <input type="submit" />
      </form>
    </body>
    </html>

となる。

まとめ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

この章では、

#. 最も簡易な、SpringMVCの設定ファイル設定方法
#. 最も簡易な、画面遷移方法
#. 画面間での値の引き渡し方法
#. シンプルな入力チェック方法

を学んだ。

上記の内容が理解できていない場合は、もう一度、本節を読み、環境構築から始めて、進めていくことで理解が深まる。

