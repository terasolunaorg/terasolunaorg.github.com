例外ハンドリング
------------------

.. only:: html

 .. contents:: 目次
    :depth: 4
    :local:

本ガイドラインで作成する、Webアプリケーションの例外ハンドリング指針について説明する。

Overview
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
本節では、Spring MVC配下の処理で発生する例外のハンドリングについて説明する。説明対象は、以下の通りである。

.. figure:: ./images/exception-handling-description-target.png
  :alt: description target
  :width: 80%
  :align: center

  **図-説明対象**

#. :ref:`exception-handling-class-label`
#. :ref:`exception-handling-method-label`


.. _exception-handling-class-label:

例外の分類
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
アプリケーション実行時に発生する例外は、以下3つに分類される。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.30\linewidth}|p{0.30\linewidth}|
.. list-table:: **表-アプリケーション実行時に発生する例外の分類**
   :header-rows: 1
   :widths: 10 30 30 30

   * - 項番
     - 分類
     - 説明
     - :ref:`exception-handling-exception-type-label`
   * - | (1)
     - | オペレータの再操作(入力値の変更など)によって、発生原因が解消できる例外
     - | オペレータの再操作によって、発生原因が解消できる例外は、アプリケーションコードで例外をハンドリングし、例外処理を行う。
     - | 1. :ref:`exception-handling-exception-type-businessexception-label`
       | 2. :ref:`exception-handling-exception-type-libraryexception-label`
   * - | (2)
     - | オペレータの再操作によって、発生原因が解消できない例外
     - | オペレータの再操作によって、発生原因が解消できない例外は、フレームワークで例外をハンドリングし、例外処理を行う。
     - | 1. :ref:`exception-handling-exception-type-systemexception-label`
       | 2. :ref:`exception-handling-exception-type-unexpectedexception-label`
       | 3. :ref:`exception-handling-exception-type-error-label`
   * - | (3)
     - | クライアントからの不正リクエストにより発生する例外
     - | クライアントからの不正リクエストにより発生する例外は、フレームワークで例外をハンドリングし、例外処理を行う。
     - | 1. :ref:`exception-handling-exception-type-request-label`

.. note:: **誰が、例外を意識する必要があるのか？**

  - (1)はアプリケーション開発者が意識する例外となる。
  - (2)と(3)はアプリケーションアーキテクトが意識する例外となる。


.. _exception-handling-method-label:

例外のハンドリング方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| アプリケーション実行時に発生する例外は、以下4つの方法でハンドリングを行う。
| ハンドリング方法毎のハンドリングフローの詳細は、\ :ref:`exception-handling-basic-flow-label`\ を参照されたい。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.35\linewidth}|p{0.25\linewidth}|
.. list-table:: **表-例外のハンドリング方法**
   :header-rows: 1
   :widths: 10 30 35 25
   :class: longtable

   * - 項番
     - ハンドリング方法
     - 説明
     - :ref:`exception-handling-pattern-label`
   * - | (1)
     - | アプリケーションコードにて、\ ``try-catch``\ を使い、例外ハンドリングを行う。
     - | リクエスト(Controllerのメソッド)単位に、例外をハンドリングする場合に使用する。
       | 詳細は、\ :ref:`exception-handling-basic-flow-catch-label`\ を参照されたい。
     - | 1. :ref:`exception-handling-class-from-middle-label`
   * - | (2)
     - | \ ``@ExceptionHandler``\ アノテーションを使い、アプリケーションコードで例外ハンドリングを行う。
     - | ユースケース(Controller)単位に、例外をハンドリングする場合に使用する。
       | 詳細は、\ :ref:`exception-handling-basic-flow-annotation-label`\ を参照されたい。
     - | 1. :ref:`exception-handling-class-from-first-label`
   * - | (3)
     - | フレームワークから提供されているHandlerExceptionResolverの仕組みを使い、例外ハンドリングを行う。
     - | サーブレット単位に、例外をハンドリングする場合に使用する。
       | HandlerExceptionResolverは、\ ``<mvc:annotation-driven>``\ を指定した際に、自動的に、\ :ref:`登録されるクラス<ExceptionHandling-annotation-driven>`\ と、共通ライブラリから提供している\ ``SystemExceptionResolver``\ を使用する。
       | 詳細は、\ :ref:`exception-handling-basic-flow-resolver-label`\ を参照されたい。
     - | 1. :ref:`exception-handling-class-systemerror-label`
       |
       | 2. :ref:`exception-handling-class-requesterror-label`
   * - | (4)
     - | サーブレットコンテナのerror-page機能を使い、例外ハンドリングを行う。
     - | 致命的なエラー、Spring MVC管理外で発生する例外をハンドリングする場合に使用する。
       | 詳細は、\ :ref:`exception-handling-basic-flow-container-label`\ を参照されたい。
     - | 1. :ref:`exception-handling-class-fatalerror-label`
       |
       | 2. :ref:`exception-handling-class-viewerror-label`

.. raw:: latex

   \newpage

.. figure:: ./images/exception-handling-method.png
  :alt: handling method
  :width: 80%
  :align: center

  **図-例外のハンドリング方法**


.. note:: **誰が例外ハンドリングを行うのか？**

  - (1)と(2)はアプリケーション開発者が設計・実装する。
  - (3)と(4)はアプリケーションアーキテクトが設計・設定する。

.. note:: **自動的に登録されるHandlerExceptionResolverについて**

  <mvc:annotation-driven> を指定した際に、自動的に登録されるHandlerExceptionResolverの役割は、以下の通りである。

  優先順は、以下の並び順の通りとなる。

  .. _ExceptionHandling-annotation-driven:

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.55\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 30 55

       * - 項番
         - クラス(優先順位)
         - 役割
       * - | (1)
         - | ExceptionHandlerExceptionResolver
           | (order=0)
         - | \ ``@ExceptionHandler``\ アノテーションが付与されているControllerクラスのメソッドを呼び出し、例外ハンドリングを行うためのクラス。
           | No.2のハンドリング方法を実現するために必要なクラス。
       * - | (2)
         - | ResponseStatusExceptionResolver
           | (order=1)
         - | クラスアノテーションとして、\ ``@ResponseStatus``\ が付与されている例外をハンドリングするためのクラス。
           | \ ``@ResponseStatus``\ に指定されている値で、\ ``HttpServletResponse#sendError(int sc, String msg)``\ が呼び出される。
       * - | (3)
         - | DefaultHandlerExceptionResolver
           | (order=2)
         - | Spring MVC内で発生するフレームワーク例外を、ハンドリングするためのクラス。
           | フレームワーク例外に対応するHTTPレスポンスコードの値で、\ ``HttpServletResponse#sendError(int sc)``\ が呼び出される。
           | 設定されるHTTPレスポンスコードの詳細は、\ :ref:`exception-handling-appendix-defaulthandlerexceptionresolver-label`\ を参照されたい。

.. note:: **共通ライブラリから提供している SystemExceptionResolver の役割は？**

  <mvc:annotation-driven> を指定した際に、自動的に登録されるHandlerExceptionResolverによって、
  ハンドリングされない例外をハンドリングするためのクラスである。
  そのため優先順は、DefaultHandlerExceptionResolverの後になるように設定する。


.. note:: **Spring Framework 3.2 より追加された@ControllerAdviceアノテーションについて**

  \ ``@ControllerAdvice``\ の登場により、サーブレット単位で、\ ``@ExceptionHandler``\ を使った例外ハンドリングを行えるようになった。
  \ ``@ControllerAdvice``\ アノテーションが付与されたクラスで、\ ``@ExceptionHandler``\ アノテーションを付与したメソッドを定義すると、サーブレット内のすべてのControllerに適用される。
  以前のバージョンで同じことを実現する場合、\ ``@ExceptionHandler``\ アノテーションが付与されたメソッドを、Controllerのベースクラスのメソッドとして定義し、
  各Controllerでベースクラスを継承する必要があった。

  **Spring Framework 4.0 より追加された@ControllerAdviceアノテーションの属性について**

  \ ``@ControllerAdvice``\ アノテーションの属性を指定することで、
  \ ``@ControllerAdvice``\ が付与されたクラスで実装したメソッドを適用するControllerを柔軟に指定できるように改善されている。
  属性の詳細については、\ :ref:`@ControllerAdviceの属性 <application_layer_controller_advice_attribute>`\ を参照されたい。

.. note:: **@ControllerAdviceアノテーションの使いどころ**

  #. サーブレット単位で行う例外ハンドリングに対して、View名と、HTTPレスポンスコードの解決以外の処理が必要な場合。
     （View名とHTTPレスポンスコードの解決のみでよい場合は、\ ``SystemExceptionResolver``\ で対応できる ）
  #. サーブレット単位で行う例外ハンドリングに対して、エラー応答用のレスポンスデータをJSPなどのテンプレートエンジンを使わずに、
     エラー用のモデル（JavaBeans）を、JSONやXML形式にシリアライズして生成したい場合
     （AJAXや、REST用のControllerを作成する際の、エラーハンドリングとして使用する）。


Detail
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. :ref:`exception-handling-exception-type-label`
#. :ref:`exception-handling-pattern-label`
#. :ref:`exception-handling-basic-flow-label`

|

.. _exception-handling-exception-type-label:

例外の種類
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
アプリケーション実行時に発生する例外は、以下6種類に分類される。


.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.30\linewidth}|
.. list-table:: **表-例外の種類**
   :header-rows: 1
   :widths: 10 20 30

   * - 項番
     - 例外の種類
     - 説明
   * - | (1)
     - | :ref:`exception-handling-exception-type-businessexception-label`
     - | ビジネスルールの違反を検知したことを通知する例外
   * - | (2)
     - | :ref:`exception-handling-exception-type-libraryexception-label`
     - | フレームワーク、およびライブラリ内で発生する例外のうち、システムが、正常稼働している時に発生する可能性のある例外
   * - | (3)
     - | :ref:`exception-handling-exception-type-systemexception-label`
     - | システムが、正常稼働している時に、発生してはいけない状態を検知したことを通知する例外
   * - | (4)
     - | :ref:`exception-handling-exception-type-unexpectedexception-label`
     - | システムが、正常稼働している時には発生しない非検査例外
   * - | (5)
     - | :ref:`exception-handling-exception-type-error-label`
     - | システム(アプリケーション)全体に影響を及ぼす、致命的な問題が発生していることを通知するエラー
   * - | (6)
     - | :ref:`exception-handling-exception-type-request-label`
     - | フレームワークが、リクエスト内容の不正を検知したことを通知する例外


.. _exception-handling-exception-type-businessexception-label:

ビジネス例外
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| **ビジネスルールの違反を検知したことを通知する例外** 。
| 本例外は、ドメイン層のロジック内で発生させる。
| アプリケーションとして想定される状態なので、システム運用者による対処は、不要である。

*  旅行を予約する際に予約日が期限を過ぎている場合
*  商品を注文する際に在庫切れの場合
*  etc ...

.. note:: **該当する例外クラス**

  - \ ``org.terasoluna.gfw.common.exception.BusinessException``\  (共通ライブラリから提供しているクラス)。
  - 細かくハンドリングする必要がある場合は、BusinessExceptionを継承した例外クラスを作成すること。
  - 共通ライブラリで用意しているビジネス例外クラスで、要件を満たせない場合は、プロジェクト毎にビジネス例外クラスを作成すること。


.. _exception-handling-exception-type-libraryexception-label:

正常稼働時に発生するライブラリ例外
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| フレームワーク、およびライブラリ内で発生する例外のうち、 **システムが、正常稼働している時に発生する可能性のある例外。**
| フレームワーク、およびライブラリ内で発生する例外とは、Spring Frameworkや、その他のライブラリ内で発生する例外クラスを対象とする。
| アプリケーションとして想定される状態なので、システム運用者による対処は、不要である。

* 複数のオペレータによって、同じデータを同時に更新しようとした場合に、発生する楽観排他例外や、悲観排他例外。
* 複数のオペレータによって、同じデータを同時に登録しようとした場合に、発生する一意制約違反例外。
* etc ...

.. note:: **該当する例外クラスの例**

  - ``org.springframework.dao.OptimisticLockingFailureException`` (楽観排他でエラーが発生した場合に発生する例外)。
  - ``org.springframework.dao.PessimisticLockingFailureException`` (悲観排他でエラーが発生した場合に発生する例外)。
  - ``org.springframework.dao.DuplicateKeyException`` (一意制約違反となった場合に発生する例外)。
  - etc ...

