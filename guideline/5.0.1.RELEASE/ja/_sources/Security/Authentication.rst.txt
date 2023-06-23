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

  * \ `LDAP Authentication <http://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/reference/htmlsingle/#ldap>`_\
  * \ `CAS Authentication <http://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/reference/htmlsingle/#cas>`_\
  * \ `Java Authentication and Authorization Service (JAAS) Provider <http://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/reference/htmlsingle/#jaas>`_\
  * \ `X.509 Authentication <http://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/reference/htmlsingle/#x509>`_\
  * \ `Basic and Digest Authentication <http://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/reference/htmlsingle/#basic>`_\

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
           | 詳細な利用方法は、\ `BasicAuthenticationFilter JavaDoc <http://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/apidocs/org/springframework/security/web/authentication/www/BasicAuthenticationFilter.html>`_\ を参照されたい。
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
       | 指定がない場合、「/spring_security_login」がデフォルトのパスになり、Spring Securityが用意しているログイン画面が使用される。
       | 「未認証ユーザ」が「認証ユーザ」しかアクセスできないページにアクセスした際に、本パスにリダイレクトされる。

       | **本ガイドラインでは、上記のデフォルト値「/spring_security_login」を使用せず、システム独自の値に変更することを推奨する。**\ この例では"/login"を指定している。
   * - | (2)
     - | \ ``default-target-url``\ 属性に認証成功時の遷移先パスを指定する。
       | 指定がない場合、「/」が、デフォルトのパスになる。

       | \ ``authentication-success-handler-ref``\ 属性の指定がある場合、本設定は使用されない。
   * - | (3)
     - | \ ``login-processing-url``\ 属性に認証処理を行うパスを指定する。
       | 指定がない場合、「/j_spring_security_check」がデフォルトのパスになる。

       | **本ガイドラインでは、上記のデフォルト値「/j_spring_security_check」を使用せず、システム独自の値に変更することを推奨する。**\ この例では"/authentication"を指定している。
   * - | (4)
     - | \ ``always-use-default-target``\ 属性に、ログイン成功後に\ ``default-target-url``\ に指定したパスに常に遷移するかどうかを設定する。
       | \ ``true``\ が指定されている場合、\ ``default-target-url``\ に指定したパスに常に遷移する。
       | \ ``false``\ (デフォルト)が指定されている場合、「ログイン前にアクセスしようとした保護ページを表示するためのパス」又は「\ ``default-target-url``\ に指定したパス」のいずれかに遷移する。

       | \ ``authentication-success-handler-ref``\ 属性の指定がある場合、本設定は使用されない。
   * - | (5)
     - | \ ``authentication-failure-url``\ に認証失敗時の遷移先を設定する。
       | 指定がない場合、\ ``login-page``\ 属性に指定したパスが適用される。

       | \ ``authentication-failure-handler-ref``\ 属性の指定がある場合、本設定は使用されない。
   * - | (6)
     - | \ ``authentication-failure-handler-ref``\ 属性に認証失敗時に呼ばれる、ハンドラクラスを指定する。
       | 詳細は、\ :ref:`authentication-failure-handler-ref`\ を参照されたい。
   * - | (7)
     - | \ ``authentication-success-handler-ref``\ 属性に認証成功時に呼ばれる、ハンドラクラスを指定する。

上記以外の属性については、\ `Spring Securityのマニュアル <http://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/reference/htmlsingle/#nsa-form-login>`_\ を参照されたい。

.. warning:: **Spring Security のデフォルト値「/spring_security_login, /j_spring_security_check」の使用を推奨しない理由**

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

  認証エラーメッセージを表示する場合は以下のコードをJSPに追加する。

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


 .. note:: **認証エラーの例外オブジェクトにJSPからアクセスする際に必要な設定について**

    認証エラーの例外オブジェクトは、セッションスコープに\ ``"SPRING_SECURITY_LAST_EXCEPTION"``\ という属性名で格納されている。
    JSPからセッションスコープに格納されているオブジェクトにアクセスするためには、JSPの\ ``page``\ ディレクティブの\ ``session``\ 属性を\ ``true``\ にする必要がある。

    * ``src/main/webapp/WEB-INF/views/common/include.jsp``

     .. code-block:: jsp

        <%@ page session="true"%>

    ブランクプロジェクトのデフォルト設定では、JSPからセッションスコープにアクセスできないようになっている。
    これは、安易にセッションが使用されないようにするためである。


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
         
   
  .. tip::
   
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

.. _AuthenticationProviderConfiguration:

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
       - | \ ``<sec:authentication-provider>``\ 要素で\ ``AuthenticationProvider``\ を定義する。デフォルトで、\ ``DaoAuthenticationProvider``\ が有効になる。これ以外の\ ``AuthenticationProvider``\ を指定する場合は\ `ref属性で、対象のAuthenticationProviderのBean IDを指定する <http://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/reference/htmlsingle/#nsa-authentication-provider>`_\ 。
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

| \ ``JdbcDaoImpl``\ は、認証ユーザー情報と認可情報を取得するためのデフォルトSQLを定義しており、これらに対応したテーブルが用意されていることが前提となっている。前提としているテーブル定義は\ `Spring Securityのマニュアル <http://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/reference/htmlsingle/#appendix-schema>`_\ を参照されたい。
| 既存のテーブルからユーザー情報、認可情報を取得したい場合は、発行されるSQLを既存のテーブルに合わせて修正すればよい。
| 使用するSQLは以下の3つである。

