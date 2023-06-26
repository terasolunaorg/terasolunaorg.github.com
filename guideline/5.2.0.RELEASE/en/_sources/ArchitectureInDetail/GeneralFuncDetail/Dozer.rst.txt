Bean Mapping (Dozer)
--------------------------------------------------------------------------------

.. only:: html

 .. contents:: Table of Contents
   :depth: 4
   :local:

|

Overview
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Bean mapping is a process to copy the field values of one Bean to another Bean.

| There are many cases where bean mapping is necessary while passing the data between different layers of the application (application layer and domain layer).

| For example, \ ``AccountForm``\  object of application layer is converted to \ ``Account``\  object of domain layer.
| Since the domain layer should not depend on the application layer, AccountForm object cannot be used as it is in the domain layer.
| Hence, \ ``AccountForm``\  object is Bean mapped to \ ``Account``\  object and \ ``Account``\  object is used in the domain layer.
| Thus, linear dependency relation can be maintained between application layer and domain layer.

.. figure:: ./images/beanmapping-overview.png
   :width: 80%

| These objects can be mapped by calling getter/setter of Bean and passing the data.
| However, since a complex process results in deterioration of the program readability, in this guideline, it is recommended to use `Dozer <http://dozer.sourceforge.net>`_  a Bean mapping library available in OSS.

| By using Dozer, different types of copies for copy source class and copy destination class and copy of nested Beans can be easily performed as shown in the following figure.

.. figure:: ./images/dozer-functionality-overview.png
   :width: 75%

Code examples with/without using Dozer are given below.

* Example of a complex process that results in the program readability deterioration

    .. code-block:: java
    
        User user = userService.findById(userId);

        XxxOutput output = new XxxOutput();
    
        output.setUserId(user.getUserId());
        output.setFirstName(user.getFirstName());
        output.setLastName(user.getLastName());
        output.setTitle(user.getTitle());
        output.setBirthDay(user.getBirthDay());
        output.setGender(user.getGender());
        output.setStatus(user.getStatus());

* Example when Dozer is used

    .. code-block:: java

        User user = userService.findById(userId);

        XxxOutput output = beanMapper.map(user, XxxOutput.class);


How to use Dozer is explained below.

 .. warning::

    Classes added by JSR-310 Date and Time API cannot be used in default state due to generation of exceptions, and hence a custom converter must be created.

|

How to use
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Dozer is a mapping function library of JavaBean.
Values are copied recursively (nested structure) from conversion source Bean to conversion destination Bean.

.. _bean-mapper-definition:

Bean definition for using Dozer
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Create an instance of ``org.dozer.Mapper`` when Dozer is to be used independently as follows.

.. code-block:: java

    Mapper mapper = new DozerBeanMapper();


Creating instance of Mapper each time deteriorates efficiency,
hence ``org.dozer.spring.DozerBeanMapperFactoryBean`` provided by Dozer should be used.


Define \ ``org.dozer.spring.DozerBeanMapperFactoryBean``\ , a Factory class that creates Mapper in Bean definition file (applicationContext.xml).

.. code-block:: xml

    <bean class="org.dozer.spring.DozerBeanMapperFactoryBean">
        <property name="mappingFiles"
            value="classpath*:/META-INF/dozer/**/*-mapping.xml" /><!-- (1) -->
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify mapping definition XML files in mappingFiles.
       | ``org.dozer.spring.DozerBeanMapperFactoryBean`` maintains ``org.dozer.Mapper`` as an interface. Therefore, specify ``Mapper`` for ``@Inject``.
       | In this example, all the (any value)-mapping.xmls in any folder of /META-INF/dozer under the class path
       | are read. The contents of these XML files are explained later.

|


``Mapper`` should be injected in the class to perform Bean mapping.

.. code-block:: java

    @Inject
    Mapper beanMapper;


.. _beanconverter-basic-mapping-label:

Mapping when the field name and the type between Beans is same
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Dozer can perform mapping by default without creating mapping definition XML file if the field name between the Beans is same.

Bean definition of conversion source

.. code-block:: java

    public class Source {
        private int id;
        private String name;
        // omitted setter/getter
    }


Bean definition of conversion destination

.. code-block:: java

    public class Destination {
        private int id;
        private String name;
        // omitted setter/getter
    }


Perform Bean mapping using ``map`` method of ``Mapper`` as given below.
After executing the method given below, a new Destination object is created and each field value of source is copied to the created Destination object.

.. code-block:: java

    Source source = new Source();
    source.setId(1);
    source.setName("SourceName");
    Destination destination = beanMapper.map(source, Destination.class); // (1)
    System.out.println(destination.getId());
    System.out.println(destination.getName());

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Pass the object to be copied as the first argument and the Bean class where it is to be copied as the second argument.


The output of the above code is as given below. The value of object to be copied is set in the created object.

.. code-block:: console

    1
    SourceName


To copy the field of \ ``source``\  object to already existing \ ``destination``\  object, perform the following

.. code-block:: java

    Source source = new Source();
    source.setId(1);
    source.setName("SourceName");
    Destination destination = new Destination();
    destination.setId(2);
    destination.setName("DestinationName");
    beanMapper.map(source, destination); // (1)
    System.out.println(destination.getId());
    System.out.println(destination.getName());

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Pass the object to be copied as the first argument and the object where it is to be copied as the second argument.


