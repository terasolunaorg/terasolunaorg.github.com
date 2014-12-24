Database access (Mybatis2)
================================================================================

.. only:: html

 .. contents:: Table of Contents
    :local:
    :depth: 3

Overview
--------------------------------------------------------------------------------

| This chapter describes how to access database using MyBatis2.x.
| This guideline presumes the use of TERASOLUNA DAO (Mybatis DAO wrapper).

 .. figure:: images/dataaccess_mybatis.png 
    :alt: Target of description
    :width: 100%
    :align: center

    **Picture - Target of description**


About Mybatis
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Mybatis is a type of O/R Mapper. The idea behind developing MyBatis is to use it for mapping the SQL statements to objects,
| rather than for mapping the database records to the objects.
| Thus, this O/R Mapper can be effectively used to access de-normalized databases or to obtain full control of SQL execution.
| For details of Mybatis2.x, refer to \ `Mybatis Developer Guide (PDF) <https://mybatis.googlecode.com/files/MyBatis-SqlMaps-2_en.pdf>`_\ .

|
About TERASOLUNA DAO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
TERASOLUNA DAO provides a DAO interface for hiding processes dependent on O/R Mapper and a DAO implementation class which uses Mybatis2.x.

DAO interfaces provided by TERASOLUNA DAO are as follows:

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table:: **DAO interfaces provided by TERASOLUNA DAO**
    :header-rows: 1
    :widths: 10 35 55
   
    * - Sr. No.
      - Class name
      - Description
    * - 1.
      - | jp.terasoluna.fw.dao.
        | QueryDAO
      - DAO interface for executing QuerySQL
    * - 2.
      - | jp.terasoluna.fw.dao.
        | UpdateDAO
      - DAO interface for executing UpdateSQL
    * - 3.
      - | jp.terasoluna.fw.dao.
        | StoredProcedureDAO
      - DAO interface for executing StoredProcedure
    * - 4.
      - | jp.terasoluna.fw.dao.
        | QueryRowHandleDAO
      - DAO interface to process each record fetched by executing QuerySQL.

Basic flow to access database using TERASOLUNA DAO (Mybatis implementation) is shown below.

 .. figure:: images/dataaccess_mybatis_basic_flow.png 
    :alt: Basic flow of TERASOLUNA DAO
    :width: 100%
    :align: center
 
    **Picture - Basic flow of TERASOLUNA DAO**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90
   
    * - Sr. No.
      - Description
    * - | (1)
      - | Call the DAO interface method of TERASOLUNA DAO, from Service or Repository.
        | Pass the object which holds the values to be embedded in SQLID and SQL, as the calling parameter of the method.
    * - | (2)
      - | TERASOLUNA DAO delegates the process to Mybatis API.
        | The object which holds the values to be embedded in SQL and the SQLID specified from Service or Repository, are also passed to Mybatis.
    * - | (3)
      - | Mybatis fetches the SQL corresponding to the specified SQLID from configuration file (\ ``sqlMap.xml``\), and passes the SQL and bind value to JDBC driver.
        | (\ ``java.sql.PreparedStatement``\  API is used for actual bind value).
    * - | (4)
      - JDBC driver executes the SQL by sending the bind value and passed SQL to database.


|
How to use
--------------------------------------------------------------------------------

pom.xml settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
To use MyBatis2 (TERASOLUNA DAO) in infrastructure layers, add the following dependency to pom.xml.

 .. code-block:: xml

    <!-- (1) -->
    <dependency>
        <groupId>org.terasoluna.gfw</groupId>
        <artifactId>terasoluna-gfw-mybatis2</artifactId>
    </dependency>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Add ``terasoluna-gfw-mybatis2`` that defines the library group relevant to MyBatis2, to dependency.

|
Application settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Datasource settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
For datasource settings, refer to the common version \ :ref:`data-access-common_howtouse_datasource`\ .

|

PlatformTransactionManager settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| To use local transactions, perform the following settings.

 When a local transaction is used, call the API of JDBC and use \ ``org.springframework.jdbc.datasource.DataSourceTransactionManager``\  that performs transaction control.

- xxx-env.xml

 .. code-block:: xml

     <bean id="transactionManager"
         class="org.springframework.jdbc.datasource.DataSourceTransactionManager"> <!-- (1) -->
         <property name="dataSource" ref="dataSource" /> <!-- (2) -->
     </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90
   
    * - Sr. No.
      - Description
    * - | (1)
      - Specify \ ``org.springframework.jdbc.datasource.DataSourceTransactionManager``\ .
    * - | (2)
      - Specify the bean of configured datasource.

|
To use the transaction manager provided by application server, perform the following settings.

 When transaction manager provided by application server is used, call the API of JTA and use \ ``org.springframework.transaction.jta.JtaTransactionManager``\  that performs transaction control.

- xxx-env.xml

 .. code-block:: xml

     <tx:jta-transaction-manager /> <!-- (1) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90
   
    * - Sr. No.
      - Description
    * - | (1)
      - |  Using  \ ``"transactionManager"``\  id, bean is defined for \ ``JtaTransactionManager``\ , which is the most appropriate transaction manager for the application server on which the application is deployed,

|
TERASOLUNA DAO settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Define the beans for factory class of \ ``SqlMapClient``\  provided by Spring Framework and TERASOLUNA DAO.

- xxx-infra.xml

 .. code-block:: xml

     <bean id="sqlMapClient"
         class="org.springframework.orm.ibatis.SqlMapClientFactoryBean"> <!-- (1) -->
         <property name="configLocations"
             value="classpath*:/META-INF/mybatis/config/*sqlMapConfig.xml" /> <!-- (2) -->
         <property name="mappingLocations"
             value="classpath*:/META-INF/mybatis/sql/**/*-sqlmap.xml" /> <!-- (3) -->
         <property name="dataSource" ref="dataSource" /> <!-- (4) -->
     </bean>

     <bean id="queryDAO"
         class="jp.terasoluna.fw.dao.ibatis.QueryDAOiBatisImpl"> <!-- (5) -->
         <property name="sqlMapClient" ref="sqlMapClient" /> <!-- (6) -->
     </bean>

     <!-- (5) (6) -->
     <bean id="updateDAO"
         class="jp.terasoluna.fw.dao.ibatis.UpdateDAOiBatisImpl">
         <property name="sqlMapClient" ref="sqlMapClient" />
     </bean>

     <!-- (5) (6) -->
     <bean id="spDAO"
         class="jp.terasoluna.fw.dao.ibatis.StoredProcedureDAOiBatisImpl">
         <property name="sqlMapClient" ref="sqlMapClient" />
     </bean>

     <!-- (5) (6) -->
     <bean id="queryRowHandleDAO"
         class="jp.terasoluna.fw.dao.ibatis.QueryRowHandleDAOiBatisImpl">
         <property name="sqlMapClient" ref="sqlMapClient" />
     </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90
   
    * - Sr. No.
      - Description
    * - | (1)
      - Specify \ ``org.springframework.orm.ibatis.SqlMapClientFactoryBean``\  as the factory class of \ ``SqlMapClient``\  class.
    * - | (2)
      - | Specify the location of Mybatis configuration file.
        | In this example, it is the file that ends with " \ ``sqlMapConfig.xml``\  " stored in "\ ``/META-INF/mybatis/config/``\ " directory in class path.
        | For configuration file, refer to \ :ref:`sqlmapconfig-label`\ .
    * - | (3)
      - | Specify the location of Mybatis SQL mapping file.
        | In this example, it is the file that ends with " \ ``-sqlmap.xml``\ " stored under "\ ``/META-INF/mybatis/sql/``\ " directory (including sub-directory) in class path.
        | For SQL mapping file, refer to \ :ref:`sqlmap-label`\ .
    * - | (4)
      - Specify the bean for the configured datasource.
    * - | (5)
      - Define the bean by specifying Mybatis implementation class of TERASOLUNA DAO.
    * - | (6)
      - Specify the bean for the factory class of \ ``SqlMapClient``\  class defined in (1).


Settings for LOB handling 
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
When Large Object like BLOB or CLOB is to be handled, specify \ ``LobHandler``\  in the factory class of ``SqlMapClient``\  class.

- xxx-infra.xml

 .. code-block:: xml

     <!-- (1) -->
    <bean id="nativeJdbcExtractor"
        class="org.springframework.jdbc.support.nativejdbc.SimpleNativeJdbcExtractor" />

    <!-- (2) -->
    <bean id="lobHandler" class="org.springframework.jdbc.support.lob.OracleLobHandler">
        <property name="nativeJdbcExtractor" ref="nativeJdbcExtractor" /> <!-- (3) -->
    </bean>

    <bean id="sqlMapClient"
        class="org.springframework.orm.ibatis.SqlMapClientFactoryBean">
        <property name="configLocations"
            value="classpath*:/META-INF/mybatis/config/*sqlMapConfig.xml" />
        <property name="mappingLocations"
            value="classpath*:/META-INF/mybatis/sql/**/*-sqlmap.xml" />
        <property name="dataSource" ref="dataSource" />
       <property name="lobHandler" ref="lobHandler" /> <!-- (4) -->
    </bean>


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90
   
    * - Sr. No.
      - Description
    * - | (1)
      - | Define bean for the implementation class of \ ``org.springframework.jdbc.support.nativejdbc.NativeJdbcExtractor``\  interface.
        | In this example, \ ``org.springframework.jdbc.support.nativejdbc.SimpleNativeJdbcExtractor``\  is specified.
        | However, at times, native datasource cannot be fetched in AP servers other than Tomcat.
        | Therefore, it is either necessary to specify another NativeJdbcExtractor provided by Spring or create a new \ ``NativeJdbcExtractor``\  for each AP server.
    * - | (2)
      - | Define bean for the implementation class of \ ``org.springframework.jdbc.support.lob.LobHandler``\  interface.
        | In this example, the \ ``org.springframework.jdbc.support.lob.OracleLobHandler``\  interface, which is specified when using Oracle is specified.
        | However, specify \ ``org.springframework.jdbc.support.lob.DefaultLobHandler``\  when Oracle is not used.
    * - | (3)
      - | Specify the bean for  \ ``NativeJdbcExtractor``\  defined in (1).
    * - | (4)
      - | Specify the bean for \ ``LobHandler``\  defined in (3).


.. _sqlmapconfig-label:

Mybatis settings
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Customize the default operations of \ ``SqlMapClient``\ . They should be customized as per the requirement.

