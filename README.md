# Cloud-Infrastructure-as-Service-Basics

**Project:** Creating a server and deploying an application on DigitalOcean

**Technologies used:** DigitalOcean, Linux, Java, Gradle

**Key project tasks and activities:**

- Setup and configure a server on DigitalOcean.
- Create and configure a new Linux user on the Droplet (Security best practice).
- Deploy and run a Java Gradle application on Droplet.
##
## Description

Tools such as **Nexus (an artifact repository)** and **Jenkins (a build automation tool)** are installed on the remote dedicated servers in the cloud and **not only** just on the local machine. This action matches real environments because you are essentially deploying and operating on servers you have control over.


**<ins>Infrastructure as a service (IaaS):</ins>** provides on-demand compute, storage and networking resources, so you rent virtual servers instead of buying physical hardware. The cloud provider manages the physical infrastructure, while the user manages the operating system, security and applications.

In this project, I used **DigitalOcean's** IaaS offering to provision an Ubuntu Droplet (virtual server), then took responsibility for configuring the OS, creating a non-root user and deploying the application myself.

On DigitalOcean, Linux virtual machines are called **Droplets**

## Prerequisites for this project

- A [DigitalOcean](https://www.digitalocean.com/) account (Sign up credits are used where possible to not get charged).
- A **SSH client** on your computer (OpenSSH is enough: `ssh`, `ssh-keygen`, `scp`).
- **Java 17** and **Gradle** on your machine if you build the course example locally. The sample app uses a Java 17 toolchain (see `java` block in `build.gradle`).
  
**Example application used was (Gradle / Spring Boot):** [java-react-example](./java-react-example/)

---
# Setting up a Server On DigitalOcean

An Ubuntu Droplet is created which you can reach over SSH and run a packaged Java application (JAR).

### 1. Create a DigitalOcean account and sign in

Complete account creation in the DigitalOcean control panel. Use the new-account credits where applicable.

### 2. Configure SSH keys

**SSH keys** let you log in to any Droplet from your machine **without a password**, using private to public-key authentication.

**On your Command Line Interface:**

1. Check for an existing key (use either `~/.ssh/id_ed25519.pub` or `~/.ssh/id_rsa.pub`).
2. If you need to create a new key pair:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com" -f ~/.ssh/id_ed25519
```

3. Copy the **public** key (`~/.ssh/id_rsa.pub` file). Then in the DigitalOcean web UI, add it under **Settings → Security → SSH keys** (or add it when you create the Droplet).

Keep your **private** key secret; never upload it to DigitalOcean or commit it to Git.

### 3. Create a Droplet (Linux Ubuntu)

1. In DigitalOcean, choose **Create → Droplets**.
2. Select an **Ubuntu** image (LTS is a good default).
3. Choose a size and datacenter region appropriate for learning.
4. Under **Authentication**, choose **SSH key** and select the key you added.
5. Create the Droplet.

**Terminology:** A Droplet is a **Linux-based virtual machine** running on DigitalOcean’s infrastructure.

### 4. Open SSH (port 22) with a firewall

You must allow **inbound** TCP traffic on **port 22** so your computer can open a SSH session.

On DigitalOcean, create or attach a **Cloud Firewall** (or equivalent networking rules) so that:

- **Inbound rules** describe traffic **into** the Droplet (here: SSH from your IP or a controlled range—tightening the source improves security).
- **Outbound rules** describe traffic **from** the Droplet out to the internet (defaults are often permissive for learning).

Exact clicks vary slightly over time; use DigitalOcean’s docs for **Cloud Firewalls** and attach the firewall to your Droplet.

### 5. SSH into the server using its public IP

1. In the Droplet’s page, copy its **public IPv4** address.
2. From your machine (first login is often as `root` when the provider configures it that way):

```bash
ssh root@YOUR_DROPLET_PUBLIC_IP
```

If you use a non-root user later, replace `root` with that username.

### 6. Install Java on the Droplet

The example application targets **Java 17**.

**Examples on Ubuntu** (run on the Droplet after SSH):

```bash
java -version
```

If Java 17 is not installed, install an OpenJDK 17 runtime (package names can vary by Ubuntu release):
