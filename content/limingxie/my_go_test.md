---
title: "Go项目的测试代码"
date: 2018-03-15T20:36:02+08:00
draft: true
---

说明为什么写测试代码？ 怎么写测试代码？ 的文章很多，这里就不多说了。

直接来点干货，看代码。

```
// add_test.go
package models

import (
	"testing"
)

func TestAdd(t *testing.T) {
	//arrange
	var x, y, res int
	x = 2
	y = 3

	//act
	res = Add(x, y)

	//assert
	if res != 5 {
		t.Fatal("Add的结果不正确")
	}
}
```

```
// add.go
package models

func Add(x, y int) int {
	return x + y
}
``` 

为了看结果需要运行以下命令。

```
$ go test -v
=== RUN   TestAdd
--- PASS: TestAdd (0.00s)
PASS
ok      gotest/test5    0.006s
```

这里先简单的解释一下编写测试代码的普遍公认的模板。

1）arrange: 测试之前需要准备的代码。

2）act: 需要测试的实际方法。

3）assert: act的方法的结果确认可以写在这里。



**项目中的应用**


