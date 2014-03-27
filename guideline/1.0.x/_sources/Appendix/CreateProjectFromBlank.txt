Blankプロジェクトから新規プロジェクト作成
================================================================================

Blankプロジェクトからの新規プロジェクトを作成する方法を説明する。

* 前提

  * Spring Tool Suite が動作可能なこと。
  * Mavenをコマンドラインでで実行可能なこと。
  * インターネットに接続できること。

この節の内容は以下のバージョンで動作確認している。

.. list-table::
   :header-rows: 1
   :widths: 75 25

   * - Product
     - Version
   * - JDK
     - 1.7.0\_40
   * - Spring Tool Suite (STS)
     - 3.4.0
   * - Maven
     - 3.0.4

.. _CreateProjectFromBlank_create-new-project:


.. note::

  インターネット接続するために、プロキシサーバーを介する必要がある場合、
  以下の作業を行うため、STSのProxy設定と、 `MavenのProxy設定 <http://maven.apache.org/guides/mini/guide-proxies.html>`_\ が必要である。

新規プロジェクト作成
--------------------------------------------------------------------------------

#. プロジェクトフォルダにコマンドプロンプトで移動する。
  
    ここではSTSのworkspaceに移動する。
    
    .. code-block:: console
    
        $ cd C:\springsource\sts-3.4.0.RELEASE\workspace

#. Maven archetypeを利用してプロジェクトの雛形を生成する

    .. code-block:: console
    
        $ mvn archetype:generate -DarchetypeCatalog=http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases # <== (1)
        [INFO] Scanning for projects...
        [INFO]
        [INFO] ------------------------------------------------------------------------
        [INFO] Building Maven Stub Project (No POM) 1
        [INFO] ------------------------------------------------------------------------
        [INFO]
        [INFO] >>> maven-archetype-plugin:2.2:generate (default-cli) @ standalone-pom >>>
        [INFO]
        [INFO] <<< maven-archetype-plugin:2.2:generate (default-cli) @ standalone-pom <<<
        [INFO]
        [INFO] --- maven-archetype-plugin:2.2:generate (default-cli) @ standalone-pom ---
        [INFO] Generating project in Interactive mode
        [INFO] No archetype defined. Using maven-archetype-quickstart (org.apache.maven.archetypes:maven-archetype-quickstart:1.0)
        Choose archetype:
        1: http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases -> org.terasoluna.gfw.blank:terasoluna-gfw-web-blank-archetype (Blank project using TERASOLUNA Global Framework)
        2: http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases -> org.terasoluna.gfw.blank:terasoluna-gfw-web-blank-jpa-archetype (Blank project using TERASOLUNA Global Framework (JPA))
        3: http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases -> org.terasoluna.gfw.blank:terasoluna-gfw-web-blank-mybatis2-archetype (Blank project using TERASOLUNA Global Framework (MyBatis2))
        Choose a number or apply filter (format: [groupId:]artifactId, case sensitive contains): : 1 # <== (2)
        Define value for property 'groupId': : example.first.application # <== (3)
        Define value for property 'artifactId': : example-first-application # <== (4)
        Define value for property 'version':  1.0-SNAPSHOT: : # <== (5)
        Define value for property 'package':  example.first.application: : # <== (6)
        Confirm properties configuration:
        groupId: example.first.application
        artifactId: example-first-application
        version: 1.0-SNAPSHOT
        package: example.first.application
         Y: : # <== (7)
        [INFO] ----------------------------------------------------------------------------
        [INFO] Using following parameters for creating project from Archetype: terasoluna-gfw-web-blank-archetype:1.0.0.RELEASE
        [INFO] ----------------------------------------------------------------------------
        [INFO] Parameter: groupId, Value: example.first.application
        [INFO] Parameter: artifactId, Value: example-first-application
        [INFO] Parameter: version, Value: 1.0-SNAPSHOT
        [INFO] Parameter: package, Value: example.first.application
        [INFO] Parameter: packageInPathFormat, Value: example/first/application
        [INFO] Parameter: package, Value: example.first.application
        [INFO] Parameter: version, Value: 1.0-SNAPSHOT
        [INFO] Parameter: groupId, Value: example.first.application
        [INFO] Parameter: artifactId, Value: example-first-application
        [INFO] project created from Archetype in dir: C:\springsource\sts-3.4.0.RELEASE\workspace\example-first-application
        [INFO] ------------------------------------------------------------------------
        [INFO] BUILD SUCCESS
        [INFO] ------------------------------------------------------------------------
        [INFO] Total time: 44.809s
        [INFO] Finished at: Fri Dec 06 11:28:47 JST 2013
        [INFO] Final Memory: 9M/23M
        [INFO] ------------------------------------------------------------------------
    
    
    .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - 項番
         - 説明
       * - | (1)
         - | Maven archetypeでプロジェクトを作成する。TERASOLUNA Global Frameworkのレポジトリを指定する。
       * - | (2)
         - | 取得するBlank Projectのtypeを指定する。
           |    1. JPA,MyBatis2は使用せずにプロジェクトを作成する場合
           |    2. JPAを使用するプロジェクトを作成する場合
           |    3. MyBatis2を使用するプロジェクトを作成する場合
       * - | (3)
         - | groupIdを指定する。
       * - | (4)
         - | artifactIdを指定する。
       * - | (5)
         - | 省略可。最新版を利用する場合は指定不要。
       * - | (6)
         - | 省略可。パッケージ名指定。groupIdと同じ場合は指定不要。
       * - | (7)
         - | 省略可。groupId,artifactId,version,packageが問題なければ、Enterで終了となる。Nを指定した場合は、(3)から再度入力となる。
    

