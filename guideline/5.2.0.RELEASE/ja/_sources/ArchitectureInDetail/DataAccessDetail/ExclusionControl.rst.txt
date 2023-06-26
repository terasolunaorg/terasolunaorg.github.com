排他制御
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

|

Overview
--------------------------------------------------------------------------------
排他制御とは、複数のトランザクションから同じデータに対して、同時に更新処理が行われる際に、データの整合性を保つために行う処理のことである。

複数のトランザクションから同じデータに対して、同時に更新処理が行われる可能性がある場合は、基本的に排他制御を行う必要がある。
ここで言うトランザクションとは、かならずしもデータベースとのトランザクションとは限らず、ロングトランザクションも含まれる。

.. note:: **ロングトランザクションとは**

    データの取得とデータの更新を、別々のデータベーストランザクションとして行う際に発生するトランザクションのことである。

    具体例としては、取得したデータを編集画面に表示し、画面で編集した値をデータベースに更新するようなアプリケーションで発生する。

| 本節では、データベース上で管理されているデータに対する排他制御について、説明する。
| しかし、データベース以外で管理されているデータ(例えば、メモリ、ファイルなど)についても、同様に排他制御を行う必要があることに留意すること。

.. _ExclusionControl-Necessity:

排他制御の必要性
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
まず、排他制御の必要性を理解してもらうために、排他制御を行わなかった際に発生する問題について、具体例を3つ挙げて説明する。

Problem1
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ここでは、ショッピングサイトにて、ユーザからTeaの注文を受け付ける場合の例を示す。

 .. figure:: ./images/ExclusionControl-problem1.png
   :alt: Exclusive Control problem1
   :width: 90%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.05\linewidth}|p{0.05\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 5 5 80

    * - 項番
      - UserA
      - UserB
      - 説明
    * - 1.
      - 〇
      - \-
      - User Aが、商品画面にてTeaの在庫が5個あることを確認する。
    * - 2.
      - \-
      - 〇
      - User Bが、商品画面にてTeaの在庫が5個あることを確認する。
    * - 3.
      - \-
      - 〇
      - User BがTeaを5個注文する。DB上のTeaの在庫を-5し、Teaの在庫は0になる。
    * - 4.
      - 〇
      - \-
      - User AがTeaを5個注文する。DB上のTeaの在庫を-5し、Teaの在庫は-5となる。

 | **User Aの注文は受け付けられたが、実際の在庫が無いため、謝りの連絡を入れることになる。**
 | **テーブルで管理しているTeaの在庫数についても、実際のTeaの在庫数と異なる値(マイナス値)になってしまう。**

Problem2
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ここでは、ショッピングサイトでTeaの在庫数を管理するスタッフが、Teaの在庫数を表示し、仕入れたTeaの数をクライアントで計算して、Teaの在庫数を更新する場合の例を示す。

 .. figure:: ./images/ExclusionControl-problem2.png
   :alt: Exclusive Control problem2
   :width: 90%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.05\linewidth}|p{0.05\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 5 5 80

    * - 項番
      - UserA
      - UserB
      - 説明
    * - 1.
      - 〇
      - \-
      - Staff AがTeaの在庫が5個あることを確認する。
    * - 2.
      - \-
      - 〇
      - Staff BがTeaの在庫が5個あることを確認する。
    * - 3.
      - \-
      - 〇
      - Staff BがTeaを10個仕入れ、在庫数をクライアントで5＋10=15個と計算して更新する。
    * - 4.
      - 〇
      - \-
      - Staff AがTeaを20個仕入れ、在庫数をクライアントで5＋20=25個と計算して更新する。

 **3の処理で追加した10個の仕入れが無くなってしまい、実際の在庫数(35個)と合わなくなってしまう。**

Problem3
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ここでは、バッチ処理によってロックされているデータに対して、オンライン処理で更新する例を示す。

 .. figure:: ./images/ExclusionControl-problem4.png
   :alt: Exclusive Control problem4
   :width: 90%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.05\linewidth}|p{0.05\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 5 5 80

    * - 項番
      - UserA
      - Batch
      - 説明
    * - 1.
      - \-
      - 〇
      - Batchがテーブルの更新対象の該当行（ここでは仮に全ての行とする。）をロックし、他の処理で更新できないようにする。
    * - 2.
      - 〇
      - \-
      - User Aが更新情報を検索する。この時点でBatchはコミットされていないため、Batch更新前の情報が取得できる。
    * - 3.
      - 〇
      - \-
      - User Aが更新要求をするが、Batchにロックされているため、更新が待たされる。
    * - 4.
      - \-
      - 〇
      - Batchが処理を終えてロックを解放する。
    * - 5.
      - 〇
      - \-
      - User Aの待たされていた更新処理が、実行可能となり更新処理を実行する。

 | **User AはBatch終了を待たされた後に、更新処理を実行する。しかし、User Aの取得した元のデータは、Batchの更新前のデータであり、Batchで更新した情報を上書く可能性がある。**
 | **また、Batch時間はオンライン処理と比べると長いものが多く、ユーザが待たされる時間が長くなる。**

トランザクションの分離レベルによる排他制御
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| :ref:`ExclusionControl-Necessity` で挙げた3つの問題をすべて解決するための最も簡単な方法は、データベースへの処理を一つひとつ順番に（シリアルに）実行されるようにすることである。
| このようにシリアルに処理させることで、トランザクションが互いに影響を及ぼし合わなくなる。
| しかしながら、シリアルに処理させる場合、単位時間内に実行可能なトランザクション数が減少するため、パフォーマンスが低下することになる。

ANSI/ISO SQL標準では、トランザクションの分離レベル（各トランザクションがそれぞれどの程度互いに影響を及ぼし合うか）を表す指標を定義している。
以下に、トランザクションの分離レベルを4つ示す。併せて、各分離レベルで起こりうる現象について説明する。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 20 20 20 20

   * - | 項番
     - | 分離レベル
     - | ダーティ・リード
       | DRITY READ
     - | 再読込不可能読取
       | NON-REPEATABLE READ
     - | ファントム・リード
       | PHANTOM READ
   * - | 1.
     - | 未コミット読込
       | READ UNCOMMITTED
     - 有
     - 有
     - 有
   * - | 2.
     - | コミット済読込
       | READ COMMITTED
     - 無
     - 有
     - 有
   * - | 3.
     - | 再読込可能読取
       | REPEATABLE READ
     - 無
     - 無
     - 有
   * - | 4.
     - | 直列化
       | SERIALIZABLE
     - 無
     - 無
     - 無

.. tip:: **ダーティ・リード（DRITY READ）**

     まだコミットされていないトランザクションが書き込んだデータを、別のトランザクションが読み込む現象のことである。

.. tip:: **再読込不可能読取（NON-REPEATABLE READ）**

     同一トランザクション内で同じレコードを2度読み込むような場合、1度目と2度目の読み込みの間に他トランザクションがコミットすると、1度目に読み込んだ内容と2度目に読み込んだ内容が異なる可能性がある。
     複数回の読み込みの結果が、他のトランザクションのコミットのタイミングによって変わることである。

.. tip:: **ファントム・リード（PHANTOM READ）**

     同一トランザクション内で、同じレコードを2回読み込む間に、他のトランザクションがレコードを追加、または削除することにより、2回目の読み込みで1回目と取得レコード数（内容）が異なることである。

| 上記の表に定義されている分離レベルは、下にいくほどトランザクションの分離レベルが高くなる。
| 分離レベルが高ければ、データは安全に守られるが、ロック待ちが多くなり、パフォーマンスが低下する。
| SERIALIZABLEは、アクセス頻度がかなり低い場合を除き、選択すべきでない。
| その理由は、SELECTを含め、すべてのデータアクセスが、一つずつ順番に行われるためである。

| トランザクション間の分離性と同時実行性は、トレードオフの関係である。
| すなわち、分離レベルを高くすれば同時実効性が下がり、分離レベルを下げると、同時実効性が上がる。
| そのため、アプリケーションの要件に合わせて、トランザクションの分離性と同時実行性のバランスをとる必要がある。

| 使用するデータベースにより、サポートされている分離レベルは違うため、使用するデータベースの特性を理解する必要がある。
| 以下に、データベース毎でサポートされている分離レベルと、デフォルト値を示す。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 20 15 15 15 15

   * - | 項番
     - | データベース
     - | READ UNCOMMITTED
     - | READ COMMITTED
     - | REPEATABLE READ
     - | SERIALIZABLE
   * - | 1.
     - | Oracle
     - | ×
     - | 〇(default)
     - | ×
     - | 〇
   * - | 2.
     - | PostgreSQL
     - | ×
     - | 〇(default)
     - | ×
     - | 〇
   * - | 3.
     - | DB2
     - | 〇
     - | 〇(default)
     - | 〇
     - | 〇
   * - | 4.
     - | MySQL InnoDB
     - | 〇
     - | 〇
     - | 〇(default)
     - | 〇

**データの整合性を保ちつつ、分離性と同時実行性のバランスをとる場合、データベースのロック機能を使用して排他制御を行う必要がある。以下に、データベースのロック機能について説明する。**

データベースのロック機能による排他制御
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| 更新対象のデータを適切な方法でロックする必要がある。その理由は、下記2点の通りである。

* データベース上で管理されているデータの整合性を保つため
* 更新処理が競合しないようにするため

| データベース上で管理されているデータをロックする方法は、下記の通り3種類ある。
| アーキテクトは、これらのロックの特徴を十分に理解した上で、アプリケーションの特性にあったロックの方法を採用すること。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.40\linewidth}|p{0.35\linewidth}|
 .. list-table:: ロックの種類
   :header-rows: 1
   :widths: 10 15 40 35

   * - 項番
     - ロック種類
     - 適用ケース
     - 特徴
   * - 1.
     - RDBMSによる自動的なロック
     - * データの更新条件として、データの整合性を保証するために必要な条件を指定できる場合。
       * 同一データに対する同時実行数が少なく、更新処理も短い時間でおわる場合。
     - * チェックと更新処理を一つのSQLで実行するため、効率的である。
       * 楽観ロックに比べ、データの整合性を保証するための条件を個別に検討する必要がある。
   * - 2.
     - 楽観ロック
     - * 事前に取得したデータが他のトランザクションによって更新されていた場合に、更新内容を確認させる必要がある場合。
       * 同一データに対する同時実行数が少なく、更新処理も短い時間でおわる場合。
     - * 取得したデータに対して、他のトランザクションからの更新が行われていないことが保証される。
       * テーブルにVersionを管理するためのカラムを定義する必要がある。
   * - 3.
     - 悲観ロック
     - * 長い時間ロックされる可能性があるデータに対して更新する場合。
       * 楽観ロックが使用できない(Versionを管理するためのカラムが定義できない)ため、処理としてデータの整合性チェックを行う必要がある場合。
       * 同一データに対する同時実行数が多く、更新処理も長い時間実行される可能性がある場合。
     - * 他のトランザクションの処理結果によって処理が失敗する可能性がなくなる。
       * 悲観ロックを取得するためのselect文を発行する必要があるので、その分コストがかかる。

