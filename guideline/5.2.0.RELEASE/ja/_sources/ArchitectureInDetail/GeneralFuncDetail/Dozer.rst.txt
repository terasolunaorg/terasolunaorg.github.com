Beanマッピング(Dozer)
--------------------------------------------------------------------------------

.. only:: html

 .. contents:: 目次
    :depth: 4
    :local:

|

Overview
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Beanマッピングは、一つのBeanを他のBeanにフィールド値をコピーすることである。

| アプリケーションの異なるレイヤ間(アプリケーション層とドメイン層)で、データの受け渡しをする場合など、Beanマッピングが必要となるケースは多い。

| 例として、アプリケーション層の\ ``AccountForm``\ オブジェクトを、ドメイン層の\ ``Account``\ オブジェクトに変換する場合を考える。
| ドメイン層は、アプリケーション層に依存してはならないため、AccountFormオブジェクトをそのままドメイン層で使用できない。
| そこで、\ ``AccountForm``\ オブジェクトを、\ ``Account``\ オブジェクトにBeanマッピングし、ドメイン層では、\ ``Account``\ オブジェクトを使用する。
| これによって、アプリケーション層と、ドメイン層の依存関係を一方向に保つことができる。

.. figure:: ./images/beanmapping-overview.png
   :width: 80%

| このオブジェクト間のマッピングは、Beanのgetter/setterを呼び出して、データの受け渡しを行うことで実現できる。
| しかしながら、処理が煩雑になり、プログラムの見通しが悪くなるため、本ガイドラインでは、BeanマッピングライブラリであるOSSで利用可能な `Dozer <http://dozer.sourceforge.net>`_ を使用することを推奨する。

| Dozerを使用することで下図のように、コピー元クラスとコピー先クラスで型が異なるコピーや、ネストしたBean同士のコピーも容易に行うことができる。

.. figure:: ./images/dozer-functionality-overview.png
   :width: 75%

Dozerをした場合と使用しない場合のコード例を挙げる。

* 煩雑になり、プログラムの見通しが悪くなる例

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

* Dozerを使用した場合の例

    .. code-block:: java

        User user = userService.findById(userId);

        XxxOutput output = beanMapper.map(user, XxxOutput.class);


以降は、Dozerの利用方法について説明する。

 .. warning::

    JSR-310 Date and Time APIで追加されたクラス群はデフォルトのままでは例外が発生するため使用できず、カスタムコンバーターを作成する必要がある。

|

How to use
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Dozerは、Java Beanのマッピング機能ライブラリである。
変換元のBeanから変換先のBeanに、再帰的（ネストした構造）に、値をコピーする。

.. _bean-mapper-definition:

Dozerを使用するためのBean定義
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Dozerは、単独で使用するとき、以下のように、 ``org.dozer.Mapper`` のインスタンスを作成する。

.. code-block:: java

    Mapper mapper = new DozerBeanMapper();


Mapper のインスタンスを毎回作成するのは、効率が悪いため、
Dozerが提供している ``org.dozer.spring.DozerBeanMapperFactoryBean`` を使用すること。


Bean定義ファイル(applicationContext.xml)に、Mapperを作成するFactoryクラスである\ ``org.dozer.spring.DozerBeanMapperFactoryBean``\ を定義する

.. code-block:: xml

    <bean class="org.dozer.spring.DozerBeanMapperFactoryBean">
        <property name="mappingFiles"
            value="classpath*:/META-INF/dozer/**/*-mapping.xml" /><!-- (1) -->
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | mappingFilesに、マッピング定義XMLファイルを指定する。
       | ``org.dozer.spring.DozerBeanMapperFactoryBean`` は、 interfaceとして ``org.dozer.Mapper`` を保持している。そのため、 ``@Inject`` 時は ``Mapper`` を指定する。
       | この例では、クラスパス直下の、/META-INF/dozerの任意フォルダ内の
       | (任意の値)-mapping.xmlを、すべて読み込む。このXMLファイルの内容については、以降で説明する。

|


