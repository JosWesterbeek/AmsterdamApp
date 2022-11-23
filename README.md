# AAPP Helm charts

[AAPP] This repository contains [Helm charts](https://helm.sh/) to easily install the AAPP stack on a Kubernetes cluster.

## External authentication

The AAPP backoffice relies on an external OpenID Connect identity provider for authentication of users. If no instance is available, [Dex](https://github.com/dexidp/dex) could be used with a static user as follows:

```bash
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm upgrade --install \
  dex stable/dex \
  --create-namespace \
  --namespace dex \
  --set "config.issuer=https://dex.aapp.example.com" \
  --set "config.staticPasswords[0].email=aapp.admin@example.com" \
  --set "config.staticPasswords[0].hash=$2a$10$2b2cU8CPhOTaGrs1HRQuAueS7JTT5ZHsHSzYiFPm1leZck7Mc8T4W" \
  --set "config.staticPasswords[0].username=admin" \
  --set "config.staticClients[0].id=aapp" \
  --set "config.staticClients[0].name=AmsterdamApp" \
  --set "config.staticClients[0].secret=somethingsecret" \
  --set "config.staticClients[0].redirectURIs[0]=https://aapp.example.com/manage/incidents" \
  --set "config.oauth2.responseTypes={token,id_token}" \
  --set "ingress.enabled=true" \
  --set "ingress.hosts[0]=dex.aapp.example.com"
```

This command will install Dex in the dex namespace and exposes an [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) on dex.aapp.example.com. It creates the following static user:

- E-mail: aapp.admin@example.com
- Password: password

Dex has a lot of benefits when connected to an Identity Provider [using connectors](https://github.com/dexidp/dex#connectors). It provides out-of-the box support for LDAP, SAML2.0, GitHub, GitLab and more.

## Install the charts

First configure the Helm repository:

```bash
helm repo add aapp https://aapp.github.io/helm-charts/
helm repo update
```

Then install the backend chart:

```bash
helm upgrade --install \
  aapp-backend aapp/backend \
  --create-namespace \
  --namespace aapp \
  --set "settings.allowedHosts=api.aapp.example.com" \
  --set "settings.defaultPdokMunicipalities=Amsterdam" \
  --set "settings.jwksUrl=https://dex.aapp.example.com" \
  --set "settings.userIdField=email" \
  --set "ingress.enabled=true" \
  --set "ingress.hosts[0]=api.aapp.example.com"
```

And install the frontend chart:

```bash
helm upgrade --install \
  aapp-frontend aapp/frontend \
  --create-namespace \
  --namespace aapp \
  --set "oidc.authEndpoint=https://dex.aapp.example.com/auth" \
  --set "config.apiBaseUrl=https://api.aapp.example.com/aapp" \
  --set "ingress.enabled=true" \
  --set "ingress.hosts[0]=aapp.example.com"
```

```

## Uninstall the charts

To delete the charts:

```bash
helm delete --namespace aapp aapp-frontend
helm delete --namespace aapp aapp-backend
```

And finally remove the namespace:

```bash
kubectl delete namespace aapp
```

## Configuration

Consult the documentation of the specific charts for an overview of all configuration options:

- [backend](./charts/backend)
- [frontend](./charts/frontend)
- [mapserver](./charts/mapserver)