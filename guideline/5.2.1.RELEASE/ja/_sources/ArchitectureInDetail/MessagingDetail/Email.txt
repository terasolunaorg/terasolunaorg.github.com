E-mail送信(SMTP)
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

Overview
--------------------------------------------------------------------------------

本節では、SMTPによるE-mailの送信方法について説明する。

本ガイドラインでは、JavaMailのAPIとSpring Frameworkから提供されているMail連携用コンポーネントを利用することを前提としている。

.. note::

    説明の対象としているのはメールを送信する部分のみである。
    メール送信に係る処理方式については言及していない。
    （\ :ref:`email-processing-method`\ において一例を紹介している。）

|

JavaMailについて
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ `JavaMail <https://java.net/projects/javamail/pages/Home>`_\ は、Javaでメールの送受信を行うためのAPIを提供している。
Java EEに含まれるものであるが、Java SEでも追加パッケージとして利用可能である。
JavaMailを利用することで、メール機能を容易にJavaアプリケーションに組み込むことができる。

なお、本ガイドラインでは、Spring FrameworkのMail連携用コンポーネントを利用する前提であるため、JavaMailのAPIについての詳細には触れていない。
JavaMailのAPI仕様については、\ `JavaMail API Design Specification <http://download.oracle.com/otn-pub/jcp/java_mail-1_5-mrel2-eval-spec/JavaMail-1.5.pdf>`_\ を参照されたい。

.. note:: **メールセッション**

   メールセッション（\ `Session <http://docs.oracle.com/javaee/7/api/javax/mail/Session.html>`_\ ）は、メールサーバに接続する際に必要となる情報を管理する。
   
   メールセッションを取得するには以下のような方法がある。
   
   * 典型的なエンタープライズアプリケーションにおいては、Java EEのコンテナで管理されたメールセッションをJNDI経由で取得する。
   * Tomcatの場合はリソースファクトリで定義したメールセッションをJNDI経由で取得する。
   * staticファクトリメソッドを利用してBean定義したメールセッションをDIコンテナから取得する。
   * Javaソースから直接\ ``Session``\ のstaticファクトリメソッドを利用して取得する。
   
   なお、後述するSpringの\ ``JavaMailSenderImpl``\ を使用すると、メールセッションを直接扱わずにメールサーバと接続することも可能である。
   
   本ガイドラインでは、以下の二つの方法による実装例を紹介する。

   * メールセッションをJNDI経由で取得する方法
   * セッションを直接扱わずに\ ``JavaMailSenderImpl``\ のプロパティに接続情報を指定する方法

|

Spring FrameworkのMail連携用コンポーネントについて
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Frameworkはメール送信を行うためのコンポーネント（\ ``org.springframework.mail``\ パッケージ）を提供している。
このパッケージに含まれるコンポーネントはメール送信に係る詳細なロジックを隠蔽し、低レベルのAPIハンドリング(JavaMailのAPI呼び出し)を代行する。

具体的な実装方法の説明を行う前に、Spring Frameworkが提供するメール送信用のコンポーネントがどのようにメールを送信しているかを説明する。

.. figure:: ./images_Email/EmailOverview.png
    :alt: Constitution of Spring Mail
    :width: 100%

.. raw:: latex

   \newpage

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 20 60
    :class: longtable

    * - 項番
      - コンポーネント
      - 説明
    * - | (1)
      - | アプリケーション
      - | \ ``JavaMailSender``\ のメソッドを呼び出し、メールの送信依頼を行う。
        |
        | \* 単純なメッセージを送信する場合は、\ ``SimpleMailMessage``\ を生成し宛先や本文を設定することでメールを送信することもできる。
    * - | (2)
      - | \ ``JavaMailSender``\
      - | アプリケーションから指定された\ ``MimeMessagePreparator``\ (JavaMailの\ ``MimeMessage``\ を作成するためのコールバックインターフェース)を呼び出し、メール送信用のメッセージ(\ ``MimeMessage``\ )の作成依頼を行う。
        |
        | \* \ ``SimpleMailMessage``\ を使用してメッセージを送信する場合はこの処理は呼びだされない。
    * - | (3)
      - | アプリケーション
        | (\ ``MimeMessagePreparator``\)
      - | \ ``MimeMessageHelper``\ のメソッドを利用して、メール送信用のメッセージを(\ ``MimeMessage``\ )の作成する。
        |
        | \* \ ``SimpleMailMessage``\ を使用してメッセージを送信する場合はこの処理は呼びだされない。
    * - | (4)
      - | \ ``JavaMailSender``\
      - | JavaMailのAPIを使用して、メールの送信依頼を行う。
    * - | (5)
      - | JavaMail
      - | メールサーバへメッセージを送信する。

.. raw:: latex

   \newpage

\

本ガイドラインでは、以下のインタフェースやクラスを使用してメール送信処理を実装する方法について説明する。

* \ ``JavaMailSender``\
    | JavaMail用のメール送信インターフェース。
    | JavaMailの\ `MimeMessage <http://docs.oracle.com/javaee/7/api/javax/mail/internet/MimeMessage.html>`_\ とSpringの\ ``SimpleMailMessage``\ の両方に対応している。
    | また、JavaMailの\ ``Session``\ の管理は\ ``JavaMailSender``\ の実装クラスによって行われるため、メール送信処理をコーディングする際に\ ``Session``\ を直接扱う必要がない。

* \ ``JavaMailSenderImpl``\
    | \ ``JavaMailSender``\ インタフェースの実装クラス。
    | このクラスでは、設定済みの\ ``Session``\をDIする方法と、プロパティに指定した接続情報から\ ``Session``\を作成する方法をサポートしている。