.. note:: **ロックの種類の採用基準について**

    どの手法を採用するかについて、アーキテクトが、機能要件および性能要件を考慮して決定すること。

    * 画面にデータを戻し、画面上でデータを変更するような、データベースとのトランザクションが切れて、次のトランザクションでデータが変わっていないことを保証するためには、楽観ロックが必要となる。
    * 1トランザクション内でロックをかける必要がある場合は、悲観ロックと楽観ロックの両方で実現できるが、悲観ロックを使用した場合、データベース内のロック制御処理が行われるため、データベース内の処理コストが高くなる可能性がある。特に問題がない場合は、楽観ロックの方がよい。
    * 更新頻度が高い処理で、1トランザクション内で多くのテーブルを更新する場合は、楽観ロックを使用すると、ロックを取得するための待ち時間は最小限に抑えることが出できるが、途中で排他エラーとなる可能性があるため、エラーが発生するポイントが増える。
      悲観ロックを使用すると、ロックを取得するまでの待ち時間が長くなる可能性はあるが、ロックを取得した後の処理で排他エラーが発生することはないため、エラーが発生するポイントが減る。

.. tip:: **業務トランザクションについて**

    実際のアプリケーション開発では、業務フローレベルのトランザクションに対して、排他制御が必要になる場合もある。
    業務フローレベルのトランザクションとなる代表例としては、旅行代理店のカウンタで、お客様と話をしながら予約作業を進めていく際に使用するアプリケーションがあげられる。

    旅行予約を行う場合、鉄道、宿泊施設、さらに追加プランなどを話しながら決めていくことになる。
    その際に、予約することに決めた宿泊施設や追加プランが、他の利用者に予約されないようにする仕組みが必要になる。
    このような場合は、テーブルにステータスを持たせ、仮予約 -> 予約 のように更新し、仮予約中の場合も、他の利用者から更新されないようにする必要がある。

    業務トランザクションに対する排他制御については、業務設計や機能設計として検討・設計すべき箇所になるので、本節の説明範囲からは省いている。

データベースの行ロック機能による排他制御
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| ほとんどのデータベースでは、レコードを更新（UPDATE,DELETE）した場合、コミットまたはロールバックされるまで、他のトランザクションからの更新を待機させるための行ロックが取得される。
| そのため、更新件数が想定通りであれば、データの整合性を保証することができる。

