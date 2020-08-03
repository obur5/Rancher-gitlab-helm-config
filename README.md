# Rancher + Gitlab(helm) + Let's Encrypt  + Clouflare

Few day's ago I started to install GitLab in my Rancher cluster, I didn't found guides how to install, so I decided to write by my own

**Postgres needs to be installed with user gitlab and database gitlabhq_production, and add secret 'gitlab-postgres-secret' with 'password' key and gitlab user password as value

## Let's Encrypt |System|
### 1) Create secret with Clouflare API Token
```
Rancher -> Resources -> Secrets -> Add Secret
 Name: cloudflare-api-token-secret
 Key:  api-token
 Value: <API Token From Clouflare account (global)> 
```

### 2) Create ClusterIssuer
Create file and apply with
kubectl create -f <FILE_NAME>.yaml

```yaml
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt-production
spec:
  acme:
    # You must replace this email address with your own.
    # Let's Encrypt will use this to contact you about expiring
    # certificates, and issues related to your account.
    email: <HERE YOU EMAIL ADDRESS>
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      # Secret resource that will be used to store the account's private key.
      name: cloudflare-issuer-account-key
    solvers:
    - dns01:
        cloudflare:
          email: <CLOUDFLARE EMAIL ADDRESS>
          apiTokenSecretRef:
            name: cloudflare-api-token-secret
            key: api-token
```

### 3) Install CertManager
We need to add Jetstack repo https://charts.jetstack.io

Install CertManager (do not forget to read notes and apply settings from notes)

## Gitlab |default|
### Add Gitlab Chart
Name: Gitlab 

Link: https://charts.gitlab.io/ 

## Installing Gitlab
### Set labels
```
certmanager-issuer.email: <EMAIL FOR CERTS>
certmanager.install: false
gitlab-runner.install: false
gitlab.webservice.ingress.tls.secretName: <SECRET_NAME_FOR_KEYS>
global.edition: ce
global.hosts.domain: <DOMAIN_NAME>
global.ingress.annotations."cert-manager\.io/cluster-issuer": letsencrypt-production
global.ingress.annotations."kubernetes\.io/tls-acme": true
global.ingress.configureCertmanager: false
global.ingress.enabled: false
global.psql.host: <POSTGRES_HOST_NAME>
global.psql.password.key: password
global.psql.password.secret: gitlab-postgres-secret
nginx-ingress.enabled: false
persistence.enable: true
persistence.size: 50Gi <USE_YOU_OWN_VALUE>
postgresql.install: false
```
This will automatically create ssl certs and ingress rule



