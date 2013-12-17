システム時刻
================================================================================

.. contents:: 目次
   :depth: 3
   :local:

Overview
--------------------------------------------------------------------------------

| アプリケーション開発中は、サーバーのシステム時刻ではなく、任意の日時でテストを行う必要が生じる。
| Production環境においても日付を戻してリカバリ処理を行うことも想定される。

| そのため、日時の取得ではサーバーのシステム時刻ではなく、開発・運用側で任意の日時に設定可能になっていることが望ましい。

| 共通ライブラリでは、返却する時刻を任意に変更できる ``org.terasoluna.gfw.common.date.DateFactory`` インタフェースを提供している。
| ``DateFactory``\ により、以下5種類の型で日付を返却することができる。

.. list-table::
   :header-rows: 1
   :widths: 30 70

   * - method
     - type
   * - newDateTime
     - org.joda.time.DateTime
   * - newTimestamp
     - java.sql.Timestamp
   * - newDate
     - java.util.Date
   * - newSqlDate
     - java.sql.Date
   * - newTime
     - java.sql.Time

|

How to use
--------------------------------------------------------------------------------

| bean定義ファイルに、\ ``DateFactory``\ の実装クラスを定義し、Javaクラスに\ ``DateFactory``\ インタフェースでインジェクションする。
| 実装クラスは使用用途に応じて、以下から選択する。


.. list-table::
   :header-rows: 1
   :widths: 30 35 35

   * - クラス名
     - 概要
     - 備考
   * - | org.terasoluna.gfw.common.date.DefaultDateFactory
     - | アプリケーションサーバーのシステム時刻を返却する。
     - | new DateTime()での取得値と同等であり、時刻の変更はできない。
   * - | org.terasoluna.gfw.common.date.JdbcFixedDateFactory
     - | DBに登録した固定の時刻を返却
     - | 完全に時刻を固定する必要のあるIntegration Test環境で使用されることを想定している。
       | Performance Test環境や、Production環境では使用しない。
       | テーブルの定義が必要である。
   * - | org.terasoluna.gfw.common.date.JdbcAdjustedDateFactory
     - | アプリケーションサーバーのシステム時刻にDBに登録した
       | 差分(ミリ秒)を加算した時刻を返却する。
     - | Integration Test環境やSystem Test環境で使用されることを想定している。
       | 差分値を0に設定することでProduction環境でも使用できる。
       | テーブルの定義が必要である。

|

    .. note::

        各クラスを設定するbean定義ファイルは、環境ごとに切り替えられるように、[projectname]-env .xmlに定義することを推奨する。
        DateFactoryを利用することにより、bean定義ファイルの設定を変更するだけで、ソースを変更せずに日時の変更が可能となる。
        bean定義ファイルの記載例は後述する。

|

サーバーのシステム時刻を返却する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``org.terasoluna.gfw.common.date.DefaultDateFactory``\ を使用する。利用方法は、以下を参照されたい。

**bean定義ファイル([projectname]-env.xml)**

.. code-block:: xml

    <bean id="dateFactory" class="org.terasoluna.gfw.common.date.DefaultDateFactory" />  <!-- (1) -->

.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | DefaultDateFactoryクラスをbean定義する。

.. _dateFactory-java:

**Javaクラス**

.. code-block:: java

    @Inject
    protected DateFactory dateFactory;  // (1)

    public TourInfoSearchCriteria setUpTourInfoSearchCriteria() {

        DateTime dateTime = dateFactory.newDateTime();  // (2)

        // omitted
    }

.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | DateFactoryを利用するクラスにインジェクションする。
   * - | (2)
     - | 利用したい日付のクラスインスタンスを返却するメソッドを呼び出す。
       | ``org.joda.time.DateTime`` 型で取得する。

|

    .. note::
       Joda Time、フォーマットなどについては、 :doc:`./Utilities/JodaTime` を参照されたい。

    .. note::
        JUnitなどで日時を変更して試験を行いたい場合、Factoryの実装クラスをmockクラスに差し替えることで、
        任意の日時を設定することも可能である。

|

