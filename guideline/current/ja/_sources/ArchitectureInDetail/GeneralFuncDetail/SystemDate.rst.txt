システム時刻
================================================================================

.. only:: html

.. contents:: 目次
  :depth: 3
  :local:

|

Overview
--------------------------------------------------------------------------------

アプリケーション開発中は、サーバーのシステム時刻ではなく、任意の日時でテストを行う必要が生じる。Production環境においても日付を戻してリカバリ処理を行うことも想定される。

そのため、日時の取得ではサーバーのシステム時刻ではなく、開発・運用側で任意の日時に設定可能になっていることが望ましい。

|

共通ライブラリから提供しているコンポーネントについて
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

共通ライブラリでは、システム時刻を取得するためのコンポーネントを提供している。

共通ライブラリから提供しているコンポーネントは、terasoluna-gfw-commonから以下の2つの機能を提供している。

* \ ``java.util.Date``\ を生成するDate Factory
* \ ``java.time.Clock``\ を生成するClock Factory

コンポーネントのクラス図を以下に示す。

.. figure:: ./images_SystemDate/FactoryClassDiagram.png
  :alt: Class Diagram of Factory
  :width: 100%

|

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
    * - | org.terasoluna.gfw.common.time.
        | ClockFactory
      - Javaから提供されている以下のクラスのインスタンスをシステム時刻として取得するためのインタフェース。

        * \ ``java.time.Clock``\

        共通ライブラリでは、本インタフェースの実装クラスとして以下のクラスを提供している。

        * \ ``org.terasoluna.gfw.common.time.DefaultClockFactory``\
        * \ ``org.terasoluna.gfw.common.time.ConfigurableClockFactory``\
        * \ ``org.terasoluna.gfw.common.time.ConfigurableAdjustClockFactory``\
        * \ ``org.terasoluna.gfw.common.time.JdbcClockFactory``\
        * \ ``org.terasoluna.gfw.common.time.JdbcAdjustClockFactory``\

        \ **本ガイドラインでは、本インタフェースに対応する実装クラスを使用することを推奨する。**\

|

How to use
--------------------------------------------------------------------------------

Clock Factoryインタフェースの実装クラスをbean定義ファイルに定義し、Clock FactoryのインスタンスをJavaクラスにインジェクションして使用する。

