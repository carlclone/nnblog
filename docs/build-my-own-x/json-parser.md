# simple JSON Parser

有了之前 lisp-compiler 的经验, json parser 变得简单许多, 只需要取出语法分析和 parser 的部分,对照 json 官方的 token 解析和结构关系表, 就可以写出一个基本的 parser

```
json string -> tokens -> ast
ast -> json string
```

基础软件, 按 TDD 方式节省时间, 先把 test case 写出来

先是最简单的字面量 null,true,false




## 参考资料

