更新履歴
================================================================================

.. tabularcolumns:: |p{0.15\linewidth}|p{0.25\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 15 25 60

    * - 更新日付
      - 更新箇所
      - 更新内容
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

        * 使用するOSSのバージョンを更新。更新内容は、\ `version 5.0.1の移行ガイド <https://github.com/terasolunaorg/terasoluna-gfw/wiki/Migration-Guide-5.0.1#step-1-update-dependency-libraries>`_\ を参照されたい。

        記載内容の改善 (\ `guideline#1148 <https://github.com/terasolunaorg/guideline/issues/1148>`_\ )

        * \ ``terasoluna-gfw-recommended-dependencies``\ 、\ ``terasoluna-gfw-recommended-web-dependencies``\ 、\ ``terasoluna-gfw-parent``\ プロジェクトの説明を追加。
        * プロジェクトの説明を修正。
        * プロジェクト間の依存関係を示す図を追加。
    * -
      - :doc:`../ImplementationAtEachLayer/CreateWebApplicationProject`
      - 記載内容の追加

        * warファイルのビルド方法を追加 (\ `guideline#1146 <https://github.com/terasolunaorg/guideline/issues/1146>`_\ )
    * -
      - :doc:`../ArchitectureInDetail/DataAccessCommon`
      - 記載内容の追加

        * データソース切り替え機能の説明を追加 (\ `guideline#1071 <https://github.com/terasolunaorg/guideline/issues/1071>`_\ )
    * -
      - :doc:`../ArchitectureInDetail/DataAccessMyBatis3`
      - ガイドラインのバグ修正

        * バッチ実行のタイミングに関する説明を修正 (\ `guideline#903 <https://github.com/terasolunaorg/guideline/issues/903>`_\ )
    * -
      - :doc:`../ArchitectureInDetail/Logging`
      - 記載内容の改善

        * \ ``<logger>``\ タグの\ ``additivity``\ 属性に関する説明を追加 (\ `guideline#977 <https://github.com/terasolunaorg/guideline/issues/977>`_\ )
    * -
      - :doc:`../ArchitectureInDetail/SessionManagement`
      - 記載内容の改善

        * セッションスコープのBeanの定義方法に関する説明を修正 (\ `guideline#1082 <https://github.com/terasolunaorg/guideline/issues/1082>`_\ )
    * -
      - :doc:`../ArchitectureInDetail/DoubleSubmitProtection`
      - 記載内容の追加

        * レスポンスをキャッシュしないように設定している時のトランザクショントークンチェックの動作を補足 (\ `guideline#1260 <https://github.com/terasolunaorg/guideline/issues/1260>`_\ )
    * -
      - :doc:`../ArchitectureInDetail/Codelist`
      - 記載内容の追加

        * コード名の表示方法を追加 (\ `guideline#1109 <https://github.com/terasolunaorg/guideline/issues/1109>`_\ )
    * -
      - | :doc:`../ArchitectureInDetail/Ajax`
        | :doc:`../ArchitectureInDetail/REST`
      - \ `CVE-2015-3192 <http://pivotal.io/security/cve-2015-3192>`_\ (XMLの脆弱性)に関する注意喚起を追加

        * StAX(Streaming API for XML)を使用する際の注意事項を追加 (\ `guideline#1211 <https://github.com/terasolunaorg/guideline/issues/1211>`_\ )
    * -
      - | :doc:`../ArchitectureInDetail/Pagination`
        | :doc:`../Appendix/TagLibAndELFunctions`
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
      - :doc:`../Appendix/EnvironmentIndependency`
      - 記載内容の追加

        * Tomcat8使用時の外部クラスパス(Tomcat7の\ ``VirtualWebappLoader``\ の代替機能)の適用方法を追加 (\ `guideline#1081 <https://github.com/terasolunaorg/guideline/issues/1081>`_\ )
    * - 2015-06-12
      - 全般
      - 5.0.0 RELEASE英語版公開
    * - 2015-03-06
      - :doc:`../ArchitectureInDetail/REST`
      - ガイドラインのバグ修正

        * 例外ハンドリング用のサンプルコード(\ ``NullPointerException``\ が発生するコードが含まれている問題)を修正。
          修正内容の詳細は、\ `guideline#918のIssue <https://github.com/terasolunaorg/guideline/issues/918>`_\ を参照されたい。
    * -
      - :doc:`../TutorialREST/index`
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
        * :doc:`../ArchitectureInDetail/DataAccessMyBatis3`
        * :doc:`../Appendix/TagLibAndELFunctions`
        * :doc:`../Appendix/Lombok`

        version 5.0.0対応に伴う更新

        * MyBatis2の説明を削除
    * -
      - :doc:`../Overview/FrameworkStack`
      - Spring IO Platform対応

        * 一部のライブラリを除き、推奨ライブラリの管理をSpring IO Platformに委譲する構成に変更した旨を追加。

        OSSバージョンの更新

        * 使用するOSSのバージョンを更新。更新内容は、\ `version 5.0.0の移行ガイド <https://github.com/terasolunaorg/terasoluna-gfw/wiki/Migration-Guide-5.0.0#step-1-update-dependency-libraries>`_\ を参照されたい。
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
      - :doc:`../TutorialTodo/index`
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
      - :doc:`../ArchitectureInDetail/DataAccessCommon`
      - 共通ライブラリのバグ改修に伴う修正

        * 共通ライブラリのバグ改修(\ `terasoluna-gfw#78 <https://github.com/terasolunaorg/terasoluna-gfw/issues/78>`_\)に伴い、全角文字のワイルドカード文字(\ ``％``\ , \ ``＿``\ )\ の扱いに関する説明を追加。
          修正内容の詳細は、\ `guideline#712のIssue <https://github.com/terasolunaorg/guideline/issues/712>`_\ を参照されたい。

        Spring Framework 4.1対応に伴う修正

        * JPA(Hibernate実装)の悲観ロックエラーがSpring Frameworkの\ ``PessimisticLockingFailureException``\ に変換されない問題に関する記載を削除。
          この問題は、\ `SPR-10815 <https://jira.spring.io/browse/SPR-10815>`_\ (Spring Framework 4.0以降)で解決済みである。

        Apache Commons DBCP 2.0対応に伴う修正

        * Apache Commons DBCP 2.0用のコンポーネントを使用するようにサンプルコード及び説明を変更。
    * -
      - :doc:`../ArchitectureInDetail/DataAccessMyBatis3`
      - 新規追加

        * O/R MapperとしてMyBatis3を使用してインフラストラクチャ層を実装する方法を追加。
    * -
      - :doc:`../ArchitectureInDetail/ExclusionControl`
      - ガイドラインのバグ修正

        * ロングトランザクションの楽観ロックのサンプルコード(レコードが取得できない時の処理)の修正。
          修正内容の詳細は、\ `guideline#450のIssue <https://github.com/terasolunaorg/guideline/issues/450>`_\ を参照されたい。

        Spring Framework 4.1対応に伴う修正

        * JPA(Hibernate実装)の悲観ロックエラーがSpring Frameworkの\ ``PessimisticLockingFailureException``\ に変換されない問題に関する記載を削除。
          この問題は、\ `SPR-10815 <https://jira.spring.io/browse/SPR-10815>`_\ (Spring Framework 4.0以降)で解決済みである。

        MyBatis3対応に伴う修正

        * MyBatis3使用時の排他制御の実装方法を追加。
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

        Bean Validation 1.1(Hibernate Validator 5.1)対応に伴う修正

        * \ ``@DecimalMin``\ と\ ``@DecimalMax``\ の\ ``inclusive``\ 属性の説明を追加。
        * Expression Languageに関する記載を追加。
        * Bean Validation 1.1から非推奨になったAPIについて記載。
        * Hibernate Validator 5.1.xの\ ``ValidationMessages.properties``\ に関するバグ(\ `HV-881 <https://hibernate.atlassian.net/browse/HV-881>`_\ )に関する記載と回避方法を追加。
    * -
      - :doc:`../ArchitectureInDetail/ExceptionHandling`
      - 記載内容の追加

        * 513バイトより小さいサイズのエラーをレスポンスするとInternet Explorerで簡易エラーページが表示される可能性がある旨の説明を追加。
          追加内容の詳細は、\ `guideline#189のIssue <https://github.com/terasolunaorg/guideline/issues/189>`_\ を参照されたい。

        Spring Framework 4.1対応に伴う修正

        * JPA(Hibernate実装)の悲観ロックエラーがSpring Frameworkの\ ``PessimisticLockingFailureException``\ に変換されない問題に関する記載を削除。
          この問題は、\ `SPR-10815 <https://jira.spring.io/browse/SPR-10815>`_\ (Spring Framework 4.0以降)で解決済みである。
    * -
      - :doc:`../ArchitectureInDetail/SessionManagement`
      - Spring Security 3.2対応に伴う修正

        * POSTリクエスト時にセッションタイムアウトではなくCSRFトークンエラーが発生する問題(\ `SEC-2422 <https://jira.springsource.org/browse/SEC-2422>`_\ )に関する記載を削除。
          Spring Security 3.2の正式版ではセッションタイムアウトを検知できる仕組みが組み込まれており、問題が解消されている。
    * -
      - :doc:`../ArchitectureInDetail/MessageManagement`
      - 共通ライブラリの変更内容の反映

        * 共通ライブラリの改善(\ `terasoluna-gfw#24 <https://github.com/terasolunaorg/terasoluna-gfw/issues/24>`_\)に伴い、新たに追加したメッセージタイプ(warning)と非推奨にしたメッセージタイプ(warn)に関する説明を追加。
          修正内容の詳細は、\ `guideline#74のIssue <https://github.com/terasolunaorg/guideline/issues/74>`_\ を参照されたい。
    * -
      - :doc:`../ArchitectureInDetail/Pagination`
      - 共通ライブラリの変更内容の反映

        * 共通ライブラリの改善(\ `terasoluna-gfw#13 <https://github.com/terasolunaorg/terasoluna-gfw/issues/13>`_\)に伴い、active状態のページリンクの説明を変更。
          修正内容の詳細は、\ `guideline#699のIssue <https://github.com/terasolunaorg/guideline/issues/699>`_\ を参照されたい。
        * 共通ライブラリの改善(\ `terasoluna-gfw#14 <https://github.com/terasolunaorg/terasoluna-gfw/issues/14>`_\)に伴い、disabled状態のページリンクの説明を変更。
          修正内容の詳細は、\ `guideline#700のIssue <https://github.com/terasolunaorg/guideline/issues/700>`_\ を参照されたい。

        Spring Data Common 1.9対応に伴う修正

        * バージョンアップに伴い、API仕様が変更されているクラス(\ ``Page``\ インタフェースなど)に対する注意点を追加。
    * -
      - :doc:`../ArchitectureInDetail/Codelist`
      - 共通ライブラリのバグ改修に伴う修正

        * 共通ライブラリのバグ改修(\ `terasoluna-gfw#16 <https://github.com/terasolunaorg/terasoluna-gfw/issues/16>`_\)に伴い、\ ``ExistInCodeList`` のメッセージキーを変更とバージョンアップ時の注意点を追加。
          修正内容の詳細は、\ `guideline#638のIssue <https://github.com/terasolunaorg/guideline/issues/638>`_\ を参照されたい。
        * 共通ライブラリのバグ改修(\ `terasoluna-gfw#256 <https://github.com/terasolunaorg/terasoluna-gfw/issues/256>`_\)に伴い、\ ``@ExistInCodeList``\ のメッセージ定義に関する注意点を追加。
          修正内容の詳細は、\ `guideline#766のIssue <https://github.com/terasolunaorg/guideline/issues/766>`_\ を参照されたい。

        共通ライブラリの変更内容の反映

        * 共通ライブラリの機能追加(\ `terasoluna-gfw#25 <https://github.com/terasolunaorg/terasoluna-gfw/issues/25>`_\)に伴い、\ ``EnumCodeList``\ クラスの使用方法を追加。
    * -
      - :doc:`../ArchitectureInDetail/Ajax`
      - Spring Security 3.2対応に伴う修正

        * CSRF対策のサンプルコード(CSRF対策用の\ ``<meta>``\ タグの生成方法)を変更。

        Jackson 2.4対応に伴う修正

        * Jackson 2.4用のコンポーネントを使用するようにサンプルコード及び説明を変更。
    * -
      - :doc:`../ArchitectureInDetail/REST`
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
      - :doc:`../ArchitectureInDetail/FileUpload`
      - ガイドラインのバグ修正

        * \ `CVE-2014-0050 <http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0050>`_\ (File Uploadの脆弱性)が解決されたApache Commons FileUploadのバージョンを修正。
          修正内容の詳細は、\ `guideline#846のIssue <https://github.com/terasolunaorg/guideline/issues/846>`_\ を参照されたい。

        記載内容の追加

        * 一部のアプリケーションサーバでServlet 3のファイルアップロード機能が文字化けする問題があるため、この事象の回避策としてApache Commons FileUploadを使用する方法を追加。
          追加内容の詳細は、\ `guideline#778のIssue <https://github.com/terasolunaorg/guideline/issues/778>`_\ を参照されたい。
    * -
      - :doc:`../ArchitectureInDetail/SystemDate`
      - 共通ライブラリの変更内容の反映

        * 共通ライブラリの改善(\ `terasoluna-gfw#224 <https://github.com/terasolunaorg/terasoluna-gfw/issues/224>`_\)に伴い、ドキュメント内の構成とパッケージ名及びクラス名を変更。
          修正内容の詳細は、\ `guideline#701のIssue <https://github.com/terasolunaorg/guideline/issues/701>`_\ を参照されたい。
    * -
      - :doc:`../ArchitectureInDetail/TilesLayout`
      - Tiles 3.0対応に伴う修正

        * Tiles 3.0用のコンポーネントを使用するように設定例及び説明を変更。

        Spring Framework 4.1対応に伴う修正

        * \ ``<mvc:view-resolvers>``\ 、\ ``<mvc:tiles>``\ 、\ ``<mvc:definitions>``\ に関する説明を追加。
          修正内容の詳細は、\ `guideline#609のIssue <https://github.com/terasolunaorg/guideline/issues/609>`_\ を参照されたい。
    * -
      - :doc:`../ArchitectureInDetail/Utilities/JodaTime`
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
      - :doc:`../Security/Tutorial`
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
      - :doc:`../TutorialREST/index`
      - 記載内容の改善

        * \ :doc:`../TutorialTodo/index`\ で作成したプロジェクトにREST APIを追加する手順にすることで、特定のインフラストラクチャ層(O/R Mapper)に依存しない内容に変更。
          修正内容の詳細は、\ `guideline#325のIssue <https://github.com/terasolunaorg/guideline/issues/325>`_\ を参照されたい。

        version 5.0.0対応に伴う更新

        * Spring Framework 4.1対応の適用。
        * Spring Security 3.2対応の適用。
        * Jackson 2.4対応の適用。
    * -
      - :doc:`../Appendix/CreateProjectFromBlank`
      - 記載内容の改善

        * マルチプロジェクト構成のプロジェクト作成方法をサポート。
        * シングルプロジェクト構成のプロジェクト作成方法を最新化。
    * -
      - :doc:`../Appendix/TagLibAndELFunctions`
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
        * :doc:`../ArchitectureInDetail/DataAccessCommon`
        * :doc:`../ArchitectureInDetail/DataAccessJpa`
        * :doc:`../ArchitectureInDetail/DataAccessMyBatis3`
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
        * :doc:`../Security/Authentication`
        * :doc:`../Security/PasswordHashing`
        * :doc:`../Security/Authorization`
        * :doc:`../Security/CSRF`
        * :doc:`../Appendix/CreateProjectFromBlank`
        * :doc:`../Appendix/Nexus`
        * :doc:`../Appendix/EnvironmentIndependency`
        * :doc:`../Appendix/ProjectStructureStandard`
        * :doc:`../Appendix/Lombok`
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

