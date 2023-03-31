.. _OAuth2:

OAuth 2.0
================================================================================

.. only:: html

.. contents:: 目次
  :local:

|

.. _OAuth2Overview:

Overview
--------------------------------------------------------------------------------
本節では、OAuth 2.0の概要とSpringプロジェクトの一つであるSpring Securityを使用してOAuth 2.0の仕様に沿った認可制御機能を実装する方法について説明する。

.. tip:: \ **Spring Security が提供するOAuth 2.0のリファレンス**\

  Spring Security が提供するOAuth 2.0は、本ガイドラインで紹介していない機能も提供している。Spring Security が提供するOAuth 2.0について詳しく知りたい場合は、\ `OAuth2 <https://docs.spring.io/spring-security/reference/6.0.1/servlet/oauth2/index.html>`__\ を参照されたい。

|

.. _OAuth2AboutOAuth2.0:

OAuth 2.0とは
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

OAuth 2.0とは、サードパーティ製アプリケーションがHTTPサービスを利用する際に、サーバ上の保護されたリソースに対するアクセス範囲の指定を可能にするための認可フレームワークのことである。

OAuth 2.0はRFCとして仕様化されており、関連する複数の技術仕様から構成されている。

以下にOAuth 2.0の主要な仕様を示す。

.. tabularcolumns:: |p{0.15\linewidth}|p{0.30\linewidth}|p{0.55\linewidth}|
.. list-table:: \ **OAuth 2.0の主要仕様**\
  :header-rows: 1
  :widths: 15 30 55
  :class: longtable

  * - RFC
    - 概要
    - 説明
  * - | RFC 6749
    - | \ `The OAuth 2.0 Authorization Framework <http://tools.ietf.org/html/rfc6749>`_\
    - | 用語や認可方式などの、OAuth 2.0としてのもっとも基本的な内容が記載されている技術仕様。
  * - | RFC 6750
    - | \ `Bearer Token Usage <https://tools.ietf.org/html/rfc6750>`_\
    - | RFC 6749に記載されている認可制御を実現する場合に利用する、「署名なしアクセストークン」（以降、アクセストークンと表す）のサーバ間の受け渡し方法に関する技術仕様。
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

従来のクライアントサーバ型の認証モデルでは、サードパーティ製アプリケーションはHTTPサービスの保護されたリソースにアクセスするために、ユーザの認証情報（ユーザ名とパスワードなど）を利用して認証を行う。

つまり、ユーザは、サードパーティ製アプリケーションにリソースへのアクセス権を与えるために自身の認証情報をサードパーティと共有する必要があるが、これはサードパーティ製アプリケーションに不具合や悪意のある操作などが存在した場合に、ユーザの意図しないアクセスや情報漏洩等のリスクにつながる。

.. figure:: ./images_OAuth2/OAuth2_TraditionalAuthenticationModel.png
  :width: 100%


これに対し、OAuth 2.0ではHTTPサービスとの認証はユーザが直接行い、サードパーティ製アプリケーションには「アクセストークン」と呼ばれる認証済みリクエストを行うための情報を払い出すことで、サードパーティに認証情報を共有することなくリソースへアクセスすることが可能となる。

また、アクセストークン発行時にリソースに対するアクセス範囲（スコープ）を指定可能とすることで従来のクライアントサーバ型の認証モデルと比較してより柔軟なアクセス制御を実現している。

.. figure:: ./images_OAuth2/OAuth2_OAuthAuthenticationModel.png
  :width: 100%

|

.. _OAuth2Architecture:

OAuth 2.0のアーキテクチャ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| ここではOAuth 2.0が定義するロール、スコープ、認可グラント及びプロトコルフローについて説明する。
| OAuth 2.0ではスコープや認可グラントという概念を定義しており、これらの概念を使用して認可の仕様を定めている。

|

.. _OAuth2Role:

ロール
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
OAuth 2.0ではロールとして以下の4つを定義している。

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: \ **OAuth 2.0におけるロール**\
  :header-rows: 1
  :widths: 25 75
  :class: longtable

  * - ロール名
    - 説明
  * - | リソースオーナ
    - | 保護されたリソースへのアクセスを許可するロール。人（エンドユーザ）など。
  * - | リソースサーバ
    - | 保護されたリソースを提供するサーバ。
  * - | 認可サーバ
    - | リソースオーナの認証と、アクセストークン（クライアントがリソースサーバにアクセスするときに必要な情報）の発行を行うサーバ。
  * - | クライアント
    - | リソースオーナの認可を得て、リソースオーナの代理として保護されたリソースに対してリクエストを行うロール。Webアプリケーションなど。クライアント情報は事前に認可サーバに登録され、認可サーバ内で一意な情報であるクライアントIDにより管理される。

      | OAuth 2.0ではクライアントクレデンシャル（クライアントの認証情報）の機密性を維持できる能力に基づき、クライアントタイプとして以下の2つを定義している。

      | (1) コンフィデンシャル
      |     クライアントクレデンシャルの機密性を維持することができるクライアント。
      | (2) パブリック
      |     リソースオーナのデバイス上で実行されるクライアントのように、クライアントクレデンシャルの機密性を維持することができず、かつ他の手段を用いたセキュアなクライアント認証が行えないクライアント。
      |
      | また、OAuth 2.0ではクライアントとして以下のような例を考慮して設計されている。
      | (1) コンフィデンシャル

      * | Webアプリケーション（Web application）
        | アプリケーションサーバ上で実行されるクライアント。

      | (2) パブリック

      * | ユーザエージェントベースアプリケーション（user-agent-based application）
        | クライアントコードがアプリケーションサーバからダウンロードされリソースオーナのユーザエージェント（ブラウザなど）内で実行されるクライアント。
        | JavaScriptアプリケーションなど。
      * | ネイティブアプリケーション（native application）
        | リソースオーナのデバイス上にインストールされ実行されるクライアント。

|

.. note::

  ユーザエージェントは、リソースオーナが使用するWebブラウザ等を指す。本ガイドラインでは、エンドユーザの操作が発生する箇所を明確にするため、リソースオーナ（エンドユーザ）とユーザエージェントを別のものとして解説する。
  
  ガイドラインでリソースオーナと明示している場合に、エンドユーザの操作が発生する。

|

.. _OAuth2Scope:

スコープ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
OAuth 2.0では保護されたリソースに対するアクセスを制御する方法としてスコープという概念を使用している。

認可サーバはクライアントからの要求に対し、認可サーバのポリシーまたはリソースオーナの指示に基づいてアクセストークンにスコープを含め、保護されたリソースに対するアクセス権（読み込み権限、書き込み権限など）を指定することが出来る。

|

.. _OAuth2ProtocolFlow:

プロトコルフロー
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
OAuth 2.0では、以下のような流れでリソースへのアクセスを行う。

.. figure:: ./images_OAuth2/OAuth2_ProtocolFlow.png
  :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: \ **OAuth 2.0のプロトコルフロー**\
  :header-rows: 1
  :widths: 10 90
  :class: longtable

  * - 項番
    - 説明
  * - | (1)
    - | リソースオーナに対して認可を要求する。上の図ではクライアントがリソースオーナに直接要求を行っているが、認可サーバを経由して行うほうが望ましい。
      | 後述するグラントタイプの中では認可コードグラントとインプリシットグラントが認可サーバを経由してリソースオーナに要求を行うフローになっている。
  * - | (2)
    - | クライアントはリソースオーナからの認可を表すクレデンシャルとして認可グラント（後述）を受け取る。
  * - | (3)
    - | クライアントは、認可サーバに対して自身の認証情報とリソースオーナが与えた認可グラントを提示することで、アクセストークンを要求する。
  * - | (4)
    - | 認可サーバはクライアントを認証し、認可グラントの正当性を確認する。認可グラントが正当な場合、アクセストークンを発行する。
  * - | (5)
    - | クライアントはリソースサーバの保護されたリソースへリクエストを行い、発行されたアクセストークンにより認証する。
  * - | (6)
    - | リソースサーバはアクセストークンの正当性を確認し、正当な場合、リクエストを受け入れリソースを応答する。

