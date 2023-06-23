ガイドラインの観点別マッピング
================================================================================
ガイドラインの4章以降は機能別に説明されているが、機能面とは別の観点で、どの節にどの内容が含まれているかのマッピングを示す。

セキュリティ対策に関するマッピング
--------------------------------------------------------------------------------

OWASP(Open Web Application Security Project)による観点
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
\ `OWASP Top 10 for 2013 <https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project>`_\ を軸として、
セキュリティに関連する機能の説明へのリンクを記載する。


.. tabularcolumns:: |p{0.10\linewidth}|p{0.40\linewidth}|p{0.50\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 40 50

   * - 項番
     - 項目名
     - 関連するガイドライン
   * - A1
     - `Injection <https://www.owasp.org/index.php/Top_10_2013-A1-Injection>`_ SQL Injection
     - * \ :doc:`../ArchitectureInDetail/DataAccessJpa`\ 
       * \ :doc:`../ArchitectureInDetail/DataAccessMybatis2`\ 
       
       (クエリにパラメータを埋め込む場合は、バインド変数を使用する旨を記載)
   * - A1
     - `Injection <https://www.owasp.org/index.php/Top_10_2013-A1-Injection>`_ XXE(XML External Entity) Injection
     - * \ :doc:`../ArchitectureInDetail/Ajax`\ 
   * - A2
     - `Broken Authentication and Session Management <https://www.owasp.org/index.php/Top_10_2013-A2-Broken_Authentication_and_Session_Management>`_
     - * \ :doc:`../Security/Authentication`\ 
     
   * - A3
     - `Cross-Site Scripting (XSS) <https://www.owasp.org/index.php/Top_10_2013-A3-Cross-Site_Scripting_(XSS)>`_
     - * \ :doc:`../Security/XSS`\ 
   * - A4
     - `Insecure Direct Object References <https://www.owasp.org/index.php/Top_10_2013-A4-Insecure_Direct_Object_References>`_
     - 特に言及なし
   * - A5
     - `Security Misconfiguration <https://www.owasp.org/index.php/Top_10_2013-A5-Security_Misconfiguration>`_
     - * \ :doc:`../ArchitectureInDetail/Logging`\ (ログのメッセージ内容に言及)
       * \ :ref:`exception-handling-how-to-use-codingpoint-jsp-exceptioncode-label`\ (システム例外時に出力するメッセージ内容に言及)
   * - A6
     - `Sensitive Data Exposure <https://www.owasp.org/index.php/Top_10_2013-A6-Sensitive_Data_Exposure>`_
     - * \ :doc:`../ArchitectureInDetail/PropertyManagement`\ 
       * \ :doc:`../Security/PasswordHashing`\  (パスワードハッシュにのみ言及)
   * - A7
     - `Missing Function Level Access Control <https://www.owasp.org/index.php/Top_10_2013-A7-Missing_Function_Level_Access_Control>`_
     - * \ :doc:`../Security/Authorization`\ 
   * - A8
     - `Cross-Site Request Forgery (CSRF) <https://www.owasp.org/index.php/Top_10_2013-A8-Cross-Site_Request_Forgery_(CSRF)>`_
     - * \ :doc:`../Security/CSRF`\ 
   * - A9
     - `Using Components with Known Vulnerabilities <https://www.owasp.org/index.php/Top_10_2013-A9-Using_Components_with_Known_Vulnerabilities>`_
     - 特に言及なし
   * - A10
     - `Unvalidated Redirects and Forwards <https://www.owasp.org/index.php/Top_10_2013-A10-Unvalidated_Redirects_and_Forwards>`_
     - * \ :doc:`../Security/Authentication`\ (オープンリダイレクタ脆弱性対策について言及)

CVE(Common Vulnerabilities and Exposures)による観点
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
ガイドラインで言及しているCVEごとにその説明とガイドラインへのリンクを記載する。
ガイドラインで言及していないCVEについては、\ `Pivotal Product Vulnerability Reports <https://pivotal.io/security>`_\を参照されたい。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.40\linewidth}|p{0.50\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 40 50

   * - CVE
     - 概要
     - ガイドラインでの言及箇所
   * - \ `CVE-2014-0050 <https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0050>`_\

       \ `CVE-2016-3092 <https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-3092>`_\
     - Apache Commons FileUploadを使用するとファイルをアップロードする処理で細工されたリクエストによるDoS攻撃を受ける可能性がある

     - * :ref:`FileUploadOverview`

       * :ref:`file-upload_usage_commons_fileupload`
   * - \ `CVE-2014-1904 <https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-1904>`_\
     - \ ``<form:form>``\タグの \ ``action``\属性を省略するとXSS攻撃を受ける可能性がある
     - * :ref:`ApplicationLayerImplementOfJsp`
   * - \ `CVE-2015-3192 <https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-3192>`_\
     - DTDを使用したDoS攻撃が可能となる
     - * :ref:`ajax_how_to_use`

       * :ref:`RESTHowToUseApplicationSettings`
   * - \ `CVE-2016-5007 <https://pivotal.io/jp/security/cve-2016-5007>`_\
     - Spring SecurityとSpring MVCのパス比較方法の差異を利用して認可のすり抜けが可能となる
     - * :ref:`authorization-intercept-url`
   * - \ `CVE-2016-6652 <https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-6652>`_\
     - \ ``Sort``\オブジェクトをそのままJPAプロバイダに受け渡すとブラインドSQLインジェクション攻撃を受ける可能性がある
     - * :ref:`how_to_specify_query_annotation-label`
   * - \ `CVE-2016-9879 <https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-9879>`_\
     - Spring SecurityとSpring MVCのパス取得方法の差異を利用して認可のすり抜けが可能となる
     - * :ref:`authorization-intercept-url`

.. raw:: latex

   \newpage

