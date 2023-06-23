Overview of Spring MVC Architecture 
-------------------------------------

.. only:: html

 .. contents:: Index
    :local:


Official website of Spring MVC says the following

`Spring Reference Document <http://docs.spring.io/spring/docs/4.1.9.RELEASE/spring-framework-reference/html/mvc.html>`_\ .

     Spring's web MVC framework is, like many other web MVC frameworks, request-driven,
     designed around a central Servlet that dispatches requests to controllers and offers other functionality
     that facilitates the development of web applications. Spring's DispatcherServlet however, does more than just that.
     It is completely integrated with the Spring IoC container and as such allows you to use every other feature that Spring has.

Overview of Spring MVC Processing Sequence
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. The request processing workflow of the Spring MVC is illustrated in the following diagram.

The processing flow of Spring MVC from receiving the request till the response is returned is shown in the following diagram.

.. figure:: ./images/RequestLifecycle.png
   :alt: request lifecycle
   :width: 100%

1. ``DispatcherServlet`` receives the request.
2. ``DispatcherServlet`` dispatches the task of selecting an appropriate ``controller`` to ``HandlerMapping``. ``HandlerMapping`` selects the controller which is mapped to the incoming request URL and returns the ``(selected Handler)`` and ``Controller`` to ``DispatcherServlet``.
3. ``DispatcherServlet`` dispatches the task of executing of business logic of ``Controller`` to ``HandlerAdapter``.
4. ``HandlerAdapter`` calls the business logic process of ``Controller``.
5. ``Controller`` executes the business logic, sets the processing result in ``Model`` and returns the logical name of view to ``HandlerAdapter``.
6. ``DispatcherServlet`` dispatches the task of resolving the ``View`` corresponding to the View name to ``ViewResolver``. ``ViewResolver`` returns the ``View`` mapped to View name.
7. ``DispatcherServlet`` dispatches the rendering process to returned ``View``.
8. ``View`` renders ``Model`` data and returns the response.

Implementation of each component
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Among the components explained previously, the extendable components are implemented.

Implementation of HandlerMapping
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Class hierarchy of ``HandlerMapping`` provided by Spring framework is shown below.

.. figure:: ./images/HandlerMapping-Hierarchy.png
   :alt: HandlerMapping Hierarchy


Usually ``org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping`` is used. This class reads ``@RequestMapping`` annotation from the ``Controller`` and uses the method of ``Controller`` that matches with URL as Handler class.

In Spring3.1, ``RequestMappingHandlerMapping`` is enabled by default when ``<mvc:annotation-driven>`` is set in Bean definition file read by ``DispatcherServlet``.
(For the settings which get enabled with the use of ``<mvc:annotation-driven>`` annotation, refer
`Web MVC framework <http://docs.spring.io/spring/docs/4.1.9.RELEASE/spring-framework-reference/html/mvc.html#mvc-config-enable>`_\.)


Implementation of HandlerAdapter
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Class hierarchy of ``HandlerAdapter`` provided by Spring framework is shown below.

.. figure:: ./images/HandlerAdapter-Hierarchy.png
   :alt: HandlerAdapter Hierarchy

Usually ``org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter`` is used.
``RequestMappingHandlerAdapter`` class calls the method of handler class (``Controller``) selected by ``HandlerMapping``.
In Spring 3.1, this class is also configured by default throught ``<mvc:annotation-driven>``.

Implementation of ViewResolver
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Classes that implement ``ViewResolver`` provided by Spring framework and dependent libraries are shown below.

.. figure:: ./images/ViewResolver-Hierarchy.png
   :alt: ViewResolver Hierarchy

Normally (When JSP is used),

* ``org.springframework.web.servlet.view.InternalResourceViewResolver`` is used, 

however, when template engine Tiles is to be used

* ``org.springframework.web.servlet.view.tiles3.TilesViewResolver``

and when stream is to be returned for file download

* ``org.springframework.web.servlet.view.BeanNameViewResolver``

Thereby, it is required to use different viewResolver based on the type of the View that needs to be returned.

| When ``View`` of multiple types is to be handled, multiple definitions of ``ViewResolver`` are required.
| A typical example of using multiple ``ViewResolver`` is the screen application for which file download process exists.
| For screen (JSP), ``View`` is resolved using  ``InternalResourceViewResolver`` and for File download ``View`` is resolved using ``BeanNameViewResolver``.
| For details, refer :doc:`../ArchitectureInDetail/FileDownload`.


Implementation of View
^^^^^^^^^^^^^^^^^^^^^^^^

Classes that implement ``View`` provided by Spring framework and its dependent libraries are shown below.

.. figure:: ./images/View-Hierarchy.png
   :alt: View Hierarchy

``View`` changes with the type of response to be returned. When JSP is to be returned, ``org.springframework.web.servlet.view.JstlView`` is used.
When ``View`` not provided by Spring framework and its dependent libraries are to be handled, it is necessary to extend the class in which ``View`` interface is implemented.
For details, refer :doc:`../ArchitectureInDetail/FileDownload`.


.. raw:: latex

   \newpage

