# cf-tg-private-chatbot

<p align="center">
  Cloudflare Worker · Telegram 双向私聊机器人（人机验证 + Forum Topics）<br/>
  Based on / 基于 <a href="https://github.com/jikssha/telegram_private_chatbot">jikssha/telegram_private_chatbot</a>
</p>

<p align="center">
  <a href="README_EN.md">English</a> · <a href="#中文">中文</a>
  · <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="MIT" /></a>
</p>

## Directory

```text
<project>/
├── README.md                  # this file on GitHub (from README.PROJECT.md)
├── README_EN.md
├── LICENSE
├── .env.example
├── wrangler.jsonc
├── wrangler.example.jsonc
├── worker.js
└── status.html
```

---

# 中文

**Telegram Private Chatbot** 是一个基于 **Cloudflare Workers** 的高性能 Telegram 双向私聊机器人。它专为解决 Telegram 上的垃圾广告骚扰而生，拥有 0 延迟的本地人机验证系统、强大的管理员指令集以及无缝的消息转发体验。

无需购买服务器，利用 Cloudflare 强大的边缘计算网络，即可免费部署一套企业级的客户服务系统。

---
    
<details>
<summary>📢 <b>v5.1 版本重要更新公告 (2026-01-05)</b></summary>
   
### 主要修复：
- **自动话题修复**：被删除话题用户不再转发到 General，会自动新建话题。
- **话题无限循环修复**：针对创建失败添加重试机制，最多重试 3 次。
- **消息路由规范化**：修复字符串与数字混用问题，统一规范化为 String 类型。
- **并发验证加固**：添加验证锁机制，彻底杜绝并发绕过漏洞。
- **数据读取保护**：实现 `safeGetJSON()` 安全读取机制，防止 KV 数据损坏导致崩溃。
- **验证系统重构**：改用索引方案，完全避免按钮回调截断问题，100% 可用。
   
### 更新功能：
**批量清理工具**：/cleanup  # 扫描并清理已删除话题的用户数据

### ⚠️ 更新指南：
Fork用户可直接点击sync 更新同步，自动更新
手动部署用户复制worker.js代码到worker,重新部署一次
</details>

---

## 📑 目录 (Table of Contents)

