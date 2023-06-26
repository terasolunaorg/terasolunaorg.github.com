国際化
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

Overview
--------------------------------------------------------------------------------

国際化とは、アプリケーションで表示するラベルやメッセージを、特定の言語のみに固定せず、ロケール(Locale)と呼ばれる言語や国・地域を表す単位の指定により、複数言語の切替に対応させることである。

本節では、画面に表示するメッセージを国際化する方法について説明する。

国際化するためには、以下の対応が必要となる。

* 画面内のテキスト要素（コード値の名称、メッセージ、GUIコンポーネントのラベルなど）は、プログラム内でハードコードせずに、プロパティファイルなどの外部定義から取得する。
* クライアントからLocaleを指定する仕組みを提供する。

クライアントからLocaleを指定する方法は通りである。

* 標準のリクエストヘッダを使用する。(ブラウザの言語設定で指定)
* リクエストパラメータを使用してCookieに保存する。
* リクエストパラメータを使用してSessionに保存する。

Localeの切り替えイメージを以下に示す。

.. figure:: ./images_Internationalization/i18n_change_image.png
    :alt: locale change image
    :width: 90%

.. note::

    Codelistの国際化方法については、 :doc:`../WebApplicationDetail/Codelist` を参照されたい。

.. note::

    エラー画面を国際化する必要がある場合、Spring MVCのControllerを使用してエラー画面に遷移すること。
    Spring MVCを介さずエラー画面に直接遷移した場合、メッセージが意図した言語で出力されない場合がある。

.. tip::

    国際化はi18nという略称が広く知られている。
    i18n という記述は、internationalization の先頭の i と語尾の n の間に nternationalizatio の
    18文字があることに起因する。

|

How to use
--------------------------------------------------------------------------------

メッセージ定義の設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

画面に表示するメッセージを国際化する場合は、メッセージを管理するためのコンポーネント(\ ``MessageSource``\)として、

* ``org.springframework.context.support.ResourceBundleMessageSource``
* ``org.springframework.context.support.ReloadableResourceBundleMessageSource``

のどちらかを使用する。

ここでは、\ ``ResourceBundleMessageSource``\ を使用する場合の設定例を紹介する。

**applicationContext.xml**

.. code-block:: xml

    <bean id="messageSource"
        class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basenames">
            <list>
                <value>i18n/application-messages</value>  <!-- (1) -->
            </list>
        </property>
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (1)
      - | プロパティファイルの基底名として、\ ``i18n/application-messages``\ を指定する。
        | 国際化対応を行う場合、i18nディレクトリ配下にメッセージプロパティファイルを格納することを推奨する。
        |
        | \ ``MessageSource``\ の詳細や定義方法は、 :doc:`../WebApplicationDetail/MessageManagement` を参照されたい。

|

**プロパティファイルの格納例**

.. figure:: ./images_Internationalization/i18n_properties_filepath.png
    :alt: properties filepath
    :width: 50%

プロパティファイルは、以下のルールに則って作成する。

* Locale毎のファイル名は、\ :file:`application-messages_XX.properties`\という形式で作成する。(XX部分はLocaleを指定)
* \ :file:`application-messages.properties`\は **必ず作成する** 。もし存在しない場合、\ ``MessageSource``\ からメッセージを取得できず、JSPにメッセージを設定する際に、\ ``JspTagException``\ が発生する。
* \ :file:`application-messages.properties`\に定義するメッセージは、デフォルトで使用する言語で作成する。

上記ルールに則ってプロパティファイルを作成すると、以下のような動作になる。

* クライアントのLocaleがzhの場合、\ :file:`application-messages_zh.properties`\が使用される。
* クライアントのLocaleがjaの場合、\ :file:`application-messages_ja.properties`\が使用される。
* クライアントのLocaleに対応するプロパティファイルが存在しない場合、デフォルトとして、\ :file:`application-messages.properties`\が使用される。(ファイル名に"_XX"部分が存在しないファイル)

.. note::

  Localeの判別方法は、以下の順番で該当するLocaleのプロパティファイルが発見されるまで、Localeを確認していくことである。

  #. クライアントから指定されたLocale
  #. アプリケーションサーバのJVMに指定されているLocale(設定されていない場合あり)
  #. アプリケーションサーバのOSに指定されているLocale

  よく間違える例として、 クライアントから指定されたLocaleのプロパティファイルが存在しない場合、デフォルトのプロパティファイルが使用されるとの誤解が挙げられる。
  実際は、次にアプリケーションサーバに指定されているLocaleを確認して、それでも該当するLocaleのプロパティファイルが見つからない場合に、デフォルトのプロパティファイルが使用されるので注意する。

