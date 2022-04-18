
.. _ImplementsOfTestByFunction:

機能ごとのテスト実装
--------------------------------------------------------------------------------

.. only:: html

 .. contents:: 目次
    :local:

本節では、レイヤ単位に当てはめられない機能のテスト方法ついて説明する。

|

入力チェックの単体テスト
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ここでは、以下の入力チェックの単体テスト実装方法を説明する。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - Validationの種類
      - 説明
    * - Bean Validation
      - Hibernate validatorを使用して実装した\ ``Validator``\ のテスト
    * - Bean Validation
      - Spring のDIコンテナを使用して実装した\ ``Validator``\ のテスト
    * - Spring Validation
      - \ ``Spring Validation``\ を使用して実装した\ ``Validator``\ のテスト

\ ``Validator``\ の単体テストは本来\ ``Controller``\ のテストとして行うが、その場合は試験パターンが多くなるため、
テストの実施コストを考慮し\ ``Controller``\ と切り分けて\ ``Validator``\ 単体としてテストを行うこともできる。
ここでは、\ ``Validator``\ 単体としてのテスト作成方法を説明する。

本節では、\ ``Bean Validation``\ を使用している場合と\ ``Spring Validation``\ を使用している場合のそれぞれについて実装方法を説明する。

.. _ImplementsOfTestByFunctionTestingBeanValidator:

Bean Validationで実装したValidatorの単体テスト
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``Bean Validation``\ のテスト行う場合、アプリケーションサーバからライブラリが提供されないため、
必要な依存ライブラリ追加する必要がある。追加方法については、\ :ref:`ValidationAddDependencyLibrary`\ を参照されたい。
なお、\ ``Hibernate Validator``\ が用意する入力チェック機能についてはテストスコープ外とする。

ここでは、以下の2通りのBean Validationのテスト方法について説明する。

* Hibernate validatorを使用した\ ``Bean Validation``\
* Spring のDIコンテナを使用した\ ``Bean Validation``\

Hibernate validatorを使用した\ ``Bean Validation``\ のテスト
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Hibernate validatorを使用した\ ``Bean Validation``\ の単体テストにおいて、作成するファイルを以下に示す。

.. figure:: ./images/ImplementsOfTestByFunctionBeanValidationItems.png

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - 作成するファイル名
      - 説明
    * - \ ``FullWidthKatakanaTest.java``\
      - \ ``@FullWidthKatakana``\ アノテーションのテストクラス
    * - \ ``FullWidthKatakanaTestBean.java``\
      - \ ``@FullWidthKatakana``\ アノテーションをフィールドに付与したBeanクラス

|

ここでは、テスト対象の\ ``@FullWidthKatakana``\ を使用したBeanクラス（\ ``FullWidthKatakanaTestBean``\）を作成し、
\ ``javax.validation.ValidatorFactory``\ から生成した\ ``javax.validation.Validator``\ の実装クラスにより
バリデーションチェックを行っている。

以下に、\ ``@FullWidthKatakana``\ アノテーションをフィールドに付与したBeanクラスの作成例を示す。

* ``FullWidthKatakanaTestBean.java``

.. code-block:: java

    public class FullWidthKatakanaTestBean {

        @FullWidthKatakana
        private String testString;

        public FullWidthKatakanaTestBean() {
            // constructor
        }

        public String getTestString() {
            return testString;
        }

        public void setTestString(String testString) {
            this.testString = testString;
        }

    }

* ``FullWidthKatakanaTest.java``

