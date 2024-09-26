# ACIT 2420 Assignment 1 Level 3 Guide

## Guide Objectives

- Create an **SSH key pair** on your local machine
- Deploy an Arch Linux droplet on DigitalOcean
- Connect to the Arch droplet using SSH securely
- Install and configure `doctl` on the Arch droplet using **Pacman**
- Automate the creation of a new droplet with `cloud-init` and `doctl`
- Verify and manage your DigitalOcean droplets using `doctl`

---

Before starting, make sure you have

- Registered a [DigitalOcean](https://www.digitalocean.com/) account.
- `ssh-keygen` installed on your local machine.
- A basic understanding of SSH and terminal commands.

## Step 1: Generate an SSH key pair on your local machine

To create an SSH key pair, open your terminal and run the command below:

```
# For Linux/Mac:
ssh-keygen -t ed25519 -f ~/.ssh/assign1 -C "your email address"
```

```
# For Windows:
ssh-keygen -t ed25519 -f $HOME\.ssh\assign1 -C "your email address"
```

This will output:
![ssh key generated](assets/1_key-gen.png)

> Explanation of this command:

- `ssh-keygen`: Generate an SSH key pair.
- `-t ed25519`: Use the Ed25519 algorithm.
- `-f ~/.ssh/assign1`: Save the key pair as `assign1` in `~/.ssh/`.
- `-C "your email address"`: Add a comment (usually your email or identifier), replace this placeholder with your own email.

This command creates 2 files or say an **SSH key pair** in your `.ssh` directory:

- Private key: `assign1`.
- Public key: `assign1.pub`.

> The reason why we created this SSH key pair:
>
> - Secure Access: Allowing safe connection to your DigitalOcean droplet without password.
> - Automation: Allowing `doctl` and `cloud-init` to automate tasks without password.
> - DigitalOcean Requirements: SSH keys are recommended for secure droplet management.

## Step 2: Add SSH key to your DigitalOcean account

1. In your DigitalOcean account, go to **Control Panel**.
2. In Control Panel, click **Settings** at the bottom of the left sidebar.
3. In Settings, click **Security**.
4. Then click **Add SSH Key**.

To copy and add your public SSH key to your DigitalOcean account, run this command in terminal:

```
# For Linux/Mac:
pbcopy < ~/.ssh/assign1.pub
```

```
# For Windows:
Get-Content $HOME\.ssh\assign1.pub | Set-Clipboard
```

> Explanation of this command:

- `pbcopy`: Copy text to clipboard.
- `< ~/.ssh/assign1.pub`: Redirect the contents of assign1.pub into `pbcopy`.

Once copied, paste the public key into the SSH Key field in DigitalOcean and click **Add**.

## Step 3: Deploy an Arch Linux droplet on DigitalOcean using SSH key

In your DigitalOcean **Control Panel**, click the green **Create** button at the top.
From the dropdown, select **Droplets** to create a new droplet.

On this page, fill in the following configuration details:

1. Choose **Region**: `San Francisco`
2. **Datacenter**: `SFO3`
3. Choose an **image** -> **Custome Image**: [Arch-Linux-x86_64-cloudimg-20240901.259602.qcow2](https://gitlab.archlinux.org/archlinux/arch-boxes/-/packages/1528)
4. Choose **Size** -> SHARED CPU: `Basic` -> CPU Options: `Premium AMD` -> `$7/mo`
5. Choose **Authentication Method** -> SSH Key: choose the key `Assign1` which you just created
6. Click **Create Droplet** at the bottom right once all details are filled

> Why configure these options?
>
> - Region & Datacenter: Selecte a region closer to your geological location ensures lower latency and better performance.
> - QCOW2 Image: Saves storage, supports snapshots, and is optimized for cloud environments with compression and dynamic allocation.

## Step 4: Connect to the Arch Linux droplet via SSH

From step 3 you have created an Arch Linux droplet named **Assignment1**(SFO3/1GB/25GB Disk) with the IP address `147.182.207.200`.

![DigitalOcean droplets](assets/2_DigitalOcean_droplets.png)

To connect to **Assignment1** via the SSH key you added to DigitalOcean earlier, run the following command:

```
# For Linux/Mac:
ssh -i .ssh/assign1 arch@147.182.207.200
```

```
# For Windows:
ssh -i $HOME\.ssh\assign1 arch@147.182.207.200
```

> Explanation of this command:

- `ssh`: Initiate an SSH connection.
- `i .ssh/assign1`: Use the private key (assign1) for authentication.
- `arch@147.182.207.200`: Specify the username (arch) and the server's IP address `147.182.207.200`.

This will output:
![ssh connect droplet](assets/3_connect_SSH.png)

After running the command above, you have successfully connected to the Arch Linux droplet via SSH key.

## Step 5: Install and configure `doctl` on the Arch Linux droplet using Pacman

First, make sure you are connected to the Arch Linux droplet with the following command:

```
# For Linux/Mac:
ssh -i .ssh/assign1 arch@147.182.207.200
```

```
# For Windows:
ssh -i $HOME\.ssh\assign1 arch@147.182.207.200
```

To install `doctl`, update your system and package database to ensure compatibility with the latest version:

```
sudo pacman -Syu
```

> Explanation of this command:

- S: Sync the package databases.
- y: Refresh the package databases.
- u: Update all out-of-date packages installed on the system.

The output will be:
![update pacman](assets/4_update_pacman.png)

Now we have `pacman` available for Arch Linux package management, we are going to use **Arch User Repository (AUR)** to install `doctl`.

Next, install the essential tools to build packages from AUR:

```
sudo pacman -S --needed base-devel git
```

> Explanation of this command:

- `sudo`: Run the command with root privileges.
- `pacman`: Install the specified packages.
- `--needed`: Skip reinstallation of already installed packages.
- `base-devel`: A group of essential development tools.
- `git`: Version control tool for cloning and managing code.

The output will be:
![install packages](assets/5_install_packages.png)

AUR contains many packages that are not in official repositories, therefore we need an AUR helper to manage these packages conveniently.  
Here, we'll use one of the most popular AUR helpers, `yay` via git clone:

```
git clone https://aur.archlinux.org/yay.git
```

Then we navigate into `yay` directory and install it:

```
cd yay
makepkg -si
```

> Explanation of this command:

- `makepkg`: Build a package from a PKGBUILD script.
- `-s`: Install missing dependencies.
- `-i`: Install the built package.

The output will be:
![install makepkg](assets/6_install_makepkg.png)

(Optional) After installation, you can remove the `yay` directory:

```
cd ..
# For Linux/Mac:
rm -rf yay
```

```
# For Windows::
Remove-Item -Recurse -Force yay
```

> Explanation of this command:

- `rm`: Remove files or directories.
- `-r`: Recursively delete the directory and all its contents.
- `-f`: Force the deletion, ignoring errors or prompts.

Once `doctl` is installed, **authenticate** it with your DigitalOcean account. Generate a personal access `token` on [DigitalOcean](https://docs.digitalocean.com/reference/api/create-personal-access-token/) here.

Finally, log into `doctl` with the following command:

```
doctl auth init
```

> Enter the `token` when prompted.
> The validation prompt will be:
> ![validate token](assets/7_validate_token.png)

## Step 6: Create a `cloud-init` configuration yaml file for automated droplet Setup

Firstly, make sure you are connected to the arch linux droplet via this command:

```
ssh -i .ssh/assign1 arch@147.182.207.200
```

To create and open a yaml file using Neovim, run:

```
nvim user-data.yaml
```

> Explanation of this command:

- `nvim`: Launch the Neovim text editor.
- `user-data.yaml`: The name of the file you want to open and create.

Once inside Neovim, press `i` on you keyboard to enter insert mode.

Copy and paste the following YAML configuration into Neovim:

```
users:
  - name: newuser
    gecos: New User
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    groups: sudo
    ssh-authorized-keys:
      - ssh-rsa YOUR_SSH_PUBLIC_KEY_HERE

packages:
  - package1
  - package2

disable_root: true
```

> Explanation of this command:

- `name: newuser `: Replace 'newuser' with preferred username.
- `gecos: New User `: Replace 'New User' with full name of the user.
- `sudo: ['ALL=(ALL) NOPASSWD:ALL']`: Allow user to run with root priviledges without password.
- `groups: sudo `: Add user to the sudo group.
- `ssh-authorized-keys:
  - ssh-rsa YOUR_SSH_PUBLIC_KEY_HERE`: Replace with your actual public SSH key.
- `packages: - package1 `: Replace with the packages you want to install.
- `disable_root: true`: Disable root login via SSH.

After replacement, we will run the command below:

```
users:
  - name: kaiwen
    gecos: Kaiwen Xiao
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    groups: sudo
    ssh-authorized-keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQ...

packages:
  - nvim
  - git

disable_root: true
```

Then press `ESC`, type `:wq`, and hit `Enter` to save changes and exit Neovim.

## Step 7: Automate droplet creation using `doctl` and `cloud-init` to create a new Arch Linux droplet

To create a new droplet via `doctl`, run the following command:

```
doctl compute droplet create \
    <name of your new droplet> \
    --size <size of CPU and RAM> \
    --image <image ID> \
    --region <region and datacenter> \
    --ssh-keys <ssh key ID> \
    --enable-backups \
    --user-data-file user-data.yaml
```

> Explanation of this command:

- `doctl compute droplet create \`: Create a new droplet on using `doctl`
- `    <name of your new droplet> \`: The name of your droplet. We are replacing it with `NewDroplet` here.
- `    --size <size of CPU and RAM> \`: The size of the droplet. We are using `s-1vcpu-1gb`, which means 1 virtual CPU and 1 GB of RAM.
- `    --image <image ID> \`: The image ID of the OS that will be installed on the droplet. We are using `165064181`, which is the ID of the Arch Linux image uploaded to DigitalOcean earlier.
- `    --region <region and datacenter> \`: Region for the droplet, we are using `sfo3` which is the closest to Vancouver.
- `    --ssh-keys <ssh key ID> \`: The ID of the SSH key for secure access.
- `    --enable-backups`: Enable automatic backups for the droplet.
- `    --user-data-file user-data.yaml`: Specify the cloud-init configuration file to execute during droplet creation.

Therefore the command we are going to run is:

```
doctl compute droplet create \
    NewDroplet \
    --size s-1vcpu-1gb \
    --image 165064181 \
    --region sfo3 \
    --ssh-keys 43471790 \
    --enable-backups \
    --user-data-file user-data.yaml
```

The output will be:
![create droplet](assets/8_create_droplet.png)

To validate if the droplet has been sucessfully created, run the command below to display a list of all active droplets in your DigitalOcean account:

```
doctl compute droplet list
```

The output will be:
![list all droplets](assets/9_list_all_droplets.png)

---

## Conclusion

In this guide, you learned how to set up an Arch Linux droplet on DigitalOcean. You started by creating an SSH key pair to secure your connection, then you deployed the droplet and connected to it. After that, you installed and configured `doctl`, a command-line tool for managing your DigitalOcean resources. You also created a `cloud-init` configuration file to automate user setup and package installation. Finally, you used `doctl` and `cloud-init` to create a new droplet.
