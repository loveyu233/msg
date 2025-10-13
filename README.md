# Go 消息发送库

这是一个 Go 语言编写的消息发送库，集成了企业微信机器人和阿里云短信功能，方便快速地发送各种通知和消息。

## 功能特性

- **企业微信机器人**
  - 支持文本、Markdown、图文、文件等多种消息类型。
  - 支持文件上传（语音、普通文件）。
  - 链式调用和便捷方法，易于集成和使用。

- **阿里云短信**
  - 对接阿里云官方的 `dysmsapi` 服务。
  - 提供简单的初始化和发送接口，快速发送模板短信。

## 安装

```bash
go get github.com/loveyu233/msg
```

## 使用方法

### 1. 企业微信机器人

首先，你需要获取企业微信群聊机器人的 Webhook Key。

#### 初始化客户端

在使用前，需要先初始化一个全局客户端实例。推荐在程序启动时进行初始化。

```go
package main

import (
	"log"
	"github.com/loveyu233/msg/gb" // 注意：根据你的项目结构，这里的包名可能是 gb
)

func main() {
	// 你的企业微信机器人 Webhook Key
	webhookKey := "YOUR_WEBHOOK_KEY" 
	gb.InitQWRobotClient(webhookKey)

	// ... 接下来可以调用发送方法
}
```

#### 发送消息示例

```go
// 发送文本消息
_, err := gb.InsQWRobot.SendText("这是一条来自 Go 的测试消息")
if err != nil {
	log.Fatalf("发送文本消息失败: %v", err)
}

// 发送 Markdown 消息
markdownContent := `
# 一级标题
> 引用文本
**加粗**
`
_, err = gb.InsQWRobot.SendMarkdown(markdownContent)
if err != nil {
	log.Fatalf("发送 Markdown 消息失败: %v", err)
}

// 发送文件消息 (先上传，后发送)
// mediaType 可以是 gb.QYWXMediaTypeFile (文件) 或 gb.QYWXMediaTypeVoice (语音)
_, err = gb.InsQWRobot.SendFile("/path/to/your/file.txt", gb.QYWXMediaTypeFile)
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
	"github.com/loveyu233/msg/gb" // 注意：根据你的项目结构，这里的包名可能是 gb
)

func main() {
	accessKeyId := "YOUR_ACCESS_KEY_ID"
	accessKeySecret := "YOUR_ACCESS_KEY_SECRET"

	err := gb.InitShortMsgSimpleClient(accessKeyId, accessKeySecret)
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

err := gb.InsShortMsg.SendSimpleMsg(targetPhoneNumber, signName, templateCode, templateParam)
if err != nil {
	log.Fatalf("发送短信失败: %v", err)
}

log.Println("短信发送成功！")
```
