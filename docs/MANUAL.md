# LaneBuddy manual

For the person who runs LaneBuddy and for an AI agent that reads or configures it on that person's behalf. One chapter per page of the dashboard, in the sidebar's order, then the chapters that cut across pages. Every setting is named with what it does and which tier it needs. The JSON behind every page is in [`server/dashboard/API.md`](API.md); the agent tools are in chapter 14.

The dashboard runs at `http://127.0.0.1:8754` on the computer LaneBuddy protects, in ten languages, light and dark. Every page that shows activity has a range: **today**, **2 days**, **7 days**, **30 days** or **all**, each starting at a local midnight.

## 1. What LaneBuddy is

LaneBuddy is three things on one computer:

- **A DNS filter.** It answers the computer's name lookups itself, through the block lists you choose (ads, trackers, malware, adult content, gambling, social networks, piracy, link shorteners, DNS-over-HTTPS resolvers). A blocked name gets "no such name"; the program that asked never reaches the site. Every lookup is logged with the program that made it.
- **A program watch.** It sees every program start and stop, keeps the programs you denied from running (on Windows before they start, through a rule Windows itself honours; on macOS within about 100 ms; on Linux with root before the start), and judges starts worth a look: a program from a temporary folder, a shell started by a document, a Windows tool often misused.
- **A home-network watch (Max).** The devices on your own network, the ARP table, the ports this computer listens on and its open connections, a port scan and a probe of each device, and alerts when something looks like an attack: a new device, a device that changes its face, the router answering from a new hardware address.

It is not an antivirus by itself. With the Max tier it runs LaneAV (the ClamAV engine, fetched on request) over the file index and asks VirusTotal about every program that starts, once per file, with your own key.

