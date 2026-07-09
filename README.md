# ⌬ NetFold

A single-file HTML tool for parsing and merging Nessus and Nmap scan output into clean, filterable tables. No install, no server, no dependencies — open `netfold.html` in a browser and drop your files in.

---

## What it does

Takes raw `.nessus` and nmap XML files and turns them into structured tables you can filter, sort, deduplicate, and export. Useful when you're juggling multiple scan files across different targets and need to make sense of it quickly.

## Views

| View | Description |
|------|-------------|
| **Nessus** | Ports from the Nessus SYN/UDP port-discovery plugins, plus a **State** column |
| **Nmap** | All ports from nmap XML |
| **Combined** | Both sources merged, matching entries collapse into a single row with a **both** badge |
| **Summary** | Groups hosts by shared IPs and hostnames (transitive). One row per group, ports displayed as a chip grid. Collapsible rows. |

## Features

- **Auto-detects format by content, not extension or filename.** A file is parsed by reading its actual XML structure (root tag / element presence), so a `.nessus` file with the wrong extension, no extension, or a misleading name still parses correctly.
- **Deduplication** across multiple files of the same type — see [Keying and merging](#keying-and-merging) below.
- **Duplicate filenames are rejected.** The first file with a given name wins (by drop order, not load order); every later file with the same name is skipped with a warning toast, so a same-batch or later duplicate can't silently overwrite already-loaded data.
- **Host-level entries (port 0).** Nessus port-0 findings from the port-discovery plugins, and Nmap hosts with no open/listed ports, show up as a synthesized `port 0 / reserved` row so a scanned-but-empty host is never invisible. Toggleable independently of "Open ports only" and unaffected by the other filters' relevance to real ports.
- **Exclusions** — IPs by exact match, CIDR (`/24`, leading zeros in the mask rejected), whole-IP range (`10.0.0.1-10.0.0.50`), nmap-style per-octet range (`10.0-5.1.1-50`), or wildcard (`192.168.1.*`); hostnames by exact match or wildcard (`*.corp.lan`). Invalid rules are flagged inline with a line-specific error instead of failing silently.
- **Port exclusions** — single ports or ranges (`8080-8090`).
- **Filters** — IP/hostname, exact port, fuzzy service name, protocol, "Open ports only" toggle, "Show host-level entries" toggle. All filters combine (AND), including host-level rows, which are still subject to the IP/port/service/protocol filters.
- **Summary grouping** — union-find algorithm groups hosts that share an IP or hostname transitively; a host known by two different names (e.g. Nessus FQDN vs Nmap PTR) shows both under one group.
- **Export CSV** — exports whatever view is active, UTF-8 with BOM (Excel-safe).
- **Copy as TSV** — copies the active view to the clipboard as tab-separated values for pasting straight into Excel/Sheets.
- **Column resizing** with per-column auto-fit sizing based on loaded data; manual resizes persist across re-renders.
- **Light/dark mode.**
- **Toast notifications** are color-coded: green for a successful parse, yellow for a recognized-but-empty file (0 rows, usually a truncated or corrupted scan) or a skipped duplicate, red for a file that couldn't be identified as Nessus or Nmap at all.

## Nessus parsing notes

Only plugin IDs `11219` (SYN scanner) and `14274` (UDP scanner) are used for port discovery; every other plugin is ignored, including at port 0.

**State** is determined per row by searching that plugin's `plugin_output` text for the phrase "was found to be open". If found, state is `open`; otherwise it's `unknown` rather than assumed open, since these plugins occasionally also fire on port 0 (host-level, not a real port) or on ports where the output text doesn't confirm an open state.

Hostname comes from whichever of `host-fqdn`, `hostname`, or `netbios-name` appears first in the file's `HostProperties` tags (not a fixed priority order independent of the file's own tag order, though `host-fqdn` conventionally comes first in real Nessus exports). IP is read from `host-ip`; if that's missing, the `ReportHost` name attribute is used as the IP only if it actually parses as valid IPv4, otherwise it becomes the hostname instead.

## Nmap parsing notes

On a dual-stack host, IPv4 is always preferred over IPv6 regardless of which `<address>` element appears first in the XML; an IPv6-only host keeps its IPv6 address. A host with no `<port>` elements at all (scanned but nothing open or listed) is still represented as a synthesized host-level row rather than disappearing from the results entirely.

## Keying and merging

Rows are deduplicated and merged using a small set of identity keys, all derived per-row rather than per-file:

- **Row identity** is `ip` (or, if the row has no IP, `hn:` + hostname, so distinct hostname-only hosts don't collapse onto a shared empty-IP key).
- **Same-source dedup** (two Nessus files, or two Nmap files, describing the same row) keys on `identity + protocol + port + source`. On a conflict, the row reporting `state=open` wins over one that doesn't, so a re-scan that confirms a port open isn't discarded just because an older, less complete file loaded first. This applies regardless of which filenames the duplicate rows came from — uploading identical content under two different names still produces one row, not two.
- **Cross-source merge** (Combined view) keys on `identity + hostname (case-insensitive) + protocol + port + state`. Two rows that agree on all of that collapse into one `both`-badged row with a combined service name. Hostname is part of the key: if Nessus and Nmap report two different non-empty hostnames for the same host/port/state, the rows are **not** merged — both stay visible as separate rows so neither name is silently dropped. An empty hostname from one side does not backfill from the other; it's just treated as its own distinct key value.

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

GNU General Public License v3.0.

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version. It is distributed in the hope that it will be useful, but without any warranty, without even the implied warranty of merchantability or fitness for a particular purpose. Full text: <https://www.gnu.org/licenses/gpl-3.0.html>

---

Made by [tunnelcat](https://github.com/tunnelcat)
