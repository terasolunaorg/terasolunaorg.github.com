JMS(Java Message Service)
==============================

.. only:: html

 .. contents:: 目次
    :depth: 4
    :local:

.. _JMSOverview:

Overview
--------------------------------------------------------------------------------

本節では、JMS APIとSpring FrameworkのJMS連携用コンポーネントを使用したメッセージの送受信方法について説明する。


|

.. _JMSOverviewAboutJMS:

JMSとは
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| JMSはJavaでMOM（Message Oriented Middleware）を利用するための標準APIである。
| JMSのアーキテクチャは、JMSプロバイダを経由してクライアントからクライアントへメッセージを交換する。
| JMSは非同期メッセージングをサポートしているため、クライアント間を疎結合にすることができる。
| また、後述するPoint-to-Pointモデルを採用することでメッセージをQueueに格納できるため、クライアントの性能に応じたメッセージ受信が可能となる。
| その反面、クライアントからクライアントへのメッセージングにはタイムラグが発生しうるので、リアルタイムな応答が求められる処理に向かない傾向がある。
| JMSの詳細については、\ `Java Message Service (JMS) <http://www.oracle.com/technetwork/java/index-jsp-142945.html>`_\ を参照されたい。
|
| JMSを使用することで、同期または非同期でのメッセージングが可能となる。

.. note::

    本ガイドラインではJMS1.1を使用することを前提としている。

|

| 利用する際には、下記に説明する配信モデルとメッセージ送受信方式を要件に合わせて選択する必要がある。


* **配信モデル**

 | 配信モデルは、Point-to-Point（PTP）と Publisher-Subscriber（Pub/Sub）の2つのモデルが存在する。
 | 2つのモデルの大きな違いは送信者と受信者が1対1であるか、1対多であるかであり、用途によって選択する必要がある。

 (1) Point-To-Point（PTP）モデル


  .. figure:: ./images_JMS/JMSQueue.png
     :alt: JMS Queue
     :width: 70%

  | PTPモデルとは、2つのクライアント間において、一方のクライアント（Producer）からメッセージを送信し、もう一方のクライアント（Consumer）のみがそのメッセージを受信するモデルである。
  | PTPモデルにおけるメッセージのあて先（Destination）をQueueと呼ぶ。
  | ProducerはQueueにメッセージを送信し、ConsumerはQueueからメッセージを取得し、処理を行う。
  | Consumerからメッセージが取得されるか、メッセージが有効期限に達するとQueueからメッセージが削除される。
  |


 (2) Publisher-Subscriber（Pub/Sub）モデル

  .. figure:: ./images_JMS/JMSTopic.png
    :alt: JMS Topic
    :width: 70%

  | Pub/Subモデルとは、一方のクライアント（Publisher）からメッセージを発行(Publishes)し、他方の複数クライアント（Subscriber）にそのメッセージを配信(Delivers)するモデルである。
  | Pub/Subモデルにおけるメッセージのあて先（Destination)をTopicと呼ぶ。
  | SubscriberはTopicに対し購読依頼(Subscribes)を行い、PublisherはTopicにメッセージを発行する。
  | Topicに購読依頼している全てのSubscriberにメッセージが配信される。

 | **本ガイドラインでは、一般的に利用されることが多いPTPモデルの実装方法について説明する。**


* **メッセージ送信方式**

 | QueueまたはTopicへのメッセージ送信方式には、同期送信方式と非同期送信方式の2通りの処理方式が考えられるが、JMS1.1では同期送信方式のみがサポートされる。

 (1) 同期送信方式
 
  | 明示的にメッセージを送信する機能を呼び出すことで、メッセージに対する処理と送信が開始される。
  | JMSプロバイダからの応答があるまで待機するため、後続処理がブロックされる。
  |

 (2) 非同期送信方式
 
  | 明示的にメッセージを送信する機能を呼び出すことで、メッセージに対する処理と送信が開始される。
  | JMSプロバイダからの応答を待たないため、後続処理を続けて実行する。
  | 非同期送信方式の詳細については、\ `Java Message Service(Version 2.0) <http://download.oracle.com/otndocs/jcp/jms-2_0-fr-eval-spec/>`_\ の"7.3. Asynchronous send"を参照されたい。



* **メッセージ受信方式**

 | QueueまたはTopicに受信したメッセージに対する処理を実装する際には、同期受信方式と非同期受信方式の2通りの処理方式を選択することができる。
 | 後述するように、同期受信方式の利用ケースは限定的であるため、一般的には非同期受信方式が利用されることが多い。
 

 (1) 非同期受信方式
 
  | QueueまたはTopicがメッセージを受信すると、受信したメッセージに対する処理が開始される。
  | 1つのメッセージに対する処理が終了しなくても別のメッセージの処理が開始されるため、並列処理に向いている。
  |

 (2) 同期受信方式
 
  | 明示的にメッセージを受信する機能を呼び出すことで、受信とメッセージに対する処理が開始される。
  | メッセージを受信する機能は、QueueまたはTopicにメッセージが存在しない場合、受信するまで待機する。
  | そのため、タイムアウト値を設定することで、メッセージの待ち時間を指定する必要がある。
  
  | メッセージの同期受信を使用する一例として、WebアプリケーションにおいてQueueに溜まったメッセージを、画面操作時など任意のタイミングで取得・処理したい場合や、
    バッチで定期的にメッセージの処理を行いたい場合に使用することができる。
  | 


| JMSではメッセージは以下のパートで構成される。
| 詳細は\ `Java Message Service(Version 1.1) <http://download.oracle.com/otndocs/jcp/7195-jms-1.1-fr-spec-oth-JSpec/>`_\ の"3. JMS Message Model"を参照されたい。

 .. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
 .. list-table::
  :header-rows: 1
  :widths: 20 80
  
  * - 構成
    - 説明
  * - | ヘッダ
    - | JMSプロバイダやアプリケーションに対して、メッセージのDestinationや識別子などの制御情報やJMSの拡張ヘッダ(JMSX)、JMSプロバイダ独自のヘッダ、アプリケーション独自のヘッダを格納する。
  * - | プロパティ
    - | ヘッダに追加する制御情報を格納する。
  * - | ペイロード
    - | メッセージ本体を格納する。
      | データ種別によって、\ ``javax.jms.BytesMessage``\ 、\ ``javax.jms.MapMessage``\ 、\ ``javax.jms.ObjectMessage``\ 、\ ``javax.jms.StreamMessage``\ 、\ ``javax.jms.TextMessage``\ の5つのメッセージタイプを提供している。
      | JavaBeanを送信したい場合は、\ ``ObjectMessage``\ を利用する。
      | その場合は、JavaBeanをクライアント間で共有する必要がある。


.. _JMSOverviewAPI:

JMSの利用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| JMSを用いた処理を実装する場合、Java EEで定義されたJMS API（以下、JMS API）を使用することで、処理を実現できる。
| ただし、本ガイドラインでは、JMS APIをそのまま使用する場合に比べてメリット（記述が容易など）が多い、Spring FrameworkのJMS連携用コンポーネントを利用する前提としている。
| そのため、JMS APIの詳細については説明しない。
| 詳細については\ `Java API <https://docs.oracle.com/javaee/7/api/javax/jms/package-summary.html>`_\ を参照されたい。

 .. note::

   JMSはJava APIの標準化はしているが、メッセージの物理的なプロトコルの標準化はしていない。

 .. note::

   Java EEサーバではJMS実装が標準で組み込まれているためデフォルトで利用可能(Java EEサーバに組み込まれているJMSプロバイダを使う場合に限られる)だが、Apache TomcatなどのようにJMS実装が組み込まれていないJava EEサーバでは、別途JMS実装が必要になる。

|
|

.. _JMSOverviewSpringJMS:

Spring Frameworkのコンポーネントを使用したJMSの利用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| Spring Frameworkでは、メッセージ送受信を行うためのライブラリとして以下を提供している。

* \ ``spring-jms``\
    | JMSを利用したメッセージングを行うためのコンポーネントを提供する。
    | このライブラリに含まれるコンポーネントを利用することで、低レベルのJMS API呼び出しが不要となり、実装を簡素化できる。
    | \ ``spring-messaging``\ を利用することが可能である。

* \ ``spring-messaging``\
    | メッセージングベースのアプリケーションを作成する際に必要となる基盤機能を抽象化するためのコンポーネントを提供する。
    | メッセージとそれを処理するメソッドを対応付けるためのアノテーションのセットが含まれている。
    | このライブラリに含まれるコンポーネントを利用することで、メッセージングの実装スタイルを合わせることができる。


| \ ``spring-jms``\ のみでも実装可能であるが、\ ``spring-messaging``\ を利用することで実装方式を合わせることが可能である。
| 本ガイドラインでは、\ ``spring-messaging``\ も利用することを推奨している。

| ここでは、具体的な実装方法の説明を行う前に、Spring Frameworkが提供するJMS連携用のコンポーネントがどのようにメッセージを送受信しているかを説明する。
| まずは、説明に登場するコンポーネントを紹介する。
| Spring Frameworkは、以下にあげるインタフェースやクラスなどを利用してJMS API経由でメッセージ送受信を行う。

* \ ``javax.jms.ConnectionFactory``\
    | JMSプロバイダへのコネクション作成用インタフェース。
    | アプリケーションからJMSプロバイダへの接続を作成する機能を提供する。

* \ ``javax.jms.Destination``\
    | あて先(QueueやTopic)であることを示すインタフェース。

* \ ``javax.jms.MessageProducer``\
    | メッセージの送信用インタフェース。

* \ ``javax.jms.MessageConsumer``\
    | メッセージの受信用インタフェース。

* \ ``javax.jms.Message``\ 
    | ヘッダとボディを保持するメッセージであることを示すインタフェース。
    | 送受信はこのインタフェースの実装クラスがやり取りされる。

* \ ``org.springframework.messaging.Message``\ 
    | さまざまなメッセージングで扱うメッセージを抽象化したインタフェース。
    | JMSでも利用可能である。
    | 前述したとおり、メッセージングの実装方式を合わせるため、基本的にはspring-messagingで提供されている\ ``org.springframework.messaging.Message``\ を使用する。
    | ただし、\ ``org.springframework.jms.core.JmsTemplate``\ を使用したほうがよい場合が存在するので、その場合には\ ``javax.jms.Message``\ を使用する。

* \ ``org.springframework.jms.core.JmsMessagingTemplate``\ および\ ``org.springframework.jms.core.JmsTemplate``\
    | JMS APIを利用するためのリソースの生成や解放などをテンプレート化したクラス。
    | メッセージの送信及びメッセージの同期受信機能を行う際に使用することで実装を簡素化できる。
    | 基本的には、\ ``org.springframework.messaging.Message``\ を扱うことができる \ ``JmsMessagingTemplate``\ を使用する。
    | \ ``JmsMessagingTemplate``\ は\ ``JmsTemplate``\ をラップしているため、\ ``JmsTemplate``\ のプロパティを利用することで設定を行うことができる。
    | ただし、一部\ ``JmsTemplate``\ をそのまま使用したほうがよい場合が存在する。具体的な使用例については後ほど説明する。

* \ ``org.springframework.jms.listener.DefaultMessageListenerContainer``\
    | \ ``DefaultMessageListenerContainer``\ はQueueからメッセージを受け取り、受け取ったメッセージを処理する\ ``MessageListener``\ を起動させる。

* \ ``@org.springframework.jms.annotation.JmsListener``\
    | JMSの\ ``MessageListener``\ として扱うメソッドであることを示すマーカアノテーション。
    | メッセージを受け取った際に処理を行うメソッドに対して\ ``@JmsListener``\ アノテーションを付与する。


.. _JMSOverviewSyncSend:

メッセージを同期送信する場合
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| メッセージを同期送信する処理の流れについて図を用いて説明する。

 .. figure:: ./images_JMS/JMSSendOverview.png
    :alt: Send of Spring JMS
    :width: 70%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | Service内で、\ ``JmsMessagingTemplate``\ に対して「送信対象のDestination名」と「送信するメッセージのペイロード」を渡して処理を実行する。
        | \ ``JmsMessagingTemplate``\ は\ ``JmsTemplate``\ に処理を委譲する。
    * - | (2)
      - | \ ``JmsTemplate``\ はJNDI経由で取得された\ ``ConnectionFactory``\ から\ ``javax.jms.Connection``\ を取得する。
    * - | (3)
      - | \ ``JmsTemplate``\ は ``MessageProducer``\ に\ ``Destination``\ とメッセージを渡す。
        | \ ``MessageProducer``\ は\ ``javax.jms.Session``\ から生成される。(\ ``Session``\ は(2)で取得した\ ``Connection``\ から生成される。)
        | また、\ ``Destination``\ は(1)で渡された「送信対象のDestination名」をもとにJNDI経由で取得される。
    * - | (4)
      - | \ ``MessageProducer``\ は送信対象の\ ``Destination``\ へメッセージを送信する。


.. _JMSOverviewAsyncReceive:

メッセージを非同期受信する場合
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| メッセージを非同期受信する処理の流れについて図を用いて説明する。

 .. figure:: ./images_JMS/JMSASyncOverview.png
    :alt: ASync of Spring JMS
    :width: 70%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | JNDI経由で取得された\ ``ConnectionFactory``\ から\ ``Connection``\ を取得する。
    * - | (2)
      - | \ ``DefaultMessageListenerContainer``\ は\ ``MessageConsumer``\ に\ ``Destination``\ を渡す。
        | \ ``MessageConsumer``\ は\ ``Session``\ から生成される。(\ ``Session``\ は(1)で取得した\ ``Connection``\ から生成される。)
        | また、\ ``Destination``\ は\ ``@JmsListener``\ アノテーションで指定された「受信対象のDestination名」をもとにJNDI経由で取得される。
    * - | (3)
      - | \ ``MessageConsumer``\ は\ ``Destination``\ からメッセージを受信する。
    * - | (4)
      - | 受信したメッセージを引数として、\ ``MessageListener``\ 内の\ ``@JmsListener``\ アノテーションが設定されたメソッド(リスナーメソッド)が呼び出される。リスナーメソッドは\ ``DefaultMessageListenerContainer``\ で管理される。
        | 

