
# Incident Logs & Case Studies (In progress)

---

### 1. Building a vSphere 8 Lab on a 16GB Laptop

***The Goal***

I wanted to get my hands dirty with VMware vSphere 8.0, so I built an enterprise-style virtual data center from scratch. The catch? I had to run the entire thing on my main laptop: a Lenovo P53 (Core i7, 16GB RAM, 512GB SSD).
The Reality Check (Infrastructure Bottlenecks)

If you’ve worked with VMware, you know a standard vCenter Server Appliance (VCSA) wants 12GB–14GB of RAM just to boot. On a 16GB machine, that leaves almost nothing for the actual hypervisors.

Because of this hard hardware ceiling, I had to make some compromises:

1. The Squeeze: I managed to spin up exactly two nested ESXi hosts (3GB RAM each) and one vCenter instance (squeezed down to 10GB).
2. The Limits: Since I couldn't run a third node or set up shared external storage (like an iSCSI SAN), I couldn't test vSphere HA (High Availability) or vMotion live migrations. Everything was bound to local SSD storage.

***How I Built It anyway***

1. Nested Virtualization: I used VMware Workstation Pro on Windows 11 as the base layer, turning on Intel VT-x so I could run hypervisors inside a hypervisor.
2. Tricking vCenter: I bypassed the standard installer pre-checks to force a "Tiny" vCenter installation footprint.
3. Networking: Set up Virtual Standard Switches (vSS) to keep my management traffic completely separate from virtual machine data.
4. Integration: Connected both nested ESXi hosts to the centralized vCenter dashboard successfully.

***What I Learned***

Even without HA or vMotion, this lab was a massive win. It forced me to understand:

1. Resource Management: You learn a lot more about memory ballooning and hypervisor swapping when you are actively running out of RAM than when you have a blank check.
2. The Core Stack: I mastered the actual installation, configuration, and networking paths for ESXi and vCenter from scratch.

Next Steps

When I upgrade my RAM or add a small mini-PC, the very first thing I’m doing is spinning up a virtual storage appliance (like TrueNAS) to unlock shared storage. That will let me cross the finish line and configure vMotion and HA clustering.

    "I wanted to learn vSphere 8, but I only had my 16GB Lenovo laptop. Instead of giving up, I treated it as a resource-optimization challenge. I squeezed vCenter and two nested ESXi hosts into that footprint by aggressively trimming the management overhead. It meant I couldn't run shared-storage features like vMotion yet, but it gave me a fantastic, ground-up understanding of hypervisor networking and resource constraints."


---


### 2. Containerizing a Two-Tier App (WordPress + MariaDB) with Podman

***The Goal***

I wanted to move away from heavy virtual machines and explore lightweight containerization. To do this, I deployed a classic two-tier web application—WordPress (frontend) and MariaDB (database backend)—using Podman.
The Podman Advantage (and My Constraints)

Since I was still working on my Lenovo P53 (16GB RAM), efficiency was key. Unlike Docker, which requires a constantly running root daemon, Podman is daemonless and rootless. This meant:

- Zero Resource Waste: No background system service eating up my precious RAM when I wasn't using it.
- Better Security: The containers ran entirely in user space without requiring root privileges.

***The Challenge: No Docker-Compose?***

In the Docker world, you usually just spin up a docker-compose.yml file to link containers. While Podman can use compose tools, I wanted to learn Podman’s native, Kubernetes-style way of doing things: Pods.

I grouped both containers inside a single Pod so they could share a local network namespace and talk to each other seamlessly.
How I Built It

    Created the Pod:
    Bash

    podman pod create --name wp-pod -p 8080:80

    This opened port 8080 on my laptop so I could access the WordPress site from my browser.

Deployed the Database (MariaDB):
I launched the database container inside the pod, attaching local directories as volumes so my blog posts and configurations wouldn't vanish if the container restarted.

    Bash

    podman run -d --pod wp-pod --name wp-db -v ./db_data:/var/lib/mysql:Z -e MYSQL_ROOT_PASSWORD=secret -e MYSQL_DATABASE=wordpress -e MYSQL_USER=wp_user -e MYSQL_PASSWORD=wppass mariadb:latest

