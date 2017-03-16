代表的なセキュリティ要件の実装例
********************************************************************************

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

はじめに
================================================================================

この章で説明すること
--------------------------------------------------------------------------------

* TERASOLUNA Server Framework for Java (5.x)を利用して代表的なセキュリティ要件を満たすための実装方法の例
* :ref:`app-description-sec` に示すサンプルアプリケーションを題材として、実装方法とソースコードの説明を行う

.. warning::
    * この章で説明している実装方法はあくまでも一例であり、実際の開発においては個別の要件を考慮して実装する必要がある
    * セキュリティ対策の網羅的な実施を保証するものではないため、必要に応じて追加の対策を検討すること

対象読者
--------------------------------------------------------------------------------

* :doc:`../ImplementationAtEachLayer/index` の内容を理解していること
* :doc:`./SpringSecurity`, :doc:`./Authentication`, :doc:`./Authorization` の内容を理解していること
* :doc:`../Tutorial/TutorialSecurity` を実施済みのこと

.. _app-description-sec:

アプリケーションの説明
================================================================================

| 本章では、代表的なセキュリティ要件を満たすサンプルアプリケーションを題材として、セキュリティ対策の具体的な実装方法の例について説明する。
| 以下に本章で実装例を解説するセキュリティ要件の一覧を示し、題材となるサンプルアプリケーションの機能、認証・認可に関する仕様を示す。
| 以降、このサンプルアプリケーションを本アプリケーションと呼ぶ。

.. _sec-requirements:

セキュリティ要件
--------------------------------------------------------------------------------

