System Date
================================================================================

.. only:: html

 .. contents:: Table of Contents
    :depth: 3
    :local:

Overview
--------------------------------------------------------------------------------

| In application development, testing may need to be carried out at any random date and time and not necessarily at system time of the server.
| Even in Production environment, there could be cases wherein recovery is performed by shifting the date to earlier date.

| Therefore, it is desirable to have a setting that can fetch any date and time at development and operation sides instead of system time of the server.

| Common library provides ``org.terasoluna.gfw.common.date.DateFactory`` interface that can arbitrarily change the time to be returned.
| Using ``DateFactory``\ , date can be returned in the following 5 formats.

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
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

| Define implementation class of \ ``DateFactory``\  in bean definition file and inject it in Java class by \ ``DateFactory``\  interface.
| Depending on the intended use, select from the following implementation classes.


.. tabularcolumns:: |p{0.30\linewidth}|p{0.35\linewidth}|p{0.35\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 30 35 35

   * - Class name
     - Overview
     - Remarks
   * - | org.terasoluna.gfw.common.date.DefaultDateFactory
     - | Returns system time of application server.
     - | Time cannot be changed as the value is equivalent to that fetched by new DateTime().
   * - | org.terasoluna.gfw.common.date.JdbcFixedDateFactory
     - | Returns the fixed time registered in DB.
     - | It is assumed to be used in Integration Test environment that requires a completely fixed time.
       | It is not used in Performance Test environment and Production environment.
       | Table definition is needed.
   * - | org.terasoluna.gfw.common.date.JdbcAdjustedDateFactory
     - | Returns the time fetched by adding difference (milliseconds) between the time registered in DB and system time of application server.
     - | It is assumed to be used in Integration Test environment and System Test environment.
       | It can be used in Production environment by setting the difference value to 0.
       | Table definition is needed.

|

    .. note::

        It is recommended to define the bean definition file that sets each class, in [projectname]-env .xml so that it can be changed according to the environment.
        Using DateFactory, the date and time can be changed just by changing the settings of bean definition file, without having to change the source code.
        Example of Bean definition file is described later.

|

Returning system time of server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Use \ ``org.terasoluna.gfw.common.date.DefaultDateFactory``\ . For details, refer to the following.

**Bean definition file ([projectname]-env.xml)**

.. code-block:: xml

    <bean id="dateFactory" class="org.terasoluna.gfw.common.date.DefaultDateFactory" />  <!-- (1) -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Define DefaultDateFactory class in bean.

.. _dateFactory-java:

**Java class**

.. code-block:: java

    @Inject
    DateFactory dateFactory;  // (1)

    public TourInfoSearchCriteria setUpTourInfoSearchCriteria() {

        DateTime dateTime = dateFactory.newDateTime();  // (2)

        // omitted
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Inject DateFactory in the class to be used.
   * - | (2)
     - | Call the method that returns the class instance of the date to be used.
       | Fetch in ``org.joda.time.DateTime`` format.

|

    .. note::
       For Joda Time and format etc., refer to :doc:`./Utilities/JodaTime` .

    .. note::
        When testing is to be carried out by changing the date and time using JUnit etc., any date and time can be set
        by replacing the Factory implementation class with mock class.

|

Returning the fixed time fetched from DB
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Use \ ``org.terasoluna.gfw.common.date.JdbcFixedDateFactory``\ . For details, refer to the following.

**Bean definition file**

.. code-block:: xml

    <bean id="dateFactory" class="org.terasoluna.gfw.common.date.JdbcFixedDateFactory" >  <!-- (1) -->
        <property name="dataSource" ref="dataSource" />  <!-- (2) -->
        <property name="currentTimestampQuery" value="SELECT now FROM system_date" />  <!-- (3) -->
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{1.00\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 100

   * - Sr. No.
     - Description
   * - | (1)
     - | Define ``org.terasoluna.gfw.common.date.JdbcFixedDateFactory`` in bean.
   * - | (2)
     - Datasource (``javax.sql.DataSource``) settings.
   * - | (3)
     - | Settings related to SQL for fetching fixed time ``currentTimestampQuery``.
       | Set the SQL query that returns the date and time specified in table.


**Example of Table settings**

| Records need to be added by creating a table as shown below.

.. code-block:: sql

  CREATE TABLE system_date(now timestamp NOT NULL);
  INSERT INTO system_date(now) VALUES (current_date);

.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 80

   * - Record number
     - now
   * - 1
     - 2013-01-01 01:01:01.000

**Java class**

.. code-block:: java

    @Inject
    DateFactory dateFactory;

    @RequestMapping(value="datetime", method = RequestMethod.GET)
    public String listConfirm(Model model) {

        for (int i=0; i < 3; i++) {
            model.addAttribute("jdbcFixedDateFactory" + i, dateFactory.newDateTime()); // (1)
            model.addAttribute("DateTime" + i, new DateTime()); // (2)
        }

        return "date/dateTimeDisplay";
    }

**Execution result**

.. figure:: ./images/system-date-jdbc-fixed-date-factory.png
   :alt: system-date-jdbc-fixed-date-factory
   :width: 30%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Pass the \ ``JdbcFixedDateFactory.newDateTime()``\  result to screen.
       | The fixed value set in DB is output.
   * - | (2)
     - | Pass the \ ``new DateTime()``\  result to screen, for confirmation.
       | Output result shows a different value each time.

**SQL log**

.. code-block:: xml

    16. SELECT now FROM system_date {executed in 0 msec}
    17. SELECT now FROM system_date {executed in 1 msec}
    18. SELECT now FROM system_date {executed in 0 msec}

| Access log is output to DB using ``JdbcFixedDateFactory.newDateTime()``.
| In order to output SQL log, \ ``Log4jdbcProxyDataSource``\  described in :doc:`./DataAccessCommon` is used.

|

Returning time obtained by adding the difference registered in DB to the server system time
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| Use \ ``org.terasoluna.gfw.common.date.JdbcAdjustedDateFactory``\ .
| Fetch the difference in time by executing SQL set in \ ``adjustedValueQuery``\  property .
| For details, refer to the following.

**Bean definition file**

.. code-block:: xml

  <bean id="dateFactory" class="org.terasoluna.gfw.common.date.JdbcAdjustedDateFactory" >
    <property name="dataSource" ref="dataSource" />
    <!-- <property name="adjustedValueQuery" value="SELECT diff FROM operation_date" /> --><!-- (1) -->
    <!-- <property name="adjustedValueQuery" value="SELECT diff * 1000 FROM operation_date" /> --><!-- (2) -->
    <property name="adjustedValueQuery" value="SELECT diff * 60 * 1000 FROM operation_date" /><!-- (3) -->
    <!-- <property name="adjustedValueQuery" value="SELECT diff * 60 * 60 * 1000 FROM operation_date" /> --><!-- (4) -->
    <!-- <property name="adjustedValueQuery" value="SELECT diff * 24 * 60 * 60 * 1000 FROM operation_date" /> --><!-- (5) -->
  </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | SQL when the difference registered in operation_date table is in "milliseconds" 
   * - | (2)
     - | SQL when the difference registered in operation_date table is in "seconds"
   * - | (3)
     - | SQL when the difference registered in operation_date table is in "minutes"
   * - | (4)
     - | SQL when the difference registered in operation_date table is in "hours"
   * - | (5)
     - | SQL when the difference registered in operation_date table is in "days"

**Example of table settings**

| Records need to be added by creating a table as shown below.

.. code-block:: sql

  CREATE TABLE operation_date(diff bigint NOT NULL);
  INSERT INTO operation_date(diff) VALUES (-1440);

.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 80

   * - Record number
     - diff
   * - 1
     - -1440

| In this example, the difference is in "minutes". (DB data is specified as -1440 minutes = previous day)
| By converting the retrieved result into milliseconds (integer value), the unit for DB value can be set to any one of the units namely, hours, minutes, seconds or milliseconds.


    .. note::

        Above SQL is for PostgreSQL. For Oracle, it is better to use \ ``NUMBER(19)``\  instead of \ ``BIGINT``\ .

**Java class**

.. code-block:: java

    @Inject
    DateFactory dateFactory;

    @RequestMapping(value="datetime", method = RequestMethod.GET)
    public String listConfirm(Model model) {

        model.addAttribute("firstExpectedDate", new DateTime());  // (1)
        model.addAttribute("serverTime", dateFactory.newDateTime());  // (2)
        model.addAttribute("lastExpectedDate", new DateTime());  // (3)

        return "date/dateTimeDisplay";
    }

**Execution result**

.. figure:: ./images/system-date-jdbc-adjusted-date-factory.png
   :alt: system-date-jdbc-fixed-date-factory
   :width: 30%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | For verification purpose, pass a time that is prior to the \ ``DateTime``\  generated by \ ``dateFactory``\ , to screen.
   * - | (2)
     - | Pass the result of \ ``JdbcAdjustedDateFactory.newDateTime()``\  to screen.
       | Fetched time is the time derived by subtracting 1440 minutes from execution time.
   * - | (3)
     - | For verification purpose, set a time that is later than the \ ``DateTime``\  generated by \ ``dateFactory``\ .

**SQL log**

.. code-block:: xml

    17. SELECT diff * 60 * 1000 FROM operation_date {executed in 1 msec}

| Access log is output to DB using ``dateFactory.newDateTime()``.

|

Caching and reloading the difference
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. _useCache:

When the difference value is set to 0 and used in production environment, performance deteriorates as the difference is fetched each time from DB.
Therefore, in JdbcAdjustedDateFactory, it is possible to cache the acquisition result.
Once the value fetched at booting is cached, table is not accessed for each request.

**Bean definition file**

.. code-block:: xml

  <bean id="dateFactory" class="org.terasoluna.gfw.common.date.JdbcAdjustedDateFactory" >
    <property name="dataSource" ref="dataSource" />
    <property name="adjustedValueQuery" value="SELECT diff * 60 * 1000 FROM operation_date" />
    <property name="useCache" value="true" /> <!-- (1) -->
  </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{1.00\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 100

   * - Sr. No.
     - Description
   * - | (1)
     - | When it is 'true', the value fetched from table is cached. By default it is 'false' so the value is not cached.
       | When it is 'false', SQL is executed each time when DateFactory is used.

When the difference value is to be changed after setting cache, cache value can be reloaded by executing \ ``JdbcAdjustedDateFactory.reload()``\  method after
changing the table value.

**Java class**

.. code-block:: java

    @Controller
    @RequestMapping(value = "reload")
    public class ReloadAdjustedValueController {

        @Inject
        JdbcAdjustedDateFactory dateFactory;

        // omitted

        @RequestMapping(method = RequestMethod.GET)
        public String reload() {

            long adjustedValue = dateFactory.reload(); // (1)

            // omitted
        }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | By executing reload method of JdbcAdjustedDateFactory, difference can be reloaded from table.

|

Testing
--------------------------------------------------------------------------------

| When carrying out testing, it may be necessary to change to another date and time instead of the current date and time.

+----------------------+-------------------------+-----------------------------------------------------------------------------------------------+
| Environment          | DateFactory to be used  | Test details                                                                                  |
+======================+=========================+===============================================================================================+
| Unit Test            | DefaultDateFactory      | Mock for DataFactory is created for date related testing                                      |
+----------------------+-------------------------+-----------------------------------------------------------------------------------------------+
| Integration Test     | DefaultDateFactory      | Testing not relating to date                                                                  |
|                      +-------------------------+-----------------------------------------------------------------------------------------------+
|                      | JdbcFixedDateFactory    | When testing is carried out by having a fixed date and time                                   |
|                      +-------------------------+-----------------------------------------------------------------------------------------------+
|                      | JdbcAdjustedDateFactory | When linked with an external system and testing is done for multiple days considering         |
|                      |                         | the date flow of a testing for a single day                                                   |
+----------------------+-------------------------+-----------------------------------------------------------------------------------------------+
| System Test          | JdbcAdjustedDateFactory | When testing is carried out by specifying the testing date or for a future date               |
+----------------------+-------------------------+-----------------------------------------------------------------------------------------------+
| Production           | DefaultDateFactory      | When there is no possibility of change in actual time                                         |
|                      +-------------------------+-----------------------------------------------------------------------------------------------+
|                      | JdbcAdjustedDateFactory || **When the possibility to change the time is to be retained in an operation.**               |
|                      |                         || **Normally the difference is set as 0. It is provided only if required.**                    |
|                      |                         || :ref:`useCache<useCache>` **should always be set to 'true'.**                                |
+----------------------+-------------------------+-----------------------------------------------------------------------------------------------+

|

Unit Test
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| In Unit Test, sometimes it needs to be verified whether the time is registered and the registered time has been updated as expected.

| In such cases, if the server time is registered as it is during the process,
| it becomes difficult to perform regression test in JUnit, as the value differs with each test execution.
| Here, by using DateFactory, the time to be registered can be fixed to any value.


| Use mock to match the time in milliseconds. An example wherein fixed date is returned by setting a value in dateFactory, is shown below.
| In this example, \ `mockito <https://code.google.com/p/mockito/>`_\  is used for mock.

**Java class**

.. code-block:: java

    import org.terasoluna.gfw.common.date.DateFactory;

    // omitted

    @Inject
    StaffRepository staffRepository;

    @Inject
    DateFactory dateFactory;

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

**JUnit source**

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

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Value specified in (2) of mock is fetched and set.
   * - | (2)
     - | Set the date and time to the return value of DateFactory in mock.
   * - | (3)
     - | **success** is returned since it is same as the fixed value that has been set.

|

Example wherein process changes with date
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| The example below illustrates a Service class which is implemented with the specification of "Reserved tour cannot be cancelled if the cancellation is sought less than 7 days before the departure day".

**Java class**

.. code-block:: java

  import org.terasoluna.gfw.common.date.DateFactory;

    // omitted

    @Inject
    DateFactory dateFactory;

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

.. tabularcolumns:: |p{0.10\linewidth}|p{1.00\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 100

   * - Sr. No.
     - Description
   * - | (1)
     - | Fetch current date and time. For ``LocalDate``, refer to :doc:`./Utilities/JodaTime`.
   * - | (2)
     - | Calculate the last date up to which the tour can be cancelled.
   * - | (3)
     - | Check if today's date is later than the last date for cancellation.
   * - | (4)
     - | \ ``BusinessException``\  is thrown if the date exceeds the last date for cancellation.

**JUnit source**

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

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Set the departure date to 2012/10/10 in the tour reservation information to be fetched from Repository class.
   * - | (2)
     - | Set the Return value of dateFactory.newDateTime() to 2012/10/1.
   * - | (3)
     - | Execute Cancel. Cancellation is successful as the date is prior to the last date for cancellation.
   * - | (4)
     - | Return value of dateFactory.newDateTime() should be 2012/10/9.
   * - | (5)
     - | Execute Cancel. Cancellation fails as the date falls after the last date for cancellation.

|

Integration Test
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| In Integration Test, there may be cases wherein data of several days (for example: files) is created and transferred in a single day,
| for communicating with the system.

.. figure:: ./images/DateFactoryIT.png
   :alt: DateFactorySI
   :width: 60%

| When the actual date is 2012/10/1
| Use JdbcAdjustedDateFactory and set the SQL to calculate the difference with test execution date.


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | 1
     - | Set the difference between 9:00-11:00 as "0 days" and return value of dateFactory as 2012/10/1.
   * - | 2
     - | Set the difference between 11:00-13:00 as "0 days" and return value of dateFactory as 2012/10/10.
   * - | 3
     - | Set the difference between 13:00-15:00 as "30 days" and return value of dateFactory as 2012/10/31.
   * - | 4
     - | Set the difference between 15:00-17:00 as "31 days" and return value of dateFactory as 2012/11/1.

Date can be changed only by changing the table value.

|

System Test
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In System Test, testing may be carried out by creating test scenarios assuming the operation date.

.. figure:: ./images/DateFactoryST.png
   :alt: DateFactoryPT
   :width: 60%

| Use JdbcAdjustedDateFactory and set SQL that calculates the date difference.
| Create a mapping table for actual date and operation date like 1, 2, 3 and 4 as shown in the figure. Testing can be carried out on the desired date, only by changing the difference value in the table.

|

Production
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| By setting the difference value to '0' using JdbcAdjustedDateFactory, the return value of dateFactory can be set to the date same as the actual date,
| without changing the source. Even the bean definition file need not be changed from System Test onwards.
| Further, even if the need to change date and time arises, return value of dateFactory can be changed by changing the table value.

    .. warning::

        When using in Production environment, verify that the difference value in the table used in Production environment is 0.

        **Configuration example**

        Execute the following

        - When using the table for the first time in Production environment
            - INSERT INTO operation_date (diff) VALUES (0);
        - When test execution is completed in Production environment
            - UPDATE operation_date SET diff=0;

        :ref:`useCache<useCache>` **should always be set to 'true'**.

| When there is no change in time, it is recommended to change the configuration file to DefaultDateFactory.

.. raw:: latex

   \newpage

