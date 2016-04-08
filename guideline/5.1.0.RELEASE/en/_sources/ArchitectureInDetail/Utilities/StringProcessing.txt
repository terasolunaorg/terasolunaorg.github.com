String Processing
--------------------------------------------------------------------------------

.. only:: html

 .. contents:: Index
    :depth: 4
    :local:

|

Overview
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| There are very few operations in string standard API of Java which specialise in Japanese language.
| A process must be devised independently for conversion of full width katakana / half width katakana and 
| determination of a string consisting of only half width katakana.

| Also, note that although in Java, all the strings are represented in Unicode
| the special characters like 𠮷 are represented in unicode by char type 2 (32 bits) which is referred to as surrogate pair.
| Even while handling these characters, an implementation which takes into account various types of characters are necessary to prevent occurrence of unexpected behavior.

| The guideline assumes a case wherein Japanese language is processed, and
| includes a typical string operation example and offering a Japanese language operation API by a common library.

How to use
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Trim
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| ``String#trim``  method can be used as well while carrying out half width blank trim operation, however, while carrying out complex trim operations like only leading and trailing trim operation, trim operation of any string etc. ``org.springframework.util.StringUtils`` provided by Spring should be preferably used.
|
| Example is given below.
|

.. code-block:: java

   String str = "  Hello World!!";

   StringUtils.trimWhitespace(str); // => "Hello World!!"

   StringUtils.trimLeadingCharacter(str, ' '); // => "Hello World!!"

   StringUtils.trimTrailingCharacter(str, '!'); // => "  Hello World"

.. note::
  There is no change in the behaviour even if surrogate pair string is specified in the first argument of ``StringUtils#trimLeadingCharacter`` and ``StringUtils#trimTrailingCharacter``. Also note that, since the second argument is of char type, surrogate pair cannot be specified.

Padding, Suppress
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| While carrying out padding (string padding) and suppress (string takeout) operations, a method
| provided by ``String`` class can be used.
|
| Example is given below.

.. code-block:: java

   int num = 1;

   String paddingStr = String.format("%03d", num); // => "001"
   String suppressStr = paddingStr.replaceFirst("^0+(?!$)", ""); // => "1"

.. warning::
  If a surrogate pair is included while carrying out padding of apparent length, appropriate results cannot be obtained since surrogate pair cannot be taken into account by ``String#format``.
  In order to achieve the padding by using surrogate pair, it is necessary to count number of characters considered as surrogate pair described later, calculate appropriate number of characters that should be padded and join the strings.

Processing of a string considered as a surrogate pair
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""


Fetching string length
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| When the length of the string considered as surrogate pair is to be fetched, it is not possible to
| simply use ``String#length`` method.
| Since surrogate pair is represented by 32 bits (char type 2), the count tends to exceed than the apparent number of characters.
|
| In the example below, 5 is assigned to variable ``len``.

.. code-block:: java

   String str = "𠮷田太郎";
   int len = str.length(); // => 5

|
| Accordingly, a method ``String#codePointCount`` is defined wherein length of the string considered as a surrogate pair is fetched from Java SE 5.
| String length can be fetched by specifying start index and end index of the target string in the argument of ``String#codePointCount``.
|
| Example is given below.

.. code-block:: java

   String str = "𠮷田太郎";
   int lenOfChar = str.length(); // => 5
   int lenOfCodePoint = str.codePointCount(0, lenOfChar); // => 4

|
| Further, Unicode consists of "concatenation".
| Since there is no appearance difference between ``\u304c`` indicating [が] and ``\u304b\u3099`` indicating [か] plus [voiced sound mark], however [か] plus [voiced sound mark] are likely to be counted as two characters.
| When number of characters are to be counted including the combining characters as well as described above, counting is done after normalization of text using ``java.text.Normalizer``.
|
| A method which returns length of the string after considering combining characters and surrogate pair is given below.

.. code-block:: java

   public int getStrLength(String str) {
     String normalizedStr  = Normalizer.normalize(str, Normalizer.Form.NFC);
     int length = normalizedStr.codePointCount(0, normalizedStr.length());

     return length;
   }


Fetch string in the specified range
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| When a string of specified range is to be fetched, unintended results may be obtained if only ``String#substring`` is used.

