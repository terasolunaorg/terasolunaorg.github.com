日付操作(JSR-310 Date and Time API)
--------------------------------------------------------------------------------

.. only:: html

 .. contents:: 目次
    :depth: 4
    :local:

|

Overview
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| 本ガイドラインでは、``java.util.Date`` , ``java.util.Calender`` に比べて、
| 様々な日時計算が提供されている JSR-310 Date and Time API の使用を推奨する。

    .. note::

        JSR-310 Date and Time APIはJava8から導入されたため、
        Java8未満の環境は、Joda Timeの利用を推奨している。
        Joda Timeの利用方法は、 :doc:`./JodaTime` を参照されたい。


How to use
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| Date and Time APIでは、日付のみ扱うクラス、時刻のみ扱うクラスなど、用途に応じた様々なクラスが提供されている。
| 本ガイドラインでは、``java.time.LocalDate`` , ``java.time.LocalTime`` , ``java.time.LocalDateTime`` を中心に説明を進めるが、主要な日時操作については、各クラスで提供されるメソッドの接頭辞が同一であるため、適時クラス名を置き換えて解釈されたい。
| 主に使われるクラスとメソッドを示す。
|
| **日時を扱う主なクラス**

.. list-table::
   :header-rows: 1
   :widths: 30 35 35

   * - クラス名
     - 説明
     - 主なファクトリメソッド
   * - | `java.time.LocalDate <https://docs.oracle.com/javase/8/docs/api/java/time/LocalDate.html>`_
       | `java.time.LocalTime <https://docs.oracle.com/javase/8/docs/api/java/time/LocalTime.html>`_
       | `java.time.LocalDateTime <https://docs.oracle.com/javase/8/docs/api/java/time/LocalDateTime.html>`_
     - タイムゾーン・時差の情報を持たない日付・時刻の操作を行うクラス
     - | ``now`` 現在日時で生成
       | ``of``  任意日時で生成
       | ``parse`` 日時文字列から生成
       | ``from``  日時情報を持つ他オブジェクトから生成
   * - | `java.time.OffsetTime <https://docs.oracle.com/javase/8/docs/api/java/time/OffsetTime.html>`_
       | `java.time.OffsetDateTime <https://docs.oracle.com/javase/8/docs/api/java/time/OffsetDateTime.html>`_
       | `java.time.ZonedDateTime <https://docs.oracle.com/javase/8/docs/api/java/time/ZonedDateTime.html>`_
     - タイムゾーン・時差を考慮した日付・時刻の操作を行うクラス
     - 同上
   * - | `java.time.chrono.JapaneseDate <https://docs.oracle.com/javase/8/docs/api/java/time/chrono/JapaneseDate.html>`_
     - 和暦の操作を行うクラス
     - 同上
     
|
| **期間の情報を扱う主なクラス**

.. list-table::
   :header-rows: 1
   :widths: 30 35 35
   
   * - クラス名
     - 説明
     - 主なファクトリメソッド
   * - | `java.time.Period <https://docs.oracle.com/javase/8/docs/api/java/time/Period.html>`_
       | `java.time.Duration <https://docs.oracle.com/javase/8/docs/api/java/time/Duration.html>`_
     - 日時ベース、時間ベースの期間を扱うクラス
     - | ``between`` 日時情報を持つ2つのオブジェクトの差から生成
       
       | ``from`` 時間量を持つ他オブジェクトから生成
       
       | ``of`` 任意期間で生成

|
| **フォーマットを扱うクラス**

.. list-table::
   :header-rows: 1
   :widths: 30 35 35
   
   * - クラス名
     - 説明
     - 主なファクトリメソッド
   * - | `java.time.format.DateTimeFormatter <https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html>`_
     - 日時のフォーマットに関する操作を行うクラス
     - | ``ofPattern`` 指定されたパターンでフォーマッタを生成


