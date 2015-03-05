.. _symfony2-versus-flat-php:

使用 Symfony 与不使用框架的对比
=====================================

**为什么用 Symfony 开发比打开一个文件直接写 PHP 代码更好？**

如果你没有接触过 PHP 框架，也不清楚什么是 MVC，或者对 Symfony 好处的传言感到好奇，那么这一章就是写给你的。在这里我们不会 *告诉* 你 Symfony 可以让你的开发更快速、更好，而是会带你你亲身见证这一切。

在本章中，我们将带你用纯 PHP 写一个简单的应用程序，然后将其重构，使之更有条理。你将会穿越时空，了解为什么网站开发在过去几年中会发生如此翻天覆地的变化。

最后带你体会为什么 Symfony 可以让你摆脱掉一切繁琐，从而真正掌控你的代码。

先用纯 PHP 写一个简单的博客程序
-------------------------------------------------------------

首先，让我们不使用框架写一个博客程序。创建一个来显示数据库里保存的文章的页面。纯 PHP 的话非常简单，但看起来并不舒服：

.. code-block:: html+php

    <?php
    // index.php
    $link = mysql_connect('localhost', 'myuser', 'mypassword');
    mysql_select_db('blog_db', $link);

    $result = mysql_query('SELECT id, title FROM post', $link);
    ?>

    <!DOCTYPE html>
    <html>
        <head>
            <title>文章列表</title>
        </head>
        <body>
            <h1>文章列表</h1>
            <ul>
                <?php while ($row = mysql_fetch_assoc($result)): ?>
                <li>
                    <a href="/show.php?id=<?php echo $row['id'] ?>">
                        <?php echo $row['title'] ?>
                    </a>
                </li>
                <?php endwhile ?>
            </ul>
        </body>
    </html>

    <?php
    mysql_close($link);
    ?>

这样写起来并不费事，运行起来也不慢，但有没有想过随着你程序规模的增大，你该如何维护它。这里列出了几个你可能遇到的问题：

* **没有出错检查**：如果数据库连接失败怎么办？

* **代码结构性差**：随着代码的增多，文件将越来越多
  最后导致你没法继续维护。如果你要处理表单，对应的代码放在
  哪儿？如何验证用户提交上来的数据？发邮件的代码写在
  哪儿呢？

* **代码重复利用率低**：因为所有的代码都写在一个文件里，
  也就没法在这个博客的别的“页面”里重复使用任何一段代码了。

.. note::

    另外一个没有指出的问题是，上面的代码只能用来连接 
    MySQL 数据库。虽然有些超出本章的范围，但还是很想让你知道，Symfony 完整集成了 `Doctrine`_，
    一个提供抽象数据库操作和表字段映射的库。

抽离表现层
~~~~~~~~~~~~~~~~~~~~~~~~~~

现在可以立即将包含了 HTML 代码的“表现层”代码单独保存为一个文件，让表现层与主“逻辑”文件分离：

.. code-block:: html+php

    <?php
    // index.php
    $link = mysql_connect('localhost', 'myuser', 'mypassword');
    mysql_select_db('blog_db', $link);

    $result = mysql_query('SELECT id, title FROM post', $link);

    $posts = array();
    while ($row = mysql_fetch_assoc($result)) {
        $posts[] = $row;
    }

    mysql_close($link);

    // 导入 HTML 表现层文件
    require 'templates/list.php';

现在 HTML 代码都保存在一个独立的文件（``templates/list.php``）中，这个文件在 HTML 代码中嵌入了模板风格的 PHP 代码：

.. code-block:: html+php

    <!DOCTYPE html>
    <html>
        <head>
            <title>文章列表</title>
        </head>
        <body>
            <h1>文章列表</h1>
            <ul>
                <?php foreach ($posts as $post): ?>
                <li>
                    <a href="/read?id=<?php echo $post['id'] ?>">
                        <?php echo $post['title'] ?>
                    </a>
                </li>
                <?php endforeach ?>
            </ul>
        </body>
    </html>

