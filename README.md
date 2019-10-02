# Full IPSec installation Ansible Role

## Requirements

### Operation systems

The role was tested on:

- CentOS 7

It may be even possible to run the role on the other distributives, but you would have to adopt file path in the role's variables files.

## Abstract

IPSec IKEv2 methods supported:

- EAP with certificates
- EAP-TLS

The role is intended to run against Linux-based machines, to install credentials on macOS/Windows/Android see [link](https://www.strongswan.org/docs/dfn_berlin_2011.pdf). `Strongswan` is intended to use as IKE keyring daemon. It will establish a policy-based VPN connection (difference between interface-based and policy-based VPN see [here](http://www.internet-computer-security.com/VPN-Guide/Policy-based-vs-Route-based-VPN.html) or [here](https://www.juniper.net/documentation/en_US/release-independent/nce/topics/concept/policy-based-route-based-vpn-comparing.html)).

This role will create a Certificate Authority center (CA), a running IPSec Linux server.

By default, IKEv2-TLS with certificates is used, you may change the behavior by choosing a specific EAP-* method (see [this](https://wiki.strongswan.org/projects/strongswan/wiki/IKEv2Examples) link) and manually change the configuration files.

For Ansible role you must either change hosts' names in the playbooks or make sure you have configured specified host groups in your inventory (e.g. `/etc/ansible/hosts`):

- `[ipsec]` - the main server where all clients will connect to. There will be stored all PKI information.
- `[clients]` - a list of linux machines where the role will automatically install, configure and run IPSec services. However, you have to transfer connection information to these clients by your hand.

### Functional roles

Variable `CURRENT_ROLES` is to specify, which function a host is assigned to.

The role can assign a host/hostgroup provided by an inventory file, here are three different functional roles:

- `ca` - specify a CA host. Key/certificate issue tasks will be accomplished here.
- `ipsec-server` - specify a host with IPSec server installed. It will serve connections from VPN users.
- `ipsec-client` - specify a host, an end-point VPN user.

To help in role assignment tasks, there are 5 variables defined (see more in "Configuration Variables" section) : `ALL_ROLES`, `CURRENT_ROLES`, `SERVER_ROLES` , `CA_ROLES`, `USER_ROLES`. These variables are system only, you should set it explicitly:

```yaml
CURRENT_ROLES: [ipsec-server, ca]
```

### Tagging

The role contains tags:

- `variables` - load and parse variables before any other tasks.
- `debug` - include debugging tasks to execute during the role.
- `packages` - install all necessary packages. Can be used to skip a long process if you have already checked that all dependencies are installed.
- `prepare` - run tasks to prepare the hosts. The tasks with `prepare` tag are dependent upon the OS type, but basically, it installs firewall policy, makes permissions in security mechanisms (such as SELinux), enables packet forwarding, etc.
- `ca` - create and configure a Certificate Authority. The CA key and certificate are issued here.
- `strongswan` - install Strongswan and configure IPSec VPN server.
- `user` - issue certificates for users and prepare configuration files for them to connect.

### Configuration variables

#### Roles

| Variable | Type | Default value| Meaning  |
| ---      |  --- | ---          |  ---     |
| ALL_ROLES    |  list|  [ipsec-client, ipsec-server, ca]       | to enumerte all possible roles |
| CURRENT_ROLES| list | [ipsec-server, ca]       | to set which roles will be set for the current host |
| SERVER_ROLES| list | [ipsec-server]       | to set a IPSec server's set of roles|
| CA_ROLES| list | [ca]       | to set a CA's set of roles|
| USER_ROLES| list | [ipsec-client]       | to set a client's set of roles|



#### Behaviour changing variables

| Variable | Type | Default value| Meaning  |
| ---      |  --- | ---          |  ---     |
| FORCE    |  bool|  false       | Force to run tasks, thus CA, VPN and users keypairs and configuration files will be refreshed |
| FORCE_USER| bool | false       | Force to update client's keypairs and configuration files |

#### Global variables

| Variable | Type | Default value| Meaning  |
| ---      |  --- | ---          |  ---     |
|  CA_FQDN         |    string   |      sun.strongswan.org        |    FQDN for CA. Can be not used, but the zone name should be along with ipsec FQDN    |
|  CA_DN         |   string    |     "C=RU, O=Technical Dept Root CA, CN={{CA_FQDN}}"         |   a string to describe the authority in CA public certificate    |
|  CA_KEY_NAME         |string|     cakey.pem         |   a convenient name for CA private key file     |
|  CA_CERT_NAME         |    string   |      cacert.pem        |    convenient name for CA certificate file     |
|  IPSEC_IP         |    string   |      "{{IPSEC_FQDN}}"        |    the actual external address (IP or FQDN) of the VPN server    |
|  IPSEC_FQDN         |   string    |      moon.strongswan.org        |  the FQDN name which will be used as Identity      |
|  IPSEC_DN         |   string    |    "C=RU, O=IPsec Server, CN={{IPSEC_FQDN}}"          |   a string to describe the IPSec server in IPSec public certificate     |
|  IPSEC_ENCRYPTION_DOMAIN         |   string    |     10.90.12.12/28         |  a virtual IP pool each VPN  where a client will be assigned an address from       |
|IPSEC_ADVERTISE_DOMAIN|string| 10.90.12.0/28|a set of comma-separated routes which will be advertised for VPN clients|
|IPSEC_ADVERTISE_DNS|string|8.8.8.8, 8.8.4.4|a set of comma-separated DNS servers server will use in the VPN connection|
|IPSEC_USERS|list|[admin,]|A list of users which will be allowed to connect. For every user, there will be automatically generated keypairs and .conf files|
|IPSEC_KEY_NAME| string|ipsec_key.pem|a convenient name for IPSec server private key file|
|IPSEC_CERT_NAME| string | ipsec_cert.pem |  a convenient name for IPSec server certificate file|

## Usage

### Prepare an inventory

Example of inventory file:

```yaml

[ipsec]
192.168.50.10

[clients]
192.168.50.20
```

### Prepare a playbook

Example of a playbook `main.yml` with default parameters:

```yaml
- hosts: ipsec
  roles:
    - role: ipsec-strongswan
      vars:
        CURRENT_ROLES: [ipsec-server, ca]


- hosts: clients
  roles:
    - role: ipsec-strongswan
      vars:
        CURRENT_ROLES: [ipsec-client]
```

Example of a playbook `main.yml` with manual parameters:

```yaml
- hosts: ipsec
  roles:
    - role: ipsec-strongswan
      vars:
        CURRENT_ROLES: [ipsec-server, ca]
        CA_FQDN: ca.mysite.com
        IPSEC_FQDN: vpn.mysite.com
        IPSEC_ADVERTISE_DOMAIN:  192.168.12.0/24, 192.168.13.0/24
        IPSEC_ADVERTISE_DNS: 1.1.1.1
        IPSEC_USERS:
          - admin
          - client1
          - client2


- hosts: clients
  roles:
    - role: ipsec-strongswan
      vars:
        CURRENT_ROLES: [ipsec-client]
```

### Running the playbook

Run the playbook:

```bash
ansible-playbook -i inventory.yml main.yml
```

Restrict to execute only users' related tasks:

```bash
ansible-playbook -i inventory.yml main.yml --tags users
```

Run the playbook without checking for dependencies installed:

```bash
ansible-playbook -i inventory.yml main.yml --skip-tags packages
```

Force to run all tasks in the playbook:

```bash
ansible-playbook -i inventory.yml main.yml -e "FORCE=true"
```
