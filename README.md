# Infrastructure as Code

Ansible playbooks for Exarep infrastructure on an RHDP AWS Blank Open Environment. Route53 hosts the zone; Cloudflare A records forward to the services host Elastic IP. Existing services (Keycloak, Nexus) run on a shared EC2 host behind a host-header proxy on 443.

## Setup

1. Clone this repository
2. Provision a new RHDP AWS Blank Open Environment
3. Configure AWS credentials (`aws configure`, region `us-east-2`)
4. Create and activate a Python virtual environment, then install dependencies
5. Install Ansible Galaxy collections
6. Create gitignored `local_vars.yaml` with secrets (see example below)
7. Run playbooks in order

```shell
git clone https://github.com/exarep/iac.git iac
cd iac

python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

ansible-galaxy collection install -r requirements.yaml

aws configure

cp local_vars.example.yaml local_vars.yaml
# edit local_vars.yaml with Cloudflare token and generated secrets

ansible-playbook playbooks/pb-validate-aws.yaml
ansible-playbook playbooks/pb-services-host.yaml
ansible-playbook playbooks/pb-services-stack.yaml -e @local_vars.yaml
```

## Details

### Secrets (`local_vars.yaml`)

Never commit secrets. `local_vars.yaml` is gitignored. Start from `local_vars.example.yaml`:

```yaml
cloudflare_api_token: "replace-me"
keycloak_admin_user: admin
keycloak_admin_password: "generate-a-strong-password"
keycloak_db_password: "generate-a-strong-password"
oauth2_proxy_client_secret: "generate-a-strong-password"
oauth2_proxy_cookie_secret: "generate-a-32-byte-urlsafe-secret"
nexus_admin_password: "generate-a-strong-password"
idp_username: demo
idp_password: "replace-me"
```

SSO users get the `exarep-admin` role (admin without Nexus self-service password change). Keep the local Nexus `admin` account for break-glass access.

### RHDP AWS Blank Open Environment

Create a temporary AWS Blank Open Environment from the Red Hat Demo Platform. Use the access key and secret from that environment with `aws configure`. Preferred region is `us-east-2`.

### Repository layout

Follows the [Ansible sample setup](https://docs.ansible.com/projects/ansible/latest/tips_tricks/sample_setup.html), with playbooks kept under `playbooks/`:

```
hosts.yaml
group_vars/all.yaml
host_vars/
playbooks/
  pb-validate-aws.yaml
  pb-services-host.yaml
  pb-services-stack.yaml
roles/
  services_host/
  services_dns/
  services_stack/
ansible.cfg
```

### Playbooks

| Playbook                            | Purpose                                                                |
| ----------------------------------- | ---------------------------------------------------------------------- |
| `playbooks/pb-validate-aws.yaml`    | Verify AWS credentials and discover VPCs/subnets                       |
| `playbooks/pb-services-host.yaml`   | Provision VPC, security group, key pair, RHEL EC2, and Elastic IP      |
| `playbooks/pb-services-stack.yaml`  | DNS, Caddy TLS proxy, Keycloak, Nexus, oauth2-proxy SSO                |

```shell
ansible-playbook playbooks/pb-validate-aws.yaml
ansible-playbook playbooks/pb-services-host.yaml
ansible-playbook playbooks/pb-services-stack.yaml -e @local_vars.yaml
```

### Endpoints

| Hostname                 | Service                                      |
| ------------------------ | -------------------------------------------- |
| `https://services.exarep.com`   | Services host landing page            |
| `https://auth.exarep.com`       | Keycloak identity provider            |
| `https://artifacts.exarep.com`  | Nexus (SSO via Keycloak)              |

Open `https://artifacts.exarep.com`, sign in at Keycloak with the `idp_username` / `idp_password` from `local_vars.yaml`, and land in Nexus.

SSH to the host:

```shell
ssh -i ~/.ssh/exarep-services.pem -o IdentitiesOnly=yes ec2-user@<public-ip>
```
