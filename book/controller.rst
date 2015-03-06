.. index::
   single: Controller

控制器（Controller）
========================

控制器是一段可以被调用的 PHP 代码，它从 HTTP 请求中获取信息，并且相应地构造和返回一个 HTTP 响应（作为 Symfony 的 ``Response`` 对象）。 响应可以是一个 HTML 页面，可以是一个 XML 文件，或是一个序列化了的 JSON 数组，或是一张图片，也可以是一个重定向甚至一个 404 错误，它可以是你能想到的一切！控制器将包含 *你的程序* 所需的一切渲染页面内容的逻辑。

通过看这个 Symfony 控制器来了解这一切是多么的简单吧！这是一个渲染著名的 ``Hello world!`` 页面的控制器::

    use Symfony\Component\HttpFoundation\Response;

    public function helloAction()
    {
        return new Response('Hello world!');
    }

控制器的目标永远都是明确的：创建并返回一个 ``Response`` 对象。在这个过程中，控制器可能会从请求（Request）中读取一些信息，载入几个数据库资源，发送一封电子邮件，或者在用户会话（Session）中写入一些东西。但不论在哪一种情况下，控制器最终都要返回将要发回客户端的 ``Response`` 对象。

这里面没有魔法，也没有需要你担心的其他需求~这儿有一些简单的例子：

* *控制器 A* 要创建一个 ``Response`` 对象来展现
  网站主页的内容。

* *控制器 B* 从用户请求中读取 ``slug`` 参数来从数据库中加载
  博客的条目然后创建一个 ``Response`` 对象来把博客的内容显示
  出来。如果数据库中没有 ``slug`` ，控制器就创建一个
  包含 404 状态码的 ``Response`` 对象并把它发送回客户端。

* *控制器 C* 来处理一个联系表格的表单子任务。它从
  用户请求中读取信息，存储在
  数据库中并给你发送一封包含联系信息的电子邮件。最后，它创建
  一个 ``Response`` 对象来把用户的浏览器重定向到
  表单的“谢谢您”页面。

.. index::
   single: Controller; Request-controller-response lifecycle

请求（Requests）、控制器（Controller）、响应（Response）生命周期
-------------------------------------------------------------------------------------------------------------------------------

每一个 Symfony 项目处理的请求都经过这个简单的生命周期。框架将接管所有重复的活动，这意味着你只需要把你自己的特有的代码写入控制器的函数即可：

#. 每个请求都被交给一个单一的前端控制器（Front Controller）处理并引导整个程序（如 ``app.php``
   或 ``app_dev.php``）；

#. ``Router（路由器）`` 从请求中读取信息（比如 URI)，并
   寻找一条符合这个信息的路由，然后读取路由信息中的 ``_controller`` 
   参数；

#. 被命中的路由信息中给出的控制器将被执行，控制器中的代码将
   创建并返回一个 ``Response`` 对象；

#. ``Response`` 对象中的 HTTP 头和内容将被送回
   客户端。

创建一个页面简单到只需要创建一个控制器（第三步中用到），再添加一条路由将 URL 映射到控制器上（第二步中用到）。

.. note::

    虽然名字很像，但“前端控制器”和本章讨论的“控制器”
    不是同一个东西。前端控制器
    在你网站目录下的一个短小的 PHP 文件，
    所有的请求都被指向它。典型的程序会有一个生产环境
    前端控制器（比如 ``app.php``）和一个开发环境前端控制器。
    （比如 ``app_dev.php``）。你基本不需要编辑、浏览或者担心
    你的程序中前端控制器的代码。

.. index::
   single: Controller; Simple example

一个简单的控制器
------------------------

虽然刚才说到控制器可以是任何一段可以被调用的 PHP 代码（比如一个函数、对象中的方法或者一个 ``Closure（闭包）``），但控制器一般都是控制器类中的一个方法。控制器也被叫做 *Action（动作）*。

.. code-block:: php

    // src/AppBundle/Controller/HelloController.php
    namespace AppBundle\Controller;

    use Symfony\Component\HttpFoundation\Response;

    class HelloController
    {
        public function indexAction($name)
        {
            return new Response('<html><body>Hello '.$name.'!</body></html>');
        }
    }