|
| 各クラス・メソッドの具体的な利用方法を、以下で説明する。
|

    .. note::

        本ガイドラインで触れなかった内容を含め、詳細は `Javadoc <https://docs.oracle.com/javase/8/docs/api/java/time/package-summary.html>`_ を参照されたい。


    .. note::

         Date and Time APIのクラスは、immutableである(日時計算等の結果は、新規オブジェクトであり、計算元オブジェクトに変化は起きない)。

日時取得
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

現在日時で取得
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| 利用用途に合わせて、 ``java.time.LocalTime`` , ``java.time.LocalDate`` , ``java.time.LocalDateTime``  を使い分けること。以下に例を示す。

1. 時刻のみ取得したい場合は、 ``java.time.LocalTime`` を使用する。

.. code-block:: java

   LocalTime localTime =  LocalTime.now();

2. 日付のみ取得したい場合は ``java.time.LocalDate`` を使用する。

.. code-block:: java

   LocalDate localDate =  LocalDate.now();

3. 日付・時刻を取得したい場合は、、 ``java.time.LocalDateTime`` を使用する。

.. code-block:: java

   LocalDateTime localDateTime = LocalDateTime.now();


|


年月日時分秒を指定して取得
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| ofメソッドを使うことで、特定の日時を指定することができる。以下に例を示す。

1. 時刻を指定して、 ``java.time.LocalTime`` を取得する。

.. code-block:: java

   // 23:30:59
   LocalTime localTime =  LocalTime.of(23, 30, 59);

2. 日付を指定して ``java.time.LocalDate`` を取得する。

.. code-block:: java

   // 2015/12/25
   LocalDate localDate =  LocalDate.of(2015, 12, 25);

3. 日付・時刻）を指定して、 ``java.time.LocalDateTime`` を取得する。

.. code-block:: java

   // 2015/12/25 23:30:59
   LocalDateTime localDateTime = LocalDateTime.of(2015, 12, 25, 23, 30, 59);

|
| また、``java.time.temporal.TemporalAdjusters`` を使うことで様々な日時を取得することができる。

.. code-block:: java

   // LeapYear(2012/2)
   LocalDate localDate1 = LocalDate.of(2012, 2, 1);
   
   // Last day of month(2012/2/29)
   LocalDate localDate2 = localDate1.with(TemporalAdjusters.lastDayOfMonth());
   
   // Next monday（2012/2/6）
   LocalDate localDate3 = localDate1.with(TemporalAdjusters.next(DayOfWeek.MONDAY));


.. note::

    ``java.util.Calendar`` の仕様とは異なり、Monthは 1 始まりである。


タイムゾーンを指定する場合の日時取得
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| 国際的なアプリケーションを作成する場合、タイムゾーンを意識した設計を行う場合がある。
| Date and Time APIでは、利用用途に合わせて、 ``java.time.OffsetTime`` , ``java.time.OffsetDateTime`` , ``java.time.ZonedDateTime``  を使い分けること。
| 以下に例を示す。

1. 時刻＋UTCとの時差を取得したい場合は、 ``java.time.OffsetTime`` を使用する。

.. code-block:: java

   // Ex, 12:30:11.567+09:00
   OffsetTime offsetTime =  OffsetTime.now();

2. 日付・時刻＋UTCとの時差を取得したい場合は、 ``java.time.OffsetDateTime`` を使用する。

.. code-block:: java

   // Ex, 2015-12-25T12:30:11.567+09:00
   OffsetDateTime offsetDateTime =  OffsetDateTime.now();

3. 日付・時刻＋UTCとの時差・地域を取得したい場合は、 ``java.time.ZonedDateTime`` を使用する。

.. code-block:: java

   // Ex, 2015-12-25T12:30:11.567+09:00[Asia/Tokyo]
   ZonedDateTime zonedDateTime = ZonedDateTime.now();