根据惯例，上面的包含所有程序逻辑的文件 ``index.php`` 被称为“Controller（控制器）”。所谓 :term:`controller` 是无论你使用的是语言还是框架都会经常听到的一个术语。简单来讲，它是一块 *你写的* 处理用户输入并准备响应的代码。

在上面的例子里，控制器从数据库里读出数据，然后导入一个模板文件来展现这些数据。通过分离控制器的代码，你将可以轻松地修改模板文件，比如以另外的格式来扩展博客文章的渲染方式（如创建一个对应 JSON 格式的 ``list.json.php`` 模板）。

分离应用程序逻辑（域）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

到目前为止，我们的程序只有一个页面。但是，如果第二个页面需要使用相同的连接数据库的代码或者要用相同的博客文章的数组呢？让我们再次重构代码，将核心的行为和数据访问功能从原来的程序代码中分离出来放入一个叫做 ``model.php`` 的新文件中：

.. code-block:: html+php

    <?php
    // model.php
    function open_database_connection()
    {
        $link = mysql_connect('localhost', 'myuser', 'mypassword');
        mysql_select_db('blog_db', $link);

        return $link;
    }

    function close_database_connection($link)
    {
        mysql_close($link);
    }

    function get_all_posts()
    {
        $link = open_database_connection();

        $result = mysql_query('SELECT id, title FROM post', $link);
        $posts = array();
        while ($row = mysql_fetch_assoc($result)) {
            $posts[] = $row;
        }
        close_database_connection($link);

        return $posts;
    }

.. tip::

   使用 ``model.php`` 来命名刚才的新文件是因为程序逻辑和数据访问
   一般被叫做“Model（模型）”层。在一个代码组织良好的
   程序中，大多数“业务逻辑”的代码
   都在模型层中（而不是控制器中）。不像
   这个例子里的模型层只关注
   访问数据库这一小部分。

现在的控制器（ ``index.php`` ）就很简单了：

.. code-block:: html+php

    <?php
    require_once 'model.php';

    $posts = get_all_posts();

    require 'templates/list.php';

现在控制器的唯一任务就是从模型层中得到数据，然后调用一个模板来渲染这些数据。这就是一个最简单的 MVC 模式。

抽离布局
~~~~~~~~~~~~~~~~~~~~

现在已经把程序重构成三个有着明显不同优势的部分，并且能在不同的页面中重复使用几乎所有的东西。

在代码中唯一 *不能* 被重用的就只有布局了，因此让我们创建一个新的 ``layout.php`` 文件来解决这个问题。

.. code-block:: html+php

    <!-- templates/layout.php -->
    <!DOCTYPE html>
    <html>
        <head>
            <title><?php echo $title ?></title>
        </head>
        <body>
            <?php echo $content ?>
        </body>
    </html>

现在模板文件（``templates/list.php``）可以简单地从基础布局中“扩展”出来。

.. code-block:: html+php

    <?php $title = '文章列表' ?>

    <?php ob_start() ?>
        <h1>文章列表</h1>
        <ul>
            <?php foreach ($posts as $post): ?>
            <li>
                <a href="/read?id=<?php echo $post['id'] ?>">
                    <?php echo $post['title'] ?>
                </a>
            </li>
            <?php endforeach ?>
        </ul>
    <?php $content = ob_get_clean() ?>

    <?php include 'layout.php' ?>

现在你已经知道了重复使用布局的方法。但不幸的是按照现在的思路，你不得不在模板中使用很多丑陋的PHP函数（诸如 ``ob_start()``、``ob_get_clean()``）。在 Symfony 中，可以使用模板组件来让这一切变得更整洁、更方便。你马上就会看到我们如何使用它。

添加一个显示博文的页面
-------------------------------------------------

我们已经重构了博客的“列表”页，使它的代码具有了更好的组织性和可重复使用性。为了检验这一点，让我们添加一个显示博文的页面，来显示被通过 ``id`` 参数标记了的单篇博文。

首先在 ``model.php`` 文件中新增一个函数，用来通过给定的 id 检索单篇博文::

    // model.php
    function get_post_by_id($id)
    {
        $link = open_database_connection();

        $id = intval($id);
        $query = 'SELECT date, title, body FROM post WHERE id = '.$id;
        $result = mysql_query($query);
        $row = mysql_fetch_assoc($result);

        close_database_connection($link);

        return $row;
    }

