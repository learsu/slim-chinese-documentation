#Slim 中文文档

##Slim 安装##

**使用 Composer 进行安装**

首先在你的项目中安装 Composer：

    curl -s https://getcomposer.org/installer | php

然后在你的项目根目录中建立名为 composer.json 的文件：

    {
        "require": {
            "slim/slim": "2.*"
        }
    }

通过 composer 进行安装：

    php composer.phar install

添加下列代码到应用的 index.php 文件中：

    <?php
    require 'vendor/autoload.php';

**手动安装**

下载并解压 Slim 框架到你的项目目录，然后在应用的 index.php 文件中载入 Slim 框架。同时你还需要注册 Slim 框架自带的自动加载类。

    <?php
    require 'Slim/Slim.php';
    \Slim\Slim::registerAutoloader();

##系统依赖

    PHP >= 5.3.0
    如果你需要对cookies进行加密，那么还需要安装 mcrypt 拓展。

##Hello World

生成一个 Slim 实例：

    $app = new \Slim\Slim();

定义一个 HTTP GET 请求路由：

    $app->get('/hello/:name', function($name){
        echo "Hello, $name";
    });

执行 Slim 应用

    $app->run();

##配置

**配置概述**

Slim 框架提供了两种方式对其进行配置。一种是在生成实例的时候进行参数设置，另一种则是在生成实例之后。所有的设置参数都可以在生成实例的时候以数组的形式传递给 Slim 的构造函数（constructor）。所有的设置参数都可以在生成实例之后获取或者修改，但是有些设置并不能简单的只依靠应用实例的 config 函数来完成，因此有必要在后面额外说明。在我罗列这些有效设置参数之前，我想简单的介绍下怎么定义或检查 Slim 应用的参数设置。

**实例生成时**

生成实例时定义设置，只需要向 Slim 的构造器传递一个关联数组。

    <?php
    $app = new Slim(array(
        'debug' => true
    ));

**实例生成之后**

要在实例生成之后定义参数设置，大部分设置都可以使用应用实例的 config 函数；config 函数的第一个参数是设置的名称，第二个参数是设置的参数值。

    <?php
    $app->config('debug', false);

你也可以通过关联数组同时设置多个参数：

    <?php
    $app->config(array(
        'debug' => true,
        'templates.path' => ' ../templates'
    ));

当你想要获取应用某项设置时，你同样可以使用 config 函数；不过这时你只需要传递一个参数 - 你需要检查的设置名称。如果你请求的设置不存在时将返回 null 值。

    <?php
    $settingValue = $app->config('templates.path'); // 返回 "../templates"

你并不仅仅局限在下面这些设置；你也可以定义属于自己的参数设置。

##应用设置

**mode**

它是应用当前操作模式的标识符。mode 设置并不影响 Slim 应用的内部功能。相应的，mode 设置只是为了让你能随意调用自己传递给 configureMode() 函数的函数代码。

应用的 mode 参数和环境变量一样，是在生成应用实例时通过以参数的形式传递给 Slim 应用的构造器（constructor）来声明的，并且不能再次修改。mode 设置参数可以是任何你想要的 --最经典的便是“development”，“test”和“production”，但是你可以随意使用其他想要的（比如：“foo”）。

    <?php
    $app = new \Slim\Slim(array(
        'mode' => 'development'
    ));
    // 数据类型：string，默认值：“development”。

**debug**

    注意！Slim 将错误信息都转换成了 ‘ErrorException’ 实例。

如果调试状态（debugging）被启用，Slim 将会使用 Slim 内置的错误处理函数（error handler）来显示未捕捉异常的诊断信息。如果调试状态被关闭，Slim 将会调用你自定义的错误处理函数，并将未捕捉的异常作为第一个，也是唯一一个参数传递给它。

    <?php
    $app = new \Slim\Slim(array(
        'debug' => true
    ));
    // 数据类型：boolean, 默认值：true
    
**log.writer**

使用自定义的 log writer 可以直接把日志内容输出到需要的目标上。默认情况下，Slim 的 logger 会把日志内容输出到 STDERR。如果你使用自定义的 log writer，它必须实现一下接口：

    public write(mixed $message, int $level);

write() 函数主要负责发送日志信息（不一定是字符串）发送到适当的输出目标（比如：文本文件，数据库或者远程的 web 服务）。

应用实例生成之后再指定 log writer，你就必须直接使用 Slim logger 的 setWriter() 函数：

    <?php
    // During instantiation
    $app = new \Slim\Slim(array(
        'log.writer' => new \My\LogWriter()
    ));

    // After instantiation
    $log = $app->getLog();
    $log->setWriter(new \My\LogWriter());

    // 数据类型：mixed, 默认值：\Slim\LogWriter

**log.level**

    注意！请使用 `\Slim\Log` 中定义的常量来代替数字。

Slim 有5个日志信息等级：

    \Slim\Log::DEBUG
    \Slim\Log::INFO
    \Slim\Log::WARN
    \Slim\Log::ERROR
    \Slim\Log::FATAL

应用设置中的 log.level 设置决定了日志信息中哪些会被关注，哪些会被忽略。比如，如果 log.level 设置是 \Slim\Log::INFO，那么 debug 信息就会被忽略，而 info，warn 和 fatal 信息则会被记录。

应用实例生成之后，你需要直接使用 Slim logger 的 setLevel()函数来修改此设置。

    <?php
    // During instantiation
    $app = new \Slim\Slim(array(
        'log.level' => \Slim\Log::DEBUG
    ));

    // After instantiation
    $log = $app->getLog();
    $log->setLevel(\Slim\Log::WARN);

    // 数据类型：integer，默认值：\Slim\Log::DEBUG

**log.enabled**

这项设置决定 Slim 日志功能的开启和关闭。应用实例生成后，你需要直接使用 SLim logger 的 setEnabled()函数来修改此设置。

    <?php
    // During instantiation
    $app = new \Slim\Slim(array(
        'log.enabled' => true
    ));

    // After instantiation
    $log = $app->getLog();
    $log->setEnabled(true);

    // 数据类型：boolean，默认值：true

**templates.path**

Slim 应用模板文件所在文件目录的相对或者绝对地址。这个路径会在 Slim 应用读取和渲染视图的时候被使用。

应用实例生成之后，你需要直接使用 Slim 视图（view）的 setTemplatesDirectory()函数来修改此设置。

    <?php
    // During instantiation
    $ap = new \Slim\Slim(array(
        'templates.path' => './templates'
    ));

    // After instantiation
    $view = $app->view();
    $view->setTemplatesDirectory('./templates');

    // 数据类型：string，默认值：'./templates'

**view**

试图类（View class）或者其实例都是供 Slim 使用。应用实例生成之后你需要使用 Slim 的 view()函数来修改。

    <?php
    // During instantiation
    $app = new \Slim\Slim(array(
        'view' => new \My\View()
    ));

    // After instantiation
    $app->view(new \My\View());

    // 数据类型：string | \Slim\View，默认值：\Slim\View

**cookies.lifetime**

该设置决定了 Slim 应用创建的 HTTP cookies 生命周期。如果值为整数，它必须是表示 cookie 过期时间的有效 UNIX 时间戳。如果值为字符串，那么其值经过 strtotime() 函数转换后也必须是 cookie 过期时间的有效 UNIX 时间戳。

    <?php
    // During instantiation
    $app = new \Slim\Slim(array(
        'cookies.lifetime' => '20 minutes'
    ));

    // After instantiation
    $app->config('cookies.lifetime', '20 minutes');

    // 数据类型：integer|string，默认值：'20 minutes'

**cookies.path**

如果 Slim 应用在调用 setCookie() 或者 setEncryptedCookie() 函数时没有指定 path 参数，那么该设置就是 HTTP cookie 的默认路径。

    <?php
    // During instantation
    $app = new \Slim\Slim(array(
        'cookies.path' => '/'
    ));

    // After instantiation
    $app->config('cookies.path', '/');

    // 数据类型：string，默认值：'/'

**cookies.domain**

如果 Slim 应用在调用 setCookie() 或者 setEncryptedCookie() 函数时没有指定 domain 参数，那么该设置就是 HTTP domain 的默认值。

    <?php
    // During instantiation
    $app = new \Slim\Slim(array(
        'cookies.domain' => 'domain.com'
    ));

    // After instantiation
    $app->config('cookies.domain', 'domain.com');

    // 数据类型：string，默认值：null

**cookies.secure**

该设置决定 cookies 是否只在 HTTPS 请求中发送。你可以通过 setCookie() 或者 setEncryptedCookie() 函数的 secure 参数来覆盖该设置。

    <?php
    // During instantiation
    $app = new \Slim\Slim(array(
        'cookies.secure' => false
    ));

    // After instantiation
    $app->config('cookies.secure', false);

    // 数据类型：boolean，默认值：false

**cookies.httponly**

该设置决定 cookies 是否只在 HTTP 协议中发送。你可以通过调用 Slim 应用的 setCookie() 或者 setEncryptedCookie() 函数来覆盖该设置。

    <?php
    // During instantiation
    $app = new \Slim\Slim(array(
        'cookies.httponly' => false
    ));

    // After instantiation
    $app->config('cookies.httponly', false);

    // 数据类型：boolean，默认值：false

**cookies.secret_key**

供 cookie 加密的加密关键词。当你的 Slim 应用开启了 HTTP cookies 加密，务必要修改此设置。

    <?php
    // During instantiation
    $app = new \Slim\Slim(array(
        'cookies.secret_key' => 'secret'
    ));

    // After instantiation
    $app->config('cookies.secret_key', 'secret');

    // 数据类型：string，默认值：'CHANGE_ME'

**cookies.cipher**

