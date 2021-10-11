
.. _UsageOfLibraryForTest:

単体テストで利用するOSSライブラリの使い方
--------------------------------------------------------------------------------

.. only:: html

 .. contents:: 目次
    :local:

本節では、単体テストで利用するOSSライブラリとして、Spring Test（MockMvc）、Mockitoについて説明する。

.. _UsageOfLibraryForTestSpringTestOverview:

Spring Test
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Test とは
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Testとは、Spring Framework上で動作するアプリケーションのテストを支援するモジュールである。
テストには、対象クラスが依存しているクラスをモックやスタブで代用して行うテストと、SpringのDIコンテナや実際の依存クラスと
組み合わせて行うテストがある。

本ガイドラインでは、モックを使用してテスト対象クラス単体でテストを行う方法と、
設定ファイルや実際の依存クラスを組み合わせてテストを行う方法の2通りの実装方法を例示する。

Spring Testでは主に以下の機能が提供されている。

* テスティングフレームワーク（JUnit）上でSpringのDIコンテナを動かす機能
* テストデータをセットアップする機能
* アプリケーションサーバ上にデプロイせずに、Spring MVCの動作を再現する機能
* テストに最適なトランザクション管理機能

その他、様々なSpring固有のアノテーションや、単体テストで利用するAPIが提供されている。

Spring Testは、テスティングフレームワーク上で動作するテスト用のフレームワーク機能として、
Spring TestContext Frameworkを提供している。

Spring TestContext Frameworkの処理フロー図を以下に示す。

.. figure:: ./images/UsageOfLibraryForTestSpringTestProcessFlow.png
   :width: 95%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - 項番
      - 説明
    * - | (1)
      - | テスト実行により、\ ``org.springframework.test.context.junit4.SpringJUnit4ClassRunner``\ クラスが呼び出される。
    * - | (2)
      - | \ ``SpringJUnit4ClassRunner``\ クラスは\ ``org.springframework.test.context.TestContextManager``\ クラスを
          生成する。
    * - | (3)
      - | \ ``TestContextManager``\ クラスは\ ``org.springframework.test.context.TestContextBootstrapper``\ インタフェース
          の\ ``org.springframework.test.context.TestContext``\ インタフェースのビルド処理を呼び出す。
    * - | (4)
      - | \ ``TestContextBootstrapper``\ クラスはテストクラスで指定された設定ファイルをマージする
          \ ``org.springframework.test.context.MergedContextConfiguration``\ クラスのビルド処理を呼び出す。
        | この時、テストクラスに明示的にブートストラップが指定されていない場合、\ ``@WebAppConfiguration``\ があれば
          \ ``org.springframework.test.context.web.WebTestContextBootstrapper``\ クラス、指定されていなければ
          \ ``org.springframework.test.context.support.DefaultTestContextBootstrapper``\ クラスが呼び出される。
    * - | (5)
      - | \ ``MergedContextConfiguration``\ クラスのビルド処理で\ ``org.springframework.test.context.SmartContextLoader``\ 
          インタフェースの実装クラスが呼び出される。
    * - | (6)
      - | ブートストラップに\ ``WebTestContextBootstrapper``\ クラスが使用されている場合は
          \ ``org.springframework.test.context.web.WebDelegatingSmartContextLoader``\ クラス、
          \ ``DefaultTestContextBootstrapper``\ クラスが使用されている場合は
          \ ``org.springframework.test.context.support.DelegatingSmartContextLoader``\ クラスが\ ``SmartContextLoader``\ 
          インタフェースの実装クラスとして呼び出される。
          \ ``SmartContextLoader``\ インタフェースの実装クラスでテストクラスの\ ``@ContextConfiguration``\ で指定された
          ApplicationContextをロードする。
    * - | (7)
      - | ブートストラップで取得した\ ``MergedContextConfiguration``\ クラスを使用して
          \ ``org.springframework.test.context.TestContext``\ インタフェースの実装クラスである
          \ ``org.springframework.test.context.support.DefaultTestContext``\ クラスを生成する。
    * - | (8)
      - | \ ``TestContextManager``\ クラスにテストクラスの\ ``@TestExecutionListeners``\ で指定された
          \ ``org.springframework.test.context.TestExecutionListener``\ インタフェースを登録し、以下のエントリポイントで
          \ ``TestExecutionListener``\ の処理を呼び出す。

          * テストケースの全テストメソッド実行前（\ ``@BeforeClass``\ ）
          * テストインスタンスの生成後
          * 各テストメソッドの実行前（\ ``@Before``\ ）
          * 各テストメソッドの実行後（\ ``@After``\ ）
          * テストケースの全テストメソッド実行後（\ ``@AfterClass``\ ）

        | トランザクションの管理やテストデータのセットアップ処理は、\ ``TestExecutionListener``\ の処理によって行われる。
        | \ ``TestExecutionListener``\ の登録については\ :ref:`UsageOfLibraryForTestRegistrationOfTestExecutionListener`\ を参照されたい。

|

.. _UsageOfLibraryForTestDIOfSpringTest:

Spring TestのDI機能
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

テストケースの\ ``@ContextConfiguration``\ に設定ファイルを指定すると、\ ``SpringJUnit4ClassRunner``\ にデフォルトで
設定されている\ ``DependencyInjectionTestExecutionListener``\ の処理によってテスト実行時にSpringのDI機能を利用することが
できる。

以下に\ ``@ContextConfiguration``\ を使用して設定ファイルを読み込む例を示す。
ここでは、アプリケーションで使用する\ ``sample-infra.xml``\ を使用してテスト対象の
\ ``com.example.domain.repository.member.MemberRepository``\ をインジェクションしている。

* ``sample-infra.xml``

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:mybatis="http://mybatis.org/schema/mybatis-spring"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="
            http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
            http://mybatis.org/schema/mybatis-spring http://mybatis.org/schema/mybatis-spring.xsd
        ">

      <import resource="classpath:/META-INF/spring/sample-env.xml" />

        <!-- define the SqlSessionFactory -->
        <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
            <property name="dataSource" ref="dataSource" />
            <property name="configLocation" value="classpath:/META-INF/mybatis/mybatis-config.xml" />
        </bean>

        <!-- scan for Mappers -->
        <mybatis:scan base-package="com.example.domain.repository" />

    </beans>