*  \ `ユーザ情報取得クエリ <http://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/apidocs/constant-values.html#org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl.DEF_USERS_BY_USERNAME_QUERY>`_\

  | ユーザ情報取得クエリに合致するテーブルを作成することで、後述する設定ファイルへのクエリ指定が不要となる。
  | 「username」、「password」、「enabled」フィールドは必須であるが、
  | 後述する設定ファイルへのクエリ指定で、別名を付与することにより、テーブル名、カラム名が一致しなくても問題ない。
  | 例えば次のようなSQLを設定すれば「email」カラムを「username」として使用することができ、「enabled」は常に\ ``true``\ となる。

  .. code-block:: sql

    SELECT email AS username, pwd AS password, true AS enabled FROM customer WHERE email = ?

  | \ :ref:`form-login-JSP`\ で前述した、「ユーザID」がクエリのパラメータに指定される。

* \ `ユーザ権限取得クエリ <http://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/apidocs/constant-values.html#org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl.DEF_AUTHORITIES_BY_USERNAME_QUERY>`_\

  | ユーザに対する認可情報を取得するクエリである。

* \ `グループ権限取得クエリ <http://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/apidocs/constant-values.html#org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl.DEF_GROUP_AUTHORITIES_BY_USERNAME_QUERY>`_\

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

  \ `TERASOLUNA Server Framework for Java (5.x)の雛形 <https://github.com/terasolunaorg/terasoluna-gfw-web-blank>`_\ を使用している場合はWEB-INF/views/common/include.jspに設定済みである。

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
| \ ``<session-management>``\ タグを指定することで、セッションの管理方法をカスタマイズする事ができる。

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
     - | \ ``<http>``\ タグの\ ``create-session``\ 属性には、セッションの作成方針を指定する。
       | 以下の値を指定することができる。

       * | \ ``always``\ :
         | Spring Securityは、既存のセッションがない場合にセッションを新規作成する、セッションが存在すれば、再利用する。

       * | \ ``ifRequired``\ : (デフォルト)
         | Spring Securityは、セッションが必要であれば作成する。セッションがすでにあれば、作成せずに再利用する。

       * | \ ``never``\ :
         | Spring Securityは、セッションを作成しないが、セッションが存在すれば、再利用する。

       * | \ ``stateless``\ :
         | Spring Securityは、セッションを作成しない、セッションが存在しても使用しない。そのため、毎回認証を行う必要がある。
   * - | (2)
     - | \ ``invalid-session-url``\ 属性には、無効なセッションIDがリクエストされた(セッションタイムアウトが発生した)場合に遷移するパスを指定する。
       | 指定しない場合、セッションの存在チェックは実行されずに後続処理が呼び出される。
       | 詳細は、「:ref:`authentication_session-timeout`」を参照されたい。
   * - | (3)
     - | \ ``session-authentication-error-url``\ 属性には、\ ``org.springframework.security.web.authentication.session.SessionAuthenticationStrategy``\ で例外が発生した場合に遷移するパスを指定する。
       | 指定しない場合、レスポンスコードに「401 Unauthorized」が設定され、エラー応答が行われる。
       |
       | 本設定は、\ ``<form-login>``\ タグを使用して認証を行う場合は使用されない。\ ``SessionAuthenticationStrategy``\ で発生した例外は、\ ``<form-login>``\ タグの\ ``authentication-failure-url``\ 属性 又は\ ``authentication-failure-handler-ref``\ 属性の定義に応じてハンドリングされる。
   * - | (4)
     - | \ ``session-fixation-protection``\ 属性には、認証成功時のセッション管理方式を指定する。
       | 以下の値を指定することができる。

       * | \ ``none``\ ：
         | ログイン前のセッションをそのまま利用する。

       * | \ ``migrateSession``\ ： (Servlet 3.0以前のコンテナ上でのデフォルト)
         | ログイン前のセッションを破棄して新しいセッションを新たに作成し、ログイン前のセッションに格納していた情報を引き継ぐ。

       * | \ ``changeSessionId``\ ： (Servlet 3.1以降のコンテナ上でのデフォルト)
         | Servlet 3.1から追加された\ ``javax.servlet.http.HttpServletRequest#changeSessionId()``\ メソッドを使用してセッションIDを変更する。

       * | \ ``newSession``\ ：
         | ログイン前のセッションを破棄して新しいセッションを新たに作成し、ログイン前のセッションに格納していた情報は引き継がない。

       | 本機能の目的は、新しいセッションIDをログイン毎に割り振ることで、\ `セッション・フィクセーション攻撃 <http://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/reference/htmlsingle/#ns-session-fixation>`_\を防ぐことにある。そのため、明確な意図がない限り、デフォルト値を使用することを推奨する。

|

.. _authentication_session-timeout:

セッションタイムアウトの検出
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

セッションタイムアウトを検出したい場合は、
\ ``invalid-session-url``\ 属性にセッションタイムアウトが発生した際に遷移するパスを指定すればよい。

\ ``invalid-session-url``\ 属性を指定すると、
\ ``http``\ 要素の\ ``pattern``\ 属性に指定したパスパターンに一致する全てのリクエストに対して、
セッションの存在チェック(リクエストされたセッションIDの存在チェック)が行われる。