DBから取得した固定の時刻を返却する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``org.terasoluna.gfw.common.date.JdbcFixedDateFactory``\ を使用する。利用方法は、以下を参照されたい。

**bean定義ファイル**

.. code-block:: xml

    <bean id="dateFactory" class="org.terasoluna.gfw.common.date.JdbcFixedDateFactory" >  <!-- (1) -->
        <property name="dataSource" ref="dataSource" />  <!-- (2) -->
        <property name="currentTimestampQuery" value="SELECT now FROM system_date" />  <!-- (3) -->
    </bean>

.. list-table::
   :header-rows: 1
   :widths: 10 100

   * - 項番
     - 説明
   * - | (1)
     - | ``org.terasoluna.gfw.common.date.JdbcFixedDateFactory`` をbean定義する。
   * - | (2)
     - データソース( ``javax.sql.DataSource`` )の設定。
   * - | (3)
     - | 固定時刻取得SQL(``currentTimestampQuery``)の設定。
       | テーブルに指定した、日時を返却するSQLを設定する。


**テーブル設定例**

| 以下のようにテーブルを作成し、レコードを追加する必要がある。

.. code-block:: sql

  CREATE TABLE system_date(now timestamp NOT NULL);
  INSERT INTO system_date(now) VALUES (current_date);

.. list-table::
   :header-rows: 1
   :widths: 20 80

   * - レコード番号
     - now
   * - 1
     - 2013-01-01 01:01:01.000

**Javaクラス**

.. code-block:: java

    @Inject
    protected DateFactory dateFactory;

    @RequestMapping(value="datetime", method = RequestMethod.GET)
    public String listConfirm(Model model) {

        for (int i=0; i < 3; i++) {
            model.addAttribute("jdbcFixedDateFactory" + i, dateFactory.newDateTime()); // (1)
            model.addAttribute("DateTime" + i, new DateTime()); // (2)
        }

        return "date/dateTimeDisplay";
    }

**実行結果**

.. figure:: ./images/system-date-jdbc-fixed-date-factory.png
   :alt: system-date-jdbc-fixed-date-factory
   :width: 30%

.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | ``JdbcFixedDateFactory.newDateTime()`` の結果を画面に渡す。
       | DBに設定した固定の値が出力されている。
   * - | (2)
     - | 確認用に\ ``new DateTime()``\ の結果を画面に渡す。
       | 出力結果が毎回異なる値となっている。

**SQLログ**

.. code-block:: xml

    16. SELECT now FROM system_date {executed in 0 msec}
    17. SELECT now FROM system_date {executed in 1 msec}
    18. SELECT now FROM system_date {executed in 0 msec}

| ``JdbcFixedDateFactory.newDateTime()`` による、DBへのアクセスログが出力される。
| SQLログを出力するために、 :doc:`./DataAccessCommon` で説明した\ ``Log4jdbcProxyDataSource``\ を使用している。

|

サーバーのシステム時刻にDBに登録した差分値を加算した時刻を返却する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| ``org.terasoluna.gfw.common.date.JdbcAdjustedDateFactory``\ を使用する。
| ``adjustedValueQuery``\ プロパティに設定されたSQLを実行して差分値を取得する。
| 利用方法は、以下を参照されたい。

**bean定義ファイル**

.. code-block:: xml

  <bean id="dateFactory" class="org.terasoluna.gfw.common.date.JdbcAdjustedDateFactory" >
    <property name="dataSource" ref="dataSource" />
    <!-- <property name="adjustedValueQuery" value="SELECT diff FROM operation_date" /> --><!-- (1) -->
    <!-- <property name="adjustedValueQuery" value="SELECT diff * 1000 FROM operation_date" /> --><!-- (2) -->
    <property name="adjustedValueQuery" value="SELECT diff * 60 * 1000 FROM operation_date" /><!-- (3) -->
    <!-- <property name="adjustedValueQuery" value="SELECT diff * 60 * 60 * 1000 FROM operation_date" /> --><!-- (4) -->
    <!-- <property name="adjustedValueQuery" value="SELECT diff * 24 * 60 * 60 * 1000 FROM operation_date" /> --><!-- (5) -->
  </bean>

