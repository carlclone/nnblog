# 构建简单的 lisp compiler

参照着[the-super-tiny-compiler](https://github.com/jamiebuilds/the-super-tiny-compiler/blob/master/the-super-tiny-compiler.js) , 写了一个 [go 语言的版本](https://github.com/carlclone/lisp-compiler/blob/master/compiler.go)

```
//嵌套function
(add 10 (subtract 10 6))

//输出
add(10, subtract(10, 6));

//program 下多条 function call
(add 2 2)
(subtract 4 2)

add(2,2);
subtract(4,2);
```

整个过程由词法分析产生 token, 遍历 token 生成lisp AST,遍历 lisp AST 生成新语言的 AST, 最后遍历新 AST 输出可执行语句,这里的可执行语句不是汇编,而是转换成了 JavaScript 语句, 相对要简单许多, 但对编译原理有了一个基本的入门

写完整个 compiler 会发现用到了许多树的遍历模板, 是一个练习树/递归结构的不错的小作业, 并且不会像算法题一样枯燥

<!--同时写了 test case, 使得优化代码的过程(go的字符串遍历,生成部分) 保证程序正确-->

并且这里的子树只有在 function call 的时候才会出现, 通常做这类题, 子树的细节是屏蔽掉的, 如下图, 这是我做树形题目的主要思路

![](/img/tree.png)
 
```
//树遍历主要框架
traverse(root Node) {
    //进入节点
    enter()
    for child:=range root.children {
        traverse(child)
    }
    //退出节点
    exit()
}
```