.. tip::

    注意这里的 *控制器* 是在 *控制器类*（``HelloController``）的 ``indexAction`` 方法。
    别被
    *控制器类* 这个名字搞糊涂了，这只是一种将几个
    控制器（方法）组合在一起的简便方法而已。一般情况下，控制器类
    里面会有一些控制器（比如 ``updateAction``、``deleteAction`` 
    等等）。

这个控制器相当明了：

* *第4行*： Symfony 使用 PHP 5.3 的命名空间这一很方便的功能来
  命名整个控制器类。``use`` 关键字将导入
  控制器必须返回的 ``Response`` 类。

* *第6行*：类名是在你给控制器类起的名字
  （比如 ``Hello``）后面加上 ``Controller`` 这个单词构成的。这个规范
  保持控制器的一致性并且将允许你只使用
  类名的第一个部分 （在这里是 ``Hello``）来配置路由。

* *第8行*：控制器类中的每一个动作都要以 ``Action`` 结尾
  这样你就可以在配置路由时只写动作本身的名字（这里是 ``index`` ）了。  在下一节你将创建一条路由将 URL 映射到这个动作上。  你也将学到如何把路由中的占位符（这里是``{name}``）变成
  动作的方法的参数（这里是``$name``）。

* *第10行*：控制器创建并返回一个 ``Response`` 对象。

.. index::
   single: Controller; Routes and controllers

将 URL 映射到控制器上
--------------------------------

这个新控制器返回一个简单的 HTML 页面。要想真正地访问这个页面，你需要创建一条将指定 URL 路径映射到对应控制器的路由：

.. configuration-block::

    .. code-block:: php-annotations

        // src/AppBundle/Controller/HelloController.php
        namespace AppBundle\Controller;

        use Symfony\Component\HttpFoundation\Response;
        use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

        class HelloController
        {
            /**
             * @Route("/hello/{name}", name="hello")
             */
            public function indexAction($name)
            {
                return new Response('<html><body>Hello '.$name.'!</body></html>');
            }
        }

    .. code-block:: yaml

        # app/config/routing.yml
        hello:
            path:      /hello/{name}
            # 使用这种特定的表达式来指向控制器 - 参阅下面的注解
            defaults:  { _controller: AppBundle:Hello:index }

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing
                http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="hello" path="/hello/{name}">
                <!-- 使用这种特定的表达式来指向控制器 - 参阅下面的注解 -->
                <default key="_controller">AppBundle:Hello:index</default>
            </route>
        </routes>

    .. code-block:: php

        // app/config/routing.php
        use Symfony\Component\Routing\Route;
        use Symfony\Component\Routing\RouteCollection;

        $collection = new RouteCollection();
        $collection->add('hello', new Route('/hello/{name}', array(
            // 使用这种特定的表达式来指向控制器 - 参阅下面的注解
            '_controller' => 'AppBundle:Hello:index',
        )));

        return $collection;

好了，现在如果你访问 ``/hello/ryan`` （比如在你使用:doc:`built-in web server </cookbook/web_server/built_in>` 链接就是 ``http://localhost:8000/app_dev.php/hello/ryan``）时， Symfony 就会执行 ``HelloController::indexAction()`` 控制器并将 ``ryan`` 传入作为``$name`` 变量的值。创建“页面”的意思只是简单地创建一个控制器的方法和对应的路由。

简单吧？

.. sidebar:: 类似 AppBundle:Hello:index 这样的表达式的语法

    如果你用 YML 或者 XML 格式，你就需要使用一种特定的
    表达式来定位一个控制器： ``AppBundle:Hello:index``。想要详细了解
    控制器定位表达式，请参阅 :ref:`controller-string-syntax`。

.. seealso::

    你可以从 :doc:`Routing chapter </book/routing>` 更详细地学习路由系统。
    

.. index::
   single: Controller; Controller arguments

.. _route-parameters-controller-arguments:

作为控制器参数的路由占位符
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

你已经知道了路由指向了 AppBundle 中的 ``HelloController::indexAction()`` 方法。更有趣的东西是传入那个方法的参数::

    // src/AppBundle/Controller/HelloController.php
    // ...
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

    /**
     * @Route("/hello/{name}", name="hello")
     */
    public function indexAction($name)
    {
        // ...
    }

控制器有一个与被命中的路由信息中的 ``{name}`` 占位符对应的参数 ``$name``（如果你访问 ``/hello/ryan`` 就是 ``ryan``）。当你的控制器被执行时，Symfony 会将控制器的参数与路由占位符一一对应。所以 ``{name}`` 的值将被传递给 ``$name``。