.. note::

  OAuth 1.0で不評だった署名とトークン交換の複雑な仕組みを簡略化するために、OAuth 2.0ではアクセストークンを扱うリクエストはHTTPS通信で行うことを必須としている。（HTTPS通信を使用することでアクセストークンの盗聴を防止する）

|

.. _OAuth2AuthorizationGrant:

認可グラント
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
認可グラントは、リソースオーナからの認可を表し、クライアントがアクセストークンを取得する際に用いられる。OAuth 2.0では、グラントタイプとして以下の4つを定義しているが、クレデンシャル項目を追加するなどの独自拡張を行うこともできる。

| クライアントはいずれかのグラントタイプを利用して認可サーバへアクセストークンを要求し、取得したアクセストークンでリソースサーバにアクセスする。
| 認可サーバはサポートするグラントタイプを必ず1つ以上定義しており、その中から使用するグラントタイプをクライアントからの認可リクエストによって決定する。

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: \ **OAuth 2.0における認可グラント**\
  :name: OAuth2AuthorizationGrantTable
  :header-rows: 1
  :widths: 25 75
  :class: longtable

  * - グラントタイプ
    - 説明
  * - | 認可コードグラント
    - | 認可コードグラントのフローでは、認可サーバがクライアントとリソースオーナの仲介となって認可コードをクライアントへ発行し、クライアントが認可コードを認可サーバに渡すことでアクセストークンを発行する。
      | 認可サーバが発行した認可コードを使用してアクセストークンを発行するため、クライアントへリソースオーナのクレデンシャルを共有する必要がない。
      | 認可コードグラントはWebアプリケーションのように、コンフィデンシャルなクライアントがOAuth 2.0を利用する際に使用する。
  * - | インプリシットグラント
    - | インプリシットグラントのフローでは、認可コードグラントと同様に認可サーバが仲介するが、認可コードの代わりに直接アクセストークンを発行する。
      | これにより応答性、効率性が高いため、スクリプト言語を使用してブラウザ上で実行されるクライアントに適している。
      | しかし、アクセストークンがURL中にエンコードされるため、リソースオーナや同一デバイス上の他のアプリケーションに漏えいする可能性があるほか、クライアントの認証を行わないことから、他のクライアントに対して発行されたアクセストークンを不正に用いた成りすまし攻撃のリスクがある。
      | セキュリティ上のリスクがあるため、応答性、効率性が求められるパブリックなクライアントでのみ使用すること。
  * - | リソースオーナパスワードクレデンシャルグラント
    - | リソースオーナパスワードクレデンシャルグラントのフローでは、クライアントがリソースオーナの認証情報を認可グラントとして使用して、直接アクセストークンを発行する。
      | クライアントへリソースオーナのクレデンシャルを共有する必要があるため、クライアントの信頼性が低い場合、クレデンシャルの不正利用や漏洩のリスクがある。
      | リソースオーナパスワードクレデンシャルグラントはリソースオーナとクライアントの間で高い信頼があり、かつ他のグラントタイプが利用できない場合にのみ使用すること。
  * - | クライアントクレデンシャルグラント
    - | クライアントクレデンシャルグラントのフローでは、クライアントの認証情報を認可グラントとして使用して、直接アクセストークンを発行する。
      | クライアントがリソースオーナであるような場合に使用する。

|

.. warning::

  \ :ref:`OAuth 2.0における認可グラント <OAuth2AuthorizationGrantTable>`\ で解説した通り、認可コードグラント以外のグラントタイプには、セキュリティ上のリスクや、使用上の制約がある。そのため、認可コードグラントの利用を優先して検討されたい。

|

認可コードグラント
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

認可コードグラントのフローを以下に示す。

.. figure:: ./images_OAuth2/OAuth2_AuthorizationCodeGrant.png
  :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: \ **認可コードグラントフロー**\
  :header-rows: 1
  :widths: 10 90
  :class: longtable

  * - 項番
    - 説明
  * - | (1)
    - | リソースオーナは、ユーザエージェント（Webブラウザなど）を介してクライアントが提供するリソースサーバの保護されたリソースにアクセスする。
      | クライアントはリソースオーナから認可の取得を行うために、リソースオーナが操作するユーザエージェントを認可サーバの認可エンドポイントにリダイレクトさせる。
      | このとき、クライアントは自身を識別するためのクライアントIDと、オプションとしてリソースに要求するスコープ、認可サーバが認可処理後にユーザエージェントを戻すリダイレクトURI、stateをリクエストパラメータに含める。
      | stateはユーザエージェントに紐付くランダムな値であり、一連のフローが同じユーザエージェントで実行されたことを保証するために利用される（CSRF対策）。
  * - | (2)
    - | ユーザエージェントは、クライアントに指示された認可サーバの認可エンドポイントにアクセスする。
      | 認可サーバはユーザエージェント経由でリソースオーナを認証し、リクエストパラメータのクライアントID、スコープ、リダイレクトURIを元に、自身に登録済みのクライアント情報と比較しパラメータの正当性確認を行う。
      | 確認完了後、アクセス要求の許可/拒否をリソースオーナに問い合わせる。
  * - | (3)
    - | リソースオーナはアクセス要求の許可/拒否を認可サーバに送信する。
      | リソースオーナがアクセスを許可した場合、認可サーバは、リクエストパラメータに含まれるリダイレクトURIを用いて、ユーザエージェントをクライアントにリダイレクトさせる指示を出す。
      | その際、認可コードをリダイレクトURIのリクエストパラメータとして付与する。
  * - | (4)
    - | ユーザエージェントは認可コードが付与されたリダイレクトURIにアクセスする。
      | クライアントの処理が完了するとリソースオーナにレスポンスを返却する。
  * - | (5)
    - | クライアントはアクセストークンを要求するために、認可コードを認可サーバのトークンエンドポイントに送信する。
      | 認可サーバのトークンエンドポイントはクライアントの認証と認可コードの正当性の検証を行い、正当である場合アクセストークンと任意でリフレッシュトークンを発行する。
      | リフレッシュトークンはアクセストークンが無効化された、または期限切れの際に新しいアクセストークンを発行するために使用される。

|

インプリシットグラント
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

インプリシットグラントのフローを以下に示す。

.. figure:: ./images_OAuth2/OAuth2_ImplicitGrant.png
  :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: \ **インプリシットグラントフロー**\
  :header-rows: 1
  :widths: 10 90
  :class: longtable

  * - 項番
    - 説明
  * - | (1)
    - | リソースオーナは、ユーザエージェントを介してクライアントが提供するリソースサーバの保護されたリソースが必要なページにアクセスする。
      | クライアントはリソースオーナから認可の取得とアクセストークンの発行を行うために、リソースオーナのユーザエージェントを認可サーバの認可エンドポイントにアクセスさせる。
      | このとき、クライアントは自身を識別するためのクライアントIDと、オプションとしてリソースに要求するスコープ、認可サーバが認可処理後にユーザエージェントを戻すリダイレクトURI、stateをリクエストパラメータに含める。
      | stateはユーザエージェントに紐付くランダムな値であり、一連のフローが同じユーザエージェントで実行されたことを保証するために利用される（CSRF対策）。
  * - | (2)
    - | ユーザエージェントは、クライアントに指示された認可サーバの認可エンドポイントにアクセスする。
      | 認可サーバはユーザエージェント経由でリソースオーナを認証し、リクエストパラメータのクライアントID、スコープ、リダイレクトURIを元に、自身に登録済みのクライアント情報と比較しパラメータの正当性確認を行う。
      | 確認完了後、アクセス要求の許可/拒否をリソースオーナに問い合わせる。
  * - | (3)
    - | リソースオーナはアクセス要求の許可/拒否を認可サーバに送信する。
      | リソースオーナがアクセスを許可した場合、認可サーバの認可エンドポイントはリクエストパラメータのリダイレクトURIを用いてユーザエージェントをクライアントリソースにリダイレクトさせる指示を出し、アクセストークンをリダイレクトURIのURLフラグメントに付与する。
      | ここで「クライアントリソース」とは、クライアントアプリケーションとは別にWebサーバ等にホストしておいた静的リソースを指す。
  * - | (4)
    - | ユーザエージェントはリダイレクトの指示に従い、クライアントリソースにリクエストを送信する。このとき、URLフラグメントの情報をローカルで保持し、リダイレクトの際にはURLフラグメントを送信しない。
      | クライアントリソースにアクセスすると、Webページ（通常は埋め込みスクリプトを含むHTMLドキュメント）が返却される。
      | ユーザエージェントはWebページに含まれるスクリプトを実行し、ローカルで保持していたURLフラグメントからアクセストークンを抽出する。
  * - | (5)
    - | ユーザエージェントはアクセストークンをクライアントに渡す。

