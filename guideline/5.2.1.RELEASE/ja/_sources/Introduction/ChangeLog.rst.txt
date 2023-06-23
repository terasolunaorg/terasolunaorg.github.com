更新履歴
================================================================================

.. tabularcolumns:: |p{0.15\linewidth}|p{0.25\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 15 25 60

    * - 更新日付
      - 更新箇所
      - 更新内容

    * - 2017-03-17
      - \-
      - 5.2.1 RELEASE版公開

        * 更新内容の詳細は、\ `5.2.1のIssue一覧 <https://github.com/terasolunaorg/guideline/issues?utf8=%E2%9C%93&q=label%3A5.2.1%20is%3Aissue%20is%3Aclosed%20>`_\ を参照されたい。

    * -
      - 全般
      - ガイドラインの誤記(タイプミスや単純な記述ミスなど)の修正

        * 修正内容の詳細は、\ `5.2.1のIssue一覧(clerical error) <https://github.com/terasolunaorg/guideline/issues?utf8=%E2%9C%93&q=%20label%3A5.2.1%20is%3Aclosed%20label%3A%22clerical%20error%22%20>`_\ を参照されたい。

        記載内容の改善

        * 改善内容の詳細は、\ `5.2.1のIssue一覧(improvement) <https://github.com/terasolunaorg/guideline/issues?utf8=%E2%9C%93&q=label%3A5.2.1%20label%3Aimprovement%20is%3Aclosed%20>`_\ を参照されたい。

        以下のプロジェクト作成時に使用する \ ``maven-archetype-plugin``\のバージョンを2.4に固定するように変更(\ `guideline#2523 <https://github.com/terasolunaorg/guideline/issues/2523>`_\ )

        * :doc:`../Overview/FirstApplication`
        * :doc:`../ImplementationAtEachLayer/CreateWebApplicationProject`
        * :doc:`../Tutorial/TutorialTodo`
        * :doc:`../Tutorial/TutorialSecurity`

    * -
      - :doc:`../Introduction/CriteriaBasedMapping`
      - 記載内容の追加

        * セキュリティ対策に関するマッピングにCVEによる観点をまとめた表を追加(\ `guideline#2439 <https://github.com/terasolunaorg/guideline/issues/2439>`_\ )

    * - 
      - :doc:`../ImplementationAtEachLayer/DomainLayer`
      - 記載内容の改善

        * シグネチャを制限するインタフェースおよび基底クラスの実装サンプルを修正(\ `guideline#2219 <https://github.com/terasolunaorg/guideline/issues/2219>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Validation`
      - 記載内容の改善

        * メッセージに入力チェック対象を含める方法の説明を追加(\ `guideline#2002 <https://github.com/terasolunaorg/guideline/issues/2002>`_\ )
        * @URLによる入力チェックのチェック内容に関する記述を修正(\ `guideline#2260 <https://github.com/terasolunaorg/guideline/issues/2260>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/SessionManagement`  
      - 記載内容の追加  

        * セッションスコープに格納しているオブジェクトを受け取る際にリクエストパラメータのバインドを防止する方法について追記(\ `guideline#1293 <https://github.com/terasolunaorg/guideline/issues/1293>`_\ )  

    * - 
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Internationalization`
      - 記載内容の追加

        * 国際化が適用されない場合の例とその対策方法を追加(\ `guideline#2427 <https://github.com/terasolunaorg/guideline/issues/2427>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessMyBatis3`
      - 記載内容の追加

        * コミット時にエラーが発生した場合にロールバック処理を呼び出すための設定に関する記載を追加(\ `guideline#2375 <https://github.com/terasolunaorg/guideline/issues/2375>`_\ )
    * -
      - :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessJpa`
      - 記載内容の追加

        * \ `CVE-2016-6652 <https://pivotal.io/security/cve-2016-6652>`_\ (ブラインドSQLインジェクション攻撃を受ける可能性がある脆弱性)に関する注意喚起と対策を追加(\ `guideline#2439 <https://github.com/terasolunaorg/guideline/issues/2439>`_\ )

    * - 
      - :doc:`../ArchitectureInDetail/MessagingDetail/Email`
      - 記載内容の追加

        * JavaMailで発生する問題とその回避方法を追加(\ `guideline#2190 <https://github.com/terasolunaorg/guideline/issues/2190>`_\ )

    * -
      - :doc:`../Security/Authentication`
      - 記載内容の追加

        * Remember Me認証に使用するチェックボックスのvalue属性値について追記(\ `guideline#785 <https://github.com/terasolunaorg/guideline/issues/785>`_\ )

        記載内容の修正

        * SecureRandomの使用についての記載を修正(\ `guideline#2177 <https://github.com/terasolunaorg/guideline/issues/2177>`_\ )

    * -  
      - :doc:`../Security/Authorization`  
      - 記載内容の追加  
  
        * 特定URLに対するアクセス制限に関するWarningを追記(\ `guideline#2399 <https://github.com/terasolunaorg/guideline/issues/2399>`_\ )  

        * \ `CVE-2016-9879 <https://pivotal.io/security/cve-2016-9879>`_\ (認可のすり抜けが可能となる脆弱性)に関する注意喚起と対策を追加(\ `guideline#2439 <https://github.com/terasolunaorg/guideline/issues/2439>`_\ )

    * - 2016-08-31
      - \-
      - 5.2.0 RELEASE版公開

        * 更新内容の詳細は、\ `5.2.0のIssue一覧 <https://github.com/terasolunaorg/guideline/issues?utf8=%E2%9C%93&q=label%3A5.2.0%20is%3Aissue%20is%3Aclosed%20>`_\ を参照されたい。

    * -
      - 全般
      - ガイドラインの誤記(タイプミスや単純な記述ミスなど)の修正

        * 修正内容の詳細は、\ `5.2.0のIssue一覧(clerical error) <https://github.com/terasolunaorg/guideline/issues?utf8=%E2%9C%93&q=%20label%3A5.2.0%20is%3Aclosed%20label%3A%22clerical%20error%22%20>`_\ を参照されたい。

        記載内容の改善

        * 改善内容の詳細は、\ `5.2.0のIssue一覧(improvement) <https://github.com/terasolunaorg/guideline/issues?utf8=%E2%9C%93&q=label%3A5.2.0%20label%3Aimprovement%20is%3Aclosed%20>`_\ を参照されたい。

        章立てを全面見直し

        * 更新内容の詳細は、\ `Optimize the order of chapters and sections #1683 <https://github.com/terasolunaorg/guideline/issues/1683>`_\ を参照されたい。

        共通ライブラリのバージョンを5.2.0.RELEASEに更新

        * 更新内容の詳細は、\ `Check Version  #2076 <https://github.com/terasolunaorg/guideline/issues/2076>`_\ を参照されたい。

        記載内容の改善

        * 共通ライブラリのpomの依存性について追加 (\ `guideline#1982 <https://github.com/terasolunaorg/guideline/issues/1982>`_\ )

    * -
      - :doc:`../Overview/FrameworkStack`
      - 記載内容の追加

        * ブランクプロジェクトの共通ライブラリ標準の組込状況を追加(\ `guideline#1700 <https://github.com/terasolunaorg/guideline/issues/1700>`_\ )
        * mybatis-typehandlers-jsr310 、jackson-datatype-jsr310をOSSスタックに追加 (\ `guideline#1966 <https://github.com/terasolunaorg/guideline/issues/1966>`_\ )
        * spring-jmsおよびその依存ライブラリをOSSスタックに追加 (\ `guideline#1992 <https://github.com/terasolunaorg/guideline/issues/1992>`_\ )

        利用するOSSのバージョン(Spring IO Platformのバージョン)を更新

        * Spring IO Platformのバージョンを2.0.6.RELEASEに更新
        * Spring Frameworkのバージョンを4.2.7.RELEASEに更新
        * Spring Securityのバージョンを4.0.4.RELEASEに更新

        Spring IO Platformのバージョン更新に伴い利用するOSSのバージョンを更新

    * -
      - :doc:`../ImplementationAtEachLayer/DomainLayer`
      - 記載内容の追加

        * MyBatis 3.3 + MyBatis-Spring 1.2 において、 @Transactinal  の  timeout  属性に指定した値は使用されない旨を追加(\ `guideline#1777 <https://github.com/terasolunaorg/guideline/issues/1777>`_\ )

    * -
      - :doc:`../ImplementationAtEachLayer/ApplicationLayer`
      - 記載内容の追加

        * HttpSessionをハンドラメソッドの引数として使用すべきでない旨を追加(\ `guideline#1313 <https://github.com/terasolunaorg/guideline/issues/1313>`_\ )
        * JSR-310 Date and Time APIを使用する際の注意点を記載 (\ `guideline#1991 <https://github.com/terasolunaorg/guideline/issues/1991>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Validation`
      - 記載内容の改善

        * メッセージプロパティファイルをNative to Asciiせずに直接扱う方法を追加(\ `guideline#994 <https://github.com/terasolunaorg/guideline/issues/994>`_\ )
        * cross-field validationについて追加(\ `guideline#1561 <https://github.com/terasolunaorg/guideline/issues/1561>`_\ )
        * @DateTimeFormat  の説明を追加(\ `guideline#1873 <https://github.com/terasolunaorg/guideline/issues/1873>`_\ )
        * ValidationMessages.propertiesについての説明を修正 (\ `guideline#1948 <https://github.com/terasolunaorg/guideline/issues/1948>`_\ )
        * Method Validationを利用した入力チェックの注意事項を追加(\ `guideline#1998 <https://github.com/terasolunaorg/guideline/issues/1998>`_\ )

        記載内容の追加

        * OSコマンドインジェクションに関する記載を追加 (\ `guideline#1957 <https://github.com/terasolunaorg/guideline/issues/1957>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/ExceptionHandling`
      - Spring Framework 4.2.7対応に伴う修正
      
        * HTTPレスポンスヘッダー出力に関する説明内容を修正(\ `guideline#1965 <https://github.com/terasolunaorg/guideline/issues/1965>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/DoubleSubmitProtection`
      - 記載内容の追加
      
        * \ ``@TransactionTokenCheck``\アノテーションのtype属性に新たに追加された \ ``TransactionTokenType.CHECK``\の仕様、利用方法に関する記載の追加 
          (\ `guideline#2071 <https://github.com/terasolunaorg/guideline/issues/2071>`_\ )

        「How To Extend プログラマティックにトランザクショントークンのライフサイクルを管理する方法について」を削除。

        * \ ``TransactionTokenContext``\が提供していたアプリケーション向けAPIを使用した場合、
          \ ``TransactionToken``\を正しい状態に維持できなくなるなど、フレームワーク内部の挙動に影響を及ぼすような作り込みができてしまうことから、
          当該APIの非推奨化がなされた。非推奨化にあわせて該当機能の利用方法の記述を削除した。 

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Internationalization`
      - 記載内容の改善

        *   リクエストパラメータ(デフォルトのパラメータ名)の説明の位置を修正(\ `guideline#1354 <https://github.com/terasolunaorg/guideline/issues/1354>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/FileUpload`
      - 記載内容の追加

        * \ `CVE-2016-3092 <https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-3092>`_\ (File Uploadの脆弱性)に関する注意喚起を追加(\ `guideline#1973 <https://github.com/terasolunaorg/guideline/issues/1973>`_\ )
        * ディレクトリトラバーサル攻撃に関する記載を追加 (\ `guideline#2010 <https://github.com/terasolunaorg/guideline/issues/2010>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/HealthCheck`
      - 新規追加

        * ヘルスチェックを追加(\ `guideline#1698 <https://github.com/terasolunaorg/guideline/issues/1698>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebServiceDetail/REST`
      - 記載内容の変更、追加

        * JSR-310 Date and Time API / Joda Timeを使う場合の設定の記述を変更 (\ `guideline#1966 <https://github.com/terasolunaorg/guideline/issues/1966>`_\ )
        * JacksonをJava SE 7環境で使用する場合の注意点を記載 (\ `guideline#1966 <https://github.com/terasolunaorg/guideline/issues/1966>`_\ )
        * JSONでJSR-310 Date and Time APIを使う場合の設定を記載 (\ `guideline#1966 <https://github.com/terasolunaorg/guideline/issues/1966>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebServiceDetail/RestClient`
      - 記載内容の改善

        * RestClientにおけるHTTP Proxyサーバの設定を追加(\ `guideline#1856 <https://github.com/terasolunaorg/guideline/issues/1856>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebServiceDetail/SOAP`
      - 記載内容の追加

        * SOAPクライアント起動時にSOAPサーバに接続しないオプションを追加(\ `guideline#1871 <https://github.com/terasolunaorg/guideline/issues/1871>`_\ )
        * SOAPクライアントのenvプロジェクトに関する説明の修正(\ `guideline#1901 <https://github.com/terasolunaorg/guideline/issues/1901>`_\ )
        * SOAP Webサービス例外発生時のステータスコード取得方法について追加(\ `guideline#2007 <https://github.com/terasolunaorg/guideline/issues/2007>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessMyBatis3`
      - 記載内容の追加

        * 暫定的なWARNログ出力回避方法を削除(\ `guideline#1292 <https://github.com/terasolunaorg/guideline/issues/1292>`_\ )
        * JSR-310 Date and Time APIをMybatis3.3で使用するための設定方法を記載 (\ `guideline#1966 <https://github.com/terasolunaorg/guideline/issues/1966>`_\ )
        * MyBatisをJava SE 7環境で使用する場合の注意点を記載 (\ `guideline#1966 <https://github.com/terasolunaorg/guideline/issues/1966>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/DataAccessDetail/ExclusionControl`
      - 記載内容の追加

        *  ExclusionControlにwarningメッセージを追加(\ `guideline#1694 <https://github.com/terasolunaorg/guideline/issues/1694>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/GeneralFuncDetail/Logging`
      - 記載内容の追加
        
        * ID付きログメッセージを出力するための拡張方法を記載 (\ `guideline#1928 <https://github.com/terasolunaorg/guideline/issues/1928>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/GeneralFuncDetail/StringProcessing`
      - 記載内容の追加

        * terasoluna-gfw-stringをdependencyに追加する例を追加(\ `guideline#1699 <https://github.com/terasolunaorg/guideline/issues/1699>`_\ )
        * @Size アノテーションの説明にサロゲートペアについての注意を追加(\ `guideline#1874 <https://github.com/terasolunaorg/guideline/issues/1874>`_\ )
        * JIS漢字\ ``U+2014``\(EM DASH)のUCS(Unicode)文字対応について記載を追加(\ `guideline#1914 <https://github.com/terasolunaorg/guideline/issues/1914>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/GeneralFuncDetail/Dozer`
      - 記載内容の追加

        * JSR-310 Date and Time APIを使用する際の注意点を記載 (\ `guideline#1966 <https://github.com/terasolunaorg/guideline/issues/1966>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/MessagingDetail/JMS`
      - 新規追加

        * JMSを追加(\ `guideline#1407 <https://github.com/terasolunaorg/guideline/issues/1407>`_\ )

    * -
      - :doc:`../Security/Authentication`
      - Spring Security 4.0.4対応に伴う修正

        * Spring 4.0.4にて authentication-failure-url の仕様が改善されたことによるコード例の修正とNoteの削除 (\ `guideline#1963 <https://github.com/terasolunaorg/guideline/issues/1963>`_\ )

    * -
      - :doc:`../Security/Authorization`
      - 記載内容の追加

        * \ `CVE-2016-5007 Spring Security / MVC Path Matching Inconsistency <https://pivotal.io/security/cve-2016-5007>`_\ の対応方法を追加 (\ `guideline#1976 <https://github.com/terasolunaorg/guideline/issues/1976>`_\ )

    * -
      - :doc:`../Security/SecureLoginDemo`
      - 記載内容の追加

        * 「セキュリティ観点での入力値チェック」を追加
        * 「監査ログ出力」を追加

    * -
      - :doc:`../Appendix/ReferenceBooks`
      - 記載内容の追加

        * 「Spring徹底入門」を参考書籍として追加(\ `guideline#2043 <https://github.com/terasolunaorg/guideline/issues/2043>`_\ )

    * - 2016-02-24
      - \-
      - 5.1.0 RELEASE版公開

        * 更新内容の詳細は、\ `5.1.0のIssue一覧 <https://github.com/terasolunaorg/guideline/issues?q=is%3Aissue+milestone%3A5.1.0+is%3Aclosed>`_\ を参照されたい。
    * -
      - 全般
      - ガイドラインの誤記(タイプミスや単純な記述ミスなど)の修正

        記載内容の改善

        * 改善内容の詳細は、\ `5.1.0のIssue一覧(improvement) <https://github.com/terasolunaorg/guideline/issues?q=milestone%3A5.1.0+label%3Aimprovement+is%3Aclosed>`_\ を参照されたい。

    * -
      - :doc:`index`
      - 記載内容の追加

        * ガイドラインに記載している内容の動作検証環境に関する記載を追加

    * -
      - :doc:`../Overview/FrameworkStack`
      - 利用するOSSのバージョン(Spring IO Platformのバージョン)を更新

        * Spring IO Platformのバージョンを2.0.1.RELEASEに更新
        * Spring Frameworkのバージョンを4.2.4.RELEASEに更新
        * Spring Securityのバージョンを4.0.3.RELEASEに更新

        Spring IO Platformのバージョン更新に伴い利用するOSSのバージョンを更新

        * 使用するOSSのバージョンを更新。更新内容は、\ `version 5.1.0の移行ガイド <https://github.com/terasolunaorg/terasoluna-gfw/wiki/Migration-Guide-5.1.0_ja#step-1-update-dependency-libraries>`_\ を参照されたい。

        新規プロジェクト追加

        * \ ``terasoluna-gfw-string``\ 、\ ``terasoluna-gfw-codepoints``\ 、\ ``terasoluna-gfw-validator``\ 、\ ``terasoluna-gfw-web-jsp``\ プロジェクトの説明を追加。

        共通ライブラリの新機能追加

        \ ``terasoluna-gfw-string``\ 
         * 半角全角変換

        \ ``terasoluna-gfw-codepoints``\
         * コードポイントチェック
         * コードポイントチェック用Bean Validation制約アノテーション

        \ ``terasoluna-gfw-validator``\
         * バイト長チェック用Bean Validation制約アノテーション
         * フィールド値比較相関チェック用Bean Validation制約アノテーション

    * -
      - :doc:`../Overview/FirstApplication`
      - 記述内容の改善

        *  Spring Security 4 対応に伴うサンプルソースの修正 (\ `guideline#1519 <https://github.com/terasolunaorg/guideline/issues/1519>`_\ )

         * \ ``AuthenticationPrincipalArgumentResolver``\のパッケージ変更

    * -
      - :doc:`../Tutorial/TutorialTodo`
      - Spring Security 4 対応に伴う修正

        *  Spring Security 4 対応に伴うソースの修正 (\ `guideline#1519 <https://github.com/terasolunaorg/guideline/issues/1519>`_\ )

         * \ ``AuthenticationPrincipalArgumentResolver``\のパッケージ変更
         *  デフォルトでtrueになる仕様のため、サンプルソースから\ ``<use-expressions="true">``\を削除

    * -
      - :doc:`../ImplementationAtEachLayer/CreateWebApplicationProject`
      - 記述内容の改善

        *  オフライン環境上でmvnコマンドを利用する方法を追加(\ `guideline#1197 <https://github.com/terasolunaorg/guideline/issues/1197>`_\ )

    * -
      - :doc:`../ImplementationAtEachLayer/ApplicationLayer`
      - 記述内容の改善

        *  EL関数を用いたリクエストURL作成方法について追加(\ `guideline#632 <https://github.com/terasolunaorg/guideline/issues/632>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessCommon`
      - 記載内容の追加

        *  \ ``Log4jdbcProxyDataSource``\のオーバヘッドに対する注意点を追加(\ `guideline#1471 <https://github.com/terasolunaorg/guideline/issues/1471>`_\ )
    * -
      - :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessMyBatis3`
      - MyBatis 3.3 対応に伴う記載内容の追加

        *  \ ``defaultFetchSize``\の設定方法を追加(\ `guideline#965 <https://github.com/terasolunaorg/guideline/issues/965>`_\ )
        * 遅延読み込み時のデフォルトが \ ``JAVASSIST``\に変更されている点を追加(\ `guideline#1384 <https://github.com/terasolunaorg/guideline/issues/1384>`_\ )
        * \ ``ResultHandler``\にGenricsを付与したサンプルコードに修正(\ `guideline#1384 <https://github.com/terasolunaorg/guideline/issues/1384>`_\ )
        * 新規追加された\ ``@Flush``\アノテーションを利用したソース例、及び注意点を追加(\ `guideline#915 <https://github.com/terasolunaorg/guideline/issues/915>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessJpa`
      - ガイドラインのバグ修正

        *  Like条件を使用するユーティリティを適切に修正(\ `guideline#1464 <https://github.com/terasolunaorg/guideline/issues/1464>`_\ )
        *  JPQLにおける真偽値の不適切な実装を修正(\ `guideline#1525 <https://github.com/terasolunaorg/guideline/issues/1525>`_\ )
        *  ページネーションの不適切な実装を修正(\ `guideline#1463 <https://github.com/terasolunaorg/guideline/issues/1463>`_\ )
        *  \ ``DateTimeProvider``\を実装したサンプルコードの不適切な実装を修正(\ `guideline#1327 <https://github.com/terasolunaorg/guideline/issues/1327>`_\ )
        *  共通Repositoryインタフェースの実装クラスのインスタンスを生成するためのFactoryクラスにおいて不適切な実装を修正(\ `guideline#1327 <https://github.com/terasolunaorg/guideline/issues/1327>`_\ )

        記載内容の改善

        *  \ ``hibernate.hbm2ddl.auto``\のデフォルト値を修正(\ `guideline#1282 <https://github.com/terasolunaorg/guideline/issues/1282>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Validation`
      - 記述内容の改善

        *  MethodValidationに対する記述を追加(\ `guideline#708 <https://github.com/terasolunaorg/guideline/issues/708>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/GeneralFuncDetail/Logging`
      - 記述内容の改善

        * Logbackの設定に\ ``ServiceLoader``\の仕組みを利用した記述の追加(\ `guideline#1275 <https://github.com/terasolunaorg/guideline/issues/1275>`_\ )
        * Spring Security 4 対応に伴うサンプルソースの修正 (\ `guideline#1519 <https://github.com/terasolunaorg/guideline/issues/1519>`_\ )

         * デフォルトでtrueになる仕様のため、サンプルソースから\ ``<use-expressions="true">``\を削除

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/SessionManagement`
      - 記述内容の改善

        *  SpEL式を用いたセッションスコープ参照の記述を追加(\ `guideline#1306 <https://github.com/terasolunaorg/guideline/issues/1306>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Internationalization`
      - 記述内容の改善

        *  JSPに適切にロケールを反映させるための記述を追加(\ `guideline#1439 <https://github.com/terasolunaorg/guideline/issues/1439>`_\ )
        *  \ ``SessionLocalResolver``\の\ ``defaultLocale``\の説明を修正(\ `guideline#686 <https://github.com/terasolunaorg/guideline/issues/686>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Codelist`
      - 記載内容の追加

        *  JdbcCodeListに\ ``JdbcTemplate``\を指定するパターンを推奨とする記述を追加(\ `guideline#501 <https://github.com/terasolunaorg/guideline/issues/501>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebServiceDetail/REST`
      - 記述内容の改善

        *  \ ``Jackson2ObjectMapperFactoryBean``\を利用したObjectMapper作成を追加(\ `guideline#1022 <https://github.com/terasolunaorg/guideline/issues/1022>`_\ )
        *  REST APIアプリケーションのドメイン層の実装にMyBatis3を前提とした形に修正 (\ `guideline#1323 <https://github.com/terasolunaorg/guideline/issues/1323>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebServiceDetail/RestClient`
      - 新規追加

        *  RESTクライアント（HTTPクライアント）を追加(\ `guideline#1307 <https://github.com/terasolunaorg/guideline/issues/1307>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebServiceDetail/SOAP`
      - 新規追加

        *  SOAP Web Service（サーバ/クライアント）を追加(\ `guideline#1340 <https://github.com/terasolunaorg/guideline/issues/1340>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/FileUpload`
      - 記述内容の改善

        * アップロード処理の基本フロー、及びその説明をSpringの\ ``MultipartFilter``\を用いた記述に修正 (\ `guideline#193 <https://github.com/terasolunaorg/guideline/issues/193>`_\ )
        * セキュリティ上の問題や、APサーバによって動作が異なる等の課題があるため、「クエリパラメータでCSRFトークンを送る方法」を削除。
          ファイルアップロードの許容サイズを超過した場合、一部APサーバでCSRFトークンチェックが正しく行われない注意点を追加(\ `guideline#1602 <https://github.com/terasolunaorg/guideline/issues/1602>`_\ )


    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/FileDownload`
      - Spring Framework4.2 対応に伴う記載内容の追加

        *  xlsx形式を操作する\ ``AbstractXlsxView``\の追加\(\ `guideline#996 <https://github.com/terasolunaorg/guideline/issues/996>`_\ )

        記述内容の改善

        * iTextの仕様変更のため、\ ``com.lowagie:itext:4.2.1``\を利用したソース例を\ ``com.lowagie:itext:2.1.7``\を利用する形に修正

    * -
      - :doc:`../ArchitectureInDetail/MessagingDetail/Email`
      - 新規追加

        *  E-mail送信(SMTP)を追加(\ `guideline#1165 <https://github.com/terasolunaorg/guideline/issues/1165>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/GeneralFuncDetail/DateAndTime`
      - 新規追加

        *  日付操作(JSR-310 Date and Time API)を追加(\ `guideline#1450 <https://github.com/terasolunaorg/guideline/issues/1450>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/GeneralFuncDetail/JodaTime`
      - 記載内容の改善・追加

        *  タイムゾーンを利用しない年月日を扱うサンプルコードのオブジェクトを\ ``LocalDate``\に修正(\ `guideline#1283 <https://github.com/terasolunaorg/guideline/issues/1283>`_\ )
        *  Java8未満のバージョンで和暦を扱う方法を追加(\ `guideline#1450 <https://github.com/terasolunaorg/guideline/issues/1450>`_\ )

    * -
      - :doc:`../ArchitectureInDetail/GeneralFuncDetail/StringProcessing`
      - 新規追加

        *  文字列処理を追加(\ `guideline#1451 <https://github.com/terasolunaorg/guideline/issues/1451>`_\ )

    * -
      - :doc:`../Security/index`
      - 構成見直し

        * \ ``パスワードハッシュ化``\は、:doc:`../Security/Authentication` に移動
        * :doc:`../Security/Authentication` から、セッション管理の項目を :doc:`../Security/SessionManagement` として独立

    * -
      - :doc:`../Security/SpringSecurity`
      - Spring Security 4 対応に伴う修正

        * 全記述の再編

         *  \ ``spring-security-testの紹介``\
         *  デフォルトでtrueになる仕様のため、サンプルソースから\ ``<use-expressions="true">``\を削除
         * \ ``RedirectAuthenticationHandler``\非推奨化に伴う説明の削除

    * -
      - :doc:`../Tutorial/TutorialSecurity`
      - Spring Security 4 対応に伴う修正

        * チュートリアルのソースをSpring Security 4 に対応した形に修正 (\ `guideline#1519 <https://github.com/terasolunaorg/guideline/issues/1519>`_\ )

    * -
      - :doc:`../Security/Authentication`
      - Spring Security 4 対応に伴う修正 (\ `guideline#1519 <https://github.com/terasolunaorg/guideline/issues/1519>`_\ )

        * 全記述の再編

         *  \ ``auto-config="true"``\の削除
         * 認証イベントリスナを\ ``@org.springframework.context.event.EventListener``\に修正
         *  \ ``AuthenticationPrincipal``\のパッケージを修正
         *  デフォルトでプレフィックスが付与されるため、サンプルソースから\ ``ROLE_``\プレフィックスの削除

    * -
      - :doc:`../Security/Authorization`
      - Spring Security 4 対応に伴う修正 (\ `guideline#1519 <https://github.com/terasolunaorg/guideline/issues/1519>`_\ )

        * 全記述の再編

         *  デフォルトでプレフィックスが付与されるため、サンプルソースから\ ``ROLE_``\プレフィックスの削除
         *  デフォルトでtrueになる仕様のため、サンプルソースから\ ``<use-expressions="true">``\を削除
         *  \ ``@PreAuthorize``\の定義例追加

    * -
      - :doc:`../Security/CSRF`
      - Spring Security 4 対応に伴う修正

        * 全記述の再編

         * CSRF無効化の設定を修正\ ``<sec:csrf disabled="true"/>``\

        * 記述内容の改善

         * マルチパートリクエストに関する項目を :doc:`../ArchitectureInDetail/WebApplicationDetail/FileUpload` に移動 (\ `guideline#1602 <https://github.com/terasolunaorg/guideline/issues/1602>`_\ )

    * -
      - :doc:`../Security/Encryption`
      - 新規追加

        * 暗号化ガイドラインの追加 (\ `guideline#1106 <https://github.com/terasolunaorg/guideline/issues/1106>`_\ )

    * -
      - :doc:`../Security/SecureLoginDemo`
      - 新規追加

        *  代表的なセキュリティ要件の実装例を追加(\ `guideline#1604 <https://github.com/terasolunaorg/guideline/issues/1604>`_\ )

    * -
      - :doc:`../Tutorial/TutorialSession`
      - 新規追加

        *  セッションチュートリアルを追加(\ `guideline#1599 <https://github.com/terasolunaorg/guideline/issues/1599>`_\ )

    * -
      - :doc:`../Tutorial/TutorialREST`
      - Spring Security 4 対応に伴う修正

        *  Spring Security 4 対応に伴うソースの修正 (\ `guideline#1519 <https://github.com/terasolunaorg/guideline/issues/1519>`_\ )

         * CSRF無効化の設定を修正\ ``<sec:csrf disabled="true"/>``\
         *  デフォルトでtrueになる仕様のため、サンプルソースから\ ``<use-expressions="true">``\を削除

    * - 2015-08-05
      - \-
      - 5.0.1 RELEASE版公開

        * 更新内容の詳細は、\ `5.0.1のIssue一覧 <https://github.com/terasolunaorg/guideline/issues?q=is%3Aissue+milestone%3A5.0.1+is%3Aclosed>`_\ を参照されたい。
    * -
      - 全般
      - ガイドラインの誤記(タイプミスや単純な記述ミスなど)の修正

        * 修正内容の詳細は、\ `5.0.1のIssue一覧(clerical error) <https://github.com/terasolunaorg/guideline/issues?q=is%3Aclosed+milestone%3A5.0.1+label%3A%22clerical+error%22>`_\ を参照されたい。

        記載内容の改善

        * 改善内容の詳細は、\ `5.0.1のIssue一覧(improvement) <https://github.com/terasolunaorg/guideline/issues?q=milestone%3A5.0.1+label%3Aimprovement+is%3Aclosed>`_\ を参照されたい。

        アプリケーションサーバに関する記載内容の修正

        * Resinに関する記載を削除
        * リファレンスページへのリンクを最新化
    * -
      - :doc:`index`
      - 記載内容の追加

        * ガイドラインに記載している内容の動作検証環境に関する記載を追加
    * -
      - :doc:`../Overview/FrameworkStack`
      - セキュリティ脆弱性対応に伴い利用するOSSのバージョン(Spring IO Platformのバージョン)を更新

        * Spring IO Platformのバージョンを1.1.3.RELEASEに更新
        * Spring Frameworkのバージョンを4.1.7.RELEASEに更新 (\ `CVE-2015-3192 <http://pivotal.io/security/cve-2015-3192>`_\ )
        * JSTLのバージョンを1.2.5に更新 (\ `CVE-2015-0254 <http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-0254>`_\ )

        Spring IO Platformのバージョン更新に伴い利用するOSSのバージョンを更新

        * 使用するOSSのバージョンを更新。更新内容は、\ `version 5.0.1の移行ガイド <https://github.com/terasolunaorg/terasoluna-gfw/wiki/Migration-Guide-5.0.1_ja#step-1-update-dependency-libraries>`_\ を参照されたい。

        記載内容の改善 (\ `guideline#1148 <https://github.com/terasolunaorg/guideline/issues/1148>`_\ )

        * \ ``terasoluna-gfw-recommended-dependencies``\ 、\ ``terasoluna-gfw-recommended-web-dependencies``\ 、\ ``terasoluna-gfw-parent``\ プロジェクトの説明を追加。
        * プロジェクトの説明を修正。
        * プロジェクト間の依存関係を示す図を追加。
    * -
      - :doc:`../ImplementationAtEachLayer/CreateWebApplicationProject`
      - 記載内容の追加

        * warファイルのビルド方法を追加 (\ `guideline#1146 <https://github.com/terasolunaorg/guideline/issues/1146>`_\ )
    * -
      - :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessCommon`
      - 記載内容の追加

        * データソース切り替え機能の説明を追加 (\ `guideline#1071 <https://github.com/terasolunaorg/guideline/issues/1071>`_\ )
    * -
      - :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessMyBatis3`
      - ガイドラインのバグ修正

        * バッチ実行のタイミングに関する説明を修正 (\ `guideline#903 <https://github.com/terasolunaorg/guideline/issues/903>`_\ )
    * -
      - :doc:`../ArchitectureInDetail/GeneralFuncDetail/Logging`
      - 記載内容の改善

        * \ ``<logger>``\ タグの\ ``additivity``\ 属性に関する説明を追加 (\ `guideline#977 <https://github.com/terasolunaorg/guideline/issues/977>`_\ )
    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/SessionManagement`
      - 記載内容の改善

        * セッションスコープのBeanの定義方法に関する説明を修正 (\ `guideline#1082 <https://github.com/terasolunaorg/guideline/issues/1082>`_\ )
    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/DoubleSubmitProtection`
      - 記載内容の追加

        * レスポンスをキャッシュしないように設定している時のトランザクショントークンチェックの動作を補足 (\ `guideline#1260 <https://github.com/terasolunaorg/guideline/issues/1260>`_\ )
    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Codelist`
      - 記載内容の追加

        * コード名の表示方法を追加 (\ `guideline#1109 <https://github.com/terasolunaorg/guideline/issues/1109>`_\ )
    * -
      - | :doc:`../ArchitectureInDetail/WebApplicationDetail/Ajax`
        | :doc:`../ArchitectureInDetail/WebServiceDetail/REST`
      - \ `CVE-2015-3192 <http://pivotal.io/security/cve-2015-3192>`_\ (XMLの脆弱性)に関する注意喚起を追加

        * StAX(Streaming API for XML)を使用する際の注意事項を追加 (\ `guideline#1211 <https://github.com/terasolunaorg/guideline/issues/1211>`_\ )
    * -
      - | :doc:`../ArchitectureInDetail/WebApplicationDetail/Pagination`
        | :doc:`../ArchitectureInDetail/WebApplicationDetail/TagLibAndELFunctions`
      - 共通ライブラリのバグ改修に伴う修正

        * 共通ライブラリのバグ改修(\ `terasoluna-gfw#297 <https://github.com/terasolunaorg/terasoluna-gfw/issues/297>`_\)に伴い、\ ``f:query``\ の仕様に関する説明を修正 (\ `guideline#1244 <https://github.com/terasolunaorg/guideline/issues/1244>`_\ )
    * -
      - :doc:`../Security/Authentication`
      - 記載内容の改善

        * \ ``ExceptionMappingAuthenticationFailureHandler``\ の親クラスのプロパティの扱いに関する注意点を追加 (\ `guideline#812 <https://github.com/terasolunaorg/guideline/issues/812>`_\ )
        * \ ``AbstractAuthenticationProcessingFilter``\ の\ ``requiresAuthenticationRequestMatcher``\ プロパティの設定例を修正 (\ `guideline#1110 <https://github.com/terasolunaorg/guideline/issues/1110>`_\ )
    * -
      - :doc:`../Security/Authorization`
      - ガイドラインのバグ修正

        * \ ``<sec:authorize>``\ タグ(JSPタグライブラリ)の\ ``access``\ 属性の設定例を修正 (\ `guideline#1003 <https://github.com/terasolunaorg/guideline/issues/1003>`_\ )
    * -
      - 環境依存性の排除
      - 記載内容の追加

        * Tomcat8使用時の外部クラスパス(Tomcat7の\ ``VirtualWebappLoader``\ の代替機能)の適用方法を追加 (\ `guideline#1081 <https://github.com/terasolunaorg/guideline/issues/1081>`_\ )
    * - 2015-06-12
      - 全般
      - 5.0.0 RELEASE英語版公開
    * - 2015-03-06
      - :doc:`../ArchitectureInDetail/WebServiceDetail/REST`
      - ガイドラインのバグ修正

        * 例外ハンドリング用のサンプルコード(\ ``NullPointerException``\ が発生するコードが含まれている問題)を修正。
          修正内容の詳細は、\ `guideline#918のIssue <https://github.com/terasolunaorg/guideline/issues/918>`_\ を参照されたい。
    * -
      - :doc:`../Tutorial/TutorialREST`
      - ガイドラインのバグ修正

        * 例外ハンドリングの処理で\ ``NullPointerException``\ が発生する問題を修正。
          修正内容の詳細は、\ `guideline#918のIssue <https://github.com/terasolunaorg/guideline/issues/918>`_\ を参照されたい。
    * - 2015-02-23
      - \-
      - 5.0.0 RELEASE版公開

        * 更新内容の詳細は、\ `5.0.0のIssue一覧 <https://github.com/terasolunaorg/guideline/issues?q=is%3Aissue+milestone%3A5.0.0+is%3Aclosed>`_\ と\ `1.0.2のバックポートIssue一覧 <https://github.com/terasolunaorg/guideline/issues?q=is%3Aclosed+milestone%3A1.0.2+label%3Abackport>`_\ を参照されたい。
    * -
      - 全般
      - ガイドラインの誤記(タイプミスや単純な記述ミスなど)の修正

        * 修正内容の詳細は、\ `1.0.2のバックポートIssue一覧(clerical error) <https://github.com/terasolunaorg/guideline/issues?q=is%3Aclosed+milestone%3A1.0.2+label%3Abackport+label%3A%22clerical+error%22>`_\ を参照されたい。

        記載内容の改善

        * 改善内容の詳細は、\ `5.0.0のIssue一覧(improvement) <https://github.com/terasolunaorg/guideline/issues?q=milestone%3A5.0.0+label%3Aimprovement+is%3Aclosed>`_\ と\ `1.0.2のバックポートIssue一覧(improvement) <https://github.com/terasolunaorg/guideline/issues?q=is%3Aclosed+milestone%3A1.0.2+label%3Aimprovement+label%3Abackport>`_\ を参照されたい。

        新規追加

        * :doc:`../ImplementationAtEachLayer/CreateWebApplicationProject`
        * :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessMyBatis3`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/TagLibAndELFunctions`
        * :doc:`../Appendix/Lombok`

        version 5.0.0対応に伴う更新

        * MyBatis2の説明を削除
    * -
      - :doc:`../Overview/FrameworkStack`
      - Spring IO Platform対応

        * 一部のライブラリを除き、推奨ライブラリの管理をSpring IO Platformに委譲する構成に変更した旨を追加。

        OSSバージョンの更新

        * 使用するOSSのバージョンを更新。更新内容は、\ `version 5.0.0の移行ガイド <https://github.com/terasolunaorg/terasoluna-gfw/wiki/Migration-Guide-5.0.0_ja#step-1-update-dependency-libraries>`_\ を参照されたい。
    * -
      - :doc:`../Overview/FirstApplication`
      - version 5.0.0対応に伴う更新

        * Spring Framework 4.1の適用。
        * ドキュメント上の構成の見直し。
    * -
      - :doc:`../Overview/ApplicationLayering`
      - 英語翻訳のバグ修正

        * ドメイン層と他の層との関係に関する翻訳ミスを修正。
          修正内容の詳細は、\ `guideline#364のIssue <https://github.com/terasolunaorg/guideline/issues/364>`_\ を参照されたい。
    * -
      - :doc:`../Tutorial/TutorialTodo`
      - version 5.0.0対応に伴う更新

        * Spring Framework 4.1の適用。
        * インフラストラクチャ層としてMyBatis3をサポート。
        * ドキュメント上の構成の見直し。
    * -
      - :doc:`../ImplementationAtEachLayer/CreateWebApplicationProject`
      - 新規追加

        * マルチプロジェクト構成のプロジェクト作成方法を追加。
    * -
      - :doc:`../ImplementationAtEachLayer/DomainLayer`
      - Spring Framework 4.1対応に伴う修正

        * JTA 1.2の\ ``@Transactional``\ の扱いに関する記載を追加。
          修正内容の詳細は、\ `guideline#562のIssue <https://github.com/terasolunaorg/guideline/issues/562>`_\ を参照されたい。
        * JPA(Hibernate実装)使用時の\ ``@Transactional(readOnly = true)``\ の扱い関する説明を修正。
          \ `SPR-8959 <https://jira.spring.io/browse/SPR-8959>`_\ (Spring Framework 4.1以降)の対応により、
          JDBCドライバに対して「読み取り専用のトランザクション」として扱うように指示できるように改善された。

        記載内容の追加

        * 「読み取り専用のトランザクション」が有効にならないケースに関する注意点を追加。
          追加内容の詳細は、\ `guideline#861のIssue <https://github.com/terasolunaorg/guideline/issues/861>`_\ を参照されたい。
    * -
      - :doc:`../ImplementationAtEachLayer/InfrastructureLayer`
      - MyBatis3対応に伴う修正

        * RepositoryImplの実装としてMyBatis3の仕組みを利用する方法を追加。
    * -
      - :doc:`../ImplementationAtEachLayer/ApplicationLayer`
      - Spring Framework 4.1対応に伴う修正

        * \ ``@ControllerAdvice``\ に追加された属性(適用対象をControllerを絞り込むための属性)に関する説明を追加。
          修正内容の詳細は、\ `guideline#549のIssue <https://github.com/terasolunaorg/guideline/issues/549>`_\ を参照されたい。
        * \ ``<mvc:view-resolvers>``\ に関する説明を追加。
          修正内容の詳細は、\ `guideline#609のIssue <https://github.com/terasolunaorg/guideline/issues/609>`_\ を参照されたい。
    * -
      - :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessCommon`
      - 共通ライブラリのバグ改修に伴う修正

        * 共通ライブラリのバグ改修(\ `terasoluna-gfw#78 <https://github.com/terasolunaorg/terasoluna-gfw/issues/78>`_\)に伴い、全角文字のワイルドカード文字(\ ``％``\ , \ ``＿``\ )\ の扱いに関する説明を追加。
          修正内容の詳細は、\ `guideline#712のIssue <https://github.com/terasolunaorg/guideline/issues/712>`_\ を参照されたい。

        Spring Framework 4.1対応に伴う修正

        * JPA(Hibernate実装)の悲観ロックエラーがSpring Frameworkの\ ``PessimisticLockingFailureException``\ に変換されない問題に関する記載を削除。
          この問題は、\ `SPR-10815 <https://jira.spring.io/browse/SPR-10815>`_\ (Spring Framework 4.0以降)で解決済みである。

        Apache Commons DBCP 2.0対応に伴う修正

        * Apache Commons DBCP 2.0用のコンポーネントを使用するようにサンプルコード及び説明を変更。
    * -
      - :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessMyBatis3`
      - 新規追加

        * O/R MapperとしてMyBatis3を使用してインフラストラクチャ層を実装する方法を追加。
    * -
      - :doc:`../ArchitectureInDetail/DataAccessDetail/ExclusionControl`
      - ガイドラインのバグ修正

        * ロングトランザクションの楽観ロックのサンプルコード(レコードが取得できない時の処理)の修正。
          修正内容の詳細は、\ `guideline#450のIssue <https://github.com/terasolunaorg/guideline/issues/450>`_\ を参照されたい。

        Spring Framework 4.1対応に伴う修正

        * JPA(Hibernate実装)の悲観ロックエラーがSpring Frameworkの\ ``PessimisticLockingFailureException``\ に変換されない問題に関する記載を削除。
          この問題は、\ `SPR-10815 <https://jira.spring.io/browse/SPR-10815>`_\ (Spring Framework 4.0以降)で解決済みである。

        MyBatis3対応に伴う修正

        * MyBatis3使用時の排他制御の実装方法を追加。
    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Validation`
      - ガイドラインのバグ修正

        * \ ``@GroupSequence``\ の説明を修正。
          修正内容の詳細は、\ `guideline#296のIssue <https://github.com/terasolunaorg/guideline/issues/296>`_\ を参照されたい。

        共通ライブラリのバグ改修に伴う修正

        * 共通ライブラリのバグ改修(\ `terasoluna-gfw#256 <https://github.com/terasolunaorg/terasoluna-gfw/issues/256>`_\)に伴い、\ ``ValidationMessages.properties``\ に関する注意点を追加。
          修正内容の詳細は、\ `guideline#766のIssue <https://github.com/terasolunaorg/guideline/issues/766>`_\ を参照されたい。

        記載内容の追加

        * Spring Validatorを使用した相関項目チェック時に、Bean ValidationのGroup Validationの仕組みと連携する方法を追加。
          追加内容の詳細は、\ `guideline#320のIssue <https://github.com/terasolunaorg/guideline/issues/320>`_\ を参照されたい。

        Bean Validation 1.1(Hibernate Validator 5.1)対応に伴う修正

        * \ ``@DecimalMin``\ と\ ``@DecimalMax``\ の\ ``inclusive``\ 属性の説明を追加。
        * Expression Languageに関する記載を追加。
        * Bean Validation 1.1から非推奨になったAPIについて記載。
        * Hibernate Validator 5.1.xの\ ``ValidationMessages.properties``\ に関するバグ(\ `HV-881 <https://hibernate.atlassian.net/browse/HV-881>`_\ )に関する記載と回避方法を追加。
    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/ExceptionHandling`
      - 記載内容の追加

        * 513バイトより小さいサイズのエラーをレスポンスするとInternet Explorerで簡易エラーページが表示される可能性がある旨の説明を追加。
          追加内容の詳細は、\ `guideline#189のIssue <https://github.com/terasolunaorg/guideline/issues/189>`_\ を参照されたい。

        Spring Framework 4.1対応に伴う修正

        * JPA(Hibernate実装)の悲観ロックエラーがSpring Frameworkの\ ``PessimisticLockingFailureException``\ に変換されない問題に関する記載を削除。
          この問題は、\ `SPR-10815 <https://jira.spring.io/browse/SPR-10815>`_\ (Spring Framework 4.0以降)で解決済みである。
    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/SessionManagement`
      - Spring Security 3.2対応に伴う修正

        * POSTリクエスト時にセッションタイムアウトではなくCSRFトークンエラーが発生する問題(\ `SEC-2422 <https://jira.springsource.org/browse/SEC-2422>`_\ )に関する記載を削除。
          Spring Security 3.2の正式版ではセッションタイムアウトを検知できる仕組みが組み込まれており、問題が解消されている。
    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/MessageManagement`
      - 共通ライブラリの変更内容の反映

        * 共通ライブラリの改善(\ `terasoluna-gfw#24 <https://github.com/terasolunaorg/terasoluna-gfw/issues/24>`_\)に伴い、新たに追加したメッセージタイプ(warning)と非推奨にしたメッセージタイプ(warn)に関する説明を追加。
          修正内容の詳細は、\ `guideline#74のIssue <https://github.com/terasolunaorg/guideline/issues/74>`_\ を参照されたい。
    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Pagination`
      - 共通ライブラリの変更内容の反映

        * 共通ライブラリの改善(\ `terasoluna-gfw#13 <https://github.com/terasolunaorg/terasoluna-gfw/issues/13>`_\)に伴い、active状態のページリンクの説明を変更。
          修正内容の詳細は、\ `guideline#699のIssue <https://github.com/terasolunaorg/guideline/issues/699>`_\ を参照されたい。
        * 共通ライブラリの改善(\ `terasoluna-gfw#14 <https://github.com/terasolunaorg/terasoluna-gfw/issues/14>`_\)に伴い、disabled状態のページリンクの説明を変更。
          修正内容の詳細は、\ `guideline#700のIssue <https://github.com/terasolunaorg/guideline/issues/700>`_\ を参照されたい。

        Spring Data Common 1.9対応に伴う修正

        * バージョンアップに伴い、API仕様が変更されているクラス(\ ``Page``\ インタフェースなど)に対する注意点を追加。
    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Codelist`
      - 共通ライブラリのバグ改修に伴う修正

        * 共通ライブラリのバグ改修(\ `terasoluna-gfw#16 <https://github.com/terasolunaorg/terasoluna-gfw/issues/16>`_\)に伴い、\ ``ExistInCodeList`` のメッセージキーを変更とバージョンアップ時の注意点を追加。
          修正内容の詳細は、\ `guideline#638のIssue <https://github.com/terasolunaorg/guideline/issues/638>`_\ を参照されたい。
        * 共通ライブラリのバグ改修(\ `terasoluna-gfw#256 <https://github.com/terasolunaorg/terasoluna-gfw/issues/256>`_\)に伴い、\ ``@ExistInCodeList``\ のメッセージ定義に関する注意点を追加。
          修正内容の詳細は、\ `guideline#766のIssue <https://github.com/terasolunaorg/guideline/issues/766>`_\ を参照されたい。

        共通ライブラリの変更内容の反映

        * 共通ライブラリの機能追加(\ `terasoluna-gfw#25 <https://github.com/terasolunaorg/terasoluna-gfw/issues/25>`_\)に伴い、\ ``EnumCodeList``\ クラスの使用方法を追加。
    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/Ajax`
      - Spring Security 3.2対応に伴う修正

        * CSRF対策のサンプルコード(CSRF対策用の\ ``<meta>``\ タグの生成方法)を変更。

        Jackson 2.4対応に伴う修正

        * Jackson 2.4用のコンポーネントを使用するようにサンプルコード及び説明を変更。
    * -
      - :doc:`../ArchitectureInDetail/WebServiceDetail/REST`
      - 記載内容の改善

        * Locationヘッダやハイパーメディアリンクに設定するURLを組み立てる方法を改善。
          改善内容の詳細は、\ `guideline#374のIssue <https://github.com/terasolunaorg/guideline/issues/374>`_\ を参照されたい。

        Spring Framework 4.1対応に伴う修正

        * \ ``@RestController``\ に関する説明を追加。
          修正内容の詳細は、\ `guideline#560のIssue <https://github.com/terasolunaorg/guideline/issues/560>`_\ を参照されたい。
        * ビルダースタイルのAPIを使用して\ ``ResponseEntity``\ を生成するようにサンプルコードを変更。

        Jackson 2.4対応に伴う修正

        * Jackson 2.4用のコンポーネントを使用するようにサンプルコード及び説明を変更。

        Spring Data Common 1.9対応に伴う修正

        * バージョンアップに伴い、API仕様が変更されているクラス(\ ``Page``\ インタフェースなど)に対する注意点を追加。
    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/FileUpload`
      - ガイドラインのバグ修正

        * \ `CVE-2014-0050 <http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0050>`_\ (File Uploadの脆弱性)が解決されたApache Commons FileUploadのバージョンを修正。
          修正内容の詳細は、\ `guideline#846のIssue <https://github.com/terasolunaorg/guideline/issues/846>`_\ を参照されたい。

        記載内容の追加

        * 一部のアプリケーションサーバでServlet 3のファイルアップロード機能が文字化けする問題があるため、この事象の回避策としてApache Commons FileUploadを使用する方法を追加。
          追加内容の詳細は、\ `guideline#778のIssue <https://github.com/terasolunaorg/guideline/issues/778>`_\ を参照されたい。
    * -
      - :doc:`../ArchitectureInDetail/GeneralFuncDetail/SystemDate`
      - 共通ライブラリの変更内容の反映

        * 共通ライブラリの改善(\ `terasoluna-gfw#224 <https://github.com/terasolunaorg/terasoluna-gfw/issues/224>`_\)に伴い、ドキュメント内の構成とパッケージ名及びクラス名を変更。
          修正内容の詳細は、\ `guideline#701のIssue <https://github.com/terasolunaorg/guideline/issues/701>`_\ を参照されたい。
    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/TilesLayout`
      - Tiles 3.0対応に伴う修正

        * Tiles 3.0用のコンポーネントを使用するように設定例及び説明を変更。

        Spring Framework 4.1対応に伴う修正

        * \ ``<mvc:view-resolvers>``\ 、\ ``<mvc:tiles>``\ 、\ ``<mvc:definitions>``\ に関する説明を追加。
          修正内容の詳細は、\ `guideline#609のIssue <https://github.com/terasolunaorg/guideline/issues/609>`_\ を参照されたい。
    * -
      - :doc:`../ArchitectureInDetail/GeneralFuncDetail/JodaTime`
      - 記載内容の追加

        * \ ``LocalDateTime``\ の使い方を追加。
          追加内容の詳細は、\ `guideline#584のIssue <https://github.com/terasolunaorg/guideline/issues/584>`_\ を参照されたい。

        Joda Time 2.5対応に伴う修正

        * バージョンアップに伴い\ ``DateMidnight``\ クラスが非推奨になったため、指定日の開始時刻(0:00:00.000)の取得方法を変更。
    * -
      - :doc:`../Security/SpringSecurity`
      - Spring Security 3.2対応に伴う修正

        * Appendixに「セキュアなHTTPヘッダー付与の設定」を追加。
    * -
      - :doc:`../Tutorial/TutorialSecurity`
      - version 5.0.0対応に伴う更新

        * インフラストラクチャ層としてMyBatis3を使用するように変更。
        * Spring Framework 4.1対応の適用。
        * Spring Security 3.2対応の適用。
        * ドキュメント上の構成の見直し。
    * -
      - :doc:`../Security/Authentication`
      - ガイドラインのバグ修正

        * \ ``<form-login>``\ 、\ ``<logout>``\ 、\ ``<session-management>``\ タグの説明不備や説明不足の修正。
          修正内容の詳細は、\ `guideline#754のIssue <https://github.com/terasolunaorg/guideline/issues/754>`_\ を参照されたい。
        * AuthenticationFilterの拡張方法を示すサンプルコードの修正(セッション・フィクセーション攻撃対策やCSRF対策を有効にするための設定を追加)。
          修正内容の詳細は、\ `guideline#765のIssue <https://github.com/terasolunaorg/guideline/issues/765>`_\ を参照されたい。

        Spring Security 3.2対応に伴う修正

        * CSRF対策を有効にしている際のログアウト方法に関する注意点を追加。
        * Controllerから\ ``UserDetails``\ (認証ユーザ情報クラス)にアクセスする方法として、\ ``@AuthenticationPrincipal``\ の説明を追加。
        * \ ``<sec:session-management>``\ の\ ``session-fixation-protection``\ 属性のパラメータとして、\ ``changeSessionId``\ の説明を追加。
        * セッションタイムアウトの検出方法と注意点を追加。
        * 同一ユーザの同時セッション数制御(Concurrent Session Control)を有効にするための設定方法を変更(\ ``<sec:concurrency-control>``\ を使用するように変更)。
        * 同一ユーザの同時セッション数制御関連のクラスが非推奨になり別のクラスが提供されている旨を追加。
    * -
      - :doc:`../Security/CSRF`
      - Spring Security 3.2対応に伴う修正

        * version 1.0.xの共通ライブラリに同封していたSpring Security 3.2.0(正式リリース前の暫定バージョン)のCSRF対策用コンポーネントに関する記載を削除。
        * CSRF対策を有効にするための設定方法をSpring Security 3.2の正式な作法(\ ``<sec:csrf>``\ を使用する方法)に変更。
        * CSRF対策用のJSPタグライブラリ(\ ``<sec:csrfInput>``\ と\ ``<sec:csrfMetaTags>``\ )に関する記載を追加。
        * CSRF対策を有効にしている時のセッションタイムアウトの検出方法と注意点を追加。

        Spring Framework 4.1対応に伴う修正

        * \ ``<form:form>``\ を使用した際に、CSRFトークンがhiddenとして出力される条件に関する記載を変更。
    * -
      - :doc:`../Tutorial/TutorialREST`
      - 記載内容の改善

        * \ :doc:`../Tutorial/TutorialTodo`\ で作成したプロジェクトにREST APIを追加する手順にすることで、特定のインフラストラクチャ層(O/R Mapper)に依存しない内容に変更。
          修正内容の詳細は、\ `guideline#325のIssue <https://github.com/terasolunaorg/guideline/issues/325>`_\ を参照されたい。

        version 5.0.0対応に伴う更新

        * Spring Framework 4.1対応の適用。
        * Spring Security 3.2対応の適用。
        * Jackson 2.4対応の適用。
    * -
      - ブランクプロジェクトから新規プロジェクトの作成
      - 記載内容の改善

        * マルチプロジェクト構成のプロジェクト作成方法をサポート。
        * シングルプロジェクト構成のプロジェクト作成方法を最新化。
    * -
      - :doc:`../ArchitectureInDetail/WebApplicationDetail/TagLibAndELFunctions`
      - 新規追加

        * 共通ライブラリから提供しているJSPタグライブラリとEL関数の説明を追加。
    * -
      - :doc:`../Appendix/Lombok`
      - 新規追加

        * Lombokを使用したボイラープレートコードの排除方法の説明を追加。
    * -
      - 英語版
      - 以下の英語版を追加

        * :doc:`../ImplementationAtEachLayer/CreateWebApplicationProject`
        * :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessCommon`
        * :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessJpa`
        * :doc:`../ArchitectureInDetail/DataAccessDetail/DataAccessMyBatis3`
        * :doc:`../ArchitectureInDetail/DataAccessDetail/ExclusionControl`
        * :doc:`../ArchitectureInDetail/GeneralFuncDetail/Logging`
        * :doc:`../ArchitectureInDetail/GeneralFuncDetail/PropertyManagement`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/Pagination`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/DoubleSubmitProtection`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/Internationalization`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/Codelist`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/Ajax`
        * :doc:`../ArchitectureInDetail/WebServiceDetail/REST`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/FileUpload`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/FileDownload`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/TilesLayout`
        * :doc:`../ArchitectureInDetail/GeneralFuncDetail/SystemDate`
        * :doc:`../ArchitectureInDetail/GeneralFuncDetail/Dozer`
        * :doc:`../Security/SpringSecurity`
        * :doc:`../Security/Authentication`
        * :doc:`../Security/Authorization`
        * :doc:`../Security/CSRF`
        * ブランクプロジェクトから新規のプロジェクトの作成
        * :doc:`../Appendix/Nexus`
        * 環境依存性の排除
        * Project Structure Standard
        * :doc:`../Appendix/Lombok`
        * :doc:`../Appendix/SpringComprehensionCheck`
    * - 2014-08-27
      - \-
      - 1.0.1 RELEASE版公開

        更新内容の詳細は、\ `1.0.1のIssue一覧 <https://github.com/terasolunaorg/guideline/issues?labels=&milestone=1&state=closed>`_\ を参照されたい。
    * -
      - 全般
      - ガイドラインのバグ(タイプミスや記述ミスなど)を修正

        更新内容の詳細は、\ `1.0.1のIssue一覧(bug & clerical error) <https://github.com/terasolunaorg/guideline/issues?labels=bug&milestone=1&state=closed>`_\ を参照されたい。
    * -
      - 日本語版
      - 以下の日本語版を追加

        * :doc:`CriteriaBasedMapping`
        * :doc:`../ArchitectureInDetail/WebServiceDetail/REST`
        * :doc:`../Tutorial/TutorialREST`
    * -
      - 英語版
      - 以下の英語版を追加

        * :doc:`index`
        * :doc:`../Overview/index`
        * :doc:`../Tutorial/TutorialTodo`
        * :doc:`../ImplementationAtEachLayer/index`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/Validation`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/ExceptionHandling`
        * :doc:`../ArchitectureInDetail/WebApplicationDetail/MessageManagement`
        * :doc:`../ArchitectureInDetail/GeneralFuncDetail/JodaTime`
        * :doc:`../Security/XSS`
        * :doc:`../Appendix/ReferenceBooks`
    * -
      - :doc:`../Overview/FrameworkStack`
      - バグ改修に伴い利用するOSSのバージョンを更新

        * GroupId「\ ``org.springframework``\」のバージョンを3.2.4.RELEASEから3.2.10.RELEASEに更新
        * GroupId「\ ``org.springframework.data``\」ArtifactId「\ ``spring-data-commons``\」のバージョンを1.6.1.RELEASEから1.6.4.RELEASEに更新
        * GroupId「\ ``org.springframework.data``\」ArtifactId「\ ``spring-data-jpa``\」のバージョンを1.4.1.RELEASEから1.4.3.RELEASEに更新
        * GroupId「\ ``org.aspectj``\」のバージョンを1.7.3から1.7.4に更新
        * GroupId「\ ``javax.transaction``\」ArtifactId「\ ``jta``\」を削除
    * -
      - :doc:`../ImplementationAtEachLayer/ApplicationLayer`
      - `CVE-2014-1904 <http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-1904>`_\(\ ``<form:form>``\タグのaction属性のXSS脆弱性)に関する注意喚起を追加
    * -
      - 日本語版

        :doc:`../ArchitectureInDetail/WebApplicationDetail/MessageManagement`
      - バグ改修に関する記載を追加

        * 共通ライブラリから提供している\ ``<t:messagesPanel>``\タグのバグ改修(\ `terasoluna-gfw#10 <https://github.com/terasolunaorg/terasoluna-gfw/issues/10>`_\)
    * -
      - 日本語版

        :doc:`../ArchitectureInDetail/WebApplicationDetail/Pagination`
      - バグ改修に関する記載を更新

        * 共通ライブラリから提供している\ ``<t:pagination>``\タグのバグ改修(\ `terasoluna-gfw#12 <https://github.com/terasolunaorg/terasoluna-gfw/issues/12>`_\)
        * Spring Data Commonsのバグ改修(\ `terasoluna-gfw#22 <https://github.com/terasolunaorg/terasoluna-gfw/issues/22>`_\)
    * -
      - 日本語版

        :doc:`../ArchitectureInDetail/WebApplicationDetail/Ajax`
      - XXE Injection対策に関する記載を更新
    * -
      - 日本語版

        :doc:`../ArchitectureInDetail/WebApplicationDetail/FileUpload`
      - `CVE-2014-0050 <http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0050>`_\(File Uploadの脆弱性)に関する注意喚起を追加

        ガイドラインのバグを修正

        * \ ``MultipartFilter``\を設定した場合、\ ``SystemExceptionResolver``\を使用して\ ``MultipartException``\をハンドリングする事が出来ないため、サーブレットコンテナのerror-page機能を使用してハンドリングする方法を追加。修正内容の詳細は、\ `guideline#59のIssue <https://github.com/terasolunaorg/guideline/issues/59>`_\ を参照されたい。
    * -
      - 日本語版
      - 以下のプロジェクト作成方法を\ ``mvn archetype:generate``\ から行うように変更

        * :doc:`../Overview/FirstApplication`
        * :doc:`../Tutorial/TutorialTodo`
        * :doc:`../Tutorial/TutorialTodo`
    * -
      - 日本語版
      - 以下のMavenアーキタイプ作成方法を微修正

        * :doc:`../Tutorial/TutorialSecurity`
        * ブランクプロジェクトから新規プロジェクトの作成
    * - 2013-12-17
      - 日本語版
      - 1.0.0 Public Review版公開

.. raw:: latex

   \newpage
