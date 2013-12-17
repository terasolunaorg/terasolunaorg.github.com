.. raw:: pdf

    PageBreak

パスワードハッシュ化
================================================================================

.. contents:: 目次
   :local:

Overview
--------------------------------------------------------------------------------
| パスワードのハッシュ化は、セキュアなアプリケーションを設計する上で考慮しなければならない点の一つである。
| 通常のシステムでパスワードを平文で登録することはありえなく、ハッシュ化は必須であるが、
| 強度が弱いアルゴリズムを選択した場合は「オフライン総あたり攻撃」や「レインボークラック」などにより
| 容易にハッシュ化元データを解析されてしまう。
| 
| Spring Securityは、パスワードのハッシュ化の仕組みとして\ ``org.springframework.security.crypto.password.PasswordEncoder``\ インタフェースが用意している。
| その実装クラスとして、

* \ ``org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder``\ 
* \ ``org.springframework.security.crypto.password.StandardPasswordEncoder``\ 

| などが、提供されている。
| 
| \ ``PasswordEncoder``\ の仕組みとして、\ ``encode(String rawPassword)``\ メソッドでハッシュ化を行い、
| \ ``matches(String rawPassword, String encodedPassword)``\ メソッドで照合を行う。

.. figure:: ./images/PasswordEncoder_class.png
   :alt: PasswordEncoder Class Diagram
   :width: 80%
   :align: center

   **Picture - PasswordEncoder Class Diagram**

|

How to use
--------------------------------------------------------------------------------
| 本節では、Spring Securityから提供されている、PasswordEncoderの実装クラスの使用方法について説明する。

**PasswordEncoderの実装クラス一覧**

.. list-table::
   :header-rows: 1
   :widths: 50 50

   * - PasswordEncoderの実装クラス
     - 概要
   * - | \ ``org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder``\
     - | "bcrypt"アルゴリズムでハッシュ化を行うエンコーダ
   * - | \ ``org.springframework.security.crypto.password.StandardPasswordEncoder``\
     - | "SHA-256"アルゴリズム + 1024回のストレッチでハッシュ化を行うエンコーダ
   * - | \ ``org.springframework.security.crypto.password.NoOpPasswordEncoder``\
     - | ハッシュ化を行わないエンコーダ（テスト用）

ハッシュ化に関する要件がない場合は、\ ``BCryptPasswordEncoder``\ を使用することを推奨する。
ただし、\ ``BCryptPasswordEncoder``\ は対攻撃性を高めるために計算時間が多いため、
認証時の性能要件を満たせない場合は\ ``StandardPasswordEncoder``\ を検討すること。

既存のシステムとの関係上、ハッシュ化するアルゴリズムや、ソルトに対して制限がある場合については、
後述する\ ``org.springframework.security.authentication.encoding.PasswordEncoder``\ インタフェースの実装クラスを使用すること。
詳細は、\ :ref:`authenticationPasswordEncoder`\ を参照されたい。

BCryptPasswordEncoder
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| \ ``BCryptPasswordEncoder``\ とは、\ ``PasswordEncoder``\ を実装した、パスワードのハッシュ化を提供しているクラスである。
| ランダムな16バイトのソルトを使用した、bcryptアルゴリズムを使用したエンコーダーである。

.. note::

  Bcryptアルゴリズムは、汎用的なアルゴリズムより意図的に計算量を増やしている。そのため、汎用アルゴリズム(SHA、MD5など)より、
  「オフライン総あたり攻撃」に強い特性を持っている。

.. _BCryptPasswordEncoder:

