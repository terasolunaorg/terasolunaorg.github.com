認証
================================================================================

.. only:: html

 .. contents:: 目次
    :local:

Overview
--------------------------------------------------------------------------------
本節では、Spring Securityで提供している認証機能を説明する。

Spring Securityでは、設定ファイルの記述のみで、ユーザ認証を実装することができる。
Spring Securityで提供している認証方式として、DB認証、LDAP認証、CAS認証、JAAS認証、X509認証、Basic認証がサポートされているが、本ガイドラインでは、DB認証についてのみ説明する。

.. tip::
  DB認証以外の詳細は、各認証方式の公式ドキュメントを参照されたい。

  * \ `LDAP Authentication <http://docs.spring.io/spring-security/site/docs/3.1.x/reference/ldap.html>`_\
  * \ `CAS Authentication <http://docs.spring.io/spring-security/site/docs/3.1.x/reference/cas.html>`_\
  * \ `Java Authentication and Authorization Service (JAAS) Provider <http://docs.spring.io/spring-security/site/docs/3.1.x/reference/jaas.html>`_\
  * \ `X.509 Authentication <http://docs.spring.io/spring-security/site/docs/3.1.x/reference/x509.html>`_\
  * \ `Basic and Digest Authentication <http://docs.spring.io/spring-security/site/docs/3.1.x/reference/basic.html>`_\

Login
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityによるログイン処理の流れを以下に示す。

.. figure:: ./images/Authentication_Login_overview.png
   :alt: Authentication(Login)
   :width: 80%
   :align: center

#. 認証処理を指定したリクエストを受信すると、認証フィルタが起動する。
#. 認証フィルタは、リクエストからユーザ、パスワードを抽出し、認証情報を生成する。
   生成した認証情報をパラメータとし、認証マネージャの認証処理を実行する。
#. 認証マネージャは、指定された認証プロバイダの認証処理を実行する。
   認証プロバイダは、データソース（DBやLDAP）からユーザ情報を取得し、パスワード照合等のユーザ認証を行う。
   認証成功時には、認証済みの情報を保持する認証情報を作成し、
   認証マネージャに返す。認証失敗の場合は、認証失敗例外を送出する。
#. 認証マネージャは、受け取った認証情報を認証フィルタに返す。
#. 認証フィルタは、受け取った認証情報（認証済み）をセッションに格納する。
#. 認証成功時は、認証前のセッション情報を初期化し、新たにセッション情報を作成する。
#. 指定された認証成功/失敗時のパスへリダイレクトする。セッションIDをクライアントに返却する。

Logout
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityによるログアウト処理の流れを以下に示す。

.. figure:: ./images/Authentication_Logout_overview.png
   :alt: Authentication(Logout)
   :width: 80%
   :align: center


#. 指定されたログアウト処理へのリクエストを受信すると、ログアウトフィルタが起動する。
#. ログアウトフィルタはセッション情報を破棄する。
   また、クライアントのクッキー（図中のCookie）を破棄するようなレスポンスを設定する。
#. 指定されたログアウト時のパスへ、リダイレクトする。

\
 .. note::
  ログアウト後、残存するセッション情報が第三者に利用されることによるなりすましを防ぐため、
  セッション情報は、ログアウト時に\ ``org.springframework.security.web.session.ConcurrentSessionFilter``\ で破棄される。

|

How to use
--------------------------------------------------------------------------------
| 認証機能を使用するために、Spring Securityの設定ファイルに記述する内容を以下に示す。
| 基本設定については、\ :doc:`SpringSecurity`\ を参照されたい。

\ ``<sec:http>``\ 要素の設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 以下の設定例のように、spring-security.xmlの\ ``<http>``\ 要素の\ ``auto-config``\ 属性を\ ``true``\ とすることで、
| Spring Securityの認証機能の基本的な設定を、省略することができる。

.. code-block:: xml

  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:sec="http://www.springframework.org/schema/security"
      xmlns:context="http://www.springframework.org/schema/context"
      xsi:schemaLocation="http://www.springframework.org/schema/security
          http://www.springframework.org/schema/security/spring-security.xsd
          http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context
          http://www.springframework.org/schema/context/spring-context.xsd">
      <sec:http auto-config="true" use-expressions="true">  <!-- (1) -->
        <!-- omitted -->
      </sec:http>
  </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``auto-config="true"``\ と設定することで、
       | \ ``<form-login>``\ 、\ ``<http-basic>``\ 、\ ``<logout>``\ 要素を設定しなくても有効になる。

