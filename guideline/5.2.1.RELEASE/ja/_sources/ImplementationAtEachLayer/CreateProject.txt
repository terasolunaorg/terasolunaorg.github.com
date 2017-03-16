開発プロジェクトのビルド
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

開発プロジェクトのビルド
--------------------------------------------------------------------------------

アプリケーションサーバにデプロイするためのwarファイル、envモジュール(ファイル環境依存ファイルを格納するモジュール)のjarファイルを作成する方法を紹介する。

Maven Archetypeで作成したプロジェクトでは、warファイルを作成する方法として以下の２つの方法を提供している。

* :ref:`CreateWebApplicationProjectBuildWarExcludeEnvJar` (**推奨**)
* :ref:`CreateWebApplicationProjectBuildWarIncludeEnvJar`


.. note:: **推奨するビルド方法について**

    本ガイドラインでは、:ref:`CreateWebApplicationProjectBuildWarExcludeEnvJar` を推奨している。
    なお、ここで紹介するビルド方法は選択肢の一つであり、他のビルド方法を採用してもよい。

    ただし、**試験環境や商用環境にリリースするwarファイルとjarファイルは、EclipseなどのIDEが提供している機能を使って作成しないようにすること。**
    Eclipseなどの一部のIDEでは、開発用に最適化された独自のコンパイラを使ってクラスファイルを作成しており、
    コンパイラの違いが原因でアプリケーション実行時に予期しないエラーが発生するリスクが生まれる。

.. note:: **プロジェクト構成の原則**
    
    原則として下記のようなプロジェクト構造とする。

    1. 必ずマルチプロジェクト構成にする。
    2. 一つのプロジェクトに環境依存性のある設定ファイル（ex. logback.xml, jdbc.properties）をできるだけ集約する。 **以降、このプロジェクトを \*-env と表現する。**
    
      * ex. terasoluna-tourreservation-env

    3. \*-env以外のプロジェクトには環境依存性のある設定値を一切持たせない。

      * もちろん、src/test/resources 配下などにテスト用の環境依存性設定ファイルを格納しておくことは許可される。

    4. 全てのソフトウェアのパッケージ済みバイナリをパッケージリポジトリ上に保管して管理する。

      * \*.jarファイルだけではなく\*.warファイルも成果物としてパッケージリポジトリにデプロイする。したがって、それらのjar/warファイルには環境依存性が含まれていてはならない。

    5. \*-env プロジェクトは下記のような構造にする。

      * 開発者のPC上での作業に必要な設定値をデフォルトとして src/main/resources 配下のファイルに格納する。
      * 試験サーバ、本番サーバ等、環境毎に異なる設定ファイルを src/main/resources 以外(ex. configs/test-server)のフォルダに格納し、mavenのprofile機能を使って環境毎に自動的に設定値を差し替えながら \*-env-x.y.z.jarファイルをビルドする。

    上記のような構造を取ることにより、ソフトウェアライフサイクルの全ての場面において、適切に開発をすることができるようになる。

    #. ローカル開発環境では、プロジェクト本体と\*-envプロジェクトの両方をチェックアウトし、envプロジェクトを本体プロジェクトのビルドパスに含めることによって、ローカル開発環境でのコーディングとテストを可能にする。
    #. CIサーバ上ではビルドツール(maven)によるテストの実行とパッケージングを行い、必要に応じてパッケージリポジトリにartifactをdeployする。
    #. 試験サーバ、本番サーバでは、パッケージリポジトリにあらかじめ保管しているプロジェクト本体に、リリース先環境にあわせてビルドした\*-envプロジェクトを追加してリリースすることにより、アプリケーションの動作が可能になる。

    詳細については\ `サンプルアプリケーション <https://github.com/terasolunaorg/terasoluna-tourreservation>`_\ を参考にされたい。

.. warning:: **ビルド環境について**

    ここではWindows環境でビルドする例になっているが、Windows環境でビルドすることを推奨しているわけではない。
    本ガイドラインでは、**アプリケーションの実行環境と同じOSとJDKのバージョンを使ってビルドすることを推奨する。**

|

| Mavenを使ってビルドする場合は、環境変数「JAVA_HOME」にコンパイル時に使用するJDKのホームディレクトリが指定されていることを確認されたい。
| 環境変数が設定されていない場合や異なるバージョンのJDKのホームディレクトリが指定されている場合は、環境変数に適切なホームディレクトリを指定すること。

**[Windowsの場合]**

.. code-block:: console

    echo %JAVA_HOME%
    set JAVA_HOME={Please set home directory of JDK}


