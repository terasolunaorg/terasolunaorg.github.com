.. _OAuth:

OAuth
================================================================================

.. only:: html

 .. contents:: 目次
    :local:

.. _OAuthOverview:

Overview
--------------------------------------------------------------------------------
本節では、OAuth 2.0の概要とSpringプロジェクトの一つである
Spring Security OAuthを使用してOAuth 2.0の仕様に沿った認可制御機能を実装する方法について説明する。

.. tip:: **Spring Security OAuth のリファレンス**

    Spring Security OAuthは、本ガイドラインで紹介していない機能も提供している。
    Spring Security OAuthについて詳しく知りたい場合は、\ `OAuth 2 Developers Guide <https://projects.spring.io/spring-security-oauth/docs/oauth2.html>`_\ を参照されたい。

|

.. _OAuthAboutOAuth2.0:

OAuth 2.0とは
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

OAuth 2.0とは、サードパーティー製アプリケーションがHTTPサービスを利用する際に、
サーバ上の保護されたリソースに対するアクセス範囲の指定を可能にするための認可フレームワークのことである。

OAuth 2.0はRFCとして仕様化されており、関連する複数の技術仕様から構成されている。

以下にOAuth 2.0の主要な仕様を示す。


.. tabularcolumns:: |p{0.15\linewidth}|p{0.30\linewidth}|p{0.55\linewidth}|
.. list-table:: **OAuth 2.0の主要仕様**
    :header-rows: 1
    :widths: 15 30 55

    * - RFC
      - 概要
      - 説明
    * - | RFC 6749
      - | \ `The OAuth 2.0 Authorization Framework <http://tools.ietf.org/html/rfc6749>`_\
      - | 用語や認可方式などの、OAuth 2.0としてのもっとも基本的な内容が記載されている技術仕様。
    * - | RFC 6750
      - | \ `Bearer Token Usage <https://tools.ietf.org/html/rfc6750>`_\
      - | RFC 6749に記載されている認可制御を実現する場合に利用する、「署名なしアクセストークン」
          （以降、アクセストークンと表す）のサーバ間の受け渡し方法に関する技術仕様。
        | アクセストークンについては後述する。
    * - | RFC 6819
      - | \ `Threat Model and Security Considerations <https://tools.ietf.org/html/rfc6819>`_\
      - | OAuth 2.0を使用するうえで考慮が必要となるセキュリティ要件に関する技術仕様。
        | 本ガイドラインでは検討項目の具体的な説明は割愛する。
    * - | RFC 7519
      - | \ `JSON Web Token (JWT) <https://tools.ietf.org/html/rfc7519>`_\
      - | 署名が可能なJSONを含んだトークンであるJSON Web Token (JWT)に関する技術仕様。
    * - | RFC 7523
      - | \ `JSON Web Token (JWT) Profile for OAuth 2.0 Client Authentication and Authorization Grants <https://tools.ietf.org/html/rfc7523>`_\
      - | RFC 6749に記載されている認可制御の実現する場合に利用するアクセストークンとして、RFC 6819で定められているJWTを利用する方法に関する技術仕様。
    * - | RFC 7009
      - | \ `OAuth 2.0 Token Revocation <https://tools.ietf.org/html/rfc7009>`_\
      - | トークンの無効化を行う追加エンドポイントに関する技術仕様。

|

従来のクライアントサーバ型の認証モデルでは、サードパーティー製アプリケーションはサーバ上の
保護されたリソースにアクセスするために、ユーザーの認証情報（ユーザー名とパスワードなど）を利用して認証を行う。

つまり、ユーザーは、サードパーティー製アプリケーションにリソースへのアクセス権を与えるために
自身の認証情報をサードパーティと共有する必要があるが、
これはサードパーティー製アプリケーションに不具合や悪意のある操作などが存在した場合に、
ユーザの意図しないアクセスや情報漏洩等のリスクにつながる。

.. figure:: ./images/OAuth_TraditionalAuthenticationModel.png
    :width: 100%


これに対し、OAuth 2.0では認証はユーザが直接行い、サードパーティー製アプリケーションには
「アクセストークン」と呼ばれる認証済みリクエストを行うための情報を払い出すことで、
サードパーティーに認証情報を共有することなくリソースへアクセスすることが可能となる。

また、アクセストークン発行時にリソースに対するアクセス範囲（スコープ）を指定可能とすることで
従来のクライアントサーバ型の認証モデルと比較してより柔軟なアクセス制御を実現している。


.. figure:: ./images/OAuth_OAuthAuthenticationModel.png
    :width: 100%

|

.. _OAuthArchitecture:

OAuth 2.0のアーキテクチャ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ここではOAuth 2.0が定義するロール、スコープ、認可グラント、及びプロトコルフローについて説明する。
OAuth 2.0ではスコープや認可グラントという概念を定義しており、これらの概念を使用して認可の仕様を定めている。

|

.. _OAuthRole:

ロール
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
OAuth 2.0ではロールとして以下の4つを定義している。

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: **OAuth 2.0におけるロール**
    :header-rows: 1
    :widths: 25 75
    :class: longtable

    * - ロール名
      - 説明
    * - | リソースオーナ
      - | 保護されたリソースへのアクセスを許可するロール。人（エンドユーザ）など。
    * - | リソースサーバ
      - | 保護されたリソースを提供するロール。Webサーバ。
    * - | 認可サーバ
      - | リソースオーナの認証と、アクセストークン（クライアントがリソースサーバにアクセスするときに必要な情報）の発行を行うロール。Webサーバ。
    * - | クライアント
      - | リソースオーナの認可を得て、リソースオーナの代理として保護されたリソースに対してリクエストを行うロール。Webアプリケーションなど。クライアントの情報は事前に認可サーバに登録され、認可サーバ内で一意な情報であるクライアントIDにより管理される。
        | OAuth 2.0ではクライアントクレデンシャル（クライアントの認証情報）の機密性を維持できる能力に基づき、クライアントタイプとして以下の2つを定義している。
        | (1) コンフィデンシャル
        |     クライアントクレデンシャルの機密性を維持することができるクライアント。
        | (2) パブリック
        |     リソースオーナのデバイス上で実行されるクライアントのように、クライアントクレデンシャルの機密性を維持することができず、かつ他の手段を用いたセキュアなクライアント認証が行えないクライアント。
        |
        | また、OAuth 2.0ではクライアントとして以下のような例を考慮して設計されている
        | (1) Webアプリケーション（web application）
        |     Webサーバー上で実行されるクライアント（コンフィデンシャル）。
        | (2) ユーザエージェントベースアプリケーション（user-agent-based application）
        |     クライアントコードがWebサーバーからダウンロードされリソースオーナのユーザエージェント内で実行されるクライアント（パブリック）。Javascriptアプリケーションなど。
        | (3) ネイティブアプリケーション（native application）
        |     リソースオーナのデバイス上にインストールされ実行されるクライアント（パブリック）。

|

.. _OAuthScope:

スコープ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
OAuth 2.0では保護されたリソースに対するアクセスを制御する方法としてスコープという概念を使用している。

認可サーバはクライアントからの要求に対し、認可サーバのポリシーまたはリソースオーナの
指示に基づいてアクセストークンにスコープを含め、保護されたリソースに対する
アクセス権（読み込み権限、書き込み権限など）を指定することが出来る。

|

.. _OAuthProtocolFlow:

プロトコルフロー
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
OAuth 2.0では、以下のような流れでリソースへのアクセスを行う。

.. figure:: ./images/OAuth_ProtocolFlow.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **OAuth 2.0のプロトコルフロー**
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | クライアントはリソースオーナに対して認可を要求する。上の図ではリソースオーナに
        | 直接要求を行ってるが、認可サーバを経由して行うほうが望ましい。
        | 後述するグラントタイプの中では認可コードグラントとインプリシットグラントが
        | 認可サーバを経由してリソースオーナに要求を行うフローになっている。
    * - | (2)
      - | クライアントはリソースオーナからの認可を表すクレデンシャルとして認可グラント（後述）を受け取る。
    * - | (3)
      - | クライアントは、認可サーバーに対して自身の認証情報とリソースオーナが与えた認可グラントを提示することで、アクセス
        | トークンを要求する。
    * - | (4)
      - | 認可サーバはクライアントを認証し、認可グラントの正当性を確認する。認可グラントが正当な場合、
        | アクセストークンを発行する。
    * - | (5)
      - | クライアントはリソースサーバの保護されたリソースへリクエストを行い、発行されたアクセス
        | トークンにより認証する。
    * - | (6)
      - | リソースサーバはアクセストークンの正当性を確認し、正当な場合、リクエストを受け入れる。

.. note::

    OAuth 1.0で不評だった署名とトークン交換の複雑な仕組みを簡略化するために、OAuth 2.0ではアクセストークンを扱うリクエストはHTTPS通信で行うことを必須としている。
    （HTTPS通信を使用することでアクセストークンの盗聴を防止する）

|

.. _AuthorizationGrant:

認可グラント
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
認可グラントは、リソースオーナからの認可を表し、クライアントがアクセストークンを取得する際に用いられる。
OAuth 2.0では、グラントタイプとして以下の4つを定義しているが、クレデンシャル項目を追加するなどの独自拡張を行うこともできる。

クライアントは4つのグラントタイプのいずれかにより、認可サーバへアクセストークンを要求し、取得したアクセストークンでリソースサーバにアクセスする。
認可サーバはサポートするグラントタイプを必ず1つ以上定義しており、その中から使用するグラントタイプをクライアントからの認可リクエストによって決定する。

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: **OAuth 2.0における認可グラント**
    :header-rows: 1
    :widths: 25 75

    * - グラントタイプ
      - 説明
    * - | 認可コードグラント
      - | 認可コードグラントのフローでは、認可サーバがクライアントとリソースオーナの仲介となって認可コードをクライアントへ発行し、クライアントが認可コードを認可サーバに渡すことでアクセストークンを発行する。
        | 認可サーバが発行した認可コードを使用してアクセストークンを発行するため、クライアントへリソースオーナのクレデンシャルを共有する必要がない。
        | 認可コードグラントはWebアプリケーションのように、コンフィデンシャルなクライアントがOAuth 2.0を利用する際に使用する。
    * - | インプリシットグラント
      - | インプリシットグラントのフローでは、認可コードグラントと同様に認可サーバが仲介するが、認可コードの代わりに直接アクセストークンを発行する。
        | アクセストークンはURL中にエンコードされるため、リソースオーナや同一デバイス上の他のアプリケーションに漏えいする可能性があるほか、
          クライアントの認証を行わないことから、他のクライアントに対して発行されたアクセストークンを不正に用いた成りすまし攻撃のリスクがある。
        | インプリシットグラントはJavascriptで実装されたクライアントなどの、クライアントタイプがパブリックである場合のみ使用すること。
    * - | リソースオーナパスワードクレデンシャルグラント
      - | リソースオーナパスワードクレデンシャルグラントのフローでは、クライアントがリソースオーナの認証情報を認可グラントとして使用して、直接アクセストークンを発行する。
        | クライアントへリソースオーナのクレデンシャルを共有する必要があるため、クライアントの信頼性が低い場合、クレデンシャルの不正利用や漏洩のリスクがある。
        | リソースオーナパスワードクレデンシャルグラントはリソースオーナとクライアントの間で高い信頼があり、かつ他のグラントタイプが利用できない場合にのみ使用すること。
    * - | クライアントクレデンシャルグラント
      - | クライアントクレデンシャルグラントのフローでは、クライアントの認証情報を認可グラントとして使用して、直接アクセストークンを発行する。
        | クライアントがリソースオーナであるような場合に使用する。

|

認可コードグラントのフローを以下に示す。

.. figure:: ./images/OAuth_AuthorizationCodeGrant.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **認可コードグラントフロー**
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | リソースオーナは、ユーザエージェントを介してクライアントが提供するリソースサーバの保護されたリソースが必要なページにアクセスする。
        | クライアントはリソースオーナから認可の取得を行うために、リソースオーナのユーザエージェントを認可サーバの認可エンドポイントにアクセスさせる。
        | このとき、クライアントは自身を識別するためのクライアントIDと、オプションとしてリソースに要求するスコープ、認可サーバが認可処理後にユーザエージェントを戻すリダイレクトURL、stateをリクエストパラメータに含める。
        | stateはユーザエージェントに紐付くランダムな値であり、一連のフローが同じユーザエージェントで実行されたことを保証するために利用される(CSRF対策)。
    * - | (2)
      - | ユーザエージェントは、クライアントに指示された認可サーバの認可エンドポイントにアクセスする。
        | 認可サーバはユーザエージェント経由でリソースオーナを認証し、リクエストパラメータのクライアントID、スコープ、リダイレクトURLを元に、自身に登録済みのクライアント情報と比較しパラメータの正当性確認を行う。
        | 確認完了後、アクセス要求の許可/拒否をリソースオーナにたずねる。
    * - | (3)
      - | リソースオーナはアクセス要求の許可/拒否を認可サーバに送信する。
        | リソースオーナがアクセスを許可した場合、認可サーバの認可エンドポイントは認可コードを発行し、リクエストパラメータのリダイレクトURLを用いてユーザエージェントをクライアントにリダイレクトさせる指示を出す。
        | その際、リダイレクトURLのリクエストパラメータとして、認可コードをリダイレクトURLに付与する。
    * - | (4)
      - | ユーザエージェントは認可コードが付与されたリダイレクトURLにアクセスする。
        | クライアントの処理が完了するとリソースオーナにレスポンスを返却する。
    * - | (5)
      - | クライアントはアクセストークンを要求するために、認可コードを認可サーバのトークンエンドポイントに送信する。
        | 認可サーバのトークンエンドポイントはクライアントの認証と認可コードの正当性の検証を行い、正当である場合アクセストークンと任意でリフレッシュトークンを発行する。
        | リフレッシュトークンはアクセストークンが無効化された、または期限切れの際に新しいアクセストークンを発行するために使用される。
    * - | (6)
      - | クライアントは取得したアクセストークンでリソースサーバにアクセスする。
        | リソースサーバはアクセストークンの正当性を確認し、正当な場合、リクエストを処理してレスポンスをクライアントに返却する。

|

インプリシットグラントのフローを以下に示す。

.. figure:: ./images/OAuth_ImplicitGrant.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **インプリシットグラントフロー**
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | リソースオーナは、ユーザエージェントを介してクライアントが提供するリソースサーバの保護されたリソースが必要なページにアクセスする。
        | クライアントはリソースオーナから認可の取得とアクセストークンの発行を行うために、リソースオーナのユーザエージェントを認可サーバの認可エンドポイントにアクセスさせる。
        | このとき、クライアントは自身を識別するためのクライアントIDと、オプションとしてリソースに要求するスコープ、認可サーバが認可処理後にユーザエージェントを戻すリダイレクトURL、stateをリクエストパラメータに含める。
        | stateはユーザエージェントに紐付くランダムな値であり、一連のフローが同じユーザエージェントで実行されたことを保証するために利用される(CSRF対策)。
    * - | (2)
      - | ユーザエージェントは、クライアントに指示された認可サーバの認可エンドポイントにアクセスする。
        | 認可サーバはユーザエージェント経由でリソースオーナを認証し、リクエストパラメータのクライアントID、スコープ、リダイレクトURLを元に、自身に登録済みのクライアント情報と比較しパラメータの正当性確認を行う。
        | 確認完了後、アクセス要求の許可/拒否をリソースオーナにたずねる。
    * - | (3)
      - | リソースオーナはアクセス要求の許可/拒否を認可サーバに送信する。
        | リソースオーナがアクセスを許可した場合、認可サーバの認可エンドポイントはリクエストパラメータのリダイレクトURLを用いてユーザエージェントをクライアントリソースにリダイレクトさせる指示を出す。その際、アクセストークンをリダイレクトURLのURLフラグメントに付与する。
    * - | (4)
      - | ユーザエージェントはリダイレクトの指示に従い、クライアントリソースにリクエストを送信する。このとき、URLフラグメントの情報をローカルで保持し、リダイレクトの際にはURLフラグメントを送信しない。
        | クライアントリソースにアクセスすると、Webページ（通常は埋め込みスクリプトを含むHTMLドキュメント）が返却される。
        | ユーザエージェントはWebページに含まれるスクリプトを実行し、ローカルで保持していたURLフラグメントからアクセストークンを抽出する。
    * - | (5)
      - | ユーザエージェントはアクセストークンをクライアントに渡す。
        | クライアントの処理が完了するとリソースオーナにレスポンスを返却する。
    * - | (6)
      - | クライアントは取得したアクセストークンでリソースサーバにアクセスする。
        | リソースサーバはアクセストークンの正当性を確認し、正当な場合、リクエストを処理してレスポンスをクライアントに返却する。

|

リソースオーナパスワードクレデンシャルグラントのフローを以下に示す。

.. figure:: ./images/OAuth_ResourceOwnerPasswordCredentialsGrant.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **リソースオーナパスワードクレデンシャルグラントフロー**
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | リソースオーナがクライアントにクレデンシャル（ユーザー名、パスワード）を提供する。
    * - | (2)
      - | クライアントはアクセストークンを要求するために、認可サーバのトークンエンドポイントにアクセスする。
        | このとき、クライアントはリソースオーナから指定されたクレデンシャルとリソースに要求するスコープをリクエストパラメータに含める。
    * - | (3)
      - | 認可サーバのトークンエンドポイントはクライアントを認証し、リソースオーナのクレデンシャルを検証する。正当である場合アクセストークンを発行する。

|

クライアントクレデンシャルグラントのフローを以下に示す。

.. figure:: ./images/OAuth_ClientCredentialsGrant.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **クライアントクレデンシャルグラントフロー**
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | クライアントはアクセストークンを要求するために、認可サーバのトークンエンドポイントにアクセスする。
        | このとき、クライアントはクライアント自身のクレデンシャルを含めてアクセストークンを要求する。
    * - | (2)
      - | 認可サーバのトークンエンドポイントはクライアントを認証し、認証に成功した場合アクセストークンを発行する。

|

.. _AccessTokenLifeCycle:

アクセストークンのライフサイクル
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
アクセストークンはクライアントが提示する認可グラントの正当性を認可サーバが確認することで発行される。
発行されたアクセストークンは、認可サーバのポリシーまたはリソースオーナの指示に基づいたスコープが与えられ、保護されたリソースに対するアクセス権を得る。
アクセストークンは発行時に有効期限が設定され、有効期限切れとなると保護されたリソースに対するアクセス権を失効される。

アクセストークンの発行から失効までの流れは以下のようになる。

.. figure:: ./images/OAuth_LifeCycleOfAccessToken.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **アクセストークンの発行から失効までのフロー**
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | クライアントが認可グラントを提示し、アクセストークンを要求する。
    * - | (2)
      - | 認可サーバはクライアントが提示した認可グラントを確認し、アクセストークンを発行する。
    * - | (3)
      - | クライアントはアクセストークンを提示し、リソースサーバの保護されたリソースを要求する。
    * - | (4)
      - | リソースサーバはクライアントが提示したアクセストークンの正当性を検証し、正当であればリソースサーバの保護されたリソースに対して処理を行う。
    * - | (5)
      - | クライアントはアクセストークン（有効期限切れ）を提示し、リソースサーバの保護されたリソースを要求する。
    * - | (6)
      - | リソースサーバはクライアントが提示したアクセストークンの正当性を検証し、アクセストークンの有効期限が切れている場合はエラーを返却する。

| アクセストークンが有効期限切れとなると保護されたリソースに対するアクセス権を失効されるが、アクセストークンが有効期限切れとなる前にアクセストークンを無効化し保護されたリソースに対するアクセス権を失効させることも可能である。
| アクセストークンが有効期限切れとなる前に無効化する場合、クライアントより認可サーバにトークンの取り消し依頼を行う。無効化されたアクセストークンは保護されたリソースに対するアクセス権を失効される。

