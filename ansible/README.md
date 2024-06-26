# Ansible Playbooks and Roles for DNS Server Infrastructure Provisioning

This directory contains Ansible playbooks and roles for automating the deployment and configuration of infrastructure components. Specifically, Ansible will provision VMs in a VMware vCenter and deploy a k3s cluster that spans these machines. 

## Contents

- `inventory/`:
    - `hosts.template.yml`: The hosts file defines the target VMs that Ansible will manage as well as some group variables.

- `playbooks/`:
    - `deploy.yml`: Ansible playbook for provisioning VMs.
    - `remove.yml`: Ansible playbook for destroying the k3s cluster in its entirety.
    - `site.yml`: (Main) Ansible playbook for deploying the k3s cluster.

- `roles/`:
    - `deploy_vms/`: Role for provisioning VMs.
    - `k3s_agent/`: Role for deploying k3s agents and joining a cluster.
    - `k3s_server/`: Role for deploying k3s server(s) and deploying a cluster.
    - `prereq/`: Role for handling prerequisite tasks on nodes before cluster is deployed.

- `ansible.template.cfg`: Ansible configuration file for project-wide settings and behaviour.

- `requirements.yml`: Ansible collections required for the project.

## Requirements
The project requires the VMware Ansible Collection for provisioing VMs in a vCenter environment. You can still it using the following command:

```bash
ansible-galaxy install -r requirements.yml
```

## Configuration

### SSH Key Generation
SSH key generation is required for Ansible to securely communicate with the target VMs. The public key will be used later and automatically copied to each VM during provisioning, allowing Ansible to access them without passwords.

1. **Generate an SSH Key**:

    * If you don't already have a key, generate one using the following command:

        ```bash
        ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
        ```

    * Follow the prompts to save the key to a default location. It is recommended to **not set a password for the key**. 

2. **Copy the public key**: 

    * Print the public key:

        ```bash
        cat ~/.ssh/id_rsa.pub
        ```

    * Copy the key to your clipboard.

### Configure VM Hardware

- Open the `roles/deploy_vms/vars/main.template.yml` file in your preferred text editor.
- Fill out the following variables with your vCenter credentials, VM hardware configurations and the public SSH key copied from earlier:

    ```yaml
    ---
    # vCenter credentials
    vcenter_hostname: ""
    vcenter_username: ""
    vcenter_password: ""

    # VM hardware configuration
    esxi_hostname: ""
    datacenter_name: ""
    folder_name: ""
    resource_pool: ""
    vm_template: ""
    vm_memory: 
    vm_num_cpus: 
    vm_user: ""
    vm_password: ""
    datastore_name: ""
    network_name: ""
    netmask: ''
    gateway: ''
    dns_servers:
    - <dns_ip_01>
    - <dns_ip_02>
    vms:
    - name: k3s-master-1
        ip: <ip>
    - name: k3s-worker-1
        ip: <ip>
    - name: k3s-worker-2
        ip: <ip>
    - name: k3s-worker-3
        ip: <ip>

    # Public SSH key for VM access
    pub_ssh_key: "<Paste your public SSH key here>"
    ```