看一下这个更有趣的例子吧：

.. configuration-block::

    .. code-block:: php-annotations

        // src/AppBundle/Controller/HelloController.php
        // ...

        use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

        class HelloController
        {
            /**
             * @Route("/hello/{firstName}/{lastName}", name="hello")
             */
            public function indexAction($firstName, $lastName)
            {
                // ...
            }
        }

    .. code-block:: yaml

        # app/config/routing.yml
        hello:
            path:      /hello/{firstName}/{lastName}
            defaults:  { _controller: AppBundle:Hello:index }

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing
                http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="hello" path="/hello/{firstName}/{lastName}">
                <default key="_controller">AppBundle:Hello:index</default>
            </route>
        </routes>

    .. code-block:: php

        // app/config/routing.php
        use Symfony\Component\Routing\Route;
        use Symfony\Component\Routing\RouteCollection;

        $collection = new RouteCollection();
        $collection->add('hello', new Route('/hello/{firstName}/{lastName}', array(
            '_controller' => 'AppBundle:Hello:index',
        )));

        return $collection;

现在，控制器可以有两个参数了::

    public function indexAction($firstName, $lastName)
    {
        // ...
    }

将路由占位符映射到控制器参数是简单且灵活的。在开发时请记住以下几条准则。.

* **控制器参数与顺序无关**

  Symfony 使用路由占位符的 **名字** 和控制器参数的
  **名字** 来进行映射。控制器参数可以被完全
  重新排序而且仍然可以完美运行::

      public function indexAction($lastName, $firstName)
      {
          // ...
      }

* **控制器需要的所有参数都必须有一个路由占位符与之对应**

  下面的代码将抛出一个 ``RuntimeException（运行时异常）`` 因为在路由中 ``foo``
  这个占位符没有被定义::

      public function indexAction($firstName, $lastName, $foo)
      {
          // ...
      }

  但是将 ``foo`` 这个参数设为可选参数是可行的。下面这个
  例子就不会抛出异常::

      public function indexAction($firstName, $lastName, $foo = 'bar')
      {
          // ...
      }

* **并不是所有的路由占位符都需要有一个控制器参数与之对应**

  如果假设 ``lastName`` 在你的控制器中并不是那么重要，
  你可以完全忽略掉它::

      public function indexAction($firstName)
      {
          // ...
      }

.. tip::

    每一个路由也都有一个特殊的 ``_route`` 占位符，它等同于
    被命中的路由的名字（比如在这里是 ``hello``）。虽然并不经常
    用到,，它同样可以被用于一个控制器参数。你也可以
    你也可以将其他来自你的路由的变量传入控制器。参阅
    :doc:`/cookbook/routing/extra_information`.

.. _book-controller-request-argument:

将 ``Request`` 作为控制器参数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

假设你需要读取一个查询参数，抓取一个请求头，或者访问一个被上传上来的文件。所有的这些信息都被存储到了 Symfony 的 ``Request（请求）`` 对象中。如果想在你的控制器中使用它，只需要将它添加为参数并 **使用Request 类对其进行类型约束（Type-Hint）** ::

    use Symfony\Component\HttpFoundation\Request;

    public function indexAction($firstName, $lastName, Request $request)
    {
        $page = $request->query->get('page', 1);

        // ...
    }

.. seealso::

    想学习关于从请求中获取信息的更多？参阅
    :ref:`Access Request Information <component-http-foundation-request>`.

.. index::
   single: Controller; Base controller class

控制器基类
-------------------------

为了更加方便，Symfony 提供了一个 ``Controller`` 基类。如果你将其继承，你就可以访问很多的帮手方法，也可以通过容器来访问你的服务（参阅 :ref:`controller-accessing-services`）。

把 ``use`` 的声明放在 ``Controller`` 类的上面，然后修改一下 ``HelloController`` 去继承基类::

    // src/AppBundle/Controller/HelloController.php
    namespace AppBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class HelloController extends Controller
    {
        // ...
    }

这并不会实际地修改你控制器工作的任何部分：它只是可以让你访问基类提供的帮手方法。这只是一些使用 Symfony 核心功能的快捷方法，这些核心功能无论你是否使用 ``Controller`` 基类都可用。查看正在运作的核心功能的方法就是看看 `Controller class`_ 。