**[Linux系の場合]**

.. code-block:: console

    echo $JAVA_HOME
    JAVA_HOME={Please set home directory of JDK}

.. note::

    環境変数「JAVA_HOME」は、ビルドを実行するOSユーザーのユーザー環境変数に設定しておくとよい。

|

.. _CreateWebApplicationProjectBuildWarExcludeEnvJar:

envモジュールのjarファイルをwarファイルに含めないビルド方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _CreateWebApplicationProjectBuildWarExcludeEnvJarStepWar:

warファイルの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

開発プロジェクトのルートディレクトリへ移動する。

.. code-block:: console

    cd C:\work\todo

|

| Mavenのプロファイル(\ ``-P``\ パラメータ)に\ ``warpack``\ を指定して、Maven installを実行する。

.. code-block:: console

    mvn -P warpack clean install

| Maven packageの実行が成功すると、webモジュールのtargetディレクトリの中に、envモジュールのjarファイルが含まれていないwarファイルが作成される。
| (例：\ ``C:\work\todo\todo-web\target\todo-web.war``\ )

.. note:: **指定するゴールについて**

    上記例ではゴールに\ ``install``\ を指定してwarファイルをローカルリポジトリへインストールしているが、

     * warファイルの作成のみ行う場合はゴールに\ ``package``\
     * Nexusなどのリモートリポジトリへデプロイする場合はゴールに\ ``deploy``\

    を指定すればよい。


|

.. _CreateWebApplicationProjectBuildWarExcludeEnvJarStepEnvJar:

envモジュールのjarファイルの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

envモジュールのディレクトリへ移動する。

.. code-block:: console

    cd C:\work\todo\todo-env

|

Mavenのプロファイル(\ ``-P``\ パラメータ)に\ **環境を識別するプロファイルID**\ を指定して、Maven packageを実行する。

.. code-block:: console

    mvn -P test-server clean package

| Maven packageの実行が成功すると、envモジュールのtargetディレクトリの中に、指定した環境用のjarファイルが作成される。
| (例：\ ``C:\work\todo\todo-env\target\todo-env-1.0.0-SNAPSHOT-test-server.jar``\ )

.. note:: **環境を識別するプロファイルIDについて**

    Maven Archetypeで作成したプロジェクトでは、以下のプロファイルIDがデフォルトで定義されている。

     * \ ``local``\  : 開発者のローカル環境向け(IDE開発環境向け)のプロファイル (デフォルトのプロファイル)
     * \ ``test-server``\  : 試験環境向けのプロファイル
     * \ ``production-server``\  : 商用環境向けのプロファイル

    デフォルトで用意しているプロファイルは上記の3つだが、開発するシステムの環境構成にあわせて追加及び修正されたい。

|

.. _CreateWebApplicationProjectBuildWarIncludeEnvJar:

envモジュールのjarファイルをwarファイルに含めるビルド方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _CreateWebApplicationProjectBuildWarIncludeEnvJarWar:

warファイルの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. warning:: **envモジュールのjarファイルをwarファイルに含める場合の注意点**

    envモジュールのjarファイルをwarファイルに含めた場合、warファイルを他の環境にデプロイすることができないため、
    間違って他の環境(特に商用環境)にデプロイされないようにwarファイルを管理すること。

    また、環境毎にwarファイルを作成して各環境へリリースする方法を採用した場合、
    商用環境へリリースされるwarファイルが厳密にいうとテスト済みのwarファイルではないという点を意識してほしい。
    これは、商用環境用のwarファイルを作成する際にコンパイルをしなおすためである。
    warファイルを環境毎に作成してリリースする場合は、GitやSubversionなどのVCS(Version Control System)の機能(タグ機能など)を活用し、
    テスト済みのソースファイルを使用して商用環境や各種テスト環境へリリースするwarファイルを作成する仕組みを確立することが特に重要である。

|

開発プロジェクトのルートディレクトリへ移動する。

.. code-block:: console

    cd C:\work\todo

|

| Mavenのプロファイル(\ ``-P``\ パラメータ)に\ ``warpack-with-env``\ とenvモジュールの中で定義している\ **環境を識別するプロファイルID**\ を指定して、Maven packageを実行する。

.. code-block:: console

    mvn -P warpack-with-env,test-server clean package

| Maven packageの実行が成功すると、webモジュールのtargetディレクトリの中に、envモジュールのjarファイルを含んだwarファイルが作成される。
| (例：\ ``C:\work\todo\todo-web\target\todo-web.war``\ )

|


.. _CreateWebApplicationProjectBuildDeploy:

