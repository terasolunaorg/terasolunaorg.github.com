Spring MVCアーキテクチャ概要
--------------------------------

.. contents:: 目次
   :local:

.. Spring MVC is explained as follows in

Spring MVCは、公式で以下のように説明されている。

`Spring Reference Document <http://static.springsource.org/spring/docs/3.2.x/spring-framework-reference/html/mvc.html>`_\ .

     Spring's web MVC framework is, like many other web MVC frameworks, request-driven,
     designed around a central Servlet that dispatches requests to controllers and offers other functionality
     that facilitates the development of web applications. Spring's DispatcherServlet however, does more than just that.
     It is completely integrated with the Spring IoC container and as such allows you to use every other feature that Spring has.

Overview of Spring MVC Processing Sequence
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. The request processing workflow of the Spring MVC is illustrated in the following diagram.

リクエストを受けてから、レスポンスを返すまでのSpring MVCの処理フローを、以下の図に示す。

.. figure:: ./images/RequestLifecycle.png
   :alt: request lifecycle
   :width: 80%

1. \ ``DispatcherServlet``\ が、リクエストを受け取る。
2. \ ``DispatcherServlet``\ は、リクエスト処理を行う\ ``Controller``\ の選択を、\ ``HandlerMapping``\ に委譲する。\ ``HandlerMapping``\ は、リクエストURLにマッピングされている\ ``Controller``\ を選定し、\ ``（Choose Handler）``\ , \ ``Controller``\ を\ ``DispatcherServlet``\ へ返却する。
3. \ ``DispatcherServlet`` は、\ ``HandlerAdapter``\ に対して、\ ``Controller``\ のビジネスロジック処理の実行を委譲する。
4. \ ``HandlerAdapter`` は、\ ``Controller``\ のビジネスロジック処理を呼び出す。
5. \ ``Controller``\ は、ビジネスロジックを実行し、処理結果を\ ``Model``\ に設定し、ビューの論理名を\ ``HandlerAdapter``\ に返却する。
6. \ ``DispatcherServlet``\ は、ビュー名に対応する\ ``View``\ の解決を、\ ``ViewResolver``\ に委譲する。\ ``ViewResolver``\ は、ビュー名にマッピングされている\ ``View``\ を返却する。
7. \ ``DispatcherServlet``\ は、返却された\ ``View``\ に、レンダリング処理を委譲する。
8. \ ``View``\ は、\ ``Model``\ が持つ情報をレンダリングして、レスポンスを返却する。

Implementations of each component
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

これまで説明したコンポーネントのうち、拡張可能なコンポーネントを紹介する。

Implementaion of HandlerMapping
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Springから提供されている\ ``HandlerMapping``\ のクラス階層を、以下に示す。

.. figure:: ./images/HandlerMapping-Hierarchy.png
   :alt: HandlerMapping Hierarchy


| 通常使用するのは、\ ``org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping``\ であり、
| このクラスはBean定義されている\ ``Contorller``\ から、\ ``@RequestMapping``\ アノテーションを読み取り、
| URLと合致する\ ``Controller``\ のメソッドを、Handlerクラスとして扱うクラスである。

| Spring3.1からは、\ ``RequestMappingHandlerMapping``\ は、\ ``DispatcherServlet``\ が読み込むBean定義ファイルに、
| \ ``<mvc:annotation-driven>``\ の設定がある場合、デフォルトで設定される。
| (\ ``<mvc:annotation-driven>``\ アノテーションで有効になる設定は、\ `Web MVC framework <http://static.springsource.org/spring/docs/3.2.x/spring-framework-reference/html/mvc.html#mvc-config-enable>`_\ を参照されたい。)


Implementaion of HandlerAdapter
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Springから提供されている\ ``HandlerAdapter``\ のクラス階層を、以下に示す。

.. figure:: ./images/HandlerAdapter-Hierarchy.png
   :alt: HandlerAdapter Hierarchy

| 通常使用するのは、\ ``org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter``\ である。
| こちらもSpring3.1からは、\ ``<mvc:annotation-driven>``\ の設定がある場合、デフォルトで設定される。
| \ ``RequestMappingHandlerAdapter``\ は、\ ``HandlerMapping``\ によって、選択されたハンドラークラス (\ ``Controller``\ ) のメソッドを呼び出すクラスである。

Implementaion of ViewResolver
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Springおよび依存ライブラリから提供されている\ ``ViewResolver``\ のクラスを、以下に示す。

.. figure:: ./images/ViewResolver-Hierarchy.jpg
   :alt: ViewResolver Hierarchy

通常(JSPを使う場合)は、

*  \ ``org.springframework.web.servlet.view.InternalResourceViewResolver``\ を使用するが、
テンプレートエンジンTilesを使う場合は、

* \ ``org.springframework.web.servlet.view.tiles2.TilesViewResolver``\
ファイルダウンロード用にストリームを返す場合は

* ``org.springframework.web.servlet.view.BeanNameViewResolver``

のように、返す\ ``View``\ によって、使い分ける必要がある。

| 複数の種類の\ ``View``\ を扱う場合は、\ ``ViewResolver``\ の定義が、複数必要となるケースがある。
| 複数の\ ``ViewResolver``\ を使う代表的な例としては、ファイルのダウンロード処理が存在する画面アプリケーションである。
| 画面(JSP)は、\ ``InternalResourceViewResolver``\ で、\ ``View``\ を解決し、
| ファイルダウンロードは、\ ``BeanNameViewResolver``\ などを使って、\ ``View``\ を解決する。
| 詳細は、\ :doc:`../ArchitectureInDetail/FileDownload`\ を参照されたい。


Implementaion of View
^^^^^^^^^^^^^^^^^^^^^

Springおよび依存ライブラリから提供されている\ ``View``\ のクラスを、以下に示す。

.. figure:: ./images/View-Hierarchy.jpg
   :alt: View Hierarchy

| \ ``View``\ は、返したいレスポンスの種類によって変わる。
| JSPを返す場合は、\ ``org.springframework.web.servlet.view.JstlView``\ が使用される。

| Springおよび依存ライブラリから提供されていない\ ``View``\ を扱いたい場合は、\ ``View``\ インタフェースを実装したクラスを、拡張実装する必要がある。
| 詳細は、\ :doc:`../ArchitectureInDetail/FileDownload`\ を参照されたい。
