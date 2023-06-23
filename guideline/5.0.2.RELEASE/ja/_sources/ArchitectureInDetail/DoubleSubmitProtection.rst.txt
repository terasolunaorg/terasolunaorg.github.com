二重送信防止
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 4
    :local:

Overview
--------------------------------------------------------------------------------

Problems
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

画面を提供するWebアプリケーションでは、以下の操作が行われると、同じ処理が複数回実行されてしまうことがある。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 30 60

   * - 項番
     - 操作
     - 操作概要
   * - | (1)
     - | 更新系ボタンの二重クリック
     - | 更新処理を行うボタンを連続してクリックする。
   * - | (2)
     - | 更新処理完了後の画面の再読み込み
     - | ブラウザの更新ボタンを使用することで、更新処理完了後の画面の再読み込みを行う。
   * - | (3)
     - | ブラウザの戻るボタンを使用した不正な画面遷移
     - | 更新処理の完了画面からブラウザの戻るボタンを使用してページを戻し、更新処理を行うボタンを再度クリックする。

それぞれ具体的な問題点を、以下に示す。

更新系ボタンの二重クリック
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 更新処理を行うボタンを連続してクリックすると、以下のような問題が発生する。
| 以下では、ショッピングサイトの商品購入を例として、対策を行わない場合にどのような問題が発生するのかを説明する。

 .. figure:: ./images/duplicate-double-click.png
   :alt: duplicate double click
   :width: 100%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 購買者が、商品購入画面で注文ボタンをクリックする。
   * - | (2)
     - | (1)のレスポンスが返る前に、購買者が誤って注文ボタンをもう一度クリックする。
   * - | (3)
     - | サーバは、(1)のリクエストで受けた商品の購入処理をDBに対して反映する。
   * - | (4)
     - | サーバは、(2)のリクエストで受けた商品の購入処理をDBに対して反映する。
   * - | (5)
     - | サーバは、(2)のリクエストで受けた商品の購入完了画面を応答する。

 .. warning::

    上記のケースでは、購入者が誤って注文ボタンを押下することで、**まったく同じ商品の購入が２回行われてしまうことになる。**
    購入者の操作ミスが原因ではあるが、アプリケーションとして上記の問題が発生しないように制御する事が望ましい。

更新処理完了後の画面の再読み込み
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 更新処理完了後の画面の再読み込みを行うと、以下のような問題が発生する。
| 以下では、ショッピングサイトの商品購入を例として、対策を行わない場合にどのような問題が発生するのかを説明する。

 .. figure:: ./images/duplicate-reload.png
   :alt: duplicate reload
   :width: 100%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 購買者が、商品購入画面で注文ボタンをクリックする。
   * - | (2)
     - | サーバは、(1)のリクエストで受けた商品の購入処理をDBに対して反映する。
   * - | (3)
     - | サーバは、(1)のリクエストで受けた商品の購入完了画面を応答する。
   * - | (4)
     - | 購買者が、誤ってブラウザのリロード機能を実行する。
   * - | (5)
     - | サーバは、(4)のリクエストで受けた商品の購入処理をDBに対して反映する。
   * - | (6)
     - | サーバは、(4)のリクエストで受けた商品の購入完了画面を応答する。

 .. warning::

    上記のケースでは、購入者が誤ってブラウザのリロード機能を実行することで、**まったく同じ商品の購入が２回行われてしまうことになる。**
    購入者の操作ミスが原因ではあるが、アプリケーションとして上記の問題が発生しないように制御する事が望ましい。

ブラウザの戻るボタンを使用した不正な画面遷移
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| ブラウザの戻るボタンを使用した不正な画面遷移を行うと、以下のような問題が発生する。
| 以下では、ショッピングサイトの商品購入を例として、対策を行わない場合にどのような問題が発生するのかを説明する。

 .. figure:: ./images/duplicate-invalid-screenflow.png
   :alt: duplicate invalid screen flow
   :width: 100%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 購買者が、商品購入画面で注文ボタンをクリックする。
   * - | (2)
     - | サーバは、(1)のリクエストで受けた商品の購入処理をDBに対して反映する。
   * - | (3)
     - | サーバは、(1)のリクエストで受けた商品の購入完了画面を応答する。
   * - | (4)
     - | 購買者が、ブラウザの戻るボタンを使って購入画面を再度表示する。
   * - | (5)
     - | 購買者が、ブラウザの戻るボタンを使って表示した購入画面で注文ボタンを再度クリックする。
   * - | (6)
     - | サーバは、(5)のリクエストで受けた商品の購入処理をDBに対して反映する。
   * - | (7)
     - | サーバは、(5)のリクエストで受けた商品の購入完了画面を応答する。

 .. note::
 
    上記のケースでは、購入者の操作ミスではないため、購入者に対して問題が発生することはない。

|

ただし、不正な画面操作を行った後でも更新処理が実行できてしまうと、以下のような問題が発生する。

 .. figure:: ./images/duplicate-allow-malicious-request.png
    :alt: duplicate allow a malicious request
    :width: 100%

 .. warning::

    上記のケースのように、不正な画面操作を行った後でも更新処理が実行できてしまうと、悪意のある攻撃者によって、正規のルート経由せずに直接更新処理が実行される危険度が高まる。
    
        .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
        .. list-table::
           :header-rows: 1
           :widths: 10 90

           * - 項番
             - 説明
           * - | (1)
             - | 攻撃者が、正規の画面遷移を行わずに、直接商品の購入を行う処理に対してリクエストを実行する。
           * - | (2)
             - | サーバは、不正なルートでリクエストが行われていることを検知することができないため、リクエストで受けた商品の購入処理をDBに対して反映してしまう。

    不正なリクエストによって購入処理を実行することで、各サーバの負荷が高くなったり、正規のルートで商品が購入できなくなるなどの問題が発生してしまう。
    結果的に、正規のルートで購入している利用者に対して問題が波及する事になるため、アプリケーションとして上記の問題が発生しないように制御する事が望ましい。

Solutions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| 上記の問題を解決する方法として、下記の対策が必要になる。
| リクエストの改竄など悪意あるオペレーションを考慮すると、 **(3)の「トランザクショントークンチェックの適用」は必須である。**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 20 70

   * - 項番
     - Solution
     - 概要
   * - | (1)
     - | JavaScriptによるボタンの2度押し防止
     - | 更新処理を行うボタンを押下した際に、JavaScriptによるボタン制御を行うことで、2度押しされた際にリクエストが送信されないようにする。
   * - | (2)
     - | PRG(Post-Redirect-Get)パターンの適用
     - | 更新処理を行うリクエスト(POSTメソッドによるリクエスト)に対する応答としてリダイレクトを返却し、その後ブラウザから自動的にリクエストされるGETメソッドの応答として遷移先の画面を返却するようにする。
       | PRGパターンを適用することで、画面表示後にページの再読み込みを行った場合に発生するリクエストがGETメソッドになるため、更新処理の再実行を防ぐことが出来る。
   * - | (3)
     - | トランザクショントークンチェックの適用
     - | 画面遷移毎にトークン値を払い出し、ブラウザから送信されたトークン値とサーバ上で保持しているトークン値を比較することで、トランザクション内で不正な画面操作が行われないようにする。
       | トランザクショントークンチェックを適用することで、ブラウザの戻るボタンを使ってページを移動した後の更新処理の再実行を防ぐことが出来る。
       | また、トークン値のチェックを行った後にサーバで管理しているトークン値を破棄することで、サーバ側の処理として二重送信を防ぐことも出来る。

 .. note::

    「トランザクショントークンチェックの適用」のみの対策だと、単純な操作ミスを行った場合でもトランザクショントークンエラーとなるため、利用者に対してユーザビリティの低いアプリケーションになってしまう。
    
    ユーザビリティを確保しつつ、二重送信で発生する問題を防止するためには、「JavaScriptによるボタンの2度押し防止」及び「PRG(Post-Redirect-Get)パターンの適用」が必要となる。
    
    **本ガイドラインでは、全ての対策を行うことを推奨するが、アプリケーションの要件によって対策の有無は判断すること。**

 .. Warning::

   AjaxとWebサービスでは、リクエスト毎に変更されるトランザクショントークンの受け渡しを行いにくいため、トランザクショントークンチェックを使用しなくてよい。
   Ajaxの場合は、JavaScriptによるボタンの2度押し防止のみで二重送信防止を行う。

 .. todo::
 
    **TBD**

    AjaxとWebサービスでのチェック方法は、今後検討の余地あり。


JavaScriptによるボタンの2度押し防止について
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 更新処理を行うボタンや、時間のかかる検索処理を行うボタンなどに対して、ボタンの二重クリックを防止する。
| ボタンが押された際に、JavaScriptを使用してボタンやリンクの無効化の制御を行う。
| 無効化するための代表的な制御例としては、

