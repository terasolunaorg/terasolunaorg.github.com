ヘルスチェック
--------------------------------------------------------------------------------

.. only:: html

 .. contents:: 目次
    :depth: 4
    :local:

|

.. _HealthCheckOverview:

Overview
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| 本節では、ヘルスチェックについて説明する。

.. _HealthCheckOverview-Loadbalancer:


ロードバランサの負荷分散と縮退運転
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Webシステムが大量のユーザからリクエストを受けることを想定し、ロードバランサ（以下、LBと言う）を利用する。
| LBは複数のサーバにリクエストを割り振ることでWebシステムにかかる負荷を分散する装置であり、サーバの追加・削除により、柔軟にWebシステムの処理能力を変更できるのが特徴である。
| ヘルスチェックは、LBがリクエストを割り振る各サーバの稼動状況を監視する機能である。LBはヘルスチェックを利用し、異常を検知したサーバにリクエストを割り振らず、正常に稼動しているサーバに割り振る。これにより、特定のサーバで障害が発生した場合も、Webシステムを停止することなく運用することが可能である。（これを縮退運転と呼ぶ）

| 以下の例では、LBは3台のサーバを管理し、リクエストを割り振っている。

.. figure:: ./images/healthcheck-overview-flow.png
   :width: 100%

   **Picture - About Load Balancing**

| LBは定期的にサーバにリクエストを送信し、サーバから返されたステータスコードやレスポンスを確認することで、サーバの稼動状況を監視する。図のサーバAで異常が発生した場合、LBがそれを検知し、サーバAにリクエストを割り振らないようにする。
| 元々サーバAに接続していたクライアントAは、LBによって、他のサーバ(ここではサーバB)にリクエストを割り振られる。

.. figure:: ./images/healthcheck-overview-flow-failure.png
   :width: 100%

   **Picture - About Fallback**

ヘルスチェックの種類
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| LBが行うヘルスチェックには、さまざまな種類がある。以下に例を示す。

.. figure:: ./images/healthcheck-overview-healthcheckFlow.png
   :width: 100%

   **Picture - HealthCheck Example**

|

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 20 70

   * - 項番
     - ヘルスチェックの種類
     - 詳細
   * - | (1)
     - | PINGでのヘルスチェック
     - | OSI参照モデルのネットワーク層レベルで稼動状況を確認する。サーバ(OS)に対してPINGを送信し、応答があれば稼動していると判断する。
   * - | (2)
     - | TCP/UDPでのヘルスチェック
     - | OSI参照モデルのトランスポート層レベルで稼動状況を確認する。Web/APサーバのTCPポート（またはUDPポート）にリクエストを送信し、応答があれば稼動していると判断する。
   * - | (3)
     - | アプリケーションでのヘルスチェック
     - | OSI参照モデルのアプリケーション層レベルで稼動状況を確認する。Web/APサーバ上で稼動するアプリケーションにHTTPリクエストを送信し、応答が正常であれば稼動していると判断する。

| PINGやTCP/UDPでのヘルスチェックでは、アプリケーションの稼動状況までは確認できない。Webアプリケーションを対象とした場合は、サーバ(OS)やWeb/APサーバが稼動しているだけでは不十分であり、アプリケーションが稼動している必要がある。
| そのため本ガイドラインでは、アプリケーションでのヘルスチェックを行うことを推奨する。

.. _HealthCheckOverview-Implementation:

本ガイドラインで示すヘルスチェックの構成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 本ガイドラインでは、アプリケーションでのヘルスチェックを行うための、アプリケーションの実装例を紹介する。
| 具体的には、LBからのリクエストを受け取る以下の図ような構成のハンドラを実装する。

.. figure:: ./images/healthcheck-overview-function.png
   :width: 100%

   **Picture - HealthCheck Configuration**