该加密密匙供 HTTP cookie 加密使用。查看[现有密匙](http://php.net/manual/en/mcrypt.ciphers.php)。

    <?php
    // During instantiation
    $app = new \Slim\Slim(array(
        'cookies.cipher' => MCRYPT_DIJNDAEL_256
    ));

    //After instantiation
    $app->config('cookies.cipher', MCRYPT_DIJNDAEL_256);

    // 数据类型：integer，默认值：MCRYPT_DIJNDEAL_256

**cookies.cipher_mode**

该密匙模式供 HTTP cookie 加密使用。查看[现有密匙模式](http://php.net/manual/en/mcrypt.ciphers.php)。

    <?php
    // During instantiation
    $app = new \Slim\Slim(array(
        'cookies.cipher_mode' => MCRYPT_MODE_CBC
    ));

    // After instantiation
    $app->config('cookies.cipher_mode', MCRYPT_MODE_CBC);

    // 数据类型：integer，默认值：MCRYPT_MODE_CBC

**http.version**

默认情况下，Slim 会想客户端返回 HTTP/1.1 首行。如果你需要返回 HTTP/1.0，可以修改此设置。当你使用 PHPFog 或 nginx 服务器，要使用代理而非直接与 HTTP 客户端通信时，此设置尤为有用。

    <?php
    // During instantiation
    $app = new \Slim\Slim(array(
        'http.version' => '1.1'
    ));

    // After instantiation
    $app->config('http.version', '1.1');

    // 数据类型：string，默认值：'1.1'

##应用名称与作用域

当你建立了一个 Slim 应用时，可能会在代码中用到多种作用域（比如全局作用域和函数作用域）。你应该希望在任何作用域中引用你的 Slim 应用。这里提供了两种方式：

    - 把应用名称传递给 Slim 应用的 getInstance() 静态函数。
    - 通过使用 use 关键词 把应用实例导入到函数作用域。

**应用名称**

你可以为每一个 Slim 应用取名。这是可选的。应用名称可以帮助你在代码的任何作用域中引用 Slim 应用，下面是设置和获取应用名称的方法：

    <?php
    $app = new \Slim\Slim();
    $app->setName('foo');
    $name = $app->getName(); // 'foo'
    
**作用域解析**

那么你怎样才可以获取 Slim 应用的引用呢？下面这个例子演示了如何在路由的回调函数中获取一个 Slim 应用的引用。$app 变量已经在定义 HTTP GET路由的全局作用域中使用。但是在路由回调函数的函数作用域中也需要使用 $app 变量来渲染模板。

    <?php
    $app = new \Slim\Slim();
    $app->get('/foo', function(){
        $app->render('foo.php'); // <---- 错误
    });

这个例子运行错误是因为 $app 变量在路由回调函数的函数作用域中是未定义的。

**柯里化（Currying）**

我们可以使用 use 关键词把 $app 变量注入到回调函数的作用域中：

    <?php
    $app = new \Slim\Slim();
    $app->get('/foo', function() use($app){
        $app->render('foo.php'); // <---- 成功
    });

**按名称获取**

你也可以在 Slim 应用中使用 getInstance() 静态函数：

    <?php
    $app = new \Slim\Slim();
    $app->get('/foo', 'foo');
    function foo(){
        $app = Slim::getInstance();
        $app->render('foo.php');
    }

##应用模式

一般来说，网页应用运行在指定模式都决定于项目当前的状态。如果你在开发应用，那么你就应该在“development”模式中运行应用；如果你在测试应用，你就应该在“testing”模式中运行应用；如果你发布了应用，那么你的应用在“production”模式中运行应用。

Slim 提供了应用模式的概念，你可以定义需要的模式以便 Slim 为当前模式的运行作适当的准备。 比如，你想要在“development”模式中开启调试（debugging）功能，但需在“production”模式中关闭。下面的例子向您演示了如何为 Slim 设置不同的模式。

**什么是模式**

从技术上来说，应用模式只不过是由字符串组成的文字，就像“development”或者“production”，有一个与之相关联的回调函数为 Slim 应用的运行作适当的配置。应用模式可以是任何名字，就比如“testing”，“production”，“development”甚至是“foo”。

**我应该如何设置应用模式？**

    注意！应用模式只能在应用实例化时设置，之后无法更改。

**使用环境变量**

如果 Slim 检测到名为 “SLIM_MODE” 的环境变量，它就会把应用模式设置为该值。

    <?php
    $_ENV['SLIM_MODE'] = 'production';

**使用应用设置**

如果环境变量未定义时，Slim 会继续检查应用设置中的模式参数。

    <?php
    $app = new \Slim\Slim(array(
        'mode' => 'production'
    ));

**默认模式**

如果环境变量和应用设置中均未定义模式，Slim 就会将应用模式设置为“development”。

**配置指定模式**

当你实例化一个 Slim 应用之后，你可以使用 Slim 应用的 configureMode() 函数对 Slim 应用的指定模式进行配置。这个函数接受两个参数：目标模式的名称以及当前应用模式与第一个参数相匹配时需要执行的回调函数。

假如应用当前模式为“produciton”，那么只有与“production”模式相关联的回调函数会被调用。而与“development”模式关联的回调函数将会被忽略，直到应用模式改变为“development”。

    <?php
    // 设置当前模式
    $app = new \Slim\Slim(array(
        'mode' => 'production'
    ));

    // 只在“production”模式中执行
    $app->configureMode('production', function() use($app){
        $app->config(array(
            'log.enable' => true,
            'debug' => false
        ));
    });

    // 只在“development”模式中执行
    $app->configureMode('development', function() use($app){
        $app->config(array(
            'log.enable' => false,
                'debug' => true
        ));
    });

##路由

**路由概述**

Slim 框架帮助你将应用 URIs 映射到特定 HTTP 请求方法（比如：GET，POST，PUT，DELETE，OPTION 或 HEAD）的回调函数上。Slim 应用将会调用与当前 HTTP 请求 的 URI 和方法相匹配的第一个路由。

如果 Slim 应用未找到与 HTTP 请求 URI 和方法相配备的路由，它将自动返回一个 404 Not Found 响应。

**GET 路由**

使用 Slim 应用的 get() 函数会将一个回调函数映射到 一个由 HTTP GET 方法请求的资源 URI。

    <?php
    $app = new \Slim\Slim();
    $app->get('/book/:id', function($id){
        // 根据书籍 $id 显示数据
    });

在这个例子中，当使用 HTTP GET 向“/books/1” 发送请求时，与它相关联的回调函数将会被执行，并会把“1”作为参数传递给回调函数。

Slim 应用 get() 函数的第一个参数是资源 URI。最后一个参数是经 is_callable() 检测后会返回 true 的任何东西。通常来说，最后一个参数是一个[匿名函数](http://php.net/manual/en/functions.anonymous.php)。

**POST 路由**

使用 Slim 应用的 post() 函数会将一个回调函数映射到 一个由 HTTP POST 方法请求的资源 URI。

    <?php
    $app = new \Slim\Slim();
    $app->post('/books', function(){
        // 创建书籍
    });

在这个例子中，当使用 HTTP POST 向“/books”发送请求时，与它相关的回调函数将会被执行。

Slim 应用 post() 函数的第一个参数是资源 URI。最后一个参数是经 is_callable() 检测后会返回 true 的任何东西。通常来说，最后一个参数是一个[匿名函数](http://php.net/manual/en/functions.anonymous.php)。

**PUT 路由**

使用 Slim 应用的 put() 函数会将一个回调函数映射到 一个由 HTTP PUT 方法请求的资源 URI。

    <?php
    $app = new \Slim\Slim();
    $app->put('/books/:id', function($id){
        // 更新 $id 对应的书籍
    });

在这个例子中，当使用 HTTP PUT 向“/books/1” 发送请求时，与它相关联的回调函数将会被执行，并会把“1”作为参数传递给回调函数。

Slim 应用 put() 函数的第一个参数是资源 URI。最后一个参数是经 is_callable() 检测后会返回 true 的任何东西。通常来说，最后一个参数是一个[匿名函数](http://php.net/manual/en/functions.anonymous.php)。

**Method 覆盖**

不幸的是当前的浏览器并不原生支持 HTTP PUT 请求。为了解决这个问题，请确保你的 HTML 表单的请求方式为“post”，然后像下面例子一样，在 HTML 表单中添加一项请求方式覆盖参数：

    <form action="/books/1" method="post">
        ... 其他表单域 ...
        <input type="hidden" name="_METHOD" value="PUT" />
        <input type="submit" value="Update Book"/>
    </form>

如果你正在使用 [Backbone.js](http://documentcloud.github.com/backbone/) 或者 HTTP 命令行客户端，你同样可以使用 X-HTTP-Method-Override 请求头来覆盖 HTTP 请求方式。

**DELETE 路由**

使用 Slim 应用的 delete() 函数会将一个回调函数映射到 一个由 HTTP DELETE 方法请求的资源 URI。

    <?php
    $app = new \Slim\Slim();
    $app->delete('/books/:id', function($id){
        // 删除 $id 对应的书籍
    });

在这个例子中，当使用 HTTP DELETE 向“/books/1” 发送请求时，与它相关联的回调函数将会被执行，并会把“1”作为参数传递给回调函数。

Slim 应用 delete() 函数的第一个参数是资源 URI。最后一个参数是经 is_callable() 检测后会返回 true 的任何东西。通常来说，最后一个参数是一个[匿名函数](http://php.net/manual/en/functions.anonymous.php)。

**Method 覆盖**

不幸的是当前的浏览器并不原生支持 HTTP DELETE 请求。为了解决这个问题，请确保你的 HTML 表单的请求方式为“post”，然后像下面例子一样，在 HTML 表单中添加一项请求方式覆盖参数：

    <form action="/books/1" method="post">
        ... 其他表单域 ...
        <input type="hidden" name="_METHOD" value="DELETE" />
        <input type="submit" value="Update Book"/>
    </form>

如果你正在使用 [Backbone.js](http://documentcloud.github.com/backbone/) 或者 HTTP 命令行客户端，你同样可以使用 X-HTTP-Method-Override 请求头来覆盖 HTTP 请求方式。

**OPTIONS 路由**

使用 Slim 应用的 options() 函数会将一个回调函数映射到 一个由 HTTP DELETE 方法请求的资源 URI。

    <?php
    $app = new \Slim\Slim();
    $app->options('/books/:id', function($id){
        // 返回响应头
    });

在这个例子中，当使用 HTTP OPTIONS 向“/books/1” 发送请求时，与它相关联的回调函数将会被执行，并会把“1”作为参数传递给回调函数。

Slim 应用 options() 函数的第一个参数是资源 URI。最后一个参数是经 is_callable() 检测后会返回 true 的任何东西。通常来说，最后一个参数是一个[匿名函数](http://php.net/manual/en/functions.anonymous.php)。

**Method 覆盖**

不幸的是当前的浏览器并不原生支持 HTTP OPTIONS 请求。为了解决这个问题，请确保你的 HTML 表单的请求方式为“post”，然后像下面例子一样，在 HTML 表单中添加一项请求方式覆盖参数：

    <form action="/books/1" method="post">
        ... 其他表单域 ...
        <input type="hidden" name="_METHOD" value="OPTIONS" />
        <input type="submit" value="Update Book"/>
    </form>

如果你正在使用 [Backbone.js](http://documentcloud.github.com/backbone/) 或者 HTTP 命令行客户端，你同样可以使用 X-HTTP-Method-Override 请求头来覆盖 HTTP 请求方式。

**自定义 HTTP 方法**

**一个路由对应多个 HTTP 方法**

有时候我可能需要通过一个路由响应多个 HTTP 方法；有时候我可能需要一个响应自定义 HTTP 方法的路由。你可以使用路由对象的 via() 函数来完成。这个例子演示了如何将一个资源 URI 映射到可以响应多种 HTTP 方法请求的回调函数。

    <?php
    $app = new \Slim\Slim();
    $app->map('/foo/bar', function(){
        echo "I respond to multiple HTTP methods!";
    })->via('GET', 'POST');
    $app->run();

这个例子中定义的路由将会同时响应请求资源定位符 “/foo/bar” 的 GET 和 POST 两种方法。只需要把每个 HTTP 请求方法以字符串的形式作为参数传递给路由对象 via() 函数。同路由的其他函数一样（比如：name() 和 conditions()），via() 函数也支持链式调用。

    <?php
    $app = new \Slim\Slim();
    $app->map('/foo/bar', function(){
        echo "Fancy,huh?";
    })->via('GET', 'POST')->name('foo');
    $app->run();

**路由参数**

你可以在路由的资源 URIs 中加入参数。在这个例子中，我在路由 URI 中加入了两个参数：“:one”和“:two”。

    <?php
    $app = new \Slim\Slim();
    $app->get('/books/:one/:two', function($one, $two){
        echo "The first paramter is " . $one;
        echo "The second paramter is " . $two;
    });

创建路由参数时，只需要在路由 URI 的参数前面加上“:”。当路由和当前 HTTP 请求相匹配时，路由中的每个参数都会从 HTTP 请求 URI 中提取出来，并把它们按照适当的顺序传递给相关联的回调函数。

**通配符路由参数**

你也可以在路由中使用通配符参数。与通配符参数相匹配的一个或者多个 URI 字段将会被捕捉并以数组的形式保存。通配符参数的标志就是以“+”为后缀，其他地方和前面讲过的普通参数相同。下面是一个例子：

    <?php
    $app = new \Slim\Slim();
    $app->get('/hello/:name+', function($name){
        // Do something
    });

当你用“/hello/Josh/T/Lockhart”作为资源 URI 调用上面这个应用，回调函数的 $name 参数就等于 array('Josh', 'T', 'Lockhart')。

**可选路由参数**

    注意！路由的可选字段还在实验中。只应当用如下演示例子的方式使用。

你也可以定义可选路由参数。这是在博客归档时使用可选路由参数的例子，你可以用例子中的方式来指定可选路由参数：

    <?php
    $app = new \Slim\Slim();
    $app->get('/archive(/:year(/:month(/:day)))', function($year = 2010, $month = 12, $day = 05){
        echo sprintf('%s-%s-%s', $year, $month, $day);
    });

每个路由字段都是可选的，这个路由将会接受如下 HTTP 请求：

    - /archive
    - /archive/2010
    - /archive/2010/12
    - /archive/2010/12/05

如果一个 HTTP 请求的可选参数被省略，回调函数中定义的默认值会被代替使用。

目前你只能如以上示例定义可选参数的方式来设置路由字段。如果使用和示例中不同的方式，你可以发现它并不能稳定运行。

**路由名称**

Slim 允许你为路由命名。为路由命名后你可以使用 urlFor 方法动态生成 URL。当你使用 Slim 的 urlFor() 方法来创建应用 URL，你可以自由的在不破坏应用的情况下修改路由参数。下面是一个命名路由：

    <?php
    $app = new \Slim\Slim();
    $app->get('/hello/:name', function($name){
        echo "Hello, $name!";
    })->name('hello');

现在你可以使用 urlFor() 函数来生成 URLs，这点会在后面讲解。路由的 name() 函数同样支持链式调用：

    <?php
    $app = new \Slim\Slim();
    $app->get('/hello/:name', function($name){
        echo "Hello, $name!";
    })->name('hello')->conditions(array('name' => '\w+'));

**路由条件**

Slim 可以让你指定路由参数的满足条件。如果未满足指定条件，那么路由就不会执行。例如，如果你需要一个路由，它的第二个字段 year 必须是4个整数，你可以这样设定条件：

    <?php
    $app = new \Slim\Slim();
    $app->get('archive/:year', function($year){
        echo "You are viewing archives from $year";
    })->conditions(array('year' => '(19|20)\d\d'));

调用路由对象的 conditions() 函数，第一个也是唯一一个参数是一个关联数组，关键字是所有匹配的路由参数，值为正则表达式。

**应用泛路由条件**

如果你 Slim 应用的很多路由都接受同样的参数和使用同样的条件，你就可以像这样定义一个默认的泛路由条件：

    <?php
    \Slim\Route::setDefaultConditions(array(
        'firstName' => '[a-zA-Z]{3,}'
    ));

在你定义应用路由之前定义应用的泛路由条件。当你定义了一个路由，它将自动指定由 \Slim\Route::setDefaultCondtions() 定义的所有泛路由条件。不管出于什么原因，你都可以使用 \Slim\Route::getDefaultConditions() 来获取已定义的应用泛路由条件。这个函数将返回所有已定义的默认路由条件。

当定义路由时，你可以像下面一样通过重新定义路由条件来覆盖默认路由条件：

    <?php
    $app = new \Slim\Slim();
    $app->get('/hello/:firstName', $callable)
        ->conditions(array('firstName' => '[a-z]{10,}'));

你也可以通过如下方式来为路由追加条件：

    <?php
    $app = new \Slim\Slim();
    $app->get('/hello/:firstName/:lastName', $callable)
        ->conditions(array('lastName' => '[a-z]{10,}'));

**路由中间件**

Slim 可以让你为指定路由关联中间件。当路由和当前 HTTP 请求相匹配被调用时，Slim 将在第一时间按顺序调用与之相关联的中间件。

**什么是路由中间件**

路由中间件是 is_callable 会返回 true 的任何事物。路由中间件会在路由回调函数被调用前按顺序被执行。

**如何添加路由中间件**

当你在使用 Slim 应用的 get()，post()，put()，delete() 函数定义一个新的应用路由时，你必须定义一个路由模式以及当路由和 HTTP 请求匹配时调用的回调函数。

    <?php
    $app = new \Slim\Slim();
    $app->get('/foo', function(){
        // Do something
    });

在上面的例子中，第一个参数是路由模式，最后一个参数时与当前 HTTP 请求匹配时要调用的回调函数。路由模式必须时第一个参数，回调函数必须是最后一个参数。

你可以把每个中间件分别作为中间参数传递给路由来指定中间件：

    <?php
    function mw1(){
        echo "This is middleware!";
    }
    function mw2(){
        echo "This is middleware!";
    }
    $app = new \Slim\Slim();
    $app->get('/foo', 'mw1', 'mw2', function(){
        // Do something
    });

当/foo 路由被调用时，mw1 和 mw2 函数也将会在路由回调函数调用之前按顺序执行。

假如你想要在路由中对当前用户进行认证，你可以像下面一样使用匿名函数：

    <?php
    $authenticateForRole = function($role = 'member') {
        return function() use($role){
            $user = User::fetchFromDatabaseSomehow();
            if ($user->belongsToRole($role) === false) {
                $app = \Slim\Slim::getInstance();
                $app->flash('error', 'Login required');
                $app->redirect('/login');
            }
        }
    };
    $app = new \Slim\Slim();
    $app->get('/foo', $authenticateForRole('admin'), function(){
        // Display admin control panel
    });

**什么参数会被传递给每个路由中间件函数？**

每个中间件函数在被调用时都可以传递一个参数，这个例子里是 \Slim\Route 对象。

    <?php
    $aBitOfInfo = function(\Slim\Route $route){
        echo "Current route is " . $route->getName();
    };
    $app->get('/foo', $aBitOfInfo, function(){
        echo "foo";
    });

**路由辅助函数**

Slim 提供了一些辅助函数（通过 Slim 实例调用）来帮助你控制应用流程。

你应该意识到下面这些应用实例的辅助函数 halt()，pass()，redirect()，stop() 都是通过异常（Exceptions）来执行的。每个都会抛出 \Slim\Exception\Stop 或者 \Slim\Exception\Pass 异常。在这些情况下，抛出异常是让你的代码停止运行的简单方法，框架会接受并发送必要的响应信息到客户端。如果没有正确认识到这点，可能会得到奇怪结果。看一下下面的代码：

    <?php
    $app->get('/', function() use($app, $obj){
        try {
            $obj->thisMightThrowException();
            $app->redirect('/success');
        } catch(\Exception $e) {
            $app->flass('error', $e->geMessage());
            $app->redirect('/error');
        }
    });

如果 $obj->thisMightThrowException() 抛出了一个异常，那么代码会按预想的情况运行。但是如果它没有抛出异常，$app->redirect('/success')在调用时也会抛出一个 \Slim\Exception\Stop 异常并被 catch 模块捕获，然后框架会将浏览器跳转到“/error”页面。你应该在应用需要抛出异常的地方使用不同类型的异常，那么 catch 模块就会捕捉指定异常而不是捕获所有的异常。在一些特殊情况下，thisMightThrowException 可能是你无法控制的外部组件，在这种情况下要抛出所有类型的异常是不可能的。那么我们可以细微的调整下代码，把 $app->redirect('/success') 移动到 try/catch 模块后面来解决这个问题。现在代码会按照预想的流程执行，除非捕获到 error redirect 抛出的异常。

    <?php
    $app->get('/', function() use($app, $obj){
        try {
            $obj->thisMightThrowException();
        } catch(Exception $e) {
            $app->flash('error', $e->getMessage());
            $app->redirect('/error');
        }
        $app->redirect('/success');
    });

**Halt**

Slim 应用的 halt() 函数会立即返回给定的 HTTP 响应状态码（status code）和报文主体（body）。该函数接受两个参数：HTTP 状态码和可选消息。Slim 会立即停止当前应用并把HTTP 响应以及指定的状态码和可选消息（作为响应正文）发送到客户端。它会覆盖已存在的 \Slim\Http\Response 对象。

    <?php
    $app = new \Slim\Slim();

    // Send a default 500 error response
    $app->halt(500);

    // Or if you encouter a balrog
    $app->halt(403, 'You shall not pass!');

如果你需要用错误信息列表来渲染一个模板，你应该使用 Slim 应用的 render() 函数来代替。

    <?php
    $app = new \Slim\Slim();
    $app->get('/foo', function() use($app){
        $errorData = array('error' => 'Permission Denied');
        $app->render('errorTemplate.php', $errorData, 403);
    });
    $app->run();

halt() 函数可以发送任何类型的 HTTP 响应到客户端：infomational，success，redirect，not found，client error，或者 server error。

**Pass**

一个路由可以使用 pass() 函数告诉 Slim 应用继续执行下一个匹配的路由。当这个函数被调用，Slim 应用会立即停止执行当前匹配路由并直接调用下一个匹配路由。如果没有后续路由被匹配，就会像客户端发送一个 **404 Not Found** 响应。下面是一个例子，假如向“/hello/Frank”发送一个 HTTP GET请求：

    <?php
    $app = new \Slim\Slim();
    $app->get('/hello/Frank', function() use($app){
        echo "You wont see this ...";
        $app->pass();
    });
    $app->get('/hello/:name', function($name) use($app){
        echo "But you will see this!";
    });
    $app->run();

**Redirect**

使用 Slim 应用的 redirect() 函数可以非常容易的让客户端跳转到另一个 URL。这个函数接受两个参数：第一个参数是客户端即将跳转的 URL；第二个可选参数是 HTTP 状态码。默认情况下，redirect() 函数会发送一个302临时跳转响应。

    <?php
    $app = new \Slim\Slim();
    $app->get('/foo', function() use($app){
        $app->redirect('/bar');
    });
    $app->run();

但是如果你想要使用永久跳转，你必须指定把目标 URL 作为第一个参数然后把 HTTP 状态码作为第二个参数。

    <?php
    $app = new \Slim\Slim();
    $app->get('/old', function() use($app){
        $app->redirect('/new', 301);
    });
    $app->run();

这个函数会自动设置 Location:header。HTTP 跳转响应会被立即发送到客户端。

**Stop**

Slim 应用的 stop() 函数会停止 Slim 应用并发送当前 HTTP 响应到客户端。不会考虑到其他情况。

    <?php
    $app = new \Slim\Slim();
    $app->get('/foo', function () use ($app) {
        echo "You will see this...";
        $app->stop();
        echo "But not this";
    });
    $app->run();

**URL For**

Slim 的 urlFor() 方法可以让你为一个已命名路由动态的创建 URL，当路由匹配模式改变时，你的 URL 也会自动更新，不必担心会影响到应用程序。下面的例子演示如何为命名路由生成 URL：

    <?php
    $app = new \Slim\Slim();
    // Create a named route
    $app->get('/hello/:name', function ($name) use ($app) {
        echo "Hello $name";
    })->name('hello');
    // Generate a URL for the named route
    $url = $app->urlFor('hello', array('name' => 'Josh'));

在这个例子中，$url 的值是“/hello/Josh”。当你使用 urlFor() 方法时，你必须先为路由命名，然后再调用 urlFor() 方法。它的第一个参数是路由名称，第二个参数是一个关联数组，它包含替换路由 URL 所需要的实际值；数组的 key 必须和路由 URI 中的参数匹配，数组的 value 会作为替换值。

**路由 URL 重写**

我强烈推荐你使用支持 URL 重写的web服务器；这样你可以在 Slim 应用中使用简洁而友好的 URLs。要使用 URL 重写功能，你需要使用服务器提供的相关工具来转发所有的 HTTP 请求到 你实例化和运行 Slim 应用的 PHP 文件上。下面是 使用 mod_php 的Apache 和 nginx 最小配置示例。这些设置并不能满足生产环境需要，但是足够让你的应用尽快运行起来。你可以继续阅读[http://www.phptherightway.com](http://www.phptherightway.com)来获取更多服务器部署相关信息。

**Apache 和 mod_rewrite**

这是一个目录结构示例：

    /path/www.mysite.com/
        public_html <-- Document root!
            .htaccess
            index.php <-- I instantiate Slim here
        lib/
            slim/ <-- I store Slim lib files here!

目录中的 .htaccess 文件包含一下内容：

    RewriteEngine On
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [QSA,L]

同时你也需要一个目录指令让 .htaccess文件 和 RewriteEngin 可以被使用。这个设置虽然可以在全局的 httpd.conf 文件中完成，但是通过将指令封装到虚拟主机设置模块的方式把指令限制在虚拟主机权限类是最好的办法。下面是设置的大致形式：

    <VirtualHost *:80>
        ServerAdmin me@mysite.com
        DocumentRoot "/path/www.mysite.com/public_html"
        ServerName mysite.com
        ServerAlias www.mysite.com

        #ErrorLog "logs/mysite.com-error.log"
        #CustomLog "logs/mysite.com-access.log" combined

        <Directory "/path/www.mysite.com/public_html">
            AllowOverride All
            Order allow,deny
            Allow from all
        </Directory>
    </VirtualHost>

现在 Apache 会将所有非文件请求转发到我 实例化和运行 Slim 应用的 index.php 脚本上。开启 URL 重写之后，以下面 index.php 中定义的 Slim 应用为例。你可以在应用路由中使用“/foo”来代替“/index.php/foo”。

    <?php
    $app = new \Slim\Slim();
    $app->get('/foo', function(){
        echo "Foo";
    });
    $app->run();

**nginx**

我们使用和上面同样的目录结构，只是把配置文件换为 nginx 的配置文件 nginx.conf。

    /path/www.mysite.com/
    public_html/ <-- Document root!
        index.php <-- I instantiate Slim here!
    lib/
        Slim/ <-- I store Slim lib files here!

下面是 nginx.conf 文件中我们使用 try_files 指令的位置片段，它让服务器处理实际存在的静态文件（图片，css，js 等）请求而把其他的都转发到 index.php 文件上。

    server {
        listen       80;
        server_name  www.mysite.com mysite.com;
        root         /path/www.mysite.com/public_html;

        try_files $uri /index.php;

        # this will only pass index.php to the fastcgi process which is generally safer but
        # assumes the whole site is run via Slim.
        location /index.php {
            fastcgi_connect_timeout 3s;     # default of 60s is just too long
            fastcgi_read_timeout 10s;       # default of 60s is just too long
            include fastcgi_params;
            fastcgi_pass 127.0.0.1:9000;    # assumes you are running php-fpm locally on port 9000
        }
    }

大多数情况下安装后会有一个默认的 fastcgi_params 文件，因此你只需要把上面的配置包含进去即可。不过有些配置文件中没有包含 SCRIPT_FILENAME 参数。你必须确保包含了该参数，否则可能被终止运行并收到来自 fastcgi 程序的未指定输入文件错误。这个问题可以直接在 location block 中添加或者简单的把参数添加到 fastcgi_params 文件中。无论哪种方式，它应该看起来像这样：

    fastcgi_param SCRIPT_FILENAME $docment_root$fastcgi_script_name;

**不使用 URL 重写**

Slim 也可以在不使用 URL 重写的情况下使用。在这种脚本中，你必须在初始化和运行Slim 应用的资源 URI中包含 PHP 文件名。举个例子，假如下面的 Slim 应用定义在你虚拟主机根目录的 index.php 文件中：

    <?php
    $app = new \Slim\Slim();
    $app->get('/foo', function(){
        echo "Foo!";
    });
    $app->run();

你可以使用“index.php/foo”来访问定义的路由。如果同样的应用被定义在 blog/ 子目录的 index.php 文件中，你可以通过 “/blog/index.php/foo” 来访问定义的路由。


##环境##

**环境概述**

Slim 框架的环境变量实现来自于 [Rack protocol](http://rack.rubyforge.org/doc/SPEC.html)。当你实例化一个 Slim 应用，它会立即检查超全局变量 $\_SERVER 并获取决定应用运行的环境变量。

**什么是环境？**

Slim 应用的环境是指一系列解析后能够被 Slim 应用和它的中间件访问的设置的关联数组。你可以在运行时自由的修改环境变量；设置的变动会立即传递给整个应用。

当你实例化一个 Slim 应用，环境变量获取于超全局变量 $\_SERVER，你不需要自己设置。但是你可以在 Slim 中间件中自由的修改或添加这些变量。

这些变量基本上决定了 Slim 应用的运行：资源 URI，HTTP 请求方式，HTTP 请求正文，URL 查询参数，错误输出等等。后面将讲到的中间件会给你提供足够的能力在 Slim 应用运行前或运行后来操纵这些环境变量。

**环境变量**

下面这些内容基本上都借鉴于 [http://rack.rubyforge.org/doc/files/SPEC.html](http://rack.rubyforge.org/doc/files/SPEC.html)上已有的信息。环境变量必须包含这些变量：

    REQUEST_METHOD
        HTTP 请求方式，必须包含这个变量且不能未空字符。
    SCRIPT_NAME
        URI 请求路径的初始部分，它和 Slim 应用的实际安装目录一致，因此应用知道它的虚拟位置。如果应用被安装在 public 目录的根目录这个变量可能是空值。变量后面没有斜线。
    PATH_INFO
        URI 请求路径的剩余部分，它决定 HTTP 请求目标资源在 Slim 应用中的虚拟位置。它总是以斜线开头，末尾斜线可有可无。
    QUERY_STRING
        HTTP 请求 URL 之后不包含“?”的部分。这个变量是必要的，但是可以是空字符。
    SERVER_NAME
        当把它和 SCRIPT_NAME 以及 PATH_INFO 连接在一起，就可以建立一个指向应用资源的完整 URL。但是，如果设有 HTTP_HOST，那么应该使用 HTTP_HOST 来代替该变量。该变量是必要的且不能为空。 
    SERVER_PORT
        当把它和 SCRIPT_NAME 以及 PATH_INFO 连接在一起，就可以建立一个指向应用资源的完整 URL。该变量是必要的且不能为空。
    HTTP_*
        与客户端发送的 HTTP 请求首部字段相匹配的变量。这些变量与当前 HTTP 请求发送的相一致。
    slim.url_scheme
        根据 HTTP 请求 URL 来决定变量是“http”或者“https”。
    slim.input
        以字符串形式表现的原始的 HTTP 请求报文主体。如果 HTTP 请求报文主体为空（比如：GET 请求），该变量也为空。
    slim.errors
        必须是可写入的资源；默认情况下，该变量为 php://stderr 的一个可写资源句柄。

Slim 应用也可以把自身数据储存到环境中。环境变量数组的关键词必须至少包含一个“.”点号以及一个独特的前缀（比如“prefix.foo”）。slim 前缀是 Slim 应用的保留前缀，不允许被 Slim 自身以外的其他使用。CGI 关键词的值必须是字符串。下面是一些相关限制：

    - slim.url_scheme 必须是“http”或者“https”。
    - slim.input 必须是字符串。
    - slim.errors 必须是一个有效可写的资源。
    - REQUEST_METHOD 必须是有效令牌。
    - SCRIPT_NAME 如果不为空，必须以“/”开头。
    - PATH_INFO 如果不为空，必须以“/”开头。
    - CONTENT_LENGTH 如果被给出，必须是有数字组成。
    - SCRIPT_NAME 或者 PATH_INFO 两者必须有一个被设置。如果 SCRIPT_NAME 为空，PATH_INFO 必须是“/”。与之相反，SCRIPT_NAME 不允许为“/”，应该使用空字符代替。

##Request##

**Request 概述**

每个 Slim 应用实例都有一个 request 对象。request 对象是对当前 HTTP 请求的抽象，可以让你方便的和 Slim 应用的环境变量进行交互。虽然每个 Slim 应用都包含一个默认的 request 对象，但是 \Slim\Http\Request 类是幂等的；你可以把它作为一个整体在任何需要的地方（中间件或者 Slim 应用的其他地方）实例化这个类而完全不影响应用。你可以用如下方式获得 Slim 应用 request 对象的一个引用：

    <?php
    // 返回 \Slim\Http\Request 实例
    $request = $app->request();

**Request 方式**

每个 HTTP 请求都有一个方式（比如：GET 或 POST）。你可以通过 Slim 应用的 request 对象来获取当前 HTTP 请求方式：

    /**
     * request 请求的方式是什么？
     * @return string (e.g. GET,POST,PUT,DELETE)
     */
    $app->request()->getMethod();

    /**
     * 是否是 GET 请求？
     * @return bool
     */
    $app->request()->isGet();

    /**
     * 是否是 POST 请求？
     * @return bool
     */
    $app->request()->isPost();

    /**
     * 是否是 PUT 请求？
     * @return bool
     */
    $app->request()->isPut();

    /**
     * 是否是 DELETE 请求？
     * @return bool
     */
    $app->request()->isDelete();

    /**
     * 是否是 HEAD 请求？
     * @return bool
     */
    $app->request()->isHead();

    /**
     * 是否是 OPTIONS 请求？
     * @return bool
     */
    $app->request()->isOptions();

    /**
     * 是否是 XHR/AJAX 请求？
     * @return bool
     */
    $app->request()->isAjax();

**Requset Headers**

Slim 应用会自动解析所有的 HTTP 请求头。你可以使用 request 对象的 headers() 函数来访问请求头。

    <?php
    $app = new \Slim\Slim();
    // Get request object
    $req = $app->request();
    // Get request headers as associative array
    $headers = $req->headers();
    // Get the ACCEPT_CHARSET header
    $charset = $req->headers('ACCEPT_CHARSET');

在第二个例子中，如果给出的名字不存在，headers() 函数可能会返回字符串或者 null。

HTTP 规范声明 HTTP 请求首行名称可以是大写、小写或者大小写混合。无论你获取一个大写、小写或者大小写混合名称的请求头，Slim 都能正确的返回请求头的值。因此你可以使用你最喜欢的命名方式。

**Requst Body**

使用 request 对象的 getBody() 函数获取从 HTTP 客户端发送来的原始 HTTP 请求报文主体。这在 Slim  应用处理 JSON 或者 XML 请求时特别有用。

    <?php
    $request = $app->request();
    $body = $request->getBody();

**Request 变量**

HTTP 请求也可以有关联变量（不要和 route 变量弄混）。当前 HTTP 请求发送的 GET、POST 或者 PUT变量都可以通过 Slim 应用的 request 对象获得。

如果你想要在不区分请求方式的情况下快速获取一个 request 变量，可以使用 request 对象的 params() 函数。

    <?php
    $req = $app->request();
    $paramValue = $req->params('paramName');

params() 函数会首先查找 PUT 变量，然后 POST 变量，最后才查找 GET 变量。如果为找到变量，会返回 null。如果你只想查找指定请求方式的变量，你可以使用一下函数来代替：

    <?php
    // Get request object
    $req = $app->request();
    // GET 变量
    $paramValue = $req->get('paramName');
    // POST 变量
    $paramValue = $req->post('paramName');
    // PUT 变量
    $paramValue = $req->put('paramName');

如果查找的变量不存在，每个函数都会返回 null。如果在调用这些函数时没有传递参数，你可以获取一个给定方式的所有变量的数组：

    <?php
    $req = $app->request();
    $allGetVars = $req-get();
    $allPostVars = $req->post();
    $allPutVars = $req->put();

**Request Cookies**

**Get Cookies**

Slim 应用会自动解析当前 HTTP 请求发送得 cookies。你可以使用 Slim 应用的 getCookie() 函数来获取 cookie 的值：

    <?php
    $foo = $app->getCookie('foo');

只有跟随当前 HTTP 请求发送的  Cookies 才能被这个函数访问。如果你在当前请求的过程中设置一个 cookie，它无法被这个函数访问，直到后续请求被发送。如果你想要获取一个跟随当前请求发送的所有 cookies 的数组，你必须使用 request 对象的 cookies()  函数：

    <?php
    $cookies = $app->request()->cookies();

当多个 cookies 同名时（比如它们拥有不同的路径），只有最具体（最长匹配路径）的一个会被返回。详情请查阅 RFC2109。

**Get Encrypted cookies**

如果你预先设置了一个加密的 cookie，你可以使用 Slim 应用的 getEncryptedCookie() 函数来获取它的解密值：

    <?php
    $value = $app->getEncryptedCookie('foo');

如果 cookie 被 HTTP 客户端修改，Slim 会自动注销这个 cookies 值使它对后面的 HTTP 响应无效。你可以把 false 作为第二个传递参数来关闭这个设置：

    <?php
    $value = $app->getEncryptedCookie('foo', false);

不管你是否注销无效的 cookies，当 cookie 无效或者不存在时将会返回 null 值。

**Request Paths**

Slim 应用接收的每个 HTTP 请求都必须包含一个根 URI 以及一个目标资源 URI。

**Root URI**

根 URI 是指实例化和运行 Slim 应用目录的物理 URL 地址。如果一个 Slim 应用是在虚拟机最上层根目录中的 index.php 文件中实例化，那么根 URI 就是一个空值。如果 Slim 应用是在虚拟机根目录的子目录中的 index.php 文件实例化，那么根 URI 就是子目录的路径（以斜线开头但不能以斜线结尾）。

**Resource URI**

resource URI 是应用资源的虚拟路径。resource URI 会被用来与 Slim 应用路由进行匹配。

假设 Slim 应用被安装在你虚拟主机 document root 目录下的 /foo子目录中，再假设完成的 HTTP 请求 URL（你在浏览器地址栏中看到的链接）是 /foo/books/1。那么 root URI 就是 /foo（Slim 应用被实例化的物理目录地址），resource URI 就是 /books/1（应用资源路径）。

你可以使用 request 对象的 getRootUri() 和 getResourceUri() 函数来获取 HTTP 请求的 root URI 和 resource URI。

    <?php
    $app = new Slim();
    // 获取 request 对象
    $req = $app->request();
    // 获取 root URI
    $rootUri = $req->getRootUri();
    // 获取 resource URI
    $resourceUri = $req->getResourceUri();

**XMLHttpRequest**

当使用像 MooTools 或者 JQuery 这样的 javascript 框架执行一个 XMLHttpRequest，XMLHttpReqest 通常会自动附带发送一个 X-Requested-With 的 HTTP 请求首部字段。Slim 应用会检查 HTTP 请求的 X-Requested-With 首部字段并把它标记为 XMLHttpRequest 请求。如果由于某些原因 XMLHttpRequest 不能发送 X-Requested-With 首部字段，你可以通过在 GET、POST 或者 PUT 的请求参数中设置一个名为“isajax”的真值来强制让 Slim 应用认定 HTTP 请求为 XMLHttpRequest。

可以通过 request 对象的 isAjax() 或者 isXhr() 函数来判断当前请求是否是一个 XHR/Ajax 请求：

    <?php
    $isXHR = $app->request->isAjax();
    $isXHR = $app->request->isXhr();

**Request Helpers**

Slim 应用的 request 对象提供了一些获取常用 HTTP 请求信息的辅助函数：

**Content Type**

获取请求的 content type（比如“application/json;charset=utf-8”）：

    <?php
    $req = $app->request;
    $req->getContentType();

**Media Type**

获取请求的 media type（比如“application/json”）：

    <?php
    $req = $app->request;
    $req->getMediaType();

**Media Type Params**

获取请求的 media type （比如charset => "utf-8"）：

    <?php 
    $req = $app->request;
    $req->getMediaTypeParams();

**Content Charset**

获取请求的内容字符集设置（比如“utf-8”）:

    <?php
    $req = $app->request;
    $req->getContentCharset();

**Content Length**

获取请求内容长度：

    <?php
    $req = $app->request;
    $req->getContentLength();

**Host**

获取请求的主机：

    <?php
    $req = $app->request;
    $req->getHost();

**Host with Port**

获取请求的主机及端口（e.g."slimframework.com:80"）：

    <?php
    $req = $app->request;
    $req->getHostWithPort();

**Port**

获取请求的端口（e.g.80）：

    <?php
    $req = $app->request;
    $req->getPort();

**Scheme**

获取请求的协议方式（e.g."http" or "https"）：

    <?php
    $req = $app->request;
    $req->getScheme();

**Path**

获取请求的路径（root URI + resource URI）：

    <?php
    $req = $app->request;
    $req->getPath();

**URL**

获取请求的 URL（scheme + host + port）：

    <?php
    $req = $app->request;
    $req->getUrl();

**IP Address**

获取请求的 IP 地址：

    <?php
    $req = $app->request;
    $req->getIp();

**Referer**

获取请求来源地址：

    <?php
    $req = $app->request;
    $req->getReferrer();

**User Agent**

获取请求的 user agent 字符串：

    <?php
    $req = $app->request;
    $req->getUserAgent();

**Response**

**Response 概述**

每个 Slim 应用实例都有一个 response 对象。这个 response 是 Slim 应用对返回给 HTTP 客户端的响应内容的抽象对象。虽然每个 Slim 应用都包含一个默认的 response 对象，但 \Slim\Http\Response 类是幂等的。你可以在需要的地方（中间件或着 Slim 应用的其他地方）实例化这个类而不会影响应用的整体性。你可以使用一下方法来获得一个 Slim 应用 response 对象的引用：

    <?php
    $app = new \Slim\Slim();
    $app->response;

一个 HTTP response 主要有三个属性：

    Status
    Header
    Body

response 对象提供了辅助函数（会面会讲到）来帮助你与这些 HTTP response 属性互动。默认 response 对象会返回 text/html 类型的 200 OK HTTP 响应。


**Response Status**

返回给客户端的 HTTP 响应会包含一个标识响应类型（e.g.200 OK，400 Bad Request，或者 500 Server Error）的状态码。你可以使用 Slim 应用的 response 对象按下面的方式来设置 HTTP 响应的状态码：

    <?php
    $app->response->setStatus(400);

如果你打算返回一个不是 200 OK 状态的 HTTP 响应，只需要设置 response 对象的状态码即可。同时你也可以简单的通过无参调用该函数来获取 response 对象当前的 HTTP 状态。

    <?php
    $status = $app->response->getStatus();

**Response Headers**

返回给 HTTP 客户端的 HTTP 响应可以包含首部字段。HTTP 首部字段是提供 HTTP 响应相关元信息的一个键值列表。你可以使用 Slim 应用的 response 对象来设置 HTTP 响应的首部字段。response 对象包含一个公有属性 headers，它是 \Slim\Helper\Set 的实例，该属性提供了一个简单的标准接口来操作 HTTP 响应首部字段。

    <?php
    $app = new \Slim\Slim();
    $app->response->headers->set('Content-Type', 'application/json');

你同样可以通过 response 对象的 headers 属性来获取 headers：

    <?php
    $contentType = $app->response->headers->get('Content-Type');

如果给出的 header 名不存在时将会返回 null 值。你可以使用大写，小写或者大小写字母混合结合破折号或下划线的方式来指定 header 名。请使用你最喜欢的命名方式。

**Response Body**

返回给客户端的 HTTP 响应可以可以包含 body（报文主体）。HTTP body 是 HTTP 响应传递给客户端的实际内容。你可以使用 Slim 应用的 response 对象来设置 HTTP 响应的报文主体：

    <?php
    $app = new \Slim\Slim();
    // 重写响应报文主体
    $app->response->setBody('Foo');
    // 追加响应报文主体
    $app->response->write('Bar');

当你重写或者追加 response 对象 body 时，response 对象会自动根据新响应正文长度来设置 Content-Length 首部字段。

你可以这样获取 response 对象的 body 内容：

    <?php
    $body = $app->response->getBody();

通常来说，你不需要手动调用 setBody() 或者 write() 函数来设置响应报文主体；相应的，Slim 应用会自动帮你完成。无论何时你在路由的回调函数中 echo() 了内容，这些 echo() 的内容都会在输出缓冲中被捕获并在 HTTP 响应被返回客户端之前追加到响应报文主体中。

**Response Cookies**

Slim 应用提供了在 HTTP 响应中发送 cookies 的辅助函数。

**Set Cookie**

下面这个例子演示了怎么使用 Slim 应用的 setCookie() 函数来创建一个在 HTTP 响应被发送的 HTTP cookie：

    <?php
    $app->setCookie('foo', 'bar', '2 days');

上面的例子创建了一个名为“foo”，值为“bar”,从现在开始两天后过期的 HTTP cookie。你也给 cookie 添加一些附加属性，包括它的 path（路径），domain（域名），secure 以及 httponly 设置。Slim 应用的 setCookie() 方法采用了和 PHP 原生的 setCookie() 函数相同的参数标识。

    <?php
    $app->setCookie(
        $name,
        $value,
        $expiresAt,
        $path,
        $domain,
        $secure,
        $httponly
    );

**设置加密 Cookie**

你可以通过把 Slim 应用的 cookies.encrypt 设置为 true 来告诉 Slim 应用加密响应 cookies。当该设置为 true，Slim 会在把响应信息返回给 HTTP 客户端之前自动加密 cookies。

下面是 Slim 应用 cookie 加密的相关有效设置：

    <?php
    $app = new \Slim\Slim(array(
        'cookies.encrypt' => true,
        'cookies.secret_key' => 'my_secret_key',
        'cookies.cipher' => MCRYPT_RIJNDAEL_256,
        'cookies.cipher_mode' => MCRYPT_MODE_CBC
    ));

**删除 Cookie**

你可以使用 Slim 应用的 deleteCookie() 方法来删除一个 cookie。它会在下一次 HTTP 请求之前从 HTTP 客户端移除 cookie。这个方法接受与 Slim 应用的 setCookie() 函数除 $expires 之外相同的参数标识。只有第一个参数是必选的。

    <?php
    $app->deleteCookie('foo');

如果你需要指定 path 和 domain：

    <?php
    $app->deleteCookie('foo', '/', 'foo.com');

你同样可以更详细的指定 secure 和 httponly 属性：

    <?php
    $app->deleteCookie('foo', '/', 'foo.com', true, true);

**Response Helpers**

response 对象提供了一些辅助函数用来检查和操作基本的 HTTP 响应。

**Finalize**

response 对象的 finalize() 函数会返回一个索引数组 array(status, header, body)。status  是一个整数；header 是一个可迭代的数据结构；body是一个字符串。你是否在 Slim 应用或它的中间件中创建了一个新的 \Slim\Http\Response 对象，你可以调用 response 对象的 finalize() 方法来生成基础 HTTP 响应的 status，header 和 body。

    <?php
    /**
     * Prepare new response object
     */
    $res = new \Slim\Http\Response();
    $res->setStatus(400);
    $res->write('You made a bad request');
    $res->headers->set('Content-Type', 'text/plain');

    /**
     * Finalize
     * @return [
     *     200,
     *     ['Content-type' => 'text/plain'],
     *     'You made a bad request'
     * ]
     */
    $array = $res->finalize();

**Redirect**

response 对象的 redirect() 方法可以设置 response 状态码和 Location：header 需要返回一个 3xx Redirect 响应。

    <?php
    $app->response->redirect('/foo', 303);

**Status 内省**

response 对象提供了另外一些辅助函数来检查 status。下面这些函数都返回一个布尔值：

    <?php
    $res = $app->response;
    // 是否是一个信息响应？
    $res->isInformational();
    // 是否是 200 OK 响应？
    $res->isOk();
    // 是否是一个 2xx 成功响应？
    $res->isSuccessful();
    // 是否是一个 3xx 跳转响应？
    $res->isRedirection();
    // 是否是某一个特定跳转响应？（301，302，303，307）
    $res->isRedirect();
    // 是否是一个禁止访问响应？
    $res->isForbidden();
    // 是否是一个未找到（not found）响应？
    $res->isNotFound();
    // 是否是一个客户端错误响应？
    $res->isClientError();
    // 是否是一个服务端错误响应？
    $res->isServerError();

## View

**视图概述**

Slim 应用把模板渲染任务委托给它的 view 对象。Slim 应用的视图是 \Slim\View 的子类并实现了下面这个接口：

    <?php
    public render(string $template);

view 对象的 render 函数必须返回模板用给定参数（$template）渲染过后的内容。

**Rendering**

你可以使用 Slim 应用的 render() 函数让当前的 view 对象根据给出的参数设置来渲染一个模板。Slim 应用的 render() 函数会 echo() 从输出缓冲中捕捉 view 对象返回的输出并自动追加到 response 对象的 body 中。它并不关心模板是如何渲染的，而是全部交给 view 对象处理。

    <?php
    $app = new \Slim\Slim();
    $app->get('/books/:id', function($id) use($app){
        $app->render('myTemplate.php', array('id' => $id));
    });

如果你需要在路由的回调函数中向 view 对象传递数据，你必须把需要的数据以用数组形式作为 Slim 应用 render() 函数的第二个参数：

    <?php
    $app->render(
        'myTemplate.php',
        array('name'=> 'Josh')
    );

你也可以在渲染模板的时候同时设置 HTTP 响应的状态码：

    <?php
    $app->render(
        'myTemplate.php',
        array('name' => 'Josh'),
        404
    );

**自定义视图类**

Slim 应用把渲染模板的工作交给它的 view 对象处理。自定义的视图类须是 \Slim\View 的子类并实现了以下接口：

    <?php
    public render(string $template);

view 对象的 render 函数必须返回模板用给定的 $template 参数渲染后的内容。当自定视图类的 render 函数被调用时，会把需要渲染的模板路径（相对于 Slim 应用的‘templates.path’设置）作为它的参数。下面是一个自定义视图类的例子：

    <?php
    class CustomView extends \Slim\View
    {
        public function render($template)
        {
            return 'The final rendered template';
        }
    }

自定义视图类可以在其内部做任何需要的操作只要它返回模板被渲染后的输出的字符串。自定义视图类可以非常方便的整合 Twig 或者 Smarty 这些流行的 PHP 模板系统。

    注意！自定义视图类可以通过访问 $this->data 来获取 Slim 应用的 render() 函数传递给它的参数。

你可以在 GitHub 的 Slim-Extras 库里找到采用了流行 PHP 模板引擎的预设自定义类。

**View 示例**

    <?php
    class CustomView extends \Slim\View
    {
        public function render($template)
        {
            // $template === 'show.php';
            // $this->data['title'] === 'Sahara';
        }
    }

**整合示例**

如果自定义视图类无法被自动加载类加载，那么它必须在 Slim 应用实例化之前被载入。

    <?php
    require 'CustomView.php';
    $app = new \Slim\Slim(array(
        'view' => new CustomView()
    ));
    $app->get('/book/:id', function($id) use($app){
        $app->render('show.php', array('title' => 'Sahara'));
    });
    $app->run();

**View 数据**

    注意！不建议你直接在 view 对象上设置或者追加数据。通常来说，你应该直接通过 Slim 应用的 render() 函数来向视图传递数据。请查看 Rendering Templates 文档。

view 对象的 setData() 和 appendData() 函数可以把数据注入 view 对象；被注入的数据可以被 view 模板使用。视图数据是以键值数组存储的。

**设置数据**

view 对象的 setData() 方法会覆盖以存在的视图数据。你可以使用这个函数为一个变量赋值：

    <?php
    $app->view->setData('color', 'red');

那么视图数据现在就包含一个关键词为“color”值为“red”的数据。你也可以使用 view 对象的 setData() 函数一次性申明整个数组数据：

    <?php
    $app->view->setData(array(
        'color' => 'red',
        'size' => 'medium'
    ));

请记住，view 的 setData() 函数会替换之前的所有数据。

**追加数据**

view 对象的也有一个向已存的视图数据中追加数据的 appendData() 函数。这个函数只接受一个数组作为唯一参数：

    <?php
    $app->view->appendData(array(
        'foo' => 'bar'
    ));

##HTTP Caching

**HTTP 缓存概述**

Slim 应用通过提供 etag()，lastModified() 和 expires() 等辅助函数对 HTTP 缓存进行内置支持。在每个路由中最好使用 etag() 或 lastModified() 中的一个与 expires() 结合使用；请不要在同一个路由回调函数中同时使用 etag() 和 lastModified()。

在路由回调函数中，etag() 和 lastModified() 应该在其他代码之前被调用；这样可以让 Slim 在执行路由回调函数的其他代码之前对 GET 请求的缓存设置进行检查。

etag() 和 lastModified() 都会命令 HTTP 客户端把资源响应存储在客户端缓存中。expires() 函数则通知 HTTP 客户端客户端缓存应该在何时过期。

**ETag**

Slim 应用使用 ETags 来内置对 HTTP caching 的支持。一个 ETag 就是资源 URI 的唯一标识符。当 Slim 应用使用 etag() 函数设置了一个 ETag 首部字段的时候，HTTP 客户端会在以后的同一个资源 URI 的每个 HTTP 请求中发送一个 If-None-Match 首部字段。如果资源 URI 的 ETag 值和 HTTP 请求中的 If-None-Match 首部字段相匹配，Slim 应用会返回一个 304 Not Modified HTTP 响应来命令 HTTP 客户端继续使用它的缓存；这也防止了 Slim 应用对资源 URI 的所有请求进行处理，可以节省带宽和响应时间。

在 Slim 中设置一个 ETag 是非常简单的。在你的路由回调函数中调用 Slim 应用的 etag() 方法，并传递一个唯一 ID 作为它的唯一参数即可。

    <?php
    $app->get('/foo', function() use($app){
        $app->etag('unique-id');
        echo "This will be cached after the initial request!";
    });

就像这样。请确保资源的 ETag ID 是唯一的。也要确保在请求资源改变之后同时改变它的 ETag ID，否则 HTTP 客户端会一直使用过时的缓存。

**Last Modified**

Slim 应用通过使用资源的最后修改时间来内置对 HTTP caching 的支持。当你指定一个最后修改时间，Slim 会告诉 HTTP 客户端该资源最后修改的日期和时间。HTTP 客户端将在给出的资源 URI 的 HTTP 请求中发送一个 If-Modified-Since 首部字段。如果你指定的最后修改日期和 HTTP 请求的 If-Modified-Since 首部字段相匹配，Slim 应用就会返回一个 304 Not Modified HTTP 响应来命令 HTTP 客户端使用它的缓存；这也防止了 Slim 应用对资源 URI 的所有请求进行处理，可以节省带宽和响应时间。

在 Slim 中设置一个最后修改时间非常简单，你只需要在你的路由回调函数中调用 Slim 应用的 lastModified() 方法并传递资源最后修改时间的 UNIX timestamp 作为参数即可。请确保 lastModified() 方法的 timestamp 和资源的最后修改时间一起更新，否则浏览器会继续使用过时的缓存。

    <?php
    $app->get('/foo', function() use($app){
        $app->lastModified(1286139652);
        echo "This will be cached after the inital request!";
    });

**Expires**

expires() 需要与 Slim 应用的 etag() 或者 lastModifed() 方法结合使用，它会在 HTTP 响应中设置一个 Expires 头来通知客户端当前资源的缓存应该何时过期。HTTP 客户端在发送给 Slim 应用的缓存设置的过期日期到达之前一直使用客户端缓存。

expires() 方法接受一个参数：一个 UNIX timestamp 整数或者一个可以用 strtotime() 转换的字符串。

    <?php
    $app->get('/foo', function() use($app){
        $app->etag('unique-resource-id');
        $app->expires('+1 week');
        echo "This will be cached client-side for one week";
    });

##Middleware

**中间件概述**

Slim 框架实现了一个版本的 Rack 协议。因此 Slim 应用可以使用中间件在 Slim 应用被调用之前或之后检查，分析或者修改应用的环境，请求和响应内容。

**中间件结构**

把 Slim 应用当作洋葱的最中心，那么洋葱的每一层都是中间件。当你在调用 Slim 应用的 run() 方法时，最外面的中间件层会先被调用。调用完毕之后，由改中间件层决定是否调用被它包裹的下一层中间件。整个过程通过每层中间件的调用逐步向中心深入，直到最核心的 Slim 应用被调用。整个过程会依次执行是因为每个中间件和 Slim 应用自身都实现了一个公共的 call() 方法。当你为 Slim 应用添加一个中间件，新添加的中间件就会就成为新的外层，包裹之前的最外中间件层（如果存在）或者 Slim 应用自身。

**应用的引用**

中间件的目的是为了检查，分析或者修改 Slim 应用被调用之前和之后的应用环境，请求和响应内容。应用引用让每个中间件非常容易的获取原 Slim 应用的一个引用以及它的环境，请求，响应的引用：

    <?php
    class MyMiddleware extends \Slim\Middleware
    {
        public function call()
        {
            // The Slim application
            $app = $this->app;
            
            // The Environment object
            $env = $app->environment;

            // The Request object
            $req = $app->request;

            // The Response object
            $res = $app->response;
        }
    }

对环境，请求和响应对象作出的修改会立即传递给整个应用以及它的其他中间件层。这是因为所有的中间件层获取的都是同一个 Slim 应用对象的引用。

**里层中间件引用**

每个中间件层都可以通过 $this->next 来引用更里层的中间件。每个中间件都需要选择是否需要调用里层中间件。这样就可以让 Slim 应用完成整个循环。如果一个中间件层选择不调用里层中间件，那么更里层的中间件以及 Slim 应用本身都将不会运行，应用的响应信息会原样返回给 HTTP 客户端。

    <?php

    class MyMiddleware extends \Slim\Middleware
    {
        public function call()
        {
            // Optionally call the next middleware
            $this->next->call();
        }
    }

**如何使用中间件**

可以使用 Slim 应用实例的 add() 方法向 Slim 应用添加一个新的中间件。新的中间件会包裹之前添加的中间件，如果之前未添加中间件则会包裹 Slim 应用本身。

**中间件示例**

这个中间件示例会将 Slim 应用的 HTTP 响应报文主体转换为大写。

    <?php
    class AllCapsMiddleware extends \Slim\Middleware 
    {
        public function call()
        {
            // 获取应用的引用
            $app = $this->app;

            // 调用里层中间件和应用
            $this->next->call();

            // 将报文主体转换为大写
            $res = $app->response;
            $body = $res->getBody();
            $res->setBody(strtoupper($body));
        }
    }

**添加中间件**

    <?php
    $app = new \Slim\Slim();
    $app->add(new \AllCapsMiddleware());
    $app->get('/foo', function() use ($app) {
        echo "Hello";                
    });
    $app->run();

Slim 应用实例的 add() 方法接受一个参数： 一个中间件实例。如果该中间件实例需要特殊配置，可以调用自己的构造函数，那样就可以在其被添加到 Slim 应用之前进行配置。

当上面示例中的 Slim 应用运行后，HTTP 响应报文主体会变成语气更加热情的 “HELLO”。

**如何编写中间件**

Slim 应用的中间件必须是 \Slim\Middleware 的子类并且实现了 call() 方法。call() 方法不接受参数。中间件可以实现自己的构造函数，属性和方法。我推荐你阅读 Slim 内置的中间件代码，并将它们作为编写模板（e.g. Slim/Middleware/ContentTpyes.php 或者 Slim/Middleware/SessionCookie.php）。

这个示例是对 Slim 应用中间件的最简单实现。它继承于 \Slim\Middleware，实现了一个公有方法 call()，并在该方法中调用了里层中间件。

    <?php
    class MyMiddleware extends \Slim\Middleware
    {
        public function call()
        {
            $this->next->call();
        }
    }

##Hooks

**Hooks 概述**

Slim 应用提供了一组钩子（hooks）让你可以注册自己的回调函数。

**什么是 hook**

一个钩子只是 Slim 应用生命周期中的一个片段，在这段时间里由钩子指定的一个具有优先级列表中的回调函数会被调用。钩子使用字符串名称进行标识。

回调函数（callback）是指所有传入 is_callable() 函数后会返回 true 值的内容。回调函数会被赋值给一个钩子并在钩子被调用时执行。如果多个回调函数被赋值给一个钩子，那么回调函数会按照赋值顺序进行调用。

**如何使用 hooks**

回调函数可以通过 Slim 应用的 hook() 方法赋值给一个钩子：

    <?php
    $app = new \Slim\Slim();
    $app->hook('the.hook.name', function(){
        // Do something            
    });

第一个参数是钩子的名字，第二个参数就是回调函数。每个钩子都维护着一个由回调函数组成的具有优先级的列表。默认情况下，每个赋值给钩子的回调函数的优先级会被设置为10。你可以通过向 hook() 方法传递一个整数作为第三个参数来为回调函数设置不同的优先级：

    <?php
    $app = new \Slim\Slim();
    $app->hook('the.hook.name', function(){
        // Do something            
    }, 5);

上面的示例把回调函数的优先级设置为5。当一个钩子被调用时，它会把所有回调函数按照优先级进行升序排列。优先级为1的回调函数会比优先级为10的回调函数更早执行。

钩子不会向回调函数传递参数。如果回调函数需要访问 Slim 应用，你可以通过 use 关键词把应用注入到回调函数中或者直接在回调函数中使用 Slim 应用的 getInstance() 静态方法：

    <?php
    $app = new \Slim\Slim();
    $app->hook('the.hook.name', function() use($app){
        // Don something            
    });

**默认钩子**

这些默认的钩子总是会在 Slim 应用中调用。

    slim.before

这个钩子会在 Slim 应用运行之前以及输出缓冲打开之前被调用。这个钩子在 Slim 生命周期中只被调用一次。

    slim.before.router

这个钩子会在输出缓冲打开之后以及路由器被分派之前被调用。这个钩子在 Slim 生命周期中只被调用一次。

    slim.before.dispatch

这个钩子在当前匹配路由分派之前被调用。通常情况下这个钩子只会调用一次，但是当匹配路由选择传递给下一个匹配路由时会被调用多次。

    slim.after.dispatch

这个钩子会在当前匹配路由分发之后被调用。通常情况下这个钩子只会调用一次，但是当匹配路由选择传递给下一个匹配路由时会被调用多次。

    slim.after.router

这个钩子会在路由器被分发之后，在响应发送给客户端之前以及输出缓冲被关闭之后被调用。这个钩子在 Slim 生命周期内只调用一次。

    slim.after

这个钩子会在输出缓冲被关闭之后并且在响应被发送到客户端之后才调用。这个钩子在 Slim 生命周期内只调用一次。

**自定义钩子**

Slim 应用可以创建和调用自定义钩子。当一个自定义钩子被 applyHook() 方法调用时，它会调用被赋值给它的所有回调函数。这也正是 Slim 应用默认钩子运行的方法。在这个例子中，我应用了一个名为“my.hook.name”的自定义钩子。所有之前注册给这个钩子的回调函数都会被调用。

    <?php
    $app = new \Slim\Slim();
    $app->applyHook('my.hook.name');

当你运行上面的代码，所有赋值给“my.hook.name”钩子的回调函数会被按照优先级升序进行调用。

你应该在钩子被应用之前把回调函数注册到钩子中。你可以这样认为：当你调用 Slim 应用的 applyHook() 方法，你其实是在请求 Slim 去调用所有注册给这个钩子名字的回调函数。

##Flash 消息

**Flash 消息概述**

    注意！ 使用 Flash 消息需要 sessions。如果你没有使用 \Slim\Middleware\SessionCookie 中间件，你就必须自己开启原生的 PHP session。

Slim 像 Rails 和其他大型 web 框架一样支持 flash 消息。Flash 消息允许你定义一条只在下一个 HTTP 请求中有效的消息。这可以让你在执行事件或遇到错误后非常方便的向用户打印消息。

如下所示，Slim 应用的 flash() 和 flashNow() 方法接受两个参数：一个关键词（key）和一条消息。这个关键词可以是任何你想要的字符，并且它定义了在视图模板中如何访问这条消息。例如，如果我在调用了 Slim 应用的 flash('foo', 'The foo message') 方法时传递了这几个参数，就可以在下一个请求的模板中通过 `flash['foo']` 访问这条消息。

Flash 消息通过 sessions 来实现持久性；sessions 是 flash 消息实现的必要部分。Flash 消息都保存在 `$_SESSION['slim.flash']` 中。

**Flash Next**

Slim 应用的 flash() 方法设置了一条在下次请求的视图模板中有效的消息。例子中的消息可以通过模板变量 `flash['error']` 访问。

    <?php
    $app->flash('error', 'User email is required');

**Flash Now**

Slim 应用的 flashNow() 方法设置了一条将在当前请求的视图模板中有效的消息。通过应用实例的 flashNow() 方法设置的消息在下次请求中无效。下面例子中的消息可以用模板变量 `flash['info']` 来访问。

    <?php
    $app->flashNow('info', 'Your credit card is expired');

**Flash Keep**

这个方法告诉 Slim 应用保存上次请求中设置的 flash 消息，它们会在下次请求中继续有效。这个方法在处理 HTTP 跳转时对保持消息的持久性非常有用。 

    <?php
    $app->flashKeep();

##Sessions

**原生会话存储**

Slim 应用不会假定任何与会话（sessions）有关的内容。如果你喜欢使用 PHP session，你必须在实例化 Slim 应用之前设置并通过 session_start() 来启用原生 PHP
session。

同时你也应该关闭 PHP 的会话缓存限制器（session cache limiter），那样 PHP 就不会在 HTTP 响应报文中发送相冲突的缓存过期首部。你可以这样关闭 PHP 的会话缓存限制器：

    <?php
    session_cache_limiter(false);
    session_start();

**Cookie Session Store**

你也可以使用 \Slim\Middleware\SessionCookie 中间件把会话数据储存到经过加密和散列的 HTTP cookies中。想要启用 session cookie 中间件，需要把 \Slim\Middleware\SeesionCookie 中间件添加到你的 Slim 应用中：

    <?php
    $app = new Slim();
    $app->add(new \Slim\Middleware\SessionCookie(array(
        'expires' => '20 minutes',
        'path' => '/',
        'domain' => null,
        'secure' => false,
        'httponly' => false,
        'name' =>'slim_session',
        'secret' => 'CHANGE_ME',
        'cipher' => MCRYPT_RIJNDAEL_256,
        'cipher_mode' => MCRYPT_MODE_CBC
    )));

SessionCookie实例化时的数组参数是可选的，把它写到这里是让你看到默认的中间件设置。这个 session cookie 中间件可以和超全局变量 `$_SESSION` 无缝结合， 因此你可以非常容易的迁移到这个会话存储中间件，而不需要修改应用的任何代码。

如果你使用 session cookie 中间件，你不需要开启原生的 PHP session。超全局变量 `$_SESSION` 依然有效，它会被中间件层储存到一个 HTTP cookie 中，而不是使用 PHP 原生的会话管理。

请记住，HTTP cookies 内容大小被固定限制在4千个字节。如果你的加密 session 数据超过了这个长度，你应该使用 PHP 原生会话或者其他替代会话储存。

    请注意，如果你需要处理敏感信息，不推荐在客户端储存中保存会话数据，即使是使用 Slim 加密的 session cookie 中间件。如果你需要保存敏感信息，你应该把会话信息加密并储存到服务器上。

##Logging

**Logging 概述**

Slim 应用提供了一个 log 对象用于把数据写入到指定输出。但数据写入操作实际是被委托给一个 log writer。

**如何记录数据**

在 Slim 应用中记录数据需要先获取 log 对象：

    <?php 
    $log = $app->log;

log 对象提供了遵循 PSR-3 的接口

    $app->log->debug(mix $object);
    $app->log->info(mix $object);
    $app->log->notice(mix $object);
    $app->log->warning(mix $object);
    $app->log->error(mix $object);
    $app->log->critical(mix $object);
    $app->log->alert(mix $object);
    $app->log->emergency(mix $object);

log 对象的每个方法都接受一个混合型参数。这个参数通常是一个字符串，但是这个参数也可以是其他任何类型。log 对象会将参数传递给它的 log writer。然后由 log writer 负责将输入写入到适当的地方。

**启用 Logging**

Slim 应用的 log 对象提供了下面这两个公有方法在运行时启用或者禁用 logging。

    <?php
    // 启用 logging
    $app->log->setEnabled(true);

    // 禁用 logging
    $app->log->setEnabled(false);

你也可以在应用初始化时采用如下设置来启用或禁用 log 对象：

    <?php 
    $app = new Slim(array(
        'log.enabled' => true                
    ));

如果 logging 被禁用，log 对象会在启用前忽略所有记录信息。

**Log Levels**

    注意！在设置 log 级别的时候请使用 \Slim\Log 常量来代替原始数字。

Slim 应用的 log 对象会根据 log level 设置来记录或忽略记录信息。当你调用 log 对象的方法时，你已经为记录的信息设置了一个级别。有效的 log 级别有：

    \Slim\Log::EMERGENCY
    Level 1
    \Slim\Log::ALERT
    Level 2
    \Slim\Log::CRITICAL
    Level 3
    \Slim\Log::ERROR
    Level 4
    \Slim\Log::WARN
    Level 5
    \Slim\Log::NOTICE
    Level 6
    \Slim\Log::INFO
    Level 7
    \Slim\Log::DEBUG
    \Level 8

只有那些低于当前 log 对象的记录级别的信息会被记录。例如，如果 log 对象的级别为 \Slim\Log::WARN(5)，那么 log 对象会忽略 \Slim\Log::DEBUG 和 \Slim\Log::INFO 消息而会接受 \Slim\Log::WARN，\Slim\Log::ERROR 和 \Slim\Log::CRITICAL 消息。

**如何设置 log level**

    <?php
    $app->log->setLevel(\Slim\Log::WARN);

你也可以在应用初始化时设置 log 对象的级别：

    <?php
    $app new \Slim\Slim(array(
        'log.level' => \Slim\Log::WARN                
    ));

**Log Writer**

Slim 应用的 log 对象包含一个 log writer。log writer 负责将记录的信息写入到适当的输出中（例如 STDERR、日志文件、远程服务器、Twitter 或者 数据库）。Slim 应用 log 对象的默认 log writer 来自于 \Slim\LogFileWriter 类，这个 log writer 把输出指向应用环境设置 slim.error 所引用的资源句柄（默认设置为“php://stderr”）。你也可以使用一个自定义的 log writer。

**如何使用自定义 log writer**

自定义 log writer 必须实现一下公有接口：

    <?php
    public function write(mixed $message);

你必须告诉 Slim 应用的 log 对象使用你自定义的 log writer。你可以在应用初始时使用如下设置：

    <?php
    $app = new \Slim\Slim(array(
        'log.writer' => new MyLogWriter()                
    ));

你也可以通过中间件来设置自定义的 log writer：

    <?php 
    class CustomLogWriterMiddleware extends \Slim\Middleware
    {
        public function call()
        {
            // 设置新的 log write
            $this->app->log->setWriter(new \MyLogWriter());

            // 调用里层中间件
            $this->next->call();
        }
    }

你同样还可以在应用钩子中或者路由回调函数中设置 log writer：

    <?php
    $app->hook('slim.before', function() use($app){
        $app->log->setWriter(new \MyLogWriter());            
    });

如果你只需要把错误信息重定向到不同的资源句柄，可以直接使用 Slim 默认的 log writer，它会把日志信息写入到资源句柄中。你所需要做的只是把 slim.errors 环境变量设置为一个有效的资源句柄即可。

##Error Handling

**错误处理概述**

我们需要面对一个现实：凡事都有可能出现错误。那么在适当的时候拦截并返回错误就非常重要。Slim 应用提供了一些辅助方法来响应错误和异常。

**注意事项**

Slim 应用遵守你的 error_reporting 设置；
Slim 应用只处理 Slim 应用内部产生的错误和异常；
Slim 应用把错误转换为 ErrorException 异常并抛出他们；
Slim 应用在 debug 设置为真时会使用内置的错误处理，其他时候将使用自定义的错误处理。

**错误处理程序**

你可以使用 Slim 的 error() 方法指定一个自定义的错误处函数，它将在错误和异常发生时被调用。自定义错误处理程序只会在应用的调试设置关闭时调用。

自定义错误处理程序应该渲染一个用户友好的消息以减少用户的困惑。和 Slim 应用的 notFound() 方法一样，error() 方法同时充当 getter 和 setter。

**设置自定义错误处理程序**

你可以通过传递一个可调用函数作为 error() 方法的第一个参数来设置自定义错误处理程序。

    <?php
    $app = new \Slim\Slim();
    $app->error(function(\Exception $e) use($app){
        $app->render('error.php');            
    });

在这个例子中，自定义错误处理函数以一个被捕捉的异常作为参数。这让你可以正确的响应不同异常。

**调用错误处理程序**

通常，Slim 应用会在错误或异常发生时自动调用错误处理程序。但是你可以通过 Slim 应用的 error() 方法（不带参数）来手动调用错误处理程序。

**Not Found Handler**

通常我们无法避免用户请求访问一个不存在的页面。Slim 应用可以让你非常容易的通过 notFound() 方法自定义一个 Not Found 处理程序。Not Found 处理程序会在 HTTP 请求不能与路由匹配时调用。这个方法同时充当了 getter 和 setter。

**设置 not found 处理程序**

如果你在调用 Slim 应用的 notFound() 方法时传递了一个可调用对象作为参数，这个方法会把可调用对象注册为 Not Found 处理函数，而之前注册的处理函数将不会被调用。

    <?php
    $app = new \Slim\Slim();
    $app->notFound(function() use($app){
        $app->render('404.html');            
    });

**调用 not found 处理程序**

如果你无参调用 Slim 应用的 notFound() 方法，这个方法会调用之前注册的 Not Found 处理函数。

    <?php
    $app = new \Slim\Slim();
    $app->get('/hello/:name', function($name) use($app){
        if ($name === 'waldo') {
            $app->notFound();
        } else {
            echo "Hello,$name";
        }            
    });

**Debugging**

你可以在应用初始化时通过设置启用调试：

    <?php
    $app = new \Slim\Slim(array(
        'debug' => true                
    ));

你也可以在运行时通过 Slim 应用的 config() 方法来启用调试：

    <?php
    $app = new \Slim\Slim();

    //启用调试（默认设置）
    $app->config('debug', true);

    // 禁用调试
    $app->config('debug', false);

如果启用了调试模式，异常或错误发生时会显示一个带有错误说明的诊断页面，包括出错的文件、错误所在文件行数以及一个错误跟踪栈。如果调试被禁用，自定义错误处理程序会被调用。

**Output Redirection**

Slim 应用的环境变量总会包含一个关键词为 slim.errors 的变量，它是日志和错误信息被写入的资源句柄。当捕捉到异常或者手动调用 log 对象时，Slim 应用的 log 对象会将日志信息写入到 slim.errors中。

如果你想要把错误输出重定向到不同的位置，你可以通过修改 Slim 应用的环境变量设置来定义自己的可写资源句柄。我推荐你使用中间件来更新环境设置：

    <?php
    class CustomErrorMiddleware extends \Slim\Middleware
    {
        public function call()
        {
            // 设置新的错误输出
            $env = $this->app->environment;
            $env['slim.errors'] = fopen('/path/to/output', 'w');

            // 调用里程中间件
            $this->next->call();
        }
    }

请记住，slim.errors 并非必须指向一个文件，它可以指向任意可写的资源。

##Dependency Injection

**依赖注入概述**

Slim 内置了一个资源定位器，为将对象注入到 Slim app 或重写 Slim app 内置对象（例如 Request、Response、Log）提供了一个非常简单的方法。

**注入简单变量**

如果你想把 Slim 作为键值对存储使用，只需要这样：

    <?php
    $app = new \Slim\Slim();
    $app->foo = 'bar';

现在，你可以在任何地方通过 `$app->foo` 获取该变量以及变量值。

**使用资源定位器**

你也可以把 Slim 作为一个资源定位器，把构造所需对象的闭包注入其中。当注入的闭包被请求时，它会被调用并返回闭包的返回值。

    <?php
    $app = new \Slim\Slim();

    // 定义一个创建 UUID 的方法
    $app->uuid = function(){
        return exec('uuidgen');
    };

    // 获取一个新的 UUID
    $uuid = $app->uuid;

**单例资源**

有时你想要定义的资源在每次请求时保持一致（例如在 Slim app 的作用域内他们应该是单例的）。这非常容易做到：

    <?php
    $app = new \Slim\Slim();

    // 定义一个 log 资源
    $app->container->singleton('log', function(){
        return new \My\Custom\Log();            
    });

    // 获取 log 资源
    $log = $app->log;

每次你通过 `$app->log` 请求 log 资源，它都会返回同一个实例。

**闭包资源**

如果你想把闭包原样储存，并且不需要调用它，你可以这样做：

    <?php
    $app = new \Slim\Slim();

    // 定义一个闭包
    $app->myClosure = $app->container->portect(function(){});

    // 返回没有调用的原始闭包
    $myClosure = $app->myClosure;

-- EOF --

