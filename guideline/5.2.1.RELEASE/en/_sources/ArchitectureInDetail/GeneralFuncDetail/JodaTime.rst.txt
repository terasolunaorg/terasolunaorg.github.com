Date Operations (Joda Time)
--------------------------------------------------------------------------------

.. only:: html

 .. contents:: Table of Contents
    :depth: 4
    :local:

|

Overview
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| The API of ``java.util.Date`` , ``java.util.Calendar`` class is poorly built to perform complex date calculations.
| This guideline recommends the usage of Joda Time which provides quality replacement for the Java Date and Time classes.

| In Joda Time, date is expressed using ``org.joda.time.DateTime`` , ``org.joda.time.LocalDate`` , or ``org.joda.time.LocalTime`` object instead of ``java.util.Date``.
| ``org.joda.time.DateTime`` , ``org.joda.time.LocalDate`` , or ``org.joda.time.LocalTime`` object is immutable (the result of date related calculations, etc. is returned in a new object).

|

How to use
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The usage of Joda Time, Joda Time JSP tags is explained below.

Fetching date
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Fetching current time
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| As per the requirements, any of the following classes can be used. 
| ``org.joda.time.DateTime`` , ``org.joda.time.LocalDate`` and ``org.joda.time.LocalTime``. The usage method is shown below.

1. Use ``org.joda.time.DateTime`` to fetch time up to milliseconds.

.. code-block:: java

   DateTime dateTime = new DateTime();

2. Use ``org.joda.time.LocalDate`` when only date, which does not include time and TimeZone, is required.

.. code-block:: java

   LocalDate localDate = new LocalDate();

3. Use ``org.joda.time.LocalTime`` when only time, which does not include date and TimeZone, is required.

.. code-block:: java

   LocalTime localTime = new LocalTime();

4. Use ``org.joda.time.DateTime.withTimeAtStartOfDay()`` to fetch the current date with the time set to start of day.

.. code-block:: java

   DateTime dateTimeAtStartOfDay = new DateTime().withTimeAtStartOfDay();

|

    .. note::

        LocalDate and LocalTime do not have the TimeZone information.

    .. note::

        It is recommended that you use \ ``org.terasoluna.gfw.common.date.jodatime.JodaTimeDateFactory``\,
        for fetching instance of DateTime, LocalDate and LocalTime at the time of fetching current time.        

            .. code-block:: java

                DateTime dateTime = dataFactory.newDateTime();

        Refer to :doc:`./SystemDate` for using ``JodaTimeDateFactory``.

        LocalDate and LocalTime can be generated in the following way.

            .. code-block:: java

                LocalDate localDate = dataFactory.newDateTime().toLocalDate();
                LocalTime localTime = dataFactory.newDateTime().toLocalTime();



|

Fetching current time by specifying the time zone
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| \ ``org.joda.time.DateTimeZone``\  class indicates time zone.
| This class is used to fetch the current time for the specified time zone. The usage method is shown below.

.. code-block:: java

    DateTime dateTime = new DateTime(DateTimeZone.forID("Asia/Tokyo"));


\ ``org.terasoluna.gfw.common.date.jodatime.JodaTimeDateFactory``\ is used as follows:

.. code-block:: java

    // Fetching current system date using default TimeZone
    DateTime dateTime = dataFactory.newDateTime();

    // Changing to TimeZone of Tokyo
    DateTime dateTimeTokyo = dateTime.withZone(DateTimeZone.forID("Asia/Tokyo"));


For the list of other available Time zone ID strings, refer to `Available Time Zones <http://joda-time.sourceforge.net/timezones.html>`_.


|

Fetching the date and time by specifying Year Month Day Hour Minute and Second
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
Specific time can be specified in the constructor. An example is given below.

* Fetching DateTime by specifying time up to milliseconds

.. code-block:: java

    DateTime dateTime = new DateTime(year, month, day, hour, minute, second, millisecond);

* Fetching LocalDate by specifying Year Month and Day

.. code-block:: java

    LocalDate localDate = new LocalDate(year, month, day);

* Fetching LocalDate by specifying Hour Minute and Second

.. code-block:: java

    LocalTime localTime = new LocalTime(hour, minutes, seconds, milliseconds);

|

Fetching Year Month Day individually
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| DateTime provides a method to fetch Year and Month. The example is shown below.

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

