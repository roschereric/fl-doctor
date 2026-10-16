# fl-doctor

**Troubleshoot Illumio FlowLink from the collector, in minutes.**

fl-doctor is a terminal tool (TUI + CLI) that runs on the FlowLink collector and shows, with evidence, where the flow
pipeline breaks: exporter → collector host → FlowLink → delivery to the PCE. One static Linux binary, no dependencies,
read-only.

![fl-doctor Home: links A–E and the wire → socket → decoded → delivered funnel](docs/img/home_en.svg)

## Why

"I configured the switch to export flows to FlowLink, but I don't see the servers on the map." Finding the break usually
takes hours of `tcpdump`, `tshark`, log reading and back-and-forth with the Illumio FlowLink user. Four failures seen in the field:

1. **FlowLink never uploaded anything**: the collector could not resolve the PCE's DNS name. The error was in the log.
2. **Millions of "bad data" drops**: they were 100% ICMP, not a data problem.
3. **Wrong capture point**: the exporter did not see the server segment, so the IPs never showed up.
4. **Wrong port**: the switch exported to a UDP port FlowLink was not listening on.

fl-doctor surfaces each of these on its first screen.

## What it answers

| Question | Where |
|---|---|
| Where does the chain break? | **Home**: links A–E and a funnel wire → socket → decoded → delivered |
| Is my switch sending, in which format, to the right port? | **Exporters**: NetFlow v5/v9, IPFIX, sFlow; templates, sampling, and whether FlowLink accepts it |
| Do I see this server's flows, and how far do they get? | **IPs › Watched**: evidence ladder N0–N4 per IP (seen on the wire → accepted → processed → delivered with HTTP 204) |
| Which IPs send the most, through which switch? | **Flows › Ranking** and dashboards |
| Which IPs did the collector see, and which does FlowLink report? | **IPs › Seen** |
| Is FlowLink configured right and uploading? | **FlowLink › Config / Log**: YAML validated key by key, cycles, 204 responses, DNS and connection errors |
| What do I send to support? | One key (`e`) builds a bundle: summary in Spanish and English, relevant log, config with secrets hidden, and a pcap (original or trimmed) |

## Get it

Binaries are in [`releases/`](releases/), one folder per version, with `SHA256SUMS` and an SBOM (not signed).

```
sha256sum -c SHA256SUMS
sudo install -m 0755 fl-doctor-0.1.0-x86_64-linux /usr/local/bin/fl-doctor
sudo /usr/local/bin/fl-doctor selftest      # checks the platform; no network, no FlowLink needed
sudo /usr/local/bin/fl-doctor               # opens the TUI
fl-doctor demo                              # the TUI with synthetic data, no root (training)
```

On RHEL and derivatives `sudo` does not search `/usr/local/bin`: use the full path.

Headless, for scripts and cron (text, `--json` or `--csv`):

```
sudo fl-doctor health                                   # checks A–D and FlowLink log indicators
sudo fl-doctor trace --ip 10.20.30.77 --observe 300     # N0–N4 ladder for one IP
sudo fl-doctor top --by exporter --observe 120          # who sends the most
sudo fl-doctor log --since 2h                           # cycles, responses, errors
sudo fl-doctor evidence --ip 10.20.30.77 --last 15m     # bundle for support
fl-doctor read capture.pcap --config config.yaml        # analyze a pcap on your laptop
```

## Documentation

In [`docs/`](docs/) (single HTML files: download and open in a browser):

- **[Documentation](docs/fl-doctor-docs_en.html)**: install, the interface and its controls, every tab and submenu step
  by step, use cases with screenshots, evidence for support, CLI reference, troubleshooting.
- **[Briefing](docs/fl-doctor-briefing_en.html)** (3 minutes): the problem, the five main use cases, guarantees.

English is the reference documentation. Spanish translations: [documentación](docs/fl-doctor-docs_es.html) ·
[briefing](docs/fl-doctor-briefing_es.html).

## Guarantees

- **Read-only.** It never restarts FlowLink or changes its config, the firewall, routes or `sysctl`. It writes only to
  `/var/tmp/fl-doctor/` and its own preferences.
- **Takes no packets away from FlowLink.** It copies traffic with AF_PACKET, like `tcpdump`, and never binds the
  collector's ports.
- **Sends nothing out.** No telemetry, no auto-update. The only outbound check is an optional reachability test to the
  PCE address from FlowLink's config, without credentials, repeated every 30 s while the interface is open (`--offline`
  turns it off).
- **Secrets stay hidden.** The API key is never shown; the evidence bundle redacts sensitive keys and secret-looking values
  (PEM, JWT). Nothing leaves the machine unless you copy it.
- **Light.** Fixed memory budget (32 / 64 / 256 MB profiles). Measured on Rocky 9 x86_64 replaying a real NetStream
  capture: about 22 MB (normal, with a brief startup peak of up to about 33 MB) and about 14 MB (lean), flat over time. Low CPU priority.
- **`sudo fl-doctor` is equivalent to root**: do not grant it in sudoers to accounts that should not have root.

## Status

**0.1.0.** Static binaries for Linux x86_64 and aarch64 (macOS for `read` and `demo`). Tested on Rocky 9 and UBI 8 and in a
Rocky 9 x86_64 VM with replayed captures. **Not yet validated on a real Illumio FlowLink user's collector; RHEL 7 is not verified.**
Release notes for each version are in its `releases/<version>/` folder.

Feedback wanted: collectors where it breaks, FlowLink versions beyond 2.0.0, exporters from other vendors.

## Disclaimer

fl-doctor is an unofficial field tool. It is **not an Illumio product**, is **not supported by Illumio**, and is provided
**"as is"**, without warranty of any kind. Use it under your own change-control process. The source code is not
published.

## Contact

Eric Roscher, Illumio LATAM Pre-Sales Engineering.
