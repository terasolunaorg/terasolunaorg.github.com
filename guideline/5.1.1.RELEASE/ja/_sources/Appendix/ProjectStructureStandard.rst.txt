--------------------------------------------------------------------------------
Project Structure Standard
--------------------------------------------------------------------------------

.. todo::

  書き直し。


ソフトウェアのソースコードツリーは、複数のプロジェクトに分かれる、いわゆるマルチプロジェクト構成にすることを推奨する。

.. note::
 
 ここではビルドツールとしてmavenを想定している。
 mavenを使う場合の標準的なマルチプロジェクト構成は、 階層型レイアウト（ Hierachical project layout）である。
 しかし、開発環境として使用するeclipseは階層型レイアウトではなくフラットレイアウト(flat layout)にしか対応していないため、
 あえてフラットレイアウトを使用する。

Simple pattern
--------------

Webアプリケーション開発プロジェクト「foo」の、最もシンプルなプロジェクト構成は下記のようになる。

* foo-parent
* foo-initdb
* foo-domain
* foo-web
* foo-env
* foo-selenium

それぞれのプロジェクトの内容は下記のようになる。

.. _foo-parent-label: 

* foo-parent　

 parent-pom（親POM）と呼ばれるプロジェクト。pom.xmlファイルだけを持ち、
 その他のソースコードや設定ファイルは一切持たない、シンプルなプロジェクト。
 他のプロジェクトのpom上で、このfoo-parentプロジェクトを<parent>タグに指定することによって、
 親POMに指定された共通設定情報を自身に反映させることができる。

* foo-initdb

 RDBMSのテーブル定義(DDL)と初期データをINSERTするためのSQL文を格納する。
 これもmavenプロジェクトとして管理する。pom.xmlに `sql-maven-plugin <http://mojo.codehaus.org/sql-maven-plugin/>`_ 
 の設定を定義することにより、ビルドライフサイクルの過程で任意のRDBMSに対するDDL文や初期データINSERT文の実行を自動化することができる。

* foo-domain

 サービスクラスやリポジトリクラスなど、ドメイン層として使われるクラスを格納する。このドメイン層のクラスを使ってfoo-web内のアプリケーション層のクラスを組み立てる。

* foo-web

 アプリケーション層のクラス、jsp、設定ファイル、単体テストケース等を格納するプロジェクト。最終的にWebアプリケーションとして*.warファイル化する。

* foo-env

 環境依存性のある設定ファイルだけを集めるプロジェクト。foo-webはfoo-envへの依存性を持つ。
 詳細は :doc:`EnvironmentIndependency` を参照のこと。

* foo-selenium

 `Selenium WebDriver <http://seleniumhq.org/projects/webdriver/>`_ によるテストケースを格納するプロジェクト。
 
Complex pattern
---------------

２つのWebアプリケーションと１つの共通ライブラリが必要となる開発プロジェクト「bar」のプロジェクト構成は下記のようになる。

* bar-parent
* bar-initdb
* bar-common
* bar-common-web
* bar-domain-a
* bar-domain-b
* bar-web-a
* bar-web-b
* bar-env
* bar-web-a-selenium
* bar-web-b-selenium

それぞれのプロジェクトの内容は下記のようになる。

* bar-parent

 （foo-parentと同じ）

* bar-initdb

 （foo-initdbと同じ）

* bar-common

  プロジェクト共通ライブラリを格納する。ここはweb非依存にし、webに関わるクラスはbar-common-webに配置する。

* bar-common-web

 プロジェクト共通webライブラリを格納する

* bar-domain

 aドメインに関わるドメイン層のjavaクラス、単体テストケース等を格納するプロジェクト。最終的に*.jarファイル化する。

* bar-domain

 bドメインに関わるドメイン層のクラス。

* bar-web-a

 アプリケーション層のjavaクラス、jsp、設定ファイル、単体テストケース等を格納するプロジェクト。最終的にWebアプリケーションとして*.warファイル化する。
 bar-web-aは、bar-commonとbar-envへの依存性を持つ。

* bar-web-b

 もう一つのサブシステムとしてのWebアプリケーション。構造はbar-web-aと同じ。

* bar-env

 環境依存性のある設定ファイルだけを集めるプロジェクト。 詳細は :doc:`EnvironmentIndependency` を参照のこと。

* bar-web-a-selenium

 web-aプロジェクトのための、`Selenium WebDriver <http://seleniumhq.org/projects/webdriver/>`_ によるテストケースを格納するプロジェクト。

* bar-web-b-selenium

 web-bプロジェクトのための、`Selenium WebDriver <http://seleniumhq.org/projects/webdriver/>`_ によるテストケースを格納するプロジェクト。


.. todo::
    JSPも分割したい場合について記述する。

.. raw:: latex

   \newpage