#. ボタンやリンクを非活性化することで、ボタンやリンクを押下できないように制御する。
#. 処理状態をフラグとして保持しておき、処理中にボタンやリンクが押された場合に処理中であることを通知するメッセージを表示する。

| などがあげられる。

下記は、ボタンを非活性化した際のイメージとなる。

 .. figure:: ./images/prevent-double-click.png
   :alt: prevent double click
   :width: 60%

 .. warning::
 
    画面上に存在する全てのボタン及びリンクを無効化してしまうと、サーバからの応答がない場合に、画面操作が行えなくなってしまう。
    そのため、「前画面に戻る」や「トップ画面へ移動」などのイベントを実行するボタンやリンクは無効化しないようにすることを推奨する。

.. _DoubleSubmitProtectionAboutPRG:

PRG(Post-Redirect-Get)パターンについて
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 更新処理を行うリクエスト(POSTメソッドによるリクエスト)に対する応答としてリダイレクトを返却し、その後ブラウザから自動的にリクエストされるGETメソッドの応答として遷移先の画面を返却するようにする。
| PRGパターンを適用することで、画面表示後にページの再読み込みを行った場合に発生するリクエストがGETメソッドになるため、更新処理の再実行を防ぐことが出来る。

 .. figure:: ./images/prevent-double-submit-reload.png
   :alt: prevent double submit by reload
   :width: 100%


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 購買者が、商品購入画面で注文ボタンをクリックする。
       | **リクエストは、POSTメソッドを使って送信される。**
   * - | (2)
     - | サーバは、(1)のリクエストで受けた商品の購入処理をDBに対して反映する。
   * - | (3)
     - | **サーバは、商品の購入完了画面を表示するためのURLに対するリダイレクト応答を行う。**
   * - | (4)
     - | ブラウザは、商品の購入完了画面を表示するためのURLにリクエストを送信する。
       | **リクエストは、GETメソッドを使って送信される。**
   * - | (5)
     - | サーバは、商品の購入完了画面を応答する。
   * - | (6)
     - | 購買者が、誤ってブラウザのリロード機能を実行する。
       | リロード機能によって要求されるリクエストは、商品の購入完了画面を表示するためのリクエストとなるため、 **更新処理が再実行されることはない。**
   * - | (7)
     - | サーバは、商品の購入完了画面を応答する。

 .. note::
 
    更新処理を伴う処理の場合は、\ :abbr:`PRG (Post-Redirect-Get)`\ パターンを適用し、ブラウザの更新ボタンが押された際に、GETメソッドのリクエストが送信されるように制御することを推奨する。

 .. warning::
 
    \ :abbr:`PRG (Post-Redirect-Get)`\ パターンでは、完了画面でブラウザの戻るボタンを押下することで、更新処理を再度実行されることを防ぐことはできない。
    ブラウザの戻るボタンを使った不正な画面遷移後の更新処理の再実行を防ぐ場合は、トランザクショントークンチェックを行う必要がある。
    
.. _double-submit_transactiontokencheck:

トランザクショントークンチェックについて
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

トランザクショントークンチェックは、

* サーバは、クライアントからリクエストが来た際に、サーバ上にトランザクションを一意に識別するための値（以下、トランザクショントークン）を保持する。
* サーバは、クライアントへトランザクショントークンを引き渡す。画面を提供するWebアプリケーションの場合は、formのhiddenタグを使用してクライアントにトランザクショントークンを引き渡す。
* クライアントは次のリクエストを送信する際に、サーバから引き渡されたトランザクショントークンを送る。サーバは、クライアントから受け取ったトランザクショントークンと、サーバ上で管理しているトランザクショントークンを比較する。

という、３つの処理で構成され、リクエストで送信されてきたトランザクショントークン値と、サーバ上で保持しているトランザクショントークン値が一致していない場合は、不正なリクエストとみなしてエラーを返す。

 .. warning::
 
    トランザクショントークンチェックの濫用は、アプリケーションのユーザビリティ低下につながるため、以下の点を考慮して、適用範囲を決めること。

    * | データの更新を伴わない参照系のリクエストや、単に画面遷移のみ行うリクエストについては、トランザクショントークンチェックの範囲に含める必要はない。
      | 必要以上にトランザクションの範囲を広げてしまうと、トランザクショントークンエラーが発生しやすくなるため、アプリケーションのユーザビリティを低下させる事になる。
    * | ビジネス観点で何回更新されても問題ないような処理（ユーザー情報更新など）では、トランザクショントークンチェックは必須ではない。
    * | 入金処理や商品の購入処理など、処理が二重で実行されると問題がある場合は、トランザクショントークンチェックが必須である。

|

以下に、トランザクショントークンチェック使用時において、想定通りの操作を行った場合の処理フローと、想定外の操作を行った場合の処理フローについて説明する。

 .. figure:: ./images/transaction-token-check-overview.png
   :alt: transaction token overview
   :width: 100%

| 想定通りの操作を行った場合の処理フローについて説明する。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | クライアントから、リクエストを送信する。
   * - | (2)
     - | サーバは、トランザクショントークン(token001)を作成し、サーバ上で保持する。
   * - | (3)
     - | サーバは、作成したトランザクショントークン(token001)を、クライアントに引き渡す。
   * - | (4)
     - | クライアントから、トランザクショントークン(token001)を含めたリクエストを送信する。
   * - | (5)
     - | サーバは、サーバ上で保持しているトランザクショントークン(token001)と、クライアントから送信されたトランザクショントークン(token001)が同一かチェックする。
       | **値が同一なので、正規のリクエストと判断される。**
   * - | (6)
     - | サーバは、次のリクエストで使用するトランザクショントークン(token002)を生成し、サーバ上で管理している値を更新する。
       | この時点で、トランザクショントークン(token001)は破棄される。
   * - | (7)
     - | サーバは、更新したトランザクショントークン(token002)を、クライアントに引き渡す。

| 想定外の操作を行った場合の処理フローについて説明する。
| ここではブラウザの戻るボタンを例にしているが、ショートカットからの直接リクエストなどでも同様である。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (8)
     - | クライアントでブラウザの戻るボタンをクリックする。
   * - | (9)
     - | クライアントから戻った画面にあるトランザクショントークン(token001)を含めたリクエストを送信する。
   * - | (10)
     - | サーバは、サーバ上に保持しているトランザクショントークン(token002)と、クライアントから送信されたトランザクショントークン(token001)が同一かチェックする。
       | **値が同一ではないので、 不正なリクエストと判断し、トランザクショントークンエラーとする。**
   * - | (11)
     - | サーバは、トランザクショントークンエラーが発生した事を通知するエラー画面を応答する。

|

トランザクショントークンチェックで防ぐことが出来るのは、以下の3つの事象である。

* 決められた画面遷移を行うことが求められる業務において、不正な画面遷移が行われる。
* 正規の画面遷移を伴わない不正なリクエストによって、データが更新される。
* 二重送信によって、更新処理が重複して実行される。

|

以下のフローによって、決められた画面遷移を行うことが求められる業務において、不正な画面遷移が行われる事を防ぐ事ができる。

 .. figure:: ./images/transaction-token-check-prevent-invalid-screenflow.png
   :alt: prevent invalid screen flow by transaction token check
   :width: 100%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 購買者が、商品購入画面で注文ボタンをクリックする。
       | サーバ上で保持しているトランザクショントークンと、クライアントから送信されたトランザクショントークンが一致するため、商品を購入する処理を実行する。
       | **このタイミングで、サーバ上で保持していたトランザクショントークの値が破棄され、新しいトークン値に更新される。**
   * - | (2)
     - | サーバは、(1)のリクエストで受けた商品の購入処理をDBに対して反映する。
   * - | (3)
     - | サーバは、(1)のリクエストで受けた商品の購入完了画面を応答する。
   * - | (4)
     - | 購買者が、ブラウザの戻るボタンを使って購入画面を再度表示する。
   * - | (5)
     - | 購買者が、ブラウザの戻るボタンを使って表示した購入画面で注文ボタンを再度クリックする。
       | **クライアントから送信されたトランザクショントークンは既に破棄された値のため、トランザクショントークンエラーとなる。**
   * - | (6)
     - | サーバは、トランザクショントークンエラーが発生した事を通知するエラー画面を応答する。

|

以下のフローによって、正規の画面遷移を伴わない不正なリクエストでデータが更新される事を防ぐことができる。

 .. figure:: ./images/transaction-token-check-prevent-malicious-request.png
   :alt: prevent malicious request by transaction token check
   :width: 100%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 攻撃者が、正規の画面遷移を行わずに、直接商品の購入を行う処理に対してリクエストを実行する。
       | **トランザクショントークンを生成するためのリクエストを実行していないため、トランザクショントークンエラーとなる。**
   * - | (2)
     - | サーバは、トランザクショントークンエラーが発生した事を通知するエラー画面を応答する。

|