実装クラスは使用用途に応じて、以下から選択する。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.30\linewidth}|p{0.40\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 30 30 40

  * - クラス名
    - 概要
    - 備考
  * - | org.terasoluna.gfw.common.time.
      | DefaultClockFactory
    - | システム・デフォルトのClockを取得する。
    - | 
  * - | org.terasoluna.gfw.common.time.
      | ConfigurableClockFactory
    - | 指定した固定日時から生成したClockを取得する。
    - | 完全に時刻を固定する必要のあるIntegration Test環境で使用されることを想定しており、Performance Test環境や、Production環境では使用しない。
  * - | org.terasoluna.gfw.common.time.
      | ConfigurableAdjustClockFactory
    - | システム・デフォルトのClockに指定した差分を加算したClockを取得する。
    - | Integration Test環境やSystem Test環境で使用されることを想定している。
      | 差分値を0に設定することでProduction環境でも使用できる。
  * - | org.terasoluna.gfw.common.time.
      | JdbcClockFactory
    - | DBに登録した固定の時刻から生成したClockを取得する。
    - | Integration Test環境やSystem Test環境で使用されることを想定している。
      | このクラスを使用するためには、日時を管理するためのテーブルが必要である。
  * - | org.terasoluna.gfw.common.time.
      | JdbcAdjustClockFactory
    - | システム・デフォルトのClockにDBに登録した差分を加算したClockを取得する。
    - | Integration Test環境やSystem Test環境で使用されることを想定している。
      | 差分値を0に設定することでProduction環境でも使用できる。
      | このクラスを使用するためには、差分値を管理するためのテーブルが必要である。

.. note::

  実装クラスを設定するbean定義ファイルは、環境ごとに切り替えられるように、[projectName]-env.xmlに定義することを推奨する。

  Clock Factoryを利用することにより、bean定義ファイルの設定を変更するだけで、ソースを変更せずに日時の変更が可能となる。

  bean定義ファイルの記載例は後述する。

|

.. _SystemDate_HowToUseDefaultClock:

サーバーのシステム・デフォルトClockを取得する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ ``org.terasoluna.gfw.common.time.DefaultClockFactory``\ を使用する。

\ **bean定義ファイル([projectname]-env.xml)**\

.. code-block:: xml

  <bean id="clockFactory" class="org.terasoluna.gfw.common.time.DefaultClockFactory" />  <!-- (1) -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | \ ``DefaultClockFactory``\ をbean定義する。

|

\ **Javaクラス**\

.. code-block:: java

  @Inject
  ClockFactory clockFactory;  // (2)

  public TourInfoSearchCriteria setUpTourInfoSearchCriteria() {

      Clock fixedClock1 = clockFactory.fixed();  // (3)
      Clock fixedClock2 = clockFactory.fixed(ZoneId.of("Asia/Tokyo"));  // (4)
      Clock tickClock1 = clockFactory.tick();  // (5)
      Clock tickClock2 = clockFactory.tick(ZoneOffset.UTC);  // (6)

      LocalDateTime fixedLocalDateTime1 = LocalDateTime.now(fixedClock1); // (7)
      LocalDateTime fixedLocalDateTime2 = LocalDateTime.now(fixedClock2); // (8)
      LocalDateTime tickLocalDateTime1 = LocalDateTime.now(tickClock1); // (9)
      LocalDateTime tickLocalDateTime2 = LocalDateTime.now(tickClock2); // (10)

      // omitted
  }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (2)
    - | Clock Factoryを利用するクラスにインジェクションする。
  * - | (3)
    - | メソッドを呼び出した瞬間の日時で固定したClockを取得する。
      | タイムゾーンはシステムのデフォルト・タイムゾーンが使用される。
  * - | (4)
    - | メソッドを呼び出した瞬間の日時で固定したClockを取得する。
      | タイムゾーンは指定したタイムゾーンが使用される。
      | この例ではJSTを指定している。
  * - | (5)
    - | メソッドを呼び出した瞬間の日時から時を刻むClockを取得する。
      | タイムゾーンはシステムのデフォルト・タイムゾーンが使用される。
  * - | (6)
    - | メソッドを呼び出した瞬間の日時から時を刻むClockを取得する。
      | タイムゾーンは指定したタイムゾーンが使用される。
      | この例ではUTCを指定している。
  * - | (7)
    - | 取得したClockを引数に指定しシステム日時を取得する。
      | \ **Clock内のタイムスタンプが固定されているため、now(clock)メソッドは毎回同じ日時で返却する。**\
  * - | (8)
    - | 取得したClockを引数に指定しシステム日時を取得する。
      | \ **Clock内のタイムスタンプが固定されているため、now(clock)メソッドは毎回同じ日時で返却する。**\
  * - | (9)
    - | 取得したClockを引数に指定しシステム日時を取得する。
      | \ **Clock内のタイムスタンプは固定されていないため、now(clock)メソッドは呼び出される度に違う日時で返却する。**\
  * - | (10)
    - | 取得したClockを引数に指定しシステム日時を取得する。
      | \ **Clock内のタイムスタンプは固定されていないため、now(clock)メソッドは呼び出される度に違う日時で返却する。**\

|

指定した固定日時から生成したClockを取得する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ ``org.terasoluna.gfw.common.time.ConfigurableClockFactory``\ を使用する。

\ **bean定義ファイル([projectname]-env.xml)**\

.. code-block:: xml

  <!-- (1) -->
  <bean id="defalutConfigurableClockFactory" class="org.terasoluna.gfw.common.time.ConfigurableClockFactory">
      <!-- (2) -->
      <constructor-arg name="localDateTimeString" value="2012-09-11T02:25:15" />
  </bean>

  <bean id="patternConfigurableClockFactory" class="org.terasoluna.gfw.common.time.ConfigurableClockFactory">
      <constructor-arg name="localDateTimeString" value="2012/09/11 02:25:15" />
      <!-- (3) -->
      <constructor-arg name="pattern" value="uuuu/MM/dd HH:mm:ss" />
  </bean>

  <bean id="styleConfigurableClockFactory" class="org.terasoluna.gfw.common.time.ConfigurableClockFactory">
      <constructor-arg name="localDateTimeString" value="2012/09/11 02:25:15" />
      <!-- (4) -->
      <constructor-arg name="dateStyle" value="#{T(java.time.format.FormatStyle).MEDIUM}" />
      <constructor-arg name="timeStyle" value="#{T(java.time.format.FormatStyle).MEDIUM}" />
  </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | \ ``ConfigurableClockFactory``\ をbean定義する。
  * - | (2)
    - | \ ``localDateTimeString``\ プロパティに、固定日時を設定する。
      | この例では日付フォーマットを設定していないため、日付フォーマットは\ `ISO_LOCAL_DATE_TIME <https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/time/format/DateTimeFormatter.html#ISO_LOCAL_DATE_TIME>`_\ が適用される。
      | そのため、日付フォーマットに合わせ"\ ``2012-09-11T02:25:15``\ "を固定日時として設定している。
  * - | (3)
    - | \ ``localDateTimeString``\ プロパティに、日付フォーマットを設定する。
      | この例では日付フォーマットを"\ ``uuuu/MM/dd HH:mm:ss``\ "で定義しているため、指定した日付フォーマットに合わせ"\ ``2012/09/11 02:25:15``\ "を固定日時として設定している。
  * - | (4)
    - | (3)のパターン文字列の代わりに、Styleを設定することも可能である。
      | \ ``dateStyle``\ プロパティに日付のStyle、\ ``timeStyle``\ プロパティに時間のStyleを設定する。
      | 入力可能な値は\ `FormatStyle <https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/time/format/FormatStyle.html#enum-constant-detail>`_\ を参照されたい。

|

\ **Javaクラス**\

| Javaクラスの説明は\ :ref:`SystemDate_HowToUseDefaultClock`\ のJavaクラスを参照されたい。
|

システム・デフォルトのClockに対し差分を追加したClockを取得する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ ``org.terasoluna.gfw.common.time.ConfigurableAdjustClockFactory``\ を使用する。

\ **bean定義ファイル([projectname]-env.xml)**\

.. code-block:: xml

  <!-- (1) -->
  <bean id="configurableAdjustClockFactory" class="org.terasoluna.gfw.common.time.ConfigurableAdjustClockFactory">
      <!-- (2) -->
      <constructor-arg name="adjustedValue" value="1" />
      <!-- (3) -->
      <constructor-arg name="adjustedValueUnit" value="#{T(java.time.temporal.ChronoUnit).DAYS}" />
  </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | \ ``ConfigurableAdjustClockFactory``\ をbean定義する。
  * - | (2)
    - | \ ``adjustedValue``\ プロパティに、差分値を設定する。日付時間単位は(3)で決定する。
  * - | (3)
    - | \ ``adjustedValueUnit``\ プロパティに、日付時間単位を設定する。
      | この例では\ ``DAYS``\ を設定しているため、Factoryで生成されるClockはシステムのデフォルトClockに1日加算した日時となる。
      | 設定できる日付時間単位については\ `ChronoUnit <https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/time/temporal/ChronoUnit.html>`_\ を参照されたい。

      .. note:: 

        \ ``adjustedValueUnit``\ プロパティには推定期間を設定することはできない。（例えば、\ ``MONTH``\ や\ ``YEARS``\ などは設定できない。）

        推定期間を設定した場合、以下の様な例外が出力される。

          .. code-block:: console

            java.time.temporal.UnsupportedTemporalTypeException: Unit must not have an estimated duration

        推定期間かどうかは\ `ChronoUnitのJavaDoc <https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/time/temporal/ChronoUnit.html>`_\ を参照されたい。

|

\ **Javaクラス**\

| Javaクラスの説明は\ :ref:`SystemDate_HowToUseDefaultClock`\ のJavaクラスを参照されたい。
|

DBに登録した固定の時刻から生成したClockを取得する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ ``org.terasoluna.gfw.common.time.JdbcClockFactory``\ を使用する。

\ **bean定義ファイル([projectname]-env.xml)**\

.. code-block:: xml

  <!-- (1) -->
  <bean id="jdbcClockFactory" class="org.terasoluna.gfw.common.time.JdbcClockFactory">
      <!-- (2) -->
      <constructor-arg name="dataSource" ref="dataSource" />
      <!-- (3) -->
      <constructor-arg name="currentTimestampQuery" value="SELECT now FROM system_date" />
  </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | \ ``JdbcClockFactory``\ をbean定義する。
  * - | (2)
    - | \ ``dataSource``\ プロパティに、固定時刻を管理するためのテーブルが存在するデータソース(\ ``javax.sql.DataSource``\ )を指定する。
  * - | (3)
    - | \ ``dataSource``\ プロパティに、固定時刻を取得するためのSQLを設定する。

|

\ **テーブル設定例**\

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

\ **Javaクラス**\

| Javaクラスの説明は\ :ref:`SystemDate_HowToUseDefaultClock`\ のJavaクラスを参照されたい。
|

システム・デフォルトのClockに対しDBから取得した差分を追加したClockを取得する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ ``org.terasoluna.gfw.common.time.JdbcAdjustClockFactory``\ を使用する。

\ **bean定義ファイル([projectname]-env.xml)**\

.. code-block:: xml

  <!-- (1) -->
  <bean id="jdbcAdjustClockFactory" class="org.terasoluna.gfw.common.time.JdbcAdjustClockFactory">
      <!-- (2) -->
      <constructor-arg name="dataSource" ref="dataSource" />
      <!-- (3) -->
      <constructor-arg name="adjustedValueQuery" value="SELECT diff FROM operation_date" />
      <!-- (4) -->
      <constructor-arg name="adjustedValueUnit" value="#{T(java.time.temporal.ChronoUnit).MINUTES}" />
  </bean>
  
.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | \ ``JdbcAdjustClockFactory``\ をbean定義する。
  * - | (2)
    - | \ ``dataSource``\ プロパティに、差分値を管理するためのテーブルが存在するデータソース(\ ``javax.sql.DataSource``\ )を指定する。
  * - | (3)
    - | \ ``adjustedValueQuery``\ プロパティに、差分値を取得するためのSQLを設定する。
  * - | (3)
    - | \ ``adjustedValueUnit``\ プロパティに、プロパティに、日付時間単位を設定する。
      | この例では\ ``SECONDS``\ を設定しているため、Factoryで生成されるClockはシステムのデフォルトClockに\ ``adjustedValueQuery``\ 秒加算した日時となる。
      | 設定できる日付時間単位については\ `ChronoUnit <https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/time/temporal/ChronoUnit.html>`_\ を参照されたい。

      .. note:: 

        \ ``adjustedValueUnit``\ プロパティには推定期間を設定することはできない。（例えば、\ ``MONTH``\ や\ ``YEARS``\ などは設定できない。）

        推定期間を設定した場合、以下の様な例外が出力される。

          .. code-block:: console

            java.time.temporal.UnsupportedTemporalTypeException: Unit must not have an estimated duration

        推定期間かどうかは\ `ChronoUnitのJavaDoc <https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/time/temporal/ChronoUnit.html>`_\ を参照されたい。

|

\ **テーブル設定例**\

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

| ここでは差分値を設定しているだけで、日付時間単位はBeanのpropertyで設定されている。
| 上記例では\ ``MINUTES``\ を設定しているため、-1440分=1日前の値となる。

.. note::

  上記のSQLはPostgreSQL用である。Oracleの場合は\ ``BIGINT``\ の代わりに\ ``NUMBER(19)``\ を使用すればよい。

|

\ **Javaクラス**\

| Javaクラスの説明は\ :ref:`SystemDate_HowToUseDefaultClock`\ のJavaクラスを参照されたい。
|

Testing
--------------------------------------------------------------------------------

テストを実施する際には、現在日時ではなく別の日時に変更することが必要になる場合がある。

.. tabularcolumns:: |p{0.15\linewidth}|p{0.25\linewidth}|p{0.60\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 15 25 60

  * - | 環境
    - | 使用するClock Factory
    - | 試験内容
  * - | Unit Test
    - | ConfigurableClockFactory
    - | 日付に関わる試験はfixメソッドを呼び出してタイムスタンプを固定する。
  * - | Integration Test
    - | DefaultJodaTimeDateFactory
    - | 日付に関わらない試験
  * - | 
    - | JdbcFixedJodaTimeDateFactory
    - | 特定の日付、時刻に固定して試験を実施する場合
  * - | 
    - | JdbcAdjustClockFactory
    - | 外部システムとの連携があり、1日の試験の中で日付の流れを考慮して複数日の試験を実施する場合
  * - | System Test
    - | ConfigurableClockFactory
    - | 試験の日付を指定して実施する場合や、未来の日付における試験を実施する場合
  * - | 
    - | JdbcClockFactory
    - | 試験の日付を指定して実施する場合や、未来の日付における試験を実施する場合
  * - | Production
    - | DefaultClockFactory
    - | 実際の時刻と変更する可能性が無い場合
  * - | 
    - | JdbcAdjustedJodaTimeDateFactory
    - | \ **時刻を変更する可能性を運用上残しておきたい場合。**\
      | \ **通常時は差を0とし、必要な際のみ差を与える。**\

      .. note::

        Factoryから生成されるClockは、生成するタイミングでのみDBにアクセスする。
         
        アプリケーションを停止せずに差分を有効にするためには、Clockが定期的に生成されるような作りを検討されたい。

|

.. _SystemDateTestingUnitTest:

Unit Test
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| Unit Testでは、時刻を登録してその時刻が想定通りに更新されたのかを検証したい場合がある。
| そのような場合、処理中にサーバー時刻をそのまま登録してしまうと、 テスト実行のたびに値が異なるため、JUnitでの回帰試験が難しくなる。 そこで、Clock Factoryを用いることで、登録する時刻を任意の値に固定化することができる。

| ミリ秒単位で時刻が一致するようにするため、mockを使用する。Date Factoryに値を設定し、固定日付を返却する例を下記に示す。
| 本例では、mockに\ `mockito <https://github.com/mockito/mockito>`_\ を使用する。

\ **Javaクラス**\

.. code-block:: java

  import org.terasoluna.gfw.common.time.ClockFactory;

  // omitted

  @Inject
  StaffRepository staffRepository;

  @Inject
  ClockFactory clockFactory;

  @Override
  public Staff staffUpdateTel(String staffId, String tel) {

      // ex staffId=0001
      Staff staff = staffRepository.findByStaffId(staffId);

      // ex tel = "0123456789"
      staff.setTel(tel);

      // set ChangeMillis
      staff.setChangeMillis(LocalDateTime.now(clockFactory.fixed())); // (1)

      staffRepository.save(staff);

      return staff;
  }

  // omitted

\ **JUnitソース**\

.. code-block:: java

  import static org.hamcrest.CoreMatchers.is;
  import static org.hamcrest.MatcherAssert.assertThat;
  import static org.mockito.Mockito.mock;
  import static org.mockito.Mockito.when;

  import java.time.Clock;
  import java.time.LocalDateTime;

  import org.junit.After;
  import org.junit.Before;
  import org.junit.Test;
  import org.terasoluna.gfw.common.time.ClockFactory;

  public class StaffServiceTest {

      StaffService service;

      StaffRepository repository;

      ClockFactory clockFactory;

      LocalDateTime now;

      @Before
      public void setUp() {
          service = new StaffService();
          clockFactory = mock(ClockFactory.class);
          repository = mock(StaffRepository.class);
          Clock clock = Clock.fixed(Clock.systemDefaultZone().instant(), ZoneId
                  .systemDefault()); // (2)
          now = LocalDateTime.now(clock);
          service.clockFactory = clockFactory;
          service.staffRepository = repository;
          when(clockFactory.fixed()).thenReturn(clock); // (2)
      }

      @After
      public void tearDown() throws Exception {
      }

      @Test
      public void testStaffUpdateTel() {

          Staff setDataStaff = new Staff();
          when(repository.findByStaffId("0001")).thenReturn(setDataStaff);

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
    - | (2)のmockで指定したClockが取得されるため、そのClockをもとにLocalDateTimeを設定する。
  * - | (2)
    - | mockで日時固定のClockをClock Factoryの戻り値に設定。

      .. note:: 

        \ ``Clock``\ は\ ``fixed``\ メソッドを使用して固定日時で生成する。

        \ ``systemDefaultZone``\ や\ ``tick``\ メソッドなどで生成した場合は時を刻むため、時間を比較した際にずれる可能性がある。

  * - | (3)
    - | 設定した固定値と同じになるため、\ **success**\ を返す。

|

日付によって処理が変わる場合の例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

"予約したツアーは出発日の7日前を過ぎるとキャンセル出来ない"という仕様を実装したServiceクラスを例に用いて説明する。

\ **Javaクラス**\

.. code-block:: java

  @Inject
  ClockFactory clockFactory;

  // omitted

  @Override
  public void cancel(String reserveNo) throws BusinessException {
  
      // omitted

      LocalDate today = LocalDate.now(clockFactory.fixed()); // (1)
      LocalDate cancelLimit = tourInfo.getDepDay().minus(7L, ChronoUnit.DAYS); // (2)

      if (today.isAfter(cancelLimit)) { // (3)
          throw new BusinessException(message); // (4)
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
    - | 現在日時を取得する。
      | \ ``LocalDate``\ については\ :doc:`./DateAndTime`\ を参照されたい。
  * - | (2)
    - | 対象のツアーのキャンセル期限日を計算する。
  * - | (3)
    - | 今日がキャンセル期限日より後であるか判定する。
  * - | (4)
    - | キャンセル期限日を過ぎた場合は\ ``BusinessException``\ をスローする。

|

\ **JUnitソース**\

.. code-block:: java

  @Before
  public void setUp() {
      service = new ReserveServiceImpl();

      // omitted

      Reserve reserveResult = new Reserve();
      reserveResult.setDepDay(LocalDate.of(2012, 10, 10)); // (5)
      when(reserveRepository.findByStaffId(anyString())).thenReturn(
              reserveResult);
      clockFactory = mock(ClockFactory.class);
      service.clockFactory = clockFactory;
  }

  @Test
  public void testCancel01() {

    // omitted

    LocalDateTime dateTime = LocalDateTime.of(2012, 10, 1, 0, 0, 0)
    Clock clock = Clock.fixed(dateTime.toInstant(ZoneOffset.ofHours(9)),
                    ZoneId.systemDefault());
    when(clockFactory.fixed()).thenReturn(clock); // (6)

    // run
    service.cancel(reserveNo); // (7)

    // omitted
  }

  @Test(expected = BusinessException.class)
  public void testCancel02() {

    // omitted

    LocalDateTime dateTime = LocalDateTime.of(2012, 10, 9, 0, 0, 0)
    Clock clock = Clock.fixed(dateTime.toInstant(ZoneOffset.ofHours(9)),
                    ZoneId.systemDefault());
    when(clockFactory.fixed()).thenReturn(clock); // (8)

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
     - | clockFactory.fixed()の返り値を2012/10/1とする。
   * - | (7)
     - | cancelを実行し、キャンセル可能な日付より前なので、キャンセルが成功する。
   * - | (8)
     - | clockFactory.fixed()の返り値を2012/10/9とする。
   * - | (9)
     - | cancel実行し、キャンセル可能な日付より後なので、キャンセルが失敗する。

|

Integration Test
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Integration Testでは、システム連携先と疎通・連携確認のために1日の間に何日分ものデータ（例えばファイル）を作成して受け渡しを行う場合がある。

.. figure:: ./images_SystemDate/IntegrationTest.png
  :alt: IntegrationTest
  :width: 90%

実際の日付が2012/10/1の場合、\ ``JdbcAdjustedJodaTimeDateFactory``\ を使用し、試験対象の日付との差分を計算するSQLを設定する。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | 1
    - | 9:00-11:00の間は差分値を"0 days"とし、Clock Factoryの返り値を2012/10/1とする。
  * - | 2
    - | 11:00-13:00の間は差分値を"9 days"とし、Clock Factoryの返り値を2012/10/10とする。
  * - | 3
    - | 13:00-15:00の間は差分値を"30 days"とし、Clock Factoryの返り値を2012/10/31とする。
  * - | 4
    - | 15:00-17:00の間は差分値を"31 days"とし、Clock Factoryの返り値を2012/11/1とする。

テーブルの値を変更するのみで、日付を変更することが可能である。

|

System Test
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

System Testでは運用日を想定してテストシナリオを作成し、試験を実施することがある。

.. figure:: ./images_SystemDate/SystemTest.png
  :alt: SystemTest
  :width: 90%

| \ ``JdbcAdjustClockFactory``\ を使用し日付差を計算するSQLを設定する。
| 図中の1、2、3、4のように実際の日付と運用日の対応表を作成する。
| テーブルの差分値を変更するのみで、思い通りの日付でテストすることが可能となる。

|

Production
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| \ ``JdbcAdjustClockFactory``\ を使用し差分値を0とすることで、ソースを変更せずCkick Factoryの返り値を実際の日付と同じにできる。bean定義ファイルもSystem Testの時から変更を必要としない。
| 日時を変更する必要が生じても、テーブルの値を変更することでClock Factoryの返り値を変更することができる。

.. warning::

  Production環境で使用する場合は、production環境で使用するテーブルの差分値が0となっていることを確認すること。

  \ **設定例**\

  - production環境で初めてテーブルを使用する場合
    - INSERT INTO operation_date (diff) VALUES (0);
  - production環境で試験実施済みの場合
    - UPDATE operation_date SET diff=0;

  を実行すること。

|

時間を変更することがない場合は、\ ``DefaultClockFactory``\ に設定ファイルを変更することを推奨する。

.. raw:: latex

  \newpage

