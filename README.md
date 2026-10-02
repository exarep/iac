# Infrastructure as Code

Ansible playbooks for Exarep infrastructure on an RHDP AWS Blank Open Environment. DNS is managed in Route53; Cloudflare forwards to Route53. Existing services (Keycloak, Nexus) run on a shared EC2 host behind a host-header proxy on 443.

## Setup

1. Provision a new RHDP AWS Blank Open Environment
2. Clone this repository
3. Create and activate a Python virtual environment, then install dependencies
4. Install Ansible Galaxy collections
5. Configure AWS credentials (`aws configure`, region `us-east-2`)
6. Run playbooks

```shell
git clone <repository-url> iac
cd iac

python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

ansible-galaxy collection install -r requirements.yaml

aws configure

ansible-playbook playbooks/pb-validate-aws.yaml
```

## Details

### RHDP AWS Blank Open Environment

Create a temporary AWS Blank Open Environment from the Red Hat Demo Platform. Use the access key and secret from that environment with `aws configure`. Preferred region is `us-east-2`.

Playbooks assume credentials are available via the standard AWS credential chain (`~/.aws/credentials` or environment variables). Do not commit credentials, keys, or secrets into this repository.

### Python virtual environment

All Ansible execution should use the project `.venv`:

```shell
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### Ansible Galaxy collections

Collections are declared in `requirements.yaml`:

```shell
ansible-galaxy collection install -r requirements.yaml
```

### Repository layout

Follows the [Ansible sample setup](https://docs.ansible.com/projects/ansible/latest/tips_tricks/sample_setup.html), with playbooks kept under `playbooks/` (see `roles_path` in `ansible.cfg`):

```
hosts.yaml                 # inventory
group_vars/
  all.yaml                 # variables for all hosts
host_vars/                 # per-host variables
playbooks/
  pb-validate-aws.yaml
roles/
ansible.cfg
```

- `ansible.cfg` points at `hosts.yaml`
- Non-secret defaults live in `group_vars/all.yaml` (`aws_region`, domain hostnames)
- Playbooks are prefixed with `pb-`

### Playbooks

| Playbook                         | Purpose                                                         |
| -------------------------------- | --------------------------------------------------------------- |
| `playbooks/pb-validate-aws.yaml` | Verify AWS credentials and discover the default VPC and subnets |

```shell
ansible-playbook playbooks/pb-validate-aws.yaml
```

Expected success output includes the AWS account ID, caller ARN, and either discovered VPC/subnet IDs or a message that no VPCs exist yet (common for a fresh RHDP Blank Open Environment).
