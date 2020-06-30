
.. _RunUnitTest:

単体テストの実行
================================================================================

.. only:: html

 .. contents:: 目次
    :local:

テストの実行方法
--------------------------------------------------------------------------------

本節では、JUnitの実行方法として、

* IDE上でJUnitを実行する方法
* MavenでJUnitを実行する方法

の2種類を説明する。用途に応じていずれかを選択されたい。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 40

    * - テスト実行方法
      - 説明
    * - IDE
      - IDE上からJUnitを実行する。
    * - Maven
      - \ ``mvn test``\ コマンド を使用してJUnitを実行する。

IDE上からJUnitを実行する場合は、IDE上で製造からテストまで行うことができるため、
テスト対象への参照も容易となる。

\ ``mvn test``\ コマンドを使用してJUnitを実行する場合は、コマンドベースで実行するため、
CIサーバ上でのテストの自動化が可能となる。

テストの実行
--------------------------------------------------------------------------------


IDE上でテストを実行
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

STSを用いてJUnitを実行する方法を紹介する。

テストクラスの実行
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

テストクラスを右クリックし、メニューを表示させる。

.. figure:: ./images/RunUnitTestIdeClickTestClass.png

|

メニューから[Run As] -> [JUnit Test]を選択し、対象テストクラスを実行する。

.. figure:: ./images/RunUnitTestIdeRunClassJunit.png

|

不具合なくテストが実行されれば、以下のような画面が表示される。

.. figure:: ./images/RunUnitTestIdeSuccessJunit.png

|

アサーションエラーの場合、
以下のようにエラーメッセージが表示される。

.. figure:: ./images/RunUnitTestIdeFailJunit.png

|

テストを実行する上でエラーが発生した場合、
以下のようにエラーメッセージが表示される。

.. figure:: ./images/RunUnitTestIdeErrorJunit.png

.. note:: **テストの失敗について**

    テストの失敗には大きく分けて2種類の要因がある。
    1つはアサーション時に失敗する（Failure）場合、
    もう1つはテストの実装や、実行環境などに不備があり実行が失敗する（Error）場合である。
    IDEでJUnitを実行した際の実行結果画面には、これら2種類の結果の違いを示すだけでなく、
    スタックトレースも合わせて表示されるため、何が原因でテストが失敗したのかを分析する際に役立つ。

|

プロジェクト、メソッド単位で実行
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

JUnitの実行はプロジェクト単位、メソッド単位でも可能である。

プロジェクト単位で実行する場合は、テストしたいプロジェクトを右クリックしてメニューを表示させる。

.. figure:: ./images/RunUnitTestIdeClickTestProject.png

|

メニューから[Run As] -> [JUnit Test]を選択し、対象プロジェクトのテストを実行する。

.. figure:: ./images/RunUnitTestIdeRunProjectJunit.png

|

プロジェクト単位で実行した場合は、選択したプロジェクトに含まれる全テストクラスの実行結果が表示される。

.. figure:: ./images/RunUnitTestIdeSuccessProjectJunit.png

|

メソッド単位で実行する場合は、テストしたいメソッドを右クリックしてメニューを表示させる。

.. figure:: ./images/RunUnitTestIdeClickTestMethod.png

|

メニューから[Run As] -> [JUnit Test]を選択し、対象メソッドのテストを実行する。

.. figure:: ./images/RunUnitTestIdeRunMethodJunit.png

|

メソッド単位で実行した場合は、選択したメソッドの実行結果のみが表示される。

.. figure:: ./images/RunUnitTestIdeSuccessMethodJunit.png

Mavenでテストを実行
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

MavenでJUnitを実行する方法を紹介する。

テストフェーズの実行
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

MavenでJUnitを実行する場合は、対象プロジェクト配下に移動し以下のコマンドを実行する。

.. code-block:: console

    mvn test

コマンドを実行すると、\ ``target/classes``\ 配下にjavaコンパイルした.classファイルを作成したのち、
\ ``target/test-classes``\ 配下にコンパイルしたテスト用.classファイルを作成し、
\ ``target/surefire-reports``\ 配下にテスト結果が作成される。

デフォルトでは、以下のパターンにマッチするファイルが対象となりテストされる。

* \ ``**/Test*.java``\ 
* \ ``**/*Test.java``\ 
* \ ``**/*Tests.java``\ 
* \ ``**/*TestCase.java``\ 

上記パターンにマッチしないテストクラスを実行させたい場合は、
\ ``pom.xml``\ に設定を追加することで、テスト対象のファイルを変更することができる。
また、テストファイルの除外についても設定することが可能である。

* \ ``pom.xml``\

.. code-block:: xml

    <project>

      // ommited

      <build>
        <plugins>
          <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.20.1</version>
            <configuration>
              <includes>
                <include>*ServiceCheck.java</include> <!-- (1) -->
              </includes>
              <excludes>
                <exclude>AccountServiceCheck.java</exclude> <!-- (2) -->
              </excludes>
            </configuration>
          </plugin>
        </plugins>
      </build>

      // ommited

    </project>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | テスト実行時に実行対象となるファイルを設定する。
    * - | (2)
      - | テスト実行時に除外対象となるファイルを設定する。

.. note::

    設定する際には、正規表現を使って指定することもできる。
    詳細は \ `maven-surefire-plugin (Regular Expression Support) <https://maven.apache.org/surefire/maven-surefire-plugin/examples/inclusion-exclusion.html>`_\ を参照されたい。


コマンドオプションによる任意クラス、メソッドの指定
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ ``mvn test``\ コマンドはオプションを用いることで任意のクラス、メソッドを指定し実行することもできる。

テスト対象のクラスを指定する場合は、以下のコマンドを用いて指定できる。

.. code-block:: console

    mvn test -Dtest=[クラス名]

「,」 区切りで複数クラスを指定することもできる。

.. code-block:: console

    mvn test -Dtest=[クラス名],[クラス名],[クラス名]...

テスト対象のメソッドを指定したい場合は、以下のコマンドを用いて指定できる。

.. code-block:: console

    mvn test -Dtest=[クラス名]#[メソッド名]

クラス名には、FQCN指定（\ ``com.example.domain.repository.MemberRepositoryTest``\ など）と
単純クラス名での指定（\ ``MemberRepositoryTest``\ など）のどちらで指定してもよい。
また、クラス名にワイルドカード（\ ``Member*Test``\ など）を用いてパターン指定することもできる。

.. warning::

    メソッド単位の指定は \ ``maven-surefire-plugin``\ のバージョンが2.7.3以上必要となる。
    詳細は \ `maven-surefire-plugin (Running a Set of Methods in a Single Test Class) <http://maven.apache.org/surefire/maven-surefire-plugin/examples/single-test.html>`_\ を参照されたい。

.. note::

    オプションに \ ``-Dmaven.test.skip=true``\ を指定することでテストのコンパイル・実行をスキップすることができる。
    実行のみスキップしたい場合は、\ ``-DskipTests=true``\ を指定することでコンパイルのみ行われるようにすることもできる。
