# Session 01 — Virtualization and Provisioning

Summary of FiiPractic 2026 — setting up the working environment with VirtualBox, Vagrant and MobaXterm.

## Environment Setup

0. Download and install an SSH client — [MobaXterm](https://mobaxterm.mobatek.net/download-home-edition.html) (Installer edition)

1. Download and install [VirtualBox](https://www.virtualbox.org/wiki/Downloads) and [Vagrant](https://developer.hashicorp.com/vagrant/install?product_intent=vagrant)

2. Reboot to apply drivers.

3. Create a new folder called `Vagrant`. Inside it, create a file named `Vagrantfile`.

4. Open Command Prompt in the folder where the `Vagrantfile` is located.

5. Verify we are in the correct folder with `dir` — we should see the `Vagrantfile`.

6. Validate and start the VM:
```bash
vagrant validate
vagrant up
```

7. Wait for the VM to be created. Verify with VirtualBox or `vagrant status`.

8. Stop the VM with `vagrant halt`, then start it from VirtualBox.

## SSH Key Generation

At this point we can connect via SSH to the VM using its private network IP (`192.168.100.10`). To avoid entering the password every time, we configure SSH key authentication.

1. In MobaXterm, open a local terminal and generate a key pair:
```bash
ssh-keygen -t rsa
```
Accept the default location, no passphrase.

2. Display the public key:
```bash
cat ~/.ssh/id_rsa.pub
```

3. Copy the entire public key content (starting with `ssh-rsa`).

4. Make sure the VM is running.

5. Connect to the VM:
```bash
ssh root@192.168.100.10
# password: fiipractic
```

6. Add the public key to authorized keys:
```bash
vim ~/.ssh/authorized_keys
```
Press `i`, paste the key with right-click, then `Esc` → `:wq` → `Enter`.

7. Open a new terminal and connect again — no password needed.

## Individual Tasks

### Task 1 — Add GitLab VM to Vagrantfile

Add configuration for a second VM with the following specs:

| Property | Value |
|---|---|
| Name | FiiPractic-GitLab |
| Hostname | gitlab.fiipractic.lan |
| CPU cores | 4 |
| RAM | 4 GB |
| IP | 192.168.100.20 |

Extra disk config for GitLab:
```ruby
gitlab_config.vm.disk :disk, size: "20GB", primary: true
```

## Extra

### Vagrantfile Automation

Automate aspects of the VM creation process — for example, copying the public SSH key to each VM during provisioning.

After modifying the provision step:
```bash
vagrant up --provision
```

Reference: [Vagrant Provisioning Docs](https://developer.hashicorp.com/vagrant/docs/provisioning)

### BashCrawler

A terminal-based game to learn Bash basics:

```bash
mkdir /root/learn
git clone https://gitlab.com/slackermedia/bashcrawl.git /root/learn
cd /root/learn/entrance && cat scroll
```