.. seealso::

    如果你很好奇控制器在 *不* 继承
    这个基类时如何工作，请参阅 :doc:`Controllers as Services </cookbook/controller/service>`。    这是可选的，但可以让你更精确地控制注入到你控制器中的
    类或者依赖。

.. index::
   single: Controller; Redirecting

重定向
~~~~~~~~~~~

如果你想将用户重定向到另一个页面，请使用 :method:`Symfony\\Bundle\\FrameworkBundle\\Controller\\Controller::redirect` 方法 ::

    public function indexAction()
    {
        return $this->redirect($this->generateUrl('homepage'));
    }

上面的 ``generateUrl()`` 方法只是一个生成给定路由的 URL 的帮手方法。获取更多信息，请参阅 :doc:`Routing </book/routing>` 章节。

在默认情况下， ``redirect()`` 方法生成的是 302（暂时）重定向。要想生成 301（永久）重定向，请修改第二个参数 ::

    public function indexAction()
    {
        return $this->redirect($this->generateUrl('homepage'), 301);
    }

.. tip::

    上面提到的 ``redirect()`` 方法只是一个创建专门重定向用户的 ``Response``
    类的快捷方式。它等价于::

        use Symfony\Component\HttpFoundation\RedirectResponse;

        return new RedirectResponse($this->generateUrl('homepage'));

.. index::
   single: Controller; Rendering templates

.. _controller-rendering-templates:

渲染模板
~~~~~~~~~~~~~~~~~~~

如果你要使用 HTML，你就一定要渲染模板。一个叫做 ``render()`` 的方法会渲染一个模板 **并且** 为你把内容放入 ``Response`` 类中::

    // 渲染 app/Resources/views/Hello/index.html.twig
    return $this->render('Hello/index.html.twig', array('name' => $name));

你也可以将模板文件放入更深的子文件夹中。但还是要避免创建不必要的更深的结构::

    // 渲染 app/Resources/views/Hello/Greetings/index.html.twig
    return $this->render('Hello/Greetings/index.html.twig', array('name' => $name));

在 :doc:`Templating </book/templating>` 一章详细讲解了 Symfony 模板引擎。

.. sidebar:: 使用储存在包内部的模板

    你也可以将模板放入一个包内的 ``Resources/views`` 目录下，
    并使用
    ``包名:目录名:文件名`` 这样的表达式来使用它。例如，
    ``AppBundle:Hello:index.html.twig`` 代表的是 
    ``src/AppBundle/Resources/views/Hello/index.html.twig`` 这个模板文件。参阅 :ref:`template-referencing-in-bundle`。

.. index::
   single: Controller; Accessing services

.. _controller-accessing-services:

访问其他服务
~~~~~~~~~~~~~~~~~~~~~~~~

Symfony 打包了很多有用的类，它们被称为服务。这些服务被用来渲染模板、发送邮件、查询数据库，也可以用来做一些你想让它们“做”的工作。当你安装新的包时，它可能会引入 *更多的* 服务。

当你继承了控制器基类时，你就可以通过 ``get()`` 方法来访问任何的 Symfony 服务。这里有一些你可能会用到的基本服务::

    $templating = $this->get('templating');

    $router = $this->get('router');

    $mailer = $this->get('mailer');

那么别的服务在哪儿呢？你可以用 ``container:debug`` 这个控制台命令列出所有的服务：

.. code-block:: bash

    $ php app/console container:debug

更多信息，请参阅 :doc:`/book/service_container` 一章。

.. index::
   single: Controller; Managing errors
   single: Controller; 404 pages

管理错误和 404 页面
------------------------------------------------

当没有找到一些东西事，你应该用好 HTTP 协议并返回一个 404 响应。为了达到目的，你可以抛出一个特殊的异常。如果你继承了控制器基类，按照下面的来做::

    public function indexAction()
    {
        // 从数据库中检索目标
        $product = ...;
        if (!$product) {
            throw $this->createNotFoundException('产品不存在');
        }

        return $this->render(...);
    }

上面用到的 ``createNotFoundException()`` 方法只是一个创建特殊的 :class:`Symfony\\Component\\HttpKernel\\Exception\\NotFoundHttpException` 对象（一个创建 HTTP 404 响应的 Symfony 类）的快捷方式。

