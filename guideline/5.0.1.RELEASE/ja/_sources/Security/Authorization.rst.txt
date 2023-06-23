認可
================================================================================

.. only:: html

 .. contents:: 目次
    :local:

Overview
--------------------------------------------------------------------------------
| 本節では、Spring Securityで提供している認可機能を説明する。

| Spring Securityのアクセス認可機能を利用して実現するため、Spring Securityの認証機能を用いることを前提とする。
| Spring Securityを利用した認証方法については、\ :doc:`Authentication`\ を参照されたい。

| アクセス認可の対象のリソースは、以下の3項目である。

#. Web(リクエストURL)

   * 特定のURLにアクセスするために必要な権限を設定できる

#. 画面項目(JSP）

   * 画面中の特定の要素を表示するために必要な権限を設定できる

#. メソッド

   * 特定のメソッドを実行するために必要な権限を設定できる

| Spring Securityでは、設定ファイルやアノテーションでアクセス認可情報を記述し、機能を実現している。


アクセス認可(リクエストURL)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. figure:: ./images/Authorization_Filter_overview.png
   :alt: Authorization(リクエストURL)
   :width: 60%

   **Picture - Authorization(リクエストURL)**

#. ユーザのリクエストに対し、Spring Securityのフィルタチェーンが割り込み処理を行う。
#. 認可制御の対象となるURLとリクエストのマッチングを行い、アクセス認可マネージャにアクセス認可の判断を問い合わせる。
#. アクセス認可マネージャが、ユーザの権限とアクセス認可情報をチェックし、
   必要なロールが割り当てられていない場合は、アクセス拒否例外をスローする。
#. 必要なロールが割り当てられている場合は、処理を継続する。

|

アクセス認可(JSP)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. figure:: ./images/Authorization_Jsp_overview.png
   :alt: Authorization(JSP)
   :width: 60%

   **Picture - Authorization(JSP)**

#. JSPから生成されたサーブレットが、アクセス認可マネージャに問い合わせる。
#. アクセス認可マネージャが、ユーザの権限とアクセス認可情報をチェックし、
   必要なロールが割り当てられていない場合は、タグの内部を評価しない。
#. 必要なロールが割り当てられている場合は、タグの内部を評価する。

|

アクセス認可(Method)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. figure:: ./images/Authorization_Method_overview.png
   :alt: Authorization(Method)
   :width: 60%

   **Picture - Authorization(Method)**

#. Springコンテナがアクセス認可情報をもとに、対象のオブジェクトに対してインターセプタを生成、割り込みさせる。
#. インターセプタは設定されたロールをもとにアクセス認可マネージャに問い合わせる。
#. アクセス認可マネージャが、ユーザが持つ権限とアクセス認可情報をチェックし、
   必要なロールが割り当てられていない場合はアクセス拒否例外をスローする。
#. 必要なロールが割り当てられている場合は、処理を継続する（設定により、処理を実行した後に権限をチェックすることもできる）。

|

How to use
--------------------------------------------------------------------------------
| アクセス認可(リクエストURL)、アクセス認可(JSP)、アクセス認可(Method)の使用方法について説明する。

アクセス認可(リクエストURL)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| アクセス認可(リクエストURL)機能を使用するために、Spring Securityの設定ファイルに記述する内容を以下に示す。
| 基本設定については、\ :doc:`SpringSecurity`\ を参照されたい。

.. _authorization-intercept-url:

\ ``<sec:intercept-url>``\ 要素の設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| \ ``<sec:http>``\ 要素の子要素である\ ``<sec:intercept-url>``\ 要素に制御対象とするURL、認可するロールを記述することで、
| URLのパス単位で認可制御を行うことができる。

| 以下に、設定例を記載する。

