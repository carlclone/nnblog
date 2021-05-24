```php
3. 下面的代码能正常编译吗？


package main

import (
	"fmt"
)

type People interface {
	Speak(string) string
}

type Stduent struct{}

func (stu *Stduent) Speak(think string) (talk string) {
	if think == "bitch" {
		talk = "You are a good boy"
	} else {
		talk = "hi"
	}
	return
}

func main() {
	var peo People = Stduent{}
	think := "bitch"
	fmt.Println(peo.Speak(think))
}
　　运行打印结果：

# command-line-arguments
.\main.go:23:6: cannot use Stduent literal (type Stduent) as type People in assignment:
        Stduent does not implement People (Speak method has pointer receiver)
　　从上面的输出信息可以看出 Student 没有实现 People 这个接口

       解析：

       我们来看一下语言规范里面定义的规则，这些规则用来说明一个类型的值或指针是否实现了该接口：

       1.类型 *T 的可调用方法集包含接收者为 *T 或 T 的所有方法集

          这条规则说的是如果我们用来调用接口方法的变量是一个指针类型，那么方法的接收者可以是值类型也可以是指针类型，

现在看一下我们的例子显然是不符合规则的，var peo People = Student{} 是一个值类型

       2. 类型 T 的可调用方法集包含接收者为 T 的所有方法集

         这条规则说的是如果我们用来调用接口方法的变量是一个值类型，那么方法的接收者必须要是值类型才可以被调用，看一下

我们的例子，方法的接收者是指针类型

        上面的代码可以这样修改：

        1.var peo People = &Student{}

        2.将方法的接收者改成值类型

        可以参考这篇文章：https://github.com/Unknwon/gcblog/blob/master/content/26-methods-interfaces-and-embedded-types-in-golang.md
        
        https://blog.csdn.net/weixin_30325793/article/details/97350152