デプロイ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _CreateWebApplicationProjectBuildDeployToTomcat:

Tomcatへのデプロイ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

WebアプリケーションをTomcat上にリリースする場合は次のような手順をとる。 

1. リリース対象のAPサーバ環境にあわせてmavenのprofileを指定し、 \*-env プロジェクトを ビルドする。
2. 上記でビルドした\*-env-x.y.z.jarファイル をあらかじめ決定したAPサーバ上のフォルダに設置する。 ex. /etc/foo/bar/abcd-env-x.y.z.jar
3. あらかじめパッケージリポジトリにデプロイ済みの\*.warファイルを [CATALINA_HOME]/webapps 配下で解凍(unjar)する。
4. Tomcat 7を使用する場合は、TomcatのVirtualWebappLoader機能を使用して /etc/foo/bar/\*.jar をクラスパスに追加する。

 * [CATALINA_HOME]/conf/[contextPath].xml ファイルに下記の定義を追加する。
 * 詳しくは、 http://tomcat.apache.org/tomcat-7.0-doc/api/org/apache/catalina/loader/VirtualWebappLoader.html と `terasoluna-tourreservation-envのconfigsフォルダ <https://github.com/terasolunaorg/terasoluna-tourreservation/tree/5.2.0.RELEASE/terasoluna-tourreservation-env/configs>`_\ を参考されたい。
 * VirtualWebappLoaderの設定例：
   
  .. code-block:: xml

   <Loader className="org.apache.catalina.loader.VirtualWebappLoader"
           virtualClasspath="/etc/foo/bar/*.jar" />
	 
 * なお、VirtualWebappLoader機能はTomcat 6でも使用可能。

5. Tomcat 8を使用する場合は、Tomcatのリソース機能を使用して /etc/foo/bar/\*.jar をクラスパスに追加する。

 * [CATALINA_HOME]/conf/[contextPath].xml ファイルに下記の定義を追加する。
 * 詳しくは、 https://tomcat.apache.org/migration-8.html#Web_application_resources と `terasoluna-tourreservation-envのconfigsフォルダ <https://github.com/terasolunaorg/terasoluna-tourreservation/tree/5.2.0.RELEASE/terasoluna-tourreservation-env/configs>`_\ を参考されたい。
 * リソースの設定例：
   
  .. code-block:: xml

   <Resources className="org.apache.catalina.webresources.StandardRoot">
     <PreResources className="org.apache.catalina.webresources.DirResourceSet"
                   base="/etc/foo/bar/"
                   internalPath="/"
                   webAppMount="/WEB-INF/lib" />
   </Resources>

.. note::

 * [CATALINA_HOME]/conf/server.xml の Host タグ上の autoDeploy 属性を false にセットしておかなければならない。さもないとwebアプリケーションの再起動のたびに[CATALINA_HOME]/conf/[contextPath].xmlが自動的に削除されてしまう。
 * autoDeployを無効化している場合、[CATALINA_HOME]/webappsにwarファイルを置くだけではWebアプリケーションは起動しない。必ずwarファイルをunjar(unzip)すること。

|

.. _CreateWebApplicationProjectBuildDeployToOtherServer:

Tomcat以外のアプリケーションサーバへのデプロイ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

アプリケーションサーバとしてTomcat以外のサーバを使用する際のデプロイ方法(手順)を紹介する。

TomcatのVirtualWebappLoaderのように、Webアプリケーションごとにクラスパスを追加する手段が提供されていない
アプリケーションサーバ（例： WebSphere,WebLogic,JBoss）にリリースする場合には、
\*-env-x.y.z.jarファイルをwarファイル内の WEB-INF/lib 配下に追加してからリリースする方法が最も簡単である。

1. リリース対象のAPサーバ環境にあわせてmavenのprofileを指定し、 \*-env プロジェクトを ビルドする。
2. あらかじめパッケージリポジトリにデプロイ済みの\*.warファイルを 作業ディレクトリにコピーする。
3. 下のように、ｊａｒコマンドの追加オプションを利用して、warファイル内の WEB-INF/lib の配下に追加する。
4. foo-x.y.z.warをAPサーバにリリースする。

.. note::

    warファイルをアプリケーションサーバへデプロイする方法は、使用するアプリケーションサーバのマニュアルを参照されたい。

|

ここでは、jarコマンドを使用して、envモジュールのjarファイルをwarファイルに組み込む方法(手順)を紹介する。

| 作業ディレクトリへ移動する。
| ここでは、envプロジェクトで作業を行う例になっている。

.. code-block:: console

    cd C:\work\todo\todo-env

