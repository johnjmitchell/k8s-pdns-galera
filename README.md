# Ansible Kubernetes Project

This project automates the deployment and management of a Kubernetes cluster using Ansible.

## Directories

- [ansible](./ansible/README.md): Contains Ansible playbooks and roles for deploying VMs and configuring infrastructure.
- [kubernetes](./kubernetes/README.md): Holds Kubernetes manifests for deploying services like MariaDB Galera, PowerDNS, and PowerDNSAdmin.

## Usage

Refer to individual README files in subdirectories for specific usage instructions.

## TODO

- [ ] Implement Helm for packaging and streamlining application installation.
- [ ] Handle MariaDB Galera bootstrap logic.
    - [ ] Handle subsequent pod/service restarts.
    - [ ] Implement environment variables to control bootstrap status.
- [ ] Add monitoring and logging setup for the Kubernetes cluster.
- [ ] Implement automated backups for MariaDB Galera database.
- [ ] Configure SSL/TLS certificates for securing external access.
- [ ] Investigate and implement CI/CD pipeline for automated deployments.
- [ ] Improve documentation for roles and playbooks.
- [ ] Implement cluster autoscaling.