BCryptPasswordEncoderの設定例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* applicationContext.xml

  .. code-block:: xml
  
    <bean id="passwordEncoder"
        class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder" />    <!-- (1) -->
  
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | passwordEncoderのクラスに\ ``BCryptPasswordEncoder``\ を指定する。
         |
         | コンストラクタの引数に、ソルトのハッシュ化のラウンド数を指定できる。指定できる値は、4～31までである。
         | 指定値を大きくすることにより、強度は増すが、計算数が指数関数的に増大するので、性能面に注意すること。
         | 指定しない場合、「10」が設定される。
       
  .. tip::
  
    How to extendで後述するが、DaoAuthenticationProvider は、\ ``org.springframework.security.crypto.password.PasswordEncoder``\ の実装クラス、
    \ ``org.springframework.security.authentication.encoding.PasswordEncoder``\ の実装クラス両方を設定することができる。
    そのため、従来のPasswordEncoder(authenticationパッケージ)から、新PasswordEncoderに移行する際も、ユーザのパスワード移行が完了後、
    DaoAuthenticationProviderのpasswordEncoderを変更するだけで対応できる。
  
  .. warning::
  
    \ ``DaoAuthenticationProvider``\ を認証プロバイダで設定している場合、\ ``UsernameNotFoundException``\ がスローされた場合、利用者にユーザが存在しないことを悟らせないために、
    \ ``UsernameNotFoundException``\ がスローされた後、意図的にパスワードをハッシュ化している。(サイドチャネル攻撃対策)
  
    上記のハッシュ化に用いる値を作成するために、アプリケーション起動時に、\ ``encode``\ メソッドを内部で1回実行している。
  
  .. warning::
  
    Linux環境でSecureRandomを使用している場合、処理の遅延や、タイムアウトが発生する場合がある。
    本問題の原因は乱数生成に関わるものであり、以下のJava Bug Databaseに説明がある。
  
    http://bugs.sun.com/bugdatabase/view_bug.do?bug_id=6202721
  
    JDK 7のb20以降のバージョンでは、修正されている。
  
    http://bugs.sun.com/bugdatabase/view_bug.do?bug_id=6521844
  
    本問題が発生する場合、JVMの起動引数に以下を設定することで、回避することができる。
  
    -Djava.security.egd=file:///dev/urandom

* Javaクラス

  .. code-block:: java
  
        @Inject
        protected PasswordEncoder passwordEncoder;  // (1)
  
        public String register(Customer customer, String rawPassword) {
            // omitted
            // Password Hashing
            String password = passwordEncoder.encode(rawPassword); // (2)
            customer.setPassword(password);
            // omitted
        }
  
        public boolean matches(Customer customer, String rawPassword) {
            return passwordEncoder.matches(rawPassword, customer.getPassword()); // (3)
        }
  
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | Bean定義した、StandardPasswordEncoderをインジェクションする。
     * - | (2)
       - | パスワードをハッシュ化する例
         | encodeメソッドの引数に平文のパスワードを指定することで、ハッシュ化されたパスワードが戻り値となる。
     * - | (3)
       - | パスワードを照合する例
         | matchesメソッドは、第1引数に平文のパスワード、第2引数にハッシュ化されたパスワードを指定することで、
         | 一致しているかチェックできるメソッドである。

StandardPasswordEncoder
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| \ ``StandardPasswordEncoder``\ はハッシュ化のアルゴリズムとして、SHA-256を利用し、1024回のストレッチを行う。
| また、ランダムに生成される8バイトのソルトを付与している。


| 以下に、\ ``StandardPasswordEncoder``\ の\ ``encode(String rawPassword)``\ メソッド、
| \ ``matches(String rawPassword, String encodedPassword)``\ メソッドの仕組みを説明する。

**encode(String rawPassword)メソッド**

.. figure:: ./images/standard_password_encoder_encode.png
   :alt: encode method
   :width: 50%
   :align: center

   **Picture - encode method**

| ランダムに生成される8バイトのソルト + 秘密鍵 + 引数に指定された、パスワードでハッシュ化される。
| 上記でハッシュ化された値に、ハッシュ化に用いたソルトを先頭に付与した値が、メソッドの戻り値となる。

**matches(String rawPassword, String encodedPassword)メソッド**

.. figure:: ./images/standard_password_encoder_matches.png
   :alt: matches method
   :width: 60%
   :align: center

   **Picture - matches method**

| 引数で渡された、encodedPasswordの先頭のsaltをsplitし、salt + secret + rawPassword でハッシュ化した値と
| encodedPasswordの先頭saltを除いた値とで比較処理を行う。
\