The output of the above code is as given below. The value of object to be copied is reflected in the destination where it is to be copied.

.. code-block:: console

    1
    SourceName

.. note::

    The value that does not exist in \ ``Source``\  class does not change before and after copying to the field of \ ``Destination``\  class.
    
    Bean definition of conversion source
    
        .. code-block:: java
        
            public class Source {
                private int id;
                private String name;
                // omitted setter/getter
            }
    
    
    Bean definition of conversion destination
    
        .. code-block:: java
        
            public class Destination {
                private int id;
                private String name;
                private String title;
                // omitted setter/getter
            }
    
    Example of Mapping
    
        .. code-block:: java
        
            Source source = new Source();
            source.setId(1);
            source.setName("SourceName");
            Destination destination = new Destination();
            destination.setId(2);
            destination.setName("DestinationName");
            destination.setTitle("DestinationTitle");
            beanMapper.map(source, destination);
            System.out.println(destination.getId());
            System.out.println(destination.getName());
            System.out.println(destination.getTitle());
    
    
    The output of the above code is as given below. Since there is no \ ``title``\  field in the \ ``Source``\  class,
    the value of \ ``title``\  field of \ ``Destination``\  object remains the same as the field value before copying.
    
        .. code-block:: console
        
            1
            SourceName
            DestinationTitle

.. _beanconverter-difference-type-mapping-label:

Mapping when the field name is the same and type between Beans is different
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When field type of Bean is different in copy source and copy destination,
mapping of the type where type conversion is supported can be done automatically.

The conversion given below is an example where it is possible to convert without a mapping definition XML file.

Example: String -> BigDecimal

Bean definition of conversion source

.. code-block:: java

    public class Source {
        private String amount;
        // omitted setter/getter
    }


Bean definition of conversion destination

.. code-block:: java

    public class Destination {
        private BigDecimal amount;
        // omitted setter/getter
    }


Example of Mapping

.. code-block:: java

    Source source = new Source();
    source.setAmount("123.45");
    Destination destination = beanMapper.map(source, Destination.class);
    System.out.println(destination.getAmount());


The output of the above code is as given below. The value can be copied even when the type is different.

.. code-block:: console

    123.45

Refer to `Manual <http://dozer.sourceforge.net/documentation/simpleproperty.html>`_ for the supported type conversions.


.. _beanconverter-difference-item-xml-mapping-label:

Mapping when the field name between Beans is different
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When field name of copy source is different from the copy destination, creating mapping definition XML file
and defining the field for Bean mapping enables conversion.

Bean definition of conversion source

.. code-block:: java

    public class Source {
        private int id;
        private String name;
        // omitted setter/getter
    }


Bean definition of conversion destination

.. code-block:: java

    public class Destination {
        private int destinationId;
        private String destinationName;
        // omitted setter/getter
    }


To define :ref:`bean-mapper-definition`\,
create mapping definition XML file called (any value)-mapping.xml in the src/main/resources/META-INF/dozer folder.

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <mappings xmlns="http://dozer.sourceforge.net" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://dozer.sourceforge.net
              http://dozer.sourceforge.net/schema/beanmapping.xsd">

        <mapping>
          <class-a>com.xx.xx.Source</class-a><!-- (1) -->
          <class-b>com.xx.xx.Destination</class-b><!-- (2) -->
          <field>
            <a>id</a><!-- (3) -->
            <b>destinationId</b><!-- (4) -->
          </field>
          <field>
            <a>name</a>
            <b>destinationName</b>
          </field>
        </mapping>

    </mappings>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify fully qualified class name (FQCN) of copy source Bean in \ ``<class-a>``\  tag.
   * - | (2)
     - | Specify fully qualified class name (FQCN) of copy destination Bean in \ ``<class-b>``\  tag.
   * - | (3)
     - | Specify field name for mapping of copy source Bean in \ ``<a>``\  tag of \ ``<field>``\  tag.
   * - | (4)
     - | Specify field name for mapping of copy destination Bean corresponding to (3) in \ ``<b>``\  tag of \ ``<field>``\  tag.

Example of Mapping

.. code-block:: java

    Source source = new Source();
    source.setId(1);
    source.setName("SourceName");
    Destination destination = beanMapper.map(source, Destination.class); // (1)
    System.out.println(destination.getDestinationId());
    System.out.println(destination.getDestinationName());

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Pass object to be copied as the first argument and the Bean class where it is to be copied as the second argument. (no difference with basic mapping.)


The output of the above code is as given below.

.. code-block:: console

    1
    SourceName


Mapping definition XML file existing under META-INF/dozer of class path is read in \ ``mappingFiles``\  property in accordance with the setting of :ref:`bean-mapper-definition`\.
File name should be (any value)-mapping.xml.
The settings are applied if mapping between \ ``Source``\  class and \ ``Destination``\  class is defined in any file.


.. note::
    It is recommended to create a mapping definition XML file in each Controller and name the file as -mapping.xml (value obtained by removing Controller from Controller name).
    For example, mapping definition XML file of TodoController is created in src/main/resources/META-INF/dozer/todo-mapping.xml.

|

