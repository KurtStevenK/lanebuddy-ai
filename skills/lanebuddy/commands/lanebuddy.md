---
description: Ask LaneBuddy how the system is doing, what is on the network, or have it block a device, address or domain, stop a program, or change a setting.
---

Use the lanebuddy skill (the `lanebuddy` MCP tools) to answer: $ARGUMENTS

If nothing follows, answer "how is my system doing": call `lanebuddy_status` and `lanebuddy_findings_list`, then report in five lines: protection, DNS answered and blocked today, programs running and denied, open alerts by kind, new devices. For a request to block, stop or change something, show the tool's preview first, ask once, then call again with `confirm=true` and repeat the undo line.