* spring-security.xml

  .. code-block:: xml
  
    <sec:http auto-config="true" use-expressions="true">
        <sec:intercept-url pattern="/admin/*" access="hasRole('ROLE_ADMIN')"/>
        <!-- omitted -->
    </sec:http>
  
  .. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 20 80
  
     * - | 属性名
       - | 説明
     * - | \ ``pattern``\ 
       - | アクセス認可を行う対象のURLパターンを記述する。ワイルドカード「*」、「**」が使用できる。
         | 「*」では、同一階層のみが対象であるのに対し、「**」では、指定階層以下の全URLが、認可設定の対象となる。
     * - | \ ``access``\ 
       - | Spring EL式でのアクセス制御式や、アクセス可能なロールを指定する。
     * - | \ ``method``\ 
       - | HTTPメソッド（GETやPOST等）を指定する。指定したメソッドのみに関して、URLパターンとマッチングを行う。
         | 指定しない場合は、任意のHTTPメソッドに適用される。主にRESTを利用したWebサービスの利用時に活用できる。
     * - | \ ``requires-channel``\ 
       - | 「http」、もしくは「https」を指定する。指定したプロトコルでのアクセスを強制する。
         | 指定しない場合、どちらでもアクセスできる。

  | 上記以外の属性については、\ `<intercept-url> <http://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/reference/htmlsingle/#nsa-intercept-url>`_\ を参照されたい。

| ログインユーザーに「ROLE_USER」「ROLE_ADMIN」というロールがある場合を例に、設定例を示す。

* spring-security.xml

  .. code-block:: xml
  
    <sec:http auto-config="true" use-expressions="true">
        <sec:intercept-url pattern="/reserve/*" access="hasAnyRole('ROLE_USER','ROLE_ADMIN')" /> <!-- (1) -->
        <sec:intercept-url pattern="/admin/*" access="hasRole('ROLE_ADMIN')" /> <!-- (2) -->
        <sec:intercept-url pattern="/**" access="denyAll" /> <!-- (3) -->
        <!-- omitted -->
    </sec:http>
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - | 項番
       - | 説明
     * - | (1)
       - | 「/reserve/\*」にアクセスするためには、「ROLE_USER」もしくは「ROLE_ADMIN」ロールが必要である。
         | \ ``hasAnyRole``\ については、後述する。
     * - | (2)
       - | 「/admin/\*」にアクセスするためには、「ROLE_ADMIN」ロールが必要である。
         | \ ``hasRole``\ については、後述する。
     * - | (3)
       - | \ ``denyAll``\ を全てのパターンに設定し、
         | 権限設定が記述されていないURLに対してはどのユーザもアクセス出来ない設定としている。
         | \ ``denyAll``\ については、後述する。

  .. note::    **URLパターンの記述順序について**

     クライアントからのリクエストに対して、intercept-urlで記述されているパターンに、上から順にマッチさせ、
     マッチしたパターンに対してアクセス認可を行う。そのため、パターンの記述は、必ず、より限定されたパターンから記述すること。

| \ ``<sec:http>``\ 属性に\ ``use-expressions="true"``\ の設定をしたことで、Spring EL式が有効になる。
| \ ``access``\ 属性に記述したSpring EL式は真偽値で評価され、式が真の場合に、アクセスが認可される。
| 以下に、使用例を示す。

* spring-security.xml

  .. code-block:: xml
  
    <sec:http auto-config="true" use-expressions="true">
        <sec:intercept-url pattern="/admin/*" access="hasRole('ROLE_ADMIN')"/>  <!-- (1) -->
        <!-- omitted -->
    </sec:http>
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | \ ``hasRole('ロール名')``\ を指定することで、ログインユーザが指定したロールを保持していれば真を返す。
  
  .. _spring-el:
  
  | **使用可能なExpression一覧例**
  
  .. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 30 70
  
     * - 属性名
       - 説明
     * - | \ ``hasRole('ロール名')``\ 
       - | ユーザが指定したロールを保持していれば、真を返す。
     * - | \ ``hasAnyRole('ロール1','ロール2')``\ 
       - | ユーザが指定したいずれかのロールを保持していれば、真を返す。
     * - | \ ``permitAll``\ 
       - | 常に真を返す。認証されていない場合も、アクセスできることに注意する。
     * - | \ ``denyAll``\ 
       - | 常に偽を返す。
     * - | \ ``isAnonymous()``\ 
       - | 匿名ユーザであれば、真を返す。
     * - | \ ``isAuthenticated()``\ 
       - | 認証されたユーザならば、真を返す。
     * - | \ ``isFullyAuthenticated()``\ 
       - | 匿名ユーザ、もしくはRememberMe機能での認証であれば、偽を返す。
     * - | \ ``hasIpAddress('IPアドレス')``\ 
       - | リクエストURL、およびJSPタグへのアクセス認可のみで、有効となる。
         | 指定のIPアドレスからのリクエストであれば、真を返す。
  
  | その他、使用可能なSpring EL式は、 \ `Common built-in expressions <http://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/reference/htmlsingle/#el-common-built-in>`_\ を参照されたい。
  
  | 演算子を使用した判定も行うことができる。
  | 以下の例では、ロールと、リクエストされたIPアドレス両方に合致した場合、アクセス可能となる。

