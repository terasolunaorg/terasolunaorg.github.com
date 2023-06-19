ブランクプロジェクトから新規プロジェクトの作成
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

本節では、ブランクプロジェクトから新規プロジェクトを作成する方法を説明する。

.. _CreateProjectFromBlankPrerequisite:

前提
--------------------------------------------------------------------------------

本節で説明する内容は、以下の条件が整っていることを前提としている。
前提条件が整っていない場合は、まずこれらのセットアップを行ってほしい。

* Spring Tool Suite が動作可能なこと。
* Mavenをコマンドラインで実行可能なこと。
* インターネットに接続できること。

.. _CreateProjectFromBlank_create-new-project:

.. note::

    インターネット接続するために、プロキシサーバーを介する必要がある場合、
    STSのProxy設定と\ `MavenのProxy設定 <http://maven.apache.org/guides/mini/guide-proxies.html>`_\ が必要である。

|

.. _CreateProjectFromBlankVerificationEnvironment:

検証環境
--------------------------------------------------------------------------------

本節で説明する内容は、以下のバージョンで動作を確認している。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - 種別
      - 名前
    * - OS
      - Windows 7
    * - JVM
      - `Java <http://www.oracle.com/technetwork/java/javase/downloads/index.html>`_ 1.7
    * - IDE
      - `Spring Tool Suite <http://spring.io/tools/sts/all>`_ 3.6.3.RELEASE (以降「STS」と呼ぶ)
    * - Build Tool
      - `Apache Maven <http://maven.apache.org/download.cgi>`_ 3.2.5 (以降「Maven」と呼ぶ)
    * - Application Server
      - `Pivotal tc Server <https://network.pivotal.io/products/pivotal-tcserver>`_ Developer Edition v3.0 (STSに同封)

|

.. _CreateProjectFromBlankTypes:

ブランクプロジェクトの種類
--------------------------------------------------------------------------------

ブランクプロジェクトは、使用用途に応じて以下の２種類を提供している。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 70

    * - 種別
      - 使用用途
    * - | `マルチプロジェクト構成のブランクプロジェクト <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank>`_
      - 商用環境にリリースするような本格的なアプリケーションを開発する際に使用する。

        プロジェクトの雛形は、MavenのArchetypeとして、以下の3種類を用意している。

        * MyBatis3用の設定が盛り込まれた雛形
        * JPA(Spring Data JPA)用の設定が盛り込まれた雛形

        **本ガイドラインでは、マルチプロジェクト構成のプロジェクトを使用する事を推奨している。**
    * - | `シングルプロジェクト構成のブランクプロジェクト <https://github.com/terasolunaorg/terasoluna-gfw-web-blank>`_
      - POC(Proof Of Concept)、プロトタイプ、サンプルなどの簡易的なアプリケーションを作成する際に使用する。

        プロジェクトの雛形は、MavenのArchetypeとして、以下の4種類を用意している。
        (EclipseのWTP用のプロジェクトも用意しているが、本節では説明は割愛する)

        * MyBatis3用の設定が盛り込まれた雛形
        * JPA(Spring Data JPA)用の設定が盛り込まれた雛形
        * O/R Mapperに依存しない雛形

        本ガイドラインでは、各種チュートリアルをシングルプロジェクトを使用して行う手順となっている。

|

.. _CreateProjectFromBlankGenerateMultipleProject:

マルチプロジェクト構成のプロジェクト作成
--------------------------------------------------------------------------------

マルチプロジェクト構成のプロジェクトを作成する方法については、
「:doc:`../ImplementationAtEachLayer/CreateWebApplicationProject`」を参照されたい。

上記のドキュメントには、以下のコンテンツが含まれている。

* マルチプロジェクト構成のプロジェクトを作成する方法
* warファイルのビルド方法
* ブランクプロジェクトからのカスタマイズ箇所の説明
* プロジェクト構成の説明


|

.. _CreateProjectFromBlankGenerateSingleProject:

シングルプロジェクト構成のプロジェクト作成
--------------------------------------------------------------------------------

シングルプロジェクト構成のプロジェクトを作成する方法について説明する。

まず、プロジェクトを作成するフォルダに移動する。
  
.. code-block:: console
    
    cd C:\work

|

`Maven Archetype Plugin <http://maven.apache.org/archetype/maven-archetype-plugin/>`_ の `archetype:generate <http://maven.apache.org/archetype/maven-archetype-plugin/generate-mojo.html>`_ を使用して、プロジェクトを作成する。