Deployed the Frontend (WordPress):
I launched WordPress into the exact same pod. Because they share a network namespace, I didn't need complex linking; WordPress could talk to the database simply by pointing to 127.0.0.1.
    Bash

    podman run -d --pod wp-pod --name wp-web -v ./wp_data:/var/www/html:Z -e WORDPRESS_DB_HOST=127.0.0.1 -e WORDPRESS_DB_USER=wp_user -e WORDPRESS_DB_PASSWORD=wppass -e WORDPRESS_DB_NAME=wordpress wordpress:latest

***What I Learned***

1. Pod Concepts: I learned how Podman maps directly to Kubernetes concepts. Grouping containers into a "Pod" makes local network mapping incredibly simple.
2. Storage Persistence: Dealing with named volumes and SELinux labels (:Z) taught me how to properly handle persistent data in a rootless container environment.
3. Resource Efficiency: Running this entire stack used a fraction of the RAM a single Linux VM would have required.

Next Steps

Podman has a killer feature: podman generate kube. My next move is to use it to export this setup into standard Kubernetes YAML manifests, preparing it to be deployed onto a real K8s cluster.

---

### 3. Building a Production-Ready Wazuh SIEM Platform with Docker
**The Project**

I deployed a complete Wazuh Security Information and Event Management (SIEM) platform on a Linux server using Docker Compose. The deployment consisted of Wazuh Manager, Wazuh Indexer (OpenSearch), and Wazuh Dashboard, with persistent storage and TLS-secured communication between all services.

The objective was not simply to get Wazuh running, but to build a deployment that followed production best practices by externalizing configuration, securing inter-service communication, and making the environment easy to maintain and reproduce.

**Why I Took on the Project**

As a Systems Administrator, I wanted practical experience with modern security monitoring and log management platforms.

Many organizations rely on SIEM solutions to detect threats, monitor endpoints, and centralize security events. Rather than learning Wazuh theoretically, I wanted to understand how each component interacts—from agent communication and log collection to indexing and visualization.

I also saw this as an opportunity to strengthen my Docker skills by deploying and managing a real-world multi-container application instead of a simple proof-of-concept container.

## Challenges I Faced
**Separating Secrets from Configuration**

The default deployment stored credentials directly inside the docker-compose.yml file. While functional, I considered this poor practice for a deployment that might eventually be version controlled or reused across environments.

I refactored the deployment by moving all sensitive configuration into a dedicated .env file and updated the Compose file to reference environment variables instead. This made the deployment cleaner, easier to maintain, and more aligned with production standards.

**Docker Compose Validation Errors**

After modifying the Compose file, Docker refused to deploy the stack due to a validation error involving the volume definitions.

Rather than assuming there was an issue with Docker itself, I reviewed the YAML structure and discovered an indentation mistake that caused one volume to be interpreted as a property of another.

Correcting the YAML formatting resolved the issue and reinforced how important syntax validation is when working with Infrastructure as Code.

**Missing Alert Indices**

Although the Dashboard was accessible, it reported that the wazuh-alerts-* index pattern could not be found.

Instead of focusing on the Dashboard, I traced the entire data pipeline from the Wazuh Manager through Filebeat to the Indexer. This led me to discover that alert data was never reaching the Indexer.

**Filebeat Authentication Failure**

Using Filebeat's built-in connectivity test, I identified a 401 Unauthorized response when attempting to connect to the Wazuh Indexer.

This narrowed the problem down to authentication rather than networking or TLS. Further investigation revealed that the manager container was still using outdated credentials because the containers had not been recreated after updating the .env file.

After recreating the containers with the updated environment variables, Filebeat authenticated successfully and the missing alert indices were created automatically.

**Understanding Wazuh's Network Architecture**

While preparing to deploy agents, I initially assumed that they communicated directly with the Dashboard because that was the interface I was interacting with.

Through the deployment process, I learned that the Dashboard is purely an administrative interface. Agents actually enroll and communicate directly with the Wazuh Manager using ports 1515 and 1514, while the Dashboard communicates with the Manager through the API.

Understanding this communication flow gave me a much clearer picture of the overall architecture and made configuring remote agents significantly easier.

**Outcome**

The completed deployment provides a fully functional, containerized Wazuh environment capable of collecting endpoint telemetry, indexing security events, and visualizing alerts through the Dashboard.

More importantly, this project strengthened my understanding of Docker Compose, Linux networking, TLS-secured services, authentication troubleshooting, and the internal architecture of an enterprise SIEM platform.

It also reinforced a lesson I apply to every infrastructure project: effective troubleshooting starts with understanding how the system is supposed to work, then validating each component methodically until the root cause is found.


---