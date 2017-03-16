システム時刻
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

Overview
--------------------------------------------------------------------------------

アプリケーション開発中は、サーバーのシステム時刻ではなく、任意の日時でテストを行う必要が生じる。
Production環境においても日付を戻してリカバリ処理を行うことも想定される。

そのため、日時の取得ではサーバーのシステム時刻ではなく、開発・運用側で任意の日時に設定可能になっていることが望ましい。

|

共通ライブラリから提供しているコンポーネントについて
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

共通ライブラリでは、システム時刻を取得するためのコンポーネント(以降では、これらのAPIを総称して「Date Factory」と呼ぶ)を提供している。

共通ライブラリから提供しているコンポーネントは、terasoluna-gfw-commonとterasoluna-gfw-jodatimeの二つのアーティファクトに分かれており、

* terasoluna-gfw-commonは、Java標準のAPIのみを利用するDate Factory
* terasoluna-gfw-jodatimeは、Joda TimeのAPIを利用するDate Factory

を提供している。

共通ライブラリから提供しているコンポーネントのクラス図を以下に示す。

.. figure:: ./images/systemdate-class-diagram.png
    :alt: Class Diagram of Date Factory
    :width: 100%

terasoluna-gfw-common
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

以下に、terasoluna-gfw-commonのコンポーネントとして提供しているインタフェースについて説明する。

.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 35 65

    * - インタフェース
      - 説明
    * - | org.terasoluna.gfw.common.date.
        | ClassicDateFactory
      - Javaから提供されている以下のクラスのインスタンスをシステム時刻として取得するためのインタフェース。

        * \ ``java.util.Date``\
        * \ ``java.sql.Timestamp``\
        * \ ``java.sql.Date``\
        * \ ``java.sql.Time``\

        共通ライブラリでは、本インタフェースの実装クラスとして以下のクラスを提供している。

        * \ ``org.terasoluna.gfw.common.date.DefaultClassicDateFactory``\


terasoluna-gfw-jodatime
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

以下に、terasoluna-gfw-jodatimeのコンポーネントとして提供しているインタフェースについて説明する。

.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 35 65

    * - インタフェース
      - 説明
    * - | org.terasoluna.gfw.common.date.jodatime.
        | JodaTimeDateTimeFactory
      - Joda Timeから提供されている以下のクラスのインスタンスをシステム時刻として取得するためのインタフェース。

        * \ ``org.joda.time.DateTime``\

    * - | org.terasoluna.gfw.common.date.jodatime.
        | JodaTimeDateFactory
      - \ ``ClassicDateFactory``\ と \ ``JodaTimeDateTimeFactory``\ を継承したインタフェース。

        共通ライブラリでは、本インタフェースの実装クラスとして以下のクラスを提供している。

        * \ ``org.terasoluna.gfw.common.date.jodatime.DefaultJodaTimeDateFactory``\
        * \ ``org.terasoluna.gfw.common.date.jodatime.JdbcFixedJodaTimeDateFactory``\
        * \ ``org.terasoluna.gfw.common.date.jodatime.JdbcAdjustedJodaTimeDateFactory``\

        **本ガイドラインでは、本インタフェースに対応する実装クラスを使用することを推奨する。**
    * - | org.terasoluna.gfw.common.date.
        | DateFactory
      - \ ``JodaTimeDateFactory``\ を継承したインタフェース(非推奨)。

        本インタフェースは、terasoluna-gfw-common 1.0.xで提供している\ ``DateFactory``\ との後方互換のために提供しているインタフェースである。

        共通ライブラリでは、本インタフェースの実装クラスとして以下のクラスを提供している。

        * \ ``org.terasoluna.gfw.common.date.DefaultDateFactory``\ (非推奨)
        * \ ``org.terasoluna.gfw.common.date.JdbcFixedDateFactory``\ (非推奨)
        * \ ``org.terasoluna.gfw.common.date.JdbcAdjustedDateFactory``\ (非推奨)

        **本インタフェース及び対応する実装クラスは非推奨のAPIであるため、新規に開発するアプリケーションで使用する事を禁止する。**

.. note::

    Joda Timeについては、 :doc:`./JodaTime` を参照されたい。

|

How to use
--------------------------------------------------------------------------------

Date Factoryインタフェースの実装クラスをbean定義ファイルに定義し、Date FactoryのインスタンスをJavaクラスにインジェクションして使用する。

