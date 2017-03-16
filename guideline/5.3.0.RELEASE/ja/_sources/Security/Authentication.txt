.. _SpringSecurityAuthentication:

認証
================================================================================

.. only:: html

 .. contents:: 目次
    :local:

.. _SpringSecurityAuthenticationOverview:

Overview
--------------------------------------------------------------------------------
本節では、Spring Securityが提供している認証機能について説明する。

認証処理は、アプリケーションを利用するユーザーの正当性を確認するための処理である。

ユーザーの正当性を確認するためのもっとも標準的な方法は、アプリケーションを使用できるユーザーをデータストアに登録しておき、
利用者が入力した認証情報（ユーザー名とパスワードなど）と照合する方法である。
ユーザーの情報を登録しておくデータストアにはリレーショナルデータベースを利用するのが一般的だが、ディレクトリサービスや外部システムなどを利用するケースもある。

また、利用者に認証情報を入力してもらう方式もいくつか存在する。
HTMLの入力フォームを使う方式やRFCで定められているHTTP標準の認証方式(Basic認証やDigest認証など)を利用するのが一般的だが、
OpenID認証やシングルサインオン認証などの認証方式を利用するケースもある。

本節では、HTMLの入力フォームで入力した認証情報とリレーショナルデータベースに格納されているユーザー情報を照合して認証処理を行う実装例を紹介しながら、
Spring Securityの認証機能の使い方を説明する。

|

認証処理のアーキテクチャ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityは、以下のような流れで認証処理を行う。

.. figure:: ./images_Authentication/AuthenticationArchitecture.png
    :width: 100%

    **認証処理のアーキテクチャ**

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | クライアントは、認証処理を行うパスに対して資格情報（ユーザー名とパスワード）を指定してリクエストを送信する。
    * - | (2)
      - | Authentication Filterは、リクエストから資格情報を取得して、\ ``AuthenticationManager``\ クラスの認証処理を呼び出す。
    * - | (3)
      - | \ ``ProviderManager``\ (デフォルトで使用される\ ``AuthenticationManager``\ の実装クラス)は、実際の認証処理を\ ``AuthenticationProvider``\ インタフェースの実装クラスに委譲する。

|

.. _SpringSecurityAuthenticationFilter:

Authentication Filter
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Authentication Filterは、認証方式に対する実装を提供するサーブレットフィルタである。
Spring Securityがサポートしている主な認証方式は以下の通り。

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: **Spring Securityが提供している主なAuthentication Filter**
    :header-rows: 1
    :widths: 25 75

    * - クラス名
      - 説明
    * - | \ ``UsernamePasswordAuthenticationFilter``\
      - | フォーム認証用のサーブレットフィルタクラスで、HTTPリクエストのパラメータから資格情報を取得する。
    * - | \ ``BasicAuthenticationFilter``\
      - | Basic認証用のサーブレットフィルタクラスで、HTTPリクエストの認証ヘッダから資格情報を取得する。
    * - | \ ``DigestAuthenticationFilter``\
      - | Digest認証用のサーブレットフィルタクラスで、HTTPリクエストの認証ヘッダから資格情報を取得する。
    * - | \ ``RememberMeAuthenticationFilter``\
      - | Remember Me認証用のサーブレットフィルタクラスで、HTTPリクエストのCookieから資格情報を取得する。
        | Remember Me認証を有効にすると、ブラウザを閉じたりセッションタイムアウトが発生しても、ログイン状態を保つことができる。

これらのサーブレットフィルタは、 :ref:`SpringSecurityProcess`\ で紹介したAuthentication Filterの１つである。

.. note::

    Spring Securityによってサポートされていない認証方式を実現する必要がある場合は、
    認証方式を実現するための\ ``Authentication Filter``\ を作成し、Spring Securityに組み込むことで実現することが可能である。

|

AuthenticationManager
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``AuthenticationManager``\ は、認証処理を実行するためのインタフェースである。
Spring Securityが提供するデフォルト実装(\ ``ProviderManager``\ )では、
実際の認証処理は\ ``AuthenticationProvider``\ に委譲し、\ ``AuthenticationProvider``\ で行われた認証処理の処理結果をハンドリングする仕組みになっている。

|

AuthenticationProvider
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``AuthenticationProvider``\ は、認証処理の実装を提供するためのインタフェースである。
Spring Securityが提供している主な\ ``AuthenticationProvider``\の実装クラスは以下の通り。

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: **Spring Securityが提供している主なAuthenticationProvider**
    :header-rows: 1
    :widths: 25 75

    * - クラス名
      - 説明
    * - | \ ``DaoAuthenticationProvider``\
      - | データストアに登録しているユーザーの資格情報とユーザーの状態をチェックして認証処理を行う実装クラス。
        | チェックで必要となる資格情報とユーザーの状態は\ ``UserDetails``\ というインタフェースを実装しているクラスから取得する。

.. note::

    Spring Securityが提供していない認証処理を実現する必要がある場合は、
    認証処理を実現するための\ ``AuthenticationProvider``\を作成し、Spring Securityに組み込むことで実現することが可能である。

|

.. _howtouse_springsecurity:

How to use
--------------------------------------------------------------------------------

認証機能を使用するために必要となるbean定義例や実装方法について説明する。

本項では :ref:`SpringSecurityAuthenticationOverview`\ で説明したとおり、
HTMLの入力フォームで入力した認証情報とリレーショナルデータベースに格納されているユーザー情報を照合して認証処理を行う方法について説明する。

.. _form-login:

フォーム認証
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityは、以下のような流れでフォーム認証を行う。

.. figure:: ./images_Authentication/AuthenticationForm.png
    :width: 100%

    **フォーム認証の仕組み**

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | クライアントは、フォーム認証を行うパスに対して資格情報（ユーザー名とパスワード）をリクエストパラメータとして送信する。
    * - | (2)
      - | \ ``UsernamePasswordAuthenticationFilter``\ クラスは、リクエストパラメータから資格情報を取得して、\ ``AuthenticationManager``\ の認証処理を呼び出す。
    * - | (3)
      - | \ ``UsernamePasswordAuthenticationFilter``\ クラスは、\ ``AuthenticationManager``\ から返却された認証結果をハンドリングする。
        | 認証処理が成功した場合は \ ``AuthenticationSuccessHandler``\ のメソッドを、認証処理が失敗した場合は\ ``AuthenticationFailureHandler``\ のメソッドを呼び出し画面遷移を行う。

|

.. _form-login-usage:

フォーム認証の適用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

フォーム認証を使用する場合は、以下のようなbean定義を行う。

* spring-security.xmlの定義例

.. code-block:: xml

    <sec:http>
        <sec:form-login />    <!-- (1) -->
        <!-- omitted -->
    </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``<sec:form-login>``\ タグを定義することで、フォーム認証が有効になる。

.. tip:: **auto-config属性について**

    \ ``<sec:http>``\ には、フォーム認証(\ ``<sec:form-login>``\ タグ)、Basic認証(\ ``<sec:http-basic>``\ タグ)、ログアウト(\ ``<sec:logout>``\ タグ)に対するコンフィギュレーションを自動で行うか否かを指定する\ ``auto-config``\ 属性が用意されている。
    デフォルト値は\ ``false``\ (自動でコンフィギュレーションしない)となっており、Spring Securityのリファレンスドキュメントでもデフォルト値の使用が推奨されている。

    本ガイドラインでも、明示的にタグを指定するスタイルを推奨する。

     .. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
     .. list-table::
         :header-rows: 1
         :widths: 25 75

         * - 要素名
           - 説明
         * - | ``<form-login>``\
           - | フォーム認証処理を行うSecurity Filter(\ ``UsernamePasswordAuthenticationFilter``\ )が適用される。
         * - | \ ``<http-basic>``\
           - | RFC1945に準拠したBasic認証を行うSecurity Filter(\ ``BasicAuthenticationFilter``\ )が適用される。
             | 詳細な利用方法は、\ `BasicAuthenticationFilterのJavaDoc <http://docs.spring.io/spring-security/site/docs/4.1.4.RELEASE/apidocs/org/springframework/security/web/authentication/www/BasicAuthenticationFilter.html>`_\ を参照されたい。
         * - | \ ``<logout>``\
           - | ログアウト処理を行うSecurity Filter(\ ``LogoutFilter``\ )が適用される。
             | ログアウト処理の詳細については、「\ :ref:`SpringSecurityAuthenticationLogout`\ 」を参照されたい。

    なお、 ``auto-config``\を定義しない場合は、フォーム認証(\ ``<sec:form-login>``\ タグ)、もしくはBasic認証(\ ``<sec:http-basic>``\ タグ)を定義する必要がある。
    これは、ひとつの\ ``SecurityFilterChain``\(\ ``<sec:http>``\)内には、ひとつ以上のAuthentication FilterのBean定義が必要であるという、Spring Securityの仕様をみたすためである。

.. _form-login-default-operation:

デフォルトの動作
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityのデフォルトの動作では、\ ``"/login"``\ に対してGETメソッドでアクセスするとSpring Securityが用意しているデフォルトのログインフォームが表示され、
ログインボタンを押下すると\ ``"/login"``\ に対してPOSTメソッドでアクセスして認証処理を行う。

|

.. _SpringSecurityAuthenticationLoginForm:

ログインフォームの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Spring Securityはフォーム認証用のログインフォームをデフォルトで提供しているが、そのまま利用するケースは少ない。
ここでは、自身で作成したログインフォームをSpring Securityに適用する方法を説明する。

まず、ログインフォームを表示するためのJSPを作成する。
ここでは、Spring MVCでリクエストをうけてログインフォームを表示する際の実装例になっている。

* ログインフォームを表示するためのJSPの作成例(xxx-web/src/main/webapp/WEB-INF/views/login/loginForm.jsp)

.. code-block:: jsp

    <%@ page contentType="text/html;charset=UTF-8" pageEncoding="UTF-8" %>
    <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
    <%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags" %>
    <%-- omitted --%>
    <div id="wrapper">
        <h3>Login Screen</h3>
        <%-- (1) --%>
        <c:if test="${param.containsKey('error')}">
            <t:messagesPanel messagesType="error"
                messagesAttributeName="SPRING_SECURITY_LAST_EXCEPTION"/> <%-- (2) --%>
        </c:if>
        <form:form action="${pageContext.request.contextPath}/login" method="post"> <%-- (3) --%>
            <table>
                <tr>
                    <td><label for="username">User Name</label></td>
                    <td><input type="text" id="username" name="username"></td>
                </tr>
                <tr>
                    <td><label for="password">Password</label></td>
                    <td><input type="password" id="password" name="password"></td>
                </tr>
                <tr>
                    <td>&nbsp;</td>
                    <td><button>Login</button></td>
                </tr>
            </table>
        </form:form>
    </div>
    <%-- omitted --%>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 認証エラーを表示するためのエリア。
    * - | (2)
      - | 認証エラー時に出力させる例外メッセージを出力する。
        | 共通ライブラリで提供している\ ``<t:messagesPanel>``\ タグを使用して出力することを推奨する。
        | \ ``<t:messagesPanel>``\ タグの使用方法については、「\ :doc:`../ArchitectureInDetail/WebApplicationDetail/MessageManagement`\ 」を参照されたい。
        | なお、認証エラーが発生した場合は、セッション又はリクエストスコープに\ ``"SPRING_SECURITY_LAST_EXCEPTION"``\ という属性名で例外オブジェクトが格納される。
    * - | (3)
      - | ユーザー名とパスワードを入力するためのログインフォーム。
        | ここではユーザー名を\ ``username``\、パスワードを\ ``passowrd``\ というリクエストパラメータで送信する。
        | また、\ ``<form:form>``\ を使用することで、CSRF対策用のトークン値がリクエストパラメータで送信される。
        | CSRF対策については、「:ref:`SpringSecurityCsrf`」で説明する。

|

つぎに、作成したログインフォームをSpring Securityに適用する。

* spring-security.xmlの定義例