Beanマッピングを行いたいクラスに、 ``Mapper`` をインジェクトすればよい。

.. code-block:: java

    @Inject
    Mapper beanMapper;


.. _beanconverter-basic-mapping-label:

Bean間のフィールド名、型が同じ場合のマッピング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

デフォルトの動作として、Dozerは対象のBean間のフィールド名が同じであれば、マッピング定義XMLファイルを作成せずにマッピングできる。

変換元のBean定義

.. code-block:: java

    public class Source {
        private int id;
        private String name;
        // omitted setter/getter
    }


変換先のBean定義

.. code-block:: java

    public class Destination {
        private int id;
        private String name;
        // omitted setter/getter
    }


以下のように、 ``Mapper`` の ``map`` メソッドを使ってBeanマッピングを行う。
下記メソッドを実行した後、Destinationオブジェクトが新たに作成され、sourceの各フィールドの値が作成されたDestinationオブジェクトにコピーされる。

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

   * - 項番
     - 説明
   * - | (1)
     - | 第一引数に、コピー元のオブジェクトを渡し、第二引数に、コピー先のBeanのクラスを渡す。


上記のコードを実行すると以下のように出力される。作成されたオブジェクトにコピー元のオブジェクトの値が設定されていることが分かる。

.. code-block:: console

    1
    SourceName


既に存在している\ ``destination``\ オブジェクトに、\ ``source``\ オブジェクトのフィールドをコピーしたい場合は、

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

   * - 項番
     - 説明
   * - | (1)
     - | 第一引数に、コピー元のオブジェクトを渡し、第二引数に、コピー先のオブジェクトを渡す。


上記のコードを実行すると以下のように出力される。コピー元のオブジェクトの値がコピー先に反映されていることが分かる。

.. code-block:: console

    1
    SourceName

.. note::

    \ ``Destination``\ クラスのフィールドで\ ``Source``\ クラスに存在しないものは、コピー前後で値は変わらない。
    
    変換元のBean定義
    
        .. code-block:: java
        
            public class Source {
                private int id;
                private String name;
                // omitted setter/getter
            }
    
    
    変換先のBean定義
    
        .. code-block:: java
        
            public class Destination {
                private int id;
                private String name;
                private String title;
                // omitted setter/getter
            }
    
    マッピング例
    
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
    
    
    上記のコードを実行すると以下のように出力される。\ ``Source``\ クラスには\ ``title``\ フィールドが
    ないため、\ ``Destination``\ オブジェクトの\ ``title``\ フィールドは、コピー前のフィールド値から変更がない。
    
        .. code-block:: console
        
            1
            SourceName
            DestinationTitle

.. _beanconverter-difference-type-mapping-label:

Bean間のフィールド名は同じ、型が異なる場合のマッピング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

コピー元と、コピー先でBeanのフィールドの型が異なる場合、
型変換がサポートされている型は、自動でマッピングできる。

以下のような変換は、マッピング定義XMLファイル無しで変換できる。

例 : String -> BigDecimal

変換元のBean定義

.. code-block:: java

    public class Source {
        private String amount;
        // omitted setter/getter
    }


変換先のBean定義

.. code-block:: java

    public class Destination {
        private BigDecimal amount;
        // omitted setter/getter
    }


マッピング例

.. code-block:: java

    Source source = new Source();
    source.setAmount("123.45");
    Destination destination = beanMapper.map(source, Destination.class);
    System.out.println(destination.getAmount());


上記のコードを実行すると以下のように出力される。型が異なる場合でも値をコピーできていることが分かる。

.. code-block:: console

    123.45

サポートされている型変換については、 `マニュアル <http://dozer.sourceforge.net/documentation/simpleproperty.html>`_ を参照されたい。


.. _beanconverter-difference-item-xml-mapping-label:

Bean間のフィールド名が異なる場合のマッピング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

コピー元と、コピー先でフィールド名が異なる場合、マッピング定義XMLファイルを作成し、
Beanマッピングするフィールドを定義することで変換できる。

変換元のBean定義