One-way/Two-way mapping
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. _beanconverter-one-two-way-mapping-label:

The mapping defined in mapping XML is two-way mapping by default.
In the above example, mapping is done from \ ``Source``\  object to \ ``Destination``\  object however,
mapping can also be done from \ ``Destination``\  object to \ ``Source``\  object.

To specify only one-way mapping, set \ ``"one-way"``\  in the \ ``type``\  attribute of \ ``<mapping>``\  tag in the mapping field definition.

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <mappings xmlns="http://dozer.sourceforge.net" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://dozer.sourceforge.net
              http://dozer.sourceforge.net/schema/beanmapping.xsd">
            <!-- omitted -->
            <mapping type="one-way">
                  <class-a>com.xx.xx.Source</class-a>
                  <class-b>com.xx.xx.Destination</class-b>
                    <field>
                      <a>id</a>
                      <b>destinationId</b>
                    </field>
                    <field>
                      <a>name</a>
                      <b>destinationName</b>
                    </field>
            </mapping>
            <!-- omitted -->
    </mappings>


Bean definition of conversion source

.. code-block:: java

    public class Source {
        private int id;
        private String name;
        // omitted setter/getter
    }


Bean definition of conversion destination

.. code-block:: java

    public class Destination {
        private int destinationId;
        private String destinationName;
        // omitted setter/getter
    }

Example of Mapping

.. code-block:: java

    Source source = new Source();
    source.setId(1);
    source.setName("SourceName");
    Destination destination = beanMapper.map(source, Destination.class);
    System.out.println(destination.getDestinationId());
    System.out.println(destination.getDestinationName());


The output of the above code is as given below.

.. code-block:: console

    1
    SourceName


If one-way is specified, an error does not occur even if the mapping is done in the reverse direction. The copy process is ignored.
This is because \ ``Source``\  field corresponding to \ ``Destination``\  field does not exist if mapping is not defined.

.. code-block:: java

    Destination destination = new Destination();
    destination.setDestinationId(2);
    destination.setDestinationName("DestinationName");

    Source source = new Source();
    source.setId(1);
    source.setName("SourceName");

    beanMapper.map(destination, source);

    System.out.println(source.getId());
    System.out.println(source.getName());

The output of the above code is as given below.

.. code-block:: console

    1
    SourceName

.. _beanconverter-custom-converter-label:


Mapping of nested fields
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
It is possible to map the field of Bean to be copied to the field with Nested attributes of the Bean where it is to be copied.
(In Dozer terminology, it is called `Deep Mapping <http://dozer.sourceforge.net/documentation/deepmapping.html>`_ .)


Bean definition of conversion source

.. code-block:: java

    public class EmployeeForm {
        private int id;
        private String name;
        private String deptId;
        // omitted setter/getter
    }


Bean definition of conversion destination

.. code-block:: java

    public class Employee {
        private Integer id;
        private String name;
        private Department department;
        // omitted setter/getter
    }

.. code-block:: java

    public class Department {
        private String deptId;
        // omitted setter/getter and other fields
    }

Example: Define as below to map the \ ``deptId``\  with \ ``EmployeeForm``\  object to \ ``deptId``\  of \ ``Department``\ with \ ``Employee``\  object.


.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <mappings xmlns="http://dozer.sourceforge.net" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://dozer.sourceforge.net
              http://dozer.sourceforge.net/schema/beanmapping.xsd">
        <!-- omitted -->
        <mapping map-empty-string="false" map-null="false">
            <class-a>com.xx.aa.EmployeeForm</class-a>
            <class-b>com.xx.bb.Employee</class-b>
            <field>
                  <a>deptId</a>
                  <b>department.deptId</b><!-- (1) -->
            </field>
        </mapping>
        <!-- omitted -->
    </mappings>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify the field of \ ``Employee``\  object for \ ``deptId``\  of \ ``Employee``\  form.


Example of Mapping

.. code-block:: java

    EmployeeForm source = new EmployeeForm();
    source.setId(1);
    source.setName("John");
    source.setDeptId("D01");

    Employee destination = beanMapper.map(source, Employee.class);
    System.out.println(destination.getId());
    System.out.println(destination.getName());
    System.out.println(destination.getDepartment().getDeptId());


The output of the above code is as given below.

.. code-block:: console

    1
    John
    D01

In the above case, a new instance of \ ``Employee``\ , a class of conversion destination is created.
The newly created \ ``Department``\  instance is set in the \ ``department``\  field of \ ``Employee``\  and
\ ``deptId``\  of the \ ``EmployeeForm``\  is copied.

When the \ ``Department``\  object is already set in the \ ``department``\  field of \ ``Employee``\ ,
a new instance is not created but the \ ``deptId``\  of \ ``EmployeeForm``\  is copied
to the \ ``deptId``\  field of the existing \ ``Department``\  object.

.. code-block:: java

    EmployeeForm source = new EmployeeForm();
    source.setId(1);
    source.setName("John");
    source.setDeptId("D01");

    Employee destination = new Employee();
    Department department = new Department();
    destination.setDepartment(department);

    beanMapper.map(source, destination);
    System.out.println(department.getDeptId());
    System.out.println(destination.getDepartment() == department);


The output of the above code is as given below.

.. code-block:: console

    D01
    true

|

