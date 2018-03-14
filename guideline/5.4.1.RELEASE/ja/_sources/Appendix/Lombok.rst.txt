ボイラープレートコードの排除(Lombok)
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

.. _LombokAbout:

Lombokとは
--------------------------------------------------------------------------------

`Lombok <http://projectlombok.org/>`_ は、
Java言語におけるボイラープレートコードをソースコードから排除するために使用するライブラリである。

ボイラープレートコードとは、言語仕様上省く事ができない定型的なコードの事である。
ボイラープレートコードは本質的なロジックでないため、アプリケーションを実装する上で冗長なコードとなる。

Java言語における代表的なボイラープレートコードには、

* メンバー変数にアクセスするための getter / setter メソッド
* \ ``equals``\ /\ ``hashCode``\ メソッド
* \ ``toString``\ メソッド
* コンストラクタ
* リソース(入出力ストリーム等)のクローズ処理
* ロガーインスタンスの生成

等がある。

Lombokは、これらのボイラープレートコードをコンパイル時に生成することで、
開発者が実装するソースコード上から冗長なコードを取り除く仕組みを提供している。

.. tip::

    リソース(入出力ストリーム等)のクローズ処理については、Java SE7から追加されたtry-with-resources文を使う事で、
    ボイラープレートコードにならないように言語仕様が改善されている。

    Java言語自体もバージョンアップする毎に、冗長なコードを記載しなくて済むように改善されている。
    Java SE8からサポートされたラムダ式は、代表的な言語仕様の改善と言える。

|

.. _LombokEffect:

Lombokの効果
--------------------------------------------------------------------------------

以下に、Lombokを使用して作成したJavaBeanのソースコードを示す。

.. code-block:: java

    package com.example.domain.model;

    @lombok.Data
    public class User {

        private String userId;
        private String password;

    }

クラスレベルに\ ``@lombok.Data``\ アノテーションを付与するだけで、
JavaBeanとして必要なメソッドがLombokによって生成される。

.. figure:: ./images_Lombok/LombokGeneratedClassStructure.png
    :alt: Generated Class Structure by Lombok
    :align: center

    **Lombokによって生成されたクラス構造**

これは、Lombokの\ ``@Data``\ アノテーションを付与しただけで、約10行のソースコードから、
約60行ある下記のソースコード(Eclipseの自動生成機能を使用して出力したソースコード)によって生成されるクラスと同じ効果を得る事ができる事を意味している。

.. code-block:: java

    package com.example.domain.model;

    public class User {

        private String userId;
        private String password;

        public User() {
        }

        public String getUserId() {
            return userId;
        }

        public void setUserId(String userId) {
            this.userId = userId;
        }

        public String getPassword() {
            return password;
        }

        public void setPassword(String password) {
            this.password = password;
        }

        @Override
        public int hashCode() {
            final int prime = 31;
            int result = 1;
            result = prime * result
                    + ((password == null) ? 0 : password.hashCode());
            result = prime * result + ((userId == null) ? 0 : userId.hashCode());
            return result;
        }

        @Override
        public boolean equals(Object obj) {
            if (this == obj)
                return true;
            if (obj == null)
                return false;
            if (getClass() != obj.getClass())
                return false;
            User other = (User) obj;
            if (password == null) {
                if (other.password != null)
                    return false;
            } else if (!password.equals(other.password))
                return false;
            if (userId == null) {
                if (other.userId != null)
                    return false;
            } else if (!userId.equals(other.userId))
                return false;
            return true;
        }

        @Override
        public String toString() {
            return "User [userId=" + userId + ", password=" + password + "]";
        }

    }

|

.. _LombokSetup:

Lombokのセットアップ
--------------------------------------------------------------------------------

.. _LombokSetupAddDependency:

依存ライブラリの追加
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Lombokが提供しているクラスを使用するために、Lombokを依存ライブラリとして追加する。

.. code-block:: xml

    <!-- (1) -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <scope>provided</scope> <!-- (2) -->
    </dependency>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - Lombokを使用するプロジェクトの :file:`pom.xml` に、Lombokを依存ライブラリとして追加する。
    * - | (2)
      - Lombokはアプリケーション実行時には必要ないライブラリなので、スコープは\ ``provided``\ が適切である。

.. note::

    上記設定例は、依存ライブラリのバージョンを親プロジェクトである terasoluna-gfw-parent で管理する前提であるため、pom.xmlでのバージョンの指定は不要である。
    上記の依存ライブラリはterasoluna-gfw-parentが利用している\ `Spring IO Platform <http://platform.spring.io/platform/>`_\ で定義済みである。

|

.. _LombokSetupIDE:

IDE連携
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

