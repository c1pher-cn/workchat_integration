
# Home Assistant 企微通集成

[](https://github.com/yzg790787394/workchat_integration/releases)
[](https://github.com/yzg790787394/workchat_integration/blob/main/LICENSE)
[](https://github.com/hacs/integration)

企微通集成允许Home Assistant与企业微信（WorkChat）无缝集成，实现双向通信功能。通过此集成，您可以：

1. 📨 从Home Assistant向企业微信发送各种类型的消息（文本、图片、文件、卡片等）
2. 📥 接收企业微信发送的消息（文本、图片、位置等）
3. 📊 在Home Assistant中展示接收的消息内容和回调信息

## 🌟 功能概览

- 📨 **消息发送**：支持多种消息类型发送到企业微信
- 📤 **文件上传**：上传媒体文件到企业微信服务器
- 🔄 **双向通信**：接收企业微信的消息回调并处理
- 📊 **传感器集成**：将接收的消息和状态展示为传感器实体
- 🔐 **安全验证**：支持企业微信的安全验证机制

## 📦 安装

### 通过HACS安装（推荐）

1. 在HACS的"集成"部分，点击右上角的三点菜单
2. 选择"自定义存储库"
3. 在存储库字段输入：`https://github.com/yzg790787394/workchat_integration`
4. 类别选择"集成"
5. 点击"添加"保存
6. 在HACS中找到"企微通"集成并点击安装
7. 重启Home Assistant

### 手动安装

1. 下载最新的[发布版本](https://github.com/yzg790787394/workchat_integration/releases)
2. 解压并将`workchat_integration`文件夹放入Home Assistant的`custom_components`目录
3. 重启Home Assistant

## ⚙️ 配置

### 步骤1：获取企业微信参数

在开始配置前，您需要从企业微信管理后台获取以下信息：

1. **企业ID(Corp ID)**：企业微信的唯一标识
2. **应用Secret**：企业微信应用的密钥
3. **应用Agent ID**：企业微信应用的ID
4. **Token**：用于回调验证的令牌
5. **EncodingAESKey**：用于消息加密的密钥

### 步骤2：在Home Assistant中添加集成

1. 进入Home Assistant的 **设置** > **设备与服务** > **集成**
2. 点击右下角的 **+ 添加集成**
3. 搜索并选择 **"企微通"**
4. 填写以下配置参数：
   - **企业ID**：您的企业ID
   - **应用Secret**：应用的密钥
   - **应用Agent ID**：应用的ID
   - **Token**：回调验证令牌
   - **EncodingAESKey**：加密密钥
   - **接收用户**：默认接收消息的用户（如`@all`或指定用户ID）
   - **外部URL**：Home Assistant的外部访问地址（自动填充）
5. 点击 **提交** 完成配置

### 步骤3：设置企业微信回调

配置完成后，您需要将回调URL设置到企业微信后台：

1. 在集成配置完成后，检查日志中显示的回调URL（格式如：`https://your-ha-domain/api/workchat_callback/your_token`）
2. 登录企业微信管理后台
3. 进入"应用管理" > 选择您的应用 > "接收消息"
4. 点击"设置API接收"
5. 填入以下信息：
   - **URL**：Home Assistant中显示的回调URL
   - **Token**：与配置中相同的Token
   - **EncodingAESKey**：与配置中相同的EncodingAESKey
6. 保存设置并启用

## 🚀 服务使用

### 1. 消息通知服务

通过`workchat_integration.notify`服务发送消息到企业微信。

#### 基本用法

```
yaml
service: workchat_integration.notify
  data:
  msg_type: text
  message: "传感器状态异常！"
```


#### 完整参数

| 参数 | 必填 | 类型 | 默认值 | 描述 |
|------|------|------|--------|------|
| `msg_type` | 是 | string | "text" | 消息类型：text, image, video, file, textcard, news |
| `message` | 是 | string | - | 消息基础内容（文本/描述） |
| `title` | 否 | string | - | 消息标题（用于卡片、视频和图文消息） |
| `media_id` | 部分类型必填 | string | - | 媒体文件ID（需先上传） |
| `url` | 部分类型必填 | string | - | 链接地址（卡片跳转/图文URL） |
| `btntxt` | 否 | string | "详情" | 卡片消息按钮文字 |
| `articles` | 图文消息必填 | list | - | 图文消息的多篇文章（YAML格式） |
| `touser` | 否 | string | 配置值 | 指定接收用户（@all或用户ID） |

#### 消息类型示例

**文本消息**
```
yaml

service: workchat_integration.notify
  data:
  msg_type: text
  message: "客厅温度过高！当前温度: {{ states('sensor.living_room_temperature') }}℃"

```
**图片消息**
```
yaml

service: workchat_integration.notify
  data:
  msg_type: image
  media_id: "1Yv-zXfHjSjU-7LH-GwtYqDGS"
  message: "客厅监控截图"

```
**卡片消息**
```
yaml

service: workchat_integration.notify
  data:
  msg_type: textcard
  title: "安防通知"
  message: "前门检测到有人活动"
  url: ""https://your-ha-domain/lovelace/security" (https://your-ha-domain/lovelace/security)"
  btntxt: "查看详情"

```
**图文消息**
```
yaml

service: workchat_integration.notify
  data:
  msg_type: news
  articles:
  - title: "天气预警"description: "今日将有暴雨，请关好门窗"url: ""https://weather.com" (https://weather.com)"picurl: ""https://example.com/weather.jpg" (https://example.com/weather.jpg)"
  - title: "设备状态"description: "所有设备运行正常"url: ""https://your-ha-domain/lovelace/devices" (https://your-ha-domain/lovelace/devices)"
```
### 2. 媒体上传服务

使用`workchat_integration.upload_media`服务上传文件到企业微信并获取media_id。

#### 基本用法
```
yaml

service: workchat_integration.upload_media
  data:
  file_path: "/config/www/snapshot.jpg"

```
#### 完整参数

| 参数 | 必填 | 类型 | 默认值 | 描述 |
|------|------|------|--------|------|
| `type` | 否 | string | "file" | 媒体类型：file, image, video, voice |
| `file_path` | 是 | string | - | 本地文件完整路径 |
| `file_name` | 否 | string | 自动获取 | 自定义文件名 |

#### 上传示例
```
yaml

service: workchat_integration.upload_media
  data:
  type: image
  file_path: "/config/www/living_room_snapshot.jpg"
  file_name: "living_room.jpg"

```
#### 响应示例

服务调用成功后会返回media_id：
```
json

{

"media_id": "2hY-zXfHjSjU-8LH-GwtYqDHT"

}

```
## 🔍 传感器

集成添加后会自动创建以下传感器实体：

### 1. 企微通文本消息传感器
- **状态**：最新收到的文本内容
- **属性**：
  - 用户：发送消息的用户ID
  - 时间：消息时间戳
  - 内容：完整消息内容

### 2. 企微通图片消息传感器
- **状态**：媒体文件ID
- **属性**：
  - 用户：发送消息的用户ID
  - 时间：消息时间戳
  - 图片URL：图片访问地址
  - 媒体ID：媒体文件ID

### 3. 企微通位置消息传感器
- **状态**：位置标签
- **属性**：
  - 用户：发送消息的用户ID
  - 时间：消息时间戳
  - 纬度：位置纬度
  - 经度：位置经度
  - 缩放级别：地图缩放比例
  - 位置标签：位置描述

### 4. 企微通回调URL信息传感器
- **状态**：配置状态
- **属性**：
  - 回调URL：用于企业微信的回调地址
  - Token：验证令牌
  - EncodingAESKey：加密密钥

### 5. 企微通上传媒体文件信息传感器
- **状态**：上传状态
- **属性**：
  - 文件名：上传的文件名
  - 文件类型：媒体类型
  - 上传时间：上传完成时间
  - 文件路径：本地文件路径
  - media_id：上传后获得的媒体ID

## ⚡ 高级功能

### 自动化示例

**当温度过高时发送通知**
```
yaml

alias: 高温报警通知

trigger:

  - platform: numeric_stateentity_id: sensor.living_room_temperatureabove: 30action:
  - service: workchat_integration.notifydata:msg_type: textcardtitle: "高温警告"message: >客厅温度过高！当前温度: {{ states('sensor.living_room_temperature') }}℃建议打开空调url: ""https://your-ha-domain/lovelace/climate" (https://your-ha-domain/lovelace/climate)"btntxt: "调整空调"
```
**接收位置消息时更新设备追踪器**
```
yaml

alias: 更新位置追踪
trigger:
  - platform: eventevent_type: workchat_messageevent_data:type: locationaction:
  - service: device_tracker.seedata:dev_id: mobile_app_workchatlocation_name: "{{ trigger.event.data.label }}"gps:
   - "{{ trigger.event.data.lat }}"
   - "{{ trigger.event.data.lon }}"
```
## 🛠 故障排除

### 常见问题

1. **回调验证失败**
- 

检查企业微信后台的回调URL、Token和EncodingAESKey是否与Home Assistant配置一致

- 确保Home Assistant的外部URL可公开访问
- 检查系统时间是否准确（时间偏差会导致签名错误）

2. 消息发送失败
   - 检查企业微信应用的权限设置
   - 验证接收用户ID是否正确
   - 确认媒体文件已正确上传且media_id有效
   - 检查日志获取详细的错误信息
3. 文件上传失败
   - 确保文件路径正确且有访问权限
   - 检查文件大小（企业微信限制为10MB）
   - 确认网络连接正常

日志分析

在Home Assistant的
"configuration.yaml"中增加日志级别设置：
```
logger:
  default: info
  logs:
    custom_components.workchat_integration: debug
```
重启后查看日志获取详细调试信息。

🤝 贡献

欢迎贡献代码、报告问题或提出功能建议！

1. 提交Issues：报告问题或功能请求
2. 提交Pull Requests：贡献代码改进
3. 项目讨论：分享使用经验或建议

❤️ 支持