.. todo::

  **JPA(Hibernate)を使用すると、現状意図しないエラーとなることが発覚している。**

  * 一意制約違反が発生した場合、\ ``DuplicateKeyException``\ ではなく、\ ``org.springframework.dao.DataIntegrityViolationException``\ が発生する。


.. _exception-handling-exception-type-systemexception-label:

システム例外
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| **システムが、正常稼働している時に、発生してはいけない状態を検知したことを通知する例外** 。
| 本例外は、アプリケーション層、およびドメイン層のロジックで発生させる。
| システム運用者による対処が必要となる。

*  事前に存在しているはずのマスタデータ、ディレクトリ、ファイルなどが存在しない場合。
* フレームワーク、ライブラリ内で発生する検査例外のうち、システム異常に分類される例外を捕捉した場合(ファイル操作時のIOExceptionなど)。
* etc ...

.. note:: **該当する例外クラス**

  - \ ``org.terasoluna.gfw.common.exception.SystemException``\  (共通ライブラリから提供しているクラス)。
  - 遷移先のエラー画面や、HTTPレスポンスコードを細かく分ける場合は、SystemExceptionを継承した例外クラスを作成すること。
  - 共通ライブラリで用意しているシステム例外クラスだと要件を満たせない場合は、プロジェクト毎にシステム例外クラスを作成すること。


.. _exception-handling-exception-type-unexpectedexception-label:

予期しないシステム例外
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| **システムが、正常稼働している時には発生しない非検査例外。**
| システム運用者による対処、またはシステム開発者による解析が必要となる。
| **予期しないシステム例外は、アプリケーションコードでハンドリング(try-catch)すべきでない。**

* アプリケーション、フレームワーク、ライブラリにバグが潜んでいる場合。
* DBサーバなどがダウンしている場合。
* etc ...

.. note:: **該当する例外クラスの例**

  - \ ``java.lang.NullPointerException``\ (バグ起因で発生する例外)。
  - \ ``org.springframework.dao.DataAccessResourceFailureException``\ (DBサーバがダウンしている場合に発生する例外)。
  - etc ...


.. _exception-handling-exception-type-error-label:

致命的なエラー
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| **システム(アプリケーション)全体に影響を及ぼす、致命的な問題が発生している事を通知するエラー** 。
| システム運用者、またはシステム開発者による対処・リカバリが必要となる。
| **致命的なエラー（java.lang.Errorを継承しているエラーオブジェクト）は、アプリケーションコードでハンドリング(try-catch)してはいけない。**

* Java仮想マシンで使用できるメモリが不足している場合。
* etc ...

.. note:: **該当するエラークラスの例**

  - \ ``java.lang.OutOfMemoryError``\ (メモリ不足時に発生するエラー)。
  - etc ...


.. _exception-handling-exception-type-request-label:

リクエスト不正時に発生するフレームワーク例外
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| **フレームワークが、リクエスト内容の不正を検知したことを通知する例外** 。
| 本例外は、フレームワーク(Spring MVC)内で発生する。
| 原因は、クライアント側に存在するため、システム運用者による対処は、不要である。

* POSTメソッドのみ許容しているリクエストパスに対して、GETメソッドでアクセスした場合に発生する例外。
* \ ``@PathVariable``\ アノテーションを使って、URIから値を抽出する際に、URIに型変換できない値が、指定された場合に発生する例外。
* etc ...

.. note:: **該当する例外クラスの例**

  - \ ``org.springframework.web.HttpRequestMethodNotSupportedException``\ (サポート外のメソッドでアクセスされた場合に発生する例外)。
  - \ ``org.springframework.beans.TypeMismatchException``\ (URIに型変換できない値が指定された場合に発生する例外)。
  - etc ...

 \ :ref:`exception-handling-appendix-defaulthandlerexceptionresolver-label`\ の中の、HTTPステータスコードが「4XX」の例外が該当するクラス。


.. _exception-handling-pattern-label:

例外ハンドリングのパターン
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 例外ハンドリングは、目的に応じて、以下6種類のパターンに分類される。
| (1)-(2)はユースケース毎、(3)-(6)はシステム(アプリケーション)全体でハンドリングを行う。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.40\linewidth}|p{0.25\linewidth}|p{0.10\linewidth}|p{0.15\linewidth}|
.. list-table:: **表-例外ハンドリングのパターン**
   :header-rows: 1
   :widths: 10 40 25 10 15
   :class: longtable

   * - 項番
     - ハンドリングの目的
     - ハンドリング対象となり得る例外
     - ハンドリング方法
     - ハンドリング単位
   * - | (1)
     - | :ref:`exception-handling-class-from-middle-label`
     - | 1. :ref:`exception-handling-exception-type-businessexception-label`
     - | アプリケーションコード
       | (try-catch)
     - | リクエスト
   * - | (2)
     - | :ref:`exception-handling-class-from-first-label`
     - | 1. :ref:`exception-handling-exception-type-businessexception-label`
       | 2. :ref:`exception-handling-exception-type-libraryexception-label`
     - | アプリケーションコード
       | (@ExceptionHandler)
     - | ユースケース
   * - | (3)
     - | :ref:`exception-handling-class-systemerror-label`
     - | 1. :ref:`exception-handling-exception-type-systemexception-label`
       | 2. :ref:`exception-handling-exception-type-unexpectedexception-label`
     - | フレームワーク
       | (ハンドリングルールを、\ ``spring-mvc.xml``\ に指定する)
     - | サーブレット
   * - | (4)
     - | :ref:`exception-handling-class-requesterror-label`
     - | 1. :ref:`exception-handling-exception-type-request-label`
     - | フレームワーク
     - | サーブレット
   * - | (5)
     - | :ref:`exception-handling-class-fatalerror-label`
     - | 1. :ref:`exception-handling-exception-type-error-label`
     - | サーブレットコンテナ
       | (ハンドリングルールを、\ ``web.xml``\ に指定する)
     - | Webアプリケーション
   * - | (6)
     - | :ref:`exception-handling-class-viewerror-label`
     - | 1. プレゼンテーション層で発生する全ての例外及びエラー
     - | サーブレットコンテナ
       | (ハンドリングルールを、\ ``web.xml``\ に指定する)
     - | Webアプリケーション

.. raw:: latex

   \newpage

.. _exception-handling-class-from-middle-label:

ユースケースの一部やり直し(途中からのやり直し)を促す場合
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
ユースケースの一部やり直し(途中からのやり直し)を促す場合は、Controllerクラスのアプリケーションコードで捕捉(try-catch)し、リクエスト単位で例外処理を行う。

.. note:: **ユースケースの一部やり直しを促す場合の例**

  - | ショッピングサイトで注文処理を行った際に、在庫不足を通知するビジネス例外が発生する場合。
    | このケースの場合、個数を減らせば注文処理が行えるため、個数が変更できる画面に遷移し、個数変更を促すメッセージを表示する。
  - etc ...

.. figure:: ./images/exception-handling-class-again-from-middle.png
  :alt: class of exception handling for again from middle
  :width: 80%
  :align: center

  **図-ユースケースの一部やり直し(途中からのやり直し)を促す場合のハンドリング方法**


.. _exception-handling-class-from-first-label:

ユースケースのやり直し(先頭からのやり直し)を促す場合
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
ユースケースのやり直し(先頭からのやり直し)を促す場合は、\ ``@ExceptionHandler``\ を使って捕捉し、ユースケース単位で例外処理を行う。

.. note:: **ユースケースのやり直し（先頭からのやり直し）を促す場合の例**

  - | ショッピングサイト(管理者向けサイト)で商品マスタの変更を行った際に、変更対象の商品マスタが他のオペレータによって変更されていた場合（楽観排他例外が発生した場合）。
    | このケースの場合、他のユーザが行った変更内容を確認してから操作してもらう必要があるため、ユースケースの先頭画面（例えば商品マスタの検索画面）に遷移し、再操作を促すメッセージを表示する。
  - etc ...

.. figure:: ./images/exception-handling-class-again-from-first.png
  :alt: class of exception handling for again from first
  :width: 80%
  :align: center

  **図-ユースケースのやり直し(先頭からのやり直し)を促す場合のハンドリング方法**


.. _exception-handling-class-systemerror-label:

システム、またはアプリケーションが、正常な状態でない事を通知する場合
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
システム、またはアプリケーションが、正常な状態でないことを通知する例外を検知する場合は、SystemExceptionResolverで捕捉し、サーブレット単位で例外処理を行う。

.. note:: **システム、またはアプリケーションが正常な状態でないことを通知する場合の例**

  - | 外部システムとの接続を行うユースケースにて、外部システムが、閉塞中であることを通知する例外が発生した場合。
    | このケースの場合、外部システムが開局するまで実行できないため、エラー画面に遷移し、外部システムが開局するまでユースケースが実行できない旨を通知する。
  - | アプリケーションで指定した値を、条件にマスタ情報の検索を行った際に、該当するマスタ情報が存在しない場合。
    | このケースの場合、マスタメンテナンス機能のバグ又はシステム運用者によるデータ投入ミス(リリースミス)の可能性があるため、システムエラー画面に遷移し、システム異常が発生した旨を通知する。
  - | ファイル操作時にAPIからIOExceptionが発生した場合。
    | このケースの場合、ディスク異常などが考えられるため、システムエラー画面に遷移し、システム異常が発生した旨を通知する。
  - etc ...


.. figure:: ./images/exception-handling-class-systemerror.png
  :alt: class of exception handling for system error
  :width: 80%
  :align: center

  **図-システム、またはアプリケーションが、正常な状態でないことを通知する例外を検知する場合のハンドリング方法**


.. _exception-handling-class-requesterror-label:

リクエスト内容が、不正であることを通知する場合
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
フレームワークによって、検知されたリクエスト不正を通知する場合は、DefaultHandlerExceptionResolverで捕捉し、サーブレット単位で例外処理を行う。

.. note:: **リクエスト内容が不正であることを通知する場合の例**

  - | POSTメソッドのみ許可されているURIで、GETメソッドを使ってアクセスした場合。
    | このケースの場合、ブラウザのお気に入り機能などを使って直接アクセスしている事が考えられるため、エラー画面に遷移し、リクエスト内容が不正であることを通知する。
  - | ``@PathVariable`` アノテーションを使ってURIから値を抽出する際に、URIから値を抽出できなかった場合。
    | このケースの場合、ブラウザのアドレスバーの値を書き換えて、直接アクセスしている事が考えられるため、エラー画面に遷移し、リクエスト内容が不正であることを通知する。
  - etc ...


.. figure:: ./images/exception-handling-class-requesterror.png
  :alt: class of exception handling for request error
  :width: 80%
  :align: center

  **図-リクエスト内容が不正であることを通知する場合のハンドリング方法**


.. _exception-handling-class-fatalerror-label:

致命的なエラーが発生したことを検知する場合
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
致命的なエラーが発生したことを検知する場合、サーブレットコンテナで捕捉し、Webアプリケーション単位で例外処理を行う。

.. figure:: ./images/exception-handling-class-fatalerror.png
  :alt: class of exception handling for fatal error
  :width: 80%
  :align: center

  **図-致命的なエラーが発生したことを検知する場合のハンドリング方法**


.. _exception-handling-class-viewerror-label:

プレゼンテーション層(JSPなど)で、例外が発生したことを通知する場合
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
プレゼンテーション層(JSPなど)で、例外が発生したことを通知する場合、サーブレットコンテナで捕捉し、Webアプリケーション単位で例外処理を行う。

.. figure:: ./images/exception-handling-class-jsperror.png
  :alt: class of exception handling for fatal error
  :width: 80%
  :align: center

  **図-プレゼンテーション層(JSPなど)で例外が発生した事を通知する場合のハンドリング方法**


.. _exception-handling-basic-flow-label:

例外ハンドリングの基本フロー
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 例外処理の基本フローを示す。
| 共通ライブラリから提供しているクラスの概要については、\ :ref:`exception-handling-about-classes-of-library-label`\ を参照されたい。
| **アプリケーションコードで行う処理(実装が必要な処理)についての説明は、太字で表現している。**
| 例外メッセージ、およびスタックトレースのログ出力は、共通ライブラリから提供しているクラス（FilterやInterceptorクラス）で行う。
| 例外メッセージ、およびスタックトレース以外の情報を、ログ出力する必要がある場合は、各ロジックで個別にログを出力すること。
| 例外ハンドリングのフロー説明であるため、Serviceクラスを呼び出すまでのフローに関する説明は、省略する。

