# Local Infrastructure Development

The two important things regarding using Ansible, is to have **SSH access** and **root privileges** on Linux servers. This document explains how to set up a **Vagrant** test server.

## Setup Test Server using Vagrant

**Vagrant** is an open source tool for managing virtual machines. Vagrant has built-in support for provisioning virtual machines with Ansible. There are multiple ways to install Vagrant (*depending on OS*).

### MacOS
One way to install Vagrant on macOS, is to use package manager (like **brew**):
```sh
$ brew install hashicorp/tap/hashicorp-vagrant
```

Output should contain the following:
```sh
==> Downloading https://releases.hashicorp.com/vagrant/2.3.4/vagrant_2.3.4_darwin_amd64.dmg
######################################################################## 100.0%
==> Installing Cask hashicorp-vagrant
==> Running installer for hashicorp-vagrant; your password may be necessary.
Package installers may write to any location; options such as `--appdir` are ignored.
Password:
installer: Package name is Vagrant
installer: Installing at base path /
installer: The install was successful.
🍺  hashicorp-vagrant was successfully installed!
```

To check that Vagrant is installed, run the following command:
```sh
$ vagrant -v
Vagrant 2.3.4
```

Alternative is to download binary package from the official HashiCorp site.

### Linux
The same procedure applies here, install Vagrant using the package manager:
```sh
$ wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
$ echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
$ sudo apt update && sudo apt install vagrant
```

Or download the binary package from the official HashiCorp site.

## Setup Working Environment
It is recommended to create a working directory (e.g. *ansible-playground*), where all Ansible related files will be placed. Once the working directory is created, run the following commands to create a Vagrant configuration file (*Vagrantfile*) for an Ubuntu 20.04 (Focal Fossa) 64-bit virtual machine image, and boot it:
```sh
$ vagrant init ubuntu/focal64
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment!
Please read the comments in the Vagrantfile as well as documentation on `vagrantup.com` for more information on using Vagrant.
```

Start the virtual environment using the following command:
```sh
$ vagrant up
```

> **Warning**: *In order to start the virtual environment, a provider must be installed!*

One thing to keep in mind related to providers is that box which will be created must support the provider; *e.g. box ubuntu/focal64 does not support Docker provider.*
So the appropriate provider must be installed, which in this case is **VirtualBox**. Once the VirtualBox is installed, and vagrant up command executed again, the output (partial) should look like:
```sh
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Box 'ubuntu/focal64' could not be found. Attempting to find and install...
    default: Box Provider: virtualbox
    default: URL: https://vagrantcloud.com/ubuntu/focal64
==> default: Adding box 'ubuntu/focal64' (v20230320.0.0) for provider: virtualbox
==> default: Successfully added box 'ubuntu/focal64' (v20230320.0.0) for 'virtualbox'!
...

```

Use the following command to SSH into new Ubuntu 20.04 virtual machine:
```sh
$ vagrant ssh
```

This approach enables shell interaction, but Ansible needs to connect to the virtual machine by using the regular **SSH client**, not the `vagrant ssh` command. To output the Vagrant SSH connection details type the following:
```sh
$ vagrant ssh-config
```

The output should be similar to the following:
```sh
Host default
  HostName 127.0.0.1
  User vagrant
  Port 2222
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile ~/ansible-playground/.vagrant/machines/default/virtualbox/private_key
  IdentitiesOnly yes
  LogLevel FATAL
```

> **Tip**: *If there are problems with ssh-config, use absolute path to private_key for IdentityFile parameter.*

The important parameters are **HostName**, **User**, **Port** and **IdentityFile**, which can be used to establish the SSH connection as in the following command:
```sh
$ ssh vagrant@127.0.0.1 -p 2222 -i ~/ansible-playground/.vagrant/machines/default/virtualbox/private_key
```

## Inform Ansible About Test Server

Ansible can manage only the servers it explicitly knows about. User provides Ansible with information about servers by specifying them in an **inventory** file. To accomplish that, the **hostname** of the server, or an **alias** with additional arguments can be used.
In the following example, an alias called **testserver** will be defined for Vagrant server. For the inventory, a file called **hosts** will be created (*in the ansible-playbook directory*), with the following content:
```sh
testserver ansible_host=127.0.0.1 ansible_port=2222 ansible_user=vagrant ansible_private_key_file=.vagrant/machines/default/virtualbox/private_key
```

Ansible can be used to connect to the server named **testserver** described in the inventory file named hosts and invoke the ping module:
```sh
$ ansible testserver -i hosts -m ping
```

If local SSH client has host-key verification enabled, a message that looks like this will be displayed if it's the first time Ansible tries to connect to the server:
```sh
The authenticity of host '[127.0.0.1]:2222 ([127.0.0.1]:2222)' can't be established.
ED25519 key fingerprint is SHA256:powwy9L86/o3vKamxe9TYkk5BRbma4l3hywS+tjdnX4.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
```

If connection was successful, output will look like this:
```sh
testserver | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

The **"changed": false** part of the output tells that executing the module did not change the state of the server. The **"ping": "pong"** parameter is output that is specific to the ping module.
The **ping** module doesn’t do anything other than check that Ansible can start an SSH session with the servers. It’s a useful tool for testing that Ansible can connect to the server.

### Simplifying the Configuration File

In the previous section it is visible that Ansible requires quite a lot information in inventory file to get the necessary information about nodes. Luckily there is a simpler way, by using **ansible.cfg** file.
An example on what **ansible.cfg** file should contain:
```
[defaults]
inventory = hosts
remote_user = vagrant
private_key_file = .vagrant/machines/default/virtualbox/private_key
host_key_checking = False
```

The above file specifies the location of the inventory file (**inventory**), the user to SSH (**remote_user**), and the SSH private key (**private_key_file**). Also, SSH host-key checking is disabled (*convenient when dealing with Vagrant machines, otherwise ~/ssh/known-hosts file needs to be edited, everytime Vagrant machine is destroyed and recreated*).