StandardPasswordEncoderの設定例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* applicationContext.xml

  .. code-block:: xml
  
    <bean id="passwordEncoder"
      class="org.springframework.security.crypto.password.StandardPasswordEncoder">
      <!-- from properties file -->
      <constructor-arg value="${passoword.encoder.secret}"/> <!-- (1) -->
    </bean>
  
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | ハッシュ化用の秘密鍵(secret)を指定する。
         | 指定した場合、ハッシュ化処理において、「内部で生成されるソルト」 + 「指定した秘密鍵」 +「パスワード」でハッシュ化される。
         | 秘密鍵(secret)を指定しない場合、レインボーテーブルを用いた攻撃方法に対する強度が下がるため、指定することを推奨する。
         |
         | **秘密鍵(secret)について**
         | 秘密鍵(secret)は、機密情報として扱うこと。
         | そのため、Spring Securityの設定ファイルに直接指定せずプロパティファイルや、環境変数などから取得する。
         | 本例では、プロパティファイルから取得する例が有効になっている。また本番環境ではプロパティファイルの格納場所にも注意する。

  .. tip::

    **秘密鍵(secret)を環境変数から取得する場合**

    StandardPasswordEncoderのbean定義の、\ ``<constructor-arg>``\ に以下の設定を行うことで取得できる。

      .. code-block:: xml
      
        <bean id="passwordEncoder"
          class="org.springframework.security.crypto.password.StandardPasswordEncoder">
          <!-- from environment variable -->
          <constructor-arg value="#{systemEnvironment['PASSWORD_ENCODER_SECRET']}" /> <!-- (1) -->
        </bean>
      
      .. list-table::
         :header-rows: 1
         :widths: 10 90
         
         * - 項番
           - 説明
         * - | (1)
           - | 環境変数:PASSWORD_ENCODER_SECRETから値を取得する。



  | Javaクラス例は\ ``BCryptPasswordEncoder``\ と同様のため、\ :ref:`BCryptPasswordEncoder`\ を参照されたい。

NoOpPasswordEncoder
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| \ ``NoOpPasswordEncoder``\ は、指定した値をそのままの文字列で返却するエンコーダーである。
| 単体テスト時など、ハッシュ化されていない文字列を使用したい場合以外に使用してはいけない。

| 設定例は、BCryptPasswordEncoderと同様のため、省略する。

.. _authenticationPasswordEncoder:

How to extend
--------------------------------------------------------------------------------
| 業務要件によっては、前述した\ ``PasswordEncoder``\ を実装したクラスでは実現できない場合がある。
| 特に、既存のアカウント情報で使用しているハッシュ化方式を踏襲したい場合などは、前述の\ ``PasswordEncoder``\ では要件を満たせないことが多い。

たとえば、既存のハッシュ方式が、以下のような場合が考えられる。
 * アルゴリズムがSHA-512である。
 * ストレッチ回数が1000回である。
 * ソルトはアカウントテーブルのカラムに格納されており、\ ``PasswordEncoder``\ の外から渡す必要がある。

| その場合、\ ``org.springframework.security.crypto.password.PasswordEncoder``\ を実装したクラスではなく、
| 異なるパッケージの\ ``org.springframework.security.authentication.encoding.PasswordEncoder``\ を実装したクラスの使用を推奨する。
\
 .. warning::

     Spring Security 3.1.4以前では、\ ``org.springframework.security.authentication.encoding.PasswordEncoder``\
     を実装したクラスをハッシュ化に使用していたが、3.1.4以降ではDeprecatedとなっている。
     そのため、Springが推奨しているパターンとは異なる。

ShaPasswordEncoderを使用した例
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 業務要件が以下の場合、
| アルゴリズムはSHA-512を使用し、ストレッチを1000回を行う。
| \ :doc:`Authentication`\ で説明した、
| DaoAuthenticationProviderを使用した、認証処理を例に説明する。

* applicationContext.xml

  .. code-block:: xml
  
    <bean id ="passwordEncoder"
        class="org.springframework.security.authentication.encoding.ShaPasswordEncoder"> <!-- (1) -->
        <constructor-arg value="512" /> <!-- (2) -->
        <property name="iterations" value="1000" /> <!-- (3) -->
    </bean>
  
  
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | passwordEncoderには、\ ``org.springframework.security.authentication.encoding.ShaPasswordEncoder``\ を指定する。
         | passwordEncoderに指定する、クラスは使用するアルゴリズムに合わせて変更すること。
     * - | (2)
       - | コンストラクタの引数に、SHAアルゴリズムの種類を設定する
         | 指定可能な値は、「1、256、384、512」である。省略した場合は、「1」が設定される。
     * - | (3)
       - | ハッシュ化時のストレッチングの回数を指定する。
         | 省略した場合は、0回となる。