.. code-block:: xml

    <sec:http>
      <sec:form-login 
          login-page="/login/loginForm"
          login-processing-url="/login"  /> <!-- (1)(2) -->
      <sec:intercept-url pattern="/login/**" access="permitAll"/>  <!-- (3) -->
      <sec:intercept-url pattern="/**" access="isAuthenticated()"/> <!-- (4) -->
    </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``login-page``\ 属性にログインフォームを表示するためのパスを指定する。
        | 匿名ユーザーが認証を必要とするWebリソースにアクセスした場合は、この属性に指定したパスにリダイレクトしてログインフォームを表示する。
        | ここでは、Spring MVCでリクエストを受けてログインフォームを表示している。
        | 詳細は 「:ref:`spring-security-authentication-mvc`」を参照されたい。
    * - | (2)
      - | \ ``login-processing-url``\ 属性に認証処理を行うためのパスを指定する。
        | デフォルトのパスも\ ``"/login"``\ であるが、ここでは明示的に指定することとする。
    * - | (3)
      - | ログインフォームが格納されている\ ``/login``\ パス配下に対し、すべてのユーザーがアクセスできる権限を付与する。
        | Webリソースに対してアクセスポリシーの指定方法については、「\ :ref:`SpringSecurityAuthorization`\ 」を参照されたい。
    * - | (4)
      - | アプリケーションで扱うWebリソースに対してアクセス権を付与する。
        | 上記例では、Webアプリケーションのルートパスの配下に対して、認証済みユーザーのみがアクセスできる権限を付与している。
        | Webリソースに対してアクセスポリシーの指定方法については、「\ :ref:`SpringSecurityAuthorization`\ 」を参照されたい。

.. note:: **Spring Security 4.0における変更**

    Spring Security 4.0から、以下の設定のデフォルト値が変更されている

    * username-parameter
    * password-parameter
    * login-processing-url
    * authentication-failure-url 

|

.. _SpringSecurityAuthenticationScreenFlowOnSuccess:

認証成功時のレスポンス
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityは、認証成功時のレスポンスを制御するためのコンポーネントとして、
\ ``AuthenticationSuccessHandler``\ というインタフェースと実装クラスを提供している。

.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table:: **AuthenticationSuccessHandlerの実装クラス**
    :header-rows: 1
    :widths: 35 65

    * - 実装クラス
      - 説明
    * - | \ ``SavedRequestAwareAuthenticationSuccessHandler``\
      - | 認証前にアクセスを試みたURLにリダイレクトを行う実装クラス。
        | **デフォルトで使用される実装クラス。**
    * - | \ ``SimpleUrlAuthenticationSuccessHandler``\
      - | \ ``defaultTargetUrl``\ にリダイレクト又はフォワードを行う実装クラス。

デフォルトの動作
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityのデフォルトの動作では、認証前にアクセスを拒否したリクエストをHTTPセッションに保存しておいて、
認証が成功した際にアクセスを拒否したリクエストを復元してリダイレクトする。
認証したユーザーにリダイレクト先へのアクセス権があればページが表示され、アクセス権がなければ認可エラーとなる。
この動作を実現するために使用されるのが、\ ``SavedRequestAwareAuthenticationSuccessHandler``\ クラスである。

ログインフォームを明示的に表示してから認証処理を行った後の遷移先はSpring Securityのデフォルトの設定では、
Webアプリケーションのルートパス(\ ``"/"``\ )となっているため、認証成功時はWebアプリケーションのルートパスにリダイレクトされる。

|

.. _SpringSecurityAuthenticationScreenFlowOnFailure:

認証失敗時のレスポンス
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityは、認証失敗時のレスポンスを制御するためのコンポーネントとして、
\ ``AuthenticationFailureHandler``\ というインタフェースと実装クラスを提供している。

.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table:: **AuthenticationFailureHandlerの実装クラス**
    :header-rows: 1
    :widths: 35 65

    * - 実装クラス
      - 説明
    * - | \ ``SimpleUrlAuthenticationFailureHandler``\
      - | 指定したパス(\ ``defaultFailureUrl``\ )にリダイレクト又はフォワードを行う実装クラス。
    * - | \ ``ExceptionMappingAuthenticationFailureHandler``\
      - | 認証例外と遷移先のURLをマッピングすることができる実装クラス。
        | Spring Securityはエラー原因毎に発生する例外クラスが異なるため、この実装クラスを使用するとエラーの種類毎に遷移先を切り替えることが可能である。
    * - | \ ``DelegatingAuthenticationFailureHandler``\
      - | 認証例外と\ ``AuthenticationFailureHandler``\ をマッピングすることができる実装クラス。 
        | \ ``ExceptionMappingAuthenticationFailureHandler``\ と似ているが、認証例外毎に\ ``AuthenticationFailureHandler``\ を指定できるので、より柔軟な振る舞いをサポートすることができる。

デフォルトの動作
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityのデフォルトの動作では、ログインフォームを表示するためのパスに\ ``"error"``\ というクエリパラメータが付与されたURLにリダイレクトする。

例として、ログインフォームを表示するためのパスが\ ``"/login"``\ の場合は\ ``"/login?error"``\ にリダイレクトされる。
  

|

DB認証
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityは、以下のような流れでDB認証を行う。

.. figure:: ./images_Authentication/AuthenticationDatabase.png
    :width: 100%

    **DB認証の仕組み**

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | Spring Securityはクライアントからの認証依頼を受け、\ ``DaoAuthenticationProvider``\ の認証処理を呼び出す。
    * - | (2)
      - | \ ``DaoAuthenticationProvider``\ は、\ ``UserDetailsService``\ のユーザー情報取得処理を呼び出す。
    * - | (3)
      - | ``UserDetailsService``\ の実装クラスは、データストアからユーザー情報を取得する。
    * - | (4)
      - | ``UserDetailsService``\ の実装クラスは、データストアから取得したユーザー情報から\ ``UserDetails``\ を生成する。
    * - | (5)
      - | \ ``DaoAuthenticationProvider``\ は、\ ``UserDetailsService``\ から返却された\ ``UserDetails``\ とクライアントが指定した認証情報との照合を行い、クライアントが指定したユーザーの正当性をチェックする。

.. raw:: latex

   \newpage

.. note:: **Spring Securityが提供するDB認証**

    Spring Securityは、ユーザー情報をリレーショナルデータベースからJDBC経由で取得するための実装クラスを提供している。

    * \ ``org.springframework.security.core.userdetails.User``\ (\ ``UserDetails``\ の実装クラス)
    * \ ``org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl`` \ (\ ``UserDetailsService``\ の実装クラス)

    これらの実装クラスは最低限の認証処理(パスワードの照合、有効ユーザーの判定)しか行わないため、そのまま利用できるケースは少ない。
    そのため、本ガイドラインでは、\ ``UserDetails``\ と\ ``UserDetailsService``\ の実装クラスを作成する方法について説明する。

|

UserDetailsの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``UserDetails``\ は、認証処理で必要となる資格情報(ユーザー名とパスワード)とユーザーの状態を提供するためのインタフェースで、以下のメソッドが定義されている。
\ ``AuthenticationProvider``\ として\ ``DaoAuthenticationProvider``\ を使用する場合は、アプリケーションの要件に合わせて\ ``UserDetails``\ の実装クラスを作成する。

*UserDetailsインタフェース*