| 

| アクセストークンが有効期限切れとなった場合、クライアントがアクセストークンを再取得するためには認可サーバへ認可グラントの再提示を行い、認可サーバによる正当性の再確認が必要になる。
  そのため、アクセストークンの有効期限を短く設定した場合はユーザビリティが下がってしまう。一方で、アクセストークンの有効期限を長く設定した場合はアクセストークンの漏洩、漏洩時に悪用されるリスクが高まってしまう。
  
| ユーザビリティを下げずに漏洩、漏洩時のリスクを下げるためにはリフレッシュトークンが用いられる。
  リフレッシュトークンはアクセストークンが無効化されたあるいは期限切れの際、認可グラントの再提示を行うことなく新しいアクセストークンを取得するために利用される。
  リフレッシュトークンも発行時に有効期限が設定され、リフレッシュトークンが有効期限切れとなった場合はアクセストークンの再発行ができなくなる。
| アクセストークンの有効期限に短い期間を設定し、リフレッシュトークンの有効期限に長い期間を設定することで、短いサイクルでアクセストークンが再発行されユーザビリティを保ちつつアクセストークン漏洩及び漏洩時の悪用のリスクも抑えることができる。

| リフレッシュトークンの発行はオプションであり、認可サーバーの判断に委ねられる。

リフレッシュトークンによるアクセストークンの再発行の流れは以下のようになる。

.. figure:: ./images/OAuth_LifeCycleOfAccessTokenWithRefreshToken.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **アクセストークンの発行から再発行までのフロー**
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | クライアントが認可グラントを提示し、アクセストークンを要求する。
    * - | (2)
      - | 認可サーバはクライアントが提示した認可グラントを確認し、アクセストークンとリフレッシュトークンを発行する。
    * - | (3)
      - | クライアントはアクセストークンを提示し、リソースサーバの保護されたリソースを要求する。
    * - | (4)
      - | リソースサーバはクライアントが提示したアクセストークンの正当性を検証し、正当であればリソースサーバの保護されたリソースに対して処理を行う。
    * - | (5)
      - | クライアントはアクセストークン（有効期限切れ）を提示し、リソースサーバの保護されたリソースを要求する。
    * - | (6)
      - | リソースサーバはクライアントが提示したアクセストークンの正当性を検証し、アクセストークンの有効期限が切れている場合はエラーを返却する。
    * - | (7)
      - | リソースサーバよりアクセストークンの有効期限切れエラーが返却された場合、クライアントはリフレッシュトークンを提示することで新しいアクセストークンを要求する。
    * - | (8)
      - | 認可サーバはクライアントが提示しリフレッシュトークンの正当性を検証し、正当であればアクセストークンとオプションでリフレッシュトークンを発行する。

.. _SpringSecurityOAuthArchitecture:

Spring Security OAuthのアーキテクチャ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Spring Security OAuthは、OAuth 2.0で定義されているロールのうち、認可サーバ、リソースサーバ、クライアントの3つのロールをSpringアプリケーションとして構築する際に必要となる機能を提供するライブラリである。
Spring Security OAuthは、Spring Framework(Spring MVC)やSpring Securityが提供する機能と連携して動作する仕組みになっており、Spring Security OAuthが提供するデフォルト実装を適切にコンフィギュレーション（Bean定義）するだけで、認可サーバ、リソースサーバ、クライアントを構築することができる。
また、Spring FrameworkやSpring Securityと同様に数多くの拡張ポイントが用意されており、Spring Security OAuthが提供するするデフォルト実装で実現できない要件を組み込むことができるようになっている。

なお、各ロール間のリクエストに対する認証・認可にはSpring Securityが提供する機能を利用するため、そちらの詳細は\ :doc:`../../Security/Authentication`\ 及び \ :doc:`../../Security/Authorization`\ を参照されたい。 

Spring Security OAuthを使用して認可サーバ、リソースサーバ、クライアントを構築した場合、以下のような流れで処理が行われる。

.. figure:: ./images/OAuth_OAuth2Architecture.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **Spring Security OAuthのフロー**
    :header-rows: 1
    :widths: 10 90


    * - 項番
      - 説明
    * - | (1)
      - | リソースオーナはユーザエージェントを介してクライアントへアクセスする。
        | クライアントはサービスより\ ``OAuth2RestTemplate``\ の呼び出しを行う。
        | 認可エンドポイントにアクセスする認可グラントの場合、ユーザエージェントへ認可サーバの認可エンドポイントへリダイレクトさせるよう指示する。
    * - | (2)
      - | ユーザエージェントは認可サーバの認可エンドポイントである\ ``AuthorizationEndpoint``\ へアクセスする。
        | \ ``AuthorizationEndpoint``\ はリソースオーナへ認可を問い合わせる画面を表示させる。
        | リソースオーナはクライアントにスコープに対して認可を行い、\ ``AuthorizationEndpoint``\ へアクセスする。
        | \ ``AuthorizationEndpoint``\ は、認可グラントが認可コードグラントである場合は認可コードを、インプリシットグラントである場合はアクセストークンを発行する。
        | 発行した認可コードまたはアクセストークンは、リダイレクトを使用してユーザエージェント経由でクライアントに渡される。
    * - | (3)
      - | クライアントは\ ``OAuth2RestTemplate``\ より認可サーバのトークンエンドポイントである\ ``TokenEndpoint``\ へアクセスする。
        | \ ``TokenEndpoint``\は\ ``AuthorizationServerTokenService``\ を呼び出しアクセストークンを発行する。発行したアクセストークンはクライアントへの応答として渡される。
        | 認可グラントが認可コードグラントである場合は認可コードを認可サーバに提示する。アクセストークン発行前に\ ``TokenEndpoint``\で認可コードの正当性を検証される。
    * - | (4)
      - | クライアントは(2)または(3)で取得したアクセストークンを指定して \ ``OAuth2RestTemplate``\ よりリソースサーバにアクセスする。
        | リソースサーバは\ ``OAuth2AuthenticationManager``\ を呼び出し、\ ``ResourceServerTokenServices``\ を介してアクセストークンに紐づく認証情報を取得する。また、認証情報の取得時にアクセストークンを検証する。
        | アクセストークンの検証に成功した場合、クライアントからのリクエストに応じたリソースを返却する。


.. note::

    前述のとおり、OAuth 2.0では各エンドポイントにおいてHTTPS通信の使用を前提としているが、HTTPS通信を使用するのがSSLアクセラレータやWebサーバまでの場合や、
    ロードバランサを使用して複数のAPサーバに分散させる場合がある。
    リソースオーナによって認可された後にクライアントに認可コードまたは、アクセストークンを連携するためのリダイレクトURLを組み立てる際に、
    SSLアクセラレータやWebサーバ、ロードバランサを指し示すリダイレクトURLを組み立てる必要がある。
    
    Spring(Spring Security OAuth)では以下のいずれかのヘッダを使用してリダイレクト用のURLを組み立てる仕組みが提供されている。
    
    * Forwardedヘッダ
    * X-Forwarded-Hostヘッダ、X-Forwarded-Portヘッダ、X-Forwarded-Protoヘッダ

    SSLアクセラレータやWebサーバ、ロードバランサ側で上記ヘッダを付与するように設定し、Spring(Spring Security OAuth)が正しいリダイレクトURLを生成できるようにする必要がある。
    これを行わない場合、認可コードグラントやインプリシットグラントにおいて行うリクエストパラメータ（リダイレクトURL）の検証に失敗する可能性がある。

.. tip::

    Spring Security OAuthが提供するエンドポイントはSpring MVCの機能を拡張して実現している。Spring Security OAuthが提供するエンドポイントには\ ``@FrameworkEndpoint``\ アノテーションがクラスに設定されている。
    これは\ ``@Controller``\ アノテーションで開発者がコンポーネントとして登録したクラスと競合させないためである。
    また、\ ``@FrameworkEndpoint``\ アノテーションでコンポーネントとして登録されたエンドポイントは、\ ``RequestMappingHandlerMapping``\ の拡張クラスである\ ``FrameworkEndpointHandlerMapping``\ がエンドポイントの\ ``@RequestMapping``\ アノテーションを読み取り、URLと合致する\ ``@FrameworkEndpoint``\ のメソッドを、Handlerクラスとして扱っている。


|

.. _AuthorizationServer:

認可サーバ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
認可サーバは、リソースオーナの認証と、認証後のリソースオーナからの認可の取得、およびアクセストークンの発行を行う。

OAuth 2.0ではアクセス範囲を指定する表現としてスコープという概念をサポートしている。
クライアントは認可リクエスト送信時にスコープを指定し、リソースオーナが指定されたスコープを認可するか、認可サーバに事前に登録されたスコープと一致する場合に、認可サーバでクライアントに対してそのスコープを認可する。
クライアントに対してのスコープの認可と :ref:`SpringSecurityAuthorization`\ の節で説明しているSpring Securiyのロールによる認可は併用可能である。

Spring Security OAuthでは、\ ``AuthorizationEndpoint``\ においてリソースオーナからの認可の取得機能を提供し、
\ ``AuthorizationEndpoint``\ または\ ``TokenEndpoint``\ においてクライアントに対してのアクセストークンの発行機能を提供している。
\ ``AuthorizationEndpoint``\と\ ``TokenEndpoint``\はアクセストークンの発行を\ ``AuthorizationServerTokenService``\ の
デフォルト実装である\ ``DefaultTokenServices``\ によって行っている。
アクセストークンの発行の際には、\ ``ClientDetailsService``\を介して認可サーバに登録済みのクライアント情報を取得し、アクセストークン発行の可否の検証に使用している。

なお、OAuth 2.0の仕様では認可サーバを利用するクライアントの登録手順については定められておらず、Spring Security OAuthにおいても
クライアント登録手続きはサポートされていない。
そのため、アプリケーションでクライアント登録のインターフェイスを提供したい場合には、独自に実装する必要がある。

リソースオーナの認証にはSpring Securityの認証機能を利用する。
詳細については :ref:`SpringSecurityAuthentication`\ を参照のこと。

以下に認可サーバの構成を示す。

.. figure:: ./images/OAuth_AutohrizationServerAuthArchitecture.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **認可サーバの動き（認可エンドポイントアクセス時）**
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ユーザエージェントが認可サーバの認可エンドポイント(/oauth/authorize)にアクセスすることで\ ``AuthorizationEndpoint``\ の処理が実行される。
    * - | (2)
      - | \ ``ClientDetailsService``\ のメソッドを呼び出し、事前に登録されているクライアント情報を取得後、リクエストパラメータを検証する。
    * - | (3)
      - | \ ``UserApprovalHandler``\ のメソッドを呼び出し、クライアントへスコープに対する認可が登録されているかチェックする。
        | 認可が未登録である場合、ユーザエージェント経由でリソースオーナへ認可を問い合わせる画面(/oauth/confirm_access)を表示させる。
        | このとき、問い合わせの対象となるスコープはリクエストパラメータと事前に登録されているクライント情報の積をとり、Spring MVCの\ ``@SessionAttributes``\ を利用して連携される。
    * - | (4)
      - | \ ``UserApprovalHandler``\ の実装である\ ``ApprovalStoreUserApprovalHandler``\ では\ ``ApprovalStore``\により認可の状態を管理する。
        | リソースオーナにより認可が行われた場合、\ ``ApprovalStore``\ のメソッドを呼び出し、指定された情報を登録する。

|

.. note::

    問い合わせの対象となるスコープは前述のとおり、認可サーバに事前に登録されているスコープと、クライアントが認可リクエスト時にリクエストパラメータで指定したスコープの積となる。
    例として、認可サーバでREADとCREATEとDELETEのスコープが割り当てられているクライアントに対して、READとCREATEのスコープをリクエストパラメータで指定した場合は(READ,CREATE,DELETE)と(READ,CREATE)の積である、スコープREAD,CREATEが割り当てられる。
    認可サーバでクライアントに割り当てられていないスコープをリクエストパラメータで指定した場合はエラーとなり、アクセスが拒否される。

|

.. figure:: ./images/OAuth_AutohrizationServerTokenArchitecture.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **認可サーバの動き（トークンエンドポイントアクセス時）**
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | クライアントが認可サーバのトークンエンドポイント(/oauth/token)にアクセスすることで\ ``TokenEndpoint``\ の処理が実行される。
    * - | (2)
      - | \ ``ClientDetailsService``\ のメソッドを呼び出し、事前に登録されているクライアント情報を取得後、リクエストパラメータのスコープがクライアントに登録済みのものかチェックする。
    * - | (3)
      - | スコープが登録済みのものであった場合、\ ``TokenGranter``\ のメソッドを呼び出し、アクセストークンを発行する。
    * - | (4)
      - | \ ``TokenGranter``\ の実装である\ ``AbstractTokenGranter``\ では\ ``AuthorizationServerTokenServices``\ のメソッドを呼び出し、アクセストークンを発行する。
        | \ ``AbstractTokenGranter``\ はグラントタイプ別に実装されている\ ``TokenGranter``\ の基底クラスであり、実際の処理は各クラスに委譲される。
    * - | (5)
      - | \ ``AuthorizationServerTokenServices``\ の実装である\ ``DefaultTokenServices``\ では\ ``TokenStore``\ のメソッドを呼び出し、アクセストークンの状態を管理する。


|

.. _ResourceServer:

リソースサーバ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
リソースサーバは、クライアントからの保護されたリソースに対するアクセス要求を処理し、レスポンスを返す。
リソースサーバは、クライアントからのリクエストに付加されるアクセストークンについて、有効期限内であることを検証し、アクセストークンに紐づく認証情報を取得する。
認証情報の取得後は、要求されたリソースが当該アクセストークンのスコープ範囲内であることを検証する。
アクセストークン検証後の処理は、通常のアプリケーションと同様に実装を行うことができる。

Spring Security OAuthでは、アクセストークンの検証機能を、Spring Securityの認証・認可の枠組みを利用して実現している。
具体的には、\ ``ServletFilter``\ にSpring Security OAuthが提供する\ ``OAuth2AuthenticationProcessingFilter``\ を使用し、
\ ``AuthenticationEntryPoint``\インタフェース として\ ``OAuth2AuthenticationEntryPoint``\ を、\ ``AuthenticationManager``\ として\ ``OAuth2AuthenticationManager``\ をそれぞれ使用している。
Spring Securityの詳細については :ref:`SpringSecurityAuthentication`\ を参照のこと。

以下にリソースサーバの構成を示す

.. figure:: ./images/OAuth_ResourceServerArchitecture.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **リソースサーバの動き**
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 初めにクライアントがリソースサーバにアクセスすると\ ``OAuth2AuthenticationProcessingFilter``\ の
        | 呼び出しが行われる。
        | \ ``OAuth2AuthenticationProcessingFilter``\ではアクセストークンの抽出を行う。
    * - | (2)
      - | アクセストークンを抽出後、\ ``AuthenticationManager``\の実装である\ ``OAuth2AuthenticationManager``\ において\ ``ResourceServerTokenServices``\ を
        | 介してアクセストークンに紐づく認証情報を取得する。また、認証情報の取得時にアクセストークンを検証する。
        | アクセストークンに紐づく認証情報の取得方法には、認可サーバに対してHTTPによる問い合わせを行うほか、認可サーバと
        | \ ``TokenStore``\ を共用して取得を行うなどの方法がある。
        | どのようにして認証情報の取得を行うかについては\ ``ResourceServerTokenServices``\ の実装に依存する。
    * - | (3)
      - | アクセストークンの検証に成功した場合、クライアントからのリクエストに応じたリソースを返却する。
    * - | (4)
      - | 認証エラー時に発生する例外は、\ ``OAuth2AuthenticationEntryPoint``\ を使用してハンドリングし、エラー応答を行う。
    * - | (5)
      - | 認可エラー時に発生する例外は、\ ``OAuth2AccessDeniedHandler``\ を使用してハンドリングし、エラー応答を行う。


|

.. _Client:

クライアント
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
クライアントは、リソースオーナの認可とアクセストークンを取得し、リソースオーナの代理としてリソースサーバの保護されたリソースに対してアクセスを行う。
その際、リソースへのリクエストには認可サーバから発行されたアクセストークンを付加する。

Spring Security OAuthでは、クライアントの基本的な機能の実現方法として\ ``RestOperations``\ のOAuth 2.0向けの実装である\ ``OAuth2RestTemplate``\ を提供している。

\ ``OAuth2RestTemplate``\ では、アクセストークンの発行やリフレッシュトークンを使用したアクセストークンの再発行、また、アクセストークンを使用したリソースサーバへのアクセスといった機能のほか、
サーブレットフィルタとして\ ``OAuth2ClientContextFilter``\ を利用することで、認可コードグラントなどで必要となる認可の機能を実現している。

また、\ ``OAuth2RestTemplate``\ では\ ``OAuth2ProtectedResourceDetails``\ にて指定されたグラントタイプに沿って認可サーバより取得したアクセストークンを\ ``OAuth2ClientContext``\ の実装である\ ``DefaultOAuth2ClientContext``\ に保持する。
\ ``DefaultOAuth2ClientContext``\ はデフォルトではセッションスコープのBeanとして定義され、複数のリクエスト間でアクセストークンを共有をすることが可能となる。

リソースサーバへのアクセス機能の開発をする際は、\ ``RestOperations``\ の実装としてSpring Webが提供する\ ``RestTemplate``\ の代わりにSpring Security OAuthが提供する\ ``OAuth2RestTemplateを``\ 使用する以外は、通常のRESTクライアントの開発と同様の実装をする。

以下に、クライアント機能として\ ``OAuth2RestTemplate``\ を使用した場合の構成を示す。

.. figure:: ./images/OAuth_ClientArchitecture.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **クライアントの動き**
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | ユーザエージェントがクライアントの\ ``Service``\ の呼び出しが行われるよう\ ``Controller``\ へアクセスを行う。
        | リソースサーバへのアクセスを伴うアクセスに対しては、(5)で発生する可能性がある\ ``UserRedirectRequiredException``\ を捕捉するためのサーブレットフィルタ（\ ``OAuth2ClientContextFilter``\ ）を適用する。
        | このサーブレットフィルタを適用することで、\ ``UserRedirectRequiredException``\ が発生した際に、ユーザエージェントを認可サーバの認可エンドポイントへアクセスさせることができる。
    * - | (2)
      - | \ ``Service``\ より\ ``OAuth2RestTemplate``\ の呼び出しを行う。
    * - | (3)
      - | リソースサーバへアクセスする前に、メンバとして保持している\ ``DefaultOAuth2ClientContext``\ よりアクセストークンを保持しているか確認を行う。
        | アクセストークンを保持しており、かつ有効期限内である場合、\ ``DefaultOAuth2ClientContext``\ より取得したアクセストークンを指定してリソースサーバへのアクセスを行う。
    * - | (4)
      - | 初回アクセス時などでアクセストークンを保持していなかった場合、または有効期限が超過していた場合、\ ``AccessTokenProvider``\ を呼び出しアクセストークンの取得を行う。
    * - | (5)
      - | \ ``AccessTokenProvider``\ では、リソースの詳細情報として\ ``OAuth2ProtectedResourceDetails``\ に定義しているグラントタイプに応じてアクセストークンの取得を行う。
        | 認可コードグラント向けの実装である\ ``AuthorizationCodeAccessTokenProvider``\ では、認可コードの取得が完了していない場合、\ ``UserRedirectRequiredException``\ をthrowする。
    * - | (6)
      - | (3)または(5)で取得したアクセストークンを指定して、リソースサーバへのアクセスを行う。
        | アクセス時にアクセストークンの有効期限切れ（\ ``AccessTokenRequiredException``\ ）などのエラーが発生した場合、保持しているアクセストークンを初期化した後、再度(4)以降の処理を行う。

|


.. _HowToUse:

How to use
--------------------------------------------------------------------------------