.. _JMSOverviewSyncReceive:

メッセージを同期受信する場合
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| メッセージを同期受信する処理の流れについて図を用いて説明する。

 .. figure:: ./images_JMS/JMSSyncOverview.png
    :alt: Sync of Spring JMS
    :width: 70%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | Service内で、\ ``JmsMessagingTemplate``\ に対して、「受信対象のDestination名」を渡す。
        | \ ``JmsMessagingTemplate``\ は\ ``JmsTemplate``\ に処理を委譲する。
    * - | (2)
      - | \ ``JmsTemplate``\ はJNDI経由で取得された\ ``ConnectionFactory``\ から\ ``Connection``\ を取得する。
    * - | (3)
      - | \ ``JmsTemplate``\ は\ ``MessageConsumer``\ に\ ``Destination``\ とメッセージを渡す。
        | \ ``MessageConsumer``\ は\ ``Session``\ から生成される。(\ ``Session``\ は(2)で取得した\ ``Connection``\ から生成される。)
        | また、\ ``Destination``\ は(1)で渡された「受信対象のDestination名」をもとにJNDI経由で取得される。
    * - | (4)
      - | \ ``MessageConsumer``\ は\ ``Destination``\ からメッセージを受信する。
        | メッセージは\ ``JmsTemplate``\ や\ ``JmsMessagingTemplate``\ を経由してServiceに返却される。


.. _JMSOverviewAboutProjectConfiguration:

プロジェクト構成について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| JMSを利用する場合のプロジェクトの推奨構成について説明する。
| シリアライズしたJavaBeanを\ ``ObjectMessage``\ 経由で送受信する場合、このJavaBeanを送信側と受信側で共有する必要がある。
| この場合、既存のブランクプロジェクトとは別にmodelプロジェクトを追加することを推奨する。


* **modelの共有**

 * 送信または受信側のクライアントがmodelを提供していない場合

   modelプロジェクトを追加して、通信先のクライアントにJarファイルを配布する。

 * 送信または受信側のクライアントがmodelを提供している場合

   提供されたmodelをライブラリに追加する。

 | modelプロジェクト、または、配布されたアーカイブファイルと既存のプロジェクトとの関係は以下のようになる。

  .. figure:: ./images_JMS/ProjectStructure.png
      :alt: Projects
      :width: 70%
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
  .. list-table::
      :header-rows: 1
      :widths: 10 30 60

      * - 項番
        - プロジェクト名
        - 説明
      * - | (1)
        - | webプロジェクト
        - | 非同期受信を行うためのリスナークラスを配置する。
      * - | (2)
        - | domainプロジェクト
        - | 非同期受信を行うためのリスナークラスから実行されるServiceを配置する。
          | その他、Repositoryなどは従来と同じである。
      * - | (3)
        - | modelプロジェクトもしくはJarファイル
        - | ドメイン層に属するクラスのうち、クライアント間で共有するクラスを使用する。

|


 | modelプロジェクトを追加するためには、以下を実施する。
 
  * modelプロジェクトの作成
  * domainプロジェクトからmodelプロジェクトへの依存関係の追加
 
 | 詳細な追加方法については、同じようにJavaBeanの共有を行っている
   \ :doc:`../WebServiceDetail/SOAP`\ の\ :ref:`SOAPAppendixAddProject` \ を参照されたい。

.. _JMSHowToUse:

How to use
--------------------------------------------------------------------------------

.. _JMSHowToUseEnviromentSetting:

メッセージの送受信に共通する設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
本節では、メッセージの送受信に必要となる共通的な設定について説明する。

.. _JMSHowToUseDependentLibrary:

依存ライブラリの設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Spring FrameworkのJMS連携用コンポーネントを利用するために、domainプロジェクトのpom.xmlにSpring Frameworkの\ ``spring-jms``\ を追加する。

- :file:`[projectName]-domain/pom.xml`

 .. code-block:: xml

    <dependencies>

         <!-- (1) -->
         <dependency>
             <groupId>org.springframework</groupId>
             <artifactId>spring-jms</artifactId>
         </dependency>

     </dependencies>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | \ ``spring-jms``\ をdependenciesに追加する。
         | \ ``spring-jms``\ は\ ``spring-messaging``\ に依存するため、\ ``spring-messaging``\ も推移的に依存ライブラリとして追加される。

 | \ ``spring-jms``\ の他に、pom.xmlにJMSプロバイダのライブラリを追加する。
 | pom.xmlへのライブラリの追加例については、:ref:`JMSAppendixSettingsDependsOnJMSProvider` を参照されたい。

 .. note::

   上記設定例は、依存ライブラリのバージョンを親プロジェクトである terasoluna-gfw-parent で管理する前提であるため、pom.xmlでのバージョンの指定は不要である。
   上記の依存ライブラリはterasoluna-gfw-parentが利用している\ `Spring IO Platform <http://platform.spring.io/platform/>`_\ で定義済みである。

|

.. _JMSHowToUseConnectionFactory:

\ ``ConnectionFactory``\ の設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| \ ``ConnectionFactory``\ の定義の方法には、アプリケーションサーバで定義する方法と、Bean定義ファイルで定義する方法がある。
| 特別な理由がない場合、Bean定義ファイルをJMSプロバイダ非依存とするため、アプリケーションサーバで定義する方法を選択する。
| 本節では、アプリケーションサーバで定義する方法についてのみ説明する。
| アプリケーションサーバで定義した\ ``ConnectionFactory``\ を使用するためには、Bean定義ファイルにJNDI経由で取得したJavaBeanを利用するための設定を行う必要がある。

- :file:`[projectName]-domain/src/main/resources/META-INF/spring/[projectName]-infra.xml`

 .. code-block:: xml

    <!-- (1) -->
    <jee:jndi-lookup id="connectionFactory" jndi-name="jms/ConnectionFactory"/>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``jndi-name``\ 属性に、アプリケーションサーバ提供の\ ``ConnectionFactory``\ のJNDI名を指定する。
        | \ ``resource-ref``\ 属性がデフォルトで\ ``true``\ のため、JNDI名にプレフィックス(java:comp/env/)がない場合は、自動的に付与される。


 .. note:: **Bean定義したConnectionFactoryを使用する場合**

    JNDIを利用しない場合、\ ``ConnectionFactory``\ の実装クラスをBean定義することでも\ ``ConnectionFactory``\ を利用することが可能である。
    この場合、\ ``ConnectionFactory``\ の実装クラスはJMSプロバイダ依存となる。詳細については、:ref:`JMSAppendixSettingsDependsOnJMSProvider` の"JNDIを使用しない場合の設定"を参照されたい。

.. _JMSHowToUseDestinationResolver:

\ ``DestinationResolver``\ の設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Destinationの名前解決には、JNDIによる解決とJMSプロバイダでの解決の二通りの方法がある。
| デフォルトではJMSプロバイダでの解決が行われるが、ポータビリティや管理の観点から、特別な理由がない場合はJNDIによる解決を推奨する。

| \ ``org.springframework.jms.support.destination.JndiDestinationResolver``\ を使用することで、JNDI名によりDestinationの名前解決を行うことができる。
| 以下に\ ``JndiDestinationResolver``\ の定義例を示す。

- :file:`[projectName]-domain/src/main/resources/META-INF/spring/[projectName]-infra.xml`

 .. code-block:: xml
   
    <!-- (1) -->
    <bean id="destinationResolver"
       class="org.springframework.jms.support.destination.JndiDestinationResolver">
       <property name="resourceRef" value="true" /> <!-- (2) -->
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``JndiDestinationResolver``\ をBean定義する。
    * - | (2)
      - | JNDI名にプレフィックス(java:comp/env/)がないときに、自動的に付与させる場合は\ ``true``\ を設定する。デフォルトは\ ``false``\ である。
        
        .. warning::

           \ ``<jee:jndi-lookup/>``\ の\ ``resource-ref``\ 属性とはデフォルト値が異なることに注意されたい。


.. _JMSHowToUseSyncSendMessage:

メッセージを同期送信する方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| PTPモデルにて、クライアント（Producer）からJMSプロバイダへメッセージを同期送信する方法を説明する。

.. _JMSHowToUseSettingForSyncSend:

基本的な同期送信
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| \ ``JmsMessagingTemplate``\ を利用して、JMSプロバイダへの同期送信処理を実現する。

| \ ``Todo``\ クラスのオブジェクトをメッセージ同期送信する場合の実装例を示す。
| 最初に\ ``JmsMessagingTemplate``\ の設定方法を示す。

- :file:`[projectName]-domain/src/main/resources/META-INF/spring/[projectName]-infra.xml`

 .. code-block:: xml

    <bean id="cachingConnectionFactory"
       class="org.springframework.jms.connection.CachingConnectionFactory"> <!-- (1) -->
       <property name="targetConnectionFactory" ref="connectionFactory" /> <!-- (2) -->
       <property name="sessionCacheSize" value="1" />  <!-- (3) -->
    </bean>

    <!-- (4) -->
    <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
       <property name="connectionFactory" ref="cachingConnectionFactory" />
       <property name="destinationResolver" ref="destinationResolver" />
    </bean>

    <!-- (5) -->
    <bean id="jmsMessagingTemplate" class="org.springframework.jms.core.JmsMessagingTemplate">
        <property name="jmsTemplate" ref="jmsTemplate"/>
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``Session``\ 、\ ``MessageProducer/Consumer``\ のキャッシュを行う\ ``org.springframework.jms.connection.CachingConnectionFactory``\ をBean定義する。
        | Bean定義もしくはJNDI名でルックアップしたJMSプロバイダ固有の\ ``ConnectionFactory``\ をそのまま使うのではなく、
          \ ``CachingConnectionFactory``\ にラップして使用することで、キャッシュ機能を使用することができる。
    * - | (2)
      - | Bean定義もしくはJNDI名でルックアップしたJMSプロバイダ固有の\ ``ConnectionFactory``\ を指定する。
    * - | (3)
      - | \ ``Session``\ のキャッシュ数を設定する。（デフォルト値は1）
        | この例では1を指定しているが、性能要件に応じて適宜キャッシュ数を変更すること。
        | このキャッシュ数を超えてセッションが必要になるとキャッシュを使用せず、新しいセッションの作成と破棄を繰り返すことになる。
        | すると処理効率が下がり、性能劣化の原因になるので注意すること。
    * - | (4)
      - | \ ``JmsTemplate``\ をBean定義する。
        | \ ``JmsTemplate``\ は低レベルのAPIハンドリング（JMS API呼出し）を代行する。
        | 設定可能な属性に関しては、下記の\ ``JmsTemplate``\ の属性一覧を参照されたい。
    * - | (5)
      - | \ ``JmsMessagingTemplate``\ をBean定義する。同期送信処理を代行する\ ``JmsTemplate``\ をインジェクションする。


