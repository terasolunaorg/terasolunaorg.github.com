文字列処理
--------------------------------------------------------------------------------

.. only:: html

 .. contents:: 目次
    :depth: 4
    :local:

|

Overview
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| Javaの文字列標準APIでは、日本語に特化した操作が少ない。
| 全角カタカナ/半角カタカナの変換や、半角カタカナのみで構成される文字列の判定を行う場合などは、
| 独自に処理を作りこむ必要がある。

| また、Javaでは全ての文字列をUnicodeで表現するが、
| Unicodeでは 𠮷 のような特殊文字は、サロゲートペアと呼ばれるchar型2つ（32ビット）で表される。
| このような文字を扱う場合にも予期せぬ挙動が起きぬよう、様々な文字を扱うことを考慮した実装を行う必要がある。

| 本ガイドラインでは、日本語を処理するケースを想定し、
| 一般的な文字列操作の例と、共通ライブラリによる日本語操作APIの提供を行う。

How to use
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

トリム
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 半角空白のトリム操作を行う場合、``String#trim`` メソッドを利用することも出来るが、前・後ろのみのトリム操作や、任意の文字列のトリム操作などの複雑な操作を行う場合は、Springから提供されている ``org.springframework.util.StringUtils`` を利用するとよい。
|
| 以下に例を示す。
|

.. code-block:: java

   String str = "  Hello World!!";

   StringUtils.trimWhitespace(str); // => "Hello World!!"

   StringUtils.trimLeadingCharacter(str, ' '); // => "Hello World!!"

   StringUtils.trimTrailingCharacter(str, '!'); // => "  Hello World"

.. note::
  | ``StringUtils#trimLeadingCharacter`` , ``StringUtils#trimTrailingCharacter`` の第1引数にサロゲートペアの文字列は指定しても挙動に変化はない。なお、第2引数はchar型のため、サロゲートペアを指定することは出来ない。

パディング・サプレス
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| パディング（文字列埋め）操作・サプレス（文字列取り）操作を行う場合は、
| ``String`` クラスから提供されているメソッドで行うことが出来る。
|
| 以下に例を示す。

.. code-block:: java

   int num = 1;

   String paddingStr = String.format("%03d", num); // => "001"
   String suppressStr = paddingStr.replaceFirst("^0+(?!$)", ""); // => "1"

.. warning::
  | ``String#format`` はサロゲートペアを考慮できないため、見た目上の長さでパディングを行いたい場合、サロゲートペアが含まれると正しい結果が得られない。
  | サロゲートペアを考慮してパディングを実現するためには、後述するサロゲートペアを考慮した文字数のカウントを行い、パディングすべき正しい文字数を算出して文字列結合を行う必要がある。

サロゲートペアを考慮した文字列処理
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. _StringProcessingHowToGetSurrogatePairStringLength:

文字列長の取得
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| サロゲートペアを考慮した文字列の長さを取得する場合、
| 単に ``String#length`` メソッドを使用することは出来ない。
| サロゲートペアは32ビット（char型2つ）で表現されるため、見た目上の文字数よりも多くカウントされてしまう。
|
| 下記例では、変数 ``len`` には5が代入される。

.. code-block:: java

   String str = "𠮷田太郎";
   int len = str.length(); // => 5

|
| そこで、Java SE 5よりサロゲートペアを考慮した文字列の長さを取得するためのメソッド ``String#codePointCount`` が定義された。
| ``String#codePointCount`` の引数に、対象文字列の開始インデックスと終了インデックスを指定することで、文字列長を取得することが出来る。
|
| 以下に例を示す。

.. code-block:: java

   String str = "𠮷田太郎";
   int lenOfChar = str.length(); // => 5
   int lenOfCodePoint = str.codePointCount(0, lenOfChar); // => 4

|
| また、Unicodeでは結合文字が存在する。
| 「が」を表す ``\u304c`` と、「か」と「濁点」を表す ``\u304b\u3099`` は、見た目上の違いは存在しないが、「か」＋「濁点」の例は2文字としてカウントされてしまう。
| こうした結合文字が入力されることも考慮して文字数をカウントする場合、 ``java.text.Normalizer`` を使用してテキストの正規化を行ってからカウントする。
|
| 以下に、結合文字とサロゲートペアを考慮をした上で、文字列の長さを返却するメソッドを示す。

.. code-block:: java

   public int getStrLength(String str) {
     String normalizedStr  = Normalizer.normalize(str, Normalizer.Form.NFC);
     int length = normalizedStr.codePointCount(0, normalizedStr.length());

     return length;
   }


指定範囲の文字列取得
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| 指定範囲の文字列を取得する場合、単に ``String#substring`` を利用すると、想定していない結果になる可能性がある。

.. code-block:: java

   String str = "𠮷田 太郎";
   int startIndex = 0;
   int endIndex = 2;
   
   String subStr = str.substring(startIndex, endIndex);

   System.out.println(subStr); // => "𠮷"