| この特性を活かし、更新時のWHERE句に対して、データの整合性を担保するための条件を指定することで、排他制御を行うことができる。
| 以下に、データベース毎の、更新時の行ロックのサポート状況を示す。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.10\linewidth}|p{0.15\linewidth}|p{0.35\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 20 10 15 35

   * - 項番
     - データベース
     - 確認Version
     - デフォルト設定時のロック
     - 備考
   * - 1.
     - Oracle
     - 11
     - 行ロック
     - ロック分メモリ使用量が増大する。
   * - 2.
     - PostgreSQL
     - 9
     - 行ロック
     - メモリ上に変更された行の情報を記憶しないので、同時にロックできる行数に、上限はない。ただし、テーブルに書き込むため、定期的にVACUUMしなければならない。
   * - 3.
     - DB2
     - 9
     - 行ロック
     - ロック分メモリ使用量が増大する。
   * - 4.
     - MySQL InnoDB
     - 5
     - 行ロック
     - ロック分メモリ使用量が増大する。

| データベースの行ロック機能による排他制御は、他のトランザクションによって更新した内容を確認する必要がない場合に使用することができる。
| 例えば、ショッピングサイトの購入処理にて、購入した商品の個数を、商品の在庫数を管理するレコードからマイナスするような処理が挙げられる。
| ステータス管理を管理する処理などでは、前のステータスが重要になるので、この方法で排他制御を実現することを推奨しない。

以下に、具体例を示す。シナリオは、以下の通りである。

* ショッピングサイトでUser A,User Bともに同じ商品の購入画面を同時に表示する。
  その際に、Stock Tableから取得した在庫数も表示されている。
* 買いたい商品を5個ずつ同時に購入したが、少しUserAの方が早く購入ボタンを押下したため、User Aが先に購入し、User Bが次に購入する。

 .. figure:: ./images/update-for-db-line-lock.png
   :alt: update for db line lock
   :width: 90%
   :align: center


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.05\linewidth}|p{0.05\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 5 5 80

    * - 項番
      - UserA
      - UserB
      - 説明
    * - 1.
      - 〇
      - \-
      - User Aが、商品の購入画面を表示する。在庫数が100個で画面に表示されている。

        .. code-block:: sql

          select quantity from Stock where ItemId = '01'

    * - 2.
      - \-
      - 〇
      - User Bが、商品の購入画面を表示する。在庫数が100個で画面に表示されている。

        .. code-block:: sql

          select quantity from Stock where ItemId = '01'

    * - 3.
      - 〇
      - \-
      - User Aが、ItemId=01の商品を5個購入する。Stock Tableから個数を-5する。

        .. code-block:: sql

          Update from Stock set quantity = quantity - 5
                                where ItemId='01' and quantity >= 5

    * - 4.
      - \-
      - 〇
      - User Bが、ItemId=01の商品を5個購入する。Stock Tableから個数を-5しようとするが、User Aのトランザクションが終了していないので、User Bの購入処理が待たされる。
    * - 5.
      - 〇
      - \-
      - User Aのトランザクションをコミットする。
    * - 6.
      - \-
      - 〇
      - | User Aのトランザクションがコミットされたため、4で待たされていたUserBの購入処理が再開する。
        | この時、在庫は画面で見ると、個数は100ではなく、95になっているが、購入数(上記例では、5個)以上の在庫が残っているため、Stock Tableから個数を-5する。

        .. code-block:: sql

          Update from Stock set quantity = quantity - 5
                                where ItemId='01' and quantity >= 5

    * - 7.
      - \-
      - 〇
      - User Bのトランザクションをコミットする。

 .. note:: **ポイント**

    SQL内で減算( ``"quantity - 5"`` )と、更新条件( ``"and quantity >= 5"`` )の指定を行うことが、ポイントとなる。

| 上と同じシナリオで、商品の購入画面を表示した際の在庫数が、9個だった場合、User Bの更新処理が再開した時点の在庫数が、4個のため、\ ``quantity >= 5``\ を満たさないので、更新件数が0件となる。
| アプリケーションでは、更新件数が0件の場合、購入処理をロールバックし、User Bに再度実行を促す。

 .. figure:: ./images/update-for-db-line-lock-not-enough.png
   :alt: update for db line lock not enough
   :width: 90%
   :align: center

 .. note:: **ポイント**

    アプリケーションで更新件数をチェックし、想定件数と異なる場合にエラーを発生させ、トランザクションをロールバックすることが、ポイントとなる。

**この方法でロックする場合、参照した情報が変わっていても条件次第で処理を進めることができ、かつ、データベースの機能によってデータの整合性を保証することができる。**

楽観ロックによる排他制御
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 楽観ロックとは、データそのものに対してロックは行わずに、更新対象のデータが、データ取得時と同じ状態であることを確認してから更新することで、データの整合性を保証する手法である。
| 楽観ロックを使用する場合は、更新対象のデータが、データ取得時と同じ状態であることを判断するために、Versionを管理するためのカラム(Versionカラム)を用意する。
| 更新時の条件として、データ取得時のVersionと、データ更新時のVersionを同じとすることで、データの整合性を保証することができる。

.. note:: **Versionカラムとは**

    レコードの更新回数を管理するためのカラムで、レコード挿入時に0を設定し、更新成功時にインクリメントしていく楽観ロック用のカラムである。
    Versionカラムは、数値以外に最終更新タイムスタンプで代用することもできる。
    しかし、タイムスタンプを用いると、同時に処理が実行された際の、一意性が保証されない。
    そこで、確実な一意性を求める場合、Versionカラムは、数値を使用する必要がある。

| 楽観ロックによる排他制御は、他のトランザクションによって更新されていた場合に、更新内容を確認させる必要がある場合に使用する。
| 例えば、ワークフローアプリケーションにおいて、申請者と承認者が同時に操作（引き戻しと承認）を行った場合を想像してほしい。
| この時、楽観ロックによる排他制御を行うことで、操作の前後で状態が変わっているため、操作が完了しなかったことを、申請者と承認者に通知することができる。

.. warning::

    楽観ロックを行う場合、IDとVersion以外の条件を加えて更新・削除するのは適切でない。
    なぜなら更新できなかった場合に、Versionが一致しないことが理由なのか、別の条件に一致しないのが理由なのか、判断できないためである。
    更新条件として別の条件がある場合は、事前の処理として条件を満たしているか、チェックを行う必要がある。

具体例を、以下に示す。シナリオは、以下の通りである。

* ショッピングサイトの在庫数を管理するスタッフ(Staff A, Staff B)が、それぞれ商品を仕入れる。Staff Aが5個、Staff Bが15個仕入れたものとする。
* 仕入れた商品を、在庫管理システムに反映するために、在庫管理画面を表示する。その際、在庫管理システムで管理されている在庫数が表示される。
* それぞれ表示された在庫数に対して、仕入れた数を加算した値を更新フォームに入力し、更新を行う。

 .. figure:: ./images/Optimistic-lock-flow.png
   :alt: Optimistic lock flow
   :width: 90%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.05\linewidth}|p{0.05\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 5 5 80

    * - 項番
      - Staff A
      - Staff B
      - 説明
    * - 1.
      - 〇
      - \-
      - Staff Aが、商品の在庫管理画面を表示する。在庫数は10個と画面に表示されている。参照したデータのVersionは ``1`` である。
    * - 2.
      - \-
      - 〇
      - Staff Bが、商品の在庫管理画面を表示する。在庫数は10個と画面に表示されている。参照したデータのVersionは ``1`` である。
    * - 3.
      - 〇
      - \-
      - Staff Aが、画面に表示されていた在庫数10に対して、仕入れた5個を加算し、変更後の在庫数を15個で更新する。更新条件として、参照したデータのVersionを含める。

        .. code-block:: sql

           UPDATE Stock SET quantity = 15, version = version + 1
                        WHERE itemId = '01' and version = 1

    * - 4.
      - \-
      - 〇
      - Staff Bが、画面に表示されていた在庫数10に対して仕入れた15個を加算し、変更後の在庫数を25個で更新しようとするが、Staff Aのトランザクションが終了していないので待たされる。更新条件として、参照したデータのVersionを含める。
    * - 5.
      - 〇
      - \-
      - Staff Aのトランザクションをコミットする。 **この時点で、Versionは  2 になる。**
    * - 6.
      - 〇
      - \-
      - Staff Aのトランザクションがコミットされたため、4で待たされていたStaff Bの更新処理が再開する。この時、Stock TableのデータのVersionが ``2`` になっているため、更新結果が0件となる。更新結果が0件の場合は排他エラーとする。

        .. code-block:: sql

           UPDATE Stock SET quantity = 25, version = version + 1
                        WHERE itemId = '01' and version = 1

    * - 7.
      - 〇
      - \-
      - Staff Bのトランザクションをロールバックする。

.. note:: **ポイント**

    SQL内でVersionのインクリメント( ``"version + 1"`` )と、更新条件( ``"and version = 1"`` )の指定を行うことが、ポイントとなる。

悲観ロックによる排他制御
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 悲観ロックとは、更新対象のデータを取得する際にロックをかけることで、他のトランザクションから更新されないようにする手法である。
| 悲観ロックを使用する場合は、トランザクション開始直後に更新対象となるレコードのロックを取得する。
| ロックされたレコードは、トランザクションが、コミットまたはロールバックされるまで、他のトランザクションから更新されないため、データの整合性を保証することができる。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.30\linewidth}|
.. list-table:: RDBMS別の悲観ロック取得方法
   :header-rows: 1
   :widths: 10 15 30

   * - 項番
     - データベース
     - 悲観ロック方法
   * - 1.
     - Oracle
     - FOR UPDATE
   * - 2.
     - PostgreSQL
     - FOR UPDATE
   * - 3.
     - DB2
     - FOR UPDATE WITH
   * - 4.
     - MySQL
     - FOR UPDATE