.. note::

  \ ``<form-login>``\ 、\ ``<http-basic>``\ 、\ ``<logout>``\ 要素について説明する。

    .. tabularcolumns:: |p{0.15\linewidth}|p{0.85\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 15 85

       * - 要素名
         - 説明
       * - | \ ``<form-login>``\ 
         - | \ ``org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter``\ が有効になる。
           | UsernamePasswordAuthenticationFilterは、ユーザ名、パスワードをPOST時に、リクエストから取り出し、認証を行うFilterである。
           | 詳細は、\ :ref:`form-login`\ を参照されたい。
       * - | \ ``<http-basic>``\ 
         - | \ ``org.springframework.security.web.authentication.www.BasicAuthenticationFilter``\ が有効になる。
           | BasicAuthenticationFilterは、Basic認証の処理を実施するFilterであり、RFC1945に準拠して実装されている。
           | 詳細な利用方法は、\ `BasicAuthenticationFilter JavaDoc <http://docs.spring.io/spring-security/site/docs/3.1.x/apidocs/org/springframework/security/web/authentication/www/BasicAuthenticationFilter.html>`_\ を参照されたい。
       * - | \ ``<logout>``\ 
         - | \ ``org.springframework.security.web.authentication.logout.LogoutFilter``\ ,
           | \ ``org.springframework.security.web.authentication.logout.SecurityContextLogoutHandler``\ が有効になる。
           | LogoutFilterは、ログアウト時に呼ばれるFilterであり、
           | \ ``org.springframework.security.web.authentication.rememberme.TokenBasedRememberMeServices``\ (Cookieの削除) や、
           | SecurityContextLogoutHandler(セッションの無効化)を呼び出している。
           | 詳細は、\ :ref:`form-logout`\ を参照されたい。

.. _form-login:

\ ``<sec:form-login>``\ 要素の設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 本節では、\ ``<sec:form-login>``\ 要素の設定方法を説明する。
|
| form-login要素の属性について、以下に示す。

spring-security.xml

.. code-block:: xml

  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:sec="http://www.springframework.org/schema/security"
      xmlns:context="http://www.springframework.org/schema/context"
      xsi:schemaLocation="http://www.springframework.org/schema/security
          http://www.springframework.org/schema/security/spring-security.xsd
          http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context
          http://www.springframework.org/schema/context/spring-context.xsd">
    <sec:http auto-config="true" use-expressions="true">
      <sec:form-login login-page="/login"
          default-target-url="/"
          login-processing-url="/authentication"
          always-use-default-target="false"
          authentication-failure-url="/login?error=true"
          authentication-failure-handler-ref="authenticationFailureHandler"
          authentication-success-handler-ref="authenticationSuccessHandler" /> <!-- 属性の指定順番で(1)～(7) -->
    </sec:http>
  </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``login-page``\ 属性にログインフォーム画面のパスを指定する。
       | 「未認証ユーザ」が「認証ユーザ」しかアクセスできないページにアクセスした際に、
       | 強制リダイレクトさせるパス。
   * - | (2)
     - | \ ``default-target-url``\ 属性に認証成功時の遷移先パスを指定する。指定がない場合、"/"が、デフォルトのパスになる。
   * - | (3)
     - | \ ``login-processing-url``\ 属性に認証処理を行うパスを指定する。指定がない場合、「j_spring_security_check」がデフォルトのパスになる。
       | **本ガイドラインでは、上記のデフォルト値「j_spring_security_check」を使用せず、システム独自の値に変更することを推奨する。**\ この例では"/authentication"を指定している。
   * - | (4)
     - | ログイン成功後に\ ``default-target-url``\ に指定したパスに常に遷移するかどうかを\ ``always-use-default-target``\ 属性に設定する。
       | デフォルトは、\ ``false``\ である。\ ``false``\ に設定されている場合、認証成功のハンドラの基底クラスである\ ``org.springframework.security.web.authentication.AbstractAuthenticationTargetUrlRequestHandler``\ で
       | リダイレクト先が指定されていれば、指定先に遷移する。指定がない場合、\ ``default-target-url``\ に指定したパスに遷移する。
   * - | (5)
     - | \ ``authentication-failure-url``\ に認証失敗時の遷移先を設定する。
       | \ ``authentication-failure-handler-ref``\ 属性の指定がない場合、認証エラーの種別を問わず、一律、本設定の遷移先に遷移する。
   * - | (6)
     - | \ ``default-target-url``\ 属性に認証失敗時に呼ばれる、ハンドラクラスを指定する。
       | 詳細は、\ :ref:`authentication-failure-handler-ref`\ を参照されたい。
   * - | (7)
     - | \ ``default-target-url``\ 属性に認証成功時に呼ばれる、ハンドラクラスを指定する。

上記以外の属性については、\ `Spring Securityのマニュアル <http://docs.spring.io/spring-security/site/docs/3.1.x/reference/appendix-namespace.html#nsa-form-login>`_\ を参照されたい。

.. warning:: **Spring Security のデフォルト値「j_spring_security_check」の使用を推奨しない理由**

  デフォルト値を使用している場合、そのアプリケーションが、Spring Securityを使用していることについて、露見してしまう。
  そのため、Spring Securityの脆弱性が発見された場合、脆弱性をついた攻撃を受けるリスクが高くなる。
  前述のリスクを避けるためにも、デフォルト値を使用しないことを推奨する。

.. _form-login-JSP:

ログインフォームの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 認証時に使用するログインフォームをJSPで作成する。

* src/main/webapp/WEB-INF/views/login.jsp

  .. code-block:: jsp

      <form:form action="${pageContext.request.contextPath}/authentication" method="post"><!-- (1) -->
          <!-- omitted -->
          <input type="text" id="username" name="j_username"><!-- (2) -->
          <input type="password" id="password" name="j_password"><!-- (5) -->
          <input type="submit" value="Login">
      </form:form>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | formのaction属性に認証処理を行うための遷移先を指定する。
         | 遷移先のパスはlogin-processing-url属性で指定した、/authentication を指定すること。
         | ${pageContext.request.contextPath}/authenticationにアクセスすることで認証処理が実行される。
         | HTTPメソッドは、「POST」を指定すること。
     * - | (4)
       - | 認証処理において、「ユーザID」として扱われる要素。
         | name属性には、Spring Securityのデフォルト値である「j_username」を指定すること。
     * - | (5)
       - | 認証処理において、「パスワード」として扱われる要素。
         | name属性には、Spring Securityのデフォルト値である「j_password」を指定すること。

  認証エラーメッセージを表示する場合は以下の追加する

  .. code-block:: jsp

      <c:if test="${param.error}"><!-- (1) -->
          <t:messagesPanel
              messagesAttributeName="SPRING_SECURITY_LAST_EXCEPTION"/><!-- (2) -->
      </c:if>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | リクエストパラメータに設定されたエラーメッセージの判定を行う。
         | form-login要素のauthentication-failure-url属性に設定された値や、
         | 認証エラーハンドラの"defaultFailureUrl"に設定された値によって、判定処理を変更する必要があるので注意すること。
         | 本例では、authentication-failure-url="/login?error=true"のような設定がある場合の、例を示している。
     * - | (2)
       - | 認証エラー時に出力させる例外メッセージを出力する。
         | 共通ライブラリで提供している\ ``org.terasoluna.gfw.web.message.MessagesPanelTag``\ を指定して出力させることを推奨する。
         | 「\ ``<t:messagesPanel>``\ 」タグの使用方法は、\ :doc:`../ArchitectureInDetail/MessageManagement`\ を参照されたい。


* spring-mvc.xml

  ログインフォームを表示するControllerを定義する。

  .. code-block:: xml

    <mvc:view-controller path="/login" view-name="login" /><!-- (1) -->
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | "/login"にアクセスされたら、view名として"login"を返却するだけのControllerを定義する。\ ``InternalResourceViewResolver``\ によってsrc/main/webapp/WEB-INF/views/login.jspが出力される。
         | この単純なコントローラはJavaによる実装が不要である。
         
   
  .. note::
   
      上記の設定は次のControllerと同義である。
      
        .. code-block:: java
        
          @Controller
          @RequestMapping("/login")
          public class LoginController {
          
              @RequestMapping
              public String index() {
                  return "login";
              }
          }

      単純にview名を返すだけのメソッドが一つだけあるControllerが必要であれば、\ ``<mvc:view-controller>``\ を使用すればよい。
      
      ログインフォームをController経由で表示するメリットは、CSRFトークンを自動で埋め込める点にある。Controllerを経由しない場合は、
      \ :doc:`Tutorial`\ で実施したようにjspに直接CSRFトークンを埋め込む必要がある。
      
      チュートリアルではController作成の説明を省くために、ログインフォームの表示はControllerを経由していない。CSRF対策の詳細は\ :doc:`CSRF`\ を参照されたい。


ログインフォームの属性名変更
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

「j_username」、「j_password」は、Spring Securityのデフォルト値である。\ ``<form-login>``\ 要素の設定で、任意の値に変更することができる。

* spring-security.xml


  \ ``username``\ 、\ ``password``\ の属性

  .. code-block:: xml

    <sec:http auto-config="true" use-expressions="true">
      <sec:form-login
          username-parameter="username"
          password-parameter="password" /> <!-- 属性の指定順番で(1)～(2) -->
      <!-- omitted -->
    </sec:http>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | \ ``username-parameter``\ 属性で\ ``username``\ の入力フィールドの\ ``name``\ 属性を、「username」に変更している。
     * - | (2)
       - |  \ ``password-parameter``\ 属性で\ ``password``\ の入力フィールドの\ ``name``\ 属性を、「password」に変更している。

認証処理の設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Spring Securityで認証処理を設定するために、\ ``AuthenticationProvider``\ と\ ``UserDetailsService``\ を定義する。

\ ``AuthenticationProvider``\ は、次の役割を担う。

* 認証に成功した場合、認証ユーザー情報を返却する
* 認証に失敗した場合、例外をスローする

\ ``UserDetailsService``\ は、認証ユーザー情報を永続化層から取得する役割を担う。

それぞれデフォルトで用意されているものを使用してもよいし、独自拡張して使用しても良い。
組み合わせも自由である。


\ ``AuthenticationProvider``\ クラスの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| \ ``AuthenticationProvider``\ の実装として、DB認証を行うためのプロバイダ\ ``org.springframework.security.authentication.dao.DaoAuthenticationProvider``\ を使用する方法を説明する。

* spring-security.xml

  .. code-block:: xml

      <sec:authentication-manager><!-- (1) -->
          <sec:authentication-provider user-service-ref="userDetailsService"><!-- (2) -->
              <sec:password-encoder ref="passwordEncoder" /><!-- (3) -->
          </sec:authentication-provider>
      </sec:authentication-manager>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | \ ``<sec:authentication-manager>``\ 要素内に\ ``<sec:authentication-provider>``\ 要素を定義する。複数指定して、認証方法を組み合わせることが可能であるが、ここでは説明しない。
     * - | (2)
       - | \ ``<sec:authentication-provider>``\ 要素で\ ``AuthenticationProvider``\ を定義する。デフォルトで、\ ``DaoAuthenticationProvider``\ が有効になる。これ以外の\ ``AuthenticationProvider``\ を指定する場合は\ `ref属性で、対象のAuthenticationProviderのBean IDを指定する <http://docs.spring.io/spring-security/site/docs/3.1.x/reference/appendix-namespace.html#nsa-authentication-provider>`_\ 。
         |
         | \ ``user-service-ref``\ 属性に、認証ユーザ情報を取得する\ ``UserDetailsService``\ のBean Idを指定する。\ ``DaoAuthenticationProvider``\ を使用する場合、この設定は必須である。
         | 詳細は、\ :ref:`userDetailsService`\ を参照されたい。
     * - | (3)
       - | パスワード照合時に、フォームから入力されたパスワードのエンコードを行うクラスのBean IDを指定する。
         | 指定がない場合に、「平文」でパスワードが扱われる。詳細は、\ :doc:`PasswordHashing`\ を参照されたい。


| 「ユーザーID」と「パスワード」だけで永続化層からデータを取得し、認証するという要件であればこの\ ``DaoAuthenticationProvider``\ を使用すれば良い。
| 永続化層からのデータ取得方法は次に説明する\ ``UserDetailsService``\ で決める。

.. _userDetailsService:

\ ``UserDetailsService``\ クラスの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| \ ``AuthenticationProvider``\ の\ ``userDetailsService``\ プロパティに指定したBeanを設定する。

\ ``UserDetailsService``\ は次のメソッドをもつインタフェースである。

.. code-block:: java

  UserDetails loadUserByUsername(String username) throws UsernameNotFoundException

このインタフェースを実装すれば、任意の保存場所から認証ユーザー情報を取得することができる。

ここでは、JDBCを使用して、DBからユーザ情報を取得する \ ``org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl``\ を説明する。

\ ``JdbcDaoImpl``\ を使用するにはspring-security.xmlに以下のBean定義を行えば良い。

.. code-block:: xml

  <!-- omitted -->

  <bean id="userDetailsService"
    class="org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl">
    <property name="dataSource" ref="dataSource"/>
  </bean>

| \ ``JdbcDaoImpl``\ は、認証ユーザー情報と認可情報を取得するためのデフォルトSQLを定義しており、これらに対応したテーブルが用意されていることが前提となっている。前提としているテーブル定義は\ `Spring Securityのマニュアル <http://docs.spring.io/spring-security/site/docs/3.1.x/reference/appendix-schema.html>`_\ を参照されたい。
| 既存のテーブルからユーザー情報、認可情報を取得したい場合は、発行されるSQLを既存のテーブルに合わせて修正すればよい。
| 使用するSQLは以下の3つである。

*  \ `ユーザ情報取得クエリ <http://docs.spring.io/spring-security/site/docs/3.1.x/apidocs/constant-values.html#org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl.DEF_USERS_BY_USERNAME_QUERY>`_\

  | ユーザ情報取得クエリに合致するテーブルを作成することで、後述する設定ファイルへのクエリ指定が不要となる。
  | 「username」、「password」、「enabled」フィールドは必須であるが、
  | 後述する設定ファイルへのクエリ指定で、別名を付与することにより、テーブル名、カラム名が一致しなくても問題ない。
  | 例えば次のようなSQLを設定すれば「email」カラムを「username」として使用することができ、「enabled」は常に\ ``true``\ となる。

  .. code-block:: sql

    SELECT email AS username, pwd AS password, true AS enabled FROM customer WHERE email = ?

  | \ :ref:`form-login-JSP`\ で前述した、「ユーザID」がクエリのパラメータに指定される。

* \ `ユーザ権限取得クエリ <http://docs.spring.io/spring-security/site/docs/3.1.x/apidocs/constant-values.html#org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl.DEF_AUTHORITIES_BY_USERNAME_QUERY>`_\ 

  | ユーザに対する認可情報を取得するクエリである。

* \ `グループ権限取得クエリ <http://docs.spring.io/spring-security/site/docs/3.1.x/apidocs/constant-values.html#org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl.DEF_GROUP_AUTHORITIES_BY_USERNAME_QUERY>`_\

  | ユーザーが所属するグループの認可情報を取得するクエリである。グループ権限はデフォルトでは無効になっており、本ガイドラインでも扱わない。

| 以下に、DBの定義例、Spring Securityの設定ファイル例を示す。

| テーブルの定義について
| DB認証処理を実装するにあたり、必要となるテーブルを定義する。
| 前述した、デフォルトのユーザ情報取得クエリ合致するテーブルとなっている。
| そのため、下記が最低限必要となるテーブルの定義となる（物理名は仮称）。

テーブル名: account

.. tabularcolumns:: |p{0.15\linewidth}|p{0.15\linewidth}|p{0.10\linewidth}|p{0.60\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 15 15 10 60

   * - 論理名
     - 物理名
     - 型
     - 説明
   * - ユーザID
     - username
     - 文字列
     - ユーザを一意に識別するためのユーザID。
   * - パスワード
     - password
     - 文字列
     - ユーザパスワード。ハッシュ化された状態で格納する。
   * - 有効フラグ
     - enabled
     - 真偽値
     - 無効ユーザ、有効ユーザを表すフラグ。「false」に設定されたユーザは無効ユーザとして、認証エラーとなる。
   * - 権限名
     - authority
     - 文字列
     - 認可機能を必要としない場合は不要。

\ ``JdbcDaoImpl``\ をカスタマイズして設定する例を以下に示す。

.. code-block:: xml

  <!-- omitted -->

  <bean id="userDetailsService"
    class="org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl">
    <property name="rolePrefix" value="ROLE_" /><!-- (1) -->
    <property name="dataSource" ref="dataSource" />
    <property name="enableGroups" value="false" /><!-- (2) -->
    <property name="usersByUsernameQuery"
      value="SELECT username, password, enabled FROM account WHERE username = ?" /><!-- (3) -->
    <property name="authoritiesByUsernameQuery"
      value="SELECT username, authority FROM account WHERE username = ?" /><!-- (4) -->
  </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 権限名のprefixを指定する。DB上に格納されている権限名が"USER"の場合、この認証ユーザーオブジェクトが持つ権限名は"ROLE_USER"になる。
       | 認可機能と命名規則を合わせて設定する必要がある。認可機能の詳細は、\ :doc:`Authorization`\ を参照されたい。
   * - | (2)
     - | 認可機能において、「グループ権限」の概念を用いる場合に指定する。
       | 本ガイドラインでは扱わない。
   * - | (3)
     - | ユーザ情報を取得するクエリを設定する。取得するデータは、「ユーザID」、「パスワード」、「有効フラグ」の順とする。
       | 「有効フラグ」による認証判定を行わない場合には、「有効フラグ」のSELECT結果を「true」固定とする。
       | なお、ユーザを一意に取得できるクエリを記述すること。複数件数取得された場合には、１件目のレコードがユーザとして使われる。
   * - | (4)
     - | ユーザの権限を取得するクエリを設定する。取得するデータは、「ユーザID」、「権限ID」の順とする。
       | 認可の機能を使用しない場合は、「権限ID」は任意の固定値でよい。

.. note::
  クエリを変更するだけでは実現できない認証を行う場合、\ ``UserDetailsService``\ を拡張して実現する必要がある。
  拡張方法については、\ :ref:`extendsuserdetailsservice`\ を参照されたい。

\ ``UserDetails``\ クラスの利用方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


| 認証に成功した後に\ ``UserDetailsService``\ が作成した\ ``UserDetails``\ の利用方法について、説明する。


Javaクラスで\ ``UserDetails``\ オブジェクトを利用する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 認証に成功した後、\ ``UserDetails``\ クラスは
| \ ``org.springframework.security.core.context.SecurityContextHolder``\ に格納される。

\ ``SecurityContextHolder``\ から\ ``UserDetails``\ を取得する例を示す。

.. code-block:: java

  public static String getUsername() {
      Authentication authentication = SecurityContextHolder.getContext()
              .getAuthentication(); // (1)
      if (authentication != null) {
          Object principal = authentication.getPrincipal(); // (2)
          if (principal instanceof UserDetails) {
              return ((UserDetails) principal).getUsername(); // (3)
          }
          return (String) principal.toString();
      }
      return null;
  }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``SecurityContextHolder``\ から\ ``org.springframework.security.core.Authentication``\ オブジェクトを取得する。
   * - | (2)
     - | \ ``Authentication``\ オブジェクトから\ ``UserDetails``\ オブジェクトを取得する。
   * - | (3)
     - | \ ``UserDetails``\ オブジェクトから、ユーザ名を取得する。


\ ``SecurityContextHolder``\ から\ ``UserDetails``\ オブジェクトを取得する方法は、どこからでもstaticメソッドで利用可能であり、
便利な反面、モジュール結合度を高めてしまう。テストも実施しづらい。

| \ ``UserDetails``\ オブジェクトは\ ``@AuthenticationPrincipal``\ を利用することで取得可能である。
| \ ``@AuthenticationPrincipal``\を利用するためには\ ``org.springframework.security.web.bind.support.AuthenticationPrincipalArgumentResolver``\ を\ ``<mvc:argument-resolvers>``\ に設定する必要がある。

- :file:`spring-mvc.xml`

.. code-block:: xml
   :emphasize-lines: 5-6

    <mvc:annotation-driven>
        <mvc:argument-resolvers>
            <bean
                class="org.springframework.data.web.PageableHandlerMethodArgumentResolver" />
            <bean
                class="org.springframework.security.web.bind.support.AuthenticationPrincipalArgumentResolver" />
        </mvc:argument-resolvers>
    </mvc:annotation-driven>


Spring MVCのController内では以下のように\ ``SecurityContextHolder``\ を使用せずに\ ``UserDetails``\ オブジェクトを取得できる。

.. code-block:: java

    @RequestMapping(method = RequestMethod.GET)
    public String view(@AuthenticationPrincipal SampleUserDetails userDetails, // (1)
            Model model) {
        // get account object
        Account account = userDetails.getAccount(); // (2)
        model.addAttribute(account);
        return "account/view";
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``@AuthenticationPrincipal``\ を利用してログインしているユーザ情報を取得する。
   * - | (2)
     - | \ ``SampleUserDetails``\ から\ アカウント情報を取得する。

.. note::

    \ ``@AuthenticationPrincipal``\アノテーションをつける引数の型は\ ``UserDetails``\型を継承したクラスである必要がある。
    通常は\ :ref:`extendsuserdetailsservice`\ で作成する\ ``UserDetails``\継承クラスを使用すればよい。

    \ ``SampleUserDetails``\ クラスは\ :doc:`Tutorial`\ で作成するクラスである。詳細は\ :ref:`Tutorial_CreateAuthService`\ を参照されたい。

\ **Controller内でUserDetailsオブジェクトにアクセスする場合はこちらの方法を推奨する**\ 。

.. note::

  ServiceクラスではControllerが取得した\ ``UserDetails``\ オブジェクトの情報を使用し、\ ``SecurityContextHolder``\ は使用しないことを推奨する。

  \ ``SecurityContextHolder``\ は\ ``UserDetails``\ オブジェクトを引数で渡せないメソッド内でのみ利用することが望ましい。

JSPで\ ``UserDetails``\ にアクセスする
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Spring Securityでは、JSPで認証情報を利用するための仕組みとして、JSP taglibを提供している。このtaglibを使うために以下の宣言が必要である。

.. code-block:: jsp

  <%@ taglib uri="http://www.springframework.org/security/tags" prefix="sec"%>

.. note::

  \ `TERASOLUNA Global Frameworkの雛形 <https://github.com/terasolunaorg/terasoluna-gfw-web-blank>`_\ を使用している場合はWEB-INF/views/common/include.jspに設定済みである。

| 認証ユーザ名をJSPで表示する場合を例に、使用方法を示す。

.. code-block:: jsp

  <sec:authentication property="principal.username" /><!-- (1) -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``<sec:authentication>`` \ タグで\ ``Authentication``\ オブジェクトにアクセスでき、\ ``property``\ 属性に指定したプロパティアクセスできる。この例では\ ``getPrincipal().getUsername()``\ の結果を出力する。



.. code-block:: jsp

  <sec:authentication property="principal" var="userDetails" /> <!-- (1) -->

  ${f:h(userDetails.username)} <!-- (2) -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``property``\ 属性に指定したプロパティを\ ``var``\ 属性にした名前で変数に格納できる。
   * - | (2)
     - | (1)で変数に格納した後はJSP内で\ ``UserDetails``\ にアクセスできる。

.. note::

  Controller内で\ ``UserDetails``\ を取得して\ ``Model``\ に追加することもできるが、JSPに表示する際はJSPタグを使用すればよい。


.. note::
  
  :ref:`userDetailsService`\ で説明した\ ``JdbcDaoImpl``\ が生成する\ ``UserDetails``\ は「ユーザーID」や「権限」といった最低限の情報しか保持していない。
  画面の表示項目として「ユーザー姓名」など他のユーザー情報が必要な場合は\ ``UserDetails``\ と \ ``UserDetailsService``\ を拡張する必要がある。
  拡張方法については、\ :ref:`extendsuserdetailsservice`\ を参照されたい。


.. _authentication(spring_security)_how_to_use_sessionmanagement:

Spring Securityにおけるセッション管理
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| ログイン時のセッション情報の生成方式や、例外発生時の設定を行う方法について説明する。
| \ ``<session-management>``\ タグを指定することで、\ ``org.springframework.security.web.session.SessionManagementFilter``\ が有効になる。
| 以下にspring-security.xmlの設定例を示す。


.. code-block:: xml

  <sec:http auto-config="true" create-session="ifRequired" ><!-- (1) -->
    <!-- omitted -->
    <sec:session-management
      invalid-session-url="/"
      session-authentication-error-url="/"
      session-fixation-protection="migrateSession" /><!-- 属性の指定順番で(2)～(4) -->
    <!-- omitted -->
  </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``create-session``\ 属性でセッションの作成方針を指定する。
       | 以下の値を指定することができる。
       | \ ``always``\ : Spring Securityは、既存のセッションがない場合にセッションを新規作成する、セッションが存在すれば、再利用する。
       | \ ``ifRequired``\ : Spring Securityは、セッションが必要であれば作成する。デフォルトの設定である。セッションがすでにあれば、作成せずに再利用する。
       | \ ``never``\ : Spring Securityは、セッションを作成しないが、セッションが存在すれば、再利用する。
       | \ ``stateless``\ : Spring Securityは、セッションを作成しない、セッションが存在しても使用しない。そのため、毎回認証を行う必要がある。
   * - | (2)
     - | \ ``invalid-session-url``\ 属性で無効なセッションIDがリクエストされた場合に遷移するパスを指定する。
       | 設定しない場合、\ ``org.springframework.security.web.session.SimpleRedirectInvalidSessionStrategy``\
       | の設定に依存したパスに遷移する。
   * - | (3)
     - | \ ``org.springframework.security.web.authentication.session.SessionAuthenticationStrategy``\ で
       | 例外が発生した場合、遷移するパスに\ ``session-authentication-error-url``\ 属性に指定する。
       | 指定しない場合、\ ``org.springframework.security.web.authentication.SimpleUrlAuthenticationFailureHandler``\
       | の設定に依存する。
   * - | (4)
     - | \ ``session-fixation-protection``\ 属性でセッション管理方式を指定する。
       | 以下の値を指定することができる。
       | \ ``migrateSession``\ ：ログイン前のセッション情報を引き継ぎ（コピー）、IDのみ新規作成する。デフォルトの設定である。
       | \ ``newSession``\ ：ログイン前のセッション情報を引き継がず、ID、セッション内容を新規作成する。
       |
       | 本機能の目的は、新しいセッションIDをログイン毎に割り振ることで、\ `セッション・フィクセーション攻撃 <http://docs.spring.io/spring-security/site/docs/3.1.x/reference/ns-config.html#ns-session-fixation>`_\を防ぐことにある。そのため、明確な意図がない限り、デフォルトの設定を推奨する。

.. _authentication_control-user-samatime-session:

Concurrent Session Controlの利用設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Spring Securityでは、1ユーザが保持できる最大セッション数を、任意に変更できる機能(\ `Concurrent Session Control <http://docs.spring.io/spring-security/site/docs/3.1.x/reference/session-mgmt.html#concurrent-sessions>`_\ )を提供している。
| ここでいうユーザとは、\ ``Authentication.getPrincipal()``\ で取得される、認証ユーザーオブジェクトのことである。

.. note::

   この機能はアプリケーションサーバが1台構成、またはセッションサーバやクラスタによるセッションレプリケーションを実施している（つまり、全てのアプリケーションが同じセッション領域を利用している）場合に有効である。
   複数台または複数インスタンスで構成していて、セッション領域が別々に存在する場合は、本機能では同時ログインを制御できないので注意すること。

| 最大セッション数を超えた場合の制御方法は、次のパターンが存在する。業務要件によって使い分けること。

#. 1ユーザの最大セッション数を超過した場合、最も使用されていないユーザを無効にする (後勝ち)
#. 1ユーザの最大セッション数を超過した場合、新規ログインを受け付けない (先勝ち)

どちらの場合も、この機能を有効にするためにはweb.xmlに以下の設定を追加する必要がある。

.. _HttpSessionEventPublisher-ref:

.. code-block:: xml

    <listener>
      <listener-class>org.springframework.security.web.session.HttpSessionEventPublisher</listener-class><!-- (1) -->
    </listener>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | Concurrent Session Control を使用するに当たり、\ ``org.springframework.security.web.session.HttpSessionEventPublisher``\ を、listenerに定義する必要がある。

.. _authentication_concurrency-control:

\ ``<sec:concurrency-control>``\ の設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

``<sec:session-management>``\ 要素では\ ``session-authentication-strategy-ref``\ 属性を指定せず、\ ``<sec:session-management>``\ 要素の子要素として\ `<sec:concurrency-control> <http://docs.spring.io/spring-security/site/docs/3.2.0.RELEASE/reference/htmlsingle/#ns-concurrent-sessions>`_\ 要素を使用することもできる。

.. code-block:: xml

  <sec:http auto-config="true" >
    <sec:session-management>
        <sec:concurrency-control
            error-if-maximum-exceeded="true"
            max-sessions="2"
            expired-url="/alreadyLogin.jsp" /><!-- 属性の指定順番で(1)～(3) -->
        </sec:session-management>
    </sec:session-management>
  </sec:http>


.. tabularcolumns:: |p{0.05\linewidth}|p{0.20\linewidth}|p{0.35\linewidth}|p{0.10\linewidth}|p{0.30\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 5 20 35 10 30

   * - 項番
     - 属性名
     - 説明
     - デフォルト値
     - デフォルト値説明
   * - | (1)
     - | \ ``error-if-maximum-exceeded``\
     - | 新規ログインの可否。
       | \ ``true``\ を設定することによりmax-sessionsの数を越えた場合、新規ログインを受け付けない。エラー後は\ ``<sec:form-login>``\要素の\ ``authentication-failure-url``\属性で指定したurlへ遷移することになる。（先勝ち）
     - | false
     - | 後からログインが可能となり、max-sessionsの数を越えた場合、先にログインしていたセッションが無効となる。先にログインしていたユーザは次のリクエストで\ ``expired-url``\属性で指定したurlへ遷移することになる。（後勝ち）
   * - | (2)
     - | \ ``max-sessions``\
     - | 1ユーザでログイン可能なセッション数を決定する。
       | 2を設定した場合、同じユーザで2つのセッションでログインが可能となる。
     - | 1
     - | デフォルトは1ユーザのみ
   * - | (3)
     - | \ ``expired-url``\
     - | セッションが無効化された場合に遷移するURL。
     - | 無し
     - | 動作としては先頭ページ"/"

.. _authentication_session-authentication-strategy-ref:

.. note::

    \ ``<sec:concurrency-control>``\ の設定を使用せずに、``<sec:session-management>``\ 要素での\ ``session-authentication-strategy-ref``\ 属性を指定することでも同じ事が可能である。
    \ ``session-authentication-strategy-ref``\ 属性で指定した場合の例を記載する。
    
    .. _authentication_user-not-used-most:
    
    1. 最も使用されていないユーザを無効にする場合
    
      spring-security.xmlに以下の設定を追加する。
    
      .. code-block:: xml
    
          <sec:http auto-config="true" >
            <!-- omitted -->
            <sec:session-management
              session-authentication-strategy-ref="sessionStrategy" />  <!-- (1) -->
    
            <sec:custom-filter position="CONCURRENT_SESSION_FILTER" ref="concurrencyFilter" />  <!-- (5) -->
            <!-- omitted -->
          </sec:http>
    
          <bean id="sessionStrategy"
              class="org.springframework.security.web.authentication.session.CompositeSessionAuthenticationStrategy">
              <constructor-arg index="0">
                  <list>
                      <!-- omitted -->
                      <bean class=
                          "org.springframework.security.web.authentication.session.ConcurrentSessionControlStrategy">  <!-- (2) -->
                          <constructor-arg index="0" ref="sessionRegistry" />  <!-- (3) -->
                          <property name="maximumSessions" value="2" />  <!-- (4) -->
                      </bean>
                  </list>
              </constructor-arg>
          </bean>
    
          <bean id="sessionRegistry" class="org.springframework.security.core.session.SessionRegistryImpl" />
    
          <bean id="concurrencyFilter"
             class="org.springframework.security.web.session.ConcurrentSessionFilter">  <!-- (6) -->
             <constructor-arg index="0" ref="sessionRegistry" />  <!-- (7) -->
             <constructor-arg index="1" value="/alreadyLogin.jsp" />  <!-- (8) -->
          </bean>
          <!-- omitted -->
    
      .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
      .. list-table::
         :header-rows: 1
         :widths: 10 90
    
         * - 項番
           - 説明
         * - | (1)
           - | \ ``<sec:session-management>``\ 要素の\ ``session-authentication-strategy-ref``\ 属性にセッションの取り扱いを決めるidを指定する。\ ``org.springframework.security.web.authentication.session.CompositeSessionAuthenticationStrategy``\ を参照指定する。
         * - | (2)
           - | 同一ユーザでログインできる数を制限するために、CompositeSessionAuthenticationStrategyの第1引数に、
             | \ ``org.springframework.security.web.authentication.session.ConcurrentSessionControlStrategy``\ を指定する。
         * - | (3)
           - | コンストラクタの第1引数に、\ ``org.springframework.security.core.session.SessionRegistryImpl``\ を参照指定する。
         * - | (4)
           - | \ ``maximumSessions``\ 属性に、1ユーザが許容する最大セッション数を定義することができる。
             | 上記例では、2を指定しているため、1ユーザが許容するセッションは2つになる。
             | ユーザーが複数のブラウザでログインした場合、使用した日時が最も古いセッションに対して期限切れにする。
             | 指定しない場合、1（1ユーザが許容するセッションは1つ）が設定される。
         * - | (5)
           - | 期限切れになったセッションが遷移するパスを指定するために、\ ``<custom-filter>``\ 要素の、\ ``position``\ 属性に\ ``CONCURRENT_SESSION_FILTER``\ を指定する。
         * - | (6)
           - | \ ``org.springframework.security.web.session.ConcurrentSessionFilter``\ クラスをBean定義する。
         * - | (7)
           - | コンストラクタの第1引数に、\ ``org.springframework.security.core.session.SessionRegistryImpl``\ を参照指定する。
         * - | (8)
           - | コンストラクタの第2引数に、期限切れになったセッションが遷移するパスを指定する。
    
    2. 新規ログインを受け付けない
    
      spring-security.xmlに以下の設定を行う。
    
      .. code-block:: xml
          :emphasize-lines: 16
    
          <bean id="concurrencyFilter"
             class="org.springframework.security.web.session.ConcurrentSessionFilter">
             <constructor-arg index="0" ref="sessionRegistry" />
             <constructor-arg index="1" value="/" />
          </bean>
    
          <bean id="sessionAuthenticationStrategy"
              class="org.springframework.security.web.authentication.session.CompositeSessionAuthenticationStrategy">
              <constructor-arg index="0">
                  <list>
                      <!-- omitted -->
                      <bean class=
                          "org.springframework.security.web.authentication.session.ConcurrentSessionControlStrategy">
                          <constructor-arg index="0" ref="sessionRegistry" />
                          <property name="maximumSessions" value="2" />
                          <property name="exceptionIfMaximumExceeded" value="true"/> <!-- (1) -->
                      </bean>
                  </list>
              </constructor-arg>
          </bean>
          <!-- omitted -->
    
      .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
      .. list-table::
         :header-rows: 1
         :widths: 10 90
    
         * - 項番
           - 説明
         * - | (1)
           - | \ ``exceptionIfMaximumExceeded``\ 属性を\ ``true``\ に設定することにより、 最大セッション数を超過した場合、
             | \ ``org.springframework.security.web.authentication.session.SessionAuthenticationException``\ がスローされる。
             | そのため、\ ``ConcurrentSessionFilter``\ の第2引数で定義したパスには遷移せず、\ ``<sec:form-login>``\ 要素の\ ``authentication-failure-url``\ 属性で指定したurlへ遷移するため注意すること。
             | \ ``exceptionIfMaximumExceeded``\ 属性の設定を省略した場合は、\ ``false``\ が設定される。(後勝ち)

.. _authentication-failure-handler-ref:

認証エラー時のハンドラクラスの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
|  \ ``<sec:form-login>``\ 要素の\ ``authentication-failure-handler-ref``\ 属性に
| \ ``org.springframework.security.web.authentication.ExceptionMappingAuthenticationFailureHandler``\ クラスの設定をし、
| 認証エラー時に送出される例外と、それに対応した遷移先を指定できる。
| 指定する遷移先は、未認証ユーザがアクセス可能であること。

spring-security.xml

.. code-block:: xml

    <sec:http auto-config="true" use-expressions="true">
      <sec:form-login login-page="/login"
          authentication-failure-handler-ref="authenticationFailureHandler"
          authentication-success-handler-ref="authenticationSuccessHandler" />
    </sec:http>

    <bean id="authenticationFailureHandler"
    class="org.springframework.security.web.authentication.ExceptionMappingAuthenticationFailureHandler">
    <property name="defaultFailureUrl" value="/login/defaultError" /><!-- (1) -->
      <property name="exceptionMappings"><!-- (2) -->
        <props>
          <prop key=
            "org.springframework.security.authentication.BadCredentialsException"><!-- (3) -->
              /login/badCredentials
          </prop>
          <prop key=
            "org.springframework.security.core.userdetails.UsernameNotFoundException"><!-- (4) -->
              /login/usernameNotFound
          </prop>
          <prop key=
            "org.springframework.security.authentication.DisabledException"><!-- (5) -->
              /login/disabled
          </prop>
          <prop key=
            "org.springframework.security.authentication.ProviderNotFoundException"><!-- (6) -->
              /login/providerNotFound
          </prop>
          <prop key=
            "org.springframework.security.authentication.AuthenticationServiceException"><!-- (7) -->
              /login/authenticationService
          </prop>
          <!-- omitted -->
        </props>
      </property>
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | エラー時のデフォルトの遷移先パスを指定する。
       | 後述する\ ``exceptionMappings``\ プロパティに定義していない例外が発生した場合、本プロパティで指定した遷移先に遷移する。
   * - | (2)
     - | catchする例外と、例外発生時の遷移先を、リスト形式で指定する。
       | keyに例外クラスを指定し、値に遷移先を設定する。


.. _SpringSecurity-Exception:

Spring Securityがスローする代表的な例外を、以下に記述する。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.25\linewidth}|p{0.25\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 25 65

   * - 項番
     - エラーの種類
     - 説明
   * - | (3)
     - \ ``BadCredentialsException``\ 
     - パスワード照合失敗による認証エラー時にスローされる。
   * - | (4)
     - \ ``UsernameNotFoundException``\ 
     - | 不正ユーザID（存在しないユーザID）による認証エラー時にスローされる。
       | \ ``org.springframework.security.authentication.dao.AbstractUserDetailsAuthenticationProvider``\ を継承したクラスを認証プロバイダに指定している場合、
       | \ ``hideUserNotFoundExceptions``\ を\ ``false``\ に変更しないと上記例外は、\ ``BadCredentialsException``\ に変更される。
   * - | (5)
     - \ ``DisabledException``\ 
     - 無効ユーザIDによる認証エラー時に、スローされる。
   * - | (6)
     - \ ``ProviderNotFoundException``\ 
     - | 認証プロバイダクラス未検出エラー時にスローされる。
       | 設定誤り等の理由から、認証プロバイダクラスが不正な場合に発生する。
   * - | (7)
     - \ ``AuthenticationServiceException``\ 
     - | 認証サービスエラー時にスローされる。
       | DB接続エラー等、認証サービス内で何らかのエラーが発生した際に発生する。

.. warning::

  本例では、\ ``UsernameNotFoundException``\ をハンドリングして遷移させているが、
  ユーザIDが存在しないことを利用者に知らせると、特定のIDの存在有無が判明するため、セキュリティの観点上望ましくない。
  そのため、ユーザに通知するメッセージには、例外の種類によって区別をしない画面遷移、メッセージにした方がよい。

.. _form-logout:

\ ``<sec:logout>``\ 要素の設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 本節では、\ ``<sec:logout>``\ 要素の設定方法を説明する。

spring-security.xml

.. code-block:: xml

  <sec:http auto-config="true" use-expressions="true">
    <!-- omitted -->
    <sec:logout
        logout-url="/logout"
        logout-success-url="/"
        invalidate-session="true"
        delete-cookies="JSESSIONID"
        success-handler-ref="logoutSuccessHandler"
      /> <!-- 属性の指定順番で(1)～(5) -->
    <!-- omitted -->
  </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``logout-url``\ 属性に、ログアウト処理を実行するためのパスを指定する。
   * - | (2)
     - | \ ``logout-success-url``\ 属性に、ログアウト後の遷移先パスを指定する。
   * - | (3)
     - | \ ``invalidate-session``\ 属性に、ログアウト時にセッションを破棄するかを設定する。デフォルトは\ ``true``\ である。\ ``true``\ の場合、ログアウト時にセッションがクリアされる。
   * - | (4)
     - | \ ``delete-cookies``\ 属性に、指定したクッキーを削除する。複数記述する場合は「,」で区切る。
   * - | (5)
     - | \ ``success-handler-ref``\ 属性に、ログアウト成功後に呼び出される、ハンドラクラスを指定する。

.. note::

    \ :doc:`./CSRF`\ で説明している\ ``<sec:csrf>``\ を利用している場合は、CSRFトークンチェックが行われるため、\ **ログアウトのリクエストをPOSTで送信し、CSRFトークンも送信する必要がある**\ 。
    CSRFトークンを埋め込む方法を以下に記述する。

    * \ :ref:`csrf_formformtag-use`\

        .. code-block:: jsp
           :emphasize-lines: 1,4

            <form:form method="POST"
              action="${pageContext.request.contextPath}/logout">
              <input type="submit" value="Logout" />
            </form:form>

        この場合は以下のようなHTMLが出力される。CSRFトークンがhiddenで設定されている。

        .. code-block:: html

            <form id="command" action="/your-context-path/logout" method="POST">
              <input type="submit" value="Logout" />
              <input type="hidden" name="_csrf" value="5826038f-0a84-495b-a851-c363e501b73b" />
            </form>

    * \ :ref:`csrf_formtag-use`\

        .. code-block:: jsp
           :emphasize-lines: 3

            <form  method="POST"
              action="${pageContext.request.contextPath}/logout">
              <sec:csrfInput/>
              <input type="submit" value="Logout" />
            </form>

        この場合も同様に以下のようなHTMLが出力される。CSRFトークンがhiddenで設定されている。

        .. code-block:: html

            <form  method="POST"
              action="/your-context-path/logout">
              <input type="hidden" name="_csrf" value="5826038f-0a84-495b-a851-c363e501b73b" />
              <input type="submit" value="Logout" />
            </form>



\ ``<sec:remember-me>``\ 要素の設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| 「\ `Remeber Me <http://docs.spring.io/spring-security/site/docs/3.1.x/reference/remember-me.html>`_\ 」とは、websiteに頻繁にアクセスするユーザの利便性を、高めるための機能の一つとして、
| ログイン状態を保持する機能である。
| 本機能は、ユーザがログイン状態を保持することを許可していた場合、ブラウザを閉じた後も
| cookieにログイン情報を保持し、ユーザ名、パスワードを再入力しなくともログインすることができる機能である。

| \ ``<sec:remember-me>``\ 要素の属性について、以下に示す。

spring-security.xml

.. code-block:: xml

  <sec:http auto-config="true" use-expressions="true">
    <!-- omitted -->
    <sec:remember-me key="terasoluna-tourreservation-km/ylnHv"
            token-validity-seconds="#{30 * 24 * 60 * 60}" />  <!-- 属性の指定順番で(1)～(2) -->
    <!-- omitted -->
  </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``key``\ 属性に、Remeber Me用のcookieを保持しておくためのユニークなキーを指定する。
       | 指定が無い場合、ユニークなキーを起動時に生成するため、起動時間向上を考えた場合指定しておくことを推奨する。
   * - | (2)
     - | 「\ ``token-validity-seconds``\ 属性に、Remeber Me用のcookieの有効時間を秒単位で指定する。この例では30日間を設定している。
       | 指定が無い場合、デフォルトで14日間が有効期限になる。

上記以外の属性については、\ `Spring Securityのマニュアル <http://docs.spring.io/spring-security/site/docs/3.1.x/reference/appendix-namespace.html#nsa-remember-me>`_\ を参照されたい。

ログインフォームには以下のように「Remeber Me」機能を有効にするためのフラグを用意する必要がある。

.. code-block:: jsp
  :emphasize-lines: 7-9

  <form method="post"
    action="${pageContext.request.contextPath}/authentication">
      <!-- omitted -->
      <label for="_spring_security_remember_me">Remember Me : </label>
      <input name="_spring_security_remember_me"
        id="_spring_security_remember_me" type="checkbox"
        checked="checked"> <!-- (1) -->
      <input type="submit" value="LOGIN">
      <!-- omitted -->
  </form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | HTTPパラメータに、\ ``_spring_security_remember_me``\ を設定することで、
       | \ ``true``\ でリクエストされた場合、次回の認証を回避することができる。

How to extend
--------------------------------------------------------------------------------

.. _extendsuserdetailsservice:

\ ``UserDetailsService``\ の拡張
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 認証時にユーザID、パスワード以外の情報も取得したい場合、

* \ ``org.springframework.security.core.userdetails.UserDetails``\ 
* \ ``org.springframework.security.core.userdetails.userDetailsService``\ 

を実装する必要がある。

ログインユーザーの氏名や所属部署などの付属情報を常に画面のヘッダーに表示させる必要がある場合、毎リクエストでDBから取得するのは非効率的である。
\ ``UserDetails``\ オブジェクトに保持させて、\ ``SecurityContext``\ や\ ``<sec:authentication>``\ タグからアクセスできようにするにはこの拡張が必要である。

\ ``UserDetails``\ の拡張
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
認証情報以外に顧客情報も保持する\ ``ReservationUserDetails``\ クラスを作成する。

.. code-block:: java

  public class ReservationUserDetails extends User { // (1)
      // omitted

      private final Customer customer; // (2)

      private static final List<? extends GrantedAuthority> DEFAULT_AUTHORITIES = Collections
              .singletonList(new SimpleGrantedAuthority("ROLE_USER"));         // (3)

      public ReservationUserDetails(Customer customer) {
          super(customer.getCustomerCode(),
                  customer.getCustomerPassword(), true, true, true, true, DEFAULT_AUTHORITIES); // (4)
          this.customer = customer;
      }

      public Customer getCustomer() { // (5)
          return customer;
      }
  }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``UserDetails``\ のデフォルトクラスである、\ ``org.springframework.security.core.userdetails.User``\ クラスを継承する。
   * - | (2)
     - | 認証情報および顧客情報をもつDomainObjectクラスを保持する。
   * - | (3)
     - | 認可情報を、\ ``org.springframework.security.core.authority.SimpleGrantedAuthority``\ のコンストラクタで作成する。ここでは"ROLE_USER"という権限を与える。
       |
       | 本実装は簡易実装であり、本来は認可情報はDB上の別のテーブルから取得すべきである。
   * - | (4)
     - | スーパークラスのコンストラクタに、DomainObjectが持つユーザID、パスワードを設定する。
   * - | (5)
     - | \ ``UserDetails``\ 経由で顧客情報にアクセスするためのメソッド。

.. note::

  \ ``User``\ クラスを継承するだけでは、業務要件を実現できない場合、\ ``UserDetails``\ インタフェースを実装すればよい。

独自\ ``UserDetailsService``\ の実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| \ ``UserDetailsService``\ を実装したReservationUserDetailsServiceクラスを作成する。
| 本例では、\ ``Customer``\ オブジェクトを取得する処理を実装した\ ``CustomerSharedService``\ クラスをインジェクションして、DBから顧客情報を取得している。

.. code-block:: java

  public class ReservationUserDetailsService implements UserDetailsService {
      @Inject
      CustomerSharedService customerSharedService;

      @Override
      public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
          Customer customer = customerSharedService.findOne(username);
          // omitted
          return new ReservationUserDetails(customer);
      }

  }

使用方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
作成した\ ``ReservationUserDetailsService``\ 、\ ``ReservationUserDetails``\ の使用方法を説明する。

* spring-security.xml

  .. code-block:: xml

    <sec:authentication-manager>
        <sec:authentication-provider user-service-ref="userDetailsService"><!-- (1) -->
            <sec:password-encoder ref="passwordEncoder" />
        </sec:authentication-provider>
    </sec:authentication-manager>

    <bean id="userDetailsService"
        class="com.example.domain.service.userdetails.ReservationUserDetailsService"><!-- (2) -->
    </bean>
    <!-- omitted -->

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | \ ``ReservationUserDetailsService``\ のBean IDをref属性に定義する。
     * - | (2)
       - | \ ``ReservationUserDetailsService``\ をBean定義する。

* JSP

  \ ``<sec:authentication>``\ タグを使用して\ ``Customer``\ オブジェクトにアクセスする。

  .. code-block:: jsp

     <sec:authentication property="principal.customer" var="customer"/><!-- (1) -->
     ${f:h(customer.customerName)}<!-- (1) -->

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``ReservationUserDetails``\ がもつ\ ``Customer``\ オブジェクトを変数に格納する。
    * - | (2)
      - | 変数に格納した\ ``Customer``\ オブジェクトの任意のプロパティを表示する。
        | \ ``f:h()``\ については、\ :doc:`XSS`\ を参照されたい。

* Controller

  .. code-block:: java

    @RequestMapping(method = RequestMethod.GET)
    public String view(Principal principal, Model model) {
        // get Authentication 
        Authentication authentication = (Authentication) principal;
        // get UserDetails
        ReservationUserDetails userDetails = (ReservationUserDetails) authentication.getPrincipal();
        // get Customer
        Customer customer = userDetails.getCustomer(); // (1)
        // omitted ...
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``ReservationUserDetails``\ から、ログイン中の\ ``Customer``\ オブジェクトを取得する。
        | このオブジェクトをServiceクラスに渡して業務処理を行う。

.. note::

  顧客情報が変更された場合、一度ログアウトしないと\ ``ReservationUserDetails``\ がもつ\ ``Customer``\ オブジェクトは変更されない。
  
  頻繁に変更されうる情報や、ログインユーザー以外のユーザー(管理者など)によって変更される情報は保持しない方がよい。


\ ``AuthenticationProvider``\ の拡張
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. todo::

  内容をもう一度見直す。

| Spring Securityで\ `デフォルトで用意されている認証プロバイダ <http://docs.spring.io/spring-security/site/docs/3.1.x/apidocs/org/springframework/security/authentication/AuthenticationProvider.html>`_\ で対応できない業務要件がある場合、
| \ ``org.springframework.security.authentication.AuthenticationProvider``\ を実装したクラスを作成する必要がある。

| \ ``AuthenticationProvider``\ では、\ ``org.springframework.security.core.Authentication``\ を実装したクラスを戻り値として返却する必要がある。
| \ ``Authentication``\ を実装したクラスには、\ ``getPrincipal``\ メソッド(認証情報の取得)、
| \ ``getCredentials``\ メソッド(principalの正当性を保証する情報の取得)を設定する必要がある。

| ここでは認証時に、ユーザ名、パスワード以外にも認証情報が必要な場合を考える。\ ``AuthenticationProvider``\ でこの値にアクセスするために、

* \ ``org.springframework.security.authentication.UsernamePasswordAuthenticationToken``\ 
* \ ``org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter``\ 

を継承したクラスを作成する必要がある。

| ユーザ名、\ **会社識別子**\ 、パスワードを認証情報とする場合の例を用いて、説明する。

.. figure:: ./images/Authentication_HowToExtends_LoginForm.png
   :alt: Authentication_HowToExtends_LoginForm
   :width: 40%

\ ``UsernamePasswordAuthenticationToken``\ の拡張
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| \ ``UsernamePasswordAuthenticationToken``\ を継承した、独自の\ ``AuthenticationToken``\ を作成する。
| \ ``UsernamePasswordAuthenticationToken``\ を拡張することで、認証情報に会社識別子を持たせることができる。

.. code-block:: java

    // import omitted
    public class CompanyIdUsernamePasswordAuthenticationToken extends
                                                                     UsernamePasswordAuthenticationToken {

        private static final long serialVersionUID = SpringSecurityCoreVersion.SERIAL_VERSION_UID;

        private final String companyId;  // (1)

        public CompanyIdUsernamePasswordAuthenticationToken(
                Object principal, Object credentials, String companyId) {  // (2)
            super(principal, credentials);

            this.companyId = companyId;
        }

        public CompanyIdUsernamePasswordAuthenticationToken(
                Object principal, Object credentials, String companyId,
                Collection<? extends GrantedAuthority> authorities) {  // (3)
            super(principal, credentials, authorities);
            this.companyId = companyId;
        }

        public String getCompanyId() {
            return companyId;
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 会社識別子用のフィールドを作成する。
   * - | (2)
     - | 認証前に、\ ``CompanyIdUsernamePasswordAuthenticationToken``\ のインスタンスを作成する時に使用するコンストラクタ。
   * - | (3)
     - | 認証成功後に、\ ``CompanyIdUsernamePasswordAuthenticationToken``\ のインスタンスを作成する時に使用するコンストラクタ。
       | 親クラスのコンストラクタの引数に認可情報も併せて渡すことで、認証済みの状態となる。

独自\ ``AuthenticationProvider``\ の実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 独自の\ ``AuthenticationProvider``\ で\ ``CompanyIdUsernamePasswordAuthenticationToken``\ を使用する。

.. code-block:: java

    // import omitted
    public class CompanyIdUsernamePasswordAuthenticationProvider implements
                                                                        AuthenticationProvider {
        // omitted

        @Override
        public Authentication authenticate(Authentication authentication) throws AuthenticationException {

            CompanyIdUsernamePasswordAuthenticationToken authenticationToken =
                (CompanyIdUsernamePasswordAuthenticationToken) authentication; // (1)

            // Obtain userName, password, companyId
            String username = authenticationToken.getName();
            String password = authenticationToken.getCredentials().toString();
            String companyId = authenticationToken.getCompanyId();
            
            // Obtain user (and grants if needed) from database or other system
            // UserDetails userDetails = ...
            
            // Business logic
            // ...
            
            return new UsernamePasswordAuthenticationToken(userDetails, authentication.getCredentials(),
                    Collections.singletonList(new SimpleGrantedAuthority("ROLE_USER"))); // (2)
        }

        @Override
        public boolean supports(Class<?> authentication) {
            return CompanyIdUsernamePasswordAuthenticationToken.class
                    .equals(authentication); // (3)
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
     - | \ ``UsernamePasswordAuthenticationFilter``\ を拡張したクラスで設定した、独自の\ ``AuthenticationToken``\ にキャストする。
       | 詳細については、後述する\ :ref:`UsernamePasswordAuthenticationFilter の拡張<authentication_custom_usernamepasswordauthenticationfilter>`\ を参照されたい。
   * - | (2)
     - | 認証処理(ユーザ情報の取得、パスワードチェック、会社識別子チェック等)実施後、
       | \ ``UsernamePasswordAuthenticationToken``\ を作成して返却する。パラメータに認可情報も併せて設定することで、
       | 認証済みの\ ``AuthenticationToken``\ インスタンスを生成する。\ ``UserDetails``\ もコンストラクタのパラメータに渡す。
       |
       | 本例では、簡易実装として、単一のロールのみ使用している。
       | 権限についての詳細は、\ :ref:`ユーザ情報管理クラスの設定 <userDetailsService>`\ を参照されたい。
   * - | (3)
     - | 独自の\ ``AuthenticationToken``\ である、
       | \ ``CompanyIdUsernamePasswordAuthenticationToken``\ クラスをサポート対象とする。

.. _authentication_custom_usernamepasswordauthenticationfilter:

\ ``UsernamePasswordAuthenticationFilter``\ の拡張
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| \ ``UsernamePasswordAuthenticationFilter``\ を継承した、独自の\ ``AuthenticationFilter``\ を作成する。
| \ ``UsernamePasswordAuthenticationFilter``\ を拡張することで、\ ``CompanyIdUsernamePasswordAuthenticationProvider``\ へ、
| \ ``CompanyIdUsernamePasswordAuthenticationToken``\ を渡すことができる。

.. code-block:: java

    // import omitted
    public class CompanyIdUsernamePasswordAuthenticationFilter extends
                                                                      UsernamePasswordAuthenticationFilter {

        @Override
        public Authentication attemptAuthentication(HttpServletRequest request,
                HttpServletResponse response) throws AuthenticationException {

            if (!request.getMethod().equals("POST")) {
                throw new AuthenticationServiceException("Authentication method not supported: "
                        + request.getMethod());
            }

            // Obtain UserName, Password, CompanyId
            String username = super.obtainUsername(request);
            String password = super.obtainPassword(request);
            String companyId = obtainCompanyId(request);

            // username required
            if (!StringUtils.hasText(username)) {
                throw new AuthenticationServiceException("UserName is required");
            }

            // validate password, companyId
            
            // omitted other process

            CompanyIdUsernamePasswordAuthenticationToken authRequest =
                    new CompanyIdUsernamePasswordAuthenticationToken(username, password, companyId);  // (1)

            // Allow subclasses to set the "details" property
            setDetails(request, authRequest);

            return this.getAuthenticationManager().authenticate(authRequest);  // (2)
        }

        protected String obtainCompanyId(HttpServletRequest request) {
            return request.getParameter("j_companyid");  // (3)
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``HttpServletRequest``\ から取得した、ユーザ名、パスワード、会社識別子を設定した、
       | \ ``CompanyIdUsernamePasswordAuthenticationToken``\ のインスタンスを生成する。
   * - | (2)
     - | 未認証状態の\ ``CompanyIdUsernamePasswordAuthenticationToken``\ のインスタンスを
       | \ ``org.springframework.security.authentication.AuthenticationManager.authenticate``\ のパラメータとして設定する。
   * - | (3)
     - | 会社識別子を、リクエストパラメータより取得する。

.. note::

  **ログイン情報の入力チェックについて**

  DBサーバへの負荷軽減等で、あきらかな入力誤りに対しては、事前にチェックを行いたい場合がある。
  その場合は、\ :ref:`authentication_custom_usernamepasswordauthenticationfilter`\ のように、
  \ ``UsernamePasswordAuthenticationFilter``\ を拡張することで、入力チェック処理を行うことができる。


使用方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* ログインフォームページ(JSP)

  \ :ref:`form-login-JSP`\ の例に、会社識別子を追加する。

  .. code-block:: jsp
    :emphasize-lines: 7-8

    <form:form action="${pageContext.request.contextPath}/authentication" method="post">
        <!-- omitted -->
        <span>User Id</span><br>
        <input type="text"
            id="username" name="j_username"><br>
        <span>Company Id</span><br>
        <input type="text"
            id="companyid" name="j_companyid"><br>  <!-- (1) -->
        <span>Password</span><br>
        <input type="password"
            id="password" name="j_password"><br>
        <!-- omitted -->
    </form:form>

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | 会社識別子の入力フィールドのnameは、j_companyid を指定する。

* spring-security.xml

  | 独自の\ ``AuthenticationProvider``\ 、\ ``AuthenticationFilter``\ を設定する。

  .. code-block:: xml

    <sec:http auto-config="false" use-expressions="true" entry-point-ref="loginUrlAuthenticationEntryPoint">  <!-- (1) -->
        <!-- omitted -->
        <sec:custom-filter position="FORM_LOGIN_FILTER" ref="companyIdUsernamePasswordAuthenticationFilter" />  <!-- (2) -->

        <!-- omitted -->

        <sec:logout logout-url="/logout" logout-success-url="/"
            delete-cookies="JSESSIONID" invalidate-session="true" />
    </sec:http>

    <bean id="companyIdUsernamePasswordAuthenticationFilter"
        class="com.example.app.common.security.CompanyIdUsernamePasswordAuthenticationFilter">  <!-- (3) -->
        <property name="authenticationManager" ref="authenticationManager" />  <!-- (4) -->
        <property name="authenticationFailureHandler" ref="authenticationFailureHandler" />  <!-- (5) -->
        <property name="authenticationSuccessHandler" ref="authenticationSuccessHandler" />  <!-- (6) -->
        <property name="filterProcessesUrl" value="/authentication" />  <!-- (7) -->
    </bean>

    <bean id="loginUrlAuthenticationEntryPoint" class="org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint">  <!-- (8) -->
        <constructor-arg value="/login" />  <!-- (9) -->
    </bean>

    <sec:authentication-manager alias="authenticationManager">
        <sec:authentication-provider ref="companyIdUsernamePasswordAuthenticationProvider" />  <!-- (10) -->
    </sec:authentication-manager>

    <bean id="companyIdUsernamePasswordAuthenticationProvider"
        class="com.example.app.common.security.CompanyIdUsernamePasswordAuthenticationProvider" />  <!-- (11) -->

    <!-- omitted -->

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | custom-filter要素で、"FORM_LOGIN_FILTER" を差し替えた場合は、auto-config="true" を指定することができない。
         | そのため、auto-configの指定を削除するか、auto-config="false" を指定する必要がある。
         | また、form-login要素も指定できないため、entry-point-ref 属性を明示的に設定する必要がある。
     * - | (2)
       - | custom-filter要素の position属性に "FORM_LOGIN_FILTER" を指定することで、
         | UsernamePasswordAuthenticationFilterからCompanyIdUsernamePasswordAuthenticationFilterに差し替えることができる。
     * - | (3)
       - | CompanyIdUsernamePasswordAuthenticationFilterをbean定義する。
     * - | (4)
       - | authenticationManagerプロパティに、authentication-manager要素のalias属性の値を設定する。
     * - | (5)
       - | authenticationFailureHandlerプロパティに、認証失敗時に呼ばれるハンドラクラスを指定する。
     * - | (6)
       - | authenticationSuccessHandlerプロパティに、認証成功時に呼ばれるハンドラクラスを指定する。
     * - | (7)
       - | filterProcessesUrlプロパティに、認証処理を行うパスを指定する。
     * - | (8)
       - | http要素のentry-point-ref属性に指定する、AuthenticationEntryPointを定義する。
         | form-login要素を指定した際のデフォルトのAuthenticationEntryPointである、
         | \ ``org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint``\ をbean定義する。
     * - | (9)
       - | コンストラクタの引数に、ログイン画面のパスを指定する。
     * - | (10)
       - | authentication-provider要素にCompanyIdUsernamePasswordAuthenticationProvider を参照設定する。
     * - | (11)
       - | CompanyIdUsernamePasswordAuthenticationProviderを、bean定義する。

  .. warning::

       auto-config="false" を指定したことで、\ ``<sec:http-basic>``\ 、\ ``<sec:logout>``\ 要素を使用する場合は、明示的に定義する必要がある。

Appendix
--------------------------------------------------------------------------------
| Spring Securityを使用した認証では、認証に成功した場合設定ファイルに記述したパスに遷移する。
| 「続きを読むにはログインする必要がある」のような業務要件がある場合、
| ログイン後の遷移先を動的に変更したい場合がある。

.. figure:: ./images/Authentication_Appendix_ScreenFlow.png
   :alt: Authentication_Appendix_Screen_Flow
   :width: 60%
   :align: center

   **Picture - Screen_Flow**

| そのような場合、共通ライブラリで提供している、
| \ ``org.terasoluna.gfw.web.security.RedirectAuthenticationHandler``\ を使用する。

| RedirectAuthenticationHandlerを使用した設定例を、下記に示す。

**遷移元画面のJSPの記述例**

.. code-block:: jsp

  <form:form action="${pageContext.request.contextPath}/login" method="get">
      <!-- omitted -->
    <input type="hidden" name="redirectTo"
      value="${pageContext.request.contextPath}/reservetour/read?
      ${f:query(reserveTourForm)}&page.page=${f:h(param['page.page'])}
      &page.size=${f:h(param['page.size'])}" />  <!-- (1) -->
  </form:form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | リダイレクトURLの設定
       | hidden項目の「redirectTo」に遷移先のURLを設定する。
       | nameに指定する値は、後述する設定ファイルに記載する、
       | targetUrlParameterと一致させること。

**ログイン画面のJSPの記述例**

.. code-block:: jsp

  <form:form action="${pageContext.request.contextPath}/authentication" method="post">
       <!-- omitted -->
       <input type="submit"
         value="Login">
       <input type="hidden" name="redirectTo" value="${f:h(param.redirectTo)}" />  <!-- (1) -->
       <!-- omitted -->
  </form:form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | リダイレクトURLの設定
       | 遷移元画面からリクエストパラメータで渡された、
       | リダイレクトURLをhidden項目に設定する。

**Spring Security 設定ファイル**

.. code-block:: xml

  <sec:http auto-config="true">
                  <!-- omitted -->
      <sec:form-login login-page="/login" default-target-url="/"
          always-use-default-target="false"
          authentication-failure-handler-ref="authenticationFailureHandler"
          authentication-success-handler-ref="authenticationSuccessHandler" />
                                                                      <!-- (1) -->
                  <!-- omitted -->
  </sec:http>

  <bean id="authenticationSuccessHandler"
      class="org.terasoluna.gfw.web.security.RedirectAuthenticationHandler">
                                                                      <!-- (2) -->
      <property name="targetUrlParameter" value="redirectTo"/>        <!-- (3) -->
  </bean>

  <bean id="authenticationFailureHandler"
      class="org.springframework.security.web.authentication.ExceptionMappingAuthenticationFailureHandler">
                                                                      <!-- (4) -->
      <property name="defaultFailureUrl" value="/login?error=true"/> <!-- (5) -->
      <property name="useForward" value="true"/>                     <!-- (6) -->
  </bean>
                  <!-- omitted -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | authentication-failure-handler-ref(認証エラー時のハンドラ設定)と
       | authentication-success-handler-ref(認証成功時のハンドラ設定)のBeanIdを指定する。
   * - | (2)
     - | authentication-success-handler-refの参照クラスとして
       | \ ``org.terasoluna.gfw.web.security.RedirectAuthenticationHandler``\ を設定する。
   * - | (3)
     - | 前述したJSPのhiddenに記載した値と一致させること。
       | 本例では、「redirectTo」を指定する。省略した場合、「redirectTo」が設定される。
   * - | (4)
     - | authentication-failure-handler-refの参照クラスとして
       | \ ``org.springframework.security.web.authentication.ExceptionMappingAuthenticationFailureHandler``\ を設定する。
   * - | (5)
     - | 認証失敗時の遷移先パスにはログイン画面のパス、エラーを示すクエリを設定する。
   * - | (6)
     - | 本機能を使用する場合はuseForwardをtrueに指定する必要がある。
       | trueに指定することで、redirectでの遷移から、forwardでの遷移となる。
       | リクエストパラメータにリダイレクト先のURLを保持しているため、
       | forwardにすることでリクエストパラメータを保持したままにする必要がある。

.. tip::
  RedirectAuthenticationHandlerは、オープンリダイレクタ脆弱性対策が施されているため、
  拡張せずに「http://google.com」のような、外部サイトへの遷移をすることはできない。
  別ドメインに移動したいときは、\ ``org.springframework.security.web.RedirectStrategy``\ を実装したクラスを作成する必要がある。
  RedirectStrategyを実装したクラスは、セッターインジェクションでRedirectAuthenticationHandlerのtargetUrlParameterRedirectStrategyに設定する。
  拡張する際の注意点としては、redirectToの値を改竄されても問題ない作りにする必要がある。
  たとえば、リダイレクト先のURLを直接指定せず番号指定にする（ページ番号指定）、
  リダイレクト先のドメインをチェックする等のいずれか対応が必要となる。

.. raw:: latex

   \newpage

