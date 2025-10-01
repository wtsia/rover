

-----

## A Proxmox Journey

Building a homelab can feel daunting, but with the right tools and a bit of guidance, any enthusiast can transform a spare PC into a versatile server capable of running anything from game servers to virtualized desktops. This guide chronicles my real-world journey of setting up a server using Proxmox VE, navigating the common hurdles, and unlocking its advanced capabilities.

### Part 1: Installing Proxmox VE

Every great server starts with a solid foundation. Proxmox VE (Virtual Environment) is a powerful, open-source hypervisor based on Debian Linux. It allows you to run both full virtual machines (VMs) and lightweight Linux Containers (LXC) on a single machine, all managed through a clean web interface.

**Key Steps:**

1.  **Installation:** The process is straightforward. Burn the Proxmox ISO to a USB drive, boot your server from it, and follow the installer prompts. The most important step is selecting the correct management interface and assigning a static IP address. This IP is your gateway to everything.
2.  **Post-Install Configuration:** Once installed, the first administrative task is to set a **Fully Qualified Domain Name (FQDN)** by editing `/etc/hosts` to ensure all system services are happy. This prevents a number of common errors down the line.

The particular steps of this process are fairly self-explanatory.

### Part 2: Mastering LXC Containers

While VMs are great for running different operating systems, LXC containers are the undisputed champions of efficiency for running Linux services. They share the host kernel, using a fraction of the RAM and CPU overhead of a full VM.

Our primary use for an LXC was to create a dedicated, isolated host for Docker. Thankfully, the Proxmox Community have an array of helper scripts to get the job done and is done as simple as running the following in the Proxmox VE shell:

```
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/docker.sh)"
```


For a different way of setting it up (untested on my end),

**Setting Up a Docker Host LXC:**

1.  **Create the Container:** Use a lightweight template like **Debian**. Assign it a static IP on your network. A good starting point for resources is 2 CPU cores, 2GB of Memory, and 2GB of Swap.
2.  **Enable Nesting:** For Docker to run correctly inside an unprivileged container, you must enable nesting. Shut down the container and, from the Proxmox host shell, edit its config file (`/etc/pve/lxc/<CT_ID>.conf`) and add:
    ```
    features: nesting=1,keyctl=1
    ```
3.  **Install Docker:** Start the container, open its console, and follow the official Docker installation guide to install `docker-ce` and `docker-compose`.

### Part 3: Advanced Networking

By default, Docker containers live on an internal, private network. To recap how embedded our container is, We have a bare-metal Proxmox VE server hosting a Linux container that is hosting Docker which is hosting a sub-container.

Usually to expose a service, you map ports. But what if you want a container to act like a real device on your LAN with its own IP? The answer I arrived at was `macvlan`.

I configured this for a game server container, allowing it to be seen on the local network without port mapping. The game will be a classical single player game that thanks to the work of some special developers, can turn into a local co-op RPG. 

**The `docker-compose.yml` for a `macvlan` Container:**

```yaml
version: "3.8"

services:
  skyrimserver:
    container_name: skyrimserver
    image: tiltedphoques/st-reborn-server:latest
    restart: unless-stopped
    networks:
      lan-net:
        ipv4_address: 10.0.0.50 # The new static IP for the container

networks:
  lan-net:
    driver: macvlan
    driver_opts:
      parent: eth0  # The LXC's primary network interface
    ipam:
      config:
        - subnet: 10.0.0.0/24
          gateway: 10.0.0.1
          ip_range: 10.0.0.48/28 # A reserved IP range for Docker
```

### Part 4: Advanced Disk Management

Expanding a disk in Proxmox is simple, but shrinking one is a high-risk operation that requires manual intervention. I successfully navigated this to reduce an oversized VM disk which I accidentally had allocated in GB (Giga-Bytes) when I was under the impression I was dealing with MB, ballooning the volume size from a few GB to a couple TB.

**The Shrinking Workflow (High-Level):**

1.  **Backup First\!** This process can destroy data. A full backup is mandatory.
2.  **Shrink from the Inside:** Boot the VM from a **GParted Live ISO** or if it's a Windows-based VM, a partition manager. Use its graphical tools to shrink the filesystem and partition to the desired size. Note the new size.
3.  **Create a New Disk:** In Proxmox, create a new virtual disk for the VM that is slightly larger than the shrunken partition.
4.  **Copy the Data:** From the Proxmox host shell, use the `dd` command to perform a block-level copy from the old, large disk to the new, smaller one. Crucially, you must use the `count` parameter to tell `dd` to stop copying after it has copied the shrunken partition's data, preventing a "no space left on device" error. In this case I used 38912 for MB.
    ```bash
    # Example for copying 38GB
    dd if=/path/to/old/disk of=/path/to/new/disk bs=1M count=38912 status=progress
    ```
5.  **Finalize:** Detach the old disk, set the new disk as the primary boot device in the VM's **Options -\> Boot Order**, and verify it boots correctly.

### Part 5: A Troubleshooting Diary: Where I Ran Into Issues

One of the most valuable lessons in any homelab is learning to troubleshoot. 

I encountered an issue while port scanning using `nmap` that the service was not available. Scanning the port should'be shown something like:

```
PS C:\Users\winst> nmap -sU -p <port> <ip_address>
Starting Nmap 7.97 ( https://nmap.org ) at 2025-09-30 15:33 -0400
Nmap scan report for <ip_address>
Host is up (0.0040s latency).

PORT      STATE         SERVICE
<port>/udp open|filtered unknown
MAC Address: AA:BB:CC:DD:EE:00 (Unknown)
```

But what I actually got was a `STATE` of `closed`. 

When the `macvlan`-enabled game server was still unreachable despite a correct Docker configuration, I followed a logical diagnostic path:

1.  **Is the Application Running?** I checked the container logs (`docker logs skyrimserver`) and confirmed the server process had successfully started and bound to port 10578. This ruled out the application itself.
2.  **Is there a Firewall *Inside* the Container?** Since the LXC was running Debian, `ufw` wasn't installed. We used `nft list ruleset` to check for the native `nftables` firewall. After adding a temporary rule (`nft add rule ...`), the port was still closed. This ruled out the container's internal firewall.
3.  **Is there a Firewall on the Proxmox Host?** This was the final suspect. By selecting the LXC container in the Proxmox UI, going to its **Firewall** tab, and adding an `ACCEPT` rule for incoming UDP traffic on port `10578`, the final block was removed and the server became accessible.

This process highlights a key principle: isolate the problem layer by layer, from the application outwards.

### Part 6: Day-to-Day Administration/Experimenting

Beyond the big projects, I covered essential administrative tasks:

  * **Changing a Guest's IP:** Edit `/etc/network/interfaces` and `/etc/hosts` on the guest itself.
  * **Deleting a Disk:** First **Detach** the disk from the VM/container in the Hardware tab, then select the resulting "Unused Disk" and **Remove** it.
  * **Fixing an Oversized LXC Disk:** Unlike VMs, the safest way to shrink an LXC disk is to **Backup** the container (which only copies used data) and then **Restore** it, specifying the correct, smaller disk size in the restore dialog.

Additionally, I'm able to spin up multiple VMs and LCXs to experiment with from Active Directory to a Wazuh container for monitoring endpoints or networks, and log analysis. 

### Conclusion

From a single physical machine, this homelab now hosts an efficient Docker environment with containers that act as first-class citizens on the network, alongside fully isolated virtual machines. The journey from a simple installation to complex troubleshooting and administration demonstrates the incredible power and flexibility of Proxmox VE. The learning curve is real, but the capabilities you unlock are well worth the effort.