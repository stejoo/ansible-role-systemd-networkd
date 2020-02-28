Ansible Role: systemd-networkd
==============================

Ansible role to configure systemd-networkd profiles.

Role Variables
--------------

```yaml
# systemd.link profiles
systemd_networkd_link: {}

# systemd.netdev profiles
systemd_networkd_netdev: {}

# systemd.network profiles
systemd_networkd_network: {}

# may the role restart systemd-networkd to apply the new profiles?
systemd_networkd_apply_config: false

# enable systemd_resolved?
systemd_networkd_enable_resolved: yes
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
