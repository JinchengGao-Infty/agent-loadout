---
name: clashverge-rules-debug
description: "Diagnose and fix ClashVerge Rev routing problems on macOS, especially TUN/fake-ip issues that break SSH, cloudflared, WebSocket, or domain-specific direct routing. Use when traffic is being hijacked by ClashVerge, domains resolve to 198.18.x.x, or specific processes/domains need DIRECT bypass. Chinese triggers: clash, 代理, 分流, 直连, 网络不通, 198.18, 翻墙规则, VPN规则."
---

# ClashVerge Rules Debug

Use this skill for macOS ClashVerge Rev debugging when a host or process must bypass TUN/proxy, especially for `ssh`, `cloudflared`, WebSocket, Cloudflare Access, campus VPN, or any hostname that starts resolving to `198.18.x.x`.

## Minimal Workflow

1. Find the active profile in `profiles.yaml`.
2. Find that profile's `merge`, `rules`, and `script` files.
3. Write DIRECT rules into the active `rules` file, usually under `prepend`.
4. Write the hostname into the active `merge` file under `dns.fake-ip-filter`.
5. If the issue must be fixed immediately, patch the generated `clash-verge.yaml` too.
6. Reload Mihomo through the unix socket controller.
7. Flush fake-ip cache and macOS DNS cache.
8. Re-test DNS and the failing command.
9. Read the service log. Trust the log over assumptions.

## What To Check First

1. Confirm the symptom.
   ```bash
   dig +short <host>
   dscacheutil -q host -a name <host>
   ```
   If the result is `198.18.x.x`, Clash fake-ip is in play.

2. Confirm Mihomo is intercepting the traffic.
   ```bash
   rg -n "<host>" ~/Library/Application\ Support/io.github.clash-verge-rev.clash-verge-rev/logs/service/service_latest.log
   ```

3. Identify the active profile and its enhancement files.
   Read `~/Library/Application Support/io.github.clash-verge-rev.clash-verge-rev/profiles.yaml`

## The Reliable Fix Pattern

For a host that must bypass Clash:

1. Add a DIRECT rule to the active rules template under `prepend`:
   ```yaml
   prepend:
     - DOMAIN,example.com,DIRECT
     - PROCESS-NAME,cloudflared,DIRECT
   ```

2. Add the hostname to `dns.fake-ip-filter` in the active merge template:
   ```yaml
   dns:
     fake-ip-filter:
       - example.com
   ```

3. Reload and flush caches:
   ```bash
   curl --unix-socket /tmp/verge/verge-mihomo.sock \
     -X PATCH 'http://localhost/configs?force=true' \
     -H 'Content-Type: application/json' \
     -d '{"path":"'$HOME'/Library/Application Support/io.github.clash-verge-rev.clash-verge-rev/clash-verge.yaml"}'
   curl --unix-socket /tmp/verge/verge-mihomo.sock -X POST http://localhost/cache/fakeip/flush
   dscacheutil -flushcache
   ```

4. Verify:
   ```bash
   dig +short example.com  # Should NOT be 198.18.x.x
   ```

## Useful Controller Commands

```bash
# Check version
curl --unix-socket /tmp/verge/verge-mihomo.sock http://localhost/version

# View current config
curl --unix-socket /tmp/verge/verge-mihomo.sock http://localhost/configs

# Reload config
curl --unix-socket /tmp/verge/verge-mihomo.sock \
  -X PATCH 'http://localhost/configs?force=true' \
  -H 'Content-Type: application/json' \
  -d '{"path":"'$HOME'/Library/Application Support/io.github.clash-verge-rev.clash-verge-rev/clash-verge.yaml"}'

# Flush fake-ip cache
curl --unix-socket /tmp/verge/verge-mihomo.sock -X POST http://localhost/cache/fakeip/flush
```

## Key Principles

- Prefer editing the active profile's `rules` and `merge` files for persistence.
- Use direct edits to `clash-verge.yaml` only for immediate fixes.
- `PROCESS-NAME` rules protect specific processes; `DOMAIN` rules protect specific hosts.
- The service log is the source of truth.
- If hostname is still `198.18.x.x` after config change, you likely missed the fake-ip cache flush.
