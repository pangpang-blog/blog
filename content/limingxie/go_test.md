---
title: "Go项目中编写测试代码的经验"
date: 2018-03-09T18:14:08+08:00
draft: true
---
http://sabzil.org/writing-a-unit-test/

http://10.202.101.25:8090/pages/viewpage.action?pageId=12551088
 

备注：以下内容是我在学习过程所整理的内容，所以有些内容有可能不是正确。

为什么写测试代码?
我们做项目写代码的时候，已知会有这样的疑惑和担忧。

"我写的正确吗?"
"我这么编写其他地方不会受到影响吗?"

我们之所有这样的担忧，我认为应该是一下2种原因导致的。
第一，编写代码以后，很难构建针对开发内容发出请求和响应请求的模块。
第二，编程的时候会忘记之前的定好的原则，违反这些原则有可能会有一些错误。

根据"测试驱动开发 TDD 实践和工具" 和 几篇博客文章以及几本书，再写测试代码的感觉能找出问题的原因。

所以我决定在进行中的项目里适用测试代码。

测试代码？
写测试代码需要注意以下几点。
1. 通过 "提问 → 回答 → 优化 → 反复" 的过程，反复的编写测试代码，失败，再整理。
2. 测试的最小单位是函数，可以以函数为单位编写测试代码。
3. 设计的时候先定义行为然后在考虑行为的属性。

怎么写测试代码?
Go 语言里的测试代码可以根据官方提供的 “testing” 包编写。

可以使用第三方工具更便捷的编写测试代码。
1. testify ( go get -u github.com/stretchr/testify )
2. goconvey ( go get -u github.com/smartystreets/goconvey )

testify提供和 assert, http, mock, require, suite 一样的的包，更方便的让我们做检验。goconvey是更方便的让我们在网页里确认测试结果的工具。 

goconvey是我们运行 goconvey会展示以下内容，让我们一眼能看出测试结果。
scr 1
我们可以通过 coverage 确认以下内容。
scr 2

测试代码
测试函数的编写技巧

测试对象函数和名字 1:1的对应。

对象函数: Getbalance() {...}
测试函数: TestGetbalance(...) {...}
函数名前缀加test方便识别。

ex) func Test_Withdraw_存折取款(...) {...}
有测试剧本，也可以按照步骤取名。

ex) func Test_VIP顾客_取款_扣率计算(...) {...}
根据测试剧本确认正常流程的结果，或测试异常流程的处理结果。

Go 语言里写测试代码需要遵守几种规则。

文件名后缀需要加_test "xxx_test.go"。

测试函数的函数应开始于"Test"。

