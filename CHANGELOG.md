# Changelog

## relayer-v0.2.1

# Trustody Relayer 0.2.1

The relayer now detects each EVM network's fee model by itself, reports on-chain results only once they are final, settles sped-up transactions on whichever one lands, can run without the default public nodes, and uses up to three default public nodes per network.

What's new

- **Fee model detection.** When you run `rpc add` and on every round, the relayer looks at the network's nodes: code at the OP Stack GasPriceOracle address (`0x420000000000000000000000000000000000000F`) means OP Stack; otherwise a latest block with a non-zero `baseFeePerGas` is EIP-1559, a zero base fee is EIP-1559 with a zero base fee, and a block without `baseFeePerGas` is legacy gas price. `rpc add` shows the detected model, and the relayer includes it in its status report to each wallet. If detection fails, the report simply leaves it out; nothing else changes.
- **Simpler custom networks.** When an operator adds a custom EVM network in the wallet, the fee model is taken from the selected relayer's detection instead of being chosen by hand, and the wallet refuses the network until that relayer has detected it. Only the OP Stack L1 data-fee margin is still entered by the operator.
- **Results are reported only once final.** A wallet settles a request as soon as the relayer reports its result, so an EVM result (success or failure) is now reported only when the block holding the receipt is at or below the node's `finalized` block; nodes that do not support the `finalized` tag need 12 blocks on top of it. A shallow reorganization can therefore no longer undo a result the wallet has already acted on. Chains with instant finality (such as Arc, Plasma and BNB Chain) settle almost immediately; Ethereum takes about 13 minutes, and Arbitrum, Base and OP Mainnet follow Ethereum finality (about 15 to 30 minutes). TRON results were already read from solidified blocks and are unchanged.
- **Sped-up transactions settle whichever one lands.** After an in-cap speed-up, the relayer looks up the result of the new transaction and of the ones it replaced (same nonce), and reports whichever one is on chain; the wallet accepts any of them, so a request no longer waits for manual reconciliation when the original transaction is mined first.
- **Optional default public nodes.** Install with `--no-public-nodes` (`-NoPublicNodes` on Windows), or turn off "Use default public nodes" next to the system tabs of the wallet's install card, to use only the nodes you add with `rpc add`: networks without your own nodes are then neither reported as supported nor executed. On an existing installation, running the install command with that option (or `trustody-relayer set public_nodes false`; `true` turns them back on) changes the setting; restart the service for it to take effect. `status` and `check` show the setting.
- **Up to three default public nodes.** With default public nodes on, the relayer uses at most three of a network's default nodes, so results can be cross-checked on several nodes without any setup. The wallet catalog now lists three tested public nodes from different providers for every EVM and TRON network, including the new Plasma and Circle Arc networks (off by default; a wallet operator turns them on).
- **Late opposite results are logged.** If the wallet has already settled a request when the relayer reports the opposite result, the relayer logs a warning; the wallet keeps the earlier result, records the report and flags the relayer for review.

Known limitations

- The OP Stack L1 data-fee margin cannot be detected and is still entered by hand.
- Same platform and signing limitations as 0.2.0.

Upgrading

- No configuration changes are needed; 0.2.0 configurations work as they are, and default public nodes stay on unless you turn them off.
- A wallet keeps working with relayers that are still on 0.2.0, but custom EVM networks can only be added through a relayer on 0.2.1 or later, because 0.2.0 does not report a fee model. Relayers on 0.2.0 also keep looking up only the newest transaction after a speed-up, and report results without waiting for finality; upgrade them.
- Installations with automatic upgrades on move to 0.2.1 once the platform approves it.

## relayer-v0.2.0

# Trustody Relayer 0.2.0

The relayer now serves a list of wallets, reads each wallet's network catalog instead of local network settings, uses several nodes per network, rotates its identity per wallet without downtime, and upgrades itself to versions the platform has approved.

What's new

- **Wallet list.** One installation serves any number of wallets. `add-wallet`, `remove-wallet` and `list-wallets` manage the list; running the install command from another wallet on a machine that already has the relayer only adds that wallet (nothing is reinstalled). Networks, chain IDs and default public nodes come from each wallet's network catalog; the relayer reports back which networks it supports.
- **Your own nodes (`rpc`).** `trustody-relayer rpc list | add | remove | move | test` configures execution nodes on the relayer machine, by URL (optional headers) or with a provider template (Alchemy, Infura, QuickNode, TronGrid). A node is added only after its chain ID (EVM) or genesis block (TRON) matches; otherwise the command refuses and shows what the node reported. Networks without your own nodes use the wallet's default public nodes. The node file is readable by the relayer's user only, and API keys never appear in status reports or error messages.
- **Several nodes per network.** Every round checks each node's network identity. Signed transactions are broadcast to all checked nodes at once, other reads fall over to the next healthy node, and each result is cross-checked on a second node; if two nodes disagree after a retry, the relayer reports the result as inconsistent so it can be checked by hand in the wallet.
- **Fee models from the catalog.** EIP-1559 networks (including those with a zero base fee), OP Stack networks and legacy gas-price networks are priced according to the model the wallet's catalog gives.
- **Identity rotation.** `rotate` creates a new identity and switches every wallet to it while the service keeps running; wallets that cannot be reached right now keep the old identity until they confirm, and the old identity file is deleted once every wallet has switched.
- **Automatic upgrades.** On by default: at start and every 6 hours the relayer looks at the latest GitHub release and upgrades only to a newer version that the platform has approved with the same checksum file, whose `SHA256SUMS` carries a valid Sigstore signature from the official release workflow, whose files match their checksums and that passes a self-check before it replaces the running program. Upgrades are skipped while the platform is unreachable; container images never upgrade themselves (redeploy with the new image digest shown in the wallet). The installers fetch a fingerprint-pinned cosign when none is installed.

Installation changes

- The install command takes the wallet number as its first argument and the platform as `--platform <id>` (`-Platform` on Windows): `sh install.sh <wallet-number> --platform <id>`.
- `--no-auto-upgrade` (`-NoAutoUpgrade` on Windows) turns automatic upgrades off.
- `--station`, `--evm*` and `--tron*` are removed; the setup wizard no longer asks for RPC nodes (use `rpc add` after installing).
- The wallet's install command always follows the newest version the platform has approved: it verifies `SHA256SUMS` against the platform's record, then the installer against `SHA256SUMS`, and stops on any mismatch.

Known limitations

- If the macOS and Windows binaries of this release are not code-signed or notarized, they still carry SHA-256 checksums and Sigstore signatures, and the installers always verify the checksum. A copy downloaded with a browser may then be blocked once — on macOS remove the quarantine flag (`xattr -d com.apple.quarantine <file>`), on Windows choose "More info → Run anyway".
- Solana is not supported yet (the `helius` provider template says so).
- One active instance per relayer identity; stop the old instance before starting a new one on another host.

Upgrading

- **0.1.0 configurations are not compatible and are not migrated; reinstall.** Uninstall 0.1.0 with its own installer (`sh install.sh --uninstall`, or `install.ps1 -Uninstall` on Windows), delete its configuration directory (`/etc/trustody-relayer` for a system service, `~/.config/trustody-relayer` on Linux, `~/Library/Application Support/trustody-relayer` on macOS, `%APPDATA%\trustody-relayer` on Windows), then run the install command shown in the wallet (Settings → Relayer) and register the new relayer ID there. 0.2.0 rejects a 0.1.0 configuration file instead of guessing.
- Later versions are installed automatically once the platform approves them, unless automatic upgrades are turned off.

## relayer-v0.1.0

# Trustody Relayer 0.1.0

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
