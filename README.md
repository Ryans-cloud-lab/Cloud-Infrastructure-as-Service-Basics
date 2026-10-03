# Cloud-Infrastructure-as-Service-Basics

**Project Description:** Creating a server and deploying an application on DigitalOcean

**Technologies used:** DigitalOcean, Linux, Java, Gradle

**Key project tasks and activities:**

- Setup and configure a server on DigitalOcean.
- Create and configure a new Linux user on the Droplet (Security best practice).
- Deploy and run a Java Gradle application on Droplet.


Tools such as **Nexus (an artifact repository)** and **Jenkins (a build automation tool)** are installed on the remote dedicated servers in the cloud and **not only** just on the local machine. This action matches real environments because you are essentially deploying and operating on servers you have control over.


**<ins>Infrastructure as a service (IaaS):</ins>** provides on-demand compute, storage and networking resources, so you rent virtual servers instead of buying physical hardware. The cloud provider manages the physical infrastructure, while the user manages the operating system, security and applications.

In this project, I used **DigitalOcean's** IaaS offering to provision an Ubuntu Droplet (virtual server), then took responsibility for configuring the OS, creating a non-root user and deploying the application myself.

On DigitalOcean, Linux virtual machines are called **Droplets**

## Prerequisites for this project

- A DigitalOcean account (use signup credits where available).
- An SSH client on your computer (OpenSSH is enough: `ssh`, `ssh-keygen`, `scp`).
- **Java 17** and **Gradle** on your machine if you build the course example locally. The sample app uses a Java 17 toolchain (see `java` block in `build.gradle`).
  
**Example application used was (Gradle / Spring Boot):** [java-react-example](./java-react-example/)
