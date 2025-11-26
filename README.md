# Firewall Role

Manage a firewall using `nftables` or `iptables2` (netfilter-persistent): install dependencies, keep only the selected backend active, render rule files from variables, and optionally define NAT rules.

## Requirements
- `apt`-based systems with `systemd` (Debian/Ubuntu).
- Privileges to manage services (`nftables` or `netfilter-persistent`).

## Key variables
- `default_interface`: default interface (dynamic `ansible_facts['default_ipv4']['interface']`).
- `firewall_backend`: default backend selector (sets `firewall.backend` if not provided). Use this to switch backends without redefining the entire `firewall` map.
- `firewall.enabled`: boolean to toggle the role (default `true`).
- `firewall.backend`: firewall engine (`nftables` or `iptables2`). Defaults to `firewall_backend` (nftables). If you define the full `firewall` dict in inventory, include this key there.
- `firewall.backend_packages`: package map per backend.
- `firewall.inbound|outbound|forwarded.default_policy`: default policy (`accept`/`drop`).
- `firewall.inbound|outbound|forwarded.rules`: list of filter rules. Optional keys: `action`, `protocol`, `dport`, `sport`, `interface`, `out_interface`, `source`, `destination`, `ctstate`, `extra`, `family` (`ip`/`ip6`).
- `firewall.nat.prerouting|postrouting`: list of NAT rules. Optional keys: `interface`, `out_interface`, `protocol`, `dport`, `sport`, `source`, `destination`/`match_dest`, `to`, `extra`.

## Task flow
1) Install packages for the selected backend.  
2) Disable the alternate backend service.  
3) Enable and start the chosen backend service.  
4) Render configuration: `/etc/nftables.conf` or `/etc/iptables/rules.v4` and `/etc/iptables/rules.v6` (including NAT if present).  
5) Enable IPv4 forwarding when forward/NAT rules exist.  
6) Notify the appropriate reload handler.

## Example usage
```yaml
- hosts: all
  roles:
    - role: 16x7.firewall
      vars:
        firewall_backend: nftables
        firewall:
          inbound:
            default_policy: drop
            rules:
              - action: accept
                protocol: tcp
                dport: 22
                interface: "{{ default_interface }}"
          outbound:
            default_policy: accept
            rules: []
          forwarded:
            default_policy: drop
            rules: []
          nat:
            prerouting: []
            postrouting: []
```

To use iptables-persistent, set `firewall.backend: iptables2`; rules will be written to `/etc/iptables/rules.v4` and `/etc/iptables/rules.v6` and reloaded via `netfilter-persistent reload`.
