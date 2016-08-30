セッション管理
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

Overview
--------------------------------------------------------------------------------

本節では、Webアプリケーションのセッション管理について説明する。

| Webアプリケーションは、HTTPを利用して、クライアントとサーバ間でのデータのやり取りを行う。
| HTTP自体には、物理的にセッションを維持する仕組みはないが、セッションを識別するための値(セッションID)を、クライアントとサーバとの間で連携することで、論理的にセッションを維持する仕組みが提供されている。
| クライアントと、サーバとの間で、セッションIDの連携する方法としては、Cookie、またはリクエストパラメータが使用される。
| 以下に、論理的なセッションの確立イメージを示す。

 .. figure:: ./images/session-management_overview_cooperation.png
   :alt: Establishment of logical session
   :width: 100%
   :align: center

   **Picture - Establishment of logical session**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | Webブラウザ(Client)は、セッションが確立していない状態で、Webアプリケーション(Server)にアクセスする。
    * - | (2)
      - | Webアプリケーションは、Webブラウザとのセッションを管理するために、\ ``HttpSession``\ オブジェクトを生成する。\ ``HttpSession``\ オブジェクトを生成したタイミングで、セッションIDが払い出される。
    * - | (3)
      - | Webアプリケーションは、Webブラウザから送信されたデータを、\ ``HttpSession``\ オブジェクトに格納する。
    * - | (4)
      - | Webアプリケーションは、Webブラウザにレスポンスを返却する。レスポンスの「Set-Cookie」ヘッダに、「JSESSIONID = 払い出されたセッションID」を設定することで、セッションIDをWebブラウザに連携する。
        | 連携したセッションIDはCookieに格納される。
    * - | (5)
      - | Webブラウザは、リクエストの「Cookie」ヘッダに、「JSESSIONID = 払い出されたセッションID」を設定することで、セッションIDをWebアプリケーションと連携する。
        | Webアプリケーションがデプロイされているアプリケーションサーバは、Webブラウザから連携されたセッションIDに対応する\ ``HttpSession``\ オブジェクトを取得し、リクエストに関連づける。
    * - | (6)
      - | Webアプリケーションは、リクエストに関連付けられた\ ``HttpSession``\ オブジェクトから、(1)のリクエストで格納したデータを取得する。
        | **リクエストをまたいで、同じデータにアクセスすることができる。**
    * - | (7)
      - | Webアプリケーションは、Webブラウザにレスポンスを返却する。

 .. note:: **セッションIDを連携するためのパラメータ名について**

    JavaEEのSerlvetの仕様では、セッションIDを連携するためのパラメータ名のデフォルトは、「JSESSIONID」となっている。

セッションのライフサイクル
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| セッションのライフサイクルの制御(生成、破棄、タイムアウト検知)は、Controllerの処理として実装するのではなく、
| フレームワークや共通ライブラリから提供されている処理を使用して行う。

 .. note::
 
    以降の説明で登場する ``"セッション"`` は、Servlet APIより提供されている ``javax.servlet.http.HttpSession`` オブジェクトの事である。
    ``HttpSession`` オブジェクトは、上記で説明した論理的なセッションを表現するJavaオブジェクトである。

セッションの生成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
本ガイドラインで推奨している方法でWebアプリケーションを作成した場合、以下のいずれかの処理でセッションが生成される。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - 1.
      - | Spring Securityから提供されている認証・認可を行う処理。
        | Spring Securityの設定により、セッションの生成有無や、生成タイミングを指定することができる。
        | Spring Securityで行われるセッション管理についての詳細は、\ :ref:`authentication(spring_security)_how_to_use_sessionmanagement`\ を参照されたい。
    * - 2.
      - | Spring Securityから提供されているCSRFトークンチェックを行う処理。
        | 既にセッションが確立されている場合は、新たなセッションは生成されない。
        | CSRFトークンチェックの詳細については、\ :doc:`../../Security/CSRF`\ を参照されたい。
    * - 3.
      - | 共通ライブラリから提供されているトランザクショントークンチェックを行う処理。
        | 既にセッションが確立されている場合は、新たなセッションは生成されない。
        | トランザクショントークンチェックの詳細については、\ :doc:`DoubleSubmitProtection`\ を参照されたい。
    * - 4.
      - | \ ``RedirectAttributes``\ インタフェースのaddFlashAttributeメソッドを使用して、リダイレクト先のリクエストにモデル（フォームオブジェクトやドメインオブジェクトなど）を引き渡す処理。
        | 既にセッションが確立されている場合は、新たなセッションは生成されない。
        | \ ``RedirectAttributes``\ およびFlash scopeについての詳細は、\ :ref:`controller_method_argument-redirectattributes-label`\ を参照されたい。
    * - 5.
      - | \ ``@SessionAttributes``\ アノテーションを使用して、モデル（フォームオブジェクトや、ドメインオブジェクトなど）をセッションに格納する処理。
        | 指定したモデル（フォームオブジェクトや、ドメインオブジェクトなど）がセッションに格納される。既にセッションが確立されている場合は、新たなセッションは生成されない。
        | \ ``@SessionAttributes``\ アノテーションの使用方法については、\ :ref:`session-management_how_to_use_sessionattributes`\ を参照されたい。
    * - 6.
      - | Spring Frameworkの、sessionスコープのBeanを使用する処理。
        | 既にセッションが確立されている場合は、新たなセッションは生成されない。
        | sessionスコープのBeanの使用方法については、\ :ref:`session-management_how_to_use_sessionscope`\ を参照されたい。

 .. note::

    上記の項番4, 5, 6については、セッションの使用有無はControllerの実装によって指定するが、セッションの生成タイミングは、フレームワークによって制御される。
    つまり、Controllerの処理として ``HttpSession`` のAPIを直接使用する必要はない。

セッションへの属性格納
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
本ガイドラインで推奨している方法でWebアプリケーションを作成した場合、以下のいずれかの処理でセッションに属性(オブジェクト)が格納される。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - 1.
      - | Spring Securityから提供されている認証を行う処理。
        | 認証されたユーザ情報がセッションに格納される。
        | Spring Securityで行われる認証処理の詳細は、\ :doc:`../../Security/Authentication`\ を参照されたい。
    * - 2.
      - | Spring Securityから提供されているCSRFトークンチェックを行う処理。
        | 払い出されたトークン値がセッションに格納される。
        | CSRFトークンチェックの詳細については、\ :doc:`../../Security/CSRF`\ を参照されたい。
    * - 3.
      - | 共通ライブラリから提供されているトランザクショントークンチェックを行う処理。
        | 払い出されたトークン値がセッションに格納される。
        | トランザクショントークンチェックの詳細については、\ :doc:`DoubleSubmitProtection`\ を参照されたい。
    * - 4.
      - | \ ``RedirectAttributes``\ インタフェースのaddFlashAttributeメソッドを使用して、リダイレクト先のリクエストにモデル（フォームオブジェクトやドメインオブジェクトなど）を引き渡す処理。
        | \ ``RedirectAttributes``\ インタフェースのaddFlashAttributeメソッドの引数に指定したオブジェクトが、セッション上に存在するFlash scopeという領域に格納される。
        | \ ``RedirectAttributes``\ およびFlash scopeについての詳細は、\ :ref:`controller_method_argument-redirectattributes-label`\ を参照されたい。
    * - 5.
      - | \ ``@SessionAttributes``\ アノテーションを使用して、モデル（フォームオブジェクトや、ドメインオブジェクトなど）をセッションに格納する処理。
        | 指定したモデル（フォームオブジェクトや、ドメインオブジェクトなど）がセッションに格納される。
        | \ ``@SessionAttributes``\ アノテーションの使用方法については、\ :ref:`session-management_how_to_use_sessionattributes`\ を参照されたい。
    * - 6.
      - | Spring Frameworkの、sessionスコープのBeanを使用する処理。
        | sessionスコープのBeanがセッションに格納される。
        | sessionスコープのBeanの使用方法については、\ :ref:`session-management_how_to_use_sessionscope`\ を参照されたい。

 .. note::

    オブジェクトをセッションに格納するタイミングはフレームワークによって制御されるため、Controllerの処理として ``HttpSession`` オブジェクトのsetAttributeメソッドを呼び出すことはない。

セッションからの属性削除
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
本ガイドラインで推奨している方法で、Webアプリケーションを作成した場合、以下のいずれかの処理でセッションから属性(オブジェクト)が削除される。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - 1.
      - | Spring Securityから提供されているログアウトを行う処理。
        | 認証されたユーザ情報がセッションから削除される。
        | Spring Securityで行われるログアウト処理についての詳細は、\ :doc:`../../Security/Authentication`\ を参照されたい。
    * - 2.
      - | 共通ライブラリから提供されているトランザクショントークンチェックを行う処理。
        | 払い出されたトークン値が、ネームスペースに割り振られている上限値を超えた場合、使用されていないトークン値がセッションから削除される。
        | トランザクショントークンチェックの詳細については、\ :doc:`DoubleSubmitProtection`\ を参照されたい。
    * - 3.
      - | Flash scopeにオブジェクトを格納した後のリダイレクト処理。
        | \ ``RedirectAttributes``\ インタフェースのaddFlashAttributeメソッドの引数に指定したオブジェクトが、セッション上に存在するFlash scopeという領域から削除される。
    * - 4.
      - | Controllerの処理として、 \ ``SessionStatus``\ オブジェクトのsetCompleteメソッドを呼び出した後のフレームワークの処理。
        | \ ``@SessionAttributes``\ アノテーションで指定したオブジェクトがセッションから削除される。

 .. note::

    セッションからオブジェクトを削除するタイミングはフレームワークによって制御されるため、Controllerの処理として ``HttpSession`` オブジェクトのremoveAttributeメソッドを呼び出すことはない。

セッションの破棄
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
本ガイドラインで推奨している方法で、Webアプリケーションを作成した場合、以下のいずれかの処理でセッションが破棄される。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - 1.
      - | Spring Securityから提供されているログアウト処理。
        | Spring Securityで行われるログアウト処理についての詳細は、\ :doc:`../../Security/Authentication`\ を参照されたい。
    * - 2.
      - | アプリケーションサーバのセッションタイムアウト検知処理。

明示的に破棄する際のイメージを、以下に示す。

 .. figure:: ./images/session-management_overview_invalidate1.png
   :alt: Invalidate session by processing of Web Application
   :width: 100%
   :align: center

   **Picture - Invalidate session by processing of Web Application**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | Webブラウザからセッションを破棄する処理に、アクセスする。
        | Spring Securityを使用する場合は、Spring Securityから提供されているログアウト処理が、セッションを破棄する処理を行っている。
        | Spring Securityで行われるログアウト処理についての詳細は、\ :doc:`../../Security/Authentication`\ を参照されたい。
    * - | (2)
      - | Webアプリケーションは、Webブラウザから連携されたセッションIDに対応する\ ``HttpSession``\ オブジェクトを破棄する。
        | この時点でサーバ側には、 ``"SESSION01"`` というIDの\ ``HttpSession``\ オブジェクトが消滅する。
    * - | (3)
      - | Webブラウザから破棄されたセッションのセッションIDを使ってアクセスされた場合、セッションIDに対応する\ ``HttpSession``\ オブジェクトが存在しないため、別のセッションを生成する。
        | 上記例では、セッションIDが、 ``"SESSION02"`` のセッションを生成している。

