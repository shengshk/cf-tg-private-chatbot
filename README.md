# cf-tg-private-chatbot

Cloudflare Worker：陌生人先跟 Bot 私聊，管理员在开启 Topics 的超级群里按话题接待。本地人机验证，消息双向转发。

上游：[jikssha/telegram_private_chatbot](https://github.com/jikssha/telegram_private_chatbot) · [说明（中文 README）](https://github.com/jikssha/telegram_private_chatbot/blob/main/README.md) · [English](https://github.com/jikssha/telegram_private_chatbot/blob/main/README_EN.md)

指令、验证题、话题行为等以上游为准，本仓不重复。

## 本仓差异

- **状态页**：打开 Worker 根地址（`GET /`）即 `setWebhook` 并注册命令菜单，不必再手拼 `api.telegram.org/bot.../setWebhook`。绿/黄/红表示已绑定 / 可切换域名 / 未绑定；点状态文字可再初始化。`/init`、`/api/status` 仍可用。
- **群菜单命令**：群里发出的 `/info@botname` 能解析（上游只认 `/info`）。斜杠命令不转发给用户。
- **部署配置**：`wrangler.jsonc` 写 KV 绑定名 **`TOPIC_MAP`**（Git 部署认这份）；`keep_vars = true` 保住 Dashboard 里的 `BOT_TOKEN` / `SUPERGROUP_ID`。`status.html` 与 `worker.js` 一起打包，不要只在面板里粘贴 `worker.js`。

## 部署

1. BotFather 关 **Group Privacy**。超级群开 Topics，Bot 设为管理员并给 **Manage Topics**。`SUPERGROUP_ID` 必须是 `-100` 开头。
2. Cloudflare 建 KV，把 Namespace ID 填进 `wrangler.jsonc` 的 `kv_namespaces`（绑定名 `TOPIC_MAP`）。换账号复制 `wrangler.example.jsonc`。
3. Dashboard 机密：`BOT_TOKEN`、`SUPERGROUP_ID`。
4. 部署二选一：Cloudflare Builds 连接本 GitHub 仓；或 `npx wrangler deploy --keep-vars`。
5. 打开 `https://<worker-name>.<account>.workers.dev/` 完成 webhook。

Git 部署后若出现 `KV 'TOPIC_MAP' not bound`：ID 必须写在 `wrangler.jsonc`，不要只在 Dashboard 手绑。

## Directory

```text
<project>/
├── README.md
├── LICENSE
├── .env.example
├── wrangler.jsonc
├── wrangler.example.jsonc
├── worker.js
└── status.html
```

## License

[MIT](LICENSE)
