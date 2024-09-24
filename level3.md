# ACIT 2420 Assignment 1 Level 3 Guide

## Guide Objectives

- Create **SSH keys** on your local machine
- Connect to the existing Arch Linux droplet
- Install and configure `doctl` under existing droplet environment using **Pacman**
- Create a new Arch Linux droplet with `doctl`
- Configure the new droplet with `doctl` and `cloud-init`

---

Before starting, make sure you have

- Registered a [DigitalOcean](https://www.digitalocean.com/) account.
- `ssh-keygen` installed on your local machine.
- An **existing droplet** running in your DigitalOcean project.
- Went through **week2 ACIT2420 lecture** to create and connect to this droplet running Arch Linux.

## Step 1: Create SSH key pair on your local machine

To create an SSH key pair, open your terminal and run the command below:

```
ssh-keygen -t ed25519 -f ~/.ssh/assign1 -C "your email address"
```

> Explanation of this command:

- `ssh-keygen`: Generate an SSH key pair.
- `-t ed25519`: Use the Ed25519 algorithm.
- `-f ~/.ssh/assign1`: Save the key pair as `assign1` in `~/.ssh/`.
- `-C "your email address"`: Add a comment (usually your email or identifier), replace this placeholder with your own email.

The command above creates a key pair in your `.ssh` directory:

- Private key: `assign1`.
- Public key: `assign1.pub`.

> The reason why we created this SSH key pair:
>
> - Secure Access: Allowing safe connection to the droplet.
> - Automation: Allowing `doctl` and `cloud-init` to automate tasks without passwords.
> - DigitalOcean Requirements: SSH keys are recommended for secure droplet creation.

## Step 2: Add SSH key pair to your DigitalOcean account

In your DigitalOcean account, go to **Control Panel**.  
In Control Panel, click **Settings** at the bottom of the left sidebar.  
In Settings, click **Security**. Then click **Add SSH Key**.

To gain Public Key, run this command in terminal:

```
pbcopy < ~/.ssh/assign1.pub
```

> Explanation of this command:

- `pbcopy`: Copy text to clipboard.
- `< ~/.ssh/assign1.pub`: Redirect the contents of assign1.pub into `pbcopy`.

Then paste to DigitalOcean from clipboard to add SSH key pair.

## Step 3: Connect to existing Arch droplet via SSH

The existing droplet we will connect to was created during ACIT 2420 week 2 lecture, named **bcit2420-server**(SFO3/1GB/25GB Disk) with IP address `147.182.207.200`.

To connect to **bcit2420-server** via the SSH key we just added to DigitalOcean, run the following command:

```
ssh -i .ssh/assign1 arch@147.182.207.200
```

> Explanation of this command:

- `ssh`: Initiate an SSH connection.
- `i .ssh/assign1`: Use the private key (assign1) for authentication.
- `arch@147.182.207.200`: Specify the username (arch) and the server's IP address `147.182.207.200`.

After running the command above, we have successfully connected to the Arch Linux droplet in DigitalOcean via SSH key pair.

## Step 4: Install and configure `doctl` under existing droplet environment using Pacman

### Notes: Adding the step of creating a droplet using the created SSH key pair???? before step 3
