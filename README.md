# Cert Manager + Let's Encrypt + traefik
cert-manager adds certificates and certificate issuers as resource types in Kubernetes clusters, and simplifies the process of obtaining, renewing and using those certificates.

# Introduction
Cert-manager is a Kubernetes add-on designed to assist with the creation and management of TLS certificates. Similar to [Certbot](https://www.linode.com/docs/guides/secure-http-traffic-certbot/), cert-manager can automate the process of creating and renewing self-signed and signed certificates for a large number of use cases, with a specific focus on container orchestration tools like Kubernetes and it can also be used with Kubernetes ingress like traefik. This documentation contains the boilerplate for installing traefik and cert-manager with Cloudflare DNS provider.

## Installing Traefik Proxy on kubernetics.
Traefik is a leading modern reverse proxy and load balancer that makes deploying microservices easy. Traefik integrates with your existing infrastructure components and configures itself automatically and dynamically.

### Refer to the instructions underneath to install the Traefik proxy.
Clone the repo and change the directory to cert-manager.
```bash
git clone https://github.com/techchad2022/cert-manager.git
cd cert-manager
```
Add Traefik's chart repository to Helm:
```bash
helm repo add traefik https://helm.traefik.io/traefik
```

You can update the chart repository by running:

```bash
helm repo update
```

And install it with the  `helm`  command line:

```bash
helm install --values=traefik/values.yml traefik traefik/traefik -n kube-system
```
Generate basic authentication for middleware to expose traefik dashboard. (requires apache2-utils).
```bash
htpasswd -nb admin password321 | openssl base64
```
Copy the generated text and replace users: <> with the previously obtained hash. Example is provided below
```bash
apiVersion: v1
kind: Secret
metadata:
	name: traefik-dashboard
	namespace: kube-system
type: Opaque
data:
	users: YWRtaW46JGFwcjEkbmJlMlNLTlgkaFFkNUpNY2EwcWdHRlRmbHJiUmxiLwoK
```
Apply the secret-dashboard.yml, middleware and ingress.
```bash
kubectl apply -f traefik/secret-dashboard.yml
kubectl apply -f traefik/middleware.yml
kubectl apply -f traefik/ingress.yml
```


## Installing Cert-Manager
Install cert-manager.crds.yaml
```bash
kubectl apply -f cert-manager/cert-manager.crds.yaml
```
Create a namespace named cert-manager
```bash
kubectl apply -f cert-manager/namespace.yml
```
Install cert-manager
```bash
 helm install cert-manager jetstack/cert-manager --namespace cert-manager --values=values.yml --version v1.9.1
```
### Before applying issuers
Update the issuers/secret-cf-token.yml with your api-token key obtained from cloudflare.
[Creating API tokens in cloudflare](https://developers.cloudflare.com/api/tokens/create/)
 Example: 
```bash
apiVersion: v1
kind: Secret
metadata:
	name: cloudflare-api-token-secret
	namespace: cert-manager
type: Opaque
stringData:
	api-token: X34pb0EQLChGdEHGNoHDu4EipfrfMYVMxPUfzmJI
```
update dnsZones in letsencrypt-production.yml and letsencrypt-staging.yml

Apply issuers.
```bash
kubectl apply -f cert-manager/issuers/.
```

### Certificates
The boilerplate for certificates are located in cert-manager/certificates please update the dnsNames before applying them.

# Author
name: techchad2022

email: techchad2022@gmail.com
