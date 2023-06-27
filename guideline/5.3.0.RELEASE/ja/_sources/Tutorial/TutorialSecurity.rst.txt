Spring Securityチュートリアル
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:


はじめに
--------------------------------------------------------------------------------

このチュートリアルで学ぶこと
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* Spring Securityによる基本的な認証・認可
* データベース上のアカウント情報を使用したログイン
* 認証済みアカウントオブジェクトの取得方法

対象読者
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* :doc:`./TutorialTodo`\ を実施ずみ (インフラストラクチャ層の実装としてMyBatis3を使用して実施していること)
* Mavenの基本的な操作を理解している

検証環境
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* :doc:`./TutorialTodo`\ と同様。

|

作成するアプリケーションの概要
--------------------------------------------------------------------------------

* ログインページでIDとパスワード指定して、アプリケーションにログインする事ができる。
* ログイン処理で必要となるアカウント情報はデータベース上に格納する。
* ウェルカムページとアカウント情報表示ページがあり、これらのページはログインしないと閲覧する事ができない。
* アプリケーションからログアウトする事ができる。

アプリケーションの概要を以下の図で示す。

.. figure:: ./images_Security/security_tutorial_applicatioin_overview.png
   :width: 90%

URL一覧を以下に示す。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|p{0.40\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 20 15 15 40

    * - 項番
      - プロセス名
      - HTTPメソッド
      - URL
      - 説明
    * - 1
      - ログインフォーム表示
      - GET
      - /login.jsp
      - ログインフォームを表示する
    * - 2
      - ログイン
      - POST
      - /authentication
      - ログインフォームから入力されたユーザー名、パスワードを使って認証する(Spring Securityが行う)
    * - 3
      - ウェルカムページ表示
      - GET
      - /
      - ウェルカムページを表示する
    * - 4
      - アカウント情報表示
      - GET
      - /account
      - ログインユーザーのアカウント情報を表示する
    * - 5
      - ログアウト
      - POST
      - /logout
      - ログアウトする(Spring Securityが行う)

|

環境構築
--------------------------------------------------------------------------------

プロジェクトの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Mavenのアーキタイプを利用し、\ `TERASOLUNA Server Framework for Java (5.x)のブランクプロジェクト <https://github.com/terasolunaorg/terasoluna-gfw-web-blank>`_\ を作成する。

本チュートリアルでは、MyBatis3用のブランクプロジェクトを作成する。

なお、Spring Tool Suite(STS)へのインポート方法やアプリケーションサーバの起動方法など基本知識については、
:doc:`./TutorialTodo` で説明済みのため、本チュートリアルでは説明を割愛する。

.. code-block:: console

    mvn archetype:generate -B^
     -DarchetypeGroupId=org.terasoluna.gfw.blank^
     -DarchetypeArtifactId=terasoluna-gfw-web-blank-mybatis3-archetype^
     -DarchetypeVersion=5.3.0.RELEASE^
     -DgroupId=com.example.security^
     -DartifactId=first-springsecurity^
     -Dversion=1.0.0-SNAPSHOT

|

チュートリアルを進める上で必要となる設定の多くは、作成したブランクプロジェクトに既に設定済みの状態である。
チュートリアルを実施するだけであれば、これらの設定の理解は必須ではないが、
アプリケーションを動かすためにどのような設定が必要なのかを理解しておくことを推奨する。

アプリケーションを動かすために必要な設定(設定ファイル)の解説については、
「:ref:`SecurityTutorialAppendixConfigurationFiles`」を参照されたい。

|

アプリケーションの作成
--------------------------------------------------------------------------------

ドメイン層の実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityの認証処理は基本的に以下の流れになる。

#. 入力された\ ``username``\ からユーザー情報を検索する。
#. ユーザー情報が存在する場合、そのユーザー情報がもつパスワードと入力されたパスワードをハッシュ化したものを比較する。
#. 比較結果が一致する場合、認証成功とみなす。

ユーザー情報が見つからない場合やパスワードの比較結果が一致しない場合は認証失敗である。

ドメイン層ではユーザー名からAccountオブジェクトを取得する処理が必要となる。実装は、以下の順に進める。

#. Domain Object(\ ``Account``\ )の作成
#. \ ``AccountRepository``\ の作成
#. \ ``AccountSharedService``\ の作成

|

Domain Objectの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 認証情報(ユーザー名とパスワード)を保持する\ ``Account``\ クラスを作成する。
| ``src/main/java/com/example/security/domain/model/Account.java``

.. code-block:: java
  
    package com.example.security.domain.model;
  
    import java.io.Serializable;
  
    public class Account implements Serializable {
        private static final long serialVersionUID = 1L;
  
        private String username;
  
        private String password;
  
        private String firstName;
  
        private String lastName;
  
        public String getUsername() {
            return username;
        }
  
        public void setUsername(String username) {
            this.username = username;
        }
  
        public String getPassword() {
            return password;
        }
  
        public void setPassword(String password) {
            this.password = password;
        }
  
        public String getFirstName() {
            return firstName;
        }
  
        public void setFirstName(String firstName) {
            this.firstName = firstName;
        }
  
        public String getLastName() {
            return lastName;
        }
  
        public void setLastName(String lastName) {
            this.lastName = lastName;
        }
  
        @Override
        public String toString() {
            return "Account [username=" + username + ", password=" + password
                    + ", firstName=" + firstName + ", lastName=" + lastName + "]";
        }
    }

|

AccountRepositoryの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``Account``\ オブジェクトをデータベースから取得する処理を実装する。

| \ ``AccountRepository``\ インタフェースを作成する。
| ``src/main/java/com/example/security/domain/repository/account/AccountRepository.java``

.. code-block:: java
  
    package com.example.security.domain.repository.account;
  
    import com.example.security.domain.model.Account;

    public interface AccountRepository {
        Account findOne(String username);
    }

|

| \ ``Account``\ を1件取得するためのSQLをMapperファイルに定義する。
| ``src/main/resources/com/example/security/domain/repository/account/AccountRepository.xml``

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.example.security.domain.repository.account.AccountRepository">

        <resultMap id="accountResultMap" type="Account">
            <id property="username" column="username" />
            <result property="password" column="password" />
            <result property="firstName" column="first_name" />
            <result property="lastName" column="last_name" />
        </resultMap>

        <select id="findOne" parameterType="String" resultMap="accountResultMap">
            SELECT
                username,
                password,
                first_name,
                last_name
            FROM
                account
            WHERE
                username = #{username}
        </select>
    </mapper>

|

AccountSharedServiceの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ユーザー名から\ ``Account``\ オブジェクトを取得する業務処理を実装する。

この処理は、Spring Securityの認証サービスから利用するため、インタフェース名は\ ``AccountSharedService``\ 、クラス名は\ ``AccountSharedServiceImpl``\ とする。

.. note::

    本ガイドラインでは、Serviceから別のServiceを呼び出す事を推奨していない。

    ドメイン層の処理(Service)を共通化したい場合は、\ ``XxxService``\ という名前ではなく、
    Serviceの処理を共通化するためのServiceであることを示すために、
    \ ``XxxSharedService``\ という名前にすることを推奨している。

    本チュートリアルで作成するアプリケーションでは共通化は必須ではないが、
    通常のアプリケーションであればアカウント情報を管理する業務のServiceと処理を共通化することが想定される。
    そのため、本チュートリアルではアカウント情報の取得処理をSharedServiceとして実装する。

|


| \ ``AccountSharedService``\ インタフェースを作成する。
| ``src/main/java/com/example/security/domain/service/account/AccountSharedService.java``

.. code-block:: java

    package com.example.security.domain.service.account;

    import com.example.security.domain.model.Account;

    public interface AccountSharedService {
        Account findOne(String username);
    }

|

| \ ``AccountSharedServiceImpl``\ クラスを作成する。
| ``src/main/java/com/example/security/domain/service/account/AccountSharedServiceImpl.java``

.. code-block:: java

    package com.example.security.domain.service.account;

    import javax.inject.Inject;

    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;
    import org.terasoluna.gfw.common.exception.ResourceNotFoundException;

    import com.example.security.domain.model.Account;
    import com.example.security.domain.repository.account.AccountRepository;

    @Service
    public class AccountSharedServiceImpl implements AccountSharedService {
        @Inject
        AccountRepository accountRepository;

        @Transactional(readOnly=true)
        @Override
        public Account findOne(String username) {
            // (1)
            Account account = accountRepository.findOne(username);
            // (2)
            if (account == null) {
                throw new ResourceNotFoundException("The given account is not found! username="
                        + username);
            }
            return account;
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | ユーザー名に一致する\ ``Account``\ オブジェクトを1件取得する。
    * - | (2)
      - | ユーザー名に一致する\ ``Account``\ が存在しない場合は、共通ライブラリから提供している\ ``ResourceNotFoundException``\ をスローする。

|

.. _Tutorial_CreateAuthService:

認証サービスの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Spring Securityで使用する認証ユーザー情報を保持するクラスを作成する。
| ``src/main/java/com/example/security/domain/service/userdetails/SampleUserDetails.java``

.. code-block:: java

    package com.example.security.domain.service.userdetails;

    import org.springframework.security.core.authority.AuthorityUtils;
    import org.springframework.security.core.userdetails.User;

    import com.example.security.domain.model.Account;

    public class SampleUserDetails extends User { // (1)
        private static final long serialVersionUID = 1L;

        private final Account account; // (2)

        public SampleUserDetails(Account account) {
            // (3)
            super(account.getUsername(), account.getPassword(), AuthorityUtils
                    .createAuthorityList("ROLE_USER")); // (4)
            this.account = account;
        }

        public Account getAccount() { // (5)
            return account;
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | \ ``org.springframework.security.core.userdetails.UserDetails``\ インタフェースを実装する。
         | ここでは\ ``UserDetails``\ を実装した\ ``org.springframework.security.core.userdetails.User`` \ クラスを継承し、本プロジェクト用の\ ``UserDetails``\ クラスを実装する。
     * - | (2)
       - | Springの認証ユーザークラスに、本プロジェクトのアカウント情報を保持させる。
     * - | (3)
       - | \ ``User``\ クラスのコンストラクタを呼び出す。第1引数はユーザー名、第2引数はパスワード、第3引数は権限リストである。
     * - | (4)
       - | 簡易実装として、\ ``"ROLE_USER"``\ というロールのみ持つ権限を作成する。
     * - | (5)
       - | アカウント情報のgetterを用意する。これにより、ログインユーザーの\ ``Account``\ オブジェクトを取得することができる。

|

| Spring Securityで使用する認証ユーザー情報を取得するサービスを作成する。
| ``src/main/java/com/example/security/domain/service/userdetails/SampleUserDetailsService.java``

.. code-block:: java

    package com.example.security.domain.service.userdetails;

    import javax.inject.Inject;

    import org.springframework.security.core.userdetails.UserDetails;
    import org.springframework.security.core.userdetails.UserDetailsService;
    import org.springframework.security.core.userdetails.UsernameNotFoundException;
    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;
    import org.terasoluna.gfw.common.exception.ResourceNotFoundException;

    import com.example.security.domain.model.Account;
    import com.example.security.domain.service.account.AccountSharedService;

    @Service
    public class SampleUserDetailsService implements UserDetailsService { // (1)
        @Inject
        AccountSharedService accountSharedService; // (2)

        @Transactional(readOnly=true)
        @Override
        public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
            try {
                Account account = accountSharedService.findOne(username); // (3)
                return new SampleUserDetails(account); // (4)
            } catch (ResourceNotFoundException e) {
                throw new UsernameNotFoundException("user not found", e); // (5)
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
       - | \ ``org.springframework.security.core.userdetails.UserDetailsService``\ インタフェースを実装する。
     * - | (2)
       - | \ ``AccountSharedService``\ をインジェクションする。
     * - | (3)
       - | \ ``username``\ から\ ``Account``\ オブジェクトを取得する処理を\ ``AccountSharedService``\ に委譲する。
     * - | (4)
       - | 取得した\ ``Account``\ オブジェクトを使用して、本プロジェクト用の\ ``UserDetails``\ オブジェクトを作成し、メソッドの返り値として返却する。
     * - | (5)
       - | 対象のユーザーが見つからない場合は、\ ``UsernameNotFoundException``\ がスローする。

|


データベースの初期化スクリプトの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

本チュートリアルでは、アカウント情報を保持するデータベースとしてH2 Database(インメモリデータベース)を使用する。
そのため、アプリケーションサーバ起動時にSQLを実行してデータベースを初期化する必要がある。

| データベースを初期化するSQLスクリプトを実行するための設定を追加する。
| ``src/main/resources/META-INF/spring/first-springsecurity-env.xml``

.. code-block:: xml
    :emphasize-lines: 4,6,30-36

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:jdbc="http://www.springframework.org/schema/jdbc"
        xsi:schemaLocation="
            http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        ">

        <bean id="dateFactory" class="org.terasoluna.gfw.common.date.jodatime.DefaultJodaTimeDateFactory" />

        <bean id="realDataSource" class="org.apache.commons.dbcp2.BasicDataSource"
            destroy-method="close">
            <property name="driverClassName" value="${database.driverClassName}" />
            <property name="url" value="${database.url}" />
            <property name="username" value="${database.username}" />
            <property name="password" value="${database.password}" />
            <property name="defaultAutoCommit" value="false" />
            <property name="maxTotal" value="${cp.maxActive}" />
            <property name="maxIdle" value="${cp.maxIdle}" />
            <property name="minIdle" value="${cp.minIdle}" />
            <property name="maxWaitMillis" value="${cp.maxWait}" />
        </bean>


        <bean id="dataSource" class="net.sf.log4jdbc.Log4jdbcProxyDataSource">
            <constructor-arg index="0" ref="realDataSource" />
        </bean>

        <!-- (1) -->
        <jdbc:initialize-database data-source="dataSource"
            ignore-failures="ALL">
            <!-- (2) -->
            <jdbc:script location="classpath:/database/${database}-schema.sql" encoding="UTF-8" />
            <!-- (3) -->
            <jdbc:script location="classpath:/database/${database}-dataload.sql" encoding="UTF-8" />
        </jdbc:initialize-database>

        <!--  REMOVE THIS LINE IF YOU USE JPA
        <bean id="transactionManager"
            class="org.springframework.orm.jpa.JpaTransactionManager">
            <property name="entityManagerFactory" ref="entityManagerFactory" />
        </bean>
              REMOVE THIS LINE IF YOU USE JPA  -->
        <!--  REMOVE THIS LINE IF YOU USE MyBatis3
        <bean id="transactionManager"
            class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <property name="dataSource" ref="dataSource" />
            <property name="rollbackOnCommitFailure" value="true" />
        </bean>
              REMOVE THIS LINE IF YOU USE MyBatis3  -->
    </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - \ ``<jdbc:initialize-database>``\ タグにデータベースを初期化するSQLスクリプトを実行するための設定を行う。

        この設定は通常、開発中のみでしか使用しない(環境に依存する設定)ため、\ ``first-springsecurity-env.xml``\ に定義する。
    * - | (2)
      - アカウント情報を保持するテーブルを作成するためのDDL文が記載されているSQLファイルを指定する。

        ブランクプロジェクトの設定では、\ ``first-springsecurity-infra.properties``\ に\ ``database=H2``\ と定義されているため、\ ``H2-schema.sql``\ が実行される。
    * - | (3)
      - デモユーザーを登録するためのDML文が記載されているSQLファイルを指定する。

        ブランクプロジェクトの設定では、\ ``first-springsecurity-infra.properties``\ に\ ``database=H2``\ と定義されているため、\ ``H2-dataload.sql``\ が実行される。

|

| アカウント情報を保持するテーブルを作成するためのDDL文を作成する。
| ``src/main/resources/database/H2-schema.sql``

.. code-block:: sql

    CREATE TABLE account(
        username varchar(128),
        password varchar(60),
        first_name varchar(128),
        last_name varchar(128),
        constraint pk_tbl_account primary key (username)
    );

|
| デモユーザー(username=demo、password=demo)を登録するためのDML文を作成する。
| ``src/main/resources/database/H2-dataload.sql``

.. code-block:: sql

    INSERT INTO account(username, password, first_name, last_name) VALUES('demo', '$2a$10$oxSJl.keBwxmsMLkcT9lPeAIxfNTPNQxpeywMrF7A3kVszwUTqfTK', 'Taro', 'Yamada'); -- (1)
    COMMIT;

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - ブランクプロジェクトの設定では、\ ``applicationContext.xml``\ にパスワードをハッシュ化するためのクラスとして\ ``org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder``\ が設定されている。

        本チュートリアルでは、\ ``BCryptPasswordEncoder``\を使用してパスワードのハッシュ化を行うため、パスワードには\ ``"demo"``\という文字列をBCryptアルゴリズムでハッシュ化した文字列を投入する。

|

ドメイン層の作成後のパッケージエクスプローラー
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ドメイン層に作成したファイルを確認する。

Package ExplorerのPackage PresentationはHierarchicalを使用している。

.. figure:: ./images_Security/security_tutorial-domain-layer-package-explorer.png
   :alt: security tutorial domain layer package explorer

|

アプリケーション層の実装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``spring-security.xml``\ にSpring Securityによる認証・認可の設定を行う。

本チュートリアルで作成するアプリケーションで扱うURLのパターンを以下に示す。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 30 70
   
   * - | URL
     - | 説明
   * - | /login.jsp
     - | ログインフォームを表示するためのURL
   * - | /login.jsp?error=true
     - | 認証エラー時に遷移するページ(ログインページ)を表示するためのURL
   * - | /login
     - | 認証処理を行うためのURL
   * - | /logout
     - | ログアウト処理を行うためのURL
   * - | /
     - | ウェルカムページを表示するためのURL
   * - | /account
     - | ログインユーザーのアカウント情報を表示するためのURL

|

.. _Tutorial_setting-spring-security:

| ブランクプロジェクトから提供されている設定に加えて、以下の設定を追加する。
| ``src/main/resources/META-INF/spring/spring-security.xml``

.. code-block:: xml
    :emphasize-lines: 12-15,16-19,23-24,31-33,34-35

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:sec="http://www.springframework.org/schema/security"
        xsi:schemaLocation="
            http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        ">

        <sec:http pattern="/resources/**" security="none"/>
        <sec:http>
            <!-- (1) -->
            <sec:form-login
                login-page="/login.jsp"
                authentication-failure-url="/login.jsp?error=true" />
            <!-- (2) -->
            <sec:logout
                logout-success-url="/"
                delete-cookies="JSESSIONID" />
            <sec:access-denied-handler ref="accessDeniedHandler"/>
            <sec:custom-filter ref="userIdMDCPutFilter" after="ANONYMOUS_FILTER"/>
            <sec:session-management />
            <!-- (3) -->
            <sec:intercept-url pattern="/login.jsp" access="permitAll" />
            <sec:intercept-url pattern="/**" access="isAuthenticated()" />
        </sec:http>

        <sec:authentication-manager>
            <!-- com.example.security.domain.service.userdetails.SampleUserDetailsService
              is scanned by component scan with @Service -->
            <!-- (4) -->
            <sec:authentication-provider
                user-service-ref="sampleUserDetailsService">
                <!-- (5) -->
                <sec:password-encoder ref="passwordEncoder" />
            </sec:authentication-provider>
        </sec:authentication-manager>

        <!-- CSRF Protection -->
        <bean id="accessDeniedHandler"
            class="org.springframework.security.web.access.DelegatingAccessDeniedHandler">
            <constructor-arg index="0">
                <map>
                    <entry
                        key="org.springframework.security.web.csrf.InvalidCsrfTokenException">
                        <bean
                            class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
                            <property name="errorPage"
                                value="/WEB-INF/views/common/error/invalidCsrfTokenError.jsp" />
                        </bean>
                    </entry>
                    <entry
                        key="org.springframework.security.web.csrf.MissingCsrfTokenException">
                        <bean
                            class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
                            <property name="errorPage"
                                value="/WEB-INF/views/common/error/missingCsrfTokenError.jsp" />
                        </bean>
                    </entry>
                </map>
            </constructor-arg>
            <constructor-arg index="1">
                <bean
                    class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
                    <property name="errorPage"
                        value="/WEB-INF/views/common/error/accessDeniedError.jsp" />
                </bean>
            </constructor-arg>
        </bean>

        <!-- Put UserID into MDC -->
        <bean id="userIdMDCPutFilter" class="org.terasoluna.gfw.security.web.logging.UserIdMDCPutFilter">
        </bean>

    </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
  
    * - 項番
      - 説明
    * - | (1)
      - \ ``<sec:form-login>``\ タグでログインフォームに関する設定を行う。

        \ ``<sec:form-login>``\ タグには、

        * \ ``login-page``\ 属性にログインフォームを表示するためのURL
        * \ ``authentication-failure-url``\ 属性に認証エラー時に遷移するページを表示するためのURL

        を設定する。
    * - | (2)
      - \ ``<sec:logout>``\ タグでログアウトに関する設定を行う。

        \ ``<sec:logout>``\ タグには、

        * \ ``logout-success-url``\ 属性にログアウト後に遷移するページを表示するためのURL(本チュートリアルではウェルカムページを表示するためのURL)
        * \ ``delete-cookies``\ 属性にログアウト時に削除するCookie名(本チュートリアルではセッションIDのCookie名)

        を設定する。
    * - | (3)
      - \ ``<sec:intercept-url>``\ タグを使用してURL毎の認可設定を行う。

        \ ``<sec:intercept-url>``\ タグには、

        * ログインフォームを表示するためのURLには、全てのユーザーのアクセスを許可する\ ``permitAll``\
        * 上記以外のURLには、認証済みユーザーのみアクセスを許可する\ ``isAuthenticated()``\

        を設定する。

        ただし、\ ``/resources/``\ 配下のURLについては、Spring Securityによる認証・認可処理を行わない設定(\ ``<sec:http pattern="/resources/**" security="none"/>``\ )が行われているため、全てのユーザーがアクセスすることができる。
    * - | (4)
      - \ ``<sec:authentication-provider>``\ タグを使用して、認証処理を行う\ ``org.springframework.security.authentication.AuthenticationProvider``\ の設定を行う。

        デフォルトでは、\ ``UserDetailsService``\ を使用して\ ``UserDetails``\ を取得し、その\ ``UserDetails``\ が持つハッシュ化済みパスワードと、ログインフォームで指定されたパスワードを比較してユーザー認証を行うクラス(\ ``org.springframework.security.authentication.dao.DaoAuthenticationProvider``\ )が使用される。

        \ ``user-service-ref``\ 属性に\ ``UserDetailsService``\ インタフェースを実装しているコンポーネントのbean名を指定する。本チュートリアルでは、ドメイン層に作成した\ ``SampleUserDetailsService``\ クラスを設定する。
    * - | (5)
      - \ ``<sec:password-encoder>``\ タグを使用して、ログインフォームで指定されたパスワードをハッシュ化するためのクラス(\ ``PasswordEncoder``\ )の設定を行う。

        本チュートリアルでは、\ ``applicationContext.xml``\ に定義されている\ ``org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder``\ を利用する。

|

ログインページの作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| ログインページにログインフォームを作成する。
| ``src/main/webapp/login.jsp``

.. code-block:: jsp
  
    <!DOCTYPE html>
    <html>
    <head>
    <title>Login Page</title>
    <link rel="stylesheet"
        href="${pageContext.request.contextPath}/resources/app/css/styles.css">
    </head>
    <body>
        <div id="wrapper">
            <h3>Login with Username and Password</h3>

            <!-- (1) -->
            <c:if test="${param.containsKey('error')}">
                <!-- (2) -->
                <t:messagesPanel messagesType="error"
                    messagesAttributeName="SPRING_SECURITY_LAST_EXCEPTION" />
            </c:if>

            <!-- (3) -->
            <form:form action="${pageContext.request.contextPath}/login">
                <table>
                    <tr>
                        <td><label for="username">User:</label></td>
                        <td><input type="text" id="username"
                            name="username" value='demo'>(demo)</td><!-- (4) -->
                    </tr>
                    <tr>
                        <td><label for="password">Password:</label></td>
                        <td><input type="password" id="password"
                            name="password" value="demo" />(demo)</td><!-- (5) -->
                    </tr>
                    <tr>
                        <td>&nbsp;</td>
                        <td><input name="submit" type="submit" value="Login" /></td>
                    </tr>
                </table>
            </form:form>
        </div>
    </body>
    </html>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
  
    * - 項番
      - 説明
    * - | (1)
      - 認証が失敗した場合、\ ``"/login.jsp?error=true"``\ が呼び出され、ログインページを表示する。
        そのため、認証エラー後の表示の時のみエラーメッセージが表示されるように\ ``<c:if>``\ タグを使用する。
    * - | (2)
      - 共通ライブラリから提供されている\ ``<t:messagesPanel>``\ タグを使用してエラーメッセージを表示する。

        認証が失敗した場合、認証エラーの例外オブジェクトが\ ``"SPRING_SECURITY_LAST_EXCEPTION"``\ という属性名でセッションスコープに格納される。
    * - | (3)
      - \ ``<form:form>``\ タグの\ ``action``\ 属性に、認証処理用のURL(\ ``"/login"``\ )を設定する。このURLはSpring Securityのデフォルトである。

        認証処理に必要なパラメータ(ユーザー名とパスワード)をPOSTメソッドを使用して送信する。
    * - | (4)
      - ユーザー名を指定するテキストボックスを作成する。

        Spring Securityのデフォルトのパラメータ名は\ ``username``\ である。
    * - | (5)
      - パスワードを指定するテキストボックス(パスワード用のテキストボックス)を作成する。

        Spring Securityのデフォルトのパラメータ名は\ ``password``\ である。

|

| セッションスコープに格納される認証エラーの例外オブジェクトをJSPから取得できるようにする。
| ``src/main/webapp/WEB-INF/views/common/include.jsp``

.. code-block:: jsp
    :emphasize-lines: 1

    <%@ page session="true"%> <!-- (6) -->
    <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
    <%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt"%>
    <%@ taglib uri="http://www.springframework.org/tags" prefix="spring"%>
    <%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>
    <%@ taglib uri="http://www.springframework.org/security/tags" prefix="sec"%>
    <%@ taglib uri="http://terasoluna.org/tags" prefix="t"%>
    <%@ taglib uri="http://terasoluna.org/functions" prefix="f"%>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (6)
      - \ ``page``\ ディレクティブの\ ``session``\ 属性を\ ``true``\ にする。

.. note::

    ブランクプロジェクトのデフォルト設定では、JSPからセッションスコープにアクセスできないようになっている。
    これは、安易にセッションが使用されないようにするためであるが、
    認証エラーの例外オブジェクトをJSPから取得する場合は、JSPからセッションスコープにアクセスできるようにする必要がある。

| 

| ブラウザのアドレスバーに http://localhost:8080/first-springsecurity/ を入力し、ウェルカムページを表示しようとする。
| 未ログイン状態のため、\ ``<sec:form-login>``\ タグの\ ``login-page``\ 属性の設定値( http://localhost:8080/first-springsecurity/login.jsp )に遷移し、以下のような画面が表示される。

.. figure:: ./images_Security/security_tutorial_login_page.png
   :width: 80%


JSPからログインユーザーのアカウント情報へアクセス
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| JSPからログインユーザーのアカウント情報にアクセスし、氏名を表示する。
| ``src/main/webapp/WEB-INF/views/welcome/home.jsp``

.. code-block:: xml
    :emphasize-lines: 10-11,17-18
  
    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="utf-8">
    <title>Home</title>
    <link rel="stylesheet"
        href="${pageContext.request.contextPath}/resources/app/css/styles.css">
    </head>

    <!-- (1) -->
    <sec:authentication property="principal.account" var="account" />

    <body>
        <div id="wrapper">
            <h1>Hello world!</h1>
            <p>The time on the server is ${serverTime}.</p>
            <!-- (2) -->
            <p>Welcome ${f:h(account.firstName)} ${f:h(account.lastName)} !!</p>
            <ul>
                <li><a href="${pageContext.request.contextPath}/account">view account</a></li>
            </ul>
        </div>
    </body>
    </html>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
  
    * - 項番
      - 説明
    * - | (1)
      - \ ``<sec:authentication>``\ タグを使用して、ログインユーザーの\ ``org.springframework.security.core.Authentication``\ オブジェクトにアクセスする。

        \ ``property``\ 属性を使用すると\ ``Authentication``\ オブジェクトが保持する任意のプロパティにアクセスする事ができ、アクセスしたプロパティ値は\ ``var``\ 属性を使用して任意のスコープに格納することできる。
        デフォルトではpageスコープの設定され、このJSP内のみで参照可能となる。

        チュートリアルでは、ログインユーザーの\ ``Account``\ オブジェクトを\ ``account``\ という属性名でpageスコープに格納する。
    * - | (2)
      - ログインユーザーの\ ``Account``\ オブジェクトにアクセスして、\ ``firstName``\ と\ ``lastName``\ を表示する。

|

ログインページのLoginボタンを押下し、ウェルカムページを表示する。

.. figure:: ./images_Security/security_tutorial_welcome_page.png
   :width: 70%


ログアウトボタンの追加
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| ログアウトするためのボタンを追加する。
| ``src/main/webapp/WEB-INF/views/welcome/home.jsp``

.. code-block:: xml
    :emphasize-lines: 18-21

    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="utf-8">
    <title>Home</title>
    <link rel="stylesheet"
        href="${pageContext.request.contextPath}/resources/app/css/styles.css">
    </head>

    <sec:authentication property="principal.account" var="account" />

    <body>
        <div id="wrapper">
            <h1>Hello world!</h1>
            <p>The time on the server is ${serverTime}.</p>
            <p>Welcome ${f:h(account.firstName)} ${f:h(account.lastName)} !!</p>
            <p>
                <!-- (1) -->
                <form:form action="${pageContext.request.contextPath}/logout">
                    <button type="submit">Logout</button>
                </form:form>
            </p>
            <ul>
                <li><a href="${pageContext.request.contextPath}/account">view account</a></li>
            </ul>
        </div>
    </body>
    </html>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - \ ``<form:form>``\ タグを使用して、ログアウト用のフォームを追加する。

        \ ``action``\ 属性には、ログアウト処理用のURL(\ ``"/logout"``\ )を指定して、Logoutボタンを追加する。このURLはSpring Securityのデフォルトである。

|

Logoutボタンを押下し、アプリケーションからログアウトする(ログインページが表示される)。

.. figure:: ./images_Security/security_tutorial_add_logout.png
    :width: 70%


Controllerからログインユーザーのアカウント情報へアクセス
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Controllerからログインユーザーのアカウント情報にアクセスし、アカウント情報をViewに引き渡す。
| ``src/main/java/com/example/security/app/account/AccountController.java``

.. code-block:: java
    :emphasize-lines: 17,19-21
  
    package com.example.security.app.account;

    import org.springframework.security.core.annotation.AuthenticationPrincipal;
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.web.bind.annotation.RequestMapping;

    import com.example.security.domain.model.Account;
    import com.example.security.domain.service.userdetails.SampleUserDetails;

    @Controller
    @RequestMapping("account")
    public class AccountController {

        @RequestMapping
        public String view(
                @AuthenticationPrincipal SampleUserDetails userDetails, // (1)
                Model model) {
            // (2)
            Account account = userDetails.getAccount();
            model.addAttribute(account);
            return "account/view";
        }
    }
  
.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
  
    * - 項番
      - 説明
    * - | (1)
      - | \ ``@AuthenticationPrincipal``\ アノテーションを指定して、ログインユーザーの\ ``UserDetails``\ オブジェクトを受け取る。
    * - | (2)
      - | \ ``SampleUserDetails``\ オブジェクトが保持している\ ``Account``\ オブジェクトを取得し、Viewに引き渡すために\ ``Model``\ に格納する。

| 

| Controllerから引き渡されたアカウント情報にアクセスし、アカウント情報を表示する。
| ``src/main/webapp/WEB-INF/views/account/view.jsp``

.. code-block:: jsp

    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="utf-8">
    <title>Home</title>
    <link rel="stylesheet"
        href="${pageContext.request.contextPath}/resources/app/css/styles.css">
    </head>
    <body>
        <div id="wrapper">
            <h1>Account Information</h1>
            <table>
                <tr>
                    <th>Username</th>
                    <td>${f:h(account.username)}</td>
                </tr>
                <tr>
                    <th>First name</th>
                    <td>${f:h(account.firstName)}</td>
                </tr>
                <tr>
                    <th>Last name</th>
                    <td>${f:h(account.lastName)}</td>
                </tr>
            </table>
        </div>
    </body>
    </html>

| 

ウェルカムページのview accountリンクを押下して、ログインユーザーのアカウント情報表示ページを表示する。

.. figure:: ./images_Security/security_tutorial_account_information_page.png
   :width: 80%


アプリケーション層の作成後のパッケージエクスプローラー
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

アプリケーション層に作成したファイルを確認する。

Package ExplorerのPackage PresentationはHierarchicalを使用している。

.. figure:: ./images_Security/security_tutorial-application-layer-package-explorer.png
   :alt: security tutorial application layer package explorer

|

おわりに
--------------------------------------------------------------------------------
本チュートリアルでは以下の内容を学習した。

* Spring Securityによる基本的な認証・認可
* 認証ユーザーオブジェクトのカスタマイズ方法
* RepositoryおよびServiceクラスを用いた認証処理の設定
* JSPでログイン済みアカウント情報にアクセスする方法
* Controllerでログイン済みアカウント情報にアクセスする方法

|

Appendix
--------------------------------------------------------------------------------

.. _SecurityTutorialAppendixConfigurationFiles:

設定ファイルの解説
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityを利用するためにどのような設定が必要なのかを理解するために、設定ファイルの解説を行う。

spring-security.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``spring-security.xml``\ には、Spring Securityに関する定義を行う。

作成したブランクプロジェクトの\ ``src/main/resources/META-INF/spring/spring-security.xml``\ は、以下のような設定となっている。

.. code-block:: xml
    :emphasize-lines: 10,13,15,17,19,21,25,28,61

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:sec="http://www.springframework.org/schema/security"
        xsi:schemaLocation="
            http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        ">

        <!-- (1) -->
        <sec:http pattern="/resources/**" security="none"/>
        <sec:http>
            <!-- (2) -->
            <sec:form-login/>
            <!-- (3) -->
            <sec:logout/>
            <!-- (4) -->
            <sec:access-denied-handler ref="accessDeniedHandler"/>
            <!-- (5) -->
            <sec:custom-filter ref="userIdMDCPutFilter" after="ANONYMOUS_FILTER"/>
            <!-- (6) -->
            <sec:session-management />
        </sec:http>

        <!-- (7) -->
        <sec:authentication-manager />

        <!-- (4) -->
        <!-- CSRF Protection -->
        <bean id="accessDeniedHandler"
            class="org.springframework.security.web.access.DelegatingAccessDeniedHandler">
            <constructor-arg index="0">
                <map>
                    <entry
                        key="org.springframework.security.web.csrf.InvalidCsrfTokenException">
                        <bean
                            class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
                            <property name="errorPage"
                                value="/WEB-INF/views/common/error/invalidCsrfTokenError.jsp" />
                        </bean>
                    </entry>
                    <entry
                        key="org.springframework.security.web.csrf.MissingCsrfTokenException">
                        <bean
                            class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
                            <property name="errorPage"
                                value="/WEB-INF/views/common/error/missingCsrfTokenError.jsp" />
                        </bean>
                    </entry>
                </map>
            </constructor-arg>
            <constructor-arg index="1">
                <bean
                    class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
                    <property name="errorPage"
                        value="/WEB-INF/views/common/error/accessDeniedError.jsp" />
                </bean>
            </constructor-arg>
        </bean>

        <!-- (5) -->
        <!-- Put UserID into MDC -->
        <bean id="userIdMDCPutFilter" class="org.terasoluna.gfw.security.web.logging.UserIdMDCPutFilter">
        </bean>

    </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - \ ``<sec:http>``\ タグを使用してHTTPアクセスに対して認証・認可を制御する。

        ブランクプロジェクトのデフォルトの設定では、静的リソース(js, css, imageファイルなど)にアクセスするためのURLを認証・認可の対象外にしている。
    * - \ (2)
      - \ ``<sec:form-login>``\ タグを使用して、フォーム認証を使用したログインに関する動作を制御する。
        \ 使用方法については、「:ref:`form-login`」 を参照されたい
    * - \ (3)
      - \ ``<sec:logout>``\ タグ を使用して、ログアウトに関する動作を制御する。
        \ 使用方法については、「:ref:`SpringSecurityAuthenticationLogout`」 を参照されたい。
    * - | (4)
      - \ ``<sec:access-denied-handler>``\ タグを使用して、アクセスを拒否した後の動作を制御する。

        ブランクプロジェクトのデフォルトの設定では、

        * 不正なCSRFトークンを検知した場合(\ ``InvalidCsrfTokenException``\ が発生した場合)の遷移先
        * トークンストアからCSRFトークンが取得できない場合(\ ``MissingCsrfTokenException``\ が発生した場合)の遷移先
        * 認可処理でアクセスが拒否された場合(上記以外の\ ``AccessDeniedException``\ が発生した場合)の遷移先

        が設定済みである。
    * - | (5)
      - Spring Securityの認証ユーザ名をロガーのMDCに格納するためのサーブレットフィルタを有効化する。
        この設定を有効化すると、ログに認証ユーザ名が出力されるため、トレーサビリティを向上することができる。
    * - | (6)
      - \ ``<sec:session-management>``\ タグを使用して、Spring Securityのセッション管理方法を制御する。

        使用方法については、「:ref:`SpringSecuritySessionManagementSetup`」を参照されたい。
    * - | (7)
      - \ ``<sec:authentication-manager>``\ タグを使用して、認証処理を制御する。

        使用方法については、「:ref:`AuthenticationProviderConfiguration`」を参照されたい。

|

spring-mvc.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``spring-mvc.xml``\ には、Spring SecurityとSpring MVCを連携するための設定を行う。

作成したブランクプロジェクトの\ ``src/main/resources/META-INF/spring/spring-mvc.xml``\ は、以下のような設定となっている。
Spring Securityと関係のない設定については、説明を割愛する。

.. code-block:: xml
    :emphasize-lines: 22-24,87-89

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xmlns:mvc="http://www.springframework.org/schema/mvc"
        xmlns:util="http://www.springframework.org/schema/util"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
        ">

        <context:property-placeholder
            location="classpath*:/META-INF/spring/*.properties" />

        <mvc:annotation-driven>
            <mvc:argument-resolvers>
                <bean
                    class="org.springframework.data.web.PageableHandlerMethodArgumentResolver" />
                <!-- (1) -->
                <bean
                    class="org.springframework.security.web.method.annotation.AuthenticationPrincipalArgumentResolver" />
            </mvc:argument-resolvers>
            <!-- workaround to CVE-2016-5007. -->
            <mvc:path-matching path-matcher="pathMatcher" />
        </mvc:annotation-driven>

        <mvc:default-servlet-handler />

        <context:component-scan base-package="com.example.security.app" />

        <mvc:resources mapping="/resources/**"
            location="/resources/,classpath:META-INF/resources/"
            cache-period="#{60 * 60}" />

        <mvc:interceptors>
            <mvc:interceptor>
                <mvc:mapping path="/**" />
                <mvc:exclude-mapping path="/resources/**" />
                <mvc:exclude-mapping path="/**/*.html" />
                <bean
                    class="org.terasoluna.gfw.web.logging.TraceLoggingInterceptor" />
            </mvc:interceptor>
            <mvc:interceptor>
                <mvc:mapping path="/**" />
                <mvc:exclude-mapping path="/resources/**" />
                <mvc:exclude-mapping path="/**/*.html" />
                <bean
                    class="org.terasoluna.gfw.web.token.transaction.TransactionTokenInterceptor" />
            </mvc:interceptor>
            <mvc:interceptor>
                <mvc:mapping path="/**" />
                <mvc:exclude-mapping path="/resources/**" />
                <mvc:exclude-mapping path="/**/*.html" />
                <bean class="org.terasoluna.gfw.web.codelist.CodeListInterceptor">
                    <property name="codeListIdPattern" value="CL_.+" />
                </bean>
            </mvc:interceptor>
            <!--  REMOVE THIS LINE IF YOU USE JPA
            <mvc:interceptor>
                <mvc:mapping path="/**" />
                <mvc:exclude-mapping path="/resources/**" />
                <mvc:exclude-mapping path="/**/*.html" />
                <bean
                    class="org.springframework.orm.jpa.support.OpenEntityManagerInViewInterceptor" />
            </mvc:interceptor>
                REMOVE THIS LINE IF YOU USE JPA  -->
        </mvc:interceptors>

        <!-- Settings View Resolver. -->
        <mvc:view-resolvers>
            <mvc:bean-name />
            <mvc:tiles />
            <mvc:jsp prefix="/WEB-INF/views/" />
        </mvc:view-resolvers>

        <mvc:tiles-configurer>
            <mvc:definitions location="/WEB-INF/tiles/tiles-definitions.xml" />
        </mvc:tiles-configurer>

        <bean id="requestDataValueProcessor"
            class="org.terasoluna.gfw.web.mvc.support.CompositeRequestDataValueProcessor">
            <constructor-arg>
                <util:list>
                    <!-- (2) -->
                    <bean
                        class="org.springframework.security.web.servlet.support.csrf.CsrfRequestDataValueProcessor" />
                    <bean
                        class="org.terasoluna.gfw.web.token.transaction.TransactionTokenRequestDataValueProcessor" />
                </util:list>
            </constructor-arg>
        </bean>

        <!-- Setting Exception Handling. -->
        <!-- Exception Resolver. -->
        <bean id="systemExceptionResolver"
            class="org.terasoluna.gfw.web.exception.SystemExceptionResolver">
            <property name="exceptionCodeResolver" ref="exceptionCodeResolver" />
            <!-- Setting and Customization by project. -->
            <property name="order" value="3" />
            <property name="exceptionMappings">
                <map>
                    <entry key="ResourceNotFoundException" value="common/error/resourceNotFoundError" />
                    <entry key="BusinessException" value="common/error/businessError" />
                    <entry key="InvalidTransactionTokenException" value="common/error/transactionTokenError" />
                    <entry key=".DataAccessException" value="common/error/dataAccessError" />
                </map>
            </property>
            <property name="statusCodes">
                <map>
                    <entry key="common/error/resourceNotFoundError" value="404" />
                    <entry key="common/error/businessError" value="409" />
                    <entry key="common/error/transactionTokenError" value="409" />
                    <entry key="common/error/dataAccessError" value="500" />
                </map>
            </property>
            <property name="defaultErrorView" value="common/error/systemError" />
            <property name="defaultStatusCode" value="500" />
        </bean>
        <!-- Setting AOP. -->
        <bean id="handlerExceptionResolverLoggingInterceptor"
            class="org.terasoluna.gfw.web.exception.HandlerExceptionResolverLoggingInterceptor">
            <property name="exceptionLogger" ref="exceptionLogger" />
        </bean>
        <aop:config>
            <aop:advisor advice-ref="handlerExceptionResolverLoggingInterceptor"
                pointcut="execution(* org.springframework.web.servlet.HandlerExceptionResolver.resolveException(..))" />
        </aop:config>

        <!-- Setting PathMatcher. -->
        <bean id="pathMatcher" class="org.springframework.util.AntPathMatcher">
            <property name="trimTokens" value="false" />
        </bean>

    </beans>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - \ ``@AuthenticationPrincipal``\ アノテーションを指定して、ログインユーザーの\ ``UserDetails``\ オブジェクトをControllerの引数として受け取れるようにするための設定。

        \ ``<mvc:argument-resolvers>``\ タグに\ ``AuthenticationPrincipalArgumentResolver``\ を指定する。
    * - | (2)
      - \ ``<form:form>``\ タグ(JSPタグライブラリ)を使用して、CSRFトークン値をHTMLフォームに埋め込むための設定。

        \ ``CompositeRequestDataValueProcessor``\ のコンストラクタに\ ``CsrfRequestDataValueProcessor``\ を指定する。


.. raw:: latex

   \newpage