LombokをIDE上で使用する場合は、IDEが提供するコンパイル(ビルド)機能と連携するために、
LombokをIDEにインストールする必要がある。

本ガイドラインでは、Spring Tool Suite(以降、「STS」と呼ぶ)にインストールする方法を紹介する。
使用するIDEによってインストール方法は異なるため、
STS以外のIDEを使用する場合は、 `こちらのページ <http://projectlombok.org/download.html>`_ を参考にされたい。

|

.. _LombokSetupIDEDownload:

Lombokのダウンロード
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Lombokのjarファイルをダウンロードする。

Lombokのjarファイルは、

* `Lombokのダウンロードページ <http://projectlombok.org/download.html>`_
* Mavenのローカルリポジトリ(通常は、:file:`${HOME}/.m2/repository/org/projectlombok/lombok/<version>/lombok-<version>.jar`)

から取得する。

|

.. _LombokSetupIDEInstall:

Lombokのインストール
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

ダウンロードしたLombokのjarファイルを実行(ダブルクリック)し、インストーラーを立ち上げる。

.. figure:: ./images_Lombok/LombokInstaller.png
    :alt: Lombok Installer
    :align: center

    **Lombokのインストーラー**

インストール対象のSTSを選択後、"Install / Update" ボタンを押下してインストールを実行する。
インストール候補のSTSは、インストーラーによって自動検出される仕組みになっているが、
自動で検出されない場合は、"Specify location ..."を押下してIDEを指定する必要がある。

.. figure:: ./images_Lombok/LombokInstallSuccessful.png
    :alt: Lombok Install Successful
    :align: center

    **インストール成功時のダイアログ**

Lombokをインストールした後にSTSを起動(又は再起動)すると、STS上でLombokを使用して開発する事ができる。

|

.. _LombokHowToUse:

Lombokの使用方法
--------------------------------------------------------------------------------

ここからは、Lombokの具体的な使い方について説明していく。

Lombokを初めて使用する場合は、まず、
Lombokの「 `Demo Video <http://projectlombok.org/>`_ 」を参照するとよい。
Demo Videoは4分弱で構成されており、最も基本的な使い方が説明されている。

|

.. _LombokHowToUseAnnotation:

Lombokが提供しているアノテーション
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

まず、Lombokが提供する代表的なアノテーションを紹介する。

各アノテーションの詳細な使用方法や、本ガイドラインで紹介していないアノテーションの使い方については、

* `Lombok features <http://projectlombok.org/features/index.html>`_

を参照されたい。

|

.. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 30 60

    * - 項番
      - アノテーション
      - 説明
    * - 1.
      - `@lombok.Getter <http://projectlombok.org/features/GetterSetter.html>`_
      - getterメソッドを生成するためのアノテーション。

        クラスレベルにアノテーションを指定すると、全てのフィールドにgetterメソッドを生成する事ができる。
    * - 2.
      - `@lombok.Setter <http://projectlombok.org/features/GetterSetter.html>`_
      - setterメソッドを生成するためのアノテーション。

        クラスレベルにアノテーションを指定すると、全ての非finalフィールドにsetterメソッドを生成する事ができる。
    * - 3.
      - `@lombok.ToString <http://projectlombok.org/features/ToString.html>`_
      - \ ``toString``\ メソッドを生成するためのアノテーション。
    * - 4.
      - `@lombok.EqualsAndHashCode <http://projectlombok.org/features/EqualsAndHashCode.html>`_
      - \ ``equals``\ と\ ``hashCode``\ メソッドを生成するためのアノテーション。
    * - 5.
      - `@lombok.RequiredArgsConstructor <http://projectlombok.org/features/Constructor.html>`_
      - 初期化が必要なフィールド(finalフィールドなど)の初期化パラメータを引数に持つコンストラクタを生成するためのアノテーション。

        全てのフィールドが任意のフィールドの場合は、デフォルトコンストラクタ(引数なしのコンストラクタ)が生成される。
    * - 6.
      - `@lombok.AllArgsConstructor <http://projectlombok.org/features/Constructor.html>`_
      - 全てのフィールドの初期化パラメータを引数に持つコンストラクタを生成するためのアノテーション。
    * - 7.
      - `@lombok.NoArgsConstructor <http://projectlombok.org/features/Constructor.html>`_
      - デフォルトコンストラクタを生成するためのアノテーション。
    * - 8.
      - `@lombok.Data <http://projectlombok.org/features/Data.html>`_
      - \ ``@Getter``\ 、 \ ``@Setter``\ 、 \ ``@ToString``\ 、 \ ``@EqualsAndHashCode``\ 、 \ ``@RequiredArgsConstructor``\ へのショートカットアノテーション。

        \ ``@Data``\ アノテーションを指定すると、上記5つのアノテーションを指定したのと同じ意味となる。
    * - 9.
      - `@lombok.extern.slf4j.Slf4j <http://projectlombok.org/features/Log.html>`_
      - SLF4Jのロガーインスタンスを生成するためのアノテーション。

