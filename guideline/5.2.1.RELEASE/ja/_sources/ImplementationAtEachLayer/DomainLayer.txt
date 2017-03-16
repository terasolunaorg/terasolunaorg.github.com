ドメイン層の実装
================================================================================

.. only:: html

 .. contents:: 目次
    :local:
    :depth: 3

ドメイン層の役割
--------------------------------------------------------------------------------
ドメイン層は、 アプリケーション層に提供する\ **業務ロジックを実装する**\ ためのレイヤとなる。

ドメイン層の実装は、以下3つに分かれる。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 30 60

   * - 項番
     - 分類
     - 説明
   * - | 1.
     - | :ref:`domainlayer_entity`
     - | 業務データを保持するためのクラス(Entityクラス)を作成する。
   * - | 2.
     - | :ref:`repository-label`
     - | 業務データを操作するためのメソッドを実装し、Serviceクラスに提供する。
       | 業務データを操作するためのメソッドとは、具体的には、Entityオブジェクトに対するCRUD操作となる。
   * - | 3.
     - | :ref:`service-label`
     - | 業務ロジックを実行するためのメソッドを実装し、アプリケーション層に提供する。
       | 業務ロジック内で必要となる業務データは、Repositoryを介して、Entityオブジェクトとして取得する。

本ガイドラインでは、以下2点を目的として、EntityクラスおよびRepositoryを作成する構成を推奨している。

#. 業務ロジック(Service)と業務データへアクセスするためのロジックを分離することで、\ **業務ロジックの実装範囲をビジネスルールに関する実装に専念させる。**\
#. 業務データに対する操作をRepositoryに集約することで、\ **業務データへのアクセスの共通化を行う。**\

 .. note::

    本ガイドラインでは、EntityクラスおよびRepositoryを作成する構成を推奨しているが、この構成で開発することを強制するものではない。

    作成するアプリケーションの特性、プロジェクトの特性(開発体制や開発プロセスなど)を加味して、採用する構成を決めて頂きたい。


ドメイン層の開発の流れ
--------------------------------------------------------------------------------
| ドメイン層の開発の流れと、役割分担について説明する。
| 下記の説明では、複数の開発チームが存在する状態でアプリケーションを構築するケースを想定しているが、 １チームで開発する場合でも、開発フロー自体は変わらない。

 .. figure:: images/service_implementation_flow.png
    :alt: implementation flow of domain layer
    :width: 100%
    :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 20 70

   * - 項番
     - 担当チーム
     - 説明
   * - | (1)
     - | 共通開発チーム
     - | 共通開発チームは、Entityクラスの設計およびEntityクラスの作成を行う。
   * - | (2)
     - | 共通開発チーム
     - | 共通開発チームは、(1)で抽出したEntityクラスに対するアクセスパターンを整理し、Repositoryインタフェースのメソッド設計を行う。
       | 複数の開発チームで共有するメソッドに対する実装については、共通開発チームで実装することが望ましい。
   * - | (3)
     - | 共通開発チーム
     - | 共通開発チームは、(1)と(2)で作成したEntityクラスと、Repositoryを業務アプリケーション開発チームに提供する。
       | このタイミングで、各業務アプリケーション開発チームに対して、Repositoryインタフェースの実装を依頼する。
   * - | (4)
     - | 業務アプリケーション開発チーム
     - | 業務アプリケーション開発チームは、自チーム担当分のRepositoryインタフェースの実装を行う。
   * - | (5)
     - | 業務アプリケーション開発チーム
     - | 業務アプリケーション開発チームは、共通開発チームから提供されたEntityクラスおよびRepositoryと自チームで作成したRepositoryを利用して、ServiceインタフェースおよびServiceクラスの実装を行う。

 .. warning::

    開発規模が大きいシステムでは、アプリケーションを複数のチームに分担して開発を行う場合がある。
    その場合は、EntityクラスおよびRepositoryを設計するための共通チームを設けることを強く推奨する。

    共通チームを設ける体制が組めない場合は、EntityクラスおよびRepositoryの作成せずに、
    ServiceからO/R Mapper(MyBatisなど)を直接呼び出して、業務データにアクセスする方法を採用することを検討すること。


.. _domainlayer_entity:

Entityの実装
--------------------------------------------------------------------------------

Entityクラスの作成方針
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Entityは原則以下の方針で作成する。
| 具体的な作成方法については、\ :ref:`domainlayer_entity_example`\ で示す。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 35 55

   * - 項番
     - 方針
     - 補足
   * - | 1.
     - | Entityクラスは、テーブル毎に作成する。
     - | ただし、テーブル間の関連を保持するためのマッピングテーブルについては、Entityクラスは不要である。
       | また、テーブルが正規化されていない場合は、必ずしもテーブル毎にはならない。テーブルが正規化されていない時のアプローチは、\ :ref:`表外の警告欄と備考欄 <domainlayer_entity_policy_warning_note>`\ を参照されたい。
   * - | 2.
     - | テーブルにFK(Foreign Key)がある場合は、FK先のテーブルのEntityクラスをプロパティとして定義する。
     - | FK先のテーブルとの関係が、1:Nになる場合は、\ ``java.util.List<E>``\ または\ ``java.util.Set<E>``\ のどちらかを使用する。
       | FK先のテーブルに対応するEntityのことを、本ガイドライン上では、関連Entityと呼ぶ。
   * - | 3.
     - | コード系テーブルは、Entityとして扱うのではなく、\ ``java.lang.String``\ などの基本型で扱う。
     - | コード系テーブルとは、コード値と、コード名のペアを管理するためのテーブルのことである。
       | コード値によって処理分岐する必要がある場合は、コード値に対応するenumクラスを作成し、作成したenumをプロパティとして定義することを推奨する。

.. _domainlayer_entity_policy_warning_note:

 .. warning::

    テーブルが正規化されていない場合は、 以下の点を考慮して **EntityクラスおよびRepositoryを作成する方式を採用すべきか検討した方がよい。**
    特に正規化されていないテーブルとJPAとの相性はあまりよくないので、テーブルが正規化されていない場合は、JPAを使用してEntityオブジェクトを操作する方式は採用しない方が無難である。

    * | Entityを作成する難易度が高くなるため、適切なEntityクラスの作成が出来ない可能性がある。
      | 加えて、Entityクラスを作成するために、必要な工数が多くなる可能性も高い。
      | 前者は、「適切に正規化できるエンジニアをアサインできるか？」という観点、後者は、「工数をかけて正規化されたEntityクラスを作成する価値があるか？」という観点で、検討することになる。
    * | 業務データにアクセスする際の処理として、Entityクラスとテーブルの構成の差分を埋めるための処理が、必要となる。
      | これは、「工数をかけて、Entityとテーブルの差分を埋めるための処理を実装する価値があるか？」という観点で検討することになる。

    EntityクラスとRepositoryを作成する方式を採用することを推奨するが、作成するアプリケーションの特性、
    プロジェクトの特性(開発体制や開発プロセスなど)を加味して、採用する構成を決めて頂きたい。

.. _domainlayer_entity_policy_note:

 .. note::

    テーブルは正規化されていないが、アプリケーションとして、正規化されたEntityとして業務データを扱いたい場合は、
    インフラストラクチャ層のRepositoryImplの実装として、MyBatisを採用することを推奨する。

    MyBatisは、データベースで管理されているレコードとオブジェクトをマッピングするという考え方ではなく、
    SQLとオブジェクトをマッピングという考え方で開発されたO/R Mapperであるため、
    SQLの実装次第で、テーブル構成に依存しないオブジェクトへのマッピングができる。


.. _domainlayer_entity_example:

Entityクラスの作成例
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Entityクラスの作成方法を、具体例を用いて説明する。
| 以下は、ショッピングサイトで商品を購入する際に必要となる業務データを、Entityクラスとして作成する例となっている。