以下のフローによって、二重送信発生時に更新処理が重複して実行される事を防ぐことができる。

 .. figure:: ./images/transaction-token-check-prevent-double-submit.png
   :alt: prevent double submit by transaction token check
   :width: 100%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 購買者が、商品購入画面で注文ボタンをクリックする。
       | サーバ上で保持しているトランザクショントークンと、クライアントから送信されたトランザクショントークンが一致するため、商品を購入する処理を実行する。
       | **このタイミングで、サーバ上で保持していたトランザクショントークの値が破棄され、新しいトークン値に更新される。**
   * - | (2)
     - | (1)のレスポンスが返る前に、購買者が誤って注文ボタンをもう一度クリックする。
       | (1)の処理が実行されることによって、 **クライアントから送信されたトランザクショントークンは既に破棄された値のため、トランザクショントークンエラーとなる。**
   * - | (3)
     - | サーバは、(2)のリクエストに対して、 **トランザクショントークンエラーが発生した事を通知するエラー画面を応答する。**
   * - | (4)
     - | サーバは、(1)のリクエストで受けた商品の購入処理をDBに対して反映する。
   * - | (5)
     - | サーバは、(1)のリクエストで受けた商品の購入完了画面を応答しようとするが、(2)のリクエストが送信された事により、(1)のリクエストに対する応答を行うためのストリームが閉じられているため、購入完了画面を応答することができない。

 .. warning::
 
    二重送信発生時に更新処理が重複して実行される事は防ぐことが出来るが、処理が完了した事を通知する画面を応答することが出来ないという問題が残る。
    そのため、JavaScriptによるボタンの2度押し防止も合わせて対応することを推奨する。

トランザクショントークンのネームスペースについて
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
共通ライブラリから提供しているトランザクショントークンチェック機能では、トランザクショントークンを管理するための器にネームスペースを設けることが出来る。
これは、タブブラウザや複数ウィンドウを使用して、更新処理を並行して操作できるようにするための仕組みである。

ネームスペースがない場合の問題点について
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| まず、ネームスペースがない場合の問題点について説明する。
| 以下の図では、clientが左右にわかれているが、実際は同一マシン上に２つのWindowを立ち上げた際の例となる。

 .. figure:: ./images/token-only-one.png
   :alt: token only one
   :width: 100%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | Window1からリクエストを送信し、応答されたトランザクショントークン(token001)をブラウザに保持する。
       | サーバ上で保持しているトランザクショントークンはtoken001となる。
   * - | (2)
     - | Window2からリクエストを送信し、応答されたトランザクショントークン(token002)をブラウザに保持する。
       | **サーバ上で保持しているトランザクショントークンはtoken002となる。このタイミングで(1)の処理で生成されたトランザクショントークン(token001)は破棄される。**
   * - | (3)
     - | Window1からブラウザで保持しているトランザクショントークン(token001)を含めてリクエストを送信する。
       | サーバ上で保持しているトランザクショントークン(token002)と、リクエストで送信されたトランザクショントークン(token002)が一致しないため、不正なリクエストと判断され、トランザクショントークンエラーとなる。

 .. warning::
 
    **ネームスペースがない場合は、更新処理を並行して操作することができないため、ユーザビリティの低いアプリケーションとなってしまう。**

|

ネームスペース指定時の動作について
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 次に、ネームスペースを付与した際の動作について説明する。
| ネームスペースがない場合は、更新処理を並行して操作することができないという問題があったが、ネームスペースも設けることで、この問題を解決することが出来る。
| 以下の図では、clientが左右にわかれているが、実際は同一マシン上に２つのWindowを立ち上げた際の例となる。

 .. figure:: ./images/token-namespace.png
   :alt: token namespace
   :width: 100%

| 上記の図の、111, 222の部分が、ネームスペースとなる。
| **ネームスペースを与えることで、トランザクションに割り振られたネームスペース内に存在するトランザクショントークンのみを操作するため、別のネームスペースのトランザクションに対して影響を与えない。**
| ここでは、ブラウザを別のWindowで記述しているが、タブブラウザでも同じである。生成されるキーや使用方法については、\ :ref:`doubleSubmit_how_to_use_transaction_token_check`\ で説明する。

|

.. _How-to-use:

How to use
--------------------------------------------------------------------------------

JavaScriptによるボタンの2度押し防止の適用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| クライアントでのボタンの二重クリック防止は、JavaScriptで実現することになる。
| ボタンをクリックした後は、再描画するまでクリックできないようにする。

 .. todo::
 
    **TBD**
    
    JavaScriptでのチェック方法については、次版以降で詳細化する予定である。

PRG(Post-Redirect-Get)パターンの適用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| PRG(Post-Redirect-Get)パターンを適用する場合の実装例について説明する。
| 以降では、入力画面 -> 確認画面 -> 完了画面 というシンプルな画面遷移を行うアプリケーションを例に説明する。

 .. figure:: ./images/staff-redirect-flow.png
   :alt: STAFF REDIRECT FLOW
   :width: 100%

| 画像の番号と、ソースのコメント番号を連動させている。
| ただし、(1)～(4)については、PRGパターンと直接関係ないため、説明は省略する。

- Controller

 .. code-block:: java
    :emphasize-lines: 35,36,47-49,52-54,56

    @Controller
    @RequestMapping("prgExample")
    public class PostRedirectGetExampleController {

        @Inject
        UserService userService;

        @ModelAttribute
        public PostRedirectGetForm setUpForm() {
            PostRedirectGetForm form = new PostRedirectGetForm();
            return form;
        }

        @RequestMapping(value = "create", 
                        method = RequestMethod.GET, 
                        params = "form") // (1)
        public String createForm(
            PostRedirectGetForm postRedirectGetForm,
            BindingResult bindingResult) {
            return "prg/createForm"; // (2)
        }

        @RequestMapping(value = "create", 
                        method = RequestMethod.POST, 
                        params = "confirm") // (3)
        public String createConfirm(
            @Validated PostRedirectGetForm postRedirectGetForm,
            BindingResult bindingResult) {
            if (bindingResult.hasErrors()) {
                return "prg/createForm";
            }
            return "prg/createConfirm"; //  (4)
        }

        @RequestMapping(value = "create", 
                        method = RequestMethod.POST) // (5)
        public String create(
            @Validated PostRedirectGetForm postRedirectGetForm,
            BindingResult bindingResult,
            RedirectAttributes redirectAttributes) {
            if (bindingResult.hasErrors()) {
                return "prg/createForm";
            }

            // omitted

            String output = "result register..."; // (6)
            redirectAttributes.addFlashAttribute("output", output); // (6)
            return "redirect:/prgExample/create?complete"; // (6)
        }

        @RequestMapping(value = "create", 
                        method = RequestMethod.GET, 
                        params = "complete") // (7)
        public String createComplete() {
            return "prg/createComplete"; // (8)
        }
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (5)
     - | 確認画面の登録ボタン(Create Userボタン)が押下時の処理を行う処理メソッド。
       | **POSTメソッドでリクエストを受け取る。**
   * - | (6)
     - | **完了画面を表示するためのURLへリダイレクトする。**
       | 上記例では、\ ``"prgExample/create?complete"``\ というURLに対して\ ``GET``\メソッドで リクエストされる。
       | リダイレクト先にデータを引き渡す場合は、 \ ``RedirectAttributes``\のaddFlashAttributeメソッドを呼び出し、引き渡すデータを追加する。
       | \ ``Model``\ のaddAttributeメソッドは、リダイレクト先にデータを引き渡すことはできない。
   * - | (7)
     - | 完了画面を表示するための処理メソッド。
       | **GETメソッドでリクエストを受け取る。**
   * - | (8)
     - | 完了画面を表示するView(JSP)を呼び出し、完了画面を応答する。
       | JSPの拡張子は :file:`spring-mvc.xml` に定義されている \ ``ViewResolver``\によって付与されるため、処理メソッドの返却値からは省略している。

 .. note::

    * リダイレクトする際は、処理メソッドの返り値として返却する遷移情報のプレフィックスとして「redirect:」を付与する。
    * リダイレクト先の処理にデータを引き渡したい場合は、\ ``RedirectAttributes``\ のaddFlashAttributeメソッドを呼び出し、引き渡すデータを追加する。

- :file:`createForm.jsp`

 .. code-block:: jsp

    <h1>Create User</h1>
    <div id="prgForm">
      <form:form 
        action="${pageContext.request.contextPath}/rpgExample/create"
        method="post" modelAttribute="postRedirectGetForm">
        <form:label path="firstName">FirstName</form:label>
        <form:input path="firstName" /><br>
        <form:label path="lastName">LastName:</form:label>
        <form:input path="lastName" /><br>
        <form:button name="confirm">Confirm Create User</form:button>
      </form:form>
    </div>

- :file:`createConfirm.jsp`

 .. code-block:: jsp
    :emphasize-lines: 5,11

    <h1>Confirm Create User</h1>
    <div id="prgForm">
      <form:form
        action="${pageContext.request.contextPath}/rpgExample/create"
        method="post"
        modelAttribute="postRedirectGetForm">
        FirstName:${f:h(postRedirectGetForm.firstName)}<br>
        <form:hidden path="firstName" />
        LastName:${f:h(postRedirectGetForm.lastName)}<br>
        <form:hidden path="lastName" />
        <form:button>Create User</form:button> <%-- (6) --%>
      </form:form>
    </div>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (6)
     - | 更新処理を行うためのボタンが押下された場合は、 **POSTメソッドでリクエスト送る。**

- :file:`createComplete.jsp`

 .. code-block:: jsp
    :emphasize-lines: 6

    <h1>Successful Create User Completion</h1>
    <div id="prgForm">
      <form:form
        action="${pageContext.request.contextPath}/rpgExample/create"
        method="get" modelAttribute="postRedirectGetForm">
        output:${f:h(output)}<br> <%-- (7) --%>
        <form:button name="backToTop">Top</form:button>
      </form:form>
    </div>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (7)
     - | リダイレクト先にて、更新処理から引き渡したデータを参照する場合は、\ ``RedirectAttributes``\ の **addFlashAttributeメソッドで追加したデータの属性名を指定する。**
       | 上記例では、 \ ``"output"``\が引き渡したデータを参照するための属性名となる。

.. _doubleSubmit_how_to_use_transaction_token_check:

トランザクショントークンチェックの適用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| トランザクショントークンチェックを適用する場合の実装例について説明する。
| トランザクショントークンチェックは、Spring MVCから提供されている機能ではなく、共通ライブラリから提供している機能となる。

共通ライブラリから提供しているトランザクショントークンチェックについて
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

共通ライブラリから提供しているトランザクショントークンチェック機能では、

* トランザクショントークンのネームスペース化
* トランザクションの開始
* トランザクション内のトークン値チェック
* トランザクションの終了

を行うために、 \ ``@org.terasoluna.gfw.web.token.transaction.TransactionTokenCheck``\アノテーションを提供している。

トランザクショントークンチェックを行う場合は、Controllerクラス及びControllerクラスの処理メソッドに対して、 \ ``@TransactionTokenCheck``\アノテーションを付与することで、
宣言的にトランザクショントークンチェックを行うことが出来る。

|

``@TransactionTokenCheck``\ アノテーションの属性について
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

``@TransactionTokenCheck``\ アノテーションに指定できる属性について説明する。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.45\linewidth}|p{0.10\linewidth}|p{0.20\linewidth}|
 .. list-table:: \ ``@TransactionTokenCheck``\ アノテーションパラメタ一覧
   :header-rows: 1
   :widths: 10 10 45 10 20

   * - 項番
     - 属性名
     - 内容
     - default
     - 例
   * - (1)
     - value
     - | 任意文字列。NameSpaceとして使用される。
     - 無
     - | value = "create"
       | 引数が1つのみの場合は、"value ="部分は省略できる。
   * - (2)
     - type
     - | **BEGIN**
       | トランザクショントークンを作成し、新たなトランザクションを開始する。
       | 
       | **IN**
       | トランザクショントークンの妥当性チェックを実施する。
       | リクエストされたトークン値とサーバ上で管理しているトークン値が一致している場合は、トランザクショントークンのトークン値を更新する。
       |
     - IN
     - | type = TransactionTokenType.BEGIN
       |
       | type = TransactionTokenType.IN
       |

 .. note::
 
    value属性に設定する値は、\ ``@RequestMapping``\ アノテーションのvalue属性の設定値と、同じ値を設定することを推奨する。

 .. note::
 
    type属性には、 **NONE** 及び **END** を指定することが出来るが、通常使用することはないため、説明は省略する。

