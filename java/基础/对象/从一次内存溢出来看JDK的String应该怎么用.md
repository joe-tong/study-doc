## 从一次内存溢出来看JDK的String应该怎么用

#### 背景

```
    JDK在String类中给我们提供的API，replace是个使用频率很高的的方法。因为他可以对字符串内容进行替换，只需要输入替换字符串和被替换字符串，就可以轻松得到你想要的字符串，功能非常强大。从JDK里的说明就能看出它有多方便了:
```

源码：

```
    public String replace(CharSequence target, CharSequence replacement) {
        return Pattern.compile(target.toString(), Pattern.LITERAL).matcher(
             this).replaceAll(Matcher.quoteReplacement(replacement.toString()));
    }
```

#### 事故回放

```
   好了，接下来描述的场景纯属虚构，如有雷同，那一定是你也踩过类似的坑！
今天产品MM找你实现一个功能，要求对你们网站上的一些不友好的评论使用“哔~”屏蔽掉，但是判断不友好的功能比较复杂，用到了NLP，所以用到了算法同事给你提供的接口判断，那么接下来事情就简单了：
  我们只需要将NLP处理之后返回来的字符串作为String.replace(target, replacement)的入参target，"哔~"作为replacement就好了。相信熟练的你很快能噼里啪啦敲出以下代码:
```

![微信图片_20190702122408](D:\work_doc\微信图片_20190702122408.jpg)

```
        // 调用NLP接口判断不友好的内容
        List<String> waitForReplace = callRpcNLP(text);
        if (waitForReplace == null || waitForReplace.size() == 0) {
            return;
        }
        // 逐一进行替(和)换(谐)
        for (String target : waitForReplace) {
            if (target == null) {
                continue;
            }
            text.replace(target, "哔~");
        }
```

​        看起来很不错，各种校验也都有了，我的代码果然写得优美又健壮，你已经忍不住陶醉在自己的杰作中了，那么这样有没问题呢?

事实上，到了真正运行的时候，内存爆了！！！

#### 案情分析

**原因之一**

```
    那么到底发生了什么，debug之下，我们定位到replace方法，发现正是因为远程RPC接口返回了空字符串""，而空字符串作为replace的入参target时，会对任意字符串匹配多次匹配成功。我们用个简单点的字符串来做下实验：
```

```
String text = "hello";
System.out.println(text.replace("","*"));

打印：
哔!h哔!e哔!l哔!l哔!o哔!
```

```
    大家能看到，替换后的字符串出现了6个“哔~”。也就是说，一个简简单单的"hello"字符串，在repalce运行之后，被""匹配出了6处需要替换的地方，那么如果不是一个"hello"，而是一大段文本，一篇几千字的论文呢？到了这里，我们距离真相已经很近了，""会导致replace方法对字符串进行多次匹配。这是内存爆了的其中一个原因。
```

**原因之二**

```
     至于另一个原因，就得说下replace的实现了，上面说到的字符串匹配，大家很容易想到正则表达式，实际上，replace内部也确实是通过调用正则表达式相关的API，来实现字符串匹配的。xxxxxxxxxx 原因之二至于另一个原因，就得说下replace的实现了，上面说到的字符串匹配，大家很容易想到正则表达式，实际上，replace内部也确实是通过调用正则表达式相关的API，来实现字符串匹配的。
     而正则表达式实现字符串匹配，实际上是个很复杂的操作，replace(target, replacement)中的target，在正则表达式中称为"模式"（pattern），而匹配的过程，需要每次从字符串拿出一个字符和模式中的字符匹配。如果匹配成功，那么字符串拿出下一个字符，模式也拿出下一个字符，继续下一轮的匹配，但是我们知道，正则表达式支持使用点“.”匹配任意字符，星号"*"匹配任意数量的字符，所以整个匹配的过程需要递归进行，没法通过两个字符串简单地一对一移动指针来完成匹配。
```

#### 总结

通过这次”事故“，我们知道了String.replace方法是有可能导致内存溢出的。总结下我们如何防止出现这类问题：

- 入参target不但需要做null校验，还要做""空字符串校验
- 防止直接输入大段文本进行匹配，可通过对文本分片实现