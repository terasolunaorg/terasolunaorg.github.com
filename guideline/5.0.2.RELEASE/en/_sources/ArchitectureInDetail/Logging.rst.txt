Logging
================================================================================

.. only:: html

 .. contents::
    :depth: 3
    :local:

.. note::

  This contents described in this chapter are just guidelines that can be customized as per the business requirements.

Overview
--------------------------------------------------------------------------------

| When a system is being operated, logs and messages are output as information sources to analyze the business process usage and 
| to determine the causes when the system is down or when a business error etc. occurs.

| It is important to output the log since effectiveness of analysis improves significantly if it is output at the time of debugging.

| If only the behavior is to be checked, it can be done by debugging in IDE or through a simple output such as \ ``System.out.println``\ .
| However, if the output result is not manually stored, it is not possible to check the results later. This hampers the effectiveness of analysis.
| Obtaining the log by implementing logging library is just writing the code to be output.
| Log can be verified at any convenient time later on.
| Considering operating time, trail and analysis, it is recommended to implement logging library.

| There are various log output methods in Java and a number of methods can be selected; however this guideline recommends \ `SLF4J <http://www.slf4j.org/>`_ (interface) + `Logback <http://logback.qos.ch/>`_\  (implementation)
| due to simplicity of coding, ease of modification and performance.


Types of Logs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| A typical log at the time of application development is shown below.

.. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.35\linewidth}|p{0.40\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 15 35 40

   * - Log level
     - Category
     - Purpose of output
     - Output contents
   * - TRACE
     - Performance log
     - | Measurement of request processing time
       | (Should not be output during production environment operations)
     - | Process start and end time, process elapsed time (ms),
       | Information that can identify the execution process etc. (execution controller + method, request URL etc.).
   * - DEBUG
     - Debug log
     - | Debug at the time of development
       | (Should not be output during production environment operations)
     - Optional (Executed query, Input parameter, Return value etc.)
   * - INFO
     - Access log
     - | Assessing business process volume
     - | Information that can identify access date and time, user (IP address, authentication information)
       | Information that can identify execution process (request URL) etc., information for leaving a trail
   * - INFO
     - External communication log
     - | Analysis of error that occurs while communicating with external system
     - Send and receive time, send and receive data etc.
   * - WARN
     - Business error log
     - Business error records
     - | Business error occurrence time, Message ID and message corresponding to business error
       | Information necessary for analysis such as input information, exception message etc.
   * - ERROR
     - System error log
     - Record the events where it is difficult to continue a system operation
     - | System error occurrence time, Message and message ID corresponding to system error
       | Information necessary for analysis such as input information, exception message etc.
       | Basically, framework is output and business process logic is not output.
   * - ERROR
     - Monitoring log
     - Monitoring the occurrence of an exception
     - | Exception occurrence time, Message ID corresponding to system error
       | Use tools for monitoring and keep the output contents to a minimum.

| Debug log, Access log, Communication log, Business error log and System error log are output to same file.
| In this guideline, the log file that outputs the above mentioned logs is called as application log.

.. note::
    The sequence of log levels of SLF4J or Logback is TRACE < DEBUG < INFO < WARN < ERROR.
    It does not include FATAL level provided in commons-loggins or Log4J.


Output contents of log
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| It is important to note the points given below for the output contents of log.

1. | ID to be output to log
   | When log is to be monitored during the operation, it is recommended to include a message ID in the log to be monitored.
   | Further, when the business process volume is to be assessed using an access log, the log should be output according to the ID assigned to each business process as explained in Message Management.
   | This facilitates overall data compilation.

 .. note::

     The readability of log is enhanced by including an ID in the log thereby reducing the time required for primary isolation of failure analysis.
     Refer to \ :doc:`MessageManagement`\  for log ID structure.
     However, there is no need to assign an ID to all the logs. ID is not required at the time of debugging. It is recommended when the system is operational so as to isolate the log quickly.

     During failure, when a system user is notified by displaying a log ID (or a message ID) on the error screen and the ID is then notified to the call center, for that user,
     failure analysis becomes easier.

     However, note that the vulnerabilities of the system may be exposed if errors are displayed on the screen along with the failure details.

     In common library, the mechanism(component) is provided to include the message ID(exception code) into the log and the screen when an exception is occurred.
     Details refer to ":doc:`ExceptionHandling`".

2. | Traceability
   | To improve the traceability, it is recommended to output a unique track ID (hereafter referred to as X-Track) at request level in each log.
   | Example of logs including X-Track is given below.

 .. code-block:: console

    date:2013-09-06 19:36:31	X-Track:85a437108e9f4a959fd227f07f72ca20	message:[START CONTROLLER] (omitted)
    date:2013-09-06 19:36:31	X-Track:85a437108e9f4a959fd227f07f72ca20	message:[END CONTROLLER  ] (omitted)
    date:2013-09-06 19:36:31	X-Track:85a437108e9f4a959fd227f07f72ca20	message:[HANDLING TIME   ] (omitted)
    date:2013-09-06 19:36:33	X-Track:948c8b9fd04944b78ad8aa9e24d9f263	message:[START CONTROLLER] (omitted)
    date:2013-09-06 19:36:33	X-Track:142ff9674efd486cbd1e293e5aa53a78	message:[START CONTROLLER] (omitted)
    date:2013-09-06 19:36:33	X-Track:142ff9674efd486cbd1e293e5aa53a78	message:[END CONTROLLER  ] (omitted)
    date:2013-09-06 19:36:33	X-Track:142ff9674efd486cbd1e293e5aa53a78	message:[HANDLING TIME   ] (omitted)
    date:2013-09-06 19:36:33	X-Track:948c8b9fd04944b78ad8aa9e24d9f263	message:[END CONTROLLER  ] (omitted)
    date:2013-09-06 19:36:33	X-Track:948c8b9fd04944b78ad8aa9e24d9f263	message:[HANDLING TIME   ] (omitted)

\

   | Logs can be linked together using Track IDs even when the output is irregular.
   | In the above example, it can be clearly understood that 4th, 8th and 9th rows in the log are pertaining to the same request.
   | In common library, \ ``org.terasoluna.gfw.web.logging.mdc.XTrackMDCPutFilter``\  to be added to MDC is provided by generating a unique key for each request.
   | \ ``XTrackMDCPutFilter``\  sets Track ID in "X-Track" of HTTP response header as well. X-Track is used as a Track ID label in the log.
   | Refer to \ :ref:`About MDC <log_MDC>`\  for the usage methods.

3. | Log mask
   | If personal information, credit card number etc. are output to the log file as is, the information that has security threat should be masked if needed.

Log output points
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. tabularcolumns:: |p{0.15\linewidth}|p{0.85\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 15 85

   * - Category
     - Output points  
   * - | Performance log
     - | The processing time of business process is measured and it is output after executing business process. Request processing time is measured and log is output when response is returned.
       | It is usually implemented in AOP or Servlet filter.
       |
       | Common library provides \ ``org.terasoluna.gfw.web.logging.TraceLoggingInterceptor``\  which outputs processing time of SpringMVC Controller method
       | in TRACE log after the execution of processing method of Controller.
   * - | Debug log
     - | When it is necessary to output debug information at the time of development, a suitable log output process is implemented in source code.
       |
       | Common library provides \ ``org.terasoluna.gfw.web.logging.HttpSessionEventLoggingListener``\  listener which outputs DEBUG log at the time of HTTP session creation/destruction/attribute addition.
   * - | Access log
     - | INFO log is output at the time of receiving a request and returning the response.
       | It is usually implemented in AOP or Servlet filter.
   * - | External communication log
     - | INFO log is output before and after external system linking.
   * - | Business error log
     - | WARN log is output when business process exception is thrown.
       | It is usually implemented in AOP.
       |
       | In common library, when \ `org.terasoluna.gfw.common.exception.BusinessException`\  is thrown at the time of business process execution, \ ``org.terasoluna.gfw.common.exception.BusinessExceptionLoggingInterceptor``\  that outputs WARN log is provided.
       | Refer to :doc:`../ArchitectureInDetail/ExceptionHandling` for details.
   * - | System error log
     - | An ERROR log is output when system exception or unexpected exception occurs.
       | It is usually implemented in AOP or Servlet filter.
       |
       | In common library, \ ``org.terasoluna.gfw.web.exception.HandlerExceptionResolverLoggingInterceptor``\  and
       | \ ``org.terasoluna.gfw.web.exception.ExceptionLoggingFilter``\  are provided.
       | Refer to \ :doc:`../ArchitectureInDetail/ExceptionHandling` \  for details. 
   * - Monitoring log
     - It is output at the same time as business error log and system error log.

.. note:: 
    Note that when the log is output, the contents should not be exactly identical to other logs. This is helpful in easily identifying the location of the output.

|

How to use
--------------------------------------------------------------------------------

The following are required to output the log in SLF4J + Logback.

#. Settings of Logback
#. Calling API of SLF4J



Settings of Logback
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Logback settings are described in logback.xml under the class path. An example of configuration is shown below.
| Refer to \ `Logback Official Manual -Logback Configuration- <http://logback.qos.ch/manual/configuration.html>`_\  for the detailed configuration of logback.xml.

.. note::

     Settings of Logback are read automatically as per the rules given below.

     #. logback.groovy on class path
     #. If file "1" is not found, logback-test.xml on class path
     #. If file "2" is not found, logback.xml on class path
     #. If file "3" is not found, settings of class which implements \ ``com.qos.logback.classic.spi.Configurator``\  interface (Specify a implementation class using \ `ServiceLoader <http://docs.oracle.com/javase/8/docs/api/java/util/ServiceLoader.html>`_\  mechanism)
     #. If class which implements \ ``Configurator``\  interface is not found, settings of BasicConfigurator class (console output)

     In this guideline, it is recommended to place logback.xml in the class path.
     Moreover, apart from automatic reading, \ `it is possible to read programmatically through an API <http://logback.qos.ch/manual/configuration.html#joranDirectly>`_\  
     and \ `specify the configuration file in system properties  <http://logback.qos.ch/manual/configuration.html#configFileProperty>`_\ .


logback.xml

.. code-block:: xml

  <?xml version="1.0" encoding="UTF-8"?>
  <configuration>

      <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender"> <!-- (1) -->
          <encoder>
              <pattern><![CDATA[date:%d{yyyy-MM-dd HH:mm:ss}\tthread:%thread\tX-Track:%X{X-Track}\tlevel:%-5level\tlogger:%-48logger{48}\tmessage:%msg%n]]></pattern> <!-- (2) -->
          </encoder>
      </appender>

      <appender name="APPLICATION_LOG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender"> <!-- (3) -->
          <file>${app.log.dir:-log}/projectName-application.log</file> <!-- (4) -->
          <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
              <fileNamePattern>${app.log.dir:-log}/projectName-application-%d{yyyyMMddHH}.log</fileNamePattern> <!-- (5) -->
              <maxHistory>7</maxHistory> <!-- (6) -->
          </rollingPolicy>
          <encoder>
              <charset>UTF-8</charset> <!-- (7) -->
              <pattern><![CDATA[date:%d{yyyy-MM-dd HH:mm:ss}\tthread:%thread\tX-Track:%X{X-Track}\tlevel:%-5level\tlogger:%-48logger{48}\tmessage:%msg%n]]></pattern>
          </encoder>
      </appender>

      <appender name="MONITORING_LOG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender"> <!-- (8) -->
          <file>${app.log.dir:-log}/projectName-monitoring.log</file>
          <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
              <fileNamePattern>${app.log.dir:-log}/projectName-monitoring-%d{yyyyMMdd}.log</fileNamePattern>
              <maxHistory>7</maxHistory>
          </rollingPolicy>
          <encoder>
              <charset>UTF-8</charset>
              <pattern><![CDATA[date:%d{yyyy-MM-dd HH:mm:ss}\tX-Track:%X{X-Track}\tlevel:%-5level\tmessage:%msg%n]]></pattern>
          </encoder>
      </appender>

      <!-- Application Loggers -->
      <logger name="com.example.sample"> <!-- (9) -->
          <level value="debug" />
      </logger>

      <!-- TERASOLUNA -->
      <logger name="org.terasoluna.gfw">
          <level value="info" />
      </logger>
      <logger name="org.terasoluna.gfw.web.logging.TraceLoggingInterceptor">
          <level value="trace" />
      </logger>
      <logger name="org.terasoluna.gfw.common.exception.ExceptionLogger">
          <level value="info" />
      </logger>
      <logger name="org.terasoluna.gfw.common.exception.ExceptionLogger.Monitoring" additivity="false"><!-- (10) -->
          <level value="error" />
          <appender-ref ref="MONITORING_LOG_FILE" />
      </logger>

      <!-- 3rdparty Loggers -->
      <logger name="org.springframework">
          <level value="warn" />
      </logger>

      <logger name="org.springframework.web.servlet">
          <level value="info" />
      </logger>

      <!--  REMOVE THIS LINE IF YOU USE JPA
      <logger name="org.hibernate.engine.transaction">
          <level value="debug" />
      </logger>
            REMOVE THIS LINE IF YOU USE JPA  -->
      <!--  REMOVE THIS LINE IF YOU USE MyBatis3
      <logger name="org.springframework.jdbc.datasource.DataSourceTransactionManager">
          <level value="debug" />
      </logger>
            REMOVE THIS LINE IF YOU USE MyBatis3  -->

      <logger name="jdbc.sqltiming">
          <level value="debug" />
      </logger>

      <!-- only for development -->
      <logger name="jdbc.resultsettable">
          <level value="debug" />
      </logger>

      <root level="warn"> <!-- (11) -->
          <appender-ref ref="STDOUT" /> <!-- (12) -->
          <appender-ref ref="APPLICATION_LOG_FILE" />
      </root>

  </configuration>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Definition of appender is specified to output the log on console.
       | It can be selected whether the output destination is standard output or standard error, however when the destination is not specified, it is considered as standard output.
   * - | (2)
     - | The output format for log is specified. If the format is not specified, the message alone is output.
       | Time and message level etc. are output according to the business requirements.
       | Here, LTSV (Labeled Tab Separated Value) of "Label:Value<TAB>Label:Value<TAB>..." format is set.
   * - | (3)
     - | Definition for appender is specified to output application log.
       | The appender to be used can also be specified in <logger>, however, here it is referred to root (11) since application log is used by default.
       | RollingFileAppender is commonly used at the time of application log output, however, FileAppender may also be used to implement log rotation using another functions such as logrotate.
   * - | (4)
     - | A current file name (File name of log being output) is specified. It should be specified when it is necessary to specify a fixed file name.
       | If <file>log file name</file> is not specified, it is output with the name in pattern (5).
   * - | (5)
     - | Name of the file after rotation is specified. Usually, date or time format is widely used.
       | Note that 24 hour clock is not set if HH is mistakenly set as hh.
   * - | (6)
     - | The number of remaining files for which rotation is performed is specified. 
   * - | (7)
     - | A character code of log file is specified.
   * - | (8)
     - | It is set so as to output the application log by default.
   * - | (9)
     - | The logger name is set so that logger under com.example.sample outputs the log above debug level.
   * - | (10)
     - | A monitoring log is set. Refer to \ :ref:`exception-handling-how-to-use-application-configuration-common-label`\  of \ :doc:`ExceptionHandling`\ .

       .. warning:: **About additivity setting value**

           Specify \ ``false``\ . If \ ``true``\ (default value) is specified, the same log will be output by upper level logger (for example, root).
           Concretely speaking, the monitoring log will be output using three appenders(\ ``MONITORING_LOG_FILE``\, \ ``STDOUT``\  and \ ``APPLICATION_LOG_FILE``\).

   * - | (11)
     - | It is set such that logger without <logger> specification outputs the log of warn level or above.
   * - | (12)
     - | It is set in such a way that ConsoleAppender, RollingFileAppender (application logs) are used by default.

.. tip:: **About LTSV(Labeled Tab Separated Value)**

    \ `LTSV <http://ltsv.org/>`_\  is one of text data formats, and mainly used as the log format.

    For log fomart, LTSV is easy to parse using some tools because it has following features.

    * It's easy to split the field compared to other delimiters because tabs is used as field delimiters.
    * Even if the field definition (changing the position of field or adding the field or removing the field) is changed, it does not affect to parsing because of including a label(name) in the field.

    It is also one of features that there are that pasting on the Excel can format it with the least effort.

|

The following three items should be set in logback.xml.

.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 80

   * - Type
     - Overview
   * - appender
     - "Location" and "Layout" to be used for output
   * - root
     - Default "Appender" and the minimum "Log level" to be used for output
   * - logger
     - "Which logger (package or class etc.)" is to be output at which minimum "log level"

|

In <appender> element, the "location" and the "layout" to be used for output are defined.
It is not used at the time of the log output only by defining the appender.
It is used for the first time when it is referred in <logger> element or <root> element.
There are two attributes, namely, "name" and "class" and both are mandatory.

.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 80

   * - Attribute
     - Overview
   * - name
     - Name of the appender. It is specified by appender-ref. Any name can be assigned as there is no restriction on assigning the name.
   * - class
     - FQCN of appender implementation class.

|

The main appenders that are provided are shown below.

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 30 70

   * - Appender
     - Overview
   * - `ConsoleAppender <http://logback.qos.ch/manual/appenders.html#ConsoleAppender>`_
     - Console output
   * - `FileAppender <http://logback.qos.ch/manual/appenders.html#FileAppender>`_
     - File output
   * - `RollingFileAppender <http://logback.qos.ch/manual/appenders.html#RollingFileAppender>`_
     - File output (Rolling possible)
   * - `AsyncAppender <http://logback.qos.ch/manual/appenders.html#AsyncAppender>`_
     - Asynchronous output. It is used for logging in processes with high performance requirement. (It is necessary to set the output destination in other Appender.)

Refer to \ `Logback Official Manual -Appenders- <http://logback.qos.ch/manual/appenders.html>`_\  for detailed Appender types.

|

Basic log output by calling API of SLF4J
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Log is output by calling a method according to each log level of SLF4J logger(\ ``org.slf4j.Logger``\ ).

.. code-block:: java

    package com.example.sample.app.welcome;

    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;

    @Controller
    public class HomeController {

        private static final Logger logger = LoggerFactory
                .getLogger(HomeController.class);   // (1)

        @RequestMapping(value = "/", method = { RequestMethod.GET,
                RequestMethod.POST })
        public String home(Model model) {
            logger.trace("This log is trace log."); // (2)
            logger.debug("This log is debug log."); // (3)
            logger.info("This log is info log.");   // (4)
            logger.warn("This log is warn log.");   // (5)
            logger.error("This log is error log."); // (6)
            return "welcome/home";
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr. No.
     - Description
   * - | (1)
     - | \ ``Logger``\  is generated from \ ``org.slf4j.LoggerFactory``\ . If Class object is set as an argument of \ ``getLogger``\ , the logger name acts as an FQCN of that class.
       | In this example, the logger name is "com.example.sample.app.welcome.HomeController".
   * - | (2)
     - | The log of TRACE level is output.
   * - | (3)
     - | The log of DEBUG level is output.
   * - | (4)
     - | The log of INFO level is output.
   * - | (5)
     - | The log of WARN level is output.
   * - | (6)
     - | The log of ERROR level is output.


Log output results are shown below. Log level of com.example.sample is DEBUG, hence TRACE log is not output.

.. code-block:: xml

    date:2013-11-06 20:13:05    thread:tomcat-http--3 X-Track:5844f073b7434b67a875cb85b131e686    level:DEBUG logger:com.example.sample.app.welcome.HomeController    message:This log is debug log.
    date:2013-11-06 20:13:05    thread:tomcat-http--3 X-Track:5844f073b7434b67a875cb85b131e686    level:INFO  logger:com.example.sample.app.welcome.HomeController    message:This log is info log.
    date:2013-11-06 20:13:05    thread:tomcat-http--3 X-Track:5844f073b7434b67a875cb85b131e686    level:WARN  logger:com.example.sample.app.welcome.HomeController    message:This log is warn log.
    date:2013-11-06 20:13:05    thread:tomcat-http--3 X-Track:5844f073b7434b67a875cb85b131e686    level:ERROR logger:com.example.sample.app.welcome.HomeController    message:This log is error log.

The description can be as given below when an argument is to be entered in placeholder of a log message.

.. code-block:: java

    int a = 1;
    logger.debug("a={}", a);
    String b = "bbb";
    logger.debug("a={}, b={}", a, b);

The log given below is output.


.. code-block:: xml

    date:2013-11-06 20:32:45    thread:tomcat-http--3   X-Track:853aa701a401404a87342a574c69efbc    level:DEBUG logger:com.example.sample.app.welcome.HomeController    message:a=1
    date:2013-11-06 20:32:45    thread:tomcat-http--3   X-Track:853aa701a401404a87342a574c69efbc    level:DEBUG logger:com.example.sample.app.welcome.HomeController    message:a=1, b=bbb

.. warning::

     Note that string concatenation such as \ ``logger.debug("a=" + a + " , b=" + b);``\  should not be carried out.

When the exception is to be caught,
ERROR log (WARN log in some cases) is output as shown below. Error message and exception generated are passed to the log method.

.. code-block:: java

    public String home(Model model) {
        // omitted

        try {
            throwException();
        } catch (Exception e) {
            logger.error("Exception happened!", e);
            // omitted
        }
        // omitted
    }

    public void throwException() throws Exception {
        throw new Exception("Test Exception!");
    }

Accordingly, stack trace of caused exception is output and the cause of the error can be easily analyzed.

.. code-block:: xml

    date:2013-11-06 20:38:04    thread:tomcat-http--5   X-Track:11d7dbdf64e44782822c5aea4fc4bb4f    level:ERROR logger:com.example.sample.app.welcome.HomeController    message:Exception happend!
    java.lang.Exception: Test Exception!
        at com.example.sample.app.welcome.HomeController.throwException(HomeController.java:40) ~[HomeController.class:na]
        at com.example.sample.app.welcome.HomeController.home(HomeController.java:31) ~[HomeController.class:na]
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.7.0_40]
        (omitted)

However, as shown below, when the exception that is caught is wrapped with other exception and is re-thrown at upper level, there is no need to output the log. This is because usually the error log is output at upper level.

.. code-block:: java

    try {
        throwException();
    } catch (Exception e) {
        throw new SystemException("e.ex.fw.9001", e);
        // no need to log
    }

\
 .. note::

     When cause exception is to be passed to a log method, a placeholder cannot be used. Only in this case,
     the message argument can be concatenated using a string.

       .. code-block:: java

           try {
               throwException();
           } catch (Exception e) {
               // NG => logger.error("Exception happend! [a={} , b={}]", e, a, b);
               logger.error("Exception happend! [a=" + a + " , b=" + b + "]", e);
               // omitted
           }


Points to be noted for the description of log output
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

SLF4J Logger internally checks the log level and outputs actual log only for the required levels.

Therefore, the log level check as given below is basically not necessary.

.. code-block:: java

    if (logger.isDebugEnabled()) {
        logger.debug("This log is Debug.");
    }

    if (logger.isDebugEnabled()) {
        logger.debug("a={}", a);
    }


However, the log level should be checked in the cases given below to prevent performance degradation.


#. When there are 3 or more arguments

    When arguments of log message are 3 or more, argument array should be passed in the API of SLF4J. Log level should be checked in order to
    to avoid the cost for generating an array and the array should be generated only when necessary.


    .. code-block:: java

        if (logger.isDebugEnabled()) {
            logger.debug("a={}, b={}, c={}", new Object[] { a, b, c });
        }

#. When it is necessary to call a method for creating an argument

    When it is necessary to call a method while creating an argument for the log message, the log level should be checked
    to avoid the method execution cost and the method should be executed only when necessary.

    .. code-block:: java

        if (logger.isDebugEnabled()) {
            logger.debug("xxx={}", foo.getXxx());
        }

|

Appendix
--------------------------------------------------------------------------------

.. _log_MDC:

Using MDC
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| A cross-sectional log can be output by using \ `MDC <http://logback.qos.ch/manual/mdc.html>`_\  (Mapped Diagnostic Context).
| Log traceability improves if same information (such as user name or unique request ID) is included in the log to be output in a request.

| MDC internally consists of a ThreadLocal map and sets value for the key. The value set in log can be output till it is removed.
| The value should be set at the beginning of the request and removed at the time of process termination.


Basic usage method
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| An example of using MDC is given below.

.. code-block:: java

    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.slf4j.MDC;

    public class Main {

        private static final Logger logger = LoggerFactory.getLogger(Main.class);

        public static void main(String[] args) {
            String key = "MDC_SAMPLE";
            MDC.put(key, "sample"); // (1)
            try {
                logger.debug("debug log");
                logger.info("info log");
                logger.warn("warn log");
                logger.error("error log");
            } finally {
                MDC.remove(key); // (2)
            }
            logger.debug("mdc removed!");
        }
    }


The value added to MDC can be output in log by defining the output format as
\ ``%X{key name}``\ format in \ ``<pattern>``\  of logback.xml.

.. code-block:: xml

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern><![CDATA[date:%d{yyyy-MM-dd HH:mm:ss}\tthread:%thread\tmdcSample:%X{MDC_SAMPLE}\tlevel:%-5level\t\tmessage:%msg%n]]></pattern>
        </encoder>
    </appender>

Execution results are as follows:

.. code-block:: xml

    date:2013-11-08 17:45:48    thread:main mdcSample:sample    level:DEBUG     message:debug log
    date:2013-11-08 17:45:48    thread:main mdcSample:sample    level:INFO      message:info log
    date:2013-11-08 17:45:48    thread:main mdcSample:sample    level:WARN      message:warn log
    date:2013-11-08 17:45:48    thread:main mdcSample:sample    level:ERROR     message:error log
    date:2013-11-08 17:45:48    thread:main mdcSample:  level:DEBUG     message:mdc removed!

\
 .. note::

    If \ ``MDC.clear()``\  is executed, all the added values are deleted.

Setting value in MDC using Filter
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""


| Common library provides \ ``org.terasoluna.gfw.web.logging.mdc.AbstractMDCPutFilter``\  as a base class to add/delete values from MDC using filter.
| Further, following are provided as implementation classes.

* \ ``org.terasoluna.gfw.web.logging.mdc.XTrackMDCPutFilter``\  to set an unique ID for each request in MDC
* \ ``org.terasoluna.gfw.security.web.logging.UserIdMDCPutFilter``\  to set authentication user name of Spring Security in MDC



| If individual values are to be added to MDC using Filter, it is desirable to implement \ ``AbstractMDCPutFilter``\ 
| based on implementation of \ ``org.terasoluna.gfw.web.logging.mdc.XTrackMDCPutFilter``\ .

How to use MDCFilter

Definition of MDCFilter is added to the filter definition of web.xml.

.. code-block:: xml

    <!-- omitted -->

    <!-- (1) -->
    <filter>
        <filter-name>MDCClearFilter</filter-name>
        <filter-class>org.terasoluna.gfw.web.logging.mdc.MDCClearFilter</filter-class>
    </filter>

    <filter-mapping>
        <filter-name>MDCClearFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!-- (2) -->
    <filter>
        <filter-name>XTrackMDCPutFilter</filter-name>
        <filter-class>org.terasoluna.gfw.web.logging.mdc.XTrackMDCPutFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>XTrackMDCPutFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!-- (3) -->
    <filter>
        <filter-name>UserIdMDCPutFilter</filter-name>
        <filter-class>org.terasoluna.gfw.security.web.logging.UserIdMDCPutFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>UserIdMDCPutFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!-- omitted -->


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr. No.
     - Description
   * - | (1)
     - | \ ``MDCClearFilter``\  that clears the contents of MDC is set.
       | Values in MDC added by various \ ``MDCPutFilter``\  are deleted by this Filter.
   * - | (2)
     - | \ ``XTrackMDCPutFilter``\  is set. \ ``XTrackMDCPutFilter``\  sets Request ID in key \ "X-Track"\ .
   * - | (3)
     - | \ ``UserIdMDCPutFilter``\  is set. \ ``UserIdMDCPutFilter``\  sets User ID in key \ "USER"\ .
       |

As shown in the sequence diagram below, \ ``MDCClearFilter``\  should be defined prior to each \ ``MDCPutFilter``\ 
to clear the contents of MDC for post-processing.

.. figure:: ./images_Logging/logging-mdcput-sequence.png
   :width: 80%


Request ID and User ID can be output to log by adding \ ``%X{X-Track}``\  and \ ``%X{USER}``\  in \ ``<pattern>``\  of logback.xml.

.. code-block:: xml

    <!-- omitted -->
    <appender name="APPLICATION_LOG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${app.log.dir:-log}/projectName-application.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${app.log.dir:-log}/projectName-application-%d{yyyyMMdd}.log</fileNamePattern>
            <maxHistory>7</maxHistory>
        </rollingPolicy>
        <encoder>
            <charset>UTF-8</charset>
            <pattern><![CDATA[date:%d{yyyy-MM-dd HH:mm:ss}\tthread:%thread\tUSER:%X{USER}\tX-Track:%X{X-Track}\tlevel:%-5level\tlogger:%-48logger{48}\tmessage:%msg%n]]></pattern>
        </encoder>
    </appender>
    <!-- omitted -->

Example of log output

.. code-block:: xml

    date:2013-09-06 23:05:22  thread:tomcat-http--3   USER:   X-Track:97988cc077f94f9d9d435f6f76027428    level:DEBUG logger:o.t.g.w.logging.HttpSessionEventLoggingListener  message:SESSIONID#D7AD1D42D3E77D61DB64E7C8C65CB488 sessionCreated : org.apache.catalina.session.StandardSessionFacade@e51960
    date:2013-09-06 23:05:22  thread:tomcat-http--3   USER:anonymousUser  X-Track:97988cc077f94f9d9d435f6f76027428    logger:o.t.gfw.web.logging.TraceLoggingInterceptor      message:[START CONTROLLER] HomeController.home(Locale,Model)
    date:2013-09-06 23:05:22  thread:tomcat-http--3   USER:anonymousUser  X-Track:97988cc077f94f9d9d435f6f76027428    level:INFO  logger:c.terasoluna.logging.app.welcome.HomeController  message:Welcome home! The client locale is ja.
    date:2013-09-06 23:05:22  thread:tomcat-http--3   USER:anonymousUser  X-Track:97988cc077f94f9d9d435f6f76027428    logger:o.t.gfw.web.logging.TraceLoggingInterceptor      message:[END CONTROLLER  ] HomeController.home(Locale,Model)-> view=home, model={serverTime=2013/09/06 23:05:22 JST}
    date:2013-09-06 23:05:22  thread:tomcat-http--3   USER:anonymousUser  X-Track:97988cc077f94f9d9d435f6f76027428    logger:o.t.gfw.web.logging.TraceLoggingInterceptor      message:[HANDLING TIME   ] HomeController.home(Locale,Model)-> 36,508,860 ns

\
 .. note::

     User information to be set in MDC by \ ``UserIdMDCPutFilter``\  is created by Spring Security Filter.
     As mentioned earlier, if \ ``UserIdMDCPutFilter`` \  is defined in web.xml, user ID is output to
     log after the completion of a series of Spring Security processes. If user information is to be output to the log immediately after it is generated,
     the information should be incorporated in Spring Security Filter as shown below by deleting web.xml definition.


     The definitions given below are added to spring-security.xml.

         .. code-block:: xml

             <sec:http auto-config="true" use-expressions="true">
                 <!-- omitted -->
                 <sec:custom-filter ref="userIdMDCPutFilter" after="ANONYMOUS_FILTER"/> <!-- (1) -->
                 <!-- omitted -->
             </sec:http>

             <!-- (2) -->
             <bean id="userIdMDCPutFilter" class="org.terasoluna.gfw.security.web.logging.UserIdMDCPutFilter">
             </bean>


         .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
         .. list-table::
             :header-rows: 1
             :widths: 10 90


             * - Sr. No.
               - Description
             * - | (1)
               - | \ ``UserIdMDCPutFilter`` \  defined in Bean is added after "ANONYMOUS_FILTER".
             * - | (2)
               - | \ ``UserIdMDCPutFilter`` \  is defined.

     In blank project, \ ``UserIdMDCPutFilter``\  is defined in spring-security.xml.

Log output related functionalities provided by common library
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


.. _logging_appendix_httpsessioneventlogginglistener:

HttpSessionEventLoggingListener
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\  ``org.terasoluna.gfw.web.logging.HttpSessionEventLoggingListener``\  is a listener class
that outputs debug log at the time of generating, discarding, activating or deactivating session, and adding or deleting session attributes.

The following should be added to web.xml.

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <web-app xmlns="http://java.sun.com/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
        version="3.0">
        <listener>
            <listener-class>org.terasoluna.gfw.web.logging.HttpSessionEventLoggingListener</listener-class>
        </listener>

        <!-- omitted -->
    </web-app>


In logback.xml, \ ``org.terasoluna.gfw.web.logging.HttpSessionEventLoggingListener``\  is set at debug level as shown below.

.. code-block:: xml

    <logger
        name="org.terasoluna.gfw.web.logging.HttpSessionEventLoggingListener"> <!-- (1) -->
        <level value="debug" />
    </logger>


Debug log as shown below is output.

.. code-block:: xml

    date:2013-09-06 16:41:33	thread:tomcat-http--3	USER:	X-Track:c004ddb56a3642d5bc5f6b5d884e5db2	level:DEBUG	logger:o.t.g.w.logging.HttpSessionEventLoggingListener 	message:SESSIONID#EDC3C240A7A1CCE87146A6BA1321AD0F sessionCreated : org.apache.catalina.session.StandardSessionFacade@f00e0f

When the lifecycle of an object is managed using a Session such as \ ``@SessionAttribute``\  etc.,
it is strongly recommended to use this listener to confirm whether the attributes added to the session are deleted as anticipated.

TraceLoggingInterceptor
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\  ``org.terasoluna.gfw.web.logging.TraceLoggingInterceptor``\  is the 
\ ``HandlerInterceptor``\  to output start and termination of Controller process to the log.
When the process is terminated, View name returned by the Controller, attributes added to Model and the time required for Controller process are also output.


\ ``TraceLoggingInterceptor``\ is added in \ ``<mvc:interceptors>``\  of spring-mvc.xml as shown below.

.. code-block:: xml

    <mvc:interceptors>
        <!-- omitted -->
        <mvc:interceptor>
            <mvc:mapping path="/**" />
            <mvc:exclude-mapping path="/resources/**" />
            <bean
                class="org.terasoluna.gfw.web.logging.TraceLoggingInterceptor">
            </bean>
        </mvc:interceptor>
        <!-- omitted -->
    </mvc:interceptors>

| By default, WARN log is output if Controller process takes more than 3 seconds.
| When the threshold value is to be changed, it is specified in nano seconds in ``warnHandlingNanos``\  property.

The following settings should be performed if the threshold value is to be changed to 10 seconds (10 * 1000 * 1000 * 1000 nano seconds).

.. code-block:: xml
    :emphasize-lines: 8

    <mvc:interceptors>
        <!-- omitted -->
        <mvc:interceptor>
            <mvc:mapping path="/**" />
            <mvc:exclude-mapping path="/resources/**" />
            <bean
                class="org.terasoluna.gfw.web.logging.TraceLoggingInterceptor">
                <property name="warnHandlingNanos" value="#{10 * 1000 * 1000 * 1000}" />
            </bean>
        </mvc:interceptor>
        <!-- omitted -->
    </mvc:interceptors>


In logback.xml, \ ``org.terasoluna.gfw.web.logging.TraceLoggingInterceptor``\  is set at trace level as shown below.

.. code-block:: xml

    <logger name="org.terasoluna.gfw.web.logging.TraceLoggingInterceptor"> <!-- (1) -->
        <level value="trace" />
    </logger>

ExceptionLogger
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
``org.terasoluna.gfw.common.exception.ExceptionLogger``\  is provided as a logger when an exception occurs.

Refer to \ :ref:`exception-handling-how-to-use-label`\  of \ :doc:`ExceptionHandling`\  for the usage method.

.. raw:: latex

   \newpage