|

トランザクショントークンの形式について
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

共通ライブラリから提供しているトランザクショントークンチェックで使用するトランザクショントークンは、以下の形式となる。

 .. figure:: ./images/transaction-token-name-pattern.png
   :alt: format of transaction token
   :width: 100%

 .. figure:: ./images/transaction-token-name-pattern-example.png
   :alt: example of transaction token
   :width: 100%

|

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 15 75

   * - 項番
     - 構成要素
     - 説明
   * - | (1)
     - NameSpace
     - * NameSpaceは、一連の画面遷移を識別するための論理的な名称を付与するための要素となる。
       * NameSpaceを設けることで、異なるNameSpaceに属するリクエストが干渉しあう事を防ぐ事が出来るため、並行して操作を行うことができる画面遷移を増やすことが出来る。
       * NameSpaceとして使用する値は、\ ``@TransactionTokenCheck``\アノテーションのvalue属性で指定した値が使用される。
       * クラスアノテーションのvalue属性とメソッドアノテーションのvalue属性の両方を指定した場合は、 両方の値を\ ``"/"``\で連結した値がNameSpaceとなる。複数のメソッドで同じ値を指定した場合は、同じNameSpaceに属するメソッドとなる。
       * クラスアノテーションにのみvalue属性を指定した場合は、そのクラスで生成されるトランザクショントークンのNameSpaceは、全てクラスアノテーションで指定した値となる。
       * メソッドアノテーションにのみvalue属性を指定した場合は、生成されるトランザクショントークンのNameSpaceはメソッドアノテーションで指定した値となる。複数のメソッドで同じ値を指定した場合は、同じNameSpaceに属するメソッドとなる。
       * クラスアノテーションのvalue属性とメソッドアノテーションのvalue属性の両方を省略した場合は、グローバルトークンに属するメソッドとなる。グローバルトークンについては、\ :ref:`doubleSubmit_appendix_global_token`\を参照されたい。
   * - | (2)
     - TokenKey
     - * TokenKeyは、ネームスペース内で管理されているトランザクションを識別するための要素となる。
       * TokenKeyは、\ ``@TransactionTokenCheck``\アノテーションのtype属性に\ ``TransactionTokenType.BEGIN``\が宣言されているメソッドが実行されたタイミングで生成れる。
       * | 複数のTokenKeyを同時に保持することが出来る数には上限数があり、デフォルト10である。TokenKeyの保持数はNameSpace毎に管理される。
       * | \ ``TransactionTokenType.BEGIN``\時にNameSpace毎に管理されている保持数が最大値に達している場合は、実行された日時が最も古いTokenKeyを破棄することで(Least Recently Used (LRU))、新しいトランザクションを有効なトランザクションとして管理する仕組みとなっている。
       * | 破棄されたトランザクショントークンを使ってアクセスした場合は、トランザクショントークンエラーとなる。
   * - | (3)
     - TokenValue
     - * TokenValueは、トランザクションのトークン値を保持するための要素となる。
       * TokenValueは、\ ``@TransactionTokenCheck``\アノテーションのtype属性に\ ``TransactionTokenType.BEGIN``\又は\ ``TransactionTokenType.IN``\が宣言されているメソッドが実行されたタイミングで生成される。

 .. warning::
 
    メソッドアノテーションにのみvalue属性を指定した場合、他のControllerで同じ値を指定している場合に、一連の画面遷移を行うためのリクエストとして扱われる点に注意する必要がある。
    この方法での指定は、Controllerを跨いだ画面遷移を同一トランザクションとして扱いたい場合にのみ、使用すること。
    
    原則的には、メソッドアノテーションにのみvalue属性を指定する方法は使用しない事を推奨する。

 .. note::
 
    NameSpaceの指定方法として、
    
    * クラスアノテーションのvalue属性とメソッドアノテーションのvalue属性の両方を指定する場合
    * クラスアノテーションにのみvalue属性を指定する場合
    
    の使い分けについては、Controllerの作成粒度に応じて使い分ける。
    
    1. | Controllerに、複数のユースケースに対応する処理メソッドを実装する場合は、クラスアノテーションのvalue属性とメソッドアノテーションのvalue属性の両方を指定する。
       | 例えば、ユーザの登録、変更、削除を一つのControllerで実装する場合は、このパターンとなる。
    2. | Controllerに、一つのユースケースに対応する処理メソッドを実装する場合は、クラスアノテーションにのみvalue属性を指定する。
       | 例えば、ユーザの登録、変更、削除毎にControllerを実装する場合は、このパターンとなる。

|

.. _LifeCycle:

トランザクショントークンのライフサイクルについて
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