| また、これらのメソッドでは全て、タイムゾーンを表す ``java.time.ZoneId`` を引数に設定することで、タイムゾーンを考慮した現在日時が取得できる。
| 以下に ``java.time.ZoneId`` の例を示す。

.. code-block:: java

   ZoneId zoneIdTokyo = ZoneId.of("Asia/Tokyo");
   OffsetTime offsetTime =  OffsetTime.now(zoneIdTokyo);

| なお、 ``java.time.ZoneId`` は地域名/地名形式で定義する方法や、UTCからの時差で定義する方法がある。

.. code-block:: java

   ZoneId.of("Asia/Tokyo");
   ZoneId.of("UTC+01:00");
   
|

| ``java.time.OffsetDateTime`` , ``java.time.ZonedDateTime`` の2クラスは用途が似ているが、具体的には以下のような違いがある。
| 作成するシステムの特性に応じて適切なクラスを選択されたい。

.. list-table::
   :header-rows: 1
   :widths: 50 50
   
   * - クラス名
     - 説明
   * - | ``java.time.OffsetDateTime``
     - 定量値（時差のみ）を持つため、各地域の時間の概念に変化がある場合も、システムに変化が起こらない。
   * - | ``java.time.ZonedDateTime``
     - 時差に加えて地域の概念があるため、各地域の時間の概念に変化があった場合、システムに変化が起こる。（政策としてサマータイム導入される場合など）

期間
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

期間の取得
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| 日付ベースの期間を扱う場合は、 ``java.time.Period`` 、時間ベースの期間を扱う場合は、 ``java.time.Duration`` を使用する。
| ``java.time.Duration`` で表される1日は厳密に24時間であるため、サマータイムの変化が解釈されずに想定通りの結果にならない可能性がある。
| 対して、 ``java.time.Period`` はサマータイムなどの概念を考慮した1日を表すため、サマータイムを扱うシステムであっても誤差は生じない。
| 以下に例を示す。

.. code-block:: java

   LocalDate date1 = LocalDate.of(2010, 01, 15);
   LocalDate date2 = LocalDate.of(2011, 03, 18);
   LocalTime time1 = LocalTime.of(11, 50, 50);
   LocalTime time2 = LocalTime.of(12, 52, 53);
   
   // One year, two months and three days.
   Period pd = Period.between(date1, date2);
   
   // One hour, two minutes and three seconds.
   Duration dn = Duration.between(time1, time2); 

|

    .. note::

        ``of`` メソッドを利用して、期間を指定して生成する方法もある。詳細は `Period, DurationのJavadoc <https://docs.oracle.com/javase/8/docs/api/java/time/package-summary.html>`_ を参照されたい。

型変換
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Date and Time APIの各クラスの相互運用性
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| ``java.time.LocalTime`` , ``java.time.LocalDate`` , ``java.time.LocalDateTime`` はそれぞれ容易に変換が可能である。以下に例を示す。

1. ``java.time.LocalTime`` から、 ``java.time.LocalDateTime`` への変換。

.. code-block:: java

   // Ex. 12:10:30
   LocalTime localTime =  LocalTime.now();
   
   // 2015-12-25 12:10:30
   LocalDateTime localDateTime = localTime.atDate(LocalDate.of(2015, 12, 25));

2. ``java.time.LocalDate`` から、 ``java.time.LocalDateTime`` への変換。

.. code-block:: java

   // Ex. 2012-12-25
   LocalDate localDate =  LocalDate.now();
   
   // 2015-12-25 12:10:30
   LocalDateTime localDateTime = localDate.atTime(LocalTime.of(12, 10, 30));

3. ``java.time.LocalDateTime`` から、 ``java.time.LocalTime`` ,  ``java.time.LocalDate`` への変換。

.. code-block:: java

   // Ex. 2015-12-25 12:10:30
   LocalDateTime localDateTime =  LocalDateTime.now();
   
   // 12:10:30
   LocalTime localTime =  localDateTime.toLocalTime();
   
   // 2012-12-25
   LocalDate localDate =  localDateTime.toLocalDate();
   