#. :ref:`exception-handling-basic-flow-catch-label`
#. :ref:`exception-handling-basic-flow-annotation-label`
#. :ref:`exception-handling-basic-flow-resolver-label`
#. :ref:`exception-handling-basic-flow-container-label`


.. _exception-handling-basic-flow-catch-label:

リクエスト単位でControllerクラスがハンドリングする場合の基本フロー
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| 例外をリクエスト単位でハンドリングする場合、Controllerクラスのアプリケーションコードで捕捉(try-catch)し、例外処理を行う。
| 基本フローは、以下の通りである。
| 下記の図は、 共通ライブラリから提供しているビジネス例外(\ ``org.terasoluna.gfw.common.exception.BusinessException``\ )をハンドリングする場合の基本フローである。
| ログは、結果メッセージを保持している例外が発生したことを記録するインタセプタ(\ ``org.terasoluna.gfw.common.exception.ResultMessagesLoggingInterceptor``\ )を使用して、出力する。

.. figure:: ./images/exception-handling-flow-catch.png
  :alt: flow of exception handling using catch
  :width: 80%
  :align: center

  **図-リクエスト単位でControllerクラスがハンドリングする場合の基本フロー**

4. Serviceクラスにて、 BusinessExceptionを生成し、スローする。
#. ResultMessagesLoggingInterceptorは、 ExceptionLoggerを呼び出し、warnレベルのログ(監視ログとアプリケーションログ)を出力する。
   ResultMessagesLoggingInterceptorはResultMessagesNotificationExceptionのサブ例外(BusinessException/ResourceNotFoundException)が発生した場合のみ、ログを出力するクラスである。
#. **Controllerクラスは、 BusinessExceptionを捕捉し、 BusinessExceptionに設定されているメッセージ情報(ResultMessage)を画面表示用にModelに設定する(6')。**
#. **Controllerクラスは、遷移先のView名を返却する。**
#. DispatcherServletは、返却されたView名に対応するJSPを呼び出す。
#. **JSPは、MessagesPanelTagを使用して、メッセージ情報(ResultMessage)を取得し、メッセージ表示用のHTMLコードを生成する。**
#. JSPで生成されたレスポンスが表示される。


.. _exception-handling-basic-flow-annotation-label:

ユースケース単位でControllerクラスがハンドリングする場合の基本フロー
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| 例外をユースケース単位でハンドリングする場合、Controllerクラスの\ ``@ExceptionHandler``\ を使って捕捉し、例外処理を行う。
| 基本フローは、以下の通りである。
| 下記の図は、 任意の例外(\ ``XxxException``\ )をハンドリングする場合の、基本フローである。
| ログは、HandlerExceptionResolverによって、例外ハンドリングすることを記録するインタセプタ(\ ``org.terasoluna.gfw.web.exception.HandlerExceptionResolverLoggingInterceptor``\ )を使用して、出力する。

.. figure:: ./images/exception-handling-flow-annotation.png
  :alt: flow of exception handling using annotation
  :width: 80%
  :align: center

  **図-ユースケース単位で、Controllerクラスがハンドリングする場合の基本フロー**

3. Controllerクラスから呼び出されたServiceクラスにて、例外(XxxException)が発生する。
#. DispatcherServletは、XxxExceptionを捕捉し、ExceptionHandlerExceptionResolverを呼び出す。
#. ExceptionHandlerExceptionResolverは、Controllerクラスに用意されている例外ハンドリングメソッドを呼び出す。
#. **Controllerクラスは、メッセージ情報(ResultMessage)を生成し、画面表示用としてModelに設定する。**
#. **Controllerクラスは、遷移先のView名を返却する。**
#. ExceptionHandlerExceptionResolverは、Controllerより返却されたView名を返却する。
#. HandlerExceptionResolverLoggingInterceptorは、ExceptionLoggerを呼び出し、例外コードに対応するレベル(info, warn, error)のログ(監視ログとアプリケーションログ)を出力する。
#. HandlerExceptionResolverLoggingInterceptorは、ExceptionHandlerExceptionResolverより返却されたView名を返却する。
#. DispatcherServletは、返却されたView名に対応するJSPを呼び出す。
#. **JSPは、MessagesPanelTagを使用して、メッセージ情報(ResultMessage)を取得し、メッセージ表示用のHTMLコードを生成する。**
#. JSPで生成されたレスポンスが表示される。


.. _exception-handling-basic-flow-resolver-label:

サーブレット単位でフレームワークがハンドリングする場合の基本フロー
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| 例外をフレームワーク(サーブレット単位)でハンドリングする場合、SystemExceptionResolverで捕捉し例外処理を行う。
| 基本フローは、以下の通りである。
| 下記の図は、 共通ライブラリから提供しているシステム例外(\ ``org.terasoluna.gfw.common.exception.SystemException``\ )を、\ ``org.terasoluna.gfw.web.exception.SystemExceptionResolver``\ を使ってハンドリングする場合の基本フローである。
| ログは、例外ハンドリングメソッドの引数に指定された例外を記録するインタセプタ(\ ``org.terasoluna.gfw.web.exception.HandlerExceptionResolverLoggingInterceptor``\ )を使用して、出力する。

.. figure:: ./images/exception-handling-flow-resolver.png
  :alt: flow of exception handling using resolver
  :width: 80%
  :align: center

  **図-サーブレット単位でフレームワークがハンドリングする場合の基本フロー**

4. Serviceクラスにて、システム例外に該当する状態を検知したため、SystemExceptionを発生させる。
#. DispatcherServletは、SystemExceptionを捕捉し、SystemExceptionResolverを呼び出す。
#. SystemExceptionResolverは、SystemExceptionから例外コードを取得し、画面表示用にHttpServletRequestに設定する(6')。
#. SystemExceptionResolverは、SystemException発生時の遷移先のView名を返却する。
#. HandlerExceptionResolverLoggingInterceptorは、ExceptionLoggerを呼び出し、例外コードに対応するレベル(info, warn, error)のログ(監視ログとアプリケーションログ)を出力する。
#. HandlerExceptionResolverLoggingInterceptorは、SystemExceptionResolverより返却されたView名を返却する。
#. DispatcherServletは、返却されたView名に対応するJSPを呼び出す。
#. **JSPは、HttpServletRequestより例外コードを取得し、メッセージ表示用のHTMLコードに埋め込む。**
#. JSPで生成されたレスポンスが表示される。


.. _exception-handling-basic-flow-container-label:

Webアプリケーション単位でサーブレットコンテナがハンドリングする場合の基本フロー
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| 例外をWebアプリケーション単位でハンドリングする場合、サーブレットコンテナで捕捉し、例外処理を行う。
| 致命的なエラー、フレームワークでハンドリング対象となっていない例外(JSP内で発生した例外など)、Filterで発生した例外をハンドリングする。
| 基本フローは以下の通りである。
| 下記フローは、java.lang.Exceptionを、"error page"でハンドリングする場合のフローである。
| ログ出力は、ハンドリングされていない例外が発生したことを記録するサーブレットフィルタ(\ ``org.terasoluna.gfw.web.exception.ExceptionLoggingFilter``\ )を使用して、出力する。

.. figure:: ./images/exception-handling-flow-container.png
  :alt: flow of exception handling using container
  :width: 80%
  :align: center

  **図-Webアプリケーション単位でサーブレットコンテナがハンドリングする場合の基本フロー**

4. DispatcherServletは、XxxErrorを捕捉し、ServletExceptionにラップしてスローする。
#. ExceptionLoggingFilterは、ServletExceptionを捕捉し、ExceptionLoggerを呼び出す。ExceptionLoggerは、例外コードに対応するレベル(info, warn, error)のログ(監視ログとアプリケーションログ)を出力する。ExceptionLoggingFilterは、ServletExceptionを再スローする。
#. ServletContainerは、ServletExceptionを捕捉し、サーバログにログを出力する。ログのレベルは、アプリケーションサーバによって異なる。
#. ServletContainerは、``web.xml`` に定義されている遷移先(HTMLなど)を呼び出す。
#. 呼び出された遷移先で生成されたレスポンスが表示される。

|

.. _exception-handling-how-to-use-label:

How to use
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
例外ハンドリング機能の使用方法について説明する。

共通ライブラリから提供している例外ハンドリング用のクラスについては、\ :ref:`exception-handling-about-classes-of-library-label`\ を参照されたい。

#. :ref:`exception-handling-how-to-use-application-configuration-label`
#. :ref:`exception-handling-how-to-use-codingpoint-service-label`
#. :ref:`exception-handling-how-to-use-codingpoint-controller-label`
#. :ref:`exception-handling-how-to-use-codingpoint-jsp-label`


.. _exception-handling-how-to-use-application-configuration-label:

アプリケーションの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 例外ハンドリングを使用する際に、必要なアプリケーション設定を、以下に示す。
| なお、ブランクプロジェクトは、既に設定済みの状態になっているので、\ **【プロジェクト毎にカスタマイズする箇所】**\ の部分を変更すればよい。

#. :ref:`exception-handling-how-to-use-application-configuration-common-label`
#. :ref:`exception-handling-how-to-use-application-configuration-domain-label`
#. :ref:`exception-handling-how-to-use-application-configuration-app-label`
#. :ref:`exception-handling-how-to-use-application-configuration-container-label`


.. _exception-handling-how-to-use-application-configuration-common-label:

共通設定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

１． 例外のログ出力を行うロガークラス（\ ``ExceptionLogger``\ ）を、bean定義に追加する。

- **applicationContext.xml**

 .. code-block:: xml
    :emphasize-lines: 3,5,11,16-17

    <!-- Exception Code Resolver. -->
    <bean id="exceptionCodeResolver"
        class="org.terasoluna.gfw.common.exception.SimpleMappingExceptionCodeResolver"> <!-- (1) -->
        <!-- Setting and Customization by project. -->
        <property name="exceptionMappings"> <!-- (2) -->
            <map>
                <entry key="ResourceNotFoundException" value="e.xx.fw.5001" />
                <entry key="BusinessException" value="e.xx.fw.8001" />
            </map>
        </property>
        <property name="defaultExceptionCode" value="e.xx.fw.9001" /> <!-- (3) -->
    </bean>

    <!-- Exception Logger. -->
    <bean id="exceptionLogger"
        class="org.terasoluna.gfw.common.exception.ExceptionLogger"> <!-- (4) -->
        <property name="exceptionCodeResolver" ref="exceptionCodeResolver" /> <!-- (5) -->
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``ExceptionCodeResolver``\ を、bean定義に追加する。
    * - | (2)
      - | ハンドリング対象とする例外名と、適用する「例外コード(メッセージID)」のマッピングを指定する。
        | 上記の設定例では、例外クラス(又は親クラス)のクラス名に、"BusinessException"が含まれている場合は、"w.xx.fw.8001"、 "ResourceNotFoundException"が含まれている場合は、"w.xx.fw.5001"が「例外コード(メッセージID)」となる。

        .. note:: **例外コード(メッセージID)について**

             ここでは、"BusinessException"に、メッセージIDが指定されなかった場合の対応で定義をしているが、
             後述の"BusinessException"を発生させる実装側で、「例外コード(メッセージID)」を指定することを推奨する。
             "BusinessException"に対する「例外コード(メッセージID)」の指定は、"BusinessException"発生時に指定されなかった場合の救済策である。

        | **【プロジェクト毎にカスタマイズする箇所】**
    * - | (3)
      - | デフォルトの「例外コード(メッセージID)」を指定する。
        | 上記の設定例では、例外クラス(または親クラス)のクラス名に"BusinessException"、または"ResourceNotFoundException"が含まれない場合、"e.xx.fw.9001"が例外コード(メッセージID)」となる。
        | **【プロジェクト毎にカスタマイズする箇所】**

        .. note:: **例外コード(メッセージID)について**

             例外コードは、ログに出力するのみ。（画面での取得もできる。JSPへのリンク）
             プロパティに定義している形式でなくとも、運用上でわかるIDにすることが可能である。
             例えば、MA7001等

    * - | (4)
      - | \ ``ExceptionLogger``\ を、bean定義に追加する。
    * - | (5)
      - | \ ``ExceptionCodeResolver``\ をDIする。

 .. raw:: latex

    \newpage

２． ログ定義を追加する。

