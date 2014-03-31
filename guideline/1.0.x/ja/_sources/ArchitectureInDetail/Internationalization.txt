国際化
================================================================================

.. contents:: 目次
   :depth: 3
   :local:

Overview
--------------------------------------------------------------------------------

国際化とは、アプリケーションで表示するラベルやメッセージを、特定の言語のみに固定せず、ロケール(Locale)と呼ばれる言語や国・地域を表す単位の指定により、複数言語の切替に対応させることである。

本章ではメッセージの国際化方法について説明する。

| 国際化に対応するには、以下の対応が必要となる。

* 状態やメッセージ、GUIコンポーネントのラベルなど、テキスト要素をプログラム内でハードコードしない。
* テキスト要素をプログラム以外の外部データに保持する。

国際化を実現するためには、Localeを保持する必要があり、主に

* セッションにLocaleを保持する方法
* CookieにLocaleを保持する方法

がある。

Localeを変更するイメージを以下に示す。

 .. figure:: ./images_Internationalization/i18n_change_image.png
    :alt: locale change image
    :width: 40%

実際にLocaleを保持する場所は実装方法により決定される。

本章では、国際化を使用する上での各種ルールや推奨実装方法を記述する。

    .. note::

      国際化はi18nという略称が広く知られている。
      i18n という記述は、internationalization の先頭の i と語尾の n の間に nternationalizatio の
      18文字があることに起因する。

    .. tip::

      Codelistの国際化方法については、 :doc:`Codelist` を参照されたい。

|

How to use
--------------------------------------------------------------------------------

Localeをユーザ端末（またはブラウザ）の設定により切り替える場合
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| Spring の ``org.springframework.context.support.ResourceBundleMessageSource`` を利用することでLocaleをユーザ端末（またはブラウザ）の設定による切り替えが実現できる。
| ここでは、MessageSourceを利用した場合の、国際化方法について説明する。

    .. tip::

     MessageSourceの詳細や定義方法は、 :doc:`MessageManagement` を参照されたい。


**bean定義ファイル**

- applicationContext.xml

 .. code-block:: xml

    <bean id="messageSource"
        class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basenames">
            <list>
                <value>i18n/application-messages</value>  <!-- (1) -->
            </list>
        </property>
    </bean>

 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (1)
      - | プロパティファイルの基底名として、i18n/application-messagesを指定する。
        | 国際化対応を行う場合、i18nディレクトリ配下にメッセージプロパティファイルを格納することを推奨する。

- spring-mvc.xml

 .. code-block:: xml

    <bean id="localeResolver"
        class="org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver" /> <!-- (1) -->

 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (1)
      - | beanタグのid属性"localeResolver"に ``org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver`` を指定する。
        | このlocaleResolverを使用すると、リクエスト毎にHTTPヘッダー”accept-language”が追加されLocaleが指定される。

 .. note::

  localeResolverが設定されていない場合、デフォルトで ``org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver`` が使用されるため、localeResolverの設定は、省略することもできる。


**ファイルパス**

 .. figure:: ./images_Internationalization/i18n_properties_filepath.png
    :alt: properties filepath
    :width: 40%

| ファイル名は、\ :file:`application-messages_XX.properties`\という形式で作成する。XX部分はLocaleを指定する。
| \ ``LocaleResolver``\によって解決されたLocaleがzhの場合、\ :file:`application-messages_zh.properties`\が使用され、
| \ ``LocaleResolver``\によって解決されたLocaleがjaの場合、\ :file:`application-messages_ja.properties`\が使用される。
| \ ``LocaleResolver``\によって解決されたLocaleに対応するプロパティファイルが存在しない場合、デフォルトとして、\ :file:`application-messages.properties`\が使用される。(ファイル名に"_XX"部分が存在しない)
| \ :file:`application-messages.properties`\を使うときは、以下のことに注意する。

* \ :file:`application-messages.properties`\に定義するメッセージは、デフォルトで使用する言語で作成すること。
* \ :file:`application-messages.properties`\は **必ず作成すること** 。もし存在しない場合、MessageSourceからメッセージを取得できず、JSPにメッセージを設定する際に、JspTagExceptionが発生する。

 .. tip::

   メッセージプロパティファイルの記載方法については、 :doc:`MessageManagement` を参照されたい。

 .. note::

  Localeの判別方法は、以下の順番で該当するLocaleのプロパティファイルが発見されるまで、Localeを確認していくことである。

  #. リクエストのHTTPヘッダー”accept-language”に指定されているLocale
  #. アプリケーションサーバのJVMに指定されているLocale(設定されていない場合あり)
  #. アプリケーションサーバのOSに指定されているLocale

  よく間違える例として、 リクエストのHTTPヘッダー”accept-language”の値に該当するLocaleのプロパティファイルが存在しない場合、デフォルトのプロパティファイルが使用されるとの誤解が挙げられる。
  実際は、次にアプリケーションサーバに指定されているLocaleを確認して、それでも該当するLocaleのプロパティファイルが見つからない場合に、デフォルトのプロパティファイルが使用されるので注意する。