Collection mapping
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Dozer supports two-way auto-mapping of the following Collection types.
When field name is same, mapping definition XML file is not required.

* ``java.util.List`` <=> ``java.util.List``
* ``java.util.List`` <=> Array
* Array <=> Array
* ``java.util.Set`` <=> ``java.util.Set``
* ``java.util.Set`` <=> Array
* ``java.util.Set`` <=> ``java.util.List``



Bean mapping with a collection of the following classes is considered.

.. code-block:: java

    package com.example.dozer;
    
    public class Email {
        private String email;
    
        public Email() {
        }
    
        public Email(String email) {
            this.email = email;
        }
    
        public String getEmail() {
            return email;
        }
    
        public void setEmail(String email) {
            this.email = email;
        }
    
        @Override
        public String toString() {
            return email;
        }
    
        // generated by Eclipse
        @Override
        public int hashCode() {
            final int prime = 31;
            int result = 1;
            result = prime * result + ((email == null) ? 0 : email.hashCode());
            return result;
        }
    
        // generated by Eclipse
        @Override
        public boolean equals(Object obj) {
            if (this == obj)
                return true;
            if (obj == null)
                return false;
            if (getClass() != obj.getClass())
                return false;
            Email other = (Email) obj;
            if (email == null) {
                if (other.email != null)
                    return false;
            } else if (!email.equals(other.email))
                return false;
            return true;
        }
    
    }


Conversion source Bean

.. code-block:: java

    package com.example.dozer;
    
    import java.util.List;
    
    public class AccountForm {
        private List<Email> emails;
    
        public void setEmails(List<Email> emails) {
            this.emails = emails;
        }
    
        public List<Email> getEmails() {
            return emails;
        }
    }

Conversion destination Bean

.. code-block:: java

    package com.example.dozer;
    
    import java.util.List;
    
    public class Account {
        private List<Email> emails;
    
        public void setEmails(List<Email> emails) {
            this.emails = emails;
        }
    
        public List<Email> getEmails() {
            return emails;
        }
    }


Example of Mapping


.. code-block:: java

    AccountForm accountForm = new AccountForm();

    List<Email> emailsSrc = new ArrayList<Email>();

    emailsSrc.add(new Email("a@example.com"));
    emailsSrc.add(new Email("b@example.com"));
    emailsSrc.add(new Email("c@example.com"));

    accountForm.setEmails(emailsSrc);

    Account account = beanMapper.map(accountForm, Account.class);

    System.out.println(account.getEmails());


The output of the above code is as given below.

.. code-block:: console

    [a@example.com, b@example.com, c@example.com]

There is no particular change in the explanation given so far.

As shown in the example below, \ **it is necessary to exercise caution when the element is already added to the Collection field of copy destination Bean.**\ 

.. code-block:: java

    AccountForm accountForm = new AccountForm();
    Account account = new Account();

    List<Email> emailsSrc = new ArrayList<Email>();
    List<Email> emailsDest = new ArrayList<Email>();

    emailsSrc.add(new Email("a@example.com"));
    emailsSrc.add(new Email("b@example.com"));
    emailsSrc.add(new Email("c@example.com"));

    emailsDest.add(new Email("a@example.com"));
    emailsDest.add(new Email("d@example.com"));
    emailsDest.add(new Email("e@example.com"));

    accountForm.setEmails(emailsSrc);
    account.setEmails(emailsDest);

    beanMapper.map(accountForm, account);

    System.out.println(account.getEmails());


The output of the above code is as given below.

.. code-block:: console

    [a@example.com, d@example.com, e@example.com, a@example.com, b@example.com, c@example.com]

All the elements of Collection of copy source Bean are added to the Collection of copy destination Bean.
Two \ ``Email``\  objects with \ ``a@exmample.com``\  are  "Equal", but are added easily.

("Equal" here signifies \ ``true``\  when compared by \ ``Email.equals``\  and has the same value as \ ``Email.hashCode``\ .)

The above behavior is called as \ **cumulative**\  in Dozer terminology and is a default behavior at the time of Collection mapping.

This behavior can be changed in mapping definition XML file.

.. code-block:: xml
    :emphasize-lines: 9

    <?xml version="1.0" encoding="UTF-8"?>
    <mappings xmlns="http://dozer.sourceforge.net" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://dozer.sourceforge.net
              http://dozer.sourceforge.net/schema/beanmapping.xsd">
        <!-- omitted -->
        <mapping>
            <class-a>com.example.dozer.AccountForm</class-a>
            <class-b>com.example.dozer.Account</class-b>
            <field relationship-type="non-cumulative"><!-- (1) -->
                <a>emails</a>
                <b>emails</b>
            </field>
        </mapping>
        <!-- omitted -->
    </mappings>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify \ ``non-cumulative``\  in  \ ``relationship-type``\  attribute of \ ``<field>``\  tag. Default value is \ ``cumulative``\ .
       | 
       | To specify \ ``non-cumulative``\  in all the fields of Bean to be mapped, specify \ ``non-cumulative``\  in the \ ``relationship-type``\  attribute of \ ``<mapping>``\  tag.

The output of the above code is as given below based on this setting.

.. code-block:: console

    [a@example.com, d@example.com, e@example.com, b@example.com, c@example.com]

