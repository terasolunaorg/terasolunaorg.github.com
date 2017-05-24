System Date
================================================================================

.. only:: html

 .. contents:: Table of Contents
    :depth: 3
    :local:

Overview
--------------------------------------------------------------------------------

In application development, testing may need to be carried out at any random date and time and not necessarily at system time of the server.
Even in Production environment, there could be cases wherein recovery is performed by shifting the date to earlier date.

Therefore, it is desirable to have a setting that can fetch any date and time at development and operation sides instead of system time of the server.

|

About the components that are offered by common library
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The common library providing a component (In the following, API is referred as "Date Factory") for obtaining the system time.

Component provided by common library is divided into two artifacts terasoluna-gfw-common and terasoluna-gfw-jodatime,

* terasoluna-gfw-common is a Date Factory that uses only the Java standard API
* terasoluna-gfw-jodatime is a Date Factory that uses Joda Time API

Following is the class diagram of components that provided by the common library.

.. figure:: ./images/systemdate-class-diagram.png
    :alt: Class Diagram of Date Factory
    :width: 100%

terasoluna-gfw-common
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Below interface provided as a component of terasoluna-gfw-common.

.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 35 65

    * - Interface
      - Description
    * - | org.terasoluna.gfw.common.date.
        | ClassicDateFactory
      - Interface to obtain an instance of the following classes as the system time provided by Java.

        * \ ``java.util.Date``\
        * \ ``java.sql.Timestamp``\
        * \ ``java.sql.Date``\
        * \ ``java.sql.Time``\

        The common library provides the following classes as an implementation class of the interface.

        * \ ``org.terasoluna.gfw.common.date.DefaultClassicDateFactory``\


terasoluna-gfw-jodatime
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Below interface provided as a component of terasoluna-gfw-jodatime.

.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 35 65

    * - Interface
      - Description
    * - | org.terasoluna.gfw.common.date.jodatime.
        | JodaTimeDateTimeFactory
      - Interface to obtain an instance of the following classes as the system time provided by Joda Time.

        * \ ``org.joda.time.DateTime``\

    * - | org.terasoluna.gfw.common.date.jodatime.
        | JodaTimeDateFactory
      - Interface that inherits from \ ``ClassicDateFactory`` \ and \ ``JodaTimeDateTimeFactory``\.

        The common library provides the following classes as an implementation class of the interface.

        * \ ``org.terasoluna.gfw.common.date.jodatime.DefaultJodaTimeDateFactory``\
        * \ ``org.terasoluna.gfw.common.date.jodatime.JdbcFixedJodaTimeDateFactory``\
        * \ ``org.terasoluna.gfw.common.date.jodatime.JdbcAdjustedJodaTimeDateFactory``\

        **In this guideline, it is recommended that you use the implementation class corresponding to this interface.**
    * - | org.terasoluna.gfw.common.date.
        | DateFactory
      - Interface that inherits from \ ``JodaTimeDateFactory``\ (Deprecated).

        This interface is an interface provided for backward compatibility with \ ``DateFactory`` \ which is offered by terasoluna-gfw-common 1.0.x.

        The common library provides the following classes as an implementation class of the interface.

        * \ ``org.terasoluna.gfw.common.date.DefaultDateFactory``\ (Deprecated)
        * \ ``org.terasoluna.gfw.common.date.JdbcFixedDateFactory``\ (Deprecated)
        * \ ``org.terasoluna.gfw.common.date.JdbcAdjustedDateFactory``\ (Deprecated)

        **Since these interfaces and corresponding implementation classes are deprecated API, it is prohibited to use in new applications development.**

.. note::

    For Joda Time, refer :doc:`./JodaTime`.

|

How to use
--------------------------------------------------------------------------------

The implementation class of Date Factory interface is defined in the bean definition file and an instance of the Date Factory is used by injection in Java class.

Depending on the intended use, select from the following implementation classes.