.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | operation_dateテーブルに登録した差分値の単位を"milliseconds"として使用する場合のSQL
   * - | (2)
     - | operation_dateテーブルに登録した差分値の単位を"seconds"として使用する場合のSQL
   * - | (3)
     - | operation_dateテーブルに登録した差分値の単位を"minutes"として使用する場合のSQL
   * - | (4)
     - | operation_dateテーブルに登録した差分値の単位を"hours"として使用する場合のSQL
   * - | (5)
     - | operation_dateテーブルに登録した差分値の単位を"days"として使用する場合のSQL

**テーブル設定例**

| 以下のようにテーブルを作成し、レコードを追加する必要がある。

.. code-block:: sql

  CREATE TABLE operation_date(diff bigint NOT NULL);
  INSERT INTO operation_date(diff) VALUES (-1440);

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

**Javaクラス**

.. code-block:: java

    @Inject
    protected DateFactory dateFactory;

    @RequestMapping(value="datetime", method = RequestMethod.GET)
    public String listConfirm(Model model) {

        model.addAttribute("firstExpectedDate", new DateTime());  // (1)
        model.addAttribute("serverTime", dateFactory.newDateTime());  // (2)
        model.addAttribute("lastExpectedDate", new DateTime());  // (3)

        return "date/dateTimeDisplay";
    }

**実行結果**

.. figure:: ./images/system-date-jdbc-adjusted-date-factory.png
   :alt: system-date-jdbc-fixed-date-factory
   :width: 30%

.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 確認用に、\ ``dateFactory``\ による\ ``DateTime``\ 生成よりも前の時刻を画面に渡す。
   * - | (2)
     - | ``JdbcAdjustedDateFactory.newDateTime()``\ の結果を画面に渡す。
       | 実行時から1440分を引いた時刻が取得されている。
   * - | (3)
     - | 確認用に、\ ``dateFactory``\ による\ ``DateTime``\ 生成よりも後の時刻を設定する

**SQLログ**

.. code-block:: xml

    17. SELECT diff * 60 * 1000 FROM operation_date {executed in 1 msec}

| ``dateFactory.newDateTime()`` による、DBへのアクセスログが出力される。

|

差分のキャッシュとリロード方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. _useCache:

差分値を0にして、本番環境で利用する場合に、差分を毎回DBから取得するのは性能が悪い。
そこで、JdbcAdjustedDateFactoryでは、取得結果をキャッシュすることを可能にしている。
起動時に取得した値をキャッシュした後、リクエスト毎のテーブルアクセスは行わない。

**bean定義ファイル**

.. code-block:: xml

  <bean id="dateFactory" class="org.terasoluna.gfw.common.date.JdbcAdjustedDateFactory" >
    <property name="dataSource" ref="dataSource" />
    <property name="adjustedValueQuery" value="SELECT diff * 60 * 1000 FROM operation_date" />
    <property name="useCache" value="true" /> <!-- (1) -->
  </bean>

.. list-table::
   :header-rows: 1
   :widths: 10 100

   * - 項番
     - 説明
   * - | (1)
     - | trueの場合、テーブルから取得した値をキャッシュする。デフォルトはfalseでキャッシュは行わない。
       | falseの場合はDateFactory利用時に毎回SQLを実行する。

キャッシュの設定をしたうえで、差分値を変更したい場合はテーブルの値を変更後、
``JdbcAdjustedDateFactory.reload()``\ メソッドを実行することで、キャッシュする値を再読み込みすることができる。

**Javaクラス**

