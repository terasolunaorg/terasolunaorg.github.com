更新履歴
================================================================================

.. tabularcolumns:: |p{0.15\linewidth}|p{0.25\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 15 25 60

    * - 更新日付
      - 更新箇所
      - 更新内容
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

