# Log of attempt to deploy K8S + Cinder on TACC and IU Jetstream

## TACC

Using the v3 credentials, all `openstack coe` commands fail with a similar error:

```
$ openstack coe cluster template list
SSL exception connecting to https://10 (HTTP 500) (Request-ID: req-36dc456e-b0bc-49e5-b11d-77850cf4d5a6)
```


## IU

Using v3 credentials (and after several attempts, see errors below), I was able to create a template and cluster with the  cinder volume driver:

```
openstack coe cluster template create \
	   --coe kubernetes \
	   --image Fedora-Atomic-27-20180419 \
	   --keypair ndslabs-ops-iu \
	   --external-network public \
	   --network-driver flannel \
	   --flavor m1.medium \
	   --master-flavor m1.small \
	   --docker-volume-size 20 \
	   --docker-storage-driver devicemapper \
	   --volume-driver cinder \
	   --floating-ip-enabled my-k8s-template
```

```
openstack coe cluster create \
	--cluster-template my-k8s-template \
	--master-count 1 \
	--node-count 1 \
	--keypair ndslabs-ops-iu \
	--master-flavor m1.small \
	--flavor m1.medium \
	--docker-volume-size 20 my-k8s-cluster1
```  

Per https://docs.openstack.org/magnum/latest/user/#storage, create a cinder volume:
```
cinder create --display-name=test-volume 1
```

Now, ssh to master and try to create an nginx pod mounting the volume `nginx.yaml`:
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: web
      image: nginx
      ports:
        - name: web
          containerPort: 80
          hostPort: 8081
          protocol: TCP
      volumeMounts:
        - name: html-volume
          mountPath: "/usr/share/nginx/html"
  volumes:
    - name: html-volume
      cinder:
        # Enter the volume ID below
        volumeID: 35e074ce-e640-4f48-a5b6-5b11b9bec2fb
        fsType: ext4        
```

```
kubectl create -f nginx

kubectl describe nginx
...
Events:
  Type     Reason                 Age              From                                            Message
  ----     ------                 ----             ----                                            -------
  Normal   Scheduled              3m               default-scheduler                               Successfully assigned aws-web to my-k8s-cluster1-jj54oumgtxev-minion-0
  Normal   SuccessfulMountVolume  3m               kubelet, my-k8s-cluster1-jj54oumgtxev-minion-0  MountVolume.SetUp succeeded for volume "default-token-mg9hw"
  Warning  FailedMount            1m               kubelet, my-k8s-cluster1-jj54oumgtxev-minion-0  Unable to mount volumes for pod "aws-web_default(f546b212-99e7-11e8-8283-fa163e9ce3cb)": timeout expired waiting for volumes to attach/mount for pod "default"/"aws-web". list of unattached/unmounted volumes=[html-volume]
  Warning  FailedMount            1m (x9 over 3m)  kubelet, my-k8s-cluster1-jj54oumgtxev-minion-0  MountVolume.SetUp failed for volume "html-volume" : exit status 32

journalctl
...
Aug 07 02:17:34 my-k8s-cluster1-jj54oumgtxev-master-0.novalocal runc[2463]: E0807 02:17:34.854626       1 reconciler.go:292] attacherDetacher.AttachVolume failed to start for volume "html-volume" (UniqueName: "kubernetes.io/cinder/35e074ce-e640-4f48-a5b6-5b11b9bec2fb") from node "my-k8s-cluster1-jj54oumgtxev-minion-0" : AttachVolume.NewAttacher failed for volume "html-volume" (UniqueName: "kubernetes.io/cinder/35e074ce-e640-4f48-a5b6-5b11b9bec2fb") from node "my-k8s-cluster1-jj54oumgtxev-minion-0" : wrong cloud type
```

Try creating a storage class and PVC:

`sc.yaml`:
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
  annotations:
    storageclass.beta.kubernetes.io/is-default-class: "true"
  labels:
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: EnsureExists
provisioner: kubernetes.io/cinder
```

```
$ kubectl create -f sc.yaml
storageclass "standard" created
```

