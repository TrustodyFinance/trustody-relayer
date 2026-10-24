# Trustody Relayer 0.1.0

## English

The first public build of the Trustody Relayer, the self-hosted executor that advances approved wallet requests for your organization.

What it does

- Picks up requests your wallet has already approved, proposes network fees (EVM) or reference blocks (TRON), fetches the transaction signed by your wallet, broadcasts it and reports the hash back.
- Holds an execution credential only, never an asset private key: it cannot approve payments or change recipients, amounts or fees; a leaked credential can at most delay execution.
- Command line: `init` (setup wizard), `check` (self-test: identity, Internet Computer reachability, clock, EVM chain IDs, TRON genesis, whether your wallet accepts this relayer), `status`, `run`, `rotate` (switch to a new identity without downtime), `set`.
- Runs on Linux (x86_64, aarch64), macOS (Apple silicon and Intel) and Windows (x86_64), as a service or in the published container image.

Known limitations

- **Test release.** The macOS and Windows binaries in this release are **not code-signed or notarized** yet. Like every other file they carry SHA-256 checksums and Sigstore signatures; the installers always verify the checksum, and the Linux/macOS installer also verifies the signature when cosign is installed. Installed with the commands shown in the wallet they run directly; a copy downloaded with a browser may be blocked once — on macOS remove the quarantine flag (`xattr -d com.apple.quarantine <file>`), on Windows choose "More info → Run anyway".
- One active instance per relayer identity; stop the old instance before starting a new one on another host.

Upgrading

- First release: nothing to upgrade from. Generate the installation command in the wallet (Settings → Executor) and verify its fingerprint before running it.

## 中文

Trustody Relayer 的首个公开版本。它是自建执行器，替你的企业推进钱包里已经批准的请求。

功能

- 取得钱包已批准的请求，提议手续费(EVM)或参考区块(TRON),取回钱包签好的交易并广播，再把交易哈希回报给钱包。
- 只持有执行凭据，不持有任何资产私钥：不能批准付款，不能修改收款方、金额或手续费；凭据泄露最多造成执行延迟。
- 命令行:`init`(配置向导)、`check`(自检：身份、互联网计算机连通、时钟、EVM 链 ID、TRON 创世区块、钱包是否已接受这个执行器)、
  `status`、`run`、`rotate`(不停机换用新身份)、`set`。
- 支持 Linux(x86_64、aarch64)、macOS(Apple 芯片与 Intel)、Windows(x86_64),可作为系统服务运行，也可使用发布的容器镜像。

已知限制

- **测试版本。** 本版本的 macOS 与 Windows 程序**尚未做代码签名与公证**。它们与其他文件一样有 SHA-256 摘要和 Sigstore 签名；安装脚本
  一定校验摘要，Linux / macOS 安装脚本在本机装有 cosign 时还会校验签名。按钱包里给出的安装命令安装可以直接运行；用浏览器下载的副本可能会被拦一次:macOS 请解除隔离
  (`xattr -d com.apple.quarantine <文件>`),Windows 请选择"更多信息 → 仍要运行"。
- 同一执行器身份只能运行一个实例；换到另一台主机前先停掉旧实例。

升级

- 首个版本，无需升级。请在钱包"设置 → 执行器"生成安装命令，核对指纹后再运行。
