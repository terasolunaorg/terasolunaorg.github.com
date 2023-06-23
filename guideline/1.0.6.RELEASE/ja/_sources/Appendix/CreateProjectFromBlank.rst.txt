Blankプロジェクトから新規プロジェクト作成
================================================================================

Blankプロジェクトからの新規プロジェクトを作成する方法を説明する。

* 前提

  * Spring Tool Suite が動作可能なこと。
  * Mavenをコマンドラインで実行可能なこと。
  * インターネットに接続できること。

この節の内容は以下のバージョンで動作確認している。

.. tabularcolumns:: |p{0.75\linewidth}|p{0.25\linewidth}|
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

#. プロジェクトを作成するフォルダにマンドプロンプトで移動する。
  
    .. code-block:: text
    
        cd C:\work

#. Maven archetypeを利用してプロジェクトの雛形を生成する

.. note:: **Maven Archetype Pluginのバージョンについて**

    Maven Archetype Plugin 3系では\ ``archetype:generate``\でMaven Central以外のリモートリポジトリを指定する場合はsettings.xmlでの設定が必須になった。

    そのため、本ガイドラインでは既存ユーザが今まで通りの使い方ができるようにMaven Archetype Plugin 2.4を明示的に使用している。

    .. code-block:: console
    
      mvn org.apache.maven.plugins:maven-archetype-plugin:2.4:generate -B^
       -DarchetypeCatalog=http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases^
       -DarchetypeGroupId=org.terasoluna.gfw.blank^
       -DarchetypeArtifactId=terasoluna-gfw-web-blank-archetype^
       -DarchetypeVersion=1.0.6.RELEASE^
       -DgroupId=example.first.application^
       -DartifactId=example-first-application^
       -Dversion=1.0.0-SNAPSHOT


    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 20 80
    
       * - パラメータ
         - 説明
       * - | \-B
         - | batch mode (対話を省略)
       * - | \-DarchetypeCatalog
         - | Maven archetypeでプロジェクトを作成する。TERASOLUNA Global Frameworkのレポジトリを指定する。
       * - | \-DarchetypeGroupId
         - | tarasoluna.gfwのブランクプロジェクトのGroupID = org.terasoluna.gfw.blank 固定
       * - | \-DarchetypeArtifactId
         - | terasoluna-gfw-web-blank-archetype = JPA,MyBatis2は使用せずにプロジェクトを作成する場合
           | terasoluna-gfw-web-blank-jpa-archetype = JPAを使用するプロジェクトを作成する場合
           | terasoluna-gfw-web-blank-mybatis2-archetype = MyBatis2を使用するプロジェクトを作成する場合
       * - | \-DarchetypeVersion
         - | terasoluna.gfwのバージョンを指定する。
       * - | \-DgroupId
         - | 作成プロジェクトのgroupIdを指定する。
       * - | \-DartifactId
         - | 作成プロジェクトのartifactIdを指定する。
       * - | \-Dversion
         - | 作成プロジェクトのバージョンを指定する。
    

.. _CreateProjectFromBlank_STS-import-project:

3. Spring Tool Suiteにプロジェクトをインポートする。

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
          <version>${com.h2database.version}</version>
          <scope>runtime</scope>
      </dependency>

.. raw:: latex

   \newpage

