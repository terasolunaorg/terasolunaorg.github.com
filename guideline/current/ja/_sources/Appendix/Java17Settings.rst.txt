Java SE 17を使用するための設定
================================================================================

.. only:: html

.. contents:: 目次
  :depth: 3
  :local:

| 本章では、Java SE 17を使用して本ガイドラインで解説している機能に関連のある主要な設定方法を解説する。
| 詳細は、各章を参照されたい。

なお、本章は\ `Oracle JDK Migration Guide <https://docs.oracle.com/en/java/javase/17/migrate/getting-started.html>`_\ に基づいて執筆している。より理解を深めるため、こちらも一読されたい。

|

Java SE 9から非推奨となったJava EE関連モジュールの削除
--------------------------------------------------------------------------------

Java SE 9から非推奨であったJava EE関連のモジュールがJava SE 11で削除された。
詳細については、\ `Oracle JDK Migration GuideのRemoval of Java EE and CORBA Modules <https://docs.oracle.com/en/java/javase/17/migrate/removed-tools-and-components.html#GUID-460800EF-A523-4B10-B694-E3536AC419C1>`_\ を参照されたい。

これにより、本ガイドラインで解説している機能のいくつかを正常に動作させるには、これらの代替となる依存ライブラリを追加することが必要となる。
以下に、本ガイドラインで記載のあるモジュールを紹介する。

なお、以下で解説する依存ライブラリの追加はあくまで検証の一結果に過ぎず、アプリケーションの実装（依存しているライブラリの種類）と動作環境により異なる対応が必要な場合があることに留意されたい。

.. _remove-jaxb-from-java11:

JAXBの削除
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Java SE 17でJAXBを利用する場合、jakarta.xml.bind-api及びjaxb-implが必要となる。
アプリケーションの依存ライブラリやAPサーバから提供されるライブラリにjakarta.xml.bind-api及びjaxb-implがない場合は、下記のようにpom.xmlに依存関係を追加すること。

.. code-block:: xml

  <dependencies>
      <dependency>
          <groupId>jakarta.xml.bind</groupId>
          <artifactId>jakarta.xml.bind-api</artifactId> <!-- (1) -->
      </dependency>
  </dependencies>

  <dependencies>
      <dependency>
          <groupId>com.sun.xml.bind</groupId>
          <artifactId>jaxb-impl</artifactId>
          <version>${jaxb-impl.version}</version> <!-- (2) -->
          <scope>runtime</scope>
      </dependency>
  </dependencies>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | jakarta.xml.bind-apiのバージョンはterasoluna-gfw-parentが依存している\ `Spring Boot <https://docs.spring.io/spring-boot/docs/3.0.1/reference/htmlsingle/#dependency-versions>`_\ で管理されているため、pom.xmlでのバージョンの指定は不要である。
    * - | (2)
      - | Jakarta EE 10に準拠した任意のバージョンを指定する。

|

.. _remove-jax-ws-from-java11:

JAX-WSの削除
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Java SE 17でJAX-WSを利用する場合、以下のようにjakarta.xml.ws-api及びjaxws-rtを依存関係に追加する必要がある。

.. code-block:: xml

    <dependencies>
        <dependency>
            <groupId>jakarta.xml.ws</groupId>
            <artifactId>jakarta.xml.ws-api</artifactId> <!-- (1) -->
        </dependency>
    </dependencies>
    <dependencies>
        <dependency>
            <groupId>com.sun.xml.ws</groupId>
            <artifactId>jaxws-rt</artifactId>
            <version>${jaxws-rt.version}</version> <!-- (2) -->
            <scope>runtime</scope>
        </dependency>
    </dependencies>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | jakarta.xml.ws-apiのバージョンはterasoluna-gfw-parentが依存している\ `Spring Boot <https://docs.spring.io/spring-boot/docs/3.0.1/reference/htmlsingle/#dependency-versions>`_\ で管理されているため、pom.xmlでのバージョンの指定は不要である。
    * - | (2)
      - | Jakarta EE 10に準拠した任意のバージョンを指定する。

|

.. _remove-common-annotations-from-java11:

Common Annotationsの削除
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Java SE 17でCommon Annotationsを利用する場合、以下のようにjakarta.annotation-apiを依存関係に追加する必要がある。

