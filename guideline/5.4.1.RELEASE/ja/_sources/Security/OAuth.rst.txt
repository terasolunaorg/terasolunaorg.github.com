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

OAuth 2.0とは、サードパーティ製アプリケーションがHTTPサービスを利用する際に、
サーバ上の保護されたリソースに対するアクセス範囲の指定を可能にするための認可フレームワークのことである。

OAuth 2.0はRFCとして仕様化されており、関連する複数の技術仕様から構成されている。

以下にOAuth 2.0の主要な仕様を示す。


.. tabularcolumns:: |p{0.15\linewidth}|p{0.30\linewidth}|p{0.55\linewidth}|
.. list-table:: **OAuth 2.0の主要仕様**
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

従来のクライアントサーバ型の認証モデルでは、サードパーティ製アプリケーションはHTTPサービスの
保護されたリソースにアクセスするために、ユーザの認証情報（ユーザ名とパスワードなど）を利用して認証を行う。

つまり、ユーザは、サードパーティ製アプリケーションにリソースへのアクセス権を与えるために
自身の認証情報をサードパーティと共有する必要があるが、
これはサードパーティ製アプリケーションに不具合や悪意のある操作などが存在した場合に、
ユーザの意図しないアクセスや情報漏洩等のリスクにつながる。

.. figure:: ./images/OAuth_TraditionalAuthenticationModel.png
    :width: 100%


これに対し、OAuth 2.0ではHTTPサービスとの認証はユーザが直接行い、サードパーティ製アプリケーションには
「アクセストークン」と呼ばれる認証済みリクエストを行うための情報を払い出すことで、
サードパーティに認証情報を共有することなくリソースへアクセスすることが可能となる。

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

    ユーザエージェントは、リソースオーナが使用するWebブラウザ等を指す。
    本ガイドラインでは、エンドユーザの操作が発生する箇所を明確にするため、リソースオーナ（エンドユーザ）とユーザエージェントを別のものとして解説する。
    ガイドラインでリソースオーナと明示している場合に、エンドユーザの操作が発生する。


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
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | リソースオーナに対して認可を要求する。上の図ではクライアントがリソースオーナに
        | 直接要求を行っているが、認可サーバを経由して行うほうが望ましい。
        | 後述するグラントタイプの中では認可コードグラントとインプリシットグラントが
        | 認可サーバを経由してリソースオーナに要求を行うフローになっている。
    * - | (2)
      - | クライアントはリソースオーナからの認可を表すクレデンシャルとして認可グラント（後述）を受け取る。
    * - | (3)
      - | クライアントは、認可サーバに対して自身の認証情報とリソースオーナが与えた認可グラントを提示することで、アクセス
        | トークンを要求する。
    * - | (4)
      - | 認可サーバはクライアントを認証し、認可グラントの正当性を確認する。認可グラントが正当な場合、
        | アクセストークンを発行する。
    * - | (5)
      - | クライアントはリソースサーバの保護されたリソースへリクエストを行い、発行されたアクセス
        | トークンにより認証する。
    * - | (6)
      - | リソースサーバはアクセストークンの正当性を確認し、正当な場合、リクエストを受け入れリソースを応答する。

.. note::

    OAuth 1.0で不評だった署名とトークン交換の複雑な仕組みを簡略化するために、OAuth 2.0ではアクセストークンを扱うリクエストはHTTPS通信で行うことを必須としている。
    （HTTPS通信を使用することでアクセストークンの盗聴を防止する）

|

.. _AuthorizationGrant:

認可グラント
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
認可グラントは、リソースオーナからの認可を表し、クライアントがアクセストークンを取得する際に用いられる。
OAuth 2.0では、グラントタイプとして以下の4つを定義しているが、クレデンシャル項目を追加するなどの独自拡張を行うこともできる。

クライアントはいずれかのグラントタイプを利用して認可サーバへアクセストークンを要求し、取得したアクセストークンでリソースサーバにアクセスする。
認可サーバはサポートするグラントタイプを必ず1つ以上定義しており、その中から使用するグラントタイプをクライアントからの認可リクエストによって決定する。

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: **OAuth 2.0における認可グラント**
    :name: OAuthAuthorizationGrant　
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
        | しかし、アクセストークンがURL中にエンコードされるため、リソースオーナや同一デバイス上の他のアプリケーションに漏えいする可能性があるほか、
          クライアントの認証を行わないことから、他のクライアントに対して発行されたアクセストークンを不正に用いた成りすまし攻撃のリスクがある。
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

    \ :ref:`OAuth 2.0における認可グラント <OAuthAuthorizationGrant>`\で解説した通り、認可コードグラント以外のグラントタイプには、セキュリティ上のリスクや、使用上の制約がある。そのため、認可コードグラントの利用を優先して検討されたい。

|

認可コードグラント
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

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
      - | リソースオーナは、ユーザエージェント(Webブラウザなど)を介してクライアントが提供するリソースサーバの保護されたリソースにアクセスする。
        | クライアントはリソースオーナから認可の取得を行うために、リソースオーナが操作するユーザエージェントを認可サーバの認可エンドポイントにリダイレクトさせる。
        | このとき、クライアントは自身を識別するためのクライアントIDと、オプションとしてリソースに要求するスコープ、認可サーバが認可処理後にユーザエージェントを戻すリダイレクトURI、stateをリクエストパラメータに含める。
        | stateはユーザエージェントに紐付くランダムな値であり、一連のフローが同じユーザエージェントで実行されたことを保証するために利用される(CSRF対策)。
    * - | (2)
      - | ユーザエージェントは、クライアントに指示された認可サーバの認可エンドポイントにアクセスする。
        | 認可サーバはユーザエージェント経由でリソースオーナを認証し、リクエストパラメータのクライアントID、スコープ、リダイレクトURIを元に、自身に登録済みのクライアント情報と比較しパラメータの正当性確認を行う。
        | 確認完了後、アクセス要求の許可/拒否をリソースオーナにたずねる。
    * - | (3)
      - | リソースオーナはアクセス要求の許可/拒否を認可サーバに送信する。
        | リソースオーナがアクセスを許可した場合、認可サーバは、リクエストパラメータに含まれるリダイレクトURIを用いて、
          ユーザエージェントをクライアントにリダイレクトさせる指示を出す。
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
        | このとき、クライアントは自身を識別するためのクライアントIDと、オプションとしてリソースに要求するスコープ、認可サーバが認可処理後にユーザエージェントを戻すリダイレクトURI、stateをリクエストパラメータに含める。
        | stateはユーザエージェントに紐付くランダムな値であり、一連のフローが同じユーザエージェントで実行されたことを保証するために利用される(CSRF対策)。
    * - | (2)
      - | ユーザエージェントは、クライアントに指示された認可サーバの認可エンドポイントにアクセスする。
        | 認可サーバはユーザエージェント経由でリソースオーナを認証し、リクエストパラメータのクライアントID、スコープ、リダイレクトURIを元に、自身に登録済みのクライアント情報と比較しパラメータの正当性確認を行う。
        | 確認完了後、アクセス要求の許可/拒否をリソースオーナにたずねる。
    * - | (3)
      - | リソースオーナはアクセス要求の許可/拒否を認可サーバに送信する。
        | リソースオーナがアクセスを許可した場合、認可サーバの認可エンドポイントはリクエストパラメータのリダイレクトURIを用いて
          ユーザエージェントをクライアントリソースにリダイレクトさせる指示を出し、アクセストークンをリダイレクトURIのURLフラグメントに付与する。
        | ここで「クライアントリソース」とは、クライアントアプリケーションとは別にWebサーバ等にホストしておいた静的リソースを指す。
    * - | (4)
      - | ユーザエージェントはリダイレクトの指示に従い、クライアントリソースにリクエストを送信する。
          このとき、URLフラグメントの情報をローカルで保持し、リダイレクトの際にはURLフラグメントを送信しない。
        | クライアントリソースにアクセスすると、Webページ（通常は埋め込みスクリプトを含むHTMLドキュメント）が返却される。
        | ユーザエージェントはWebページに含まれるスクリプトを実行し、ローカルで保持していたURLフラグメントからアクセストークンを抽出する。
    * - | (5)
      - | ユーザエージェントはアクセストークンをクライアントに渡す。

|

リソースオーナパスワードクレデンシャルグラント
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

リソースオーナパスワードクレデンシャルグラントのフローを以下に示す。

.. figure:: ./images/OAuth_ResourceOwnerPasswordCredentialsGrant.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **リソースオーナパスワードクレデンシャルグラントフロー**
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

.. figure:: ./images/OAuth_ClientCredentialsGrant.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **クライアントクレデンシャルグラントフロー**
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

.. _AccessTokenLifeCycle:

アクセストークンのライフサイクル
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
アクセストークンはクライアントが提示する認可グラントの正当性を認可サーバが確認することで発行される。
発行されたアクセストークンは、認可サーバのポリシーまたはリソースオーナの指示に基づいたスコープが与えられ、保護されたリソースに対するアクセス権を保持する。
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

| リフレッシュトークンの発行はオプションであり、認可サーバの判断に委ねられる。

リフレッシュトークンによるアクセストークンの再発行の流れは以下のようになる。

.. figure:: ./images/OAuth_LifeCycleOfAccessTokenWithRefreshToken.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **アクセストークンの発行から再発行までのフロー**
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
      - | リソースサーバよりアクセストークンの有効期限切れエラーが返却された場合、
          クライアントはリフレッシュトークン（有効期限切れ）を提示することで新しいアクセストークンを要求する。
    * - | (8)
      - | 認可サーバはクライアントが提示したリフレッシュトークンの正当性を検証し、正当であればアクセストークンとオプションでリフレッシュトークンを発行する。

|

リフレッシュトークンの有効期限が期限切れとなった場合は認可サーバへ認可グラントの再提示を行う。

.. figure:: ./images/OAuth_LifeCycleOfRefreshToken.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **リフレッシュトークンの発行から再発行までのフロー**
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | クライアントが有効期限切れのアクセストークンを提示し、リソースサーバよりアクセストークンの有効期限切れエラーが返却された場合、
        | クライアントはリフレッシュトークン（有効期限切れ）を提示することで新しいアクセストークンを要求する。
    * - | (2)
      - | 認可サーバはクライアントが提示したリフレッシュトークンの正当性を検証し、リフレッシュトークンの有効期限が切れている場合はエラーを返却する。
    * - | (3)
      - | 認可サーバよりリフレッシュトークンの有効期限切れエラーが返却された場合、クライアントは認可グラントを再提示し、アクセストークンを要求する。
    * - | (4)
      - | 認可サーバはクライアントが提示した認可グラントを確認し、アクセストークンとリフレッシュトークンを発行する。

|


.. _SpringSecurityOAuthArchitecture:

Spring Security OAuthのアーキテクチャ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Spring Security OAuthは、OAuth 2.0で定義されているロールのうち、認可サーバ、リソースサーバ、クライアントの3つのロールをSpringアプリケーションとして構築する際に必要となる機能を提供するライブラリである。
Spring Security OAuthは、Spring Framework(Spring MVC)やSpring Securityが提供する機能と連携して動作する仕組みになっており、Spring Security OAuthが提供するデフォルト実装を適切にコンフィギュレーション（Bean定義）するだけで、認可サーバ、リソースサーバ、クライアントを構築することができる。
また、Spring FrameworkやSpring Securityと同様に数多くの拡張ポイントが用意されており、Spring Security OAuthが提供するデフォルト実装で実現できない要件を組み込むことができるようになっている。

なお、各ロール間のリクエストに対する認証・認可にはSpring Securityが提供する機能を利用するため、そちらの詳細は\ :doc:`../../Security/Authentication`\ 及び \ :doc:`../../Security/Authorization`\ を参照されたい。

.. note::

    一般的に、OAuth 2.0では全てのアプリケーションを1つのプロバイダが提供するのではなく、プロバイダが認可サーバ、リソースサーバを提供し、それらと連携するクライアントのみを実装するようなケースも多くある。そういった場合に連携するアプリケーションがSpring Security OAuthを使用して実装されているとは限らない。
    実装方法の解説では、Spring Security OAuth以外のアーキテクチャで実装されたアプリケーションと連携する方法についても、適宜Note等で補足していく。

    連携するアプリケーションの仕様に応じて、適宜本ガイドラインで紹介していている実装方法をカスタマイズして利用されたい。

|

Spring Security OAuthを使用して認可サーバ、リソースサーバ、クライアントを構築した場合、以下のような流れで処理が行われる。

.. figure:: ./images/OAuth_OAuth2Architecture.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **Spring Security OAuthのフロー**
    :header-rows: 1
    :widths: 10 90
    :class: longtable


    * - 項番
      - 説明
    * - | (1)
      - | リソースオーナはユーザエージェントを介してクライアントへアクセスする。
        | クライアントはサービスより\ ``org.springframework.security.oauth2.client.OAuth2RestTemplate``\ の呼び出しを行う。
        | 認可エンドポイントにアクセスする認可グラントの場合、ユーザエージェントへ認可サーバの認可エンドポイントへリダイレクトさせるよう指示する。
    * - | (2)
      - | ユーザエージェントは認可サーバの認可エンドポイント(\ ``org.springframework.security.oauth2.provider.endpoint.AuthorizationEndpoint``\)へアクセスする。
        | 認可エンドポイントはリソースオーナへ認可を問い合わせる画面を表示した後に、リソースオーナからの
        | 認可リクエストを受け取り認可コードまたはアクセストークンを発行する。
        | 発行した認可コードまたはアクセストークンは、リダイレクトを使用してユーザエージェント経由でクライアントに渡される。
    * - | (3)
      - | クライアントは\ ``OAuth2RestTemplate``\ を使用して認可サーバのトークンエンドポイント(\ ``org.springframework.security.oauth2.provider.endpoint.TokenEndpoint``\ )へアクセスする。
        | トークンエンドポイントは\ ``org.springframework.security.oauth2.provider.token.AuthorizationServerTokenServices``\ を呼び出し、クライアントが提示した認可グラントの検証及びアクセストークンの発行を行う。
        | 発行したアクセストークンはクライアントへの応答として渡される。
    * - | (4)
      - | クライアントは \ ``OAuth2RestTemplate``\ を使用し、認可サーバから取得したアクセストークンをリクエストヘッダに設定してリソースサーバにアクセスする。
        | リソースサーバは\ ``org.springframework.security.oauth2.provider.authentication.OAuth2AuthenticationManager``\ を呼び出し、\ ``org.springframework.security.oauth2.provider.token.ResourceServerTokenServices``\ を介してアクセストークンとアクセストークンに紐づく認証情報の検証を行う。
        | アクセストークンの検証に成功した場合、クライアントからのリクエストに応じたリソースを返却する。


.. note::

    前述のとおり、OAuth 2.0では各エンドポイントにおいてHTTPS通信の使用を前提としているが、HTTPS通信を使用するのがSSLアクセラレータやWebサーバまでの場合や、
    ロードバランサを使用して複数のアプリケーションサーバに分散させる場合がある。
    リソースオーナによって認可された後にクライアントに認可コードまたは、アクセストークンを連携するためのリダイレクトURIを組み立てる際に、
    SSLアクセラレータやWebサーバ、ロードバランサを指し示すリダイレクトURIを組み立てる必要がある。

    Spring(Spring Security OAuth)では以下のいずれかのヘッダを使用してリダイレクト用のURLを組み立てる仕組みが提供されている。

    * Forwardedヘッダ
    * X-Forwarded-Hostヘッダ、X-Forwarded-Portヘッダ、X-Forwarded-Protoヘッダ

    SSLアクセラレータやWebサーバ、ロードバランサ側で上記ヘッダを付与するように設定し、Spring(Spring Security OAuth)が正しいリダイレクトURIを生成できるようにする必要がある。
    これを行わない場合、認可コードグラントやインプリシットグラントにおいて行うリクエストパラメータ（リダイレクトURI）の検証に失敗する可能性がある。

.. tip::

    Spring Security OAuthが提供するエンドポイントはSpring MVCの機能を拡張して実現している。Spring Security OAuthが提供するエンドポイントには\ ``@FrameworkEndpoint``\ アノテーションがクラスに設定されている。
    これは\ ``@Controller``\ アノテーションで開発者がコンポーネントとして登録したクラスと競合させないためである。
    また、\ ``@FrameworkEndpoint``\ アノテーションでコンポーネントとして登録されたエンドポイントは、\ ``RequestMappingHandlerMapping``\ の拡張クラスである\ ``org.springframework.security.oauth2.provider.endpoint.FrameworkEndpointHandlerMapping``\ がエンドポイントの\ ``@RequestMapping``\ アノテーションを読み取り、URLと合致する\ ``@FrameworkEndpoint``\ のメソッドを、ハンドラメソッドとして扱っている。

|

.. _AuthorizationServer:

認可サーバ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
認可サーバでは、リソースオーナの認証、リソースオーナからの認可の取得、およびアクセストークンの発行を行う機能を提供する。
一部のグラントタイプでは、アクセストークンを発行するためにリソースオーナに認可を問い合わせる必要があるため、
リソースオーナの認証と、リソースオーナからの認可の取得を行う機能についても提供している。

認可サーバはクライアント情報（どのクライアントに、どのリソースに対するどのスコープの認可を与えるかの情報）
に基づいて、リソースにどのスコープでのアクセスを認可するかや、アクセストークンを発行してよいかの検証を行う。

なお、クライアント情報は事前に認可サーバに登録しておく必要があるが、
OAuth 2.0の仕様では認可サーバを利用するクライアント情報の登録手順について定められておらず、
Spring Security OAuthにおいてもクライアント情報の登録手続きの機能は提供されていない。
そのため、アプリケーションでクライアント情報を登録するためのインターフェイスを提供したい場合には、独自に実装する必要がある。

クライアントの認証
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
後述する各エンドポイントでは、認可サーバが管理するクライアント情報に基づき、リクエストに含まれるクライアントの認証を行う機能を提供している。


リソースオーナの認証
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
後述するリソースオーナからの認可の取得を行う場合、認可サーバはリソースオーナの認証も行う必要がある。この機能はSpring Securityが提供している認証機能を使用して実現する。

詳細については :ref:`SpringSecurityAuthentication`\ を参照のこと。


リソースオーナからの認可の取得
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
Spring Security OAuthは、認可エンドポイント（\ ``AuthorizationEndpoint``\）をSpring MVCの機能を利用して提供している。

クライアントは認可リクエスト送信時に利用したいスコープを指定し、リソースオーナが指定されたスコープを認可するか、
認可サーバに事前に登録されている、クライアントに割り当てられたスコープと一致する場合に、認可サーバでクライアントに対してそのスコープを認可する。
クライアントに対して認可を行う際には :ref:`SpringSecurityAuthorization`\ の節で説明しているSpring Securityのロールによる認可も併用することができる。

以下に認可エンドポイントアクセス時のフローを示す。

.. figure:: ./images/OAuth_AutohrizationServerAuthArchitecture.png
    :width: 100%
.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **認可サーバの動き（認可エンドポイントアクセス時）**
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | クライアントからの指示により、ユーザエージェントが認可サーバの認可エンドポイント(\ ``/oauth/authorize``\ )に
          アクセスすることで認可サーバの処理が実行される。
    * - | (2)
      - | アクセス先のURL(\ ``/oauth/authorize``\ )を処理する\ ``AuthorizationEndpoint``\ では、
          \ ``org.springframework.security.oauth2.provider.ClientDetailsService``\ のメソッドを呼び出し、事前に登録されているクライアント情報を取得後、リクエストパラメータを検証する。
    * - | (3)
      - | \ ``org.springframework.security.oauth2.provider.approval.UserApprovalHandler``\ のメソッドを呼び出し、クライアントへスコープに対する認可が登録されているかチェックする。
    * - | (4)
      - | 認可が未登録である場合、\ ``UserApprovalHandler``\ よりリソースオーナに認可を問い合わせる画面(認可画面)の要素を取得し、
          認可画面を表示させるURL(\ ``/oauth/confirm_access``\ )へフォワードさせる。
        | このとき、問い合わせの対象となるスコープはリクエストパラメータと事前に登録されているクライント情報の積をとり、
          Spring MVCの\ ``@SessionAttributes``\ を利用して連携される。
    * - | (5)
      - | フォワード先のURL(\ ``/oauth/confirm_access``\ )を処理するコントローラでは、
          ユーザエージェント経由でリソースオーナに認可画面を表示させる。
    * - | (6)
      - | リソースオーナの画面操作により認可が行われた場合、再度認可エンドポイント(\ ``/oauth/authorize``\ )にアクセスが行われる。
        | このとき、リクエストパラメータとして\ ``user_oauth_approval``\ が付与される。
    * - | (7)
      - | \ ``AuthorizationEndpoint``\ では\ ``user_oauth_approval``\ パラメータを付与しアクセスされた場合、\ ``UserApprovalHandler``\ の
          メソッドを呼び出し認可情報を登録する。
    * - | (8)
      - | \ ``UserApprovalHandler``\ の実装である\ ``org.springframework.security.oauth2.provider.approval.ApprovalStoreUserApprovalHandler``\ では\ ``org.springframework.security.oauth2.provider.approval.ApprovalStore``\により認可の状態を管理する。
        | リソースオーナにより認可が行われた場合、\ ``ApprovalStore``\ のメソッドを呼び出し、指定された情報を登録する。

|

.. note::

    問い合わせの対象となるスコープは前述のとおり、認可サーバに事前に登録されているスコープと、クライアントが認可リクエスト時にリクエストパラメータで指定したスコープの積となる。
    例として、認可サーバでREADとCREATEとDELETEのスコープが割り当てられているクライアントに対して、READとCREATEのスコープをリクエストパラメータで指定した場合は(READ,CREATE,DELETE)と(READ,CREATE)の積である、スコープREAD,CREATEが割り当てられる。
    認可サーバでクライアントに割り当てられていないスコープをリクエストパラメータで指定した場合はエラーとなり、アクセスが拒否される。

|

.. _DefinitionOfBadClientError:

不正クライアントエラー
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
RFC 6749の\ `4.1.2.1. Error Response <https://tools.ietf.org/html/rfc6749#section-4.1.2.1>`_\ にあるとおり、
リクエストパラメータの検証でリダイレクトURIもしくはクライアントIDの正当性が確認できない場合、不正なクライアントであるとみなす。
この場合、ユーザエージェントを不正なクライアントへリダイレクトさせず、ユーザエージェントを認可サーバが提供するエラー画面へと遷移させることで、リソースオーナに不正なクライアントであることを通知する。
本ガイドラインでは、上記エラーを\ **不正クライアントエラー**\ と呼ぶこととする。

認可エンドポイントで不正クライアントエラーが発生した場合、クライアントにエラー通知を行わず認可サーバのエラー画面へ遷移する。
認可エンドポイントで不正クライアントエラー以外のエラーが発生した場合、認可サーバからクライアントへのリダイレクトによりエラー通知が行われる。
認可エンドポイントでエラーが発生した場合のフローを以下に示す。

.. figure:: ./images/OAuth_AuthorizationEndpointErrorHandling.png
    :width: 100%

.. raw:: latex

   \newpage

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **認可サーバの動き（不正クライアントエラーのエラーハンドリング）**
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | A-1
      - | 不正クライアントエラーが発生した場合、エラー画面を表示させるURL(\ ``/oauth/error``\ )へフォワードさせる。
    * - | A-2
      - | フォワード先のURL(\ ``/oauth/error``\ )を処理するコントローラでは、 ユーザエージェント経由でリソースオーナにエラー画面を表示させる。

|

.. raw:: latex

   \newpage

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **認可サーバの動き（不正クライアントエラー以外のエラーハンドリング）**
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | B-1
      - | 不正クライアントエラー以外のエラーが発生した場合、認可サーバからクライアントへのリダイレクトによりエラー通知が行われる。
        | リダイレクトURIにリクエストパラメータとしてエラー情報が付与される。
    * - | B-2
      - | \ ``OAuth2RestTemplate``\ を経由して\ ``org.springframework.security.oauth2.client.token.AccessTokenProviderChain``\ にエラー情報が渡される。
    * - | B-3
      - | 渡されたエラー情報を例外に復元して、例外としてスローする。

|

.. _IssueOfAccessToken:

アクセストークンの発行
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
Spring Security OAuthは、トークンエンドポイント（\ ``TokenEndpoint``\）をSpring MVCの機能を利用して提供している。

トークンエンドポイントはアクセストークンの発行を\ ``AuthorizationServerTokenServices``\ のデフォルト実装である\ ``org.springframework.security.oauth2.provider.token.DefaultTokenServices``\ によって行っている。
アクセストークンの発行の際には、\ ``ClientDetailsService``\を介して認可サーバに登録済みのクライアント情報を取得し、アクセストークン発行の可否の検証に使用している。

