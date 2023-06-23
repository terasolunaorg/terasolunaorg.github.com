日付操作(Joda Time)
--------------------------------------------------------------------------------

.. only:: html

 .. contents:: 目次
    :depth: 4
    :local:

|

Overview
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| ``java.util.Date`` 、 ``java.util.Calendar`` クラスのAPIは、非常に貧弱であるため、複雑な日付計算ができない。
| 本ガイドラインでは、日付計算が強力なJoda Timeの使用を推奨している。

| Joda Timeでは、 ``java.util.Date`` の代わりに、 ``org.joda.time.DateTime`` 、 ``org.joda.time.LocalDate`` や ``org.joda.time.LocalTime`` オブジェクトを用いて日付を表現する。
| なお、 ``org.joda.time.DateTime`` 、 ``org.joda.time.LocalDate`` や ``org.joda.time.LocalTime`` オブジェクトは、immutableである(日付計算等の結果は、新規オブジェクトである)。

|

How to use
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Joda Time, Joda Time JSP tags の利用方法を、以下で説明する。

日付取得
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

現在時刻を取得
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| 利用用途に併せて、 ``org.joda.time.DateTime`` , ``org.joda.time.LocalDate`` , ``org.joda.time.LocalTime`` を使い分けること。以下に、使用方法を示す。

1. ミリ秒まで取得したい場合は、 ``org.joda.time.DateTime`` を使用する。

.. code-block:: java

   DateTime dateTime = new DateTime();

2. TimeZoneと、時間を除いた日付だけが必要な場合は、 ``org.joda.time.LocalDate`` を使用する。

.. code-block:: java

   LocalDate localDate = new LocalDate();

3. TimeZoneと、日付を除いた時間だけが必要な場合は、 ``org.joda.time.LocalTime`` を使用する。

.. code-block:: java

   LocalTime localTime = new LocalTime();

4. 日付開始時刻と現在日付を取得したい場合は、 ``org.joda.time.DateTime.withTimeAtStartOfDay()`` を使用する。

.. code-block:: java

   DateTime dateTimeAtStartOfDay = new DateTime().withTimeAtStartOfDay();

|

    .. note::

        LocalDateとLocalTimeは、TimeZone情報を持たない。

    .. note::

        実際ServiceやControllerで現在時刻を取得するときのDateTime, LocalDate や、 LocalTimeのインスタンス取得には、
        \ ``org.terasoluna.gfw.common.date.jodatime.JodaTimeDateFactory``\を利用することを推奨する。

            .. code-block:: java

                DateTime dateTime = dataFactory.newDateTime();

        DateFactoryの利用方法は、 :doc:`./SystemDate` を参照されたい。

        LocalDateやLocalTimeの生成は

            .. code-block:: java

                LocalDate localDate = dataFactory.newDateTime().toLocalDate();
                LocalTime localTime = dataFactory.newDateTime().toLocalTime();


        とすればよい。

|

タイムゾーンを指定して現在時刻を取得
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| \ ``org.joda.time.DateTimeZone``\ は、timezoneを表すクラスである。
| Timezoneを指定して取得したい場合に使用する。以下に、使用方法を示す。

.. code-block:: java

    DateTime dateTime = new DateTime(DateTimeZone.forID("Asia/Tokyo"));


\ ``org.terasoluna.gfw.common.date.jodatime.JodaTimeDateFactory``\を利用する場合は、以下のようになる。

.. code-block:: java

    // Fetching current system date using default TimeZone
    DateTime dateTime = dataFactory.newDateTime();

    // Changing to TimeZone of Tokyo
    DateTime dateTimeTokyo = dateTime.withZone(DateTimeZone.forID("Asia/Tokyo"));


他の使用可能なTimezone ID文字列の一覧は、 `Available Time Zones <http://joda-time.sourceforge.net/timezones.html>`_ を参照されたい。


|

タイムゾーンを指定せず現在時刻を取得
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| タイムゾーンを指定せず現在時刻を取得したい場合に使用する。以下に、使用方法を示す。

.. code-block:: java

    LocalDateTime localDateTime = new LocalDateTime();

\ ``org.terasoluna.gfw.common.date.jodatime.JodaTimeDateFactory``\ を利用する場合は、以下のようになる。

.. code-block:: java

    // Fetching current system date using default TimeZone
    LocalDateTime localDateTime = dateFactory.newDateTime().toLocalDateTime();