タイムアウトによって、自動的に破棄される際のイメージを、以下に示す。

 .. figure:: ./images/session-management_overview_invalidate2.png
   :alt: Invalidate session by timeout Application Server
   :width: 100%
   :align: center

   **Picture - Invalidate session by Application Server**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 確立されたセッションに対して一定時間アクセスがない場合、アプリケーションサーバは、セッションタイムアウトを検知する。
    * - | (2)
      - | アプリケーションサーバは、セッションタイムアウトが検知されたセッションを破棄する。
    * - | (3)
      - | セッションタイムアウト発生後に、Webブラウザからアクセスされた場合、Webブラウザから送られてきたセッションIDに対応する\ ``HttpSession``\ オブジェクトが存在しないため、セッションタイムアウトエラーをWebブラウザに返却する。

 .. note:: **セッションタイムアウトの設計**

       セッションにデータを格納する場合は、必ずセッションタイムアウトの設計を行うこと。特に、格納するデータのサイズが大きくなる場合は、タイムアウトは、可能な限り短く設定することを推奨する。

 .. note:: **デフォルトのセッションタイムアウト時間について**

       デフォルトのセッションタイムアウト時間は、アプリケーションサーバによって異なる。

       * Tomcat : 1800 秒 (30分)
       * WebLogic : 3600 秒 (60分)
       * WebSphere : 1800 秒 (30分)
       * JBoss : 1800 秒 (30分)

セッションタイムアウト後のリクエスト検知
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
本ガイドラインで推奨している方法でWebアプリケーションを作成した場合、以下のいずれかの処理で、セッションタイムアウト後のリクエストを検知する。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - 1.
      - | Spring Securityから提供されているセッションのタイムアウトチェック処理。
        | Spring Securityのデフォルトの設定では、セッションのタイムアウトチェックは行われない。
        | そのため、セッションにデータを格納する場合は、Spring Securityのセッションのタイムアウトチェック処理を有効化するための設定が、必要となる。
        | Spring Securityで行われるセッションのタイムアウトチェック処理の詳細は、\ :ref:`authentication(spring_security)_how_to_use_sessionmanagement`\ を参照されたい。
    * - 2.
      - | Spring Securityを使用しない場合は、Servlet Filter、または、Spring MVCの\ ``HandlerInterceptor``\ にて、セッションのタイムアウトチェックを行う処理を実装する必要がある。

Spring Securityから提供されているセッションチェック処理を使用して、セッションタイムアウトを検知する際のイメージについて、以下に示す。

 .. figure:: ./images/session-management_overview_sessiontimeout.png
   :alt: Detected a request of after session timeout by Spring Security
   :width: 100%
   :align: center

   **Picture - Detected a request of after session timeout by Spring Security**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 確立されたセッションに対して、一定時間アクセスがない場合、アプリケーションサーバは、セッションタイムアウトを検知し、セッションを破棄する。
    * - | (2)
      - | セッションタイムアウト発生後に、Webブラウザからアクセスが発生する。
    * - | (3)
      - | Spring Securityから提供されているセッションの存在チェック処理は、クライアントから連携されたセッションIDに対応する\ ``HttpSession``\ オブジェクトが存在しないため、セッションタイムアウトエラーとする。
        | Spring Securityのデフォルト実装では、エラー画面を表示するための、URLへのリダイレクト要求が応答される。

 .. note:: **セッションのタイムアウトチェックの必要性**

    「セッションにデータが格納されていること」が事前条件となる処理については、必ずセッションのタイムアウトチェックを行うこと。
    セッションのタイムアウトチェックを行わないと、処理で必要なデータが取得できないため、予期しないシステムエラーの発生や、想定外の動作を引き起こす可能性がある。

 
セッションの利用について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 複数の画面(複数のリクエスト)をまたがって、データの持ち回りが必要になる場合は、持ち回り対象のデータをセッションに格納することで、簡単にデータを持回ることができる。
| ただし、セッションにデータを格納すると、データの持ち回りが簡単になるというメリットがある反面、アプリケーション上の制約などが発生するというデメリットもあるため、アプリケーションおよびシステム要件を考慮して、使用有無を決めること。

 .. note::
 
    本ガイドラインでは、安易にセッションにデータを格納するのではなく、まずはセッションを使わない方針で検討し、本当に必要なデータのみセッションに格納することを推奨する。

 .. note::
 
    以下の条件にあてはまるデータについては、セッションにデータを格納した方がよい場合がある。
    
    * | ユースケース間で連携はしないが、別のユースケースに移って戻った際に、状態を保持しておく必要があるデータ。
      | 例えば、一覧画面の検索条件が、このパターンに該当する。
      | 一覧画面の検索条件は、別のユースケース（例えば、「検索したデータを変更する」ユースケース）から戻った際に、別のユースケースに移る前の状態を保持することが機能要件となる事が多い。
      | 検索条件をhiddenで持ち回る方法もあるが、ユースケース間に余計な依存関係が生まれ、アプリケーションの実装も複雑になることが予想される。

    * | ユースケース間で連携が必要なデータ。
      | たとえば、ショッピングサイトのカートに格納するデータが、このパターンに該当する。
      | ショッピングサイトのカートに格納するデータは、「商品をカートに追加する」ユースケース、「カートを表示する」ユースケース、「カートの状態を変更する」ユースケース、「カートにいれた商品を購入する」ユースケースでデータの連携が必要となるためである。
      | ただし、スケラビリティを考慮する必要がある場合は、セッションではなくデータベースに格納した方がよいケースがある。
 

セッション利用時のメリットとデメリット
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
セッション利用時のメリットとデメリットは、以下の通りである。

* **メリット**

  * | 複数の画面(複数のリクエスト)をまたいで、データを持ち回ることができるため、ウィザード画面のような複数の画面で、1つ処理を構成する場合に、簡単にデータが持ち回れる。
  * | 取得したデータをセッションに格納しておくことで、データの取得処理の実行回数を、減らすことができる。

* **デメリット**

  * | 同一処理の画面を、複数のブラウザやタブで立ち上げた場合、互いの操作がセッション上に格納しているデータに干渉しあうため、データの整合性を保つことができなくなる。
    | データの整合性を保つためには、同一処理の画面を複数立ち上げることができないように、制御する必要がある。
    | データの整合性を保つための制御は、共通ライブラリから提供しているトランザクショントークンチェックを使用することで実現する事ができるが、ユーザビリティの低いアプリケーションとなってしまう。
  * | セッションは、通常アプリケーションサーバ上のメモリとして管理されるため、セッションに格納するデータの量に比例して、メモリの使用量も増大する。
    | 処理で使用されなくなったデータを残したままにすると、ガベージコレクションの対象外となり、メモリ枯渇の原因となるため、不要になった段階でセッションから削除する必要がある。
    | セッションから不要となったデータを削除するタイミングについて、別途設計を行う必要がある。
  * | 処理で扱うデータをセッションに格納すると、APサーバのスケーラビリティを低下させる要因となりうる。

    .. note::

        APサーバをスケールアウトする場合、以下のいずれかの仕組みが必要となる。
    
        1. | セッションをレプリケーションし、すべてのAPサーバでセッション情報を共有する。
           | セッションをレプリケーションする場合は、セッションに格納されるデータの量とレプリケーション対象となるAPサーバの数に比例してレプリケーション処理にかかる負荷が高くなる。
           | そのため、スケールアウトすることで、レスポンスタイムなどが劣化する可能性がある点に注意が必要となる。

        2. | ロードバランサによって、同一セッション内で発生するリクエストを全て同じAPサーバに振り分ける。
           | 同じAPサーバに振り分ける場合は、APサーバがダウンした場合に別のAPサーバで処理を継続することができない。
           | そのため、高い可用性(サービスレベル)が求められるアプリケーションでは使用できない可能性がある点に注意が必要となる。

        それぞれの注意点を考慮した上で、スケールアウトする方法を判断すること。

セッションを利用しない時のメリットとデメリット
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| セッション使用時のデメリットを回避するためには、サーバの処理で必要となるすべてのデータを、リクエストパラメータとして連携することで、実現することができる。
| セッションを利用しない時の、メリットとデメリットは、以下の通りである。

* **メリット**

  * | サーバ側でデータを保持しないため、複数ブラウザや複数タブを使用しても、互いの操作が干渉することはない。そのため、同一処理の画面を複数立ち上げることもできるので、ユーザビリティが損なわれることはない。
  * | サーバ側でデータを保持しないため、持続的に使用するメモリの使用量を、抑えることができる。
  * | APサーバのスケーラビリティを低下させる要因が少なくなる。

* **デメリット**

  * | サーバの処理で必要となるデータを、リクエストパラメータとして送信する必要があるため、画面表示に表示していない項目についても、hidden項目に指定する必要がある。
    | そのため、JSPの実装コードが増える。
    | これは、JSPタグライブラリを作成することで、最小限に抑えることが可能である。
  * | サーバの処理で必要となるデータを、すべてのリクエストで送信する必要があるため、ネットワーク上に流れるデータ量が増える。
  * | 画面表示に必要なデータを、その都度取得する必要があるため、データの取得処理の実行回数が増える。

セッションに格納するデータについて
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
セッションに格納するデータは、以下の点を考慮する必要がある。

* シリアライズすることができるオブジェクト(\ ``java.io.Serializable``\ を実装しているオブジェクト)であること。
* メモリ枯渇の原因となるような容量の大きいオブジェクトでないこと。

シリアライズ可能なオブジェクト
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| セッションに格納するデータは、特定の条件下において、ディスク、またはネットワークへの入出力が行われる可能性がある。
| そのため、シリアライズすることができるオブジェクトである必要がある。

ディスクへの入出力が発生するケースは、以下の通りである。

* | アクティブなセッションが存在する状態で、アプリケーションサーバが停止された場合、セッションおよびセッションに格納されていたデータは、ディスクに退避される。
  | 退避されたセッション、および格納されていたデータは、アプリケーションサーバ起動時に復元される。
  | データの復元に関するこの動作は、アプリケーションサーバによってサポート状況が異なる。

* | セッションの格納領域が溢れそうになった場合や、最終アクセスから一定時間アクセスがない場合、セッションのスワップアウトが発生する可能性がある。
  | スワップアウトされたセッションは、アクセスが発生した際にスワップインされる。
  | スワップアウトの発生条件などは、アプリケーションサーバによって異なる。

ネットワークへの入出力が発生するケースは、以下の通りである。

* | セッションを、別のアプリケーションサーバにレプリケーションする場合、セッションに格納したデータが、ネットワークを経由して、別のアプリケーションサーバに送信される。