.. code-block:: java

    @Controller
    @RequestMapping(value = "reload")
    public class ReloadAdjustedValueController {

        @Inject
        protected JdbcAdjustedDateFactory dateFactory;

        // omitted

        @RequestMapping(method = RequestMethod.GET)
        public String reload() {

            long adjustedValue = dateFactory.reload(); // (1)

            // omitted
        }

.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | JdbcAdjustedDateFactoryのreloadメソッドを実行することで、
       | テーブルから差分を読み直す。

|

Testing
--------------------------------------------------------------------------------

| テストを実施する際には、現在日時ではなく別の日時に変更することが必要になる場合がある。

+----------------------+-------------------------+-----------------------------------------------------------------------------------------------+
| 環境                 | 使用するDateFactory     | 試験内容                                                                                      |
+======================+=========================+===============================================================================================+
| Unit Test            | DefaultDateFactory      | 日付に関わる試験はDataFactoryをmock化する。                                                   |
+----------------------+-------------------------+-----------------------------------------------------------------------------------------------+
| Integration Test     | DefaultDateFactory      | 日付に関わらない試験                                                                          |
|                      +-------------------------+-----------------------------------------------------------------------------------------------+
|                      | JdbcFixedDateFactory    | 特定の日付、時刻に固定して試験を実施する場合                                                  |
|                      +-------------------------+-----------------------------------------------------------------------------------------------+
|                      | JdbcAdjustedDateFactory | 外部システムとの連携があり、1日の試験の中で日付の流れを考慮して複数日の試験を実施する場合     |
+----------------------+-------------------------+-----------------------------------------------------------------------------------------------+
| System Test          | JdbcAdjustedDateFactory | 試験の日付を指定して実施する場合や、未来の日付における試験を実施する場合                      |
+----------------------+-------------------------+-----------------------------------------------------------------------------------------------+
| Production           | DefaultDateFactory      | 実際の時刻と変更する可能性が無い場合                                                          |
|                      +-------------------------+-----------------------------------------------------------------------------------------------+
|                      | JdbcAdjustedDateFactory || **時刻を変更する可能性を運用上残しておきたい場合。**                                         |
|                      |                         || **通常時は差を0とし、必要な際のみ差を与える。**                                              |
|                      |                         || **必ず、** :ref:`useCache<useCache>` **をtrueに設定すること**                                |
+----------------------+-------------------------+-----------------------------------------------------------------------------------------------+

|

Unit Test
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| UnitTestでは、時刻を登録してその時刻が想定通りに更新されたのかを検証したい場合がある。

| そのような場合、処理中にサーバー時刻をそのまま登録してしまうと、
| テスト実行のたびに値が異なるため、JUnitでの回帰試験が難しくなる。
| そこで、DateFacotyを用いることで、登録する時刻を任意の値に固定化することができる。


| ミリ秒単位で時刻が一致するようにするため、mockを使用する。dateFactoryに値を設定し、固定日付を返却する例を下記に示す。
| 本例では、mockに\`mockito <https://code.google.com/p/mockito/>\` を使用する。

**Javaクラス**

.. code-block:: java

    import org.terasoluna.gfw.common.date.DateFactory;

    // omitted

    @Inject
    protected StaffRepository staffRepository;

    @Inject
    protected DateFactory dateFactory;

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
    import org.terasoluna.gfw.common.date.DateFactory;

    public class StaffServiceTest {

        StaffService service;

        StaffRepository repository;

        DateFactory dateFactory;

        DateTime now;

        @Before
        public void setUp() {
            service = new StaffService();
            dateFactory = mock(DateFactory.class);
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

.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | (2)のmockで指定した値が取得され設定される。
   * - | (2)
     - | mockで日時をDataFactoryの戻り値に設定。
   * - | (3)
     - | 設定した固定値と同じになるため、 **success** を返す。

|

日付によって処理が変わる場合の例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| "予約したツアーは出発日の7日前を過ぎるとキャンセル出来ない"という仕様を実装したServiceクラスを例に用いて説明する。

**Javaクラス**

.. code-block:: java

  import org.terasoluna.gfw.common.date.DateFactory;

    // omitted

    @Inject
    protected DateFactory dateFactory;

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

.. list-table::
   :header-rows: 1
   :widths: 10 100

   * - 項番
     - 説明
   * - | (1)
     - | 現在日時を取得する。``LocalDate`` については :doc:`./Utilities/JodaTime` を参照されたい。
   * - | (2)
     - | 対象のツアーのキャンセル期限日を計算する。
   * - | (3)
     - | 今日がキャンセル期限日より後であるかの判定する。
   * - | (4)
     - | キャンセル期限日を過ぎた場合は\ ``BusinessException``\ をスローする。

**JUnitソース**

.. code-block:: java

  @Before
  public void setUp() {
      service = new ReserveServiceImpl();

      // omitted

      Reserve reserveResult = new Reserve();
      reserveResult.setDepDay(new LocalDate(2012, 10, 10)); // (1)
      when(reserveRepository.findOne((String) anyObject())).thenReturn(
              reserveResult);
      dateFactory = mock(DateFactory.class);
      service.dateFactory = dateFactory;
  }

  @Test
  public void testCancel01() {

    // omitted

    now = new DateTime(2012, 10, 1, 0, 0, 0, 0);
    when(dateFactory.newDateTime()).thenReturn(now); // (2)

    // run
    service.cancel(reserveNo); // (3)

    // omitted
  }

  @Test(expected = BusinessException.class)
  public void testCancel02() {

    // omitted

    now = new DateTime(2012, 10, 9, 0, 0, 0, 0);
    when(dateFactory.newDateTime()).thenReturn(now); // (4)

    try {
        // run
        service.cancel(reserveNo); // (5)
        fail("Illegal Route");
    } catch (BusinessException e) {
        // assert message if required
        throw e;
    }
  }

.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | Repositoryクラスからの取得するツアー予約情報の出発日を2012/10/10とする。
   * - | (2)
     - | dateFactory.newDateTime()の返り値を2012/10/1とする。
   * - | (3)
     - | cancelを実行し、キャンセル可能な日付より前なので、キャンセルが成功する。
   * - | (4)
     - | dateFactory.newDateTime()の返り値を2012/10/9とする。
   * - | (5)
     - | cancel実行し、キャンセル可能な日付より後なので、キャンセルが失敗する。

|

Integration Test
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| Integration Testでは、システム連携先と疎通・連携確認のために1日の間に
| 何日分ものデータ（例えばファイル）を作成して受け渡しを行う場合がある。

.. figure:: ./images/DateFactoryIT.png
   :alt: DateFactorySI
   :width: 60%

| 実際の日付が2012/10/1の場合
| JdbcAdjustedDateFactoryを使用し、試験対象の日付との差分を計算するSQLを設定する。


.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | 1
     - | 9:00-11:00の間は差分値を"0 days"とし、dateFactoryの返り値を2012/10/1とする。
   * - | 2
     - | 11:00-13:00の間は差分値を"9 days"とし、dateFactoryの返り値を2012/10/10とする。
   * - | 3
     - | 13:00-15:00の間は差分値を"30 days"とし、dateFactoryの返り値を2012/10/31とする。
   * - | 4
     - | 15:00-17:00の間は差分値を"31 days"とし、dateFactoryの返り値を2012/11/1とする。

テーブルの値を変更するのみで、日付を変更することが可能である。

|

System Test
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

System Testでは運用日を想定してテストシナリオを作成し、試験を実施することがある。

.. figure:: ./images/DateFactoryST.png
   :alt: DateFactoryPT
   :width: 60%

| JdbcAdjustedDateFactoryを使用し、日付差を計算するSQLを設定する。
| 図中の1,2,3,4のように実際の日付と運用日の対応表を作成する。テーブルの差分値を変更するのみで、思い通りの日付でテストすることが可能となる。

|

Production
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| JdbcAdjustedDateFactoryを使用し、差分値を0とすることで、ソースを変更せずdateFactoryの返り値を、
| 実際の日付と同じにできる。bean定義ファイルもSystem Testの時から変更を必要としない。
| また、日時を変更する必要が生じてもテーブルの値を変更することで、dateFactoryの返り値を変更できる。

    .. warning::

        Production環境で使用する場合は、production環境で使用するテーブルの差分値が0となっていることを確認すること。

        **設定例**

        - production環境で初めてテーブルを使用する場合
            - INSERT INTO operation_date (diff) VALUES (0);
        - production環境で試験実施済みの場合
            - UPDATE operation_date SET diff=0;

        を実行すること。

        **必ず、** :ref:`useCache<useCache>` **をtrueに設定すること**

| 時間を変更することがない場合は、DefaultDateFactoryに設定ファイルを変更することを推奨する。
