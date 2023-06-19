アプリケーションのレイヤ化
********************************************************************************

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

本ガイドラインでは、アプリケーションを、次の3レイヤに分割する。

* アプリケーション層
* ドメイン層
* インフラストラクチャ層

各層には、以下のコンポーネントが含まれる。

.. figure:: images/ApplicationLayer.png
   :alt: application layers
   :width: 95%



| アプリケーション層とインフラストラクチャ層は、ドメイン層に依存するが、\ **ドメイン層が、他の層に依存してはいけない。**
| ドメイン層の変更によって、アプリケーション層に変更が生じるのは良いが、
| アプリケーション層の変更によって、ドメイン層の変更が生じるべきではない。

各層について、説明する。

.. note::

  アプリケーション層、ドメイン層、インフラストラクチャー層は
  Eric Evansの"Domain-Driven Design (2004, Addison-Wesley)"で説明されてる用語である。
  ただし、用語は使用しているが以後Domain Driven Designの考えにのっとっているわけではない。

|

レイヤの定義
================================================================================

入力から出力までのデータの流れは、アプリケーション層→ドメイン層→インフラストラクチャ層であるため、
この順に説明する。

アプリケーション層
--------------------------------------------------------------------------------

アプリケーション層は、クライアントとのデータの入出力を制御する層である。

この層では、

* データの入出力を行うUI(User Interface)の提供
* クライアントからのリクエストハンドリング
* 入力データの妥当性チェック
* リクエスト内容に対応するドメイン層のコンポーネントの呼び出し

などの実装を行う。

**この層で行う実装は、できるだけ薄く保たれるべきであり、ビジネスルールを含んではいけない。**

Controller
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Controllerは、主に以下の役割を担う。

* 画面遷移の制御（リクエストマッピングと処理結果に対応するViewを返却する)
* ドメイン層のServiceの呼び出し (リクエストに対応する主処理を実行する)

Spring MVCでは、\ ``@Controller``\ アノテーションが付与されているPOJOクラスが該当する。

.. note::

    クライアントとの入出力データをセッションに格納する場合は、
    セッションに格納するデータのライフサイクルを制御する役割も担う。

|

View
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Viewは、クライアントへの出力(UIの提供を含む)を担う。HTML/PDF/Excel/JSONなど、様々な形式で出力結果を返す。

Spring MVCでは、\ ``View``\ クラスが該当する。

.. tip::

    REST APIやAjax向けのリクエストでJSONやXML形式の出力を行う場合は、\ ``HttpMessageConverter``\ クラスが\ ``View``\の役割を担う。

    詳細は、「:doc:`../ArchitectureInDetail/REST`」を参照されたい。

|

Form
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Formは、主に以下の役割を担う。

* HTMLのフォームを表現（フォームのデータをControllerに渡したり、処理結果をフォームに出力する）
* 入力チェックルールの宣言 (Bean Validationのアノテーションを付与する)

Spring MVCでは、Formオブジェクトは、リクエストパラメータを保持するPOJOクラスが該当する。form backing beanと呼ばれる。

.. note::

    ドメイン層がアプリケーション層に依存しないようにするために、以下の変換処理をアプリケーション層で行う。

    * FormからDomain Object(Entity等)への変換処理
    * Domain ObjectからFormへの変換処理

    これらの変換処理をController内で行うと、ソースコードが長くなり、
    本来のControllerの処理(画面遷移など)の見通しが、悪くなりがちである。

    変換処理のコードが多くなる場合は、以下のいずれか又は両方の対策を行い、
    Controller内のソースコードをシンプルな状態に保つこと推奨する。

    * Helperクラスを作成して変換処理を委譲する
    * :doc:`Dozer <../ArchitectureInDetail/Utilities/Dozer>` を使用する

.. tip::

    REST APIやAjax向けのリクエストでJSONやXML形式の入力を受ける場合は、\ ``Resource``\ クラスが\ ``Form``\の役割を担う。
    また、JSONやXML形式の入力データを\ ``Resource``\ クラスに変換する役割は、\ ``HttpMessageConverter``\ クラスが担う。

    詳細は、「:doc:`../ArchitectureInDetail/REST`」を参照されたい。

