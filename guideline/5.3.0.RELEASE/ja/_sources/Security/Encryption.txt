暗号化
================================================================================

.. only:: html

 .. contents:: 目次
    :local:

.. _EncryptionOverview:

Overview
--------------------------------------------------------------------------------

個人情報やパスワードなどの機密情報は、以下のようなケースで暗号化が求められる。

* インターネットなどのネットワークを介して機密情報の送受信を行う
* データベースやファイルなどの外部リソースに機密情報を保存する

| Spring Securityの主機能は「認証」と「認可」であるが、暗号化に関する機能も提供している。
| ただし、提供される機能は限定的なものであるため、Spring Securityがサポートしていない暗号化方式については、個別に実装する必要がある。

本ガイドラインでは、以下の処理について説明を行う。

* Spring Securityが提供しているクラスを利用した共通鍵暗号化方式の暗号化と復号
* Spring Securityが提供しているクラスを利用した疑似乱数の生成
* JCA (Java Cryptography Architecture) を利用した公開鍵暗号化方式の暗号化と復号
* JCAを利用したハイブリッド暗号化方式の暗号化と復号

Spring Securityの暗号化機能の詳細については、\ `Spring Security Reference -Spring Security Crypto Module- <http://docs.spring.io/spring-security/site/docs/4.1.4.RELEASE/reference/htmlsingle/#crypto>`_\ を参照されたい。

.. _EncryptionOverviewEncryptionScheme:

暗号化方式
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
暗号化方式について説明する。

共通鍵暗号化方式
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 暗号化と復号を行う際に同じ鍵を使用する方式である。
| 復号に使用する鍵を暗号化側へ共有しておく方式であるため、鍵を暗号化側へ安全に受け渡す経路が別途必要となる。

公開鍵暗号化方式
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 復号側が用意した公開鍵を使用して暗号化し、公開鍵とペアとなる秘密鍵を使用して復号する方式である。
| 暗号文を復号する際に使用する秘密鍵は公開されないためセキュリティの強度は高いが、暗号化と復号処理のコストは高い。

ハイブリッド暗号化方式
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 共通鍵暗号化方式の処理コストが低いという利点と、公開鍵暗号化方式の鍵の管理・配布が容易でセキュリティ強度が高いという利点の両方を組み合わせた方式である。
| この方式はSSL/TLSなどで利用されている。

たとえば、HTTPS通信では、クライアント側で生成した共通鍵をサーバ側の公開鍵で暗号化したうえで送信し、サーバ側は公開鍵とペアとなる秘密鍵を利用して共通鍵を復号する。
その後の通信は、共有された共通鍵を使用した共通鍵暗号化方式で通信を行う。

この方式では、

* サイズが大きくなる可能性がある機密情報自体を、処理コストの低い共通鍵暗号化方式で暗号化
* サイズが小さく配布を安全に行う必要のある共通鍵を、セキュリティ強度の高い公開鍵暗号化方式で暗号化

するのがポイントである。
機密情報を復号する際に使用する共通鍵は秘密鍵によって守られているため、
公開鍵暗号化方式のセキュリティ強度を保ちつつ、公開鍵暗号化方式より高速な暗号化と復号処理を実現できる。

ハイブリッド暗号化方式における、暗号化から復号までの処理フローを以下の図に示す。

.. figure:: ./images_Encryption/EncryptionHybrid.png
   :alt: Hybrid Encryption
   :width: 100%

1. 送信側が平文を暗号化するための共通鍵を生成する。
2. 送信側が生成した共通鍵で平文を暗号化する。
3. 送信側が受信側の公開鍵で共通鍵を暗号化する。
4. 送信側が暗号化した共通鍵とともに暗号文を送信する。
5. 受信側が暗号化された共通鍵を受信側の秘密鍵で復号する。
6. 受信側が復号した共通鍵で暗号文を復号する。

|

.. _EncryptionOverviewEncryptionAlgorithm:

暗号化アルゴリズム
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
暗号化アルゴリズムについて説明する。

DES / 3DES
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| DES (Data Encryption Standard) は共通暗号化方式のアルゴリズムとして、アメリカ合衆国の標準規格として規格化されたものである。鍵長が56ビットと短いため現在では推奨されていない。
| 3DES (トリプルDES) は、鍵を変えながらDESを繰り返す暗号化アルゴリズムである。

.. _EncryptionOverviewEncryptionAlgorithmAes:

AES
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| AES (Advanced Encryption Standard) は共通鍵暗号化方式のアルゴリズムである。DESの後継として制定された暗号化規格であり、暗号化における現在のデファクトスタンダードとして利用されている。
| また、ブロック長より長いメッセージを暗号化するメカニズムである暗号利用モードとしてECB (Electronic Codebook) 、CBC (Cipher Block Chaining) 、OFB (Output Feedback) など存在する。その中で、最も広く利用されているものはCBCである。