ex) func Test_Minus_小数_减去_大数(t *testing.T) { }
看上述例子可以看出需要接 testing.T。 testing.T 类型是包含 Error, Fail, Fatal, Log 这些测试需要的要素. (详细内容请参考 https://golang.org/pkg/testing/ )

测试代码例子(1)
写一个简单求和的方法来详细的讲解测试代码的编写。

```
// add_test.go
package add

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
package add

func Add(x, y int) int {
	return x + y
}
``` 

为了看结果需要运行以下命令。

$ go test -v
=== RUN   TestAdd
--- PASS: TestAdd (0.00s)
测试代码例子(2)
这次试用 testify/assert 包编写测试代码，之后运行 goconvey 通过游览器查看测试结果。

```
sample_1$ goconvey

2018/02/20 17:02:39 goconvey.go:61: Initial configuration: [host: 127.0.0.1] [port: 8080] [poll: 250ms] [cover: true]
2018/02/20 17:02:42 tester.go:19: Now configured to test 10 packages concurrently.
2018/02/20 17:02:42 goconvey.go:192: Serving HTTP at: http://127.0.0.1:8080
2018/02/20 17:02:42 integration.go:122: File system state modified, publishing current folders... 0 3038222316
2018/02/20 17:02:42 goconvey.go:118: Received request from watcher to execute tests...
2018/02/20 17:02:42 executor.go:69: Executor status: 'executing'
2018/02/20 17:02:42 coordinator.go:46: Executing concurrent tests: sample_1
2018/02/20 17:02:42 goconvey.go:105: Launching browser on 127.0.0.1:8080
2018/02/20 17:02:46 goconvey.go:113: ATTENTION: default value of option force_s3tc_enable overridden by environment.
创建新的游览器窗口.
```
```
// oprcfg.go

type CfgItem struct {
  Name        string
  Value       string
  Owner       string
}

type Config struct {
  Cfg map[string][]CfgItem
}

func NewConfig() (*Config, error) {
  oc := &Config{}
  oc.Cfg = make(map[string][]CfgItem)
  return oc, nil
}

func (oc *Config) GetOprCfg(owner string) []OprCfg {
  if owner == "" {
    return nil
  }

  globalCategory := "global"
  srchCategory := owner

  globalConfigItem := oc.Cfg[globalCategory]
  srchConfigItem := oc.Cfg[srchCategory]

  var resOpcf []OprCfg

  for _, cf := range globalConfigItem {
    opcf := OprCfg{Name: cf.Name, Value: cf.Value, Owner: cf.Owner}
    resOpcf = append(resOpcf, opcf)
  }

  for _, cf := range srchConfigItem {
    opcf := OprCfg{Name: cf.Name, Value: cf.Value, Owner: cf.Owner}
    resOpcf = append(resOpcf, opcf)
  }

  return resOpcf
}
```
Config 有创建对象的 NewConfig() 函数, Config里有给 owner返回 OprCfg的 GetOprCfg() 函数。
下面是测试代码。

```
// oprcfg_test.go

func Test_Config_New(t *testing.T) {
  // arrange

  // act
  oc, _ := NewConfig()

  // assert
  assert.NotNil(t, oc)
}

func Test_GetOprCfg(t *testing.T) {
  // arrange
  oc, _ := NewConfig()

  oc.SetCfgItem("key1", "value1", "key1-value1", "owner1")
  oc.SetCfgItem("key2", "value2", "key2-value2", "owner2")
  oc.SetCfgItem("key3", "value3", "key3-value3", "owner1")
  oc.SetCfgItem("key4", "value4", "key4-value4", "owner1")
  oc.SetCfgItem("key5", "value5", "key5-value5", "owner2")
  oc.SetCfgItem("key6", "value6", "key6-value6", "owner3")
  oc.SetCfgItem("key7", "value7", "key7-value7", "owner1")
  oc.SetCfgItem("key8", "value8", "key8-value8", "global")

  // act
  opcfs := oc.GetOprCfg("owner1")
  t.Log(opcfs)

  // assert
  var res []OprCfg
  res = make([]OprCfg, 5)

  // res에는 global이 먼저 쌓이고, 타겟 owner가 쌓인다
  res[0] = OprCfg{Name: "key8", Value: "value8", Owner: "global"}
  res[1] = OprCfg{Name: "key1", Value: "value1", Owner: "owner1"}
  res[2] = OprCfg{Name: "key3", Value: "value3", Owner: "owener1"}
  res[3] = OprCfg{Name: "key4", Value: "value4", Owner: "owner1"}
  res[4] = OprCfg{Name: "key7", Value: "value7", Owner: "owner1"}
  assert.True(t, assert.ObjectsAreEqual(&res, &opcfs))
}
```

创建 “Test” 开头的测试方法。
func Test_Config_New(t *testing.T) 里有一下注释


// arrange
...
// act
...
// assert
...
这是编写测试代码的一种模板。
arrange: 测试之前需要准备的代码。
act: 需要测试的实际方法。
assert: act的方法的结果确认可以写在这里。

第一个测试代码 Test_Config_New 是确认 Config 对象的创建是否正常。

arrange 这部分没有什么特意准备的内容所以省略了。
act 这部分需要确认的是使用 NewConfig() 函数创建 Config 对象。 $ assert 部分是确认创建的对象是否正确的过程，而且确认对象是不是nil，这时使用 testify/assert 包。

assert 包含着很多验证的功能，assert.NotNil是判断对象是不是nil的方法。assert多样的功能可以参照 godoc( https://godoc.org/github.com/stretchr/testify/assert ) 。

第二个测试代码 Test_GetOprCfg函数是 Config内包含多种 owner的 CfgItem, 查询包含 golbal和owner的CfgItem转换成 OprCfg 返回的测试。


arrange 部分是为了测试 GetOprCfg 函数，先创建 Config 对象以及使用 (SetCfgItem) 赋随机值(CfgItem) 。
act 部分是 GetOprCfg 函数里传 "owner1” 返回 OprCfg 的值。
assert 部分是确认 OperCfg 值是不是有效的值（global 和 owner1）。

assert 包的 ObjectsAreEqual是确认两个 Object 是否一致。
assert 包 True是确认传进来的值是不是true.

可以这么写最基本的测试代码。但是这也有局限的，比如以下情况下写测试代码比较困难。

同步和并发问题

访问控制

GUI

依赖模块的测试

上述问题可以先不考虑(为什么? 因为我是写测试代码的新手！)

测试代码例子(3)
存在依赖的时候可以使用测试替身（test double）。

为了时间保存密码的功能，我们需要密码加密模块。预先知道加密的接口和加密的方式，我们可以把加密的模块做成Mock。 
已经确定好的接口功能定义为Cipher，之后再声明一个实现接口的 MockMD5Cipher，返回 encrypt 和decrypt 内部的 “potato”， potato的 md5 hash 值.无条件的返回hash值
根据以上情况，我们可以预想依赖 MessageQueue的开发情景。写测试代码的时候MessageQueue的行为可以视为测试替身。

"测试替身" 的用语是来与影视剧的替身。
测试替身可以分为以下几种类型 "Dummy", "Stub", "Fake", "Spy", "Mock" 

Dummy: 之声明一个空壳。毫无功能。 (只需要一个对象的时候视同)

Stub: 有返回值。(我们可以直接写硬代码返回结果)

Fake: 比Stub复杂，里面有逻辑处理。

Spy: 有输入值和输出值。

Mock: Mock多用于行为的验证。

下面吧依赖 MessageQueue 的部分做成测试替身。
编写确认变更事件的GetChangeEventInfo 函数。
测试代码的所有逻辑是这样。

调用GetChangeEventInfo函数的时候，会有事件变更的请求。

GetChangeEventInfo 函数使用接收 eventID 值给 和MessageQueue的 Agent，发送事件变更请求。

Agent是从 Database查询结果传输给 MessageQueue。

sample_1可以手机通过MessageQueue传输的响应。

确认返回的结果值。

写测试代码之前的代码是这样的

```
func (d *Agent) GetChangeEventInfo(eventID string) (*EventInfo, error) {
  msg := MakeChangeMsg(“blah-blah-chg-key”, “agent”)

  eventUUID := UUID_Data { UUID: eventID }
  any, _ := Any.Encoding(&eventUUID)
  msg.Payload(any) /* message queue 有关的其他功能 ... */ SendMessage(msg) val := RecvMessage() data := val.GetPayload()
  var ei EventInfo
  Any.Decoding(data[0], &ei)

  return &ei, nil
}
```

以上代码是 sample_1 对(MessageQueue)的事件区分的部分
还有如果向MessageQueue发出请求会重复的运行。所以这部分可以提取函数。

```
func (d *Agent) GetChangeEventInfo(eventID string) (*EventInfo, error) {
  sendInfo := &MqSendInfo{
    key:     “blah-blah-chg-key”,
    recv: “agent”,
  }

  eventUUID := UUID_Data{ UUID: eventID }
  any, _ := Any.Encoding(&eventUUID)

  anyData, err := MqSend(sendInfo, any)
  if err != nil { return nil, err }

  var ei EventInfo
  Any.Decoding(anyData[0], &ei)

  return &ei, nil
}
```
我们可以写一个(MqSend)函数，MqSendInfo可以作文参数穿进去

 
```
type MqSendInfo struct {
  key     string
  recv string
}

func MqSend(info *MqSendInfo, any *Any) ([]*Any, error) {
  msg := MakeChangeMsg(info.key, info.recv)

  msg.Payload(any)

  /* message queue 有关的其他功能 ... */

  SendMessage(msg)
  val := RecvMessage()

  return val.Msgs, nil
}
```
这么写可以减少很多代码，但是MessageQueue的依赖依然存在。

使用我们传的eventID值MqSend让函数返回。是否调用MqSend和使用函数的结果GetChangeEventInfo的结果是我们所想要的结果就能判断测试成功与否。

所以想创建一个MqSend函数的测试替身。
先把 MqSendInfo 结构和 MqSend 函数的名字改成 MqMsg 和Send方法

```
type Messenger interface {
  Send(any *Any) ([]*Any, error)
}

type MqMsg struct {
  key     string
  recv string
}

func (m *MqMsg) Send(any *Any) ([]*Any, error) {
  msg := MakeChangeMsg(m.key, m.recv)

  msg.Payload(any)

  /* message queue 有关的其他功能 ... */

  SendMessage(msg)
  val := RecvMessage()

  return val.Msgs, nil
}
```

现在我们可以这么写测试代码

 
```
type MockAgentMsgr struct {
  mock.Mock
}

func (m *MockAgentMsgr) Send(any *Any) ([]*Any, error) {
  args := m.Called(any, wait)
  return args.Get(0).([]*Any), args.Error(1)
}

func Test_GetChangeEventInfo(t *testing.T) {
  // arrange

  // 输入值
  eventID := "event--uuid-1"

  UUID := UUID_Data { UUID: eventID }
  par1, _ := Any.Encoding(&UUID)

  // 输出值
  var resAnytypes []*Any
  res1 := EventInfo { UUID: eventID, Name: "event1", Type: ET_START }

  resAnytype1, _ := Any.Encoding(&res1)
  resAnytypes = append(resAnytypes, resAnytype1)

  m := &MockAgentMsgr{}
  m.On("Send", par1).Return(resAnytypes, nil)

  agentMg := NewAgent(m)

  // act
  event, err := agentMg.GetChangeEventInfo(eventID)

  // assert
  assert.Equal(t, nil, err)
  assert.True(t, assert.ObjectsAreEqual(&res1, event))
  assert.True(t, m.AssertCalled(t, "Send", par1, true))
}
```
为了事项MqMsg的测试替身 Messenger 接口定义 Send 方法。
MockAgentMsgr 结构是实现 Messenger 接口的测试替身。
MockAgentMsgr的 Send 方法是需要指定输入值和输出值。
现在需要指定 Test_GetChangeEventInfo 函数的输入值 Send 方法的返回值。MessageQueue里做任何操作，根据输入值返回处处志。所以编写的代码应该是这样。

```
m := &MockAgentMsgr{}
m.On("Send", par1).Return(resAnytypes, nil)
```
调用Send 方法的时候根据输入值定义输出值。

但是现在执行代码的机构是只能调用规定好的的(MqMsg的 Send 方法). 需要修改这些。 所以声明 Agent 对象的时候也指定MessageQueue。

```
type Agent struct {
  Msgr Messenger
}

func NewAgent(msgr Messenger) *Agent {
  return &Agent{Msgr: msgr}
}

func (d *Agent) GetChangeEventInfo(eventID string) (*EventInfo, error) {
  eventUUID := UUID_Data{ UUID: evnetID }
  any, _ := Any.Encoding(&eventUUID)

  anyData, err := d.Msgr.Send(any)
  if err != nil { return nil, err }

  var ei EventInfo
  Any.Decoding(anyData[0], &ei)

  return &ei, nil
}
```
现在可以向测试代码，NewAgent声明Agent对象之前先传Messenger. (但是实际使用 Agent的时候有重复代码，)

不用为了实现 GetChangeEventInfo 函数，链接 MessageQueue 做调试，测试代码可以确保 sample_1 里需要实现的内容。


结论
只要输入值和 MessageQueue 对象输入值的定义明确的话, 不直接链接MessageQueue我们也可以实现 sample_1 的功能。
还有违背编写代码的时候原则也可以直接用测试代码确认，不用做另一种文档的管理。