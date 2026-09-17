# Pico 官方仓库自动同步到 Gitee

同步源：https://github.com/LuckfoxTECH/Luckfox-Pico-docs

同步目标：https://gitee.com/LuckfoxTECH/Luckfox-Pico-docs

目标 SSH 地址已写入工作流的 GITEE_SSH_URL，无需添加仓库变量。

推送官方 GitHub 仓库的 main 或 Git 标签会触发 Sync to Gitee，也可以在 Actions 页面手动运行。工作流仅在 LuckfoxTECH/Luckfox-Pico-docs 仓库中执行。

## 首次配置

1. 在有目标仓库写权限的 Gitee 个人账号 SSH 公钥设置中添加专用同步公钥。仓库部署公钥只支持拉取，不能用于推送。
2. 在官方 GitHub 仓库的 Settings → Secrets and variables → Actions 中添加仓库 Secret：名称 GITEE_SSH_PRIVATE_KEY，值为对应私钥的完整内容，包括 BEGIN/END 行。
3. 推送工作流后，检查 Actions 运行结果，并核对两边 main 和标签的提交编号。

| 配置位置 | 内容 |
| --- | --- |
| Gitee 个人账号 SSH 公钥 | 公钥文件内容，文件名通常以 .pub 结尾 |
| GitHub 仓库 Actions Secret | 同一对密钥的私钥，文件名没有 .pub 后缀 |
| .github/gitee_known_hosts | Gitee 官方服务器公开主机公钥，用于校验服务器身份 |

GitHub 个人账号 SSH 公钥用于本机向 GitHub 推送，不能代替仓库 Actions Secret。同步账号的私钥和公钥均应保存在 Git 仓库之外。

## 同步行为

- 同步 main 的已提交文件和历史，并同步存在的 Git 标签。
- Gitee main 只是落后时，正常快进更新；存在独立提交时会拒绝覆盖，需要先处理历史差异。
- 保留 Gitee 其他分支，不删除标签，不使用强制推送。
- GitHub Releases 附件、Issues 和仓库设置不属于 Git 内容，不通过本工作流复制。
- Docs/en/ipc 和 Docs/zh/ipc 当前为嵌套仓库引用；本工作流同步父仓库保存的提交引用，不会把内部文件转换成普通文件。文件结构调整需单独处理。
- 首次传输包含仓库的二进制文件；若 Gitee 明确报告容量或单文件限制，应按实际错误处理。

## 验证与排障

工作流入口：https://github.com/LuckfoxTECH/Luckfox-Pico-docs/actions/workflows/sync-gitee.yml

- Secret 缺失：在官方仓库添加名称完全一致的 GITEE_SSH_PRIVATE_KEY。
- DeployKey does not support push code：将公钥配置到有仓库写权限的 Gitee 个人账号 SSH 公钥中。
- Permission denied (publickey)：核对两边是否使用同一对密钥。
- non-fast-forward：检查 Gitee 是否有独立提交，合并需要保留的内容后再运行。
- Host key verification failed：对照 https://help.gitee.com/account/gitees-ssh-key-fingerprints 核验主机公钥。

本地个人 fork 与官方仓库是不同的 GitHub 仓库。只有进入官方仓库 main 的提交才会触发官方仓库的自动同步。
