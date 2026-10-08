# Infrastructure as Code

- The entire infrastructure lives in a temporary AWS Blank Open Environment provisioned on the Red Hat Demo Platform.
- All playbooks will have access to an AWS environment based on the `aws configure` run. 
- The domains live in Cloudflare. DNS should be managed in the AWS environment with the Cloudflare DNS simply forwarding to the Route53 DNS. 
- Everything should be tied to SSO using the Identity Provider (IDP). 

## Domains

There are two domains: `exarep.com` and `intexarep.com`. External endpoints such as the marketing website, customer portal, external API endpoints should all use the `exarep.com` domain for their endpoints. For internal endpoints, the subdomain `intexarep.com` should be used. 

## Existing Services 

There is a single EC2 instance that hosts Exarep's "existing" services. These are services typically hosted outside of OpenShift and are already in place. The single EC2 instance will host a myriad of services, which will require the host to have a proxy listening on port 443 and routing based on the request header. 

* Hostname - svcinte - svcinte.intexarep.com
* Identity Provider - auth.intexarep.com (Keycloak)
* Artifact Repository - artifacts.intexarep.com (Sonatype Nexus Repository)

## All Services

* Internal Identity Provider - Keycloak - https://auth.intexarep.com
  * Realms: administrator
* Artifact Repository - Sonatype Nexus Repository - https://artifacts.intexarep.com
  * SSO through administrator realm
* Build Pipelines - OpenShift Pipelines - https://build.intexarep.com

### Clusters

Cluster DNS uses `type.env.intexarep.com` (role first, environment second). Shared services stay flat on `*.intexarep.com` and do not use this pattern.

- Hub clusters use type `mgmt`.
- Spoke/workload clusters use type `wl`.
- Preprod hub env is `preprod`; preprod workload envs are `dev` and `test` (not `wl.preprod`).
- Prod hub and workload both use env `prod`.

#### Preproduction

* Management Cluster (hub) - `mgmt.preprod.intexarep.com` - https://console.apps.mgmt.preprod.intexarep.com
* Development Cluster (spoke) - `wl.dev.intexarep.com` - https://console.apps.wl.dev.intexarep.com
* Test Cluster (spoke) - `wl.test.intexarep.com` - https://console.apps.wl.test.intexarep.com

#### Production

* Management Cluster (hub) - `mgmt.prod.intexarep.com` - https://console.apps.mgmt.prod.intexarep.com
* Production Cluster (spoke) - `wl.prod.intexarep.com` - https://console.apps.wl.prod.intexarep.com

## Internal 

* Administration Portal - Internal administration portal - https://administrator.intexarep.com
* Internal API Endpoint - Internal Data API Endpoint - https://api.intexarep.com

## External

* Public Portal - Marketing and New Customer Sign Up - https://exarep.com and https://www.exarep.com 
* Public API Endpoint - Public Data API Endpoint - https://api.exarep.com