|

リソースオーナパスワードクレデンシャルグラント
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

リソースオーナパスワードクレデンシャルグラントのフローを以下に示す。

.. figure:: ./images_OAuth2/OAuth2_ResourceOwnerPasswordCredentialsGrant.png
  :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: \ **リソースオーナパスワードクレデンシャルグラントフロー**\
  :header-rows: 1
  :widths: 10 90
  :class: longtable

  * - 項番
    - 説明
  * - | (1)
    - | リソースオーナがクライアントにクレデンシャル（ユーザ名、パスワード）を提供する。
  * - | (2)
    - | クライアントはアクセストークンを要求するために、認可サーバのトークンエンドポイントにアクセスする。
      | このとき、クライアントはリソースオーナから指定されたクレデンシャルとリソースに要求するスコープをリクエストパラメータに含める。
  * - | (3)
    - | 認可サーバのトークンエンドポイントはクライアントを認証し、リソースオーナのクレデンシャルを検証する。正当である場合アクセストークンを発行する。

|

クライアントクレデンシャルグラント
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

クライアントクレデンシャルグラントのフローを以下に示す。

.. figure:: ./images_OAuth2/OAuth2_ClientCredentialsGrant.png
  :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: \ **クライアントクレデンシャルグラントフロー**\
  :header-rows: 1
  :widths: 10 90
  :class: longtable

  * - 項番
    - 説明
  * - | (1)
    - | クライアントはアクセストークンを要求するために、認可サーバのトークンエンドポイントにアクセスする。
      | このとき、クライアントはクライアント自身のクレデンシャルを含めてアクセストークンを要求する。
  * - | (2)
    - | 認可サーバのトークンエンドポイントはクライアントを認証し、認証に成功した場合アクセストークンを発行する。

|

.. _OAuth2AccessTokenLifeCycle:

アクセストークンのライフサイクル
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
アクセストークンはクライアントが提示する認可グラントの正当性を認可サーバが確認することで発行される。発行されたアクセストークンは、認可サーバのポリシーまたはリソースオーナの指示に基づいたスコープが与えられ、保護されたリソースに対するアクセス権を保持する。アクセストークンは発行時に有効期限が設定され、有効期限切れとなると保護されたリソースに対するアクセス権を失効される。

アクセストークンの発行から失効までの流れは以下のようになる。

.. figure:: ./images_OAuth2/OAuth2_LifeCycleOfAccessToken.png
  :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: \ **アクセストークンの発行から失効までのフロー**\
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

| アクセストークンが有効期限切れとなった場合、クライアントがアクセストークンを再取得するためには認可サーバへ認可グラントの再提示を行い、認可サーバによる正当性の再確認が必要になる。そのため、アクセストークンの有効期限を短く設定した場合はユーザビリティが下がってしまう。一方で、アクセストークンの有効期限を長く設定した場合はアクセストークンの漏洩、漏洩時に悪用されるリスクが高まってしまう。

| ユーザビリティを下げずに漏洩、漏洩時のリスクを下げるためにはリフレッシュトークンが用いられる。リフレッシュトークンはアクセストークンが無効化されたあるいは期限切れの際、認可グラントの再提示を行うことなく新しいアクセストークンを取得するために利用される。リフレッシュトークンも発行時に有効期限が設定され、リフレッシュトークンが有効期限切れとなった場合はアクセストークンの再発行ができなくなる。
| アクセストークンの有効期限に短い期間を設定し、リフレッシュトークンの有効期限に長い期間を設定することで、短いサイクルでアクセストークンが再発行されユーザビリティを保ちつつアクセストークン漏洩及び漏洩時の悪用のリスクも抑えることができる。

| リフレッシュトークンの発行はオプションであり、認可サーバの判断に委ねられる。

リフレッシュトークンによるアクセストークンの再発行の流れは以下のようになる。

.. figure:: ./images_OAuth2/OAuth2_LifeCycleOfAccessTokenWithRefreshToken.png
  :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: \ **アクセストークンの発行から再発行までのフロー**\
  :header-rows: 1
  :widths: 10 90
  :class: longtable

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
    - | 認可サーバはクライアントが提示したリフレッシュトークンの正当性を検証し、正当であればアクセストークンとオプションでリフレッシュトークンを発行する。

|

リフレッシュトークンの有効期限が期限切れとなった場合は認可サーバへ認可グラントの再提示を行う。

.. figure:: ./images_OAuth2/OAuth2_LifeCycleOfRefreshToken.png
  :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: \ **リフレッシュトークンの発行から再発行までのフロー**\
  :header-rows: 1
  :widths: 10 90
  :class: longtable

  * - 項番
    - 説明
  * - | (1)
    - | クライアントが有効期限切れのアクセストークンを提示し、リソースサーバよりアクセストークンの有効期限切れエラーが返却された場合、クライアントはリフレッシュトークン（有効期限切れ）を提示することで新しいアクセストークンを要求する。
  * - | (2)
    - | 認可サーバはクライアントが提示したリフレッシュトークンの正当性を検証し、リフレッシュトークンの有効期限が切れている場合はエラーを返却する。
  * - | (3)
    - | 認可サーバよりリフレッシュトークンの有効期限切れエラーが返却された場合、クライアントは認可グラントを再提示し、アクセストークンを要求する。
  * - | (4)
    - | 認可サーバはクライアントが提示した認可グラントを確認し、アクセストークンとリフレッシュトークンを発行する。

|

.. _OAuth2SpringSecurityOAuth2Architecture:

Spring Securityより提供されるOAuth2のアーキテクチャ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Spring Security が提供するOAuth2は、OAuth 2.0で定義されているロールのうち、リソースサーバ、クライアントの2つのロールをSpringアプリケーションとして構築する際に必要となる機能を提供するライブラリである。
| Spring Security が提供するOAuth2は、Spring Framework（Spring MVC）やSpring Securityが提供する機能と連携して動作する仕組みになっており、Spring Security が提供するデフォルト実装を適切にコンフィギュレーション（Bean定義）するだけで、リソースサーバ、クライアントを構築することができる。また、Spring Security が提供するデフォルト実装で実現できない要件を組み込むことができるようになっている。

| なお、各ロール間のリクエストに対する認証・認可にはSpring Securityが提供する機能を利用するため、そちらの詳細は\ :doc:`../../Security/Authentication`\ 及び\ :doc:`../../Security/Authorization`\ を参照されたい。

|

Spring Security を使用してリソースサーバ、クライアントを構築した場合、以下のような流れで処理が行われる。

