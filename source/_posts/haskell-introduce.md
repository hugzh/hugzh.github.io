title: Haskell初接触感悟
categories:
  - 其它
date: 2014-09-03 00:00:00
tags:
---

最近选修函数程序设计这门选修课，初步接触到函数式编程语言 Haskell 的思想，在学习的过程中有意识地拿它跟Python去做一些比较，发现两者之间在元组Tuple以及List这种灵活数据类型上面有比较大的接近。但是在设计思想以及语法上面，两者又存在较大差别。先说说Haskell语言是什么东西：
<!--more-->
Haskell 是一门纯函数式编程语言 (purely functional programming language)。在命令式语言中执行操作需要给电脑安排一组命令，随着命令的执行，状态就会随之发生改变。例如你指派变量 a 的值为 5，而随后做了其它一些事情之后 a 就可能变成的其它值。有控制流程 (control flow)，你就可以重复执行操作。然而在纯函数式编程语言中，你不是像命令式语言那样命令电脑“要做什么”，而是通过用函数来描述出问题“是什么”，如“阶乘是指从1到某个数的乘积”，"一个串列中数字的和"是指把第一个数字跟剩余数字的和相加。你用宣告函数是什么的形式来写程序。另外，变量 (variable) 一旦被指定，就不可以更改了，你已经说了 a 就是 5，就不能再另说 a 是别的什么数。（译注：其实用 variable 来表达造成字义的 overloading，会让人联想到 imperative languages 中 variable 是代表状态，但在 functional languages 中 variable 是相近于数学中使用的 variable。x=5 代表 x 就是 5，不是说 x 在 5 这个状态。) 所以说，在纯函数式编程语言中的函数能做的唯一事情就是利用引数计算结果，不会产生所谓的"副作用 (side effect)" (译注：也就是改变非函数内部的状态，像是 imperative languages 里面动到 global variable 就是 side effect)。一开始会觉得这限制很大，不过这也是他的优点所在：若以同样的参数调用同一个函数两次，得到的结果一定是相同。这被称作“引用透明 (Referential Transparency)” (译注：这就跟数学上函数的使用一样)。如此一来编译器就可以理解程序的行为，你也很容易就能验证一个函数的正确性，继而可以将一些简单的函数组合成更复杂的函数。(来源:《Learn You a Haskell For Great Good》)也就是说，面向对象或者基于过程的程序设计语言和函数式编程语言是存在本质区别的。
Haskell语言有什么特点呢，要记的特点（或者说优势）还是比较多的，这里介绍一两个我刚刚理解的方面。第一，灵活的数据结构——List和Tuple，关于这两种数据结构是有比较多的内建函数需要去掌握的，比如：head 返回一个 List 的头部，也就是 List 的首个元素

```haskell
ghci> head [5,4,3,2,1]

5
```
`；又比如：take 返回一个 List 的前几个元素

```haskell
ghci> take 3 [5,4,3,2,1]
[5,4,3]
ghci> take 1 [3,9,3]
[3]
ghci> take 5 [1,2]
[1,2]
ghci> take 0 [6,6,6]
[]

```

在这里我想要分享的是关于模式匹配这一章，模式匹配是Haskell最大的特点之一，也是学习Haskell的最大难点之一（个人感觉，目前还没有完全搞懂）。
函数的模式匹配在递归函数的运用还是比较多的，除了一种显示的匹配（类似于C语言里面的CASE）如下：

```haskell
sayMe :: (Integral a) => a -> String
sayMe 1 = "One!"
sayMe 2 = "Two!"
sayMe 3 = "Three!"
sayMe 4 = "Four!"
sayMe 5 = "Five!"
sayMe x = "Not between 1 and 5"
```

除此之外，比较有用的模式匹配就是x:xs这种模式，这在函数递归里面非常重要。而函数递归有事Haskell的核心内容，因此，模式匹配也就是Haskell的核心内容。以下是一个递归求和的例子

```haskell
sum' [] = 0
sum' (x:xs) = x + sum' xs
```

在这段代码里面，现匹配列表为空的情况，余下的情况，提取出列表头部x，x加上余下列表的递归sum'函数，这样就会一直把列表里面的元素相加，知道列表为空，也就是边界条件。这就是模式匹配在函数递归里面的简单应用。至于更强大的函数功能，大家可以参考网上教程 http://learnyouahaskell-zh-tw.csie.org/zh-cn/chapters.html 去更深入的学习Haskell的精髓。