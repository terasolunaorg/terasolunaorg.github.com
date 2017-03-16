プロパティ管理
===================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

Overview
--------------------------------------------------------------------------------

本節では、プロパティの管理方法について説明する。

プロパティとして管理が必要となる値は、以下の2つに分類することができる。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.35\linewidth}|p{0.35\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 20 35 35

    * - 項番
      - 分類
      - 説明
      - 例
    * - 1.
      - 環境依存設定値
      - アプリケーションが動作する環境に応じて指定する値を変える必要がある設定値。

        システム構成などの非機能要件に依存する。
      - * データベースの接続情報(接続URL、接続ユーザ、パスワードなど)
        * ファイルの保存先(ディレクトリのパスなど)
        * more ...
    * - 2.
      - アプリケーション設定値
      - アプリケーションの動作をカスタマイズできる様にするための設定値。

        アプリケーションの機能要件に依存する。
      - * パスワード有効日数
        * 予約期間日数
        * more ...

.. note::

    本ガイドラインでは、これらの設定値については、
    プロパティとして管理(プロパティファイルに定義)することを推奨している。

    これらの設定値をプロパティから取得する仕組みにしておくと、設定値を変更する際に、
    アプリケーション(warファイルやjarファイル)を再ビルドする必要がないため、
    テスト済みのアプリケーションをプロダクト環境にリリースする事が可能になる。

.. tip::

    プロパティとして管理している値は、JVMのシステムプロパティ(-Dオプション)やOSの環境変数から取得することができる。
    アクセス順番については、「:ref:`PropertiesManagementHowToUse`」を参照されたい。

|

プロパティとして管理されている値は、以下の2箇所で利用することができる。

* bean定義ファイル
* DIコンテナで管理するJavaクラス

|

.. _PropertiesManagementHowToUse:

How to use
--------------------------------------------------------------------------------

.. _technical-details_label:

プロパティファイル定義方法について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Bean定義ファイルに ``<context:property-placeholder/>`` タグを定義することで、JavaクラスやBean定義ファイル内でプロパティファイル中の値にアクセスできるようになる。
| ``<context:property-placeholder/>`` タグは、指定されたプロパティファイル群を読み込み、
| ``@Value`` アノテーションや、 Bean定義ファイル中で、\ ``${xxx}``\ 形式で指定されたプロパティファイルのキー \ ``xxx``\ に対する値を取得できる。

 .. note::

    \ ``${xxx:defaultValue}``\ 形式で指定すると、プロパティファイルにキー \ ``xxx``\ の設定が存在しない場合に\ ``defaultValue``\ を使用する。

|

以下に、プロパティファイルの定義方法について説明する。

**bean定義ファイル**

- applicationContext.xml
- spring-mvc.xml

 .. code-block:: xml

    <context:property-placeholder location="classpath*:META-INF/spring/*.properties"/>  <!-- (1) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | locationに設定する値は、リソースのロケーションパスを設定すること。
        | location属性には、カンマ区切りで複数のパスを指定することができる。
        | 上記設定により、クラスパス中のMETA-INF/springディレクトリ配下のpropertiesファイルを読み込む。
        | 一度設定すれば、あとはMETA-INF/spring以下にpropertiesファイルを追加するだけで良い。
        | locationの設定値の詳細は、\ `リファレンス <http://docs.spring.io/spring/docs/4.3.5.RELEASE/spring-framework-reference/html/resources.html>`_\ を参照されたい。

 .. note::

    \ ``<context:property-placeholder>``\ の定義は、 ``applicationContext.xml`` と ``spring-mvc.xml`` の両方に定義が必要である。

|

デフォルトでは、以下の順番でプロパティにアクセスする。

#. 実行中のJVMのシステムプロパティ
#. 環境変数
#. アプリケーション定義のプロパティファイル

| デフォルトでは、すべての環境関連のプロパティ(JVMのシステムプロパティと環境変数)を読み込んだ後に、アプリケーションに定義されたプロパティファイルが検索され、読み込まれる。
| 読み込み順番を変更するには、 ``<context:property-placeholder/>`` タグのlocal-override属性をtrueに設定する。
| このように設定することで、アプリケーションに定義されたプロパティが、優先的に有効になる。




**bean定義ファイル**

 .. code-block:: xml

   <context:property-placeholder
       location="classpath*:META-INF/spring/*.properties" 
       local-override="true" /> <!-- (1) -->

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | local-override属性をtrueに設定すると、以下の順番でプロパティにアクセスする。
       | 1. アプリケーション定義のプロパティ
       | 2. 実行中のJVMのシステムプロパティ
       | 3. 環境変数

|

 .. note::

        通常は上記の設定で十分である。
        複数の ``<context:property-placeholder/>`` タグを指定する場合、order属性の値を設定することで、読み込みの順位付けをすることができる。

            **bean定義ファイル**

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
               :class: longtable

               * - 項番
                 - 説明
               * - | (1)
                 - | order属性を(2)より低い値を設定することにより、(2)より先にlocation属性に該当するプロパティファイルが読み込まれる。
                   | (2)で読み込んだプロパティファイル内のキーと重複するキーが存在する場合、(1)で取得した値が優先される。
                   | ignore-unresolvable属性をtrueにすることで、(2)のプロパティファイルのみにキーが存在する場合にエラーが発生するのを防ぐ。
               * - | (2)
                 - | order属性を(1)より高い値を設定することにより、(1)の次にlocation属性に該当するプロパティファイルが読み込まれる。
                   | (1)で読み込んだプロパティファイル内のキーと重複するキーが存在する場合、(1)で取得した値が設定される。
                   | ignore-unresolvable属性をtrueにすることで、(1)のプロパティファイルのみにキーが存在する場合にエラーが発生するのを防ぐ。

|

.. _bean-definition-file_label:

bean定義ファイル内でプロパティを使用する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| データソースの設定ファイルを例に説明を行う。
| 以下の例では、プロパティファイル定義( ``<context:property-placeholder/>`` )が指定されている前提で行う。
| 基本的には、bean定義ファイルに、プロパティファイルのキーを ``${}`` プレースホルダで設定することで、プロパティ値を設定することができる。

**プロパティファイル**

 .. code-block:: properties

   database.url=jdbc:postgresql://localhost:5432/shopping
   database.password=postgres
   database.username=postgres
   database.driverClassName=org.postgresql.Driver

|

**bean定義ファイル**

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

   * - 項番
     - 説明
   * - | (1)
     - | ``${database.driverClassName}`` を設定することで、読み込まれたプロパティファイルのキー\ ``database.driverClassName``\ に対する値が代入される。
   * - | (2)
     - | ``${database.url}`` を設定することで、読み込まれたプロパティファイルのキー\ ``database.url``\ に対する値が代入される。
   * - | (3)
     - | ``${database.username}`` を設定することで、読み込まれたプロパティファイルのキー\ ``database.username``\ に対する値が代入される。
   * - | (4)
     - | ``${database.password}`` を設定することで、読み込まれたプロパティファイルのキー\ ``database.password``\ に対する値が代入される。

|

propertiesファイルのキーが読み込まれた結果、以下のように置換される。

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

Javaクラス内でプロパティを使用する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Javaクラスでプロパティを利用する場合、プロパティの値を格納したいフィールドに ``@Value`` アノテーションを指定することで実現できる。
| ``@Value`` アノテーションを使用するためには、そのオブジェクトはSpringのDIコンテナに管理されている必要がある。

| 以下の例では、プロパティファイル定義( ``<context:property-placeholder/>`` )が指定されている前提で行う。
| 基本的に、変数に ``@Value`` アノテーションを付与し、valueにpropetyファイルのキーを ``${}`` プレースホルダで設定することで外部参照することができる。

**プロパティファイル**

 .. code-block:: properties

   item.upload.title=list of update file
   item.upload.dir=file:/tmp/upload
   item.upload.maxUpdateFileNum=10

**Javaクラス**

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

   * - 項番
     - 説明
   * - | (1)
     - | ``@Value`` アノテーションのvalueに ``${item.upload.title}`` を設定することで、読み込まれたプロパティファイルのキー\ ``item.upload.title``\ に対する値が代入される。
       | ``uploadTitle`` にはStringクラスに"list of update file"が代入される。
   * - | (2)
     - | ``@Value`` アノテーションのvalueに ``${item.upload.dir}`` を設定することで、読み込まれたプロパティファイルのキー\ ``item.upload.dir``\ に対する値が代入される。
       | ``uploadDir`` には初期値"/tmp/upload"でオブジェクト生成された\ ``org.springframework.core.io.Resource``\ オブジェクトが格納される。
   * - | (3)
     - | ``@Value`` アノテーションのvalueに ``${item.upload.maxUpdateFileNum}`` を設定することで、読み込まれたプロパティファイルのキー\ ``item.upload.maxUpdateFileNum``\ に対する値が代入される。
       | ``maxUpdateFileNum`` には整数型に10が代入される。

 .. warning::

        Utilityクラスなどのstaticメソッドからプロパティ値を利用したい場合も考えられるが、Bean定義されないクラスでは \ ``@Value``\ アノテーションによるプロパティ値の取得は行えない。
        このような場合には、 ``@Component`` アノテーションを付けたHelperクラスを作成し、\ ``@Value``\ アノテーションでプロパティ値を取得することを推奨する。(当然、該当クラスはcomponent-scanの対象にする必要がある。)
        プロパティ値を利用したいクラスは、Utilityクラスにすべきでない。

|

How to extend
--------------------------------------------------------------------------------
プロパティ値の取得方法の拡張について説明する。プロパティ値の取得方法の拡張は
``org.springframework.context.support.PropertySourcesPlaceholderConfigurer`` クラスを拡張することで実現できる。

拡張例として、暗号化したプロパティファイルを使用するケースを挙げる。

|

暗号化したプロパティ値を復号して使用する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| セキュリティを強化するため、プロパティファイルを暗号化しておきたい場合がある。
| 例として、プロパティ値が暗号化されている場合に復号を行う実装を示す。(具体的な暗号化、復号方法は省略する。)

**Bean定義ファイル**

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

   * - 項番
     - 説明
   * - | (1)
     - | ``<context:property-placeholder/>``\ の代わりに拡張したPropertySourcesPlaceholderConfigurerを定義する。 ``<context:property-placeholder/>``\ タグを削除しておくこと。
   * - | (2)
     - | propertyタグのname属性に"locations"を設定し、value属性に読み込むプロパティファイルパスを指定する。
       | 読み込むプロパティファイルパスの指定方法は :ref:`technical-details_label` と同じ。

**Javaクラス**

- 拡張したPropertySourcesPlaceholderConfigurer

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

   * - 項番
     - 説明
   * - | (1)
     - | 拡張したPropertySourcesPlaceholderConfigurerは ``org.springframework.context.support.PropertySourcesPlaceholderConfigurer`` をextendする。
   * - | (2)
     - | ``org.springframework.context.support.PropertySourcesPlaceholderConfigurer`` クラスの ``doProcessProperties`` メソッドをoverrideする。
   * - | (3)
     - | 親クラスの ``doProcessProperties`` を呼び出すが、 ``valueResolver`` は独自実装したvalueResolver( ``EncryptedValueResolver`` )を使用する。
       | ``EncryptedValueResolver`` クラス内で、プロパティファイルの暗号化されたvalueを取得した場合に復号する。

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

   * - 項番
     - 説明
   * - | (1)
     - | 拡張した ``EncryptedValueResolver`` は、 ``org.springframework.util.StringValueResolver`` を実装する。
   * - | (2)
     - | コンストラクタで ``EncryptedValueResolver`` クラスを生成したときに、 ``EncryptedPropertySourcesPlaceholderConfigurer`` から引き継いできた ``StringValueResolver`` を設定する。
   * - | (3)
     - | ``org.springframework.util.StringValueResolver`` の ``resolveStringValue`` メソッドをovewrideする。
       | ``resolveStringValue`` メソッド内にて、プロパティファイルの暗号化されたvalueを取得した場合に復号する。
       | 以降、(5)～(6)は一例の処理になるため、実装によって処理が異なる。
   * - | (4)
     - | コンストラクタで設定した ``StringValueResolver`` の ``resolveStringValue`` メソッドの引数にキーを指定して値を取得している。この値は実際にプロパティファイルに定義されている値である。
   * - | (5)
     - | プロパティファイルの値が暗号化された値かどうかをチェックする。判定方法については実装によって異なる。
       | ここでは値が"Encrypted:"から始まるかどうかで、暗号化されているかどうかを判断する。
       | 暗号化されている場合、(6)で復号を実施し、暗号化されていない場合、そのままの値を返却する。
   * - | (6)
     - | プロパティファイルの暗号化されたvalueの復号を行っている。(具体的な復号処理については省略する。)
       | 復号の方法については実装によって異なる。

- プロパティを取得するHelper

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

   * - 項番
     - 説明
   * - | (1)
     - | ``@Value`` アノテーションのvalueに ``${encrypted.property.string}`` を設定することで、読み込まれたプロパティファイルのキー\ ``encrypted.property.string``\ に対する値が復号されて代入される。
       | ``testString`` にはStringクラスに復号された値が代入される。
   * - | (2)
     - | ``@Value`` アノテーションのvalueに ``${encrypted.property.int}`` を設定することで、読み込まれたプロパティファイルのキー\ ``encrypted.property.int``\ に対する値が復号されて代入される。
       | ``testInt`` には整数型に復号された値が代入される。
   * - | (3)
     - | ``@Value`` アノテーションのvalueに ``${encrypted.property.integer}`` を設定することで、読み込まれたプロパティファイルのキー\ ``encrypted.property.integer``\ に対する値が復号されて代入される。
       | ``testInteger`` にはIntegerクラスに復号された値が代入される。
   * - | (4)
     - | ``@Value`` アノテーションのvalueに ``${encrypted.property.file}`` を設定することで、読み込まれたプロパティファイルのキー\ ``encrypted.property.file``\ に対する値が復号されて代入される。
       | ``testFile`` には初期値に復号された値でオブジェクト生成されたFileオブジェクトが格納される。(自動変換)

**プロパティファイル**

| プロパティ値として、暗号化した値のprefixに、暗号化されていることを示す"Encrypted:"を付加している。
| 暗号化されているため、プロパティファイルの中身を見ても理解できない状態になっている。

 .. code-block:: properties

   encrypted.property.string=Encrypted:ZlpbQRJRWlNAU1FGV0ASRVteXhJQVxJXXFFAS0JGV1Yc
   encrypted.property.int=Encrypted:AwI=
   encrypted.property.integer=Encrypted:AwICAgI=
   encrypted.property.file=Encrypted:YkBdQldARkt/U1xTVVdfV1xGHFpGX14=

.. raw:: latex

   \newpage