|

    .. note::

        TimeZoneを意識する必要がない場合は、\ ``DateTime``\ ではなく\ ``LocalDateTime``\ を利用することを推奨する。

|


年月日時分秒を指定して取得
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
コンストラクタで、特定の時間を指定することができる。以下に例を示す。

* ミリ秒まで指定して、DateTimeを取得したい場合

.. code-block:: java

    DateTime dateTime = new DateTime(year, month, day, hour, minite, second, millisecond);

* 年月日を指定して、LocalDateを取得したい場合

.. code-block:: java

    LocalDate localDate = new LocalDate(year, month, day);

* 時分秒を指定して、LocalDate取得したい場合

.. code-block:: java

    LocalTime localTime = new LocalTime(hour, minutes, seconds, milliseconds);

|

年月日等の個別取得
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| DateTimeでは、年、月などを取得するメソッドを用意している。以下に、利用例を示す。

.. code-block:: java

    DateTime dateTime = new DateTime(2013, 1, 10, 2, 30, 22, 123);

    int year = dateTime.getYear();  // (1)
    int month = dateTime.getMonthOfYear();  // (2)
    int day = dateTime.getDayOfMonth();  // (3)
    int week = dateTime.getDayOfWeek();  // (4)
    int hour = dateTime.getHourOfDay();  // (5)
    int min = dateTime.getMinuteOfHour();  // (6)
    int sec = dateTime.getSecondOfMinute();  // (7)
    int millis = dateTime.getMillisOfSecond();  // (8)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 年を取得する。本例では、\ ``2013``\ が返却される。
   * - | (2)
     - | 月を取得する。本例では、\ ``1``\ が返却される。
   * - | (3)
     - | 日を取得する。本例では、\ ``10``\ が返却される。
   * - | (4)
     - | 曜日を取得する。本例では、\ ``4``\ が返却される。
       | 返却される値と曜日の対応は、[1:月曜、2:火曜、3:水曜、4:木曜、5:金曜、6:土曜、7:日曜]となる。
   * - | (5)
     - | 時を取得する。本例では、\ ``2``\ が返却される。
   * - | (6)
     - | 分を取得する。本例では、\ ``30``\ が返却される。
   * - | (7)
     - | 秒を取得する。本例では、\ ``22``\ が返却される。
   * - | (8)
     - | ミリ秒を取得する。本例では、\ ``123``\ が返却される。

|

    .. note::

        ``java.util.Calendar`` の仕様とは異なり、getDayOfMonth()は、1始まりである。

|

型変換
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

java.util.Dateとの相互運用性
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| DateTimeでは、 ``java.util.Date`` との型変換を、容易に行える。

.. code-block:: java

    Date date = new Date();

    DateTime dateTime = new DateTime(date);  // (1)

    Date convertDate = dateTime.toDate();  // (2)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | DateTimeのコンストラクタの引数に、 ``java.util.Date`` を引数に渡すことで、 ``java.util.Date`` -> DateTime への変換を行う。
   * - | (2)
     - | DateTime#toDate メソッドで、DateTime -> ``java.util.Date`` への変換を行う。

|

文字列へのフォーマット
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

.. code-block:: java

    DateTime dateTime = new DateTime();

    dateTime.toString("yyyy-MM-dd HH:mm:ss");  // (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | "yyyy-MM-dd HH:mm:ss" 形式で変換された、文字列が取得される。
       | toStringの引数として指定可能な値については、 `Input and Output <http://www.joda.org/joda-time/userguide.html#Input_and_Output>`_ を参照されたい。

|

文字列からのパース
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

.. code-block:: java

    LocalDate localDate = DateTimeFormat.forPattern("yyyy-MM-dd").parseLocalDate("2012-08-09");  // (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | "yyyy-MM-dd" 形式の文字列を、LocalDate型に変換する。
       | DateTimeFormat#forPatternの引数として指定可能な値は、 `Formatters <http://www.joda.org/joda-time/userguide.html#Input_and_Output>`_ を参照されたい。

|

日付操作
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

日付の計算
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| LocalDateには、日付の加減算を行うメソッドが用意されている。以下に、利用例を示す。