.. note::

    セッションタイムアウトを検出するパスと検出しないパスが混在する場合は、\ ``http``\ 要素を複数定義する必要がある。
    \ ``http``\ 要素を複数定義すると、設定が冗長になりメンテナンス性が低下する事があるので注意が必要である。

    セッションタイムアウトを検出するために設定が冗長になる場合は、適用パスや除外パスの指定ができるカスタムフィルタを作成することを検討してほしい。
    カスタムフィルタを作成する際には、Spring Securityから提供されている以下のクラスを利用又は参考にするとよい。

    * | ``org.springframework.security.web.session.SessionManagementFilter``
      | セッションの存在チェック(リクエストされたセッションIDの存在チェック)を行う処理が実装されている。

    * | ``org.springframework.security.web.session.SimpleRedirectInvalidSessionStrategy``
      | セッションタイムアウト(無効なセッションID)を検出した後の処理が実装されている。
      | デフォルトでは、セッションを生成した後に指定されたパスへリダイレクトする。

    * | ``org.springframework.security.web.util.matcher.RequestMatcher``
      | リクエストのマッチング判定を行うためのインタフェースであり、適用パスや除外パスの判定処理で利用できる。
      | 同じパッケージ内にいくつかの便利な実装クラスが提供されている。

.. note::

    \ ``<csrf>``\ 要素を指定して\ :doc:`CSRF`\ を行っている場合は、CSRF対策機能によってセッションタイムアウトが検出されるケースがある。

    以下に、CSRF対策機能によってセッションタイムアウトが検出される条件を示す。

    * CSRFトークンの保存先をHTTPセッション(デフォルト)にしている。
    * HTTPセッションからCSRFトークンが取得できない。
    * \ :ref:`CSRFトークンのチェック対象になっているリクエスト <csrf_default-add-token-method>` \ である。

    CSRF対策機能によってセッションタイムアウトが検出された場合は、以下のいずれかの動作となる。

    * \ ``invalid-session-url``\ 属性の指定がある場合は、セッションを生成した後に\ ``invalid-session-url``\ に指定したパスへリダイレクトされる。
    * \ ``invalid-session-url``\ 属性の指定がない場合は、\ ``<access-denied-handler>``\ 要素に指定した\ ``org.springframework.security.web.access.AccessDeniedHandler``\ の定義に従ったハンドリングが行われる。

    \ ``AccessDeniedHandler``\ の定義方法については、「:ref:`csrf_spring-security-setting`」を参照されたい。

|

.. _authentication_control-user-samatime-session:

Concurrent Session Controlの利用設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Spring Securityでは、1ユーザが同時にログインできるセッション数を制御する機能(\ `Concurrent Session Control <http://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/reference/htmlsingle/#concurrent-sessions>`_\ )を提供している。
| ここでいうユーザとは、\ ``Authentication.getPrincipal()``\ で取得される、認証ユーザーオブジェクトのことである。

.. note::

   この機能はアプリケーションサーバが1台構成、またはセッションサーバやクラスタによるセッションレプリケーションを実施している（つまり、全てのアプリケーションが同じセッション領域を利用している）場合に有効である。
   複数台または複数インスタンスで構成していて、セッション領域が別々に存在する場合は、本機能では同時ログインを制御できないので注意すること。

|

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

Concurrent Session Controlを利用する場合は、
\ ``<sec:session-management>``\ 要素の子要素として\ `<sec:concurrency-control> <http://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/reference/htmlsingle/#ns-concurrent-sessions>`_\ 要素を指定する。