Duplication of equivalent objects is eliminated.

.. note::

    It is necessary to exercise caution for updating conversion source object with conversion destination object.
    In the above example, \ ``a@exmample.com``\  of \ ``AccountForm``\  is stored in the copy destination.
    
    
        .. figure:: ./images/dozer_noncumulativeupdate.png
           :alt: noncumulative update using dozer
           :width: 60%

It can be implemented using the mapping definition XML file settings even to remove the fields existing only in the copy destination collection.

.. code-block:: xml
    :emphasize-lines: 9

    <?xml version="1.0" encoding="UTF-8"?>
    <mappings xmlns="http://dozer.sourceforge.net" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://dozer.sourceforge.net
              http://dozer.sourceforge.net/schema/beanmapping.xsd">
        <!-- omitted -->
        <mapping>
            <class-a>com.example.dozer.AccountForm</class-a>
            <class-b>com.example.dozer.Account</class-b>
            <field relationship-type="non-cumulative" remove-orphans="true" ><!-- (1) -->
                <a>emails</a>
                <b>emails</b>
            </field>
        </mapping>
        <!-- omitted -->
    </mappings>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Set \ ``true``\  in the \ ``remove-orphans``\  attribute of \ ``<field>``\  tag. Default value is \ ``false``\ .


The output of the above code is as given below based on this setting.

.. code-block:: console

    [a@example.com, b@example.com, c@example.com]

Only the objects to be copied are reflected in the collection where they are to be copied.

The same result is obtained even with the settings given below.

.. code-block:: xml
    :emphasize-lines: 9

    <?xml version="1.0" encoding="UTF-8"?>
    <mappings xmlns="http://dozer.sourceforge.net" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://dozer.sourceforge.net
              http://dozer.sourceforge.net/schema/beanmapping.xsd">
        <!-- omitted -->
        <mapping>
            <class-a>com.example.dozer.AccountForm</class-a>
            <class-b>com.example.dozer.Account</class-b>
            <field copy-by-reference="true"><!-- (1) -->
                <a>emails</a>
                <b>emails</b>
            </field>
        </mapping>
        <!-- omitted -->
    </mappings>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Set \ ``true``\  in the \ ``copy-by-reference``\  attribute of \ ``<field>``\  tag. Default value is \ ``false``\ .

The behavior explained so far is given in the figure.

* Default behavior (cumulative)

    .. figure:: ./images/dozer-collection-cumulative.png
       :width: 60%

* non-cumulative

    .. figure:: ./images/dozer-collection-non-cumulative.png
       :width: 60%

* non-cumulative and remove-orphans=true

    .. figure:: ./images/dozer-collection-non-cumulative-and-orphan-remove.png
       :width: 60%

    copy-by-reference is also of this pattern.

.. note::

  The difference in "non-cumulative and remove-orphans=true" pattern and "copy-by-reference" pattern is whether the container of Collection after Bean conversion is copy destination or copy source.
  
  In case of "non-cumulative and remove-orphans=true" pattern, the container of Collection after Bean conversion is copy destination whereas in case of "copy-by-reference" , the container is copy source.
  It is explained in the figure given below.

  * non-cumulative and remove-orphans=true

    .. figure:: ./images/dozer-differrence1.png
       :width: 50%

  * copy-by-reference
    
    .. figure:: ./images/dozer-differrence2.png
       :width: 50%
  
  \ **It is necessary to exercise caution when the copy destination has one-to many and many-to-many relationship in the entity of JPA (Hibernate)**\ . An unexpected issue may occur when copy destination entity is under EntityManager control.
  For example, an SQL of DELETE ALL + INSERT ALL may be issued when the container of collection is changed and an SQL to UPDATE (DELETE or INSERT when the number of elements differ) the changes may be issued when copied with "non-cumulative and remove-orphans=true".
  Any pattern can be used depending on the requirements.


.. warning::

    When the Bean to be mapped has \ ``String``\  collection, a \ `bug wherein the behavior is not as expected <http://sourceforge.net/p/dozer/bugs/382/>`_\  is generated.
    
        .. code-block:: java
        
            StringListSrc src = new StringListSrc;
            StringListDest dest = new StringListDest();
        
            List<String> stringsSrc = new ArrayList<String>();
            List<String> stringsDest = new ArrayList<String>();
        
            stringsSrc.add("a");
            stringsSrc.add("b");
            stringsSrc.add("c");
        
            stringsDest.add("a");
            stringsDest.add("d");
            stringsDest.add("e");
        
            src.setStrings(stringsSrc);
            dest.setStrings(stringsDest);
        
            beanMapper.map(src, dest);
        
            System.out.println(dest.getStrings());
    
    If code given above is executed in the non-cumulative and remove-orphans=true settings, following is expected to be output.
    
        .. code-block:: console
    
            [a, b, c]

    However, following is output.
    
        .. code-block:: console
    
            [b, c]

    and \ **duplicate String is removed**\ .
    
    When the code is executed in the copy-by-reference="true" settings, following is output.
    
        .. code-block:: console
    
            [a, b, c]

    

