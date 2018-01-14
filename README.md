# arouteserver

A role to install and configure [ARouteServer](https://github.com/pierky/arouteserver).

## Requirements

No requirements.

## Role Variables

Variables used by this role are listed below.

### Package installation

* (optional) `arouteserver_local_package_file`: when set, the role installs ARouteServer using the package at this local path, otherwise the last version from PyPI is fetched and installed (default).

### Route server configuration: general policy (`general.yml`)

* (optional) `arouteserver_general_cfg_file`: set this variable to the local path of the `general.yml` file that will be used to configure ARouteServer. If not set, the `configure` command will be used to setup the policy definition file using best practives and suggested settings (default).

### Route server configuration: client list (`clients.yml`)

Mandatory, one among the 3 following variables:

* `arouteserver_clients_cfg_file`: local path of the `clients.yml` file.
* `arouteserver_clients_from_euroix_file`: local path of an Euro-IX member list file that will be used to import the list of route server clients.
* `arouteserver_clients_from_euroix_url`: URL of an Euro-IX member list that will be used to import the list of route server clients.

* (mandatory when Euro-IX import is used) `arouteserver_clients_from_euroix_ixp_id`: ID of the IXP referenced within the Euro-IX member list file.

* (optional) `arouteserver_clients_from_euroix_extra_args`: any extra arguments that should be used with the `clients-from-euroix` command.

### Integration with other roles

* (optional) `arouteserver_notify_on_rs_change`: when set, the role will notify this handler when route server configuration files are updated.

### Directories layout

Directories and path used to install the role components. Default values are reported below:

* `arouteserver_venv_dir`: `~/.virtualenvs/arouteserver`.
* `arouteserver_bin`: `{{arouteserver_venv_dir}}/bin/arouteserver`.
* `arouteserver_dir`: `~/arouteserver`.
* `arouteserver_var`: `~/arouteserver_var`.
* `bgpq3_dir`: `~/bgpq3`.

### Host variable names

The following variables define the name of the hostvars used to gather some information from the route servers hosts.
For example, the variable `rs_asn` must be defined for the route server hosts and must contain the ASN of the route server.

* `arouteserver_varname_rs_asn`: `rs_asn`, the ASN of the route server. Ex. `64496`.
* `arouteserver_varname_daemon`: `daemon`, the BGP daemon used on the host. One of `bird` or `openbgpd`.
* `arouteserver_varname_daemon_version`: `daemon_version`, the version of the BGP daemon. Ex. `1.6.3`.
* `arouteserver_varname_router_id`: `router_id`, the router-id of the host. Ex. `192.0.2.1`.
* `arouteserver_varname_local_networks`: `local_networks`, comma-separated list of the local networks used by the IXP (needed to build filters that allow the route server to reject any announcement for the IXP own prefixes). Ex. `192.0.2.0/24,2001:db8::/32`.

## Dependencies

The hosts that represent the route servers must be part of the group `arouteserver_managed_routeservers`.
The variables referenced by the names reported in the *Host variable names* section must be configured on each route server host.

Example:

```
$ cat hosts 
[arouteserver_hosts]
172.17.0.2

[arouteserver_managed_routeservers]
rs1
rs2

$ cat group_vars/arouteserver_managed_routeservers
rs_asn: 64496
local_networks:
- 192.0.2.0/24
- 2001:db8::/32

$ cat host_vars/rs1 
daemon: bird
daemon_version: 1.6.3
router_id: 192.0.2.1
$ cat host_vars/rs2
daemon: openbgpd
daemon_version: 6.2
router_id: 192.0.2.2
```

## Example Playbook

```
$ cat hosts 
[arouteserver_hosts]
172.17.0.2

[arouteserver_managed_routeservers]
rs1
rs2
```

```
$ cat group_vars/arouteserver_managed_routeservers
rs_asn: 64496
local_networks:
- 192.0.2.0/24
- 2001:db8::/32
```

```
$ cat host_vars/rs1 
daemon: bird
daemon_version: 1.6.3
router_id: 192.0.2.1
```

```
$ cat host_vars/rs2
daemon: openbgpd
daemon_version: 6.2
router_id: 192.0.2.2
```

```
$ cat site.yml
---
- hosts: arouteserver_hosts
  become: yes
    gather_facts: False

  vars:
    arouteserver_clients_from_euroix_url: "https://portal.lonap.net/apiv1/member-list/list"
    routeserver_clients_from_euroix_ixp_id: 1

  roles:
  - arouteserver
```

```
$ ansible-playbook -i hosts site.yml
```

## License

GPLv3

## Author Information

Pier Carlo Chiodi - https://pierky.com

Blog: https://blog.pierky.com Twitter: [@pierky](https://twitter.com/pierky)
