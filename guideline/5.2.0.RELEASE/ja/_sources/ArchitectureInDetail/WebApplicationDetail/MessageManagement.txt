メッセージ管理
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 4
    :local:

Overview
--------------------------------------------------------------------------------

| メッセージとは、画面や帳票等に表示する固定文言、またはユーザの画面操作の結果に応じて表示する動的文言を指す。
| また、エラーメッセージは、できるだけ細かく定義することを推奨する。

\

    .. warning::
       以下の場合において、運用中、あるいは運用前の試験の際、エラーの原因を究明できなくなるリスクが生じる。(開発中は、特に困らないかもしれない。)

       * エラーメッセージを、1つのみ定義している
       * エラーメッセージを、「重要」と「警告」の2つしか定義していない

       その結果、開発メンバが少ない中で、メッセージの定義変更を行い、開発が進むにつれて、修正コストが増えることになる。
       そのため、あらかじめメッセージは、細かい粒度で定義しておくことを推奨する。

メッセージタイプ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| ユーザの画面操作の結果に応じて表示するメッセージは、内容に応じて、以下3種類のメッセージタイプに分けて管理する。
| メッセージを定義する際は、出力するメッセージが、どのタイプに属するか意識すること。

.. _message-level-table-label:

.. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 20 60

   * - メッセージタイプ
     - カテゴリ
     - 概要
   * - info
     - 情報メッセージ
     - ユーザの操作による処理が正常に実行された後、画面に表示するメッセージ。
   * - warn
     - 警告メッセージ
     - 処理は継続できるが、注意喚起が必要な状態である場合に表示するメッセージ。（例：パスワード有効期限切れが近い場合の通知メッセージ）
   * - error
     - 入力エラーメッセージ
     - ユーザの入力値が不正な場合に、入力画面に表示するメッセージ。
   * -
     - 業務エラーメッセージ
     - 業務ロジックでエラーと判定された場合に表示するメッセージ
   * -
     - システムエラーメッセージ
     - システム起因のエラー（データベースとの接続失敗等）が発生し、ユーザの操作でリカバリできない場合に表示するメッセージ

|

パターン別メッセージタイプの分類
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

メッセージの出力パターンを、以下に示す。

.. figure:: ./images/message-pattern.png
   :alt: message pattern
   :width: 95%

メッセージパターンとメッセージの表示内容、及びメッセージタイプを、以下に示す。

