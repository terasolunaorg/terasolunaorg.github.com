This document covers the following
================================================================================

This guideline provides best practices to develop highly maintainable Web applications using
full stack framework focussing on Spring, Spring MVC and JPA, MyBatis.

This guideline helps to proceed with the software development (mainly coding) smoothly.

Target readers of this document
================================================================================

This guideline is written for the architects and programmers having software development experience
and knowledge of the following.

* Basic knowledge of DI and AOP of Spring Framework
* Web development experience using Servlet/JSP
* Knowledge of SQL
* Experience of building Web Application using Maven

This guideline is not for beginners.

In order to check whether the readers have basic knowledge of Spring Framework,
refer to \ :doc:`../Appendix/SpringComprehensionCheck`\ .
It is recommended to study the following books if one is not able to answer 40% of the comprehension test.

* `Spring徹底入門 (翔泳社) [Japanese] <http://www.shoeisha.co.jp/book/detail/9784798142470>`_
* `Spring3入門―Javaフレームワーク・より良い設計とアーキテクチャ（技術評論社）[Japanese] <http://gihyo.jp/book/2012/978-4-7741-5380-3>`_
* `Pro Spring 4th Edition (Apress) <http://www.apress.com/9781430261513>`_

Structure of this document
================================================================================

* \ :doc:`../Overview/index`\ 
    Overview of Spring MVC and basic concepts of TERASOLUNA Server Framework for Java (5.x) is explained.
* \ :doc:`../ImplementationAtEachLayer/index`\ 
    Knowledge and methods for application development using TERASOLUNA Server Framework for Java (5.x) are explained.
* \ :doc:`../ArchitectureInDetail/index`\
    Method to implement the functions required for general application development using TERASOLUNA Server Framework for Java (5.x) or features of each function is explained.
* \ :doc:`../Security/index`\ 
    Security measures are explained focusing on Spring Security.
* \ :doc:`../Tutorial/index`\ 
    Experience in application development using TERASOLUNA Server Framework for Java (5.x) through simple application development.
* \ :doc:`../Appendix/index`\
    Describing the additional information when TERASOLUNA Server Framework for Java (5.x) is being used.

Reading this document
================================================================================

| Firstly read "\ :doc:`../Overview/index`\ ".
| Implement "\ :doc:`../Overview/FirstApplication`\ " for beginners of Spring MVC.
| Read "\ :doc:`../Overview/ApplicationLayering`\ " as the terminology and concepts used in this guideline are explained here.
| 
| Then continue with "\ :doc:`../Tutorial/index`\ ".
| Get a feel of application development using TERASOLUNA Global
| Framework by firstly trying it as per the proverb "Practice makes perfect".
| 
| Use this tutorial to study the details of application development in "\ :doc:`../ImplementationAtEachLayer/index`\ ".
| Since the knowhow for development is explained using Spring MVC in "\ :doc:`../ImplementationAtEachLayer/ApplicationLayer`\ ",
| it is recommended to read it again and again several times.
| One can get a better understanding by studying "\ :doc:`../Tutorial/index`\ " once again after reading this chapter.
| 
| **It is strongly recommended that all the developers who use TERASOLUNA Server Framework for Java (5.x) read it.**
| 
| Refer to "\ :doc:`../ArchitectureInDetail/index`\ " and "\ :doc:`../Security/index`\ "
| as per the requirement. However, read ":doc:`../ArchitectureInDetail/WebApplicationDetail/Validation`" since it is normally required for application development.
| 
| The technical leader understands all the contents and checks the type of policy to be decided in the project.


.. note::

    If you do not have sufficient time, first go through the following.
    
    #. \ :doc:`../Overview/FirstApplication`\ 
    #. \ :doc:`../Overview/ApplicationLayering`\ 
    #. \ :doc:`../Tutorial/TutorialTodo`\ 
    #. \ :doc:`../ImplementationAtEachLayer/index`\ 
    #. \ :doc:`../Tutorial/TutorialTodo`\ 
    #. \ :doc:`../ArchitectureInDetail/WebApplicationDetail/Validation`\ 

Tested environments of this document
================================================================================

For tested environments of contents  described in this guideline,
refer to "\ `Tested Environment <https://github.com/terasolunaorg/terasoluna-gfw-functionaltest/wiki/Tested-Environment>`_\".

.. raw:: latex

   \newpage

