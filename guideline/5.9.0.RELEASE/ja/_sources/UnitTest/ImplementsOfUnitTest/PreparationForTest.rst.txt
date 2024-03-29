
.. _PreparationForTest:

テストの事前準備
--------------------------------------------------------------------------------

.. only:: html

.. contents:: 目次
  :local:

本節では単体テストを実施するための事前準備として、利用するOSSライブラリの設定方法とデータセットアップ方法およびテスト実装例で使用する設定ファイルについて説明する。

|

OSSライブラリの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

単体テストを実行するプロジェクト（domainプロジェクト、webプロジェクト）のPOMファイルにテストで利用するOSSライブラリを設定する。

* \ ``pom.xml``\

.. code-block:: xml

  <!-- == Begin Database == -->
  <!-- <dependency> -->
  <!-- <groupId>org.postgresql</groupId> -->
  <!-- <artifactId>postgresql</artifactId> -->
  <!-- <scope>test</scope> -->
  <!-- </dependency> -->
  <!-- == End Database == -->

  <!-- == Begin Unit Test == -->
  <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <scope>test</scope>
  </dependency>
  <dependency>
      <groupId>org.mockito</groupId>
      <artifactId>mockito-core</artifactId>
      <scope>test</scope>
  </dependency>
  <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <scope>test</scope>
  </dependency>
  <dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-el</artifactId>
    <scope>test</scope>
  </dependency>
  <dependency>
    <groupId>org.dbunit</groupId>
    <artifactId>dbunit</artifactId>
    <scope>test</scope>
  </dependency>
  <dependency>
    <groupId>com.github.springtestdbunit</groupId>
    <artifactId>spring-test-dbunit</artifactId>
    <scope>test</scope>
  </dependency>
  <dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <scope>test</scope>
  </dependency>
  <!-- == End Unit Test == -->

.. note::

  上記設定例は、依存ライブラリのバージョンを親プロジェクトであるterasoluna-gfw-parentで管理する前提であるため、\ ``pom.xml``\ でのバージョン指定は不要である。また、\ ``hamcrest``\ については、\ ``junit``\ が依存関係を解決しているため、改めて定義する必要はない。

.. tip:: \ **PostgreSQLドライバの追加方法について**\

  データアクセスを伴うテストでPostgreSQLのドライバを使用する場合は、POMファイル内のPostgreSQLのドライバのコメントアウトを外すこと。なお、テストのために依存ライブラリが必要になる場合のスコープは\ ``test``\ が適切である。

|

データベースのセットアップ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _PreparationForTestDataSetupWithSpringTest:

スキーマとテストデータのセットアップ（Spring Test標準機能のみを利用したテストの場合）
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

