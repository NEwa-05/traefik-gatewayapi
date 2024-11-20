# traefik-gatewayapi
testing gateway api with traefik

## Init env to connect to K8s platform and set variables

```bash
source .env
```

## Generate wildcard

```bash
LEGO_DISABLE_CNAME_SUPPORT=true lego -a --path .lego/. --email mageekbox@gmail.com --dns gandiv5 -d "${CLUSTERNAME}.${DOMAINNAME}" -d "*.${CLUSTERNAME}.${DOMAINNAME}" run
```

## Deploy Traefik Hub

### deploy K8s experimental crd for gatewayAPI

```bash
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.2.0/standard-install.yaml
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.2.0/experimental-install.yaml
```

### Create Traefik Hub namespace

```bash
kubectl create ns traefik
```

### deploy wildcard cert

```bash
kubectl create secret tls wildcard-mageekbox --namespace traefik --cert=.lego/certificates/${CLUSTERNAME}.${DOMAINNAME}.crt --key=.lego/certificates/${CLUSTERNAME}.${DOMAINNAME}.key
```

### Set Traefik Hub token

```bash
kubectl create secret generic hub-license --from-literal=token="${HUB_TOKEN}" -n traefik
```

### Set DNS Token for LE DNS Challenge

```bash
kubectl create secret generic dnsapikey --namespace traefik --from-literal=token=${GANDIV5_API_KEY}
```

### Deploy Traefik hub with helm chart

#### update helm repo

```bash
helm repo update
```

```bash
helm upgrade --install traefik traefik/traefik -n traefik --values hub-values.yaml
```