* \ ``MimeMessagePreparator``\
    | JavaMailの\ ``MimeMessage``\ を作成するためのコールバックインターフェース。
    | \ ``JavaMailSender``\ の\ ``send``\ メソッド内から呼び出される。
    | \ ``MimeMessagePreparator``\ の\ ``prepare``\ メソッドで発生した例外は\ ``MailPreparationException``\ （実行時例外）にラップされ再スローされる。

* \ ``MimeMessageHelper``\
    | JavaMailの\ ``MimeMessage``\ の作成を容易にするためのヘルパークラス。
    | \ ``MimeMessageHelper``\ には、\ ``MimeMessage``\ に値を設定するための便利なメソッドがいくつも用意されている。

* \ ``SimpleMailMessage``\
    | 単純なメールメッセージを作成するためのクラス。
    | 英文のプレーンテキストメールを作成する際に使用できる。
    | UTF-8等の特定のエンコード指定、HTMLメールや添付ファイル付きメールの送信、あるいはメールアドレスに個人名を付随させるといったリッチなメッセージの作成を行う際は、JavaMailの\ ``MimeMessage``\ を使用する必要がある。

How to use
--------------------------------------------------------------------------------

依存ライブラリについて
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring FrameworkのMail連携用コンポーネントを利用する場合、以下のライブラリが追加で必要となる。

* `JavaMail <https://java.net/projects/javamail/pages/Home>`_

| 上記ライブラリに対する依存関係を\ :file:`pom.xml`\ に追加する。
| マルチプロジェクト構成の場合は、domainプロジェクトの\ :file:`pom.xml`\ (:file:`projectName-domain/pom.xml`)に追加する。

.. code-block:: xml

    <dependencies>

        <!-- (1) -->
        <dependency>
            <groupId>com.sun.mail</groupId>
            <artifactId>javax.mail</artifactId>
        </dependency>

    </dependencies>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | JavaMailのライブラリをdependenciesに追加する。
        | アプリケーションサーバ提供のメールセッションを使用する場合、\ ``<scope>``\ を\ ``provided``\ に設定する。

.. note::

    上記設定例は、依存ライブラリのバージョンを親プロジェクトである terasoluna-gfw-parent で管理する前提であるため、pom.xmlでのバージョンの指定は不要である。
    上記の依存ライブラリはterasoluna-gfw-parentが利用している\ `Spring IO Platform <http://platform.spring.io/platform/>`_\ で定義済みである。

|

JavaMailSenderの設定方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ ``JavaMailSender``\ をDIするためのBean定義を行う。

.. note::

    マルチプロジェクト構成の場合は、envプロジェクトの\ :file:`projectName-env.xml`\ に設定することを推奨する。
    なお、本ガイドラインでは、マルチプロジェクト構成を採用することを推奨している。


アプリケーションサーバ提供のメールセッションを使用する場合
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

アプリケーションサーバ提供のメールセッションを使用する場合の設定例を以下に示す。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table:: **アプリケーションサーバから提供されているメールセッション**
    :header-rows: 1
    :widths: 10 35 55

    * - 項番
      - アプリケーションサーバ
      - 参照ページ
    * - 1.
      - Apache Tomcat 8
      - | \ `Apache Tomcat 8 User Guide(JNDI Resources HOW-TO) <http://tomcat.apache.org/tomcat-8.0-doc/jndi-resources-howto.html#JavaMail_Sessions>`_\ (JavaMail Sessions)を参照されたい。
    * - 2.
      - Oracle WebLogic Server 12c
      - \ `Oracle WebLogic Server 12.2.1.0 Documentation <http://docs.oracle.com/middleware/1221/wls/WLACH/taskhelp/mail/CreateMailSessions.html>`_\ を参照されたい。
    * - 3.
      - IBM WebSphere Application Server Version 8.5
      - \ `WebSphere Application Server Version 8.5.5 documentation <https://www.ibm.com/support/knowledgecenter/SSD28V_8.5.5/com.ibm.websphere.wlp.core.doc/ae/twlp_admin_javamail.html>`_\ を参照されたい。
    * - 4.
      - Red Hat JBoss Enterprise Application Platform Version 6.4
      - \ `Product Documentation <https://access.redhat.com/documentation/en-US/JBoss_Enterprise_Application_Platform/6.4/html/Administration_and_Configuration_Guide/chap-Mail_subsystem.html>`_\ を参照されたい。


JNDI経由で取得したメールセッションをBeanとして登録するための設定を行う。

.. code-block:: xml

   <jee:jndi-lookup id="mailSession" jndi-name="mail/Session" /> <!-- (1) -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``<jee:jndi-lookup>``\ 要素の\ ``jndi-name``\ 属性に、アプリケーションサーバ提供のメールセッションのJNDI名を指定する。


次に、\ ``JavaMailSender``\ をBean定義する。

.. code-block:: xml

   <!-- (1) -->
   <bean id="mailSender" class="org.springframework.mail.javamail.JavaMailSenderImpl">
       <property name="session" ref="mailSession" /> <!-- (2) -->
   </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``JavaMailSenderImpl``\ をBean定義する。
   * - | (2)
     - | \ ``session``\ プロパティに設定済みのメールセッションのBeanを指定する。


アプリケーションサーバ提供のメールセッションを使用しない場合（認証なし）
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

認証が必要ない場合の設定例を以下に示す。

\ ``JavaMailSender``\ をBean定義する。

