Spring Framework理解度チェックテスト
================================================================================

#. Beanの依存関係が以下の図のようになるように(1)～(4)を埋めてください。import文は省略してください。

    .. figure:: images/appendix-spring_comprehension_check-dependency_relation.png
       :width: 80%

    .. code-block:: java
    
        @Contoller
        public class XxxController {
          (1)
          protected (2) yyyService;
        
          // omitted
        }
    
    .. code-block:: java
    
        @Service
        @Transactional
        public class YyyServiceImpl implements YyyService {
          (1)
          protected (4) zzzRepository;
        
          // omitted
        }

    .. note::
    
        ``@Service``\ ,\ ``@Controller``\ は\ ``org.springframework.stereotype``\ パッケージのアノテーション、\ ``@Transactional``\ は\ ``org.springframework.transaction.annotation``\ のアノテーションである。

#. \ ``@Controller``\ と\ ``@Service``\ と\ ``@Repository``\ はそれぞれどういう場合に使用するか説明してください。
    
    .. note::
    
        それぞれ\ ``org.springframework.stereotype``\ パッケージのアノテーションです。

#. \ ``@Resource``\ と\ ``@Inject``\ の違いを説明してください

    .. note:: \ ``@Resource``\ は\ ``javax.annotation``\ パッケージ、\ ``@Inject``\ は\ ``javax.inject``\ パッケージのアノテーションです。
    
#. Scopeがsingletonの場合とprototypeの場合の違いを説明してください。

#. Scopeに関する次の説明で(1)～(3)を埋めてください。ただし(1)、(2)には"singleton"または"prototype"のどちらが入り、同じ値は入りません。またimport文は省略してください。

    .. code-block:: java
    
        @Component
        (3)
        public class XxxComponent {
          // omitted
        }
        
    .. note::
        
        \ ``@Component``\ は\ ``org.springframework.stereotype.Component``\ 
        
    \ ``@Component``\ をつけたBeanのscopeはデフォルトで(1)である。scopeを(2)にする場合、(3)をつければよい(上記ソース参照)。

#. 次のBean定義を行った場合、どのようなBeanがDIコンテナに登録されますか。

    .. code-block:: xml
    
        <bean id="foo" class="xxx.yyy.zzz.Foo" factory-method="create">
            <constructor-arg index="0" value="aaa" />
            <constructor-arg index="1" value="bbb" />
        </bean>

#. \ ``com.example.domain``\ パッケージ以下がcomponent scanの対象となるように以下のBean定義の(1)～(3)を埋めてください。


    .. code-block:: text
    
        <context:(1) (2)="(3)" />
        
    .. note::
    
        Bean定義ファイルには
        
        xmlns:context="http://www.springframework.org/schema/context"
        
        の定義があるものとする。
        

#. プロパティファイルに関する次の説明で(1)～(2)を埋めてください。import文は省略してください。

    設定値をプロパティファイルに外出しし、Bean定義ファイル内から\ ``${key}``\ 形式で参照したい場合に\ ``<context:property-placeholder>``\ 要素の\ ``locations``\ 属性にプロパティファイルのパスを設定すれば読み込むことができる。クラスパス直下のMETA-INF/springディレクトリ以下の任意のプロパティファイルを読み込む場合は(1)のように指定する。また読み込んだプロパティ値はBeanにもインジェクション可能であり下記コードのように@(2)アノテーションをつければよい。

    .. code-block:: xml

        <context:property-placeholder locations="(1)" />

    .. code-block:: properties
    
        emails.min.count=1
        emails.max.count=4

    .. code-block:: java

        @Service
        @Transactional
        public class XxxServiceImpl implements XxxService {
          @xxx("${emails.min.count}") // (2)xxx部分
          protected int emailsMinCount;
          @xxx("${emails.max.count}") // (2)xxx部分
          protected int emailsMaxCount;
          // omitted
        }

    .. note::
    
          Bean定義ファイルには
          
          xmlns:context="http://www.springframework.org/schema/context"
          
          の定義があるものとする。
        

#. Springが提供するAOPのAdviceについての次の説明で(1)～(5)を埋めてください。尚、(1)～(5)には全て別の内容が入ります。

    .. note::
    
        特定のメソッド呼び出しの前に処理を割り込ませたい場合のAdviceは(1)で、メソッド呼び出し後に割り込ませたい場合のAdviceは(2)である。前後両方に割り込ませたい場合は(3) Adviceを使用すればよい。メソッドが正常終了したときにのみ実行されるAdviceは(4)であり、例外発生時に実行されるAdviceは(5)である。


#. \ ``@Transactional``\ アノテーションによるトランザクション管理を行うために以下のBean定義の(*)を埋めてください。

    .. code-block:: text

        <tx:(*) />

    .. note::
    
       Bean定義ファイルには
       
       xmlns:tx="http://www.springframework.org/schema/tx"
       
       の定義があるものとする。

.. raw:: latex

   \newpage