.. figure:: ./images_OAuth2/OAuth2_OAuth2Architecture.png
  :width: 80%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: \ **Spring Security OAuthのフロー**\
  :header-rows: 1
  :widths: 10 90
  :class: longtable

  * - 項番
    - 説明
  * - | (1)
    - | リソースオーナはユーザエージェントを介してクライアントへアクセスする。
  * - | (2)
    - | クライアントは\ ``OAuth2AuthorizedClientManager``\ を介して\ ``OAuth2AuthorizedClient``\を要求する。
  * - | (3)
    - | 要求された\ ``OAuth2AuthorizedClient``\ が認可サーバで許可されていない場合、ユーザエージェントへ認可サーバの認可エンドポイントへリダイレクトさせるよう指示する。
  * - | (4)
    - | ユーザエージェントは認可サーバの認可エンドポイントへリダイレクトする。
  * - | (5)
    - | 認可エンドポイントはリソースオーナへ認可を問い合わせる画面を表示した後に、リソースオーナからの認可リクエストを受け取り認可コードを発行する。
      | 発行した認可コードは、リダイレクトURIのリクエストパラメータとしてユーザエージェント経由でクライアントに渡される。
  * - | (6)
    - | クライアントは受け取った認可コードを内部のコンテキストに保持し、リクエストを再度処理する。
  * - | (7)
    - | 認可コードを取得している場合、認可コードを使用してトークンエンドポイントからアクセストークンを取得する。
      | 取得したアクセストークン、リフレッシュトークンをもとに\ ``OAuth2AuthorizedClient``\ を生成し、Spring Securityが管理するリポジトリに格納する。
      | 格納後にSpring Security Filterは元の処理へリダイレクトを行う。
  * - | (8)
    - | クライアントは\ ``OAuth2AuthorizedClientManager``\ を介して(7)で生成した\ ``OAuth2AuthorizedClient``\ を要求する。
      | この時、\ ``OAuth2AuthorizedClient``\ が認可された状態となっているため、認可サーバへのリダイレクトは実施しない。
  * - | (9)
    - | \ ``OAuth2AuthorizedClient``\ に設定されているアクセストークンの有効期限が切れている場合は、トークンエンドポイントに対しアクセストークンの再払い出しを要求する。
  * - | (10)
    - | クライアントは\ ``RestTemplate``\ のヘッダに(9)で取得したアクセストークンを設定し、リソースサーバにアクセスする。
  * - | (11)
    - | リソースサーバはアクセストークンを受け取ると、アクセストークンの検証を行う。
  * - | (12)
    - | リソースサーバはアクセストークンの検証に成功した場合、クライアントからのリクエストに応じたリソースを返却する。

|

.. _OAuth2AuthorizationServer:

認可サーバ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 認可サーバでは、リソースオーナの認証、リソースオーナからの認可の取得及びアクセストークンの発行を行う機能を提供する。
| 一部のグラントタイプでは、アクセストークンを発行するためにリソースオーナに認可を問い合わせる必要があるため、リソースオーナの認証と、リソースオーナからの認可の取得を行う機能についても提供している。

認可サーバはクライアント情報（どのクライアントに、どのリソースに対するどのスコープの認可を与えるかの情報）に基づいて、リソースにどのスコープでのアクセスを認可するか、アクセストークンを発行してよいかの検証を行う。

.. tip::

  Spring Securityでは\ `認可サーバをサポートしない旨 <https://spring.io/blog/2019/11/14/spring-security-oauth-2-0-roadmap-update>`_\ が報告されている。
    
  本ガイドラインでは、認可サーバについては詳細な説明を行わないため、要件に合う認可サーバの使用を検討されたい。

|

.. _OAuth2ResourceServer:

リソースサーバ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
リソースサーバでは、アクセストークン自体の妥当性とアクセストークンが保持するスコープ内のリソースへのアクセスであることを検証する機能を提供する。

Spring Securityでは、アクセストークンが付与されていないリクエストを受信した際に\ ``WWW-Authenticate``\ ヘッダーをクライアントへ送信し、アクセストークンでの認証が必要なことを伝える。アクセストークンが送信されると、Spring Securityはアクセストークンの検証を行い、問題がない場合にアプリケーションロジックを実行する。

|

.. _OAuth2NoBearerToken:

アクセストークン受信前
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| アクセストークン受信前のフローを以下に示す。
| クライアントの実装によっては、本フローは実施されず常にアクセストークンを設定した状態でリクエストしてくる可能性がある点に注意されたい。

.. figure:: ./images_OAuth2/OAuth2_BearerTokenAuthentication_NoAuth.png
  :width: 60%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: \ **アクセストークン受信前のフロー**\
  :header-rows: 1
  :widths: 10 90
  :class: longtable

  * - 項番
    - 説明
  * - | (1)
    - | クライアントからアクセストークンを付与していないリクエストを受信する。
  * - | (2)
    - | リソースサーバはアクセストークンが付与されていないリクエストを受信すると\ ``AuthorizationFilter``\ で\ ``AccessDeniedException``\ をスローしアクセスを拒否する。
  * - | (3)
    - | \ ``ExceptionTranslationFilter``\ でアクセスを拒否したことを検知し、クライアントへ\ ``WWW-Authenticate``\ ヘッダーを送信する。
      | クライアントは\ ``WWW-Authenticate``\ が返却されることでアクセストークンでの認証が必要である旨を検知する。

|

.. _OAuth2BearerToken:

アクセストークン受信後
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
アクセストークン受信後のフローを以下に示す。

.. figure:: ./images_OAuth2/OAuth2_BearerTokenAuthentication_Auth.png
  :width: 80%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: \ **アクセストークン受信後のフロー**\
  :header-rows: 1
  :widths: 10 90
  :class: longtable


  * - 項番
    - 説明
  * - | (1)
    - | クライアントからアクセストークンを付与したリクエストを受信する。
  * - | (2)
    - | \ ``HttpServletRequest``\ から\ ``BearerTokenAuthenticationToken``\ を抽出し、\ ``AuthenticationManager``\ へ値を渡す。
  * - | (3)
    - | \ ``AuthenticationManager``\ は受け取った\ ``BearerTokenAuthenticationToken``\ の検証を行う。
        
      .. note::
          
        \ ``AuthenticationManager``\ はリソースサーバの構成によって、JWT認証、Opaqueトークン認証のどちらかの処理が行われる。本ガイドラインではJWT認証を用いた認証方法で説明を行う。
             
        それぞれの認証方法の詳細は\ `JWT <https://docs.spring.io/spring-security/reference/6.0.1/servlet/oauth2/resource-server/jwt.html>`_\ 及び\ `Opaqueトークン <https://docs.spring.io/spring-security/reference/6.0.1/servlet/oauth2/resource-server/opaque-token.html>`_\ を参照されたい。
        
  * - | (4)
    - | (3)の検証結果より、クライアントから受け取ったアクセストークンでの認証が成功した場合、認証情報を\ ``SecurityContextHolder``\ に保存する。
  * - | (4)'
    - | (3)の検証結果より、クライアントから受け取ったアクセストークンでの認証が失敗した場合、\ ``SecurityContextHolder``\ の情報を削除し、クライアントへ\ ``WWW-Authenticate``\ ヘッダーを送信する。
  * - | (5)
    - | クライアントからリクエストされたスコープに基づいたリソースに対する処理を行う。
      
|

.. _OAuth2ClientServer:

クライアント
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
クライアントでは、リソースオーナからの認可を取得するために認可サーバへリダイレクトさせる機能と、認可サーバからアクセストークンを取得してリソースサーバへアクセスする機能を提供する。

Spring Securityは、アクセストークンを取得してリソースサーバへアクセスするために\ ``OAuth2AuthorizedClient``\ 、\ ``OAuth2AuthorizationRequestRedirectFilter``\ 、\ ``OAuth2AuthorizedClientRepository``\ 、\ ``OAuth2AuthorizedClientManager``\ 、\ ``OAuth2AuthorizedClientProvider``\ を提供している。

\ ``OAuth2AuthorizedClientManager``\ で\ ``OAuth2AuthorizedClient``\ を管理し、\ ``OAuth2AuthorizedClient``\ が認可前の場合に\ ``ClientAuthorizationRequiredException``\ を発生させ、\ ``OAuth2AuthorizationRequestRedirectFilter``\ で例外をハンドリングして認可サーバへリダイレクトさせることが可能となる。

また、\ ``OAuth2AuthorizedClientManager``\ では認可された\ ``OAuth2AuthorizedClient``\ を\ ``OAuth2AuthorizedClientRepository``\ で管理する機能を提供する。これにより、複数のリクエスト間でアクセストークンを共有することが可能となる。

|

クライアント認可処理
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
クライアントの認可処理のフローを以下に示す。

