# JupyterHub + Kubernetes on Nebula (OpenStack)

The following is based on Andrea Zonca's [Deploy scalable JupyterHub with Kubernetes on Jetstream](https://zonca.github.io/2017/12/scalable-jupyterhub-kubernetes-jetstream.html) with minor modifications for NCSA Nebula.  We will provide a separate set of instruction using the Terraform/Kubespray process.  

This is a demonstration of how to setup a scalable JupyterHub cluster on NCSA Nebula using Rook for shared storage and the Zero-to-JupyterHub helm chart.  This starts with a 2-node cluster but the same process can be used to create larger clusters. Unlike Jetstream, Nebula does not provide DNS names for instances.

See also:
* [Zero-to-JupyterHub](https://zero-to-jupyterhub.readthedocs.io/en/latest/index.html)
* [Data-8 Kubeadm bootstrap](https://github.com/data-8/kubeadm-bootstrap)

## Create Two Ubuntu VMs

The [Data-8 Kubeadm bootstrap](https://github.com/data-8/kubeadm-bootstrap) process assumes Ubuntu 16.04.  Create two VMs -- one will serve as master/controller and the other as a worker. Both nodes will serve Rook volumes:

On Nebula (NDS-hackathon project) the following settings where used:

```
Instance name: esip-hub
Flavor: 1m.large
Instance count: 2
Boot from Image: Ubuntu 16.04
Key pair: ndslabs-ops
Security groups: default, HTTP/HTTPS, remote SSH
```
Associate a floating IP to both VMs.

## Install Kubernetes
Following the [Data-8 Kubeadm bootstrap](https://github.com/data-8/kubeadm-bootstrap) method, setup your master and worker nodes. This installs Kubernetes, configures `kubectl`, and the `helm` package manager. `kubectl` is the primary tool used to configure a Kuberntes cluster.

Setup master:
```
ssh ubuntu@node1

git clone https://github.com/data-8/kubeadm-bootstrap
cd kubeadm-bootstrap
sudo ./install-kubeadm.bash
sudo -E ./init-master.bash
sudo kubeadm token create --print-join-command
```

Wait for the services to start:
```
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                                    READY     STATUS    RESTARTS   AGE
kube-system   etcd-esiphub-1                                          1/1       Running   0          1m
kube-system   kube-apiserver-esiphub-1                                1/1       Running   0          1m
kube-system   kube-controller-manager-esiphub-1                       1/1       Running   0          2m
kube-system   kube-dns-6f4fd4bdf-7tbc5                                3/3       Running   0          2m
kube-system   kube-flannel-ds-hrx7x                                   1/1       Running   1          2m
kube-system   kube-proxy-cc9cp                                        1/1       Running   0          2m
kube-system   kube-scheduler-esiphub-1                                1/1       Running   0          1m
kube-system   tiller-deploy-69cb6984f-gt2hz                           1/1       Running   0          2m
support       support-nginx-ingress-controller-9gkwv                  1/1       Running   0          24s
support       support-nginx-ingress-default-backend-cb84895fb-xgsnq   1/1       Running   0          24s
```

Now, setup the worker node:
```
ssh ubuntu@node2

git clone https://github.com/data-8/kubeadm-bootstrap
cd kubeadm-bootstrap
sudo ./install-kubeadm.bash
```

Then run the join command from above:
```
kubeadm join --token <some-secret> <master-ip>:6443 --discovery-token-ca-cert-hash sha256:<another-secret>
```
 
Back on master (node1), confirm that you see both nodes:
```
$ sudo kubectl get nodes
NAME           STATUS     ROLES     AGE       VERSION
incorehub1-1   Ready      master    6m        v1.9.2
incorehub1-2   Ready    <none>    34s       v1.9.2
```

## Installing Rook

Distributed storage for container clusters on OpenStack is a challenging problems.  We have seen and tried multiple methods -- from NFS to GlusterFS and now Rook.  Andrea's Rook-based approach is shown here.

Via OpenStack Horizon or CLI, create two 20GB volumes and attach them to the two nodes. These will now be available on the nodes under `/dev/vdb`:

```
$ lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
...
vdb    253:16   0  20G  0 disk
```

Create the filesystem:
```
sudo mkfs -t ext4 /dev/vdb
sudo mount -t ext4 /dev/vdb /data
```

On master/node1, install Rook via helm chart:
```
sudo helm repo add rook-alpha https://charts.rook.io/alpha
sudo helm install rook-alpha/rook
```


Use `kubectl` to confirm pods are running:
```
$ sudo kubectl get pods
NAME                            READY     STATUS    RESTARTS   AGE
rook-agent-2v86r                1/1       Running   0          1h
rook-agent-7dfl9                1/1       Running   0          1h
rook-operator-88fb8f6f5-tss5t   1/1       Running   0          1h
```


To configure Rook:
```
wget https://raw.githubusercontent.com/zonca/jupyterhub-deploy-kubernetes-jetstream/master/storage_rook/rook-cluster.yaml
```

Modify this file and change `/vol_b` to `/data`.

Apply this configuration:
```
sudo kubectl create -f rook-cluster.yaml
```

And wait for the pods to be in a `Running` state:
```
$ sudo kubectl -n rook get pods
NAME                              READY     STATUS    RESTARTS   AGE
rook-api-68b87d48d5-xmkpv         1/1       Running   0          6m
rook-ceph-mgr0-5ddd685b65-kw9bz   1/1       Running   0          6m
rook-ceph-mgr1-5fcf599447-j7bpn   1/1       Running   0          6m
rook-ceph-mon0-g7xsk              1/1       Running   0          7m
rook-ceph-mon1-zbfqt              1/1       Running   0          7m
rook-ceph-mon2-c6rzf              1/1       Running   0          6m
rook-ceph-osd-82lj5               1/1       Running   0          6m
rook-ceph-osd-cpln8               1/1       Running   0          6m
```

Create the storage class:
```
wget https://raw.githubusercontent.com/zonca/jupyterhub-deploy-kubernetes-jetstream/master/storage_rook/rook-storageclass.yaml
sudo kubectl create -f rook-storageclass.yaml

sudo kubectl get storageclass
NAME         PROVISIONER     AGE
rook-block   rook.io/block   16s
```

### Setup Lets Encrypt
The `kube-lego` chart can be used to automatically create HTTPS certificates for Kubernetes ingress rules.

```
sudo helm install --name lego stable/kube-lego --namespace=support --set config.LEGO_EMAIL=<your email>
```

For non-production environments, setup a role:
```
sudo kubectl create clusterrolebinding add-on-cluster-admin --clusterrole=cluster-admin --serviceaccount=support:default
```

Wait for the pod:
```
$ sudo kubectl get pods -n support
NAME                                                    READY     STATUS    RESTARTS   AGE
lego-kube-lego-76f84957bf-vvjxj                         1/1       Running   0          21s
```

## Install JupyterHub
Finally, installing JupyterHub.  This step will require DNS, which we setup using our `ndslabs.org` domain.


Create a file `config_jupyterhub_heml.yaml`
```
hub:
  # output of second execution of 'openssl rand -hex 32'
  cookieSecret: ""
  db:
    type: sqlite-pvc
    pvc:
      accessModes:
        - ReadWriteOnce
      storage: 1Gi
      storageClassName: rook-block

proxy:
  # output of second execution of 'openssl rand -hex 32'
  secretToken: ""

singleuser:
  memory:
    guarantee: 1G
    limit: 2G
  storage:
    type: dynamic
    capacity: 1Gi
    dynamic:
      storageClass: rook-block

ingress:
  enabled: true
  annotations:
    kubernetes.io/tls-acme: "true"
  hosts:
    - esiphub.ndslabs.org
  tls:
   - hosts:
      - esiphub.ndslabs.org
     secretName: kubelego-tls-jupyterhub
```

Optionally, configure Oauth via Github:
```
auth:
  type: github
  github:
    clientId: "CLIENT_ID"
    clientSecret: "CLIENT_SECRET"
    callbackUrl: "https://esiphub.ndslabs.org/hub/oauth_callback"
```

And re-apply config:
```
sudo helm upgrade jup jupyterhub/jupyterhub -f config_jupyterhub_helm.yaml
````

## Test it out

Go to https://esiphub.ndslabs.org. With Github authentication enabled, you should be prompted to login and authorize the application. Select "Start Server" to start your Jupyter Notebook server.

To see the running notebook container:
```
$ kubectl get pods --namespace=jup
...
jup           jupyter-craig-2dwillis                                  1/1       Running   0          1m
```
