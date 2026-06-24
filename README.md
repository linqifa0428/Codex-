# VS Code Codex 本地与远程服务器部署教程

本教程用于说明如何在 **本地 VS Code** 和 **SSH 远程服务器** 中登录 Codex 插件，并通过 `auth.json` 与 SSH 端口转发解决远程登录失败的问题。

> 重要提醒：`auth.json` 是登录缓存文件，可能包含敏感授权信息。它只能上传到你自己的远程服务器配置目录中，**不要提交到 GitHub 仓库**，也不要分享给他人。本仓库已在 `.gitignore` 中忽略该文件。

## 目录

- [1. 本地 VS Code 登录 Codex](#1-本地-vs-code-登录-codex)
- [2. 将本地 auth.json 上传到远程服务器](#2-将本地-authjson-上传到远程服务器)
- [3. 配置 SSH RemoteForward](#3-配置-ssh-remoteforward)
- [4. 配置远程 VS Code 代理设置](#4-配置远程-vs-code-代理设置)
- [5. 重启 VS Code 并验证远程登录](#5-重启-vs-code-并验证远程登录)
- [6. 常见问题](#6-常见问题)

## 1. 本地 VS Code 登录 Codex

1. 确保当前网络环境可以正常访问 OpenAI 官方登录页面。不要选择无法访问 OpenAI 的地区或节点。
2. 打开本地 VS Code，进入扩展市场，搜索并安装 Codex 插件。

![Codex 插件](assets/01-codex-extension.png)

3. 点击 Codex 扩展中的登录入口，选择通过 GPT / OpenAI 账号登录。
4. 页面会跳转到 `openai.com` 官方登录页面，通过账号密码或验证码完成登录。
5. 登录成功后，Codex 界面会显示 `Signed in to Codex` 或类似状态。

如果登录失败，可能会出现如下错误界面：

![Codex 登录失败提示](assets/02-sign-in-error.png)

遇到该问题时，优先检查网络环境是否能稳定访问 `openai.com`，必要时更换可用节点后重新登录。

## 2. 将本地 auth.json 上传到远程服务器

远程服务器登录 Codex 前，先确保本地 VS Code 的 Codex 扩展已经成功登录。然后将本地登录成功后的 `auth.json` 上传到远程服务器的 Codex 配置目录。

### 2.1 在 Windows 本地找到 auth.json

Windows 默认路径通常为：

```text
C:\Users\你的用户名\.codex\auth.json
```

请确认该文件存在，并且是本地 Codex 成功登录后生成的文件。

### 2.2 上传到远程服务器

在 VS Code 中通过 SSH 连接远程服务器后，找到远程服务器用户目录下的 `.codex` 配置文件夹。一般位置类似：

```text
/home/你的用户名/.codex/
```

![远程服务器 .codex 文件夹](assets/03-remote-codex-folder.png)

将本地的 `auth.json` 上传到该目录中：

![上传 auth.json](assets/04-auth-json-location.png)

上传后的远程路径一般应为：

```text
/home/你的用户名/.codex/auth.json
```

## 3. 配置 SSH RemoteForward

在 VS Code 中打开 SSH 配置文件：

![打开 SSH config](assets/05-open-ssh-config.png)

在目标服务器的 `Host` 配置段中添加 `RemoteForward`。例如本地代理端口为 `7897` 时：

```sshconfig
Host your-server
    HostName xxx.xxx.xxx.xxx
    User your_username
    Port 22
    IdentityFile ~/.ssh/id_rsa
    RemoteForward 7897 127.0.0.1:7897
```

![SSH RemoteForward 配置示例](assets/06-ssh-remote-forward.png)

其中：

- 第一个 `7897`：远程服务器上开放的转发端口。
- `127.0.0.1:7897`：本地机器上的代理地址和端口。
- 如果你的本地代理端口不是 `7897`，需要把两处端口都改成你自己的端口。

## 4. 配置远程 VS Code 代理设置

在 VS Code 中按下：

```text
Ctrl + Shift + P
```

搜索并打开：

```text
Preferences: Open Remote Settings (JSON)
```

![打开 Remote Settings JSON](assets/07-remote-settings-json.png)

在远程设置 JSON 中加入以下配置。下面示例仍以 `7897` 为端口，如果你的端口不同，请同步修改。

```json
{
  "http.proxy": "http://127.0.0.1:7897",
  "http.proxySupport": "on",
  "http.proxyStrictSSL": false,
  "terminal.integrated.env.linux": {
    "http_proxy": "http://127.0.0.1:7897",
    "https_proxy": "http://127.0.0.1:7897"
  }
}
```

如果你的远程设置文件中已经有其他配置，请不要直接覆盖整个文件，而是把上述字段合并进去，并保证 JSON 格式合法。

## 5. 重启 VS Code 并验证远程登录

完成以上配置后，需要重启 VS Code，使 SSH 转发和远程代理配置生效。

重启后按以下流程检查：

1. 重新通过 SSH 连接远程服务器。
2. 打开远程服务器中的 Codex 插件。
3. 尝试登录或检查登录状态。
4. 如果显示 `Signed in to Codex` 或类似状态，说明远程服务器 Codex 登录成功。

远程登录失败时，通常需要优先检查：

- 本地 Codex 是否已经登录成功。
- 远程服务器的 `/home/你的用户名/.codex/auth.json` 是否存在。
- SSH `RemoteForward` 是否写在正确的 `Host` 配置段中。
- 本地代理端口是否与 `RemoteForward` 和 Remote Settings 中的端口一致。
- VS Code 是否已经完整重启并重新连接服务器。

## 6. 常见问题

### 6.1 出现 `Sign-in could not be completed` 怎么办？

该错误通常表示 Codex 登录阶段无法完成 token exchange。可以按以下顺序排查：

1. 确认当前网络环境可以访问 `openai.com`。
2. 更换可用的网络节点后重新登录。
3. 确认本地 VS Code 的 Codex 已经成功登录。
4. 确认远程服务器中已经上传本地成功登录后的 `auth.json`。
5. 确认远程 VS Code 的代理配置使用的是 `http://127.0.0.1:你的端口号`。
6. 重启 VS Code 并重新连接远程服务器。

### 6.2 可以把 auth.json 放进 GitHub 吗？

不可以。`auth.json` 是登录授权缓存文件，不应该被公开提交。请只把教程、截图和配置模板上传到 GitHub，不要上传真实的 `auth.json`。

### 6.3 端口必须是 7897 吗？

不是。`7897` 只是示例端口。你需要根据自己的本地代理端口进行替换，并确保以下三个位置保持一致：

1. SSH config 中的 `RemoteForward` 端口。
2. Remote Settings JSON 中的 `http.proxy` 端口。
3. Remote Settings JSON 中 `terminal.integrated.env.linux` 里的 `http_proxy` 和 `https_proxy` 端口。