| 上記の例では、0文字目（先頭）から2文字を取り出し、 「𠮷田」 を取得しようと試みているが、サロゲートペアは32ビット（char型2つ）で表現されるため「𠮷」しか取得できない。
| このような場合には、``String#offsetByCodePoints`` を利用し、サロゲートペアを考慮した開始位置と終了位置を求めてから ``String#substring`` メソッドを使う必要がある。
|
| 以下に、先頭から2文字（苗字部分）を取り出す例を示す。

.. code-block:: java

   String str = "𠮷田 太郎";
   int startIndex = 0;
   int endIndex = 2;

   int startIndexSurrogate = str.offsetByCodePoints(0, startIndex); // => 0
   int endIndexSurrogate = str.offsetByCodePoints(0, endIndex); // => 3

   String subStrSurrogate = str.substring(startIndexSurrogate, endIndexSurrogate); // => "𠮷田"

|

文字列分割
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| ``String#split`` メソッドは、サロゲートペアにデフォルトで対応している。
| 以下に例を示す。


.. code-block:: java

   String str = "𠮷田 太郎";
   
   str.split(" "); // => {"𠮷田", "太郎"}

|

    .. note::
      | サロゲートペアを区切り文字として、 ``String#split`` の引数に指定することも出来る。
      
    .. note::
      | Java SE 7までの環境とJava SE 8で、 ``String#split`` に空文字を渡したときの挙動に変化があるため留意されたい。 参照： `Pattern#splitのJavadoc <http://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html#split-java.lang.CharSequence-int->`_
      
      .. code-block:: java
      
        String str = "ABC";
        String[] elems = str.split("");
        
        // Java SE 7 => {, A, B, C}
        // Java SE 8 => {A, B, C}


.. _StringProcessingHowToUseFullHalfConverter:

全角・半角文字列変換
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

全角文字と半角文字の変換は、共通ライブラリが提供する\ ``org.terasoluna.gfw.common.fullhalf.FullHalfConverter``\ クラスのAPIを使用して行う。

\ ``FullHalfConverter``\ クラスは、変換対象にしたい全角文字と半角文字のペア定義(\ ``org.terasoluna.gfw.common.fullhalf.FullHalfPair``\ )を事前に登録しておくスタイルを採用している。
共通ライブラリでは、デフォルトのペア定義が登録されている\ ``FullHalfConverter``\ オブジェクトを、
\ ``org.terasoluna.gfw.common.fullhalf.DefaultFullHalf``\ クラスの\ ``INSTANCE``\ 定数として提供している。
デフォルトのペア定義については、`DefaultFullHalfのソース <https://github.com/terasolunaorg/terasoluna-gfw/blob/5.2.0.RELEASE/terasoluna-gfw-common-libraries/terasoluna-gfw-string/src/main/java/org/terasoluna/gfw/common/fullhalf/DefaultFullHalf.java>`_ を参照されたい。

.. note::

    共通ライブラリが提供しているデフォルトのペア定義で変換要件が満たせない場合は、独自のペア定義を登録した\ ``FullHalfConverter``\ オブジェクトを作成すればよい。
    具体的な作成方法については、:ref:`StringOperationsHowToUseCustomFullHalfConverter` を参照されたい。



共通ライブラリの適用方法
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| :ref:`StringProcessingHowToUseFullHalfConverter` を使う場合は、共通ライブラリを依存ライブラリとして以下の通り追加する必要がある。

.. code-block:: xml

   <dependencies>
       <dependency>
           <groupId>org.terasoluna.gfw</groupId>
           <artifactId>terasoluna-gfw-string</artifactId>
       </dependency>
   </dependencies>

.. note::

    上記設定例では、依存ライブラリのバージョンは親プロジェクトで管理する前提である。
    そのため、\ ``<version>``\ 要素は指定していない。

全角文字列に変換
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

半角文字を全角文字へ変換する場合は、\ ``FullHalfConverter``\ の\ ``toFullwidth``\ メソッドを使用する。

.. code-block:: java

   String fullwidth = DefaultFullHalf.INSTANCE.toFullwidth("ｱﾞ!A8ｶﾞザ");    // (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 半角文字が含まれる文字列を\ ``toFullwidth``\ メソッドの引数に渡し、全角文字列へ変換する。
       | 本例では、\ ``"ア゛！Ａ８ガザ"``\ に変換される。なお、ペア定義されていない文字（本例の\ ``"ザ"``\ ）はそのまま返却される。


半角文字列に変換
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

全角文字を半角文字へ変換する場合は、\ ``FullHalfConverter``\ の\ ``toHalfwidth``\ メソッドを使用する。

.. code-block:: java

   String halfwidth = DefaultFullHalf.INSTANCE.toHalfwidth("Ａ！アガｻ");    // (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | 全角文字が含まれる文字列を\ ``toHalfwidth``\ メソッドの引数に渡し、半角文字列へ変換する。
       | 本例では、\ ``"A!ｱｶﾞｻ"``\ に変換される。なお、ペア定義されていない文字（本例の\ ``"ｻ"``\ ）はそのまま返却される。

