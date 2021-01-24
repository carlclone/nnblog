

# Composer 是如何工作的

本文翻译自[How Composer Autoloads PHP Files](https://jinoantony.com/blog/how-composer-autoloads-php-files) , 作者写的文章阅读体验很舒适


Composer 是 PHP 的应用程序级依赖管理器。除了管理依赖项，composer还将自动加载应用程序所需的文件。大多数框架(Laravel，Symfony 等)都使用 composer，默认情况下是自动加载的。


## 什么是自动加载

自动加载意味着自动加载项目/应用程序所需的文件。即包含应用程序所需的文件，而不显式地将其包含在 include ()或 require ()函数中。

## 为什么需要自动加载

如果您的项目包含的文件较少 , 使用 include ()或 require () 引入文件也是没问题的，但是现实世界中的大多数应用程序都包含一个巨大的文件库。因此，使用上述函数引入文件将成为一项乏味的工作。如果我们的项目依赖于大量的外部库/包，那么这将变得更加困难。

所以我们需要一个简单的方法来加载文件。自动加载可以帮助我们。

## 如何运作

让我们来看一个简单的自动加载程序。

```php
class Autoloader
{
    public function autoLoad($className) 
    {
        //logic for finding class path
        include $fullyResolvedPath;
    }
}

spl_autoload_register(['Autoloader', 'autoLoad']);

$obj = new DemoClass;
```

这里我们使用内置的 php 函数 `spl _ autoload _ register ()`来注册我们的 autoload 函数

“嘿，如果你自己找不到这个 Class，让我帮你找吧。如果我也找不到，就抛出一个错误。”

在我们提供的 autoloader 中，我们可以找到类的路径，然后使用 include ()函数包含它。


## 自动加载的类型

编写器支持以下类型的自动加载。

- Classmap
- PSR-0
- PSR-4
- Files

自动加载类型及其规则可以在 composer.json 文件中定义。可以在单个应用程序中配置多种类型的自动加载。其中 PSR-4和 PSR-0是由 PHP-FIG 小组提出的自动加载标准。

### Classmap

顾名思义，Classmap 创建了指定目录中所有类到单个 key = > value 数组的映射，该数组可以在生成的文件 `vendor/composer/autoload _ Classmap.php `中找到。在composer安装/更新时自动生成该文件。

```
{
    "autoload": {
        "classmap": ["src/", "lib/", "Something.php"]
    }
}
```

映射是通过扫描指定目录内部的所有`.php` 和`.inc`类来完成。这是自动加载的最快方法，因为它使用数组查找(哈希表)来查找类。

### PSR-0
这是 PSR-4之前用于自动加载文件的 PSR 标准，现在已不推荐使用。您可以在配置文件中定义 PSR-0规则，作为从名称空间到路径(相对于包/应用程序根)的映射。

```
{
    "autoload": {
        "psr-0": {
            "Acme\\Foo\\": "src/",
            "Vendor\\Namespace\\": "src/",
            "Vendor_Namespace_": "src/"
        }
    }
}
```

所有 PSR-0引用都组合成一个单独的 key = > value 数组，该数组可以在生成的  `filevendor/composer/autoload _ namespaces.php` 中找到。在自动加载 PSR-0期间，在生成的数组中查找名称空间并从其值中解析路径。

例如，`Acme\Foo\Bar` 将解析为 `src/Acme/Foo/Bar.php`。PSR-0还支持类名中的下划线(_)。类名中的每个 _ 字符都转换为 `DIRECTORY _ separator`。因此，`Acme _ foo _ bar` 也将被解析为 `src/Acme/foo/bar.php`。

### PSR-4

PSR-4是目前推荐的自动加载方式，因为它提供了更大的易用性。


```
{
    "autoload": {
        "psr-4": {
            "Acme\\Foo\\": "src/",
            "Vendor\\Namespace\\": ""
        }
    }
}
```

与 PSR-0下划线不同，它不会转换为 `DIRECTORY _ separator`，并且命名空间前缀不存在于路径中。

例如，`Acme\Foo\Bar` 将解析为 `src/Bar.php`。


在安装/更新期间，所有 PSR-4引用都被组合成一个单独的 key = > value 数组，该数组可以在 `vendor/composer/autoload _ psr4.php` 中找到。

### Files

Classmap、 PSR-0和 PSR-4仅处理类。如果你想自动加载函数，你可以使用文件自动加载。这对于加载辅助函数很有用。每次请求时都会加载这些文件。

```
{
    "autoload": {
        "files": ["src/helpers.php"]
    }
}

```

## 关于 Classmap 和 PSR-4的说明

Classmap 自动加载是自动加载器中最快的，因为它从预构建的数组中加载类。但是 classmap 的问题是，每次创建新的类文件时都需要重新生成 classmap 数组。因此，在开发环境中 PSR-4自动加载将是最适合的一种。

PSR-4自动加载将比 classmap 慢，因为它需要在决定性地解析一个 classname 之前检查文件系统。但是在生产过程中，你需要尽可能快的速度。

基于这个原因，composer 提供了从 PSR-4生成 classmap 的方法。Classmap 生成将所有 PSR-4/PSR-0规则转换为 Classmap 规则。所以自动加载会更快。如果在生成的类映射中没有找到 PSR-4类，则自动加载回退到 PSR-4规则。

可以通过以下任何方式启用 Classmap 生成。

- set "optimize-autoloader": true inside the config key of composer.json
- call install or 或update with 与-o / --optimize-autoloader
- call dump-autoload with 与-o / --optimize



## 组合起来

 现在我们可以看看composer 是如何自动加载我们的文件的 。首先，我们需要包含 composer 的 `autoload.php` 文件。

```
require __DIR__.'/../vendor/autoload.php';
```

它需要另一个文件`composer/autoload _ real.php`，并对生成的编写器 autoloader 类调用 getLoader ()方法。

```
require_once __DIR__ . '/composer/autoload_real.php';

return ComposerAutoloaderInitcac18dc222d8ea1e03bdef8b44290883::getLoader();
```

在 getLoader ()方法中，composer 加载在 `composer install/update` 或 `composer dump-autoload` 上生成的所有 autoloader 数组，并使用前面提到的 `spl _ autoload _ register ()`注册 autoload 函数。

```
// autoload_real.php
public static function getLoader()
{
    // ...
    self::$loader = $loader = new \Composer\Autoload\ClassLoader();
    // Loads all autoloader arrays
    $loader->register(true);
    // ...
    return $loader;
}
```

ClassLoader 类中的 register ()方法注册自动加载函数 loadClass ()。

```
class ClassLoader
{
    public function register($prepend = false)
    {
        spl_autoload_register(array($this, 'loadClass'), true, $prepend);
    }
}
```

loadClass ()方法很简单。它使用类名调用另一个方法 findFile () ，如果该方法返回一个有效的文件路径，那么它使用文件路径为另一个 helper 函数 includeFile()类化，该函数只是在给定的文件路径中包含文件。

```
public function loadClass($class)
{
    if ($file = $this->findFile($class)) {
        includeFile($file);

        return true;
    }
}
```



findFile ()方法负责查找文件路径。由于它的实现有点复杂，我们不在这里讨论它。但是您需要知道的是，它以下面的方式检查文件路径查找。

- 检查 classmap 是否包含指定的类，如果发现立即返回文件路径
- 如果在 classmap 中没有找到该文件，则执行 psr-4查找，如果找到带有生成的文件路径则返回
- 如果 psr-4 查找也不成功，则执行 psr-0查找，如果找到, 则返回生成的文件路径
  

以上。composer可以很好地自动加载文件，并为您提供各种配置选项，使开发变得轻而易举。