.. figure:: ./images_OAuth2/OAuth2_ClientArchitectureFirst.png
  :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: \ **クライアント認可処理のフロー**\
  :header-rows: 1
  :widths: 10 90
  :class: longtable

  * - 項番
    - 説明
  * - | (1)
    - | ユーザエージェントがクライアントのServiceの呼び出しが行われるよう、\ `Security Filter <https://docs.spring.io/spring-security/reference/6.0.1/servlet/architecture.html#servlet-security-filters>`_\ の処理を実施後にControllerへアクセスする。
  * - | (2)
    - | Serviceより\ ``OAuth2AuthorizedClientManager``\ を呼び出し、\ ``OAuth2AuthorizedClient``\ を要求する。
  * - | (3)
    - | \ ``OAuth2AuthorizedClient``\ が認可前のため、\ ``OAuth2AuthorizedClientProvider``\ は\ ``ClientAuthorizationRequiredException``\ を発生させ、\ ``OAuth2AuthorizationRequestRedirectFilter``\ でハンドリングさせることで、ユーザエージェントに対し認可サーバへのリダイレクトを促す。
      | この時、(1)で受け付けた\ ``HttpServletRequest``\ はHTTPセッションに保存される。
  * - | (4)
    - | ユーザエージェントから認可サーバの認可エンドポイントに対しリダイレクトする。
  * - | (5)
    - | 認可サーバで、リソースオーナーの認証、クライアントの認可を実施後、ユーザエージェントに対しクライアントへのリダイレクト処理を促す。その際に、認可コードをパラメータとして設定する。
  * - | (6)
    - | ユーザエージェントは認可処理後のリダイレクトURIに対しリダイレクトを行う。
  * - | (7)
    - | 認可サーバから払いだされた認可コードを使用し、認可サーバのトークンエンドポイントに対しアクセストークンを要求する。取得したアクセストークン及びリフレッシュトークンをもとに\ ``OAuth2AuthorizedClient``\ を生成する。
      | 生成した\ ``OAuth2AuthorizedClient``\ は\ ``OAuth2AuthorizedClientRepository``\ に格納される。
      | 格納後、(3)でHTTPセッションに保存した\ ``HttpServletRequest``\ を取り出し再処理を行う。
  * - | (8)
    - | Serviceより\ ``OAuth2AuthorizedClientManager``\ を呼び出し、(7)で生成した\ ``OAuth2AuthorizedClient``\ を要求する。
  * - | (9)
    - | (8)で取得した\ ``OAuth2AuthorizedClientManager``\ に設定されているアクセストークンをヘッダに設定し、リソースサーバへアクセスする。

|

クライアント認可後
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
クライアントが認可後のフローを以下に示す。

.. figure:: ./images_OAuth2/OAuth2_ClientArchitectureSecond.png
  :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: \ **クライアント認可後のフロー**\
  :header-rows: 1
  :widths: 10 90
  :class: longtable

  * - 項番
    - 説明
  * - | (1)
    - | ユーザエージェントがクライアントのServiceの呼び出しが行われるよう、\ `Security Filter <https://docs.spring.io/spring-security/reference/6.0.1/servlet/architecture.html#servlet-security-filters>`_\ の処理を実施後にControllerへアクセスする。
  * - | (2)
    - | Serviceより\ ``OAuth2AuthorizedClientManager``\ を呼び出し、\ ``OAuth2AuthorizedClient``\ を要求する。
  * - | (3)
    - | 認可済みの\ ``OAuth2AuthorizedClient``\ に格納されたアクセストークンを取得する。アクセストークンの有効期限が切れている場合、リフレッシュトークンを使用し、認可サーバのトークンエンドポイントに対しアクセストークンのリフレッシュを要求する。リフレッシュされたアクセストークン及びリフレッシュトークンを\ ``OAuth2AuthorizedClient``\ に設定する。
      | リフレッシュトークンの有効期限も切れている場合は\ ``OAuth2AuthorizationException``\ がスローされる。例外がスローされた場合、\ ``OAuth2AuthorizedClientRepository``\ に保存している\ ``OAuth2AuthorizedClient``\ は削除される。
  * - | (4)
    - | (3)で取得したアクセストークンをヘッダに設定し、リソースサーバへアクセスする。

|

.. _OAuth2HowToUse:

How to use
--------------------------------------------------------------------------------
Spring Security が提供するOAuth2を使用するために必要となるBean定義例や実装方法について説明する。

|

.. _OAuth2HowToUseConfiguration:

How to Useの構成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ :ref:`OAuth2AuthorizationGrant`\ で記載した通り、OAuth 2.0ではグラントタイプにより認可サーバ、クライアント間のフローが異なる。そのため、アプリケーションがサポートするグラントタイプに沿った実装を行う必要がある。

本ガイドラインでは、認可コードグラントについてリソースサーバ、クライアントの実装方法の解説を行う。

|

.. _OAuth2ImplementationAutorizationCodeGrant:

認可コードグラントの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
認可コードグラントを利用したリソースサーバ、クライアントの実装方法について説明する。

|

.. _OAuth2ImplementationOAuthResourceServerOfAutorizationCode:

リソースサーバの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
リソースサーバの実装方法について説明する。

リソースサーバでは、アクセストークンの検証とリソースに対しての認可制御をSpring Securityの機能を使用して提供する。

ここではTODOリソースのREST APIに対して認可制御を実現する方法を説明する。

|

依存ライブラリ設定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
アクセストークンによる認証を行えるようにするため、\ ``pom.xml``\ にライブラリを追加する。マルチプロジェクト構成の場合は、domainプロジェクトの\ ``pom.xml``\ に追加する。

* \ ``pom.xml``\

  .. code-block:: xml

    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-oauth2-resource-server</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-oauth2-jose</artifactId>
    </dependency>

  .. note::

    上記設定例の\ ``spring-security-oauth2-resource-server``\ と\ ``spring-security-oauth2-jose``\ は、依存ライブラリのバージョンを親プロジェクトである terasoluna-gfw-parent で管理する前提であるため、\ ``pom.xml``\ でのバージョンの指定は不要である。

|

設定ファイルの作成（リソースサーバ）
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
リソースサーバを実装する際には新たにOAuth 2.0用のBean定義ファイルを作成する。

ここでは \ ``oauth2-resource.xml``\ とする。

\ ``oauth2-resource.xml``\ には以下の設定を追加する。

* \ ``oauth2-resource.xml``\

  .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:sec="http://www.springframework.org/schema/security"
        xsi:schemaLocation="
            http://www.springframework.org/schema/security https://www.springframework.org/schema/security/spring-security.xsd
            http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        ">

        <sec:http pattern="/api/v1/todos/**">  <!-- (1) -->
            <!-- omitted -->
            <sec:oauth2-resource-server>
                <sec:jwt jwk-set-uri="https://idp.example.org/.well-known/jwks.json" /> <!-- (2) -->
            </sec:oauth2-resource-server>
        </sec:http>

    </beans>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``pattern``\ 属性には認可制御の対象とするパスのパターンを指定する。
    * - | (2)
      - | \ ``jwk-set-uri``\ を指定することでリソースサーバーは自動的にJWTエンコードされたアクセストークンを検証するように構成される。
        | 設定するJWK Set uriは使用する認可サーバのアーキテクチャ仕様を確認されたい。

|

作成した\ ``oauth2-resource.xml``\を読み込むように\ ``web.xml``\ に設定を追加する。

* \ ``web.xml``\

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
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``oauth2-resource.xml``\ で設定したパスのパターンを内包するようなパスが\ ``spring-security.xml``\ にアクセス制御対象として設定されている場合を考慮し、先に\ ``oauth2-resource.xml``\ を読み込むようにする。

|

リソースにアクセス可能なスコープの設定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

リソースごとにアクセス可能なスコープを定義するために、OAuth 2.0用のBean定義ファイルにスコープを追加する。

実装例は以下の通りである。

