<div align="center">
** 奶狗微信框架 一个现代化、插件化的微信开发框架**

基于 Go + Wails3 + React + Ant Design + 千寻微信 DLL 构建

[快速开始](#-快速开始) · [功能特性](#-功能特性) · [插件开发](#-插件开发) · [API 文档](#-api-文档)

</div>

---

## 📖 项目简介

奶狗微信框架是一个功能强大的微信开发框架，提供了完整的微信 API 代理、插件系统和现代化的用户界面。

### ✨ 核心特性

- 🚀 **现代化技术栈**
  - 后端：Go 1.24 + Wails3
  - 前端：React 18 + Ant Design + TailwindCSS
- 🔌 **强大的插件系统**
  - 支持热加载
  - 独立窗口运行
  - 事件实时广播
  - .dog 格式插件包
- 📡 **完整的 API 支持**
  - 47+ 微信 API 接口
  - SSE 实时事件推送
- 🎨 **优雅的用户界面**
  - 响应式设计
  - 主题切换（亮色/暗色/跟随系统）
  - 实时日志查看

---

## 🚀 快速开始

### 环境要求

- **Go**: 1.24.0 或更高版本
- **Node.js**: 18+
- **操作系统**: Windows
- **微信客户端**: 已安装

### 首次运行

1. 启动后会自动打开主窗口
2. 点击"微信管理"登录微信账号（支持多开）
3. 在"插件管理"中安装或开发插件
4. 在"日志面板"查看实时事件

---

## 🎯 功能特性

### 1. 账号管理

- ✅ 多账号登录支持
- ✅ 账号状态实时监控
- ✅ 授权信息自动更新
- ✅ 心跳检测机制

### 2. 插件系统

```
plugins/
└── my-plugin/
    ├── plugin.json      # 插件配置
    ├── icon.png        # 插件图标
    └── frontend/       # 前端资源
        └── index.html  # 入口页面
```

**插件特性**：

- 热加载/热重载
- 独立窗口运行
- 事件广播机制
- 文件上传支持

### 3. 事件系统

支持的事件类型：

- `loginSuccess` - 登录成功
- `recvMsg` - 接收消息
- `friendReq` - 好友请求
- `groupMemberChanges` - 群成员变动
- `transPay` - 转账事件
- `authExpire` - 授权到期

### 4. API 代理

提供 **47+** 微信 API 接口：

| 类别     | 接口数量 | 说明                           |
| -------- | -------- | ------------------------------ |
| 基础接口 | 7        | 版本、登录、二维码等           |
| 消息发送 | 13       | 文本、图片、文件、小程序等     |
| 信息获取 | 5        | 个人信息、好友列表、群聊列表等 |
| 好友管理 | 7        | 添加、删除、备注等             |
| 群聊管理 | 9        | 创建、添加成员、修改昵称等     |
| 其他接口 | 6        | 转账、浏览器、云函数等         |

---

## 🔌 插件开发

### 插件模板

- [万能自定义回复插件模板](https://pan.baidu.com/s/1XFtkhNMffJweCD0h6iBfCQ?pwd=vd61)

### 快速创建插件

#### 1. 创建插件目录结构

```
my-plugin/
├── plugin.json          # 必需：插件元数据
├── icon.png            # 可选：插件图标 (128x128)
└── frontend/           # 必需：前端资源
    └── index.html      # 必需：入口页面
```

#### 2. 编写 plugin.json

```json
{
  "id": "my-plugin",
  "name": "我的插件",
  "version": "1.0.0",
  "author": "你的名字",
  "description": "插件描述",
  "icon": "icon.png",
  "entry": "frontend/index.html",
  "type": "window"
}
```

#### 3. 创建入口页面

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <title>我的插件</title>
  </head>
  <body>
    <h1>我的插件</h1>
    <div id="app"></div>

    <script>
      const API_BASE = "http://localhost:9001/api/plugin";

      // 监听微信事件
      const eventSource = new EventSource(`${API_BASE}/events`);
      eventSource.onmessage = (event) => {
        const data = JSON.parse(event.data);
        console.log("收到事件:", data);

        // 处理不同类型的事件
        if (data.type === "recvMsg") {
          handleMessage(data.data);
        }
      };

      // 处理消息
      function handleMessage(msgData) {
        const { msg, fromWxid } = msgData.data;
        console.log(`收到消息: ${msg} 来自: ${fromWxid}`);

        // 自动回复示例
        if (msg === "你好") {
          sendTextMessage(msgData.port, fromWxid, "你好！我是机器人");
        }
      }

      // 发送文本消息
      async function sendTextMessage(port, wxid, message) {
        const response = await fetch(
          `http://localhost:9001/api/wechat/sendText?port=${port}`,
          {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ wxid, msg: message }),
          }
        );
        return await response.json();
      }

      // 发送日志到主程序
      async function sendLog(message, type = "信息") {
        await fetch(`${API_BASE}/log`, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            pluginId: "my-plugin",
            timeStamp: new Date().toLocaleString("zh-CN"),
            response: "我的插件",
            logType: type,
            msg: message,
            color: "#409EFF",
          }),
        });
      }
    </script>
  </body>
</html>
```

#### 4. 打包插件

```bash
# 将插件文件夹压缩为 .dog 格式（实际上是ZIP）
zip -r my-plugin.dog my-plugin/