.. code-block:: java

    LocalDate localDate = new LocalDate(); // localDate is 2013-01-10
    LocalDate yesterday = localDate.minusDays(1);  // (1)
    LocalDate tomorrow = localDate.plusDays(1);  // (2)
    LocalDate afterThreeMonth = localDate.plusMonths(3);  // (3)
    LocalDate nextYear = localDate.plusYears(1);  // (4)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | LocalDate#minusDays 引数に、指定した値分の日付が減算される。本例では\ ``2013-01-09``\となる。
   * - | (2)
     - | LocalDate#plusDays 引数に、指定した値分の日付が加算される。本例では\ ``2013-01-11``\となる。
   * - | (3)
     - | LocalDate#plusMonths 引数に、指定した値分の月数が加算される。本例では\ ``2013-04-10``\となる。
   * - | (4)
     - | LocalDate#plusYears 引数に、指定した値分の年数が加算される。本例では\ ``2014-01-10``\となる。

上記で示したメソッド以外は、 `LocalDate JavaDoc <http://joda-time.sourceforge.net/apidocs/org/joda/time/LocalDate.html>`_ を参照されたい。

|

月末月初の取得
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| 現在日時を基準日とした、月末日と月初日の取得方法を、以下に示す。

.. code-block:: java

    LocalDate localDate = new LocalDate(); // dateTime is 2013-01-10
    Property dayOfMonth = localDate.dayOfMonth(); // (1)
    LocalDate firstDayOfMonth = dayOfMonth.withMinimumValue(); // (2)
    LocalDate lastDayOfMonth = dayOfMonth.withMaximumValue(); // (3)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 現在月の日付に関する属性値を保持するPropertyオブジェクトを取得する。
   * - | (2)
     - | Propertyオブジェクトから最小値を取得する事で、月初日を取得する事ができる。本例では\ ``2013-01-01``\となる。
   * - | (3)
     - | Propertyオブジェクトから最大値を取得する事で、月末日を取得する事ができる。本例では\ ``2013-01-31``\となる。

|

週末週初の取得
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| 現在日時を基準日とした、週末日と週初日の取得方法を、以下に示す。

.. code-block:: java

    LocalDate localDate = new LocalDate(); // dateTime is 2013-01-10
    Property dayOfWeek = localDate.dayOfWeek(); // (1)
    LocalDate firstDayOfWeek = dayOfWeek.withMinimumValue(); // (2)
    LocalDate lastDayOfWeek = dayOfWeek.withMaximumValue(); // (3)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 現在週の日付に関する属性値を保持するPropertyオブジェクトを取得する。
   * - | (2)
     - | Propertyオブジェクトから最小値を取得する事で、週初日(月曜日)を取得する事ができる。本例では\ ``2013-01-07``\となる。
   * - | (3)
     - | Propertyオブジェクトから最大値を取得する事で、週末日(日曜日)を取得する事ができる。本例では\ ``2013-01-13``\となる。


日時の比較
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
日時を比較して過去か未来を判定できる。

.. code-block:: java

  DateTime dt1 = new DateTime();
  DateTime dt2 = dt1.plusHours(1);
  DateTime dt3 = dt1.minusHours(1);


  System.out.println(dt1.isAfter(dt1)); // false
  System.out.println(dt1.isAfter(dt2)); // false
  System.out.println(dt1.isAfter(dt3)); // true
  
  System.out.println(dt1.isBefore(dt1)); // false
  System.out.println(dt1.isBefore(dt2)); // true
  System.out.println(dt1.isBefore(dt3)); // false
  
  System.out.println(dt1.isEqual(dt1)); // true
  System.out.println(dt1.isEqual(dt2)); // false
  System.out.println(dt1.isEqual(dt3)); // false


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``isAfter``\ メソッドは対象の日時が引数の日時より未来の場合に\ ``true``\ を返す。
   * - | (2)
     - | \ ``isBefore``\ メソッドは対象の日時が引数の日時より過去の場合に\ ``true``\ を返す。
   * - | (3)
     - | \ ``isEqual``\ メソッドは対象の日時が引数の日時と同じ場合に\ ``true``\ を返す。


期間の取得
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Joda-Timeでは、期間に関して、いくつかのクラスが提供されている。ここでは以下の2クラスについて説明する。

* ``org.joda.time.Interval``
* ``org.joda.time.Period``

Interval
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

2つのインスタンス（DateTime）の期間を表すクラス。

Intervalで調べられることは、以下4つである。

* 期間内に指定の日付や期間が含まれるかのチェック
* 2つの期間が連続するかのチェック
* 2つの期間の差を期間で取得
* 2つの期間の重なった期間を取得

実装例は、以下を参照されたい。