.. _CreateProjectFromBlank_STS-import-project:

#. Spring Tool Suiteにプロジェクトをインポートする。

    [STS] -> [File] -> [Import] -> [Maven] -> [Exsiting Maven Projects] ->[ Browse...]でMaven archetypeで作成したプロジェクトを指定 -> 1つ表示されるpom.xmlにチェックが入っていることを確認して[Finish]
  
    以下のような状態になる。
  
    .. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlank_import_blank_project.png
       :alt: import blank project
       :width: 100%


#. アプリケーションサーバに作成したプロジェクトを追加する。

    ここではデフォルトでSTSに付属するVMware vFabric tc Server Developer Edition v2.9を使用する。
  
    [VMware vFabric tc Server Developer Edition v2.9]を右クリック -> [Add and Remove] -> 作成したプロジェクトを選択して[Add] -> [Finish]
  
    以下のような状態になる。
  
    .. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlank_add_server_blank_project.png
       :alt: add server blank project
       :width: 100%


#. アプリケーションサーバを起動する。

    Serverのスタートボタンを押して起動する。Consoleに以下が出力されれば起動成功となる。
    
    .. code-block:: console
    
      FrameworkServlet 'appServlet': initialization completed
  
    下記の出力例を見ると、Consoleにログが出力されるが、赤文字の一行上の行に"\ ``FrameworkServlet 'appServlet': initialization completed``\ "が出力される(スクリーンキャプチャ上には表示されていない)。
  
    .. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlank_server_start_blank_project.png
       :alt: server start blank project
       :width: 100%


#. 起動したアプリケーションにアクセスする。

    ブラウザでhttp://localhost:8080/example-first-application/にアクセスする。
  
    以下のような画面が表示される。
  
    .. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlank_access_blank_project.png
       :alt: access blank project
       :width: 50%
  
    「Hellow world!」が表示されれば、プロジェクトの作成準備は完了である。
    ガイドラインに沿って、必要な機能を追加していくこと。


簡単なEchoプロジェクトの作成
--------------------------------------------------------------------------------

\ :doc:`../Overview/FirstApplication`\ で説明した\ :ref:`first-application-create-an-echo-application`\ と基本的には同じ手順となるため、ソースは割愛する。

\ :doc:`../Overview/FirstApplication`\ ではspring-mvc.xmlの中で、\ ``<context:component-scan base-package="com.example.helloworld" />``\ を設定しているが、
がBlank Projectから作成した場合は\ ``<context:component-scan base-package="example.first.application.app" />``\ と設定される。

\ ``EchoController``\ は\ ``example.first.application.app.echo``\ パッケージで作成すること。

.. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlank_echo_input_blank_project.png
   :alt: echo input blank project
   :width: 50%

.. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlank_echo_output_blank_project.png
   :alt: echo output blank project
   :width: 50%

.. todo::

  **TBD**

   今回はシングルプロジェクト構成での作成方法の説明をしているが、これは主に検証目的で使用されることを想定している。
   実際には\ :ref:`マルチプロジェクト構成 <application-layering_project-structure>`\ でプロジェクトを構築する必要がある。
   マルチプロジェクト構成での作成方法は今後追記する。

.. warning::

  Blankプロジェクトのpom.xmlに定義されている、以下の設定用はサンプルアプリケーションを簡単試すためのものであり、実際の開発で使用されることを想定していない。
  実際のプロジェクトでは削除すること。
  
    .. code-block:: xml
    
      <dependency>
          <groupId>com.h2database</groupId>
          <artifactId>h2</artifactId>
          <version>1.3.172</version>
          <scope>compile</scope>
      </dependency>