* ``MemberRepositoryTest.java``

.. code-block:: java

    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration(locations = {
            "classpath:META-INF/spring/sample-infra.xml" }) //(1)
    @Transactional
    public class MemberRepositoryTest {

        @Inject
        MemberRepository target; // (2)
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@ContextConfiguration``\ に\ ``sample-infra.xml``\ を指定する。
    * - | (2)
      - | \ ``sample-infra.xml``\ に定義された\ ``<mybatis:scan>``\ でBean登録されている\ ``MemberRepository``\ を
          インジェクションする。


.. _UsageOfLibraryForTestRegistrationOfTestExecutionListener:

TestExecutionListenerの登録
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

テストケースに\ ``@TestExecutionListeners``\ アノテーションを明示的に指定しない場合、Spring Testが提供している以下の
\ ``org.springframework.test.context.TestExecutionListener``\ インタフェースの実装クラスがデフォルトで登録される。

なお、\ ``@TestExecutionListeners``\ アノテーションを明示的に指定しない場合、デフォルトで登録される
\ ``TestExecutionListener``\ はOrderを持っており、呼び出し順は下記表の順に固定されている。
\ ``TestExecutionListener``\ が個別に指定された場合は、指定された順番通りに呼び出される。

.. tabularcolumns:: |p{0.50\linewidth}|p{0.50\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 50 50

    * - TestExecutionListenerの実装クラス
      - 説明
    * - ServletTestExecutionListener
      - \ ``WebApplicationContext``\ のテストをサポートするモックサーブレットAPIを設定する機能を提供している。
    * - DirtiesContextBeforeModesTestExecutionListener
      - テストで使用するDIコンテナのライフサイクル管理機能を提供している。
        テストクラスまたはテストメソッドの実行前に呼び出される。
    * - DependencyInjectionTestExecutionLiLstener
      - テストで使用するインスタンスへのDI機能を提供している。
    * - DirtiesContextTestExecutionListener
      - テストで使用するDIコンテナのライフサイクル管理機能を提供している。
        テストクラスまたはテストメソッドの実行後に呼び出される。
    * - TransactionalTestExecutionListener
      - テスト実行時のトランザクション管理機能を提供している。
    * - SqlScriptsTestExecutionListener
      - \ ``@Sql``\ アノテーションで指定されているSQLを実行する機能を提供している。


各\ ``TestExecutionListener``\ の詳細は\ `Spring Framework Documentation -TestExecutionListener Configuration- <https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/testing.html#testcontext-tel-config>`_\を参照されたい。

\ ``TestExecutionListener``\ は通常、デフォルト設定から変更する必要はないが、テストライブラリが独自に
提供している\ ``TestExecutionListener``\ を使用する場合は\ ``@TestExecutionListeners``\ アノテーションを使用して
\ ``TestContextManager``\ に登録する必要がある。

ここでは例として、Spring Test DBUnitが提供する\ ``TransactionDbUnitTestExecutionListener``\ を登録する方法を説明する。

* ``MemberRepositoryDbunitTest.java``

.. code-block:: java

    @TestExecutionListeners({                               // (1)
            DirtiesContextBeforeModesTestExecutionListener.class,
            DependencyInjectionTestExecutionListener.class,
            DirtiesContextTestExecutionListener.class,
            TransactionDbUnitTestExecutionListener.class})  // (2)
    @Transactional
    public class MemberRepositoryDbunitTest {

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | クラスレベルに\ ``@TestExecutionListeners``\ アノテーションを付けて\ ``TestExecutionListener``\ インタフェース
          の実装クラスを指定することで、テスト実行時に指定した\ ``TestExecutionListener``\ の処理を呼び出すことができる。
          詳細は\ `@TestExecutionListenersのJavadoc <https://docs.spring.io/spring/docs/5.3.2/javadoc-api/org/springframework/test/context/TestExecutionListeners.html>`_\
          を参照されたい。
    * - | (2)
      - | \ ``TransactionDbUnitTestExecutionListener``\ はSpring Test DBUnitが提供する\ ``TestExecutionListener``\ 
          インタフェースの実装クラスである。\ ``@DatabaseSetup``\ や\ ``@ExpectedDatabase``\ 、\ ``@DatabaseTearDown``\
          などのアノテーションを使用したデータのセットアップ、検証、後処理の機能を提供している。
        | \ ``TransactionDbUnitTestExecutionListener``\ は内部で\ ``TransactionalTestExecutionListener``\ と
          \ ``com.github.springtestdbunit.DbUnitTestExecutionListener``\ をチェインしている。


.. warning:: **DbUnitTestExecutionListenerの注意点**

    テストケース内で\ ``@Transactional``\ を指定せずにSpring Test DBUnitの提供する\ ``DbUnitTestExecutionListener``\ を
    使用した場合、\ ``@DatabaseSetup``\ などのアノテーションのトランザクションと、テスト対象クラスのトランザクションは別に
    なるため、データのセットアップが反映されないなど正常に動作しない可能性があることに注意されたい。なお、テストケース内で
    \ ``@Transactional``\ を指定する場合は\ ``DbUnitTestExecutionListener``\の代わりに
    \ ``TransactionDbUnitTestExecutionListener``\ が提供されているため、そちらを使用する必要がある。

|

MockMvc
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ ``MockMvc``\ は、本来Spring Testの機能に含まれるが、
本章ではアプリケーション層の単体テストにおいて使用しているため、
Spring Testの説明と切り出して詳しく説明する。

.. _UsageOfLibraryForTestMockMvcOverview:

MockMvcとは
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Testには、Spring MVC フレームワークと結合した状態でテストするための仕組みとして、
\ ``org.springframework.test.web.servlet.MockMvc``\ クラスを提供している。
\ ``MockMvc``\ を使用すると、アプリケーションサーバ上にデプロイすることなくSpring MVCの動作を再現できるため、
サーバやデータベースを用意する手間を省くことができる。
なお、Spring MVCの詳細については\ :ref:`SpringMVCOverview`\ を参照されたい。

テスト実行時にリクエストを受けてから、レスポンスを返すまでの
\ ``MockMvc``\ の処理フローを、以下の図に示す。

.. figure:: ./images/UsageOfLibraryForTestMockMvcProcessFlow.png
   :width: 95%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | テストメソッドは、Spring Testが用意した\ ``org.springframework.test.web.servlet.TestDispatcherServlet``\ にリクエストするデータをセットアップする。
    * - | (2)
      - | \ ``MockMvc``\ は\ ``TestDispatcherServlet``\ に疑似的なリクエストを行なう。
    * - | (3)
      - | \ ``TestDispatcherServlet``\ は、リクエスト内容に一致する\ ``Controller``\ のメソッドを呼び出す。
    * - | (4)
      - | テストメソッドは、\ ``MockMvc``\ から実行結果を受け取り、実行結果の妥当性を検証する。

|

また、\ ``MockMvc``\ には2つの動作オプションが実装されている。
テストを行う際は、それぞれの特性を把握し、用途毎に適したオプションを選択されたい。

以下に、2つのオプションの概要を示す。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 動作オプション
      - 概要
    * - | webAppContextSetup
      - | \ ``spring-mvc.xml``\などで定義したSpring MVC の設定を読み込み、
          \ ``WebApplicationContext``\ を生成することで、デプロイ時とほぼ同じ状態でテストすることができる。
    * - | standaloneSetup
      - | \ ``Controller``\ にDIされているコンポーネントを、テストで利用する設定ファイルに定義することで、
          Spring Testが生成したDIコンテナを用いてテストを行うことができる。
          よって、Spring MVC のフレームワーク機能を利用しつつ、\ ``Controller``\ のテストを単体テスト観点で行なうことができる。

以下に、2つのオプションのメリット、デメリットを示す。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.45\linewidth}|p{0.45\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 45 45

    * - 動作オプション
      - メリット
      - デメリット
    * - | webAppContextSetup
      - | 実際の稼働で使用する設定ファイルを読み込むことで、
          アプリケーションを動かさなければ確認できないこともデプロイなしで検証することができる。
          実際の設定ファイルを読み込みテストするため、設定ファイルが正しく作成されているかを確認することもできる。
        | また、Bean定義にモッククラスを指定しておけば、\ ``Controller``\ にDIされる\ ``Service``\ などをモック化することも可能である。
      - | 巨大なアプリケーションをテストする場合や、膨大なBean定義を読み込む場合は実行に時間がかかってしまう。
          そのため、デプロイする場合の設定ファイルから、必要な記述だけを抽出した設定ファイルを用意するなどの工夫が必要となる。
    * - | standaloneSetup
      - | 生成されるDIコンテナに特定の\ ``Interceptor``\ や\ ``Resolver``\ 等を適用してテストを実施できる。
          そのため、Springの設定ファイルを参照せずコントローラ単体だけ見たい場合は、\ ``webAppContextSetup``\ よりも実施コストが低い。
      - | \ ``Interceptor``\ や\ ``Resolver``\などを多く適用するテストにおける設定コストが高い。
        | また、あくまで\ ``Contoroller``\ の単体テスト観点で動作するため、
          Spring MVC のフレームワーク機能と合わせて\ ``Controller``\ のテストを行いたい場合は、
          \ ``webAppContextSetup``\ でのテストを検討する必要があることに留意されたい。

.. _UsageOfLibraryForTestSettingMockMvc:

MockMvcのセットアップ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ここでは\ ``MockMvc``\ の2つのオプションについて、
実際にテストで使用する際のセットアップ方法を説明する。

.. _UsageOfLibraryForTestSettingMockMvcWithWebAppContextSetup:

webAppContextSetupによるセットアップ
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

ここでは、\ ``webAppContextSetup``\ でテストを行うためのセットアップ方法について説明する。

MockMvcのセットアップ設定例を以下に示す。

* MockMvcのセットアップ設定例

.. code-block:: java

    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextHierarchy({ @ContextConfiguration({ // (1)
            "classpath:META-INF/spring/applicationContext.xml",
            "classpath:META-INF/spring/spring-security.xml" }),
            @ContextConfiguration("classpath:META-INF/spring/spring-mvc.xml") })
    @WebAppConfiguration // (2)
    public class MemberRegisterControllerWebAppContextTest {

        @Inject
        WebApplicationContext webApplicationContext; // (3)

        MockMvc mockMvc;

        @Before
        public void setUp() {

            mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext) // (4)
                    .alwaysDo(log()).build();
        }

        @Test
        public void testRegisterConfirm01() throws Exception {

            ResultActions results = mockMvc.perform(post("/member/register")
                    // omitted
                    .param("confirm", "");

            results.andExpect(status().is(200));

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
      - | テスト実行時に生成するDIコンテナの設定ファイルを指定する。
          DIコンテナの階層関係については、\ ``@org.springframework.test.context.ContextHierarchy``\
          を使うことで再現することができる。
          DIコンテナの階層関係については\ :ref:`CreateWebApplicationProjectAppendixApplicationContext`\ を参照されたい。
    * - | (2)
      - | Webアプリケーション向けのDIコンテナ（\ ``WebApplicationContext``\）が作成できるようになる。
          また、\ ``@WebAppConfiguration``\ を指定すると開発プロジェクト内の\ ``src/main/webapp``\ がWebアプリケーションのルートディレクトリ
          になるが、これはMavenの標準構成と同じなので特別に設定を加える必要はない。
    * - | (3)
      - | テスト実行時に使用するDIコンテナをインジェクションする。
    * - | (4)
      - | テスト実行時に使用するDIコンテナを指定して、MockMvcを生成する。


standaloneSetupによるセットアップ
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

ここでは、\ ``standaloneSetup``\ でテストを行うためのセットアップ方法について説明する。

MockMvcのセットアップ設定例を以下に示す。

* MockMvcのセットアップ設定例

.. code-block:: java

    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration(locations = {
            "classpath:META-INF/spring/applicationContext.xml",
            "classpath:META-INF/spring/test-context.xml",
            "classpath:META-INF/spring/spring-mvc-test.xml"})
    public class MemberRegisterControllerStandaloneTest {

        @Inject
        MemberRegisterController target;

        MockMvc mockMvc;

        @Before
        public void setUp() {
            mockMvc = MockMvcBuilders.standaloneSetup(target).alwaysDo(log()).build(); // (1)
        }

        @Test
        public void testRegisterConfirm01() throws Exception {

            ResultActions results = mockMvc.perform(post("/member/register")
                    // omitted
                    .param("password", "testpassword")
                    .param("reEnterPassword", "testpassword"));

            results.andExpect(status().is(200));

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
      - | テスト対象の\ ``Controller``\ を指定して、MockMvcを生成する。
          必要に応じて\ ``org.springframework.test.web.servlet.setup.StandaloneMockMvcBuilder``\ のメソッドを呼び出して、
          Spring Testが生成するDIコンテナをカスタマイズすることができる。
          カスタマイズするためのメソッドについての詳細は、
          \ `StandaloneMockMvcBuilderのJavadoc <https://docs.spring.io/spring/docs/5.3.2/javadoc-api/org/springframework/test/web/servlet/setup/StandaloneMockMvcBuilder.html>`_\
          を参照されたい。

MockMvcによるテストの実装
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ここではMockMvcによるテスト実行の流れとして、リクエストデータの設定から、
リクエスト送信の実装方法、実行結果の検証、出力まで説明する。

.. _UsageOfLibraryForTestSettingOfRequestData:

リクエストデータの設定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

リクエストデータの設定は、\ ``org.springframework.test.web.servlet.request.MockHttpServletRequestBuilder``\ や
\ ``org.springframework.test.web.servlet.request.MockMultipartHttpServletRequestBuilder``\ のファクトリメソッドを使用して行う。

ここでは、2つのクラスのファクトリメソッドの中から主要なメソッドについて紹介する。
詳細は、\ `MockHttpServletRequestBuilder のJavadoc <https://docs.spring.io/spring/docs/5.3.2/javadoc-api/org/springframework/test/web/servlet/request/MockHttpServletRequestBuilder.html>`_\
または\ `MockMultipartHttpServletRequestBuilder のJavadoc <https://docs.spring.io/spring/docs/5.3.2/javadoc-api/org/springframework/test/web/servlet/request/MockMultipartHttpServletRequestBuilder.html>`_\ を参照されたい。

\

.. tabularcolumns:: |p{0.15\linewidth}|p{0.85\linewidth}|
.. list-table:: **MockHttpServletRequestBuilderの主なメソッド**
    :header-rows: 1
    :widths: 15 85

    * - メソッド名
      - 説明
    * - \ ``param``\ / \ ``params``\
      - テスト実行時のリクエストに、リクエストパラメータを追加するメソッド。
    * - \ ``content``\
      - テスト実行時のリクエストに、リクエストボディを追加するメソッド。
    * - \ ``header``\ / \ ``headers``\
      - テスト実行時のリクエストに、リクエストヘッダーを追加するメソッド。
        \ ``contentType``\ や\ ``accept``\ などの特定のヘッダーを指定するためのメソッドも提供されている。
    * - \ ``requestAttr``\
      - リクエストスコープにオブジェクトを設定するメソッド。
    * - \ ``flashAttr``\
      - フラッシュスコープにオブジェクトを設定するメソッド。
    * - \ ``sessionAttr``\
      - セッションスコープにオブジェクトを設定するメソッド。
    * - \ ``cookie``\
      - テスト実行時のリクエストに、指定したcookieを追加するメソッド。

\

.. tabularcolumns:: |p{0.15\linewidth}|p{0.85\linewidth}|
.. list-table:: **MockMultipartHttpServletRequestBuilderの主なメソッド**
    :header-rows: 1
    :widths: 15 85

    * - メソッド名
      - 説明
    * - \ ``file``\
      - テスト実行時のリクエストに、アップロードするファイルを設定するメソッド。
    * - \ ``part``\
      - テスト実行時のリクエストに、 \ ``multipart/form-data``\ のPOSTリクエストで受信されたパーツまたはフォームアイテムを追加するメソッド。
        このメソッドを利用することで、テスト実行時のリクエストにファイル以外のリクエストパラメータを追加することができる。

ここでは、\ ``param``\ メソッドを用いたリクエストデータの設定と、
\ ``post``\ メソッドを用いたリクエスト実行の例を示す。

以下に、テスト対象の\ ``Controller``\ の実装を示す。

* テスト対象の\ ``Controller``\ クラス

.. code-block:: java

    @Controller
    @RequestMapping("member/register")
    @TransactionTokenCheck("member/register")
    public class MemberRegisterController {

        @RequestMapping(method = RequestMethod.POST, params = "confirm")
        public String registerConfirm(@Validated MemberRegisterForm memberRegisterForm,
            BindingResult result, Model model) {

            // omitted

            return "C1/memberRegisterConfirm";
        }
    }

以下に、リクエスト送信の実装例を示す。

* リクエスト送信の実装例

.. code-block:: java

    @Test
    public void testRegisterConfirm01() throws Exception {
        mockMvc.perform(
             post("/member/register")
            .param("confirm", "")); // (1)
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``confirm``\ をリクエストパラメータに持つリクエストデータを設定している。

.. _UsageOfLibraryForTestExecutionOfRequest:

リクエスト送信の実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

設定したリクエストデータを\ ``MockMvc``\ の\ ``perform``\ メソッドの引数として渡すことで、
テストで利用するリクエストデータを設定し、\ ``DispatcherServlet``\ に疑似的なリクエストを行なう。
\ ``MockMvcRequestBuilders``\ のメソッドには、\ ``get``\ 、\ ``post``\ 、\ ``fileUpload``\ といったメソッドが、リクエストの種類ごとに提供されている。
詳細は、\ `MockMvcRequestBuilders のJavadoc <https://docs.spring.io/spring/docs/5.3.2/javadoc-api/org/springframework/test/web/servlet/request/MockMvcRequestBuilders.html>`_\ を参照されたい。

以下に、リクエスト送信の実装例を示す。

* リクエスト送信の実装例

.. code-block:: java

    @Test
    public void testRegisterConfirm01() throws Exception {
        mockMvc.perform( // (1)
             post("/member/register") // (2)
            .param("confirm", ""));
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | リクエストを実行し、返り値として実行結果の検証を行うための\ ``ResultActions``\ クラスを返す。
          詳細は後述の\ :ref:`UsageOfLibraryForTestImplementationOfExecutionResultVerification`\ を参照されたい。
    * - | (2)
      - | \ ``/member/register``\ へPOSTリクエストを実行するように設定している。

.. warning:: **テスト時のトランザクショントークンチェック、CSRFチェック**

    テスト対象がトランザクショントークンチェックやCSRFチェックを利用している場合は、
    \ ``mockMvc``\ のリクエストについてもチェックが適用されることに注意されたい。
    なお、本章では\ ``spring-security``\ の設定は無効にしているため、CSRFチェックは行われていない。

.. note :: **"/"から始まらないパスへリクエストを送信する際の挙動**

    \ ``MockMvcRequestBuilders``\ の呼び出す\ ``UriComponentBuilder``\ は仕様上、スキームまたはパスから始める必要がある。そのため、Spring Framework 5.2.3以前では"/"から始まらないパスへリクエストを送る場合\ ``404 Not Found``\が発生していたが、
    Spring Framework 5.2.4からはスキームが正しいことをアサートする処理が追加されたためアサーションエラーが発生するようになった。

.. _UsageOfLibraryForTestImplementationOfExecutionResultVerification:

実行結果検証の実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

実行結果の検証には、\ ``org.springframework.test.web.servlet.ResultActions``\ の\ ``andExpect``\ メソッドを使用する。
\ ``andExpect``\ メソッドの引数には\ ``org.springframework.test.web.servlet.ResultMatcher``\ を指定する。
Spring Testは、\ ``org.springframework.test.web.servlet.result.MockMvcResultMatchers``\ のファクトリメソッドを介して
さまざまな\ ``ResultMatcher``\ を提供している。

ここでは、\ ``andExpect``\ メソッドの引数として、主要となる\ ``MockMvcResultMatchers``\ のメソッドを紹介する。
ここで紹介しないメソッドについては、
\ `MockMvcResultMatchers のJavadoc <https://docs.spring.io/spring/docs/5.3.2/javadoc-api/org/springframework/test/web/servlet/result/MockMvcResultMatchers.html>`_\ を参照されたい。

\

.. tabularcolumns:: |p{0.15\linewidth}|p{0.85\linewidth}|
.. list-table:: **MockMvcResultMatchersの主なメソッド**
    :header-rows: 1
    :widths: 15 85

    * - メソッド名
      - 説明
    * - \ ``status``\
      - HTTPステータスコードを検証するメソッド。
    * - \ ``view``\
      - \ ``Controller``\ が返却したView名を検証するメソッド。
    * - \ ``model``\
      - Spring MVCの\ ``Model``\ について検証するメソッド
    * - \ ``request``\
      - リクエストスコープおよびセッションスコープの状態、
        Servlet 3.0からサポートされている非同期処理の処理状態を検証するメソッド。
    * - \ ``flash``\
      - フラッシュスコープの状態を検証するメソッド。
    * - \ ``redirectedUrl``\
      - リダイレクト先のパスを検証するメソッド。
        \ ``redirectedUrlPattern``\ メソッドを用いたパターンによる検証も提供されている。
    * - \ ``fowardedUrl``\
      - フォワード先のパスを検証するメソッド。
        \ ``forwardedUrlPattern``\ メソッドを用いたパターンによる検証も提供されている。
    * - \ ``content``\
      - レスポンスボディの中身を検証するメソッド。
        jsonPathやxPathなどの特定のコンテンツ向けのメソッドも提供されている。
    * - \ ``header``\
      - レスポンスヘッダーの状態を検証するメソッド。
    * - \ ``cookie``\
      - cookieの状態を検証するメソッド。

以下に、テストの実行結果検証の実装例を示す。

* 実行結果検証の実装例

.. code-block:: java

    @Test
    public void testRegisterConfirm01() throws Exception {
        mockMvc.perform(post("/member/register")
            .param("confirm", ""));
            .andExpect(status().is(302)) // (1)
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | テスト実行時のリクエストデータを設定している。
          \ ``andExpect``\ メソッドは\ ``ResultActions``\ からチェーンして記述することができるため、
          IDEの補完機能によってコーディングの負担を減らすことができる。

.. warning:: **Modelの検証とアサーションライブラリ**

    Spring Testでは\ ``Model``\ の検証として、\ ``model``\メソッドにチェーンする形で
    \ ``org.springframework.test.web.servlet.result.ModelResultMatchers``\ の\ ``attribute``\ メソッドを使用することが
    できる。このメソッドを用いることで\ ``Model``\ の中身を検証することができるが、引数としてHamcrestの
    \ ``org.hamcrest.Matcher``\ を使用するため、Hamcrest以外のアサーションライブラリを使用する場合は注意されたい。

    Hamcrest以外のアサーションライブラリを併用する場合は、\ ``MvcResult``\ から\ ``ModelAndView``\ オブジェクトを取得し、
    さらに\ ``ModelAndView``\ オブジェクトから\ ``Model``\に格納されたオブジェクトを取得することで、使用している
    アサーションライブラリを使って\ ``Model``\ を検証することができる。
    
    以下に\ ``ModelAndView``\ オブジェクトから取得した\ ``Model``\ の検証例を示す。

     .. code-block:: java

         @Test
         public void testRegisterConfirm01() throws Exception {
             MvcResult mvcResult = mockMvc.perform(post("/member/register").param("confirm", "")
                         .param("kanjiFamilyName", "電電")
                         .andExpect(status().is(200))
                         .andReturn(); // (1)

             ModelAndView mav = mvcResult.getModelAndView(); // (2)

             MemberRegisterForm actForm = (MemberRegisterForm) mav.getModel().get("memberRegisterForm");

             assertThat(actForm.getKanjiFamilyName(), is("電電"));
             // omitted
         }

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
         :header-rows: 1
         :widths: 10 90

         * - 項番
           - 説明
         * - | (1)
           - | \ ``ResultActions``\ の\ ``andReturn``\メソッドを使用して \ ``MvcResult``\ オブジェクトを取得する。
         * - | (2)
           - | \ ``MvcResult``\ から\ ``ModelAndView``\ オブジェクトを取得し、\ ``ModelAndView``\ オブジェクトから
               \ ``Model``\ に格納されたオブジェクトを取得して\ ``Model``\ の検証を行う。

実行結果出力の実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

テスト実行時のログ出力などを有効化する場合は、\ ``ResultActions``\ の\ ``alwaysDo``\ メソッドや\ ``andDo``\ メソッドを使う。
ログの出力などは共通処理になる場合が多いため、
\ ``MockMvc``\ 生成時に\ ``StandaloneMockMvcBuilder``\ の\ ``alwaysDo``\ メソッドを使うことを推奨する。

\ ``alwaysDo``\ メソッドの引数には、実行結果に対して任意の処理を行なう\ ``org.springframework.test.web.servlet.ResultHandler``\
を指定する。Spring Testでは、\ ``org.springframework.test.web.servlet.result.MockMvcResultHandlers``\ のファクトリメソッドを介して
さまざまな\ ``ResultHandler``\ を提供している。
ここでは、\ ``alwaysDo``\ メソッドの引数として主要となる\ ``MockMvcResultHandlers``\ のメソッドを紹介する。
各メソッドの詳細については、
\ `MockMvcResultHandlers のJavadoc <https://docs.spring.io/spring/docs/5.3.2/javadoc-api/org/springframework/test/web/servlet/result/MockMvcResultHandlers.html>`_\ を参照されたい。

\

.. tabularcolumns:: |p{0.15\linewidth}|p{0.85\linewidth}|
.. list-table:: **MockMvcResultHandlersの主なメソッド**
    :header-rows: 1
    :widths: 15 85

    * - メソッド名
      - 説明
    * - \ ``log``\
      - 実行結果をデバッグレベルでログ出力するメソッド。
        ログ出力時に使用されるロガー名は\ ``org.springframework.test.web.servlet.result``\ である。
    * - \ ``print``\
      - 実行結果を任意の出力先に出力するメソッド。出力先を指定しない場合、標準出力が出力先になる。

以下に、テストの実行結果出力の設定例を示す。

* 実行結果出力の設定例

.. code-block:: java

    @Before
    public void setUp() {
        mockMvc = MockMvcBuilders.standaloneSetup(target).alwaysDo(log()).build(); // (1)
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``alwayDo``\ メソッドの引数に\ ``log``\ メソッドを指定することで、
          \ ``mockMvc``\ を用いたテスト実行の際は、常に実行結果をログとして出力する。

.. note :: **テストケースごとの出力設定**

    テスト実行時のログ出力などをテストケースごとに有効化する場合は、\ ``ResultActions``\ の\ ``andDo``\ メソッドを使う。
    \ ``andDo``\ メソッドも\ ``alwaysDo``\ メソッドと同じく引数に\ ``ResultHandler``\ を指定する。

    以下に、ログ出力をテストケースごとに有効化する場合の設定例を示す。

    * ログ出力を常に有効化する場合の設定例

     .. code-block:: java

         @Test
         public void testSearchForm() throws Exception {
             mockMvc.perform(get("/ticket/search").param("form", ""))
                     .andDo(log()); // (1)
         }

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
         :header-rows: 1
         :widths: 10 90

         * - 項番
           - 説明
         * - | (1)
           - | テストの実行結果をログ出力する。

|

Mockito
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ここでは、モックの概要、Mockitoの使い方について説明する。

Mockitoとは
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Mockitoとは、単体テストにおいてテスト対象の依存クラスをモック化する際に使われる
モックライブラリの一つである。
モックライブラリを利用することで、モックの生成を簡単に行なうことができるため、
単体テストの実装時にはよく利用されている。

ここからは、モックについての説明を行なう。

モックの概要
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

モックとは、テスト対象が依存するクラスの代用となる疑似クラスである。
このように依存するクラスをモックに置き換えることをモック化と呼ぶ。
モックは単体テストにおいて、依存クラスが正しく使用されているかを検証するために使用される。
テスト対象のクラスのみ着目してテストを行いたい場合や、テスト対象の依存クラスが完成していないときは、
依存クラスをモック化してテストを行なうことを考えるべきである。

以下に、モックを利用したテストのフロー図を示す。

.. figure:: ./images/UsageOfLibraryForTestMockCreationOfMockito.png
   :width: 50%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 依存クラスが作成され、動作も保障されている場合は、
          そのまま依存クラスを用いてテストすればよい。
    * - | (2)
      - | 依存クラスの動作が保障されていない場合や、作成されていない場合など、
          依存クラスを利用できない場合は、モッククラスを用いてテストする。

実際に依存クラスをモック化する場合、通常テスト実施者自身でモッククラスを用意する必要がある。
しかし、検証のためのクラスをテストごとに一から作成していては、
テスト実施に多大なコストがかかると予想される。
そのような場合に利用されるのが、モック作成のためのモックライブラリである。
モックライブラリを用いることで、より簡単にモックを作成することができるため、
モックを利用するときはモックライブラリの使用を推奨する。

本章では、代表的なモックライブラリとしてMockitoを使用し説明を行なう。

Mockitoの利用
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Mockitoは、依存クラスのモック化、メソッドの呼び出し検証、
メソッドの引数検証など、テストを行なう上で必要となる機能を提供している。
しかし、テスト対象のコードによってはMockitoを利用できない場合もあるので注意されたい。

ここでは、Mockitoを利用できない状況の中でも、
特に注意が必要となる状況について紹介する。

モック化の制限
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Mockitoでは、\ ``final``\ 宣言、\ ``private``\ 宣言されたクラス/メソッド、
\ ``static``\ 宣言されたメソッドをモック化することができない。
通常のオブジェクト指向に沿った実装であれば、モック化に制限のかかるようなテストになることは考えにくいため、
このような事態に直面した場合はテスト対象の設計に問題がないか一度確認したほうがよい。

その他のMockitoの制限については、
\ `What are the limitations of Mockito <https://github.com/mockito/mockito/wiki/FAQ#what-are-the-limitations-of-mockito>`_\
を参照されたい。

Mockitoの機能
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ここでは、Mockitoの代表的な機能として、モックの作成、
モック化されたメソッドの定義、検証について紹介する。

.. _UsageOfLibraryForTestCreateMockObject:

モックの生成
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Mockitoのモック化には2種類の方法が存在する。
1つは、\ ``mock``\メソッドを用いて依存クラスをすべてモックにする方法、
もう1つは、\ ``spy``\ メソッドを用いて依存クラスの一部のメソッドのみをモックにする方法である。

ここではより単純な、依存クラスをすべてモック化する方法について紹介する。
依存クラスの一部のみをモックにする方法については、
\ `MockitoのJavadoc <https://javadoc.io/doc/org.mockito/mockito-core/3.6.28/org/mockito/Mockito.html#13>`__\
を参照されたい。

完全にモック化する場合、基本的には\ ``mock``\ メソッドを用いてモック化する。

以下に、\ ``mock``\ メソッドを用いたモック化の例を示す。

.. code-block:: java

    public class TicketReserveServiceImpl implements TicketReserveService {

        @Inject // (1)
        ReservationRepository reservationRepository;

        // omitted
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | テスト対象の\ ``TicketReserveServiceImpl``\ は
          \ ``ReservationRepository``\ をインジェクトしているため、
          \ ``ReservationRepository``\ に依存した実装となっている。

.. code-block:: java

    public class TicketReserveServiceImplMockTest {

        ReservationRepository mockReservationRepository = mock(ReservationRepository.class); // (1)

        private TicketReserveServiceImpl target;

        @Test
        public void testRegisterReservation() {

            target.reservationRepository = mockReservationRepository; // (2)

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
      - | \ ``mock``\ メソッドを使うことで、\ ``TicketReserveServiceImpl``\ が依存していた
          \ ``ReservationRepository``\ をモック化している。
    * - | (2)
      - | テスト対象のフィールドにモッククラスのオブジェクトを適用する。

|

\ ``mock``\ メソッドを使用すると、テスト実施者自身が1つずつモックを適用していく必要があった。
そのため、テスト対象の依存クラスが数多く存在する場合は、記述量も増加し実装コストもかかる。
そのような場合は、\ ``@Mock``\ アノテーションを使用したモックを自動で適用させる方法を推奨する。
また、本章ではモック化の際に、\ ``mock``\ メソッドより簡潔に記述できる\ ``@Mock``\ アノテーションを使用している。

以下に、\ ``@Mock``\ アノテーションを用いたモック化の例を示す。

* ``TicketReserveServiceImplMockTest.java``

.. code-block:: java

    public class TicketReserveServiceImplMockTest {

        @Mock // (1)
        ReservationRepository mockReservationRepository;

        @InjectMocks // (2)
        private TicketReserveServiceImpl target;

        @Rule // (3)
        public MockitoRule mockito = MockitoJUnit.rule();

        // omitted
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@Mock``\ アノテーションをモック化したいクラスに付与することで、対象クラスのモックオブジェクトが
          Mockitoによって自動的に代入される。モッククラスを別途定義する必要はない。
    * - | (2)
      - | \ ``@InjectMocks``\ アノテーションをテスト対象としたい具象クラスに付与することで、対象クラスのインスタンスが
          Mockitoによって自動的に代入され、さらに対象クラスが依存しているクラスと、
          \ ``@Mock``\ アノテーションが付与されたクラスが一致する場合、自動的にモックオブジェクトが設定される。
    * - | (3)
      - | JUnitでMockitoを利用するための宣言。
          \ ``@Rule``\ により、後述のアノテーションベースのモックオブジェクトの初期化機能が利用可能になる。

.. _UsageOfLibraryForTestMockingMethods:

メソッドのモック化
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

モック化したオブジェクトの持つすべてのメソッドは、
返り値がプリミティブ型の場合はそれぞれの型の初期値(例: int型の場合は0)を、
それ以外の場合は\ ``null``\ を返すようなメソッドとして定義される。
そのため、テストを行なう際は実施するテスト内容に合わせて、
メソッドの返り値を改めて定義する必要がある。

引数、返り値の設定
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

メソッドのモック化には、\ ``Mockito``\ クラスの\ ``when``\ メソッドと、
\ ``when``\ メソッドが返す \ ``org.mockito.stubbing.OngoingStubbing``\ インスタンスのメソッドを使用する。
\ ``when``\ メソッドの引数にはモック化するメソッドとその引数を指定し、実行時の返り値を\ ``OngoingStubbing``\ のメソッドで定義する。

以下に、\ ``OngoingStubbing``\ の主なメソッドを示す。
\ ``OngoingStubbing``\ の詳細については、
\ `OngoingStubbingのJavadoc <https://javadoc.io/doc/org.mockito/mockito-core/3.6.28/org/mockito/stubbing/OngoingStubbing.html>`_\ を、
また\ ``when``\ メソッドについては
\ `MockitoのJavadoc <https://javadoc.io/doc/org.mockito/mockito-core/3.6.28/org/mockito/Mockito.html>`__\ を参照されたい。

.. tabularcolumns:: |p{0.15\linewidth}|p{0.85\linewidth}|
.. list-table:: **OngoingStubbingの主なメソッド**
    :header-rows: 1
    :widths: 15 85

    * - メソッド名
      - 説明
    * - \ ``thenReturn``\
      - メソッドが呼び出されるときの返り値を引数に設定するメソッド。
    * - \ ``thenThrow``\
      - メソッドが呼び出されるときにthrowされる\ ``Throwable``\ オブジェクトを引数に設定するメソッド。
        引数にはthrowしたい例外クラスの\ ``java.lang.Class``\ オブジェクトを指定することもできる。

以下に、\ ``thenReturn``\ の使用例を示す。

.. code-block:: java

    public class TicketReserveServiceImplMockTest {

        @Test
        public void testRegisterReservation() {

            Reservation testReservation = new Reservation();
            reservation.setReserveNo("0000000001");

            when(reservationRepository.insert(testReservation)).thenReturn(1); // (2)
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``when``\ メソッドの引数には、動作を定義したいメソッドとその引数を指定する。
    * - | (2)
      - | \ ``insert``\ メソッドの引数に\ ``testReservation``\ を指定することで、
          テスト対象が\ ``insert``\ メソッドを引数\ ``testReservation``\ で実行するとき、
          返り値は"\ ``1``\" になる。

任意の引数による定義
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

モック化したいメソッドの引数に\ ``org.mockito.ArgumentMatchers``\ のメソッドを用いることで、
任意の引数を対象に返り値を定義することもできる。

以下に、\ ``ArgumentMatchers``\ の主なメソッドを示す。
詳細については、\ `ArgumentMatchersのJavadoc <https://javadoc.io/doc/org.mockito/mockito-core/3.6.28/org/mockito/ArgumentMatchers.html>`_\ を参照されたい。

.. tabularcolumns:: |p{0.15\linewidth}|p{0.85\linewidth}|
.. list-table:: **ArgumentMatchersの主なメソッド**
    :header-rows: 1
    :widths: 15 85

    * - メソッド名
      - 説明
    * - \ ``any``\
      - モック化されるメソッドの引数が任意の型の\ ``Object``\ であることを示すメソッド。
    * - \ ``anyString``\
      - モック化されるメソッドの引数が\ ``null``\ 以外の任意の\ ``String``\ であることを示すメソッド。
    * - \ ``anyInt``\
      - モック化されるメソッドの引数が任意のint型、または\ ``null``\ 以外の任意の\ ``Integer``\ であることを示すメソッド。

以下に、\ ``any``\ の使用例を示す。

.. code-block:: java

    public class TicketReserveServiceImplMockTest {

        @Test
        public void testRegisterReservation() {

            // omitted

            when(reservationRepository.insert(any(Reservation.class))).thenReturn(0); // (1)
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``insert``\ メソッドの引数として
          \ ``any``\ メソッドで\ ``Reservation``\クラスを指定することで、
          \ ``insert``\ メソッドを任意の\ ``Reservation``\ 引数で実行するとき、
          返り値が"\ ``0``\" になるように設定している。

返り値がvoid型であるメソッドのモック化
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

モック化された返り値が\ ``void``\ 型であるメソッドは、デフォルトでは何も動作しないメソッドとして定義される。
そのため、例外をスローさせたい場合などは改めて定義する必要がある。

動作の設定
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

\ ``when``\ メソッドでは定義できない返り値が\ ``void``\ 型であるメソッドについては、
\ ``Mockito``\ クラスの\ ``doThrow``\ メソッドなどを用いることで定義できる。

以下に、返り値が\ ``void``\ 型であるメソッドを再定義するための\ ``Mockito``\ の主なメソッドを示す。
詳細については、
\ `MockitoのJavadoc <https://javadoc.io/doc/org.mockito/mockito-core/3.6.28/org/mockito/Mockito.html>`__\
を参照されたい。

.. tabularcolumns:: |p{0.15\linewidth}|p{0.85\linewidth}|
.. list-table:: **返り値がvoid型であるメソッドを再定義するMockitoの主なメソッド**
    :header-rows: 1
    :widths: 15 85

    * - メソッド名
      - 説明
    * - \ ``doThrow``\
      - 返り値が\ ``void``\ 型であるメソッドにthrowさせる例外を設定する場合に用いるメソッド。
    * - \ ``doNothing``\
      - 返り値が\ ``void``\ 型であるメソッドに何もさせないよう設定する場合に用いるメソッド。

以下に、\ ``doThrow``\ の使用例を示す。

.. code-block:: java

    doThrow(new RuntimeException()).when(mock).someVoidMethod(); // (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``doThrow``\ メソッドは\ ``when``\ メソッドの前に記述し、
          \ ``when``\ メソッドはその後、チェーンする形で記載する。
        | \ ``doThrow``\ メソッドの引数にスローしたい例外を指定することで、
          モック化したメソッドが実行されるときに例外をスローするようになる。

.. _UsageOfLibraryForTestValidationOfMockMethod:

モック化したメソッドの検証
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Mockitoで作成したオブジェクトをモックとして用いる場合は、
\ ``Mockito``\ クラスの\ ``verify``\ メソッドを用いることで、
モック化したメソッドの呼び出しについて検証することができる。

\ ``verify``\ メソッドは引数にモックを指定し、チェーンする形で
モック化したメソッドを続けることで、そのメソッドが正しく呼ばれているかどうかを検証できる。
また、\ ``verify``\ メソッドの引数としてモックと\ ``org.mockito.verification.VerificationMode``\
を指定することで、より詳しくメソッドの呼び出しについて検証できる。

以下に、\ ``VerificationMode``\ の主なメソッドを示す。
詳細については、\ `VerificationModeのJavadoc <https://javadoc.io/doc/org.mockito/mockito-core/3.6.28/org/mockito/verification/VerificationMode.html>`_\ を参照されたい。

.. tabularcolumns:: |p{0.15\linewidth}|p{0.85\linewidth}|
.. list-table:: **VerificationModeの主なメソッド**
    :header-rows: 1
    :widths: 15 85

    * - メソッド名
      - 説明
    * - \ ``times``\
      - 期待する呼び出し回数を設定するメソッド。
        引数に期待する呼び出し回数を設定できる。
        \ ``verify``\ メソッドの引数に\ ``VerificationMode``\を指定しない場合は
        \ ``times(1)``\ が設定される。
    * - \ ``never``\
      - 呼び出されていないことを期待する場合に設定するメソッド。

以下に、\ ``times``\ の使用例を示す。

.. code-block:: java

    @Test
    public void testRegisterReservation() {

        // omitted

        when(reservationRepository.insert(testReservation)).thenReturn(1);

        TicketReserveDto ticketReserveDto = target.registerReservation(testReservation); // (1)

        verify(reservationRepository, times(1)).insert(testReservation); // (2)
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``target``\ の\ ``insert``\ メソッドでは、
          \ ``ReservationRepository``\ の\ ``insert``\ メソッドが1回実行されるような実装になっている。
    * - | (2)
      - | \ ``verify``\ メソッドの引数にモックオブジェクトと、
          \ ``times``\ メソッドを指定することで、\ ``insert``\ メソッドが引数\ ``testReservation``\ で
          正しく1回呼ばれているかを検証することができる。
        | この場合は、\ ``times``\ メソッドの引数が"\ ``1``\" なので省略しても同様の検証となる。

