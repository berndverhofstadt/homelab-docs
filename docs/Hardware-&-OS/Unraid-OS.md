---
authors:
    - Bernd Verhofstadt
weight: 32
---
# Unraid OS
[Unraid](https://unraid.net) is an operating system (OS) designed for personal and small business use, particularly in the context of network-attached storage (NAS) and server applications. It is known for its unique approach to storage management and the ability to run multiple virtual machines and Docker containers on a single system. Here are the key features and functionalities of Unraid:

### Storage Management
Parity Protection: Unraid uses a parity system to protect against data loss. Parity works by storing the XOR value of data across multiple drives, allowing the system to recover data in case of a drive failure.Mixed Drive Sizes: Unraid supports the use of different-sized hard drives in the array, providing flexibility in expanding storage capacity over time.

### Virtualization
- **Virtual Machines (VMs)**: Unraid supports the creation and management of virtual machines, allowing users to run multiple operating systems on a single hardware platform. This is particularly useful for testing, development, or running applications that may require different operating systems.
- **Docker Containers**: Unraid utilizes Docker containers for application deployment. Docker allows for lightweight, portable, and easily scalable application deployment, enhancing efficiency and resource utilization.

### User-Friendly Web Interface
Unraid features a web-based graphical user interface (GUI) that simplifies system management and configuration. Users can monitor storage health, manage virtual machines, and access various settings through a web browser.
Plugin System:Unraid has a plugin system that enables users to extend the functionality of their system. There are plugins available for additional features, such as backup solutions, media servers, and more.

### Expandability and Upgradability
Unraid is designed to be easily expandable. Users can add new hard drives to the array as needed, and the system can dynamically adjust to different drive sizes. Upgrading the Unraid OS itself is a straightforward process, often accomplished through the web interface.

### Community Support
Unraid has a vibrant and active community that contributes to forums, discussions, and plugin development. This community support is valuable for troubleshooting, sharing tips, and exploring new use cases.

### Media Server Capabilities
Unraid is often used as a platform for building media servers. Its combination of storage flexibility, virtualization, and Docker support makes it well-suited for hosting media-related applications like Plex, Emby, or Jellyfin.
Unraid's unique approach to storage management, coupled with its virtualization and Docker capabilities, makes it a popular choice for users looking to build a robust and flexible home server or NAS solution.

### Usefull Unraid plugins
Via the community app, many plugins are available, I have been using the following set of plugins (not containers):

- Appdata Backup by Robin Kluth
- Dynamix File Manager by bergware
- rclone by Waseh's Repository
- User Scripts by Squid's Repository

For details on how I back up and restore the overall Unraid configuration (including container templates), see the dedicated backup page.
