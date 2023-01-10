# IKEv2 Playbook for Ubuntu
## Intro
This playbook is designed to turn cheap KVM VPS machines into IKEv2 server. Supported OS is Ubuntu 20.04 or Ubuntu 22.04.

Most VPS providers will provide a machine with the root account and password. SSH will also be open for root and password. 
This playbook will install its own user, and sudo enabled group and close root access on the SSH. If it isn't something you want to do, comment out or remove first five lines in playbook_vpn.yml

## Quick start
### Pre-requisites
You will need the following software on your client machine:
1) ansible (https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
2) sshpass

sshpass wouldn't install through brew on Mac. On Mac install hudochenkov/sshpass/sshpass
```
brew install hudochenkov/sshpass/sshpass
```

You will need to following information:
1) Your VPS IP address or FQDN
2) Root password
3) SSH private key, that you will use to connect to the VPS after root account is disabled.
4) SSH public key for the private key in 3)

### Configuration
#### Inventory
Open inventory/vpn_servers and add your server to the list. E.g (for IP address)
```yaml
[vpnservers]
200.20.20.1
```
Or for FQDN
```yaml
[vpnservers]
rando.vpn.com
```

#### Host vars
1. Create a file under host_vars directory, of the same name as your machine's ip address or FQDN.
2. Inside that file, configure two variables: ip_address OR fqdn (there must be only one) and use_once_root_ssh_pass
E.g (for IP address)
```yaml
ip_address: 200.20.20.1
use_once_root_ssh_pass: my-secret-pw 
```
OR for FQDN
```yaml
fqdn: rando.vpn.com
use_once_root_ssh_pass: my-secret-pw 
```

#### SSH Keys
Inside the file group_vars/all configure both ssh keys. 
Set the path to your private key into the **ansible_ssh_private_key_file** and copy the contents of your public key into **ansible_ssh_public_key**. E.g

```yaml
ansible_ssh_private_key_file: ~/.ssh/vpn-private
ansible_ssh_public_key: ssh-rsa AAAA-vpn-public
```
#### VPN Users
In group_vars/all configure the users using the following syntax
```yaml
vpn_users:
  - username: <user 1>
    password: <password>
  - username: <user 2>
    password: <password>
```

If you want to set up users individually per machine, do the same thing, but in host_vars/<ip address or fqdn> file

### Run the playbook
Run the playbook using the ansible-playbook command. First 
```
ansible-playbook playbook_vpn.yml
```

"Test if vpnadmin user is accessible" command will fail the first time, ignore it. Once the playbook is finished, the VPN server is setup. You can configure your client. If you need to access the machine over SSH, use the vpnadmin account and ssh key you have supplied.

```
ssh vpnadmin@200.20.20.1 -i ~/.ssh/vpn-private
```

### Configure clients
ca-cert.pem file in the root of this project is the certificate authority that you will need to import and trust.

How to do it in your OS refer Step 7 in the following doc 

https://www.digitalocean.com/community/tutorials/how-to-set-up-an-ikev2-vpn-server-with-strongswan-on-ubuntu-20-04