.. note:: **AES with GCM**

  GCM (Galois/Counter Mode) という、並列処理が可能でありCBCより処理効率が優れていると一般的にいわれている暗号利用モードをAESで利用することも可能である。


RSA
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| RSAは公開鍵暗号化方式のアルゴリズムである。素因数分解の困難性に基づいているため、計算機の能力向上により危殆化することとなる。いわゆる「暗号化アルゴリズムの2010年問題」として指摘されているように充分な鍵長が必要であり、現時点では2048ビットが標準的に利用されている。

DSA / ECDSA
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| DSA (Digital Signature Algorithm) は、デジタル署名のための標準規格である。離散対数問題の困難性に基づいている。
| ECDSA (Elliptic Curve Digital Signature Algorithm : 楕円曲線DSA) は、楕円曲線暗号を用いたDSAの変種である。楕円曲線暗号においては、セキュリティレベルを確保するために必要となる鍵長が短くなるというメリットがある。

.. _EncryptionOverviewPseudoRandomNumber:

疑似乱数 (生成器)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 鍵の生成などで乱数が用いられる。
| このとき、乱数として生成される値が予測可能だと暗号化の安全性が保てなくなるため、結果の予測が困難な乱数 (疑似乱数) を利用する必要がある。
| 疑似乱数の生成に用いられるのが疑似乱数生成器である。

.. _EncryptionOverviewCipher:

javax.crypto.Cipherクラス
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| \ ``Cipher``\ クラスは、暗号化および復号の機能を提供する。AESやRSAなどの暗号化アルゴリズム、ECBやCBCなどの暗号利用モード、PKCS1などのパディング方式の組み合わせを指定する。
| 
| 暗号利用モードとは、\ :ref:`EncryptionOverviewEncryptionAlgorithmAes`\ で説明したとおり、ブロック長より長いメッセージを暗号化するメカニズムである。
| また、パディング方式とは、ブロック長に満たない暗号化対象を暗号化する場合の保管方式である。
| 
| Javaアプリケーションでは、\ ``"<暗号化アルゴリズム>/<暗号利用モード>/<パディング方式>"``\ または、\ ``"<暗号化アルゴリズム>"``\ という形で組み合わせを指定する。たとえば、\ ``"AES/CBC/PKCS5Padding"``\ または、\ ``"RSA"``\ となる。
  詳細は、\ `CipherクラスのJavaDoc <https://docs.oracle.com/javase/8/docs/api/javax/crypto/Cipher.html>`_\ を参照されたい。

.. _EncryptionOverviewSpringSecurity:

Spring Securityにおける暗号化機能
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Spring Securityでは、共通鍵暗号化方式を使用した暗号化および復号の機能を提供している。
| 暗号化アルゴリズムは256-bit AES using PKCS #5's PBKDF2 (Password-Based Key Derivation Function #2) である。
| 暗号利用モードはCBC、パディング方式はPKCS5Paddingである。

暗号化・復号用のコンポーネント
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityは、共通鍵暗号化方式での暗号化および復号の機能として以下のインターフェイスを提供している。

* \ ``org.springframework.security.crypto.encrypt.TextEncryptor``\  (テキスト用)
* \ ``org.springframework.security.crypto.encrypt.BytesEncryptor``\  (バイト配列用)

また、これらのインターフェイスの実装クラスとして以下のクラスを提供しており、内部では\ ``Cipher``\ クラスを利用している。

* \ ``org.springframework.security.crypto.encrypt.HexEncodingTextEncryptor``\  (テキスト用)
* \ ``org.springframework.security.crypto.encrypt.AesBytesEncryptor``\  (バイト配列用)


乱数生成用のコンポーネント
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityは、乱数(鍵)生成の機能として以下のインターフェイスを提供している。

* \ ``org.springframework.security.crypto.keygen.StringKeyGenerator``\  (テキスト用)
* \ ``org.springframework.security.crypto.keygen.BytesKeyGenerator``\  (バイト配列用)

また、これらのインターフェイスの実装クラスとして以下のクラスを提供している。

* \ ``org.springframework.security.crypto.keygen.HexEncodingStringKeyGenerator``\  (テキスト用)
* \ ``org.springframework.security.crypto.keygen.SecureRandomBytesKeyGenerator``\  (バイト配列用。\ ``generateKey``\ メソッドで、異なる鍵長を生成して返却)
* \ ``org.springframework.security.crypto.keygen.SharedKeyGenerator``\  (バイト配列用。\ ``generateKey``\ メソッドで、コンストラクタで設定した同一の鍵長を返却)