.. tip::

   メッセージプロパティファイルの記載方法については、 :doc:`../WebApplicationDetail/MessageManagement` を参照されたい。

|

Localeをブラウザの設定により切り替える
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

AcceptHeaderLocaleResolverの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ブラウザの設定を使用してLocaleを切り替える場合は、\ ``AcceptHeaderLocaleResolver``\ を使用する。

**spring-mvc.xml**

.. code-block:: xml

    <bean id="localeResolver"
        class="org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver" /> <!-- (1) -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (1)
      - | beanタグのid属性"localeResolver"に ``org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver`` を指定する。
        | この\ ``LocaleResolver``\ を使用すると、リクエスト毎に設定されるHTTPヘッダー(”accept-language”)に指定されているLocaleが使用される。

.. note::

  \ ``LocaleResolver``\ が設定されていない場合、デフォルトで ``org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver`` が使用されるため、\ ``LocaleResolver``\ の設定は、省略することもできる。

|

メッセージの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

以下に、メッセージの設定例を示す。

**application-messages.properties**

.. code-block:: properties

    title.admin.top = Admin Top

**application-messages_ja.properties**

.. code-block:: properties

    title.admin.top = 管理画面 Top

|

JSPの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

以下に、JSPの実装例を示す。

**include.jsp(インクルード用の共通jspファイル)**

.. code-block:: jsp

  <%@ page session="false"%>
  <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
  <%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt"%>
  <%@ taglib uri="http://www.springframework.org/tags" prefix="spring"%>  <!-- (1) -->
  <%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>
  <%@ taglib uri="http://www.springframework.org/security/tags" prefix="sec"%>
  <%@ taglib uri="http://terasoluna.org/functions" prefix="f"%>
  <%@ taglib uri="http://terasoluna.org/tags" prefix="t"%>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
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

|

**画面表示用JSPファイル**

.. code-block:: jsp

  <spring:message code="title.admin.top" />  <!-- (2) -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (2)
      - | JSPでは、Springのタグライブラリである、 ``<spring:message>`` を用いてメッセージ出力を行う。
        | code属性に、プロパティで指定したキーを設定する。
        | 本例では、Localeが、jaの場合、"管理画面 Top"、それ以外のLocaleの場合、"Admin Top"が出力される。

|

Localeを画面操作等で動的に変更する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Localeを画面操作等で動的に変更する方法は、ユーザ端末（ブラウザ）の設定に関係なく、特定の言語を選択させたい場合に有効である。

画面操作でLocaleを変更する場合のイメージを以下に示す。

.. figure:: ./images_Internationalization/i18n_change_locale_on_screen.png
    :alt: i18n change locale on screen
    :align: center
    :width: 40%

ユーザが使用する言語を選択する場合は、\ ``org.springframework.web.servlet.i18n.LocaleChangeInterceptor``\ を用いることで実現する事ができる。

\ ``LocaleChangeInterceptor``\ は、リクエストパラメータに指定されたLocaleの値を、
\ ``org.springframework.web.servlet.LocaleResolver``\ のAPIを使用してサーバ又はクライアントに保存するためのインタセプターである。

使用する\ ``LocaleResolver``\ の実装クラスを、以下の表から選択する。

.. tabularcolumns:: |p{0.05\linewidth}|p{0.60\linewidth}|p{0.35\linewidth}|
.. list-table:: **LocaleResolverの種類**
    :header-rows: 1
    :widths: 5 60 35

    * - No
      - 実装クラス
      - Localeの保存方法
    * - 1.
      - ``org.springframework.web.servlet.i18n.SessionLocaleResolver``
      - | サーバーに保存(\ ``HttpSession``\ を使用)
    * - 2.
      - ``org.springframework.web.servlet.i18n.CookieLocaleResolver``
      - | クライアントに保存(\ ``Cookie``\ を使用)

.. note::

 \ ``LocaleResolver``\ に\ ``org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver``\ を使用する場合、
 \ ``org.springframework.web.servlet.i18n.LocaleChangeInterceptor``\ を使用してLocaleを動的に変更することはできない。

|

LocaleChangeInterceptorの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

リクエストパラメータを使用してLocaleを切り替える場合は、\ ``LocaleChangeInterceptor``\ を使用する。

**spring-mvc.xml**

.. code-block:: xml

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

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (1)
      - | Spring MVCのインタセプターに、 ``org.springframework.web.servlet.i18n.LocaleChangeInterceptor`` を定義する。
        | この設定により、"リクエストURL?locale=xx"で :ref:`使用可能<i18n_set_locale_jsp>` となる。