以下、メッセージ定義の設定例を示す。

**プロパティファイル**

- application-messages.properties

 .. code-block:: properties

    title.admin.top = Admin Top

- application-messages_jp.properties

 .. code-block:: properties

    title.admin.top = 管理画面 Top

**JSPファイル**

- include.jsp(インクルード用の共通jspファイル)

 .. code-block:: jsp

  <%@ page session="false"%>
  <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
  <%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt"%>
  <%@ taglib uri="http://www.springframework.org/tags" prefix="spring"%>  <!-- (1) -->
  <%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>
  <%@ taglib uri="http://www.springframework.org/security/tags" prefix="sec"%>
  <%@ taglib uri="http://terasoluna.org/functions" prefix="f"%>
  <%@ taglib uri="http://terasoluna.org/tags" prefix="t"%>

 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (1)
      - | JSPで出力する場合、Springのタグライブラリを用いてメッセージ出力を行うため、カスタムタグを定義する必要がある。
        | ``<%@taglib uri="http://www.springframework.org/tags" prefix="spring"%>`` を定義すること。

 .. note::

  インクルード用の共通jspファイルの詳細は :ref:`view_jsp_include-label` を参照されたい。


- 画面表示用JSPファイル

 .. code-block:: java

  <spring:message code="title.admin.top" />  <!-- (1) -->

 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (1)
      - | JSPでは、Springのタグライブラリである、 ``<spring:message>`` を用いてメッセージ出力を行う。
        | code属性に、プロパティで指定したキーを設定する。
        | 本例では、Localeが、jaの場合、"管理画面 Top"、それ以外のLocaleの場合、"Admin Top"が出力される。

|

Localeを画面操作等で動的に変更する場合
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Localeを画面操作等で動的に変更する方法は、ユーザ端末（ブラウザ）の設定に関係なく、特定の言語を選択させたい場合に有効である。

| ``org.springframework.web.servlet.i18n.LocaleChangeInterceptor`` を用いることで実現できる。
| LocaleChangeInterceptorとは、リクエストパラメータの値に指定された
| Localeの値を用いて、 ``org.springframework.web.servlet.LocaleResolver`` に保存するインタセプターである。

| LocaleResolverの実装クラスは使用するLocaleの保存先により、以下の表から選択する。

 .. list-table:: **Interceptorを利用する場合に使用するLocaleResolverの種類**
    :header-rows: 1
    :widths: 5 60 35

    * - No
      - 実装クラス
      - Locale保存方法
    * - 1.
      - ``org.springframework.web.servlet.i18n.SessionLocaleResolver``
      - | サーバーに保存
    * - 2.
      - ``org.springframework.web.servlet.i18n.CookieLocaleResolver``
      - | クライアントに保存　

.. note::

 LocaleResolverに ``org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver`` を使用する場合、
 ``org.springframework.web.servlet.i18n.LocaleChangeInterceptor`` を使用してLocaleを動的に変更することはできない。

**bean定義ファイル**

SessionLocaleResolver の場合

- spring-mvc.xml

 .. code-block:: xml

  <!-- omitted -->
  <mvc:interceptors>
    <mvc:interceptor>
      <mvc:mapping path="/**" />
      <mvc:exclude-mapping path="/resources/**" />
      <mvc:exclude-mapping path="/**/*.html" />
      <bean
        class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor">  <!-- (1) -->
      </bean>
	  <!-- omitted -->
    </mvc:interceptor>
  </mvc:interceptors>

  <bean id="localeResolver" class="org.springframework.web.servlet.i18n.SessionLocaleResolver">  <!-- (2) -->
      <property name="defaultLocale" value="en"/>  <!-- (3) -->
  </bean>

 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (1)
      - | Spring MVCのインタセプターに、 ``org.springframework.web.servlet.i18n.LocaleChangeInterceptor`` を定義する。
    * - | (2)
      - | beanタグのid属性を"localeResolver"で定義し、 ``org.springframework.web.servlet.LocaleResolver`` を実装したクラスを指定する。
        | 本例では、セッションにLocaleを保存する ``org.springframework.web.servlet.i18n.SessionLocaleResolver`` を指定している。
        | beanタグのid属性は"localeResolver"と設定すること。
        | この設定により、 ``org.springframework.web.servlet.i18n.LocaleChangeInterceptor`` 内で使用されるLocaleResolverが、(3)のLocaleResolverに設定される。
    * - | (3)
      - | リクエストパラメータでLocaleを指定しない場合、defaultLocaleに指定されたLocaleが有効になる。この場合、 ``HttpServletRequest#getLocale`` での取得値が有効になる。