.. tabularcolumns:: |p{0.05\linewidth}|p{0.15\linewidth}|p{0.20\linewidth}|p{0.10\linewidth}|p{0.50\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 5 15 20 10 50

   * - 記号
     - パターン
     - 表示内容
     - メッセージタイプ
     - 例
   * - | (A)
     - | タイトル
     - | 画面のタイトル
     - | -
     - * 従業員登録画面
   * - |
     - | ラベル
     - | 画面の項目名
       | 帳票の項目名
       | コメント
       | ガイダンス
     - | -
     - * ユーザー名
       * パスワード
   * - | (B)
     - | ダイアログ
     - | 確認メッセージ
     - | info
     - * 登録してよろしいでしょうか？
       * 削除してよろしいでしょうか？
   * - | (C)
     - | 結果メッセージ
     - | 正常終了
     - | info
     - * 登録しました。
       * 削除しました。
   * - | (D)
     - |
     - | 警告
     - | warn
     - * パスワードの有効期限切れが間近です。パスワードを変更して下さい。
       * サーバが混み合っています。時間をおいてから再度実行して下さい。
   * - | (E)
     - |
     - | 単項目チェックエラー
     - | error
     - * "ユーザー名"は必須です。
       * "名前"は20桁以内で入力してください。
       * "金額"には数字を入力してください。
   * - | (F)
     - |
     - | 相関チェックエラー
     - | error
     - * "パスワード"と"パスワード(確認用)"が一致しません。
   * - | (G)
     - |
     - | 業務エラー
     - | error
     - * キャンセル可能期間を過ぎているため、予約を取り消せません。
       * 登録可能件数を超えているため、登録できません。
   * - | (H)
     - |
     - | システムエラー
     - | error
     - * XXXシステム閉塞中のため、しばらく経ってから再度実行して下さい
       * タイムアウトが発生しました。
       * システムエラーが発生しました。

メッセージID体系
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| メッセージは、メッセージIDをつけて管理することを推奨する。
| 主な理由は、以下3つの利点を得るためである。

* メッセージ変更時に、ソースコードを修正することなくメッセージを変更するため
* メッセージの出力箇所を特定しやすくするため
* 国際化に対応できるため

メッセージIDの決め方は、メンテナンス性向上のため、規約を作って統一することを強く推奨する。

| メッセージパターン毎のメッセージID規約例を以下に示す。
| 開発プロジェクトでメッセージID規約が定まっていない場合は、参考にされたい。

タイトル
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 画面のタイトルに使用する、メッセージIDの決め方について説明する。


* フォーマット

    .. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 20 20 20 20 20

       * - 接頭句
         - 区切り
         - 業務名
         - 区切り
         - 画面名
       * - | title
         - | .
         - | nnn*
         - | .
         - | nnn*

* 記述内容

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.25\linewidth}|p{0.35\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 10 25 35

       * - 項目
         - 位置
         - 内容
         - 備考
       * - | 接頭句
         - | 1-5桁目 (5桁)
         - | "title" (固定)
         - |
       * - | 業務名
         - | 可変長：任意
         - | spring-mvc.xmlで定義したviewResolverのprefixの下のディレクトリ（JSPの上位ディレクトリ）
         - |
       * - | 画面名
         - | 可変長：任意
         - | JSP名
         - | ファイル名が"aaa.jsp"の場合"aaa"の部分

* 定義例

    .. code-block:: properties

        # "/WEB-INF/views/admin/top.jsp"の場合
        title.admin.top=Admin Top
        # "/WEB-INF/views/staff/createForm.jsp"の場合
        title.staff.createForm=Staff Register Input

    .. tip::

       本例は、Tilesを利用する場合に有効である。詳細は :doc:`../WebApplicationDetail/TilesLayout` を参照されたい。
       Tilesを利用しない場合は、次に説明する\ :ref:`message-management_label-rule`\ の規約を利用しても良い。

|

.. _message-management_label-rule:

ラベル
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

画面のラベル、帳票の固定文言に使用する、メッセージIDの決め方について説明する。


* フォーマット

    .. tabularcolumns:: |p{0.14\linewidth}|p{0.14\linewidth}|p{0.16\linewidth}|p{0.14\linewidth}|p{0.14\linewidth}|p{0.14\linewidth}|p{0.14\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 14 14 16 14 14 14 14

       * - 接頭句
         - 区切り
         - プロジェクト区分
         - 区切り
         - 業務名
         - 区切り
         - 項目名
       * - | label
         - | .
         - | xx
         - | .
         - | nnn*
         - | .
         - | nnn*


* 記述内容

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.25\linewidth}|p{0.35\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 10 25 35

       * - 項目
         - 位置
         - 内容
         - 備考
       * - | 接頭句
         - | 1-5桁目 (5桁)
         - | "label" (固定)
         - |
       * - | プロジェクト区分
         - | 7-8桁名 (2桁)
         - | プロジェクト名のアルファベット2桁表記
         - |
       * - | 業務名
         - | 可変長：任意
         - |
         - |
       * - | 項目名
         - | 可変長：任意
         - | ラベル名、説明文名
         - |


    .. note::

        入力チェックエラーのメッセージに項目名(論理名)を含める場合は、

        * フォームのモデル名 + "." + フィールド名

         .. code-block:: properties

            staffForm.staffName = Staff name

        * フィールド名

         .. code-block:: properties

            staffName = Staff name

        にする必要がある。



* 使用例

    .. code-block:: properties

        # スタッフ登録画面のフォームの項目名
        # プロジェクト区分=em (Event Management System)
        label.em.staff.staffName=Staff name
        # ツアー検索画面に表示する説明文の場合
        # プロジェクト区分=tr (Tour Reservation System)
        label.tr.tourSearch.tourSearchMessage=You can search tours with the specified conditions.

    .. note::

        プロジェクトが複数存在する場合に、メッセージIDが重複しないようにプロジェクト区分を定義する。
        単一プロジェクトの場合でも、将来を見据えてプロジェクト区分を定義することを推奨する。

.. _message-management_result-rule:

結果メッセージ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

業務間で共通して使用するメッセージ
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

同一メッセージを定義しないように、複数の業務間で共有するメッセージについて説明する。

* フォーマット

    .. tabularcolumns:: |p{0.12\linewidth}|p{0.12\linewidth}|p{0.14\linewidth}|p{0.12\linewidth}|p{0.14\linewidth}|p{0.12\linewidth}|p{0.12\linewidth}|p{0.12\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 12 12 14 12 14 12 12 12

       * - メッセージタイプ
         - 区切り
         - プロジェクト区分
         - 区切り
         - 共通メッセージ区分
         - 区切り
         - エラーレベル
         - 連番
       * - | x
         - | .
         - | xx
         - | .
         - | fw
         - | .
         - | 9
         - | 999

* 記述内容

    .. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.40\linewidth}|p{0.10\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 20 20 40 10

       * - 項目
         - 位置
         - 内容
         - 備考
       * - | メッセージタイプ
         - | 1桁目 (1桁)
         - | info  : i
           | warn  : w
           | error : e
         - |
       * - | プロジェクト区分
         - | 3-4桁目 (2桁)
         - | プロジェクト名のアルファベット2桁表記
         - |
       * - | 共通メッセージ区分
         - | 6-7桁目 (2桁)
         - | "fw" (固定)
         - |
       * - | エラーレベル
         - | 9桁 (1桁)
         - | 0-1 : 正常メッセージ
           | 2-4 : 業務エラー（準正常）
           | 5-7 : 入力チェックエラー
           | 8 : 業務エラー（エラー）
           | 9 : システムエラー
         - |
       * - | 連番
         - | 10-12桁目 (3桁)
         - | 連番で利用する(000-999)
         - | メッセージ削除となっても連番は空き番として、削除しない

* 使用例

    .. code-block:: properties

        # 登録が成功した場合（正常メッセージ）
        i.ex.fw.0001=Registered successfully.
        # サーバリソース不足
        w.ex.fw.9002=Server busy. Please, try again.
        # システムエラー発生の場合（システムエラー）
        e.ex.fw.9001=A system error has occurred.

.. _message-properties-example:

各業務で個別に使用するメッセージ
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

業務で個別に使用するメッセージについて説明する。

* フォーマット

    .. tabularcolumns:: |p{0.12\linewidth}|p{0.12\linewidth}|p{0.14\linewidth}|p{0.12\linewidth}|p{0.14\linewidth}|p{0.12\linewidth}|p{0.12\linewidth}|p{0.12\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 12 12 14 12 14 12 12 12

       * - メッセージタイプ
         - 区切り
         - プロジェクト区分
         - 区切り
         - 業務メッセージ区分
         - 区切り
         - エラーレベル
         - 連番
       * - | x
         - | .
         - | xx
         - | .
         - | xx
         - | .
         - | 9
         - | 999

* 記述内容

    .. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.40\linewidth}|p{0.10\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 20 20 40 10

       * - 項目
         - 位置
         - 内容
         - 備考
       * - | メッセージタイプ
         - | 1桁目 (1桁)
         - | info  : i
           | warn  : w
           | error : e
         - |
       * - | プロジェクト区分
         - | 3-4桁目 (2桁)
         - | プロジェクト名のアルファベット2桁表記
         - |
       * - | 業務メッセージ区分
         - | 6-7桁目 (2桁)
         - | 業務IDなど業務毎に決める2桁の文字
         - |
       * - | エラーレベル
         - | 9桁 (1桁)
         - | 0-1 : 正常メッセージ
           | 2-4 : 業務エラー（準正常）
           | 5-7 : 入力チェックエラー
           | 8 : 業務エラー（エラー）
           | 9 : システムエラー
         - |
       * - | 連番
         - | 10-12桁目 (3桁)
         - | 連番で利用する(000-999)
         - | メッセージ削除となっても連番は空き番として、削除しない


* 使用例

    .. code-block:: properties

        # ファイルのアップロードが成功した場合
        i.ex.an.0001={0} upload completed.
        # パスワードの推奨変更期間が過ぎている場合
        w.ex.an.2001=The recommended change interval of password has passed. Please change your password.
        # ファイルサイズが制限を超えている場合
        e.ex.an.8001=Cannot upload, Because the file size must be less than {0}MB.
        # データに不整合がある場合
        e.ex.an.9001=There are inconsistencies in the data.

|

入力チェックエラーメッセージ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

入力チェックでエラーがある場合に出力するメッセージについては、\ :ref:`Validation_message_def`\ を参照されたい。


    .. note::

        入力チェックエラーの出力場所に関する基本方針を、以下に示す。

        * | 単項目入力チェックエラーのメッセージは、対象の項目がわかるように項目の横に表示させる。
        * | 相関入力チェックエラーのメッセージは、ページ上部などにまとめて表示させる。
        * | 単項目チェックでもメッセージを項目の横に表示させにくい場合は、ページ上部に表示させる。
          | その場合は、メッセージに項目名を含める。

|

How to use
--------------------------------------------------------------------------------

プロパティファイルに設定したメッセージの表示
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

プロパティを使用する際の設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
メッセージ管理を行う\ ``org.springframework.context.MessageSource``\ の実装クラスの定義を行う。

* applicationContext.xml

    .. code-block:: xml

        <!-- Message -->
        <bean id="messageSource"
            class="org.springframework.context.support.ResourceBundleMessageSource"> <!-- (1) -->
            <property name="basenames"> <!-- (2) -->
                <list>
                    <value>i18n/application-messages</value>
                </list>
            </property>
        </bean>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | ``MessageSource``\ の定義。ここでは\ ``ResourceBundleMessageSource``\ を使用する。
       * - | (2)
         - | 使用するメッセージプロパティの基底名を定義する。クラスパス相対で指定する。
           | この例では"src/main/resources/i18n/application-messages.properties"を読み込む。

.. _properties-display:

プロパティに設定したメッセージの表示
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* application-messages.properties

    ここでは、\ :file:`application-messages.properties`\ にメッセージを定義する例を示す。

    .. code-block:: properties

        label.aa.bb.year=Year
        label.aa.bb.month=Month
        label.aa.bb.day=Day


    .. note::

        文字コード「ISO-8859-1」では表現できない文字(日本語など)は\ ``native2ascii``\ コマンドで
        ISO-8859-1に変換して使用することが多かった。しかし、JDK 6からは文字コードを指定できるようになったため、
        変換する必要はない。文字コードUTF-8にすることで、propertiesファイルに直接日本語等を使用できる。

        * application-messages.properties

            .. code-block:: properties

                label.aa.bb.year=年
                label.aa.bb.month=月
                label.aa.bb.day=日

        この場合、以下のように、\ ``ResourceBundleMessageSource``\ にも読み込む文字コードを指定する必要がある。

        * applicationContext.xml

            .. code-block:: java
                :emphasize-lines: 8

                <bean id="messageSource"
                    class="org.springframework.context.support.ResourceBundleMessageSource">
                    <property name="basenames">
                        <list>
                            <value>i18n/application-messages</value>
                        </list>
                    </property>
                    <property name="defaultEncoding" value="UTF-8" />
                </bean>

        デフォルトではISO-8859-1が使用されるため、日本語等をpropertiesファイルに直接記述したい場合は、
        必ず\ ``defaultEncoding``\ を設定すること。

* JSP

    上記で設定したメッセージをJSPからは、\ ``<spring:message>``\ タグを用いて表示できる。
    \ :ref:`view_jsp_include-label`\ の設定が必要である。

    .. code-block:: jsp

        <spring:message code="label.aa.bb.year" />
        <spring:message code="label.aa.bb.month" />
        <spring:message code="label.aa.bb.day" />

    フォームのラベルと使用する場合は、以下のように使用すれば良い。

    .. code-block:: jsp
        :emphasize-lines: 3,7,11

        <form:form modelAttribute="sampleForm">
            <form:label path="year">
                <spring:message code="label.aa.bb.year" />
            </form:label>: <form:input path="year" />
            <br>
            <form:label path="month">
                <spring:message code="label.aa.bb.month" />
            </form:label>: <form:input path="month" />
            <br>
            <form:label path="day">
                <spring:message code="label.aa.bb.day" />
            </form:label>: <form:input path="day" />
        </form:form>


    ブラウザで表示すると以下のように出力される。

    .. figure:: ./images_MessageManagement/message-management-ymd.png
        :width: 40%

    .. tip::

        国際化に対応する場合は、

        .. code-block:: text

            src/main/resources/i18n
                                ├ application-messages.properties (英語メッセージ)
                                ├ application-messages_fr.properties (フランス語メッセージ)
                                ├ ...
                                └ application-messages_ja.properties (日本語メッセージ)

        というように各言語用のpropertiesファイルを作成すればよい。
        詳細は、\ :doc:`../WebApplicationDetail/Internationalization`\ を参照されたい。


.. _message-display:

結果メッセージの表示
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| サーバサイドでの処理の成功や、失敗を示す結果メッセージを格納するクラスとして、
| 共通ライブラリでは、\ ``org.terasoluna.gfw.common.message.ResultMessages``\ 、および\ ``org.terasoluna.gfw.common.message.ResultMessage``\ を提供している。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 20 80

  * - クラス名
    - 説明
  * - | ``ResultMessages``
    - | 結果メッセージの一覧とメッセージタイプを持つクラス。
      | 結果メッセージの一覧は\ ``List<ResultMessage>``\ 、メッセージタイプは\ ``org.terasoluna.gfw.common.message.ResultMessageType``\ インタフェースで表現される。
  * - | ``ResultMessage``
    - | 結果メッセージのメッセージID、または、メッセージ本文を持つクラス。

| この結果メッセージをJSPで表示するためのJSPタグライブラリとして、\ ``<t:messagesPanel>``\ タグも提供される。

基本的な結果メッセージの使用方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Controllerで\ ``ResultMessages``\ を生成して画面に渡し、JSPで\ ``<t:messagesPanel>``\ タグを使用して、
結果メッセージを表示する方法を説明する。

* Controllerクラス

    ``ResultMessages``\ オブジェクトの生成方法、および画面へメッセージを渡す方法を示す。
    application-messages.proertiesには、\ :ref:`message-properties-example`\ の例が定義されていることとする。

    .. code-block:: java

        package com.example.sample.app.message;

        import org.springframework.stereotype.Controller;
        import org.springframework.ui.Model;
        import org.springframework.web.bind.annotation.RequestMapping;
        import org.springframework.web.bind.annotation.RequestMethod;
        import org.terasoluna.gfw.common.message.ResultMessages;

        @Controller
        @RequestMapping("message")
        public class MessageController {

          @RequestMapping(method = RequestMethod.GET)
          public String hello(Model model) {
            ResultMessages messages = ResultMessages.error().add("e.ex.an.9001"); // (1)
            model.addAttribute(messages); // (2)
            return "message/index";
          }
        }


    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - 項番
        - 説明
      * - | (1)
        - | メッセージタイプが"error"である\ ``ResultMessages``\ を作成し、
          | メッセージIDが"e.ex.an.9001"である結果メッセージを設定する。
          | この処理は次と同義である。
          | ``ResultMessages.error().add(ResultMessage.fromCode("e.ex.an.9001"));``
          | メッセージIDを指定する場合は、\ ``ResultMessage``\ オブジェクトの生成を省略できるため、省略することを推奨する。
      * - | (2)
        - | \ ``ResultMessages``\ をModelに追加する。
          | 属性は指定しなくてよい。(属性名は"resultMessages"になる)



* JSP

    WEB-INF/views/message/index.jspを、以下のように記述する。

    .. code-block:: jsp

        <!DOCTYPE HTML>
        <html>
        <head>
        <meta charset="utf-8">
        <title>Result Message Example</title>
        </head>
        <body>
            <h1>Result Message</h1>
            <t:messagesPanel /><!-- (1) -->
        </body>
        </html>


    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - 項番
        - 説明
      * - | (1)
        - | ``<t:messagesPanel>`` タグをデフォルト設定で使用する。
          | デフォルトでは、属性名が"resultMessages"のオブジェクトを表示する。
          | そのため、デフォルトではControllerからModelに\ ``ResultMessages``\ を設定する際に、属性名を設定する必要がない。

    ブラウザで表示すると、以下のように出力される。


    .. figure:: ./images_MessageManagement/message-management-resultmessage-basic.png
        :width: 40%


    \ ``<t:messagesPanel>`` によって出力されるHTMLを、以下に示す(説明しやすくするために整形している)。

    .. code-block:: html

        <div class="alert alert-error"><!-- (1) -->
          <ul><!-- (2) -->
            <li>There are inconsistencies in the data.</li><!-- (3) -->
          </ul>
        </div>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - 項番
        - 説明
      * - | (1)
        - | メッセージタイプに対応して"alert-error"クラスが付与されている。デフォルトでは\ ``<div>``\ タグのclassに"error error-[メッセージタイプ]"が付与される。
      * - | (2)
        - | 結果メッセージのリストが\ ``<ul>``\ タグで出力される。
      * - | (3)
        - | メッセージIDに対応するメッセージが\ ``MessageSource``\ から解決される。


    ``<t:messagesPanel>``\ はclassを付けたHTMLを出力するだけであるため、見栄えは出力されたclassに合わせてCSSでカスタマイズする必要がある(後述する)。

    .. note::

        \ ``ResultMessages.error().add(ResultMessage.fromText("There are inconsistencies in the data."));``\ というように、
        メッセージの本文をハードコードすることもできるが、保守性を高めるため、メッセージキーを使用して\ ``ResultMessage``\ オブジェクトを作成し、
        メッセージ本文はプロパティファイルから取得することを推奨する。

|

メッセージのプレースホルダに値を埋める場合は、次のように\ ``add``\ メソッドの第二引数以降に設定すればよい。

.. code-block:: java

    ResultMessages messages = ResultMessages.error().add("e.ex.an.8001", 1024);
    model.addAttribute(messages);

この場合、\ ``<t:messagesPanel />``\ タグにより、以下のようなHTMLが出力される。

.. code-block:: html

    <div class="alert alert-error">
      <ul>
        <li>Cannot upload, Because the file size must be less than 1,024MB.</li>
      </ul>
    </div>

\

 .. warning:: **terasoluna-gfw-web 1.0.0.RELEASEを使用してプレースホルダに値を埋める場合の注意点**

    terasoluna-gfw-web 1.0.0.RELEASEを使用している場合、\ **プレースホルダにユーザの入力値を埋め込むとXSS脆弱性の危険がある。**\
    ユーザの入力値にXSS対策が必要な文字が含まれる可能性がある場合は、プレースホルダに値を埋め込まないようにすること。
    
    terasoluna-gfw-web 1.0.1.RELEASE以上を使用している場合は、ユーザの入力値をプレースホルダに埋め込んでもXSS脆弱性は発生しない。

 .. note::

    \ ``ResourceBundleMessageSource``\ はメッセージを生成する際に\ ``java.text.MessageFormat``\ が使用するため、\ ``1024``\ は
    カンマ区切りで\ ``1,024``\ と表示される。カンマが不要な場合は、プロパティファイルには以下のように設定する。

        .. code-block:: properties

            e.ex.an.8001=Cannot upload, Because the file size must be less than {0,number,#}MB.

    詳細は、\ `Javadoc <http://docs.oracle.com/javase/8/docs/api/java/text/MessageFormat.html>`_\ を参照されたい。

|

以下のように、複数の結果メッセージを設定することもできる。

.. code-block:: java

    ResultMessages messages = ResultMessages.error()
        .add("e.ex.an.9001")
        .add("e.ex.an.8001", 1024);
    model.addAttribute(messages);

この場合は、次のようなHTMLが出力される(JSPの変更は、不要である)。

.. code-block:: html

    <div class="alert alert-error">
      <ul>
        <li>There are inconsistencies in the data.</li>
        <li>Cannot upload, Because the file size must be less than 1,024MB.</li>
      </ul>
    </div>

infoメッセージを表示したい場合は、次のように\ ``ResultMessages.info()``\ メソッドで\ ``ResultMessages``\ オブジェクトを作成すればよい。

.. code-block:: java

    ResultMessages messages = ResultMessages.info().add("i.ex.an.0001", "XXXX");
    model.addAttribute(messages);

以下のようなHTMLが、出力される。

.. code-block:: html

  <div class="alert alert-info"><!-- (1) -->
    <ul>
      <li>XXXX upload completed.</li>
    </ul>
  </div>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | メッセージタイプに対応して、出力されるclass名が"alert alert-**info**"に変わっている。

標準では、以下のメッセージタイプが用意されている。


.. tabularcolumns:: |p{0.15\linewidth}|p{0.30\linewidth}|p{0.25\linewidth}|p{0.30\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 15 30 25 30

  * - メッセージタイプ
    - \ ``ResultMessages``\ オブジェクトの作成
    - デフォルトで出力されるclass名
    - 備考
  * - | success
    - | ``ResultMessages.success()``\
    - | alert alert-success
    - | \-
  * - | info
    - | \ ``ResultMessages.info()``\
    - | alert alert-info
    - | \-
  * - | warn
    - | \ ``ResultMessages.warn()``\
    - | alert alert-warn
    - | メッセージタイプ「warning」の追加に伴い、terasoluna-gfw-common 5.0.0.RELEASEから非推奨。
      | \ **このメッセージタイプは将来削除される可能性がある。**\
  * - | warning
    - | \ ``ResultMessages.warning()``\
    - | alert alert-warning
    - | CSSフレームワークである\ `Bootstrap <http://getbootstrap.com/>`_ の\ `Alertsコンポーネント <http://getbootstrap.com/components/#alerts>`_\ で用意されているメッセージタイプをデフォルトでサポートするために、terasoluna-gfw-common 5.0.0.RELEASEから追加。
  * - | error
    - | \ ``ResultMessages.error()``\
    - | alert alert-error
    - | \-
  * - | danger
    - | \ ``ResultMessages.danger()``\
    - | alert alert-danger
    - | \-

メッセージタイプに応じてCSSを定義されたい。以下に、CSSを適用した場合の例を示す。

.. code-block:: css

    .alert {
      margin-bottom: 15px;
      padding: 10px;
      border: 1px solid;
      border-radius: 4px;
      text-shadow: 0 1px 0 #ffffff;
    }
    .alert-info {
      background: #ebf7fd;
      color: #2d7091;
      border-color: rgba(45, 112, 145, 0.3);
    }
    .alert-warning {
      background: #fffceb;
      color: #e28327;
      border-color: rgba(226, 131, 39, 0.3);
    }
    .alert-error {
      background: #fff1f0;
      color: #d85030;
      border-color: rgba(216, 80, 48, 0.3);
    }

* \ ``ResultMessages.error().add("e.ex.an.9001")``\ を\ ``<t:messagesPanel />``\ で出力した例


    .. figure:: ./images_MessageManagement/message-management-resultmessage-error.jpg
        :width: 100%


* \ ``ResultMessages.warning().add("w.ex.an.2001")``\ を\ ``<t:messagesPanel />``\ で出力した例


    .. figure:: ./images_MessageManagement/message-management-resultmessage-warn.jpg
        :width: 100%


* \ ``ResultMessages.info().add("i.ex.an.0001", "XXXX")``\ を\ ``<t:messagesPanel />``\ で出力した例


    .. figure:: ./images_MessageManagement/message-management-resultmessage-info.jpg
        :width: 100%

    .. note::

        successとdangerは、スタイルに多様性を持たせるために用意されている。本ガイドラインでは、successとinfo、errorとdangerは同義である。

    .. tip::

        CSSフレームワークである\ `Bootstrap <http://getbootstrap.com/>`_ 3.0.0の\ `Alertsコンポーネント <http://getbootstrap.com/components/#alerts>`_\ は、\ ``<t:messagesPanel />``\ のデフォルト設定で利用できる。

    .. warning::

        本例では、メッセージキーをハードコードで設定している。しかしながら、保守性を高めるためにも、メッセージキーは、定数クラスにまとめることを推奨する。

        :ref:`message-management-messagekeysgen`\ を参照されたい。

結果メッセージの属性名指定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| \ ``ResultMessages``\ をModelに追加する場合、基本的には属性名を省略できる。
| ただし、\ ``ResultMessages``\ は一つのメッセージタイプしか表現できない。
| 1画面に異なるメッセージタイプの\ ``ResultMessages``\ を\ **同時に**\ 表示したい場合は、明示的に属性名を指定してModelに設定する必要がある。

* Controller (MessageControllerに追加)

    .. code-block:: java

        @RequestMapping(value = "showMessages", method = RequestMethod.GET)
        public String showMessages(Model model) {

            model.addAttribute("messages1",
                        ResultMessages.warning().add("w.ex.an.2001")); // (1)
            model.addAttribute("messages2",
                        ResultMessages.error().add("e.ex.an.9001")); // (2)

            return "message/showMessages";
        }



    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - 項番
        - 説明
      * - | (1)
        - | メッセージタイプが"warning"である、\ ``ResultMessages``\ を属性名"messages1"でModelに追加する。
      * - | (2)
        - | メッセージタイプが"info"である、\ ``ResultMessages``\ を属性名"messages2"でModelに追加する。


* JSP (WEB-INF/views/message/showMessages.jsp)

    .. code-block:: jsp

        <!DOCTYPE HTML>
        <html>
        <head>
        <meta charset="utf-8">
        <title>Result Message Example</title>
        <style type="text/css">
        .alert {
            margin-bottom: 15px;
            padding: 10px;
            border: 1px solid;
            border-radius: 4px;
            text-shadow: 0 1px 0 #ffffff;
        }

        .alert-info {
            background: #ebf7fd;
            color: #2d7091;
            border-color: rgba(45, 112, 145, 0.3);
        }

        .alert-warning {
            background: #fffceb;
            color: #e28327;
            border-color: rgba(226, 131, 39, 0.3);
        }

        .alert-error {
            background: #fff1f0;
            color: #d85030;
            border-color: rgba(216, 80, 48, 0.3);
        }
        </style>
        </head>
        <body>
            <h1>Result Message</h1>
            <h2>Messages1</h2>
            <t:messagesPanel messagesAttributeName="messages1" /><!-- (1) -->
            <h2>Messages2</h2>
            <t:messagesPanel messagesAttributeName="messages2" /><!-- (2) -->
        </body>
        </html>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - 項番
        - 説明
      * - | (1)
        - | 属性名が"messages1"である\ ``ResultMessages``\ を表示する。
      * - | (2)
        - | 属性名が"messages2"である\ ``ResultMessages``\ を表示する。

    ブラウザで表示すると、以下のように出力される。

    .. figure:: ./images_MessageManagement/message-management-multiple-messages.jpg
        :width: 80%

業務例外メッセージの表示
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| \ ``org.terasoluna.gfw.common.exception.BusinessException``\ と\ ``org.terasoluna.gfw.common.exception.ResourceNotFoundException``\ は
| 内部で\ ``ResultMessages``\ を保持している。

| 業務例外メッセージを表示する場合は、Serviceクラスで\ ``ResultMessages``\ を設定した\ ``BusinessException``\ をスローすること。
| Controllerクラスでは\ ``BusinessException``\ をキャッチし、例外中の結果メッセージをModelに追加する。

* Serviceクラス

    .. code-block:: java

        @Service
        @Transactional
        public class UserServiceImpl implements UserService {
            // omitted

            public void create(...) {

                // omitted...

                if (...) {
                    // illegal state!
                    ResultMessages messages = ResultMessages.error()
                                                            .add("e.ex.an.9001"); // (1)
                    throw new BusinessException(messages);
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
        - | エラーメッセージを\ ``ResultMessages``\ で作成し、\ ``BusinessException``\ に設定する。

* Controllerクラス

    .. code-block:: java

        @RequestMapping(value = "create", method = RequestMethod.POST)
        public String create(@Validated UserForm form, BindingResult result, Model model) {
            // omitted

            try {
                userService.create(user);
            } catch (BusinessException e) {
                ResultMessages messages = e.getResultMessages(); // (1)
                model.addAttribute(messages);

                return "user/createForm";
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
        - | \ ``BusinessException``\ が保持する\ ``ResultMessages``\ を取得し、Modelに追加する。


通常、エラーメッセージ表示する場合は、Controllerで\ ``ResultMessages``\ オブジェクトを作成するのではなく、
こちらの方法を使用する。

|

How to extend
--------------------------------------------------------------------------------

独自メッセージタイプを作成する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| メッセージタイプを追加したい場合の、独自メッセージタイプ作成方法について説明する。
| 通常は、用意されているメッセージタイプのみで十分であるが、採用しているCSSライブラリによっては
| メッセージタイプを追加したい場合がある。例えば"notice"というメッセージタイプを追加する場合を説明する。


| まず、以下のように\ ``org.terasoluna.gfw.common.message.ResultMessageType``\ インタフェースを実装した
| 独自メッセージタイプクラスを作成する。

.. code-block:: java

    import org.terasoluna.gfw.common.message.ResultMessageType;

    public enum ResultMessageTypes implements ResultMessageType { // (1)
        NOTICE("notice");

        private ResultMessageTypes(String type) {
            this.type = type;
        }

        private final String type;

        @Override
        public String getType() { // (2)
            return this.type;
        }

        @Override
        public String toString() {
            return this.type;
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | \ ``ResultMessageType``\ インタフェースを実装したEnumを定義する。定数オブジェクトで作成してもよいが、Enumで作成することを推奨する。
  * - | (2)
    - | \ ``getType``\ の返り値が出力されるCSSのclass名に対応する。

| このメッセージタイプを使用して以下のように\ ``ResultMessages``\ を作成する。

.. code-block:: java

    ResultMessages messages = new ResultMessages(ResultMessageTypes.NOTICE) // (1)
            .add("w.ex.an.2001");
    model.addAttribute(messages);

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | \ ``ResultMessages``\ のコンストラクタに対象の\ ``ResultMessageType``\ を指定する。

この場合、\ ``<t:messagesPanel />`` \ で以下のようなHTMLが出力される。

.. code-block:: html

    <div class="alert alert-notice">
      <ul>
        <li>The recommended change interval has passed password. Please change your password.</li>
      </ul>
    </div>

\

    .. tip::

        拡張方法は、\ ``org.terasoluna.gfw.common.message.StandardResultMessageType``\ が参考になる。

|

Appendix
--------------------------------------------------------------------------------

.. _message-management-messagepanel-attribute:

<t:messagesPanel>タグの属性変更
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ ``<t:messagesPanel>``\ タグには、表示形式を変更する属性がいくつか用意されている。

.. tabularcolumns:: |p{0.25\linewidth}|p{0.55\linewidth}|p{0.20\linewidth}|
.. list-table:: \ ``<t:messagesPanel>``\ タグ 属性一覧
   :header-rows: 1
   :widths: 25 55 20

   * - オプション
     - 内容
     - defaultの設定値
   * - panelElement
     - 結果メッセージ表示パネルの要素
     - div
   * - panelClassName
     - 結果メッセージ表示パネルのCSS class名。
     - alert
   * - panelTypeClassPrefix
     - CSS class名の接頭辞
     - alert-
   * - messagesType
     - メッセージタイプ。この属性が設定された場合。設定されたメッセージタイプが\ ``ResultMessages``\ がもつメッセージタイプより優先されて使用される。
     -
   * - outerElement
     - 結果メッセージ一覧を構成するHTMLの外側のタグ
     - ul
   * - innerElement
     - 結果メッセージ一覧を構成するHTMLの内側のタグ
     - li
   * - disableHtmlEscape
     - | HTMLエスケープ処理を無効化するためのフラグ。
       | \ ``true``\ を指定する事で、出力するメッセージに対してHTMLエスケープ処理が行われなくなる。
       | この属性は、出力するメッセージにHTMLを埋め込むことで、メッセージの装飾などができるようにするために用意している。
       | **trueを指定する場合は、XSS対策が必要な文字がメッセージ内に含まれない事が保証されていること。**
       |
       | terasoluna-gfw-web 1.0.1.RELEASE以上で利用可能な属性である。
     - ``false``


例えば、CSSフレームワーク"\ `BlueTrip <http://www.bluetrip.org/>`_\ "では以下のようなCSSが用意されている。

.. code-block:: css

    .error,.notice,.success {
        padding: .8em;
        margin-bottom: 1.6em;
        border: 2px solid #ddd;
    }

    .error {
        background: #FBE3E4;
        color: #8a1f11;
        border-color: #FBC2C4;
    }

    .notice {
        background: #FFF6BF;
        color: #514721;
        border-color: #FFD324;
    }

    .success {
        background: #E6EFC2;
        color: #264409;
        border-color: #C6D880;
    }

| このCSSを使用したい場合、\ ``<div class="error">...</div>``\ というようにメッセージが出力されてほしい。
| この場合、\ ``<t:messagesPanel>``\ タグを以下のように使用すればよい(Controllerは修正不要である)。

.. code-block:: jsp

    <t:messagesPanel panelClassName="" panelTypeClassPrefix="" />

出力されるHTMLは以下のようになる。

.. code-block:: html

    <div class="error">
      <ul>
        <li>There are inconsistencies in the data.</li>
      </ul>
    </div>

ブラウザで表示すると、以下のように出力される。

.. figure:: ./images_MessageManagement/message-management-bluetrip-error.jpg
    :width: 80%

メッセージ一覧を表示するために\ ``<ul>``\ タグを使用したくない場合は、
\ ``outerElement``\ 属性と\ ``innerElement``\ 属性を使用することでカスタマイズできる。

以下のように属性を設定した場合は、

.. code-block:: jsp

    <t:messagesPanel outerElement="" innerElement="span" />


次のようにHTMLが出力される。


.. code-block:: html

    <div class="alert alert-error">
        <span>There are inconsistencies in the data.</span>
        <span>Cannot upload, Because the file size must be less than 1,024MB.</span>
    </div>

以下のようCSSを設定することで、

.. code-block:: css

    .alert > span {
        display: block; /* (1) */
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | "alert"クラスの要素の子となる\ ``<span>``\ タグをブロックレベル要素にする。

ブラウザで次のように表示される。


.. figure:: ./images_MessageManagement/message-management-messagespanel-span.jpg
    :width: 60%


| disableHtmlEscape属性を\ ``true``\にした場合、以下のような出力イメージにする事ができる。
| 下記の例では、メッセージの一部のフォントを「16pxの赤字」に装飾している。 

- jsp

 .. code-block:: jsp
    :emphasize-lines: 4

    <spring:message var="informationMessage" code="i.ex.od.0001" />
    <t:messagesPanel messagesAttributeName="informationMessage"
        messagesType="alert alert-info"
        disableHtmlEscape="true" />

- properties

 .. code-block:: properties

    i.ex.od.0001 = Please confirm order content. <font style="color: red; font-size: 16px;">If this orders submitted, cannot cancel.</font>

- 出力イメージ

 .. figure:: ./images_MessageManagement/message-management-disableHtmlEscape-true.png
    :width: 100%
    
 disableHtmlEscape属性が\ ``false``\(デフォルト)の場合は、HTMLエスケープされて以下のような出力となる。

 .. figure:: ./images_MessageManagement/message-management-disableHtmlEscape-false.png
    :width: 100%


ResultMessagesを使用しない結果メッセージの表示
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ ``<t:messagesPanel>``\ タグは\ ``ResultMessages``\ オブジェクト以外にも

* ``java.lang.String``
* ``java.lang.Exception``
* ``java.util.List``

オブジェクトも出力できる。

| 通常は\ ``<t:messagesPanel>``\ タグは\ ``ResultMessages``\ オブジェクトの出力用に使用するが、
| フレームワークがリクエストスコープに設定した文字列(エラーメッセージなど)を表示する場合にも使用できる。

| 例えば、Spring Securityは認証エラー時に、"SPRING_SECURITY_LAST_EXCEPTION"という属性名で発生した例外クラスを
| リクエストスコープに設定する。

| この例外メッセージを、結果メッセージ同様に\ ``<t:messagesPanel>``\ タグで出力したい場合は、以下のように設定すればよい。


.. code-block:: jsp

    <!DOCTYPE HTML>
    <html>
    <head>
    <meta charset="utf-8">
    <title>Login</title>
    <style type="text/css">
    /* (1) */
    .alert {
        margin-bottom: 15px;
        padding: 10px;
        border: 1px solid;
        border-radius: 4px;
        text-shadow: 0 1px 0 #ffffff;
    }

    .alert-error {
        background: #fff1f0;
        color: #d85030;
        border-color: rgba(216, 80, 48, 0.3);
    }
    </style>
    </head>
    <body>
        <c:if test="${param.containsKey('error')}">
            <t:messagesPanel messagesType="error"
                messagesAttributeName="SPRING_SECURITY_LAST_EXCEPTION" /><!-- (2) -->
        </c:if>
        <form:form
            action="${pageContext.request.contextPath}/authentication"
            method="post">
            <fieldset>
                <legend>Login Form</legend>
                <div>
                    <label for="username">Username: </label><input
                        type="text" id="username" name="username">
                </div>
                <div>
                    <label for="username">Password:</label><input
                        type="password" id="password" name="password">
                </div>
                <div>
                    <input type="submit" value="Login" />
                </div>
            </fieldset>
        </form:form>
    </body>
    </html>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | 結果メッセージ表示用のCSSを再掲する。実際はCSSファイルに記述することを強く推奨する。
  * - | (1)
    - | ``Exception``\ オブジェクトが格納されている属性名を\ ``messagesAttributeName``\ 属性で指定する。
      | また、\ ``ResultMessages``\ オブジェクトとは異なり、メッセージタイプの情報をもたないため、
      | \ ``messagesType``\ 属性で、明示的に、メッセージタイプを指定する必要がある。

認証エラー時に出力されるHTMLは

.. code-block:: html

    <div class="alert alert-error"><ul><li>Bad credentials</li></ul></div>

であり、ブラウザでは以下のように出力される。

.. figure:: ./images_MessageManagement/message-management-login-error.jpg
    :width: 60%

\

    .. tip::

        ログイン用のJSPの内容については、\ :doc:`../../Security/Authentication`\ を参照されたい。

.. _message-management-messagekeysgen:

メッセージキー定数クラスの自動生成ツール
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| これまでの例ではメッセージキーを文字列のハードコードで設定していたが、
| メッセージキーは定数クラスにまとめることを推奨する。

| ここでは、簡易ツールとして、propertiesファイルからメッセージキー定数クラスを
| 自動生成するプログラムおよび使用方法を紹介する。必要に応じてカスタマイズして利用されたい。

#. メッセージキー定数クラスの作成

    まず空のメッセージキー定数クラスを作成する。ここでは\ ``com.example.common.message.MessageKeys``\ とする。

    .. code-block:: java


        package com.example.common.message;

        public class MessageKeys {

        }

#. 自動生成クラスの作成

    次に\ ``MessageKeys``\ クラスと同じパッケージに\ ``MessageKeysGen``\ クラスを作成し、以下のように記述する。

    .. code-block:: java

        package com.example.common.message;

        import java.io.BufferedReader;
        import java.io.File;
        import java.io.FileInputStream;
        import java.io.IOException;
        import java.io.InputStream;
        import java.io.InputStreamReader;
        import java.io.PrintWriter;
        import java.util.regex.Pattern;

        import org.apache.commons.io.FileUtils;
        import org.apache.commons.io.IOUtils;

        public class MessageKeysGen {
            public static void main(String[] args) throws IOException {
                // message properties file
                InputStream inputStream = new FileInputStream("src/main/resources/i18n/application-messages.properties");
                BufferedReader br = new BufferedReader(new InputStreamReader(inputStream));
                Class<?> targetClazz = MessageKeys.class;
                File output = new File("src/main/java/"
                        + targetClazz.getName().replaceAll(Pattern.quote("."), "/")
                        + ".java");
                System.out.println("write " + output.getAbsolutePath());
                PrintWriter pw = new PrintWriter(FileUtils.openOutputStream(output));

                try {
                    pw.println("package " + targetClazz.getPackage().getName() + ";");
                    pw.println("/**");
                    pw.println(" * Message Id");
                    pw.println(" */");
                    pw.println("public class " + targetClazz.getSimpleName() + " {");

                    String line;
                    while ((line = br.readLine()) != null) {
                        String[] vals = line.split("=", 2);
                        if (vals.length > 1) {
                            String key = vals[0].trim();
                            String value = vals[1].trim();
                            pw.println("    /** " + key + "=" + value + " */");
                            pw.println("    public static final String "
                                    + key.toUpperCase().replaceAll(Pattern.quote("."),
                                            "_").replaceAll(Pattern.quote("-"), "_")
                                    + " = \"" + key + "\";");
                        }
                    }
                    pw.println("}");
                    pw.flush();
                } finally {
                    IOUtils.closeQuietly(br);
                    IOUtils.closeQuietly(pw);
                }
            }
        }

#. メッセージプロパティファイルの用意

    src/main/resource/i18m/application-messages.propertiesにメッセージを定義する。ここでは例として、以下のように設定する。


    .. code-block:: properties

        i.ex.an.0001={0} upload completed.
        w.ex.an.2001=The recommended change interval has passed password. Please change your password.
        e.ex.an.8001=Cannot upload, Because the file size must be less than {0}MB.
        e.ex.an.9001=There are inconsistencies in the data.

#. 自動生成クラスの実行


    .. figure:: ./images_MessageManagement/message-management-messagekeysgen.png
        :width: 60%

    ``MessageKeys``\ クラスが、以下のように上書きされる。


    .. code-block:: java

        package com.example.common.message;
        /**
         * Message Id
         */
        public class MessageKeys {
            /** i.ex.an.0001={0} upload completed. */
            public static final String I_EX_AN_0001 = "i.ex.an.0001";
            /** w.ex.an.2001=The recommended change interval has passed password. Please change your password. */
            public static final String W_EX_AN_2001 = "w.ex.an.2001";
            /** e.ex.an.8001=Cannot upload, Because the file size must be less than {0}MB. */
            public static final String E_EX_AN_8001 = "e.ex.an.8001";
            /** e.ex.an.9001=There are inconsistencies in the data. */
            public static final String E_EX_AN_9001 = "e.ex.an.9001";
        }

\

.. raw:: latex

   \newpage

