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
- Pull secret at `~/.pull-secret`
- SSH public key at `~/.ssh/id_rsa.pub`

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
