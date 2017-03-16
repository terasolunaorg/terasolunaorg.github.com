コードリスト
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 4
    :local:

Overview
--------------------------------------------------------------------------------

コードリストとは、「コード値(value)とその表示名(label)」の集合である。

画面のセレクトボックスなどコード値を画面で表示する際のラベルへのマッピング表として利用される。

共通ライブラリでは、

* xmlファイルやDBに定義されたコードリストをアプリケーション起動時に読み込みキャッシュする機能
* JSPやJavaクラスからコードリストを参照する機能
* コードリストを用いて入力チェックする機能

を提供している。

また、応用的な使い方として、

* コードリストの国際化対応
* キャッシュされたコードリストのリロード

もサポートしている。

.. note::

    標準でリロードが可能なのは、DBに定義されたコードリストを使用する場合のみである。

|

共通ライブラリでは、以下4種類のコードリスト実装を提供している。

.. _listOfCodeList:

.. tabularcolumns:: |p{0.50\linewidth}|p{0.30\linewidth}|p{0.20\linewidth}|
.. list-table:: **コードリスト種類一覧**
   :header-rows: 1
   :widths: 50 30 20

   * - 種類
     - 内容
     - Reloadable
   * - ``org.terasoluna.gfw.common.codelist.SimpleMapCodeList``
     - xmlファイルに直接記述した内容を使用する。
     - NO
   * - ``org.terasoluna.gfw.common.codelist.NumberRangeCodeList``
     - 数値の範囲のリストを作成する際に使用する。
     - NO
   * - ``org.terasoluna.gfw.common.codelist.JdbcCodeList``
     - DBから対象のコードをSQLで取得して使用する。
     - YES
   * - ``org.terasoluna.gfw.common.codelist.EnumCodeList``
     - \ ``Enum``\ クラスに定義した定数からコードリストを作成する際に使用する。
     - NO
   * - ``org.terasoluna.gfw.common.codelist.i18n.SimpleI18nCodeList``
     - java.util.Localeに応じたコードリストを使用する。
     - NO

上記コードリストのインターフェースについて、共通ライブラリに ``org.terasoluna.gfw.common.codelist.CodeList`` を提供している。

共通ライブラリで提供しているコードリストのクラス図構成を以下に示す。

.. figure:: ./images/codelist-class-diagram.png
   :alt: codelist class diagram
   :align: center

   **Picture - Image of codelist class diagram**

|

How to use
--------------------------------------------------------------------------------

本項では、各種コードリストを使用する上での設定や実装方法を記述する。

* :ref:`codelist-simple`
* :ref:`codelist-number`
* :ref:`codelist-jdbc`
* :ref:`codelist-enum`
* :ref:`codelisti18n`
* :ref:`codelist-display-label`
* :ref:`codelist-validate`

|

.. _codelist-simple:

SimpleMapCodeListの使用方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
``org.terasoluna.gfw.common.codelist.SimpleMapCodeList`` とは、
xmlファイルに定義したコード値をアプリケーション起動時に読み込み、そのまま使用するコードリストである。

**SimpleMapCodeListのイメージ**

.. figure:: ./images/codelist-simple.png
   :alt: codelist simple
   :width: 100%

|

コードリスト設定例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**bean定義ファイル(xxx-codelist.xml)の定義**

bean定義ファイルは、コードリスト用に作成することを推奨する。

.. code-block:: xml
   :emphasize-lines: 1,4

    <bean id="CL_ORDERSTATUS" class="org.terasoluna.gfw.common.codelist.SimpleMapCodeList"> <!-- (1) -->
        <property name="map">
            <util:map>
                <entry key="1" value="Received" /> <!-- (2) -->
                <entry key="2" value="Sent" />
                <entry key="3" value="Cancelled" />
            </util:map>
        </property>
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | SimpleMapCodeListクラスをbean定義する。
       | beanIDは、後述する ``org.terasoluna.gfw.web.codelist.CodeListInterceptor`` のIDパターンに合致する名称にすること。
   * - | (2)
     - | Mapの Key、Valueを定義する。
       | map-class属性を省略した場合、 ``java.util.LinkedHashMap`` で登録されるため、上記例では、「名前と値」が、登録順にMapへ保持される。

|

**bean定義ファイル(xxx-domain.xml)の定義**

コードリスト用bean定義ファイルを作成後、既存bean定義ファイルにimportを行う必要がある。

.. code-block:: xml
   :emphasize-lines: 1,4

    <import resource="classpath:META-INF/spring/projectName-codelist.xml" /> <!-- (3) -->
    <context:component-scan base-package="com.example.domain" />

    <!-- omitted -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (3)
     - | コードリスト用bean定義ファイルをimportする。
       | component-scanしている間にimport先の情報が必要な場合があるため、
       | importは ``<context:component-scan base-package="com.example.domain" />`` より上で設定する必要がある。

|

.. _clientSide:

JSPでのコードリスト使用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

共通ライブラリから提供しているインタセプターを用いることで、
リクエストスコープに自動的に設定し、JSPからコードリストを容易に参照できる。

**bean定義ファイル(spring-mvc.xml)の定義**

.. code-block:: xml
   :emphasize-lines: 3,5,6

    <mvc:interceptors>
      <mvc:interceptor>
        <mvc:mapping path="/**" /> <!-- (1) -->
        <bean
          class="org.terasoluna.gfw.web.codelist.CodeListInterceptor"> <!-- (2) -->
          <property name="codeListIdPattern" value="CL_.+" /> <!-- (3) -->
        </bean>
      </mvc:interceptor>

      <!-- omitted -->

    </mvc:interceptors>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 適用対象のパスを設定する。
   * - | (2)
     - | CodeListInterceptor クラスをbean定義する。
   * - | (3)
     - | 自動でリクエストスコープに設定するコードリストのbeanIDのパターンを設定する。
       | パターンには ``java.util.regex.Pattern`` で使用する正規表現を設定すること。
       | 上記例では、idが"CL\_XXX"形式で定義されているデータのみを対象とする。その場合、idが"CL\_"で始まらないbean定義は取り込まれない。
       | "CL\_"で定義したbeanIDは、リクエストスコープに設定されるため、JSPで使用可能となる。
       |
       | \ ``codeListIdPattern``\ プロパティは省略可能である。
       | \ ``codeListIdPattern``\ を省略した場合は、すべてのコードリスト(\ ``org.terasoluna.gfw.common.codelist.CodeList``\ インタフェースを実装しているbean)がJSPで使用可能となる。

|

**jspの実装例**

.. code-block:: jsp

  <form:select path="orderStatus">
    <form:option value="" label="--Select--" /> <!-- (4) -->
    <form:options items="${CL_ORDERSTATUS}" /> <!-- (5) -->
  </form:select>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (4)
     - | セレクトボックスの先頭にダミーの値を設定する場合、valueに空文字を指定すること。
   * - | (5)
     - | コードリストを定義したbeanIDを指定する。

**出力HTML**

.. code-block:: html

  <select id="orderStatus" name="orderStatus">
     <option value="">"--Select--</option>
     <option value="1">Received</option>
     <option value="2">Sent</option>
     <option value="3">Cancelled</option>
  </select>