`pvc.yaml`
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
 name: pvc-test
 annotations:
   volume.beta.kubernetes.io/storage-class: standard
spec:
 accessModes:
  - ReadWriteOnce
 resources:
   requests:
     storage: 5Gi
```

```
$ kubectl create -f pvc.yaml
persistentvolumeclaim "pvc-test" created

$ kubectl describe pvc pvc-test
Name:          pvc-test
Namespace:     default
StorageClass:  standard
Status:        Pending
Volume:
Labels:        <none>
Annotations:   volume.beta.kubernetes.io/storage-class=standard
               volume.beta.kubernetes.io/storage-provisioner=kubernetes.io/cinder
Finalizers:    []
Capacity:
Access Modes:
Events:
  Type     Reason              Age               From                         Message
  ----     ------              ----              ----                         -------
  Warning  ProvisioningFailed  0s (x4 over 34s)  persistentvolume-controller  Failed to provision volume with StorageClass "standard": Cloud provider not initialized properly

$journalctl
...
Aug 07 02:29:56 my-k8s-cluster1-jj54oumgtxev-master-0.novalocal runc[2463]: E0807 02:29:56.774779       1 cinder.go:200] Cloud provider not initialized properly
Aug 07 02:29:56 my-k8s-cluster1-jj54oumgtxev-master-0.novalocal runc[2463]: I0807 02:29:56.775287       1 event.go:218] Event(v1.ObjectReference{Kind:"PersistentVolumeClaim", Namespace:"default", Name:"pvc-test", UID:"973ca4bc-99e9-11e8-8283-fa163e9ce3cb", APIVersion:"v1", ResourceVersion:"9147", FieldPath:""}): type: 'Warning' reason: 'ProvisioningFailed' Failed to provision volume with StorageClass "standard": Cloud provider not initialized properly

```



### IU Errors
Several cluster create attempts failed with various errors:

Attempt 1:
```
| status_reason       | Resource CREATE failed: InternalServerError: resources.kube_masters.resources[0].resources.calico_service_deployment: An unexpected error prevented the server from fulfilling your request. (HTTP 500) (Request-ID: req-50adcf94-02a0-4cd6-9fb8-1ddfdfdcfd41)
...
| faults              | {'0': 'InternalServerError: resources[0].resources.calico_service_deployment: An unexpected error prevented the server from fulfilling your request. (HTTP 500) (Request-ID: req-50adcf94-02a0-4cd6-9fb8-1ddfdfdcfd41)', 'calico_service_deployment': 'InternalServerError: resources.calico_service_deployment: An unexpected error prevented the server from fulfilling your request. (HTTP 500) (Request-ID: req-50adcf94-02a0-4cd6-9fb8-1ddfdfdcfd41)', 'kube_masters': 'InternalServerError: resources.kube_masters.resources[0].resources.calico_service_deployment: An unexpected error prevented the server from fulfilling your request. (HTTP 500) (Request-ID: req-50adcf94-02a0-4cd6-9fb8-1ddfdfdcfd41)'} |
```

Attempt 2:
```
| faults              | {'0': 'Unauthorized: resources[0].resources.kube-minion: The request you have made requires authentication. (HTTP 401) (Request-ID: req-ebcdac3f-59c3-4bb9-8ec4-4e8c51df65b8)', 'kube_minions': 'Unauthorized: resources.kube_minions.resources[0].resources.kube-minion: The request you have made requires authentication. (HTTP 401) (Request-ID: req-ebcdac3f-59c3-4bb9-8ec4-4e8c51df65b8)', 'kube-minion': 'Unauthorized: resources.kube-minion: The request you have made requires authentication. (HTTP 401) (Request-ID: req-ebcdac3f-59c3-4bb9-8ec4-4e8c51df65b8)'} |
```

Attempt 3:
```
| faults              | {'secgroup_kube_minion': "OverQuotaClient: resources.secgroup_kube_minion: Quota exceeded for resources: ['security_group'].\nNeutron server returns request_ids: ['req-3986b1c5-b088-4762-86b9-43be29fbc9dd']"}
```
Looks like security groups weren't cleaned up on delete
