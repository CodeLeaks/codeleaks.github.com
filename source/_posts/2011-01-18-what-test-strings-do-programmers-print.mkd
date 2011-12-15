--- 
categories: []
comments: true
layout: post
published: true
status: publish
tags: []
title: 当程序员打印测试字符串时，他们打印些什么？
type: post
---
这个问题的起源是一个朋友前两天发版本的时候忘了删代码里的 "fuck" 字符串，结果从服务器拉回来的 log 里一堆的脏话。而我还有个朋友在写代码的时候喜欢打印 "sucker" 或者 "shit" 之类的字符串。恰好之前我又看过 CoolShell 的<a href="http://coolshell.cn/articles/1850.html">《JavaScript 程序员嘴最脏？》</a>一文，便猜想在程序中使用脏话作注释或者测试字符串或许并不是一个特例。

一方面为了验证这个猜想，另一方面也是好奇大家都用什么做测试字符串，昨天我在 Twitter 上提了一个问题：<a href="http://twitter.com/#!/luosheng/status/26920416706568192">「大家在写代码的时候如果要打印一个测试字符串一般会用什么？」</a>

到目前为止，一共收到 37 位推友的有效答复。个人觉得这些答案还挺有代表性的，于是就写一篇博客来总结下。顺便按答案把各位程序员归个类——不当之处还望各位海涵。 :D

<h3>单字符重复型</h3>

单字符重复型指的是那些输出 "aaaaaa"、"bbbbbb"、"11111" 等的程序员。含有单字符重复型的答案总共有 9 个，是程序员们选择最多的一种类型。当然这也很好理解，毕竟单字符打起来方便，而且混在其他的输出结果中也显得那么地拉风那么地有气势。

<h3>你好世界型</h3>

嗯，你好世界型的程序员选择的测试字符串当然是 "hello world" 了。含有 "hello world" 或者是 "hello" 的答案总共有 8 个。说实话之前没有想到程序员对 "hello world" 那么有感情——我还以为大家是只在写第一个程序的时候才用这个的。

<h3>老实巴交型</h3>

既然是打印测试字符串，那么输出 "test" 什么的显然就最老实不过了。含有 "test" 的答案一共有 5 个。 

<h3>指法练习型</h3>

有两名程序员的答案是 "the quick brown fox jumps over the lazy dog"——其实这个更应该归类到「键盘测试型」中。另外还有几位程序员的答案则是基本键位上的指法练习，打印的是 "asdf" 或者 "asdfghjk"。

<h3>乐观向上型</h3>

乐观向上型的程序员喜欢在代码中打印各种代表笑声的字符串如 "haha"、"hehe"、"hoho"、"heihei" 等。

<h3>拉或不拉型</h3>

@5p3ct3r 的回答是 "lalala" 而 @localhost_8080 的回答则是 "blahblahblah"——我笑点<del>滴</del>低了。

<h3>失意体前屈型</h3>

@Karloku 的回答是 "orz" "orzorz" "orzotl"。

<h3>Forever Alone 型</h3>

这里重点要提一下的是 @overboming 的回答：<a href="http://twitter.com/#!/overboming/status/26921891675504640">「我打的是 holy shit, this should never be happening, how can that be, whatever, something has happened, ok I can see %@ 之类的..」</a>。昨天看到这个回答的时候简直是全身颤抖不能自已。一个程序员能和自己的代码进行如此深层次地沟通和交淡，这是多么不容易的事情。而在我向 @overboming 投去敬慕眼光的时候，他只是轻轻地扔回我一个 Forever Alone 的 YouTube 链接（<a href="http://www.youtube.com/watch?v=Ny5qIH7v1SQ">http://www.youtube.com/watch?v=Ny5qIH7v1SQ</a>），然后转过身继续写代码去了……

扯淡文到此结果。不过综上可见，大家都是文明的程序员。:D 路过的读者不妨也在留言区说说看你平时都用什么作为测试字符？