セッションに格納するデータの容量
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
**セッションに格納するデータは、できる限りコンパクトにすることを推奨する。**

セッションに格納されているデータの容量が大きい場合は、致命的なパフォーマンス低下を引き起こす原因となるので、容量の大きいデータは、セッションに格納しないように設計することを推奨する。

パフォーマンス低下を引き起こす主な原因は、以下の通り。

* | セッションに容量の大きいデータを格納する場合、メモリ枯渇が発生しないようにするために、アプリケーションサーバの設定をスワップアウトが発生しやすい設定にしておく必要がある。
  | スワップアウト処理は、「重い」処理であるため、スワップアウトが頻繁に発生すると、アプリケーション全体のパフォーマンスに影響を与える可能性がある。
  | スワップアウトに関する動作や設定方法は、アプリケーションサーバによってサポート状況が異なる。

* | セッションをレプリケーションする場合、オブジェクトのシリアライズとデシリアライズが行われる。
  | 容量の大きいオブジェクトのシリアライズとデシリアライズ処理は、「重い」処理であるため、レスポンスタイムなどのパフォーマンスに影響を与える可能性がある。
  

セッションに格納するデータをコンパクトにするために、以下の条件にあてはまるデータについては、セッションスコープではなく、リクエストスコープに格納することを検討すること。

* | 画面操作で編集することができない読み取り専用のデータ。
  | データが必要になったタイミングで最新のデータを取得し、取得したデータをリクエストスコープに格納することでView(JSP)で表示すれば、セッションに格納する必要はない。
* | 画面操作で編集できるが、生存期間がユースケース内の画面操作に閉じているデータ。
  | HTMLフォームのhidden項目として、全ての画面遷移でデータを引き回せば、セッションに格納する必要はない。

APサーバ多重化時の考慮点について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 通常のシステム構成では、アプリケーションサーバが1台で構成されることはほとんどなく、可用性要件、性能要件などを考慮して、複数台で構成することになる。
| そのため、セッションにデータを格納する場合は、システム要件にあわせて以下の何れかの仕組みを適用する必要がある。

1. | 高い可用性(サービスレベル)が求められるシステムの場合は、APサーバダウン時に別のAPサーバで処理が継続できるようにする必要がある。
   | APサーバダウン時に別のAPサーバで処理を継続するためには、全てのAPサーバでセッション情報を共有しておく必要があるので、アプリケーションサーバをクラスタ構成としてセッションをレプリケーションする必要がある。
   | セッション情報を共有する別の方法としては、セッションの保存先をOracle Coherenceのようなキャッシュサーバやデータベースにすることで実現することも可能である。
   | APサーバの台数、セッションに格納されるデータの容量、同時に貼らせるセッション数が大量になる場合は、セッションの保存先をOracle Coherenceのようなキャッシュサーバやデータベースにすることを検討した方がよい。
  
2. | 高い可用性(サービスレベル)が求められないシステムの場合は、APサーバダウン時に別のAPサーバで処理を継続できるようにする必要はない。
   | そのため、全てのAPサーバでセッション情報を共有する必要はないので、ロードバランサの機能を使って同一セッション内で発生するリクエストを全て同じAPサーバに振り分けるようにすればよい。

.. warning::

    本ガイドラインで推奨している方法でWebアプリケーションを作成した場合、以下のデータがセッションに格納されるため、何れかの仕組みを適用する必要がある。
    
    * Spring Securityの認証処理で認証されたユーザ情報。
    * Spring SecurityのCSRFトークンチェックで払い出されたトークン値。
    * 共通ライブラリから提供しているトランザクショントークンチェックで払い出されたトークン値。

セッションの保存先について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| セッションの保存先は、APサーバのメモリだけではなく、Key-Value StoreやOracle Coherenceのようなインメモリデータグリッドにすることも可能である。
| スケーラビリティが求められる場合は検討の余地がある。
| セッションの保存先を変更する際の実装方法については、APサーバーや保存先によって異なるため、本ガイドラインでは説明は割愛する。

|

How to use
--------------------------------------------------------------------------------

本ガイドラインでは、セッションにデータを格納する場合は、以下のいずれかの方法を使用して行うことを推奨している。

#. :ref:`session-management_how_to_use_sessionattributes`
#. :ref:`session-management_how_to_use_sessionscope`

.. warning::
 
    Controllerのハンドラメソッドの引数に ``HttpSession`` オブジェクトを指定することで、 ``HttpSession`` のAPIを直接呼び出すことができるが、
    **原則としてはHttpSessionのAPIを直接使用しないことを強く推奨する。** 

    ``HttpSession`` を直接使わないと実現できない処理については、 ``HttpSession`` のAPIを直接使用してもよいが、
    多くの業務処理において、HttpSessionのAPIを直接使用する必要はないため、原則Controllerのハンドラメソッドの引数として、 ``HttpSession`` オブジェクトを指定しないようにすること。

.. _session-management_how_to_use_sessionattributes:

\ ``@SessionAttributes``\ アノテーションの使用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
\ ``@SessionAttributes``\ アノテーションは、Controller内で行われる画面遷移において、データを持ち回る場合に使用する。

| ただし、入力画面、確認画面、完了画面がそれぞれ１ページで構成されるような場合は、セッションを使わずにHTMLフォームのhiddenを使ってデータを持ち回った方がよい。
| 入力画面が複数のページで構成されるような場合や、複雑な画面遷移を伴う場合は、 \ ``@SessionAttributes``\ アノテーションを使用してフォームオブジェクトをセッションに格納する方法の採用すべきか検討すること。
| フォームオブジェクトをセッションに格納することで、アプリケーションの設計及び実装がシンプルになる可能性がある。

 .. figure:: ./images/session-management_overview_sessionattributes.png
   :alt:
   :width: 80%
   :align: center