接下来创建一个新的叫做 ``show.php`` 文件，作为新页面的控制器:

.. code-block:: html+php

    <?php
    require_once 'model.php';

    $post = get_post_by_id($_GET['id']);

    require 'templates/show.php';

最后创建新的模板文件 ``templates/show.php`` ，来渲染单篇博文:

.. code-block:: html+php

    <?php $title = $post['title'] ?>

    <?php ob_start() ?>
        <h1><?php echo $post['title'] ?></h1>

        <div class="date"><?php echo $post['date'] ?></div>
        <div class="body">
            <?php echo $post['body'] ?>
        </div>
    <?php $content = ob_get_clean() ?>

    <?php include 'layout.php' ?>

现在创建第二页已经非常容易了，也没有写重复的代码。然而这一页还有一堆的问题。选择一个框架吧，把这些问题交给它来解决。例如，缺失或无效的 ``id`` 参数会导致页面崩溃。如果能够触发 404 页面将会更好，但做到这一点并不容易。更糟的是，如果你忘记了用 ``intval()`` 函数对 ``id`` 参数进行清理的话，你将会让整个数据库陷入 SQL 注入攻击的危险之中。

另一个问题就是每一个单独的控制器都必须包含 ``model.php`` 文件。如果每个控制器都突然需要包含一个别的文件或者执行其它全局任务（如安全管理）呢？按照目前的情况，这些代码必须添加到每个控制器文件中。如果你忘了包含某个文件，希望这不会给我们带来不安全的因素…

用一个“前端控制器”来解救
--------------------------------------------------------------------

现在，使用 :term:`front controller`: 来解救我们的程序吧，它是一个单独的 PHP 文件，我们可以通过它来处理 *所有* 的请求。有了前端控制器，程序的 URI 略有变化，但开始变得更灵活了：

.. code-block:: text

    没有前端控制器
    /index.php          => 博客的列表页（index.php 被运行）
    /show.php           => 博客的博文展示页（show.php 被运行）

    使用 index.php 作为前端控制器
    /index.php          => 博客的列表页（index.php 被运行）
    /index.php/show          => 博客的博文展示页（index.php 被运行）

.. tip::
    如果使用了 Apache 网页服务器的 rewrite 规则
    （或别的网页服务器的相同功能），URI 中的 index.php 部分就可以省略掉了。这样的话，博客的
    博文展示页的 URI 结果就可以简单地用 ``/show`` 来表示。

当使用前端控制器时，单个 PHP 文件（在这里是 ``index.php`` ）将渲染 *所有的* 请求，对于博文展示页来说， ``/index.php/show`` 最终实际执行的是 ``index.php`` ，它现在负责用完整的 URI 来进行内部路由请求。。如你所见，前端控制器是个非常强大的工具。

制作前端控制器
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

我们就要对程序进行 **重大** 改动了。一旦单个文件接管了所有的请求，你就可以集中精力处理诸如安全、加载配置、路由等等这类事情了。在这个例子里， ``index.php`` 要足够智能，以便根据请求的 URL 区分并渲染博客列表页和博文展示页：

.. code-block:: html+php

    <?php
    // index.php

    // 加载并初始化任何全局库
    require_once 'model.php';
    require_once 'controllers.php';

    // 在内部路由用户的请求
    $uri = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);
    if ('/index.php' == $uri) {
        list_action();
    } elseif ('/index.php/show' == $uri && isset($_GET['id'])) {
        show_action($_GET['id']);
    } else {
        header('Status: 404 Not Found');
        echo '<html><body><h1>页面未找到！</h1></body></html>';
    }

为了更好地组织代码，将两个控制器（之前分别在 ``index.php`` 和 ``show.php``里）写成两个 PHP 函数，并放到新的 ``controllers.php`` 文件里：

.. code-block:: php

    function list_action()
    {
        $posts = get_all_posts();
        require 'templates/list.php';
    }

    function show_action($id)
    {
        $post = get_post_by_id($id);
        require 'templates/show.php';
    }

