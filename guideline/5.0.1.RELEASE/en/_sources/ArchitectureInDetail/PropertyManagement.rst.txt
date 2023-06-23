Properties Management
===================================================================================

.. only:: html

 .. contents:: Table of Contents
    :depth: 3
    :local:

Overview
--------------------------------------------------------------------------------

This section explains how to manage properties.

Value that needs to be managed as properties can be classified into following two categories.


.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.35\linewidth}|p{0.35\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 20 35 35

    * - Sr. No.
      - Classification
      - Description
      - Example
    * - 1.
      - Environment dependent setting
      - The setting needs to be changed according to the environment on which the application is running.

        It depends on non-functional requirements such as system configuration.
      - * Database connection information (connection URL, connection user, password, etc)
        * Destination of the file storage (directory path etc)
        * more ...
    * - 2.
      - Application settings
      - The settings that can be customize the application behavior.

        It depends on functional requirements.
      - * Password valid days
        * Reservation period days
        * more ...

.. note::

    In this guideline, it is recommended to manage these settings as properties (properties file).

    If an application is mechanized such a way that acquires setting from the properties,
    there is no need to re-build the application even if there is any changes in these values.
    Therefore it is possible to release the tested application on production environment.


    About how to release the tested application, refer to ":doc:`../Appendix/EnvironmentIndependency`".

.. tip::

    Values that are managed as properties can be acquired from JVM system properties (-D option) or OS environment variables.
    About access order, refer to ":ref:`PropertiesManagementHowToUse`".

|


Values that are managed as properties can be used at the following two locations.

* Bean definition file
* Java classes managed by DI container

|

.. _PropertiesManagementHowToUse:

How to use
--------------------------------------------------------------------------------

.. _technical-details_label:

About properties file definition
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Properties file values in Java class and bean definition file  can be accessed by defining ``<context:property-placeholder/>`` tag in bean definition file.
| ``<context:property-placeholder/>`` tag reads the group of specified properties files
| and can fetch values for properties files key \ ``xxx``\  specified in \ ``${xxx}``\  format in ``@Value`` annotation or bean definition files.

 .. note::

    When specified in \ ``${xxx:defaultValue}``\ format and when key \ ``xxx``\  is not set in properties file, \ ``defaultValue``\   is used.

|

See the method below for defining a properties file

**bean definition file**

- applicationContext.xml
- spring-mvc.xml

 .. code-block:: xml

    <context:property-placeholder location="classpath*:META-INF/spring/*.properties"/>  <!-- (1) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | In location, set the resource location path.
        | Multiple paths separated by comma can be specified in location attribute.
        | By performing above settings, read properties file under META-INF/spring directory of class path.
        | Once these settings are done, just add the properties file under META-INF/spring.
        | For details on location value, see \ `Reference <http://docs.spring.io/spring/docs/4.1.7.RELEASE/spring-framework-reference/html/resources.html>`_\ .

 .. note::

    \ ``<context:property-placeholder>``\  needs to be defined in both ``applicationContext.xml`` and ``spring-mvc.xml``.

|

Properties are accessed in the following order by default.

#. System properties of active JVM 
#. Environment variables
#. Application definition properties

| As per default setting, properties file defined in application is searched and read after all environment related properties (JVM system properties and environment variables) are read.
| Read sequence can be changed by setting local-override attribute of ``<context:property-placeholder/>`` tag to true.
| By performing these settings, the properties defined in application are enabled with higher priority.




**bean definition file**

 .. code-block:: xml

   <context:property-placeholder
       location="classpath*:META-INF/spring/*.properties" 
       local-override="true" /> <!-- (1) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Access properties in the following order when local-override attribute is set to true.
       | 1. Application definition properties
       | 2. System properties of active JVM
       | 3. Environment variables

|

 .. note::

        Normally the above settings are sufficient.
        When multiple ``<context:property-placeholder/>`` tags are specified, read order can be defined by setting order attribute value.

            **bean definition file**

            .. code-block:: xml

               <context:property-placeholder
                    location="classpath:/META-INF/property/extendPropertySources.properties"
                    order="1" ignore-unresolvable="true" /> <!-- (1) -->
               <context:property-placeholder
                    location="classpath*:/META-INF/spring/*.properties"
                    order="2" ignore-unresolvable="true" /> <!-- (2) -->

            .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
            .. list-table::
               :header-rows: 1
               :widths: 10 90

               * - Sr. No.
                 - Description
               * - | (1)
                 - | By setting the order attribute to a value which is less than (2), properties file corresponding to location attribute is read before (2).
                   | When a key overlapping with the key in properties file read in (2) exists, value fetched in (1) is given preference.
                   | By setting ignore-unresolvable attribute to true, error which occurs when key exists only in properties file of (2) can be prevented.
               * - | (2)
                 - | By setting the order attribute to value greater than (1), properties file corresponding to location attribute is read after (1).
                   | When a key overlapping with the key in properties file read in (1) exists, value fetched in (1) is set.
                   | By setting ignore-unresolvable attribute to true, error which occurs when key exists only in properties file of (1) can be prevented.

|

.. _bean-definition-file_label:

Using properties in bean definition file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| See the example below of datasource configuration file.
| In the following example, it is assumed that properties file definition ( ``<context:property-placeholder/>`` ) is specified.
| Basically, property value can be set by setting properties file key in bean definition file using ``${}`` placeholder.

**Properties file**

 .. code-block:: properties

   database.url=jdbc:postgresql://localhost:5432/shopping
   database.password=postgres
   database.username=postgres
   database.driverClassName=org.postgresql.Driver

|

**bean definition file**

 .. code-block:: xml

   <bean id="dataSource" 
       destroy-method="close" 
       class="org.apache.commons.dbcp2.BasicDataSource">
       <property name="driverClassName" 
                 value="${database.driverClassName}"/>  <!-- (1) -->
       <property name="url" value="${database.url}"/>  <!-- (2) -->
       <property name="username" value="${database.username}"/>  <!-- (3) -->
       <property name="password" value="${database.password}"/>  <!-- (4) -->
       <!-- omitted -->
   </bean>

|

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | By setting ``${database.driverClassName}``, the value for read properties file key \ ``database.driverClassName``\  gets substituted.
   * - | (2)
     - | By setting ``${database.url}``, the value for read properties file key \ ``database.url``\  gets substituted.
   * - | (3)
     - | By setting ``${database.username}``, the value for read properties file key \ ``database.username``\  gets substituted.
   * - | (4)
     - | By setting ``${database.password}``, the value for read properties file key \ ``database.password``\  gets substituted.

|

As a result of reading the properties file key, the values are replaced as follows:

 .. code-block:: xml

   <bean id="dataSource" 
       destroy-method="close" 
       class="org.apache.commons.dbcp2.BasicDataSource">
       <property name="driverClassName" value="org.postgresql.Driver"/>
       <property name="url" 
                 value="jdbc:postgresql://localhost:5432/shopping"/>
       <property name="username" value="postgres"/>
       <property name="password" value="postgres"/>
       <!-- omitted -->
   </bean>

|

Using properties in Java class
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| It is possible to use properties in Java class by specifying ``@Value`` annotation in the field wherein properties values are to be stored.
| To use ``@Value`` annotation, the corresponding object needs to be stored in DI container of Spring.

| In the following example, it is assumed that properties file definition ( ``<context:property-placeholder/>`` ) is specified.
| External reference is possible by adding ``@Value``  annotation to variables and setting properties file key in value using ``${}`` placeholder.

**Properties file**

 .. code-block:: properties

   item.upload.title=list of update file
   item.upload.dir=file:/tmp/upload
   item.upload.maxUpdateFileNum=10

**Java class**

 .. code-block:: java

   @Value("${item.upload.title}")  // (1)
   private String uploadTitle;

   @Value("${item.upload.dir}")  // (2)
   private Resource uploadDir;

   @Value("${item.upload.maxUpdateFileNum}")  // (3)
   private int maxUpdateFileNum;

   // Getters and setters omitted

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | By setting ``${item.upload.title}`` to ``@Value`` annotation value, the value for read properties file key \ ``item.upload.title``\  gets substituted.
       | ``uploadTitle`` is substituted by "list of update file" in String class.
   * - | (2)
     - | By setting ``${item.upload.dir}`` to ``@Value`` annotation value, the value for read properties file key \ ``item.upload.dir``\  gets substituted.
       | \ ``org.springframework.core.io.Resource``\  object created with initial value "/tmp/upload" is stored in ``uploadDir``.
   * - | (3)
     - | By setting ``${item.upload.maxUpdateFileNum}`` to ``@Value`` annotation value, the value for read properties file key \ ``item.upload.maxUpdateFileNum``\  gets substituted.
       | ``maxUpdateFileNum`` is substituted by 10.

 .. warning::

        There could be cases wherein properties values are to be used in static methods of Utility classes etc.; however properties value cannot be fetched using \ ``@Value``\  annotation in classes for which bean definition is not done.
        In this case, it is recommended to create Helper class with ``@Component``  annotation and to fetch properties values using \ ``@Value``\  annotation. (This class needs to be included in the component-scan scope.)
        Classes in which values from properties file is to be used, should not be made Utility classes.

|

How to extend
--------------------------------------------------------------------------------
Extension of method for fetching properties values is explained below. This can be achieved by
extending ``org.springframework.context.support.PropertySourcesPlaceholderConfigurer`` class.

The example below illustrates a case wherein encrypted properties file is used.

|

Decrypting encrypted values and using them
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| To strengthen security, properties file needs to be encrypted in some cases.
| The example below illustrates decryption of encrypted properties values. (No specific encrypting and decrypting methods are mentioned.)

**bean definition file**

- applicationContext.xml
- spring-mvc.xml

 .. code-block:: xml

    <!-- (1) -->
    <bean class="com.example.common.property.EncryptedPropertySourcesPlaceholderConfigurer">
        <!-- (2) -->
        <property name="locations" 
                  value="classpath*:/META-INF/spring/*.properties" />
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Define the extended PropertySourcesPlaceholderConfigurer instead of  ``<context:property-placeholder/>``\. ``<context:property-placeholder/>``\  tag should be deleted.
   * - | (2)
     - | Set "locations" in name attribute of property tag and specify the path of the properties file to be read, in value attribute.
       | The method of specifying path of the properties file to be read is same as :ref:`technical-details_label`.

**Java class**

- Extended PropertySourcesPlaceholderConfigurer

 .. code-block:: java

    public class EncryptedPropertySourcesPlaceholderConfigurer extends 
        PropertySourcesPlaceholderConfigurer { // (1)
        @Override
        protected void doProcessProperties(
                ConfigurableListableBeanFactory beanFactoryToProcess,
                StringValueResolver valueResolver) { // (2)
            super.doProcessProperties(beanFactoryToProcess, 
                new EncryptedValueResolver(valueResolver)); // (3)
        }
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Inherited PropertySourcesPlaceholderConfigurer, should extend ``org.springframework.context.support.PropertySourcesPlaceholderConfigurer`` class.
   * - | (2)
     - | Override ``doProcessProperties`` method of ``org.springframework.context.support.PropertySourcesPlaceholderConfigurer`` class.
   * - | (3)
     - | Call ``doProcessProperties`` of parent class; however, use valueResolver( ``EncryptedValueResolver`` ) ``valueResolver`` wherein ``valueResolver`` is implemented independently.
       | In ``EncryptedValueResolver`` class, decrypt when encrypted values of properties file are fetched.

|

- EncryptedValueResolver.java

 .. code-block:: java

    public class EncryptedValueResolver implements 
                                        StringValueResolver { // (1)

        private final StringValueResolver valueResolver;

        EncryptedValueResolver(StringValueResolver stringValueResolver) { // (2)
            this.valueResolver = stringValueResolver;
        }

        @Override
        public String resolveStringValue(String strVal) { // (3)

            // Values obtained from the property file to the naming
            // as seen with the encryption target
            String value = valueResolver.resolveStringValue(strVal); // (4)

            // Target messages only, implement coding
            if (value.startsWith("Encrypted:")) { // (5)
                value =  value.substring(10); // (6)
                // omitted decryption
            }
            return value;
        }
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Inherited ``EncryptedValueResolver`` should implement ``org.springframework.util.StringValueResolver``.
   * - | (2)
     - | When ``EncryptedValueResolver``  class is created in constructor, set ``StringValueResolver`` inherited from ``EncryptedPropertySourcesPlaceholderConfigurer``.
   * - | (3)
     - | Override ``resolveStringValue`` method of ``org.springframework.util.StringValueResolver``.
       | If the values fetched from properties file are encrypted, these must be decrypted in ``resolveStringValue`` method.
       | The process mentioned in steps (5) and (6) is just an example, the process differs depending on type of implementation.
   * - | (4)
     - | The value is being fetched by specifying key as an argument of ``resolveStringValue``  method of ``StringValueResolver`` set in constructor. This value is defined in properties file.
   * - | (5)
     - | Check whether values of properties file are encrypted. The method to determine whether the values are encrypted differs depending on type of implementation.
       | Here, the value can be considered encrypted if it starts with "Encrypted:".
       | If the values are encrypted, decrypt them in step (6) and if they are not encrypted, return them as is.
   * - | (6)
     - | Encrypted values of properties file are being decrypted. (No specific decryption process is mentioned.)
       | Decryption method differs depending on type of implementation.

- Helper to fetch properties

 .. code-block:: java

    @Value("${encrypted.property.string}") // (1)
    private String testString;

    @Value("${encrypted.property.int}") // (2)
    private int testInt;

    @Value("${encrypted.property.integer}") // (3)
    private Integer testInteger;

    @Value("${encrypted.property.file}") // (4)
    private File testFile;

    // Getters and setters omitted

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | By setting ``${encrypted.property.string}`` to ``@Value`` annotation value, the value for read properties file key \ ``encrypted.property.string``\  is decrypted and then substituted.
       | Value decrypted in String class is substituted in  ``testString``.
   * - | (2)
     - | By setting ``${encrypted.property.int}`` to ``@Value``  annotation value, the value for read properties file key \ ``encrypted.property.int``\  is decrypted and then substituted.
       | Value decrypted in integer type is substituted in ``testInt``.
   * - | (3)
     - | By setting ``${encrypted.property.integer}`` to ``@Value``  annotation value, the value for read properties file key \ ``encrypted.property.integer``\  is decrypted and then substituted.
       | Value decrypted in Integer class is substituted in ``testInteger``.
   * - | (4)
     - | By setting ``${encrypted.property.file}`` to ``@Value`` annotation value, the value for read properties file key \ ``encrypted.property.file``\  is decrypted and then substituted.
       | In ``testFile``, File object is stored as initial value which is created using the decrypted value (auto conversion). 

**Properties file**

The values encrypted as properties values are prefixed with "Encrypted:" to indicate that they are encrypted.
Although one can view the contents of properties file, but cannot understand them as the values are encrypted.

 .. code-block:: properties

   encrypted.property.string=Encrypted:ZlpbQRJRWlNAU1FGV0ASRVteXhJQVxJXXFFAS0JGV1Yc
   encrypted.property.int=Encrypted:AwI=
   encrypted.property.integer=Encrypted:AwICAgI=
   encrypted.property.file=Encrypted:YkBdQldARkt/U1xTVVdfV1xGHFpGX14=

.. raw:: latex

   \newpage