当然，你可以自由的从你的控制器中抛出任何 ``Exception（异常）`` 类——Symfony 将会自动的生成 HTTP 500（内部服务器错误）响应。

.. code-block:: php

    throw new \Exception('出错了！');

任何情况下，最终用户看到的都是错误页面，开发者看到的都是完整的调试信息 （例如当你使用 ``app_dev.php`` 时——参阅 :ref:`page-creation-environments`）。

你一定想自定义终端用户看到的错误页面。为达到目的，请参阅技巧书中的 ":doc:`/cookbook/controller/error_pages`" 这一技巧。

.. index::
   single: Controller; The session
   single: Session

管理会话
--------------------

Symfony 提供一个很好用的会话类，你可以用它在请求间存储用户（可以是一个使用浏览器的真实的人类，或是一个蜘蛛机器人，或是一个网络服务）的信息。默认情况下，Symfony 使用 PHP 原生的会话管理工具将这些信息储存在 Cookie 中。

不管在哪个控制器中，向会话写入信息和从会话中读取信息都可以轻易实现::

    use Symfony\Component\HttpFoundation\Request;

    public function indexAction(Request $request)
    {
        $session = $request->getSession();

        // 存储一个在处理用户之后的请求时会用到的属性
        $session->set('foo', 'bar');

        // 获取在别的会话中别的控制器设置的属性
        $foobar = $session->get('foobar');

        // 在属性不存在时使用一个默认值
        $filters = $session->get('filters', array());
    }

这些属性将持续到用户其余的请求中。

.. index::
   single: Session; Flash messages

闪电消息
~~~~~~~~~~~~~~

你也可以向用户会话存储一条只在紧接着的下一个请求中可用的短消息。这在处理表格时很有用：你想将用户重定向并在 *下一个* 页面中显示一条特定的消息。这种消息被称为“闪电”消息。

设想你正在处理一个提交上来的表格::

    use Symfony\Component\HttpFoundation\Request;

    public function updateAction(Request $request)
    {
        $form = $this->createForm(...);

        $form->handleRequest($request);

        if ($form->isValid()) {
            // 做一些处理

            $request->getSession()->getFlashBag()->add(
                'notice',
                '更改已保存！'
            );

            return $this->redirect($this->generateUrl(...));
        }

        return $this->render(...);
    }

处理完请求后，控制器在会话中设置了一个叫做 ``notice`` 的闪电消息并重定向。名字（上面的例子里是``notice``）并没有特殊的意义，只是个你起的名字，方便你在下一步中使用它。

在下一个页面中的模板里（更聪明的方法是写入主模板框架），下面的代码将渲染 ``notice`` 这个消息。

.. configuration-block::

    .. code-block:: html+jinja

        {% for flashMessage in app.session.flashbag.get('notice') %}
            <div class="flash-notice">
                {{ flashMessage }}
            </div>
        {% endfor %}

    .. code-block:: html+php

        <?php foreach ($view['session']->getFlash('notice') as $message): ?>
            <div class="flash-notice">
                <?php echo "<div class='flash-error'>$message</div>" ?>
            </div>
        <?php endforeach ?>

闪电消息被专门设计为只能在紧接着的请求中使用（它们像闪电一样转瞬即逝）。像刚才这样在重定向时传递消息就可以用到闪电消息。

.. index::
   single: Controller; Response object

Response（响应）对象
----------------------------------------------

对控制器的要求只有一个：返回一个 ``Response`` 对象。Symfony 中的 :class:`Symfony\\Component\\HttpFoundation\\Response` 类是对 HTTP 响应的抽象：响应头和内容被填入基于文本的消息中发回客户端::

    use Symfony\Component\HttpFoundation\Response;

    // 创建一个有 200 状态码（默认）的简单响应
    $response = new Response('Hello '.$name, 200);

    // 创建一个有 200 状态码（默认）的 JSON 响应
    $response = new Response(json_encode(array('name' => $name)));
    $response->headers->set('Content-Type', 'application/json');

上面的 ``headers`` 属性是一个 :class:`Symfony\\Component\\HttpFoundation\\HeaderBag` 类，它有一些很棒的读写响应头的方法。响应头的名字是标准化了的，所以用 ``Content-Type`` 等价于 ``content-type`` 或者 ``content_type``。

也有一些可以简单快速地创建其他类型的响应的类。