* \ ``oauth2-resource.xml``\

  .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:sec="http://www.springframework.org/schema/security"
        xsi:schemaLocation="
            http://www.springframework.org/schema/security https://www.springframework.org/schema/security/spring-security.xsd
            http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        ">

        <sec:http pattern="/api/v1/todos/**">
            <!-- omitted -->
            <sec:intercept-url pattern="/api/v1/todos/**" method="GET" access="hasAuthority('SCOPE_READ')" />  <!-- (1) -->
            <sec:intercept-url pattern="/api/v1/todos/**" method="POST" access="hasAuthority('SCOPE_CREATE')" />  <!-- (1) -->
            <sec:intercept-url pattern="/api/v1/todos/**" method="PUT" access="hasAuthority('SCOPE_UPDATE')" />  <!-- (1) -->
            <sec:intercept-url pattern="/api/v1/todos/**" method="DELETE" access="hasAuthority('SCOPE_DELETE')" />  <!-- (1) -->
            <sec:oauth2-resource-server>
                <sec:jwt jwk-set-uri="https://idp.example.org/.well-known/jwks.json" />
            </sec:oauth2-resource-server>
        </sec:http>

    </beans>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``intercept-url``\ を使用してリソースに対してスコープによるアクセスポリシーを定義する。
      
        * \ ``pattern``\ 属性には保護したいリソースのパスのパターンを指定する。本実装例では\ ``/api/v1/todos/``\ 配下のリソースが保護される。
        * \ ``method``\ 属性にはリソースのHTTPメソッドを指定する。
        * \ ``access``\ 属性にはリソースへのアクセスを認可するscopeを指定する。設定値は大文字、小文字を区別し、スコープの前に\ ``SCOPE_``\ を付ける。
 
        .. note::
          
          OAuth 2.0認可サーバーから発行されるJWTは\ ``scope``\ 属性または\ ``scp``\ 属性のいずれかを持ち、付与されたスコープや権限を示す。

          リソースサーバは、これらの属性で受け取った各スコープの前に自動的に\ ``SCOPE_``\ を付けるため、スコープによる制御を行うためには、\ ``SCOPE_``\ を付ける必要がある。

|