単体テストで利用するデータベースのセットアップについて、以下の方法がある。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.40\linewidth}|p{0.30\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 30 40 30

  * - セットアップ方法
    - 特徴
    - 利用シーン
  * - | \ ``<jdbc:initialize-database>``\ 要素を使用する。
    - | \ ``<jdbc:initialize-database>``\ 要素を定義した設定ファイルをテスト実施時に読み込みセットアップする。
    - | インメモリデータベース(H2 Database)をセットアップする際に使用する。
  * - | initdbプロジェクトを使用する。
    - | テスト実施と分離して事前にDBの初期化ができる。
    - | テスト実施前にまとめてデータベースをセットアップする際に使用する。
  * - | \ ``@org.springframework.test.context.jdbc.Sql``\ アノテーションを使用する。
    - | \ ``@Sql``\ アノテーションの引数で指定したSQLを発行する。
      | \ ``@Sql``\ アノテーションはメソッドレベル、クラスレベルで指定できる。
      | メソッドレベルで指定した場合は指定したテストメソッドだけで、クラスレベルで指定した場合は\ ``@Sql``\ アノテーションの指定がないすべてのテストメソッドで、実行前後にSQLを発行できる。
    - | テストごとにテストデータをセットアップする際に使用する。

.. warning::

  \ ``<jdbc:initialize-database>``\ タグに設定するSQLファイルには、明示的に「COMMIT;」を記述すること。

| 単体テストで利用するスキーマのセットアップは、テストごとではなくテスト実施前にまとめて実施されることが想定される。
| そのため、本章ではテストと分離したinitdbプロジェクトを使用してスキーマを作成することを前提に説明する。
| initdbプロジェクトについては、\ :ref:`CreateWebApplicationProjectConfigurationInitdb`\ を参照されたい。

| 一方、テストデータのセットアップはテストごとに実施されることが想定される。
| そのため、本章ではテストクラスまたはテストメソッド毎にSQLを発行できる\ ``@Sql``\ アノテーションを使用することを前提に説明する。

以下に、メソッドレベルに\ ``@Sql``\ アノテーションを付与する場合のテストデータのセットアップ例を示す。

* \ ``MemberRepositoryTest.java``\

.. code-block:: java

  public class MemberRepositoryTest {

  @Test
  @Sql(scripts = "classpath:META-INF/sql/setupMemberLogin.sql" // (1)
       config = @SqlConfig(encoding = "utf-8")) // (2)
  public void testUpdateMemberLogin() {
      // omitted
  }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | \ ``@Sql``\ アノテーションに、テストに必要なデータを投入するSQLファイルを指定する。
  * - | (2)
    - | \ ``@SqlConfig``\ アノテーションを使用してSQLファイルのエンコードを指定する。

.. _PreparationForTestTipSqlAnnotation:

.. Tip:: \ **@Sqlについて**\

  \ ``@Sql``\ アノテーションの引数には、以下を指定できる。

  * SQLファイル（\ ``scripts``\ または \ ``value``\ ）
  * SQLステートメント（\ ``statements``\ ）
  * SQL実行フェイズ（\ ``executionPhase``\ ）
  * SQL解析メタデータ(\ ``config``\ に\ ``@SqlConfig``\ アノテーションを指定)

  また、\ ``@Sql``\ アノテーションはデフォルトで有効になっている\ ``SqlScriptsTestExecutionListener``\ によって実行される。

  詳細は、\ `Executing SQL scripts declaratively with @Sql <https://docs.spring.io/spring-framework/docs/6.1.3/reference/html/testing.html#testcontext-executing-sql-declaratively>`_\ を参照されたい。

  なお、\ ``@Sql``\ アノテーションと\ ``@SqlConfig``\ アノテーションによる構成は\ ``<jdbc:initialize-database>``\ 要素による構成の上位セットである。

.. _PreparationForTestNoteOmittedSqlFilePath:

.. note:: \ **@SqlのSQLファイルパスの省略**\

  \ ``@Sql``\ アノテーションは、\ ``@ContextConfiguration``\ アノテーション同様、SQLファイルのパスを省略でき、省略した場合\ ``@Sql``\ アノテーションが指定された場所に基づいてSQLファイルの検索が行われる。

  例えば、以下のようにデフォルトのパスにあるファイルがロードされる。

  \ ``com.example.domain.repository.SampleRepositoryTest``\ に指定した場合 → \ ``classpath:com/example/domain/repository/SampleRepositoryTest.sql``\
    
  \ ``SampleRepositoryTest#testUpdate()``\ に指定した場合 → \ ``classpath:com/example/domain/repository/SampleRepositoryTest.testUpdate.sql``\

  なお、デフォルトのパスを検出できない場合は、\ ``java.lang.IllegalStateException``\ がthrowされる。

.. note:: \ **@Sqlの複数指定**\

  \ ``@Sql``\ にはJava SE8から追加された\ ``@Repeatable``\ が付与されているため、同じ箇所に複数指定することができる。

  なお、\ ``@org.springframework.test.context.jdbc.SqlGroup``\ を使用して、\ ``@Sql``\ を配列で複数指定することも可能である。

|

.. _PreparationForTestDataSetupWithDBUnit:

テストデータのセットアップ（Spring Test DBUnitを利用したテスト場合）
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| DBUnitとは、データベースに依存するクラスのテストを行うためのJUnit拡張フレームワークである。
| DBUnitとSpring Test DBUnitを使用して、テスト用データベースをセットアップする方法を説明する。

| DBUnitは、表形式で記載したデータベース情報をJavaオブジェクトとして抽象化して操作するための\ ``org.dbunit.dataset.IDataSet``\ インタフェースを提供している。
| \ ``IDataSet``\ インタフェースを使用することで、テストデータや期待結果データを定義したデータ定義ファイルを読み込むことができ、デフォルトではFlat XML形式のファイルが使用される。
| DBUnitはFlat XML形式の他に、Excel形式（.xlsx）やCSV形式などに対応した\ ``IDataSet``\ インタフェースの実装クラスを持つ。

| Spring Test DBUnitではデータ定義ファイルの読込機能を\ ``com.github.springtestdbunit.dataset.DataSetLoader``\ インタフェースの実装クラスに委譲している。デフォルトではXML形式のデータ定義ファイルが読み込まれる。
| ファイル形式を変更したい場合は、変更したい形式に対応した\ ``IDataSet``\ インタフェースの実装クラスを生成する\ ``DataSetLoader``\ インタフェースの実装クラスを作成することで実現できる。

| なお、Spring Test DBUnitを使用してデータのセットアップをする場合は、\ ``@DatabaseSetup``\ アノテーションを使用することでテストコードにテストデータを定義したファイルを読み込ませることができる。
| \ ``@DatabaseSetup``\ アノテーションはクラスレベル、メソッドレベルで指定でき、メソッドレベルに指定した場合は指定したメソッド、クラスレベルで指定した場合は各メソッドのテスト実行前に指定したファイルでデータのセットアップが行われる。

| 本章では、Excel形式（.xlsx）のデータ定義ファイルを使用することを前提に説明する。
| Excel形式に対応する\ ``DataSetLoader``\ インタフェースの実装例を以下に示す。

* \ ``XlsDataSetLoader.java``\

.. code-block:: java

  public class XlsDataSetLoader extends AbstractDataSetLoader { // (1)

      @Override
      protected IDataSet createDataSet(
              Resource resource) throws IOException, DataSetException {
          try (InputStream inputStream = resource.getInputStream()) {
              return new XlsDataSet(inputStream);
          }
      }
  }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | Spring Test DBUnitが提供する抽象基底クラスである\ ``com.github.springtestdbunit.dataset.AbstractDataSetLoader``\ を利用して、Excel形式のデータ定義ファイルの\ ``XlsDataSetLoader``\ クラスを定義する。

* \ ``MemberRepositoryDbunitTest.java``\

.. code-block:: java

  // omitted
  @DbUnitConfiguration(dataSetLoader = XlsDataSetLoader.class) // (1)
  public class MemberRepositoryDbunitTest {
      // omitted

      @Test
      @DatabaseSetup("classpath:META-INF/dbunit/setup_MemberLogin.xlsx")
      @ExpectedDatabase(value = "classpath:META-INF/dbunit/expected_testUpdateMemberLogin.xlsx", assertionMode = DatabaseAssertionMode.NON_STRICT_UNORDERED)
      public void testUpdateMemberLogin() {

          // omitted
  }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | \ ``@DbUnitConfiguration``\ アノテーションに\ ``XlsDataSetLoader``\ クラスを指定することで、\ ``@DatabaseSetup``\ アノテーションを使用したExcel形式のデータ定義ファイル読込みができるようになる。

* Excel形式のデータ定義ファイル（setup_MemberLogin.xlsx）

.. figure:: ./images_PreparationForTest/PreparationForTestExcelFile.png
  :width: 70%

| Excel形式のデータ定義ファイルでは、各シートが各テーブルに対応する。
| シート名にはテーブル名、シートの一行目にはカラム名を設定する。
| 二行目以降にテーブルに挿入されるデータを記述する。

.. note:: \ **CSV形式のデータ定義ファイルを使用する場合**\

  DBUnitでCSV形式のデータ定義ファイルを使用する場合は、\ ``IDataSet``\ インタフェースの実装クラスとして\ ``org.dbunit.dataset.csv.CsvDataSet.CsvDataSet``\ クラスを使用することで実現できる。

.. note:: \ **DBUnitがデフォルトで読み込むファイル形式について**\

  DBUnitは、デフォルトでFlat XML形式のデータ定義ファイルをサポートしている。
    
  Spring Test DBUnitを使用した場合は、\ ``@DbUnitConfiguration``\ に\ ``dataSetLoader``\ を指定しなかった場合、Flat XML形式のファイルに対応した\ ``IDataSet``\ インタフェースの実装クラスである\ ``org.dbunit.dataset.xml.FlatXmlDataSet``\ クラスが使用される。

  Flat XML形式のデータ定義ファイル例を以下に示す。

  * \ ``setup_MemberLogin.xml``\

    .. code-block:: xml

      <!-- (1) -->
      <?xml version='1.0' encoding='UTF-8'?>
      <dataset>
          <MEMBER_LOGIN CUSTOMER_NO="0000000001" PASSWORD="ABCDE" LAST_PASSWORD="VWXYZ" LOGIN_DATE_TIME="2017-09-13 16:47:04.283" LOGIN_FLG="FALSE" />
      </dataset>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - 項番
        - 説明
      * - | (1)
        - | \ ``dataset``\ 要素配下の各XML要素は、テーブルのレコードに対応しており、各XMLの要素名にテーブル名、属性名にカラム名、属性値に投入するデータを定義する。例では、\ ``MEMBER_LOGIN``\ テーブルに値を定義している。

|

.. _PreparationForTestMakeSettingFileForSpringTest:

テスト実装例で使用する設定ファイル
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| Spring Testの DI機能を使用することでテストで使用するBeanを定義した設定ファイルを読み込み、テスト時に使用することができる。
| 詳細は\ :ref:`UsageOfLibraryForTestDIOfSpringTest`\ を参照されたい。

.. tabs::
  .. group-tab:: Java Config

    本章では、テストを行う際に必要な設定を\ ``TestContextConfig.java``\ に定義し、その設定ファイルをテスト時の共通設定としている。なお、\ ``TestContextConfig.java``\ はdomainプロジェクトの\ ``src/test/java/com/example/config/TestContextConfig.java``\ から各層ごとにアプリケーションが保持する設定ファイル（\ ``SampleInfraConfig.java``\ など）と組み合わせて読み込む方針でテストを実装している。

    .. note:: \ **単体テストで利用する設定ファイルの作成単位**\

      本章では上記のように設定ファイルを作成しているが、実際に設定ファイルを用意する際には、アーキテクトが業務要件を考慮して共通設定を定義し、それを元にテスト実装チームで必要な設定を追加するようにして対応すること。

    以下に本章の実装例で使用する設定ファイルを示す。

    * \ ``TestContextConfig.java``\

    .. code-block:: java

      @Configuration
      @EnableTransactionManagement
      public class TestContextConfig {

          /**
           * Configure {@link PropertySourcesPlaceholderConfigurer} bean.
           * @param properties Property files to be read
           * @return Bean of configured {@link PropertySourcesPlaceholderConfigurer}
           */
          @Bean
          public static PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer(
                  @Value("classpath*:/META-INF/spring/*.properties") Resource... properties) {
              PropertySourcesPlaceholderConfigurer bean = new PropertySourcesPlaceholderConfigurer();
              bean.setLocations(properties);
              return bean;
          }

          // (1)
          /**
           * Configure {@link ExceptionLogger} bean.
           * @return Bean of configured {@link ExceptionLogger}
           */
          @Bean("exceptionLogger")
          public ExceptionLogger exceptionLogger() {
              return new ExceptionLogger();
          }
              
          /**
           * Configure {@link JdbcTemplate} bean.
           * @param dataSource Bean defined by SampleEnvConfig#dataSource
           * @see com.example.config.app.SampleEnvConfig#dataSource()
           * @return Bean of configured {@link JdbcTemplate}
           */
          @Bean("jdbcTemplate")
          public JdbcTemplate jdbcTemplateForCodeList(DataSource dataSource) {
              JdbcTemplate bean = new JdbcTemplate();
              bean.setDataSource(dataSource);
              return bean;
          }
              
          // (2)
          /**
           * Configure {@link PasswordEncoder} bean.
           * @return Bean of configured {@link DelegatingPasswordEncoder}
           */
          @Bean("passwordEncoder")
          public PasswordEncoder passwordEncoder() {
              Map<String, PasswordEncoder> idToPasswordEncoder = new HashMap<>();
              idToPasswordEncoder.put("pbkdf2@SpringSecurity_v5_8", pbkdf2PasswordEncoder());
              idToPasswordEncoder.put("bcrypt", bCryptPasswordEncoder());
              /* When using commented out PasswordEncoders, you need to add bcprov-jdk18on.jar to the dependency.
              idToPasswordEncoder.put("argon2@SpringSecurity_v5_8", argon2PasswordEncoder());
              idToPasswordEncoder.put("scrypt@SpringSecurity_v5_8", sCryptPasswordEncoder());
              */
              return new DelegatingPasswordEncoder("pbkdf2@SpringSecurity_v5_8", idToPasswordEncoder);
          }

          /**
           * Configure {@link Pbkdf2PasswordEncoder} bean.
           * @return Bean of configured {@link Pbkdf2PasswordEncoder}
           */
          @Bean
          public Pbkdf2PasswordEncoder pbkdf2PasswordEncoder() {
              return Pbkdf2PasswordEncoder.defaultsForSpringSecurity_v5_8();
          }

          /**
           * Configure {@link BCryptPasswordEncoder} bean.
           * @return Bean of configured {@link BCryptPasswordEncoder}
           */
          @Bean
          public BCryptPasswordEncoder bCryptPasswordEncoder() {
              return new BCryptPasswordEncoder();
          }
          /* When using commented out PasswordEncoders, you need to add bcprov-jdk18on.jar to the dependency.
          @Bean
          public Argon2PasswordEncoder argon2PasswordEncoder() {
              return Argon2PasswordEncoder.defaultsForSpringSecurity_v5_8();
          }
          @Bean
          public SCryptPasswordEncoder sCryptPasswordEncoder() {
              return SCryptPasswordEncoder.defaultsForSpringSecurity_v5_8();
          }
          */

          // 業務に応じてBean定義を追加
      }


    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - 項番
        - 説明
      * - | (1)
        - | テスト実施に必要なBeanを定義する。
      * - | (2)
        - | ここでは、テスト例を実装するために\ ``passwordEncoder``\ のBean定義を追加している。
          | Bean定義については、業務に応じて適宜追加されたい。

  .. group-tab:: XML Config

    本章では、テストを行う際に必要な設定を\ ``test-context.xml``\ に定義し、その設定ファイルをテスト時の共通設定としている。なお、\ ``test-context.xml``\ はdomainプロジェクトの\ ``src/test/resources/test-context.xml``\ から\ ``<import resource="classpath:META-INF/spring/projectName-domain.xml" />``\ を削除し、各層ごとにアプリケーションが保持する設定ファイル（\ ``sample-infra.xml``\ など）と組み合わせて読み込む方針でテストを実装している。

    .. note:: \ **単体テストで利用する設定ファイルの作成単位**\

      本章では上記のように設定ファイルを作成しているが、実際に設定ファイルを用意する際には、アーキテクトが業務要件を考慮して共通設定を定義し、それを元にテスト実装チームで必要な設定を追加するようにして対応すること。

    以下に本章の実装例で使用する設定ファイルを示す。

    * \ ``test-context.xml``\

    .. code-block:: xml

      <?xml version="1.0" encoding="UTF-8"?>
      <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

        <context:property-placeholder
          location="classpath*:/META-INF/spring/*.properties" />

        <!-- (1) -->
        <bean id="exceptionLogger" class="org.terasoluna.gfw.common.exception.ExceptionLogger" />

        <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
          <property name="dataSource" ref="dataSource" />
        </bean>

        <!-- (2) -->
        <bean id="passwordEncoder" class="org.springframework.security.crypto.password.DelegatingPasswordEncoder">
          <constructor-arg name="idForEncode" value="pbkdf2@SpringSecurity_v5_8" />
          <constructor-arg name="idToPasswordEncoder">
            <map>
              <entry key="pbkdf2@SpringSecurity_v5_8">
                <bean class="org.springframework.security.crypto.password.Pbkdf2PasswordEncoder" factory-method="defaultsForSpringSecurity_v5_8" />
              </entry>
              <entry key="bcrypt">
                <bean class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder" />
              </entry>
              <!-- When using commented out PasswordEncoders, you need to add bcprov-jdk18on.jar to the dependency.
              <entry key="argon2@SpringSecurity_v5_8">
                <bean class="org.springframework.security.crypto.argon2.Argon2PasswordEncoder" factory-method="defaultsForSpringSecurity_v5_8" />
              </entry>
              <entry key="scrypt@SpringSecurity_v5_8">
                <bean class="org.springframework.security.crypto.scrypt.SCryptPasswordEncoder" factory-method="defaultsForSpringSecurity_v5_8" />
              </entry>
              -->
            </map>
          </constructor-arg>
        </bean>
        <!-- 業務に応じてBean定義を追加 -->

      </beans>


    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - 項番
        - 説明
      * - | (1)
        - | テスト実施に必要なBeanを定義する。
      * - | (2)
        - | ここでは、テスト例を実装するために\ ``passwordEncoder``\ のBean定義を追加している。
          | Bean定義については、業務に応じて適宜追加されたい。

.. raw:: latex

  \newpage