# 或使用 PowerShell
Compress-Archive -Path my-plugin -DestinationPath my-plugin.dog
```

#### 5. 安装插件

1. 打开框架主窗口
2. 进入"插件管理"页面
3. 点击"上传插件"按钮
4. 选择 `.dog` 文件
5. 刷新插件列表
6. 点击"运行插件"

---

## 📡 API 文档

### 插件开发 API

#### 1. 获取配置文件

```http
GET /api/plugin/config
```

**响应示例**：

```json
{
  "code": 200,
  "data": "server:\n  address: :9001\n..."
}
```

#### 2. 获取当前微信账号

```http
GET /api/plugin/wechat
```

**响应示例**：

```json
{
  "code": 200,
  "data": {
    "list": [
      {
        "wxid": "wxid_xxx",
        "nick": "昵称",
        "port": 19088,
        "pid": 12345,
        "expireTime": "2024-12-31",
        "isExpire": 0
      }
    ]
  }
}
```

#### 3. 发送日志

```http
POST /api/plugin/log
Content-Type: application/json

{
  "pluginId": "my-plugin",
  "timeStamp": "2024-01-01 12:00:00",
  "response": "我的插件",
  "logType": "信息",
  "msg": "日志内容",
  "color": "#409EFF"
}
```

**日志颜色参考**：

- 成功: `#67C23A`
- 信息: `#409EFF`
- 警告: `#E6A23C`
- 错误: `#F56C6C`

#### 4. 监听事件 (SSE)

```javascript
const eventSource = new EventSource("http://localhost:9001/api/plugin/events");

eventSource.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log("事件类型:", data.type);
  console.log("事件数据:", data.data);
};
```

### 微信 API

所有微信 API 使用统一格式：

```http
POST /api/wechat/{接口名}?port={端口号}
Content-Type: application/json

{
  // 接口参数
}
```

#### 常用接口示例

**发送文本消息**：

```javascript
await fetch("http://localhost:9001/api/wechat/sendText?port=19088", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    wxid: "wxid_xxx",
    msg: "你好",
  }),
});
```

**获取好友列表**：

```javascript
await fetch("http://localhost:9001/api/wechat/getFriendList?port=19088", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    type: "1", // 1=从缓存获取，2=重新遍历
  }),
});
```

**获取群聊列表**：

```javascript
await fetch("http://localhost:9001/api/wechat/getGroupList?port=19088", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    type: "2",
  }),
});
```

---

## ⚙️ 配置说明

### 主配置文件 (configs/config.yaml)

```yaml
server:
  address: :9001 # HTTP服务地址
  callBackUrl: wechat/callback # 回调路径

system:
  appName: 奶狗微信框架 V1.0.0
  resourcePath: resources
  theme: light # 主题: light/dark/system

wechat:
  cachePath: C:\Users\...\NdogCache\
  clearLog: 100 # 日志清理阈值
  decodePict: "1" # 是否解码图片
  groupMemberEvent: "1" # 群成员事件
  hookSilk: "1" # Hook语音
  ignoreMsg: 5 # 忽略消息数
  installationPath: D:\soft\Weixin
  logs: true # 是否记录日志
  resend: "1" # 重发消息
  timeOut: 9000 # 超时时间
  update: "1" # 自动更新
  version:
    - 4.12.17 # 支持的微信版本
```

---

## 🛠️ 开发指南

### 项目结构

```
wechat-framework/
├── cmd/app/              # 应用程序入口
├── internal/             # 内部包
│   ├── core/            # 核心业务逻辑
│   ├── api/             # API层
│   ├── config/          # 配置管理
│   └── server/          # HTTP服务器
├── pkg/                  # 公共包
│   ├── logger/          # 日志服务
│   └── types/           # 类型定义
├── service/              # 旧版服务（兼容）
├── frontend/             # 前端代码
│   ├── src/
│   │   ├── pages/       # 页面组件
│   │   ├── components/  # 公共组件
│   │   └── hooks/       # 自定义Hooks
│   └── bindings/        # Wails绑定
├── configs/              # 配置文件
├── resources/            # 资源文件
└── plugins/              # 插件目录
```

### 开发模式

```bash
# 启动开发模式（热重载）
wails3 dev

# 或使用 task
wails3 task dev
```

### 生产构建

```bash
# 构建前端
cd frontend
npm run build
cd ..

# 构建应用
wails3 build
```

## ❓ 常见问题

### Q: 启动失败？

**A**: 检查以下几点：

1. 端口 9001 是否被占用
2. 微信客户端是否已安装
3. 查看日志文件排查错误

### Q: 插件无法加载？

**A**:

1. 检查 `plugin.json` 格式是否正确
2. 确认插件 ID 唯一
3. 查看控制台错误信息

### Q: 事件接收不到？

**A**:

1. 确认微信已登录
2. 检查回调地址配置
3. 查看网络连接状态

### Q: 配置保存失败？

**A**:

1. 确认 `configs` 目录存在
2. 检查文件权限
3. 查看后端日志

---

## 🤝 贡献指南

欢迎贡献代码、报告问题或提出建议！

---

## ⚠️ 免责声明

本项目仅供学习和研究使用，请勿用于商业用途、违法内容或生产环境。使用本项目所产生的一切后果由使用者自行承担，与作者无关。

---

## 📄 许可证

不需要，随意二开、使用。

---

## 🙏 致谢

- [千寻](https://www.cnblogs.com/daen/p/-/qxpro)

---

## 📮 联系方式

- **QQ 交流群**: 1654819

---

<div align="center">

**如果这个项目对你有帮助，请给一个 ⭐️ Star 支持一下！**

</div>