- **logback.xml**

 監視ログ用のログ定義を追加する。

 .. code-block:: xml
    :emphasize-lines: 1,13-15

    <appender name="MONITORING_LOG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender"> <!-- (1) -->
        <file>${app.log.dir:-log}/projectName-monitoring.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${app.log.dir:-log}/projectName-monitoring-%d{yyyyMMdd}.log</fileNamePattern>
            <maxHistory>7</maxHistory>
        </rollingPolicy>
        <encoder>
            <charset>UTF-8</charset>
            <pattern><![CDATA[date:%d{yyyy-MM-dd HH:mm:ss}\tX-Track:%X{X-Track}\tlevel:%-5level\tmessage:%msg%n]]></pattern>
        </encoder>
    </appender>

    <logger name="org.terasoluna.gfw.common.exception.ExceptionLogger.Monitoring" additivity="false"> <!-- (2) -->
        <level value="error" /> <!-- (3) -->
        <appender-ref ref="MONITORING_LOG_FILE" /> <!-- (4) -->
    </logger>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 監視ログを出力するための、appender定義を指定する。上記の設定例では、ファイルに出力するappenderとしているが、システム要件に一致するappenderを使うこと。
        | **【プロジェクト毎にカスタマイズする箇所】**
    * - | (2)
      - | 監視ログ用の、ロガー定義を指定する。ExceptionLoggerを作成する際に、任意のロガー名を指定していない場合は、上記設定のままでよい。

        .. warning:: **additivityの設定値について**

            \ ``false``\ を指定すること。\ ``true``\ を指定すると、上位のロガー(例えば、root)によって、同じログが出力されてしまう。

    * - | (3)
      - | 出力レベルを指定する。ExceptionLoggerではinfo, warn, errorの3種類のログを出力しているが、システム要件にあったレベルを指定すること。errorレベルを推奨する。
        | **【プロジェクト毎にカスタマイズする箇所】**
    * - | (4)
      - | 出力先となるappenderを指定する。
        | **【プロジェクト毎にカスタマイズする箇所】**


 アプリケーションログ用のログ定義を追加する。

 .. code-block:: xml
    :emphasize-lines: 1,13

    <appender name="APPLICATION_LOG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender"> <!-- (1) -->
        <file>${app.log.dir:-log}/projectName-application.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${app.log.dir:-log}/projectName-application-%d{yyyyMMdd}.log</fileNamePattern>
            <maxHistory>7</maxHistory>
        </rollingPolicy>
        <encoder>
            <charset>UTF-8</charset>
            <pattern><![CDATA[date:%d{yyyy-MM-dd HH:mm:ss}\tthread:%thread\tX-Track:%X{X-Track}\tlevel:%-5level\tlogger:%-48logger{48}\tmessage:%msg%n]]></pattern>
        </encoder>
    </appender>

    <logger name="org.terasoluna.gfw.common.exception.ExceptionLogger"> <!-- (2) -->
        <level value="info" /> <!-- (3) -->
    </logger>

    <root level="warn">
        <appender-ref ref="STDOUT" />
        <appender-ref ref="APPLICATION_LOG_FILE" /> <!-- (4) -->
    </root>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | アプリケーションログを出力するための、appender定義を指定する。上記の設定例では、ファイルに出力するappenderとしているが、システム要件に一致するappenderを使うこと。
        | **【プロジェクト毎にカスタマイズする箇所】**
    * - | (2)
      - | アプリケーションログ用の、ロガー定義を指定する。ExceptionLoggerを作成する際に、任意のロガー名を指定していない場合は、上記設定のままでよい。

        .. note:: **アプリケーションログ出力用のappender定義について**

             アプリケーションログ用のappenderは、例外出力用に個別に定義するのではなく、フレームワークや、アプリケーションコードで出力するログ用のappenderと、同じものを使うことを推奨する。
             同じ出力先にすることで、例外の発生するまでの過程が追いやすくなる。

    * - | (3)
      - | 出力レベルを指定する。ExceptionLoggerでは、info, warn, errorの3種類のログを出力しているが、システム要件にあったレベルを指定すること。本ガイドラインでは、infoレベルを推奨する。
        | **【プロジェクト毎にカスタマイズする箇所】**
    * - | (4)
      - | (2)で設定したロガーは、appenderを指定していないので、rootに流れる。そのため、出力先となるappenderを指定する。ここでは、"STDOUT"と"APPLICATION_LOG_FILE"に出力される。
        | **【プロジェクト毎にカスタマイズする箇所】**


.. _exception-handling-how-to-use-application-configuration-domain-label:

ドメイン層の設定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
ResultMessagesを保持する例外(BisinessException,ResourceNotFoundException)が発生した際に、ログを出力するためのインタセプタクラス（\ ``ResultMessagesLoggingInterceptor``\ ）と、AOPの設定を、bean定義に追加する。

- **xxx-domain.xml**

.. _exception-handling-how-to-use-service-pointcut-aop-label:

 .. code-block:: xml
    :emphasize-lines: 3,4,10

    <!-- interceptor bean. -->
    <bean id="resultMessagesLoggingInterceptor"
          class="org.terasoluna.gfw.common.exception.ResultMessagesLoggingInterceptor"> <!-- (1) -->
          <property name="exceptionLogger" ref="exceptionLogger" /> <!-- (2) -->
    </bean>

    <!-- setting AOP. -->
    <aop:config>
        <aop:advisor advice-ref="resultMessagesLoggingInterceptor"
                     pointcut="@within(org.springframework.stereotype.Service)" /> <!-- (3) -->
    </aop:config>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``ResultMessagesLoggingInterceptor``\ を、bean定義に追加する。
    * - | (2)
      - | 例外のログ出力を行うロガーオブジェクトをDIする。\ ``applicationContext.xml``\ に定義している "exceptionLogger" を指定する。
    * - | (3)
      - | Serviceクラス(\ ``@Service``\ アノテーションが付いているクラス)のメソッドに対して、ResultMessagesLoggingInterceptorを適用する。


.. _exception-handling-how-to-use-application-configuration-app-label:

アプリケーション層の設定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
<mvc:annotation-driven> を指定した際に、自動的に登録されるHandlerExceptionResolverによって、ハンドリングされない例外をハンドリングするためのクラス（\ ``SystemExceptionResolver``\ ）を、bean定義に追加する。

- **spring-mvc.xml**

 .. code-block:: xml
    :emphasize-lines: 3-4,6-7,15,23-24,29

    <!-- Setting Exception Handling. -->
    <!-- Exception Resolver. -->
    <bean class="org.terasoluna.gfw.web.exception.SystemExceptionResolver"> <!-- (1) -->
        <property name="exceptionCodeResolver" ref="exceptionCodeResolver" /> <!-- (2) -->
        <!-- Setting and Customization by project. -->
        <property name="order" value="3" /> <!-- (3) -->
        <property name="exceptionMappings"> <!-- (4) -->
            <map>
                <entry key="ResourceNotFoundException" value="common/error/resourceNotFoundError" />
                <entry key="BusinessException" value="common/error/businessError" />
                <entry key="InvalidTransactionTokenException" value="common/error/transactionTokenError" />
                <entry key=".DataAccessException" value="common/error/dataAccessError" />
            </map>
        </property>
        <property name="statusCodes"> <!-- (5) -->
            <map>
                <entry key="common/error/resourceNotFoundError" value="404" />
                <entry key="common/error/businessError" value="409" />
                <entry key="common/error/transactionTokenError" value="409" />
                <entry key="common/error/dataAccessError" value="500" />
            </map>
        </property>
        <property name="defaultErrorView" value="common/error/systemError" /> <!-- (6) -->
        <property name="defaultStatusCode" value="500" /> <!-- (7) -->
    </bean>

    <!-- Settings View Resolver. -->
    <mvc:view-resolvers>
        <mvc:jsp prefix="/WEB-INF/views/" /> <!-- (8) -->
    </mvc:view-resolvers>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``SystemExceptionResolver``\ を、bean定義に追加する。
    * - | (2)
      - | 例外コード(メッセージID)を解決するオブジェクトをDIする。\ ``applicationContext.xml``\ に定義している、\ "exceptionCodeResolver"\ を指定する。
    * - | (3)
      - | ハンドリングの優先順位を指定する。値は、基本的に「3」で良い。\ ``<mvc:annotation-driven>``\ を指定した際に、自動的に、\ :ref:`登録されるクラス<ExceptionHandling-annotation-driven>`\ の方が、優先順位が上となる。

        .. hint:: **DefaultHandlerExceptionResolverで行われる例外ハンドリングを無効化する方法**

            \ ``DefaultHandlerExceptionResolver``\ で例外ハンドリングされた場合、HTTPレスポンスコードは設定されるが、Viewの解決がされないため、Viewの解決は、\ ``web.xml``\ のError Pageで行う必要がある。
            Viewの解決を\ ``web.xml``\ ではなく、\ ``HandlerExceptionResolver``\ で行いたい場合は、\ ``SystemExceptionResolver``\ の優先順位を「1」にすると、\ ``DefaultHandlerExceptionResolver``\ より前にハンドリング処理を実行することができる。
            \ ``DefaultHandlerExceptionResolver``\ でハンドリングされた場合の、HTTPレスポンスコードのマッピングについては、\ :ref:`exception-handling-appendix-defaulthandlerexceptionresolver-label`\ を参照されたい。

    * - | (4)
      - | ハンドリング対象とする例外名と、遷移先となるView名のマッピングを指定する。
        | 上記の設定では、例外クラス(または親クラス)のクラス名に".DataAccessException"が含まれている場合、"common/error/dataAccessError"が、遷移先のView名となる。
        | 例外クラスが"ResourceNotFoundException"の場合、"common/error/resourceNotFoundError"が、遷移先のView名となる。
        | **【プロジェクト毎にカスタマイズする箇所】**
    * - | (5)
      - | 遷移先となるView名と、HTTPステータスコードのマッピングを指定する。
        | 上記の設定では、View名が"common/error/resourceNotFoundError"の場合に、"404(Not Found)"がHTTPステータスコードとなる。
        | **【プロジェクト毎にカスタマイズする箇所】**
    * - | (6)
      - | 遷移するデフォルトのView名を、指定する。
        | 上記の設定では、例外クラスに"ResourceNotFoundException"、"BusinessException"、"InvalidTransactionTokenException"や例外クラス(または親クラス)のクラス名に、".DataAccessException"が含まれない場合、"common/error/systemError"が、遷移先のView名となる。
        | **【プロジェクト毎にカスタマイズする箇所】**
    * - | (7)
      - | レスポンスヘッダに設定するHTTPステータスコードのデフォルト値を指定する。 **"500"(Internal Server Error)** を設定することを推奨する。

        .. warning:: **指定を省略した場合の挙動**

            \ **"200"(OK)**\ 扱いになるので、注意すること。

    * - | (8)
      - 実際に遷移する\ ``View``\ は、\ ``ViewResolver``\ の設定に依存する。

        上記の設定では、

        * ``/WEB-INF/views/common/error/systemError.jsp``
        * ``/WEB-INF/views/common/error/resourceNotFoundError.jsp``
        * ``/WEB-INF/views/common/error/businessError.jsp``
        * ``/WEB-INF/views/common/error/transactionTokenError.jsp``
        * ``/WEB-INF/views/common/error/dataAccessError.jsp``

        が遷移先となる。

        .. tip::

            \ ``<mvc:view-resolvers>``\ 要素はSpring Framework 4.1から追加されたXML要素である。
            \ ``<mvc:view-resolvers>``\ 要素を使用すると、\ ``ViewResolver``\ をシンプルに定義することが出来る。

            従来通り\ ``<bean>``\ 要素を使用した場合の定義例を以下に示す。

             .. code-block:: xml

                <bean id="viewResolver"
                    class="org.springframework.web.servlet.view.InternalResourceViewResolver">
                    <property name="prefix" value="/WEB-INF/views/" />
                    <property name="suffix" value=".jsp" />
                </bean>

 .. raw:: latex

    \newpage

\ ``HandlerExceptionResolver``\ でハンドリングされた例外を、ログに出力するためのインタセプタクラス（\ ``HandlerExceptionResolverLoggingInterceptor``\ ）と、AOPの設定を、bean定義に追加する。