.. code-block:: java

   String str = "𠮷田 太郎";
   int startIndex = 0;
   int endIndex = 2;
   
   String subStr = str.substring(startIndex, endIndex);

   System.out.println(subStr); // => "𠮷"

| In the example above, when you try to fetch "𠮷田" by taking out 2 characters from 0th character (beginning), only "𠮷" could be fetched since the surrogate pair is represented by 32 bits (char type 2).
| In such a case, ``String#substring`` method must be used by searching start and end positions considering the surrogate pair, by using ``String#offsetByCodePoints``.
|
| An example wherein 2 characters are taken from the beginning (surname part) is shown below.

.. code-block:: java

   String str = "𠮷田 太郎";
   int startIndex = 0;
   int endIndex = 2;

   int startIndexSurrogate = str.offsetByCodePoints(0, startIndex); // => 0
   int endIndexSurrogate = str.offsetByCodePoints(0, endIndex); // => 3

   String subStrSurrogate = str.substring(startIndexSurrogate, endIndexSurrogate); // => "𠮷田"

|

String split
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
| ``String#split`` method handles the surrogate pair as a default.
| Example is given below.


.. code-block:: java

   String str = "𠮷田 太郎";
   
   str.split(" "); // => {"𠮷田", "太郎"}

|

    .. note::
      | Surrogate pair can also be specified in the argument of ``String#split`` as a delimiter.

    .. note::
      | Please note that behaviour while passing a blank character in ``String#split`` changes in Java SE 7 environment, and Java SE 8. Refer `Pattern#split Javadoc <http://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html#split-java.lang.CharSequence-int->`_
      
      .. code-block:: java 
      
        String str = "ABC";
        String[] elems = str.split("");
        
        // Java SE 7 => {, A, B, C}
        // Java SE 8 => {A, B, C}


Full width, half width string conversion
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Full width and half width character conversion is carried out by using API of \ ``org.terasoluna.gfw.common.fullhalf.FullHalfConverter``\  class provided by common library.

\ ``FullHalfConverter``\  class adopts a style wherein pair definition of full width and half width characters for conversion (\ ``org.terasoluna.gfw.common.fullhalf.FullHalfPair``\ ) is registered in advance.
\ ``FullHalfConverter``\  object for which default pair definition is registered, is provided as a \ ``INSTANCE``\  constant
of \ ``org.terasoluna.gfw.common.fullhalf.DefaultFullHalf``\  class in the common library.
For default pair definition, refer `DefaultFullHalf source <https://github.com/terasolunaorg/terasoluna-gfw/blob/master/terasoluna-gfw-string/src/main/java/org/terasoluna/gfw/common/fullhalf/DefaultFullHalf.java>`_ .

.. note::

    When change requirements are not met in the default pair definition provided by common library, \ ``FullHalfConverter``\  object registering a unique pair definition should be created.
    For basic methods for creation, refer :ref:`StringOperationsHowToUseCustomFullHalfConverter` .


Conversion to full width string
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

\ ``toFullwidth``\  method of \ ``FullHalfConverter``\  is used while converting a half width character to full width character.

.. code-block:: java

   String fullwidth = DefaultFullHalf.INSTANCE.toFullwidth("ｱﾞ!A8ｶﾞザ");    // (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Pass the string which contains half width characters to argument of \ ``toFullwidth``\  method and convert to full width string.
       | In this example, it is converted to \ ``"ア゛！Ａ８ガザ"``\ . Also, note that the characters for which a pair is not defined (\ ``"ザ"``\  in this example) are returned without any change.


Conversion to half width string
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

\ ``toHalfwidth``\  method of \ ``FullHalfConverter``\  is used while converting a full width character to half width character.

.. code-block:: java

   String halfwidth = DefaultFullHalf.INSTANCE.toHalfwidth("Ａ！アガｻ");    // (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Pass the string which contains full width characters to argument of \ ``toHalfwidth``\  method and convert to half width characters.
       | In this example, it is converted to \ ``"A!ｱｶﾞｻ"``\ . Also, note that the characters for which a pair is not defined (\ ``"ｻ"``\  of this example) are returned without any change.