.. tabularcolumns:: |p{0.30\linewidth}|p{0.30\linewidth}|p{0.40\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 30 30 40

   * - Class name
     - Overview
     - Remarks
   * - | org.terasoluna.gfw.common.date.jodatime.
       | DefaultJodaTimeDateFactory
     - Returns system time of application server.
     - Time cannot be changed as the value is equivalent to that fetched by new \ ``new DateTime()``\.
   * - | org.terasoluna.gfw.common.date.jodatime.
       | JdbcFixedJodaTimeDateFactory
     - Returns the fixed time registered in DB.
     - It is assumed to be used in Integration Test environment that requires a completely fixed time. It is not used in Performance Test environment and Production environment.

       In order to use this class, table is required for managing a fixed time.
   * - | org.terasoluna.gfw.common.date.jodatime.
       | JdbcAdjustedJodaTimeDateFactory
     - Returns the time fetched by adding difference (milliseconds) between the time registered in DB and system time of application server.
     - It is assumed to be used in Integration Test environment and System Test environment but It can also be used in Production environment by setting the difference value as 0.

       In order to use this class, table is required for managing a difference value.

.. note::

    It is recommended to define the bean definition file that sets implementation class in [projectName]-env.xml so that it can be changed according to the environment. 
    Using Date Factory, the date and time can be changed just by changing the settings of bean definition file, without having to change the source code. 
    Example of Bean definition file is described later.

.. tip::

    If you want to test in JUnit by changing the date and time, it is also possible to set a random time by replacing the implementation class of the interface to mock class.
    For replacement method refer [:ref:`SystemDateTestingUnitTest`].

|

pom.xml setting
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| Add terasoluna-gfw-jodatime dependency.
| In case of multi-project configuration, add in the \ :file:`pom.xml`\(:file:`projectName-domain/pom.xml`) of the domain project.

If project is created from `blank project <https://github.com/terasolunaorg/terasoluna-gfw-web-multi-blank>`_\, dependencies of terasoluna-gfw-jodatime is pre-configured.

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

    * - Sr. No.
      - Description
    * - (1)
      - Add a terasoluna-gfw-jodatime-dependencies into dependencies.
        The dependencies of Joda Time related to Date Factory and Joda Time libraries are defined for terasoluna-gfw-jodatime-dependencies.

.. note:: 
	In the above setting example, since it is assumed that the dependent library version is managed by the parent project terasoluna-gfw-parent, specifying the version in pom.xml is not necessary.

|

Returning system time of server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Use \ ``org.terasoluna.gfw.common.date.jodatime.DefaultJodaTimeDateFactory``\.

**Bean definition file([projectname]-env.xml)**

.. code-block:: xml

    <bean id="dateFactory" class="org.terasoluna.gfw.common.date.jodatime.DefaultJodaTimeDateFactory" />  <!-- (1) -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Define \ ``DefaultJodaTimeDateFactory`` \ class in bean.

|

.. _dateFactory-java:

**Java class**

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

   * - Sr. No.
     - Description
   * - | (2)
     - | Inject Date Factory in the class to be used.
   * - | (3)
     - | Call the method that returns the class instance of the date to be used
       | In above example \ ``org.joda.time.DateTime`` \ type instance is fetched.

|

Returning the fixed time fetched from DB
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Use \ ``org.terasoluna.gfw.common.date.jodatime.JdbcFixedJodaTimeDateFactory``\.

**Bean definition file**

.. code-block:: xml

    <bean id="dateFactory" class="org.terasoluna.gfw.common.date.jodatime.JdbcFixedJodaTimeDateFactory" >  <!-- (1) -->
        <property name="dataSource" ref="dataSource" />  <!-- (2) -->
        <property name="currentTimestampQuery" value="SELECT now FROM system_date" />  <!-- (3) -->
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - Define \ ``JdbcFixedJodaTimeDateFactory`` \ in bean.
   * - | (2)
     - Specifies the datasource (\ ``javax.sql.DataSource``\ ) in the \ ``dataSource`` \ property in which table to manage a fixed time is present.
   * - | (3)
     - Set the SQL in \ ``currentTimestampQuery`` \ property for obtaining a fixed time.

|

**Example of Table settings**

Records need to be added by creating a table as shown below.

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

|

**Java class**

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

   * - Sr. No.
     - Description
   * - | (4)
     - Pass the system time retrieved from Date Factory to the screen.

       When confirming the results, the output is the fixed value set in DB.
   * - | (5)
     - Pass the result of \ ``new DateTime()`` \ for confirmation.

       When confirming the results, each time the output is different value (System time of the application server).

|

**Execution result**

.. figure:: ./images/system-date-jdbc-fixed-date-factory.png
    :alt: system-date-jdbc-fixed-date-factory
    :width: 40%

|

**SQL log**

.. code-block:: console

    16. SELECT now FROM system_date {executed in 0 msec}
    17. SELECT now FROM system_date {executed in 1 msec}
    18. SELECT now FROM system_date {executed in 0 msec}

Access log is output to DB if Date Factory is called. 
In order to output SQL log, \ ``Log4jdbcProxyDataSource`` \ described in :doc:`../DataAccessDetail/DataAccessCommon` is used.

|

Returning time obtained by adding the difference registered in DB to the server system time
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Use \ ``org.terasoluna.gfw.common.date.jodatime.JdbcAdjustedJodaTimeDateFactory``\.

**bean definition file**

.. code-block:: xml

  <bean id="dateFactory" class="org.terasoluna.gfw.common.date.jodatime.JdbcAdjustedJodaTimeDateFactory" > <!-- (1) -->
    <property name="dataSource" ref="dataSource" /> <!-- (2) -->
    <property name="adjustedValueQuery" value="SELECT diff * 60 * 1000 FROM operation_date" /> <!-- (3) -->
  </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - Define \ ``JdbcAdjustedJodaTimeDateFactory`` \ in bean.
   * - | (2)
     - Specifies the datasource (\ ``javax.sql.DataSource``\ ) in the \ ``dataSource`` \ property in which table to manage a difference value is present.
   * - | (3)
     - Set the SQL in \ ``adjustedValueQuery`` \ property for obtaining a difference value.

       The above SQL is the SQL of the difference values in "minutes" unit.

|

**Example of table settings**

Records need to be added by creating a table as shown below.

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

| In this example, the difference is in “minutes”. (DB data is specified as -1440 minutes = previous day)
| By converting the retrieved result into milliseconds (integer value), the unit for DB value can be set to any one of the units namely, hours, minutes, seconds or milliseconds.


.. note::

    Above SQL is for PostgreSQL. For Oracle, it is better to use \ `NUMBER(19)` \ instead of \ `BIGINT`\.

.. tip::

    If you want to make difference value unit other than the "minutes", the following SQL can be specified in the \ `adjustedValueQuery` \ property.

     .. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
     .. list-table::
         :header-rows: 1
         :widths: 25 75
         :class: longtable

         * - Difference value unit
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

**Java class**

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

   * - Sr. No.
     - Description
   * - | (4)
     - Pass the time that retrieved before calling the Date Factory method to the screen for confirmation.
   * - | (5)
     - Pass the system time retrieved from Date Factory to the screen.

       When confirming the results, the output is the time that is 1440 minutes subtracted from the execution time.
   * - | (6)
     - Pass the time that retrieved after calling the Date Factory method to the screen for confirmation.

|

**Execution result**

.. figure:: ./images/system-date-jdbc-adjusted-date-factory.png
    :alt: system-date-jdbc-fixed-date-factory
    :width: 40%

|

**SQL log**

.. code-block:: xml

    17. SELECT diff * 60 * 1000 FROM operation_date {executed in 1 msec}

Access log is output to DB if Date Factory is called.

|

Caching and reloading the difference
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. _useCache:

When the difference value is set to 0 and used in production environment, performance deteriorates as the difference is fetched each time from DB. 
Therefore, in \ `JdbcAdjustedJodaTimeDateFactory`\, it is possible to cache the difference values obtained by the SQL. 
Once the value fetched at booting is cached, table is not accessed for each request.

**bean definition file**

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

   * - Sr. No.
     - Description
   * - | (1)
     - | When it is \ `true`\, the difference value fetched from table is cached. By default it is \ `false` \ so the value is not cached.
       | When it is \ `false`\ , SQL is executed each time when the method of Date Factory is called.

|

When the difference value is to be changed after setting cache, 
cache value can be reloaded by executing \ `JdbcAdjustedJodaTimeDateFactory.reload()` \ method after changing the table value.

**Java class**

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

   * - Sr. No.
     - Description
   * - | (2)
     - By executing \ `reload` \ method of the \ `JdbcAdjustedJodaTimeDateFactory`\, difference can be reloaded from table.

|

Testing
--------------------------------------------------------------------------------

When carrying out testing, it may be necessary to change to another date and time instead of the current date and time.

.. tabularcolumns:: |p{0.15\linewidth}|p{0.25\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 15 25 60

    * - Environment
      - Date Factory to be used
      - Test details
    * - Unit Test
      - DefaultJodaTimeDateFactory
      - Mock for DataFactory is created for date related testing
    * - Integration Test
      - DefaultJodaTimeDateFactory
      - Testing not relating to date
    * -
      - JdbcFixedJodaTimeDateFactory
      - When testing is carried out by having a fixed date and time
    * -
      - JdbcAdjustedJodaTimeDateFactory
      - When linked with an external system and testing is done for multiple days considering the date flow of a testing for a single day
    * - System Test
      - JdbcAdjustedJodaTimeDateFactory
      - When testing is carried out by specifying the testing date or for a future date
    * - Production
      - DefaultJodaTimeDateFactory
      - When there is no possibility of change in actual time
    * -
      - JdbcAdjustedJodaTimeDateFactory
      - **When the possibility to change the time is to be retained in an operation.**

        **Normally the difference is set as 0. It is provided only if required.**
        **Always,** :ref:`useCache<useCache>` **should be set to 'true'.**

|

.. _SystemDateTestingUnitTest:

Unit Test
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In Unit Test, sometimes it needs to be verified whether the time is registered and the registered time has been updated as expected.

In such cases, if the server time is registered as it is during the process, 
it becomes difficult to perform regression test in JUnit, as the value differs with each test execution. 
Here, by using Date Factory, the time to be registered can be fixed to any value.


Use mock to match the time in milliseconds. An example wherein fixed date is returned by setting a value in Date Factory, is shown below. 
In this example, \ `mockito <https://code.google.com/p/mockito/>`_ \ is used for mock.

**Java class**

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

**JUnit source**

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

The example below illustrates a Service class which is implemented with the specification of "Reserved tour cannot be cancelled if the cancellation is sought less than 7 days before the departure day".

**Java class**

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

   * - Sr. No.
     - Description
   * - | (1)
     - | Fetch current date and time. For ``LocalDate``, refer to :doc:`./JodaTime`.
   * - | (2)
     - | Calculate the last date up to which the tour can be cancelled.
   * - | (3)
     - | Check if today’s date is later than the last date for cancellation.
   * - | (4)
     - | \ ``BusinessException`` \  is thrown if the date exceeds the last date for cancellation.

|

**JUnit source**

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

   * - Sr. No.
     - Description
   * - | (5)
     - | Set the departure date to 2012/10/10 in the tour reservation information to be fetched from Repository class.
   * - | (6)
     - | Set the Return value of dateFactory.newDateTime() to 2012/10/1.
   * - | (7)
     - | Execute Cancel. Cancellation is successful as the date is prior to the last date for cancellation.
   * - | (8)
     - | Return value of dateFactory.newDateTime() should be 2012/10/9.
   * - | (9)
     - | Execute Cancel. Cancellation fails as the date falls after the last date for cancellation.

|

Integration Test
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In Integration Test, there may be cases wherein data of several days (for example: files) is created and transferred in a single day, for communicating with the system.

.. figure:: ./images/DateFactoryIT.png
   :alt: DateFactorySI
   :width: 90%

When the actual date is 2012/10/1, 
Use \ `JdbcAdjustedJodaTimeDateFactory` \ and set the SQL to calculate the difference with test execution date.


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | 1
     - | Set the difference between 9:00-11:00 as "0 days" and return value of Date Factory as 2012/10/1.
   * - | 2
     - | Set the difference between 11:00-13:00 as "0 days" and return value of Date Factory as 2012/10/10.
   * - | 3
     - | Set the difference between 13:00-15:00 as "30 days" and return value of Date Factory as 2012/10/31.
   * - | 4
     - | Set the difference between 15:00-17:00 as "31 days" and return value of Date Factory as 2012/11/1.

Date can be changed only by changing the table value.

|

System Test
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In System Test, testing may be carried out by creating test scenarios assuming the operation date.

.. figure:: ./images/DateFactoryST.png
   :alt: DateFactoryPT
   :width: 90%

Use \ `JdbcAdjustedJodaTimeDateFactory` \  and set SQL that calculates the date difference. 
Create a mapping table for actual date and operation date like 1, 2, 3 and 4 as shown in the figure. Testing can be carried out on the desired date, only by changing the difference value in the table.

|

Production
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

By setting the difference value to '0' using \ `JdbcAdjustedJodaTimeDateFactory`\, 
the return value of Date Factory can be set to the date same as the actual date without changing the source. 
Even the bean definition file need not be changed from System Test onwards. 
Further, even if the need to change date and time arises, return value of Date Factory can be changed by changing the table value.

.. warning::

    When using in Production environment, verify that the difference value in the table used in Production environment is 0.

    **Configuration example**

    - 窶｢When using the table for the first time in Production environment,
        - INSERT INTO operation_date (diff) VALUES (0);
    - 窶｢When test execution is completed in Production environment
        - UPDATE operation_date SET diff=0;

    **Always,** :ref:`useCache<useCache>` **should be set to 'true'.**

When there is no change in time, it is recommended to change the configuration file to \ `DefaultJodaTimeDateFactory`\.

.. raw:: latex

   \newpage