.. code-block:: java

    public class Source {
        private int id;
        private String name;
        // omitted setter/getter
    }


変換先のBean定義

.. code-block:: java

    public class Destination {
        private int destinationId;
        private String destinationName;
        // omitted setter/getter
    }


:ref:`bean-mapper-definition`\ の定義がある場合、
src/main/resources/META-INF/dozerフォルダ内に、(任意の値)-mapping.xmlという、マッピング定義XMLファイルを作成する。

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

   * - 項番
     - 説明
   * - | (1)
     - | \ ``<class-a>``\ タグ内にコピー元のBeanの、完全修飾クラス名(FQCN)を指定する。
   * - | (2)
     - | \ ``<class-b>``\ タグ内にコピー先のBeanの、完全修飾クラス名(FQCN)を指定する。
   * - | (3)
     - | \ ``<field>``\ タグ内の\ ``<a>``\ タグ内にコピー元のBeanの、マッピング用のフィールド名を指定する。
   * - | (4)
     - | \ ``<field>``\ タグ内の\ ``<b>``\ タグ内に(3)に対応するコピー先のBeanの、マッピング用のフィールド名を指定する。

マッピング例

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

   * - 項番
     - 説明
   * - | (1)
     - | 第一引数に、コピー元のオブジェクトを渡し、第二引数に、コピー先のBeanのクラスを渡す。(基本マッピングと違いはない。)


上記のコードを実行すると以下のように出力される。

.. code-block:: console

    1
    SourceName


:ref:`bean-mapper-definition`\ の設定によって、\ ``mappingFiles``\ プロパティにクラスパス直下のMETA-INF/dozer配下に存在するマッピング定義XMLファイルが読み込まれる。
ファイル名は(任意の値)-mapping.xmlである必要がある。
いずれかのファイル内に\ ``Source``\ クラスと\ ``Destination``\ クラス間におけるマッピング定義があれば、その設定が適用される。


.. note::
    マッピング定義XMLファイルは、Controller単位で作成し、ファイル名は、(Controller名からControllerを除いた値)-mapping.xmlにすることを推奨する。
    例えば、TodoControllerに対するマッピング定義XMLファイルは、src/main/resources/META-INF/dozer/todo-mapping.xmlに作成する。

|

単方向・双方向マッピング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. _beanconverter-one-two-way-mapping-label:

マッピングXMLで定義されているマッピングは、デフォルトで、双方向マッピングである。
すなわち前述の例では\ ``Source``\ オブジェクトから\ ``Destination``\ オブジェクトへのマッピングを行ったが、
\ ``Destination``\ オブジェクトから\ ``Source``\ オブジェクトのマッピングも可能である。

単方向をのみを指定したい場合は、マッピング・フィールド定義に、\ ``<mapping>``\ タグの\ ``type``\ 属性に\ ``"one-way"``\ を設定する。

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


変換元のBean定義

.. code-block:: java

    public class Source {
        private int id;
        private String name;
        // omitted setter/getter
    }


変換先のBean定義

.. code-block:: java

    public class Destination {
        private int destinationId;
        private String destinationName;
        // omitted setter/getter
    }

マッピング例

.. code-block:: java

    Source source = new Source();
    source.setId(1);
    source.setName("SourceName");
    Destination destination = beanMapper.map(source, Destination.class);
    System.out.println(destination.getDestinationId());
    System.out.println(destination.getDestinationName());


上記のコードを実行すると以下のように出力される。

.. code-block:: console

    1
    SourceName


単方向を指定している場合に、逆方向のマッピングを行ってもエラーは発生しない。コピー処理は無視される。
なぜなら、マッピング定義がないと\ ``Destination``\ のフィールドに該当する\ ``Source``\ のフィールドが存在ないとみなされるためである。

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

上記のコードを実行すると以下のように出力される。

.. code-block:: console

    1
    SourceName

.. _beanconverter-custom-converter-label:


Nestしたフィールドのマッピング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
コピー元Beanが持つフィールドを、コピー先Beanが持つNestしたBeanのフィールドにも、マッピングできることである。
(Dozerの用語で、 `Deep Mapping <http://dozer.sourceforge.net/documentation/deepmapping.html>`_ と呼ばれる。)


変換元のBean定義

.. code-block:: java

    public class EmployeeForm {
        private int id;
        private String name;
        private String deptId;
        // omitted setter/getter
    }


変換先のBean定義

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

例 : \ ``EmployeeForm``\ オブジェクトが持つ\ ``deptId``\ を、\ ``Employee``\ オブジェクトが持つ\ ``Department``\ の\ ``deptId``\ にマップしたい場合、
以下のように定義する。

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

   * - 項番
     - 説明
   * - | (1)
     - | \ ``Employee``\ フォームの\ ``deptId``\ に対する、\ ``Employee``\ オブジェクトのフィールドを指定する。


マッピング例

.. code-block:: java

    EmployeeForm source = new EmployeeForm();
    source.setId(1);
    source.setName("John");
    source.setDeptId("D01");

    Employee destination = beanMapper.map(source, Employee.class);
    System.out.println(destination.getId());
    System.out.println(destination.getName());
    System.out.println(destination.getDepartment().getDeptId());


上記のコードを実行すると以下のように出力される。

.. code-block:: console

    1
    John
    D01

上記の場合は、変換先クラスである\ ``Employee``\ の新規インスタンスが作成される。
\ ``Employee``\ の中の\ ``department`` フィールドにも、新規に作成された\ ``Department``\ インスタンスが設定され、
\ ``EmployeeForm``\ の\ ``deptId``\ が、コピーされる。

下記ように\ ``Employee``\ の中の\ ``department`` フィールドに既に\ ``Department``\ オブジェクトが設定されている場合は、
新規インスタンスは作成されず、既存の\ ``Department``\ オブジェクトの\ ``deptId``\ フィールドに、
\ ``EmployeeForm``\ の\ ``deptId``\ がコピーされる。

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


上記のコードを実行すると以下のように出力される。

.. code-block:: console

    D01
    true

|

Collectionマッピング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Dozerは、以下のCollectionタイプの双方向自動マッピングをサポートしている。
フィールド名が同じである場合、マッピング定義XMLファイルが不要である。

* ``java.util.List`` <=> ``java.util.List``
* ``java.util.List`` <=> Array
* Array <=> Array
* ``java.util.Set`` <=> ``java.util.Set``
* ``java.util.Set`` <=> Array
* ``java.util.Set`` <=> ``java.util.List``



次のクラスのコレクションをもつBeanのマッピングについて考える。

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


変換元のBean

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

変換先のBean

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


マッピング例


.. code-block:: java

    AccountForm accountForm = new AccountForm();

    List<Email> emailsSrc = new ArrayList<Email>();

    emailsSrc.add(new Email("a@example.com"));
    emailsSrc.add(new Email("b@example.com"));
    emailsSrc.add(new Email("c@example.com"));

    accountForm.setEmails(emailsSrc);

    Account account = beanMapper.map(accountForm, Account.class);

    System.out.println(account.getEmails());


上記のコードを実行すると以下のように出力される。

.. code-block:: console

    [a@example.com, b@example.com, c@example.com]

ここまではこれまで説明したことと特に変わりはない。

次の例のように、\ **コピー先のBeanのCollectionフィールドに既に要素が追加されている場合は要注意である。**\ 

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


上記のコードを実行すると以下のように出力される。

.. code-block:: console

    [a@example.com, d@example.com, e@example.com, a@example.com, b@example.com, c@example.com]

コピー元BeanのCollectionの全要素が、コピー先BeanのCollectionに追加されている。
\ ``a@exmample.com``\ をもつ2つの\ ``Email``\ オブジェクトは"等価"であるが、単純に追加される。

(ここでいう"等価"とは\ ``Email.equals`` で比較すると\ ``true``\ になり、\ ``Email.hashCode``\ の値も同じであることを意味する。)