|
| 同様に、``java.time.OffsetTime`` , ``java.time.OffsetDateTime`` , ``java.time.ZonedDateTime`` もそれぞれ容易に変換が可能である。以下に例を示す。

1. ``java.time.OffsetTime`` から、 ``java.time.OffsetDateTime`` への変換。

.. code-block:: java

   // Ex, 12:30:11.567+09:00
   OffsetTime offsetTime =  OffsetTime.now();
   
   // 2015-12-25T12:30:11.567+09:00
   OffsetDateTime OffsetDateTime = offsetTime.atDate(LocalDate.of(2015, 12, 25));

2. ``java.time.OffsetDateTime`` から、 ``java.time.ZonedDateTime`` への変換。

.. code-block:: java

   // Ex, 2015-12-25T12:30:11.567+09:00
   OffsetDateTime offsetDateTime =  OffsetDateTime.now();
   
   // 2015-12-25T12:30:11.567+09:00[Asia/Tokyo]
   ZonedDateTime zonedDateTime = offsetDateTime.atZoneSameInstant(ZoneId.of("Asia/Tokyo"));

3. ``java.time.ZonedDateTime`` から、 ``java.time.OffsetDateTime`` ,  ``java.time.OffsetTime`` への変換。

.. code-block:: java

   // Ex, 2015-12-25T12:30:11.567+09:00[Asia/Tokyo]
   ZonedDateTime zonedDateTime =  ZonedDateTime.now();
   
   // 2015-12-25T12:30:11.567+09:00
   OffsetDateTime offsetDateTime =  zonedDateTime.toOffsetDateTime();
   
   // 12:30:11.567+09:00
   OffsetTime offsetTime =  zonedDateTime.toOffsetDateTime().toOffsetTime();
   
|
| また、時差情報を加えることで、``java.time.LocalTime`` を ``java.time.OffsetTime`` に変換することも可能である。

.. code-block:: java

   // Ex, 12:30:11.567
   LocalTime localTime =  LocalTime.now();
   
   // 12:30:11.567+09:00
   OffsetTime offsetTime = localTime.atOffset(ZoneOffset.ofHours(9));

|
| この他についても、不足している情報（ ``LocalTime`` から ``LocalDateTime`` の変換であれば日付の情報が不足している の要領）を加えることで別のクラスへ変換が可能である。
| 変換メソッドは接頭辞が ``at`` または ``to`` で始まる。詳細は `各クラスのJavadoc <https://docs.oracle.com/javase/8/docs/api/java/time/package-summary.html>`_ を参照されたい。

java.util.Dateとの相互運用性
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

``java.time.LocalDate`` 等のクラスを、``java.util.Date`` に直接変換するメソッドは提供されていない。

| ただし、Java8からは ``java.util.Date`` にDate and Time APIが提供する  ``java.time.Instant`` を変換するメソッドが追加されているため、``java.time.Instant`` を経由して変換を行うことが可能である。
| 以下に例を示す。

1. ``java.time.LocalDateTime`` から、 ``java.util.Date`` への変換。

.. code-block:: java

   LocalDateTime localDateTime = LocalDateTime.now();
   Instant instant = localDateTime.toInstant(ZoneOffset.ofHours(9));
   Date date = Date.from(instant);

2. ``java.util.Date`` から、 ``java.time.LocalDateTime`` への変換。

.. code-block:: java

   Date date = new Date();
   Instant instant = date.toInstant();
   LocalDateTime localDateTime = LocalDateTime.ofInstant(instant, ZoneId.systemDefault());

|

    .. note::

        ``java.time.LocalTime`` , ``java.time.LocalDate`` はInstant値を持たないため、一度 ``java.time.LocalDateTime`` に変換する必要がある。

