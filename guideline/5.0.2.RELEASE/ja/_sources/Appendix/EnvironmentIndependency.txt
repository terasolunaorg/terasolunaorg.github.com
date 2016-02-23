環境依存性の排除
================================================================================


.. todo::

  書き直し。


Webアプリケーションの開発プロジェクトでは、必ず環境依存性の問題が発生する。

もしも、datasource.xml ファイルに jdbcurl=hdbc:mysql:127.0.0.1...
と書いて、あるいはもしも、logback.xml ファイルに level="DEBUG" と書いて、
それらすべてがwarファイルに同梱されてしまったら、
そのWebアプリケーションは、あなたのローカルPCでしか正常に作動せず、試験用サーバにはリリースできない。

過去、意外と多くの開発プロジェクトが、この単純な問題を軽視していた。
そして結合テストの直前になってから、開発したWebアプリケーションを試験用サーバで作動させることが難しいことに気づき、
問題の解決のために膨大な時間を費やすことになった。

この章では、環境依存性の問題を解決するための原則と具体的な方法を解説する。

目的
--------------------------------------------------------------------------------

あなたのチームがこれから開発する全てのソースコード、あるいはそのバイナリは、
以下のすべてのシチュエーションでシームレスに動作可能でなければならない。

* 全ての開発者のPCのIDE(eclipse)上で設定されたAPサーバ上でのアプリケーションの実行
* 全ての開発者のPCのIDE上のJUnitプラグインによるテストの実行
* 全ての開発者のPC上のビルドツール（maven/ant）によるテストの実行
* CIサーバ上でのテストの実行
* CIサーバ上でのパッケージング（jar/warファイルの生成）
* 試験サーバ上でのアプリケーションの実行
* 本番サーバ上でのアプリケーションの実行

原則
--------------------------------------------------------------------------------

前述の目的を実現するために、原則として下記のようなプロジェクト構造とする。

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

デプロイ
--------------------------------------------------------------------------------

.. _EnvironmentIndependencyDeployTomcat:

Tomcatへのデプロイ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

WebアプリケーションをTomcat上にリリースする場合は次のような手順をとる。 

1. リリース対象のAPサーバ環境にあわせてmavenのprofileを指定し、 \*-env プロジェクトを ビルドする。
2. 上記でビルドした\*-env-x.y.z.jarファイル をあらかじめ決定したAPサーバ上のフォルダに設置する。 ex. /etc/foo/bar/abcd-env-x.y.z.jar
3. あらかじめパッケージリポジトリにデプロイ済みの\*.warファイルを [CATALINA_HOME]/webapps 配下で解凍(unjar)する。
4. Tomcat 7を使用する場合は、TomcatのVirtualWebappLoader機能を使用して /etc/foo/bar/\*.jar をクラスパスに追加する。

 * [CATALINA_HOME]/conf/[contextPath].xml ファイルに下記の定義を追加する。
 * 詳しくは、 http://tomcat.apache.org/tomcat-7.0-doc/api/org/apache/catalina/loader/VirtualWebappLoader.html と `terasoluna-tourreservation-envのconfigsフォルダ <https://github.com/terasolunaorg/terasoluna-tourreservation/tree/master/terasoluna-tourreservation-env/configs>`_\ を参考されたい。
 * VirtualWebappLoaderの設定例：
   
  .. code-block:: xml

   <Loader className="org.apache.catalina.loader.VirtualWebappLoader"
           virtualClasspath="/etc/foo/bar/*.jar" />
	 
 * なお、VirtualWebappLoader機能はTomcat 6でも使用可能。

5. Tomcat 8を使用する場合は、Tomcatのリソース機能を使用して /etc/foo/bar/\*.jar をクラスパスに追加する。

 * [CATALINA_HOME]/conf/[contextPath].xml ファイルに下記の定義を追加する。
 * 詳しくは、 https://tomcat.apache.org/migration-8.html#Web_application_resources と `terasoluna-tourreservation-envのconfigsフォルダ <https://github.com/terasolunaorg/terasoluna-tourreservation/tree/master/terasoluna-tourreservation-env/configs>`_\ を参考されたい。
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

.. _EnvironmentIndependencyDeployOther:

他のアプリケーションサーバーへのデプロイ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

TomcatのVirtualWebappLoaderのように、Webアプリケーションごとにクラスパスを追加する手段が提供されていない
アプリケーションサーバ（例： WebSphere,WebLogic,JBoss）にリリースする場合には、
\*-env-x.y.z.jarファイルをwarファイル内の WEB-INF/lib 配下に追加してからリリースする方法が最も簡単である。

1. リリース対象のAPサーバ環境にあわせてmavenのprofileを指定し、 \*-env プロジェクトを ビルドする。
2. あらかじめパッケージリポジトリにデプロイ済みの\*.warファイルを 作業ディレクトリにコピーする。
3. 下のように、ｊａｒコマンドの追加オプションを利用して、warファイル内の WEB-INF/lib の配下に追加する。
4. foo-x.y.z.warをAPサーバにリリースする。

.. _EnvironmentIndependencyContinuousDeploy:

継続的なデプロイ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

SNAPSHOTバージョンのソフトウェアのデリバリーフローは下図のように簡潔である。

.. figure:: ./images/ContinuousDelivery-snapshot.png
   :alt: Continuous delivery for SNAPSHOT version.
   :width: 600px

1. 開発用trunkからソースコードをチェックアウトする。
2. コンパイル、コードメトリクスの測定、テストを実行する。

 * コンパイルエラー、コードメトリクスでの一定以上のviolationの発生、テストの失敗の場合、以降の作業を中止する。

3. mavenパッケージリポジトリサーバにartifact(jar,warファイル)をアップロード(mvn deploy)する。

.. todo:: 後でキャプチャをはる


RELEASEバージョンの運用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

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

 pom.xmlファイルの<version>タグの変更は `versions-maven-plugin <http://mojo.codehaus.org/versions-maven-plugin/>`_ で可能である。
 
 .. code-block:: bash
 
  mvn versions:set -DnewVersion=1.0.0
 
 上記のようなコマンドで、pom.xml内のversionタグを<version>1.0.0</version>のように編集することができる。

.. todo:: 後でキャプチャをはる


アプリケーションサーバへのリリース
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

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

 * Tomcatの場合は、\*-resources-x.y.z.jarをwarファイル内部に追加するのではなく、Tomcatサーバ上の任意のパスにコピーし、そのパスをVirtualWebappLoaderの拡張クラスパスに指定する。詳細は :doc:`EnvironmentIndependency` を参照。

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
 
.. todo:: あとでキャプチャをはる

.. raw:: latex

   \newpage

