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