Spring Security OAuthを使用するために必要となるBean定義例や実装方法について説明する。

|

.. _OAuthSetup:

Spring Security OAuthのセットアップ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Security OAuthが提供しているクラスを使用するために、Spring Security OAuthを依存ライブラリとして追加する。

.. code-block:: xml

    <!-- (1) -->
    <dependency>
        <groupId>org.springframework.security.oauth</groupId>
        <artifactId>spring-security-oauth2</artifactId>
    </dependency>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | Spring Security OAuthを使用するプロジェクトの :file:`pom.xml` に、Spring Security OAuthを依存ライブラリとして追加する。
        | リソースサーバ、認可サーバ、クライアントを別プロジェクトとして作成する場合、それぞれについて記述を追加すること。

.. note::

    上記設定例は、依存ライブラリのバージョンを親プロジェクトである terasoluna-gfw-parent で管理する前提であるため、pom.xmlでのバージョンの指定は不要である。
    上記の依存ライブラリはterasoluna-gfw-parentが利用している\ `Spring IO Platform <http://platform.spring.io/platform/>`_\ で定義済みである。

|

.. _OAuthHowToUseApplicationSettings:

アプリケーションの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Security OAuthを使用するアプリケーションの設定について説明する。

「\ :ref:`AuthorizationGrant`\」にて示したとおり、OAuth 2.0ではグラントタイプにより認可サーバ、クライアント間のフローが異なる。
そのため、Spring Security OAuthを使用するアプリケーションでは、アプリケーションがサポートするグラントタイプに沿った設定を行う必要がある。
グラントタイプ別の設定内容については各ロールの実装を参照。

|

.. _ImplementationOAuthAuthorizationServer:

認可サーバの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

認可サーバの実装方法について説明する。

認可サーバでは、「リソースオーナからの認可の取得」「アクセストークンの発行」を行うためのエンドポイントをSpring Security OAuthの機能を使用して提供する。
なお、上記のエンドポイントにアクセスする場合はリソースオーナまたはクライアントの認証が必要であり、本ガイドラインではSpring Securityの認証・認可の仕組みを使用して実現する。

.. _OAuthAuthorizationServerCreateSettingFile:

設定ファイルの作成（認可サーバ）
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 認可サーバに関する定義を行うための設定ファイルとして  \ ``oauth2-auth.xml``\ を作成する。
| \ ``oauth2-auth.xml``\ では、認可サーバの機能を提供するためのエンドポイントのBean定義およびそれらのエンドポイントに対するセキュリティ設定、認可サーバのサポートするグラントタイプの設定を行う。

* ``oauth2-auth.xml``

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:sec="http://www.springframework.org/schema/security"
           xmlns:oauth2="http://www.springframework.org/schema/security/oauth2"
           xsi:schemaLocation="http://www.springframework.org/schema/security
               http://www.springframework.org/schema/security/spring-security.xsd
               http://www.springframework.org/schema/security/oauth2
               http://www.springframework.org/schema/security/spring-security-oauth2.xsd
               http://www.springframework.org/schema/beans
               http://www.springframework.org/schema/beans/spring-beans.xsd">


    </beans>

次に、作成した\ ``oauth2-auth.xml``\ を読み込むように\ ``web.xml``\ に設定を記述する。

* ``web.xml``

.. code-block:: xml

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            classpath*:META-INF/spring/applicationContext.xml
            classpath*:META-INF/spring/oauth2-auth.xml  <!-- (1) -->
            classpath*:META-INF/spring/spring-security.xml
        </param-value>
    </context-param>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ OAuth 2.0用のBean定義ファイルの指定を行う。\ ``oauth2-auth.xml``\ で設定したアクセス制御の対象のURLが\ ``spring-security.xml``\ で設定したアクセス制御の対象のURLに含まれる場合を考慮し、\ ``spring-security.xml``\より先に記述すること。

|

.. _OAuthAuthorizationServerDefinition:

認可サーバの定義
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
次に、認可サーバの定義を追加する。

* ``oauth2-auth.xml``

.. code-block:: xml

        <oauth2:authorization-server
             token-endpoint-url="/oth2/token"
             authorization-endpoint-url="/oth2/authorize" >  <!-- (1) -->
            <oauth2:authorization-code />  <!-- (2) -->
            <oauth2:implicit />  <!-- (3) -->
            <oauth2:refresh-token />  <!-- (4) -->
            <oauth2:client-credentials />  <!-- (5) -->
            <oauth2:password />  <!-- (6) -->
        </oauth2:authorization-server>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``<oauth2:authorization-server>``\ タグを使用し、認可サーバの設定を定義を行う。
        | \ ``<oauth2:authorization-server>``\ タグを使用することで、認可を行うための認可エンドポイントと、アクセストークンを発行するためのトークンエンドポイントがコンポーネントとして登録される。
        | \ ``token-endpoint-url``\ 属性にトークンエンドポイントのURLを指定する。指定しない場合はデフォルト値である "/oauth/token" が指定される。
        | \ ``authorization-endpoint-url``\ 属性に認可エンドポイントのURLを指定する。指定しない場合はデフォルト値である "/oauth/authorize" が指定される。
    * - | (2)
      - | \ ``<oauth2:authorization-code>``\ タグを使用して、認可コードグラントをサポートする。
    * - | (3)
      - | \ ``<oauth2:implicit>``\ タグを使用して、インプリシットグラントをサポートする。
    * - | (4)
      - | \ ``<oauth2:refresh-token>``\ タグを使用して、リフレッシュトークンをサポートする。
    * - | (5)
      - | \ ``<oauth2:client-credentials>``\ タグを使用して、クライアントクレデンシャルグラントをサポートする。
    * - | (6)
      - | \ ``<oauth2:password>``\ タグを使用して、リソースオーナパスワードクレデンシャルグラントをサポートする。

.. note::

    サポートするグラントタイプを複数指定する場合は上記の順番で指定する必要がある。

.. note::

    認可コードは、認可コードが発行されてからアクセストークンの発行までの短い期間しか使われないため、デフォルトではインメモリで管理される。
    認可サーバが複数台構成の場合は、複数サーバ間で認可コードを共有するためにDBで管理する必要がある。
    認可コードをDBで管理する場合は、主キーとなる認可コードを保持するカラムと、認証情報を保持するカラムによって構成された以下のようなテーブルを作成する。以下の例ではPostgreSQLを使用した場合のDB定義を説明する。
    
    .. figure:: ./images/OAuth_ERDiagramCode.png
        :width: 30%
    
    認可サーバの設定ファイルには、\ ``<oauth2:authorization-code>``\ タグの\ ``authorization-code-services-ref``\ に、認可コードをDB管理する\ ``org.springframework.security.oauth2.provider.code.JdbcAuthorizationCodeServices``\ のBean IDを指定する。
    \ ``JdbcAuthorizationCodeServices``\ のコンストラクタには、認可コード格納用のテーブルに接続するためのデータソースを指定する。
    認可コードをDBにて永続管理する場合の注意点については\ :ref:`OAuthAuthorizationServerHowToControllTarnsaction`\ を **必ず** 参照のこと。
    
    * ``oauth2-auth.xml``
    
    .. code-block:: xml
    
            <oauth2:authorization-server
                 token-endpoint-url="/oth2/token"
                 authorization-endpoint-url="/oth2/authorize" >
                <oauth2:authorization-code authorization-code-services-ref="authorizationCodeServices"/>
                <!-- omitted -->
            </oauth2:authorization-server>
            
            <bean id="authorizationCodeServices"
                  class="org.springframework.security.oauth2.provider.code.JdbcAuthorizationCodeServices">
                <constructor-arg ref="codeDataSource"/>
            </bean>
            
            <!-- omitted -->

|

.. _OAuthAuthorizationServerClientAuthentication:

クライアントの認証
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
エンドポイントに対してアクセスしてきたクライアントについては、登録済みのクライアントか確認するために認証を行う必要がある。
クライアントの認証は、クライアントよりパラメータで渡されたクライアントIDとパスワードを、認可サーバで保持しているクライアント情報をもとに検証することで行う。認証にはBasic認証を用いて行う。

Spring Security OAuthではクライアント情報を取得するためのインタフェースである\ ``oorg.springframework.security.oauth2.provider.ClientDetailsService``\ の実装クラスを提供している。 
また、クライアントの情報を保持するためのクラスとして\ ``ClientDetails``\ インタフェースの実装クラスである\ ``org.springframework.security.oauth2.provider.client.BaseClientDetails``\ クラスを提供している。
\ ``BaseClientDetails``\ ではクライアントIDやサポートするグラントタイプなどのOAuth 2.0を利用する上での基本的なパラメータを提供しており、\ ``BaseClientDetails``\ を拡張することで独自のパラメータを追加することも可能である。
ここでは\ ``BaseClientDetails``\の拡張と\ ``ClientDetailsService``\ の実装クラス作成を行い、独自パラメータとして クライアント名 を追加したクライアント情報をDBを用いて管理、および認証を行う場合の実装方法について説明する。

まず、以下のようなDBを作成する。

.. figure:: ./images/OAuth_ERDiagram.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | クライアント情報を保持するテーブル。client_idを主キーとする。
        | 各カラムの役割は以下のとおりである。
        |  ・client_id：クライアントを識別するIDであるクライアントIDを保持するカラム。
        |  ・client_secret：クライアントの認証に使用するパスワードを保持するカラム。
        |  ・client_name：クライアント名を保持する独自カラム。独自カラムであるため必須ではない。
        |  ・access_token_validity：アクセストークンの有効期間[秒]を保持するカラム。
        |  ・refresh_token_validity：リフレッシュトークンの有効期間[秒]を保持するカラム。
    * - | (2)
      - | スコープ情報を保持するテーブル。client_idを外部キーとし、クライアント情報と対応付けする。
        | scopeカラムに、クライアント認可に使用するスコープを保持する。クライアントがもつスコープの数だけレコードを登録する。
    * - | (3)
      - | リソース情報を保持するテーブル。client_idを外部キーとし、クライアント情報と対応付けする。
        | resource_idカラムに、クライアントのアクセス可能なリソースかどうかを、リソースサーバが識別するために使用するリソースIDを保持する。
        | リソースサーバが保持するリソースに対して定義しているリソースIDがここで登録されているリソースIDに含まれる場合のみ、リソースへのアクセスを許可される。
        | クライアントがアクセス可能なリソースIDの数だけレコードを登録する。
        | リソースIDを一件も登録しなかった場合は、全てのリソースに対してアクセス可能となるため、登録しない場合は注意が必要である。
    * - | (4)
      - | グラント情報を保持するテーブル。client_idを外部キーとし、クライアント情報と対応付けする。
        | authorized_grant_typeカラムに、クライアントの使用するグラントを保持する。
        | クライアントが利用するグラントの数だけレコードを登録する。
    * - | (5)
      - | リダイレクトURL情報を保持するテーブル。client_idを外部キーとし、クライアント情報と対応付けする。
        | web_server_redirect_uriカラムに、リソースオーナによる認可後にユーザエージェントをリダイレクトさせるURLを保持する。
        | リダイレクトURLは認可コードグラント、インプリシットグラントの場合のみ使用される。
        | 認可コードグラント、インプリシットグラント以外のグラントタイプを使用する場合はテーブル自体が不要となる。
        | クライアントが認可リクエスト時に申告するURLと、ホストとルートパスが一致するリダイレクトURLがない場合はエラーとなる。
        | クライアントが申告する可能性のあるURLの数だけレコードを登録する。


クライアント情報を保持するモデルを作成する。

* ``Client.java``

.. code-block:: java

        public class Client implements Serializable{
            private String clientId; // (1)
            private String clientSecret; // (2)
            private String clientName; // (3)
            private Integer accessTokenValidity; // (4)
            private Integer refreshTokenValidity; // (5)
            // Getters and Setters are omitted
        }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | クライアントを識別するクライアントIDを保持するフィールド。
    * - | (2)
      - | クライアントの認証に使用するパスワードを保持するフィールド。
    * - | (3)
      - | Spring Security OAuthでは提供されていない、クライアント名を保持する拡張フィールド。
        | 拡張フィールドであるため必須ではない。
    * - | (4)
      - | アクセストークンの有効期間[秒]を保持するフィールド。
    * - | (5)
      - | リフレッシュトークンの有効期間[秒]を保持するフィールド。


\ ``BaseClientDetails``\ クラスを継承させたクラスを作成することで、クライアントの詳細情報を簡単に拡張することができる。

* ``OAuthClientDetails.java``

.. code-block:: java

        public class OAuthClientDetails extends BaseClientDetails{
            private Client client;
            // Getter and Setter are omitted
        }


\ ``org.springframework.security.oauth2.provider.ClientDetailsService``\ は、認可処理で必要となるクライアント詳細情報をデータストアから取得するためのインタフェースである。
以下では、\ ``ClientDetailsService``\ の実装クラスの作成について説明する。

* ``OAuthClientDetailsService.java``

.. code-block:: java

        @Service("clientDetailsService") // (1)
        @Transactional
        public class OAuthClientDetailsService implements ClientDetailsService {
        
            @Inject
            ClientRepository clientRepository;
        
            @Override
            public ClientDetails loadClientByClientId(String clientId)
                    throws ClientRegistrationException {
                
                Client client = clientRepository.findClientByClientId(clientId); // (2)
                
                if (client == null) { // (3)
                      throw new NoSuchClientException("No client with requested id: " + clientId);
                }
                
                // (4)
                Set<String> clientScopes = clientRepository.findClientScopesByClientId(clientId);
                Set<String> clientResources = clientRepository.findClientResourcesByClientId(clientId);
                Set<String> clientGrants = clientRepository.findClientGrantsByClientId(clientId);
                Set<String> clientRedirectUris = clientRepository.findClientRedirectUrisByClientId(clientId);
                
                
                 // (5)
                OAuthClientDetails clientDetails = new OAuthClientDetails();
                clientDetails.setClientId(client.getClientId());
                clientDetails.setClientSecret(client.getClientSecret());
                clientDetails.setAccessTokenValiditySeconds(client.getAccessTokenValidity());
                clientDetails.setRefreshTokenValiditySeconds(client.getRefreshTokenValidity());
                clientDetails.setResourceIds(clientResources);
                clientDetails.setScope(clientScopes);
                clientDetails.setAuthorizedGrantTypes(clientGrants);
                clientDetails.setRegisteredRedirectUri(clientRedirectUris);
                clientDetails.setClient(client);
                
                return clientDetails;
            }
            
        }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | Serviceとしてcomponent-scanの対象とするため、クラスレベルに\ ``@Service``\ アノテーションをつける。
        | Bean名を\ ``clientDetailsService``\ として指定する。
    * - | (2)
      - | データベースから取得したクライアント情報をClientモデルに保持させる。
    * - | (3)
      - | クライアント情報が見つからない場合は、Spring Security OAuthの例外である\ ``NoSuchClientException``\ を発生させる。
    * - | (4)
      - | クライアントに紐付く情報を取得する。
        | 複数回にわけてRepositoryの呼び出しを行うことにより処理性能が落ちるような場合は(2)で一括取得する。
    * - | (5)
      - | 取得した各種情報を\ ``OAuthClientDetails``\ のフィールドに設定する。


\ ``oauth2-auth.xml``\ にクライアント認証に必要な設定を追記する。

* ``oauth2-auth.xml``

.. code-block:: xml

        <sec:http pattern="/oth2/*token*/**" 
            authentication-manager-ref="clientAuthenticationManager" realm="Realm">  <!-- (1) -->
            <sec:http-basic />           <!-- (2) -->
            <sec:csrf disabled="true"/>  <!-- (3) -->
            <sec:intercept-url pattern="/**" access="isAuthenticated()"/>  <!-- (4) -->
        </sec:http>

        <oauth2:authorization-server 
             token-endpoint-url="/oth2/token"
             authorization-endpoint-url="/oth2/authorize"
             client-details-service-ref="clientDetailsService">  <!-- (5) -->
            <oauth2:authorization-code />
            <oauth2:implicit />
            <oauth2:refresh-token />
            <oauth2:client-credentials />
            <oauth2:password />
        </oauth2:authorization-server>

        <sec:authentication-manager id="clientAuthenticationManager">  <!-- (6) -->
            <sec:authentication-provider user-service-ref="clientDetailsUserService" >  <!-- (7) -->
                <sec:password-encoder ref="passwordEncoder"/>  <!-- (8) -->
            </sec:authentication-provider>
        </sec:authentication-manager>

        <bean id="clientDetailsUserService"
            class="org.springframework.security.oauth2.provider.client.ClientDetailsUserDetailsService">  <!-- (9) -->
            <constructor-arg ref="clientDetailsService" />  <!-- (10) -->
        </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | アクセストークン操作に関するエンドポイントへのセキュリティ設定を行うために、エンドポイントURLとして
          \ ``/oth2/*token*/``\ 配下をアクセス制御の対象として指定する。
        | ここでは、アクセス制御対象となるエンドポイントURLを \ ``/oth2/``\ から始まる値としているが、
          Spring Security OAuthにより定義されているエンドポイントURL、およびそのデフォルト値は以下のとおりである。
        |  ・トークン払い出しに使用するエンドポイントのエンドポイントURLである\ ``/oauth/token``\ 
        |  ・トークンを検証するエンドポイントのエンドポイントURLである\ ``/oauth/check_token``\ 
        |  ・JWTの署名を公開鍵暗号方式で作成した場合に、公開鍵を取得するために使用するエンドポイントのエンドポイントURLである\ ``/oauth/token_key``\ 
        | \ ``authentication-manager-ref``\ 属性に(5)で定義しているクライアント認証用の\ ``AuthenticationManager``\のBeanを指定する。
        | また、(2)のようにXML NamespaceでBasic認証を有効にした場合、Basic認証のRealm名が \ ``"Spring Security Application"``\ となり、
        | アプリケーションの内部情報が露呈してしまうため、\ ``realm``\ 属性に適切な値を指定する。
    * - | (2)
      - | クライアント認証にBasic認証を適用する。
        | 詳細については http://docs.spring.io/spring-security/site/docs/4.1.4.RELEASE/reference/html/basic.html を参照されたい。
    * - | (3)
      - | \ ``/oth2/*token*/**``\ へのアクセスに対してCSRF対策機能を無効化する。
        | Spring Security OAuthでは、OAuth 2.0のCSRF対策として推奨されている、stateパラメータを使用したリクエストの正当性確認を採用している。
    * - | (4)
      - | エンドポイントURLの配下に対して、認証済みユーザーのみがアクセスできる権限を付与する設定。
        | Webリソースに対してアクセスポリシーの指定方法については、\ :doc:`../../Security/Authorization`\ を参照されたい。
    * - | (5)
      - | \ ``client-details-service-ref``\ 属性に\ ``OAuthClientDetailsService``\ のBeanを指定する。
        | 指定するBeanIDは、\ ``ClientDetailsService``\ の実装クラスで指定したBeanIDと合わせる必要がある。
    * - | (6)
      - | クライアントを認証するための\ ``AuthenticationManager``\ をBean定義する。
        | リソースオーナの認証で使用する\ ``AuthenticationManager``\ と別名のBean IDを指定する必要がある。
        | リソースオーナの認証については\ :ref:`OAuthAuthorizationServerResourceOwnerAuthentication`\を参照されたい。
    * - | (7)
      - | \ ``sec:authentication-provider``\ の\ ``user-service-ref``\ 属性に(9)で定義している\ ``ClientDetailsUserDetailsService``\のBeanを指定する。
    * - | (8)
      - | クライアントの認証に使用するパスワードのハッシュ化に使用する\ ``PasswordEncoder``\ のBeanを指定する。
        | パスワードハッシュ化の詳細については \ :ref:`SpringSecurityAuthenticationPasswordHashing`\ を参照されたい。
    * - | (9)
      - | \ ``UserDetailsService``\ インタフェースの実装クラスである\ ``ClientDetailsUserDetailsService``\ をBean定義する。
        | リソースオーナの認証で使用する\ ``UserDetailsService``\ と別名のBean IDを指定する必要がある。
    * - | (10)
      - | コンストラクタの引数に、データベースからクライアント情報を取得する\ ``OAuthClientDetailsService``\のBeanを指定する。
        | 指定するBean IDは、\ ``ClientDetailsService``\ の実装クラスで指定したBean IDと合わせる必要がある。