| 同期送信に関連する\ ``JmsTemplate``\ の属性は以下が存在する。
| 必要に応じて設定を行う必要がある。

 .. tabularcolumns:: |p{0.05\linewidth}|p{0.20\linewidth}|p{0.50\linewidth}|p{0.15\linewidth}|p{0.10\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 5 20 50 15 10
    :class: longtable

    * - 項番
      - 設定項目
      - 内容
      - 必須
      - デフォルト値
    * - 1.
      - \ ``connectionFactory``\
      - | 使用する\ ``ConnectionFactory``\ を設定する。
      - ○
      - なし（必須であるため）
    * - 2.
      - \ ``pubSubDomain``\
      - | メッセージングモデルについて設定する。
        | PTP（Queue）モデルは\ ``false``\ 、Pub/Sub（Topic）は\ ``true``\ に設定する。
      - \-
      - \ ``false``\ 
    * - 3.
      - \ ``sessionTransacted``\
      - | セッションでのトランザクション管理をするかどうか設定する。
        | 本ガイドラインでは、後述するトランザクション管理を使用するため、デフォルトのままの\ ``false``\ を推奨する。
      - \-
      - \ ``false``\ 
    * - 4.
      - \ ``messageConverter``\
      - | メッセージコンバータを設定する。
        | 本ガイドラインで紹介している範囲では、デフォルトのままで問題ない。
      - \-
      - \ ``SimpleMessageConverter``\ (\*1)が使用される。
    * - 5.
      - \ ``destinationResolver``\
      - | DestinationResolverを設定する。
        | 本ガイドラインでは、JNDIで名前解決を行う、\ ``JndiDestinationResolver``\ を設定することを推奨する。
      - \-
      - | \ ``DynamicDestinationResolver``\ (\*2)が使用される。
        | (\ ``DynamicDestinationResolver``\ を利用するとJMSプロバイダでDestinationの名前解決が行われる。)
    * - 6.
      - \ ``defaultDestination``\
      - | 既定のDestinationを設定する。
        | Destinationを明示的に指定しない場合、このDestinationが使用される。
      - \-
      - null(既定のDestinationなし)
    * - 7.
      - \ ``deliveryMode``\
      - | 配信モードを1(NON_PERSISTENT)、2(PERSISTENT)から設定する。
        | 2(PERSISTENT)は、メッセージの永続化を行う。
        | 1(NON_PERSISTENT)は、メッセージの永続化を行わない。
        | そのため、性能は上がるが、JMSプロバイダの再起動などが起こるとメッセージが消失する可能性がある。
        | 本ガイドラインでは、メッセージの消失を避けるため、 2(PERSISTENT)を使用することを推奨する。
        | この設定を使用する場合、後述する\ ``explicitQosEnabled``\ に\ ``true``\ を設定する必要があるので注意すること。
      - \-
      - 2(PERSISTENT)
    * - 8.
      - \ ``priority``\
      - | メッセージの優先度を設定する。優先度は0から9まで設定できる。
        | 数値が大きいほど優先度が高くなる。
        | 同期送信時にメッセージがQueueに格納される時点で優先度が評価され、優先度が高いメッセージは低いメッセージより先に取り出されるように格納される。
        | 優先度が同じメッセージはFIFO（First-In First-Out）で扱われる。
        | この設定を使用する場合、後述する\ ``explicitQosEnabled``\ に\ ``true``\ を設定する必要があるので注意すること。
      - \-
      - 4
    * - 9.
      - \ ``timeToLive``\
      - | メッセージの有効期限をミリ秒で設定する。
        | メッセージが有効期限に達すると、JMSプロバイダはQueueからメッセージを削除する。
        | この設定を使用する場合、後述する\ ``explicitQosEnabled``\ に\ ``true``\ を設定する必要があるので注意すること。
      - \-
      - 0（無制限）
    * - 10.
      - \ ``explicitQosEnabled``\
      - | \ ``deliveryMode``\ 、\ ``priority``\ 、\ ``timeToLive``\ を有効にする場合は\ ``true``\ を設定する。
      - \-
      - \ ``false``\
 .. raw:: latex

    \newpage

(\*1)\ ``org.springframework.jms.support.converter.SimpleMessageConverter``\ 

(\*2)\ ``org.springframework.jms.support.destination.DynamicDestinationResolver``\ 

|

| 次に送信対象のJavaBeanを作成する。

- :file:`[projectName]-domain/src/main/java/com/example/domain/model/Todo.java`

 .. code-block:: java

    package com.example.domain.model;
    
    import java.io.Serializable;
    
    public class Todo implements Serializable { // (1)
    
        private static final long serialVersionUID = -1L;
    
        // omitted

        private String description;

        // omitted

        private boolean finished;

        // omitted
    
        public String getDescription() {
            return description;
        }
    
        public void setDescription(String description) {
            this.description = description;
        }
    
        public boolean isFinished() {
            return finished;
        }
    
        public void setFinished(boolean finished) {
            this.finished = finished;
        }
    
    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 基本的には通常のJavaBeanで問題ないが、シリアライズして送信するため、\ ``java.io.Serializable``\ インタフェース を実装する必要がある。


| 最後に実際に同期送信を行う処理を記述する。
| 以下では、指定したテキストをもつ\ ``Todo``\ オブジェクトをQueueに同期送信する実装例を示す。

- :file:`[projectName]-domain/src/main/java/com/example/domain/service/todo/TodoServiceImpl.java`

 .. code-block:: java

    package com.example.domain.service.todo;

    import javax.inject.Inject; 
    import org.springframework.jms.core.JmsMessagingTemplate;
    import org.springframework.stereotype.Service; 
    import com.example.domain.model.Todo;
    
    @Service
    public class TodoServiceImpl implements TodoService {

        @Inject
        JmsMessagingTemplate jmsMessagingTemplate;    // (1)

        @Override
        public void sendMessage(String message) {
       
           Todo todo = new Todo();
           // omitted
           
           jmsMessagingTemplate.convertAndSend("jms/queue/TodoMessageQueue", todo);  // (2)
          
        }
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``JmsMessagingTemplate``\ をインジェクションする。
    * - | (2)
      - | \ ``JmsMessagingTemplate``\ の\ ``convertAndSend``\ メソッドを使用して、引数のJavaBeanを\ ``org.springframework.messaging.Message``\ インタフェースの実装クラスに変換し、指定したDestinationに対しメッセージを同期送信する。
        | デフォルトで変換には、\ ``org.springframework.jms.support.converter.SimpleMessageConverter``\ が使用される。
        | \ ``SimpleMessageConverter``\ を使用すると、\ ``javax.jms.Message``\ 、\ ``java.lang.String``\ 、\ ``byte配列``\ 、\ ``java.util.Map``\ 、\ ``java.io.Serializable``\ インタフェースを実装したクラスを送信可能である。
          
 .. note:: **業務ロジック内でJMSの例外ハンドリング**
    
    \ `JMS (Java Message Service)のIntroduction <http://docs.spring.io/spring/docs/4.3.5.RELEASE/javadoc-api/org/springframework/jms/core/JmsTemplate.html>`_\ で触れられているように、Spring Frameworkでは検査例外を非検査例外に変換している。
    そのため、業務ロジック内でJMSの例外をハンドリングする場合は、非検査例外を扱う必要がある。

     .. tabularcolumns:: |p{0.20\linewidth}|p{0.60\linewidth}|p{0.20\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 20 60 20

        * - Templateクラス
          - 例外の変換を行うメソッド
          - 変換後の例外
        * - | \ ``JmsMessagingTemplate``\ 
          - | \ ``JmsMessagingTemplate``\ の\ ``convertJmsException``\ メソッド
          - | \ ``MessagingException``\ (\*1)及びそのサブ例外
        * - | \ ``JmsTemplate``\ 
          - | \ ``JmsAccessor``\ の\ ``convertJmsAccessException``\ メソッド
          - | \ ``JmsException``\ (\*2)及びそのサブ例外

    (\*1) \ ``org.springframework.messaging.MessagingException``\ 

    (\*2) \ ``org.springframework.jms.JmsException``\ 

|

.. _JMSHowToUseSettingForSendWithHeader:

メッセージヘッダを編集して同期送信する場合
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``JmsMessagingTemplate``\ の\ ``convertAndSend``\ メソッドの引数にKey-Value形式のヘッダ属性と値を指定することで、ヘッダ属性を編集して同期送信することが可能である。
ヘッダの詳細については、\ `javax.jms.Messages  <https://docs.oracle.com/javaee/7/api/javax/jms/Message.html>`_\ を参照されたい。
送信、応答メッセージなどを紐付ける役割の\ ``JMSCorrelationID``\ を同期送信時に指定する場合の実装例を示す。


- :file:`[projectName]-domain/src/main/java/com/example/domain/service/todo/TodoServiceImpl.java`

 .. code-block:: java
 
  package com.example.domain.service.todo;
  
  import java.util.Map;
  import javax.inject.Inject;
  import org.springframework.jms.core.JmsMessagingTemplate;
  import org.springframework.stereotype.Service;
  import org.springframework.jms.support.JmsHeaders;
  import com.example.domain.model.Todo;
  
  @Service
  public class TodoServiceImpl implements TodoService {
  
  @Inject
  JmsMessagingTemplate jmsMessagingTemplate;
  
    public void sendMessageWithCorrelationId(String correlationId) {
    
      Todo todo = new Todo();
      // omitted
      
      Map<String, Object> headers = new HashMap<>();
      headers.put(JmsHeaders.CORRELATION_ID, correlationId);// (1)
      
      jmsMessagingTemplate.convertAndSend("jms/queue/TodoMessageQueue",
              todo, headers); // (2)
      
    }
  }
  
 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``Map``\ の実装クラスに対し、ヘッダ属性名とその値を設定してヘッダ情報を作成する。
    * - | (2)
      - | \ ``JmsMessagingTemplate``\ の\ ``convertAndSend``\ メソッドを使用することで、(2)で作成したヘッダ情報を付与したメッセージを同期送信する。

 .. warning:: **編集可能なヘッダ属性について**
 
   Spring Frameworkの\ ``SimpleMessageConverter``\ によるメッセージ変換時には、ヘッダ属性の一部(\ ``JMSDestination``\ 、\ ``JMSDeliveryMode``\ 、\ ``JMSExpiration``\ 、\ ``JMSMessageID``\ 、\ ``JMSPriority``\ 、\ ``JMSRedelivered``\ と\ ``JMSTimestamp``\ )をread-onlyとして扱っている。
   そのため、上記の実装例のようにread-onlyのヘッダ属性を設定しても、送信したメッセージのヘッダには格納されない。（メッセージのプロパティとして保持される。）
   read-onlyのヘッダ属性うち、\ ``JMSDeliveryMode``\ や\ ``JMSPriority``\ については、\ ``JmsTemplate``\ 単位での設定が可能である。
   詳細については、:ref:`JMSHowToUseSettingForSyncSend` の\ ``JmsTemplate``\ の属性一覧を参照されたい。



.. _JMSHowToUseSettingForSyncSendTransactionManagement:

トランザクション管理
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| データの一貫性を保証する必要がある場合は、トランザクション管理機能を使用する。
| 本ガイドラインで推奨する「宣言型トランザクション管理」を利用した実装例を以下に示す。
| 「宣言型トランザクション管理」の詳細は、\ :ref:`service_transaction_management` \ を参照されたい。
|
| トランザクション管理を実現するためには、\ ``org.springframework.jms.connection.JmsTransactionManager``\ を利用する。
| 最初に設定例を示す。

- :file:`[projectName]-domain/src/main/resources/META-INF/spring/[projectName]-domain.xml`

 .. code-block:: xml

    <!-- (1) -->
    <bean id="sendJmsTransactionManager"
       class="org.springframework.jms.connection.JmsTransactionManager">
       <!-- (2) -->  
       <property name="connectionFactory" ref="cachingConnectionFactory" />
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``JmsTransactionManager``\ をBean定義する。
      
        .. note:: **TransactionManagerのbean名について**

            \ ``@Transactional``\ アノテーションを付与した場合、デフォルトではBean名\ ``transactionManager``\ で登録されているBeanが使用される。
            (詳細は、\ :ref:`DomainLayerAppendixTransactionManagement` \ を参照されたい)
            
            Blankプロジェクトには、\ ``transactionManager``\ というBean名で\ ``DataSourceTransactionManager``\ が定義されているため、上記の設定では別名でBeanを定義している。
            
            そのため、アプリケーション内で、\ ``TransactionManager``\ を1つしか使用しない場合は、bean名を\ ``transactionManager``\ にすることで\ ``@Transactional``\ アノテーションでの\ ``transactionManager``\ 属性の指定を省略することができる。
       
       
    * - | (2)
      - | トランザクションを管理する\ ``CachingConnectionFactory``\ を指定する。

トランザクション管理を行い、\ ``Todo``\ オブジェクトをQueueに同期送信する実装例を以下に示す。

- :file:`[projectName]-domain/src/main/java/com/example/domain/service/todo/TodoServiceImpl.java`

 .. code-block:: java

    package com.example.domain.service.todo;

    import javax.inject.Inject; 
    import org.springframework.jms.core.JmsMessagingTemplate;
    import org.springframework.stereotype.Service; 
    import org.springframework.transaction.annotation.Transactional; 
    import com.example.domain.model.Todo;
    
    @Service
    @Transactional("sendJmsTransactionManager")  // (1)
    public class TodoServiceImpl implements TodoService {
       @Inject
       JmsMessagingTemplate jmsMessagingTemplate;

       @Override
       public void sendMessage(String message) {
          
          Todo todo = new Todo();
          // omitted
          
          jmsMessagingTemplate.convertAndSend("jms/queue/TodoMessageQueue", todo);  // (2)
       }

    }
    
 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@Transactional``\ アノテーションを利用してトランザクション境界を宣言する。
        | これにより、クラス内の各メソッドの開始時にトランザクションが開始され、メソッドの終了時にトランザクションがコミットされる。
    * - | (2)
      - | Queueにメッセージを同期送信する。
        | ただし、実際にメッセージがQueueに送信されるのはトランザクションがコミットされるタイミングとなるので注意すること。

|

DBのトランザクション管理を行う必要があるアプリケーションでは、業務の要件をもとにJMSとDBのトランザクションの関連を精査した上でトランザクションの管理方針を決定すること。

* **JMSとDBのトランザクションを分けてコミットやロールバックする場合**

  | JMSのトランザクションとDBのトランザクションを分ける場合は個別にトランザクション境界を宣言する。
  | 以下に、JMSのトランザクション管理に\ :ref:`JMSHowToUseSettingForSyncSendTransactionManagement`\ の\ ``sendJmsTransactionManager``\ を使用し、DBのトランザクション管理にBlankプロジェクトのデフォルトの設定で定義されている\ ``transactionManager``\ を使用する実装例を示す。
  
 - :file:`[projectName]-domain/src/main/java/com/example/domain/service/todo/TransactionalTodoServiceImpl.java`

   .. code-block:: java

      package com.example.domain.service.todo;

      import javax.inject.Inject; 
      import org.springframework.jms.core.JmsMessagingTemplate;
      import org.springframework.stereotype.Service; 
      import org.springframework.transaction.annotation.Transactional; 
      import com.example.domain.model.Todo;
      
      @Service
      @Transactional("sendJmsTransactionManager")  // (1)
      public class TransactionalTodoServiceImpl implements TransactionalTodoService {
         @Inject
         JmsMessagingTemplate jmsMessagingTemplate;

         @Inject
         TodoService todoService;

         @Override
         public void sendMessage(String message) {
             Todo todo = new Todo();
             // omitted
             
             jmsMessagingTemplate.convertAndSend("jms/queue/TodoMessageQueue", todo);

             // omitted
             todoService.update(todo);
         }

      }


 - :file:`src/main/java/com/example/domain/service/todo/TodoServiceImpl.java`

   .. code-block:: java

      import org.springframework.stereotype.Service;
      import org.springframework.transaction.annotation.Transactional;
      import com.example.domain.model.Todo;

      @Transactional // (2)
      @Service
      public class TodoServiceImpl implements TodoService {
          
          @Override
          public void update(Todo todo) {
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
        - | \ ``@Transactional``\ アノテーションを使用してJMSのトランザクション境界を宣言する。
          | JMSのトランザクション管理を行う\ ``sendJmsTransactionManager``\ を指定する。
      * - | (2)
        - | \ ``@Transactional``\ アノテーションを使用してDBのトランザクション境界を宣言する。
          | valueを省略しているため、デフォルトで、Bean名\ ``transactionManager``\ を参照する。
          | \ ``@Transactional``\ アノテーションの詳細については、\ :doc:`../../ImplementationAtEachLayer/DomainLayer`\ の\ :ref:`service_transaction_management`\ を参照されたい。

|

* **JMSとDBのトランザクションを一度にコミットやロールバックする場合**


  JMSとDBのトランザクションの連携にはJTAによるグローバルトランザクションを使用する方法があるが、プロトコルの特性上、性能面のオーバーヘッドがかかるため、"Best Effort 1 Phase Commit"の使用を推奨する。詳細は以下を参照されたい。

  | \ `Distributed transactions in Spring, with and without XA <http://www.javaworld.com/article/2077963/open-source-tools/distributed-transactions-in-spring--with-and-without-xa.html>`_\ 
  | \ `Spring Distributed transactions using Best Effort 1 Phase Commit <http://gharshangupta.blogspot.jp/2015/03/spring-distributed-transactions-using_2.html>`_\ 


  .. warning:: **メッセージ受信後にJMSプロバイダとの接続が切れるなどでJMSプロバイダにトランザクションの処理結果が返らない場合**
    
    メッセージ受信後にJMSプロバイダとの接続が切れるなどで、JMSプロバイダにトランザクションの処理結果が返らない場合、トランザクションの扱いはJMSプロバイダに依存する。
    そのため、\ **受信したメッセージの消失などを考慮した設計**\ を行うこと。
    特に、メッセージの消失が絶対に許されないような場合には、\ **メッセージの消失を補う仕組みを用意するか、グローバルトランザクションなどの利用を検討する**\ 必要がある。

  | "Best Effort 1 Phase Commit"は\ ``org.springframework.data.transaction.ChainedTransactionManager``\ を利用することで実現する。
  | 以下に、JMSのトランザクション管理に\ :ref:`JMSHowToUseSettingForSyncSendTransactionManagement`\ の\ ``sendJmsTransactionManager``\ を使用し、DBのトランザクション管理にBlankプロジェクトのデフォルトの設定で定義されている\ ``transactionManager``\ を使用する設定例を示す。

  - :file:`xxx-env.xml`

   .. code-block:: xml
     
      <!-- (1) -->
      <bean id="sendChainedTransactionManager" class="org.springframework.data.transaction.ChainedTransactionManager">
          <constructor-arg>
              <list>
                  <!-- (2) -->
                  <ref bean="sendJmsTransactionManager" />
                  <ref bean="transactionManager" />
              </list>
          </constructor-arg>
      </bean>

   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 90
     
      * - 項番
        - 説明
      * - | (1)
        - | \ ``ChainedTransactionManager``\ をBean定義する。
      * - | (2)
        - | JMSとDBのトランザクションマネージャを指定する。
          | 登録した順にトランザクションが開始され、登録した逆順にトランザクションがコミットされる。

  上記の設定を利用した実装例を以下に示す。


 - :file:`[projectName]-domain/src/main/java/com/example/domain/service/todo/ChainedTransactionalTodoServiceImpl.java`

   .. code-block:: java

      package com.example.domain.service.todo;

      import javax.inject.Inject; 
      import org.springframework.jms.core.JmsMessagingTemplate;
      import org.springframework.stereotype.Service; 
      import org.springframework.transaction.annotation.Transactional; 
      import com.example.domain.model.Todo;
      
      @Service
      @Transactional("sendChainedTransactionManager")  // (1)
      public class ChainedTransactionalTodoServiceImpl implements ChainedTransactionalTodoService {
         @Inject
         JmsMessagingTemplate jmsMessagingTemplate;

         @Inject
         TodoSharedService todoSharedService;

         @Override
         public void sendMessage(String message) {
             Todo todo = new Todo();
             // omitted
             
             jmsMessagingTemplate.convertAndSend("jms/queue/TodoMessageQueue", todo); // (2)

             // omitted
             todoSharedService.insert(todo); // (3)
         }

      }

   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - 項番
        - 説明
      * - | (1)
        - | \ ``@Transactional``\ アノテーションに\ ``sendChainedTransactionManager``\ を指定することで、JMSとDBのトランザクション管理を行う。
          | \ ``@Transactional``\ アノテーションの詳細については、\ :doc:`../../ImplementationAtEachLayer/DomainLayer`\ の\ :ref:`service_transaction_management`\ を参照されたい。
      * - | (2)
        - | メッセージの同期送信を行う。
      * - | (3)
        - | DBアクセスを伴う処理を実行する。この例では、DBの更新を伴うSharedServiceを実行している。


   .. note::

      業務上、JMSとDBなど複数のトランザクションをまとめて管理する必要がある場合、グローバルトランザクションを検討する。
      グローバルトランザクションについては、\ :ref:`service_enable_transaction_management`\ の"複数DB（複数リソース）に対するトランザクション管理（グローバルトランザクションの管理）が必要な場合"を参照されたい。

|


.. _JMSHowToUseAsyncReceiveMessage:

メッセージを非同期受信する方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| \ :ref:`JMSOverviewAboutJMS`\ の"メッセージ受信方式"で述べたように、一般的に受信処理を行う場合には非同期受信を利用する。
| 非同期受信機能を司る\ ``DefaultMessageListenerContainer``\ に対し、\ ``@JmsListener``\ アノテーションが付与されたリスナーメソッドを登録することで非同期受信処理を実現する。
| 非同期受信時の処理を行うリスナーメソッドの役割として、以下が存在する。

#. | **メッセージを受け取るためのメソッドを提供する。**
   | \ ``@JmsListener``\ アノテーションが付与されたメソッドを実装することで、メッセージを受け取ることができる。
#. | **業務処理の呼び出しを行う。**
   | リスナーメソッドでは業務処理の実装は行わず、Serviceのメソッドに処理を委譲する。
#. | **業務ロジックで発生した例外のハンドリングを行う。**
   | ビジネス例外や正常稼働時に発生するライブラリ例外のハンドリングを行う。
#. | **処理結果をメッセージ送信する。**
   | 応答メッセージなどの送信が必要なメソッドでは、\ ``org.springframework.jms.listener.adapter.JmsResponse``\ を利用することで、指定したDestinationに対してリスナーメソッドや業務ロジックの処理結果をメッセージ送信することができる。

.. _JMSHowToUseListenerContainer:

基本的な非同期受信
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| \ ``@JmsListener``\ アノテーションを利用した非同期受信の方法について説明をする。
| 非同期受信の実装には下記の設定が必要となる。

* JMS Namespaceを定義する。
* \ ``@JmsListener``\ アノテーションを有効化する。
* DIコンテナで管理しているコンポーネントのメソッドに\ ``@JmsListener``\ アノテーションを指定する。

| それぞれの詳細な実装方法について、以下に記述する。

- :file:`[projectName]-web/src/main/resources/META-INF/spring/applicationContext.xml`

 .. code-block:: xml

    <!-- (1) -->
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:jms="http://www.springframework.org/schema/jms"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/jms http://www.springframework.org/schema/jms/spring-jms.xsd">

        <!-- (2) -->
        <jms:annotation-driven />


        <!-- (3) -->
        <jms:listener-container
            factory-id="jmsListenerContainerFactory"
            destination-resolver="destinationResolver"
            concurrency="1"
            cache="consumer"
            transaction-manager="jmsAsyncReceiveTransactionManager"/>

 .. tabularcolumns:: |p{0.05\linewidth}|p{0.21\linewidth}|p{0.74\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 26 64
    :class: longtable

    * - 項番
      - 属性名
      - 内容
    * - | (1)
      - xmlns:jms
      - | JMS Namespaceを定義する。
        | 値として\ ``http://www.springframework.org/schema/jms``\ を指定する。
        | JMS Namespaceの詳細については、\ `JMS Namespace Support <http://docs.spring.io/autorepo/docs/spring-framework/4.3.5.RELEASE/spring-framework-reference/html/jms.html#jms-namespace>`_\ を参照されたい。
    * - 
      - xsi:schemaLocation
      - | スキーマのURLを指定する。
        | 値に\ ``http://www.springframework.org/schema/jms``\ と\ ``http://www.springframework.org/schema/jms/spring-jms.xsd``\ を追加する。
    * - | (2)
      - \-
      - | \ ``<jms:annotation-driven />``\ を利用して、\ ``@JmsListener``\ アノテーションや\ ``@SendTo``\ アノテーション等のJMS関連のアノテーション機能を有効化する。
    * - | (3)
      - \-
      - | \ ``<jms:listener-container/>``\ を利用して\ ``DefaultMessageListenerContainer``\ を生成するファクトリへパラメータを与えることで、\ ``DefaultMessageListenerContainer``\ の設定を行う。
        | \ ``<jms:listener-container/>``\ の属性には、利用したい\ ``ConnectionFactory``\ のBeanを指定できる\ ``connection-factory``\ 属性が存在する。\ ``connection-factory``\ 属性のデフォルト値は\ ``connectionFactory``\ である。
        | この例では、\ :ref:`JMSHowToUseConnectionFactory`\ で示した\ ``ConnectionFactory``\ のBean(Bean名は\ ``connectionFactory``\ )を利用するため、\ ``connection-factory``\ 属性を省略している。
        | \ ``<jms:listener-container/>``\ には、ここで紹介した以外の属性も存在する。
        | 詳細については、\ `Attributes of the JMS <listener-container> element <http://docs.spring.io/spring/docs/4.3.5.RELEASE/spring-framework-reference/html/jms.html#jms-namespace-listener-container-tbl>`_\ を参照されたい。

        .. warning::

            \ ``DefaultMessageListenerContainer``\ 内部には独自のキャッシュ機能が備わっているため、非同期受信の場合、\ ``CachingConnectionFactory``\ は使用してはいけない。
            詳細については、\ `DefaultMessageListenerContainerのJavadoc <http://docs.spring.io/autorepo/docs/spring-framework/4.3.5.RELEASE/javadoc-api/org/springframework/jms/listener/DefaultMessageListenerContainer.html>`_\ を参照されたい。
            上記のため、\ ``<jms:listener-container/>``\ の\ ``connection-factory``\ 属性には、\ :ref:`JMSHowToUseConnectionFactory`\ で定義した\ ``ConnectionFactory``\ を指定すること。

    * - 
      - \ ``concurrency``\
      - | \ ``DefaultMessageListenerContainer``\ が管理する各リスナーメソッドの並列数の上限を指定する。
        | \ ``concurrency``\ 属性のデフォルトは1である。
        | 並列数の下限と上限を指定することも可能である。例えば、下限を5、上限を10とする場合は"5-10"と指定する。
        | リスナーメソッドの並列数が設定した上限値に達した場合は、並列に処理されず待ち状態となる。
        | 必要に応じて値を設定すること。

        .. note::

          リスナーメソッド単位で並列数を指定したい場合は、\ ``@JmsListener``\ アノテーションの\ ``concurrency``\ 属性を利用することができる。

    * - 
      - \ ``destination-resolver``\
      - | 非同期受信時のDestination名解決で使用する\ ``DestinationResolver``\ のBean名を設定する。
        | \ ``DestinationResolver``\ のBean定義については、\ :ref:`JMSHowToUseDestinationResolver`\ を参照されたい。
        | \ ``destination-resolver``\ 属性を指定していない場合は\ ``DefaultMessageListenerContainer``\ 内で生成された\ ``DynamicDestinationResolver``\ が利用される。
    * - 
      - \ ``factory-id``\
      - | Bean定義を行う\ ``DefaultJmsListenerContainerFactory``\ の名前を設定する。
        | \ ``@JmsListener``\ アノテーションがデフォルトでBean名\ ``jmsListenerContainerFactory``\ を参照するため、\ ``<jms:listener-container/>``\ が一つの場合はBean名を\ ``jmsListenerContainerFactory``\ とすることを推奨する。
    * - 
      - \ ``cache``\
      - | \ ``Connection``\ 、\ ``Session``\ や\ ``Consumer``\ などのキャッシュ対象を決定するために、キャッシュレベルを指定する。
        | デフォルトは\ ``auto``\ である。
        | 後述する\ ``transaction-manager``\ 属性の設定時に、アプリケーションサーバ内でConnectionなどをプールしない場合は、性能向上のため、\ ``consumer``\ を指定することを推奨する。
        
        .. note::
        
           \ ``auto``\ の場合、\ ``transaction-manager``\ 属性の未設定時は、\ ``consumer``\ (\ ``Consumer``\ をキャッシュ)と同じ挙動となる。
           しかし、\ ``transaction-manager``\ 属性の設定時は、グローバルトランザクションなどによるアプリケーションサーバ内のプールを考慮して、\ ``none``\ (キャッシュが無効)と同じ挙動となる。

    * - 
      - \ ``transaction-manager``\
      - | 非同期受信時のトランザクション管理を行うBeanの名前を指定する。詳細については\ :ref:`JMSHowToUseTransactionManagementForAsyncReceive`\ を参照されたい。

 .. raw:: latex

    \newpage



 DIコンテナで管理しているコンポーネントのメソッドに\ ``@JmsListener``\ アノテーションを指定することで、指定したDestinationより非同期でメッセージを受信する。
 実装方法を以下に示す。


- :file:`[projectName]-web/src/main/java/com/example/listener/todo/TodoMessageListener.java`

 .. code-block:: java

    package com.example.listener.todo;

    import org.springframework.jms.annotation.JmsListener;
    import org.springframework.stereotype.Component;
    import com.example.domain.model.Todo; 
    @Component
    public class TodoMessageListener {
   
       @JmsListener(destination = "jms/queue/TodoMessageQueue")   // (1)
       public void receive(Todo todo) {
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
      - | 非同期受信用のメソッドに対し\ ``@JmsListener``\ アノテーションを設定する。\ ``destination``\ 属性には、受信先のDestination名を指定する。
      


 \ ``@JmsListener``\ アノテーションの主な属性の一覧を以下に示す。
 詳細やその他の属性については、\ `@JmsListenerアノテーションのJavadoc <http://docs.spring.io/spring-framework/docs/4.3.5.RELEASE/javadoc-api/org/springframework/jms/annotation/JmsListener.html#destination-->`_\ を参照されたい。


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 70

    * - 項番
      - 項目
      - 内容
    * - 1.
      - \ ``destination``\
      - | 受信するDestinationを指定する。
    * - 2.
      - \ ``containerFactory``\
      - | リスナーメソッドの管理を行う\ ``DefaultJmsListenerContainerFactory``\ のBean名を指定する。
        | デフォルトは\ ``jmsListenerContainerFactory``\ である。
    * - 3.
      - \ ``selector``\
      - | 受信するメッセージを限定するための条件であるメッセージセレクタを指定する。
        | 明示的に値を指定しない場合、デフォルトは""(空文字)であり、すべてのメッセージが受信対象となる。
        | 利用方法については、\ :ref:`JMSHowToUseMessageSelectorForAsyncReceive`\ を参照されたい。
    * - 3.
      - \ ``concurrency``\
      - | リスナーメソッドの並列数の上限を指定する。
        | \ ``concurrency``\ 属性のデフォルトは1である。
        | 並列数の下限と上限を指定することも可能である。例えば、下限を5、上限を10とする場合は"5-10"と指定する。
        | リスナーメソッドの並列数が設定した上限値に達した場合は、並列に処理されず待ち状態となる。
        | 必要に応じて値を設定すること。

.. _JMSHowToUseListenerContainerGetHeader:

メッセージのヘッダ情報を取得する
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 非同期受信の処理結果をProducer側で指定したDestination(ヘッダ属性\ ``JMSReplyTo``\ の値)に送信する場合など、メッセージのヘッダ情報をリスナーメソッド内で利用する場合には、\ ``@org.springframework.messaging.handler.annotation.Header``\ アノテーションを利用する。
| 実装例を以下に示す。

- :file:`[projectName]-web/src/main/java/com/example/listener/todo/TodoMessageListener.java`

 .. code-block:: java

    @JmsListener(destination = "jms/queue/TodoMessageQueue")
    public JmsResponse<Todo> receiveAndResponse(
            Todo todo, @Header("jms_replyTo") Destination storeResponseMessageQueue) { // (1)

        // omitted

         return JmsResponse.forDestination(todo, storeResponseMessageQueue);
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 受信メッセージのヘッダ属性\ ``JMSReplyTo``\ の値を取得するために、\ ``@Header``\ アノテーションを指定する。
        | JMSの標準ヘッダ属性を取得する場合に指定するキーの値については、\ `JmsHeadersの定数の定義 <https://static.javadoc.io/org.springframework/spring-jms/4.3.5.RELEASE/constant-values.html#org.springframework.jms.support.JmsHeaders.CORRELATION_ID>`_\ を参照されたい。


.. _JMSHowToUseListenerContainerReSendMessage:

非同期受信後の処理結果をメッセージ送信
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| \ ``@JmsListener``\ アノテーションを定義したメソッドの処理結果を、応答メッセージとしてDestinationに送信する方法が用意されている。
| 処理結果の送信先を指定する方法として、以下の2つが存在する。

* 処理結果の送信先を静的に指定する場合
* 処理結果の送信先を動的に指定する場合

それぞれについて、以下に説明する。

* **処理結果の送信先を静的に指定する場合**
     | \ ``@JmsListener``\ アノテーションが定義されているメソッドに対し、Destinationを指定した\ ``@SendTo``\ アノテーションを定義することで、固定のDestinationへの処理結果のメッセージ送信を実現する。
     | 実装例を以下に示す。

 - :file:`[projectName]-web/src/main/java/com/example/listener/todo/TodoMessageListener.java`

  .. code-block:: java

     @JmsListener(destination = "jms/queue/TodoMessageQueue")
     @SendTo("jms/queue/ResponseMessageQueue") // (1)
     public Todo receiveMessageAndSendTo(Todo todo) {

         // omitted
         return todo; // (2)
     }


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | \ ``@SendTo``\ アノテーションを定義することで、処理結果の送信先のデフォルトのDestinationを指定できる。
     * - | (2)
       - | \ ``@SendTo``\ アノテーションに定義したDestinationに送信するデータを返却する。
         | 許可されている返却値の型は\ ``org.springframework.messaging.Message``\ 、\ ``javax.jms.Message``\ 、\ ``String``\ 、\ ``byte``\ 配列、\ ``Map``\ 、\ ``Serializable``\ インタフェースを実装したクラス である。


* **処理結果の送信先を動的に変更する場合**

 | 動的に送信先のDestinationを変更する場合は\ ``JmsResponse``\ クラスの\ ``forDestination``\ や\ ``forQueue``\ メソッドを用いる。
 | 送信先のDestinationやDestination名を動的に変更することで、任意のDestinationに処理結果を送信することができる。実装例を以下に示す。
 
 - :file:`[projectName]-web/src/main/java/com/example/listener/todo/TodoMessageListener.java`

  .. code-block:: java

     @JmsListener(destination = "jms/queue/TodoMessageQueue")
     public JmsResponse<Todo> receiveMessageJmsResponse(Todo todo) {
   
         // omitted
   
         String resQueue = null;
   
         if (todo.isFinished()) {
             resQueue = "jms/queue/FinihedTodoMessageQueue";
         } else {
             resQueue = "jms/queue/ActiveTodoMessageQueue";
         }
   
         return JmsResponse.forQueue(todo, resQueue); // (1)
     }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | 処理内容に応じて送信先のQueueを変更する場合は\ ``JmsResponse``\ クラスの\ ``forDestination``\ や\ ``forQueue``\ メソッドを使用する。 
         | この例では、\ ``forQueue``\ メソッドを利用して、Destination名から送信を行っている。
         
         .. note::
         
            \ ``JmsResponse``\ クラスの\ ``forQueue``\ メソッドを利用する場合は、文字列であるDestination名を利用する。
            Destination名の解決には、\ ``DefaultMessageListenerContainer``\ に指定した\ ``DestinationResolver``\ が利用される。


.. note:: **処理結果の送信先をProducer側で指定する場合**

   以下のように実装することで、Producer側で指定した任意のDestinationに処理結果のメッセージを送信することができる。

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 実装箇所
         - 実装内容
       * - | Producer側
         - | JMS標準に則りメッセージのヘッダ属性\ ``JMSReplyTo``\ にDestinationを指定する。
           | ヘッダ属性の編集については、\ :ref:`JMSHowToUseSettingForSendWithHeader`\ を参照されたい。
       * - | Consumer側
         - | メッセージ送信するオブジェクトを返却する。

   ヘッダ属性\ ``JMSReplyTo``\ はConsumer側で指定したデフォルトのDestinationよりも優先される。
   詳細については、\ `Response management <http://docs.spring.io/spring/docs/4.3.5.RELEASE/spring-framework-reference/htmlsingle/#jms-annotated-response>`_\ を参照されたい。


.. _JMSHowToUseMessageSelectorForAsyncReceive:

非同期受信するメッセージを限定する場合
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

受信時にメッセージセレクタを指定することで受信するメッセージを限定することができる。


- :file:`[projectName]-web/src/main/java/com/example/listener/todo/TodoMessageListener.java`

 .. code-block:: java

    @JmsListener(destination = "jms/queue/MessageQueue" , selector = "TodoStatus = 'deleted'")    // (1)
    public void receive(Todo todo) {
        // omitted
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``selector``\ 属性を利用することで受信対象の条件を設定することができる。
        | ヘッダ属性の\ ``TodoStatus``\ が\ ``deleted``\ のメッセージのみ受信する。
        | メッセージセレクタはSQL92条件式構文のサブセットに基づいている。
        | 詳細は\ `Message Selectors <http://docs.oracle.com/javaee/7/api/javax/jms/Message.html>`_\ を参照されたい。


.. _JMSHowToUseValidationForAsyncReceive:

非同期受信したメッセージの入力チェック
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| セキュリティなどの観点から、不正なデータを保持したメッセージを業務ロジック内で処理しないよう、入力チェックを行うべきである。
| Method Validationを利用してServiceのメソッドで入力チェックを実装し、入力チェックエラー時の例外をリスナーメソッドでハンドリングする。
| これは、トランザクション管理を行う場合に、入力チェックエラー時の例外によって無用なロールバック処理が起こることを回避するためである。トランザクション管理については、\ :ref:`JMSHowToUseTransactionManagementForAsyncReceive`\ を参照されたい。
| Method Validationの設定や実装方法の詳細は、\ :doc:`../WebApplicationDetail/Validation`\ の\ :ref:`MethodValidation`\ を参照されたい。
| \ :ref:`JMSHowToUseSettingForSyncSend`\ で示した\ ``Todo``\ のオブジェクトに対して入力チェックを行う実装例を以下に示す。

- :file:`[projectName]-domain/src/main/java/com/example/domain/service/todo/TodoServiceImpl.java`

 .. code-block:: java

    package com.example.domain.service.todo;

    import javax.validation.Valid;
    import org.springframework.validation.annotation.Validated;
    import com.example.domain.model.Todo;
    
    @Validated // (1)
    public interface TodoService {

        void updateTodo(@Valid Todo todo); // (2)

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@Validated``\ アノテーションを付けることで、このインタフェースが入力チェック対象であることを宣言する。
    * - | (2)
      - | Bean Validationの制約アノテーションをメソッドの引数として指定する。


- :file:`[projectName]-domain/src/main/java/com/example/domain/model/Todo.java`

 .. code-block:: java

    package com.example.domain.model;
    
    import java.io.Serializable;
    import javax.validation.constraints.Null;
    
    // (1)
    public class Todo implements Serializable {
    
        private static final long serialVersionUID = -1L;
    
        // omitted

        @Null
        private String description;

        // omitted

        private boolean finished;

        // omitted
    
        public String getDescription() {
            return description;
        }
    
        public void setDescription(String description) {
            this.description = description;
        }
    
        public boolean isFinished() {
            return finished;
        }
    
        public void setFinished(boolean finished) {
            this.finished = finished;
        }
    
    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | Bean ValidationでJavaBeanの入力チェックを定義する。
        | この例では一例として\ ``@Null``\ アノテーションを設定している。
        | 詳細は「\ :doc:`../WebApplicationDetail/Validation`\ 」を参照されたい。


- :file:`[projectName]-web/src/main/java/com/example/listener/todo/TodoMessageListener.java`

 .. code-block:: java

    @Inject
    TodoService todoService;

    @JmsListener(destination = "jms/queue/MessageQueue")
    public void receive(Todo todo) {
        try {
            todoService.updateTodo(todo); // (1)
        } catch (ConstraintViolationException e) { // (2) 
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
      - | 入力チェックを行うServiceのメソッドを実行する。
    * - | (2)
      - | 制約違反時に発生する\ ``ConstraintViolationException``\ を捕捉する。
        | 捕捉後には任意の処理を実行可能である。
        | 論理的なエラーメッセージを格納するためのQueueを利用する場合など、別のQueueにメッセージ送信する例については、\ :ref:`JMSHowToUseExceptionHandlingForAsyncReceive`\ を参照されたい。

|

.. _JMSHowToUseTransactionManagementForAsyncReceive:

トランザクション管理
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| データの一貫性を保証する必要がある場合は、トランザクション管理機能を使用する。
| \ ``<jms:listener-container/>``\ に対し、トランザクションマネージャを設定することで、非同期受信時のトランザクション管理を実現できる。

    .. note:: 

       メッセージがQueueに戻されると、そのメッセージが再度非同期受信されるため、エラーの原因が解決していない場合は、ロールバック、非同期受信を繰り返すこととなる。
       JMSプロバイダによっては、ロールバック後の再送信回数に閾値を設定でき、再送信された回数が閾値を超えた場合、Dead Letter Queueにメッセージを格納する。


設定方法を以下に示す。

- :file:`[projectName]-domain/src/main/resources/META-INF/spring/[projectName]-domain.xml`

 .. code-block:: xml

    <!-- (1) -->
    <bean id="jmsAsyncReceiveTransactionManager"
       class="org.springframework.jms.connection.JmsTransactionManager">
       <!-- (2) -->  
       <property name="connectionFactory" ref="connectionFactory" />
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 非同期受信用の\ ``JmsTransactionManager``\ をBean定義する。
    * - | (2)
      - | トランザクションを管理する\ ``ConnectionFactory``\ を指定する。非同期受信の場合は、\ ``CachingConnectionFactory``\ を利用できないことに注意されたい。


- :file:`[projectName]-web/src/main/resources/META-INF/spring/applicationContext.xml`

 .. code-block:: xml

    <!-- (1) -->
    <jms:listener-container
        factory-id="jmsListenerContainerFactory"
        destination-resolver="destinationResolver"
        concurrency="1"
        error-handler="jmsErrorHandler"
        cache="consumer"
        transaction-manager="jmsAsyncReceiveTransactionManager"/>

 .. tabularcolumns:: |p{0.05\linewidth}|p{0.26\linewidth}|p{0.69\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 26 64

    * - 項番
      - 属性名
      - 内容
    * - | (1)
      - \ ``cache``\ 
      - | \ ``Connection``\ 、\ ``Session``\ や\ ``Consumer``\ などのキャッシュ対象を決定するために、キャッシュレベルを指定する。
        | デフォルトは\ ``auto``\ である。
        | \ :ref:`JMSHowToUseListenerContainer`\ で前述したように、アプリケーションサーバ内でConnectionなどをプールしない場合は\ ``consumer``\ を指定する。
    * - 
      - \ ``transaction-manager``\ 
      - | 利用する\ ``JmsTransactionManager``\ のBean名を指定する。
        | \ ``CachingConnectionFactory``\ を管理していない\ ``JmsTransactionManager``\ であることに注意されたい。
        
 .. warning:: 
 
    アプリケーションサーバによっては、アプリケーション内での\ ``Connection``\ や\ ``Session``\ のキャッシュを禁止している場合があるため、使用するアプリケーションサーバの仕様に応じてキャッシュの有効化、無効化を決定すること。

|

.. note:: **特定の例外の場合にロールバック以外の例外ハンドリングを行う方法** 

   トランザクション管理を有効にした場合、入力チェックなどで発生した例外を捕捉せずにthrowすると、ロールバックによってメッセージがQueueに戻される。
   リスナーメソッドはQueueに戻されたメッセージを再度非同期受信するため、非同期受信→エラー発生→ロールバックがJMSプロバイダの設定回数分繰り返されることになる。
   リトライによってエラーの原因が解消されないような例外の場合は、上記のような無駄な処理を抑えるため、例外を補足してリスナーメソッドからthrowしないようにハンドリングを行う。
   詳細については、\ :ref:`JMSHowToUseExceptionHandlingForAsyncReceive`\ を参照されたい。


DBのトランザクション管理を行う必要があるアプリケーションでは、業務の要件をもとにJMSとDBのトランザクションの関連を精査した上でトランザクションの管理方針を決定すること。


* **JMSとDBのトランザクションを分けてコミットやロールバックする場合**

  | JMSのトランザクションはロールバックするが、DBのトランザクションのみをコミットしたい場合が存在する。
  | その場合は、JMSとDBのトランザクションを別々に管理する必要がある。
  | リスナーメソッドから呼び出すServiceクラスに\ ``@Transactional``\ アノテーションを定義することで、JMSとDBのトランザクションを分けたトランザクション管理が可能となる。
  | JMSのトランザクション管理には\ :ref:`JMSHowToUseTransactionManagementForAsyncReceive`\ の\ ``jmsListenerContainerFactory``\ を使用し、DBのトランザクション管理にはBlankプロジェクトのデフォルトの設定で定義されている\ ``transactionManager``\ を使用する場合の実装例を以下に示す。
  
  - :file:`[projectName]-web/src/main/java/com/example/listener/todo/TodoMessageListener.java`
  
   .. code-block:: java
   
      package com.example.listener.todo;
      
      import javax.inject.Inject;
      import org.springframework.jms.annotation.JmsListener;
      import org.springframework.stereotype.Component;
      import com.example.domain.service.todo.TodoService;
      import com.example.domain.model.Todo; 
      @Component
      public class TodoMessageListener {
          @Inject
          TodoService todoService;

          @JmsListener(destination = "TransactedQueue") // (1)
          public void receiveTransactedMessage(Todo todo) {

              todoService.update(todo);

          }
      }

  - :file:`[projectName]-domain/src/main/java/com/example/domain/service/todo/TodoServiceImpl.java`
  
   .. code-block:: java
  
      package com.example.domain.service.todo;
  
      import org.springframework.stereotype.Service;
      import org.springframework.transaction.annotation.Transactional;
      import com.example.domain.model.Todo;
      
      @Transactional // (2)
      @Service
      public class TodoServiceImpl implements TodoService {
          
          @Override
          public void update(Todo todo) {
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
        - | \ ``@JmsListener``\ アノテーションを定義し、JMSのトランザクション管理を有効にした\ ``DefaultJmsListenerContainerFactory``\ を指定する。
          | \ ``@JmsListener``\ アノテーションはデフォルトでBean名\ ``jmsListenerContainerFactory``\ を参照するため、\ ``containerFactory``\ 属性を省略している。
      * - | (2)
        - | DBのトランザクション境界を定義する。
          | valueを省略しているため、デフォルトで、Bean名\ ``transactionManager``\ を参照する。
          | \ ``@Transactional``\ アノテーションの詳細については、\ :doc:`../../ImplementationAtEachLayer/DomainLayer`\ の\ :ref:`service_transaction_management`\ を参照されたい。

   .. note::

      トランザクション境界のネストの順序は業務要件によるが、JMSプロバイダは外部システムとの連携に使用される場合が多い。
      その場合はJMSトランザクション境界をDBトランザクション境界の外側に置き、内向きのDBトランザクションを先に完結する方がリカバリは容易である。
      DBのトランザクションをコミットし、JMSのトランザクションがロールバックした場合、メッセージがQueueに戻されるため、同じメッセージを再度処理することになる。
      設計上の考慮点として、業務処理の再実行時にDB更新処理を再試行しても問題ないように設計する必要がある。

|

* **JMSとDBのトランザクションを一度にコミットやロールバックする場合**

  JMSとDBのトランザクションの連携にはJTAによるグローバルトランザクションを使用する方法があるが、プロトコルの特性上、性能面のオーバーヘッドがかかるため、"Best Effort 1 Phase Commit"の使用を推奨する。詳細は以下を参照されたい。

  | \ `Distributed transactions in Spring, with and without XA <http://www.javaworld.com/article/2077963/open-source-tools/distributed-transactions-in-spring--with-and-without-xa.html>`_\ 
  | \ `Spring Distributed transactions using Best Effort 1 Phase Commit <http://gharshangupta.blogspot.jp/2015/03/spring-distributed-transactions-using_2.html>`_\ 

  .. warning:: **メッセージ受信後にJMSプロバイダとの接続が切れた場合などでJMSプロバイダにトランザクションの処理結果が返らない場合**
    
    メッセージ受信後にJMSプロバイダとの接続が切れた場合などで、JMSプロバイダにトランザクションの処理結果が返らない場合、トランザクションの扱いはJMSプロバイダに依存する。
    そのため、\ **受信したメッセージの消失や、ロールバックによるメッセージの再処理などを考慮した設計**\ を行うこと。
    特に、メッセージの消失が絶対に許されないような場合には、\ **メッセージの消失を補う仕組みを用意するか、グローバルトランザクションなどの利用を検討する**\ 必要がある。

  | "Best Effort 1 Phase Commit"は\ ``org.springframework.data.transaction.ChainedTransactionManager``\ を利用することで実現する。
  | 以下に、JMSのトランザクション管理に\ :ref:`JMSHowToUseTransactionManagementForAsyncReceive`\ の\ ``jmsAsyncReceiveTransactionManager``\ を使用し、DBのトランザクション管理にBlankプロジェクトのデフォルトの設定で定義されている\ ``transactionManager``\ を使用する設定例を示す。

  - :file:`[projectName]-env/src/main/resources/META-INF/spring/[projectName]-env.xml`

   .. code-block:: xml
     
      <!-- (1) -->
      <bean id="chainedTransactionManager" class="org.springframework.data.transaction.ChainedTransactionManager">
          <constructor-arg>
              <list>
                  <!-- (2) -->
                  <ref bean="jmsAsyncReceiveTransactionManager" />
                  <ref bean="transactionManager" />
              </list>
          </constructor-arg>
      </bean>

   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 90
     
      * - 項番
        - 説明
      * - | (1)
        - | \ ``ChainedTransactionManager``\ をBean定義する。
      * - | (2)
        - | JMSとDBのトランザクションマネージャを指定する。
          | 登録した順にトランザクションが開始され、登録した逆順にトランザクションがコミットされる。

  - :file:`[projectName]-web/src/main/resources/META-INF/spring/applicationContext.xml`

   .. code-block:: xml
     
      <!-- (1) -->
      <jms:listener-container
          factory-id="chainedTransactionJmsListenerContainerFactory"
          destination-resolver="destinationResolver"
          concurrency="1"
          error-handler="jmsErrorHandler"
          cache="consumer"
          transaction-manager="chainedTransactionManager"
          acknowledge="transacted"/>

   .. tabularcolumns:: |p{0.05\linewidth}|p{0.26\linewidth}|p{0.69\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 26 64

      * - 項番
        - 属性名
        - 内容
      * - | (1)
        - \-
        - | \ ``ChainedTransactionManager``\ を利用するための\ ``<jms:listener-container/>``\ を定義する。
      * - 
        - \ ``factory-id``\ 
        - | \ ``DefaultJmsListenerContainerFactory``\ のBean名を設定する。
          | この例では、\ :ref:`JMSHowToUseListenerContainer`\ の\ ``<jms:listener-container/>``\ と併用することを考慮し、Bean名を\ ``chainedTransactionJmsListenerContainerFactory``\ としている。
      * - 
        - \ ``transaction-manager``\ 
        - | \ ``ChainedTransactionManager``\ のBean名を指定する。
      * - 
        - \ ``acknowledge``\ 
        - | トランザクションを有効にするため、確認応答モードに\ ``transacted``\ を指定する。デフォルトは\ ``auto``\ である。
        
          .. note::
             
             \ ``DefaultMessageListenerContainer``\ 内では、\ ``transaction-manager``\ 属性に\ ``org.springframework.transaction.support.ResourceTransactionManager``\ の実装クラスのBeanが指定された場合、そのBeanを利用したトランザクション管理が有効化される。
             しかし、\ ``ChainedTransactionManager``\ は\ ``ResourceTransactionManager``\ を実装していないため、トランザクション管理が有効にならない。
             トランザクション管理を有効にするには、\ ``acknowledge``\ 属性に\ ``transacted``\ を指定する必要がある。
             これにより、トランザクション管理の対象となるSessionが生成され、\ ``ChainedTransactionManager``\ でのトランザクション管理が可能となる。

  上記の設定を利用した実装例を以下に示す。

  - :file:`[projectName]-web/src/main/java/com/example/listener/todo/TodoMessageListener.java`
  
   .. code-block:: java
     
     @Inject
     TodoService todoService;

     @JmsListener(containerFactory = "chainedTransactionJmsListenerContainerFactory", destination = "jms/queue/TodoMessageQueue") // (1)
     public void receiveTodo(Todo todo) {
         // omitted
     }

   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - 項番
        - 説明
      * - | (1)
        - | Bean名が\ ``chainedTransactionJmsListenerContainerFactory``\ の\ ``DefaultJmsListenerContainerFactory``\ を利用するため、\ ``containerFactory``\ 属性に\ ``chainedTransactionJmsListenerContainerFactory``\ を指定する。


  上記の設定、実装例に従ってアプリケーションを作成した場合の挙動について説明する。

  * **リスナーメソッドの処理が正常に終了した場合**

   | JMSとDBのトランザクションをともにコミットする。
   | JMS、DBの順にトランザクションを開始し、リスナーメソッドを実行した後に、DB、JMSの順にトランザクションを終了する。

    .. figure:: ./images_JMS/JMSDBTransactionAllCommit.png
        :alt: JMS/DB Transaction
        :width: 80%
    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | JMSのトランザクションを開始する。
       * - | (2)
         - | DBのトランザクションを開始する。
       * - | (3)
         - | リスナーメソッドが正常終了する。
       * - | (4)
         - | DBのトランザクションをコミットし、DBのトランザクションを終了する。
       * - | (5)
         - | JMSのトランザクションをコミットし、JMSのトランザクションを終了する。


  * **リスナーメソッドまたは業務ロジックで予期せぬ例外が発生した場合**

   リスナーメソッドで例外が発生した場合

    .. figure:: ./images_JMS/JMSDBTransactionAllRollback.png
        :alt: JMS/DB Transaction
        :width: 80%
    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | JMSのトランザクションを開始する。
       * - | (2)
         - | DBのトランザクションを開始する。
       * - | (3)
         - | リスナーメソッドまたは業務ロジックで予期しない例外が発生する。
       * - | (4)
         - | DBのトランザクションをロールバックし、DBのトランザクションを終了する。
       * - | (5)
         - | JMSのトランザクションをロールバックし、JMSのトランザクションを終了する。
           | JMSのトランザクションがロールバックするため、メッセージがQueueに戻される。

  * **メッセージ受信後にJMSプロバイダとの接続が切れた場合などで、DBのトランザクションのみコミットしてしまう場合**

   | JMSのトランザクションの扱いはJMSプロバイダに依存するが、DBのトランザクションはコミットするため、JMSとDBの状態に不整合が生じる可能性がある。
   | JMSのトランザクションがロールバックされることを考慮し、同じメッセージを複数受信した場合のデータの完全性を保障する必要がある。
   | データの完全性を保障する方法の例を以下に示す。

   * 非同期受信後の処理が複数回実行された場合に、処理後の状態が同じになるような処理として設計する。
   * \ ``JMSMessageID``\ を記録するよう設計する。メッセージの受信ごとに、過去に記録した\ ``JMSMessageID``\ と受信したメッセージの\ ``JMSMessageID``\ を比較し、一致した場合は受信したメッセージを破棄するよう設計する。

    .. figure:: ./images_JMS/JMSDBTransactionUnexpectedError.png
        :alt: JMS/DB Transaction
        :width: 80%
    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | JMSのトランザクションを開始する。
       * - | (2)
         - | DBのトランザクションを開始する。
       * - | (3)
         - | リスナーメソッドが正常終了する。
       * - | (4)
         - | DBのトランザクションをコミットし、DBのトランザクションを終了する。
       * - | (5)
         - | JMSプロバイダとの接続が切れるなどの予期せぬエラーが発生する。
       * - | (6)
         - | JMSのトランザクションのコミットに失敗している場合がある。
           | そのため、メッセージ消失などに備え、整合性を担保するための仕組みを用意する必要がある。

    .. note::

       上記のような事象を避け、JMSとDBなど複数のトランザクションを厳密に管理する必要がある場合には、グローバルトランザクションの利用を検討する。
       グローバルトランザクションについては、各種製品マニュアルを参照されたい。

|

.. _JMSHowToUseExceptionHandlingForAsyncReceive:

非同期受信時の例外ハンドリング
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| トランザクション管理を行う場合には、ロールバック処理を考慮した例外のハンドリングを行う必要がある。
| トランザクション管理の詳細については、\ :ref:`JMSHowToUseTransactionManagementForAsyncReceive`\ を参照されたい。
| JMSの例外ハンドリングは、目的に応じて以下の2種類のパターンに分類される。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.40\linewidth}|p{0.25\linewidth}|p{0.10\linewidth}|p{0.15\linewidth}|
 .. list-table:: **表-例外ハンドリングのパターン**
    :header-rows: 1
    :widths: 10 40 25 10 15

    * - 項番
      - ハンドリングの目的
      - ハンドリング対象となり得る例外の例
      - ハンドリング方法
      - ハンドリング単位
    * - | (1)
      - | ビジネス層で発生した例外を個別にハンドリングする場合
      - | 入力チェックエラーなどのビジネス例外
      - | リスナーメソッド
        | (try-catch)
      - | リスナーメソッド単位
    * - | (2)
      - | リスナーメソッドからthrowされた例外を統一的にハンドリングする場合
      - | 入出力エラーなどのシステム例外
      - | \ ``ErrorHandler``\ 
      - | JMSListenerContainer単位


* **ビジネス層で発生した例外を個別にハンドリングする場合**

  | メッセージの内容が不正である場合など、ビジネス層で発生した例外をリスナーメソッドで捕捉(try-catch)し、リスナーメソッド単位でハンドリングを行う。
  | トランザクション管理を行う場合、ロールバックが必要なケースは例外を\ ``DefaultMessageListenerContainer``\ にthrowする必要があるため、補足した例外をthrowし直すこと。
  | 実装例を以下に示す。
  
  - :file:`[projectName]-web/src/main/java/com/example/listener/todo/TodoMessageListener.java`
  
   .. code-block:: java
     
     @Inject
     TodoService todoService;

     @JmsListener(destination = "jms/queue/TodoMessageQueue")
     public JmsResponse<Todo> receiveTodo(Todo todo) {
         try {
             todoService.insertTodo(todo);
         } catch (BusinessException e) {
             return JmsResponse.forQueue(todo, "jms/queue/ErrorMessageQueue"); // (1)
         }
         return null; // (2)
     }

   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - 項番
        - 説明
      * - | (1)
        - | \ ``JmsResponse``\ クラスの\ ``forQueue``\ メソッドを利用し、任意のオブジェクトを論理的なエラーメッセージを格納するためのQueueに送信することができる。
          | この例では、AOPでログ出力が行われる\ ``BusinessException``\ を捕捉しているため、明示的にログ出力処理などを記述していないが、例外の原因を消失させないように例外をハンドリングする必要がある。
          | トランザクション管理を行い、ロールバックしてメッセージの再処理を行いたい場合には、捕捉した例外をthrowする必要がある。
      * - | (2)
        - | メッセージを送信しない場合は、返り値を\ ``null``\ にする。

* **リスナーメソッドからthrowされた例外を統一的にハンドリングする場合**

  | 例外ごとに共通的なハンドリングを行う場合には、\ ``<jms:listener-container/>``\ の\ ``error-handler``\ 属性に定義した\ ``ErrorHandler``\ の実装クラスを利用する。
  | 設定方法を以下に示す。

  - :file:`[projectName]-web/src/main/resources/META-INF/spring/applicationContext.xml`

   .. code-block:: xml

       <!-- (1) -->
       <jms:listener-container
           factory-id="jmsListenerContainerFactory"
           destination-resolver="destinationResolver"
           concurrency="1"
           error-handler="jmsErrorHandler"
           cache="consumer"
           transaction-manager="jmsAsyncReceiveTransactionManager"/>

       <!-- (2) -->
       <bean id="jmsErrorHandler"
           class="com.example.domain.service.todo.JmsErrorHandler">
       </bean>


   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - 項番
        - 説明
      * - | (1)
        - | \ ``<jms:listener-container/>``\ の\ ``error-handler``\ 属性にエラーハンドリングクラスのBean名を定義する。
      * - | (2)
        - | エラーハンドリングクラスをBean定義する。
       

  実装方法を以下に示す。

  - :file:`[projectName]-web/src/main/java/com/example/listener/todo/JmsErrorHandler.java`
  
   .. code-block:: java
  
      package com.example.listener.todo;
      
      import org.springframework.util.ErrorHandler;
      import org.terasoluna.gfw.common.exception.SystemException;
     
      public class JmsErrorHandler implements ErrorHandler {  // (1)
             
         @Override
          public void handleError(Throwable t) { // (2)
              // omitted
              if (t.getCause() instanceof SystemException) {  // (3)
               
                  // omitted system error handling
               
              } else {
                  // omitted error handling
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
        - | \ ``ErrorHandler``\ インタフェースを実装したエラーハンドリングクラスを作成する。
      * - | (2)
        - | リスナーメソッド内で発生した例外は\ ``org.springframework.jms.listener.adapter.ListenerExecutionFailedException``\ にラップされ、引数として渡される。
      * - | (3)
        - | 任意の例外クラスを判定し、例外に沿ったエラーハンドリングを実施する。
          | アプリケーション内で発生した例外を取得するには\ ``t.getCause()``\ を実行する必要がある。

|

.. _JMSHowToUseSyncReceiveMessage:

メッセージを同期受信する方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| \ ``JmsMessagingTemplate``\ を利用して、JMSプロバイダへの同期受信処理を実現する。
| 同期受信を利用することで、任意のタイミングでメッセージの受信が可能である。
| 同期受信を利用しない実現方法を十分に検討した上で、アーキテクチャを決定すること。

| 同期受信のBean定義ファイルの設定を以下に示す。

- :file:`[projectName]-domain/src/main/resources/META-INF/spring/[projectName]-infra.xml`

 .. code-block:: xml

    <bean id="cachingConnectionFactory"
       class="org.springframework.jms.connection.CachingConnectionFactory"> <!-- (1) -->
       <property name="targetConnectionFactory" ref="connectionFactory" /> <!-- (2) -->
       <property name="sessionCacheSize" value="1" />  <!-- (3) -->
    </bean>

    <!-- (4) -->
    <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
       <property name="connectionFactory" ref="cachingConnectionFactory" />
       <property name="destinationResolver" ref="destinationResolver" />
    </bean>

    <!-- (5) -->
    <bean id="jmsMessagingTemplate" class="org.springframework.jms.core.JmsMessagingTemplate">
        <property name="jmsTemplate" ref="jmsTemplate"/>
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``Session``\ 、\ ``MessageProducer/Consumer``\ のキャッシュを行う\ ``org.springframework.jms.connection.CachingConnectionFactory``\ をBean定義する。
        | Bean定義もしくはJNDI名でルックアップしたJMSプロバイダ固有の\ ``ConnectionFactory``\ をそのまま使うのではなく、
          \ ``CachingConnectionFactory``\ にラップして使用することで、キャッシュ機能を使用することができる。
    * - | (2)
      - | Bean定義もしくはJNDI名でルックアップした\ ``ConnectionFactory``\ を指定する。
    * - | (3)
      - | \ ``Session``\ のキャッシュ数を設定する。（デフォルト値は1）
        | この例では1を指定しているが、性能要件に応じて適宜キャッシュ数を変更すること。
        | このキャッシュ数を超えてセッションが必要になるとキャッシュを使用せず、新しいセッションの作成と破棄を繰り返すことになる。
        | すると処理効率が下がり、性能劣化の原因になるので注意すること。
    * - | (4)
      - | \ ``JmsTemplate``\ をBean定義する。
        | \ ``JmsTemplate``\ は低レベルのAPIハンドリング（JMS API呼出し）を代行する。
        | 設定可能な属性に関しては、下記の\ ``JmsTemplate``\ の属性一覧を参照されたい。
    * - | (5)
      - | \ ``JmsMessagingTemplate``\ をBean定義する。同期受信処理を代行する\ ``JmsTemplate``\ をインジェクションする。


| 同期受信に関連する\ ``JmsTemplate``\ の属性一覧を以下に示す。
| 必要に応じて設定を行う必要がある。

 .. tabularcolumns:: |p{0.05\linewidth}|p{0.20\linewidth}|p{0.50\linewidth}|p{0.15\linewidth}|p{0.10\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 5 20 50 15 10
    :class: longtable

    * - 項番
      - 設定項目
      - 内容
      - 必須
      - デフォルト値
    * - 1.
      - \ ``connectionFactory``\
      - | 使用する\ ``ConnectionFactory``\ を設定する。
      - ○
      - なし（必須であるため）
    * - 2.
      - \ ``pubSubDomain``\
      - | メッセージングモデルについて設定する。
        | PTP（Queue）モデルは\ ``false``\ 、Pub/Sub（Topic）は\ ``true``\ に設定する。
      - \-
      - \ ``false``\ 
    * - 3.
      - \ ``sessionTransacted``\
      - | セッションでのトランザクション管理をするかどうか設定する。
        | 本ガイドラインでは、後述するトランザクション管理を使用するため、デフォルトのままの\ ``false``\ を推奨する。
      - \-
      - \ ``false``\ 
    * - 4.
      - \ ``sessionAcknowledgeMode``\
      - | \ ``sessionAcknowledgeMode``\ はセッションの確認応答モードを設定する。
        | 詳細については\ `JmsTemplateのJavaDoc <http://docs.spring.io/spring/docs/4.3.5.RELEASE/javadoc-api/org/springframework/jms/core/JmsTemplate.html>`_\ を参照されたい。

        .. todo::

           sessionAcknowledgeModeの詳細については今後追記する。
           
      - \-
      - | 1 
    * - 5.
      - \ ``receiveTimeout``\
      - | 同期受信時のタイムアウト時間（ミリ秒）を設定する。未設定の場合、メッセージを受信するまで待機する。
        | 未設定の状態だと、後続の処理に影響が出てしまうため、必ず適切なタイムアウト時間を設定すること。
      - \-
      - | 0 
    * - 6.
      - \ ``messageConverter``\
      - | メッセージコンバータを設定する。
        | 本ガイドラインで紹介している範囲では、デフォルトのままで問題ない。
      - \-
      - \ ``SimpleMessageConverter``\ (\*1)が使用される。
    * - 7.
      - \ ``destinationResolver``\
      - | DestinationResolverを設定する。
        | 本ガイドラインでは、JNDIで名前解決を行う、\ ``JndiDestinationResolver``\ を設定することを推奨する。
      - \-
      - | \ ``DynamicDestinationResolver``\ (\*2)が使用される。
        | (\ ``DynamicDestinationResolver``\ を利用するとJMSプロバイダでDestinationの名前解決が行われる。)
    * - 8.
      - \ ``defaultDestination``\
      - | 既定のDestinationを設定する。
        | Destinationを明示的に指定しない場合、このDestinationが使用される。
      - \-
      - null(既定のDestinationなし)

 .. raw:: latex

    \newpage

(\*1)\ ``org.springframework.jms.support.converter.SimpleMessageConverter``\ 

(\*2)\ ``org.springframework.jms.support.destination.DynamicDestinationResolver``\ 


\ ``JmsMessagingTemplate``\ クラスの\ ``receiveAndConvert``\ メソッドにより、メッセージの同期受信を行う。実装例を以下に示す。

- :file:`[projectName]-domain/src/main/java/com/example/domain/service/todo/TodoServiceImpl.java`

 .. code-block:: java

    package com.example.domain.service.todo;
    
    import javax.inject.Inject; 
    import org.springframework.jms.core.JmsMessagingTemplate;
    import org.springframework.stereotype.Service; 
    import com.example.domain.model.Todo;
    
    @Service
    public class TodoServiceImpl implements TodoService {
        @Inject
        JmsMessagingTemplate jmsMessagingTemplate;
        
        @Override
        public String receiveTodo() {
       
           // omitted
           Todo retTodo = jmsMessagingTemplate.receiveAndConvert("jms/queue/TodoMessageQueue", Todo.class);   // (1)

        }
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``JmsMessagingTemplate``\ の\ ``receiveAndConvert``\ メソッドにより、指定したDestinationからメッセージを受信する。
        | \ ``receiveAndConvert``\ メソッドは、第2引数に変換先のクラスを指定することで型変換したクラスが取得できる。
        | ヘッダ項目を参照する場合は\ ``receive``\ メソッドを使用することにより、Spring Frameworkの\ ``Message``\ オブジェクトで取得することができる。

|

Appendix
--------------------------------------------------------------------------------

.. _JMSAppendixSettingsDependsOnJMSProvider:

JMSプロバイダに依存する設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

JMSプロバイダごとに設定が異なる場合がある。
以下にJMSプロバイダごとの設定についてを説明する。


Apache ActiveMQを利用する場合
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Apache ActiveMQを利用する場合の設定について説明する。

* **アプリケーションサーバに対するJMSプロバイダ固有の設定**

  | JMSプロバイダによっては、固有の設定が必要な場合がある。
  | Apache ActiveMQでは、受信するメッセージのペイロードが許可されたオブジェクトで構成されていることを保障するために、環境変数をアプリケーションサーバの起動引数に追加する必要がある。
  | 詳細については、\ `ObjectMessage <http://activemq.apache.org/objectmessage.html>`_\ を参照されたい。
  | 環境変数をApache Tomcatの起動引数に追加する例を以下に示す。JBoss Enterprise Application Platform 7.0の場合は\ `Configuring JBoss EAP to Run as a Service <https://access.redhat.com/documentation/en/red-hat-jboss-enterprise-application-platform/7.0/paged/installation-guide/chapter-4-configuring-jboss-eap-to-run-as-a-service>`_\ を、JBoss Enterprise Application Platform 6.4の場合は\ `Service Configuration <https://access.redhat.com/documentation/en-US/JBoss_Enterprise_Application_Platform/6.4/html/Installation_Guide/sect-Service_Configuration.html>`_\ を、Weblogicの場合は\ `Starting Managed Servers with a Startup Script <http://docs.oracle.com/middleware/1221/wls/START/overview.htm#START120>`_\ を参照されたい。

  - :file:`$CATALINA_HOME/bin/setenv.sh`

   .. code-block:: properties

      # omitted 
      # (1)
      -Dorg.apache.activemq.SERIALIZABLE_PACKAGES=java.lang,java.util,org.apache.activemq,org.fusesource.hawtbuf,com.thoughtworks.xstream.mapper,com.example.domain.model
      # omitted 

   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | 許可する任意のオブジェクトのパッケージを追加する。\ ``java.lang``\ , \ ``java.util``\ , \ ``org.apache.activemq``\ , \ ``org.fusesource.hawtbuf``\ , \ ``com.thoughtworks.xstream.mapper``\ はApache ActiveMQを使用する場合に必要な設定である。
           | このサンプルで必要な設定値として、"com.example.domain.model"を追加している。

* **ライブラリの追加**

  | \ ``spring-jms``\ ライブラリにはJMS APIが含まれない。
  | JMSプロバイダのライブラリにはJMS APIを含むことが多いが、JMSプロバイダのライブラリにJMS APIが含まれない場合は、pom.xmlにJMS APIを追加する。


  | domainプロジェクトとwebプロジェクトのpom.xmlに\ ``activemq-client``\ をビルド用のライブラリとして追加する。
  | また、アプリケーションサーバに\ ``activemq-client``\ とその依存ライブラリを追加する。

  - :file:`[projectName]-domain/pom.xml`
  - :file:`[projectName]-web/pom.xml`

   .. code-block:: xml

       <dependencies>

           <!-- (1) -->
           <dependency>
               <groupId>org.apache.activemq</groupId>
               <artifactId>activemq-client</artifactId>
               <scope>provided</scope>
           </dependency>

       </dependencies>

   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | Apache ActiveMQのクライアントライブラリをビルド用としてdependenciesに追加する。\ ``activemq-client``\ のライブラリにはJMS APIも含まれているため、JMS APIをライブラリとして追加する必要はない。

 .. note::

   上記設定例は、依存ライブラリのバージョンを親プロジェクトである terasoluna-gfw-parent で管理する前提であるため、pom.xmlでのバージョンの指定は不要である。
   上記の依存ライブラリはterasoluna-gfw-parentが利用している\ `Spring IO Platform <http://platform.spring.io/platform/>`_\ で定義済みである。

|

 .. warning::

   TERASOLUNA Server Framework for Javaで使用している\ `Spring IO Platform <http://platform.spring.io/platform/>`_\ では、Apache ActiveMQと接続などを行う際に使用するライブラリのバージョンを定義している。
   そのため、Apache ActiveMQのバージョンを決定する際には注意すること。
   また、TERASOLUNA Server Framework for Javaのバージョンアップの際には、ライブラリとミドルウェアのバージョンの整合性が取れなくなる可能性があるので注意すること。

|


* **アプリケーションサーバへのJNDI登録**

  | アプリケーションサーバへのJNDI登録については、\ `Manually integrating Tomcat and ActiveMQ <http://activemq.apache.org/tomcat.html>`_\ を参照されたい。


* **JNDIを使用しない場合の設定**

  | 本ガイドラインではJNDIによる名前解決する方法を推奨しているが、
  | アプリケーションサーバ上で動かせない単体テストの実施において、JMSプロバイダと接続する場合などには、JNDIを利用しないケースがある。
  | その場合、\ ``ConnectionFactory``\ の実装クラスのBeanを生成する必要がある。
  | また、QueueについてもJNDIを用いて指定していたが、JMSプロバイダーの機能を用いてDestinationに指定したQueueが存在しない場合に、指定した名前のQueueを動的に生成させることができる。
  | アプリケーションサーバを介さずに接続を行うにはApache ActiveMQの内部Brokerを用いる必要がある。
  | Apache ActiveMQの内部Brokerの設定については\ `How do I embed a Broker inside a Connection  <http://activemq.apache.org/how-do-i-embed-a-broker-inside-a-connection.html>`_\ を参照されたい。
  | テスト用のコンテキストに下記の設定を追加すること。

  - :file:`[projectName]-domain/src/main/resources/META-INF/spring/[projectName]-infra.xml`

   .. code-block:: xml

      <!-- (1) -->
      <bean id="connectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory"> 
          <constructor-arg value="tcp://localhost:61616"/>  <!-- (2) -->
      </bean>

   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - 項番
        - 説明
      * - | (1)
        - | Apache ActiveMQの\ ``ConnectionFactory``\ をBean定義する。
      * - | (2)
        - | Apache ActiveMQの起動URLを指定する。起動URLは各環境に沿った値を設定する。

 .. note::
 
   開発フェーズなどによって、ConnectionFactoryの設定方法をJNDIとBean定義で切り替えたい場合、
   \ ``[projectName]-env/src/main/resources/META-INF/spring/[projectName]-env.xml``\ に設定を記述すること。

.. _JMSAppendixSendManySameMessages:

同一メッセージの大量送信
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| 同一メッセージの大量送信の実装において\ ``JmsMessageTemplate``\ を使用する場合、メモリ使用量が増えてしまう可能性がある。
| そのため、\ ``JmsTemplate``\ クラスの\ ``send``\ メソッドを使用して実装を行うことを検討する必要がある。
| 理由としては、\ ``JmsMessageTemplate``\ ではメッセージ送信処理を行うたびに\ ``org.springframework.jms.core.MessageCreator``\ というクラスのインスタンスが生成されてしまう。
| 無駄なインスタンスの生成を防ぐために、送信処理時に\ ``MessageCreator``\ のインスタンスが生成されない\ ``JmsTemplate``\ クラスの\ ``send``\ メソッドで送信を行うことでメモリの使用量を削減するようにする。
| 以下に、ある文字列を同一Destinationに100件送信を行う場合コード例を示す。

- :file:`[projectName]-domain/src/main/java/com/example/domain/service/todo/TodoServiceImpl.java`

 .. code-block:: java

     package com.example.domain.service.todo;
     
     import java.io.IOException;
     import javax.inject.Inject;
     import javax.jms.JMSException;
     import javax.jms.Message;
     import javax.jms.Session;
     import javax.jms.TextMessage;
     import org.springframework.jms.core.JmsTemplate;
     import org.springframework.jms.core.MessageCreator;
     import org.springframework.stereotype.Service; 
     
     @Service
     public class TodoServiceImpl implements TodoService {

        @Inject
        JmsTemplate jmsTemplate; // (1)

        @Override
        public void sendManyMessage(final String messageStr) throws IOException {
            MessageCreator mc = new MessageCreator() { // (2)
                public Message createMessage(Session session) throws JMSException {
                    TextMessage message = session.createTextMessage(); // (3)
                    message.setText(messageStr);

                    // omitted
                    return message;
                }
            };
            for (int i = 0; i < 100; i++) {
                jmsTemplate.send("jms/queue/TodoMessageQueue", mc); // (4)
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
      - | \ ``JmsMessagingTemplate``\ を使用すると、送信のたびに\ ``MessageCreater``\ の生成が行われてしまうため、\ ``MessageCreater``\ の生成を送信と分離して定義できる\ ``JmsTemplate``\ を利用する。
    * - | (2)
      - | JMSの\ ``Message``\ を作成するために\ ``MessageCreator``\ のインスタンスを生成する。
    * - | (3)
      - | \ ``JmsTemplate``\ クラスの\ ``send``\ メソッドでメッセージを送信することで、ループごとに\ ``MessageCreator``\ のインスタンスを生成が行われなくなり、
          メモリの使用量を削減させることができるようになる。


.. _JMSAppendixSendLargeData:

サイズの大きなデータの送受信
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
画像データなどサイズの大きなデータ(目安として1 MB以上)を扱う場合、同時トランザクション数やヒープサイズによっては\ ``OutOfMemoryError``\ が発生する可能性がある。
JMSの標準APIではサイズの大きなデータをストリームとして扱うことができるのはプリミティブ型のデータの送信を行う\ ``StreamMessage``\ と未解釈のバイトストリームの送信を行える\ ``ByteMessage``\ のみである。
そのため、JMS APIではなく、JMSプロバイダベンダ毎に用意している固有のAPIを使用するケースがある。


Apache ActiveMQを利用する場合
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\ `Blob Message <http://activemq.apache.org/blob-messages.html>`_\ を使用することでサイズの大きなメッセージを送受信することができる。実装例を以下に示す。


 .. note::
    \ ``org.apache.activemq.BlobMessage``\ を使用する場合、Apache ActiveMQ独自のAPIを使用することになるため、
    Spring Frameworkが提供している\ ``Message``\ や\ ``CachingConnectionFactory``\ を使用することはできない。
    性能影響を考慮し、\ ``BlobMessage``\ を使用する場合は\ ``BlobMessage``\ 用の\ ``JmsTemplate``\ を別途定義することを推奨する。


* **設定**

  \ ``BlobMessage``\ を用いたメッセージの送信では、メッセージはヒープ領域ではなく、一時的にApache ActiveMQが起動しているサーバに格納される。
  メッセージの格納先の定義例を以下に示す。

  - :file:`[projectName]-domain/src/main/resources/META-INF/spring/[projectName]-infra.xml`

   .. code-block:: xml

      <bean id="connectionFactory"
         class="org.apache.activemq.ActiveMQConnectionFactory">
          <property name="brokerURL">
            <!-- (1) -->
            <value>tcp://localhost:61616?jms.blobTransferPolicy.uploadUrl=/tmp</value>
          </property>
      </bean>

   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - 項番
        - 説明
      * - | (1)
        - | 一時的にメッセージを格納するApache ActiveMQのサーバのディレクトリを定義する。
          | \ ``jms.blobTransferPolicy.uploadUrl``\ にはデフォルトで\ ``http://localhost:8080/uploads/``\ が設定されており、デフォルトか\ ``brokerURL``\ をオーバーロードすることで一時ファイルの置き場を指定できる。
          | 例では\ ``/tmp``\ に一時的にファイルを格納している。 


* **送信**

  \ ``Blob Message``\ を利用した送信クラスの実装例を以下に示す。


  - :file:`[projectName]-domain/src/main/java/com/example/domain/service/todo/TodoServiceImpl.java`

   .. code-block:: java

      package com.example.domain.service.todo;
      
      import java.io.IOException;
      import java.io.InputStream;
      import java.nio.file.Files;
      import java.nio.file.Path;
      import java.nio.file.Paths;
      import javax.inject.Inject;
      import javax.jms.JMSException;
      import javax.jms.Message;
      import javax.jms.Session;
      import org.apache.activemq.ActiveMQSession;
      import org.apache.activemq.BlobMessage;
      import org.springframework.jms.core.JmsTemplate;
      import org.springframework.jms.core.MessageCreator;
      import org.springframework.stereotype.Service;
      
      @Service
      public class TodoServiceImpl implements TodoService {
          @Inject
          JmsTemplate jmsTemplate;
          
          @Override
          public void sendBlobMessage(String inputFilePath) throws IOException {

              Path path = Paths.get(inputFilePath);
              try (final InputStream inputStream = Files.newInputStream(path)) {

                  jmsTemplate.send("jms/queue/TodoMessageQueue", new MessageCreator() {
                      public Message createMessage(Session session) throws JMSException {

                          ActiveMQSession activeMQSession = (ActiveMQSession) session;  // (1)

                          BlobMessage blobMessage = activeMQSession.createBlobMessage(inputStream);  // (2)
                          return blobMessage;
                      }
                  });
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
        - | \ ``BlobMessage``\ を使用するにはApache ActiveMQ独自APIである\ ``org.apache.activemq.ActiveMQSession``\ を使用する。
      * - | (2)
        - | \ ``ActiveMQSession``\ より、送信データを指定して\ ``BlobMessage``\ を生成する。 
          | \ ``createBlobMessage``\ メソッドの引数は\ ``File``\ 、\ ``InputStream``\ 、\ ``URL``\ クラスが指定可能である。


* **受信**

  受信クラスの実装例を以下に示す。


  - :file:`[projectName]-web/src/main/java/com/example/listener/todo/TodoMessageListener.java`
  
   .. code-block:: java
   
      package com.example.listener.todo;
      
      import java.io.IOException;
      import javax.inject.Inject;
      import javax.jms.JMSException;
      import org.apache.activemq.BlobMessage;
      import org.springframework.jms.annotation.JmsListener;
      import org.springframework.stereotype.Component;
      import com.example.domain.service.todo.TodoService;
      @Component
      public class TodoMessageListener {
          @Inject
          TodoService todoService;
          @JmsListener(destination = "jms/queue/TodoMessageQueue")
          public void receiveBlobMessage(BlobMessage message) throws IOException, JMSException {
           todoService.fileInputBlobMessage(message);
              // omitted
          }
      }

  - :file:`[projectName]-domain/src/main/java/com/example/domain/service/todo/TodoServiceImpl.java`

   .. code-block:: java

      package com.example.domain.service.todo;
      
      import java.io.IOException;
      import java.io.InputStream;
      import java.nio.file.Files;
      import java.nio.file.Path;
      import java.nio.file.Paths;
      import org.apache.activemq.BlobMessage;
      import org.springframework.stereotype.Service;
      
      @Service
      public class TodoServiceImpl implements TodoService {
      
          @Override
          public void fileInputBlobMessage(BlobMessage message) throws IOException {
              try(InputStream is =  message.getInputStream()){   // (1)
                  Path path = Paths.get("outputFilePath");
                  Files.copy(is, path);
                  // omitted
              }
          }
      }

   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 13 87

      * - 項番
        - 説明
      * - | (1)
        - | 受信した\ ``BlobMessage``\ より、\ ``InputStream``\ としてデータを取得する。


.. raw:: latex

   \newpage