以下にトークンエンドポイントアクセス時のフローを示す。

.. figure:: ./images/OAuth_AutohrizationServerTokenArchitecture.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **認可サーバの動き（トークンエンドポイントアクセス時）**
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | クライアントが認可サーバのトークンエンドポイント(\ ``/oauth/token``\)にアクセスすることでトークンエンドポイントの処理が実行される。
    * - | (2)
      - | \ ``ClientDetailsService``\ のメソッドを呼び出し、事前に登録されているクライアント情報を取得後、リクエストパラメータのスコープがクライアントに登録済みのものかチェックする。
    * - | (3)
      - | スコープが登録済みのものであった場合、\ ``org.springframework.security.oauth2.provider.TokenGranter``\ のメソッドを呼び出し、アクセストークンを発行する。
    * - | (4)
      - | \ ``TokenGranter``\ の実装である\ ``org.springframework.security.oauth2.provider.token.AbstractTokenGranter``\ では\ ``AuthorizationServerTokenServices``\ のアクセストークンを発行するメソッドを呼び出す。
        | \ ``AbstractTokenGranter``\ はグラントタイプ別に実装されている\ ``TokenGranter``\ の基底クラスであり、実際の処理は各クラスに委譲される。
    * - | (5)
      - | \ ``AuthorizationServerTokenServices``\ の実装である\ ``DefaultTokenServices``\ で\ ``ClientDetailsService``\ のメソッドを呼び出し、発行するアクセストークンに設定する有効期限、リフレッシュトークン発行の有無、有効な場合は有効期限を取得する。
    * - | (6)
      - | \ ``DefaultTokenServices``\ で\ ``ClientDetailsService``\ から取得した情報を基にアクセストークンを発行する。
        | 発行したアクセストークンは、アクセストークンを管理する\ ``org.springframework.security.oauth2.provider.token.TokenStore``\ のメソッドにて登録される。

|

.. _ResourceServer:

リソースサーバ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
リソースサーバでは、アクセストークン自体の妥当性とアクセストークンが保持するスコープ内のリソースへのアクセスであることを検証する機能を提供する。

Spring Security OAuthでは、アクセストークンの検証機能を、Spring Securityの認証・認可の枠組みを利用して実現している。
具体的には、Authentication FilterにSpring Security OAuthが提供する\ ``org.springframework.security.oauth2.provider.authentication.OAuth2AuthenticationProcessingFilter``\ を、
認証処理に\ ``AuthenticationManager``\ の実装クラスである\ ``OAuth2AuthenticationManager``\ を、
認証エラーの応答に\ ``AuthenticationEntryPoint``\ の実装クラスである\ ``org.springframework.security.oauth2.provider.error.OAuth2AuthenticationEntryPoint``\ をそれぞれ使用している。

Spring Securityの詳細については :ref:`SpringSecurityAuthentication`\ を参照のこと。

以下にリソースサーバの構成を示す