.. code-block:: java

    DateTime start1 = new DateTime(2013,8,14,0,0,0);
    DateTime end1 = new DateTime(2013,8,16,0,0,0);

    DateTime start2 = new DateTime(2013,8,16,0,0,0);
    DateTime end2 = new DateTime(2013,8,18,0,0,0);

    DateTime anyDate = new DateTime(2013, 8, 15, 0, 0, 0);

    Interval interval1 = new Interval(start1, end1);
    Interval interval2 = new Interval(start2, end2);

    interval1.contains(anyDate);  // (1)

    interval1.abuts(interval2);  // (2)

    DateTime start3 = new DateTime(2013,8,18,0,0,0);
    DateTime end3 = new DateTime(2013,8,20,0,0,0);
    Interval interval3 = new Interval(start3, end3);

    interval1.gap(interval3);  // (3)

    DateTime start4 = new DateTime(2013,8,15,0,0,0);
    DateTime end4 = new DateTime(2013,8,17,0,0,0);
    Interval interval4 = new Interval(start4, end4);

    interval1.overlap(interval4);  // (4)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | Interval#containsメソッドで、期間内に指定の日付や期間が含まれるかのチェックを行う。
       | 期間内に含まれる場合、"true"、含まれない場合、"false"を返却する。
   * - | (2)
     - | Interval#abutsメソッドで、2つの期間が連続するかのチェックを行う。
       | 2つの期間が連続する場合は"true"、連続しない場合は"false"を返却する。
   * - | (3)
     - | Interval#gapメソッドで、2つの期間の差を期間(Interval)で取得する。
       | 本例では、"2013-08-16～2013-08-18" の期間が取得される。
       | 期間の差が存在しない場合、nullが戻り値となる。
   * - | (4)
     - | Interval#overlapメソッドで、2つの期間の重なった期間(Interval)を取得する。
       | 本例では、"2013-08-15～2013-08-16" の期間が取得される。
       | 重なった期間が存在しない場合、nullが戻り値となる。

Interval同士を比較したい場合は、Periodに変換して行う。

* 月、日、などより抽象的な観点で比較をしたい場合は、Periodに変換すること。

.. code-block:: java


    // Convert to Period
    interval1.toPeriod();

|
|

Period
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Periodは、期間を、年、月、週などの単位で表すクラスである。

| たとえば、「3月1日」を表すInstant（DateTime）に「1ヶ月」に相当するPeriodを追加した場合、DateTimeは「4月1日」になる。
| 「3月1日」と「4月1日」に対して、「1か月」に相当するPeriodを追加した時の結果を以下に示す。

* 「3月1日」に「1ヶ月」というPeriodを追加したときの日数は「31日」
* 「4月1日」に「1ヶ月」というPeriodを追加したときの日数は「30日」

「1ヶ月」に相当するPeriodの追加は、対象のDateTimeによって、違う意味を持つ。

| Periodは、さらに2種類の実装が用意されている。

* Single field Period (例：「1日」や「1ヶ月」など一つの単位の値しか持たないタイプ)
* Any field Period (例：「1ヶ月2日4時間」など、複数の単位の値を持てて期間を表すタイプ)

詳細は、 `Period <http://joda-time.sourceforge.net/key_period.html>`_ を参照されたい。

|

JSP Tag Library
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| JSTLの fmt:formatDate タグは、java.util.Dateと、java.util.TimeZoneオブジェクトを扱う。
| Joda-timeのDateTime, LocalDateTime, LocalDate, LocalTimeと、DateTimeZoneオブジェクトを扱うためには、Jodaのタグライブラリを使う。
| 機能面でJSTLとほぼ同じであるため、JSTLの知識がある場合は、JodaのJSPタグライブラリを容易に使える。

|

設定方法
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

タブライブラリを利用するには、以下のtaglib定義が必要である。

.. code-block:: jsp

    <%@ taglib uri="http://www.joda.org/joda/time/tags" prefix="joda"%>

joda:format タグ
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

joda:format タグとは、DateTime, LocalDateTime, LocalDate, LocalTimeオブジェクトをフォーマットするタグである。

.. code-block:: jsp

    <% pageContext.setAttribute("now", new org.joda.time.DateTime()); %>

    <span>Using pattern="yyyyMMdd" to format the current system date</span><br/>
    <joda:format value="${now}" pattern="yyyyMMdd" />
    <br/>
    <span>Using style="SM" to format the current system date</span><br/>
    <joda:format value="${now}" style="SM" />

**出力結果**

.. figure:: images/joda_format_tag.png
   :alt: /jodatime
   :width: 55%