.. code-block:: xml

   <!-- (1) -->
   <bean id="mailSender" class="org.springframework.mail.javamail.JavaMailSenderImpl">
       <property name="host" value="${mail.smtp.host}"/> <!-- (2) -->
       <property name="port" value="${mail.smtp.port}"/> <!-- (3) -->
   </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``JavaMailSenderImpl``\ をBean定義する。
   * - | (2)
     - | \ ``host``\ プロパティにSMTPサーバのホスト名を指定する。
       | この例では、プロパティファイルで定義した値（キー「\ ``mail.smtp.host``\ 」に対する値）を設定している。
   * - | (3)
     - | \ ``port``\ プロパティにSMTPサーバのポート番号を指定する。
       | この例では、プロパティファイルで定義した値（キー「\ ``mail.smtp.port``\ 」に対する値）を設定している。

.. note::

   プロパティファイルについての詳細は、:doc:`../GeneralFuncDetail/PropertyManagement` を参照されたい。


アプリケーションサーバ提供のメールセッションを使用しない場合（認証あり）
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

認証が必要な場合の設定例を以下に示す。

\ ``JavaMailSender``\ をBean定義する。

.. code-block:: xml

   <bean id="mailSender" class="org.springframework.mail.javamail.JavaMailSenderImpl">
       <property name="host" value="${mail.smtp.host}"/>
       <property name="port" value="${mail.smtp.port}"/>
       <property name="username" value="${mail.smtp.user}"/> <!-- (1) -->
       <property name="password" value="${mail.smtp.password}"/> <!-- (2) -->
       <property name="javaMailProperties">
           <props>
               <prop key="mail.smtp.auth">true</prop> <!-- (3) -->
           </props>
       </property>
   </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``username``\ プロパティにSMTPサーバのユーザ名を指定する。
       | この例では、プロパティファイルで定義した値（キー「\ ``mail.smtp.user``\ 」に対する値）を設定している。
   * - | (2)
     - | \ ``password``\ プロパティにSMTPサーバのパスワードを指定する。
       | この例では、プロパティファイルで定義した値（キー「\ ``mail.smtp.password``\ 」に対する値）を設定している。
   * - | (3)
     - | \ ``javaMailProperties``\ プロパティにキー「\ ``mail.smtp.auth``\ 」として\ ``true``\ を設定する。

.. note::

   プロパティファイルについての詳細は、:doc:`../GeneralFuncDetail/PropertyManagement` を参照されたい。

.. tip::

   TLSによる接続が必要な場合、\ ``javaMailProperties``\ プロパティにキー「\ ``mail.smtp.starttls.enable``\ 」として\ ``true``\ を設定する。
   なお、左記のとおり指定した場合でもSMTPサーバがSTARTTLSをサポートしていない場合は平文による通信が行われる。
   必要に応じて\ ``javaMailProperties``\ プロパティにキー「\ ``mail.smtp.starttls.required``\ 」として\ ``true``\ を設定することで、STARTTLSを利用できない場合にエラーとすることも可能である。

|

SimpleMailMessageによるメール送信方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

英文のプレーンテキストメール（エンコードの指定や添付ファイル等が不要なメール）を送信する場合は、Springが提供している\ ``SimpleMailMessage``\ クラスを使用する。

以下に、\ ``SimpleMailMessage``\ クラスを使用したメール送信方法を説明する。

**Bean定義例**

.. code-block:: xml

   <!-- (1) -->
   <bean id="templateMessage" class="org.springframework.mail.SimpleMailMessage">
       <property name="from" value="info@example.com" /> <!-- (2) -->
       <property name="subject" value="Registration confirmation." /> <!-- (3) -->
   </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | テンプレートとして\ ``SimpleMailMessage``\ をBean定義する。
       | テンプレートの\ ``SimpleMailMessage``\ を利用するのは必須ではないが、メールメッセージで固定的な箇所（例えば送信元メールアドレス等）をテンプレート化しておくことで、メールメッセージ作成時に個別に設定する必要がなくなる。
   * - | (2)
     - | \ ``from``\ プロパティにFromヘッダの内容を指定する。
   * - | (3)
     - | \ ``subject``\ プロパティにSubjectヘッダの内容を指定する。

**Javaクラスの実装例**

.. code-block:: java

    @Inject
    JavaMailSender mailSender; // (1)

    @Inject
    SimpleMailMessage templateMessage; // (2)

    public void register(User user) {
        // omitted
        
        // (3)
        SimpleMailMessage message = new SimpleMailMessage(templateMessage);
        message.setTo(user.getEmailAddress());
        String text = "Hi "
                + user.getUserName()
                + ", welcome to EXAMPLE.COM!\r\n"
                + "If you were not an intended recipient, Please notify the sender.";
        message.setText(text);
        mailSender.send(message);
        
        // omitted
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``JavaMailSender``\ をインジェクションする。
   * - | (2)
     - | テンプレートとしてBean定義した\ ``SimpleMailMessage``\ をインジェクションする。
   * - | (3)
     - | テンプレートのBeanを利用して\ ``SimpleMailMessage``\ のインスタンスを生成し、Toヘッダと本文を設定して送信する。

.. note::

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}||p{0.60\linewidth}|
    .. list-table:: **SimpleMailMessageで設定可能なプロパティ**
       :header-rows: 1
       :widths: 10 30 60

       * - 項番
         - プロパティ
         - 説明
       * - | 1.
         - | \ ``from``\ 
         - | Fromヘッダを設定する。
       * - | 2.
         - | \ ``to``\ 
         - | Toヘッダを設定する。
       * - | 3.
         - | \ ``cc``\ 
         - | Ccヘッダを設定する。
       * - | 4.
         - | \ ``bcc``\ 
         - | Bccヘッダを設定する。
       * - | 5.
         - | \ ``subject``\ 
         - | Subjectヘッダを設定する。
       * - | 6.
         - | \ ``replyTo``\ 
         - | Reply-Toヘッダを設定する。
       * - | 7.
         - | \ ``sentDate``\ 
         - | Dateヘッダを設定する。
           | なお、明示的に設定しない場合は送信時にシステム時刻（\ ``new Date()``\ ）が自動設定される。
       * - | 8.
         - | \ ``text``\ 
         - | 本文を設定する。

   To、Cc、Bccに複数の宛先を設定する場合は配列にして設定する。
   

