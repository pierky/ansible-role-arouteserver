# ansible-role-arouteserver

A role to install and configure [ARouteServer](https://github.com/pierky/arouteserver).

## Description

This roles...

* installs ARouteServer and bgpq4

* copies a local [general policy definition file](https://arouteserver.readthedocs.io/en/latest/CONFIG.html#route-server-s-configuration) (`general.yml` file) or builds one on the basis of best practices and suggestions

* copies a local list of clients (`clients.yml` file) or import it from an [IX-F Member Export JSON file](https://github.com/euro-ix/json-schemas) (like those exported by [IXP-Manager](https://github.com/inex/IXP-Manager))

* builds route server configuration files that can be finally pushed to the real route servers.

Please note: it does not sets up the real route server hosts, but only the host where ARouteServer will be executed.

The real route server hosts must be part of the group `arouteserver_managed_routeservers` to allow this role to find them.

Most of the behaviours of this role can be set using some variables that are documented below.

### Installation

ARouteServer is installed using `pip` via PyPI or from a local package on the control machine. When the `upgrade` tag is used, the `--upgrade` argument is passed to `pip` to allow an upgrade of the installation.

Any local file within the role's `templates/config` directory is copied into the ARouteServer's directory (Jinja2 templates are supported).

### General policy (`general.yml`)

The general policy can be copied from a local file (Jinja2 templates are supported) or can be built on the basis of best practices and suggestions.

A `general.yml` file will be created for each route server host. Route server details (ASN, router-id, BGP daemon) are gathered from the variables of the host itself.

### Clients list (`clients.yml`)

The list of clients can be copied from a local file or imported from IX-F Member Export JSON files, that can also be fetched via HTTP/HTTPS.

When it changes, the route server configuration files building is also triggered.

### Route servers configuration files building

The route server configuration files are saved into the ARouteServer's directory; their names follow this schema: `<hostname>-[bird4|bird6|openbgpd].cfg`.

If set, an external handler is notified when the configuration files change.

## Tags

* `configure_policy`: when set, only the general policy definition file (`general.yml`) is built.

* `configure_clients`: when set, only the list of clients is updated. If it changes, also the configuration files building is triggered.

* `build_rs_config`: when set, only the route server configuration files are built.

* `upgrade`: when set, an upgrade of the ARouteServer's package is attempted.

## Requirements

No requirements.

## Role Variables

Variables used by this role are listed below, grouped by topic.

### Package installation

* (optional) `arouteserver_local_package_file`: when set, the role installs ARouteServer using the package at this local path, otherwise the last version from PyPI is fetched and installed (default).

### Route server configuration: general policy (`general.yml`)

* (optional) `arouteserver_general_cfg_file`: set this variable to the local path of the `general.yml` file that will be used to configure ARouteServer (a Jinja2 template can be used).
If not set, the `configure` [command](https://arouteserver.readthedocs.io/en/latest/EXAMPLES.html#configure-command-output) will be used to setup the policy definition file using best practices and suggested settings (default).

### Route server configuration: client list (`clients.yml`)

Mandatory, one of the 3 following variables:

* `arouteserver_clients_cfg_file`: local path of the `clients.yml` file.
* `arouteserver_clients_from_euroix_file`: local path of an Euro-IX member list file that will be used to import the list of route server clients.
* `arouteserver_clients_from_euroix_url`: URL of an Euro-IX member list that will be used to import the list of route server clients. This can be used to [integrate ARouteServer with IXP-Manager](https://arouteserver.readthedocs.io/en/latest/USAGE.html#integration-with-ixp-manager) and to fetch the client list from there.

* (mandatory when Euro-IX import is used) `arouteserver_clients_from_euroix_ixp_id`: ID of the IXP referenced within the Euro-IX member list file.

* (optional) `arouteserver_clients_from_euroix_extra_args`: any extra arguments that should be used with the `clients-from-euroix` [command](https://arouteserver.readthedocs.io/en/latest/USAGE.html#create-clients-yml-file-from-euro-ix-member-list-json-file). Example: `--merge-from-peeringdb as-set max-prefix --vlan-id 123`.

### Route server operations: [RFC8326](https://datatracker.ietf.org/doc/html/rfc8326) graceful shutdown

The variable `arouteserver_perform_graceful_shutdown`, when set, instruct ARouteServer to build the following configuration with the [graceful shutdown](https://arouteserver.readthedocs.io/en/latest/USAGE.html#route-server-graceful-shutdown) option enabled, to temporarily drain traffic during a maintenance event.

Given the nature of the graceful shutdown operation, it's suggested to not set this variable to `true` permanently, but rather [to pass it at runtime](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#defining-variables-at-runtime) only before the maintenance is performed.

### Integration with other roles

* (optional) `arouteserver_notify_on_rs_change`: when set, the role will notify this handler when route server configuration files are updated.

### Directories layout

Directories and paths used to install the role components. Default values are reported below:

* `arouteserver_venv_dir`: `~/.virtualenvs/arouteserver`.
* `arouteserver_bin`: `{{arouteserver_venv_dir}}/bin/arouteserver`.
* `arouteserver_dir`: `~/arouteserver`.
* `arouteserver_var`: `~/arouteserver_var`.
* `bgpq4_dir`: `~/bgpq4`.

### Host variable names

The following variables define the name of the hostvars used to gather some information from the route servers hosts.
For example, the variable referenced by `arouteserver_varname_rs_asn` (`rs_asn` by default) must be defined for the route server hosts and must contain the ASN of the route server.

Please see the *Example Playbook* section for an example.

* `arouteserver_varname_rs_asn`: `rs_asn`, the ASN of the route server. Ex. `64496`.
* `arouteserver_varname_daemon`: `daemon`, the BGP daemon used on the host. One of `bird` or `openbgpd`.
* `arouteserver_varname_daemon_version`: `daemon_version`, the version of the BGP daemon. Ex. `1.6.3`.
* `arouteserver_varname_router_id`: `router_id`, the router-id of the host. Ex. `192.0.2.1`.
* `arouteserver_varname_local_networks`: `local_networks`, comma-separated list of the local networks used by the IXP (needed to build filters that allow the route server to reject any announcement for the IXP own prefixes). Ex. `192.0.2.0/24,2001:db8::/32`.

The values used to set the variables referenced by `arouteserver_varname_daemon` and `arouteserver_varname_daemon_version` (by default `daemon` and `daemon_version` respectively) must be set with one of the daemon and its version supported by ARouteServer.

They will be used to set the main command and the `--target-version` argument when executing the tool:

```
arouteserver <daemon> --target-version <daemon_version>
```

The help commands `arouteserver --help` and `arouteserver <daemon> --help` can be used to get a list of the currently supported values.

## Dependencies

The hosts that represent the route servers must be part of the group `arouteserver_managed_routeservers`.

The variables referenced by the names reported in the *Host variable names* section must be configured on each route server host (or inherited by a `group_var`).

Please see the *Example Playbook* section for an example.

## Example Playbook

**hosts** file:
```
[arouteserver_hosts]
172.17.0.2      # The host where ARouteServer will be installed and
                # executed to build route server configuration files.

[arouteserver_managed_routeservers]
rs1		# The hosts where the route servers will run.
rs2
```

**group_vars/arouteserver_managed_routeservers** file:
```
rs_asn: 64496
local_networks:
- 192.0.2.0/24
- 2001:db8::/32
```

**host_vars/rs1** file:
```
daemon: bird
daemon_version: 1.6.3
router_id: 192.0.2.1
```

**host_vars/rs2** file:
```
daemon: openbgpd
daemon_version: 6.2
router_id: 192.0.2.2
```

**site.yml** file:
```
---
- hosts: arouteserver_hosts
  gather_facts: False

  vars:
    arouteserver_clients_from_euroix_url: "http://ixp-manager.example.com/api/v4/member-export/ixf/0.6?apikey=123456"
    routeserver_clients_from_euroix_ixp_id: 1

  roles:
  - ansible-role-arouteserver
```

```
$ ansible-playbook -i hosts site.yml
```

## License

GPLv3

## Author Information

Pier Carlo Chiodi - https://pierky.com

Blog: https://blog.pierky.com Twitter: [@pierky](https://twitter.com/pierky)
