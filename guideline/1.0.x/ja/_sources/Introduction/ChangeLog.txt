更新履歴
================================================================================

.. tabularcolumns:: |p{0.15\linewidth}|p{0.25\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 15 25 60

    * - 更新日付
      - 更新箇所
      - 更新内容
    * - 2015-02-xx
      - \-
      - 1.0.2 RELEASE版公開

        更新内容の詳細は、\ `1.0.2のIssue一覧 <https://github.com/terasolunaorg/guideline/issues?q=is%3Aissue+milestone%3A1.0.2+is%3Aclosed>`_\ を参照されたい。
    * -
      - 全般
      - ガイドラインの誤記(タイプミスや単純な記述ミスなど)の修正

        * 修正内容の詳細は、\ `1.0.2のIssue一覧(clerical error) <https://github.com/terasolunaorg/guideline/issues?q=is%3Aissue+milestone%3A1.0.2+is%3Aclosed+label%3A%22clerical+error%22>`_\ を参照されたい。

        記載内容の改善

        * 改善内容の詳細は、\ `1.0.2のIssue一覧(improvement) <https://github.com/terasolunaorg/guideline/issues?q=is%3Aissue+milestone%3A1.0.2+label%3Aimprovement+is%3Aclosed>`_\ を参照されたい。

        新規追加

        * :doc:`../Appendix/TagLibAndELFunctions`
    * -
      - :doc:`../Overview/FrameworkStack`
      - Spring Frameworkのバグ(セキュリティ脆弱性)改修に伴い利用するOSSのバージョンを更新

        * GroupId「\ ``org.springframework``\」のバージョンを3.2.10.RELEASEから3.2.13.RELEASEに更新
    * -
      - :doc:`../Overview/ApplicationLayering`
      - 英語翻訳のバグ修正

        * ドメイン層と他の層との関係に関する翻訳ミスを修正。
          修正内容の詳細は、\ `guideline#364のIssue <https://github.com/terasolunaorg/guideline/issues/364>`_\ を参照されたい。
    * -
      - :doc:`../ArchitectureInDetail/DataAccessCommon`
      - 共通ライブラリのバグ改修に伴う修正

        * 共通ライブラリのバグ改修(\ `terasoluna-gfw#78 <https://github.com/terasolunaorg/terasoluna-gfw/issues/78>`_\)に伴い、全角文字のワイルドカード文字(\ ``％``\ , \ ``＿``\ )\ の扱いに関する説明を追加。
          修正内容の詳細は、\ `guideline#712のIssue <https://github.com/terasolunaorg/guideline/issues/712>`_\ を参照されたい。
    * -
      - :doc:`../ArchitectureInDetail/DataAccessMybatis2`
      - ガイドラインのバグ修正

        * LOB型を扱うための設定と説明を修正。
          修正内容の詳細は、\ `guideline#402のIssue <https://github.com/terasolunaorg/guideline/issues/402>`_\ を参照されたい。
    * -
      - :doc:`../ArchitectureInDetail/ExclusionControl`
      - ガイドラインのバグ修正

        * ロングトランザクションの楽観ロックのサンプルコード(レコードが取得できない時の処理)の修正。
          修正内容の詳細は、\ `guideline#450のIssue <https://github.com/terasolunaorg/guideline/issues/450>`_\ を参照されたい。
    * -
      - :doc:`../ArchitectureInDetail/Validation`
      - ガイドラインのバグ修正

        * \ ``@GroupSequence``\ の説明を修正。
          修正内容の詳細は、\ `guideline#296のIssue <https://github.com/terasolunaorg/guideline/issues/296>`_\ を参照されたい。

        共通ライブラリのバグ改修に伴う修正

        * 共通ライブラリのバグ改修(\ `terasoluna-gfw#256 <https://github.com/terasolunaorg/terasoluna-gfw/issues/256>`_\)に伴い、\ ``ValidationMessages.properties``\ に関する注意点を追加。
          修正内容の詳細は、\ `guideline#766のIssue <https://github.com/terasolunaorg/guideline/issues/766>`_\ を参照されたい。

        記載内容の追加

        * Spring Validatorを使用した相関項目チェック時に、Bean ValidationのGroup Validationの仕組みと連携する方法を追加。
          追加内容の詳細は、\ `guideline#320のIssue <https://github.com/terasolunaorg/guideline/issues/320>`_\ を参照されたい。
    * -
      - :doc:`../ArchitectureInDetail/ExceptionHandling`
      - 記載内容の追加

        * 513バイトより小さいサイズのエラーをレスポンスするとInternet Explorerで簡易エラーページが表示される可能性がある旨の説明を追加。
          追加内容の詳細は、\ `guideline#189のIssue <https://github.com/terasolunaorg/guideline/issues/189>`_\ を参照されたい。
    * -
      - :doc:`../ArchitectureInDetail/Codelist`
      - 共通ライブラリのバグ改修に伴う修正

        * 共通ライブラリのバグ改修(\ `terasoluna-gfw#256 <https://github.com/terasolunaorg/terasoluna-gfw/issues/256>`_\)に伴い、\ ``@ExistInCodeList``\ のメッセージ定義に関する注意点を追加。
          修正内容の詳細は、\ `guideline#766のIssue <https://github.com/terasolunaorg/guideline/issues/766>`_\ を参照されたい。
    * -
      - :doc:`../ArchitectureInDetail/Ajax`
      - 記載内容の改善

        * Spring Security 3.2(version 5.0.0で利用)との互換性を持たせるため、CSRF対策のサンプルコード(\ ``<meta>``\ タグの\ ``name``\ 属性に設定する値)を修正。
          改善内容の詳細は、\ `guideline#680のIssue <https://github.com/terasolunaorg/guideline/issues/680>`_\ を参照されたい。
    * -
      - :doc:`../ArchitectureInDetail/REST`
      - 記載内容の改善

        * Locationヘッダやハイパーメディアリンクに設定するURLを組み立てる方法を改善。
          改善内容の詳細は、\ `guideline#374のIssue <https://github.com/terasolunaorg/guideline/issues/374>`_\ を参照されたい。
    * -
      - :doc:`../ArchitectureInDetail/FileUpload`
      - ガイドラインのバグ修正

        * \ `CVE-2014-0050 <http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0050>`_\ (File Uploadの脆弱性)が解決されたApache Commons FileUploadのバージョンを修正。
          修正内容の詳細は、\ `guideline#846のIssue <https://github.com/terasolunaorg/guideline/issues/846>`_\ を参照されたい。

        記載内容の追加

        * 一部のアプリケーションサーバでServlet 3のファイルアップロード機能が文字化けする問題があるため、この事象の回避策としてApache Commons FileUploadを使用する方法を追加。
          追加内容の詳細は、\ `guideline#778のIssue <https://github.com/terasolunaorg/guideline/issues/778>`_\ を参照されたい。
    * -
      - :doc:`../ArchitectureInDetail/Utilities/JodaTime`
      - 記載内容の追加

        * \ ``LocalDateTime``\ の使い方を追加。
          追加内容の詳細は、\ `guideline#584のIssue <https://github.com/terasolunaorg/guideline/issues/584>`_\ を参照されたい。
    * -
      - :doc:`../Security/Authentication`
      - ガイドラインのバグ修正

        * \ ``<form-login>``\ 、\ ``<logout>``\ 、\ ``<session-management>``\ タグの説明不備や説明不足の修正。
          修正内容の詳細は、\ `guideline#754のIssue <https://github.com/terasolunaorg/guideline/issues/754>`_\ を参照されたい。
        * AuthenticationFilterの拡張方法を示すサンプルコードの修正(セッション・フィクセーション攻撃対策やCSRF対策を有効にするための設定を追加)。
          修正内容の詳細は、\ `guideline#765のIssue <https://github.com/terasolunaorg/guideline/issues/765>`_\ を参照されたい。
    * -
      - :doc:`../Appendix/TagLibAndELFunctions`
      - 新規追加

        * 共通ライブラリから提供しているJSPタグライブラリとEL関数の説明を追加。
    * -
      - 英語版
      - 以下の英語版を追加

        * :doc:`../ArchitectureInDetail/DataAccessCommon`
        * :doc:`../ArchitectureInDetail/DataAccessJpa`
        * :doc:`../ArchitectureInDetail/DataAccessMybatis2`
        * :doc:`../ArchitectureInDetail/ExclusionControl`
        * :doc:`../ArchitectureInDetail/Logging`
        * :doc:`../ArchitectureInDetail/PropertyManagement`
        * :doc:`../ArchitectureInDetail/Pagination`
        * :doc:`../ArchitectureInDetail/DoubleSubmitProtection`
        * :doc:`../ArchitectureInDetail/Internationalization`
        * :doc:`../ArchitectureInDetail/Codelist`
        * :doc:`../ArchitectureInDetail/Ajax`
        * :doc:`../ArchitectureInDetail/REST`
        * :doc:`../ArchitectureInDetail/FileUpload`
        * :doc:`../ArchitectureInDetail/FileDownload`
        * :doc:`../ArchitectureInDetail/TilesLayout`
        * :doc:`../ArchitectureInDetail/SystemDate`
        * :doc:`../ArchitectureInDetail/Utilities/Dozer`
        * :doc:`../Security/SpringSecurity`
        * :doc:`../Security/PasswordHashing`
        * :doc:`../Security/Authorization`
        * :doc:`../Appendix/CreateProjectFromBlank`
        * :doc:`../Appendix/Nexus`
        * :doc:`../Appendix/EnvironmentIndependency`
        * :doc:`../Appendix/ProjectStructureStandard`
        * :doc:`../Appendix/SpringComprehensionCheck`
    * - 2014-08-27
      - \-
      - 1.0.1 RELEASE版公開
        
        更新内容の詳細は、\ `1.0.1のIssue一覧 <https://github.com/terasolunaorg/guideline/issues?labels=&milestone=1&state=closed>`_\ を参照されたい。
    * - 
      - 全般
      - ガイドラインのバグ(タイプミスや記述ミスなど)を修正

        更新内容の詳細は、\ `1.0.1のIssue一覧 <https://github.com/terasolunaorg/guideline/issues?labels=bug&milestone=1&state=closed>`_\ を参照されたい。
    * - 
      - 日本語版
      - 以下の日本語版を追加
      
        * :doc:`CriteriaBasedMapping`
        * :doc:`../ArchitectureInDetail/REST`
        * :doc:`../TutorialREST/index`
    * - 
      - 英語版
      - 以下の英語版を追加
      
        * :doc:`index`
        * :doc:`../Overview/index`
        * :doc:`../TutorialTodo/index`
        * :doc:`../ImplementationAtEachLayer/index`
        * :doc:`../ArchitectureInDetail/Validation`
        * :doc:`../ArchitectureInDetail/ExceptionHandling`
        * :doc:`../ArchitectureInDetail/MessageManagement`
        * :doc:`../ArchitectureInDetail/Utilities/JodaTime`
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
      
        :doc:`../ArchitectureInDetail/MessageManagement`
      - バグ改修に関する記載を追加
      
        * 共通ライブラリから提供している\ ``<t:messagesPanel>``\タグのバグ改修(\ `terasoluna-gfw#10 <https://github.com/terasolunaorg/terasoluna-gfw/issues/10>`_\)
    * - 
      - 日本語版
      
        :doc:`../ArchitectureInDetail/Pagination`
      - バグ改修に関する記載を更新
      
        * 共通ライブラリから提供している\ ``<t:pagination>``\タグのバグ改修(\ `terasoluna-gfw#12 <https://github.com/terasolunaorg/terasoluna-gfw/issues/12>`_\)
        * Spring Data Commonsのバグ改修(\ `terasoluna-gfw#22 <https://github.com/terasolunaorg/terasoluna-gfw/issues/22>`_\)
    * - 
      - 日本語版
      
        :doc:`../ArchitectureInDetail/Ajax`
      - XXE Injection対策に関する記載を更新
    * - 
      - 日本語版
      
        :doc:`../ArchitectureInDetail/FileUpload`
      - `CVE-2014-0050 <http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0050>`_\(File Uploadの脆弱性)に関する注意喚起を追加
      
        ガイドラインのバグを修正
        
        * \ ``MultipartFilter``\を設定した場合、\ ``SystemExceptionResolver``\を使用して\ ``MultipartException``\をハンドリングする事が出来ないため、サーブレットコンテナのerror-page機能を使用してハンドリングする方法を追加。修正内容の詳細は、\ `guideline#59のIssue <https://github.com/terasolunaorg/guideline/issues/59>`_\ を参照されたい。
    * - 
      - 日本語版
      - 以下のプロジェクト作成方法を\ ``mvn archetype:generate``\ から行うように変更
      
        * :doc:`../Overview/FirstApplication`
        * :doc:`../TutorialTodo/index`
        * :doc:`../TutorialTodo/index`
    * - 
      - 日本語版
      - 以下のMavenアーキタイプ作成方法を微修正
      
        * :doc:`../Security/Tutorial`
        * :doc:`../Appendix/CreateProjectFromBlank`
    * - 2013-12-17
      - 日本語版
      - 1.0.0 Public Review版公開

.. raw:: latex

   \newpage

