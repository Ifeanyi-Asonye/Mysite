# Logs (Periodically updated)
 
---
*These are projects, incidents, troubleshooting and case studies i have encountered and still encountering..lol.*


## Project 1: Building a VMware vSphere 8.0 Home Lab

**What the Project Is**

I built a virtual data center environment from scratch using **VMware vSphere 8.0** to get hands-on experience with enterprise virtualization.

**Why I Took It On**

I wanted to master the actual installation, configuration, and management workflows of ESXi hypervisors and vCenter Server instead of just reading about them.

**The Challenges I Faced**

-   **Hardware Squeeze:** I had to run this entire multi-server setup on my personal laptop (Lenovo P53 with a Core i7, 16GB RAM, and a 512GB SSD). A standard vCenter instance alone demands 12GB to 14GB of RAM just to boot, leaving almost nothing for the actual hypervisors.
    
-   **Feature Blockers:** Because my hardware was pushed to its absolute limit, I could only spin up two nested ESXi hosts. Without a third node or shared external storage (like a SAN), I couldn't test advanced features like **vSphere HA (High Availability)** or **vMotion** live migrations.
    
**How I Resolved Them**

-   **Nested Virtualization:** I used VMware Workstation Pro on my laptop's base Windows OS, turning on Intel VT-x hardware flags so I could run hypervisors inside a hypervisor.
    
-   **Bypassing Resource Checks:** I forced vCenter to install using a hidden "Tiny" footprint configuration file, aggressively clipping its memory allocation down to 10GB so the rest of the lab had room to breathe.
    
-   **Smart Resource Skimming:** I provisioned each nested ESXi host with just 3GB of RAM, relying on hypervisor memory ballooning to keep the lab stable enough to configure standard virtual switches (vSS) and map out local storage.


---


## Project 2: Containerizing WordPress and MariaDB with Podman

**What the Project Is**

I deployed a classic two-tier web application—a **WordPress** frontend connected to a **MariaDB** database backend—using **Podman**.

**Why I Took It On**

I wanted to move away from heavy virtual machines and learn lightweight containerization. I specifically chose Podman over Docker to learn its daemonless, rootless architecture and understand Kubernetes-style "Pods."

**The Challenges I Faced**

-   **The "Localhost" Connection Trap:** When I first booted the containers, WordPress kept throwing a fatal database connection error, even though my database username and password were completely correct.
    
-   **The Data Lockout (Missing Key):** After configuring the site and making a small change to my deployment script, I restarted the containers. Suddenly, the app threw a massive 500 Cryptography Error, locking me out of my own data.
    

**How I Resolved Them**

-   **Fixing Container Networking:** I realized that setting `DB_HOST=localhost` inside the WordPress container made it look for the database inside itself. I resolved this by grouping both containers into a single native **Podman Pod**, allowing them to share a network namespace so WordPress could talk to MariaDB over `127.0.0.1`.
    
-   **Hardcoding the App Key:** I discovered that Podman was generating a new random encryption key for WordPress every time the container restarted, making it impossible to decrypt the existing database. I resolved it by manually generating a static `APP_KEY` via the CLI and saving it directly into a permanent environment file.

---


## Project 3: Deploying Snipe-IT Asset Management via Docker

**What the Project Is**

I migrated our IT asset management system, **Snipe-IT**, out of a traditional, resource-heavy Virtual Machine and deployed it as a two-tier containerized application (App frontend + MariaDB backend) using Docker.

**Why I Took It On**

Our on-prem infrastructure was bogged down by heavy standalone VMs. I took this on to reduce our server memory footprint, simplify our backup routines, and prove we could run core IT tools efficiently using Docker Compose.

**he Challenges I Faced**

-   **The "Connection Refused" Loop:** During my initial setup tests on my laptop, the Snipe-IT container completely refused to talk to the database container, throwing constant SQL errors even though my credentials matched perfectly.
    
-   **The 500 Cryptography Error (Data Lockout):** I successfully logged in and logged a few test assets. However, the moment I stopped the containers to tweak my Docker Compose file and brought them back up, the entire application crashed, locking me out of the database.
    

**How I Resolved Them**

-   **Correcting the Network Scope:** I had misconfigured `DB_HOST=localhost` in my environment file. In Docker, `localhost` means _inside that specific container_. I fixed it by changing the host variable to the actual container name of my MariaDB instance (`DB_HOST=snipe-db`), allowing Docker's internal DNS to route the traffic across their shared bridge network.
    
-   **Baking a Static Encryption Key:** Snipe-IT requires an `APP_KEY` to encrypt its data. Because I hadn't hardcoded one, Docker generated a brand-new, random key when the container recreated itself, making the old data unreadable. I fixed this by generating a permanent key via the CLI, saving it to a static `.env` file, and locking down file permissions on the production server.

________


## Project 4: Deploying Wazuh SIEM for Security Monitoring (In progress)

**What the Project Is**

I deployed **Wazuh**, an enterprise-grade Security Information and Event Management (SIEM) and XDR platform, using Docker Compose on our on-prem Linux infrastructure.

**Why I Took It On**

I wanted to establish centralized security monitoring, log analysis, and vulnerability detection across our environment without dedicating massive, expensive physical hardware or multiple standalone VMs to run it.

**The Challenges I Faced**


---