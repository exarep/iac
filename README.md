# exarep - Infrastructure as Code

This repository is the bootstrap for the Exarep application portfolio.

## OpenShift Clusters

| Cluster    | Role                                          |
| ---------- | --------------------------------------------- |
| Services   | Quay registry, OpenShift Pipelines            |
| Management | ACM, GitOps, Hosted Control Planes            |
| Workload   | Development, Testing, Production (via labels) |

## DNS Architecture

Cloudflare is authoritative for `exarep.com`. A subdomain `cluster.exarep.com` is delegated to an AWS Route 53 hosted zone where external-dns manages records for OpenShift clusters and applications.

```
exarep.com                     ← Cloudflare, authoritative
│
└── cluster.exarep.com         ← NS delegation to Route 53
      ├── api.svc.cluster.exarep.com
      ├── apps.svc.cluster.exarep.com
      ├── api.mgmt.cluster.exarep.com
      ├── apps.mgmt.cluster.exarep.com
      └── ... workload cluster records
```

## Getting Started

### Prerequisites

- Python 3
- AWS CLI configured (`aws configure`) with Route 53 and EC2 permissions
- Cloudflare API token with DNS edit permissions for `exarep.com`
- `openshift-install` CLI (for cluster creation)
- Pull secret at `~/.pull-secret.txt`
- SSH key pair at `~/.ssh/exarep`
- Red Hat registry (registry.redhat.io) credentials (for Keycloak image)

### Setup

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
ansible-galaxy collection install -r collections/requirements.yaml
```

### AWS Credentials

```bash
aws configure
```

### Cloudflare API Token

The Cloudflare API token is read at runtime from the file path defined by `cloudflare_api_token_file` in `inventory/group_vars/all/dns.yaml`.

## Playbooks

### pb-dns-setup.yaml

Creates the Route 53 hosted zone for `cluster.exarep.com` and configures NS delegation records in Cloudflare.

```bash
ansible-playbook playbooks/pb-dns-setup.yaml
```

### pb-cluster-create.yaml

Creates an OpenShift cluster on AWS using IPI. Cluster definitions are in `inventory/group_vars/all/clusters.yaml`.

```bash
ansible-playbook playbooks/pb-cluster-create.yaml -e cluster_name=svc
ansible-playbook playbooks/pb-cluster-create.yaml -e cluster_name=mgmt
```

Install artifacts (kubeconfig, kubeadmin password) are saved to `.temp/clusters/<cluster_name>/`.

### pb-services-host-create.yaml

Provisions the EC2 services host on AWS with a security group, Elastic IP, and an A record for `auth.exarep.com` in Cloudflare. Instance configuration is in `inventory/group_vars/all/ec2.yaml`.

```bash
ansible-playbook playbooks/pb-services-host-create.yaml
```

### pb-services-host-configure.yaml

Configures the services host with Nginx, Keycloak, and PostgreSQL. Requires the services host to be provisioned first. Prompts for Red Hat registry credentials, Keycloak admin password, and database password at runtime.

```bash
ansible-playbook playbooks/pb-services-host-configure.yaml
```

This playbook:
- Installs Podman, Nginx, and certbot
- Deploys Keycloak and PostgreSQL as Podman containers
- Obtains a Let's Encrypt TLS certificate for `auth.exarep.com`
- Configures Nginx as a reverse proxy on port 443
- Creates the `customer` and `administrator` realms in Keycloak
