Health Check
--------------------------------------------------------------------------------

.. only:: html

 .. contents:: Table of Contents
    :depth: 4
    :local:

|

.. _HealthCheckOverview:

Overview
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| This chapter explains about health check.

.. _HealthCheckOverview-Loadbalancer:


Load sharing and fallback of load balancer
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| A load balancer (hereafter referred to as LB) is used assuming that the Web system receives requests from a large number of users.
| A LB is a device to distribute the load on the Web system  by allocating requests to multiple servers and characterized in flexibly changing process efficiency of Web system by adding and deleting servers.
| Health check is a function to monitor operational status of each server allocating the requests. LB uses the health check function wherein it allocates the requests to the servers which are running normally and does not allocate requests to the servers which are malfunctioning. Accordingly, it is possible to continue the operation without stopping the Web system even when a failure occurs in a specific server. (it is called as a fallback)

| In the example below, LB manages 3 servers and allocates requests accordingly.

.. figure:: ./images/healthcheck-overview-flow.png
   :width: 100%

   **Picture - About Load Balancing**

| LB periodically sends requests to the server and monitors operational status of the server by checking status code and response returned from the server. When an abnormality occurs in Server A of Fig., LB detects the same and does not allocate the request to server A.
| Client A which is originally connected to server A allocates the request to another server (here, server B) through LB.

.. figure:: ./images/healthcheck-overview-flow-failure.png
   :width: 100%

   **Picture - About Fallback**

Types of health checks
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Health checks performed by LB are of various types. Example is shown below.

.. figure:: ./images/healthcheck-overview-healthcheckFlow.png
   :width: 100%

   **Picture - HealthCheck Example**

|

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 20 70

   * - Sr. No.
     - Types of health checks
     - Details
   * - | (1)
     - | Health check in PING
     - | Verify operational status at the network layer level of OSI reference model. PING is sent for a server (OS) and determined as "server running" if appropriate response is received from the server.
   * - | (2)
     - | Health check in TCP/UDP
     - | Verify operational status in transport layer level of OSI reference model. A request is sent to TCP port (or UDP port) of Web/AP server and determined as "server running" if the appropriate response is received from  the server.
   * - | (3)
     - | Health check in application
     - | Verify operational status at the application layer level of OSI reference model. A HTTP request is sent to application running on Web/AP server and determined as "running" if the appropriate response is received from  the server.

| Verification is not possible for operational status of the application, in health check in PING or TCP/UDP. In case of Web application, the application must also be "running" along with running of server (OS) or Web/AP server.
| Hence, this guideline recommends a health check to be performed for the application.

.. _HealthCheckOverview-Implementation:

Structure of health check shown in the guideline
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| This guideline introduces an implementation example for the application wherein a health check is performed in the application.
| Basically, a handler consisting of the structure shown in Fig. is implemented which receives a request from LB.

.. figure:: ./images/healthcheck-overview-function.png
   :width: 100%

   **Picture - HealthCheck Configuration**

|

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Run Controller, Service and Repository by receiving a request from LB.
       | A method to perform  a simple health check also exists when only operational status is to be verified. However, this guideline also verifies whether the structure and framework used by the application is running correctly, using the health check and implements Controller, Service and Repository to achieve technological configuration of targeted application as much as possible.
   * - | (2)
     - | Issue SQL from Repository and verify that the database is running.
       | This is because, in an application with database access, an operation cannot be performed normally if an abnormality occurs in database, even if the application itself is running.
   * - | (3)
     - | Use JSP as a View which returns a response.
       | Although this guideline explains about JSP, communication method and response format should be changed appropriately in accordance with the characteristics of application, while using REST or SOAP. For details, refer \ :doc:`../../../ArchitectureInDetail/WebServiceDetail/REST`\ or \ :doc:`../../../ArchitectureInDetail/WebServiceDetail/SOAP`\.

| Status codes and responses returned by the implementation example of the guideline are as below.

