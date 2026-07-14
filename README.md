# ⌬ NetFold

A single-file HTML tool for parsing and merging Nessus and Nmap scan output into clean, filterable tables. No install, no server, no dependencies. Open `netfold.html` in a browser and drop your files in.

---

## What it does

Takes raw `.nessus` and nmap XML files and turns them into structured tables you can filter, sort, deduplicate, and export. Useful when you're juggling multiple scan files across different targets and need to make sense of it quickly.

## Usage

1. Download `netfold.html`
2. Open it in any modern browser
3. Drop your scan files in
4. Filter, sort, export

No internet connection required after the initial font load (Google Fonts). For fully offline use, the font falls back to `Fira Code` or monospace.

## Supported formats

- `.nessus` (Nessus v2 XML)
- Nmap XML (`-oX`)

## Views

| View | Description |
|------|-------------|
| **Nessus** | Ports from the Nessus SYN/UDP port-discovery plugins. |
| **Nmap** | All ports from nmap XML. |
| **Combined** | Both sources merged; matching entries collapse into one row with a `both` badge. |
| **Summary** | Hosts grouped by shared IPs and hostnames (transitive). One row per group, ports shown as a chip grid, rows collapsible. |

## Features

- **Format auto-detection**: by content, not extension or filename. A file is parsed by its actual XML structure (root tag and element presence), so a file with the wrong extension, no extension, or a misleading name still parses if valid.
- **Deduplication**: across multiple files of the same type. See [Keying and merging](#keying-and-merging).
- **Duplicate filenames are rejected**:
  - The first file with a given name wins, by drop order (not load order).
  - Every later file with that name is skipped with a warning, so a same-batch or later duplicate cannot silently overwrite loaded data.
- **Host-level entries (port 0)**:
  - Nessus port-0 findings from the port-discovery plugins, and Nmap hosts with no open or listed ports, appear as a synthesized `port 0 / reserved` row, so a scanned-but-empty host is never invisible.
  - Toggled independently of "Open ports only", and still subject to the IP/port/service/protocol filters.
- **Exclusions**: one rule per line, with invalid rules flagged inline by line number.
  - **IP exact**: `192.168.1.1`
  - **CIDR**: `10.0.0.0/24` 
  - **Whole-IP range**: `10.0.0.1-10.0.0.50`
  - **Per-octet range** (nmap-style): `10.0-5.1.1-50`
  - **Hostname exact**: `web.corp.lan`
  - **Wildcard**: `*` anywhere, any number of them, spanning dots, matched against both the IP and the hostname. Examples: `192.168.1.*`, `172.*.23.*`, `172*`, `*corp.la*`, `*.corp.lan`, `fe80:*` (IPv6).
- **Port exclusions**: single ports or ranges (`8080-8090`).
- **Filters**: IP/hostname, exact port, fuzzy service name, protocol, plus "Open ports only" and "Show host-level entries" toggles. All filters combine with AND, including on host-level rows.
- **Summary grouping**: a union-find pass groups hosts that share an IP or hostname, transitively, so a host known by two names (Nessus FQDN vs Nmap PTR) shows both under one group.
- **Export CSV**: exports the active view, UTF-8 with BOM (Excel-safe).
- **Copy as TSV**: copies the active view to the clipboard, for pasting into Excel or Sheets.
- **Column resizing**: per-column auto-fit sizing from the loaded data; manual resizes persist across re-renders.
- **Light and dark mode.**
- **Color-coded toast notifications**

## Nessus parsing notes

- **Plugins**: only `11219` (SYN scanner) and `14274` (UDP scanner) are used for port discovery.
- **State**: NetFold scans the plugin's `plugin_output` for the phrase "was found to be open". Found means `open`; not found means `unknown` (never assumed open).
- **Hostname**: whichever of `host-fqdn`, `hostname`, or `netbios-name` appears first in the file.
- **IP**: read from `host-ip`. If that's missing, NetFold falls back to the `ReportHost` name attribute, using it as the IP only if it's valid IPv4; otherwise that value becomes the hostname.

## Nmap parsing notes

- **Dual-stack hosts**: IPv4 is always used, even if IPv6 appears first. IPv6-only hosts keep their IPv6 address.
- **Hosts with no ports listed**: still appear, as a synthesized host-level row, instead of disappearing from the results.

## Keying and merging

NetFold identifies a row by its host (IP, or hostname if there's no IP), protocol, and port. That identity is used two ways:

- **Same-source deduplication**: two Nessus files (or two Nmap files) describing the same host/protocol/port collapse into one row, whatever the files are named. If they disagree on state, `open` wins, so confirming a port open on a re-scan isn't lost to an older file loading first.
- **Cross-source merge** (Combined view only): a Nessus row and an Nmap row combine into one `both` row only if they also agree on hostname (case-insensitive) and state. If the two sources report different hostnames for the same host/port, both rows stay separate so neither name is dropped. An empty hostname on one side never matches a non-empty one on the other.

## License

GNU General Public License v3.0.

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version. It is distributed in the hope that it will be useful, but without any warranty, without even the implied warranty of merchantability or fitness for a particular purpose. Full text: <https://www.gnu.org/licenses/gpl-3.0.html>

---

Made by [tunnelcat](https://github.com/tunnelcat)