上記の振る舞いは、Dozerの用語では\ **cumulative**\ と呼ばれ、Collectionをマッピングする際のデフォルトの挙動となっている。

この挙動はマッピング定義XMLファイルにおいて変更することができる。

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

   * - 項番
     - 説明
   * - | (1)
     - | \ ``<field>``\ タグの\ ``relationship-type``\ 属性に\ ``non-cumulative``\ を指定する。デフォルト値は\ ``cumulative``\ である。
       | 
       | マッピング対象のBeanの全フィールドに対して\ ``non-cumulative``\ を指定したい場合は、\ ``<mapping>``\ タグの\ ``relationship-type``\ 属性に\ ``non-cumulative``\ を指定することもできる。

この設定のもと、前述のコードを実行すると以下のように出力される。

.. code-block:: console

    [a@example.com, d@example.com, e@example.com, b@example.com, c@example.com]

等価であるオブジェクトの重複がなくなっていることが分かる。

.. note::

    変換元のオブジェクトが、変換先のオブジェクトで更新されることに注意されたい。
    上記の例では\ ``AccountForm``\ の中の\ ``a@exmample.com``\ がコピー先に格納される。
    
    
        .. figure:: ./images/dozer_noncumulativeupdate.png
           :alt: noncumulative update using dozer
           :width: 60%

コピー先のコレクションにのみに存在する項目は除外したい場合も、マッピング定義XMLファイルの設定で実現することができる。

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

   * - 項番
     - 説明
   * - | (1)
     - | \ ``<field>``\ タグの\ ``remove-orphans``\ 属性に\ ``true``\ を設定する。デフォルト値は\ ``false``\ である。


この設定のもと、前述のコードを実行すると以下のように出力される。

.. code-block:: console

    [a@example.com, b@example.com, c@example.com]

コピー元にあるオブジェクトだけがコピー先のコレクション内に残っていることが分かる。

いかのように設定しても同じ結果が得られる。

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

   * - 項番
     - 説明
   * - | (1)
     - | \ ``<field>``\ タグの\ ``copy-by-reference``\ 属性に\ ``true``\ を設定する。デフォルト値は\ ``false``\ である。

これまでの挙動を図で表現する。

* デフォルトの挙動(cumulative)

    .. figure:: ./images/dozer-collection-cumulative.png
       :width: 60%

* non-cumulative

    .. figure:: ./images/dozer-collection-non-cumulative.png
       :width: 60%

* non-cumulativeかつremove-orphans=true

    .. figure:: ./images/dozer-collection-non-cumulative-and-orphan-remove.png
       :width: 60%

    copy-by-referenceもこのパターンである。

.. note::

  「non-cumulativeかつremove-orphans=true」のパターンと「copy-by-reference」のパターンの違いは、Bean変換後のCollectionのコンテナがコピー先のものか、コピー元のものかで異なる点である。
  
  「non-cumulativeかつremove-orphans=true」のパターンの場合は、Bean変換後のCollectionのコンテナはコピー先のものであり、「copy-by-reference」のパターンはコピー元のものである。
  以下に図で説明する。

  * non-cumulativeかつremove-orphans=true

    .. figure:: ./images/dozer-differrence1.png
       :width: 50%

  * copy-by-reference
    
    .. figure:: ./images/dozer-differrence2.png
       :width: 50%
  
  \ **コピー先がJPA (Hibernate)のエンティティで1対多や多対多の関連を持つ場合は要注意である**\ 。コピー先のエンティティがEntityManagerの管理下にある場合、予期せぬトラブルに遭うことがある。
  例えばコレクションのコンテナが変更されると全件DELETE + 全件INSERTのSQLが発行され、「non-cumulativeかつremove-orphans=true」でコピーした場合は変更内容をUPDATE(要素数が異なる場合はDELETE or INSERT)のSQLが発行される場合がある。
  どちらが良いかは要件次第である。