- **spring-mvc.xml**

 .. code-block:: xml
    :emphasize-lines: 3,4,8

    <!-- Setting AOP. -->
    <bean id="handlerExceptionResolverLoggingInterceptor"
        class="org.terasoluna.gfw.web.exception.HandlerExceptionResolverLoggingInterceptor"> <!-- (1) -->
        <property name="exceptionLogger" ref="exceptionLogger" /> <!-- (2) -->
    </bean>
    <aop:config>
        <aop:advisor advice-ref="handlerExceptionResolverLoggingInterceptor"
            pointcut="execution(* org.springframework.web.servlet.HandlerExceptionResolver.resolveException(..))" /> <!-- (3) -->
    </aop:config>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``HandlerExceptionResolverLoggingInterceptor``\ を、bean定義に追加する。
    * - | (2)
      - | 例外のログ出力を行うロガーオブジェクトを、DIする。\ ``applicationContext.xml``\ に定義している\ "exceptionLogger"\ を指定する。
    * - | (3)
      - | \ ``HandlerExceptionResolver``\ インタフェースのresolveExceptionメソッドに対して、\ ``HandlerExceptionResolverLoggingInterceptor``\ を適用する。
        |
        | デフォルトの設定では、共通ライブラリから提供している ``org.terasoluna.gfw.common.exception.ResultMessagesNotificationException`` のサブクラスの例外は、このクラスで行われるログ出力の対象外となっている。
        | ``ResultMessagesNotificationException`` のサブクラスの例外をログ出力対象外としている理由は、 ``org.terasoluna.gfw.common.exception.ResultMessagesLoggingInterceptor`` によってログ出力されるためである。
        | デフォルトの設定を変更する必要がある場合は、 :ref:`exception-handling-about-handlerexceptionresolverlogginginterceptor` を参照されたい。

致命的なエラー、Spring MVC管理外で発生する例外を、ログに出力するためのFilterクラス（\ ``ExceptionLoggingFilter``\ ）を、bean定義と\ ``web.xml``\ に追加する。

- **applicationContext.xml**

 .. code-block:: xml
    :emphasize-lines: 3,4

    <!-- Filter. -->
    <bean id="exceptionLoggingFilter"
        class="org.terasoluna.gfw.web.exception.ExceptionLoggingFilter" > <!-- (1) -->
        <property name="exceptionLogger" ref="exceptionLogger" /> <!-- (2) -->
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``ExceptionLoggingFilter``\ を、bean定義に追加する。
    * - | (2)
      - | 例外のログ出力を行うロガーオブジェクトを、DIする。\ ``applicationContext.xml``\ に定義している\ "exceptionLogger"\ を指定する。