* spring-security.xml

  .. code-block:: xml
  
    <sec:http auto-config="true" use-expressions="true">
        <sec:intercept-url pattern="/admin/*" access="hasRole('ROLE_ADMIN') and hasIpAddress('192.168.10.1')"/>
        <!-- omitted -->
    </sec:http>
  
  | **使用可能な演算子一覧**
  
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


アクセス認可制御を行わないURLの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| トップページやログイン画面、cssファイルへのパスなど、認証が必要のないURLに対しては、
| http要素のpattern属性、およびsecurity属性を利用する。

  * spring-security.xml
  
  .. code-block:: xml
  
    <sec:http pattern="/css/*" security="none"/>  <!-- 属性の指定順番で(1)～(2) -->
    <sec:http pattern="/login" security="none"/>
    <sec:http auto-config="true" use-expressions="true">
        <!-- omitted -->
    </sec:http>
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - | 項番
       - | 説明
     * - | (1)
       - | \ ``pattern``\ 属性に設定を行う対象のURLパターンを記述する。\ ``pattern``\ 属性を記述しない場合、すべてのパターンにマッチする。
     * - | (2)
       - | \ ``security``\ 属性に\ ``none``\ を指定することで、\ ``pattern``\ 属性に記述されたパスは、Spring Securityフィルタチェインを回避することができる。


URLパターンでの例外処理
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 認可されていないURLにアクセスした場合、\ ``org.springframework.security.access.AccessDeniedException``\ がスローされる。
| デフォルトの設定では、\ ``org.springframework.security.web.access.ExceptionTranslationFilter``\ に設定された
| \ ``org.springframework.security.web.access.AccessDeniedHandlerImpl``\ が、エラーコード403を返却する。
| http要素に、アクセス拒否時のエラーページを設定することで、アクセス拒否時に指定のエラーページに遷移させることができる。

* spring-security.xml

  .. code-block:: xml
  
    <sec:http auto-config="true" use-expressions="true">
        <!-- omitted -->
        <sec:access-denied-handler error-page="/accessDeneidPage" />  <!-- (1) -->
    </sec:http>
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - | 項番
       - | 説明
     * - | (1)
       - | \ ``<sec:access-denied-handler>``\ 要素の\ ``error-page``\ 属性に、遷移先のパスを指定する。


アクセス認可(JSP)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 画面表示項目を制御するには、Spring Securityが提供しているカスタムJSPタグ\ ``<sec:authorize>``\ を利用する。
| ``<%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags" %>``
| のタグライブラリの使用宣言設定をされていることが、前提条件である。