.. code-block:: java

    public interface UserDetails extends Serializable {
        String getUsername(); // (1)
        String getPassword(); // (2)
        boolean isEnabled(); // (3)
        boolean isAccountNonLocked(); // (4)
        boolean isAccountNonExpired(); // (5)
        boolean isCredentialsNonExpired(); // (6)
        Collection<? extends GrantedAuthority> getAuthorities(); // (7)
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.25\linewidth}|p{0.65\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 25 65
    :class: longtable

    * - 項番
      - メソッド名
      - 説明
    * - | (1)
      - | \ ``getUsername``\
      - | ユーザー名を返却する。
    * - | (2)
      - | \ ``getPassword``\
      - | 登録されているパスワードを返却する。
        | このメソッドで返却したパスワードとクライアントから指定されたパスワードが一致しない場合は、\ ``DaoAuthenticationProvider``\ は\ ``BadCredentialsException``\ を発生させる。
    * - | (3)
      - | \ ``isEnabled``\
      - | 有効なユーザーかを判定する。有効な場合は\ ``true``\ を返却する。
        | 無効なユーザーの場合は、\ ``DaoAuthenticationProvider``\ は\ ``DisabledException``\ を発生させる。
    * - | (4)
      - | \ ``isAccountNonLocked``\
      - | アカウントのロック状態を判定する。ロックされていない場合は\ ``true``\ を返却する。
        | アカウントがロックされている場合は、\ ``DaoAuthenticationProvider``\ は\ ``LockedException``\ を発生させる。
    * - | (5)
      - | \ ``isAccountNonExpired``\
      - | アカウントの有効期限の状態を判定する。有効期限内の場合は\ ``true``\ を返却する。
        | 有効期限切れの場合は、\ ``DaoAuthenticationProvider``\ は\ ``AccountExpiredException``\ を発生させる。
    * - | (6)
      - | \ ``isCredentialsNonExpired``\
      - | 資格情報の有効期限の状態を判定する。有効期限内の場合は\ ``true``\ を返却する。
        | 有効期限切れの場合は、\ ``DaoAuthenticationProvider``\ は\ ``CredentialsExpiredException``\ を発生させる。
    * - | (7)
      - | \ ``getAuthorities``\
      - | ユーザーに与えられている権限リストを返却する。
        | このメソッドは認可処理で使用される。

.. raw:: latex

   \newpage

.. note:: **認証例外による遷移先の切り替え**

    \ ``DaoAuthenticationProvider``\ が発生させる例外毎に画面遷移を切り替えたい場合は、
    \ ``AuthenticationFailureHandler``\ として\ ``ExceptionMappingAuthenticationFailureHandler``\ を使用すると実現することができる。

    例として、ユーザーのパスワードの有効期限が切れた際にパスワード変更画面に遷移させたい場合は、
    \ ``ExceptionMappingAuthenticationFailureHandler``\ を使って\ ``CredentialsExpiredException``\ をハンドリングすると画面遷移を切り替えることができる。
    
    詳細は、:ref:`SpringSecurityAuthenticationCustomizingScreenFlowOnFailure`\ を参照されたい。

.. note:: **Spring Securityが提供する資格情報**

    Spring Securityは、資格情報(ユーザー名とパスワード)とユーザーの状態を保持するための実装クラス(\ ``org.springframework.security.core.userdetails.User``\ )を提供してるが、
    このクラスは認証処理に必要な情報しか保持することができない。
    一般的なアプリケーションでは、認証処理で使用しないユーザーの情報（ユーザーの氏名など）も必要になるケースが多いため、\ ``User``\ クラスをそのまま利用できるケースは少ない。

|

ここでは、アカウントの情報を保持する\ ``UserDetails``\ の実装クラスを作成する。 
本例は\ ``User``\ を継承することでも実現することができるが、\ ``UserDetails``\  を実装する方法の例として紹介している。

* UserDetailsの実装クラスの作成例


.. code-block:: java

    public class AccountUserDetails implements UserDetails { // (1)

        private final Account account;
        private final Collection<GrantedAuthority> authorities;

        public AccountUserDetails(
            Account account, Collection<GrantedAuthority> authorities) {
            // (2)
            this.account = account;
            this.authorities = authorities;
        }

        // (3)
        public String getPassword() {
            return account.getPassword();
        }
        public String getUsername() {
            return account.getUsername();
        }
        public boolean isEnabled() {
            return account.isEnabled();
        }
        public Collection<GrantedAuthority> getAuthorities() {
            return authorities;
        }

        // (4)
        public boolean isAccountNonExpired() {
            return true;
        }
        public boolean isAccountNonLocked() {
            return true;
        }
        public boolean isCredentialsNonExpired() {
            return true;
        }

        // (5)
        public Account getAccount() {
            return account;
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``UserDetails``\ インタフェースを実装したクラスを作成する。
    * - | (2)
      - | ユーザー情報と権限情報をプロパティに保持する。
    * - | (3)
      - | \ ``UserDetails``\ インタフェースに定義されているメソッドを実装する。
    * - | (4)
      - | 本節の例では、「アカウントのロック」「アカウントの有効期限切れ」「資格情報の有効期限切れ」に対するチェックは未実装であるが、要件に合わせて実装されたい。
    * - | (5)
      - | 認証処理成功後の処理でアカウント情報にアクセスできるようにするために、getterメソッドを用意する。

|

Spring Securityは、\ ``UserDetails``\ の実装クラスとして\ ``User``\ クラスを提供している。
\ ``User``\ クラスを継承すると資格情報とユーザーの状態を簡単に保持することができる。

* Userクラスを継承したUserDetails実装クラスの作成例

.. code-block:: java

    public class AccountUserDetails extends User {

        private final Account account;

        public AccountUserDetails(Account account, boolean accountNonExpired,
                boolean credentialsNonExpired, boolean accountNonLocked,
                Collection<GrantedAuthority> authorities) {
            super(account.getUsername(), account.getPassword(),
                    account.isEnabled(), true, true, true, authorities);
            this.account = account;
        }

        public Account getAccount() {
            return account;
        }
    }

|

.. _SpringSecurityAuthenticationUserDetailsService:

UserDetailsServiceの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``UserDetailsService``\ は、認証処理で必要となる資格情報とユーザーの状態をデータストア
から取得するためのインタフェースで、以下のメソッドが定義されている。
\ ``AuthenticationProvider``\ として\ ``DaoAuthenticationProvider``\ を使用する場合は、
アプリケーションの要件に合わせて\ ``UserDetailsService``\ の実装クラスを作成する。

* UserDetailsServiceインタフェース

.. code-block:: java

    public interface UserDetailsService {
        UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
    }

|

ここでは、データベースからアカウント情報を検索して、\ ``UserDetails``\ のインスタンス
を生成するためのサービスクラスを作成する。
本サンプルでは、\ ``SharedService``\ を使用して、アカウント情報を取得している。
\ ``SharedService``\ については、:ref:`service-label`\ を参照されたい。

* AccountSharedServiceインタフェースの作成例

.. code-block:: java

    public interface AccountSharedService {
        Account findOne(String username);
    }

* AccountSharedServiceの実装クラスの作成例

.. code-block:: java

    // (1)
    @Service
    @Transactional
    public class AccountSharedServiceImpl implements AccountSharedService {
        @Inject
        AccountRepository accountRepository;

        // (2)
        @Override
        public Account findOne(String username) {
            Account account = accountRepository.findOneByUsername(username);
            if (account == null) {
                throw new ResourceNotFoundException("The given account is not found! username="
                        + username);
            }
            return account;
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``AccountSharedService``\ インタフェースを実装したクラスを作成し、\ ``@Service``\ を付与する。
        | 上記例では、コンポーネントスキャン機能を使って\ ``AccountSharedServiceImpl``\ をDIコンテナに登録している。
    * - |  (2)
      - | データベースからアカウント情報を検索する。
        | アカウント情報が見つからない場合は、共通ライブラリの例外である\ ``ResourceNotFoundException``\ を発生させる。
        | Repositoryの作成例については、「:doc:`../Tutorial/TutorialSecurity`」を参照されたい。

* UserDetailsServiceの実装クラスの作成例

.. code-block:: java

    // (1)
    @Service
    @Transactional
    public class AccountUserDetailsService implements UserDetailsService {
        @Inject
        AccountSharedService accountSharedService;

        public UserDetails loadUserByUsername(String username)
                throws UsernameNotFoundException {

            try {
                Account account = accountSharedService.findOne(username);
                // (2)
                return new AccountUserDetails(account, getAuthorities(account));
            } catch (ResourceNotFoundException e) {
                // (3)
                throw new UsernameNotFoundException("user not found", e);
            }
        }

        // (4)
        private Collection<GrantedAuthority> getAuthorities(Account account) {
            if (account.isAdmin()) {
                return AuthorityUtils.createAuthorityList("ROLE_USER", "ROLE_ADMIN");
            } else {
                return AuthorityUtils.createAuthorityList("ROLE_USER");
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
      - | \ ``UserDetailsService``\ インタフェースを実装したクラスを作成し、\ ``@Service``\ を付与する。
        | 上記例では、コンポーネントスキャン機能を使って\ ``UserDetailsService``\ をDIコンテナに登録している。
    * - | (2)
      - | \ ``AccountSharedService``\ を使用してアカウント情報を取得する。
        | アカウント情報が見つかった場合は、\ ``UserDetails``\ を生成する。
        | 上記例では、ユーザー名、パスワード、ユーザーの有効状態をアカウント情報から取得している。
    * - | (3)
      - | アカウント情報が見つからない場合は、\ ``UsernameNotFoundException``\ を発生させる。
    * - | (4)
      - | ユーザーが保持する権限(ロール)情報を生成する。ここで生成した権限(ロール)情報は、認可処理で使用される。

.. note:: **認可で使用する権限情報**

    Spring Securityの認可処理は、\ ``"ROLE_"``\ で始まる権限情報をロールとして扱う。
    そのため、ロールを使用してリソースへのアクセス制御を行う場合は、 ロールとして扱う権限情報に\ ``"ROLE_"``\ プレフィックスを付与する必要がある。

.. note:: **認証例外情報の隠蔽**

    Spring Securityのデフォルトの動作では、\ ``UsernameNotFoundException``\ は\ ``BadCredentialsException``\ という例外に変換してからエラー処理を行う。
    \ ``BadCredentialsException``\ は、クライアントから指定された資格情報のいずれかの項目に誤りがあることを通知するための例外であり、具体的なエラー理由がクライアントに通知されることはない。

|

.. _AuthenticationProviderConfiguration:

DB認証の適用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

作成した\ ``UserDetailsService``\ を使用して認証処理を行うためには、
\ ``DaoAuthenticationProvider``\ を有効化して、作成した\ ``UserDetailsService``\ を適用する必要がある。

* spring-security.xmlの定義例

.. code-block:: xml

    <sec:authentication-manager> <!-- (1) -->
        <sec:authentication-provider user-service-ref="accountUserDetailsService"> <!-- (2) -->
            <sec:password-encoder ref="passwordEncoder" /> <!-- (3) -->
        </sec:authentication-provider>
    </sec:authentication-manager>

    <bean id="passwordEncoder"
        class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder" /> <!-- (4) -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``AuthenticationManager``\ をbean定義する。
    * - | (2)
      - | \ ``<sec:authentication-manager>``\ 要素内に ``<sec:authentication-provider>``\ 要素を定義する。
        | ``user-service-ref``\ 属性に「:ref:`SpringSecurityAuthenticationUserDetailsService`」で作成した ``AccountUserDetailsService``\ のbeanを指定する。
        | 本定義により、デフォルト設定の\ ``DaoAuthenticationProvider``\ が有効になる。
    * - | (3)
      - | パスワード照合時に使用する\ ``PasswordEncoder``\ のbeanを指定する。
    * - | (4)
      - | パスワード照合時に使用する\ ``PasswordEncoder``\ をBean定義する。
        | 上記例では、パスワードをBCryptアルゴリズムでハッシュ化する\ ``BCryptPasswordEncoder``\ を定義している。
        | パスワードのハッシュ化については、「:ref:`SpringSecurityAuthenticationPasswordHashing`」を参照されたい。

|

.. _SpringSecurityAuthenticationPasswordHashing:

パスワードのハッシュ化
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

パスワードをデータベースなどに保存する場合は、パスワードそのものではなくパスワードの
ハッシュ値を保存するのが一般的である。

Spring Securityは、パスワードをハッシュ化するためのインタフェースと実装クラスを
提供しており、認証機能と連携して動作する。

Spring Securityが提供するインタフェースには、以下の2種類がある。

* \ ``org.springframework.security.crypto.password.PasswordEncoder``\
* \ ``org.springframework.security.authentication.encoding.PasswordEncoder``\

どちらも\ ``PasswordEncoder``\ という名前のインタフェースであるが、
\ ``org.springframework.security.authentication.encoding``\ パッケージの\ ``PasswordEncoder``\
は非推奨になっている。
パスワードのハッシュ化要件に制約がない場合は、\ ``org.springframework.security.crypto.password``\
パッケージの\ ``PasswordEncoder``\ インタフェースの実装クラスを使用することを推奨する。

.. note::

    非推奨の\ ``PasswordEncoder``\ の利用方法については、
    「:ref:`AuthenticationHowToExtendUsingDeprecatedPasswordEncoder`」を参照されたい。

|

*org.springframework.security.crypto.password.PasswordEncoderのメソッド定義*

.. code-block:: java

    public interface PasswordEncoder {
        String encode(CharSequence rawPassword);
        boolean matches(CharSequence rawPassword, String encodedPassword);
    }

.. tabularcolumns:: |p{0.15\linewidth}|p{0.85\linewidth}|
.. list-table:: **PasswordEncoderに定義されているメソッド**
    :header-rows: 1
    :widths: 15 85

    * - メソッド名
      - 説明
    * - | \ ``encode``\
      - | パスワードをハッシュ化するためのメソッド。
        | アカウントの登録処理やパスワード変更処理などでデータストアに保存するパスワードをハッシュ化する際に使用できる。
    * - | \ ``matches``\
      - | 平文のパスワードとハッシュ化されたパスワードを照合するためのメソッド。
        | このメソッドはSpring Securityの認証処理でも利用されるが、パスワード変更処理などで現在のパスワードや過去に使用していたパスワードと照合する際にも使用できる。

|

Spring Securityは、\ ``PasswordEncoder``\ インタフェースの実装クラスとして、以下のクラスを提供している。

.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table:: **PasswordEncoderの実装クラス**
    :header-rows: 1
    :widths: 35 65

    * - 実装クラス
      - 説明
    * - | \ ``BCryptPasswordEncoder``\
      - | BCryptアルゴリズムを使用してパスワードのハッシュ化及び照合を行う実装クラス。
        | **パスワードのハッシュ化要件に制約がない場合は、このクラスを使用することを推奨する。**
        | 詳細は、\ `BCryptPasswordEncoderのJavaDoc <http://docs.spring.io/spring-security/site/docs/4.1.4.RELEASE/apidocs/org/springframework/security/crypto/bcrypt/BCryptPasswordEncoder.html>`_\ を参照されたい。
    * - | \ ``StandardPasswordEncoder``\
      - | SHA-256アルゴリズムを使用してパスワードのハッシュ化及び照合を行う実装クラス。
        | 詳細は、\ `StandardPasswordEncoderのJavaDoc <http://docs.spring.io/spring-security/site/docs/4.1.4.RELEASE/apidocs/org/springframework/security/crypto/password/StandardPasswordEncoder.html>`_\ を参照されたい。
    * - | \ ``NoOpPasswordEncoder``\
      - | ハッシュ化しない実装クラス。
        | テスト用のクラスなであり、実際のアプリケーションで使用することはない。

本節では、Spring Securityが利用を推奨している\ ``BCryptPasswordEncoder``\ の使い方について説明する。

|

BCryptPasswordEncoder
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``BCryptPasswordEncoder``\ は、BCryptアルゴリズムを使用してパスワードのハッシュ化及びパスワードの照合を行う実装クラスである。
:ref:`ソルト<SpringSecurityAuthenticationPasswordHashSalt>` には16バイトの乱数(\ ``java.security.SecureRandom``\ )が使用され、
デフォルトでは1,024(2の10乗)回 :ref:`ストレッチング<SpringSecurityAuthenticationPasswordHashStength>` を行う。

* applicationContext.xmlの定義例

.. code-block:: xml

  <bean id="passwordEncoder"
      class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder" > <!-- (1) -->
      <constructor-arg name="strength" value="11" /> <!-- (2) -->
  </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | passwordEncoderのクラスに\ ``BCryptPasswordEncoder``\ を指定する。
    * - | (2)
      - | コンストラクタの引数に、ハッシュ化のストレッチング回数のラウンド数を指定する。
        | 本引数は省略可能であり、指定できる値は\ ``4``\から\ ``31``\ である。
        | なお、未指定時のデフォルト値は\ ``10``\ である。
        | 本ガイドラインでは説明を省略するが、コンストラクタ引数として\ ``java.security.SecureRandom.SecureRandom``\ を指定することも可能である。

.. warning:: **SecureRandomの使用について**
  
    Linux環境で\ ``SecureRandom``\ を使用する場合、処理の遅延やタイムアウトが発生する場合がある。
    これは使用する乱数生成器に左右される事象であり、以下のドキュメントに説明がある。
  
    * https://docs.oracle.com/javase/8/docs/api/java/security/SecureRandom.html 
  
    本事象が発生する場合は、以下のいずれかの設定を追加することで回避することができる。
  
    * Javaコマンド実行時に ``-Djava.security.egd=file:/dev/urandom`` を指定する。 
  
    * ``${JAVA_HOME}/jre/lib/security/java.security`` 内の ``securerandom.source=/dev/random`` を ``securerandom.source=/dev/urandom`` に変更する。
  
    Java SE 7のb19以前のバージョン(正式リリース前)で本事象が発生する場合は、 ``/dev/urandom`` ではなく ``/dev/./urandom`` を指定する必要がある。ただし、 \ ``SecureRandom``\ で使用するアルゴリズムが \ ``NativePRNG``\ の場合は回避することができない。

|

\ ``BCryptPasswordEncoder``\ を使用して処理を行うクラスでは、\ ``PasswordEncoder``\ をDIコンテナからインジェクションして使用する。

.. code-block:: java

    @Service
    @Transactional
    public class AccountServiceImpl implements AccountService {

        @Inject
        AccountRepository accountRepository;

        @Inject
        PasswordEncoder passwordEncoder; // (1)

        public Account register(Account account, String rawPassword) {
            // omitted
            String encodedPassword = passwordEncoder.encode(rawPassword); // (2)
            account.setPassword(encodedPassword);
            // omitted
            return accountRepository.save(account);
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``PasswordEncoder``\ をインジェクションする。
    * - | (2)
      - | インジェクションした\ ``PasswordEncoder``\ のメソッドを呼び出す。
        | ここでは、データストアに保存するパスワードをハッシュ化していいる。

.. _SpringSecurityAuthenticationPasswordHashSalt:

.. note:: **ソルト**

    ハッシュ化対象のデータに追加する文字列のことである。
    ソルトをパスワードに付与することで、実際のパスワードより桁数が長くなるため、レインボークラックなどのパスワード解析を困難にすることができる。
    なお、**ソルトはユーザーごとに異なる値（ランダム値等）を設定することを推奨する。**
    これは、同じソルトを使用していると、ハッシュ値からハッシュ化前の文字列(パスワード)がわかってしまう可能性があるためである。

.. _SpringSecurityAuthenticationPasswordHashStength:

.. note:: **ストレッチング**

    ハッシュ関数の計算を繰り返し行うことで、保管するパスワードに関する情報を繰り返し暗号化することである。
    パスワードの総当たり攻撃への対策として、パスワード解析に必要な時間を延ばすために行う。
    しかし、ストレッチングはシステムの性能に影響を与えるので、システムの性能を考慮してストレッチング回数を決める必要がある。

    Spring Securityのデフォルトでは1,024(2の10乗)回ストレッチングを行うが、この回数はコンストラクタ引数(\ ``strength``\ )で変更することができる。
    \ ``strength``\ には4(16回)から31(2,147,483,648回)を指定することが可能である。
    ストレッチング回数が多いほどパスワードの強度は増すが、計算量が多くなるため性能にあたえる影響も大きくなる。

|

.. _SpringSecurityAuthenticationEvent:

認証イベントのハンドリング
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityは、Spring Frameworkが提供しているイベント通知の仕組みを利用して、
認証処理の処理結果を他のコンポーネントと連携する仕組みを提供している。

この仕組みを利用すると、以下のようなセキュリティ要件をSpring Securityの認証機能に組み込むことが可能である。

* 認証成功、失敗などの認証履歴をデータベースやログに保存する。
* パスワードを連続して間違った場合にアカウントをロックする。

認証イベントの通知は、以下のような仕組みで行われる。

.. figure:: ./images_Authentication/AuthenticationEventNotification.png
    :width: 100%

    **イベント通知の仕組み**

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | Spring Securityの認証機能は、認証結果(認証情報や認証例外)を
        | \ ``AuthenticationEventPublisher``\ に渡して認証イベントの通知依頼を行う。
    * - | (2)
      - | \ ``AuthenticationEventPublisher``\ インタフェースのデフォルトの実装クラスは
        | \ 認証結果に対応する認証イベントクラスのインスタンスを生成し、\ ``ApplicationEventPublisher``\ に渡してイベントの通知依頼を行う。
    * - | (3)
      - | \ ``ApplicationEventPublisher``\ インタフェースの実装クラスは、\ ``ApplicationListener``\ インタフェースの実装クラスにイベントを通知する。
    * - | (4)
      - | ``ApplicationListener``\ の実装クラスの一つである\ ``ApplicationListenerMethodAdaptor``\ は、
        | \ ``@org.springframework.context.event.EventListener``\ が付与されているメソッドを呼び出してイベントを通知する。

.. note:: **メモ**

    Spring 4.1までは\ ``ApplicationListener``\ インタフェースの実装クラスを作成してイベントを受け取る必要があったが、
    Spring 4.2からはPOJOに\ ``@EventListener``\ を付与したメソッドを実装するだけでイベントを受け取ることが可能である。
    なお、Spring 4.2以降でも、従来通り\ ``ApplicationListener``\ インタフェースの実装クラスを作成してイベントを受け取ることもで可能である。

Spring Security使用しているイベントは、認証が成功したことを通知するイベントと認証が失敗したことを通知するイベントの2種類に分類される。
以下にSpring Securityが用意しているイベントクラスを説明する。

|

認証成功イベント
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

認証が成功した時にSpring Securityが通知する主なイベントは以下の3つである。
この3つのイベントは途中でエラーが発生しなければ、以下の順番ですべて通知される。

.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table:: **認証が成功したことを通知するイベントクラス**
    :header-rows: 1
    :widths: 35 65

    * - イベントクラス
      - 説明
    * - \ ``AuthenticationSuccessEvent``\
      - \ ``AuthenticationProvider``\ による認証処理が成功したことを通知するためのイベントクラス。
        このイベントをハンドリングすると、クライアントが正しい認証情報を指定したことを検知することが可能である。
        なお、このイベントをハンドリングした後の後続処理でエラーが発生する可能性がある点に注意されたい。
    * - \ ``SessionFixationProtectionEvent``\
      - セッション固定攻撃対策の処理(セッションIDの変更処理)が成功したことを通知するためのイベントクラス。
        このイベントをハンドリングすると、変更後のセッションIDを検知することがで可能になる。
    * - \ ``InteractiveAuthenticationSuccessEvent``\
      - 認証処理がすべて成功したことを通知するためのイベントクラス。
        このイベントをハンドリングすると、画面遷移を除くすべての認証処理が成功したことを検知することが可能になる。

|

認証失敗イベント
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

認証が失敗した時にSpring Securityが通知する主なイベントは以下の通り。
認証に失敗した場合は、いずれか一つのイベントが通知される。

.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table:: **認証が失敗したことを通知するイベントクラス**
    :header-rows: 1
    :widths: 35 65

    * - イベントクラス
      - 説明
    * - | \ ``AuthenticationFailureBadCredentialsEvent``\
      - | \ ``BadCredentialsException``\ が発生したことを通知するためのイベントクラス。
    * - | \ ``AuthenticationFailureDisabledEvent``\
      - | \ ``DisabledException``\ が発生したことを通知するためのイベントクラス。
    * - | \ ``AuthenticationFailureLockedEvent``\
      - | \ ``LockedException``\ が発生したことを通知するためのイベントクラス。
    * - | \ ``AuthenticationFailureExpiredEvent``\
      - | \ ``AccountExpiredException``\ が発生したことを通知するためのイベントクラス。
    * - | \ ``AuthenticationFailureCredentialsExpiredEvent``\
      - | \ ``CredentialsExpiredException``\ が発生したことを通知するためのイベントクラス。
    * - | \ ``AuthenticationFailureServiceExceptionEvent``\
      - | \ ``AuthenticationServiceException``\ が発生したことを通知するためのイベントクラス。

|

イベントリスナの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

認証イベントの通知を受け取って処理を行いたい場合は、\ ``@EventListener``\ を付与したメソッドを実装したクラスを作成し、DIコンテナに登録する。

* イベントリスナクラスの実装例

.. code-block:: java

    @Component
    public class AuthenticationEventListeners {

        private static final Logger log =
                LoggerFactory.getLogger(AuthenticationEventListeners.class);

    @EventListener // (1) 
    public void handleBadCredentials( 
        AuthenticationFailureBadCredentialsEvent event) { // (2) 
        log.info("Bad credentials is detected. username : {}", event.getAuthentication().getName()); 
        // omitted 
    } 


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ``@EventListener``\ をメソッドに付与したメソッドを作成する。
    * - | (2)
      - | メソッドの引数にハンドリングしたい認証イベントクラスを指定する。

上記例では、クライアントが指定した認証情報に誤りがあった場合に通知される\ ``AuthenticationFailureBadCredentialsEvent``\ をハンドリングするクラスを作成する例としているが、
他のイベントも同じ要領でハンドリングすることが可能である。

|

.. _SpringSecurityAuthenticationLogout:

ログアウト
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityは、以下のような流れでログアウト処理を行いう。

.. figure:: ./images_Authentication/AuthenticationLogout.png
    :width: 100%

    **ログアウト処理の仕組み**

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | クライアントは、ログアウト処理を行うためのパスにリクエストを送信する。
    * - | (2)
      - | \ ``LogoutFilter``\ は、\ ``LogoutHandler``\ のメソッドを呼び出し、実際のログアウト処理を行う。
    * - | (3)
      - | \ ``LogoutFilter``\ は、\ ``LogoutSuccessHandler``\ のメソッドを呼び出し、画面遷移を行う。

|

\ ``LogoutHandler``\ の実装クラスは複数存在し、それぞれ以下の役割をもっている。

.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table:: **主なLogoutHandlerの実装クラス**
    :header-rows: 1
    :widths: 35 65

    * - 実装クラス
      - 説明
    * - | \ ``SecurityContextLogoutHandler``\
      - | ログインユーザーの認証情報のクリアとセッションの破棄を行うクラス。
    * - | \ ``CookieClearingLogoutHandler``\
      - | 指定したクッキーを削除するためのレスポンスを行うクラス。
    * - | \ ``CsrfLogoutHandler``\
      - | CSRF対策用トークンの破棄を行うクラス。

これらの\ ``LogoutHandler``\ は、Spring Securityが提供しているbean定義をサポートするクラスが自動で\ ``LogoutFilter``\ に設定する仕組みになっているため、
基本的にはアプリケーションの開発者が直接意識する必要はない。
また、:ref:`Remember Me認証機能<SpringSecurityAuthenticationRememberMe>` を有効にすると、Remember Me認証用のTokenを破棄するための\ ``LogoutHandler``\ の実装クラスも設定される。

|

ログアウト処理の適用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ログアウト処理を適用するためには、以下のようなbean定義を行う。

* spring-security.xmlの定義例

.. code-block:: xml

  <sec:http>
      <!-- omitted -->
      <sec:logout /> <!-- (1) -->
      <!-- omitted -->
  </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``<sec:logout>``\ タグを定義することで、ログアウト処理が有効となる。

.. note:: **Spring Security 4.0における変更**

    Spring Security 4.0から、以下の設定のデフォルト値が変更されている

    * logout-url 

.. tip:: **Cookieの削除**

   本ガイドラインでは説明を割愛するが、 \ ``<sec:logout>``\ タグには、ログアウト時に指定したCookieを削除するための\ ``delete-cookies``\ 属性が存在する。
   ただし、この属性を使用しても正常にCookieが削除できないケースが報告されている。

   詳細はSpring Securityの以下のJIRAを参照されたい。

   * https://jira.spring.io/browse/SEC-2091

デフォルトの動作
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityのデフォルトの動作では、\ ``"/logout"``\ というパスにリクエストを送るとログアウト処理が行われる。
ログアウト処理では、「ログインユーザーの認証情報のクリア」「セッションの破棄」が行われる。

また、

* CSRF対策を行っている場合は、「CSRF対策用トークンの破棄」
* Remember Me認証機能を使用している場合は、「Remember Me認証用のTokenの破棄」

も行われる

.. _SpringSecurityAuthenticationLogoutForm:

* ログアウト処理を呼び出すためのJSPの実装例

.. code-block:: jsp

    <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
    <%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags" %>
    <%-- omitted --%>
    <form:form action="${pageContext.request.contextPath}/logout" method="post"> <%-- (1) --%>
        <button>ログアウト</button>
    </form:form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ログアウト用のフォームを作成する。
        | また、\ ``<form:form>``\ を使用することで、CSRF対策用のトークン値がリクエストパラメータで送信される。
        | CSRF対策については、「:ref:`SpringSecurityCsrf`」で説明する。

.. note:: **CSRFトークンの送信**

    CSRF対策を有効にしている場合は、CSRF対策用のトークンをPOSTメソッドを使って送信する必要がる。

|

ログアウト成功時のレスポンス
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityは、ログアウト成功時のレスポンスを制御するためのコンポーネントとして、
\ ``LogoutSuccessHandler``\ というインタフェースと実装クラスを提供している。

.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table:: **AuthenticationFailureHandlerの実装クラス**
    :header-rows: 1
    :widths: 35 65

    * - 実装クラス
      - 説明
    * - | \ ``SimpleUrlLogoutSuccessHandler``\
      - | 指定したパス(\ ``defaultTargetUrl``\ )にリダイレクトを行う実装クラス。


デフォルトの動作
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityのデフォルトの動作では、ログインフォームを表示するためのパスに\ ``"logout"``\
というクエリパラメータが付与されたURLにリダイレクトする。

例として、ログインフォームを表示するためのパスが\ ``"/login"``\ の場合は\ ``"/login?logout"``\
にリダイレクトされる。

|

.. _SpringSecurityAuthenticationAccess:

認証情報へのアクセス
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

認証されたユーザーの認証情報は、Spring Securityのデフォルト実装ではセッションに格納される。
セッションに格納された認証情報は、リクエスト毎に\ ``SecurityContextPersistenceFilter``\ クラスによって\ ``SecurityContextHolder``\ というクラスに格納され、同一スレッド内であればどこからでもアクセスすることができるようになる。

ここでは、認証情報から\ ``UserDetails``\ を取得し、取得した\ ``UserDetails``\ が保持している情報にアクセスする方法を説明する。

Javaからのアクセス
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

一般的な業務アプリケーションでは、「いつ」「誰が」「どのデータに」「どのようなアクセスをしたか」を記録する監査ログを取得することがある。
このような要件を実現する際の「誰が」は、認証情報から取得することができる。

* Javaから認証情報へアクセスする実装例

.. code-block:: java

    Authentication authentication =
            SecurityContextHolder.getContext().getAuthentication(); // (1)
    String userUuid = null;
    if (authentication.getPrincipal() instanceof AccountUserDetails) {
        AccountUserDetails userDetails =
                AccountUserDetails.class.cast(authentication.getPrincipal()); // (2)
        userUuid = userDetails.getAccount().getUserUuid(); // (3)
    }
    if (log.isInfoEnabled()) {
        log.info("type:Audit\tuserUuid:{}\tresource:{}\tmethod:{}",
                userUuid, httpRequest.getRequestURI(), httpRequest.getMethod());
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``SecurityContextHolder``\ から認証情報(\ ``Authentication``\ オブジェクト) を取得する。
    * - | (2)
      - | \ ``Authentication#getPrincipal()``\ メソッドを呼び出して、\ ``UserDetails``\ オブジェクトを取得する。
        | 認証済みでない場合(匿名ユーザーの場合)は、匿名ユーザーであることを示す文字列が返却されるため注意されたい。
    * - | (3)
      - | \ ``UserDetails``\ から処理に必要な情報を取得する。
        | ここでは、ユーザーを一意に識別するための値(UUID)を取得している。

.. warning:: **認証情報へのアクセスと結合度**

    Spring Securityのデフォルト実装では、認証情報をスレッドローカルの変数に格納しているため、リクエストを受けたスレッドと同じスレッドであればどこからでもアクセス可能である。
    この仕組みは便利ではあるが、認証情報を必要とするクラスが\ ``SecurityContextHolder``\ クラスに直接依存してしまうため、乱用するとコンポーネントの疎結合性が低下するので注意が必要である。

    Spring Securityでは、Spring MVCの機能と連携してコンポーネント間の疎結合性を保つための仕組みを別途提供している。
    Spring MVCとの連携方法については、「:ref:`SpringSecurityAuthenticationIntegrationWithSpringMVC`」で説明する。
    **本ガイドラインではSpring MVCとの連携を使用して認証情報を取得することを推奨する。**

.. note::

    認証処理用のフィルタ(FORM_LOGIN_FILTER)をカスタマイズする場合は、
    \ ``<sec:concurrency-control>``\ 要素の指定に加えて、以下の２つの\ ``SessionAuthenticationStrategy``\ クラスを有効化する必要がある。

    * | ``org.springframework.security.web.authentication.session.ConcurrentSessionControlAuthenticationStrategy``
      | 認証成功後にログインユーザ毎のセッション数をチェックするクラス。

    * | ``org.springframework.security.web.authentication.session.RegisterSessionAuthenticationStrategy``
      | 認証に成功したセッションをセッション管理領域に登録するクラス。

    version 1.0.x.RELEASEで依存しているSpring Security 3.1では、\ ``org.springframework.security.web.authentication.session.ConcurrentSessionControlStrategy``\ というクラスが提供されていたが、
    Spring Security 3.2より非推奨のAPIになり、Spring Security 4.0より廃止になっている。
    Spring Security 3.1からSpring Security 3.2以降にバージョンアップする場合は、以下のクラスを組み合わせて使用するように変更する必要がある。

    * ``ConcurrentSessionControlAuthenticationStrategy`` (Spring Security 3.2で追加)
    * ``RegisterSessionAuthenticationStrategy`` (Spring Security 3.2で追加)
    * ``org.springframework.security.web.authentication.session.SessionFixationProtectionStrategy``

    具体的な定義方法については、
    `Spring Security Reference -Web Application Security (Concurrency Control)- <http://docs.spring.io/spring-security/site/docs/4.1.4.RELEASE/reference/htmlsingle/#concurrent-sessions>`_ のサンプルコードを参考にされたい。

|

JSPからのアクセス
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

一般的なWebアプリケーションでは、ログインユーザーのユーザー情報などを画面に表示することがある。
このような要件を実現する際のログインユーザーのユーザー情報は、認証情報から取得することができる。

* JSPから認証情報へアクセスする実装例

.. code-block:: jsp

    <%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags" %>
    <%-- omitted --%>
    ようこそ、
    <sec:authentication property="principal.account.lastName"/> <%-- (1) --%>
    さん。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | Spring Securityから提供されている\ ``<sec:authentication>``\ タグを使用して、認証情報(\ ``Authentication``\ オブジェクト) を取得する。
        | \ ``property``\ 属性にアクセスしたいプロパティへのパスを指定する。
        | ネストしているオブジェクトへアクセスしたい場合は、プロパティ名を\ ``"."``\ でつなげればよい。

.. tip:: **認証情報の表示方法**

    ここでは、認証情報が保持するユーザー情報を表示する際の実装例を説明したが、\ ``var``\ 属性と\ ``scope``\ 属性を組み合わせて任意のスコープ変数に値を格納することも可能である。
    ログインユーザーの状態によって表示内容を切り替えたい場合は、ユーザー情報を変数に格納しておき、JSTLのタグライブラリなどを使って表示を切り替えることが可能である。

    上記の例は、以下のように記述することでも実現することができる。
    本例では、\ ``scope``\ 属性を省略しているため、\ ``page``\スコープが適用される。

        .. code-block:: jsp

            <%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags" %>
            <%-- omitted --%>
            <sec:authentication var="principal" property="principal"/>
            <%-- omitted --%>
            ようこそ、
            ${f:h(principal.account.lastName)}
            さん。

|

.. _SpringSecurityAuthenticationIntegrationWithSpringMVC:

認証処理とSpring MVCの連携
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityは、Spring MVCと連携するためのコンポーネントをいくつか提供している。
ここでは、認証処理と連携するためのコンポーネントの使い方を説明する。

認証情報へのアクセス
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityは、認証情報(\ ``UserDetails``\ )をSpring MVCのコントローラーのメソッドに引き渡すためのコンポーネントとして、\ ``AuthenticationPrincipalArgumentResolver``\ クラスを提供している。
\ ``AuthenticationPrincipalArgumentResolver``\ を使用すると、コントローラーのメソッド引数として\ ``UserDetails``\ インタフェースまたはその実装クラスのインスタンスを受け取ることができるため、コンポーネントの疎結合性を高めることができる。

認証情報(\ ``UserDetails``\ )をコントローラーの引数として受け取るためには、まず\ ``AuthenticationPrincipalArgumentResolver``\ をSpring MVCに適用する必要がある。
\ ``AuthenticationPrincipalArgumentResolver``\ を適用するためのbean定義は以下の通りである。
\ なお、`ブランクプロジェクト <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank>`_\ には\ ``AuthenticationPrincipalArgumentResolver``\ が設定済みである。

* spring-mvc.xmlの定義例

.. code-block:: xml

    <mvc:annotation-driven>
        <mvc:argument-resolvers>
            <!-- omitted -->
            <!-- (1) -->
            <bean class="org.springframework.security.web.method.annotation.AuthenticationPrincipalArgumentResolver" />
            <!-- omitted -->
        </mvc:argument-resolvers>
  </mvc:annotation-driven>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``HandlerMethodArgumentResolver``\ の実装クラスとして、\ ``AuthenticationPrincipalArgumentResolver``\ をSpring MVCに適用する。

|

認証情報(\ ``UserDetails``\ )をコントローラーのメソッドで受け取る際は、以下のようなメソッドを作成する。

* 認証情報(UserDetails)を受け取るメソッドの作成例

.. code-block:: java

    @RequestMapping("account")
    @Controller
    public class AccountController {

        public String view(
                @AuthenticationPrincipal AccountUserDetails userDetails, // (1)
                Model model) {
            model.addAttribute(userDetails.getAccount());
            return "profile";
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 認証情報(\ ``UserDetails``\ ) を受け取るための引数を宣言し、\ ``@org.springframework.security.core.annotation.AuthenticationPrincipal``\を引数アノテーションとして指定する。
        | \ ``AuthenticationPrincipalArgumentResolver``\ は、\ ``@AuthenticationPrincipal``\ が付与されている引数に認証情報(\ ``UserDetails``\ )が設定される。

|

.. _SpringSecurityAuthenticationHowToExtend:

How to extend
--------------------------------------------------------------------------------

本節では、Spring Securityが用意しているカスタマイズポイントや拡張方法について説明する。

Spring Securityは、多くのカスタマイズポイントを提供しているため、すべてのカスタマイズポイントを紹介することはできないため、ここでは代表的なカスタマイズポイントに絞って説明を行う。

|

.. _SpringSecurityAuthenticationCustomizingForm:

フォーム認証のカスタマイズ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

フォーム認証処理のカスタマイズポイントを説明する。

認証パスの変更
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityのデフォルトでは、認証処理を実行するためのパスは「\ ``"/login"``\」であるが、
以下のようなbean定義を行うことで変更することが可能である。

* spring-security.xmlの定義例

.. code-block:: xml

  <sec:http>
    <sec:form-login login-processing-url="/authentication" /> <!-- (1) --> 
    <!-- omitted -->
  </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``login-processing-url``\ 属性に認証処理を行うためのパスを指定する。

.. note::

    認証処理のパスを変更した場合は、:ref:`ログインフォーム<SpringSecurityAuthenticationLoginForm>` のリクエスト先も変更する必要がある。

|

資格情報を送るリクエストパラメータ名の変更
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityのデフォルトでは、資格情報(ユーザー名とパスワード)を送るためのリクエストパラメータは「\ ``username``\」と「\ ``password``\ 」であるが、
以下のようなbean定義を行うことで変更することが可能である。

* spring-security.xmlの定義例

.. code-block:: xml

  <sec:http>
      <sec:form-login
          username-parameter="uid"
          password-parameter="pwd" /> <!-- (1) (2) -->
      <!-- omitted -->
  </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``username-parameter``\ 属性にユーザー名のリクエストパラメータ名を指定する。
    * - | (2)
      - | \ ``password-parameter``\ 属性にパスワードのリクエストパラメータ名を指定する。

.. note::

    リクエストパラメータ名を変更した場合は、:ref:`ログインフォーム<SpringSecurityAuthenticationLoginForm>` 内の項目名も変更する必要がある。

|

認証成功時のレスポンスのカスタマイズ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

認証成功時のレスポンスのカスタマイズポイントを説明する。

デフォルト遷移先の変更
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ログインフォームを自分で表示して認証処理を行った後の遷移先(デフォルトURL)は、
Webアプリケーションのルートパス(\ ``"/"``\ )だが、以下のようなbean定義を行うことで変更することが可能である。

* spring-security.xmlの定義例

.. code-block:: xml

  <sec:http>
      <sec:form-login default-target-url="/menu" /> <!-- (1) -->
  </sec:http>

.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``default-target-url``\ 属性に認証成功時に遷移するデフォルトのパスを指定する。

|

遷移先の固定化
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityのデフォルトの動作では、未認証時に認証が必要なページへのリクエストを受信した場合は、受信したリクエストを一旦HTTPセッションに保存し、認証ページに遷移する。
認証成功時にリクエストを復元してリダイレクトするが、以下のようなbean定義を行うことで常に同じ画面に遷移させることが可能である。

* spring-security.xmlの定義例

.. code-block:: xml

  <sec:http>
      <sec:form-login
          default-target-url="/menu"
          always-use-default-target="true" /> <!-- (1) -->
  </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``always-use-default-target``\ 属性に\ ``true``\ を指定する。

|

AuthenticationSuccessHandlerの適用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityが提供しているデフォルトの動作をカスタマイズする仕組みだけでは要件をみたせない場合は、
以下のようなbean定義を行うことで\ ``AuthenticationSuccessHandler``\ インタフェースの実装クラスを直接適用することができる。

* spring-security.xmlの定義例

.. code-block:: xml

  <bean id="authenticationSuccessHandler" class="com.example.app.security.handler.MyAuthenticationSuccessHandler"> <!-- (1) -->

  <sec:http>
      <sec:form-login authentication-success-handler-ref="authenticationSuccessHandler" /> <!-- (2) -->
  </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``AuthenticationSuccessHandler``\ インタフェースの実装クラスをbean定義する。
    * - | (2)
      - | ``authentication-success-handler-ref``\ 属性に定義した\ ``authenticationSuccessHandler``\ を指定する。

.. warning:: **AuthenticationSuccessHandlerの責務**

    \ ``AuthenticationSuccessHandler``\ は、認証成功時におけるWeb層の処理(主に画面遷移に関する処理)を行うためのインタフェースである。
    そのため、認証失敗回数のクリアなどのビジネスルールに依存する処理（ビジネスロジック）をこのインタフェースの実装クラスを経由して呼び出すべきではない。

    ビジネスルールに依存する処理の呼び出しは、前節で紹介している「:ref:`SpringSecurityAuthenticationEvent`」の仕組みを使用されたい。

|

.. _SpringSecurityAuthenticationCustomizingScreenFlowOnFailure:

認証失敗時のレスポンスのカスタマイズ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

認証失敗時のレスポンスのカスタマイズポイントを説明する。

遷移先の変更
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityのデフォルトの動作では、ログインフォームを表示するためのパスに\ ``"error"``\ というクエリパラメータが付与されたURLにリダイレクトするが、
以下のようなbean定義を行うことで変更することが可能である。

* spring-security.xmlの定義例

.. code-block:: xml

  <sec:http>
      <sec:form-login authentication-failure-url="/loginFailure" /> <!-- (1) -->
  </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - |  (1)
      - | \ ``authentication-failure-url``\ 属性に認証失敗時に遷移するパスを指定する。

|

AuthenticationFailureHandlerの適用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityが提供しているデフォルトの動作をカスタマイズする仕組みだけでは要件をみたせない場合は、
以下のようなbean定義を行うことで\ ``AuthenticationFailureHandler``\ インタフェースの実装クラスを直接適用することができる。

* spring-security.xmlの定義例

.. code-block:: xml

   <!-- (1) -->
  <bean id="authenticationFailureHandler"
      class="org.springframework.security.web.authentication.ExceptionMappingAuthenticationFailureHandler" />
      <property name="defaultFailureUrl" value="/login/systemError" /> <!-- (2) -->
      <property name="exceptionMappings"> <!-- (3) -->
          <props>
              <prop key="org.springframework.security.authentication.BadCredentialsException"> <!-- (4) -->
                  /login/badCredentials
              </prop>
              <prop key="org.springframework.security.core.userdetails.UsernameNotFoundException"> <!-- (5) -->
                  /login/usernameNotFound
              </prop>
              <prop key="org.springframework.security.authentication.DisabledException"> <!-- (6) -->
                  /login/disabled
              </prop>
              <!-- omitted -->
          </props>
      </property>
  </bean>

  <sec:http>
      <sec:form-login authentication-failure-handler-ref="authenticationFailureHandler" /> <!-- (7) -->
  </sec:http>


.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 80
    :class: longtable

    * - | 項番
      - | 説明
    * - | (1)
      - | \ ``AuthenticationFailureHandler``\ インタフェースの実装クラスをbean定義する。
    * - | (2)
      - | \ ``defaultFailureUrl``\ 属性にデフォルトの遷移先のURLを指定する。
        | 下記(4)-(6)の定義に合致しない例外が発生した際は、本設定の遷移先に遷移する。
    * - | (3)
      - | \ ``exceptionMappings``\ プロパティにハンドルする\ ``org.springframework.security.authentication.AuthenticationServiceException``\ の実装クラスと例外発生時の遷移先を \ ``Map``\ 形式で設定する。
        | キーに\ ``org.springframework.security.authentication.AuthenticationServiceException``\ 実装クラスを設定し、値に遷移先URLを設定する。
    * - | (4)
      - | \ ``BadCredentialsException``\ 
        | パスワード照合失敗による認証エラー時にスローされる。
    * - | (5)
      - | \ ``UsernameNotFoundException``\ 
        | 不正ユーザーID（存在しないユーザーID）による認証エラー時にスローされる。
        | ``org.springframework.security.authentication.dao.AbstractUserDetailsAuthenticationProvider``\ を
        | 継承したクラスを認証プロバイダに指定している場合、``hideUserNotFoundExceptions``\ プロパティを\ ``false``\ に変更しないと本例外は、\ ``BadCredentialsException``\ に変更される。
    * - | (6)
      - | \  ``DisabledException``\
        | 無効ユーザーIDによる認証エラー時にスローされる。
    * - | (7)
      - | \ ``authentication-failure-handler-ref``\ 属性に\ ``authenticationFailureHandler``\ を設定する。

.. raw:: latex

   \newpage

.. note:: **例外発生時の制御**

    \ ``exceptionMappings``\ プロパティに定義した例外が発生した場合、例外にマッピングした遷移先にリダイレクトされるが、
    発生した例外オブジェクトがセッションスコープに格納されないため、Spring Securityが生成したエラーメッセージを画面に表示する事ができない。

    そのため、遷移先の画面で表示するエラーメッセージは、リダイレクト先の処理(Controller又はViewの処理)で生成する必要がある。

    また、以下のプロパティを参照する処理が呼び出されないため、設定値を変更しても動作が変わらないという点を補足しておく。

    * ``useForward``
    * ``allowSessionCreation``

|

ログアウト処理のカスタマイズ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ログアウト処理のカスタマイズポイントを説明する。

ログアウトパスの変更
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityのデフォルトでは、ログアウト処理を実行するためのパスは「\ ``"/logout"``\」であるが、
以下のようなbean定義を行うことで変更することが可能である。

* spring-security.xmlの定義例

.. code-block:: xml

  <sec:http>
      <!-- omitted -->
      <sec:logout logout-url="/auth/logout" /> <!-- (1) -->
      <!-- omitted -->
  </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``logout-url``\ 属性を設定し、ログアウト処理を行うパスを指定する。

.. note::

    ログアウトパスを変更した場合は、:ref:`ログアウトフォーム<SpringSecurityAuthenticationLogoutForm>` のリクエスト先も変更する必要がある。

.. tip:: **システムエラー発生時の振る舞い**
    システムエラーが発生した場合は、業務継続不可となるケースが多いと考えられる。
    システムエラー発生後、業務を継続させたくない場合は、以下のような対策を講じることを推奨する。
    
      * システムエラー発生時にセッション情報をクリアする。
      * システムエラー発生時に認証情報をクリアする。
    
    ここでは、共通ライブラリの例外ハンドリング機能を使用してシステム例外発生時に認証情報をクリアする例を説明する。
    例外ハンドリング機能の詳細についは「\ :doc:`../ArchitectureInDetail/WebApplicationDetail/ExceptionHandling`\」を参照されたい。

      .. code-block:: java

        // (1)
        public class LogoutSystemExceptionResolver extends SystemExceptionResolver {
            // (2)
            @Override
            protected ModelAndView doResolveException(HttpServletRequest request,
                    HttpServletResponse response, java.lang.Object handler,
                    java.lang.Exception ex) {

                // SystemExceptionResolverの処理を行う
                ModelAndView resulut = super.doResolveException(request, response,
                        handler, ex);

                // 認証情報をクリアする (2)
                SecurityContextHolder.clearContext();

                return resulut;
            }
        }

      .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
      .. list-table::
          :header-rows: 1
          :widths: 10 90
      
          * - 項番
            - 説明
          * - | (1)
            - | \ ``org.terasoluna.gfw.web.exception.SystemExceptionResolver.SystemExceptionResolver``\ を拡張する。
          * - | (2)
            - | \ 認証情報をクリアする。

    なお、認証情報をクリアする方法以外にも、セッションをクリアすることでも、同様の要件を満たすことができる。
    プロジェクトの要件に合わせて実装されたい。

|

.. _SpringSecurityLogoutCustomizingScreenFlowOnSuccess:

ログアウト成功時のレスポンスのカスタマイズ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ログアウト処理成功時のレスポンスのカスタマイズポイントを説明する。

遷移先の変更
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* spring-security.xmlの定義例

.. code-block:: xml

  <sec:http>
    <!-- omitted -->
    <sec:logout logout-success-url="/logoutSuccess" /> <!-- (1) -->
    <!-- omitted -->
  </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``logout-success-url``\ 属性を設定し、ログアウト成功時に遷移するパスを指定する。

|

LogoutSuccessHandlerの適用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* spring-security.xmlの定義例

.. code-block:: xml
  
  <!-- (1) -->
  <bean id="logoutSuccessHandler" class="com.example.app.security.handler.MyLogoutSuccessHandler" /> 

  <sec:http>
      <!-- omitted -->
      <sec:logout success-handler-ref="logoutSuccessHandler" /> <!-- (2) -->
      <!-- omitted -->
  </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``LogoutSuccessHandler``\ インタフェースの実装クラスをbean定義する。
    * - | (2)
      - | ``success-handler-ref``\ 属性に\ ``LogoutSuccessHandler``\ を設定する。

|

.. _SpringSecurityAuthenticationCustomizingMessage:

エラーメッセージのカスタマイズ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

認証に失敗した場合、Spring Securityが用意しているエラーメッセージが表示されるが、
このエラーメッセージは変更することが可能である。

メッセージ変更方法の詳細については、\ :doc:`../ArchitectureInDetail/WebApplicationDetail/MessageManagement`\ を参照されたい。

システムエラー時のメッセージ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

認証処理の中で予期しないエラー（システムエラーなど）が発生した場合、\ ``InternalAuthenticationServiceException``\ という例外が発生する。
\ ``InternalAuthenticationServiceException``\ が保持するメッセージには、原因例外のメッセージが設定されるため、画面にそのまま表示するのは適切ではない。

例えばユーザー情報をデーターベースから取得する時にDBアクセスエラーが発生した場合、\ ``SQLException``\ が保持する例外メッセージが画面に表示されることになる。
システムエラーの例外メッセージを画面に表示させないためには、\ ``ExceptionMappingAuthenticationFailureHandler``\ を使用して\ ``InternalAuthenticationServiceException``\ をハンドリングし、
システムエラーが発生したことを通知するためのパスに遷移させるなどの対応が必要となる。

* spring-security.xmlの定義例

.. code-block:: xml

    <bean id="authenticationFailureHandler"
        class="org.springframework.security.web.authentication.ExceptionMappingAuthenticationFailureHandler">
        <property name="defaultFailureUrl" value="/login?error" />
        <property name="exceptionMappings">
            <props>
                <prop key="org.springframework.security.authentication.InternalAuthenticationServiceException">
                    /login?systemError
                </prop>
                <!-- omitted -->
            </props>
        </property>
    </bean>

  <sec:http>
      <sec:form-login authentication-failure-handler-ref="authenticationFailureHandler" />
  </sec:http>

|

ここでは、システムエラーが発生したことを識別するためのクエリパラメータ(\ ``systemError``\ )を付けてログインフォームに遷移させている。
遷移先に指定したログインフォームでは、クエリパラメータに\ ``systemError``\ が指定されている場合は、認証例外のメッセージを表示するのではなく、
固定のエラーメッセージを表示するようにしている。

* ログインフォームの実装例

.. code-block:: jsp

    <c:choose>
        <c:when test="${param.containsKey('error')}">
            <span style="color: red;">
                <c:out value="${SPRING_SECURITY_LAST_EXCEPTION.message}"/>
            </span>
        </c:when>
        <c:when test="${param.containsKey('systemError')}">
            <span style="color: red;">
                System Error occurred.
            </span>
        </c:when>
    </c:choose>

.. note::

    ここでは、ログインフォームに遷移させる場合の実装例を紹介したが、システムエラー画面に遷移させてもよい。

|

.. _SpringSecurityAuthenticationBeanValidation:

認証時の入力チェック
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

DBサーバへの負荷軽減等で、認証ページおける、あきらかな入力誤りに対しては、事前にチェックを行いたい場合がある。
このような場合は、Bean Validationを使用した入力チェックも可能である。

Bean Validationによる入力チェック
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

以下にBean Validationを使用した入力チェックの例を説明する。
Bean Validationに関する詳細は \ :doc:`../ArchitectureInDetail/WebApplicationDetail/Validation`\ を参照すること。

* フォームクラスの実装例

.. code-block:: java

    public class LoginForm implements Serializable {

        // omitted
        @NotEmpty // (1)
        private String username;

        @NotEmpty // (1)
        private String password;
        // omitted

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 本例では、\ ``username``\ 、\ ``password``\ をそれぞれ必須入力としている。


* コントローラクラスの実装例

.. code-block:: java

    @ModelAttribute
    public LoginForm setupForm() { // (1)
        return new LoginForm();
    }

    @RequestMapping(value = "login")
    public String login(@Validated LoginForm form, BindingResult result) {
        // omitted
        if (result.hasErrors()) {
            // omitted
        }
        return "forward:/authenticate"; // (2)
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``LoginForm``\ を初期化する。
    * - | (2)
      - | forwardで\ ``<sec:form-login>``\ 要素の\ ``login-processing-url``\ 属性に指定したパスに **Forward** する。
        | 認証に関する設定は、\ :ref:`SpringSecurityAuthenticationCustomizingForm`\を参照すること。

加えて、Forwardによる遷移でもSpring Securityの処理が行われるよう、認証パスをSpring Securityサーブレットフィルタに追加する。

* web.xmlの設定例

.. code-block:: xml

    <filter>
        <filter-name>springSecurityFilterChain</filter-name>
        <filter-class>
            org.springframework.web.filter.DelegatingFilterProxy
        </filter-class>
    </filter>
    <filter-mapping>
        <filter-name>springSecurityFilterChain</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    <!-- (1) -->
    <filter-mapping>
        <filter-name>springSecurityFilterChain</filter-name>
        <url-pattern>/authenticate</url-pattern>
        <dispatcher>FORWARD</dispatcher>
    </filter-mapping>    

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | Forwardで認証するためのパターンを指定する
        | ここでは認証パスである\ ``"/authenticate"``\ を指定している。

|

認証処理の拡張
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityから提供されている\ `認証プロバイダ <http://docs.spring.io/spring-security/site/docs/4.1.4.RELEASE/apidocs/org/springframework/security/authentication/AuthenticationProvider.html>`_\ で対応できない認証要件がある場合は、
\ ``org.springframework.security.authentication.AuthenticationProvider``\ インタフェースを実装したクラスを作成する必要がある。

ここでは、ユーザー名、パスワード、\ **会社識別子(独自の認証パラメータ)**\ の3つのパラメータを使用してDB認証を行うための拡張例を示す。

.. figure:: ./images_Authentication/Authentication_HowToExtends_LoginForm.png
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
      - | ユーザー名、パスワード、会社識別子を保持する\ ``org.springframework.security.core.Authentication``\ インタフェースの実装クラス。
        | ここでは、\ ``org.springframework.security.authentication.UsernamePasswordAuthenticationToken``\ クラスを継承して作成する。
    * - | (2)
      - | ユーザー名、パスワード、会社識別子を使用してDB認証を行う\ ``org.springframework.security.authentication.AuthenticationProvider``\ の実装クラス。
        | ここでは、\ ``org.springframework.security.authentication.dao.DaoAuthenticationProvider``\ クラスを継承して作成する。
    * - | (3)
      - | ユーザー名、パスワード、会社識別子をリクエストパラメータから取得して、\ ``AuthenticationManager``\ (\ ``AuthenticationProvider``\ )に渡す\ ``Authentication``\ を生成するためのAuthentication Filterクラス。
        | ここでは、\ ``org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter``\ クラスを継承して作成する。

.. note::

    ここでは、認証用のパラメータとして独自のパラメータを追加する例にしているため、
    \ ``Authentication``\ インタフェースの実装クラスと\ ``Authentication``\ を生成するためのAuthentication Filterクラスの拡張が必要となる。

    ユーザー名とパスワードのみで認証する場合は、\ ``AuthenticationProvider``\ インタフェースの実装クラスを作成するだけで、
    認証処理を拡張することができる。

|

Authenticationインターフェースの実装クラスの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``UsernamePasswordAuthenticationToken``\ クラスを継承し、ユーザー名とパスワードに加えて、会社識別子(独自の認証パラメータ)を保持するクラスを作成する。

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
     - | 会社識別子を保持するフィールドを作成する。
   * - | (2)
     - | 認証前の情報(リクエストパラメータで指定された情報)を保持するインスタンスを作成する際に使用するコンストラクタを作成する。
   * - | (3)
     - | 認証済みの情報を保持するインスタンスを作成する際に使用するコンストラクタを作成する。
       | 親クラスのコンストラクタの引数に認可情報を渡すことで、認証済みの状態となる。

|

AuthenticationProviderインターフェースの実装クラスの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``DaoAuthenticationProvider``\ クラスを継承し、ユーザー名、パスワード、会社識別子を使用してDB認証を行うクラスを作成する。

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
            String companyId = ((SampleUserDetails) userDetails).getAccount().getCompanyId();
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
     - | 親クラスのメソッドを呼び出し、Spring Securityが提供しているチェック処理を実行する。
       | この処理にはパスワード認証処理も含まれる。
   * - | (2)
     - | パスワード認証が成功した場合は、会社識別子(独自の認証パラメータ)の妥当性をチェックする。
       | 上記例では、リクエストされた会社識別子とテーブルに保持している会社識別子が一致するかをチェックしている。
   * - | (3)
     - | パスワード認証及び独自の認証処理が成功した場合は、認証済み状態の\ ``CompanyIdUsernamePasswordAuthenticationToken``\ を作成して返却する。
   * - | (4)
     - | \ ``CompanyIdUsernamePasswordAuthenticationToken``\ にキャスト可能な\ ``Authentication``\ が指定された場合に、本クラスを使用して認証処理を行うようにする。

.. note::

    ユーザーの存在チェック、ユーザーの状態チェック(無効ユーザー、ロック中ユーザー、利用期限切れユーザーなどのチェック)は、
    \ ``additionalAuthenticationChecks``\ メソッドが呼び出される前に親クラスの処理として行われる。

|

.. _authentication_custom_usernamepasswordauthenticationfilter:

Authentication Filterの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``UsernamePasswordAuthenticationFilter``\ クラスを継承し、
認証情報(ユーザー名、パスワード、会社識別子)を\ ``AuthenticationProvider``\ に引き渡すためのAuthentication Filterクラスを作成する。

\ ``attemptAuthentication``\ メソッドの実装は、\ ``UsernamePasswordAuthenticationFilter``\ クラスのメソッドをコピーしてカスタマイズしたものである。

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
            return request.getParameter("companyId");
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | リクエストパラメータから取得した認証情報(ユーザー名、パスワード、会社識別子)より、\ ``CompanyIdUsernamePasswordAuthenticationToken``\ のインスタンスを生成する。
   * - | (2)
     - | リクエストパラメータで指定された認証情報(\ ``CompanyIdUsernamePasswordAuthenticationToken``\ のインスタンス)を指定して、\ ``org.springframework.security.authentication.AuthenticationManager``\ の\ ``authenticate``\ メソッドを呼び出す。
       | 
       | \ ``AuthenticationManager``\ のメソッドを呼び出すと、\ ``AuthenticationProvider``\ の認証処理が呼び出される。
   * - | (3)
     - | 会社識別子は、\ ``"companyId"``\ というリクエストパラメータより取得する。

|

ログインフォームの修正
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ :ref:`SpringSecurityAuthenticationLoginForm`\ で作成したログインフォーム(JSP)に対して、会社識別子を追加する。

.. code-block:: jsp

    <form:form action="${pageContext.request.contextPath}/login" method="post">
        <!-- omitted -->
            <tr>
                <td><label for="username">User Name</label></td>
                <td><input type="text" id="username" name="username"></td>
            </tr>
            <tr>
                <td><label for="companyId">Company Id</label></td>
                <td><input type="text" id="companyId" name="companyId"></td> <!-- (1) -->
            </tr>
            <tr>
                <td><label for="password">Password</label></td>
                <td><input type="password" id="password" name="password"></td>
            </tr>
        <!-- omitted -->
    </form:form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 会社識別子の入力フィールド名に\ ``"companyId"``\ を指定する。

|

拡張した認証処理の適用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ユーザー名、パスワード、会社識別子(独自の認証パラメータ)を使用したDB認証機能をSpring Securityに適用する。

* spring-security.xmlの定義例

.. code-block:: xml

    <!-- omitted -->

    <!-- (1) -->
    <sec:http
        entry-point-ref="loginUrlAuthenticationEntryPoint">

        <!-- omitted -->

        <!-- (2) -->
        <sec:custom-filter
            position="FORM_LOGIN_FILTER" ref="companyIdUsernamePasswordAuthenticationFilter" />

        <!-- omitted -->

        <sec:csrf token-repository-ref="csrfTokenRepository" />

        <sec:logout
            logout-url="/logout"
            logout-success-url="/login" />

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
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | (2)の\ ``<sec:custom-filter>``\ タグを使用して\ ``"FORM_LOGIN_FILTER"``\ を差し替える場合は、\ ``<sec:http>``\ タグの属性に以下の設定を行う必要がある。

        * 自動設定を使用することができないため、\ ``auto-config="false"``\ を指定するか、\ ``auto-config``\ 属性を削除する。
        * \ ``<sec:form-login>``\ タグが使用できないため、\ ``entry-point-ref``\ 属性を使用して\ ``AuthenticationEntryPoint``\ を明示的に指定する。

    * - | (2)
      - | \ ``<sec:custom-filter>``\ タグを使用して\ ``"FORM_LOGIN_FILTER"``\ を差し替える。
        | 
        | \ ``<sec:custom-filter>``\ タグの\ ``position``\ 属性に\ ``"FORM_LOGIN_FILTER"``\を指定し、\ ``ref``\ 属性に拡張したAuthentication Filterのbeanを指定する。
    * - | (3)
      - | \ ``<sec:http>``\ タグの\ ``entry-point-ref``\ 属性に使用する\ ``AuthenticationEntryPoint``\ のbeanを指定する。
        | 
        | ここでは、\ ``<sec:form-login>``\ タグを指定した際に使用される\ ``org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint``\ クラスのbeanを指定している。
    * - | (4)
      - | \ ``"FORM_LOGIN_FILTER"``\ として使用するAuthentication Filterクラスのbeanを定義する。
        | 
        | ここでは、拡張したAuthentication Filterクラス(\ ``CompanyIdUsernamePasswordAuthenticationFilter``\ )のbeanを定義している。
    * - | (5)
      - | \ ``requiresAuthenticationRequestMatcher``\ プロパティに、認証処理を行うリクエストを検出するための\ ``RequestMatcher``\ インスタンスを指定する。
        | 
        | ここでは、\ ``"/authentication"``\ というパスにリクエストがあった場合に認証処理を行うように設定している。
        | これは、\ ``<sec:form-login>``\ タグの\ ``login-processing-url``\ 属性に\ ``"/authentication"``\ を指定したのと同義である。
    * - | (6)
      - | \ ``authenticationManager``\ プロパティに、\ ``<sec:authentication-manager>``\ タグの\ ``alias``\ 属性に設定した値を指定する。
        | 
        | \ ``<sec:authentication-manager>``\ タグの\ ``alias``\ 属性を指定すると、
        | Spring Securityが生成した\ ``AuthenticationManager``\ のbeanを、他のbeanへDIすることができる様になる。
    * - | (6')
      - | Spring Securityが生成する\ ``AuthenticationManager``\ に対して、拡張した\ ``AuthenticationProvider``\ (\ ``CompanyIdUsernamePasswordAuthenticationProvider``\ )を設定する。
    * - | (7)
      - | \ ``sessionAuthenticationStrategy``\ プロパティに、認証成功時のセッションの取扱いを制御するコンポーネント(\ ``SessionAuthenticationStrategy``\ )のbeanを指定する。
        | 
    * - | (7')
      - | 認証成功時のセッションの取扱いを制御するコンポーネント(\ ``SessionAuthenticationStrategy``\ )のbeanを定義する。
        | 
        | ここでは、Spring Securityから提供されている、
         
        * CSRFトークンを作り直すコンポーネント(\ ``CsrfAuthenticationStrategy``\ )
        * セッション・フィクセーション攻撃を防ぐために新しいセッションを生成するコンポーネント(\ ``SessionFixationProtectionStrategy``\ )
        
        | を有効化している。
    * - | (8)
      - | \ ``authenticationFailureHandler``\ プロパティに、認証失敗時に呼ばれるハンドラクラスを指定する。
    * - | (9)
      - | \ ``authenticationSuccessHandler``\ プロパティに、認証成功時に呼ばれるハンドラクラスを指定する。

.. raw:: latex

   \newpage

.. note:: **auto-configについて**

    \ ``auto-config="false"``\ を指定又は指定を省略した際にBasic認証処理とログアウト処理を有効化したい場合は、\ ``<sec:http-basic>``\ タグと\ ``<sec:logout>``\ タグを明示的に定義する必要がある。

|

.. _AuthenticationHowToExtendUsingDeprecatedPasswordEncoder:

非推奨パッケージのPasswordEncoderの利用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

セキュリティ要件によっては、前述した\ ``PasswordEncoder``\ を実装したクラスでは実現できない場合がある。
特に、既存のアカウント情報で使用しているハッシュ化要件を踏襲する必要がある場合は、前述の\ ``PasswordEncoder``\ では要件を満たせないことがある。

具体的には、既存のハッシュ化要件が以下のようなケースである。

* アルゴリズムがSHA-512である。
* ストレッチング回数が1000回である。
* ソルトがアカウントテーブルのカラムに格納されており、\ ``PasswordEncoder``\ の外から渡す必要がある。

このようなケースでは、\ ``org.springframework.security.crypto.password.PasswordEncoder``\ インタフェースの実装クラスではなく、
\ ``org.springframework.security.authentication.encoding.PasswordEncoder``\ インタフェースの実装クラスの使用することで要件を満たすことができる。

.. warning::

    Spring Security 3.1.4以前では、\ ``org.springframework.security.authentication.encoding.PasswordEncoder``\
    を実装したクラスをハッシュ化に使用していたが、3.1.4以降では非推奨となっている。

|

ShaPasswordEncoderの利用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

本ガイドラインでは、\ ``ShaPasswordEncoder``\ を例に、非推奨パッケージの\ ``PasswordEncoder``\ の利用について説明する。

ハッシュ化要件が以下のケースの場合は、\ ``ShaPasswordEncoder``\ を利用することで要件を満たすことができる。

* アルゴリズムがSHA-512
* ストレッチング回数を1000回

|

まず、\ ``ShaPasswordEncoder``\ のbeanを定義する。

* applicationContext.xmlの定義例

.. code-block:: xml
  
    <bean id ="passwordEncoder"
        class="org.springframework.security.authentication.encoding.ShaPasswordEncoder"> <!-- (1) -->
        <constructor-arg value="512" /> <!-- (2) -->
        <property name="iterations" value="1000" /> <!-- (3) -->
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
  
    * - 項番
      - 説明
    * - | (1)
      - | \ ``org.springframework.security.authentication.encoding.ShaPasswordEncoder``\ のbeanを定義する。
    * - | (2)
      - | SHAアルゴリズムの種類を指定する。
        | 指定可能な値は、「\ ``1``\ 、\ ``256``\ 、\ ``384``\ 、\ ``512``\ 」である。
        | 省略した場合は、「\ ``1``\ 」となる。
    * - | (3)
      - | ハッシュ化時のストレッチング回数を指定する。
        | 省略した場合は、1回となる。

|

次に、\ ``ShaPasswordEncoder``\ をSpring Securityの認証処理(\ ``DaoAuthenticationProvider``\ )に適用する。

* spring-security.xmlの定義例

.. code-block:: xml
  
    <bean id="authenticationProvider"
        class="org.springframework.security.authentication.dao.DaoAuthenticationProvider">
        <!-- omitted -->
        <property name="saltSource" ref="saltSource" /> <!-- (1) -->
        <property name="userDetailsService" ref="userDetailsService" />
        <property name="passwordEncoder" ref="passwordEncoder" /> <!-- (2) -->
    </bean>
  
    <bean id="saltSource"
        class="org.springframework.security.authentication.dao.ReflectionSaltSource"> <!-- (3) -->
        <property name="userPropertyToUse" value="username" /> <!-- (4) -->
    </bean>
  
.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
  
    * - 項番
      - 説明
    * - | (1)
      - | \ ``saltSource``\ プロパティに\ ``org.springframework.security.authentication.dao.SaltSource``\ インタフェースの実装クラスのbeanを指定する。
        | \ ``SaltSource``\ は、ソルトを\ ``UserDetails``\ から取得するためのインタフェースである。
    * - | (2)
      - | \ ``passwordEncoder``\ プロパティに\ ``org.springframework.security.authentication.encoding.PasswordEncoder``\ インタフェースの実装クラスのbeanを指定する。
        | 上記例では、\ ``ShaPasswordEncoder``\ のbeanを指定している。
    * - | (3)
      - | \ ``SaltSource``\ のbeanを定義する。
        | 上記例では、リフレクションを使用して\ ``UserDetails``\ のプロパティからソルトを取得するクラス(\ ``ReflectionSaltSource``\ )を利用している。
    * - | (4)
      - | ソルトが格納されている\ ``UserDetails``\ のプロパティを指定する。
        | 上記例では、\ ``UserDetails``\ の\ ``username``\ プロパティの値をソルトとして使用する。

|

アプリケーションの処理で非推奨の\ ``PasswordEncoder``\ を使用する場合は、\ ``PasswordEncoder``\ をインジェクションして使用する。

* Javaクラスの実装例

.. code-block:: java
  
    @Inject
    PasswordEncoder passwordEncoder;
  
    public String register(Customer customer, String rawPassword, String userSalt) {
        // omitted
        String password = passwordEncoder.encodePassword(rawPassword, userSalt); // (1)
        customer.setPassword(password);
        // omitted
    }
  
    public boolean matches(Customer customer, String rawPassword, String userSalt) {
        return passwordEncoder.isPasswordValid(customer.getPassword(), rawPassword, userSalt); // (2)
    }
  
.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
  
    * - 項番
      - 説明
    * - | (1)
      - | パスワードをハッシュ化する場合は、\ ``encodePassword``\ メソッドを使用する。
        | メソッドの引数には、パスワード、ソルト文字列の順で指定する。
    * - | (2)
      - | パスワードを照合する場合ば、\ ``isPasswordValid``\ メソッドを使用する。
        | メソッドの引数には、ハッシュ化済みのパスワード、平文のパスワード、ソルト文字列の順で指定する。

|

Appendix
--------------------------------------------------------------------------------

.. _spring-security-authentication-mvc:

Spring MVCでリクエストを受けてログインフォームを表示する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Spring MVCでリクエストを受けてログインフォームを表示する方法を説明する。

* spring-mvc.xmlの定義例

ログインフォームを表示するControllerの定義例。

.. code-block:: java

    @Controller
    @RequestMapping("/login")
    public class LoginController { // (1)

        @RequestMapping
        public String index() {
            return "login";
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | view名として"login"を返却する。\ ``InternalResourceViewResolver``\ によってsrc/main/webapp/WEB-INF/views/login.jspが出力される。

本例のように、単純にview名を返すだけのメソッドが一つだけあるControllerであれば、\ ``<mvc:view-controller>``\ を使用して代用することも可能である。  

詳しくは、\ :ref:`controller_method_return-html-label`\を参照されたい。

|

.. _SpringSecurityAuthenticationRememberMe:

Remember Me認証の利用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

「\ `Remember Me認証 <http://docs.spring.io/spring-security/site/docs/4.1.4.RELEASE/reference/htmlsingle/#remember-me>`_\ 」とは、
Webサイトに頻繁にアクセスするユーザーの利便性を高めるための機能の一つで、ログイン状態を通常のライフサイクルより長く保持するための機能である。
本機能を使用すると、ブラウザを閉じた後やセッションタイムが発生した後でも、Cookieに保持しているRemember Me認証用のTokenを使用して、
ユーザ名とパスワードを再入力することなく自動でログインすることができる。
なお、本機能は、ユーザーがログイン状態を保持することを許可した場合のみ有効となる。

Spring Securityは、「`Hash-Based Token <http://docs.spring.io/spring-security/site/docs/4.1.4.RELEASE/reference/htmlsingle/#remember-me-hash-token>`_ 方式のRemember Me認証」と「`Persistent Token <http://docs.spring.io/spring-security/site/docs/4.1.4.RELEASE/reference/htmlsingle/#remember-me-persistent-token>`_ 方式のRemember Me認証」をサポートしており、
デフォルトではHash-Based Token方式が使用される。

|

Remember Me認証を利用する場合は、\ ``<sec:remember-me>``\ タグを追加する。

* spring-security.xmlの定義例

.. code-block:: xml

    <sec:http>
        <!-- omitted -->
        <sec:remember-me key="terasoluna-tourreservation-km/ylnHv"
            token-validity-seconds="#{30 * 24 * 60 * 60}" />  <!-- (1) (2) -->
        <!-- omitted -->
    </sec:http>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``key``\ 属性に、Remember Me認証用のTokenを生成したアプリケーションを識別するキー値を指定する。
        | キー値の指定が無い場合、アプリケーションの起動毎にユニークな値が生成される。
        | なお、Hash-Based Tokenが保持しているキー値とサーバーで保持しているキー値が異なる場合、無効なTokenとして扱われる。
        | つまり、アプリケーションを再起動する前に生成したHash-Based Tokenを有効なTokenとして扱いたい場合は、\ ``key``\ 属性の指定は必須である。
    * - | (2)
      - | \ ``token-validity-seconds``\ 属性に、Remember Me認証用のTokenの有効時間を秒単位で指定する。
        | 指定が無い場合、デフォルトで14日間が有効時間になる。
        | 上記例では、有効時間として30日間を設定している。

上記以外の属性については、\ `Spring Security Reference -The Security Namespace (<remember-me>) - <http://docs.spring.io/spring-security/site/docs/4.1.4.RELEASE/reference/htmlsingle/#nsa-remember-me>`_\ を参照されたい。

.. note:: **Spring Security 4.0における変更**

    Spring Security 4.0から、以下の設定のデフォルト値が変更されている

    * remember-me-parameter
    * remember-me-cookie

|

ログインフォームには、「Remember Me認証」機能の利用有無を指定するためのフラグ(チェックボックス項目)を用意する。

* ログインフォームのJSPの実装例

.. code-block:: jsp

    <form:form action="${pageContext.request.contextPath}/login" method="post">
            <!-- omitted -->
            <tr>
                <td><label for="remember-me">Remember Me : </label></td>
                <td><input name="remember-me" id="remember-me" type="checkbox" checked="checked" value="true"></td> <!-- (1) -->
            </tr>
            <!-- omitted -->
    </form:form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 「Remember Me認証」機能の利用有無を指定するためのフラグ(チェックボックス項目)を追加し、フィールド名(リクエストパラメータ名)には、\ ``remember-me-parameter``\ のデフォルト値である\ ``remember-me``\ を指定する。
        | チェックボックスの\ ``value``\ 属性には、\ ``true``\を設定する。
        | チェックボックスをチェック状態にしてから認証処理を実行すると、以降のリクエストから「Remember Me認証」機能が適用される。

.. tip:: **value属性の設定値について**

    \ ``value``\ 属性には、\ ``true``\を設定する旨が\ `rememberMeRequestedのJavaDoc <http://docs.spring.io/autorepo/docs/spring-security/4.1.4.RELEASE/apidocs/org/springframework/security/web/authentication/rememberme/AbstractRememberMeServices.html#rememberMeRequested-javax.servlet.http.HttpServletRequest-java.lang.String->`_\ に記載されているが、
    実装上は\ ``on``\ 、\ ``yes``\ 、\ ``1``\ も設定可能である。

.. raw:: latex

   \newpage