.. note::

    \ ``FullHalfConverter``\  cannot convert combining characters that represent a single character using 2 or more characters (Example: "\ ``"シ"``\ (\ ``\u30b7``\ ) + voiced sound mark(\ ``\u3099``\ )"）to half width character (Example: \ ``"ｼﾞ"``\ ).
    When combining characters are to be converted to half width characters, \ ``FullHalfConverter``\  must be used after converting the same to integrated characters （Example:\ ``"ジ"``\ (\ ``\u30b8``\ )）by carrying out text normalization.
    
    \ ``java.text.Normalizer``\  is used while carrying out text normalization.
    Note that, when combining characters are to be converted to integrated characters, NFC or NFKC is used as a normalization format.


    Implementation example wherein NFD (analyse by canonical equivalence) is used as a normalization format
    
      .. code-block:: java

         String str1 = Normalizer.normalize("モジ", Normalizer.Form.NFD); // str1 = "モシ + Voiced sound mark(\u3099)"
         String str2 = Normalizer.normalize("ﾓｼﾞ", Normalizer.Form.NFD);  // str2 = "ﾓｼﾞ"

    Implementation example wherein NFC (analyse by canonical equivalence, and integrate again) is used as a normalization format
    
      .. code-block:: java

         String mojiStr = "モシ\u3099";                                   // "モシ + Voiced sound mark(\u3099)"
         String str1 = Normalizer.normalize(mojiStr, Normalizer.Form.NFC); // str1 = "モジ（\u30b8）"
         String str2 = Normalizer.normalize("ﾓｼﾞ", Normalizer.Form.NFC);   // str2 = "ﾓｼﾞ"
    
    Implementation example wherein NFKD (analyse by compatibility equivalent) is used as a normalization format
    
      .. code-block:: java

         String str1 = Normalizer.normalize("モジ", Normalizer.Form.NFKD); // str1 = "モシ + Voiced sound mark(\u3099)"
         String str2 = Normalizer.normalize("ﾓｼﾞ", Normalizer.Form.NFKD);  // str2 = "モシ + Voiced sound mark(\u3099)"
    
    Implementation example wherein NFKC (analyse by compatibility equivalent and integrate again) is used as a normalization format
    
      .. code-block:: java

         String mojiStr = "モシ\u3099";                                    // "モシ + Voiced sound mark(\u3099)"
         String str1 = Normalizer.normalize(mojiStr, Normalizer.Form.NFKC); // str1 = "モジ（\u30b8）"
         String str2 = Normalizer.normalize("ﾓｼﾞ", Normalizer.Form.NFKC) ;  // str2 = "モジ"
    
    
    For details, refer \ `Normalizer JavaDoc <https://docs.oracle.com/javase/8/docs/api/java/text/Normalizer.html>`_\ .


.. _StringOperationsHowToUseCustomFullHalfConverter:

Creating FullHalfConverter class for which a unique full width and half width character pair definition is registered
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| \ ``FullHalfConverter``\  for which a unique full width and half width character pair definition is registered can also be used without using \ ``DefaultFullHalf``\ .
| How to use \ ``FullHalfConverter``\  for which a unique full width character and half width character pair definition is registered, is shown below.

**Implementation example of a class that provides FullHalfConverter for which a unique pair definition is registered**

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

    * - Sr. No.
      - Description
    * - | (1)
      - | Use \ ``org.terasoluna.gfw.common.fullhalf.FullHalfPairsBuilder``\  and create \ ``org.terasoluna.gfw.common.fullhalf.FullHalfPairs``\  which represents a set of pair definition of full width and half width characters.
    * - | (2)
      - | Half width character corresponding to \ ``"ー"``\ of full width character set to \ ``"ｰ"``\ (\ ``\uFF70``\ ) in \ ``DefaultFullHalf``\  is changed to \ ``"-"``\ (\ ``\u002D``\ ) in this example.
        | Further, although \ ``"-"``\ (\ ``\u002D``\ ) is also included in the process target given below (3), the pair definition defined earlier is given the precedence.
    * - | (3)
      - | In this example, a pair is defined for code values of full width of unicode from \ ``"！"``\  to \ ``"～"``\  and of half width of unicode from \ ``"!"``\  to \ ``"~"``\  using a loop process which use the characteristic "code value sequence is same".
    * - | (4)
      - | Since code value sequence for the characters other than given in (3) does not match for full width characters and half width characters, define a pair individually for respective characters.
    * - | (5)
      - | Use \ ``FullHalfPairs``\  created by \ ``FullHalfPairsBuilder``\  and create \ ``FullHalfConverter``\ .

.. note::

    For the values that can be specified in the argument of \ ``FullHalfPairsBuilder#pair``\  method,
    refer `FullHalfPair constructor JavaDoc <https://github.com/terasolunaorg/terasoluna-gfw/blob/master/terasoluna-gfw-string/src/main/java/org/terasoluna/gfw/common/fullhalf/FullHalfPair.java>`_
    

|

**How to use FullHalfConverter for which a unique pair definition is registered**

.. code-block:: java
 
    String halfwidth = CustomFullHalf.INSTANCE.toHalfwidth("ハローワールド！"); // (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Use \ ``toHalfwidth``\  method of \ ``FullHalfConverter``\  object for which a unique pair definition is registered and convert the string containing full width characters to half width string.
        | In this example, it is converted to \ ``"ﾊﾛ-ﾜ-ﾙﾄﾞ!"``\ . (\ ``"-"``\  is \ ``\u002D``\ ）


|

.. _StringProcessingHowToUseCodePoints:

Code point set check (character type check)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

A code point set function provided by common library should be used for checking character type.

Here, how to implement a character type check by using a code point set function is explained.

* :ref:`StringProcessingHowToUseCodePointsConstruction`
* :ref:`StringProcessingHowToUseCodePointsOperations`
* :ref:`StringProcessingHowToUseCodePointsCheck`
* :ref:`StringProcessingHowToUseCodePointsValidator`
* :ref:`StringProcessingHowToUseCodePointsClassCreation`


.. _StringProcessingHowToUseCodePointsConstruction:

Creating code point set
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| \ ``org.terasoluna.gfw.common.codepoints.CodePoints``\  is a class that represents a code point set.
| How to create \ ``CodePoints``\  instance is shown below.

**When an instance is created by calling a factory method (cache)**

| A method wherein an instance is created from a code point set class ( \ ``Class<? extends CodePoints>``\ ) and the created instance is then cached, is explained below.
| This method is recommended for cache process since it is not necessary to create multiple specific code point sets.

.. code-block:: java

   CodePoints codePoints = CodePoints.of(ASCIIPrintableChars.class);  // (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Pass code point set class in \ ``CodePoints#of``\  method (factory method) and fetch an instance.
       | In this example, an instance of code point set class (\ ``org.terasoluna.gfw.common.codepoints.catalog.ASCIIPrintableChars``\ ) of Ascii printable characters is fetched.

.. note::

    Code point set class exists multiple times in the module, same as \ ``CodePoints``\  class.
    Although other modules which provide code point set also exist, these modules must be added to their own projects when required.
    For details, refer :ref:`StringProcessingHowToUseCodePointsClasses`.

    Further, a new code point set class can also be created.
    For details, refer :ref:`StringProcessingHowToUseCodePointsClassCreation`.

|

**When an instance is created by calling a constructor of code point set class**

| A method wherein an instance is created from code point set class.
| When this method is used, it is recommended to use the process which need not be cached (argument of set operation) since the created instance is not cached.

.. code-block:: java

   CodePoints codePoints = new ASCIIPrintableChars();  // (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Call constructor by using \ ``new``\  operator and generate an instance of code point set class.
       | In this example, an instance of code point set class ( \ ``ASCIIPrintableChars``\ ) of Ascii printable characters is generated.

|

**When an instance is created by calling constructor of CodePoints**

| A method wherein an instance is created from \ ``CodePoints``\  is shown below.
| When this method is used, it is recommended to use the process which need not be cached (argument of set operation) since the created instance is not cached.

* When the codepoint ( \ ``int``\ ) is to be passed by using a variable length argument

  .. code-block:: java

      CodePoints codePoints = new CodePoints(0x0061 /* a */, 0x0062 /* b */);  // (1)

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - Sr. No.
        - Description
      * - | (1)
        - | Generate an instance by passing \ ``int``\  code point in \ ``CodePoints``\  constructor.
          | In this example, an instance of code point set for characters \ ``"a"``\  and \ ``"b"``\  is generated.

|

* When the \ ``Set``\  of code point ( \ ``int``\ ) is to be passed

  .. code-block:: java

      Set<Integet> set = new HashSet<>();
      set.add(0x0061 /* a */);
      set.add(0x0062 /* b */);
      CodePoints codePoints = new CodePoints(set);  // (1)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - Sr. No.
        - Description
      * - | (1)
        - | Add code point of \ ``int``\  to \ ``Set``\  and  generate an instance by passing the \ ``Set``\  in constructor of \ ``CodePoints``\ .
          | In this example, an instance of code point set for characters \ ``"a"``\  and \ ``"b"``\  is generated.

|

* When code point set string is to be passed by using variable length argument

  .. code-block:: java

      CodePoints codePoints = new CodePoints("ab");         // (1)

  .. code-block:: java

      CodePoints codePoints = new CodePoints("a", "b");  // (2)

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - Sr. No.
        - Description
      * - | (1)
        - | Generate an instance by passing code point set string in constructor of \ ``CodePoints``\ .
          | In this example, an instance of code point set for characters \ ``"a"``\  and \ ``"b"``\  is generated.
      * - | (2)
        - | Code point set string can also be passed by dividing it in the arguments. Result is same as (1).

|

.. _StringProcessingHowToUseCodePointsOperations:

Set operation of the code point sets
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| A new code point set instance can be created by performing set operation for code point set.
| Further, note that status of source code point set does not change due to set operation.
| A method wherein an instance of code point set is created by using set operation is given below.


**When an instance of code point set is created by using union set method**

.. code-block:: java

    CodePoints abCp = new CodePoints(0x0061 /* a */, 0x0062 /* b */);
    CodePoints cdCp = new CodePoints(0x0063 /* c */, 0x0064 /* d */);

    CodePoints abcdCp = abCp.union(cdCp);    // (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Calculate union of two code point sets by using  \ ``CodePoints#union``\  method and create an instance of new code point set.
        | In this example, union of "code point set included in string\ ``"ab"``\ " and "code point set included in string \ ``"cd"``\ " is calculated and an instance of new code point set (code point set included in string \ ``"abcd"``\ ) is generated.

|

**When an instance of code point set is created by using difference set method**

.. code-block:: java

    CodePoints abcdCp = new CodePoints(0x0061 /* a */, 0x0062 /* b */,
            0x0063 /* c */, 0x0064 /* d */);
    CodePoints cdCp = new CodePoints(0x0063 /* c */, 0x0064 /* d */);

    CodePoints abCp = abcdCp.subtract(cdCp);    // (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Calculate difference set of two code point sets by using \ ``CodePoints#subtract``\  method and create an instance of new code point set.
        | In this example, difference set of "code point set included in string \ ``"abcd"``\ " and "code point set included in string  \ ``"cd"``\ " is calculated and an instance of new code point set (code point set included in string  \ ``"ab"``\ ) is created.

|

**When an instance of new code point set is to be created by intersection set**

.. code-block:: java

    CodePoints abcdCp = new CodePoints(0x0061 /* a */, 0x0062 /* b */,
            0x0063 /* c */, 0x0064 /* d */);
    CodePoints cdeCp = new CodePoints(0x0063 /* c */, 0x0064 /* d */, 0x0064 /* e */);

    CodePoints cdCp = abcdCp.intersect(cdeCp);    // (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Calculate intersection set of two code point sets by using \ ``CodePoints#intersect``\  method and create an instance of new code point set.
        | In this example, calculate intersection set of "code point set included in string \ ``"abcd"``\ " and "code point set included in string \ ``"cde"``\ " is calculated and an instance of new code point set (code point set included in string \ ``"cd"``\ ）is created.

|

.. _StringProcessingHowToUseCodePointsCheck:

String check by using code point set
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| A string can be checked by using a method provided in \ ``CodePoints``\ .
| How to use a method which is used while checking the string is given below.

**containsAll method**

Determine whether the entire string for checking is included in the code point set.

.. code-block:: java

    CodePoints jisX208KanaCp = CodePoints.of(JIS_X_0208_Katakana.class);

    boolean result;
    result = jisX208KanaCp.containsAll("カ");     // true
    result = jisX208KanaCp.containsAll("カナ");   // true
    result = jisX208KanaCp.containsAll("カナa");  // false

|

**firstExcludedContPoint method**

Return the first code point which is not included in the code point set, from the string targeted for checking.
Further, return \ ``CodePoints#NOT_FOUND``\  when the entire string for checking is included in code point set.

.. code-block:: java

    CodePoints jisX208KanaCp = CodePoints.of(JIS_X_0208_Katakana.class);

    int result;
    result = jisX208KanaCp.firstExcludedCodePoint("カナa");  // 0x0061 (a)
    result = jisX208KanaCp.firstExcludedCodePoint("カaナ");  // 0x0061 (a)
    result = jisX208KanaCp.firstExcludedCodePoint("カナ");   // CodePoints#NOT_FOUND

|

**allExcludedCodePoints method**

Return \ ``Set``\  of the code point which is not included in the code point set, from the string targeted for checking.

.. code-block:: java

    CodePoints jisX208KanaCp = CodePoints.of(JIS_X_0208_Katakana.class);

    Set<Integer> result;
    result = jisX208KanaCp.allExcludedCodePoints("カナa");  // [0x0061 (a)]
    result = jisX208KanaCp.allExcludedCodePoints("カaナb"); // [0x0061 (a), 0x0062 (b)]
    result = jisX208KanaCp.allExcludedCodePoints("カナ");   // []

|

.. _StringProcessingHowToUseCodePointsValidator:

String check linked with Bean Validation
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| It can be checked whether the entire string targeted for checking is included in the specified code point set by specifying code point set class in \ ``@org.terasoluna.gfw.common.codepoints.ConsistOf``\  annotation.
| How to use is shown below.

**When there is only one code point set used for checking**

.. code-block:: java

    @ConsisOf(JIS_X_0208_Hiragana.class)    // (1)
    private String firstName;

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Check whether the string specified in the targeted field is entirely "Hiragana of JIS X 0208".

|

**When there are multiple code point sets used for checking**

.. code-block:: java

    @ConsisOf({JIS_X_0208_Hiragana.class, JIS_X_0208_Katakana.class})    // (1)
    private String firstName;

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Check whether the string specified in the targeted field is entirely "Hiragana of JIS X 0208" or "Katakana of JIS X 0208".

.. note::

    If string of length N is checked by code point sets M, a checking process that contains N x M is employed. When the string is large, it is likely to cause performance degradation.
    Hence, a new code point set class that acts as a union set of code point set used for checking is created, that class alone should be specified.


|

.. _StringProcessingHowToUseCodePointsClassCreation:

Creating new code point set class
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| When a new code point set class is to be created, specify code point using constructor by inheriting \ ``CodePoints``\  class.
| A method wherein a new code point set class is created is given below.
|

**When a new code point set class is created by specifying code point**

How to create code point set which is formed by "only numbers"

.. code-block:: java

     public class NumberChars extends CodePoints {
         public NumberCodePoints() {
             super(0x0030 /* 0 */, 0x0031 /* 1 */, 0x0032 /* 2 */, 0x0033 /* 3 */,
                     0x0034 /* 4 */, 0x0035 /* 5 */, 0x0036 /* 6 */,
                     0x0037 /* 7 */, 0x0038 /* 8 */, 0x0039 /* 9 */);
         }
     }

|

**When a new code point set class is created by using a set operation method of code point set class**

How to create a code point set using a union set consisting of "Hiragana" and "Katakana"

.. code-block:: java

    public class FullwidthHiraganaKatakana extends CodePoints {
        public FullwidthHiraganaKatakana() {
            super(new X_JIS_0208_Hiragana().union(new X_JIS_0208_Katakana()));
        }
    }

How to create a code point set using difference set consisting of "half width katakana excluding symbols (｡｢｣､･)"

.. code-block:: java

    public class HalfwidthKatakana extends CodePoints {
        public HalfwidthKatakana() {
            CodePoints symbolCp = new CodePoints(0xFF61 /* ｡ */, 0xFF62 /* ｢ */,
                    0xFF63 /* ｣ */, 0xFF64 /* ､ */, 0xFF65 /* ･ */);

            super(new JIS_X_0201_Katakana().subtract(symbolCp));
        }
    }

.. note::

    When the code point set class used in set operation (\ ``X_JIS_0208_Hiragana``\  or \ ``X_JIS_0208_Katakana``\ etc in this example) is not to be used individually, it must be ensured that code point is not needlessly cached, by using \ ``new``\  operator and calling constructor.
    If it is cached by using \ ``CodePoints#of``\ method, code point set used only during set operation calculation remains in the heap resulting in load on the memory.
    On the other hand, if it is used individually, it should be cached using \ ``CodePoints#of``\  method.

|

.. _StringProcessingHowToUseCodePointsClasses:

Code point set class provided by common library
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Code point class provided by common library (\ ``org.terasoluna.gfw.common.codepoints.catalog``\  package class) and 
artifact information to be incorporated while using are given below.

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.30\linewidth}|p{0.40\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 20 30 40

   * - Sr. No.
     - Class name
     - Description
     - Artifact information
   * - | (1)
     - | \ ``ASCIIControlChars``\
     - | Ascii control characters set.
       | (\ ``0x0000``\ -\ ``0x001F``\ 、\ ``0x007F``\ )
     - .. code-block:: xml

           <dependency>
               <groupId>org.terasoluna.gfw</groupId>
               <artifactId>terasoluna-gfw-codepoints</artifactId>
           </dependency>
   * - | (2)
     - | \ ``ASCIIPrintableChars``\
     - | Ascii printable characters set.
       | (\ ``0x0020``\ -\ ``0x007E``\ )
     - | (Same as above)
   * - | (3)
     - | \ ``CRLF``\
     - | Linefeed code set.
       | \ ``0x000A``\ (LINE FEED) and \ ``0x000D``\ (CARRIAGE RETURN)。
     - | (Same as above)
   * - | (4)
     - | \ ``JIS_X_0201_Katakana``\
     - | JIS X 0201  katakana set.
       | Symbols (｡｢｣､･) included as well.
     - .. code-block:: xml

           <dependency>
               <groupId>org.terasoluna.gfw.codepoints</groupId>
               <artifactId>terasoluna-gfw-codepoints-jisx0201</artifactId>
           </dependency>
   * - | (5)
     - | \ ``JIS_X_0201_LatinLetters``\
     - | JIS X 0201 Latin characters set.
     - | (Same as above)
   * - | (6)
     - | \ ``JIS_X_0208_SpecialChars``\
     - | Row 2 of JIS X 0208: Special characters set.
     - .. code-block:: xml

           <dependency>
               <groupId>org.terasoluna.gfw.codepoints</groupId>
               <artifactId>terasoluna-gfw-codepoints-jisx0208</artifactId>
           </dependency>
   * - | (7)
     - | \ ``JIS_X_0208_LatinLetters``\
     - | Row 3 of JIS X 0208: Alphanumeric set.
     - | (Same as above)
   * - | (8)
     - | \ ``JIS_X_0208_Hiragana``\
     - | Row 4 of JIS X 0208: Hiragana set.
     - | (Same as above)
   * - | (9)
     - | \ ``JIS_X_0208_Katakana``\
     - | Row 5 of JIS X 0208: Katakana set.
     - | (Same as above)
   * - | (10)
     - | \ ``JIS_X_0208_GreekLetters``\
     - | Row 6 of JIS X 0208: Greek letters set.
     - | (Same as above)
   * - | (11)
     - | \ ``JIS_X_0208_CyrillicLetters``\
     - | Row 7 of JIS X 0208: Cyrillic letters set.
     - | (Same as above)
   * - | (12)
     - | \ ``JIS_X_0208_BoxDrawingChars``\
     - | Row 8 of JIS X 0208: Box drawing characters.
     - | (Same as above)
   * - | (13)
     - | \ ``JIS_X_0208_Kanji``\
     - | Kanji 6355 characters specified in JIS X 208.
       | First and second level kanjis.
     - .. code-block:: xml

           <dependency>
               <groupId>org.terasoluna.gfw.codepoints</groupId>
               <artifactId>terasoluna-gfw-codepoints-jisx0208kanji</artifactId>
           </dependency>
   * - | (14)
     - | \ ``JIS_X_0213_Kanji``\
     - | Kanji 10050 characters specified in JIS X 0213:2004.
       | First, second, third and fourth level kanjis.
     - .. code-block:: xml

           <dependency>
               <groupId>org.terasoluna.gfw.codepoints</groupId>
               <artifactId>terasoluna-gfw-codepoints-jisx0213kanji</artifactId>
           </dependency>

|

.. raw:: latex

   \newpage

