---
title: OpenClaw 搭建指南：从零开始部署你的私人 AI 助手
date: 2026-02-13 10:00:00
tags:
  - AI
  - OpenClaw
  - 教程
  - 自部署
categories:
  - 技术
---

## 前言

最近 **OpenClaw** 火得一塌糊涂，GitHub Star 数一路飙升到 189k+，成为当下最受关注的开源项目之一。

简单来说，OpenClaw 是一个运行在你自己设备上的**私人 AI 助手**。它不只是能聊天——它能真正动手干活：读写文件、执行命令、控制浏览器、管理日历邮件，7×24 小时不休息。你可以通过 WhatsApp、Telegram、飞书、钉钉、Discord 等聊天工具随时指挥它。

本文将手把手带你从零搭建 OpenClaw，涵盖云服务器部署和本地部署两种方式。

---

## OpenClaw 是什么

OpenClaw（原名 Clawdbot → Moltbot → OpenClaw）由 PSPDFKit 创始人 Peter Steinberger 发布，是一个开源的本地优先 AI 智能体框架。

### 核心特性

- **本地优先**：所有数据在本地处理，隐私完全可控
- **系统级执行能力**：能读写文件、执行 Shell 命令、控制浏览器、调用 API
- **多平台接入**：支持 WhatsApp、Telegram、Slack、Discord、飞书、钉钉、企业微信、iMessage 等
- **持久记忆**：记住你的偏好和上下文，跨会话连贯工作
- **7×24h 在线**：支持定时任务、主动提醒、后台监控

### 传统 AI vs OpenClaw

| 场景 | 传统 AI | OpenClaw |
|------|---------|----------|
| 收件箱满了 | "建议你清理一下邮件" | 直接打开邮箱，分类删除垃圾邮件 |
| 想订机票 | "推荐你上携程看看" | 打开浏览器，搜索航班，完成预订 |
| 代码部署 | "你可以运行 git push" | 自动执行 git 提交、推送、创建 PR |
| 数据分析 | "建议用 Python 处理" | 编写并执行脚本，输出分析结果 |

---

## 部署方案选择

### 三种方案对比

| 对比维度 | 本地部署 | Mac Mini | 云服务器 |
|----------|---------|----------|---------|
| 成本 | 免费 | 中等（一次性） | 较低（按年） |
| 在线时长 | 随电脑开关 | 7×24h | 7×24h |
| 公网访问 | 需内网穿透 | 需内网穿透 | ✅ 独立公网 IP |
| 稳定性 | 一般 | 极高 | 高 |
| 推荐度 | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

**快速选择**：
- 想先试试 → 本地部署
- 有闲置 Mac Mini → Mac Mini 方案
- 需要稳定在线 → **云服务器（推荐）**

### 云厂商方案对比

| 厂商 | 价格 | 配置 | 特点 |
|------|------|------|------|
| 阿里云 | 68 元/年 | 2核2G/200M/40G | 性价比最高，支持海外 |
| 腾讯云 | 99 元/年 | 2核2G/50M/50G | IM 接入最全 |
| 百度云 | 0.01 元/首月 | 2核4G/5M/200G | 零成本尝鲜 |
| AWS | 半年免费+100$ | 2核2G | 可定制性强，海外原生 |

---

## 方式一：云服务器从零部署（推荐）

以一台 Linux 云服务器为例，完整演示部署过程。

### 1. 连接服务器

```bash
# 修改密钥权限
chmod 600 ~/Downloads/your-key.pem

# SSH 连接
ssh -i ~/Downloads/your-key.pem root@你的服务器公网IP
```

建议配置 SSH 快捷方式：

```bash
# 编辑 ~/.ssh/config
Host openclaw
    HostName <服务器公网IP>
    User root
    IdentityFile ~/Downloads/your-key.pem
    StrictHostKeyChecking no

# 之后直接用
ssh openclaw
```

### 2. 配置 Swap 内存（2G 内存必做）

2GB 内存直接装 OpenClaw 可能 OOM，建议配置 4G Swap：

```bash
# 创建 Swap
fallocate -l 4G ~/swapfile
chmod 600 ~/swapfile
sudo mkswap ~/swapfile

# 永久挂载
echo "$HOME/swapfile none swap sw 0 0" | sudo tee -a /etc/fstab

# 激活
sudo swapon --all

# 验证
free -h
```

### 3. 安装 Node.js 22+

OpenClaw 要求 Node.js 22 以上：

```bash
# 安装 nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.4/install.sh | bash
source ~/.bashrc

# 安装 Node.js
nvm install node

# 验证
node --version  # 应该是 v22.x.x
```

### 4. 安装 OpenClaw

**方式 A：一键脚本（推荐）**

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

**方式 B：npm 安装**