.. warning::

    マッピング対象のBeanが\ ``String``\ のコレクションを持つ場合、\ `期待通りの挙動にならないバグ <http://sourceforge.net/p/dozer/bugs/382/>`_\ がある。
    
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
    
    上記のコードをnon-cumulativeかつremove-orphans=trueの設定で実行すると、
    
        .. code-block:: console
    
            [a, b, c]

    と出力されることを期待するが、実際には
    
        .. code-block:: console
    
            [b, c]

    と出力され、\ **重複したStringが除かれてしまう**\ 。
    
    copy-by-reference="true"の設定で実行すると、期待通り
    
        .. code-block:: console
    
            [a, b, c]

    と出力される。

.. tip::

   Dozerでは、Genericsを使用しないリスト間でもマッピングできる。このとき、変換元と変換先に含まれているオブジェクトのデータ型をHINTとして指定できる。
   詳細は、 `Dozerの公式マニュアル -Collection and Array Mapping(Using Hints for Collection Mapping)- <http://dozer.sourceforge.net/documentation/collectionandarraymapping.html#Using_Hints_for_Collection_Mapping>`_ を参照されたい。

.. todo::

    Collection<T>を使用したBean間のマッピングは失敗することが確認されている。

    例 :

        .. code-block:: java
        
            public class ListNestedBean<T> {
               private List<T> nest;
               // omitted other declarations
            }

     実行結果 :
     
        .. code-block:: console
        
            java.lang.ClassCastException: sun.reflect.generics.reflectiveObjects.TypeVariableImpl cannot be cast to java.lang.Class


How to extend
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

カスタムコンバーターの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Dozerがサポートしていないデータ型のマッピングの場合、カスタムコンバーター経由でマッピングできる。

* 例 : ``java.lang.String`` <=> ``org.joda.time.DateTime``

| カスタムコンバーターは、Dozerが提供している ``org.dozer.CustomConverter`` を実装したクラスである。
| カスタムコンバーターの指定は、以下3パターンで行える。

* Global Configuration
* クラスレベル
* フィールドレベル

アプリケーション全体で、同様のロジックにより変換を行いたい場合は、Global Configurationを推奨する。

カスタムコンバーターを実装する場合は\ ``org.dozer.DozerConverter``\ を継承するのが便利である。

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

   * - 項番
     - 説明
   * - | (1)
     - | \ ``org.dozer.DozerConverter``\ を継承する。
   * - | (2)
     - | コンストラクタで対象の2つのクラスを設定する。
   * - | (3)
     - | \ ``String``\ から\ ``DateTime``\ の変換ロジックを記述する。本例ではデフォルトLocaleを使用する。
   * - | (4)
     - | \ ``DateTime``\ から\ ``String``\ の変換ロジックを記述する。本例ではデフォルトLocaleを使用する。

作成したカスタムコンバーターを、マッピングに利用するために定義する必要がある。

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

   * - 項番
     - 説明
   * - | (1)
     - | すべてのカスタムコンバーターが属する、\ ``custom-converters``\ を定義する。
   * - | (2)
     - | 個別の変換の行うconverterを定義する。converterのタイプに、実装クラスの完全修飾クラス名(FQCN)を指定する。
   * - | (3)
     - | 変換元Beanの完全修飾クラス名(FQCN)
   * - | (4)
     - | 変換先Beanの完全修飾クラス名(FQCN)

上記のマッピングを行ったことで、アプリケーション全体で、``java.lang.String`` <=> ``org.joda.time.DateTime``\ の変換が必要な場合、標準のマッピングではなく、カスタムコンバーター呼び出しでマッピングが行われる。

例 :

変換元のBean定義

.. code-block:: java

    public class Source {
        private int id;
        private String date;
        // omitted setter/getter
    }


変換先のBean定義

.. code-block:: java

    public class Destination {
        private int id;
        private DateTime date;
        // omitted setter/getter
    }


マッピング (双方向例)

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

カスタムコンバーターに関する詳細は、 `Dozerの公式マニュアル -Custom Converters- <http://dozer.sourceforge.net/documentation/customconverter.html>`_ を参照されたい。