* \ ``<sec:authorize>``\ タグの属性一覧

  .. tabularcolumns:: |p{0.15\linewidth}|p{0.85\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 15 85
  
     * - | 属性名
       - | 説明
     * - | \ ``access``\ 
       - | アクセス制御式を記述する。真であれば、タグ内が評価される。
     * - | \ ``url``\ 
       - | 設定したURLに対して権限が与えられている場合に、タグ内が評価される。リンクの表示の制御等に利用する。
     * - | \ ``method``\ 
       - | HTTPメソッド（GETやPOST等）を指定する。 url属性と合わせて利用し、指定したメソッドのみに関して、
         | 指定したURLパターンとマッチングを行う。指定しない場合、GETが適用される。
     * - | \ ``ifAllGranted``\ 
       - | 設定したロールが全て与えられている場合に、タグ内が評価される。ロール階層機能は効かない。
     * - | \ ``ifAnyGranted``\ 
       - | 設定したロールについて、いずれかが与えられている場合に、タグ内が評価される。ロール階層機能は効かない。
     * - | \ ``ifNotGranted``\ 
       - | 設定されたロールが与えられていない場合、タグの中身が評価される。ロール階層機能は効かない。
     * - | \ ``var``\ 
       - | タグの評価結果を格納するpageスコープの変数を宣言する。同等の権限チェックをページ内で行う場合に利用する。

| 以下に、\ ``<sec:authorize>``\ タグの使用例を示す。

* spring-security.xml

  .. code-block:: jsp
  
    <div>
      <sec:authorize access="hasRole('ROLE_USER')">  <!-- (1) -->
          <p>This screen is for ROLE_USER</p>
      </sec:authorize>
      <sec:authorize url="/admin/menu">  <!-- (2) -->
          <p>
            <a href="/admin/menu">Go to admin screen</a>
          </p>
      </sec:authorize>
    </div>
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - | 項番
       - | 説明
     * - | (1)
       - | 「ROLE_USER」を持つ場合のみ、タグ内が表示される。
     * - | (2)
       - | 「/admin/menu」に対してアクセスが認可されている場合、タグ内が表示される。

  .. warning::

     \ ``<sec:authorize>``\ タグによる認可処理は、\ **画面表示の制御でしかない**\ ため、特定の権限でリンクを表示されなくても、URLが推測されれば、直接リンク先のURLにアクセスできてしまう。
     そのため、必ず、前述の「アクセス認可(リクエストURL)」、もしくは、後述の「アクセス認可(Method)」を併用して、本質的な認可制御をに行うこと。


アクセス認可(Method)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| メソッドに対して、認可制御ができる。
| SpringのDIコンテナで管理されているBeanが、認可の対象となる。

| 前述の2つの認可方法はアプリケーション層での認可制御であったが、
| メソッドレベルの認可制御はドメイン層(Serviceクラス)に対して行う。
| 制御したいメソッドに対して\ ``org.springframework.security.access.prepost.PreAuthorize``\ アノテーションを設定すればよい。

* spring-security.xml

  .. code-block:: xml
  
    <sec:global-method-security pre-post-annotations="enabled"/>  <!-- (1) -->
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - | 項番
       - | 説明
     * - | (1)
       - | \ ``<sec:global-method-security>``\ 要素の\ ``pre-post-annotations``\ 属性を\ ``enabled``\ に指定する。
         | デフォルトは\ ``disabled``\ である。

* Javaコード

  .. code-block:: java

    @Service
    @Transactional
    public class UserServiceImpl implements UserService {
        // omitted

        @PreAuthorize("hasRole('ROLE_ADMIN')") // (1)
        @Override
        public User create(User user) {
           // omitted
        }


        @PreAuthorize("isAuthenticated()")
        @Override
        public User update(User user) {
           // omitted
        }
    }

  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - | 項番
       - | 説明
     * - | (1)
       - | アクセス制御式を記述する。メソッドを実行する前に式が評価され、真であれば、メソッドが実行される。
         | 偽であれば、\ ``org.springframework.security.access.AccessDeniedException``\ がスローされる。
         | 設定可能な値は、\ :ref:`authorization-intercept-url`\ で述べたExpressionや、および
         | \ `Spring Expression Language (SpEL) <http://docs.spring.io/spring/docs/4.1.7.RELEASE/spring-framework-reference/html/expressions.html>`_\ で記述された式である。

  .. tip::
  
    上記の設定では\ ``org.springframework.security.access.prepost.PreAuthorize``\ 以外にも、以下のアノテーションを使用できる。
  
    * \ ``org.springframework.security.access.prepost.PostAuthorize``\ 
    * \ ``org.springframework.security.access.prepost.PreFilter``\ 
    * \ ``org.springframework.security.access.prepost.PostFilter``\ 
  
    これらの詳細は\ `Spring Security マニュアル <http://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/reference/htmlsingle/#el-pre-post-annotations>`_\ を参照されたい。

  .. note::

    Spring SecurityではJava標準であるJSR-250の\ ``javax.annotation.security.RolesAllowed``\ アノテーションによる認可制御も可能であるが、
    \ ``@RolesAllowed``\ ではSpELによる記述ができない。\ ``@PreAuthorize``\ であればSpELを用いて、spring-security.xmlの設定と同じ記法で認可制御


  .. note::
  
    リクエストパスに対する認可制御はControllerのメソッドにアノテーションをつけるのではなく、spring-security.xmlに設定を行うことを推奨する。
    
    ServiceがWeb経由でしか実行されず、リクエストパスのすべてのパターンが認可制御されているのであればServiceの認可制御は行わなくても良い。
    Serviceがどこから実行されるか分からず、認可制御が必要な場合にアノテーションを使用するとよい。

How to extend
--------------------------------------------------------------------------------

ロール階層機能
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| ロールに階層関係を設定することができる。
| 上位に設定したロールは、下位ロールに認可されたすべてのアクセスが可能となる。
| ロールの関係が複雑な場合は、階層機能を検討されたい。

| ROLE_ADMINを上位ロール、ROLE_USERを下位ロールとして階層関係を設定する例で説明する。

.. figure:: ./images/Authorization_RoleHierarchy.png
   :alt: RoleHierarchy
   :width: 30%
   :align: center

   **Picture - RoleHierarchy**

| このとき、下記のようにアクセス認可を設定すると、
| 「ROLE_ADMIN」のロールを持つユーザも、「/user/\*」のURLにアクセスできる。

**Spring Security 設定ファイル**

.. code-block:: xml

  <sec:http auto-config="true" use-expressions="true">
      <sec:intercept-url pattern="/user/*" access="hasAnyRole('ROLE_USER')" />
      <!-- omitted -->
  </sec:http>

| アクセス認可(リクエストURL)、アクセス認可(JSP)、アクセス認可(Method)のそれぞれで設定方法が異なるため、
| 使用方法について、以降で説明する。


共通設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 共通で必要な設定について述べる。
| 階層関係を管理する\ ``org.springframework.security.access.hierarchicalroles.RoleHierarchy`` クラスのBean定義を行う。

* spring-security.xml

  .. code-block:: xml
  
    <bean id="roleHierarchy"
        class="org.springframework.security.access.hierarchicalroles.RoleHierarchyImpl"> <!-- (1) -->
        <property name="hierarchy">
            <value> <!-- (2) -->
                ROLE_ADMIN > ROLE_STAFF
                ROLE_STAFF > ROLE_USER
            </value>
        </property>
    </bean>
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - | 項番
       - | 説明
     * - | (1)
       - | \ ``RoleHierarchy``\ のデフォルト ``org.springframework.security.access.hierarchicalroles.RoleHierarchyImpl`` クラスを指定する。
     * - | (2)
       - | \ ``hierarchy``\ プロパティに階層関係を定義する。
         | 書式:
         | [上位ロール] > [下位ロール]
         | 例では、STAFFはUSERに認可されたすべてのリソースに、アクセスできる。
         | ADMINはUSER、STAFFに認可されたすべてのリソースに、アクセスできる。


アクセス認可(リクエストURL)、アクセス認可(JSP)での使用方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| リクエストURL、JSPに対するロール階層の設定について述べる。

* spring-security.xml

  .. code-block:: xml
  
    <bean id="webExpressionHandler"
        class="org.springframework.security.web.access.expression.DefaultWebSecurityExpressionHandler">  <!-- (1) -->
        <property name="roleHierarchy" ref="roleHierarchy"/>  <!-- (2) -->
    </bean>
  
    <sec:http auto-config="true" use-expression="true">
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
       - | クラスに\ ``org.springframework.security.web.access.expression.DefaultWebSecurityExpressionHandler``\ を指定する。
     * - | (2)
       - | \ ``roleHierarchy``\ プロパティに\ ``RoleHierarchy``\ のBean IDをプロパティに設定する。
     * - | (3)
       - | \ ``expression-handler``\ 要素に、\ ``org.springframework.security.access.expression.SecurityExpressionHandler``\ を実装したハンドラクラスのBean IDを指定する。


アクセス認可(Method)での使用方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Serviceのメソッドにアノテーションをつけて認可制御を行う場合のロール階層設定について説明する。


* spring-security.xml

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
  
     * - | 項番
       - | 説明
     * - | (1)
       - | クラスに\ ``org.springframework.security.access.expression.method.DefaultMethodSecurityExpressionHandler``\ を指定する。
     * - | (2)
       - | \ ``roleHierarchy``\ プロパティに\ ``RoleHierarchy``\ のBean IDをプロパティに設定する。
     * - | (3)
       - | \ ``expression-handler``\ 要素に、\ ``org.springframework.security.access.expression.SecurityExpressionHandler``\ を実装したハンドラクラスのBean IDを指定する。

.. raw:: latex

   \newpage

