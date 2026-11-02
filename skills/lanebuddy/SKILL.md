---
name: lanebuddy
description: Read and configure LaneBuddy, the DNS filter, program watch and home-network watch on this computer, through its lane-mcp tools. Use when the user asks how their system is doing, what is running, what was blocked, which devices are on the network, why a site or program is blocked, or asks to block a device, address or domain, stop a program, or change a LaneBuddy setting. Also for /lanebuddy <question>.
---

# LaneBuddy

LaneBuddy runs on this computer. It answers the computer's DNS through block lists, watches every program start and stop, keeps denied programs from running, and (Max tier) watches the home network. Its tools come from the `lanebuddy` MCP server (`lane-mcp`), one group per dashboard page. The full tool table is in [reference/tools.md](reference/tools.md); the manual is a tool call away: `lanebuddy_help` with a chapter number, a page name or a tool name.

## Always

1. **Start with `lanebuddy_status`.** It is the one-screen answer and it tells you the tier. If it fails, say whether LaneBuddy is not running or not reachable, in the tool's own words, and point to `lanebuddy_help topic=troubleshooting`. Never guess numbers.
2. **Mind the tier.** A tool for a page above the install's tier answers "403 max" or "403 pro". Say which tier the page needs and move on; do not retry.
3. **Ask once, then confirm.** Every tool that blocks, stops, denies, deletes or sends something out takes `confirm=true`. Call it without `confirm` first: the preview says exactly what would happen. Show that to the user in one line, get a yes, then call again with `confirm=true`. After it, repeat the undo line the tool gives.
4. **Read before you set.** For a setting, call the `_get` tool, state the change in one line, confirm, set, then read again and show the result.
5. **Never** set a VirusTotal key, a tunnel key, or the tier unless the user named the tier. Never open a Lanny connection. The tools do not offer these; do not look for other ways.
6. **Answer short.** Numbers in a small table, program paths shortened to the file name, names as the tools give them, in the user's language.

## What the user says, and what to call

**"How is my system doing", "status", "/lanebuddy"** → `lanebuddy_status`, then `lanebuddy_findings_list` with `severity` in mind (alerts only), then, on Max, nothing more: status already carries the new devices. Answer in five lines: protection strength and what is on; DNS answered and blocked today and the top list that blocked; programs running, any denied; open alerts by kind with the one worth acting on; new devices. End with one suggested next step if an alert needs one.

**"What is on my network", "devices", "who is on the LAN"** → `lanebuddy_devices_list`. A table: name (or vendor when unnamed), address, hardware address, known or new, last seen, blocked. Devices behind a virtual adapter (Hyper-V, WSL, Docker) are inside this computer: list them apart or leave them out. During the learning phase say so: nothing is new yet.

**"What is X doing", "why does X connect to Y"** → `lanebuddy_program_detail` (path from `lanebuddy_processes` or `lanebuddy_programs_list`), then `lanebuddy_program_connections`. Name the busiest hosts, the countries, and whether anything was blocked.

**"Why was Y blocked", "site Y does not work"** → `lanebuddy_dns_log name=Y` (the exact host name; `lanebuddy_dns_suggest q=` finds it) and `lanebuddy_name_detail name=Y`. Say which list blocked it. If the user wants it through: `lanebuddy_domain_allow` with confirm; an allow wins over every list.

**"What was blocked today", "requests"** → `lanebuddy_dns_summary`, then `lanebuddy_dns_names result=blocked`.

**"What is running", "what uses the CPU"** → `lanebuddy_processes` (`limit` 10).

**"Block the iPad", "block 192.168.1.23", "block ads.example.com", "block aa:bb:cc:dd:ee:ff"** → `lanebuddy_block_add target=<what the user said>` without confirm. The tool works out the kind: a hardware address or a device name blocks the device wherever DHCP puts it; an address or subnet blocks it for every program; a name no device carries is a domain rule. If it refuses because a name is both a device and a domain, ask which and pass `kind`. Show the preview, confirm once, call with `confirm=true`, repeat the undo line. If LaneBuddy refuses (this computer's own or the router's hardware address, a home subnet), say so as the tool says it; that is LaneBuddy protecting the user.

**"Unblock X"** → `lanebuddy_block_remove target=X`, same flow. `lanebuddy_blocks_list` shows what is blocked.

**"Stop X", "close X", "kill X"** → `lanebuddy_program_stop path=<path>`. Prefer `mode=quit`; use `end` when the user says kill or when quit answered `no_window`; use `end_and_block` only when the user says "and never again". A `critical` refusal is LaneBuddy's protection of the system, not an error.

**"Never let X start", "allow X again"** → `lanebuddy_program_rule action=deny_start` / `allow_start`.

**"Block country Z"** → `lanebuddy_geo_countries` for the code, then `lanebuddy_block_country cc=.. direction=out` (or `both` when the user wants nothing from there either).

**"Set this up for my child", "family filter"** → `lanebuddy_protection_get`, then propose strength `pro` with threats, adult, gambling, shorteners and DNS-over-HTTPS on; confirm; `lanebuddy_protection_set` with only what changes.

**"Is my router safe", "ARP"** → `lanebuddy_arp_table`; if the gateway is not pinned, offer `lanebuddy_arp_pin`.

**"Open ports", "what listens"** → `lanebuddy_ports`; name the ports open on every network and their programs.

**"Any virus", "scan"** → `lanebuddy_av_status` and `lanebuddy_vt_status`; `lanebuddy_av_run what=scan` starts a pass (confirm).

**"Is something injected", "what runs inside program X", "threads"** → `lanebuddy_threads_overview` for the remote threads and the checks' alerts, `lanebuddy_threads_programs` for every program's shield, `lanebuddy_threads_program pid=N` for one program's threads, modules and memory. `lanebuddy_threads_scan pid=N` checks one again now. Suspending a thread, the protection switches and hardening (`lanebuddy_threads_thread`, `lanebuddy_threads_settings`, `lanebuddy_threads_harden`) change the computer: preview first, and only on the user's word.

**"Trace to X"** → `lanebuddy_trace target=X` (waits for the hops).

**"How hot", "sensors"** → `lanebuddy_sensors`; the hot ones are marked.

**"Explain Z", "what does setting Z do", "how do I ..."** → `lanebuddy_help topic=Z` (a page name, a tool name, or a chapter number), and answer from the chapter.

## Demonstrating LaneBuddy

For a demo of blocking: `lanebuddy_devices_list` first, pick a device the user names, show its address and hardware address, confirm, `lanebuddy_block_add` with `confirm=true`, then `lanebuddy_blocks_list` to show it listed, then `lanebuddy_block_remove` to put it back. Say at the end that the Devices page shows the same.

## Install

```bash
claude mcp add --transport stdio lanebuddy -- lane-mcp     # Claude Code
npx skills add KurtStevenK/lanebuddy-ai --skill lanebuddy           # this skill, into Claude Code, Codex, Cursor and others
```

Codex: `[mcp_servers.lanebuddy] command = "lane-mcp"` in `config.toml`. Cursor: `{ "mcpServers": { "lanebuddy": { "command": "lane-mcp" } } }`; the `/lanebuddy` command comes from [commands/lanebuddy.md](commands/lanebuddy.md). `lane-mcp` ships beside LaneBuddy and finds the running engine by itself; `lane-mcp --check` says what it found.
