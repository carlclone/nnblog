why we need scanner (abstraction , separation of concern, some impl combine all , and become hard to read,maintain)
what is scanner

how many methods to implement (easy -> hard)
by case scan , and sub-case (most easy to maintain , clean code , which hcl use)
regular expression (build your own , and CS164 )
DFA (by hand , or  lexical grammar (LEX auto generate a DFA machine))
the way I do in go , and some small technique (like go generate)
what problem i solved and by what tech in the process
what i learned? (go error handle https://lailin.xyz/post/go-training-03.html , go generate)
can this use in production , daily work ? (custom script/conf lang , document / web page parse)

reference source
clox scanner optimization https://craftinginterpreters.com/scanning-on-demand.html#identifiers-and-keywords
go standard ast interface and go generate https://lailin.xyz/post/41140.html
jlox source code
crafting-inter... related page
cs164 https://inst.eecs.berkeley.edu/~cs164/fa20/
hashicorp configuration language
go-workwx ast
json

还得把 byte code machine 看了
scanner.go
tokens.go