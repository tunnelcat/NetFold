# ⌬ NetFold

A single-file HTML tool for parsing and merging Nessus and Nmap scan output into clean, filterable tables. No install, no server, no dependencies — open `netfold.html` in a browser and drop your files in.

---

## What it does

Takes raw `.nessus` and nmap XML files and turns them into structured tables you can filter, sort, deduplicate, and export. Useful when you're juggling multiple scan files across different targets and need to make sense of it quickly.

## Views

| View | Description |
|------|-------------|
| **Nessus** | Ports from Nessus SYN/UDP scanner results only |
| **Nmap** | All ports from nmap XML (open TCP/UDP) |
| **Combined** | Both sources merged — matching `ip+host+proto+port` entries collapse into a single row with a **both** badge and combined service name |
| **Summary** | Groups hosts by shared IPs and hostnames (transitive). One row per group, ports displayed as a chip grid. Collapsible rows. |

## Features

- **Auto-detects format** — drop `.nessus` or nmap XML into the same box, it figures it out
- **Deduplication** — `ip+proto+port` keyed, across multiple files of the same type
- **Exclusions** — exclude IPs by exact, CIDR, range (`10.0.0.1-10.0.0.50`), or wildcard (`192.168.1.*`); hostnames by exact or wildcard (`*.corp.lan`)
- **Port exclusions** — single ports or ranges (`8080-8090`)
- **Filters** — IP/hostname, exact port, fuzzy service name, protocol, open ports only toggle
- **Summary grouping** — union-find algorithm groups hosts that share an IP or hostname transitively
- **Export CSV** — exports whatever view is active, UTF-8 with BOM (Excel-safe)
- **Light/dark mode**

## Nessus parsing notes

Only plugin IDs `11219` (SYN scanner) and `14274` (UDP scanner) are used for port discovery. Everything else is ignored. Port 0 is always dropped.

Hostname resolution priority: `host-fqdn` → `hostname` → `netbios-name`. IP is read from `host-ip`, falling back to the `ReportHost` name attribute only if missing.

## Usage

1. Download `netfold.html`
2. Open it in any modern browser
3. Drop your scan files in
4. Filter, sort, export

No internet connection required after the initial font load (Google Fonts). If you need fully offline use, the font will fall back to `Fira Code` or monospace.

## Supported formats

- `.nessus` (Nessus v2 XML)
- Nmap XML (`-oX`)

## License

MIT — do whatever you want with it.

---

Made by [tunnelcat](https://github.com/tunnelcat)
