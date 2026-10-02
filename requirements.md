# Infrastructure as Code

- The entire infrastructure lives in a temporary AWS Blank Open Environment provisioned on the Red Hat Demo Platform.
- All playbooks will have access to an AWS environment based on the `aws configure` run. 
- The domain lives in Cloudflare. DNS should be managed in the AWS environment with the Cloudflare DNS simply forwarding to the Route53 DNS. 
- Everything should be tied to SSO using the Identity Provider (IDP). 

## Existing Services 

There is a single EC2 instance that hosts Exarep's "existing" services. These are services typically hosted outside of OpenShift and are already in place. The single EC2 instance will host a myriad of services, which will require the host to have a proxy listening on port 443 and routing based on the request header. 

* Host - services.exarep.com
* Identity Provider - auth.exarep.com (Keycloak)
* Artifact Repository - artifacts.exarep.com (Sonatype Nexus Repository)