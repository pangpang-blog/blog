---
title: "通过1000个代码重构，学习到的知识点"
date: 2018-03-09T18:25:31+08:00
draft: true
---

# 通过1000个代码重构，学习到的知识点  (What I learned from doing 1000 code reviews)(翻译)

在LinkedIn工作的时候，我大部分的工作都是做代码检查。有一些建议不断地反复出现，所以我决定把我和团队分享的清单放在一起。 

**建议 1: 出错时请抛（throw）出一个异常（exception）**

我看到的一个常见的模式是：

```
List<String> getSearchResults(...) {
  try {
    List<String> results = // make REST call to search service
    return results;
  } catch (RemoteInvocationException e) {
    return Collections.emptyList();
  }
}
```

这种模式实际上造成了我开发过的移动应用程序的中断。

我们使用的搜索后端开始抛出异常。然而，应用程序的API服务器有一些类似于上面的代码。因此，从应用程序的角度来看，它获得了200的成功响应，并愉快地显示了空列表。

相反，如果API抛出了一个异常，那么我们的监控系统就会立即将其提取出来，并修复。

很多时候，当你遇到异常时，为了便利返回空对象。在java中空对象的例子是Optional.empty()，null,empty list。在解析URL时总是出现这样的问题。如果无法从字符串中解析URL，返回值为null。可以自问：“为什么URL是错误的？”我们应该在上游解决的数据问题吗？“

空对象不是处理这项工作的合适工具。**如果发生异常，则应抛出异常**。



**建议2：尽可能使用具体的类型(type)** 

这个建议基本上跟[stringly typed programming](http://wiki.c2.com/?StringlyTyped)（强类型编程）是相反的。

我经常看到类似这些例子的代码。

```
void doOperation(String opType, Data data); 
// where opType is "insert", "append", or "delete", this should have clearly been an enum

String fetchWebsite(String url);
// where url is "https://google.com", this should have been an URN

String parseId(Input input);
// the return type is String but ids are actually Longs like "6345789"
```



用具体的类型可以让你避免一类错误，也是推荐java，这种强类型语言的理由。

现在的问题是：如何成为终结“写坏的强类型代码（stringly typed code）”的程序员？。答案是：因为外部不使用强类型。有很多的方式接受字符串，比如：

- query and path parameters in urls
- JSON
- databases that don’t support enums
- poorly written libraries（写的不好的libraries）

在这些情况下，你应该使用以下策略来避免这个问题：

将字符串解析和序列化保持在程序的上下文中。

下面举个例子：

```
// Step 1: Take a query param representing a company name / member id pair and parse it
// example: context=Pair(linkedin,456)
Pair<String, Long> companyMember = parseQueryParam("context");
// this should throw an exception if malformed

// Step 2: Do all the stuff in your application
// MOST if not all of your code should live in this area

// Step 3: Convert the parameter back into a String at the very end if necessary
String redirectLink = serializeQueryParam("context");
```



这赋予了许多优点。可以发现错误的数据。如果出现问题，应用程序会提前报错。在数据经过一次验证之后，你也不必捕获解析异常。此外，使用强类型会展现更多方法的叙述，你不需要为每一个方法写javadocs。



**建议3：请用Optionals代替null**

java8中提供[Optional](http://download.java.net/jdk8/docs/api/java/util/Optional.html)类。Optional类的Javadoc描述如下：

这是一个可以为null的容器对象。如果值存在则isPresent()方法会返回true，调用get()方法会返回该对象。

问题：什么是唯一有自己的缩写的异常？答：NPE(Null Pointer Exception)空指针异常 。

这是在java中经常出现的异常，[被称为十亿美元的错误](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare)。

使用`Optional` 可以避免NPE，然而，它必须正确使用。

这里有一些建议关于`Optional`的使用。

- 你不能简单的调用`get() `方法，需要考虑`Optional`没有值的情况，并提出一个合理的默认值。
- 如果没有一个合理的默认值，那么可以使用`.map()` 和 `.flatMap()`方法延迟定义。
- 如果外部库返回null，用`Optional.ofNullable()`包裹相应的值，相信我，你以后会感谢你自己的。null会在程序中`冒泡(bubble up)`，所以停止在程序中使用它。
- 在方法的返回类型中使用`Optional`. 因为你不需要读Javadoc，来找不存在值的情况。



你应该避免使用类似这种方法：

```
// AVOID:
CompletableFuture<T> method(CompletableFuture<S> param);
// PREFER: 
T method(S param);

// AVOID:
List<T> method(List<S> param);
// PREFER:
T method(S param);

// AVOID: 
T method(A param1, B param2, Optional<C> param3);
// PREFER:
T method(A param1, B param2, C param3);
T method(A param1, B param2);
// This method is clearly doing two things, it should be two methods
// The same is true for boolean parameters
```



以上的方法中有共同之处是什么？它们使用容器对象，如Optional，List，Task作为方法的参数。当返回类型是相同的容器时，情况更糟。（一个带有`Optional`参数的方法并返回一个`Optional`。

Why?

1) `Promise<A> method(Promise<B> param)`
is less flexible than simply having

2) `A method(B param)`.

如果你有一个`Promise<B>`，那么你可以使用1)，或者你可以使用2)，通过“提升”函数和map (ie. promise.map(method))。

但是，如果您只有B，那么您可以轻松地使用2)，但是您不能使用1），这使得2）更灵活。

我喜欢称之为“unlifting”，因为它是相对常见的功能实用方法“lift”。应用这些改写使方法更灵活、更容易使用的用户。

**参考**

看看我的另一篇关于"Practical Functional Programming"的文章：

[https://hackernoon.com/practical-functional-programming-6d7932abc58b](https://hackernoon.com/practical-functional-programming-6d7932abc58b)