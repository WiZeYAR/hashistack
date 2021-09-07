# Hashistack

Hashistack is a bash script to setup a highly-opinionated cluster running Conul, Vault, and Nomad. The script can help you install, update, and configure each applications individually on servers and set up a fully functioning cluster.

## Usage

### Basic Usage

For full command and usage documentation, run `./hashistack help`. Hashistack requires root to run and the command is expected to be run on the server where the application needs to be installed.

```bash
# Install applications
$ ./hashistack install [consul | vault | nomad]

# Configure installed applications (Requires parameters)
$ ./hashistack configure [consul | vault | nomad] [PARAMS]
```

### Configuration Commands

The `hashistack` commands needs to be run individually for every application installation and configuration. The applications need to be bootstrapped on installation, which generates encryption tokens and ACLs and sets up the configuration files for each application. There is no particular order in which the configuration commands should run, but assuming a fresh installation with default configuration, the order of configuration would be: Consul, Vault, Nomad. This is because both Vault and Nomad are dependent on Consul ACLs to function, and Nomad requires Vault for secrets access.

#### Consul Configuration

```bash
# Bootstrap the Consul cluster after installation.
$ ./hashistack configure consul --bootstrap-expect=[int] \
			--datacenter=[string] --mode=server --retry-join=[string] --bootstrap

# Configure Consul servers and clients with existing ACL tokens.
$ ./hashistack configure consul --bootstrap-expect=[int] \
			--datacenter=[string] --mode=[server|client] --retry-join=[string] \
			--consul-encryption-token=[token] --consul-master-token=[token] \
			--consul-default-token=[token]
```

#### Vault Configuration

```bash
# Bootstrap the Vault cluster after installation
# Vault requires Consul ACL Master token to bootstrap. This token can
# be any Consul token capable of generating new tokens.
#
# If Consul is not running on the same server as Vault, specify
# --consul-address with the address to the Consul Instance.
$ ./hashistack configure vault --consul-master-token=[token] --vault-kms-key=[string] \
			--bootstrap --generate-root-token

# Configure Vault servers with existing tokens.
$ ./hashistack configure vault --vault-kms-key=[string] --consul-vault-token=[token]
```

#### Nomad Configuration

```bash
# Bootstrap the Nomad cluster after installation
# Nomad requires Vault root token and Consul ACL Master token to bootstrap.
# These tokens can be any token with enough capability to generate new tokens.
#
# If Consul or Vault is not running on the same server as Nomad, specify
# --consul-address and --vault-address.
$ ./hashistack configure nomad --bootstrap-expect=[int] \
			--datacenter=[string] --mode=server --retry-join=[string] \
			--consul-master-token=[token] --vault-root-token=[token] \
			--install-cni --bootstrap

# Configure Nomad servers and clients with existing tokens.
$ ./hashistack configure nomad --bootstrap-expect=[int] \
			--datacenter=[string] --mode=[server|client] --retry-join=[string] \
			--consul-nomad-token=[token] --nomad-encryption-token=[token] --nomad-vault-token=[token] \
			--install-cni
```