|

Helper
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Helperは、Controllerを補助する役割を担う。

Helperの作成はオプションである。必要に応じて、POJOクラスとして作成すること。

.. note::

  Controllerの役割はルーティング(URLマッピングと遷移先の返却)であり、
  それ以外の処理(JavaBeanの変換等)が必要になったらHelperに切り出して、そちらに処理を委譲することを推奨する。
  
  HelperはControllerの見通しを良くするためのものであるため、HelperはControllerの一部として扱ってよい。
  (Controller内のprivateメソッドみたいなものである)

|

ドメイン層
--------------------------------------------------------------------------------

ドメイン層は、アプリケーションのコアとなる層であり、ビジネスルールを実行(業務処理を提供)する。

この層では、

* Domain Object
* Domain Objectに対するビジネスルールのチェック(口座へ入金する場合に、残高が十分であるかどうかのチェックなど)
* Domain Objectに対するビジネスルールの実行(ビジネスルールに則った値の反映)
* Domain Objectに対するCRUD操作

などの実装を行う。

ドメイン層は、他の層からは疎であり、再利用できる。

Domain Object
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Domain Objectはビジネスを行う上で必要な資源や、ビジネスを行っていく過程で発生するものを表現するモデルである。

Domain Objectは、大きく分けて、以下3つに分類される。