|

.. _OAuthAuthorizationServerResourceOwnerAuthentication:

リソースオーナの認証
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

アクセストークンの取得に認可コードグラントを用いる場合、ログイン画面を用意する等、なんらかの方法でリソースオーナを認証する必要がある。

| 本ガイドラインでは、リソースオーナの認証にSpring Securityを利用する前提とする。
| 認可の設定には、認証済みユーザーのみ認可エンドポイントURLへアクセスできるよう、認可エンドポイントURLを含んだURLをアクセスポリシーとして定義する必要がある。
  また、認可画面の表示を行うコントローラのURLと、認可エンドポイントでの例外をハンドリングするコントローラのURLも同様にアクセスポリシーとして定義する必要がある。
| 認可画面の表示を行うコントローラについては \ :ref:`OAuthAuthorizationServerHowToCustomizeAuthorizeView`\ を、認可エンドポイントでの例外をハンドリングするコントローラについては \ :ref:`OAuthAuthorizationServerHowToHandleError`\ を参照されたい。

Spring Securityの詳細については \ :doc:`../../Security/Authentication`\ 及び \ :doc:`../../Security/Authorization`\ を参照されたい。

以下に認可エンドポイントURL、認可画面の表示を行うコントローラのURL、認可エンドポイントのエラーハンドリングを行うコントローラのURLを含んだアクセスポリシーの定義例を示す。

* ``spring-security.xml``

.. code-block:: xml

        <sec:http authentication-manager-ref="userLoginManager"> <!-- (1) -->
            <sec:form-login login-page="/login"
                authentication-failure-url="/login?error=true"
                login-processing-url="/login" />
            <sec:logout logout-url="/logout" 
                logout-success-url="/" 
                delete-cookies="JSESSIONID" />
            <sec:access-denied-handler ref="accessDeniedHandler"/>
            <sec:custom-filter ref="userIdMDCPutFilter" after="ANONYMOUS_FILTER"/>
            <sec:session-management />
            <sec:intercept-url pattern="/login/**" access="permitAll" />
            <sec:intercept-url pattern="/oth2/**" access="isAuthenticated()" /> <!-- (2) -->
            <!-- omitted -->
        </sec:http>
        
         <sec:authentication-manager id="userLoginManager"> <!-- (3) -->
            <sec:authentication-provider
                user-service-ref="userDetailsService">
                <sec:password-encoder ref="passwordEncoder" />
            </sec:authentication-provider>
        </sec:authentication-manager>
        
        <bean id="userDetailsService"
            class="org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl">
            <property name="dataSource" ref="dataSource" />
        </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 認可エンドポイントURLである\ ``/oth2/authorize``\ 、認可画面の表示を行うコントローラのURLである\ ``/oauth/confirm_access``\ 、認可エンドポイントのエラーハンドリングを行うコントローラのURLである\ ``/oauth/error``\ を含んだルート("\ ``/``\ ")配下をアクセス制御の対象として指定する。
    * - | (2)
      - | 認可エンドポイントURLである\ ``/oth2/authorize``\ 、認可画面の表示を行うコントローラのURLである\ ``/oauth/confirm_access``\ 、認可エンドポイントのエラーハンドリングを行うコントローラのURLである\ ``/oauth/error``\ を含んだルート("\ ``/oth2/``\ ")配下を認証済みユーザーのみがアクセスできるよう指定する。
    * - | (3)
      - | リソースオーナを認証するための\ ``AuthenticationManager``\ をBean定義する。
        | クライアントの認証で使用する\ ``AuthenticationManager``\ と別名のBean IDを指定する必要がある。


.. _OAuthAuthorizationServerHowToAuthorizeByScope: 

スコープごとの認可
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
リソースオーナに認可を求める際に、要求されたスコープを一括で認可するのではなく、各スコープを個別に認可する場合の設定方法を説明する。

認可サーバを再起動した際に認可情報を失わないよう永続管理するために、また複数台の認可サーバで認可情報を共有するためには、認可情報をDBで管理する必要がある。
スコープごとに認可情報を格納するためのDBとして、以下のDBを作成する。以下の例ではPostgreSQLを使用した場合のDB定義を説明する。

.. figure:: ./images/OAuth_ERDiagramApprovals.png
    :width: 50%


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 認可情報を保持するテーブル。userId、clientId、scopeを主キーとする。
        | 各カラムの役割は以下のとおりである。
        |  ・userId：認可を行ったリソースオーナのユーザ名を保持するカラム。
        |  ・clientId：リソースオーナによって認可されたクライアントのクライアントIDを保持するカラム。
        |  ・scope：リソースオーナに認可されたスコープを保持するカラム。
        |  ・status：リソースオーナに認可されたかどうかを保持するカラム。認可された場合は\ ``APPROVED``\ 、拒否された場合は\ ``DENIED``\ が設定される。
        |  ・expiresAt：認可情報の有効期限を保持するカラム。
        |  ・lastModifiedAt：認可情報が最後に更新された日時を保持するカラム。

リソースオーナからスコープ毎の認可を取得し、DBに保存して管理するための設定を行う。

実装例は以下のとおりである。

* ``oauth2-auth.xml``

.. code-block:: xml

        <oauth2:authorization-server
             client-details-service-ref="clientDetailsService"
             token-endpoint-url="/oth2/token"
             authorization-endpoint-url="/oth2/authorize"
             user-approval-handler-ref="userApprovalHandler"> <!-- (1) -->
             
             <!-- omitted -->
             
        </oauth2:authorization-server>

        <bean id="userApprovalHandler"
              class="org.springframework.security.oauth2.provider.approval.ApprovalStoreUserApprovalHandler">  <!-- (2) -->
            <property name="clientDetailsService" ref="clientDetailsService"/>
            <property name="approvalStore" ref="approvalStore"/>
            <property name="requestFactory" ref="requestFactory"/>
            <property name="approvalExpiryInSeconds" value="3200" />
        </bean>

        <bean id="approvalStore"
              class="org.springframework.security.oauth2.provider.approval.JdbcApprovalStore">  <!-- (3) -->
            <constructor-arg ref="approvalDataSource"/>
        </bean>

        <bean id="requestFactory"
              class="org.springframework.security.oauth2.provider.request.DefaultOAuth2RequestFactory">
            <constructor-arg ref="clientDetailsService"/>
        </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | スコープの認可処理を行う\ ``UserApprovalHandler``\ として、\ ``user-approval-handler-ref``\ に(2)で定義している\ ``ApprovalStoreUserApprovalHandler``\ のBeanを指定する。
    * - | (2)
      - | スコープの認可処理を行う\ ``ApprovalStoreUserApprovalHandler``\ をBean定義する。
        | リソースオーナの認可結果を管理する\ ``approvalStore``\ プロパティには、(3)で定義している\ ``JdbcApprovalStore``\ のBeanを指定する。
        | スコープの認可処理に使用するクライアント情報の取得をする\ ``clientDetailsService``\ プロパティには、\ ``OAuthClientDetailsService``\ のBeanを指定する。
        | \ ``requestFactory``\ プロパティには、\ ``DefaultOAuth2RequestFactory``\ のBeanを指定する。
        | \ ``requestFactory``\ プロパティに設定したBeanは\ ``ApprovalStoreUserApprovalHandler``\ によって使用されないが、設定を行っていない場合は\ ``ApprovalStoreUserApprovalHandler``\ のBean生成時にエラーとなるため、\ ``requestFactory``\ プロパティへの設定が必要である。
        | 認可情報の有効期間[秒]を指定する場合は、\ ``approvalExpiryInSeconds``\ プロパティに、有効期間[秒]を設定する。設定を行わない場合は、認可情報は認可から一ヶ月間有効となる。
    * - | (3)
      - | 認可情報をDBで管理する\ ``JdbcApprovalStore``\ をBean定義する。
        | コンストラクタには、認可情報格納用のテーブルに接続するためのデータソースを指定する。
        | 認可情報をDBにて永続管理する場合の注意点については\ :ref:`OAuthAuthorizationServerHowToControllTarnsaction`\ を **必ず** 参照のこと。

.. note::

    認可情報を永続管理する必要がなく、DBではなくインメモリで管理したい場合は、\ ``approvalStore``\ として\ ``org.springframework.security.oauth2.provider.approval.InMemoryApprovalStore``\ をBean定義すればよい。


.. _OAuthAuthorizationServerHowToCustomizeAuthorizeView: 

スコープ認可画面のカスタマイズ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

スコープ認可画面をカスタマイズしたい場合、コントローラとJSPを作成することでカスタマイズできる。以下ではスコープ認可画面のカスタマイズした場合の例を説明する。

リソースオーナに認可を求めるエンドポイントの呼び出しを行う場合、(コンテキストパス)/oauth/confirm_accessにフォワードされる。
(コンテキストパス)/oauth/confirm_accessをハンドリングするコントローラを作成する。

* ``OAuth2ApprovalController.java``

.. code-block:: java

        @Controller
        public class OAuth2ApprovalController {
                
            @RequestMapping("/oauth/confirm_access") // (1)
            public String confirmAccess() {
                // omitted
                return "approval/oauthConfirm";
            }
        
        }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@RequestMapping``\ アノテーションを使用して、\ ``"/oauth/confirm_access"``\ へのアクセスに対するメソッドとしてマッピングを行う。


次に、スコープ認可画面のJSPを作成する。
認可対象のスコープは\ ``scopes``\ キーでModelに登録されているため、これを利用してスコープ認可画面を表示する。

* ``oauthConfirm.jsp``

.. code-block:: jsp

    <%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags" %>
    
    <body>
        <div id="wrapper">
            <h1>OAuth Approval</h1>
            <p>Do you authorize '${f:h(authorizationRequest.clientId)}' to access your protected resources?</p>
            <form id='confirmationForm' name='confirmationForm' action='${pageContext.request.contextPath}/oth2/authorize' method='post'>
                <c:forEach var="scope" items="${scopes}" varStatus="status">  <!-- (1) -->
                    <li>
                        ${f:h(scope.key)}  <!-- (2) -->
                        <input type='radio' name="${f:h(scope.key)}" value='true'/>Approve
                        <input type='radio' name="${f:h(scope.key)}" value='false'/>Deny
                    </li>
                </c:forEach>
                <input name='user_oauth_approval' value='true' type='hidden'/>  <!-- (3) -->
                <sec:csrfInput />  <!-- (4) -->
                <label>
                    <input name='authorize' value='Authorize' type='submit'/>
                </label>
            </form>
        </div>
    </body>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1) 
      - | スコープ毎に認可を指定するためのラジオボタンを出力する。対象のスコープは\ ``scopes``\ リストに含まれるため、\ ``items``\ に\ ``scopes``\ を指定する。
    * - | (2)
      - | \ ``scopes``\ リストが保持する要素のキー名が、それぞれのスコープ名となっているため、キー名を画面表示する。
        | 許可、拒否を選ぶために、Approve、Denyのラジオボタンの出力設定を行う。
    * - | (3)
      - | \ ``user_oauth_approval``\ をhidden項目として埋め込むことで、Spring Security OAuthがリクエストパラメータに\ ``user_oauth_approval``\ を付与する。
        | リクエストパラメータに付与された\ ``user_oauth_approval``\ は、認可エンドポイントのスコープ認可を行うメソッドを実行するために用いられる。
    * - | (4)
      - | CSRFを引き渡すために、HTMLの\ ``<form>``\ 要素の中に\ ``<sec:csrfInput>``\ 要素を指定する。

.. _OAuthAuthorizationServerHowToHandleError:

認可リクエスト時のエラーハンドリング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

認可エンドポイントで認可エラー（クライアント未存在エラー等のセキュリティに関わるエラーや、リダイレクトURLチェックエラー）が発生した場合、Spring Security OAuthが提供する\ ``OAuth2Exception``\ が\ ``throw``\され 、リクエストは(コンテキストパス)/oauth/errorにフォワードされる。
そのため認可エンドポイントでの例外をハンドリングする場合は(コンテキストパス)/oauth/errorをハンドリングするコントローラを作成する必要がある。

* ``OAuth2ErrorController.java``

.. code-block:: java

        @Controller
        public class OAuth2ErrorController {
                
            @RequestMapping("/oauth/error") // (1)
            public String handleError() {
                // omitted
                return "common/error/oauthError";
            }
        
        }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@RequestMapping``\ アノテーションを使用して、\ ``"/oauth/error"``\ へのアクセスに対するメソッドとしてマッピングを行う。


次に、表示させるエラー画面のJSPを作成する。
認可エンドポイントで発生したエラー内容は\ ``error``\ キーでModelに登録されているため、これを利用してエラー内容を画面表示する。

* ``oauthError.jsp``

.. code-block:: jsp

    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="utf-8">
    <title>OAuth Error!</title>
    <link rel="stylesheet"
        href="${pageContext.request.contextPath}/resources/app/css/styles.css">
    </head>
    <body>
        <div id="wrapper">
            <h1>OAuth Error!</h1>
            <p>${f:h(error.OAuth2ErrorCode)}</p> <!-- (1) -->
            <p>${f:h(error.message)}</p> <!-- (2) -->
        <br>
        </div>
    </body>
    </html>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``error``\ に含まれるエラーレスポンスを出力する。
    * - | (2) 
      - | \ ``error``\ に含まれるエラーメッセージを出力する。

.. note::

    認可エンドポイントで発生したエラーが、認可エラー（クライアント未存在エラー等のセキュリティに関わるエラーや、リダイレクトURLチェックエラー）以外の場合、
    リダイレクトすることでクライアント側にエラー通知を行う。



.. _OAuthAuthorizationServerHowToConfigureAccessToken:

リソースサーバとのアクセストークン共有方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
リソースサーバがアクセストークンを元にリソースへのアクセスに対する認可判定を行えるよう、認可サーバは\ ``TokenServices``\ を介してアクセストークンを連携する。
連携方法は以下に示すとおり複数存在する。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 20 70
    :class: longtable

    * - 項番
      - 連携方法
      - 説明
    * - | (1)
      - | DBを介した連携
      - | 共有DBを利用し、アクセストークンを連携する方法。
        | リソースサーバと認可サーバがDBを共有している場合に利用可能。
        | 認可サーバはTokenServiceの実装として\ ``DefaultTokenServices``\ を、TokenStoreの実装として\ ``JdbcTokenStore``\ を指定する。
    * - | (2)
      - | HTTPアクセスを介した連携
      - | HTTPアクセスにより、アクセストークンを連携する方法。
        | リソースサーバと認可サーバが共有DBを利用できない場合に、この方法を利用する。
        | リソースサーバはアクセストークンの取得及び検証を認可サーバに依頼するため、認可サーバに負荷がかかる。
        | 認可サーバはTokenServiceの実装として\ ``DefaultTokenServices``\ を指定する。
        | アクセストークンをDBで管理する場合は\ ``JdbcTokenStore``\ を、メモリで管理する場合は\ ``InMemoryTokenStore``\ をTokenStoreの実装として指定する。
        | アクセストークンをメモリで管理する実装はサーバ再起動などでアクセストークンが失われるため、テスト用途専用の実装である。
    * - | (3)
      - | JWTを利用した連携
      - | JWTを利用し、アクセストークンを連携する方法。
        | リソースサーバと認可サーバが共有DBを利用できない場合に、この方法を利用する。
        | HTTPアクセスを介した連携と比べ、認可サーバにアクセストークンの取得を依頼しないため、認可サーバへの負荷がかからない。
        | 認可サーバはTokenServiceの実装として\ ``DefaultTokenServices``\ を、TokenStoreの実装として\ ``JwtTokenStore``\ を指定する。
        | \ ``org.springframework.security.oauth2.provider.token.store.JwtAccessTokenConverter``\ を利用することでアクセストークンの署名とエンコード、デコードを行う。
        | アクセストークンの署名とその検証には公開鍵を使用する方法と、共通鍵を使用する方法がある。
    * - | (4)
      - | メモリを介した連携
      - | メモリを共有することで、アクセストークンを連携する方法。
        | リソースサーバと認可サーバが一つのプロセスとなるようアプリケーションを設計している場合に利用可能。
        | 認可サーバはTokenServiceの実装として\ ``DefaultTokenServices``\ を、TokenStoreの実装として\ ``InMemoryTokenStore``\ を指定する。
        | メモリを介して連携させるため、共有DBやHTTPアクセスによるアクセストークンの連携が不要となる。
        | メモリを介してアクセストークンを共有する実装はサーバ再起動などでアクセストークンが失われるため、テスト用途専用の実装である。

| 

.. todo:: **TBD**
    
    JWTを利用した連携の実装方法については、次版以降で詳細化する予定である。



ここでは、代表的な連携方法として共有DBを介して連携させる方法を説明する。
HTTPアクセスを介した連携ついては本節のHow To Extendにて説明している。

 * \ :ref:`OAuthAuthorizationServerHowToCooperateWithHttp`\
 
共有DBを介して連携させる場合、Spring Security OAuthが提供している\ ``JdbcTokenStore``\ を使用する。

実装例は以下のとおりである。


* ``oauth2-auth.xml``

.. code-block:: xml

        <oauth2:authorization-server
             client-details-service-ref="clientDetailsService"
             token-endpoint-url="/oth2/token"
             authorization-endpoint-url="/oth2/authorize"
             user-approval-handler-ref="userApprovalHandler"
             token-services-ref="tokenServices">  <!-- (1) -->
            <oauth2:authorization-code />
            <oauth2:implicit />
            <oauth2:refresh-token />
            <oauth2:client-credentials />
            <oauth2:password />
        </oauth2:authorization-server>

        <bean id="tokenServices"
            class="org.springframework.security.oauth2.provider.token.DefaultTokenServices">  <!-- (2) -->
            <property name="tokenStore" ref="tokenStore" />
            <property name="clientDetailsService" ref="clientDetailsService" />
            <property name="supportRefreshToken" value="true" />  <!-- (3) -->
        </bean>

        <bean id="tokenStore"
          class="org.springframework.security.oauth2.provider.token.store.JdbcTokenStore"> <!-- (4) -->
          <constructor-arg ref="tokenDataSource" />
        </bean>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 認可サーバが使用するTokenServiceとして\ ``token-services-ref``\ 属性に(2)で定義している\ ``tokenServices``\ を指定する。
    * - | (2)
      - | \ ``tokenServices``\ のクラスに\ ``DefaultTokenServices``\を指定する。
        | アクセストークンを管理するトークンストアとして、\ ``tokenStore``\ プロパティに(4)で定義している\ ``TokenStore``\ を指定する。
        | 共有DBを介してリソースサーバとアクセストークンの連携を行う場合、リソースサーバでも本設定を行うこと。
    * - | (3)
      - | リフレッシュトークンを有効にする場合は\ ``supportRefreshToken``\ 属性に\ ``true``\ を指定する。
    * - | (4)
      - | トークンストアとして \ ``JdbcTokenStore``\ をBean定義する。
        | コンストラクタには、トークン情報格納用のテーブルに接続するためのデータソースを指定する。


\ ``JdbcTokenStore``\ がアクセストークンを連携するために、Spring Security OAuthがスキーマ定義している以下のDBを作成する。
以下の例では共有DBとしてPostgreSQLを使用した場合のDB定義を説明する。

