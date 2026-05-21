---
date: 2026-05-21
categories: 
    - Technical
---

# SSH into WSL Via Powershell: A Simple Guide

Most times, i feel overwhelmed when i want to learn a new technology. Learning ssh was one i had a bit of difficulty with. Although i learned it the hard way, i mean using EC2 instance and all but i found another way. Bear in mind that this is just the basics, something to get you up to speed quickly and not a substitute for learning with cloud infra. This guide shows you how to connect to your Windows Subsystem for Linux (WSL), a presumed remote instance via SSH from PowerShell. 

<!-- more -->
### 1. Set Up WSL
First, ensure WSL is installed on your Windows PC. If it's not, you can install it by opening PowerShell as an administrator and running:

```bash
wsl --install
```

Follow any prompts to complete the installation and set up your Linux distribution (e.g., Ubuntu). 

### 2. Install SSH in WSL

Once WSL is set up and you've launched your Linux distribution, you'll need to install the SSH server. Open your WSL terminal and run the following commands:

```bash
sudo apt update
sudo apt install openssh-server
```

After installation, start the SSH service: sometimes, after installation, it starts automatically

```bash
sudo systemctl status ssh #to see if it is active, if not, then run the next cammand
sudo service ssh start
```

You might also want to enable it to start automatically on boot:

```bash
sudo systemctl enable ssh
```

### 3. Get Your WSL IP Address

In your WSL terminal, find your WSL instance's IP address. This is typically assigned to `eth0`. Type:

```bash
ip a
```

Look for the `inet` address listed under `eth0`. It will usually be in the format `172.x.x.x` or similar. **Make a note of this IP address.**

### 4. Connect via SSH from PowerShell

Now, open your **PowerShell terminal** (on your Windows machine, not WSL). Use the following command to connect, replacing `wsl_user_name` with your WSL username and `Wsl_ip_addr` with the IP address you noted in the previous step:

```powershell
ssh wsl_user_name@Wsl_ip_addr
```

For example, if your WSL username is "john" and your IP address is "172.20.10.4", the command would be:

```powershell
ssh ifeanyi@172.20.10.40
```

The first time you connect, you may be asked to confirm the authenticity of the host. Type `yes` and press Enter. You will then be prompted to enter your WSL user's password.

And you are in!!!
Congratulations, tech bro or sis, you are now in the top percent of cloud engineers. 

There's another layer where you can create an ssh key and all, but that's beyound this guide. 