.. code-block:: xml

  <dependency>
      <groupId>jakarta.annotation</groupId>
      <artifactId>jakarta.annotation-api</artifactId> <!-- (1) -->
  </dependency>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
  :header-rows: 1
  :widths: 10 90

  * - 項番
    - 説明
  * - | (1)
    - | jakarta.annotation-apiのバージョンはterasoluna-gfw-parentが依存している\ `Spring Boot <https://docs.spring.io/spring-boot/docs/3.0.1/reference/htmlsingle/#dependency-versions>`_\ で管理されているため、pom.xmlでのバージョンの指定は不要である。

|

.. _conflict-deprecated-modules-from-java11:

推移的に解決されるJava EE関連モジュールの競合
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Java SE 17以降での開発を円滑に行うため、いくつかのOSSライブラリはこれまでに説明したJava SE 17で削除されたJava EE関連モジュールを依存ライブラリとして推移的に解決してくれるよう改善されている。

\ **不幸にも、実行環境によってはこれが原因となりアプリケーションが起動しない、処理に問題が生じるといったケースがあるため、留意されたい。**\

| 具体的には、本来アプリケーションサーバのモジュールを参照していたところを部分的にアプリケーションの依存ライブラリを参照してしまい、バージョン不整合によるリンケージエラーや\ ``NoSuchMethodException``\ が発生する、エラーにはならないが期待した挙動と異なるといった不具合が発生する。
| この場合は、アプリケーションのビルド時にアプリケーションサーバから提供されるモジュールを除外する、アプリケーションサーバのクラスローダ設定によりアプリケーションの依存ライブラリを優先するといった対策が有効である。

|

.. _change-default-locale--data-from-java9:

デフォルトで使用されるロケール・データの変更
--------------------------------------------------------------------------------

Java SE 9以降では、Unicodeコンソーシアムの共通ロケール・データ・リポジトリ(CLDR)データがデフォルトのロケール・データとして有効化されている。

これにより、Java SE 9以降では、Java SE 8以前とは日付、時間、数値などの書式で文字列を出力した場合に結果が変わる可能性がある。

Java SE 17の標準の設定値から変更してJava SE 8以前と同じ書式で出力したい場合は、システム・プロパティ\ ``java.locale.providers``\ のCLDRの前にCOMPATを設定する必要がある。(例:\ ``java.locale.providers=COMPAT,CLDR,SPI``\ )

詳細については、\ `Oracle JDK Migration GuideのUse CLDR Locale Data by Default <https://docs.oracle.com/en/java/javase/17/migrate/removed-apis.html#GUID-A20F2989-BFA9-482D-8618-6CBB4BAAE310>`_\ を参照されたい。

|

.. _support-tls1.3-by-default-from-java11:

HTTP通信におけるTLS(Transport Layer Security) v1.3のサポート
--------------------------------------------------------------------------------

| Java SE 11より、TLS(Transport Layer Security) バージョン1.3がサポートされ、デフォルトで1.3が使用されるようになった。
|
| TLSのバージョンはJVMレベルで利用するバージョンを変更することが可能である。
| クライアントで使用するバージョンは、JVMのシステム・プロパティ\ ``jdk.tls.client.protocols``\ を設定することで変更可能である。
| APサーバを介さずに公開するサーバで使用するバージョンは、同様にシステム・プロパティ\ ``jdk.tls.server.protocols``\ を設定することで変更可能である。

詳細は\ `JDK 11 Release Notes <https://www.oracle.com/java/technologies/javase/11-relnote-issues.html#JDK-8145252>`_\ を参照されたい。

.. note::

  LinuxのSSL通信を制御するopenssl はバージョン1.1.1でTLS 1.3に対応するが、コンパイル済みのパッケージは頒布されておらず、開発者が自らコンパイルして組み込む必要がある。

  Tomcat等のミドルウェアはopensslを利用してHTTPS通信を行なうが、ミドルウェアが内包するopensslをアップデートするためには、ミドルウェア自体も再コンパイルする必要がある。同様に、opensslをアップデートすることによりOSの機能が正常に動作しなくなる可能性がある。
   
  このため、独自にopensslをコンパイルしてアップデートすることは、一般的な開発者には推奨しない。TLS 1.3に対応したopensslを内包したOSにアップデートして、環境を構築しなおすべきである。

|

.. _performance-differences-between-java:

ガベージコレクションの変更
--------------------------------------------------------------------------------

Java SE 9 以降のガベージコレクションはThe Garbage-First Garbage Collector (G1 GC) がデフォルトとなりました。

詳細は\ `Changes to Garbage Collection <https://docs.oracle.com/en/java/javase/17/migrate/migrating-jdk-8-later-jdk-releases.html#GUID-1F270BDA-50B0-49C8-807E-0B727CCC5169>`_\ を参照されたい。

.. raw:: latex

  \newpage