セッションに格納するオブジェクトの指定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\ ``@SessionAttributes``\ アノテーションをクラスに指定し、セッションに格納するオブジェクトを指定する。

 .. code-block:: java

    @Controller
    @RequestMapping("wizard")
    @SessionAttributes(types = { WizardForm.class, Entity.class }) // (1)
    public class WizardController {
        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@SessionAttributes``\ アノテーションのtypes属性に、セッションに格納するオブジェクトの型を指定する。
        | \ ``@ModelAttribute``\ アノテーション、または\ ``Model``\ のaddAttributeメソッドを使用して、\ ``Model``\ オブジェクトに追加されたオブジェクトのうち、types属性で指定した型に一致するオブジェクトが、セッションに格納される。
        | 上記例では、 \ ``WizardForm``\ クラスと \ ``Entity``\ クラスのオブジェクトが、セッションに格納される。

 
 .. note:: **ライフサイクルの管理単位**

    \ ``@SessionAttributes``\ アノテーションを使って、セッションに格納したオブジェクトは、Controller単位で、ライフサイクルが管理される。

    \ ``SessionStatus``\ オブジェクトのsetCompleteメソッドを呼び出すと、\ ``@SessionAttributes``\ アノテーションで指定したオブジェクトが、すべてセッションから削除される。
    そのため、ライフサイクルが異なるオブジェクトを、セッションに格納する場合は、Controllerを分割する必要がある。

 .. warning:: **@SessionAttributesアノテーション使用時の注意点**

    Controller単位で、ライフサイクルされると上で説明したが、複数のControllerで同じ属性名のオブジェクトを、\ ``@SessionAttributes``\ アノテーションを使って、セッションに格納した場合は、
    Controllerをまたいで、ライフサイクルが管理される。

    別ウィンドウやタブを開いて、同時に画面操作できる処理の場合は、同じオブジェクトに対してアクセスすることになるため、不具合を引き起こす原因になりうる。
    そのため、複数のControllerで、同じフォームオブジェクトのクラスを使用する場合は、\ ``@ModelAttribute``\ アノテーションのvalue属性に、それぞれ別の値(属性名)を指定した上で、 ``@SessionAttributes`` アノテーションの value属性に ``@ModelAttribute``\ アノテーションのvalue属性に指定した値と同じ値を指定すること。

| セッションに格納するオブジェクトの指定は、属性名で指定することも出来る。
| 以下に、指定方法を示す。

 .. code-block:: java

    @Controller
    @RequestMapping("wizard")
    @SessionAttributes(value = { "wizardCreateForm" }) // (2)
    public class WizardController {

        // ...

        @ModelAttribute(value = "wizardCreateForm")
        public WizardForm setUpWizardForm() {
            return new WizardForm();
        }

        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (2)
      - | \ ``@SessionAttributes``\ アノテーションのvalue属性に、セッションに格納するオブジェクトの属性名を指定する。
        | \ ``@ModelAttribute``\ アノテーション、または\ ``Model``\ のaddAttributeメソッドを使用して、\ ``Model``\ オブジェクトに追加されたオブジェクトのうち、value属性で指定した属性名に一致するオブジェクトが、セッションに格納される。
        | 上記例では、属性名が\ ``"wizardCreateForm"``\ のオブジェクトが、セッションに格納される。

セッションにオブジェクトを追加
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
セッションにオブジェクトを追加する場合、以下2つの方法を使用する。

* \ ``@ModelAttribute``\ アノテーションが付与されたメソッドにて、セッションに追加するオブジェクトを返却する。
* \ ``Model``\ オブジェクトのaddAttributeメソッドを使用して、セッションに格納するオブジェクトを追加する。

\ ``Model``\ オブジェクトに追加されたオブジェクトは、\ ``@SessionAttributes``\ アノテーションのtypesと、value属性の属性値にしたがって、
セッションに格納されるため、Controllerのハンドラメソッドで、セッションを意識した実装を行う必要はない。

| \ ``@ModelAttribute``\ アノテーションが付与されたメソッドを使用して、セッションに格納するオブジェクトを返却する方法について、説明する。
| フォームオブジェクトをセッションに格納する場合は、この方法を使用して、オブジェクトを生成することを推奨する。

 .. code-block:: java

    @ModelAttribute(value = "wizardForm") // (1)
    public WizardForm setUpWizardForm() {
        return new WizardForm();
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | \ ``Model``\ オブジェクトに格納する属性名を、value属性に指定する。
        | 上記例では、返却したオブジェクトが、\ ``"wizardForm"``\ という属性名でセッションに格納される。
        | value属性を指定した場合、セッションにオブジェクトを格納した後のリクエストで、\ ``@ModelAttribute``\ アノテーションの付与されたメソッドが呼び出されなくなるため、無駄なオブジェクトの生成が行われないというメリットがある。

 .. warning:: **@ModelAttributeアノテーションのvalue属性を省略した場合の動作について**

    value属性を省略した場合、デフォルトの属性名を生成するために、すべてのリクエストで、\ ``@ModelAttribute``\ アノテーションの付与されたメソッドが呼ばれる。
    そのため、無駄なオブジェクトが生成されるというデメリットがあるので、 **セッションに格納する場合は、この方法は原則使用しないこと。**
    
     .. code-block:: java
    
        @ModelAttribute // (1)
        public WizardForm setUpWizardForm() {
            return new WizardForm();
        }
    
     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :widths: 10 90
        :header-rows: 1
    
        * - 項番
          - 説明
        * - | (1)
          - | \ ``@ModelAttribute``\ アノテーションが付与されたメソッドにて、セッションに追加するオブジェクトを生成し、返却する。
            | 上記例では、\ ``"wizardForm"``\ アノテーションという属性名で返却したオブジェクトが、セッションに格納される。

| \ ``Model``\ オブジェクトのaddAttributeメソッドを使用し、セッションにオブジェクトを追加する方法について、説明する。
| Domainオブジェクトをセッションに格納する場合は、この方法を使用して、オブジェクトを追加することになる。

 .. code-block:: java

    @RequestMapping(value = "update/{id}", params = "form1")
    public String updateForm1(@PathVariable("id") Integer id, WizardForm form,
            Model model) {
        Entity loadedEntity = entityService.getEntity(id);
        model.addAttribute(loadedEntity); // (3)
        beanMapper.map(loadedEntity, form);
        return "wizard/form1";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (3)
      - | \ ``Model``\ オブジェクトのaddAttributeメソッドを使用して、セッションに格納するオブジェクトを追加する。
        | 上記例では、\ ``"entity"``\ という属性名で、ドメイン層から取得したオブジェクトを、セッションに格納している。

セッションに格納されているオブジェクトの取得
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| セッションに格納されているオブジェクトは、Controllerのハンドラメソッドの引数として、受け取ることができる。
| セッションに格納されているオブジェクトは、\ ``@SessionAttributes``\ アノテーションの属性値にしたがって、\ ``Model``\ オブジェクトに格納されるため、Controllerのハンドラメソッドでは、セッションを意識した実装を行う必要はない。

 .. code-block:: java

    @RequestMapping(value = "save", method = RequestMethod.POST)
    public String save(@Validated({ Wizard1.class, Wizard2.class,
            Wizard3.class }) WizardForm form,   // (1)
            BindingResult result,
            Entity entity,                      // (2)
            RedirectAttributes redirectAttributes) {
        // ...
        return "redirect:/wizard/save?complete";
    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | \ ``Model``\ オブジェクトに格納されているオブジェクトを取得する。
        | 上記例では、\ ``"wizardForm"``\ という属性名でセッションスコープに格納されているオブジェクトが、引数formに渡される。
        | ``@Validated`` アノテーションで指定している ``Wizard1.class`` , ``Wizard2.class`` , ``Wizard3.class`` については、 Appendixの :ref:`session-management_appendix_sessionattribute` を参照されたい。
    * - | (2)
      - | 上記例では、\ ``"entity"``\ という属性名でセッションスコープに格納されているオブジェクトが、引数entityに渡される。

Controllerのハンドラメソッドの引数に渡すオブジェクトが、\ ``Model``\ オブジェクトに存在しない場合、\ ``@ModelAttribute``\ アノテーションの指定の有無で、動作が変わる。

* \ ``@ModelAttribute``\ アノテーションを指定していない場合は、新しいオブジェクトが生成されて引数に渡される。
  生成されたオブジェクトは ``Model`` オブジェクトに格納されるため、セッションにも格納される。

 .. note:: **リダイレクト時の動作について**

    遷移先をリダイレクトにした場合は、生成されたオブジェクトは、セッションに格納されない。
    そのため、生成されたオブジェクトを、リダイレクト先の処理で参照したい場合は、\ ``RedirectAttributes``\ のaddFlashAttributeメソッドを使用して、Flashスコープにオブジェクトを格納する必要がある。

* \ ``@ModelAttribute`` アノテーションを指定している場合は、\ ``org.springframework.web.HttpSessionRequiredException``\ が発生する。

 .. code-block:: java

    @RequestMapping(value = "save", method = RequestMethod.POST)
    public String save(@Validated({ Wizard1.class, Wizard2.class,
            Wizard3.class }) WizardForm form, // (3)
            BindingResult result,
            @ModelAttribute Entity entity, // (4)
            RedirectAttributes redirectAttributes) {
        // ...
        return "redirect:/wizard/save?complete";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (3)
      - | \ ``@Validated``\ アノテーションで、特定の検証グループ(\ ``Wizard1.class``\ , \ ``Wizard2.class``\ , \ ``Wizard3.class``\ )を設定して入力チェックを行っている。
        | 入力チェックの詳細については、\ :doc:`../WebApplicationDetail/Validation`\ を参照されたい。
    * - | (4)
      - | 引数に、\ ``@ModelAttribute``\ アノテーションを指定している場合、セッションに対象のオブジェクトが存在しない時に呼び出されると、\ ``HttpSessionRequiredException``\ が発生する。
        | \ ``HttpSessionRequiredException``\ は、ブラウザバックや、URL直接指定のアクセスなどの、クライアントの操作に起因して発生する例外になるため、クライアントエラーとして、例外ハンドリングを行う必要がある。

\ ``HttpSessionRequiredException``\ をクライアントエラーとするための設定は、以下の通りである。

- spring-mvc.xml

 .. code-block:: xml

    <bean class="org.terasoluna.gfw.web.exception.SystemExceptionResolver">
        <property name="exceptionCodeResolver" ref="exceptionCodeResolver" />
        <!-- ... -->
        <property name="exceptionMappings">
            <map>
                <!-- ... -->
                <entry key="HttpSessionRequiredException "
                       value="common/error/operationError" /> <!-- (5) -->
            </map>
        </property>
        <property name="statusCodes">
            <map>
                <!-- ... -->
                <entry key="common/error/operationError" value="400" /> <!-- (6) -->
            </map>
        </property>
        <!-- ... -->
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (5)
      - | 共通ライブラリから提供している\ ``SystemExceptionResolver``\ の\ ``exceptionMappings``\ に、\ ``HttpSessionRequiredException``\ の例外ハンドリングの定義を追加する。
        | 上記例では、  例外発生時の遷移先として、\ ``/WEB-INF/views/common/error/operationError.jsp``\ を指定している。
    * - | (6)
      - | \ ``SystemExceptionResolver``\ の\ ``statusCodes``\ に、\ ``HttpSessionRequiredException``\ 発生時の、HTTPレスポンスコードを指定する。
        | 上記例では、  例外発生時のHTTPレスポンスコードとして、 Bad Request(\ ``400``\ )を指定している。

- applicationContext.xml

 .. code-block:: xml

    <bean id="exceptionCodeResolver"
        class="org.terasoluna.gfw.common.exception.SimpleMappingExceptionCodeResolver">
        <!-- Setting and Customization by project. -->
        <property name="exceptionMappings">
            <map>
                <!-- ... -->
                <entry key="HttpSessionRequiredException" value="w.xx.0003" /> <!-- (7) -->
            </map>
        </property>
        <property name="defaultExceptionCode" value="e.xx.0001" /> <!-- (8) -->
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (7)
      - | 共通ライブラリから提供している\ ``SimpleMappingExceptionCodeResolver``\ の\ ``exceptionMappings``\ に、\ ``HttpSessionRequiredException``\ の例外ハンドリングの定義を追加する。
        | 上記例では、  例外発生時の例外コードとして、\ ``"w.xx.0003"``\ を指定している。
        | この設定を追加しない場合は、デフォルトの例外コードが、ログに出力される。
    * - | (8)
      - | 例外発生時のデフォルトの例外コード。

セッションに格納したオブジェクトの削除
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| \ ``@SessionAttributes``\ を用いてセッションに格納したオブジェクトを削除する場合、 \ ``org.springframework.web.bind.support.SessionStatus``\ のsetCompleteメソッドを、Controllerのハンドラメソッドから呼び出す。
| \ ``SessionStatus``\ オブジェクトのsetCompleteメソッドを呼び出すと、 \ ``@SessionAttributes``\ アノテーションの属性値に指定されているオブジェクトが、セッションから削除される。

 .. note:: **セッションから削除されるタイミングについて**

    \ ``SessionStatus``\ オブジェクトのsetCompleteメソッドを呼び出すことで、\ ``@SessionAttributes``\ アノテーションの属性値に指定されているオブジェクトが、セッションから削除される。
    ただし、実際に削除されるタイミングは、setCompleteメソッドを呼び出したタイミングではない。

    \ ``SessionStatus``\ オブジェクトのsetCompleteメソッド自体は、内部のフラグを変更しているだけなので、実際の削除は、Controllerのハンドラメソッドの処理が終了した後に、フレームワークによって行われる。

 .. note:: **View(JSP)からのオブジェクトの参照について**

    \ ``SessionStatus``\ オブジェクトのsetCompleteメソッドを呼び出すことで、セッションから削除されるが、同じオブジェクトが、\ ``Model``\ オブジェクトに残っているため、View(JSP)から参照することができる。

|

セッションに格納したオブジェクトの削除は、以下3カ所で行う必要がある。

* | 完了画面を表示するためのリクエスト。**(必須)**
  | 完了画面を表示した後に、セッションに格納したオブジェクトにアクセスすることはないため、不要になったオブジェクトを削除する。

 .. warning:: **削除が必要な理由**

    セッションに格納されているオブジェクトは、ガベージコレクションの対象とならないため、不要になったオブジェクトを削除しないと、メモリ枯渇の原因になりうる。
    また、不要なオブジェクトがセッションに格納されていると、セッションのスワットアウトが発生した際の処理が重くなり、アプリケーション全体の性能に影響を与える可能性がある。

* | 一連の画面操作を中止するためのリクエスト。**(必須)**
  | 「メニューへ戻る」や「中止」などの、一連の画面操作を中止するためのイベントについても、セッションに格納したオブジェクトにアクセスすることはないため、不要になったオブジェクトを削除すること。

* | 入力画面を初期表示するためのリクエスト。(任意)

 .. warning:: **削除が必要な理由**

    画面操作の途中でブラウザやタブを閉じた場合、セッションに格納されているフォームオブジェクトに入力途中の情報が残るため、初期表示時に削除しないと、入力途中の情報が画面に表示されてしまう。
    ただし、入力途中の情報が画面に表示されてもよい場合は、初期表示するためのリクエストで削除は必須ではない。

|

完了画面を表示するためのリクエストで削除する際の実装例は、以下の通りである。

 .. code-block:: java

    // (1)
    @RequestMapping(value = "save", method = RequestMethod.POST)
    public String save(@ModelAttribute @Validated({ Wizard1.class,
            Wizard2.class, Wizard3.class }) WizardForm form,
            BindingResult result, Entity entity,
            RedirectAttributes redirectAttributes) {
        // ...
        return "redirect:/wizard/save?complete"; // (2)
    }

    // (3)
    @RequestMapping(value = "save", params = "complete", method = RequestMethod.GET)
    public String saveComplete(SessionStatus sessionStatus) {
        sessionStatus.setComplete(); // (4)
        return "wizard/complete";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | 更新処理を行うためのハンドラメソッド。
    * - | (2)
      - | 完了画面を表示するためのリクエスト(3)へ、リダイレクトする。
    * - | (3)
      - | 完了画面を表示するためのハンドラメソッド。
    * - | (4)
      - | \ ``SessionStatus``\ オブジェクトのsetCompleteメソッドを呼び出し、オブジェクトをセッションから削除する。
        | \ ``Model``\ オブジェクトに同じオブジェクトが残っているため、直接、View(JSP)の表示処理に影響は与えない。

一連の画面操作を中止するためのリクエストで削除する際の実装例は、以下の通りである。

 .. code-block:: java

    // (1)
    @RequestMapping(value = "save", params = "cancel", method = RequestMethod.POST)
    public String saveCancel(SessionStatus sessionStatus) {
        sessionStatus.setComplete(); // (2)
        return "redirect:/wizard/menu"; // (3)
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | 一連の画面操作を中止するためのハンドラメソッド。
    * - | (2)
      - | \ ``SessionStatus``\ オブジェクトのsetCompleteメソッドを呼び出し、オブジェクトをセッションから削除する。
    * - | (3)
      - | 上記例では、メニュー画面へ、リダイレクトしている。

入力画面を、初期表示するためのリクエストで削除する際の実装例は、以下の通りである。

 .. code-block:: java

    // (1)
    @RequestMapping(value = "create", method = RequestMethod.GET)
    public String initializeCreateWizardForm(SessionStatus sessionStatus) {
        sessionStatus.setComplete();              // (2)
        return "redirect:/wizard/create?form1";   // (3)
    }

    // (4)
    @RequestMapping(value = "create", params = "form1")
    public String createForm1() {
        return "wizard/form1";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | 入力画面を初期表示するためのハンドラメソッド。
    * - | (2)
      - | \ ``SessionStatus``\ オブジェクトのsetCompleteメソッドを呼び出す。
    * - | (3)
      - | 入力画面を表示するためのリクエスト(4)へ、リダイレクトする。
        | \ ``SessionStatus``\ オブジェクトのsetCompleteメソッドを呼び出すことで、
        | セッションからは削除されるが、\ ``Model``\ オブジェクトに同じオブジェクトが残っているため、
        | 直接View(JSP)を呼び出してしまうと、入力途中の情報が表示されてしまう。
        | そのため、\ **セッションから削除したうえで、入力画面を表示するためのリクエストへ、リダイレクトする必要がある。**\
    * - | (4)
      - | 入力画面を表示するためのハンドラメソッド。

@SessionAttributesを使った処理の実装例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
より具体的な実装例については、Appendixの\ :ref:`session-management_appendix_sessionattribute`\ を参照されたい。

.. _session-management_how_to_use_sessionscope:

Spring FrameworkのsessionスコープのBeanの使用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Spring FrameworkのsessionスコープのBeanは、
| 複数のControllerをまたいだ画面遷移において、データを持ち回る場合に使用する。

 .. figure:: ./images/session-management_overview_sessionscope.png
   :alt:
   :width: 90%
   :align: center

sessionスコープのBean定義
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Spring FrameworkのsessionスコープのBeanを、定義する。

sessionスコープのBeanを定義する方法は、以下2種類の方法がある。

* component-scanを使用してbeanを定義する。
* Bean定義ファイル(XML)にbeanを定義する。

component-scanを使用する方法を、以下に示す。

- クラス

 .. code-block:: java

    @Component
    @Scope(value = "session", proxyMode = ScopedProxyMode.TARGET_CLASS) // (1)
    public class SessionCart implements Serializable {

        private static final long serialVersionUID = 1L;

        private Cart cart;

        public Cart getCart() {
            if (cart == null) {
                cart = new Cart();
            }
            return cart;
        }

        public void setCart(Cart cart) {
            this.cart = cart;
        }

        public void clearCart() { // (2)
            cart.clearCart();
        }
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | Beanのスコープを\ ``"session"``\ にする。また、proxyMode 属性で \ ``"ScopedProxyMode.TARGET_CLASS"``\ を指定し、scoped-proxyを有効にする。
    * - | (2)
      - | 注文が完了した際にカートの状態をクリア(カート内の商品を削除)するためのメソッドを用意する。
 .. note::

    JPAで扱うEntityクラスをsessionスコープのBeanとして定義したい場合は、直接sessionスコープのBeanとして定義するのではなく、ラッパークラスを用意することを推奨する。

    JPAで扱うEntityクラスをsessionスコープのBeanとして定義すると、JPAのAPIでsessionスコープのBeanを直接扱うことができない(直接あつかうと、エラーとなる)。
    そのため、JPAで扱うことができるEntityオブジェクトへの変換処理が、必要になってしまう。

    上記例では、\ ``Cart``\ というJPAのEntityクラスを、\ ``SessionCart``\ というラッパークラスに包んで、sessionスコープのBeanとしている。
    こうすることで、JPAで扱うことができるEntityオブジェクトへの変換処理が不要となるため、Controllerで行う処理がシンプルになる。

 .. note:: **scoped-proxyを有効化する理由について**

     sessionスコープのBeanをsingletonスコープのControllerにInjectするために、scoped-proxyを有効化する必要がある。

- spring-mvc.xml

 .. code-block:: xml

    <context:component-scan base-package="xxx.yyy.zzz.app" /> // (2)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (2)
      - | ``<context:component-scan>`` 要素でベースとなるパッケージを指定する。

|

Bean定義ファイル(XML)に定義する方法を、以下に示す。

- JavaBean

 .. code-block:: xml

    <beans:bean id="sessionCart" class="xxx.yyy.zzz.app.SessionCart"
                scope="session"> <!-- (3) -->
        <aop:scoped-proxy /> <!-- (4) -->
    </beans:bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (3)
      - | Beanのスコープを\ ``"session"``\ にする。
    * - | (4)
      - | ``<aop:scoped-proxy />`` 要素を指定し、scoped-proxyを有効にする。

sessionスコープのBeanの利用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| sessionスコープのBeanを利用して、オブジェクトをセッションに格納・取得する場合は、
| sessionスコープのBeanを、ControllerにInjectする。

 .. code-block:: java

    @Inject
    SessionCart sessionCart; // (1)

    @RequestMapping(value = "add")
    public String addCart(@Validated ItemForm form, BindingResult result) {
        if (result.hasErrors()) {
            return "item/item";
        }
        CartItem cartItem = beanMapper.map(form, CartItem.class);
        Cart addedCart = cartService.addCartItem(sessionCart.getCart(), // (2)
                cartItem);
        sessionCart.setCart(addedCart); // (3)
        return "redirect:/cart";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | sessionスコープのBeanを、ControllerにInjectする。
    * - | (2)
      - | sessionスコープのBeanのメソッド呼び出しを行うと、セッションに格納されているオブジェクトが返却される。
        | セッションにオブジェクトが格納されていない場合は、新たに生成されたオブジェクトが返却され、セッションにも格納される。
        | 上記例では、カートに追加する前に在庫数などのチェックを行うため、Serviceのメソッドを呼び出している。
    * - | (3)
      - | 上記例では、\ ``CartService``\ のaddCartItemメソッドの引数に渡した\ ``Cart``\ オブジェクトと、
        | 返り値で返却される\ ``Cart``\ オブジェクトが、別のインスタンスになる可能性があるため、
        | 返却された\ ``Cart``\ オブジェクトをsessionスコープのBeanに設定している。

 .. note:: **View(JSP)からsessionスコープのBeanを参照する方法**

     SpEL(Spring Expression Language)式を用いることでControllerで\ ``Model``\ オブジェクトにBeanを追加しなくても、JSPからsessionスコープのBeanを参照することができる。

 .. code-block:: jsp

    <spring:eval var="cart" expression="@sessionCart.cart" />     <%-- (1) --%>
    
    <%-- omitted --%>
    
    <c:forEach var="item" items="${cart.cartItems}">     <%-- (2) --%>
        <tr>
            <td>${f:h(item.id)}</td>
            <td>${f:h(item.itemCode)}</td>
            <td>${f:h(item.quantity)}</td>
        </tr>
    </c:forEach>　　　　
    
 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | sessionスコープのBeanを参照する
    * - | (2)
      - | sessionスコープのBeanを表示する


セッションに格納したオブジェクトの削除
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 不要になったオブジェクトをセッション上から削除する場合は、sessionスコープのBeanのフィールドをリセットする。

 .. note:: 

    sessionスコープのBeanは、セッションが切れる時にDIコンテナによって破棄される。
    
    DIコンテナがsessionスコープのBeanのライフサイクルを管理しているので、Bean自体の破棄はDIコンテナにまかせる。

 .. code-block:: java

    @Controller
    @RequestMapping("order")
    public class OrderController {

        @Inject
        SessionCart sessionCart; // (1)

        // ...

        @RequestMapping(method = RequestMethod.POST)
        public String order() {
            // ...
            return "redirect:/order?complete";
        }

        @RequestMapping(params = "complete", method = RequestMethod.GET)
        public String complete(Model model, SessionStatus sessionStatus) {
            sessionCart.clearCart(); // (2)
            return "order/complete";
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | sessionスコープのBeanをインジェクションする。
    * - | (2)
      - | sessionスコープのBeanの状態をクリアし、注文済みの商品をカートから削除する

sessionスコープのBeanを使った処理の実装例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
より具体的な実装例については、Appendixの\ :ref:`session-management_appendix_sessionscope`\ を参照されたい。

セッション操作のデバッグログ出力
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| セッションに対して行われた操作を、デバッグログに出力するクラスを、共通ライブラリとして提供している。
| セッションに対する操作が、想定通りに動作しているか確認する必要がある場合に、このクラスで出力するログが有効である。

共通ライブラリの詳細は、\ :ref:`logging_appendix_httpsessioneventlogginglistener`\ を参照されたい。

JSPの暗黙オブジェクト ``sessionScope`` を使用する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
JSPの暗黙オブジェクトである \ ``sessionScope``\ を使用する場合は、 pageディレクティブのsession属性の値を ``true`` にする必要がある。
ブランクプロジェクトから提供している :file:`include.jsp` では、 ``false`` となっている。

:file:`include.jsp` は、 :file:`src/main/webapp/WEB-INF/views/common` ディレクトリに格納されている。

- :file:`include.jsp`

 .. code-block:: jsp

    <%@ page session="true"%>     <%-- (1) --%>
    
    <%-- omitted --%>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - |  pageディレクティブのsession属性の値を ``true`` にする。


|

How to extend
--------------------------------------------------------------------------------

同一セッション内のリクエストの同期化
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| \ ``@SessionAttributes``\ アノテーション、またはsessionスコープのBeanを使用する場合は、同一セッション内のリクエストを同期化することを推奨する。
| 同期化しない場合、セッションに格納されているオブジェクトに、同時にアクセスする可能性があるため、想定外のエラーや、動作を引き起こす原因になりうる。

| 例えば、入力チェック済みのフォームオブジェクトに対して、不正な値が設定される可能性がある。
| これを防ぐ方法として、\ ``org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter``\ の、synchronizeOnSessionをtrueにして、同一セッション内のリクエストを同期化することを、強く推奨する。

以下のようなBeanPostProcessorを作成し、Bean定義することで実現できる。

- コンポーネント

 .. code-block:: java

    package com.example.app.config;

    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.beans.BeansException;
    import org.springframework.beans.factory.config.BeanPostProcessor;
    import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter;

    public class EnableSynchronizeOnSessionPostProcessor 
        implements BeanPostProcessor {
        private static final Logger logger = LoggerFactory
                .getLogger(EnableSynchronizeOnSessionPostProcessor.class);

        @Override
        public Object postProcessBeforeInitialization(Object bean, String beanName) 
            throws BeansException {
            // NO-OP
            return bean;
        }

        @Override
        public Object postProcessAfterInitialization(Object bean, String beanName) 
            throws BeansException {
            if (bean instanceof RequestMappingHandlerAdapter) {
                RequestMappingHandlerAdapter adapter = 
                    (RequestMappingHandlerAdapter) bean;
                logger.info("enable synchronizeOnSession => {}", adapter);
                adapter.setSynchronizeOnSession(true); // (1)
            }
            return bean;
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | \ ``org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter``\ のsetSynchronizeOnSessionメソッドの引数に、\ ``true``\ を指定すると、同一セッション内でのリクエストが同期化される。

- spring-mvc.xml

 .. code-block:: xml

     <bean class="com.example.app.config.EnableSynchronizeOnSessionPostProcessor" /> <!-- (2) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (2)
      - | (1)で作成した、``BeanPostProcessor`` をBeanを定義する。

|

Appendix
--------------------------------------------------------------------------------

.. _session-management_appendix_sessionattribute:

\ ``@SessionAttributes``\ アノテーションを使ったウィザード形式の画面遷移の実装例
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
ウィザード形式の画面遷移を行う処理を例に、\ ``@SessionAttributes``\ アノテーションを使った実装の説明を行う。

処理の仕様は、以下の通りとする。

* Entityの登録と、更新を行うための画面を提供する。
* 入力画面は、3画面で構成され、各画面で1項目ずつ入力を行う。
* 入力した値は、保存(登録/更新)する前に、確認画面で確認できる。
* 入力チェックは、画面遷移するタイミングで行い、エラーがある場合は、入力画面に戻る。
* 保存(登録/更新)する前に、すべての入力値に対する入力チェックを再度行い、エラーがある場合は、不正操作を通知するエラー画面を表示する。
* すべての入力値に対するチェックが妥当な場合は、入力データをデータベースに保存する。

基本的な画面遷移は、以下の通りとする。

 .. figure:: ./images/session-management_appendix_screenflow1.png
   :alt: Invalidate session by processing of Web Application
   :width: 100%
   :align: center

実装例は、以下の通りである。

- フォームオブジェクト

 .. code-block:: java

    public class WizardForm implements Serializable {

        private static final long serialVersionUID = 1L;

        // (1)
        @NotEmpty(groups = { Wizard1.class })
        private String field1;

        // (2)
        @NotEmpty(groups = { Wizard2.class })
        private String field2;

        // (3)
        @NotEmpty(groups = { Wizard3.class })
        private String field3;

        // ...

        // (4)
        public static interface Wizard1 {
        }

        // (5)
        public static interface Wizard2 {
        }

        // (6)
        public static interface Wizard3 {
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | 1ページ目の入力画面で入力するフィールド。
    * - | (2)
      - | 2ページ目の入力画面で入力するフィールド。
    * - | (3)
      - | 3ページ目の入力画面で入力するフィールド。
    * - | (4)
      - | 1ページ目の入力画面で入力されるフィールドであることを示すための、検証グループインタフェース。
    * - | (5)
      - | 2ページ目の入力画面で入力されるフィールドであることを示すための、検証グループインタフェース。
    * - | (6)
      - | 3ページ目の入力画面で入力されるフィールドであることを示すための、検証グループインタフェース。

 .. note:: **検証グループについて**

    画面遷移時の入力チェックでは、該当ページのフィールドのみチェックする必要がある。
    Bean Validationでは、検証グループを表すクラス、またはインタフェースを設けることで、検証するルールをグループ化することができる。
    今回の実装例のケースでは、画面毎に検証グループを用意することで、画面毎の入力チェックを実現している。

- Controller

 .. code-block:: java

    @Controller
    @RequestMapping("wizard")
    @SessionAttributes(types = { WizardForm.class, Entity.class }) // (7)
    public class WizardController {

        @Inject
        WizardService wizardService;

        @Inject
        Mapper beanMapper;

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (7)
      - | 上記例では、フォームオブジェクト(\ ``WizardForm.class``\ )と、エンティティ(\ ``Entity.class``\ )のオブジェクトを、セッションに格納する。

 .. code-block:: java

        @ModelAttribute("wizardForm") // (8)
        public WizardForm setUpWizardForm() {
            return new WizardForm();
        }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (8)
      - | 上記例では、セッションに格納するフォームオブジェクト(\ ``WizardForm``\ )を生成している。 無駄なオブジェクトの生成をなくすために、\ ``@ModelAttribute``\ アノテーションのvalue属性を指定している。

 .. code-block:: java

        // (9)
        @RequestMapping(value = "create", method = RequestMethod.GET)
        public String initializeCreateWizardForm(SessionStatus sessionStatus) {
            sessionStatus.setComplete();
            return "redirect:/wizard/create?form1";
        }

        // (10)
        @RequestMapping(value = "create", params = "form1")
        public String createForm1() {
            return "wizard/form1";
        }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (9)
      - | 登録用入力画面を、初期表示するためのハンドラメソッド。
        | 操作途中のオブジェクトが、セッションに格納されている可能性があるため、このハンドラメソッドで、セッションに格納されているオブジェクトを削除しておく。
    * - | (10)
      - | １ページ目の登録用入力画面を、表示するためのハンドラメソッド。

 .. code-block:: java

        // (11)
        @RequestMapping(value = "{id}/update", method = RequestMethod.GET)
        public String initializeUpdateWizardForm(@PathVariable("id") Integer id,
                RedirectAttributes redirectAttributes, SessionStatus sessionStatus) {
            sessionStatus.setComplete();
            return "redirect:/wizard/{id}/update?form1";
        }

        // (12)
        @RequestMapping(value = "{id}/update", params = "form1")
        public String updateForm1(@PathVariable("id") Integer id, WizardForm form,
                Model model) {
            Entity loadedEntity = wizardService.getEntity(id);
            beanMapper.map(loadedEntity, form); // (13)
            model.addAttribute(loadedEntity); // (14)
            return "wizard/form1";
        }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (11)
      - | 更新用入力画面を、初期表示するためのハンドラメソッド。
    * - | (12)
      - | １ページ目の更新用入力画面を、表示するためのハンドラメソッド。
    * - | (13)
      - | 取得したエンティティの状態をフォームオブジェクトに設定する。上記例では、DozerというBeanマッパーライブラリを使用している。
    * - | (14)
      - | 取得したエンティティを ``Model`` オブジェクトに追加し、セッションに格納する。
        | 上記例では、\ ``"entity"``\ という属性名で、セッションに格納される。

 .. code-block:: java

        // (15)
        @RequestMapping(value = "save", params = "form2", method = RequestMethod.POST)
        public String saveForm2(@Validated(Wizard1.class) WizardForm form, // (16)
                BindingResult result) {
            if (result.hasErrors()) {
                return saveRedoForm1();
            }
            return "wizard/form2";
        }

        // (17)
        @RequestMapping(value = "save", params = "form3", method = RequestMethod.POST)
        public String saveForm3(@Validated(Wizard2.class) WizardForm form, // (18)
                BindingResult result) {
            if (result.hasErrors()) {
                return saveRedoForm2();
            }
            return "wizard/form3";
        }

        // (19)
        @RequestMapping(value = "save", params = "confirm", method = RequestMethod.POST)
        public String saveConfirm(@Validated(Wizard3.class) WizardForm form, // (20)
                BindingResult result) {
            if (result.hasErrors()) {
                return saveRedoForm3();
            }
            return "wizard/confirm";
        }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (15)
      - | 2ページ目の入力画面を、表示するためのハンドラメソッド。
    * - | (16)
      - | 1ページ目の入力画面で入力された値のみ、入力チェックするために、\ ``@Validated``\ アノテーションのvalue属性に、1ページ目の入力画面の検証グループ(\ ``Wizard1.class``\ )を指定する。
    * - | (17)
      - | 3ページ目の入力画面を、表示するためのハンドラメソッド。
    * - | (18)
      - | 2ページ目の入力画面で入力された値のみ、入力チェックするために、\ ``@Validated``\ アノテーションのvalue属性に、2ページ目の入力画面の検証グループ(\ ``Wizard2.class``\ )を指定する。
    * - | (19)
      - | 確認画面を表示するためのハンドラメソッド。
    * - | (20)
      - | 3ページ目の入力画面で入力された値のみ、入力チェックするために、\ ``@Validated``\ アノテーションのvalue属性に、3ページ目の入力画面の検証グループ(\ ``Wizard3.class``\ )を指定する。

 .. code-block:: java

        // (21)
        @RequestMapping(value = "save", method = RequestMethod.POST)
        public String save(@ModelAttribute @Validated({ Wizard1.class,
                Wizard2.class, Wizard3.class }) WizardForm form, // (22)
                BindingResult result,
                Entity entity, // (23)
                RedirectAttributes redirectAttributes) {
            if (result.hasErrors()) {
                throw new InvalidRequestException(result); // (24)
            }

            beanMapper.map(form, entity);

            entity = wizardService.saveEntity(entity); // (25)

            redirectAttributes.addFlashAttribute(entity); // (26)

            return "redirect:/wizard/save?complete";
        }

        // (27)
        @RequestMapping(value = "save", params = "complete", method = RequestMethod.GET)
        public String saveComplete(SessionStatus sessionStatus) {
            sessionStatus.setComplete();
            return "wizard/complete";
        }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (21)
      - | 保存処理を実行するためのハンドラメソッド。
    * - | (22)
      - | 入力画面で入力された値を全てチェックするために、\ ``@Validated``\ アノテーションのvalue属性に、各入力画面の検証グループインタフェース(\ ``Wizard1.class``\ , \  ``Wizard2.class``\ , \ ``Wizard3.class``\ )を指定する。
    * - | (23)
      - | 保存する\ ``Entity.class``\ のオブジェクトを取得する。
        | 登録処理の場合は、新たに生成されたオブジェクト、更新処理の場合は、(14)の処理でセッションに格納したオブジェクトが取得される。
    * - | (24)
      - | アプリケーションが提供しているボタンを使って、画面遷移を行っていれば、このタイミングでエラーは発生しないので、不正な操作が行われた場合に\ ``InvalidRequestException``\ がthrowされる。
        | なお、\ ``InvalidRequestException``\は共通ライブラリから提供している例外クラスではないため、別途作成する必要がある。
    * - | (25)
      - | 入力値が反映された\ ``Entity.class``\ のオブジェクトを保存する。
    * - | (26)
      - | リダイレクト先のハンドラメソッドで保存した\ ``Entity.class``\ のオブジェクトを参照できるようにするために、Flashスコープに格納する。
    * - | (27)
      - | 完了画面を表示するためのハンドラメソッド。

 .. code-block:: java

        // (28)
        @RequestMapping(value = "save", params = "redoForm1")
        public String saveRedoForm1() {
            return "wizard/form1";
        }

        // (29)
        @RequestMapping(value = "save", params = "redoForm2")
        public String saveRedoForm2() {
            return "wizard/form2";
        }

        // (30)
        @RequestMapping(value = "save", params = "redoForm3")
        public String saveRedoForm3() {
            return "wizard/form3";
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (28)
      - | 1ページ目の入力画面を、再表示するためのハンドラメソッド。
    * - | (29)
      - | 2ページ目の入力画面を、再表示するためのハンドラメソッド。
    * - | (30)
      - | 3ページ目の入力画面を、再表示するためのハンドラメソッド。

- Controllerの全ソース

 .. code-block:: java

    @Controller
    @RequestMapping("wizard")
    @SessionAttributes(types = { WizardForm.class, Entity.class })
    // (7)
    public class WizardController {
    
        @Inject
        EntityService wizardService;
    
        @Inject
        Mapper beanMapper;
    
        @ModelAttribute("wizardForm")
        // (8)
        public WizardForm setUpWizardForm() {
            return new WizardForm();
        }
    
        // (9)
        @RequestMapping(value = "create", method = RequestMethod.GET)
        public String initializeCreateWizardForm(SessionStatus sessionStatus) {
            sessionStatus.setComplete();
            return "redirect:/wizard/create?form1";
        }
    
        // (10)
        @RequestMapping(value = "create", params = "form1")
        public String createForm1() {
            return "wizard/form1";
        }
    
        // (11)
        @RequestMapping(value = "{id}/update", method = RequestMethod.GET)
        public String initializeUpdateWizardForm(@PathVariable("id") Integer id,
                RedirectAttributes redirectAttributes, SessionStatus sessionStatus) {
            sessionStatus.setComplete();
            return "redirect:/wizard/{id}/update?form1";
        }
    
        // (12)
        @RequestMapping(value = "{id}/update", params = "form1")
        public String updateForm1(@PathVariable("id") Integer id, WizardForm form,
                Model model) {
            Entity loadedEntity = wizardService.getEntity(id);
            beanMapper.map(loadedEntity, form); // (13)
            model.addAttribute(loadedEntity); // (14)
            return "wizard/form1";
        }
    
        // (15)
        @RequestMapping(value = "save", params = "form2", method = RequestMethod.POST)
        public String saveForm2(@Validated(Wizard1.class) WizardForm form, // (16)
                BindingResult result) {
            if (result.hasErrors()) {
                return saveRedoForm1();
            }
            return "wizard/form2";
        }
    
        // (17)
        @RequestMapping(value = "save", params = "form3", method = RequestMethod.POST)
        public String saveForm3(@Validated(Wizard2.class) WizardForm form, // (18)
                BindingResult result) {
            if (result.hasErrors()) {
                return saveRedoForm2();
            }
            return "wizard/form3";
        }
    
        // (19)
        @RequestMapping(value = "save", params = "confirm", method = RequestMethod.POST)
        public String saveConfirm(@Validated(Wizard3.class) WizardForm form, // (20)
                BindingResult result) {
            if (result.hasErrors()) {
                return saveRedoForm3();
            }
            return "wizard/confirm";
        }
    
        // (21)
        @RequestMapping(value = "save", method = RequestMethod.POST)
        public String save(@ModelAttribute @Validated({ Wizard1.class,
                Wizard2.class, Wizard3.class }) WizardForm form, // (22)
                BindingResult result, Entity entity, // (23)
                RedirectAttributes redirectAttributes) {
            if (result.hasErrors()) {
                throw new InvalidRequestException(result); // (24)
            }
    
            beanMapper.map(form, entity);
    
            entity = wizardService.saveEntity(entity); // (25)
    
            redirectAttributes.addFlashAttribute(entity); // (26)
    
            return "redirect:/wizard/save?complete";
        }
    
        // (27)
        @RequestMapping(value = "save", params = "complete", method = RequestMethod.GET)
        public String saveComplete(SessionStatus sessionStatus) {
            sessionStatus.setComplete();
            return "wizard/complete";
        }
    
        // (28)
        @RequestMapping(value = "save", params = "redoForm1")
        public String saveRedoForm1() {
            return "wizard/form1";
        }
    
        // (29)
        @RequestMapping(value = "save", params = "redoForm2")
        public String saveRedoForm2() {
            return "wizard/form2";
        }
    
        // (30)
        @RequestMapping(value = "save", params = "redoForm3")
        public String saveRedoForm3() {
            return "wizard/form3";
        }
    
    }


- 1ページ目の入力画面(JSP)

 .. code-block:: jsp

    <html>
    <head>
    <title>Wizard Form(1/3)</title>
    </head>
    <body>
        <h1>Wizard Form(1/3)</h1>
        <form:form action="${pageContext.request.contextPath}/wizard/save" 
            modelAttribute="wizardForm">
            <form:label path="field1">Field1</form:label> : 
            <form:input path="field1" />
            <form:errors path="field1" />
            <div>
                <form:button name="form2">Next</form:button>
            </div>
        </form:form>
    </body>
    </html>

- 2ページ目の入力画面(JSP)

 .. code-block:: jsp

    <html>
    <head>
    <title>Wizard Form(2/3)</title>
    </head>
    <body>
        <h1>Wizard Form(2/3)</h1>
        <%-- (31) --%>
        <form:form action="${pageContext.request.contextPath}/wizard/save" 
            modelAttribute="wizardForm">
            <form:label path="field2">Field2</form:label> : 
            <form:input path="field2" />
            <form:errors path="field2" />
            <div>
                <form:button name="redoForm1">Back</form:button>
                <form:button name="form3">Next</form:button>
            </div>
        </form:form>
    </body>
    </html>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (31)
      - | フォームオブジェクトをセッションに格納しているため、1ページ目の入力画面のフィールドを、hidden項目にする必要はない。

- 3ページ目の入力画面(JSP)

 .. code-block:: jsp

    <html>
    <head>
    <title>Wizard Form(3/3)</title>
    </head>
    <body>
        <h1>Wizard Form(3/3)</h1>
        <%-- (32) --%>
        <form:form action="${pageContext.request.contextPath}/wizard/save" 
            modelAttribute="wizardForm">
            <form:label path="field3">Field3</form:label> : 
            <form:input path="field3" />
            <form:errors path="field3" />
            <div>
                <form:button name="redoForm2">Back</form:button>
                <form:button name="confirm">Confirm</form:button>
            </div>
        </form:form>
    </body>
    </html>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (32)
      - | フォームオブジェクトをセッションに格納しているため、1ページ目と2ページ目の入力画面のフィールドを、hidden項目にする必要はない。

- 確認画面(JSP)

 .. code-block:: jsp

    <html>
    <head>
    <title>Confirm</title>
    </head>
    <body>
        <h1>Confirm</h1>
        <%-- (33) --%>
        <form:form action="${pageContext.request.contextPath}/wizard/save" 
            modelAttribute="wizardForm">
            <div>
                Field1 : ${f:h(wizardForm.field1)}
            </div>
            <div>
                Field2 : ${f:h(wizardForm.field2)}
            </div>
            <div>
                Field3 : ${f:h(wizardForm.field3)}
            </div>
            <div>
                <form:button name="redoForm3">Back</form:button>
                <form:button>OK</form:button>
            </div>
        </form:form>
    </body>
    </html>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (33)
      - | フォームオブジェクトをセッションに格納しているため、入力画面のフィールドを、hidden項目にする必要はない。

- 完了画面(JSP)

 .. code-block:: jsp

    <html>
    <head>
    <title>Complete</title>
    </head>
    <body>
        <h1>Complete</h1>
        <div>
            <div>
                ID : ${f:h(entity.id)}
            </div>
            <div>
                Field1 : ${f:h(entity.field1)}
            </div>
            <div>
                Field2 : ${f:h(entity.field2)}
            </div>
            <div>
                Field3 : ${f:h(entity.field3)}
            </div>
        </div>
        <div>
            <a href="${pageContext.request.contextPath}/wizard/create">
                Continue operation of Create
            </a>
        </div>
        <div>
            <a href="${pageContext.request.contextPath}/wizard/${entity.id}/update">
                Continue operation of Update
            </a>
        </div>
    </body>
    </html>

- spring-mvc.xml

 .. code-block:: xml

    <bean class="org.terasoluna.gfw.web.exception.SystemExceptionResolver">
        <property name="exceptionCodeResolver" ref="exceptionCodeResolver" />
        <!-- ... -->
        <property name="exceptionMappings">
            <map>
                <!-- ... -->
                <entry key="InvalidRequestException"
                       value="common/error/operationError" /> <!-- (34) -->
            </map>
        </property>
        <property name="statusCodes">
            <map>
                <!-- ... -->
                <entry key="common/error/operationError" value="400" /> <!-- (35) -->
            </map>
        </property>
        <!-- ... -->
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (34)
      - | 共通ライブラリから提供している\ ``SystemExceptionResolver``\ の\ ``exceptionMappings``\ に、保存処理実行時に不正なリクエストを検知したことを、通知する例外\ ``InvalidRequestException``\ の、例外ハンドリングの定義を追加する。
        | 上記例では、 例外発生時の遷移先として、\ ``/WEB-INF/views/common/error/operationError.jsp``\ を指定している。
    * - | (35)
      - | \ ``SystemExceptionResolver``\ の\ ``statusCodes`` に、\ ``HttpSessionRequiredException``\ 発生時のHTTPレスポンスコードを指定する。
        | 上記例では、 例外発生時のHTTPレスポンスコードとして、Bad Request(\ ``400``\ )を指定している。

- applicationContext.xml

 .. code-block:: xml

    <bean id="exceptionCodeResolver"
        class="org.terasoluna.gfw.common.exception.SimpleMappingExceptionCodeResolver">
        <!-- Setting and Customization by project. -->
        <property name="exceptionMappings">
            <map>
                <!-- ... -->
                <entry key="InvalidRequestException" value="w.xx.0004" /> <!-- (36) -->
            </map>
        </property>
        <property name="defaultExceptionCode" value="e.xx.0001" /> <!-- (37) -->
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (36)
      - | 共通ライブラリから提供している\ ``SimpleMappingExceptionCodeResolver``\ の\ ``exceptionMappings``\ に、\ ``InvalidRequestException``\ の例外ハンドリングの定義を追加する。
        | 上記例では、  例外発生時の例外コードとして、\ ``"w.xx.0004"``\ を指定している。
        | この設定を追加しない場合は、デフォルトの例外コードが、ログに出力される。
    * - | (37)
      - | 例外発生時のデフォルトの例外コード。


.. _session-management_appendix_sessionscope:

sessionスコープのBeanを使った複数のControllerを跨いだ画面遷移の実装例
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
複数のControllerをまたいで画面遷移を行う処理を例に、sessionスコープのBeanを使った実装の説明を行う。

処理の仕様は、以下の通りとする。

* 商品をカートに追加する処理を提供する。
* カートに追加されている商品の、数量変更を行う処理を提供する。
* カートに格納されている商品を、注文する処理を提供する。
* 上記3つの処理は、それぞれ独立した機能として提供するため、別Controller(ItemController, CartController, OrderController)とする。
* カートは、上記3つの処理で共有するため、セッションに格納する。
* 商品をカートに追加した場合は、カート画面に遷移する。

画面遷移は、以下の通りとする。

 .. figure:: ./images/session-management_appendix_screenflow2.png
   :alt: Invalidate session by processing of Web Application
   :width: 100%
   :align: center

実装例は、以下の通りである。

- sessionスコープのBeanとして定義するJavaBean

 .. code-block:: java

    @Component
    @Scope(value = "session", proxyMode = ScopedProxyMode.TARGET_CLASS)
    public class SessionCart implements Serializable {

        private static final long serialVersionUID = 1L;

        private Cart cart; // (1)

        public Cart getCart() {
            if (cart == null) {
                cart = new Cart();
            }
            return cart;
        }

        public void setCart(Cart cart) {
            this.cart = cart;
        }

        public void clearCart() { // (2)
            cart.clearCart();
        }
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (1)
      - | \ ``Cart``\ というEntity(Domainオブジェクト)をラップしている。
    * - | (2)
      - | カートに追加された商品のオブジェクトを\ ``cart``\から削除し，カートが空の状態にする。

- ItemController

 .. code-block:: java

    @Controller
    @RequestMapping("item")
    public class ItemController {

        @Inject
        SessionCart sessionCart;

        @Inject
        CartService cartService;

        @Inject
        Mapper beanMapper;

        @ModelAttribute
        public ItemForm setUpItemForm() {
            return new ItemForm();
        }

        // (3)
        @RequestMapping
        public String view(Model model) {
            return "item/item";
        }

        // (4)
        @RequestMapping(value = "add")
        public String addCart(@Validated ItemForm form, BindingResult result) {
            if (result.hasErrors()) {
                return "item/item";
            }
            CartItem cartItem = beanMapper.map(form, CartItem.class);
            Cart cart = cartService.addCartItem(sessionCart.getCart(), // (5)
                    cartItem);
            sessionCart.setCart(cart); // (6)
            return "redirect:/cart"; // (7)
        }
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (3)
      - | 商品画面を、表示するためのハンドラメソッド。
    * - | (4)
      - | 指定された商品を、カートに追加するためのハンドラメソッド。
    * - | (5)
      - | セッションに格納されている\ ``Cart``\ オブジェクトを、Serviceのメソッドに渡す。
    * - | (6)
      - | Serviceのメソッドから返却された\ ``Cart``\ オブジェクトを、sessionスコープのBeanに反映する。
        | sessionスコープのBeanに反映することで、\ ``Cart``\ オブジェクトがセッションに格納される。
    * - | (7)
      - | 商品をカートに追加した後に、カート画面を表示するためのリクエストに、リダイレクトする。
        | **別Controllerの画面に遷移する場合は、直接View(JSP)を呼び出すのではなく、画面を表示するためのリクエストにリダイレクトすることを推奨する。**

- CartController

 .. code-block:: java

    @Controller
    @RequestMapping("cart")
    public class CartController {

        @Inject
        SessionCart sessionCart;

        @Inject
        CartService cartService;

        @Inject
        Mapper beanMapper;

        @ModelAttribute
        public CartForm setUpCartForm() {
            return new CartForm();
        }

        // (8)
        @RequestMapping
        public String cart(CartForm form) {
            beanMapper.map(sessionCart.getCart(), form);
            return "cart/cart";
        }

        // (9)
        @RequestMapping(params = "edit", method = RequestMethod.POST)
        public String edit(@Validated CartForm form, BindingResult result,
                Model model) {
            if (result.hasErrors()) {
                return "cart/cart";
            }

            Cart cart = sessionCart.getCart();
            Iterator<CartItemForm> itemForm = form.getCartItems().iterator();
            for (CartItem item : cart.getCartItems()) {
                beanMapper.map(itemForm.next(), item);
            }

            cart = cartService.saveCart(cart);
            sessionCart.setCart(cart); // (10)

            return "redirect:/cart"; // (11)
        }


    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (8)
      - | カート画面(数量変更画面)を表示するためのハンドラメソッド。
    * - | (9)
      - | 数量変更を、行うためのハンドラメソッド。
    * - | (10)
      - | Serviceのメソッドから返却された\ ``Cart``\ オブジェクトをsessionスコープのBeanに反映する。
        | sessionスコープのBeanに反映することで、セッションに反映される。
    * - | (11)
      - | 数量変更を行った後に、カート画面(数量変更画面)を表示するためのリクエストに、リダイレクトする。
        | **更新処理を行った場合は、直接View(JSP)を呼び出すのではなく、画面を表示するためのリクエストにリダイレクトすることを推奨する。**

- OrderController

 .. code-block:: java

    @Controller
    @RequestMapping("order")
    public class OrderController {

        @Inject
        SessionCart sessionCart;

        @ModelAttribute
        public OrderForm setUpOrderForm() {
            return new OrderForm();
        }

        // (12)
        @RequestMapping
        public String view() {
            return "order/order";
        }

        // (13)
        @RequestMapping(method = RequestMethod.POST)
        public String order() {
            // ...
            return "redirect:/order?complete";
        }

        // (14)
        @RequestMapping(params = "complete", method = RequestMethod.GET)
        public String complete(Model model, SessionStatus sessionStatus) {
            sessionCart.clearCart();
            return "order/complete";
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (12)
      - | 注文画面を、表示するためのハンドラメソッド。
    * - | (13)
      - | 注文処理を行うためのハンドラメソッド。
    * - | (14)
      - | 注文完了画面を表示するためのハンドラメソッド。

- 商品画面(JSP)

 .. code-block:: jsp

    <html>
    <head>
    <title>Item</title>
    </head>
    <body>
        <h1>Item</h1>
        <form:form action="${pageContext.request.contextPath}/item/add" 
            modelAttribute="itemForm">
            <form:label path="itemCode">Item Code</form:label> : 
            <form:input path="itemCode" />
            <form:errors path="itemCode" />
            <br>
            <form:label path="quantity">Quantity</form:label> : 
            <form:input path="quantity" />
            <form:errors path="quantity" />
            <div>
                <%-- (15) --%>
                <form:button>Add</form:button>
            </div>
        </form:form>
        <div>
            <a href="${pageContext.request.contextPath}/cart">Go to Cart</a>
        </div>
    </body>
    </html>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (15)
      - | 商品を追加するためのボタン。

- カート画面(JSP)

 .. code-block:: jsp

    <html>
    <head>
    <title>Cart</title>
    </head>
    <body>
        <%-- (16) --%>
        <spring:eval var="cart" experssion="@sessionCart.cart" />
        <h1>Cart</h1>
        <c:choose>
            <c:when test="${ empty cart.cartItems }">
                <div>Cart is empty.</div>
            </c:when>
            <c:otherwise>
                CART ID :
                ${f:h(cart.id)}
                <form:form modelAttribute="cartForm">
                    <table border="1">
                        <thead>
                            <tr>
                                <th>ID</th>
                                <th>ITEM CODE</th>
                                <th>QUANTITY</th>
                            </tr>
                        </thead>
                        <tbody>
                            <c:forEach var="item" 
                                items="${cart.cartItems}" 
                                varStatus="rowStatus">
                                <tr>
                                    <td>${f:h(item.id)}</td>
                                    <td>${f:h(item.itemCode)}</td>
                                    <td>
                                        <form:input 
                                            path="cartItems[${rowStatus.index}].quantity" />
                                        <form:errors 
                                            path="cartItems[${rowStatus.index}].quantity" />
                                    </td>
                                </tr>
                            </c:forEach>
                        </tbody>
                    </table>
                    <%-- (17) --%>
                    <form:button name="edit">Save</form:button>
                </form:form>
            </c:otherwise>
        </c:choose>
        <c:if test="${ not empty cart.cartItems }">
            <div>
                <%-- (18) --%>
                <a href="${pageContext.request.contextPath}/order">Go to Order</a>
            </div>
        </c:if>
        <div>
            <a href="${pageContext.request.contextPath}/item">Back to Item</a>
        </div>
    </body>
    </html>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (16)
      - | SpEL式を用いてsessionスコープのBeanを参照する。
    * - | (17)
      - | 数量を更新するためのボタン。
    * - | (18)
      - | 注文画面を表示するためのリンク。

- 注文画面(JSP)

 .. code-block:: jsp

    <html>
    <head>
    <title>Order</title>
    </head>
    <body>
        <spring:eval var="cart" experssion="@sessionCart.cart" />
        <h1>Order</h1>
        <table border="1">
            <thead>
                <tr>
                    <th>ID</th>
                    <th>ITEM CODE</th>
                    <th>QUANTITY</th>
                </tr>
            </thead>
            <tbody>
                <c:forEach var="item" items="${cart.cartItems}" 
                    varStatus="rowStatus">
                    <tr>
                        <td>${f:h(item.id)}</td>
                        <td>${f:h(item.itemCode)}</td>
                        <td>${f:h(item.quantity)}</td>
                    </tr>
                </c:forEach>
            </tbody>
        </table>
        <form:form modelAttribute="orderForm">
            <%-- (19) --%>
            <form:button>Order</form:button>
        </form:form>
        <div>
            <a href="${pageContext.request.contextPath}/cart">Back to Cart</a>
        </div>
        <div>
            <a href="${pageContext.request.contextPath}/item">Back to Item</a>
        </div>
    </body>
    </html>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - 項番
      - 説明
    * - | (19)
      - | 注文するためのボタン。

- 注文完了画面(JSP)

 .. code-block:: jsp

    <html>
    <head>
    <title>Order Complete</title>
    </head>
    <body>
        <h1>Order Complete</h1>
        ORDER ID :
        ${f:h(order.id)}
        <table border="1">
            <thead>
                <tr>
                    <th>ID</th>
                    <th>ITEM CODE</th>
                    <th>QUANTITY</th>
                </tr>
            </thead>
            <tbody>
                <c:forEach var="item" items="${order.orderItems}" 
                    varStatus="rowStatus">
                    <tr>
                        <td>${f:h(item.id)}</td>
                        <td>${f:h(item.itemCode)}</td>
                        <td>${f:h(item.quantity)}</td>
                    </tr>
                </c:forEach>
            </tbody>
        </table>
        <br>
        <div>
            <a href="${pageContext.request.contextPath}/item">Back to Item</a>
        </div>
    </body>
    </html>

\

.. raw:: latex

   \newpage