本アプリケーションが満たすセキュリティ要件の一覧を以下に示す。各分類ごとに、:ref:`implement-description` にて実装例の解説を行う。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.50\linewidth}|p{0.20\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 20 30 40
    :class: longtable

    * - 項番
      - 分類
      - 要件
      - 概説
    * - | (1)
      - :ref:`パスワード変更の強制・促進 <password-change>`
      - 初期パスワード使用時のパスワード変更の強制
      - 初期パスワードを使用して認証成功した際に、パスワードの変更を強制する
    * - | (2)
      -
      - 期限切れパスワードの変更の強制
      - | 一定期間パスワードを変更していないユーザに対して、認証成功時にパスワードの変更を強制する
        | 本アプリケーションでは、管理ユーザのみを対象とする
    * - | (3)
      -
      - パスワード変更を促すメッセージの表示
      - 一定期間パスワードを変更していないユーザに対して、認証成功時にパスワードの変更を促すメッセージを表示する
    * - | (4)
      - :ref:`パスワードの品質チェック <password-strength>`
      - パスワードの最小文字数指定
      - パスワードとして設定できる文字数の最小値を指定する
    * - | (5)
      -
      - パスワードの文字種別指定
      - パスワード中に含めなければならない文字種別（英大文字、英小文字、数字、記号）を指定する
    * - | (6)
      -
      - ユーザ名を含むパスワードの禁止
      - パスワード中にアカウントのユーザ名を含めることを禁止する
    * - | (7)
      -
      - 管理ユーザパスワードの再使用禁止
      - 管理ユーザが、以前使用したパスワードを短期間のうちに再使用することを禁止する
    * - | (8)
      - :ref:`アカウントのロックアウト <account-lock>`
      - アカウントロックアウト
      - あるアカウントが短期間の間に一定回数以上認証に失敗した場合、そのアカウントを認証不能な状態（ロックアウト状態）にする
    * - | (9)
      -
      - アカウントロックアウト期間の指定
      - アカウントのロックアウト状態の継続時間を指定する
    * - | (10)
      -
      - 管理ユーザによるロックアウトの解除
      - 管理ユーザは任意のアカウントのロックアウト状態を解除できる
    * - | (11)
      - :ref:`最終ログイン日時の表示 <last-login>`
      - 前回ログイン日時の表示
      - あるアカウントで認証成功した後、トップ画面にそのアカウントが前回認証に成功した日時を表示する
    * - | (12)
      - :ref:`パスワード再発行のための認証情報の生成 <reissue-info-create>`
      - パスワード再発行用URLへのランダム文字列の付与
      - 不正なアクセスを防ぐため、パスワード再発行画面にアクセスするためのURLに十分に推測困難な文字列を付与する
    * - | (13)
      -
      - パスワード再発行用秘密情報の発行
      - パスワード再発行時のユーザ確認に用いるために、事前に十分に推測困難な秘密情報（ランダム文字列）を生成する
    * - | (14)
      - :ref:`パスワード再発行のための認証情報の配布 <reissue-info-delivery>`
      - パスワード再発行画面URLのメール送付
      - パスワード再発行ページにアクセスするためのURLは、アカウントの登録済みメールアドレスへ送付する
    * - | (15)
      -
      - パスワード再発行画面のURLと秘密情報の別配布
      - パスワード再発行画面のURLの漏えいに備え、秘密情報はメール以外の方法でユーザに配布する
    * - | (16)
      - :ref:`パスワード再発行実行時の検査 <reissue-info-validate>`
      - パスワード再発行用の認証情報の有効期限の設定
      - パスワード再発行画面のURLと秘密情報に有効期限を設定し、有効期限が切れた場合はパスワード再発行画面のURLと秘密情報を使用不能にする
    * - | (17)
      - :ref:`パスワード再発行の失敗上限回数の設定 <reissue-info-invalidate>`
      - パスワード再発行の失敗上限回数の設定
      - パスワード再発行時の認証に一定回数失敗した場合、パスワード再発行画面のURLと秘密情報を使用不能にする
    * - | (18)
      - :ref:`セキュリティ観点での入力値チェック <secure-input-validation>`
      - リクエストパラメータに対する共通的な禁止文字の設定
      - リクエストパラメータに含まれる文字列に対し、アプリケーション全体で共通の禁止文字を設定する
    * - | (19)
      -
      - アップロードファイル名に対する共通的な禁止文字列の設定
      - アップロードされるファイル名に対し、アプリケーション全体で共通の禁止文字を設定する
    * - | (20)
      -
      - 制御文字の入力チェック
      - 入力値に制御文字が含まれていないかをチェックする
    * - | (21)
      -
      - ファイル拡張子の入力チェック
      - アップロードされるファイルの拡張子がアプリケーションで許可されたものであるかをチェックする
    * - | (22)
      -
      - ファイル名の入力チェック
      - アップロードされるファイルのファイル名がアプリケーションで許可されたパターンと一致するかをチェックする
    * - | (23)
      -
      - URLのドメインに対する入力チェック
      - 入力されたURLのドメインがアプリケーションで許可されたものであるかをチェックする
    * - | (24)
      -
      - メールアドレスのドメインに対する入力チェック
      - 入力されたメールアドレスのドメインがアプリケーションで許可されたものであるかをチェックする
    * - | (25)
      - :ref:`監査ログ出力 <audit-logging>`
      - 監査ログ出力
      - 各リクエストに対し、日時、ユーザ名、操作内容、操作結果をログ出力する

.. raw:: latex

   \newpage

機能
--------------------------------------------------------------------------------

本アプリケーションは、:doc:`../Tutorial/TutorialSecurity` で作成したアプリケーションに加え、以下の機能を持つ。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - 機能名
      - 説明
    * - アカウント新規作成機能
      - アカウントを新規作成する機能
    * - パスワード変更機能
      - ログイン済みのユーザが、自分のアカウントのパスワードを変更する機能
    * - アカウントロックアウト機能
      - 短期間に一定回数以上認証に失敗したアカウントを認証不能な状態にする機能
    * - ロックアウト解除機能
      - アカウントロックアウト機能により認証不能な状態になったアカウントを再び認証可能な状態に戻す機能
    * - パスワード再発行機能
      - ユーザがパスワードを忘れてしまった場合に、ユーザ確認を行った後、新しいパスワードを設定できる機能

.. note::
  本アプリケーションはセキュリティ対策に関するサンプルであるため、本来は当然必要となるパスワード以外の登録情報の更新機能等を作成していない。

認証・認可に関する仕様
--------------------------------------------------------------------------------

本アプリケーションにおける、認証・認可に関する仕様についてそれぞれ以下に示す。

認証
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* 認証に使用するための初期パスワードはアプリケーション側から払い出されるものとする

認可
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* ログイン画面、アカウント作成に使用する画面、パスワード再発行に使用する画面以外の画面へのアクセスには、認証が必要
* 「一般ユーザ」と「管理ユーザ」の二種類のロールが存在する
    * 一つのアカウントが複数のロールを持つことができる
* アカウントロックアウト解除機能は、管理ユーザの権限を持つアカウントのみが使用できる

パスワード再発行時の認証
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* パスワード再発行の認証にはアプリケーションが生成する次の二つの情報を用いる
    * パスワード再発行画面のURL
    * 認証用の秘密情報
* アプリケーションが生成するパスワード再発行画面のURLは以下の形式である
    * {baseUrl}/reissue/resetpassword?form&token={token}
        * {baseUrl} : アプリケーションのベースURL
        * {token} : UUID version4形式の文字列（ハイフン込みで36文字、128bit）
* パスワード再発行画面のURLには30分の有効期限を設け、有効期限内のみ認証可能

設計情報
--------------------------------------------------------------------------------

画面遷移
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

画面遷移図を以下に示す。エラー時の画面遷移は省略している。

.. figure:: ./images/SecureLogin_page_transition.png
   :alt: Page Transition
   :width: 80%
   :align: center

.. tabularcolumns:: |p{0.20\linewidth}|p{0.50\linewidth}|p{0.30\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 50 30
    :class: longtable

    * - | 項番
      - | 画面名
      - | アクセスコントロール
    * - | (1)
      - | ログイン画面
      - | -
    * - | (2)
      - | アカウント新規作成画面
      - | -
    * - | (3)
      - | アカウント新規作成入力確認画面
      - | -
    * - | (4)
      - | アカウント新規作成完了画面
      - | -
    * - | (5)
      - | パスワード再発行のための認証情報生成画面
      - | -
    * - | (6)
      - | パスワード再発行のための認証情報生成完了画面
      - | -
    * - | (7)
      - | パスワード再発行画面
      - | -
    * - | (8)
      - | パスワード再発行完了画面
      - | -
    * - | (9)
      - | トップ画面
      - | 認証済みユーザのみ
    * - | (10)
      - | アカウント情報表示画面
      - | 認証済みユーザのみ
    * - | (11)
      - | ロックアウト解除画面
      - | 管理ユーザのみ
    * - | (12)
      - | ロックアウト解除完了画面
      - | 管理ユーザのみ
    * - | (13)
      - | パスワード変更画面
      - | 認証済みユーザのみ
    * - | (14)
      - | パスワード変更完了画面
      - | 認証済みユーザのみ

.. raw:: latex

   \newpage

URL一覧
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
URL一覧を以下に示す。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|p{0.40\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 20 15 15 40
    :class: longtable

    * - 項番
      - プロセス名
      - HTTPメソッド
      - URL
      - 説明
    * - 1
      - ログイン画面表示
      - GET
      - /login
      - ログイン画面を表示する
    * - 2
      - ログイン
      - POST
      - /login
      - ログイン画面から入力されたユーザー名、パスワードを使って認証する(Spring Securityが行う)
    * - 3
      - ログアウト
      - POST
      - /logout
      - ログアウトする(Spring Securityが行う)
    * - 4
      - トップ画面表示
      - GET
      - /
      - トップ画面を表示する
    * - 5
      - アカウント情報表示
      - GET
      - /accounts
      - ログインユーザーのアカウント情報を表示する
    * - 6
      - アカウント新規作成画面
      - GET
      - /accounts/create?form
      - アカウント新規作成画面を表示する
    * - 7
      - アカウント新規作成入力確認画面
      - POST
      - /accounts/create?confirm
      - アカウント新規作成入力確認画面を表示する
    * - 8
      - アカウント新規作成
      - POST
      - /accounts/create
      - 入力された内容でアカウントを新規に作成する
    * - 9
      - アカウント新規作成完了画面
      - GET
      - /accounts/create?complete
      - アカウント新規作成完了画面を表示する
    * - 10
      - パスワード変更画面表示
      - GET
      - /password?form
      - パスワード変更画面を表示する
    * - 11
      - パスワード変更
      - POST
      - /password
      - パスワード変更画面で入力された情報を使用して、アカウントのパスワードを変更する
    * - 12
      - パスワード変更完了画面表示
      - GET
      - /password?complete
      - パスワード変更完了画面を表示する
    * - 13
      - ロックアウト解除画面表示
      - GET
      - /unlock?form
      - ロックアウト解除画面を表示する
    * - 14
      - ロックアウト解除
      - POST
      - /unlock
      - ロック解除画面に入力された情報を使用してアカウントのロックアウトを解除する
    * - 15
      - ロックアウト解除完了画面表示
      - GET
      - /unlock?complete
      - ロックアウト解除完了画面を表示する
    * - 16
      - パスワード再発行のための認証情報生成画面表示
      - GET
      - /reissue/create?form
      - パスワード再発行のための認証情報生成画面を表示する
    * - 17
      - パスワード再発行のための認証情報生成
      - POST
      - /reissue/create
      - パスワード再発行のための認証情報を生成する
    * - 18
      - パスワード再発行のための認証情報生成完了画面表示
      - GET
      - /reissue/create?complete
      - パスワード再発行のための認証情報生成完了画面を表示する
    * - 19
      - パスワード再発行画面表示
      - GET
      - /reissue/resetpassword?form&token={token}
      - 二つのリクエストパラメータを使用して、ユーザ専用のパスワード再発行画面表示を表示する
    * - 20
      - パスワード再発行
      - POST
      - /reissue/resetpassword
      - パスワード再発行画面に入力された情報を使用してパスワードを再発行する
    * - 21
      - パスワード再発行完了画面表示
      - GET
      - /reissue/resetpassword?complete
      - パスワード再発行完了画面を表示する

.. raw:: latex

   \newpage

ER図
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

本アプリケーションにおけるER図を以下に示す。

.. figure:: ./images/SecureLogin_ER.png
   :alt: Entity-Relation Diagram
   :width: 80%
   :align: center

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.40\linewidth}|p{0.30\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 20 40 30
    :class: longtable

    * - 項番
      - エンティティ名
      - 説明
      - 属性
    * - | (1)
      - | アカウント
      - | ユーザの登録済みアカウント情報
      - | username : ユーザ名
        | password : パスワード（ハッシュ化済み）
        | firstName : 名
        | lastName : 姓
        | email : E-mailアドレス
        | url : 個人のWebサイトやブログのURL
        | profile : プロフィール
        | roles : ロール(複数可)
    * - | (2)
      - | アカウント画像
      - | アカウントに対してユーザが登録する画像
      - | username : アカウント画像に対応するアカウントのユーザ名
        | body : 画像ファイルのバイナリ
        | extension : 画像ファイルの拡張子
    * - | (3)
      - | ロール
      - | 認可に使用する権限
      - | roleValue : ロールの識別子
        | roleLabel : ロールの表示名
    * - | (4)
      - | 認証成功イベント
      - | アカウントの最終ログイン日時を取得するために、認証成功時に残す情報
      - | username : ユーザ名
        | authenticationTimestamp : 認証成功日時
    * - | (5)
      - | 認証失敗イベント
      - | アカウントのロックアウト機能で用いるために、認証失敗時に残す情報
      - | username : ユーザ名
        | authenticationTimestamp : 認証失敗日時
    * - | (6)
      - | パスワード変更履歴
      - | パスワードの有効期限の判定等に用いるために、パスワード変更時に残す情報
      - | username : ユーザ名
        | useFrom : 変更後のパスワードの使用開始日時
        | password : 変更後のパスワード
    * - | (7)
      - | パスワード再発行用の認証情報
      - | パスワード再発行時に、ユーザの確認に用いる情報
      - | token : パスワード再発行画面のURLを一意かつ推測不能にするために用いる文字列
        | username : ユーザ名
        | secret : ユーザの確認に用いる文字列
        | experyDate : パスワード再発行用の認証情報の有効期限
    * - | (8)
      - | パスワード再発行失敗イベント
      - | パスワード再発行用の試行回数を制限するために、パスワード再発行失敗に残す情報
      - | token : パスワード再発行に失敗した際に使用したtoken
        | attemptDate : パスワード再発行を試行した日時

.. raw:: latex

   \newpage

.. tip ::

   初期パスワードやパスワード有効期限切れの判定を行うために、アカウントエンティティにフィールドを追加してパスワードの最終変更日時等の情報を持たせるといった設計も可能である。
   そのような方法で実装を行う場合、アカウントのテーブルに様々な状態を判定するためのカラムが追加され、エントリが頻繁に更新されるという状況に繋がりがちである。

   本アプリケーションでは、テーブルをシンプルな状態に保ち、エントリの不要な更新を避けて単純に挿入と削除を使用することで要件を実現するために、認証成功イベントエンティティ等のイベントエンティティを用いた設計を採用している。

.. _implement-description:

実装方法とコード解説
================================================================================

| セキュリティ要件の分類ごとに、本アプリケーションにおける実装の方法とコードの説明を行う。
| ここでは各分類ごとに要件の実現のために必要最小限のコード片のみを掲載している。コード全体を確認したい場合は `GitHub <https://github.com/terasolunaorg/tutorial-apps/tree/release/5.2.0.RELEASE/secure-login-demo>`_ を参照すること。
| 本アプリケーションを動作させるための初期データ登録用SQLは `ここ <https://github.com/terasolunaorg/tutorial-apps/tree/release/5.2.0.RELEASE/secure-login-demo/secure-login-demo/secure-login-env/src/main/resources/database>`_ に配置されている。

.. note::

   本アプリケーションでは、ボイラープレートコードの排除のために、Lombokを使用している。Lombokについては、:doc:`../Appendix/Lombok` を参照。

.. _password-change:

パスワード変更の強制・促進
--------------------------------------------------------------------------------

実装する要件一覧
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* :ref:`初期パスワード使用時のパスワード変更の強制 <sec-requirements>`
* :ref:`期限切れ管理ユーザパスワードの変更の強制 <sec-requirements>`
* :ref:`パスワード変更を促すメッセージの表示 <sec-requirements>`

動作イメージ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. figure:: ./images/SecureLogin_change_password.png
   :alt: Change Password
   :width: 80%
   :align: center

実装方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 本アプリケーションでは、パスワードを変更した際の履歴を「パスワード変更履歴」エンティティとしてデータベースに保存し、このパスワード変更履歴エンティティを使用して、初期パスワードの判定およびパスワードの有効期限切れの判定を行う。
| また、その判定結果に基づいてパスワード変更画面へのリダイレクトや、画面へのメッセージの表示を制御する。
| 具体的には以下の処理を実装して用いることで、要件を実現する。

* パスワード変更履歴エンティティの保存

  パスワードを変更した際に、以下の情報を持ったパスワード変更履歴エンティティをデータベースに登録する。

  * パスワードを変更したアカウントのユーザ名
  * 変更後のパスワードの使用開始日時

* 初期パスワード、パスワード有効期限切れの判定

  | 認証後、認証されたアカウントのパスワード変更履歴エンティティをデータベースから検索し、一件も見つからなければ初期パスワードを使用していると判断する。
  | そうでない場合には、最新のパスワード変更履歴エンティティを取得し、現在日時とパスワードの使用開始日時の差分を計算して、パスワードの有効期限が切れているかどうかの判定を行う。

* パスワード変更画面への強制リダイレクト

  パスワードの変更を強制するために、以下のいずれかに該当する場合には、パスワード変更画面以外へのリクエストが要求された際に、パスワード変更画面へリダイレクトさせる。

  * 認証済みのユーザが初回パスワードを使用している場合
  * 認証済みのユーザが管理ユーザであり、かつパスワードの有効期限が切れている場合

  \ ``org.springframework.web.servlet.handler.HandlerInterceptor`` \ を利用して、Controllerのハンドラメソッド実行前に上記の条件に該当するかどうかの判定を行う。

  .. tip ::

     認証後にパスワード変更画面へリダイレクトさせる方法は他にもあるが、方法によってはリダイレクト後にURLを直打ちすることでパスワード変更を避けて別画面にアクセスできてしまう可能性がある。
     \ ``HandlerInterceptor`` \を使用する方法ではハンドラメソッド実行前に処理を行うため、URLを直打ちするなどの方法で回避することはできない。

  .. tip ::
     \ ``HandlerInterceptor`` \の代わりにServlet Filterを用いることもできる。両者の説明については :ref:`controller-common-process` を参照すること。
     ここでは、アプリケーションが許可したリクエストのみに対して処理を行うために、\ ``HandlerInterceptor`` \を用いている。

* パスワード変更を促すメッセージの表示

  Controllerの中で前述のパスワード有効期限切れ判定処理を呼び出す。判定結果をViewに渡し、Viewでメッセージの表示・非表示を切り替える。

コード解説
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

上記の実装方法に従って実装されたコードについて順に解説する。

* パスワード変更履歴エンティティの保存

  パスワード変更時にパスワード変更履歴エンティティをデータベースに登録するための一連の実装を示す。

  * Entityの実装

    パスワード変更履歴エンティティの実装は以下の通り。

    .. code-block:: java

       package org.terasoluna.securelogin.domain.model;

       // omitted

       @Data
       public class PasswordHistory {

           private String username; // (1)

           private String password; // (2)

           private DateTime useFrom; // (3)

       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | パスワードを変更したアカウントのユーザ名
       * - | (2)
         - | 変更後のパスワード
       * - | (3)
         - | 変更後のパスワードの使用開始日時

  * Repositoryの実装

    データベースに対するパスワード変更履歴エンティティの登録、検索を行うためのRepositoryを以下に示す。

    .. code-block:: java

       package org.terasoluna.securelogin.domain.repository.passwordhistory;

       // omitted

       public interface PasswordHistoryRepository {

           int create(PasswordHistory history); // (1)

           List<PasswordHistory> findByUseFrom(@Param("username") String username,
                   @Param("useFrom") LocalDateTime useFrom); // (2)

           List<PasswordHistory> findLatest(@Param("username") String username,
                   @Param("limit") int limit); // (3)

       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | 引数として与えられた\ ``PasswordHistory`` \ オブジェクトをデータベースのレコードとして登録するメソッド
       * - | (2)
         - | 引数として与えられたユーザ名をキーとして、パスワードの使用開始日時が指定された日付よりも新しい\ ``PasswordHistory`` \ オブジェクトを降順(新しい順)に取得するメソッド
       * - | (3)
         - | 引数として与えられたユーザ名をキーとして、指定された個数の\ ``PasswordHistory`` \ オブジェクトを新しい順に取得するメソッド

    マッピングファイルは以下の通り。

    .. code-block:: xml

       <?xml version="1.0" encoding="UTF-8"?>
       <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

       <mapper
           namespace="org.terasoluna.securelogin.domain.repository.passwordhistory.PasswordHistoryRepository">

           <resultMap id="PasswordHistoryResultMap" type="PasswordHistory">
               <id property="username" column="username" />
               <id property="password" column="password" />
               <id property="useFrom" column="use_from" />
           </resultMap>

           <select id="findByUseFrom" resultMap="PasswordHistoryResultMap">
           <![CDATA[
               SELECT
                   username,
                   password,
                   use_from
               FROM
                   password_history
               WHERE
                   username = #{username} AND
                   use_from >= #{useFrom}
               ORDER BY use_from DESC
           ]]>
           </select>

           <select id="findLatest" resultMap="PasswordHistoryResultMap">
           <![CDATA[
               SELECT
                   username,
                   password,
                   use_from
               FROM
                   password_history
               WHERE
                   username = #{username}
               ORDER BY use_from DESC
               LIMIT #{limit}
           ]]>
           </select>

           <insert id="create" parameterType="PasswordHistory">
           <![CDATA[
               INSERT INTO password_history (
                   username,
                   password,
                   use_from
               ) VALUES (
                   #{username},
                   #{password},
                   #{useFrom}
               )
           ]]>
           </insert>
       </mapper>


  * Serviceの実装

    パスワード変更履歴エンティティの操作は :ref:`パスワードの品質チェック <password-strength>` においても使用する。
    そのため、以下のようにSharedServiceからRepositoryのメソッドを呼び出す。

    .. code-block:: java

       package org.terasoluna.securelogin.domain.service.passwordhistory;

       // omitted

       @Service
       @Transactional
       public class PasswordHistorySharedServiceImpl implements
               PasswordHistorySharedService {

           @Inject
           PasswordHistoryRepository passwordHistoryRepository;

           @Transactional(propagation = Propagation.REQUIRES_NEW)
           public int insert(PasswordHistory history) {
               return passwordHistoryRepository.create(history);
           }

           @Transactional(readOnly = true)
           public List<PasswordHistory> findHistoriesByUseFrom(String username,
                   LocalDateTime useFrom) {
               return passwordHistoryRepository.findByUseFrom(username, useFrom);
           }

           @Override
           @Transactional(readOnly = true)
           public List<PasswordHistory> findLatest(String username, int limit) {
               return passwordHistoryRepository.findLatest(username, limit);
           }

       }

    パスワード変更時にパスワード変更履歴エンティティをデータベースに保存する処理の実装を以下に示す。

    .. code-block:: java

       package org.terasoluna.securelogin.domain.service.account;

       // omitted

       @Service
       @Transactional
       public class AccountSharedServiceImpl implements AccountSharedService {

           @Inject
           ClassicDateFactory dateFactory;

           @Inject
           PasswordHistorySharedService passwordHistorySharedService;

           @Inject
           AccountRepository accountRepository;

           @Inject
           PasswordEncoder passwordEncoder;

           // omitted

           public boolean updatePassword(String username, String rawPassword) { // (1)
               String password = passwordEncoder.encode(rawPassword);
               boolean result = accountRepository.updatePassword(username, password); // (2)

               LocalDateTime passwordChangeDate = dateFactory.newTimestamp().toLocalDateTime();

               PasswordHistory passwordHistory = new PasswordHistory(); // (3)
               passwordHistory.setUsername(username);
               passwordHistory.setPassword(password);
               passwordHistory.setUseFrom(passwordChangeDate);
               passwordHistorySharedService.insert(passwordHistory); // (4)

               return result;
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
         - | パスワードを変更する際に呼び出されるメソッド
       * - | (2)
         - | データベース上のパスワードを更新する処理を呼び出す。
       * - | (3)
         - | パスワード変更履歴エンティティを作成し、ユーザ名、変更後のパスワード、変更後のパスワードの使用開始日時を設定する。
       * - | (4)
         - | 作成したパスワード変更履歴エンティティをデータベースに登録する処理を呼び出す。


* 初期パスワード、パスワード有効期限切れの判定

  データベースに登録されたパスワード変更履歴エンティティを用いて、初期パスワードを使用しているかどうかの判定と、パスワードの有効期限が切れているかどうかを判定する処理の実装を以下に示す。

  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.account;

     // omitted

     @Service
     @Transactional
     public class AccountSharedServiceImpl implements AccountSharedService {

         @Inject
         ClassicDateFactory dateFactory;

         @Inject
         PasswordHistorySharedService passwordHistorySharedService;

         @Value("${security.passwordLifeTimeSeconds}") // (1)
         int passwordLifeTimeSeconds;

         // omitted

        @Transactional(readOnly = true)
        @Override
        @Cacheable("isInitialPassword")
        public boolean isInitialPassword(String username) { // (2)
            List<PasswordHistory> passwordHistories = passwordHistorySharedService
                    .findLatest(username, 1); // (3)
            return passwordHistories.isEmpty(); // (4)
        }

        @Transactional(readOnly = true)
        @Override
        @Cacheable("isCurrentPasswordExpired")
        public boolean isCurrentPasswordExpired(String username) { // (5)
            List<PasswordHistory> passwordHistories = passwordHistorySharedService
                    .findLatest(username, 1); // (6)

            if (passwordHistories.isEmpty()) { // (7)
                return true;
            }

            if (passwordHistories
                    .get(0)
                    .getUseFrom()
                    .isBefore(
                            dateFactory.newTimestamp().toLocalDateTime()
                                    .minusSeconds(passwordLifeTimeSeconds))) { // (8)
                return true;
            }

            return false;
        }

     }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | プロパティファイルからパスワードが有効である期間の長さ（秒単位）を取得し、設定する。
     * - | (2)
       - | 初期パスワードを使用しているかどうかを判定し、使用している場合はtrue、そうでなければfalseを返すメソッド
     * - | (3)
       - | データベースから最新のパスワード変更履歴エンティティを一件取得する処理を呼び出す。
     * - | (4)
       - | データベースからパスワード変更履歴エンティティが取得できなかった場合に、初期パスワードを使用していると判定し、trueを返す。そうでなければfalseを返す。
     * - | (5)
       - | 現在使用中のパスワードの有効期限が切れているかどうかを判定し、切れている場合はtrue、そうでなければfalseを返すメソッド
     * - | (6)
       - | データベースから最新のパスワード変更履歴エンティティを一件取得する処理を呼び出す。
     * - | (7)
       - | データベースからパスワード変更履歴エンティティが取得できなかった場合には、パスワードの有効期限が切れていると判定し、trueを返す。
     * - | (8)
       - | パスワード変更履歴エンティティから取得したパスワードの使用開始日時と現在日時の差分が、(1)で設定したパスワード有効期間よりも大きい場合、パスワードの有効期限が切れていると判定し、trueを返す。
     * - | (9)
       - | (7), (8)のいずれの条件にも該当しない場合、パスワード有効期限内であると判定し、falseを返す。

  .. tip::

     isInitialPassword および isCurrentPasswordExpired に付与されている \ ``@Cacheable``\ は Spring の Cache Abstraction 機能を使用するためのアノテーションである。
     \ ``@Cacheable`` \ アノテーションを付与することで、メソッドの引数に対する結果をキャッシュすることができる。
     ここでは、キャッシュの使用により初期パスワード判定、パスワード期限切れ判定のたびにデータベースへのアクセスが発生することを防止し、パフォーマンスの低下を防いでいる。
     Cache Abstraction については `公式ドキュメント - Cache <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/cache.html>`_ を参照すること。

     尚、キャッシュを使用する際には、必要なタイミングでキャッシュをクリアする必要があることに注意すること。
     本アプリケーションではパスワード変更時や、ログアウト時には再度初期パスワード判定、パスワード期限切れ判定を行うためにキャッシュをクリアする。

     また、必要に応じてキャッシュのTTL(生存時間)を設定すること。TTLは使用するキャッシュの実装によっては設定不能であることに注意。


* パスワード変更画面への強制リダイレクト

  パスワードの変更を強制するために、パスワード変更画面へリダイレクトさせる処理の実装を以下に示す。

  .. code-block:: java

     package org.terasoluna.securelogin.app.common.interceptor;

     // omitted

     public class PasswordExpirationCheckInterceptor extends
             HandlerInterceptorAdapter { // (1)

         @Inject
         AccountSharedService accountSharedService;

         @Override
         public boolean preHandle(HttpServletRequest request,
                 HttpServletResponse response, Object handler) throws IOException { // (2)
             Authentication authentication = (Authentication) request
                     .getUserPrincipal();

             if (authentication != null) {
                 Object principal = authentication.getPrincipal();
                 if (principal instanceof UserDetails) { // (3)
                     LoggedInUser userDetails = (LoggedInUser) principal; // (4)
                     if ((userDetails.getAccount().getRoles().contains(Role.ADMIN) && accountSharedService
                             .isCurrentPasswordExpired(userDetails.getUsername())) // (5)
                             || accountSharedService.isInitialPassword(userDetails
                                     .getUsername())) { // (6)
                         response.sendRedirect(request.getContextPath()
                                 + "/password?form"); // (7)
                         return false; // (8)
                     }
                 }
             }

             return true;
         }
     }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | Controllerのハンドラメソッド実行前に処理を挟み込むために、\ ``org.springframework.web.servlet.handler.HandlerInterceptorAdapter`` \を継承する。
     * - | (2)
       - | Controllerのハンドラメソッド実行前に実行されるメソッド
     * - | (3)
       - | 取得したユーザ情報が\ ``org.springframework.security.core.userdetails.UserDetails`` \のオブジェクトであるかどうかを確認する。
     * - | (4)
       - | \ ``UserDetails`` \のオブジェクトを取得する。本アプリケーションでは、\ ``UserDetails`` \の実装として\ ``LoggedInUser`` \というクラスを作成して用いている。
     * - | (5)
       - | \ ``UserDetails`` \オブジェクトからロールを取得してユーザが管理ユーザであるかどうかを判定する。その後、パスワード有効期限が切れているかどうかを判定する処理を呼び出す。二つの判定結果の論理積(And)をとる。
     * - | (6)
       - | 初回パスワードを使用しているかどうかを判定する処理を呼び出す。
     * - | (7)
       - | (5)または(6)のいずれかが真である場合、\ ``javax.servlet.http.HttpServletResponse`` \の\ ``sendRedirect`` \ メソッドを使用して、パスワード変更画面へリダイレクトさせる。
     * - | (8)
       - | 続けてControllerのハンドラメソッドが実行されることを防ぐために、falseを返す。

  上記のリダイレクト処理を有効にするための設定は以下の通り。

  **spring-mvc.xml**

  .. code-block:: xml

    <!-- omitted -->

    <mvc:interceptors>

        <!-- omitted -->

        <mvc:interceptor>
            <mvc:mapping path="/**" /> <!-- (1) -->
            <mvc:exclude-mapping path="/password/**" /> <!-- (2) -->
            <mvc:exclude-mapping path="/reissue/**" /> <!-- (3) -->
            <mvc:exclude-mapping path="/resources/**" />
            <mvc:exclude-mapping path="/**/*.html" />
            <bean
                class="org.terasoluna.securelogin.app.common.interceptor.PasswordExpirationCheckInterceptor" /> <!-- (4) -->
        </mvc:interceptor>

        <!-- omitted -->

    </mvc:interceptors>

    <!-- omitted -->

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | "/"以下のすべてのパスに対するアクセスに\ ``HandlerInterceptor`` \を適用する。
     * - | (2)
       - | パスワード変更画面からパスワード変更画面へのリダイレクトを防ぐため、 "/password" 以下のパスは適用対象外とする。
     * - | (3)
       - | パスワード再発行時にはパスワード有効期限のチェックを行う必要はないため、 "/reissue" 以下のパスは適用対象外とする。
     * - | (4)
       - | \ ``HandlerInterceptor`` \のクラスを指定する。

* パスワード変更を促すメッセージの表示

  トップ画面にパスワード変更を促すメッセージを表示するための、Controllerの実装を以下に示す。

  .. code-block:: java

     package org.terasoluna.securelogin.app.welcome;

     // omitted

     @Controller
     public class HomeController {

         @Inject
         AccountSharedService accountSharedService;

         @RequestMapping(value = "/", method = { RequestMethod.GET,
                 RequestMethod.POST })
         public String home(@AuthenticationPrincipal LoggedInUser userDetails, // (1)
                 Model model) {

             Account account = userDetails.getAccount(); // (2)

             model.addAttribute("account", account);

             if (accountSharedService
                    .isCurrentPasswordExpired(account.getUsername())) { // (3)
                 ResultMessages messages = ResultMessages.warning().add(
                         "w.sl.pe.0001");
                 model.addAttribute(messages);
             }

             // omitted

             return "welcome/home";

         }

     }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | \ ``AuthenticationPrincipal`` \アノテーションを指定して、\ ``UserDetails`` \を実装した\ ``LoggedInUser`` \のオブジェクトを取得する。
     * - | (2)
       - | \ ``LoggedInUser`` \が保持しているアカウント情報を取得する。
     * - | (3)
       - | アカウント情報から取得したユーザ名を引数として、パスワードの有効期限切れ判定処理を呼び出す。判定結果がtrueの場合、プロパティファイルからメッセージを取得してModelに設定し、Viewに渡す。

  Viewの実装は以下の通り。

  **トップ画面(home.jsp)**

  .. code-block:: jsp

     <!-- omitted -->

     <body>
        <div id="wrapper">
            <span id="expiredMessage">
                <t:messagesPanel /> <!-- (1) -->
            </span>

            <!-- omitted -->

        </div>
     </body>

     <!-- omitted -->

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | messagesPanelタグを用いて、Controllerから渡されたパスワード有効期限切れメッセージを表示する。

.. _password-strength:

パスワードの品質チェック
--------------------------------------------------------------------------------
実装する要件一覧
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
* :ref:`パスワードの最小文字数指定 <sec-requirements>`
* :ref:`パスワードの文字種別指定 <sec-requirements>`
* :ref:`ユーザ名を含むパスワードの禁止 <sec-requirements>`
* :ref:`管理ユーザパスワードの再使用禁止 <sec-requirements>`

動作イメージ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. figure:: ./images/SecureLogin_password_validation.png
   :alt: Password Validation
   :width: 80%
   :align: center

実装方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| パスワード変更時等にユーザが指定したパスワードの品質を検査するためには、 :doc:`../ArchitectureInDetail/WebApplicationDetail/Validation` の機能を利用することができる。本アプリケーションではBean Validationを用いてパスワードの品質を検査する。
| パスワードの品質として求められる要件はアプリケーションによって異なり、多岐に渡る。
| そこで、パスワード入力チェック用のライブラリとして `Passay <http://www.passay.org/>`_ を利用し、必要なBean Validationのアノテーションを作成する。
| Passayではパスワード入力チェックで一般的に使用される機能の多くを提供しており、提供されていない機能についても標準機能を拡張することで容易に実装することができる。
| Passayの概要については :ref:`Appendix <passay_overview>` を参照。
| 具体的には以下の設定、処理を記述し、使用することで要件を実現する。

* Passayの検証規則の作成

  要件の実現に用いるために、以下の検証規則を作成する。

    * パスワード長の最小値を設定した検証規則
    * パスワードに含めなければならない文字種別を設定した検証規則
    * パスワードがユーザ名を含まないことをチェックするための検証規則
    * 同一のパスワードを過去に使用していないことをチェックするための検証規則

* Passayの検証器の作成

  上記で作成した検証規則を設定した、Passayの検証器を作成する。

* Bean Validationのアノテーションの作成

  Passayの検証器を使用してパスワードの入力チェックを行うためのアノテーションを作成する。
  一つのアノテーションですべての検証規則を検査することもできるが、多種の規則の検査を行うことで処理が複雑になり視認性が下がることを避けるため、以下の二つに分けて実装する。

    * パスワード自体の性質を検証するアノテーション

      「パスワードが最小文字列長よりも長いこと」、「指定した文字種別の文字を含むこと」、「ユーザ名を含まないこと」の三つの検証規則をチェックする
    * 過去のパスワードとの比較を行うアノテーション

      管理ユーザが、以前使用したパスワードを短期間のうちに再使用していないことをチェックする

  いずれのアノテーションも、ユーザ名と新しいパスワードを用いる相関入力チェックルールとなる。
  両方のルールに違反した入力を行った場合、それぞれのエラーメッセージが表示される。

* パスワードの入力チェック

  作成したBean Validationアノテーションを用いて、パスワードの入力チェックを行う。

コード解説
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

上記の実装方法に従って実装されたコードについて順に解説する。Passayを用いたパスワード入力チェックについては :ref:`password_validation` にて説明する。

* Passayの検証規則の作成

  | 本アプリケーションで使用するほとんどの検証規則は、Passayにデフォルトで用意されたクラスを利用することで定義できる。
  | しかしながら、Passayが提供するクラスでは、\ ``org.springframework.security.crypto.password.PasswordEncoder`` \でハッシュ化された過去のパスワードと比較する検証規則を定義することができない。
  | そのため、Passayが提供するクラスを拡張し、独自の検証規則のクラスを以下のように作成する必要がある。

  .. code-block:: java

     package org.terasoluna.securelogin.app.common.validation.rule;

     // omitted

     public class EncodedPasswordHistoryRule extends HistoryRule { // (1)

         PasswordEncoder passwordEncoder; // (2)

         public EncodedPasswordHistoryRule(PasswordEncoder passwordEncoder) {
             this.passwordEncoder = passwordEncoder;
         }

         @Override
         protected boolean matches(final String clearText,
                 final PasswordData.Reference reference) { // (3)
             return passwordEncoder.matches(clearText, reference.getPassword()); // (4)
         }
     }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | パスワードが過去に使用したパスワードに含まれないをチェックするための\ ``org.passay.HistoryRule`` \を拡張する。
     * - | (2)
       - | パスワードのハッシュ化に用いている\ ``PasswordEncoder`` \ をインジェクションする。
     * - | (3)
       - | 過去のパスワードとの比較を行うメソッドをオーバーライドする。
     * - | (4)
       - | \ ``PasswordEncoder`` \ の \ ``matches`` \ メソッドを使用してハッシュ化されたパスワードとの比較を行う。

  Passayの検証規則を以下に示す通りBean定義する。

  **applicationContext.xml**

  .. code-block:: xml

     <bean id="lengthRule" class="org.passay.LengthRule"> <!-- (1) -->
         <property name="minimumLength" value="${security.passwordMinimumLength}" />
     </bean>
     <bean id="upperCaseRule" class="org.passay.CharacterRule"> <!-- (2) -->
         <constructor-arg name="data">
             <util:constant static-field="org.passay.EnglishCharacterData.UpperCase" />
         </constructor-arg>
         <constructor-arg name="num" value="1" />
     </bean>
     <bean id="lowerCaseRule" class="org.passay.CharacterRule"> <!-- (3) -->
         <constructor-arg name="data">
             <util:constant static-field="org.passay.EnglishCharacterData.LowerCase" />
         </constructor-arg>
         <constructor-arg name="num" value="1" />
     </bean>
     <bean id="digitRule" class="org.passay.CharacterRule"> <!-- (4) -->
         <constructor-arg name="data">
             <util:constant static-field="org.passay.EnglishCharacterData.Digit" />
         </constructor-arg>
         <constructor-arg name="num" value="1" />
     </bean>
     <bean id="specialCharacterRule" class="org.passay.CharacterRule"> <!-- (5) -->
         <constructor-arg name="data">
             <util:constant static-field="org.passay.EnglishCharacterData.Special" />
         </constructor-arg>
         <constructor-arg name="num" value="1" />
     </bean>
     <bean id="characterCharacteristicsRule" class="org.passay.CharacterCharacteristicsRule"> <!-- (6) -->
         <property name="rules">
             <list>
                 <ref bean="upperCaseRule" />
                 <ref bean="lowerCaseRule" />
                 <ref bean="digitRule" />
                 <ref bean="specialCharacterRule" />
             </list>
         </property>
         <property name="numberOfCharacteristics" value="3" />
     </bean>
     <bean id="usernameRule" class="org.passay.UsernameRule" /> <!-- (7) -->
     <bean id="encodedPasswordHistoryRule"
         class="org.terasoluna.securelogin.app.common.validation.rule.EncodedPasswordHistoryRule"> <!-- (8) -->
         <constructor-arg name="passwordEncoder" ref="passwordEncoder" />
     </bean>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | パスワードの長さをチェックするための\ ``org.passay.LengthRule`` \のプロパティに、プロパティファイルから取得したパスワードの最短長を設定する。
     * - | (2)
       - | 半角英大文字を一文字以上含むことをチェックする検証規則。パスワードに含まれる文字種別に関するチェックを行うための\ ``org.passay.CharacterRule`` \のコンストラクタに、\ ``org.passay.EnglishCharacterData.UpperCase`` \と数値の1を設定する。
     * - | (3)
       - | 半角英小文字を一文字以上含むことをチェックする検証規則。パスワードに含まれる文字種別に関するチェックを行うための\ ``org.passay.CharacterRule`` \のコンストラクタに、\ ``org.passay.EnglishCharacterData.LowerCase`` \と数値の1を設定する。
     * - | (4)
       - | 半角数字を一文字以上含むことをチェックする検証規則。パスワードに含まれる文字種別に関するチェックを行うための\ ``org.passay.CharacterRule`` \のコンストラクタに、\ ``org.passay.EnglishCharacterData.Digit`` \と数値の1を設定する。
     * - | (5)
       - | 半角記号を一文字以上含むことをチェックする検証規則。パスワードに含まれる文字種別に関するチェックを行うための\ ``org.passay.CharacterRule`` \のコンストラクタに、\ ``org.passay.EnglishCharacterData.Special`` \と数値の1を設定する。
     * - | (6)
       - | (2)-(5)の4つの検証規則のうち、3つを満たすことをチェックするための検証規則。\ ``org.passay.CharacterCharacteristicsRule`` \のプロパティに、(2)-(5)で定義したBeanのリストと、数値の3を設定する。
     * - | (7)
       - | パスワードにユーザ名が含まれていないことをチェックするための検証規則
     * - | (8)
       - | パスワードが過去に使用したものの中に含まれていないことをチェックするための検証規則

* Passayの検証器の作成

  前述したPassayの検証規則を用いて、実際に検証を行う検証器のBean定義を以下に示す。

  **applicationContext.xml**

  .. code-block:: xml

     <bean id="characteristicPasswordValidator" class="org.passay.PasswordValidator"> <!-- (1) -->
         <constructor-arg name="rules">
             <list>
                 <ref bean="lengthRule" />
                 <ref bean="characterCharacteristicsRule" />
                 <ref bean="usernameRule" />
             </list>
         </constructor-arg>
     </bean>
     <bean id="encodedPasswordHistoryValidator" class="org.passay.PasswordValidator"> <!-- (2) -->
         <constructor-arg name="rules">
             <list>
                 <ref bean="encodedPasswordHistoryRule" />
             </list>
         </constructor-arg>
     </bean>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | パスワード自体の性質を検証するための検証器。プロパティとして、\ ``LengthRule`` \, \ ``CharacterCharacteristicsRule`` \, \ ``UsernameRule`` \のBeanを設定する。
     * - | (2)
       - | 過去に使用したパスワードの履歴を使用したチェックを行うための検証器。プロパティとして\ ``EncodedPasswordHistoryRule`` \のBeanを設定する。

* Bean Validationのアノテーションの作成

  要件を実現するために、前述した検証器を使用する2つのアノテーションを作成する。

  * パスワード自体の性質を検証するアノテーション

    パスワードが最小文字列長よりも長いこと、指定した文字種別の文字を含むこと、ユーザ名を含まないことという三つの検証規則をチェックするアノテーションの実装を以下に示す。

    .. code-block:: java

       package org.terasoluna.securelogin.app.common.validation;

       // omitted

       @Documented
       @Constraint(validatedBy = { StrongPasswordValidator.class }) // (1)
       @Target({ TYPE, ANNOTATION_TYPE })
       @Retention(RUNTIME)
       public @interface StrongPassword {
           String message() default "{org.terasoluna.securelogin.app.common.validation.StrongPassword.message}";

           Class<?>[] groups() default {};

           String usernamePropertyName(); // (2)

           String newPasswordPropertyName(); // (3)

           @Target({ TYPE, ANNOTATION_TYPE })
           @Retention(RUNTIME)
           @Documented
           public @interface List {
               StrongPassword[] value();
           }

           Class<? extends Payload>[] payload() default {};
       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | アノテーション付与時に使用する\ ``ConstraintValidator`` \を指定する。
       * - | (2)
         - | ユーザ名のプロパティ名を指定するためのプロパティ。
       * - | (3)
         - | パスワードのプロパティ名を指定するためのプロパティ。

    .. code-block:: java

       package org.terasoluna.securelogin.app.common.validation;

       // omitted

       public class StrongPasswordValidator implements
               ConstraintValidator<StrongPassword, Object> {

           @Inject
           @Named("characteristicPasswordValidator") // (1)
           PasswordValidator characteristicPasswordValidator;

           private String usernamePropertyName;

           private String newPasswordPropertyName;

           @Override
           public void initialize(StrongPassword constraintAnnotation) {
               usernamePropertyName = constraintAnnotation.usernamePropertyName();
               newPasswordPropertyName = constraintAnnotation.newPasswordPropertyName();
           }

           @Override
           public boolean isValid(Object value, ConstraintValidatorContext context) {
               BeanWrapper beanWrapper = new BeanWrapperImpl(value);
               String username = (String) beanWrapper.getPropertyValue(usernamePropertyName);
               String newPassword = (String) beanWrapper
                       .getPropertyValue(newPasswordPropertyName);

               RuleResult result = characteristicPasswordValidator
                       .validate(PasswordData.newInstance(newPassword, username, null)); // (2)

               if (result.isValid()) { // (3)
                   return true;
               } else {
                   context.disableDefaultConstraintViolation();
                   for (String message : characteristicPasswordValidator
                           .getMessages(result)) { // (4)
                       context.buildConstraintViolationWithTemplate(message)
                               .addPropertyNode(newPasswordPropertyName)
                               .addConstraintViolation();
                   }
                   return false;
               }
           }
       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | Passayの検証器をインジェクションする。
       * - | (2)
         - | パスワードとユーザ名を指定した\ ``org.passay.PasswordData`` \のインスタンスを作成し、検証器で入力チェックを行う。
       * - | (3)
         - | チェックの結果を確認し、OKならばtrueを返し、そうでなければfalseを返す。
       * - | (4)
         - | パスワード入力チェックエラーメッセージをすべて取得し、設定する。

  * 過去のパスワードとの比較を行うアノテーション

    | 管理ユーザが、以前使用したパスワードを短期間のうちに再使用していないことをチェックするアノテーションの実装を以下に示す。
    | 過去に使用したパスワードを取得するために、パスワード変更履歴エンティティを用いる。パスワード変更履歴エンティティについては :ref:`パスワード変更の強制・促進 <password-change>` を参照。

    .. note ::

       「いくつ前までのパスワードの再使用を禁止するか」のみの設定では、短時間の間にパスワード変更を繰り返すことでパスワードを再使用することが可能となってしまう。
       これを防ぐために、本アプリケーションでは「いつ以降使用したパスワードの再使用を禁止するか」を設定して検査を行う。

    .. code-block:: java

       package org.terasoluna.securelogin.app.common.validation;

       @Documented
       @Constraint(validatedBy = { NotReusedPasswordValidator.class }) // (1)
       @Target({ TYPE, ANNOTATION_TYPE })
       @Retention(RUNTIME)
       public @interface NotReusedPassword {
           String message() default "{org.terasoluna.securelogin.app.common.validation.NotReusedPassword.message}";

           Class<?>[] groups() default {};

           String usernamePropertyName(); // (2)

           String newPasswordPropertyName(); // (3)

           @Target({ TYPE, ANNOTATION_TYPE })
           @Retention(RUNTIME)
           @Documented
           public @interface List {
               NotReusedPassword[] value();
           }

           Class<? extends Payload>[] payload() default {};
       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | アノテーション付与時に使用する\ ``ConstraintValidator`` \を指定する。
       * - | (2)
         - | ユーザ名のプロパティ名を指定するためのプロパティ。データベースから過去に使用したパスワードを検索するために必要となる。
       * - | (3)
         - | パスワードのプロパティ名を指定するためのプロパティ。

    .. code-block:: java

       package org.terasoluna.securelogin.app.common.validation;

       // omitted

       public class NotReusedPasswordValidator implements
               ConstraintValidator<NotReusedPassword, Object> {

           @Inject
           ClassicDateFactory dateFactory;

           @Inject
           AccountSharedService accountSharedService;

           @Inject
           PasswordHistorySharedService passwordHistorySharedService;

           @Inject
           PasswordEncoder passwordEncoder;

           @Inject
           @Named("encodedPasswordHistoryValidator") // (1)
           PasswordValidator encodedPasswordHistoryValidator;

           @Value("${security.passwordHistoricalCheckingCount}") // (2)
           int passwordHistoricalCheckingCount;

           @Value("${security.passwordHistoricalCheckingPeriod}") // (3)
           int passwordHistoricalCheckingPeriod;

           private String usernamePropertyName;

           private String newPasswordPropertyName;

           private String message;

           @Override
           public void initialize(NotReusedPassword constraintAnnotation) {
               usernamePropertyName = constraintAnnotation.usernamePropertyName();
               newPasswordPropertyName = constraintAnnotation.newPasswordPropertyName();
               message = constraintAnnotation.message();
           }

           @Override
           public boolean isValid(Object value, ConstraintValidatorContext context) {
               BeanWrapper beanWrapper = new BeanWrapperImpl(value);
               String username = (String) beanWrapper.getPropertyValue(usernamePropertyName);
               String newPassword = (String) beanWrapper
                       .getPropertyValue(newPasswordPropertyName);

               Account account = accountSharedService.findOne(username);
               String currentPassword = account.getPassword();

               boolean result = checkNewPasswordDifferentFromCurrentPassword(
                       newPassword, currentPassword, context); // (4)
               if (result && account.getRoles().contains(Role.ADMIN)) { // (5)
                   result = checkHistoricalPassword(username, newPassword, context);
               }

               return result;
           }

           private boolean checkNewPasswordDifferentFromCurrentPassword(
                   String newPassword, String currentPassword,
                   ConstraintValidatorContext context) {
               if (!passwordEncoder.matches(newPassword, currentPassword)) {
                   return true;
               } else {
                   context.disableDefaultConstraintViolation();
                   context.buildConstraintViolationWithTemplate(message)
                           .addPropertyNode(newPasswordPropertyName).addConstraintViolation();
                   return false;
               }
           }

           private boolean checkHistoricalPassword(String username,
                   String newPassword, ConstraintValidatorContext context) {
               LocalDateTime useFrom = dateFactory.newTimestamp().toLocalDateTime()
                       .minusMinutes(passwordHistoricalCheckingPeriod);
               List<PasswordHistory> historyByTime = passwordHistorySharedService
                       .findHistoriesByUseFrom(username, useFrom);
               List<PasswordHistory> historyByCount = passwordHistorySharedService
                       .findLatest(username, passwordHistoricalCheckingCount);
               List<PasswordHistory> history = historyByCount.size() > historyByTime
                       .size() ? historyByCount : historyByTime; // (6)

               List<PasswordData.Reference> historyData = new ArrayList<>();
               for (PasswordHistory h : history) {
                   historyData.add(new PasswordData.HistoricalReference(h
                           .getPassword())); // (7)
               }

               PasswordData passwordData = PasswordData.newInstance(newPassword,
                       username, historyData); // (8)
               RuleResult result = encodedPasswordHistoryValidator
                       .validate(passwordData); // (9)

               if (result.isValid()) { // (10)
                   return true;
               } else {
                   context.disableDefaultConstraintViolation();
                   context.buildConstraintViolationWithTemplate(
                           encodedPasswordHistoryValidator.getMessages(result).get(0)) // (11)
                           .addPropertyNode(newPasswordPropertyName).addConstraintViolation();
                   return false;
               }
           }
       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90
       :class: longtable

       * - 項番
         - 説明
       * - | (1)
         - | Passayの検証器をインジェクションする。
       * - | (2)
         - | いくつ前までのパスワードの再使用を禁止するかの閾値をプロパティファイルから取得し、インジェクションする。
       * - | (3)
         - | いつ以降使用したパスワードの再使用を禁止するかの閾値（秒数）をプロパティファイルから取得し、インジェクションする。
       * - | (4)
         - | 新しいパスワードが現在使用しているものと異なるかどうかをチェックする処理を呼び出す。このチェックは一般ユーザ・管理ユーザにかかわらず行う。
       * - | (5)
         - | 管理ユーザの場合は、新しいパスワードが過去に使用したパスワードに含まれていないかをチェックする処理を呼び出す。
       * - | (6)
         - | (2)で指定した個数分のパスワード変更履歴エンティティと、(3)で指定した期間分のパスワード変更履歴エンティティを取得し、どちらか数の多い方を以降のチェックに用いる。
       * - | (7)
         - | Passayの検証器で過去のパスワードとの比較を行うために、パスワード変更履歴エンティティからパスワードを取得し、\ ``org.passay.PasswordData.HistoricalReference`` \のリストを作成する。
       * - | (8)
         - | パスワード、ユーザ名、過去のパスワードのリストを指定した\ ``org.passay.PasswordData`` \のインスタンスを作成する。
       * - | (9)
         - | 検証器で入力チェックを行う。
       * - | (10)
         - | チェック結果を確認し、OKならばtrueを返し、そうでなければfalseを返す。
       * - | (11)
         - | パスワード入力チェックエラーメッセージを取得する。

    .. raw:: latex

       \newpage

* パスワードの入力チェック

  Bean Validationアノテーションを使用してアプリケーション層で、パスワード入力チェックを行う。
  Formクラスに付与されたアノテーションによってNullチェック以外の入力チェックが網羅されていることから、単項目チェックとしては\ ``@NotNull`` \のみを付与している。

  .. code-block:: java

     package org.terasoluna.securelogin.app.passwordchange;

     // omitted

     import lombok.Data;

     @Data
     @Compare(source = "newPasssword", destination = "confirmNewPassword", operator = Compare.Operator.EQUAL) // (1)
     @StrongPassword(usernamePropertyName = "username", newPasswordPropertyName = "newPassword") // (2)
     @NotReusedPassword(usernamePropertyName = "username", newPasswordPropertyName = "newPassword") // (3)
     @ConfirmOldPassword(usernamePropertyName = "username", oldPasswordPropertyName = "oldPassword") // (4)
     public class PasswordChangeForm implements Serializable {

         private static final long serialVersionUID = 1L;

         @NotNull
         private String username;

         @NotNull
         private String oldPassword;

         @NotNull
         private String newPassword;

         @NotNull
         private String confirmNewPassword;

     }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | 新しいパスワードの二回の入力が一致しているかをチェックするためのアノテーション。詳細は :ref:`Validation_terasoluna_gfw_list` を参照すること。
     * - | (2)
       - | 上述した、パスワード自体の性質を検証するアノテーション
     * - | (3)
       - | 過去のパスワードとの比較を行うアノテーション
     * - | (4)
       - | 入力された現在のパスワードが正しいことをチェックするアノテーション。定義は割愛する。

  .. code-block:: java

     package org.terasoluna.securelogin.app.passwordchange;

     // omitted

     @Controller
     @RequestMapping("password")
     public class PasswordChangeController {

         @Inject
         PasswordChangeService passwordService;

         // omitted

         @RequestMapping(method = RequestMethod.POST)
         public String change(@AuthenticationPrincipal LoggedInUser userDetails,
                 @Validated PasswordChangeForm form, BindingResult bindingResult, // (1)
                 Model model) {

             Account account = userDetails.getAccount();
             if (bindingResult.hasErrors()
                     || !account.getUsername().equals(form.getUsername())) { // (2)
                 model.addAttribute(account);
                 return "passwordchange/changeForm";
             }

             passwordService.updatePassword(form.getUsername(),
                     form.getNewPassword());

             return "redirect:/password?complete";
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
       - | パスワード変更時に呼び出されるハンドラメソッド。パラメータ中のFormに\ ``@Validated`` \ アノテーションを付与して、入力チェックを行う。
     * - | (2)
       - | パスワード変更対象のユーザ名がログイン中のアカウントのユーザ名と一致していることを確認する。両者が異なる場合には、再度パスワード変更画面へ遷移させる。

  .. note::

     本アプリケーションではBean Valiidationでユーザ名を用いたパスワード入力チェックを行うために、ユーザ名をFormから取得している。
     Viewでは\ ``Model`` \に設定したユーザ名をhiddenで保持することを想定しているが、改ざんされる恐れがあるため、パスワード変更前にFormから取得したユーザ名の確認を行っている。

.. _account-lock:

アカウントのロックアウト
--------------------------------------------------------------------------------
実装する要件一覧
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
* :ref:`アカウントロックアウト <sec-requirements>`
* :ref:`アカウントロックアウト期間の指定 <sec-requirements>`
* :ref:`管理ユーザによるロックアウトの解除 <sec-requirements>`

動作イメージ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* アカウントロックアウト

.. figure:: ./images/SecureLogin_lockout_ss.png
   :alt: Lockout
   :width: 80%
   :align: center

| ログインフォームにて、あるユーザ名に対して短時間に一定回数連続して誤ったパスワードで認証を試行すると、そのユーザのアカウントはロックアウト状態となる。
  ロックアウト状態のアカウントは、正しいユーザ名とパスワードの組を入力した場合であっても認証されない。
| ロックアウト状態は一定期間経過するか、ロックアウト解除を行うことで解消される。

* ロックアウト解除

.. figure:: ./images/SecureLogin_unlock_ss.png
   :alt: Unlock
   :width: 80%
   :align: center

管理権限を持つユーザでログインした場合にのみ、ロックアウト解除機能を使用することができる。
ロックアウト状態を解消したいユーザ名を入力してロックアウト解除を実行すると、そのユーザのアカウントは再び認証可能な状態に戻る。

実装方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Spring Securityでは、\ ``org.springframework.security.core.userdetails.UserDetails`` \に対してアカウントのロックアウト状態を設定することができる。
| 「ロックアウト状態である」と設定した場合、Spring Securityがその設定を読み取って\ ``org.springframework.security.authentication.LockedException`` \をthrowする。
| この機能を用いることにより、アカウントがロックアウト状態であるか否かを判定して\ ``UserDetails`` \に設定する処理のみを実装すれば、ロックアウト機能が実現できる。

| 本アプリケーションでは、認証に失敗した履歴を「認証失敗イベント」エンティティとしてデータベースに保存し、この認証失敗イベントエンティティを使用してアカウントのロックアウト状態の判定を行う。
| 具体的には以下の三つの処理を実装して用いることにより、アカウントのロックアウトに関する各要件を実現する。

* 認証失敗イベントエンティティの保存

  不正な認証情報の入力によって認証に失敗した際に、Spring Securityが発生させるイベントをハンドリングし、認証に使用したユーザ名と認証を試みた日時を認証失敗イベントエンティティとしてデータベースに登録する。

* ロックアウト状態の判定

  あるアカウントについて、現在時刻から一定以上新しい認証失敗イベントエンティティが一定個数以上存在する場合、該当アカウントはロックアウト状態であると判定する。
  認証時にこの判定処理を呼び出し、判定結果を\ ``UserDetails`` \の実装クラスに設定する。

* 認証失敗イベントエンティティの削除

  | あるアカウントについて、認証失敗イベントエンティティをすべて削除する。
  | ロックアウトの対象となるのは連続して認証に失敗した場合のみであるため、認証に成功した際には認証失敗イベントエンティティを削除する。
  | また、アカウントのロックアウト状態は認証失敗イベントエンティティを用いて判定されるため、認証失敗イベントエンティティを消去することでロックアウト解除機能が実現できる。
    アカウントのロックアウトは認可機能を用いて、管理ユーザ以外実行できないようにする。

.. warning::

   認証失敗イベントエンティティはロックアウトの判定のみを目的としているため、不要になったタイミングで消去する。
   認証ログが必要な場合は必ず別途ログを保存しておくこと。

認証失敗イベントエンティティを用いたロックアウト機能の動作例を以下の図を用いて説明する。
例として3回の認証失敗でロックアウトされるものとし、ロックアウト継続時間は10分とする。

.. figure:: ./images/SecureLogin_lockout.png
   :alt: Account Lockout
   :width: 60%
   :align: center

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 過去10分以内に、誤ったパスワードでの認証が3回試行されており、データベースには3回分の認証失敗イベントエンティティが保存されている。
       | そのため、アカウントはロックアウト状態であると判定される。
   * - | (2)
     - | データベースには3回分の認証失敗イベントエンティティが保存されている。
       | しかしながら、過去10分以内の認証失敗イベントエンティティは2回分のみであるため、ロックアウト状態ではないと判定される。

同様に、ロックアウトを解除する場合の動作例を以下の図で説明する。

.. figure:: ./images/SecureLogin_unlock.png
   :alt: Account Lockout
   :width: 60%
   :align: center

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 過去10分以内に、誤ったパスワードでの認証が3回試行されている。
       | その後、認証失敗イベントエンティティが消去されているため、データベースには認証失敗イベントエンティティが保存されておらず、ロックアウト状態ではないと判定される。

コード解説
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* 共通部分

  本アプリケーションにおいて、アカウントのロックアウトに関する機能を実現するためには、データベースに対する認証失敗イベントエンティティの登録、検索、削除が共通的に必要となる。
  そのため、まずは認証失敗イベントエンティティに関するドメイン層・インフラストラクチャ層の実装を示す。

  * Entityの実装

    ユーザ名と認証試行日時を持つ認証失敗イベントエンティティの実装を以下に示す。

    .. code-block:: java

      package org.terasoluna.securelogin.domain.model;

      // omitted

      @Data
      public class FailedAuthentication implements Serializable {
        private static final long serialVersionUID = 1L;

        private String username; // (1)

        private LocalDateTime authenticationTimestamp; // (2)
      }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | 認証に使用したユーザ名
       * - | (2)
         - | 認証を試行した日時

  * Repositoryの実装

    認証失敗イベントエンティティの検索、登録、削除のためのRepositoryを以下に示す。

    .. code-block:: java

      package org.terasoluna.securelogin.domain.repository.authenticationevent;

      // omitted

      public interface FailedAuthenticationRepository {

        int create(FailedAuthentication event); // (1)

        List<FailedAuthentication> findLatest(@Param("username") String username,
                @Param("count") long count); // (2)

        int deleteByUsername(@Param("username") String username); // (3)
      }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | 引数として与えられた\ ``FailedAuthentication``\ オブジェクトをデータベースのレコードとして登録するメソッド
       * - | (2)
         - | 引数として与えられたユーザ名をキーとして、指定された個数の\ ``FailedAuthentication``\ オブジェクトを新しい順に取得するメソッド
       * - | (3)
         - | 引数として与えられたユーザ名をキーとして、認証失敗イベントエンティティのレコードを一括削除するメソッド

    マッピングファイルは以下の通り。

    .. code-block:: xml

      <?xml version="1.0" encoding="UTF-8"?>
      <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

      <mapper
        namespace="org.terasoluna.securelogin.domain.repository.authenticationevent.FailedAuthenticationRepository">

        <resultMap id="failedAuthenticationResultMap"
                type="FailedAuthentication">
                <id property="username" column="username" />
                <id property="authenticationTimestamp" column="authentication_timestamp" />
        </resultMap>

        <insert id="create" parameterType="FailedAuthentication">
          <![CDATA[
              INSERT INTO failed_authentication (
                  username,
                  authentication_timestamp
              ) VALUES (
                #{username},
                  #{authenticationTimestamp}
              )
          ]]>
        </insert>

        <select id="findLatest" resultMap="failedAuthenticationResultMap">
             <![CDATA[
                  SELECT
                      username,
                      authentication_timestamp
                  FROM
                      failed_authentication
                  WHERE
                      username = #{username}
                  ORDER BY authentication_timestamp DESC
                  LIMIT #{count}
             ]]>
        </select>

        <delete id="deleteByUsername">
           <![CDATA[
                DELETE FROM
                    failed_authentication
                WHERE
                    username = #{username}
           ]]>
        </delete>
      </mapper>

  * Serviceの実装

    作成したRepositoryのメソッドを呼び出すServiceを以下の通り定義する。

    .. code-block:: java

       package org.terasoluna.securelogin.domain.service.authenticationevent;

       // omitted

       @Service
       @Transactional
       public class AuthenticationEventSharedServiceImpl implements
               AuthenticationEventSharedService {

           // omitted

           @Inject
           ClassicDateFactory dateFactory;

           @Inject
           FailedAuthenticationRepository failedAuthenticationRepository;

           @Inject
           AccountSharedService accountSharedService;

           @Transactional(readOnly = true)
           @Override
           public List<FailedAuthentication> findLatestFailureEvents(
                           String username, int count) {
               return failedAuthenticationRepository.findLatestEvents(username, count);
           }


           @Transactional(propagation = Propagation.REQUIRES_NEW)
           @Override
           public void authenticationFailure(String username) { // (1)
               if (accountSharedService.exists(username)) {
                   FailedAuthentication failureEvents = new FailedAuthentication();
                   failureEvents.setUsername(username);
                   failureEvents.setAuthenticationTimestamp(dateFactory.newTimestamp()
                           .toLocalDateTime());

                   failedAuthenticationRepository.create(failureEvents);
               }
           }

           @Override
           public int deleteFailureEventByUsername(String username) {
               return failedAuthenticationRepository.deleteByUsername(username);
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
         - | 認証失敗イベントエンティティを作成してデータベースに登録するメソッド。
           | 引数として受け取ったユーザ名のアカウントが存在しない場合、データベースの外部キー制約に違反するため、データベースへの登録処理をスキップする。
           | 本メソッド実行後の例外により認証失敗イベントエンティティが登録されない可能性を考慮し、トランザクションの伝搬方法に\ ``REQUIRES_NEW`` \を指定している。

以下、実装方法に従って実装されたコードについて順に解説する。

* 認証失敗イベントエンティティの保存

  認証失敗時に発生するイベントをハンドリングして処理を行うために、\ ``@EventListener`` \アノテーションを使用する。
  \ ``@EventListener`` \アノテーションによるイベントのハンドリングについては :ref:`SpringSecurityAuthenticationEvent` を参照すること。

  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.account;

     // omitted

     @Component
     public class AccountAuthenticationFailureBadCredentialsEventListener{

         @Inject
         AuthenticationEventSharedService authenticationEventSharedService;

         @EventListener // (1)
         public void onApplicationEvent(
                         AuthenticationFailureBadCredentialsEvent event) {

             String username = (String) event.getAuthentication().getPrincipal(); // (2)

             authenticationEventSharedService.authenticationFailure(username); // (3)
         }

     }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | \ ``@EventListener`` \アノテーションを付与することで、誤ったパスワード等の不正な認証情報によって認証が失敗した際に、\ ``onApplicationEvent`` \メソッドが実行される。
     * - | (2)
       - | \ ``AuthenticationFailureBadCredentialsEvent`` \オブジェクトから、認証に使用したユーザ名を取得する。
     * - | (3)
       - | 認証失敗イベントエンティティを作成してデータベースに登録する処理を呼び出す。

* ロックアウト状態の判定

  認証失敗イベントエンティティを用いてアカウントのロックアウト状態を判定する処理を記述する。

  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.account;

     // omitted

     @Service
     @Transactional
     public class AccountSharedServiceImpl implements AccountSharedService {

         // omitted

         @Inject
         ClassicDateFactory dateFactory;

         @Inject
         AuthenticationEventSharedService authenticationEventSharedService;

         @Value("${security.lockingDurationSeconds}") // (1)
         int lockingDurationSeconds;

         @Value("${security.lockingThreshold}") // (2)
         int lockingThreshold;

         @Transactional(readOnly = true)
         @Override
         public boolean isLocked(String username) {
             List<FailedAuthentication> failureEvents = authenticationEventSharedService
                     .findLatestFailureEvents(username, lockingThreshold); // (3)

             if (failureEvents.size() < lockingThreshold) { // (4)
                 return false;
             }

             if (failureEvents
                     .get(lockingThreshold - 1) // (5)
                     .getAuthenticationTimestamp()
                     .isBefore(
                             dateFactory.newTimestamp().toLocalDateTime()
                                     .minusSeconds(lockingDurationSeconds))) {
                 return false;
             }

             return true;
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
       - | ロックアウトの継続時間を秒単位で指定する。プロパティファイルに定義された値をインジェクションしている。
     * - | (2)
       - | ロックアウトの閾値を指定する。ここで指定した回数だけ認証に失敗すると、アカウントがロックアウトされる。プロパティファイルに定義された値をインジェクションしている。
     * - | (3)
       - | 認証失敗イベントエンティティを、ロックアウトの閾値と同じ数だけ新しい順に取得する。
     * - | (4)
       - | 取得した認証失敗イベントエンティティの個数がロックアウトの閾値より小さい場合、ロックアウト状態ではないと判定する。
     * - | (5)
       - | 取得した認証失敗イベントエンティティのうち最も古い認証失敗時刻と現在時刻の差分が、ロックアウト継続時間よりも大きい場合には、ロックアウト状態ではないと判定する。

  | \ ``UserDetails`` \の実装クラスである\ ``org.springframework.security.core.userdetails.User`` \では、コンストラクタにロックアウト状態を渡すことができる。
  | 本アプリケーションでは以下のように\ ``User`` \を継承したクラスと、\ ``org.springframework.security.core.userdetails.UserDetailsService`` \を実装したクラスを用いる。

  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.userdetails;

     // omitted

     public class LoggedInUser extends User {

        // omitted

        private final Account account;

        public LoggedInUser(Account account, boolean isLocked,
                LocalDateTime lastLoginDate,
                List<SimpleGrantedAuthority> authorities) {
            super(account.getUsername(), account.getPassword(), true, true, true,
                    !isLocked, authorities); // (1)
            this.account = account;

            // omitted
        }

         public Account getAccount() {
             return account;
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
       - | 親クラスである\ ``User`` \のコンストラクタに **ロックアウト状態でないかどうか** を真理値で渡す。ロックアウト状態でない場合にtrueを渡す必要があることに注意する。

  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.userdetails;

     // omitted

     @Service
     public class LoggedInUserDetailsService implements UserDetailsService {

         @Inject
         AccountSharedService accountSharedService;

         @Transactional(readOnly = true)
         @Override
         public UserDetails loadUserByUsername(String username)
                 throws UsernameNotFoundException {
             try {
                Account account = accountSharedService.findOne(username);
                List<SimpleGrantedAuthority> authorities = new ArrayList<>();
                for (Role role : account.getRoles()) {
                    authorities.add(new SimpleGrantedAuthority("ROLE_"
                            + role.getRoleValue()));
                }
                return new LoggedInUser(account,
                        accountSharedService.isLocked(username), // (1)
                        accountSharedService.getLastLoginDate(username),
                        authorities);
             } catch (ResourceNotFoundException e) {
                 throw new UsernameNotFoundException("user not found", e);
             }
         }

     }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | \ ``LoggedInUser`` \のコンストラクタに、\ ``isLocked`` \メソッドによるロックアウト状態の判定結果を渡す。

  作成した\ ``UserDetailsService`` \を使用するための設定は以下の通り。

  **spring-security.xml**

  .. code-block:: xml

    <!-- omitted -->

    <sec:authentication-manager>
        <sec:authentication-provider
            user-service-ref="loggedInUserDetailsService"> <!-- (1) -->
            <sec:password-encoder ref="passwordEncoder" />
        </sec:authentication-provider>
    </sec:authentication-manager>

    <!-- omitted -->

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | \ ``UserDetailsService`` \のBeanのidを指定する。

* 認証失敗イベントエンティティの削除

  * 認証成功時の認証失敗イベントエンティティの削除

    連続した認証失敗のみをロックアウトの判定に使用するため、認証に成功した際にはアカウントの認証失敗イベントエンティティを削除する。
    共通部分として作成したServiceに、認証成功時に実行するメソッドを作成する。

    .. code-block:: java

       package org.terasoluna.securelogin.domain.service.authenticationevent;

       // omitted

       @Service
       @Transactional
       public class AuthenticationEventSharedServiceImpl implements
               AuthenticationEventSharedService {

           // omitted

           @Transactional(propagation = Propagation.REQUIRES_NEW)
           @Override
           public void authenticationSuccess(String username) {

               // omitted

               deleteFailureEventByUsername(username); // (1)
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
         - | 引数として渡されたユーザ名のアカウントに関する認証失敗イベントエンティティを削除する。


    認証成功時に発生するイベントをハンドリングして処理を行うために、 \ ``@EventListener`` \アノテーションを使用する。

    .. code-block:: java

       package org.terasoluna.securelogin.domain.service.account;

       // omitted

       @Component
       public class AccountAuthenticationSuccessEventListener{

           @Inject
           AuthenticationEventSharedService authenticationEventSharedService;

           @EventListener // (1)
           public void onApplicationEvent(
                           AuthenticationSuccessEvent event) {

               LoggedInUser details = (LoggedInUser) event.getAuthentication()
                       .getPrincipal();

               authenticationEventSharedService.authenticationSuccess(details.getUsername()); // (2)

           }

       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | \ ``@EventListener`` \アノテーションを付与することで、認証が成功した際に\ ``onApplicationEvent`` \メソッドが実行される。
       * - | (2)
         - | \ ``AuthenticationSuccessEvent`` \からユーザ名を取得し、認証失敗イベントエンティティを削除する処理を呼び出す。


  * ロックアウト状態の解除

    ロックアウト状態の判定に認証失敗イベントエンティティを使用しているため、認証失敗イベントエンティティを削除することでロックアウト状態を解除することができる。
    ロックアウト解除機能の使用を管理権限を持つユーザに限定するための認可の設定と、ドメイン層・アプリケーション層の実装を行う。

    * 認可の設定

      ロックアウトの解除を行うことができるユーザの権限を以下の通りに設定する。

      **spring-security.xml**

      .. code-block:: xml

        <!-- omitted -->

          <sec:http pattern="/resources/**" security="none" />
          <sec:http>

              <!-- omitted -->

              <sec:intercept-url pattern="/unlock/**" access="hasRole('ADMIN')" /> <!-- (1) -->

              <!-- omitted -->

          </sec:http>

        <!-- omitted -->

      .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
      .. list-table::
         :header-rows: 1
         :widths: 10 90

         * - 項番
           - 説明
         * - | (1)
           - | /unlock 以下のURLへのアクセス権限を管理ユーザに限定する。

    * Serviceの実装

      .. code-block:: java

         package org.terasoluna.securelogin.domain.service.unlock;

         // omitted

         @Transactional
         @Service
         public class UnlockServiceImpl implements UnlockService {

             @Inject
             AccountSharedService accountSharedService;

             @Inject
             AuthenticationEventSharedService authenticationEventSharedService;

             @Override
             public void unlock(String username) {
                 authenticationEventSharedService.deleteFailureEventByUsername(username); // (1)
             }

         }

      .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
      .. list-table::
         :header-rows: 1
         :widths: 10 90

         * - 項番
           - 説明
         * - | (1)
           - | 認証失敗イベントエンティティを消去することによりロックアウト状態を解除する。

    * Formの実装

      .. code-block:: java

        package org.terasoluna.securelogin.app.unlock;

        @Data
        public class UnlockForm implements Serializable {

            private static final long serialVersionUID = 1L;

            @NotEmpty
            private String username;
        }

    * Viewの実装

      **トップ画面(home.jsp)**

      .. code-block:: jsp

        <!-- omitted -->

        <body>
            <div id="wrapper">

                <!-- omitted -->

                <sec:authorize url="/unlock"> <!-- (1) -->
                <div>
                    <a id="unlock" href="${f:h(pageContext.request.contextPath)}/unlock?form">
                        Unlock Account
                    </a>
                </div>
                </sec:authorize>

                <!-- omitted -->

            </div>
        </body>

        <!-- omitted -->

      .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
      .. list-table::
         :header-rows: 1
         :widths: 10 90

         * - 項番
           - 説明
         * - | (1)
           - | /unlock 以下のアクセス権限を持つユーザに対してのみ表示する。

      **ロックアウト解除フォーム(unlokcForm.jsp)**

      .. code-block:: jsp

        <!-- omitted -->

        <body>
            <div id="wrapper">
                <h1>Unlock Account</h1>
                <t:messagesPanel />
                <form:form action="${f:h(pageContext.request.contextPath)}/unlock"
                    method="POST" modelAttribute="unlockForm">
                    <table>
                        <tr>
                            <th><form:label path="username" cssErrorClass="error-label">Username</form:label>
                            </th>
                            <td><form:input path="username" cssErrorClass="error-input" /></td>
                            <td><form:errors path="username" cssClass="error-messages" /></td>
                        </tr>
                    </table>

                    <input id="submit" type="submit" value="Unlock" />
                </form:form>
                <a href="${f:h(pageContext.request.contextPath)}/">go to Top</a>
            </div>
        </body>

        <!-- omitted -->

      **ロックアウト解除完了画面(unlockComplete.jsp)**

      .. code-block:: jsp

        <!-- omitted -->

        <body>
            <div id="wrapper">
                  <h1>${f:h(username)}'s account was successfully unlocked.</h1>
                  <a href="${f:h(pageContext.request.contextPath)}/">go to Top</a>
            </div>
        </body>

        <!-- omitted -->

    * Controllerの実装

      .. code-block:: java

         package org.terasoluna.securelogin.app.unlock;

         // omitted

         @Controller
         @RequestMapping("/unlock") // (1)
         public class UnlockController {

             @Inject
             UnlockService unlockService;

             @RequestMapping(params = "form")
             public String showForm(UnlockForm form) {
                 return "unlock/unlockForm";
             }

             @RequestMapping(method = RequestMethod.POST)
             public String unlock(@Validated UnlockForm form,
                     BindingResult bindingResult, Model model,
                     RedirectAttributes attributes) {
                 if (bindingResult.hasErrors()) {
                         return showForm(form);
                 }

                 try {
                     unlockService.unlock(form.getUsername()); // (2)
                     attributes.addFlashAttribute("username", form.getUsername());
                     return "redirect:/unlock?complete";
                 } catch (BusinessException e) {
                     model.addAttribute(e.getResultMessages());
                     return showForm(form);
                 }
             }

             @RequestMapping(method = RequestMethod.GET, params = "complete")
             public String unlockComplete() {
                 return "unlock/unlockComplete";
             }

         }

      .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
      .. list-table::
         :header-rows: 1
         :widths: 10 90

         * - 項番
           - 説明
         * - | (1)
           - | /unlock 以下のURLにマッピングする。認可の設定によって、管理ユーザのみがアクセス可能となる。
         * - | (2)
           - | Formから取得したユーザ名を引数として、アカウントのロックアウトを解除する処理を呼び出す。

.. _last-login:

最終ログイン日時の表示
--------------------------------------------------------------------------------
実装する要件一覧
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
* :ref:`前回ログイン日時の表示 <sec-requirements>`

動作イメージ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. figure:: ./images/SecureLogin_last_login.png
   :alt: Last Login Date
   :width: 80%
   :align: center

実装方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 本アプリケーションでは、認証に成功した履歴を「認証成功イベント」エンティティとしてデータベースに保存し、この認証成功イベントエンティティを用いて、トップ画面にアカウントの前回ログイン日時を表示する。
| 具体的には以下の二つの処理を実装することで、要件を実現する。

* 認証成功イベントエンティティの保存

  認証に成功した際にSpring Securityが発生させるイベントをハンドリングし、認証に使用したユーザ名と認証に成功した日時を認証成功イベントエンティティとしてデータベースに登録する。

* 前回ログイン日時の取得と表示

  認証時に、アカウントにおける最新の認証成功イベントエンティティをデータベースから取得し、イベントエンティティから認証成功日時を取得して\ ``org.springframework.security.core.userdetails.UserDetails`` \に設定する。
  jspに\ ``UserDetails`` \が保持している認証成功日時をフォーマットして渡し、表示する。

コード解説
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* 共通部分

  本アプリケーションにおいて、前回ログイン日時を表示するためには、データベースに対する認証成功イベントエンティティの登録、検索が必要となる。
  そのため、まずは認証成功イベントエンティティに関するドメイン層・インフラストラクチャ層の実装から解説を行う。

  * Entityの実装

    ユーザ名と認証成功日時を持つ認証成功イベントエンティティの実装は以下の通り。

    .. code-block:: java

       package org.terasoluna.securelogin.domain.model;

       // omitted

       @Data
       public class SuccessfulAuthentication implements Serializable {

           private static final long serialVersionUID = 1L;

           private String username; // (1)

           private LocalDateTime authenticationTimestamp; // (2)

       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | 認証に使用したユーザ名
       * - | (2)
         - | 認証を試行した日時

  * Repositoryの実装

    認証成功イベントエンティティの検索、登録を行うためのRepositoryを以下に示す。

    .. code-block:: java

       package org.terasoluna.securelogin.domain.repository.authenticationevent;

       // omitted

       public interface SuccessfulAuthenticationRepository {

           int create(SuccessfulAuthentication event); // (1)

           List<SuccessfulAuthentication> findLatest(
                  @Param("username") String username, @Param("count") long count); // (2)
       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | 引数として与えられた\ ``SuccessfulAuthentication``\ オブジェクトをデータベースのレコードとして登録するメソッド
       * - | (2)
         - | 引数として与えられたユーザ名をキーとして、指定された個数の\ ``SuccessfulAuthentication``\ オブジェクトを新しい順に取得するメソッド

    マッピングファイルは以下の通り。

    .. code-block:: xml

       <?xml version="1.0" encoding="UTF-8"?>
       <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

       <mapper
           namespace="org.terasoluna.securelogin.domain.repository.authenticationevent.SuccessfulAuthenticationRepository">

           <resultMap id="successfulAuthenticationResultMap"
                   type="SuccessfulAuthentication">
               <id property="username" column="username" />
               <id property="authenticationTimestamp" column="authentication_timestamp" />
           </resultMap>

           <insert id="create" parameterType="SuccessfulAuthentication">
           <![CDATA[
               INSERT INTO successful_authentication (
                   username,
                   authentication_timestamp
               ) VALUES (
                   #{username},
                   #{authenticationTimestamp}
               )
           ]]>
           </insert>

           <select id="findLatest" resultMap="successfulAuthenticationResultMap">
           <![CDATA[
               SELECT
                   username,
                   authentication_timestamp
               FROM
                   successful_authentication
               WHERE
                   username = #{username}
               ORDER BY authentication_timestamp DESC
               LIMIT #{count}
           ]]>
           </select>
       </mapper>

  * Serviceの実装

    作成したRepositoryのメソッドを呼び出すServiceを以下に示す。

    .. code-block:: java

       package org.terasoluna.securelogin.domain.service.authenticationevent;

       // omitted

       @Service
       @Transactional
       public class AuthenticationEventSharedServiceImpl implements
            AuthenticationEventSharedService {

           // omitted

           @Inject
           ClassicDateFactory dateFactory;

           @Inject
           SuccessfulAuthenticationRepository successAuthenticationRepository;

           @Transactional(readOnly = true)
           @Override
           public List<SuccessfulAuthentication> findLatestSuccessEvents(
                   String username, int count) {
               return successAuthenticationRepository.findLatest(username, count);
           }

           @Transactional(propagation = Propagation.REQUIRES_NEW)
           @Override
             public void authenticationSuccess(String username) {
                 SuccessfulAuthentication successEvent = new SuccessfulAuthentication();
                 successEvent.setUsername(username);
                 successEvent.setAuthenticationTimestamp(dateFactory.newTimestamp()
                         .toLocalDateTime());

                 successAuthenticationRepository.create(successEvent);
                 deleteFailureEventByUsername(username);
             }

       }

以下、実装方法に従って実装されたコードについて順に解説する。

* 認証成功イベントエンティティの保存

  認証成功時に発生するイベントをハンドリングして処理を行うために、\ ``@EventListener`` \アノテーションを使用する。

  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.account;

     // omitted

     @Component
     public class AccountAuthenticationSuccessEventListener{

         @Inject
         AuthenticationEventSharedService authenticationEventSharedService;

         @EventListener // (1)
         public void onApplicationEvent(AuthenticationSuccessEvent event) {
             LoggedInUser details = (LoggedInUser) event.getAuthentication()
                     .getPrincipal(); // (2)

             authenticationEventSharedService.authenticationSuccess(details
                     .getUsername()); // (3)
         }

     }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | \ ``@EventListener`` \アノテーションを付与することで、認証が成功した際に、\ ``onApplicationEvent`` \メソッドが実行される。
     * - | (2)
       - | \ ``AuthenticationSuccessEvent`` \オブジェクトから、\ ``UserDetails`` \の実装クラスを取得する。このクラスについては以降で説明する。
     * - | (3)
       - | 認証成功イベントエンティティを作成し、データベースに登録する処理を呼び出す。

* 前回ログイン日時の取得と表示

  認証成功イベントエンティティから前回ログイン日時を取得するためのServiceを以下に示す。

   .. code-block:: java

      package org.terasoluna.securelogin.domain.service.account;

      // omitted

      @Service
      @Transactional
      public class AccountSharedServiceImpl implements AccountSharedService {

          // omitted

          @Inject
          AuthenticationEventSharedService authenticationEventSharedService;

          @Transactional(readOnly = true)
          @Override
          public LocalDateTime getLastLoginDate(String username) {
              List<SuccessfulAuthentication> events = authenticationEventSharedService
                      .findLatestSuccessEvents(username, 1); // (1)

              if (events.isEmpty()) {
                  return null; // (2)
              } else {
                  return events.get(0).getAuthenticationTimestamp(); // (3)
              }
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
       - | 引数として与えられたユーザ名をキーとして、最新の認証成功イベントエンティティを一件取得する。
     * - | (2)
       - | 初回ログイン時等、認証成功イベントエンティティが一件も取得できない場合にはnullを返す。
     * - | (3)
       - | 認証成功イベントエンティティから、認証日時を取得して返す。

  ログイン時に前回ログイン日時を取得して\ ``UserDetails`` \に保持させるために、以下のように\ ``User`` \を継承したクラスと、\ ``UserDetailsService`` \を実装したクラスを作成する。

  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.userdetails;

     // omitted

     public class LoggedInUser extends User {

         private final Account account;

         private final LocalDateTime lastLoginDate; // (1)

         public LoggedInUser(Account account, boolean isLocked,
                 LocalDateTime lastLoginDate,
                 List<SimpleGrantedAuthority> authorities) {

             super(account.getUsername(), account.getPassword(), true, true, true,
                     !isLocked, authorities);
             this.account = account;
             this.lastLoginDate = lastLoginDate; // (2)
         }

         // omitted

         public LocalDateTime getLastLoginDate() { // (3)
             return lastLoginDate;
         }

     }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | 前回ログイン日時を保持するためのフィールドを宣言する。
     * - | (2)
       - | 引数として与えられた前回ログイン日時をフィールドに設定する。
     * - | (3)
       - | 保持している前回ログイン日時を返すメソッド

  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.userdetails;

     // omitted

     @Service
     public class LoggedInUserDetailsService implements UserDetailsService {

         @Inject
         AccountSharedService accountSharedService;

         @Transactional(readOnly = true)
         @Override
         public UserDetails loadUserByUsername(String username)
                 throws UsernameNotFoundException {
             try {
                 Account account = accountSharedService.findOne(username);
                 List<SimpleGrantedAuthority> authorities = new ArrayList<>();
                 for (Role role : account.getRoles()) {
                         authorities.add(new SimpleGrantedAuthority("ROLE_"
                                + role.getRoleValue()));
                 }
                 return new LoggedInUser(account,
                         accountSharedService.isLocked(username),
                         accountSharedService.getLastLoginDate(username), // (1)
                         authorities);
             } catch (ResourceNotFoundException e) {
                 throw new UsernameNotFoundException("user not found", e);
             }
         }

     }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | Serviceのメソッドを呼び出して前回ログイン日時を取得し、\ ``LoggedInUser`` \のコンストラクタに渡す。

  トップ画面に前回ログイン日時を表示するためのアプリケーション層の実装を行う。

  .. code-block:: java

     package org.terasoluna.securelogin.app.welcome;

     // omitted

     @Controller
     public class HomeController {

        @Inject
        AccountSharedService accountSharedService;

        @RequestMapping(value = "/", method = { RequestMethod.GET,
                RequestMethod.POST })
        public String home(@AuthenticationPrincipal LoggedInUser userDetails, // (1)
                Model model) {

            // omitted

            LocalDateTime lastLoginDate = userDetails.getLastLoginDate(); // (2)
            if (lastLoginDate != null) {
                model.addAttribute("lastLoginDate", lastLoginDate
                        .format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"))); // (3)
            }

            return "welcome/home";

        }

     }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | \ ``@AuthenticationPrincipal`` \を使用してUserDetailsオブジェクトを取得する。
     * - | (2)
       - | \ ``LoggedInUserDetails`` \から最終ログイン日時を取得する。
     * - | (3)
       - | 最終ログイン日時をフォーマットしてModelに設定し、Viewに渡す。

  **トップ画面(home.jsp)**

  .. code-block:: jsp

    <body>
      <div id="wrapper">

          <!-- omitted -->

          <c:if test="${!empty lastLoginDate}"> <!-- (1) -->
              <p id="lastLogin">
                  Last login date is ${f:h(lastLoginDate)}. <!-- (2) -->
              </p>
          </c:if>

          <!-- omitted -->

      </div>
    </body>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | 前回ログイン日時がnullの場合は表示しない。
     * - | (2)
       - | Controllerから渡された前回ログイン日時を表示する。

.. _reissue-info-create:

パスワード再発行のための認証情報の生成
--------------------------------------------------------------------------------
実装する要件一覧
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
* :ref:`パスワード再発行用URLへのランダム文字列の付与 <sec-requirements>`
* :ref:`パスワード再発行用秘密情報の発行 <sec-requirements>`

動作イメージ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. figure:: ./images/SecureLogin_password_reissue_generate.png
   :alt: Generate Password Reissue Information
   :width: 80%
   :align: center

パスワード再発行のための認証情報生成画面で、パスワードを再発行するユーザ名を入力する。このとき、パスワード再発行時の認証に使用する秘密情報と、トークンが生成される。
秘密情報は画面に表示され、トークンを含んだパスワード再発行画面のURLはユーザの登録済みメールアドレスに送付される。

メール送付されたURLには有効期限があり、有効期限内にアクセスして秘密情報と新しいパスワードを入力することで、パスワードを変更することができる。
有効期限が切れた後にメール送付されたURLにアクセスした場合、エラー画面に遷移する。

ここでは、上記の流れのうち、秘密情報とトークンの生成について説明を行う。

実装方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| パスワードの再発行を行う際には、ユーザがアカウントの所有者であることを確認するためのパスワードに代わる手段が必要である。
| 本アプリケーションでは、ユーザを確認するための情報として、パスワード再発行画面のURLと秘密情報の二つを用いる。
| パスワード再発行画面のURLを一意かつ推測困難にするために、ランダムな文字列を生成しURLに付加する。万が一URLが漏えいした場合に備え、ランダムな文字列である秘密情報を生成し、これを用いて認証を行う。
| 二つのランダムな文字列は、片方からもう一方を推測することが不可能となるように、それぞれ異なる方法で生成する。
| 具体的には以下の処理を実装することで要件を実現する。

* パスワード再発行のための認証情報の生成と保存

  以下の4つの情報を、パスワード再発行のための認証情報としてデータベースに保存する。

  * ユーザ名：パスワードを再発行するアカウントのユーザ名
  * トークン：パスワード再発行画面のURLを、一意かつ推測不能にするために生成するランダムな文字列
  * 秘密情報：パスワード再発行時にユーザに入力させるために生成するランダムな文字列
  * 有効期限：パスワード再発行のための認証情報の有効期限

  トークンの生成には\ ``java.util.UUID`` \クラスの\ ``randomUUID`` \メソッドを用い、秘密情報の生成にはPassayのパスワード生成機能を用いる。

  秘密情報については、パスワードと同様にハッシュ化してデータベースへ保存する。
  有効期限の設定と確認処理については、:ref:`パスワード再発行実行時の検査 <reissue-info-validate>` に記す。
  パスワード再発行のための認証情報をユーザに配布する方法については、:ref:`パスワード再発行のための認証情報の配布 <reissue-info-delivery>` を参照。

コード解説
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* 共通部分

  上記の実装方法に従って実装を進める上で、パスワード再発行のための認証情報をデータベースに登録、検索する処理が共通的に必要となる。
  そのため、まずはパスワード再発行のための認証情報に関連するEntityとRepositoryの実装から解説する。

  * Entityの作成

    パスワード再発行のための認証情報のEntityを作成する。

    .. code-block:: java

       package org.terasoluna.securelogin.domain.model;

       // omitted

       @Data
       public class PasswordReissueInfo {

           private String username; // (1)

           private String token; // (2)

           private String secret; // (3)

           private LocalDateTime expiryDate; // (4)

       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | パスワード再発行対象のユーザ名
       * - | (2)
         - | パスワード再発行用URLに含めるために生成される文字列（トークン）
       * - | (3)
         - | パスワード再発行時にユーザを確認するための文字列（秘密情報）
       * - | (2)
         - | パスワード再発行のための認証情報の有効期限

  * Repositoryの実装

    パスワード再発行のための認証情報の検索、登録、削除を行うためのRepositoryを以下に示す。

    .. code-block:: java

       package org.terasoluna.securelogin.domain.repository.passwordreissue;

       // omitted

       public interface PasswordReissueInfoRepository {

           void create(PasswordReissueInfo info); // (1)

           PasswordReissueInfo findOne(@Param("token") String token); // (2)

           int delete(@Param("token") String token); // (3)

           // omitted

       }

   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - 項番
        - 説明
      * - | (1)
        - | 引数として与えられた\ ``PasswordReissueInfo``\ オブジェクトをデータベースのレコードとして登録するメソッド
      * - | (2)
        - | 引数として与えられたトークンをキーとして、\ ``PasswordReissueInfo``\ オブジェクトを検索し、取得するメソッド
      * - | (3)
        - | 引数として与えられたトークンをキーとして、\ ``PasswordReissueInfo``\ オブジェクトを削除するメソッド

   マッピングファイルは以下の通り。

   .. code-block:: xml

      <?xml version="1.0" encoding="UTF-8"?>
      <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

      <mapper
          namespace="org.terasoluna.securelogin.domain.repository.passwordreissue.PasswordReissueInfoRepository">

          <resultMap id="PasswordReissueInfoResultMap" type="PasswordReissueInfo">
              <id property="username" column="username" />
              <id property="token" column="token" />
              <id property="secret" column="secret" />
              <id property="expiryDate" column="expiry_date" />
          </resultMap>

          <select id="findOne" resultMap="PasswordReissueInfoResultMap">
          <![CDATA[
              SELECT
                  username,
                  token,
                  secret,
                  expiry_date
              FROM
                  password_reissue_info
              WHERE
                  token = #{token}
          ]]>
          </select>

          <insert id="create" parameterType="PasswordReissueInfo">
          <![CDATA[
              INSERT INTO password_reissue_info (
                  username,
                  token,
                  secret,
                  expiry_date
              ) VALUES (
                  #{username},
                  #{token},
                  #{secret},
                  #{expiryDate}
              )
          ]]>
          </insert>

          <delete id="delete">
          <![CDATA[
              DELETE FROM
                  password_reissue_info
              WHERE
                  token = #{token}
          ]]>
          </delete>

          <!-- omitted -->

      </mapper>

以下、実装方法に従って実装されたコードについて順に解説する。

* パスワード再発行のための認証情報の生成と保存

  * パスワード生成器の定義

    Passayのパスワード生成機能を使用するための、パスワード生成器と生成規則の定義を以下に示す。
    パスワード生成器や生成規則に関しては :ref:`password_generation` を参照。

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | Passayのパスワード生成機能で用いるパスワード生成器のBean定義
       * - | (2)
         - | Passayのパスワード生成機能で用いるパスワード生成規則のBean定義。 :ref:`password-strength` で使用した検証規則を使用し、半角英大文字、半角英小文字、半角数字をそれぞれ一文字以上含むパスワードの生成規則を定義する。

    **applicationContext.xml**

    .. code-block:: xml

       <bean id="passwordGenerator" class="org.passay.PasswordGenerator" /> <!-- (1) -->
       <util:list id="passwordGenerationRules">
           <ref bean="upperCaseRule" />
           <ref bean="lowerCaseRule" />
           <ref bean="digitRule" />
       </util:list>

  * Serviceの実装

    パスワード再発行のための認証情報を作成し、データベースへ保存するための処理の実装を以下に示す。この処理中で生成した認証情報をメール送信する。メール送信については後述するため、ここでは省略する。

    .. code-block:: java

       package org.terasoluna.securelogin.domain.service.passwordreissue;

       // omitted

       @Service
       @Transactional
       public class PasswordReissueServiceImpl implements PasswordReissueService {

           @Inject
           ClassicDateFactory dateFactory;

           @Inject
           PasswordReissueInfoRepository passwordReissueInfoRepository;

           @Inject
           AccountSharedService accountSharedService;

           @Inject
           PasswordEncoder passwordEncoder;

           @Inject
           PasswordGenerator passwordGenerator; // (1)

           @Resource(name = "passwordGenerationRules")
           List<CharacterRule> passwordGenerationRules; //(2)

           @Value("${security.tokenLifeTimeSeconds}")
           int tokenLifeTimeSeconds; // (3)

           // omitted

           @Override
           public String createAndSendReissueInfo(String username) {

               String rowSecret = passwordGenerator.generatePassword(10, passwordGenerationRules); // (4)

               if (!accountSharedService.exists(username)) { // (5)
                   return rowSecret;
               }

               Account account= accountSharedService.findOne(username); // (6)

               String token = UUID.randomUUID().toString(); // (7)

               LocalDateTime expiryDate = dateFactory.newTimestamp().toLocalDateTime()
                       .plusSeconds(tokenLifeTimeSeconds); // (8)

               PasswordReissueInfo info = new PasswordReissueInfo(); // (9)
               info.setUsername(username);
               info.setToken(token);
               info.setSecret(passwordEncoder.encode(rowSecret)); // (10)
               info.setExpiryDate(expiryDate);

               passwordReissueInfoRepository.create(info); // (11)

               // omitted (Send E-Mail)

               return rowSecret; // (12)

           }

           // omitted

       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90
       :class: longtable

       * - 項番
         - 説明
       * - | (1)
         - | Passayのパスワード生成機能で用いるパスワード生成器をインジェクションする。
       * - | (2)
         - | Passayのパスワード生成機能で用いるパスワード生成ルールをインジェクションする。
       * - | (3)
         - | パスワード再発行用の認証情報が有効である期間の長さを秒単位で指定する。プロパティファイルに定義された値をインジェクションしている。
       * - | (4)
         - | 秘密情報として用いるために、Passayのパスワード生成機能を用いて、パスワード生成規則に従った、長さ10のランダムな文字列を生成する。
       * - | (5)
         - | 引数として渡されてきたユーザ名のアカウントが存在するかどうか確認する。存在しなかった場合、ユーザが存在しないことを知られないためにダミーの秘密情報を返す。
       * - | (6)
         - | パスワード再発行用の認証情報に含まれるユーザ名のアカウント情報を取得する。
       * - | (7)
         - | トークンとして用いるために、\ ``java.util.UUID`` \クラスの\ ``randomUUID`` \メソッドを用いてランダムな文字列を生成する。
       * - | (8)
         - | 現在時刻に(3)の値を加えることにより、パスワード再発行用の認証情報の有効期限を計算する。
       * - | (9)
         - | パスワード再発行用の認証情報を作成し、ユーザ名、トークン、秘密情報、有効期限を設定する。
       * - | (10)
         - | 秘密情報はハッシュ化を行ってから\ ``PasswordReissueInfo`` \に設定する。
       * - | (11)
         - | パスワード再発行用の認証情報をデータベースに登録する。
       * - | (12)
         - | 生成した秘密情報を返す。

    .. raw:: latex

       \newpage

  * Formの実装

    .. code-block:: java

       package org.terasoluna.securelogin.app.passwordreissue;

       // omitted

       @Data
       public class CreateReissueInfoForm implements Serializable {

           private static final long serialVersionUID = 1L;

           @NotEmpty
           private String username;
       }

  * Viewの実装

    **パスワード再発行のための認証情報生成画面(createReissueInfoForm.xml)**

    .. code-block:: jsp

       <!-- omitted -->

       <body>
           <div id="wrapper">
               <h1>Reissue password</h1>
               <t:messagesPanel />
               <form:form
                   action="${f:h(pageContext.request.contextPath)}/reissue/create"
                   method="POST" modelAttribute="createReissueInfoForm">
                   <table>
                       <tr>
                           <th><form:label path="username" cssErrorClass="error-label">Username</form:label>
                           </th>
                           <td><form:input path="username" cssErrorClass="error-input" /></td>
                           <td><form:errors path="username" cssClass="error-messages" /></td>
                       </tr>
                   </table>

                   <input id="submit" type="submit" value="Reissue password" />
               </form:form>
           </div>
       </body>

       <!-- omitted -->

  * Controllerの実装

    .. code-block:: java

       package org.terasoluna.securelogin.app.passwordreissue;

       // omitted

       @Controller
       @RequestMapping("/reissue")
       public class PasswordReissueController {

           @Inject
           PasswordReissueService passwordReissueService;

           @RequestMapping(value = "create", params = "form")
           public String showCreateReissueInfoForm(CreateReissueInfoForm form) {
               return "passwordreissue/createReissueInfoForm";
           }

           @RequestMapping(value = "create", method = RequestMethod.POST)
           public String createReissueInfo(@Validated CreateReissueInfoForm form,
                   BindingResult bindingResult, Model model,
                   RedirectAttributes attributes) {
               if (bindingResult.hasErrors()) {
                   return showCreateReissueInfoForm(form);
               }

               String rawSecret = passwordReissueService.createAndSendReissueInfo(form.getUsername()); // (1)
               attributes.addFlashAttribute("secret", rawSecret);
               return "redirect:/reissue/create?complete";
           }

           @RequestMapping(value = "create", params = "complete", method = RequestMethod.GET)
           public String createReissueInfoComplete() {
               return "passwordreissue/createReissueInfoComplete";
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
         - | Formから取得したユーザ名から、パスワード再発行のための認証情報を生成し、データベースに登録する処理を呼び出す。


.. _reissue-info-delivery:

パスワード再発行のための認証情報の配布
--------------------------------------------------------------------------------
実装する要件一覧
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
* :ref:`パスワード再発行画面のURLと秘密情報の別配布 <sec-requirements>`
* :ref:`パスワード再発行画面のURLのメール送付 <sec-requirements>`

動作イメージ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. figure:: ./images/SecureLogin_password_reissue_give.png
   :alt: Givee Password Reissue Information
   :width: 80%
   :align: center

:ref:`reissue-info-create` では、パスワード再発行のための認証情報の生成について説明した。
ここでは、生成した認証情報の配布について説明する。

パスワード再発行のための認証は、パスワード再発行画面のURLと秘密情報を用いて行う。
この二つの情報が一度に漏れることを防ぐため、それぞれ別の方法でユーザに配布する。
本アプリケーションでは、パスワード再発行画面のURLはユーザの登録済みメールアドレスへ送付し、秘密情報は画面に表示する。

実装方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| :ref:`パスワード再発行のための認証情報の生成 <reissue-info-create>` で生成した認証情報を二つに分け、それぞれ別の方法でユーザに配布する。
| 以下の二つの処理を実装して用いることで要件を実現する。

* 秘密情報の画面表示

  :ref:`パスワード再発行のための認証情報の生成 <reissue-info-create>` で生成したハッシュ化前の秘密情報を、画面に表示させることでユーザに配布する。

* パスワード再発行画面のURLのメール送付

  :ref:`パスワード再発行のための認証情報の生成 <reissue-info-create>` で生成したトークンを含むパスワード再発行画面のURLを、Spring FrameworkのMail連携用コンポーネントを用いて、メールで送付する。


コード解説
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

上記の実装方法に従って実装されたコードについて順に解説する。

* 秘密情報の画面表示

  Controllerから秘密情報の生成処理を呼び出し、Viewに表示するための一連の実装を以下に示す。

  .. code-block:: java

     package org.terasoluna.securelogin.app.passwordreissue;

     // omitted

     @Controller
     @RequestMapping("/reissue")
     public class PasswordReissueController {

         @Inject
         PasswordReissueService passwordReissueService;

         // omitted

         @RequestMapping(value = "create", method = RequestMethod.POST)
         public String createReissueInfo(@Validated CreateReissueInfoForm form,
                 BindingResult bindingResult, Model model,
                 RedirectAttributes attributes) {
             if (bindingResult.hasErrors()) {
                 return showCreateReissueInfoForm(form);
             }

             String rawSecret = passwordReissueService.createAndSendReissueInfo(form
                     .getUsername()); // (1)
             attributes.addFlashAttribute("secret", rawSecret); // (2)
             return "redirect:/reissue/create?complete"; // (3)
         }

         @RequestMapping(value = "create", params = "complete", method = RequestMethod.GET)
         public String createReissueInfoComplete() {
             return "passwordreissue/createReissueInfoComplete";
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
       - | 秘密情報を生成する処理を呼び出す。
     * - | (2)
       - | RedirectAttributesを利用して、リダイレクト先に秘密情報を渡す。
     * - | (3)
       - | パスワード再発行用の認証情報完了画面にリダイレクトする。


  **パスワード再発行用の認証情報生成完了画面(createReissueInfoComplete.jsp)**

  .. code-block:: jsp

     <!-- omitted -->

     <body>
         <div id="wrapper">
             <h1>Your Password Reissue URL was successfully generated.</h1>
             The URL was sent to your registered E-mail address.<br /> Please
             access the URL and enter the secret shown below.
             <h3>Secret : <span id=secret>${f:h(secret)}</span></h3> <!-- (1) -->
         </div>
     </body>

     <!-- omitted -->

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | 秘密情報を画面に表示する。

* パスワード再発行画面のURLのメール送付

  パスワード再発行用の認証情報からパスワード再発行画面のURLを作成し、メール送付する処理の実装を以下に示す。
  依存ライブラリの追加方法やメールセッションの取得方法等の詳細については、:doc:`../ArchitectureInDetail/MessagingDetail/Email` を参照。

  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.mail;

     // omitted

     @Service
     public class PasswordReissueMailSharedServiceImpl implements
             PasswordReissueMailSharedService {

         @Inject
         JavaMailSender mailSender; // (1)

         @Inject
         @Named("passwordReissueMessage")
         SimpleMailMessage templateMessage; // (2)

         // omitted

         @Override
         public void send(String to, String text) {
             SimpleMailMessage message = new SimpleMailMessage(templateMessage); // (3)
             message.setTo(to);
             message.setText(text);
             mailSender.send(message);
         }

     }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | \ ``org.springframework.mail.javamail.JavaMailSender`` \のBeanをインジェクションする。
     * - | (2)
       - | 送信元のメールアドレスとメールタイトルが設定された、\ ``org.springframework.mail.SimpleMailMessage`` \のBeanをインジェクションする。
         | 本アプリケーションでは\ ``SimpleMailMessage`` \ のBeanは一つしか定義されていないが、一般にはメールのテンプレートとして複数のBeanが定義されるため、 \ ``@Named`` \でBean名を指定している。
     * - | (3)
       - | テンプレートから\ ``SimpleMailMessage`` \ のインスタンスを生成し、引数として与えられた宛先メールアドレスと本文を設定して送信する。

  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.passwordreissue;

     // omitted

     @Service
     @Transactional
     public class PasswordReissueServiceImpl implements PasswordReissueService {

         @Inject
         ClassicDateFactory dateFactory;

         @Inject
         PasswordReissueMailSharedService mailSharedService;

         @Inject
         AccountSharedService accountSharedService;

         @Inject
         PasswordEncoder passwordEncoder;

         @Value("${security.tokenLifeTimeSeconds}")
         int tokenLifeTimeSeconds;

         @Value("${app.applicationBaseUrl}") // (1)
         String baseUrl;

         @Value("${app.passwordReissueProtocol}")
         String protocol;

         // omitted

         @Override
         public String createAndSendReissueInfo(String username) {

             String rowSecret = passwordGenerator.generatePassword(10,
                     passwordGenerationRules);

             if (!accountSharedService.exists(username)) {
                 return rowSecret;
             }

             Account account= accountSharedService.findOne(username);

             String token = UUID.randomUUID().toString();

             LocalDateTime expiryDate = dateFactory.newTimestamp().toLocalDateTime()
                     .plusSeconds(tokenLifeTimeSeconds);

             PasswordReissueInfo info = new PasswordReissueInfo();
             info.setUsername(username);
             info.setToken(token);
             info.setSecret(passwordEncoder.encode(rowSecret));
             info.setExpiryDate(expiryDate);

             passwordReissueInfoRepository.create(info);

             UriComponentsBuilder uriBuilder = UriComponentsBuilder.fromUriString(baseUrl);
             uriBuilder.pathSegment("reissue").pathSegment("resetpassword")
                     .queryParam("form").queryParam("token", info.getToken());  // (2)
             String passwordResetUrl = uriBuilder.build().encode().toUriString();

             mailSharedService.send(account.getEmail(), passwordResetUrl); // (3)

             return rowSecret;

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
       - | パスワード再発行画面のURLに使用するベースURLをプロパティファイルから取得する。
     * - | (2)
       - | (1)で取得した値と、生成したパスワード再発行用の認証情報に含まれるトークンを使用して、ユーザに配布するパスワード再発行画面のURLを作成する。
         | URLの作成には \ ``org.springframework.web.util.UriComponentsBuilder`` \ を利用する。\ ``UriComponentsBuilder`` \ については、:ref:`RESTAppendixHyperMediaLink` の中で説明されている。
     * - | (3)
       - | ユーザの登録メールアドレス宛てに、パスワード再発行画面のURLを本文に記したメールを送付する。


.. _reissue-info-validate:

パスワード再発行実行時の検査
--------------------------------------------------------------------------------
実装する要件一覧
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
* :ref:`パスワード再発行用の認証情報の有効期限の設定 <sec-requirements>`

動作イメージ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. figure:: ./images/SecureLogin_password_reissue_execute.png
   :alt: Use Password Reissue Information
   :width: 80%
   :align: center

:ref:`reissue-info-delivery` では、パスワード再発行のための認証情報の配布について説明した。
ここでは、配布された認証情報を使用する際の処理について説明する。

パスワード再発行時の認証として、:ref:`reissue-info-delivery` でそれぞれ別配布したパスワード再発行画面のURLと秘密情報を照合する。
URLに含まれるトークンと秘密情報の組が正しい場合にのみ、パスワードが再発行される。

また、一般的にはパスワードの再発行は認証情報の生成から間を置かずに行われるため、不必要に長期間有効となることが無いように、認証情報に有効期限を設定する。
パスワード再発行画面のURLにアクセスした際に、認証情報が有効期限内であればパスワード再発行画面を表示し、有効期限が切れていればエラー画面に遷移する。

実装方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| メールで送付されるパスワード再発行画面のURLには、リクエストパラメータとしてトークンが含まれている。パスワード再発行画面へアクセスされた際にトークンを取得し、このトークンをキーとしてデータベースからパスワード再発行のための認証情報を検索する。
| 認証情報生成時にあらかじめ有効期限を設定しておき、データベースから取得した際に有効期限切れのチェックを行う。有効期限内であればパスワード変更画面を表示して秘密情報と新しいパスワードの入力を受け付ける。
| データベースから取得した認証情報中の秘密情報と、ユーザが入力した秘密情報が一致すれば認証成功であり、パスワードを再発行する。
| 具体的には以下の三つの処理を実装することで要件を実現する。

* パスワード再発行用の認証情報の有効期限の設定

  :ref:`reissue-info-create` で説明した処理の中で、生成した認証情報に有効期限を設定する。

* パスワード再発行のための認証情報の有効期限の検査

  パスワード再発行画面にアクセスされた際に、リクエストパラメータに含まれるトークンを取得し、トークンをキーとしてデータベースに保存されているパスワード再発行のための認証情報を検索する。
  認証情報に含まれる有効期限と現在時刻を比較し、有効期限が切れていればエラー画面に遷移させる。

* パスワード再発行のための認証情報を用いたユーザの確認

  パスワードの再発行を行う際に、ユーザ名、トークンとユーザが入力した秘密情報の組み合わせがデータベース内の認証情報と一致しているかどうかを確認する。
  一致する場合にはパスワードを再発行し、不一致の場合にはエラーメッセージを表示する。


コード解説
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* パスワード再発行用の認証情報の有効期限の設定

  パスワード再発行用の認証情報への有効期限の設定自体は、 :ref:`reissue-info-create` で説明した処理に含まれている。ここでは、関連する実装箇所のみ再掲する。

  * Serviceの実装

    .. code-block:: java

       package org.terasoluna.securelogin.domain.service.passwordreissue;

       // omitted

       @Service
       @Transactional
       public class PasswordReissueServiceImpl implements PasswordReissueService {

           @Inject
           ClassicDateFactory dateFactory;

           @Inject
           PasswordReissueInfoRepository passwordReissueInfoRepository;

           @Value("${security.tokenLifeTimeSeconds}")
           int tokenLifeTimeSeconds; // (1)

           // omitted

           @Override
           public String createAndSendReissueInfo(String username) {

               // omitted

               LocalDateTime expiryDate = dateFactory.newTimestamp().toLocalDateTime()
                       .plusSeconds(tokenLifeTimeSeconds); // (2)

               PasswordReissueInfo info = new PasswordReissueInfo(); // (3)
               info.setUsername(username);
               info.setToken(token);
               info.setSecret(passwordEncoder.encode(rowSecret));
               info.setExpiryDate(expiryDate);

               passwordReissueInfoRepository.create(info); // (4)

               // omitted (Send E-Mail)

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
         - | パスワード再発行用の認証情報が有効である期間の長さを秒単位で指定する。プロパティファイルに定義された値をインジェクションしている。
       * - | (2)
         - | 現在時刻に(1)の値を加えることにより、パスワード再発行用の認証情報の有効期限を計算する。
       * - | (3)
         - | パスワード再発行用の認証情報を作成し、ユーザ名、トークン、秘密情報、有効期限を設定する。
       * - | (4)
         - | パスワード再発行用の認証情報をデータベースに登録する。

* パスワード再発行のための認証情報の有効期限の検査

  パスワード再発行画面にアクセスされた際に、リクエストパラメータとしてURLに含まれるトークンからパスワード再発行のための認証情報を取得し、有効期限内であるかどうかを検査する処理の実装を以下に示す。
  この処理中ではパスワード再発行の失敗上限を超過しているかどうかの検査も行うが、後述するため、ここでは省略する。

  * Serviceの実装

    .. code-block:: java

       package org.terasoluna.securelogin.domain.service.passwordreissue;

       // omitted

       @Service
       @Transactional
       public class PasswordReissueServiceImpl implements PasswordReissueService {

           @Inject
           ClassicDateFactory dateFactory;

           @Inject
           PasswordReissueInfoRepository passwordReissueInfoRepository;

           // omitted

           @Override
           @Transactional(readOnly = true)
           public PasswordReissueInfo findOne(String token) {
               PasswordReissueInfo info = passwordReissueInfoRepository.findOne(token); // (1)

               if (info == null) {
                   throw new ResourceNotFoundException(ResultMessages.error().add(
                           MessageKeys.E_SL_PR_5002, token));
               }

               if (dateFactory.newTimestamp().toLocalDateTime()
                        .isAfter(info.getExpiryDate())) { // (2)
                   throw new BusinessException(ResultMessages.error().add(
                           MessageKeys.E_SL_PR_2001));
               }

               // omitted (attempts exceeded upper bounds)

               return info;
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
         - | 引数として与えられたトークンをキーとして、パスワード再発行のための認証情報をデータベースから取得する。
       * - | (2)
         - | 有効期限が切れている場合は、\ ``org.terasoluna.gfw.common.exception.BusinessException`` \をthrowする。

  * Controllerの実装

    .. code-block:: java

       package org.terasoluna.securelogin.app.passwordreissue;

       // omitted

       @Controller
       @RequestMapping("/reissue")
       public class PasswordReissueController {

           @Inject
           PasswordReissueService passwordReissueService;

           // omitted

           public String showPasswordResetForm(PasswordResetForm form, Model model,
                   @RequestParam("token") String token) { // (1)

               PasswordReissueInfo info = passwordReissueService.findOne(token); // (3)

               form.setUsername(info.getUsername());
               form.setToken(token);
               model.addAttribute("passwordResetForm", form);
               return "passwordreissue/passwordResetForm";
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
         - | パスワード再発行画面のURLにリクエストパラメータとして含まれるトークンを取得する。
       * - | (2)
         - | Serviceのメソッドにトークンを渡して呼び出す。データベースから認証情報が取得され、有効期限が検査される。

* パスワード再発行のための認証情報を用いたユーザの確認

  パスワード再発行画面においてユーザが入力した秘密情報と、パスワード再発行画面のURLに含まれるトークンの組が正しいかどうかを確認する処理の実装を以下に示す。
  この確認処理はパスワード再発行固有のロジックであり、かつデータベースの内容によって結果が異なるチェックであることから、Bean ValidationやSpring Validatorを用いず、Serviceに実装している。

  * Serviceの実装

    .. code-block:: java

       package org.terasoluna.securelogin.domain.service.passwordreissue;

       // omitted

       public interface PasswordReissueService {

           // omitted

           boolean resetPassword(String username, String token, String secret, // (1)
                   String rawPassword);

           // omitted

       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | 引数として与えられたユーザ名、トークン、秘密情報を用いてユーザの確認を行った後、新しいパスワードを設定するメソッド


    .. code-block:: java

       package org.terasoluna.securelogin.domain.service.passwordreissue;

       // omitted

       @Service
       @Transactional
       public class PasswordReissueServiceImpl implements PasswordReissueService {

           @Inject
           PasswordReissueFailureSharedService passwordReissueFailureSharedService;

           @Inject
           PasswordReissueInfoRepository passwordReissueInfoRepository;

           @Inject
           AccountSharedService accountSharedService;

           @Inject
           PasswordEncoder passwordEncoder;

           // omitted

           @Override
           public boolean resetPassword(String username, String token, String secret,
                   String rawPassword) {
               PasswordReissueInfo info = this.findOne(token); // (1)
               if (!passwordEncoder.matches(secret, info.getSecret())) { // (2)
                   passwordReissueFailureSharedService.resetFailure(username, token);
                   throw new BusinessException(ResultMessages.error().add(
                       MessageKeys.E_SL_PR_5003));
               }
               failedPasswordReissueRepository.deleteByToken(token);
               passwordReissueInfoRepository.delete(token); // (3)

               return accountSharedService.updatePassword(username, rawPassword); // (4)

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
         - | 引数として与えられたトークンを用いて、データベースからパスワード再発行用の認証情報を取得する。このとき、有効期限が改めて検査される。
       * - | (2)
         - | パスワード再発行用の認証情報に含まれるハッシュ化された秘密情報と、引数として与えられた秘密情報を比較する。異なる場合には\ ``BusinessException`` \をthrowする。この場合、パスワードの再発行は失敗となる。
       * - | (3)
         - | 使用された認証情報を再使用不能にするために、データベースから消去する。
       * - | (4)
         - | 引数として渡されたユーザ名を持つアカウントのパスワードを、指定された新しいパスワードに更新する。

  * Formの実装

    クラスに付与されたアノテーションによってNullチェック以外の入力チェックが網羅されていることから、単項目チェックとしては\ ``@NotNull`` \のみを付与している。

    .. code-block:: java

       package org.terasoluna.securelogin.app.passwordreissue;

       // omitted

       @Data
       @Compare(source = "newPasssword", destination = "confirmNewPassword", operator = Compare.Operator.EQUAL)
       @StrongPassword(usernamePropertyName = "username", newPasswordPropertyName = "newPassword") // (1)
       @NotReusedPassword(usernamePropertyName = "username", newPasswordPropertyName = "newPassword") // (2)
       public class PasswordResetForm implements Serializable{

           private static final long serialVersionUID = 1L;

           @NotNull
           private String username;

           @NotNull
           private String token;

           @NotNull
           private String secret;

           @NotNull
           private String newPassword;

           @NotNull
           private String confirmNewPassword;
       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | パスワードの強度を検査するためのアノテーション。詳細は :ref:`パスワードの品質チェック <password-strength>` を参照。
       * - | (2)
         - | パスワードの再利用を検査するためのアノテーション。詳細は :ref:`パスワードの品質チェック <password-strength>` を参照。

  * Viewの実装

    **パスワード再発行画面(passwordResetForm.jsp)**

    .. code-block:: jsp

       <body>
           <div id="wrapper">
               <h1>Reset Password</h1>
               <t:messagesPanel />
               <form:form
                   action="${f:h(pageContext.request.contextPath)}/reissue/resetpassword"
                   method="POST" modelAttribute="passwordResetForm">
                   <table>
                       <tr>
                           <th><form:label path="username">Username</form:label></th>
                           <td>${f:h(passwordResetForm.username)} <form:hidden
                                   path="username" value="${f:h(passwordResetForm.username)}" />  <!-- (1) -->
                           </td>
                           <td></td>
                       </tr>
                       <form:hidden path="token" value="${f:h(passwordResetForm.token)}" /> <!-- (2) -->
                       <tr>
                           <th><form:label path="secret" cssErrorClass="error-label">Secret</form:label>
                           </th>
                           <td><form:password path="secret" cssErrorClass="error-input" /></td> <!-- (3) -->
                           <td><form:errors path="secret" cssClass="error-messages" /></td>
                       </tr>
                       <tr>
                           <th><form:label path="newPassword" cssErrorClass="error-label">New password</form:label>
                           </th>
                           <td><form:password path="newPassword"
                                   cssErrorClass="error-input" /></td>
                           <td><form:errors path="newPassword" cssClass="error-messages"
                                   htmlEscape="false" /></td>
                       </tr>
                       <tr>
                           <th><form:label path="confirmNewPassword"
                                   cssErrorClass="error-label">New password(Confirm)</form:label></th>
                           <td><form:password path="confirmNewPassword"
                                   cssErrorClass="error-input" /></td>
                           <td><form:errors path="confirmNewPassword"
                                   cssClass="error-messages" /></td>
                       </tr>
                   </table>

                   <input id="submit" type="submit" value="Reset password" />
               </form:form>
           </div>
       </body>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | ユーザ名をhidden項目として保持する。
       * - | (2)
         - | トークンをhidden項目として保持する。
       * - | (3)
         - | ユーザの確認のために、秘密情報を入力させる。

    **パスワード再発行画面(passwordResetComplete.jsp)**

    .. code-block:: jsp

       <body>
           <div id="wrapper">
               <h1>Your password was successfully reset.</h1>
               <a href="${f:h(pageContext.request.contextPath)}/">go to Top</a>
           </div>
       </body>

  * Controllerの実装

    .. code-block:: java

       package org.terasoluna.securelogin.app.passwordreissue;

       // omitted

       @Controller
       @RequestMapping("/reissue")
       public class PasswordReissueController {

           @Inject
           PasswordReissueService passwordReissueService;

           // omitted

           @RequestMapping(value = "resetpassword", method = RequestMethod.POST)
           public String resetPassword(@Validated PasswordResetForm form,
                   BindingResult bindingResult, Model model) {
               if (bindingResult.hasErrors()) {
                   return showPasswordResetForm(form, model, form.getToken());
               }

               try {
                   passwordReissueService.resetPassword(form.getUsername(),
                           form.getToken(), form.getSecret(), form.getNewPassword()); // (1)
                   return "redirect:/reissue/resetpassword?complete";
               } catch (BusinessException e) {
                   model.addAttribute(e.getResultMessages());
                   return showPasswordResetForm(form, model, form.getToken());
               }
           }

           @RequestMapping(value = "resetpassword", params = "complete", method = RequestMethod.GET)
           public String resetPasswordComplete() {
               return "passwordreissue/passwordResetComplete";
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
         - | Serviceのメソッドにユーザ名、トークン、秘密情報、新しいパスワードを渡す。ユーザ名、トークン、秘密情報の組み合わせが正しい場合、新しいパスワードに更新される。

.. _reissue-info-invalidate:

パスワード再発行の失敗上限回数の設定
--------------------------------------------------------------------------------
実装する要件一覧
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
* :ref:`パスワード再発行の失敗上限回数の設定 <sec-requirements>`

動作イメージ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. figure:: ./images/SecureLogin_invalidate_token.png
   :alt: Page Transition
   :width: 80%
   :align: center

パスワード再発行画面のURLが何らかの原因で漏えいした場合であっても、秘密情報が漏えいしていなければパスワードが不正に再発行されることはない。
秘密情報には十分に推測困難なランダム値を用いているため簡単に破られる可能性は低いが、ブルートフォース攻撃を阻止する目的で認証失敗の回数に上限値を設定する。
上限値を超えてパスワード再発行のための認証に失敗した場合、そのURL（トークン）でのパスワード再発行が行えないようにする。

実装方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 本アプリケーションでは、パスワード再発行に失敗した履歴を「パスワード再発行失敗イベント」エンティティとしてデータベースに保存し、このパスワード再発行失敗イベントエンティティを用いて、パスワード再発行の失敗回数をカウントする。
| 失敗回数があらかじめ設定した上限値以上であれば、パスワード再発行画面へのアクセス時に例外をスローする。
| 具体的には、以下の二つの処理を実装して用いることにより、要件を実現する。

* パスワード再発行失敗イベントエンティティの保存

  :ref:`reissue-info-validate` における、「パスワード再発行のための認証情報を用いたユーザの確認」処理の中で、ユーザの確認に失敗した場合に、使用したトークンと失敗日時の組をパスワード再発行失敗イベントエンティティとしてデータベースに登録する。

* パスワード再発行時の例外のスロー

  パスワード再発行のために認証情報をデータベースから取得した際に、パスワード再発行失敗イベントエンティティの数をカウントし、上限値以上であれば例外をスローする。

.. warning ::

   パスワード再発行失敗イベントエンティティはパスワード再発行の失敗回数のカウントのみを目的としているため、不要になったタイミングで消去する。
   パスワード再発行の失敗時のログが必要な場合は必ず別途ログを保存しておくこと。

コード解説
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* 共通部分

  前提として、 :ref:`reissue-info-validate` に記した各処理が実装されているものとする。
  その他に共通的に必要な、データベースに対するパスワード再発行失敗イベントエンティティの登録、検索、削除に関する実装を以下に示す。

  * Entityの実装

    パスワード再発行失敗イベントエンティティの実装の実装は以下の通り。

    .. code-block:: java

       package org.terasoluna.securelogin.domain.model;

       // omitted

       @Data
       public class FailedPasswordReissue {

           private String token; // (1)

           private LocalDateTime attemptDate; // (2)

       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | パスワード再発行に使用したトークン
       * - | (2)
         - | パスワード再発行を試行した日時

  * Repositoryの実装

    Entityの検索、登録、削除を行うためのRepositoryを以下に示す。

    .. code-block:: java

       package org.terasoluna.securelogin.domain.repository.passwordreissue;

       // omitted

       public interface FailedPasswordReissueRepository {

           int countByToken(@Param("token") String token); // (1)

           int create(FailedPasswordReissue event); // (2)

           int deleteByToken(@Param("token") String token); // (3)

           // omitted

       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | 引数として与えられたトークンをキーとして\ ``FailedPasswordReissue``\ オブジェクトの個数を取得するメソッド
       * - | (2)
         - | 引数として与えられた\ ``FailedPasswordReissue``\ オブジェクトをデータベースのレコードとして登録するメソッド
       * - | (3)
         - | 引数として与えられたトークンをキーとして\ ``FailedPasswordReissue``\ オブジェクトを削除するメソッド

    マッピングファイルは以下の通り。

    .. code-block:: xml

       <?xml version="1.0" encoding="UTF-8"?>
       <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

       <mapper
        namespace="org.terasoluna.securelogin.domain.repository.passwordreissue.FailedPasswordReissueRepository">

        <select id="countByToken" resultType="_int">
           <![CDATA[
               SELECT
                   COUNT(*)
               FROM
                   failed_password_reissue
               WHERE
                   token = #{token}
           ]]>
        </select>

        <insert id="create" parameterType="FailedPasswordReissue">
           <![CDATA[
               INSERT INTO failed_password_reissue (
                   token,
                   attempt_date
               ) VALUES (
                #{token},
                   #{attemptDate}
               )
           ]]>
        </insert>

        <delete id="deleteByToken">
           <![CDATA[
            DELETE FROM
                failed_password_reissue
            WHERE
                token = #{token}
           ]]>
        </delete>

       </mapper>

以下、実装方法に従って実装されたコードについて順に解説する。

* パスワード再発行失敗イベントエンティティの保存

  パスワード再発行失敗時に行う処理を実装したクラスを以下に示す。

  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.passwordreissue;

     public interface PasswordReissueFailureSharedService {

         void resetFailure(String username, String token);

     }

  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.passwordreissue;

     // omitted

     @Service
     @Transactional
     public class PasswordReissueFailureSharedServiceImpl implements
             PasswordReissueFailureSharedService {

         @Inject
         ClassicDateFactory dateFactory;

         @Inject
         FailedPasswordReissueRepository failedPasswordReissueRepository;

         // omitted

         @Transactional(propagation = Propagation.REQUIRES_NEW) // (1)
         @Override
         public void resetFailure(String username, String token) {
             FailedPasswordReissue event = new FailedPasswordReissue(); // (2)
             event.setToken(token);
             event.setAttemptDate(dateFactory.newTimestamp().toLocalDateTime());
             failedPasswordReissueRepository.create(event); // (3)
         }

     }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | パスワード再発行に失敗した際に呼び出されるメソッドであり、呼び出し元で実行時例外を発生させる設計としている。
         | そのため、呼び出し元のServiceとは別にトランザクション管理を行うために、伝搬方法を「REQUIRES_NEW」に指定する。
     * - | (2)
       - | パスワード再発行失敗イベントエンティティを作成し、トークンと失敗日時を設定する。
     * - | (3)
       - | (2)で作成したパスワード再発行失敗イベントエンティティをデータベースに登録する。

  :ref:`reissue-info-validate` の「パスワード再発行のための認証情報を用いたユーザの確認」処理の中から、パスワード再発行失敗時の処理を呼び出す。

  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.passwordreissue;

     // omitted

     @Service
     @Transactional
     public class PasswordReissueServiceImpl implements PasswordReissueService {

         @Inject
         PasswordReissueFailureSharedService passwordReissueFailureSharedService;

         @Inject
         PasswordReissueInfoRepository passwordReissueInfoRepository;

         @Inject
         AccountSharedService accountSharedService;

         @Inject
         PasswordEncoder passwordEncoder;

         // omitted

         @Override
         public boolean resetPassword(String username, String token, String secret,
                 String rawPassword) {
             PasswordReissueInfo info = this.findOne(token); // (1)
             if (!passwordEncoder.matches(secret, info.getSecret())) { // (2)
                 passwordReissueFailureSharedService.resetFailure(username, token); // (3)
                 throw new BusinessException(ResultMessages.error().add(  // (4)
                     MessageKeys.E_SL_PR_5003));
             }

             //omitted

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
       - | 引数として与えられたトークンを用いて、データベースからパスワード再発行用の認証情報を取得する。
     * - | (2)
       - | パスワード再発行用の認証情報に含まれるハッシュ化された秘密情報と、引数として与えられた秘密情報を比較する。
     * - | (3)
       - | パスワード再発行失敗時の処理を行うSharedServiceのメソッド呼び出す。
     * - | (4)
       - | 実行時例外をthrowするが、パスワード再発行失敗時の処理は別のトランザクションで実行されるため、影響を与えることはない。

* パスワード再発行時の例外のスロー

  パスワード再発行の失敗回数の取得と、失敗回数が上限に達した際の処理の実装を以下に示す。

  .. code-block:: java

     package org.terasoluna.securelogin.domain.service.passwordreissue;

     // omitted

     @Service
     @Transactional
     public class PasswordReissueServiceImpl implements PasswordReissueService {

         @Inject
         FailedPasswordReissueRepository failedPasswordReissueRepository;

         @Inject
         PasswordReissueInfoRepository passwordReissueInfoRepository;

         @Value("${security.tokenValidityThreshold}")
         int tokenValidityThreshold; // (1)

         // omitted

         @Override
         @Transactional(readOnly = true)
         public PasswordReissueInfo findOne(String token) {

             // omitted

             int count = failedPasswordReissueRepository.countByToken(token); // (2)
             if (count >= tokenValidityThreshold) { // (3)
                 throw new BusinessException(ResultMessages.error().add(
                         MessageKeys.E_SL_PR_5004));
             }

             return info;
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
       - | パスワード再発行の失敗回数の上限値をプロパティファイルから取得して設定する。
     * - | (2)
       - | 引数として与えられたトークンをキーとして、データベースからパスワード再発行失敗イベントエンティティの数を取得。
     * - | (3)
       - | 取得したパスワード再発行の失敗イベントエンティティの数と失敗回数の上限値を比較し、上限値以上ならば例外をスローする。

.. _secure-input-validation :

セキュリティ観点での入力値チェック
--------------------------------------------------------------------------------

実装する要件一覧
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* :ref:`リクエストパラメータに対する共通的な禁止文字の設定 <sec-requirements>`
* :ref:`アップロードファイル名に対する共通的な禁止文字列の設定 <sec-requirements>`
* :ref:`制御文字の入力チェック <sec-requirements>`
* :ref:`ファイル拡張子の入力チェック <sec-requirements>`
* :ref:`ファイル名の入力チェック <sec-requirements>`
* :ref:`URLのドメインに対する入力チェック <sec-requirements>`
* :ref:`メールアドレスのドメインに対する入力チェック <sec-requirements>`

動作イメージ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* 共通的な禁止文字列の設定

.. figure:: ./images/SecureLogin_common_input_validation.png
   :alt: Common input validation
   :width: 80%
   :align: center

* 個別の入力チェック

.. figure:: ./images/SecureLogin_input_validation.png
   :alt: Input validation
   :width: 80%
   :align: center

実装方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| アプリケーション全体の共通的な禁止文字列の設定と個別の入力チェックとではチェックの対象範囲が大きく異なるため、それぞれ別の方法で実装を行う。
| 共通的な禁止文字列を設定するためには、:ref:`controller-common-process` に記した二つの方法を用いることができる。本アプリケーションでは、Controllerのハンドラメソッドにマッピングされるか否かに関わらずチェックを行うために、Servlet Filterを用いて実装を行う。入力エラーが発生した場合、通常のユーザ操作の結果としては想定できない入力値が入力されたことを意味するため、ユーザビリティの低下を考慮せずに共通的なエラー画面へ遷移させるように設定ファイルへ記述する。
| 個別の入力チェックには :doc:`../ArchitectureInDetail/WebApplicationDetail/Validation` の機能を利用することができる。本アプリケーションではBean Validationを用いて入力値のチェックを実現する。個別の入力エラーに対しては、Bean Validation当該入力項目に対してエラーメッセージを表示して再入力を促すよう実装を行う。

コード解説
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

上記の実装方法に従って実装されたコードについて順に解説する。

* Servlet Filterの実装

  | ユーザからの入力をアプリケーション内で使用する場合、SQLインジェクションやXSS、ディレクトリトラバーサル（パストラバーサル）といったインジェクション攻撃の対象となる可能性がある。
  | これらの攻撃への対策として、アプリケーション全体でユーザからの入力を検証するためのServlet Filterの実装例を示す。
  | 本アプリケーションでは、以下の項目に対してそれぞれ設定した禁止文字を含んでいないことを検証する。

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 20 70

     * - 項番
       - 項目
       - 説明
     * - | (1)
       - | リクエストパラメータ
       - | リクエストパラメータはユーザからの入力を受け付けるために一般的に利用されるため、入力チェックの対象とする
         | パラメータ名と値の両方がユーザからの入力となる可能性があるため、両方をチェックする
     * - | (2)
       - | アップロードファイル名
       - | 本アプリケーションではファイルアップロード機能を実装しているため、ユーザからの入力であるアップロードファイル名を入力チェックの対象とする

  | 尚、以下のコードでは \ ``org.springframework.web.multipart.MultipartRequest`` \を使用するため、\ ``org.springframework.web.multipart.support.MultipartFilter`` \の使用を前提としている。\ ``MultipartFilter`` \ については \ :doc:`../ArchitectureInDetail/WebApplicationDetail/FileUpload`\を参照のこと。

  .. code-block:: java

     package org.terasoluna.securelogin.app.common.filter;

     // omitted

     public class InputValidationFilter extends OncePerRequestFilter { // (1)

         private final List<Character> prohibitedChars;

         private final List<Character> prohibitedCharsForFileName;

         public InputValidationFilter(char[] prohibitedChars,
                 char[] prohibitedCharsForFileName) {
             this.prohibitedChars = Chars.asList(prohibitedChars); // (2)
             this.prohibitedCharsForFileName = Chars
                     .asList(prohibitedCharsForFileName); // (3)
         }

         @Override
         protected void doFilterInternal(HttpServletRequest request,
                 HttpServletResponse response, FilterChain filterChain)
                 throws ServletException, IOException {
             if (request != null) {
                 validateRequestParams(request); // (4)

                 if (request instanceof MultipartRequest) {
                     validateFileNames((MultipartRequest) request); // (5)
                 }
             }

             filterChain.doFilter(request, response); // (6)
         }

         private void validateRequestParams(HttpServletRequest request) {
             Map<String, String[]> params = request.getParameterMap();
             for (Map.Entry<String, String[]> entry : params.entrySet()) {
                 validate(entry.getKey(), prohibitedChars); // (7)
                 for (String value : entry.getValue()) {
                     validate(value, prohibitedChars); // (8)
                 }
             }
         }

         private void validateFileNames(MultipartRequest request) {
             for (Map.Entry<String, MultipartFile> entry : request.getFileMap()
                     .entrySet()) {
                 String filename = new File(entry.getValue().getOriginalFilename())
                         .getName(); // (9)
                 validate(filename, prohibitedCharsForFileName); // (10)
             }
         }

         private void validate(String target, List<Character> prohibited) {
             if (StringUtils.hasLength(target)) {
                 List<Character> chars = Chars.asList(target.toCharArray());
                 for(Character prohibitedChar : prohibited) { // (11)
                     if (chars.contains(prohibitedChar)) {
                         throw new InvalidCharacterException(
                             "The request contains prohibited charcter.");
                     }
                 }
             }
         }

     }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
     :class: longtable

     * - 項番
       - 説明
     * - | (1)
       - | \ ``org.springframework.web.filter.OncePerRequestFilter`` \ を継承することで、リクエストに対して一度だけ処理が実行されることが保証される
     * - | (2)
       - | リクエストパラメータの禁止文字の一覧を文字列として受け取り、文字のリストとして保持する
     * - | (3)
       - | ファイル名の禁止文字の一覧を文字列として受け取り、文字のリストとして保持する
     * - | (4)
       - | リクエストパラメータの入力チェックを行うメソッドを呼び出す
     * - | (5)
       - | ファイルアップロードリクエスト(Content-Typeが\ ``multipart/form-data`` \のPOSTリクエスト)の場合、\ ``MultipartFilter`` \の機能によって、ここでの\ ``request`` \オブジェクトは\ ``MultipartRequest`` \を実装した\ ``StandardMultipartHttpServletRequest`` \のインスタンスとなる
         | \ ``MultipartRequest`` \のAPIを用いてファイル名を取り出し、入力チェックを行うメソッドを呼び出す
     * - | (6)
       - | 入力チェックが完了した後、後続のServlte Filterの処理を実行するためのメソッドを呼び出す
     * - | (7), (8)
       - | \ ``HttpServletRequest`` \ からリクエストパラメータの一覧を取得し、各リクエストパラメータ名およびリクエストパラメータ値について、実際の入力チェックを行うメソッドを呼び出す
     * - | (9)
       - | \ ``MultipartRequest`` \ からアップロードされたファイルの一覧を取得し、実際のファイル名を取得する
         | クライアントのブラウザやOSによってはファイル名にパスが含まれたり、パス区切り文字が異なるため、ファイル名のみを取得する際にこのような処理が必要となる
     * - | (10)
       - | アップロードされた各ファイルのファイル名について、実際の入力チェックを行うメソッドを呼び出す
     * - | (11)
       - | 入力チェック対象の文字列を一文字ずつ順に、禁止文字に含まれているかどうかをチェックし、含まれている場合は例外を投げる
         | \ ``InvalidCharacterException`` \ は \ ``RuntimeException`` \を継承して作成した例外である。コードは省略する

  .. raw:: latex

     \newpage

  .. tip::

     本アプリケーションではリクエストパラメータおよびファイル名のチェックのみ共通的な禁止文字のチェックを行っている。
     必要に応じて、\ ``HttpServletRequest`` \ の \ ``getHeaders`` \ や \ ``getCookies`` \ を用いてHTTPヘッダやCookieの値を取得することにより、HTTPヘッダやCookieに対しても同様の方法でチェックすることができる。

* Servlet Filterの設定

  | 作成した\ ``InputValidationFilter`` \を有効にするため、web.xmlに設定を行う。
  | 禁止文字をプロパティファイルから読み込み、\ ``InputValidationFilter`` \ をBean定義した上で、\ ``org.springframework.web.filter.DelegatingFilterProxy`` \ を使用して設定する。

  **web.xml**

  .. code-block:: xml

     <filter>
         <filter-name>MultipartFilter</filter-name>
         <filter-class>org.springframework.web.multipart.support.MultipartFilter</filter-class>  <!-- (1) -->
     </filter>
     <filter-mapping>
         <filter-name>MultipartFilter</filter-name>
         <url-pattern>/*</url-pattern>
     </filter-mapping>

     <filter>
         <filter-name>inputValidationFilter</filter-name>
         <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>  <!-- (2) -->
     </filter>
     <filter-mapping>
         <filter-name>inputValidationFilter</filter-name>
         <url-pattern>/*</url-pattern>
     </filter-mapping>

     <!-- omitted -->

     <error-page>
         <exception-type>org.terasoluna.securelogin.app.common.filter.exception.InvalidCharacterException</exception-type>  <!-- (3) -->
         <location>/WEB-INF/views/common/error/invalidCharacterError.jsp</location>
     </error-page>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | \ ``InputValidationFilter`` \ の使用の前提となっている \ ``MultipartFilter`` \ を設定する
         | \ ``MultipartFilter`` \を\ ``InputValidationFilter`` \よりも前に定義する必要があることに注意すること
     * - | (2) 
       - | \ ``DelegatingFilterProxy`` \ を用いて、Bean定義した \ ``InputValidationFilter`` \ を設定する
         | \ ``<filter-name>`` \にはBean名を指定すること
     * - | (3) 
       - | \ ``InvalidCharacterException`` \ がスローされた際に表示するエラー画面を設定する

  .. note::

     ファイル名の入力チェックのために\ ``MultipartFilter`` \を利用しているため、ここに記述した内容に加えて :ref:`file-upload_how_to_usr_application_settings` に記したServlet 3.0のアップロード機能を有効化するための設定が必要となる。

  **invalidCharacterError.jsp**

  .. code-block:: jsp

     <% response.setStatus(HttpServletResponse.SC_BAD_REQUEST); %>  <!-- (1) -->
     <!DOCTYPE html>
     <html>
     <head>
     <meta charset="utf-8">
     <title>Invalid Character Error!</title>

     <!-- omitted -->

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | \ ``InvalidCharacterException`` \はクライアントの入力に起因して発生する例外であるため、のHTTPステータスコードを\ ``"400"`` \(Bad Request)に設定する

  **applicationContext.xml**

  .. code-block:: xml

     <bean id="inputValidationFilter" class="org.terasoluna.securelogin.app.common.filter.InputValidationFilter">
         <constructor-arg index="0" value="${app.security.prohibitedChars}"/>  <!-- (1) -->
         <constructor-arg index="1" value="${app.security.prohibitedCharsForFileName}"/>  <!-- (2) -->
     </bean>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | リクエストパラメータの禁止文字の一覧を文字列としてプロパティから取得する
     * - | (2)
       - | ファイル名の禁止文字の一覧を文字列としてプロパティから取得する

  **application.properties**

  .. code-block:: properties

     ## (1)
     app.security.prohibitedChars=&\\!"<>*

     ## (2)
     app.security.prohibitedCharsForFileName=&\\!"<>*;:

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | 本アプリケーションにおいて、リクエストパラメータに含まれることを想定していない文字のリストを文字列として指定する
     * - | (2)
       - | 本アプリケーションにおいて、アップロードファイル名に含まれることを想定していない文字のリストを文字列として指定する

* Bean Validationのアノテーションの作成

  アプリケーションの仕様で想定していない入力値を入力されることによるセキュリティ上のリスクを軽減するために、要件に合わせて入力値を検証するBean Validationのアノテーションを作成する。

  * 制御文字が含まれないことを検証するアノテーション

    | 入力値に制御文字が含まれている場合、アプリケーションが予期しない問題を引き起こす可能性がある。そのため、制御文字を入力する必要がない入力項目に対して制御文字が含まれていないことをチェックする。
    | 文章の入力の際には制御文字のうち改行コードのみを許容する場合も多いため、改行コードを許可するアノテーションも別途作成する。
    | 正規表現を用いたチェックを行うことで制御文字が含まれていないことを検証することができる。

    .. code-block:: java

       package org.terasoluna.securelogin.app.common.validation;

       // omitted
       @Documented
       @Constraint(validatedBy = {})
       @Target({ FIELD })
       @Retention(RUNTIME)
       @ReportAsSingleViolation  // (1)
       @Pattern(regexp = "^\\P{Cntrl}*$") // (2)
       public @interface NotContainControlChars {
           String message() default "{org.terasoluna.securelogin.app.common.validation.NotContainControlChars.message}";

           Class<?>[] groups() default {};

           @Target({ FIELD })
           @Retention(RUNTIME)
           @Documented
           public @interface List {
               NotContainControlChars[] value();
           }

           Class<? extends Payload>[] payload() default {};
       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | エラーメッセージとして、このアノテーションで指定したメッセージのみを出力するために\ ``@ReportAsSingleViolation`` \アノテーションを付与する
       * - | (2)
         - | 正規表現を用いた入力チェックを行うために\ ``@Pattern`` \アノテーションを付与する
           | \ ``\P{Cntrl}`` \はJavaの正規表現において「制御文字以外の文字」を意味するため、\ ``^\\P{Cntrl}*$`` \は最初から最後まで制御文字を含まない文字列のみにマッチする

    | 同様に、改行コードを許容する場合のアノテーションをの実装例を以下に示す。

    .. code-block:: java

       package org.terasoluna.securelogin.app.common.validation;

       // omitted
       @Documented
       @Constraint(validatedBy = {})
       @Target({ FIELD })
       @Retention(RUNTIME)
       @ReportAsSingleViolation
       @Pattern(regexp = "^[\\r\\n\\P{Cntrl}]*$") // (1)
       public @interface NotContainControlCharsExceptNewlines {
           String message() default "{org.terasoluna.securelogin.app.common.validation.NotContainControlCharsExceptNewlines.message}";

           Class<?>[] groups() default {};

           @Target({ FIELD })
           @Retention(RUNTIME)
           @Documented
           public @interface List {
               NotContainControlCharsExceptNewlines[] value();
           }

           Class<? extends Payload>[] payload() default {};
       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | 「制御文字以外の文字(\ ``\P{Cntrl}`` \)」と改行コード(\ ``\r`` \, \ ``\n`` \)のみで構成される文字列にマッチするように、正規表現を指定する。

  * アップロードされるファイルの拡張子が許容されているものであることを検証するアノテーション

    | ユーザからのファイルのアップロードを受け付ける際、受け付けるファイルの形式を制限したい場合がある。そのような場合に、許容するファイルの拡張子の一覧を設定し、アップロードされたファイルの拡張子が一覧に含まれているかをチェックする。
    | 拡張子の大文字・小文字を区別するか否かを切り替えられる仕様とする。

    .. warning::
       ファイルの拡張子は容易に偽装され得るため、拡張子のチェックを行った場合であっても無条件にファイル形式を信用してはならない。

    .. code-block:: java

       package org.terasoluna.securelogin.app.common.validation;

       // omitted

       @Documented
       @Constraint(validatedBy = { FileExtensionValidator.class })
       @Target({ ElementType.FIELD })
       @Retention(RetentionPolicy.RUNTIME)
       public @interface FileExtension {
           String message() default "{org.terasoluna.securelogin.app.common.validation.FileExtension.message}";

           Class<?>[] groups() default {};

           Class<? extends Payload>[] payload() default {};

           String[] extensions();  // (1)

           boolean ignoreCase() default true;  // (2)

           @Target({ ElementType.FIELD })
           @Retention(RetentionPolicy.RUNTIME)
           @Documented
           public @interface List {
               FileExtension[] value();
           }
       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | 許容する拡張子の一覧を設定する
       * - | (2)
         - | 大文字・小文字の違いを無視するか否か。デフォルト値は \ ``true`` \ （無視する）

    .. code-block :: java

       package org.terasoluna.securelogin.app.common.validation;

       // omitted

       public class FileExtensionValidator implements
               ConstraintValidator<FileExtension, MultipartFile> {

           private Set<String> extensions;

           private boolean ignoreCase;

           @Override
           public void initialize(FileExtension constraintAnnotation) {
               this.extensions = new HashSet<String>(
                       Arrays.asList(constraintAnnotation.extensions()));
               this.ignoreCase = constraintAnnotation.ignoreCase();
           }

           @Override
           public boolean isValid(MultipartFile value,
                   ConstraintValidatorContext context) {
               if (value == null) {  // (1)
                   return true;
               }

               String fileNameExtension = StringUtils.getFilenameExtension(value
                       .getOriginalFilename());  // (2)
               if (!StringUtils.hasLength(fileNameExtension)) {  // (3)
                   return false;
               }

               for (String extension : extensions) {  // (4)
                   if (fileNameExtension.equals(extension) || ignoreCase
                           && fileNameExtension.equalsIgnoreCase(extension)) {
                       return true;
                   }
               }
               return false;
           }

       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | \ ``null`` \ チェックは他のアノテーションを用いて行うため、\ ``null`` \ の場合は \ ``true`` \ を返す
       * - | (2)
         - | \ ``org.springframework.util.StringUtils`` \ の \ ``getFilenameExtension`` \ メソッドを用いてファイル名から拡張子を取得する
       * - | (3)
         - | 拡張子の無いファイルは許容しないため、拡張子が無い場合は \ ``false`` \ を返す
       * - | (4)
         - | 許容する拡張子の一覧から拡張子を一つずつ取り出し、ファイル名から取得した文字列と比較する
           | 一つでも一致するものがあれば \ ``true`` \ を返し、いずれとも一致しなければ　\ ``false`` \ を返す。

  * アップロードされるファイルのファイル名が許容されているパターンと一致することを検証するアノテーション

    ユーザからのファイルのアップロードを受け付ける際、受け付けるファイル名の形式を制限したい場合がある。そのような場合に、許容するファイル名のパターンを正規表現で設定し、アップロードされたファイルのファイル名がパターンと一致するかをチェックする。

    .. code-block:: java

       package org.terasoluna.securelogin.app.common.validation;

       // omitted

       @Documented
       @Constraint(validatedBy = { FileNamePatternValidator.class })
       @Target({ ElementType.FIELD })
       @Retention(RetentionPolicy.RUNTIME)
       public @interface FileNamePattern {

           String message() default "{org.terasoluna.securelogin.app.common.validation.FileNamePattern.message}";

           Class<?>[] groups() default {};

           Class<? extends Payload>[] payload() default {};

           String pattern() default "";  // (1)

           @Target({ ElementType.FIELD })
           @Retention(RetentionPolicy.RUNTIME)
           @Documented
           public @interface List {
               FileNamePattern[] value();
           }

       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | 許容するファイル名のパターン

    .. code-block:: java

       package org.terasoluna.securelogin.app.common.validation;

       // omitted

       public class FileNamePatternValidator implements
               ConstraintValidator<FileNamePattern, MultipartFile> {

           private Pattern pattern;

           @Override
           public void initialize(FileNamePattern constraintAnnotation) {
               this.pattern = Pattern.compile(constraintAnnotation.pattern());  // (1)
           }

           @Override
           public boolean isValid(MultipartFile value,
                   ConstraintValidatorContext context) {
               if (value == null) {  // (2)
                   return true;
               }

               String filename = new File(value.getOriginalFilename()).getName();  // (3)
               return pattern.matcher(filename).matches();  // (4)
           }

       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | チェックする正規表現のパターンを \ ``Pattern`` \ として作成し、保持する
       * - | (2)
         - | \ ``null`` \ チェックは他のアノテーションを用いて行うため、\ ``null`` \ の場合は \ ``true`` \ を返す
       * - | (3)
         - | \ ``MultipartRequest`` \ から実際のファイル名を取得する。クライアントのブラウザやOSによってはファイル名にパスが含まれたり、パス区切り文字が異なるため、ファイル名のみを取得する際にこのような処理が必要となる
       * - | (4)
         - | (1)で作成した \ ``Pattern`` \ とファイル名がマッチする場合は \ ``true`` \ を返し、マッチしない場合は\ ``false`` \ を返す


  * 入力されたURLのドメインが許容されているものであることを検証するアノテーション

    | ユーザからURLの入力を受け付ける際、許容するドメインを制限したい場合がある。そのような場合に、許容するドメインの一覧を設定し、入力されたURLのドメインが一覧に含まれているドメインかまたはそのサブドメインであるかをチェックする。
    | 同時にURL形式であることをチェックするため、\ ``org.hibernate.validator.constraints.URL`` \ と組み合わせて実装する。

    .. code-block:: java

       package org.terasoluna.securelogin.app.common.validation;

       // omitted

       @Documented
       @Constraint(validatedBy = { DomainRestrictedURLValidator.class })
       @Target({ FIELD })
       @Retention(RUNTIME)
       @URL  // (1)
       public @interface DomainRestrictedURL {

           String message() default "{org.terasoluna.securelogin.app.common.validation.DomainRestrictedURL.message}";

           Class<?>[] groups() default {};

           String[] allowedDomains() default {};  // (2)

           @Target({ FIELD })
           @Retention(RUNTIME)
           @Documented
           public @interface List {
               DomainRestrictedURL[] value();
           }

           Class<? extends Payload>[] payload() default {};

       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | URL形式であることをチェックするために、\ ``@URL`` \ を付与する
       * - | (2)
         - | 許容するドメインの一覧

    .. code-block:: java

       package org.terasoluna.securelogin.app.common.validation;

       // omitted

       public class DomainRestrictedURLValidator implements
               ConstraintValidator<DomainRestrictedURL, String> {

           private static final Pattern URL_REGEX = Pattern  // (1)
               .compile( "(?i)^(?:[a-z](?:[-a-z0-9\\+\\.])*)" + // protocol
                        ":(?:\\/\\/([^\\/:]+)" + // auth+host/ip
                        "(?::([0-9]*))?" + // port
                        "(?:\\/.*)*)$"
                );

           private Set<String> allowedDomains;

           @Override
           public void initialize(DomainRestrictedURL constraintAnnotation) {
               allowedDomains = new HashSet<String>(Arrays.asList(constraintAnnotation
                       .allowedDomains()));  // (2)
           }

           @Override
           public boolean isValid(String value, ConstraintValidatorContext context) {
               Matcher urlMatcher = URL_REGEX.matcher(value);
               if (urlMatcher.matches()) {  // (3)
                   String host = urlMatcher.group(1);
                   for(String domain : allowedDomains) {  // (4)
                       if (StringUtils.hasLength(host) && host.endsWith("."+domain)) {
                           return true;
                       }
                   }
                   return false;
               } else {
                   return true;
               }
           }

       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | URLのドメインを取得するための正規表現のパターンを \ ``Pattern`` \ として作成し、保持する
           | 正しいURL形式であるかどうかの検証は\ ``@URL`` \で行うため、ここでは必要最小限の正規表現を指定している
       * - | (2)
         - | 許容するドメインの一覧を取得し、保持する
       * - | (3)
         - | URL形式であるかどうかをチェックする。不正な形式の場合は、組み合わせて利用している\ ``URL`` \アノテーションによって検証エラーとなるため、ここでは \ ``true`` \ を返す
       * - | (4)
         - | 許容されるドメインの一覧から一つずつ取り出し、URLのホスト部の末尾と一致するかをチェックする。一つでも一致すれば \ ``true`` \ を返し、一つも一致しなければ \ ``false`` \ を返す


  * 入力されたメールアドレスのドメインが許容されているものであることを検証するアノテーション

    | ユーザからメールアドレスの入力を受け付ける際、許容するドメインを制限したい場合がある。そのような場合に、許容するドメインの一覧を設定し、入力されたメールアドレスのドメインが一覧に含まれているかをチェックする。許容するドメインのサブドメインまで許すか否かを切り替えられる仕様とする。
    | 同時にメールアドレス形式であることをチェックするため、\ ``org.hibernate.validator.constraints.Email`` \ と組み合わせて実装する。

    .. code-block:: java

       package org.terasoluna.securelogin.app.common.validation;

       // omitted

       @Documented
       @Constraint(validatedBy = { DomainRestrictedEmailValidator.class })
       @Target({ FIELD })
       @Retention(RUNTIME)
       @Email  // (1)
       public @interface DomainRestrictedEmail {
           String message() default "{org.terasoluna.securelogin.app.common.validation.DomainRestrictedEmail.message}";

           Class<?>[] groups() default {};

           String[] allowedDomains() default {};  // (2)

           boolean allowSubDomain() default false;  // (3)

           @Target({ FIELD })
           @Retention(RUNTIME)
           @Documented
           public @interface List {
               DomainRestrictedEmail[] value();
           }

           Class<? extends Payload>[] payload() default {};
       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | メールアドレス形式であることをチェックするために、\ ``@Email`` \ を付与する
       * - | (2)
         - | 許容するドメインの一覧
       * - | (3)
         - | サブドメインを許容するか否か。デフォルト値は \ ``false`` \ （許容しない）

    .. code-block:: java

       package org.terasoluna.securelogin.app.common.validation;

       // omitted

       public class DomainRestrictedEmailValidator implements
               ConstraintValidator<DomainRestrictedEmail, CharSequence> {

           private Set<String> allowedDomains;

           private boolean allowSubDomain;

           @Override
           public void initialize(DomainRestrictedEmail constraintAnnotation) {
               allowedDomains = new HashSet<String>(Arrays.asList(constraintAnnotation
                       .allowedDomains()));  // (1)
               allowSubDomain = constraintAnnotation.allowSubDomain();  // (2)
           }

           @Override
           public boolean isValid(CharSequence value,
                   ConstraintValidatorContext context) {
               if (value == null) {  // (3)
                   return true;
               }

               for (String domain : allowedDomains) {  // (4)
                   if (value.toString().endsWith("@" + domain)
                           || (allowSubDomain && value.toString().endsWith(
                                   "." + domain))) {
                       return true;
                   }
               }
               return false;
           }

       }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | 許容するドメインの一覧を取得し、保持する
       * - | (2)
         - | サブドメインを許容するかどうかを表す真理値を取得し、保持する
       * - | (3)
         - | \ ``null`` \ チェックは他のアノテーションを用いて行うため、\ ``null`` \ の場合は \ ``true`` \ を返す
       * - | (4)
         - | 許容されるドメインの一覧から一つずつ取り出し、メールアドレスのドメイン部と一致するかをチェックする。一つでも一致すれば \ ``true`` \ を返し、一つも一致しなければ \ ``false`` \ を返す

* Formクラスへのアノテーションの付与

  作成したアノテーションをアカウント新規作成のFormクラスのフィールドに付与する。

  .. code-block:: java

     package org.terasoluna.securelogin.app.account;

     // omitted

     public class AccountCreateForm implements Serializable {

         // omitted

         @NotNull
         @NotContainControlChars  // (1)
         @Size(min=4, max=128)
         private String username;

         // omitted

         @NotNull
         @NotContainControlChars
         @Size(min=1, max=128)
         @DomainRestrictedEmail(allowedDomains={ "domainexample.co.jp",
                  "somedomainexample.co.jp" }, allowSubDomain=true)  // (2)
         private String email;

         // omitted

         @NotNull
         @NotContainControlChars
         @DomainRestrictedURL(allowedDomains={ "jp" })  // (3)
         private String url;

         @UploadFileRequired
         @UploadFileNotEmpty
         @UploadFileMaxSize
         @FileExtension(extensions = { "jpg", "png", "gif" })  // (4)
         @FileNamePattern(pattern = "[a-zA-Z0-9_-]+\\.[a-zA-Z]{3}")  // (5)
         private MultipartFile image;

         @NotNull
         @NotContainControlCharsExceptNewlines  // (6)
         private String profile;

     }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | 制御文字が含まれないことをチェックする
     * - | (2)
       - | メールアドレスのドメインが "ntt.co.jp", "nttdata.co.jp" またはそのサブドメインであることチェックする
     * - | (3)
       - | URLのドメインが "jp" またはそのサブドメインであることをチェックする
     * - | (4)
       - | ファイルの拡張子が "jpg", "png", "gif" のどれかであることをチェックする
     * - | (5)
       - | ファイル名が『「半角英数字, "_", "-" の1文字以上の繰り返し」 + 「 "." 」 + 「半角英字3文字」』というパターンとなっていることをチェックする
     * - | (6)
       - | 改行コード以外の制御文字が含まれないことをチェックする

.. _audit-logging:

監査ログ出力
--------------------------------------------------------------------------------

実装する要件一覧
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
* :ref:`監査ログ出力 <sec-requirements>`

動作イメージ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. figure:: ./images/SecureLogin_logging.png
   :alt: Logging
   :width: 80%
   :align: center


監査目的で、アプリケーションに対していつ、誰が、どのような操作を行ったのかといった情報を確認できるようにするため、サービスクラスのメソッドを呼び出す際に、呼び出し日時、呼び出し元ユーザ名、メソッド名をログ出力する。
また、メソッド実行の結果として例外が発生しなければ操作成功としてログを出力し、例外が発生した場合は操作失敗としてログを出力する。

ログのフォーマットは以下の通りとする。

\ ``｛メソッド呼び出し日時｝｛スレッド｝｛呼び出し元ユーザ名｝｛X-Track｝｛ログレベル｝｛ロガー｝｛メッセージ｝`` \

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 30 70
   :class: longtable

   * - 項目
     - 説明
   * - | メソッド呼び出し日時
     - | サービスクラスのメソッドを呼び出した日時を"yyyy-MM-dd HH:mm:ss"形式で出力
   * - | スレッド
     - | ログ出力を行ったスレッド
   * - | 呼び出し元ユーザ名
     - | サービスクラスのメソッドを呼び出したSpring Securityのユーザ名を出力
       | 未ログインの場合には空欄とする
   * - | X-Track
     - | トレーサビリティ向上のために、リクエストごとに設定するID
       | 詳細は\ :doc:`../ArchitectureInDetail/GeneralFuncDetail/Logging` \を参照すること
   * - | ログレベル
     - | 出力されたログのレベル
       | 本アプリケーションにおいては\ ``info`` \レベルで出力する
   * - | ロガー
     - | ログを出力したロガー
   * - | メッセージ
     - | メソッド呼び出し時："[START SERVICE] ServiceClassName.methodName"
       | メソッド正常終了時："[COMPLETE SERVICE] ServiceClassName.methodName"
       | 例外発生時："[SERVICE THROWS EXCEPTION] ExceptionClassName.methodName"

.. raw:: latex

   \newpage

実装方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| ログにユーザ名を出力するために、Spring Securityの認証情報からユーザ名を取得する。 ユーザ名の取得およびログ出力は\ :doc:`../ArchitectureInDetail/GeneralFuncDetail/Logging` \ で解説している \ ``org.terasoluna.gfw.security.web.logging.UserIdMDCPutFilter`` \ を用いて実現する。
| さらに、本アプリケーションにおいて、リクエストに対する操作内容と操作結果をそれぞれ以下の通り定義し、ログに出力する。

* 操作内容：呼び出されたサービスクラスのメソッド名
* 操作結果：メソッドの処理を実行した結果例外が発生したか否か

| すべてのサービスクラスのメソッド呼び出しに対してログ出力を行うといった、横断的な機能を実現するためには、Springが提供するAOP(Aspect Oriented Programming)の機能を利用することができる。
| Springが提供しているAOPの実装方法は複数あるが、本アプリケーションでは `共通ライブラリ <https://github.com/terasolunaorg/terasoluna-gfw>`_ で提供しているロギング関連の部品の実装と合わせることを重視し、\ ``org.aopalliance.intercept.MethodInterceptor`` \を実装する方式を採用する。
| 具体的には以下の実装・設定を行うことで要件を実現する。

* \ ``UserIdMDCPutFilter`` \ を設定する

* メソッド呼び出し時および実行後にログ出力を行うアドバイスを作成する

* 上記で定義したアドバイスを\ ``@Service`` \ の付与されたクラスに対して適用するための設定を行う

 .. note::

    アドバイスとは、AOPにおいて指定されたタイミングで実行する処理のことを指す。
    また、アドバイスを織り込むことのできる箇所のことをジョインポイントと呼び、どのジョインポイントにアドバイスを織り込むかを定義したものポイントカットと呼ぶ。
    Springが提供するAOP機能に関しては、`公式ドキュメント - AOP <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/aop.html>`_ を参照すること。

コード解説
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

上記の実装方法に従って実装されたコードについて順に解説する。

* \ ``UserIdMDCPutFilter`` \ を設定する

  ログにSpring Securityの認証ユーザ名を出力するための設定を以下に示す。

  **spring-security.xml**

  .. code-block:: xml

     <!-- omitted -->

     <sec:http>
         <!-- omitted -->
         <sec:custom-filter ref="userIdMDCPutFilter" after="ANONYMOUS_FILTER" />
         <!-- omitted -->
     </sec:http>

     <!-- omitted -->
     <bean id="userIdMDCPutFilter" class="org.terasoluna.gfw.security.web.logging.UserIdMDCPutFilter">
     </bean>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | ユーザ情報が生成された後すぐにログ出力するために、Spring SecurityのFilter Chainに\ ``UserIdMDCPutFilter`` \ を設定する
         | \ ``UserIdMDCPutFilter`` \を設定することによって、MDCに\ ``USER`` \というキーで認証ユーザ名が追加される。

  **logback.xml**

  .. code-block:: xml

     <!-- omitted -->

     <appender name="AUDIT_LOG_FILE"
         class="ch.qos.logback.core.rolling.RollingFileAppender">  <!-- (1) -->
         <file>log/security-audit.log</file>
         <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
             <fileNamePattern>log/security-audit-%d{yyyyMMdd}.log</fileNamePattern>
             <maxHistory>7</maxHistory>
         </rollingPolicy>
         <encoder>
             <charset>UTF-8</charset>
             <pattern><![CDATA[date:%d{yyyy-MM-dd HH:mm:ss}\tthread:%thread\tUSER:%X{USER}\tX-Track:%X{X-Track}\tlevel:%-5level\tlogger:%-48logger{48}\tmessage:%msg%n]]></pattern>  <!-- (2) -->
         </encoder>
     </appender>

     <!-- omitted -->

     <logger 
        name="org.terasoluna.securelogin.domain.common.interceptor.ServiceCallLoggingInterceptor"
        additivity="false">  <!-- (3) -->
        <level value="info" />
        <appender-ref ref="AUDIT_LOG_FILE" />
     </logger>

     <!-- omitted -->

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | 監査ログ出力用のappenderを定義する
     * - | (2)
       - | pattern定義内に"USER:%X{USER}"を記述する
     * - | (3)
       - | 監査ログ出力用のloggerを定義する
         | \ ``org.terasoluna.securelogin.domain.common.interceptor.ServiceCallLoggingInterceptor`` \の実装については以降で説明する

* メソッド呼び出し時および実行後にログ出力を行うアドバイスを作成する

  操作内容と操作結果をログ出力するための実装および設定を以下に示す。

  .. code-block:: java

     package org.terasoluna.securelogin.domain.common.interceptor;

     // omitted

     public class ServiceCallLoggingInterceptor implements MethodInterceptor {  // (1)

       private static final Logger logger = LoggerFactory
               .getLogger(ServiceCallLoggingInterceptor.class);

       @Override
       public Object invoke(MethodInvocation invocation) throws Throwable {  // (2)
           String methodName = invocation.getMethod().getName();
           String className = invocation.getMethod().getDeclaringClass()
                   .getSimpleName();
           logger.info("[START SERVICE]{}.{}", className, methodName);  // (3)
           try {
               Object result = invocation.proceed();  // (4)
               logger.info("[COMPLETE SERVICE]{}.{}", className, methodName);  // (5)
               return result;  // (6)
           } catch (Throwable e) {
               logger.info("[SERVICE THROWS EXCEPTION]{}.{}", className,  // (7)
                       methodName);
               logger.info(Exception : {}, Message : {}, e.getClass().getName(),
                       e.getMessage()); // (8)
               throw e;  // (9)
           }
       }
   }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | メソッド呼び出しの前後で行う処理を記述するために、\ ``MethodInterceptor`` \ を実装する
     * - | (2)
       - | \ ``MethodInterceptor`` \ に定義されている、\ ``invoke`` \メソッドをオーバーライドする
         | 引数の \ ``org.aopalliance.intercept.MethodInvocation`` \ オブジェクトから、呼び出すメソッドの名前等の情報を取得することができる
     * - | (3)
       - | メソッド呼び出しを行う前に、呼び出すメソッド名をログ出力する
     * - | (4)
       - | 実際のメソッド呼び出しを行い、結果を取得する
     * - | (5)
       - | メソッド呼び出しの結果例外が発生しなければ、操作成功のログメッセージを出力する
     * - | (6)
       - | メソッド呼び出しの結果のオブジェクトを返す
     * - | (7)
       - | メソッド呼び出しの結果例外が発生した場合、操作失敗のログメッセージを出力する
     * - | (8)
       - | 発生した例外クラスと例外メッセージを出力する
         | 監査目的であることから、ログが冗長になることを避けるためスタックトレースは出力していない
     * - | (9)
       - | 発生した例外オブジェクトを投げる

  .. tip::

     本アプリケーションの例を拡張することで、メソッド呼び出し時の引数などより詳細な内容を出力することも可能である。
     ただし、その場合はハッシュ化されていないパスワード等がログ出力される可能性があるため、マスキングを行う等の対策が必要となることに注意すること。

* アドバイスを\ ``@Service`` \ の付与されたクラスに対して適用するための設定を行う

  アドバイスに対するポイントカットの設定を以下に示す。

  **secure-login-domain.xml**

  .. code-block:: xml

     <!-- omitted -->

     <bean id="serviceCallLoggingInterceptor"
         class="org.terasoluna.securelogin.domain.common.interceptor.ServiceCallLoggingInterceptor" />  <!-- (1) -->
     <aop:config>
         <aop:advisor advice-ref="serviceCallLoggingInterceptor"
             pointcut="@within(org.springframework.stereotype.Service)" />  <!-- (2) -->
     </aop:config>

     <!-- omitted -->

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | 上記で作成したアドバイスクラス(\ ``MethodInterceptor`` \の実装クラス)をBean定義する
     * - | (2)
       - | \ ``aop:advisor`` \ タグの \ ``advice-ref`` \ 属性にアドバイスが実装されているBeanを、 \ ``pointcut`` \ 属性にポイントカットをそれぞれ設定する
         | \ ``@within(org.springframework.stereotype.Service)`` \ というポイントカットの定義により、 \ ``@Service`` \ の付与されたクラスのメソッド呼び出しがアドバイスの対象となる

  .. note::

     SpringのAOPは、自動的に作成されたプロキシクラスがメソッド呼び出しをハンドリングする、プロキシ方式を採用している。
     プロキシ方式のAOPの制限として、可視性が\ ``public`` \以外のメソッドの呼び出しや、同一クラス内のメソッド呼び出しの際にはアドバイスが実行されない点に注意する必要がある。
     詳細は `公式ドキュメント <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/aop.html#aop-understanding-aop-proxies>`_ を参照すること。

  ログの出力結果を以下に示す。

  .. code-block:: console

     date:2016-08-18 13:45:42   thread:tomcat-http--7   USER:demo   X-Track:f514cc4159324ba28d8393f2c3062d89    level:INFO  logger:o.t.s.d.c.i.ServiceCallLoggingInterceptor        message:[START SERVICE]AccountSharedService.isInitialPassword
     date:2016-08-18 13:45:42   thread:tomcat-http--7   USER:demo   X-Track:f514cc4159324ba28d8393f2c3062d89    level:INFO  logger:o.t.s.d.c.i.ServiceCallLoggingInterceptor        message:[START SERVICE]PasswordHistorySharedService.findLatest
     date:2016-08-18 13:45:42   thread:tomcat-http--7   USER:demo   X-Track:f514cc4159324ba28d8393f2c3062d89    level:INFO  logger:o.t.s.d.c.i.ServiceCallLoggingInterceptor        message:[COMPLETE SERVICE]PasswordHistorySharedService.findLatest
     date:2016-08-18 13:45:42   thread:tomcat-http--7   USER:demo   X-Track:f514cc4159324ba28d8393f2c3062d89    level:INFO  logger:o.t.s.d.c.i.ServiceCallLoggingInterceptor        message:[COMPLETE SERVICE]AccountSharedService.isInitialPassword

  例外発生時のログは以下のようになる。ログイン前に行われた処理に対するログであるため、ユーザ名が表示されていない。

  .. code-block:: console

     date:2016-08-18 13:52:32   thread:tomcat-http--10  USER:   X-Track:1a37a9a280014216a300b61e2f4bbb66    level:INFO  logger:o.t.s.d.c.i.ServiceCallLoggingInterceptor        message:[SERVICE THROWS EXCEPTION]AccountSharedService.findOne
     date:2016-08-18 13:52:32   thread:tomcat-http--10  USER:   X-Track:1a37a9a280014216a300b61e2f4bbb66    level:INFO  logger:o.t.s.d.c.i.ServiceCallLoggingInterceptor        message:[SERVICE THROWS EXCEPTION]UserDetailsService.loadUserByUsername
     date:2016-08-18 13:52:32   thread:tomcat-http--10  USER:   X-Track:1a37a9a280014216a300b61e2f4bbb66    level:INFO  logger:o.t.s.d.c.i.ServiceCallLoggingInterceptor        message:user not found

おわりに
================================================================================

| 本章では、サンプルアプリケーションを題材としてセキュリティ対策の実装方法の例を説明した。
| 実際の開発においては、本アプリケーションにおける実装方法をそのまま利用できないケースも考えられるため、本章の内容を参考にしつつ要件に合わせてカスタマイズしたり別の方法を考えるようにしてほしい。

Appendix
================================================================================

.. _passay_overview:

Passay
--------------------------------------------------------------------------------

Passayはパスワード入力チェック機能とパスワード生成機能を提供するライブラリである。
PassayのAPIは以下の三つの主要コンポーネントで構成される。

* 検証規則

  パスワードが満たすべき条件の定義。パスワードの長さや含まれる文字種別等の一般的によく利用される規則についてはライブラリが提供するクラスを使用して容易に作成することができる。その他、必要な規則を自分で定義することもできる。

* 検証器

  検証規則に基づいて実際にパスワードのチェックを行うコンポーネント。複数の検証規則を一つの検証器に設定することができる。

* 生成器

  与えられた文字種別に関する検証規則に適合するパスワードを生成するコンポーネント。

Passayの機能を使用する場合は、pom.xmlに以下の定義を追加すること。

.. code-block:: xml

   <dependencies>
     <dependency>
         <groupId>org.passay</groupId>
         <artifactId>passay</artifactId>
         <version>1.1.0</version>
     </dependency>
   <dependencies>

.. _password_validation:

パスワード入力チェック
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Overview
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| Passayにおけるパスワード入力チェックの流れの概略図を以下に示す。

.. figure:: ./images/SecureLogin_passay.png
   :alt: Password Vaildation
   :width: 60%
   :align: center

.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``org.passay.PasswordData`` \のインスタンスを作成し、入力チェック対象のパスワードに関する情報を設定する。
       | \ ``PasswordData`` \は、パスワード、ユーザ名に加え、過去に使用したパスワードのリスト等をプロパティとして持つことができる。
       | 過去に使用したパスワード等は\ ``org.passay.PasswordData.Reference`` \のインスタンスとして保持する。
   * - | (2)
     - | 検証規則に従い、検証器を用いて\ ``PasswordData`` \に対する入力チェックを行う。
       | 検証規則は\ ``org.passay.Rule`` \の実装クラスのインスタンスとして作成する。検証器は\ ``org.passay.PasswordValidator`` \のインスタンスであり、複数の検証規則をプロパティとして持つことができる。
   * - | (3)
     - | 検証器による入力チェックの結果として\ ``org.passay.RuleResult`` \のインスタンスが作成される。
   * - | (4)
     - | \ ``RuleResult`` \ からパスワード入力チェックの結果を\ ``boolean`` \として得ることができる。また、検証器を使って\ ``RuleResult`` \からエラーメッセージが取得できる。

Passayが提供している検証規則のクラスの一部を以下の表に示す。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.40\linewidth}|p{0.40\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 40 40

   * - クラス名
     - 説明
     - 主なプロパティ
   * - | \ ``LengthRule`` \
     - | パスワード長の最小値、最大値を規定するための検証規則のクラス
     - | \ ``minimuxLength`` \ : パスワード長の最小値(\ ``int`` \)。コンストラクタまたはsetterで設定。
       | \ ``maximumLength`` \ : パスワード長の最大値(\ ``int`` \)。コンストラクタまたはsetterで設定。
   * - | \ ``CharacterRule`` \
     - | パスワードに含まれるべき文字種別と、その文字種別の最低文字数を規定するための検証規則のクラス
     - | \ ``characterData``\ : 文字種別(\ ``org.passay.CharacterData`` \)。コンストラクタで設定。
       | \ ``numberOfCharacters`` \ : 最低文字数(\ ``int`` \)。コンストラクタまたはsetterで設定。
   * - | \ ``CharacterCharacteristicsRule`` \
     - | 複数の\ ``CharacterRule`` \のうち、いくつ以上の規則を満たす必要があるかを規定するための検証規則のクラス
     - | \ ``rules``\ : 文字種別に関する検証規則のリスト(\ ``List<CharacterRule>`` \)。setterで設定。
       | \ ``numberOfCharacteristics`` \ : 満たすべき検証規則の数の最小値(\ ``int`` \)。setterで設定。
   * - | \ ``HistoryRule`` \
     - | パスワードが以前に使用したパスワードと一致していないことをチェックするための検証規則のクラス
     - | なし
   * - | \ ``UsernameRule`` \
     - | パスワードがユーザ名を含まないことをチェックするための検証規則のクラス
     - | \ ``matchBackwards`` \ : ユーザ名を逆にした文字列もチェックする(\ ``boolean`` \)。コンストラクタまたはsetterで設定。
       | \ ``ignoreCase`` \ : 大文字、小文字を区別しない(\ ``boolean`` \)。コンストラクタまたはsetterで設定。

この他にも、特定の文字を含む/含まないことのチェックや、正規表現によるチェックを行うための検証規則のクラス等が提供されている。
詳細は `<http://www.passay.org/>`_ を参照。

How to use
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ ``PasswordValidator`` \のコンストラクタに\ ``org.passay.Rule`` \のインスタンスのリストを渡すことによって、検証器を作成することができる。
検証規則を設定した検証器を以下のようにBeanとして定義しておくことでDIが可能となる。
尚、複数の検証規則をBean定義する場合、\ ``@Inject`` \と\ ``@Named`` \を併用することでBean名によるDIを行うこと。

.. code-block:: xml

   <!-- Password Rules. -->
   <bean id="upperCaseRule" class="org.passay.CharacterRule"> <!-- (1) -->
       <constructor-arg name="data">
           <util:constant static-field="org.passay.EnglishCharacterData.UpperCase" /> <!-- (2) -->
       </constructor-arg>
       <constructor-arg name="num" value="1" /> <!-- (3) -->
   </bean>
   <bean id="lowerCaseRule" class="org.passay.CharacterRule"> <!-- (4) -->
       <constructor-arg name="data">
           <util:constant static-field="org.passay.EnglishCharacterData.LowerCase" />
       </constructor-arg>
       <constructor-arg name="num" value="1" />
   </bean>
   <bean id="digitRule" class="org.passay.CharacterRule"> <!-- (5) -->
       <constructor-arg name="data">
           <util:constant static-field="org.passay.EnglishCharacterData.Digit" />
       </constructor-arg>
       <constructor-arg name="num" value="1" />
   </bean>

   <!-- Password Validator. -->
   <bean id="characterPasswordValidator" class="org.passay.PasswordValidator"> <!-- (6) -->
       <constructor-arg name="rules">
           <list>
               <ref bean="upperCaseRule" />
               <ref bean="lowerCaseRule" />
               <ref bean="digitRule" />
           </list>
       </constructor-arg>
   </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.60\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | パスワードに含まれるべき文字種別と、その文字種別の最低文字数を規定するための検証規則のBean定義
   * - | (2)
     - | 文字種別を指定する。ここでは、\ ``org.passay.EnglishCharacterData.UpperCase`` \を渡しているため、半角英大文字に関する検証規則となる。
   * - | (3)
     - | 文字数を指定する。ここでは"1"を渡しているため、半角英大文字を一文字以上含むことをチェックする検証規則となる。
   * - | (4)
     - | (1)-(3)と同様だが、文字種別として\ ``org.passay.EnglishCharacterData.UpperCase`` \を渡しているため、半角英小文字を一文字以上含むことをチェックする検証規則のBean定義となる。
   * - | (5)
     - | (1)-(3)と同様だが、文字種別として\ ``org.passay.EnglishCharacterData.Digit`` \を渡しているため、半角数字を一文字以上含むことをチェックする検証規則のBean定義となる。
   * - | (6)
     - | 検証器のBean定義。コンストラクタに検証規則のリストを渡す。

作成した検証器を使用してパスワード入力チェックを行う。

.. code-block:: java

   @Inject
   PasswordValidator characterPasswordValidator;

   // omitted

   public void validatePassword(String password) {

       PasswordData pd = new PasswordData(password); // (1)
       RuleResult result = characterPasswordValidator.validate(pd); // (2)
       if (result.isValid()) { // (3)
          logger.info("Password is valid");
       } else {
          logger.error("Invalid password:");
          for (String msg : characterPasswordValidator.getMessages(result)) { // (4)
              logger.error(msg);
          }
       }

   }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 検証対象のパスワードを \ ``PasswordData`` \ のコンストラクタに渡し、インスタンスを作成する。
   * - | (2)
     - | \ ``PasswordValidator`` \ の \ ``validate`` \ メソッドに \ ``PasswordData`` \を引数として渡し、パスワード入力チェックを実行する。
   * - | (3)
     - | \ ``RuleResult`` \ の \ ``isValid`` \ メソッドを使用して、パスワード入力チェックの結果を真理値で取得する。
   * - | (4)
     - | \ ``PasswordValidator`` \ の \ ``getMessages`` \ メソッドに \ ``RuleResult`` \を引数として渡し、エラーメッセージを取得する。

.. _password_generation:

パスワード生成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Overview
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| Passayにおけるパスワード生成機能では、パスワードの生成器と生成規則を用いる。生成器は\ ``org.passay.PasswordGenerator`` \のインスタンスであり、生成規則は文字種別に関する検証規則(\ ``org.passay.CharacterRule`` \)のリストである。
| 生成器のメソッドに生成するパスワードの長さと生成規則を引数として与えることで、生成規則を満たしたパスワードが生成される。

How to use
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

生成規則に含まれる、文字種別に関する検証規則の作成方法は、:ref:`password_validation` と同様である。
生成規則と生成器を以下のようにBeanとして定義しておくことでDIが可能となる。

.. code-block:: xml

   <!-- Password Rules. -->
   <bean id="upperCaseRule" class="org.passay.CharacterRule"> <!-- (1) -->
       <constructor-arg name="data">
           <util:constant static-field="org.passay.EnglishCharacterData.UpperCase" /> <!-- (2) -->
       </constructor-arg>
       <constructor-arg name="num" value="1" /> <!-- (3) -->
   </bean>
   <bean id="lowerCaseRule" class="org.passay.CharacterRule"> <!-- (4) -->
       <constructor-arg name="data">
           <util:constant static-field="org.passay.EnglishCharacterData.LowerCase" />
       </constructor-arg>
       <constructor-arg name="num" value="1" />
   </bean>
   <bean id="digitRule" class="org.passay.CharacterRule"> <!-- (5) -->
       <constructor-arg name="data">
           <util:constant static-field="org.passay.EnglishCharacterData.Digit" />
       </constructor-arg>
       <constructor-arg name="num" value="1" />
   </bean>

    <!-- Password Generator. -->
    <bean id="passwordGenerator" class="org.passay.PasswordGenerator" /> <!-- (6) -->
    <util:list id="passwordGenerationRules"> <!-- (7) -->
        <ref bean="upperCaseRule" />
        <ref bean="lowerCaseRule" />
        <ref bean="digitRule" />
    </util:list>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | パスワードに含まれるべき文字種別と、その文字種別の最低文字数を規定するための検証規則のBean定義
   * - | (2)
     - | 文字種別を指定する。ここでは、\ ``org.passay.EnglishCharacterData.UpperCase`` \を渡しているため、半角英大文字に関する検証規則となる。
   * - | (3)
     - | 文字数を指定する。ここでは"1"を渡しているため、半角英大文字を一文字以上含むことをチェックする検証規則となる。
   * - | (4)
     - | (1)-(3)と同様だが、文字種別として\ ``org.passay.EnglishCharacterData.UpperCase`` \を渡しているため、半角英小文字を一文字以上含むことをチェックする検証規則のBean定義となる。
   * - | (5)
     - | (1)-(3)と同様だが、文字種別として\ ``org.passay.EnglishCharacterData.Digit`` \を渡しているため、半角数字を一文字以上含むことをチェックする検証規則のBean定義となる。
   * - | (6)
     - | 生成器のBean定義
   * - | (7)
     - | 生成規則のBean定義。(1)-(5)で定義した、文字種別に関する検証規則のリストとして定義する。

作成した生成器と生成規則を使用してパスワード生成を行う。

.. code-block:: java

   @Inject
   PasswordGenerator passwordGenerator;

   @Resource(name = "passwordGenerationRules")
   List<CharacterRule> passwordGenerationRules;

   // omitted

   public void generatePassword() {

       String password = passwordGenerator.generatePassword(10, passwordGenerationRules); // (1)

   }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``PasswordGenerator`` \の\ ``generatePassword`` \メソッドに、生成するパスワードの長さと生成規則を引数として渡すと、生成規則を満たしたパスワードが生成される。

.. tip::

   Bean定義したコレクションをDIする際には、\ ``@Inject`` \ + \ ``@Named`` \では期待した動作をしない。
   そのため、代わりに\ ``@Resource`` \を使用してBean名でDIする。

.. raw:: latex

   \newpage

