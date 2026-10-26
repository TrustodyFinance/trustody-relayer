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
