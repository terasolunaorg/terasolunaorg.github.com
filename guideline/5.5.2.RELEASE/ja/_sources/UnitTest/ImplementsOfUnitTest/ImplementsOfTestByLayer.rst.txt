
.. _ImplementsOfTestByLayer:

レイヤごとのテスト実装
--------------------------------------------------------------------------------

.. only:: html

 .. contents:: 目次
    :local:

レイヤごとの単体テスト対象クラス、テスト方法およびその概要の一覧を以下に示す。

なお、本章で提示するテスト方法および実装はあくまで一例であり、実際はテスト方針に合わせたテスト方法および実装を検討いただきたい。


.. tabularcolumns:: |p{0.15\linewidth}|p{0.25\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 15 25 60

    * - レイヤ
      - テスト方法
      - 概要
    * - インフラストラクチャ層
      - \ :ref:`ImplementsOfTestByLayerTestingRepositoryWithSpringTest`\
      - Spring Testの標準的な機能を使用してデータアクセスのテストを行う。
    * - インフラストラクチャ層
      - \ :ref:`ImplementsOfTestByLayerTestingRepositoryWithSpringTestDBUnit`\
      - DBUnitとSpring Test DBUnitの機能を使用してデータアクセスのテストを行う。
    * - ドメイン層
      - \ :ref:`ImplementsOfTestByLayerTestingServiceWithSpringTest`\
      - Spring TestのDI機能を使用して\ ``Service``\ をインジェクションし、インフラストラクチャ層と結合し\ ``Service``\ のテストを行う。
    * - ドメイン層
      - \ :ref:`ImplementsOfTestByLayerTestingServiceWithMockito`\
      - Mockitoを使用して依存するクラスをモック化し\ ``Service``\ のテストを行う。
    * - アプリケーション層
      - \ :ref:`ImplementsOfTestByLayerTestingControllerWithStandaloneSetup`\
      - Spring TestのDI機能を使用して\ ``Controller``\ をインジェクションし、ドメイン層、インフラストラクチャ層と結合した状態で\ ``Controller``\ のテストを行う。
    * - アプリケーション層
      - \ :ref:`ImplementsOfTestByLayerTestingControllerWithWebAppContextSetup`\
      - Spring TestのMockMvcを使用して業務で作成した\ ``spring-mvc.xml``\ と\ ``applicationContext.xml``\ を適用し、\ ``Controller``\ のテストを行う。
    * - アプリケーション層
      - \ :ref:`ImplementsOfTestByLayerTestingControllerWithMockito`\
      - Mockitoを使用して依存するクラスをモック化し\ ``Controller``\ のテストを行う。
    * - アプリケーション層
      - \ :ref:`ImplementsOfTestByLayerUnitTestOfHelper`\
      - \ ``Helper``\ のテストを行う。テスト方法の詳細は\ ``Service``\ のテスト方法を参照。

インフラストラクチャ層の単体テスト
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

本節では、開発ガイドラインの\ :ref:`LayerOfInfrastructure`\ の単体テストについて説明する。

.. figure:: ./images/ImplementsOfTestByLayerLayerOfTestTargetRepository.png
   :width: 95%

インフラストラクチャ層では、RepositoryからMyBatis（O/R Mapper）を利用したデータアクセスのテストを行う。
MyBatis3の使用方法の詳細については、\ :ref:`repository-mybatis3-label`\ を参照されたい。

MyBatisにより自動生成される\ ``RepositoryImpl``\ はSpringのDIコンテナ上で実行されるため、テストには、
本番同様のBean定義と、SpringのDI機能を提供するSpring Testの\ ``SpringJUnit4ClassRunner``\ を使用する。
Spring Testの詳細は\ :ref:`UsageOfLibraryForTestSpringTestOverview`\ を参照されたい。

テスト実行後のデータ検証方法には以下の2通りある。
どちらを使用するかは別途業務要件に合わせて検討いただきたい。

* テスト実行後のデータベースの状態をSELECT文を使用して取得し検証する。
* DBUnitとSpring Test DBUnitを使用して検証する。

本節では、SELECT文を使用した検証方法として\ ``JdbcTemplate``\ を使用した場合を例に説明する。
\ ``JdbcTemplate``\ とはSpring JDBCサポートのコアクラスである。JDBC APIではデータソースからコネクションの取得、
\ ``PreparedStatement``\ の作成、\ ``ResultSet``\ の解析、コネクションの解放などを行う必要があるが、
\ ``JdbcTemplate``\ を使用することでこれらの処理の多くが隠蔽され、より簡単にデータアクセスを行うことができる。

.. note::

    \ :ref:`ApplicationLayering`\ では、\ ``Repository``\ インターフェイスはドメイン層の成果物であるが、
    インフラストラクチャ層の単体テスト対象として紹介している。\ ``Service``\ とのインターフェイスが正しいことは、
    ドメイン層の単体テストでも確認することを推奨する。

|

.. _ImplementsOfTestByLayerUnitTestOfRepository:

Repositoryの単体テスト
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

本節では、以下の\ ``Repository``\ の単体テスト実装方法を説明する。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - テスト方法
      - 説明
    * - \ :ref:`ImplementsOfTestByLayerTestingRepositoryWithSpringTest`\
      - \ ``JdbcTemplate``\ を使用してテスト結果の検証を行う。
    * - \ :ref:`ImplementsOfTestByLayerTestingRepositoryWithSpringTestDBUnit`\
      - DBUnit、Spring Test DBUnitの機能を使用してテスト結果の検証を行う。

ここでは、以下の成果物に対するテストを例に説明する。
なお、Repositoryの実装の詳細は、\ :ref:`repository-mybatis3-label`\ を参照されたい。

* \ ``Repository``\ インタフェース（\ ``MemberRepository``\）の更新処理（\ ``updateMemberLogin``\ メソッド）
* マッピングファイル（\ ``MemberRepository.xml``\）

以下に、テスト対象の実装例を示す。

* ``MemberRepository.java``

.. code-block:: java

    public interface MemberRepository {

        int updateMemberLogin(Member member);
    }

* ``MemberRepository.xml``

.. code-block:: xml

    <mapper namespace="com.example.domain.repository.member.MemberRepository">

      <update id="updateMemberLogin" parameterType="Member">
        UPDATE member_login SET
            last_password = password,
            password = #{memberLogin.password}
        WHERE
            customer_no = #{membershipNumber}
      </update>

    </mapper>

|

.. _ImplementsOfTestByLayerTestingRepositoryWithSpringTest:

Spring Test標準機能のみを利用したテスト
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Spring Testを使用した\ ``Repository``\ の単体テストにおいて、作成するファイルを以下に示す。
なお、データベースのセットアップ方法については\ :ref:`PreparationForTestDataSetupWithSpringTest` \ を参照されたい。
また、Spring Testを使用して単体テストを行う際に使用する設定ファイルは\ :ref:`PreparationForTestMakeSettingFileForSpringTest`\
を参照されたい。

.. figure:: ./images/ImplementsOfTestByLayerRepositorySpringTestItems.png

.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 35 65

    * - 作成するファイル名
      - 説明
    * - \ ``MemberRepositoryTest.java``\
      - \ ``MemberRepository.java``\ のテストクラス。
    * - \ ``test-context.xml``\ 
      - Spring Testを使用して単体テストを行う際に必要な設定を補うための設定ファイル。
    * - \ ``setupMemberLogin.sql``\
      - 単体テストで利用するデータベースのデータをセットアップするためのSQLファイル。

.. note:: **単体テストで利用するSQLファイルの作成単位**

    ここでは、１テストメソッドに１つのSQLを作成している。実際の作成単位については、テスト方針や内容に応じて
    適宜検討されたい。
    なお、\ ``@Sql``\ にSQLファイルパスを省略した場合、\ ``@Sql``\ の指定場所に基づいてSQLファイルの検索が行われる。
    詳細は、\ :ref:`@SqlのSQLファイルパスの省略<PreparationForTestNoteOmittedSqlFilePath>`\ を参照されたい。

|

Spring Testを使用する場合の\ ``Repository``\ のテストクラス作成方法を説明する。

以下に、データアクセスを利用してテストするために使用する設定ファイルを示す。

* ``sample-infra.xml``

.. code-block:: xml

    <import resource="classpath:/META-INF/spring/sample-env.xml" />

    <!-- define the SqlSessionFactory -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <property name="configLocation" value="classpath:/META-INF/mybatis/mybatis-config.xml" />
    </bean>

    <!-- scan for Mappers -->
    <mybatis:scan base-package="com.example.domain.repository" />

* ``sample-env.xml``

.. code-block:: xml

    <bean id="realDataSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
      <property name="driverClassName" value="org.postgresql.Driver" />
      <property name="url" value="jdbc:postgresql://localhost:5432/sample" />
      <property name="username" value="sample" />
      <property name="password" value="xxxx" />
      <property name="defaultAutoCommit" value="false" />
      <property name="maxTotal" value="96" />
      <property name="maxIdle" value="16" />
      <property name="minIdle" value="0" />
      <property name="maxWaitMillis" value="60000" />
    </bean>

    <bean id="dataSource" class="net.sf.log4jdbc.Log4jdbcProxyDataSource">
      <constructor-arg index="0" ref="realDataSource" />
    </bean>

    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
      <property name="dataSource" ref="dataSource" />
    </bean>

  <bean id="dateFactory" class="org.terasoluna.gfw.common.date.jodatime.DefaultJodaTimeDateFactory" />

以下に、Spring Testを使用した\ ``Repository``\ のテスト作成方法について説明する。
ここでは、テスト用のスキーマは作成済みであることを前提に、\ ``@Sql``\ アノテーションを使用して\ ``MemberLogin``\ テーブル
をセットアップし、\ ``MemberLogin``\ のパスワード「ABCDE」が新しいパスワード「FGHIJ」に更新されることを更新後の
\ ``MemberLogin``\ テーブルを取得して確認している。


* ``MemberRepositoryTest.java``

.. code-block:: java

    import static org.hamcrest.CoreMatchers.*;
    import static org.junit.Assert.*;

    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration(locations = {
            "classpath:META-INF/spring/sample-infra.xml",   //(1)
            "classpath:META-INF/spring/test-context.xml" }) //(1)
    @Transactional // (2)
    public class MemberRepositoryTest {

        @Inject
        MemberRepository target; // (3)

        @Inject
        JdbcTemplate jdbctemplate; // (4)

        @Test
        @Sql(scripts = "classpath:META-INF/sql/setupMemberLogin.sql", config = @SqlConfig(encoding = "utf-8"))
        public void testUpdateMemberLogin() {

            // (5)
            // setup test data
            MemberLogin memberLogin = new MemberLogin();
            memberLogin.setPassword("FGHIJ");
            Member member = new Member();
            member.setMembershipNumber("0000000001");
            member.setMemberLogin(memberLogin);

            // (6)
            // run the test
            int updateCounts = target.updateMemberLogin(member);

            // (7)
            MemberLogin updateMemberLogin = getMemberLogin("0000000001");

            // (8)
            // assertion
            assertThat(updateCounts, is(1));
            assertThat(updateMemberLogin.getPassword(), is("FGHIJ"));
            assertThat(updateMemberLogin.getLastPassword(), is("ABCDE"));
        }

        private Member getMemberLogin(String customerNo) {

            MemberLogin memberLogin = (MemberLogin) jdbctemplate.queryForObject(
                    "SELECT * FROM member_login WHERE customer_no=?", 
                    new Object[] {customerNo }, 
                    new RowMapper<MemberLogin>() {

                        public MemberLogin mapRow(ResultSet rs,
                                    int rowNum) throws SQLException {

                                MemberLogin mapMemberLogin = new MemberLogin();

                                mapMemberLogin.setPassword(rs.getString(
                                        "password"));
                                mapMemberLogin.setLastPassword(rs.getString(
                                        "last_password"));
                                mapMemberLogin.setLoginDateTime(rs.getDate(
                                        "login_date_time"));
                                mapMemberLogin.setLoginFlg(rs.getBoolean(
                                        "login_flg"));

                                return mapMemberLogin;
                        }
                    });

            return memberLogin;
        }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``MemberRepository``\ クラスを動作させるために必要なアプリケーションが保持する\ ``sample-infra.xml``\ と
          \ ``test-context.xml``\ を読み込む。
    * - | (2)
      - | \ ``@Transactional``\ アノテーションを付与すると、テスト実行開始から終了まで一トランザクションとなり、デフォルト
          ではテスト終了後にロールバックされる。クラスレベルでアノテーションを定義すると、全テストメソッドに対して
          \ ``@Transactional``\ アノテーションが有効になる。
    * - | (3)
      - | テスト対象である\ ``MemberRepository``\ クラスをインジェクションする。
    * - | (4)
      - | \ ``JdbcTemplate``\ クラスをインジェクションする。
    * - | (5)
      - | テスト対象メソッドを実行するためのテストデータを作成する。
    * - | (6)
      - | テスト対象メソッドを実行する。
    * - | (7)
      - | 更新後のデータベースの情報を取得する。
          \ ``org.springframework.jdbc.core.RowMapper<T>``\ を使用することで、データベースから取得した\ ``ResultSet``\ を
          特定のPOJOクラスにマッピングすることができる。
    * - | (8)
      - | 更新件数、更新結果を確認する。

.. note:: **テスト時のトランザクションをロールバックさせない方法**

    \ ``@Transactional``\ アノテーションをテストケースに指定した場合、デフォルトでテストメソッド実行後にロールバック
    される。後続のテストでテストデータを使用するなどの目的でロールバックをさせたくない場合は、\ ``@Transactional``\
    アノテーションに加えて\ ``@Rollback(false)``\ アノテーションまたは\ ``@Commit``\ アノテーションを指定することで、
    テスト時のトランザクションをコミットすることができる。

.. warning:: **Spring Framework 4.2 以降の@TransactionConfigurationについて**

    Spring Framework 4.2 以降、クラスレベルで\ ``@Rollback``\ または\ ``@Commit``\ の設定が可能となった。
    これに伴い\ ``@TransactionConfiguration``\ が非推奨となった。但し、Spring Framework 4.2 より前のバージョンで
    クラスレベルでロールバックをする場合は\ ``@TransactionConfiguration(defaultRollback = true)``\ を設定すること。

|

.. _ImplementsOfTestByLayerTestingRepositoryWithSpringTestDBUnit:

Spring Test DBUnitを利用したテスト
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

データアクセスにDBUnitを使用する場合の\ ``Repository``\ の単体テスト実装方法について説明する。
なお、ここではDBUnitのデータ定義ファイルにExcel形式（.xlsx）のファイルを使用した場合を例に説明する。
データ定義ファイルとデータベースのセットアップ方法については、\ :ref:`PreparationForTestDataSetupWithDBUnit`\ を参照されたい。

また、DBUnitにSpring Test DBUnitの機能を組み合わせて使用するには、\ ``@TestExecutionListeners``\ アノテーションを使って、
\ ``com.github.springtestdbunit.TransactionDbUnitTestExecutionListener``\ を登録する必要がある。
登録方法ついては、\ :ref:`UsageOfLibraryForTestRegistrationOfTestExecutionListener`\ を参照されたい。

DBUnitを利用した\ ``Repository``\ の単体テストにおいて、作成するファイルを以下に示す。

.. figure:: ./images/ImplementsOfTestByLayerRepositoryDbunitItems.png

.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 35 65

    * - 作成するファイル名
      - 説明
    * - \ ``MemberRepositoryDbunitTest.java``\
      - \ ``MemberRepository.java``\ のテストクラス(DBUnitと連携する場合)
    * - \ ``XlsDataSetLoader.java``\
      - Excel形式に対応する\ ``DataSetLoader``\ インタフェースの実装クラス。
        実装方法については、\ :ref:`PreparationForTestDataSetupWithDBUnit`\ を参照されたい。
    * - \ ``expected_testUpdateMemberLogin.xlsx``\
      - テストの期待結果検証用ファイル
    * - \ ``setup_MemberLogin.xlsx``\
      - テストデータセットアップ用ファイル
    * - \ ``test-context.xml``\
      - Spring Testを使用して単体テストを行う際に使用する設定ファイル。\ :ref:`ImplementsOfTestByLayerTestingRepositoryWithSpringTest`\ で
        作成した設定ファイルと同じものを使用する。

.. note:: **単体テストで利用するExcelファイルの作成単位**

    ここでは、１テストメソッドにデータセットアップ用のファイルと期待結果検証用のファイルをそれぞれ１つずつ作成している。
    実際の作成単位については、テスト方針や内容に応じて適宜検討されたい。

|

DBUnitを使用する場合の\ ``Repository``\ のテストクラス作成方法を説明する。

ここでは、テスト用のスキーマは作成済みであることを前提に、\ ``@DatabaseSetup``\ アノテーションを使用して
\ ``MemberLogin``\ テーブルをセットアップし、\ ``MemberLogin``\ のパスワード「ABCDE」が新しいパスワード「FGHIJ」
に更新されることを\ ``@ExpectedDatabase``\ アノテーションを使用して確認している。

以下に、Spring TestとDBUnitを使用した\ ``Repository``\ のテスト作成方法を説明する。

* ``MemberRepositoryDbunitTest.java``

.. code-block:: java

    import static org.hamcrest.CoreMatchers.*;
    import static org.junit.Assert.*;

    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration(locations = {
            "classpath:META-INF/spring/sample-infra.xml",   // (1)
            "classpath:META-INF/spring/test-context.xml" }) // (1)
    @TestExecutionListeners({
            DirtiesContextBeforeModesTestExecutionListener.class,
            DependencyInjectionTestExecutionListener.class,
            DirtiesContextTestExecutionListener.class,
            TransactionDbUnitTestExecutionListener.class})
    @Transactional
    @DbUnitConfiguration(dataSetLoader = XlsDataSetLoader.class)
    public class MemberRepositoryDbunitTest {

        @Inject
        MemberRepository target;

        @Test
        @DatabaseSetup("classpath:META-INF/dbunit/setup_MemberLogin.xlsx")
        @ExpectedDatabase( // (2)
                value = "classpath:META-INF/dbunit/expected_testUpdateMemberLogin.xlsx",
                assertionMode = DatabaseAssertionMode.NON_STRICT_UNORDERED)
        public void testUpdate() {

            // setup
            MemberLogin memberLogin = new MemberLogin();
            memberLogin.setPassword("FGHIJ");
            Member member = new Member();
            member.setMembershipNumber("0000000001");
            member.setMemberLogin(memberLogin);

            // run the test
            int updateCounts = target.updateMemberLogin(member);

            // assertion
            assertThat(updateCounts, is(1));
        }
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``MemberRepository``\ クラスを動作させるために必要な設定ファイル（アプリケーションが保持する
          \ ``sample-infra.xml``\ とそれを補う\ ``test-context.xml``\）を読み込む。
    * - | (2)
      - | \ ``@ExpectedDatabase``\ アノテーションにテストの期待結果検証用ファイルを指定することでテストメソッド
          実行後にDBUnitによってテーブルと期待結果データファイルが自動で比較検証される。
        | \ ``@DatabaseSetup``\ アノテーション同様に、クラスレベルとメソッドレベルで付与できる。
        | ファイルフォーマットはテストセットアップ用データファイルと同じである。\ ``assertionMode``\ 属性には、
          以下の値が設定可能である。

        * \ ``DEFAULT``\（または指定なし）：全てのテーブルとカラムの一致を比較する。
        * \ ``NON_STRICT``\ ：期待結果データファイルに存在しないテーブル、カラムが実際のデータベースに存在しても無視する。
        * \ ``NON_STRICT_UNORDERED``\ ：\ ``NON_STRICT``\ モードに加え、行の順序についても無視する。


.. warning:: **外部キー制約のあるテーブル**

    外部キー制約のあるテーブルに対し、DBUnitを用いてデータベースを初期化すると、参照条件によってはエラーが発生するため、
    参照整合性を保つようにデータセットの順序を指定する必要があることに注意されたい。

.. note:: **シーケンスの検証方法**

    シーケンスは、トランザクションをロールバックしても進んだ値は戻らないという特徴を持つ。
    そのため、シーケンスから採番したカラムを持つレコードをDBUnitで検証する場合、以下のいずれかの対応を行う必要がある。

    * シーケンスから採番したカラムは検証対象外とする
    * 明示的にシーケンスの初期化を行うSQLを実行し、テストの実施前に初期化する
    * テスト実行時にシーケンスの値を確認し、確認した値を基準値として検証を行う

|

ドメイン層の単体テスト
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

本節では、開発ガイドラインの\ :ref:`LayerOfDomain`\ の単体テストについて説明する。

.. figure:: ./images/ImplementsOfTestByLayerLayerOfTestTargetDomain.png
   :width: 95%

ドメイン層では、\ ``Service``\ の業務ロジックと\ ``@Transactional``\ のテストを行う。
\ ``Service``\ をインジェクションし、インフラストラクチャ層を結合してテストを行う場合は、\ ``Repository``\ の
テスト実装方法と同様にBean定義と、Spring Testの\ ``SpringJUnit4ClassRunner``\ を使用してテストを行う。
Spring Testの詳細は\ :ref:`UsageOfLibraryForTestSpringTestOverview`\ を参照されたい。

|

.. _ImplementsOfTestByLayerUnitTestOfService:

Serviceの単体テスト
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

本節では、以下の\ ``Service``\ のテスト実装方法を説明する。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - テスト方法
      - 説明
    * - \ :ref:`ImplementsOfTestByLayerTestingServiceWithSpringTest`\
      - \ ``Service``\ をインジェクションし、インフラストラクチャ層と結合してテストを行う。
    * - \ :ref:`ImplementsOfTestByLayerTestingServiceWithMockito`\
      - \ ``Service``\ の実装クラスが依存するクラスをすべてモック化してテストを行う。

ここでは、以下の成果物に対するテストを例に説明する。
なお、Serviceの実装の詳細は、\ :ref:`service-label`\ を参照されたい。

* Serviceの実装クラス（\ ``TicketReserveServiceImpl``\）

以下に、テスト対象の実装例を示す。

* ``TicketReserveServiceImpl.java``

.. code-block:: java

    @Service
    @Transactional
    public class TicketReserveServiceImpl implements TicketReserveService {

        @Inject
        ReservationRepository reservationRepository;

        @Override
        public TicketReserveDto registerReservation(Reservation reservation)
                throws BusinessException {

            List<ReserveFlight> reserveFlightList = reservation.getReserveFlightList();

            // repository access
            int reservationInsertCount = reservationRepository.insert(reservation);
            if (reservationInsertCount != 1) {
                throw new SystemException(LogMessages.E_AR_A0_L9002.getCode(),
                        LogMessages.E_AR_A0_L9002.getMessage(reservationInsertCount, 1));
            }

            String reserveNo = reservation.getReserveNo();

            Date paymentDate = reserveFlightList.get(0).getFlight().getDepartureDate();

            return new TicketReserveDto(reserveNo, paymentDate);
        }
    }

以下に、テスト対象が使用するマッピングファイルを示す。

* ``ReservationRepository.xml``

.. code-block:: xml

    <mapper namespace="com.example.domain.repository.reservation.ReservationRepository">

      <insert id="insert" parameterType="Reservation">
        <selectKey keyProperty="reserveNo" resultType="String" order="BEFORE">
          SELECT TO_CHAR(NEXTVAL('sq_reservation_1'), 'FM0999999999')
        </selectKey>
        INSERT INTO reservation
        (
            reserve_no,
            reserve_date,
            total_fare,
            rep_family_name,
            rep_given_name,
            rep_age,
            rep_gender,
            rep_tel,
            rep_mail,
            rep_customer_no
        )
        VALUES
        (
            #{reserveNo},
            #{reserveDate},
            #{totalFare},
            #{repFamilyName},
            #{repGivenName},
            #{repAge},
            #{repGender.code},
            #{repTel},
            #{repMail},
            NULLIF(#{repMember.membershipNumber}, '')
        )
      </insert>

.. _ImplementsOfTestByLayerTestingServiceWithSpringTest:

依存クラスを利用したテスト
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

\ ``Service``\ をインジェクションし、インフラストラクチャ層を結合して行う。\ ``Service``\ のテストにおいて、
作成するファイルを以下に示す。

.. figure:: ./images/ImplementsOfTestByLayerServiceSpringTestItems.png

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - 作成するファイル名
      - 説明
    * - \ ``TicketReserveServiceImplTest.java``\
      - \ ``TicketReserveServiceImpl.java``\ のテストクラス
    * - \ ``test-context.xml``\
      - \ :ref:`PreparationForTestMakeSettingFileForSpringTest`\ で定義した設定ファイルを使用する。

|

テスト対象のServiceの実装クラスをインジェクションしてインフラストラクチャ層と結合してテストを行う場合のテスト作成方法を説明する。

以下に、テスト時に読み込む設定ファイルを示す。

* ``sample-domain.xml``

.. code-block:: xml

    <context:component-scan base-package="com.example.domain" />
    <tx:annotation-driven />

    <import resource="classpath:META-INF/spring/sample-infra.xml" />
    <import resource="classpath:META-INF/spring/sample-codelist.xml" />

    <bean id="resultMessagesLoggingInterceptor" 
          class="org.terasoluna.gfw.common.exception.ResultMessagesLoggingInterceptor">
      <property name="exceptionLogger" ref="exceptionLogger" />
    </bean>

    <aop:config>
      <aop:advisor advice-ref="resultMessagesLoggingInterceptor"
        pointcut="@within(org.springframework.stereotype.Service)" />
    </aop:config>

以下に、テスト実装例を示す。
テスト対象の\ ``TicketReserveServiceImpl#registerReservation()``\ メソッドを実行し、戻り値を確認している。
なお、データベースの状態の検証方法は\ :ref:`ImplementsOfTestByLayerUnitTestOfRepository`\ を参照されたい。

* ``TicketReserveServiceImplTest.java``

.. code-block:: java

    import static org.hamcrest.CoreMatchers.*;
    import static org.junit.Assert.*;

    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration(locations = {
            "classpath:META-INF/spring/sample-domain.xml", // (1)
            "classpath:META-INF/spring/test-context.xml"}) // (1)
    @Transactional
    public class TicketReserveServiceImplTest {

        @Inject
        TicketReserveService target;

        @Inject
        private JdbcTemplate jdbcTemplate;

        @Test
        @Sql(statements = "ALTER SEQUENCE sq_reservation_1 RESTART WITH 1") // (2)
        public void testRegisterReservation() {

            // setup
            Reservation inputReservation = new Reservation();
            inputReservation.setTotalFare(39200);
            inputReservation.setReserveNo("0000000001");
            // omitted

            // run the test
            TicketReserveDto actTicketReserveDto = target.registerReservation(
                    reservation);

            // assertion
            assertThat(actTicketReserveDto.getReserveNo(), is("0000000001"));
            // omitted
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``TicketReserveServiceImpl``\ クラスを動作させるために必要な設定ファイル（アプリケーションが保持する
          \ ``sample-domain.xml``\ とそれを補う\ ``test-domain.xml``\）を読み込む。
    * - | (2)
      - | \ ``@Sql``\ の\ ``statements``\ 属性を使用することでSQL文を直接指定することもできる。
          ここではテストメソッド実行前にシーケンスの初期化を行っている。

.. warning:: **テスト時のトランザクション管理**

    テストケースに\ ``@Transactional``\ アノテーションを付与すると、テスト実行開始から終了まで一トランザクションとなる。
    そのため、テストケースから\ ``@Transactional``\ アノテーションを付与した\ ``Service``\ クラスを呼び出した場合、
    テストケースからトランザクションが引き継がれる点に注意すること。
    例えば、トランザクションの伝播方法がデフォルト（\ ``REQUIRED``\ ）の場合、テストケースで開始した
    トランザクションでテスト対象の処理が行われ、コミット/ロールバックのタイミングもテスト終了時になる。
    トランザクションの伝播方法については\ :ref:`transaction-management-declare-transaction-info-label`\ を参照されたい。

.. _ImplementsOfTestByLayerTestingServiceWithMockito:

モックを利用したテスト
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

\ ``Service``\ の依存クラスをすべてモック化して行う\ ``Service``\ の単体テストにおいて、作成するファイルを以下に示す。

.. figure:: ./images/ImplementsOfTestByLayerServiceMockItems.png

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - 作成するファイル名
      - 説明
    * - \ ``TicketReserveServiceImplMockTest.java``\
      - \ ``TicketReserveServiceImpl.java``\ のテストクラス（モックを使用する場合）

|

テスト対象の\ ``Service``\ の実装クラスが依存するクラスをモック化する場合のテスト作成方法を説明する。
ここでは、\ ``ReservationRepository#insert()``\ メソッドをモック化し、テスト対象の
\ ``TicketReserveServiceImpl#registerReservation()``\ メソッドでモック化したメソッドが呼び出されることとテスト対象の
戻り値を確認している。

* ``TicketReserveServiceImplMockTest.java``

.. code-block:: java

    import static org.hamcrest.CoreMatchers.*;
    import static org.junit.Assert.*;
    import static org.mockito.Mockito.*;

    public class TicketReserveServiceImplMockTest {

        @Rule // (1)
        public MockitoRule mockito = MockitoJUnit.rule();

        @Mock // (2)
        ReservationRepository reservationRepository;

        @InjectMocks // (3)
        private TicketReserveServiceImpl target;

        @Test
        public void testRegisterReservation() {

            // setup
            Reservation inputReservation = new Reservation();
            inputReservation.setTotalFare(39200);
            inputReservation.setReserveNo("0000000001");
            // omitted

            when(reservationRepository.insert(inputReservation)).thenReturn(1); // (4)

            // run the test
            TicketReserveDto ticketReserveDto = target.registerReservation(inputReservation);

            // assertion
            verify(reservationRepository).insert(inputReservation); // (5)
            assertThat(ticketReserveDto.getReserveNo(), is("0000000001"));
            // omitted
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | モックの初期化とインジェクションをアノテーションベースで行うための宣言。
          詳細は\ :ref:`UsageOfLibraryForTestCreateMockObject`\ を参照されたい。
    * - | (2)
      - | \ ``@Mock``\ アノテーションを付与することで、\ ``TicketReserveServiceImpl``\ が依存している
          \ ``MemberRepository``\ をモック化している。
          詳細は\ :ref:`UsageOfLibraryForTestCreateMockObject`\ を参照されたい。
    * - | (3)
      - | \ ``@InjectMocks``\ アノテーションを付与することで、自動的にモックオブジェクトが代入される。
          詳細は\ :ref:`UsageOfLibraryForTestCreateMockObject`\ を参照されたい。
    * - | (4)
      - | \ ``ReservationRepository``\ の\ ``insert``\ メソッドについて、
          引数が\ ``inputReservation``\ の場合、返り値として"\ ``1``\" を返すように設定する。
          メソッドのモック化については、\ :ref:`UsageOfLibraryForTestMockingMethods`\ を参照されたい。
    * - | (5)
      - | \ ``ReservationRepository``\ の\ ``insert``\ メソッドについて、
          引数に\ ``inputReservation``\ が渡されて1回呼び出されたことを検証する。
          モック化したメソッドの検証については、\ :ref:`UsageOfLibraryForTestValidationOfMockMethod`\ を参照されたい。

|

アプリケーション層の単体テスト
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

アプリケーション層の単体テスト対象
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
本節では、開発ガイドラインの\ :ref:`LayerOfApplication`\ の単体テストについて説明する。

.. figure:: ./images/ImplementsOfTestByLayerLayerOfTestTargetApplication.png
   :width: 95%

アプリケーション層では、\ ``Controller``\ と\ ``Helper``\ のロジックを確認するためのテストを行う。
\ ``Controller``\ については以下の項目を確認する。

* @RequestMapping(リクエストパス、HTTPメソッド、リクエストパラメータ)
* 返却されるVIEW名

\ ``View``\ については、本来アプリケーション層に含まれるが、本ガイドラインでは対象外とする。

Spring Testは\ ``Controller``\ クラスをテストするためのサポートクラス(\ ``org.springframework.test.web.servlet.MockMvc``\
など)を用意している。
\ ``Controller``\ は\ ``MockMVC``\ を使用して疑似リクエストを送信してテストをするため、\ ``MockMVC``\ を提供する
Spring Testの\ ``SpringJUnit4ClassRunner``\ を使用する。
\ ``MockMvc``\ は\ ``Controller``\ に疑似リクエストを送信する仕組みを持ち、デプロイしたアプリケーションを模したテストを
行うことができる。\ ``MockMVC``\ の詳細は\ :ref:`UsageOfLibraryForTestMockMvcOverview`\ を参照されたい。

.. note:: **Formのバリデーションテスト**

    \ ``Form``\ のテストは、本来\ ``Controller``\ と組み合わせて実際の動作に近い形で行う必要があるが、
    \ ``Validation``\ の全パターンを\ ``Controller``\ と組み合わせるとテストの負担が大きくなる。
    そのため、単純な\ ``Validation``\ の確認であれば、\ ``Controller``\ と切り離して \ ``Form``\ 単体で\ ``Validation``\
    の確認を行うこともできる。テスト方法はテスト対象のFormを使用して\ :ref:`ImplementsOfTestByFunctionTestingBeanValidator`\ を実施すればよい。

|

Controllerの単体テスト
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ここでは、以下の\ ``Controller``\ の単体テスト実装方法を説明する。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - テスト方法
      - 説明
    * - \ :ref:`ImplementsOfTestByLayerTestingControllerWithStandaloneSetup`\
      - Spring Testが提供するデフォルトのコンテキストを使用し指定した設定ファイルを読み込むことでテストを行う。
    * - \ :ref:`ImplementsOfTestByLayerTestingControllerWithWebAppContextSetup`\
      - 実際に使用する\ ``applicationContext.xml``\ と\ ``spring-mvc.xml``\ を使用してテストを行う。
    * - \ :ref:`ImplementsOfTestByLayerTestingControllerWithMockito`\
      - \ ``Controller``\ が依存するクラスをすべてモック化してテストを行う。

ここでは、以下の成果物に対するテストを例に説明する。\ ``Controller``\ の実装の詳細は、\ :ref:`controller-label`\ を参照されたい。

* \ ``Controller``\ クラス（TicketSearchController）
* \ ``Controller``\ クラス（MemberRegisterController）

なお、インジェクションとモック化を組み合わせてテストを行いたい場合は、適宜以下に説明する実装方法を組み合わせて
実装されたい。

.. _ImplementsOfTestByLayerTestingControllerWithStandaloneSetup:

StandaloneSetupを利用したテスト
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

\ ``Controller``\ の依存クラスが利用できモック化する必要がない場合の\ ``Controller``\ の単体テストにおいて、\ ``StandaloneSetup``\ で作成するファイルを以下に示す。

.. figure:: ./images/ImplementsOfTestByLayerControllerStandaloneSetupItems.png

.. tabularcolumns:: |p{0.50\linewidth}|p{0.50\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 50 50

    * - 作成するファイル名
      - 説明
    * - \ ``MemberRegisterControllerStandaloneTest.java``\
      - \ ``MemberRegisterController.java``\ のテストクラス
    * - \ ``spring-mvc-test.xml``\
      - アプリケーション層に依存するコンポーネントを読み込むための\ ``component-scan``\ をテスト用に抽出した設定ファイル。
    * - \ ``test-context.xml``\
      - \ ``Controller``\ をドメイン層、インフラストラクチャ層と結合してテストを行う場合に使用する設定ファイル。

|

\ ``spring-mvc.xml``\ を使ってテストをすることが望ましいが、Spring Testが作成したコンテキストと
Spring MVCが作成したコンテキストが衝突しテスト実行ができないことがある。
そのため対応策として、テストに必要な設定のみ抽出し、テスト用の設定ファイルを用意する。

以下に、必要な設定のみ抽出した設定ファイルを示す。

* ``spring-mvc-test.xml``

.. code-block:: xml

    <context:component-scan base-package="com.example.app" />

\ ``ServiceImpl``\ クラスなどテスト対象の\ ``Controller``\ クラスが依存するクラスをインジェクションする場合の
テスト作成方法を説明する。
なお、テストでデータアクセスする場合の検証方法は\ :ref:`ImplementsOfTestByLayerUnitTestOfRepository`\ を、
呼び出すドメイン層のロジックを確認する方法は\ :ref:`ImplementsOfTestByLayerUnitTestOfService`\ を参照されたい。

以下に、テスト対象となる\ ``Controller``\ の実装例を示す。

* ``MemberRegisterController.java``

.. code-block:: java

    @Controller
    @RequestMapping("member/register")
    @TransactionTokenCheck("member/register")
    public class MemberRegisterController {

        @TransactionTokenCheck(type = TransactionTokenType.IN)
        @RequestMapping(method = RequestMethod.POST)
        public String register(@Validated MemberRegisterForm memberRegisterForm,
            BindingResult result, Model model, RedirectAttributes redirectAttributes) {

            if (result.hasErrors()) {
                throw new BadRequestException(result);
            }

            // omitted

            return "redirect:/member/register?complete";
        }
    }

ここでは、テスト対象の\ ``MemberRegisterController``\ クラスの\ ``register``\ メソッドを呼び出し、
リクエストマッピングと返却されるVIEWおよびリダイレクトされること（\ ``testRegisterConfirm01``\）、
不正な入力値を送信したときに\ ``BadRequestException``\ がthrowされていること（\ ``testRegisterConfirm02``\）の確認を行う。

以下に、\ ``ServiceImpl``\ クラスなどテスト対象の\ ``Controller``\ クラスが依存するクラスをインジェクションする場合の
テスト作成方法を説明する。なお、テストでデータアクセスする場合の検証方法は\ :ref:`ImplementsOfTestByLayerUnitTestOfRepository`\ を参照されたい。

* ``MemberRegisterControllerStandaloneTest.java``

.. code-block:: java

    import static org.hamcrest.CoreMatchers.*;
    import static org.junit.Assert.*;
    import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
    import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.*;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration(locations = {
            "classpath:META-INF/spring/applicationContext.xml", // (1)
            "classpath:META-INF/spring/test-context.xml",       // (1)
            "classpath:META-INF/spring/spring-mvc-test.xml"})   // (1)
    public class MemberRegisterControllerStandaloneTest {

        @Inject
        MemberRegisterController target;

        MockMvc mockMvc;

        @Before
        public void setUp() {

            // setup
            mockMvc = MockMvcBuilders.standaloneSetup(target).alwaysDo(log()).build(); // (2)
        }

        @Test
        public void testRegisterConfirm01() throws Exception {

            // setup and run the test
            mockMvc.perform(post("/member/register")
                        // omitted
                        .param("password", "testpassword")          // (3)
                        .param("reEnterPassword", "testpassword"))) // (3)
                        // assert
                        .andExpect(status().is(302))                                  // (4)
                        .andExpect(view().name("redirect:/member/register?complete")) // (4)
                        .andExpect(model().hasNoErrors());                            // (4)
        }

        @Test
        public void testRegisterConfirm02() throws Exception {

            try {
                // setup and run the test
                mockMvc.perform(post("/member/register")
                        // omitted
                        .param("password", "testpassword")
                        .param("reEnterPassword", "")) // (5)
                        // assert
                        .andExpect(status().is(400))
                        .andExpect(view().name("common/error/badRequest-error"))
                        .andReturn();

                fail("test failure!");
            } catch (Exception e) {

                // assert
                assertThat(e, is(instanceOf(NestedServletException.class)));         // (6)
                assertThat(e.getCause(), is(instanceOf(BadRequestException.class))); // (6)
            }
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``MemberRegisterController``\ クラスが依存する\ ``Service``\ 、\ ``Repository``\ を動作させるために必要な
          設定ファイル（アプリケーションが保持する\ ``applicationContext.xml``\ とそれを補う\ ``test-context.xml``\、 \ ``spring-mvc-test.xml``\）
          を読み込む。\ ``test-context.xml``\ は、\ :ref:`PreparationForTestMakeSettingFileForSpringTest`\ を使用している。
    * - | (2)
      - | 読み込んだBean定義から生成した\ ``Controller``\ を使用して、\ ``MockMvc``\ をセットアップする。
          セットアップの詳細については\ :ref:`UsageOfLibraryForTestSettingMockMvc`\ を参照されたい。
    * - | (3)
      - | \ ``MemberRegisterController``\ クラスの\ ``registerConfirm``\ メソッドを呼び出すため、
          \ ``member/register``\ に対してPOSTメソッドでリクエストを送信する。リクエストパラメータには\ ``Form``\ の情報を設定する。
          リクエストデータの設定方法については\ :ref:`UsageOfLibraryForTestSettingOfRequestData`\ を、リクエスト送信の実装方法については
          \ :ref:`UsageOfLibraryForTestExecutionOfRequest`\ を参照されたい。
    * - | (4)
      - | \ ``perform``\ メソッドから返却された\ ``ResultActions``\ の\ ``andExpect``\メソッドで取得した\ ``MvcResult``\ を使用して実行結果の妥当性を検証する。
          検証方法の詳細については\ :ref:`UsageOfLibraryForTestImplementationOfExecutionResultVerification`\ を参照されたい。
    * - | (5)
      - | 不正な入力値を送信する。
    * - | (6)
      - | \ ``SystemExceptionResolver``\ を有効にしていないため、例外ハンドリングされずに\ ``NestedServletException``\ がサーブレットコンテナに通知される。
          \ ``NestedServletException``\ の\ ``getCause``\ メソッドにより取得された例外から、\ ``Controller``\ で期待した例外がthrowされていることを検証する。

|

.. _ImplementsOfTestByLayerTestingControllerWithWebAppContextSetup:

WebAppContextSetupを利用したテスト
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

\ ``Controller``\ の依存クラスが利用できモック化する必要がない場合の\ ``Controller``\ の単体テストにおいて、\ ``WebAppContextSetup``\ で作成するファイルを以下に示す。

.. figure:: ./images/ImplementsOfTestByLayerControllerWebAppContextSetupItems.png

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - 作成するファイル名
      - 説明
    * - \ ``MemberRegisterControllerWebAppContextTest.java``\
      - \ ``MemberRegisterController.java``\ のテストクラス

\ :ref:`ImplementsOfTestByLayerTestingControllerWithStandaloneSetup`\ の例では、パスへのリクエストや\ ``Controller``\ が返す\ ``View``\ 名などは確認できるが、
\ ``TransactionTokenInterceptor``\ や\ ``SystemExceptionResolver``\ といったSpringに追加して利用する機能は適用されていないため、
トランザクショントークンチェックが正しく設定されているか、エラーページへの遷移が正しいかを判断することはできない。
そのような場合は、\ ``MockMvc``\ を\ ``webAppContextSetup``\ でセットアップすることにより、
Springに追加して利用する\ ``Interceptor``\ や\ ``ExceptionResolver``\ などをテスト時に自動で適用させることができる。

ここでは、\ :ref:`ImplementsOfTestByLayerTestingControllerWithStandaloneSetup`\ で説明したテストと、
\ ``@TransactionTokenCheck``\ アノテーション、\ ``SystemExceptionResolver``\ が有効になった場合のテストとを比べた時の相違点について説明する。

以下に、テスト対象となる\ ``Controller``\ の実装例を示す。

* ``MemberRegisterController.java``

.. code-block:: java

    @Controller
    @RequestMapping("member/register")
    @TransactionTokenCheck("member/register")
    public class MemberRegisterController {

        @TransactionTokenCheck(type = TransactionTokenType.BEGIN) // (1)
        @RequestMapping(method = RequestMethod.POST, params = "confirm")
        public String registerConfirm(@Validated MemberRegisterForm memberRegisterForm,
            BindingResult result, Model model) {

            // omitted

            return "C1/memberRegisterConfirm";
        }

        @TransactionTokenCheck(type = TransactionTokenType.IN) // (1)
        @RequestMapping(method = RequestMethod.POST)
        public String register(@Validated MemberRegisterForm memberRegisterForm,
            BindingResult result, Model model, RedirectAttributes redirectAttributes) {

            if (result.hasErrors()) {
                throw new BadRequestException(result); // (2)
            }

            // omitted

            return "redirect:/member/register?complete";
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@TransactionTokenCheck``\ アノテーションを設定することで不正なリクエストを無効にする。
          トランザクショントークンチェックについては、\ :ref:`double-submit_transactiontokencheck`\ を参照されたい。
    * - | (2)
      - | リクエスト時に検証エラーがある場合は改ざんとみなしてエラーをthrowする。

初めに、\ ``@TransactionTokenCheck``\ を有効にした場合におけるテスト作成方法の相違点について説明する。
なお、テストでデータアクセスする場合の検証方法は\ :ref:`ImplementsOfTestByLayerUnitTestOfRepository`\ を参照されたい。

* ``MemberRegisterControllerWebAppContextTest.java``

.. code-block:: java

    import static org.hamcrest.CoreMatchers.*;
    import static org.junit.Assert.*;
    import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
    import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.*;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextHierarchy({@ContextConfiguration(                                   // (1)
            "classpath:META-INF/spring/applicationContext.xml"),                // (1)
            @ContextConfiguration("classpath:META-INF/spring/spring-mvc.xml")}) // (1)
    @WebAppConfiguration                                                        // (1)
    public class MemberRegisterControllerWebAppContextTest {

        @Inject
        WebApplicationContext webApplicationContext; // (2)

        MockMvc mockMvc;

        @Before
        public void setUp() {

            // setup
            mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext) // (2)
                    .alwaysDo(log()).build();
        }

        @Test
        public void testRegisterConfirm01() throws Exception {

            // setup and run the test
            MvcResult mvcResult = mockMvc.perform(post("/member/register") // (3)
                        .param("confirm", "")                              // (3)
                        // omitted
                        .param("password", "testpassword")                 // (3)
                        .param("reEnterPassword", "testpassword"))         // (3)
                        // assert
                        .andExpect(status().is(200))
                        .andExpect(view().name("C1/memberRegisterConfirm"))
                        .andReturn();

            TransactionToken actTransactionToken = (TransactionToken) mvcResult.getRequest()
                    .getAttribute(TransactionTokenInterceptor.NEXT_TOKEN_REQUEST_ATTRIBUTE_NAME); // (4)

            MockHttpSession mockSession = (MockHttpSession) mvcResult.getRequest().getSession();  // (5)

            // setup and run the test
            mockMvc.perform(post("/member/register")              // (6)
                        // omitted
                        .param("password", "testpassword")        // (6)
                        .param("reEnterPassword", "testpassword") // (6)
                        .param(TransactionTokenInterceptor.TOKEN_REQUEST_PARAMETER, 
                                actTransactionToken.getTokenString()) // (6)
                        .session(mockSession)) // (6)
                        // assert
                        .andExpect(status().is(302))                                   // (7)
                        .andExpect(view().name("redirect:/member/register?complete")); // (7)
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | 業務でカスタムした\ ``Interceptor``\ や\ ``ExceptionResolver``\ などを動作させるために\ ``spring-mvc.xml``\ を読み込む。
    * - | (2)
      - | 読み込んだBean定義から生成したWebアプリケーションコンテキストを使用して、\ ``MockMvc``\ をセットアップする。
    * - | (3)
      - | トランザクショントークンを生成するために、\ ``@TransactionTokenCheck(type = TransactionTokenType.BEGIN)``\ が設定された
          メソッドに対してリクエストを送信する。
    * - | (4)
      - | BEGINしたリクエスト（\ ``registerConfirm``\ メソッド）からINのリクエスト（\ ``register``\ メソッド）
          にトランザクショントークンを引き継ぐため、リクエスト属性からトランザクショントークンを取得する。
    * - | (5)
      - | サーバ側は発行したトランザクショントークンをセッションに保持するため、次のリクエストでも同じセッションを参照する必要があるが、
          \ ``MockMvc``\ では１リクエストごとに新規セッションが使われてしまうため、明示的に同じセッションを使用するよう指定する。
    * - | (6)
      - | 再度、リクエストパス（\ ``member/register``\）に対してPOSTメソッドでリクエストを送信する。
          リクエストパラメータには\ ``Form``\ の情報、(4)で取得したトランザクショントークンを設定し、
          セッションには(5)で取得したセッションを設定する。
    * - | (7)
      - | トランザクショントークンチェックの設定が正しいことを確認するために、トークンチェックエラーになっていないことを検証する。

|

次に、\ ``SystemExceptionResolver``\ を有効にした場合におけるテスト作成方法の相違点を説明する。

以下に、\ ``SystemExceptionResolver``\の定義例を示す。

* ``spring-mvc.xml``

.. code-block:: xml

    <bean class="org.terasoluna.gfw.web.exception.SystemExceptionResolver">
      <property name="order" value="3" />
      <property name="exceptionMappings">
        <map>
          <entry key="InvalidTransactionTokenException" value="common/error/token-error" />
          <entry key="BadRequestException" value="common/error/badRequest-error" />
          <entry key="Exception" value="common/error/system-error" />
        </map>
      </property>
      <property name="statusCodes">
        <map>
          <entry key="common/error/token-error" value="409" />
          <entry key="common/error/badRequest-error" value="400" />
        </map>
      </property>
      <property name="excludedExceptions">
          <array>
              <value>org.springframework.web.util.NestedServletException</value>
          </array>
      </property>
      <property name="defaultStatusCode" value="500" />
      <property name="exceptionCodeResolver" ref="exceptionCodeResolver" />
      <property name="preventResponseCaching" value="true" />
    </bean>

以下に、テスト作成方法の相違点について説明する。

* ``MemberRegisterControllerWebAppContextTest.java``

.. code-block:: java

    import static org.hamcrest.CoreMatchers.*;
    import static org.junit.Assert.*;
    import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
    import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.*;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextHierarchy({@ContextConfiguration(
            "classpath:META-INF/spring/applicationContext.xml"),
            @ContextConfiguration("classpath:META-INF/spring/spring-mvc.xml")})
    @WebAppConfiguration
    public class MemberRegisterControllerWebAppContextTest {

        @Inject
        WebApplicationContext webApplicationContext;

        MockMvc mockMvc;

        @Before
        public void setUp() {

            // setup
            mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext)
                    .alwaysDo(log()).build();
        }

        @Test
        public void testRegisterConfirm02() throws Exception {

            // omitted

            // setup and run the test
            mvcResult = mockMvc.perform(post("/member/register")
                        .param("password", "testpassword")
                        .param("reEnterPassword", "") // (1)
                        .param(TransactionTokenInterceptor.TOKEN_REQUEST_PARAMETER, 
                                actTransactionToken.getTokenString()) // (2)
                        .session(mockSession)) // (2)
                        // assert
                        .andExpect(status().is(400))                             // (3)
                        .andExpect(view().name("common/error/badRequest-error")) // (3)
                        .andReturn();

            // assert
            Exception exception = mvcResult.getResolvedException();                   // (4)
            assertThat(exception, is(instanceOf(BadRequestException.class)));         // (4)
            assertThat(exception.getMessage(), is("不正リクエスト(パラメータ改竄)")); // (4)
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | \ ``Form``\ の情報を不正な値にすることで、\ ``register``\ メソッドの内でエラーをthrowさせている。
    * - | (2)
      - | 前述と同様に、生成したトランザクショントークン情報を設定する。
    * - | (3)
      - | ここでは\ ``SystemExceptionResolver``\ が有効になっているため、
          定義したエラーのステータスコード、エラーページの遷移先が正しく設定されていることを検証する。
    * - | (4)
      - | \ ``SystemExceptionResolver``\ で例外ハンドリングされたエラーから、
          期待したエラーがthrowされていることを検証する。

.. note:: **Sessionを利用する場合**

    \ ``Controller``\ クラスがSessionを利用している場合は\ ``org.springframework.mock.web.MockHttpSession``\ を使ってテストを行う。

    * \ ``MockHttpSession``\ を利用したテストメソッドの例

     .. code-block:: java

        public class SessionControllerTest {

            // (1)
            MockHttpSession mockSession = new MockHttpSession();

            // omitted

            @Test
            public void testSession() throws Exception {
                String formName = "todoForm";

                TodoForm form = new TodoForm();
                String todoId = "1111";
                String todoTitle = "test";

                form.setTodoId(todoId);
                form.setTodoTitle(todoTitle);

                // (2)
                mockSession.setAttribute(formName, form);

                // (3)
                ResultActions results = mockMvc.perform(post("/todo/operation")
                    .param("create", "create")
                    .param("todoId", todoId)
                    .param("todoTitle", todoTitle)
                    .session(mockSession));

                // (4)
                results.andExpect(request().sessionAttribute(formName, isA(TodoForm.class)));

                // omitted

                // (5)
                results = mockMvc.perform(get("/todo/create").param("redo", "redo"));
                results.andExpect(request().sessionAttribute(formName, isA(TodoForm.class)));

                // omitted
            }
        }

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
         :header-rows: 1
         :widths: 10 90

         * - 項番
           - 説明
         * - | (1)
           - | セッションのモックオブジェクトを生成する。クラスの詳細については、
               \ `MockHttpSession のJavadoc <https://docs.spring.io/spring-framework/docs/5.1.4.RELEASE/javadoc-api/org/springframework/mock/web/MockHttpSession.html>`_\
               を参照されたい。
         * - | (2)
           - | 生成したセッションのモックオブジェクトに、格納したいオブジェクトをセットする。
         * - | (3)
           - | \ ``MockMvcRequestBuilders``\ の\ ``post``\ メソッドで
               リクエストのモックを生成し、生成したリクエストに\ ``session``\ メソッドでセッションのモックを登録する。
         * - | (4)
           - | (2)でセットしたオブジェクトが、セッションスコープに格納されていることを確認する。
         * - | (5)
           - | 再度リクエストを発行し、セッションスコープに格納したオブジェクトが保持されているか確認する。

|

.. _ImplementsOfTestByLayerTestingControllerWithMockito:

モックを利用したテスト
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

\ ``Controller``\ の依存クラスをモック化する必要がある場合の\ ``Controller``\ の単体テストにおいて、
作成するファイルを以下に示す。

.. figure:: ./images/ImplementsOfTestByLayerControllerMockTest.png

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - 作成するファイル名
      - 説明
    * - \ ``TicketSearchControllerMockTest.java``\
      - \ ``TicketSearchController.java``\ のテストクラス

|

テスト対象の\ ``Controller``\ クラスが依存するクラスを、モック化する場合のテスト作成方法を説明する。

以下に、テスト対象となる\ ``Controller``\ の実装例を示す。

* ``TicketSearchController.java``

.. code-block:: java

    @Controller
    @RequestMapping("ticket/search")
    public class TicketSearchController {

        @Inject
        TicketSearchHelper ticketSearchHelper;

        @RequestMapping(method = RequestMethod.GET, params = "form")
        public String searchForm(Model model) {

            model.addAttribute(ticketSearchHelper.createDefaultTicketSearchForm());

            model.addAttribute(ticketSearchHelper.createFlightSearchOutputDto());

            model.addAttribute("isInitialSearchUnnecessary", true);

            return "B1/flightSearch";
        }
    }

以下に、\ ``Controller``\ のテスト実装例を示す。

* ``TicketSearchControllerMockTest.java``

.. code-block:: java

    import static org.hamcrest.CoreMatchers.*;
    import static org.junit.Assert.*;
    import static org.mockito.Mockito.*;
    import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
    import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.*;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

    public class TicketSearchControllerMockTest {

        @Rule // (1)
        public MockitoRule mockito = MockitoJUnit.rule();

        @InjectMocks // (2)
        TicketSearchController target;

        @Mock // (3)
        TicketSearchHelper ticketSearchHelper;

        MockMvc mockMvc;

        @Before
        public void setUp() {

            // setup
            TicketSearchForm ticketSearchForm = new TicketSearchForm();
            ticketSearchForm.setFlightType(FlightType.RT);
            ticketSearchForm.setDepAirportCd("HND");
            // omitted

            when(ticketSearchHelper.createDefaultTicketSearchForm()).thenReturn(ticketSearchForm); // (4)

            mockMvc = MockMvcBuilders.standaloneSetup(target).alwaysDo(log()).build();
        }

        @Test
        public void testSearchForm() throws Exception {

            // setup and run the test
            MvcResult mvcResult = mockMvc.perform(get("/ticket/search").param("form", ""))
                        // assert
                        .andExpect(status().is(200))
                        .andExpect(view().name("B1/flightSearch"))
                        .andReturn();

            // assert
            verify(ticketSearchHelper).createDefaultTicketSearchForm(); // (5)

            // omitted
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | モックの初期化とインジェクションをアノテーションベースで行うための宣言。
          詳細は\ :ref:`UsageOfLibraryForTestCreateMockObject`\ を参照されたい。
    * - | (2)
      - | \ ``@Mock``\ アノテーションを付与することで、\ ``TicketSearchController``\ が依存している
          \ ``TicketSearchHelper``\ をモック化している。
          詳細は\ :ref:`UsageOfLibraryForTestCreateMockObject`\ を参照されたい。
    * - | (3)
      - | \ ``@InjectMocks``\ アノテーションを付与することで、自動的にモックオブジェクトが代入される。
          詳細は\ :ref:`UsageOfLibraryForTestCreateMockObject`\ を参照されたい。
    * - | (4)
      - | すべてのテストメソッドにおいて、
          \ ``ticketSearchHelper``\ の\ ``createDefaultTicketSearchForm``\ メソッドの返り値として
          \ ``createMockForm``\ メソッドの返り値を設定する。メソッドのモック化については、\ :ref:`UsageOfLibraryForTestMockingMethods`\ を参照されたい。
    * - | (5)
      - | \ ``ticketSearchHelper``\ の\ ``createDefaultTicketSearchForm``\ メソッドについて1回呼び出されたことを検証する。
          モック化したメソッドの検証については、\ :ref:`UsageOfLibraryForTestValidationOfMockMethod`\ を参照されたい。

.. _ImplementsOfTestByLayerUnitTestOfHelper:

Helperの単体テスト
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``Helper``\ の単体テストは、\ ``Service``\ と同様の実装でテストすることができる。
実装方法については、\ :ref:`ImplementsOfTestByLayerUnitTestOfService`\ を参照されたい。
