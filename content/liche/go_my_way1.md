---
title: "Go My Way #1 - web 框架"
date: 2018-03-09T18:23:09+08:00
draft: true
---

# [Go My Way #1 - web 框架](https://jaehue.github.io/post/go-my-way-1-webframework/)

Jun 12, 2017

Ruby的Ruby on Rails，Java的Spring，Python的Django,Nodejs的Express。多数有人气的语言都有主流的web框架。所以不需要多想使用主流框架就可以以，但是Go没有这些。Go提倡根据情况合理的组合package。熟练了以后非常方便，但对于第一次接触Go的人选择package是困难的事情。

接触Go已有3年，过去1年中积极使用了Go语言。起初公司内只有自己使用Go，现在使用Go开发的同事多了起来。我们公司在中国提供it服务，目前阿里云里运营着5个linux服务器和2个windows服务器。共有50左右的服务，其中用Go开发的服务占10个左右。

就这样使用Go语言开发，有了自己的开发方式。我想通过3偏文章介绍Go语言开发。（太冗长，读者没有耐心读完，自己觉得也麻烦，呵呵）

- Go My Way #1 - web框架
- [Go My Way #2 - 数据库, 日志](https://jaehue.github.io/post/go-my-way-2-database-and-logging/)
- Go My Way #3 - Configuration, Tracing, etc.
-  gomobile

第一个主题是web框架。我觉得对这部分关注的人比较多，所以选择了它。

但是开发web应用程序，只有web框架是不够的。保存数据需要数据库，想保存日志，想追溯处理web请求的记录。想一一解决各种问题，需要更多的选择。所以在第2章和第3章对 DB, Logging, Tracing, Vendoring, Confugration, Test等功能一一介绍。

最后会对， [gomobile](https://github.com/golang/go/wiki/Mobile) 进行介绍。

在几个大会当中有对它的介绍，但实际用Go开发移动端有诸多不便。但我们公司的服务的app中有Go package。gomobile有制约事项，但考虑好制约事项开发项目也不错。最后会介绍在移动端怎么使用Go。

此后会用3个篇幅介绍开发web应用程序的方法。没有绝对正确的方式，Go也不提倡这种方式。我将介绍我们公司业务环境下对web应用程序的优化方式，环境不一样可能方式也不一样。在我们公司也有用其他方式开发项目。

------

**对所介绍的内容的讨论，建议，争论，质疑，等**

**我将感谢任何形式的意见。**

**我的思想是永远开放的. ^^**

------

[ECHO](https://echo.labstack.com)

先讲结论，开发web应用程序时用echo来开发。

根据之前所说的内容开发了参考项目echosample([https://github.com/pangpanglabs/echosample](https://github.com/pangpanglabs/echosample))。有新的内容添加时会持续更新 [echosample](https://github.com/pangpanglabs/echosample)。开发新的服务时可以参考echosample。

非常欢迎想一起开发[echosample](https://github.com/pangpanglabs/echosample)的同事。

# main 框架 - echo

[echo](https://echo.labstack.com/)是没有明显缺点，只有必要的功能的轻量级框架。

Go有许多web框架，但功能大同小异。大部分会提供一下的功能。

- 定义路由
- Query String或者Body转换成 struct.
- 提供上下文保存web请求的值
- 提供多种返回类型（JSON, XML, Template, etc.）的函数.
- 方便添加，每个请求前后逻辑的中间件
- 提供 Auth, Logging, Recover, CORS等 中间件

> 当然也可以组合多个package来实现各个功能。比如，基于[negroni](https://github.com/urfave/negroni)中间件， [gorilla mux](https://github.com/gorilla/mux)或者 [httprouter](https://github.com/julienschmidt/httprouter)定义路由，使用 [render](https://github.com/unrolled/render) 类似的package来组合成大的形态。这种方式我将在下面重新介绍。

大部分web框架提供类似的功能，但选择 [echo](https://echo.labstack.com/)的理由？

首先排除了（[beego](https://beego.me/), [revel](https://revel.github.io/)）全栈行框架。虽然提供很多功能，但是它们的运行方式，使用方式都不符合Go的思想。（对“不符合Go的思想”希望有一个充分的讨论时间…^_^）。起初人气非常高的 [martini](https://github.com/go-martini/martini)，也听到了不符合Go的思想的意见，自称地球上最快的web框架[iris](https://github.com/kataras/iris)，有吹嘘的成分所以犹豫了。

之后有[gin](https://github.com/gin-gonic/gin)和 [echo](https://echo.labstack.com/)。虽然github的like数gin更多，但对echo更动心。实际上github的like数超过5000后用like数来决定框架是无意义的。受到了开发者们的响应，已经得到了验证。对echo更动心的原因是echo的社区比gin更活跃。用echo开发过服务，非常好用。

在选择web框架是没有考虑性能。

Go的package中有强调性能的package，但我无视此部分。iris每秒可以处理450,000以上的请求，但我们需要这么高行的的服务么？能每秒处理10,000以上请求的服务已经是非常成功的服务，多数Go框架可以轻松处理。只要不是需要超高并发的少数的服务，性能不是很重要因素。

# package 结构

多数Go的框架不强制目录结构。提供必要的功能，除外的让开发者决定。

所以像使用别的语言的方式构建过目录，模仿过github的有名的Go项目。其中找到了非常不错的项目就是[Package Oriented Design](https://www.goinggo.net/2017/02/package-oriented-design.html)。非常像Go的风格，所以构建成这样。

    $ tree
    .
    ├── account/
    │   ├── model.go
    │   └── service.go
    ├── cart/
    │   ├── model.go
    │   └── service.go
    ├── catalog/
    │   ├── model.go
    │   └── service.go
    ├── cmd
    |   └── server/
    |       ├── config.production.yml
    |       ├── config.staging.yml
    |       ├── config.yml
    |       ├── handler.go
    |       ├── main.go
    |       ├── main_test.go
    |       └── swagger.yml
    └── kit/
        ├── httpreq/
        ├── number/
        ├── scanner/
        └── transport/

各模块以library的形式开发成package，为了运行项目的package构建在cmd里。以这种方式构建web应用程序，让链接各模块的handler文件 /cmd/server/handler.go 越来越大，又出现根据各模块分离handler的必要。越来越接近分成controller目录和model目录的传统mvc项目形态。

[Package Oriented Design](https://www.goinggo.net/2017/02/package-oriented-design.html)中推荐的方式，github的许多Go项目的package结构更符合对多个项目library的package。在web中这种结构不太符合。不是给多个项目调用的服务，决定不这么构建目录。最后在web中使用了熟悉的controller/model的模式。

    $ tree
    .
    ├── controllers/
    ├── filters/
    ├── models/
    ├── static/
    │   ├── css/
    │   └── js/
    |── views/
    |   ├── 404.html
    |   ├── 500.html    
    |   ├── index.html
    |   └── layout/
    └── main.go

# 以前是怎样使用的?

前面说的方式是，经过1～2年开发后的最终产物。那之前是怎样使用的呢？介绍为什么现在不用那种方式也很有意义。

## negroni + gorilla mux + render

[negroni](https://github.com/urfave/negroni), [gorilla mux](https://github.com/gorilla/mux) ， [render](https://github.com/unrolled/render)是我推荐过的方式。[Go 语言web 入门](https://thebook.io/006806/)中提到过用 [多种多样的package来构建微框架](https://thebook.io/006806/ch09/)。

起初非常轻便的使用web框架，后来发现每次都在重复的构建。添加JWT验证中间件，添加session／cookie的使用法方法，还要设置rendering的方式。为什么每次都要组合多个package来实现相同的功能？有必要么？是不是使用基本功能都包含的框架更好一些？越来越对negroni方式产生了质疑。经过了多种开发方式，觉得echo非常适合。

## go-kit

和其他框架一样，收到很多推荐的go-kit, 但用起来为什么这么别扭...

包含circuit breaker, metrics, tracing等开发者们喜欢的内容，感觉必须要使用以A toolkit for microservices著称的go-kit来开发微服务。受到go-kit影响的框架，package也非常多...

直接使用感觉很不舒服，也没有什么开发文档，有很多example，但是这些代码也感觉不太友好。
这时候有了用go-kit的需求了。

我会在“Go My Way #3 - gomobile”中介绍我们公司的moblie app中使用了go-kit。有网络不太稳定时候（地下建筑中信号非常弱），需要压缩数据量。

解决的方法是使用go-kit提供的grpc。

使用go-kit的transport就可以转换成grpc。根据情况使用http或者grpc，多好的功能！

所以开始用go-kit来开发相关的项目。

- 用endpoint暴露相应的功能 
- 对每个endpoint 添加了 request, response转换代码
- client和 server中 添加了 transport layer的功能
- 对所有的 request, response 开发了 decode/encode 函数
- 为了使用grpc 对所用的reqeust, response 添加了 proto file.

就这样使用go-kit封装了代码，但是有个一个难点。

我们用NGINX来做API Gateway 的角色。为了服务的无中断发布用[Blue-Green Deploy](https://martinfowler.com/bliki/BlueGreenDeployment.html)的方式，动态变化的服务地址登记在 [consul](https://www.consul.io/)。

NGINX监听consul，根据服务地址的变化，变更NGINX的配置文件。

NGINX通过Reverse Proxy（反向代理），转发请求到相应的service。

最终，我们不暴漏实际的服务地址。

但是NGINX对grpc不能实现Reverse Proxy（反向代理）。所以使用grpc的服务只能用 [Caddy](https://caddyserver.com/)来做API Gateway的角色。

为了使用go-kit + grpc做了很多工作，一定要做到这个地步么？

所以又确认了一遍

之前用的http和grpc通讯没有太大的区别。通过NGINX压缩过的JSON数据量和grpc的数据量没有太大的差别。所以grpc的优势只有减少对CPU的使用量，我们的服务还不至于限制CPU使用量的地步。做这么多的工作比较可惜。circuit breaker, metrics, tracing等这些功能对我们来说不是必要的功能，有必要再用go-kit就可以了。

所以go-kit也OUT！

##  `net/http` 标准包的使用 

根据情况也可以直接使用标准包的net/http。不是太复杂的服务用net/http足矣。

比如，单独开发 [CAPTCHA](https://en.wikipedia.org/wiki/CAPTCHA)功能，就可以直接使用net/http。