.. figure:: ./images/OAuth_ERDiagramToken.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | アクセストークンを管理するテーブル。認可サーバで発行したアクセストークンの情報をリソースサーバと共有するために使用する。
        | 各カラムの役割は以下のとおりである。
        |  ・authentication_id：認証情報を一意に識別する認証キーを保持するカラム。主キーとする。
        |  ・token：トークンの情報をシリアル化してバイナリとして保持するカラム。保持するトークンの情報としては、アクセストークンの有効期限、スコープ、アクセストークンのトークンID、リフレッシュトークンのトークンID、使用しているトークンの種類を表すトークンタイプを保持する。
        |  ・token_id：アクセストークンを一意に識別するトークンIDを保持するカラム。
        |  ・user_name：認証されたリソースオーナのユーザ名を保持するカラム。
        |  ・client_id：認証されたクライアントのクライアントIDを保持するカラム。
        |  ・authentication：リソースオーナとクライアントの認証情報をシリアル化してバイナリとして保持するカラム。
        |  ・refresh_token：アクセストークンに紐付くリフレッシュトークンのトークンIDを保持するカラム。
    * - | (2)
      - | アクセストークンに紐付くリフレッシュトークンを管理するテーブル。
        | 各カラムの役割は以下のとおりである。
        |  ・token_id：リフレッシュトークンを一意に識別するトークンIDを保持するカラム。主キーとする。
        |  ・token：トークンの情報をシリアル化してバイナリとして保持するカラム。リフレッシュトークンの有効期限を保持する。
        |  ・authentication：リソースオーナとクライアントの認証情報をシリアル化してバイナリとして保持するカラム。アクセストークンを管理するテーブルで保持している認証情報と同じ情報を保持する。

| 

.. note::

    共有DBでトークンを管理する場合、有効期限切れとなったトークンはクライアントがアクセストークンを利用するタイミングに削除される。
    そのためトークンが有効期限切れになったとしても、クライアントがアクセストークンを利用しなければ削除されない。
    有効期限切れとなったトークンをDBから削除するためには、バッチ処理等で別途パージを行う必要がある。


.. _OAuthAuthorizationServerHowToCancelToken:

トークンの取り消し（認可サーバ）
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
発行したアクセストークンの取り消しの実装方法について説明する。

アクセストークンの取り消しは、インタフェース\ ``ConsumerTokenService``\ を実装したクラスの
\ ``revokeToken``\ メソッドを呼び出すことで実現できる。
クラス \ ``DefaultTokenService``\ はインタフェース\ ``ConsumerTokenService``\ を実装している。

アクセストークンの取り消し時に認可情報も削除することが可能である。
認可コードグラントやインプリシットグラントを使用している場合に、アクセストークンの取り消し後に認可情報を削除せずに認可リクエストを行うと、前回の認可リクエスト時の認可情報が再利用される場合がある。
前回の認可リクエスト時の認可情報は、認可情報の有効期限が有効であり、認可リクエストしたスコープが全て認可されている場合に再利用される。

以下に、実装例を示す。

トークンの取り消しを行うサービスクラスのインタフェースと実装クラスを作成する。

* ``RevokeTokenService.java``

.. code-block:: java

    public interface RevokeTokenService {
        
        String revokeToken(String tokenValue, String clientId);
        
    }

* ``RevokeTokenServiceImpl.java``

