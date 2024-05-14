# Kubernetes-based DNS Management with PowerDNS, PowerDNS Admin, and Galera

This directory contains the necessary Kubernetes manifests and configurations to deploy a DNS management solution using PowerDNS with a MariaDB Galera cluster for database replication and PowerDNS Admin for web-based management. Metallb is used for external load balancing. Manifests can be applied using:

```bash
kubectl apply -f <manifest.yml> 
```

## Contents

- `configmaps/`: Contains ConfigMap manifests for various deployed services.
- `deployments/`: Contains Deployment manifests for various deployed services.
- `namespaces/`: Contains Namespace manifests for organizing resources.
- `networkpolicies/`: Contains NetworkPolicy manifests for enforcing network access rules.
- `rbac/`: Contains Role-Based Access Control (RBAC) manifests for defining access permissions.
- `scripts/`: Contains scripts for deployment and management tasks.
- `services/`: Contains Service manifests for exposing applications internally.
- `statefulsets/`: Contains StatefulSet manifests for managing stateful applications.
- `volumes/`: Contains PersistentVolume and PersistentVolumeClaim manifests.
- `kind-config.yml`: Configuration file for Kubernetes in Docker (kind) for local testing.
- `metallb-native.yaml`: Manifest file for deploying Metallb in native mode.
- `metallb-config.template.yml`: Template for MetalLB configuration.

## Configuration

### MetalLB
MetalLB is configured to provide external load balancing for services within the cluster. Before deploying MetalLB, ensure that you customise the `metallb-config.template.yaml` with your network configuration. 

#### Editing MetalLB Configuration

1. **Open MetalLB Configuration File**: Open the `metallb-config.template.yaml` file in your preferred text editor.

2. **Configure IP Address Range**: In the file, modify the `addresses` section to suit your environment. This section defines the IPs that MetalLB will use for external load balancer IPs.

    ```yaml
    addresses:
    - 192.168.1.240-192.168.1.250
    ```

3. Save the file as `metallb-config.yaml`.
### Secrets

## Usage

### Deploying Load Balancer
Metallb is the external load balancer. To deploy it, run the following commands:

```bash
kubectl apply -f metallb-native.yaml
kubectl apply -f metallb-config.yaml
```

### Deploying MariaDB Galera

### Deploying PowerDNS

### Deploying PowerDNSAdmin