.. tip::

   In Dozer, the mapping can be performed even between the lists which do not use Generics. At this time, data type of the object included in conversion source and conversion destination can be specified as HINT.
   Refer to `Dozer Official Manual -Collection and Array Mapping(Using Hints for Collection Mapping)- <http://dozer.sourceforge.net/documentation/collectionandarraymapping.html>`_ for details.

.. todo::

    It is checked that the mapping between Beans that uses Collection<T> fails.

    Example :

        .. code-block:: java
        
            public class ListNestedBean<T> {
               private List<T> nest;
               // omitted other declarations
            }

     Execution result :
     
        .. code-block:: console
        
            java.lang.ClassCastException: sun.reflect.generics.reflectiveObjects.TypeVariableImpl cannot be cast to java.lang.Class


How to extend
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Creating custom convertor
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Mapping of the data type not supported by Dozer can be performed through a custom converter.

* Example : ``java.lang.String`` <=> ``org.joda.time.DateTime``

| Custom converter is a class that implements ``org.dozer.CustomConverter`` provided by Dozer.
| Custom converter can be specified in the following 3 patterns.

* Global Configuration
* Class level
* Field level

Global Configuration is recommended to perform conversion using the same logic in the entire application.

It is convenient to inherit \ ``org.dozer.DozerConverter``\  to implement a custom converter.