.. note:: **悲観ロックのタイムアウトについて**

     悲観ロックには、悲観ロック取得時に他のトランザクションによってロックが取得されていた場合に、どのような動作にするかをオプションとして指定することがある。
     Oracleの場合は、

     * デフォルトでは、\ ``select for update [wait]``\ となり、ロックが解除されるまで待つ。
     * \ ``select for update nowait``\ とすると、他にロックされている場合は、即時にリソースビジーのエラーとなる。
     * \ ``select for update wait 5``\ とすると5秒待ち、5秒間ロックが解除されない場合は、リソースビジーのエラーが返却される。

     DBにより機能に差はあるが、悲観ロックを使用する際は、どの手法を採用するか検討が必要である。

.. note:: **JPA(Hibernate)を使用する場合**

     悲観ロックの取得方法はデータベースによって異なるが、その差分はJPA(Hibernate)によって吸収される。
     HibernateのサポートしているRDBMSについては、 `Hibernate Developer Guide <http://docs.jboss.org/hibernate/orm/4.3/devguide/en-US/html_single/#d5e233>`_ を参照されたい。

悲観ロックによる排他制御は、以下3ケースのいずれかに当てはまる場合に使用する。

#. | 更新対象のデータが複数のテーブルに分かれて管理されている。
   | 更新対象のテーブルが複数のテーブルに分かれている場合、各テーブルに対して更新が終わるまでの間に、他のトランザクションから更新がされないことを保証するために、必要となる。

#. | 更新処理を行う前に取得したデータの状態をチェックする必要がある。
   | チェック処理が終わった後に、他のトランザクションから更新がされていないことを保証するために、必要となる。

#. | バッチ実行中にオンラインの処理が実行されることがある。
   | バッチ処理では、実行途中に排他エラーが発生しないようにするために、更新対象となるデータのロックを一括で取得することがある。
   | 一括で取得されたロックが取得された場合、オンラインの処理が待たされる時間が長くなる可能性がある。その場合、タイムアウト時間を指定して、悲観ロックを使用するのが妥当である。

具体例を以下に示す。シナリオは、以下の通りである。

* バッチ処理が既に実行済みで、オンラインで更新するデータを悲観ロックしている。
* オンライン処理は10秒のタイムアウト時間を指定して、更新対象のデータのロックを取得する。
* バッチ処理は5秒後(タイムアウト前)に終了する。

 .. figure:: ./images/Pessimistic-lock.png
   :alt: Pessimistic lock
   :width: 90%
   :align: center