* EmployeeやCustomer, Productなどのリソース系モデル(一般的には、名詞で表現される）
* Order, Paymentなどイベント系モデル(一般的には動詞で表現される)
* YearlySales, MonthlySalesなどのサマリ系モデル

データベースのテーブルの1レコードを表現するクラスであるEntityは、Domain Objectである。

.. note::

   本ガイドラインでは主に、\ `状態のみもつモデル <http://martinfowler.com/bliki/AnemicDomainModel.html>`_\ を扱う。

   Martin Fowlerの"Patterns of Enterprise Application Architecture (2002, Addison-Wesley)"では、
   Domain Modelは、\ `状態と振る舞いをもつもの <http://martinfowler.com/eaaCatalog/domainModel.html>`_\ と定義されているが、
   厳密には触れない。

   Eric Evansの提唱するような\ `Richなドメインモデル <http://domaindrivendesign.org>`_\ も、本ガイドラインでは扱わないが、
   分類上はここに含まれる。

|

Repository
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Domain Objectのコレクションのような位置づけであり、Domain Objectの問い合わせや、作成、更新、削除のようなCRUD処理を担う。

この層では、インタフェースのみ定義する。

実体はインフラストラクチャ層のRepositoryImplで実装するため、
どのようなデータアクセスが行われているかについての情報は持たない。

|

Service
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

業務処理を提供する。

本ガイドラインでは、Serviceのメソッドをトランザクション境界にすることを推奨している。

.. note::

    Serviceでは、FormやHttpRequestなど、Webに関わる情報を扱うべきではない。

    これらの情報は、Serviceのメソッドを呼び出す前に、アプリケーション層でドメイン層のオブジェクトに変換すべきである。

|

インフラストラクチャ層
--------------------------------------------------------------------------------

インフラストラクチャ層は、ドメイン層(Repositoryインタフェース)の実装を提供する層である。

データストア(RDBMSや、NoSQLなどのデータを格納する場所)への永続化や、メッセージの送信などを担う。

RepositoryImpl
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

RepositoryImplは、Repositoryインタフェースの実装として、Domain Objectのライフサイクル管理を行う処理を提供する。

RepositoryImplの実装はRepositoryインタフェースによって隠蔽されるため、
ドメイン層のコンポーネント(Serviceなど)では、どのようにデータアクセスされているか意識しなくて済む。

要件によっては、この処理もトランザクション境界となりうる。

.. tip::

    MyBatis3やSpring Data JPAを使用する場合は、RepositoryImplの実体を(一部)自動で作成する仕組みが提供されている。

|

O/R Mapper
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

O/R Mapperは、データベースとEntityの相互マッピングを担う。

MyBatis / JPA / Spring JDBCが、本機能を提供する。

具体的には、

* MyBatis3を用いる場合は、Mapperインタフェースや\ ``SqlSession``\
* JPAを用いる場合は、\ ``EntityManager``\
* Spring JDBCを用いる場合は、\ ``JdbcTemplate``\

が、O/R Mapperに該当する。

O/R Mapperは、Repositoryインタフェースの実装に用いられる。

.. note::

  MyBatis, Spring JDBCは「O/R Mapper」というより、「SQL Mapper」と呼んだ方が正確であるが、本ガイドラインでは「O/R Mapper」に分類する。

|

Integration System Connector
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Integration System Connectorは、
データベース以外のデータストア（メッセージングシステム、Key-Value-Store、Webサービス、既存システム、外部システムなど）との連携を担う。

Integration System Connectorは、Repositoryインタフェースの実装に用いられる。

|

レイヤ間の依存関係
================================================================================

冒頭で説明したとおり、ドメイン層がコアとなり、アプリケーション層、インフラストラクチャ層がそれに依存する形となる。

本ガイドラインでは、実装技術として、

* アプリケーション層にSpring MVC
* インフラストラクチャ層にMyBatis, Spring Data JPA

を使用することを想定しているが、本質的には、実装技術が変わっても、それぞれの層で違いが吸収され、ドメイン層には影響を与えない。
レイヤ間の結合部は、インタフェースとして公開することで、各層が使用している実装技術に依存しない形式とすることができる。

レイヤ化を意識して、疎結合な設計を行うことを推奨する。

.. figure:: images/LayerDependencies.png
   :width: 95%

|

各レイヤのオブジェクトの依存関係は、DIコンテナによって解決される。

.. figure:: images/LayerDependencyInjection.png
   :width: 95%

|

Repositoryを使用する時の処理の流れ
--------------------------------------------------------------------------------

入力から出力までの流れで表現すると、次の図のようになる。

.. figure:: images/LayeringPattern1.png
   :alt: Data flow from request to response
   :width: 100%

更新系の処理を例に、シーケンスを説明する。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - 1.
      - Controllerが、Requestを受け付ける
    * - 2.
      - (Optional) Controllerは、Helperを呼び出し、Formの情報を、Domain ObjectまたはDTOに変換する
    * - 3.
      - Controllerは、Domain ObjectまたはDTOを用いて、Serviceを呼び出す
    * - 4.
      - Serviceは、Repositoryを呼び出して、業務処理を行う
    * - 5.
      - Repositoryは、O/R Mapperを呼び出し、Domain ObjectまたはDTOを永続化する
    * - 6.
      - (実装依存) O/R Mapperは、DBにDomain ObjectまたはDTOの情報を保存する
    * - 7.
      - Serviceは、業務処理結果のDomain ObjectまたはDTOを、Controllerに返却する
    * - 8.
      - (Optional) Controllerは、Helperを呼び出し、Domain ObjectまたはDTOを、Formに変換する
    * - 9.
      - Controllerは、遷移先のView名を返却する
    * - 10.
      - Viewは、Responseを出力する。

|

各コンポーネント間の呼び出し可否を、以下にまとめる。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|
.. list-table:: **コンポーネント間の呼び出し可否**
    :header-rows: 1
    :stub-columns: 1
    :widths: 20 20 20 20 20

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


注意するべきことは、\ **基本的にServiceからServiceの呼び出しは、禁止している**\ 点である。

もし他のサービスからも利用可能なサービスが必要な場合は、呼び出し可否を明確にするために、SharedServiceを作成すること。
詳細については、\ :doc:`../ImplementationAtEachLayer/DomainLayer`\ を参照されたい。


.. note::

    この呼び出し可否ルールを守ることは、アプリケーション開発の初期段階では、煩わしく感じられるかもしれない。
    確かに、一つの処理だけみると、たとえばControllerから直接Repositoryを呼び出したほうが、速くアプリケーションを作成できる。
    しかし、ルールを守らない場合、開発規模が大きくなった際に、修正の影響範囲が分かりにくくなったり、横断的な共通処理を追加しにくくなるなど、
    保守性に大きな問題が生じることが多い。後で問題にならないように、初めから依存関係に気を付けて開発することを強く推奨する。

|

Repositoryを使用しない時の処理の流れ
--------------------------------------------------------------------------------

Repositoryを作成することにより、永続化技術を隠蔽できたり、データアクセス処理を共通化できるなどのメリットがある。

しかし、プロジェクトのチーム体制によっては、データアクセスの共通化が難しい場合がある（複数の会社が、別々に業務処理を実装し、共通化のコントロールが難しい場合など）。
その場合、データアクセスの抽象化が必要ないのであれば、Repositoryは作成せず、以下の図のように、Serviceから直接O/R Mapperを呼び出すようにすればよい。

.. figure:: images/LayeringPattern2.png
   :alt: Data flow from request to response (without Repository)
   :width: 100%

|

各コンポーネント間の呼び出し可否を、以下にまとめる。

.. tabularcolumns:: |p{0.25\linewidth}|p{0.25\linewidth}|p{0.25\linewidth}|p{0.25\linewidth}|
.. list-table:: **コンポーネント間の呼び出し可否 (without Repository)**
    :header-rows: 1
    :stub-columns: 1
    :widths: 25 25 25 25

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

|

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - プロジェクト名
      - 説明
    * - [projectName]-domain
      - ドメイン層に関するクラス・設定ファイルを格納するプロジェクト
    * - [projectName]-web
      - アプリケーション層に関するクラス・設定ファイルを格納するプロジェクト
    * - [projectName]-env
      - 環境に依存するファイル等を格納するプロジェクト

([projectName]には、対象のプロジェクト名を入れること)


.. note::

    RepositoryImplなどインフラストラクチャ層のクラスも、project-domainに含める。

    本来は、[projectName]-infraプロジェクトを別途作成すべきであるが、
    通常infraプロジェクトを隠蔽化する必要がなく、domainプロジェクトに格納されている方が開発しやすいためである。
    必要であれば、[projectName]-infraプロジェクトを作成してよい。


.. tip::

    マルチプロジェクト構成の例として、\ `サンプルアプリケーション <https://github.com/terasolunaorg/terasoluna-tourreservation>`_\ や\ `共通ライブラリのテストアプリケーション <https://github.com/terasolunaorg/terasoluna-gfw-functionaltest>`_\ を参照されたい。

|

[projectName]-domain
--------------------------------------------------------------------------------

[projectName]-domainのプロジェクト推奨構成を、以下に示す。

.. code-block:: console

    [projectName]-domain
      └src
          └main
              ├java
              │  └com
              │      └example
              │          └domain ...(1)
              │              ├model ...(2)
              │              │  ├Xxx.java
              │              │  ├Yyy.java
              │              │  └Zzz.java
              │              ├repository ...(3)
              │              │  ├xxx
              │              │  │  └XxxRepository.java
              │              │  ├yyy
              │              │  │  └YyyRepository.java
              │              │  └zzz
              │              │      ├ZzzRepository.java
              │              │      └ZzzRepositoryImpl.java
              │              └service ...(4)
              │                  ├aaa
              │                  │  ├AaaService.java
              │                  │  └AaaServiceImpl.java
              │                  └bbb
              │                      ├BbbService.java
              │                      └BbbServiceImpl.java
              └resources
                  └META-INF
                      └spring
                          ├[projectName]-codelist.xml ...(5)
                          ├[projectName]-domain.xml ...(6)
                          └[projectName]-infra.xml ...(7)


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - ドメイン層の構成要素を格納するパッケージ。
    * - | (2)
      - Domain Objectを格納するパッケージ。
    * - | (3)
      - リポジトリを格納するパッケージ。

        エンティティごとにパッケージを作成する。
        関連するエンティティがあれば、主となるエンティティのパッケージに、従となるエンティティ(OrderとOrderLineの関係であればOrderLine)のRepositoryも配置する。
        また、検索条件などを保持するDTOなどが必要な場合は、このパッケージに配置する。

        RepositoryImplは、インフラストラクチャ層に属するが、通常、このプロジェクトに含めても問題ない。
        異なるデータストアを使うなど、複数の永続化先があり、実装を隠蔽したい場合は、別プロジェクト(またはパッケージ)に、RepositoryImplを実装するようにする。
    * - | (4)
      - サービスを格納するパッケージ。

        業務(またはエンティティ)ごとに、パッケージインタフェースと実装を、同じ階層に配置する。入出力クラスが必要な場合は、このパッケージに配置する。
    * - | (5)
      - コードリストのBean定義を行う。
    * - | (6)
      - ドメイン層に関するBean定義を行う。
    * - | (7)
      - インフラストラクチャ層に関するBean定義を行う。

|

[projectName]-web
--------------------------------------------------------------------------------

[projectName]-webのプロジェクト推奨構成を、以下に示す。

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
                  ├resources ...(7)
                  └WEB-INF
                      ├views ...(8)
                      │  ├abc
                      │  │ ├list.jsp
                      │  │ └createForm.jsp
                      │  └def
                      │     ├list.jsp
                      │     └createForm.jsp
                      └web.xml ...(9)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - アプリケーション層の構成要素を格納するパッケージ。
    * - | (2)
      - アプリケーション全体に関するBean定義を行う。
    * - | (3)
      - アプリケーションで使用するプロパティを定義する。
    * - | (4)
      - SpringMVCの設定を行うBean定義を行う。
    * - | (5)
      - SpringSecurityの設定を行うBean定義を行う。
    * - | (6)
      - 画面表示用のメッセージ(国際化対応)定義を行う。
    * - | (7)
      - 静的リソース(css、js、画像など)を格納する。
    * - | (8)
      - View(jsp)を格納する。
    * - | (9)
      - Servletのデプロイメント定義を行う。

|

[projectName]-env
--------------------------------------------------------------------------------

[projectName]-envのプロジェクト推奨構成を、以下に示す。

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
      - 全環境の環境依存ファイルを管理するためのディレクトリ。
    * - | (2)
      - 環境毎の環境依存ファイルを管理するためのディレクトリ。

        ディレクトリ名は、環境を識別する名前を指定する。
    * - | (3)
      - 環境毎の設定ファイルを管理するためのディレクトリ。

        サブディレクトリの構成や管理する設定ファイルは、(4)と同様。
    * - | (4)
      - ローカル開発環境用の設定ファイルを管理するためのディレクトリ。
    * - | (5)
      - ローカル開発環境用のBean定義(DataSource等)を行う。
    * - | (6)
      - ローカル開発環境用のプロパティを定義する。
    * - | (7)
      - ローカル開発環境用のログ出力定義を行う。


.. note::

    [projectName]-domainと[projectName]-webを別プロジェクトに分ける理由は、依存関係の逆転を防ぐためである。
  
    [projectName]-webが[projectName]-domainを使用するのは当然であるが、[projectName]-domainが[projectName]-webを参照してはいけない。
  
    1つのプロジェクトに[projectName]-webと[projectName]-domainの構成要素をまとめてしまうと、誤って不正な参照をしてしまうことがある。
    プロジェクトを分けて参照順序をつけることで[projectName]-domainが[projectName]-webを参照できないようにすることを強く推奨する。

.. note::

    [projectName]-envを作成する理由は環境に依存する情報を外出し、環境毎に切り替えられるようにするためである。
  
    たとえばデフォルトではローカル開発環境用の設定をして、アプリケーションビルド時には[projectName]-envを除いてwarを作成する。
    結合テスト用の環境やシステムテスト用の環境を別々のjarとして作成すると、そこだけ差し替えてデプロイするということが可能である。
  
    また使用するRDBMSが変わるようなプロジェクトの場合にも影響を最小限に抑えることができる。
  
    この点を考慮しない場合は、環境ごとに設定ファイルの内容を行いビルドしなおすという作業が入る。
  
    環境依存に関するファイルを別プロジェクトにする意義については、\ :doc:`../Appendix/EnvironmentIndependency`\ を参照されたい。

.. raw:: latex

   \newpage

