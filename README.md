# Cloud-Infrastructure-as-Service-Basics

**Project:** Creating a server and deploying an application on DigitalOcean

**Technologies used:** DigitalOcean, Linux, Java, Gradle

**Key project tasks and activities:**

- Setup and configure a server on DigitalOcean.
- Create and configure a new Linux user on the Droplet (Security best practice).
- Deploy and run a Java Gradle application on Droplet.
##
## Description

In real environments, applications and tools such as **Nexus (artifact repository)** and **Jenkins (build automation)** run on remote servers in the cloud, not on a developer's laptop. This project replicates that: I provisioned a cloud server, secured access to it, and deployed and ran an application on it.


**<ins>Infrastructure as a service (IaaS):</ins>** provides on-demand compute, storage and networking resources, so you rent virtual servers instead of buying physical hardware. The cloud provider manages the physical infrastructure, while the user manages the operating system, security and applications.

In this project, I used **DigitalOcean's** IaaS offering to provision an Ubuntu Droplet (virtual server), then took responsibility for configuring the OS, creating a non-root user and deploying the application myself.

In this project, I used DigitalOcean's IaaS offering to provision an Ubuntu Droplet (virtual server), then took responsibility for configuring the OS, creating a non-root user and deploying the application myself.

On DigitalOcean, Linux virtual machines are called **Droplets**

## Prerequisites for this project