テーブル構成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
商品を購入する際に必要となる業務データを保持するテーブルは、以下の構成となっている。

 .. figure:: images/service_entity_table_layout.png
    :alt: Example of table layout
    :width: 100%
    :align: center

 .. raw:: latex

    \newpage

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.15\linewidth}|p{0.55\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 15 55
    :class: longtable

    * - 項番
      - 分類
      - テーブル名
      - 説明
    * - | (1)
      - | トランザクション系
      - | t_order
      - | 注文を保持するテーブル。１つの注文に対して1レコードが格納される。
    * - | (2)
      - |
      - | t_order_item
      - | １つの注文で購入された商品を保持するテーブル。１つの注文で複数の商品が購入された場合は商品数分レコードが格納される。
    * - | (3)
      - |
      - | t_order_coupon
      - | １つの注文で使用されたクーポンを保持するテーブル。１つの注文で複数のクーポンが使用された場合はクーポン数分レコードが格納される。クーポンを使用しなかった場合はレコードは格納されない。
    * - | (4)
      - | マスタ系
      - | m_item
      - | 商品を定義するマスタテーブル。
    * - | (5)
      - |
      - | m_category
      - | 商品のカテゴリを定義するマスタテーブル。
    * - | (6)
      - |
      - | m_item_category
      - | 商品が所属するカテゴリを定義するマスタテーブル。商品とカテゴリのマッピングを保持している。1つの商品は複数のカテゴリに属すことができるモデルとなっている。
    * - | (7)
      - |
      - | m_coupon
      - | クーポンを定義するマスタテーブル。
    * - | (8)
      - | コード系
      - | c_order_status
      - | 注文ステータスを定義するコードテーブル。

.. raw:: latex

   \newpage

Entity構成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
上記テーブルから作成方針に則ってEntityクラスを作成すると、以下のような構成となる。

 .. figure:: images/service_entity_entity_layout.png
    :alt: Example of entity layout
    :width: 100%
    :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.65\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 15 65
    :class: longtable

    * - 項番
      - クラス名
      - 説明
    * - | (1)
      - | Order
      - | t_orderテーブルの1レコードを表現するEntityクラス。
        | 関連Entityとして、\ ``OrderItem``\ および\ ``OrderCoupon``\ を複数保持する。
    * - | (2)
      - | OrderItem
      - | t_order_itemテーブルの1レコードを表現するEntityクラス。
        | 関連Entityとして、 ``Item`` を保持する。
    * - | (3)
      - | OrderCoupon
      - | t_order_couponテーブルの1コードを表現するEntityクラス。
        | 関連Entityとして、\ ``Coupon``\ を保持する。
    * - | (4)
      - | Item
      - | m_itemテーブルの1コードを表現するEntityクラス。
        | 関連Entityとして、所属している\ ``Category``\ を複数保持する。\ ``Item``\ と\ ``Category``\ の紐づけは、m_item_categoryテーブルによって行われる。
    * - | (5)
      - | Category
      - | m_categoryテーブルの1レコードを表現するEntityクラス。
    * - | (6)
      - | ItemCategory
      - | m_item_categoryテーブルは、m_itemテーブルとm_categoryテーブルとの関連を保持するためのマッピングテーブルなので、Entityクラスは作成しない。
    * - | (7)
      - | Coupon
      - | m_couponテーブルの1レコードを表現するEntityクラス。
    * - | (8)
      - | OrderStatus
      - | c_order_statusテーブルはコード系テーブルなので、Entityクラスは作成しない。

.. raw:: latex

   \newpage

上記のエンティティ図をみると、ショッピングサイトのアプリケーションとして主体のEntityクラスとして扱われるのは、
Orderクラスのみと思ってしまうかもしれないが、主体となる得るEntityクラスはOrderクラス以外にも存在する。

以下に、主体のEntityとしてなり得るEntityと、主体のEntityにならないEntityを分類する。

 .. figure:: images/service_entity_entity_class_layout.png
    :alt: Example of entity layout
    :width: 100%
    :align: center

|

ショッピングサイトのアプリケーションを作成する上で、主体のEntityとしてなり得るのは、以下4つである。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 30 60

   * - 項番
     - Entityクラス
     - 主体のEntityとなる得る理由
   * - | (1)
     - | Orderクラス
     - | ショッピングサイトにおいて、最も重要な主体となるEntityクラスのひとつである。
       | Orderクラスは、注文そのものを表現するEntityであり、Orderクラスなくしてショッピングサイトを作成することはできない。
   * - | (2)
     - | Itemクラス
     - | ショッピングサイトにおいて、最も重要な主体となるEntityクラスのひとつである。
       | Itemクラスは、ショッピングサイトで扱っている商品そのものを表現するEntityであり、Itemクラスなくしてショッピングサイトを作成することはできない。
   * - | (3)
     - | Categoryクラス
     - | 一般的なショッピングサイトでは、トップページや共通的メニューとして、サイトで扱っている商品のカテゴリを表示している。
       | このようなショッピングサイトのアプリケーションでは、Categoryクラスを主体のEntityとして扱うことになる。カテゴリの一覧検索などの処理が想定される。
   * - | (4)
     - | Couponクラス
     - | ショッピングサイトにおいて、商品の販売促進を行う手段としてクーポンによる値引きを行うことがある。
       | このようなショッピングサイトのアプリケーションでは、Couponクラスを主体のEntityとして扱うことなる。クーポンの一覧検索などの処理が想定される。


ショッピングサイトのアプリケーションを作成する上で、主体のEntityとならないのは、以下2つである。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 30 60

   * - 項番
     - Entityクラス
     - 主体のEntityにならない理由
   * - | (5)
     - | OrderItemクラス
     - | このクラスは、1つの注文で購入された商品1つを表現するクラスであり、Orderクラスの関連Entityとしてのみ存在するクラスとなる。
       | そのため、OrderItemクラスが、主体のEntityとして扱われることは原則ない。
   * - | (6)
     - | OrderCoupon
     - | このクラスは、1つの注文で使用されたクーポン1つを表現するクラスであり、Orderクラスの関連Entityとしてのみ存在するクラスとなる。
       | そのため、OrderCouponクラスが主体のEntityとして扱われることは原則ない。


.. _repository-label:

Repositoryの実装
--------------------------------------------------------------------------------

Repositoryの役割
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Repositoryは、以下2つの役割を担う。

1. | **Serviceに対して、Entityのライフサイクルを制御するための操作（Repositoryインタフェース）を提供する。**
   | Entityのライフサイクルを制御するための操作は、EntityオブジェクトへのCRUD操作となる。

 .. figure:: images/repository_responsibility_1.png
    :alt: provide access operations to entity
    :width: 100%
    :align: center


2. | **Entityを永続化する処理(Repositoryインタフェースの実装クラス)を提供する。**
   | Entityオブジェクトは、アプリケーションのライフサイクル(サーバの起動や、停止など)に依存しないレイヤに、永続化しておく必要がある。
   | Entityの永続先は、リレーショナルデータベースになることが多いが、NoSQLデータベース、キャッシュサーバ、外部システム、ファイル（共有ディスク）などになることもある。
   | 実際の永続化処理は、O/R Mapperなどから提供されているAPIを使って行う。
   | この役割は、インフラストラクチャ層のRepositoryImplで実装することになる。詳細については、\ :doc:`InfrastructureLayer`\ を参照されたい。

 .. figure:: images/repository_responsibility_2.png
    :alt: persist entity
    :width: 100%
    :align: center


Repositoryの構成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Repositoryは、RepositoryインタフェースとRepositoryImplで構成され、それぞれ以下の役割を担う。

 .. figure:: images/repository_classes_responsibility.png
   :alt: persist entity
   :width: 100%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.30\linewidth}|p{0.40\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 20 30 40

   * - 項番
     - クラス(インタフェース)
     - 役割
     - 説明

   * - | (1)
     - | Repositoryインタフェース
     - | 業務ロジック(Service)を実装する上で必要となるEntityのライフサイクルを制御するメソッドを定義する。
     - | 永続先に依存しないEntityの、CRUD操作用のメソッドを定義する。
       | Repositoryインタフェースは、業務ロジック(Service)を実装する上で必要となるEntityの操作を定義する役割を担うので、ドメイン層に属することになる。

   * - | (2)
     - | RepositoryImpl
     - | Repositoryインタフェースで定義されたメソッドの実装を行う。
     - | 永続先に依存したEntityのCRUD操作の実装を行う。実際のCRUD処理は、Spring Framework、O/R Mapper、ミドルウェアなどから提供されている永続処理用のAPIを利用して行う。
       | RepositoryImplは、Repositoryインタフェースで定義された操作の実装を行う役割を担うので、インフラストラクチャ層に属することになる。
       | RepositoryImplの実装については、\ :doc:`InfrastructureLayer`\ を参照されたい。


| 永続先が複数になる場合、以下のような構成となる。
| 以下のような構成を取ることで、Entityの永続先に依存したロジックを、業務ロジック(Service)から排除することができる。

 .. figure:: images/repository_not_depends_on.png
   :alt: persist entity
   :width: 100%
   :align: center

 .. note:: **永続先に依存したロジックを、Serviceから100％排除できるのか？**

    永続先の制約や、使用するライブラリの制約などにより、排除できないケースもある。
    可能な限り、永続先に依存するロジックは、Serviceではなく、RepositoryImplで実装することを推奨するが、
    永続先に依存するロジックを排除するのが難しい場合や、排除することで得られるメリットが少ない場合は、
    無理に排除せず、業務ロジック(Service)の処理として、永続先に依存するロジックを実装してもよい。

    排除できない具体例として、Spring Data JPAから提供されている\ ``org.springframework.data.jpa.repository.JpaRepository``\ インタフェース
    のsaveメソッドの呼び出し時に、一意制約エラーをハンドリングしたい場合である。
    JPAではEntityへの操作はキャッシュされ、トランザクションコミット時にSQLを発行する仕組みになっている。
    そのため、JpaRepositoryのsaveメソッドを呼び出しても、SQLは発行されないので、一意制約違反をロジックでハンドリングすることができない。
    JPAでは、明示的にSQLを発行する手段として、キャッシュされている操作を反映するためのメソッド（flushメソッド）があり、
    JpaRepositoryではsaveAndFlush、flushというメソッドが同じ目的で提供されている。
    そのため、Spring Data JPAのJpaRepositoryを使って、一意制約違反エラーをハンドリングする必要がある場合は、
    JPA依存のメソッド（saveAndFlushや、flush）を呼び出す必要がある。

 .. warning::

    Repositoryを設ける最も重要な目的は、永続先に依存するロジックを、業務ロジックから排除することではないという点である。
    最も重要な目的は、業務データへアクセスするための操作をRepositoryへ分離することで、業務ロジック(Service)の実装範囲をビジネスルールに関する実装に専念させるという点である。
    結果として、永続先に依存するロジックは業務ロジック(Service)ではなく、Repository側に実装される事になる。


Repositoryの作成方針
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Repositoryは原則以下の方針で作成する。


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 35 55

   * - 項番
     - 方針
     - 補足
   * - | 1.
     - | Repositoryは、主体となるEntityに対して作成する。
     - | これは、関連Entityを操作するためだけのRepositoryが不要であることを意味する。
       | ただし、アプリケーションの特性(高い性能要件があるアプリケーションなど)では、関連Entityを操作するためのRepositoryを設けた方が、よい場合もある。
   * - | 2.
     - | Repositoryインタフェースと、RepositoryImplは、基本的にドメイン層の同じパッケージに配置する。
     - | Repositoryは、Repositoryインタフェースがドメイン層、RepositoryImplがインフラストラクチャ層に属することとなるが、
       | Javaのパッケージとしては、基本的には、ドメイン層のRepositoryインタフェースと同じパッケージでよい。
   * - | 3.
     - | Repositoryで使用するDTOは、Repositoryインタフェースと同じパッケージに配置する。
     - | 例えば、検索条件を保持するDTOや、Entityの一部の項目のみを定義したサマリ用のDTOなどがあげられる。


Repositoryの作成例
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Repositoryの作成例を説明する。
| 以下は、\ :ref:`domainlayer_entity_example`\ の説明で使用した、EntityクラスのRepositoryを作成する例となっている。


Repository構成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\ :ref:`domainlayer_entity_example`\ の説明で使用した、EntityクラスのRepositoryを作成すると、以下のような構成となる。

 .. figure:: images/domainlayer_repository_layout.png
   :alt: Example of repository layout
   :width: 100%
   :align: center


| 主体となるEntityクラスに対して、Repositoryを作成している。
| パッケージの推奨構成については、\ :ref:`application-layering_project-structure`\ を参照されたい。


.. _repository-interface-label:

Repositoryインタフェースの定義
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Repositoryインタフェースの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

以下にRepositoryインタフェースの作成例を紹介する。

- :file:`SimpleCrudRepository.java`

 | このインタフェースは、シンプルなCRUD操作のみを提供している。
 | メソッドのシグネチャは、Spring Dataから提供されている\ ``CrudRepository``\ インタフェースや、\  ``PagingAndSortingRepository``\ インタフェースを参考に作成している。

 .. code-block:: java

     public interface SimpleCrudRepository<T, ID extends Serializable> {
         // (1)
         T findOne(ID id);
         // (2)
         boolean exists(ID id);
         // (3)
         List<T> findAll();
         // (4)
         Page<T> findAll(Pageable pageable);
         // (5)
         long count();
         // (6)
         T save(T entity);
         // (7)
         void delete(T entity);
     }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 指定したIDに対応するEntityを、取得するためのメソッド。
    * - | (2)
      - | 指定したIDに対応するEntityが、存在するか判定するためのメソッド。
    * - | (3)
      - | 全てのEntityを取得するためのメソッド。 Spring Dataでは、\ ``java.util.Iterable``\ であったが、サンプルとしては、\ ``java.util.List``\ にしている。
    * - | (4)
      - | 指定したページネーション情報（取得開始位置、取得件数、ソート情報）に該当するEntityのコレクションを取得するためのメソッド。
        | ``Pageable`` インタフェースおよび\ ``Page``\ インタフェースはSpring Dataより提供されているクラス（インターフェース）である。
    * - | (5)
      - | Entityの総件数を取得するためのメソッド。
    * - | (6)
      - | 指定されたEntityのコレクションを保存（作成、更新）するためのメソッド。
    * - | (7)
      - | 指定したEntityを、削除するためのメソッド。


- :file:`TodoRepository.java`

 下記は、チュートリアルで作成したTodoエンティティのRepositoryを、上で作成した\ ``SimpleCrudRepository``\ インタフェースベースに作成した場合の例である。

 .. code-block:: java

     // (1)
     public interface TodoRepository extends SimpleCrudRepository<Todo, String> {
         // (2)
         long countByFinished(boolean finished);
     }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明

    * - | (1)
      - | エンティティの型を示すジェネリック型「T」にTodoエンティティ、エンティティのID型を示すジェネリック型「ID」にStringクラスを指定することで、
        | Todoエンティティ用のRepositoryインタフェースが生成される。
    * - | (2)
      - | \ ``SimpleCrudRepository``\ インタフェースから提供されていないメソッドを追加している。
        | ここでは、「指定したタスクの終了状態に一致するTodoエンティティの件数を取得するメソッド」を追加している。


Repositoryインタフェースのメソッド定義
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 汎用的なCRUD操作を行うメソッドについては、Spring Dataから提供されている\ ``CrudRepository``\ や、\ ``PagingAndSortingRepository``\ と同じシグネチャにすることを推奨する。
| ただし、コレクションを返却する場合は、\ ``java.lang.Iterable``\ ではなく、ロジックで扱いやすいインタフェース（\ ``java.util.Collection``\ や、\ ``java.util.List``\ ）でもよい。
| 実際のアプリケーション開発では、汎用的なCRUD操作のみで開発できることは稀で、かならずメソッドの追加が必要になる。
| 追加するメソッドは、以下のルールに則り追加することを推奨する。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 70
    :class: longtable

    * - 項番
      - メソッドの種類
      - ルール
    * - 1.
      - 1件検索系のメソッド
      - #. メソッド名は、条件に一致するEntityを、1件取得するためのメソッドであることを明示するために、\ **findOneBy**\ で始める。
        #. メソッド名のfindOneBy以降は、検索条件となるフィールドの物理名、または、論理的な条件名などを指定し、どのような状態のEntityが取得されるのか、推測できる名前とする。
        #. 引数は、条件となるフィールド毎に用意する。ただし、条件が多い場合は、条件をまとめたDTOを用意してもよい。
        #. 返り値は、Entityクラスを指定する。
    * - 2.
      - 複数件検索系のメソッド
      - #. メソッド名は、条件に一致するEntityを、すべて取得するためのメソッドであることを明示するために、 **findAllBy** で始める。
        #. メソッド名のfindAllBy以降は、検索条件となるフィールドの物理名または論理的な条件名を指定し、どのような状態のEntityが取得されるのか推測できる名前とする。
        #. 引数は、条件となるフィールド毎に用意する。ただし、条件が多い場合は、条件をまとめたDTOを用意してもよい。
        #. 返り値は、Entityクラスのコレクションを指定する。
    * - 3.
      - 複数件ページ検索系のメソッド
      - #. メソッド名は、条件に一致するEntityの該当ページ部分を取得するためのメソッドである事を明示するために、 **findPageBy** で始める。
        #. メソッド名のfindPageBy以降は、検索条件となるフィールドの物理名または論理的な条件名を指定し、どのような状態のEntityが取得されるのか推測できる名前とする。
        #. 引数は、条件となるフィールド毎に用意する。ただし、条件が多い場合は、条件をまとめたDTOを用意してもよい。ページネーション情報（取得開始位置、取得件数、ソート情報）は、Spring Dataより提供されている ``Pageable`` インタフェースとすることを推奨する。
        #. 返り値は、Spring Dataより提供されている ``Page`` インタフェースとすることを推奨する。
    * - 4.
      - 件数のカウント系のメソッド
      - #. メソッド名は、条件に一致するEntityの件数をカウントするためのメソッドである事を明示するために、 **countBy** で始める。
        #. 返り値は、long型にする。
        #. メソッド名のcountBy以降は、検索条件となるフィールドの物理名または論理的な条件名を指定し、どのような状態のEntityの件数が取得されるのか推測できる名前とする。
        #. 引数は、条件となるフィールド毎に用意する。ただし、条件が多い場合は、条件をまとめたDTOを用意してもよい。
    * - 5.
      - 存在判定系のメソッド
      - #. メソッド名は、条件に一致するEntityが存在するかチェックするためのメソッドである事を明示するために、 **existsBy** で始める。
        #. メソッド名のexistsBy以降は、検索条件となるフィールドの物理名または論理的な条件名を指定し、どのような状態のEntityの存在チェックを行うのか推測できる名前とする。
        #. 引数は、条件となるフィールド毎に用意する。ただし、条件が多い場合は、条件をまとめたDTOを用意してもよい。
        #. 返り値は、boolean型にする。

 .. raw:: latex

    \newpage

 .. note::

     更新系のメソッドも、同様のルールに則り、追加することを推奨する。
     findの部分が、updateまたはdeleteとなる。


- :file:`Todo.java` (Entity)

 .. code-block:: java

     public class Todo implements Serializable {
         private String todoId;
         private String todoTitle;
         private boolean finished;
         private Date createdAt;
         // ...
      }

|

- :file:`TodoRepository.java`

 .. code-block:: java

      public interface TodoRepository extends SimpleCrudRepository<Todo, String> {
          // (1)
          Todo findOneByTodoTitle(String todoTitle);
          // (2)
          List<Todo> findAllByUnfinished();
          // (3)
          Page<Todo> findPageByUnfinished();
          // (4)
          long countByExpired(int validDays);
          // (5)
          boolean existsByCreateAt(Date date);
      }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | タイトルが一致するTODO(todoTitle=引数で指定した値のTODO)を取得するメソッドの定義例。
        | findOneBy以降に、条件となるフィールドの物理名(todoTitle)を指定している。
    * - | (2)
      - | 未完了のTODO(finished=falseのTODO)を全件取得するメソッドの定義例。
        | findAllBy以降に、論理的な条件名を指定している。
    * - | (3)
      - | 未完了のTODO(finished=falseのTODO)の該当ページ部分を取得するメソッドの定義例。
        | findPageBy以降に、論理的な条件名を指定している。
    * - | (4)
      - | 完了期限を過ぎたTODO(createdAt < sysdate - 引数で指定した有効日数 && finished=falseのTODO)の件数を取得するメソッドの定義例。
        | countBy以降に、論理的な条件名を指定している。
    * - | (5)
      - | 指定日に作成されている、TODO(createdAt=指定日)が存在するか判定するメソッドの定義例。
        | existsBy以降に、条件となるフィールドの物理名(createdAt)を指定している。


RepositoryImplの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
RepositoryImplの実装については、\ :doc:`InfrastructureLayer`\ を参照されたい。


.. _service-label:

Serviceの実装
--------------------------------------------------------------------------------

Serviceの役割
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Serviceは、以下2つの役割を担う。

1. | **Controllerに対して業務ロジックを提供する。**
   | 業務ロジックは、アプリケーションで使用する業務データの参照、更新、整合性チェックおよびビジネスルールに関わる各種処理で構成される。
   | 業務データの参照および更新処理をRepository(またはO/R Mapper)に委譲し、\ **Serviceではビジネスルールに関わる処理の実装に専念することを推奨する。**\

 .. note:: **ControllerとServiceで実装するロジックの責任分界点について**

    本ガイドラインでは、ControllerとServiceで実装するロジックは、以下のルールに則って実装することを推奨する。

    1. クライアントからリクエストされたデータに対する単項目チェック、相関項目チェックはController側(Bean ValidationまたはSpring Validator)で行う。

    2. Serviceに渡すデータへの変換処理(Bean変換、型変換、形式変換など)は、ServiceではなくController側で行う。

    3. \ **ビジネスルールに関わる処理はServiceで行う。**\ 業務データへのアクセスは、RepositoryまたはO/R Mapperに委譲する。

    4. ServiceからControllerに返却するデータ（クライアントへレスポンスするデータ）に対する値の変換処理(型変換、形式変換など)は、Serviceではなく、Controller側（Viewクラスなど）で行う。


 .. figure:: images/service_responsibility-of-logic.png
    :alt: responsibility of logic
    :width: 90%
    :align: center


2. | **トランザクション境界を宣言する。**
   | データの一貫性を保障する必要がある処理（主にデータの更新処理）を行う業務ロジックの場合、トランザクション境界を宣言する。
   | データの参照処理の場合でも業務要件によっては、トランザクション管理が必要になる場合もあるので、その場合は、トランザクション境界を宣言する。
   | \ **トランザクション境界は、原則Serviceに設ける。**\ アプリケーション層(Web層)にトランザクション境界が設けられている場合、業務ロジックの抽出が正しく行われていない可能性があるので、見直しを行うこと。

 .. figure:: images/service_transaction-boundary.png
    :alt: transaction boundary
    :width: 90%
    :align: center

 詳細は、\ :ref:`service_transaction_management`\を参照されたい。


.. _service-constitution-role-label:

Serviceのクラス構成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| Serviceは、ServiceクラスとSharedServiceクラスで構成され、それぞれ以下の役割を担う。
| 本ガイドラインでは、\ ``@Service``\ アノテーションが付与されたPOJO(Plain Old Java Object)のことを、ServiceクラスおよびSharedServiceクラスと定義しているが、メソッドのシグネチャを限定するようなインタフェースや、基底クラスを作成することを、禁止しているわけではない。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.30\linewidth}|p{0.45\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 15 30 45

   * - 項番
     - クラス
     - 役割
     - 依存関係に関する注意点

   * - 1.
     - Serviceクラス
     - | **特定のControllerに対して業務ロジックを提供する。**
       | Serviceクラスのメソッドは、\ **再利用されることを考慮したロジックは実装しない。**\
     - #. \ **他のServiceクラスのメソッドを呼び出すことは、原則禁止とする（※図中1-1）。**\ 他のServiceと処理を共有したい場合は、SharedServiceクラスのメソッドを作成し、呼び出すようにすることを推奨する。
       #. Serviceクラスのメソッドは、複数のControllerから呼び出してもよい（※図中1-2）。ただし、\ **呼び出し元のControllerによって、処理分岐が必要になる場合は、Controller毎に、Serviceクラスのメソッドを作成することを推奨する。**\ その上で共通的な処理は、SharedServiceクラスのメソッドを作成し呼び出すようにする。
   * - 2
     - SharedServiceクラス
     - | 複数のControllerやServiceクラスで、\ **共有(再利用)されるロジックを提供する。**\
     - #. 他のSharedServiceクラスのメソッドを呼び出してもよいが（※図中2-1）、 **呼び出し階層が複雑にならないように考慮すること。** 呼び出し階層が複雑になると保守性が低下する危険性が高まるので注意が必要。
       #. ControllerからSharedServiceクラスのメソッドを呼び出してもよい（※図中2-2）が、\ **トランザクション管理の観点で問題がない場合に限る。**\ 直接呼び出した場合に、トランザクション管理の観点で問題がある場合は、Serviceクラスにメソッドを用意し、適切なトランザクション管理が行われるようにすること。
       #. SharedServiceクラスから\ **Serviceクラスのメソッドを呼び出すことは禁止する（※図中2-3）。**\


| Serviceクラスと、SharedServiceクラスの依存関係を、以下に示す。
| 図中の番号は、上の表の「依存関係に関する注意点」欄の記載と連動しているため、あわせて確認すること。

 .. figure:: images/service_class-dependency.png
   :alt: class dependency
   :width: 100%
   :align: center


ServiceクラスとSharedServiceクラスを分ける理由について
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 業務ロジックを構成する処理の中には、再利用できない(すべきでない)ロジックと再利用できる（すべき）ロジックが存在する。
| この二つのロジックを、同じクラスのメソッドとして実装してしまうと、再利用してよいメソッドか否かの判断が、難しくなる。
| この問題を回避する目的として、本ガイドラインでは、\ **再利用されることを想定しているメソッドについては、SharedServiceクラスに実装することを強く推奨している。**\


Serviceクラスから、別のServiceクラスの呼び出しを禁止する理由について
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 本ガイドラインでは、Serviceクラスのメソッドから、別のServiceクラスのメソッドを呼び出すことを、原則禁止としている。
| これは、Serviceクラスは、特定のControllerに対して業務ロジックを提供するクラスであり、別のServiceから利用される前提で作成しないためである。
| 仮に、別のServiceクラスから直接呼び出してしまうと、以下のような状況が発生しやすくなり、\ **保守性などを低下させる危険性が、高まる。**\

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 発生しうる状況
   * - 1.
     - | 本来は、呼び出し元のServiceクラスで実装すべきロジックが、処理を一ヶ所にまとめたいという理由などにより、呼び出し先のServiceクラスで実装されてしまう。
       | その際に、\ **呼び出し元を意識するための引数（フラグ）などが、安易に追加され、間違った共通化が行われてしまう。結果として、見通しの悪いモジュール構成になってしまう。**\
   * - 2.
     - | 呼び出し経路やパターンが多くなることで、\ **仕様変更や、バグ改修の際のソース修正に対する影響範囲の把握が難しくなる。**\


メソッドのシグネチャを限定するようなインタフェースや基底クラスについて
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 業務ロジックの作りを統一したい場合に、シグネチャを限定するようなインタフェースや、基底クラスを作成することがある。
| シグネチャを限定するインタフェースや基底クラスを設けることで、開発者ごとに、作りの違いが発生しないようにする目的もある。

 .. note::

    大規模開発において、サービスイン後の保守性等を考慮して業務ロジックの作りを合わせておきたい場合や、開発者のひとりひとりのスキルがあまり高くない場合などの状況下では、
    シグネチャを限定するようなインタフェースを設けることも、選択肢の一つとして考えてもよい。

    本ガイドラインでは、シグネチャを限定するようなインタフェースを作成することは、特に推奨していないが、
    プロジェクトの特性を加味して、どのようなアーキテクチャにするか決めて頂きたい。

\


 .. note:: **シグネチャを制限するインタフェースおよび基底クラスの実装サンプル**
    - シグネチャを限定するようなインタフェース

     .. code-block:: java

        // (1)
        public interface BLogic<I, O> {
          O execute(I input);
        }

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90

        * - 項番
          - 説明
        * - | (1)
          - | 業務ロジックの実装メソッドのシグニチャを制限するためのインタフェース。
            | 上記例では、入力情報(I)と出力情報(O)の総称型として定義されており、 業務ロジックを実行するためのメソッド(execute)を一つもつ。
            | 本ガイドラインでは、上記のようなインタフェースを、BLogicインタフェースと呼ぶ。

    定型的な共通処理をServiceに盛り込む場合、ビジネスロジックの処理フローを統一したい場合に、メソッドのシグネチャを限定するような基底クラスを作成することがある。

    - シグネチャを限定するような基底クラス

     .. code-block:: java


        // (2)
        @Service
        @Transactional
        public abstract class AbstractBLogic<I, O> implements BLogic<I, O> {

            public O execute(I input){
              try{

                  // omitted

                  // (3)
                  preExecute(input);

                  // (4)
                  O output = doExecute(input);

                  // omitted

                  return output;
              } finally {
                  // omitted
              }

            }

            protected abstract void preExecute(I input);

            protected abstract O doExecute(I input);

        }


     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90

        * - 項番
          - 説明
        * - | (2)
          - | 基底クラスを作成する場合、\ `@Transactional`\ の仕様上、AOPの対象となるのは外部から実行されるメソッドもしくはメソッドを実装しているクラスであるため、トランザクション制御が必要な場合はこの基底クラスに付与する。
            | \ `@Servicve`\ も同様に、\ `ResultMessagesLoggingInterceptor`\ のようにAOPによってServiceを対象とするような場合はこの基底クラスに付与する必要がある。
        * - | (3)
          - | 基底クラスより、業務ロジックを実行する前の、事前処理を行うメソッドを呼び出す。
            | 上記のような事前処理を行うメソッドでは、ビジネスルールのチェックなどを実装することになる。
        * - | (4)
          - | 基底クラスより、業務ロジックを実行するメソッドを呼び出す。


    以下に、シグネチャを限定するような、基底クラスを継承する場合の、サンプルを示す。


    - BLogicクラス(Service)

     .. code-block:: java

        // (5)
        public interface XxxBLogic extends BLogic<XxxInput, XxxOutput> {

        }


     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90

        * - 項番
          - 説明
        * - | (5)
          - | タイプセーフなインジェクションを可能にするために、BLogicインタフェースを継承したインタフェースを作成する。
            | 親インタフェースのメソッド経由での呼び出しを行うために、BLogicを継承したサブインタフェースを実装する。


     .. code-block:: java

        @Service
        public class XxxBLogicImpl extends AbstractBLogic<XxxInput, XxxOutput> implements XxxBLogic {

            // (6)
            @Override
            protected void preExecute(XxxInput input) {

                // omitted
                Tour tour = tourRepository.findOne(input.getTourId());
                Date reservationLimitDate = tour.reservationLimitDate();
                if(input.getReservationDate().after(reservationLimitDate)){
                    throw new BusinessException(ResultMessages.error().add("e.xx.xx.0001"));
                }

            }

            // (7)
            @Override
            protected XxxOutput doExecute(XxxInput input) {
                TourReservation tourReservation = new TourReservation();

                // omitted

                tourReservationRepository.save(tourReservation);
                XxxOutput output = new XxxOutput();
                output.setTourReservation(tourReservation);

                // omitted
                return output;
            }

        }


     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90

        * - 項番
          - 説明
        * - | (6)
          - | 業務ロジックを実行する前の事前処理を実装する。
            | ビジネスルールのチェックなどを実装する事になる。
        * - | (7)
          - | 業務ロジックを実装する。
            | ビジネスルールを充たすために、ロジックを実装する事になる。

    - Controller

     .. code-block:: java

        // (8)
        @Inject
        XxxBLogic xxxBLogic;

        public String reserve(XxxForm form, RedirectAttributes redirectAttributes) {

            XxxInput input = new XxxInput();
            // omitted

            // (9)
            XxxOutput output = xxxBlogic.execute(input);

            // omitted

            redirectAttributes.addFlashAttribute(output.getTourReservation());
            return "redirect:/xxx?complete";
        }

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90

        * - 項番
          - 説明
        * - | (8)
          - | Controllerは、呼び出すBLogicインタフェースをInjectする。
        * - | (9)
          - | Controllerは、BLogicインタフェースのexecuteメソッドを呼び出し、業務ロジックを実行する。


.. _service-creation-unit-label:

Serviceの作成単位
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Serviceの作成単位は主に以下の３パターンとなる。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.25\linewidth}|p{0.50\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 15 25 50
   :class: longtable

   * - 項番
     - 単位
     - 作成方法
     - 特徴

   * - 1.
     - | Entity毎
     - | 主体となるEntityと対でServiceを作成する。
     - | 主体となるEntityとは、業務データの事であり、 **業務データを中心にしてアプリケーションを設計・実装する場合は、この単位でServiceを作成することを推奨する。**
       |
       | この単位でServiceを作成すると、業務データ毎に業務ロジックが集約されるため、業務処理の共通化が図られやすい。
       | ただし、このパターンでServiceを作成した場合、同時に大量の開発者を投入して作成するアプリケーションとの相性は、あまりよくない。どちらかと言うと、小規模・中規模のアプリケーションを開発する場合に向いているパターンと言える。
   * - 2.
     - | ユースケース毎
     - | ユースケースと対でServiceを作成する。
     - | **画面からのイベントを中心にしてアプリケーションを設計・実装する場合は、この単位でServiceを作成することになる。**
       |
       | この単位でServiceを作成する場合は、ユースケース毎に担当者を割り当てることが出来るため、同時に大量の開発者を投入して開発するアプリケーションとの相性はよい。
       | 一方で、このパターンでServiceを作成すると、ユースケース内での業務ロジックの共通化は行うことができるが、ユースケースを跨いだ業務ロジックの共通化は行われない可能性が高くなる。
       | ユースケースを跨いで業務ロジックの共通化を行う必要がある場合は、共通化を行うための共通チームを設けるなどの工夫が必要となる。
   * - 3
     - | イベント毎
     - | 画面から発生するイベントと対でServiceを作成する。
     - | **画面からのイベントを中心にしてアプリケーションを設計・実装する場合で且つ「TERASOLUNA ViSC」を使用してBLogicクラスを生成する場合は、この単位でServiceを作成することになる。**
       | 本ガイドラインでは、このような単位で作成されるServiceクラスの事を、BLogicと呼ぶ。
       |
       | この単位でServiceを作成する場合の特徴としては、基本的にはユースケース毎に作成する際と同じである。
       | ただし、イベント毎にServiceクラスを設計・実装する事になるため、ユースケース毎に作成する場合に比べて、より共通化が行われない可能性が高くなる。
       | 本ガイドラインとしては、イベント毎に作成するパターンは特に推奨しない。ただし、大規模開発において、保守性等を考慮して業務ロジックの作りを合わせておきたいといった理由がある場合は、イベント毎に作成する事を選択肢の一つとして考えてもよい。

 .. raw:: latex

    \newpage

 .. warning::

    **Serviceの作成単位については、開発するアプリケーションの特性や開発体制などを加味して決めて頂きたい。**

    また、提示した３つの作成パターンの **どれか一つのパターンに絞る必要はない。**
    無秩序にいろいろな単位のServiceを作成する事は避けるべきだが、 **アーキテクトによって方針が示されている状況下においては、併用しても特に問題はない。**
    例えば、以下のような組み合わせが考えられる。

    【組み合わせて使用する場合の例】

    * アプリケーションとして重要な業務ロジックについては、Entity毎のSharedServiceクラスとして作成する。
    * 画面からのイベントを処理するための業務ロジックについては、Controller毎のServiceクラスとして作成する。
    * Controller毎のServiceクラスでは、必要に応じてSharedServiceクラスのメソッドを呼び出す事で業務ロジックを実装する。

 .. tip::

     「TERASOLUNA ViSC」を使用する場合は、BLogicは設計書から出力される。

|

Entity毎にServiceを作成する際の開発イメージ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Entity毎にServiceを作成する場合は、以下のような開発イメージとなる。

 .. note::

    Entity毎にServiceを作成する代表的なアプリケーションの例としては、RESTアプリケーションがあげられる。
    RESTアプリケーションは、HTTP上に公開するリソースに対してCRUD操作(HTTPのPOST, GET, PUT, DELETE)を提供する事になる。
    HTTP上に公開するリソースは、業務データ(Entity)または業務データ(Entity)の一部となる事が多いため、Entity毎にServiceを作成する方法との相性がよい。

    RESTアプリケーションの場合は、ユースケースがEntity毎に抽出されることが多い。そのため、ユースケース毎に作成する際の構成イメージと似た構成となる。

|

 .. figure:: images/service_unit_resource.png
   :alt: multiple controller unit
   :width: 100%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | Entity毎に開発者を割り当てて、Serviceを実装する。
       | 特に理由がない場合は、ControllerもEntity毎に作成し、Serviceと同じ開発者を担当者にすることが望ましい。
   * - | (2)
     - | 複数の業務ロジックで共有したいロジックがある場合は、SharedServiceに実装する。
       | 上の図では、別の開発者(共通チームの担当者)を割り当てているが、プロジェクトの体制によっては(1)と同じ開発者でもよい。

|


ユースケース毎に作成する際の開発イメージ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| ユースケース毎にServiceを作成する場合は、以下のような開発イメージとなる。
| EntityのCRUD操作を行う様なユースケースの場合は、Entity毎にServiceを作成する際の構成イメージと同じ構成となる。


 .. figure:: images/service_unit_controller.png
   :alt: controller unit
   :width: 100%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | ユースケース毎に開発者を割り当てて、Serviceを実装する。
       | 特に理由がない場合は、Controllerもユースケース毎に作成し、Serviceと同じ開発者を担当者にすることが望ましい。
   * - | (2)
     - | 複数の業務ロジックで共有したいロジックがある場合は、SharedServiceに実装する。
       | 上の図では、別の開発者(共通チームの担当者)を割り当てているが、プロジェクトの体制によっては(1)と同じ開発者でもよい。

 .. note::

    ユースケースの規模が大きくなると、一人が担当する開発範囲が大きくなるため、作業分担しづらくなる。
    同時に大量の開発者を投入して開発するアプリケーションの場合は、ユースケースを更に分割して、担当者を割り当てる事を検討すること。

|

| ユースケースを更に分割した場合は、以下のような開発イメージとなる。
| ユースケースの分割を行うことで、SharedServiceに影響はないため、説明は割愛している。

 .. figure:: images/service_unit_controller2.png
   :alt: multiple controller unit
   :width: 100%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | ユースケースを構成する処理単位に分割し、処理毎に開発者を割り当てて、Serviceを実装する。
       | ここで言う処理とは、検索処理、登録処理、更新処理、削除処理といった単位であり、画面から発生するイベント毎の処理ではない点に注意すること。
       | 例えば「更新処理」であれば、「更新対象データの取得」や「更新内容の妥当性チェック」といった単位の処理が複数含まれる。
       | 特に理由がない場合は、Controllerも処理毎に作成し、Serviceと同じ開発者を担当者にすることが望ましい。

 .. tip::

    本ガイドライン上で使っている「ユースケース」と「処理」の事を、「ユースケースグループ」と「ユースケース」と呼ぶプロジェクトもある。

|

イベント毎に作成する際の開発イメージ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
イベント毎にService(BLogic)を作成する場合は、以下のような開発イメージとなる。

 .. figure:: images/service_unit_business-ligic.png
   :alt: constitution image of business logic unit
   :width: 100%
   :align: center

 .. raw:: latex

    \newpage

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90
   :class: longtable

   * - 項番
     - 説明
   * - | (1)
     - | イベント毎に開発者を割り当てて、Service(BLogic)を実装する。
       | 上記例ではそれぞれ別の担当者を割り当てる図になっているが、これは極端な例である。
       | 実際は、ユースケース毎に担当者を割り当てる事になる。
   * - | (2)
     - | 特に理由がない場合は、Controllerはユースケース毎に作成することが望ましい。
   * - | (3)
     - | イベント毎にService(BLogic)を実装する場合でも、担当者はユースケース毎に割り当てることを推奨する。
   * - | (4)
     - | 複数の業務ロジックで共有したいロジックがある場合は、SharedServiceに実装する。
       | 上の図では、別の開発者(共通チームの担当者)を割り当てているが、プロジェクトの体制によっては(1)と同じ開発者でもよい。

 .. raw:: latex

    \newpage

 .. note::

    ユースケースの規模が大きくなると、一人が担当する開発範囲が大きくなるため、作業分担しづらくなる。
    同時に大量の開発者を投入して開発するアプリケーションの場合は、ユースケースを更に分割して、担当者を割り当てる事を検討すること。

|

| ユースケースを更に分割した場合は、以下のような開発イメージとなる。
| ユースケースの分割を行うことで、SharedServiceに影響はないため、説明は割愛している。

 .. figure:: images/service_unit_business-ligic2.png
   :alt: multiple controller unit
   :width: 100%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | ユースケースを構成する処理単位に分割し、処理毎に開発者を割り当てて、Service(BLogic)を実装する。
       | ここで言う処理とは、検索処理、登録処理、更新処理、削除処理といった単位であり、画面から発生するイベント毎の処理ではない点に注意すること。
       | 例えば「更新処理」であれば、「更新対象データの取得」や「更新内容の妥当性チェック」といった単位の処理が複数含まれる。
       | 特に理由がない場合は、Controllerも処理毎に作成し、Serviceと同じ開発者を担当者にすることが望ましい。

.. _service-class-label:

Serviceクラスの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _service-class-creation-label:

Serviceクラスの作成方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Serviceクラスを作成する際の注意点を、以下に示す。

- Serviceインタフェースの作成

 .. code-block:: java

    public interface CartService { // (1)
        // omitted
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | **Serviceインタフェースを作成することを推奨する。**
       | インタフェースを設けることで、Serviceとして公開するメソッドを明確にすることが出来る。

\

 .. note:: **アーキテクチャ観点でのメリット例**

    #. AOPを使う場合に、JDK標準のDynamic proxies機能が使われる。
       インタフェースがない場合はSpring Frameworkに内包されているCGLIBが使われるが、finalメソッドに対してAdviceできないなどの制約がある。
       詳細は、\ `Spring Reference Document -Aspect Oriented Programming with Spring(Proxying mechanisms)- <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/aop.html#aop-proxying>`_\ を参照されたい。
    #. 業務ロジックをスタブ化しやすくなる。
       アプリケーション層とドメイン層を別々の体制で並行して開発する場合は、アプリケーション層を開発するために、Serviceのスタブが必要になるケースがある。
       スタブを作成する必要がある場合は、インタフェースを設けておくことを推奨する。

- Serviceクラスの作成

 .. code-block:: java

    @Service // (1)
    @Transactional // (2)
    public class CartServiceImpl implements CartService { // (3) (4)
        // omitted
    }

 .. code-block:: xml

    <context:component-scan base-package="xxx.yyy.zzz.domain" /> <!-- (1) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | **クラスに @Service アノテーションを付加する。**
       | アノテーションを付与することで、componentがscan対象となり、設定ファイルへのbean定義が、不要となる。
       | <context:component-scan>要素のbase-package属性に、componentをscanする対象のパッケージを指定する。
       | 上記設定の場合、「xxx.yyy.zzz.domain」パッケージ配下に格納されているクラスが、コンテナに登録される。
   * - | (2)
     - | **クラスに @Transactional アノテーションを付加する。**
       | アノテーションを付与することで、すべての業務ロジックに対してトランザクション境界が設定される。
       | 属性値については、要件に応じた値を指定すること。
       | 詳細は、\ :ref:`transaction-management-declare-transaction-info-label`\ を参照されたい。

       | また、\ ``@Transactional``\ アノテーションを使用する際の注意点を理解するために、「:ref:`DomainLayerAppendixTransactionManagement`」を合わせて確認するとよい。
   * - | (3)
     - | **インターフェース名はXxxService、クラス名はXxxServiceImplとする。**
       | 上記以外の命名規約でもよいが、ServiceクラスとSharedServiceクラスは、区別できる命名規約を設けることを推奨する。
   * - | (4)
     - | **Serviceクラスでは状態は保持せず、singletonスコープのbeanとしてコンテナに登録する 。**
       | フィールド変数には、スレッド毎に状態が変わるオブジェクト(Entity/DTO/VOなどのPOJO)や、値(プリミティブ型、プリミティブラッパークラスなど)を保持してはいけない。
       | また、\ ``@Scope``\ アノテーションを使ってsingleton以外のスコープ(prototype, request, session)にしてはいけない。

\

 .. note:: **クラスに @Transactional アノテーションを付加する理由**

    トランザクション境界の設定が必須なのは更新処理を含む業務ロジックのみだが、設定漏れによるバグを防ぐ事を目的として、クラスレベルにアノテーションを付与することを推奨している。
    もちろん必要な箇所（更新処理を行うメソッド）のみに、\ ``@Transactional``\ アノテーションを定義する方法を採用してもよい。

 .. note:: **singleton以外のスコープを禁止する理由**

    #. prototype, request, sessionは、状態を保持するbeanを登録するためのスコープであるため、Serviceクラスに対して使用すべきでない。
    #. スコープをrequestやprototypeにした場合、DIコンテナによるbeanの生成頻度が高くなるため、性能に影響を与えることがある。
    #. スコープをrequestやsessionにした場合、Webアプリケーション以外のアプリケーション(例えば、Batchアプリケーションなど)で使用できなくなる。

.. _service-class-method-creation-label:

Serviceクラスのメソッドの作成方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Serviceクラスのメソッドを作成する際の注意点を、以下に示す。

- Serviceインタフェースのメソッド作成

 .. code-block:: java

    public interface CartService {
        Cart createCart(); // (1) (2)
        Cart findCart(String cartId); // (1) (2)
    }

- Serviceクラスのメソッドの作成

 .. code-block:: java

    @Service
    @Transactional
    public class CartServiceImpl implements CartService {

        @Inject
        CartRepository cartRepository;

        public Cart createCart() { // (1) (2)
            Cart cart = new Cart();
            // ...
            cartRepository.save(cart);
            return cart;
        }

        @Transactional(readOnly = true) // (3)
        public Cart findCart(String cartId) { // (1) (2)
            Cart cart = cartRepository.findByCartId(cartId);
            // ...
            return cart;
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | **Serviceクラスのメソッドは、業務ロジック毎に作成する。**
   * - | (2)
     - | **業務ロジックは、Serviceインタフェースでメソッドの定義を行い、Serviceクラスのメソッドで実装を行う。**
   * - | (3)
     - | **業務ロジックのトランザクション定義をデフォルト（クラスアノテーションで指定した定義）から変更する場合は、@Transactionalアノテーションを付加する。**
       | 属性値については、要件に応じた値を指定すること。
       | 詳細は、\ :ref:`transaction-management-declare-transaction-info-label` を参照されたい。

       | また、\ ``@Transactional``\ アノテーションを使用する際の注意点を理解するために、「:ref:`DomainLayerAppendixTransactionManagement`」を合わせて確認するとよい。

\

 .. tip:: **参照系の業務ロジックのトランザクション定義について**

    参照系の業務ロジックを実装する場合は、\ ``@Transactional(readOnly = true)``\ を指定することで、
    JDBCドライバに対して「読み取り専用のトランザクション」のもとでSQLを実行するように指示することができる。

    読み取り専用のトランザクションの扱い方は、JDBCドライバの実装に依存するため、使用するJDBCドライバの仕様を確認されたい。


 .. note:: **「読み取り専用のトランザクション」を使用する際の注意点**

    コネクションプールからコネクションを取得する際にヘルスチェックを行う設定にしている場合、「読み取り専用のトランザクション」が有効にならないケースがある。
    本事象の詳細及び回避方法については、:ref:`「読み取り専用のトランザクション」が有効にならないケースについて <DomainLayerTransactionManagementWarningDisableCase>` を参照されたい。


 .. note:: **新しいトランザクションを開始する必要がある場合のトランザクション定義について**

    呼び出し元のメソッドが参加しているトランザクションには参加せず、
    新しいトランザクションを開始する必要がある場合は、\ ``@Transactional(propagation = Propagation.REQUIRES_NEW)``\ を設定する。

.. _service-class-method-args-return-label:

Serviceクラスのメソッド引数と返り値について
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Serviceクラスのメソッド引数と返り値は、以下の点を考慮すること。

| Serviceクラスの引数と返り値は、Serialize可能なクラス(\ ``java.io.Serializable``\ を実装しているクラス)とする。
| Serviceクラスは、分散アプリケーションとしてデプロイされる可能性もあるので、引数と返り値は、Serialize可能なクラスのみ、許可することを推奨する。

**メソッド引数/返り値となる代表的な型を以下に示す。**

 * プリミティブ型(\ ``int``\ , \ ``long``\ など)
 * プリミティブラッパークラス(\ ``java.lang.Integer``\ , \ ``java.lang.Long``\ など)
 * java標準クラス(\ ``java.lang.String``\ , \ ``java.util.Date``\ など)
 * ドメインオブジェクト(Entity、DTOなど)
 * 入出力オブジェクト(DTO)
 * 上記型のコレクション(\ ``java.util.Collection``\ の実装クラス)
 * void
 * etc ...

\

 .. note:: **入出力オブジェクトとは**

     #. 入力オブジェクトとは、Serviceのメソッドを実行するために必要な入力値をまとめたオブジェクトのことをさす。
     #. 出力オブジェクトとは、Serviceのメソッドの実行結果（出力値）をまとめたオブジェクトのことをさす。

      「TERASOLUNA ViSC」を使用して、業務ロジック(BLogicクラス)を生成する場合、BLogicの引数と返り値には、入出力オブジェクトを使用することになる。

**メソッド引数/返り値として禁止するものを以下に示す。**

 * アプリケーション層の実装アーキテクチャ(Servlet APIやSpringのweb層のAPIなど)に依存するオブジェクト(``javax.servlet.http.HttpServletRequest`` 、 ``javax.servlet.http.HttpServletResponse`` 、 ``javax.servlet.http.HttpSession`` 、 ``org.springframework.http.server.ServletServerHttpRequest`` など)
 * アプリケーション層のモデル(Form,DTOなど)
 * ``java.util.Map`` の実装クラス

 .. note:: **禁止する理由**

    #. アプリケーション層の実装アーキテクチャに依存するオブジェクトを許可してしまうと、アプリケーション層とドメイン層が密結合になってしまう。
    #. \ ``java.util.Map``\ は、インタフェースとして汎用性が高すぎるため、メソッドの引数や返り値に使うと、
       どのようなオブジェクが格納されているかわかりづらい。 また、値の管理がキー名で行われるため、以下の問題が発生しやすくなる。

     * 値を設定する処理と値を取得する処理で異なるキー名を指定してしまい、値が取得できない。
     * キー名の変更した場合の影響範囲の把握が困難になる。


**アプリケーション層とドメイン層で同じDTOを共有する場合の方針を、以下に示す。**

* ドメイン層のパッケージに属するDTOとして作成し、アプリケーション層で利用する。

\

 .. warning::

   アプリケーション層のFormやDTOを、ドメイン層で利用してはいけない。

.. _shared-service-class-label:

SharedServiceクラスの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _shared-service-class-creation-label:

SharedServiceクラスの作成方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| SharedServiceクラスを作成する際の注意点を、以下に示す。
| ここではServiceクラスと異なる箇所にフォーカスを当てて説明する。

#. | **必要に応じて、クラスに @Transactional アノテーションを付加する。**
   | データアクセスを伴わない場合は、\ ``@Transactional``\ アノテーションは不要である。

#. | **インターフェース名はXxxSharedService、クラス名はXxxSharedServiceImplとする。**
   | 上記以外の命名規約でもよいが、ServiceクラスとSharedServiceクラスは、区別できる命名規約を設けることを推奨する。

.. _shared-service-class-method-creation-label:

SharedServiceクラスのメソッドの作成方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| SharedServiceクラスのメソッドを作成する際の注意点を、以下に示す。
| ここでは、Serviceクラスと異なる箇所にフォーカスを当てて説明する。

#. **SharedServiceクラスのメソッドは、複数の業務ロジックで共有されるロジック毎に作成する。**

#. | **必要に応じて、クラスに @Transactional アノテーションを付加する。**
   | データアクセスを伴わない場合は、アノテーションは不要である。

.. _shared-service-class-method-args-return-label:

SharedServiceクラスのメソッド引数と返り値について
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\ :ref:`service-class-method-args-return-label`\ と同様の点を考慮すること。

.. _service-implementation-label:

処理の実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
ServiceおよびSharedServiceのメソッドで実装する処理について説明する。

ServiceおよびSharedServiceでは、アプリケーションで使用する業務データの取得、更新、整合性チェックおよびビジネスルールに関わる各種ロジックの実装を行う。

以下に、代表的な処理の実装例について説明する。

業務データを操作する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
業務データ(Entity)の取得、更新の実装例については、

* MyBatis3を使う場合は、\ :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessMyBatis3`\
* JPAを使う場合は、\ :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessJpa`\

を参照されたい。


.. _service-return-message-label:

メッセージを返却する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Serviceで解決すべきメッセージは、警告メッセージ、業務エラーメッセージの２つとなる(下図赤破線部参照)。
| それ以外のメッセージは、アプリケーション層で解決される。
| メッセージの種類とメッセージのパターンについては、\ :doc:`../ArchitectureInDetail/WebApplicationDetail/MessageManagement`\ を参照されたい。

 .. figure:: images/service_target-resolving-message.png
   :alt: target of resolving message
   :width: 100%
   :align: center

\

 .. note:: **メッセージの解決について**

    Serviceで解決するのは、メッセージ文言ではなく、\ **メッセージ文言を組み立てるために必要な情報（メッセージコード、メッセージ埋め込み値）の解決**\ であるという点を補足しておく。

詳細な実装方法は、

* :ref:`service-return-warnmessage-label`
* :ref:`service-return-businesserrormessage-label`

を参照されたい。

.. _service-return-warnmessage-label:

警告メッセージを返却する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 警告メッセージの返却は、戻り値としてメッセージオブジェクトを返却する。
| Entityなどのドメイン層のオブジェクトと一緒に返却する必要がある場合は、出力オブジェクト(DTO)にメッセージオブジェクトとドメインオブジェクトを詰めて返却する。

| 共通ライブラリとしてメッセージオブジェクト(\ ``org.terasoluna.gfw.common.message.ResultMessages``\ )を用意している。
| 共通ライブラリで用意しているクラスだと要件を満たせない場合は、プロジェクト毎にメッセージオブジェクトを作成すること。

- DTOの作成

 .. code-block:: java

    public class OrderResult implements Serializable {
        private ResultMessages warnMessages;
        private Order order;

        // omitted

    }

|

- Serviceクラスのメソッドの実装

 下記の例では、注文した商品の中に取り寄せ商品が含まれているため、分割配達となる可能性がある旨を警告メッセージとして表示する場合の実装例である。

 .. code-block:: java

    public OrderResult submitOrder(Order order) {

        // omitted

        boolean hasOrderProduct = orderRepository.existsByOrderProduct(order); // (1)

        // omitted

        Order order = orderRepository.save(order);

        // omitted

        ResultMessages warnMessages = null;
        // (2)
        if(hasOrderProduct) {
            warnMessages = ResultMessages.warn().add("w.xx.xx.0001");
        }
        // (3)
        OrderResult orderResult = new OrderResult();
        orderResult.setOrder(order);
        orderResult.setWarnMessages(warnMessages);
        return orderResult;
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 取り寄せ商品が含まれる場合は、\ ``hasOrderProduct``\ に\ ``true``\ が設定される。
   * - | (2)
     - | 上記例では、取り寄せ商品が含まれる場合に、警告メッセージを生成している。
   * - | (3)
     - | 上記例では、登録した\ ``Order``\ オブジェクトと警告メッセージを一緒に返却するために、\ ``OrderResult``\ というDTOにオブジェクトを格納して返却している。

.. _service-return-businesserrormessage-label:

業務エラーを通知する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 業務ロジック実行中に、ビジネスルールの違反が発生した場合はビジネス例外をスローする。
| 例えば次のような場合である。

-  旅行を予約する際に予約日が期限を過ぎている場合
-  商品を注文する際に在庫切れの場合
-  etc ...

| 共通ライブラリとしてビジネス例外(\ ``org.terasoluna.gfw.common.exception.BusinessException``\ )を用意している。
| 共通ライブラリで用意しているビジネス例外クラスだと要件を満たせない場合は、プロジェクト毎にビジネス例外クラスを作成すること。
| **ビジネス例外クラスは、java.lang.RuntimeException のサブクラスとして作成することを推奨する** 。

\

 .. note:: **ビジネス例外を非検査例外にする理由**

   ビジネス例外は、Controllerでハンドリングが必要になるため、本来は検査例外にした方がよい。
   しかし、本ガイドラインでは、設定漏れによるバグを防ぐ事を目的として、デフォルトでロールバックされる java.lang.RuntimeException のサブクラスとすることを推奨する。
   もちろん検査例外のサブクラスとしてビジネス例外を作成し、ビジネス例外クラスをロールバック対象として定義する方法を採用してもよい。

| ビジネス例外のスロー例を以下に示す。
| 下記の例では、予約期限日が過ぎていることを業務エラーとして通知する際の実装例である。

 .. code-block:: java

    // omitted

    if(currentDate.after(reservationLimitDate)) { // (1)
        throw new BusinessException(ResultMessages.error().add("e.xx.xx.0001"));
    }

    // omitted

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明

   * - | (1)
     - 旅行を予約する際に、予約日が期限を過ぎているので、ビジネス例外をスローしている。

例外ハンドリング全体の詳細は、\ :doc:`../ArchitectureInDetail/WebApplicationDetail/ExceptionHandling`\ を参照されたい。

.. _service-return-systemerrormessage-label:

システムエラーを通知する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 業務ロジック実行中に、システムとして異常な状態が発生した場合は、システム例外をスローする。
| 例えば、次のような場合である。

-  事前に存在しているはずのマスタデータ、ディレクトリ、ファイルなどが存在しない場合
-  利用しているライブラリのメソッドから発生する検査例外のうち、システム異常に分類される例外を補足した場合
-  etc ...

| 共通ライブラリとしてシステム例外(\ ``org.terasoluna.gfw.common.exception.SystemException``\ )を用意している。
| 共通ライブラリで用意しているシステム例外クラスだと要件を満たせない場合は、プロジェクト毎にシステム例外クラスを作成すること。
| **システム例外クラスは、java.lang.RuntimeException のサブクラスとして作成することを推奨する** 。
| 理由は、システム例外は、アプリケーションのコード上でハンドリングする必要がないという点と、\ ``@Transactinal``\ アノテーションのデフォルトのロールバック対象が、\ ``java.lang.RuntimeException``\ のためである。

| システム例外のスロー例を以下に示す。
| 下記の例では、指定された商品が、商品マスタに存在しないことを、システムエラーとして通知する際の実装例である。

 .. code-block:: java

    ItemMaster itemMaster = itemMasterRepository.findOne(itemCode);
    if(itemMaster == null) { // (1)
        throw new SystemException("e.xx.fw.0001",
            "Item master data is not found. item code is " + itemCode + ".");
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明

   * - | (1)
     - 事前に存在しているはずのマスタデータがないので、システム例外をスローしている。（ロジックで、システム異常を検知した場合の実装例）

下記の例では、ファイルコピー時のIOエラーをシステムエラーとして通知する際の実装例である。

 .. code-block:: java

    // ...

    try {
        FileUtils.copy(srcFile, destFile);
    } catch(IOException e) { // (1)
        throw new SystemException("e.xx.fw.0002",
            "Failed file copy. src file '" + srcFile + "' dest file '" + destFile + "'.", e);
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 利用しているライブラリのメソッドから、システム異常に分類される例外が発生したシステム例外をスローしている。
       | **利用しているライブラリから発生した例外は、原因例外としてシステム例外クラスに必ず渡すこと。**
       | 原因例外が失われると、スタックトレースよりエラー発生箇所および本質的なエラー原因が追えなくなってしまう。

\

 .. note:: **データアクセスエラーの扱いについて**

    業務ロジック実行中に、RepositoryやO/R Mapperでデータアクセスエラーが発生した場合、\ ``org.springframework.dao.DataAccessException``\ のサブクラスに変換されてスローされる。
    基本的には、業務ロジックではキャッチせず、アプリケーション層でエラーハンドリングすればよいが、
    一意制約違反などの一部のエラーについては、業務要件によっては、業務ロジックでハンドリングする必要がある。
    詳細は、\ :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessCommon`\ を参照されたい。

.. _service_transaction_management:

トランザクション管理について
--------------------------------------------------------------------------------
データの一貫性を保証する必要がある処理ではトランザクションの管理が必要となる。

トランザクション管理の方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
トランザクションの管理方法はいろいろあるが、本ガイドラインでは、\ **Spring Frameworkから提供されている「宣言型トランザクション管理」を利用することを推奨する。**\

宣言型トランザクション管理
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
「宣言型トランザクション管理」では、トランザクション管理に必要な情報を以下に２つの方法で宣言することができる。

* XML(bean定義ファイル)で宣言する。
* **アノテーション（@Transactional）で宣言する。（推奨）**

Spring Frameworkから提供されている「宣言型トランザクション管理」の詳細については、\ `Spring Reference Document -Transaction Management(Declarative transaction management)- <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/transaction.html#transaction-declarative>`_\ を参照されたい。
\

 .. note:: **「アノテーションで指定する」方法を推奨する理由**

    #. ソースコードを見ただけで、どのようなトランザクション管理が行われるかについて、把握することができる。
    #. XMLにトランザクション管理するためのAOPの設定が不要であり、XMLがシンプルになる。

.. _transaction-management-declare-transaction-info-label:

「宣言型トランザクション管理」で必要となる情報
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| トランザクション管理対象とするクラスまたはクラスメソッドに対して\ ``@Transactional``\ アノテーションを指定する。
| トランザクション制御に必要となる情報は、\ ``@Transactional``\ アノテーションの属性で指定する。

 .. note::

    本ガイドラインでは、Spring Frameworkから提供されている \ ``@org.springframework.transaction.annotation.Transactional``\ アノテーションを使用する前提である。

 .. tip::

    Spring 4からは、JTA 1.2から追加された \ ``@javax.transaction.Transactional``\ アノテーションを使用する事ができる。

    ただし、本ガイドラインでは、「宣言型トランザクション管理」で必要となる情報をより細かく指定できるSpring Frameworkのアノテーションを使用することを推奨する。

    Spring Frameworkのアノテーションを使用すると、

    * トランザクションの伝播方法(\ ``propagation``\ 属性)の属性値として\ ``NESTED``\(JDBCのセーブポイント)
    * トランザクションの独立レベル(\ ``isolation``\ 属性)
    * トランザクションのタイムアウト時間(\ ``timeout``\ 属性)
    * トランザクションの読み取り専用フラグ(\ ``readOnly``\ 属性)

    の指定が可能となる。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 10 80
    :class: longtable

    * - 項番
      - 属性名
      - 説明

    * - 1
      - propagation
      - | トランザクションの伝播方法を指定する。
        |
        | **[REQUIRED]**
        | トランザクションが開始されていなければ開始する。 (省略時のデフォルト)
        | **[REQUIRES_NEW]**
        | 常に、新しいトランザクションを開始する。
        | **[SUPPORTS]**
        | トランザクションが開始されていれば、それを利用する。開始されていなければ、利用しない。
        | **[NOT_SUPPORTED]**
        | トランザクションを利用しない。
        | **[MANDATORY]**
        | トランザクションが開始されている必要がある。開始されていなければ、例外が発生する。
        | **[NEVER]**
        | トランザクションを利用しない（開始されていてはいけない）。開始していれば、例外が発生する。
        | **[NESTED]**
        | セーブポイントが設定される。JDBCのみ有効である。
    * - 2
      - isolation
      - | トランザクションの独立レベルを指定する。
        | この設定は、DBの仕様に依存するため、使用するDBの仕様を確認し、設定値を決めること。
        |
        | **[DEFAULT]**
        | DBが提供するデフォルトの独立性レベル。(省略時のデフォルト)
        | **[READ_UNCOMMITTED]**
        | 他のトランザクションで変更中（未コミット）のデータが読める。
        | **[READ_COMMITTED]**
        | 他のトランザクションで変更中（未コミット）のデータは読めない。
        | **[REPEATABLE_READ]**
        | 他のトランザクションが読み出したデータは更新できない。
        | **[SERIALIZABLE]**
        | トランザクションを完全に独立させる。
        |
        | トランザクションの独立レベルは、排他制御に関連するパラメータとなる。
        | 排他制御については、\ :doc:`../ArchitectureInDetail/DataAccessDetail/ExclusionControl`\ を参照されたい。
    * - 3
      - timeout
      - | トランザクションのタイムアウト時間(秒)を指定する。
        | デフォルトは-1(使用するDBの仕様や設定に依存)
    * - 4
      - readOnly
      - | トランザクションの読み取り専用フラグを指定する。
        | デフォルトはfalse(読み取り専用でない)
    * - 5
      - rollbackFor
      - | トランザクションのロールバック対象とする例外クラスのリストを指定する。
        | デフォルトは空（指定なし）
    * - 6
      - rollbackForClassName
      - | トランザクションのロールバック対象とする例外クラス名のリストを指定する。
        | デフォルトは空（指定なし）
    * - 7
      - noRollbackFor
      - | トランザクションのコミット対象とする例外クラスのリストを指定する。
        | デフォルトは空（指定なし）
    * - 8
      - noRollbackForClassName
      - | トランザクションのコミット対象とする例外クラス名のリストを指定する。
        | デフォルトは空（指定なし）

 .. raw:: latex

    \newpage

\

 .. note:: **@Transactionalアノテーションを指定する場所**

    **クラスまたはクラスのメソッドに指定することを推奨する。**
    インタフェースまたはインタフェースのメソッドでない点が、ポイント。
    理由は、\ `Spring Reference Document -Transaction Management(Using @Transactional)- <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/transaction.html#transaction-declarative-annotations>`_\ の2個めのTipsを参照されたい。

 .. warning:: **例外発生時のrollbackとcommitのデフォルト動作**

    rollbackForおよびnoRollbackForを指定しない場合、Spring Frameworkは、以下の動作となる。

    * 非検査例外クラス（java.lang.RuntimeExceptionおよびjava.lang.Error）またはそのサブクラスの例外が発生した場合は、rollbackする。
    * 検査例外クラス（java.lang.Exception）またはそのサブクラスの例外が発生した場合は、commitする。\ **(注意が必要)**\

 .. note:: **@Transactionalアノテーションのvalue属性について**

    \ ``@Transactional``\ アノテーションにはvalue属性があるが、これは複数のTransaction Managerを宣言した際に、どのTransaction Managerを使うのかを指定する属性である。
    Transaction Managerが一つの場合は指定は不要である。
    複数のTransaction Managerを使う必要がある場合は、\ `Spring Reference Document -Transaction Management(Multiple Transaction Managers with @Transactional)- <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/transaction.html#tx-multiple-tx-mgrs-with-attransactional>`_\ を参照されたい。

 .. note:: **主要DBのisolationのデフォルトについて**

    主要DBのデフォルトの独立性レベルは、以下の通りである。

    * Oracle : READ_COMMITTED
    * DB2 : READ_COMMITTED
    * PostgreSQL : READ_COMMITTED
    * SQL Server : READ_COMMITTED
    * MySQL : REPEATABLE_READ

.. _DomainLayerTransactionManagementWarningDisableCase:

 .. note:: **「読み取り専用のトランザクション」が有効にならないケースについて**

    \ ``readOnly = true``\ を指定することで「読み取り専用のトランザクション」のもとでSQLを実行する仕組みが提供されているが、
    以下の条件にすべて一致する場合、「読み取り専用のトランザクション」が有効にならないJDBCドライバが存在する。

    **[本事象の発生条件]**

    * コネクションプールからコネクションを取得する際に、ヘルスチェックを行う。
    * コネクションプールから取得したコネクションの自動コミットを無効にする。
    * \ ``PlatformTransactionManager``\ として、\ ``DataSourceTransactionManager``\ 又は\ ``JpaTransactionManager``\ を使用する。(\ ``JtaTransactionManager``\ を使用する場合は本事象は発生しない)

    **[本事象の発生が確認されているJDBCドライバ]**

    * ``org.postgresql:postgresql:9.3-1102-jdbc41`` (PostgreSQL 9.3向けJDBC4.1互換のJDBCドライバ)

    **[本事象の回避方法]**

    「読み取り専用のトランザクション」が有効にならないケースに一致する場合は、
    \ ``readOnly = true``\ を指定すると無駄な処理が行われる事になるため、
    参照系の処理についても「更新可能なトランザクション」のもとで実行することを推奨する。

    他の回避方法として、

    * コネクションプールからコネクションを取得する際に、ヘルスチェックを行わない。
    * コネクションプールから取得したコネクションの自動コミットを有効にする。(トランザクション管理が必要な時のみ自動コミットを無効にする)

    という方法もあるが、本事象を回避するために、ヘルスチェックや自動コミットに対する設計を変更する事は避けるべきである。

    **[備考]**

    * 本事象の再現確認は、PostgreSQL 9.3及びOracle 12cで行っており、他のデータベース及びバージョンでは行っていない。
    * PostgreSQL 9.3では、\ ``java.sql.Connection#setReadOnly(boolean)``\  メソッドを呼び出した際に\ ``SQLException``\ が発生する。
    * \ :ref:`log4jdbc <DataAccessCommonDataSourceDebug>`\ を使用してSQLやJDBCのAPIの呼び出しをロギングしている場合、JDBCドライバから発生した\ ``SQLException``\ はERRORレベルでログに出力される。
    * **JDBCドライバから発生するSQLExceptionはSpring Frameworkが行う例外処理によって無視されるため、アプリケーションの動作としてはエラーにはならないが、「読み取り専用のトランザクション」は有効にならない。**
    * Oracle 12cでは、本事象の発生は確認されていない。

    **[参考]**

    \ :ref:`log4jdbc <DataAccessCommonDataSourceDebug>`\ を使用して以下のようなログが出力された場合は、本事象に該当するケースとなる。

     .. code-block:: text

        date:2015-02-20 16:11:56	thread:main	user:	X-Track:	level:ERROR	logger:jdbc.audit                                      	message:3. Connection.setReadOnly(true)
        org.postgresql.util.PSQLException: Cannot change transaction read-only property in the middle of a transaction.
            at org.postgresql.jdbc2.AbstractJdbc2Connection.setReadOnly(AbstractJdbc2Connection.java:741) ~[postgresql-9.3-1102-jdbc41.jar:na]
            ...

 .. note:: **@Transactionalアノテーションのtimeout属性について**

    \ ``@Transactional``\ アノテーションには\ ``timeout``\属性があるが、MyBatis 3.3とMyBatis-Spring 1.2の組み合わせでは
    \ ``timeout``\属性に指定した値は無視され、使用されない。


トランザクションの伝播
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| トランザクションの伝播方法は、ほとんどの場合は「REQUIRED」でよい。
| ただし、 **アプリケーションの要件によっては「REQUIRES_NEW」を使うこともある** ので、「REQUIRED」と「REQUIRES_NEW」を指定した場合のトランザクション制御フローを、以下に示す。
| 他の伝播方法の使用頻度は低いと思われるので、本ガイドラインでの説明は省略する。

| **トランザクションの伝播方法を「REQUIRED」にした場合のトランザクション管理フロー**
| トランザクションの伝播方法を「REQUIRED」にした場合、Controllerから呼び出された一連の処理が、すべて同じトランザクション内で処理される。

 .. figure:: images/service_transaction-propagation-required.png
    :alt: transaction management flow of REQUIRED
    :width: 100%
    :align: center

#. Controllerからトランザクション管理対象のServiceのメソッドを呼び出す。
   この時点で開始されているトランザクションは存在しないため、\ ``TransactionInterceptor``\ によってトランザクションが開始される。
#. \ ``TransactionInterceptor``\ は、トランザクション開始した後に、トランザクション管理対象のメソッドを呼び出す。
#. Serviceからトランザクション管理対象の\ ``SharedService``\ のメソッドを呼び出す。
   この時点で開始済みのトランザクションが存在しているため、\ ``TransactionInterceptor``\ は、新たにトランザクションは開始せず、開始済みのトランザクションに参加する。
#. \ ``TransactionInterceptor``\ は、開始済みのトランザクションに参加した後に、トランザクション管理対象のメソッドを呼び出す。
#. \ ``TransactionInterceptor``\ は、処理結果に応じてコミットまたはロールバックを行い、トランザクションを終了する。


.. note:: **org.springframework.transaction.UnexpectedRollbackExceptionが発生する理由**

  トランザクションの伝播方法を「REQUIRED」にした場合、物理的なトランザクションは一つだが、Spring Frameworkでは内部的なトランザクション制御境界が設けられている。
  上記例だと、SharedServiceが呼び出された際に実行される\ ``TransactionInterceptor``\ が、内部的なトランザクション制御を行っている。
  そのため、\ ``SharedService``\ でロールバック対象の例外が発生した場合、\ ``TransactionInterceptor``\ によって、
  トランザクションはロールバック状態（rollback-only）に設定され、トランザクションをコミットすることはできなくなる。
  この状態でトランザクションのコミットを行おうとすると、Spring Frameworkは、\ ``UnexpectedRollbackException``\ を発生させ、トランザクション制御に矛盾が発生している事を通知してくれる。
  \ ``UnexpectedRollbackException``\ が発生した場合、rollbackForおよびnoRollbackForの定義に、矛盾がないか、確認すること。

| **トランザクションの伝播方法を「REQUIRES_NEW」にした場合のトランザクション管理フロー**
| トランザクションの伝播方法を「REQUIRES_NEW」にした場合、Controllerから呼び出された時に行われる一連の処理の一部（SharedServiceで行っている処理）が別のトランザクションで処理される。

 .. figure:: images/service_transaction-propagation-requires_new.png
    :alt: transaction management flow of REQUIRES_NEW
    :width: 100%
    :align: center

#. Controllerからトランザクション管理対象のServiceのメソッドを呼び出す。この時点で開始されているトランザクションは存在しないため、 ``TransactionInterceptor`` によってトランザクションが開始される(ここで開始したトランザクションを以降「Transaction A」と呼ぶ)。
#. ``TransactionInterceptor`` は、トランザクション（Transaction A）を開始した後に、トランザクション管理対象のメソッドを呼び出す。
#. Serviceからトランザクション管理対象の ``SharedService`` のメソッドを呼び出す。この時点で開始済みのトランザクション（Transaction A）が存在しているが、トランザクションの伝播方法が「REQUIRES_NEW」なので ``TransactionInterceptor`` によって新しいトランザクションが開始される(ここで開始したトランザクションを以降「Transaction B」と呼ぶ)。この時点で「Transaction A」のトランザクションは、中断され再開待ちの状態となる。
#. \ ``TransactionInterceptor``\ は、トランザクション（Transaction B）を開始した後に、トランザクション管理対象のメソッドを呼び出す。
#. \ ``TransactionInterceptor``\ は、処理結果に応じてコミットまたはロールバックを行い、トランザクション（Transaction B）を終了する。
   この時点で、「Transaction A」のトランザクションが再開され、アクティブな状態になる。
#. \ ``TransactionInterceptor``\ は、処理結果に応じてコミットまたはロールバックを行い、トランザクション（Transaction A）を終了する。

トランザクション管理対象となるメソッドの呼び出し方
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Spring Frameworkから提供されている「宣言型トランザクション管理」はAOPで実現されているため、AOPが有効となるメソッド呼び出しに対してのみ、トランザクション管理が適用される。
| デフォルトのAOPモードが、\ **proxyモードなので、別のクラスからpublicメソッドが呼び出された場合のみトランザクション管理対象となる。**\
| \ **publicメソッドであっても、内部呼び出しの場合は、トランザクション管理対象にならない**\ ので注意が必要となる。

- **トランザクション管理対象となるメソッドの呼び出し方**

 .. figure:: images/service_transaction-valid-call.png
   :alt: enabled method calls of transaction management
   :width: 100%
   :align: center

- **トランザクション管理対象にならないメソッドの呼び出し方**

 .. figure:: images/service_transaction-invalid-call.png
   :alt: not enabled method calls of transaction management
   :width: 100%
   :align: center

 .. note:: **内部呼び出しをトランザクション管理対象にしたい場合**

   AOPモードを\ ``"aspectj"``\ にすることで、内部呼び出しをトランザクション管理対象にすることができる。
   ただし、内部呼び出しもトランザクション管理対象にしてしまうと、トランザクション管理の経路が複雑になる可能性があるので、
   基本的にはAOPモードはデフォルトの\ ``"proxy"``\ を使用することを推奨する。

.. _service_enable_transaction_management:
.. _DomainLayerAppendixTransactionManagement:

トランザクション管理を使うための設定について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

トランザクション管理を使うために必要な設定について説明する。

PlatformTransactionManagerの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| トランザクション管理を行う場合、\ ``PlatformTransactionManager``\ のbeanを設定する必要がある。
| Spring Frameworkより用途毎のクラスが提供されているので、使用するクラスを指定すればよい。

- :file:`xxx-env.xml`

 以下に、DataSourceから取得されるJDBCコネクションの機能を使って、トランザクションを管理する場合の設定例を示す。

 .. code-block:: xml

     <!-- (1) -->
     <bean id="transactionManager"
           class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
         <property name="dataSource" ref="dataSource" />
         <property name="rollbackOnCommitFailure" value="true" />
     </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明

    * - | (1)
      - | 用途にあった\ ``PlatformTransactionManager``\ の実装クラスを指定する。
        | idは「transactionManager」としておくことを推奨する。

\

 .. note:: **複数DB（複数リソース）に対するトランザクション管理（グローバルトランザクションの管理）が必要な場合**

     * \ ``org.springframework.transaction.jta.JtaTransactionManager``\ を利用し、アプリケーションサーバから提供されているJTAの機能を使って、トランザクション管理を行う必要がある。
     * WebSphere、Oracle WebLogic ServerでJTAを使う場合、<tx:jta-transaction-manager/> を指定することで、
       アプリケーションサーバ用に拡張された\ ``JtaTransactionManager``\ が、自動的で設定される。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table:: **Spring Frameworkから提供されているPlatformTransactionManagerの実装クラス**
    :header-rows: 1
    :widths: 10 35 55

    * - 項番
      - クラス名
      - 説明
    * - 1.
      - | org.springframework.jdbc.datasource.
        | DataSourceTransactionManager
      - | JDBC(\ ``java.sql.Connection``\ )のAPIを呼び出して、トランザクションを管理するための実装クラス。
        | MyBatisや、\ ``JdbcTemplate``\ を使う場合は、本クラスを使用する。
    * - 2.
      - | org.springframework.orm.jpa.
        | JpaTransactionManager
      - | JPA(\ ``javax.persistence.EntityTransaction``\ )のAPIを呼び出して、トランザクションを管理するための実装クラス。
        | JPAを使う場合は、本クラスを使用する。
    * - 3.
      - | org.springframework.transaction.jta.
        | JtaTransactionManager
      - | JTA(\ ``javax.transaction.UserTransaction``\ )のAPIを呼び出してトランザクションを管理するための実装クラス。
        | アプリケーションサーバから提供されているJTS(Java Transaction Service)を利用して、リソース(データベース/メッセージングサービス/汎用EIS(Enterprise Information System)など)とのトランザクションを管理する場合は、本クラスを使用する。
        | 複数のリソースに対する操作を同一トランザクションで行う必要がある場合は、JTAを利用して、リソースとのトランザクションを管理する必要がある。

@Transactionalを有効化するための設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 本ガイドラインでは、\ ``@Transactional``\ アノテーションを使った「宣言型トランザクション管理」を使って、トランザクション管理することを推奨している。
| ここでは、\ ``@Transactional``\ アノテーションを使うために、必要な設定について説明する。

- :file:`xxx-domain.xml`

 .. code-block:: xml

     <tx:annotation-driven /> <!-- (1) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明

    * - | (1)
      - <tx:annotation-driven>要素をXML（bean定義ファイル）に追加することで、\ ``@Transactional``\ アノテーションを使ったトランザクション境界の指定が有効となる。

 .. note:: **トランザクション管理の落とし穴について**

    IBM DeveloperWorksに「トランザクションの落とし穴を理解する」という記事がある。
    この記事ではトランザクション管理で注意しなくてはいけないことや、Spring Frameworkの@Transactionalを使う場合の注意点がまとめられているので、ぜひ一読してほしい。
    詳細は、\ `IBM DeveloperWorksの記事 <http://www.ibm.com/developerworks/java/library/j-ts1/index.html>`_\ を参照されたい。

    ※IBM DeveloperWorksの記事は2009年の記事のため(古いため)、一部の内容がSpring Framework 4.1使用時の動作と異なる部分がある。

    具体的には、「Listing 7. Using read-only with REQUIRED propagation mode — JPA」の内容である。

    Spring Framework 4.1より、JPAのプロバイダとしてHibernate ORM 4.2以上を使用している場合は、
    JDBCドライバに対して「読み取り専用のトランザクション」のもとでSQLを実行するように指示することが出来るように改善(\ `SPR-8959 <https://jira.spring.io/browse/SPR-8959>`_\ )されている。

    読み取り専用のトランザクションの扱い方は、JDBCドライバの実装に依存するため、使用するJDBCドライバの仕様を確認されたい。


 .. note:: **プログラマティックにトランザクションを管理する方法**

    本ガイドラインでは、「宣言型トランザクション管理」を推奨しているが、プログラマティックにトランザクションを管理することもできる。
    詳細については、\ `Spring Reference Document -Transaction Management(Programmatic transaction management)- <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/transaction.html#transaction-programmatic>`_\ を参照されたい。


<tx:annotation-driven>要素の属性について
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

<tx:annotation-driven>にはいくつかの属性が指定でき、デフォルトの振る舞いを拡張することができる。

- :file:`xxx-domain.xml`

 .. code-block:: xml

     <tx:annotation-driven
          transaction-manager="txManager"
          mode="aspectj"
          proxy-target-class="true"
          order="0" />

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 15 75

    * - 項番
      - 属性
      - 説明

    * - 1
      - transaction-manager
      - \ ``PlatformTransactionManager``\ のbeanを指定する。省略した場合「transactionManager」というbean名で登録されているbeanが使用される。

    * - 2
      - mode
      - AOPのモードを指定する。省略した場合、\ ``"proxy"``\ となる。\ ``"aspectj"``\ を指定できるが、原則デフォルトの\ ``"proxy"``\ を使う。

    * - 3
      - proxy-target-class
      - proxyのターゲットをクラスに限定するかを指定するフラグ（mode="proxy"の場合のみ、有効な設定）。省略した場合「false」となる。

        * false の場合、対象がインタフェースを実装している場合は、JDK標準のDynamic proxies機能によってproxyされ、
          インタフェースを実装していない場合はSpring Frameworkに内包されているGCLIBの機能によってproxyされる。
        * true の場合、インタフェースの実装有無に関係なく、GCLIBの機能によってproxyされる。

    * - 4
      - order
      - AOPでAdviceされる順番（優先度）を指定する。省略した場合「最後（もっとも低い優先度）」となる。

|

Tips
--------------------------------------------------------------------------------

.. _tips_business_error-label:

ビジネスルールの違反をフィールドエラーとして扱う方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| ビジネスルールのエラーをフィールド毎に出力する必要がある場合、Controller側(Bean ValidationまたはSpring Validator)の仕組みを利用する必要がある。
| このケースの場合、チェックロジック自体はServiceとして実装し、Bean ValidationまたはSpring ValidatorからServiceのメソッドを呼び出す方式で実現することを推奨する。
| 詳細は、\ :doc:`../ArchitectureInDetail/WebApplicationDetail/Validation`\ の業務ロジックチェックを参照されたい。

.. raw:: latex

   \newpage