トランザクショントークンのライフサイクル(生成、更新、破棄)制御は、以下のタイミングで行われる。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 20 70

   * - 項番
     - ライフサイクル制御
     - 説明
   * - | (1)
     - | トークンの生成
     - | \ ``@TransactionTokenCheck``\アノテーションのtype属性に\ ``TransactionTokenType.BEGIN``\が指定されたメソッドの処理が終了したタイミングで新たなトークンが生成され、トランザクションが開始される。
   * - | (2)
     - | トークンの更新
     - | \ ``@TransactionTokenCheck``\アノテーションのtype属性に\ ``TransactionTokenType.IN``\が指定されたメソッドの処理が終了したタイミングでトークン(TokenValue)が更新され、トランザクションが継続される。
   * - | (3)
     - | トークンの破棄
     - | 以下の何れかのタイミングで破棄され、トランザクションが終了される。
       |
       | [1]
       | \ ``@TransactionTokenCheck``\アノテーションのtype属性に\ ``TransactionTokenType.BEGIN``\が指定されたメソッドを呼び出すタイミングで、リクエストパラメータに指定されているトランザクショントークンが破棄され、不要なトランザクションが終了される。
       |
       | [2]
       | NameSpace内で保持することが出来るトランザクショントークン(TokenKey)の数が上限数に達している状態で、新たにトランザクションが開始される場合、実行された日時が最も古いトランザクショントークンが破棄され、トランザクションが強制終了される。
       |
       | [3]
       | システムエラーなどの例外が発生した場合、リクエストパラメータに指定されているトランザクショントークンが破棄され、トランザクションを終了される。

 .. note::
 
    NameSpace内で保持することが出来るトランザクショントークン(TokenKey)の数には上限数が設けられており、新たにトランザクショントークンを生成する際に
    上限値に達していた場合は、実行された日時が最も古いTokenKeyをもつトランザクショントークンを破棄(Least Recently Used (LRU))することで、
    新しいトランザクションを有効なトランザクションとして管理する仕組みとなっている。

    NameSpaceごとに保持できるトランザクショントークンの上限数のデフォルト10個である。
    上限値を変更する場合は、\ :ref:`doubleSubmit_how_to_extend_change_max_count`\を参照されたい。

|

| 以下に、新たにトランザクショントークンを生成する際に上限値に達していた場合の動作について説明する。
| 前提条件は以下の通りとする。

* NameSpace内で保持することが出来るトランザクショントークンの数には上限数は、デフォルト値(10)が指定されている。
* Controllerのクラスアノテーションとして、 \ ``@TransactionTokenCheck("name")``\が指定されている。
* 同じNameSpaceのトランザクショントークンが上限値に達している状態である。

 .. figure:: ./images/transaction-token-count.png
   :alt: transaction token count
   :width: 100%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 同じNameSpaceのトランザクショントークンが上限値に達している状態で、新たなトランザクションを開始するリクエストを受け付ける。
   * - | (2)
     - | 新たにトランザクショントークンを生成する。
   * - | (3)
     - | 生成したトランザクショントークンをトークン格納先に追加する。
       | **この時点で上限数を超えるトランザクショントークンがNameSpace内に存在する状態となる。**
   * - | (4)
     - | NameSpace内で保持することが出来るトランザクショントークンの数には上限数を超える分のトランザクショントークンを削除する。
       | **トランザクショントークンを削除する際は、実行された日時が最も古いものから順に削除する。**

|

.. _setting:

トランザクショントークンチェックを使用するための設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

共通ライブラリから提供しているトランザクショントークンチェックを使用するための設定を、以下に示す。

- :file:`spring-mvc.xml`

 .. code-block:: xml
    :emphasize-lines: 2-9,16,17

    <mvc:interceptors>
        <mvc:interceptor> <!-- (1) -->
            <mvc:mapping path="/**" /> <!-- (2) -->
            <mvc:exclude-mapping path="/resources/**" /> <!-- (2) -->
            <mvc:exclude-mapping path="/**/*.html" /> <!-- (2) -->
            <!-- (3) -->
            <bean
                class="org.terasoluna.gfw.web.token.transaction.TransactionTokenInterceptor" />
        </mvc:interceptor>
    </mvc:interceptors>

    <bean id="requestDataValueProcessor"
        class="org.terasoluna.gfw.web.mvc.support.CompositeRequestDataValueProcessor">
        <constructor-arg>
            <util:list>
                <!-- (4) -->
                <bean class="org.terasoluna.gfw.web.token.transaction.TransactionTokenRequestDataValueProcessor" />
                <!-- omitted -->
            </util:list>
        </constructor-arg>
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | トランザクショントークンの生成及びチェックを行うための \ ``HandlerInterceptor``\を設定する。
   * - | (2)
     - | \ ``HandlerInterceptor``\を適用するリクエストパスを指定する。
       | 上記例では、 /resources配下へのリクエストとHTMLへのリクエストを除く、全てのリクエストに対して適用している。
   * - | (3)
     - | \ ``@TransactionTokenCheck``\ アノテーションを使用して、トランザクショントークンの生成及びチェックを実施するためのクラス(\ ``TransactionTokenInterceptor``\)を指定する。
   * - | (4)
     - | トランザクショントークンを、Spring MVCの\ ``<fomr:form>``\タグを使用してHidden領域に自動的に埋め込むためのクラス(\ ``TransactionTokenRequestDataValueProcessor``\)を設定する。


トランザクショントークンエラーをハンドリングするための設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| トランザクショントークンエラーが発生した場合は、 \ ``org.terasoluna.gfw.web.token.transaction.InvalidTransactionTokenException`` が発生する。

| そのため、トランザクショントークンエラーをハンドリングするためには、 

* :file:`applicationContext.xml` に定義されている \ ``ExceptionCodeResolver``\
* :file:`spring-mvc.xml` に定義されている \ ``SystemExceptionResolver``\

の設定に対して、 \ ``InvalidTransactionTokenException``\のハンドリング定義を追加する必要がある。

設定の追加方法については、

* :ref:`exception-handling-how-to-use-application-configuration-common-label`
* :ref:`exception-handling-how-to-use-application-configuration-app-label`

を参照されたい。


トランザクショントークンチェックのControllerでの利用方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| トランザクショントークンチェックを行う場合、Controllerではトランザクションを開始するメソッドの定義、チェックを行うメソッドの定義が必要となる。
| 以下では、1つのcontrollerで、1つのユースケースで必要となる処理メソッドを実装する場合の説明となる。

