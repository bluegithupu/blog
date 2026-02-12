---
title: "OpenClaw 命令行（CLI）使用说明：常用命令速查"
date: 2026-02-12T08:34:51+0800
draft: false
tags: ["openclaw", "cli", "工具"]
---

这篇文章整理一份 **OpenClaw CLI** 的常用命令速查，覆盖：查看状态、启动/管理 Gateway、发消息、定时任务（cron）、配置管理与排错。

> 文档入口：<https://docs.openclaw.ai/cli>

## 1. 核心概念：CLI vs Gateway

- **openclaw（CLI）**：你在终端里敲的命令。
- **Gateway**：OpenClaw 的后台服务（WebSocket），负责承接消息、跑 agent、调度 cron、管理插件等。

多数“自动化能力”（消息转发、定时、浏览器/节点等）都依赖 Gateway 正常运行。

## 2. 全局选项（很常用）

```bash
openclaw -V
openclaw --help
```

- `--profile <name>`：使用命名 profile（把配置/状态隔离到 `~/.openclaw-<name>`）
- `--dev`：开发 profile（同样是隔离环境，默认端口会变化）
- `--no-color`：关闭彩色输出（适合日志/CI）

## 3. 一句话检查：我现在是不是正常？

```bash
openclaw status
openclaw status --all
openclaw status --deep
openclaw status --json
```

- `status`：快速看渠道健康 + session 概况
- `--all`：更完整但仍然是只读诊断（适合复制粘贴排查）
- `--deep`：对各 channel 做更深入的探测

## 4. Gateway 管理（最关键的一组命令）

```bash
openclaw gateway status
openclaw gateway start
openclaw gateway stop
openclaw gateway restart
```

前台运行（调试用）：

```bash
openclaw gateway run
```

常用参数（按需）：

```bash
openclaw gateway run --port 18789
openclaw gateway run --force
openclaw gateway run --verbose
openclaw --dev gateway run
```

> `--force` 会尝试杀掉占用目标端口的进程；线上环境建议谨慎用。

## 5. 发送消息（把 OpenClaw 当“消息命令行”用）

```bash
openclaw message send --channel feishu --target <dest> --message "Hi"
openclaw message send --channel telegram --target @mychat --message "Hi"
```

查看 `--help` 能看到支持的 channel 列表以及可选参数（媒体附件、reply-to、thread-id 等）：

```bash
openclaw message send --help
```

## 6. Cron：定时任务 / 提醒

```bash
openclaw cron status
openclaw cron list
openclaw cron add
openclaw cron edit
openclaw cron disable
openclaw cron enable
openclaw cron run
openclaw cron runs
openclaw cron rm
```

快速理解：
- `cron list` 看有哪些任务
- `cron runs` 看历史执行记录
- `cron run` 手动触发一次用于调试

> 具体 `add/edit` 的参数会跟你的 payload 类型、channel、sessionTarget 等有关；如果你只想做“提醒”，通常让 agent/系统事件发一条消息即可。

## 7. Config：读取/修改配置（小心但很好用）

```bash
openclaw config get <dot.path>
openclaw config set <dot.path> <value>
openclaw config unset <dot.path>
```

例子（示意）：

```bash
openclaw config get gateway.bind
openclaw config get gateway.port
```

> 修改配置可能需要重启 gateway 才生效：`openclaw gateway restart`。

## 8. 常用“辅助类”命令（按需）

下面这些命令在日常排障/运维很顺手：

- 健康检查与快速修复：
  ```bash
  openclaw doctor --help
  openclaw health --help
  ```
- 日志：
  ```bash
  openclaw logs --help
  ```
- 会话与历史：
  ```bash
  openclaw sessions --help
  ```
- 设备/节点（手机、树莓派等）：
  ```bash
  openclaw devices --help
  openclaw nodes --help
  openclaw node --help
  ```
- 技能与插件：
  ```bash
  openclaw skills --help
  openclaw plugins --help
  ```

## 9. 一个推荐的“排错顺序”

1) 先看总体：

```bash
openclaw status --all
```

2) Gateway 起来了吗：

```bash
openclaw gateway status
```

3) 不行就重启（成本最低）：

```bash
openclaw gateway restart
```

4) 仍有问题，再开更详细日志：

```bash
openclaw status --deep
openclaw gateway run --verbose
```

## 10. 结语

OpenClaw 的 CLI 设计思路很像“运维工具箱”：先把 Gateway 跑稳，再用 `message / cron / sessions / skills` 把重复劳动变成自动化。

如果你愿意，我也可以在下一篇把“从 0 配好一个飞书 DM 机器人 + 起床提醒 + 博客发布流水线”的整套流程写成一篇可复制粘贴的教程。