作为前端控制器， ``index.php`` 扮演了一个新的角色：加载核心库并且路由所有的请求，以便使两个控制器之一（ ``list_action()`` 或 ``show_action()`` 函数）被调用。实际上，前端控制器看来去也变得很像 Symfony 中处理请求和路由请求的机制了。

.. tip::

   前端控制器另一个优点就是可以提供更灵活的 URL 。注意，
   博客显示页的URL只需在一个位置修改一下，
   就可以从 ``/show`` 变成 ``/read`` 。而在此之前，你需要将整个文件
   重命名。在 Symfony 中，URL 将更加灵活。

现在，我们的程序已经从单个的文件发展为拥有良好架构并允许代码重新使用的程序了。你应该觉得高兴，但别感到满意。例如，“路由”系统是多变的，列表页（ ``/index.php`` ）也要可以通过 ``/``来访问（如果添加了 Apache 重写规则的话）。而且，大量的时间花费在“架构”（如路由、控制器和模板等）上，而非花在真正的博客的开发上。你还需要在处理提交上来的表单、验证用户的输入、记录运行日志和安全上花费更多的时间。为什么你要重新发明这些轮子呢？

.. _add-a-touch-of-symfony2:

接触一下 Symfony
~~~~~~~~~~~~~~~~~~~~~~

Symfony 来支援我们啦！在用 Symfony 之前，你要先下载它。你可以用 Composer ，它会给你下载正确的版本并安装相关依赖，而且还提供了一个自动加载器。自动加载器是一个可以让你在没有明确声明包含所用的 PHP 类文件时，就可以使用这个类的一个工具。

在网站的根目录创建 ``composer.json`` 文件并写入以下内容：

.. code-block:: json

    {
        "require": {
            "symfony/symfony": "2.3.*"
        },
        "autoload": {
            "files": ["model.php","controllers.php"]
        }
    }

下一步， `download Composer`_ 并运行以下命令来把 Symfony 下载到 vendor/ 目录下：

.. code-block:: bash

    $ composer install

Composer 在下载依赖的时候会同时生成 ``vendor/autoload.php`` 文件，这个文件会自动装载 Symfony 的所有的文件到 ``composer.json`` 描述的自动装载的文件中。

Symfony 哲学的核心是：程序的主要任务就是解释每个请求并返回对应的响应。因此，Symfony 提供了  :class:`Symfony\\Component\\HttpFoundation\\Request` 和 :class:`Symfony\\Component\\HttpFoundation\\Response`  ，
class. 这两个类是原始的 HTTP 中处理请求和返回响应的面向对象的表述。使用它们来改善我们的博客：

.. code-block:: html+php

    <?php
    // index.php
    require_once 'vendor/autoload.php';

    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\HttpFoundation\Response;

    $request = Request::createFromGlobals();

    $uri = $request->getPathInfo();
    if ('/' == $uri) {
        $response = list_action();
    } elseif ('/show' == $uri && $request->query->has('id')) {
        $response = show_action($request->query->get('id'));
    } else {
        $html = '<html><body><h1>页面未找到！</h1></body></html>';
        $response = new Response($html, 404);
    }

    // 输出响应头并发回响应
    $response->send();

现在控制器可以通过返回一个  ``Response`` 对象来返回响应。为了更加方便，你可以加入一个新的 ``render_template()`` 函数，该函数的行为很像 Symfony 的模板引擎：

.. code-block:: php

    // controllers.php
    use Symfony\Component\HttpFoundation\Response;

    function list_action()
    {
        $posts = get_all_posts();
        $html = render_template('templates/list.php', array('posts' => $posts));

        return new Response($html);
    }

    function show_action($id)
    {
        $post = get_post_by_id($id);
        $html = render_template('templates/show.php', array('post' => $post));

        return new Response($html);
    }

    // 模板渲染帮手函数
    function render_template($path, array $args)
    {
        extract($args);
        ob_start();
        require $path;
        $html = ob_get_clean();

        return $html;
    }