|

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | LBからのリクエストを受け、Controller、Service、Repositoryを実行する。
       | 単に稼動状況を確認する、という点では、よりシンプルにヘルスチェックを実現する方法も存在する。しかし本ガイドラインでは、ヘルスチェックによってアプリケーションが使用している仕組みやフレームワーク自体が正しく動作していることも確認するべく、対象アプリの使用技術構成にできる限り近づけるために、Controller、Service、Repositoryを実装する。
   * - | (2)
     - | RepositoryからSQLを発行し、データベースが稼動していることを確認する。
       | これは、データベースアクセスを伴うアプリケーションの場合、アプリケーションが稼動していても、データベースに異常がある場合は正常に業務を行うことができないためである。
   * - | (3)
     - | レスポンスを返すViewとしてJSPを使用する。
       | 本ガイドラインではJSPを例にとって説明するが、RESTやSOAPを用いる場合など、アプリケーションの特性に合わせて通信方式やレスポンス形式は適宜変更すること。詳細は、\ :doc:`../../../ArchitectureInDetail/WebServiceDetail/REST`\や、\ :doc:`../../../ArchitectureInDetail/WebServiceDetail/SOAP`\を参照されたい。

| 本ガイドラインの実装例で返却されるステータスコードおよびレスポンスは以下の通りである。