java.sql パッケージとの相互運用性
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| Java8 より ``java.sql`` パッケージに改修が入り、 ``java.time`` パッケージとの相互変換メソッドが定義された。
| 以下に例を示す。

1. ``java.sql.Date`` から、 ``java.time.LocalDate`` への変換。

.. code-block:: java

   java.sql.Date date =  new java.sql.Date(System.currentTimeMillis());
   LocalDate localDate = date.toLocalDate();

2. ``java.time.LocalDate`` から、 ``java.sql.Date`` への変換。

.. code-block:: java

   LocalDate localDate = LocalDate.now();
   java.sql.Date date =  java.sql.Date.valueOf(localDate);
   
3. ``java.sql.Time`` から、 ``java.time.LocalTime`` への変換。

.. code-block:: java

   java.sql.Time time =  new java.sql.Time(System.currentTimeMillis());
   LocalTime localTime = time.toLocalTime();

4. ``java.time.LocalTime`` から、 ``java.sql.Time`` への変換。

.. code-block:: java

   LocalTime localTime = LocalTime.now();
   java.sql.Time time =  java.sql.Time.valueOf(localTime);


5. ``java.sql.Timestamp`` から、 ``java.time.LocalDateTime`` への変換。

.. code-block:: java

   java.sql.Timestamp timestamp =  new java.sql.Timestamp(System.currentTimeMillis());
   LocalDateTime localDateTime = timestamp.toLocalDateTime();

6. ``java.time.LocalDateTime`` から、 ``java.sql.Timestamp`` への変換。

.. code-block:: java

   LocalDateTime localDateTime = LocalDateTime.now();
   java.sql.Timestamp timestamp =  java.sql.Timestamp.valueOf(localDateTime);

org.terasoluna.gfw.common.date パッケージの利用方法
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| 現在、Date and Time API用のDate Factoryは共通ライブラリから提供されていない。（参照： :doc:`../SystemDate` ）
| ただし、暫定対処として、 ``org.terasoluna.gfw.common.date.ClassicDateFactory`` と ``java.sql.Date`` を利用することで、 ``java.time.LocalDate`` を生成できる。
| ``java.time.LocalTime`` , ``java.time.LocalDateTime`` クラスに関しても、 ``java.time.LocalDate`` から変換することで生成できる。
| 以下に例を示す。

**bean定義ファイル([projectname]-env.xml)**

.. code-block:: xml

    <bean id="dateFactory" class="org.terasoluna.gfw.common.date.DefaultClassicDateFactory" />

**Javaクラス**

.. code-block:: java

   @Inject
   ClassicDateFactory dateFactory;
   
   public DateFactorySample getSystemDate() {

    java.sql.Date date = dateFactory.newSqlDate();
    LocalDate localDate = date.toLocalDate();

    // omitted
   }
   
|

    .. note::

        Date and Time APIに対応したDate Factoryは今後追加予定である。


文字列へのフォーマット
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| 日時情報を持つオブジェクトを文字列に変換するには、``toString`` メソッドを使用する方法と、``java.time.fomat.DateTimeFormatter`` を使用する方法がある。
| 任意の日時文字列で出力したい場合は、``java.time.fomat.DateTimeFormatter`` を使用し様々な日時文字列へ変換することが出来る。
|
| ``java.time.fomat.DateTimeFormatter`` は、事前定義されたISOパターンのフォーマッタを利用する方法と、任意のパターンのフォーマットを定義して利用する方法がある。

.. code-block:: java

   DateTimeFormatter formatter1 = DateTimeFormatter.BASIC_ISO_DATE;
                                             
   DateTimeFormatter formatter2 = DateTimeFormatter.ofPattern("G uuuu/MM/dd E")
                                             .withLocale(Locale.JAPANESE)
                                             .withResolverStyle(ResolverStyle.STRICT);

