# ansible-cybera-common

This role does the following:

* Configures a server to use an upstream apt-caching server (Debian/Ubuntu only).
* Clones and installs https://github.com/cybera/cybera-dotfiles.
* Configures iptables to be a deny-by-default firewall.
* Whitelists and blacklists specified IPv4 and IPv6 subnets.
* Configures IPv6 RAs and temp addresses.
* Installs and removes a specified set of packages.
* Configures the server for automated security updates (Debian/Ubuntu only).
* Manages SSH authorized keys.
* Configures systemd journal.
* Applies a set of Ubuntu quirk fixes.
* Manages specified system users.
* Configures and manages rsyslog client settings.
* Configures and manages Postfix to send mail to an upstream smtp server.
* Configures and manages mail aliases in /etc/aliases.

## Usage

### Variables

See the [defaults/main.yml](./defaults/main.yml) file for the variables
you can set in your Ansible playbooks.

You'll need to specify them in either the global `group_vars/all/cybera.yml`
file or specific to individual roles, such as `group_vars/netbox/cybera.yml`

### Within a Playbook

This role can be used in a playbook like so:

```yaml
- name: Foo
  hosts: all
  tasks:
    - name: Apply common settings
      include_role:
        name: "ansible-cybera-common"
```

This will cause the `tasks/main.yml` file to trigger, which will subsequently
run nearly all the plays in the `tasks` directory.

Two notable tasks sets are not included by default - sensu_agent and google_authenticator.

If you only want to run specific plays from the `tasks` directory, you will
need to do something like:

```yaml
- name: Foo
  hosts: all
  tasks:
    - name: Manage iptables
      include_role:
        name: "ansible-cybera-common"
        tasks_from: iptables

    - name: Manage SSH keys
      include_role:
        name: "ansible-cybera-common"
        tasks_from: ssh
```

## Task List
### Default Tasks
* `acng` - Managing caching for apt packages. Included by default on Debian machines.
* `dotfiles` - Installs [Cybera's dotfiles](https://github.com/cybera/dotfiles). Set `cybera_install_dotfiles` to false to disable
* `iptables` - Firewall rules
* `ipv6` - IPv6 sysctls for consistent behaviour
* `packages` - Installs defined packages from variables.
* `postfix_client` - Default postfix configuration
* `rsyslog_client` - Default rsyslog configuration
* `security_updates` - Ensure Ubuntu security updates are enabled
* `ssh` - Install keys and disable password authentication
* `systemd` - Tweaks for systemd to prevent it filling the disk
* `timezone` - Set timezone
* `ubuntu` - Ubuntu specific items we want to disable
* `users` - Manage users

### Optional Tasks
The optional tasks are *not* run by default and must be manually opted into. For example:
```yaml
- name: Foo
  hosts: all
  tasks:
    - name: Manage 2FA TOTP codes
      include_role:
        name: "ansible-cybera-common"
        tasks_from: google_authenticator
```

* `google_authenticator` - Install and configure TOTP using lib-googleauthenticator. The backup codes will be copied to where you are running Ansible from. Be sure to **save the TOTP and Backup codes to your password manager**
* `sensu_agent` - Install and configure sensu agent. Config is expected to be in `cybera_sensu_agent_config`
* `sol` - Configure Serial over LAN so the OS will display over SoL

## Requirements

See the `requirements.yml` file for all required collections and roles.

You can install these requirements by doing:

```shell
$ ansible-galaxy install -r requirements.yml
```

## Testing

Install Docker and Molecule:

```shell
# Docker
apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
apt-get update -qq
apt-get install -y docker-ce docker-ce-cli containerd.io

# Molecule
python3 -m pip install "molecule[docker,lint]"

modprobe ip6table_filter
echo ip6table_filter >> /etc/modules
```

And then do:

```shell
$ mkdir work
$ cd work
$ git clone https://git.cybera.ca/cybera/ansible-cybera-common
$ cd ansible-cybera-common
$ molecule test
```