- A [DigitalOcean](https://www.digitalocean.com/) account (Sign up credits are used where possible to not get charged).
- A **SSH client** on your computer (OpenSSH is enough: `ssh`, `ssh-keygen`, `scp`).
- **Java 17** and **Gradle** installed locally (the app uses a Java 17 toolchain, defined in `build.gradle`)
  
**Example application (Gradle / Spring Boot):** [java-react-example](./java-react-example/)

**Server used:** Ubuntu [24.04 LTS] Droplet, 1 vCPU / 512 MB RAM / 10 GB disk, London region ( `lon1` ).

---
# Setting up a Server On DigitalOcean

Goal: provision an Ubuntu Droplet, reachable securely over SSH, that can run a packaged Java application (JAR).

### 1. Create a DigitalOcean account and sign in

Created an account and signed in to the DigitalOcean control panel.

### 2. Configure SSH keys

**SSH keys** let you log in to any Droplet from your machine **without a password**, using private to public-key authentication.

**On your Command Line Interface:**

1. Check for an existing key (use either `~/.ssh/id_ed25519.pub` or `~/.ssh/id_rsa.pub`).
2. If you need to create a new key pair:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com" -f ~/.ssh/id_ed25519
```

3. Copy the **public** key (`~/.ssh/id_rsa.pub` file). Then in the DigitalOcean web UI, add it under **Settings → Security → SSH keys** (or add it when you create the Droplet).

Added the public key to DigitalOcean (Settings → Security → SSH keys). The private key stays only on my machine and is never committed to Git.

### 3. Create a Droplet (Linux Ubuntu)

Created an Ubuntu [24.04 LTS] <ins>**Droplet**</ins> (1 vCPU / 512 MB / 10 GB, London) with SSH key authentication selected instead of a password.

**Droplet:** a Linux-based virtual machine running on DigitalOcean's infrastructure.


### 4. Open SSH (port 22) with a firewall

I allowed **inbound** TCP traffic on **port 22** so my local machine can open a SSH session.

Created or attached a **Cloud Firewall** on DigitalOcean so that:

- **Inbound rules** describe traffic **into** the Droplet (here: SSH from your IP or a controlled range—tightening the source improves security).
- **Outbound rules** describe traffic **from** the Droplet out to the internet (defaults are often permissive for learning).

Exact clicks vary slightly over time; use DigitalOcean’s docs for **Cloud Firewalls** and attach the firewall to your Droplet.

### 5. SSH into the server using its public IP

Connected using the Droplet's public IPv4 address (initially as root):

```bash
ssh root@YOUR_DROPLET_PUBLIC_IP
```

### 6. Install Java on the Droplet

Installed the OpenJDK 17 runtime (headless, since the server has no GUI) and verified the version:

**Examples on Ubuntu** (run on the Droplet after SSH):

```bash
sudo apt update
sudo apt install -y openjdk-17-jre-headless
java -version
```

Commands may differ on other distributions; adjust to match your Droplet’s OS.

---

## Deploy and run application artifact on Droplet

Goal: **build** the application into an executable artifact locally, transfer it to the server and run it remotely, the standard build → ship → run flow.

Gradle produces two JARs: the executable Spring Boot fat JAR (`java-react-example.jar`) and a `-plain.jar` without dependencies. I deployed the fat JAR.

*You may use `root` or another account for this first pass; the next section documents creating dedicated Linux users as a security best practice for ongoing work.*

### 1. Build the JAR file

On the **local** machine, from this repository’s `java-react-example` directory (for example after `cd` into the repo root):

```bash
cd java-react-example
gradle bootJar
```

Confirmed the artifact:

```bash
ls -1 build/libs/
```

Use `java-react-example.jar` for deployment.

### 2. Copied the JAR to the remote server
Transferred it securely with `scp` (over SSH):

```bash
scp build/libs/java-react-example.jar root@YOUR_DROPLET_PUBLIC_IP:~/
```

In the case of deploying under a different Linux user, replace `root` with that username and ensure that user’s home (or target directory) is writable.

### 3. Run the application

SSH to the Droplet, then:

```bash
java -jar ~/java-react-example.jar
```

Spring Boot listens on **8080** by default unless the `server.port` is changed in configuration. For a quick test from your laptop (example):

```bash
curl http://YOUR_DROPLET_PUBLIC_IP:8080/
```

**Firewall note:** port 8080 had to be opened inbound. SSH (22) alone does not allow web traffic.

Limitation: the app runs in the foreground and stops when the SSH session closes. In production I would run it as a **systemd service** (see Improvements).

---

## Create and configure a Linux user on cloud server

Goal: apply security best practice by avoiding routine use of root and separating responsibilities with dedicated Linux accounts (principle of least privilege).

**Context:** New Droplets start with root access. I created a dedicated non-root user `ryan` for day-to-day administration and deployment.

### 1. Why avoid routine use of `root`

`root` can change anything on the system. Mistakes are catastrophic, and attackers who compromises `root` controls the entire VM. A normal user with sudo should be used for administration, and dedicated users for services.

### 2. Create an administrative user (example: `admin`)

Created the user, granted sudo rights, and set up key-based SSH login for it:

```bash
adduser admin
usermod -aG sudo admin
```

Set a strong password or rely on SSH keys for `admin` (keys are preferable). To allow SSH key login for `admin`, copied the authorized key pattern from `/root/.ssh/authorized_keys` to `/home/admin/.ssh/authorized_keys` with correct ownership:

```bash
mkdir -p /home/admin/.ssh
cp /root/.ssh/authorized_keys /home/admin/.ssh/authorized_keys
chown -R admin:admin /home/admin/.ssh
chmod 700 /home/admin/.ssh
chmod 600 /home/admin/.ssh/authorized_keys
```

Verified login as the new user:

```bash
ssh admin@YOUR_DROPLET_PUBLIC_IP
```

### 3. Optional: dedicated user for the application

In production, the app would run under its own user that owns only its files:

```bash
sudo adduser --disabled-password --gecos "" myapp
sudo mkdir -p /opt/myapp
sudo mv /path/to/java-react-example.jar /opt/myapp/
sudo chown -R myapp:myapp /opt/myapp
```


### 4. Summary

- **Avoided** using root for daily administration.
- **Used** a non-root sudo user for maintenance and deployment.
- **Applied** least privilege: separate users for separate responsibilities.
- **Aligned** file permissions and SSH keys so each user can only do what it should.

---

## References

- DigitalOcean documentation: [Droplets](https://docs.digitalocean.com/products/droplets/), [How to Connect to Droplets with SSH](https://docs.digitalocean.com/products/droplets/how-to/connect-with-ssh/), [Cloud Firewalls](https://docs.digitalocean.com/networking/firewalls/)