- **web.xml**

 .. code-block:: xml
    :emphasize-lines: 2,3,6,7

    <filter>
        <filter-name>exceptionLoggingFilter</filter-name> <!-- (1) -->
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class> <!-- (2) -->
    </filter>
    <filter-mapping>
        <filter-name>exceptionLoggingFilter</filter-name> <!-- (3) -->
        <url-pattern>/*</url-pattern> <!-- (4) -->
    </filter-mapping>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | フィルター名を指定する。\ ``applicationContext.xml``\ に定義した\ ``ExceptionLoggingFilter``\ のbean名と、一致させる。
    * - | (2)
      - | フィルタークラスを指定する。\ ``org.springframework.web.filter.DelegatingFilterProxy``\ 固定。
    * - | (3)
      - | マッピングするフィルターのフィルター名を指定する。(1)で指定した値。
    * - | (4)
      - | フィルターを適用するURLパターンを指定する。致命的なエラー、Spring MVC管理外をログ出力するため、\ ``/*``\ を推奨する。

- 出力ログ

 .. code-block:: guess

    date:2013-09-25 19:51:52	thread:tomcat-http--3	X-Track:f94de92148f1489b9ceeac3b2f17c969	level:ERROR	logger:o.t.gfw.common.exception.ExceptionLogger        	message:[e.xx.fw.9001] An exception occurred processing JSP page /WEB-INF/views/exampleJSPException.jsp at line 13


.. _exception-handling-how-to-use-application-configuration-container-label:

サーブレットコンテナの設定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
Spring MVCの、デフォルトの例外ハンドリング機能によって行われるエラー応答（HttpServletResponse#sendError）、致命的なエラー、Spring MVC管理外で発生する例外をハンドリングするために、サーブレットコンテナのError Page定義を追加する。

- **web.xml**

 Spring MVCの、デフォルトの例外ハンドリング機能によって行われるエラー応答（HttpServletResponse#sendError）を、ハンドリングするための定義を追加する。

 .. code-block:: xml

   <error-page>
       <!-- (1) -->
       <error-code>404</error-code>
       <!-- (2) -->
       <location>/WEB-INF/views/common/error/resourceNotFoundError.jsp</location>
   </error-page>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | ハンドリング対象とする **HTTPレスポンスコード** を指定する。
       | **【プロジェクト毎にカスタマイズする箇所】**
       | Spring MVCの、デフォルトの例外ハンドリング機能で応答されるHTTPレスポンスコードについては、\ :ref:`exception-handling-appendix-defaulthandlerexceptionresolver-label`\ を参照されたい。
   * - | (2)
     - | 遷移するファイル名を指定する。Webアプリケーションルートからのパスで、指定する。上記の設定では、"${WebAppRoot}/WEB-INF/views/common/error/resourceNotFoundError.jsp"が、遷移先のファイルとなる。
       | **【プロジェクト毎にカスタマイズする箇所】**


 致命的なエラー、Spring MVC管理外で発生する例外をハンドリングするための定義を追加する。

 .. code-block:: xml

    <error-page>
        <!-- (3) -->
        <location>/WEB-INF/views/common/error/unhandledSystemError.html</location>
    </error-page>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (3)
     - | 遷移するファイル名を指定する。Webアプリケーションルートからのパスで指定する。上記の設定では、"${WebAppRoot}/WEB-INF/views/common/error/unhandledSystemError.html"が、遷移先のファイルとなる。
       | **【プロジェクト毎にカスタマイズする箇所】**

.. note:: **locationに指定するパスについて**

  動的コンテンツのパスを指定した場合、致命的なエラーが発生していた場合に、別のエラーが発生する可能性が高くなるため、
  locationには、JSPなどの動的コンテンツでなく、 **HTMLなどの静的コンテンツへのパスを指定することを推奨する。**

.. note:: **開発中に原因が特定できないエラーが発生した場合**

  上記の設定が行われている状態で想定外のエラー応答（HttpServletResponse#sendError）が発生した場合、どのようなエラー応答が発生したのか特定できないケースがある。

  locationタグに指定したエラー画面が表示されるが、ログなどからエラーの原因を特定できない場合は、
  上記設定をコメントアウトして動かすことで、発生したエラー応答(HTTPレスポンスコード)を、画面で確認することできる。


  Spring MVC管理外で発生する例外を、個別にハンドリングする必要がある場合は、例外毎の定義を追加する。

    .. code-block:: xml

      <error-page>
          <!-- (4) -->
          <exception-type>java.io.IOException</exception-type>
          <!-- (5) -->
          <location>/WEB-INF/views/common/error/systemError.jsp</location>
      </error-page>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - 項番
        - 説明
      * - | (4)
        - | ハンドリング対象とする **例外クラス名(FQCN)** を指定する。
      * - | (5)
        - | 遷移するファイル名を指定する。Webアプリケーションルートからのパスで指定する。上記の設定では、"${WebAppRoot}/WEB-INF/views/common/error/systemError.jsp"が遷移先のファイルとなる。
          | **【プロジェクト毎にカスタマイズする箇所】**


.. _exception-handling-how-to-use-codingpoint-service-label:

コーディングポイント（Service編）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
例外ハンドリングを行う際の、Serviceでのコーディングポイントを、以下に示す。

#. :ref:`exception-handling-how-to-use-codingpoint-service-business-label`
#. :ref:`exception-handling-how-to-use-codingpoint-service-system-label`
#. :ref:`exception-handling-how-to-use-codingpoint-service-continue-label`


.. _exception-handling-how-to-use-codingpoint-service-business-label:

ビジネス例外を発生させる
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
ビジネス例外(BusinessException)の発生方法を、以下に示す。

.. note:: **ビジネス例外の発生方法に関する注意事項**

  - | 基本的には、ロジックでビジネスルールの違反を検知して、ビジネス例外を発生させる方法を推奨する。
  - | 既存資材や、基盤機能(FWや共通機能)のAPI仕様として、ビジネスルールの違反が、例外によって通知される場合のみ、例外を捕捉してビジネス例外を発生させてもよい。
    | 例外を、処理フローを制御するために使用すると、処理全体の見通しが悪くなり、保守性を低下させる可能性がある。

| ロジックでビジネスルールの違反を検知して、ビジネス例外を発生させる。

.. warning::

  - デフォルトでは、ビジネス例外は、Serviceで発生させることを想定している。\ :ref:`AOPの設定<exception-handling-how-to-use-service-pointcut-aop-label>`\ で、
    \ ``@Service``\ アノテーションを付与したクラスで発生したビジネス例外のログを出力としている。
    Controllerなどでビジネス例外は、ログを出力しない。プロジェクトでの考えがある場合は変更すること。

- xxxService.java

 .. code-block:: java

    ...
    @Service
    public class ExampleExceptionServiceImpl implements ExampleExceptionService {
        @Override
        public String throwBisinessException(String test) {
            ...
            // int stockQuantity = 5;
            // int orderQuantity = 6;

            if (stockQuantity < orderQuantity) {                  // (1)
                ResultMessages messages = ResultMessages.error(); // (2)
                messages.add("e.ad.od.5001", stockQuantity);      // (3)
                throw new BusinessException(messages);            // (4)
            }
            ...

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ビジネスルールの違反がないか、チェックを行う。
    * - | (2)
      - | 違反している場合、ResultMessagesを生成する。上記の実装例では、errorレベルのResultMessagesを生成している。
        | ResultMessagesの生成方法の詳細については、\ :doc:`../WebApplicationDetail/MessageManagement`\ を参照されたい。
    * - | (3)
      - | ResultMessagesに、ResultMessageを追加する。第1引数(必須)にメッセージIDを、第2引数(任意)にメッセージ埋め込み値を指定する。
        | メッセージ埋め込み値は、可変長パラメータなので、複数指定することができる。
    * - | (4)
      - | ResultMessagesを指定して、BusinessExceptionを発生させる。


 .. tip::

    上記の ``xxxService.java`` は説明用に(2)-(4)に分けて処理をしているが、1ステップで実装することができる。

     .. code-block:: java

        throw new BusinessException(ResultMessages.error().add(
             "e.ad.od.5001", stockQuantity));


- xxx.properties

  参考としてプロパティの設定を記述する。

  .. code-block:: properties

    e.ad.od.5001 = Order number is higher than the stock quantity={0}. Change the order number.

下記のようなアプリケーションログが出力される。

 .. code-block:: console

    date:2013-09-17 22:25:55	thread:tomcat-http--8	X-Track:6cfb0b378c124b918e40ac0c32a1fac7	level:WARN 	logger:o.t.gfw.common.exception.ExceptionLogger        	message:[e.xx.fw.8001] ResultMessages [type=error, list=[ResultMessage [code=e.ad.od.5001, args=[5], text=null]]]
    org.terasoluna.gfw.common.exception.BusinessException: ResultMessages [type=error, list=[ResultMessage [code=e.ad.od.5001, args=[5], text=null]]]

    // stackTarace omitted
    ...

    date:2013-09-17 22:25:55	thread:tomcat-http--8	X-Track:6cfb0b378c124b918e40ac0c32a1fac7	level:DEBUG	logger:o.t.gfw.web.exception.SystemExceptionResolver   	message:Resolving exception from handler [public java.lang.String org.terasoluna.exception.app.example.ExampleExceptionController.home(java.util.Locale,org.springframework.ui.Model)]: org.terasoluna.gfw.common.exception.BusinessException: ResultMessages [type=error, list=[ResultMessage [code=e.ad.od.5001, args=[5], text=null]]]
    date:2013-09-17 22:25:55	thread:tomcat-http--8	X-Track:6cfb0b378c124b918e40ac0c32a1fac7	level:DEBUG	logger:o.t.gfw.web.exception.SystemExceptionResolver   	message:Resolving to view 'common/error/businessError' for exception of type [org.terasoluna.gfw.common.exception.BusinessException], based on exception mapping [BusinessException]
    date:2013-09-17 22:25:55	thread:tomcat-http--8	X-Track:6cfb0b378c124b918e40ac0c32a1fac7	level:DEBUG	logger:o.t.gfw.web.exception.SystemExceptionResolver   	message:Applying HTTP status code 409
    date:2013-09-17 22:25:55	thread:tomcat-http--8	X-Track:6cfb0b378c124b918e40ac0c32a1fac7	level:DEBUG	logger:o.t.gfw.web.exception.SystemExceptionResolver   	message:Exposing Exception as model attribute 'exception'

表示される画面

 .. figure:: ./images/exception-handling-screen-businessexception.png
    :alt: screen business exception
    :width: 50%

 .. warning::
    ビジネス例外は、Controllerでハンドリングし、各業務画面でメッセージを表示させることを推奨する。
    上記例は、Controllerでハンドリングしなかった場合に、表示される画面となる。


例外を捕捉して、ビジネス例外を発生させる

 .. code-block:: java

    try {
        order(orderQuantity, itemId );
    } catch (StockNotEnoughException e) {                  // (1)
        throw new BusinessException(ResultMessages.error().add(
                "e.ad.od.5001", e.getStockQuantity()), e); // (2)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ビジネスルールに違反した際に、発生する例外を捕捉する。
    * - | (2)
      - | ResultMessagesと、\ **原因例外(e)**\ を指定して、BusinessExceptionを発生させる。


.. _exception-handling-how-to-use-codingpoint-service-system-label:

システム例外を発生させる
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
システム例外(SystemException)の発生方法を、以下に示す。


ロジックで、システム異常を検知し、システム例外を発生させる。

 .. code-block:: java

    if (itemEntity == null) {                                      // (1)
        throw new SystemException("e.ad.od.9012",
            "not found item entity. item code [" + itemId + "]."); // (2)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | システムが、正常な状態であることをチェックする。
        | ここでは、例として、リクエストされた商品コード（itemId）が、商品マスタ（Item Mastar）上に存在するかチェックし、
        | 存在しない場合、システムで用意するべきリソースがないと判断して、システムエラーにしている。
    * - | (2)
      - | システムが異常な状態の場合、第1引数に例外コード(メッセージID)を指定する。第2引数に例外メッセージを指定して、SystemExceptionを発生させる。
        | 上記の実装例では、メッセージ本文に、変数"itemId"の値を埋め込んでいる。

下記のような、アプリケーションログが出力される。

 .. code-block:: console

  date:2013-09-19 21:03:06	thread:tomcat-http--3	X-Track:c19eec546b054d54a13658f94292b24f	level:DEBUG	logger:o.t.gfw.web.exception.SystemExceptionResolver   	message:Resolving exception from handler [public java.lang.String org.terasoluna.exception.app.example.ExampleExceptionController.home(java.util.Locale,org.springframework.ui.Model)]: org.terasoluna.gfw.common.exception.SystemException: not found item entity. item code [10-123456].
  date:2013-09-19 21:03:06	thread:tomcat-http--3	X-Track:c19eec546b054d54a13658f94292b24f	level:DEBUG	logger:o.t.gfw.web.exception.SystemExceptionResolver   	message:Resolving to default view 'common/error/systemError' for exception of type [org.terasoluna.gfw.common.exception.SystemException]
  date:2013-09-19 21:03:06	thread:tomcat-http--3	X-Track:c19eec546b054d54a13658f94292b24f	level:DEBUG	logger:o.t.gfw.web.exception.SystemExceptionResolver   	message:Applying HTTP status code 500
  date:2013-09-19 21:03:06	thread:tomcat-http--3	X-Track:c19eec546b054d54a13658f94292b24f	level:DEBUG	logger:o.t.gfw.web.exception.SystemExceptionResolver   	message:Exposing Exception as model attribute 'exception'
  date:2013-09-19 21:03:06	thread:tomcat-http--3	X-Track:c19eec546b054d54a13658f94292b24f	level:ERROR	logger:o.t.gfw.common.exception.ExceptionLogger        	message:[e.ad.od.9012] not found item entity. item code [10-123456].
  org.terasoluna.gfw.common.exception.SystemException: not found item entity. item code [10-123456].
  	at org.terasoluna.exception.domain.service.ExampleExceptionServiceImpl.throwSystemException(ExampleExceptionServiceImpl.java:14) ~[ExampleExceptionServiceImpl.class:na]
  ...
  // stackTarace omitted

表示される画面

 .. figure:: ./images/exception-handling-screen-systemexception.png
   :alt: screen system exception
   :width: 60%

 .. note::

    システムエラー画面は、個別に用意せず、共通的に決めることを推奨する。

    本ガイドラインの画面では、システムエラーのためのメッセージID（業務毎）を表示し、文言は固定にしている。
    その理由は、オペレータに対して、エラーの細かい内容を知らせる必要がなく、システムに異常があることだけを伝えればよいためである。
    そこで、開発側では、解析を簡易にするために、キーとなるメッセージIDを画面に表示して、システム異常の問い合わせに対するレスポンスを向上しようとしている。
    表示される画面については、各プロジェクトでUI規約に従い、用意すること。


例外を捕捉して、システム例外を発生させる

 .. code-block:: java

    try {
        return new File(preUploadDir.getFile(), key);
    } catch (FileNotFoundException e) { // (1)
        throw new SystemException("e.ad.od.9007",
            "not found upload file. file is [" + preUploadDir.getDescription() + "]."
            e); // (2)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | システム異常に分類される検査例外を捕捉する。
    * - | (2)
      - | 例外コード(メッセージID)、メッセージ、\ **原因例外(e)**\ を指定して、SystemExceptionを発生させる。


.. _exception-handling-how-to-use-codingpoint-service-continue-label:

例外をキャッチして、処理を継続させる
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| 例外をキャッチして、処理を継続させる必要がある場合、発生した例外をログに出力してから、処理を継続するようにする。

| 下記は、外部システムから、顧客対応履歴の取得に失敗した場合に、顧客対応履歴以外の情報を取得する処理を、継続する場合の例である。
| この例では、顧客対応履歴の情報が取得できなくても、業務は継続できるため、処理を継続している。

 .. code-block:: java

    @Inject
    ExceptionLogger exceptionLogger; // (1)

    // ...

 .. code-block:: java

    InteractionHistory interactionHistory = null;
    try {
        interactionHistory = restTemplete.getForObject(uri, InteractionHistory.class, customerId);
    } catch (RestClientException e) { // (2)
        exceptionLogger.log(e); // (3)
    }

    // (4)
    Customer customer = customerRepository.findOne(customerId);

    // ...

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 共通ライブラリで提供している\ ``org.terasoluna.gfw.common.exception.ExceptionLogger``\ をログ出力するため、オブジェクトをDIする。
    * - | (2)
      - | ハンドリング対象の例外を、キャッチする。
    * - | (3)
      - | ハンドリングした例外を、ログに出力する。例では、logメソッドを呼び出しているが、出力レベルが決まっており、
        | 後に変更する可能性がない場合は、info、warn、errorメソッドを直接呼び出してもよい。
    * - | (4)
      - | (3)でログを出力したのみで、処理を継続する。


下記のような、アプリケーションログが出力される。

 .. code-block:: console

  date:2013-09-19 21:31:47	thread:tomcat-http--3	X-Track:df5271ece2304b12a2c59ff494806397	level:ERROR	logger:o.t.gfw.common.exception.ExceptionLogger        	message:[e.xx.fw.9001] Test example exception
  org.springframework.web.client.RestClientException: Test example exception
  ...
  // stackTarace omitted

 .. warning::

    \ ``exceptionLogger``\ で、log()を使用した場合には、errorレベルで出力されるため、デフォルトで監視ログにも出力される。

 .. code-block:: console

      date:2013-09-19 21:31:47	X-Track:df5271ece2304b12a2c59ff494806397	level:ERROR	message:[e.xx.fw.9001] Test example exception

次の例のように、処理を継続させて問題ない場合に、運用監視で監視ログを監視している場合は、出力レベルで監視されないレベルにするか、メッセージから監視されないよう定義が必要である。

 .. code-block:: java

      } catch (RestClientException e) {
          exceptionLogger.info(e);
      }

 | デフォルトの設定では、errorレベル以外の監視ログは出力されない。アプリケーションログには、以下のように出力される。

 .. code-block:: console

    date:2013-09-19 22:17:53	thread:tomcat-http--3	X-Track:999725b111b4445b8d10b4ea44639c61	level:INFO 	logger:o.t.gfw.common.exception.ExceptionLogger        	message:[e.xx.fw.9001] Test example exception
    org.springframework.web.client.RestClientException: Test example exception


.. _exception-handling-how-to-use-codingpoint-controller-label:


コーディングポイント（Controller編）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
例外ハンドリングを行う際の、Controllerでのコーディングポイントを、以下に示す。

#. :ref:`exception-handling-how-to-use-codingpoint-controller-request-label`
#. :ref:`exception-handling-how-to-use-codingpoint-controller-usecase-label`


.. _exception-handling-how-to-use-codingpoint-controller-request-label:

リクエスト単位で例外をハンドリングする方法
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| 例外をリクエスト単位でハンドリングし、引き継ぎ情報（メッセージ情報）を、Modelに設定する。
| その後、遷移する画面を表示するためのメソッドを呼び出すことで、遷移先で必要なモデルを生成し、View名を決定する。

 .. code-block:: java

    @RequestMapping(value = "change", method = RequestMethod.POST)
    public String change(@Validated UserForm userForm,
                         BindingResult result,
                         RedirectAttributes redirectAttributes,
                         Model model) {         // (1)

        // omitted

        User user = userHelper.convertToUser(userForm);
        try {
            userService.change(user);
        } catch (BusinessException e) {                                   // (2)
            model.addAttribute(e.getResultMessages());                    // (3)
            return viewChangeForm(user.getUserId(), model);               // (4)
        }

        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | エラー情報を、Viewと連携するためのオブジェクトとして、Modelを引数に定義する。
    * - | (2)
      - | ハンドリング対象となる例外を、アプリケーションコードで捕捉する。
    * - | (3)
      - | ResultMessagesオブジェクトを、Modelに追加する。
    * - | (4)
      - | エラー時の遷移先を表示するためのメソッドを呼び出し、View表示に必要なモデルと、View名を取得した後に、表示するView名を返却する。


.. _exception-handling-how-to-use-codingpoint-controller-usecase-label:

ユースケース単位で例外をハンドリングする方法
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| 例外を、ユースケース単位でハンドリングし、引き継ぎ情報（メッセージ情報など）が格納されたModelMap（ExtendedModelMap）を生成する。
| その後、遷移する画面を表示するためのメソッドを呼び出すことで、遷移先で必要なモデルを生成し、View名を決定する。

 .. code-block:: java

    @ExceptionHandler(BusinessException.class) // (1)
    @ResponseStatus(HttpStatus.CONFLICT) // (2)
    public ModelAndView handleBusinessException(BusinessException e) {
        ExtendedModelMap modelMap = new ExtendedModelMap();                 // (3)
        modelMap.addAttribute(e.getResultMessages());                       // (4)
        String viewName = top(modelMap);                                    // (5)
        return new ModelAndView(viewName, modelMap);                        // (6)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@ExceptionHandler``\ アノテーションのvalue属性に、ハンドリング対象とする例外クラスを指定する。ハンドリング対象とする例外は、複数指定することもできる。
    * - | (2)
      - | \ ``@ResponseStatus``\ アノテーションの、value属性に返却するHTTPステータスコードを指定する。例では、「409:Conflict」を指定している。
    * - | (3)
      - | エラー情報と、モデル情報を、Viewと連携するためのオブジェクトとして、ExtendedModelMapを生成する。
    * - | (4)
      - | ResultMessagesオブジェクトを、ExtendedModelMapに追加する。
    * - | (5)
      - | エラー時の遷移先を表示するためのメソッドを呼び出し、View表示に必要なモデルと、View名を取得する。
    * - | (6)
      - | (3)-(5)の処理で取得したView名と、Modelが格納されているModelAndViewを生成し、返却する。


.. _exception-handling-how-to-use-codingpoint-jsp-label:

コーディングポイント（JSP編）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
例外ハンドリングを行う際の、JSPでのコーディングポイントを、以下に示す。

#. :ref:`exception-handling-how-to-use-codingpoint-jsp-panel-label`
#. :ref:`exception-handling-how-to-use-codingpoint-jsp-exceptioncode-label`

.. tip::

    Internet Explorerがサポートブラウザとなっている場合は、
    エラー画面として応答するHTMLのサイズが513バイト以上になるように実装する必要がある。

    Internet Explorerでは、
    
    * 応答されたステータスコードがエラー系(4xxと5xx)
    * 応答されたHTMLが512バイト以下
    * ブラウザの設定が「HTTP簡易メッセージを表示する」が有効な状態
    
    という３つの条件を充たした際に、Internet Explorerが用意している簡易メッセージが表示される仕組みになっているためである。

.. _exception-handling-how-to-use-codingpoint-jsp-panel-label:

MessagesPanelTagを使用して、メッセージを画面表示する方法
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
任意の場所に、ResultMessagesを出力する際の実装例を、以下に示す。

 .. code-block:: xml

    <t:messagesPanel /> <!-- (1) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      -  メッセージを出力したい場所に、<t:messagesPanel>タグを指定する。 <t:messagesPanel>タグの使用方法の詳細については、\ :doc:`../WebApplicationDetail/MessageManagement`\ を参照されたい。


.. _exception-handling-how-to-use-codingpoint-jsp-exceptioncode-label:

システム例外の例外コードを、画面表示する方法
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
任意の場所に、例外コード(メッセージID)と、固定メッセージを表示する際の実装例を、以下に示す。

 .. code-block:: xml

    <p>
        <c:if test="${!empty exceptionCode}">  <!-- (1) -->
            [${f:h(exceptionCode)}]            <!-- (2) -->
        </c:if>
        <spring:message code="e.cm.fw.9999" /> <!-- (3) -->
    </p>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 例外コード(メッセージID)の存在チェックを行う。上記の実装例のように、記号などで例外コード(メッセージID)を囲む場合は、存在チェックを行うこと。
    * - | (2)
      - | 例外コード(メッセージID)を出力する。
    * - | (3)
      - | メッセージ定義より取得した、固定メッセージを出力する。

- 出力画面（exceptionCode有り）

 .. figure:: ./images/exception-handling-screen-systemexception-messagecode.png
   :alt: screen system exception messagecode
   :width: 40%

- 出力画面（exceptionCode無し）

 .. figure:: ./images/exception-handling-screen-systemexceptionbyjspexception.png
   :alt: screen system exception no messagecode
   :width: 40%

 .. note:: **システム例外時に出力するメッセージについて**

    - システム例外が発生した場合、エラー原因が特定できる、または推測できる詳細メッセージを出力せず、システム例外が発生したことだけを伝えるメッセージを表示することを推奨する。
    - エラー原因が特定できる、または推測できる詳細メッセージを表示した場合、システムの脆弱性を公開してしまう可能性がある。

 .. note:: **例外コード(メッセージID)について**

    - システム例外が発生した場合、詳細メッセージの代わりに、例外コード(メッセージID)を出力することを推奨する。
    - 例外コード(メッセージID)を出力することで、システム利用者からの問い合わせに、素早く対応することができる。
    - 例外コード(メッセージID)からエラー原因を特定できるのは、システム管理者だけなので、システムの脆弱性を公開する危険性は少なくなる。

|

How to use (Ajax)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ajaxの例外ハンドリングについては、\ :doc:`../WebApplicationDetail/Ajax`\ を参照されたい。

|

Appendix
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. :ref:`exception-handling-about-classes-of-library-label`
#. :ref:`exception-handling-about-systemexceptionresolver-label`
#. :ref:`exception-handling-appendix-defaulthandlerexceptionresolver-label`

|

.. _exception-handling-about-classes-of-library-label:

共通ライブラリから提供している例外ハンドリング用のクラスについて
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Spring MVCが提供しているクラスとは別に、共通ライブラリより例外ハンドリングを行うためのクラスを提供している。
| クラスの役割は、以下の通りである。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.65\linewidth}|
.. list-table:: **表- org.terasoluna.gfw.common.exception パッケージ配下のクラス**
   :header-rows: 1
   :widths: 10 20 65
   :class: longtable

   * - 項番
     - クラス
     - 役割
   * - | (1)
     - | ExceptionCode
       | Resolver
     - | 例外クラスに対応する例外コード（メッセージID）を解決するためのインタフェース。
       | 例外コードとは、どのような例外が発生したのかを識別するためのコードで、システムエラー画面や、ログに出力することを想定している。
       | \ ``ExceptionLogger``\ 、\ ``SystemExceptionResolver``\ などから参照される。
   * - | (2)
     - | SimpleMapping
       | ExceptionCode
       | Resolver
     - | \ ``ExceptionCodeResolver``\ の実装クラスで、例外クラスの名前と、例外コードのマッピングを保持することで、例外コードの解決を実現する。
       | 例外クラスの名前は、FQCNではなく、FQCNの一部や、親クラスの名前でもよい。

        .. warning::

            - FQCNの一部を指定した場合、場合によっては想定していなかったクラスとマッチしてしまうことがあるので注意が必要である。
            - 親クラスの名前を指定した場合、全ての子クラスがマッチするので注意が必要である。

   * - | (3)
     - | enums.
       | ExceptionLevel
     - | 例外クラスに対応する例外レベルを表現するenum。
       | INFO, WARN, ERRORが定義されている。
   * - | (4)
     - | ExceptionLevel
       | Resolver
     - | 例外クラスに対応する例外レベル（ログレベル）を解決するためのインタフェース。
       | 例外レベルとは、どのようなレベルの例外が発生したのかを識別するためのコードで、ログの出力レベルを切り替えるために使われる。
       | ``ExceptionLogger`` から参照される。
   * - | (5)
     - | DefaultException
       | LevelResolver
     - | ``ExceptionLevelResolver`` の実装クラスで、例外コードの先頭１文字で、例外レベルを解決している。
       | 先頭の１文字目（case insensitive）が、
       |   1. "i"の場合は ``ExceptionLevel.INFO``
       |   2. "w"の場合は ``ExceptionLevel.WARN``
       |   3. "e"の場合は ``ExceptionLevel.ERROR``
       |   4. 上記以外の場合は ``ExceptionLevel.ERROR``
       | レベルとして扱う。
       | 本クラスは、\ :doc:`メッセージ<../WebApplicationDetail/MessageManagement>`\ のガイドラインに記載されている、メッセージIDのルールに則った実装となっている。
   * - | (6)
     - | ExceptionLogger
     - | 例外をログ出力するためのクラス。
       | 監視ログ(メッセージのみ)と、アプリケーションログ(メッセージと、スタックトレースの両方)を出力することができる。
       | 本クラスは、フレームワークより提供しているFilterや、Interceptorクラスから使用されている。
       | アプリケーションコードで、例外をハンドリングして処理を継続する場合、本クラスを用いて、ログを出力すること。
   * - | (7)
     - | ResultMessages
       | LoggingInterceptor
     - | \ ``ResultMessages``\ を保持している例外(\ ``ResultMessagesNotificationException``\ のサブ例外 )が発生した事をログに出力するためのInterceptorクラス。
       | ログは全てWARNレベルで出力する。
       | 本Interceptorは、 ``@Service`` アノテーションが付与されているクラスのメソッドに対して適用することを想定している。
       | ログは、\ ``ExceptionLogger``\ を使用して出力している。
   * - | (8)
     - | BusinessException
     - | ビジネスルールの違反を検知したことを通知するための例外クラスで、ドメイン層のロジックで発生させる例外である。
       | \ ``java.lang.RumtimeException``\ を継承しているため、デフォルトの動作として、トランザクションは、ロールバックされる。
       | トランザクションをコミットしたい場合は、\ ``@Transactional``\ アノテーションの noRollbackFor 、または noRollbackForClassName に、本例外クラスを指定する必要がある。
   * - | (9)
     - | Resource
       | NotFoundException
     - | 指定されたリソース（データ）が、システム内に存在しないことを通知するための例外クラスで、主に、ドメイン層のロジックで発生させる例外である。
       | ``java.lang.RumtimeException`` を継承しているため、デフォルトの動作として、トランザクションは、ロールバックされる。
   * - | (10)
     - | ResultMessages
       | Notification
       | Exception
     - | 結果メッセージ（\ ``ResultMessages``\ ）を保持している例外であることを通知するための抽象例外クラスで、共通ライブラリでは、\ ``BusinessException``\ と、\ ``ResourceNotFoundException``\ が継承している。
       | \ ``java.lang.RumtimeException``\ を継承しているため、デフォルトの動作としてはトランザクションはロールバックされる。
       | 本例外クラスを継承すると、\ ``ResultMessagesLoggingInterceptor``\ によって、warnレベルのログが出力される。
   * - | (11)
     - | SystemException
     - | システム又はアプリケーションの異常を検知した事を通知するための例外クラスで、アプリケーション層又はドメイン層のロジックで発生させる例外である。
       | \ ``java.lang.RumtimeException``\ を継承しているため、デフォルトの動作として、トランザクションは、ロールバックされる。
   * - | (12)
     - | ExceptionCodeProvider
     - | 例外コードを保持する役割があることを示すインタフェースで、共通ライブラリでは、\ ``SystemException``\ が実装している。
       | 本インタフェースを実装した例外クラスを作成すると、共通ライブラリから提供している例外ハンドリング処理にて、例外で保持している例外コードで、そのまま使われる。

.. raw:: latex

   \newpage

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.65\linewidth}|
.. list-table:: **表- org.terasoluna.gfw.web.exception パッケージ配下のクラス**
   :header-rows: 1
   :widths: 10 20 65

   * - 項番
     - クラス
     - 役割
   * - | (13)
     - | SystemException
       | Resolver
     - | \ ``<mvc:annotation-driven>``\ を指定した際に、自動的に登録される\ ``HandlerExceptionResolver``\ によって、ハンドリングされない例外をハンドリングするためのクラス。
       | Spring MVCより提供されている\ ``SimpleMappingExceptionResolver``\ を継承し、例外コードのResultMessagesを、Viewから参照できるように機能追加を行っている。
   * - | (14)
     - | HandlerException
       | ResolverLogging
       | Interceptor
     - | \ ``HandlerExceptionResolver``\ でハンドリングされた例外を、ログに出力するためのInterceptorクラス。
       | 本Interceptorクラスでは、\ ``HandlerExceptionResolver``\ で解決されたHTTPレスポンスコードの分類に応じて、ログの出力レベルを切り替えている。
       |   1. "100-399"の場合は、 INFOレベルで出力する。
       |   2. "400-499"の場合は、 WARNレベルで出力する。
       |   3. "500-"の場合は ERRORレベルで出力する。
       |   4. "-99"の場合は ログ出力しない。
       | 本Interceptorを使用することで、Spring MVC管理下で発生する全ての例外を、ログに出力することができる。
       | ログは、\ ``ExceptionLogger``\ を使用して出力している。
   * - | (15)
     - | ExceptionLogging
       | Filter
     - | 致命的なエラー、Spring MVC管理外で発生する例外を、ログに出力するためのFilterクラス。
       | ログは、すべてERRORレベルで出力する。
       | 本Filterを使用した場合、致命的なエラー、およびSpring MVC管理外で発生するすべての例外を、ログに出力することができる。
       | ログは、\ ``ExceptionLogger``\ を使用して出力している。

.. raw:: latex

   \newpage

.. _exception-handling-about-systemexceptionresolver-label:

SystemExceptionResolverの設定項目について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
本編で説明していない設定項目について、説明する。
要件に応じて、設定を行うこと。

.. tabularcolumns:: |p{0.05\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|p{0.45\linewidth}|p{0.20\linewidth}|
.. list-table:: **本編で説明していない設定項目一覧**
   :header-rows: 1
   :widths: 5 15 15 45 20
   :class: longtable

   * - 項番
     - 項目名
     - プロパティ名
     - 説明
     - デフォルト値
   * - | (1)
     - | 結果メッセージの属性名
     - | resultMessagesAttribute
     - | ビジネス例外に設定されているメッセージ情報として、モデルに設定する際の属性名(String)を指定する。
       | View(JSP)から結果メッセージにアクセスする際の、属性名となる。
     - resultMessages
   * - | (2)
     - | 例外コード(メッセージID)の属性名
     - | exceptionCode
       | Attribute
     - | 例外コード(メッセージID)として、HttpServletRequestに設定する際の属性名(String)を指定する。
       | View(JSP)から例外コード(メッセージID)にアクセスする際の属性名となる。
     - exceptionCode
   * - | (3)
     - | 例外コード(メッセージID)のヘッダー名
     - | exceptionCode
       | Header
     - | 例外コード(メッセージID)として、HttpServletResponseのレスポンスヘッダーに設定する際のヘッダー名(String)を指定する。
     - X-Exception-Code
   * - | (4)
     - | 例外オブジェクトの属性名
     - | exceptionAttribute
     - | ハンドリングした例外オブジェクトとして、モデルに設定する際の属性名(String)を指定する。
       | View(JSP)から例外オブジェクトにアクセスする際の属性名となる。
     - exception
   * - | (5)
     - | 本ExceptionResolverとして、使用するハンドラー(Controller)のオブジェクト一覧
     - | mappedHandlers
     - | 本ExceptionResolverを使用するハンドラーの、オブジェクト一覧(Set)を指定する。
       | 指定したハンドラーオブジェクトで発生した例外のみ、ハンドリングが行われる。
       | **この設定項目は指定してはいけない。**
     - | 指定なし
       |
       | **指定した場合の動作は、保証しない。**
   * - | (6)
     - | 本ExceptionResolverを使用するハンドラー(Controller)のクラス一覧
     - | mappedHandlerClasses
     - | 本ExceptionResolverを使用するハンドラーのクラス一覧(Class[])を指定する。
       | 指定したハンドラークラスで発生した例外のみハンドリングが行われる。
       | **この設定項目は指定してはいけない。**
     - | 指定なし
       |
       | **指定した場合の動作は、保証しない。**
   * - | (7)
     - | HTTPレスポンスのキャッシュ制御有無
     - | preventResponseCaching
     - | HTTPレスポンス時のキャッシュ制御の有無(true:有 false:無)を指定する。
       | true:有を指定すると、キャッシュを無効にするためのHTTPレスポンスヘッダーが追加される。
     - | false:無

.. raw:: latex

   \newpage

| (1)-(3)は、\ ``org.terasoluna.gfw.web.exception.SystemExceptionResolver``\ の設定項目。
| (4)は、\ ``org.springframework.web.servlet.handler.SimpleMappingExceptionResolver``\ の設定項目。
| (5)-(7)は、\ ``org.springframework.web.servlet.handler.AbstractHandlerExceptionResolver``\ の設定項目。


結果メッセージの属性名
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| SystemExceptionResolverでハンドリングして設定したメッセージと、アプリケーションコードでハンドリングして設定したメッセージを、View(JSP)で別のmessagesPanelとして出力したい場合は、SystemExceptionResolver専用の属性名を指定する。
| 下記に示す例は、デフォルト値から「resultMessagesForExceptionResolver」に変更する場合の、設定&実装例である。

- **spring-mvc.xml**

  .. code-block:: xml

    <bean class="org.terasoluna.gfw.web.exception.SystemExceptionResolver">

        <!-- omitted -->

        <property name="resultMessagesAttribute" value="resultMessagesForExceptionResolver" /> <!-- (1) -->

        <!-- omitted -->
    </bean>

- **jsp**

  .. code-block:: xml

    <t:messagesPanel messagesAttributeName="resultMessagesForExceptionResolver"/> <!-- (2) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 結果メッセージの属性名(resultMessagesAttribute)に、"resultMessagesForExceptionResolver"を指定する。
    * - | (2)
      - | メッセージ属性名(messagesAttributeName)に、SystemExceptionResolverで設定した属性名を指定する。


例外コード(メッセージID)の属性名
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| デフォルトの属性名をアプリケーションコードで使用している場合は、重複を避けるために、別の値を設定すること。重複がない場合は、デフォルト値を変更する必要はない。
| 下記は、デフォルト値から、「exceptionCodeForExceptionResolver」に変更する場合の、設定&実装例である。

- **spring-mvc.xml**

  .. code-block:: xml

    <bean class="org.terasoluna.gfw.web.exception.SystemExceptionResolver">

        <!-- omitted -->

        <property name="exceptionCodeAttribute" value="exceptionCodeForExceptionResolver" /> <!-- (1) -->

        <!-- omitted -->
    </bean>

- **jsp**

  .. code-block:: xml

    <p>
        <c:if test="${!empty exceptionCodeForExceptionResolver}">  <!-- (2) -->
            [${f:h(exceptionCodeForExceptionResolver)}]            <!-- (3) -->
        </c:if>
        <spring:message code="e.cm.fw.9999" />
    </p>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 例外コード(メッセージID)の属性名(exceptionCodeAttribute)に、"exceptionCodeForExceptionResolver"を指定する。
    * - | (2)
      - | SystemExceptionResolverに設定した値(exceptionCodeForExceptionResolver)を、テスト対象(空チェック対応)の変数名として指定する。
    * - | (3)
      - | SystemExceptionResolverに設定した値(exceptionCodeForExceptionResolver)を、出力対象の変数名として指定する。


例外コード(メッセージID)のヘッダー名
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| デフォルトのヘッダー名が使用されている場合、重複を避けるために、別の値を設定すること。重複がない場合は、デフォルト値を変更する必要はない。
| 下記は、デフォルト値から「X-Exception-Code-ForExceptionResolver」に変更する場合の、設定&実装例である。

- **spring-mvc.xml**

  .. code-block:: xml

    <bean class="org.terasoluna.gfw.web.exception.SystemExceptionResolver">

        <!-- omitted -->

        <property name="exceptionCodeHeader" value="X-Exception-Code-ForExceptionResolver" /> <!-- (1) -->

        <!-- omitted -->
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 例外コード(メッセージID)のヘッダー名(exceptionCodeHeader)に、"X-Exception-Code-ForExceptionResolver"を指定する。


例外オブジェクトの属性名
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| デフォルトの属性名をアプリケーションコードで使用している場合は、重複を避けるために、別の値を設定すること。重複がない場合は、デフォルト値を変更する必要はない。
| 下記は、デフォルト値から「exceptionForExceptionResolver」に変更する場合の、設定&実装例である。

- **spring-mvc.xml**

  .. code-block:: xml

    <bean class="org.terasoluna.gfw.web.exception.SystemExceptionResolver">

        <!-- omitted -->

        <property name="exceptionAttribute" value="exceptionForExceptionResolver" /> <!-- (1) -->

        <!-- omitted -->
    </bean>

- **jsp**

  .. code-block:: xml

    <p>[Exception Message]</p>
    <p>${f:h(exceptionForExceptionResolver.message)}</p> <!-- (2) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 例外オブジェクトの属性名(exceptionAttribute)に、"exceptionForExceptionResolver"を指定する。
    * - | (2)
      - | SystemExceptionResolverに設定した値(exceptionForExceptionResolver)を、例外オブジェクトからメッセージを取得するための変数名として、指定する。

.. _exception-handling-http-response-cache:

HTTPレスポンスのキャッシュ制御有無
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| HTTPレスポンスに、キャッシュ制御用のヘッダーを追加したい場合は、true:有を指定する。

- **spring-mvc.xml**

  .. code-block:: xml

    <bean class="org.terasoluna.gfw.web.exception.SystemExceptionResolver">

        <!-- omitted -->

        <property name="preventResponseCaching" value="true" /> <!-- (1) -->

        <!-- omitted -->
    </bean>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | HTTPレスポンスのキャッシュ制御有無(preventResponseCaching)に、true:有を指定する。

 .. note:: **有を指定した場合のHTTPレスポンスヘッダー**

    HTTPレスポンスのキャッシュ制御有無を有にすると、以下のHTTPレスポンスヘッダーが出力される。

     | Cache-Control:no-store

    \ ``SystemExceptionResolver``\によるキャッシュ制御用のヘッダー追加はブラウザキャッシュによる意図しないエラー画面の表示を抑止するためのオプションであるが、Spring Securityの機能を使用してセキュリティの観点からキャッシュ制御用のヘッダーを追加することも可能である。
    Spring Securityの機能については、:ref:`SpringSecurityLinkageWithBrowser`\を参照されたい。

.. _exception-handling-about-handlerexceptionresolverlogginginterceptor:

HandlerExceptionResolverLoggingInterceptorの設定項目について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
本編で説明していない設定項目について、説明する。
要件に応じて、設定を行うこと。

.. tabularcolumns:: |p{0.05\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|p{0.45\linewidth}|p{0.20\linewidth}|
.. list-table:: **本編で説明していない設定項目一覧**
   :header-rows: 1
   :widths: 5 15 15 45 20

   * - 項番
     - 項目名
     - プロパティ名
     - 説明
     - デフォルト値
   * - | (1)
     - | ログ出力対象から除外する例外クラスの一覧
     - | ignoreExceptions
     - | ``HandlerExceptionResolver`` によってハンドリングされた例外のうち、ログ出力しない例外クラスをリスト形式で指定する。
       | 指定した例外クラス及びサブクラスの例外が発生した場合、 本クラスでログの出力は行われない。
       | 本項目に指定する例外クラスは、別の場所(別の仕組み)でログ出力される例外のみ指定すること。
     - | ``ResultMessagesNotificationException.class``
       |
       | ``ResultMessagesNotificationException.class`` 及びサブクラスの例外は、 ``ResultMessagesLoggingInterceptor`` でログ出力されるため、デフォルト設定として除外している。

ログ出力対象から除外する例外クラスの一覧
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
プロジェクトで用意した例外クラスをログ出力対象から除外したい場合は、以下のような設定となる。

- **spring-mvc.xml**

 .. code-block:: xml

    <bean id="handlerExceptionResolverLoggingInterceptor"
        class="org.terasoluna.gfw.web.exception.HandlerExceptionResolverLoggingInterceptor">
        <property name="exceptionLogger" ref="exceptionLogger" />
        <property name="ignoreExceptions">
            <set>
                <!-- (1) -->
                <value>org.terasoluna.gfw.common.exception.ResultMessagesNotificationException</value>
                <!-- (2) -->
                <value>com.example.common.XxxException</value>
            </set>
        </property>
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 共通ライブラリのデフォルト設定で指定されている ``ResultMessagesNotificationException`` を除外対象に指定する。
    * - | (2)
      - | プロジェクトで用意した例外クラスを除外対象に指定する。

|

全ての例外クラスをログ出力対象とする場合は、以下のような設定となる。

- **spring-mvc.xml**

 .. code-block:: xml

    <bean id="handlerExceptionResolverLoggingInterceptor"
        class="org.terasoluna.gfw.web.exception.HandlerExceptionResolverLoggingInterceptor">
        <property name="exceptionLogger" ref="exceptionLogger" />
        <!-- (3) -->
        <property name="ignoreExceptions"><null /></property>
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (3)
      - | ignoreExceptionsプロパティに ``null`` を指定する。
        | ``null`` を指定すると、全ての例外クラスがログ出力対象となる。

.. _exception-handling-appendix-defaulthandlerexceptionresolver-label:

DefaultHandlerExceptionResolverで設定されるHTTPレスポンスコードについて
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
DefaultHandlerExceptionResolverでハンドリングされるフレームワーク例外と、HTTPステータスコードのマッピングを、以下に記載する。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.60\linewidth}|p{0.20\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 60 20
   :class: longtable

   * - 項番
     - ハンドリングされるフレームワーク例外
     - HTTPステータスコード
   * - | (1)
     - | org.springframework.web.servlet.mvc.multiaction.NoSuchRequestHandlingMethodException
     - | 404
   * - | (2)
     - | org.springframework.web.HttpRequestMethodNotSupportedException
     - | 405
   * - | (3)
     - | org.springframework.web.HttpMediaTypeNotSupportedException
     - | 415
   * - | (4)
     - | org.springframework.web.HttpMediaTypeNotAcceptableException
     - | 406
   * - | (5)
     - | org.springframework.web.bind.MissingServletRequestParameterException
     - | 400
   * - | (6)
     - | org.springframework.web.bind.ServletRequestBindingException
     - | 400
   * - | (7)
     - | org.springframework.beans.ConversionNotSupportedException
     - | 500
   * - | (8)
     - | org.springframework.beans.TypeMismatchException
     - | 400
   * - | (9)
     - | org.springframework.http.converter.HttpMessageNotReadableException
     - | 400
   * - | (10)
     - | org.springframework.http.converter.HttpMessageNotWritableException
     - | 500
   * - | (11).
     - | org.springframework.web.bind.MethodArgumentNotValidException
     - | 400
   * - | (12)
     - | org.springframework.web.multipart.support.MissingServletRequestPartException
     - | 400
   * - | (13)
     - | org.springframework.validation.BindException
     - | 400
   * - | (14)
     - | org.springframework.web.servlet.NoHandlerFoundException
     - | 404

.. raw:: latex

   \newpage

