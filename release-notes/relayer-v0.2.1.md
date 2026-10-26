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