- Save the file as `main.yml`.
- **Notes**: 

    * Hardware requirements for different sized k3s clusters can be found on the k3s documentation [here](https://docs.k3s.io/installation/requirements?os=rhel).

    * This template creates VMs with identical hardware and configurations, including the user. If different hardware or usernames for different machines are required, you'll need to modify the `main.yml` file that specifies the VMs individually.

### Modify Ansible Inventory

- Open the `inventory/hosts.template.yml` file in your preferred text editor.
- Add the IP addresses of the VMs you provisioned under the appropriate groups:

    ```yaml
    server:
      hosts:
        k3s-master-1:
          ansible_host: <k3s_master_1_ip>
    agent:
      hosts:
        k3s-worker-1:
          ansible_host: <k3s_worker_1_ip>
        k3s-worker-2:
          ansible_host: <k3s_worker_2_ip>
        k3s-worker-3:
          ansible_host: <k3s_worker_3_ip>
    ```

- Save the file as `hosts.yml`.

### Ansible Configuration
- Open the `ansible.template.cfg` file in your preferred text editor.
- Fill out the following variables:

    ```ini
    [defaults]
    # Remote user for SSH connections
    remote_user = <your_remote_user>

    # Path to the private key file that was generated
    private_key_file = /path/to/private/key
    ```
    
- Save the file as `ansible.cfg`.

## Usage

### VM Provisioning

After configuring the files, the VMs can be provisioned using the following command:

```bash
ansible-playbook playbooks/deploy.yml
```

This playbook provisions VMs in your vCenter environment as per the configurations provided.

### Kubernetes (K3s) Cluster Deployment

After the VMs have been provisioned, K3s can be deployed using the following command:

```bash
ansible-playbook playbooks/site.yml --ask-become-pass
```

This playbook deploys k3s on the server (master) node, stores the generated node token, and then deploys k3s on the agent nodes which join the cluster using the node token and API endpoint.

#### Verifying K3s Nodes

After deploying the k3s cluster, you may want to ensure it is functioning correctly. Follow these steps to do so:

1. **SSH into the Server Node**:
    - Use the following command to SSH into the server node:

        ```bash
        ssh <your_vm_user>@<server_node_ip>
        ```

    - Note: SSH access will not require a password as SSH keys have been set up in the provisioning process.
    
2. **Verify K3s Server Installation**:
    - Once you have connected via SSH, you can verify the installation of k3s with the following command:

        ```bash
        sudo kubectl get nodes
        ```

    - This command should display the server node as well as any joined agent nodes:
        
        ```bash
        [john@k3s-master-1 ~]$ sudo kubectl get nodes
        k3s-master-1   Ready    control-plane,master   4m15s   v1.29.2+k3s1
        k3s-worker-1   Ready    <none>                 3m21s   v1.29.2+k3s1
        k3s-worker-2   Ready    <none>                 3m20s   v1.29.2+k3s1
        k3s-worker-3   Ready    <none>                 3m15s   v1.29.2+k3s1
        ```

3. **SSH into the Agent Nodes (Optional)**:
    - You can SSH into the agent nodes in a similar manner to the server node:

        ```bash
        ssh <your_vm_user>@<agent_node_ip>
        ```

4. **Verify K3s Agent Installation (Optional)**:
    - After logging into the agent nodes, you can run the same command as before to check the k3s installation:

        ```bash
        sudo kubectl get nodes
        ```

    - This command should display the agent node is part of the cluster.

#### Configuring External Access to Cluster
To access the cluster externally, the generated `kubeconfig` must be copied to the machine where you want to manage the cluster.
Typically, the file is located at `$HOME/.kube/config`, however k3s installs this file at `/etc/rancher/k3s/k3s.yaml`. Follow these steps to copy it to another machine:

1. **SSH into the Server Node**:

    ```bash
    ssh <your_vm_user>@<server_node_ip>
    ```

2. **Copy the `kubeconfig` File**:

    ```bash
    sudo scp /etc/rancher/k3s/k3s.yaml <your_user>@<external_machine_ip>:~/.kube/config
    ```

3. **Modify Server Address in the File**:

    - In the external machine, open the `~/.kube/config` file in your preferred text editor.
    - Update the following line with the IP address of your server node:

        ```yaml
        server: https://<server_ip>:6443
        ```

    - Save the file.

4. **Test Access**:

    - Your local `kubectl` command should now communicate with the external cluster. This can be tested by using the following command:

        ```bash
        kubectl get nodes
        ```

### Removing the Cluster

If you need to remove the entire k3s cluster, you can use the following command:

```bash
ansible-playbook playbooks/remove.yml --ask-become-pass
```