.. note:: **Spring Security RSA**

   \ `spring-security-rsa <https://github.com/dsyer/spring-security-rsa>`_\ は、暗号化アルゴリズムとしてRSAを使用した公開鍵暗号化方式とハイブリッド暗号化方式用のAPIを提供している。
   spring-security-rsaは現在、\ Springの公式リポジトリ <https://github.com/spring-projects>_\ として管理されていない。今後、Springの公式リポジトリ配下に移動した際は、本ガイドラインで利用方法を説明する予定である。

   spring-security-rsaでは以下２つのクラスを提供している。

   * \ ``org.springframework.security.crypto.encrypt.RsaRawEncryptor``\ 

     公開鍵暗号化方式を使用した暗号化および復号の機能を提供するクラス。

   * \ ``org.springframework.security.crypto.encrypt.RsaSecretEncryptor``\ 

     ハイブリッド暗号化方式を使用した暗号化および復号の機能を提供するクラス。

|

.. _EncryptionHowToUse:

How to use
--------------------------------------------------------------------------------

Oracleなど、一部のJava製品ではAESの鍵長256ビットを扱うためには、強度が無制限のJCE管轄ポリシーファイルを適用する必要がある。

.. note:: **JCE管轄ポリシーファイル**

   輸入規制の関係上、一部のJava製品ではデフォルトの暗号化アルゴリズム強度が制限されている。より強力なアルゴリズムを利用する場合は、強度が無制限のJCE管轄ポリシーファイルを入手し、JDK/JREにインストールする必要がある。詳細については、\ `Java Cryptography Architecture Oracle Providers Documentation <https://docs.oracle.com/javase/8/docs/technotes/guides/security/SunProviders.html>`_\を参照されたい。

   JCE管轄ポリシーファイルのダウンロード先

   * \ `Oracle Java 8 用 <http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html>`_\
   * \ `Oracle Java 7 用 <http://www.oracle.com/technetwork/java/embedded/embedded-se/downloads/jce-7-download-432124.html>`_\

.. _EncryptionHowToUseCommonKey:

共通鍵暗号化方式
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 暗号化アルゴリズムとしてAESを利用した方法について説明する。

文字列の暗号化
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

