アプリケーションのレイヤ化
********************************************************************************

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

本ガイドラインでは、アプリケーションを、次の3レイヤで分割する。

* アプリケーション層
* ドメイン層
* インフラストラクチャ層

各層には、以下のコンポーネントが含まれる。

.. figure:: images/ApplicationLayer.png
   :alt: application layers
   :width: 80%



| アプリケーション層も、インフラストラクチャ層も、ドメイン層に依存するが、\ **ドメイン層が、他の層に依存してはいけない。**
| ドメイン層の変更によって、アプリケーション層に変更が生じるのは良いが、
| アプリケーション層の変更によって、ドメイン層の変更が生じるべきではない。

各層について、説明する。

.. note::

  アプリケーション層、ドメイン層、インフラストラクチャー層は
  Eric Evansの"Domain-Driven Design (2004, Addison-Wesley)"で説明されてる用語である。
  ただし、用語は使用しているが以後Domain Driven Designの考えにのっとっているわけではない。


レイヤの定義
================================================================================

入力から出力までのデータの流れは、アプリケーション層→ドメイン層→インフラストラクチャ層であるため、
この順に説明する。

アプリケーション層
--------------------------------------------------------------------------------

| 情報の入出力となるUIを提供したり、リクエスト情報をドメイン層や、他システムから呼び出し、表示用の出力を返す手続きを行うなど、
| アプリケーションを構築するための層である。\ **この層は、できるだけ薄く保たれるべきであり、ビジネスルールを含んではいけない。**

Controller
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| 基本的には、リクエストを処理にマッピングし、結果をViewに渡すという画面遷移と、セッション管理を担う。
| 主処理は、Controller内では行わず、ドメイン層のServiceを呼び出す。

Spring MVCでは、\ ``@Controller``\ アノテーションがついた、POJOクラスが該当する。
Controllerの結果がView(の論理名)になる。


View
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| クライアントへの出力を担う。JSP/PDF/Excel/JSONなど、様々な出力結果を返す。
| Spring MVCでは、\ ``View``\ クラスが該当する。

Form
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| 画面のフォームを表現する。フォームの情報をControllerに渡したり、Contollerからフォームに出力する際に用いられる。
| ドメイン層がアプリケーション層に依存しないように、FormからDomain Object(Entity等)への変換や、
| Domain ObjectからFormへの変換は、アプリケーション層で行う必要ある。

.. note::
 
  変換処理を実装する際、Controller内で行うと、ソースコードが長くなり、
  本来のControllerの処理(画面遷移など)の見通しが、悪くなりがちである。
  その場合は、Helperクラスを作成し、変換処理を委譲することを推奨する。

Spring MVCでは、Formオブジェクトは、リクエストパラメータを保持するPOJOクラスが該当する。form backing beanと呼ばれる。


Helper
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| Controllerを補助する役割を担い、Application層とDomain層のモデル相互変換など、Controller本来の処理以外の処理を行う。
| Controllerの一部とみなしてよい。

Helperはoptionであり、必要に応じて、POJOクラスとして作成すること。

.. note::

  HelperはControllerの見通しを良くするためのものであり、HelperはControllerの一部だと思えばよい。
  
  Controllerの役割はルーティング(URLマッピングと遷移先の返却)であり、それ以外の処理(JavaBeanの変換等)が必要になったらHelperに切り出してそちらに処理を移すことを推奨する。
  
  あくまでもControllerをすっきりさせて、本来のControllerの処理が見やすくなることを目的としており、Controller内のprivateメソッドみたいなものである。

    

ドメイン層
--------------------------------------------------------------------------------

| ドメイン層は、アプリケーションのコアとなる層である。ビジネス上の解決すべき問題を表現し、
| ビジネスオブジェクトや、ビジネスルールを含む(口座へ入金する場合に、残高が十分であるかどうかのチェックなど)。
| ドメイン層は、他の層からは疎であり、再利用できる。