.. _i18n_change_locale_key:

* リクエストパラメータに設定するLocaleのキーを変更する場合

- spring-mvc.xml

 .. code-block:: xml

      <bean
        class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor">
        <property name="paramName" value="lang"/>  <!-- (1) -->
      </bean>

 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (1)
      - | paramNameにリクエストパラメータに設定されたLocaleのキーを指定する。ここの例では"リクエストURL?lang=xx"となる。
        | **キーを指定しない場合、"locale"が設定される。** "リクエストURL?locale=xx"で :ref:`使用可能<i18n_set_locale_jsp>` となる。


CookieLocaleResolverの場合

- spring-mvc.xml

 .. code-block:: xml

  <!-- omitted -->
  <mvc:interceptors>
    <mvc:interceptor>
      <mvc:mapping path="/**" />
      <mvc:exclude-mapping path="/resources/**" />
      <mvc:exclude-mapping path="/**/*.html" />
      <bean
        class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor">  <!-- (1) -->
      </bean>
	  <!-- omitted -->
    </mvc:interceptor>
  </mvc:interceptors>

  <bean id="localeResolver" class="org.springframework.web.servlet.i18n.CookieLocaleResolver">  <!-- (2) -->
        <property name="defaultLocale" value="en"/>  <!-- (3) -->
        <property name="cookieName" value="localeCookie"/>  <!-- (4) -->
  </bean>

 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (1)
      - | Spring MVCのインタセプターに、 ``org.springframework.web.servlet.i18n.LocaleChangeInterceptor`` を定義する。
    * - | (2)
      - | beanタグのid属性"localeResolver"に ``org.springframework.web.servlet.i18n.CookieLocaleResolver`` を指定する。
        | beanタグのid属性は"localeResolver"と設定すること。
        | この設定により、 ``org.springframework.web.servlet.i18n.LocaleChangeInterceptor`` 内で使用されるLocaleResolverが、(3)のLocaleResolverに設定される。
    * - | (3)
      - | Localeを指定しない場合、defaultLocaleに指定されたLocaleが有効になる。この場合、 ``HttpServletRequest#getLocale`` での取得値が有効になる。
    * - | (4)
      - | cookieNameプロパティに指定した値が、cookie名となる。指定しない場合、 ``org.springframework.web.servlet.i18n.CookieLocaleResolver.LOCALE`` となる。springframeworkを使用していることがわかるため、変更することを推奨する。

* リクエストパラメータに設定するLocaleのキーを変更する場合

SessionLocaleResolverと :ref:`設定<i18n_change_locale_key>` は同様である。


以下、プロパティの設定例を示す。

**プロパティファイル**

- application-messages.properties

 .. code-block:: properties

    i.xx.yy.0001 = changed locale
    i.xx.yy.0002 = Confirm change of locale at next screen

- application-messages_ja.properties

 .. code-block:: properties

    i.xx.yy.0001 = Localeを変更しました。
    i.xx.yy.0002 = 次の画面でのLocale変更を確認

.. _i18n_set_locale_jsp:

**JSPファイル**

- 画面表示用JSPファイル

 .. code-block:: jsp

    <a href='${pageContext.request.contextPath}?locale=en'>English</a>  <!-- (1) -->
    <a href='${pageContext.request.contextPath}?locale=ja'>Japanese</a>
    <spring:message code="i.xx.yy.0001" />

 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (1)
      - | LocaleChangeInterceptorのparamNameで指定された、リクエストパラメータのキーを送信する。
        | 上記例の場合、Englishリンクで英語Locale、Japaneseリンクで日本語Localeに変更している。
        | 以降は、選択したLocaleが有効になる。
        | 英語Localeは"en"用のプロパティファイルが存在しないため、デフォルトのプロパティファイルから読み込まれる。

 .. tip::

     * インクルード用の共通jspにSpringのタグライブラリを定義する必要がある。
     * インクルード用の共通jspファイルの詳細は :ref:`view_jsp_include-label` を参照されたい。

画面操作でLocaleを変更する場合のイメージを以下に示す。

.. figure:: ./images_Internationalization/i18n_change_locale_on_screen.png
   :alt: i18n change locale on screen
   :width: 30%
   :align: center