**出力画面**

.. figure:: ./images/codelist_selectbox.png
   :alt: codelist selectbox
   :width: 40%

|

.. _serverSide:

Javaクラスでのコードリスト使用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Javaクラスでコードリストを利用する場合、 ``javax.inject.Inject`` アノテーションと、
``javax.inject.Named`` アノテーションを設定してコードリストをインジェクションする。
``@Named`` にコードリスト名を指定する。

.. code-block:: java

  import javax.inject.Named;

  import org.terasoluna.gfw.common.codelist.CodeList;

  public class OrderServiceImpl implements OrderService {

      @Inject
      @Named("CL_ORDERSTATUS")
      CodeList orderStatusCodeList; // (1)

      public boolean existOrderStatus(String target) {
          return orderStatusCodeList.asMap().containsKey(target); // (2)
      }
  }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | beanIDが、"CL_ORDERSTATUS"であるコードリストをインジェクションする。
   * - | (2)
     - | CodeList#asMapメソッドでコードリストを ``java.util.Map`` 形式で取得する。

|

.. _codelist-number:

NumberRangeCodeListの使用方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``org.terasoluna.gfw.common.codelist.NumberRangeCodeList`` とは、
アプリケーション起動時に、指定した数値の範囲をリストにするコードリストである。
主に数だけのセレクトボックス、月や日付などのセレクトボックスに使用することを想定している。

**NumberRangeCodeListのイメージ**

.. figure:: ./images/codelist-number.png
   :alt: codelist number
   :width: 100%

.. tip::

    NumberRangeCodeListはアラビア数字のみ対応しており、漢数字やローマ数字には対応していない。
    漢数字やローマ数字を表示したい場合はJdbcCodeList、SimpleMapCodeListに定義することで対応可能である。

|

NumberRangeCodeListには、以下の特徴がある。

#. Fromの値をToの値より小さくする場合、昇順にinterval分増加した値をFrom～Toの範囲分リストにする。
#. Toの値をFromの値より小さくする場合、降順にinterval分減少した値をTo～Fromの範囲分リストにする。
#. 増加分(減少分)はintervalを設定することで変更できる。

|

ここでは、昇順の\ ``NumberRangeCodeList``\ について説明をする。
降順の\ ``NumberRangeCodeList``\とインターバルの変更方法については、「:ref:`CodeListAppendixNumberRangeCodeListVariation`」を参照されたい。

|

コードリスト設定例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Fromの値をToの値より小さくする(From < To)場合の実装例を、以下に示す。

**bean定義ファイル(xxx-codelist.xml)の定義**

.. code-block:: xml

    <bean id="CL_MONTH"
        class="org.terasoluna.gfw.common.codelist.NumberRangeCodeList"> <!-- (1) -->
        <property name="from" value="1" /> <!-- (2) -->
        <property name="to" value="12" /> <!-- (3) -->
        <property name="valueFormat" value="%d" /> <!-- (4) -->
        <property name="labelFormat" value="%02d" /> <!-- (5) -->
        <property name="interval" value="1" /> <!-- (6) -->
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | NumberRangeCodeListをbean定義する。
   * - | (2)
     - | 範囲開始の値を指定する。省略した場合、"0"が設定される。
   * - | (3)
     - | 範囲終了の値を設定する。指定必須。
   * - | (4)
     - | コード値のフォーマット形式を設定する。フォーマット形式は ``java.lang.String.format`` の形式が使用される。
       | 省略した場合、"%s"が設定される。
   * - | (5)
     - | コード名のフォーマット形式を設定する。フォーマット形式は ``java.lang.String.format`` の形式が使用される。
       | 省略した場合、"%s"が設定される。
   * - | (6)
     - | 増加する値を設定する。省略した場合、"1"が設定される。

|

JSPでのコードリスト使用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

設定例の詳細は、前述した :ref:`JSPでのコードリスト使用<clientSide>` を参照されたい。

**jspの実装例**

.. code-block:: jsp

  <form:select path="depMonth" items="${CL_MONTH}" />

**出力HTML**

.. code-block:: html

  <select id="depMonth" name="depMonth">
    <option value="1">01</option>
    <option value="2">02</option>
    <option value="3">03</option>
    <option value="4">04</option>
    <option value="5">05</option>
    <option value="6">06</option>
    <option value="7">07</option>
    <option value="8">08</option>
    <option value="9">09</option>
    <option value="10">10</option>
    <option value="11">11</option>
    <option value="12">12</option>
  </select>

**出力画面**

.. figure:: ./images/codelist_numberrenge.png
   :alt: codelist numberrenge
   :width: 5%

|

Javaクラスでのコードリスト使用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

設定例の詳細は、前述した :ref:`Javaクラスでのコードリスト使用<serverSide>` を参照されたい。

|

.. _codelist-jdbc:

JdbcCodeListの使用方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| ``org.terasoluna.gfw.common.codelist.JdbcCodeList`` とは、アプリケーション起動時にDBから値を取得し、コードリストを作成するクラスである。
| ``JdbcCodeList`` はアプリケーション起動時にキャッシュを作るので、リスト表示時はDBアクセスによる遅延がない。

| 起動時の読み込み時間を抑えたいならば、取得数の上限を設定するとよい。
| ``JdbcCodeList`` には ``org.springframework.jdbc.core.JdbcTemplate`` を設定するフィールドがある。
| ``JdbcTemplate`` の ``fetchSize`` に上限を設定すれば、その分だけのレコードが起動時に読み込まれる。  
| なお、取得する値はリロードにより動的に変更できる。詳細は :ref:`codeListTaskScheduler` 参照されたい。

**JdbcCodeListのイメージ**

.. figure:: ./images/codelist-jdbc.png
   :alt: codelist simple
   :width: 100%

|

コードリスト設定例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**テーブル(authority)の定義**

