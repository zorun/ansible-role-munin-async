# Ansible role for munin-async agent

This Ansible role installs and configure munin-async, a monitoring agent
that complements munin-node.

Compared to a bare munin-node setup, the addition of munin-async allows to:

- **fetch data over SSH** instead of the insecure TCP-based communication protocol used by munin-node
- **fetch data asynchronously:** the munin-async agent will poll the local munin-node
  every 5 minutes and persist the resulting data, allowing a munin master to
  fetch data from munin-async at any time (e.g. every 30 minutes)

See the [Munin documentation on munin-async](http://guide.munin-monitoring.org/en/latest/node/async.html)
for a more complete overview.

## Features and overview

This role does only two things, but does them well:

- install `munin-async` and `munin-node` on a target system, with support for several operating systems
- copy the public SSH key of the munin master so that it can fetch data from this target system

It is recommended to use a separate role to configure munin-node to your liking,
such as the [munin-node role from geerlingguy](https://github.com/geerlingguy/ansible-role-munin-node).

Configuration of the munin master needs to be done separately.

## Requirements

This role has been tested to run on Debian (from wheezy to bullseye), Ubuntu,
CentOS 7 and 8, Fedora.  It should work on all Debian-derivatives and RedHat-derivatives.

## Role variables

This role has a single variable:

    munin_async_sshpubkeys: []

A list of SSH public keys that will be installed in the munin-async user home, to allow the munin master node to connect via SSH.

## How to use

Start by generating a pair of SSH keys on your **munin master**:

    $ sudo -u munin ssh-keygen
    $ cat /var/lib/munin/.ssh/id_rsa.pub


On your **ansible controller**, add this SSH key to the `munin_async_sshpubkeys` variable.
This can be done either through a `group_vars` file, or directly in your playbook.
Assuming you have a group named `monitorednodes`:

    # Playbook: munin-async.yml
    - hosts: monitorednodes
      vars:
        munin_async_sshpubkeys: ["ssh-rsa AAAAB3NzaC1y...m/mRpZR munin@munin-master"]
      roles:
        - { role: munin-async }

Apply this playbook to your nodes:

    $ ansible-playbook munin-async.yml

This will install and configure `munin-async` with the SSH key of the munin master.

Finally, on your **munin master**, you need to declare each node that needs to be monitored:

    # /etc/munin/munin-conf.d/machines.conf
    [foo.example.com]
      address ssh://munin-async@foo.example.com:22
      use_node_name yes

Of course, you could write a simple role/playbook to manage that through ansible as well.


## License

MIT