- sqlMapConfig.xml

 .. code-block:: xml

     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE sqlMapConfig 
                 PUBLIC "-//ibatis.apache.org//DTD SQL Map Config 2.0//EN"
                 "http://ibatis.apache.org/dtd/sql-map-config-2.dtd"> <!-- (1) -->

     <sqlMapConfig>
         <settings useStatementNamespaces="true" /> <!-- (2) -->
     </sqlMapConfig>

 .. tabularcolumns:: |p{0.06\linewidth}|p{0.94\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 6 94
   
    * - Sr. No.
      - Description
    * - | (1)
      - Specify the DTD file. This will enable schema check and IDE code completion.
    * - | (2)
      - By setting \ ``useStatementNamespaces="true"``\ , namespace specified in SQL mapping file is used as SQLID.


- About sqlMapConfig child elements
 | \ ``properties``\ , \ ``settings``\ , \ ``resultObjectFactory``\ , \ ``typeAlias``\ , \ ``transactionManager``\  and \ ``sqlMap``\  are the child elements.
 | They should be set if required.
 | For details, refer to "The SQL Map XML Configuration File" (P.8-16) of Mybatis Developer Guide (PDF).

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table:: **child elements of sqlMapConfig**
    :header-rows: 1
    :widths: 10 20 70
   
    * - Sr. No.
      - Elements
      - Description
    * - 1.
      - properties
      - | Elements for reading a property file. Properties defined in the read property file
        | can be referred in a \ ``"${Property name}"``\  format from the Mybatis configuration file and SQL mapping file.
        | It is used while defining environment dependent values or common configuration values.
        | For details, refer to "The SQL Map XML Configuration File" (P.9) of Mybatis Developer Guide (PDF).
    * - 2.
      - settings
      - | Element for customizing default operations of \ ``SqlMapClient``\ .
        | For details of setting field, refer to "The SQL Map XML Configuration File" (P.9-11) of Mybatis Developer Guide (PDF).
    * - 3.
      - resultObjectFactory
      - | Element for specifying the factory class which generates the instance for class specified in the class attribute of resultMap element or the resultClass attribute of select element, statement element and procedure element of SQL mapping file.
        | When not specified, the instance generated by the default implementation method \ ``java.lang.Class#newInstance()``\  is used.
        | For details, refer to "The SQL Map XML Configuration File" (P.11-12) of Mybatis Developer Guide (PDF).
    * - 4.
      - typeAlias
      - | Element for assigning an alias (short name) to class name (FQCN).
        | The alias name defined here can be used at locations where Mybatis configuration file and SQL mapping file class are specified. Normally, a simple class name without the package name is specified.
        | For details, refer to "The SQL Map XML Configuration File" (P.12) of Mybatis Developer Guide (PDF).
    * - 5.
      - transactionManager
      - It is not necessary to define transaction manager since Spring Framework functions are used for transaction management.
    * - 6.
      - sqlMap
      - It is not necessary to define sqlMap since it is already set in TERASOLUNA DAO settings.


.. _sqlmap-label:

Implementing SQL mapping (Basic version)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Basic SQL mapping implementation is shown below.

Implement the SQL to be used in application.

- xxx-sqlmap.xml

 .. code-block:: xml

     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE sqlMap 
                 PUBLIC "-//ibatis.apache.org//DTD SQL Map 2.0//EN"
                 "http://ibatis.apache.org/dtd/sql-map-2.dtd"> <!-- (1) -->
 
     <sqlMap namespace="xxx"> <!-- (2) -->
     
         <!-- (3) -->
         <select id="findOne">
             <!-- ... -->
         </select>
         
         <!-- ... -->
     
     </sqlMap>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90
   
    * - Sr. No.
      - Description
    * - | (1)
      - Specify the DTD file. This will enable schema check and STS code completion.
    * - | (2)
      - Specify namespace.
    * - | (3)
      - It is set such that namespace is used as SQLID in \ ``sqlMapConfig.xml``\ . Hence, the SQLID specified for executing this SQL is "\ ``xxx.findOne``\ ".


- About sqlMap child elements
 \ ``cacheModel``\ , \ ``typeAlias``\ , \ ``parameterMap``\ , \ ``resultMap``\ , \ ``select``\ , \ ``insert``\ , \ ``update``\ , \ ``delete``\ , \ ``statement``\ , \ ``sql``\  and  \ ``procedure``\  are the child elements.
 
 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table:: **Child elements of sqlMap**
    :header-rows: 1
    :widths: 10 20 70

    * - Sr. No.
      - Elements
      - Description
    * - 1.
      - typeAlias
      - Same as \ ``typeAlias``\  of \ ``sqlMapConfig.xml``\ .
    * - 2.
      - cacheModel
      - Element defining an object cache
    * - 3.
      - parameterMap
      - Element defining the mapping of SQL bind parameters (object)
    * - 4.
      - resultMap
      - Element defining the mapping of records and objects returned as SQL execution result
    * - 5.
      - select
      - Element describing SELECT statement
    * - 6.
      - insert
      - Element describing INSERT statement
    * - 7.
      - update
      - Element describing UPDATE statement
    * - 8.
      - delete
      - Element describing DELETE statement
    * - 9.
      - statement
      - Generic elements including elements like \ ``select``\ , \ ``insert``\ , \ ``update``\ , \ ``delete``\  and \ ``procedure``\ . It is recommended to use individual elements (\ ``select``\ , \ ``insert``\ , \ ``update``\ , \ ``delete``\ , \ ``procedure``\ ).
    * - 10.
      - sql
      - Element describing the SQL statement (part of SQL statement) to be included from \ ``select``\ , \ ``insert``\ , \ ``update``\ , \ ``delete``\ , \ ``statement``\ . Effective use of these elements enables standardization of duplicate parts used in multiple SQLs.
    * - 11.
      - procedure
      - Element describing PROCEDURE call

 .. note ::
     For details, refer to the following chapters of Mybatis Developer Guide (PDF). 

     * | The SQL Map XML File (P.17-18)
       | Simple definition example of SQL mapping file is described.
     * | Mapped Statements (P.18-26)
       | Basic use of elements for building SQL is described.
     * | Parameter Maps and Inline Parameters (P.27-31)
       | Mapping of SQL bind parameters (objects) is described in detail.
     * | Substitution Strings (P.32)
       | SQL bind variables are described.
     * | Result Maps (P.32-41)
       | Mapping of records and objects returned as SQL execution result is described in detail.
     * | Supported Types for Parameter Maps and Result Maps (P.42-43)
       | The types and extension methods supported by ParameterMap and ResultMap are described.
     * | Caching Mapped Statement Results (P.44-47)
       | Cache is described in detail.
     * | Dynamic Mapped Statements (P.48-53)
       | Dynamic SQL is described in detail.
     * | Simple Dynamic SQL Elements (P.53)
       | Simple implementation of dynamic SQL is described.

 .. warning::

    Appropriate value should be set in fetchSize attribute to describe a query such as, returning a large volume of data using \ ``statement``\ , \ ``select``\  and \ ``procedure``\  elements.
    fetchSize attribute is the parameter that sets the number of data records which are retrieved in a single communication between JDBC driver and database.
    Default value of each JDBC driver is used when fetchSize attribute is omitted. However, care needs to be taken in case of JDBC driver wherein default value is used to fetch all the records, as it may result in memory exhaustion.


select element implementation
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Define mapping for the searched record column and JavaBean property before implementing select elements.

- xxx-sqlmap.xml

 .. code-block:: xml

     <resultMap id="resultMap_Todo"
                class="xxxxxx.yyyyyy.zzzzzz.domain.model.Todo"> <!-- (1) -->
         <result property="todoId" column="todo_id" /> <!-- (2) -->
         <result property="todoTitle" column="todo_title" />
         <result property="finished" column="finished" />
         <result property="createdAt" column="created_at" />
         <result property="version" column="version" />
     </resultMap>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 10 80

    * - Sr. No.
      - Attribute
      - Description
    * - | (1)
      - | -
      - Map the searched records and JavaBean. For details, refer to Developer Guide.
    * - |
      - | id
      - Specify the ID to identify mapping. It is referred from select attribute.
    * - |
      - | class
      - Specify the FQCN of JavaBean to be mapped.
    * - | (2)
      - | -
      - Map JavaBean property with searched record columns.
    * - |
      - | property
      - Specify JavaBean property name.
    * - |
      - | column
      - Specify the column name of the record to be mapped in property specified in "properties" attribute.


Implement select element.

- xxx-sqlmap.xml

 .. code-block:: xml

     <select id="findOne"
             parameterClass="java.lang.String"
             resultMap="resultMap_Todo"> <!-- (3) -->
         SELECT
             * 
         FROM
             todo 
         WHERE
             todo_id = #todoId#   /* (4) */
     </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 10 80

    * - Sr. No.
      - Attribute
      - Description
    * - | (3)
      - | -
      - Execute the Search SQL.
    * - |
      - | id
      - Specify the ID for identifying Search SQL.
    * - |
      - | parameterClass
      - | Specify the type of bind object.
        | In the example, \ ``java.lang.String``\  is specified. However, JavaBean can also be specified when multiple parameters (search conditions) are to be passed.
    * - |
      - | resultMap
      - | Specify the resultMap defined in (1).
        | It is possible to map automatically to the JavaBean properties specified in class attribute, without using resultMap. However, column name of fetched record should match with the JavaBean property name.
        | To match the column name of fetched record with JavaBean property name, an alias is assigned to the column using AS clause.
          For example, when SQL is set as \ ``"SELECT todo_title AS todoTitle, ..."``\ , a value is set to the todoTitle property of JavaBean.
    * - | (4)
      - | -
      - | Specify bind value in SQL.
        | In the example, since a single object ( \ ``java.lang.String``\  ) is used instead of JavaBean, any name can be specified for bind variable.
        | When JavaBean is used for bind object, bind variable name must match with the JavaBean property name.

 .. note:: **About automatic mapping**

     It is possible to map automatically to the JavaBean property specified in resultClass attribute, without using resultMap. However, the column name of fetched record must match with the JavaBean property name.
     To match the column name of fetched record with JavaBean property name, an alias is assigned to the column using AS clause. Implementation with auto mapping is shown below.

      .. code-block:: xml
         :emphasize-lines: 3,5-6,8

         <select id="findOne"
                 parameterClass="java.lang.String"
                 resultClass="xxxxxx.yyyyyy.zzzzzz.domain.model.Todo">
             SELECT
                 todo_id AS todoId,
                 todo_title AS todoTitle,
                 finished,
                 created_at AS createdAt,
                 version
             FROM
                 todo
             WHERE
                 todo_id = #todoId#
         </select>

     Auto mapping is the simplest method for mapping fetched records and JavaBean. However, following restrictions and precautions should be considered while using auto mapping,

    * Type declaration and conversion definition of the value fetched in SQL cannot be performed.
    * Complex mapping (for example, mapping to nested JavaBean) cannot be performed.
    * There is a slight degradation in performance as \ ``java.sql.ResultSetMetaData``\  is accessed during mapping.


"insert" element implementation
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Implement insert element.

- xxx-sqlmap.xml

 .. code-block:: xml

     <insert id="insert"
             parameterClass="xxxxxx.yyyyyy.zzzzzz.domain.model.Todo"> <!-- (1) -->
         INSERT INTO todo
             (
                 todo_id
                 ,todo_title
                 ,finished
                 ,created_at
                 ,version
             )
             values(
                 #todoId#       /* (2) */
                 ,#todoTitle#
                 ,#finished#
                 ,#createdAt#
                 ,1
             )
     </insert>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 10 80

    * - Sr. No.
      - Attribute
      - Description
    * - | (1)
      - | -
      - Implement Insert SQL.
    * - |
      - | id
      - Specify the ID to identify Insert SQL.
    * - |
      - | parameterClass
      - Specify the type of bind object. JavaBean can also be specified.
    * - | (2)
      - | -
      - Specify the bind value in SQL. When JavaBean is used for bind object, bind variable name must match with the JavaBean property name.

 .. note::

    Type declaration or conversion definition of SQL bind value can be performed by using the parameterMap attribute or "Inline Parameter Maps" feature.
    For example, when bind value is \ ``null``\ , default value can be set. For details, refer to "Parameter Maps and Inline Parameters" (P.27-31) of Mybatis Developer Guide (PDF).


"update" element implementation
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Implement update element.

- xxx-sqlmap.xml

 .. code-block:: xml

     <update id="update"
             parameterClass="xxxxxx.yyyyyy.zzzzzz.domain.model.Todo"> <!-- (1) -->
         UPDATE todo SET
             todo_id = #todoId#
             ,todo_title = #todoTitle#
             ,finished = #finished#
             ,version = (#version# + 1)
         WHERE
             todo_id = #todoId# 
         AND version = #version#
     </update>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Implement Update SQL.


"delete" element implementation
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Implement delete element.

- xxx-sqlmap.xml

 .. code-block:: xml

     <delete id="delete" parameterClass="java.lang.String">  <!-- (1) -->
         DELETE FROM
             todo
         WHERE
             todo_id = #todoId#
     </delete>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Implement Delete SQL.


"procedure" element implementation
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Calling the function created in PostgreSQL using procedure element is shown below.

SQL to create functions (PL/pgSQL implementation) and tables is as follows:

 .. code-block:: sql

    CREATE TABLE sales (
        itemno INT4 PRIMARY KEY,
        quantity INT4 NOT NULL,
        price INT4 NOT NULL
    );

 .. code-block:: guess

    CREATE
        FUNCTION sales_item(p_itemno INT4) RETURNS TABLE (
            quantity INT4
            ,total INT4
        ) AS $$ BEGIN RETURN QUERY
            SELECT
                    s.quantity
                    ,s.quantity * s.price
                FROM
                    sales s
                WHERE
                    itemno = p_itemno;
    END;
    $$ LANGUAGE plpgsql;

Implement parameterMap element.

 .. code-block:: xml

    <!-- (1) -->
    <parameterMap id="salesItemMap" class="xxxxxx.yyyyyy.zzzzzz.domain.model.SalesItem">
        <!-- (2) -->
        <parameter property="id" jdbcType="INTEGER" javaType="java.lang.Integer" mode="IN" />
        <!-- (3) -->
        <parameter property="quantity" jdbcType="INTEGER" javaType="java.lang.Integer" mode="OUT" />
        <parameter property="total" jdbcType="INTEGER" javaType="java.lang.Integer" mode="OUT" />
    </parameterMap>

 .. code-block:: java

    // (4)
    public class SalesItem implements Serializable {
        private Integer id;
        private Integer quantity;
        private Integer total;
        // ...
    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Define mapping of IN parameter and OUT parameter to be passed to the function.
    * - | (2)
      - Mapping of IN parameter is defined. \ ``SalesItem#id``\  is mapped with IN parameter.
    * - | (3)
      - Mapping of OUT parameters is defined. 1st OUT parameter is mapped with \ ``SalesItem#quantity``\  and the 2nd OUT parameter is mapped with \ ``SalesItem#total``\ .
    * - | (4)
      - JavaBean to be mapped.

 .. note::

    Mapping can also be carried out with  "Inline Parameter Maps" feature, without using parameterMap attribute.
    For specific examples, refer to "Parameter Maps and Inline Parameters" (P.31) of Mybatis Developer Guide(PDF).


Implement procedure element.

 .. code-block:: xml

    <procedure id="findSalesItem" parameterMap="salesItemMap"> <!-- (1) -->
        {call sales_item(?,?,?)}
    </procedure>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify the Procedure or Function to be called in "{call Procedure/Function name (IN parameter ...,OUT parameter...)}" format.
        | In the example, one IN parameter and two OUT parameters are specified for \ ``sales_item``\  function.
        | Bound values serve as a definition sequence of mapping definitions specified in parameterMap element.


"sql" element implementation
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Implement sql element.

- xxx-sqlmap.xml

 .. code-block:: xml

     <sql id="fragment_where_byFinished"> <!-- (1) -->
         WHERE
             finished = #finished#
     </sql>

     <select id="findByFinished"
             parameterClass="Boolean"
             resultMap="resultMap_Todo">  <!-- (2) -->
         SELECT
             *
         FROM
             todo 
         <include refid="fragment_where_byFinished" /> <!-- (3) -->
         ORDER BY
             created_at DESC
     </select>

     <select id="countByFinished"
             parameterClass="Boolean"
             resultClass="long"> <!-- (4) -->
         SELECT
             count(*)
         FROM
             todo 
         <include refid="fragment_where_byFinished" /> <!-- (5) -->
     </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - WHERE clause shared by the SQLs of (2) and (4) is defined. SQL to be included should be defined before the SQL that implements 'include'.
    * - | (2)
      - SQL for fetching the data that matches the condition.
    * - | (3)
      - Include the SQL implemented by WHERE clause defined in (1).
    * - | (4)
      - SQL for fetching data records that match the conditions
    * - | (5)
      - Include the SQL implemented by WHERE clause defined in (1).


.. _data-access-mybatis2_howtouse_lob_update:

LOB update implementation
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Implementation for updating Large Objects like BLOB and CLOB in database is shown below.
| In the following example, records are inserted in the table that handles BLOB.

- DDL

 .. code-block:: sql

    CREATE TABLE upload_binary (
        file_id CHAR(36) NOT NULL,
        file_name VARCHAR(256) NOT NULL,
        content BLOB NOT NULL, -- (1)
        CONSTRAINT pk_upload_binary PRIMARY KEY (file_id)
    );

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Define BLOB column.
        | In the above example, the DDL is assumed to use Oracle as a database.


- DTO (JavaBean)

 .. code-block:: java

    public class BinaryFile implements Serializable {
        // omitted

        private String fileId;
        private String fileName;
        private InputStream content; // (2)

        // omitted setter/getter

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (2)
      - | Define the property that stores BLOB value in \ ``java.io.InputStream``\  type.
        | In the above example, input stream of uploaded file is set in \ ``InputStream``\ .

 .. warning::

    It is recommended to always define the type of property that handles BLOB, in \ ``InputStream``\  type.
    BLOB can also be handled as a byte array. However, it may cause memory exhaustion in case of increased data capacity.

    It is recommended to always define the type of property that handles CLOB, in \ ``java.io.Reader``\  type.
    CLOB can also be handled as a string. However, it may cause memory exhaustion in case of increased data capacity.


- xxx-sqlmap.xml

 .. code-block:: xml

    <parameterMap id="uploadBinaryParameterMap"
                  class="xxxxxx.yyyyyy.zzzzzz.domain.service.BinaryFile">
        <parameter property="fileId" />
        <parameter property="fileName" />
        <!-- (3) -->
        <parameter property="content"
                   jdbcType="BLOB"
                   typeHandler="jp.terasoluna.fw.orm.ibatis.support.BlobInputStreamTypeHandler" />
    </parameterMap>

    <!-- (4) -->
    <insert id="uploadBinary" parameterMap="uploadBinaryParameterMap">
        INSERT INTO upload_binary
        (
            file_id
            ,file_name
            ,content
        )
        VALUES
        (
            ?
            ,?
            ,?
        )
    </insert>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (3)
      - | Specify the definition that is necessary to register the parameter that stores the registration values of BLOB column.
        | Specify \ ``"BLOB"``\  in jdbcType attribute and \ ``"jp.terasoluna.fw.orm.ibatis.support.BlobInputStreamTypeHandler"``\  in typeHandler attribute.
    * - | (4)
      - | SQL to register records in the table with a BLOB column.

 .. note::

    While handling CLOB, specify \ ``"CLOB"``\  in jdbcType attribute and \ ``"jp.terasoluna.fw.orm.ibatis.support.ClobReaderTypeHandler"``\  in typeHandler attribute.

 .. tip::

    Class name specified in FQCN can be described in a simple manner by assigning an alias to it using typeAlias element.

     .. code-block:: xml

        <!-- (5) -->
        <typeAlias alias="BinaryFile"
                   type="xxxxxx.yyyyyy.zzzzzz.domain.service.BinaryFile"/>
        <typeAlias alias="BlobInputStreamTypeHandler"
                   type="jp.terasoluna.fw.orm.ibatis.support.BlobInputStreamTypeHandler"/>

        <parameterMap id="uploadBinaryParameterMap"
                      class="BinaryFile"> <!-- (6) -->

            <!-- omitted -->

            <parameter property="content" jdbcType="BLOB"
                       typeHandler="BlobInputStreamTypeHandler" /> <!-- (6) -->

        </parameterMap>

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90

        * - Sr. No.
          - Description
        * - | (5)
          - | Assign an alias to the class name (FQCN) using typeAlias element.
            | In the above example, an alias is assigned to \ ``BinaryFile``\  class and \ ``BlobInputStreamTypeHandler``\  class.
            | typeAlias element can be defined in both \ :file:`sqlMapConfig.xml`\  and \ :file:`xxx-sqlmap.xml`\ .
        * - | (6)
          - | Specify the alias name for the class name (FQCN) assigned in (5).


- Service

 .. code-block:: java

    // omitted

    @Inject
    UpdateDAO updateDAO;

    // omitted

    public BinaryFile uploadBinaryFile(String fileName,
            InputStream contentInputStream) {

        // (7)
        BinaryFile inputFile = new BinaryFile();
        inputFile.setFileId(UUID.randomUUID().toString());
        inputFile.setFileName(fileName);
        inputFile.setContent(contentInputStream);

        // (8)
        updateDAO.execute("example.uploadBinary", inputFile);

        return inputFile;
    }

    // omitted

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (7)
      - | Set the information required for registering records in DTO.
        | In the above example, file ID is numbered as UUID and the file name received by argument and the  \ ``InputStream``\  object that stores the file contents are set in DTO.
    * - | (8)
      - | With DTO that stores information required for registration as the argument, call \ ``UpdateDAO``\ .
        | Method for calling DAO is same as the method when BLOB is not handled.


- Controller

 .. code-block:: java


    @RequestMapping("uploadBinary")
    public String uploadBinaryFile(
            @RequestPart("file") MultipartFile multipartFile, Model model) throws IOException {
        // (9)
        BinaryFile uploadedFile = uploadService.uploadBinaryFile(multipartFile
                .getOriginalFilename(), multipartFile.getInputStream());
        model.addAttribute(uploadedFile);
        return "upload/form";
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (9)
      - | Call Service method with uploaded file name and \ ``InputStream``\  that stores the file contents in argument.


Implementation for fetching LOB type
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Implementation for fetching Large Objects like BLOB and CLOB from database is shown below.
| In the following example, records are fetched from the table that handles BLOB.
| For details on DDL and DTO (JavaBean) that are used to create necessary tables, refer to \ :ref:`data-access-mybatis2_howtouse_lob_update`\ .

- xxx-sqlmap.xml

 .. code-block:: xml

    <resultMap id="selectBinaryResultMap" class="BinaryFile">
        <result property="fileId" column="file_id" />
        <result property="fileName" column="file_name" />
        <!-- (1) -->
        <result property="content" column="content" jdbcType="BLOB"
                typeHandler="BlobInputStreamTypeHandler" />
    </resultMap>

    <!-- (2) -->
    <select id="selectBinary" parameterClass="java.lang.String"
            resultMap="selectBinaryResultMap">
        SELECT
            *
        FROM
            upload_binary
        WHERE
            file_id = #fileId#
    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify the definition required to fetch the value for the property that stores the value fetched from BLOB column.
        | Specify \ ``"BLOB"``\  in jdbcType attribute and \ ``"jp.terasoluna.fw.orm.ibatis.support.BlobInputStreamTypeHandler"``\  in typeHandler attribute.
    * - | (2)
      - | SQL for fetching records from the table with BLOB column.

 .. note::

    While handling CLOB, specify \ ``"CLOB"``\  in jdbcType attribute and \ ``"jp.terasoluna.fw.orm.ibatis.support.ClobReaderTypeHandler"``\  in typeHandler attribute.


- Service / Repository

 .. code-block:: java

    // omitted

    @Inject
    QueryDAO queryDAO;

    // omitted

    public BinaryFile getBinaryFile(String fileId) {
        // (3)
        BinaryFile loadedFile = queryDAO.executeForObject(
                "article.selectBinary", fileId, BinaryFile.class);
        return loadedFile;
    }

    // omitted

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (3)
      - | With fetch conditions specified by the controller as argument, call \ ``QueryDAO``\ .
        | In the above example, upload file information matching with file ID is fetched.
        | Method for calling DAO is same as the method when BLOB is not handled.


SQL mapping implementation (dynamic SQL version)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| There is default mechanism provided in Mybatis to build SQL dynamically.
| Methods to build SQL dynamically are given below.
| For details, refer to "Dynamic Mapped Statements" (P.48-53) of Developer Guide (PDF).

Determining whether parameter object is specified
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
SQL can be built by determining whether the parameter object passed to SQL is specified.

Decision elements are as follows:


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
 .. list-table::
    :widths: 10 15 75
    :header-rows: 1

    * - Sr. No.
      - Element
      - Description
    * - 1.
      - isParameterPresent
      - Element for building SQL when parameter object is specified (Not NULL).
    * - 2.
      - isNotParameterPresent
      - Element for building SQL when parameter object is not specified (NULL).

Implementation is as follows:

 .. code-block:: xml

    <select id="findOne" parameterClass="java.lang.Integer" resultMap="...">
        SELECT
            *
        FROM
            t_order
        WHERE

        <isParameterPresent> <!-- (1) -->
            id = #id#
        </isParameterPresent>

        <isNotParameterPresent> <!-- (2) -->
            1 = 2
        </isNotParameterPresent>

        <!-- ... -->

    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | In the example, id column is set in WHERE clause when the parameter object is specified.
    * - | (2)
      - | In the example, the condition "\ ``1=2``\ " is set so that there are no matching records, when the parameter object is not specified.

Following 2 patterns of SQL are generated by the dynamic SQL mentioned above.

 .. code-block:: sql

    -- (1) parameterObject(id)=1
    SELECT * FROM t_order WHERE id = 1

    -- (2)
    SELECT * FROM t_order WHERE 1 = 2


Determining whether parameter object (JavaBean) properties exist
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
SQL can be built by determining whether the properties specified in parameter object (JavaBean) passed to SQL, exist.

Decision elements are as follows:

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
 .. list-table::
    :widths: 10 15 75
    :header-rows: 1

    * - Sr. No.
      - Element
      - Description
    * - 1.
      - isPropertyAvailable
      - Element for building SQL when the specified property exists.
    * - 2.
      - isNotPropertyAvailable
      - Element for building SQL when the specified property does not exist.

Implementation is as follows:

 .. code-block:: xml

    <select id="findOne" parameterClass="OrderCriteria" resultMap="...">
        SELECT
            *
        FROM
            t_order
        WHERE

        <isPropertyAvailable property="statusCode"> <!-- (1) -->
            status_code = #statusCode#
        </isPropertyAvailable>

        <isNotPropertyAvailable property="statusCode"> <!-- (2) -->
            <![CDATA[
            status_code <> 'completed'
            ]]>
        </isNotPropertyAvailable>

        <!-- ... -->

    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | In the example, when \ ``statusCode``\  property exists, WHERE clause is set such that status_code column fetches records matching with \ ``statusCode``\ .
    * - | (2)
      - | In the example, when \ ``statusCode``\  property does not exist, WHERE clause is set such that status_code column fetches records other than \ ``'completed'``\ .

Following 2 patterns of SQL are generated by dynamic SQL mentioned above.

 .. code-block:: sql

    -- (1) statusCode='checking'
    SELECT * FROM t_order WHERE status_code = 'checking'

    -- (2)
    SELECT * FROM t_order WHERE status_code <> 'completed'


Determining whether property value of parameter object (JavaBean) is set
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
SQL can be built by determining whether value is specified in the parameter object (JavaBean) property passed to SQL.

Determination of elements is as follows:

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
 .. list-table::
    :widths: 10 15 75
    :header-rows: 1

    * - Sr. No.
      - Element 
      - Description
    * - 1.
      - isNull
      - Element for building SQL when property value is \ ``null``\ .
    * - 2.
      - isNotNull
      - Element for building SQL when property value is not \ ``null``\ .
    * - 3.
      - isEmpty
      - Element for building SQL when property value is \ ``null``\  or empty.
        It can be specified for \ ``Collection``\  and \ ``String``\ .
    * - 4.
      - isNotEmpty
      - Element for building SQL when property value is ``null`` and not empty.
        It can be specified for \ ``Collection``\  and \ ``String``\ .

Implementation is as follows:

 .. code-block:: xml

    <select id="findOne" parameterClass="OrderCriteria" resultMap="">
        SELECT
            *
        FROM
            t_order
        WHERE

        <isNull property="orderedDate"> <!-- (1) -->
            <![CDATA[
            CURRENT_DATE - '1 months'::interval <= ordered_date
            ]]>
        </isNull>

        <isNotNull property="orderedDate"> <!-- (2) -->
            ordered_date = #orderedDate#
        </isNotNull>

        <isEmpty property="statusCodes" prepend="AND"> <!-- (3) -->
            <![CDATA[
            status_code <> 'completed'
            ]]>
        </isEmpty>

        <isNotEmpty property="statusCodes" prepend="AND"> <!-- (4) -->
            status_code IN
            <iterate property="statusCodes" open="(" close=")" conjunction=",">
                #statusCodes[]#
            </iterate>
        </isNotEmpty>

    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | In the example, when \ ``orderedDate``\  property (Date type) value is \ ``null``\ , WHERE clause is set such that the ordered_date column can fetch records starting from one month prior to the current date.
    * - | (2)
      - | In the example, when \ ``orderedDate``\  property (Date type) value is not \ ``null``\ , WHERE clause is set such that the ordered_date column can fetch records that match with \ ``orderedDate``\ .
    * - | (3)
      - | In the example, when \ ``statusCodes``\  property (List<String> type) value is empty, WHERE clause is set such that the status_code column can fetch records other than \ ``'completed'``\ .
    * - | (4)
      - | In the example, when \ ``statusCodes``\  property (List<String> type) value is not empty, WHERE clause is set such that the status_code column can fetch records that match with any of the values stored in \ ``statusCodes``\ .
        | iterate element is described later.

Following 4 SQL patterns are generated by the dynamic SQL mentioned above.

 .. code-block:: sql

    -- (1) orderedDate=null, statusCodes=[]
    SELECT * FROM t_order WHERE CURRENT_DATE - '1 months'::interval <= ordered_date
        AND status_code <> 'completed'

    -- (2) orderedDate=null, statusCodes=['accepted','checking']
    SELECT * FROM t_order WHERE CURRENT_DATE - '1 months'::interval <= ordered_date
        AND status_code IN ('accepted','checking')

    -- (3) orderedDate=2013/12/31, statusCodes=null
    SELECT * FROM t_order WHERE ordered_date = '2013/12/31'
        AND status_code <> 'completed'

    -- (4) orderedDate=2013/12/31, statusCodes=['accepted']
    SELECT * FROM t_order WHERE ordered_date = '2013/12/31'
        AND status_code IN ('accepted')

|

Determining the property value of parameter object (JavaBean)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
SQL can be built by determining the value specified in the property of parameter object (JavaBean) passed to SQL.

Decision elements are as follows:

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
 .. list-table::
    :widths: 10 15 75
    :header-rows: 1

    * - Sr. No.
      - Element
      - Description
    * - 1.
      - isEqual
      - Element for building SQL when property value matches with the specified value.
    * - 2.
      - isNotEqual
      - Element for building SQL when property value does not match with the specified value.
    * - 3.
      - isGreaterThan
      - Element for building SQL when property value is greater than the specified value.
    * - 4.
      - isGreaterEqual
      - Element for building SQL when property value is greater than or equal to the specified value.
    * - 5.
      - isLessThan
      - Element for building SQL when property value is less than the specified value.
    * - 6.
      - isLessEqual
      - Element for building SQL when property value is less than or equal to the specified value.

Implementation is as follows:

 .. code-block:: xml

    <select id="findOne" parameterClass="OrderCriteria" resultMap="...">
        SELECT
            *
        FROM
            t_order
        WHERE
            (
            <![CDATA[
            status_code <> 'completed'
            ]]>
            <isEqual property="containCompletedOrder"
                     compareValue="true"
                     prepend="OR"> <!-- (1) -->
                status_code = 'completed'
            </isNull>
            )

    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | In the example, when the \ ``containCompletedOrder``\  property (Boolean type) value is \ ``true``\ , WHERE clause is set such that the status_code column fetches \ ``'completed'``\  records as well.

 .. note::

    Other property values in JavaBean can be compared by using compareProperty attribute.

Following 2 SQL patterns are generated by the dynamic SQL mentioned above.

 .. code-block:: sql

    -- (1) containCompletedOrder=false
    SELECT * FROM t_order WHERE (status_code <>  'completed')

    -- (2) containCompletedOrder=true
    SELECT * FROM t_order WHERE (status_code <>  'completed' OR status_code = 'completed')


Common attributes of decision elements
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Elements that build dynamic SQL have the following common attributes.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
 .. list-table::
    :widths: 10 15 75
    :header-rows: 1

    * - Sr. No.
      - Attribute
      - Description
    * - 1.
      - prepend
      - When the SQL statement is determined as \ ``true``\  by the decision element used for building dynamic SQL, specify the character string set at the beginning of the SQL.
    * - 2.
      - open
      - Specify the character string to be added before the built SQL in the decision element used for building dynamic SQL.
    * - 3.
      - close
      - Specify the character string to be added at the end of the built SQL in the decision element used for building dynamic SQL.

Implementation is as follows:

 .. code-block:: xml

    <select id="findOne" parameterClass="OrderCriteria" resultMap="...">
        SELECT
            *
        FROM
            t_order

        <isNotEmpty property="statusCode"
                    prepend="WHERE"
                    open="("
                    close=")"> <!-- (1) -->
            status_code = #statusCode#
            <isEqual property="containCompletedOrder" compareValue="true" prepend="OR">
                status_code = 'completed'
            </isEqual>
        </isNotEmpty>
        
    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
 .. list-table::
    :widths: 10 15 75
    :header-rows: 1 

    * - Sr. No.
      - Attribute
      - Description
    * - | (1)
      - | -
      - | In the example, when a value is specified in \ ``statusCode``\  property, on setting the WHERE clause in status_code column, if the
        | \ ``containCompletedOrder``\  property (Boolean type) value is \ ``true``\ , WHERE clause is set such that status_code column fetches \ ``'completed'``\  records as well.
    * - | -
      - | prepend
      - | \ ``"WHERE"``\  clause is set in SQL when value is specified in \ ``statusCode``\  property.
    * - | -
      - | open
      - | When the \ ``containCompletedOrder``\  property (Boolean type) value is \ ``true``\ ,
        | opening character \ ``"("``\ , used for grouping the conditions for status_code column is specified, so as to add the OR condition.
    * - | -
      - | close
      - | Closing character  \ ``")"``\  used for grouping the conditions for status_code column is specified.

Following 3 SQL patterns are generated by dynamic SQL mentioned above.

 .. code-block:: sql

    -- (1) statusCode=null, containCompletedOrder=false
    SELECT * FROM t_order

    -- (2) statusCode='accepted', containCompletedOrder=false
    SELECT * FROM t_order WHERE (status_code = 'accepted')

    -- (3) statusCode='checking', containCompletedOrder=true
    SELECT * FROM t_order WHERE (status_code = 'checking' OR status_code = 'completed')


Collection iteration
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
When a collection or array of bind values is passed to SQL, the SQL can be built by iterating the process for element part of the collection and array.

Element is as follows:

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
 .. list-table::
    :widths: 10 15 75
    :header-rows: 1

    * - Sr. No.
      - Element
      - Description
    * - 1.
      - iterate
      - Element that builds SQL by carrying out iteration process for collection and array.

Implementation is as follows:

 .. code-block:: xml

    <select id="findOne" parameterClass="OrderCriteria" resultMap="...">
        SELECT
            *
        FROM
            t_order

        <isNotNull property="statusCodes" prepend="WHERE">
            <iterate property="statusCodes"
                     prepend="status_code IN"
                     open="("
                     conjunction=","
                     close=")" > <!-- (1) -->
                #statusCodes[]#
            </iterate>
        </isNotNull>
        
    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
 .. list-table::
    :widths: 10 15 75
    :header-rows: 1 

    * - Sr. No.
      - Attribute
      - Description
    * - | (1)
      - | -
      - | In the example,  value stored in \ ``statusCodes``\  property (List<String>) is set as a value for IN clause.
    * - | -
      - | prepend
      - | When collection or array elements exist, specify the character string to be set first. In the example, the column name to be added to the condition as well as IN clause are specified.
    * - | -
      - | open
      - | Specify the character string to be set before processing the first element of collection or array. In the example, the starting enclosure character \ ``"("``\  of the value specified in IN clause is specified.
    * - | -
      - | conjunction
      - | If next element exists, specify the character string to be set before processing the next element. In the example, delimiter \ ``","``\  of the value specified in IN clause is specified.
    * - | -
      - | close
      - | Specify the character string to be set after processing the last element of collection or array. In the example, ending enclosure character \ ``")"``\  of the value specified in IN clause is specified. 

 .. note::

    The above example illustrates the implementation wherein the JavaBean properties are of collection type. However, the parameter objects themselves can be set to collection type.
    In that case, it can be accessed in \ ``#[]#``\  format without specifying the property attribute.

    JavaBean can also be stored in collection and JavaBean nested collection can also be accessed.
    For details, refer to "Dynamic Mapped Statements" (P.52) of the Developer Guide (PDF).

Following 3 SQL patterns are generated by the dynamic SQL mentioned above.

 .. code-block:: sql

    -- (1) statusCodes=null
    SELECT * FROM t_order

    -- (2) statusCodes=[]
    SELECT * FROM t_order

    -- (3) statusCodes=['accepted','checking']
    SELECT * FROM t_order WHERE status_code IN ('accepted' , 'checking')


Dynamic SQL blocking
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
By blocking individual dynamic SQL, prepend, open and close attributes can be controlled as an entire block.

The elements are as follows:

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
 .. list-table::
    :widths: 10 15 75
    :header-rows: 1

    * - Sr. No.
      - Element
      - Description
    * - 1.
      - dynamic
      - Element that blocks the elements building the dynamic SQL.

Implementation is as follows:

 .. code-block:: xml

    <select id="findOne" parameterClass="OrderCriteria" resultMap="...">
        SELECT
            *
        FROM
            t_order
        WHERE

        <dynamic prepend="WHERE"
                 open="("
                 close=")"> <!-- (1) -->

            <isNotEmpty property="id" prepend="AND"> <!-- (2) -->
                id = #id#
            </isNotEmpty>

            <isNotEmpty property="statusCode" prepend="AND"> <!-- (3) -->
                status_code = #statusCode#
            </isNotEmpty>

        </dynamic>

    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
 .. list-table::
    :widths: 10 15 75
    :header-rows: 1

    * - Sr. No.
      - Attribute
      - Description
    * - | (1)
      - | -
      - | Dynamic SQLs of (2) and (3) are blocked.
    * - | -
      - | prepend
      - | Specify the character string to be prepended to the SQL built within the block. Value specified here is used as the prepend attribute value of the dynamic SQL matched in the block at first.
        | In the above example, when value is specified in the \ ``id``\  property, the value of prepend attribute in (2) becomes  \ ``"WHERE"``\  rather than \ ``"AND"``\ .
    * - | -
      - | open
      - | Specify the character string to be added before the SQL built in block.
    * - | -
      - | close
      - | Specify the character string to be added after the SQL built in block.

Following 4 SQL patterns are generated by the dynamic SQL mentioned above.

 .. code-block:: sql

    -- (1) id=null, statusCode=null
    SELECT * FROM t_order

    -- (2) id=1, statusCode=null
    SELECT * FROM t_order WHERE (id = 1)

    -- (3) id=null, statusCode='accepted'
    SELECT * FROM t_order WHERE (status_code = 'accepted')

    -- (4) id=1, statusCode='accepted'
    SELECT * FROM t_order WHERE (id = 1 AND status_code = 'accepted')


How to use QueryDAO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Single record search
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Implementation is as follows when a query with search results as 0 to 1 record is issued.

- Xxx.java

 .. code-block:: java

     String todoId = "xxxxx....";
     Todo loadedTodo = queryDAO.executeForObject( // (1)
             "todo.findOne",    // (2)
             todoId,            // (3)
             Todo.class);       // (4)
     if (loadedTodo == null) {  // (5)
         // ...                 // (6)
     }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Call the method (\ ``QueryDAO#executeForObject``\ ) that fetches search result as the object of type specified in (4).
    * - | (2)
      - | Specify SQLID of the SQL with search result 0 to 1 record.
        | Mybatis throws \ ``java.sql.SQLException``\  when the search results are more than one.
    * - | (3)
      - | Specify SQL bind parameter.
        | In the example, bind parameter is \ ``java.lang.String``\ . However, when multiple parameters (search conditions) are to be passed, JavaBean can also be specified.
    * - | (4)
      - Specify the type of object that maps the fetch results of SQL.
    * - | (5)
      - When the search result shows 0 records, the value becomes null. Hence, null check is necessary.
    * - | (6)
      - Implement the process when search result is 0 records.


Multiple records search
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Implementation is as follows when query with search result 0 to N records is issued and all the data matching with the conditions is fetched.

- Xxx.java

 .. code-block:: java

     boolean finished = false;
     List<Todo> unfinishedTodoList = queryDAO.executeForObjectList( // (1)
             "todo.findByFinished",     // (2)
             finished);                 // (3)
     if(unfinishedTodoList.isEmpty()){  // (4)
         // ...                         // (5)
     }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Call the method to fetch the objects list.
    * - | (2)
      - Specify SQLID of the SQL with search result 0 to N records.
    * - | (3)
      - | Specify the SQL bind parameter.
        | In the example, bind parameter is set as boolean. However, when multiple parameters (search conditions) are to be passed, JavaBean can also be specified.
    * - | (4)
      - When search result is 0 records, an empty list is returned. null check is not required as null value is not returned.
    * - | (5)
      - Implement the process when search result is 0 records.


Pagination search (TERASOLUNA DAO standard function system)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| When issuing a query with search result 0 to N records and fetching a part of data (specified part of page) matching with the condition, the implementation is as follows:
| Following implementation is done by using the API provided by TERASOLUNA DAO.
\
 .. warning:: **Points to be noted when an extremely large number of data records match with search conditions**

    In the TERASOLUNA DAO standard function pagination search, the part up to starting position of the records that are fetched using \ ``java.sql.ResultSet#next``\  is skipped.
    As a result, when the number of data records matching with search conditions is huge, it may impact the process performance.
    When there is a possibility of a huge number of data records matching the search conditions, SQL refinement should be adopted instead of pagination search of TERASOLUNA DAO standard function.

- Xxx.java

 .. code-block:: java

     Pageable pageable = new PageRequest(0, 10); // (1)
     boolean finished = false;
     long totalCount = queryDAO.executeForObject(
             "todo.countByFinished", // (2)
             finished,
             Long.class);            // (3)

     List<Todo> unfinishedTodoList = null;
     if(0 < totalCount) {
         unfinishedTodoList = queryDAO.executeForObjectList(
             "todo.findByFinished",   // (4)
             finished,
             pageable.getOffset(),    // (5)
             pageable.getPageSize()); // (6)
     } else {
         unfinishedTodoList = new ArrayList<Todo>();
     }

     Page<Todo> page = new PageImpl<Todo>( // (7)
             unfinishedTodoList, // (8)
             pageable,           // (9)
             totalCount);        // (10)

- xxx-sqlmap.xml

 .. code-block:: xml

     <select id="findByFinished"
             parameterClass="boolean"
             resultMap="resultMap_Todo"> <!-- (11) -->
         SELECT
             *
         FROM
             todo
         WHERE
             finished = #finished#
         ORDER BY
             created_at DESC
     </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Generate paging search object (\ ``org.springframework.data.domain.PageRequest``\ ) provided by Spring Data.
        Pageable object can also be received as Controller argument by specifying in request parameters. For details, refer to \ :doc:`Pagination`\ .
    * - | (2)
      - Execute by specifying the SQLID of the SQL that fetches total number of data records matching with the condition.
    * - | (3)
      - Specify Long.class since this involves fetching records.
    * - | (4)
      - Execute by specifying the SQLID of the SQL with search result 0 to N records.
    * - | (5)
      - | Specify the start position from where records are to be fetched.
        | Starting from 0. When 10 records are to be fetched, on specifying 10, records from 11 to 20th are fetched.
    * - | (6)
      - | Specify the number of records to be fetched.
        | When the start position for fetching records is 0, on specifying 10, records from 1 to 10th are fetched.
    * - | (7)
      - Generate the page object (\ ``org.springframework.data.domain.PageImpl``\ ) provided by Spring Data.
    * - | (8)
      - Carry out pagination search and specify the fetched list.
    * - | (9)
      - Specify the paging search object (Pageable) used in pagination search.
    * - | (10)
      - Specify the total number of data records that match with the condition.
    * - | (11)
      - SQL implementation. It is not necessary to consider the fetch position in case of SQL.


Pagination search (SQL refinement)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| When a query with search result 0 to N records is issued and a part of data (specified part of page) matching with the condition is fetched, the implementation is as follows:
| Following implementation is done by using SQL instead of the API provided by TERASOLUNA DAO.

- PageableBindParams.java (Sample class)
  
 .. code-block:: java

     public class PageableBindParams<P> implements Serializable { // (1)
         private static final long serialVersionUID = 1L;
         private final P bindParams;
         private final Pageable pageable;
         public PageableBindParams(P bindParams, Pageable pageable) {
             this.bindParams = bindParams;
             this.pageable = pageable;
         }
         public P getBindParams() {
             return bindParams;
         }
         public Pageable getPageable() {
             return pageable;
         }
      }

- Xxx.java

 .. code-block:: java

     Pageable pageable = new PageRequest(0, 10);
     boolean finished = false;
     long totalCount = queryDAO.executeForObject(
             "todo.countByFinished",
             finished,
             Long.class); // (2)

     List<Todo> unfinishedTodoList = null;
     if(0 < totalCount) {
         PageableBindParams<Boolean> pageableBindParams =
                 new PageableBindParams<Boolean>( // (3)
                          finished,  // (4)
                         pageable); // (5)
         unfinishedTodoList = queryDAO.executeForObjectList(
                 "todo.findPageByFinished", // (6)
                 pageableBindParams);       // (7)
     } else {
         unfinishedTodoList = new ArrayList<Todo>();
     }

     Page<Todo> page = new PageImpl<Todo>(
             unfinishedTodoList,
             pageable,
             totalCount); // (8)

- xxx-sqlmap.xml

 .. code-block:: xml

     <select id="findPageByFinished"
             parameterClass="xxxxxx.yyyyyy.zzzzzz.domain.dto.PageableBindParams"
             resultMap="resultMap_Todo"> <!-- (9) -->
         SELECT
             *
         FROM
             todo
         WHERE
             finished = #bindParams#
         ORDER BY
             created_at DESC
         OFFSET
             #pageable.offset#    /* (10) */
         LIMIT
             #pageable.pageSize#  /* (11) */
    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - JavaBean storing the parameters (bind parameter) that form search conditions and paging search object provided by Spring Data (\ ``org.springframework.data.domain.Pageable``\ ).
        An aggregate object like this class is necessary since only a single object can be passed to DAO. This class should be provided as required in each project since it is a sample implementation.
    * - | (2)
      - Fetch the total number of records similar to using TERASOLUNA DAO standard function.
    * - | (3)
      - | Generate the bind object to be passed to DAO.
        | In the example, class provided in (1) is used.
    * - | (4)
      - | Specify the search condition to narrow down target data.
        | In the example, "false" is specified as the value of "finished".
    * - | (5)
      - | Specify the search condition for narrowing down data on the corresponding page.
        | In the example, paging search object provided by Spring Data (\ ``org.springframework.data.domain.PageRequest``\ ) is specified.
        | The Pageable object can also be received as Controller argument by specifying it in request parameters. For details, refer to \ :doc:`Pagination`\ .
    * - | (6)
      - Specify SQLID of the SQL executed by the SQL that extracts data on the corresponding page.
    * - | (7)
      - Specify the bind object generated in (3).
    * - | (8)
      - Same as using TERASOLUNA DAO standard function, generate the page object provided by Spring Data (``org.springframework.data.domain.PageImpl``).
    * - | (9)
      - SQL implementation. In the example, PostgreSQL function (OFFSET, LIMIT) are used. As an SQL, notice the fetch position.
    * - | (10)
      - | Specify the start position from where records are to be fetched.
        | Starting from 0. When 10 records are to be fetched, on specifying 10, records from 11 to 20th are fetched. (Use PostgreSQL function).
    * - | (11)
      - | Specify the number of records to be fetched.
        | When the start position for fetching records is 0, on specifying 10, records from 1 to 10th are fetched. (Use PostgreSQL function).


How to use UpdateDAO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Inserting a single record
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
The implementation is as follows when a single data record is to be inserted.

- Xxx.java

 .. code-block:: java

     // (1)
     Todo todo = new Todo();
     todo.setTodoId(todoId);
     todo.setTodoTitle(todoTitle);
     todo.setFinished(false);
     todo.setCreatedAt(now);
     int insertedCount = updateDAO.execute("todo.insert", todo); // (2)
     if(insertedCount != 1){  // (3)
         // ...               // (4)
     }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Generate the data to be inserted (JavaBean).
    * - | (2)
      - Execute DAO by specifying SQLID of Insert SQL and the data to be inserted (JavaBean).
    * - | (3)
      - If required, check the number of data records actually inserted. In the example, it is checked whether a single record is inserted.
    * - | (4)
      - If required, carry out the process when the number of actually inserted records differs from number of expected records.


Inserting multiple records (batch execution)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Implementation in case of inserting multiple data records through batch execution of multiple SQLs is as follows:
| Use \ ``jp.terasoluna.fw.dao.SqlHolder``\  provided by TERASOLUNA DAO.

- Xxx.java

 .. code-block:: java

     // (1)
     Todo todo = new Todo();
     todo.setTodoId(todoId);
     todo.setTodoTitle(todoTitle);
     todo.setFinished(false);
     todo.setCreatedAt(now);

     // (2)
     Todo todo2 = new Todo();
     todo2.setTodoId(todoId2);
     todo2.setTodoTitle(todoTitle2);
     todo2.setFinished(false);
     todo2.setCreatedAt(now);

     List<SqlHolder> sqlHolders = new ArrayList<SqlHolder>(); // (3)
     sqlHolders.add(new SqlHolder("todo.insert", todo));      // (4)
     sqlHolders.add(new SqlHolder("todo.insert", todo2));     // (4)
     int insertedCount = updateDAO.executeBatch(sqlHolders);  // (5)
     if(insertedCount != 2){  // (6)
         // ...               // (7)
     }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Generate the data to be inserted (JavaBean). 1st data record.
    * - | (2)
      - Generate the data to be inserted (JavaBean). 2nd data record.
    * - | (3)
      - Generate a list of \ ``jp.terasoluna.fw.dao.SqlHolder``\  provided by TERASOLUNA DAO, for batch execution.
    * - | (4)
      - Add the data generated in (1), (2) to the SqlHolder list as bind objects. In the example, 2 records are added to the list.
    * - | (5)
      - Execute batch by specifying the SqlHolder list generated in (1)~(4).
    * - | (6)
      - | If required, check the number of actually inserted data records.
        | In the example, it is checked whether 2 records are inserted.
    * - | (7)
      - If required, carry out the process when the number of actually inserted records differs from the number of expected records.
\
 .. warning:: **About number of records inserted in batch execution**

    There are cases where, on batch execution, exact number of rows cannot be fetched using JDBC driver.
    When a driver that is unable to fetch the exact number of rows is used, batch execution should not be carried out for the cases wherein number of inserted records need to be checked.
    (This holds true for updated records and deleted records as well).


Updating a single record
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Implementation is as follows while updating a single data record.
| It is the same as in case of inserting a single record. SQL to be used is Update SQL.

- Xxx.java

 .. code-block:: java

     Todo loadedTodo = queryDAO.executeForObject("todo.findOne",
             todoId,
             Todo.class);     // (1)
     todo2.setFinished(true); // (2)
     int updatedCount = updateDAO.execute("todo.update", todo); // (3)
     if(updatedCount != 1){   // (4)
         // ...               // (5)
     }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Search the data to be updated (JavaBean).
    * - | (2)
      - Update data. In the example, "finished" is updated from false to true.
    * - | (3)
      - Execute DAO by specifying SQLID of the Update SQL and the data to be updated (JavaBean).
    * - | (4)
      - | If required, check the number of actually updated data records.
        | In the example, it is checked whether a single record is updated.
    * - | (5)
      - If required, carry out the process when the number of actually updated records differs from the number of expected records.


Updating multiple records (batch execution)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Implementation for updating multiple data records through batch execution is same as in case of inserting multiple records (batch execution).
| Updating multiple records by batch execution is effective when the update value differs for each record.


Updating multiple records (Specifying WHERE clause)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Implementation is as follows when the data that matches with the condition specified by SQL is collectively updated.
| Updating multiple records by specifying WHERE clause is effective when all records are collectively updated to the same value.


- Xxx.java

 .. code-block:: java

     int deadlineDays = 7;
     int updatedCount = updateDAO.execute("todo.update", deadlineDays); // (1)

- xxx-sqlmap.xml

 .. code-block:: xml

     <update id="updateFinishedDeadlineByUnfinished" parameterClass="int"> <!-- (2) -->
         <![CDATA[
         UPDATE
             todo
         SET
             todo_title = '[Finished Deadline]' || todo_title
             ,version = (version + 1)
         WHERE
             finished = false
         AND
             created_at < current_date - #deadlineDays#
         ]]>
    </update>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Execute DAO by specifying SQLID of the SQL used for batch update and condition that extracts the data to be updated.
    * - | (2)
      - Batch update SQL implementation. In the example, string \ "[Finished Deadline] "\  is added to the beginning of incomplete TODO titles for which 7 days have passed since their creation.


Deleting a single record
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Implementation is as follows while deleting a single data record.

- Xxx.java

 .. code-block:: java

     String todoId = "xxxxx....";
     int deletedCount = updateDAO.execute("todo.delete", todoId); // (1)
     if(deletedCount != 1){
         // ...               // (2)
     }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Execute DAO by specifying PK and SQLID of the Delete SQL.
        | In the example, \ ``java.lang.String``\  is specified. However, JavaBean can also be specified in case of composite keys.
    * - | (2)
      - If required, carry out the process when the number of actually deleted records differs than the expected records.


Deleting multiple records (batch execution)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Implementation for deleting multiple data records through batch execution using multiple SQL is same as that in case of updating multiple records (batch execution).
| When it is necessary to share the process for deleting a single record, use multiple records deletion by batch execution. However, when a large amount of data is to be deleted, the batch deletion by specifying the WHERE clause should be considered.


Deleting multiple records (Specifying WHERE clause)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Implementation for collectively deleting the data matching with SQL specified conditions is same as that in case of updating multiple records (batch execution).
| However, when a large number of records are to be deleted, batch deletion by specifying WHERE clause is effective.


Example showing use of StoredProcedureDAO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The implementation is as follows when a procedure or a function is called.

- Xxx.java

 .. code-block:: java

    SalesItem item = new SalesItem(); // (1)
    item.setId(Integer.valueOf(1));  // (2)
    storedProcedureDAO.executeForObject("todo.findSalesItem", item);  // (3)
    // (4)
    logger.debug("Quantity is {}.", item.getQuantity());
    logger.debug("Total is {}.", item.getTotal());


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Generate the bind object that stores IN and OUT parameters of a procedure or function.
    * - | (2)
      - Set the ID for IN parameter. In the example, \ ``1``\  is set as the ID.
    * - | (3)
      - Call \ ``StoredProcedureDAO``\  method with bind object and SQLID of the SQL used for calling stored procedure as the argument.
    * - | (4)
      - | When the method of \ ``StoredProcedureDAO``\  is called successfully,
        | OUT parameter of the procedure or function is set in bind object.
        | In the example, the OUT parameter value set in bind object is output to log.


Example showing use of QueryRowHandleDAO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- Xxx.java

 .. code-block:: java


     boolean finished = false;
     queryRowHandleDAO.executeWithRowHandler(
             "todo.findByFinished",            // (1)
             finished,                         // (2)
             new DataRowHandler() {            // (3)
                 public void handleRow(Object valueObject) { // (4)
                     Todo todo = (Todo) valueObject;
                     logger.info(todo.toString());  // (5)
                 }
             });

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No
      - Description
    * - | (1)
      - Specify SQLID of the SQL with search result 0 to N records.
    * - | (2)
      - | Specify SQL bind parameter.
        | In the example, it is specified as boolean. However, when multiple parameters (search conditions) are passed, JavaBean can also be specified.
    * - | (3)
      - | Specify the implementation object of \ ``jp.terasoluna.fw.dao.event.DataRowHandler``\ .
        | In the example, an unnamed class is used. However, creating an implementation class should be considered in actual project.
    * - | (4)
      - | handleRow method is called for each record of search result.
        | Object that is passed to an argument becomes object of the class specified in class attribute of resultMap element or the class specified in resultClass attribute of select element.
    * - | (5)
      - In the example, only log output is carried out. However, while using it in actual project, processes for file output, aggregate of each record value and value processing etc. are carried out.


.. _data-access-mybatis2_howtouse_like_escape:

Escaping during LIKE search
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| While performing LIKE search, the values to be used as search conditions need to be escaped.
| Escaping for LIKE search can be done using methods of \ ``org.terasoluna.gfw.common.query.QueryEscapeUtils``\  class provided by common library.
| For specifications of escaping provided by common library, refer to \ :ref:`data-access-common_appendix_like_escape`\  of \ :doc:`DataAccessCommon`\ .

| How to use escaping provided by common library, is explained below.


How to use escaping when matching method is specified in Query
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
When matching method (forward match, backward match and partial match) are specified as JPQL, use a method that performs only escaping.

- :file:`xxx-sqlmap.xml`

 .. code-block:: xml

    // (1) (2)
    <select id="findAllByWord" parameterClass="String" resultMap="resultMap_Article">
      SELECT
          *
      FROM
          article
      WHERE
          title LIKE '%' || #word# || '%' ESCAPE '~'
      OR
          overview LIKE '%' || #word# || '%' ESCAPE '~'
    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify the wild card (\ ``"%"``\  or \ ``"_"``\ ) for LIKE search, in SQL.
        | In the above example, a matching method is a "partial match" by specifying wild card (\ ``"%"``\ ) before and after \ ``word``\  argument.
    * - | (2)
      - | Specify \ ``"ESCAPE '~'"``\  at the end of LIKE clause since \ ``"~"``\  is used as escape character in the escaping provided by common library.


- Service or Repository

 .. code-block:: java

    @Inject
    QueryDAO queryDAO;

    @Transactional(readOnly = true)
    public Page<Article> searchArticle(ArticleSearchCriteria criteria,
            Pageable pageable) {

        String escapedWord = QueryEscapeUtils.toLikeCondition(criteria.getWord()); // (3)

        long total = queryDAO.executeForObject("article.countByWord",
                escapedWord, Long.class);
        List<Article> contents = null;
        if (0 < total) {
            contents = queryDAO.executeForObjectList("article.findAllByWord",
                    escapedWord, pageable.getOffset(), pageable.getPageSize()); // (4)
        } else {
            contents = Collections.emptyList();
        }
        return new PageImpl<Article>(contents, pageable, total);
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (3)
      - | When the matching method of LIKE search is specified in Query, call \ ``QueryEscapeUtils#toLikeCondition(String)``\  method and perform escaping for LIKE search only.
    * - | (4)
      - | Pass the value escaped for LIKE search, to the bind parameter of \ ``QueryDAO``\ .


How to use escaping when the matching method is specified in Logic
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
When matching method (forward match, backward match and partial match) is determined at Logic side, use the method wherein wild card is assigned to the escaped value.

- :file:`xxx-sqlmap.xml`

 .. code-block:: xml

    // (1)
    <select id="findAllByWord" parameterClass="String" resultMap="resultMap_Article">
      SELECT
          *
      FROM
          article
      WHERE
          title LIKE #word# ESCAPE '~'
      OR
          overview LIKE #word# ESCAPE '~'
    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Do not specify the wild card for LIKE search in SQL.


- Service or Repository

 .. code-block:: java

    @Inject
    QueryDAO queryDAO;

    @Transactional(readOnly = true)
    public Page<Article> searchArticle(ArticleSearchCriteria criteria,
            Pageable pageable) {

        String escapedWord  = QueryEscapeUtils
                .toContainingCondition(criteria.getWord()); // (2)

        long total = queryDAO.executeForObject("article.countByWord",
                escapedWord, Long.class);
        List<Article> contents = null;
        if (0 < total) {
            contents = queryDAO.executeForObjectList("article.findAllByWord",
                    escapedWord, pageable.getOffset(), pageable.getPageSize()); // (3)
        } else {
            contents = Collections.emptyList();
        }
        return new PageImpl<Article>(contents, pageable, total);
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (2)
      - | Call any one of the following methods while specifying the matching method in logic and assign the Escape and wild card values for LIKE search.
        |   ``QueryEscapeUtils#toStartingWithCondition(String)``
        |   ``QueryEscapeUtils#toEndingWithCondition(String)``
        |   ``QueryEscapeUtils#toContainingCondition(String)``
    * - | (3)
      - | Pass the escape + wild card value assigned for LIKE search to the bind parameter of \ ``QueryDAO``\ .


About the measures to be taken against SQL Injection
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| When building SQL, it is necessary to ensure prevention of SQL Injection.
| Mybatis2 provides 2 methods for embedding values in SQL.

* | Method to embed values using bind variable
  | By this method, values can be safely embedded post SQL building, using \ ``java.sql.PreparedStatement``\ .
  | **When embedding an input value from a user in SQL, bind variable should be used.** 

* | Method to embed values using substitution variable.
  | This method does not assure safe embedding of values since they get substituted as strings during SQL building. 

 .. warning::

    It should be noted that, when an input value from the user is embedded using substitution variable, a risk of SQL Injection attack is higher.
    When it is necessary to use substitution variable to embed input value from the user, input check that ensures prevention of SQL Injection must always be implemented.

    Basically, **it is strongly recommended not to use the values entered by a user as is.**


Embedding values using bind variable
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Use ParameterMap or Inline Parameters in case of bind variable.
| An example of usage is shown below.

Following example shows the use of ParameterMap.

 .. code-block:: xml

    <!-- (1) -->
    <parameterMap id="uploadBinaryParameterMap" class="BinaryFile">
        <parameter property="fileId" />
        <parameter property="fileName" />
        <parameter property="content" jdbcType="BLOB" typeHandler="BlobInputStreamTypeHandler" />
    </parameterMap>

    <insert id="uploadBinary" parameterMap="uploadBinaryParameterMap">
        INSERT INTO upload_binary
        (
            file_id
            ,file_name
            ,content
        )
        VALUES
        (
            ?   /* (2) */
            ,?
            ,?
        )
    </insert>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Define the property that embeds the value as bind variable. Defined order should correspond with the position of \ ``?``\  specified in (2).
    * - | (2)
      - | Specify the bind variable in SQL. Values are bound to  \ ``?``\  part as per the order defined in (1).


Following example shows the use of Inline Parameters.

 .. code-block:: xml

    <insert id="insert"
            parameterClass="xxxxxx.yyyyyy.zzzzzz.domain.model.Todo"> <!-- (1) -->
        INSERT INTO todo
            (
                todo_id
                ,todo_title
                ,finished
                ,created_at
                ,version
            )
            values(
                #todoId#       /* (3) */
                ,#todoTitle#
                ,#finished#
                ,#createdAt#
                ,1
            )
    </insert>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (3)
      - | Specify name of the property that stores bind value as bind variable by enclosing it in \ ``#``\ .


How to embed a value using substitution variable
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Following example shows the use of bind variable.

 .. code-block:: xml

    <select id="findByFinished"
            parameterClass="..."
            resultMap="resultMap_Todo">
        SELECT
            *
        FROM
            todo
        WHERE
            finished = #finished#
        ORDER BY
            created_at $direction$  /* (4) */
    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (4)
      - | Enclose name of the property that stores the value to be substituted, with  \ ``$``\  and specify it as substitution variable.
        | In the above example, \ ``$direction$``\  is replaced either by \ ``"DESC"``\  or \ ``"ASC"``\ .

 .. warning::

    It is recommended to embed values using substitution variable on ensuring that the value is safe for the application and limiting its usage to table names, column names and sort conditions etc.
    
    As shown in the example below, it is desirable to store the code value and the safe value to be actually used, in Map as a pair

      .. code-block:: java
      
        Map<String, String> safeValueMap = new HashMap<String, String>();
        safeValueMap.put("1", "ASC");
        safeValueMap.put("2", "DESC");
      
    and to convert the value during SQL execution so that actual input is changed to code value.

      .. code-block:: java
      
        String direction = safeValueMap.get(input.getDirection());

    \ :doc:`Codelist`\  can also be used.

|

Appendix
--------------------------------------------------------------------------------

Implementation wherein related objects are fetched collectively by a single SQL 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| How to fetch related objects collectively by a single SQL while accessing database, by providing a JavaBean such as Entity for each table, is explained below.
| This method is also used to prevent N+1.

.. warning::

  Following points should be noted while using this implementation.

  * In this example, all related objects are fetched collectively in a single SQL, in order to explain the method.
    However, in actual project, it must be ensured to fetch only the related objects that are required in the process.
    This is because the simultaneous fetching of unused related objects can result in performance degradation in some cases.
  * Related objects with 1:N ratio that are rarely used, are not fetched collectively.
    Fetching such related objects individually, when required, has proved to be better.
    They may be fetched collectively if the performance requirements are satisfied.
  * When there are many related objects with 1:N ratio, fetching them collectively has degraded the performance in some cases as unnecessary data not used in mapping is fetched.
    They may be fetched collectively if the performance requirements are satisfied. However, it is better to look out for other methods.

.. tip::

  For details on how to avoid N+1, refer to "Result Maps/Avoiding N+1 Selects (1:1)" (P.37-38) and "Result Maps/Avoiding N+1 Selects (1:M and M:N)" (P.39-40) of Mybatis Developer Guide (PDF).


| Implementation using order table is explained below.
| Table used is as follows:

 .. figure:: images/dataaccess_er.png
    :alt: ER diagram
    :width: 90%
    :align: center

    **Picture - ER diagram**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.15\linewidth}|p{0.55\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 15 55

    * - Sr. No.
      - Category
      - Table name
      - Description
    * - | (1)
      - Transaction system
      - t_order
      - Table that stores the order. 1 record is stored for 1 order.
    * - | (2)
      -
      - t_order_item
      - Table that stores products purchased in a single order. When multiple products are purchased in a single order, records of multiple products are stored.
    * - | (3)
      -
      - t_order_coupon
      - Table that stores coupons used in one order. When multiple coupons are used in one order, multiple coupon records are stored. No record is stored if a coupon is not used.
    * - | (4)
      - Master system
      - m_item
      - Master table for defining the product.
    * - | (5)
      -
      - m_category
      - Master table for defining the category.
    * - | (6)
      -
      - m_item_category
      - Master table for defining category to which the product belongs. It stores the mapping of product and category. It serves as a model wherein a single product can belong to multiple categories.
    * - | (7)
      -
      - m_coupon
      - Master table for defining the coupon.
    * - | (8)
      - Code system
      - c_order_status
      - Code table for defining the order status.


Transaction table layout and records stored are as follows:

 **t_order**

 .. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 20 20

    * - id(PK)
      - status_code
    * - 1
      - accepted
    * - 2
      - checking

|

 **t_order_item**

 .. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 20 20 20

    * - order_id(PK)
      - item_code(PK)
      - quantity
    * - 1
      - ITM0000001
      - 10
    * - 1
      - ITM0000002
      - 20
    * - 2
      - ITM0000001
      - 30
    * - 2
      - ITM0000002
      - 40


 **t_order_coupon**

 .. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 20 20

    * - order_id(PK)
      - coupon_code(PK)
    * - 1
      - CPN0000001
    * - 1
      - CPN0000002

|

Master table layout and records stored are as follows:

 **m_item**

 .. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 20 20 20

    * - code(PK)
      - name
      - price
    * - ITM0000001
      - Orange juice
      - 100
    * - ITM0000002
      - NotePC
      - 100000

|

 **m_category**

 .. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 20 20

    * - code(PK)
      - name
    * - CTG0000001
      - Drink
    * - CTG0000002
      - PC
    * - CTG0000003
      - Hot selling

|

 **m_item_category**

 .. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 20 20

    * - item_code(PK)
      - category_code(PK)
    * - ITM0000001
      - CTG0000001
    * - ITM0000002
      - CTG0000002
    * - ITM0000002
      - CTG0000003

|

 **m_coupon**

 .. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 20 20 20

    * - code(PK)
      - name
      - price
    * - CPN0000001
      - Join coupon
      - 3000
    * - CPN0000002
      - PC coupon
      - 30000

|

Code table layout and records stored are as follows:

 **c_order_status**

 .. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 20 20

    * - code(PK)
      - name
    * - accepted
      - Order accepted
    * - checking
      - Stock checking
    * - shipped
      - Item Shipped

|

In the following implementation, data stored in above tables is fetched by mapping with the JavaBean mentioned below.

 .. figure:: images/dataaccess_entity.png
    :alt: Class(JavaBean) diagram
    :width: 90%
    :align: center

    **Picture - Class (JavaBean) diagram**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.65\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 15 65

    * - Sr. No.
      - Class name
      - Description
    * - | (1)
      - Order
      - JavaBean that represents one record of t_order table. \ ``OrderStatus``\ , \ ``OrderItem``\  and  \ ``OrderCoupon``\  are stored as multiple related objects.
    * - | (2)
      - OrderItem
      - JavaBean that represents one record of t_order_item table. \ ``Item``\  is stored as related object.
    * - | (3)
      - OrderCoupon
      - JavaBean that represents one record of t_order_coupon table. \ ``Coupon``\  is stored as related object.
    * - | (4)
      - Item
      - JavaBean that represents one record of m_item table. Multiple  \ ``Category``\  to which the records belong are stored as related objects. \ ``Item``\  and \ ``Category``\  are associated by m_item_category table.
    * - | (5)
      - Category
      - JavaBean that represents one record of m_category table.
    * - | (6)
      - Coupon
      - JavaBean that represents one record of m_coupon table.
    * - | (7)
      - OrderStatus
      - JavaBean that represents one record of c_order_status table.


JavaBean property definition is as follows:

- Order.java

 .. code-block:: java

    public class Order implements Serializable {
        private int id;
        private List<OrderItem> orderItems;
        private List<OrderCoupon> orderCoupons;
        private OrderStatus status;
        // ...
    }

- OrderItem.java

 .. code-block:: java

    public class OrderItem implements Serializable {
        private int orderId;
        private String itemCode; // <!-- (1) -->
        private Item item;
        private int quantity;
        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Value to be stored overlaps with \ ``code``\  property of the immediately subsequent \ ``item``\  variable. This is defined since it is required while grouping the records using groupBy attribute of resultMap element, described later.

- OrderCoupon.java

 .. code-block:: java

    public class OrderCoupon implements Serializable {
        private int orderId;
        private String couponCode; // (1)
        private Coupon coupon;
        // ...
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Value to be stored overlaps with \ ``code``\  property of the immediately subsequent \ ``Coupon``\  variable. This is defined since it is required while grouping the records using groupBy attribute of resultMap element, described later.

- Item.java

 .. code-block:: java

    public class Item implements Serializable {
        private String code;
        private String name;
        private int price;
        private List<Category> categories;
        // ...
    }

- Category.java

 .. code-block:: java

    public class Category implements Serializable {
        private String code;
        private String name;
        // ...
    }

- Coupon.java

 .. code-block:: java

    public class Coupon implements Serializable {
        private String code;
        private String name;
        private int price;
        // ...
    }

- OrderStatus.java

 .. code-block:: java

    public class OrderStatus implements Serializable {
        private String code;
        private String name;
        // ...
    }


| Implement SQL mapping.
| When related objects are collectively fetched in a single SQL, JOIN the tables to be fetched and acquire all the records necessary for mapping.
| Perform mapping definition for the fetched records in resultMap element and map with JavaBean.

| Following implementations show the SQL that fetches a single Order (findOne) and one that fetches all Orders (findAll).
| Further, in following implementation,  \ ``"order"``\  is specified in name space and the class name without package name, defined using typeAlias, is assumed for the JavaBean to be mapped.

- sqlMapConfig.xml

 .. code-block:: xml

    <typeAlias alias="Order" type="xxxxxx.yyyyyy.zzzzzz.domain.model.Order"/>
    <typeAlias alias="OrderStatus" type="xxxxxx.yyyyyy.zzzzzz.domain.model.OrderStatus"/>
    <typeAlias alias="OrderItem" type="xxxxxx.yyyyyy.zzzzzz.domain.model.OrderItem"/>
    <typeAlias alias="OrderCoupon" type="xxxxxx.yyyyyy.zzzzzz.domain.model.OrderCoupon"/>
    <typeAlias alias="Item" type="xxxxxx.yyyyyy.zzzzzz.domain.model.Item"/>
    <typeAlias alias="Category" type="xxxxxx.yyyyyy.zzzzzz.domain.model.Category"/>
    <typeAlias alias="Coupon" type="xxxxxx.yyyyyy.zzzzzz.domain.model.Coupon"/>

- order-sqlmap.xml

 .. code-block:: xml

    <sqlMap namespace="order">

        <!-- ... -->

    </sqlMap>

|

| At first, describe the SQL
| Actual definition should be subsequent to the resultMap element.

Implementing SQL

- SQL definitions (sql elements) of the common parts of findOne/findAll

 .. code-block:: xml

    <sql id="fragment_selectFormJoin">         <!-- (1) -->
        SELECT                                   /* (2) */
            o.id
            ,os.code AS status_code
            ,os.name AS status_name
            ,ol.quantity
            ,i.code AS item_code
            ,i.name AS item_name
            ,i.price AS item_price
            ,ct.code AS category_code
            ,ct.name AS category_name
            ,cp.code AS coupon_code
            ,cp.name AS coupon_name
            ,cp.price AS coupon_price
        FROM
            t_order o
        INNER JOIN                                /* (3) */
            c_order_status os
                ON os.code = o.status_code
        INNER JOIN
            t_orderline ol
                ON ol.order_id = o.id
        INNER JOIN
            m_item i
                ON i.code = ol.item_code
        INNER JOIN
            m_item_category ic
                ON ic.item_code = i.code
        INNER JOIN
            m_category ct
                ON ct.code = ic.category_code
        LEFT JOIN                                  /* (4) */
            t_order_coupon oc
                ON oc.order_id = o.id
        LEFT JOIN
            m_coupon cp
                ON cp.code = oc.coupon_code
    </sql>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - sql elements used to share SELECT clause, FROM clause and JOIN clause in findOne and findAll. They have been standardized as there are many common parts in the SQLs for findOne and findAll.
    * - | (2)
      - Fetch all the data required to generate related objects. Ensure that the column name is not repeated. In the above example, since there are duplicate \ ``code``\ , \ ``name``\  and \ ``price``\ , an alias is specified for them using AS clause.
    * - | (3)
      - Join the tables storing the data required to generate related objects.
    * - | (4)
      - Perform Outer JOIN for the tables in which data may not be stored. When coupon is not used, records are not stored in t_group_coupon; hence Outer JOIN should be used. This is also applicable for t_coupon that joins with t_group_coupon.


- SQL definition for findOne

 .. code-block:: xml

    <select id="findOne" parameterClass="java.lang.Integer" resultMap="orderResultMap"> <!-- (1) -->
        <include refid="fragment_selectFormJoin"/> <!-- (2) -->
        WHERE
            o.id = #id#         /* (3) */
        ORDER BY                /* (4) */
            item_code ASC       /* (5) */
            ,category_code ASC  /* (6) */
            ,coupon_code ASC    /* (7) */
    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - SQL for fetching related object and \ ``Order``\  object of the specified order ID.
    * - | (2)
      - Includes the SQL that implements SELECT clause, FROM clause and JOIN clause which are shared with findAll.
    * - | (3)
      - Specify the order ID  passed by bind value, in WHERE clause.
    * - | (4)
      - When related objects with 1:N ratio exist, specify the ORDER BY clause to control the order within the list. Need not be specified if there is no significance to the order.
    * - | (5)
      - To be specified in order to set the \ ``Order#orderItems``\  list in ascending order of code column of t_item table.
    * - | (6)
      - To be specified in order to set the \ ``Item#categories``\  list in ascending order of code column of t_category table.
    * - | (7)
      - To be specified in order to set the \ ``Order#orderCoupons``\  list in ascending order of t_coupon codes.


- SQL definition of findAll

 .. code-block:: xml

    <select id="findAll" resultMap="orderResultMap"> <!-- (1) -->
        <include refid="fragment_selectFormJoin"/> <!-- (2) -->
        ORDER BY
            o.id DESC     /* (3) */
            ,i.code ASC
            ,ct.code ASC
            ,cp.code ASC
    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - SQL to fetch all Orders and related objects.
    * - | (2)
      - sql element that shares the SELECT clause, FROM clause and JOIN clause in findOne and findAll.
    * - | (3)
      - To be specified so as to set the order of fetched list in descending order of t_order ids.


| Following records are fetched on implementing the above SQL (findAll).
| 2 records are fetched as order records. However a total of 9 records are fetched as it joins with related table that stores multiple records.
| 1st~3rd rows are records that generate \ ``Order``\  objects with order ID  \ ``2``\  whereas rows 4th~9th are records that generate \ ``Order``\  objects with order ID  \ ``1``\ .

 .. figure:: images/dataaccess_sql_result.png
    :alt: Result Set of findAll
    :width: 100%
    :align: center

    **Picture - Result Set of findAll**

|

How to map the above records with \ ``Order``\  object and related objects is explained below.

| Implementing resultMap element
| Each element is explained later.


 .. code-block:: xml

    <resultMap id="orderResultMap" class="Order" groupBy="id">
        <result property="id" column="id" />
        <result property="status" resultMap="order.orderStatusResultMap" />
        <result property="orderItems" resultMap="order.orderItemResultMap" />
        <result property="orderCoupons" resultMap="order.orderCouponResultMap" />
    </resultMap>

    <resultMap id="orderStatusResultMap" class="OrderStatus" groupBy="code">
        <result property="code" column="status_code" />
        <result property="name" column="status_name" />
    </resultMap>

    <resultMap id="orderItemResultMap" class="OrderItem" groupBy="itemCode">
        <result property="itemCode" column="item_code" />
        <result property="item" resultMap="order.itemResultMap" />
        <result property="quantity" column="quantity" />
    </resultMap>

    <resultMap id="itemResultMap" class="Item" groupBy="code">
        <result property="code" column="item_code" />
        <result property="name" column="item_name" />
        <result property="price" column="item_price" />
        <result property="categories" resultMap="order.categoryResultMap" />
    </resultMap>

    <resultMap id="categoryResultMap" class="Category" groupBy="code">
        <result property="code" column="category_code" />
        <result property="name" column="category_name" />
    </resultMap>

    <resultMap id="orderCouponResultMap" class="OrderCoupon" groupBy="couponCode">
        <result property="couponCode" column="coupon_code" />
        <result property="coupon" resultMap="order.couponResultMap" />
    </resultMap>

    <resultMap id="couponResultMap" class="Coupon" groupBy="code">
        <result property="code" column="coupon_code" />
        <result property="name" column="coupon_name" />
        <result property="price" column="coupon_price" />
    </resultMap>

|

The role of each resultMap element and its dependency is as shown below.

 .. figure:: images/dataaccess_resultmap.png
    :alt: Implementation of ResultMap
    :width: 100%
    :align: center

    **Picture - Implementation of ResultMap**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Definition to map the fetched records to \ ``Order``\  objects.
        | Mapping of related objects (\ ``OrderStatus``\ , \ ``OrderItem``\  and \ ``OrderCoupon``\ ) is delegated to another resultMap.
    * - | (2)
      - | Definition to map the fetched records to \ ``OrderStatus``\  objects.
    * - | (3)
      - | Definition to map the fetched records to \ ``OrderItem``\  objects.
        | Mapping of related object (\ ``Item``\ ) is delegated to another resultMap.
    * - | (4)
      - | Definition to map the fetched records to  \ ``Item``\  objects.
        | Mapping of related object (\ ``Category``\ ) is delegated to another resultMap.
    * - | (5)
      - | Definition to map the fetched records to \ ``Category``\  objects.
    * - | (6)
      - | Definition to map the fetched records to \ ``OrderCoupon``\  objects.
        | Mapping of related object (\ ``Coupon``\ ) is delegated to another resultMap.
    * - | (7)
      - | Definition to map the fetched records to \ ``Coupon``\  objects.


Mapping the records to \ ``Order``\  object.

 .. code-block:: xml

    <resultMap id="orderResultMap" class="Order" groupBy="id"> <!-- (1) -->
        <result property="id" column="id" /> <!-- (2) -->
        <result property="status" resultMap="order.orderStatusResultMap" /> <!-- (3) -->
        <result property="orderItems" resultMap="order.orderItemResultMap" /> <!-- (4) -->
        <result property="orderCoupons" resultMap="order.orderCouponResultMap" /> <!-- (5) -->
    </resultMap>

 .. figure:: images/dataaccess_resultmap_order.png
    :alt: ResultMap for Order
    :width: 100%
    :align: center

    **Picture - ResultMap for Order**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | It is necessary to group the fetched records for each order. For this, specify the \ ``id``\  property storing the value which uniquely identifies the order, in groupBy attribute.
        | In this example, as the records are grouped by \ ``id``\  property, 2 \ ``Order``\  objects namely, \ ``id=1``\  and \ ``id=2``\  are generated.
    * - | (2)
      - | Set the \ ``id``\  column value of fetched records in \ ``Order#id``\ .
    * - | (3)
      - | Delegate generation of \ ``OrderStatus``\  objects to the \ ``id="order.orderStatusResultMap"``\  resultMap and set the generated objects in \ ``Order#status``\ .
    * - | (4)
      - | Delegate generation of \ ``OrderItem``\  objects to the \ ``id="order.orderItemResultMap"``\  resultMap and add the generated objects in \ ``Order#orderItems``\  list.
    * - | (5)
      - Delegate generation of \ ``OrderCoupon``\  objects to the \ ``id="order.orderCouponResultMap"``\  resultMap and add the generated objects in \ ``Order#orderCoupons``\  list.


Following is described mainly focusing on \ ``id=1``\   \ ``Order``\  object mapping.


Mapping the records to \ ``OrderStatus``\  objects.

 .. code-block:: xml

    <resultMap id="orderStatusResultMap" class="OrderStatus"> <!-- (1) -->
        <result property="code" column="status_code" /> <!-- (2) -->
        <result property="name" column="status_name" /> <!-- (3) -->
    </resultMap>

 .. figure:: images/dataaccess_resultmap_orderstatus.png
    :alt: ResultMap for OrderStatus
    :width: 100%
    :align: center

    **Picture - ResultMap for OrderStatus**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | It is not necessary to specify groupBy attribute since there is a 1:1 relationship between \ ``Order``\  and \ ``OrderStatus``\ .
        | In this example, \ ``OrderStatus``\  object of \ ``code=accepted``\  is generated.
    * - | (2)
      - | Set the \ ``status_code``\  column values of fetched records to \ ``OrderStatus#code``\ .
    * - | (3)
      - | Set the \ ``status_name``\  column values of fetched records to \ ``OrderStatus#name``\ .


Mapping to \ ``OrderItem``\  objects.

 .. code-block:: xml

    <resultMap id="orderItemResultMap" class="OrderItem" groupBy="itemCode"> <!-- (1) -->
        <result property="itemCode" column="item_code" /> <!-- (2) -->
        <result property="item" resultMap="order.itemResultMap" /> <!-- (3) -->
        <result property="quantity" column="quantity" /> <!-- (4) -->
    </resultMap>

 .. figure:: images/dataaccess_resultmap_orderitem.png
    :alt: ResultMap for OrderItem
    :width: 100%
    :align: center

    **Picture - ResultMap for OrderItem**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | It is necessary to specify groupBy attribute since there is a 1:N relationship between \ ``Order``\  and  \ ``OrderItem``\ .
        | Order products need to be grouped with the primary keys of t_order_item (order_id, item_code). Since order_id column is specified in the parent resultMap, specify only \ ``itemCode``\  property that stores the values of item_code column.
        | In this example, as the order products are grouped in \ ``itemCode``\  property, 2 \ ``OrderItem``\  objects namely \ ``itemCode=ITM0000001``\  and \ ``itemCode=ITM0000002``\  are generated.
    * - | (2)        
      - | \ ``item_code``\  column values of fetched records are set in \ ``OrderItem#itemCode``\ .
        | \ ``itemCode``\  property overlaps with \ ``Item#code``\  generated in (3). However, it is necessary for grouping \ ``OrderItem``\ .
    * - | (3)
      - | Generation of \ ``Item``\  object is delegated to \ ``id="order.itemResultMap"``\  resultMap and the generated object is set in \ ``OrderItem#item``\ .
    * - | (4)
      - | \ ``quantity``\  column values of fetched records are set in \ ``OrderItem#quantity``\ .


Mapping the records to \ ``Item``\  objects.

 .. code-block:: xml

    <resultMap id="itemResultMap" class="Item" groupBy="code"> <!-- (1) -->
        <result property="code" column="item_code" /> <!-- (2) -->
        <result property="name" column="item_name" /> <!-- (3) -->
        <result property="price" column="item_price" /> <!-- (4) -->
        <result property="categories" resultMap="order.categoryResultMap" /> <!-- (5) -->
    </resultMap>

 .. figure:: images/dataaccess_resultmap_item.png
    :alt: ResultMap for Item
    :width: 100%
    :align: center

    **Picture - ResultMap for Item**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | There is a 1:1 relationship between \ ``OrderItem``\  and \ ``Item``\ ; however since \ ``Item``\  and \ ``Category``\  have 1:N relationship, groupBy attribute needs to be specified.
        | It is necessary to group the categories for each product. Therefore, specify \ ``code``\  property storing the value which uniquely identifies the order, in groupBy attribute.
        | In this example, \ ``Item``\  object of \ ``code=ITM0000001``\ is generated for \ ``OrderItem#itemCode=ITM0000001``\  and  \ ``Item``\  object of \ ``code=ITM0000002``\  is generated for \ ``OrderItem#itemCode=ITM0000002``\ . (Total 2 objects are generated.)
    * - | (2)
      - | \ ``item_code``\  column values of fetched records are set in \ ``Item#code``\ .
    * - | (3)
      - | \ ``item_name``\  column values of fetched records are set in \ ``Item#name``\ .
    * - | (4)
      - | \ ``item_price``\  column values of fetched records are set in \ ``Item#price``\ .
    * - | (5)
      - | Delegate generation of \ ``Category``\  objects to \ ``id="order.categoryResultMap"``\  resultMap and add the generated objects to  ``Item#categories`` list.


Mapping the records to \ ``Category``\  objects.

 .. code-block:: xml

    <resultMap id="categoryResultMap" class="Category" groupBy="code"> <!-- (1) -->
        <result property="code" column="category_code" /> <!-- (1) -->
        <result property="name" column="category_name" /> <!-- (1) -->
    </resultMap>

 .. figure:: images/dataaccess_resultmap_category.png
    :alt: ResultMap for Item
    :width: 100%
    :align: center

    **Picture - ResultMap for Item**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No
      - Description
    * - | (1)
      - | In this example, multiple tables with 1:N relationship (t_order and t_order_line, t_order and t_order_coupon) are combined. As a result, if multiple records are stored in t_order_coupon, there is a duplication of list of \ ``Category``\  objects stored in \ ``Item``\  objects.
        | In order to avoid this duplication, specify \ ``code``\  property storing the value which uniquely identifies a category, in groupBy attribute. \ ``Category``\  objects with same \ ``code``\  property value are merged in one and thus preventing duplication.
        | In this example, \ ``Category``\  object \ ``code=CTG0000001``\  is generated for \ ``Item#code=ITM0000001``\  and 2 \ ``Category``\  objects namely, \ ``code=CTG0000002``\  and \ ``code=CTG0000003``\  are generated for \ ``Item#code=ITM0000002``\ . (Total 3 objects are generated.)
    * - | (2)
      - |  \ ``item_code``\  column values of fetched records are set in \ ``Item#code``\ .
    * - | (3)
      - | \ ``item_name``\  column values of fetched records are set in \ ``Item#name``\ .


Mapping the records to \ ``OrderCoupon``\  objects.

 .. code-block:: xml

    <resultMap id="orderCouponResultMap" class="OrderCoupon" groupBy="couponCode"> <!-- (1) -->
        <result property="couponCode" column="coupon_code" /> <!-- (2) -->
        <result property="coupon" resultMap="order.couponResultMap" /> <!-- (3) -->
    </resultMap>

 .. figure:: images/dataaccess_resultmap_ordercoupon.png
    :alt: ResultMap for OrderCoupon
    :width: 100%
    :align: center

    **Picture - ResultMap for OrderCoupon**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | It is necessary to specify groupBy attribute since there is a 1:N relationship between \ ``Order``\  and \ ``OrderCoupon``\ .
        | Order coupons need to be grouped with the primary keys of t_order_coupon (order_id, coupon_code). However, order_id column is specified in the parent resultMap. Hence, specify only the  \ ``couponCode``\  property that stores coupon_code column values here.
        | In this example, as the order coupons are grouped in \ ``couponCode``\  property, 2 \  ``OrderCoupon``\ objects, namely, \ ``couponCode=CPN0000001``\  and \ ``couponCode=CPN0000002``\  are generated.
    * - | (2)
      - | \ ``coupon_code``\  column values of fetched records are set in \ ``OrderCoupon#couponCode``\ .
        | \ ``couponCode``\  property overlaps with \ ``Coupon#code``\  generated in (3). However, the property is required for grouping \ ``OrderCoupon``\ .
    * - | (3)
      - | Delegate generation of \ ``Coupon``\  object to \ ``id="order.couponResultMap"``\  resultMap and set the generated objects in \ ``OrderCoupon#coupon``\ .


Mapping the records to \ ``Coupon``\  objects

 .. code-block:: xml

    <resultMap id="couponResultMap" class="Coupon"> <!-- (1) -->
        <result property="code" column="coupon_code" /> <!-- (2) -->
        <result property="name" column="coupon_name" /> <!-- (3) -->
        <result property="price" column="coupon_price" /> <!-- (4) -->
    </resultMap>

 .. figure:: images/dataaccess_resultmap_coupon.png
    :alt: ResultMap for Coupon
    :width: 100%
    :align: center

    **Picture - ResultMap for Coupon**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | It is not necessary to specify groupBy attribute since there is a 1:1 relationship between \ ``OrderCoupon``\  and \ ``Coupon``\ .
        | In this example, \ ``Coupon``\  object of \ ``code=CPN0000001``\ is generated for \ ``OrderCoupon#couponCode=CPN0000001``\  and \ ``Coupon``\  object of \ ``code=CPN0000001``\  is generated for \ ``OrderCoupon#couponCode=CPN0000001``\ . (Total 2 objects are generated.)
    * - | (2)
      - | \ ``coupon_code``\  column values of fetched records are set in \ ``Coupon#code``\ .
    * - | (3)
      - | \ ``coupon_name``\  column values of fetched records are set in \ ``Coupon#name``\ .
    * - | (4)
      - | \ ``coupon_price``\  column values of fetched records are set in \ ``Coupon#price``\ .


| Records and columns mapped to JavaBean are as follows:
| By specifying groupBy attribute, grayed out part is merged with the non-grayed part.

 .. figure:: images/dataaccess_sql_result_used.png
    :alt: Valid Result Set for result mapping
    :width: 100%
    :align: center

    **Picture - Valid Result Set for result mapping**


.. _data-access-mybatis2_warning_sqlmapping_bulk:

 .. warning::

     It should be noted that when the records with 1:N relationship are mapped using JOINs, fetching the data of grayed out part becomes unnecessary.

     If the same SQL is used in a process that does not use data from the N part, it results in fetching of unnecessary data. Therefore, two separate SQLs, one that fetches the N part and another that does not fetch the N part should be created.


The status of actually mapped  \ ``Order``\  objects and related objects is as follows:

 .. figure:: images/dataaccess_object.png
    :alt: Mapped object diagram
    :width: 90%
    :align: center

    **Picture - Mapped object diagram**

 .. tip::

     Another method to fetch related objects includes executing a separate SQL internally, using values of fetched records.
     Implementing a separate SQL internally simplifies the definition of resultMap element and individual SQL to a great extent.
     However, it is necessary to be aware of the fact that fetching related objects by this method can cause N+1.

     For details on how to execute a separate SQL internally, refer "Result Maps/Complex Properties" (P.36-37) and "Result Maps/Composite Keys or Multiple Complex Parameters Properties" (P.40-41) of Mybatis Developer Guide (PDF).


 .. tip::

     When a separate SQL is internally executed, related objects are "Eager Loaded". As a result, SQL gets executed even when the related objects are not used.
     To avoid this, Mybatis provides an option of "Lazy Load" for related objects.

     Settings to enable "Lazy Load" are as follows:

     * Set the enhancementEnabled attribute of setting element of the Mybatis configuration file, to \ ``true``\ .
     * Add CGLIB 2.x to class path.

.. raw:: latex

   \newpage

