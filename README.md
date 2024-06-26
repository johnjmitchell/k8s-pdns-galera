# DNS Server Deployment

This project automates the deployment of a DNS server infrastructure using Ansible for provisioning and Kubernetes for orchestrating the DNS server components. Here's how it works:

- Infrastructure Provisioning with Ansible: Ansible handles the deployment of virtual machines (VMs) and installs k3s, a lightweight Kubernetes distribution, on these VMs. This sets up the foundation for hosting the DNS server components.

- Deployment with Kubernetes: Once the infrastructure is in place, Kubernetes takes over to deploy and manage the DNS server components within the k3s cluster. This includes deploying pods, services, and other necessary resources to ensure the DNS server functions correctly.

## Getting Started

To get started, clone this repository to your local machine:

```bash
git clone <repository_url>
```

Then navigate to the project directory:

```bash
cd <repository-directory>
```

## Requirements

Ensure you have an existing vCenter environment where the VMs will be provisioned. 

Ensure the following software is installed on your [control machine](https://docs.ansible.com/ansible/latest/network/getting_started/basic_concepts.html#control-node):

- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) - Automation tool to manage remote machines over the SSH protocol
- [kubectl](https://kubernetes.io/docs/tasks/tools/) - Tool to run commands against Kubernetes clusters
- [pyVmomi](https://pypi.org/project/pyvmomi/#installing) - Python SDK for the VMware vSphere API

After installation, verify the versions:

```bash
ansible --version
kubectl version --client
pip show pyvmomi
```

## Usage

1. **Provision VMs and Setup Kubernetes**:
    - Navigate to the [`ansible`](./ansible/) directory.
    - Follow the instructions in the Ansible README to provision VMs and setup Kubernetes.

2. **Deploy DNS Server**:
    - Navigate to the [`kubernetes/`](./kubernetes/) directory.
    - Follow the instructions in the Kubernetes README to deploy the DNS server components.

## TODO

- [ ] Create dedicated user for PowerDNS to avoid using root.
- [ ] Investigate '[Warning] Aborted connection x to db: 'pdns' user: 'root' host: 'localhost' (Got timeout reading communication packets)' message.
- [ ] Investigate suitability of Helm or Kustomize for easier deployment.
- [ ] Handle MariaDB Galera bootstrap logic:
    - [ ] Handle subsequent pod/service restarts.
    - [ ] Implement environment variables to control bootstrap status.
- [ ] Add monitoring and logging setup for the Kubernetes cluster.
- [ ] Implement automated backups for MariaDB Galera database.
- [ ] Implement cluster autoscaling.
- [ ] Implement Hashicorp Vault for secret management.