通过运用 Symfony 的一小部分，我们的程序变得更加灵活可靠。``Request`` 类提供了一个访问 HTTP 请求信息的可靠方式。具体来说， ``getPathInfo()`` 方法返回一个被清理过的的 URI（比如它会返回 ``/show`` ，而不会是 ``/index.php/show``）。因此即使用户在地址栏里写的是 ``/index.php/show``，应用程序也会智能地将请求路由到 ``show_action()``。

在构造 HTTP 响应时， ``Response`` 对象十分灵活，它允许通过一个面向对象的接口写入响应头和内容。虽然在我们的这个博客程序中响应是很简单的，但你将体会到当程序增长时这种灵活性将带来的好处。

.. _the-sample-application-in-symfony2:

Symfony 程序示例
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

我们的程序走到现在花了 *很长* 的时间，相信你已经体会到即使这么简单的程序也包含了大量的代码。一路走来，我们制作了简单的路由系统，并且还写了一个使用 ``ob_start()`` 和 ``ob_get_clean()`` 渲染模板的方法。如果在你下一次从零开始搭建“框架”的时候，你至少可以使用 Symfony 中的独立 `Routing`_  和 `Templating`_ 组件，因为它们已经帮你解决了很多问题。

为了不用重新发明轮子，你可以让 Symfony 接管一些部分，下面是我们的程序基于 Symfony 的写法::

    // src/AppBundle/Controller/BlogController.php
    namespace AppBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class BlogController extends Controller
    {
        public function listAction()
        {
            $posts = $this->get('doctrine')
                ->getManager()
                ->createQuery('SELECT p FROM AcmeBlogBundle:Post p')
                ->execute();

            return $this->render('Blog/list.html.php', array('posts' => $posts));
        }

        public function showAction($id)
        {
            $post = $this->get('doctrine')
                ->getManager()
                ->getRepository('AppBundle:Post')
                ->find($id);

            if (!$post) {
                // 抛出 404 错误
                throw $this->createNotFoundException();
            }

            return $this->render('Blog/show.html.php', array('post' => $post));
        }
    }

这两个控制器仍然很轻量，它们都使用 :doc:`Doctrine ORM 库 </book/doctrine>` 从数据库中检索对象，并使用模板组件渲染模板，最后返回 ``Response`` 对象。模板文件现在超级简单：

.. code-block:: html+php

    <!-- app/Resources/views/Blog/list.html.php -->
    <?php $view->extend('layout.html.php') ?>

    <?php $view['slots']->set('title', 'List of Posts') ?>

    <h1>文章列表</h1>
    <ul>
        <?php foreach ($posts as $post): ?>
        <li>
            <a href="<?php echo $view['router']->generate(
                'blog_show',
                array('id' => $post->getId())
            ) ?>">
                <?php echo $post->getTitle() ?>
            </a>
        </li>
        <?php endforeach ?>
    </ul>

布局文件几乎没变：

.. code-block:: html+php

    <!-- app/Resources/views/layout.html.php -->
    <!DOCTYPE html>
    <html>
        <head>
            <title><?php echo $view['slots']->output(
                'title',
                'Default title'
            ) ?></title>
        </head>
        <body>
            <?php echo $view['slots']->output('_content') ?>
        </body>
    </html>

.. note::

    在这里我们将博文展示页面模板留做练习，实现它相对于实现
    博文列表模板来说几乎微不足道。

在 Symfony 引擎（我们称其为 ``Kernel``）启动时，它需要根据一张地图来判断请求信息需要被路由到哪个控制器。所谓的路由表则是一张我们也能读懂的“地图”：

.. code-block:: yaml

    # app/config/routing.yml
    blog_list:
        path:     /blog
        defaults: { _controller: AppBundle:Blog:list }

    blog_show:
        path:     /blog/show/{id}
        defaults: { _controller: AppBundle:Blog:show }

现在 Symfony 就开始处理所有的简单任务了。前端控制器极其简单，它被创建之后你就无须再去接触它了。（如果你使用 Symfony 的发行版，你都无须去创建它）::

    // web/app.php
    require_once __DIR__.'/../app/bootstrap.php';
    require_once __DIR__.'/../app/AppKernel.php';

    use Symfony\Component\HttpFoundation\Request;

    $kernel = new AppKernel('prod', false);
    $kernel->handle(Request::createFromGlobals())->send();

