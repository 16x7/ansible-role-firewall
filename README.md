# Firewall Role

Manage a firewall using `nftables` or `iptables2` (netfilter-persistent): install dependencies, keep only the selected backend active, and render rule files from variables.

## Requirements
- `apt`-based systems with `systemd` (Debian/Ubuntu).
- Privileges to manage services (`nftables` or `netfilter-persistent`).

## Key variables
- `default_interface`: default interface (dynamic `ansible_default_ipv4.interface`).
- `firewall.enabled`: boolean to toggle the role (default `true`).
- `firewall.backend`: `nftables` (default) or `iptables2`.
- `firewall.backend_packages`: package map per backend.
- `firewall.inbound|outbound|forwarded.default_policy`: default policy (`accept`/`drop`).
- `firewall.*.rules`: list of rules. Optional keys: `action`, `protocol`, `dport`, `sport`, `interface`, `out_interface`, `source`, `destination`, `ctstate`, `extra`, `family` (`ip`/`ip6`).

## Task flow
1) Install packages for the selected backend.  
2) Disable the alternate backend service.  
3) Enable and start the chosen backend service.  
4) Render configuration: `/etc/nftables.conf` or `/etc/iptables/rules.v4` and `/etc/iptables/rules.v6`.  
5) Notify the appropriate reload handler.

## Example usage
```yaml
- hosts: all
  roles:
    - role: 16x7.firewall
      vars:
        firewall:
          backend: nftables
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
```

To use iptables-persistent, set `firewall.backend: iptables2`; rules will be written to `/etc/iptables/rules.v4` and `/etc/iptables/rules.v6` and reloaded via `netfilter-persistent reload`.