.. note::

    **Localeを指定するリクエストパラメータ名の変更方法**

     .. code-block:: xml

        <bean
            class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor">
            <property name="paramName" value="lang"/>  <!-- (2) -->
        </bean>

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90

        * - | 項番
          - | 説明
        * - | (2)
          - | \ ``paramName``\ プロパティにリクエストパラメータ名を指定する。上記例では、"リクエストURL?lang=xx"となる。

|

SessionLocaleResolverの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Localeをサーバに保存する場合は、\ ``SessionLocaleResolver``\ を使用する。

**spring-mvc.xml**

.. code-block:: xml

  <bean id="localeResolver" class="org.springframework.web.servlet.i18n.SessionLocaleResolver">  <!-- (1) -->
      <property name="defaultLocale" value="en"/>  <!-- (2) -->
  </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (1)
      - | beanタグのid属性を"localeResolver"で定義し、 ``org.springframework.web.servlet.LocaleResolver`` を実装したクラスを指定する。
        | 本例では、セッションにLocaleを保存する ``org.springframework.web.servlet.i18n.SessionLocaleResolver`` を指定している。
        | beanタグのid属性は"localeResolver"と設定すること。
        | この設定により、 ``LocaleChangeInterceptor`` 内の処理で\ ``SessionLocaleResolver``\ が使用される。
    * - | (2)
      - | \ ``defaultLocale``\ プロパティにLocaleを指定する。セッションからLocaleが取得できない場合、\ ``value``\ の設定値が有効になる。

        .. note::

         \ ``defaultLocale``\ プロパティを省略した場合、ユーザ端末（ブラウザ）に設定されたLocaleが有効になる。

|

CookieLocaleResolverの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Localeをクライアントに保存する場合は、\ ``CookieLocaleResolver``\ を使用する。

**spring-mvc.xml**

.. code-block:: xml

  <bean id="localeResolver" class="org.springframework.web.servlet.i18n.CookieLocaleResolver">  <!-- (1) -->
        <property name="defaultLocale" value="en"/>  <!-- (2) -->
        <property name="cookieName" value="localeCookie"/>  <!-- (3) -->
  </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (1)
      - | beanタグのid属性"localeResolver"に ``org.springframework.web.servlet.i18n.CookieLocaleResolver`` を指定する。
        | beanタグのid属性は"localeResolver"と設定すること。
        | この設定により、 ``LocaleChangeInterceptor`` 内の処理で\ ``CookieLocaleResolver``\ が使用される。
    * - | (2)
      - | \ ``defaultLocale``\ プロパティにLocaleを指定する。CookieからLocaleが取得できない場合、\ ``value``\ の設定値が有効になる。

        .. note::

         \ ``defaultLocale``\ プロパティを省略した場合、ユーザ端末（ブラウザ）に設定されたLocaleが有効になる。

    * - | (3)
      - | \ ``cookieName``\ プロパティに指定した値が、cookie名となる。指定しない場合、\ ``org.springframework.web.servlet.i18n.CookieLocaleResolver.LOCALE``\ となる。**Spring Frameworkを使用していることがわかるため、変更することを推奨する。**

|

メッセージの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

以下に、メッセージの設定例を示す。

**application-messages.properties**

.. code-block:: properties

    i.xx.yy.0001 = changed locale
    i.xx.yy.0002 = Confirm change of locale at next screen

**application-messages_ja.properties**

.. code-block:: properties

    i.xx.yy.0001 = Localeを変更しました。
    i.xx.yy.0002 = 次の画面でのLocale変更を確認

|

.. _i18n_set_locale_jsp:

JSPの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

以下に、JSPの実装例を示す。

**画面表示用JSPファイル**

.. code-block:: jsp

    <a href='${pageContext.request.contextPath}?locale=en'>English</a>  <!-- (1) -->
    <a href='${pageContext.request.contextPath}?locale=ja'>Japanese</a>
    <spring:message code="i.xx.yy.0001" />

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - | 項番
      - | 説明
    * - | (1)
      - | Localeを切り替えるためのパラメータを送信する。
        | リクエストパラメータ名は、\ ``LocaleChangeInterceptor``\の\ ``paramName``\ プロパティに指定した値となる。（上記例では、デフォルトのパラメータ名を使用している）
        | 上記例の場合、Englishリンクで英語Locale、Japaneseリンクで日本語Localeに変更している。
        | 以降は、選択したLocaleが有効になる。
        | 英語Localeは"en"用のプロパティファイルが存在しないため、デフォルトのプロパティファイルから読み込まれる。

.. tip::

    * インクルード用の共通jspにSpringのタグライブラリを定義する必要がある。
    * インクルード用の共通jspファイルの詳細は :ref:`view_jsp_include-label` を参照されたい。

.. raw:: latex

   \newpage

