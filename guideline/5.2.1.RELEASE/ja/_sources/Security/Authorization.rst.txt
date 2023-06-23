.. _SpringSecurityAuthorization:

認可
================================================================================

.. only:: html

 .. contents:: 目次
    :local:

Overview
--------------------------------------------------------------------------------
本節では、Spring Securityが提供している認可機能について説明する。

認可処理は、アプリケーションの利用者がアクセスできるリソースを制御するための処理である。
利用者がアクセスできるリソースを制御するためのもっとも標準的な方法は、
リソース(又はリソースの集合)毎にアクセスポリシーを定義してき、利用者がリソースにアクセスしようとした時にアクセスポリシーを調べて制御する方法である。

アクセスポリシーには、どのリソースにどのユーザーからのアクセスを許可するかを定義する。
Spring Securityでは、以下の3つのリソースに対してアクセスポリシーを定義することができる。

* Webリソース
* Javaメソッド
* ドメインオブジェクト \ [#fSpringSecurityAuthorization1]_\
* JSPの画面項目

本節では、「Webリソース」「Javaメソッド」「JSPの画面項目」のアクセスに対して認可処理を適用するための実装例(定義例)を紹介しながら、Spring Securityの認可機能について説明する。

.. [#fSpringSecurityAuthorization1] ドメインオブジェクトのアクセスに対する認可処理については、 \ `Spring Security Reference -Domain Object Security (ACLs)- <http://docs.spring.io/spring-security/site/docs/4.0.4.RELEASE/reference/htmlsingle/#domain-acls>`_\ を参照されたい。

|

認可処理のアーキテクチャ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityは、以下のような流れで認可処理を行う。

.. figure:: ./images_Authorization/AuthorizationArchitecture.png
    :width: 100%

    **認可処理のアーキテクチャ**

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | クライアントは、任意のリソースにアクセスする。
    * - | (2)
      - | \ ``FilterSecurityInterceptor``\ クラスは、\ ``AccessDecisionManager``\ インタフェースのメソッドを呼び出し、リソースへのアクセス権の有無をチェックする。
    * - | (3)
      - | \ ``AffirmativeBased``\ クラス(デフォルトで使用される\ ``AccessDecisionManager``\ の実装クラス)は、\ ``AccessDecisionVoter``\ インタフェースのメソッドを呼び出し、アクセス権の有無を投票させる。
    * - | (4)
      - | \ ``FilterSecurityInterceptor``\ は、\ ``AccessDecisionManager``\ によってアクセス権が付与された場合に限り、リソースへアクセスする。

|

ExceptionTranslationFilter
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``ExceptionTranslationFilter``\ は、認可処理(\ ``AccessDecisionManager``\ )で発生した例外をハンドリングし、クライアントへ適切なレスポンスを行うためのSecurity Filterである。
デフォルトの実装では、未認証ユーザーからのアクセスの場合は認証を促すレスポンス、認証済みのユーザーからのアクセスの場合は認可エラーを通知するレスポンスを返却する。

|

FilterSecurityInterceptor
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``FilterSecurityInterceptor``\ は、HTTPリクエストに対して認可処理を適用するためのSecurity Filterで、実際の認可処理は\ ``AccessDecisionManager``\ に委譲する。
\ ``AccessDecisionManager``\ インタフェースのメソッドを呼び出す際には、クライアントがアクセスしようとしたリソースに指定されているアクセスポリシーを連携する。

|

AccessDecisionManager
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``AccessDecisionManager``\ は、アクセスしようとしたリソースに対してアクセス権があるかチェックを行うためのインタフェースである。

Spring Securityが提供する実装クラスは3種類存在するが、いずれも\ ``AccessDecisionVoter``\というインタフェースのメソッドを呼び出してアクセス権を付与するか否かを判定させている。
\ ``AccessDecisionVoter``\ は「付与」「拒否」「棄権」のいずれかを投票し、\ ``AccessDecisionManager``\ の実装クラスが投票結果を集約して最終的なアクセス権を判断する。
アクセス権がないと判断した場合は、\ ``AccessDeniedException``\ を発生させアクセスを拒否する。

なお、すべての投票結果が「棄権」であった場合、Spring Securityのでデフォルトでは、「アクセス権なし」と判定される。

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: **Spring Securityが提供するAccessDecisionManagerの実装クラス**
    :header-rows: 1
    :widths: 25 75

    * - クラス名
      - 説明
    * - | \ ``AffirmativeBased``\
      - | \ ``AccessDecisionVoter``\ に投票させ、「付与」が１件投票された時点でアクセス権を与える実装クラス。
        | **デフォルトで使用される実装クラス。**
    * - | \ ``ConsensusBased``\
      - | 全ての\ ``AccessDecisionVoter``\ に投票させ、「付与」の投票数が多い場合にアクセス権を与える実装クラス。
        | 「付与」「拒否」が１件以上、且つ同数の場合、Spring Securityのデフォルトでは、「アクセス権あり」と判定される。
    * - | \ ``UnanimousBased``\
      - | \ ``AccessDecisionVoter``\ に投票させ、「拒否」が１件投票された時点で **アクセス権を与えない** 実装クラス。

.. note:: **AccessDecisionVoterの選択**

    使用する\ ``AccessDecisionVoter``\ が1つの場合はどの実装クラスを使っても動作に違いはない。
    複数の\ ``AccessDecisionVoter``\ を使用する場合は、要件に合わせて実装クラスを選択されたい。

|

AccessDecisionVoter
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``AccessDecisionVoter``\ は、アクセスしようとしたリソースに指定されているアクセスポリシーを参照してアクセス権を付与するかを投票するためのインタフェースである。

Spring Securityが提供する主な実装クラスは以下の通り。

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: **Spring Securityが提供するAccessDecisionVoterの主な実装クラス**
    :header-rows: 1
    :widths: 25 75

    * - クラス名
      - 説明
    * - | \ ``WebExpressionVoter``\
      - | SpEL経由で認証情報(\ ``Authentication``\ )が保持する権限情報とリクエスト情報(\ ``HttpServletRequest``\ )を参照して投票を行う実装クラス。
    * - | \ ``RoleVoter``\
      - | 利用者が持つロールを参照して投票を行う実装クラス。
    * - | \ ``RoleHierarchyVoter``\
      - | 利用者が持つ階層化されたロールを参照して投票を行う実装クラス。
    * - | \ ``AuthenticatedVoter``\
      - | 認証状態を参照して投票を行う実装クラス。

.. note:: **デフォルトで適用されるAccessDecisionVoter**

    デフォルトで適用される\ ``AccessDecisionVoter``\ インタフェースの実装クラスは、Spring Security 4.0から\ ``WebExpressionVoter``\ に統一されている。
    \ ``WebExpressionVoter``\ は、\ ``RoleVoter``\ 、\ ``RoleHierarchyVoter``\ 、\ ``AuthenticatedVoter``\ を使用した時と同じことが実現できるため、
    本ガイドラインでも、デフォルトの\ ``WebExpressionVoter``\ を使って認可処理を行う前提で説明を行う。

|

How to use
--------------------------------------------------------------------------------

認可機能を使用するために必要となるbean定義例(アクセスポリシーの指定方法)や実装方法について説明する。

|

.. _SpringSecurityAuthorizationPolicy:

アクセスポリシーの記述方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

アクセスポリシーの記述方法を説明する。

Spring Securityは、アクセスポリシーを指定する記述方法としてSpring Expression Language(SpEL)をサポートしている。
SpELを使わない方法もあるが、本ガイドラインではExpressionを使ってアクセスポリシーを指定する方法で説明を行う。
SpELの使い方については本節でも紹介するが、より詳しい使い方を知りたい場合は \ `Spring Framework Reference Documentation -Spring Expression Language (SpEL)- <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/htmlsingle/#expressions>`_\ を参照されたい。

|

Built-InのCommon Expressions
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityが用意している共通的なExpressionは以下の通り。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table:: **Spring Securityが提供している共通的なExpression**
    :header-rows: 1
    :widths: 30 70
    :class: longtable

    * - Expression
      - 説明
    * - | \ ``hasRole(String role)``\
      - | ログインユーザーが、引数に指定したロールを保持している場合に\ ``true``\ を返却する。
    * - | \ ``hasAnyRole(String... roles)``\
      - | ログインユーザー、が引数に指定したロールのいずれかを保持している場合に\ ``true``\ を返却する。
    * - | \ ``isAnonymous()``\
      - | ログインしていない匿名ユーザーの場合に\ ``true``\ を返却する。
    * - | \ ``isRememberMe()``\
      - | Remember Me認証によってログインしたユーザーの場合に\ ``true``\ を返却する。
    * - | \ ``isAuthenticated()``\
      - | ログイン中の場合に\ ``true``\ を返却する。
    * - | \ ``isFullyAuthenticated()``\
      - | Remember Me認証ではなく通常の認証プロセスによってログインしたユーザーの場合に\ ``true``\ を返却する。
    * - | \ ``permitAll``\
      - | 常に\ ``true``\ を返却する。
    * - | \ ``denyAll``\
      - | 常に\ ``false``\ を返却する。
    * - | \ ``principal``\
      - | 認証されたユーザーのユーザー情報(\ ``UserDetails``\ インタフェースを実装したクラスのオブジェクト)を返却する。
    * - | \ ``authentication``\
      - | 認証されたユーザーの認証情報(\ ``Authentication``\ インタフェースを実装したクラスのオブジェクト)を返却する。

.. raw:: latex

   \newpage

.. note:: **Expressionを使用した認証情報へのアクセス**

    Expressionとして\ ``principal``\ や\ ``authentication``\ を使用すると、ログインユーザーのユーザー情報や認証情報を参照することができるため、ロール以外の属性を使ってアクセスポリシーを設定することが可能になる。

.. note:: **ロール名のプレフィックス** 

    Spring Security 3.2までは、ロール名には\ ``"ROLE_"`` \ プレフィックスを指定する必要があったが、Spring Security 4.0から\ ``"ROLE_"`` \ プレフィックスの指定が不要となっている。 

    例）

    * Spring Secuirty 3.2以前 : \ ``hasRole('ROLE_USER')``\ 
    * Spring Security 4.0以降 : \ ``hasRole('USER')``\ 

|

Built-InのWeb Expressions
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityが用意しているWebアプリケーション向けExpressionは以下の通り。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table:: **Spring Securityが提供するWebアプリケーション向けExpression**
    :header-rows: 1
    :widths: 30 70

    * - Expression
      - 説明
    * - | \ ``hasIpAddress(String ipAddress)``\
      - | リクエスト元のIPアドレスが、引数に指定したIPアドレス体系に一致する場合に\ ``true``\ を返却する。

演算子の使用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

演算子を使用した判定も行うことができる。
以下の例では、ロールと、リクエストされたIPアドレス両方に合致した場合、アクセス可能となる。

* spring-security.xmlの定義例

  .. code-block:: xml
  
    <sec:http>
        <sec:intercept-url pattern="/admin/**" access="hasRole('ADMIN') and hasIpAddress('192.168.10.1')"/>
        <!-- omitted -->
    </sec:http>
  
  **使用可能な演算子一覧**
  
  .. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 20 80
  
     * - 演算子
       - 説明
     * - | \ ``[式1] and [式2]``\ 
       - | 式1、式2が、どちらも真の場合に、真を返す。
     * - | \ ``[式1] or [式2]``\ 
       - | いずれかの式が、真の場合に、真を返す。
     * - | \ ``![式]``\ 
       - | 式が真の場合は偽を、偽の場合は真を返す。

|

Webリソースへの認可
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityは、サーブレットフィルタの仕組みを利用してWebリソース(HTTPリクエスト)に対して認可処理を行う。

認可処理の適用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Webリソースに対して認可処理を適用する場合は、以下のようなbean定義を行う。

* spring-security.xmlの定義例

.. code-block:: xml

    <sec:http>
        <!-- omitted -->
        <sec:intercept-url pattern="/**" access="isAuthenticated()" />  <!-- (1) -->
        <!-- omitted -->
    </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``<sec:intercept-url>``\ タグに、HTTPリクエストに対してアクセスポリシーを定義する。
        | ここでは、SpELを使用して「Webアプリケーション配下の全てのリクエストに対して認証済みのユーザーのみアクセスを許可する」というアクセスポリシーを定義している。

.. note:: **use-expressionsのデフォルト定義**

    Spring Security 4.0から、\ ``<sec:http>``\  タグの\ ``use-expressions``\ 属性のデフォルト値が\ ``true``\ に変更になっているため、\ ``true``\を使用する場合に明示的な記述は不要となった。

アクセスポリシーの定義
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

bean定義ファイルを使用して、Webリソースに対してアクセスポリシーを定義する方法について説明する。


.. _access_policy_designate_web_resource:

アクセスポリシーを適用するWebリソースの指定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''


まず、アクセスポリシーを適用するリソース(HTTPリクエスト)を指定する。
アクセスポリシーを適用するリソースの指定は、\ ``<sec:intercept-url>``\ タグの以下の属性を使用する。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table:: **アクセスポリシーを適用するリソースを指定するための属性**
    :header-rows: 1
    :widths: 20 80

    * - 属性名
      - 説明
    * - | \ ``pattern``\
      - | Ant形式又は正規表現で指定したパスパターンに一致するリソースを適用対象にするための属性。
    * - | \ ``method``\
      - | 指定したHTTPメソッド(GET,POSTなど)を使ってアクセスがあった場合に適用対象にするための属性。
    * - | \ ``requires-channel``\ 
      - | 「http」、もしくは「https」を指定する。指定したプロトコルでのアクセスを強制するための属性。
        | 指定しない場合、どちらでもアクセス可能である。

上記以外の属性については、\ `<intercept-url> <http://docs.spring.io/spring-security/site/docs/4.0.4.RELEASE/reference/htmlsingle/#nsa-intercept-url>`_\ を参照されたい。

* \ ``<sec:intercept-url>``\ タグ\ ``pattern``\ 属性の定義例（spring-security.xml）

.. code-block:: xml

    <sec:http >
        <sec:intercept-url pattern="/admin/accounts/**" access="..."/>
        <sec:intercept-url pattern="/admin/**" access="..."/>
        <sec:intercept-url pattern="/**" access="..."/>
        <!-- omitted -->
    </sec:http>


Spring Securityは定義した順番でリクエストとのマッチング処理を行い、最初にマッチした定義を適用する。
そのため、bean定義ファイルを使用してアクセスポリシーを指定する場合も定義順番には注意が必要である。

.. tip:: **パスパターンの解釈**

    Spring Securityのデフォルトの動作では、パスパターンはAnt形式で解釈する。
    パスパターンを正規表現で指定したい場合は、\ ``<sec:http>``\ タグの\ ``request-matcher``\ 属性に
    \ ``"regex"``\ を指定すること。

      .. code-block:: xml

          <sec:http request-matcher="regex">
              <sec:intercept-url pattern="/admin/accounts/.*" access=hasRole('ACCOUNT_MANAGER')" />
              <!-- omitted -->
          </sec:http>

.. warning::
    Spring MVCとSpring Securityでは、リクエストとのマッチングの仕組みが厳密には異なっており、この差異を利用してSpring Securityの認可機能を突破し、ハンドラメソッドにアクセスできる脆弱性が存在する。
    本事象の詳細は「\ `CVE-2016-5007 Spring Security / MVC Path Matching Inconsistency <https://pivotal.io/security/cve-2016-5007>`_\」を参照されたい。

    本事象は、\ `trimTokens` \ プロパティに \ `false` \ を設定した \ `org.springframework.util.AntPathMatcher` \ のBeanをSpring MVCに適用することで回避することができる。

      .. code-block:: xml

          <mvc:annotation-driven>
              <mvc:path-matching path-matcher="pathMatcher" />
          </mvc:annotation-driven>

          <bean id="pathMatcher" class="org.springframework.util.AntPathMatcher">
              <property name="trimTokens" value="false" />
          </bean>

    上記の対策をTERASOLUNA Server Framework for Javaで提供するブランクプロジェクトでは設定しているが、
    設定を外すと脆弱性にさらされてしまうので注意する必要がある。

    また、特定のURLに対してアクセスポリシーを設ける(\ ``pattern``\属性に\ ``*``\や\ ``**``\などのワイルドカード指定を含めない)場合、
    拡張子を付けたパターンとリクエストパスの末尾に\ ``/``\を付けたパターンに対するアクセスポリシーの追加が必須である。

    下記の設定例は、\ ``/restrict``\に対して「ROLE_ADMIN」ロールを持つユーザからのアクセスのみを許可している。

      .. code-block:: xml

          <sec:http>
              <sec:intercept-url pattern="/restrict.*" access="hasRole('ADMIN')" /> <!-- (1) --> 
              <sec:intercept-url pattern="/restrict/" access="hasRole('ADMIN')" /> <!-- (2) --> 
              <sec:intercept-url pattern="/restrict" access="hasRole('ADMIN')" /> <!-- (3) -->
              <!-- omitted -->
          </sec:http> 
          
      .. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
      .. list-table::
         :header-rows: 1
         :widths: 20 80
         :class: longtable

         * - 項番
           - 説明
         * - | (1)
           - | \ ``/restrict``\に拡張子を付けたパターン(\ ``/restrict.json``\など)のアクセスポリシーを定義する。
         * - | (2)
           - | \ ``/restrict``\にリクエストパスの末尾に\ ``/``\を付けたパターン(\ ``/restrict/``\など)のアクセスポリシーを定義する。
         * - | (3)
           - | \ ``/restrict``\に対するアクセスポリシーを定義する。

.. warning::

    Spring SecurityとSpring MVCではアクセスされたURLを取得する方法が異なっているため、この差異を利用してSpring Securityの認可機能を突破しハンドラメソッドにアクセスできる脆弱性が存在する。
    本事象は、APサーバまたはバージョンによって発生状況が異なる。
    詳細は「\ `CVE-2016-9879 Encoded "/" in path variables <https://pivotal.io/jp/security/cve-2016-9879>`_\」を参照されたい。

    対策として、本事象への対策が行われているTERASOLUNA Server Framework for Java 5.3.0.RELEASE以降にバージョンアップされたい。

アクセスポリシーの指定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

つぎに、アクセスポリシーを指定する。
アクセスポリシーの指定は、\ ``<sec:intercept-url>``\ タグの\ ``access``\ 属性に指定する。

* \ ``<sec:intercept-url>``\ タグ\ ``access``\ 属性の定義例（\ ``spring-security.xml``\ ）

  .. code-block:: xml
  
    <sec:http>
        <sec:intercept-url pattern="/admin/accounts/**" access="hasRole('ACCOUNT_MANAGER')"/>
        <sec:intercept-url pattern="/admin/configurations/**" access="hasIpAddress('127.0.0.1') and hasRole('CONFIGURATION_MANAGER')" />
        <sec:intercept-url pattern="/admin/**" access="hasRole('ADMIN')" />
        <!-- omitted -->
    </sec:http>
  
  .. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
  .. list-table:: **アクセスポリシーを指定するための属性**
     :header-rows: 1
     :widths: 20 80
  
     * - 属性名
       - 説明
     * - | \ ``access``\ 
       - | SpELでのアクセス制御式や、アクセス可能なロールを指定する。

| ログインユーザーに「ROLE_USER」「ROLE_ADMIN」というロールがある場合を例に、設定例を示す。

* \ ``<sec:intercept-url>``\ タグ\ ``pattern``\ 属性の定義例（spring-security.xml）

  .. code-block:: xml
  
    <sec:http>
        <sec:intercept-url pattern="/reserve/**" access="hasAnyRole('USER','ADMIN')" /> <!-- (1) -->
        <sec:intercept-url pattern="/admin/**" access="hasRole('ADMIN')" /> <!-- (2) -->
        <sec:intercept-url pattern="/**" access="denyAll" /> <!-- (3) -->
        <!-- omitted -->
    </sec:http>
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 「/reserve/\**」にアクセスするためには、「ROLE_USER」もしくは「ROLE_ADMIN」ロールが必要である。
         | \ ``hasAnyRole``\ については、後述する。
     * - | (2)
       - | 「/admin/\**」にアクセスするためには、「ROLE_ADMIN」ロールが必要である。
         | \ ``hasRole``\ については、後述する。
     * - | (3)
       - | \ ``denyAll``\ を全てのパターンに設定し、
         | 権限設定が記述されていないURLに対してはどのユーザーもアクセス出来ない設定としている。
         | \ ``denyAll``\ については、後述する。

  .. note:: **URLパターンの記述順序について**

     クライアントからのリクエストに対して、intercept-urlで記述されているパターンに、上から順にマッチさせ、マッチしたパターンに対してアクセス認可を行う。
     そのため、パターンの記述は、必ず、より限定されたパターンから記述すること。

\ Spring Securiyではデフォルトで、SpELが有効になっている。 
\ ``access``\ 属性に記述したSpELは真偽値で評価され、式が真の場合に、アクセスが認可される。
以下に使用例を示す。

* spring-security.xmlの定義例

  .. code-block:: xml
  
    <sec:http>
        <sec:intercept-url pattern="/admin/**" access="hasRole('ADMIN')"/>  <!-- (1) -->
        <!-- omitted -->
    </sec:http>
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | \ ``hasRole('ロール名')``\ を指定することで、ログインユーザーが指定したロールを保持していれば真を返す。
  
  .. _spring-el:
  
使用可能な主なExpressionは、:ref:`SpringSecurityAuthorizationPolicy` を参照されたい。

|

メソッドへの認可
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityは、Spring AOPの仕組みを利用してDIコンテナで管理しているBeanのメソッド呼び出しに対して認可処理を行う。

メソッドに対する認可処理は、ドメイン層(サービス層)のメソッド呼び出しに対して行うことを想定して提供されている。
メソッドに対する認可処理を使用すると、ドメインオブジェクトのプロパティを参照することができるため、きめの細かいアクセスポリシーの定義を行うことが可能になる。

|

AOPの有効化
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

メソッドへの認可処理を使用する場合は、メソッド呼び出しに対して認可処理を行うためのコンポーネント(AOP)を有効化する必要がある。
AOPを有効化すると、アクセスポリシーをメソッドのアノテーションに定義できるようになる。

Spring Securityは、以下のアノテーションをサポートしている。

* \ ``@PreAuthorize``\ 、\ ``@PostAuthorize``\ 、\ ``@PreFilter``\ 、\ ``@PostFilter``\
* JSR-250 (\ ``javax.annotation.security``\ パッケージ)のアノテーション(\ ``@RolesAllowed``\ など)
* \ ``@Secured``\

本ガイドラインでは、アクセスポリシーをExpressionで使用することができる\ ``@PreAuthorize``\、\ ``@PostAuthorize``\ を使用する方法を説明する。

* spring-security.xmlの定義例

.. code-block:: xml

    <sec:global-method-security pre-post-annotations="enabled" /> <!-- (1) (2) -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``<sec:global-method-security>``\ タグを付与すると、メソッド呼び出しに対する認可処理を行うAOPが有効になる。
    * - | (2)
      - | \ ``pre-post-annotations``\ 属性に\ ``true``\ を指定する。
        | \ ``pre-post-annotations``\ 属性に\ ``true``\ を指定すると、Expressionを指定してアクセスポリシーを定義できるアノテーションが有効になる。

|

認可処理の適用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

メソッドに対して認可処理を適用する際は、アクセスポリシーを指定するアノテーションを使用して、メソッド毎にアクセスポリシーを定義する。

アクセスポリシーの定義
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

メソッド実行前に適用するアクセスポリシーの指定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

メソッドの実行前に適用するアクセスポリシーを指定する場合は、\ ``@PreAuthorize``\ を使用する。

\ ``@PreAuthorize``\ の\ ``value``\ 属性に指定したExpressionの結果が\ ``true``\ になるとメソッドの実行が許可される。
下記例では、管理者以外は、他人のアカウント情報にアクセスできないように定義している。

* \ ``@PreAuthorize``\ の定義例

.. code-block:: java

    // (1) (2)
    @PreAuthorize("hasRole('ADMIN') or (#username == principal.username)")
    public Account findOne(String username) {
        return accountRepository.findOne(username);
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 認可処理を適用したいメソッドに、\ ``@PreAuthorize``\ を付与する。
    * - | (2)
      - | \ ``value``\ 属性に、メソッドに対してアクセスポリシーを定義する。
        | ここでは、「管理者の場合は全てのアカウントへのアクセスを許可する」「管理者以外の場合は自身のアカウントへのアクセスのみ許可する」というアクセスポリシーを定義している。

ここでポイントになるのは、Expressionの中からメソッドの引数にアクセスしている部分である。
具体的には、「\ ``#username``\ 」の部分が引数にアクセスしている部分である。
Expression内で「# + 引数名」形式のExpressionを指定することで、メソッドの引数にアクセスすることができる。

.. tip:: **引数名を指定するアノテーション**

    Spring Securityは、クラスに出力されているデバッグ情報から引数名を解決する仕組み
    になっているが、アノテーション(\ ``@org.springframework.security.access.method.P``\ )
    を使用して明示的に引数名を指定することもできる。

    以下のケースにあてはまる場合は、アノテーションを使用して明示的に変数名を指定する。

    * クラスに変数のデバッグ情報を出力しない
    * Expressionの中から実際の変数名とは別の名前を使ってアクセスしたい (例えば短縮した名前)

      .. code-block:: java

          @PreAuthorize("hasRole('ADMIN') or (#username == principal.username)")
          public Account findOne(@P("username") String username) {
              return accountRepository.findOne(username);
          }
    
    なお、\ ``#username``\ と、メソッドの引数である \ ``username``\ の名称が一致している場合は \ ``@P``\ を省略することが可能である。
    ただし、Spring Securityは引数名の解決を、実装クラスの引数名を使用して行っているため ``@PreAuthorize`` アノテーションをインターフェースに定義している場合には、
    **実装クラスの引数名を、 @PreAuthorize 内で指定した #username と一致させる必要がある** ので、注意されたい。

    JDK 8 から追加されたコンパイルオプション(\ ``-parameters``\ )を使用すると、メソッドパラメータにリフレクション用のメタデータが生成されるため、アノテーションを指定しなくても引数名が解決される。

メソッド実行後に適用するアクセスポリシーの指定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

メソッドの実行後に適用するアクセスポリシーを指定する場合は、\ ``@PostAuthorize``\ を使用する。

\ ``@PostAuthorize``\ の\ ``value``\ 属性に指定したExpressionの結果が\ ``true``\ になるとメソッドの実行結果が呼び出し元に返却される。
下記例では、所属する部署が違うユーザーのアカウント情報にアクセスできないように定義している。

* \ ``@PostAuthorize``\ の定義例

.. code-block:: java

    @PreAuthorize("...")
    @PostAuthorize("(returnObject == null) " +
            "or (returnObject.departmentCode == principal.account.departmentCode)")
    public Account findOne(String username) {
        return accountRepository.findOne(username);
    }

ここでポイントになるのは、Expressionの中からメソッドの返り値にアクセスしている部分である。
具体的には、「\ ``returnObject.departmentCode``\ 」の部分が返り値にアクセスしている部分である。
Expression内で「\ ``returnObject``\ 」を指定すると、メソッドの返り値にアクセスすることができる。

|

JSPの画面項目への認可
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityは、JSPタグライブラリを使用してJSPの画面項目に対して認可処理を適用することができる。

ここでは最もシンプルな定義を例に、JSPの画面項目のアクセスに対して認可処理を適用する方法について説明する。

|

アクセスポリシーの定義
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

JSPタグライブラリを使用してJSPの画面項目に対してアクセスポリシーを定義する際は、表示を許可する条件(アクセスポリシー)をJSPに定義する。

* アクセスポリシー定義例

.. code-block:: jsp

    <%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags" %>

    <!-- (1) -->
    <sec:authorize access="hasRole('ADMIN')"> <!-- (2) -->
        <h2>Admin Menu</h2>
        <!-- omitted -->
    </sec:authorize>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | アクセスポリシーを適用したい部分を\ ``<sec:authorize>``\ タグで囲む。
    * - | (2)
      - | \ ``access``\ 属性にアクセスポリシーを定義する。ここでは、「管理者の場合は表示を許可する」というアクセスポリシーを定義している。

|

Webリソースに指定したアクセスポリシーとの連動
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ボタンやリンクなど(サーバーへのリクエストを伴う画面項目)に対してアクセスポリシーを定義する際は、リクエスト先のWebリソースに定義されているアクセスポリシーと連動させる。
Webリソースに指定したアクセスポリシーと連動させる場合は、\ ``<sec:authorize>``\ タグの\ ``url``\ 属性を使用する。

\ ``url``\ 属性に指定したWebリソースにアクセスできる場合に限り\ ``<sec:authorize>``\ タグの中に実装したJSPの処理が実行される。

* Webリソースに定義されているアクセスポリシーとの連携例

.. code-block:: jsp

    <ul>
        <!-- (1) -->
        <sec:authorize url="/admin/accounts"> <!-- (2) -->
            <li>
                <a href="<c:url value='/admin/accounts' />">Account Management</a>
            </li>
        </sec:authorize>
    </ul>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ボタンやリンクを出力する部分を\ ``<sec:authorize>``\ タグで囲む。
    * - | (2)
      - | \ ``<sec:authorize>``\ タグの\ ``url``\ 属性にWebリソースへアクセスするためのURLを指定する。
        | ここでは、「\ ``"/admin/accounts"``\ というURLが割り振られているWebリソースにアクセス可能な場合は表示を許可する」というアクセスポリシーを定義しており、Webリソースに定義されているアクセスポリシーを直接意識する必要がない。

.. note:: **HTTPメソッドによるポリシーの指定**

    Webリソースのアクセスポリシーの定義をする際に、HTTPメソッドによって異なるアクセスポリシーを指定している場合は、\ ``<sec:authorize>``\ タグの\ ``method``\ 属性を指定して、連動させる定義を特定すること。

.. warning:: **表示制御に関する留意点**

    ボタンやリンクなどの表示制御を行う場合は、必ずWebリソースに定義されているアクセスポリシーと連動させること。

    ボタンやリンクに対して直接アクセスポリシーの指定を行い、Webリソース自体にアクセスポリシーを定義していないと、
    URLを直接してアクセスするような不正なアクセスを防ぐことができない。

|

認可処理の判定結果を変数に格納
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``<sec:authorize>``\ タグを使って呼び出した認可処理の判定結果は、変数に格納して使いまわすことができる。

* JSPの実装例

.. code-block:: jsp

    <sec:authorize url="/admin/accounts"
                   var="hasAccountsAuthority"/> <!-- (1) -->

    <c:if test="${hasAccountsAuthority}"> <!-- (2) -->
        <!-- omitted -->
    </c:if>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - |  (1)
      - | \ ``var``\ 属性に判定結果を格納するための変数名を指定する。
        | アクセスが許可された場合は、変数に\ ``true``\ が設定される。
    * - | (2)
      - | 変数の値を参照して表示処理を実装する。

|

認可エラー時のレスポンス
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityは、リソースへのアクセスを拒否した場合、以下のような流れでエラーをハンドリングしてレスポンスの制御を行う。

.. figure:: ./images_Authorization/AuthorizationAccessDeniedHandling.png
    :width: 100%

    **認可エラーのハンドリングの仕組み**

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | Spring Securityは、リソースやメソッドへのアクセスを拒否するために、\ ``AccessDeniedException``\ を発生させる。
    * - | (2)
      - | \ ``ExceptionTranslationFilter``\ クラスは、\ ``AccessDeniedException``\ をキャッチし、\ ``AccessDeniedHandler``\ または\ ``AuthenticationEntryPoint``\ インタフェースのメソッドを呼び出してエラー応答を行う。
    * - | (3)
      - | 認証済みのユーザーからのアクセスの場合は、\ ``AccessDeniedHandler``\ インタフェースのメソッドを呼び出してエラー応答を行う。
    * - | (4)
      - | 未認証のユーザーからのアクセスの場合は、\ ``AuthenticationEntryPoint``\ インタフェースのメソッドを呼び出してエラー応答を行う。

|

AccessDeniedHandler
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``AccessDeniedHandler``\ インタフェースは、認証済みのユーザーからのアクセスを拒否した際のエラー応答を行うためのインタフェースである。
Spring Securityは、\ ``AccessDeniedHandler``\ インタフェースの実装クラスとして以下のクラスを提供している。

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: **Spring Securityが提供するAccessDeniedHandlerの実装クラス**
    :header-rows: 1
    :widths: 25 75

    * - クラス名
      - 説明
    * - | \ ``AccessDeniedHandlerImpl``\
      - | HTTPレスポンスコードに403(Forbidden)を設定し、指定されたエラーページに遷移する。
        | エラーページの指定がない場合は、HTTPレスポンスコードに403(Forbidden)を設定してエラー応答(\ ``HttpServletResponse#sendError``\ )を行う。
    * - | \ ``InvalidSessionAccessDeniedHandler``\
      - | \ ``InvalidSessionStrategy``\ インタフェースの実装クラスに処理を委譲する。
        | このクラスは、CSRF対策とセッション管理機能を使用してセッションタイムアウトを検知する設定を有効にした際に、CSRFトークンがセッションに存在しない(つまりセッションタイムアウトが発生している)場合に使用される。
    * - | \ ``DelegatingAccessDeniedHandler``\
      - | \ ``AccessDeniedException``\ と\ ``AccessDeniedHandler``\ インタフェースの実装クラスのマッピングを行い、発生した\ ``AccessDeniedException``\に対応する\ ``AccessDeniedHandler``\ インタフェースの実装クラスに処理を委譲する。
        | \ ``InvalidSessionAccessDeniedHandler``\ はこの仕組みを利用して呼び出されている。


Spring Securityのデフォルトの設定では、エラーページの指定がない\ ``AccessDeniedHandlerImpl``\ が使用される。

|

AuthenticationEntryPoint
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``AuthenticationEntryPoint``\ インタフェースは、未認証のユーザーからのアクセスを拒否した際のエラー応答を行うためのインタフェースである。
Spring Securityは、\ ``AuthenticationEntryPoint``\ インタフェースの実装クラスとして以下のクラスを提供している。

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: **Spring Securityが提供する主なAuthenticationEntryPointの実装クラス**
    :header-rows: 1
    :widths: 25 75

    * - クラス名
      - 説明
    * - | \ ``LoginUrlAuthenticationEntryPoint``\
      - | フォーム認証用のログインフォームを表示する。
    * - | \ ``BasicAuthenticationEntryPoint``\
      - | Basic認証用のエラー応答を行う。
        | 具体的には、HTTPレスポンスコードに401(Unauthorized)を、レスポンスヘッダとしてBasic認証用の「\ ``WWW-Authenticate``\ 」ヘッダを設定してエラー応答(\ ``HttpServletResponse#sendError``\ )を行う。
    * - | \ ``DigestAuthenticationEntryPoint``\
      - | Digest認証用のエラー応答を行う。
        | 具体的には、HTTPレスポンスコードに401(Unauthorized)を、レスポンスヘッダとしてDigest認証用の「\ ``WWW-Authenticate``\ 」ヘッダを設定してエラー応答(\ ``HttpServletResponse#sendError``\ )を行う。
    * - | \ ``Http403ForbiddenEntryPoint``\
      - | HTTPレスポンスコードに403(Forbidden)を設定してエラー応答(\ ``HttpServletResponse#sendError``\ )を行う。
    * - | \ ``DelegatingAuthenticationEntryPoint``\
      - | \ ``RequestMatcher``\ と\ ``AuthenticationEntryPoint``\ インタフェースの実装クラスのマッピングを行い、HTTPリクエストに対応する\ ``AuthenticationEntryPoint``\ インタフェースの実装クラスに処理を委譲する。

Spring Securityのデフォルトの設定では、認証方式に対応する\ ``AuthenticationEntryPoint``\ インタフェースの実装クラスが使用される。

|

.. _SpringSecurityAuthorizationOnError:


認可エラー時の遷移先
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityのデフォルトの設定だと、認証済みのユーザーからのアクセスを拒否した際は、アプリケーションサーバのエラーページが表示される。
アプリケーションサーバーのエラーページを表示してしまうと、システムのセキュリティを低下させる要因になるのため、適切なエラー画面を表示することを推奨する。
エラーページの指定は、以下のようなbean定義を行うことで可能である。

* spring-security.xmlの定義例

.. code-block:: xml

    <sec:http>
        <!-- omitted -->
        <sec:access-denied-handler
            error-page="/WEB-INF/views/common/error/accessDeniedError.jsp" /> <!-- (1) -->
        <!-- omitted -->
    </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``<sec:access-denied-handler>``\ タグの\ ``error-page``\ 属性に認可エラー用のエラーページを指定する。

.. tip:: **サーブレットコンテナのエラーページ機能の利用**

    認可エラーのエラーページは、サーブレットコンテナのエラーページ機能を使って指定することもできる。

    サーブレットコンテナのエラーページ機能を使う場合は、\ ``web.xml``\ の\ \ ``<error-page>``\ タグを使用してエラーページを指定する。

     .. code-block:: xml

         <error-page>
             <error-code>403</error-code>
             <location>/WEB-INF/views/common/error/accessDeniedError.jsp</location>
         </error-page>

How to extend
--------------------------------------------------------------------------------

本節では、Spring Securityが用意しているカスタマイズポイントや拡張方法について説明する。

Spring Securityは、多くのカスタマイズポイントを提供しているため、すべてのカスタマイズポイントは紹介しない。
本節では代表的なカスタマイズポイントに絞って説明を行う。

|

認可エラー時のレスポンス (認証済みユーザー編)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ここでは、認証済みユーザーからのアクセスを拒否した際の動作をカスタマイズする方法を説明する。

.. _SpringSecurityAuthorizationAccessDeniedHandler:

AccessDeniedHandlerの適用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityが提供しているデフォルトの動作をカスタマイズする仕組みだけでは要件をみたせない場合は、\ ``AccessDeniedHandler``\ インタフェースの実装クラスを直接適用することができる。

例えば、Ajaxのリクエスト(REST APIなど)で認可エラーが発生した場合は、エラーページ(HTML)ではなくJSON形式でエラー情報を応答することが求められるケースがある。
そのような場合は、\ ``AccessDeniedHandler``\ インタフェースの実装クラスを作成してSpring Securityに適用することで実現することができる。

* AccessDeniedHandlerインタフェースの実装クラスの作成例

.. code-block:: java

    public class JsonDelegatingAccessDeniedHandler implements AccessDeniedHandler {

        private final RequestMatcher jsonRequestMatcher;
        private final AccessDeniedHandler delegateHandler;

        public JsonDelegatingAccessDeniedHandler(
                RequestMatcher jsonRequestMatcher, AccessDeniedHandler delegateHandler) {
            this.jsonRequestMatcher = jsonRequestMatcher;
            this.delegateHandler = delegateHandler;
        }

        public void handle(HttpServletRequest request, HttpServletResponse response,
                           AccessDeniedException accessDeniedException)
                throws IOException, ServletException {
            if (jsonRequestMatcher.matches(request)) {
                // response error information of JSON format
                response.setStatus(HttpServletResponse.SC_FORBIDDEN);
                // omitted
            } else {
                // response error page of HTML format
                delegateHandler.handle(
                        request, response, accessDeniedException);
            }
        }

    }

* spring-security.xmlの定義例

.. code-block:: xml

    <!-- (1) -->
    <bean id="accessDeniedHandler"
          class="com.example.web.security.JsonDelegatingAccessDeniedHandler">
        <constructor-arg>
            <bean class="org.springframework.security.web.util.matcher.AntPathRequestMatcher">
                <constructor-arg value="/api/**"/>
            </bean>
        </constructor-arg>
        <constructor-arg>
            <bean class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
                <property name="errorPage"
                          value="/WEB-INF/views/common/error/accessDeniedError.jsp"/>
            </bean>
        </constructor-arg>
    </bean>

    <sec:http>
        <!-- omitted -->
        <sec:access-denied-handler ref="accessDeniedHandler" />  <!-- (2) -->
        <!-- omitted -->
    </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - \ (1)
      - \ ``AccessDeniedHandler``\ インタフェースの実装クラスをbean定義してDIコンテナに登録する。
    * - \ (2)
      - \ ``<sec:access-denied-handler>``\ タグの\ ``ref``\ 属性に\ ``AccessDeniedHandler``\ のbeanを指定する。

|

認可エラー時のレスポンス (未認証ユーザー編)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ここでは、未認証ユーザーからのアクセスを拒否した際の動作をカスタマイズする方法を説明する。

リクエスト毎にAuthenticationEntryPointを適用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

認証済みユーザーと同様に、Ajaxのリクエスト(REST APIなど)で認可エラーが発生した場合は、ログインページ(HTML)ではなくJSON形式でエラー情報を応答することが求められるケースがある。
そのような場合は、リクエストのパターン毎に\ ``AuthenticationEntryPoint``\ インタフェースの実装クラスをSpring Securityに適用することで実現することができる。

* spring-security.xmlの定義例

.. code-block:: xml

    <!-- (1) -->
    <bean id="authenticationEntryPoint"
          class="org.springframework.security.web.authentication.DelegatingAuthenticationEntryPoint">
        <constructor-arg>
            <map>
                <entry>
                    <key>
                        <bean class="org.springframework.security.web.util.matcher.AntPathRequestMatcher">
                            <constructor-arg value="/api/**"/>
                        </bean>
                    </key>
                    <bean class="com.example.web.security.JsonAuthenticationEntryPoint"/>
                </entry>
            </map>
        </constructor-arg>
        <property name="defaultEntryPoint">
            <bean class="org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint">
                <constructor-arg value="/login"/>
            </bean>
        </property>
    </bean>

    <sec:http entry-point-ref="authenticationEntryPoint"> <!-- (2) -->
        <!-- omitted -->
    </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``AuthenticationEntryPoint``\ インタフェースの実装クラスをbean定義してDIコンテナに登録する。
        | ここでは、Spring Securityが提供している\ ``DelegatingAuthenticationEntryPoint``\ クラスを利用して、リクエストのパターン毎に\ ``AuthenticationEntryPoint``\ インタフェースの実装クラスを適用している。
    * - | (2)
      - | \ ``<sec:http>``\ タグの\ ``entry-point-ref``\ 属性に\ ``AuthenticationEntryPoint``\ のbeanを指定する。

.. note:: **デフォルトで適用されるAuthenticationEntryPoint**

    リクエストに対応する\ \ ``AuthenticationEntryPoint``\ インタフェースの実装クラスの指定がない場合は、Spring Securityがデフォルトで定義する\ ``AuthenticationEntryPoint``\ インタフェースの実装クラスが使用される仕組みになっている。
    認証方式としてフォーム認証を使用する場合は、\ ``LoginUrlAuthenticationEntryPoint``\ クラスが使用されログインフォームが表示される。

|

ロールの階層化
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
認可処理では、ロールに階層関係を設けることができる。

上位に指定したロールは、下位のロールにアクセスが許可されているリソースにもアクセスすることができる。
ロールの関係が複雑な場合は、階層関係も設けることも検討されたい。

例えば、「ROLE_ADMIN」が上位ロール、「ROLE_USER」が下位ロールという階層関係を設けた場合、
下記のようアクセスポリシーを設定すると、「ROLE_ADMIN」権限を持つユーザーは、
\ ``"/user"``\ 配下のパス(「ROLE_USER」権限を持つユーザーがアクセスできるパス)にアクセスすることができる。

* spring-security.xmlの定義例

.. code-block:: xml

    <sec:http>
        <sec:intercept-url pattern="/user/**" access="hasAnyRole('USER')" />
        <!-- omitted -->
    </sec:http>

|

階層関係の設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ロールの階層関係は、\ ``org.springframework.security.access.hierarchicalroles.RoleHierarchy``\ インタフェースの実装クラスで解決する。

* spring-security.xmlの定義例

.. code-block:: xml

    <bean id="roleHierarchy"
        class="org.springframework.security.access.hierarchicalroles.RoleHierarchyImpl"> <!-- (1) -->
        <property name="hierarchy"> <!-- (2) -->
            <value>
                ROLE_ADMIN > ROLE_STAFF
                ROLE_STAFF > ROLE_USER
            </value>
        </property>
    </bean>
  
.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | \ ``org.springframework.security.access.hierarchicalroles.RoleHierarchyImpl`` クラスを指定する。
         | \ ``RoleHierarchyImpl``\ は、Spring Securityが提供するデフォルトの実装クラスである。
     * - | (2)
       - | \ ``hierarchy``\ プロパティに階層関係を定義する。
         |
         | 書式: [上位ロール] > [下位ロール]
         |
         | 上記例では、
         | STAFFは、USERに認可されたリソースにもアクセス可能である。
         | ADMINは、USERとSTAFFに認可されたリソースにもアクセス可能である。

|

Webリソースの認可処理への適用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ロールの階層化を、WebリソースとJSPの画面項目に対する認可処理に適用する方法を説明する。

* spring-security.xmlの定義例

.. code-block:: xml
  
    <!-- (1) -->
    <bean id="webExpressionHandler"
        class="org.springframework.security.web.access.expression.DefaultWebSecurityExpressionHandler">
        <property name="roleHierarchy" ref="roleHierarchy"/>  <!-- (2) -->
    </bean>
  
    <sec:http>
        <!-- omitted -->
        <sec:expression-handler ref="webExpressionHandler" />  <!-- (3) -->
    </sec:http>
  
.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - | 項番
       - | 説明
     * - | (1)
       - | \ ``org.springframework.security.web.access.expression.DefaultWebSecurityExpressionHandler``\ のBeanを定義する。
     * - | (2)
       - | \ ``roleHierarchy``\ プロパティに\ ``RoleHierarchy``\ インタフェースの実装クラスのBeanを指定する。
     * - | (3)
       - | \ ``<sec:expression-handler>``\ タグの\ ``ref``\ 属性に、\ ``org.springframework.security.access.expression.SecurityExpressionHandler``\ インタフェースの実装クラスのBeanを指定する。

|

メソッドの認可処理への適用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ロールの階層化を、Javaメソッドに対する認可処理に適用する方法を説明する。

* spring-security.xmlの定義例

.. code-block:: xml
  
    <bean id="methodExpressionHandler"
        class="org.springframework.security.access.expression.method.DefaultMethodSecurityExpressionHandler"> <!-- (1) -->
        <property name="roleHierarchy" ref="roleHierarchy"/> <!-- (2) -->
    </bean>
  
    <sec:global-method-security pre-post-annotations="enabled">
        <sec:expression-handler ref="methodExpressionHandler" /> <!-- (3) -->
    </sec:global-method-security>
  
.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | \ ``org.springframework.security.access.expression.method.DefaultMethodSecurityExpressionHandler``\ のBeanを定義する。
     * - | (2)
       - | \ ``roleHierarchy``\ プロパティに\ ``RoleHierarchy``\ インタフェースの実装クラスのBeanを指定する。
     * - | (3)
       - | \ ``<sec:expression-handler>``\ タグの\ ``ref``\ 属性に、\ ``org.springframework.security.access.expression.SecurityExpressionHandler``\ インタフェースの実装クラスのBeanを指定する。

.. raw:: latex

   \newpage

