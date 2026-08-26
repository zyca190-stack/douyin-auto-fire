# GitHub Actions 使用教程

本教程介绍如何使用 **GitHub Actions** 运行 `douyin-auto-fire`。

使用这种方式不需要自己准备服务器，也不需要电脑每天开机。配置完成后，GitHub Actions 会按照设定时间自动运行任务。

> 建议第一次只配置 **1 个抖音账号 + 1 个好友 + 1 条文字消息**。确认正常运行后，再添加其他好友、原生表情、随机消息或多账号。

---

## 1. Fork 项目

打开项目仓库：

**https://github.com/unmev/douyin-auto-fire**

点击右上角 **Fork**，将项目 Fork 到自己的 GitHub 账号。

![Fork 项目](https://img.908988.xyz/file/教程/douyin-auto-fire/DKPd0GVi.webp)

Fork 完成后，后面的所有操作都在 **你自己 Fork 出来的仓库** 中进行。

---

## 2. 启用 GitHub Actions

进入自己 Fork 后的仓库，点击顶部的 **Actions**。

如果 GitHub 提示 Fork 仓库的 Workflow 被禁用，点击启用工作流。

启用以后应该可以看到：

```text
Send Douyin Messages
```

这就是项目每天自动运行使用的工作流。

---

## 3. 获取抖音 Cookie

程序需要 Cookie 才能保持抖音登录状态。

### 3.1 登录抖音网页版

使用电脑浏览器打开：

**https://www.douyin.com/**

登录自己的抖音账号，并确认能够正常进入私信页面。

### 3.2 安装 Cookie-Editor

推荐使用浏览器扩展 **Cookie-Editor**：

**https://chromewebstore.google.com/detail/hlkenndednhfkekhgcdicdfddnkalmdm**

安装完成后，回到已经登录抖音的页面并打开 Cookie-Editor。

![打开 Cookie-Editor](https://img.908988.xyz/file/教程/douyin-auto-fire/STZqIxDn.webp)

### 3.3 导出 Cookie

点击 Cookie-Editor 的导出功能，导出格式选择 **JSON**。

![导出 Cookie](https://img.908988.xyz/file/教程/douyin-auto-fire/1rilVYmK.webp)

然后复制完整的 JSON 内容。

![复制 Cookie JSON](https://img.908988.xyz/file/教程/douyin-auto-fire/QKQHfndn.webp)

正确格式大致如下：

```json
[
  {
    "name": "xxx",
    "value": "xxx",
    "domain": ".douyin.com",
    "path": "/"
  }
]
```

请注意：

- 必须复制完整的 `[ ... ]` JSON 数组。
- 不要使用 `name=value; name=value;` 形式。
- 不要删除 Cookie 中的字段。
- 不要把 Cookie 提交到 GitHub 仓库。

> ⚠️ Cookie 相当于账号登录凭证，请不要发送给其他人，也不要公开到 Issue、日志或截图中。

---

## 4. 生成发送配置

除了 Cookie，程序还需要知道给谁发送、发送什么内容以及消息发送间隔。

如果不想自己写 JSON，可以直接使用配置生成器：

**https://douyin-config.pages.dev/**

生成完成后复制网站生成的完整 JSON。

一个最简单的配置例如：

```json
{
  "friends": ["好友昵称"],
  "messages": [
    {"type": "text", "value": "续火花 ✨"}
  ],
  "send_interval_seconds": {
    "min": 3,
    "max": 8
  },
  "prevent_duplicates": false
}
```

第一次使用建议先只配置：

```text
1 个好友 + 1 条文字消息
```

先把最基础的流程跑通，再增加其他功能。

---

## 5. 添加 GitHub Secrets

进入自己 Fork 的仓库，依次打开：

```text
Settings
↓
Secrets and variables
↓
Actions
↓
New repository secret
```

![进入 Secrets](https://img.908988.xyz/file/教程/douyin-auto-fire/aiPBHuxJ.webp)

![创建 Secret](https://img.908988.xyz/file/教程/douyin-auto-fire/BKtXckyQ.webp)

第一次使用至少需要添加下面两个 Secret：

| Secret | 内容 | 必须 |
| --- | --- | --- |
| `DOUYIN_COOKIE` | Cookie-Editor 导出的完整 Cookie JSON | ✅ |
| `DOUYIN_CONFIG` | 配置生成器生成的完整配置 JSON | ✅ |

### 5.1 添加 `DOUYIN_COOKIE`

点击 **New repository secret**。

Name 填：

```text
DOUYIN_COOKIE
```

Secret 粘贴刚刚导出的完整 Cookie JSON，然后保存。

### 5.2 添加 `DOUYIN_CONFIG`

再次点击 **New repository secret**。

Name 填：

```text
DOUYIN_CONFIG
```

Secret 粘贴刚刚生成的完整配置 JSON，然后保存。

配置完成后至少应该存在：

```text
DOUYIN_COOKIE
DOUYIN_CONFIG
```

GitHub 保存 Secret 后不会再次显示具体内容，这是正常现象。

---

## 6. 第一次运行：Dry Run

配置完成后，不建议第一次就直接真实发送。

项目提供了 **Dry Run** 模式，用来检查：

- Cookie 是否有效；
- 是否能够正常登录抖音；
- 是否能够找到目标好友；
- 配置是否正确。

Dry Run **不会真正发送消息**。

进入：

```text
Actions
↓
Send Douyin Messages
↓
Run workflow
```

第一次运行时，将 `dry_run` 开启（即 `true`），然后点击 **Run workflow**。

![运行 GitHub Actions](https://img.908988.xyz/file/教程/douyin-auto-fire/NLFF8g94.webp)

如果最后显示绿色的 `✓`，说明本次运行成功。

如果失败，点击本次 Workflow Run，进入：

```text
send
↓
Run
```

查看具体错误日志。不要只看最下面的 `Process completed with exit code 1`，真正的报错通常在它前面。

---

## 7. 测试真实发送

Dry Run 成功后，再手动运行一次工作流。

这一次关闭 `dry_run`，也就是：

```text
dry_run = false
```

然后运行。

这一次程序会真正向好友发送消息。

第一次真实发送仍建议只保留 **1 个测试好友**，确认好友、消息和发送结果都正确以后，再增加其他好友。

---

## 8. 每天自动运行

项目已经自带 GitHub Actions 定时任务。

工作流文件位于：

```text
.github/workflows/send.yml
```

当前默认配置：

```yaml
schedule:
  - cron: "0 0 * * *"
    timezone: "Asia/Shanghai"
```

表示每天北京时间 **00:00** 自动运行一次。

定时触发会直接进行真实发送，不会自动进入 Dry Run。

> 注意：GitHub Actions 的 `schedule` 是尽力而为的定时触发，不是精确闹钟。高负载时任务可能延迟几分钟甚至更久，但不会因为延迟而改变任务内容。若完全没有运行记录，请先确认 Actions 已启用、工作流文件位于默认分支，并检查仓库是否因长期无活动而自动停用了计划任务。

GitHub 计划任务只会在默认分支的最新提交上运行。修改 `send.yml` 后，先将修改推送到默认分支；然后在仓库的 Actions 页面确认 `Send Douyin Messages` 为启用状态。若只是希望验证配置，不要把手动运行的成功误认为计划任务已恢复：应在该工作流的运行列表中确认事件类型为 `schedule`。

### 修改运行时间

例如每天北京时间 **08:30**：

```yaml
schedule:
  - cron: "30 8 * * *"
    timezone: "Asia/Shanghai"
```

每天北京时间 **20:00**：

```yaml
schedule:
  - cron: "0 20 * * *"
    timezone: "Asia/Shanghai"
```

格式为：

```text
分钟 小时 * * *
```

---

## 9. Cookie 失效怎么办？

Cookie 并不是永久有效。

如果 Actions 日志提示登录失效、需要重新登录、安全验证或 Cookie 无效：

1. 使用浏览器重新登录抖音网页版；
2. 用 Cookie-Editor 重新导出 Cookie JSON；
3. 打开仓库 `Settings`；
4. 进入 `Secrets and variables` → `Actions`；
5. 更新 `DOUYIN_COOKIE`；
6. 保存后手动执行一次 `dry_run = true`。

Dry Run 成功后即可继续正常使用。

---

## 10. 钉钉通知（可选）

如果希望通过钉钉接收任务结果，可以额外添加：

| Secret | 内容 |
| --- | --- |
| `DINGTALK_WEBHOOK` | 钉钉机器人 Webhook |
| `DINGTALK_SECRET` | 钉钉机器人 Secret |

这两个 Secret 必须同时配置。

如果不需要钉钉通知，两个都不要添加即可，不影响项目正常运行。

---

## 11. 多账号（可选）

项目当前最多支持 **5 个抖音账号**。

第一次使用不建议直接配置多账号。先确保单账号模式下的：

```text
DOUYIN_COOKIE
DOUYIN_CONFIG
```

能够正常运行。

之后可以按照账号添加：

```text
DOUYIN_COOKIE_ACCOUNT1
DOUYIN_CONFIG_ACCOUNT1

DOUYIN_COOKIE_ACCOUNT2
DOUYIN_CONFIG_ACCOUNT2

DOUYIN_COOKIE_ACCOUNT3
DOUYIN_CONFIG_ACCOUNT3
```

以此类推，最多到 `ACCOUNT5`。

每个账号的 Cookie 和 Config 必须成对配置，不能只添加其中一个。

### 老用户增加第二个账号

如果以前一直使用：

```text
DOUYIN_COOKIE
DOUYIN_CONFIG
```

不需要删除原来的配置。

可以直接增加：

```text
DOUYIN_COOKIE_ACCOUNT2
DOUYIN_CONFIG_ACCOUNT2
```

原来的 `DOUYIN_COOKIE` / `DOUYIN_CONFIG` 会继续作为第一个账号使用。

---

## 12. 运行失败后的诊断文件

如果 GitHub Actions 运行失败，项目会自动上传诊断文件，可能包括：

```text
run.log
result.json
screenshots/
traces/
```

进入失败的 Workflow 页面，在页面底部找到 **Artifacts** 即可下载。

失败诊断 Artifact 默认保留 **3 天**。

这些文件可以帮助判断：

- Cookie 是否失效；
- 是否出现安全验证；
- 好友是否没有找到；
- 页面结构是否变化；
- Playwright 在哪一步失败。

> ⚠️ 截图和日志可能包含聊天内容或账号相关信息，请不要直接公开上传。

---

## 第一次使用推荐流程

```text
Fork 项目
    ↓
启用 Actions
    ↓
登录抖音
    ↓
导出 Cookie
    ↓
生成发送配置
    ↓
添加 DOUYIN_COOKIE
    ↓
添加 DOUYIN_CONFIG
    ↓
开启 Dry Run
    ↓
确认运行成功
    ↓
关闭 Dry Run
    ↓
测试真实发送
    ↓
确认成功
    ↓
等待每天自动运行
```

第一次不要同时配置多账号、多个好友、原生表情、随机消息和钉钉通知。

先把最基础的流程跑通，这样即使出现问题，也更容易判断是哪一步出了问题。

---

## 返回项目主页

👉 [返回 douyin-auto-fire](../README.md)