Domain Object
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| Domain Objectはビジネスを行う上で必要な資源や、ビジネスを行っていく過程で発生するものを表現するモデル。
| 大きく分けて、以下3つに分類される。
* EmployeeやCustomer, Productなどのリソース系モデル(一般的には、名詞で表現される）,
* Order, Paymentなどイベント系モデル(一般的には動詞で表現される)、
* YearlySales, MonthlySalesなどのサマリ系モデル

データベースのあるテーブルの、1レコードを表現するオブジェクトを表現するEntityは、Domain Objectである。

.. note::
  本ガイドラインでは主に、\ `状態のみもつモデル <http://martinfowler.com/bliki/AnemicDomainModel.html>`_\ を扱う。

  Martin Fowlerの"Patterns of Enterprise Application Architecture (2002, Addison-Wesley)"では、
  Domain Modelは、\ `状態と振る舞いをもつもの <http://martinfowler.com/eaaCatalog/domainModel.html>`_\ と定義されているが、
  厳密には触れない。

  Eric Evansの提唱するような\ `Richなドメインモデル <http://domaindrivendesign.org>`_\ も、本ガイドラインでは扱わないが、
  分類上はここに含まれる。

Repository
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| Domain Objectのコレクションのような位置づけであり、Domain Objectの問い合わせや、作成、更新、削除のようなCRUD処理を担う。
| この層では、インタフェースのみ定義され、実体は、インフラストラクチャ層のRepositoryImplで実装されるため、
| どのようなデータアクセスが行われているかについての情報は持たない。

Service
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

業務処理を提供する。
この処理も、トランザクション境界となる。

| Serviceでは、FormやHttpRequestなど、Webに関わる情報を扱うべきではない。
| これらの情報は、Serviceの前のApplication層で、ドメイン層のオブジェクトに変換されるべきである。

インフラストラクチャ層
--------------------------------------------------------------------------------

| インフラストラクチャ層では、ドメイン層(Repositoryインタフェース)の実装を提供する。
| データストア(RDBMSや、NoSQLなどのデータを格納する場所)への永続化や、メッセージの送信などを担う。

RepositoryImpl
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| RepositoryImplは、Repositoryの実装であり、Domain Objectのライフサイクル管理を隠蔽する。
| これにより、ドメイン層がどのようにデータアクセスされているか意識しなくて済む。

Spring Data JPAを使用する場合は、Spring Data JPAが実体を(一部)自動で作成する。

O/R Mapper
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| データベースとEntityの相互マッピングを担う。
| JPAや、MyBatis, Spring JDBCが本機能を提供する。
| 特に、JPAを用いる場合はEntityManager、MyBatis2(TERASOLUNA DAO)を用いる場合は、QueryDAO, UpdateDAOが該当する。

.. note::

  MyBatis, Spring JDBCは「O/R Mapper」というより、「SQL Mapper」と呼んだ方が正確であるが、本ガイドラインでは「O/R Mapper」に分類する。

Integration System Connector
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| メッセージングシステムや、Key-Value-Store、Webサービス、既存システムなど、
| データベース以外のデータストア、あるいは外部システムとの連携を担う。
| Repositoryの実装に用いられる。

|

レイヤ間の依存関係
================================================================================

| 冒頭で説明したとおり、ドメイン層がコアとなり、アプリケーション層、インフラストラクチャ層がそれに依存する形となる。

| 本ガイドラインでは、実装技術として、
* アプリケーション層にSpring MVC
* インフラストラクチャ層にSpring Data JPA, MyBatis
| を使用することを想定しているが、本質的には、実装技術が変わっても、それぞれの層で違いが吸収され、ドメイン層には影響を与えない。
| レイヤ間の結合部は、インタフェースとして公開することで、各層が使用している実装技術に依存しない形式とすることができる。

レイヤ化を意識して、疎結合な設計を行うことを推奨する。

.. figure:: images/LayerDependencies.png
   :width: 80%


各レイヤのオブジェクトの依存関係は、DIコンテナによって解決される。

.. figure:: images/LayerDependencyInjection.png
   :width: 90%


入力から出力までの流れで表現すると、次の図のようになる。

.. figure:: images/LayeringPattern1.png
   :alt: Data flow from request to reponse
   :width: 100%

更新系の処理を例に、シーケンスを説明する。

#. Controllerが、Requestを受け付ける
#. (Optional) Controllerは、Helperを呼び出し、Formの情報を、Domain ObjectまたはDTOに変換する
#. Controllerは、Domain ObjectまたはDTOを用いて、Serviceを呼び出す
#. Serviceは、Repositoryを呼び出して、業務処理を行う
#. Repositoryは、O/R Mapperを呼び出し、Domain ObjectまたはDTOを永続化する
#. (実装依存) O/R Mapperは、DBにDomain ObjectまたはDTOの情報を保存する
#. Serviceは、業務処理結果のDomain ObjectまたはDTOを、Controllerに返却する
#. (Optional) Controllerは、Helperを呼び出し、Domain ObjectまたはDTOを、Formに変換する
#. Controllerは、遷移先のView名を返却する
#. Viewは、Responseを出力する。


この場合の各コンポーネント間の呼び出し可否を、以下にまとめる。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|
.. list-table:: コンポーネント間の呼び出し可否
    :header-rows: 1
    :stub-columns: 1

    * - Caller/Callee
      - Controller
      - Service
      - Repository
      - O/R Mapper
    * - Controller
      - .. image:: images/cross.png
           :align: center
      - .. image:: images/tick.png
           :align: center
      - .. image:: images/cross.png
           :align: center
      - .. image:: images/cross.png
           :align: center
    * - Service
      - .. image:: images/cross.png
           :align: center
      - .. image:: images/exclamation.png
           :align: center
      - .. image:: images/tick.png
           :align: center
      - .. image:: images/cross.png
           :align: center
    * - Repository
      - .. image:: images/cross.png
           :align: center
      - .. image:: images/cross.png
           :align: center
      - .. image:: images/cross.png
           :align: center
      - .. image:: images/tick.png
           :align: center


| 注意するべきことは、\ **基本的にServiceからServiceの呼び出しは、禁止している**\ 点である。
| もし他のサービスからも利用可能なサービスが必要な場合は、呼び出し可否を明確にするために、SharedServiceを作成すること。
| 詳細については、\ :doc:`../ImplementationAtEachLayer/DomainLayer`\ を参照されたい。


.. note::
   この呼び出し可否ルールを守ることは、アプリケーション開発の初期段階では、煩わしく感じられるかもしれない。
   確かに、一つの処理だけみると、たとえばControllerから直接Repositoryを呼び出したほうが、速くアプリケーションを作成できる。
   しかし、ルールを守らない場合、開発規模が大きくなった際に、修正の影響範囲が分かりにくくなったり、横断的な共通処理を追加しにくくなるなど、
   保守性に大きな問題が生じることが多い。後で問題にならないように、初めから依存関係に気を付けて開発することを強く推奨する。


| Repositoryを作成することにより、永続化技術を隠蔽できたり、データアクセス処理を共通化できるなどのメリットがある。
| しかし、プロジェクトのチーム体制によっては、データアクセスの共通化が難しい場合がある（複数の会社が、別々に業務処理を実装し、共通化のコントロールが難しい場合など）。
| その場合、データアクセスの抽象化が必要ないのであれば、Repositoryは作成せず、以下の図のように、Serviceから直接O/R Mapperを呼び出すようにすればよい。

.. figure:: images/LayeringPattern2.png
   :alt: Data flow from request to reponse (without Repository)
   :width: 100%


この場合の呼び出し可否は、次のようになる。

.. tabularcolumns:: |p{0.25\linewidth}|p{0.25\linewidth}|p{0.25\linewidth}|p{0.25\linewidth}|
.. list-table:: コンポーネント間の呼び出し可否 (without Repository)
    :header-rows: 1
    :stub-columns: 1

    * - Caller/Callee
      - Controller
      - Service
      - O/R Mapper
    * - Controller
      - .. image:: images/cross.png
           :align: center
      - .. image:: images/tick.png
           :align: center
      - .. image:: images/cross.png
           :align: center
    * - Service
      - .. image:: images/cross.png
           :align: center
      - .. image:: images/exclamation.png
           :align: center
      - .. image:: images/tick.png
           :align: center

|

.. _application-layering_project-structure:

プロジェクト構成
================================================================================

上記のように、アプリケーションのレイヤ化を行った場合に推奨する構成について、説明する。

ここでは、Mavenの標準ディレクトリ構造を前提とする。

基本的には、以下の構成でマルチプロジェクトを作成することを推奨する。

* [projectname]-domain ... ドメイン層に関するクラス・設定ファイルを格納するプロジェクト
* [projectname]-web ... アプリケーション層に関するクラス・設定ファイルを格納するプロジェクト
* [projectname]-env ... 環境に依存するファイル等を格納するプロジェクト

([projectname]には、対象のプロジェクト名を入れること)


.. note::

  RepositoryImplなどインフラストラクチャ層のクラスも、project-domainに含める。

  本来は、[projectname]-infraプロジェクトを別途作成すべきであるが、
  通常infraプロジェクトを隠蔽化する必要がなく、domainプロジェクトに格納されている方が開発しやすいためである。
  必要であれば、[projectname]-infraプロジェクトを作成してよい。


.. tip::

  マルチプロジェクト構成の例として、\ `サンプルアプリケーション <https://github.com/terasolunaorg/terasoluna-tourreservation>`_\ や\ `共通ライブラリのテストアプリケーション <https://github.com/terasolunaorg/terasoluna-gfw-functionaltest>`_\ を参照されたい。

[projectname]-domain
--------------------------------------------------------------------------------

[projectname]-domainのプロジェクト推奨構成を、以下に示す。

.. code-block:: console

    [projectName]-domain
      └src
          └main
              ├java
              │  └com
              │      └example
              │          └domain ...(1)
              │              ├model
              │              │  ├Xxx.java
              │              │  ├Yyy.java
              │              │  └Zzz.java
              │              ├repository ...(2)
              │              │  ├xxx
              │              │  │  └XxxRepository.java
              │              │  ├yyy
              │              │  │  └YyyRepository.java
              │              │  └zzz
              │              │      ├ZzzRepository.java
              │              │      └ZzzRepositoryImpl.java
              │              └service ...(3)
              │                  ├aaa
              │                  │  ├AaaService.java
              │                  │  └AaaServiceImpl.java
              │                  └bbb
              │                      ├BbbService.java
              │                      └BbbServiceImpl.java
              └resources
                  └META-INF
                      └spring
                          ├[projectname]-domain.xml ...(4)
                          └[projectname]-infra.xml ...(5)


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ドメインオブジェクトを格納する。
    * - | (2)
      - | リポジトリを格納する。エンティティごとにパッケージを作成する。
        | 関連するエンティティがあれば、主となるエンティティのパッケージに、従となるエンティティのRepositoryも配置する。
        | (OrderとOrderLineなど)。DTOが必要な場合は、このパッケージに配置する。
        | RepositoryImplは、インフラストラクチャ層に属するが、通常、このプロジェクトに含めても問題ない。
        | 異なるデータストアを使うなど、複数の永続化先があり、実装を隠蔽したい場合は、別プロジェクト(またはパッケージ)に、RepositoryImplを実装するようにする。
    * - | (3)
      - | サービスを格納する。業務(またはエンティティ)ごとに、パッケージインタフェースと実装を、同じ階層に配置する。
        | 入出力クラスが必要な場合は、このパッケージに配置する。
    * - | (4)
      - | ドメイン層に関するBean定義を行う。
    * - | (5)
      - | インフラストラクチャ層に関するBean定義を行う。


[projectname]-web
--------------------------------------------------------------------------------

[projectname]-webのプロジェクト推奨構成を、以下に示す。

.. code-block:: console

    [projectName]-web
      └src
          └main
              ├java
              │  └com
              │      └example
              │          └app ...(1)
              │              ├abc
              │              │  ├AbcController.java
              │              │  ├AbcForm.java
              │              │  └AbcHelper.java
              │              └def
              │                  ├DefController.java
              │                  ├DefForm.java
              │                  └DefOutput.java
              ├resources
              │  ├META-INF
              │  │  └spring
              │  │      ├applicationContext.xml ...(2)
              │  │      ├application.properties ...(3)
              │  │      ├spring-mvc.xml ...(4)
              │  │      └spring-security.xml ...(5)
              │  └i18n
              │      └application-messages.properties ...(6)
              └webapp
                  └WEB-INF
                      ├views ...(7)
                      │  ├abc
                      │  │ ├list.jsp
                      │  │ └createForm.jsp
                      │  └def
                      │     ├list.jsp
                      │     └createForm.jsp
                      └web.xml

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | アプリケーション層の構成要素を格納するパッケージ。
    * - | (2)
      - | アプリケーション全体に関するBean定義を行う。
    * - | (3)
      - | アプリケーションで使用するプロパティを定義する。
    * - | (4)
      - | SpringMVCの設定を行うBean定義を行う。
    * - | (5)
      - | SpringSecurityの設定を行うBean定義を行う。
    * - | (6)
      - | 画面表示用のメッセージ(国際化対応)定義を行う。
    * - | (7)
      - | View(jsp)を格納する。

[projectname]-env
--------------------------------------------------------------------------------

[projectname]-envのプロジェクト推奨構成を、以下に示す。

.. code-block:: console

    [projectName]-env
      └src
          └main
              └resources
                  └META-INF
                      └spring
                          ├[projectname]-env.xml ...(1)
                          └[projectname]-infra.properties ...(2)


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 環境に依存するBean定義(DataSource等)を行う。
    * - | (2)
      - | 環境に依存するプロパティを定義する。


.. note::

  [projectname]-domainと[projectname]-webを別プロジェクトに分ける理由は、依存関係の逆転を防ぐためである。
  
  [projectname]-webが[projectname]-domainを使用するのは当然であるが、[projectname]-domainが[projectname]-webを参照してはいけない。
  
  1つのプロジェクトに[projectname]-webと[projectname]-domainの構成要素をまとめてしまうと、誤って不正な参照してしまうことがある。
  プロジェクトを分けて参照順序をつけることで[projectname]-domainが[projectname]-webを参照できないようにすることを強く推奨する。

.. note::

  [projectname]-envを作成する理由は環境に依存する情報を外出し、環境毎に切り替えられるようにするためである。
  
  たとえばデフォルトではローカル開発環境用の設定をして、アプリビルド時には[projectname]-envを除いてwarを作成する。
  結合テスト用の環境やシステムテスト用の環境を別々のjarとして作成すると、そこだけ差し替えてデプロイするということが可能である。
  
  また使用するRDBMSが変わるようなプロジェクトの場合にも影響を最小限に抑えることができる。
  
  この点を考慮しない場合は、環境ごとに設定ファイルの内容を行いビルドしなおすという作業が入る。
  
  環境依存に関するファイルを別プロジェクトにする意義については、\ :doc:`../Appendix/EnvironmentIndependency`\ を参照されたい。

.. raw:: latex

   \newpage

