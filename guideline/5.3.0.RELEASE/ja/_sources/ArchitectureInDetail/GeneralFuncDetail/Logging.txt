ロギング
================================================================================

.. only:: html

 .. contents::
    :depth: 3
    :local:

.. note::

  本ガイドラインで説明する内容はあくまで指針のため、業務要件に合わせて柔軟に対応すること。

Overview
--------------------------------------------------------------------------------

| システムを運用する上、業務使用量の調査、システムダウンや、
| 業務エラー等でその原因を究明するための情報源として、ログおよびメッセージを出力する。

| デバッグ時にログ出力を取り入れることで、解析の作業効率が格段に向上するため、ログを出力しておくことは重要である。

| 動きを確認するだけであれば、IDEのデバッグ実行や、\ ``System.out.println``\ のような簡易的な出力で行える。
| しかし、出力結果を手動で保存しておかないと、後に結果の確認ができず、解析の作業効率が格段に下がる。
| ロギングライブラリを導入してログをとることは、出力するコードを書くのみで、
| その後、好きなタイミングでログを確認することができる。
| 作業の時間、証跡、解析を考えると、ロギングライブラリを導入することを推奨する。

| Javaでは、ログ出力の方法は複数あり、多くの方法が選べるが、コーディングの簡易性、変更の容易性、性能を判断して、
| 本ガイドラインでは、ロギングライブラリに、\ `SLF4J <http://www.slf4j.org/>`_ (インタフェース) + `Logback <http://logback.qos.ch/>`_\  (実装)を推奨している。


