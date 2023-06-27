Spring Framework Comprehension Check
================================================================================

#.  Fill (1)-(4) such that the Bean dependency relation is as follows. Skip import statement.

    .. figure:: images/appendix-spring_comprehension_check-dependency_relation.png
       :width: 80%

    .. code-block:: java
    
        @Controller
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
    
        ``@Service``\  , \ ``@Controller``\  are \ ``org.springframework.stereotype``\  package annotations and \ ``@Transactional``\  is the annotation of \ ``org.springframework.transaction.annotation``\ .

#. Explain when to use \ ``@Controller``\ , \ ``@Service``\ , and \ ``@Repository``\  respectively.
    
    .. note::
    
        Each of them is \ ``org.springframework.stereotype``\  package annotation.

#. Explain the difference between \ ``@Resource``\  and \ ``@Inject``\ .

    .. note:: \ ``@Resource``\  is \ ``javax.annotation``\  package annotation, and \ ``@Inject``\  is \ ``javax.inject``\  package annotation.
    
#. Explain the difference between singleton scope and prototype scope.

#. Fill (1)-(3) in the following Scope related description. However, either of "singleton" or "prototype" can be entered in (1) and (2), but same value cannot be entered for both. Skip the import statement.

    .. code-block:: java
    
        @Component
        (3)
        public class XxxComponent {
          // omitted
        }
        
    .. note::
        
        \ ``@Component``\  is \ ``org.springframework.stereotype.Component``\ .
        
    Scope of bean with \ ``@Component``\  is (1) by default. When changing scope to (2), it is better to add (3) (refer to above source code).

#. In case of following Bean definition, what type of Bean is registered in DI container?

    .. code-block:: xml
    
        <bean id="foo" class="xxx.yyy.zzz.Foo" factory-method="create">
            <constructor-arg index="0" value="aaa" />
            <constructor-arg index="1" value="bbb" />
        </bean>

#. Fill (1)-(3) of the following Bean definition such that contents in \ ``com.example.domain``\  package becomes target of component scan.


    .. code-block:: text
    
        <context:(1) (2)="(3)" />
        
    .. note::
    
        Bean definition file should include the definition of
        
        xmlns:context="http://www.springframework.org/schema/context"
        
        


#. Fill (1)-(2) in the following Properties file related description. Skip import statement.

    It is possible to read the Bean definition file in \ ``${key}``\  format by removing the settings in properties file, if properties file path is set in the \ ``locations``\  attribute of  \ ``<context:property-placeholder>``\  element. Specify as shown in (1) to read any properties file under META-INF/spring directory under the class path. Moreover, @(2) annotation should be added as shown in the following codes where the read properties value can also be injected in Bean.

    .. code-block:: xml

        <context:property-placeholder locations="(1)" />

    .. code-block:: properties
    
        emails.min.count=1
        emails.max.count=4

    .. code-block:: java

        @Service
        @Transactional
        public class XxxServiceImpl implements XxxService {
          @xxx("${emails.min.count}") // (2)
          protected int emailsMinCount;
          @xxx("${emails.max.count}") // (2)
          protected int emailsMaxCount;
          // omitted
        }

    .. note::
    
          Bean definition file should include the definitions of
          
          xmlns:context="http://www.springframework.org/schema/context"
          
          


#. Fill (1)-(5) in the following description for AOP Advice of Spring. The contents of (1)-(5) are all different.

    .. note::
    
        Advice (1) should be used when interrupting a process before calling a specific method, Advice (2) should be used when interrupting a process after calling a specific method. Advice (3) should be used when interrupting a process before and after calling a specific method. Advice (4) should be used only when the process is ended normally and Advice (5) should be used when there is an exception.


#. Insert (*) of following Bean definition for performing transaction management using  \ ``@Transactional``\  annotation.

    .. code-block:: text

        <tx:(*) />

    .. note::
    
       Bean definition file should include the definitions of
       
       xmlns:tx="http://www.springframework.org/schema/tx"
       
       
.. raw:: latex

   \newpage

