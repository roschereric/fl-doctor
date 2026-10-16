# fl-doctor 0.1.0

First release.

## Files

| File | For |
|---|---|
| `fl-doctor-0.1.0-x86_64-linux` | the FlowLink collector (Linux x86_64, static, no dependencies) |
| `fl-doctor-0.1.0-aarch64-linux` | Linux aarch64 collectors |
| `fl-doctor-0.1.0-aarch64-macos` | a Mac with Apple silicon: `read` (analyze a pcap or an evidence bundle) and `demo` only |
| `SHA256SUMS` | checksums: `sha256sum -c SHA256SUMS` |
| `fl-doctor-0.1.0.cdx.json` | software bill of materials (CycloneDX) |

Not signed: verify with `SHA256SUMS`. On macOS, a downloaded binary may be blocked by Gatekeeper the first time
(`xattr -d com.apple.quarantine fl-doctor-0.1.0-aarch64-macos`).

## What it does

TUI and CLI to troubleshoot Illumio FlowLink on the collector: exporters and ports, IPs in the flows (evidence ladder
N0–N4), FlowLink configuration and log (cycles, 204 responses, PCE errors), IP rankings, and an evidence bundle for
support. See the documentation in `docs/`.

## Known limits

- Not yet validated on a real Illumio FlowLink user's collector. RHEL 7 (kernel 3.10) is not verified.
- `sudo fl-doctor` is equivalent to root: do not grant it in sudoers to accounts that should not have root.
- Times are shown in UTC.
- A few texts still come out in the other language (for example the usage errors of the command line, which are checked before the language is known, and some status labels of the Exporters tab with `--lang es`); the documentation lists them under "Troubleshooting and limits".
- The exit test to the PCE repeats every 30 s while the interface is open; its detail says how long ago it ran (` · tested N s ago`) and, past 120 s, D1–D4 read "no data". `--offline` turns it off.
- A watched network or range cannot be removed from the interface; edit `watchlist.toml`.

Unofficial tool, provided "as is", without Illumio support.