|

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.05\linewidth}|p{0.05\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 5 5 80

    * - 項番
      - Online
      - Batch
      - 説明
    * - 1.
      - \-
      - 〇
      - バッチ処理が、オンライン処理で更新するデータの悲観ロックを取得する。
    * - 2.
      - 〇
      - \-
      - オンライン処理が、更新対象のデータの悲観ロックを行うが、バッチ処理のトランザクションによって悲観ロックされているので待たされる。

        .. code-block:: sql

           SELECT * FROM Stock WHERE quantity < 5 FOR UPDATE WAIT 10

    * - 3.
      - \-
      - 〇
      - バッチ処理が、データを更新する。
    * - 4.
      - \-
      - 〇
      - バッチ処理のトランザクションをコミットする。
    * - 5.
      - 〇
      - \-
      - バッチ処理のトランザクションがコミットされたため、オンライン処理の処理が再開する。取得されるデータはバッチ処理の更新結果が反映されているので、データ不整合が発生することはない。
    * - 6.
      - 〇
      - \-
      - オンライン処理が、データを更新する。
    * - 7.
      - 〇
      - \-
      - オンライン処理のトランザクションをコミットする。

| 以下は、タイムアウトとなった場合の流れとなる。
| バッチ処理の終了まで待たずに排他エラーとなる。

 .. figure:: ./images/Pessimistic-lock-timeout.png
   :alt: Pessimistic lock
   :width: 90%
   :align: center


| 以下は、悲観ロックの取得待ちを行わない設定のとき、他のトランザクションによって、悲観ロックが取得されていた場合の流れとなる。
| 悲観ロックの解放を待つことなく、すぐに排他エラーとなる。

 .. figure:: ./images/Pessimistic-lock-nowait.png
   :alt: Pessimistic lock
   :width: 90%
   :align: center

**バッチ処理とオンライン処理が競合する可能性があり、かつバッチ処理の処理時間が長くなる場合は、悲観排他のタイムアウト時間を指定することを推奨する。**
**タイムアウト時間については、オンライン処理の処理要件に応じて決めること。**

デッドロックの予防
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| データベースのロック機能を使用する場合、同一トランザクション内で複数のレコードを更新すると、以下2通りの、デッドロックが発生する可能性があるため、注意する必要がある。

*  :ref:`Dead-Lock-Record`
*  :ref:`Dead-Lock-Table`

.. _Dead-Lock-Record:

テーブル内でのデッドロック
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
以下(1)～(5)の流れで、複数のトランザクションから、同一テーブルのレコードに対してロックを行うと、デッドロックとなる。

 .. figure:: ./images/Dead-Lock-Record.png
   :alt: Dead Lock Record
   :width: 90%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.05\linewidth}|p{0.05\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 5 5 80

    * - 項番
      - Program A
      - Program B
      - 説明
    * - | (1)
      - 〇
      - \-
      - Program Aは、Record X に対するロックを取得する。
    * - | (2)
      - 〇
      - \-
      - Program Bは、Record Y に対するロックを取得する。
    * - | (3)
      - 〇
      - \-
      - Program Aは、Program BのトランザクションによってロックされているRecord Y に対してロックの取得を試みるが、(2)のロック状態が解放されていないので、解放待ちの状態となる。
    * - | (4)
      - \-
      - 〇
      - Program Bは、Program AのトランザクションによってロックされているRecord X に対してロックの取得を試みるが、(1)のロック状態が解放されていないので、解放待ちの状態となる。
    * - | (5)
      - \-
      - \-
      - Program AとProgram Bが、お互いが保持しているロックの解放待ちの状態となるため、デッドロックとなる。デッドロックが発生した場合、データベースによって検知されエラーとなる。

 .. note:: **デッドロックの解決方法について**

    タイムアウトやリトライ実施での解消する方法もあるが、同一テーブル上でのレコードの更新順序にルールを決めることが重要である。
    1行ずつ更新する場合は、PK(PRIMARY KEY)順の若い順に更新するなどのルールを定めること。

    仮にProgram AもProgram BもRecord Xから更新するというルールに準じていれば、上記\ :ref:`Dead-Lock-Record`\ の図のようなデッドロックは発生しなくなる。

.. _Dead-Lock-Table:

テーブル間でのデッドロック
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 以下(1)～(5)の流れで、複数のトランザクションから、別テーブルのレコードに対してロックを行うと、デッドロックとなる。
| 基本的な考え方は、 :ref:`Dead-Lock-Record` と同じである。

 .. figure:: ./images/Dead-Lock-Table.png
   :alt: Dead Lock Table
   :width: 90%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.05\linewidth}|p{0.05\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 5 5 80

    * - 項番
      - Program A
      - Program B
      - 説明
    * - | (1)
      - 〇
      - \-
      - Program Aは、Table A の Record X に対するロックを取得する。
    * - | (2)
      - 〇
      - \-
      - Program Bは、Table B の Record Y に対するロックを取得する。
    * - | (3)
      - 〇
      - \-
      - Program Aは、Program BのトランザクションによってロックされているTable B の Record Y に対してロックの取得を試みるが、(2)のロック状態が解放されていないので、解放待ちの状態となる。
    * - | (4)
      - \-
      - 〇
      - Program Bは、Program AのトランザクションによってロックされているTable A の Record X に対してロックの取得を試みるが、(1)のロック状態が解放されていないので、解放待ちの状態となる。
    * - | (5)
      - \-
      - \-
      - Program AとProgram Bが、お互いが保持しているロックの解放待ちの状態となるため、デッドロックとなる。デッドロックが発生した場合、データベースによって検知されエラーとなる。

.. note:: **デッドロックの解決方法について**

    タイムアウトやリトライ実施での解消する方法もあるが、テーブルを跨った際も、更新順序をルール化しておくことが重要である。

    仮にProgram AもProgram BもTable Aから更新するというルールに準じていれば、上記\ :ref:`Dead-Lock-Table`\ の図のような、デッドロックは発生しなくなる。

.. warning::

    注意としては、どの方法を採用したとしても、レコードをロックする順序により、デッドロックが発生する可能性がある。
    テーブル、レコードのロック順序については、ルールを決めること。

|

How to use
--------------------------------------------------------------------------------

ここからは、O/R Mapperを使用した排他制御の実現方法について説明を行う。

使用するO/R Mapperの実装方法を確認されたい。

* :ref:`ExclusionControlHowToUseMyBatis3`
* :ref:`ExclusionControlHowToUseJpa`

また、排他エラーのハンドリング方法については、

* :ref:`ExclusionControlHowToUseExceptionHandling`

を参照されたい。

|

.. _ExclusionControlHowToUseMyBatis3:

MyBatis3使用時の実装方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

RDBMSの行ロック機能
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

RDBMSの行ロック機能を使って排他制御を行う場合は、SQLの中で、

* SET句に指定する更新内容
* WHERE句に指定する更新条件

を意識する必要がある。

|

- Repositoryインタフェースにメソッドを定義する。

 .. code-block:: java

     public interface StockRepository {
        // (1)
        boolean decrementQuantity(@Param("itemCode") String itemCode,
                                  @Param("quantity") int quantity);
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - Repositoryインタフェースに、
        RDBMSの行ロック機能を使ってデータを更新するメソッドを定義する。

        上記例では、在庫数を減らすためのメソッドを定義している。
        在庫数の減らす事ができた場合は、\ ``true``\が返却される。

|

- RDBMSの行ロック機能を使った排他制御が有効となるSQLを定義する。

 .. code-block:: xml

    <!-- (2) -->
    <update id="decrementQuantity">
    <![CDATA[
        UPDATE
            m_stock
        SET
            /* (3) */
            quantity = quantity - #{quantity}
        WHERE
            item_code = #{itemCode}
        AND
            /* (4) */
            quantity >= #{quantity}
    ]]>
    </update>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (2)
      - RDBMSの行ロック機能を使ってデータを更新するためのステートメント(SQL)を定義する。

        上記例では、在庫数を減らすためのSQLを定義している。

        RDBMSの行ロック機能を使う場合は、

        * 他のトランザクションが同一データに対してロックを取得している場合は、
          ロックが解放(コミット or ロールバック)された後にSQLが実行される。

        * 在庫数を減らすことに成功した場合は、
          RDBMSの行ロックが取得され、他のトランザクションからの更新がロックされる。

        という動作になるため、データを安全に更新する事ができる。
    * - | (3)
      - 在庫数の減算処理(\ ``quantity = quantity - #{quantity}``\)は、SQLの中で行う。
    * - | (4)
      - 更新条件として、「在庫数が注文数以上ある事(\ ``quantity >= #{quantity}``\)」を加える。

|

- Repositoryのメソッドを呼び出し、RDBMSの行ロック機能を使用してデータを安全に更新する。

 .. code-block:: java

    // (5)
    boolean updated = stockRepository.decrementQuantity(itemCode, quantityOfOrder);
    // (6)
    if (!updated) {
        // (7)
        ResultMessages messages = ResultMessages.error().add(ResultMessage
                .fromText("Not enough stock. Please, change quantity."));
        throw new BusinessException(messages);
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (5)
      - Repositoryのメソッドを呼び出し、更新処理を行う。
    * - | (6)
      - Repositoryのメソッドの呼び出し結果を判定する。

        \ ``false``\ の場合、更新条件を充たしていないため、在庫数が不足していることになる。
    * - | (7)
      - 業務エラーを発生させる。

        上記例では、ビジネスルールのチェック(在庫数チェック)を排他制御しながら行っているだけなので、
        更新条件を充たさない場合は、排他エラーではなく業務エラーとしている。

        発生させた業務エラーは、Controllerで適切にハンドリングすること。

|

楽観ロック
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| MyBatis3では、ライブラリとして楽観ロックを行う仕組みは提供していない。
| そのため、楽観ロックを行う場合は、SQLの中でバージョンを意識する必要がある。

- Entityにバージョン管理用のプロパティを定義する。

 .. code-block:: java

    public class Stock implements Serializable {
        private static final long serialVersionUID = 1L;

        private String itemCode;
        private int quantity;
        // (1)
        private long version;

        // ...

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - Entityにバージョン管理用のプロパティを用意する。

|

- Repositoryインタフェースにメソッドを定義する。

 .. code-block:: java

    public interface StockRepository {
        // (2)
        Stock findOne(String itemCode);
        // (3)
        boolean update(Stock stock);
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (2)
      - Repositoryインタフェースに、Entityを取得するためにメソッドを定義する。
    * - | (3)
      - Repositoryインタフェースに、楽観ロック機能を使ってデータを更新するメソッドを定義する。

        上記例では、指定されたEntityの内容でレコードを更新するためのメソッドを定義している。
        更新できた場合は、\ ``true``\が返却される。

|

- マッピングファイルにSQLを定義する。

 .. code-block:: xml

    <!-- (4) -->
    <select id="findOne" parameterType="string" resultType="Stock">
        SELECT
            item_code,
            quantity,
            version
        FROM
            m_stock
        WHERE
            item_code = #{itemCode}
    </select>

    <!-- (5) -->
    <update id="update" parameterType="Stock">
        UPDATE
            m_stock
        SET
            quantity = #{quantity},
            /* (6) */
            version = version + 1
        WHERE
            item_code = #{itemCode}
        AND
            /* (7) */
            version = #{version}
    </update>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (4)
      - Entityを取得するためのステートメント(SQL)を定義する。

        楽観ロックを使用する場合は、Entity取得時にバージョンを取得しておく必要がある。
    * - | (5)
      - 楽観ロック機能を使ってデータを更新するためのステートメント(SQL)を定義する。

        上記例では、指定されたEntityの内容でレコードを更新するSQLを定義している。
    * - | (6)
      -  バージョンの更新(\ ``version = version + 1``\)は、SQLの中で行う。
    * - | (4)
      - 更新条件として、「バージョンが変わっていない事(\ ``version = #{version}``\)」を加える。

|

- Repositoryのメソッドを呼び出し、楽観ロック機能を使用してデータを安全に更新する。

 .. code-block:: java

    // (5)
    Stock stock = stockRepository.findOne(itemCode);
    if (stock == null) {
        ResultMessages messages = ResultMessages.error().add(ResultMessage
                .fromText("Stock not found. itemCode : " + itemCode));
        throw new ResourceNotFoundException(messages);
    }

    // (6)
    stock.setQuantity(stock.getQuantity() + addedQuantity);

    // (7)
    boolean updated = stockRepository.update(stock);
    if(!updated) {
        // (8)
        throw new ObjectOptimisticLockingFailureException(Stock.class, itemCode);
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (5)
      - RepositoryインタフェースのfindOneメソッドを呼び出し、Entityを取得する。
    * - | (6)
      - (5)で取得したEntityに対して、更新する値を指定する。

        上記例では、仕入れた在庫数を加算している。
    * - | (7)
      - Repositoryインタフェースのupdateメソッドを呼び出し、
        (5)の処理で更新したEntityを永続層(DB)に反映する。
    * - | (8)
      - 更新結果を判定し、更新結果が\ ``false``\ の場合は、
        他のトランザクションによってEntityが更新されたことになるので、
        楽観ロックエラー(\ ``org.springframework.orm.ObjectOptimisticLockingFailureException``\ )を発生させる。

|

ロングトランザクションに対して楽観ロックを行う場合は、以下の点に注意すること。

 .. warning::

    ロングトランザクションに対して楽観ロックを行う場合は、更新時のチェックとは別に、
    データ取得時にもバージョンのチェックを行うこと。


以下に、実装例を示す。

- データ取得時にもバージョンのチェックを行う。

 .. code-block:: java

    Stock stock = stockRepository.findOne(itemCode);
    if (stock == null || stock.getVersion() != version) {
        // (9)
        throw new ObjectOptimisticLockingFailureException(Stock.class, itemCode);
    }

    stock.setQuantity(stock.getQuantity() + addedQuantity);
    boolean updated = stockRepository.update(stock);
    // ...

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (9)
      - 別のデータベーストランザクションで取得したEntityのバージョンと、
        (5)で取得したEntityのバージョンを比較する。

        バージョンが異なる場合は、他のトランザクションによってデータが更新されているので、
        楽観ロックエラー(\ ``org.springframework.dao.ObjectOptimisticLockingFailureException``\ )を発生させる。

        データが存在しない(\ ``stock == null``\)時の考慮も必要であり、
        アプリケーションの仕様に対応した実装を行う必要がある。
        上記例では、楽観ロックエラーとしている。


|

RDBMSの行ロック機能と楽観ロック機能を併用するアプリケーション場合は、以下の点に注意すること。

 .. warning::

    RDBMSの行ロック機能を利用して排他制御を行う処理と、
    楽観ロック機能を利用して排他制御を行う処理が共存するアプリケーションの場合は、
    RDBMSの行ロック機能を使うSQLの中で、**バージョンの更新(インクリメント)が必要となる。**

    仮にRDBMSの行ロック機能を使って排他制御を行うSQLの中でバージョンを更新しなかった場合、
    楽観ロック機能を利用して排他制御を行っているSQLでデータを上書きしてしまう可能性がある。

以下に、実装例を示す。

- SQL内でバージョンを更新する。

 .. code-block:: xml

    <update id="decrementQuantity">
    <![CDATA[
        UPDATE
            m_stock
        SET
            quantity = quantity - #{quantity},
            /* (10) */
            version = version + 1
        WHERE
            item_code = #{itemCode}
        AND
            quantity >= #{quantity}
    ]]>
    </update>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (10)
      - バージョンの更新(インクリメント)を行う。

|

悲観ロック
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| MyBatis3では、ライブラリとして悲観ロックを行う仕組みは提供していない。
| そのため、悲観ロックを行う場合は、SQLの中でロックを取得するためのキーワードを指定する必要がある。

- SQLの中でロックを取得するためのキーワードを指定する

 .. code-block:: xml

    <select id="findOneForUpdate" parameterType="string" resultType="Stock">
        SELECT
            item_code,
            quantity,
            version
        FROM
            m_stock
        WHERE
            item_code = #{itemCode}
        /* (1) */
        FOR UPDATE
    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - 悲観ロックの取得が必要なSQLに対して、悲観ロックを取得するためのキーワードを指定する。

        キーワードやキーワードの指定位置は、データベースによって異なる。

|

.. _ExclusionControlHowToUseJpa:

JPA(Spring Data JPA)使用時の実装方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

RDBMSの行ロック機能
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| RDBMSの行ロック機能を使って排他制御を行う場合は、RepositoryインタフェースにQueryメソッドを追加して実現する。
| Queryメソッドについては、\ :ref:`data-access-jpa_how_to_use_querymethod`\ と、\ :ref:`data-access-jpa_howtouse_querymethod_modifying`\ を参照されたい。

- Repositoryインタフェース

 .. code-block:: java
   :emphasize-lines: 5, 7

     public interface StockRepository extends JpaRepository<Stock, String> {

        @Modifying
        @Query("UPDATE Stock s"
                + " SET s.quantity = s.quantity - :quantity"
                + " WHERE s.itemCode = :itemCode"
                + " AND :quantity <= s.quantity")  // (1)
        public int decrementQuantity(@Param("itemCode") String itemCode,
                @Param("quantity") int quantity);

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | Queryメソッドに、在庫数が注文数以上ある場合に、在庫数を減らすJPQLを指定する。
        | 更新件数をチェックする必要があるので、Queryメソッドの返り値は、\ ``int``\ を指定する。

- Service

 .. code-block:: java

    String itemCodeOfOrder = "ITM0000001";
    int quantityOfOrder = 31;

    int updateCount = stockRepository.decrementQuantity(itemCodeOfOrder, quantityOfOrder); // (2)
    if (updateCount == 0) { // (3)
        ResultMessages message = ResultMessages.error();
        message.add(ResultMessage
                .fromText("Not enough stock. Please, change quantity."));
        throw new BusinessException(message); // (4)
    }

 .. code-block:: sql

    update m_stock set quantity=quantity-31
                   where item_code='ITM0000001' and 31<=quantity -- (5)


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (2)
      - Queryメソッドを呼び出す。
    * - | (3)
      - Queryメソッドの呼び出し結果を判定する。\ ``0``\ の場合、更新条件を満たしていないので、在庫数が不足していることになる。
    * - | (4)
      - | 在庫がない、または不足している旨のメッセージを格納し、業務エラーを発生させる。
        | 発生させたエラーは、Controllerで要件に応じて適切にハンドリングすること。
        | 上記例では、ビジネスルールのチェックを排他制御しながら行っているだけなので、更新条件を満たさない場合は、排他エラーではなく業務エラーとしている。
        | エラーのハンドリング方法については、\ :ref:`exception-handling-how-to-use-codingpoint-controller-label`\ を参照されたい。
    * - | (5)
      - Queryメソッド呼び出し時に実行されるSQL。

楽観ロック
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

JPAでは、バージョン管理用のプロパティに、\ ``@javax.persistence.Version``\ アノテーションを指定することで、楽観ロックを行うことができる。

- Entity

 .. code-block:: java
   :emphasize-lines: 11

    @Entity
    @Table(name = "m_stock")
    public class Stock implements Serializable {

        @Id
        @Column(name = "item_code")
        private String itemCode;

        private int quantity;

        @Version // (1)
        private long version;

        // ...

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - バージョン管理用のプロパティに、\ ``@Version``\ アノテーションを指定する。

- Service

 .. code-block:: java

    String itemCode = "ITM0000001";
    int newQuantity = 30;

    Stock stock = stockRepository.findOne(itemCode); // (2)
    if (stock == null) {
        ResultMessages messages = ResultMessages.error().add(ResultMessage
                .fromText("Stock not found. itemCode : " + itemCode));
        throw new ResourceNotFoundException(messages);
    }

    stock.setQuantity(newQuantity); // (3)

    stockRepository.flush(); // (4)

 .. code-block:: sql

    update m_stock set quantity=30, version=7
                   where item_code='ITM0000001' and version=6 -- ( 5)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (2)
      - RepositoryインタフェースのfindOneメソッドを呼び出し、Entityを取得する。
    * - | (3)
      - (2)で取得したEntityに対して、更新する値を指定する。
    * - | (4)
      - | (3)の変更内容を永続層(DB)に反映する。この処理は説明のために行っている処理のため、通常は不要である。
        | 通常は、トランザクションコミット時に自動で反映される。
        | 上記例だと、(2)で取得したEntityがもつバージョンと永続層(DB)で保持しているバージョンが一致しない場合に、楽観ロックエラー(\ ``org.springframework.dao.OptimisticLockingFailureException``\ ) が発生する。
    * - | (5)
      - (4)の永続層(DB)に反映する際に実行されるSQL。

ロングトランザクションに対する楽観ロックを行う場合は、以下の点に注意すること。

.. warning::

  ロングトランザクションに対する楽観ロックについては、\ ``@Version``\ アノテーションを付与するだけでは不十分である。
  ロングトランザクションに対して楽観ロックを行う場合は、JPAの機能で行われる更新時のチェックに加えて、更新対象のデータを取得する際にも、バージョンのチェックを行うこと。

以下に、実装例を示す。

- Service

 .. code-block:: java

    long version = 12;
    String itemCode = "ITM0000001";
    int newQuantity = 30;

    Stock stock = stockRepository.findOne(itemCode); // (1)
    if (stock == null || stock.getVersion() != version) { // (2)
        throw new ObjectOptimisticLockingFailureException(Stock.class, itemCode); // (3)
    }

    stock.setQuantity(newQuantity);

    stockRepository.flush();

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - 永続層(DB)からEntityを取得する。
    * - | (2)
      - | 事前に別のデータベーストランザクションで取得されたEntityのバージョンと、(1)で取得した永続層(DB)の最新のバージョンを比較する。
        | バージョンが一致する場合は、以降の処理で\ ``@Version``\ アノテーションを使った楽観ロックの仕組みが有効となる。
    * - | (3)
      - バージョンが異なる場合は、楽観ロックエラー(\ ``org.springframework.dao.ObjectOptimisticLockingFailureException``\ )を発生させる。

.. warning:: **Version管理用のプロパティへの値の設定について**

    Repositoryインタフェースを使って取得したEntityは、「管理状態のEntity」と呼ばれる。

     **「管理状態のEntity」に対して、処理でVersion管理用のプロパティの値を設定することはできないので、注意すること。**

    以下のような処理をしても、「管理状態のEntity」に設定したバージョンの値は反映されないため、楽観ロックを取得する際に使用されることはない。楽観ロックで使用されるのは、findOneメソッドで取得した時点のバージョンとなる。

     .. code-block:: java
        :emphasize-lines: 11

        long version = 12;
        String itemCode = "ITM0000001";
        int newQuantity = 30;

        Stock stock = stockRepository.findOne(itemCode);
        if (stock == null) {
            ResultMessages messages = ResultMessages.error().add(ResultMessage
                    .fromText("Stock not found. itemCode : " + itemCode));
            throw new ResourceNotFoundException(messages);
        }
        stock.setVersion(version); // ★ Invalid Processing
        stock.setQuantity(newQuantity);

        stockRepository.flush();

    例えば、画面から送られてきたバージョンの値を上書きしても、Entityには反映されないため、排他制御が正しく行われなくなってしまう。


.. note:: **ロングトランザクションに対する楽観ロック処理の共通化について**

  複数の処理でロングトランザクションに対して楽観ロックが必要になる場合は、上記の(1)～(3)の処理を共通的なメソッドにすることを検討した方がよい。
  共通化の方法については、\ :ref:`data-access-jpa_how_to_extends_custommethod`\ を参照されたい。

RDBMSの行ロック機能と、楽観ロック機能を両方使用する場合は、以下の点に注意すること。

.. warning::

  同じデータに対して、RDBMSの行ロック機能を利用して排他制御を行う処理と、
  楽観ロック機能を利用して排他制御を行う処理が共存するアプリケーションの場合は、
  RDBMSの行ロック機能を使うQueryメソッドにて、\ **Versionの更新を必ず行う必要がある。**\

  RDBMSの行ロック機能を使って、排他制御を行うQueryメソッドでVersionを更新しない場合、
  Queryメソッドで更新した内容が、別のトランザクションの処理で上書きされる可能性があるため、正しく排他制御が行われない。

以下に、実装例を示す。

- Repositoryインタフェース

 .. code-block:: java
   :emphasize-lines: 5

    public interface StockRepository extends JpaRepository<Stock, String> {

        @Modifying
        @Query("UPDATE Stock s SET s.quantity = s.quantity - :quantity"
                + ", s.version = s.version + 1" // (1)
                + " WHERE s.itemCode = :itemCode"
                + " AND :quantity <= s.quantity")
        public int decrementQuantity(@Param("itemCode") String itemCode,
                @Param("quantity") int quantity);

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | Versionの更新(\ ``s.version = s.version + 1``\ )を行う必要がある。

悲観ロック
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Spring Data JPAでは、\ ``@org.springframework.data.jpa.repository.Lock``\ アノテーションを指定することで、悲観ロックを行うことができる。

- Repositoryインタフェース

 .. code-block:: java

    public interface StockRepository extends JpaRepository<Stock, String> {

        @Lock(LockModeType.PESSIMISTIC_WRITE) // (1)
        @Query("SELECT s FROM Stock s WHERE s.itemCode = :itemCode")
        Stock findOneForUpdate(@Param("itemCode") String itemCode);

    }

 .. code-block:: sql

    -- (2)
    SELECT
            stock0_.item_code AS item1_5_
            ,stock0_.quantity AS quantity2_5_
            ,stock0_.version AS version3_5_
        FROM
            m_stock stock0_
        WHERE
            stock0_.item_code = 'ITM0000001'
        FOR UPDATE;

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - Queryメソッドに、\ ``@Lock``\ アノテーションを指定する。
    * - | (2)
      - 実行されるSQL。上記例ではPostgreSQLを使用した場合に実行されるSQLとなる。

\ ``@Lock``\ アノテーションで指定することができる悲観ロックの種類は、以下の通りである。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.25\linewidth}|p{0.45\linewidth}|p{0.20\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 25 45 20

   * - 項番
     - LockModeType
     - 説明
     - 発行されるSQL
   * - 1.
     - PESSIMISTIC_READ
     - | 参照用の悲観ロックが取得される。データベースによっては、排他ロックではなく共有ロックとなる。
       | コミットまたはロールバック時に、ロック解放される
     - | select ... for update / select ... for share
   * - 2.
     - PESSIMISTIC_WRITE
     - | 更新用の悲観ロックが取得され、排他ロックがかかる。
       | 排他ロックの場合、既にロックがかかっていた場合には、ロックが解放されるまで待機してからエンティティが取得される。
       | コミットまたはロールバック時に、ロック解放される
     - | select ... for update
   * - 3.
     - PESSIMISTIC_FORCE_INCREMENT
     - | エンティティを取得した時点から、対象データに対して排他ロックがかかる。取得直後に強制的にバージョンの更新も行われる。
       | コミットまたはロールバック時に、ロック解放される
     - | select ... for update + update

 .. note:: **ロックタイムアウト時間について**

    JPA(\ ``EntityManager``\ )の設定またはQueryヒントとして、\ ``"javax.persistence.lock.timeout"``\ を指定することで、タイムアウト時間を指定することができる。

ロックのタイムアウト時間の指定は、全体に適用する方法と、Query毎に適用する２つの方法が用意されている。

全体に適用する方法は、以下の通りである。

- :file:`xxx-infra.xml`

 .. code-block:: xml

     <bean id="entityManagerFactory"
         class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
         <property name="packagesToScan" value="xxxxxx.yyyyyy.zzzzzz.domain.model" />
         <property name="dataSource" ref="dataSource" />
         <property name="jpaVendorAdapter" ref="jpaVendorAdapter" />
         <property name="jpaPropertyMap">
             <util:map>
                 <!-- ... -->
                 <entry key="javax.persistence.lock.timeout" value="1000" /> <!-- (1) -->
             </util:map>
         </property>
     </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - タイムアウトをミリ秒で指定する。\ ``1000``\ を指定すると、1秒となる。

 .. note:: **nowaitのサポート**

    OracleとPostgreSQLについては、\ ``0``\ を指定した場合、\ ``nowait``\ が付加され、他のトランザクションによってロックされていた場合に、ロックの解放待ちを行わずに排他エラーとなる。

 .. warning:: **PostgreSQLの制約**

    PostgreSQLではnowaitの指定はできるが、wait時間の指定ができない。
    そのため、Queryのタイムアウトを別途設けておくなどの対策を行う必要がある。

Query毎に適応する方法は、以下の通りである。


- Repositoryインタフェース

 .. code-block:: java

    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @QueryHints(@QueryHint(name = "javax.persistence.lock.timeout", value = "2000")) // (1)
    @Query("SELECT s FROM Stock s WHERE s.itemCode = :itemCode")
    Stock findOneForUpdate(@Param("itemCode") String itemCode);

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | タイムアウトをミリ秒で指定する。\ ``2000``\ を指定すると、2秒となる。
        | 全体に指定した値は、上書きされる。

|

.. _ExclusionControlHowToUseExceptionHandling:

排他エラーのハンドリング方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

楽観ロックの失敗時のエラーハンドリング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

楽観ロックの失敗時には、\ ``org.springframework.dao.OptimisticLockingFailureException``\ が発生するため、
Controllerで適切にハンドリングする必要がある。

| ハンドリング方法は、楽観ロックエラーが発生した時のアプリケーションの動作仕様によって異なる。

リクエスト単位に動作を変える必要がない場合は、\ ``@ExceptionHandler``\ アノテーションを使用してハンドリングする。

 .. code-block:: java

    @ExceptionHandler(OptimisticLockingFailureException.class) // (1)
    public ModelAndView handleOptimisticLockingFailureException(
            OptimisticLockingFailureException e) {
        // (2)
        ExtendedModelMap modelMap = new ExtendedModelMap();
        ResultMessages resultMessages = ResultMessages.warning();
        resultMessages.add(ResultMessage.fromText("Other user updated!!"));
        modelMap.addAttribute(setUpForm());
        modelMap.addAttribute(resultMessages);
        String viewName = top(modelMap);
        return new ModelAndView(viewName, modelMap);
    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - \ ``@ExceptionHandler``\ アノテーションのvalue属性に、\ ``OptimisticLockingFailureException.class``\ を指定する。
    * - | (2)
      - | エラーハンドリングの処理を実装する。エラーを通知するためのメッセージ、画面表示に必要な情報（フォームやその他のモデル）を生成し、遷移先を指定した\ ``ModelAndView``\ を返却する。
        | エラーハンドリングの詳細については、\ :ref:`exception-handling-how-to-use-codingpoint-controller-usecase-label`\ を参照されたい。

リクエスト単位に動作を変える必要がある場合は、Controllerのハンドラメソッドの中で、\ ``try - catch``\ を使用してハンドリングする。

 .. code-block:: java

    @RequestMapping(value = "{itemId}/update", method = RequestMethod.POST)
    public String update(StockForm form, Model model, RedirectAttributes attributes){

        // ...

        try {
            stockService.update(...);
        } catch (OptimisticLockingFailureException e) { // (1)
            // (2)
            ResultMessages resultMessages = ResultMessages.warn();
            resultMessages.add(ResultMessage.fromText("Other user updated!!"));
            model.addAttribute(resultMessages);
            return updateRedo(modelMap);
        }

        // ...

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - ``OptimisticLockingFailureException`` をcatchする。
    * - | (2)
      - | エラーハンドリングの処理を実装する。エラーを通知するためのメッセージ、画面表示に必要な情報（フォームやその他のモデル）を生成し、遷移先のview名を返却する。
        | エラーハンドリングの詳細については、\ :ref:`exception-handling-how-to-use-codingpoint-controller-request-label`\ を参照されたい。

悲観ロックの失敗時のエラーハンドリング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

悲観ロックの失敗時には、\ ``org.springframework.dao.PessimisticLockingFailureException``\ が発生するため、 Controllerで適切にハンドリングする必要がある。

ハンドリング方法は、悲観ロックエラーが発生した時のアプリケーションの動作仕様によって異なる。

リクエスト単位に動作を変える必要がない場合は、\ ``@ExceptionHandler``\ アノテーションを使用してハンドリングする。

 .. code-block:: java

    @ExceptionHandler(PessimisticLockingFailureException.class) // (1)
    public ModelAndView handlePessimisticLockingFailureException(
            PessimisticLockingFailureException e) {
        // (2)
        ExtendedModelMap modelMap = new ExtendedModelMap();
        ResultMessages resultMessages = ResultMessages.warning();
        resultMessages.add(ResultMessage.fromText("Other user updated!!"));
        modelMap.addAttribute(setUpForm());
        modelMap.addAttribute(resultMessages);
        String viewName = top(modelMap);
        return new ModelAndView(viewName, modelMap);
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - \ ``@ExceptionHandler``\ アノテーションのvalue属性に、\ ``PessimisticLockingFailureException.class``\ を指定する。
    * - | (2)
      - | エラーハンドリングの処理を実装する。エラーを通知するためのメッセージ、画面表示に必要な情報（フォームやその他のモデル）を生成し、遷移先を指定した\ ``ModelAndView``\ を返却する。
        | エラーハンドリングの詳細については、\ :ref:`exception-handling-how-to-use-codingpoint-controller-usecase-label`\ を参照されたい。

リクエスト単位に動作を変える必要がある場合は、Controllerのハンドラメソッドの中で、\ ``try - catch``\ を使用してハンドリングする。

 .. code-block:: java

    @RequestMapping(value = "{itemId}/update", method = RequestMethod.POST)
    public String update(StockForm form, Model model, RedirectAttributes attributes){

        // ...

        try {
            stockService.update(...);
        } catch (PessimisticLockingFailureException e) { // (1)
            // (2)
            ResultMessages resultMessages = ResultMessages.warn();
            resultMessages.add(ResultMessage.fromText("Other user updated!!"));
            model.addAttribute(resultMessages);
            return updateRedo(modelMap);
        }

        // ...

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - \ ``PessimisticLockingFailureException``\ をcatchする。
    * - | (2)
      - | エラーハンドリングの処理を実装する。エラーを通知するためのメッセージ、画面表示に必要な情報（フォームやその他のモデル）を生成し、遷移先のview名を返却する。
        | エラーハンドリングの詳細については、\ :ref:`exception-handling-how-to-use-codingpoint-controller-request-label`\ を参照されたい。


.. raw:: latex

   \newpage