joda:formatタグの属性一覧は、以下の通りである。

.. tabularcolumns:: |p{0.05\linewidth}|p{0.10\linewidth}|p{0.85\linewidth}|
.. list-table:: **属性情報**
   :header-rows: 1
   :widths: 5 10 85

   * - No.
     - Attributes
     - Description
   * - 1.
     - | value
     - | ReadableInstantかReadablePartialのインスタンスを設定する。
   * - 2.
     - | var
     - | 時刻情報を持つ変数名
   * - 3.
     - | scope
     - | 時刻情報を持つ変数名のスコープ
   * - 4.
     - | locale
     - | ロケール情報
   * - 5.
     - | style
     - | フォーマットするためのスタイル情報（2桁。日付部分と時刻部分それぞれのスタイルを設定する。入力可能な値は S=Short, M=Medium, L=Long, F=Full, -=None）
   * - 6.
     - | pattern
     - | フォーマットするためのパターン（yyyyMMddなど）。入力可能なパターンは、 `Input and Output <http://www.joda.org/joda-time/userguide.html#Input_and_Output>`_ を参照されたい。
   * - 7.
     - | dateTimeZone
     - | タイムゾーン

Joda-Timeのほかのタグは、 `Joda Time JSP tags User guide <http://joda-time.sourceforge.net/contrib/jsptags/userguide.html>`_ を参照されたい。

    .. note::
        style属性を指定して日付と時刻部分を表示する場合、ブラウザのlocaleによって表示内容が異なる。
        上記style属性で表示した形式のlocaleは"en"である。

|

応用例(カレンダーの表示)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring MVCを使って、月単位のカレンダーを表示するサンプルを示す。

.. tabularcolumns:: |p{0.33\linewidth}|p{0.33\linewidth}|p{0.33\linewidth}|
.. list-table::
    :header-rows: 1

    * - 処理名
      - URL
      - ハンドラメソッド
    * - 今月のカレンダー表示
      - /calendar
      - today
    * - 指定月のカレンダー表示
      - /calendar/month?year=yyyy&month=m
      - month

コントローラの実装は、以下のようになる。

.. code-block:: java

    @Controller
    @RequestMapping("calendar")
    public class CalendarController {

        @RequestMapping
        public String today(Model model) {
            LocalDate today = new LocalDate();
            int year = today.getYear();
            int month = today.getMonthOfYear();
            return month(year, month, model);
        }

        @RequestMapping(value = "month")
        public String month(@RequestParam("year") int year,
                @RequestParam("month") int month, Model model) {
            LocalDate firstDayOfMonth = new LocalDate(year, month, 1);
            LocalDate lastDayOfMonth = firstDayOfMonth.dayOfMonth()
                    .withMaximumValue();

            LocalDate firstDayOfCalendar = firstDayOfMonth.dayOfWeek()
                    .withMinimumValue();
            LocalDate lastDayOfCalendar = lastDayOfMonth.dayOfWeek()
                    .withMaximumValue();

            List<List<LocalDate>> calendar = new ArrayList<List<LocalDate>>();
            List<LocalDate> weekList = null;
            for (int i = 0; i < 100; i++) {
                LocalDate d = firstDayOfCalendar.plusDays(i);
                if (d.isAfter(lastDayOfCalendar)) {
                    break;
                }

                if (weekList == null) {
                    weekList = new ArrayList<LocalDate>();
                    calendar.add(weekList);
                }

                if (d.isBefore(firstDayOfMonth) || d.isAfter(lastDayOfMonth)) {
                    // skip if the day is not in this month
                    weekList.add(null);
                } else {
                    weekList.add(d);
                }

                int week = d.getDayOfWeek();
                if (week == DateTimeConstants.SUNDAY) {
                    weekList = null;
                }
            }

            LocalDate nextMonth = firstDayOfMonth.plusMonths(1);
            LocalDate prevMonth = firstDayOfMonth.minusMonths(1);
            CalendarOutput output = new CalendarOutput();
            output.setCalendar(calendar);
            output.setFirstDayOfMonth(firstDayOfMonth);
            output.setYearOfNextMonth(nextMonth.getYear());
            output.setMonthOfNextMonth(nextMonth.getMonthOfYear());
            output.setYearOfPrevMonth(prevMonth.getYear());
            output.setMonthOfPrevMonth(prevMonth.getMonthOfYear());

            model.addAttribute("output", output);

            return "calendar";
        }
    }