|

| 作成したwarファイルを作業ディレクトリへコピーする。
| ここでは、Mavenリポジトリからwarファイルを取得する例になっている。(warファイルを\ ``install``\ または\ ``deploy``\ している前提とする)

.. code-block:: console

    mvn org.apache.maven.plugins:maven-dependency-plugin:2.5:get^
     -DgroupId=com.example.todo^
     -DartifactId=todo-web^
     -Dversion=1.0.0-SNAPSHOT^
     -Dpackaging=war^
     -Ddest=target/todo-web.war

| コマンドの実行が成功すると、envモジュールのtargetディレクトリの中に、指定したwarファイルがコピーされる。
| (例：\ ``C:\work\todo\todo-env\target\todo-web.war``\ )

.. note::

    * \ ``-DgroupId``\ 、\ ``-DartifactId``\ 、\ ``-Dversion``\ 、\ ``-Ddest``\ には、適切な値を指定すること。
    * Linux系で実行する場合は、行末の \ ``^``\  を \ ``\``\  に読み替えること。

|

作成したjarファイルを作業ディレクト(\ ``target\WEB-INF\lib``\ )へ一旦コピーし、warファイルの中に追加する。

**[Windowsの場合]**

.. code-block:: console

    mkdir target\WEB-INF\lib
    copy target\todo-env-1.0.0-SNAPSHOT-test-server.jar target\WEB-INF\lib\.
    cd target
    jar -uvf todo-web.war WEB-INF\lib

**[Linux系の場合]**

.. code-block:: console

    mkdir -p target/WEB-INF/lib
    cp target/todo-env-1.0.0-SNAPSHOT-test-server.jar target/WEB-INF/lib/.
    cd target
    jar -uvf todo-web.war WEB-INF/lib

.. note:: **jarコマンドが見つからない場合の対処**

    jarコマンドが見つからない場合は、以下のいずれかの対処を行うことで解決することができる。

    * \ ``JAVA_HOME/bin``\ を環境変数「PATH」に追加する。
    * jarコマンドをフルパスで指定する。Windowの場合は\ ``%JAVA_HOME%\bin\jar``\ 、Linux系の場合は\ ``${JAVA_HOME}/bin/jar``\ を指定すればよい。


.. _CreateWebApplicationProjectBuildDeployContinuedDeployment:

継続的なデプロイ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

プロジェクト（ソースコードツリー）の構造、バージョン管理、インスペクションとビルド作業、ライフサイクル管理の工程を恒常的にループさせることによって目的のソフトウェアをリリースし続けることが、継続的デプロイメントである。

開発の途中では、SNAPSHOTバージョンのソフトウェアをパッケージリポジトリや開発用APサーバにリリースし、テストを実施する。
ソフトウェアを正式にリリースする場合には、バージョン番号を固定したうえでVCS上でソースコードツリーに対してタグづけを行う必要がある。
このように、スナップショットリリースの場合と正式リリースの場合で、ビルドとデプロイのフローが少し異なる。

また、Webサービスを提供するAPサーバにアプリケーションをデプロイする場合には、スナップショットバージョンか正式リリースバージョンかに関わらず、
デプロイ先のAPサーバ環境に合わせた環境依存性設定ファイル群と*.warファイルをセットでデプロイする必要がある。

そこで、環境依存性設定を持たない状態のライブラリ(jar,war)をmavenリポジトリに登録する作業と、
それらを実際にAPサーバにデプロイする作業を分離することによって、デプロイ作業を簡潔に実施可能にする。

.. note::

 mavenの世界では、pom.xml上の<version>タグの内容によってそれがSNAPSHOTバージョンなのかRELEASEバージョンなのかが自動的に判別される。

 * 末尾が -SNAPSHOT である場合にSNAPSHOTとみなされる。例：<version>1.0-SNAPSHOT</version>
 * 末尾が -SNAPSHOT ではない場合はRELEASEとみなされる。例：<version>1.0</version>

 また、mavenパッケージリポジトリにはsnapshotsリポジトリとreleaseリポジトリの2種類があり、いくつかの制約があることに注意する。

 * SNAPSHOTバージョンのソフトウェアをreleaseリポジトリに登録することはできない。その逆も不可能。
 * releaseリポジトリには、同一のGAV情報を持つartifactは1回しか登録できない。（GAV=groupId,artifactId,version）
 * snapshotリポジトリには、同一のGAV情報を持つartifactを何度でも登録しなおすことができる。

SNAPSHOTバージョンの運用
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

SNAPSHOTバージョンのソフトウェアのデリバリーフローは下図のように簡潔である。

