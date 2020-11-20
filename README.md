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
run all other plays in the `tasks` directory.

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