|

.. _LombokHowToUseJavaBean:

JavaBeanの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

本ガイドラインが推奨する方法でアプリケーションを構築した場合、

* Formクラス
* Resourceクラス(REST API構築時)
* Entityクラス
* DTOクラス

などのJavaBeanを作成する必要がある。

以下に、JavaBeanの作成例を示す。

.. code-block:: java

    package com.example.domain.model;

    import lombok.Data;

    @Data // (1)
    public class User {

        private String userId;
        private String password;

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - クラスレベルに、\ ``@Data``\ アノテーションを指定し、

        * getter/setterメソッド
        * \ ``equals``\ / \ ``hashCode``\ メソッド
        * \ ``toString``\ メソッド
        * デフォルトコンストラクタ

        を生成する。

|

.. _LombokHowToUseJavaBeanExcludeToString:

toStringの対象から特定のフィールドを除外する方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

オブジェクトの状態を文字列に変換する際は、

* 相互参照関係をもつオブジェクトを保持するフィールド
* 個人情報やパスワードなどの機密情報を保持するフィールド

などを文字列変換の対象から除外する事が必要になるケースがある。
これらのフィールドを変換対象から除外しない場合、

* 前者は、循環参照となり\ ``StackOverflowError``\ や \ ``OutOfMemoryError``\ などが発生する
* 後者は、変換後の文字列の使用方法によっては、個人情報の漏洩に繋がる

可能性があるので、注意が必要である。

.. warning::

    JPAのEntityクラスに\ ``@Data``\ や\ ``@ToString``\ アノテーションを使用する場合は、
    循環参照になりやすいので特に注意が必要である。

|

以下に、特定のフィールドを文字列変換の対象から除外する方法を示す。

.. code-block:: java

    package com.example.domain.model;

    import lombok.Data;
    import lombok.ToString;

    @Data
    @ToString(exclude = "password") // (1)
    public class User {

        private String userId;
        private String password;

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - クラスレベルに\ ``@ToString``\ アノテーションを指定し、\ ``exclude``\ 属性に除外したいフィールド名を列挙する。

        上記例のソースコードから生成されたクラスの\ ``toString``\ メソッドを呼び出すと、

        * \ ``User(userId=U00001)``\

        という文字列に変換される。

|

.. _LombokHowToUseJavaBeanExcludeEqualsAndHashCode:

equalsとhashCodeの対象から特定のフィールドを除外する方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Lombokのアノテーションを使用して\ ``equals``\ メソッドと\ ``hashCode``\ メソッドを作成する場合は、
相互参照関係をもつオブジェクトを保持するフィールドを除外して生成する必要がある。

これらのフィールドを除外せずに生成した場合、
循環参照となり\ ``StackOverflowError``\ や \ ``OutOfMemoryError``\ などが発生するので、注意が必要である。

.. warning::

    JPAのEntityクラスに\ ``Data``\ アノテーション、\ ``Value``\ アノテーション、\ ``@EqualsAndHash``\を使用する場合は、
    循環参照になりやすいので特に注意が必要である。

|

以下に、特定のフィールドを除外する方法を示す。

.. code-block:: java

    package com.example.domain.model;

    import java.util.List;

    import lombok.Data;

    @Data
    public class Order {

        private String orderId;
        private List<OrderLine> orderLines;

    }

.. code-block:: java

    package com.example.domain.model;

    import lombok.Data;
    import lombok.EqualsAndHashCode;
    import lombok.ToString;

    @Data
    @ToString(exclude = "order")
    @EqualsAndHashCode(exclude = "order") // (1)
    public class OrderLine {

        private Order order;
        private String itemCode;
        private int quantity;

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - クラスレベルに\ ``@EqualsAndHashCode``\ アノテーションを指定し、\ ``exclude``\ 属性に除外したいフィールド名を列挙する。

.. tip::

    除外するフィールドを指定するのではなく、特定のフィールドのみを使用するように指定することもできる。

     .. code-block:: java

        @Data
        @ToString(exclude = "order")
        @EqualsAndHashCode(of = "itemCode") // (2)
        public class OrderLine {

            private final Order order;
            private final String itemCode;
            private final int quantity;

        }

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 10 90
        :class: longtable

        * - 項番
          - 説明
        * - | (2)
          - 特定のフィールドのみを使用する場合は、\ ``@EqualsAndHashCode``\ アノテーションの\ ``of``\ 属性に対象のフィールド名を列挙する。

            上記例では、\ ``itemCode``\ フィールドのみを参照して処理を行う\ ``equals``\ メソッドと\ ``hashCode``\ メソッドが生成される。


|

.. _LombokHowToUseJavaBeanConstructor:

フィールド初期化用のコンストラクタを生成する方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

アプリケーションの実装コードからJavaBeanのインスタンスを生成する場合は、
フィールドの初期値を引数に渡す事ができるコンストラクタがあった方が便利であり、
冗長なコードを排除することもできる。

デフォルトコンストラクタを使用してインスタンスを生成した場合は、以下のようなコードとなる。

.. code-block:: java

    public void login(String userId, String password) {
        User user = new User();
        user.setUserId(userId);
        user.setPassword(password);
        // ...
    }

|

以下に、フィールドの初期値を指定するコンストラクタを生成する方法を示す。

.. code-block:: java

    package com.example.domain.model;

    import lombok.AllArgsConstructor;
    import lombok.Data;
    import lombok.NoArgsConstructor;
    import lombok.ToString;

    @Data
    @AllArgsConstructor // (1)
    @NoArgsConstructor  // (2)
    @ToString(exclude = "password")
    public class User {

        private String userId;
        private String password;

    }

.. code-block:: java

    public void login(String userId, String password) {
        User user = new User(userId, password); // (3)
        // ...
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - クラスレベルに\ ``@AllArgsConstructor``\ アノテーションを指定し、全てのフィールドの初期値を引数にとるコンストラクタを生成する。
    * - | (2)
      - クラスレベルに\ ``@NoArgsConstructor``\ アノテーションを指定し、デフォルトコンストラクタを生成する。

        JavaBeanとして使用する場合は、デフォルトコンストラクタも生成しておく必要がある。
    * - | (3)
      - フィールドの初期値を指定するコンストラクタを呼び出し、JavaBeanのインスタンスを生成する。

        デフォルトコンストラクタを使用した場合は3ステップ必要だったものが、
        1ステップでインスタンスの生成が出来るようになった。

.. tip::

    上記例で扱っている\ ``User``\クラスを、JavaBeanではなく、Immutableなクラスにしたい場合は、
    \ ``@lombok.Value``\ アノテーションを使用するとよい。
    \ ``@Value``\ アノテーションについては、`Lombokのリファレンス <http://projectlombok.org/features/Value.html>`_ を参照されたい。


|

.. _LombokHowToUseLogger:

ロガーインスタンスの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

デバッグログやアプリケーションログを出力するために、ロガーインスタンスを生成する必要がある場合は、
ロガーインスタンスを生成するためのアノテーションを使用するとよい。

Lombokのアノテーションを使用しないでロガーインスタンスを作成する場合は、以下のようなコードになる。

.. code-block:: java

    package com.example.domain.service;

    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.stereotype.Service;

    @Service
    public class AuthenticationService {

        private static final Logger log = LoggerFactory.getLogger(AuthenticationService.class);

        public void login(String userId, String password) {
            log.info("{} had tried login.", userId);
            // ...
        }

    }

|

以下に、Lombokのアノテーションを使用してロガーインスタンスを作成する方法を示す。

.. code-block:: java

    package com.example.domain.service;

    import org.springframework.stereotype.Service;

    import lombok.extern.slf4j.Slf4j;

    @Slf4j // (1)
    @Service
    public class AuthenticationService {

        public void login(String userId, String password) {
            log.info("{} had tried login.", userId); // (2)
            // ...
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - クラスレベルに\ ``@Slf4j``\ アノテーションを指定し、SLF4Jのロガーインスタンスを生成する。

        本ガイドラインでは、SLF4Jの\ ``org.slf4j.Logger``\ を使用してログを出力する前提である。

        デフォルトでは、アノテーションを付与したクラスのFQCN(上記例だと\ ``com.example.domain.service.LoginService``\)がロガー名として使用され、
        ロガー名に対応するロガーインスタンスが\ ``log``\ という名前のフィールドに設定される。
    * - | (2)
      - Lombokによって生成されたSLF4Jのロガーインスタンスのメソッドを呼び出し、ログを出力する。

        上記例では、

        * \ ``11:29:45.838 [main] INFO  c.e.d.service.AuthenticationService - U00001 had tried login.``\

        というログが出力される。

.. tip::

    デフォルトで使用されるロガー名を変更したい場合は、
    \ ``@Slf4j``\ アノテーションの\ ``topic``\ 属性に、任意のロガー名を指定すればよい。

.. raw:: latex

   \newpage