.. code-block:: xml

  <sec:http auto-config="true" >
    <sec:session-management>
        <sec:concurrency-control
            error-if-maximum-exceeded="true"
            max-sessions="2"
            expired-url="/expiredSessionError.jsp" /><!-- 属性の指定順番で(1)～(3) -->
    </sec:session-management>
  </sec:http>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.30\linewidth}|p{0.10\linewidth}|p{0.30\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 20 30 10 30

   * - 項番
     - 属性名
     - 説明
     - デフォルト値
     - デフォルト値説明
   * - | (1)
     - | \ ``error-if-maximum-exceeded``\
     - | ログイン可能な最大セッション数を超えている状態でログイン要求があった場合の挙動を指定する。
       | \ ``true``\ を設定した場合、認証エラーを発生させて、新規ログインを受け付けない。（先勝ち）
     - | false
     - | ログインが可能となり、最も使用されていない(最終アクセス時刻が最も古い)セッションが無効化される。無効化されたセッションを利用しているクライアントからリクエストが発生した場合は、\ ``expired-url``\属性で指定したURLへ遷移する。（後勝ち）
   * - | (2)
     - | \ ``max-sessions``\
     - | 1ユーザでログイン可能な最大セッション数を指定する。
       | 2を設定した場合、同じユーザで2つのセッションでログインが可能となる。
     - | 1
     - | デフォルトは1セッションのみ
   * - | (3)
     - | \ ``expired-url``\
     - | 無効化されたセッションを利用しているクライアントからリクエストが発生した場合に遷移するURL。
     - | 無し
     - | セッションが無効化されたことを通知する固定メッセージが応答される。

.. _authentication_session-authentication-strategy-ref:

.. note::

    認証処理用のフィルタ(FORM_LOGIN_FILTER)をカスタマイズする場合は、
    \ ``<sec:concurrency-control>``\ 要素の指定に加えて、以下の２つの\ ``SessionAuthenticationStrategy``\ クラスを有効化する必要がある。

    * | ``org.springframework.security.web.authentication.session.ConcurrentSessionControlAuthenticationStrategy``
      | 認証成功後にログインユーザ毎のセッション数をチェックするクラス。

    * | ``org.springframework.security.web.authentication.session.RegisterSessionAuthenticationStrategy``
      | 認証に成功したセッションをセッション管理領域に登録するクラス。

    version 1.0.x.RELEASEで依存しているSpring Security 3.1では、\ ``org.springframework.security.web.authentication.session.ConcurrentSessionControlStrategy``\ というクラスが提供されていたが、
    Spring Security 3.2より非推奨のAPIになっている。
    Spring Security 3.1からSpring Security 3.2以降にバージョンアップする場合は、以下のクラスを組み合わせて使用するように変更する必要がある。

    * ``ConcurrentSessionControlAuthenticationStrategy`` (Spring Security 3.2で追加)
    * ``RegisterSessionAuthenticationStrategy`` (Spring Security 3.2で追加)
    * ``org.springframework.security.web.authentication.session.SessionFixationProtectionStrategy``

    具体的な定義方法については、
    `Spring Security Reference -Web Application Security (Concurrency Control)- <http://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/reference/htmlsingle/#concurrent-sessions>`_ のサンプルコードを参考にされたい。

|

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

.. note::

    \ ``exceptionMappings``\ プロパティに定義した例外が発生した場合、例外にマッピングした遷移先にリダイレクトされるが、
    発生した例外オブジェクトがセッションスコープに格納されないため、Spring Securityが生成したエラーメッセージを画面に表示する事ができない。

    そのため、遷移先の画面で表示するエラーメッセージは、リダイレクト先の処理(Controller又はViewの処理)で生成する必要がある。

    また、以下のプロパティを参照する処理が呼び出されないため、設定値を変更しても動作が変わらないという点を補足しておく。

    * ``useForward``
    * ``allowSessionCreation``


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
       | 指定がない場合、「/j_spring_security_logout」がデフォルトのパスになる。

       | **本ガイドラインでは、上記のデフォルト値「/j_spring_security_logout」を使用せず、システム独自の値に変更することを推奨する。** この例では”/logout”を指定している。
   * - | (2)
     - | \ ``logout-success-url``\ 属性に、ログアウト後の遷移先パスを指定する。
       | 指定がない場合、「/」がデフォルトのパスになる。

       | 本属性を指定した場合、\ ``success-handler-ref``\ 属性を指定すると起動時にエラーとなる。
   * - | (3)
     - | \ ``invalidate-session``\ 属性に、ログアウト時にセッションを破棄するかを設定する。
       | デフォルトは\ ``true``\ である。
       | \ ``true``\ の場合、ログアウト時にセッションが破棄される。
   * - | (4)
     - | \ ``delete-cookies``\ 属性に、ログアウト時に削除するクッキー名を列挙する。
       | 複数記述する場合は「,」で区切る。
   * - | (5)
     - | \ ``success-handler-ref``\ 属性に、ログアウト成功後に呼び出すハンドラクラスを指定する。

       | 本属性を指定した場合、\ ``logout-success-url``\ 属性を指定すると起動時にエラーとなる。

.. warning:: **Spring Security のデフォルト値「/j_spring_security_logout」の使用を推奨しない理由**

    デフォルト値を使用している場合、そのアプリケーションが、Spring Securityを使用していることについて、露見してしまう。
    そのため、Spring Securityの脆弱性が発見された場合、脆弱性をついた攻撃を受けるリスクが高くなる。
    前述のリスクを避けるためにも、デフォルト値を使用しないことを推奨する。

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

| 「\ `Remeber Me <http://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/reference/htmlsingle/#remember-me>`_\ 」とは、websiteに頻繁にアクセスするユーザの利便性を、高めるための機能の一つとして、
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

上記以外の属性については、\ `Spring Securityのマニュアル <http://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/reference/htmlsingle/#nsa-remember-me>`_\ を参照されたい。

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
    public String view(@AuthenticationPrincipal ReservationUserDetails userDetails, Model model) {
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

|

\ ``AuthenticationProvider``\ の拡張
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityから提供されている\ `認証プロバイダ <http://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/apidocs/org/springframework/security/authentication/AuthenticationProvider.html>`_\ で対応できない業務要件がある場合、
\ ``org.springframework.security.authentication.AuthenticationProvider``\ インタフェースを実装したクラスを作成する必要がある。

ここでは、ユーザ名、パスワード、\ **会社識別子(独自の認証パラメータ)**\ の3つのパラメータを使用してDB認証を行うための拡張例を示す。

.. figure:: ./images/Authentication_HowToExtends_LoginForm.png
   :alt: Authentication_HowToExtends_LoginForm
   :width: 50%

上記の要件を実現するためには、以下に示すクラスを作成する必要がある。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - ユーザ名、パスワード、会社識別子(独自の認証パラメータ)を保持する\ ``org.springframework.security.core.Authentication``\ インタフェースの実装クラス。

        ここでは、\ ``org.springframework.security.authentication.UsernamePasswordAuthenticationToken``\ クラスを継承して作成する。
    * - | (2)
      - ユーザ名、パスワード、会社識別子(独自の認証パラメータ)を使用してDB認証を行う\ ``org.springframework.security.authentication.AuthenticationProvider``\ の実装クラス。

        ここでは、\ ``org.springframework.security.authentication.dao.DaoAuthenticationProvider``\ クラスを継承して作成する。
    * - | (3)
      - ユーザ名、パスワード、会社識別子(独自の認証パラメータ)をリクエストパラメータから取得して、\ ``AuthenticationManager``\ (\ ``AuthenticationProvider``\ )に渡す\ ``Authentication``\ を生成するためのサーブレットフィルタクラス。

        ここでは、\ ``org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter``\ クラスを継承して作成する。

.. tip::

    ここでは、認証用のパラメータとして独自のパラメータを追加する例にしているため、
    \ ``Authentication``\ インタフェースの実装クラスと\ ``Authentication``\ を生成するためのサーブレットフィルタクラスの拡張が必要となる。

    ユーザ名とパスワードのみで認証する場合は、\ ``AuthenticationProvider``\ インタフェースの実装クラスを作成するだけで、
    認証処理を拡張することができる。

|

\ ``UsernamePasswordAuthenticationToken``\ の拡張
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ここでは、\ ``UsernamePasswordAuthenticationToken``\ クラスを継承し、
ユーザ名とパスワードに加えて、会社識別子(独自の認証パラメータ)を保持するクラスを作成する。

.. code-block:: java

    // import omitted
    public class CompanyIdUsernamePasswordAuthenticationToken extends
        UsernamePasswordAuthenticationToken {

        private static final long serialVersionUID = SpringSecurityCoreVersion.SERIAL_VERSION_UID;

        // (1)
        private final String companyId;

        // (2)
        public CompanyIdUsernamePasswordAuthenticationToken(
                Object principal, Object credentials, String companyId) {
            super(principal, credentials);

            this.companyId = companyId;
        }

        // (3)
        public CompanyIdUsernamePasswordAuthenticationToken(
                Object principal, Object credentials, String companyId,
                Collection<? extends GrantedAuthority> authorities) {
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
     - 会社識別子を保持するフィールドを作成する。
   * - | (2)
     - 認証前の情報(リクエストパラメータで指定された情報)を保持するインスタンスを作成する際に使用するコンストラクタを作成する。
   * - | (3)
     - | 認証済みの情報を保持するインスタンスを作成する際に使用するコンストラクタを作成する。
       | 親クラスのコンストラクタの引数に認可情報を渡すことで、認証済みの状態となる。

|

\ ``DaoAuthenticationProvide``\ の拡張
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ここでは、\ ``DaoAuthenticationProvider``\ クラスを継承し、
ユーザ名、パスワード、会社識別子を使用してDB認証を行うクラスを作成する。

.. code-block:: java

    // import omitted
    public class CompanyIdUsernamePasswordAuthenticationProvider extends
        DaoAuthenticationProvider {

        // omitted

        @Override
        protected void additionalAuthenticationChecks(UserDetails userDetails,
                UsernamePasswordAuthenticationToken authentication)
                throws AuthenticationException {

            // (1)
            super.additionalAuthenticationChecks(userDetails, authentication);

            // (2)
            CompanyIdUsernamePasswordAuthenticationToken companyIdUsernamePasswordAuthentication =
                (CompanyIdUsernamePasswordAuthenticationToken) authentication;
            String requestedCompanyId = companyIdUsernamePasswordAuthentication.getCompanyId();
            String companyId = ((SampleUserDetails) userDetails)
                    .getAccount().getCompanyId();
            if (!companyId.equals(requestedCompanyId)) {
                throw new BadCredentialsException(messages.getMessage(
                        "AbstractUserDetailsAuthenticationProvider.badCredentials",
                        "Bad credentials"));
            }
        }

        @Override
        protected Authentication createSuccessAuthentication(Object principal,
                Authentication authentication, UserDetails user) {
            String companyId = ((SampleUserDetails) user).getAccount()
                    .getCompanyId();
            // (3)
            return new CompanyIdUsernamePasswordAuthenticationToken(user,
                    authentication.getCredentials(), companyId,
                    user.getAuthorities());
        }

        @Override
        public boolean supports(Class<?> authentication) {
            // (4)
            return CompanyIdUsernamePasswordAuthenticationToken.class
                    .isAssignableFrom(authentication);
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - 親クラスのメソッドを呼び出し、Spring Securityが提供しているチェック処理を実行する。

       このタイミングでパスワード認証が行われる。
   * - | (2)
     - パスワード認証が成功した場合は、会社識別子(独自の認証パラメータ)の妥当性をチェックする。

       上記例では、リクエストされた会社識別子とテーブルに保持している会社識別子が一致するかをチェックしている。
   * - | (3)
     - パスワード認証及び独自の認証処理が成功した場合は、認証済み状態の\ ``CompanyIdUsernamePasswordAuthenticationToken``\ を作成して返却する。
   * - | (4)
     - \ ``CompanyIdUsernamePasswordAuthenticationToken``\ にキャスト可能な\ ``Authentication``\ が指定された場合に、
       本クラスを使用して認証処理を行うようにする。

.. tip::

    ユーザの存在チェック、ユーザの状態チェック(無効ユーザ、ロック中ユーザ、利用期限切れユーザなどのチェック)は、
    \ ``additionalAuthenticationChecks``\ メソッドが呼び出される前に、親クラスの処理として行われる。

|

.. _authentication_custom_usernamepasswordauthenticationfilter:

\ ``UsernamePasswordAuthenticationFilter``\ の拡張
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ここでは、\ ``UsernamePasswordAuthenticationFilter``\ クラスを継承し、
認証情報(ユーザ名、パスワード、会社識別子)を\ ``AuthenticationProvider``\ に引き渡すためのサーブレットフィルタクラスを作成する。

\ ``attemptAuthentication``\ メソッドの実装は、\ ``UsernamePasswordAuthenticationFilter``\ クラスのメソッドをコピーして、カスタマイズしたものである。

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

            // (1)
            // Obtain UserName, Password, CompanyId
            String username = super.obtainUsername(request);
            String password = super.obtainPassword(request);
            String companyId = obtainCompanyId(request);
            if (username == null) {
                username = "";
            } else {
                username = username.trim();
            }
            if (password == null) {
                password = "";
            }
            CompanyIdUsernamePasswordAuthenticationToken authRequest =
                new CompanyIdUsernamePasswordAuthenticationToken(username, password, companyId);

            // Allow subclasses to set the "details" property
            setDetails(request, authRequest);

            return this.getAuthenticationManager().authenticate(authRequest); // (2)
        }

        // (3)
        protected String obtainCompanyId(HttpServletRequest request) {
            return request.getParameter("companyid");
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - リクエストパラメータから取得した認証情報(ユーザ名、パスワード、会社識別子)より、\ ``CompanyIdUsernamePasswordAuthenticationToken``\ のインスタンスを生成する。
   * - | (2)
     - リクエストパラメータで指定された認証情報(\ ``CompanyIdUsernamePasswordAuthenticationToken``\ のインスタンス)を指定して、
       \ ``org.springframework.security.authentication.AuthenticationManager``\ の\ ``authenticate``\ メソッドを呼び出す。

       \ ``AuthenticationManager``\ のメソッドを呼び出すと、\ ``AuthenticationProvider``\ の認証処理が呼び出される仕組みになっている。
   * - | (3)
     - 会社識別子は、\ ``"companyid"``\ というリクエストパラメータより取得する。

.. note:: **認証情報の入力チェックについて**

    DBサーバへの負荷軽減等で、あきらかな入力誤りに対しては、事前にチェックを行いたい場合がある。
    その場合は、\ :ref:`authentication_custom_usernamepasswordauthenticationfilter`\ のように、
    \ ``UsernamePasswordAuthenticationFilter``\ を拡張することで、入力チェック処理を行うことができる。

    なお、上記例では入力チェックは行っていない。

.. todo::

    認証情報の入力チェックは、Controllerクラスでリクエストをハンドリングして、Bean Validationを使用して行う事も可能である。

    Bean Validationを使用した入力チェックの方法については、今後追加する予定である。

|

拡張した認証処理の適用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ユーザ名、パスワード、会社識別子(独自の認証パラメータ)を使用したDB認証機能をSpring Securityに適用する。

``spring-security.xml``

.. code-block:: xml

    <!-- omitted -->

    <!-- (1) -->
    <sec:http
        auto-config="false"
        use-expressions="true"
        entry-point-ref="loginUrlAuthenticationEntryPoint">

        <!-- omitted -->

        <!-- (2) -->
        <sec:custom-filter
            position="FORM_LOGIN_FILTER"
            ref="companyIdUsernamePasswordAuthenticationFilter" />

        <!-- omitted -->

        <sec:csrf token-repository-ref="csrfTokenRepository" />

        <sec:logout
            logout-url="/logout"
            logout-success-url="/login"
            delete-cookies="JSESSIONID" />

        <!-- omitted -->

        <sec:intercept-url pattern="/login" access="permitAll" />
        <sec:intercept-url pattern="/**" access="isAuthenticated()" />

        <!-- omitted -->

    </sec:http>

    <!-- (3) -->
    <bean id="loginUrlAuthenticationEntryPoint"
        class="org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint">
        <constructor-arg value="/login" />
    </bean>

    <!-- (4) -->
    <bean id="companyIdUsernamePasswordAuthenticationFilter"
        class="com.example.app.common.security.CompanyIdUsernamePasswordAuthenticationFilter">
        <!-- (5) -->
        <property name="requiresAuthenticationRequestMatcher">
            <bean class="org.springframework.security.web.util.matcher.AntPathRequestMatcher">
                <constructor-arg index="0" value="/authentication" />
                <constructor-arg index="1" value="POST" />
            </bean>
        </property>
        <!-- (6) -->
        <property name="authenticationManager" ref="authenticationManager" />
        <!-- (7) -->
        <property name="sessionAuthenticationStrategy" ref="sessionAuthenticationStrategy" />
        <!-- (8) -->
        <property name="authenticationFailureHandler">
            <bean class="org.springframework.security.web.authentication.SimpleUrlAuthenticationFailureHandler">
                <constructor-arg value="/login?error=true" />
            </bean>
        </property>
        <!-- (9) -->
        <property name="authenticationSuccessHandler">
            <bean class="org.springframework.security.web.authentication.SimpleUrlAuthenticationSuccessHandler" />
        </property>
    </bean>

    <!-- (6') -->
    <sec:authentication-manager alias="authenticationManager">
        <sec:authentication-provider ref="companyIdUsernamePasswordAuthenticationProvider" />
    </sec:authentication-manager>
    <bean id="companyIdUsernamePasswordAuthenticationProvider"
        class="com.example.app.common.security.CompanyIdUsernamePasswordAuthenticationProvider">
        <property name="userDetailsService" ref="sampleUserDetailsService" />
        <property name="passwordEncoder" ref="passwordEncoder" />
    </bean>

    <!-- (7') -->
    <bean id="sessionAuthenticationStrategy"
        class="org.springframework.security.web.authentication.session.CompositeSessionAuthenticationStrategy">
        <constructor-arg>
            <util:list>
                <bean class="org.springframework.security.web.csrf.CsrfAuthenticationStrategy">
                    <constructor-arg ref="csrfTokenRepository" />
                </bean>
                <bean class="org.springframework.security.web.authentication.session.SessionFixationProtectionStrategy" />
            </util:list>
        </constructor-arg>
    </bean>

    <bean id="csrfTokenRepository"
        class="org.springframework.security.web.csrf.HttpSessionCsrfTokenRepository" />


    <!-- omitted -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - \ ``custom-filter``\ 要素を使用して"FORM_LOGIN_FILTER"を差し替える場合は、\ ``http``\ 要素の属性に、以下の設定を行う必要がある。

        * 自動設定を使用することができないため、\ ``auto-config="false"``\ を指定するか、\ ``auto-config``\ 属性を削除する。
        * \ ``form-login``\ 要素を使用できないため、\ ``entry-point-ref``\ 属性を使用して、使用する\ ``AuthenticationEntryPoint``\ を明示的に指定する。
    * - | (2)
      - \ ``custom-filter``\ 要素を使用して"FORM_LOGIN_FILTER"を差し替える。

        \ ``custom-filter``\ 要素の\ ``position``\ 属性に\ ``"FORM_LOGIN_FILTER"``\を指定し、\ ``ref``\ 属性に拡張したサーブレットフィルタのbean IDを指定する。
    * - | (3)
      - \ ``http``\ 要素の\ ``entry-point-ref``\ 属性に指定する\ ``AuthenticationEntryPoint``\ のbeanを定義する。

        ここでは、\ ``form-login``\ 要素を指定した際の使用される\ ``org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint``\ クラスのbeanを定義している。
    * - | (4)
      - "FORM_LOGIN_FILTER"として使用するサーブレットフィルタのbeanを定義する。

        ここでは、拡張したサーブレットフィルタクラス(\ ``CompanyIdUsernamePasswordAuthenticationFilter``\ )のbeanを定義している。
    * - | (5)
      - \ ``requiresAuthenticationRequestMatcher``\ プロパティに、認証処理を行うリクエストを検出するための\ ``RequestMatcher``\ インスタンスを指定する。

        ここでは、\ ``/authentication``\ というパスにリクエストがあった場合に認証処理を行うように設定している。
        これは、\ ``form-login``\ 要素の\ ``login-processing-url``\ 属性に\ ``"/authentication"``\ を指定したのと同義である。
    * - | (6)
      - \ ``authenticationManager``\ プロパティに、\ ``authentication-manager``\ 要素の\ ``alias``\ 属性に設定した値を指定する。

        \ ``authentication-manager``\ 要素の\ ``alias``\ 属性を指定すると、
        Spring Securityが生成した\ ``AuthenticationManager``\ のbeanを、他のbeanへDIすることができる様になる。
    * - | (6')
      - Spring Securityが生成する\ ``AuthenticationManager``\ に対して、拡張した\ ``AuthenticationProvider``\ (\ ``CompanyIdUsernamePasswordAuthenticationProvider``\ )を設定する。
    * - | (7)
      - \ ``sessionAuthenticationStrategy``\ プロパティに、認証成功時のセッションの取扱いを制御するコンポーネント(\ ``SessionAuthenticationStrategy``\ )のbeanを指定する。

    * - | (7')
      - 認証成功時のセッションの取扱いを制御するコンポーネント(\ ``SessionAuthenticationStrategy``\ )のbeanを定義する。

        ここでは、Spring Securityから提供されている、

        * CSRFトークンを作り直すコンポーネント(\ ``CsrfAuthenticationStrategy``\ )
        * セッション・フィクセーション攻撃を防ぐために新しいセッションを生成するコンポーネント(\ ``SessionFixationProtectionStrategy``\ )

        を有効化している。
    * - | (8)
      - | \ ``authenticationFailureHandler``\ プロパティに、認証失敗時に呼ばれるハンドラクラスを指定する。
    * - | (9)
      - | \ ``authenticationSuccessHandler``\ プロパティに、認証成功時に呼ばれるハンドラクラスを指定する。

.. note::

    \ ``auto-config="false"``\ を指定した場合、\ ``<sec:http-basic>``\ 要素と\ ``<sec:logout>``\ 要素は、明示的に定義しないと有効にならない。

|

ログインフォームの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ここでは、\ :ref:`form-login-JSP`\ で紹介した画面(JSP)に対して、会社識別子を追加する。

.. code-block:: jsp
    :emphasize-lines: 5-6

    <form:form action="${pageContext.request.contextPath}/authentication" method="post">
        <!-- omitted -->
        <span>User Id</span><br>
        <input type="text" id="username" name="j_username"><br>
        <span>Company Id</span><br>
        <input type="text" id="companyid" name="companyid"><br>  <!-- (1) -->
        <span>Password</span><br>
        <input type="password" id="password" name="j_password"><br>
        <!-- omitted -->
    </form:form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 会社識別子の入力フィールド名に、\ ``"companyid"``\ を指定する。

|

Appendix
--------------------------------------------------------------------------------

遷移先の指定が可能な認証成功ハンドラ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityを使用した認証では、認証に成功した場合は、

* bean定義ファイル(\ ``spring-security.xml``\ )に記述したパス(\ ``<form-login>``\ 要素の\ ``default-target-url``\ 属性に指定したパス)
* ログイン前にアクセスした「認証が必要な保護ページ」を表示するためのパス

に遷移する。

共通ライブラリでは、Spring Securityが提供している機能に加えて、
遷移先のパスをリクエストパラメータで指定できるクラス(\ ``org.terasoluna.gfw.security.web.RedirectAuthenticationHandler``\ )を提供している。

\ ``RedirectAuthenticationHandler``\ は、以下のような仕組みを実現するために作成されたクラスである。

* ページを表示するためにログインを行う必要がある
* ログイン後の遷移先のページをJSP側(遷移元のJSP)で指定したい

.. figure:: ./images/Authentication_Appendix_ScreenFlow.png
   :alt: Authentication_Appendix_Screen_Flow
   :width: 70%
   :align: center

   **Picture - Screen_Flow**

| \ ``RedirectAuthenticationHandler``\ の使用例を、下記に示す。

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
     - | hidden項目として、「ログイン成功後に遷移するページのURL」を設定する。
       | hidden項目のフィールド名(リクエストパラメータ名)は「\ ``redirectTo``\ 」を指定する。
       |
       | フィールド名(リクエストパラメータ名)は、\ ``RedirectAuthenticationHandler``\ の\ ``targetUrlParameter``\ プロパティの値と一致させる必要がある。

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
     - | hidden項目として、遷移元画面からリクエストパラメータで渡された「ログイン成功後に遷移するページのURL」を設定する。
       | hidden項目のフィールド名(リクエストパラメータ名)は「\ ``redirectTo``\ 」を指定する。
       |
       | フィールド名(リクエストパラメータ名)は、\ ``RedirectAuthenticationHandler``\ の\ ``targetUrlParameter``\ プロパティの値と一致させる必要がある。

**Spring Security 設定ファイル**

.. code-block:: xml

  <sec:http auto-config="true">
      <!-- omitted -->
      <!-- (1) -->
      <sec:form-login
          login-page="/login"
          login-processing-url="/authentication"
          authentication-failure-handler-ref="authenticationFailureHandler"
          authentication-success-handler-ref="authenticationSuccessHandler" />
      <!-- omitted -->
  </sec:http>

  <!-- (2) -->
  <bean id="authenticationSuccessHandler"
      class="org.terasoluna.gfw.security.web.RedirectAuthenticationHandler">
  </bean>

  <!-- (3) -->
  <bean id="authenticationFailureHandler"
      class="org.springframework.security.web.authentication.ExceptionMappingAuthenticationFailureHandler">
      <property name="defaultFailureUrl" value="/login?error=true"/> <!-- (4) -->
      <property name="useForward" value="true"/> <!-- (5) -->
  </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``authentication-failure-handler-ref``\ (認証エラー時のハンドラ設定)と\ ``authentication-success-handler-ref``\ (認証成功時のハンドラ設定)のBeanIdを指定する。
   * - | (2)
     - | \ ``authentication-success-handler-ref``\ から参照されるbeanとして\ ``org.terasoluna.gfw.security.web.RedirectAuthenticationHandler``\ を定義する。
   * - | (3)
     - | \ ``authentication-failure-handler-ref``\ から参照されるbeanとして\ ``org.springframework.security.web.authentication.ExceptionMappingAuthenticationFailureHandler``\ を定義する。
   * - | (4)
     - | 認証失敗時の遷移先パスを指定する。
       | 上記例では、ログイン画面のパスと認証エラー後の遷移であることを示すクエリ(\ ``error=true``\ )を設定している。
   * - | (5)
     - | **本機能を使用する場合はuseForwardをtrueに指定する必要がある。**
       | \ ``true``\に指定することで、認証失敗時に表示する画面(ログイン画面)に遷移する際に、RedirectではなくForwardが使用される。

       | これは、認証処理を行うリクエストのリクエストパラメータの中に「ログイン成功後に遷移するページのURL」を含める必要があるためである。
       | Redirectを使用して認証エラー画面を表示してしまうと、「ログイン成功後に遷移するページのURL」がリクエストパラメータから引き継ぎことが出来ないため、ログインが成功しても指定した画面に遷移することが出来ない。
       | この事象を回避するためには、Forwardを使用して「ログイン成功後に遷移するページのURL」をリクエストパラメータから引き継げるようにしておく必要がある。

.. tip::

  \ ``RedirectAuthenticationHandler``\ は、オープンリダイレクタ脆弱性対策が施されているため、
  「http://google.com」のような外部サイトへ遷移をすることはできない。
  外部サイトへ移動したい場合は、\ ``org.springframework.security.web.RedirectStrategy``\ を実装したクラスを作成し、
  \ ``RedirectAuthenticationHandler``\ の\ ``targetUrlParameterRedirectStrategy``\ プロパティにインジェクションすることで実現する事ができる。

  拡張する際の注意点としては、\ ``redirectTo``\ の値を改竄されても問題が発生しないようにする必要がある。
  たとえば、以下のような対策が考えられる。

  * 遷移先のURLを直接指定するのではなく、ページ番号などのIDを指定してIDに対応するURLにリダイレクトする。
  * 遷移先のURLをチェックし、ホワイトリストに一致するURLのみリダイレクトする。

.. raw:: latex

   \newpage