.. tabularcolumns:: |p{0.40\linewidth}|p{0.60\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 40 60

   * - authority_id
     - authority_name
   * - | 01
     - | STAFF_MANAGEMENT
   * - | 02
     - | MASTER_MANAGEMENT
   * - | 03
     - | STOCK_MANAGEMENT
   * - | 04
     - | ORDER_MANAGEMENT
   * - | 05
     - | SHOW_SHOPPING_CENTER

|

**bean定義ファイル(xxx-codelist.xml)の定義**

.. code-block:: xml

    <bean id="jdbcTemplateForCodeList" class="org.springframework.jdbc.core.JdbcTemplate" > <!-- (1) -->
        <property name="dataSource" ref="dataSource" />
        <property name="fetchSize" value="${codelist.jdbc.fetchSize:1000}" /> <!-- (2) -->
    </bean>

    <bean id="AbstractJdbcCodeList"
        class="org.terasoluna.gfw.common.codelist.JdbcCodeList" abstract="true"> <!-- (3) -->
        <property name="jdbcTemplate" ref="jdbcTemplateForCodeList" /> <!-- (4) -->
    </bean>

    <bean id="CL_AUTHORITIES" parent="AbstractJdbcCodeList" > <!-- (5) -->
        <property name="querySql"
            value="SELECT authority_id, authority_name FROM authority ORDER BY authority_id" /> <!-- (6) -->
        <property name="valueColumn" value="authority_id" /> <!-- (7) -->
        <property name="labelColumn" value="authority_name" /> <!-- (8) -->
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90
   :class: longtable

   * - 項番
     - 説明
   * - | (1)
     - | ``org.springframework.jdbc.core.JdbcTemplate`` クラスをbean定義する。
       | 独自に ``fetchSize`` を設定するために必要となる。
   * - | (2)
     - | ``fetchSize`` を設定する。
       | ``fetchSize`` のデフォルト設定が、全件取得になっている場合があるため適切な値を設定すること。
       | ``fetchSize`` の設定が全件取得のままだと、 ``JdbcCodeList`` の読み込む件数が大きい場合に、DBからリストを取得する際の処理性能が落ちてしまい、アプリケーションの起動時間が長期化する可能性がある。
   * - | (3)
     - | ``JdbcCodeList`` の共通bean定義。
       | 他の ``JdbcCodeList`` の共通部分を設定している。そのため、基本 ``JdbcCodeList`` のbean定義はこのbean定義を親クラスに設定する。
       | abstract属性をtrueにすることで、このbeanはインスタンス化されない。
   * - | (4)
     - | (1)で設定した ``jdbcTemplate`` を設定。
       | ``fetchSize`` を設定した ``JdbcTemplate`` を、 ``JdbcCodeList`` に格納している。
   * - | (5)
     - | ``JdbcCodeList`` のbean定義。
       | parent属性を(3)のbean定義を親クラスとして設定することで、 ``fetchSize`` を設定した ``JdbcCodeList`` が設定される。
       | このbean定義では、クエリに関する設定のみを行い、必要なCodeList分作成する。
   * - | (6) 
     - | querySqlプロパティに取得するSQLを記述する。その際、 **必ず「ORDER BY」を指定し、順序を確定させること。**
       | 「ORDER BY」を指定しないと、取得する度に順序が変わってしまう。
   * - | (7)
     - | valueColumnプロパティに、MapのKeyに該当する値を設定する。この例ではauthority_idを設定している。
   * - | (8)
     - | labelColumnプロパティに、MapのValueに該当する値を設定する。この例ではauthority_nameを設定している。      

.. raw:: latex

   \newpage

|

JSPでのコードリスト使用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 下記に示す設定の詳細について、前述した :ref:`JSPでのコードリスト使用<clientSide>` を参照されたい。

**jspの実装例**

.. code-block:: jsp

  <form:checkboxes items="${CL_AUTHORITIES}"/>

**出力HTML**

.. code-block:: html

  <span>
    <input id="authorities1" name="authorities" type="checkbox" value="01"/>
    <label for="authorities1">STAFF_MANAGEMENT</label>
  </span>
  <span>
    <input id="authorities2" name="authorities" type="checkbox" value="02"/>
    <label for="authorities2">MASTER_MANAGEMENT</label>
  </span>
  <span>
    <input id="authorities3" name="authorities" type="checkbox" value="03"/>
    <label for="authorities3">STOCK_MANAGEMENT</label>
  </span>
  <span>
    <input id="authorities4" name="authorities" type="checkbox" value="04"/>
    <label for="authorities4">ORDER_MANAGEMENT</label>
  </span>
  <span>
    <input id="authorities5" name="authorities" type="checkbox" value="05"/>
    <label for="authorities5">SHOW_SHOPPING_CENTER</label>
  </span>

**出力画面**

.. figure:: ./images/codelist_checkbox.png
   :alt: codelist checkbox
   :width: 50%

|

Javaクラスでのコードリスト使用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

下記に示す設定の詳細について、前述した :ref:`Javaクラスでのコードリスト使用<serverSide>` を参照されたい。

|

.. _codelist-enum:

EnumCodeListの使用方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
\ ``org.terasoluna.gfw.common.codelist.EnumCodeList``\ は、
\ ``Enum``\ クラスに定義した定数からコードリストを作成するクラスである。

.. note::

    以下の条件に一致するアプリケーションでコードリストを扱う場合は、
    \ ``EnumCodeList``\ を使用して、コードリストのラベルを\ ``Enum``\ クラスで管理することを検討してほしい。
    コードリストのラベルを\ ``Enum``\ クラスで管理することで、
    コード値に紐づく情報と操作を\ ``Enum``\ クラスに集約する事ができる。

    * コード値を\ ``Enum``\ クラスで管理する必要がある(つまり、Javaのロジックでコード値を意識した処理を行う必要がある)
    * UIの国際化(多言語化)の必要がない

|

以下に、\ ``EnumCodeList``\ の使用イメージを示す。

.. figure:: ./images/codelist-enum.png
   :alt: codelist enum
   :width: 100%

.. note::

    \ ``EnumCodeList``\ では、\ ``Enum``\ クラスからコードリストを作成するために必要な情報(コード値とラベル)を取得するためのインタフェースとして、
    \ ``org.terasoluna.gfw.common.codelist.EnumCodeList.CodeListItem``\ インタフェースを提供している。

    \ ``EnumCodeList``\を使用する場合は、作成する\ ``Enum``\ クラスで\ ``EnumCodeList.CodeListItem``\ インタフェースを実装する必要がある。

|

コードリスト設定例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**Enumクラスの作成**

\ ``EnumCodeList``\ を使用する場合は、
\ ``EnumCodeList.CodeListItem``\ インタフェースを実装した\ ``Enum``\ クラスを作成する。
以下に作成例を示す。

.. code-block:: java

    package com.example.domain.model;

    import org.terasoluna.gfw.common.codelist.EnumCodeList;

    public enum OrderStatus
        // (1)
        implements EnumCodeList.CodeListItem {

        // (2)
        RECEIVED  ("1", "Received"),
        SENT      ("2", "Sent"),
        CANCELLED ("3","Cancelled");

        // (3)
        private final String value;
        private final String label;

        // (4)
        private OrderStatus(String codeValue, String codeLabel) {
            this.value = codeValue;
            this.label = codeLabel;
        }

        // (5)
        @Override
        public String getCodeValue() {
            return value;
        }

        // (6)
        @Override
        public String getCodeLabel() {
            return label;
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - コードリストとして使用する\ ``Enum``\ クラスでは、
        共通ライブラリから提供している\ ``org.terasoluna.gfw.common.codelist.EnumCodeList.CodeListItem``\ インタフェースを実装する。

        \ ``EnumCodeList.CodeListItem``\ インタフェースには、コードリストを作成するために必要な情報(コード値とラベル)を取得するためのメソッドとして、

        * コード値を取得する\ ``getCodeValue()``\ メソッド
        * ラベルを取得する\ ``getCodeLabel()``\ メソッド

        が定義されている。
    * - | (2)
      - 定数を定義する。

        定数を生成する際に、コードリストを作成するために必要な情報(コード値とラベル)を指定する。

        上記例では、以下の3つの定数を定義している。

        * \ ``RECEIVED``\ (コード値=\ ``"1"``\ , ラベル=\ ``"Received"``\ )
        * \ ``SENT``\  (コード値=\ ``"2"``\ , ラベル=\ ``"Sent"``\ )
        * \ ``CANCELLED``\  (コード値=\ ``"3"``\ , ラベル=\ ``"Cancelled"``\ )

        .. note::

            \ ``EnumCodeList``\ を使用した際のコードリストの並び順は、定数の定義順となる。

    * - | (3)
      - コードリストを作成するために必要な情報(コード値とラベル)を保持するプロパティを用意する。
    * - | (4)
      - コードリストを作成するために必要な情報(コード値とラベル)を受け取るコンストラクタを用意する。
    * - | (5)
      - 定数が保持するコード値を返却する。

        このメソッドは、\ ``EnumCodeList.CodeListItem``\ インタフェースで定義されているメソッドであり、
        \ ``EnumCodeList``\ が定数からコード値を取得する際に呼び出す。
    * - | (6)
      - 定数が保持するラベルを返却する。

        このメソッドは、\ ``EnumCodeList.CodeListItem``\ インタフェースで定義されているメソッドであり、
        \ ``EnumCodeList``\ が定数からラベルを取得する際に呼び出す。

|

**bean定義ファイル(xxx-codelist.xml)の定義**

コードリスト用のbean定義ファイルに、\ ``EnumCodeList``\を定義する。
以下に定義例を示す。

.. code-block:: xml

    <bean id="CL_ORDERSTATUS"
          class="org.terasoluna.gfw.common.codelist.EnumCodeList"> <!-- (7) -->
        <constructor-arg value="com.example.domain.model.OrderStatus" /> <!-- (8) -->
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (7)
      - コードリストの実装クラスとして、\ ``EnumCodeList``\ クラスを指定する。
    * - | (8)
      - \ ``EnumCodeList``\ クラスのコンストラクタに、\ ``EnumCodeList.CodeListItem``\ インタフェースを実装した\ ``Enum``\ クラスのFQCNを指定する。

|

JSPでのコードリスト使用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

JSPでコードリストを使用する方法については、前述した :ref:`clientSide` を参照されたい。

|

Javaクラスでのコードリスト使用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Javaクラスでコードリストを使用する方法については、
前述した :ref:`serverSide` を参照されたい。

|

.. _codelisti18n:

SimpleI18nCodeListの使用方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``org.terasoluna.gfw.common.codelist.i18n.SimpleI18nCodeList`` は、国際化に対応しているコードリストである。
ロケール毎にコードリストを設定することで、ロケールに対応したコードリストを返却できる。

**SimpleI18nCodeListのイメージ**

.. figure:: ./images/codelist-i18n.png
   :alt: codelist i18n
   :width: 100%

|

コードリスト設定例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

``SimpleI18nCodeList``\ は行が\ ``Locale``\ 、列がコード値、セルの内容がラベルである2次元のテーブルをイメージすると理解しやすい。

料金を選択するセレクトボックスの場合の例に上げると以下のようなテーブルができる。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.14\linewidth}|p{0.14\linewidth}|p{0.14\linewidth}|p{0.14\linewidth}|p{0.14\linewidth}|
.. list-table::
   :header-rows: 1
   :stub-columns: 1
   :widths: 10 15 15 15 15 15 15

   * - row=Locale,column=Code
     - 0
     - 10000
     - 20000
     - 30000
     - 40000
     - 50000
   * - en
     - unlimited
     - Less than \\10,000
     - Less than \\20,000
     - Less than \\30,000
     - Less than \\40,000
     - Less than \\50,000
   * - ja
     - 上限なし
     - 10,000円以下
     - 20,000円以下
     - 30,000円以下
     - 40,000円以下
     - 50,000円以下



この国際化対応コードリストのテーブルを構築するために\ ``SimpleI18nCodeList``\ は3つの設定方法を用意している。

* 行単位でLocale毎の\ ``CodeList``\ を設定する
* 行単位でLocale毎の\ ``java.util.Map``\ (key=コード値, value=ラベル)を設定する
* 列単位でコード値毎の\ ``java.util.Map``\ (key=Locale, value=ラベル)を設定する

基本的には、「行単位でLocale毎の\ ``CodeList``\ を設定する」方法でコードリストを設定することを推奨する。

上記例の料金を選択するセレクトボックスの場合を行単位でLocale毎の\ ``CodeList``\ を設定する方法について説明する。
他の設定方法については  :ref:`afterCodelisti18n` 参照されたい。

|

**Bean定義ファイル(xxx-codelist.xml)の定義**

.. code-block:: xml
  
    <bean id="CL_I18N_PRICE"
        class="org.terasoluna.gfw.common.codelist.i18n.SimpleI18nCodeList">
        <property name="rowsByCodeList"> <!-- (1) -->
            <util:map>
                <entry key="en" value-ref="CL_PRICE_EN" />
                <entry key="ja" value-ref="CL_PRICE_JA" />
            </util:map>
        </property>
    </bean>
  
.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
  
    * - 項番
      - 説明
    * - | (1)
      - | rowsByCodeListプロパティにkeyが\ ``java.lang.Locale``\ のMapを設定する。
        | Mapには、keyにロケール、value-refにロケールに対応したコードリストクラスの参照先を指定する。
        | Mapのvalueは各ロケールに対応したコードリストクラスを参照する。

|

**Locale毎にSimpleMapCodeListを用意する場合のBean定義ファイル(xxx-codelist.xml)の定義**

.. code-block:: xml
  
    <bean id="CL_I18N_PRICE"
        class="org.terasoluna.gfw.common.codelist.i18n.SimpleI18nCodeList">
        <property name="rowsByCodeList">
            <util:map>
                <entry key="en" value-ref="CL_PRICE_EN" />
                <entry key="ja" value-ref="CL_PRICE_JA" />
            </util:map>
        </property>
    </bean>
  
    <bean id="CL_PRICE_EN" class="org.terasoluna.gfw.common.codelist.SimpleMapCodeList">  <!-- (2) -->
        <property name="map">
            <util:map>
                <entry key="0" value="unlimited" />
                <entry key="10000" value="Less than \\10,000" />
                <entry key="20000" value="Less than \\20,000" />
                <entry key="30000" value="Less than \\30,000" />
                <entry key="40000" value="Less than \\40,000" />
                <entry key="50000" value="Less than \\50,000" />
            </util:map>
        </property>
    </bean>
  
    <bean id="CL_PRICE_JA" class="org.terasoluna.gfw.common.codelist.SimpleMapCodeList">  <!-- (3) -->
        <property name="map">
            <util:map>
                <entry key="0" value="上限なし" />
                <entry key="10000" value="10,000円以下" />
                <entry key="20000" value="20,000円以下" />
                <entry key="30000" value="30,000円以下" />
                <entry key="40000" value="40,000円以下" />
                <entry key="50000" value="50,000円以下" />
            </util:map>
        </property>
    </bean>
  
.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
  
    * - 項番
      - 説明
    * - | (2)
      - | ロケールが"en"であるbean定義 ``CL_PRICE_EN`` について、コードリストクラスを ``SimpleMapCodeList`` で設定している。
    * - | (3)
      - | ロケールが"ja"であるbean定義 ``CL_PRICE_JA`` について、コードリストクラスを ``SimpleMapCodeList`` で設定している。

|

**Locale毎にJdbcCodeListを用意する場合のBean定義ファイル(xxx-codelist.xml)の定義**

.. code-block:: xml
  
    <bean id="CL_I18N_PRICE"
        class="org.terasoluna.gfw.common.codelist.i18n.SimpleI18nCodeList">
        <property name="rowsByCodeList">
            <util:map>
                <entry key="en" value-ref="CL_PRICE_EN" />
                <entry key="ja" value-ref="CL_PRICE_JA" />
            </util:map>
        </property>
    </bean>
  
    <bean id="CL_PRICE_EN" parent="AbstractJdbcCodeList">  <!-- (4) -->
        <property name="querySql"
            value="SELECT code, label FROM price WHERE locale = 'en' ORDER BY code" />
        <property name="valueColumn" value="code" />
        <property name="labelColumn" value="label" />
    </bean>
  
    <bean id="CL_PRICE_JA" parent="AbstractJdbcCodeList">  <!-- (5) -->
        <property name="querySql"
            value="SELECT code, label FROM price WHERE locale = 'ja' ORDER BY code" />
        <property name="valueColumn" value="code" />
        <property name="labelColumn" value="label" />
    </bean>
  
.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
  
    * - 項番
      - 説明
    * - | (4)
      - | ロケールが"en"であるbean定義 ``CL_PRICE_EN`` について、コードリストクラスを ``JdbcCodeList`` で設定している。
    * - | (5)
      - | ロケールが"ja"であるbean定義 ``CL_PRICE_JA`` について、コードリストクラスを ``JdbcCodeList`` で設定している。
  

テーブル定義(priceテーブル)には以下のデータを投入する。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 20 60
    :class: longtable
  
    * - locale
      - code
      - label
    * - | en
      - | 0
      - | unlimited
    * - | en
      - | 10000
      - | Less than \\10,000
    * - | en
      - | 20000
      - | Less than \\20,000
    * - | en
      - | 30000
      - | Less than \\30,000
    * - | en
      - | 40000
      - | Less than \\40,000
    * - | en
      - | 50000
      - | Less than \\50,000
    * - | ja
      - | 0
      - | 上限なし
    * - | ja
      - | 10000
      - | 10,000円以下
    * - | ja
      - | 20000
      - | 20,000円以下
    * - | ja
      - | 30000
      - | 30,000円以下
    * - | ja
      - | 40000
      - | 40,000円以下
    * - | ja
      - | 50000
      - | 50,000円以下

.. raw:: latex

   \newpage

.. warning::

    現時点で ``SimpleI18nCodeList`` はreloadableに対応していない。
    ``SimpleI18nCodeList`` が参照している ``JdbcCodeList`` (reloadableなCodeList)をリロードしても、 ``SimpleI18nCodeList`` には反映されないことに注意。
    もし、reloadableに対応したい場合は独自実装する必要がある。
    実装方法については、 :ref:`originalCustomizeCodeList` を参照されたい。

|

JSPでのコードリスト使用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

基本的な設定は、前述した :ref:`JSPでのコードリスト使用<clientSide>` と同様のため、説明は省略する。

**bean定義ファイル(spring-mvc.xml)の定義**

.. code-block:: xml

    <mvc:interceptors>
      <mvc:interceptor>
        <mvc:mapping path="/**" />
        <bean
          class="org.terasoluna.gfw.web.codelist.CodeListInterceptor">
          <property name="codeListIdPattern" value="CL_.+" />
          <property name="fallbackTo" value="en" />  <!-- (1) -->
        </bean>
      </mvc:interceptor>

      <!-- omitted -->

    </mvc:interceptors>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | リクエストのロケールがコードリスト定義されていなかった場合、
       | fallbackToプロパティに設定されたロケールでコードリストを取得する。
       | fallbackToプロパティが設定されていない場合、JVMのデフォルトロケールがfallbackToプロパティとして使用される。
       | fallbackToプロパティに設定されたロケールでも、コードリストが取得されない場合、WARNログを出力し、空のMapを返却する。

|

**jspの実装例**

.. code-block:: jsp

  <form:select path="basePrice" items="${CL_I18N_PRICE}" />

**出力HTML lang=en**

.. code-block:: html

  <select id="basePrice" name="basePrice">
    <option value="0">unlimited</option>
    <option value="1">Less than \\10,000</option>
    <option value="2">Less than \\20,000</option>
    <option value="3">Less than \\30,000</option>
    <option value="4">Less than \\40,000</option>
    <option value="5">Less than \\50,000</option>
  </select>

**出力HTML lang=ja**

.. code-block:: html

  <select id="basePrice" name="basePrice">
    <option value="0">上限なし</option>
    <option value="1">10,000円以下</option>
    <option value="2">20,000円以下</option>
    <option value="3">30,000円以下</option>
    <option value="4">40,000円以下</option>
    <option value="5">50,000円以下</option>
  </select>

**出力画面 lang=en**

.. figure:: ./images/codelist_i18n_en.png
   :alt: codelist i18n en
   :width: 20%

**出力画面 lang=ja**

.. figure:: ./images/codelist_i18n_ja.png
   :alt: codelist i18n ja
   :width: 20%

|

Javaクラスでのコードリスト使用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

基本的な設定は、前述した :ref:`Javaクラスでのコードリスト使用<serverSide>` と同様のため、説明は省略する。

.. code-block:: java

    @RequestMapping("orders")
    @Controller
    public class OrderController {

        @Inject
        @Named("CL_I18N_PRICE")
        I18nCodeList priceCodeList;

        // ...

        @RequestMapping(method = RequestMethod.POST, params = "confirm")
        public String confirm(OrderForm form, Locale locale) {
            // ...
            String priceMassage = getPriceMessage(form.getPriceCode(), locale);
            // ...
        }

        private String getPriceMessage(String targetPrice, Locale locale) {
             return priceCodeList.asMap(locale).get(targetPrice);  // (1)
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | I18nCodeList#asMap(Locale)で対応したロケールのMapを取得することができる。

|

.. _codelist-display-label:

特定のコード値からコード名を表示する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

JSPからコードリストを参照する場合は、 ``java.util.Map`` インタフェースと同じ方法で参照することができる。

コードリストを用いて特定のコード値からコード名を表示する方法について、以下に実装例を示す。

**jspの実装例**

.. code-block:: jsp

    Order Status : ${f:h(CL_ORDERSTATUS[orderForm.orderStatus])}

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - コードリストを定義したbeanID(この例では ``"CL_ORDERSTATUS"`` ) を属性名として、コードリスト( ``java.util.Map`` インタフェース)を取得する。
       取得した ``Map`` インタフェースのキーとしてコード値(この例では ``orderStatus`` に格納された値) を指定することで、対応するコード名を表示することができる。


|

.. _codelist-validate:

コードリストを用いたコード値の入力チェック
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

入力値がコードリスト内に定義されたコード値であるかどうかチェックするような場合、
共通ライブラリでは、BeanValidation用のアノテーション、 ``org.terasoluna.gfw.common.codelist.ExistInCodeList`` を提供している。

BeanValidationや、メッセージ出力方法の詳細については、 :doc:`../WebApplicationDetail/Validation` を参照されたい。

\ ``@ExistInCodeList``\ アノテーションを使用して入力チェックを行う場合は、
\ ``@ExistInCodeList``\ 用の「:ref:`Validation_message_def`」を行う必要がある。

`ブランクプロジェクト <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank>`_ \ からプロジェクトを生成した場合は、
\ ``xxx-web/src/main/resources``\ の直下の\ ``ValidationMessages.properties``\ ファイルの中に以下のメッセージが定義されている。
メッセージは、アプリケーションの要件に合わせて変更すること。

.. code-block:: properties

    org.terasoluna.gfw.common.codelist.ExistInCodeList.message = Does not exist in {codeListId}

.. note::

    terasoluna-gfw-common 5.0.0.RELEASEより、
    メッセージのプロパティキーの形式を、Bean Validationのスタンダードな形式(アノテーションのFQCN + \ ``.message``\ )に変更している。

     .. tabularcolumns:: |p{0.40\linewidth}|p{0.60\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 40 60

        * - バージョン
          - メッセージのプロパティキー
        * - | version 5.0.0.RELEASE以降
          - | ``org.terasoluna.gfw.common.codelist.ExistInCodeList.message``
        * - | version 1.0.x.RELEASE
          - | ``org.terasoluna.gfw.common.codelist.ExistInCodeList``

    version 1.0.x.RELEASEからversion 5.0.0.RELEASE以降にバージョンアップする際に、
    アプリケーション要件に合わせてメッセージを変更している場合は、
    プロパティキーの変更が必要になる。

.. note::

    terasoluna-gfw-common 1.0.2.RELEASEより、
    \ ``@ExistInCodeList``\ のメッセージを定義した\ ``ValidationMessages.properties``\ を、
    jarファイルの中に含めないようにしている。
    これは、「`ValidationMessages.propertiesが複数存在する場合にメッセージが表示されないバグ <https://github.com/terasolunaorg/terasoluna-gfw/issues/256>`_」を修正するためである。

    version 1.0.1.RELEASE以前からversion 1.0.2.RELEASE以降にバージョンアップする際に、
    terasoluna-gfw-commonのjarの中に含まれる\ ``ValidationMessages.properties``\ に定義しているメッセージを使用している場合は、
    \ ``ValidationMessages.properties``\ を作成してメッセージを定義する必要がある。

|

@ExistInCodeList の設定例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

コードリストを用いた入力チェック方法について、以下に実装例を示す。

**bean定義ファイル(xxx-codelist.xml)の定義**

.. code-block:: xml

    <bean id="CL_GENDER" class="org.terasoluna.gfw.common.codelist.SimpleMapCodeList">
        <property name="map">
            <map>
                <entry key="M" value="Male" />
                <entry key="F" value="Female" />
            </map>
        </property>
    </bean>

**Formオブジェクト**

.. code-block:: java

    public class Person {
        @ExistInCodeList(codeListId = "CL_GENDER")  // (1)
        private String gender;

        // getter and setter omitted
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 入力チェックを行いたいフィールドに対して、 ``@ExistInCodeList`` アノテーションを設定し、
       | codeListIdにチェック元となる、コードリストを指定する。

上記の結果、 ``gender`` にM、F以外の文字が格納されている場合、エラーになる。

.. tip::

    ``@ExistInCodeList`` の入力チェックでサポートしている型は、 \ ``CharSequence``\ インタフェースの実装クラス(\ ``String``\ など) または \ ``Character``\ のみである。
    そのため、 \ ``@ExistInCodeList``\ をつけるフィールドは意味的に整数型であっても、\ ``String``\ で定義する必要がある。(年・月・日等)

|


How to extend
--------------------------------------------------------------------------------

.. _codeListTaskScheduler:

コードリストをリロードする場合
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

前述した共通ライブラリで提供しているコードリストは、アプリケーション起動時に読み込まれ、それ以降は、基本的に更新されない。
しかし、コードリストのマスタデータを更新した時、コードリストも更新したい場合がある。

例：JdbcCodeListを使用して、DBのマスタを変更した時にコードリストの更新を行う場合。

共通ライブラリでは、 ``org.terasoluna.gfw.common.codelist.ReloadableCodeList`` インタフェースを用意している。
上記インタフェースを実装したクラスは、refreshメソッドを実装しており、refreshメソッドを呼ぶことでコードリストの更新が可能となる。
JdbcCodeListは、ReloadableCodeListインターフェースを実装しているため、コードリストの更新ができる。

コードリストの更新方法としては、以下2点の方法がある。

#. Task Schedulerで実現する方法
#. Controller(Service)クラスでrefreshメソッドを呼び出す方法

本ガイドラインでは、\ `Springから提供されているTask Scheduler <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/scheduling.html>`_\ を使用して、コードリストを定期的にリロードする方式を基本的に推奨する。

ただし、任意のタイミングでコードリストをリフレッシュする必要がある場合はControllerクラスでrefreshメソッドを呼び出す方法で実現すればよい。

.. note::

    ReloadableCodeListインターフェースを実装しているコードリストについては、 :ref:`コードリスト種類一覧<listOfCodeList>` を参照されたい。

|

Task Schedulerで実現する方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Task Schedulerの設定例について、以下に示す。

**bean定義ファイル(xxx-codelist.xml)の定義**

.. code-block:: xml

    <task:scheduler id="taskScheduler" pool-size="10"/>  <!-- (1) -->

    <task:scheduled-tasks scheduler="taskScheduler">  <!-- (2) -->
        <task:scheduled ref="CL_AUTHORITIES" method="refresh" cron="${cron.codelist.refreshTime}"/>  <!-- (3) -->
    </task:scheduled-tasks>

    <bean id="CL_AUTHORITIES" parent="AbstractJdbcCodeList">
        <property name="querySql"
            value="SELECT authority_id, authority_name FROM authority ORDER BY authority_id" />
        <property name="valueColumn" value="authority_id" />
        <property name="labelColumn" value="authority_name" />
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | ``<task:scheduler>`` の要素を定義する。pool-size属性にスレッドのプールサイズを指定する。
       | pool-size属性を指定しない場合、"1" が設定される。
   * - | (2)
     - | ``<task:scheduled-tasks>`` の要素を定義し、scheduler属性に、 ``<task:scheduler>`` のIDを設定する。
   * - | (3)
     - | ``<task:scheduled>`` 要素を定義する。method属性に、refreshメソッドを指定する。
       | cron属性に、``org.springframework.scheduling.support.CronSequenceGenerator`` でサポートされた形式で記述すること。
       | cron属性は開発環境、商用環境など環境によってリロードするタイミングが変わることが想定されるため、プロパティファイルや、環境変数等から取得することを推奨する。
       |
       | **cron属性の設定例**
       | 「秒 分 時 月 年 曜日」で指定する。
       | 毎秒実行               「\* \* \* \* \* \*」
       | 毎時実行               「0 0 \* \* \* \*」
       | 平日の9-17時の毎時実行 「0 0 9-17 \* \* MON-FRI」
       |
       | 詳細はJavaDocを参照されたい。
       | http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/scheduling/support/CronSequenceGenerator.html

|

Controller(Service)クラスでrefreshメソッドを呼び出す方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

refreshメソッドを直接呼び出す場合について、
JdbcCodeListのrefreshメソッドをServiceクラスで呼び出す場合の実装例を、以下に示す。

**bean定義ファイル(xxx-codelist.xml)の定義**

.. code-block:: xml

    <bean id="CL_AUTHORITIES" parent="AbstractJdbcCodeList">
        <property name="querySql"
            value="SELECT authority_id, authority_name FROM authority ORDER BY authority_id" />
        <property name="valueColumn" value="authority_id" />
        <property name="labelColumn" value="authority_name" />
    </bean>

**Controllerクラス**

.. code-block:: java

    @Controller
    @RequestMapping(value = "codelist")
    public class CodeListContoller {

        @Inject
        CodeListService codeListService; // (1)

        @RequestMapping(method = RequestMethod.GET, params = "refresh")
        public String refreshJdbcCodeList() {
            codeListService.refresh(); // (2)
            return "codelist/jdbcCodeList";
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | ReloadableCodeListクラスのrefreshメソッドを実行するServiceクラスをインジェクションする。
   * - | (2)
     - | ReloadableCodeListクラスのrefreshメソッドを実行するServiceクラスのrefreshメソッドを実行する。

**Serviceクラス**

以下は実装クラスのみ記述し、インターフェースクラスは省略。

.. code-block:: java

    @Service
    public class CodeListServiceImpl implements CodeListService { // (3)

        @Inject
        @Named(value = "CL_AUTHORITIES") // (4)
        ReloadableCodeList codeListItem; // (5)

        @Override
        public void refresh() { // (6)
            codeListItem.refresh(); // (7)
        }
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (3)
     - | 実装クラス ``CodeListServiceImpl`` は、インターフェース ``CodeListService`` を実装する。
   * - | (4)
     - | コードリストをインジェクションするとき、 ``@Named`` で、該当するコードリストを指定する。
       | value属性に取得したいbeanのIDを指定すること。
       | Bean定義ファイルに定義されているbeanタグのID属性"CL_AUTHORITIES"のコードリストがインジェクションされる。
   * - | (5)
     - | フィールドの型にReloadableCodeListインターフェースを定義すること。
       | (4)で指定したBeanはReloadableCodeListインターフェースを実装していること。
   * - | (6)
     - | Serviceクラスで定義したrefreshメソッド。
       | Controllerクラスから呼び出されている。
   * - | (7)
     - | ReloadableCodeListインターフェースを実装したコードリストのrefreshメソッド。
       | refreshメソッドを実行することで、コードリストが更新される。

|

.. _originalCustomizeCodeList:

コードリストを独自カスタマイズする方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

共通ライブラリで提供している4種類のコードリストで実現できないコードリストを作成したい場合、コードリストを独自にカスタマイズすることができる。
独自カスタマイズする場合、作成できるコードリストの種類と実装方法について、以下の表に示す。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.30\linewidth}|p{0.45\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 15 30 45

   * - 項番
     - Reloadable
     - 継承するクラス
     - 実装箇所
   * - | (1)
     - | 不要
     - | ``org.terasoluna.gfw.common.codelist.AbstractCodeList``
     - | ``asMap`` をオーバライド
   * - | (2)
     - | 必要
     - | ``org.terasoluna.gfw.common.codelist.AbstractReloadableCodeList``
     - | ``retrieveMap`` をオーバライド

``org.terasoluna.gfw.common.codelist.CodeList`` 、 ``org.terasoluna.gfw.common.codelist.ReloadableCodeList`` インターフェースを直接実装しても実現はできるが、共通ライブラリで提供されている抽象クラスを拡張することで、最低限の実装で済む。

以下に、独自カスタマイズの実例について示す。
例として、今年と来年の年のリストを作るコードリストについて説明する。
(例：今年が2013の場合、コードリストには、"2013、2014"の順で格納される。)

**コードリストクラス**

.. code-block:: java

    @Component("CL_YEAR") // (1)
    public class DepYearCodeList extends AbstractCodeList { // (2)

        @Inject
        JodaTimeDateFactory dateFactory; // (3)

        @Override
        public Map<String, String> asMap() {  // (4)
            DateTime dateTime = dateFactory.newDateTime();
            DateTime nextYearDateTime = dateTime.plusYears(1);

            Map<String, String> depYearMap = new LinkedHashMap<String, String>();

            String thisYear = dateTime.toString("Y");
            String nextYear = nextYearDateTime.toString("Y");
            depYearMap.put(thisYear, thisYear);
            depYearMap.put(nextYear, nextYear);

            return Collections.unmodifiableMap(depYearMap);
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | ``@Component`` で、コードリストをコンポーネント登録する。
       | Valueに ``"CL_YEAR"`` を指定することで、bean定義で設定したコードリストインターセプトによりコードリストをコンポーネント登録する。
   * - | (2)
     - | ``org.terasoluna.gfw.common.codelist.AbstractCodeList`` を継承する。
       | 今年と来年の年のリストを作る時、動的にシステム日付から算出して作成しているため、リロードは不要。
   * - | (3)
     - | システム日付のDateクラスを作成する ``org.terasoluna.gfw.common.date.jodatime.JodaTimeDateFactory`` をインジェクトしている。
       | ``JodaTimeDateFactory`` を利用して今年と来年の年を取得することができる。
       | 事前に、bean定義ファイルにDataFactory実装クラスを設定する必要がある。
   * - | (4)
     - | ``asMap()`` メソッドをオーバライドして、今年と来年の年のリストを作成する。
       | 作成したいコードリスト毎に実装が異なる。

|

**jspの実装例**

.. code-block:: jsp

  <form:select path="mostRecentYear" items="${CL_YEAR}" /> <!-- (5) -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (5)
     - | items属性にコンポーネント登録した ``"CL_YEAR"`` を ``${}`` プレースホルダー で指定することで、該当のコードリストを取得することができる。

**出力HTML**

.. code-block:: html

  <select id="mostRecentYear" name="mostRecentYear">
     <option value="2013">2013</option>
     <option value="2014">2014</option>
  </select>

**出力画面**

.. figure:: ./images/codelist_customizeCodelist.png
   :alt: customized codelist
   :width: 10%

.. note::

    リロード可能であるCodeListを独自カスタマイズする場合、スレッドセーフになるように実装すること。

|

Appendix
--------------------------------------------------------------------------------

.. _afterCodelisti18n:

SimpleI18nCodeListのコードリスト設定方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

SimpleI18nCodeListのコードリスト設定について、 :ref:`codelisti18n` で設定されているコードリスト設定の他に2つ設定方法がある。
料金を選択するセレクトボックスの場合の例を用いて、それぞれの設定方法を説明する。

行単位でLocale毎の\ ``java.util.Map``\ (key=コード値, value=ラベル)を設定する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**bean定義ファイル(xxx-codelist.xml)の定義**

.. code-block:: xml

    <bean id="CL_I18N_PRICE"
        class="org.terasoluna.gfw.common.codelist.i18n.SimpleI18nCodeList">
        <property name="rows"> <!-- (1) -->
            <util:map>
                <entry key="en">
                    <util:map>
                        <entry key="0" value="unlimited" />
                        <entry key="10000" value="Less than \\10,000" />
                        <entry key="20000" value="Less than \\20,000" />
                        <entry key="30000" value="Less than \\30,000" />
                        <entry key="40000" value="Less than \\40,000" />
                        <entry key="50000" value="Less than \\50,000" />
                    </util:map>
                </entry>
                <entry key="ja">
                    <util:map>
                        <entry key="0" value="上限なし" />
                        <entry key="10000" value="10,000円以下" />
                        <entry key="20000" value="20,000円以下" />
                        <entry key="30000" value="30,000円以下" />
                        <entry key="40000" value="40,000円以下" />
                        <entry key="50000" value="50,000円以下" />
                    </util:map>
                </entry>
            </util:map>
        </property>
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | rowsプロパティに対して、"MapのMap"を設定する。外側のMapのkeyは\ ``java.lang.Locale``\ である。
       | 内側のMapのkeyはコード値、valueはロケールに対応したラベルである。

|

列単位でコード値毎の\ ``java.util.Map``\ (key=Locale, value=ラベル)を設定する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

**bean定義ファイル(xxx-codelist.xml)の定義**

.. code-block:: xml

    <bean id="CL_I18N_PRICE"
        class="org.terasoluna.gfw.common.codelist.i18n.SimpleI18nCodeList">
        <property name="columns"> <!-- (1) -->
            <util:map>
                <entry key="0">
                    <util:map>
                        <entry key="en" value="unlimited" />
                        <entry key="ja" value="上限なし" />
                    </util:map>
                </entry>
                <entry key="10000">
                    <util:map>
                        <entry key="en" value="Less than \\10,000" />
                        <entry key="ja" value="10,000円以下" />
                    </util:map>
                </entry>
                <entry key="20000">
                    <util:map>
                        <entry key="en" value="Less than \\20,000" />
                        <entry key="ja" value="20,000円以下" />
                    </util:map>
                </entry>
                <entry key="30000">
                    <util:map>
                        <entry key="en" value="Less than \\30,000" />
                        <entry key="ja" value="30,000円以下" />
                    </util:map>
                </entry>
                <entry key="40000">
                    <util:map>
                        <entry key="en" value="Less than \\40,000" />
                        <entry key="ja" value="40,000円以下" />
                    </util:map>
                </entry>
                <entry key="50000">
                    <util:map>
                        <entry key="en" value="Less than \\50,000" />
                        <entry key="ja" value="50,000円以下" />
                    </util:map>
                </entry>
            </util:map>
        </property>
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | columnsプロパティに対して、"MapのMap"を設定する。外側のMapのkeyはコード値である。
       | 内側のMapのkeyは\ ``java.lang.Locale``\、valueはロケールに対応したラベルである。

|

.. _CodeListAppendixNumberRangeCodeListVariation:

NumberRangeCodeListのバリエーション
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

降順のNumberRangeCodeListの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

次に、Toの値をFromの値より小さくする(To < From)場合の実装例を、以下に示す。

**bean定義ファイル(xxx-codelist.xml)の定義**

.. code-block:: xml

    <bean id="CL_BIRTH_YEAR"
        class="org.terasoluna.gfw.common.codelist.NumberRangeCodeList">
        <property name="from" value="2013" /> <!-- (1) -->
        <property name="to" value="2000" /> <!-- (2) -->
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 範囲開始の値を指定する。name属性"to"のvalue属性の値より大きい値を指定する。
        | この指定によって、interval分減少した値を、To～Fromの範囲分のリストとして、降順に表示する。
        | intervalは設定していないため、デフォルトの値1が適用される。
    * - | (2)
      - | 範囲終了の値を設定する。
        | 本例では、2000を指定することにより、リストには2013～2000までの範囲で1ずつ減少して格納される。

|

**jspの実装例**

.. code-block:: jsp

    <form:select path="birthYear" items="${CL_BIRTH_YEAR}" />

**出力HTML**

.. code-block:: html

  <select id="birthYear" name="birthYear">
    <option value="2013">2013</option>
    <option value="2012">2012</option>
    <option value="2011">2011</option>
    <option value="2010">2010</option>
    <option value="2009">2009</option>
    <option value="2008">2008</option>
    <option value="2007">2007</option>
    <option value="2006">2006</option>
    <option value="2005">2005</option>
    <option value="2004">2004</option>
    <option value="2003">2003</option>
    <option value="2002">2002</option>
    <option value="2001">2001</option>
    <option value="2000">2000</option>
  </select>

**出力画面**

.. figure:: ./images/codelist_numberrenge2.png
    :alt: codelist numberrenge2

|

NumberRangeCodeListのインターバルの変更
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

次に、interval値を設定する場合の実装例を、以下に示す。

**bean定義ファイル(xxx-codelist.xml)の定義**

.. code-block:: xml

    <bean id="CL_BULK_ORDER_QUANTITY_UNIT"
        class="org.terasoluna.gfw.common.codelist.NumberRangeCodeList">
        <property name="from" value="10" />
        <property name="to" value="50" />
        <property name="interval" value="10" /> <!-- (1) -->
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 増加(減少)値を指定する。この指定によって、interval値を増加(減少)した値を、From～Toの範囲内でコードリストとして格納する。
        | 上記の例だと、コードリストには\ ``10``\,\ ``20``\,\ ``30``\,\ ``40``\,\ ``50``\の順で格納される。

|

**jspの実装例**

.. code-block:: jsp

  <form:select path="quantity" items="${CL_BULK_ORDER_QUANTITY_UNIT}" />

**出力HTML**

.. code-block:: html

    <select id="quantity" name="quantity">
        <option value="10">10</option>
        <option value="20">20</option>
        <option value="30">30</option>
        <option value="40">40</option>
        <option value="50">50</option>
    </select>

**出力画面**

.. figure:: ./images/codelist_numberrenge3.png
    :alt: codelist numberrenge3

.. note::

    interval値分増加(減少)した値が、Form～Toの値が範囲を超えた場合は、コードリストに格納されない。

    具体的には、

     .. code-block:: xml

        <bean id="CL_BULK_ORDER_QUANTITY_UNIT"
            class="org.terasoluna.gfw.common.codelist.NumberRangeCodeList">
            <property name="from" value="10" />
            <property name="to" value="55" />
            <property name="interval" value="10" />
        </bean>

    という定義を行った場合、

    コードリストには\ ``10``\,\ ``20``\,\ ``30``\,\ ``40``\,\ ``50``\の計5つが格納される。
    次のintervalである\ ``60``\及び範囲の閾値である\ ``55``\はコードリストに格納されない。


.. raw:: latex

   \newpage

