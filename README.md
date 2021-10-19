# Telegram Archive Server

[![Docker](https://github.com/oott123/telegram-archive-server/actions/workflows/ci.yaml/badge.svg)](https://github.com/oott123/telegram-archive-server/actions/workflows/ci.yaml) [![CJK Ready](https://img.shields.io/badge/CJK-ready-66ccff)](./README.md) [![Releases](https://img.shields.io/github/package-json/v/oott123/telegram-archive-server/master?label=version)](https://github.com/oott123/telegram-archive-server/releases) [![quay.io](https://img.shields.io/badge/Browse%20on-quay.io-blue?logo=docker&logoColor=white)](https://quay.io/repository/oott123/telegram-archive-server?tab=tags) [![BSD 3 Clause Licensed](https://img.shields.io/github/license/oott123/telegram-archive-server)](./LICENSE)

一个适合 CJK 环境的，Telegram 群聊搜索和归档机器人。

## 功能概览

- 支持群成员鉴权，仅群友可以搜索
- 支持导入历史聊天记录，自动去重
- 使用 MeiliSearch 对中文进行搜索，索引效果好
- 有简单的网页界面，可以显示头像
- 搜索结果可以跳转打开聊天界面

## 展示

### 聊天鉴权

![](./docs/assets/search-command.jpg)

点击【搜索】按钮即可自动鉴权打开搜索界面。

### 搜索界面

![](./docs/assets/search-ui.jpg)

点击时间链接即可跳转聊天界面。

![](./docs/assets/search-and-jump.gif)

## 部署

### 准备

你需要：

- 一个 Bot 帐号，事先获取它的 token
- 一个公网可及的 https 服务器（当然 http 的也凑合用，只是不能开 webhook 了）
- 一个**超级群**，目前只支持超级群
- 一个 MeiliSearch 实例，配不配置 key 都行
- 一个 Redis 实例，没有也行
- 一个 Docker 环境

### 配置

下载 [`.env.example`](./.env.example) 文件，参考内部注释，进行相应配置。

你可以将它保存为 `.env` ，或是作为环境变量配置。

### 运行

```bash
docker run -d --restart=always --env-file=.env quay.io/oott123/telegram-archive-server
```

当然，也可以使用 Kubernetes 或者 docker-compose 运行。

### 使用

在群里发送 `/search`。Bot 可能会提示你设置 Domain，按提示设置即可。

![](./docs/assets/bot-set-domain.gif)

### 导入老的聊天记录

**当前仅支持超级群导入。**

在桌面客户端点击三点按钮 - Export chat history，等待导出完成，得到 `result.json`。

执行：

```bash
curl \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $AUTH_IMPORT_TOKEN" \
  --data @result.json \
  http://localhost:3100/api/v1/import/fromTelegramGroupExport
```

即可导入记录。注意只能导入单个群的记录。

## 开发

```bash
DEBUG=app:*,grammy* yarn start:debug
```

### 前端开发

搜索服务鉴权后，服务端会跳转到：`$HTTP_UI_URL/index.html` 并带上以下 URL 参数：

- `tas_server` - 服务器基础 URL，形如 `http://localhost:3100/api/v1`
- `tas_indexName` - 群号，形如 `supergroup1234567890`
- `tas_authKey` - 服务器签发的 JWT，可以用来作为 MeiliSearch 的 api key 使用。

### MeiliSearch 兼容

在 `/api/v1/search/compilable/meili` 处可以当作普通的 MeiliSearch 实例进行搜索。

索引名应该使用形如 `supergroup1234567890` 的群号； API Key 则是服务端签发的 JWT。

请注意 filter 由于安全原因暂时不可使用。