前端控制器的唯一工作就是初始化 Symfony 引擎（``内核``）并把一个需要处理的 ``Request`` 对象传入内核。Symfony 内核再根据路由表来确定调用哪个控制器。和之前一样，控制器方法负责返回最终的 ``Response`` 对象。对它来说就真的没有别的可做的了。

至于 Symfony 如何处理请求，请参阅
:ref:`请求处理流程图  <request-flow-figure>`。

.. _where-symfony2-delivers:

进入 Symfony 的世界
~~~~~~~~~~~~~~~~~~~~~~

在接下来的章节中，我们将学到更多关于 Symfony 的各部分的工作原理，以及推荐的项目组织形式。现在，看看我们的博客程序从纯 PHP 迁移到 Symfony 后有什么优势：

* 现在我们的应用程序代码 **很整洁，组织很好** （虽然
   Symfony 并不强制你做到这一点）。这提高了我们代码的 **重用率** 并且
  可以让新加入项目的开发者很快进入角色；

* 所写的代码100％是为了 *你的* 程序，你 **不再需要
  开发和维护低级的程序了**，比如 :ref:`自动载入 <autoloading-introduction-sidebar>`、
  :doc:`路由 </book/routing>`、 或渲染 :doc:`控制器 </book/controller>`；

* Symfony 可以让你 **使用开源工具** 如 Doctrine 、
  模板、安全、表单、验证组建（只是
  几个例子）；

* 感谢路由组件让我们的程序拥有 **十分灵活的URL**
  ；

* Symfony 以 HTTP 为中心的架构可以让你使用强大的工具，
  例如使用 **Symfony 的内建 HTTP 缓存** 或更为强大的
   `Varnish`_ 来实现 **HTTP 缓存**。这将在稍后的 :doc:`缓存 </book/http_cache>` 一章中进行讲解
  。

最值得高兴的是，通过使用 Symfony，你现在可以获得一整套 Symfony 社区开发的高品质开源工具。想获得 Symfony 社区工具请移步 `KnpBundles.com`_。

更好的模板
--------------------------------------------

Symfony 标配的模板引擎叫 `Twig`_，如果你选择使用它，它将让你可以更快地书写更有可读性的模板。这意味着我们的博客程序可以用更少的代码来写。比如，列表模板用 Twig 写的话是下面的样子：

.. code-block:: html+jinja

    {# app/Resources/views/Blog/list.html.twig #}
    {% extends "layout.html.twig" %}

    {% block title %}文章列表{% endblock %}

    {% block body %}
        <h1>文章列表</h1>
        <ul>
            {% for post in posts %}
            <li>
                <a href="{{ path('blog_show', {'id': post.id}) }}">
                    {{ post.title }}
                </a>
            </li>
            {% endfor %}
        </ul>
    {% endblock %}

同样的， ``layout.html.twig`` 也不难写：

.. code-block:: html+jinja

    {# app/Resources/views/layout.html.twig #}
    <!DOCTYPE html>
    <html>
        <head>
            <title>{% block title %}默认标题{% endblock %}</title>
        </head>
        <body>
            {% block body %}{% endblock %}
        </body>
    </html>

Symfony 很好地支持 Twig。虽然 Symfony 永远支持 PHP 风格模板，但我们将继续讨论 Twig 的更多优势。更多信息请参阅 :doc:`模板章节 </book/templating>`。

从技巧书中再学一些
-------------------------------------------------------

* :doc:`/cookbook/templating/PHP`
* :doc:`/cookbook/controller/service`

.. _`Doctrine`: http://www.doctrine-project.org
.. _`download Composer`: http://getcomposer.org/download/
.. _`Routing`: https://github.com/symfony/Routing
.. _`Templating`: https://github.com/symfony/Templating
.. _`KnpBundles.com`: http://knpbundles.com/
.. _`Twig`: http://twig.sensiolabs.org
.. _`Varnish`: https://www.varnish-cache.org/
.. _`PHPUnit`: http://www.phpunit.de
