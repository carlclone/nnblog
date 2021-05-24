---
title: Laravel中的Marcoable
tags: 归档
date: 2018-04-08 14:57:31
---

4月8号一篇源码阅读笔记

<!-- more -->

# Laravel 中的 Marcoable

## 源码阅读
官方文档中的例子 , 动态的添加方法到类中

```
use Illuminate\Support\Str;

Collection::macro('toUpper', function () {
    return $this->map(function ($value) {
        return Str::upper($value);
    });
});

$collection = collect(['first', 'second']);

$upper = $collection->toUpper();

// ['FIRST', 'SECOND']
```

### Collection.php
`use Macroable;`

### Marcoable.php ( Trait )

```
protected static $macros = [];  //静态变量marcos

//Collection::marco的时候将动态加入的方法名和Closure以键值对存入marcos
public static function macro($name, $macro)
    {
        static::$macros[$name] = $macro;
    }

```

生成Collection实例,调用自定义方法时

```
$collection = collect(['first', 'second']);

$upper = $collection->toUpper();
```

执行Marcoable Trait中的

```
public function __call($method, $parameters)
    {
        if (! static::hasMacro($method)) {
            throw new BadMethodCallException("Method {$method} does not exist.");
        }
        //取出自定义方法的Closure
        $macro = static::$macros[$method];

        if ($macro instanceof Closure) {
            //将Closure绑定为创建的Collection实例(这里是$collection)的动态方法,然后call_user_func_array传入参数并调用
            return call_user_func_array($macro->bindTo($this, static::class), $parameters);
        }

        //如果不是Closure (那就是有名字的普通function) , 不必绑定到Collection实例,直接调用 (好像就跟Collection没关系了,还不如单独调用function?)
        return call_user_func_array($macro, $parameters);
    }
```

## Summary

可以应用到自己的类中使用