- テキスト（文字列）を暗号化する。

  .. code-block:: java

    public static String encryptText(
        String secret, String salt, String plainText) {
        TextEncryptor encryptor = Encryptors.text(secret, salt); // (1)

        return encryptor.encrypt(plainText); // (2)
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 共通鍵とソルトを指定して\ ``Encryptors#text``\ メソッドを呼び出し、\ ``TextEncryptor``\ クラスのインスタンスを生成する。
         | 生成したインスタンスの初期化ベクトルがランダムであるため、暗号化の際に異なる結果を返す。なお、暗号利用モードはCBCとなる。
         | このときに指定した共通鍵とソルトは、復号時にも同じものを利用する。

     * - | (2)
       - | 平文を\ ``encrypt``\ メソッドで暗号化する。

  .. note:: **暗号化の結果について**

    \ ``encrypt``\ メソッドの返り値 (暗号化の結果) は実行毎に異なる値を返すが、
    鍵とソルトが同一であれば復号処理の結果は同一になる (正しく復号できる) 。

| 

- 同一の暗号化結果を取得する。

  この方法は、暗号化した結果を用いてデータベースの検索を行うようなケースで利用できる。
  ただし、セキュリティ強度が落ちる点を踏まえ、使用の可否を検討してほしい。

  .. code-block:: java

    public static void encryptTextResult(
        String secret, String salt, String plainText) {
        TextEncryptor encryptor = Encryptors.queryableText(secret, salt); // (1)
        System.out.println(encryptor.encrypt(plainText)); // (2)
        System.out.println(encryptor.encrypt(plainText)); //
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 暗号化した結果として同じ値が必要な場合は、\ ``Encryptors#queryableText``\ メソッドを利用して\ ``TextEncryptor``\ クラスのインスタンスを生成する。
     * - | (2)
       - | \ ``Encryptors#queryableText``\ メソッドで生成したインスタンスは、\ ``encrypt``\ メソッドでの暗号化の結果として同一の値を返す。

| 

- GCMを用いたAESを使用してテキスト（文字列）を暗号化する。

  GCMを用いたAESはSpring Security4.0.2以降で利用可能である。\ :ref:`EncryptionOverviewEncryptionAlgorithmAes`\ で説明したとおり、CBCより処理効率が良い。

  .. code-block:: java

    public static String encryptTextByAesWithGcm(String secret, String salt, String plainText) {
        TextEncryptor aesTextEncryptor = Encryptors.delux(secret, salt); // (1)

        return aesTextEncryptor.encrypt(plainText); // (2)
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 共通鍵とソルトを指定して\ ``Encryptors#delux``\ メソッドを呼び出し、\ ``TextEncryptor``\ クラスのインスタンスを生成する。
         | このときに指定する共通鍵とソルトは、復号時にも同じものを利用する。

     * - | (2)
       - | 平文を\ ``encrypt``\ メソッドで暗号化する。

  .. note:: **GCMを用いたAESへのJavaの対応状況**

    GCMを用いたAESはJava SE8以降で使用可能である。詳細については、\ `JDK 8セキュリティの拡張機能 <http://docs.oracle.com/javase/jp/8/docs/technotes/guides/security/enhancements-8.html>`_\を参照されたい。

|

文字列の復号
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

- テキスト（文字列）の暗号文を復号する。

  .. code-block:: java

    public static String decryptText(String secret, String salt, String cipherText) {
        TextEncryptor decryptor = Encryptors.text(secret, salt); // (1)

        return decryptor.decrypt(cipherText); // (2)
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 共通鍵とソルトを指定して\ ``Encryptors#text``\ メソッドを呼び出し、\ ``TextEncryptor``\ クラスのインスタンスを生成する。
         | 共通鍵とソルトは、暗号化した際に利用したものを指定する。

     * - | (2)
       - | 暗号文を\ ``decrypt``\ メソッドで復号する。

|

- GCMを用いたAESを使用してテキスト（文字列）の暗号文を復号する。

  .. code-block:: java

    public static String decryptTextByAesWithGcm(String secret, String salt, String cipherText) {
        TextEncryptor aesTextEncryptor = Encryptors.delux(secret, salt); // (1)

        return aesTextEncryptor.decrypt(cipherText); // (2)
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 共通鍵とソルトを指定して\ ``Encryptors#delux``\ メソッドを呼び出し、\ ``TextEncryptor``\ クラスのインスタンスを生成する。
         | 共通鍵とソルトは、暗号化した際に利用したものを指定する。

     * - | (2)
       - | 暗号文を\ ``decrypt``\ メソッドで復号する。

|

バイト配列の暗号化
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

- バイト配列を暗号化する。

  .. code-block:: java

    public static byte[] encryptBytes(String secret, String salt, byte[] plainBytes) {
        BytesEncryptor encryptor = Encryptors.standard(secret, salt); // (1)

        return encryptor.encrypt(plainBytes); // (2)
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 共通鍵とソルトを指定して\ ``Encryptors#standard``\ メソッドを呼び出し、\ ``BytesEncryptor``\ クラスのインスタンスを生成する。
         | このときに指定した共通鍵とソルトは、復号時にも同じものを利用する。

     * - | (2)
       - | バイト配列の平文を\ ``encrypt``\ メソッドで暗号化する。

|

- GCMを用いたAESを使用してバイト配列を暗号化する。

  .. code-block:: java

    public static byte[] encryptBytesByAesWithGcm(String secret, String salt, byte[] plainBytes) {
        BytesEncryptor aesBytesEncryptor = Encryptors.stronger(secret, salt); // (1)

        return aesBytesEncryptor.encrypt(plainBytes); // (2)
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 共通鍵とソルトを指定して\ ``Encryptors#stronger``\ メソッドを呼び出し、\ ``BytesEncryptor``\ クラスのインスタンスを生成する。
         | このときに指定した共通鍵とソルトは、復号時にも同じものを利用する。

     * - | (2)
       - | バイト配列の平文を\ ``encrypt``\ メソッドで暗号化する。

|

バイト配列の復号
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

- バイト配列の暗号文を復号する。

  .. code-block:: java

    public static byte[] decryptBytes(String secret, String salt, byte[] cipherBytes) {
        BytesEncryptor decryptor = Encryptors.standard(secret, salt); // (1)

        return decryptor.decrypt(cipherBytes); // (2)
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 共通鍵とソルトを指定して\ ``Encryptors#standard``\ メソッドを呼び出し、\ ``BytesEncryptor``\ クラスのインスタンスを生成する。
         | 共通鍵とソルトは、暗号化した際に利用したものを指定する。

     * - | (2)
       - | バイト配列の暗号文を\ ``decrypt``\ メソッドで復号する。

|

- GCMを用いたAESによりバイト配列を復号する。

  .. code-block:: java

    public static byte[] decryptBytesByAesWithGcm(String secret, String salt, byte[] cipherBytes) {
        BytesEncryptor aesBytesEncryptor = Encryptors.stronger(secret, salt); // (1)

        return aesBytesEncryptor.decrypt(cipherBytes); // (2)
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 共通鍵とソルトを指定して\ ``Encryptors#stronger``\ メソッドを呼び出し、\ ``BytesEncryptor``\ クラスのインスタンスを生成する。
         | 共通鍵とソルトは、暗号化した際に利用したものを指定する。

     * - | (2)
       - | バイト配列の暗号文を\ ``decrypt``\ メソッドで復号する。

|

.. _EncryptionHowToUsePublicKey:

公開鍵暗号化方式
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| Spring Securityでは公開鍵暗号化方式に関する機能は提供されていないため、JCAおよびOpenSSLを利用した方法をサンプルコードを用いて説明する。

事前準備（JCAによるキーペアの生成）
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

- JCAでキーペア(公開鍵 / 秘密鍵の組み合わせ)を生成し、公開鍵で暗号化、秘密鍵で復号処理を行う。

  .. code-block:: java

    public void generateKeysByJCA() {
        try {
            KeyPairGenerator generator = KeyPairGenerator.getInstance("RSA"); // (1)
            generator.initialize(2048); // (2)
            KeyPair keyPair = generator.generateKeyPair(); // (3)
            PublicKey publicKey = keyPair.getPublic();
            PrivateKey privateKey = keyPair.getPrivate();

            byte[] cipherBytes = encryptByPublicKey("Hello World!", publicKey);  // (4)
            String plainText = decryptByPrivateKey(cipherBytes, privateKey); // (5)
            System.out.println(plainText);
        } catch (NoSuchAlgorithmException e) {
            throw new SystemException("e.xx.xx.9002", "No Such setting error.", e);
        }
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | RSAアルゴリズムを指定して\ ``KeyPairGenerator``\ クラスのインスタンスを生成する。

     * - | (2)
       - | 鍵長として2048ビットを指定する。

     * - | (3)
       - | キーペアを生成する。

     * - | (4)
       - | 公開鍵を利用して暗号化処理を行う。処理内容は後述する。

     * - | (5)
       - | 秘密鍵を利用して復号処理を行う。処理内容は後述する。

  .. note:: **暗号化したデータを文字列として扱いたい場合**

    外部システム連携等、暗号化したデータを文字列でやり取りしたい場合は、1つの手段としてBase64エンコードが挙げられる。Java SE8以降の場合は、Java標準の\ ``java.util.Base64``\ を使用する。それ以前の場合は、Spring Securityの\ ``org.springframework.security.crypto.codec.Base64``\ を使用する。

    Base64エンコードおよびデコードする方法をJava標準の\ ``java.util.Base64``\ を使用して説明する。
    
   * Base64エンコード

    .. code-block:: java

            // omitted
            byte[] cipherBytes = encryptByPublicKey("Hello World!", publicKey);  // 暗号化処理
            String cipherString = Base64.getEncoder().encodeToString(cipherBytes);  // バイト配列の暗号文を文字列に変換
            // omitted

   * Base64デコード

    .. code-block:: java

            // omitted
            byte[] cipherBytes = Base64.getDecoder().decode(cipherString); // 文字列の暗号文をバイト配列に変換
            String plainText = decryptByPrivateKey(cipherBytes, privateKey); // 復号処理
            // omitted

|

暗号化
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

- 公開鍵を利用して文字列を暗号化する。

  .. code-block:: java

    public byte[] encryptByPublicKey(String plainText, PublicKey publicKey) {
        try {
            Cipher cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding"); // (1)
            cipher.init(Cipher.ENCRYPT_MODE, publicKey);                       // (2)
            return cipher.doFinal(plainText.getBytes(StandardCharsets.UTF_8)); //
        } catch (NoSuchAlgorithmException | NoSuchPaddingException e) {
            throw new SystemException("e.xx.xx.9002", "No Such setting error.", e);
        } catch (InvalidKeyException |
                 IllegalBlockSizeException |
                 BadPaddingException e) {
            throw new SystemException("e.xx.xx.9003", "Invalid setting error.", e);
        }
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 暗号化アルゴリズム、暗号利用モード、パディング方式を指定して、\ ``Cipher``\ クラスのインスタンスを生成する。

     * - | (2)
       - | 暗号化処理を実行する。

|

復号
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

- 秘密鍵を利用してバイト配列を復号する。

  .. code-block:: java

    public String decryptByPrivateKey(byte[] cipherBytes, PrivateKey privateKey) {
        try {
            Cipher cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding"); // (1)
            cipher.init(Cipher.DECRYPT_MODE, privateKey);           // (2)
            byte[] plainBytes = cipher.doFinal(cipherBytes); //
            return new String(plainBytes, StandardCharsets.UTF_8);
        } catch (NoSuchAlgorithmException | NoSuchPaddingException e) {
            throw new SystemException("e.xx.xx.9002", "No Such setting error.", e);
        } catch (InvalidKeyException |
                 IllegalBlockSizeException |
                 BadPaddingException e) {
            throw new SystemException("e.xx.xx.9003", "Invalid setting error.", e);
        }
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 暗号化アルゴリズム、暗号利用モード、パディング方式を指定して、\ ``Cipher``\ クラスのインスタンスを生成する。

     * - | (2)
       - | 復号処理を実行する。

|

OpenSSL
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Cipherが同一であれば、公開鍵暗号化方式は別の方法で暗号化および復号を行うことが可能である。
| ここでは、OpenSSLを利用してあらかじめキーペアを作成しておき、その公開鍵を利用してJCAによる暗号化を行う。
  そして、その秘密鍵を利用してOpenSSLで復号処理を行う方法を説明する。

.. note:: **OpenSSL**

   OpenSSLでキーペアを作成する際はソフトウェアをインストールしておく必要がある。下記サイトよりダウンロードできる。

   OpenSSLのダウンロード先

   * \ `Linux 用 <https://www.openssl.org/source/>`_\
   * \ `Windows 用 <http://slproweb.com/products/Win32OpenSSL.html>`_\

|

- 事前準備として、OpenSSLでキーペアを作成する。

  .. code-block:: console

     $ openssl genrsa -out private.pem 2048  # (1)

     $ openssl pkcs8 -topk8 -nocrypt -in private.pem -out private.pk8 -outform DER  # (2)

     $ openssl rsa -pubout -in private.pem -out public.der -outform DER  # (3)

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | OpenSSLで2048ビットの秘密鍵 (DER形式) を生成する。

     * - | (2)
       - | Javaアプリケーションから読み込むために、秘密鍵をPKCS #8形式に変換する。

     * - | (3)
       - | 秘密鍵から公開鍵 (DER形式) を生成する。

|

- アプリケーションではOpenSSLで作成した公開鍵を読み込み、読み込んだ公開鍵を利用して暗号化処理を行う。

  .. code-block:: java

    public void useOpenSSLDecryption() {
        try {
            KeySpec publicKeySpec = new X509EncodedKeySpec(
                    Files.readAllBytes(Paths.get("public.der"))); // (1)
            KeyFactory keyFactory = KeyFactory.getInstance("RSA");
            PublicKey publicKey = keyFactory.generatePublic(publicKeySpec); // (2)

            byte[] cipherBytes = encryptByPublicKey("Hello World!", publicKey); // (3)

            Files.write(Paths.get("encryptedByJCA.txt"), cipherBytes);
            System.out.println("Please execute the following command:");
            System.out
                    .println("openssl rsautl -decrypt -inkey hoge.pem -in encryptedByJCA.txt");
        } catch (IOException e) {
            throw new SystemException("e.xx.xx.9001", "input/output error.", e);
        } catch (NoSuchAlgorithmException e) {
            throw new SystemException("e.xx.xx.9002", "No Such setting error.", e);
        } catch (InvalidKeySpecException e) {
            throw new SystemException("e.xx.xx.9003", "Invalid setting error.", e);
        }
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 公開鍵ファイルからバイナリデータを読み込む。

     * - | (2)
       - | バイナリデータから\ ``PublicKey``\ クラスのインスタンスを生成する。

     * - | (3)
       - | 公開鍵を利用して暗号化処理を行う。

|

- JCAで暗号化した内容がOpenSSLで復号できることを確認する。

  .. code-block:: console

     $ openssl rsautl -decrypt -inkey private.pem -in encryptedByJCA.txt  # (1)

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 秘密鍵を利用してOpenSSLで復号する。

|

| 続いて、OpenSSLで作成したキーペアを利用してOpenSSLで暗号化、JCAで復号する方法を説明する。

- OpenSSLのコマンドを使用して暗号化処理を行う。

  .. code-block:: console

     $ echo Hello | openssl rsautl -encrypt -keyform DER -pubin -inkey public.der -out encryptedByOpenSSL.txt  # (1)
     
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 公開鍵を利用してOpenSSLで暗号化する。

|

- アプリケーションではOpenSSLで作成した秘密鍵を読み込み、読み込んだ秘密鍵を利用して復号処理を行う。

  .. code-block:: java

    public void useOpenSSLEncryption() {
        try {
            KeySpec privateKeySpec = new PKCS8EncodedKeySpec(
                    Files.readAllBytes(Paths.get("private.pk8"))); // (1)
            KeyFactory keyFactory = KeyFactory.getInstance("RSA");
            PrivateKey privateKey = keyFactory.generatePrivate(privateKeySpec); // (2)

            String plainText = decryptByPrivateKey(
                   Files.readAllBytes(Paths.get("encryptedByOpenSSL.txt")),
                   privateKey); // (3)
            System.out.println(plainText);
        } catch (IOException e) {
            throw new SystemException("e.xx.xx.9001", "input/output error.", e);
        } catch (NoSuchAlgorithmException e) {
            throw new SystemException("e.xx.xx.9002", "No Such setting error.", e);
        } catch (InvalidKeySpecException e) {
            throw new SystemException("e.xx.xx.9003", "Invalid setting error.", e);
        }
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | PKCS #8形式の秘密鍵ファイルからバイナリデータを読み込み\ ``PKCS8EncodedKeySpec``\ クラスのインスタンスを生成する。

     * - | (2)
       - | \ ``KeyFactory``\ クラスから\ ``PrivateKey``\ クラスのインスタンスを生成する。

     * - | (3)
       - | 秘密鍵を利用して復号処理を行う。

|

.. _EncryptionHowToUseHybrid:

ハイブリッド暗号化方式
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| 公開鍵暗号化方式と同様、Spring Securityではハイブリッド暗号化方式に関する機能は提供されていないため、サンプルコードを用いて説明する。
| このサンプルコードは、spring-security-rsaの\ `RsaSecretEncryptorクラス <https://github.com/dsyer/spring-security-rsa/blob/1.0.1.RELEASE/src/main/java/org/springframework/security/rsa/crypto/RsaSecretEncryptor.java>`_\ を参考にしている。

暗号化
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

  .. code-block:: java

    public byte[] encrypt(byte[] plainBytes, PublicKey publicKey, String salt) {
        byte[] random = KeyGenerators.secureRandom(32).generateKey(); // (1)
        BytesEncryptor aes = Encryptors.standard(
                new String(Hex.encode(random)), salt); // (2)

        try (ByteArrayOutputStream result = new ByteArrayOutputStream()) {
            final Cipher cipher = Cipher.getInstance("RSA"); // (3)
            cipher.init(Cipher.ENCRYPT_MODE, publicKey); // (4)
            byte[] secret = cipher.doFinal(random); // (5)

            byte[] data = new byte[2]; // (6)
            data[0] = (byte) ((secret.length >> 8) & 0xFF); //
            data[1] = (byte) (secret.length & 0xFF); //
            result.write(data); //

            result.write(secret); // (7)
            result.write(aes.encrypt(plainBytes)); // (8)

            return result.toByteArray(); // (9)
        } catch (IOException e) {
            throw new SystemException("e.xx.xx.9001", "input/output error.", e);
        } catch (NoSuchAlgorithmException | NoSuchPaddingException e) {
            throw new SystemException("e.xx.xx.9002", "No Such setting error.", e);
        } catch (InvalidKeyException | IllegalBlockSizeException | BadPaddingException e) {
            throw new SystemException("e.xx.xx.9003", "Invalid setting error.", e);
        }
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 鍵長として32バイトを指定して\ ``KeyGenerators#secureRandom``\ メソッドを呼び出し、\ ``BytesKeyGenerator``\ クラスのインスタンスを生成する。
         | \ ``BytesKeyGenerator#generateKey``\ メソッドを呼び出し、共通鍵を生成する。
         | 詳細については、\ :ref:`EncryptionHowToUsePseudoRandomNumber`\ を参照されたい。

     * - | (2)
       - | 生成した共通鍵とソルトを指定して\ ``BytesEncryptor``\ クラスのインスタンスを生成する。

     * - | (3)
       - | 暗号化アルゴリズムとしてRSAを指定して、\ ``Cipher``\ クラスのインスタンスを生成する。

     * - | (4)
       - | 暗号化モード定数と公開鍵を指定して\ ``Cipher``\ クラスのインスタンスを初期化する。

     * - | (5)
       - | 共通鍵の暗号化処理を実行する。この暗号化処理は公開鍵暗号化方式となる。

     * - | (6)
       - | 暗号化した共通鍵の長さをバイト配列の暗号文に格納する。格納された共通鍵の長さは復号時に使用される。

     * - | (7)
       - | 暗号化した共通鍵をバイト配列の暗号文に格納する。

     * - | (8)
       - | 平文を暗号化してバイト配列の暗号文に格納する。この暗号化処理は共通鍵暗号化方式となる。

     * - | (9)
       - | バイト配列の暗号文を返却する。

|

復号
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

  .. code-block:: java

    public byte[] decrypt(byte[] cipherBytes, PrivateKey privateKey, String salt) {

        try (ByteArrayInputStream input = new ByteArrayInputStream(cipherBytes);
                ByteArrayOutputStream output = new ByteArrayOutputStream()) {
            byte[] b = new byte[2]; // (1)
            input.read(b); //
            int length = ((b[0] & 0xFF) << 8) | (b[1] & 0xFF); //

            byte[] random = new byte[length]; // (2)
            input.read(random); //
            final Cipher cipher = Cipher.getInstance("RSA"); // (3)
            cipher.init(Cipher.DECRYPT_MODE, privateKey); // (4)
            String secret = new String(Hex.encode(cipher.doFinal(random))); // (5)
            byte[] buffer = new byte[cipherBytes.length - random.length - 2]; // (6)
            input.read(buffer); //
            BytesEncryptor aes = Encryptors.standard(secret, salt); // (7)
            output.write(aes.decrypt(buffer)); // (8)

            return output.toByteArray(); // (9)
        } catch (IOException e) {
            throw new SystemException("e.xx.xx.9001", "input/output error.", e);
        } catch (NoSuchAlgorithmException | NoSuchPaddingException e) {
            throw new SystemException("e.xx.xx.9002", "No Such setting error.", e);
        } catch (InvalidKeyException | IllegalBlockSizeException | BadPaddingException e) {
            throw new SystemException("e.xx.xx.9003", "Invalid setting error.", e);
        }
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 暗号化された共通鍵の長さを取得する。

     * - | (2)
       - | 暗号化された共通鍵を取得する。

     * - | (3)
       - | 暗号化アルゴリズムとしてRSAを指定して、\ ``Cipher``\ クラスのインスタンスを生成する。

     * - | (4)
       - | 復号モード定数と秘密鍵を指定して\ ``Cipher``\ クラスのインスタンスを初期化する。

     * - | (5)
       - | 共通鍵の復号処理を実行する。この復号処理は公開鍵暗号化方式となる。

     * - | (6)
       - | 復号対象を取得する。

     * - | (7)
       - | 復号した共通鍵とソルトを指定して\ ``BytesEncryptor``\ クラスのインスタンスを生成する。

     * - | (8)
       - | 復号処理を実行する。この復号処理は共通鍵暗号化方式となる。

     * - | (9)
       - | 復号したバイト配列の平文を返却する。

|

.. _EncryptionHowToUsePseudoRandomNumber:

乱数生成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

文字列型の疑似乱数生成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

  .. code-block:: java

    public static void createStringKey() {
        StringKeyGenerator generator = KeyGenerators.string(); // (1)
        System.out.println(generator.generateKey()); // (2)
        System.out.println(generator.generateKey()); //
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 鍵 (疑似乱数) 生成器\ ``StringKeyGenerator``\ クラスのインスタンスを生成する。
         | この生成器で鍵を生成すると、毎回異なる値となる。
         |
         | 鍵長は指定できず、常に8バイトの鍵が生成される。

     * - | (2)
       - | \ ``generateKey``\ メソッドで鍵 (疑似乱数) を生成する。

|

バイト配列型の疑似乱数生成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

- 異なる鍵を生成する。

  .. code-block:: java

    public static void createDifferentBytesKey() {
        BytesKeyGenerator generator = KeyGenerators.secureRandom(); // (1)
        System.out.println(Arrays.toString(generator.generateKey())); // (2)
        System.out.println(Arrays.toString(generator.generateKey())); //
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | \ ``KeyGenerators#secureRandom``\ メソッドを呼び出し、鍵 (疑似乱数) 生成器\ ``BytesKeyGenerator``\ クラスのインスタンスを生成する。
         | この生成器で鍵を生成すると、毎回異なる値となる。
         |
         | 鍵長を指定しない場合、デフォルトで8バイトの鍵が生成される。

     * - | (2)
       - | \ ``generateKey``\ メソッドで鍵を生成する。

|

- 同一の鍵を生成する。

  .. code-block:: java

    public static void createSameBytesKey() {
        BytesKeyGenerator generator = KeyGenerators.shared(32); // (1)
        System.out.println(Arrays.toString(generator.generateKey())); // (2)
        System.out.println(Arrays.toString(generator.generateKey())); //
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 鍵長として32バイトを指定して\ ``KeyGenerators#shared``\ メソッドを呼び出し、鍵 (疑似乱数) 生成器\ ``BytesKeyGenerator``\ クラスのインスタンスを生成する。
         | この生成器で鍵を生成すると、毎回同じ値となる。
         |
         | 鍵長の指定は必須である。

     * - | (2)
       - | \ ``generateKey``\ メソッドで鍵を生成する。

|

.. raw:: latex

   \newpage