* JSON对应的类是 :class:`Symfony\\Component\\HttpFoundation\\JsonResponse`。  参阅 :ref:`component-http-foundation-json-response`。

* 文件对应的类是 :class:`Symfony\\Component\\HttpFoundation\\BinaryFileResponse`。  参阅 :ref:`component-http-foundation-serving-files`。

* 流式响应对应的类在 :class:`Symfony\\Component\\HttpFoundation\\StreamedResponse`。  参阅 :ref:`streaming-response`。

.. seealso::

    别担心！在足见的文档里还有很多关于响应对象的
    信息。参阅 :ref:`component-http-foundation-response`。

.. index::
   single: Controller; Request object

请求（Request）对象
--------------------------------------------

除了来自路由占位符的值，控制器还可以访问 ``Request（请求）`` 对象。如果一个变量被使用 :class:`Symfony\\Component\\HttpFoundation\\Request` 进行类型约束，框架就会将 ``请求`` 对象注入控制器中::

    use Symfony\Component\HttpFoundation\Request;

    public function indexAction(Request $request)
    {
        $request->isXmlHttpRequest(); // 是一个Ajax请求吗？

        $request->getPreferredLanguage(array('en', 'fr'));

        $request->query->get('page'); // 获取一个 $_GET 的参数

        $request->request->get('page'); // 获取一个 $_POST 的参数
    }

就像 ``响应`` 对象一样，请求头被存储在 ``HeaderBag（请求头包）`` 对象中，访问起来很容易。

.. seealso::

    别担心！在足见的文档里还有很多关于请求对象的
    信息。参阅 :ref:`component-http-foundation-request`。

创建静态页面
------------------------------

你也可以创建一个不需要控制器的静态页面（只需要路由和模板）。

参阅 :doc:`/cookbook/templating/render_without_controller`。

.. index::
   single: Controller; Forwarding

重定向到另一个控制器
-----------------------------------------------

虽然不是很常用，但你还是可以使用 :method:`Symfony\\Bundle\\FrameworkBundle\\Controller\\Controller::forward` 这一方法来在内部重定向到别的控制器。这样做并不会重定向用户的浏览器，而会建立一个内部子请求并调用对应的控制器。刚才提到的 ``forward()`` 方法会返回一个来自 *那个（重定向到的）* 控制器的 ``Response`` 对象::

    public function indexAction($name)
    {
        $response = $this->forward('AppBundle:Something:fancy', array(
            'name'  => $name,
            'color' => 'green',
        ));

        // ... 做一些别的更改或者直接返回它

        return $response;
    }

请注意 ``forward()`` 方法使用一种特殊的控制器定位表达式（参阅 :ref:`controller-string-syntax`）。在这个例子中，目标控制器是 AppBundle 中的 ``SomethingController::fancyAction()`` 控制器。作为方法的参数的数组将会被作为控制器参数传入目标控制器。在将控制器嵌入模板时也会用到这一方法（参阅 :ref:`templating-embedding-controller`）。目标控制器可以像下面这样工作::

    public function fancyAction($name, $color)
    {
        // ... 创建并返回一个 Response 对象
    }

就像在给路由创建控制器时那样， ``fancyAction`` 的参数的顺序并不影响运行。Symfony 会将数组的键名（比如 ``name``）与控制器方法的参数名（比如 ``$name``）对应起来。如果你更改了参数的顺序，Symfony 还是会将正确的值传递给各个变量。

结语
--------------

不论在什么时候，当你创建一个页面时，你最终都需要写一些包括这个页面的逻辑的代码。在 Symfony 里，这被称为控制器，并且它是一个可以为了返回最终会被返回给用户的 ``Response`` 对象而做任何事的 PHP 函数。

简单起见，你可以选择继承 ``Controller`` 基类，它包含了很多控制器要做的基本的事情的快捷方式。比如，因为你不想在控制器里写 HTML 代码，你就可以用 ``render()`` 方法来从模板中渲染内容并返回。

在别的章节中，你将学到控制器如何将对象持久化到数据库中或从数据库中获取对象、在子任务中处理、处理缓存还有更多更多。

从技巧书中再学一些
-------------------------------------------------------

* :doc:`/cookbook/controller/error_pages`
* :doc:`/cookbook/controller/service`

.. _`Controller class`: https://github.com/symfony/symfony/blob/master/src/Symfony/Bundle/FrameworkBundle/Controller/Controller.php