ログの種類
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| アプリケーション開発時における代表的なログを、以下に示す。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.35\linewidth}|p{0.40\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 15 35 40
   :class: longtable

   * - ログレベル
     - カテゴリ
     - 出力目的
     - 出力内容
   * - TRACE
     - 性能ログ
     - | リクエストの処理時間の測定
       | (本番環境運用時は出力対象としない)
     - | 処理開始終了時間、処理経過時間(ms)、
       | 実行処理を判別できる情報(実行コントローラ + メソッド、リクエストURLなど)等
   * - DEBUG
     - デバッグログ
     - | 開発時のデバッグ
       | (本番環境運用時には出力対象としない)
     - 任意(実行したクエリ、入力パラメータ、戻り値など)
   * - INFO
     - アクセスログ
     - | 業務量の把握
     - | アクセス日時、利用ユーザを判別できる情報(IPアドレス、認証情報)、
       | 実行処理を判別できる情報(リクエストURL)等、証跡を残すための情報
   * - INFO
     - 外部通信ログ
     - | 外部システムとの通信におけるエラー解析
     - 送信受信時間、送受信データなど
   * - WARN
     - 業務エラーログ
     - 業務エラーの記録
     - | 業務エラー発生時間、業務エラーに対応するメッセージIDとメッセージ
       | 入力情報、例外メッセージなど解析に必要な情報
   * - ERROR
     - システムエラーログ
     - システム運用の継続が困難な事象の記録
     - | システムエラー発生時間、システムエラーに対応するメッセージIDとメッセージ
       | 入力情報、例外メッセージなど解析に必要な情報
       | 基本的には、フレームワークが出力し、ビジネスロジックは出力しない。
   * - ERROR
     - 監視ログ
     - 例外発生の監視
     - | 例外発生時間、システムエラーに対応するメッセージID
       | ツールを用いて監視することを考慮し、出力内容は最低限とすること

.. raw:: latex

   \newpage

| デバッグログ、アクセスログ、外部通信ログ、業務エラーログ、システムエラーログは、同一のファイルに出力する。
| 本ガイドラインでは、上記を出力するログファイルを、アプリケーションログと呼ぶこととする。

.. note::
    SLF4JやLogbackのログレベルの順番は、 TRACE < DEBUG < INFO < WARN < ERROR である。
    commons-logginsや、Log4Jで用意されていたFATALレベルは、存在しない。


ログの出力内容
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| ログの出力内容として考慮すべき点を、以下に示す。

1. | ログに出力するIDについて
   | ログを運用で監視する場合は、運用監視で使用するログに、メッセージIDを含めることを推奨する。
   | また、アクセスログを用いて業務量を把握する場合は、集計を容易にするため、メッセージ管理で示しているように、業務ごとに切り分けられるIDをあわせて出力すること。

 .. note::

     ログにIDを含めることにより、ログの可読性が高まるため、システム運用時は、故障解析の一次切り分けの短時間化につながる。
     ログIDの体系は、\ :doc:`../WebApplicationDetail/MessageManagement`\ を参考にすると良い。
     ただし、すべてのログにIDを付与する必要はなく、debug時には、IDは不要である。運用時に、素早く切り分け可能になることを推奨する。

     障害発生時に、ログID(またはメッセージID)を、エラー画面に表示して、システム利用者に通知し、
     利用者に対して、そのIDをコールセンターに通知してもらうような運用にすると、障害解析が容易になる。

     ただし、障害の内容までエラーが画面に表示してしまうと、システムの脆弱性を晒してしまう可能性があるため、注意すること。

     例外が発生した際に、ログや画面にメッセージID(例外コード)を含めるための仕組み(コンポーネント)を共通ライブラリから提供している。
     詳細については、「:doc:`../WebApplicationDetail/ExceptionHandling`」を参照されたい。

2. | トレーサビリティ
   | トレーサビリティ向上のために、各ログにリクエスト単位で、一意となるようなTrack ID(以降X-Trackと呼ぶ)を出力させることを推奨する。
   | X-Trackを含めたログの例を、以下に示す。

 .. code-block:: console

    date:2013-09-06 19:36:31	X-Track:85a437108e9f4a959fd227f07f72ca20	message:[START CONTROLLER] (omitted)
    date:2013-09-06 19:36:31	X-Track:85a437108e9f4a959fd227f07f72ca20	message:[END CONTROLLER  ] (omitted)
    date:2013-09-06 19:36:31	X-Track:85a437108e9f4a959fd227f07f72ca20	message:[HANDLING TIME   ] (omitted)
    date:2013-09-06 19:36:33	X-Track:948c8b9fd04944b78ad8aa9e24d9f263	message:[START CONTROLLER] (omitted)
    date:2013-09-06 19:36:33	X-Track:142ff9674efd486cbd1e293e5aa53a78	message:[START CONTROLLER] (omitted)
    date:2013-09-06 19:36:33	X-Track:142ff9674efd486cbd1e293e5aa53a78	message:[END CONTROLLER  ] (omitted)
    date:2013-09-06 19:36:33	X-Track:142ff9674efd486cbd1e293e5aa53a78	message:[HANDLING TIME   ] (omitted)
    date:2013-09-06 19:36:33	X-Track:948c8b9fd04944b78ad8aa9e24d9f263	message:[END CONTROLLER  ] (omitted)
    date:2013-09-06 19:36:33	X-Track:948c8b9fd04944b78ad8aa9e24d9f263	message:[HANDLING TIME   ] (omitted)

\

   | Track ID を出力させることで、不規則に出力された場合でも、ログを結びつけることができる。
   | 上記の例だと、4行目と8,9行目が、同じリクエストに関するログであることがわかる。
   | 共通ライブラリでは、リクエスト毎のユニークキーを生成し、MDCに追加する\ ``org.terasoluna.gfw.web.logging.mdc.XTrackMDCPutFilter``\ を提供している。
   | \ ``XTrackMDCPutFilter``\ は、HTTPレスポンスヘッダの"X-Track"にもTrack IDを設定する。ログ中では、Track IDのラベルとして、X-Trackを使用している。
   | 使用方法については、\ :ref:`MDCについて<log_MDC>`\ を参照されたい。

3. | ログのマスクについて
   | 個人情報、クレジットカード番号など、
   | ログファイルにそのまま出力すると、セキュリティ上問題のある情報は、必要に応じてマスクすること。

ログの出力ポイント
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. tabularcolumns:: |p{0.15\linewidth}|p{0.85\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 15 85
   :class: longtable

   * - カテゴリ
     - 出力ポイント
   * - | 性能ログ
     - | 業務処理の処理時間を計測し、業務処理実行後に出力したり、リクエストの処理時間を計測し、レスポンスを返す際に、ログを出力する。
       | 通常は、AOPやサーブレットフィルタ等で実装する。
       |
       | 共通ライブラリでは、SpringMVCのControllerのハンドラメソッドの処理時間を、Controllerのハンドラメソッド実行後に、TRACEログで出力する、
       | \ ``org.terasoluna.gfw.web.logging.TraceLoggingInterceptor``\ を提供している。
   * - | デバッグログ
     - | 開発時にデバッグ情報を出力する必要がある場合、ソースコード中に、適宜ログ出力処理を実装する。
       |
       | 共通ライブラリでは、HTTPセッションの生成・破棄・属性追加のタイミングで、DEBUGログを出力するリスナー\ ``org.terasoluna.gfw.web.logging.HttpSessionEventLoggingListener``\ を提供している。
   * - | アクセスログ
     - | リクエストの受付時、レスポンス返却時に、INFOログを出力する。
       | 通常は、AOPやサーブレットフィルタで実装する。
   * - | 外部通信ログ
     - | 外部のシステムと連携前後で、INFOログを出力する。
   * - | 業務エラーログ
     - | 業務例外がスローされたタイミング等で、WARNログを出力する。
       | 通常は、AOPで実装する。
       |
       | 共通ライブラリでは、業務処理実行時に\ `org.terasoluna.gfw.common.exception.BusinessException`\ がスローされた場合に、WARNログを出力する\ ``org.terasoluna.gfw.common.exception.ResultMessagesLoggingInterceptor``\ を提供している。
       | 詳細は  :doc:`../WebApplicationDetail/ExceptionHandling` を参照。
   * - | システムエラーログ
     - | システム例外や、予期せぬ例外が発生した際に、ERRORログを出力する。
       | 通常は、AOPやサーブレットフィルタ等で実装する。
       |
       | 共通ライブラリでは、\ ``org.terasoluna.gfw.web.exception.HandlerExceptionResolverLoggingInterceptor``\ や、
       | \ ``org.terasoluna.gfw.web.exception.ExceptionLoggingFilter``\ を提供している。
       | 詳細は、\ :doc:`../WebApplicationDetail/ExceptionHandling` \ を参照されたい。
   * - 監視ログ
     - 業務エラーログ、システムエラーログの出力タイミングと同様である。

.. raw:: latex

   \newpage

.. note::
    ログを出力する際は、どこで出力されたかわかりやすくなるように、他のログと、全く同じ内容を出力にならないように注意すること。

|

How to use
--------------------------------------------------------------------------------

SLF4J + Logbackでログを出力するには、

#. Logbackの設定
#. SLF4JのAPI呼び出し

が必要である。

Logbackの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Logbackの設定は、クラスパス直下のlogback.xmlに記述する。以下に、設定例を示す。
| logback.xmlの詳細な設定方法については、\ `Logbackの公式マニュアル -Logback configuration- <http://logback.qos.ch/manual/configuration.html>`_\ を参照されたい。

.. note::

     Logbackの設定は、以下のルールによる自動で読み込まれる。

     #. クラスパス上のlogback.grovy
     #. 「1」のファイルが見つからない場合、クラスパス上のlogback-test.xml
     #. 「2」のファイルが見つからない場合、クラスパス上のlogback.xml
     #. 「3」のファイルが見つからない場合、\ ``com.qos.logback.classic.spi.Configurator``\ インタフェースの実装クラスの設定内容 (\ `ServiceLoader <http://docs.oracle.com/javase/8/docs/api/java/util/ServiceLoader.html>`_\ の仕組みを使用して実装クラスを指定)
     #. \ ``Configurator``\ インタフェースの実装クラスが見つからない場合、BasicConfiguratorクラスの設定内容(コンソール出力)

     本ガイドラインでは、logback.xmlをクラスパス上に配置することを推奨する。
     このほか、自動読み込み以外にも、\ `APIによってプログラマティックに読み込んだり <http://logback.qos.ch/manual/configuration.html#joranDirectly>`_\ 、
     \ `システムプロパティで設定ファイルを指定 <http://logback.qos.ch/manual/configuration.html#configFileProperty>`_\ することができる。


logback.xml

.. code-block:: xml

  <?xml version="1.0" encoding="UTF-8"?>
  <configuration>

      <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender"> <!-- (1) -->
          <encoder>
              <pattern><![CDATA[date:%d{yyyy-MM-dd HH:mm:ss}\tthread:%thread\tX-Track:%X{X-Track}\tlevel:%-5level\tlogger:%-48logger{48}\tmessage:%msg%n]]></pattern> <!-- (2) -->
          </encoder>
      </appender>

      <appender name="APPLICATION_LOG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender"> <!-- (3) -->
          <file>${app.log.dir:-log}/projectName-application.log</file> <!-- (4) -->
          <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
              <fileNamePattern>${app.log.dir:-log}/projectName-application-%d{yyyyMMddHH}.log</fileNamePattern> <!-- (5) -->
              <maxHistory>7</maxHistory> <!-- (6) -->
          </rollingPolicy>
          <encoder>
              <charset>UTF-8</charset> <!-- (7) -->
              <pattern><![CDATA[date:%d{yyyy-MM-dd HH:mm:ss}\tthread:%thread\tX-Track:%X{X-Track}\tlevel:%-5level\tlogger:%-48logger{48}\tmessage:%msg%n]]></pattern>
          </encoder>
      </appender>

      <appender name="MONITORING_LOG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender"> <!-- (8) -->
          <file>${app.log.dir:-log}/projectName-monitoring.log</file>
          <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
              <fileNamePattern>${app.log.dir:-log}/projectName-monitoring-%d{yyyyMMdd}.log</fileNamePattern>
              <maxHistory>7</maxHistory>
          </rollingPolicy>
          <encoder>
              <charset>UTF-8</charset>
              <pattern><![CDATA[date:%d{yyyy-MM-dd HH:mm:ss}\tX-Track:%X{X-Track}\tlevel:%-5level\tmessage:%msg%n]]></pattern>
          </encoder>
      </appender>

      <!-- Application Loggers -->
      <logger name="com.example.sample"> <!-- (9) -->
          <level value="debug" />
      </logger>

      <!-- TERASOLUNA -->
      <logger name="org.terasoluna.gfw">
          <level value="info" />
      </logger>
      <logger name="org.terasoluna.gfw.web.logging.TraceLoggingInterceptor">
          <level value="trace" />
      </logger>
      <logger name="org.terasoluna.gfw.common.exception.ExceptionLogger">
          <level value="info" />
      </logger>
      <logger name="org.terasoluna.gfw.common.exception.ExceptionLogger.Monitoring" additivity="false"><!-- (10) -->
          <level value="error" />
          <appender-ref ref="MONITORING_LOG_FILE" />
      </logger>

      <!-- 3rdparty Loggers -->
      <logger name="org.springframework">
          <level value="warn" />
      </logger>

      <logger name="org.springframework.web.servlet">
          <level value="info" />
      </logger>

      <!--  REMOVE THIS LINE IF YOU USE JPA
      <logger name="org.hibernate.engine.transaction">
          <level value="debug" />
      </logger>
            REMOVE THIS LINE IF YOU USE JPA  -->
      <!--  REMOVE THIS LINE IF YOU USE MyBatis3
      <logger name="org.springframework.jdbc.datasource.DataSourceTransactionManager">
          <level value="debug" />
      </logger>
            REMOVE THIS LINE IF YOU USE MyBatis3  -->

      <logger name="jdbc.sqltiming">
          <level value="debug" />
      </logger>

      <!-- only for development -->
      <logger name="jdbc.resultsettable">
          <level value="debug" />
      </logger>

      <root level="warn"> <!-- (11) -->
          <appender-ref ref="STDOUT" /> <!-- (12) -->
          <appender-ref ref="APPLICATION_LOG_FILE" />
      </root>

  </configuration>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90
   :class: longtable

   * - 項番
     - 説明
   * - | (1)
     - | コンソールにログを出力するための、アペンダ定義を指定する。
       | 出力先を標準出力にするか、標準エラーにするか選べるが、指定しない場合は、標準出力となる。
   * - | (2)
     - | ログの出力形式を指定する。何も記述しなければ、メッセージだけが出力される。
       | 時刻やメッセージレベルなど、業務要件に合わせて出力させる。
       | ここでは"ラベル:値<TAB>ラベル:値<TAB>..."形式のLTSV(Labeled Tab Separated Value)フォーマットを設定している。
   * - | (3)
     - | アプリケーションログを出力するための、アペンダ定義を指定する。
       | どのアペンダを使用するかは、<logger>に指定することもできるが、ここではアプリケーションログはデフォルトで使用するため、root（11）に参照させている。
       | アプリケーションログを出力する際によく使用されるのは、RollingFileAppenderであるが、ログのローテーションをlogrotateなど別機能で実施する場合、FileAppenderを使用することもある。
   * - | (4)
     - | カレントファイル名(出力中のログのファイル名)を指定する。固定のファイル名としたい場合は指定すること。
       | <file>ログファイル名</file>を指定しないと、(5)のパターンの名称で出力される。
   * - | (5)
     - | ローテーション後のファイル名を指定する。通常は、日付か時間の形式が、多く採用される。
       | 誤ってHHをhhと設定してしまうと、24時間表記されないため注意すること。
   * - | (6)
     - | ローテーションしたファイルをいくつ残すかを指定する。
   * - | (7)
     - | ログファイルの文字コードを指定する。
   * - | (8)
     - | デフォルトでアプリケーションログが出力されるように設定する。
   * - | (9)
     - | ロガー名は、com.example.sample以下のロガーが、debugレベル以上のログを出力するように設定する。
   * - | (10)
     - | 監視ログの設定を行う。\ :doc:`../WebApplicationDetail/ExceptionHandling`\ の\ :ref:`exception-handling-how-to-use-application-configuration-common-label`\ を参照されたい。

       .. warning:: **additivityの設定値について**

           \ ``false``\ を指定すること。\ ``true``\ (デフォルト値)を指定すると、上位のロガー(例えば、root)によって、同じログが出力されてしまう。
           具体的には、監視ログは3つのアペンダー(\ ``MONITORING_LOG_FILE``\、\ ``STDOUT``\、\ ``APPLICATION_LOG_FILE``\)によって出力される。

   * - | (11)
     - | <logger>の指定が無いロガーが、warnレベル以上のログを出力するように設定する。
   * - | (12)
     - | デフォルトでConsoleAppender, RollingFileAppender(アプリケーションログ)が使用されるように設定する。

.. raw:: latex

   \newpage

.. tip:: **LTSV(Labeled Tab Separated Value)について**

    \ `LTSV <http://ltsv.org/>`_\ は、テキストデータのフォーマットの一つであり、主にログのフォーマットとして使用される。

    LTSVは、

    * フィールドの区切り文字としてタブを使用することで、他の区切り文字に比べてフィールドを分割しやすい。
    * フィールドにラベル(名前)を設けることで、フィールド定義の変更(定義位置の変更、フィールドの追加、フィールドの削除)を行ってもパース処理には影響を与えない。

    また、エクセルに貼付けるだけで最低限のフォーマットが行える点も特徴の一つである。

|

logback.xmlで設定するものは、次の3つになる。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 80

   * - 種類
     - 概要
   * - appender
     - 「どの場所に」「どんなレイアウト」で出力するのか
   * - root
     - デフォルトでは、「どのログレベル」以上で「どのappender」に出力するのか
   * - logger
     - 「どのロガー(パッケージやクラス等)」は、「どのログレベル」以上で出力するのか

|

<appender>要素には、「どの場所に」「どんなレイアウト」で出力するのかを定義する。
appenderを定義しただけではログ出力の際に使用されず、
<logger>要素や<root>要素に参照されると、初めて使用される。
属性は、nameとclassの2つで、共に必須である。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 80

   * - 属性
     - 概要
   * - name
     - appenderの名前。appender-refで指定される。好きな名前をつけてよい。
   * - class
     - appender実装クラスのFQCN。

|

提供されている主なappenderを、以下に示す

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 30 70

   * - Appender
     - 概要
   * - `ConsoleAppender <http://logback.qos.ch/manual/appenders.html#ConsoleAppender>`_
     - コンソール出力
   * - `FileAppender <http://logback.qos.ch/manual/appenders.html#FileAppender>`_
     - ファイル出力
   * - `RollingFileAppender <http://logback.qos.ch/manual/appenders.html#RollingFileAppender>`_
     - ファイル出力(ローリング可能)
   * - `AsyncAppender <http://logback.qos.ch/manual/appenders.html#AsyncAppender>`_
     - 非同期出力。性能を求められる処理中のロギングに使用する。（出力先は、他のAppenderで設定する必要がある。）

Appenderの詳細な種類は、\ `Logbackの公式マニュアル -Appenders- <http://logback.qos.ch/manual/appenders.html>`_\ を参照されたい。

|

SLF4JのAPI呼び出しによる基本的なログ出力
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

SLF4Jのロガー(\ ``org.slf4j.Logger``\ )の各ログレベルに応じたメソッドを呼び出してログを出力する。

.. code-block:: java

    package com.example.sample.app.welcome;

    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;

    @Controller
    public class HomeController {

        private static final Logger logger = LoggerFactory
                .getLogger(HomeController.class);   // (1)

        @RequestMapping(value = "/", method = { RequestMethod.GET,
                RequestMethod.POST })
        public String home(Model model) {
            logger.trace("This log is trace log."); // (2)
            logger.debug("This log is debug log."); // (3)
            logger.info("This log is info log.");   // (4)
            logger.warn("This log is warn log.");   // (5)
            logger.error("This log is error log."); // (6)
            return "welcome/home";
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | \ ``org.slf4j.LoggerFactory``\ から\ ``Logger``\ を生成する。\ ``getLogger``\ の引数にClassオブジェクトを
       | 設定した場合は、ロガー名は、そのクラスのFQCNになる。
       | この例では、"com.example.sample.app.welcome.HomeController"が、ロガー名になる。
   * - | (2)
     - | TRACEレベルのログを出力する。
   * - | (3)
     - | DEBUGレベルのログを出力する。
   * - | (4)
     - | INFOレベルのログを出力する。
   * - | (5)
     - | WARNレベルのログを出力する。
   * - | (6)
     - | ERRORレベルのログを出力する。


ログの出力結果を、以下に示す。このcom.example.sampleのログレベルは、DEBUGなので、TRACEログは出力されない。

.. code-block:: console

    date:2013-11-06 20:13:05    thread:tomcat-http--3 X-Track:5844f073b7434b67a875cb85b131e686    level:DEBUG logger:com.example.sample.app.welcome.HomeController    message:This log is debug log.
    date:2013-11-06 20:13:05    thread:tomcat-http--3 X-Track:5844f073b7434b67a875cb85b131e686    level:INFO  logger:com.example.sample.app.welcome.HomeController    message:This log is info log.
    date:2013-11-06 20:13:05    thread:tomcat-http--3 X-Track:5844f073b7434b67a875cb85b131e686    level:WARN  logger:com.example.sample.app.welcome.HomeController    message:This log is warn log.
    date:2013-11-06 20:13:05    thread:tomcat-http--3 X-Track:5844f073b7434b67a875cb85b131e686    level:ERROR logger:com.example.sample.app.welcome.HomeController    message:This log is error log.

ログメッセージのプレースホルダに引数を埋め込む場合は、次のように記述すればよい。

.. code-block:: java

    int a = 1;
    logger.debug("a={}", a);
    String b = "bbb";
    logger.debug("a={}, b={}", a, b);

以下のようなログが出力される。


.. code-block:: console

    date:2013-11-06 20:32:45    thread:tomcat-http--3   X-Track:853aa701a401404a87342a574c69efbc    level:DEBUG logger:com.example.sample.app.welcome.HomeController    message:a=1
    date:2013-11-06 20:32:45    thread:tomcat-http--3   X-Track:853aa701a401404a87342a574c69efbc    level:DEBUG logger:com.example.sample.app.welcome.HomeController    message:a=1, b=bbb

.. warning::

     \ ``logger.debug("a=" + a + " , b=" + b);``\ というように、文字列連結を行わないように注意すること。

例外をキャッチする際は、
以下のようにERRORログ(場合によってはWARNログ)を出力し、ログメソッドにエラーメッセージと発生した例外を渡す。

.. code-block:: java

    public String home(Model model) {
        // omitted

        try {
            throwException();
        } catch (Exception e) {
            logger.error("Exception happend!", e);
            // omitted
        }
        // omitted
    }

    public void throwException() throws Exception {
        throw new Exception("Test Exception!");
    }

これにより、起因例外のスタックトレースが出力され、エラーの原因を解析しやすくなる。

.. code-block:: console

    date:2013-11-06 20:38:04    thread:tomcat-http--5   X-Track:11d7dbdf64e44782822c5aea4fc4bb4f    level:ERROR logger:com.example.sample.app.welcome.HomeController    message:Exception happend!
    java.lang.Exception: Test Exception!
        at com.example.sample.app.welcome.HomeController.throwException(HomeController.java:40) ~[HomeController.class:na]
        at com.example.sample.app.welcome.HomeController.home(HomeController.java:31) ~[HomeController.class:na]
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.7.0_40]
        (omitted)

ただし、以下のようにキャッチした例外を別の例外にラップして、上位に再スローする場合はログを出力しなくてもよい。通常は上位でエラーログが出力されるためである。

.. code-block:: java

    try {
        throwException();
    } catch (Exception e) {
        throw new SystemException("e.ex.fw.9001", e);
        // no need to log
    }

\
 .. note::

     起因例外をログメソッドに渡す場合は、プレースホルダーを使用できない。この場合に限り、
     メッセージの引数を文字列で連結してもよい。

       .. code-block:: java

           try {
               throwException();
           } catch (Exception e) {
               // NG => logger.error("Exception happend! [a={} , b={}]", e, a, b);
               logger.error("Exception happend! [a=" + a + " , b=" + b + "]", e);
               // omitted
           }

.. _note-description-of-log-output:

ログ出力の記述の注意点
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

SLF4JのLoggerは、内部でログレベルのチェックを行い、必要なレベルの場合にのみ実際にログを出力する。

したがって、次のようなログレベルのチェックは、基本的に不要である。

.. code-block:: java

    if (logger.isDebugEnabled()) {
        logger.debug("This log is Debug.");
    }

    if (logger.isDebugEnabled()) {
        logger.debug("a={}", a);
    }


ただし、次の場合は性能劣化を防ぐために、ログレベルのチェックを行うこと。


#. 引数が3個以上の場合

    ログメッセージの引数が3以上の場合、SLF4JのAPIでは引数の配列を渡す必要がある。配列生成のコストを避けるため、
    ログレベルのチェックを行い、必要なときのみ、配列が生成されるようにすること。


    .. code-block:: java

        if (logger.isDebugEnabled()) {
            logger.debug("a={}, b={}, c={}", new Object[] { a, b, c });
        }

#. 引数の生成にメソッド呼び出しが必要な場合

    ログメッセージの引数を生成する際にメソッド呼び出しが必要な場合、メソッド実行コストを避けるため、
    ログレベルのチェックを行い、必要なときのみメソッドが実行されるようにすること。

    .. code-block:: java

        if (logger.isDebugEnabled()) {
            logger.debug("xxx={}", foo.getXxx());
        }



How to extend
--------------------------------------------------------------------------------
ログ出力仕様は監視製品や要件等で独自の規定があるケースが多く、個別に実装するケースが想定される。ここでは、以下の2例を説明する。

#. ログメッセージの一元管理
#. ログメッセージの出力フォーマットの統一

ログメッセージの一元管理
^^^^^^^^^^^^^^^^^^^^^^^^^
| ログメッセージの一元管理によるメンテナンス性向上等を目的とした実装例を紹介する。
| ログメッセージの一元管理は、ログメッセージをプロパティファイル等の別ファイルにまとめ、ログ出力時にメッセージ解決を行うことで実現できる。
| ここでは実装例として、ログ出力メソッドの引数にログIDを設定できるようにし、プロパティファイルの中のログIDに対応するメッセージを出力する方法を説明する。

 .. note::

     ログIDとログメッセージの管理方法は、Javaのenumを用いてまとめる方法も存在するが、本ガイドラインでは一般的なプロパティファイルを用いた方法を紹介する。

本実装例では

#. Loggerラッパークラス
#. プロパティファイル

| を作成することで実現する。
| ここではLoggerラッパークラスを\ ``LogIdBasedLogger``\、プロパティファイルを\ ``log-messages.properties``\とする。

- `LogIdBasedLogger`  (Loggerラッパークラス)

.. code-block:: java

    package com.example.sample.common.logger;

    import java.text.MessageFormat;
    import java.util.Arrays;
    import java.util.Locale;

    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.context.NoSuchMessageException;
    import org.springframework.context.support.ResourceBundleMessageSource;

    public class LogIdBasedLogger {

        private static final String UNDEFINED_MESSAGE_FORMAT = "UNDEFINED-MESSAGE id:{0} arg:{1}";   // (1)

        private static ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();// (2)

        static {    // (3)
            messageSource.setDefaultEncoding("UTF-8");          // (4)
            messageSource.setBasenames("i18n/log-messages");    // (5)
        }

        private final Logger logger;

        private LogIdBasedLogger(Class<?> clazz) {
            logger = LoggerFactory.getLogger(clazz);            // (6)
        }

        public static LogIdBasedLogger getLogger(Class<?> clazz) {
            return new LogIdBasedLogger(clazz);
        }

        public boolean isDebugEnabled() {                       // (7)
            return logger.isDebugEnabled();
        }

        public void debug(String format, Object... args) {
            logger.debug(format, args);                         // (8)
        }

        public void info(String id, Object... args) {
            if (logger.isInfoEnabled()) {
                logger.info(createLogMessage(id, args));        // (9)
            }
        }

        public void warn(String id, Object... args) {
            if (logger.isWarnEnabled()) {
                logger.warn(createLogMessage(id, args));        // (9)
            }
        }

        public void error(String id, Object... args) {
            if (logger.isErrorEnabled()) {
                logger.error(createLogMessage(id, args));       // (9)
            }
        }

        public void trace(String id, Object... args) {
            if (logger.isTraceEnabled()) {
                logger.trace(createLogMessage(id, args));       // (9)
            }
        }

        public void warn(String id, Throwable t, Object... args) {
            if (logger.isWarnEnabled()) {
                logger.warn(createLogMessage(id, args), t);     // (9)
            }
        }

        public void error(String id, Throwable t, Object... args) {
            if (logger.isErrorEnabled()) {
                logger.error(createLogMessage(id, args), t);    // (9)
            }
        }

        private String createLogMessage(String id, Object... args) {
            return getMessage(id, args);
        }
        
        private String getMessage(String id, Object... args) {
            String message;
            try {
                message = messageSource.getMessage(id, args, Locale
                        .getDefault());
            } catch (NoSuchMessageException e) {                // (10)
                message = MessageFormat.format(UNDEFINED_MESSAGE_FORMAT, id, Arrays
                        .toString(args));
            }
            return message;
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
     - | ログID未定義時のログメッセージ。ここでは例として \ ``org.terasoluna.gfw.common.exception.ExceptionLogger``\ と同じメッセージを使用する。
   * - | (2)
     - | \ ``MessageSource``\ でログメッセージを取得する実装例。
       | メッセージデータを管理する \ ``MessageSource``\ は、汎用性を高めるため\ ``static``\ 領域に格納している。
       | このような実装をすることでDIコンテナへのアクセス可否に依存しなくなるため、Loggerラッパークラスをいつでも使用することができるようになる。
   * - | (3)
     - | staticイニシャライザにて\ ``MessageSource``\ を生成する。
       | 本実装では\ ``i18n``\に配置した\ ``log-messages.properties``\ を読み込む。
   * - | (4)
     - | プロパティファイルをパースする際に使用する文字コードを設定する。
       | 本実装ではプロパティファイルはUTF-8エンコードとしたのでUTF-8を指定する。
       | 詳細は、\ :doc:`../../ArchitectureInDetail/WebApplicationDetail/MessageManagement`\ の\ :ref:`properties-display`\ を参照されたい。
   * - | (5)
     - | 国際化を考慮し\ ``setBasenames``\ メソッドを使用してプロパティファイルを指定する。
       | \ ``setBasenames``\ の詳細は\ `ReloadableResourceBundleMessageSourceクラスのsetBasenamesのJavaDoc <http://docs.spring.io/spring/docs/4.3.5.RELEASE/javadoc-api/org/springframework/context/support/ReloadableResourceBundleMessageSource.html#setBasenames-java.lang.String...->`_\を参照されたい。
   * - | (6)
     - | Loggerラッパークラスにおいても、SLF4Jを使用する。ロギングライブラリの実装を直接使用しない。
   * - | (7)
     - | DEBUGレベルのログ出力を許可してるか、判定する。
       | 使用時の注意点については、\ :ref:`note-description-of-log-output`\ を参照されたい。
   * - | (8)
     - | 本実装例ではDEBUGレベルのログにはログIDを使わない。引数のログメッセージをそのまま、ログ出力する。
   * - | (9)
     - | TRACE/INFO/WARN/ERRORレベルのログはログIDに該当するログメッセージをプロパティファイルから取得して、ログ出力する。
   * - | (10)
     - | getMessageを呼び出す際にプロパティファイルにログIDが記載されていないと例外:\ ``NoSuchMessageException``\ が発生する。
       | そのため\ ``NoSuchMessageException``\ をcatchし、ログIDがプロパティファイルに定義されていない旨のログメッセージを出力する。

.. raw:: latex

   \newpage

- `log-messages.properties`  (プロパティファイル)

.. code-block:: console

    i.ab.cd.1001 = This message is Info-Level. {0}
    w.ab.cd.2001 = This message is Warn-Level. {0}
    e.ab.cd.3001 = This message is Error-Level. {0}
    t.ab.cd.4001 = This message is Trace-Level. {0}

\

 .. note::

     本ガイドラインでは、 画面出力用メッセージとログ出力用メッセージを別々に管理するため、新たにプロパティファイルを作成しているが1ファイルにしてもかまわない。
     
     アプリケーションの性質やメッセージの管理方法に合わせてファイルの単位を決めること。



実行結果は、以下のようになる。


- 呼び出しサンプル

.. code-block:: java

    package com.example.sample.app.welcome;

    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;

    import com.example.sample.common.logger.LogIdBasedLogger;

    @Controller
    public class HomeController {

        private static final LogIdBasedLogger logger = LogIdBasedLogger
                .getLogger(HomeController.class);

        @RequestMapping(value = "/", method = { RequestMethod.GET,
                RequestMethod.POST })
        public String home(Model model) {
            logger.debug("debug log");
            logger.info("i.ab.cd.1001","replace_value_1");
            logger.warn("w.ab.cd.2001","replace_value_2");
            logger.error("e.ab.cd.3001","replace_value_3");
            logger.trace("t.ab.cd.4001","replace_value_4");
            logger.info("i.ab.cd.1002","replace_value_5");
            return "welcome/home";
        }
    }


- ログ出力例

.. code-block:: console

    date:2016-05-30 17:34:18.590  thread:http-bio-8080-exec-3  X-Track:e2a65cd9160b48d6aaeb63fe6e751c6b  level:DEBUG  logger:com.example.sample.app.welcome.HomeController   message:debug log
    date:2016-05-30 17:34:18.590  thread:http-bio-8080-exec-3  X-Track:e2a65cd9160b48d6aaeb63fe6e751c6b  level:INFO   logger:com.example.sample.app.welcome.HomeController   message:This message is Info-Level. replace_value_1
    date:2016-05-30 17:34:18.590  thread:http-bio-8080-exec-3  X-Track:e2a65cd9160b48d6aaeb63fe6e751c6b  level:WARN   logger:com.example.sample.app.welcome.HomeController   message:This message is Warn-Level. replace_value_2
    date:2016-05-30 17:34:18.590  thread:http-bio-8080-exec-3  X-Track:e2a65cd9160b48d6aaeb63fe6e751c6b  level:ERROR  logger:com.example.sample.app.welcome.HomeController   message:This message is Error-Level. replace_value_3
    date:2016-05-30 17:34:18.590  thread:http-bio-8080-exec-3  X-Track:e2a65cd9160b48d6aaeb63fe6e751c6b  level:TRACE  logger:com.example.sample.app.welcome.HomeController   message:This message is Trace-Level. replace_value_4
    date:2016-05-30 17:34:18.590  thread:http-bio-8080-exec-3  X-Track:e2a65cd9160b48d6aaeb63fe6e751c6b  level:INFO   logger:com.example.sample.app.welcome.HomeController   message:UNDEFINED-MESSAGE id:i.ab.cd.1002 arg:[replace_value_5]


ログメッセージの出力フォーマットの統一
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| ログメッセージの出力フォーマットは、下表のとおりログ出力の方式ごとで異なる。
| そのため出力ログフォーマットの統一には、ログ出力フォーマットをもう一方のフォーマットに合わせる、または、両方とも独自のフォーマットに統一する必要がある。
| 本ガイドラインでは、業務ロジックで出力するログにフォーマットを定める例と、両方とも独自のフォーマット（[{例外コード(メッセージID)またはログID}], {メッセージまたはログメッセージ}）に統一する例を説明する。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.30\linewidth}|p{0.30\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 30 30 30

   * - 項番
     - ログ出力方式
     - 該当ログ
     - デフォルトフォーマット
   * - | (1)
     - | 業務ロジックで明示的にログを出力
     - | アクセスログ・外部通信ログなど
     - | なし
   * - | (2)
     - | フレームワークが例外を検知して暗黙的にログを出力
     - | 業務エラーログ・システムエラーログなど
     - | [{例外コード(メッセージID)}] {メッセージ}

.. note::

     \ :ref:`共通ライブラリ<\exception-handling-about-classes-of-library-label>` の例外ハンドリングの仕組みにより、例外発生時に出力される「業務エラーログ」および「システムエラーログ」は上記の表のデフォルトフォーマットで出力される。

フレームワークが例外を検知して出力するログのフォーマットに統一
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 業務ロジックで出力するログをフレームワークが例外を検知して出力するログのフォーマットに合わせるための実装例を示す。
| 本ガイドラインではLoggerラッパークラス(\ ``LogIdBasedLogger`` \)に、フォーマットを行う処理を追加して実現する。

.. code-block:: java

    package com.example.sample.common.logger;

    import java.text.MessageFormat; // (1)

    // omitted

    public class LogIdBasedLogger {

        private static final String LOG_MESSAGE_FORMAT = "[{0}] {1}"; // (2)

        // omitted

        private String createLogMessage(String id, String... args) {
            return MessageFormat.format(LOG_MESSAGE_FORMAT, id, getMessage(id,
                    args)); // (1)
        }

        // omitted

    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | ログメッセージフォーマットを元にログメッセージを作成する処理を追加する
   * - | (2)
     - | フォーマットを定義する。
       | \ ``{0}``\ はログID、\ ``{1}``\ はログメッセージがリプレースされる。


実行結果は、以下のようになる。

.. code-block:: console

  date:2016-05-30 16:32:33.239  thread:http-bio-8080-exec-4  X-Track:4f61314a51524ab3a41832b0ceae7119  level:DEBUG  logger:com.example.sample.app.welcome.HomeController   message:debug log
  date:2016-05-30 16:32:33.239  thread:http-bio-8080-exec-4  X-Track:4f61314a51524ab3a41832b0ceae7119  level:INFO   logger:com.example.sample.app.welcome.HomeController   message:[i.ab.cd.1001] This message is Info-Level. replace_value_1
  date:2016-05-30 16:32:33.239  thread:http-bio-8080-exec-4  X-Track:4f61314a51524ab3a41832b0ceae7119  level:WARN   logger:com.example.sample.app.welcome.HomeController   message:[w.ab.cd.2001] This message is Warn-Level. replace_value_2
  date:2016-05-30 16:32:33.239  thread:http-bio-8080-exec-4  X-Track:4f61314a51524ab3a41832b0ceae7119  level:ERROR  logger:com.example.sample.app.welcome.HomeController   message:[e.ab.cd.3001] This message is Error-Level. replace_value_3
  date:2016-05-30 17:34:18.590  thread:http-bio-8080-exec-3  X-Track:4f61314a51524ab3a41832b0ceae7119  level:TRACE  logger:com.example.sample.app.welcome.HomeController   message:[t.ab.cd.4001] This message is Trace-Level. replace_value_4
  date:2016-05-30 16:32:33.239  thread:http-bio-8080-exec-4  X-Track:4f61314a51524ab3a41832b0ceae7119  level:INFO   logger:com.example.sample.app.welcome.HomeController   message:[i.ab.cd.1002] UNDEFINED-MESSAGE id:i.ab.cd.1002 arg:[replace_value_5]

独自のフォーマットに統一
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 業務ロジックとフレームワークが出力するログを独自のフォーマット（[{例外コード(メッセージID)またはログID}], {メッセージまたはログメッセージ}）に統一する実装例を示す。

業務ロジックで出力するログにフォーマットを定義
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

| 業務ロジックで出力するログを前述のフォーマットで出力する例を示す。
| 本ガイドラインではLoggerラッパークラス(\ ``LogIdBasedLogger`` \)に、フォーマットを行う処理を追加して実現する。

.. code-block:: java

    package com.example.sample.common.logger;

    import java.text.MessageFormat; // (1)

    // omitted

    public class LogIdBasedLogger {

        private static final String LOG_MESSAGE_FORMAT = "[{0}], {1}"; // (2)

        // omitted

        private String createLogMessage(String id, String... args) {
            return MessageFormat.format(LOG_MESSAGE_FORMAT, id, getMessage(id,
                    args)); // (1)
        }

        // omitted

    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | ログメッセージフォーマットを元にログメッセージを作成する処理を追加する
   * - | (2)
     - | フォーマットを定義する。
       | \ ``{0}``\ はログID、\ ``{1}``\ はログメッセージがリプレースされる。


実行結果は、以下のようになる。

.. code-block:: console

  date:2016-05-30 16:32:33.239  thread:http-bio-8080-exec-4  X-Track:4f61314a51524ab3a41832b0ceae7119  level:DEBUG  logger:com.example.sample.app.welcome.HomeController   message:debug log
  date:2016-05-30 16:32:33.239  thread:http-bio-8080-exec-4  X-Track:4f61314a51524ab3a41832b0ceae7119  level:INFO   logger:com.example.sample.app.welcome.HomeController   message:[i.ab.cd.1001], This message is Info-Level. replace_value_1
  date:2016-05-30 16:32:33.239  thread:http-bio-8080-exec-4  X-Track:4f61314a51524ab3a41832b0ceae7119  level:WARN   logger:com.example.sample.app.welcome.HomeController   message:[w.ab.cd.2001], This message is Warn-Level. replace_value_2
  date:2016-05-30 16:32:33.239  thread:http-bio-8080-exec-4  X-Track:4f61314a51524ab3a41832b0ceae7119  level:ERROR  logger:com.example.sample.app.welcome.HomeController   message:[e.ab.cd.3001], This message is Error-Level. replace_value_3
  date:2016-05-30 17:34:18.590  thread:http-bio-8080-exec-3  X-Track:4f61314a51524ab3a41832b0ceae7119  level:TRACE  logger:com.example.sample.app.welcome.HomeController   message:[t.ab.cd.4001], This message is Trace-Level. replace_value_4
  date:2016-05-30 16:32:33.239  thread:http-bio-8080-exec-4  X-Track:4f61314a51524ab3a41832b0ceae7119  level:INFO   logger:com.example.sample.app.welcome.HomeController   message:[i.ab.cd.1002], UNDEFINED-MESSAGE arg:[replace_value_5]



フレームワークが出力するログのフォーマットを変更
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

| フレームワークが出力するログを前述のフォーマットで出力する例を示す。
| 業務エラーログやシステムエラーログのフォーマットを変更するには、\ ``applicationContext.xml``\ の\ ``ExceptionLogger``\ のbean定義を変更する。
| 以下に、\ ``ExceptionLogger``\ の定義の例を挙げる。

- **applicationContext.xml**

.. code-block:: xml

    <!-- Exception Logger. -->
    <bean id="exceptionLogger"
        class="org.terasoluna.gfw.common.exception.ExceptionLogger">
        <property name="exceptionCodeResolver" ref="exceptionCodeResolver" />
        <property name="logMessageFormat" value="[{0}], {1}" />    <!-- (1) -->
    </bean>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``logMessageFormat``\ にフォーマットを定義する。
       | \ ``{0}``\ は例外コード(メッセージID)、\ ``{1}``\ はメッセージがリプレースされる。

実行結果は、以下のようになる。

.. code-block:: console

    date:2013-09-19 21:03:06   thread:tomcat-http--3   X-Track:c19eec546b054d54a13658f94292b24f    level:ERROR logger:o.t.gfw.common.exception.ExceptionLogger         message:[e.ad.od.9012], not found item entity. item code [10-123456].
    ...
    // stackTarace omitted


Appendix
--------------------------------------------------------------------------------

.. _log_MDC:

MDCの使用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| \ `MDC <http://logback.qos.ch/manual/mdc.html>`_\ (Mapped Diagnostic Context)を利用することで、横断的なログ出力が可能となる。
| 1リクエスト中に出力されるログに、同じ情報(ユーザー名やリクエストで一意なID)を
| 埋め込んで出力することにより、ログのトレーサビリティが向上する。

| MDCは、スレッドローカルなMapを内部にもち、キーに対して値をputする。removeされるまで、ログにputした値を出力することができる。
| Filterなどでリクエストの先頭でputし、処理終了時にremoveすればよい。


基本的な使用方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 次に、MDCを用いた例を挙げる。

.. code-block:: java

    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.slf4j.MDC;

    public class Main {

        private static final Logger logger = LoggerFactory.getLogger(Main.class);

        public static void main(String[] args) {
            String key = "MDC_SAMPLE";
            MDC.put(key, "sample"); // (1)
            try {
                logger.debug("debug log");
                logger.info("info log");
                logger.warn("warn log");
                logger.error("error log");
            } finally {
                MDC.remove(key); // (2)
            }
            logger.debug("mdc removed!");
        }
    }


logback.xmlの\ ``<pattern>``\ に\ ``%X{キー名}``\ 形式で出力フォーマットを定義することで、
MDCに追加した値をログに出力できる。

.. code-block:: xml

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern><![CDATA[date:%d{yyyy-MM-dd HH:mm:ss}\tthread:%thread\tmdcSample:%X{MDC_SAMPLE}\tlevel:%-5level\t\tmessage:%msg%n]]></pattern>
        </encoder>
    </appender>

実行結果は、以下のようになる。

.. code-block:: console

    date:2013-11-08 17:45:48    thread:main mdcSample:sample    level:DEBUG     message:debug log
    date:2013-11-08 17:45:48    thread:main mdcSample:sample    level:INFO      message:info log
    date:2013-11-08 17:45:48    thread:main mdcSample:sample    level:WARN      message:warn log
    date:2013-11-08 17:45:48    thread:main mdcSample:sample    level:ERROR     message:error log
    date:2013-11-08 17:45:48    thread:main mdcSample:  level:DEBUG     message:mdc removed!

\
 .. note::

    \ ``MDC.clear()``\ を実行すると、追加したすべての値が削除される。

FilterでMDCに値をPutする
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""


| 共通ライブラリからはFilterでMDCへ値の追加・削除するためのベースクラスとして、\ ``org.terasoluna.gfw.web.logging.mdc.AbstractMDCPutFilter``\
| を提供している。またその実装クラスとして、

* リクエスト毎にユニークなIDをMDCに設定する\ ``org.terasoluna.gfw.web.logging.mdc.XTrackMDCPutFilter``
* Spring Securityの認証ユーザ名をMDCに設定する\ ``org.terasoluna.gfw.security.web.logging.UserIdMDCPutFilter``

| を提供している。

| Filterで独自の値をMDCに追加したい場合は\ ``org.terasoluna.gfw.web.logging.mdc.XTrackMDCPutFilter``\ の実装を参考に
| ``AbstractMDCPutFilter``\ を実装すればよい。

MDCFilterの使用方法

web.xmlのfilter定義にMDCFilterの定義を追加する。

.. code-block:: xml

    <!-- omitted -->

    <!-- (1) -->
    <filter>
        <filter-name>MDCClearFilter</filter-name>
        <filter-class>org.terasoluna.gfw.web.logging.mdc.MDCClearFilter</filter-class>
    </filter>

    <filter-mapping>
        <filter-name>MDCClearFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!-- (2) -->
    <filter>
        <filter-name>XTrackMDCPutFilter</filter-name>
        <filter-class>org.terasoluna.gfw.web.logging.mdc.XTrackMDCPutFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>XTrackMDCPutFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!-- (3) -->
    <filter>
        <filter-name>UserIdMDCPutFilter</filter-name>
        <filter-class>org.terasoluna.gfw.security.web.logging.UserIdMDCPutFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>UserIdMDCPutFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!-- omitted -->


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | MDCの内容をクリアする\ ``MDCClearFilter``\ を設定する。
       | 各種\ ``MDCPutFilter``\ が追加したMDCへの値を、このFilterが消去する。
   * - | (2)
     - | \ ``XTrackMDCPutFilter``\ を設定する。\ ``XTrackMDCPutFilter``\ はキー\ "X-Track"\ にリクエストIDをputする。
   * - | (3)
     - | \ ``UserIdMDCPutFilter``\ を設定する。\ ``UserIdMDCPutFilter``\ はキー\ "USER"\ にユーザーIDをputする。
       |

\ ``MDCClearFilter``\ は以下のシーケンス図のように、後処理としてMDCの内容をクリアするため、
各種\ ``MDCPutFilter``\ よりも、先に定義すること。

.. figure:: ./images_Logging/logging-mdcput-sequence.png
   :width: 80%


logback.xmlの\ ``<pattern>``\ に\ ``%X{X-Track}``\ および、\ ``%X{USER}``\ を追加することで、リクエストIDとユーザーIDをログに出力することができる。

.. code-block:: xml

    <!-- omitted -->
    <appender name="APPLICATION_LOG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${app.log.dir:-log}/projectName-application.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${app.log.dir:-log}/projectName-application-%d{yyyyMMdd}.log</fileNamePattern>
            <maxHistory>7</maxHistory>
        </rollingPolicy>
        <encoder>
            <charset>UTF-8</charset>
            <pattern><![CDATA[date:%d{yyyy-MM-dd HH:mm:ss}\tthread:%thread\tUSER:%X{USER}\tX-Track:%X{X-Track}\tlevel:%-5level\tlogger:%-48logger{48}\tmessage:%msg%n]]></pattern>
        </encoder>
    </appender>
    <!-- omitted -->

ログの出力例

.. code-block:: xml

    date:2013-09-06 23:05:22  thread:tomcat-http--3   USER:   X-Track:97988cc077f94f9d9d435f6f76027428    level:DEBUG logger:o.t.g.w.logging.HttpSessionEventLoggingListener  message:SESSIONID#D7AD1D42D3E77D61DB64E7C8C65CB488 sessionCreated : org.apache.catalina.session.StandardSessionFacade@e51960
    date:2013-09-06 23:05:22  thread:tomcat-http--3   USER:anonymousUser  X-Track:97988cc077f94f9d9d435f6f76027428    logger:o.t.gfw.web.logging.TraceLoggingInterceptor      message:[START CONTROLLER] HomeController.home(Locale,Model)
    date:2013-09-06 23:05:22  thread:tomcat-http--3   USER:anonymousUser  X-Track:97988cc077f94f9d9d435f6f76027428    level:INFO  logger:c.terasoluna.logging.app.welcome.HomeController  message:Welcome home! The client locale is ja.
    date:2013-09-06 23:05:22  thread:tomcat-http--3   USER:anonymousUser  X-Track:97988cc077f94f9d9d435f6f76027428    logger:o.t.gfw.web.logging.TraceLoggingInterceptor      message:[END CONTROLLER  ] HomeController.home(Locale,Model)-> view=home, model={serverTime=2013/09/06 23:05:22 JST}
    date:2013-09-06 23:05:22  thread:tomcat-http--3   USER:anonymousUser  X-Track:97988cc077f94f9d9d435f6f76027428    logger:o.t.gfw.web.logging.TraceLoggingInterceptor      message:[HANDLING TIME   ] HomeController.home(Locale,Model)-> 36,508,860 ns

\
 .. note::

     \ ``UserIdMDCPutFilter``\ がMDCにputするユーザー情報はSpring SecurityのFilterにより作成される。
     前述のように\ ``UserIdMDCPutFilter``\ をweb.xmlに定義した場合、ユーザーIDがログに出力されるのは
     Spring Securityの一連の処理が終わった後になる。ユーザー情報が生成された後、すぐにログに出力したい場合は、
     web.xmlの定義は削除して、以下のようにSpring SecurityのFilterに組み込む必要がある。


     spring-security.xmlには以下のような定義を追加する。

         .. code-block:: xml

             <sec:http>
                 <!-- omitted -->
                 <sec:custom-filter ref="userIdMDCPutFilter" after="ANONYMOUS_FILTER"/> <!-- (1) -->
                 <!-- omitted -->
             </sec:http>

             <!-- (2) -->
             <bean id="userIdMDCPutFilter" class="org.terasoluna.gfw.security.web.logging.UserIdMDCPutFilter">
             </bean>


         .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
         .. list-table::
             :header-rows: 1
             :widths: 10 90


             * - 項番
               - 説明
             * - | (1)
               - | Bean定義した\ ``UserIdMDCPutFilter`` \ を"ANONYMOUS_FILTER"の後に追加する。
             * - | (2)
               - | \ ``UserIdMDCPutFilter`` \ を定義する。

     blankプロジェクトでは\ ``UserIdMDCPutFilter``\ をspring-security.xmlに定義している。

共通ライブラリが提供するログ出力関連機能
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


.. _logging_appendix_httpsessioneventlogginglistener:

HttpSessionEventLoggingListener
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\  ``org.terasoluna.gfw.web.logging.HttpSessionEventLoggingListener``\ は、
セッションの生成・破棄・活性・非活性、セッションへの属性の追加・削除のタイミングでdebugログを出力するためのリスナークラスである。

web.xmlに、以下を追加すればよい。

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <web-app xmlns="http://java.sun.com/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
        version="3.0">
        <listener>
            <listener-class>org.terasoluna.gfw.web.logging.HttpSessionEventLoggingListener</listener-class>
        </listener>

        <!-- omitted -->
    </web-app>


logback.xmlには、以下のように\ ``org.terasoluna.gfw.web.logging.HttpSessionEventLoggingListener``\ を、debugレベルで設定する。

.. code-block:: xml

    <logger
        name="org.terasoluna.gfw.web.logging.HttpSessionEventLoggingListener"> <!-- (1) -->
        <level value="debug" />
    </logger>


以下のようなデバッグログが出力される。

.. code-block:: xml

    date:2013-09-06 16:41:33	thread:tomcat-http--3	USER:	X-Track:c004ddb56a3642d5bc5f6b5d884e5db2	level:DEBUG	logger:o.t.g.w.logging.HttpSessionEventLoggingListener 	message:SESSIONID#EDC3C240A7A1CCE87146A6BA1321AD0F sessionCreated : org.apache.catalina.session.StandardSessionFacade@f00e0f

\ ``@SessionAttributes``\ など、Sessionを使用してオブジェクトのライフサイクルを管理している場合、
本リスナーを利用して、セッションへ追加した属性が、想定通りに削除されているか確認することを、強く推奨する。

TraceLoggingInterceptor
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\  ``org.terasoluna.gfw.web.logging.TraceLoggingInterceptor``\ は、
Controllerの処理開始、終了をログ出力する\ ``HandlerInterceptor``\ である。
終了時にはControllerが返却したView名とModelに追加された属性、およびControllerの処理に要した時間も出力する。


spring-mvc.xmlの\ ``<mvc:interceptors>``\ 内に以下のように\ ``TraceLoggingInterceptor``\ を追加する。

.. code-block:: xml

    <mvc:interceptors>
        <!-- omitted -->
        <mvc:interceptor>
            <mvc:mapping path="/**" />
            <mvc:exclude-mapping path="/resources/**" />
            <bean
                class="org.terasoluna.gfw.web.logging.TraceLoggingInterceptor">
            </bean>
        </mvc:interceptor>
        <!-- omitted -->
    </mvc:interceptors>

| デフォルトでは、Controllerの処理に3秒以上かかった場合にWARNログを出力する。
| この閾値を変える場合は、\ ``warnHandlingNanos``\ プロパティにナノ秒単位で指定する。

閾値を10秒(10 * 1000 * 1000 * 1000 ナノ秒)に変更したい場合は以下のように設定すればよい。

.. code-block:: xml
    :emphasize-lines: 8

    <mvc:interceptors>
        <!-- omitted -->
        <mvc:interceptor>
            <mvc:mapping path="/**" />
            <mvc:exclude-mapping path="/resources/**" />
            <bean
                class="org.terasoluna.gfw.web.logging.TraceLoggingInterceptor">
                <property name="warnHandlingNanos" value="#{10 * 1000 * 1000 * 1000}" />
            </bean>
        </mvc:interceptor>
        <!-- omitted -->
    </mvc:interceptors>


logback.xmlには、以下のように、\ ``org.terasoluna.gfw.web.logging.TraceLoggingInterceptor``\ をtraceレベルで設定する。

.. code-block:: xml

    <logger name="org.terasoluna.gfw.web.logging.TraceLoggingInterceptor"> <!-- (1) -->
        <level value="trace" />
    </logger>

ExceptionLogger
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
例外発生時のロガーとして、\ ``org.terasoluna.gfw.common.exception.ExceptionLogger``\ が提供されている。

使用方法は、"\ :doc:`../WebApplicationDetail/ExceptionHandling`\ "の"\ :ref:`exception-handling-how-to-use-label`\ "を参照されたい。

.. raw:: latex

   \newpage