.. code-block:: java

    public class FullWidthKatakanaTest {

        private static Validator validator;

        @BeforeClass
        public static void setUpBeforeClass() {

            // setup
            ValidatorFactory validatorFactory = Validation.buildDefaultValidatorFactory();

            // (1)
            validator = validatorFactory.getValidator();
        }

        @Test
        public void testFullWidthKatakana() {

            // setup
            FullWidthKatakanaTestBean form = new FullWidthKatakanaTestBean();
            form.setTestString("テスト");

            // run the test
            Set<ConstraintViolation<FullWidthKatakanaTestBean>> violations = validator.validate(form); // (2)

            // assert
            assertThat(violations, is(empty())); // (3)
        }
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``getValidator``\ メソッドにより、\ ``Validator``\ を取得する。
        | \ ``Validator``\ を取得することで、\ ``validate``\ メソッドを使った入力チェックが可能となる。
    * - | (2)
      - | \ ``validate``\ メソッドを使い、入力チェックを行う。
        | \ ``validate``\ メソッドを実行することで、入力チェックエラーの数だけ\ ``ConstrainViolation``\ の\ ``Set``\ が返ってくる。
          \ ``validate``\ メソッドの引数には\ ``FullWidthKatakanaBean``\ クラスのオブジェクトを指定する。
    * - | (3)
      - | (2)で取得した\ ``Set``\ から、エラーが発生したかどうかを確認する。
        | 今回はエラーがないため、空の\ ``Set``\ が返ってくる。

.. note:: **バリデーショングループを使用したテスト**

    バリデーショングループを設定している場合、入力チェックを行なう際の\ ``validate``\ メソッド引数に、
    グループを示す任意の\ ``java.lang.Class``\ オブジェクトを指定することで、
    指定したグループの\ ``Validator``\ のみ適用して実行できる。
    バリデーショングループについては、\ :ref:`ValidationGroupValidation`\ を参照されたい。

    以下に、バリデーショングループを使用した\ ``Form``\ 例を示す。

     * テスト対象の ``FullWidthKatakanaTestBean.java``

     .. code-block:: java

         public class FullWidthKatakanaTestBean {

             public interface Search {};
             public interface Register {};

             // (1)
             @Size(min = 5, max = 10, groups = Search.class)
             @FullWidthKatakana(groups = Register.class)
             @NotNull
             private String testString;

             public FullWidthKatakanaTestBean() {
                 // constructor
             }

             public String getTestString() {
                 return testString;
             }

             public void setTestString(String testString) {
                 this.testString = testString;
             }

         }

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
         :header-rows: 1
         :widths: 10 90

         * - 項番
           - 説明
         * - | (1)
           - | フィールドに設定する\ ``Validator``\ をグループ化している。

     * ``FullWidthKatakanaTest.java``

     .. code-block:: java

         public class FullWidthKatakanaTest {

             // omitted

             @Test
             public void testFullWidthKatakana() {

                 // setup
                 FullWidthKatakanaTestBean form = new FullWidthKatakanaTestBean();
                 form.setTestString("テスト");

                 // run the test
                 // (1)
                 Set<ConstraintViolation<FullWidthKatakanaTestBean>> violations = 
                         validator.validate(form, Default.class, Search.class);

                 // assert
                 assertThat(violations, is(empty())); // (2)
             }
         }

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
         :header-rows: 1
         :widths: 10 90

         * - 項番
           - 説明
         * - | (1)
           - | \ ``validate``\ メソッドの引数に、\ ``java.lang.Class``\ オブジェクトを追加することで、
               設定したバリデーショングループに対して入力チェックを実行できる。
               また、\ ``java.lang.Class``\ オブジェクトは例のように複数指定することもできる。
         * - | (2)
           - | エラーが発生したかどうかを確認する。

|

Spring のDIコンテナを使用した\ ``Bean Validation``\ のテスト
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Spring のDIコンテナを使用した\ ``Bean Validation``\ の単体テストにおいて、作成するファイルを以下に示す。

.. figure:: ./images/ImplementsOfTestByFunctionExistInCodeListItems.png

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - 作成するファイル名
      - 説明
    * - \ ``ExistInCodeListTest.java``\
      - Spring のDIコンテナを使用した\ ``Bean Validation``\のテストクラス
    * - \ ``test-context.xml``\
      - Spring Testを使用して単体テストを行う際に必要な設定を補うための設定ファイル。

|

Spring のDIコンテナを利用した\ ``Bean Validation``\は、
\ ``org.springframework.validation.beanvalidation.LocalValidatorFactoryBean``\ から\ ``Validator``\ オブジェクトを
生成することでテストすることができる。

ここでは、Spring のDIコンテナを利用した入力チェックとして\ ``@ExistInCodeList``\ を例にテストの実装方法を説明する。
\ ``@ExistInCodeList``\ についての詳細は\ :ref:`codelist-validate`\ を参照されたい。

テストで使用する設定ファイルに、\ ``Validator``\ オブジェクトを生成するための\ ``LocalValidatorFactoryBean``\ をBean定義する。

* ``test-context.xml``

.. code-block:: xml

    <!-- (1) -->
    <bean id="validator" class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean" />

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@ExistInCodeList``\ でDIコンテナからコードリストBeanを取得するため、
          \ ``test-context.xml``\ でBean定義した\ ``LocalValidatorFactoryBean``\ から生成した\ ``Validator``\ を使う必要がある。

以下に、\ ``@ExistInCodeList``\ が使われている\ ``Form``\ クラスの実装例を示す。

* ``TicketSearchForm.java``

.. code-block:: java

    public class TicketSearchForm implements Serializable {

        @NotNull
        @ExistInCodeList(codeListId = "CL_AIRPORT") // (1)
        private String depAirportCd;

        // omitted
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``depAirportCd``\ フィールドに対して、コードリストに存在する値かどうか検証する。


以下に、\ ``@ExistInCodeList``\ のテストクラス作成方法を説明する。
ここでは、\ ``sample-codelist.xml``\ に定義したコードリスト（\ ``CL_AIRPORT``\）に定義していない値を設定し、
インジェクションした\ ``javax.validation.Validator``\ の実装クラスによりバリデーションチェックエラーになることを確認している。


* ``ExistInCodeListTest.java``

.. code-block:: java

    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration(locations = {
            "classpath:META-INF/spring/sample-infra.xml",
            "classpath:META-INF/spring/sample-codelist.xml",
            "classpath:META-INF/spring/test-context.xml" })
    public class ExistInCodeListTest {

        // (1)
        @Inject
        private Validator validator;

        @Test
        public void testExistInCodeList() {

            // setup
            TicketSearchForm ticketSearchForm = new TicketSearchForm();
            // (2)
            ticketSearchForm.setDepAirportCd("AAA");

            // omitted

            // run the test
            Set<ConstraintViolation<TicketSearchForm>> violations = validator
                    .validate(ticketSearchForm);

            // assert
            // (3)
            assertThat(violations.size(), is(1));
            ConstraintViolation<TicketSearchForm> violation = violations.iterator().next();
            // (4)
            assertThat(violation.getPropertyPath().toString(), is("depAirportCd"));
            // (5)
            assertThat((String) violation.getInvalidValue(), is("AAA"));
            // (6)
            assertThat(violation.getMessage(), is("Does not exist in CL_AIRPORT"));
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
      - | \ ``Validator``\ にSpringの\ ``LocalValidatorFactoryBean``\ から生成した\ ``Validator``\ をDIしている。
        | \ ``LocalValidatorFactoryBean``\ から生成した\ ``Validator``\ はSpringのDIコンテナ上で動作し、
          \ ``@ContextConfiguration``\ で読み込んだコードリストのBeanを取得することができる。
          これにより、\ ``@ExistInCodeList``\ を期待通りに動作させることができる。
    * - | (2)
      - | コードリストに存在しないコードを入力し、\ ``@ExistInCodeList``\ でエラーが発生することを期待する。
    * - | (3)
      - | \ ``size``\ メソッドを使って入力チェックエラーの数を取得し、エラーが発生したかどうかを確認する。
    * - | (4)
      - | 違反したフィールドが想定した箇所であるかを確認する。
    * - | (5)
      - | 違反した入力値が想定した値であるかを確認する。
    * - | (6)
      - | 発生したエラーのメッセージを確認する。


Spring Validatorで実装したValidatorの単体テスト
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``Validator(Spring Validation)``\ の単体テストにおいて、作成するファイルを以下に示す。

.. figure:: ./images/ImplementsOfTestByFunctionSpringValidationItems.png

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - 作成するファイル名
      - 説明
    * - \ ``TicketSearchValidatorTest.java``\
      - \ ``TicketSearchValidator.java``\ のテストクラス

|

以下に、テスト対象のクラスを示す。

* ``TicketSearchValidator.java``

.. code-block:: java

    @Component
    public class TicketSearchValidator implements Validator {

        @Override
        public boolean supports(Class<?> clazz) {
            return (TicketSearchForm.class).isAssignableFrom(clazz);
        }

        @Override
        public void validate(Object target, Errors errors) {

            TicketSearchForm form = (TicketSearchForm) target;

            if (!errors.hasFieldErrors("depAirportCd")
                && !errors.hasFieldErrors("arrAirportCd")) {
                String depAirport = form.getDepAirportCd();
                String arrAirport = form.getArrAirportCd();
                if (depAirport.equals(arrAirport)) {
                    errors.reject(TicketSearchErrorCode.E_AR_B1_5001.code());
                }
            }

            // omitted
        }
    }

以下に、\ ``Validator(Spring Validation)``\ のテストクラス作成方法を説明する。
ここでは、テスト対象の\ ``TicketSearchValidator``\ でエラーになる値を\ ``TicketSearchForm``\ に設定して
バリデーションエラーになることと、エラーメッセージが正しいことを確認している。

* ``TicketSearchValidatorTest.java``

.. code-block:: java

    public class TicketSearchValidatorTest {

        private TicketSearchValidator validator;

        private TicketSearchForm ticketSearchForm;

        private BindingResult result;

        @BeforeClass
        public void setUpBeforeClass() {

            // setup
            validator = new TicketSearchValidator();
        }

        @Test
        public void testTicketSearchValidator() {

            // setup
            ticketSearchForm = new TicketSearchForm();
            result = new DirectFieldBindingResult(ticketSearchForm, "TicketSearchForm");

            ticketSearchForm.setFlightType(FlightType.RT);
            ticketSearchForm.setDepAirportCd("HND");
            ticketSearchForm.setArrAirportCd("HND");
            // omitted

            // run the test
            // (1)
            validator.validate(ticketSearchForm, result);

            // (2)
            assertThat(result.hasErrors(), is(true));

            // (3)
            ObjectError error = result.getGlobalError();

            // (4)
            ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
            // (5)
            messageSource.setBasename("i18n/sample-messages");
            // (6)
            messageSource.setUseCodeAsDefaultMessage(true);

            String code = error.getCode();
            // assert
            // (7)
            assertThat(code, is(TicketSearchErrorCode.E_AR_B1_5001.code()));
            assertThat(messageSource.getMessage(error, Locale.JAPAN),
                    is("出発空港と到着空港に同じ空港は指定できません。区間をご確認ください。"));
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
      - | \ ``validate``\ メソッドの引数に、\ ``ticketSearchForm``\ と、
          \ ``BindingResult``\ インターフェースのオブジェクトを指定することで、
          \ ``ticketSearchForm``\ に対する入力チェックの結果が、
          \ ``BindingResult``\ クラスのオブジェクトに格納される。
    * - | (2)
      - | \ ``hasErrors``\ メソッドを使って、エラーの有無を判定する。
        | エラーがある場合はtrueが返り値として返り、エラーがない場合はfalseが返り値として返る。
    * - | (3)
      - | \ ``getGlobalError``\ メソッドで、エラー内容を取得する。
    * - | (4)
      - | エラーメッセージの内容を確認するために、\ ``MessageSource``\ の実装クラスである
          \ ``org.springframework.context.support.ResourceBundleMessageSource``\ のオブジェクトを生成する。
          クラスの詳細については、
          \ `ResourceBundleMessageSourceのJavadoc <https://docs.spring.io/spring-framework/docs/5.3.18/javadoc-api/org/springframework/context/support/ResourceBundleMessageSource.html>`_\
          を参照されたい。
    * - | (5)
      - | \ ``setBasename``\ メソッドに、メッセージが定義されたプロパティファイルを指定して読み込ませる。
    * - | (6)
      - | \ ``setUseCodeAsDefaultMessage``\ メソッドにtrueを指定すると、エラーコードに対応するメッセージが定義されていない場合にエラーコードが返される。
          falseを指定すると、エラーコードに対応するメッセージが定義されていない場合に\ ``NoSuchMessageException``\ が返される。
          デフォルトではfalseが適用されている。
    * - | (7)
      - | エラーコード、メッセージ内容を検証する。


