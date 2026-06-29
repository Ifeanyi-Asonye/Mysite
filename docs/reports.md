
# Incident Logs & Case Studies (In progress)

---

### Building a vSphere 8 Lab on a 16GB Laptop

***The Goal***

I wanted to get my hands dirty with VMware vSphere 8.0, so I built an enterprise-style virtual data center from scratch. The catch? I had to run the entire thing on my main laptop: a Lenovo P53 (Core i7, 16GB RAM, 512GB SSD).
The Reality Check (Infrastructure Bottlenecks)

If you’ve worked with VMware, you know a standard vCenter Server Appliance (VCSA) wants 12GB–14GB of RAM just to boot. On a 16GB machine, that leaves almost nothing for the actual hypervisors.

Because of this hard hardware ceiling, I had to make some compromises:

    The Squeeze: I managed to spin up exactly two nested ESXi hosts (3GB RAM each) and one vCenter instance (squeezed down to 10GB).

    The Limits: Since I couldn't run a third node or set up shared external storage (like an iSCSI SAN), I couldn't test vSphere HA (High Availability) or vMotion live migrations. Everything was bound to local SSD storage.

***How I Built It anyway***

    Nested Virtualization: I used VMware Workstation Pro on Windows 11 as the base layer, turning on Intel VT-x so I could run hypervisors inside a hypervisor.

    Tricking vCenter: I bypassed the standard installer pre-checks to force a "Tiny" vCenter installation footprint.

    Networking: Set up Virtual Standard Switches (vSS) to keep my management traffic completely separate from virtual machine data.

    Integration: Connected both nested ESXi hosts to the centralized vCenter dashboard successfully.

***What I Learned***

Even without HA or vMotion, this lab was a massive win. It forced me to understand:

    Resource Management: You learn a lot more about memory ballooning and hypervisor swapping when you are actively running out of RAM than when you have a blank check.

    The Core Stack: I mastered the actual installation, configuration, and networking paths for ESXi and vCenter from scratch.

Next Steps

When I upgrade my RAM or add a small mini-PC, the very first thing I’m doing is spinning up a virtual storage appliance (like TrueNAS) to unlock shared storage. That will let me cross the finish line and configure vMotion and HA clustering.

    "I wanted to learn vSphere 8, but I only had my 16GB Lenovo laptop. Instead of giving up, I treated it as a resource-optimization challenge. I squeezed vCenter and two nested ESXi hosts into that footprint by aggressively trimming the management overhead. It meant I couldn't run shared-storage features like vMotion yet, but it gave me a fantastic, ground-up understanding of hypervisor networking and resource constraints."


---


### Containerizing a Two-Tier App (WordPress + MariaDB) with Podman

***The Goal***

I wanted to move away from heavy virtual machines and explore lightweight containerization. To do this, I deployed a classic two-tier web application—WordPress (frontend) and MariaDB (database backend)—using Podman.
The Podman Advantage (and My Constraints)

Since I was still working on my Lenovo P53 (16GB RAM), efficiency was key. Unlike Docker, which requires a constantly running root daemon, Podman is daemonless and rootless. This meant:

    Zero Resource Waste: No background system service eating up my precious RAM when I wasn't using it.

    Better Security: The containers ran entirely in user space without requiring root privileges.

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

    (Note: The :Z flag flag is a lifesaver—it fixes SELinux permission issues on local volumes automatically).

    Deployed the Frontend (WordPress):
    I launched WordPress into the exact same pod. Because they share a network namespace, I didn't need complex linking; WordPress could talk to the database simply by pointing to 127.0.0.1.
    Bash

    podman run -d --pod wp-pod --name wp-web -v ./wp_data:/var/www/html:Z -e WORDPRESS_DB_HOST=127.0.0.1 -e WORDPRESS_DB_USER=wp_user -e WORDPRESS_DB_PASSWORD=wppass -e WORDPRESS_DB_NAME=wordpress wordpress:latest

***What I Learned***

    Pod Concepts: I learned how Podman maps directly to Kubernetes concepts. Grouping containers into a "Pod" makes local network mapping incredibly simple.

    Storage Persistence: Dealing with named volumes and SELinux labels (:Z) taught me how to properly handle persistent data in a rootless container environment.

    Resource Efficiency: Running this entire stack used a fraction of the RAM a single Linux VM would have required.

Next Steps

Podman has a killer feature: podman generate kube. My next move is to use it to export this setup into standard Kubernetes YAML manifests, preparing it to be deployed onto a real K8s cluster.

---