* spring-mvc.xml

  .. code-block:: xml
  
    <bean id="authenticationProvider"
        class="org.springframework.security.authentication.dao.DaoAuthenticationProvider">
        <!-- omitted -->
        <property name="saltSource" ref="saltSource" /> <!-- (1) -->
        <property name="userDetailsService" ref="userDetailsService" />
        <property name="passwordEncoder" ref="passwordEncoder" /> <!-- (2) -->
    </bean>
  
    <bean id="saltSource"
        class="org.springframework.security.authentication.dao.ReflectionSaltSource"> <!-- (3) -->
        <property name="userPropertyToUse" value="username" /> <!-- (4) -->
    </bean>
  
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | ソルトを外部定義したい場合、\ ``org.springframework.security.authentication.dao.SaltSource``\ を
         | 実装したクラスのBeanIdを設定する。
         | 本例では、 ユーザ情報クラスに設定された値をリフレクションで取得する、
         | \ ``org.springframework.security.authentication.dao.ReflectionSaltSource``\ を定義している。
     * - | (2)
       - | passwordEncoderには、\ ``org.springframework.security.authentication.encoding.ShaPasswordEncoder``\ を指定する。
         | passwordEncoderに指定する、クラスは使用するアルゴリズムに合わせて変更すること。
     * - | (3)
       - | ソルトの作成方法を決める\ ``org.springframework.security.authentication.dao.SaltSource``\ を指定する。
         | ここでは\ ``UserDetails``\ オブジェクトのプロパティをリフレクションで取得する\ ``ReflectionSaltSource``\ を使用する。
     * - | (4)
       - | \ ``UserDetails``\ オブジェクトの\ ``usernamte``\ プロパティをsaltとして使用する。

* Javaクラス

  .. code-block:: java
  
      @Inject
      protected PasswordEncoder passwordEncoder;
  
      public String register(Customer customer, String rawPassword, String userSalt) {
          // omitted
          String password = passwordEncoder.encodePassword(rawPassword,
                  userSalt); // (1)
          customer.setPassword(password);
          // omitted
      }
  
      public boolean matches(Customer customer, String rawPassword, String userSalt) {
          return passwordEncoder.isPasswordValid(customer.getPassword(),
                     rawPassword, userSalt); // (2)
      }
  
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | パスワードをハッシュ化する場合、
         | \ ``org.springframework.security.authentication.encoding.PasswordEncoder``\ を実装したクラスでは、
         | \ ``encodePassword``\ メソッドの引数にパスワードと、ソルト文字列を指定する。
     * - | (2)
       - | パスワードを照合する場合、
         | \ ``isPasswordValid``\ メソッドを使用し、引数にハッシュ化されたパスワード、
         | 平文のパスワード、ソルト文字列を指定することで、ハッシュ化されたパスワードと平文のパスワードを比較する。

Appendix
--------------------------------------------------------------------------------

.. note::    **ストレッチとは**

  ハッシュ関数の計算を繰り返し行うことで、保管するパスワードに関する情報を繰り返し暗号化することである。
  パスワードの総当たり攻撃への対策として、パスワード解析に必要な時間を延ばすために行う。
  しかし、ストレッチはシステムの性能に影響を与えるので、システムの性能を考慮してストレッチ回数を決める必要がある。


.. note::    **ソルトとは**

  暗号化する元となるデータに追加する文字列である。
  ソルトをパスワードに付与することで、見かけ上、パスワード長を長くし、レインボークラックなどのパスワード解析を困難にするために利用する。
  なお、複数のユーザに対して同一のソルトを利用していると、同一パスワードを設定しているユーザが存在した時に、
  ハッシュ値から同一のパスワードである事が分かってしまう。
  そのため、ソルトはユーザごとに異なる値（ランダム値等）を設定することを推奨する。