| また、Bean定義でスコープを指定せずに、以下の様にメソッドアノテーションを使用することでもスコープによる保護が可能となる。
| メソッドアノテーションの詳細については\ :ref:`AuthorizationToMethod`\ を参照されたい。

  .. code-block:: java

    @GetMapping
    @PreAuthorize("hasAuthority('SCOPE_READ')") // (1)
    @ResponseStatus(HttpStatus.OK)
    public List<TodoResource> getTodosWithPreAuthorize() {
        return getTodos();
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``PreAuthorize``\ を使用してリソースに対してスコープによるアクセスポリシーを定義する。

|

クライアントの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
クライアントの実装方法について説明する。

クライアントでは、グラントタイプやスコープなどのアプリケーション用件に沿ったパラメータを定義することで、OAuth 2.0機能を使用したリソースへのアクセスが可能となる。

|

依存ライブラリ設定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
Spring Security のOAuth2.0クライアント機能を使用するため、\ ``pom.xml``\ にライブラリを追加する。マルチプロジェクト構成の場合は、domainプロジェクトの\ ``pom.xml``\ に追加する。

* \ ``pom.xml``\

  .. code-block:: xml

    <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-oauth2-client</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-oauth2-jose</artifactId>
    </dependency>
    <dependency>
      <groupId>jakarta.servlet</groupId>
      <artifactId>jakarta.servlet-api</artifactId>
      <scope>provided</scope>
    </dependency>

  .. note::
  
    上記設定例は、依存ライブラリのバージョンを親プロジェクトである terasoluna-gfw-parent で管理する前提であるため、\ ``pom.xml``\ でのバージョンの指定は不要である。

|

設定ファイルの作成（クライアント）
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| クライアントを実装する際には新たにOAuth 2.0用のBean定義ファイルを作成する。
| ここでは\ ``oauth2-client.xml``\ とする。

| \ ``oauth2-client.xml``\ には以下の設定を追加する。

* \ ``oauth2-client.xml``\

  .. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:sec="http://www.springframework.org/schema/security"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/security https://www.springframework.org/schema/security/spring-security.xsd
            http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

        <sec:http pattern="/api/v1/todos/**">
          <!-- omitted -->
          <sec:oauth2-client /> <!-- (1) -->
        </sec:http>
    
    </beans>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | <sec:http>要素の子要素として<sec:oauth2-client>要素を指定する。
        | <sec:oauth2-client>要素を指定すると、OAuth 2.0 クライアント機能が適用される。

  .. note::

    本ガイドラインではOAuth 2.0 クライアント機能をデフォルトの状態で使用している。\ ``OAuth2AuthorizedClient``\ を管理するための\ ``ClientRegistrationRepository``\ 及び\ ``OAuth2AuthorizedClientRepository``\ が自動的にBean定義されるが、要件に応じ適切にカスタマイズされたい。
  
    カスタマイズ可能な構成オプションについては\ `OAuth 2.0 Client <https://docs.spring.io/spring-security/reference/6.0.1/servlet/oauth2/client/index.html>`_\ を参照されたい。

|

作成した\ ``oauth2-client.xml``\ を読み込むように\ ``web.xml``\ に設定を追加する。

* \ ``web.xml``\

  .. code-block:: xml

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            classpath*:META-INF/spring/applicationContext.xml
            classpath*:META-INF/spring/oauth2-client.xml <!-- (1) -->
            classpath*:META-INF/spring/spring-security.xml
        </param-value>
    </context-param>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``oauth2-client.xml``\ で設定したパスのパターンを内包するようなパスが\ ``spring-security.xml``\ にアクセス制御対象として設定されている場合を考慮し、先に\ ``oauth2-client.xml``\ を読み込むようにする。

|

例外ハンドリングの除外設定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| Spring Securityでは、リソースオーナによる認可が取得できていない状態でリソースサーバへのアクセスを試みた場合に\ ``ClientAuthorizationRequiredException``\ を発生させ、\ ``OAuth2AuthorizationRequestRedirectFilter``\ でハンドリングすることで認可サーバが提供するリソースオーナの認可を取得するためのページへリダイレクトさせている。
| \ ``OAuth2AuthorizationRequestRedirectFilter``\ はSecurity Filterとして提供されており、ブランクプロジェクトで予め設定している\ ``SystemExceptionResolver``\ が先に\ ``ClientAuthorizationRequiredException``\ をハンドリングしてしまうと期待した動作にならない。そのため、\ ``spring-mvc.xml``\ の設定を変更し、\ ``SystemExceptionResolver``\ が\ ``ClientAuthorizationRequiredException``\ をハンドリングしないようにする必要がある。
| \ ``SystemExceptionResolver``\ の詳しい解説については\ :doc:`../ArchitectureInDetail/WebApplicationDetail/ExceptionHandling`\ を参照されたい。

* \ ``spring-mvc.xml``\

  .. code-block:: xml

    <bean class="org.terasoluna.gfw.web.exception.SystemExceptionResolver">
      
      <!-- omitted -->
      
      <property name="excludedExceptions">
        <array>
          <!-- (1) -->
          <value>org.springframework.security.oauth2.client.ClientAuthorizationRequiredException</value>
        </array>
      </property>
    </bean>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``ClientAuthorizationRequiredException``\ を\ ``SystemExceptionResolver``\ のハンドリング対象から除外する。

|

クライアントの登録
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| \ ``ClientRegistration``\ を使用し、認可サーバに登録されたクライアントを定義する。
| クライアントの定義情報は\ ``ClientRegistrationRepository``\ に保持される。

* \ ``oauth2-client.xml``\

  .. code-block:: xml

    <sec:client-registrations>
        <sec:client-registration
            registration-id="okta-client-id"
            client-id="readClient"
            client-secret="okta-client-secret"
            authorization-grant-type="authorization_code"
            redirect-uri="{baseUrl}/authorized/okta"
            scope="READ,CREATE"
            provider-id="okta" /> <!-- (1)～(7) -->
        <sec:provider provider-id="okta"
            authorization-uri="https://dev-1234.oktapreview.com/oauth2/v1/authorize"
            token-uri="https://dev-1234.oktapreview.com/oauth2/v1/token"
            /> <!-- (8)～(9) -->
    </sec:client-registrations>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``registration-id``\ 属性に、ClientRegistrationを一意に識別するBean IDを設定する。
    * - | (2)
      - | \ ``client-id``\ 属性に、認可サーバに登録されたクライアントIDを設定する。
    * - | (3)
      - | \ ``client-secret``\ 属性に、認可サーバに登録されたクライアントの認可に用いるパスワードを設定する。
    * - | (4)
      - | \ ``authorization-grant-type``\ 属性に、グラントタイプを設定する。認可コードグラントの場合\ ``authorization_code``\ を指定する。
    * - | (5)
      - | \ ``redirect-uri``\ 属性に、認可サーバが認可コード発行後にユーザエージェントからクライアントへリダイレクトさせるためのリダイレクトURIを設定する。このURIへリダイレクト後、Spring Securityは\ ``ClientAuthorizationRequiredException``\ を発生させたリクエストの再処理を行う。リダイレクトURIは認可サーバに登録されているリダイレクトURIと一致させる必要がある。
        | 詳しくは\ `RFC6749#section-3.1.2 <https://datatracker.ietf.org/doc/html/rfc6749#section-3.1.2>`_\ を参照されたい。

        .. note::

          \ ``{baseUrl}``\ はURIテンプレート変数であり、\ ``{baseScheme}://{baseHost}{basePort}{basePath}``\ のことを指す。

          URIテンプレートを使用することで、展開時に\ ``X-Forwarded-*``\ ヘッダが使用される。これにより、User agentとクライアントがProxy Serverを経由して通信している場合であっても、クライアント側のURIを取得することが可能となる。

          詳しくは、\ `Initiating the Authorization Request <https://docs.spring.io/spring-security/reference/6.0.1/reactive/oauth2/client/authorization-grants.html#_initiating_the_authorization_request>`_\ を参照されたい。
            
        .. note::

          元のリクエストが再処理されるのは、CSRF対策が無効またはGETメソッドで処理された場合であり、CSRF対策が有効（ブランクのデフォルト設定）かつGETメソッド以外で処理された場合は、リダイレクトURIにリダイレクトされた後元のリクエストが再処理されずにリダイレクトURIでそのまま処理される。
            
          リダイレクトURIに対応したControllerを作成する場合、そのままでは元のリクエスト情報は取得できない。必要に応じて元のリクエストをセッションに保存する等の対処が必要があるため、注意すること。本ガイドラインでは、具体的な対処方法の説明については割愛する。
            
          CSRF対策を無効化する場合、URLパターンを設けるなどセキュリティリスクが小さくなるように実装すること。なお、元のリクエストパスにはGETメソッドでリダイレクトするため、ControllerはGETメソッドを許容するようにする必要がある。
            
          CSRF対策については\ :ref:`SpringSecurityCsrf`\ を参照されたい。
            
    * - | (6)
      - | \ ``scope``\属性には、認可リクエストとしてクライアントにリクエストするスコープをカンマ区切りで設定する。
      
        .. note::
          
          問い合わせの対象となるスコープは、認可サーバに事前に登録されているスコープと、クライアントが認可リクエスト時にリクエストパラメータで指定したスコープの積となる。
            
          例として、認可サーバでREADとCREATEとDELETEのスコープが割り当てられているクライアントに対してREADとCREATEのスコープをリクエストパラメータで指定した場合は、（READ,CREATE,DELETE）と（READ,CREATE）の積であるスコープ（READ,CREATE）が割り当てられる。
          
          認可サーバでクライアントに割り当てられていないスコープをリクエストパラメータで指定した場合はエラーとなり、アクセスが拒否される。
             
    * - | (7)
      - | \ ``provider-id``\ 属性には、エンドポイントを設定したプロバイダのIDを設定する。
    * - | (8)
      - | \ ``authorization-uri``\ 属性には、認可サーバーの認可エンドポイントURIを設定する。クライアントが認可前の場合は当認可エンドポイントURIに対しリクエストを送信する。
        | 認可エンドポイントURIは使用する認可サーバにより異なるため、認可サーバのアーキテクチャ仕様を確認されたい。
    * - | (9)
      - | \ ``token-uri``\ 属性には、認可サーバーのトークンエンドポイントURIを設定する。アクセストークンの取得やアクセストークンのリフレッシュを行う際に当トークンエンドポイントURIに対しリクエストを送信する。
        | トークンエンドポイントURIは使用する認可サーバにより異なるため、認可サーバのアーキテクチャ仕様を確認されたい。

|

OAuth2AuthorizedClientManagerの実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| \ ``OAuth2AuthorizedClient``\ を管理するための、\ ``OAuth2AuthorizedClientManager``\ をBean定義する。
| \ ``OAuth2AuthorizedClientManager``\ が管理する内容及び処理については\ `OAuth2AuthorizedClientManager/OAuth2AuthorizedClientProvider <https://docs.spring.io/spring-security/reference/6.0.1/servlet/oauth2/client/core.html#oauth2Client-authorized-manager-provider>`_\ を参照されたい。

以下にJavaConfigクラスを用いたBean定義の実装例を示す。

.. note::

  本ガイドラインのBean定義は基本的にXML-based configurationを用いているが、\ ``OAuth2AuthorizedClientManager``\ のBean定義に関してはJava-based configurationを用いている。
  
  \ ``OAuth2AuthorizedClientManager``\ に設定する\ ``OAuth2AuthorizedClientProvider``\ がBuilderパターンを使用していることに加え、Spring Securityが\ ``OAuth2AuthorizedClientManager``\ に対するXML DLSを提供していないため、\ `OAuth2AuthorizedClientManager/OAuth2AuthorizedClientProvider <https://docs.spring.io/spring-security/reference/6.0.1/servlet/oauth2/client/core.html#oauth2Client-authorized-manager-provider>`_\ の実装例に従いJava-based configurationで実装している。Java-based configurationで定義するクラスは、コンポーネントスキャンが有効となるパッケージ配下に配置されたい。詳しくは\ `Java-based configuration <https://docs.spring.io/spring-framework/docs/6.0.3/reference/html/core.html#beans-java>`_\ を参照されたい。

* \ ``SecurityConfig.java``\

  .. code-block:: java

    @Configuration
    public class SecurityConfig {

        // omitted

        @Bean
        public OAuth2AuthorizedClientManager authorizedClientManager(
                ClientRegistrationRepository clientRegistrationRepository,
                OAuth2AuthorizedClientRepository authorizedClientRepository) {
      
            // (1)
            OAuth2AuthorizedClientProvider authorizedClientProvider =
                OAuth2AuthorizedClientProviderBuilder
                    .builder()
                    .authorizationCode()
                    .refreshToken()
                    .build();
      
            // (2)
            DefaultOAuth2AuthorizedClientManager authorizedClientManager = new DefaultOAuth2AuthorizedClientManager(
                clientRegistrationRepository, authorizedClientRepository);
      
            // (3)
            authorizedClientManager.setAuthorizedClientProvider(authorizedClientProvider);

            return authorizedClientManager;
        }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | OAuth 2.0クライアントを認可（または再認可）するためのストラテジーを実装する。ここでは、認可コードグラントとリフレッシュトークンを設定している。
    * - | (2)
      - | \ ``HttpServletRequest``\ のコンテキスト内で操作するため、デフォルト実装である\ ``DefaultOAuth2AuthorizedClientManager``\ を実装する。
      
        .. note::
          
          \ ``HttpServletRequest``\ のコンテキスト外で操作する場合は、\ ``AuthorizedClientServiceOAuth2AuthorizedClientManager``\ を実装されたい。
      
    * - | (3)
      - | (2)で作成した\ ``OAuth2AuthorizedClientManager``\ に対し\ ``OAuth2AuthorizedClientProvider``\ を設定する。これにより、\ ``OAuth2AuthorizedClientManager``\ 経由でOAuth 2.0クライアントを認可することが可能となる。

|

.. _OAuth2GetAccessToken:

アクセストークンの取得
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

認可サーバへのアクセスは\ ``OAuth2AuthorizedClientManager``\ 及びSecurity Filterにより隠蔽されるため、認可サーバへのアクセスについて意識する必要はない。アクセストークンを取得するには\ ``OAuth2AuthorizedClientManager``\ で管理している\ ``OAuth2AuthorizedClient``\ を取得し、\ ``OAuth2AuthorizedClient``\ からアクセストークンを取得するだけでよい。

以下にServiceクラスの実装例を示す。

* \ ``OAuth2TokenServiceImpl.java``\

  .. code-block:: java

    @Service
    public class OAuth2TokenServiceImpl implements OAuth2TokenService {

        @Inject
        OAuth2AuthorizedClientManager authorizedClientManager;

        // omitted
    
        @Override
        public OAuth2AccessToken getToken(String registrationId) {
    
            Authentication authentication = SecurityContextHolder.getContext()
                    .getAuthentication();
    
            // (1)
            OAuth2AuthorizeRequest authorizeRequest = OAuth2AuthorizeRequest
                    .withClientRegistrationId(registrationId).principal(
                            authentication).build();
    
            OAuth2AuthorizedClient authorizedClient = null;
            try {
                // (2)
                authorizedClient = this.authorizedClientManager.authorize(
                        authorizeRequest);
            } catch (OAuth2AuthorizationException e) {
                if ("invalid_grant".equals(e.getError().getErrorCode())) {
                    // (3)
                    LOGGER.warn("refresh token expired. registrationId = {}",
                            registrationId);
    
                    throw new ClientAuthorizationRequiredException(registrationId);
                }
                throw e;
            }

            // (4)
            OAuth2AccessToken accessToken = authorizedClient.getAccessToken();
    
            return accessToken;
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
      - | \ ``registrationId``\ によって識別された\ ``OAuth2AuthorizedClient``\ を認可するためのリクエストを作成する。
    * - | (2)
      - | \ ``registrationId``\ によって識別された\ ``OAuth2AuthorizedClient``\ の認可（または再認可）を行う。
        | \ ``OAuth2AuthorizedClient``\ が認可されていない場合は、\ ``AuthorizationCodeOAuth2AuthorizedClientProvider``\ が\ ``ClientAuthorizationRequiredException``\ をスローし、Security Filterで捕捉することで認可処理を実行する。
        | また、\ ``OAuth2AuthorizedClient``\ が認可済みの状態でアクセストークンの有効期限が切れている場合は、リフレッシュトークンによりアクセストークンがリフレッシュされる。
      
        .. note::
          
          アクセストークンのリフレッシュが必要と判断される時間は、デフォルトの設定ではアクセストークンの有効期限切れの60秒前となっている。
      
    * - | (3)
      - | 有効期限切れのリフレッシュトークンをリクエストした場合は、\ ``OAuth2AuthorizationException``\ がスローされ例外コードに\ ``invalid_grant``\ が設定される。
        | ここでは、リフレッシュトークンの有効期限が切れていた場合に処理を継続させるため、（２）と同様に\ ``ClientAuthorizationRequiredException``\ をスローし\ ``OAuth2AuthorizedClient``\ の認可処理を行っている。当処理は一例であるため、有効期限が切れた際にそのままエラー画面に遷移させたい場合など、要件に合わせ適切にエラーハンドリングを実施されたい。
          
        .. note::
          
          \ ``invalid_grant``\ は、\ `RFC6749#section-5.2 <https://datatracker.ietf.org/doc/html/rfc6749#section-5.2>`_\ に記載されている通りリフレッシュトークンの有効期限切れ以外の場合にもスローされる。そのため、短時間で該当箇所のログが大量に出力されるような場合は不正なアクティビティの可能性がある点に注意されたい。
      
    * - | (4)
      - | 認可済みの\ ``OAuth2AuthorizedClient``\ からアクセストークンを取得する。

|

.. _OAuth2AccessTokenSettings:

アクセストークンの設定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| アクセストークンでの認証を行うためには\ ``Authorization: Bearer``\ ヘッダにアクセストークンを設定する必要がある。
| ここでは、RestTemplateにInterceptorでヘッダにアクセストークンを設定している。
| RestTemplateに関する詳細な説明は、\ :doc:`../ArchitectureInDetail/WebServiceDetail/RestClient`\ を参照されたい。

* \ ``RestTemplateAccessTokenInterceptor``\

  .. code:: java

    public class RestTemplateAccessTokenInterceptor implements
                                                    ClientHttpRequestInterceptor {  // (1)

        private String registrationId;

        @Inject
        OAuth2TokenService oAuth2TokenService;

        @Override
        public ClientHttpResponse intercept(HttpRequest request, byte[] body,
                ClientHttpRequestExecution execution) throws IOException {

            OAuth2AccessToken accessToken = this.oAuth2TokenService.getToken(
                    this.registrationId); // (2)
            request.getHeaders().setBearerAuth(accessToken.getTokenValue()); // (3)

            return execution.execute(request, body);
        }

        public void setRegistrationId(String registrationId) {
            this.registrationId = registrationId;
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
      - | \ ``ClientHttpRequestInterceptor``\ を実装し、ClientHttpRequestに対するInterceptorを作成する。
    * - | (2)
      - | アクセストークンを取得する。
        | アクセストークンの取得方法は\ :ref:`アクセストークンの取得 <OAuth2GetAccessToken>`\ を参照されたい。
    * - | (3)
      - | アクセストークンでの認証を行うために、\ ``Authorization: Bearer``\ ヘッダに(2)で取得したアクセストークンを設定する。
        | \ ``Authorization: Bearer``\ ヘッダについては\ `RFC6750 <https://datatracker.ietf.org/doc/html/rfc6750>`_\ を参照されたい。

|

| InterceptorをRestTemplateに設定する。

* \ ``applicationContext.xml``\

  .. code:: xml

    <bean id="restTemplate" class="org.springframework.web.client.RestTemplate">
        <!-- omitted -->
        <property name="interceptors">
            <!-- (1) -->
            <bean class="com.example.todo.app.interceptor.RestTemplateAccessTokenInterceptor">
                <property name="registrationId" value="${registrationId}" />
            </bean>
        </property>
    </bean>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``RestTemplateAccessTokenInterceptor``\ をInterceptorとして設定する。

|

リソースサーバへのアクセス
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
RestTemplateを用いてリソースサーバへアクセスする方法を説明する。

認可サーバへのアクセスは\ ``OAuth2AuthorizedClientManager``\ 及びSecurity Filterにより隠蔽されるため、開発の際には認可サーバを意識する必要がなく、リソースサーバが提供するREST APIに対して行う処理を、通常のREST APIへのアクセスと同様に記述する。

以下にServiceクラスの実装例を示す。

* \ ``TodoServiceImpl.java``\

  .. code-block:: java

    @Service
    @Transactional
    public class TodoServiceImpl implements TodoService {
    
        @Value("${resource.serverUrl}/api/v1/todos/")
        private String resourceServerUri;
    
        @Inject
        RestTemplate restTemplate; // (1)

        @Override
        @Transactional(readOnly = true)
        public Todo findOne(@NotEmpty String todoId) {
    
            // @formatter:off
            RequestEntity<Void> requestEntity = RequestEntity
                    .get(this.resourceServerUri + todoId) // (2)
                    .build();
            // @formatter:on
    
            ResponseEntity<Todo> responseEntity = getRestTemplate(registrationId)
                    .exchange(requestEntity, Todo.class); // (3)
            Todo todo = responseEntity.getBody();
    
            return todo;
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
      - | \ ``RestTemplate``\ をインジェクションする。
    * - | (2)
      - | ここでは\ ``GET``\ メソッドを指定している。リソースサーバが提供するREST APIに合わせ適切に設定されたい。
        | なお、アクセストークンの設定については、\ :ref:`アクセストークンの設定 <OAuth2AccessTokenSettings>`\ を参照されたい。
    * - | (3)
      - | URLで指定したリソースサーバに対しRESTでアクセスし、結果を\ ``Todo.class``\ として受け取る。

.. raw:: latex

   \newpage
