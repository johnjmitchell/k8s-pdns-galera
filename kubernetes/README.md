# Kubernetes-based DNS Management with PowerDNS, PowerDNS Admin, and Galera

This directory contains the necessary Kubernetes manifests and configurations to deploy a DNS management solution using PowerDNS with a MariaDB Galera cluster for database replication and PowerDNS Admin for web-based management. MetalLB is used for external load balancing. Manifests can be applied using:

```bash
kubectl apply -f <manifest.yaml> 
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
In Kubernetes, secrets are used to store sensitive information such as passwords, tokens and connection strings. Secrets can be created directly from the command line or using a dedicated secrets file.

While secrets alone don't encrypt the stored data, they are a critical component in Kubernetes clusters. They provide a way of securely storing any sensitive information and keeping them hidden. However, inputting sensitive information directly into the command line, as shown in the example below, can expose them in the shell's history, logs or command-line history.

```bash
kubectl create secret generic db-user-pass \
    --from-literal=username=admin \
    --from-literal=password='password'
```

#### Generating Secrets from Files
To mitigate this risk, it's recommended to store sensitive data in a file and then pass it to `kubectl` to generate secrets. This approach keeps the sensitive information out of the command line, reducing the risk of exposure. 

Use the following commands to generate files containing the relevant data:

```bash
# Create a directory to store secrets
mkdir -p secrets

# Generate and save PowerDNS API key to a txt file
echo -n $(openssl rand -hex 32) > secrets/powerdns-api-key.txt

# Generate and save root password for MariaDB to a txt file
echo -n "s1mplepassw0rd" > secrets/mariadb-root-password.txt

# Generate a random password for PowerDNS and MySQL connection
password=$(openssl rand -base64 12)

# Generate and save PowerDNS password to a txt file
echo -n $password > secrets/powerdns-password.txt

# Generate and save MySQL connection string to a txt file
TODO
```

Then use the following command to generate the secrets:

```bash
kubectl create secret generic dns-secrets \
    --from-file=powerdns-api-key=./secrets/powerdns-api-key.txt \
    --from-file=mariadb-root-password=./secrets/mariadb-root-password.txt \
    --from-file=powerdns-password=./secrets/powerdns-password.txt
```

While this approach improves security, a secure vault such as [HashiCorp Vault](https://developer.hashicorp.com/vault) should be used. Vault provides a centralised and secure way to manage and distribute secrets across Ansible environments and Kubernetes clusters. It offers features such as encryption, access control, auditing and secret rotation, ensuring that sensitive information remains protected at all times. 

## Usage

### Deploying MetalLB
To deploy MetalLB, use the following commands: 

```bash
kubectl apply -f metallb-native.yaml
kubectl apply -f metallb-config.yaml
```

You can verify it deployed successfully with:

```bash
kubectl get pods -n metallb-system
```

### Deploying MariaDB Galera

To deploy MariaDB Galera into the cluster, follow these steps:

### 1. Apply Manifests

First, apply the ConfigMap, Service and StatefulSet YAML files to create the necessary resources.

```bash
kubectl apply -f configmaps/mariadb-configmap.yaml
kubectl apply -f services/mariadb-service.yaml
kubectl apply -f statefulsets/mariadb-statefulset.yaml
```

### 2. Check Pod Status
To verify the service was deployed successfully, you can check the status of the pods by using the following command:

```bash 
kubectl get pods -l app=mariadb-galera
```

This should display:

```bash
NAME               READY   STATUS    RESTARTS   AGE
mariadb-galera-0   1/1     Running   0          13m
mariadb-galera-1   1/1     Running   0          13m
mariadb-galera-2   1/1     Running   0          12m
```

### 3. Initialise Database
Once the pods are running, you need to initialise the database. This involves executing SQL commands to import the required database schema and create a user.

#### 3.1 Importing the PowerDNS Schema
To import the PowerDNS [schema](https://doc.powerdns.com/authoritative/backends/generic-mysql.html), you can import it directly into the database using the following command:

```bash
kubectl exec mariadb-galera-0 -- mariadb -uroot -p"$(kubectl get secret dns-secrets -o jsonpath="{.data.mariadb-root-password}" | base64 --decode)" pdns < schema.sql
```

- **Note**: This step can be automated by implementing a startup script in the Docker entrypoint.

#### 3.2 Creating PowerDNSAdmin User & Database
PowerDNSAdmin requires its own database for storing users, configuration and application-specific data. Enter the following commands to create the necessary database and user:

```bash
kubectl exec mariadb-galera-0 -- mysql -uroot -p"$(kubectl get secret dns-secrets -o=jsonpath='{.data.mariadb-root-password}' | base64 --decode)" <<EOF
CREATE DATABASE IF NOT EXISTS powerdnsadmin CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
GRANT ALL PRIVILEGES ON powerdnsadmin.* TO 'pdnsadminuser'@'localhost' IDENTIFIED BY '$(kubectl get secret dns-secrets -o=jsonpath='{.data.powerdns-password}' | base64 --decode)';
FLUSH PRIVILEGES;
EOF
```


#### 3.3 Accessing MariaDB Console
To access the MariaDB database, you can use the following command:

```bash
kubectl exec -it mariadb-galera-0 -- mariadb -uroot -p"$(kubectl get secret dns-secrets -o jsonpath="{.data.mariadb-root-password}" | base64 --decode)"
```

### Deploying PowerDNS
To deploy the PowerDNS Authoritative server into the cluster, follow these steps:

### 1. Apply Manifests

First, apply the ConfigMap, Service and Deployment YAML files to create the necessary resources.

```bash
kubectl apply -f configmaps/pdns-configmap.yaml
kubectl apply -f services/pdns-service.yaml
kubectl apply -f deployments/pdns-deployment.yaml
```

### 2. Verify Deployment
To verify the service was deployed successfully, you can check the status of the pods by using the following command:

#### 2.1 Check Pod Status

```bash 
kubectl get pods -l app=powerdns
```

This should display something similar to this:

```bash
NAME                        READY   STATUS    RESTARTS   AGE
powerdns-7658b9f678-6t2v7   1/1     Running   0          14m
powerdns-7658b9f678-p78xg   1/1     Running   0          14m
powerdns-7658b9f678-qtrkx   1/1     Running   0          14m
```

#### 2.2 Check External IP Assignment
To see that MetalLB assigned an external IP from the configured IP range, use the following command:

```bash
kubectl get svc powerdns
```

This should display:

```bash
NAME       TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)                                        AGE
powerdns   LoadBalancer   10.43.41.23   <EXTERNAL_IP>   5353:31145/UDP,5353:31145/TCP,8081:30728/TCP   17m
```

### 3. Testing DNS 
To verify that MetalLB is routing traffic to your PowerDNS service, you can perform a simple DNS query using the `dig` command and the external IP assigned by MetalLB:

```bash
dig @<EXTERNAL_IP> -p 5353 example.com
```

If MetalLB is correctly configured and your PowerDNS service is functioning, you should receive a response with DNS information for the specified domain (if it exists).

### Deploying PowerDNSAdmin