.. code-block:: java

    @Service
    @Transactional
    public class RevokeTokenServiceImpl implements RevokeTokenService {
        
        @Inject
        ConsumerTokenServices consumerService; // (1)
        
        @Inject
        TokenStore tokenStore; // (2)
        
        @Inject
        ApprovalStore approvalStore; // (3)
        
        public String revokeToken(String tokenValue, String clientId){ // (4)
            // (5)
            OAuth2Authentication authentication = tokenStore.readAuthentication(tokenValue);
            if (authentication != null) {                
                if (clientId.equals(authentication.getOAuth2Request().getClientId())) { // (6)
                    // (7)
                    Authentication user = authentication.getUserAuthentication();
                    if (user != null) {
                        Collection<Approval> approvals = new ArrayList<Approval>();
                        for (String scope : authentication.getOAuth2Request().getScope()) {
                            approvals.add(
                                    new Approval(user.getName(), clientId, scope, new Date(), ApprovalStatus.APPROVED));
                        }
                        approvalStore.revokeApprovals(approvals);
                    }
                    consumerService.revokeToken(tokenValue); // (8)
                    return "success";
                    
                } else {
                    return "invalid client";
                }
            } else {
                return "invalid token";
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
      - | アクセストークンの取り消しを行うインタフェース\ ``ConsumerTokenService``\ の実装クラスをインジェクションする。
    * - | (2)
      - | アクセストークン発行時の認証情報を取得するために使用する\ ``TokenStore``\ の実装クラスをインジェクションする。
    * - | (3)
      - | アクセストークンの発行時の認可情報を取得するために使用する\ ``ApprovalStore``\ の実装クラスをインジェクションする。
        | アクセストークンの取り消し時に認可情報を削除しない場合は不要となる。
    * - | (4)
      - | 取り消しを行うアクセストークンの値と、クライアントのチェックを行うために使用するクライアントIDをパラメータとして受け取る。
    * - | (5)
      - | \ ``TokenStore``\ の実装クラスの\ ``readAuthentication``\ メソッドを呼び出し、アクセストークンを発行した際の認証情報を取得する。
        | 認証情報が正常に取得できた場合のみ、トークンの削除処理を行う。
    * - | (6)
      - | 認証情報より、アクセストークン発行時に使用したクライアントIDを取得し、リクエストパラメータのクライアントIDと一致するかを確認する。
        | アクセストークン発行時のクライアントIDと一致する場合のみ、アクセストークンの削除を行う。
    * - | (7)
      - | 認証情報より、アクセストークン発行時のリソースオーナの認証情報を取得する。
        | リソースオーナの認証情報が取得できた場合、\ ``TokenStore``\ の実装クラスの\ ``revokeApprovals``\ メソッドを呼び出し、認可情報の削除を行う。
        | クライアントクレデンシャルグラントを使用している場合はリソースオーナの認証情報が存在しないため、\ ``revokeApprovals``\ メソッドに渡すパラメータが生成できない。
        | そのため、リソースオーナの認証情報が取得できない場合は認可情報の削除処理は行わない。
        | アクセストークンの取り消し時に認可情報を削除しない場合、この処理は不要となる。
    * - | (8)
      - | \ ``ConsumerTokenService``\ の実装クラスの\ ``revokeToken``\ メソッドを呼び出し、アクセストークンとアクセストークンに紐付くリフレッシュトークンの削除を行う。


トークンの取り消しリクエストを受けるコントローラを作成する。

* ``TokenRevocationRestController.java``

.. code-block:: java

    @RestController
    @RequestMapping("oth2")
    public class TokenRevocationRestController {
        
        @Inject
        RevokeTokenService revokeTokenService;
        
        @RequestMapping(value = "tokens/revoke", method = RequestMethod.POST) // (1)
        @ResponseStatus(HttpStatus.OK)
        public String revoke(@RequestParam("token") String token,
            @AuthenticationPrincipal UserDetails user){
            
            // (2)
            String clientId = user.getUsername();
            String result = revokeTokenService.revokeToken(token, clientId); // (3)
            return result;
        }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@RequestMapping``\ アノテーションを使用して、\ ``"/oth2/tokens/revoke"``\ へのアクセスに対するメソッドとしてマッピングを行う。
        | ここで指定するパスは\ :ref:`OAuthAuthorizationServerClientAuthentication`\で行った設定と同様に、Basic認証の適用とCSRF対策機能の無効化を行う必要がある。
    * - | (2)
      - | Basic認証で生成された認証情報からトークンの取り消し時のチェックで使用するクライアントIDを取得する。
    * - | (3)
      - | \ ``RevokeTokenService``\ を呼び出し、トークンの取り消しを行う。
        | リクエストパラメータとして受け取ったアクセストークンの値と、認証情報から取得したクライアントIDをパラメータとして渡す。

| 

.. tip::

    RFC 7009ではリクエストパラメータとして\ ``token_type_hint``\ を任意で付与してよいことが記載されている。
    \ ``token_type_hint``\ は削除対象のトークンがアクセストークンとリフレッシュトークンのどちらであるかを判別するためのヒントである。
    Spring Security OAuthが提供する\ ``ConsumerTokenService``\ はアクセストークンを渡すことでアクセストークンとリフレッシュトークンの両方削除するため、上記の実装例では使用していない。

.. note::

    認可サーバにトークンの取り消しをリクエストしたクライアントは、認可サーバの削除後にクライアントで保持しているトークンも取り消す必要がある。
    クライアントサーバのトークンの取り消しについては\ :ref:`OAuthClientServerHowToCancelToken`\を参照されたい。


|

.. _OAuthAuthorizationServerHowToControllTarnsaction:

トランザクション制御
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
認可サーバにおけるトランザクション制御の注意点について説明する。

認可サーバにおいてSpring Security OAuthが取り扱う情報（認可コード、認可情報、トークン）をDBにて管理する場合には、トランザクション制御の考慮が必要となる。

\ ``AuthorizationServerTokenServices``\ のデフォルト実装である\ ``DefaultTokenServices``\ ではアクセストークン、リフレッシュトークンを発行するメソッドである
\ ``createAccessToken``\ 、\ ``refreshAccessToken``\ にそれぞれ\ ``@Transactional``\ が付与されていることでトランザクション管理下で呼び出しが行われるが、それ以外は非トランザクション管理となる。

そのため、\ ``DataSource``\ から取得する\ ``Connection``\ の設定が\ ``autoCommit=false``\ となっている場合、管理対象となる情報がDBに登録されないためトランザクション管理が必須である。
また、\ ``autoCommit=true``\ の場合は必須ではないが、データの一貫性を担保するためのトランザクション管理の考慮が必要となるため、注意すること。

認可コード、認可情報をDBにて管理する場合のトランザクション制御設定例を以下に示す。

* ``oauth2-auth.xml``

.. code-block:: xml
    
    <!-- omitted -->
    
    <tx:advice id="oauthTransactionAdvice">
        <tx:attributes>
            <tx:method name="*"/>
        </tx:attributes>
    </tx:advice>
    
    <aop:config>
        <aop:pointcut id="authorizationOperation"
                      expression="execution(* org.springframework.security.oauth2.provider.code.AuthorizationCodeServices.*(..))"/> <!-- (1) -->
        <aop:pointcut id="approvalOperation"
                      expression="execution(* org.springframework.security.oauth2.provider.approval.UserApprovalHandler.*(..))"/> <!-- (2) -->
        <aop:advisor pointcut-ref="authorizationOperation" advice-ref="oauthTransactionAdvice"/>
        <aop:advisor pointcut-ref="approvalOperation" advice-ref="oauthTransactionAdvice"/>
    </aop:config>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | AOPを使用し、認可コードを操作する各メソッドにトランザクション境界を設定する。
    * - | (2)
      - | AOPを使用し、認可情報を操作する各メソッドにトランザクション境界を設定する。

|

リソースサーバの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ここではTODOリソースのREST APIに対して認可設定を行う実装例を用いて、リソースサーバの実装方法について説明する。

設定ファイルの作成（リソースサーバ）
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

リソースサーバを実装する際には新たにOAuth 2.0用のBean定義ファイルを作成する。

ここでは \ ``oauth2-resource.xml``\ とする。

\ ``oauth2-resource.xml``\ には以下の設定を追加する。

* ``oauth2-resource.xml``

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:oauth2="http://www.springframework.org/schema/security/oauth2"
           xmlns:sec="http://www.springframework.org/schema/security"
           xsi:schemaLocation="http://www.springframework.org/schema/security
               http://www.springframework.org/schema/security/spring-security.xsd
               http://www.springframework.org/schema/security/oauth2
               http://www.springframework.org/schema/security/spring-security-oauth2.xsd
               http://www.springframework.org/schema/beans
               http://www.springframework.org/schema/beans/spring-beans.xsd">

        <sec:http pattern="/api/v1/todos/**" create-session="stateless"
                       entry-point-ref="oauth2AuthenticationEntryPoint"> <!-- (1) -->
            <sec:access-denied-handler ref="oauth2AccessDeniedHandler"/> <!-- (2) -->
            <sec:csrf disabled="true"/> <!-- (3) -->
            <sec:custom-filter ref="oauth2AuthenticationFilter"
                                    before="PRE_AUTH_FILTER" /> <!-- (4) -->
        </sec:http>

        <bean id="oauth2AccessDeniedHandler"
                  class="org.springframework.security.oauth2.provider.error.OAuth2AccessDeniedHandler" /> <!-- (5) -->
        
        <bean id="oauth2AuthenticationEntryPoint"
                  class="org.springframework.security.oauth2.provider.error.OAuth2AuthenticationEntryPoint" /> <!-- (6) -->

        <oauth2:resource-server id="oauth2AuthenticationFilter" resource-id="todoResource"
                  token-services-ref="tokenServices" entry-point-ref="oauth2AuthenticationEntryPoint" /> <!-- (7) -->

    </beans>

|

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``pattern``\属性にはOAuth 2.0の認可設定の対象とするパスのパターンを指定する。
        | \ ``entry-point-ref``\には\ ``OAuth2AuthenticationEntryPoint``\のBeanを指定する。ここでの設定は定義上必要だが指定したBeanは使用されない。
          実際に使用されるのは後述する\ ``OAuth2AuthenticationProcessingFilter``\に指定している\ ``OAuth2AuthenticationEntryPoint``\のBeanである。
    * - | (2)
      - | \ ``access-denied-handler``\には\ ``OAuth2AccessDeniedHandler``\のBeanを設定する。ここでは後述する\ ``oauth2AccessDeniedHandler``\を指定している。
    * - | (3)
      - | CSRFトークンチェックによってPOST、PUT、DELETEといったリクエストを受けると必ずエラーになるため、CSRFを無効にする。
    * - | (4)
      - | \ ``custom-filter``\にはリソースサーバ用の認証フィルタを指定する。ここでは後述する\ ``oauth2AuthenticationFilter``\を指定している。
        | この指定により\ ``OAuth2AuthenticationProcessingFilter``\が設定される。
        | \ ``OAuth2AuthenticationProcessingFilter``\はリクエストに含まれるアクセストークンを利用してPre-Authenticationを行うためのフィルタであるため、
        | \ ``before``\に\ ``PRE_AUTH_FILTER``\を指定し\ ``PRE_AUTH_FILTER``\の前に\ ``OAuth2AuthenticationProcessingFilter``\の処理が実行されるように設定する。
        | Pre-Authenticationについては\ `こちら <http://docs.spring.io/spring-security/site/docs/3.0.x/reference/preauth.html>`_\を参照されたい。
    * - | (5)
      - | Spring Security OAuthが提供するリソースサーバ用の\ ``AccessDeniedHandler``\を定義する。
        | \ ``OAuth2AccessDeniedHandler``\は、認可エラー時に発生する例外をハンドリングしてエラー応答を行う。
    * - | (6)
      - | OAuth用のエラー応答を行うための\ ``OAuth2AuthenticationEntryPoint``\をBean定義する。
    * - | (7)
      - | Spring Security OAuthが提供するリソースサーバ用のServletFilterを定義する。
        | \ ``id``\属性に指定した文字列はBeanのIDとなる。ここでは ``oauth2AuthenticationFilter``\を指定している。
        | \ ``resource-id``\属性にはサーバが提供するリソースのIDを指定する。ここでは\ ``todoResource``\を指定している。
        | アクセストークンに紐付くクライアント情報のリソースIDに対し、\ ``resource-id``\属性に指定したリソースIDが含まれているか検証が行われる。
        | 検証の結果、リソースIDが含まれている場合のみリソースに対してのアクセスを許可する。
        | なお、\ ``resource-id``\の定義は任意であり、定義しない場合はリソースIDの検証が行われない。
        | \ ``token-services-ref``\属性には\ ``TokenServices``\のIDを指定する。\ ``TokenServices``\については後述する。
        | \ ``entry-point-reff``\属性には\ ``OAuth2AuthenticationEntryPoint``\のBeanを指定する。ここでは\ ``oauth2AuthenticationEntryPoint``\を指定している。

|

作成した\ ``oauth2-resource.xml``\を読み込むように\ ``web.xml``\に設定を追加する。


* ``web.xml``

.. code-block:: xml

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            classpath*:META-INF/spring/applicationContext.xml
            classpath*:META-INF/spring/oauth2-resource.xml <!-- (1) -->
            classpath*:META-INF/spring/spring-security.xml
        </param-value>
    </context-param>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | oauth2-resource.xmlで設定したパスのパターンを内包するようなパスが ``spring-security.xml``\にアクセス制御対象として設定されている場合を考慮し、先にoauth2-resource.xmlを読み込むようにする。

|

リソースにアクセス可能なスコープの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

リソースごとにアクセス可能なスコープを定義するために、OAuth 2.0用のBean定義ファイルに
スコープの定義とSpEL式をサポートするためのBean定義を追加する。


実装例は以下のとおりである。
なお、認可サーバ、リソース間でセキュリティが担保されていることを前提としているため、アクセス制御の設定において、クライアントに対するBasic認証は行っていない。

* ``oauth2-resource.xml``

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:oauth2="http://www.springframework.org/schema/security/oauth2"
           xmlns:sec="http://www.springframework.org/schema/security"
           xsi:schemaLocation="http://www.springframework.org/schema/security
               http://www.springframework.org/schema/security/spring-security.xsd
               http://www.springframework.org/schema/security/oauth2
               http://www.springframework.org/schema/security/spring-security-oauth2.xsd
               http://www.springframework.org/schema/beans
               http://www.springframework.org/schema/beans/spring-beans.xsd">

        <sec:http pattern="/api/v1/todos/**" create-session="stateless"
                       entry-point-ref="oauth2AuthenticationEntryPoint">
            <sec:access-denied-handler ref="oauth2AccessDeniedHandler"/>
            <sec:csrf disabled="true"/>
            <sec:expression-handler ref="oauth2ExpressionHandler"/> <!-- (1) -->
            <sec:intercept-url pattern="/**" method="GET"
                                    access="#oauth2.hasScope('READ')" /> <!-- (2) -->
            <sec:intercept-url pattern="/**" method="POST"
                                    access="#oauth2.hasScope('CREATE')" /> <!-- (2) -->
            <sec:custom-filter ref="oauth2ProviderFilter"
                                    before="PRE_AUTH_FILTER" />
        </sec:http>

        <!-- omitted -->
        
        <oauth2:web-expression-handler id="oauth2ExpressionHandler" /> <!-- (3) -->

    </beans>

|

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``expression-handler``\には\ ``OAuth2WebSecurityExpressionHandler``\のBeanを指定する。
    * - | (2)
      - | \ ``intercept-url``\を使用してリソースに対してスコープによるアクセスポリシーを定義する。
        | \ ``pattern``\属性には保護したいリソースのパスのパターンを指定する。本実装例では /api/v1/todos/ 配下のリソースが保護される。
        | \ ``method``\属性にはリソースのHTTPメソッドを指定する。
        | \ ``access``\属性にはリソースへのアクセスを認可するscopeを指定する。設定値は大文字、小文字を区別する。
        | ここではSpEL式を用いて指定を行っている。
    * - | (3)
      - | \ ``OAuth2WebSecurityExpressionHandler``\をBean定義する。
        | このBeanを定義することでSpring Security OAuthが提供するOAuth 2.0の認可設定を行うためのSpELがサポートされる。
        | なお、``id``\属性に指定した値がこのbeanのidとなる。

|

Spring Security OAuthが用意している主なExpressionを紹介する。

詳細については\ ``OAuth2SecurityExpressionMethods``\の\ `JavaDoc <http://docs.spring.io/spring-security/oauth/apidocs/org/springframework/security/oauth2/provider/expression/OAuth2SecurityExpressionMethods.html>`_\を参照されたい。

.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table:: **Spring Security OAuthが用意しているExpression**
    :header-rows: 1
    :widths: 35 65

    * - Expression
      - 説明
    * - | \ ``clientHasRole(String role)``\
      - | クライアントが引数に指定されたロールを持っている場合に\ ``true``\を返却する。
    * - | \ ``clientHasAnyRole(String... roles)``\
      - | クライアントが引数に指定されたいずれかのロールを持っている場合に\ ``true``\を返却する。
    * - | \ ``hasScope(String scope)``\
      - | クライアントがリソースオーナから認可されているスコープと引数のスコープが一致する場合に\ ``true``\を返却する。
    * - | \ ``hasAnyScope(String... scopes)``\
      - | クライアントがリソースオーナから認可されているスコープと引数のスコープのいずれかが一致する場合に\ ``true``\を返却する。
    * - | \ ``hasScopeMatching(String scopeRegex)``\
      - | クライアントがリソースオーナから認可されているスコープと引数に指定された正規表現が一致する場合に\ ``true``\を返却する。
    * - | \ ``hasAnyScopeMatching(String... scopesRegex)``\
      - | クライアントがリソースオーナから認可されているスコープと引数に指定されたいずれかの正規表現が一致する場合に\ ``true``\を返却する。
    * - | \ ``denyOAuthClient``\
      - | OAuth 2.0でのリクエストを拒否する。リソースオーナのみがリソースにアクセスできるようにするために使用される。
    * - | \ ``isOAuth``\
      - | OAuth 2.0でのリクエストを許可する。クライアントがリソースにアクセスできるようにするために使用される。

.. note::

    SpEL式についてはSpring Securityが提供するSpELを合わせて使用することができる。
    
    Spring Securityが提供するSpELについては :ref:`built-incommon-expressions` を参照されたい。

|

アクセストークンに関する設定（リソースサーバ）
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

認可サーバとリソースサーバはアクセストークンを\ ``TokenServices``\を介して連携する。

連携方法の種類については\ :ref:`OAuthAuthorizationServerHowToConfigureAccessToken`\を参照されたい。

ここではデータベースを共有する方法で設定を行う。

設定の解説については認可サーバでの\ ``TokenServices``\の設定と同様なため\ :ref:`OAuthAuthorizationServerHowToConfigureAccessToken`\を参照されたい。


* ``oauth2-resource.xml``

.. code-block:: xml

        <bean id="tokenServices"
            class="org.springframework.security.oauth2.provider.token.DefaultTokenServices">
            <property name="tokenStore" ref="tokenStore" />
        </bean>

        <bean id="tokenStore"
          class="org.springframework.security.oauth2.provider.token.store.JdbcTokenStore">
          <constructor-arg ref="tokenDataSource" />
        </bean>

.. note:: 

    認可サーバとリソースサーバを連携させる方法として、Spring Security OAuthで提供されている\ ``RemoteTokenServices``\を利用する方法や、
    JSON Web Tokenを利用する方法がある。
    Spring Security OAuthで提供されている\ ``RemoteTokenServices``\を利用する方法については\ :ref:`OAuthAuthorizationServerHowToCooperateWithHttp`\を参照されたい。

|

ユーザ情報の取得
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

リソースサーバでは、\ :ref:`SpringSecurityAuthenticationIntegrationWithSpringMVC`\で説明されている認証情報の取得方法と同様に、Controllerクラスのメソッド引数に\ ``UserDetails``\ を指定し\ ``@AuthenticationPrincipal``\ アノテーションを付与することにより認証されたリソースオーナの情報を受け取ることができる。
以下に実装例を示す。


.. code-block:: java

    @RestController
    @RequestMapping("api")
    public class TodoRestController {
        
        // omitted

        @RequestMapping(value = "todos", method = RequestMethod.GET)
        @ResponseStatus(HttpStatus.OK)
        public Collection<Todo> list(@AuthenticationPrincipal UserDetails user) { // (1)
        
            // omitted
        
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ 引数 \ ``user``\にリソースオーナの認証情報が格納される。

\ ``UserDetails``\ を指定した実装の場合、認証処理が行われないクライアントクレデンシャルグラントでは意図した情報が取得できないため注意が必要である。
クライアントクレデンシャルグラントを使用する場合は、Controllerのメソッド引数として\ ``String``\ を指定し\ ``@AuthenticationPrincipal``\ アノテーションを付与することによりクライアントIDを取得することができる。

また、\ ``UserDetails``\ を指定した実装の場合、クライアントID等のクライアントの認証情報は取得できない。
クライアントの認証情報を取得する場合は、Controllerクラスのメソッド引数に\ ``OAuth2Authentication``\ を指定することで可能である。
以下にControllerクラスのメソッド引数に\ ``OAuth2Authentication``\ を指定してクライアントとリソースオーナの認証情報を取得する例を示す。

.. code-block:: java

    @RestController
    @RequestMapping("api")
    public class TodoRestController {
        
        // omitted

        @RequestMapping(value = "todos", method = RequestMethod.GET)
        @ResponseStatus(HttpStatus.OK)
        public Collection<Todo> list(OAuth2Authentication authentication) { // (1)
        
            String username = authentication.getUserAuthentication().getName(); // (2)
            String clientId = authentication.getOAuth2Request().getClientId();  // (3)
        
            // omitted
        
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ 引数 \ ``authentication``\にリソースオーナ、クライアントの認証情報が格納される。
    * - | (2)
      - | \ \ ``authentication``\よりリソースオーナ名を取得する。
    * - | (3)
      - | \ \ ``authentication``\よりクライアントIDを取得する。

.. note::

    上記はアプリケーション層で\ ``OAuth2Authentication``\ を使用する場合の実装例である。
    \ ``OAuth2Authentication``\ に依存しない実装を行いたい場合\ ``HandlerMethodArgumentResolver``\ を実装することで同様の機能が実現可能である。
    具体的な実装方法については、\ :ref:`methodargumentresolver`\ を参照されたい。

| 

クライアントの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

クライアントの実装方法は、使用するグラントタイプにより、大きく以下の２つに分類される。


.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 35 65

    * - 実装方法
      - グラントタイプ
    * - | OAuth2RestTemplate
      - | 認可コードグラント
        | リソースオーナパスワードクレデンシャルグラント
        | クライアントクレデンシャルグラント
    * - | Javascript
      - | インプリシットグラント

本ガイドラインでは、上記の分類にて\ :ref:`OAuthClientUsingOAuth2RestTemplate`\ 、\ :ref:`OAuthClientUsingJavaScript`\ をそれぞれ説明する。

|

.. _OAuthClientUsingOAuth2RestTemplate:

OAuth2RestTemplateを使用したリソースへのアクセス
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

認可コードグラント、リソースオーナパスワードクレデンシャルグラント、クライアントクレデンシャルグラント向けの
クライアントの実装として、\ ``OAuth2RestTemplate``\ を使用してリソースへのアクセスを実現する方法を説明する。

Spring Security OAuthでは、\ ``RestOperations``\ のOAuth 2.0向けの実装として\ ``OAuth2RestTemplate``\ を提供している。

\ ``OAuth2RestTemplate``\ では、OAuth 2.0独自の機能として、\ ``AccessTokenProvider``\ によるグラントタイプに応じた
アクセストークンの取得や、\ ``OAuth2ClientContext``\ による複数リクエスト間でのアクセストークンの共有、リソースサーバへのアクセス時のエラーハンドリングといった機能を実装している。


クライアントでは、\ ``OAuth2RestTemplate``\を利用し、グラントタイプやスコープなどのアプリケーション要件に沿った
パラメータを定義することで、OAuth 2.0機能を使用したリソースへのアクセスが可能となる。

|

設定ファイルの作成（クライアント）
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

まず、OAuth 2.0に関する定義を行うための設定ファイルとして\ ``oauth2-client.xml``\を作成する。

* ``oauth2-client.xml``

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:sec="http://www.springframework.org/schema/security"
        xmlns:oauth2="http://www.springframework.org/schema/security/oauth2"
        xsi:schemaLocation="
            http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/security/oauth2 http://www.springframework.org/schema/security/spring-security-oauth2.xsd
        ">


    </beans>

|

\ ``web.xml``\に、作成した\ ``oauth2-client.xml``\を読み込む設定を追加する。

* ``web.xml``

.. code-block:: xml

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            classpath*:META-INF/spring/applicationContext.xml
            classpath*:META-INF/spring/oauth2-client.xml  <!-- (1) -->
            classpath*:META-INF/spring/spring-security.xml
        </param-value>
    </context-param>

    <!-- omitted -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``oauth2-client.xml``\ を読み込むように設定する。

|

OAuth2ClientContextFilterの適用
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

\ ``OAuth2ClientContextFilter``\ をサーブレットフィルタとして登録する。

\ ``OAuth2ClientContextFilter``\ を登録することでリソースオーナによる認可が取得できていない状態でリソースサーバへのアクセスを試みた場合に
発生する\ ``UserRedirectRequiredException``\を捕捉し、認可サーバが提供するリソースオーナの認可を取得するためのページへリダイレクトする
機能を組み込むことができる。

\ ``oauth2-client.xml``\ には以下の設定を追加する。

* ``oauth2-client.xml``

.. code-block:: xml

    <oauth2:client id="oauth2ClientContextFilter" /> <!-- (1) -->


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - |  \ ``<oauth2:client>``\ タグを使用することで、\ ``OAuth2ClientContextFilter``\のBeanが定義される。\ ``id``\属性に指定した文字列はBeanのIDとして使用される。

|

\ ``web.xml``\に、\ ``OAuth2ClientContextFilter``\の設定を追加する。

* ``web.xml``

.. code-block:: xml

    <filter> <!-- (1) -->
        <filter-name>oauth2ClientContextFilter</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>oauth2ClientContextFilter</filter-name>
        <url-pattern>/*</url-pattern> <!-- (2) -->
    </filter-mapping>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``DelegatingFilterProxy``\ を使用して、フィルタ名(\ ``<filter-name>``\ 要素に指定した値)とBean IDが一致するBeanをサーブレットフィルタとして登録する。
        | フィルタ名には\ ``<oauth2:client>``\ の\ ``id``\ 属性に指定したBean IDと同じ値を設定する。
        | なお、Spring Security OAuthで発生する例外が意図しない例外ハンドリングが行われないようにするために、\ ``OAuth2ClientContextFilter``\ はサーブレットフィルタの定義の一番最後に記述することを推奨する。
    * - | (2)
      - | \ ``UserRedirectRequiredException``\ が発生する可能性があるパスに対して\ ``OAuth2ClientContextFilter``\ を適用している。
        | 上記例では、すべてのリクエストに対して\ ``OAuth2ClientContextFilter``\ を適用している。

.. note::
    \ ``OAuth2ClientContextFilter``\ は、認可サーバが認可処理後にユーザエージェントを戻すリダイレクトURLの生成をフィルタの前処理で行っているため、
    \ ``/*``\ のような広範囲な定義にすると\ ``UserRedirectRequiredException``\ が発生しないパスに対して無駄な処理が行われることになる。

.. note::
    \ ``OAuth2ClientContextFilter``\ は、認可サーバがリソースオーナの認可を取得した後にリダイレクトさせるURLをリクエストスコープに
    \ ``currentUri``\ という属性名で格納する。そのため、クライアントでは\ ``currentUri``\ という属性名を使用することはできない。

|

前述のとおり、\ ``Oauth2ClientContextFilter``\ は\ ``UserRedirectRequiredException``\ を捕捉し、認可サーバに対してリダイレクトさせるサーブレットフィルタである。

ただし、ブランクプロジェクトで予め設定されている\ ``SystemExceptionResolver``\ が先に\ ``UserRedirectRequiredException``\ をハンドリングしてしまうと
\ ``Oauth2ClientContextFilter``\は期待した動作にならない。

そのため、\ ``spring-mvc.xm``\ の設定を変更し、\ ``SystemExceptionResolver``\ が\ ``UserRedirectRequiredException``\をハンドリングしないようにする必要がある。
\ ``SystemExceptionResolver``\ の詳しい解説については\ :doc:`../ArchitectureInDetail/WebApplicationDetail/ExceptionHandling`\を参照されたい。

\ ``spring-mvc.xml``\に、\ ``UserRedirectRequiredException``\ を\ ``SystemExceptionResolver``\ の除外対象として追加する。

* ``spring-mvc.xml``

.. code-block:: xml

    <bean id="systemExceptionResolver"
        class="org.terasoluna.gfw.web.exception.SystemExceptionResolver">
        
        <!-- omitted -->
    
        <property name="excludedExceptions">
            <array>
                <!-- (1) -->
                <value>org.springframework.security.oauth2.client.resource.UserRedirectRequiredException
                </value>
            </array>
        </property>
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``UserRedirectRequiredException``\ を\ ``SystemExceptionResolver``\ のハンドリング対象から除外する。

|

.. _OAuth2RestTemplateSettings:

OAuth2RestTemplateの設定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

各グラントタイプごとの\ ``OAuth2RestTemplate``\の設定方法を説明する。

認可コードグラント使用時のリソースの設定
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

認可コードグラントでアクセスする場合の\ ``OAuth2RestTemplate``\の設定例を示す。

* ``oauth2-client.xml``

.. code-block:: xml

    <oauth2:resource id="todoAuthCodeGrantResource" client-id="firstSec"
                     client-secret="firstSecSecret"
                     type="authorization_code"
                     scope="READ,WRITE"
                     access-token-uri="${auth.serverUrl}/oth2/token"
                     user-authorization-uri="${auth.serverUrl}/oth2/authorize"/> <!-- (1) -->

    <oauth2:rest-template id="todoAuthCodeGrantResourceRestTemplate" resource="todoAuthCodeGrantResource" /> <!-- (2) -->


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``OAuth2RestTemplate``\が参照する、アクセス対象となるリソースに関する詳細情報を定義する。
        | 各項目の設定値については下記表を参照のこと。
    * - | (2)
      - | \ ``OAuth2RestTemplate``\ を定義する。
        | \ ``id``\ には\ ``OAuth2RestTemplate``\ のBean IDを指定する。
        | \ ``resource``\には(1)で定義したBeanの\ ``id``\ を指定する。

|

     .. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
     .. list-table:: **リソース詳細情報**
         :header-rows: 1
         :widths: 35 65
     
         * - 項目
           - 説明
         * - | \ ``id``\
           - | リソースのBean ID。
         * - | \ ``client-id``\
           - | 認可サーバにてクライントを識別するID。
         * - | \ ``client-secret``\
           - | 認可サーバにてクライアントの認証に用いるパスワード。
         * - | \ ``type``\
           - | グラントタイプ。認可コードグラントの場合\ ``authorization_code``\を指定する。
         * - | \ ``scope``\
           - | 認可を要求するスコープをカンマ区切りで列挙する。設定値は大文字、小文字を区別する。
             | 省略時は認可サーバにおいてクライアントに対して設定しているスコープを全て要求する。
         * - | \ ``access-token-uri``\
           - | アクセストークンの発行を依頼するための認可サーバのエンドポイントのURL。
         * - | \ ``user-authorization-uri``\
           - | リソースオーナの認可を得るための認可サーバのエンドポイントのURL。

.. note::

    \ ``<oauth2:resource>``\ タグでは、アクセストークン取得時のクライアント認証方法を指定する方法として
    \ ``client-authentication-scheme``\ パラメータが用意されている。
    \ ``client-authentication-scheme``\ パラメータに指定可能な値は以下の通り。
    
        * \ ``header``\ : Authorizationヘッダを使用したBasic認証。デフォルト値。
        * \ ``query``\  : リクエスト時のURLクエリパラメータを使用した認証。
        * \ ``form``\   : リクエスト時のボディパラメータを使用した認証。
    
    本ガイドラインではクライアントの認証にBasic認証を利用するため上記の設定例では未指定としているが、
    アプリケーション要件に沿ったパラメータの指定を行うこと。
    
     .. code-block:: xml

        <oauth2:resource id="todoAuthCodeGrantResource" client-id="firstSec"
                         client-secret="firstSecSecret"
                         type="authorization_code"
                         scope="READ,WRITE"
                         access-token-uri="${auth.serverUrl}/oth2/token"
                         user-authorization-uri="${auth.serverUrl}/oth2/authorize"
                         client-authentication-scheme="form" />


.. _OAuth2ResourceOwnerPasswordCredentialGrantResourceSettings:

リソースオーナパスワードクレデンシャルグラント使用時のリソースの設定
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

| リソースオーナパスワードクレデンシャルグラントでは、クライアントがリソースオーナのユーザ名およびパスワードを使用してアクセストークンの発行を依頼する。
| \ ``OAuth2RestTemplate``\にはリソースオーナのユーザ名およびパスワードをそれぞれパラメータとして設定する必要があるが、
  複数のリソースオーナが同じクライアントを利用する場合、リソースオーナ毎に設定内容を切り替える考慮が必要となる。
| ここでは、\ ``OAuth2RestTemplate``\のリソースをSessionスコープのBeanで設定し、そのBeanにリソースオーナの情報を格納することによって、
  リソースオーナ毎の設定内容の切り替えを実現する方法を説明する。

リソースオーナパスワードクレデンシャルグラントでアクセスする場合の\ ``OAuth2RestTemplate``\の設定例を示す。

リソースオーナ毎に設定内容を切り替えられるよう、\ ``ResourceOwnerPasswordResourceDetails``\ をSessionスコープで定義し、\ ``OAuth2RestTemplate``\への設定を行う。

* ``oauth2-client.xml``

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:sec="http://www.springframework.org/schema/security"
        xmlns:oauth2="http://www.springframework.org/schema/security/oauth2"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xsi:schemaLocation="
            http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/security/oauth2 http://www.springframework.org/schema/security/spring-security-oauth2.xsd
            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
        ">

        <!-- omitted -->

        <bean id="todoPasswordGrantResource" class="org.springframework.security.oauth2.client.token.grant.password.ResourceOwnerPasswordResourceDetails"
            scope="session"> 
            <aop:scoped-proxy>
            <property name="clientId" value="firstSec" />
            <property name="clientSecret" value="firstSecSecret" />
            <property name="accessTokenUri" value="${auth.serverUrl}/oth2/token" />
            <property name="scope" value="READ,WRITE" />
        </bean> <!-- (1) -->

        <oauth2:rest-template id="todoPasswordGrantResourceRestTemplate" resource="todoPasswordGrantResource" /> <!-- (2) -->



.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``OAuth2RestTemplate``\が参照する、アクセス対象となるリソースに関する詳細情報を定義する。
        | 各項目の設定値については下記表を参照のこと。
    * - | (2)
      - | \ ``OAuth2RestTemplate``\を定義する。
        | \ ``id``\には\ ``OAuth2RestTemplate``\ のBean IDを指定する。
        | \ ``resource``\には(1)で定義したBeanの\ ``id``\ を指定する。


|

     .. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
     .. list-table::
         :header-rows: 1
         :widths: 35 65

         * - 項目
           - 説明
         * - | \ ``class``\
           - | \ ``OAuth2RestTemplate``\のリソースとするBeanを指定する。ここでは\ ``ResourceOwnerPasswordResourceDetails``\を指定する。
         * - | \ ``scope``\
           - | sessionを指定し、スコープ範囲をHTTPSessionとする。
         * - | \ ``<aop:scoped-proxy>``\
           - | SessionスコープのBeanをSingletonのBeanである\ ``OAuth2RestTemplate``\ にインジェクションするため設定する。
             | これは、SessionスコープのBeanよりSingletonのBeanの方がライフサイクルが長いため必要になる設定である。
             | このタグを使用するために\ ``aop``\のネームスペースとスキーマを追加している。
         * - | \ ``clientId``\プロパティ
           - |  Beanの\ ``clientId``\に対して認可サーバにてクライントを識別するIDを設定する。
         * - | \ ``clientSecret``\プロパティ
           - | Beanの\ ``clientSecrett``\に対して認可サーバにてクライアントの認証に用いるパスワードを設定する。
         * - | \ ``accessTokenUri``\プロパティ
           - | アクセストークンの発行を依頼するための認可サーバのエンドポイントのURLを指定する。
         * - | \ ``scope``\プロパティ
           - | Beanの\ ``scope``\に対して認可を要求するスコープの一覧を設定する。

|

.. note::

    リソースオーナのユーザ名、パスワードの取得は、アクセスが必要となったタイミングでクライアントの画面等でリソースオーナから入力され、Beanに格納することを想定している。
    本ガイドラインでは、ユーザ名、パスワードの具体的な取得方法については説明を割愛する。

.. warning::

    本ガイドラインで説明している認可サーバでは、認証に使用するパスワードに対してハッシュ化の後比較検証を行うため、\ ``ResourceOwnerPasswordResourceDetails``\に設定するパスワードは平文である必要がある。
    クライアントがリソースオーナのパスワードを平文で扱うことによるリスクが高いため、リソースオーナパスワードクレデンシャルグラントは
    クライアント-リソースオーナ間で高い信頼関係があり、かつクライアントがセキュアな環境に配置されているなどの非常に限定的な状況でのみ利用すること。
    認可サーバにおけるハッシュ化の設定については \ :ref:`OAuthAuthorizationServerClientAuthentication`\ を参照のこと。

|

クライアントクレデンシャルグラント使用時のリソースの設定
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

クライアントクレデンシャルグラントでアクセスする場合の\ ``OAuth2RestTemplate``\の設定例を示す。

なお、認可コードグラントと共通する設定についての解説は割愛する。

* ``oauth2-client.xml``

.. code-block:: xml

    <oauth2:resource id="todoClientGrantResource" client-id="firstSecClient"
                    client-secret="firstSecSecret"
                    type="client_credentials"
                    access-token-uri="${auth.serverUrl}/oth2/token" /> <!-- (1) -->

    <oauth2:rest-template resource=id="todoClientGrantResourceRestTemplate" resource="todoClientGrantResource" />


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``type``\属性にはグラントタイプを指定する。クライアントクレデンシャルグラントの場合\ ``client_credentials``\を指定する。


リソースサーバへのアクセス
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

\ ``OAuth2RestTemplate``\を用いてリソースサーバへアクセスする方法を説明する。

認可サーバへのアクセスは\ ``OAuth2RestTemplate``\と\ ``OAuth2ClientContextFilter``\により隠蔽されるため、開発の際には
認可サーバを意識する必要がなく、リソースサーバが提供するREST APIに対して行う処理を、通常のREST APIへのアクセスと同様に記述する。

以下にServiceクラスの実装例を示す。

.. code-block:: java

    import org.springframework.web.client.RestOperations;

    @Service
    public class TodoServiceImpl implements TodoService {

        @Inject
        @Named("todoAuthCodeGrantResourceRestTemplate")
        RestOperations restOperations; // (1)

        @Value("${resource.serverUrl}/api/v1/todos")
        String uri;

        @Override
        public List<Todo> getTodoList() {
            Todo[] todoArray = restOperations.
                getForObject(url, Todo[].class); // (2)
            return Arrays.asList(todoArray);
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``RestOperations``\をInejectする。
        | 上記例では、Bean IDが\ ``todoAuthCodeGrantResourceRestTemplate``\ であるBeanをInjectしている。\ ``@Named``\ の指定は\ ``OAuth2RestTemplate``\ が複数定義されている場合に必要となる。
    * - | (2)
      - | 指定したURLにRESTでメソッドGETでアクセスし結果をリストで受け取る。

.. note::

    \ ``OAuth2RestTemplate``\ を使用してリソースサーバにアクセスしようとした時点でアクセストークンが発行されていない場合、
    一度認可サーバにリダイレクトされる。
    その後、アクセストークンの発行が完了するとクライアントへリダイレクトされ、再度リソースサーバへのアクセス処理が呼び出される形となる。
    この時、クライアントへのリダイレクトはGETにより行われるため、認可サーバからリダイレクトされる可能性のあるControllerはGETを許容する必要がある。
    
    また、リダイレクト前のリソースサーバへのアクセスがPOSTである場合、リダイレクト後のGETではパラメータが失われてしまう。
    その場合、POSTパラメータをセッションに保持する等の対処が必要となるため、注意すること。
    本ガイドラインでは、具体的な対処方法の説明については割愛する。

|

.. _OAuthClientUsingJavaScript:

JavaScriptを使用したリソースへのアクセス
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

インプリシットグラントでは、Webブラウザなどのユーザエージェントが認可サーバに対して認可の要求を行う。

そのため、通常、インプリシットグラントはWebブラウザ上で動作するJavaScriptなどで利用されるが、
Spring Security OAuthではJavaScriptライブラリを提供していないため、インプリシットグラントを使用する場合は
独自にクライアントを実装する必要がある。

本ガイドラインでは、インプリシットグラント向けのクライアントの実装として、JavaScriptを使用してリソースサーバから
JSON形式のデータを取得し、画面に表示させる方法を説明する。

.. note::

    以降に説明する実装例ではJavaScriptライブラリとしてjQueryを使用する。
    jsファイルと合わせて \ ``src/main/webapp``\  配下に格納されていることを想定している。


OAuth 2.0機能を独自に実装したAPI例を示す。

* ``oauth2.js``

.. code-block:: js

    var oauth2Func = (function(exp, $) {
        "use strict";
    
        var
            config = {},
            DEFAULT_LIFETIME = 3600;
    
        var uuid = function() {
            return "xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx".replace(/[xy]/g, function(c) {
                var r = Math.random()*16|0, v = c == "x" ? r : (r&0x3|0x8);
                return v.toString(16);
            });
        };
    
        var encodeURL= function(url, params) {
            var res = url;
            var k, i = 0;
            for(k in params) {
                res += (i++ === 0 ? "?" : "&") + encodeURIComponent(k) + "=" + encodeURIComponent(params[k]);
            }
            return res;
        };
    
        var epoch = function() {
            return Math.round(new Date().getTime()/1000.0);
        };
    
        var parseQueryString = function (qs) {
            var e,
                a = /\+/g,
                r = /([^&;=]+)=?([^&;]*)/g,
                d = function (s) { return decodeURIComponent(s.replace(a, " ")); },
                q = qs,
                urlParams = {};
    
            while (e = r.exec(q)) {
               urlParams[d(e[1])] = d(e[2]);
            }
    
            return urlParams;
        };
    
        var saveState = function(state, obj) {
            localStorage.setItem("state-" + state, JSON.stringify(obj));
        };
    
        var getState = function(state) {
            var obj = JSON.parse(localStorage.getItem("state-" + state));
            localStorage.removeItem("state-" + state);
            return obj;
        };
    
        var hasScope = function(token, scope) {
            if (!token.scopes) return false;
            var i;
            for(i = 0; i < token.scopes.length; i++) {
                if (token.scopes[i] === scope) return true;
            }
            return false;
        };
    
        var filterTokens = function(tokens, scopes) { // (1)
            if (!scopes) scopes = [];
    
            var i, j,
            result = [],
            now = epoch(),
            usethis;
            for(i = 0; i < tokens.length; i++) {
                usethis = true;
    
                if (tokens[i].expires && tokens[i].expires < (now+1)) usethis = false;
    
                for(j = 0; j < scopes.length; j++) {
                    if (!hasScope(tokens[i], scopes[j])) usethis = false;
                }
    
                if (usethis) result.push(tokens[i]);
            }
            return result;
        };
    
        var saveTokens = function(provider, tokens) {
            localStorage.setItem("tokens-" + provider, JSON.stringify(tokens));
        };
    
        var getTokens = function(provider) {
            var tokens = JSON.parse(localStorage.getItem("tokens-" + provider));
            if (!tokens) tokens = [];
    
            return tokens;
        };
    
        var wipeTokens = function(provider) {
            localStorage.removeItem("tokens-" + provider);
        };
    
        var saveToken = function(provider, token) {
            var tokens = getTokens(provider);
            tokens = filterTokens(tokens);
            tokens.push(token);
            saveTokens(provider, tokens);
        };
    
        var getToken = function(provider, scopes) {
            var tokens = getTokens(provider);
            tokens = filterTokens(tokens, scopes);
            if (tokens.length < 1) return null;
            return tokens[0];
        };
    
        var sendAuthRequest = function(providerId, scopes) { // (2)
            if (!config[providerId]) throw "Could not find configuration for provider " + providerId;
            var co = config[providerId];
    
            var state = uuid();
            var request = {
                "response_type": "token"
            };
            request.state = state;
    
            if (co["redirectUrl"]) {
                request["redirect_uri"] = co["redirectUrl"];
            }
            if (co["clientId"]) {
                request["client_id"] = co["clientId"];
            }
            if (scopes) {
                request["scope"] = scopes.join(" ");
            }
    
            var authurl = encodeURL(co.authorization, request);
    
            if (window.location.hash) {
                request["restoreHash"] = window.location.hash;
            }
            request["providerId"] = providerId;
            if (scopes) {
                request["scopes"] = scopes;
            }
    
            saveState(state, request);
            redirect(authurl);
    
        };
    
        var checkForToken = function(providerId) { // (3)
            var h = window.location.hash;
    
            if (h.length < 2) return true;
    
            if (h.indexOf("error") > 0) { // (4)
                h = h.substring(1);
                var errorinfo = parseQueryString(h);
                handleError(providerId, errorinfo);
                return false;
            }
    
            if (h.indexOf("access_token") === -1) {
                return true;
            }
            h = h.substring(1);
            var atoken = parseQueryString(h);
    
            if (!atoken.state) {
                return true;
            }
    
            var state = getState(atoken.state);
            if (!state) throw "Could not retrieve state";
            if (!state.providerId) throw "Could not get providerId from state";
            if (!config[state.providerId]) throw "Could not retrieve config for this provider.";
    
            var now = epoch();
            if (atoken["expires_in"]) {
                atoken["expires"] = now + parseInt(atoken["expires_in"]);
            } else {
                atoken["expires"] = now + DEFAULT_LIFETIME;
            }
    
            if (atoken["scope"]) {
                atoken["scopes"] = atoken["scope"].split(" ");
            } else if (state["scopes"]) {
                atoken["scopes"] = state["scopes"];
            }
    
            saveToken(state.providerId, atoken);
    
            if (state.restoreHash) {
                window.location.hash = state.restoreHash;
            } else {
                window.location.hash = "";
            }
            return true;
        };
    
        var handleError = function(providerId, cause) { // (5)
            if (!config[providerId]) throw "Could not retrieve config for this provider.";
    
            var co = config[providerId];
            var errorDetail = cause["error"];
    
            // redirect error page
            if(co["errRedirectUrl"]) {
                redirect(co["errRedirectUrl"] + "/" + errorDetail);
            } else {
                alert("Access Error. cause: " + errorDetail);
            }
        };
    
    
        var redirect = function(url) {
            window.location = url;
        };
    
        var initialize = function(c) {
            config = c;
            try {
                var key, providerId;
                for(key in c) {
                        providerId = key;
                }
                return checkForToken(providerId);
            } catch(e) {
                console.log("Error when retrieving token from hash: " + e);
                window.location.hash = "";
                return false;
            }
        };
    
        var clearTokens = function() {
            var key;
            for(key in config) {
                wipeTokens(key);
            }
        };
    
        var oajax = function(settings) { // (6)
            var providerId = settings.providerId;
            var scopes = settings.scopes;
            var token = getToken(providerId, scopes);
    
            if (!token) {
                sendAuthRequest(providerId, scopes);
                return;
            }
    
            if (!settings.headers) settings.headers = {};
            settings.headers["Authorization"] = "Bearer " + token["access_token"];
    
            $.ajax(settings);
        };
    
        return {
            initialize: function(config) {
                return initialize(config);
            },
            clearTokens: function() {
                return clearTokens();
            },
            oajax: function(settings) {
                return oajax(settings);
            }
        };
    
    })(window, jQuery);


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | アクセストークンの有効期限とスコープの可否をチェックする関数。
        | クライントが使用可能なアクセストークンを返却する。
    * - | (2)
      - | 認可サーバに対して認可を要求する関数。
        | コンフィギュレーション情報より必要パラメータを取得し、リクエストを作成する。
    * - | (3)
      - | 認可の応答よりアクセストークンを取得する関数。
        | アクセストークンが取得できた場合、ローカルストレージに情報を格納する。
    * - | (4)
      - | 認可の結果、エラーが返却された場合エラー処理として\ ``handleError``\ を呼び出す。
    * - | (5)
      - | 認可時のエラーを処理する関数。
        | 本ガイドラインでは、エラー処理の実装例としてコンフィギュレーション情報にて指定されている
          エラー時のリダイレクト先URLへのリダイレクトを行っている。
    * - | (6)
      - | リソースサーバに対してリソースへのアクセスを要求する関数。
        | ローカルストレージよりアクセストークンを取得し、jQueryのajax関数を用いてリソースサーバへリクエストを行う。

|

以下にクライアントの実装例を示す。

* ``todoList.jsp``

.. code-block:: jsp

    <script type="text/javascript" src="${pageContext.request.contextPath}/resources/vendor/jquery/jquery.js"></script> <!-- (1) -->
    <script type="text/javascript" src="${pageContext.request.contextPath}/resources/app/js/oth2-implicit.js"></script> <!-- (1) -->
    <script type="text/javascript">
    "use strict";
    
    $(document).ready(function() {
        var result = oauth2Func.initialize({ // (2)
            "todo" : { // (3)
                clientId : "client", // (4)
                redirectUrl : "${client.serverUrl}/oth2/api", // (5)
                errRedirectUrl : "${client.serverUrl}/oth2/error", // (6)
                authorization : "${auth.serverUrl}/oth2/authorize" // (7)
            }
        });
    
        if (result) {
            oauth2Func.oajax({ // (8)
                url : "${resource.serverUrl}/api/v1/todos", // (9)
                providerId : "todo",  // (10)
                scopes : [ "READ" ],  // (11)
                dataType : "json",    // (12)
                type : "GET",  // (13)
                success : function(data) { // (14)
                    $("#message").text(JSON.stringify(data));
                },
                error : function() {
                    oauth2Func.clearTokens();
                }
            });
        } else {
            oauth2Func.clearTokens(); // (15)
        }
    
    
    };
    
    </script>
    <div id="wrapper">
        <p id="message"></p>
    </div>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | 後述する、OAuth 2.0機能を独自に実装したjsファイル、jQueryをそれぞれ格納したパスを指定する。
    * - | (2)
      - | 認可要求に使用するコンフィギュレーション情報を定義し、初期化する。
    * - | (3)
      - | クライアント別にコンフィギュレーション情報を区別するための識別子として一意な値を指定する。
        | 後述するリソースへのアクセス処理では、本項目をキーにコンフィギュレーション情報を管理・取得する。
    * - | (4)
      - | クライアントを識別するIDを指定する。
    * - | (5)
      - | 認可サーバのリソースオーナ認証後にクライアントをリダイレクトさせるURLを指定する。
        | 本実装例ではURLのパラメータをControllerから受け取ることを想定している。
    * - | (6)
      - | 認可応答として認可サーバよりエラーを受信した場合にリダイレクトさせるURLを指定する。
        | 本実装例ではURLのパラメータをControllerから受け取ることを想定している。
        | 本ガイドラインではエラー受信時の実装例として、クライアントの画面にリダイレクトしエラー画面を
          表示させる方法を示す。なお、クライアントのControllerについては説明を割愛する。
    * - | (7)
      - | 認可サーバの認可エンドポイントURLを指定する。
        | 本実装例ではURLのパラメータをControllerから受け取ることを想定している。
    * - | (8)
      - | リソースへのアクセスを実行する。
    * - | (9)
      - | リソースサーバのアクセス先URLを指定する。
    * - | (10)
      - | 参照するコンフィギュレーション情報の識別子を指定する。
    * - | (11)
      - | リソースへ要求するスコープを指定する。設定値は大文字、小文字を区別する。
    * - | (12)
      - | レスポンスの型を指定する。
    * - | (13)
      - | メソッドGETでリソースサーバへアクセスする。
    * - | (14)
      - | 処理成功時に行う処理を指定する。\ ``message``\には処理成功時のレスポンスが格納される。
    * - | (15)
      - | アクセストークンをクリアする。

|

 .. todo:: **TBD**

    JavaScriptで作られたクライアントから同一ドメインでない認可サーバやリソースサーバへのアクセスを行う場合、認可サーバやリソースサーバで\ ``Cross-Origin Resource Sharing``\のサポートが必要になる。

    詳細については、次版以降に記載する予定である。

.. _OAuthClientServerHowToCancelToken:

トークンの取り消し（クライアントサーバ）
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
発行したアクセストークンの取り消しの実装方法について説明する。

| アクセストークンの取り消しは、認可サーバにリクエストを行い、\ ``TokenStore``\ からアクセストークンの削除を行う。認可サーバへのリクエスト時はクライアントのBasic認証を行うため、Basic認証用のリクエストヘッダを設定する。
| 認可サーバのトークンの取り消しについては \ :ref:`OAuthAuthorizationServerHowToCancelToken`\ を参照されたい。
| クライアントサーバはアクセストークンの取り消しを認可サーバにリクエスト後、\ ``OAuth2RestTemplate``\で保持しているアクセストークンを削除する必要がある。


以下に、実装例を示す。

まず、認可サーバにトークン取り消し要求を行うための\ ``RestTemplate``\の設定を設定ファイルに追記する。

* ``oauth2-client.xml``

.. code-block:: xml
    
    <!-- (1) -->
    <bean id="revokeRestTemplate" class="org.springframework.web.client.RestTemplate">
        <property name="interceptors">
            <list>
                <ref bean="basicAuthInterceptor" />
            </list>
        </property>
    </bean>
    
    <bean id="basicAuthInterceptor" class="com.example.oauth2.client.restclient.BasicAuthInterceptor" />


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 認可サーバにトークン取り消し要求を行うための\ ``RestTemplate``\ をBean定義する。
        | Basic認証用のリクエストヘッダを設定するため、\ ``interceptors``\ プロパティに\ ``ClientHttpRequestInterceptor``\ の実装クラスを指定する。
        | Basic認証用のリクエストヘッダを設定する\ ``ClientHttpRequestInterceptor``\ の実装クラスの実装方法については\ :ref:`RestClientHowToExtendClientHttpRequestInterceptorBasicAuthentication`\ を参照されたい。


トークンの取り消しを行うサービスクラスのインタフェースと実装クラスを作成する。

* ``RevokeTokenClientService.java``

.. code-block:: java

    public interface RevokeTokenClientService {
        
        String revokeToken();
        
    }

* ``RevokeTokenClientServiceImpl.java``

.. code-block:: java

    @Service
    public class RevokeTokenClientServiceImpl implements RevokeTokenClientService {
    
        @Value("${auth.serverUrl}/api/v1/oth2/tokens/revoke")
        String revokeTokenUrl; // (1)
        
        @Inject
        @Named("todoAuthCodeGrantResourceRestTemplate")
        OAuth2RestOperations oauth2RestOperations; // (2)
        
        @Inject
        @Named("revokeRestTemplate")
        RestOperations revokeRestOperations; // (3)
        
        @Override
        public String revokeToken() {
            
            String token = getTokenValue(oauth2RestOperations);
            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
            MultiValueMap<String, String> variables = new LinkedMultiValueMap<String, String>();
            variables.add("token", token);
            
            String result = revokeRestOperations.postForObject(revokeTokenUrl,
                new HttpEntity<MultiValueMap<String, String>>(variables, headers),
                String.class); // (4)
            // (5)
            if ("success".equals(result)) {
                initContextToken(oauth2RestOperations);
            }
            return result;
        }
    
        // (6)
        private String getTokenValue(OAuth2RestOperations oauth2RestOperations) {
            String tokenValue = "";
            OAuth2AccessToken token = oauth2RestOperations.getAccessToken();
            if (token != null) {
                tokenValue = token.getValue();
            }
            return tokenValue;
        }
    
        // (7)
        private void initContextToken(OAuth2RestOperations oauth2RestOperations) {
            oauth2RestOperations.getOAuth2ClientContext().setAccessToken(null);
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | アクセストークンの取り消しを認可サーバに依頼する際に使用するURL。
    * - | (2)
      - | 取り消しを行うアクセストークンを保持している\ ``OAuth2RestTemplate``\ をインジェクションする。
    * - | (3)
      - | アクセストークンの取り消しを行う\ ``RestTemplate``\ をインジェクションする。
    * - | (4)
      - | 認可サーバにアクセストークンの取り消しを行うために、RESTでメソッドPOSTでアクセスする。
        | 取り消しを行うアクセストークンの値を認可サーバに渡すためにリクエストパラメータに設定する。
        | アクセストークンは(5)で定義している\ ``getTokenValue``\ メソッドに\ ``OAuth2RestOperations``\ を渡して取得する。
    * - | (5)
      - | 認可サーバの処理結果を判定し、正常の場合のみ\ ``OAuth2RestOperations``\で保持しているアクセストークンを削除する。
        | アクセストークンの削除は(6)で定義している\ ``initContextToken``\ メソッドにアクセストークンを保持している\ ``OAuth2RestOperations``\ を渡して削除する。
    * - | (6)
      - | \ ``OAuth2RestOperations``\ で保持しているアクセストークンを取得するメソッド。
        | パラメータとして渡された\ ``OAuth2RestOperations``\ の\ ``getAccessToken``\ メソッドを呼び出すことでアクセストークンを取得し、返却する。
    * - | (7)
      - | \ ``OAuth2RestOperations``\ で保持しているアクセストークンを削除するメソッド。
        | パラメータとして渡された\ ``OAuth2RestOperations``\ の\ ``setAccessToken``\ メソッドにnullを渡すことでアクセストークンを削除する。



| クライアントは上記で作成したサービスをアクセストークンが不要になったタイミングで呼び出すことでトークンの取り消しを行う。
| Spring Security OAuthのデフォルト実装ではセッションスコープでアクセストークンを保持するため、クライアントのユーザがログアウトした場合やセッションタイムアウトによってセッションが破棄されるタイミングでトークンの取り消しを行うことが考えられる。


.. _OAuthHowToExtend:

How to extend
--------------------------------------------------------------------------------

エンドポイントを介した認可サーバとリソースサーバの連携
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

リソースサーバと認可サーバは、認可サーバのチェックトークンエンドポイントにリソースサーバからHTTPアクセスを行うことで連携が可能である。

チェックトークンエンドポイントは、リソースサーバからアクセストークンの値を受け取り、リソースサーバの代わりにトークンの検証を行うエンドポイントである。
チェックトークンエンドポイントは\ ``TokenServices``\を用いてアクセストークンの取得、検証を行い、検証に問題がなければアクセストークンに紐づく情報をリソースサーバに渡す。


.. _OAuthAuthorizationServerHowToCooperateWithHttp:

HTTPアクセスを介した連携
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| リソースサーバが認可サーバのチェックトークンエンドポイントにアクセスするためには\ ``TokenServices``\の実装クラスである\ ``org.springframework.security.oauth2.provider.token.RemoteTokenServices``\ を使用する。
| \ ``RemoteTokenServices``\ は\ ``org.springframework.web.client.RestTemplate``\ を用いてチェックトークンエンドポイントへHTTPアクセスを行い、アクセストークンに紐づく情報を取得する。
| アクセストークンの検証はチェックトークンエンドポイントが行うため、\ ``RemoteTokenServices``\ では行わない。


以下に、実装例を示す。

まず、認可サーバにトークンを検証するための\ ``org.springframework.security.oauth2.provider.endpoint.CheckTokenEndpoint``\ クラスをコンポーネントとして登録する設定を行う。

* ``oauth2-auth.xml``

.. code-block:: xml

        <sec:http pattern="/oth2/check-token" security="none" />  <!-- (1) -->

        <oauth2:authorization-server
             client-details-service-ref="clientDetailsService"
             token-endpoint-url="/oth2/token"
             authorization-endpoint-url="/oth2/authorize"
             user-approval-handler-ref="userApprovalHandler"
             token-services-ref="tokenServices"
             check-token-enabled="true"
             check-token-endpoint-url="/oth2/check-token">  <!-- (2) -->
            <oauth2:authorization-code />
            <oauth2:implicit />
            <oauth2:refresh-token />
            <oauth2:client-credentials />
            <oauth2:password />
        </oauth2:authorization-server>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 本ガイドラインでは、チェックトークンエンドポイントへのアクセスはリソースサーバのみ許可するネットワーク構成になっていることを前提とし、Spring Securityのセキュリティ機能(Security Filter)の適用対象外とする。
        | 他のセキュリティ設定よりも先に評価されるように、設定ファイルの先頭に記載する必要がある。
    * - | (2)
      - | \ ``<oauth2:authorization-server>タグ``\ の\ ``check-token-enabled``\ 属性に\ ``true``\ を指定することで\ ``CheckTokenEndpoint``\ がコンポーネントとして登録される。
        | \ ``<oauth2:authorization-server>タグ``\ の\ ``check-token-endpoint-url``\ 属性に\ ``CheckTokenEndpoint``\ のURLを指定する。指定しない場合はデフォルト値である “/oauth/check_token” が指定される。


.. warning:: **チェックトークンエンドポイントのセキュリティ対策**

  - チェックトークンエンドポイントは、認可サーバとリソースサーバ間のみアクセス可能とし、オープンなネットワークからアクセスできないようにする。
    チェックトークンエンドポイントを公開する場合は、適切なセキュリティ対策を行う必要がある。


.. note::

    Spring Security OAuthのバージョン2.0.12以前を使用する場合、\ ``check-token-endpoint-url``\ は、\ ``authorization-endpoint-url``\ または\ ``token-endpoint-url``\ を指定していない場合は反映されないため注意が必要である。
    これは以下のissueで取り上げられており、バージョン2.0.13で改修される予定である。
    
    https://github.com/spring-projects/spring-security-oauth/issues/897


.. note::

    \ ``TokenServices``\ は、共有DBを介して連携させる場合と同様に\ ``DefaultTokenServices``\ を使用する。
    \ ``TokenServices``\が参照する\ ``TokenStore``\はアプリケーションの要件に合ったインタフェースの実装クラスを使用する。

| 

リソースサーバの設定ファイルに\ ``TokenServices``\ として\ ``org.springframework.security.oauth2.provider.token.RemoteTokenServices``\ を使用する設定を行う。

* ``oauth2-resource.xml``

.. code-block:: xml

        <bean id="tokenServices"
            class="org.springframework.security.oauth2.provider.token.RemoteTokenServices">
            <property name="checkTokenEndpointUrl" value="${auth.serverUrl}/oth2/check-token" />  <!-- (1) -->
        </bean>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 認可サーバのチェックトークンエンドポイントにアクセスしてアクセストークンに紐付く情報を取得できるよう、\ ``RemoteTokenServices``\ をBean定義する。
        | チェックトークンエンドポイントにアクセスするためのURLを\ ``checkTokenEndpointUrl``\ プロパティに設定する。


.. note::

    チェックトークンエンドポイントでアクセストークンの検証エラーが発生した場合、\ ``RemoteTokenServices``\ にHTTPステータスコード400(Bad Request)が返却される。
    \ ``RestTemplate``\ のデフォルト実装ではHTTPステータスコード400(Bad Request)が返却された場合、エラーハンドリングを行いクライアントエラー例外を発生させる。
    \ ``RemoteTokenServices``\ がデフォルトで使用する\ ``RestTemplate``\ はアクセストークンの検証エラーをクライアントサーバに連携するために、レスポンスのHTTPステータスコードが400の場合はエラーハンドリングしないよう拡張されている。
    \ ``RemoteTokenServices``\ に\ ``RestTemplate``\ をインジェクションした場合、この拡張が適用されなくなるため注意が必要である。

| 

\ ``RemoteTokenServices``\ をリソースサーバで使用した場合、ハンドラメソッドでアノテーション\ ``@AuthenticationPrincipal``\ を\ ``String``\ に引数アノテーションとして指定することでリソースオーナのユーザ名が取得できる。

実装例は以下のようになる。

.. code-block:: java

    @RestController
    @RequestMapping("api")
    public class TodoRestController {
        
        // omitted

        @RequestMapping(value = "todos", method = RequestMethod.GET)
        @ResponseStatus(HttpStatus.OK)
        public Collection<Todo> list(@AuthenticationPrincipal String userName) { // (1)
        
            // omitted
        
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ 引数 \ ``userName``\にリソースオーナのユーザ名が格納される。


| 

.. _OAuthAuthorizationServerHowToGetOtherThanUserName:

DefaultAccessTokenConverterの拡張
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

認可サーバとリソースサーバ間でDBを共有しない構成の場合でも、ユーザ情報に付随する項目を認可サーバからリソースサーバに連携したいというケースはありうるが、
リソースサーバの使用する\ ``TokenServices``\ として\ ``RemoteTokenServices``\ を使用する場合、リソースサーバのハンドラメソッド引数のアノテーション\ ``@AuthenticationPrincipal``\ ではユーザ名以外の情報を取得することができない。

そこで、ここでは\ ``RemoteTokenServices``\ を使用してアクセストークンを連携するときに使用するクラスである\ ``org.springframework.security.oauth2.provider.token.DefaultAccessTokenConverter``\ を拡張し、
ユーザ名以外の情報をリソースサーバに連携する例を示す。

DefaultAccessTokenConverterとは
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

\ ``RemoteTokenServices``\ を使用したアクセストークンの連携では、\ ``RestTemplate``\ を使用してリソースサーバから認可サーバに対してアクセストークン値に紐づくリソースオーナ、クライアントの認証情報を要求し、結果をMapとして取得する。
このとき、\ ``DefaultAccessTokenConverter``\ は、認可サーバでは認証情報からMapへ、リソースサーバではMapから認証情報へ変換するためのコンバーターとしての役割を持つ。

これを利用し、認可サーバからの返却値をMapに追加するよう\ ``DefaultAccessTokenConverter``\ の拡張を行うことで、認可サーバ、リソースサーバ間で連携するパラメータをカスタマイズすることが出来るようになる。

以下の説明では、認可サーバ側で\ ``DefaultAccessTokenConverter``\ と、そのプロパティである\ ``DefaultUserAuthenticationConverter``\ をそれぞれカスタマイズすることで、ユーザ情報に関連した独自項目と、それ以外の独自項目を連携する例を示す。


認可サーバの実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

認可サーバ側の実装方法について説明する。

まず、ユーザ情報に関連した独自項目を追加するため、\ ``DefaultUserAuthenticationConverter``\ を拡張する。

* ``AuthCustomUserTokenConverter.java``

.. code-block:: java

    public class CustomUserTokenConverter extends DefaultUserAuthenticationConverter {
        @Override
        public Map<String, ?> convertUserAuthentication(
                Authentication authentication) {
            Map<String, Object> response = new LinkedHashMap<String, Object>();
            response.put(USERNAME, authentication.getName());
            
            if (authentication.getAuthorities() != null &&
                    !authentication.getAuthorities().isEmpty()) {
                response.put(AUTHORITIES, AuthorityUtils.authorityListToSet(
                        authentication.getAuthorities()));
            }
            response.put("user_additional_key", "user_additional_value"); // (1)
            return response;
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | リソースサーバに引き渡す情報を独自項目\ ``user_additional_key``\ として定義し、\ ``response``\ に設定する。
        | \ ``response``\ に設定した情報は、チェックトークンエンドポイントのトークン検証時にレスポンスBODYとしてJSON形式でリソースサーバへ返却される。


次に、ユーザ情報以外の独自項目を追加するため、\ ``DefaultAccessTokenConverter``\ を拡張する。

* ``CustomAccessTokenConverter.java``

.. code-block:: java

        public class CustomAccessTokenConverter extends DefaultAccessTokenConverter {
        
            @Override
            public Map<String, ?> convertAccessToken(OAuth2AccessToken token, OAuth2Authentication authentication) {
                
                @SuppressWarnings("unchecked")
                Map<String, Object> response = (Map<String, Object>) super.convertAccessToken(token, authentication);
                response.put("client_additional_key","client_additional_value"); // (1)
                // omitted
                
                return response;
            }
        }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | リソースサーバに引き渡す情報を独自項目\ ``client_additional_key``\ として定義し、\ ``response``\ に設定する。
        | \ ``response``\ に設定した情報は、チェックトークンエンドポイントのトークン検証時にレスポンスBODYとしてJSON形式でリソースサーバへ返却される。

認可サーバの設定ファイルに、作成した\ ``CustomUserTokenConverter``\ 、\ ``CustomAccessTokenConverter``\ の設定を行う。

* ``oauth2-auth.xml``

.. code-block:: xml

        <oauth2:authorization-server
             client-details-service-ref="clientDetailsService"
             token-endpoint-url="/oth2/token"
             authorization-endpoint-url="/oth2/authorize"
             user-approval-handler-ref="userApprovalHandler"
             token-services-ref="tokenServices"
             check-token-endpoint-url="/oth2/check-token">  <!-- (1) -->
            <oauth2:authorization-code />
            <oauth2:implicit />
            <oauth2:refresh-token />
            <oauth2:client-credentials />
            <oauth2:password />
        </oauth2:authorization-server>

        <bean id="checkTokenEndpoint"
            class="org.springframework.security.oauth2.provider.endpoint.CheckTokenEndpoint">  <!-- (2) -->
            <constructor-arg ref="tokenServices" />
            <property name="accessTokenConverter" ref="accessTokenConverter" />
        </bean>

        <bean id="accessTokenConverter"
            class="com.example.oauth2.auth.converter.CustomAccessTokenConverter"/>  <!-- (3) -->
            <property name="userTokenConverter">
                <bean
                    class="com.example.oauth2.auth.converter.CustomUserTokenConverter" />
            </property>
        </bean>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``<oauth2:authorization-server>タグ``\ の\ ``check-token-endpoint-url``\ 属性に(2)で定義している\ ``CheckTokenEndpoint``\ のURLを指定する。指定しない場合はデフォルト値である “/oauth/check_token” が指定される。
        | \ ``CheckTokenEndpoint``\ のBean定義を(2)で独自で行っているため、\ ``<oauth2:authorization-server>タグ``\ の\ ``check-token-enabled``\ 属性は指定しない。          
    * - | (2)
      - | \ ``CheckTokenEndpoint``\ をBean定義する。
        | \ ``accessTokenConverter``\ プロパティに(2)で定義している\ ``CustomAccessTokenConverter``\ のBeanを指定することで\ ``CustomAccessTokenConverter``\ と\ ``CustomUserTokenConverter``\ に追加した独自項目をリソースサーバに連携するようになる。
    * - | (3)
      - | \ ``DefaultAccessTokenConverter``\ を拡張した\ ``CustomAccessTokenConverter``\ をBean定義する。
        | \ ``userTokenConverter``\ プロパティに\ ``DefaultUserAuthenticationConverter``\ を拡張した\ ``CustomUserTokenConverter``\ のBeanを指定する。


リソースサーバの実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

リソースサーバに、認可サーバから連携された情報をハンドラメソッド引数のアノテーション\ ``@AuthenticationPrincipal``\ で取得できるよう機能の追加を行う。
まず、アノテーション\ ``@AuthenticationPrincipal``\ で取得する情報を保持する\ ``OauthUser``\ クラスを作成する。

* ``OauthUser.java``

.. code-block:: java

        public class OauthUser implements Serializable{
        
            private static final long serialVersionUID = 1L;
            
            private String username;
            
            private String userAdditionalValue;
            
            private String clientAdditionalValue;
            
            // omitted
            
            public User(String username, String additionalValue){
                this.username = username;
                this.additionalValue = additionalValue;
            }
            
            // Getters and Setters are omitted
            
        }

アノテーション\ ``@AuthenticationPrincipal``\ でユーザ情報が取得できるように設定を行う\ ``org.springframework.security.oauth2.provider.token.DefaultAccessTokenConverter``\ を拡張し、ユーザ名以外の情報も取得できるよう機能の追加を行う。

* ``CustomUserTokenConverter.java``

.. code-block:: java

        public class CustomUserTokenConverter extends DefaultUserAuthenticationConverter{
            
            private Collection<? extends GrantedAuthority> defaultAuthorities; // (1)
            
            public void setDefaultAuthorities(String[] defaultAuthorities) {
                this.defaultAuthorities = AuthorityUtils.commaSeparatedStringToAuthorityList(StringUtils
                        .arrayToCommaDelimitedString(defaultAuthorities));
            }
            
             // (2)
            @Override
            public Authentication extractAuthentication(Map<String, ?> map) {
                if (map.containsKey(USERNAME)) {
                    Collection<? extends GrantedAuthority> authorities = getAuthorities(map);
                    OauthUser user = new OauthUser(
                            (String) map.get(USERNAME),
                            (String) map.get("user_additional_key"),
                            (String) map.get("client_additional_key"),
                            (String) map.get("client_id")); // (3)
                    
                    // omitted
                    
                    return new UsernamePasswordAuthenticationToken(user, "N/A", authorities); // (4)
                }
                return null;
            }
            
            private Collection<? extends GrantedAuthority> getAuthorities(Map<String, ?> map) {
                if (!map.containsKey(AUTHORITIES)) {
                    return defaultAuthorities;
                }
                Object authorities = map.get(AUTHORITIES);
                if (authorities instanceof String) {
                    return AuthorityUtils.commaSeparatedStringToAuthorityList((String) authorities);
                }
                if (authorities instanceof Collection) {
                    return AuthorityUtils.commaSeparatedStringToAuthorityList(StringUtils
                            .collectionToCommaDelimitedString((Collection<?>) authorities));
                }
                throw new IllegalArgumentException("Authorities must be either a String or a Collection");
            }
        }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``DefaultUserAuthenticationConverter``\ に実装されている\ ``getAuthorities``\ メソッドがprivateで定義されているため、\ ``getAuthorities``\ メソッドで使用される\ ``defaultAuthorities``\ と\ ``getAuthorities``\ メソッドを実装する。
    * - | (2)
      - | 認可サーバから連携された情報から認証情報を抽出するメソッド。
    * - | (3)
      - | 認可サーバから連携された情報を\ ``OauthUser``\ クラスに設定する。
    * - | (4)
      - | \ ``UsernamePasswordAuthenticationToken``\ の第一引数に\ ``OauthUser``\ を設定することで、認可サーバから連携された情報をアノテーション\ ``@AuthenticationPrincipal``\ で取得できるようになる。



リソースサーバの設定ファイルに、\ ``CustomUserTokenConverter``\ の設定を行う。

* ``oauth2-resource.xml``

.. code-block:: xml

        <bean id="tokenServices"
            class="org.springframework.security.oauth2.provider.token.RemoteTokenServices">
            <property name="checkTokenEndpointUrl" value="${auth.serverUrl}/oth2/check-token" />
            <property name="accessTokenConverter" ref="accessTokenConverter" />
        </bean>
        
        <bean id="accessTokenConverter"
            class="org.springframework.security.oauth2.provider.token.DefaultAccessTokenConverter">
            <property name="userTokenConverter">
                <bean class="com.example.oauth2.resource.converter.CustomUserTokenConverter"/>  <!-- (1) -->
            </property>
        </bean>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``accessTokenConverter``\ の\ ``userTokenConverter``\ プロパティに\ ``CustomUserTokenConverter``\ クラスを指定することで、ユーザ名以外の情報をアノテーション\ ``@AuthenticationPrincipal``\ で取得できるようになる。


