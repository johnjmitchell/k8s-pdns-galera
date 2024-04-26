# Kubernetes-based DNS Management with PowerDNS, PowerDNS Admin, and Galera

This repository contains the necessary Kubernetes manifests and configurations to deploy a DNS management solution using PowerDNS with a MariaDB Galera cluster for database replication and PowerDNS Admin for web-based management.

## Project Overview

This project aims to create a scalable, resilient DNS management platform suitable for both development and production environments. It leverages Kubernetes for orchestration, PowerDNS as the DNS server, PowerDNS Admin for an intuitive web interface, and MariaDB Galera for a replicated database backend.

### Components

- **PowerDNS**: An open-source, authoritative DNS server that provides high performance and flexibility.
- **PowerDNS Admin**: A web-based management tool for PowerDNS, offering an easy-to-use interface for DNS management.
- **MariaDB Galera**: A multi-master database cluster solution for MariaDB, ensuring data is replicated across all nodes for high availability.