実装クラスは使用用途に応じて、以下から選択する。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.30\linewidth}|p{0.40\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 30 30 40

   * - クラス名
     - 概要
     - 備考
   * - | org.terasoluna.gfw.common.date.jodatime.
       | DefaultJodaTimeDateFactory
     - アプリケーションサーバーのシステム時刻を返却する。
     - \ ``new DateTime()``\ での取得値と同等であり、時刻の変更はできない。
   * - | org.terasoluna.gfw.common.date.jodatime.
       | JdbcFixedJodaTimeDateFactory
     - DBに登録した固定の時刻を返却する。
     - 完全に時刻を固定する必要のあるIntegration Test環境で使用されることを想定しており、
       Performance Test環境や、Production環境では使用しない。

       このクラスを使用するためには、固定時刻を管理するためのテーブルが必要である。
   * - | org.terasoluna.gfw.common.date.jodatime.
       | JdbcAdjustedJodaTimeDateFactory
     - アプリケーションサーバーのシステム時刻にDBに登録した差分(ミリ秒)を加算した時刻を返却する。
     - Integration Test環境やSystem Test環境で使用されることを想定しているが、
       差分値を0に設定することでProduction環境でも使用できる。

       このクラスを使用するためには、差分値を管理するためのテーブルが必要である。

.. note::

    実装クラスを設定するbean定義ファイルは、環境ごとに切り替えられるように、[projectName]-env.xmlに定義することを推奨する。
    Date Factoryを利用することにより、bean定義ファイルの設定を変更するだけで、ソースを変更せずに日時の変更が可能となる。
    bean定義ファイルの記載例は後述する。

.. tip::

    JUnitなどで日時を変更して試験を行いたい場合、インタフェースの実装クラスをmockクラスに差し替えることで、
    任意の日時を設定することも可能である。
    差し替え方法については、「:ref:`SystemDateTestingUnitTest`」を参照されたい。

|

pom.xmlの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| terasoluna-gfw-jodatimeへの依存関係を追加する。
| マルチプロジェクト構成の場合は、domainプロジェクトの\ :file:`pom.xml`\(:file:`projectName-domain/pom.xml`)に追加する。

`ブランクプロジェクト <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank>`_ \ からプロジェクトを生成した場合は、terasoluna-gfw-jodatimeへの依存関係は、設定済の状態である。

.. code-block:: xml

    <dependencies>

        <!-- (1) -->
        <dependency>
            <groupId>org.terasoluna.gfw</groupId>
            <artifactId>terasoluna-gfw-jodatime-dependencies</artifactId>
            <type>pom</type>
        </dependency>

    </dependencies>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.80\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 80

    * - 項番
      - 説明
    * - (1)
      - terasoluna-gfw-jodatime-dependenciesをdependenciesに追加する。
        terasoluna-gfw-jodatime-dependenciesには、Joda Time用のDate FactoryとJoda Time関連のライブラリへの依存関係が定義されている。

.. note::

    上記設定例は、依存ライブラリのバージョンを親プロジェクトである terasoluna-gfw-parent で管理する前提であるため、pom.xmlでのバージョンの指定は不要である。

|

サーバーのシステム時刻を返却する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ ``org.terasoluna.gfw.common.date.jodatime.DefaultJodaTimeDateFactory``\ を使用する。

**bean定義ファイル([projectname]-env.xml)**

.. code-block:: xml

    <bean id="dateFactory" class="org.terasoluna.gfw.common.date.jodatime.DefaultJodaTimeDateFactory" />  <!-- (1) -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``DefaultJodaTimeDateFactory``\ クラスをbean定義する。

|

.. _dateFactory-java:

**Javaクラス**

.. code-block:: java

    @Inject
    JodaTimeDateFactory dateFactory;  // (2)

    public TourInfoSearchCriteria setUpTourInfoSearchCriteria() {

        DateTime dateTime = dateFactory.newDateTime();  // (3)

        // omitted
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (2)
     - | Date Factoryを利用するクラスにインジェクションする。
   * - | (3)
     - | 利用したい日付のクラスインスタンスを返却するメソッドを呼び出す。
       | 上記例では、\ ``org.joda.time.DateTime``\ 型のインスタンスを取得している。

|

DBから取得した固定の時刻を返却する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ ``org.terasoluna.gfw.common.date.jodatime.JdbcFixedJodaTimeDateFactory``\ を使用する。

**bean定義ファイル**

.. code-block:: xml

    <bean id="dateFactory" class="org.terasoluna.gfw.common.date.jodatime.JdbcFixedJodaTimeDateFactory" >  <!-- (1) -->
        <property name="dataSource" ref="dataSource" />  <!-- (2) -->
        <property name="currentTimestampQuery" value="SELECT now FROM system_date" />  <!-- (3) -->
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - \ ``JdbcFixedJodaTimeDateFactory``\ をbean定義する。
   * - | (2)
     - \ ``dataSource``\ プロパティに、固定時刻を管理するためのテーブルが存在するデータソース(\ ``javax.sql.DataSource``\ )を指定する。
   * - | (3)
     - \ ``currentTimestampQuery``\ プロパティに、固定時刻を取得するためのSQLを設定する。

|

**テーブル設定例**

以下のようにテーブルを作成し、レコードを追加する必要がある。

.. code-block:: sql

  CREATE TABLE system_date(now timestamp NOT NULL);
  INSERT INTO system_date(now) VALUES (current_date);

.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 80

   * - レコード番号
     - now
   * - 1
     - 2013-01-01 01:01:01.000

|

**Javaクラス**

.. code-block:: java

    @Inject
    JodaTimeDateFactory dateFactory;

    @RequestMapping(value="datetime", method = RequestMethod.GET)
    public String listConfirm(Model model) {

        for (int i=0; i < 3; i++) {
            model.addAttribute("jdbcFixedDateFactory" + i, dateFactory.newDateTime()); // (4)
            model.addAttribute("DateTime" + i, new DateTime()); // (5)
        }

        return "date/dateTimeDisplay";
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (4)
     - Date Factoryから取得したシステム時刻を画面に渡す。

       実行結果を確認すると、DBに設定した固定の値が出力されている事がわかる。
   * - | (5)
     - 確認用に\ ``new DateTime()``\ の結果を画面に渡す。

       実行結果を確認すると、毎回異なる値(アプリケーションサーバのシステム時刻)が出力されている事がわかる。

|

**実行結果**

.. figure:: ./images/system-date-jdbc-fixed-date-factory.png
    :alt: system-date-jdbc-fixed-date-factory
    :width: 40%

|

**SQLログ**

.. code-block:: console

    16. SELECT now FROM system_date {executed in 0 msec}
    17. SELECT now FROM system_date {executed in 1 msec}
    18. SELECT now FROM system_date {executed in 0 msec}

Date Factoryのメソッドを呼び出すと、DBへのアクセスログが出力される。
SQLログを出力するために、 :doc:`../DataAccessDetail/DataAccessCommon` で説明した\ ``Log4jdbcProxyDataSource``\ を使用している。

|

サーバーのシステム時刻にDBに登録した差分値を加算した時刻を返却する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ ``org.terasoluna.gfw.common.date.jodatime.JdbcAdjustedJodaTimeDateFactory``\ を使用する。

**bean定義ファイル**

.. code-block:: xml

  <bean id="dateFactory" class="org.terasoluna.gfw.common.date.jodatime.JdbcAdjustedJodaTimeDateFactory" > <!-- (1) -->
    <property name="dataSource" ref="dataSource" /> <!-- (2) -->
    <property name="adjustedValueQuery" value="SELECT diff * 60 * 1000 FROM operation_date" /> <!-- (3) -->
  </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - \ ``JdbcAdjustedJodaTimeDateFactory``\ をbean定義する。
   * - | (2)
     - \ ``dataSource``\ プロパティに、差分値を管理するためのテーブルが存在するデータソース(\ ``javax.sql.DataSource``\ )を指定する。
   * - | (3)
     - \ ``adjustedValueQuery``\ プロパティに、差分値を取得するためのSQLを設定する。

       上記SQLは、差分値の単位を"minutes"にする場合のSQLである。

|

**テーブル設定例**

以下のようにテーブルを作成し、レコードを追加する必要がある。

.. code-block:: sql

  CREATE TABLE operation_date(diff bigint NOT NULL);
  INSERT INTO operation_date(diff) VALUES (-1440);

.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 80

   * - レコード番号
     - diff
   * - 1
     - -1440

| 本例では、差分値の単位を"minutes"としている。(DBのデータは-1440分=1日前を指定)
| 取得結果をミリ秒（整数値）に変換することで、DB上の値の単位は、日・時・分・秒・ミリ秒のいずれでも問題ない。


.. note::

    上記のSQLはPostgreSQL用である。Oracleの場合は\ ``BIGINT``\ の代わりに\ ``NUMBER(19)``\ を使用すればよい。

.. tip::

    差分値の単位を"minutes"以外にしたい場合は、以下のようなSQLを\ ``adjustedValueQuery``\ プロパティに指定すればよい。

     .. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
     .. list-table::
         :header-rows: 1
         :widths: 25 75
         :class: longtable

         * - 差分値の単位
           - SQL
         * - milliseconds
           - SELECT diff FROM operation_date
         * - seconds
           - SELECT diff * 1000 FROM operation_date
         * - hours
           - SELECT diff * 60 * 60 * 1000 FROM operation_date
         * - days
           - SELECT diff * 24 * 60 * 60 * 1000 FROM operation_date

|

**Javaクラス**

.. code-block:: java

    @Inject
    JodaTimeDateFactory dateFactory;

    @RequestMapping(value="datetime", method = RequestMethod.GET)
    public String listConfirm(Model model) {

        model.addAttribute("firstExpectedDate", new DateTime());  // (4)
        model.addAttribute("serverTime", dateFactory.newDateTime());  // (5)
        model.addAttribute("lastExpectedDate", new DateTime());  // (6)

        return "date/dateTimeDisplay";
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (4)
     - 確認用に、Date Factoryのメソッドを呼び出す前の時刻を画面に渡す。
   * - | (5)
     - Date Factoryから取得したシステム時刻を画面に渡す。

       実行結果を確認すると、実行時から1440分を引いた時刻が出力されている事がわかる。
   * - | (6)
     - 確認用に、Date Factoryのメソッドを呼び出した後の時刻を画面に渡す。

|

**実行結果**

.. figure:: ./images/system-date-jdbc-adjusted-date-factory.png
    :alt: system-date-jdbc-fixed-date-factory
    :width: 40%

|

**SQLログ**

.. code-block:: xml

    17. SELECT diff * 60 * 1000 FROM operation_date {executed in 1 msec}

Date Factoryのメソッドを呼び出すと、DBへのアクセスログが出力される。

|

差分のキャッシュとリロード方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. _useCache:

差分値を0にして、本番環境で利用する場合に、差分を毎回DBから取得するのは性能が悪い。
そこで、\ ``JdbcAdjustedJodaTimeDateFactory``\ では、SQLを発行して取得した差分値をキャッシュすることを可能にしている。
起動時に取得した値をキャッシュした後、リクエスト毎のテーブルアクセスは行わない。

**bean定義ファイル**

.. code-block:: xml

  <bean id="dateFactory" class="org.terasoluna.gfw.common.date.jodatime.JdbcAdjustedJodaTimeDateFactory" >
    <property name="dataSource" ref="dataSource" />
    <property name="adjustedValueQuery" value="SELECT diff * 60 * 1000 FROM operation_date" />
    <property name="useCache" value="true" /> <!-- (1) -->
  </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``true``\ の場合、テーブルから取得した差分値をキャッシュする。デフォルトは\ ``false``\ でキャッシュは行わない。
       | \ ``false``\ の場合はDate Factoryのメソッド呼び出し時に毎回SQLを実行する。

|

キャッシュの設定をしたうえで差分値を変更したい場合は、テーブルの値を変更後、
``JdbcAdjustedJodaTimeDateFactory.reload()``\ メソッドを実行することで、キャッシュする値を再読み込みすることができる。

**Javaクラス**

.. code-block:: java

    @Controller
    @RequestMapping(value = "reload")
    public class ReloadAdjustedValueController {

        @Inject
        JdbcAdjustedJodaTimeDateFactory dateFactory;

        // omitted

        @RequestMapping(method = RequestMethod.GET)
        public String reload() {

            long adjustedValue = dateFactory.reload(); // (2)

            // omitted
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (2)
     - \ ``JdbcAdjustedJodaTimeDateFactory``\ の\ ``reload``\ メソッドを実行することで、
       テーブルから差分を読み直す。

|

Testing
--------------------------------------------------------------------------------

テストを実施する際には、現在日時ではなく別の日時に変更することが必要になる場合がある。

.. tabularcolumns:: |p{0.15\linewidth}|p{0.25\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 15 25 60

    * - 環境
      - 使用するDate Factory
      - 試験内容
    * - Unit Test
      - DefaultJodaTimeDateFactory
      - 日付に関わる試験はDate Factoryをmock化する。
    * - Integration Test
      - DefaultJodaTimeDateFactory
      - 日付に関わらない試験
    * -
      - JdbcFixedJodaTimeDateFactory
      - 特定の日付、時刻に固定して試験を実施する場合
    * -
      - JdbcAdjustedJodaTimeDateFactory
      - 外部システムとの連携があり、1日の試験の中で日付の流れを考慮して複数日の試験を実施する場合
    * - System Test
      - JdbcAdjustedJodaTimeDateFactory
      - 試験の日付を指定して実施する場合や、未来の日付における試験を実施する場合
    * - Production
      - DefaultJodaTimeDateFactory
      - 実際の時刻と変更する可能性が無い場合
    * -
      - JdbcAdjustedJodaTimeDateFactory
      - **時刻を変更する可能性を運用上残しておきたい場合。**

        **通常時は差を0とし、必要な際のみ差を与える。**
        **必ず、** :ref:`useCache<useCache>` **をtrueに設定すること**

|

.. _SystemDateTestingUnitTest:

Unit Test
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Unit Testでは、時刻を登録してその時刻が想定通りに更新されたのかを検証したい場合がある。

そのような場合、処理中にサーバー時刻をそのまま登録してしまうと、
テスト実行のたびに値が異なるため、JUnitでの回帰試験が難しくなる。
そこで、Date Factoryを用いることで、登録する時刻を任意の値に固定化することができる。


ミリ秒単位で時刻が一致するようにするため、mockを使用する。Date Factoryに値を設定し、固定日付を返却する例を下記に示す。
本例では、mockに\ `mockito <https://code.google.com/p/mockito/>`_\ を使用する。

**Javaクラス**

.. code-block:: java

    import org.terasoluna.gfw.common.date.jodatime.JodaTimeDateFactory;

    // omitted

    @Inject
    StaffRepository staffRepository;

    @Inject
    JodaTimeDateFactory dateFactory;

    @Override
    public Staff staffUpdateTel(String staffId, String tel) {

        // ex staffId=0001
        Staff staff = staffRepository.findOne(staffId);

        // ex tel = "0123456789"
        staff.setTel(tel);

        // set ChangeMillis
        staff.setChangeMillis(dateFactory.newDateTime()); // (1)

        staffRepository.save(staff);

        return staff;
    }

    // omitted

**JUnitソース**

.. code-block:: java

    import static org.junit.Assert.*;
    import static org.hamcrest.CoreMatchers.*;
    import static org.mockito.Mockito.*;

    import org.joda.time.DateTime;
    import org.junit.Before;
    import org.junit.Test;
    import org.terasoluna.gfw.common.date.jodatime.JodaTimeDateFactory;

    public class StaffServiceTest {

        StaffService service;

        StaffRepository repository;

        JodaTimeDateFactory dateFactory;

        DateTime now;

        @Before
        public void setUp() {
            service = new StaffService();
            dateFactory = mock(JodaTimeDateFactory.class);
            repository = mock(StaffRepository.class);
            now = new DateTime();
            service.dateFactory = dateFactory;
            service.staffRepository = repository;
            when(dateFactory.newDateTime()).thenReturn(now); // (2)
        }

        @After
        public void tearDown() throws Exception {
        }

        @Test
        public void testStaffUpdateTel() {

            Staff setDataStaff = new Staff();
            when(repository.findOne("0001")).thenReturn(setDataStaff);

            // execute
            Staff staff = service.staffUpdateTel("0001", "0123456789");

            //assert
            assertThat(staff.getChangeMillis(), is(now)); // (3)

        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | (2)のmockで指定した値が取得され設定される。
   * - | (2)
     - | mockで日時をData Factoryの戻り値に設定。
   * - | (3)
     - | 設定した固定値と同じになるため、 **success** を返す。

|

日付によって処理が変わる場合の例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

"予約したツアーは出発日の7日前を過ぎるとキャンセル出来ない"という仕様を実装したServiceクラスを例に用いて説明する。

**Javaクラス**

.. code-block:: java

    import org.terasoluna.gfw.common.date.jodatime.JodaTimeDateFactory;

    // omitted

    @Inject
    JodaTimeDateFactory dateFactory;

    // omitted

    @Override
    public void cancel(String reserveNo) throws BusinessException {
        // omitted

        LocalDate today = dateFactory.newDateTime().toLocalDate(); // (1)
        LocalDate cancelLimit = tourInfo.getDepDay().minusDays(7); // (2)

        if (today.isAfter(cancelLimit)) { // (3)
            // omitted (4)
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
     - | 現在日時を取得する。``LocalDate`` については :doc:`./JodaTime` を参照されたい。
   * - | (2)
     - | 対象のツアーのキャンセル期限日を計算する。
   * - | (3)
     - | 今日がキャンセル期限日より後であるかの判定する。
   * - | (4)
     - | キャンセル期限日を過ぎた場合は\ ``BusinessException``\ をスローする。

|

**JUnitソース**

.. code-block:: java

  @Before
  public void setUp() {
      service = new ReserveServiceImpl();

      // omitted

      Reserve reserveResult = new Reserve();
      reserveResult.setDepDay(new LocalDate(2012, 10, 10)); // (5)
      when(reserveRepository.findOne((String) anyObject())).thenReturn(
              reserveResult);
      dateFactory = mock(JodaTimeDateFactory.class);
      service.dateFactory = dateFactory;
  }

  @Test
  public void testCancel01() {

    // omitted

    now = new DateTime(2012, 10, 1, 0, 0, 0, 0);
    when(dateFactory.newDateTime()).thenReturn(now); // (6)

    // run
    service.cancel(reserveNo); // (7)

    // omitted
  }

  @Test(expected = BusinessException.class)
  public void testCancel02() {

    // omitted

    now = new DateTime(2012, 10, 9, 0, 0, 0, 0);
    when(dateFactory.newDateTime()).thenReturn(now); // (8)

    try {
        // run
        service.cancel(reserveNo); // (9)
        fail("Illegal Route");
    } catch (BusinessException e) {
        // assert message if required
        throw e;
    }
  }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (5)
     - | Repositoryクラスからの取得するツアー予約情報の出発日を2012/10/10とする。
   * - | (6)
     - | dateFactory.newDateTime()の返り値を2012/10/1とする。
   * - | (7)
     - | cancelを実行し、キャンセル可能な日付より前なので、キャンセルが成功する。
   * - | (8)
     - | dateFactory.newDateTime()の返り値を2012/10/9とする。
   * - | (9)
     - | cancel実行し、キャンセル可能な日付より後なので、キャンセルが失敗する。

|

Integration Test
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Integration Testでは、システム連携先と疎通・連携確認のために1日の間に
何日分ものデータ（例えばファイル）を作成して受け渡しを行う場合がある。

.. figure:: ./images/DateFactoryIT.png
   :alt: DateFactorySI
   :width: 90%

実際の日付が2012/10/1の場合、
\ ``JdbcAdjustedJodaTimeDateFactory``\ を使用し、試験対象の日付との差分を計算するSQLを設定する。


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | 1
     - | 9:00-11:00の間は差分値を"0 days"とし、Date Factoryの返り値を2012/10/1とする。
   * - | 2
     - | 11:00-13:00の間は差分値を"9 days"とし、Date Factoryの返り値を2012/10/10とする。
   * - | 3
     - | 13:00-15:00の間は差分値を"30 days"とし、Date Factoryの返り値を2012/10/31とする。
   * - | 4
     - | 15:00-17:00の間は差分値を"31 days"とし、Date Factoryの返り値を2012/11/1とする。

テーブルの値を変更するのみで、日付を変更することが可能である。

|

System Test
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

System Testでは運用日を想定してテストシナリオを作成し、試験を実施することがある。

.. figure:: ./images/DateFactoryST.png
   :alt: DateFactoryPT
   :width: 90%

\ ``JdbcAdjustedJodaTimeDateFactory``\ を使用し、日付差を計算するSQLを設定する。
図中の1,2,3,4のように実際の日付と運用日の対応表を作成する。テーブルの差分値を変更するのみで、思い通りの日付でテストすることが可能となる。

|

Production
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ ``JdbcAdjustedJodaTimeDateFactory``\ を使用し、差分値を0とすることで、ソースを変更せずDate Factoryの返り値を、
実際の日付と同じにできる。bean定義ファイルもSystem Testの時から変更を必要としない。
また、日時を変更する必要が生じてもテーブルの値を変更することで、Date Factoryの返り値を変更できる。

.. warning::

    Production環境で使用する場合は、production環境で使用するテーブルの差分値が0となっていることを確認すること。

    **設定例**

    - production環境で初めてテーブルを使用する場合
        - INSERT INTO operation_date (diff) VALUES (0);
    - production環境で試験実施済みの場合
        - UPDATE operation_date SET diff=0;

    を実行すること。

    **必ず、** :ref:`useCache<useCache>` **をtrueに設定すること**

時間を変更することがない場合は、\ ``DefaultJodaTimeDateFactory``\ に設定ファイルを変更することを推奨する。

.. raw:: latex

   \newpage