.. note::

    ``String``\ から\ ``java.utl.Date``\ など標準の日付・時刻オブジェクトへの変換については":ref:`beanconverter-string-and-datetime`"で述べる。

Appendix
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

マッピング定義XMLファイルで指定できるオプションを説明する。

すべてのオプションは、 `Dozerの公式マニュアル -Custom Mappings Via Dozer XML Files- <http://dozer.sourceforge.net/documentation/mappings.html>`_ で確認できる。

.. _fieldexclude:

フィールド除外設定 (field-exclude)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Beanを変換する際に、コピーしてほしくないフィールドを除外することができる。

以下のようなBeanの変換を考える。

変換元のBean定義

.. code-block:: java

    public class Source {
        private int id;
        private String name;
        private String title;
        // omitted setter/getter
    }


コピー先のBean定義

.. code-block:: java

    public class Destination {
        private int id;
        private String name;
        private String title;
        // omitted setter/getter
    }


コピー元のBeanから任意のフィールドをマッピングから除外したい場合は以下のように定義する。

フィールド除外の設定は、マッピング定義XMLファイルで、以下のように行う。

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

   * - 項番
     - 説明
   * - | (1)
     - | 除外したいフィールドを、<field-exclude>要素で設定する。この例の場合、指定した上でmapメソッドを実行すると、SourceオブジェクトからDestinationオブジェクトをコピーする際に、destinationのtitleの値が、上書きされない。


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


上記のコードを実行すると以下のように出力される。

.. code-block:: console

    1
    SourceName
    DestinationTitle


マッピング後、destinationのtitleの値は、前の状態のままである。

|

マッピングの特定化 (map-id)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
:ref:`fieldexclude`\ で示したマッピングは、アプリケーション全体でBean変換する際に適用される。
マッピングの適用範囲を制限(特定化)したい場合は、以下のように、map-idを指定して定義する。

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


上記の設定を行うと、\ ``map``\ メソッドにmap-id(mapidTitleFieldExclude)を渡すことでtitleのコピーを除外できる。
map-idを指定しない場合はこの設定は適用されず、全フィールドがコピーされる。


``map`` メソッドにmap-idを渡す例を、以下に示す。


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

   * - 項番
     - 説明
   * - | (1)
     - | 通常のマッピング。
   * - | (2)
     - | 第三引数にmap-idを渡し、特定のマッピングルールを適用する。


上記のコードを実行すると以下のように出力される。

.. code-block:: console

    1
    SourceName
    SourceTitle

    1
    SourceName
    DestinationTitle

|

.. tip::

   map-idの指定は、mapping項目だけでなく、フィールドの定義でも行える。
   詳細は、 `Dozerの公式マニュアル -Context Based Mapping- <http://dozer.sourceforge.net/documentation/contextmapping.html>`_ を参照されたい。

|

.. note::

   Webアプリケーションにおいて、新規追加・更新両方の操作で同じフォームオブジェクトを使う場合がある。
   このとき、フォームオブジェクトをドメインオブジェクトにコピー(マップ)する上で、操作によってはコピーしたくないフィールドもある。
   この場合に、\ ``<field-exclude>``\ を使用する。

   * 例：新規作成のフォームではuserIdを含むが、更新用のフォームではuserIdを含まない。
   
   この場合に同じフォームオブジェクトを使用すると、更新時にuserIdにnullが設定される。コピー先のオブジェクトをDBから取得して、
   フォームオブジェクトをそのままコピーすると、コピー先のuserIdまでnullとなる。これを回避するために、
   更新用のmap-idを用意し、更新時はuserIdに対して、フィールド除外の設定を行う。

|

コピー元のnull・空フィールドを除外する設定 (map-null, map-empty)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
コピー元のBeanのフィールドが、\ ``null``\ の場合、あるいは空の場合に、マッピングから除外することができる。
以下のように、マッピング定義XMLファイルに設定する。

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

   * - 項番
     - 説明
   * - | (1)
     - | コピー元のBeanのフィールドが\ ``null``\ の場合にマッピングから除外したい場合は\ ``map-null``\ 属性に\ ``false``\ を設定する。デフォルト値は\ ``true``\ である。
       | 空の場合に、マッピングから除外したい場合は\ ``map-empty-string``\ 属性に\ ``false``\ を設定する。デフォルト値は\ ``true``\ である。