.. figure:: ./images/ContinuousDelivery-snapshot.png
   :alt: Continuous delivery for SNAPSHOT version.
   :width: 600px

1. 開発用trunkからソースコードをチェックアウトする。
2. コンパイル、コードメトリクスの測定、テストを実行する。

 * コンパイルエラー、コードメトリクスでの一定以上のviolationの発生、テストの失敗の場合、以降の作業を中止する。

3. mavenパッケージリポジトリサーバにartifact(jar,warファイル)をアップロード(mvn deploy)する。
|

RELEASEバージョンの運用
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

正式なリリースの場合、バージョン番号の付与作業が必要なため、SNAPSHOTリリースよりもやや複雑なフローとなる。

.. figure:: ./images/ContinuousDelivery-release.png
   :alt: Continuous delivery for RELEASE version.
   :width: 600px

1. リリースに与えるバージョン番号を決定する。（例：1.0.1）
2. 開発用trunk（またはリリース用branch)からソースコードをチェックアウトする。
3. pom.xml上の<version>タグを変更する。（例：<version>1.0.1</version>）
4. VCS上にtagを付与する。（例： tags/1.0.1）
5. コンパイル、コードメトリクスの測定、テストを実行する。

 * コンパイルエラー、コードメトリクスでの一定以上のviolationの発生、テストの失敗の場合、以降の作業を中止する。
 * 失敗した場合はVCS上のtagを削除する。

6. mavenパッケージリポジトリサーバにartifact(jar,warファイル)をアップロード(mvn deploy)する。

.. todo:: 
 
 ここで最後にtrunkのソースツリーのpom.xmlのversionタグを、
 次のSNAPSHOTバージョンに書き変えてコミットするところまで書くべきか？！

.. note::

 pom.xmlファイルの<version>タグの変更は `versions-maven-plugin <http://www.mojohaus.org/versions-maven-plugin/>`_ で可能である。
 
 .. code-block:: bash
 
  mvn versions:set -DnewVersion=1.0.0
 
 上記のようなコマンドで、pom.xml内のversionタグを<version>1.0.0</version>のように編集することができる。

アプリケーションサーバへのリリース
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

Webサービスを提供するAPサーバにアプリケーションをリリースする場合、
あらかじめmavenパッケージリポジトリに登録済みのwarファイルと、
リリース先のAPサーバ環境に合わせた環境依存性設定ファイル群とを、セットでリリースする。
これはスナップショットリリースか正式リリースかに関わらず同じフローとなる。

.. figure:: ./images/ContinuousDelivery-apserver.png
   :alt: Continuous delivery for webapp to application server.
   :width: 600px

1. リリース対象バージョンのwarファイルをmavenパッケージリポジトリからダウンロードする
2. \*-resourcesプロジェクト（環境依存性設定ファイルを集約しているプロジェクト）をVCSからチェックアウトする
3. mavenのprofileを機能によって、リリース先の環境に合わせた設定ファイル群で内容を差し替えてresourcesプロジェクトをパッケージし、\*-resources-x.y.z.jarを生成する。
4. 生成した\*-resources-x.y.z.jarファイルを、warファイル内のWEB-INF/libフォルダ配下に追加する。

 * Tomcatの場合は、\*-resources-x.y.z.jarをwarファイル内部に追加するのではなく、Tomcatサーバ上の任意のパスにコピーし、そのパスをVirtualWebappLoaderの拡張クラスパスに指定する。

5. warファイルをアプリケーションサーバにデプロイする。

.. note::

 mavenパッケージリポジトリからのwarファイルのダウンロードは、maven-dependency-pluginのgetゴールで可能である。

 .. code-block:: bash

  mvn org.apache.maven.plugins:maven-dependency-plugin:2.5:get \
   -DgroupId=com.example \
   -DartifactId=mywebapp \
   -Dversion=0.0.1-SNAPSHOT \
   -Dpackaging=war \
   -Ddest=${WORKSPACE}/target/mywebapp.war

 これで、targetというディレクトリ配下にmywebapp.warファイルがダウンロードされる。
 
 さらに、下記のようなコマンドで環境依存設定ファイルのパッケージをmywebapp.warファイル内に追加することができる。

 .. code-block:: bash

  mkdir -p $WORKSPACE/target/WEB-INF/lib
  cd $WORKSPACE/target
  cp ./mywebapp-resources*.jar WEB-INF/lib
  jar -ufv mywebapp.war WEB-INF/lib



.. _CreateProjectCustomize:

.. raw:: latex

   \newpage