.. tabularcolumns:: |p{0.25\linewidth}|p{0.30\linewidth}|p{0.30\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 25 30 30

   * - ヘルスチェック処理結果
     - ステータスコード
     - レスポンス内容
   * - | 成功
     - | 200(正常)
     - | \ ``OK.``\の3文字
   * - | エラー発生
     - | 例外ハンドリング機能で設定されたステータスコード
     - | 例外ハンドリング機能で設定されたレスポンス

| 例外ハンドリングの設定を変更する場合は、\ :doc:`../../../ArchitectureInDetail/WebApplicationDetail/ExceptionHandling`\を参照されたい。


.. _HealthCheckHowToUse:

How to use
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| \ :ref:`HealthCheckOverview-Implementation`\で示した実装例について説明する。

.. _HealthCheckHowToUseRepository:

Repositoryインタフェース
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| まず、\ ``HealthCheckRepository``\を作成する。\ ``HealthCheckRepository``\はヘルスチェック用のSQLを実行し、データベースの稼動を確認する
| なお、ここではMyBatis3を用いてデータベースにアクセスする例を示す。他の方式を採用する場合は\ :doc:`../../../ArchitectureInDetail/DataAccessDetail/index`\を参照されたい。

**HealthCheckRepository.java**

.. code-block:: java

   package com.example.domain.repository.healthcheck;
   
   public interface HealthCheckRepository {
      void healthcheck();
   }

| ここでは、データベースへのアクセスが正しく行えていることさえ確認できればよいので、必要最小限のSQLを設定する。
| 本ガイドラインでは、SQLは以下の条件を満たすように設定している。

* 参照系であること
* パラメータが不要であること

| 以下は、PostgreSQLを使用した場合のマッピングファイルの例である。

**HealthCheckRepository.xml(PostgreSQLを使用した場合)**

.. code-block:: xml

   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

   <mapper namespace="com.example.domain.repository.healthcheck.HealthCheckRepository">

      <select id="healthcheck" resultType="String">
         SELECT '1'
      </select>

   </mapper>

| また、以下は、Oracleを使用した場合のマッピングファイルの例である。

**HealthCheckRepository.xml(Oracleを使用した場合)**

.. code-block:: xml

   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

   <mapper namespace="com.example.domain.repository.healthcheck.HealthCheckRepository">

      <select id="healthcheck" resultType="String">
         SELECT '1' FROM DUAL
      </select>

   </mapper>

.. _HealthCheckHowToUseService:

Serviceクラス
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 次に、\ ``HealthCheckService``\インタフェースと、\ ``HealthCheckService``\インタフェースを実装した\ ``HealthCheckServiceImpl``\クラスを作成する。
| \ ``HealthCheckServiceImpl``\は、\ ``healthcheckRepository``\の\ ``healthcheck``\メソッドを呼び出し、データベースのヘルスチェックを行う。

**HealthCheckService.java**

.. code-block:: java

   package com.example.domain.service.healthcheck;

   public interface HealthCheckService {
      void healthcheck();
   }

**HealthCheckServiceImpl.java**

.. code-block:: java

   package com.example.domain.service.healthcheck;

   import healthcheck.domain.repository.healthcheck.HealthCheckRepository;
   
   import javax.inject.Inject;

   import org.springframework.stereotype.Service;
   import org.springframework.transaction.annotation.Transactional;

   @Service
   @Transactional(readOnly = true)
   public class HealthCheckServiceImpl implements HealthCheckService {
   
      @Inject
      HealthCheckRepository healthcheckRepository;
      
      @Override
      public void healthcheck() {
         healthcheckRepository.healthcheck();
      }
   }

.. _HealthCheckHowToUseController:

Controllerクラス
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 次に、\ ``HealthCheckController``\を作成する。
| \ ``HealthcheckService``\の\ ``healthcheck``\メソッドを呼び出し、実行結果によって指定されたパスに遷移する。データベースの稼動が確認できた場合は、\ ``OK.``\を表示するためのビューを返す。

**HealthCheckController.java**

.. code-block:: java
   
   package com.example.app.healthcheck;

   import healthcheck.domain.service.healthcheck.HealthCheckService;

   import javax.inject.Inject;

   import org.springframework.stereotype.Controller;
   import org.springframework.web.bind.annotation.RequestMapping;

   @Controller
   public class HealthCheckController {
   
      @Inject
      HealthCheckService healthcheckService;

      @RequestMapping(value = "healthcheck") // (1)
      public String healthcheck(){
         healthcheckService.healthcheck();
         return "common/healthcheck/ok"; // (2)
      }
   }

|

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``value``\属性は、稼動状態を調べるためのヘルスチェック用のURLとなる。
    * - | (2)
      - | Apache Tilesの設定を受けないようにするため、JSPファイルを配置するディレクトリを1階層深くしている。詳細は、\ :ref:`HealthCheckAppendixAppatchTiles`\を参照されたい。

.. _HealthCheckHowToUseJsp:

JSPファイル
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 最後に、ヘルスチェック成功時に遷移するJSPファイルを作成する。
| JSPファイル作成時、以下に示すように\ ``<%@page>``\ディレクティブと\ ``OK.``\の間に改行文字などを挟まないようにする。
| これは、レスポンスのデータ量を最低限にするためである。

**ok.jsp**

.. code-block:: jsp

   <%@page contentType="text/plain; charset=utf-8" language="java" pageEncoding="utf-8" %>OK.

| レスポンスのデータ量を最低限にするにあたり、他にも注意するべき点が存在する。詳細は\ :ref:`HealthCheckAppendixMinResponce`\を参照されたい。

.. _HealthCheckHowToUseSecurity:

アクセス権の設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| ヘルスチェック処理を使用する際は、認証・認可機能などによりヘルスチェック用のURLがアクセス不可にならないように注意する必要がある。
| 例えば、どのロールでもアクセスできるようにするには、spring-security.xmlの\ ``<sec:intercept-url>``\を設定する。
| \ ``/common/healthcheck``\ 配下の除外設定を行う例を以下に示す。
| 詳細は\ :doc:`../../../Security/Authorization`\を参照されたい。

**spring-security.xml**

.. code-block:: xml

   <sec:http>
      <sec:intercept-url pattern="/healthcheck/**" access="permitAll"/>
      <!-- omitted -->
   </sec:http>

.. note::

   認可制御を外すと、誰でもヘルスチェック用URLにアクセスできるようになってしまうので、
   外部からアクセスされたくない場合はLBなどで防ぐ対処が必要である。

.. _HealthCheckAppendix:

Appendix
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _HealthCheckAppendixMinResponce:

レスポンスのデータ量を最低限にする設定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| \ :ref:`HealthCheckHowToUseJsp`\で示した通り、レスポンスのデータ量を最低限にするにあたり、主に以下の点に注意が必要である。

* \ :ref:`Apache Tilesの設定を受けないようにする<HealthCheckAppendixAppatchTiles>`\
* \ :ref:`ヘッダファイルやフッタファイルを読み込まないようにする<HealthCheckAppendixHeader>`\
* \ :ref:`レスポンスから余計な改行を削除する<HealthCheckAppendixTrimWhitespace>`\

上記について、それぞれ実装例を以下に示す。

.. _HealthCheckAppendixAppatchTiles:

Apache Tilesの設定を受けないようにする
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| \ :ref:`HealthCheckHowToUseController`\で示した通り、Apache Tilesの設定を受けないよう、tiles-definitions.xmlの\ ``<put-attribute>``\タグに従わないディレクトリ配下にJSPファイルを配置する必要がある。
| ブランクプロジェクトのデフォルトの設定では、 \ ``/WEB-INF/views/{1}/{2}.jsp``\に該当するJSPにApache Tilesが適用される設定となっているため、1階層深いディレクトリを作成し、\ ``/WEB-INF/views/common/healthcheck/``\配下にJSPファイルを配置している。 
| 詳細は\ :doc:`../../../ArchitectureInDetail/WebApplicationDetail/TilesLayout`\を参照されたい。

.. _HealthCheckAppendixHeader:

ヘッダファイルやフッタファイルを読み込まないようにする
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| JSPファイルはweb.xmlの\ ``<jsp-config>``\タグの影響を受けることに注意が必要である。
| \ ``<jsp-config>``\タグ内で\ ``<include-prelude>``\タグや\ ``<include-coda>``\タグを設定した場合、
| JSPファイルにヘッダファイルやフッタファイルが読み込まれてしまう。ヘッダファイルやフッタファイルを読み込む設定となっていないか注意すること。
| 設定の例を以下に示す。

**web.xml**

.. code-block:: xml

    <jsp-config> 
        <jsp-property-group>
            <url-pattern>/WEB-INF/views/common/healthcheck/ok.jsp</url-pattern>
            <el-ignored>false</el-ignored>
            <page-encoding>UTF-8</page-encoding>
            <scripting-invalid>false</scripting-invalid>
            // (1)
        </jsp-property-group> 
    </jsp-config>

|

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 余計なヘッダファイルやフッタファイルを読み込まないために、\ ``<include-prelude>``\タグや\ ``<include-coda>``\タグは設定しない。

.. _HealthCheckAppendixTrimWhitespace:

レスポンスから余計な改行を削除する
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| 例えば、タグライブラリを使用するために、JSPの先頭に\ ``<%@taglib>``\ディレクティブを設定した場合、レスポンスの先頭に余計な改行が出力されてしまう。
| そのため、\ ``<%@page>``\ディレクティブに\ ``trimDirectiveWhitespaces``\属性を設定し、ok.jspに余計な改行を出力しないようにする。

**ok.jsp(trimDirectiveWhitespaces属性を設定する場合)**

.. code-block:: jsp

   <%@page contentType="text/plain; charset=utf-8" language="java" pageEncoding="utf-8" trimDirectiveWhitespaces="true" %>OK.

.. note::

   ブランクプロジェクトのデフォルトの設定では、web.xmlの\ ``<include-prelude>``\に\ ``/WEB-INF/views/common/include.jsp``\が設定されている。
   そのため、上記に示した設定をするか、もしくは、ok.jspを含まないように\ ``<url-pattern>``\を修正する必要がある。
   そうしないと、include.jspがすべてのJSPファイルに読み込まれok.jspには改行が出力されてしまう。

.. warning::

   WebLogicを使用した場合、上記に示した\ ``trimDirectiveWhitespaces``\属性を設定しても、\ ``<%@page>``\ディレクティブよりも前に余計な文字があった場合に改行が削除されない。
   そのため、別の方法で対応する必要がある。
   例として、web.xmlの\ ``<jsp-property-group>``\タグに、\ ``<trim-directive-whitespaces>``\タグを設定する対応法を以下に示す。
   
   **web.xml(<trim-directive-whitespaces>タグを設定する場合)**
   
    .. code-block:: xml

        <jsp-config> 
            <jsp-property-group>
                <url-pattern>/WEB-INF/views/common/healthcheck/ok.jsp</url-pattern> // (1)
                <el-ignored>false</el-ignored>
                <page-encoding>UTF-8</page-encoding>
                <scripting-invalid>false</scripting-invalid>
                <trim-directive-whitespaces>true</trim-directive-whitespaces> // (2)
            </jsp-property-group>
        </jsp-config>

    |

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90
        :class: longtable

        * - 項番
          - 説明
        * - | (1)
          - | ok.jspのみに\ ``<trim-directive-whitespaces>``\タグを適用するため、\ ``<url-pattern>``\タグでok.jspのみを指定する。
        * - | (2)
          - | \ ``<trim-directive-whitespaces>``\タグに\ ``true``\を設定することで、対象のJSPファイル(ok.jsp)から余計な改行を削除する。

.. raw:: latex

   \newpage

