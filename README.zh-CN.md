# Trustody Relayer

[English](README.md)

这里是 Trustody Relayer 的官方二进制分发仓库。Relayer 将满足审批和放行条件的钱包请求推进至已接入的网络，并协助跟踪执行结果。

执行器身份是访问钱包执行接口的认证凭据，**不是资产私钥**。执行器不能自行批准付款，也不能修改已批准的收款方或金额。自建执行器可减少对平台执行服务的依赖，但不改变平台对钱包程序的升级权限。

## 版本与下载

尚未发布正式版本。

官方正式版程序通过 [Releases](https://github.com/TrustodyFinance/trustody-relayer/releases) 下载。本仓库仅分发正式版，不公开实现源码。GitHub 自动提供的 **Source code (zip)** 和 **Source code (tar.gz)** 包含本分发仓库的文档等公开文件，不包含 Relayer 实现源码。

首个经过验证的正式版本发布后提供下载。

## 安装与配置

优先使用钱包 **设置 → 执行器 → 切换到自建执行器** 生成的固定版本安装命令。命令核对安装脚本指纹，安装程序再验证可执行文件。钱包编号、平台编号、EVM 网络与 TRON 配置均应采用当前钱包的实际值；给执行器添加网络配置，不会自动开启钱包的对应网络转出权限。

正式版本发布后提供安装命令。

安装向导创建配置和私密身份文件，并显示公开的执行器 ID。回到钱包登记此 ID，完成相应治理审批；提交请求不等于切换已经生效。不要把 PEM 身份文件内容粘贴到钱包，也不要将它放进日志或公开问题报告。

## 日常运行

以下命令假定程序已安装，且 `config.toml` 是本次操作使用的配置。请替换为安装实例的实际配置路径。Linux 系统安装应使用专用服务账号执行配置与身份操作，并明确指定配置路径。

```sh
trustody-relayer --version
trustody-relayer --help
trustody-relayer status --config ./config.toml
trustody-relayer check --config ./config.toml --lang zh
trustody-relayer run --config ./config.toml
```

`status` 显示身份、钱包接受情况与服务状态，命令成功不等于服务健康。`check` 检查配置、权限、网络连接与钱包访问。`run` 为前台运行，关闭终端即停止。**同一身份只运行一个实例**，跨主机也应遵守；已有后台服务时，不要再启动前台副本。需要持续运行时，使用主机的服务管理器或容器平台托管。

重启、升级或重新部署时，应保留并保护配置与身份材料。锁与钱包发现缓存的路径须由运行账号可写，私密文件须限制为所有者访问。通过 `set` 修改配置后，先检查，再重启进程；运行中的程序不会自动加载修改。

## 容器与 Secret

正式版本发布后提供签名镜像及其固定摘要。

配置可来自私密文件或 `TRUSTODY_RELAYER_CONFIG_TOML`。当前身份在配置字段 `identity_pem` 与环境变量 `TRUSTODY_RELAYER_IDENTITY_PEM` 中**只能选一种**，同时提供会被拒绝。`next_identity_pem` 与 `TRUSTODY_RELAYER_NEXT_IDENTITY_PEM` 也遵循相同规则。本机文件路径不会自动出现在容器内。

云端采用环境配置时，删除已通过对应 PEM Secret 提供身份的文件字段。将顶层 `lock_file`、`state_file` 放在 TOML 的第一个 `[表名]` 之前，使用私有可写挂载目录或 `/tmp` 下的可写路径。身份 Secret 应独立于容器临时存储持久保存。具有 Secret 查看权限的管理员可以读取执行凭据，应限制访问权限。

## 身份轮换、升级与卸载

文件配置部署可运行 `trustody-relayer rotate --config ./config.toml` 生成下一个身份。重启服务以加载新旧身份，再到钱包提交新 ID 并完成审批。所有配置的钱包均接受新身份、拒绝旧身份后，运行 `trustody-relayer rotate --finish --config ./config.toml`，重启并确认正常工作，最后再使用 `trustody-relayer rotate --cleanup --config ./config.toml` 清理停用的本地身份文件。

云端 Secret 部署先在本机运行 `trustody-relayer init --identity-out next.pem`，将新身份作为 `TRUSTODY_RELAYER_NEXT_IDENTITY_PEM` 提供，并重启或重新部署。完成钱包审批，确认所有受管理钱包均接受新身份、拒绝旧身份后，将 `TRUSTODY_RELAYER_IDENTITY_PEM` 替换为新 PEM，移除 NEXT Secret，再次重启或重新部署。本地 `--finish`、`--cleanup` 命令不能修改云端环境配置。

升级需手动进行。核对目标版本与文件指纹，保留配置和身份，先停止旧实例，再启动新实例，随后运行 `check`、`status` 并观察钱包请求进度。卸载或删除部署前，先安排替代执行器，或在钱包完成切回平台执行器的审批；替代实例正常工作前，应保留身份与配置材料。卸载程序不会改变钱包内的授权。

钱包概念与详细操作流程请参阅 Trustody 钱包内的使用指南。报告分发问题时，可提供版本和脱敏后的错误信息；不要附带配置 Secret 或身份文件。