.. figure:: ./images/OAuth_ResourceServerArchitecture.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **リソースサーバの動き**
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | 初めにクライアントがリソースサーバにアクセスすると\ ``OAuth2AuthenticationProcessingFilter``\ の呼び出しが行われる。
        | \ ``OAuth2AuthenticationProcessingFilter``\ではリクエストよりアクセストークンを抽出する。
    * - | (2)
      - | アクセストークンを抽出後、\ ``AuthenticationManager``\の実装である\ ``OAuth2AuthenticationManager``\ において\ ``ResourceServerTokenServices``\ の
          メソッドを呼び出しアクセストークンに紐づく認証情報を取得する。また、認証情報の取得時にアクセストークンを検証する。
        | アクセストークンに紐づく認証情報の取得方法は、認可サーバに対してHTTPによる問い合わせを行うほか、認可サーバと\ ``TokenStore``\ を共用して取得を行うなどの方法がある。
        | どのようにして認証情報の取得を行うかについては\ ``ResourceServerTokenServices``\ の実装クラスに依存する。
    * - | (2')
      - | \ ``OAuth2AuthenticationProcessingFilter``\にて認証エラーが発生した場合、\ ``AuthenticationEntryPoint``\ の実装である\ ``OAuth2AuthenticationEntryPoint``\ に処理を委譲し、エラー応答を行う。
    * - | (3)
      - | \ ``OAuth2AuthenticationProcessingFilter``\ の処理が完了した場合、次のSecurity Filter(\ ``ExceptionTranslationFilter``\ )の呼び出しが行われる。
        | \ ``ExceptionTranslationFilter``\ の詳細については \ :ref:`AuthorizationErrorResponse`\ を参照のこと。
    * - | (3')
      - | \ ``ExceptionTranslationFilter``\ にて例外をキャッチした場合、\ ``AccessDeniedHandler``\ の実装である、\ ``org.springframework.security.oauth2.provider.error.OAuth2AccessDeniedHandler``\ に処理を委譲し、エラー応答を行う。
    * - | (4)
      - | リクエストの認証・認可の検証に成功した場合、クライアントからのリクエストに応じたリソースを返却する。


|

.. _Client:

クライアント
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
クライアントでは、リソースオーナからの認可を取得するために認可サーバへ誘導（リダイレクト）する機能と、アクセストークンを取得してリソースサーバへアクセスする機能を提供する。

Spring Security OAuthは、アクセストークンを取得してリソースサーバへアクセスするために利用する\ ``OAuth2RestTemplate``\ や\ ``org.springframework.security.oauth2.client.filter.OAuth2ClientContextFilter``\ を提供している。

\ ``OAuth2RestTemplate``\ は\ ``RestTemplate``\ を拡張しOAuth 2.0向けの機能を追加したクラスで、リフレッシュトークンを使用してアクセストークンを再取得する仕組みや、アクセストークンを取得する際にリソースオーナから認可が必要な場合に例外（\ ``org.springframework.security.oauth2.client.resource.UserRedirectRequiredException``\ ）をスローする仕組みを備えている。
なお、\ ``OAuth2ClientContextFilter``\ をサーブレットフィルタに登録することで、\ ``OAuth2RestTemplate``\ で発生した\ ``UserRedirectRequiredException``\ をハンドリングして認可サーバへ誘導（リダイレクト）することが可能となる。

また、\ ``OAuth2RestTemplate``\ では\ ``org.springframework.security.oauth2.client.resource.OAuth2ProtectedResourceDetails``\ にて指定されたグラントタイプに沿って認可サーバより取得したアクセストークンを\ ``org.springframework.security.oauth2.client.OAuth2ClientContext``\ の実装である\ ``org.springframework.security.oauth2.client.DefaultOAuth2ClientContext``\ に保持する。
\ ``DefaultOAuth2ClientContext``\ はデフォルトではセッションスコープのBeanとして定義され、複数のリクエスト間でアクセストークンを共有をすることが可能となる。

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
      - | ユーザエージェントがクライアントの\ ``Service``\ の呼び出しが行われるよう\ ``Controller``\ へアクセスする。
        | リソースサーバへのアクセスを伴うアクセスに対しては、(5)で発生する可能性がある\ ``UserRedirectRequiredException``\ を捕捉するためのサーブレットフィルタ（\ ``OAuth2ClientContextFilter``\ ）を適用する。
        | このサーブレットフィルタを適用することで、\ ``UserRedirectRequiredException``\ が発生した際に、ユーザエージェントを認可サーバの認可エンドポイントへアクセスさせることができる。
    * - | (2)
      - | \ ``Service``\ より\ ``OAuth2RestTemplate``\ の呼び出しを行う。
    * - | (3)
      - | \ ``OAuth2ClientContext``\ に保持しているアクセストークンを取得する。
        | 有効なアクセストークンを取得できた場合は、(6)の処理に移る。
    * - | (4)
      - | 初回アクセス時などでアクセストークンを保持していなかった場合、または有効期限が超過していた場合、\ ``org.springframework.security.oauth2.client.token.AccessTokenProvider``\ を呼び出しアクセストークンの取得を行う。
    * - | (5)
      - | \ ``AccessTokenProvider``\ では、リソースの詳細情報として\ ``OAuth2ProtectedResourceDetails``\ に定義しているグラントタイプに応じてアクセストークンの取得を行う。
        | 認可コードグラント向けの実装である\ ``org.springframework.security.oauth2.client.token.grant.code.AuthorizationCodeAccessTokenProvider``\ では、認可コードの取得が完了していない場合、\ ``UserRedirectRequiredException``\ をスローする。
    * - | (6)
      - | (3)または(5)で取得したアクセストークンを指定して、リソースサーバへのアクセスを行う。
        | アクセス時にアクセストークンの有効期限切れ（\ ``org.springframework.security.oauth2.client.http.AccessTokenRequiredException``\ ）などの例外が発生した場合、保持しているアクセストークンを初期化した後、再度(4)以降の処理を行う。

|

.. note::

    インプリシットグラントは一般的にJavaScriptなどで実装されたクライアントで採用されるため、本ガイドラインでもJavaScriptを用いて実装を行う方法を紹介する。

|

.. _HowToUse:

How to use
--------------------------------------------------------------------------------

Spring Security OAuthを使用するために必要となるBean定義例や実装方法について説明する。

.. _OAuthHowToUseConfiguration:

How to Useの構成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

「\ :ref:`AuthorizationGrant`\」で示したとおり、OAuth 2.0ではグラントタイプにより認可サーバ、クライアント間のフローが異なる。
そのため、アプリケーションがサポートするグラントタイプに沿った実装を行う必要がある。

本ガイドラインでは、グラントタイプごとに認可サーバ、リソースサーバ、クライアントの実装方法の解説を行う。

- *利用するグラントタイプに応じた読み進め方*
    グラントタイプごとの実装方法については、まずは認可コードグラントで一通りの実装方法を解説し、その他のグラントタイプでは認可コードグラントからの変更点を解説する形式としている。
    **どのグラントタイプを利用する場合も、必ず認可コードグラントでの解説を一読した上で、利用するグラントタイプの解説を読み進められたい。**
- *作成するアプリケーションに応じた読み進め方*
    認可サーバ、リソースサーバとクライアントの実装は独立しており、すべてのアプリケーションの実装方法を理解する必要はない。
    **いずれかのみ実装したい場合は、実装したいアプリケーションの解説のみ読み進められたい。**

|

.. _OAuthSetup:

Spring Security OAuthのセットアップ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Security OAuthが提供している機能を使用するために、Spring Security OAuthを依存ライブラリとして追加する。

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
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | Spring Security OAuthを使用するプロジェクトの :file:`pom.xml` に、Spring Security OAuthを依存ライブラリとして追加する。
        | リソースサーバ、認可サーバ、クライアントを別プロジェクトとして作成する場合、それぞれについて記述を追加すること。

.. note::

    上記設定例は、依存ライブラリのバージョンを親プロジェクトである terasoluna-gfw-parent で管理する前提であるため、pom.xmlでのバージョンの指定は不要である。
    上記の依存ライブラリはterasoluna-gfw-parentが利用している\ `Spring IO Platform <http://platform.spring.io/platform/>`_\ で定義済みである。

|

.. _ImplementationAutorizationCodeGrant:

認可コードグラントの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

認可コードグラントを利用した認可サーバ、リソースサーバ、クライアントの実装方法について説明する。

.. _ImplementationOAuthAuthorizationServerOfAutorizationCode:

認可サーバの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

認可サーバの実装方法について説明する。

認可サーバでは、「リソースオーナからの認可の取得」「アクセストークンの発行」を行うためのエンドポイントをSpring Security OAuthの機能を使用して提供する。
なお、上記のエンドポイントにアクセスする場合はリソースオーナまたはクライアントの認証が必要であり、本ガイドラインではSpring Securityの認証・認可の仕組みを使用して実現する。

.. _OAuthAuthorizationServerCreateSettingFile:

設定ファイルの作成（認可サーバ）
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

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
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ OAuth 2.0用のBean定義ファイルの指定を行う。\ ``oauth2-auth.xml``\ で設定したアクセス制御の対象のURLが\ ``spring-security.xml``\ で設定したアクセス制御の対象のURLに含まれる場合を考慮し、\ ``spring-security.xml``\より先に記述すること。

|

.. _OAuthAuthorizationServerDefinition:

認可サーバの定義
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
次に、認可サーバの定義を追加する。

* ``oauth2-auth.xml``

.. code-block:: xml

        <oauth2:authorization-server>  <!-- (1) -->
            <oauth2:authorization-code />  <!-- (2) -->
            <oauth2:refresh-token />  <!-- (3) -->
        </oauth2:authorization-server>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``<oauth2:authorization-server>``\ タグを使用し、認可サーバの定義を行う。
        | \ ``<oauth2:authorization-server>``\ タグを使用することで、認可を行うための認可エンドポイントと、
          アクセストークンを発行するためのトークンエンドポイントがコンポーネントとして登録される。
        |
        | 各コンポーネントにアクセスするため、以下のパスが設定される。

        * 認可エンドポイント(\ ``/oauth/authorize``\)
        * トークンエンドポイント(\ ``/oauth/token``\)
        * リソースオーナから認可を取得する際のフォワード先(\ ``/oauth/confirm_access``\)
        * 認可エンドポイントで不正クライアントエラーが発生した場合のフォワード先(\ ``/oauth/error``\)

    * - | (2)
      - | \ ``<oauth2:authorization-code />``\ タグを使用して、認可コードグラントをサポートする。
    * - | (3)
      - | \ ``<oauth2:refresh-token />``\ タグを使用して、リフレッシュトークンをサポートする。

.. warning::

    \ ``<oauth2:authorization-code />``\ タグと\ ``<oauth2:refresh-token />``\ タグは上記の順番で設定する必要がある。

    詳細は\ :ref:`OrderOfAuthoraizationServerSupportSetting`\を参照されたい。

|

.. note::

    \ ``<oauth2:authorization-server>``\ タグを使用することで登録されるエンドポイント、およびフォワード先のパスは
    それぞれカスタマイズすることが可能である。

    詳細は\ :ref:`CustomizeEndPoint`\、\ :ref:`CustomizeForward`\を参照されたい。

|

.. note::

    上記の設定ファイルは\ ``client-details-service-ref``\パラメータの設定を行っていないため、IDEによっては文法誤りによるエラーが検知されることがある。
    後述する設定を追加することでエラーは解消される。

|

.. note::

    認可コードは、認可コードが発行されてからアクセストークンの発行までの短い期間しか使われないため、デフォルトではインメモリで管理される。
    認可サーバが複数台構成の場合は、複数サーバ間で認可コードを共有するためにDBで管理する必要がある。
    認可コードをDBで管理する場合は、主キーとなる認可コードを保持するカラムと、認証情報を保持するカラムによって構成された以下のようなテーブルを作成する。以下の例はPostgreSQLを使用した場合のDB定義である。

    .. figure:: ./images/OAuth_ERDiagramCode.png
        :width: 30%

    認可サーバの設定ファイルには、\ ``<oauth2:authorization-code />``\ タグの\ ``authorization-code-services-ref``\ に、認可コードをDB管理する\ ``org.springframework.security.oauth2.provider.code.JdbcAuthorizationCodeServices``\ のBeanIDを指定する。
    \ ``JdbcAuthorizationCodeServices``\ のコンストラクタには、認可コード格納用のテーブルに接続するためのデータソースを指定する。
    認可コードをDBにて永続管理する場合の注意点については\ :ref:`OAuthAuthorizationServerHowToControllTarnsaction`\ を **必ず** 参照のこと。

    * ``oauth2-auth.xml``

    .. code-block:: xml

            <oauth2:authorization-server>
                <oauth2:authorization-code authorization-code-services-ref="authorizationCodeServices"/>
                <!-- omitted -->
            </oauth2:authorization-server>

            <bean id="authorizationCodeServices"
                  class="org.springframework.security.oauth2.provider.code.JdbcAuthorizationCodeServices">
                <constructor-arg ref="dataSource"/>
            </bean>

|

.. _OAuthAuthorizationServerClientAuthentication:

クライアントの認証
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
エンドポイントに対してアクセスしてきたクライアントについては、登録済みのクライアントか確認するために認証を行う必要がある。
クライアントの認証は、クライアントよりパラメータで渡されたクライアントIDとパスワードを、認可サーバで保持しているクライアント情報をもとに検証することで行う。認証にはBasic認証を用いて行う。

Spring Security OAuthではクライアント情報を取得するためのインタフェースである\ ``ClientDetailsService``\ の実装クラスを提供している。
また、クライアント情報を保持するためのクラスとして\ ``org.springframework.security.oauth2.provider.ClientDetails``\ インタフェースの実装クラスである\ ``org.springframework.security.oauth2.provider.client.BaseClientDetails``\ クラスを提供している。
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

        * client_id：クライアントを識別するIDであるクライアントIDを保持するカラム。
        * client_secret：クライアントの認証に使用するパスワードを保持するカラム。
        * client_name：クライアント名を保持する独自カラム。独自カラムであるため必須ではない。
        * access_token_validity：アクセストークンの有効期間[秒]を保持するカラム。
        * refresh_token_validity：リフレッシュトークンの有効期間[秒]を保持するカラム。

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
      - | リダイレクトURI情報を保持するテーブル。client_idを外部キーとし、クライアント情報と対応付けする。
        | web_server_redirect_uriカラムに、リソースオーナによる認可後にユーザエージェントをリダイレクトさせるURIを保持する。
        | 保持するリダイレクトURIは、クライアントが認可リクエスト時に申告するリダイレクトURIのホワイトリストチェックで使用される。
        | クライアントが申告する可能性のあるURIの数だけレコードを登録する。


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
    :class: longtable

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


.. note::

    クライアント情報を操作するためのRepositoryクラスを作成する必要があるが、ここでは説明を割愛する。
    具体的な実装方法については\ :ref:`repository-label`\ を参照されたい。

|

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
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | Serviceとしてcomponent-scanの対象とするため、クラスレベルに\ ``@Service``\ アノテーションをつける。
        | Bean名を\ ``clientDetailsService``\ として指定する。
    * - | (2)
      - | DBからクライアント情報を取得する。
    * - | (3)
      - | クライアント情報が見つからない場合は、Spring Security OAuthの例外である\ ``org.springframework.security.oauth2.provider.NoSuchClientException``\ を発生させる。
        | なお、認可エンドポイントでもクライアント情報の取得のため本処理が呼び出されるが、\ ``NoSuchClientException``\ が発生した場合は認可エンドポイントによってハンドリングされ、
        | \ ``org.springframework.security.oauth2.common.exceptions.OAuth2Exception``\ のサブクラスである\ ``org.springframework.security.oauth2.common.exceptions.BadClientCredentialsException``\がスローされる。
        | 認可エンドポイントで\ ``OAuth2Exception``\ が発生した場合のハンドリング方法については\ :ref:`OAuthAuthorizationServerHowToHandleError`\ を参照されたい。
    * - | (4)
      - | クライアントに紐付く情報を取得する。
        | 複数回にわけてRepositoryの呼び出しを行うことにより処理性能が落ちるような場合は(2)で一括取得する。
    * - | (5)
      - | 取得した各種情報を\ ``OAuthClientDetails``\ のフィールドに設定する。


\ ``oauth2-auth.xml``\ にクライアント認証に必要な設定を追記する。

* ``oauth2-auth.xml``

.. code-block:: xml

        <oauth2:authorization-server
             client-details-service-ref="clientDetailsService">  <!-- (1) -->
            <oauth2:authorization-code />
            <oauth2:refresh-token />
        </oauth2:authorization-server>

        <sec:http pattern="/oauth/*token*/**"
            authentication-manager-ref="clientAuthenticationManager">  <!-- (2) -->
            <sec:http-basic entry-point-ref="oauthAuthenticationEntryPoint" />           <!-- (3) -->
            <sec:csrf disabled="true"/>  <!-- (4) -->
            <sec:intercept-url pattern="/**" access="isAuthenticated()"/>  <!-- (5) -->
            <sec:access-denied-handler ref="oauth2AccessDeniedHandler"/>  <!-- (6) -->
        </sec:http>

        <sec:authentication-manager alias="clientAuthenticationManager">  <!-- (7) -->
            <sec:authentication-provider user-service-ref="clientDetailsUserService" >  <!-- (8) -->
                <sec:password-encoder ref="passwordEncoder"/>  <!-- (9) -->
            </sec:authentication-provider>
        </sec:authentication-manager>

        <bean id="oauthAuthenticationEntryPoint" class="org.springframework.security.oauth2.provider.error.OAuth2AuthenticationEntryPoint">
            <property name="typeName" value="Basic" />  <!-- (10) -->
            <property name="realmName" value="Realm" />  <!-- (11) -->
        </bean>

        <bean id="oauth2AccessDeniedHandler" class="org.springframework.security.oauth2.provider.error.OAuth2AccessDeniedHandler" />  <!-- (12) -->

        <bean id="clientDetailsUserService"
            class="org.springframework.security.oauth2.provider.client.ClientDetailsUserDetailsService">  <!-- (13) -->
            <constructor-arg ref="clientDetailsService" />  <!-- (14) -->
        </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``client-details-service-ref``\ 属性に\ ``OAuthClientDetailsService``\ のBeanを指定する。
        | 指定するBeanIDは、\ ``ClientDetailsService``\ の実装クラスで指定したBeanIDと合わせる必要がある。
    * - | (2)
      - | アクセストークン操作に関するエンドポイントへのセキュリティ設定を行うために、エンドポイントとして
          \ ``/oauth/*token*/``\ 配下をアクセス制御の対象として指定する。
          Spring Security OAuthにより定義されているエンドポイント、およびそのデフォルト値は以下のとおりである。

        * トークン払い出しに使用するトークンエンドポイント(\ ``/oauth/token``\)
        * トークンを検証するチェックトークンエンドポイント(\ ``/oauth/check_token``\)
        * JWTの署名を公開鍵暗号方式で作成した場合に、公開鍵を取得するために使用するエンドポイント(\ ``/oauth/token_key``\ )

        | \ ``authentication-manager-ref``\ 属性に(7)で定義しているクライアント認証用の\ ``AuthenticationManager``\のBeanを指定する。
    * - | (3)
      - | クライアント認証にBasic認証を適用する。
        | 詳細については\ `Basic and Digest Authentication <http://docs.spring.io/spring-security/site/docs/4.2.4.RELEASE/reference/html/basic.html>`_\を参照されたい。
    * - | (4)
      - | \ ``/oauth/*token*/**``\ へのアクセスに対してCSRF対策機能を無効化する。
        | Spring Security OAuthでは、OAuth 2.0のCSRF対策として推奨されている、stateパラメータを使用したリクエストの正当性確認を採用している。
    * - | (5)
      - | エンドポイント配下に対して、認証済みユーザのみがアクセスできる権限を付与する設定。
        | Webリソースに対してアクセスポリシーの指定方法については、\ :doc:`../../Security/Authorization`\ を参照されたい。
    * - | (6)
      - | \ ``access-denied-handler``\には\ ``OAuth2AccessDeniedHandler``\のBeanを設定する。ここでは(12)で定義している\ ``oauth2AccessDeniedHandler``\のBeanを指定する。
    * - | (7)
      - | クライアントを認証するための\ ``AuthenticationManager``\ をBean定義する。
        | リソースオーナの認証で使用する\ ``AuthenticationManager``\ と別名のBeanIDを指定する必要がある。
        | リソースオーナの認証については\ :ref:`OAuthAuthorizationServerResourceOwnerAuthentication`\を参照されたい。
    * - | (8)
      - | \ ``sec:authentication-provider``\ の\ ``user-service-ref``\ 属性に(13)で定義している\ ``org.springframework.security.oauth2.provider.client.ClientDetailsUserDetailsService``\のBeanを指定する。
    * - | (9)
      - | クライアントの認証に使用するパスワードのハッシュ化に使用する\ ``PasswordEncoder``\ のBeanを指定する。
        | パスワードハッシュ化の詳細については \ :ref:`SpringSecurityAuthenticationPasswordHashing`\ を参照されたい。
    * - | (10)
      - | クライアントの認証に失敗した場合に、レスポンスヘッダでクライアントに提示するクライアント認証の認証スキームを指定する。
    * - | (11)
      - | クライアントの認証に失敗した場合に、レスポンスヘッダでクライアントに提示するクライアント認証のレルムを指定する。
        | クライアント認証のレルムをカスタマイズしたい場合に設定する。
        | 指定しない場合はデフォルト値である\ ``oauth``\ が設定される。
    * - | (12)
      - | Spring Security OAuthが提供するOAuth 2.0用の\ ``AccessDeniedHandler``\を定義する。
        | \ ``OAuth2AccessDeniedHandler``\は、認可エラー時に発生する例外をハンドリングしてエラー応答を行う。
    * - | (13)
      - | \ ``UserDetailsService``\ インタフェースの実装クラスである\ ``ClientDetailsUserDetailsService``\ をBean定義する。
        | リソースオーナの認証で使用する\ ``UserDetailsService``\ と別名のBeanIDを指定する必要がある。
    * - | (14)
      - | コンストラクタの引数に、DBからクライアント情報を取得する\ ``OAuthClientDetailsService``\のBeanを指定する。
        | 指定するBeanIDは、\ ``ClientDetailsService``\ の実装クラスで指定したBeanIDと合わせる必要がある。


|

.. _OAuthAuthorizationServerResourceOwnerAuthentication:

リソースオーナの認証
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

アクセストークンの取得に認可コードグラントを用いる場合、ログイン画面を用意する等、なんらかの方法でリソースオーナを認証する必要がある。

| 本ガイドラインでは、リソースオーナの認証にSpring Securityを利用する前提とする。
| 認可の設定には、認証済みユーザのみ認可エンドポイントURLへアクセスできるよう、認可エンドポイントURLを含んだURLをアクセスポリシーとして定義する必要がある。
  また、リソースオーナから認可を取得する際のフォワード先と、認可エンドポイントで不正クライアントエラーが発生した場合のフォワード先も同様にアクセスポリシーとして定義する必要がある。
| リソースオーナから認可を取得する際のフォワードをハンドリングするコントローラについては \ :ref:`OAuthAuthorizationServerHowToCustomizeAuthorizeView`\ を、
  認可エンドポイントでの不正クライアントエラーをハンドリングするコントローラについては \ :ref:`OAuthAuthorizationServerHowToHandleError`\ を参照されたい。

Spring Securityの詳細については \ :doc:`../../Security/Authentication`\ 及び \ :doc:`../../Security/Authorization`\ を参照されたい。

以下に認可エンドポイント、リソースオーナから認可を取得する際のフォワード先、認可エンドポイントで例外が発生した場合のフォワード先を含んだアクセスポリシーの定義例を示す。

* ``spring-security.xml``

.. code-block:: xml

        <sec:http authentication-manager-ref="authenticationManager"> <!-- (1) -->
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
            <sec:intercept-url pattern="/oauth/**" access="isAuthenticated()" /> <!-- (2) -->
            <!-- omitted -->
        </sec:http>

         <sec:authentication-manager alias="authenticationManager"> <!-- (3) -->
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
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | 認可サーバにフォーム認証を適用し、\ ``authentication-manager-ref``\ 属性に(3)で定義している\ ``authenticationManager``\ を指定する。
        | \ ``oauth2-auth.xml``\ でも\ ``AuthenticationManager``\ を定義しているため、別名のBeanIDを指定する必要がある。
    * - | (2)
      - | 以下を含んだパス(\ ``/oauth/``\)配下を認証済みユーザのみがアクセスできるよう指定する。

        * 認可エンドポイント(\ ``/oauth/authorize``\)
        * リソースオーナから認可を取得する際のフォワード先(\ ``/oauth/confirm_access``\)
        * 認可エンドポイントで不正クライアントエラーが発生した場合のフォワード先(\ ``/oauth/error``\)

    * - | (3)
      - | リソースオーナを認証するための\ ``authenticationManager``\ をBean定義する。


.. _OAuthAuthorizationServerHowToAuthorizeByScope:

スコープごとの認可
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
リソースオーナからの認可の取得時に、要求されたスコープを一括で認可するのではなく、各スコープを個別に認可する場合の設定方法を説明する。

認可サーバを再起動した際に認可情報を失わないよう永続管理するために、また複数台の認可サーバで認可情報を共有するためには、認可情報をDBで管理する必要がある。
スコープごとに認可情報を格納するためのDBとして、以下のDBを作成する。以下の例ではPostgreSQLを使用した場合のDB定義を説明する。

.. figure:: ./images/OAuth_ERDiagramApprovals.png
    :width: 50%


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | 認可情報を保持するテーブル。userId、clientId、scopeを主キーとする。
        | 各カラムの役割は以下のとおりである。

        * userId：認可を行ったリソースオーナのユーザ名を保持するカラム。
        * clientId：リソースオーナによって認可されたクライアントのクライアントIDを保持するカラム。
        * scope：リソースオーナに認可されたスコープを保持するカラム。
        * status：リソースオーナに認可されたかどうかを保持するカラム。認可された場合は\ ``APPROVED``\ 、拒否された場合は\ ``DENIED``\ が設定される。
        * expiresAt：認可情報の有効期限を保持するカラム。
        * lastModifiedAt：認可情報が最後に更新された日時を保持するカラム。

リソースオーナからスコープ毎の認可を取得し、DBに保存して管理するための設定を行う。

実装例は以下のとおりである。

* ``oauth2-auth.xml``

.. code-block:: xml

        <oauth2:authorization-server
             client-details-service-ref="clientDetailsService"
             user-approval-handler-ref="userApprovalHandler"> <!-- (1) -->

             <!-- omitted -->

        </oauth2:authorization-server>

        <!-- omitted -->

        <bean id="userApprovalHandler"
              class="org.springframework.security.oauth2.provider.approval.ApprovalStoreUserApprovalHandler">  <!-- (2) -->
            <property name="clientDetailsService" ref="clientDetailsService"/>
            <property name="approvalStore" ref="approvalStore"/>
            <property name="requestFactory" ref="requestFactory"/>
            <property name="approvalExpiryInSeconds" value="3200" />
        </bean>

        <bean id="approvalStore"
              class="org.springframework.security.oauth2.provider.approval.JdbcApprovalStore">  <!-- (3) -->
            <constructor-arg ref="dataSource"/>
        </bean>

        <bean id="requestFactory"
              class="org.springframework.security.oauth2.provider.request.DefaultOAuth2RequestFactory">
            <constructor-arg ref="clientDetailsService"/>
        </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | スコープの認可処理を行う\ ``UserApprovalHandler``\ として、\ ``user-approval-handler-ref``\ に(2)で定義している\ ``ApprovalStoreUserApprovalHandler``\ のBeanを指定する。
    * - | (2)
      - | スコープの認可処理を行う\ ``ApprovalStoreUserApprovalHandler``\ をBean定義する。
        | リソースオーナの認可結果を管理する\ ``approvalStore``\ プロパティには、(3)で定義している\ ``org.springframework.security.oauth2.provider.approval.JdbcApprovalStore``\ のBeanを指定する。
        | スコープの認可処理に使用するクライアント情報の取得をする\ ``clientDetailsService``\ プロパティには、\ ``OAuthClientDetailsService``\ のBeanを指定する。
        | \ ``requestFactory``\ プロパティには、\ ``org.springframework.security.oauth2.provider.request.DefaultOAuth2RequestFactory``\ のBeanを指定する。
        | \ ``requestFactory``\ プロパティに設定したBeanは\ ``ApprovalStoreUserApprovalHandler``\ によって使用されないが、設定を行っていない場合は\ ``ApprovalStoreUserApprovalHandler``\ のBean生成時にエラーとなるため、\ ``requestFactory``\ プロパティへの設定が必要である。
        | 認可情報の有効期間[秒]を指定する場合は、\ ``approvalExpiryInSeconds``\ プロパティに、有効期間[秒]を設定する。設定を行わない場合は、認可情報は認可から一ヶ月間有効となる。
    * - | (3)
      - | 認可情報をDBで管理する\ ``JdbcApprovalStore``\ をBean定義する。
        | コンストラクタには、認可情報格納用のテーブルに接続するためのデータソースを指定する。
        | データソースの設定方法については、\ :ref:`data-access-common_howtouse_datasource`\ を参照されたい。

        .. warning::

            認可情報をDBにて永続管理する場合の注意点については\ :ref:`OAuthAuthorizationServerHowToControllTarnsaction`\ を **必ず** 参照のこと。

        .. note::

            認可情報を永続管理する必要がなく、DBではなくインメモリで管理したい場合は、\ ``approvalStore``\ として\ ``org.springframework.security.oauth2.provider.approval.InMemoryApprovalStore``\ をBean定義すればよい。


.. _OAuthAuthorizationServerHowToCustomizeAuthorizeView:

スコープ認可画面のカスタマイズ
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

スコープ認可画面をカスタマイズしたい場合、コントローラとJSPを作成することでカスタマイズできる。以下ではスコープ認可画面のカスタマイズした場合の例を説明する。

リソースオーナからの認可を取得するためにエンドポイントの呼び出しを行う場合、\ ``/oauth/confirm_access``\ にフォワードされる。
\ ``/oauth/confirm_access``\ をハンドリングするコントローラを作成する。

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
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@RequestMapping``\ アノテーションを使用して、\ ``/oauth/confirm_access``\ へのアクセスに対するメソッドとしてマッピングを行う。


次に、スコープ認可画面のJSPを作成する。
認可対象のスコープは\ ``scopes``\ キーとして、リクエストパラメータは\ ``authorizationRequest``\
としてModelに登録されているため、これを利用してスコープ認可画面を表示する。

* ``oauthConfirm.jsp``

.. code-block:: jsp

    <%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags" %>

    <body>
        <div id="wrapper">
            <h1>OAuth Approval</h1>
            <p>Do you authorize ${f:h(authorizationRequest.clientId)} to access your protected resources?</p>  <!-- (1) -->
            <form id="confirmationForm" name="confirmationForm" action="${pageContext.request.contextPath}/oauth/authorize" method="post">
                <c:forEach var="scope" items="${scopes}">  <!-- (2) -->
                    <li>
                        ${f:h(scope.key)}
                        <input type="radio" name="${f:h(scope.key)}" value="true"/>Approve
                        <input type="radio" name="${f:h(scope.key)}" value="false"/>Deny
                    </li>
                </c:forEach>
                <input name="user_oauth_approval" value="true" type="hidden"/>  <!-- (3) -->
                <sec:csrfInput />  <!-- (4) -->
                <label>
                    <input name="authorize" value="Authorize" type="submit"/>
                </label>
            </form>
        </div>
    </body>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1) 
      - | クライアントIDを画面に表示させる。
        | \ ``authorizationRequest``\ の型は\ ``org.springframework.security.oauth2.provider.AuthorizationRequest``\ であり、クライアントIDは\ ``authorizationRequest.clientId``\ と指定することで出力される。
    * - | (2)
      - | スコープ毎に認可可否を指定するためのラジオボタンを出力する。対象のスコープは\ ``scopes``\ に含まれるため、\ ``items``\ に\ ``scopes``\ を指定する。
        | \ ``scopes``\ の型は\ ``LinkedHashMap``\ であり、スコープ名をキーとして 、そのスコープに対する認可可否の情報を保持している。
    * - | (3)
      - | \ ``user_oauth_approval``\ をhidden項目として埋め込むことで、Spring Security OAuthがリクエストパラメータに\ ``user_oauth_approval``\ を付与する。
        | リクエストパラメータに付与された\ ``user_oauth_approval``\ は、認可エンドポイントのスコープ認可を行うメソッドを実行するために用いられる。
    * - | (4)
      - | CSRFトークン値を引き渡すために、HTMLの\ ``<form>``\ 属性の中に\ ``<sec:csrfInput />``\ 属性を指定する。

.. _OAuthAuthorizationServerHowToHandleError:

認可リクエスト時のエラー画面のカスタマイズ
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

認可エンドポイントで不正クライアントエラー（クライアント未存在エラー等のセキュリティに関わるエラーや、リダイレクトURIチェックエラー）が発生した場合、Spring Security OAuthが提供する\ ``OAuth2Exception``\ がスローされ 、リクエストは\ ``/oauth/error``\ にフォワードされる。
そのため\ ``/oauth/error``\ をハンドリングするコントローラを作成する必要がある。
不正クライアントエラーの詳細については\ :ref:`DefinitionOfBadClientError`\を参照されたい。

以下にコントローラの実装例を示す。

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
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@RequestMapping``\ アノテーションを使用して、\ ``/oauth/error``\ へのアクセスに対するメソッドとしてマッピングを行う。


次に、表示させるエラー画面のJSPを作成する。
認可エンドポイントで発生した例外がModelの属性名\ ``error``\ に登録されているため、例外の内容を画面に表示する。

* ``oauthError.jsp``

.. code-block:: jsp

    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="utf-8">
    <title>OAuth Error!</title>
    <link rel="stylesheet" href="${pageContext.request.contextPath}/resources/app/css/styles.css">
    </head>
    <body>
        <div id="wrapper">
            <h1>OAuth Error!</h1>
            <c:if test="${not empty error}">
                <p>${f:h(error.oAuth2ErrorCode)}</p> <!-- (1) -->
                <p>${f:h(error.message)}</p> <!-- (2) -->
            </c:if>
        <br>
        </div>
    </body>
    </html>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | 例外に設定されているOAuth 2.0のエラーコードを出力する。
        | エラーコードは、\ ``invalid_request``\ 、\ ``invalid_client``\ といった値を持つ。
    * - | (2) 
      - | 例外に設定されているエラーメッセージを出力する。

.. note::

    認可エンドポイントで発生したエラーが、不正クライアントエラー以外の場合、
    クライアントの呼び出し元コントローラにリダイレクトすることでクライアント側にエラー通知を行う。
    エラー通知を行う際のエラー情報はリダイレクトURIにリクエストパラメータとして付与される。
    エラーハンドリングについては\ :ref:`OAuthClientHandleErrorWhenAccessingAuthorizationEndpointWithCode`\を参照されたい。

.. tip::

    アプリケーションでInternet Explorer/Microsoft Edgeをサポートする場合、エラー画面の応答として生成されるHTMLのサイズに注意する必要がある。

    Internet Explorer/Microsoft Edgeでは、応答されたHTMLのサイズが規定値以下だと、アプリケーションが用意したエラー画面の代わりに、Internet Explorer/Microsoft Edgeが用意した簡易メッセージが表示されるためである。

    参考までに、Internet Explorerでの詳細な条件は、「`Friendly HTTP Error Pages <https://blogs.msdn.microsoft.com/ieinternals/2010/08/18/friendly-http-error-pages/>`_」を参照されたい。

.. _OAuthAuthorizationServerHowToConfigureAccessToken:

リソースサーバへのアクセストークンの連携方法
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
リソースサーバがアクセストークンを元にリソースへのアクセスに対する認可判定を行えるよう、認可サーバは\ ``AuthorizationServerTokenServices``\ を介してアクセストークンを連携する。
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
        | 認可サーバは\ ``AuthorizationServerTokenServices``\ の実装クラスとして\ ``DefaultTokenServices``\ を、\ ``TokenStore``\ として\ ``org.springframework.security.oauth2.provider.token.store.JdbcTokenStore``\ を指定する。
    * - | (2)
      - | HTTPアクセスを介した連携
      - | HTTPアクセスにより、アクセストークンを連携する方法。
        | リソースサーバと認可サーバが共有DBを利用できない場合に、この方法を利用する。
        | リソースサーバはアクセストークンの取得及び検証を認可サーバに依頼するため、認可サーバに負荷がかかる。
        | 認可サーバは\ ``AuthorizationServerTokenServices``\ の実装クラスとして\ ``DefaultTokenServices``\ を指定する。
        | アクセストークンをDBで管理する場合は\ ``JdbcTokenStore``\ を、メモリで管理する場合は\ ``org.springframework.security.oauth2.provider.token.store.InMemoryTokenStore``\ を\ ``TokenStore``\ として指定する。
        | アクセストークンをメモリで管理する実装はサーバ再起動などでアクセストークンが失われるため、テスト用途専用の実装である。
    * - | (3)
      - | JWTを利用した連携
      - | JWTを利用し、アクセストークンを連携する方法。
        | リソースサーバと認可サーバが共有DBを利用できない場合に、この方法を利用する。
        | HTTPアクセスを介した連携と比べ、認可サーバにアクセストークンの取得を依頼しないため、認可サーバへの負荷がかからない。
        | 認可サーバは\ ``AuthorizationServerTokenServices``\ の実装クラスとして\ ``DefaultTokenServices``\ を、\ ``TokenStore``\ として\ ``org.springframework.security.oauth2.provider.token.store.JwtTokenStore``\ を指定する。
        | \ ``org.springframework.security.oauth2.provider.token.store.JwtAccessTokenConverter``\ を利用することでアクセストークンの署名とエンコード、デコードを行う。
        | アクセストークンの署名とその検証には公開鍵を使用する方法と、共通鍵を使用する方法がある。
    * - | (4)
      - | メモリを介した連携
      - | メモリを共有することで、アクセストークンを連携する方法。
        | リソースサーバと認可サーバが一つのプロセスとなるようアプリケーションを設計している場合に利用可能。
        | 認可サーバは\ ``AuthorizationServerTokenServices``\ の実装クラスとして\ ``DefaultTokenServices``\ を、\ ``TokenStore``\ として\ ``InMemoryTokenStore``\ を指定する。
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
             user-approval-handler-ref="userApprovalHandler"
             token-services-ref="tokenServices">  <!-- (1) -->
            <oauth2:authorization-code />
            <oauth2:refresh-token />
        </oauth2:authorization-server>

        <!-- omitted -->

        <bean id="tokenServices"
            class="org.springframework.security.oauth2.provider.token.DefaultTokenServices">  <!-- (2) -->
            <property name="tokenStore" ref="tokenStore" />
            <property name="clientDetailsService" ref="clientDetailsService" />
            <property name="supportRefreshToken" value="true" />  <!-- (3) -->
        </bean>

        <bean id="tokenStore"
          class="org.springframework.security.oauth2.provider.token.store.JdbcTokenStore"> <!-- (4) -->
          <constructor-arg ref="dataSource" />
        </bean>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

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
        | データソースの設定方法については、\ :ref:`data-access-common_howtouse_datasource`\ を参照されたい。


\ ``JdbcTokenStore``\ がアクセストークンを連携するために、Spring Security OAuthがスキーマ定義している以下のDBを作成する。
以下の例では共有DBとしてPostgreSQLを使用した場合のDB定義を説明する。

.. figure:: ./images/OAuth_ERDiagramToken.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | アクセストークンを管理するテーブル。認可サーバで発行したアクセストークンの情報をリソースサーバと共有するために使用する。
        | 各カラムの役割は以下のとおりである。

        * authentication_id：認証情報を一意に識別する認証キーを保持するカラム。主キーとする。
        * | token：トークンの情報をシリアル化してバイナリとして保持するカラム。
          | 保持するトークンの情報としては、アクセストークンの有効期限、スコープ、アクセストークンのトークンID、リフレッシュトークンのトークンID、使用しているトークンの種類を表すトークンタイプを保持する。
        * token_id：アクセストークンを一意に識別するトークンIDを保持するカラム。
        * user_name：認証されたリソースオーナのユーザ名を保持するカラム。
        * client_id：認証されたクライアントのクライアントIDを保持するカラム。
        * authentication：リソースオーナとクライアントの認証情報をシリアル化してバイナリとして保持するカラム。
        * refresh_token：アクセストークンに紐付くリフレッシュトークンのトークンIDを保持するカラム。

    * - | (2)
      - | アクセストークンに紐付くリフレッシュトークンを管理するテーブル。
        | 各カラムの役割は以下のとおりである。

        * token_id：リフレッシュトークンを一意に識別するトークンIDを保持するカラム。主キーとする。
        * token：トークンの情報をシリアル化してバイナリとして保持するカラム。リフレッシュトークンの有効期限を保持する。
        * | authentication：リソースオーナとクライアントの認証情報をシリアル化してバイナリとして保持するカラム。
          | アクセストークンを管理するテーブルで保持している認証情報と同じ情報を保持する。

|

.. note::

    共有DBでトークンを管理する場合、有効期限切れとなったトークンはクライアントがアクセストークンを利用するタイミングに削除される。
    そのためトークンが有効期限切れになったとしても、クライアントがアクセストークンを利用しなければ削除されない。
    有効期限切れとなったトークンをDBから削除するためには、バッチ処理等で別途パージを行う必要がある。


.. _OAuthAuthorizationServerHowToCancelToken:

トークンの取り消し（認可サーバ）
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
発行したアクセストークンの取り消しの実装方法について説明する。

アクセストークンの取り消しは、\ ``ConsumerTokenService``\ インタフェースを実装したクラスの
\ ``revokeToken``\ メソッドを呼び出すことで実現できる。
\ ``DefaultTokenServices``\ クラスは\ ``org.springframework.security.oauth2.provider.token.ConsumerTokenServices``\ インタフェースを実装している。

アクセストークンの取り消し時に認可情報も削除することが可能である。
アクセストークンの取り消し後に認可情報を削除せずに認可リクエストを行うと、前回の認可リクエスト時の認可情報が再利用される場合がある。
前回の認可リクエスト時の認可情報は、認可情報の有効期限が有効であり、認可リクエストしたスコープが全て認可されている場合に再利用される。

以下に、実装例を示す。

トークンの取り消しを行うサービスクラスのインタフェースと実装クラスを作成する。

* ``RevokeTokenService.java``

.. code-block:: java

    public interface RevokeTokenService {

        ResponseEntity<Map<String,String>> revokeToken(String tokenValue, String clientId);

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

        @Inject
        JodaTimeDateFactory dateFactory;

        public String revokeToken(String tokenValue, String clientId){ // (4)
            // (5)
            OAuth2Authentication authentication = tokenStore.readAuthentication(tokenValue);

            Map<String,String> map = new HashMap<>();

            if (authentication != null) {
                if (clientId.equals(authentication.getOAuth2Request().getClientId())) { // (6)
                    // (7)
                    Authentication user = authentication.getUserAuthentication();
                    if (user != null) {
                        Collection<Approval> approvals = new ArrayList<>();
                        for (String scope : authentication.getOAuth2Request().getScope()) {
                            approvals.add(
                                    new Approval(user.getName(), clientId, scope, dateFactory.newDate(), ApprovalStatus.APPROVED));
                        }
                        approvalStore.revokeApprovals(approvals);
                    }
                    consumerService.revokeToken(tokenValue); // (8)
                    return ResponseEntity.ok().body(map);

                } else {
                    map.put("error", "invalid_client");
                    return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body(map);
                }
            } else {
                map.put("error", "invalid_request");
                return ResponseEntity.badRequest().body(map);
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
      - | アクセストークンの取り消しを行う\ ``ConsumerTokenService``\ インタフェースのBeanをインジェクションする。
    * - | (2)
      - | アクセストークン発行時の認証情報を取得するために使用する\ ``TokenStore``\ のBeanをインジェクションする。
    * - | (3)
      - | アクセストークンの発行時の認可情報を取得するために使用する\ ``ApprovalStore``\ のBeanをインジェクションする。
        | アクセストークンの取り消し時に認可情報を削除しない場合は不要となる。
    * - | (4)
      - | 取り消しを行うアクセストークンの値と、クライアントのチェックを行うために使用するクライアントIDをパラメータとして受け取る。
    * - | (5)
      - | \ ``TokenStore``\ の\ ``readAuthentication``\ メソッドを呼び出し、アクセストークンを発行した際の認証情報を取得する。
        | 認証情報が正常に取得できた場合のみ、トークンの削除処理を行う。
    * - | (6)
      - | 認証情報より、アクセストークン発行時に使用したクライアントIDを取得し、リクエストパラメータのクライアントIDと一致するかを確認する。
        | アクセストークン発行時のクライアントIDと一致する場合のみ、アクセストークンの削除を行う。
    * - | (7)
      - | 認証情報より、アクセストークン発行時のリソースオーナの認証情報を取得する。
        | リソースオーナの認証情報が取得できた場合、\ ``TokenStore``\ の\ ``revokeApprovals``\ メソッドを呼び出し、認可情報の削除を行う。
        | クライアントクレデンシャルグラントを使用している場合はリソースオーナの認証情報が存在しないため、\ ``revokeApprovals``\ メソッドに渡すパラメータが生成できない。
        | そのため、リソースオーナの認証情報が取得できない場合は認可情報の削除処理は行わない。
        | アクセストークンの取り消し時に認可情報を削除しない場合、この処理は不要となる。
    * - | (8)
      - | \ ``ConsumerTokenService``\ の\ ``revokeToken``\ メソッドを呼び出し、アクセストークンとアクセストークンに紐付くリフレッシュトークンの削除を行う。


トークンの取り消しリクエストを受けるコントローラを作成する。

* ``TokenRevocationRestController.java``

.. code-block:: java

    @RestController
    @RequestMapping("oauth")
    public class TokenRevocationRestController {

        @Inject
        RevokeTokenService revokeTokenService;

        @RequestMapping(value = "tokens/revoke", method = RequestMethod.POST) // (1)
        public ResponseEntity<Map<String,String>> revoke(@RequestParam("token") String tokenValue,
            @AuthenticationPrincipal UserDetails user){

            // (2)
            String clientId = user.getUsername();
            ResponseEntity<Map<String,String>> result = revokeTokenService.revokeToken(tokenValue, clientId); // (3)
            return result;
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
      - | \ ``@RequestMapping``\ アノテーションを使用して、\ ``/oauth/tokens/revoke``\ へのアクセスに対するメソッドとしてマッピングを行う。
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
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
認可サーバにおけるトランザクション制御の注意点について説明する。

Spring Security OAuthが取り扱う情報（認可コード、認可情報、トークン）をDBにて管理する場合には、トランザクション管理について考慮する必要がある。

アクセストークンやリフレッシュトークンを発行する\ ``DefaultTokenServices``\ には\ ``@Transactional``\ アノテーションが付与されており、
トランザクション管理が行われるが、認可コードを操作する\ ``org.springframework.security.oauth2.provider.code.AuthorizationCodeServices``\ や認可情報を操作する
\ ``UserApprovalHandler``\ には\ ``@Transactional``\ アノテーションが付与されておらず、トランザクション管理が行われない。

そのため、\ ``DataSource``\ から取得するコネクションの自動コミットが無効となっている場合、管理対象となる情報がDBに登録されないため、トランザクション制御の設定が必須となる。
なお、自動コミットが有効な場合でも、データの一貫性を担保するためにトランザクションを管理する必要がある。

認可コード、認可情報をDBにて管理する場合のトランザクション制御設定例を以下に示す。

* ``oauth2-auth.xml``

.. code-block:: xml

    <!-- omitted -->

    <tx:advice id="oauthTransactionAdvice"> <!-- (1) -->
        <tx:attributes>
            <tx:method name="*"/>
        </tx:attributes>
    </tx:advice>

    <aop:config>
        <aop:pointcut id="authorizationOperation"
                      expression="execution(* org.springframework.security.oauth2.provider.code.AuthorizationCodeServices.*(..))"/> <!-- (2) -->
        <aop:pointcut id="approvalOperation"
                      expression="execution(* org.springframework.security.oauth2.provider.approval.UserApprovalHandler.*(..))"/> <!-- (3) -->
        <aop:advisor pointcut-ref="authorizationOperation" advice-ref="oauthTransactionAdvice"/>
        <aop:advisor pointcut-ref="approvalOperation" advice-ref="oauthTransactionAdvice"/>
    </aop:config>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | AOPを利用したトランザクション管理を行なうため、\ ``tx:advice``\ を設定する。
        | このタグを使用するために\ ``tx``\のネームスペースとスキーマを追加している。
    * - | (2)
      - | AOPを使用し、認可コードを操作する各メソッドにトランザクション境界を設定する。
        | このタグを使用するために\ ``aop``\のネームスペースとスキーマを追加している。
    * - | (3)
      - | AOPを使用し、認可情報を操作する各メソッドにトランザクション境界を設定する。

|

.. _ImplementationOAuthResourceServerOfAutorizationCode:

リソースサーバの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

リソースサーバの実装方法について説明する。

リソースサーバでは、アクセストークンの検証とリソースに対しての認可制御をSpring Security OAuthの機能を使用して提供する。

ここではTODOリソースのREST APIに対して認可制御を実現する方法を説明する。

設定ファイルの作成（リソースサーバ）
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

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
            <sec:custom-filter ref="userIdMDCPutFilter" after="ANONYMOUS_FILTER"/>
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
      - | \ ``pattern``\属性には認可制御の対象とするパスのパターンを指定する。
        | \ ``entry-point-ref``\には\ ``OAuth2AuthenticationEntryPoint``\のBeanを指定する。ここでの設定は定義上必要だが指定したBeanは使用されない。
          実際に使用されるのは後述する\ ``OAuth2AuthenticationProcessingFilter``\に指定している\ ``OAuth2AuthenticationEntryPoint``\のBeanである。
    * - | (2)
      - | \ ``access-denied-handler``\には\ ``OAuth2AccessDeniedHandler``\のBeanを設定する。ここでは(5)で定義している\ ``oauth2AccessDeniedHandler``\のBeanを指定する。
    * - | (3)
      - | 本実装例ではCSRF対策を無効化する。
        | CSRF対策については\ :ref:`RESTHowToUseSecurityCsrf`\を参照されたい。
    * - | (4)
      - | \ ``custom-filter``\にはリソースサーバ用の認証フィルタとして\ ``OAuth2AuthenticationProcessingFilter``\を設定する。
        | ここでは(7)で定義している\ ``oauth2AuthenticationFilter``\のBeanを指定する。
        | \ ``OAuth2AuthenticationProcessingFilter``\はリクエストに含まれるアクセストークンを利用してPre-Authenticationを行うためのフィルタであるため、
          \ ``before``\に\ ``PRE_AUTH_FILTER``\を指定し\ ``PRE_AUTH_FILTER``\の前に\ ``OAuth2AuthenticationProcessingFilter``\の処理が実行されるように設定する。
        | Pre-Authenticationについては\ `Pre-Authentication Scenarios <http://docs.spring.io/spring-security/site/docs/3.0.x/reference/preauth.html>`_\を参照されたい。
    * - | (5)
      - | Spring Security OAuthが提供するリソースサーバ用の\ ``AccessDeniedHandler``\を定義する。
        | \ ``OAuth2AccessDeniedHandler``\は、認可エラー時に発生する例外をハンドリングしてエラー応答を行う。
    * - | (6)
      - | OAuth用のエラー応答を行うための\ ``OAuth2AuthenticationEntryPoint``\をBean定義する。
    * - | (7)
      - | Spring Security OAuthが提供するリソースサーバ用のServletFilterを定義する。
        | \ ``<oauth2:resource-server>``\ タグを使用することで、Authentication Filterとして\ ``OAuth2AuthenticationProcessingFilter``\ が登録される。
        | \ ``id``\属性に指定した文字列はBeanのIDとなる。ここでは ``oauth2AuthenticationFilter``\を指定している。
        | \ ``resource-id``\属性にはサーバが提供するリソースのIDを指定する。ここでは\ ``todoResource``\を指定している。
        | アクセストークンに紐付くクライアント情報のリソースIDに対し、\ ``resource-id``\属性に指定したリソースIDが含まれているか検証が行われる。
        | 検証の結果、リソースIDが含まれている場合のみリソースに対してのアクセスを許可する。
        | なお、\ ``resource-id``\の定義は任意であり、定義しない場合はリソースIDの検証が行われない。
        | \ ``token-services-ref``\属性には\ ``TokenServices``\のIDを指定する。\ ``TokenServices``\については後述する。
        | \ ``entry-point-ref``\属性には\ ``OAuth2AuthenticationEntryPoint``\のBeanを指定する。ここでは\ ``oauth2AuthenticationEntryPoint``\を指定している。

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
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``oauth2-resource.xml``\ で設定したパスのパターンを内包するようなパスが\ ``spring-security.xml``\ にアクセス制御対象として設定されている場合を考慮し、先に\ ``oauth2-resource.xml``\ を読み込むようにする。

|

リソースにアクセス可能なスコープの設定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

リソースごとにアクセス可能なスコープを定義するために、OAuth 2.0用のBean定義ファイルに
スコープの定義とSpEL式をサポートするためのBean定義を追加する。


実装例は以下のとおりである。

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
            <sec:custom-filter ref="oauth2AuthenticationFilter"
                                    before="PRE_AUTH_FILTER" />
            <sec:custom-filter ref="userIdMDCPutFilter" after="ANONYMOUS_FILTER"/>
        </sec:http>

        <!-- omitted -->

        <oauth2:web-expression-handler id="oauth2ExpressionHandler" /> <!-- (3) -->

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
      - | \ ``expression-handler``\には\ ``org.springframework.security.oauth2.provider.expression.OAuth2WebSecurityExpressionHandler``\のBeanを指定する。
    * - | (2)
      - | \ ``intercept-url``\を使用してリソースに対してスコープによるアクセスポリシーを定義する。
        | \ ``pattern``\属性には保護したいリソースのパスのパターンを指定する。本実装例では /api/v1/todos/ 配下のリソースが保護される。
        | \ ``method``\属性にはリソースのHTTPメソッドを指定する。
        | \ ``access``\属性にはリソースへのアクセスを認可するscopeを指定する。設定値は大文字、小文字を区別する。
        | ここではSpEL式を用いて指定を行っている。
    * - | (3)
      - | \ ``OAuth2WebSecurityExpressionHandler``\をBean定義する。
        | このBeanを定義することでSpring Security OAuthが提供する認可制御を行うためのSpELがサポートされる。
        | なお、``id``\属性に指定した値がこのbeanのidとなる。

|

Spring Security OAuthが用意している主なExpressionを紹介する。

詳細については\ ``org.springframework.security.oauth2.provider.expression.OAuth2SecurityExpressionMethods``\の\ `JavaDoc <http://docs.spring.io/spring-security/oauth/apidocs/org/springframework/security/oauth2/provider/expression/OAuth2SecurityExpressionMethods.html>`_\を参照されたい。

.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table:: **Spring Security OAuthが用意しているExpression**
    :header-rows: 1
    :widths: 35 65
    :class: longtable

    * - Expression
      - 説明
    * - | \ ``hasScope(String scope)``\
      - | クライアントがリソースオーナから認可されているスコープと引数のスコープが一致する場合に\ ``true``\を返却する。
    * - | \ ``hasAnyScope(String... scopes)``\
      - | クライアントがリソースオーナから認可されているスコープと引数のスコープのいずれかが一致する場合に\ ``true``\を返却する。
    * - | \ ``hasScopeMatching(String scopeRegex)``\
      - | クライアントがリソースオーナから認可されているスコープと引数に指定された正規表現が一致する場合に\ ``true``\を返却する。
    * - | \ ``hasAnyScopeMatching(String... scopesRegex)``\
      - | クライアントがリソースオーナから認可されているスコープと引数に指定されたいずれかの正規表現が一致する場合に\ ``true``\を返却する。
    * - | \ ``clientHasRole(String role)``\
      - | クライアントが引数に指定されたロールを持っている場合に\ ``true``\を返却する。
    * - | \ ``clientHasAnyRole(String... roles)``\
      - | クライアントが引数に指定されたいずれかのロールを持っている場合に\ ``true``\を返却する。
    * - | \ ``denyOAuthClient``\
      - | OAuth 2.0でのリクエストを拒否する。リソースオーナのみがリソースにアクセスできるようにするために使用される。
    * - | \ ``isOAuth``\
      - | OAuth 2.0でのリクエストを許可する。クライアントがリソースにアクセスできるようにするために使用される。

.. note::

    SpEL式についてはSpring Securityが提供するSpELを合わせて使用することができる。

    Spring Securityが提供するSpELについては :ref:`built-incommon-expressions` を参照されたい。

|

認可サーバとのアクセストークンの連携方法
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

認可サーバとリソースサーバはアクセストークンを\ ``TokenServices``\を介して連携する。

連携方法の種類については\ :ref:`OAuthAuthorizationServerHowToConfigureAccessToken`\を参照されたい。

ここではDBを共有する方法で設定を行う。

設定の解説については認可サーバでの\ ``TokenServices``\の設定と同様なため\ :ref:`OAuthAuthorizationServerHowToConfigureAccessToken`\を参照されたい。


* ``oauth2-resource.xml``

.. code-block:: xml

        <bean id="tokenServices"
            class="org.springframework.security.oauth2.provider.token.DefaultTokenServices">
            <property name="tokenStore" ref="tokenStore" />
        </bean>

        <bean id="tokenStore"
          class="org.springframework.security.oauth2.provider.token.store.JdbcTokenStore">
          <constructor-arg ref="dataSource" />
        </bean>

.. note::

    認可サーバとリソースサーバを連携させる方法として、Spring Security OAuthで提供されている\ ``org.springframework.security.oauth2.provider.token.RemoteTokenServices``\を利用する方法や、
    JSON Web Tokenを利用する方法がある。
    Spring Security OAuthで提供されている\ ``RemoteTokenServices``\を利用する方法については\ :ref:`OAuthAuthorizationServerHowToCooperateWithHttp`\を参照されたい。

|

.. _OAuthResourceServerGetPrincipal:

ユーザ情報の取得
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

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
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ 引数 \ ``user``\にリソースオーナの認証情報が格納される。

|


クライアントの認証情報を取得したい場合は、Controllerクラスのメソッド引数に\ ``org.springframework.security.oauth2.provider.OAuth2Authentication``\ を指定する。
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
    :class: longtable

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


.. _ImplementationOAuthClientServerOfAutorizationCode:

クライアントの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

クライアントの実装方法について説明する。

ここでは\ ``OAuth2RestTemplate``\ を使用して実現する方法を説明する。

\ ``OAuth2RestTemplate``\ では、OAuth 2.0独自の機能として、\ ``AccessTokenProvider``\ によるグラントタイプに応じた
アクセストークンの取得や、\ ``OAuth2ClientContext``\ による複数リクエスト間でのアクセストークンの共有、リソースサーバへのアクセス時のエラーハンドリングといった機能を実装している。

クライアントでは、\ ``OAuth2RestTemplate``\を利用し、グラントタイプやスコープなどのアプリケーション用件に沿ったパラメータを定義することで、OAuth 2.0機能を使用したリソースへのアクセスが可能となる。

.. note::

    Spring Security OAuth以外のアーキテクチャで実装されているリソースサーバにアクセスする場合は、REST APIでのアクセスを受け付けているか確認されたい。リソースサーバのAPIが異なる場合は\ `OAuth2RestTemplate`\以外の手段でアクセスする必要がある。

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
    :class: longtable

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
    :class: longtable

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
        <url-pattern>/oauth/*</url-pattern> <!-- (2) -->
    </filter-mapping>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``DelegatingFilterProxy``\ を使用して、フィルタ名(\ ``<filter-name>``\ 属性に指定した値)とBeanIDが一致するBeanをサーブレットフィルタとして登録する。
        | フィルタ名には\ ``<oauth2:client>``\ の\ ``id``\ 属性に指定したBeanIDと同じ値を設定する。
        | なお、Spring Security OAuthで発生する例外が意図しない例外ハンドリングが行われないようにするために、\ ``OAuth2ClientContextFilter``\ はサーブレットフィルタの定義の一番最後に記述することを推奨する。
    * - | (2)
      - | \ ``UserRedirectRequiredException``\ が発生する可能性があるパスに対して\ ``OAuth2ClientContextFilter``\ を適用している。
        | 上記例では、すべてのリクエストに対して\ ``OAuth2ClientContextFilter``\ を適用している。

.. note::
    \ ``OAuth2ClientContextFilter``\ は、認可サーバが認可処理後にユーザエージェントを戻すリダイレクトURIの生成をフィルタの前処理で行っているため、
    \ ``/*``\ のような広範囲な定義にすると\ ``UserRedirectRequiredException``\ が発生しないパスに対して無駄な処理が行われることになる。

.. note::
    \ ``OAuth2ClientContextFilter``\ は、認可サーバがリソースオーナの認可を取得した後にリダイレクトさせるURLをリクエストスコープに
    \ ``currentUri``\ という属性名で格納する。そのため、クライアントでは\ ``currentUri``\ という属性名を使用することはできない。

|

前述のとおり、\ ``org.springframework.security.oauth2.client.filter.OAuth2ClientContextFilter``\ は\ ``UserRedirectRequiredException``\ を捕捉し、認可サーバに対してリダイレクトさせるサーブレットフィルタである。

ただし、ブランクプロジェクトで予め設定されている\ ``SystemExceptionResolver``\ が先に\ ``UserRedirectRequiredException``\ をハンドリングしてしまうと
\ ``OAuth2ClientContextFilter``\は期待した動作にならない。

そのため、\ ``spring-mvc.xml``\ の設定を変更し、\ ``SystemExceptionResolver``\ が\ ``UserRedirectRequiredException``\をハンドリングしないようにする必要がある。
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
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``UserRedirectRequiredException``\ を\ ``SystemExceptionResolver``\ のハンドリング対象から除外する。

|

.. _OAuth2RestTemplateSettings:

OAuth2RestTemplateの設定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

\ ``OAuth2RestTemplate``\の設定例を示す。

* ``oauth2-client.xml``

.. code-block:: xml

    <oauth2:resource id="todoAuthCodeGrantResource" client-id="firstSec"
                     client-secret="firstSecSecret"
                     type="authorization_code"
                     scope="READ,WRITE"
                     access-token-uri="${auth.serverUrl}/oauth/token"
                     user-authorization-uri="${auth.serverUrl}/oauth/authorize"/> <!-- (1) -->

    <oauth2:rest-template id="todoAuthCodeGrantResourceRestTemplate" resource="todoAuthCodeGrantResource" /> <!-- (2) -->


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``OAuth2RestTemplate``\が参照する、アクセス対象となるリソースに関する詳細情報を定義する。
        | 各項目の設定値については下記表を参照のこと。
    * - | (2)
      - | \ ``OAuth2RestTemplate``\ を定義する。
        | \ ``id``\ には\ ``OAuth2RestTemplate``\ のBeanIDを指定する。
        | \ ``resource``\には(1)で定義したBeanの\ ``id``\ を指定する。

|

.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table:: **リソース詳細情報**
    :header-rows: 1
    :widths: 35 65
    :class: longtable

    * - 項目
      - 説明
    * - | \ ``id``\
      - | リソースのBeanID。
    * - | \ ``client-id``\
      - | 認可サーバにてクライントを識別するID。
    * - | \ ``client-secret``\
      - | 認可サーバにてクライアントの認証に用いるパスワード。
    * - | \ ``type``\
      - | グラントタイプ。認可コードグラントの場合\ ``authorization_code``\を指定する。
    * - | \ ``scope``\
      - | 認可を要求するスコープをカンマ区切りで列挙する。設定値は大文字、小文字を区別する。
          省略時は認可サーバにおいてクライアントに対して設定しているスコープを全て要求する。
    * - | \ ``access-token-uri``\
      - | アクセストークンの発行を依頼するための認可サーバのエンドポイント。
    * - | \ ``user-authorization-uri``\
      - | リソースオーナの認可を得るための認可サーバのエンドポイント。

.. note::

    本実装例では、実装中に設定する各サーバのコンテキストルートを以下のプレースホルダで表現している。

        .. tabularcolumns:: |p{0.50\linewidth}|p{0.50\linewidth}|
        .. list-table::
            :header-rows: 1
            :stub-columns: 1
            :widths: 50 50
            :class: longtable

            * - プレースホルダのキー
              - 設定値
            * - \ ``auth.serverUrl``\
              - 認可サーバのコンテキストルート
            * - \ ``resource.serverUrl``\
              - リソースサーバのコンテキストルート
            * - \ ``client.serverUrl``\
              - クライアントのコンテキストルート

    また、各サーバのエンドポイントの絶対パスを表現する場合は、パスが明示的にわかるよう、\ ``${auth.serverUrl}/oauth/token``\ のように
    プレースホルダ以下にパスを指定している。実際にアプリケーションを開発する際には、
    例えばエンドポイントが外部システムである場合、外部システム仕様変更時のエンドポイントパス変更に対応するためにパス全体をプレースホルダで表現するなど、要件に応じてパスの指定方法を検討することを推奨する。


.. note::

    \ ``<oauth2:resource>``\ タグでは、アクセストークン取得時のクライアント認証方法を指定する方法として
    \ ``client-authentication-scheme``\ 属性が用意されている。
    \ ``client-authentication-scheme``\ 属性に指定可能な値は以下の通り。

        * \ ``header``\ ：Authorizationヘッダを使用したBasic認証（デフォルト値）
        * \ ``query``\ ：リクエスト時のURLクエリパラメータを使用した認証
        * \ ``form``\ ：リクエスト時のボディパラメータを使用した認証

    本ガイドラインではクライアントの認証にBasic認証を利用するため上記の設定例では未指定としているが、
    アプリケーション要件に沿ったパラメータの指定を行うこと。


.. note::

    Spring Security OAuth以外のアーキテクチャで実装されている認可サーバに対してアクセストークンの発行を依頼する場合は
    リクエストパラメータが上記とは異なる可能性があるため注意されたい。
    その場合はアーキテクチャの仕様を確認し、必要なリクエストパラメータを設定されたい。

|

.. _OAuthAccessOfResourceServer:

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
        RestOperations restOperations; // (1)

        @Value("${resource.serverUrl}/api/v1/todos")
        String url;

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
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``RestOperations``\をインジェクションする。
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

.. warning::

    Spring Securityを利用している場合、ユーザが認証されていない状態で\ ``OAuth2RestTemplate``\ を使用して
    アクセストークンを取得しようとすると\ ``org.springframework.security.authentication.InsufficientAuthenticationException``\
    が発生するため、\ ``OAuth2RestTemplate``\ を使用するパスでは必ずユーザが既に認証されていることを確認されたい。

    具体的には、以下のいずれかが実施されていることを確認すれば良い。

    * \ ``OAuth2RestTemplate``\ を使用するServiceメソッドに対して、\ ``@PreAuthorize``\ により認証済みのチェックをしていること
    * \ ``OAuth2RestTemplate``\ を使用するパスに対して、\ ``<sec:intercept-url>``\ により認証済みのチェックをしていること

    詳細は \ :ref:`AuthorizationToWebResources`\ 及び\ :ref:`AuthorizationToMethod`\ を参照されたい。

    なお、クライアントがリソースオーナとなるためリソースオーナの認証を必要としないクライアントクレデンシャルグラントと、
    \ ``OAuth2RestTemplate``\ を使用しないインプリシットグラントの場合はエラーが発生しない。


|


.. _OAuthClientServerHowToCancelToken:

トークンの取り消し（クライアントサーバ）
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
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
    
    <bean id="basicAuthInterceptor" class="org.springframework.http.client.support.BasicAuthorizationInterceptor">
        <constructor-arg index="0" value="${api.auth.username}" />
        <constructor-arg index="1" value="${api.auth.password}" />
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | 認可サーバにトークン取り消し要求を行うための\ ``RestTemplate``\ をBean定義する。
        | Basic認証用のリクエストヘッダを設定するため、\ ``interceptors``\ プロパティに\ ``ClientHttpRequestInterceptor``\ の実装クラスである\ ``BasicAuthorizationInterceptor``\ を指定する。
        | \ ``BasicAuthorizationInterceptor``\ の設定については\ :ref:`RestClientBasicAuthorizationInterceptorBeanDefinition`\ を参照されたい。


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

        @Value("${auth.serverUrl}/api/v1/oauth/tokens/revoke")
        String revokeTokenUrl; // (1)

        @Inject
        @Named("todoAuthCodeGrantResourceRestTemplate")
        OAuth2RestOperations oauth2RestOperations; // (2)

        @Inject
        @Named("revokeRestTemplate")
        RestOperations revokeRestOperations; // (3)

        @Override
        public String revokeToken() {

            String tokenValue = getTokenValue(oauth2RestOperations);

            String result = "";

            if(StringUtils.hasLength(tokenValue)){
                HttpHeaders headers = new HttpHeaders();
                headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
                MultiValueMap<String, String> variables = new LinkedMultiValueMap<>();
                variables.add("token", tokenValue);

                try {
                    revokeRestOperations.postForObject(url,
                            new HttpEntity<MultiValueMap<String, String>>(variables, headers),
                            Void.class); // (4)
                    result = "success";
                    initContextToken(oauth2RestOperations); // (5)
                } catch (HttpClientErrorException e) {
                    result = "invalid request";
                }
            }else{
                result ="token not exist";
            }
            return result;
        }

        // (6)
        private String getTokenValue(OAuth2RestOperations oauth2RestOperations) {
            OAuth2AccessToken token = restOperations.getOAuth2ClientContext()
                    .getAccessToken();
            if (token == null) {
                return "";
            }
            return token.getValue();
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
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | アクセストークンの取り消しを認可サーバに依頼する際に使用するURL。
    * - | (2)
      - | 取り消しを行うアクセストークンを保持している\ ``OAuth2RestTemplate``\ をインジェクションする。
        | \ ``RestOperations``\の実装が複数存在するため、\ ``@Named``\でBean名を指定してインジェクションする。
    * - | (3)
      - | アクセストークンの取り消しを行う\ ``RestTemplate``\ をインジェクションする。
        | \ ``RestOperations``\の実装が複数存在するため、\ ``@Named``\でBean名を指定してインジェクションする
    * - | (4)
      - | 認可サーバにアクセストークンの取り消しを行うために、RESTでメソッドPOSTでアクセスする。
        | 取り消しを行うアクセストークンの値を認可サーバに渡すためにリクエストパラメータに設定する。
        | アクセストークンは(6)で定義している\ ``getTokenValue``\ メソッドに\ ``org.springframework.security.oauth2.client.OAuth2RestOperations``\ を渡して取得する。
    * - | (5)
      - | \ ``OAuth2RestOperations``\で保持しているアクセストークンを削除する。
        | アクセストークンの削除は(7)で定義している\ ``initContextToken``\ メソッドにアクセストークンを保持している\ ``OAuth2RestOperations``\ を渡して削除する。
    * - | (6)
      - | \ ``OAuth2RestOperations``\ で保持しているアクセストークンを取得するメソッド。
        | パラメータとして渡された\ ``OAuth2RestOperations``\ の\ ``getAccessToken``\ メソッドを呼び出すことでアクセストークンを取得し、返却する。
    * - | (7)
      - | \ ``OAuth2RestOperations``\ で保持しているアクセストークンを削除するメソッド。
        | パラメータとして渡された\ ``OAuth2RestOperations``\ の\ ``setAccessToken``\ メソッドにnullを渡すことでアクセストークンを削除する。



| クライアントは上記で作成したサービスをアクセストークンが不要になったタイミングで呼び出すことでトークンの取り消しを行う。
| Spring Security OAuthのデフォルト実装ではセッションスコープでアクセストークンを保持するため、クライアントのユーザがログアウトした場合やセッションタイムアウトによってセッションが破棄されるタイミングでトークンの取り消しを行うことが考えられる。


.. note::

    Spring Security OAuth以外のアーキテクチャで実装されている認可サーバでは、アクセストークンの削除をREST APIで受け付けていない場合がある。

    その場合は認可サーバのアーキテクチャ仕様を確認し、適切に実装する必要がある。

|

.. _OAuthClientErrorHandlingWithCode:

エラーハンドリング
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

\ ``OAuth2RestTemplate``\を用いた認可サーバ、リソースサーバへのアクセス時に発生するエラーのハンドリング方法について説明する。

.. _OAuthClientHandleErrorWhenAccessingAuthorizationEndpointWithCode:

認可エンドポイントアクセス時のエラーハンドリング
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
認可エンドポイントで発生するエラーは不正アクセスエラーと不正クライアントエラー以外の例外に分類される。
不正クライアントエラーの詳細については、\ :ref:`DefinitionOfBadClientError`\を参照されたい。
不正クライアントエラーが発生した場合のエラーハンドリングは、\ :ref:`OAuthAuthorizationServerHowToHandleError`\ を参照されたい。
本節では不正クライアントエラー以外エラーが発生した場合のエラーハンドリングについて説明する。

クライアントに通知されるエラーでハンドリングが必要なものは以下となる。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
.. list-table:: **認可エンドポイントで発生するエラー**
    :header-rows: 1
    :widths: 10 20 70
    :class: longtable

    * - 項番
      - 発生するエラー
      - 説明
    * - | (1)
      - | リソースオーナによる認可拒否
        | \ ``org.springframework.security.oauth2.common.exceptions.UserDeniedAuthorizationException``\ 
      - | リソースオーナがスコープ認可画面で、クライアントが指定したスコープを全て拒否した場合に発生する。
        | リダイレクトURIのリクエストパラメータに、\ ``error=access_denied``\ と\ ``error_description=User denied access``\ が設定される。

| 

.. warning:: **エラー時のレスポンスで扱うリクエストパラメータについて**

    RFC 6749の\ `4.1.2.1. Error Response <https://tools.ietf.org/html/rfc6749#section-4.1.2.1>`_\ に定められている通り、
    OAuth 2.0ではエラー時のレスポンスにリクエストパラメータ\ ``error``\ および\ ``error_description``\ を含む。
    このため業務実装時にこれらの名前のパラメータを使用していると、パラメータ名が競合してしまう場合があることに注意されたい。

| 

上記の\ ``OAuth2RestTemplate``\ によってスローされる例外に対するハンドリング処理を実装する必要がある。
本ガイドラインでは、以下の要件に沿ってエラーハンドリングの実装を行う。

* リソースオーナの操作に起因する例外かどうか判断して、エラー画面を変更する
    認可サーバで発生する例外は、認可拒否などリソースオーナの操作に起因する例外と、
    クライアントが指定したスコープが認可サーバのクライアント情報に存在しないなどのアプリケーションの不備に起因する例外に分類できる。
    例外の分類に基づき、以下のように適切にハンドリングする必要がある。

    * リソースオーナの操作に起因する例外は「アクセス拒否画面」に遷移し、業務の再実施を促す
    * アプリケーションの不備に起因しない例外は「システムエラー画面」に遷移し、アプリケーションの設定を見直すよう促す

* URLへの不要なパラメータの露出を防止するため、エラー画面にリダイレクトする
    認可コードグラントやインプリシットグラントのフローでは、URLパラメータに以下のパラメータを付与するが、フローの途中でエラーが発生した場合、
    そのままエラー画面にフォワードするとブラウザのアドレスバーなどにパラメータが露出してしまう。
    これを防止するために、エラー画面にリダイレクトする必要がある。

    * 認可コード（認可コードグラントのみ）
        認可コードの発行からアクセストークン発行までの間にエラーが発生した場合
    * \ ``state``\ パラメータ
        認可リクエストからリソースオーナの認可完了までにエラーが発生した場合

    認可コードグラントやインプリシットグラント以外のグラントタイプでは、上記パラメータを使用しないためリダイレクトではなくフォワードを使用してもよい。

以下に、リソースオーナの認可拒否による\ ``UserDeniedAuthorizationException``\ をハンドリングしてアクセス拒否画面に遷移する例を示す。

なお、システムエラー画面に遷移するハンドリングも同様に実装する必要がある。
ハンドリング対象の例外については、\ :ref:`OAuthAppendixOccuringErrors`\ を参照されたい。
また、例では\ ``UserDeniedAuthorizationException``\ をアプリケーション全体で共通的にハンドリングすることを想定して、
\ ``@ControllerAdvice``\ アノテーションを使用している。詳細は\ :ref:`application_layer_controller_advice`\ を参照されたい。

以下に実装例を示す。

.. code-block:: java

    @ControllerAdvice
    public class OAuth2ExceptionHandler {

        // omitted

        @ExceptionHandler(UserDeniedAuthorizationException.class) // (1)
        public String handleUserDeniedAuthorizationException(
                UserDeniedAuthorizationException e, RedirectAttributes attributes) {
            // omitted
            attributes.addFlashAttribute("exception", e);
            return "redirect:/oauthAccessDeniedError"; // (2)
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
      - | コントローラで発生したエラーをハンドリングする。
        | \ ``@ExceptionHandler``\ のパラメータに指定されている\ ``UserDeniedAuthorizationException``\ が発生した場合に実行される。
    * - | (2)
      - | 認可コードや\ ``state``\ パラメータを隠蔽するためリダイレクトを行う。
        | リダイレクト先に例外を引き渡す場合は、\ ``RedirectAttributes``\ の\ ``addFlashAttribute``\ メソッドを使用して引き渡す例外を設定する。

| 

また、\ ``OAuth2ExceptionHandler``\ の\ ``@ExceptionHandler``\ にてリダイレクトを行うため、
リダイレクト先のパス\ ``/oauthAccessDeniedError``\ に対応するコントローラを定義する必要がある。

以下にコントローラの定義例を示す。

* ``OAuth2ErrorController.java``

.. code-block:: java

        @Controller
        public class OAuth2ErrorController {

            @RequestMapping("/oauthAccessDeniedError") // (1)
            public String handleAccessDeniedError() {
                return "common/error/accessDeniedError";
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
      - | \ ``@RequestMapping``\ アノテーションを使用して、\ ``/oauthAccessDeniedError``\ へのアクセスに対するメソッドとしてマッピングを行う。
        | \ ``/oauthAccessDeniedError``\ が呼び出された場合、アクセス拒否画面を表示する。

|

.. _OAuthClientHandleErrorWhenAccessingTokenEndpointWithCode:

トークンエンドポイント及びリソースサーバアクセス時のエラーハンドリング
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

認可コードグラントではトークンエンドポイントおよびリソースサーバで発生するエラーはすべてシステムエラーとして扱えば良い。
発生するエラーについては \ :ref:`OAuthAppendixOccuringErrors`\ を参照されたい。
また、エラーハンドリングについては\ :ref:`OAuthClientHandleErrorWhenAccessingAuthorizationEndpointWithCode`\ を参照されたい。

.. _ImplementationImplicitGrant:

インプリシットグラントの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

インプリシットグラントを利用した認可サーバ、リソースサーバ、クライアントの実装方法について解説する。

解説は認可コードグラントからの変更点のみを対象に行っている。変更点以外の実装、解説については\ :ref:`ImplementationAutorizationCodeGrant`\を参照されたい。

以下に認可コードグラントの実装から各サーバを実装する際の変更点を示す。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :stub-columns: 1
    :widths: 30 70
    :class: longtable

    * - サーバ
      - 認可コードグラントからの変更点
    * - 認可サーバ
      - \ :ref:`OAuthAuthorizationServerDefinition`\
    * - リソースサーバ
      - なし
    * - クライアント
      - \ :ref:`ImplementationOAuthClientServerOfAutorizationCode`\の全て

|

.. _ImplementationOAuthAuthorizationServerOfImplicit:

認可サーバの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

認可コードグラントの実装から\ ``oauth2-auth.xml``\を変更する必要がある。

\ ``oauth2-auth.xml``\以外の実装は\ :ref:`ImplementationOAuthAuthorizationServerOfAutorizationCode`\を参照し作成されたい。


認可サーバの定義
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

認可コードグラントからの\ ``oauth2-auth.xml``\の変更点を以下に示す。

* ``oauth2-auth.xml``

.. code-block:: xml

        <oauth2:authorization-server
            client-details-service-ref="clientDetailsService"
            user-approval-handler-ref="userApprovalHandler"
            token-services-ref="tokenServices">
            <oauth2:implicit />  <!-- (1) -->
            <oauth2:refresh-token />
        </oauth2:authorization-server>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``<oauth2:implicit />``\ タグを使用して、インプリシットグラントをサポートする。

|

.. warning::

    \ ``<oauth2:implicit />``\ タグと\ ``<oauth2:refresh-token />``\ タグは上記の順番で設定する必要がある。

    詳細は\ :ref:`OrderOfAuthoraizationServerSupportSetting`\を参照されたい。

|

.. _ImplementationOAuthResourceServerOfImplicit:

リソースサーバの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

認可コードグラントの実装から変更は不要である。

実装は\ :ref:`ImplementationOAuthResourceServerOfAutorizationCode`\を参照し作成されたい。

.. _ImplementationOAuthClientServerOfImplicit:

クライアントの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

インプリシットグラントでは一般的にJavaScriptなどで実装されたクライアントが採用される。

Spring Security OAuthではJava以外のライブラリを提供していないため、本ガイドラインではJavaScriptを用いて独自にクライアントを実装する方法について解説する。

.. _OAuthClientUsingJavaScript:

JavaScriptを使用したリソースサーバへのアクセス
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

本ガイドラインでは、インプリシットグラント向けのクライアントの実装として、JavaScriptを使用してリソースサーバから
JSON形式のデータを取得し、画面に表示させる方法を説明する。

.. note::

    以降に説明する実装例ではJavaScriptライブラリとしてjQuery(バージョン3.1.1)を使用する。
    JQueryは\ ``src/main/webapp/resources/vendor``\に格納し、作成したJavaScriptファイルは、\ ``src/main/webapp/resources/app/js``\に格納する。



.. note:: **アクセストークンの格納先**

    HTML5準拠のブラウザを利用する場合、アクセストークンの格納先の代表的な例として、ローカルストレージとセッションストレージが挙げられる。

    以下にローカルストレージとセッションストレージの保持期間、用途を示す。

        .. tabularcolumns:: |p{0.15\linewidth}|p{0.20\linewidth}|p{0.65\linewidth}|
        .. list-table::
            :header-rows: 1
            :stub-columns: 1
            :widths: 15 20 65
            :class: longtable

            * - 格納先
              - 保持期間
              - 用途
            * - ローカルストレージ
              - 明示的にクリアするまで
              - 端末のブラウザを一人のユーザのみが利用する場合で、アクセストークンの発行回数を減らし、サービスの利便性を向上させたい場合
            * - セッションストレージ
              - タブやブラウザを閉じるまで
              - 端末のブラウザを複数のユーザが共用で利用する場合で、アクセストークンの不正利用を防止し、セキュリティを強化したい場合

    本実装例ではローカルストレージを採用する例を紹介する。

|

以下にOAuth 2.0機能を独自に実装したJavaScriptと、それを利用したクライアントの実装例を示す。

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

        var saveTokens = function(providerId, tokens) {
            localStorage.setItem("tokens-" + providerId, JSON.stringify(tokens));
        };

        var getTokens = function(providerId) {
            var tokens = JSON.parse(localStorage.getItem("tokens-" + providerId));
            if (!tokens) tokens = [];

            return tokens;
        };

        var wipeTokens = function(providerId) {
            localStorage.removeItem("tokens-" + providerId);
        };

        var saveToken = function(providerId, token) { // (2)
            var tokens = getTokens(providerId);
            tokens = filterTokens(tokens);
            tokens.push(token);
            saveTokens(providerId, tokens);
        };

        var getToken = function(providerId, scopes) {
            var tokens = getTokens(providerId);
            tokens = filterTokens(tokens, scopes);
            if (tokens.length < 1) return null;
            return tokens[0];
        };

        var sendAuthRequest = function(providerId, scopes) { // (3)
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

        var checkForToken = function(providerId) { // (4)
            var h = window.location.hash;

            if (h.length < 2) return true;

            if (h.indexOf("error") > 0) { // (5)
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

        var handleError = function(providerId, cause) { // (6)
            if (!config[providerId]) throw "Could not retrieve config for this provider.";

            var co = config[providerId];
            var errorDetail = cause["error"];
            var errorDescription = cause["error_description"];

            // redirect error page
            if(co["errRedirectUrl"]) {
                var request = {};
                if (errorDetail) {
                    request["error"] = errorDetail;
                }
                if (errorDescription) {
                    request["error_description"] = errorDescription;
                }
                redirect(encodeURL(co["errRedirectUrl"], request));
            } else {
                alert("Access Error. cause: " + errorDetail + "/"
                        + errorDescription);
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

        var clearTokens = function(config) { // (7)
            var key;
            for(key in config) {
                wipeTokens(key);
            }
        };

        var oajax = function(settings) { // (8)
            var providerId = settings.providerId;
            var scopes = settings.scopes;
            var token = getToken(providerId, scopes);

            if (!token) {
                sendAuthRequest(providerId, scopes);
                return;
            }

            if (!settings.headers) settings.headers = {};
            settings.headers["Authorization"] = "Bearer " + token["access_token"];

            return $.ajax(settings);
        };

        var parseFailureJSON = function(providerId, data) { // (9)
            var res = data.responseJSON;
            var error = res.error;
            var errorDescription = res.error_description;
            var co = config[providerId];

            if (error === "invalid_token" && errorDescription) {
                var tokens = getTokens(providerId);
                for (var token of tokens) {
                    if (errorDescription.indexOf(token["access_token"]) >= 0) {
                        if (errorDescription.indexOf("Invalid access token") < 0) {
                            // clear expired tokens
                            wipeTokens(providerId)

                            // redirect for get access token
                            if (co["redirectUrl"]) {
                                redirect(co["redirectUrl"]);
                                return;
                            }
                        }
                        break;
                    }
                }
            }

            if (co["errRedirectUrl"]) {
                var request = {};
                if (error) {
                    request["error"] = error;
                }
                if (errorDescription) {
                    request["error_description"] = errorDescription;
                }
                redirect(encodeURL(co["errRedirectUrl"], request));
            } else {
                alert("Access Error. cause: " + error + "/" + errorDescription);
            }
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
            },
            parseFailureJSON : function(providerId, data) {
                return parseFailureJSON(providerId, data);
            }
        };

    })(window, jQuery);


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | クライアントが使用可能なアクセストークンを判別し返却する関数(\ ``filterTokens``\)。
        | ローカルストレージに格納されているアクセストークンより、有効期限が超過しておらず、かつパラメータで指定されたスコープと一致するものを返却する。
        | スコープが指定されていない場合は有効期限が超過していないアクセストークンを返却する。
    * - | (2)
      - | ローカルストレージにアクセストークンを格納する関数(\ ``saveToken``\)。
        | 後述するコンフィギュレーション情報の\ ``providerId``\ を引数として受け取り、アクセストークンの配列をローカルストレージに格納する際のキーにする。
        | アクセストークンの配列は\ ``filterTokens``\ から得たアクセストークンに、引数の\ ``token``\を加えたものである。
    * - | (3)
      - | 認可サーバに対して認可を要求する関数(\ ``sendAuthRequest``\)。
        | コンフィギュレーション情報より必要パラメータを取得し、リクエストを作成する。
    * - | (4)
      - | 認可の応答よりアクセストークンを取得する関数(\ ``checkForToken``\)。
        | URLのハッシュで返却される認可の応答を検証し、正常である場合は\ ``saveToken``\ を用いてローカルストレージに情報を格納する。
    * - | (5)
      - | 認可の結果、エラーが返却された場合エラー処理として\ ``handleError``\ を呼び出す。
    * - | (6)
      - | 認可時のエラーを処理する関数(\ ``handleError``\)。
        | 本ガイドラインでは、エラー処理の実装例としてコンフィギュレーション情報にて指定されている
          エラー時のリダイレクト先URLへのリダイレクトを行っている。
    * - | (7)
      - | ローカルストレージに格納されているアクセストークンをクリアする関数(\ ``clearTokens``\)。
    * - | (8)
      - | リソースサーバに対してリソースへのアクセスを要求する関数(\ ``oajax``\)。
        | ローカルストレージよりアクセストークンを取得し、jQueryの\ ``ajax``\関数を用いてリソースサーバへリクエストを行う。
    * - | (9)
      - | \ ``ajax``\ 関数でエラーが発生した場合に返却されるJSONを解析して遷移先を決定する関数(\ ``parseFailureJSON``\ )。
        | アクセストークンが有効期限切れかどうか判断し、有効期限切れの場合はアクセストークンを削除して再度画面表示を行う。
        | 例として、以下の条件で有効期限切れを判断することが可能である。

        * \ ``error``\ ：\ ``invalid_token``\ であること
        * \ ``error_description``\ ：アクセストークンを含み、かつ、\ ``Invalid access token``\ を含まないこと

        | 有効期限切れでない場合は、エラー時のリダイレクト先URLに\ ``error``\ と\ ``error_description``\ を付与してリダイレクトを行う。

        .. note:: **判定条件について**

            上記の判定条件はSpring Security OAuthの仕様で定義されたものではない。
            あくまで現状の\ ``TokenServices``\ の実装に基づいたワークアラウンド的な判定条件であり、
            今後のSpring Security OAuthの実装変更に合わせて変更が必要となる可能性がある点に留意されたい。

            * `DefaultTokenServices <https://github.com/spring-projects/spring-security-oauth/blob/master/spring-security-oauth2/src/main/java/org/springframework/security/oauth2/provider/token/DefaultTokenServices.java#L235>`_\ ：有効期限切れを示す\ ``expired``\ という文字列とアクセストークンを返却する
            * `RemoteTokenServices <https://github.com/spring-projects/spring-security-oauth/blob/master/spring-security-oauth2/src/main/java/org/springframework/security/oauth2/provider/token/RemoteTokenServices.java#L110>`_\ ：有効期限切れを明確に判断できる文字列はなくアクセストークンのみ返却する

            もし、リソースサーバが\ ``RemoteTokenServices``\ を使用しない場合は、以下の条件に緩和することが出来る。

            * \ ``error``\ ：\ ``invalid_token``\ であること
            * \ ``error_description``\ ：\ ``expired``\ を含むこと

|

クライアントの画面を表示するコントローラの実装例を示す。
アクセスする認可サーバやリソースサーバのURLを可変とするため、コントローラでプロパティや環境変数からURLを取得し、画面に連携する。

* ``TodoController.java``

.. code-block:: java

    @Controller
    public class TodoListController {

        @Value("${client.serverUrl}") // (1)
        String applicationContextUrl;

        @Value("${auth.serverUrl}") // (2)
        String authServerUrl;

        @Value("${resource.serverUrl}") // (3)
        String resourceServerUrl;

        @RequestMapping(value = "todo/get", method = RequestMethod.GET)
        public String getTodo(Model model) {
            model.addAttribute("client.serverUrl", applicationContextUr);
            model.addAttribute("auth.serverUrl", authServerUrl);
            model.addAttribute("resource.serverUrl", resourceServerUrl);
            return "todo/todoList";
        }

    }

|

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | クライアントのルートパスを取得する。
    * - | (2)
      - | 認可サーバのルートパスを取得する。
    * - | (3)
      - | リソースサーバのルートパスを取得する。

|

また、エラー時のリダイレクト先URL(\ ``/oauth/error``\ )のハンドリングを行うコントローラを追加する。
以下にコントローラの実装例を示す。

* ``OAuth2ErrorController.java``

.. code-block:: java

        @Controller
        @RequestMapping("/oauth/error")
        public class OAuth2ErrorController {

            // omitted

            @RequestMapping(params = { "error=access_denied",
                    "error_description=User denied access" }) // (1)
            public String handleAccessDeniedError(
                    @RequestParam("error") String error,
                    @RequestParam("error_description") String description) {

                // omitted

                return "common/error/accessDeniedError";
            }

            @RequestMapping // (2)
            public String handleError(
                    @RequestParam(name = "error", required = false) String error,
                    @RequestParam(name = "error_description", required = false) String description) {

                // omitted

                return "common/error/systemError";
            }

        }

|

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | リクエストパラメータがリソースオーナによる認可拒否を示す以下の条件の場合、アクセス拒否画面に遷移する。

        * \ ``error``\ ：\ ``access_denied``\ 
        * \ ``error_description``\ ：\ ``User denied access``\ 
    * - | (2)
      - | (1)以外の場合、システムエラー画面に遷移する。

|

クライアントの画面（JSP）の実装例を示す。
JSPでは、前述の独自に実装したJavaScriptを利用して、認可サーバやリソースサーバにアクセスする。

* ``todoList.jsp``

.. code-block:: jsp

    <script type="text/javascript" src="${pageContext.request.contextPath}/resources/vendor/jquery/jquery.js"></script> <!-- (1) -->
    <script type="text/javascript" src="${pageContext.request.contextPath}/resources/app/js/oauth2.js"></script> <!-- (1) -->
    <script type="text/javascript">
    "use strict";

    $(document).ready(function() {
        var result = oauth2Func.initialize({ // (2)
            "todo" : { // (3)
                clientId : "client", // (4)
                redirectUrl : "${client.serverUrl}/todo/get", // (5)
                errRedirectUrl : "${client.serverUrl}/oauth/error", // (6)
                authorization : "${auth.serverUrl}/oauth/authorize" // (7)
            }
        });

        if (result) {
            oauth2Func.oajax({ // (8)
                url : "${resource.serverUrl}/api/v1/todos", // (9)
                providerId : "todo",  // (10)
                scopes : [ "READ" ],  // (11)
                dataType : "json",    // (12)
                type : "GET"  // (13)
            }).done(function(data) {  // (14)
                $("#message").text(JSON.stringify(data));
            }).fail(function(data) {  // (15)
                oauth2Func.parseFailureJSON("todo", data);
            });
        }
    });

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
      - | 前述の、jQuery、独自に実装したJavaScriptをそれぞれ格納したパスを指定する。
    * - | (2)
      - | 認可要求に使用するコンフィギュレーション情報を定義し、初期化する。
    * - | (3)
      - | クライアント別にコンフィギュレーション情報を区別するための識別子として一意な値を指定する。
        | 後述するリソースへのアクセス処理では、本項目をキーにコンフィギュレーション情報を管理・取得する。
    * - | (4)
      - | クライアントを識別するIDを指定する。
    * - | (5)
      - | 認可サーバのリソースオーナ認証後にクライアントをリダイレクトさせるURLを指定する。
    * - | (6)
      - | 認可応答として認可サーバよりエラーを受信した場合にリダイレクトさせるURLを指定する。
        | 本ガイドラインではエラー受信時の実装例として、クライアントの画面にリダイレクトしエラー画面を
          表示させる方法を示す。
    * - | (7)
      - | 認可サーバの認可エンドポイントを指定する。
    * - | (8)
      - | リソースへのアクセスを実行する。
    * - | (9)
      - | リソースサーバのアクセス先URLを指定する。
    * - | (10)
      - | 参照するコンフィギュレーション情報の識別子を指定する。
    * - | (11)
      - | 認可を要求するスコープを指定する。設定値は大文字、小文字を区別する。
    * - | (12)
      - | レスポンス形式を指定する。
    * - | (13)
      - | メソッドGETでリソースサーバへアクセスする。
    * - | (14)
      - | 処理成功時に行う処理を指定する。\ ``message``\には処理成功時のレスポンスが格納される。
        | 本ガイドラインではレスポンスをそのまま出力している。
    * - | (15)
      - | 処理失敗時に行う処理を指定する。

|


.. warning::

    本実装例ではローカルストレージにアクセストークンを格納する際のキーとして固定の値を使用している。

    本実装例のようにローカルストレージに固定のキーで情報を格納すると、同一端末のブラウザを複数のユーザが利用する場合に、
    あるユーザが格納した情報が別のユーザに意図せず参照されてしまう可能性がある。

    同一端末のブラウザを複数のユーザが利用することが想定される場合は、セッションストレージを採用し、
    ユーザごとにユニークなキーで格納するなど、意図しない不正利用が起こらない仕組みを実装する必要がある。

|

.. note::

    Spring Security OAuth以外のアーキテクチャで実装されている認可サーバに対してアクセストークンの発行を依頼する場合は
    リクエストパラメータが上記とは異なる可能性があるため注意されたい。
    その場合はアーキテクチャの仕様を確認し、必要なリクエストパラメータを設定されたい。

|

 .. todo:: **TBD**

    JavaScriptで作られたクライアントから同一ドメインでない認可サーバやリソースサーバへのアクセスを行う場合、認可サーバやリソースサーバで\ ``Cross-Origin Resource Sharing``\のサポートが必要になる。

    詳細については、次版以降に記載する予定である。

|


トークンの取り消し(クライアントサーバ)
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

発行したアクセストークンの取り消しについて説明する。

認可サーバからアクセストークンを削除する方法については\ :ref:`OAuthAuthorizationServerHowToCancelToken`\を参照されたい。

クライアントではアクセストークンが不要になったタイミングで\ ``oauth2.js``\の\ ``clearTokens``\を呼び出すことでローカルストレージに格納したアクセストークンを削除できる。

以下に実装例を示す。

.. code-block:: js

    $(document).ready(function() {
        oauth2Func.clearTokens({ // (1)
            "todo" : {} // (2)
        });
    });


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | ローカルストレージからアクセストークンを削除する\ ``clearTokens``\関数を呼び出す。
    * - | (2)
      - | アクセストークンを参照するキーとして\ ``initialize``\で設定した一意な値と同じ値を設定する。

|

.. warning::

    ローカルストレージからアクセストークンを削除しても、\ :ref:`OAuthAuthorizationServerHowToCancelToken`\で説明しているように、リソースオーナに対する認可の要求を行わずにアクセストークンを再取得できるケースがある。
    クライアントからアクセストークンを削除するだけではセキュリティ面での対処としては不十分であることに注意されたい。

|

.. warning::

    アクセストークンの削除を行うタイミングによっては、ブラウザの終了などでJavaScriptが実行されない場合がある。
    セキュリティ要件上アクセストークンを削除する必要がある場合は、ユーザがブラウザを終了してしまうことも考慮し、
    ユースケース上必ず実行されるポイントでアクセストークンの削除を行う必要があることに注意されたい。

|

.. _OAuthClientErrorHandlingWithImplicit:

エラーハンドリング
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

インプリシットグラントにおける認可サーバ、リソースサーバへのアクセス時に発生するエラーのハンドリング方法について説明する。

.. _OAuthClientHandleErrorWhenAccessingAuthorizationEndpointWithImplicit:

認可エンドポイントアクセス時のエラーハンドリング
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

認可エンドポイントで発生するエラーでハンドリングするエラーは認可コードグラントと同様であるため\ :ref:`OAuthClientHandleErrorWhenAccessingAuthorizationEndpointWithCode`\を参照されたい。
エラーハンドリングについては\ :ref:`OAuthClientUsingJavaScript`\を参照されたい。

.. _OAuthClientHandleErrorWhenAccessingTokenEndpointWithImplicit:

リソースサーバアクセス時のエラーハンドリング
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

リソースサーバへアクセス時のエラーでハンドリングが必要なものは以下となる。

|

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
.. list-table:: **リソースサーバで発生するエラー**
    :header-rows: 1
    :widths: 10 20 70
    :class: longtable

    * - 項番
      - 発生するエラー
      - 説明
    * - | (1)
      - | アクセストークン検証エラー
        | \ ``org.springframework.security.oauth2.common.exceptions.InvalidTokenException``\ 
      - | リソースサーバが受け取ったアクセストークンの正当性を確認出来ない場合（異常系）や、アクセストークンの有効期限が切れている場合（正常系）に発生する。
        |
        | アクセストークン検証エラーは正常系と異常系が混在しているため、インプリシットグラント等\ ``OAuth2RestTemplate``\ を利用しない場合にこのエラーを受け取った場合、アクセストークンの有効期限切れかどうか判定する必要がある。

|

エラーハンドリングについては\ :ref:`OAuthClientUsingJavaScript`\を参照されたい。

.. note::

    上記ではリソースオーナの操作により発生するエラーのハンドリングのみ紹介しているが、システムエラーもハンドリングする必要がある。
    発生するエラーについては \ :ref:`OAuthAppendixOccuringErrors`\ を参照されたい。

|

.. _ImplementationResourceOwnerPasswordCredentialsGrant:

リソースオーナパスワードクレデンシャルグラントの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

リソースオーナパスワードクレデンシャルグラントを利用した認可サーバ、リソースサーバ、クライアントの実装方法について説明する。

解説は認可コードグラントからの変更点のみを対象に行っている。変更点以外の実装、解説については\ :ref:`ImplementationAutorizationCodeGrant`\を参照されたい。

以下に認可コードグラントの実装から各サーバを実装する際の変更点を示す。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :stub-columns: 1
    :widths: 30 70
    :class: longtable

    * - サーバ
      - 認可コードグラントからの変更点
    * - 認可サーバ
      - | \ :ref:`OAuthAuthorizationServerDefinition`\
        | \ :ref:`OAuthAuthorizationServerClientAuthentication`\
        | \ :ref:`OAuthAuthorizationServerHowToCancelToken`\
    * - リソースサーバ
      - なし
    * - クライアント
      - \ :ref:`OAuth2RestTemplateSettings`\

|

.. _ImplementationOAuthAuthorizationServerOfPassword:

認可サーバの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

認可コードグラントの実装から\ ``oauth2-auth.xml``\を変更し、DB及びServiceクラスから不要な定義を削除する必要がある。

\ ``oauth2-auth.xml``\以外の実装は\ :ref:`ImplementationOAuthAuthorizationServerOfAutorizationCode`\を参照し作成されたい。


認可サーバの定義
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

認可コードグラントからの\ ``oauth2-auth.xml``\の変更点を以下に示す。

* ``oauth2-auth.xml``

.. code-block:: xml

        <oauth2:authorization-server
            client-details-service-ref="clientDetailsService"
            user-approval-handler-ref="userApprovalHandler"
            token-services-ref="tokenServices">
            <oauth2:refresh-token />
            <oauth2:password />  <!-- (1) -->
        </oauth2:authorization-server>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``<oauth2:password />``\ タグを使用して、リソースオーナパスワードクレデンシャルグラントをサポートする。

|

.. warning::

    \ ``<oauth2:password />``\ タグと\ ``<oauth2:refresh-token />``\ タグは上記の順番で設定する必要がある。

    詳細は\ :ref:`OrderOfAuthoraizationServerSupportSetting`\を参照されたい。

|

クライアントの認証
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

リソースオーナパスワードクレデンシャルグラントでは\ :ref:`AuthorizationGrant`\で説明したとおりアクセストークンを直接発行する。
そのため、リソースオーナに対して認可の要求は行われず、ユーザエージェントのリダイレクトは発生しない。

認可コードグラントの\ :ref:`OAuthAuthorizationServerClientAuthentication`\で定義したテーブル、\ ``web_server_redirect_uris``\は不要となるため削除する。


トークンの取り消し（認可サーバ）
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

認可コードグラントの実装では、\ ``RevokeTokenServiceImpl.java``\にて認可サーバのトークンの取り消しに合わせて、認可情報の取り消しを行っている。

リソースオーナに対して認可の要求を行わないリソースオーナパスワードクレデンシャルグラントでは認可情報の取り消しが不要であるため、この処理を削除する。

以下に、認可情報の取り消しをコメントアウトした\ ``RevokeTokenServiceImpl.java``\を示す。

.. raw:: latex

   \newpage

* ``RevokeTokenServiceImpl.java``

.. code-block:: java

    @Service
    @Transactional
    public class RevokeTokenServiceImpl implements RevokeTokenService {

        @Inject
        ConsumerTokenServices consumerService;

        @Inject
        TokenStore tokenStore;

        @Inject
        ApprovalStore approvalStore;

        @Inject
        JodaTimeDateFactory dateFactory;

        public String revokeToken(String tokenValue, String clientId){

            OAuth2Authentication authentication = tokenStore.readAuthentication(tokenValue);
            if (authentication != null) {
                if (clientId.equals(authentication.getOAuth2Request().getClientId())) {
                    /* Authentication user = authentication.getUserAuthentication();
                    if (user != null) {
                        Collection<Approval> approvals = new ArrayList<>();
                        for (String scope : authentication.getOAuth2Request().getScope()) {
                            approvals.add(
                                    new Approval(user.getName(), clientId, scope, dateFactory.newDate(), ApprovalStatus.APPROVED));
                        }
                        approvalStore.revokeApprovals(approvals);
                    } */
                    consumerService.revokeToken(tokenValue);
                    return "success";

                } else {
                    return "invalid client";
                }
            } else {
                return "invalid token";
            }
        }
    }

|

.. _ImplementationOAuthResourceServerOfPassword:

リソースサーバの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

認可コードグラントの実装から変更は不要である。

実装は\ :ref:`ImplementationOAuthResourceServerOfAutorizationCode`\を参照し作成されたい。

.. _ImplementationOAuthClientServerOfPassword:

クライアントの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

認可コードグラントの実装から\ ``oauth2-client.xml``\を変更する必要がある。

\ ``oauth2-client.xml``\以外の実装は\ :ref:`ImplementationOAuthClientServerOfAutorizationCode`\を参照し作成されたい。


.. _OAuth2ResourceOwnerPasswordCredentialGrantResourceSettings:

OAuth2RestTemplateの設定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| リソースオーナパスワードクレデンシャルグラントでは、クライアントがリソースオーナのユーザ名およびパスワードを使用してアクセストークンの発行を依頼する。
| \ ``OAuth2RestTemplate``\にはリソースオーナのユーザ名およびパスワードをそれぞれパラメータとして設定する必要があるが、
  複数のリソースオーナが同じクライアントを利用する場合、リソースオーナ毎に設定内容を切り替える考慮が必要となる。
| ここでは、\ ``OAuth2RestTemplate``\のリソースをSessionスコープのBeanで設定し、そのBeanにリソースオーナの情報を格納することによって、
  リソースオーナ毎の設定内容の切り替えを実現する方法を説明する。
| リソースとして通常使用する\ ``<oauth2:resource>``\ タグはSingletonスコープのBeanとなるため、Sessionスコープに変更する場合は独自にBean定義する必要がある。


\ ``OAuth2RestTemplate``\の設定例を以下に示す。

リソースオーナ毎に設定内容を切り替えられるよう、\ ``org.springframework.security.oauth2.client.token.grant.password.ResourceOwnerPasswordResourceDetails``\ をSessionスコープで定義し、\ ``OAuth2RestTemplate``\への設定を行う。

* ``oauth2-client.xml``

.. code-block:: xml

    <bean id="todoPasswordGrantResource" class="org.springframework.security.oauth2.client.token.grant.password.ResourceOwnerPasswordResourceDetails"
        scope="session">
        <aop:scoped-proxy />
        <property name="clientId" value="firstSec" />
        <property name="clientSecret" value="firstSecSecret" />
        <property name="accessTokenUri" value="${auth.serverUrl}/oauth/token" />
        <property name="scope">
            <list>
                <value>READ</value>
                <value>WRITE</value>
            </list>
        </property>
    </bean> <!-- (1) -->

    <oauth2:rest-template id="todoPasswordGrantResourceRestTemplate" resource="todoPasswordGrantResource" /> <!-- (2) -->



.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``OAuth2RestTemplate``\が参照する、アクセス対象となるリソースに関する詳細情報を定義する。
        | 各項目の設定値については下記表を参照のこと。
    * - | (2)
      - | \ ``OAuth2RestTemplate``\を定義する。
        | \ ``id``\には\ ``OAuth2RestTemplate``\ のBeanIDを指定する。
        | \ ``resource``\には(1)で定義したBeanの\ ``id``\ を指定する。


|

     .. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
     .. list-table::
         :header-rows: 1
         :widths: 35 65
         :class: longtable

         * - 項目
           - 説明
         * - | \ ``class``\
           - | \ ``OAuth2RestTemplate``\のリソースとするBeanを指定する。ここでは\ ``ResourceOwnerPasswordResourceDetails``\を指定する。
         * - | \ ``scope``\
           - | sessionを指定し、スコープ範囲をHTTPSessionとする。
         * - | \ ``<aop:scoped-proxy />``\
           - | SessionスコープのBeanをSingletonのBeanである\ ``OAuth2RestTemplate``\ にインジェクションするため設定する。
             | これは、SessionスコープのBeanよりSingletonのBeanの方がライフサイクルが長いため必要になる設定である。
             | このタグを使用するために\ ``aop``\のネームスペースとスキーマを追加している。
         * - | \ ``clientId``\プロパティ
           - |  Beanの\ ``clientId``\に対して認可サーバにてクライントを識別するIDを設定する。
         * - | \ ``clientSecret``\プロパティ
           - | Beanの\ ``clientSecret``\に対して認可サーバにてクライアントの認証に用いるパスワードを設定する。
         * - | \ ``accessTokenUri``\プロパティ
           - | アクセストークンの発行を依頼するための認可サーバのエンドポイントを指定する。
         * - | \ ``scope``\プロパティ
           - | Beanの\ ``scope``\に対して認可を要求するスコープの一覧を設定する。
             | \ ``<oauth2:resource>``\タグを使用する場合とは異なり、\ ``scope``\をリスト形式で指定する。

|

.. note::

    Spring Security OAuth以外のアーキテクチャで実装されている認可サーバに対してアクセストークンの発行を依頼する場合は
    リクエストパラメータが上記とは異なる可能性があるため注意されたい。
    その場合はアーキテクチャの仕様を確認し、必要なリクエストパラメータを設定されたい。

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

.. _OAuthClientErrorHandlingWithPassword:

エラーハンドリング
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

リソースオーナパスワードクレデンシャルグラントにおける認可サーバ、リソースサーバへのアクセス時に発生するエラーのハンドリング方法について説明する。

.. _OAuthClientHandleErrorWhenAccessingTokenEndpointWithPassword:

トークンエンドポイント及びリソースサーバアクセス時のエラーハンドリング
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

リソースサーバで発生するエラーでハンドリングする必要のあるエラー及びエラーハンドリングについては認可コードグラントと同様であるため\ :ref:`OAuthClientHandleErrorWhenAccessingTokenEndpointWithCode`\を参照されたい。

トークンエンドポイントで発生する例外は、クライアントの\ ``OAuth2RestTemplate``\ で\ ``org.springframework.security.oauth2.client.resource.OAuth2AccessDeniedException``\ にラップされる。
\ ``OAuth2AccessDeniedException``\ でラップされるエラーでハンドリングが必要なものは以下となる。

|

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
.. list-table:: **トークンエンドポイントで発生するエラー**
    :header-rows: 1
    :widths: 10 20 70
    :class: longtable

    * - 項番
      - 発生するエラー
      - 説明
    * - | (1)
      - | リソースオーナ認証エラー
        | \ ``org.springframework.security.oauth2.common.exceptions.InvalidGrantException``\ 
      - | リソースオーナパスワードクレデンシャルグラント使用時に提示したリソースオーナの認証情報に誤りがある場合に発生する。
        | エラー発生時には例外クラスとして\ ``InvalidGrantException``\ が認可サーバでハンドリングされる。

|

上記の\ ``OAuth2RestTemplate``\ によってスローされる例外に対するハンドリング処理を実装する必要がある。
以下に実装例を示す。
実装例で使用している\ ``@ControllerAdvice``\ アノテーションについては \ :ref:`application_layer_controller_advice`\ を参照されたい。

.. code-block:: java

    @ControllerAdvice
    public class OAuth2ExceptionHandler {

        // omitted

        @ExceptionHandler(OAuth2AccessDeniedException.class) // (1)
        public String handleOAuth2AccessDeniedException(OAuth2AccessDeniedException e,
                Model model) {

            // omitted

            // (2)
            Throwable cause = e.getCause();
            if (cause instanceof InvalidGrantException) {
                return handleInvalidGrantException(((InvalidGrantException) cause),
                        model);
            }

            model.addAttribute("exception", e);

            return "common/error/systemError";
        }

        private String handleInvalidGrantException(InvalidGrantException e,
                Model model) {
            model.addAttribute("exception", e);

            return "common/error/accessDeniedError";
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
      - | コントローラで発生したエラーをハンドリングする。
        | \ ``@ExceptionHandler``\ のパラメータに指定されている\ ``OAuth2AccessDeniedException``\ が発生した場合に実行される。
    * - | (2)
      - | 発生したエラーの原因により遷移先画面を決定する。
        | エラーの原因が\ ``InvalidGrantException``\ の場合はアクセス拒否画面に遷移する。
        | 上記以外の場合はシステムエラー画面に遷移する。


.. note::

    上記では上記ではリソースオーナの操作により発生するエラーのハンドリングのみ紹介しているが、システムエラーもハンドリングする必要がある。
    発生するエラーについては \ :ref:`OAuthAppendixOccuringErrors`\ を参照されたい。


|

.. _ClientCredentialsGrant:

クライアントクレデンシャルグラントの実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

クライアントクレデンシャルグラントを利用した認可サーバ、リソースサーバ、クライアントの実装方法について説明する。

解説は認可コードグラントからの変更点のみを対象に行っている。変更点以外の実装、解説については\ :ref:`ImplementationAutorizationCodeGrant`\を参照されたい。

以下に認可コードグラントの実装から各サーバを実装する際の変更点を示す。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :stub-columns: 1
    :widths: 30 70
    :class: longtable

    * - サーバ
      - 認可コードグラントからの変更点
    * - 認可サーバ
      - | \ :ref:`OAuthAuthorizationServerDefinition`\
        | \ :ref:`OAuthAuthorizationServerClientAuthentication`\
        | \ :ref:`OAuthAuthorizationServerHowToCancelToken`\
    * - リソースサーバ
      - \ :ref:`OAuthResourceServerGetPrincipal`\
    * - クライアント
      - | \ :ref:`OAuth2RestTemplateSettings`\
        | \ :ref:`OAuthAccessOfResourceServer`\

|

.. _ImplementationOAuthAuthorizationServerOfClientCredentials:

認可サーバの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

認可コードグラントの実装から\ ``oauth2-auth.xml``\を変更し、DB及びServiceクラスから不要な定義を削除する必要がある。

\ ``oauth2-auth.xml``\以外の実装は\ :ref:`ImplementationOAuthAuthorizationServerOfAutorizationCode`\を参照し作成されたい。


認可サーバの定義
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

認可コードグラントからの\ ``oauth2-auth.xml``\の変更点を以下に示す。

* ``oauth2-auth.xml``

.. code-block:: xml

        <oauth2:authorization-server
            client-details-service-ref="clientDetailsService"
            token-endpoint-url="/oauth/token"
            token-services-ref="tokenServices">
            <oauth2:refresh-token />
            <oauth2:client-credentials />  <!-- (1) -->
        </oauth2:authorization-server>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``<oauth2:client-credentials />``\ タグを使用して、クライアントクレデンシャルグラントをサポートする。

|

.. warning::

    \ ``<oauth2:client-credentials />``\ タグと\ ``<oauth2:refresh-token />``\ タグは上記の順番で設定する必要がある。

    詳細は\ :ref:`OrderOfAuthoraizationServerSupportSetting`\を参照されたい。

|

クライアントの認証
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

クライアントクレデンシャルグラントでは\ :ref:`AuthorizationGrant`\で説明したとおりアクセストークンを直接発行する。
そのため、リソースオーナに対して認可の要求は行われず、ユーザエージェントのリダイレクトは発生しない。

認可コードグラントの\ :ref:`OAuthAuthorizationServerClientAuthentication`\で定義したテーブル、\ ``web_server_redirect_uris``\は不要となるため削除する。


トークンの取り消し（認可サーバ）
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

認可コードグラントの実装では、\ ``RevokeTokenServiceImpl.java``\にて認可サーバのトークンの取り消しに合わせて、認可情報の取り消しを行っている。

リソースオーナに対して認可の要求を行わないクライアントクレデンシャルグラントでは認可情報の取り消しが不要であるため、この処理を削除する。

以下に、認可情報の取り消しをコメントアウトした\ ``RevokeTokenServiceImpl.java``\を示す。

.. raw:: latex

   \newpage

* ``RevokeTokenServiceImpl.java``

.. code-block:: java

    @Service
    @Transactional
    public class RevokeTokenServiceImpl implements RevokeTokenService {

        @Inject
        ConsumerTokenServices consumerService;

        @Inject
        TokenStore tokenStore;

        @Inject
        ApprovalStore approvalStore;

        @Inject
        JodaTimeDateFactory dateFactory;

        public String revokeToken(String tokenValue, String clientId){

            OAuth2Authentication authentication = tokenStore.readAuthentication(tokenValue);
            if (authentication != null) {
                if (clientId.equals(authentication.getOAuth2Request().getClientId())) {
                    /* Authentication user = authentication.getUserAuthentication();
                    if (user != null) {
                        Collection<Approval> approvals = new ArrayList<>();
                        for (String scope : authentication.getOAuth2Request().getScope()) {
                            approvals.add(
                                    new Approval(user.getName(), clientId, scope, dateFactory.newDate(), ApprovalStatus.APPROVED));
                        }
                        approvalStore.revokeApprovals(approvals);
                    } */
                    consumerService.revokeToken(tokenValue);
                    return "success";

                } else {
                    return "invalid client";
                }
            } else {
                return "invalid token";
            }
        }
    }

|

.. _ImplementationOAuthResourceServerOfClientCredentials:

リソースサーバの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

認可コードグラントの実装から\ :ref:`OAuthResourceServerGetPrincipal`\について変更する必要がある。

\ :ref:`OAuthResourceServerGetPrincipal`\以外の実装は\ :ref:`ImplementationOAuthResourceServerOfAutorizationCode`\を参照されたい。

ユーザ情報の取得
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

ユーザの認証が行われないクライアントクレデンシャルグラントではリソースオーナの情報がSpring Securityに保持されない。

このため認可コードグラントでの実装のように、Controllerクラスの引数として\ ``@AuthenticationPrincipal``\ アノテーションを付与した\ ``UserDetails``\ や\ ``OAuth2Authentication``\ を指定してもリソースオーナの情報を受け取ることはできない。

代わりに、クライアント情報を保持しているため、Controllerのメソッド引数に\ ``String``\ を指定し\ ``@AuthenticationPrincipal``\ アノテーションを付与することによりクライアントIDを取得することができる。

認可コードグラントからの変更点を以下に示す。

.. code-block:: java

    @RestController
    @RequestMapping("api")
    public class TodoRestController {

        // omitted

        @RequestMapping(value = "todos", method = RequestMethod.GET)
        @ResponseStatus(HttpStatus.OK)
        public Collection<Todo> list(@AuthenticationPrincipal String clientId) { // (1)

            // omitted

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
      - | \ 引数 \ ``clientId``\にクライアントIDが格納される。

|

.. _ImplementationOAuthClientServerOfClientCredentials:

クライアントの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

認可コードグラントの実装から\ ``oauth2-client.xml``\を変更する必要がある。

\ ``oauth2-client.xml``\以外の実装は\ :ref:`ImplementationOAuthClientServerOfAutorizationCode`\を参照し作成されたい。

OAuth2RestTemplateの設定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

\ ``OAuth2RestTemplate``\の設定例を以下に示す。

* ``oauth2-client.xml``

.. code-block:: xml

    <oauth2:resource id="todoClientGrantResource" client-id="firstSecClient"
                    client-secret="firstSecSecret"
                    type="client_credentials"
                    access-token-uri="${auth.serverUrl}/oauth/token" /> <!-- (1) -->

    <oauth2:rest-template id="todoClientGrantResourceRestTemplate" resource="todoClientGrantResource" />


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``type``\属性にはグラントタイプを指定する。クライアントクレデンシャルグラントの場合\ ``client_credentials``\を指定する。

|

.. note::

    Spring Security OAuth以外のアーキテクチャで実装されている認可サーバに対してアクセストークンの発行を依頼する場合は
    リクエストパラメータが上記とは異なる可能性があるため注意されたい。
    その場合はアーキテクチャの仕様を確認し、必要なリクエストパラメータを設定されたい。

|

.. _OAuthClientErrorHandlingWithClient:

エラーハンドリング
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

.. _OAuthClientHandleErrorWhenAccessingTokenEndpointWithClient:

トークンエンドポイント及びリソースサーバアクセス時のエラーハンドリング
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

クライアントクレデンシャルグラントではトークンエンドポイントおよびリソースサーバで発生するエラーはすべてシステムエラーとして扱えば良い。
発生するエラーについては \ :ref:`OAuthAppendixOccuringErrors`\ を参照されたい。

リソースオーナの認証
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

クライアントクレデンシャルグラントではクライアントがリソースオーナとなるため、\ :ref:`OAuthAuthorizationServerResourceOwnerAuthentication`\で説明したようなリソースオーナの認証を必要としない。

|

.. _OAuthHowToExtend:

How to extend
--------------------------------------------------------------------------------

.. _OrderOfAuthoraizationServerSupportSetting:

認可サーバで複数のグラントタイプをサポートする場合
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ :ref:`AuthorizationGrant`\ で紹介したように認可サーバは複数のグラントタイプをサポートすることができる。

グラントタイプを複数指定する場合、タグの順番はXMLスキーマで定められているため以下の表の番号順に設定する必要がある。

番号順に設定を行わないと、アプリケーション起動時にXML解釈の失敗により\ ``org.springframework.beans.factory.xml.XmlBeanDefinitionStoreException``\が発生する。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
    :header-rows: 1
    :stub-columns: 1
    :widths: 20 80
    :class: longtable

    * - 順番
      - タグ
    * - 1
      - \ `<oauth2:authorization-code />`\
    * - 2
      - \ `<oauth2:implicit />`\
    * - 3
      - \ `<oauth2:refresh-token />`\
    * - 4
      - \ `<oauth2:client-credentials />`\
    * - 5
      - \ `<oauth2:password />`\

|

例として、認可コードグラント、リソースオーナパスワードクレデンシャルグラントおよびリフレッシュトークンをサポートする場合の設定例を以下に示す。

* ``oauth2-auth.xml``

.. code-block:: xml

        <oauth2:authorization-server>
            <oauth2:authorization-code />  <!-- (1) -->
            <oauth2:refresh-token />  <!-- (2) -->
            <oauth2:password />  <!-- (3) -->
        </oauth2:authorization-server>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ `<oauth2:authorization-code />`\タグを使用して、認可コードグラントをサポートする。
    * - | (2)
      - | \ `<oauth2:refresh-token />`\タグを使用して、リフレッシュトークンをサポートする。
    * - | (3)
      - | \ `<oauth2:password />`\タグを使用して、リソースオーナパスワードクレデンシャルグラントをサポートする。


.. _OAuthAuthorizationServerHowToCooperateWithHttp:

HTTPアクセスを介した認可サーバとリソースサーバの連携
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

リソースサーバと認可サーバは、認可サーバのチェックトークンエンドポイントにリソースサーバからHTTPアクセスを行うことで連携が可能である。

チェックトークンエンドポイントは、リソースサーバからアクセストークンの値を受け取り、リソースサーバの代わりにトークンの検証を行うエンドポイントである。
チェックトークンエンドポイントは\ ``TokenServices``\を用いてアクセストークンの取得、検証を行い、検証に問題がなければアクセストークンに紐づく情報をリソースサーバに渡す。

なお、チェックトークンエンドポイントはRFCに定義されていないSpring Security OAuth独自の機能であるが、
本ガイドラインでは、RFCに定義されている他のエンドポイントと同様の形式でレスポンスを行うように設定する。

リソースサーバが認可サーバのチェックトークンエンドポイントにアクセスするためには\ ``TokenServices``\の実装クラスである\ ``RemoteTokenServices``\ を使用する。
\ ``RemoteTokenServices``\ は\ ``RestTemplate``\ を用いてチェックトークンエンドポイントへHTTPアクセスを行い、アクセストークンに紐づく情報を取得する。
アクセストークンの検証はチェックトークンエンドポイントが行うため、\ ``RemoteTokenServices``\ では行わない。

以下に、実装例を示す。

認可サーバの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

まず、認可サーバにトークンを検証するための\ ``org.springframework.security.oauth2.provider.endpoint.CheckTokenEndpoint``\ クラスをコンポーネントとして登録する設定を行う。

* ``oauth2-auth.xml``

.. code-block:: xml

        <sec:http pattern="/oauth/*token*/**"
            authentication-manager-ref="clientAuthenticationManager">  <!-- (1) -->
            <sec:http-basic entry-point-ref="oauthAuthenticationEntryPoint" />
            <sec:csrf disabled="true"/>
            <sec:intercept-url pattern="/**" access="isAuthenticated()"/>
        </sec:http>

        <oauth2:authorization-server
             client-details-service-ref="clientDetailsService"
             user-approval-handler-ref="userApprovalHandler"
             token-services-ref="tokenServices"
             check-token-enabled="true"
             check-token-endpoint-url="/oauth/check-token">  <!-- (2) -->
            <oauth2:authorization-code />
            <oauth2:implicit />
            <oauth2:refresh-token />
            <oauth2:client-credentials />
            <oauth2:password />
        </oauth2:authorization-server>

        <!-- omitted -->


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | チェックトークンエンドポイントへのセキュリティ設定を行うために、エンドポイントとして
          \ ``/oauth/*token*/``\ 配下をアクセス制御の対象として指定する。
    * - | (2)
      - | \ ``<oauth2:authorization-server>``\ タグの\ ``check-token-enabled``\ 属性に\ ``true``\ を指定することで\ ``CheckTokenEndpoint``\ がコンポーネントとして登録される。
        | チェックトークンエンドポイントとして、\ ``/oauth/check_token``\ が設定される。


.. warning:: **チェックトークンエンドポイントのセキュリティ対策**

    認可サーバのチェックトークンエンドポイントは、リソースサーバのみアクセス可能とし、リソースサーバ以外が利用できないように制限する必要がある。
    本ガイドラインではチェックトークンエンドポイントに対してBasic認証にてアクセス制限を行っているが、可能であればネットワークにて特定URLに対するIP制限など、より上位のアクセス制限を行うことを推奨する。


.. note::

    \ ``TokenServices``\ は、共有DBを介して連携させる場合と同様に\ ``DefaultTokenServices``\ を使用する。
    \ ``TokenServices``\が参照する\ ``TokenStore``\はアプリケーションの要件に合ったインタフェースの実装クラスを使用する。

|

リソースサーバの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

リソースサーバの設定ファイルに\ ``TokenServices``\ として\ ``RemoteTokenServices``\ を使用する設定を行う。

* ``oauth2-resource.xml``

.. code-block:: xml

        <bean id="tokenServices"
            class="org.springframework.security.oauth2.provider.token.RemoteTokenServices">
            <property name="checkTokenEndpointUrl" value="${auth.serverUrl}/oauth/check_token" />  <!-- (1) -->
        </bean>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | 認可サーバのチェックトークンエンドポイントにアクセスしてアクセストークンに紐付く情報を取得できるよう、\ ``RemoteTokenServices``\ をBean定義する。
        | チェックトークンエンドポイントにアクセスするためのURLを\ ``checkTokenEndpointUrl``\ プロパティに設定する。
    * - | (2)
      - | \ ``clientId``\ プロパティに、認可サーバにてリソースサーバを識別するIDを設定する。
        | 設定したIDはチェックトークンエンドポイントにアクセスする際にBasic認証のユーザ名として使用される。
        | 設定するクライアントIDは、認可サーバのクライアント情報に登録しておく必要がある。クライアント情報の登録については\ :ref:`OAuthAuthorizationServerClientAuthentication`\ を参照されたい。
    * - | (3)
      - | \ ``clientSecret``\ プロパティに、認可サーバにてリソースサーバの認証に用いるパスワードを設定する。
        | 設定したパスワードは、チェックトークンエンドポイントにアクセスする際にBasic認証のパスワードとして使用される。
        | 設定するパスワードは、認可サーバのクライアント情報に登録されているパスワードと一致する必要がある。

|


.. note::

    チェックトークンエンドポイントでアクセストークンの検証エラーが発生した場合、\ ``RemoteTokenServices``\ にHTTPステータスコード400(Bad Request)が返却される。
    \ ``RestTemplate``\ のデフォルト実装ではHTTPステータスコード400(Bad Request)が返却された場合、エラーハンドリングを行い\ ``HttpClientErrorException``\ を発生させる。
    \ ``RemoteTokenServices``\ がデフォルトで使用する\ ``RestTemplate``\ はアクセストークンの検証エラーをクライアントサーバに連携するために、レスポンスのHTTPステータスコードが400の場合はエラーハンドリングしないよう拡張されている。
    \ ``RemoteTokenServices``\ に\ ``RestTemplate``\ をインジェクションした場合、この拡張が適用されなくなるため注意が必要である。

|


\ ``RemoteTokenServices``\ をリソースサーバで使用した場合、ハンドラメソッドで\ ``@AuthenticationPrincipal``\ アノテーションを\ ``String``\ に引数アノテーションとして指定することでリソースオーナのユーザ名が取得できる。

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
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ 引数 \ ``userName``\にリソースオーナのユーザ名が格納される。


|

.. _OAuthAuthorizationServerHowToGetOtherThanUserName:

リソースサーバへの独自項目連携方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

認可サーバとリソースサーバ間でDBを共有しない構成の場合でも、ユーザ情報に付随する項目を認可サーバからリソースサーバに連携したいというケースはありうるが、
リソースサーバの使用する\ ``TokenServices``\ として\ ``RemoteTokenServices``\ を使用する場合、リソースサーバのハンドラメソッド引数の\ ``@AuthenticationPrincipal``\ アノテーションではユーザ名以外の情報を取得することができない。

そこで、ここでは\ ``RemoteTokenServices``\ を使用してアクセストークンを連携するときに使用するクラスである\ ``org.springframework.security.oauth2.provider.token.DefaultAccessTokenConverter``\ を拡張し、
ユーザ名以外の情報をリソースサーバに連携する例を示す。

DefaultAccessTokenConverterとは
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

\ ``RemoteTokenServices``\ を使用したアクセストークンの連携では、\ ``RestTemplate``\ を使用してリソースサーバから認可サーバに対してアクセストークン値に紐づくリソースオーナ、クライアントの認証情報を要求し、結果を\ ``Map``\ として取得する。
このとき、\ ``DefaultAccessTokenConverter``\ は、認可サーバでは認証情報から\ ``Map``\ へ、リソースサーバでは\ ``Map``\ から認証情報へ変換するためのコンバーターとしての役割を持つ。

これを利用し、認可サーバからの返却値を\ ``Map``\ に追加するよう\ ``DefaultAccessTokenConverter``\ の拡張を行うことで、認可サーバ、リソースサーバ間で連携するパラメータをカスタマイズすることが出来るようになる。

以下の説明では、認可サーバ側で\ ``DefaultAccessTokenConverter``\ と、そのプロパティである\ ``org.springframework.security.oauth2.provider.token.DefaultUserAuthenticationConverter``\ をそれぞれカスタマイズすることで、ユーザ情報に関連した独自項目と、それ以外の独自項目を連携する例を示す。


認可サーバの実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

認可サーバ側の実装方法について説明する。

まず、ユーザ情報に関連した独自項目を追加するため、\ ``DefaultUserAuthenticationConverter``\ を拡張する。

* ``CustomUserAuthenticationConverter.java``

.. code-block:: java

    public class CustomUserAuthenticationConverter extends DefaultUserAuthenticationConverter {
        @Override
        public Map<String, ?> convertUserAuthentication(
                Authentication authentication) {
            Map<String, Object> response = new LinkedHashMap<>();
            response.put(USERNAME, authentication.getName());

            if (authentication.getAuthorities() != null &&
                    !authentication.getAuthorities().isEmpty()) {
                response.put(AUTHORITIES, AuthorityUtils.authorityListToSet(
                        authentication.getAuthorities()));
            }
            response.put("company_id", "COMZZZ"); // (1)
            return response;
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
      - | リソースサーバに引き渡す情報を独自項目\ ``company_id``\ として定義し、\ ``response``\ に設定する。
        | \ ``response``\ に設定した情報は、チェックトークンエンドポイントのトークン検証時にレスポンスBODYとしてJSON形式でリソースサーバへ返却される。


次に、ユーザ情報以外の独自項目を追加するため、\ ``DefaultAccessTokenConverter``\ を拡張する。

* ``CustomAccessTokenConverter.java``

.. code-block:: java

        public class CustomAccessTokenConverter extends DefaultAccessTokenConverter {

            @Override
            public Map<String, ?> convertAccessToken(OAuth2AccessToken token, OAuth2Authentication authentication) {

                @SuppressWarnings("unchecked")
                Map<String, Object> response = (Map<String, Object>) super.convertAccessToken(token, authentication);
                response.put("business_id","BIDXXX"); // (1)
                // omitted

                return response;
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
      - | リソースサーバに引き渡す情報を独自項目\ ``business_id``\ として定義し、\ ``response``\ に設定する。
        | \ ``response``\ に設定した情報は、チェックトークンエンドポイントのトークン検証時にレスポンスBODYとしてJSON形式でリソースサーバへ返却される。

認可サーバの設定ファイルに、作成した\ ``CustomUserAuthenticationConverter``\ 、\ ``CustomAccessTokenConverter``\ の設定を行う。

* ``oauth2-auth.xml``

.. code-block:: xml

        <oauth2:authorization-server
             client-details-service-ref="clientDetailsService"
             user-approval-handler-ref="userApprovalHandler"
             token-services-ref="tokenServices">  <!-- (1) -->
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
            class="com.example.oauth2.auth.converter.CustomAccessTokenConverter">  <!-- (3) -->
            <property name="userTokenConverter">
                <bean
                    class="com.example.oauth2.auth.converter.CustomUserAuthenticationConverter" />
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
      - | \ ``CheckTokenEndpoint``\ のBean定義を(2)で独自で行っているため、\ ``<oauth2:authorization-server>タグ``\ の\ ``check-token-enabled``\ 属性は指定しない。
        | トークンチェックエンドポイントとして、\ ``/oauth/check_token``\ が設定される。
    * - | (2)
      - | \ ``CheckTokenEndpoint``\ をBean定義する。
        | \ ``accessTokenConverter``\ プロパティに(3)で定義している\ ``CustomAccessTokenConverter``\ のBeanを指定することで\ ``CustomAccessTokenConverter``\ と\ ``CustomUserAuthenticationConverter``\ に追加した独自項目をリソースサーバに連携するようになる。
    * - | (3)
      - | \ ``DefaultAccessTokenConverter``\ を拡張した\ ``CustomAccessTokenConverter``\ をBean定義する。
        | \ ``userTokenConverter``\ プロパティに\ ``DefaultUserAuthenticationConverter``\ を拡張した\ ``CustomUserAuthenticationConverter``\ のBeanを指定する。


リソースサーバの実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

リソースサーバに、認可サーバから連携された情報をハンドラメソッド引数の\ ``@AuthenticationPrincipal``\ アノテーションで取得できるよう機能の追加を行う。
まず、\ ``@AuthenticationPrincipal``\ アノテーションで取得する情報を保持する\ ``OauthUser``\ クラスを作成する。

* ``ResourceOwner.java``

.. code-block:: java

        public class OauthUser implements Serializable{

            private static final long serialVersionUID = 1L;

            private String username;

            private String companyId;

            private String businessId;

            private String clientId;

            // omitted

            public OauthUser(String username, String companyId, String businessId, String clientId){
                this.username = username;
                this.companyId = companyId;
                this.businessId = businessId;
                this.clientId = clientId;
            }

            // Getters and Setters are omitted

        }

\ ``@AuthenticationPrincipal``\ アノテーションでユーザ情報が取得できるように設定を行う\ ``org.springframework.security.oauth2.provider.token.DefaultAccessTokenConverter``\ を拡張し、ユーザ名以外の情報も取得できるよう機能の追加を行う。

* ``CustomUserAuthenticationConverter.java``

.. code-block:: java

        public class CustomUserAuthenticationConverter extends DefaultUserAuthenticationConverter{

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
                            (String) map.get("company_id"),
                            (String) map.get("business_id"),
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
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``DefaultUserAuthenticationConverter``\ に実装されている\ ``getAuthorities``\ メソッドがprivateで定義されているため、\ ``getAuthorities``\ メソッドで使用される\ ``defaultAuthorities``\ と\ ``getAuthorities``\ メソッドを実装する。
    * - | (2)
      - | 認可サーバから連携された情報から認証情報を抽出するメソッド。
    * - | (3)
      - | 認可サーバから連携された情報を\ ``OauthUser``\ クラスに設定する。
    * - | (4)
      - | \ ``UsernamePasswordAuthenticationToken``\ の第一引数に\ ``OauthUser``\ を設定することで、認可サーバから連携された情報を\ ``@AuthenticationPrincipal``\ アノテーションで取得できるようになる。



リソースサーバの設定ファイルに、\ ``CustomUserAuthenticationConverter``\ の設定を行う。

* ``oauth2-resource.xml``

.. code-block:: xml

        <bean id="tokenServices"
            class="org.springframework.security.oauth2.provider.token.RemoteTokenServices">
            <property name="checkTokenEndpointUrl" value="${auth.serverUrl}/oauth/check_token" />
            <property name="accessTokenConverter" ref="accessTokenConverter" />
        </bean>

        <bean id="accessTokenConverter"
            class="org.springframework.security.oauth2.provider.token.DefaultAccessTokenConverter">
            <property name="userTokenConverter">
                <bean class="com.example.oauth2.resource.converter.CustomUserAuthenticationConverter"/>  <!-- (1) -->
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
      - | \ ``DefaultAccessTokenConverter``\ をBean定義する。ユーザ名以外の情報を\ ``@AuthenticationPrincipal``\ アノテーションで取得できるようにするため、\ ``userTokenConverter``\ プロパティに\ ``CustomUserAuthenticationConverter``\ クラスを指定する。



認可サーバにおけるパスのカスタマイズ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

認可サーバではエンドポイントと、特定の状況が起きたときのフォワード先についてパスを変更することができる。
本節では、認可サーバにおけるパス、および関連箇所の設定変更方法を説明する。

.. _CustomizableUrl:

カスタマイズ可能なパス
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

How to useにて説明したとおり、認可サーバでは\ ``<oauth2:authorization-server>``\タグによる定義を行うことで、
RFCに準拠したエンドポイントや、認可サーバ内でフォワードされたリクエストを処理するためのControllerがコンポーネントとして登録される。
また、認可サーバがクライアントやリソースサーバに公開するエンドポイントは、<oauth2:authorization-server>タグの
属性値を変更することにより、カスタマイズが可能である。

以下に、コンポーネントとして登録されるエンドポイント、およびフォワード先のデフォルトパスと、
エンドポイントのカスタマイズ時に変更する<oauth2:authorization-server>タグの属性値を示す。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|p{0.40\linewidth}|
.. list-table:: **エンドポイント**
    :header-rows: 1
    :widths: 20 20 20 40
    :class: longtable

    * - 名前
      - 属性値
      - デフォルトパス
      - 説明
    * - | 認可エンドポイント
      - | \ ``authorization-endpoint-url``\
      - | \ ``/oauth/authorized``\
      - | クライアントがリソースオーナから認可を得るために利用するエンドポイント。
    * - | トークンエンドポイント
      - | \ ``token-endpoint-url``\
      - | \ ``/oauth/token``\
      - | クライアントがアクセストークンを発行するために利用するエンドポイント。
    * - | チェックトークンエンドポイント
        | (\ :ref:`OAuthAuthorizationServerHowToCooperateWithHttp`\ の実装を行っている場合のみ設定する。)
      - | \ ``check-token-endpoint-url``\
      - | \ ``/oauth/check_token``\
      - | リソースサーバがアクセストークンを検証するために利用するエンドポイント。
        | HTTPアクセスを介してアクセストークンの連携を行う場合に利用する。なお、本エンドポイントは、
        | RFCの定めるエンドポイントではなく、Spring Security OAuthが独自に拡張したエンドポイントである。

|

.. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|p{0.40\linewidth}|
.. list-table:: **フォワード先**
    :header-rows: 1
    :widths: 20 20 20 40
    :class: longtable

    * - 名前
      - 属性値
      - デフォルトパス
      - 説明
    * - | 認可取得時のフォワード先
      - | \ ``user-approval-page``\
      - | \ ``/oauth/confirm_access``\
      - | リソースオーナに認可画面を返却するために利用するフォワード先。
        | 認可サーバの処理内で利用するパスであり、クライアントやリソースサーバには公開しない。
    * - | 不正クライアントエラー発生時のフォワード先
      - | \ ``error-page``\
      - | \ ``/oauth/error``\
      - | リソースオーナに認可エンドポイントにおけるエラーを通知するために利用するフォワード先。
        | 認可サーバの処理内で利用するパスであり、クライアントやリソースサーバには公開しない。

|

これらのパスは認可サーバ側の設定を変更することにより、カスタマイズが可能である。
具体的な設定方法を以降に示す。


.. _CustomizePath:

パスのカスタマイズ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
.. _CustomizeEndPoint:

エンドポイントのカスタマイズ
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

前述のとおり、各エンドポイントは、<oauth2:authorization-server>タグの属性値を変更することにより、カスタマイズが可能である。

エンドポイントを変更する際の実装例を以下に示す。

* ``oauth2-auth.xml``

.. code-block:: xml

        <oauth2:authorization-server
            client-details-service-ref="clientDetailsService"
            user-approval-handler-ref="userApprovalHandler"
            token-services-ref="tokenServices"
            token-endpoint-url="/api/token"
            authorization-endpoint-url="/api/authorize"
            check-token-endpoint-url="/api/check_token"
            check-token-enabled="true">  <!-- (1) -->
            <!-- omitted -->
        </oauth2:authorization-server>

        <sec:http pattern="/api/*token*/**"
            authentication-manager-ref="clientAuthenticationManager" realm="Realm">  <!-- (2) -->
            <!-- omitted -->
        </sec:http>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``<oauth2:authorization-server>``\タグの変更したいエンドポイントに対する属性値（\ ``authorization-endpoint-url``\、
          \ ``token-endpoint-url``\、\ ``check-token-endpoint-url``\）にURLを設定する。
    * - | (2)
      - | アクセストークン操作に関するエンドポイントのURLを含むように、アクセス制御の対象を変更する。


|

変更したエンドポイントに合わせて、エンドポイントを参照する設定を変更する必要がある。
認可サーバでは、\ :ref:`OAuthAuthorizationServerResourceOwnerAuthentication`\の\ ``spring-security.xml``\と
\ :ref:`OAuthAuthorizationServerHowToCustomizeAuthorizeView`\の\ ``oauthConfirm.jsp``\のエンドポイント設定を参照されたい。

また、クライアントの実装を行っている場合は、認可リクエストの設定として認可サーバのエンドポイントを設定しているため、
\ :ref:`OAuth2RestTemplateSettings`\の\ ``oauth2-client.xml``\も参照されたい。

.. _CustomizeForward:

フォワード先のカスタマイズ
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

リソースオーナからの認可の取得時に、デフォルト設定では\ ``/oauth/confirm_access``\ にフォワードされる。
また、認可エンドポイントで不正クライアントエラーが発生した際には、デフォルト設定では\ ``/oauth/error``\ にフォワードされる。
これらのフォワード先は変更が可能である。

フォワード先を変更する際の実装例を以下に示す。

* ``oauth2-auth.xml``

.. code-block:: xml

        <oauth2:authorization-server
            client-details-service-ref="clientDetailsService"
            user-approval-handler-ref="userApprovalHandler"
            token-services-ref="tokenServices"
            error-page="forward:/api/error"
            user-approval-page="forward:/api/confirm_access">  <!-- (1) -->
            <!-- omitted -->
        </oauth2:authorization-server>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``<oauth2:authorization-server>``\タグの変更したいフォワード先に対する属性値（\ ``error-page``\または\ ``user-approval-page``\）にURLを設定する。
          設定するURLは先頭に\ ``forward:``\を付ける必要がある。

フォワード先を変更した際は対応するコントローラについても変更する必要があるため注意されたい。
本実装例では以下を参照されたい。

* \ :ref:`OAuthAuthorizationServerHowToCustomizeAuthorizeView`\の\ ``OAuth2ApprovalController.java``\
* \ :ref:`OAuthAuthorizationServerHowToHandleError`\の\ ``OAuth2ErrorController.java``\

| 

.. _OAuthAppendix:

Appendix
--------------------------------------------------------------------------------

.. _OAuthAppendixOccuringErrors:

クライアントから認可サーバ、リソースサーバアクセス時に発生するエラー
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

本ガイドラインではクライアントから認可サーバ、リソースサーバアクセス時に発生するエラーについて、リソースオーナの操作によって発生するエラーのハンドリング方法を記載している。
リソースオーナの操作起因以外のエラーも含め、発生するエラーについて以下に説明する。

認可エンドポイントで発生するエラー
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

認可エンドポイントで発生するエラーは不正クライアントエラーとそれ以外に分類され、不正クライアントエラーの場合は認可サーバでエラー画面にフォワードされることでエラー通知される。
不正クライアントエラーの詳細については\ :ref:`DefinitionOfBadClientError`\を参照されたい。
認可エンドポイントで発生する不正クライアントエラーについて以下で説明する。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
.. list-table:: **認可エンドポイントで発生する不正クライアントエラー**
    :header-rows: 1
    :widths: 10 20 70
    :class: longtable

    * - 項番
      - 発生するエラー
      - 説明
    * - | (1)
      - | クライアント未存在エラー
        | \ ``NoSuchClientException``\ 
      - | ユーザエージェントを認可サーバへアクセスさせたクライアントが、認可サーバの保持するクライアント情報に登録されていない場合に発生する。
    * - | (2)
      - | リダイレクトURI不一致エラー
        | \ ``org.springframework.security.oauth2.common.exceptions.RedirectMismatchException``\ 
      - | クライアントがパラメータとして認可サーバに渡したリダイレクトURIのホストとルートパスが、認可サーバに登録されているリダイレクトURIと一致しない場合に発生する。

| 

発生したエラーが不正クライアントエラー以外の場合はクライアントにリクエストパラメータとしてエラー情報（\ ``error``\ 、\ ``error_description``\ ）が通知され、\ ``OAuth2RestTemplate``\ にて例外に復元されスローされる。
認可エンドポイントで発生する不正クライアントエラー以外のエラーについて以下で説明する。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
.. list-table:: **認可エンドポイントで発生する不正クライアントエラー以外のエラー**
    :header-rows: 1
    :widths: 10 20 70
    :class: longtable

    * - 項番
      - 発生するエラー
      - 説明
    * - | (1)
      - | スコープ検証エラー
        | \ ``org.springframework.security.oauth2.common.exceptions.InvalidScopeException``\ 
      - | クライアントが指定したスコープが、認可サーバの保持するクライアント情報に存在しない場合に発生する。
    * - | (2)
      - | リクエスト検証エラー
        | \ ``org.springframework.security.oauth2.common.exceptions.InvalidRequestException``\ 
      - | 認可後の遷移先を示すリダイレクトURIが設定されていない場合や認可画面を表示せずに認可が行われた場合に発生する。
    * - | (3)
      - | レスポンスタイプエラー
        | \ ``org.springframework.security.oauth2.common.exceptions.UnsupportedResponseTypeException``\ 
      - | クライアントが指定した\ ``response_type``\ パラメータを認可サーバがサポートしない場合に発生する。
    * - | (4)
      - | グラントタイプエラー
        | \ ``InvalidGrantException``\ 
      - | クライアントが使用できるグラントタイプがない場合や、認可コードグラント、インプリシットグラント以外のリダイレクトURIを使用出来ないグラントタイプが指定された場合に発生する。
    * - | (5)
      - | リソースオーナによる認可拒否
        | \ ``UserDeniedAuthorizationException``\ 
      - | リソースオーナがスコープ認可画面で、クライアントが指定したスコープを全て拒否した場合に発生する。
    * - | (6)
      - | リソースオーナ未認証エラー
        | \ ``org.springframework.security.authentication.InsufficientAuthenticationException``\ 
      - | 認可エンドポイントへアクセス時にリソースオーナがユーザ認証を行っていない場合に発生する。認可サーバで適切なセキュリティ設定を行っていれば発生しない。

| 

トークンエンドポイントで発生するエラー
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

トークンエンドポイントでエラーが発生した場合、クライアントの\ ``OAuth2RestTemplate``\ で\ ``OAuth2AccessDeniedException``\ にラップされる。
トークンエンドポイントで発生するエラーについて以下で説明する。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
.. list-table:: **トークンエンドポイントで発生するエラー**
    :header-rows: 1
    :widths: 10 20 70
    :class: longtable

    * - 項番
      - 発生するエラー
      - 説明
    * - | (1)
      - | Basic認証エラー
        | \ ``HttpClientErrorException``\ 
      - | クライアントのBasic認証に失敗した場合に発生する。
    * - | (2)
      - | クライアント未認証エラー
        | \ ``InsufficientAuthenticationException``\ 
      - | トークンエンドポイントの処理前に認証が行われていない場合に発生する。認可サーバで適切なセキュリティ設定を行っていれば発生しない。
    * - | (3)
      - | スコープ検証エラー
        | \ ``InvalidScopeException``\ 
      - | クライアントが指定したスコープが、認可サーバの保持するクライアント情報に存在しない場合に発生する。
        | 認可コードグラント使用時は認可エンドポイントでエラーとなるため\ ``OAuth2RestTemplate``\ を使用していれば発生しない。
    * - | (4)
      - | リソースオーナ認証エラー
        | \ ``InvalidGrantException``\ 
      - | リソースオーナパスワードクレデンシャルグラント使用時に提示したリソースオーナの認証情報に誤りがある場合に発生する。
    * - | (5)
      - | 認可コード未存在エラー
        | \ ``InvalidGrantException``\ 
      - | 認可コードグラント使用時に、クライアントから認可サーバに渡した認可コードが認可サーバに存在しない場合に発生する。

本ガイドラインでは、クライアントとしてSpring Security OAuthの\ ``OAuth2RestTemplate``\ を利用する前提であり、
\ ``OAuth2RestTemplate``\ が自動的に管理している設定に関してエラーが発生しない構造となっている。
Spring Security OAuthを使用せずにクライアントを開発する場合のために、Spring Security OAuthの\ ``OAuth2RestTemplate``\ を使用しない場合に発生するエラーについて以下で説明する。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
.. list-table:: **【参考】トークンエンドポイントで発生するエラー（OAuth2RestTemplateを使用しない場合）**
    :header-rows: 1
    :widths: 10 20 70
    :class: longtable

    * - 項番
      - 発生するエラー
      - 説明
    * - | (1)
      - | クライアント検証エラー
        | \ ``org.springframework.security.oauth2.common.exceptions.InvalidClientException``\ 
      - | 認証済みクライアントとリクエストパラメータに設定されているクライアントが不一致の場合に発生する。
        | 具体的にはトークンエンドポイントで認証したクライアントのクライアントIDと以下のクライアントIDが一致しない場合に発生する。

        * 認可リクエストのリクエストパラメータに設定されているクライアントID
        * アクセストークンリクエストのリクエストパラメータに設定されているクライアントID
    * - | (2)
      - | グラントタイプエラー
        | \ ``org.springframework.security.oauth2.common.exceptions.UnsupportedGrantTypeException``\ 
      - | トークンエンドポイントへのリクエストに指定した\ ``grant_type``\ パラメータに誤りがある場合に発生する。

        * クライアントが指定したグラントタイプが認可サーバがサポートしていないグラントタイプの場合
        * リフレッシュトークンをサポートしていない認可サーバにリフレッシュトークンを使用した場合
    * - | (3)
      - | リクエスト検証エラー
        | \ ``InvalidRequestException``\ 
      - | トークンエンドポイントへのリクエストに\ ``grant_type``\パラメータが存在しない場合に発生する。
    * - | (4)
      - | リダイレクトURI検証エラー
        | \ ``RedirectMismatchException``\ 
      - | 認可エンドポイントアクセス時に使用したリダイレクトURIと、トークンエンドポイントアクセス時にパラメータに設定したリダイレクトURIが一致しない場合に発生する。

| 

リソースサーバで発生するエラー
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

リソースサーバでエラーが発生した場合、クライアントの\ ``OAuth2RestTemplate``\ の処理は発生したエラーによって異なる。
リソースサーバで発生するエラーについて以下で説明する。


.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
.. list-table:: **リソースサーバで発生するエラー**
    :header-rows: 1
    :widths: 10 20 70
    :class: longtable

    * - 項番
      - 発生するエラー
      - 説明
    * - | (1)
      - | スコープ検証エラー
        | \ ``org.springframework.security.oauth2.common.exceptions.InsufficientScopeException``\ 
      - | 保護されたリソースへのアクセスに必要なスコープが、アクセストークンが保持するスコープに存在しなかった場合に発生する。
    * - | (2)
      - | リソースID検証エラー
        | \ ``UserDeniedAuthorizationException``\ 
      - | 保護されたリソースへのアクセスに必要なリソースIDが、アクセストークンに紐付くクライアント情報のリソースIDに存在しなかった場合に発生する。
        | クライアントの\ ``OAuth2RestTemplate``\ で\ ``OAuth2AccessDeniedException``\ にラップされる。
    * - | (3)
      - | アクセストークン検証エラー
        | \ ``InvalidTokenException``\ 
      - | リソースサーバが受け取ったアクセストークンの正当性を確認出来ない場合（異常系）や、アクセストークンの有効期限が切れている場合（正常系）に発生する。
        |
        | アクセストークン検証エラーは正常系と異常系が混在しているため、インプリシットグラント等\ ``OAuth2RestTemplate``\ を利用しない場合にこのエラーを受け取った場合、アクセストークンの有効期限切れかどうか判定する必要がある。
        | 具体的な実装例については、\ :ref:`OAuthClientUsingJavaScript`\ を参照されたい。

        .. note::

            本ガイドラインにおいてインプリシットグラント以外のグラントタイプでは\ ``OAuth2RestTemplate``\ を利用している。
            \ ``OAuth2RestTemplate``\ は、リソースサーバへのアクセス前にアクセストークンの有効期限チェックを行い、有効期限が切れていた場合にはアクセストークンの再発行を行っているため、
            通常はリソースサーバでアクセストークンの有効期限切れは発生しない。
            \ ``OAuth2RestTemplate``\ はアクセストークン取得後すぐにリソースにアクセスする構造のため、アクセストークンの取得直後に\ ``OAuth2RestTemplate``\ では有効期限内にも関わらず、
            リソースサーバで有効期限切れととなる状態は、アクセストークンの発行に何らかの問題があることになる。この場合、\ ``OAuth2RestTemplate``\ で\ ``AccessTokenRequiredException``\ がスローされる。

    * - | (4)
      - | チェックトークンエンドポイントBasic認証エラー
        | \ ``HttpClientErrorException``\ 
      - | \ :ref:`OAuthAuthorizationServerHowToCooperateWithHttp`\ において、チェックトークンエンドポイントへアクセス時にBasic認証に失敗した場合に発生する。
        | クライアントでエラーハンドリングを行う場合は、リソースサーバからクライアントに適切なエラーを返却するようリソースサーバでエラーハンドリングを行う必要がある。


Spring Security OAuthの拡張について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

RFC 6749の \ `8. Extensibility <https://tools.ietf.org/html/rfc6749#section-8>`_\ にはOAuth 2.0の拡張仕様が定められいる。
OAuth 2.0による認可機能を提供するアプリケーションではこれに沿ってカスタマイズすることが可能である。

Spring Security OAuthでは、前述のRFC 6749に規定された拡張ポイントに対してどのようにサポートしているかは明示的に公表されていないが、
以下の拡張ポイントがサポートされていると考えられる。

* \ `8.1 Defining Access Token Types <https://tools.ietf.org/html/rfc6749#section-8.1>`_\
* \ `8.2 Defining New Endpoint Parameters <https://tools.ietf.org/html/rfc6749#section-8.2>`_\
* \ `8.3 Defining New Authorization Grant Types <https://tools.ietf.org/html/rfc6749#section-8.3>`_\

本節では、特に\ `8.2 Defining New Endpoint Parameters <https://tools.ietf.org/html/rfc6749#section-8.2>`_\
に関連するアクセストークンリクエストとそのレスポンスの拡張ポイントについて主要なコンポーネントと処理の流れを解説する。


アクセストークンリクエストとそのレスポンスの拡張
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
アクセストークンリクエストとそのレスポンスの拡張ポイントとして、
任意のパラメータやヘッダをリクエストやレスポンスに付与することができる。

ただし、認可リクエストには前述したような拡張ポイントが設けられていない。
そのため、例えば認可リクエストに独自のパラメータを追加したい場合には、
Spring Security OAuthのAPI自体を改修する必要があり、比較的大きな改修が必要となることに注意されたい。


.. _OAuthAppendixClient:

クライアント
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

クライアントでは、アクセストークンリクエストの拡張ポイントとして、
\ ``RequestEnhancer``\ インタフェースが用意されている。
以下に拡張ポイントに関連する主要なコンポーネント、およびその関連図を示す。


.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table:: **クライアントの主要なコンポーネント**
    :header-rows: 1
    :widths: 35 65
    :class: longtable

    * - クラス・インタフェース名
      - 説明
    * - | \ ``OAuth2RestTemplate``\
      - | \ ``RestTemplate``\ を拡張しOAuth 2.0向けの機能を追加したクラス。
        | グラントタイプに応じたアクセストークンの取得など、OAuth 2.0独自の機能を持つ。
    * - | \ ``OAuth2ProtectedResourceDetails``\
      - | リソースサーバが保持するリソースにアクセスするための詳細情報のインタフェース。
        | グラントタイプに応じたパラメータを保持するために派生クラスが提供されており、認可コードグラントの場合は\ ``AuthorizationCodeResourceDetails``\ が実装クラスとなる。
    * - | \ ``AccessTokenProvider``\
      - | 認可サーバよりアクセストークンを取得するための機能を提供するインタフェース。
        | グラントタイプに応じた派生クラスが提供されており、グラントタイプごとのプロバイダは必ず
          \ ``AccessTokenProvider``\ を実装して\ ``OAuth2AccessTokenSupport``\ を継承している。
        | 認可コードグラントの場合は\ ``AuthorizationCodeAccessTokenProvider``\ が実装クラスとなる。
    * - | \ ``OAuth2AccessTokenSupport``\
      - | 認可サーバよりアクセストークンを取得するための機能を提供する基底クラス。
        | グラントタイプ共通の処理として、認可サーバに対してアクセストークンリクエストを送信する機能を提供している。
        | アクセストークンリクエストの生成には、後述する\ ``ClientAuthenticationHandler``\ と\ ``RequestEnhancer``\ を使用している。
    * - | \ ``AuthorizationCodeAccessTokenProvider``\
      - | 認可コードグラントにてアクセストークンを取得するための機能を提供するクラス。
        | 認可コードグラント独自の処理として、認可リクエストを作成する機能を提供している。
    * - | \ ``ClientAuthenticationHandler``\
      - | \ ``OAuth2ProtectedResourceDetails``\ をもとにヘッダ、またはフォームにクライアント認証情報を設定するための
          機能を提供するインタフェース。\ ``DefaultClientAuthenticationHandler``\ がデフォルトの実装クラスとなる。
    * - | \ ``RequestEnhancer``\
      - | アクセストークンリクエストおよび\ ``OAuth2ProtectedResourceDetails``\ をもとにヘッダ、またはフォームに
          パラメータを設定するための機能を提供するインタフェース。
          \ ``DefaultRequestEnhancer``\ がデフォルトの実装クラスとなる。
        | **アクセストークンリクエストにおける拡張ポイントの一つ。**
        |
        | なお、リクエストパラメータに独自の項目を追加したい場合は、デフォルトで使用される\ ``DefaultRequestEnhancer``\の
          Bean定義を変更するだけで良く、独自に\ ``RequestEnhancer``\の実装クラスを作成する必要はない。
        |
        | 具体的には、以下のようにBean定義すれば良い。
        | 
        | (1) \ ``DefaultRequestEnhancer``\ 
        |     \ ``parameterIncludes``\ 属性に追加するパラメータのキー値を設定する
        | (2) \ ``AuthorizationCodeAccessTokenProvider``\
        |     \ ``tokenRequestEnhancer``\ 属性に(1)の\ ``DefaultRequestEnhancer``\を設定する
        | (3) \ ``OAuth2RestTemplate``\
        |     \ ``access-token-provider``\ 属性に(2)の\ ``AuthorizationCodeAccessTokenProvider``\ を設定する

|

例として、認可コードグラントにおける、クライアントからリソースにアクセスする時のフローを以下に示す。

.. figure:: ./images/OAuth_ClientAccessTokenRequest.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **クライアントの動き（アクセストークンリクエスト）**
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | リソースサーバが保持しているリソースにアクセスする際、\ ``OAuth2RestTemplate``\ の呼び出しが行われることでクライアント側の処理が開始される。
        | 認可コード発行までのクライアント側のフローについては、\ :ref:`Client`\ を参照されたい。
    * - | (2)
      - | 認可コード発行後、認可サーバのリダイレクトにより再度\ ``OAuth2RestTemplate``\ の呼び出しが行われる。
          このとき、認可コードはセッション(\ ``OAuth2ClientContext``\ )に保持される。
        | \ ``OAuth2RestTemplate``\ では、\ ``OAuth2ProtectedResourceDetails``\ に定義しているグラントタイプに応じて\ ``AccessTokenProvider``\ の呼び出しを行う。
        | \ ``AuthorizationCodeAccessTokenProvider``\ が\ ``AccessTokenProvider``\ の実装クラスとして呼び出される。
    * - | (3)
      - | \ ``AuthorizationCodeAccessTokenProvider``\ では、\ ``OAuth2AccessTokenSupport``\ が保持する\ ``ClientAuthenticationHandler``\ からはクライアント認証情報を、
          \ ``RequestEnhancer``\ からはリクエストパラメータをヘッダ、またはフォームに設定し、アクセストークンリクエストを作成する。
    * - | (4)
      - | \ ``OAuth2AccessTokenSupport``\は(3)にて作成したリクエストを使用し、認可サーバに対してアクセストークンリクエストを送信する。
    * - | (5)
      - | 認可サーバでは、クライアントを認証し認可グラントの正当性を確認する。
        | 認可グラントが正当な場合、アクセストークンを発行する。
        | アクセストークン発行のフローについては、\ :ref:`IssueOfAccessToken`\ を参照されたい。
    * - | (6)
      - | \ ``AccessTokenProvider``\は取得したアクセストークンを\ ``OAuth2RestTemplate``\ に返却する。
        | \ ``OAuth2RestTemplate``\では、セッション(\ ``OAuth2ClientContext``\ )にアクセストークンを保持し、それを用いてリソースサーバに対してリクエストを送信する。

|

.. _OAuthAppendixAuthorizationServer:

認可サーバ
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

認可サーバでは、アクセストークンレスポンスの拡張ポイントとして、
\ ``TokenEnhancer``\ インタフェースが用意されている。
以下に拡張ポイントに関連する主要なコンポーネント、およびその関連図を示す


.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table:: **認可サーバの主要なコンポーネント**
    :header-rows: 1
    :widths: 35 65
    :class: longtable

    * - クラス・インタフェース名
      - 説明
    * - | \ ``AbstractEndpoint``\
      - | 認可サーバにおいて、クライアントからのリクエストを受けるエンドポイントの共通的な機能を提供する基底クラス。
          \ ``TokenEndpoint``\が実装クラスとなる。
    * - | \ ``TokenEndpoint``\
      - | クライアントからのリクエストに対し、アクセストークンを発行するためのエンドポイントの機能を提供するクラス。
        | クライアントや、リクエストパラメータに含まれるパラメータを検証する機能を持つ。
    * - | \ ``ClientDetailsService``\
      - | OAuth 2.0機能を利用するクライアントを検証するための機能を提供するインタフェース。
        | クライアント情報を管理する媒体ごとに派生クラスが提供されており、\ ``JdbcClientDetailsService``\ 、\ ``InMemoryClientDetailsService``\ が実装クラスとなる。
    * - | \ ``TokenGranter``\
      - | アクセストークンを発行するための機能を提供するインタフェース。
        | グラントタイプに応じた派生クラスが提供されており、グラントタイプごとのプロバイダは必ず\ ``TokenGranter``\ を実装して\ ``AbstractTokenGranter``\ を継承している。
        | 認可コードグラントの場合は\ ``AuthorizationCodeTokenGranter``\ が実装クラスとなる。
    * - | \ ``AbstractTokenGranter``\
      - | アクセストークンを発行するための機能を提供する基底クラス。
        | リクエストパラメータからクライアント情報を取得し、グラントタイプの検証と固有処理の呼び出しを行う機能を持つ。
    * - | \ ``AuthorizationCodeTokenGranter``\
      - | 認可コードグラントにてアクセストークンを発行するための機能を提供するクラス。
        | 認可コードグラント独自の処理として、アクセストークンリクエストに含まれる認可コードやリダイレクトURIを検証する機能を提供している。
    * - | \ ``AuthorizationCodeServices``\
      - | 認可コードグラントにて認可コードの発行、および管理をするための機能を提供するインタフェース。
        | 認可コードを管理する媒体ごとに派生クラスが提供されており、\ ``JdbcAuthorizationServerTokenServices``\ 、\ ``InMemoryAuthorizationServerTokenServices``\ が実装クラスとなる。
    * - | \ ``AuthorizationServerTokenServices``\
      - | 認可サーバとして必要となる、アクセストークンやリフレッシュトークンを発行するための機能を提供するインタフェース。\ ``DefaultTokenServices``\ がデフォルトの実装クラスとなる。
    * - | \ ``TokenEnhancer``\
      - | \ ``AuthorizationServerTokenServices``\によって生成されたトークンに追加情報等を付与するための機能を提供するインタフェース。
        | 本インタフェースを拡張することでアクセストークンとして管理する情報、およびクライアントに返却する情報に独自パラメータを付与することが出来る。
        | **アクセストークンレスポンスにおける拡張ポイントの一つ。**
    * - | \ ``TokenStore``\
      - | アクセストークンを管理するための機能を提供するインタフェース。
        | アクセストークンを管理する媒体ごとに派生クラスが提供されており、\ ``JdbcTokenStore``\ 、\ ``InMemoryTokenStore``\ などが実装クラスとなる。

|

例として、認可コードグラントにおける、認可サーバのトークンエンドポイントアクセス時のフローを以下に示す。

.. figure:: ./images/OAuth_AutohrizationServerAccessTokenResponse.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **認可サーバの動き（アクセストークンレスポンス）**
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``TokenEndpoint``\ の呼び出しが行われることで認可サーバ側の処理が開始される。
    * - | (2)
      - | \ ``TokenEndpoint``\ では、\ ``AbstractEndpoint``\ が保持する\ ``ClientDetailsService``\ の呼び出しを行いアクセストークンリクエストに含まれる
          クライアント情報やパラメータの検証後、\ ``TokenGranter``\ の呼び出しを行う。
        | \ ``AuthorizationCodeTokenGranter``\ が\ ``TokenGranter``\ の実装クラスとなる。
    * - | (3)
      - | \ ``TokenGranter``\ では、まず基底クラスである\ ``AbstractTokenGranter``\ の処理が行われる。
        | \ ``AbstractTokenGranter``\ では、\ ``ClientDetailsService``\ の呼び出しを行うことでリクエストパラメータに指定されたクライアント情報を取得し、
          グラントタイプの検証後、グラントタイプ固有の処理（\ ``AuthorizationCodeTokenGranter``\ ）の呼び出しを行う。
    * - | (4)
      - | \ ``AuthorizationCodeTokenGranter``\ では、リクエストパラメータから取得した認可コードを指定して\ ``AuthorizationCodeServices``\ の呼び出しを行い、発行済み認可コードの削除を行うとともに認可リクエスト時の情報を取得する。
    * - | (5)
      - | \ ``AuthorizationCodeServices``\ では、認可コードグラントのアクセストークンリクエストパラメータとして指定されたクライアントID、リダイレクトURIと(4)にて取得した認可リクエストの内容を比較し、アクセストークンリクエストの妥当性の検証を行う。
        | 検証結果に問題がない場合、リクエストパラメータと認可リクエスト時に取得したリソースオーナの認証情報（ユーザ名など）より、認証情報を作成し呼び出し元に返却する。
    * - | (6)
      - | \ ``AbstractTokenGranter``\ では、\ ``AuthorizationServerTokenServices``\ のアクセストークン取得処理の呼び出しを行う。
        | \ ``DefaultTokenServices``\ が\ ``AuthorizationServerTokenServices``\ の実装クラスとなる。
    * - | (7)
      - | \ ``DefaultTokenServices``\ ではアクセストークンを作成する。
        | \ ``TokenEnhancer``\ が指定されている場合、アクセストークンを拡張し、追加パラメータなどを付与することが出来る。
    * - | (8)
      - | \ ``TokenStore``\の呼び出しを行い、(5)にて作成した認証情報とともに(7)で作成したアクセストークンを登録し、アクセストークンを呼び出し元の\ ``TokenEndpoint``\ に返却する。
        | \ ``TokenEndpoint``\ では(7)で作成したアクセストークンをもとにレスポンスを作成し、リクエストの応答を行う。
