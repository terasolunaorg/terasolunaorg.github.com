.. raw:: pdf

    PageBreak

Spring Security概要
================================================================================

.. contents:: 目次
   :local:

Overview
--------------------------------------------------------------------------------

| Spring Securityとは、アプリケーションのセキュリティを担う「認証」、「認可」の2つを
| 主な機能として提供している。
| 認証機能とは、なりすましによる不正アクセスに対抗するため、ユーザを識別する機能である。
| 認可機能とは、認証された（ログイン中の）ユーザの権限に応じて、
| システムのリソースに対するアクセス制御を行う機能である。

| Spring Securityの概要図を、以下に示す。

.. figure:: ./images/spring_security_overview.png
   :alt: Spring Security Overview
   :width: 80%
   :align: center

   **Picture - Spring Security Overview**

| Spring Securityは、認証、認可のプロセスを何層にも連なる
| ServletFilter の集まりで実現している。
| また、パスワードハッシュ機能や、JSPの認可タグライブラリなども提供している。

認証
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 認証とは、正当性を確認する行為であり、ネットワークやサーバへ接続する際に
| ユーザ名とパスワードの組み合わせを使って、利用ユーザにその権利があるかどうかや、
| その人が利用ユーザ本人であるかどうかを確認することである。
| Spring Securityでの使用方法は、\ :doc:`Authentication`\ を参照されたい。

パスワードハッシュ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 平文のパスワードから、ハッシュ関数を用いて計算されたハッシュ値を、元のパスワードと置き換えることである。
| Spring Securityでの使用方法は、\ :doc:`PasswordHashing`\ を参照されたい。

認可
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 認可とは、認証された利用者がリソースにアクセスしようとしたとき、
| アクセス制御処理でその利用者がそのリソースの使用を許可されていることを調べることである。
| Spring Securityでの使用方法は、\ :doc:`Authorization`\ を参照されたい。

|

.. _howtouse_springsecurity:

How to use
--------------------------------------------------------------------------------

| Spring Securityを使用するために、以下の設定を定義する必要がある。

pom.xmlの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Spring Securityを使用する場合、以下のdependencyを、pom.xmlに追加する必要がある。

.. code-block:: xml

    <dependency>
        <groupId>org.terasoluna.gfw</groupId>
        <artifactId>terasoluna-gfw-security-core</artifactId>  <!-- (1) -->
    </dependency>

    <dependency>
        <groupId>org.terasoluna.gfw</groupId>
        <artifactId>terasoluna-gfw-security-web</artifactId>  <!-- (2) -->
    </dependency>

.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | terasoluna-gfw-security-coreは、webに依存しないため、ドメイン層のプロジェクトから使用する場合は、
       | terasoluna-gfw-security-coreのみをdependencyに追加すること。
   * - | (2)
     - | terasoluan-gfw-webはwebに関連する機能を提供する。terasoluna-gfw-security-coreにも依存しているため、
       | Webプロジェクトは、terasoluna-gfw-security-webのみをdependencyに追加すること。

Web.xmlの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: xml
   :emphasize-lines: 5,13-20

    <context-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>  <!-- (1) -->
          classpath*:META-INF/spring/applicationContext.xml
          classpath*:META-INF/spring/spring-security.xml
      </param-value>
    </context-param>
    <listener>
      <listener-class>
        org.springframework.web.context.ContextLoaderListener
      </listener-class>
    </listener>
    <filter>
      <filter-name>springSecurityFilterChain</filter-name>  <!-- (2) -->
      <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>  <!-- (3) -->
    </filter>
    <filter-mapping>
      <filter-name>springSecurityFilterChain</filter-name>
      <url-pattern>/*</url-pattern>  <!-- (4) -->
    </filter-mapping>

.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | contextConfigLocationには、applicationContext.xmlに加えて、
       | クラスパスにSpring Security設定ファイルを追加する。本ガイドラインでは、「spring-security.xml」とする。
   * - | (2)
     - | filter-nameには、Spring Securityの内部で使用されるBean名、「springSecurityFilterChain」 で定義すること。
   * - | (3)
     - 各種機能を有効にするための、Spring Securityのフィルタ設定。
   * - | (4)
     - 全てのリクエストに対して設定を有効にする。

spring-security.xmlの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| web.xmlにおいて指定したパスに、spring-security.xmlを配置する。
| 通常はsrc/main/resources/META-INF/spring/spring-security.xmlに設定する。
| 以下の例は、雛形のみであるため、詳細な説明は、次章以降を参照されたい。

* spring-mvc.xml

  .. code-block:: xml

    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:sec="http://www.springframework.org/schema/security"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/security
            http://www.springframework.org/schema/security/spring-security.xsd
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd">
        <sec:http  use-expressions="true">  <!-- (1) -->
          <!-- omitted -->
        </sec:http>
    </beans>

  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - 項番
       - 説明
     * - | (1)
       - | use-expressions="true"と記載することで、アクセス属性のSpring EL式を有効することができる。
  \
      .. note::
          use-expressions="true" で有効になるSpring EL式は、以下を参照されたい。

          \ `Expression-Based Access Control <http://static.springsource.org/spring-security/site/docs/3.1.x/reference/el-access.html>`_\