.. code-block:: java

    package com.example.yourproject.common.bean.converter;
  
    import org.dozer.DozerConverter;
    import org.joda.time.DateTime;
    import org.joda.time.format.DateTimeFormat;
    import org.joda.time.format.DateTimeFormatter;
    import org.springframework.util.StringUtils;
  
    public class StringToJodaDateTimeConverter extends
                                              DozerConverter<String, DateTime> { // (1)
        public StringToJodaDateTimeConverter() {
            super(String.class, DateTime.class); // (2)
        }
  
        @Override
        public DateTime convertTo(String source, DateTime destination) {// (3)
            if (!StringUtils.hasLength(source)) {
                return null;
            }
            DateTimeFormatter formatter = DateTimeFormat
                    .forPattern("yyyy-MM-dd HH:mm:ss");
            DateTime dt = formatter.parseDateTime(source);
            return dt;
        }
  
        @Override
        public String convertFrom(DateTime source, String destination) {// (4)
            if (source == null) {
                return null;
            }
            return source.toString("yyyy-MM-dd HH:mm:ss");
        }
        
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Inherit \ ``org.dozer.DozerConverter``\.
   * - | (2)
     - | Set 2 target classes in the constructor.
   * - | (3)
     - | Describe the logic to convert from \ ``String``\  to \ ``DateTime``\. In this example, Locale is used by default.
   * - | (4)
     - | Describe the logic to convert from \ ``DateTime``\  to \ ``String``\. In this example, Locale is used by default.

The created custom converter should be defined for mapping.

dozer-configration-mapping.xml

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <mappings xmlns="http://dozer.sourceforge.net" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://dozer.sourceforge.net
              http://dozer.sourceforge.net/schema/beanmapping.xsd">
    
        <configuration>
            <custom-converters><!-- (1) -->
                <!-- these are always bi-directional -->
                <converter
                    type="com.example.yourproject.common.bean.converter.StringToJodaDateTimeConverter"><!-- (2) -->
                    <class-a>java.lang.String</class-a><!-- (3) -->
                    <class-b>org.joda.time.DateTime</class-b><!-- (4) -->
                </converter>
            </custom-converters>
        </configuration>
        <!-- omitted -->
    </mappings>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Define \ ``custom-converters``\  to which all the custom converters belong.
   * - | (2)
     - | Define converter for performing individual conversion. Specify fully qualified class name (FQCN) of implementation class in the converter type.
   * - | (3)
     - | Fully qualified class name (FQCN) of conversion source Bean
   * - | (4)
     - | Fully qualified class name (FQCN) of conversion destination Bean

When \ ``java.lang.String`` <=> ``org.joda.time.DateTime``\  conversion needs to be performed in the entire application by using the mapping given above, the mapping is performed by calling custom converter instead of standard mapping.

Example :

Bean definition of conversion source

.. code-block:: java

    public class Source {
        private int id;
        private String date;
        // omitted setter/getter
    }


Bean definition of conversion destination

.. code-block:: java

    public class Destination {
        private int id;
        private DateTime date;
        // omitted setter/getter
    }


Mapping (two-way)

.. code-block:: java

    Source source = new Source();
    source.setId(1);
    source.setDate("2012-08-10 23:12:12");

    DateTimeFormatter formatter = DateTimeFormat.forPattern("yyyy-MM-dd HH:mm:ss");
    DateTime dt = formatter.parseDateTime(source.getDate());

    // Source to Destination Bean Mapping (String to org.joda.time.DateTime)
    Destination destination = dozerBeanMapper.map(source, Destination.class);
    assertThat(destination.getId(), is(1));
    assertThat(destination.getDate(),is(dt));

    // Destination to Source Bean Mapping (org.joda.time.DateTime to String)
    dozerBeanMapper.map(destination, source);

    assertThat(source.getId(), is(1));
    assertThat(source.getDate(),is("2012-08-10 23:12:12"));

Refer to `Dozer Official Manual -Custom Converters- <http://dozer.sourceforge.net/documentation/customconverter.html>`_ for the details of custom converter.


.. note::

    The conversion from ``String``\  to the standard date/time object such as \ ``java.utl.Date``\  is explained in ":ref:`beanconverter-string-and-datetime`".

Appendix
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The options that can be specified in the mapping definition XML file are explained.

All options can be confirmed in `Dozer Official Manual -Custom Mappings Via Dozer XML Files- <http://dozer.sourceforge.net/documentation/mappings.html>`_.

.. _fieldexclude:

Field exclusion settings (field-exclude)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The fields that are not to be copied can be excluded at the time of Bean conversion.

Bean conversion is as given below.

Bean definition of conversion source

.. code-block:: java

    public class Source {
        private int id;
        private String name;
        private String title;
        // omitted setter/getter
    }


Bean definition of copy destination

.. code-block:: java

    public class Destination {
        private int id;
        private String name;
        private String title;
        // omitted setter/getter
    }


Define as follows to exclude any field of copy source Bean from mapping.

Carry out the settings of field exclusion in the mapping definition XML file as given below.

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <mappings xmlns="http://dozer.sourceforge.net" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://dozer.sourceforge.net
              http://dozer.sourceforge.net/schema/beanmapping.xsd">
        <!-- omitted -->
        <mapping>
            <class-a>com.xx.xx.Source</class-a>
            <class-b>com.xx.xx.Destination</class-b>
            <field-exclude><!-- (1) -->
                <a>title</a>
                <b>title</b>
            </field-exclude>
        </mapping>
        <!-- omitted -->
    </mappings>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Set the field you want to exclude in the <field-exclude> element. In this example, if map method is executed after specification, the title value of destination is not overwritten while copying Destination object from Source object.


.. code-block:: java

    Source source = new Source();
    source.setId(1);
    source.setName("SourceName");
    source.setTitle("SourceTitle");

    Destination destination = new Destination();
    destination.setId(2);
    destination.setName("DestinationName");
    destination.setTitle("DestinationTitle");
    beanMapper.map(source, destination);
    System.out.println(destination.getId());
    System.out.println(destination.getName());
    System.out.println(destination.getTitle());


The output of the above code is as given below.

.. code-block:: console

    1
    SourceName
    DestinationTitle


The title value of destination after mapping is same as in the previous state.

|

Specifying mapping (map-id)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
The mapping indicated in :ref:`fieldexclude`\  is applied while performing Bean conversion in the entire application.
To specify the applicable range of mapping, define map-id as given below.

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <mappings xmlns="http://dozer.sourceforge.net" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://dozer.sourceforge.net
              http://dozer.sourceforge.net/schema/beanmapping.xsd">
        <!-- omitted -->
        <mapping map-id="mapidTitleFieldExclude">
            <class-a>com.xx.xx.Source</class-a>
            <class-b>com.xx.xx.Destination</class-b>
            <field-exclude>
                <a>title</a>
                <b>title</b>
            </field-exclude>
        </mapping>
        <!-- omitted -->
    </mappings>


When the above settings are carried out, title can be excluded from copying by passing map-id (mapidTitleFieldExclude) to \ ``map``\  method.
When map-id is not specified, all the fields are copied without applying the settings.


The example of passing map-id to ``map`` method is shown below.


.. code-block:: java

    Source source = new Source();
    source.setId(1);
    source.setName("SourceName");
    source.setTitle("SourceTitle");

    Destination destination1 = new Destination();
    destination1.setId(2);
    destination1.setName("DestinationName");
    destination1.setTitle("DestinationTitle");
    beanMapper.map(source, destination1); // (1)
    System.out.println(destination1.getId());
    System.out.println(destination1.getName());
    System.out.println(destination1.getTitle());

    Destination destination2 = new Destination();
    destination2.setId(2);
    destination2.setName("DestinationName");
    destination2.setTitle("DestinationTitle");
    beanMapper.map(source, destination2, "mapidTitleFieldExclude"); // (2)
    System.out.println(destination2.getId());
    System.out.println(destination2.getName());
    System.out.println(destination2.getTitle());

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Normal mapping.
   * - | (2)
     - | Pass map-id as the third argument and apply the specific mapping rules.


The output of the above code is as given below.

.. code-block:: console

    1
    SourceName
    SourceTitle

    1
    SourceName
    DestinationTitle

|

.. tip::

   map-id can be specified not only by mapping items but also by defining the field.
   Refer to `Dozer Official Manual -Context Based Mapping- <http://dozer.sourceforge.net/documentation/contextmapping.html>`_ for details.

|

.. note::

   The same form object can be used for both creating new/updating the existing Web applications.
   Form object is copied (mapped) to domain object, however a field may exist where the object is not to be copied depending on the operations.
   In this case, use \ ``<field-exclude>``\.

   * Example: userId is included in the New form whereas it is not included in the Update form.
   
   In this case, null is set in the userId at the time of update if the same form object is used. If form object is copied as it is by
   fetching copy destination object from DB, userId of copy destination becomes null. In order to avoid this,
   provide map-id for update and carry out the settings to exclude field of userId at the time of update.

|

Settings to exclude null/empty field of copy source (map-null, map-empty)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\ ``null``\  or empty field of copy source Bean can be excluded from mapping.
Set in the mapping definition XML file as given below.

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <mappings xmlns="http://dozer.sourceforge.net" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://dozer.sourceforge.net
              http://dozer.sourceforge.net/schema/beanmapping.xsd">
        <!-- omitted -->
        <mapping map-null="false" map-empty-string="false"><!-- (1) -->
            <class-a>com.xx.xx.Source</class-a>
            <class-b>com.xx.xx.Destination</class-b>
        </mapping>
        <!-- omitted -->
    </mappings>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | To exclude the \ ``null``\  field of copy source Bean from mapping, set \ ``false``\  in the \ ``map-null``\  attribute. Default value is \ ``true``\ .
       | To exclude the empty field of copy source Bean from mapping, set \ ``false``\  in the \ ``map-empty-string``\  attribute. Default value is \ ``true``\ .


Bean definition of conversion source

.. code-block:: java

    public class Source {
        private int id;
        private String name;
        private String title;
        // omitted setter/getter
    }


Bean definition of conversion destination

.. code-block:: java

    public class Destination {
        private int id;
        private String name;
        private String title;
        // omitted setter/getter
    }


Mapping example

.. code-block:: java

    Source source = new Source();
    source.setId(1);
    source.setName(null);
    source.setTitle("");

    Destination destination = new Destination();
    destination.setId(2);
    destination.setName("DestinationName");
    destination.setTitle("DestinationTitle");
    beanMapper.map(source, destination);
    System.out.println(destination.getId());
    System.out.println(destination.getName());
    System.out.println(destination.getTitle());


The output of the above code is as given below.

.. code-block:: console

    1
    DestinationName
    DestinationTitle


\ ``name``\  and \ ``title``\  field of copy source Bean are \ ``null``\  or empty, and are excluded from mapping.


.. _beanconverter-string-and-datetime:

Mapping from string to date/time object
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The copy source string field can be mapped to the copy destination date/time field.

Following 6 types of conversions are supported.

Date/time

* ``java.lang.String`` <=> ``java.util.Date``
* ``java.lang.String`` <=> ``java.util.Calendar``
* ``java.lang.String`` <=> ``java.util.GregorianCalendar``
* ``java.lang.String`` <=> ``java.sql.Timestamp``

Date only

* ``java.lang.String`` <=> ``java.sql.Date``

Time only

* ``java.lang.String`` <=> ``java.sql.Time``


| Perform date/time conversion as follows.
| For example, the conversion to \ ``java.util.Date``\  is explained.
| Conversions for ``java.util.Calendar``\ ,\ ``java.util.GregorianCalendar``\ ,\ ``java.sql.Timestamp``\  can also be performed by the same method.

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <mappings xmlns="http://dozer.sourceforge.net" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://dozer.sourceforge.net
              http://dozer.sourceforge.net/schema/beanmapping.xsd">
        <!-- omitted -->
        <mapping>
            <class-a>com.xx.xx.Source</class-a>
            <class-b>com.xx.xx.Destination</class-b>
            <field>
                <a date-format="yyyy-MM-dd HH:mm:ss:SS">date</a><!-- (1) -->
                <b>date</b>
            </field>
        </mapping>
        <!-- omitted -->
    </mappings>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify field name and date format of copy source.


Bean definition of conversion source

.. code-block:: java

    public class Source {
        private String date;
        // omitted setter/getter
    }


Bean definition of conversion destination

.. code-block:: java

    public class Destination {
        private Date date;
        // omitted setter/getter
    }

Mapping

.. code-block:: java

    Source source = new Source();
    source.setDate("2013-10-10 11:11:11.111");
    Destination destination = beanMapper.map(source, Destination.class);
    assert(destination.getDate().equals(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.SSS").parse("2013-10-10 11:11:11.111")));


| There are many cases where it is preferable to set a common date format in the project rather than setting an individual date for each mapping definition.
| In such a case, it is recommended to set in the Global configuration file of Dozer.
| The date format set in the mapping of the entire application is applied.

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <mappings xmlns="http://dozer.sourceforge.net" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://dozer.sourceforge.net
              http://dozer.sourceforge.net/schema/beanmapping.xsd">
        <!-- omitted -->
        <configuration>
            <date-format>yyyy-MM-dd HH:mm:ss.SSS</date-format>
            <!-- omitted other configuration -->
        </configuration>
        <!-- omitted -->
    </mappings>


| There are no restrictions for the file name however, src/main/resources/META-INF/dozer/dozer-configration-mapping.xml is recommended.
| Global Configuration that impacts the entire application in this configuration file can be used within the scope of settings for dozer-configration-mapping.xml.

Refer to the `Dozer Official Manual -Global Configuration- <http://dozer.sourceforge.net/documentation/globalConfiguration.html>`_ for the details of items that can be configured.


Mapping error
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
If mapping process fails during mapping, \ ``org.dozer.MappingException``\  (runtime exception) is thrown.

The typical examples of \ ``MappingException``\  being thrown are given below.

* map-id that does not exist in the \ ``map``\  method is passed.
* map-id existing in \ ``map``\  method is passed however, the source/target type passed to map process differs from the definition specified in map-id.
* When conversion is not supported by Dozer and custom converter for conversion does not exist as well.

Since these are normal program bugs, the sections called by \ ``map``\  method should be modified appropriately.

.. raw:: latex

   \newpage
