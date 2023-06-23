このドキュメントが示すこと
================================================================================

本ガイドラインではSpring、Spring MVCやJPA、MyBatisを中心としたフルスタックフレームワークを利用して、
保守性の高いWebアプリケーション開発をするためのベストプラクティスを提供する。

本ガイドラインを読むことで、ソフトウェア開発(主にコーディング)が円滑に進むことを期待する。

このドキュメントの対象読者
================================================================================

本ガイドラインはソフトウェア開発経験のあるアーキテクトやプログラマ向けに書かれており、
以下の知識があることを前提としている。

* Spring FrameworkのDIやAOPに関する基礎的な知識がある
* Servlet/JSPを使用してWebアプリケーションを開発したことがある
* SQLに関する知識がある
* Mavenを使用してWebアプリケションをビルドしたことがある

これからJavaを勉強し始めるという人向けではない。

Spring Frameworkに関して、本ドキュメントを読むための基礎知識があるかどうかを測るために
\ :doc:`../Appendix/SpringComprehensionCheck`\ を参照されたい。
この理解度テストが4割回答できない場合は、別途以下のような入門書籍で学習することを推奨する。

* `Spring徹底入門 (翔泳社) [日本語] <http://www.shoeisha.co.jp/book/detail/9784798142470>`_
* `Spring3入門―Javaフレームワーク・より良い設計とアーキテクチャ (技術評論社) [日本語] <http://gihyo.jp/book/2012/978-4-7741-5380-3>`_
* `Pro Spring 4th Edition (Apress) <http://www.apress.com/9781430261513>`_

このドキュメントの構成
================================================================================

* \ :doc:`../Overview/index`\ 
    Spring MVCの概要や、TERASOLUNA Server Framework for Java (5.x)の基本的な考え方を説明する。
* \ :doc:`../ImplementationAtEachLayer/index`\ 
    TERASOLUNA Server Framework for Java (5.x)を利用してアプリケーション開発する上で必ず押さえておかなくてはならない知識や作法について説明する。
* \ :doc:`../ArchitectureInDetail/WebApplicationDetail/index`\
    Webアプリケーション開発で必要となる機能をどう実装するか、何に気を付けるべきかを説明する。
* \ :doc:`../ArchitectureInDetail/WebServiceDetail/index`\
    Webサービス開発で必要となる機能をどう実装するか、何に気を付けるべきかを説明する。
* \ :doc:`../ArchitectureInDetail/DataAccessDetail/index`\
    データアクセス機能をどう実装するか、何に気を付けるべきかを説明する。
* \ :doc:`../ArchitectureInDetail/GeneralFuncDetail/index`\
    アプリケーション形態に依存しない汎用機能をどう実装するか、何に気を付けるべきかを説明する。
* \ :doc:`../ArchitectureInDetail/MessagingDetail/index`\
    メッセージ連携機能をどう実装するか、何に気を付けるべきかを説明する。
* \ :doc:`../Security/index`\
    Spring Securityを中心としたセキュリティ対策について説明する。
* \ :doc:`../Tutorial/index`\
    簡単なアプリケーション開発を通して、TERASOLUNA Server Framework for Java (5.x)によるアプリケーション開発を体験する。
* \ :doc:`../Appendix/index`\
    TERASOLUNA Server Framework for Java (5.x)を利用する場合の付加情報を説明する。

このドキュメントの読み方
================================================================================

まずは"\ :doc:`../Overview/index`\ "
から読み進めていただきたい。特にSpring MVCの経験がない場合は"\ :doc:`../Overview/FirstApplication`\ "を実施すること。
"\ :doc:`../Overview/ApplicationLayering`\ "は本ガイドラインで共通する用語と概念の説明を行っているため、必ず一読されたい。

次に"\ :doc:`../Tutorial/index`\ "に進む。
このチュートリアルでは"習うより慣れろ"を目的として、
詳細な説明の前にまず手を動かして、TERASOLUNA Server Framework for Java (5.x)によるアプリケーション開発を体感していただきたい。

チュートリアルを実践したのちに、"\ :doc:`../ImplementationAtEachLayer/index`\ "でアプリケーション開発の詳細を学ぶ。
特に"\ :doc:`../ImplementationAtEachLayer/ApplicationLayer`\ "でSpring MVCによる開発のノウハウを凝集して説明しているため、
何度も読み返すことを推奨する。
本章を読み終えた後にもう一度"\ :doc:`../Tutorial/index`\ "を振り返るとより理解が深まる。

**ここまではTERASOLUNA Server Framework for Java (5.x)を使用するすべての開発者が読むことを強く推奨する。**

"\ :doc:`../ArchitectureInDetail/WebApplicationDetail/index`\ "、"\ :doc:`../ArchitectureInDetail/WebServiceDetail/index`\ "、"\ :doc:`../ArchitectureInDetail/DataAccessDetail/index`\ "、"\ :doc:`../ArchitectureInDetail/GeneralFuncDetail/index`\ "、"\ :doc:`../ArchitectureInDetail/MessagingDetail/index`\ "、"\ :doc:`../Security/index`\ "については
目的に応じて必要なタイミングで参照すればよい。ただし、":doc:`../ArchitectureInDetail/WebApplicationDetail/Validation`"はアプリケーション開発で通常は必要となるため、基本的には読んでおくこと。

テクニカルリーダーはこれらをすべて読み内容を把握した上で
プロジェクトにおいて、どのような方針を定めるか検討していただきたい。


.. note::

    時間がない場合、まずは
    
    #. \ :doc:`../Overview/FirstApplication`\ 
    #. \ :doc:`../Overview/ApplicationLayering`\ 
    #. \ :doc:`../Tutorial/TutorialTodo`\ 
    #. \ :doc:`../ImplementationAtEachLayer/index`\ 
    #. \ :doc:`../Tutorial/TutorialTodo`\ 
    #. \ :doc:`../ArchitectureInDetail/WebApplicationDetail/Validation`\ 
    
    を読むとよい。

このドキュメントの動作検証環境
================================================================================

本ガイドラインで説明している内容の動作検証環境については、
「\ `テスト済み環境 <https://github.com/terasolunaorg/terasoluna-gfw-functionaltest/wiki/Tested-Environment>`_\」を参照されたい。


.. raw:: latex

   \newpage