.. note::

    \ ``FullHalfConverter``\ は、2文字以上で1文字を表現する結合文字（例：「\ ``"シ"``\ (\ ``\u30b7``\ ) + 濁点(\ ``\u3099``\ )」）を半角文字（例：\ ``"ｼﾞ"``\ ）へ変換することが出来ない。
    結合文字を半角文字へ変換する場合は、テキスト正規化を行って合成文字（例：\ ``"ジ"``\ (\ ``\u30b8``\ )）に変換してから \ ``FullHalfConverter``\ を使用する必要がある。
    
    テキスト正規化を行う場合は、\ ``java.text.Normalizer``\ を使用する。
    なお、結合文字を合成文字に変換する場合は、正規化形式としてNFCまたはNFKCを利用する。


    正規化形式としてNFD（正準等価性によって分解する）を使用する場合の実装例
    
      .. code-block:: java

         String str1 = Normalizer.normalize("モジ", Normalizer.Form.NFD); // str1 = "モシ + Voiced sound mark(\u3099)"
         String str2 = Normalizer.normalize("ﾓｼﾞ", Normalizer.Form.NFD);  // str2 = "ﾓｼﾞ"

    正規化形式としてNFC（正準等価性によって分解し、再度合成する）を使用する場合の実装例
    
      .. code-block:: java

         String mojiStr = "モシ\u3099";                                   // "モシ + Voiced sound mark(\u3099)"
         String str1 = Normalizer.normalize(mojiStr, Normalizer.Form.NFC); // str1 = "モジ（\u30b8）"
         String str2 = Normalizer.normalize("ﾓｼﾞ", Normalizer.Form.NFC);   // str2 = "ﾓｼﾞ"
    
    正規化形式としてNFKD（互換等価性によって分解する）を使用する場合の実装例
    
      .. code-block:: java

         String str1 = Normalizer.normalize("モジ", Normalizer.Form.NFKD); // str1 = "モシ + Voiced sound mark(\u3099)"
         String str2 = Normalizer.normalize("ﾓｼﾞ", Normalizer.Form.NFKD);  // str2 = "モシ + Voiced sound mark(\u3099)"
    
    正規化形式としてNFKC（互換等価性によって分解し、再度合成する）を使用する場合の実装例
    
      .. code-block:: java

         String mojiStr = "モシ\u3099";                                    // "モシ + Voiced sound mark(\u3099)"
         String str1 = Normalizer.normalize(mojiStr, Normalizer.Form.NFKC); // str1 = "モジ（\u30b8）"
         String str2 = Normalizer.normalize("ﾓｼﾞ", Normalizer.Form.NFKC) ;  // str2 = "モジ"
    
    
    詳細は \ `NormalizerのJavaDoc <https://docs.oracle.com/javase/8/docs/api/java/text/Normalizer.html>`_\ を参照されたい。


.. _StringOperationsHowToUseCustomFullHalfConverter:

独自の全角文字と半角文字のペア定義を登録したFullHalfConverterクラスの作成
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| \ ``DefaultFullHalf``\ を使用せず、独自の全角文字と半角文字のペア定義を登録した\ ``FullHalfConverter``\ を使用することも出来る。
| 以下に、独自の全角文字と半角文字のペア定義を登録した \ ``FullHalfConverter``\ を使用する方法を示す。

**独自のペア定義を登録したFullHalfConverterを提供するクラスの実装例**

