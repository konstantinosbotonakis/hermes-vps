# 02, Create and Connect to the VPS

This part gets the server ready.

The exact VPS provider does not matter. Hetzner, DigitalOcean, Vultr, Linode, AWS Lightsail, and similar providers are all fine.

## Step 1, Create the VPS

In your VPS provider dashboard:

- Create a new server.
- Choose Ubuntu 24.04 LTS.
- Pick a small plan.
- Use at least 1 vCPU and 2 GB RAM if possible.
- Choose a location near you.
- Add your SSH key if you have one.
- Finish the server creation.

When the server is ready, the provider will show you:

- IP address.
- Username, usually `root`.
- Password, unless you use an SSH key.

## Step 2, Connect to the server

On Mac or Linux, open Terminal.

Run this command, replacing the IP address:

```bash
ssh root@YOUR_SERVER_IP
```

Example:

```bash
ssh root@123.123.123.123
```

The first time you connect, the server may ask:

```text
Are you sure you want to continue connecting?
```

Type:

```text
yes
```

Then press Enter.

## Step 3, Update Ubuntu

Run:

```bash
apt update && apt upgrade -y
```

This updates the server packages.

It may take a few minutes.

## Step 4, Install basic tools

Run:

```bash
apt install -y curl git ufw nano
```

These tools are useful for downloading files, editing small config files, and controlling the firewall.

## Step 5, Enable a basic firewall

Run:

```bash
ufw allow OpenSSH
ufw enable
ufw status
```

If the server asks whether to continue, type:

```text
y
```

This keeps SSH open and blocks random public access to other ports.

## Step 6, Optional but recommended, create a normal user

You can run Hermes as root, but it is cleaner to create a normal user.

Run:

```bash
adduser hermes
usermod -aG sudo hermes
```

Then switch to that user:

```bash
su - hermes
```

From now on, the commands in this guide assume you are using this `hermes` user.

If you prefer to keep using `root`, the commands are mostly the same, but service paths and permissions may differ.

## Step 7, Check that the server is ready

Run:

```bash
whoami
```

You should see:

```text
hermes
```

Then run:

```bash
pwd
```

You should see something like:

```text
/home/hermes
```

That means you are in the right place to install Hermes Agent.