.. tabularcolumns:: |p{0.1\linewidth}|p{0.9\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Get Year. In this example, \ ``2013``\  is returned.
   * - | (2)
     - | Get Month. In this example, \ ``1``\  is returned.
   * - | (3)
     - | Get Day. In this example, \ ``10``\  is returned.
   * - | (4)
     - | Get Day of Week. In this example, \ ``4``\  is returned.
       | The mapping of returned values and days of week is as follows:
       | [1:Monday, 2:Tuesday:, 3:Wednesday, 4:Thursday, 5:Friday, 6:Saturday, 7:Sunday]
   * - | (5)
     - | Get Hour. In this example, \ ``2``\  is returned.
   * - | (6)
     - | Get Minute. In this example, \ ``30``\  is returned.
   * - | (7)
     - | Get Second. In this example, \ ``22``\  is returned.
   * - | (8)
     - | Get Millisecond. In this example, \ ``123``\  is returned.

|

    .. note::

       getDayOfMonth() starts with 1, differing from the specifications of ``java.util.Calendar``.

|

Type conversion
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Interoperability with java.util.Date
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| In DateTime, type conversion with ``java.util.Date`` can be easily performed.

.. code-block:: java

    Date date = new Date();

    DateTime dateTime = new DateTime(date);  // (1)

    Date convertDate = dateTime.toDate();  // (2)

.. tabularcolumns:: |p{0.1\linewidth}|p{0.9\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Convert ``java.util.Date`` to DateTime by passing ``java.util.Date`` as an argument to DateTime constructor.
   * - | (2)
     - | Convert DateTime to ``java.util.Date`` using DateTime#toDate method.

|

String format
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

.. code-block:: java

    DateTime dateTime = new DateTime();

    dateTime.toString("yyyy-MM-dd HH:mm:ss");  // (1)

.. tabularcolumns:: |p{0.1\linewidth}|p{0.9\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | String of "yyyy-MM-dd HH:mm:ss" format is fetched.
       | For values that can be specified as arguments of toString, refer to `Input and Output <http://www.joda.org/joda-time/userguide.html#Input_and_Output>`_ .

|

Parsing from string
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

.. code-block:: java

    LocalDate localDate = DateTimeFormat.forPattern("yyyy-MM-dd").parseLocalDate("2012-08-09");  // (1)

.. tabularcolumns:: |p{0.1\linewidth}|p{0.9\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Convert "yyyy-MM-dd" string format to LocalDate type.
       | For values that can be specified as arguments of DateTimeFormat#forPattern, refer to `Formatters <http://www.joda.org/joda-time/userguide.html#Input_and_Output>`_.

|

Date operations
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Date calculations
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| LocalDate provides methods to perform date calculations. Examples are shown below.

.. code-block:: java

    LocalDate localDate = new LocalDate(); // localDate is 2013-01-10
    LocalDate yesterday = localDate.minusDays(1);  // (1)
    LocalDate tomorrow = localDate.plusDays(1);  // (2)
    LocalDate afterThreeMonth = localDate.plusMonths(3);  // (3)
    LocalDate nextYear = localDate.plusYears(1);  // (4)

.. tabularcolumns:: |p{0.1\linewidth}|p{0.9\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | The value specified in argument of LocalDate#minusDays is subtracted from the date. In this example, it becomes \ ``2013-01-09``\.
   * - | (2)
     - | The value specified in argument of LocalDate#plusDays is added to the date. In this example, it becomes \ ``2013-01-11``\.
   * - | (3)
     - | The value specified in argument of LocalDate#plusMonths is added to the number of months. In this example, it becomes \ ``2013-04-10``\.
   * - | (4)
     - | The value specified in argument of LocalDate#plusYears is added to the number of years. In this example, it becomes \ ``2014-01-10``\.

For methods other than those mentioned above, refer to `LocalDate JavaDoc <http://joda-time.sourceforge.net/apidocs/org/joda/time/LocalDate.html>`_ .

|

Fetching first and last day of the month
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| The method of fetching the first and the last day of the month by considering the current date and time as base, is shown below.

.. code-block:: java

    LocalDate localDate = new LocalDate(); // dateTime is 2013-01-10
    Property dayOfMonth = localDate.dayOfMonth(); // (1)
    LocalDate firstDayOfMonth = dayOfMonth.withMinimumValue(); // (2)
    LocalDate lastDayOfMonth = dayOfMonth.withMaximumValue(); // (3)

.. tabularcolumns:: |p{0.1\linewidth}|p{0.9\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Get Property object that holds the attribute values related to day of current month.
   * - | (2)
     - | Get first day of the month by fetching the minimum value from Property object. In this example, it becomes \ ``2013-01-01``\.
   * - | (3)
     - | Get last day of the month by fetching the maximum value from Property object. In this example, it becomes \ ``2013-01-31``\.

|

Fetching the first and the last day of the week
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| The method of fetching the first and the last day of the week by considering the current date and time as base, is shown below.

.. code-block:: java

    LocalDate localDate = new LocalDate(); // dateTime is 2013-01-10
    Property dayOfWeek = localDate.dayOfWeek(); // (1)
    LocalDate firstDayOfWeek = dayOfWeek.withMinimumValue(); // (2)
    LocalDate lastDayOfWeek = dayOfWeek.withMaximumValue(); // (3)

.. tabularcolumns:: |p{0.1\linewidth}|p{0.9\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Get Property object that holds attribute values related to the day of current week.
   * - | (2)
     - | Get first day of the week (Monday) by fetching the minimum value from Property object. In this example, it becomes \ ``2013-01-07``\.
   * - | (3)
     - | Get last day of the week (Sunday) by fetching the maximum value from Property object. In this example, it becomes \ ``2013-01-13``\.


Comparison of date time
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
By comparing the date and time, it can be determined whether it is a past or future date.

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


.. tabularcolumns:: |p{0.1\linewidth}|p{0.9\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | \ ``isAfter``\  method returns \ ``true``\  when the target date and time is later than the date and time of argument.
   * - | (2)
     - | \ ``isBefore``\  method returns \ ``true``\  when the target date and time is prior to the date and time of argument.
   * - | (3)
     - | \ ``isEqual``\  method returns \ ``true``\  when the target date and time is same as the date and time of argument.


Fetching the duration
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Joda-Time provides several classes related to duration. The following 2 classes are explained here.

* ``org.joda.time.Interval``
* ``org.joda.time.Period``

Interval
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Class indicating the interval between two instances (DateTime）.

The following 4 are checked using the Interval class.

* Checking whether the interval includes the specified date or interval.
* Checking whether the 2 intervals are consecutive.
* Fetching the difference between 2 intervals in an interval
* Fetching the overlapping interval between 2 intervals

For implementation, refer to the following example.

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

.. tabularcolumns:: |p{0.1\linewidth}|p{0.9\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Check whether the interval includes the specified date and interval, using Interval#contains method.
       | If the interval includes the specified date and interval, return "true". If not, return "false".
   * - | (2)
     - | Check whether the 2 intervals are consecutive, using Interval#abuts method.
       | If the 2 intervals are consecutive, return "true". If not, return "false".
   * - | (3)
     - | Fetch the difference between 2 intervals in an interval, using Interval#gap method.
       | In this example, the interval between "2013-08-16~2013-08-18" is fetched.
       | When there is no difference between intervals, return null.
   * - | (4)
     - | Fetch the overlapping interval between 2 intervals, using Interval#overlap method.
       | In this example, the interval between "2013-08-15~2013-08-16" is fetched.
       | When there is no overlapping interval, return null.

Intervals can be compared by converting into Period.

* Convert the intervals into Period when comparing them from abstract perspectives such as Month or Day.

.. code-block:: java


    // Convert to Period
    interval1.toPeriod();

|
|

Period
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Period is a class indicates duration in terms of Year, Month and Week.

| For example, when Period of "1 month" is added to Instant（DateTime）indicating "March 1", DateTime will be "April 1".
| The result of adding the Period of "1 month" to "March 1" and "April 1" is as shown below.

* Number of days is "31" when a Period of "1 month" is added to "March 1".
* Number of days is "30" when a Period of "1 month" is added to "April 1".

The result of adding a Period of "1 month" differs depending on the target DateTime.

| Two different types of implementations have been provided for Period.

* Single field Period (Example：Type having values of single unit such as "1 Day" or "1 month")
* Any field Period (Example：Type indicating the period and having values of multiple units such as "1 month 2 days 4 hours")

For details, refer to `Period <http://joda-time.sourceforge.net/key_period.html>`_.

|

JSP Tag Library
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| fmt:formatDate tag of JSTL handles objects of java.util.Date and java.util.TimeZone.
| Use Joda tag library to handle DateTime, LocalDateTime, LocalDate, LocalTime and DateTimeZone objects of Joda-time.
| The functionalities of JSP Tag Library are almost same as those of JSTL, hence it can be used easily if the person has knowledge of JSTL.

|

How to set
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

The following taglib definition is required to use the tag library.

.. code-block:: jsp

    <%@ taglib uri="http://www.joda.org/joda/time/tags" prefix="joda"%>

joda:format tag
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

joda:format tag is used to format the objects of DateTime, LocalDateTime, LocalDate and LocalTime.

.. code-block:: jsp

    <% pageContext.setAttribute("now", new org.joda.time.DateTime()); %>

    <span>Using pattern="yyyyMMdd" to format the current system date</span><br/>
    <joda:format value="${now}" pattern="yyyyMMdd" />
    <br/>
    <span>Using style="SM" to format the current system date</span><br/>
    <joda:format value="${now}" style="SM" />

**Output result**

.. figure:: images/joda_format_tag.png
   :alt: /jodatime
   :width: 55%

The list of attributes of joda:format tag is as follows:

.. tabularcolumns:: |p{0.05\linewidth}|p{0.1\linewidth}|p{0.85\linewidth}|
.. list-table:: **Attribute information**
   :header-rows: 1
   :widths: 5 10 85

   * - No.
     - Attributes
     - Description
   * - 1.
     - | value
     - | Set the instance of ReadableInstant or ReadablePartial.
   * - 2.
     - | var
     - | Variable name
   * - 3.
     - | scope
     - | Scope of variables
   * - 4.
     - | locale
     - | Locale information
   * - 5.
     - | style
     - | Style information for doing formatting (2 digits. Set the style for date and time. Values that can be entered are S=Short, M=Medium, L=Long, F=Full, -=None) 
   * - 6.
     - | pattern
     - | Pattern for doing formatting (yyyyMMdd etc.). For patterns that can be entered, refer to `Input and Output <http://www.joda.org/joda-time/userguide.html#Input_and_Output>`_.
   * - 7.
     - | dateTimeZone
     - | Time zone

For other Joda-Time tags, refer to `Joda Time JSP tags User guide <http://joda-time.sourceforge.net/contrib/jsptags/userguide.html>`_.

    .. note::
        When the date and time is displayed by specifying style attributes, the displayed contents differ depending on browser locale.
        Locale of the format displayed in the above style attribute is "en".

|

Example (display of calendar)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Using Spring MVC, sample for displaying a month wise calendar, is shown below.

.. tabularcolumns:: |p{0.33\linewidth}|p{0.33\linewidth}|p{0.33\linewidth}|
.. list-table::
    :header-rows: 1

    * - Process name
      - URL
      - Handler method
    * - Display of current month's calendar
      - /calendar
      - today
    * - Display of specified month's calendar
      - /calendar/month?year=yyyy&month=m
      - month

The controller is implemented as follows:

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

The ``CalendarOutput`` class mentioned below is JavaBean having the consolidated information to be output on the screen.


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

        For the sake of simplicity, this sample code includes all the logic in the handler method of Controller,
        but in real scenario, this logic should be delegated to Helper classes to improve maintainability.

|

In JSP(calendar.jsp), it is output as follows:

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

Access {contextPath}/calendar to display the calendar below (showing result of November 2012).

.. figure:: images/calendar-today.jpg
   :alt: /calendar
   :width: 30%

Access {contextPath}/calendar/month?year=2012&month=12 to display the calendar below.

.. figure:: images/calendar-month.jpg
   :alt: /calendar/month?year=2012&month=12
   :width: 30%


.. raw:: latex

   \newpage

Appendix
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Japanese calendar operation before Java8
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| The class called ``java.time.chrono.JapaneseDate`` is offered in Java8 for Japanese calendar operation however it is possible to deal with the Japanese calendar by ``java.util.Calendar`` class in older Java version.
| Specifically, it is necessary to specify the following ``java.util.Locale`` in the ``java.util.Calendar`` class and ``java.text.DateFormat`` class.

.. code-block:: java

   Locale locale = new Locale("ja", "JP", "JP");

| Below, it shows an example of the Japanese calendar display using the ``Calendar`` class.

.. code-block:: java

   Locale locale = new Locale("ja", "JP", "JP");
   Calendar cal = Calendar.getInstance(locale); // Ex, 2015-06-05
   String format1 = "Gy.MM.dd";
   String format2 = "GGGGyy/MM/dd";

   DateFormat df1 = new SimpleDateFormat(format1, locale);
   DateFormat df2 = new SimpleDateFormat(format2, locale);

   df1.format(cal.getTime()); // "H27.06.05"
   df2.format(cal.getTime()); // "平成27/06/05"

| Similarly, it can also perform parsing the string.

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

        | The ``java.util.JapaneseImperialCalendar`` object corresponding to the Japanese calendar is created by specifying the ``new Locale("ja", "JP", "JP")`` into the ``getInstance`` method.
        | If you specify the other, ``java.util.GregorianCalendar`` object gets created therefore it should be noted.

