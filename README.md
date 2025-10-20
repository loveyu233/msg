# Go 消息发送库

这是一个 Go 语言编写的消息发送库，集成了企业微信机器人、阿里云短信和微信公众号功能，方便快速地发送各种通知和消息。

## 功能特性

- **企业微信机器人**
  - 支持文本、Markdown、图文、文件等多种消息类型。
  - 支持文件上传（语音、普通文件）。
  - 链式调用和便捷方法，易于集成和使用。

- **阿里云短信**
  - 对接阿里云官方的 `dysmsapi` 服务。
  - 提供简单的初始化和发送接口，快速发送模板短信。

- **微信公众号**
  - 基于 `PowerWeChat` 封装，提供完整的公众号服务端能力。
  - 支持接收并处理用户消息和事件（如关注、取关）。
  - 支持发送模板消息。
  - 以 Gin 中间件方式集成，方便与现有 Web 项目整合。

## 安装

```bash
go get github.com/loveyu233/msg
```

## 使用方法

### 1. 企业微信机器人

首先，你需要获取企业微信群聊机器ンの Webhook Key。

#### 初始化客户端

在使用前，需要先初始化一个全局客户端实例。推荐在程序启动时进行初始化。

```go
package main

import (
	"log"
	"github.com/loveyu233/msg" 
)

func main() {
	// 你的企业微信机器人 Webhook Key
	webhookKey := "YOUR_WEBHOOK_KEY" 
	msg.InitQWRobotClient(webhookKey)

	// ... 接下来可以调用发送方法
}
```

#### 发送消息示例

```go
// 发送文本消息
_, err := msg.InsQWRobot.SendText("这是一条来自 Go 的测试消息")
if err != nil {
	log.Fatalf("发送文本消息失败: %v", err)
}

// 发送 Markdown 消息
markdownContent := `
# 一级标题
> 引用文本
**加粗**
`
_, err = msg.InsQWRobot.SendMarkdown(markdownContent)
if err != nil {
	log.Fatalf("发送 Markdown 消息失败: %v", err)
}

// 发送文件消息 (先上传，后发送)
// mediaType 可以是 msg.QYWXMediaTypeFile (文件) 或 msg.QYWXMediaTypeVoice (语音)
_, err = msg.InsQWRobot.SendFile("/path/to/your/file.txt", msg.QYWXMediaTypeFile)
if err != nil {
    log.Fatalf("发送文件失败: %v", err)
}
```

### 2. 阿里云短信

你需要准备阿里云的 AccessKey ID 和 AccessKey Secret。

#### 初始化客户端

推荐在程序启动时进行初始化。

```go
package main

import (
	"log"
	"github.com/loveyu233/msg"
)

func main() {
	accessKeyId := "YOUR_ACCESS_KEY_ID"
	accessKeySecret := "YOUR_ACCESS_KEY_SECRET"

	err := msg.InitShortMsgSimpleClient(accessKeyId, accessKeySecret)
	if err != nil {
		log.Fatalf("初始化阿里云短信客户端失败: %v", err)
	}
    
    // ... 接下来可以调用发送方法
}
```

#### 发送短信示例

```go
// 接收短信的手机号
targetPhoneNumber := "188xxxxxxxx"
// 短信签名名称
signName := "你的短信签名"
// 短信模板CODE
templateCode := "SMS_12345678"
// 短信模板变量对应的JSON字符串
templateParam := `{"code":"666888"}`

err := msg.InsShortMsg.SendSimpleMsg(targetPhoneNumber, signName, templateCode, templateParam)
if err != nil {
	log.Fatalf("发送短信失败: %v", err)
}

log.Println("短信发送成功！")
```

### 3. 微信公众号 (WeChat Official Account)

本库还提供了对微信公众号消息的支持，包括接收用户消息、事件推送以及发送模板消息。

#### 初始化服务

公众号服务的初始化需要提供公众号的配置以及事件处理的实现。

```go
package main

import (
    "fmt"
    "log"
    "github.com/ArtisanCloud/PowerWeChat/v3/src/kernel/contract"
    "github.com/ArtisanCloud/PowerWeChat/v3/src/officialAccount/user/response"
    "github.com/gin-gonic/gin"
    "github.com/loveyu233/msg"
)

// 1. 实现事件处理接口
type MyWXOfficialImp struct{}

func (h *MyWXOfficialImp) Subscribe(rs *response.ResponseGetUserInfo, event contract.EventInterface) error {
    fmt.Printf("用户 %s 关注了公众号
", rs.OpenID)
    // 在这里处理用户关注逻辑，例如保存用户信息到数据库
    return nil
}

func (h *MyWXOfficialImp) UnSubscribe(rs *response.ResponseGetUserInfo, event contract.EventInterface) error {
    fmt.Printf("用户 %s 取消了关注
", rs.OpenID)
    // 在这里处理用户取关逻辑
    return nil
}

func (h *MyWXOfficialImp) PushHandler(c *gin.Context) (toUsers []string, message string) {
    // 这是自定义主动推送接口 /wx/push 的逻辑
    // 示例：从请求中获取要推送的用户和消息
    // toUsers = []string{"USER_OPENID_1", "USER_OPENID_2"}
    // message = "这是一条群发消息"
    return
}


func main() {
    // 2. 配置公众号信息
    config := msg.OfficialAccountAppServiceConfig{
        OfficialAccount: msg.OfficialAccount{
            AppID:         "YOUR_APP_ID",
            AppSecret:     "YOUR_APP_SECRET",
            MessageToken:  "YOUR_TOKEN",
            MessageAesKey: "YOUR_AES_KEY", // 加密模式需要
            HttpDebug:     true,
        },
        WXOfficialImp: &MyWXOfficialImp{},
    }

    // 3. 初始化服务
    wxOfficialService, err := msg.InitWXOfficialAccountAppService(config)
    if err != nil {
        log.Fatalf("初始化公众号服务失败: %v", err)
    }

    // 4. 注册到 Gin 路由
    router := gin.Default()
    apiGroup := router.Group("/api") // URL前缀
    wxOfficialService.RegisterHandlers(apiGroup)

    // 启动服务
    // router.Run(":8080")
    
    // 你现在可以使用 wxOfficialService 实例来调用方法
}
```

#### 发送模板消息

初始化服务后，你可以使用 `PushTemplateMessage` 方法向指定用户发送模板消息。

```go
// wxOfficialService 是前面步骤中初始化得到的实例

// 定义模板消息的数据结构
type MyTemplateData struct {
    First    string `json:"first"`
    Keyword1 string `json:"keyword1"`
    Keyword2 string `json:"keyword2"`
    Remark   string `json:"remark"`
}

// 准备数据
data := MyTemplateData{
    First:    "您好，您的订单已发货。",
    Keyword1: "KF20251020",
    Keyword2: "顺丰速运",
    Remark:   "感谢您的购买！",
}

// 发送消息
resp, err := wxOfficialService.PushTemplateMessage(
    "USER_OPENID",      // 接收者 OpenID
    "TEMPLATE_ID",      // 模板消息 ID
    data,               // 模板数据 (需要与模板匹配)
)

if err != nil {
    log.Printf("发送模板消息失败: %v", err)
} else {
    log.Printf("消息发送成功, MsgID: %s", resp.MsgID)
}
```