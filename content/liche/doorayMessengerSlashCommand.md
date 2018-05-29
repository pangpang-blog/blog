---
title: "Dooray! > Messenger > slash command 介绍"
date: 2018-05-28T18:23:09+08:00
draft: true
---

## Dooray! > Messenger >  *slash command 介绍*

### 联动服务

在Dooray messenger中可以用命令语句的方式接收信息，并提高业务效率。想实现messenger中未提供的功能时，可以使用此功能并结合自己的业务，高效率的自动化完成业务。

要介绍的功能是如何使用command，以后会支持对bot、API等多样的联动服务。

### slash command

slash command是在`/`后面加字母的方式实现特定功能的命令语句。Dooray messenger提供 `/mute`, `/status`, `/search` 等基本语句。比如，查询信息内容，变更自己的状态等功能，在不使用鼠标点击情况下，通过命令快速实现。

![2](http://static.toastoven.net/prod_dooray_messenger/integration/2.png)

还可以通过自定义命令，来完成自己想要的功能。比如，每天早上接收交通情况、天气预报，通过输入简单的命名生成投票功能。

下图是command 服务和messenger服务的通信过程

![3](http://static.toastoven.net/prod_dooray_messenger/integration/3.png)

用户输入 `/`文字和对话窗登记过的command。输入的command通过messenger服务传输到command 服务。command 服务中把处理结果返回到messenger服务。messenger服务是通过command 服务返回的数据展示给用户。

### 使用环境

用户可以在pc和手机中输入command确认返回结果。所以，命令制作者需要考虑手机环境下的layout。目前只对pc支持自定义命令功能。

### 制作介绍

关于command命令的制作、登记，在以下文档中介绍。

------

## 添加command命令

为了使用自定义command命令，需添加Dooray messenger。

### 移动到添加页面

Dooray messenger的左上方点击自己的名字选择 '联动服务'。

![4](http://static.toastoven.net/prod_dooray_messenger/integration/4_2.png)

和下面一样，显示空画面。

![5](http://static.toastoven.net/prod_dooray_messenger/integration/5.png)

### app 添加

首先制作app。app是联动服务的捆扎单位。可以在一个app中添加多个command。输入下面的信息，制作app。各信息会在登记command时，显示在目录中。![6](http://static.toastoven.net/prod_dooray_messenger/integration/6.png)

| 구분        | 설명                                        |
| ----------- | ------------------------------------------- |
| Image       | 在对话窗中使用command时显示成传输者的icon。 |
| Name        | 在对话窗中使用command时显示成传输者的姓名。 |
| Description | app的介绍。                                 |

app创建成功。可以看见token和空的内容。token会在command请求时一同传送，参与请求验证。请注意不要泄漏token。如果token泄漏，请通过'Regenerate' 按钮重新发放token。

![7](http://static.toastoven.net/prod_dooray_messenger/integration/7.png)

### 添加command

登记app后，点击slash command '添加'按钮。建议在一个app中添加密切相关的command。

如果未制作过command，请添加下面的例`/hi` 。

![8](http://static.toastoven.net/prod_dooray_messenger/integration/8.png)

| 区分           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| Command        | 在对话窗输入包含`/`的命令。命令词最好是能表达内部功能的直观性 |
| Request URL    | 输入运行Command的服务器URL。                                 |
| Description    | 使用Command说明。让用户便于了解功能。                        |
| Parameter Hint | 介绍参数（可以输入地域，时间，用户，日期，文字，数字等）。   |
| Public         | 让其他人在对话窗中使用Command。                              |

成功添加了Command

![9](http://static.toastoven.net/prod_dooray_messenger/integration/9.png)

### Interactive Request URL 输入

通过按钮和下拉框菜单接收用户的事件，需要设置处理Interactive Message的URL。



![10](http://static.toastoven.net/prod_dooray_messenger/integration/10.png)

| 区分                             | 说明                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| Interactive Message Request URL  | 通过按钮和下拉框菜跟用户交互时，需要输入传递用户请求的URL。  |
| Interactive Message Optional URL | 如果message的菜单列表dataSource从外部提供，输入要请求菜单列表的URL。 |

------

## 发送"Hello World!" 消息

制作一个在对话窗中输入 `/hi`，回复"Hello world!"的Command。

### 执行Command

用户通过Dooray!messenger执行`/hi`Command时，Command服务会返回如下的JSON数据。

```
{
    "tenantId": "1234567891234567891",
    "tenantDomain": "guide.dooray.com",
    "channelId": "1234567891234567891",
    "channelName": "커맨드 가이드 채널",
    "userId": "1234567891234567891",
    "command": "/hi",
    "text": "/hi"
    "responseUrl": "https://guide.dooray.com/messenger/api/commands/hook/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "appToken": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "triggerId": "1234567891234.xxxxxxxxxxxxxxxxxxxx"
}
```

| 字段名称     | 说明                            |
| ------------ | ------------------------------- |
| tenantId     | 注册Command的tenent ID          |
| tenantDomain | 注册Command的tenentDomain       |
| channelId    | Command请求的对话窗ID           |
| channelName  | Command请求的对话窗名字         |
| userId       | Command请求的用户ID             |
| command      | 请求的Command                   |
| text         | Command请求的内容               |
| responseUrl  | 响应Command请求的web hook URL   |
| appToken     | 注册app 的token，验证请求时使用 |
| triggerId    | 触发事件的ID                    |

### 响应

Command服务会根据参数，返回数据。 `/hi` Command没有特殊的数据处理，返回`Hello World!`。

```
{
    "text": "Hello World!",
    "responseType": "ephemeral"
}
```

如上面一样响应时，只有执行Command的用户可以看见信息。

如果想共享给对话窗内的全部成员，设置 `responseType`为 `inChannel`。

```
{
    "text": "Hello World!",
    "responseType": "inChannel"
}
```

| 字段名称     | 默认值    | 说明                                                         |
| ------------ | --------- | ------------------------------------------------------------ |
| responseType | ephemeral | 设置message的显示type。- inChannel:显示给全部成员- ephemeral:只显示给调用Command的用户 |
| text         |           | message 内容                                                 |

## 发送message的4种方法

执行Command时，message服务有4种方式发送message。

- 首次发送信息
- 发送message后追加发送
- 更新已发送的message
- 删除已发送的message并发送新的messaage

### 首次发送message

首次发送message方法非常简单。

```
{
    "responseType": "inChannel",
    "text": "Hello World!"
}
```

### 追加发送message

`replaceOriginal`的值为 `false`会追加发送。

```
{
    "replaceOriginal": false,
    "responseType": "inChannel",
    "text": "Hello World!"
}
```

### 更新已发送的message

`replaceOriginal`值为`true`更新之前发送的message，也不会收到alert通知。但不能修改发送过的`responseType`的值，想改变 `responseType`的值需要重新发送。

```
{
    "responseType": "inChannel",
    "replaceOriginal": true,
    "text": "Hello World!(Updated)"
}
```

### 删除已发送的message并发送新的messaage

此操作会让参与话窗的成员收到alert通知。`deleteOriginal`值为 `true`时，删除已发送的message并发送新的messaage。

```
{
    "responseType": "inChannel",
    "deleteOriginal": true,
    "text": "Hello World!(Updated)"
}
```

请选择符合需要的发送message方法，制作command。

------

## 在message中添加按钮

响应的message可以根据 `attachments` 字段设置按钮。收到message的用户可以点击按钮来进行相互。 下面介绍通过选择按钮和点击按钮结果的处理方法。 

下面是包含attachments字段，在对话窗中是否要发送message的例子。

```
{
    "text": "Message",
    "attachments": [
        {
            "callbackId": "send-a1b2c3", // 用户之间有相互作用时，将同步发送。在识别出相互作用的attachment时可以使用。
            "actions": [
                {
                    "name": "send",
                    "type": "button",
                    "text": "Send", // 给用户输出的按钮文字
                    "value": "posting", // Action的动作（不会给用户显示）的值
                    "style": "primary" // 可以更改按钮的样式. primary, danger, default
                },
                {
                    "name": "send",
                    "type": "button",
                    "text": "Cancel",
                    "value": "cancel"
                }
            ]
        }
    ]
}
```

收到相应message时，显示 'Send'和'Cancel'按钮 。

![11](http://static.toastoven.net/prod_dooray_messenger/integration/11.png)

点击'Send'按钮，会把以下的数据发送到Command服务的Interactive Request URL中。

```
{
    "tenant": {
        "id": "1234567891234567891",
        "domain": "guide.dooray.com"
    },
    "channel": {
        "id": "1234567891234567891",
        "name" "command介绍channel"
    },
    "user": {
        "id": "1234567891234567891",
    },
    "commandName": "/post",
    "command": "/post",
    "text": "",
    "callbackId": "send-a1b2c3", // 用户选择的Action所属的Attachment callbackId
    "actionText": "Send", // actionText - 用户选择的 Action Text.
    "actionValue": "posting", // actionValue - 用户选择的 Action Value.
    "appToken": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "cmdToken": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "triggerId": "",
    "commandRequestUrl": "https://command.example.com/req",
    "channelLogId": "-7386965175150134411",
    "originalMessage": { /* 包含action的信息 */ }
    }
}
```

用户点击按钮的逻辑处理，需要在command服务器上实现。

------

## 在message中添加下拉框

attachments message中可以添加下拉框。

下面我们用大家喜爱的棋牌游戏，做一个投票的例子。

### 静态下拉框菜单

通过`options` 字段定义菜单内容。

```
"attachments": [
   {
       "name": "games_list",
       "text": "Pick a game...",
       "type": "select",
       "options": [
           {
               "text": "rock-scissors-paper",
               "value": "rcp"
           },
           {
               "text": "monopoly",
               "value": "monopoly"
           },
           {
               "text": "Checkers",
               "value": "checkers"
           },
           {
               "text": "Chess",
               "value": "chess"
           },
           {
               "text": "Poker",
               "value": "poker"
           },
           {
               "text": "scrabble",
               "value": "scrabble"
           }
       ]
   }
]
```

通过attachments字段，可以显示一下的下拉框菜单。

![12](http://static.toastoven.net/prod_dooray_messenger/integration/12.png)

### 动态下拉框菜单

动态下拉框菜单用`dataSource`来替代 `options` 。`dataSource`可以根据相应的值显示目录，成员，对话窗，外部数据。

| 区分       | 说明     |
| ---------- | -------- |
| dataSource | 目录     |
| users      | 成员     |
| channels   | 对话窗   |
| external   | 外部数据 |

#### 成员目录

设置`dataSource`的值为 `users`发送message时，可以显示当前对话窗的成员目录。用户可以在下拉框中输入搜索词，搜索全部成员。

```
"attachments": [
    {
        "type": "select",
        "name": "sel_users",
        "text": "사용자 출력",
        "dataSource": "users"
    }
]
```

![13](http://static.toastoven.net/prod_dooray_messenger/integration/13.png)

#### 对话窗目录

设置`dataSource`的值为`channels`发送message，可以显示用户所属的对话窗目录。

```
"attachments": [
    {
        "type": "select",
        "name": "sel_channels",
        "text": "대화방 출력",
        "dataSource": "channels"
    }
]
```

![14](http://static.toastoven.net/prod_dooray_messenger/integration/14.png)

#### 外部数据目录

设置`dataSource`的值为`external`发送message，可以显示外部数据目录。外部数据目录是从app注册时设置的 Interactive Optional URL中获取。

```
"attachments": [
    {
        "type": "select",
        "name": "sel_external",
        "text": "외부 데이터",
        "dataSource": "external"
    }
]
```

客户端会把以下数据发送到Interactive Optional URL，获取目录数据。

```
{
    "type": "interactive_message",
    "tenant": {
        "id": "1234567891234567891",
        "domain": "guide.dooray.com"
    },
    "channel": {
        "id": "1234567891234567891",
        "name": "Command 가이드 채널"
    },
    "user": {
        "id": "1234567891234567891",
        "name": "홍길동"
    },
    "callbackId": "sample",
    "actionName": "sel_externel",
    "actionTs": 1524734546105
}
```

command服务会根据上面的内容响应下拉框菜单。

```
{
    "options": [
        {
            "text": "external",
            "value": "value1"
        },
        {
            "text": "load",
            "value": "value2"
        },
        {
            "text": "success",
            "value": "value3"
        }
    ]
}
```

![15](http://static.toastoven.net/prod_dooray_messenger/integration/15.png)

------

## 发送attachments message

command可以发送attachments形态的特殊message。attachments组件中除了按钮和下拉框，还有其他多种多样的菜单。有效的运用attachments，可以提高沟通效率。

### attachments message

下面的attachment块是title,说明，图片，超链接，按钮，下拉框等菜单。最多20个attachment块组合成attachments message。

![16](http://static.toastoven.net/prod_dooray_messenger/integration/16.png)

| 序号 | 名字               | 说明                                                 |
| ---- | ------------------ | ---------------------------------------------------- |
| 1    | text               | message内容                                          |
| 2    | attachment         | message附加的内容。多个attachment组合构成attachments |
| 3    | authorName         | 作者名字。可以用authorLink添加link                   |
| 4    | title              | attachment题目                                       |
| 5    | text               | attachment内容                                       |
| 6    | thumbUrl           | attachment的thumb图片URL                             |
| 7    | imageUrl           | attachment图片URL                                    |
| 8    | field              | 根据short的值，一行显示一个或两个字段                |
| 10   | Interactive Menu   | 下拉框菜单                                           |
| 11   | Interactive Button | 按钮                                                 |

### Data

数据格式如下

```
{
    "text": "NHN IT News!",
    "attachments": [
        {
            "callbackId": "guide-a1b2c3",
            "text": "애플은 오늘 오전 2시에 WWDC를 통해 아이폰X 출시를 알렸다.",
            "title": "아이폰 X 출시",
            "titleLink": "https://dooray.com/",
            "authorName": "NHN News",
            "authorLink": "https://dooray.com/",
            "imageUrl": "http://it.chosun.com/data/photos/cdn/20180423/2850453_09555838720000.jpg",
            "thumbUrl": "http://www.kinews.net/news/photo/201804/119143_167793_5622.png",
        },
        {
            "fields": [
                {
                    "title": "출시 예정일",
                    "value": "2018년 겨울",
                    "short": true
                },
                {
                    "title": "예상 가격",
                    "value": "125만원",
                    "short": true
                }
            ]
        },
        {
            "fields": [
                {
                    "title": "설명",
                    "value": "한국 미출시",
                }
            ]
        },
        {
            "fields": [
                {
                    "title": "IOS",
                    "value": "High Sierra OS",
                }
            ]
        },
        {
            "actions": [
                {
                    "type": "select",
                    "text": "채널선택",
                    "name": "guide-sel",
                    "dataSource": "channels"
                }
            ]
        },
        {
            "actions": [
                {
                    "type": "button",
                    "text": "공유하기",
                    "name": "guide-btn",
                    "value": "btnValue"
                },
                {
                    "type": "button",
                    "text": "다음 뉴스",
                    "name": "guide-btn",
                    "value": "btnValue"
                },
            ]
        }
    ]
}
```

### 组件分析

仔细看一下组件的构成要素。

#### Message Object

| 字段名          | 默认值      | 说明                                                         |
| --------------- | ----------- | ------------------------------------------------------------ |
| text            |             | message内容                                                  |
| attachments     |             | Attachment数组                                               |
| responseType    | "ephemeral" | - "ephemeral": 展示给调用Command用户- "inChannel":展示给channel内的全部成员 |
| replaceOriginal | true        | 响应Interactive Message时，是否修改已发送的message           |
| deleteOriginal  | false       | 响应Interactive Message时，是否删除已发送的message           |

#### Attachment Object

| 字段名     | 默认值  | 说明                                    |
| ---------- | ------- | --------------------------------------- |
| text       |         | Attachment 内容                         |
| title      |         | Attachment 题目                         |
| titleLink  |         | 点击Attachment题目时，要移动的link      |
| authorName |         | Attachment 作者                         |
| authorLink |         | 点击Attachment 作者时，要移动的link     |
| fields     |         | Field의 排序                            |
| actions    |         | Action의 排序                           |
| callbackId |         | Action执行时的参数（维持session的作用） |
| imageUrl   |         | 图片地址                                |
| thumbUrl   |         | thumb地址                               |
| color      | #4757C4 | Attachment 颜色(HTML 颜色代码)          |

#### Field Object

| 字段名 | 默认值 | 说明                               |
| ------ | ------ | ---------------------------------- |
| title  |        | Field 题目                         |
| value  |        | Field 文字                         |
| short  | false  | Field 宽度设置(true时，宽度为一半) |

#### Action Object

| 字段名     | 默认值    | 说明                                                         |
| ---------- | --------- | ------------------------------------------------------------ |
| type       |           | Actiontype- "button": 按钮- "select": 下拉框菜单             |
| text       |           | 按钮下拉框显示的文字                                         |
| name       |           | 发送到command服务的字段名                                    |
| value      |           | 发送到command服务的值                                        |
| style      | "default" | 按钮颜色- "primary": 强调颜色- "default": 基本颜色           |
| options    |           | Option의 数组                                                |
| dataSource |           | 代替'options'定义的值-"users":成员目录- "channels":channel目录"external":数据从Interactive Message Optional URL获取 |

#### Option Object

| 字段名 | 默认值 | 说明                      |
| ------ | ------ | ------------------------- |
| text   |        | Option 文字               |
| value  |        | 发送到command服务的参数值 |

------

## 在对话窗中注册command

### 对话窗中注册command

command可以在1:1对话，group对话中注册并使用。打开添加command页面，有2个方法。

第一，通过Messenger右上方设置按钮添加。

![17](http://static.toastoven.net/prod_dooray_messenger/integration/17.png)

第二，通过在对话窗中输入 `/`后，显示的联动服务按钮添加。

![18](http://static.toastoven.net/prod_dooray_messenger/integration/18.png)

command添加页面中会显示公开的command或自己生成的command。把需要的command通过'添加'按钮添加到对话窗中。如果没有command，可以通过本上面的文章中介绍的方法制作command。

![19](http://static.toastoven.net/prod_dooray_messenger/integration/19.png)

![20](http://static.toastoven.net/prod_dooray_messenger/integration/20.png)

### 公开command



如果想把自己制作的command共享给其他成员，可以设置成公开。组织内的其他成员可以自由使用。

变更为非公开，已经添加command的成员也可以继续使用。如火不想让其他成员使用，请删除command。

------

## 例子：投票command

### command服务器要求

command服务需要提供可执行的REST API。

| API 种类                          | 说明                                        | 必填项 | 方法 |
| --------------------------------- | ------------------------------------------- | ------ | ---- |
| command Request URL               | 处理用户command运行请求的URL                | O      | POST |
| Interactive Message의 Request URL | 处理用户action（按钮点击，下拉框选择）的URL | X      | POST |
| Interactive Message의Optional URL | 在下拉框中使用外部数据的URL                 | X      | POST |

### 投票command

在对话窗中制作投票command的例子。

![21](http://static.toastoven.net/prod_dooray_messenger/integration/21.png)

#### API

只使用command Request URL 和 Interactive Message Request URL

#### 执行command格式

用户可以输入如下的投票command

```
/vote {题目} {项目1} "{空项目}" ... {项目n}
```

#### 步骤

1. 用户执行command
2. 输出只能调用command用户可见的投票massage
3. 通过点击生成按钮给对话窗中的成员输出投票message
4. 可以通过对话窗中按钮参与投票
5. 生成投票的用户结束投票
6. 输出投票结果

### 执行投票command

用户将如下进行投票command。

![22](http://static.toastoven.net/prod_dooray_messenger/integration/22_1.png)

command服务从Request UR接收json参数。

```
{
    "tenantId": "1234567891234567891",
    "tenantDomain": "guide.dooray.com",
    "channelId": "1234567891234567891",
    "channelName": "Command 가이드",
    "userId": "1234567891234567891",
    "command": "/vote",
    "text": "점심식사 짜장면 짬뽕 \"사천 탕수육\"",
    "responseUrl": "https://guide.dooray.com/messenger/api/commands/hook/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "appToken": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "triggerId": "1234567891234.xxxxxxxxxxxxxxxxxxxx"
}
```

| 字段名       | 说明                             |
| ------------ | -------------------------------- |
| tenantId     | 注册Command的tenent ID           |
| tenantDomain | 注册Command的Domain              |
| channelId    | Command请求的对话窗ID            |
| channelName  | Command请求的对话窗名字          |
| userId       | Command请求的用户ID              |
| command      | Command名                        |
| text         | 用户请求的Command内容            |
| responseUrl  | 请求command的对话窗的Webhook URL |
| appToken     | 注册app 的token，验证请求时使用  |
| triggerId    | 触发事件的ID                     |

### 执行command请求的响应

![23](http://static.toastoven.net/prod_dooray_messenger/integration/23_1.png)

执行command请求的响应，只显示给调用的用户。为了生成投票或者取消的按钮，给用户发送如下message。

```
{
    "responseType": "ephemeral",
    "text": "Click 'Submit' button to start the vote.",
    "attachments": [
        {
            "title": "점심식사",
            "fields": [
                {
                    "title": "Item 1",
                    "value": "짜장면",
                    "short": true
                },
                {
                    "title": "Item 2",
                    "value": "짬뽕",
                    "short": true
                },
                {
                    "title": "Item 3",
                    "value": "사천 탕수육",
                    "short": true
                }
            ]
        },
        {
            "callbackId": "vote",
            "actions": [
                {
                    "name": "vote",
                    "type": "button",
                    "text": "Submit",
                    "value": "점심식사 짜장면 짬뽕 \"사천 탕수육\"",
                    "style": "primary"
                },
                {
                    "name": "vote",
                    "type": "button",
                    "text": "Cancel",
                    "value": "cancel"
                }
            ]
        }
    ]
}
```

### 执行action请求

用户点击'Submit'按钮时，会向 Interactive Message의 Request URL发送如下数据。

```
{
    "tenant": {
        "id": "1234567891234567891",
        "domain": "guide.dooray.com"
    },
    "channel": {
        "id": "1234567891234567891",
        "name": "Command 가이드 채널"
    },
    "user": {
        "id": "1234567891234567891",
        "name": "홍길동"
    },
    "commandName": "/vote",
    "command": "/vote",
    "text": "점심식사 짜장면 짬뽕 \"사천 탕수육\"",
    "callbackId": "vote",
    "actionText": "Submit",
    "actionValue": "점심식사 짜장면 짬뽕 \"사천 탕수육\"",
    "appToken": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "cmdToken": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "triggerId": "1234567891234.xxxxxxxxxxxxxxxxxxxx",
    "commandRequestUrl": "https://command.guide.doc/req",
    "channelLogId": "-7386965175150134411",
    "originalMessage": { /* Message Object */ }
}
```

| 字段名            | 说明                                |
| ----------------- | ----------------------------------- |
| callbackId        | 用户选择的Action所属的Attachment ID |
| actionText        | 用户选择的actionText                |
| actionValue       | 用户选择的action值                  |
| commandRequestUrl | command Request URL                 |
| channelLogId      | message ID                          |
| originalMessage   | 上一个响应message                   |

### 对action的响应

![24](http://static.toastoven.net/prod_dooray_messenger/integration/24_1.png)

通过'Submit'按钮的响应，发送投票message。生成确认message不再需要，所以删除后重新创建message。

```
{
    "responseType": "inChannel", 
    "deleteOriginal": true, 
    "text": "(dooray://1234567891234567891/members/1234567891234567891 \"member\") created the vote!",
    "attachments": [
        {
            "callbackId": "1525223162093-(dooray://1234567891234567891/members/1234567891234567891 \"member\")",
            "title": "점심식사",
            "actions": [
                {
                    "name": "vote",
                    "type": "button",
                    "text": "짜장면",
                    "value": 0
                },
                {
                    "name": "vote",
                    "type": "button",
                    "text": "짬뽕",
                    "value": 1
                },
                {
                    "name": "vote",
                    "type": "button",
                    "text": "사천 탕수육",
                    "value": 2
                }
            ],
            "color": "#4286f4"
        },
        {
            "callbackId": "1525223162093-(dooray://1234567891234567891/members/1234567891234567891 \"member\")",
            "text": "",
            "actions": [
                {
                    "name": "vote",
                    "type": "button",
                    "text": "Close the vote (Show result)",
                    "value": "end"
                }
            ]
        }
    ]
}
```

| 字段名          | 必填项 | 说明                                |
| --------------- | ------ | ----------------------------------- |
| deleteOriginal  | false  | 在创建新message前是否删除已有的信件 |
| replaceOriginal | true   | 是否更新当前message                 |

之后的是用户点击按钮事件的请求和响应信息的反复。