| この際、文字列の形式の他に ``Locale`` と ``ResolverStyle`` （厳密性）を定義できる。
| ``Locale`` のデフォルト値はシステムによって変化するため、初期化時に設定することが望ましい。
| また、 ``ResolverStyle`` （厳密性）は ``ofPattern`` メソッドを使う場合、デフォルトで ``ResolverStyle.SMART`` が設定されるが、本ガイドラインでは予期せぬ挙動が起こらないよう、厳密に日付を解釈する ``ResolverStyle.STRICT`` の設定を推奨している。（ISOパターンのフォーマッタを利用する場合は ``ResolverStyle.STRICT`` が設定されている)
|
| なお、Date and Time API では書式 ``yyyy`` は暦に対する年を表すため、暦によって解釈が異なる(西暦なら2015と解釈されるが、和暦なら0027と解釈される）。
| 西暦を表したい場合は、 ``yyyy`` 形式に変えて ``uuuu`` 形式を利用することを推奨する。定義されている書式一覧は `DateTimeFormatter <http://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#patterns>`_ を参照されたい。
|
| 以下に例を示す。


.. code-block:: java

   DateTimeFormatter formatter1 = DateTimeFormatter.BASIC_ISO_DATE;
                                             
   DateTimeFormatter formatter2 = DateTimeFormatter.ofPattern("G uuuu/MM/dd E")
                                             .withLocale(Locale.JAPANESE)
                                             .withResolverStyle(ResolverStyle.STRICT);
      
   LocalDate localDate1 = LocalDate.of(2015, 12, 25);
   
   // "2015-12-25"
   System.out.println(localDate1.toString()); 
   // "20151225"
   System.out.println(formatter1.format(localDate1));
   // "西暦 2015/12/25 金"
   System.out.println(formatter2.format(localDate1));

|
| また、これらの文字列を画面上に表示したい場合、
| Date and Time APIでは、Joda Timeと異なり、専用のJSPタグは存在していない。
| JSTLの ``fmt:formatDate`` タグは、``java.util.Date`` と、 ``java.util.TimeZone`` オブジェクトのみを扱うため、
| JSP上でDate and Time APIのオブジェクトが持つ日時情報を表示する場合は、フォーマット済みの文字列を渡して表示する。

**Controllerクラス**

.. code-block:: java

  @Controller
  public class HomeController {

      @RequestMapping(value = "/", method = {RequestMethod.GET, RequestMethod.POST})
      public String home(Model model, Locale locale) {
      
          DateTimeFormatter dateFormatter = DateTimeFormatter.ofPattern("uuuu/MM/dd")
                                             .withLocale(locale)
                                             .withResolverStyle(ResolverStyle.STRICT);
                                                       
          LocalDate localDate1 = LocalDate.now();

          model.addAttribute("currentDate", localDate1.toString());
          model.addAttribute("formattedCurrentDateString", dateFormatter.format(localDate1));

      // omitted

      }
  }
  
**jspファイル**

.. code-block:: jsp

  <p>currentDate =  ${f:h(currentDate)}.</p>
  <p>formattedCurrentDateString =  ${f:h(formattedCurrentDateString)}.</p>

**出力結果例(html)**

.. code-block:: html

  <p>currentDate =  2015-12-25.</p>
  <p>formattedCurrentDateString =  2015/12/25.</p>


文字列からのパース
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| 文字列への変換と同様に、``java.time.fomat.DateTimeFormatter`` を用いることで、様々な日付文字列をDate and Time APIのクラスへ変換することが出来る。
| 以下に例を示す。

.. code-block:: java

   DateTimeFormatter formatter1 = DateTimeFormatter.ofPattern("uuuu/MM/dd")
                                              .withLocale(Locale.JAPANESE)
                                              .withResolverStyle(ResolverStyle.STRICT);
   
   DateTimeFormatter formatter2 = DateTimeFormatter.ofPattern("HH:mm:ss")
                                              .withLocale(Locale.JAPANESE)
                                              .withResolverStyle(ResolverStyle.STRICT);

   LocalDate localDate = LocalDate.parse("2015/12/25", formatter1);
   LocalTime localTime = LocalDate.parse("14:09:20", formatter2);
 
|

日付操作
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Date and Time APIでは、日時の計算や比較などを容易に行うことが出来る。
| 以下に例を示す。




日時の計算
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''


| 日時の計算をするために、 ``plus`` メソッドと ``minus`` メソッドが提供されている。


1. 時間の計算を行う場合の例。

.. code-block:: java

   LocalTime localTime =  LocalTime.of(20, 30, 50);
   LocalTime plusHoursTime = localTime.plusHours(2);
   LocalTime plusMinutesTime = localTime.plusMinutes(10);
   LocalTime minusSecondsTime = localTime.minusSeconds(15);

2. 日付の計算を行う場合の例。

.. code-block:: java

   LocalDate localDate =  LocalDate.of(2015, 12, 25);
   LocalDate plusYearsDate = localDate.plusYears(10);
   LocalDate minusMonthsTime = localDate.minusMonths(1);
   LocalDate plusDaysTime = localDate.plusDays(3);


|

    .. note::

        ``plus`` メソッドに負数を渡すと、 ``minus`` メソッドを利用した場合と同様の結果が得られる。 ``minus`` メソッドも同様。



日時の比較
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| Date and Time APIでは、過去・未来・同時などの時系列の比較が行える。
| 以下に例を示す。

1. 時間の比較を行う場合の例。

.. code-block:: java

   LocalTime morning =  LocalTime.of(7, 30, 00);
   LocalTime daytime =  LocalTime.of(12, 00, 00);
   LocalTime evening =  LocalTime.of(17, 30, 00);
   
   daytime.isBefore(morning); // false
   morning.isAfter(evening); // true
   evening.equals(LocalTime.of(17, 30, 00)); // true
   
   daytime.isBefore(daytime); // false
   morning.isAfter(morning); // false

2. 日付の比較を行う場合の例。

.. code-block:: java

   LocalDate may =  LocalDate.of(2015, 6, 1);
   LocalDate june =  LocalDate.of(2015, 7, 1);
   LocalDate july =  LocalDate.of(2015, 8, 1);
   
   may.isBefore(june); // true
   june.isAfter(july); // false
   july.equals(may); // false
   
   may.isBefore(may); // false
   june.isAfter(june); // false
   
|
| なお、Date and Time APIでは現在、Joda Timeの ``Interval`` に当たるクラスは存在しない。


日時の判定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| 以下に日時の判定の例を示す。


1. 妥当な日時文字列かを判定したい場合、 ``java.time.format.DateTimeParseException`` の発生有無で判定できる。

.. code-block:: java

   String strDateTime = "aabbcc";
   DateTimeFormatter timeFormatter  = DateTimeFormatter.ofPattern("HHmmss")
                                 .withLocale(Locale.JAPANESE)
                                 .withResolverStyle(ResolverStyle.STRICT);;
                                 
   DateTimeFormatter dateFormatter  = DateTimeFormatter.ofPattern("uuMMdd")
                                 .withLocale(Locale.JAPANESE)
                                 .withResolverStyle(ResolverStyle.STRICT);;

   try {
       // DateTimeParseException
       LocalTime localTime = LocalTime.parse(strDateTime, timeFormatter);
   }
   catch (DateTimeParseException e) {
       System.out.println("Invalid time string !!");
   }
   
   try {
       // DateTimeParseException
       LocalDate localDate = LocalDate.parse(strDateTime, dateFormatter);
   }
   catch (DateTimeParseException e) {
       System.out.println("Invalid date string !!");
   }



2. うるう年かを判定したい場合、``java.time.LocalDate`` の ``isLeapYear`` メソッドで判定できる。

.. code-block:: java

   LocalDate date1 = LocalDate.of(2012, 1, 1);
   LocalDate date2 = LocalDate.of(2015, 1, 1);
   
   date1.isLeapYear(); // true
   date2.isLeapYear(); // false


年月日時分秒の取得
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| 年月日時分秒をそれぞれ取得したい場合は、 ``get`` メソッドを利用する。
| 以下に日付に関する情報を取得する例を示す。

.. code-block:: java

   LocalDate localDate = LocalDate.of(2015, 2, 1);
   
   // 2015
   int year = localDate.getYear();
   
   // 2
   int month = localDate.getMonthValue();
   
   // 1
   int dayOfMonth = localDate.getDayOfMonth();

   // 32 ( day of year )
   int dayOfYear = localDate.getDayOfYear();


和暦（JapaneseDate）
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Date and Time APIでは、``java.time.chrono.JapaneseDate`` という、和暦を扱うクラスが提供されている。

    .. note::

        ``java.time.chrono.JapaneseDate`` は、グレゴリオ暦が導入された明治6年(西暦1873年)より前は利用できない。

和暦の取得
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| ``java.time.LocalDate`` と同様に、 ``now`` メソッド,  ``of`` メソッドで取得できる。
| また、``java.time.chrono.JapaneseEra`` クラスを使うことで、和暦を指定した取得も行うことが出来る。

| 以下に例を示す。

.. code-block:: java

   JapaneseDate japaneseDate1 = JapaneseDate.now();
   JapaneseDate japaneseDate2 = JapaneseDate.of(2015, 12, 25); 
   JapaneseDate japaneseDate3 = JapaneseDate.of(JapaneseEra.HEISEI, 27, 12, 25); 

| 明治6年より前を引数に指定すると例外が発生する。

.. code-block:: java

   // DateTimeException
   JapaneseDate japaneseDate = JapaneseDate.of(1500, 1, 1);
  
文字列へのフォーマット
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| ``java.time.fomat.DateTimeFormatter`` を用いることで、和暦日付へ変換することが出来る。利用の際には、 ``DateTimeFormatter#withChronology`` メソッドで暦を ``java.time.chrono.JapaneseChronology`` に設定する。
| 和暦日付でも様々なフォーマットを扱うことが出来るため、0埋めや空白埋めなど要件に応じた出力が行える。
| 以下に空白埋めで和暦を表示する例を示す。

.. code-block:: java

   DateTimeFormatter formatter = DateTimeFormatter.ofPattern("Gppy年ppM月ppd日")
                                    .withLocale(Locale.JAPANESE)
                                    .withResolverStyle(ResolverStyle.STRICT)
                                    .withChronology(JapaneseChronology.INSTANCE);
                                              
   JapaneseDate japaneseDate = JapaneseDate.of(1992, 1, 1);
   
   // "平成 4年 1月 1日"
   System.out.println(formatter.format(japaneseDate));


文字列からのパース
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| ``java.time.fomat.DateTimeFormatter`` を用いることで、和暦文字列から ``java.time.chrono.JapaneseDate`` へ変換することが出来る。
| 以下に例を示す。

.. code-block:: java

   DateTimeFormatter formatter = DateTimeFormatter.ofPattern("Gy年MM月dd日")
                                   .withLocale(Locale.JAPANESE)
                                   .withResolverStyle(ResolverStyle.STRICT)
                                   .withChronology(JapaneseChronology.INSTANCE);
                                        
   JapaneseDate japaneseDate1 = JapaneseDate.from(formatter.parse("平成27年12月25日"));
   JapaneseDate japaneseDate2 = JapaneseDate.from(formatter.parse("明治6年01月01日"));


西暦・和暦の変換
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| fromメソッドを使うことで、``java.time.LocalDate`` からの変換を容易に行える。

.. code-block:: java

   LocalDate localDate = LocalDate.of(2015, 12, 25);
   JapaneseDate jpDate = JapaneseDate.from(localDate);