.. code-block:: java
 
    public class CustomFullHalf {
        
        private static final int FULL_HALF_CODE_DIFF = 0xFEE0;
        
        public static final FullHalfConverter INSTANCE;
        
        static {
            // (1)
            FullHalfPairsBuilder builder = new FullHalfPairsBuilder();
        
            // (2)
            builder.pair("ー", "-");
            
            // (3)
            for (char c = '!'; c <= '~'; c++) {
                String fullwidth = String.valueOf((char) (c + FULL_HALF_CODE_DIFF));
                builder.pair(fullwidth, String.valueOf(c));
            }
            
            // (4)
            builder.pair("。", "｡").pair("「", "｢").pair("」", "｣").pair("、", "､")
                    .pair("・", "･").pair("ァ", "ｧ").pair("ィ", "ｨ").pair("ゥ", "ｩ")
                    .pair("ェ", "ｪ").pair("ォ", "ｫ").pair("ャ", "ｬ").pair("ュ", "ｭ")
                    .pair("ョ", "ｮ").pair("ッ", "ｯ").pair("ア", "ｱ").pair("イ", "ｲ")
                    .pair("ウ", "ｳ").pair("エ", "ｴ").pair("オ", "ｵ").pair("カ", "ｶ")
                    .pair("キ", "ｷ").pair("ク", "ｸ").pair("ケ", "ｹ").pair("コ", "ｺ")
                    .pair("サ", "ｻ").pair("シ", "ｼ").pair("ス", "ｽ").pair("セ", "ｾ")
                    .pair("ソ", "ｿ").pair("タ", "ﾀ").pair("チ", "ﾁ").pair("ツ", "ﾂ")
                    .pair("テ", "ﾃ").pair("ト", "ﾄ").pair("ナ", "ﾅ").pair("ニ", "ﾆ")
                    .pair("ヌ", "ﾇ").pair("ネ", "ﾈ").pair("ノ", "ﾉ").pair("ハ", "ﾊ")
                    .pair("ヒ", "ﾋ").pair("フ", "ﾌ").pair("ヘ", "ﾍ").pair("ホ", "ﾎ")
                    .pair("マ", "ﾏ").pair("ミ", "ﾐ").pair("ム", "ﾑ").pair("メ", "ﾒ")
                    .pair("モ", "ﾓ").pair("ヤ", "ﾔ").pair("ユ", "ﾕ").pair("ヨ", "ﾖ")
                    .pair("ラ", "ﾗ").pair("リ", "ﾘ").pair("ル", "ﾙ").pair("レ", "ﾚ")
                    .pair("ロ", "ﾛ").pair("ワ", "ﾜ").pair("ヲ", "ｦ").pair("ン", "ﾝ")
                    .pair("ガ", "ｶﾞ").pair("ギ", "ｷﾞ").pair("グ", "ｸﾞ")
                    .pair("ゲ", "ｹﾞ").pair("ゴ", "ｺﾞ").pair("ザ", "ｻﾞ")
                    .pair("ジ", "ｼﾞ").pair("ズ", "ｽﾞ").pair("ゼ", "ｾﾞ")
                    .pair("ゾ", "ｿﾞ").pair("ダ", "ﾀﾞ").pair("ヂ", "ﾁﾞ")
                    .pair("ヅ", "ﾂﾞ").pair("デ", "ﾃﾞ").pair("ド", "ﾄﾞ")
                    .pair("バ", "ﾊﾞ").pair("ビ", "ﾋﾞ").pair("ブ", "ﾌﾞ")
                    .pair("べ", "ﾍﾞ").pair("ボ", "ﾎﾞ").pair("パ", "ﾊﾟ")
                    .pair("ピ", "ﾋﾟ").pair("プ", "ﾌﾟ").pair("ペ", "ﾍﾟ")
                    .pair("ポ", "ﾎﾟ").pair("ヴ", "ｳﾞ").pair("\u30f7", "ﾜﾞ")
                    .pair("\u30fa", "ｦﾞ").pair("゛", "ﾞ").pair("゜", "ﾟ").pair("　", " ");
            
            // (5)
            INSTANCE = new FullHalfConverter(builder.build());
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``org.terasoluna.gfw.common.fullhalf.FullHalfPairsBuilder``\ を使用して、全角文字と半角文字のペア定義のセットを表現する\ ``org.terasoluna.gfw.common.fullhalf.FullHalfPairs``\ を作成する。
    * - | (2)
      - | \ ``DefaultFullHalf``\ では、全角文字の\ ``"ー"``\ に対する半角文字を\ ``"ｰ"``\ (\ ``\uFF70``\ )に設定しているところを、本例では\ ``"-"``\ (\ ``\u002D``\ )に変更している。
        | なお、\ ``"-"``\ (\ ``\u002D``\ )は、下記(3)の処理対象にも含まれているが、先に定義したペア定義が優先される仕組みになっている。
    * - | (3)
      - | 本例では、Unicodeの全角の\ ``"！"``\ から\ ``"～"``\ までと半角の\ ``"!"``\ から\ ``"~"``\ までのコード値を、コード値の並び順が同じであるという特徴を利用して、ループ処理を使ってペア定義を行っている。
    * - | (4)
      - | 上記(3)以外の文字はコード値の並び順が全角文字と半角文字で一致しないため、それぞれ個別にペア定義を行う。
    * - | (5)
      - | \ ``FullHalfPairsBuilder``\ より作成した \ ``FullHalfPairs``\ を使用して、 \ ``FullHalfConverter``\ を作成する。

.. note::

    \ ``FullHalfPairsBuilder#pair``\ メソッドの引数に指定可能な値については、
    `FullHalfPairのコンストラクタのJavaDoc <https://github.com/terasolunaorg/terasoluna-gfw/blob/5.2.0.RELEASE/terasoluna-gfw-common-libraries/terasoluna-gfw-string/src/main/java/org/terasoluna/gfw/common/fullhalf/FullHalfPair.java>`_
    を参照されたい。

|

**独自のペア定義を登録したFullHalfConverterの使用例**

.. code-block:: java
 
    String halfwidth = CustomFullHalf.INSTANCE.toHalfwidth("ハローワールド！"); // (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 独自のペア定義が登録された \ ``FullHalfConverter``\ オブジェクトの\ ``toHalfwidth``\ メソッドを使用して、全角文字が含まれる文字列を半角文字列へ変換する。
        | 本例では、\ ``"ﾊﾛ-ﾜ-ﾙﾄﾞ!"``\ に変換される。（\ ``"-"``\ は \ ``\u002D``\ ）


|

.. _StringProcessingHowToUseCodePoints:

コードポイント集合チェック(文字種チェック)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

文字種チェックを行う場合は、共通ライブラリから提供しているコードポイント集合機能を使用してチェックするとよい。

ここでは、コードポイント集合機能を使用した文字種チェックの実装方法を説明する。

* :ref:`StringProcessingHowToUseCodePointsConstruction`
* :ref:`StringProcessingHowToUseCodePointsOperations`
* :ref:`StringProcessingHowToUseCodePointsCheck`
* :ref:`StringProcessingHowToUseCodePointsValidator`
* :ref:`StringProcessingHowToUseCodePointsClassCreation`


共通ライブラリの適用方法
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| :ref:`StringProcessingHowToUseCodePoints` を使う場合は、 :ref:`StringProcessingHowToUseCodePointsClasses` 等を依存ライブラリとして追加する必要がある。

.. _StringProcessingHowToUseCodePointsConstruction:

コードポイント集合の作成
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| \ ``org.terasoluna.gfw.common.codepoints.CodePoints``\ は、コードポイント集合を表現するクラスである。
| \ ``CodePoints``\ のインスタンスの作成方法を以下に示す。

**ファクトリメソッドを呼び出してインスタンスを作成する場合（キャッシュあり）**

| コードポイント集合クラス( \ ``Class<? extends CodePoints>``\ )からインスタンスを作成し、作成したインスタンスをキャッシュする方法を以下に示す。
| 特定のコードポイント集合は、複数回作成する必要はないため、この方法を使用してキャッシュすることを推奨する。

.. code-block:: java

   CodePoints codePoints = CodePoints.of(ASCIIPrintableChars.class);  // (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``CodePoints#of``\ メソッド(ファクトリメソッド)にコードポイント集合クラスを渡してインスタンスを取得する。
       | 本例では、 Ascii印字可能文字のコードポイント集合クラス(\ ``org.terasoluna.gfw.common.codepoints.catalog.ASCIIPrintableChars``\ )のインスタンスを取得している。

.. note::

    コードポイント集合クラスは、\ ``CodePoints``\ クラスと同じモジュール内に複数存在する。
    その他にもコードポイント集合を提供するモジュールが存在するが、それらのモジュールは必要に応じて自プロジェクトに追加する必要がある。
    詳細は、:ref:`StringProcessingHowToUseCodePointsClasses` を参照されたい。

    また、 新規にコードポイント集合クラスを作成することも出来る。
    詳細は、 :ref:`StringProcessingHowToUseCodePointsClassCreation` を参照されたい。

|

**コードポイント集合クラスのコンストラクタを呼び出してインスタンスを作成する場合**

| コードポイント集合クラスからインスタンスを作成する方法を以下に示す。
| この方法を使用した場合、作成したインスタンスはキャッシュされないため、キャッシュすべきでない処理（集合演算の引数等）で使用することを推奨する。

.. code-block:: java

   CodePoints codePoints = new ASCIIPrintableChars();  // (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``new``\ 演算子を使用してコンストラクタを呼び出し、コードポイント集合クラスのインスタンスを生成する。
       | 本例では、 Ascii印字可能文字のコードポイント集合クラス( \ ``ASCIIPrintableChars``\ )のインスタンスを生成している。

|

**CodePointsのコンストラクタを呼び出してインスタンスを作成する場合**

| \ ``CodePoints``\ からインスタンスを作成する方法を以下に示す。
| この方法を使用した場合、作成したインスタンスはキャッシュされないため、キャッシュすべきでない処理（集合演算の引数等）で使用することを推奨する。

* コードポイント( \ ``int``\ )を可変長引数を使用して渡す場合

  .. code-block:: java

      CodePoints codePoints = new CodePoints(0x0061 /* a */, 0x0062 /* b */);  // (1)

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - 項番
        - 説明
      * - | (1)
        - | \ ``int``\ のコードポイントを、\ ``CodePoints``\ のコンストラクタに渡してインスタンスを生成する。
          | 本例では、 文字\ ``"a"``\ と\ ``"b"``\ のコードポイント集合のインスタンスを生成している。

|

* コードポイント( \ ``int``\ )の \ ``Set``\ を渡す場合

  .. code-block:: java

      Set<Integet> set = new HashSet<>();
      set.add(0x0061 /* a */);
      set.add(0x0062 /* b */);
      CodePoints codePoints = new CodePoints(set);  // (1)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - 項番
        - 説明
      * - | (1)
        - | \ ``int``\ のコードポイントを \ ``Set``\ に追加し、\ ``Set``\ を \ ``CodePoints``\ のコンストラクタに渡してインスタンスを生成する。
          | 本例では、 文字\ ``"a"``\ と\ ``"b"``\ のコードポイント集合のインスタンスを生成している。

|

* コードポイント集合文字列を可変長引数を使用して渡す場合

  .. code-block:: java

      CodePoints codePoints = new CodePoints("ab");         // (1)

  .. code-block:: java

      CodePoints codePoints = new CodePoints("a", "b");  // (2)

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - 項番
        - 説明
      * - | (1)
        - | コードポイント集合文字列を \ ``CodePoints``\ のコンストラクタに渡してインスタンスを生成する。
          | 本例では、 文字\ ``"a"``\ と\ ``"b"``\ のコードポイント集合のインスタンスを生成している。
      * - | (2)
        - | コードポイント集合文字列を複数に分けて渡すことも出来る。(1)と同じ結果となる。

|

.. _StringProcessingHowToUseCodePointsOperations:

コードポイント集合同士の集合演算
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| コードポイント集合に対して集合演算を行い、新規のコードポイント集合のインスタンスを作成することが出来る。
| なお、集合演算によって元のコードポイント集合の状態が変更されることは無い。
| 集合演算を使用してコードポイント集合のインスタンスを作成する方法を以下に示す。


**和集合メソッドを使用してコードポイント集合のインスタンスを作成する場合**

.. code-block:: java

    CodePoints abCp = new CodePoints(0x0061 /* a */, 0x0062 /* b */);
    CodePoints cdCp = new CodePoints(0x0063 /* c */, 0x0064 /* d */);

    CodePoints abcdCp = abCp.union(cdCp);    // (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``CodePoints#union``\ メソッドを使用して２つのコードポイント集合の和集合を計算し、新規のコードポイント集合のインスタンスを作成する。
        | 本例では、「文字列\ ``"ab"``\ に含まれるコードポイント集合」と「文字列\ ``"cd"``\ に含まれるコードポイント集合」の和集合を計算し、新規のコードポイント集合（文字列\ ``"abcd"``\ に含まれるコードポイント集合）のインスタンスを作成している。

|

**差集合メソッドを使用してコードポイント集合のインスタンスを作成する場合**

.. code-block:: java

    CodePoints abcdCp = new CodePoints(0x0061 /* a */, 0x0062 /* b */,
            0x0063 /* c */, 0x0064 /* d */);
    CodePoints cdCp = new CodePoints(0x0063 /* c */, 0x0064 /* d */);

    CodePoints abCp = abcdCp.subtract(cdCp);    // (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``CodePoints#subtract``\ メソッドを使用して２つのコードポイント集合の差集合を計算し、新規のコードポイント集合のインスタンスを作成する。
        | 本例では、「文字列\ ``"abcd"``\ に含まれるコードポイント集合」と「文字列\ ``"cd"``\ に含まれるコードポイント集合」の差集合を計算し、新規のコードポイントの集合（文字列\ ``"ab"``\ に含まれるコードポイント集合）のインスタンスを作成している。

|

**積集合で新規のコードポイント集合のインスタンスを作成する場合**

.. code-block:: java

    CodePoints abcdCp = new CodePoints(0x0061 /* a */, 0x0062 /* b */,
            0x0063 /* c */, 0x0064 /* d */);
    CodePoints cdeCp = new CodePoints(0x0063 /* c */, 0x0064 /* d */, 0x0064 /* e */);

    CodePoints cdCp = abcdCp.intersect(cdeCp);    // (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``CodePoints#intersect``\ メソッドを使用して２つのコードポイント集合の積集合を計算し、新規のコードポイント集合のインスタンスを作成する。
        | 本例では、「文字列\ ``"abcd"``\ に含まれるコードポイント集合」と「文字列\ ``"cde"``\ に含まれるコードポイント集合」の積集合を計算し、新規のコードポイントの集合（文字列\ ``"cd"``\ に含まれるコードポイント集合）のインスタンスを作成している。

|

.. _StringProcessingHowToUseCodePointsCheck:

コードポイント集合を使った文字列チェック
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| \ ``CodePoints``\ に用意されているメソッドを使用して文字列チェックを行うことが出来る。
| 以下に、文字列チェックを行う際に使用するメソッドの使用例を示す。

**containsAllメソッド**

チェック対象の文字列が全てコードポイント集合に含まれているか判定する。

.. code-block:: java

    CodePoints jisX208KanaCp = CodePoints.of(JIS_X_0208_Katakana.class);

    boolean result;
    result = jisX208KanaCp.containsAll("カ");     // true
    result = jisX208KanaCp.containsAll("カナ");   // true
    result = jisX208KanaCp.containsAll("カナa");  // false

|

**firstExcludedContPointメソッド**

チェック対象の文字列のうち、コードポイント集合に含まれない最初のコードポイントを返却する。
なお、チェック対象の文字列が全てコードポイント集合に含まれている場合は、\ ``CodePoints#NOT_FOUND``\ を返却する。

.. code-block:: java

    CodePoints jisX208KanaCp = CodePoints.of(JIS_X_0208_Katakana.class);

    int result;
    result = jisX208KanaCp.firstExcludedCodePoint("カナa");  // 0x0061 (a)
    result = jisX208KanaCp.firstExcludedCodePoint("カaナ");  // 0x0061 (a)
    result = jisX208KanaCp.firstExcludedCodePoint("カナ");   // CodePoints#NOT_FOUND

|

**allExcludedCodePointsメソッド**

チェック対象の文字列のうち、コードポイント集合に含まれないコードポイントの \ ``Set``\ を返却する。

.. code-block:: java

    CodePoints jisX208KanaCp = CodePoints.of(JIS_X_0208_Katakana.class);

    Set<Integer> result;
    result = jisX208KanaCp.allExcludedCodePoints("カナa");  // [0x0061 (a)]
    result = jisX208KanaCp.allExcludedCodePoints("カaナb"); // [0x0061 (a), 0x0062 (b)]
    result = jisX208KanaCp.allExcludedCodePoints("カナ");   // []

|

.. _StringProcessingHowToUseCodePointsValidator:

Bean Validationと連携した文字列チェック
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| \ ``@org.terasoluna.gfw.common.codepoints.ConsistOf``\ アノテーションにコードポイント集合クラスを指定することで、チェック対象の文字列が指定したコードポイント集合に全て含まれるかをチェックすることが出来る。
| 以下に使用例を示す。

**チェックに用いるコードポイント集合が一つの場合**

.. code-block:: java

    @ConsisOf(JIS_X_0208_Hiragana.class)    // (1)
    private String firstName;

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 対象のフィールドに設定された文字列が、全て「JIS X 0208のひらがな」であることをチェックする。

|

**チェックに用いるコードポイント集合が複数の場合**

.. code-block:: java

    @ConsisOf({JIS_X_0208_Hiragana.class, JIS_X_0208_Katakana.class})    // (1)
    private String firstName;

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 対象のフィールドに設定された文字列が、全て「JIS X 0208のひらがな」または「JIS X 0208のカタカナ」であることをチェックする。

.. note::

    長さNの文字列をM個のコードポイント集合でチェックした場合、N x M回のチェック処理が発生する。文字列の長さが大きい場合は、性能劣化の要因となる恐れがある。
    そのため、チェックに使用するコードポイント集合の和集合となる新規コードポイント集合のクラスを作成し、そのクラスのみを指定したほうが良い。


|

.. _StringProcessingHowToUseCodePointsClassCreation:

コードポイント集合クラスの新規作成
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| コードポイント集合クラスを新規で作成する場合、\ ``CodePoints``\ クラスを継承してコンストラクタでコードポイントを指定する。
| コードポイント集合クラスを新規で作成する方法を以下に示す。
|

**コードポイントを指定して新規にコードポイント集合のクラスを作成する場合**

「数字のみ」からなるコードポイント集合の作成例

.. code-block:: java

     public class NumberChars extends CodePoints {
         public NumberCodePoints() {
             super(0x0030 /* 0 */, 0x0031 /* 1 */, 0x0032 /* 2 */, 0x0033 /* 3 */,
                     0x0034 /* 4 */, 0x0035 /* 5 */, 0x0036 /* 6 */,
                     0x0037 /* 7 */, 0x0038 /* 8 */, 0x0039 /* 9 */);
         }
     }

|

**コードポイント集合クラスの集合演算メソッドを使用して新規にコードポイント集合クラスを作成する場合**

「ひらがな」と「カタカナ」からなる和集合を用いたコードポイント集合の作成例

.. code-block:: java

    public class FullwidthHiraganaKatakana extends CodePoints {
        public FullwidthHiraganaKatakana() {
            super(new X_JIS_0208_Hiragana().union(new X_JIS_0208_Katakana()));
        }
    }

「記号（｡｢｣､･）を除いた半角カタカナ」からなる差集合を用いたコードポイント集合の作成例

.. code-block:: java

    public class HalfwidthKatakana extends CodePoints {
        public HalfwidthKatakana() {
            CodePoints symbolCp = new CodePoints(0xFF61 /* ｡ */, 0xFF62 /* ｢ */,
                    0xFF63 /* ｣ */, 0xFF64 /* ､ */, 0xFF65 /* ･ */);

            super(new JIS_X_0201_Katakana().subtract(symbolCp));
        }
    }

.. note::

    集合演算で使用するコードポイント集合クラス（本例では \ ``X_JIS_0208_Hiragana``\ や、 \ ``X_JIS_0208_Katakana``\ 等）を個別に使用するケースがない場合は、 \ ``new``\ 演算子を使用してコンストラクタを呼び出し、コードポイント集合が無駄にキャッシュされないようにすべきである。
    \ ``CodePoints#of``\ メソッドを使用してキャッシュさせてしまうと、集合演算の途中計算のみで使用するコードポイント集合がヒープに残り、メモリを圧迫してしまう。
    逆に個別に使用するケースがある場合は、\ ``CodePoints#of``\ メソッドを使用してキャッシュすべきである。

|

.. _StringProcessingHowToUseCodePointsClasses:

共通ライブラリから提供しているコードポイント集合クラス
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

共通ライブラリから提供しているコードポイント集合クラス(\ ``org.terasoluna.gfw.common.codepoints.catalog``\ パッケージのクラス)と、
使用する際に取込む必要があるアーティファクトの情報を以下に示す。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.30\linewidth}|p{0.40\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 20 30 40
   :class: longtable

   * - 項番
     - クラス名
     - 説明
     - アーティファクト情報
   * - | (1)
     - | \ ``ASCIIControlChars``\
     - | Ascii制御文字の集合。
       | (\ ``0x0000``\ -\ ``0x001F``\ 、\ ``0x007F``\ )
     - .. code-block:: xml

           <dependency>
               <groupId>org.terasoluna.gfw</groupId>
               <artifactId>terasoluna-gfw-codepoints</artifactId>
           </dependency>
   * - | (2)
     - | \ ``ASCIIPrintableChars``\
     - | Ascii印字可能文字の集合。
       | (\ ``0x0020``\ -\ ``0x007E``\ )
     - | (同上)
   * - | (3)
     - | \ ``CRLF``\
     - | 改行コードの集合。
       | \ ``0x000A``\ (LINE FEED)と\ ``0x000D``\ (CARRIAGE RETURN)。
     - | (同上)
   * - | (4)
     - | \ ``JIS_X_0201_Katakana``\
     - | JIS X 0201 のカタカナの集合。
       | 記号(｡｢｣､･)も含まれる。
     - .. code-block:: xml

           <dependency>
               <groupId>org.terasoluna.gfw.codepoints</groupId>
               <artifactId>terasoluna-gfw-codepoints-jisx0201</artifactId>
           </dependency>
   * - | (5)
     - | \ ``JIS_X_0201_LatinLetters``\
     - | JIS X 0201 のLatin文字の集合。
     - | (同上)
   * - | (6)
     - | \ ``JIS_X_0208_SpecialChars``\
     - | JIS X 0208 の1-2区：特殊文字の集合。
     - .. code-block:: xml

           <dependency>
               <groupId>org.terasoluna.gfw.codepoints</groupId>
               <artifactId>terasoluna-gfw-codepoints-jisx0208</artifactId>
           </dependency>
   * - | (7)
     - | \ ``JIS_X_0208_LatinLetters``\
     - | JIS X 0208 の3区：英数字の集合。
     - | (同上)
   * - | (8)
     - | \ ``JIS_X_0208_Hiragana``\
     - | JIS X 0208 の4区：ひらがなの集合。
     - | (同上)
   * - | (9)
     - | \ ``JIS_X_0208_Katakana``\
     - | JIS X 0208 の5区：カタカナの集合。
     - | (同上)
   * - | (10)
     - | \ ``JIS_X_0208_GreekLetters``\
     - | JIS X 0208 の6区：ギリシア文字の集合。
     - | (同上)
   * - | (11)
     - | \ ``JIS_X_0208_CyrillicLetters``\
     - | JIS X 0208 の7区：キリル文字の集合。
     - | (同上)
   * - | (12)
     - | \ ``JIS_X_0208_BoxDrawingChars``\
     - | JIS X 0208 の8区：罫線素片の集合。
     - | (同上)
   * - | (13)
     - | \ ``JIS_X_0208_Kanji``\
     - | JIS X 208で規定される漢字6355字。
       | 第一・第二水準漢字。
     - .. code-block:: xml

           <dependency>
               <groupId>org.terasoluna.gfw.codepoints</groupId>
               <artifactId>terasoluna-gfw-codepoints-jisx0208kanji</artifactId>
           </dependency>
   * - | (14)
     - | \ ``JIS_X_0213_Kanji``\
     - | JIS X 0213:2004で規定される漢字10050字。
       | 第一・第二・第三・第四水準漢字。
     - .. code-block:: xml

           <dependency>
               <groupId>org.terasoluna.gfw.codepoints</groupId>
               <artifactId>terasoluna-gfw-codepoints-jisx0213kanji</artifactId>
           </dependency>

.. raw:: latex

   \newpage

.. note::

    上記設定例は、依存ライブラリのバージョンを親プロジェクトである terasoluna-gfw-parent で管理する前提であるため、pom.xmlでのバージョンの指定は不要である。
    
    \ ``JIS_X_0208_SpecialChars``\コードポイント集合クラスはJIS漢字(JIS X 0208)の01-02区に該当する特殊文字集合である。
    JIS漢字の全角ダッシュ(―)はEM DASHであり、対応するUCS(ISO/IEC 10646-1, JIS X 0221, Unicode)のコードポイントは、一般的に\ ``U+2014``\に相当する。
    しかし、Unicodeコンソーシアムが提供する変換表では、Unicodeで対応する文字がEM DASHでなく\ `HORINZONTAL BAR (U+2015) <http://www.unicode.org/Public/MAPPINGS/OBSOLETE/EASTASIA/JIS/JIS0208.TXT>`_\になっている。
    実用されている一般的な変換ルールと、Unicode変換表が異なっているため、Unicode変換表通りにコードポイント集合を定義してしまうと実用上問題が出るケースが発生する可能性がある。そのため、\ ``JIS_X_0208_SpecialChars``\コードポイント集合クラスではHORINZONTAL BAR (\ ``U+2015``\)をEM DASH (\ ``U+2014``\)に変更してコードポイント集合を定義している。

.. raw:: latex

   \newpage