* [✨ 核心特性](#-核心特性)
* [🛠️ 管理员指令](#-管理员指令)
* [🚀 部署教程](#-部署教程)
    * [方法一：GitHub 一键连接 (推荐)](#方法一github-一键连接部署-推荐-)
    * [方法二：命令行部署](#方法二命令行部署)
    * [最后一步：打开状态页](#最后一步打开状态页自动激活-webhook)
* [❓ 常见问题 (FAQ)](#-常见问题-faq)
* [📈 Star History](#-star-history)

---

## ✨ 核心特性

v4.0 版本移除了所有不稳定的外部 API 依赖，专注于**极致的速度**与**绝对的稳定性**。

| 特性 | 描述 |
| :--- | :--- |
| **⚡ 0 延迟验证** | 采用**本地精选常识题库**。秒开秒验，彻底告别网络超时与接口报错，验证成功率 100%。 |
| **🛡️ 智能防骚扰** | **短 ID 机制**修复了 Telegram 按钮点击失效的 Bug。验证通过后提供 **30 天免打扰期**，兼顾安全与用户体验。 |
| **💬 话题群组管理** | 利用 **Telegram Forum Topics** 功能，自动为每位私聊用户创建一个独立的话题，消息隔离，管理井井有条。 |
| **👮 隐形指令系统** | 自动**拦截**用户端发送的 `/` 开头指令，防止普通用户骚扰管理员。管理指令仅在管理员群组内生效。 |
| **🔒 权限控制** | 强大的指令集：支持 **封禁 (/ban)**、**解封 (/unban)**、**结单 (/close)** 和 **永久信任 (/trust)** 等操作。 |
| **☁️ Serverless** | 完全基于 Cloudflare Workers 运行。**0 成本**、无需服务器、无需运维、抗高并发。 |
| **📸 多媒体支持** | 完美支持文本、图片、视频、文件等多种消息格式的双向转发，不丢失任何细节。 |

---

## 🛠️ 管理员指令

> **注意**：以下指令仅在 **管理员群组的话题内** 有效。用户在私聊窗口发送指令会被静默拦截，不会对管理员造成骚扰。

| 指令 | 作用 | 适用场景 |
| :--- | :--- | :--- |
| `/close` | **强制关闭对话**<br>机器人会提示用户对话已结束，并拒收新消息。 | 工单处理完成，礼貌结束咨询。 |
| `/open` | **重新开启对话**<br>恢复对该用户的消息转发。 | 误操作关闭，或用户需再次联系。 |
| `/ban` | **封禁用户**<br>机器人将完全无视该用户的所有消息（无提示）。 | 遇到恶意刷屏、广告机器人。 |
| `/unban` | **解封用户**<br>恢复该用户的正常通讯权限。 | 给予改过自新的机会。 |
| `/trust` | **永久信任**<br>该用户将永久免除人机验证（永不过期）。 | 熟人、VIP 客户、长期合作伙伴。 |
| `/reset` | **重置验证**<br>强制清除该用户的验证状态，下次需重新验证。 | 测试验证流程，或怀疑账号被盗。 |
| `/info` | **查看信息**<br>显示当前用户的 UID、话题 ID 和链接。 | 查询用户资料。 |
| `/cleanup` | **批量清理**<br>扫描并清理已删除话题的用户数据。 | 清理失效用户。 |

---

## 🚀 部署教程

### 前置准备
1.  **Telegram Bot**：找 [@BotFather](https://t.me/BotFather) 申请一个机器人，获取 `Token`。
    * *重要设置*：在 BotFather 中关闭 **Group Privacy** (`/mybots` > Settings > Group Privacy > Turn off)。
2.  **管理员群组**：创建一个 Telegram 群组，并**开启话题功能 (Topics)**。
    * 将机器人拉入群组，并设为**管理员**（给予管理话题权限）。
    * 获取群组 ID（通常以 `-100` 开头）。
     ``获取 SUPERGROUP_ID 小技巧：
在 Telegram 桌面端右键群内任意消息，复制消息链接；链接里会有一段 -100xxxxxxxxxx 或 xxxxxxxxxx；若只看到纯数字 xxxxxxxxxx，在前面加上 -100，就是完整的 SUPERGROUP_ID（私密频道/群组同理）。``

KV 绑定写在仓库根目录的 **`wrangler.jsonc`**（与 `wrangler.toml` 内容相同）。Git 部署认这份文件；绑定名必须是 **`TOPIC_MAP`**。换账号部署时复制 `wrangler.example.jsonc`，把 `id` 改成新账号的 KV Namespace ID。`BOT_TOKEN` / `SUPERGROUP_ID` 放 Dashboard 机密，靠 `keep_vars` 部署时不清掉。

### 方法一：GitHub 一键连接部署 (推荐 ★)

这是最简单的自动化部署方式，当您更新 GitHub 仓库时，Cloudflare 会自动重新部署您的 Worker。

1.  **Fork 本仓库** 到您的 GitHub 账户。
2.  在 Cloudflare 建一个 KV 命名空间，把 Namespace ID 填进 `wrangler.jsonc` 的 `kv_namespaces`（绑定名 `TOPIC_MAP`）。
3.  登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)。
4.  导航到 **Workers & Pages** -> **Create Application**。
5.  点击 **Connect to Git** 标签页。
6.  授权 Cloudflare 访问您的 GitHub，并选择本仓库。
7.  **配置部署设置**：
    * 项目名称：`tg-private-chatbot`（与 `wrangler.jsonc` 的 `name` 一致）。
    * 生产分支：通常是 `main`。
    * 其余保持默认，点击 **Save and Deploy**。
8.  在 Worker **设置 → 变量和机密** 中添加：
    * `BOT_TOKEN`: 机器人 Token（建议用机密）。
    * `SUPERGROUP_ID`: 群组 ID（`-100` 开头）。
    KV 不必再在 Dashboard 里手绑；已写在 `wrangler.jsonc` 里。

### 方法二：命令行部署

需要把 `worker.js` 和 `status.html` 一起打包，不要只在 Dashboard 里粘贴 `worker.js`。

```bash
# 填 wrangler.jsonc 的 KV Namespace ID，Dashboard 里配置 BOT_TOKEN / SUPERGROUP_ID
npx wrangler deploy --keep-vars
```

`keep_vars = true`，部署不会清掉面板里的明文变量。Token 建议用机密。

---

### 最后一步：打开状态页（自动激活 Webhook）

部署完成后打开 Worker 根地址（例如 `https://<worker-name>.<account>.workers.dev/`）。

打开即登记 Telegram webhook 和命令菜单，**不必**再手动访问 `api.telegram.org/bot.../setWebhook`。

页面为绿/黄/红：已绑定 / 可切换域名 / 未绑定。点状态文字可重新初始化。

---

## ❓ 常见问题 (FAQ)

**Q1: 为什么点击验证按钮没有反应？**
A: 打开 Worker 根地址状态页，点状态文字重新初始化。Webhook 必须允许 `callback_query`（状态页 `/init` 已带上）。

**Q2: 为什么机器人无法在群里创建话题？**
A: 请确保：1. 群组 ID 正确（-100开头）；2. 群组已开启 Topics 功能；3. 机器人是群管理员且拥有 "Manage Topics" 权限。

**Q3: 为什么人机验证能通过收不到转发的消息？**
A: 请仔细检查所有变量名称和 id 是否准确。然后打开 Worker 根地址状态页，点状态文字重新绑定 webhook。 
  
  如果依然无法正常转发消息，尝试完成所有步骤后，最后再添加bot的管理员权限。
  
**Q5: 为什么重新部署后 Worker 变成 `Error: KV 'TOPIC_MAP' not bound`？**
A: Git 部署以 `wrangler.jsonc` 为准。把 KV Namespace ID 写进 `kv_namespaces`（绑定名必须是 `TOPIC_MAP`），不要只在 Dashboard 里手绑。

---

## 🔒 安全说明

> [!IMPORTANT]
> 请妥善保管您的 Bot API Token ，不要泄露，这些信息关系到您服务的安全性。

---

## 📈 Star History

[![Star History Chart](https://api.star-history.com/svg?repos=shengshk/cf-tg-private-chatbot&type=date&legend=top-left)](https://www.star-history.com/#shengshk/cf-tg-private-chatbot&type=date&legend=top-left)

---
**如果这个项目对你有帮助，请给个 Star ⭐️ 吧！**

## License

[MIT](LICENSE)