.. tabularcolumns:: |p{0.25\linewidth}|p{0.30\linewidth}|p{0.30\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 25 30 30

   * - Health check process results
     - Status code
     - Response details
   * - | Success
     - | 200 (Normal)
     - | 3 characters of \ ``OK.``\
   * - | Error occurrence
     - | Status code set by exception handling function
     - | Response set by exception handling function

| For changing configuration of exception handling. refer \ :doc:`../../../ArchitectureInDetail/WebApplicationDetail/ExceptionHandling`\.


.. _HealthCheckHowToUse:

How to use
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| An implementation example shown in \ :ref:`HealthCheckOverview-Implementation`\ is explained.

.. _HealthCheckHowToUseRepository:

Repository interface
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| At first, \ ``HealthCheckRepository``\  is created. \ ``HealthCheckRepository``\  executes SQL for health check and verifies database operation.
| Note that, an example to access database by using MyBatis3 is explained here. For adopting other methods, refer \ :doc:`../../../ArchitectureInDetail/DataAccessDetail/index`\.

**HealthCheckRepository.java**

.. code-block:: java

   package com.example.domain.repository.healthcheck;
   
   public interface HealthCheckRepository {
      void healthcheck();
   }

| A requisite minimum SQL is configured since only whether database can be accessed correctly is to be verified here.
| In this guideline, SQL is configured so as to meet the following conditions.

* It should be a reference system
* It should not require parameters

| An example of mapping file while using PostgreSQL is given below.

**HealthCheckRepository.xml(when PostgreSQL is used)**

.. code-block:: xml

   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

   <mapper namespace="com.example.domain.repository.healthcheck.HealthCheckRepository">

      <select id="healthcheck" resultType="String">
         SELECT '1'
      </select>

   </mapper>

| Further, an example of mapping file while using Oracle is given below.

**HealthCheckRepository.xml(When Oracle is used)**

.. code-block:: xml

   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

   <mapper namespace="com.example.domain.repository.healthcheck.HealthCheckRepository">

      <select id="healthcheck" resultType="String">
         SELECT '1' FROM DUAL
      </select>

   </mapper>

.. _HealthCheckHowToUseService:

Service class
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Next, a \ ``HealthCheckService``\  interface and a \ ``HealthCheckServiceImpl``\  class which implements \ ``HealthCheckService``\  interface are created.
| \ ``HealthCheckServiceImpl``\  calls \ ``healthcheck``\  method of \ ``healthcheckRepository``\ and performs health check for the database.

**HealthCheckService.java**

.. code-block:: java

   package com.example.domain.service.healthcheck;

   public interface HealthCheckService {
      void healthcheck();
   }

**HealthCheckServiceImpl.java**

.. code-block:: java

   package com.example.domain.service.healthcheck;

   import healthcheck.domain.repository.healthcheck.HealthCheckRepository;
   
   import javax.inject.Inject;

   import org.springframework.stereotype.Service;
   import org.springframework.transaction.annotation.Transactional;

   @Service
   @Transactional(readOnly = true)
   public class HealthCheckServiceImpl implements HealthCheckService {
   
      @Inject
      HealthCheckRepository healthcheckRepository;
      
      @Override
      public void healthcheck() {
         healthcheckRepository.healthcheck();
      }
   }

.. _HealthCheckHowToUseController:

Controller class
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Next, a \ ``HealthCheckController``\  is created.
| \ ``healthcheck``\  method of \ ``HealthcheckService``\  is called and transition is done to the specified path based on execution results. When it is verified that "database is running", a view to display \ ``OK.``\  is returned.

**HealthCheckController.java**

.. code-block:: java
   
   package com.example.app.healthcheck;

   import healthcheck.domain.service.healthcheck.HealthCheckService;

   import javax.inject.Inject;

   import org.springframework.stereotype.Controller;
   import org.springframework.web.bind.annotation.RequestMapping;

   @Controller
   public class HealthCheckController {
   
      @Inject
      HealthCheckService healthcheckService;

      @RequestMapping(value = "healthcheck") // (1)
      public String healthcheck(){
         healthcheckService.healthcheck();
         return "common/healthcheck/ok"; // (2)
      }
   }

|

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | \ ``value``\ attribute acts as a URL for health check in order to verify operational status.
    * - | (2)
      - | The directory where JSP file is placed, is kept one layer deep so as not to receive configuration of Apache Tiles. For details, refer \ :ref:`HealthCheckAppendixAppatchTiles`\.

.. _HealthCheckHowToUseJsp:

JSP file
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Finally, a JSP file that is moved once the health check is successful, is created.
| While creating JSP file, ensure that a newline character is not inserted between \ ``<%@page>``\  directive and \ ``OK.``\  as shown below.
| This is to minimise the data volume for the response.

**ok.jsp**

.. code-block:: jsp

   <%@page contentType="text/plain; charset=utf-8" language="java" pageEncoding="utf-8" %>OK.

| Other precautions must also be taken along with minimising the data volume for the response. For details, refer \ :ref:`HealthCheckAppendixMinResponce`\.

.. _HealthCheckHowToUseSecurity:

Configuring access rights
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| A URL for health check must always be kept active by using authentication and authorization functions, during a health check process.
| For example, \ ``<sec:intercept-url>``\  of spring-security.xml is configured for enabling the access for any role.
| An example to perform  exclusion settings under \ ``/common/healthcheck``\  is shown below.
| For details, refer \ :doc:`../../../Security/Authorization`\.

**spring-security.xml**

.. code-block:: xml

   <sec:http>
      <sec:intercept-url pattern="/healthcheck/**" access="permitAll"/>
      <!-- omitted -->
   </sec:http>

.. note::

   If authorization control is removed, URL for health check can be accessed by anyone.
   Hence, countermeasures like using LB to prevent external access are required.

.. _HealthCheckAppendix:

Appendix
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _HealthCheckAppendixMinResponce:

Configuration for minimising the data volume for response
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| As shown in \ :ref:`HealthCheckHowToUseJsp`\, precautions must be primarily taken for following points in order to minimise data volume for response.

* \ :ref:`Apache Tiles settings should not be received<HealthCheckAppendixAppatchTiles>`\
* \ :ref:`Header file and footer file should not be read<HealthCheckAppendixHeader>`\
* \ :ref:`Additional line breaks are to be removed from the response<HealthCheckAppendixTrimWhitespace>`\

Respective implementation examples for above are shown below.

.. _HealthCheckAppendixAppatchTiles:

Should not receive settings of Apache Tiles
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| As shown in \ :ref:`HealthCheckHowToUseController`\, JSP file must be placed under a directory which does not conform to \ ``<put-attribute>``\  tag of tiles-definitions.xml so as not to receive Apache Tiles configuration.
| In the default configuration of blank project, since Apache Tiles configuration is applied to JSP corresponding to \ ``/WEB-INF/views/{1}/{2}.jsp``\, a directory which is a level deep in the hierarchy is created and JSP file is placed under \ ``/WEB-INF/views/common/healthcheck/``\.
| For details, refer \ :doc:`../../../ArchitectureInDetail/WebApplicationDetail/TilesLayout`\.

.. _HealthCheckAppendixHeader:

Prevent reading of header and footer files
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| Precautions must be taken to check the impact of \ ``<jsp-config>``\ tag of web.xml on JSP file.
| When \ ``<include-prelude>``\  and \ ``<include-coda>``\  tags are set in \ ``<jsp-config>``\  tag,
| header and footer files are read in JSP file. Check whether the configuration includes reading of header and footer files.
| Configuration example is shown below.

**web.xml**

.. code-block:: xml

    <jsp-config> 
        <jsp-property-group>
            <url-pattern>/WEB-INF/views/common/healthcheck/ok.jsp</url-pattern>
            <el-ignored>false</el-ignored>
            <page-encoding>UTF-8</page-encoding>
            <scripting-invalid>false</scripting-invalid>
            // (1)
        </jsp-property-group> 
    </jsp-config>

|

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | \ ``<include-prelude>``\  tag and \ ``<include-coda>``\  tag are not set in order to not to incorporate additional headers and footers. 

.. _HealthCheckAppendixTrimWhitespace:

Remove additional line breaks from response
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| For example, when \ ``<%@taglib>``\  directive is set at the beginning of JSP, in order to use tag library, additional line breaks are output at the beginning of response.
| Hence, \ ``trimDirectiveWhitespaces``\  attribute is set in \ ``<%@page>``\  directive and output of additional line breaks in ok.jsp is prevented.

**ok.jsp (while setting trimDirectiveWhitespaces attribute)**

.. code-block:: jsp

   <%@page contentType="text/plain; charset=utf-8" language="java" pageEncoding="utf-8" trimDirectiveWhitespaces="true" %>OK.

.. note::

   \ ``/WEB-INF/views/common/include.jsp``\  is configured in \ ``<include-prelude>``\ of web.xml, in the default configuration of Blank project.
   Hence, it is necessary to configure as shown above or modify \ ``<url-pattern>``\ so as not to include ok.jsp.
   If these actions are not taken, include.jsp is read by entire JSP file and line breaks are output in ok.jsp.

.. warning::

   When WebLogic is used, line breaks are not removed when an additional character exists before \ ``<%@page>``\ directive, even when \ ``trimDirectiveWhitespaces``\ attribute is configured as shown above.
   Hence, a different method should be used.
   As an example, how to configure a \ ``<trim-directive-whitespaces>``\  tag in \ ``<jsp-property-group>``\  tag of web.xml is shown below.
   
   **web.xml(while setting <trim-directive-whitespaces>  tag)**
   
    .. code-block:: xml

        <jsp-config> 
            <jsp-property-group>
                <url-pattern>/WEB-INF/views/common/healthcheck/ok.jsp</url-pattern> // (1)
                <el-ignored>false</el-ignored>
                <page-encoding>UTF-8</page-encoding>
                <scripting-invalid>false</scripting-invalid>
                <trim-directive-whitespaces>true</trim-directive-whitespaces> // (2)
            </jsp-property-group>
        </jsp-config>

    |

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90
        :class: longtable

        * - Sr. No.
          - Description
        * - | (1)
          - | Specify only ok.jsp in \ ``<url-pattern>``\  tag in order to apply \ ``<trim-directive-whitespaces>``\  tag only in ok.jsp.
        * - | (2)
          - | Remove additional line breaks from targeted jsp file (ok.jsp) by setting \ ``true``\  in \ ``<trim-directive-whitespaces>``\  tag.

.. raw:: latex

   \newpage

