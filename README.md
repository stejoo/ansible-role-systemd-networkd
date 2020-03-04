Ansible Role: systemd-networkd
==============================

Ansible role to configure systemd-networkd profiles.

Role Variables
--------------

```yaml
# systemd.link configuration file(s)
systemd_networkd_link: {}

# systemd.netdev configuration files(s)
systemd_networkd_netdev: {}

# systemd.network configuration files(s)
systemd_networkd_network: {}

# Apply configuration by (re)starting systemd-networkd immediately?
# default: no
#   So systemd-networkd configures the network config on next reboot.
#   Can prevent a misconfigured network locking you out unexpectedly.
systemd_networkd_apply_config: no

# Enable systemd-networkd on boot?
systemd_networkd_enable: yes

# Enable systemd-resolved for DNS queries?
systemd_networkd_enable_resolved: yes

# File symlink /etc/resolv.conf should point to, possiblities:
# - /run/systemd/resolve/stub-resolv.conf, the dynamic stub-resolver,
#   which is the recommended mode of operation as of systemd 216.
# - /usr/lib/systemd/resolv.conf, the static stub-resolver
# - /run/systemd/resolve/resolv.conf, which points clients to
#   (discovered/configured) DNS servers directly.
# - a custom path of your choosing.
#
# The dynamic the stub resolver is used by default, unless your systemd version is < 216
systemd_resolved_resolv_file: ...

# Configure contents of /etc/systemd/resolved.conf.
# Defaults turn DNSSEC off to prevent DNS resolver failure with a misbehaving
# DNS server in the chain. Instead of systemd-resolved's default of
# 'allow-downgrading'.
systemd_resolved_config:
  - DNSSEC: "no"

# Attempt to disable default networking service(s) of your distribution?
disable_default_networking: yes
```

Dependencies
------------

None

Example Playbook
-------------------------

1) Configure a network profile

```yaml
systemd_networkd_enable_resolved: yes
systemd_networkd_network:
  eth0:
    - Match:
      - Name: "eth0"
    - Network:
      - DHCP: "no"
      - IPv6AcceptRouterAdvertisements: "no"
      - DNS: 8.8.8.8
      - DNS: 8.8.4.4
      - Domains: "your.tld"

      - Address: "192.0.2.176/24"
      - Gateway: "192.0.2.1"

      - Address: "2001:db8::302/64"
      - Address: "fc00:0:0:103::302/64"
      - Gateway: "2001:db8::1"
```

This will create a `eth0.network` profile in `/etc/systemd/network/`, and enable
`systemd-networkd` and `systemd-resolved`.

Variables `systemd_networkd_{link,netdev,network}` correspond to a
 profile type to create (`systemd_networkd_network` to a `.network` type,
 `systemd_networkd_link` to a `.link` type, `systemd_networkd_netdev` to a
 `.netdev` type). Every key under the profile type defines the filename of the
type. This key contains a list, of which each item is a key corresponding to a
section as documented in `systemd.network`, `systemd.link` or `systemd.netdev`.
And this key in turn contains a list of `key: value` pairs matching the
 `Key=value` pairs documented for this profile type.

2) Configure a bonded interface

```yaml
systemd_networkd_netdev:
  bond0:
    - NetDev:
      - Name: "bond0"
      - Kind: "bond"
    - Bond:
      - Mode: "802.3ad"
      - LACPTransmitRate: "fast"

systemd_networkd_network:
  bond0:
    - Match:
      - Name: "eth*"
    - Network:
      - DHCP: "yes"
      - Bond: "bond0"
```

This will create a LACP bond interface `bond0`, combining all interfaces
starting with `eth`.

License
-------

Role is written under the BSD license. Do not hesitate to report bugs, ask me
 some questions or do some pull request if you want to!