以下の ``CalendarOutput`` クラスは、画面に出力する情報をまとめたJavaBeanである。


.. code-block:: java

    public class CalendarOutput {
        private List<List<LocalDate>> calendar;

        private LocalDate firstDayOfMonth;

        private int yearOfNextMonth;

        private int monthOfNextMonth;

        private int yearOfPrevMonth;

        private int monthOfPrevMonth;

        // omitted getter/setter
    }

|

    .. warning::

        このサンプルコードは単純なためControllerのハンドラメソッドに全ての処理を記述しているが、
        メンテナンス性向上のため本来この処理は、Helperクラスに記述すべきである。

|

JSP(calendar.jsp)で、次のように出力する。

 .. code-block:: jsp

    <p>
        <a
            href="${pageContext.request.contextPath}/calendar/month?year=${f:h(output.yearOfPrevMonth)}&month=${f:h(output.monthOfPrevMonth)}">&larr;
            Prev</a> <a
            href="${pageContext.request.contextPath}/calendar/month?year=${f:h(output.yearOfNextMonth)}&month=${f:h(output.monthOfNextMonth)}">Next
            &rarr;</a> <br>
        <joda:format value="${output.firstDayOfMonth}"
            pattern="yyyy-M" />
    </p>
    <table>
        <tr>
            <th>Mon.</th>
            <th>Tue.</th>
            <th>Wed.</th>
            <th>Thu.</th>
            <th>Fri.</th>
            <th>Sat.</th>
            <th>Sun.</th>
        </tr>
        <c:forEach var="week" items="${output.calendar}">
            <tr>
                <c:forEach var="day" items="${week}">
                    <td><c:choose>
                            <c:when test="${day != null}">
                                <joda:format value="${day}"
                                    pattern="d" />
                            </c:when>
                            <c:otherwise>&nbsp;</c:otherwise>
                        </c:choose></td>
                </c:forEach>
            </tr>
        </c:forEach>
    </table>

{contextPath}/calendarにアクセスすると、以下のカレンダーが表示される（2012年11月時点での結果である）。

.. figure:: images/calendar-today.jpg
   :alt: /calendar
   :width: 30%

{contextPath}/calendar/month?year=2012&month=12にアクセスすると、以下のカレンダーが表示される。

.. figure:: images/calendar-month.jpg
   :alt: /calendar/month?year=2012&month=12
   :width: 30%

.. raw:: latex

   \newpage

Appendix
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Java8未満の和暦操作
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Java8では ``java.time.chrono.JapaneseDate`` という和暦操作クラスが提供されているが、Java8未満の環境では ``java.util.Calendar`` クラスで和暦を扱うことが出来る。
| 具体的には、 ``java.util.Calendar`` クラス、 ``java.text.DateFormat`` クラスに以下の ``java.util.Locale`` を指定する必要がある。

.. code-block:: java

   Locale locale = new Locale("ja", "JP", "JP");

| 以下に、``Calendar`` クラスを利用した和暦表示の例を示す。

.. code-block:: java

   Locale locale = new Locale("ja", "JP", "JP");
   Calendar cal = Calendar.getInstance(locale); // Ex, 2015-06-05
   String format1 = "Gy.MM.dd";
   String format2 = "GGGGyy/MM/dd";

   DateFormat df1 = new SimpleDateFormat(format1, locale);
   DateFormat df2 = new SimpleDateFormat(format2, locale);

   df1.format(cal.getTime()); // "H27.06.05"
   df2.format(cal.getTime()); // "平成27/06/05"

| また、同様に文字列からのパースも行うことが出来る。

.. code-block:: java

   Locale locale = new Locale("ja", "JP", "JP");
   String format1 = "Gy.MM.dd";
   String format2 = "GGGGyy/MM/dd";
   
   DateFormat df1 = new SimpleDateFormat(format1, locale);
   DateFormat df2 = new SimpleDateFormat(format2, locale);
   
   Calendar cal1 = Calendar.getInstance(locale);
   Calendar cal2 = Calendar.getInstance(locale);

   cal1.setTime(df1.parse("H27.06.05"));
   cal2.setTime(df2.parse("平成27/06/05"));

|

    .. note::

        | ``new Locale("ja", "JP", "JP")`` を ``getInstance`` メソッドに指定することで、 和暦に対応した ``java.util.JapaneseImperialCalendar`` オブジェクトが作成される。
        | その他を指定すると ``java.util.GregorianCalendar`` オブジェクトが作成されるため、留意されたい。
