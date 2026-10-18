# fl-doctor 0.2.0

Second release. It adds a mascot to the interface, writes preferences with English keys (migrating the file of 0.1.0 by
itself), closes the limits listed in 0.1.0 and hardens what fl-doctor does with the privileges of `sudo`.

Downloads, checksums and the documentation of every version: https://github.com/roschereric/fl-doctor

## Files

| File | For |
|---|---|
| `fl-doctor-0.2.0-x86_64-linux` | the FlowLink collector (Linux x86_64, static, no dependencies) |
| `fl-doctor-0.2.0-aarch64-linux` | Linux aarch64 collectors |
| `fl-doctor-0.2.0-aarch64-macos` | a Mac with Apple silicon: `read` (analyze a pcap or an evidence bundle) and `demo` only |
| `SHA256SUMS` | checksums: `sha256sum -c SHA256SUMS` |
| `fl-doctor-0.2.0.cdx.json` | software bill of materials (CycloneDX) |

Not signed: verify with `SHA256SUMS`. On macOS, a downloaded binary may be blocked by Gatekeeper the first time
(`xattr -d com.apple.quarantine fl-doctor-0.2.0-aarch64-macos`).

## What is new

**The mascot.** An owl (the default), a heart monitor, a fish or a crab, drawn in pixel art with half blocks, or none (`off`).
It shows in a welcome banner (at start, 1.5 s, any key skips it, and on demand with `:about`), in a corner of Home
(terminal of 150×40 or more, when it takes no panel away); the header carries no mascot. It follows the four themes
(`brand`, `terminal`, `light`, `mono`) and reacts in eleven ways (receiving data, heavy traffic, asleep, paused, capturing
to disk, building evidence, reading the history, warning, failure, memory under pressure, watched IP seen); the line under it
says the same in words, so it is only decoration. It moves only while traffic arrives, never faster than 8 frames per
second, and stays still without traffic (no extra CPU). Choose it in Settings › Mascot, with `--mascot` or with `mascot` in
`config.toml`; `--no-splash` skips the banner and `--no-motion` keeps it still.

**Preferences in English.** `config.toml`, `site.toml`, `watchlist.toml`, `filters.toml` and `state.toml` now write their keys
(and the closed values such as `theme = "brand"`) in English. The Spanish keys of 0.1.0 are still read. The first time it loads
a `config.toml` of 0.1.0, fl-doctor rewrites it in English with `version = 2`, keeping your comments, and leaves the original as
`config.toml.v1.bak`. `historial.jsonl` is now `history.jsonl`, the `plantillas/` cache is `templates/` and a corrupt
`state.toml` is renamed `state.toml.corrupt`.

**Interface.**
- A watched network or range can be removed from the interface: `Tab` twice in IPs › Watched gives the focus to the Watched
  networks table and `Del` removes the selected row.
- The Capture tab also lists the capture to disk of one IP started with `C`, and `s` or `Del` stop it.
- Host says how old `rp_filter`, the firewall and the clock readings are ("Read N s ago"); the "tested N s ago" of the exit to
  the PCE and these ages use the host's monotonic clock, so a clock adjustment does not distort them.
- While the history of the rotated logs is read, the FlowLink tab, the indicators and the text report say "reading history N
  of M", and the history is read without gaps.
- The mascot corner never takes a panel away from Home: it is drawn only if Home keeps the same panels with it as without it
  (from 150 to 159 columns, and from 182 with the owl, 186 with the monitor or the crab, 192 with the fish). At 160 columns
  the Log indicators panel stays and no mascot is drawn. When fl-doctor analyzes a file the mascot
  sleeps and says "analyzing a file".

**One language at a time.** With `--lang es` (or `L`) everything fl-doctor shows is in Spanish, and with `--lang en`
everything is in English: tabs, panels, columns, the header, the key bar, help, palette, forms, notices, `--help`, usage
errors and the text of every subcommand and of the evidence package. What stays the same in both languages: what you write
(options, subcommands, filter language, configuration keys), the JSON output and the `FL-DOCTOR-RESULT` line, paths, units,
protocols and what is read from FlowLink's log. A test renders every screen, notice and command output in both languages and
fails on any word of the other one.

**File names in English.** The ready-made tcpdump, dumpcap and tshark commands, the captures to disk and the `Y` copy now use
English names in both languages (`flows.pcap`, `capture.pcapng`, `capture-…`, `copy-….txt`, `fl-doctor-data/` in portable
mode). Captures named `captura-…` by 0.1.0 still count towards the disk budget and are still cleaned up, and an existing
`fl-doctor-datos/` portable folder is still used.

**`q quit` in the key bar.** The bottom bar now always ends with `←→ tabs · Tab panel · q quit · ? help`.

**Safer under `sudo`.** As root, fl-doctor reads only FlowLink's own folders and its own, and only when the whole chain of
directories above them belongs to root; any other file (`--profile`, `--config`, `--patterns`, `--file`, evidence packages,
CSV lists) is opened with the credentials of the user who ran `sudo`, and a FlowLink that runs under another account is read
with that account (only while that FlowLink process is running; `--flowlink-user` and `--flowlink-exe` extend this trust only
to a system account that is not the one running `sudo`). Under `sudo`, links inside those folders are not followed: a
`config.yaml` managed as a symbolic link needs `su -`. When a file is not read for these reasons, the message names the link or
the directory and why. A `--file` that cannot be read now ends with exit code 3 and a permission message instead of "no data".
Two fl-doctor sessions on the same output folder share one disk budget (a lock keeps them from counting the same space twice),
and the black box and your own captures no longer block each other. A hung DNS lookup of the PCE is retried at most every
5 minutes. More invisible characters are filtered before anything reaches the terminal.

**Faster filters.** The `watched` and `list:` filters use an index instead of walking every entry.

**Download address.** The welcome banner, `fl-doctor --help` and every page of the documentation end with the address of the
public repository where the binaries and the documentation are downloaded.

## Upgrading from 0.1.0

Replace the binary. Scripts that call fl-doctor by the Spanish names (`salud`, `rastrear`, `--observar`, `--salida`…) keep
working. The output schema is the same (`schema=2`). If you have a `config.toml` of 0.1.0, expect `config.toml.v1.bak` next to it
the first time you start 0.2.0; delete it when you have checked the result.

## Known limits

- Not yet validated on a real Illumio FlowLink user's collector. RHEL 7 (kernel 3.10) is not verified.
- `sudo fl-doctor` is equivalent to root: do not grant it in sudoers to accounts that should not have root.
- Times are shown in UTC.
- The folders under `/var/tmp/fl-doctor` (`capturas`, `caja-negra`) and the column headers of the evidence package CSVs keep
  their 0.1.0 names, so existing paths and packages keep working.
- A notice already on screen is not re-translated when you switch language with `L`.
- The exit test to the PCE repeats every 30 s while the interface is open; `--offline` turns it off.

Unofficial tool, provided "as is", without Illumio support.