変換元のBean定義

.. code-block:: java

    public class Source {
        private int id;
        private String name;
        private String title;
        // omitted setter/getter
    }


変換先のBean定義

.. code-block:: java

    public class Destination {
        private int id;
        private String name;
        private String title;
        // omitted setter/getter
    }


マッピング例

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


上記のコードを実行すると以下のように出力される。

.. code-block:: console

    1
    DestinationName
    DestinationTitle


コピー元Beanの\ ``name``\ と\ ``title``\ フィールドは、\ ``null``\ 、あるいは空で、マッピングから除外されている。


.. _beanconverter-string-and-datetime:

文字列から日付・時刻オブジェクトへのマッピング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

コピー元の文字列型のフィールドを、コピー先の日付・時刻系のフィールドにマッピングできる。

以下6種類の変換をサポートしている。

日付・時刻系

* ``java.lang.String`` <=> ``java.util.Date``
* ``java.lang.String`` <=> ``java.util.Calendar``
* ``java.lang.String`` <=> ``java.util.GregorianCalendar``
* ``java.lang.String`` <=> ``java.sql.Timestamp``

日付のみ

* ``java.lang.String`` <=> ``java.sql.Date``

時刻のみ

* ``java.lang.String`` <=> ``java.sql.Time``


| 日付・時刻系の変換は、以下のように行う。
| 例として、\ ``java.util.Date``\ への変換を説明する。
| ``java.util.Calendar``\ ,\ ``java.util.GregorianCalendar``\ ,\ ``java.sql.Timestamp``\ も同じ方法で行える。

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

   * - 項番
     - 説明
   * - | (1)
     - | コピー元のフィールド名と日付形式を指定する。


変換元のBean定義

.. code-block:: java

    public class Source {
        private String date;
        // omitted setter/getter
    }


変換先のBean定義

.. code-block:: java

    public class Destination {
        private Date date;
        // omitted setter/getter
    }

マッピング

.. code-block:: java

    Source source = new Source();
    source.setDate("2013-10-10 11:11:11.111");
    Destination destination = beanMapper.map(source, Destination.class);
    assert(destination.getDate().equals(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.SSS").parse("2013-10-10 11:11:11.111")));


| 日付形式は、個別のマッピング定義毎に設定するよりも、プロジェクトで一括で設定したいケースが多い。
| その場合はDozerのGlobal configurationファイルで設定することを推奨する。
| その場合、アプリケーション全体のマッピングで設定された日付形式が、適用される。

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


| ファイル名には制限はないが、src/main/resources/META-INF/dozer/dozer-configration-mapping.xmlを推奨する。
| dozer-configration-mapping.xml内の設定の範囲は、この設定ファイル内でアプリケーション全体に影響を与える、Global Configurationを行えばよい。

設定可能な項目の詳細について、 `Dozerの公式マニュアル -Global Configuration- <http://dozer.sourceforge.net/documentation/globalConfiguration.html>`_ を参照されたい。


マッピングのエラー
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
マッピング中にマッピング処理が失敗したら、\ ``org.dozer.MappingException``\ (実行時例外)がスローされる。

\ ``MappingException`` がスローされる代表的な例を、以下に挙げる。

* \ ``map``\ メソッドに存在しないmap-idが渡されている。
* \ ``map``\ メソッドに存在するmap-idを渡したが、マップ処理に渡したソース・ターゲット型は、そのmap-idに指定している定義とは異なる。
* Dozerがサポートしていない変換の場合、かつ、その変換用のカスタムコンバーターも存在しない場合。

これらは通常プログラムバグであるので、\ ``map``\ メソッドの呼び出しの部分を正しく修正する必要がある。

.. raw:: latex

   \newpage