.. warning::

   メールヘッダを設定する場合、メールヘッダ・インジェクションに対する考慮が必要となる。
   詳細は\ :ref:`email-header-injection`\ を参照されたい。

|

MimeMessageによるメール送信方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

英文以外のメールやHTMLメール、添付ファイルの送信を行う場合、
\ ``javax.mail.internet.MimeMessage``\ クラスを使用する。
本ガイドラインでは\ ``MimeMessageHelper``\ クラスを使用して\ ``MimeMessage``\ を作成する方法を推奨している。

本項では、\ ``MimeMessageHelper``\ クラスを使用した以下のメール送信方法を説明する。

* :ref:`email-text`
* :ref:`email-html`
* :ref:`email-attachment`
* :ref:`email-inline-resource`

.. _email-text:

テキストメールの送信
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``MimeMessageHelper``\ クラスを使用して、テキストメールを送信する実装例を以下に示す。

**Javaクラスの実装例**

.. code-block:: java

    @Inject
    JavaMailSender mailSender; // (1)

    public void register(User user) {
        // omitted
        
        // (2)
        mailSender.send(new MimeMessagePreparator() {

            @Override
            public void prepare(MimeMessage mimeMessage) throws Exception {
                MimeMessageHelper helper = new MimeMessageHelper(mimeMessage,
                        StandardCharsets.UTF_8.name()); // (3)
                helper.setFrom("EXAMPLE.COM <info@example.com>"); // (4)
                helper.setTo(user.getEmailAddress()); // (5)
                helper.setSubject("Registration confirmation."); // (6)
                String text = "Hi "
                        + user.getUserName()
                        + ", welcome to EXAMPLE.COM!\r\n"
                        + "If you were not an intended recipient, Please notify the sender.";
                helper.setText(text); // (7)
            }
        });
        
        // omitted
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``JavaMailSender``\ をインジェクションする。
   * - | (2)
     - | \ ``JavaMailSender``\ の\ ``send``\ メソッドを利用してメールを送信する。
       | 引数には\ ``MimeMessagePreparator``\ を実装した匿名内部クラスを定義する。
   * - | (3)
     - | 文字コードを指定して、\ ``MimeMessageHelper``\ のインスタンスを生成する。
       | この例では、文字コードにUTF-8を指定している。
   * - | (4)
     - | Fromヘッダの内容を設定する。
       | この例では、"名前 <アドレス>"の形式で設定している。
   * - | (5)
     - | Toヘッダの内容を設定する。
   * - | (6)
     - | Subjectヘッダの内容を設定する。
   * - | (7)
     - | 本文の内容を設定する。

.. warning::

   メールヘッダを設定する場合、メールヘッダ・インジェクションに対する考慮が必要となる。
   詳細は\ :ref:`email-header-injection`\ を参照されたい。

.. note::

   日本語のメールを送信する際、UTF-8をサポートしていないメールクライアントもサポートする必要がある場合はエンコードにISO-2022-JPを利用することも考えられる。
   エンコードにISO-2022-JPを利用する際に考慮すべき事項について、\ :ref:`email-iso-2022-jp`\ を参照されたい。

.. _email-html:

HTMLメールの送信
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``MimeMessageHelper``\ クラスを使用して、HTMLメールを送信する実装例を以下に示す。

**Javaクラスの実装例**

.. code-block:: java

    @Inject
    JavaMailSender mailSender; // (1)

    public void register(User user) {
        // omitted
        
        // (2)
        mailSender.send(new MimeMessagePreparator() {

            @Override
            public void prepare(MimeMessage mimeMessage) throws Exception {
                MimeMessageHelper helper = new MimeMessageHelper(mimeMessage,
                        StandardCharsets.UTF_8.name()); // (3)
                helper.setFrom("EXAMPLE.COM <info@example.com>"); // (4)
                helper.setTo(user.getEmailAddress()); // (5)
                helper.setSubject("Registration confirmation."); // (6)
                String text = "<html><body><h3>Hi "
                        + user.getUserName()
                        + ", welcome to EXAMPLE.COM!</h3>"
                        + "If you were not an intended recipient, Please notify the sender.</body></html>";
                helper.setText(text, true); // (7)
            }
        });
        
        // omitted
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``JavaMailSender``\ をインジェクションする。
   * - | (2)
     - | \ ``JavaMailSender``\ の\ ``send``\ メソッドを利用してメールを送信する。
       | 引数には\ ``MimeMessagePreparator``\ を実装した匿名内部クラスを定義する。
   * - | (3)
     - | 文字コードを指定して、\ ``MimeMessageHelper``\ のインスタンスを生成する。
       | この例では、文字コードにUTF-8を指定している。
   * - | (4)
     - | Fromヘッダの内容を設定する。
       | この例では、"名前 <アドレス>"の形式で設定している。
   * - | (5)
     - | Toヘッダの内容を設定する。
   * - | (6)
     - | Subjectヘッダの内容を設定する。
   * - | (7)
     - | 本文の内容を設定する。\ ``setText``\ メソッドの第二引数に\ ``true``\ を指定することで、Content-Typeがtext/htmlになる。

.. warning::

   メール本文のHTMLを生成する際に外部から入力された値を使用する場合はXSS攻撃への対策を行うこと。


.. _email-attachment:

添付ファイル付きメールの送信
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``MimeMessageHelper``\ クラスを使用して、添付ファイル付きメールを送信する実装例を以下に示す。

**Javaクラスの実装例**

.. code-block:: java

    @Inject
    JavaMailSender mailSender; // (1)

    public void register(User user) {
        // omitted
        
        // (2)
        mailSender.send(new MimeMessagePreparator() {

            @Override
            public void prepare(MimeMessage mimeMessage) throws Exception {
                MimeMessageHelper helper = new MimeMessageHelper(mimeMessage,
                        true, StandardCharsets.UTF_8.name()); // (3)
                helper.setFrom("EXAMPLE.COM <info@example.com>"); // (4)
                helper.setTo(user.getEmailAddress()); // (5)
                helper.setSubject("Registration confirmation."); // (6)
                String text = "Hi "
                        + user.getUserName()
                        + ", welcome to EXAMPLE.COM!\r\n"
                        + "Please find attached the file.\r\n\r\n"
                        + "If you were not an intended recipient, Please notify the sender.";
                helper.setText(text); // (7)
                ClassPathResource file = new ClassPathResource("doc/quickstart.pdf");
                helper.addAttachment("QuickStart.pdf", file); // (8)
            }
        });
        
        // omitted
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``JavaMailSender``\ をインジェクションする。
   * - | (2)
     - | \ ``JavaMailSender``\ の\ ``send``\ メソッドを利用してメールを送信する。
       | 引数には\ ``MimeMessagePreparator``\ を実装した匿名内部クラスを定義する。
   * - | (3)
     - | 文字コードを指定して、\ ``MimeMessageHelper``\ のインスタンスを生成する。
       | この例では、文字コードにUTF-8を指定している。
       | \ ``MimeMessageHelper``\ のコンストラクタの第二引数に\ ``true``\ を指定することで、マルチパートモード（デフォルトの\ ``MULTIPART_MODE_MIXED_RELATED``\ ）になる。
   * - | (4)
     - | Fromヘッダの内容を設定する。
   * - | (5)
     - | Toヘッダの内容を設定する。
   * - | (6)
     - | Subjectヘッダの内容を設定する。
   * - | (7)
     - | 本文の内容を設定する。
   * - | (8)
     - | 添付ファイル名を指定して添付するファイルを設定する。
       | この例では、\ ``"QuickStart.pdf"``\ というファイル名で、クラスパス上にある\ :file:`doc/quickstart.pdf`\ というファイルを添付している。


.. _email-inline-resource:

インラインリソース付きメールの送信
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``MimeMessageHelper``\ クラスを使用して、インラインリソース付きメールを送信する実装例を以下に示す。

**Javaクラスの実装例**

.. code-block:: java

    @Inject
    JavaMailSender mailSender; // (1)

    public void register(User user) {
        // omitted
        
        // (2)
        mailSender.send(new MimeMessagePreparator() {

            @Override
            public void prepare(MimeMessage mimeMessage) throws Exception {
                MimeMessageHelper helper = new MimeMessageHelper(mimeMessage,
                        true, StandardCharsets.UTF_8.name()); // (3)
                helper.setFrom("EXAMPLE.COM <info@example.com>"); // (4)
                helper.setTo(user.getEmailAddress()); // (5)
                helper.setSubject("Registration confirmation."); // (6)
                String cid = "identifier1234";
                String text = "<html><body><img src='cid:"
                        + cid
                        + "' /><h3>Hi "
                        + user.getUserName()
                        + ", welcome to EXAMPLE.COM!\r\n</h3>"
                        + "If you were not an intended recipient, Please notify the sender.</body></html>";
                helper.setText(text, true); // (7)
                ClassPathResource res = new ClassPathResource("image/logo.jpg");
                helper.addInline(cid, res); // (8)
            }
        });
        
        // omitted
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``JavaMailSender``\ をインジェクションする。
   * - | (2)
     - | \ ``JavaMailSender``\ の\ ``send``\ メソッドを利用してメールを送信する。
       | 引数には\ ``MimeMessagePreparator``\ を実装した匿名内部クラスを定義する。
   * - | (3)
     - | 文字コードを指定して、\ ``MimeMessageHelper``\ のインスタンスを生成する。
       | この例では、文字コードにUTF-8を指定している。
       | \ ``MimeMessageHelper``\ のコンストラクタの第二引数に\ ``true``\ を指定することで、マルチパートモードになる。
   * - | (4)
     - | Fromヘッダの内容を設定する。
   * - | (5)
     - | Toヘッダの内容を設定する。
   * - | (6)
     - | Subjectヘッダの内容を設定する。
   * - | (7)
     - | 本文の内容を設定する。\ ``setText``\ メソッドの第二引数に\ ``true``\ を指定することで、Content-Typeがtext/htmlになる。
   * - | (8)
     - | インラインリソースのコンテンツIDを指定してインラインリソースを設定する。
       | この例では、\ ``"identifier1234"``\ というコンテンツIDで、クラスパス上にある\ :file:`image/logo.jpg`\ というファイルを設定している。

.. note::

   \ ``addInline``\ メソッド\ は、``setText``\ メソッドの後に呼び出すこと。
   そうしないと、メールクライアントがインラインリソースを正しく参照できないことがある。

|

メール送信時の例外について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ ``JavaMailSender``\ の\ ``send``\ メソッドを利用してメール送信を行う際に発生する例外は\ ``org.springframework.mail.MailException``\ を継承した例外である。
\ ``MailException``\ を継承した例外クラスと、それぞれの例外の発生条件について、以下の表に示す。

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table:: **メール送信時の例外**
    :header-rows: 1
    :widths: 10 35 55

    * - 項番
      - 例外クラス
      - 発生条件
    * - 1.
      - `MailAuthenticationException <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/mail/MailAuthenticationException.html>`_
      - | 認証失敗時に発生する。
    * - 2.
      - `MailParseException <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/mail/MailParseException.html>`_
      - | メールメッセージのプロパティに不正な値が設定されている場合に発生する。
    * - 3.
      - `MailPreparationException <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/mail/MailPreparationException.html>`_
      - | メールメッセージを作成中に想定外のエラーが起きた場合に発生する。
          想定外のエラーとしては、例えばテンプレートライブラリで発生するエラーといったものがある。
        | \ ``MimeMessagePreparator``\ で発生した例外が\ ``MailPreparationException``\ にラップされてスローされる。
    * - 4.
      - `MailSendException <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/mail/MailSendException.html>`_
      - | メールの送信エラーが起きた場合に発生する。

.. note::

   特定の例外に対するエラー画面遷移については、 :doc:`../WebApplicationDetail/ExceptionHandling` を参照されたい。

|

How to extend
--------------------------------------------------------------------------------

テンプレートを使用したメール本文の作成方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

上で示した実装例のようにJavaソースでメール本文を直接組み立てるのは、以下の理由から推奨しない。

* メール本文をJavaソースで組み立てるのは可読性が悪くエラーを作りやすい。
* 表示ロジックとビジネスロジックの境界が曖昧となる。
* メール本文のデザインを変更するために、Javaソースの修正、コンパイル、デプロイが必要になる。

よって、メール本文のデザインを定義するためにテンプレートライブラリを使用することを推奨する。
特にメール本文が複雑になるような場合はテンプレートライブラリを使用すべきである。

FreeMarkerを使用したメール本文の作成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

本ガイドラインでは、テンプレートライブラリとして\ `FreeMarker <http://freemarker.org/>`_\ を使用する方法について説明する。

* FreeMarkerを使用するために、依存ライブラリを設定する。

    **pom.xmlの設定例**
    
    .. code-block:: xml
    
        <dependencies>
    
            <!-- (1) -->
            <dependency>
                <groupId>org.freemarker</groupId>
                <artifactId>freemarker</artifactId>
            </dependency>
    
        </dependencies>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
        :header-rows: 1
        :widths: 10 90
    
        * - 項番
          - 説明
        * - | (1)
          - | FreeMarkerのライブラリをdependenciesに追加する。

    .. note::

       上記設定例は、依存ライブラリのバージョンを親プロジェクトである terasoluna-gfw-parent で管理する前提であるため、pom.xmlでのバージョンの指定は不要である。
       上記の依存ライブラリはterasoluna-gfw-parentが利用している\ `Spring IO Platform <http://platform.spring.io/platform/>`_\ で定義済みである。


* \ ``freemarker.template.Configuration``\ を生成するためのFactoryBeanをBean定義する。

    **Bean定義ファイルの設定例**
    
    .. code-block:: xml
    
       <!-- (1) -->
       <bean id="freemarkerConfiguration"
           class="org.springframework.ui.freemarker.FreeMarkerConfigurationFactoryBean">
           <property name="templateLoaderPath" value="classpath:/META-INF/freemarker/" /> <!-- (2) -->
           <property name="defaultEncoding" value="UTF-8" /> <!-- (3) -->
       </bean>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - 項番
         - 説明
       * - | (1)
         - | \ ``FreeMarkerConfigurationFactoryBean``\ をBean定義する。
       * - | (2)
         - | \ ``templateLoaderPath``\ プロパティにテンプレートファイルの格納された場所を指定する。
           | この例では、クラスパス上にある\ :file:`META-INF/freemarker/`\ ディレクトリを設定している。
       * - | (3)
         - | \ ``defaultEncoding``\ プロパティにデフォルトのエンコードを指定する。
           | この例では、UTF-8を設定している。

    .. note::

       上記以外の設定については、\ `FreeMarkerConfigurationFactoryBeanのJavaDoc <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/ui/freemarker/FreeMarkerConfigurationFactoryBean.html>`_\ を参照されたい。
       また、FreeMarker自体の設定については、\ `FreeMarker Manual (Programmer's Guide / The Configuration) <http://freemarker.org/docs/pgui_config.html>`_\ を参照されたい。

* メール本文のテンプレートファイルを作成する。

    **テンプレートファイルの設定例**
    
    .. code-block:: text
    
       <#escape x as x?html> <#-- (1) -->
       <html>
           <body>
               <h3>Hi ${userName}, welcome to TERASOLUNA.ORG!</h3> <#-- (2) -->
    
               <div>
                   If you were not an intended recipient, Please notify the sender.
               </div>
           </body>
       </html>
       </#escape>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - 項番
         - 説明
       * - | (1)
         - | XSS攻撃への対策としてHTMLエスケープを行うように設定している。
       * - | (2)
         - | データモデルに設定された\ ``userName``\ の値を埋め込む。

    .. note::

       テンプレート言語（FTL）の詳細については、\ `FreeMarker Manual (Template Language Reference) <http://freemarker.org/docs/ref.html>`_\ を参照されたい。

* テンプレートを使用してメール本文を生成し、メール送信する。

    **Javaクラスの実装例**
    
    .. code-block:: java
    
        @Inject
        JavaMailSender mailSender;
    
    	@Inject
    	Configuration freemarkerConfiguration; // (1)
    	
        public void register(User user) {
            // omitted
            
            mailSender.send(new MimeMessagePreparator() {

                @Override
                public void prepare(MimeMessage mimeMessage) throws Exception {
                    MimeMessageHelper helper = new MimeMessageHelper(mimeMessage,
                            StandardCharsets.UTF_8.name());
                    helper.setFrom("EXAMPLE.COM <info@example.com>");
                    helper.setTo(user.getEmailAddress());
                    helper.setSubject("Registration confirmation.");
                    Template template = freemarkerConfiguration
                            .getTemplate("registration-confirmation.ftl"); // (2)
                    String text = FreeMarkerTemplateUtils
                            .processTemplateIntoString(template, user); // (3)
                    helper.setText(text, true);
                }
            });
            
            // omitted
        }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - 項番
         - 説明
       * - | (1)
         - | \ `Configuration <http://freemarker.org/docs/api/freemarker/template/Configuration.html>`_\ をインジェクションする。
       * - | (2)
         - | \ ``Configuration``\ の\ ``getTemplate``\ メソッドを利用して\ `Template <http://freemarker.org/docs/api/freemarker/template/Template.html>`_\ を取得する。
           | この例では、テンプレートファイルとして"registration-confirmation.ftl"を指定している。
       * - | (3)
         - | 取得した\ ``Template``\ をもとに、\ ``org.springframework.ui.freemarker.FreeMarkerTemplateUtils``\ の\ ``processTemplateIntoString``\ メソッドを利用してテンプレートから文字列を生成する。
           | この例では、データモデルとして\ ``userName``\ プロパティを持つ\ ``User``\ オブジェクト（JavaBeans）を指定している。
             これにより、テンプレートファイルの\ ``${userName}``\ の箇所に\ ``userName``\ プロパティの値が埋め込まれる。

|

Appendix
--------------------------------------------------------------------------------

.. _email-iso-2022-jp:

ISO-2022-JPのエンコードについての考慮
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

日本語のメールを送信する際、送信したメールを受信するメールクライアントを限定できない場合は、エンコードにISO-2022-JPを利用することを検討する必要がある。
この理由としては、レガシーなメールクライアントがUTF-8に対応していない場合を考慮するためである。

MS932で入力された文字列に対し、エンコードにISO-2022-JPをはじめとするJIS X 0208の文字集合をベースとしたエンコードを設定した場合、
以下の表に記載する7文字において文字化けが発生する。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.10\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|p{0.20\linewidth}|
.. list-table::
   :header-rows: 2
   :widths: 20 15 15 15 15 20

   * - 変換前
     - 
     - 
     - 変換後
     - 
     - 
   * - | MS932
       | 入力文字
     - | 入力値
       | （SJIS）
     - | Unicode
       | （UTF-16）
     - | Unicode
       | （UTF-16）
     - | ISO-2022-JP
       | （JIS）
     - | JIS X 0208
       | 代替文字
   * - | ―（全角ハイフン）
     - | 815D
     - | U+2015
     - | U+2014
     - | 213E
     - | —（EM ダッシュ）
   * - | －（ハイフンマイナス）
     - | 817C
     - | U+FF0D
     - | U+2212
     - | 215D
     - | −（全角マイナス）
   * - | ～（全角チルド）
     - | 8160
     - | U+FF5E
     - | U+301C
     - | 2141
     - | 〜（波ダッシュ）
   * - | ∥（平行記号）
     - | 8161
     - | U+2225
     - | U+2016
     - | 2142
     - | ‖（双柱）
   * - | ￠（全角セント記号）
     - | 8191
     - | U+FFE0
     - | U+00A2
     - | 2171
     - | ¢（セント記号）
   * - | ￡（全角ポンド記号）
     - | 8192
     - | U+FFE1
     - | U+00A3
     - | 2172
     - | £（ポンド記号）
   * - | ￢（全角否定記号）
     - | 81CA
     - | U+FFE2
     - | U+00AC
     - | 224C
     - | ¬（否定記号）

この問題は、Unicodeを介して文字コード変換を行う際に、MS932に有りJIS X 0208に無い文字が存在するためであり、
文字化けを回避するためには、文字化けする文字について代替文字に文字コードを置き換えるなどの対処を行う必要がある。
なお、後述するx-windows-iso2022jpを使用する場合、変換処理は不要である。

以下に、変換処理の実装例を示す。

.. code-block:: java

    public static String convertISO2022JPCharacters(String targetStr) {

        if (targetStr == null) {
            return null;
        }

        char[] ch = targetStr.toCharArray();

        for (int i = 0; i < ch.length; i++) {
            switch (ch[i]) {

            // '―'（全角ハイフン）
            case '\u2015':
                ch[i] = '\u2014';
                break;
            // '－'（全角マイナス）
            case '\uff0d':
                ch[i] = '\u2212';
                break;
            // '～'（波ダッシュ）
            case '\uff5e':
                ch[i] = '\u301c';
                break;
            // '∥'（双柱）
            case '\u2225':
                ch[i] = '\u2016';
                break;
            // '￠'（セント記号)
            case '\uffe0':
                ch[i] = '\u00A2';
                break;
            // '￡'（ポンド記号）
            case '\uffe1':
                ch[i] = '\u00A3';
                break;
            // '￢'（否定記号）
            case '\uffe2':
                ch[i] = '\u00AC';
                break;
            default:
                break;
            }
        }

        return String.valueOf(ch);
    }

.. note::

   Unicodeへのマッピング時の問題であるため、入力値の文字コードに依らず変換は必要である。
   変換対象となるのは日本語を含む文字列が設定される可能性のあるヘッダおよび本文の文字列である。
   日本語を含む可能性があり一般的によく使われると考えられるヘッダとしては、From、To、Cc、Bcc、Reply-To、Subjectが挙げられる。

また、エンコードにISO-2022-JPを設定する場合、以下のような範囲外となる拡張文字が文字化けする。

.. figure:: ./images_Email/EmailOutofEscapeCharacter.png
    :alt: Out of EscapeCharacter
    :width: 100%
    :align: center
    
    **図-範囲外となる拡張文字の例**

これらの文字は本来使用すべきではない。
もし、これらの文字を使用する必要がある場合、JVMの起動オプションとして以下のように設定することで
ISO-2022-JPのエンコードが指定された場合にx-windows-iso2022jpでマッピングするように差し替えることが可能である。

.. code-block:: text

   -Dsun.nio.cs.map=x-windows-iso2022jp/ISO-2022-JP

.. warning::

   x-windows-iso2022jpはISO-2022-JPの規格と異なるマッピング（MS932ベース）を含むISO-2022-JP実装である。   
   メールヘッダでISO-2022-JPのエンコードが指定された場合に範囲外の拡張文字を扱えるような実装となっているかはメールクライアントに依存する。
   このため、x-windows-iso2022jpを使用してマッピングした場合でも、すべてのメールクライアントで確実に文字化けしないことが保証されるわけではない。

拡張文字を代替文字に変換してもよい場合、前述した7文字と同様にアプリケーションで独自に変換を行う方法も合わせて検討されたい。

|

.. _email-note-of-when-sending:

JavaMailでマルチバイト文字を使用する際の注意点
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

JavaMailでは、送信するメールの本文の終端がマルチバイト文字で終わっていると、終端に余計な文字（「?」や「w)」等）が出力される場合がある。  
本事象は以下の方法で回避できる。  

* メール本文の終端文字を半角文字にする  
* メール本文の終端を改行コード（CRLF）にする  

|

.. _email-header-injection:

メールヘッダ・インジェクション対策
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

メールヘッダ・インジェクション攻撃が成功すると、本来意図していない宛先にメール送信され、
迷惑メール送信の踏み台に悪用される可能性がある。
メールヘッダ（Subject等）の内容に外部から入力された文字列を利用する場合、メールヘッダ・インジェクション攻撃への対策が必要となる。

例えば、\ ``MimeMessageHelper``\ の\ ``setSubject``\ メソッドで以下の文字列を設定すると、Bccヘッダを追加し本文を改ざんすることが可能となる。

.. code-block:: text

   Notification\r\nBcc: attacker@exapmle.com\r\n\r\nManipulated body.

メールヘッダ・インジェクション攻撃への対策としては、以下のような方法が考えられる。

* メールヘッダに設定する内容は固定値とし、外部から入力された文字列はすべてメール本文に出力する。
* メールヘッダに設定する内容に改行文字が含まれないことをチェックする。

|

.. _email-processing-method:

処理方式
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

メール送信は時間のかかる処理であるため、Webアプリケーションのリクエストの中で送信処理を行うと応答時間が長くなってしまう。
このため、通常はWebアプリケーションのリクエストの中では送信処理を行わず、非同期でメール送信を行う処理方式とすることが多い。
メール送信の処理方式について詳細については言及しないが、以下に一例を示すので参考にされたい。

データベースまたはメッセージキューに保持されたメール情報をもとにメール送信を行う
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

データベースまたはメッセージキューに保持されたメール情報をもとにメール送信を行うには、以下のような機能をアプリケーションに組み込む。

* 送信するメールの情報（宛先や本文、添付ファイル等）をデータベース（またはメッセージキュー）に登録する。
* データベース（またはメッセージキュー）から未送信のメール情報を定期的に取得し、SMTPによるメール送信を行う。
* 送信結果をデータベース（またはメッセージキュー）に登録する。

なお、以下の点を含めて検討する必要がある。

* 登録されたメール情報やメール送信結果の確認方法
* メール送信エラー時の取り扱い

.. tip::

   メールサービスによっては、連続してメールが送信された場合に、スパムメールと判定されることがある。
   左記への対策としては、同一ドメインに対し連続で送信処理を行わないように、送信順序をランダムにする方法が考えられる。

|

.. _email-test-with-greenmail:

GreenMailを利用したテスト
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

メール送信機能をテストするためにフェイクサーバとして\ `GreenMail <http://www.icegreen.com/greenmail/>`_\ を利用する方法を紹介する。
GreenMailはライブラリとして利用する以外に、warファイルをデプロイして利用することも可能である。

GreenMailを利用したテストコードの実装例を以下に示す。

**pom.xmlの設定例**

.. code-block:: xml

    <dependencies>

        <!-- (1) -->
        <dependency>
            <groupId>com.icegreen</groupId>
            <artifactId>greenmail</artifactId>
            <version>1.4.1</version>
            <scope>test</scope>
        </dependency>

    </dependencies>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | GreenMailのライブラリをdependenciesに追加する。

**JUnitソースの実装例**

.. code-block:: java

    @Inject
    EmailService emailService;

    @Rule
    public final GreenMailRule greenMail = new GreenMailRule(
            ServerSetupTest.SMTP); // (1)

    @Test
    public void testSend() {

        String from = "info@example.com";
        String to = "foo@example.com";
        String subject = "Registration confirmation.";
        String text = "Hi "
                + to
                + ", welcome to EXAMPLE.COM!\r\n"
                + "If you were not an intended recipient, Please notify the sender.";
        emailService.send(from, to, subject, text);

        assertTrue(greenMail.waitForIncomingEmail(3000, 1)); // (2)

        Message[] messages = greenMail.getReceivedMessages(); // (3)

        assertNotNull(messages);
        assertEquals(1, messages.length);
        // omitted
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``ServerSetupTest.SMTP``\ を指定した\ ``GreenMailRule``\ をルールとして設定する。
       | SMTPのポート番号はデフォルトで\ ``3025``\ が使用される。
   * - | (2)
     - | \ ``waitForIncomingEmail``\ メソッドを利用してメールの到達を待機する。
       | 別スレッドで非同期にメール送信が行われる際に利用する。
       | この例では、メール送信が非同期で行われている前提で、1通のメールが到達するまで最大3秒待機する。
   * - | (3)
     - | \ ``getReceivedMessages``\ メソッドを利用してすべての受信メールを取得する。
       | GreenMailで送信したメールは宛先に係らず、すべてGreenMailで受信される。

.. raw:: latex

   \newpage

