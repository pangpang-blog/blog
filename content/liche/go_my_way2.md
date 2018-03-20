---
title: "Go My Way #2 - DataBase, Logging"
date: 2018-03-09T18:23:37+08:00
draft: true
---

# [Go My Way #2 - DataBase, Logging](https://jaehue.github.io/post/go-my-way-2-database-and-logging/)

Jun 14, 2017

**Go My Way**是介绍我用Go语言开发web应用时首选方案的3篇文章 . 没读过之前的文章，请下面的链接.

- [Go My Way #1 - web 框架](https://jaehue.github.io/post/go-my-way-1-webframework/)
- Go My Way #2 - DataBase, Logging
- [Go My Way #3 - Configuration, Tracing, etc.](https://jaehue.github.io/post/go-my-way-3-tracing/)
- gomobile

这篇文章介绍`数据库`和`日志`

# 数据库

跟其他语言的ORM（Ruby的active record，.Net的entityframework,Java的JPA,等）相比，Go的DB相关的Package不足之处较多。但是Go社区出现了相关讨论的文章。“什么样的Go ORM 框架优秀? ”的问题([Golang which ORM is better](https://www.reddit.com/r/golang/comments/3ajqa6/golang_which_orm_is_better/))中，大部分人的意见是提供轻便的query mapper(执行sql语句的返回值绑定到Struct)功能就足够。也有推荐用native query(sql 语句，或者转化成sql语句)开发的文章([Our Go is fine but our SQL is great](https://medium.com/bumpers/our-go-is-fine-but-our-sql-is-great-b4857950a243))。

根据Go社区的趋势判断，不会出现 Active Record或者 Hibernate类似的ORM框架。

本人以前也认为操作DB时应该使用ORM，相比native query，使用ORM提供的抽象化的DSL是更明智的选择。为什么会这么想？应该用面向对象的方式开发代码，觉得用二维的方式操作表是开发水平低下的表现。

应该分开考虑modeling和实际DB处理的方式。定义好模型，基于模型开发项目，DB的处理方式不是很重要。不管你用native query,ORM还是混合使用，找到最佳的方式最重要。一直使用Go语言开发项目，解决问题时相比理论更关注实用性(这才是Go的哲学)。关于ORM，Go社区也一定程度的同意这种观点。(不同意怎么办？想自己开发一个，实力不足，只能适应)。

对糟糕的Go ORM工具的辩解，到此为止！

## xorm

最终我选择的是[xorm](http://xorm.io/)。之前一直使用[gorm](https://github.com/jinzhu/gorm)，但是xorm的有个功能让我非常满意。字段的类型不是built-in 时(slice, map, custom type)xorm会自动转换成json字符串。

比如，

```
type Product struct {
	Id     int64
	Images map[string]Image
}
type Image struct {
	URL    string
	Width  int32
	Height int32
}
```

把Product的Images字段定义成`map[string]Image`，xorm会把DB表中的images的字段类型定义成text，把 `map[string]Image` 保存成JSON字符串。DB中读取数据也是如此，会把JSON字符串自动转换成`map[string]Image`类型。也可以重写关于实际DB字段的绑定规则。只要实现[`Conversion` interface](https://github.com/go-xorm/core/blob/master/converstion.go#L5) 接口中定义的方法。

```
// https://github.com/go-xorm/core/blob/master/converstion.go#L5
type Conversion interface {
    FromDB([]byte) error
    ToDB() ([]byte, error)
}
```

我们公司的业务经常变动，所以很难定义模型。这种动态的数据，不定义明确的字段类型，直接转成JSON保存在一个字段中。在没有使用xorm之前，为了保存上面例子中的Images，会把保存JSON字符串用途的 `ImagesRaw` 字段定义成`string`类型，DB中只保存`ImagesRaw`字段。每次对`Product`做CRU(Create/Read/Update)操作时都要手动转成JSON。这是个非常麻烦的事情。xorm中内置了这种功能，所以非常方便。

## 跟其他DB工具的比较

[awesome-go](https://github.com/avelino/awesome-go#database)中提供了许多关于DB的package(跟web framework一样多)。其中只简单介绍本人使用过的package。

> 作为参考，可以相信[awesome-go](https://github.com/avelino/awesome-go)中提供的package，并使用它。awesome-go不是盲目的记录package的，他们有内部的[检验标准](https://github.com/avelino/awesome-go/blob/master/CONTRIBUTING.md#quality-standards)。不符合标准也会被拒绝，已记录的package，不符合标准时也会被清除。比如， [Remove iris from listing #1135](https://github.com/avelino/awesome-go/pull/1135)中 [iris](https://github.com/kataras/iris)遭到了清除。之后iris的作者再次申请还是被拒绝。如此看来，他们对质量的要求还是比较严格。

## [gorp](https://github.com/go-gorp/gorp)

会把query的执行结果绑定到struct中。

自称为`an ORM-ish library for Go`，但不是很像ORM。

单纯的query mapper。

类似于韩国开发者熟悉的ibatis。

## [sqlx](https://github.com/jmoiron/sqlx)

单纯的query mapper，跟gorp类似。

相比gorp，sqlx的社区比较活跃，使用者也较多。

单纯的想把query的执行结果绑定到struct，推荐使用sqlx。

## [gorm](https://github.com/jinzhu/gorm)

基于struct提供CRUD的功能。(基本的ORM功能)

模型之间可以定义Associations*(belongs-to, has-one, has-many, many-to-many, polymorphism)*。

但是实际使用当中有诸多不便。不会自动根据模型的关系获取数据。

```
type User struct { 
        ID       int64
        Emails   []Email
}
type Email struct {
        ID      int64
        UserID  int64
}
```

如上所述，用has-many关系定义模型时， 想获取`User`的 `Emails`，需要添加以下代码。

```
db.Model(&user).Related(&emails)
```

实际使用，可以发现gorm提供的Associations功能不会有太大的帮助。

但是gorm的like数最多。

## [xorm](http://xorm.io/)

基于struct提供CRUD的功能，跟gorm类似。

但是没有定义模型之间的Associations功能。

提供缓存功能。

字段的类型不是built-in(基本数据类型，比如 int，string，bool等)时，自动转换成JSON。

## [squirrel](https://github.com/Masterminds/squirrel)

轻便的 query builder。基于Go代码生成query。

类似这种感觉。

```
sql, args, err := sq.
        Select("*").
        From("users").
        Join("emails USING (email_id)").
        Where(sq.Eq{"deleted_at": nil}).
        ToSql()
// SELECT * FROM users JOIN emails USING (email_id) WHERE deleted_at IS NULL
```

```
sql, args, err := sq.
        Insert("users").Columns("name", "age").
        Values("moe", 13).Values("larry", sq.Expr("? + 5", 12)).
        ToSql()
// INSERT INTO users (name,age) VALUES (?,?),(?,? + 5)
```

但是想执行这样开发的query时，必须使用 `database/sql` package执行语句，接受返回值。squirrel和sqlx一起使用也是个好方式。(既然如此，还不如使用其他的package...)

## 运用

为了处理数据库选择package时，在实际开发当中如何使用DB对象的案例有许多。几天前在facebook的Golang Korea组织中提出了许多[关于DB对象管理的问题](https://www.facebook.com/groups/golangko/permalink/772578392919642/)，也有许多人对此提出了好的答复。

本人开发web应用程序时，在 `main`中创建DB对象，再通过中间件向每个request创建Session，并传递到request内部 `context`中。这是为了管理事务。为了消除在handler中管理事务的不便，把事务管理迁移到中间件。

```
main(){
        /* ... */
	db, err := xorm.NewEngine(driver, connection)
	if err != nil {
		panic(err)
	}
	defer db.Close()

        e := echo.New()
        e.Use(dbContext(db))
        /* ... */
}

func dbContext(db *xorm.Engine) echo.MiddlewareFunc {	
	return func(next echo.HandlerFunc) echo.HandlerFunc {
		return func(c echo.Context) error {
			session := db.NewSession()
			defer session.Close()

			req := c.Request()
			c.SetRequest(req.WithContext(
				context.WithValue(
					req.Context(),
					"DB",
					session,
				),
			))

			switch req.Method {
			case "POST", "PUT", "DELETE":
				if err := session.Begin(); err != nil {
					return echo.NewHTTPError(500, err.Error())
				}
				if err := next(c); err != nil {
					session.Rollback()
					return echo.NewHTTPError(500, err.Error())
				}
				if c.Response().Status >= 500 {
					session.Rollback()
					return nil
				}
				if err := session.Commit(); err != nil {
					return echo.NewHTTPError(500, err.Error())
				}
			default:
				if err := next(c); err != nil {
					return echo.NewHTTPError(500, err.Error())
				}
			}

			return nil
		}
	}
}
```

database connection的 scoped是全局的，为了管理事务在每个request中创建session时，database session scoped的单位是request。此时觉得在request scoped为单位管理的Context中保存比较正确。访问DB时需要在Context中获取DB session。

```
func (Model) GetById(ctx context.Context, id int64) (*Model, error) {
	db := ctx.Value("DB").(*xorm.Session)
	var v Model
	if has, err := db.ID(id).Get(&v); err != nil {
		return nil, err
	} else if !has {
		return nil, nil
	}
	return &v, nil
}
func (d *Model) Update(ctx context.Context) (err error) {
	db := ctx.Value("DB").(*xorm.Session)
	_, err = db.ID(d.Id).Update(d)
	return
}
```

但是无需管理事务，边界明确的模型中，直接创建全局的DB对象也是一个不错的方式。

这种方式也已适用在 [第一章](https://jaehue.github.io/post/go-my-way-1/)介绍的 [echosample](https://github.com/pangpanglabs/echosample)当中。

# Logging

在介绍Logging之前，对Go糟糕的`log` package做一下小小的辩解。

Go中提供了 `log` package，但只能在 `io.Writer`中输出。多数日志是无法保存日志等级，对此让我重新思考。

什么是`warning`？又不是出现了error，可能以后会成为error？还不如把日志的等级定义成 `info`或 `error`，定义成 `warning`情况是不是微乎其微(有可能会出现)？`warning`和 `info`的区分有点模糊。

`fatal` 的等级又是什么？Go的标准包的`log`中`log.Fatal()`输出日志的同时会结束程序。多数 leveled logging的package都类似。既然如此直接使用`log.Fatal()`就可以，为什么还要使用 leveled logging 方式呢？

关于`error`等级需要思考。当记录`error`等级的日志时需要保存出现error值时的处理error前的 `error` 等级。但是处理过的error就不能算是error了。这时候应该是 `info`。呵呵… ,`error`和 `info`变成一样的了。

日志在information输出。只是这样。用最少的功能开发`log package`，就跟Go标准包提供的`log`一样了。

但实际上多数是用`level-based-logging` package(我也在用)。

## logrus

本人使用 [logrus](https://github.com/sirupsen/logrus)。使用时会发现最方便。使用logrus也经过了长时间的旅程。

实际上最初使用的`log package`也是logrus。但因为 [zap](https://github.com/uber-go/zap)的[性能炫耀](https://github.com/uber-go/zap#performance)也有一段时间使用过zap。性能好不好无从所知，但使用起来不方便。用zap记录日志时必须定义字段类型。

```
logger.Info("Failed to fetch URL.",
	zap.String("url", url),
	zap.Int("attempt", 3),
	zap.Duration("backoff", time.Second),
)
```

非常麻烦。想记录struct全部的值，需要定义每一个字段的类型。但考虑到性能，这样使用是正确的。开发中想轻便的记录日志、运营环境下想临时记录日志的情况也很多。此时还是记录整个值比较方便，像这样一一定义字段类型是非常麻烦的事情(只能使用built-in类型)。

使用logrus指定的Formatter(JsonFormatter, TextFormatter, Custom Formater)可以记录任何值。使用TextFormatter像 `fmt.Print` 一样用标准输出格式记录日志。而且log的外观漂亮(日志漂亮，才能提高开发效率)，还可以在每个`log action`中添加Hook。有人已经开发了[不错的 Hook](https://github.com/sirupsen/logrus#hooks)。

我们通过kafka把日志传送到Hadoop，必要时利用 [presto](https://prestodb.io/)查看日志。想给日志添加事件时使用logrus的Hook功能比较方便。

[go-kit log](https://github.com/go-kit/kit/tree/master/log)也好用，但go-kit log也只能保存built-in类型，不太方便。不需要使用logrus的多种功能，对性能要求比较高时，使用[zap](https://github.com/uber-go/zap)也是不错的选择。

## 运用

在实际运营环境下，在同一个时间点会处理多个requset。此时通过log中的timestamp很难推测request是怎样被处理的。在每个请求中分配request_id，并记录在log中。就算有多个混淆的request，也可以进行推测。

所以在每个request中创建带有request_id的log对象，并传送到context。

```
func main() {
	e := echo.New()

	// 각 request마다 고유의 ID를 부여
	e.Use(middleware.RequestID())
	e.Use(Logger())

        /* ... */

}

func Logger() echo.MiddlewareFunc {
	logger := logrus.New()
        /* ... logger 초기화 */
	return func(next echo.HandlerFunc) echo.HandlerFunc {
		return func(c echo.Context) error {
			logEntry := logrus.NewEntry(logger)                        

			// request_id를 가져와 logEntry에 셋팅
			id := c.Request().Header.Get(echo.HeaderXRequestID)
			if id == "" {
				id = c.Response().Header().Get(echo.HeaderXRequestID)
			}
                        logEntry = logEntry.WithField("request_id", id)

			// logEntry를 Context에 저장
			req := c.Request()
			c.SetRequest(req.WithContext(
				context.WithValue(
					req.Context(), 
					"LOG", 
					logEntry,
				),
			))

			return next(c)
		}
	}
}
```

echo app中添加 `middleware.RequestID()` 中间件，对每个request赋予固定的ID。把生成的 `request_id`添加到 `logEntry`，并保存在Context。使用log时，从Context中获取log对象。

```
logger := ctx.Value("LOG").(*logrus.Entry)
logger.WithFields(logrus.Fields{
	"url":     url,
	"attempt": 3,
	"backoff": time.Second,
}).Info("Failed to fetch URL.")
```

像这样记录日志， `url`, `attempt`, `backoff` 的值会跟保存在中间件的 `request_id`一起输出。

------

**这次主题中讨论了多种多样的数据库和日志的实现方法。**

**有更好的方案，希望一起讨论。**

**不要犹豫，请给我意见^ ^**