.. code-block:: console

    mvn archetype:generate -B^
     -DarchetypeCatalog=http://repo.terasoluna.org/nexus/content/repositories/terasoluna-gfw-releases^
     -DarchetypeGroupId=org.terasoluna.gfw.blank^
     -DarchetypeArtifactId=terasoluna-gfw-web-blank-mybatis3-archetype^
     -DarchetypeVersion=5.0.1.RELEASE^
     -DgroupId=todo^
     -DartifactId=todo^
     -Dversion=1.0.0-SNAPSHOT

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 25 75
    
    * - パラメータ
      - 説明
    * - \-B
      - | batch mode (対話を省略)
    * - | \-DarchetypeCatalog
      - TERASOLUNA Server Framework for Java (5.x)のレポジトリを指定する。(固定)
    * - | \-DarchetypeGroupId
      - ブランクプロジェクトのgroupIdを指定する。(固定)
    * - | \-DarchetypeArtifactId
      - ブランクプロジェクトのarchetypeId(雛形を特定するためのID)を指定する。**(カスタマイズが必要)**

        以下の何れかのarchetypeIdを指定する。

        * | ``terasoluna-gfw-web-blank-mybatis3-archetype``
          | MyBatis3用の設定が盛り込まれた雛形
        * | ``terasoluna-gfw-web-blank-jpa-archetype``
          | JPA(Spring Data JPA)用の設定が盛り込まれた雛形
        * | ``terasoluna-gfw-web-blank-archetype``
          | O/R Mapperに依存しない雛形

        上記例では、\ ``terasoluna-gfw-web-blank-mybatis3-archetype``\ を指定している。
    * - | \-DarchetypeVersion
      - ブランクプロジェクトのバージョンを指定する。(固定)
    * - | \-DgroupId
      - 作成するプロジェクトのgroupIdを指定する。**(カスタマイズが必要)**

        上記例では、\ ``"todo"``\ を指定している。
    * - | \-DartifactId
      - 作成するプロジェクトのartifactIdを指定する。**(カスタマイズが必要)**

        上記例では、\ ``"todo"``\ を指定している。
    * - | \-Dversion
      - 作成するプロジェクトのバージョンを指定する。**(カスタマイズが必要)**

        上記例では、\ ``"1.0.0-SNAPSHOT"``\ を指定している。

.. warning::

    ブランクプロジェクトの\ ``pom.xml``\ には、インメモリデータベース(H2 Database)への依存関係が指定されている。
    これはちょっとした動作検証(プロトタイプ作成やPOC(Proof Of Concept))を行うための設定であり、実際の開発で使用することは想定していない。

     .. code-block:: xml

        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>

    **H2 Databaseを使用しない場合は、この設定は削除すること。**

|

.. _CreateProjectFromBlank_STS-import-project:

IDE(STS)へのプロジェクトのインポート
--------------------------------------------------------------------------------

作成したプロジェクトをSTSへインポートする方法について説明する。

.. note::

    ここでは、シングルプロジェクトをインポートする例になっているが、マルチプロジェクトも同じ手順でインポート可能である。

|

STSのメニューから、[File] -> [Import] -> [Maven] -> [Existing Maven Projects] -> [Next]を選択し、archetypeで作成したプロジェクトを選択するダイアログを開く。

.. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlankImportSelect.png
    :alt: Open the dialog to import project
    :width: 80%

|

Root Directoryに \ ``C:\work\todo``\ を設定し、Projectsにtodoのpom.xmlが選択された状態で、 [Finish] を押下する。

.. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlankImportProject.png
    :alt: Import project
    :width: 80%

|

インポートが完了すると、Package Explorerに次のようなプロジェクトが表示される。

.. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlankPackageExplorerAfterImport.png
    :alt: workspace

.. _CreateProjectFromBlank_STS-import-project-update-project:

.. note::

    インポート後にビルドエラーが発生する場合は、プロジェクト名を右クリックし、「Maven」->「Update Project...」をクリックし、
    「OK」ボタンをクリックすることでエラーが解消されるケースがある。

     .. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlankUpdateProject.png
        :width: 70%

.. tip::

    パッケージの表示形式は、デフォルトは「Flat」だが、「Hierarchical」にしたほうが見通しがよい。

    Package Explorerの「View Menu」 (右端の下矢印)をクリックし、「Package Presentation」->「Hierarchical」を選択する。

     .. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlankPresentationHierarchical.png
        :width: 80%

    Package PresentationをHierarchicalにすると、以下の様な表示になる。

     .. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlankPresentationHierarchicalView.png

|

.. _CreateProjectFromBlankDeployAndStartup:

デプロイとアプリケーションサーバ(ts Server)の起動
--------------------------------------------------------------------------------

STS上のアプリケーションサーバにプロジェクトをデプロイし、起動する方法について説明する。

.. note::

    マルチプロジェクトの場合は、アプリケーション層(Web層)のコンポーネントを管理するプロジェクト(archetypeId-web)がデプロイ対象となる。

|

インポートしたプロジェクトを右クリックして「Run As」->「Run on Server」を選択する。

.. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlankRunOnServer.jpg
    :width: 80%

|

APサーバー(Pivotal tc Server Developer Edition v3.0)を選択し、「Next」をクリックする。

.. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlankTcServerNext.jpg
    :width: 80%

|

選択したプロジェクトが「Configured」に含まれていることを確認し、「Finish」をクリックしてサーバーを起動する。

.. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlankTcServerFinish.jpg
    :width: 80%

.. note::

    アプリケーションサーバの起動時にエラーが発生する場合は、以下に示すクリーン作業を行うと解決されるケースがある。

    * | プロジェクトのクリーン
      | STSのメニューから、[Project] -> [Clean...] を選択し、Cleanダイアログで対象を選択して「OK」ボタンを押下する。
    * | Mavenの\ :ref:`Update Project <CreateProjectFromBlank_STS-import-project-update-project>`\
    * | デプロイ済みリソースのクリーン
      | 「Servers」ビューの「tc Server」を右クリック -> [Clean...]
    * | アプリケーションサーバ(tc Server)のワークディレクトリのクリーン
      | 「Servers」ビューの「tc Server」を右クリック -> [Clean tc Server Work Directory...]

|

ブラウザで http://localhost:8080/todo にアクセスすると、以下のような画面が表示される。

.. figure:: ./images_CreateProjectFromBlank/CreateProjectFromBlankTopPage.png
    :width: 80%


.. raw:: latex

   \newpage