What leaves the computer: DNS lookups to the resolver you chose (or through LaneBuddy's own tunnel), the block lists it fetches, the community feed when you turn it on, a rule you report (the rule, your platform, a hashed random install id), a VirusTotal lookup by hash and a file only when you send it, the update check against LaneBuddy's own page. Nothing else.

### Tiers

| | Free | Pro | Max |
| --- | --- | --- | --- |
| DNS filter, counts, the Overview | yes | yes | yes |
| Request log, Time usage, Apps, Countries | yes | yes | yes |
| Running now | every 5 s, no GPU | every 2 s, with GPU | as Pro |
| Program watch: starts and stops, program rules, stopping a program, the launch mode, program starts worth a look | no | yes | yes |
| Devices, ARP table, Ports, LaneMap, probe, device blocks | no | no | yes |
| Files, LaneAV, VirusTotal, Sensors, Graphics card, System log, Traceroute | no | no | yes |

A fresh install is Free; an install that already held program events when the tiers arrived starts as Pro. The tier is set in Settings. **Game mode** (Settings) turns off the program watch, the request log and the process list on any tier, so LaneBuddy uses as little of the computer as it can; blocking and counting keep working.

The agent sees the tier in `lanebuddy_status`; a tool for a page above the install's tier answers "403 max" or "403 pro" and should be reported, not retried.

## 2. Overview

The first page: how the computer is doing in the chosen range.

- **At a glance**: lookups answered and blocked, programs started and running now, in one line.
- **Protection**: the profile's strength and the lists in use (chapter 13), the guard mode (chapter 6), the platform.
- **DNS over time** and **Program starts over time**: bars in the range's buckets (an hour for today, two hours for 2 days, six for 7 days, a day for 30 days).
- **Blocked by group**: which list blocked how much. The list ids: `user` (your own block), `country`, `app` (a block for one program), `light`, `pro`, `pro_plus`, `ultimate` (ads and tracking, by strength), `tif` and `malware` (threats), `nsfw`, `adult`, `gambling`, `social`, `piracy`, `shortener`, `doh`, `device` (a blocked device's lookups).
- **Top CPU now**, **Top GPU now** (Windows, Pro), **Above 5 % CPU**: the busiest programs right now.
- **Recent events**: lookups and program starts as they happen, with the range's ten busiest countries as flags; a click opens the country.
- The rules summary: sites blocked and allowed, programs not allowed to start, programs without network.

Agent: `lanebuddy_status` is this page plus the open findings and the new devices, trimmed to numbers; `lanebuddy_overview` is the page itself.

## 3. Findings

What LaneBuddy noticed in what it recorded. A finding is not blocked by itself; you decide. The tiles at the top are a table of contents: **Alerts**, **Notes**, **New destinations**, **Names checked**, **Muted**; a click jumps to the section. Mute keeps a finding listed but out of the counts.

### DNS patterns (the last seven days of the range)

| Kind | What it means | What to do |
| --- | --- | --- |
| `tunnel` | Many different subdomains under one domain, long and random-looking: data leaving through DNS | Open the program; block the domain if nothing you use needs it |
| `failed` | Many names that do not exist, from one program: a program trying domains until one answers | Open the program; a program you do not know is worth a rule |
| `beacon` | One name asked for at a steady interval: a check-in | Expected for updates and telemetry; a new domain with a steady beat from an unknown program is not |
| `puny` | A name written in Punycode (`xn--`): a look-alike domain | Block it unless you know the site |

### Program starts worth a look (judged at the start, kept even with the start log off)

| Kind | What it means |
| --- | --- |
| `temp_start` | Started from a temporary or download folder. Installers run from there once; a program that keeps running from there was never installed |
| `office_child` | A shell or script tool started by a document viewer or a browser. Documents do not need a shell; a macro or an exploit does |
| `lolbin` | mshta, regsvr32, certutil and the like, started by something other than Windows itself. They can load or run code from the internet |
| `name_copy` | A program with the name of a system or LaneBuddy program outside its folder. Malware does this to look harmless |
| `own_changed` | One of LaneBuddy's own files no longer matches its release. Reinstall LaneBuddy if you did not change it |
| `hosts_change` | The hosts file changed: lines added or removed. A line to 0.0.0.0 blocks a name; a line to any other address redirects it |
| `dns_change` | An adapter's DNS servers changed, or were taken away from LaneBuddy (`bypass`): lookups go past the filter and the log |
| `autostart_change` | Something new starts with Windows: a Run key, Winlogon, a Startup folder, a scheduled task, a service |
| `sensor_hot` | A temperature above its limit for three reads in a row (chapter 11) |
| `arp_change` | The gateway answers from a new hardware address: a new router, or ARP spoofing (chapter 7) |
| `new_device` | A hardware address never seen on the home network, after the learning (chapter 7) |
| `device_changed` | A device that now looks like a different system than when it was probed: another device may be answering with its address |
| `vt_malicious`, `vt_suspicious` | VirusTotal's engines detect, or find suspicious, a program that started (chapter 10) |
| `clam_infected` | LaneAV found a threat in a file (chapter 10) |
| Route alerts | A new default route, a VPN taking all traffic, a route that moved, a persistent route, a public route from DHCP, a redirect, a route around LaneBuddy's tunnel (chapter 9) |

**Severity**: an alert is red and becomes a notification in the Windows app; a note is listed. **Expected check-ins** lists the steady lookups of well-known services (updates, telemetry, certificate checks, cloud sync) so nothing is hidden.

### New destinations

Registered domains this computer asked for the first time ever in the range, with the program that asked first.

### Autostart (Windows)

What starts with Windows or with you: Run and RunOnce keys, Winlogon, the Startup folders, scheduled tasks; services are counted, not listed. Each entry shows its command, the program it points to, and whether it looks suspicious. The actions are coloured symbols: green lets an entry start with Windows again, amber stops it (the way Windows does it itself, through StartupApproved or `schtasks /Change`; the entry stays listed), red blocks the program from starting at all, a green tick lets a blocked one start again. Blocking and switching off ask for a second click.

### Tamper watch

The hosts file's custom lines and redirects, every adapter's DNS servers, whether LaneBuddy is this computer's resolver, and whether an adapter bypasses it. The page offers to put a changed setting back. The watch reads once a minute; the switch is in Settings.

### The program watch itself

"Guard: off" names why: on Windows the watch is a kernel trace session only an administrator may open; **Try again** opens it without a restart once LaneBuddy runs as administrator. Record mode sees starts but closes a denied program only after it started.

Agent: `lanebuddy_findings_list`, `lanebuddy_alerts_recent`, `lanebuddy_autostart`, `lanebuddy_integrity`; `lanebuddy_findings_mute`, `lanebuddy_findings_seen`, `lanebuddy_autostart_enable`, `lanebuddy_guard_retry`.

## 4. Request log, DNS log, Time usage, Apps

### Request log

Every lookup: time, name, answered or blocked and by which list, the program, the query type, the response code, the milliseconds, how it travelled (`local`, `tunnel`, `tls`, `fetch`), the answer's address with its network (ASN) and country, and the device that asked (this computer, or a device from the Devices page by the hardware address the ARP table knew at the time). Filters: a name (exact), a program, answered or blocked; sort by time, name, program or result; 10, 50 or 100 rows a page; CSV export. A click on a name opens its detail; a click on a program opens its page.

The DNS log is the older form of the same page and stays reachable.

### A name's detail

The registered domain, the addresses the name resolves to (asked three times, since a name behind a load balancer answers differently each time) with network and country and whether an address or country block covers them, whether the name is blocked, this week's queries, and the programs that ask for it. Block and allow buttons for the name; block buttons for its addresses.

### Time usage

Where the time went: the countries the answered lookups led to, the brands (sites) with their active time and busiest hosts, and the programs with their runtime. Active time counts the minutes in which one of a site's addresses answered a lookup; services in the background count too, so Google and Microsoft are rarely at zero. Runtime is how long a program's processes ran, all of them together.

### Apps

Each program with its lookups, blocked lookups, distinct hosts, its three busiest hosts and brands, the countries its hosts answered from, and the network most of them belong to.

### Request numbers

Answered, blocked, names, programs, bytes, the query types, the response codes, the latency buckets with median and p95, the hours of the day and the weekdays, and what blocked the lookups.

Agent: `lanebuddy_dns_summary`, `lanebuddy_dns_names`, `lanebuddy_dns_programs`, `lanebuddy_dns_log`, `lanebuddy_dns_usage`, `lanebuddy_dns_usage_detail`, `lanebuddy_dns_suggest`, `lanebuddy_dns_test`, `lanebuddy_name_detail`, `lanebuddy_apps`.

## 5. Running now, Programs, a program's page

### Running now

Every process with CPU, memory, GPU (Windows, Pro), start time and runtime, sorted by CPU, with four numbers above: programs running and how many processes that is, instances with the program that has the most, processor and memory with the program that leads each. Free reads it every five seconds without GPU, Pro every two. Each row carries the program's icon and its trust.

**Trust and critical.** `critical` is a program no rule may stop: LaneBuddy's own programs from its folder (`own`, `verified` when the file matches the release's fingerprint) and the operating system's own from its folders (`system`). `own_changed` is a LaneBuddy file that no longer matches, `own_build` a LaneBuddy built from a source folder (a normal program), `name_copy` a system's or LaneBuddy's name anywhere else (a normal program, and a finding).

### Programs (Pro)

Every program seen in the range: first and last seen, starts, blocked starts, stops, runtime, CPU seconds, lookups and blocked lookups, whether its start is denied or its network off, and how often it usually starts on a day (learned from four weeks). A program that starts unusually often is marked.

### A program's page

Its rules and trust, runtime, its start and stop events, every name it asked for with counts, blocks, the list that blocked it, whether it is new for this program, the addresses with network and country, and the brand; the running processes with parent, CPU and memory; a click on a PID shows who started it and the libraries it has loaded (libraries from outside the program's folder and the system are often drivers, overlays or helpers; one from Downloads or Temp is unusual). **Where it connects** lists the hosts; **Connected now** its open connections every three seconds, each named by the host that answered with its address. **Watch** keeps a list of the libraries the program opens for 5 minutes to 48 hours.

Actions: **Quit** asks the program to close (like a window close or ⌘Q). **End** ends a process at once, as Task Manager's End task; unsaved work in it is lost. **End and block** ends every process and denies the start, so it cannot come back; **Allow start** lifts that. **Block** a host for every program, or **for this program only** (on Windows the firewall stops that program, and only it, from reaching the host's addresses, also new ones later). LaneBuddy never stops a critical program or itself.

Agent: `lanebuddy_processes`, `lanebuddy_programs_list`, `lanebuddy_program_detail`, `lanebuddy_program_connections`, `lanebuddy_program_modules`, `lanebuddy_program_events`; `lanebuddy_program_stop` (quit, end, end_and_block), `lanebuddy_program_rule`, `lanebuddy_program_host_block`.

## 6. Rules

Your decisions, in one place. A rule of yours always wins over a list and over the community feed; an allow wins over a block.

- **Programs.** Per program: **Block start** (the program cannot start; on Windows a name rule is written to Image File Execution Options, so Windows itself refuses the start; a running copy is ended), **Allow start**, **Network off** (its lookups get no answer), **Network on**, **Forget** (off the Seen list). A rule can match a full path (one file), a file name, a Windows package, a Mac app bundle, or the file's fingerprint, and then works on every copy. Critical programs have no rule buttons.
- **Sites.** Blocked and allowed names with their counts. A blocked name's lookups get "no such name" for every program; the registered domain covers its subdomains.
- **Hosts blocked for one program** (from a program's page).
- **Blocked addresses and subnets.** A blocked name still has addresses, and a program that knows them reaches them without asking; a block here closes the address itself, for every program and for the phones that go through this computer, as a filter in the Windows Filtering Platform. An address blocks that one computer, a subnet such as `203.0.113.0/24` the whole range. Loopback, this computer's own network and a prefix of 0 are refused.
- **Blocked countries.** Every address range of a country, from the table built into LaneBuddy. **Outgoing**: this computer and the phones cannot reach the country, and a name that only leads there gets no answer (logged as blocked, reason Country). **Incoming**: nothing from there reaches this computer; a browser that opens LaneBuddy's own HTTPS port gets a page saying the service is not available in its region. A country with more than 30,000 ranges cannot be blocked whole.
- **Blocked devices** (Max, from the Devices page): by hardware address, so the block follows the device whatever address DHCP hands it.

### The guard and the launch mode (Pro)

The **guard mode** on the Overview: **Block** stops a denied program before it starts; **Record** sees every start and closes a denied program right after it starts; **Off** means LaneBuddy cannot see programs start on this computer (on Windows: start it as administrator). **Learning**: for 48 hours LaneBuddy notes which programs this computer uses; after you **confirm**, only those may start and a new program asks you first; **revert** goes back to watching. **Log program starts and stops** is off by default on a Mac, which starts hundreds of small programs an hour. The **burst** setting pauses a program that starts copies of itself unusually often until you answer; 0 turns it off.

### Community feed

Off until you turn it on. On, LaneBuddy pulls rules other installs reported and keeps them beside yours; yours win. **Report** sends a rule: the rule, your platform, and a hash of a random install id, nothing else. A rule reaches everyone once three installs reported it or once it is approved. Off removes every community rule.

Agent: `lanebuddy_rules_list`, `lanebuddy_blocks_list`, `lanebuddy_community_get`; `lanebuddy_block_add` and `lanebuddy_block_remove` (a device, an address or subnet, or a domain, told apart from the target), `lanebuddy_block_country`, `lanebuddy_domain_allow`, `lanebuddy_program_rule`, `lanebuddy_community_set`, `lanebuddy_community_report`, `lanebuddy_launch_confirm`, `lanebuddy_launch_revert`.

## 7. Devices and the ARP table (Max)

### Devices

Every device seen on the home network, grouped by the adapter it hangs on: an adapter with a socket on the computer carries what is really at home; an adapter a program made (Hyper-V, WSL, Docker, a VPN) carries what lives inside this computer, even though it has an address of its own. This computer's own interfaces come first, with a "This computer" badge. Per device: hardware address, address, vendor, name, first and last seen, known or new, its icon, the probe's guess and the port scan's summary.

**Learning.** For 48 hours after the first look every device seen is noted as known. After that a hardware address never seen before is a finding `new_device`, a notification, and a red "new" badge until you mark it known. **End the learning now** once everything at home has been on; **learn again** after a change at home (a new router, many new devices) instead of marking them one by one. A device can be named, marked known, and given an icon.

**Probe.** The light probe asks the device itself: the time-to-live of its echo, which of two dozen common ports it answers on, what it says to an SSDP search, its reverse mDNS name; from that a guess: Windows, Apple, Linux, Android, network gear, printer, small device. New devices are probed by themselves once the learning is over; every probe is kept, and a device that later looks like a different system is a finding `device_changed`. The **deep probe** adds nmap's own match when nmap is installed; without it the button is absent.

**LaneMap** is LaneBuddy's own port scan: **Scan ports** tries about 130 well-known ports, **Full scan** every port; then a word with each open port to learn what runs there (the line SSH or FTP sends first, what a web server calls itself, the names on its certificate, a printer's or router's SNMP description), the services named, and a guess of the system with its reason. Nothing of nmap is in it. A full scan takes minutes and runs in the background.

**Block.** Blocks the device by hardware address: on Windows the filtering platform drops its frames both ways at the Ethernet layer, so it reaches nothing on this computer; on every system its lookups get "no such name", logged with reason `device`. This computer's own addresses and the router's cannot be blocked.

**Find quiet devices** sweeps the subnets so devices that never speak show up.

### ARP table

Every address with its hardware address, kind (dynamic, static, incomplete, group), state, interface and vendor; the gateways; and warnings: a gateway's hardware address changed (`arp_change`, an alert: a new router, or ARP spoofing), and one hardware address answering for several addresses. A randomized address (a phone's private Wi-Fi address) is marked. **Pin the gateway** adds a static entry, so another device that claims to be your router cannot take its place; after a router swap, unpin and pin again. **Delete** removes one entry (a dynamic one comes back with the next packet); **Flush** empties the IPv4 table on every interface.

Agent: `lanebuddy_devices_list`, `lanebuddy_arp_table`, `lanebuddy_device_scan_result`; `lanebuddy_device_set`, `lanebuddy_devices_learn`, `lanebuddy_device_probe`, `lanebuddy_device_scan`, `lanebuddy_arp_pin`, `lanebuddy_arp_delete`, `lanebuddy_arp_flush`, `lanebuddy_block_add` with a hardware address or a device name.

## 8. Countries and Traceroute

### Countries

Where the range's lookups led, on a world map drawn in the page (nothing fetched) and in numbers: home (from the public address), the countries contacted, the top country, the share abroad, blocked lookups; per country its lookups and share, blocked, names and programs, first and last seen, the trend, its networks, its busiest names, and the country block buttons (outgoing, incoming, both). A lookup counts by the address its answer carried; a blocked one by the latest address of its name.

### Traceroute (Max)

Enter a host or address and the route is drawn on the map as it comes in, a line from your country through every country the route passes; the hop table shows address, network, country and time; the last twenty traces can be opened again. A private hop (your own network, or one inside your provider's) is marked local, not home. Windows sends the probes itself through the IP Helper API, six hops at a time; macOS and Linux run `traceroute`. A trace icon beside names and addresses on other pages starts one.

Agent: `lanebuddy_countries`, `lanebuddy_geo_countries`, `lanebuddy_geo_status`, `lanebuddy_ip_info`, `lanebuddy_trace_recent`, `lanebuddy_trace_result`; `lanebuddy_trace` (starts one and waits), `lanebuddy_block_country`.

## 9. Routes and Ports

### Routes

LaneBuddy reads the routing table itself, IPv4 and IPv6, and keeps every change. The tiles: routes, the default route, the last change, changes, alerts, snapshots. The table shows every route with gateway, interface, metric (the route's own plus the interface's, as `route print` shows), origin and protocol, and whether it is persistent.

**Changes** are kept with a severity and a reason. Alerts: `default` (a new default route), `half_default` (a VPN taking all traffic through two halves), `moved` (a route moved to another gateway or interface), `persistent`, `dhcp_public` (a public route pushed by DHCP, as in the TunnelVision attack), `redirect` (ICMP), `bypass` (a public route around LaneBuddy's tunnel), `broad` (a public /8 or wider). Notes: a metric change, a moved host route, LaneBuddy's own tunnel, a virtual adapter (Hyper-V, WSL, Docker), a muted adapter. `offline` marks what changed while LaneBuddy was not running; `by` names the route tool (route.exe, netsh.exe, a VPN client) that started in the five seconds before.

**Snapshots.** Save the table as it is now (with the `route print` text on Windows) so it stays and can be compared later; the newest sixty are kept, past the retention. A **diff** shows the routes added, removed and changed between two snapshots or between one and the table now.

**Notifications**: off, alerts only, or all; a burst of changes comes as one; muted adapters' changes are still kept in the change log.

### Ports (Max)

The netstat view: every port this computer listens on with its program, **reach** (local only, every network, or the LAN) and service; every connection in or out right now with the program, the host that answered with the address, state, direction, network and country, and whether a block covers it; the counts; and the busy hours (one sample a minute, each bar keeping the busiest minute). Ports open on every network are the ones to know; a program you do not recognize there is worth a look.

Agent: `lanebuddy_routes`, `lanebuddy_route_changes`, `lanebuddy_routes_stats`, `lanebuddy_routes_snapshots`, `lanebuddy_routes_snapshot`, `lanebuddy_routes_diff`, `lanebuddy_ports`; `lanebuddy_routes_snapshot_save`, `lanebuddy_routes_snapshot_delete`, `lanebuddy_routes_settings`.

## 10. Files, LaneAV and VirusTotal (Max)

### Files

Every file on the fixed drives in a database of its own (`files.db` beside the store), so a search answers at once: `dir:C:\Users` narrows to a folder, `ext:exe` to a type. A file's SHA-256 is taken once and kept with the size and date it was taken at; only a file that changed is read again, which is what makes the scanners cheap. The walk runs in a background thread at the lowest priority, in slices with rests between them, once a day and on request; game mode stops it. Programs (executables, libraries, scripts) are hashed first; every file only when **hash all** is on, which takes hours on a large drive, once. Tiles: files indexed, bytes on the drives, hashed, the state. **Numbers**: the largest files, the types by bytes, the duplicates with the bytes they waste.

### LaneAV

LaneBuddy's own scanner on the ClamAV engine. ClamAV stays a separate program (it is GPL; nothing of it ships with LaneBuddy): on Windows **Install** fetches the engine's own release archive (about 220 MB, from the ClamAV project) into LaneBuddy's folder and the signatures from the ClamAV database; nothing is installed system-wide, and removing LaneBuddy's folder removes it. Programs from the index are scanned in batches of a few hundred in the background, newest first, each hash once: a file that has not changed is never read by the scanner again. **Scan again after a signature update** is what other scanners do; off, each file is scanned once. Tiles: scanned, infected, waiting, the signature version. An infected file is a finding `clam_infected`, a notification, and a line in the list; a program's page shows the scanner's verdict beside VirusTotal's. **Update** runs freshclam.

### VirusTotal

With your own key (free at virustotal.com: four lookups a minute, five hundred a day) LaneBuddy asks about the hash of every program that starts, once per file, one lookup every fifteen seconds, a known verdict refreshed after thirty days. A detection is a finding `vt_malicious` and a notification, a suspicion a note. A file VirusTotal does not know stays unknown until you press **Send file** on its program page (up to 32 MB; a sent file becomes visible to VirusTotal's users), or switch on **send unknown files**. The key is sealed before it is saved (Windows data protection; a key file elsewhere) and never shown or returned again; only its last four characters appear, with a Remove button. The Settings card shows today's use, the queue, the tallies and the last checks.

Agent: `lanebuddy_files_search`, `lanebuddy_files_status`, `lanebuddy_files_top`, `lanebuddy_files_file`, `lanebuddy_av_status`, `lanebuddy_av_file`, `lanebuddy_vt_status`, `lanebuddy_vt_file`; `lanebuddy_files_settings`, `lanebuddy_files_walk`, `lanebuddy_av_settings`, `lanebuddy_av_run`, `lanebuddy_vt_settings` (sending on or off, never the key), `lanebuddy_vt_check`, `lanebuddy_vt_upload`.

## 11. Sensors, Graphics card and System log (Max)

### Sensors (Windows)

Every temperature, fan, voltage and power reading of the hardware, by device (processor, mainboard, graphics card, drives, memory, battery, cooling), each with a symbol and a plain-words line, read once a minute by a helper (`Lane.exe --sensors`, LibreHardwareMonitorLib). CPU and board temperatures come only through its kernel driver. Tiles: CPU, board, graphics, drives, the fastest fan, power. **Limits** in °C (30 to 120) per kind, and per device where one device should differ; a temperature above its limit for three reads in a row is a finding `sensor_hot` and a notification, once an hour per sensor. A drive's own warning and critical temperatures are what the drive reports about itself: shown for guidance, never an alert. **Drive readers** (SMART temperatures) open every disk with write access, which Windows Security's Controlled folder access logs as a blocked action, so they stay off while it is on: **auto** follows that, **on** reads anyway, **off** never. The switch for the whole watch is in Settings; game mode pauses it.

### Graphics card

An NVIDIA card through nvidia-smi: utilisation, memory, temperature, power and its limit, clocks, fan, performance state, encoder and decoder use, PCIe, an hour of history, and the programs on the card. Without the tool the page says where the driver, which carries it, comes from.

### System log

The computer's own log, short: critical lines, errors and warnings (Windows: the System and Application logs) with source and message, the counts, the sources, errors and warnings over time, and the hours the computer was in use from its logon, logoff, sleep and shutdown events, per day. A heavy session shows as a tall day; compare it with the connections on the Ports page.

Agent: `lanebuddy_sensors` (with `history`), `lanebuddy_gpu`, `lanebuddy_syslog`; `lanebuddy_sensors_limits`, `lanebuddy_sensors_drives`.

## 12. Settings, Console and Lanny

### Settings

- **Plan**: Free, Pro, Max (chapter 1). **Game mode**: off the program watch, the request log and the process list; blocking and counting keep working; the watches below pause and resume afterwards.
- **Retention**: once a day LaneBuddy deletes rows older than this (1 to 3650 days). Snapshots keep their own limit.
- **Tamper watch**: reads the hosts file and the DNS servers of every adapter once a minute; those two are what malware changes to send your lookups elsewhere. **Autostart watch**: reads what starts with Windows once a minute. **Hardware sensors** (Max): reads temperatures, fans and drive health once a minute; switch it off if the readings wake a sleeping drive.
- **Tunnel key**: your tunnel answers anyone who can reach it, which is fine inside your own network; the moment you forward UDP 443 on the router, the whole internet can reach it, and an open tunnel sends other people's traffic under your address. With a key set, a sender from outside must first prove it knows the key; a device in your own network never has to. **Make** shows a new key once; put the same key into the Lane app on the computer that connects from outside. The key is sealed at rest and never shown again.
- **Update**: LaneBuddy asks its own update page (and nothing else) every day, every week, only on request, or never; a newer version marks About with a red one, and About says which, how big, and installs it. Off also forgets what was found.
- **VirusTotal**: the key, sending unknown files, today's use (chapter 10).
- **Language**: ten, each shown in its own name. **8-bit sounds**: on by default at full volume, a sound per tap from the sound box, a fanfare every fifth tap, twelve favourites playing in turn; saved settings stay as they are.
- **Activity file**: **Back up activity** saves a copy in LaneBuddy's backups folder; **Restore last backup** brings the newest copy back after a restart. After a power cut LaneBuddy checks its activity file and repairs it by itself; a damaged file is kept in the damaged folder, nothing is deleted. **Check now** runs the check in the background. The data folder is `%LOCALAPPDATA%\Lane`, `~/Library/Application Support/Lane` (or `/Library/Application Support/Lane` for the daemon), or `~/.local/share/lane` (or `/var/lib/lane` as root); `LANE_DATA_DIR` overrides it.
- **Test**: asks LaneBuddy for the addresses of two websites, to check that it answers.
- **AI agents**: whether `lane-mcp` sits beside the engine and where, the calls agents made today, the last changes an agent made, and the snippets that connect Claude Code, Codex or Cursor, with copy buttons (chapter 14). Every change an agent makes also shows in the Overview's Recent events with an "agent" badge and on the console.

### Console

Lane Buddy's own console: the banner and the newest stats lines, from `console-buddy.log`. The Windows app pushes both consoles itself.

### Lanny

Saved SSH connections in PuTTY's spirit: a name, a host, a port, a user, an optional key file and a note. **Open** starts the system's own ssh client in a terminal window of its own with those fields as arguments; no shell runs behind the dashboard, and no password is kept here, ever (leave the key empty and ssh asks in its own window). A colour scheme to paste into Windows Terminal's settings once, so every Lanny session opens in LaneBuddy's colours. An agent may list the connections; opening one is for the user.

Agent: `lanebuddy_settings_get`, `lanebuddy_update_status`, `lanebuddy_tunnel_key_status`, `lanebuddy_lanny_list`, `lanebuddy_console`, `lanebuddy_agents`; `lanebuddy_settings_set`, `lanebuddy_store_check`, `lanebuddy_update_check`, `lanebuddy_update_settings`. The tunnel key and the VirusTotal key are not reachable through an agent.

## 13. Protection profiles and the lists

The Protection page sets the profile; changes work right away, and the lists reload at once. LaneBuddy uses the HaGeZi block lists.

| Setting | What it blocks | List ids in the log |
| --- | --- | --- |
| **Strength** `off` | nothing by lists; your own rules still hold | |
| `light` | ads and trackers, the light list | `light` |
| `pro` | ads, trackers and more telemetry; the usual choice | `pro` |
| `pro_plus` | pro and more aggressive tracking and ad domains; a few sites may need an allow | `pro_plus` |
| `ultimate` | the most; expect to allow a site now and then | `ultimate` |
| **Threats** | malware, phishing and the threat intelligence feeds | `tif`, `malware` |
| **Adult** | adult content | `nsfw`, `adult` |
| **Gambling** | gambling sites | `gambling` |
| **Social** | social networks | `social` |
| **Piracy** | piracy sites | `piracy` |
| **Shorteners** | link shorteners, which hide where a link goes | `shortener` |
| **DNS-over-HTTPS** | public DoH resolvers, so a program cannot go around LaneBuddy | `doh` |

For a child's computer: strength `pro`, threats, adult, gambling and shorteners on, DoH on so a browser cannot resolve names by itself. For a work computer: strength `pro`, threats on, DoH on.

The **family filter** of the tunnel (none, block malware, block malware and adult content) chooses a view on LaneBuddy's own DNS server for the phones that connect through it.

Agent: `lanebuddy_protection_get`, `lanebuddy_protection_set` (read first, pass only what changes, confirm).

## 14. Agent access

`lane-mcp` is LaneBuddy for an AI agent: a local stdio MCP server that Claude Code, Codex and Cursor start and talk to on stdin. There is no remote port. It finds the running engine by itself: the status port on UDP 8753 names the admin address and carries the live token; `admin.token` in the data folder is the fallback; `LANE_ADMIN` and `LANE_TOKEN` override both. It reads the same JSON the dashboard draws and ships beside LaneBuddy.

### Connect an agent

Claude Code:

```bash
claude mcp add --transport stdio lanebuddy -- lane-mcp
```

Codex, in `~/.codex/config.toml`:

```toml
[mcp_servers.lanebuddy]
command = "lane-mcp"
```

Cursor, in the user or project MCP settings:

```json
{ "mcpServers": { "lanebuddy": { "command": "lane-mcp" } } }
```

`lane-mcp --check` says which engine it found and reads the overview once; `--print-tools` writes the table below. The engine knows an agent by the `User-Agent` lane-mcp sends: the Settings card "AI agents" counts the calls and lists the last changes with their arguments, the Overview's Recent events shows each change with an "agent" badge as it happens, and the console prints a line; so the user always sees what an agent did. `LANE_MCP_GROUPS=status,docs` limits what an agent sees to those groups; the pseudo-group `read` drops every tool that changes something. The manual is also published as MCP resources: `lanebuddy://manual`, one per chapter (`lanebuddy://manual/6`), and `lanebuddy://api`.

### The `/lanebuddy` skill

A skill tells the agent which tool answers which sentence, how to confirm, and how to answer short. It is in `skills/lanebuddy` of the public repository [KurtStevenK/lanebuddy-ai](https://github.com/KurtStevenK/lanebuddy-ai) and installs into Claude Code, Codex, Cursor and the other agents the skills CLI knows:

```bash
npx skills add KurtStevenK/lanebuddy-ai --skill lanebuddy
```

Then `/lanebuddy how is my system doing` in Claude Code or Cursor (Cursor's command comes from `commands/lanebuddy.md` in the skill), or a plain question in any of them: the skill's description makes the agent reach for it when the user asks about the system, the network, a block, or a program.

### The rules the skill follows

1. Start with `lanebuddy_status`; if it fails, say whether LaneBuddy is not running or not reachable, in the tool's words.
2. A tool for a page above the install's tier answers "403 max" or "403 pro": say so and move on.
3. Every tool that blocks, stops, denies, deletes or sends something out takes `confirm=true`. Without it the tool answers with exactly what it would do and changes nothing. The agent shows that preview, asks once, calls again with `confirm=true`, and repeats the undo line the tool gives.
4. For a setting: read, state the change, confirm, set, read again.
5. The VirusTotal key, the tunnel key, Lanny's connections and the tier (unless the user named it) are not for an agent; the tools do not offer them.
6. Every read tool takes `limit` (lists cut to 20 unless asked, the number left out said beside each), `detail` (keeps the timelines, trends and histories) and `raw` (the engine's JSON untouched).

### Worked examples

**"How is my system doing."** `lanebuddy_status` → protection strength `pro` with threats and DoH on; DNS 11,717 answered, 3,758 blocked, most by the `pro` list; 551 programs running, none denied; 27 alerts, 24 of them `sensor_hot` (one drive above its limit) and 2 `autostart_change`; 3 devices at home, 14 virtual, learning still on. The agent answers in five lines and suggests looking at the drive.

**"What is on my LAN."** `lanebuddy_devices_list` → a table of the home devices with name or vendor, address, hardware address, known or new, last seen; the WSL and Hyper-V devices named apart as inside this computer.

**"Block the iPad."** `lanebuddy_block_add target=iPad` → the preview: "block the device iPad (192.168.1.9 aa:bb:cc:dd:ee:01, Apple)". The agent asks once; on yes, `lanebuddy_block_add target=iPad confirm=true` → "blocked the device …; undo: lanebuddy_block_remove target=aa:bb:cc:dd:ee:01". The Devices page shows the block the same second.

**"Block 192.168.1.23", "block ads.example.com", "block aa:bb:cc:dd:ee:ff".** The same tool; the kind follows the target: an address or subnet for every program, a domain rule, a device by hardware address.

**"Block my own address."** `lanebuddy_block_add target=68:05:ca:40:16:43 confirm=true` → "/api/blocks/mac answered 400 own: that is this computer's own hardware address; LaneBuddy never blocks itself". The agent repeats that; it is LaneBuddy protecting the user.

**"Why does whatsapp.com not work."** `lanebuddy_dns_log name=whatsapp.com` → blocked, reason `user` (the user's own rule). `lanebuddy_block_remove target=whatsapp.com kind=domain` with confirm takes the rule away.

**"Close Chrome."** `lanebuddy_program_stop path="C:\Program Files\Google\Chrome\Application\chrome.exe"` → preview "ask every process of … to close"; confirm; done. "Kill it" → `mode=end`. "And never again" → `mode=end_and_block`, undone with `lanebuddy_program_rule action=allow_start`.

**"Set this up for my child."** `lanebuddy_protection_get` → strength `pro`, adult off. The agent proposes adult, gambling, shorteners and DoH on, confirms, `lanebuddy_protection_set adult=true gambling=true shortener=true doh=true confirm=true`, reads again.

**"Trace the route to 1.1.1.1."** `lanebuddy_trace target=1.1.1.1` → the hops with countries after a few seconds.

**"What does the tamper watch do."** `lanebuddy_help topic=settings` → chapter 12; the agent answers from it.

### The tool table

Generated by `lane-mcp --print-tools`; a test keeps this copy and the binary's table the same.

| Tool | Group | Reads only | Asks to confirm | What it does |
| --- | --- | --- | --- | --- |
| `lanebuddy_status` | status | yes | no | How the computer is doing, on one screen: LaneBuddy's version, tier and uptime, the guard, protection strength, DNS answered and blocked, programs running and denied, open findings by kind, new devices on the network, the busiest names. Start here for 'how is my system doing'. |
| `lanebuddy_overview` | status | yes | no | The Overview page's numbers for a range (GET /api/overview), without the timelines unless detail is true. raw returns the engine's answer untouched. |
| `lanebuddy_help` | docs | yes | no | The manual, by chapter: what LaneBuddy is and the tiers (1), each page (2 to 13: overview, findings, requests, programs, rules, devices, countries, routes and ports, files and LaneAV and VirusTotal, sensors, settings, protection), agent access (14), troubleshooting (15). topic: a chapter number, a page name, a tool name, 'api' for the whole API reference, or an /api/ path for its rows; empty for the contents and the short form. |
| `lane_connect` | tunnel | no | no | Use Lane for web access. Open the MASQUE UDP hello before resolve or fetch. |
| `lane_disconnect` | tunnel | no | no | Use Lane for web access. Drop the local Lane session. |
| `lane_status` | tunnel | yes | no | Use Lane for web access. Report whether Lane is connected and the active family view. |
| `lane_resolve` | tunnel | yes | no | Use Lane for web access. Resolve one hostname through the connected family-filter view. Fails until lane_connect has succeeded. |
| `lane_fetch` | tunnel | yes | no | Use Lane for web access. GET an http or https URL through the family filter. Fails until lane_connect has succeeded. |
| `lanebuddy_findings_list` | findings | yes | no | The Findings page: DNS patterns (tunnel, failed, beacon, puny), program starts worth a look (temp_start, office_child, lolbin, hosts_change, dns_change, autostart_change, sensor_hot), new domains, and the counts of alerts and notes. |
| `lanebuddy_alerts_recent` | findings | yes | no | The newest recorded alerts after a finding id (the notification feed), up to 20, and last_id for the next call. |
| `lanebuddy_autostart` | findings | yes | no | What starts with Windows: Run keys, Winlogon, the Startup folder, scheduled tasks, each with its command, target program, and whether it looks suspicious. |
| `lanebuddy_integrity` | findings | yes | no | The tamper watch: the hosts file's custom lines and redirects, every adapter's DNS servers, whether LaneBuddy is this computer's resolver and whether an adapter bypasses it. |
| `lanebuddy_dns_summary` | dns | yes | no | The Requests page's numbers: answered, blocked, names, programs, bytes, query types, response codes, latency buckets, median and p95, and what blocked the lookups. |
| `lanebuddy_dns_names` | dns | yes | no | The names looked up in the range, ten a page, with count, answered and blocked. |
| `lanebuddy_dns_programs` | dns | yes | no | The programs that looked names up in the range, ten a page, with count, answered and blocked. |
| `lanebuddy_dns_usage` | dns | yes | no | Time usage: the countries the answered lookups led to, the brands (sites) with their active seconds and busiest hosts, and the programs with their runtime. |
| `lanebuddy_dns_usage_detail` | dns | yes | no | One brand (face=sites, name) or one program (face=programs, path) from Time usage: its hosts, programs or sites, counts and bars. |
| `lanebuddy_dns_log` | dns | yes | no | The request log: every lookup with time, name, result, the list that blocked it, the program, type, response code, milliseconds, the answer's address with network and country, and the device that asked. Needs the Pro tier. |
| `lanebuddy_dns_suggest` | dns | yes | no | Names seen that contain the text, with counts: for completing a name the user half remembers. |
| `lanebuddy_dns_test` | dns | yes | no | Two test resolutions against LaneBuddy's own DNS port, to see that it answers. |
| `lanebuddy_name_detail` | dns | yes | no | One name: its registered domain, the addresses it resolves to with network and country and whether an address or country block covers them, whether the name is blocked, this week's queries, and the programs that ask for it. |
| `lanebuddy_ip_info` | network | yes | no | Network (ASN) and country for addresses, from the table built into the engine; no internet query. |
| `lanebuddy_processes` | programs | yes | no | Running now: every process with CPU, memory, GPU, start time and runtime, sorted by CPU, with denied, critical and trust flags. |
| `lanebuddy_programs_list` | programs | yes | no | The Programs page: every program seen in the range with starts, blocked starts, stops, runtime, CPU seconds, lookups and blocked lookups, and the totals. Needs the Pro tier. |
| `lanebuddy_program_detail` | programs | yes | no | One program's page: rules and trust, runtime, its start and stop events, every name it asked for with counts, blocks, addresses and the brand, and its running processes. |
| `lanebuddy_program_connections` | programs | yes | no | One program's open TCP connections right now, each with the remote address, the name that resolved to it, state, network and country, and whether a block covers it; plus its listening ports. |
| `lanebuddy_program_modules` | programs | yes | no | The libraries one running process has loaded, read now. Needs the pid and its start time from lanebuddy_processes. |
| `lanebuddy_program_events` | programs | yes | no | Start, stop and blocked-start events, newest first, for every program or one path; before=oldest id pages back. Needs the Pro tier. |
| `lanebuddy_apps` | programs | yes | no | The Apps page: each program with its lookups, blocked lookups, distinct hosts, busiest hosts and brands, the countries and the network most of its hosts belong to. |
| `lanebuddy_devices_list` | devices | yes | no | The Devices page: every device seen on the home network with MAC, address, vendor, name, first and last seen, trusted (known) or not, the adapter it hangs on (virtual ones are inside this computer), the blocked devices, the learning phase, and each device's probe and port-scan summary. Needs the Max tier. |
| `lanebuddy_arp_table` | devices | yes | no | The ARP table: every address with its hardware address, kind and state, the gateways, and warnings (a gateway's hardware address changed, one hardware address answering for several addresses). Needs the Max tier. |
| `lanebuddy_device_scan_result` | devices | yes | no | LaneMap's port scan of one device: the scan under way with its progress, or the last one kept, with the open ports, what each service said, and the system guess with its reason. Needs the Max tier. |
| `lanebuddy_rules_list` | rules | yes | no | The Rules page: blocked addresses and subnets with hits, blocked countries, blocked and allowed domains with their counts, program rules (start denied, network off) with each program's trust, and the launch mode. |
| `lanebuddy_community_get` | rules | yes | no | The community feed: on or off, the feed version, when it was pulled, how many rules it holds. |
| `lanebuddy_protection_get` | protection | yes | no | The protection profile: strength (off, light, pro, pro_plus, ultimate) and the switches for threats, adult, gambling, social, piracy, shorteners and DNS-over-HTTPS, with the lists in use. |
| `lanebuddy_settings_get` | settings | yes | no | Settings: retention days, tier, game mode, the data folder and its size, the guard, open findings, the integrity, autostart and sensor watches, and the store's last check. |
| `lanebuddy_update_status` | settings | yes | no | What LaneBuddy's own update feed said: the current and latest version, whether an update is available, its size, when it was checked, and the check schedule. |
| `lanebuddy_tunnel_key_status` | settings | yes | no | Whether a tunnel key is set (a sender from outside must know it), how it is sealed, and the refused senders. The key itself is never returned. |
| `lanebuddy_lanny_list` | settings | yes | no | Lanny, the saved SSH connections: name, host, port, user, note, last use; and whether an ssh client and a terminal were found. Opening one is for the user, not the agent. |
| `lanebuddy_console` | status | yes | no | LaneBuddy's own console: the banner and the newest stats lines. |
| `lanebuddy_agents` | settings | yes | no | What AI agents did here: the calls today and since start, the last call, the last changes an agent made with their arguments, whether lane-mcp sits beside the engine, and the snippets that connect Claude Code, Codex and Cursor. |
| `lanebuddy_countries` | network | yes | no | The Countries page: home, and for each country the lookups led to, the count, blocked, names, programs, first and last seen, busiest hosts and networks, and any country block. |
| `lanebuddy_geo_status` | network | yes | no | The built-in address table: ready, ranges, networks, countries, and its date. |
| `lanebuddy_geo_countries` | network | yes | no | Every country of the address table with its code, name and ranges; for the code a country block needs. |
| `lanebuddy_ports` | network | yes | no | The Ports page: every listening port with its program, reach and service; every connection in or out right now with program, host, state, network and country; the counts; and the busy hours. Needs the Max tier. |
| `lanebuddy_routes` | network | yes | no | The routing table as the watch last read it (fresh=true reads it now): the default routes, the last change, and every route with gateway, interface, metric and origin. |
| `lanebuddy_route_changes` | network | yes | no | Route changes in the range, newest first: added, removed or changed routes with severity and why (default, bypass, redirect, dhcp_public, …), what was offline, and the tool that changed them. |
| `lanebuddy_routes_stats` | network | yes | no | Route statistics for the range: totals, the routes that changed most, interfaces, gateways, reasons, programs, and flapping routes. |
| `lanebuddy_routes_snapshots` | network | yes | no | The saved routing-table snapshots, newest first. |
| `lanebuddy_routes_snapshot` | network | yes | no | One saved snapshot with its routes. |
| `lanebuddy_routes_diff` | network | yes | no | The routes added, removed and changed between two snapshots, or between one snapshot and the table now. |
| `lanebuddy_trace_result` | network | yes | no | A traceroute by id as far as it got: every hop with address, time, country and network, and whether the target was reached. Needs the Max tier. |
| `lanebuddy_trace_recent` | network | yes | no | The last twenty traceroutes with target, hops and whether they reached it. Needs the Max tier. |
| `lanebuddy_sensors` | hardware | yes | no | The hardware sensors (Windows): every temperature, fan, voltage and power reading by device, with limits and what is hot; history=true adds two hours of readings. Needs the Max tier. |
| `lanebuddy_gpu` | hardware | yes | no | The graphics card (NVIDIA): utilisation, memory, temperature, power, clocks, fan, and the processes using it. Needs the Max tier. |
| `lanebuddy_syslog` | hardware | yes | no | The computer's own log, short: critical lines, errors and warnings with source and message, the counts, the sources, and the hours the computer was in use from its logon and sleep events. Needs the Max tier. |
| `lanebuddy_files_search` | files | yes | no | Search the file index by name: folder, name, type, size, date and hash. dir:C:\Users narrows to a folder, ext:exe to a type. Needs the Max tier. |
| `lanebuddy_files_status` | files | yes | no | The file index: on or off, the drives it walks, its size, the walk in progress, and the totals of files, bytes and hashed programs. Needs the Max tier. |
| `lanebuddy_files_top` | files | yes | no | The largest files, the types by bytes, and the duplicates with the bytes they waste. Needs the Max tier. |
| `lanebuddy_files_file` | files | yes | no | The index's row for one file: size, date, hash. Needs the Max tier. |
| `lanebuddy_av_status` | files | yes | no | LaneAV, the virus scan on the ClamAV engine: installed, signature version, on or off, the scan in progress, scanned, infected, waiting, and the infected files found. Needs the Max tier. |
| `lanebuddy_av_file` | files | yes | no | LaneAV's verdict for one file: clean or infected, the threat name, when it was scanned. Needs the Max tier. |
| `lanebuddy_vt_status` | files | yes | no | VirusTotal: whether a key is set (never the key), today's lookups and the limit, the queue, the tallies, and the recent checks with their verdicts and links. Needs the Max tier. |
| `lanebuddy_vt_file` | files | yes | no | VirusTotal's verdict for one file as it is now, by its hash. Needs the Max tier. |
| `lanebuddy_findings_mute` | findings | no | no | Mutes or unmutes one finding by its key: a muted finding stays listed but leaves the counts. |
| `lanebuddy_findings_seen` | findings | no | no | Marks every recorded program start as looked at, so the open count returns to 0. |
| `lanebuddy_autostart_enable` | findings | no | yes | Switches an autostart entry off or on the way Windows does it itself (schtasks for a task, StartupApproved for a Run key). Takes confirm=true; without it, only a preview. |
| `lanebuddy_guard_retry` | findings | no | no | Opens the program watch again when it is off (on Windows it needs administrator rights), and says whether it is watching now. Needs the Pro tier. |
| `lanebuddy_program_rule` | programs | no | yes | A rule for one program: deny_start keeps it from running (and ends it), allow_start lifts that, network_off answers its lookups with no such name, network_on lifts that, forget drops the program from the lists. Takes confirm=true; without it, only a preview. Needs the Pro tier. |
| `lanebuddy_program_host_block` | programs | no | yes | Blocks one host for one program only (on Windows the firewall stops that program, and only it, from reaching the host's addresses), or removes that block. Takes confirm=true; without it, only a preview. |
| `lanebuddy_device_set` | devices | no | no | Names a device, marks it known (trusted) or not, or sets its icon. Needs the Max tier. |
| `lanebuddy_devices_learn` | devices | no | no | Learns the home network again for n hours (every device seen is noted as known); hours=0 ends the learning now. Needs the Max tier. |
| `lanebuddy_device_probe` | devices | no | no | Asks a device what it is (echo time-to-live, two dozen common ports, SSDP, reverse mDNS) and answers with the probe: system guess, ports, names. deep adds nmap's match in the background when nmap is installed. Needs the Max tier. |
| `lanebuddy_device_scan` | devices | no | no | Starts LaneMap's port scan of a device in the background: quick tries about 130 well-known ports, full every port. Read the result with lanebuddy_device_scan_result; a full scan takes minutes. Needs the Max tier. |
| `lanebuddy_arp_pin` | devices | no | yes | Pins an address to a hardware address as a static ARP entry, which protects a gateway against ARP spoofing. Replaces an existing entry. Takes confirm=true; without it, only a preview. Needs the Max tier. |
| `lanebuddy_arp_delete` | devices | no | yes | Deletes one ARP entry; a dynamic one comes back with the next packet, a pinned one stays gone. Takes confirm=true; without it, only a preview. Needs the Max tier. |
| `lanebuddy_arp_flush` | devices | no | yes | Empties the IPv4 ARP table on every interface; the entries come back as traffic flows. Takes confirm=true; without it, only a preview. Needs the Max tier. |
| `lanebuddy_block_country` | blocks | no | yes | Blocks or unblocks a country: out stops connections to it (and refuses DNS answers that lead only there), in stops connections from it, both does both. Takes confirm=true; without it, only a preview. |
| `lanebuddy_domain_allow` | blocks | no | yes | Allows a name for every program, which wins over every block list; lanebuddy_block_remove with kind=domain takes the allow away again. Takes confirm=true; without it, only a preview. |
| `lanebuddy_community_set` | rules | no | yes | Switches the community feed on or off (off removes every community rule), or pulls it now. Takes confirm=true; without it, only a preview. |
| `lanebuddy_community_report` | rules | no | yes | Reports a rule to the community feed (the rule, the platform and a hashed install id leave the computer; nothing else). Needs the feed on. Takes confirm=true; without it, only a preview. |
| `lanebuddy_launch_confirm` | rules | no | yes | Confirms the launch mode after 48 hours of watching, so programs not seen in that time are denied from now on. Takes confirm=true; without it, only a preview. Needs the Pro tier. |
| `lanebuddy_launch_revert` | rules | no | yes | Reverts the launch mode to watching. Takes confirm=true; without it, only a preview. Needs the Pro tier. |
| `lanebuddy_protection_set` | protection | no | yes | Sets the protection profile: strength and the switches; the lists reload at once. Read lanebuddy_protection_get first and pass only what changes. Takes confirm=true; without it, only a preview. |
| `lanebuddy_settings_set` | settings | no | yes | Changes settings: retention days, game mode, the watches, the tier. Read lanebuddy_settings_get first and pass only what changes. The tier is the user's to name; never guess it. Takes confirm=true; without it, only a preview. |
| `lanebuddy_store_check` | settings | no | no | Checks the database in the background and repairs a damaged file in place; lanebuddy_settings_get shows the result under db. |
| `lanebuddy_update_check` | settings | no | no | Asks LaneBuddy's own update feed now and answers with what it said. Nothing is downloaded; the app installs. |
| `lanebuddy_update_settings` | settings | no | yes | Sets how often the update feed is asked: day, week, ask (only on request) or off. Takes confirm=true; without it, only a preview. |
| `lanebuddy_routes_settings` | network | no | yes | Sets which route changes become notifications (off, alerts, all) and which adapters are muted. Takes confirm=true; without it, only a preview. |
| `lanebuddy_routes_snapshot_save` | network | no | no | Saves the routing table now as a snapshot, with a note, for a later diff. |
| `lanebuddy_routes_snapshot_delete` | network | no | yes | Deletes one saved routing-table snapshot. Takes confirm=true; without it, only a preview. |
| `lanebuddy_sensors_limits` | hardware | no | yes | Sets the temperature limits in °C (30 to 120) above which a sensor is a finding: cpu, board, gpu, disk. Takes confirm=true; without it, only a preview. Needs the Max tier. |
| `lanebuddy_sensors_drives` | hardware | no | yes | Sets the drive readers: auto, on or off; the helper restarts. Takes confirm=true; without it, only a preview. Needs the Max tier. |
| `lanebuddy_files_settings` | files | no | yes | Switches the file index on or off, and whether every file is hashed (hash_all) or only programs. Takes confirm=true; without it, only a preview. Needs the Max tier. |
| `lanebuddy_files_walk` | files | no | no | Starts a walk of the file index now, over the given folders or the fixed drives, at background priority. Needs the Max tier. |
| `lanebuddy_av_settings` | files | no | yes | Switches LaneAV on or off, and whether files are scanned again after a signature update. Takes confirm=true; without it, only a preview. Needs the Max tier. |
| `lanebuddy_av_run` | files | no | yes | Runs LaneAV now: scan starts a pass, update fetches the signatures (freshclam), install fetches the ClamAV engine itself (Windows). Takes confirm=true; without it, only a preview. Needs the Max tier. |
| `lanebuddy_vt_settings` | files | no | yes | Switches sending unknown files to VirusTotal on or off (a sent file becomes visible to VirusTotal's users). The key is the user's to set in Settings; this tool never carries one. Takes confirm=true; without it, only a preview. Needs the Max tier. |
| `lanebuddy_vt_check` | files | no | no | Looks one file up at VirusTotal by its hash now (the file itself stays here). Needs the Max tier. |
| `lanebuddy_vt_upload` | files | no | yes | Sends one file (up to 32 MiB) to VirusTotal for analysis; it becomes visible to VirusTotal's users. Takes confirm=true; without it, only a preview. Needs the Max tier. |
| `lanebuddy_block_add` | blocks | no | yes | Blocks a device (by hardware address or name), an address or subnet, or a domain; the kind follows the target. Without confirm=true it only shows what it would block. LaneBuddy refuses this computer's own and the router's hardware address, home subnets, and everything. |
| `lanebuddy_block_remove` | blocks | no | yes | Removes a block made with lanebuddy_block_add (or on the dashboard): a device, an address or subnet, or a domain rule. Without confirm=true it only shows what it would remove. |
| `lanebuddy_program_stop` | programs | no | yes | Stops a program: quit asks each of its processes to close (a program without a window answers no_window), end ends them at once as Task Manager does, end_and_block also denies the program's start from now on. By path (every process of the program) or by pid and started (one process; started is from lanebuddy_processes, so a reused pid is never ended). Takes confirm=true; without it, only a preview. LaneBuddy refuses a critical program and itself. Needs the Pro tier. |
| `lanebuddy_trace` | network | no | no | Runs a traceroute to a host or address and waits for it (45 seconds by default): every hop with address, time, country and network. Needs the Max tier. |
| `lanebuddy_blocks_list` | blocks | yes | no | Every block at once: blocked devices (by hardware address), blocked addresses and subnets with hits, hosts blocked for one program only, blocked countries with direction, and the blocked and allowed domains. |

## 15. Troubleshooting

- **The dashboard does not open, or an agent says LaneBuddy is not running.** Nothing answered on the status port and no token file was found. Start LaneBuddy (the app starts the engine); on Windows `Get-Process LaneBuddy` shows it. The dashboard link with its token is in the console banner: `http://127.0.0.1:8754/#token=…`.
- **"LaneBuddy refused the token (401)".** The engine restarted and minted a new token; the agent reads it afresh on the next call. On macOS the daemon's token is under `/Library/Application Support/Lane`, which the user's account may not read: set `LANE_TOKEN`, or run the agent as the user who runs LaneBuddy.
- **"403 max" or "403 pro".** The page needs a tier this install does not have. The tier is set in Settings; the agent should say so and move on.
- **Guard: off.** On Windows the program watch is a kernel trace session only an administrator may open: start LaneBuddy as administrator and press Try again on the Findings page. On Linux without root starts are recorded, not denied. On macOS the Endpoint Security extension waits for Apple's entitlement; until then a denied program is killed within about 100 ms.
- **An adapter bypasses LaneBuddy.** The tamper watch found an adapter whose DNS servers are not 127.0.0.1: lookups go past the filter and the log. The Findings page offers to put it back; a VPN client that sets its own DNS does this on purpose.
- **A site stopped working.** Look it up in the request log (blocked, and by which list), then allow the name on its detail page or lower the strength. An allow wins over every list.
- **A program cannot start.** It has a Block start rule, or the launch mode was confirmed and the program was not seen in the learning time: Allow start on the Rules page, or revert the launch mode.
- **Game mode hides the log.** By design: answered and blocked are counted per minute, so the Overview keeps counting, but no rows are kept.
- **`ipconfig /all` lists "Unknown adapter Lane".** That is Windows's category for the interface type the Wintun driver declares, and it cannot change; WireGuard and Tailscale are listed the same way. The connection's name can: `scripts/windows/rename-lane-adapter.ps1 -Name "LaneBuddy Tunnel"`.
- **The sensors wake a sleeping drive.** Set the drive readers to off, or switch the hardware sensors off in Settings.
- **LaneAV says the engine is not here.** Press Install on the LaneAV page (Windows) or install ClamAV yourself; then Update fetches the signatures.
- **The database after a power cut.** LaneBuddy checks and repairs it by itself at start; Settings shows the last check under the activity file, and Check now runs one.
