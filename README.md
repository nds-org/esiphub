# ESIPHub

## Bootstrapping JupyterHub on SDSC Cloud

This describes the basic setup steps to get the Zero-to-JupyterHub helm chart running on a single-node Kubernetes environment on SDSC Cloud:

### OpenStack setup
* Created SSH and HTTP/HTTPS security groups
* Provisioned Ubuntu 16.04 VM, assigned public IP, SSH and HTTP/HTTPS security groups

### Bootstrap Kubernetes

Following the data8 instructions (https://github.com/data-8/kubeadm-bootstrap):

```
ssh ubuntu@IP
git clone https://github.com/data-8/kubeadm-bootstrap
cd kubeadm-bootstrap/
sudo ./install-kubeadm.bash
sudo -E ./init-master.bash
```

This process worked without error, but did require changing owndership of ~/ubuntu/.helm:

```
 sudo chown -R ubuntu:ubuntu ~/.helm
```

### DNS setup

Registered esiphub.ndslabs.org domain to point to instance public IP.

### Configure Oauth

Configured Github Oauth using https://esiphub.ndslabs.org/hub/oauth_callback

### Create config.yaml

Created the following config.yaml to customize JupyterHub deployment. This disables user storage until we have persistent volume claim support:
```
hub:
  # output of first execution of 'openssl rand -hex 32'
  cookieSecret: "XXX"
proxy:
  # output of second execution of 'openssl rand -hex 32'
  secretToken: "XXX"
auth:
  type: github
  github:
    clientId: "XXX"
    clientSecret: "XXX"
    callbackUrl: "https://esiphub.ndslabs.org/hub/oauth_callback"
singleuser:
  storage:
    type: none
```

### Create namespace and persistent volume:

JupyerHub requires a persistent volume for a sqlite database.  Since we don't have formal PVC support on this instance, create a simple persistent volume mounting a local path. 

pv.yaml
```
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-hub
  labels:
    type: local
spec:
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/data"
```

Create the namespace and volume:
```
kubectl create ns esiphub
mkdir /data
kubectl create -f pv.yaml
```


### Install helm chart

```
helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/
helm repo update

helm install jupyterhub/jupyterhub \
    --version=v0.6 \
    --name=esiphub \
    --namespace=esiphub \
    -f config.yaml
```

### Temporary workaround for loadbalancer

After deploying, edit the proxy-public service and change to Nodeport:

```
kubectl edit svc proxy-public --namespace=esiphub
```
Change type to NodePort and add externalIP:
```
apiVersion: v1
kind: Service
metadata:
  labels:
    chart: jupyterhub-v0.6
    component: proxy
    heritage: Tiller
    release: worn-kudu
  name: proxy-public
  namespace: default
spec:
  clusterIP: 10.97.121.5
  externalIPs:
  - 10.128.21.7
  externalTrafficPolicy: Cluster
  ports:
  - name: http
    nodePort: 30209
    port: 80
    protocol: TCP
    targetPort: 80
  - name: https
    nodePort: 31852
    port: 443
    protocol: TCP
    targetPort: 443
  selector:
    component: proxy
    name: proxy
    release: worn-kudu
  type: NodePort
```

Create simple ingress rule:
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: esiphub
spec:
  tls:
  - hosts:
    - esiphub.ndslabs.org
  rules:
  - host: esiphub.ndslabs.org
    http:
      paths:
      - path:
        backend:
          serviceName: proxy-public
          servicePort: 80
```

Note, there appears to be a better approach described here but not tested:
https://github.com/jupyterhub/zero-to-jupyterhub-k8s/issues/340

