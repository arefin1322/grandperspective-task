# grandperspective-task
This project is for implementating Grandperspective GmbH's ansible task as part of its recruitment process. 

## Goal

    The goal is to demonstrate how you approach infrastructure automation and secure service
    deployment. The challenge is deliberately small-scale — the main focus will be on how you think
    about reliability, security, and maintainability, not on writing perfect code.

## Task

    Ansible Playbook(s): 
    Write an Ansible playbook that performs the following on a fresh Ubuntu 22.04 LTS server:
    1. System updates: Updates all system packages.
    2. User management: Creates a dedicated user named “appuser” with sudo access.
    3. Security baseline:
        o Harden SSH: disable password auth and root login.
        o Configure a firewall (UFW) that allows only SSH and HTTP.
    4. Service deployment:
        o Installs and enables Docker
        o Deploys a simple Nginx container, mapping it to host port 80. You can use the
        official nginx:alpine image without any modifications.
    5. Documentation:
        Please include a README.md file that explains:
        o How to run the playbook (command + inventory structure).
        o One or two extra hardening steps you’d add in production.
        o Any assumptions you made.

It’s a plus but not mandatory if you can do below as well.

GitHub Actions Workflow: Create a basic GitHub Actions workflow (.github/workflows/test-
ansible.yml) that uses ansible-lint to validate your Ansible playbook on every push to the main
branch.

----------------------------------------------------------------------------------------
# Solution

## Assumtions :
    1. Assumed passwordless publickey Based Authentication for SSH into to the remote VM from the host machine has been implemented already
        with the command : "ssh-copy-id -i <PUBKEY_PATH> <VM_USER>@<VM_IP>" (example)
    2. The invertory ip address input/argument is supported for a single valid ip address only 
    (Note: SSH with multiple ips or domain-specific names are not supported in this case)
    3. Assumed Host machine's public key file exists at the specific location as : "/home/${USER}/.ssh/id_rsa.pub"
    4. Assumed OpenSSH is pre-installed in the remote VM (ubuntu 22.04 LTS) by default
    5. Assumed remote VM's (ubuntu 22.04 LTS) default user with sudo access is : "ubuntu" (no password)
    6. Assumed remote VM's (ubuntu 22.04 LTS) PasswordAuthentication is set to "yes" by default in "/etc/ssh/sshd_config"
    7. New dedicated sudo user to create in the remote VM is predefined as : "appuser" (no password)

## Prerequisite on Host Machine : 
    Python 3.0+ , Ansible 2.8+ , id_rsa.pub

## How to run the Ansible playbooks :

   1. Command to test the connectivity of the remote VM from the host machine (Optional): 

```
    ansible-playbook -e "vm_ip=<VM_IP>" test_challenge.yml --tags "connect"
```

   2. Command to try a dry run before running the test challenge (Optional): 

```
    ansible-playbook -i '<VM_IP>,' -e "vm_ip=<VM_IP>" test_challenge.yml --check
```

   3. Command to run the test challenge :

```
    ansible-playbook -i '<VM_IP>,' -e "vm_ip=<VM_IP>" test_challenge.yml
```

   4. Command to run the playbooks individually with tags:

```
    ansible-playbook -e "vm_ip=<VM_IP>" test_challenge.yml --tags "connect,harden,deploy"
```

## Structure overview of the Ansible project :
    Due to the simpler use case, 'roles/' base structure has not been considered for this ansible project.
    So, to keep it simple, I have created ansible playbooks at the root directory as -
     1. test_challange.yml : the main/master playbook which invokes other playbooks sequentially with tag features included
     2. connect_vm.yml : the playbook to test connectivity of the ubuntu vm from the host
     3. harden_vm.yml : the playbook for implementing new ubuntu vm's security baseline or hardening
     4. deploy_nginx.yml : the playbook for deploying nginx webapps and its dependencies
     5. .gitignore : to ignore unwanted local configs, tmp and secret files during remote git push
     6. README.md : for documentation purposes of this project

## Possible Extra Hardening steps :
    1. Enable Unattended Security Updates → Keeps system patched automatically
    2. Enforce Auditd for Logging → Tracks file changes, user actions, and SSH logins
    3. Limit sudo privileges with /etc/sudoers.d/appuser → estricts to specific admin commands
    4. Install "Fail2ban" → To prevent malicious activity like, repeated failed login attempts (Brute-force) or exploit scanning

## Possible Improvement steps :
    1. Extend it to a role base modular structure
    2. Group common vars to a yml file to be used by the roles playbooks
    3. Make it more Idempotent, Dynamic and Scalable to be used by multiple remote hosts with ips or domain names and service deployments
  