```bash
npm install -g openclaw@latest
```

### 5. 初始化配置

```bash
openclaw onboard --install-daemon
```

按提示操作：

1. **安全确认**：选择 `Yes`
2. **Onboarding mode**：选择 `QuickStart`
3. **Model provider**：选择你的模型提供商（推荐 Anthropic 或 Qwen）
4. **Channel**：可以先 `Skip for now`，后续再配置
5. **Skills**：可以先跳过
6. **启动方式**：选择 `Hatch in TUI`

看到 TUI 聊天界面就说明安装成功了，输入 `Hello` 测试一下。

### 6. 检查服务状态

```bash
openclaw status
```

看到 `Gateway service: running` 就说明一切正常。

---

## 方式二：本地部署（macOS/Linux）

如果你只是想体验一下，本地部署最简单：

```bash
# 1. 确保 Node.js >= 22
node --version

# 2. 安装 OpenClaw
npm install -g openclaw@latest

# 3. 初始化
openclaw onboard --install-daemon

# 4. 启动
openclaw gateway start
```

macOS 用户还可以使用菜单栏 App，体验更好。

---

## 接入聊天平台

OpenClaw 支持接入多种聊天平台，这里以 Telegram 和飞书为例。

### 接入 Telegram

1. 在 Telegram 找 `@BotFather`，创建一个新 Bot，获取 Bot Token
2. 找 `@getidsbot` 获取你的 Telegram 数字 ID
3. 运行配置：

```bash
openclaw configure
```

选择 Telegram，填入 Bot Token 和 Owner ID 即可。

### 接入飞书

1. 在[飞书开放平台](https://open.feishu.cn/)创建应用，启用"机器人"能力
2. 开通 `im:message` 等消息权限
3. 事件订阅选择**长连接**模式（无需公网域名）
4. 获取 App ID 和 App Secret
5. 在 OpenClaw 中配置：

```bash
openclaw configure
# 选择 Feishu/Lark，填入 App ID 和 App Secret
```

配置完成后，就可以在飞书里和你的 AI 助手对话了。

---

## 安装技能包（Skills）

Skills 是 OpenClaw 的扩展能力，相当于给 AI 装插件。

### 通过命令安装

```bash
npx clawhub@latest install <skill名称>
```

### 通过 ClawHub 浏览

访问 [clawhub.ai](https://clawhub.ai/skills) 浏览可用技能，包括：
- 百度搜索
- AI 绘本生成
- 智能 PPT 生成
- 编程动画制作
- 更多社区技能...

### 让 AI 自己装

最酷的方式——直接跟 OpenClaw 说：

> "帮我安装百度搜索技能"

它会自己操作服务器完成安装。让 AI 给自己装技能，这才是 AI 时代该有的操作。

---

## 实用玩法

### 1. AI 追热点

> "帮我获取今天 AI 相关的资讯热点"

还可以设置定时任务，每天早上自动推送。

### 2. 灵感记录器

把 OpenClaw 当超级备忘录，随时在手机上说一句就行。它不只帮你记，还会自动分类整理。

### 3. 随身程序员

在手机上发条消息：

> "帮我写一个图片批量压缩的网页工具，写完直接部署到服务器上"

几分钟后，一个能用的在线工具就出来了。

### 4. 自动化工作流

- 定时检查服务器状态
- 监控邮件并自动分类
- 自动回复群消息
- 定时生成日报/周报

---

## 常见问题

### 安装时内存不足（OOM）

配置更大的 Swap，或升级到 4G 内存的服务器。

### npm install 失败

国内服务器可能需要配置镜像源：

```bash
npm config set registry https://registry.npmmirror.com
```

### 无法访问 Web UI

确保防火墙放通了 18789 端口：

```bash
# 云服务器安全组放通 18789
# 或者用 SSH 隧道
ssh -N -L 18789:127.0.0.1:18789 openclaw
```

然后本地访问 `http://localhost:18789`。

### 模型连接失败

运行诊断：

```bash
openclaw doctor
```

检查 API Key 是否正确，网络是否通畅。

---

## 总结

OpenClaw 把 AI 助手的门槛拉到了地板上。几分钟搭建，一条消息指挥，7×24 小时在线。

不管你是开发者、学生还是普通上班族，都值得试试。毕竟，谁不想有个不休息、不抱怨、随叫随到的 AI 打工人呢？

**相关链接**：
- GitHub：[github.com/openclaw/openclaw](https://github.com/openclaw/openclaw)
- 官网：[openclaw.ai](https://openclaw.ai/)
- 文档：[docs.openclaw.ai](https://docs.openclaw.ai/)
- 技能商店：[clawhub.ai](https://clawhub.ai/)
- 官方 Discord：[discord.gg/clawd](https://discord.gg/clawd)