- Controller

 .. code-block:: java
    :emphasize-lines: 3,12,18,24,30,32,36

    @Controller
    @RequestMapping("transactionTokenCheckExample")
    @TransactionTokenCheck("transactionTokenCheckExample") // (1)
    public class TransactionTokenCheckExampleController {

        @RequestMapping(params = "first", method = RequestMethod.GET)
        public String first() {
            return "transactionTokenCheckExample/firstView";
        }

        @RequestMapping(params = "second", method = RequestMethod.POST)
        @TransactionTokenCheck(type = TransactionTokenType.BEGIN) // (2)
        public String second() {
            return "transactionTokenCheckExample/secondView";
        }

        @RequestMapping(params = "third", method = RequestMethod.POST)
        @TransactionTokenCheck // (3)
        public String third() {
            return "transactionTokenCheckExample/thirdView";
        }

        @RequestMapping(params = "fourth", method = RequestMethod.POST)
        @TransactionTokenCheck // (3)
        public String fourth() {
            return "transactionTokenCheckExample/fourthView";
        }

        @RequestMapping(params = "fifth", method = RequestMethod.POST)
        @TransactionTokenCheck // (3)
        public String fifth() {
            return "redirect:/transactionTokenCheckExample?complete";
        }

        @RequestMapping(params = "complete", method = RequestMethod.GET) 
        public String complete() { // (4)
            return "transactionTokenCheckExample/fifthView";
        }

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | クラスアノテーションのvalue属性でNameSpaceを指定する。
       | 上記例では、本ガイドラインの推奨パターンである \ ``@RequestMapping``\のvalue属性と同じ値を指定している。
   * - | (2)
     - | トランザクションを開始し、新しいトランザクショントークンを払い出す。
       | ここでは、Controller単位でトランザクショントークンを管理するため、メソッドアノテーションのvalue属性を指定しない。
   * - | (3)
     - | トランザクショントークンをチェックし、トランザクショントークンのトークン値を更新する。
       | type属性を省略した場合は、\ ``@TransactionTokenCheck(type = TransactionTokenType.IN)``\を指定した時と同じ動作となる。
   * - | (4)
     - | ユースケースの完了を通知する画面を表示するためのリクエストでは、トランザクショントークンチェックを行う必要はないため\ ``@TransactionTokenCheck``\アノテーションの指定は行っていない。

 .. note::

    * \ ``@TransactionTokenCheck``\アノテーションのtype属性にBEGINを指定した場合は、新しくTokenKeyが生成されるため、トランザクショントークンのチェックは行われない。
    * \ ``@TransactionTokenCheck``\アノテーションのtype属性にINが指定された場合は、リクエストで指定されたトークン値とサーバ上で保持しているトークン値が同一のものがあるかをチェックする。

.. _doubleSubmit_how_to_use_transaction_token_check_jsp:

トランザクショントークンチェックのView(JSP)での利用方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| トランザクショントークンチェックを行う場合、払い出されたトランザクショントークンが、リクエストパラメータとして送信されるようにView(JSP)を実装する必要がある。
| リクエストパラメータとして送信されるようにする方法としては、\ :ref:`setting`\を行った上で、\ ``<form:form>``\タグを使して自動的にトランザクショントークンをhidden要素に埋め込む方法を推奨する。

- :file:`firstView.jsp`

 .. code-block:: jsp

    <h1>First</h1>
    <form:form method="post" action="transactionTokenCheckExample">
      <input type="submit" name="second" value="second" />
    </form:form>

- :file:`secondView.jsp`

 .. code-block:: jsp
    :emphasize-lines: 2

    <h1>Second</h1>
    <form:form method="post" action="transactionTokenCheckExample"><!-- (1) -->
      <input type="submit" name="third" value="third" />
    </form:form>

- :file:`thirdView.jsp`

 .. code-block:: jsp
    :emphasize-lines: 2

    <h1>Third</h1>
    <form:form method="post" action="transactionTokenCheckExample"><!-- (1) -->
      <input type="submit" name="fourth" value="fourth" />
    </form:form>

- :file:`fourthView.jsp`

 \ ``<form:form>``\タグを使用する場合

 .. code-block:: jsp
    :emphasize-lines: 2

    <h1>Fourth</h1>
    <form:form method="post" action="transactionTokenCheckExample"><!-- (1) -->
      <input type="submit" name="fifth" value="fifth" />
    </form:form>

.. _fourthView:

 \ HTMLの\ ``<form>``\タグを使用する場合

 .. code-block:: jsp
    :emphasize-lines: 3,4-6

    <h1>Fourth</h1>
    <form method="post" action="transactionTokenCheckExample">
      <t:transaction /><!-- (2) -->
      <!-- (3) -->
      <input type="hidden" name="${f:h(_csrf.parameterName)}"
                           value="${f:h(_csrf.token)}"/>
      <input type="submit" name="fifth" value="fifth" />
    </form>

- :file:`fifthView.jsp`

 .. code-block:: jsp

    <h1>Fifth</h1>
    <form:form method="get" action="transactionTokenCheckExample">
      <input type="submit" name="first" value="first" />
    </form:form>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | JSPで、\ ``<form:form>``\タグを使用した場合は、\ ``@TransactionTokenCheck``\ アノテーションのtype属性にBEGINかINを指定すると、\ ``name="_TRANSACTION_TOKEN"``\に対するValueが、hiddenタグとして自動的に埋め込まれる。
   * - | (2)
     - | HTMLの\ ``<form>``\タグを使用する場合は、\ ``<t:transaction />`` を使用することで、(1)と同様のhiddenタグが埋め込まれる。
   * - | (3)
     - | HTMLの\ ``<form>``\タグを使用する場合は、Spring Securityから提供されているCSRFトークンチェックで必要となるcsrfトークンをhidden項目として埋め込む必要がある。
       | CSRFトークンチェックで必要となるcsrfトークンについては、\ :ref:`csrf_formtag-use`\ を参照されたい。

 .. note::
    
    \ ``<form:form>``\タグでを使用すると、CSRFトークンチェックで必要となるパラメータも自動的に埋め込まれる。 CSRFトークンチェックで必要となるパラメータについては、\ :ref:`csrf_formformtag-use`\ を参照されたい。

 .. note::
    
    \ ``<t:transaction />``\は、共通ライブラリから提供しているJSPタグライブラリである。
    (2)で使用している「t:」については、\ :ref:`view_jsp_include-label`\ を参照されたい。

* HTMLの出力例

 .. figure:: ./images/transaction-token-html.png
   :alt: transaction token html
   :width: 100%

出力されたHTMLを確認すると、

* | NameSpaceは、クラスアノテーションのvalue属性で指定した値が設定される。
  | 上記例だと、 \ ``"transactionTokenCheckExample"``\(橙色の下線)がNameSpaceとなる。
* | TokenKeyは、トランザクション開始時に払い出された値が引き回されて設定される。
  | 上記例だと、 \ ``"c0123252d531d7baf730cd49fe0422ef"``\(青色の下線)がTokenKeyとなる。
* | TokenValueは、リクエスト毎に値が変化している。
  | 上記例だと、 \ ``"3f610684e1cb546a13b79b9df30a7523"``\、\ ``"da770ed81dbca9a694b232e84247a13b"``\、
  | \ ``"bd5a2d88ec446b27c06f6d4f486d4428"``\(緑色の下線)がTokenValueとなる。

ことが、わかる。


1つのController内で複数のユースケースを実施する場合
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 1つのController内で複数のユースケースの処理を実装する場合のトランザクショントークンチェックの実装例を以下に示す。
| 下記の例では、(2),(3),(4)を別々のユースケースの画面遷移として扱っている。

- Controller

 .. code-block:: java
    :emphasize-lines: 3,16-17,25-26,41-42,50-51,66-67,75-76

    @Controller
    @RequestMapping("transactionTokenChecFlowkExample")
    @TransactionTokenCheck("transactionTokenChecFlowkExample") // (1)
    public class TransactionTokenCheckFlowExampleController {

        @RequestMapping(value = "flowOne",
                        params = "first", 
                        method = RequestMethod.GET)
        public String flowOneFirst() {
            return "transactionTokenChecFlowkExample/flowOneFirstView";
        }

        @RequestMapping(value = "flowOne",
                        params = "second",
                        method = RequestMethod.POST)
        @TransactionTokenCheck(value = "flowOne",
                               type = TransactionTokenType.BEGIN) // (2)
        public String flowOneSecond() {
            return "transactionTokenChecFlowkExample/flowOneSecondView";
        }

        @RequestMapping(value = "flowOne",
                        params = "third",
                        method = RequestMethod.POST)
        @TransactionTokenCheck(value = "flowOne",
                               type = TransactionTokenType.IN)   // (2)
        public String flowOneThird() {
            return "transactionTokenChecFlowkExample/flowOneThirdView";
        }

        @RequestMapping(value = "flowTwo",
                       params = "first",
                        method = RequestMethod.GET)
        public String flowTwoFirst() {
            return "transactionTokenChecFlowkExample/flowTwoFirstView";
        }

        @RequestMapping(value = "flowTwo",
                        params = "second",
                        method = RequestMethod.POST)
        @TransactionTokenCheck(value = "flowTwo",
                               type = TransactionTokenType.BEGIN) // (3)
        public String flowTwoSecond() {
            return "transactionTokenChecFlowkExample/flowTwoSecondView";
        }

        @RequestMapping(value = "flowTwo",
                        params = "third",
                        method = RequestMethod.POST)
        @TransactionTokenCheck(value = "flowTwo",
                               type = TransactionTokenType.IN) // (3)
        public String flowTwoThird() {
            return "transactionTokenChecFlowkExample/flowTwoThirdView";
        }

        @RequestMapping(value = "flowThree",
                        params = "first",
                        method = RequestMethod.GET)
        public String flowThreeFirst() {
            return "transactionTokenChecFlowkExample/flowThreeFirstView";
        }

        @RequestMapping(value = "flowThree",
                        params = "second",
                        method = RequestMethod.POST)
        @TransactionTokenCheck(value = "flowThree",
                               type = TransactionTokenType.BEGIN) // (4)
        public String flowThreeSecond() {
            return "transactionTokenChecFlowkExample/flowThreeSecondView";
        }

        @RequestMapping(value = "flowThree",
                        params = "third",
                        method = RequestMethod.POST)
        @TransactionTokenCheck(value = "flowThree",
                               type = TransactionTokenType.IN) // (4)
        public String flowThreeThird() {
            return "transactionTokenChecFlowkExample/flowThreeThirdView";
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | クラスアノテーションのvalue属性でNameSpaceを指定する。
       | 上記例では、本ガイドラインの推奨パターンである \ ``@RequestMapping``\のvalue属性と同じ値を指定している。
   * - | (2)
     - | \ ``"flowOne"``\という名前を持つユースケースの処理に対して、トランザクショントークンチェックを行う。
       | 上記例では、本ガイドラインの推奨パターンである \ ``@RequestMapping``\のvalue属性と同じ値を指定している。
   * - | (3)
     - | \ ``"flowTwo"``\という名前を持つユースケースの処理に対して、トランザクショントークンチェックを行う。
       | 上記例では、本ガイドラインの推奨パターンである \ ``@RequestMapping``\のvalue属性と同じ値を指定している。
   * - | (4)
     - | \ ``"flowThree"``\という名前を持つユースケースの処理に対して、トランザクショントークンチェックを行う。
       | 上記例では、本ガイドラインの推奨パターンである \ ``@RequestMapping``\のvalue属性と同じ値を指定している。

 .. note::
 
    ユースケース毎にNameSpaceを割り振ることで、各ユースケース毎にトランザクショントークンのチェックを行うことが出来る。


トランザクショントークンチェックの代表的な適用例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

「入力画面 -> 確認画面 -> 完了画面」といったシンプルな画面遷移を行うユースケースに対して、トランザクショントークンチェックを適用する際の実装例を以下に示す。

- Controller

 .. code-block:: java
    :emphasize-lines: 3,9,16-17,27,37

    @Controller
    @RequestMapping("user")
    @TransactionTokenCheck("user") // (1)
    public class UserController {

        // omitted

        @RequestMapping(value = "create", params = "form")
        public String createForm(UserCreateForm form) { // (2)
          return "user/createForm";
        }

        @RequestMapping(value = "create", 
                      params = "confirm", 
                      method = RequestMethod.POST)
        @TransactionTokenCheck(value = "create", 
                             type = TransactionTokenType.BEGIN) // (3)
        public String createConfirm(@Validated
        UserCreateForm form, BindingResult result) {

            // omitted

            return "user/createConfirm";
        }

        @RequestMapping(value = "create", method = RequestMethod.POST)
        @TransactionTokenCheck(value = "create") // (4)
        public String create(@Validated
        UserCreateForm form, BindingResult result) {

            // omitted

            return "redirect:/user/create?complete";
        }

        @RequestMapping(value = "create", params = "complete")
        public String createComplete() { // (5)
            return "user/createComplete";
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
     - | クラスアノテーションとして、\ ``"user"``\というNameSpaceを設定している。
       | 上記例では、推奨パターンの\ ``@RequestMapping``\アノテーションのvalue属性と同じ値を指定している。
   * - | (2)
     - | 入力画面の表示するための処理メソッド。
       | **ユースケースを開始するための画面ではあるが、データの更新を伴わない表示のみの処理であるため、トランザクションを開始する必要はない。**
       | そのため、上記例では \ ``@TransactionTokenCheck``\アノテーションを指定していない。
   * - | (3)
     - | 入力チェックを行い、確認画面を表示するための処理メソッド。
       | 確認画面には更新処理を実行するためのボタンが配置されているため、このタイミングでトランザクションを開始する。
       | 遷移先には、View（JSP）を指定する。
   * - | (4)
     - | 更新処理を実行するための処理メソッド。
       | **更新処理を行うメソッドなので、トランザクショントークンのチェックを行う。**
   * - | (4)
     - | 完了画面を表示するための処理メソッド。
       | **完了画面を表示するだけなので、トランザクショントークンのチェックは不要である。**
       | そのため、上記例では \ ``@TransactionTokenCheck``\アノテーションを指定していない。

 .. warning::

    \ ``@TransactionTokenCheck``\ アノテーションを定義した処理メソッドの遷移先は、View(JSP)を指定する必要がある。
    リダイレクト先などのView（JSP）以外を遷移先に指定すると、次の処理でTransactionTokenの値が変わっており、必ずTransactionTokenエラーが発生する。

セッション使用時の並行処理の排他制御について
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\ ``@SessionAttribute``\ アノテーションを使用してフォームオブジェクトなどをセッションに格納した場合、
同じ処理の画面遷移を複数並行して行うと、互いの画面操作が干渉しあい、画面に表示されている値とセッション上で保持している値が一致しなくなってしまう事がある。

こうような不整合な状態になっている画面からのリクエストを不正なリクエストとして防ぐ方法として、トランザクショントークンチェック機能を使用することができる。

NameSpaceごとに保持できるトランザクショントークンの上限数を1を設定する。

- :file:`spring-mvc.xml`

 .. code-block:: xml
    :emphasize-lines: 6

    <mvc:interceptor>
        <mvc:mapping path="/**" />
        <!-- omitted -->
        <bean
            class="org.terasoluna.gfw.web.token.transaction.TransactionTokenInterceptor">
            <constructor-arg value="1"/> <!-- (1) -->
        </bean>
    </mvc:interceptor>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | NameSpaceごとのトランザクショントークンの保持数を、"1"に設定する。

 .. note::
 
    \ ``@SessionAttribute``\ アノテーションを使用してフォームオブジェクトなどをセッションに格納した場合は、 NameSpaceごとのトランザクショントークンの保持数を"1"に設定するとこで、
    古いデータを表示している画面からのリクエストを不正なリクエストとして防ぐことが可能となる。

|

How to extend
--------------------------------------------------------------------------------

プログラマティックにトランザクショントークンのライフサイクルを管理する方法について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

以下の設定を追加することで、Controllerの処理メソッドの引数として\ ``org.terasoluna.gfw.web.token.transaction.TransactionTokenContext``\を受け取り、プログラマティックにトランザクショントークンのライフサイクルを管理することができる。

- :file:`spring-mvc.xml`

 .. code-block:: xml
    :emphasize-lines: 3-5

    <mvc:annotation-driven>
      <mvc:argument-resolvers>
        <!-- (1) -->
        <bean
          class="org.terasoluna.gfw.web.token.transaction.TransactionTokenContextHandlerMethodArgumentResolver" />
      </mvc:argument-resolvers>
    </mvc:annotation-driven>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``<mvc:argument-resolvers>``\要素に、Controllerのメソッドの引数として、プログラマティックにトランザクショントークンのライフサイクルを管理するためのオブジェクト(\ ``TransactionTokenContext``\)を引き渡すためのクラス(\ ``TransactionTokenContextHandlerMethodArgumentResolver``\)を設定をする。
       | プログラマティックにトランザクショントークンのライフサイクルを管理する必要がない場合は、本設定は不要である。

 .. note::
 
    使用されなくなったトランザクショントークンは、1つのNameSpaceで保持することが出来る上限値を超えた時点で自動的に破棄されていくため、基本的には、本設定は不要である。

.. _doubleSubmit_how_to_extend_change_max_count:

トランザクショントークンの上限数の変更方法について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

以下の設定を行うことで、1つのNameSpace上で保持する事ができるトランザクショントークンの上限数を変更することができる。

- :file:`spring-mvc.xml`

 .. code-block:: xml
    :emphasize-lines: 8

    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/**" />
            <mvc:exclude-mapping path="/resources/**" />
            <mvc:exclude-mapping path="/**/*.html" />
            <bean
                class="org.terasoluna.gfw.web.token.transaction.TransactionTokenInterceptor" />
            <constructor-arg value="5"/> <!-- (1) -->
        </mvc:interceptor>
    </mvc:interceptors>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``TransactionTokenInterceptor``\のコンストラクタの値として、1つのNameSpace上で保持する事ができるトランザクショントークンの上限数を指定する。
       | デフォルト値(デフォルトコンストラクタ使用時に設定される値)は、10となっている。
       | 上記例では、 デフォルト値(10)から5に変更している。

Appendix
--------------------------------------------------------------------------------

.. _double-submit_disable-cache:

ブラウザキャッシュ無効時のトランザクショントークンチェック
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
HTTPレスポンスヘッダの\ ``Cache-Control``\ の設定により、ブラウザキャッシュが無効になっている場合は、
「\ :ref:`double-submit_transactiontokencheck`\ 」の想定外の操作を行った際に、
トランザクショントークンエラーが発生する前にWebブラウザの有効期限切れメッセージが表示される。

具体的には(8)のブラウザの戻るボタンをクリックすると以下の画面が表示される。図はInternet Explorer 11を使用した場合である。

 .. figure:: ./images_DoubleSubmitProtection/page-expired.png
   :width: 60%

この場合でも二重送信自体は防止されているため、問題はない。
バージョン5.0.0.RELEASE以降の\ :doc:`雛形プロジェクト <../ImplementationAtEachLayer/CreateWebApplicationProject>`\ では、
\ :ref:`Spring Securityの機能 <SpringSecurityAppendixSecHeaders>`\ でキャッシュが無効になる設定が行われている。

もしこの画面の表示が出る代わりにトランザクショントークンエラー画面を表示したい場合は、
\ ``<sec:cache-control />``\ の設定を除外する必要があるが、セキュリティ観点では\ ``<sec:cache-control />``\ を設定しておくことを推奨する。



.. _doubleSubmit_appendix_global_token:

グローバルトークン
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| \ ``@TransactionTokenCheck``\アノテーションのvalue属性の指定を省略すると、グローバルなトランザクショントークンとして扱われる。
| グローバルなトランザクショントークンのNameSpaceには、\ ``"globalToken"``\(固定値)が使用される。

 .. note::

    アプリケーション全体として、単一の画面遷移のみを許容する場合は、NameSpaceごとに保持できるトランザクショントークンの上限数を1に設定し、グルーバルトークンを使用することで実現することが出来る。

アプリケーション全体として、単一の画面遷移のみを許容する場合場合の設定及び実装例を以下に示す。
 
NameSpaceごとに保持できるトランザクショントークンの上限数の変更
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

NameSpaceごとに保持できるトランザクショントークンの上限数を1を設定する。

- :file:`spring-mvc.xml`

 .. code-block:: xml
    :emphasize-lines: 6

    <mvc:interceptor>
        <mvc:mapping path="/**" />
        <!-- omitted -->
        <bean
            class="org.terasoluna.gfw.web.token.transaction.TransactionTokenInterceptor">
            <constructor-arg value="1"/> <!-- (1) -->
        </bean>
    </mvc:interceptor>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | NameSpaceごとのトランザクショントークンの保持数を、"1"に設定する。

Controllerの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

グルーバルトークン用のNameSpaceとなるようにするために、\ ``@TransactionTokenCheck``\アノテーションのvalue属性には、値を指定しない。

- Controller

 .. code-block:: java
    :emphasize-lines: 3,11,17,23

    @Controller
    @RequestMapping("globalTokenCheckExample")
    public class GlobalTokenCheckExampleController { // (1)

        @RequestMapping(params = "first", method = RequestMethod.GET)
        public String first() {
            return "globalTokenCheckExample/firstView";
        }

        @RequestMapping(params = "second", method = RequestMethod.POST)
        @TransactionTokenCheck(type = TransactionTokenType.BEGIN) // (2)
        public String second() {
            return "globalTokenCheckExample/secondView";
        }

        @RequestMapping(params = "third", method = RequestMethod.POST)
        @TransactionTokenCheck // (2)
        public String third() {
            return "globalTokenCheckExample/thirdView";
        }

        @RequestMapping(params = "fourth", method = RequestMethod.POST)
        @TransactionTokenCheck // (2)
        public String fourth() {
            return "globalTokenCheckExample/fourthView";
        }

        @RequestMapping(params = "fifth", method = RequestMethod.POST)
        public String fifth() {
            return "globalTokenCheckExample/fifthView";
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | クラスアノテーションとして、\ ``@TransactionTokenCheck``\ アノテーションを指定しない。
   * - | (2)
     - | メソッドアノテーションとして指定する \ ``@TransactionTokenCheck``\ アノテーションのvalue属性を指定しない。

* HTMLの出力例

 | JSPは、\ :ref:`doubleSubmit_how_to_use_transaction_token_check_jsp`\で用意したJSPと同等のものを用意する。
 | actionを、\ ``"transactionTokenCheckExample"``\から\ ``"globalTokenCheckExample"``\に変更したのみで、他は同じである。

 .. figure:: ./images/transaction-token-global-html.png
   :alt: transaction token global html
   :width: 100%

出力されたHTMLを確認すると、

* | NameSpaceは、\ ``"globalToken"``\という固定値が設定される。
* | TokenKeyは、トランザクション開始時に払い出された値が引き回されて設定される。
  | 上記例だと、 \ ``"9d937be4adc2f5dd2032292d153f1133"``\(青色の下線)がTokenKeyとなる。
* | TokenValueは、リクエスト毎に値が変化している。
  | 上記例だと、 \ ``"9204d7705ce7a17f16ca6cec24cfd88b"``\、\ ``"69c809fefcad541dbd00bd1983af2148"``\、
  | \ ``"6b83f33b365f1270ee1c1b263f046719"``\(緑色の下線)がTokenValueとなる。

ことが、わかる。

以下に、NameSpaceごとのトランザクショントークンの上限数を1に設定して、グローバルトークンを使用した場合の動作について説明する。

 .. figure:: ./images/transaction-token-globaltoken.png
   :alt: transaction token globaltoken
   :width: 90%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | window1の処理にて、TransactionTokenType.BEGINを行い、グローバルトークンを生成する。
   * - | (2)
     - | window2の処理にて、TransactionTokenType.BEGINでtokenを更新する。
       | 内部的に更新ではなく入れ替えとなるが、サーバ上保持できるトランザクショントークンは1つなので、トークンが更新されるイメージとなる。
   * - | (3)
     - | window1の処理のTransactionTokenType.INにて、tokenの値をチェックする。
       | \ **1の処理で生成したトランザクショントークンをリクエストパラメータとして送信するが、サーバ上に指定したトークンが存在しないため、トランザクショントークンエラーとなる。**\
   * - | (4)
     - | window2の処理のTransactionTokenType.INにて、tokenの値をチェックする。
       | 2の処理で生成したトランザクショントークンをリクエストパラメータとして送信し、サーバ上で保持しているトークン値と一致することをチェックする。
       | 一致している場合は、処理が継続される。
   * - | (5)
     - | (4)と同様。
   * - | (6)
     - | (4)と同様。
   * - | (7)
     - | リダイレクトを使用して画面を表示する場合は、トランザクショントークン用のhiddenタグは存在しない。

 .. note::
 
    サーバ上に残っているトランザクショントークンは、グローバルトークンが新たに生成されたタイミングで自動的に削除される。


Quick Reference
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


| 以下の表では、AccountとCustomerを管理する業務アプリケーションを例として、トランザクショントークンに関する設定をどのようにすべきか、また、その際の業務的な制限を示す。
| 例で示す業務アプリケーションで想定するユースケースは、Account,Customerのcreate,update,deleteとする。
| 下記の表を参考に、システム要件にあったトークンの上限数と、Namespaceの設定を行うこと。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.15\linewidth}|p{0.20\linewidth}|p{0.15\linewidth}|p{0.20\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 20 15 20 15 20

   * - 番号
     - Namespace毎に保持するトークン数
     - classで指定したnamespace値
     - メソッドで指定したnamespace値
     - 生成されるトークンの例
     - 業務制限
   * - | (1)
     - | 10 (Default)
     - | account
     - | 指定無し
     - | account~key~value
     - | Accountユースケース全体(create/update/delete)の同時実行数は、10に制限される。
   * - | (2)
     - | 10 (Default)
     - | account
     - | create
     - | account/create~key~value
     - | Accountユースケースのcreate業務の同時実行数は、10に制限される。
   * - | (3)
     - | 10 (Default)
     - | account
     - | update
     - | account/update~key~value
     - | Accountユースケースのupdate業務の同時実行数は、10に制限される。
   * - | (4)
     - | 10 (Default)
     - | account
     - | delete
     - | account/delete~key~value
     - | Accountユースケースのdelete業務の同時実行数は、10に制限される。 ((2),(3),(4)の指定で、accountユースケース全体の同時実行数は、30になること。ほとんどのアプリケーションに対して、この設定は広過ぎるため、デフォルトの10より少ない値でも十分である。)
   * - | (5)
     - | 10 (Default)
     - | 指定無し
     - | create
     - | create~key~value
     - | アプリケーション全体で、createという同一のNamespaceが作成され、その中の同時実行数は、10に制限される。Accountと、Customerという業務が、別にあり、その中でも、createメソッドでTransactionTokenのNameSpaceに"create"と指定した場合、Accountと、Customerのcreateの合計同時実行数は、10に制限される。
   * - | (6)
     - | 10 (Default)
     - | 指定無し
     - | update
     - | update~key~value
     - | (5)と同じ
   * - | (7)
     - | 10 (Default)
     - | 指定無し
     - | delete
     - | delete~key~value
     - | (5)と同じ
   * - | (8)
     - | 10 (Default)
     - | 指定無し
     - | 指定無し
     - | globalToken~key~value
     - | AccountとCustomerユースケース全体の合計同時実行数は10に制限される。
   * - | (9)
     - |  1 (Custom Setting in spring-mvc.xml)
     - | account
     - | 指定無し
     - | account~key~value
     - | Accountユースケース全体の同時実行数は1に制限されること。Accountのcreate/update/deleteは同時には一つの業務しか出来ない。1画面のみを使用した画面遷移を想定した場合、有効。
   * - | (10)
     - |  1 (Custom Setting in spring-mvc.xml)
     - | account
     - | create
     - | account/create~key~value
     - | Accountユースケースのcreate業務の同時実行数は、1に制限されること。Accountのcreateは、2画面開いての実行が、同時にできない。
   * - | (11)
     - |  1 (Custom Setting in spring-mvc.xml)
     - | account
     - | update
     - | account/update~key~value
     - | (10)と同じ
   * - | (12)
     - |  1 (Custom Setting in spring-mvc.xml)
     - | account
     - | delete
     - | account/delete~key~value
     - | (10)と同じ
   * - | (13)
     - |  1 (Custom Setting in spring-mvc.xml)
     - | 指定無し
     - | create
     - | create~key~value
     - | アプリケーション全体でcreateという同一のNamespaceが作成され、その中の同時実行数は、1に制限されること。Accountと、Customerという業務が別にあり、createメソッドでTransactionTokenのNameSpaceに"create"と指定した場合、Accountと、Customerのcreateは、同時に行えない。
   * - | (14)
     - |  1 (Custom Setting in spring-mvc.xml)
     - | 指定無し
     - | update
     - | update~key~value
     - | (13)と同じ
   * - | (15)
     - |  1 (Custom Setting in spring-mvc.xml)
     - | 指定無し
     - | delete
     - | delete~key~value
     - | (13)と同じ
   * - | (16)
     - | 1 (Custom Setting in spring-mvc.xml)
     - | 指定無し
     - | 指定無し
     - | globalToken~key~value
     - | アプリケーション全体の同時実行できる業務は、1に制限される。1セッションでは、1つの操作のみをするプロジェクトで使用すること。

.. raw:: latex

   \newpage

