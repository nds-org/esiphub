# Notes from load testing ESIPhub

Per https://github.com/nds-org/esiphub/issues/7, this documents the process for running the (now archived?) `jupyterhub-loadtest` chart. 
Test 1 is a  20 user test to make sure the load test is performing as expected.  Test 2 is a more thorough 60 user test.

The chart is installed from source:
```
git clone https://github.com/yuvipanda/jupyterhub-loadtest
```

# Test 2
Under this test, `esiphub3` is using the nfs-provisioner for the hub claim and no persistent storage for users. All pods are started within 2 minutes and shutdown after 3 minutes of activity. The test worked as advertised with no apparent issues.

```
...
  storage:
    type: none
...
```

loadtest/values.yml
```
...
hub:
  url: "https://esiphub3.ndslabs.org"

users:
  count: 20
  startTime:
    max: 120
  runTime:
    min: 120
    max: 180
...
```

```
helm upgrade --install --wait --namespace=test3 test3 loadtest
```

```
$ kubectl get pods -n esiphub3
jupyter-jh-2dload-2dtest3-2dxv8gj0    1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dxv8gj1    1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dxv8gj10   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dxv8gj11   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dxv8gj12   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dxv8gj13   1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dxv8gj14   1/1       Running   0          3m
jupyter-jh-2dload-2dtest3-2dxv8gj15   1/1       Running   0          3m
jupyter-jh-2dload-2dtest3-2dxv8gj16   1/1       Running   0          3m
jupyter-jh-2dload-2dtest3-2dxv8gj17   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dxv8gj18   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dxv8gj19   1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dxv8gj2    1/1       Running   0          3m
jupyter-jh-2dload-2dtest3-2dxv8gj3    1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dxv8gj4    1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dxv8gj5    1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dxv8gj6    1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dxv8gj7    1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dxv8gj8    1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dxv8gj9    1/1       Running   0          1m
```

```
helm delete --purge test3
```


## Test 2
Same as above but for 60 users

```
helm upgrade --install --wait --namespace=test3 test3 loadtest
```

```
$ kubectl get pods -n esiphub3
NAME                                  READY     STATUS     RESTARTS   AGE
hub-d88f4657-9rfm5                    1/1       Running    2          48m
jupyter-jh-2dload-2dtest3-2dnw99z0    1/1       Running    0          37s
jupyter-jh-2dload-2dtest3-2dnw99z1    1/1       Running    0          25s
jupyter-jh-2dload-2dtest3-2dnw99z10   1/1       Running    0          45s
jupyter-jh-2dload-2dtest3-2dnw99z11   1/1       Running    0          16s
jupyter-jh-2dload-2dtest3-2dnw99z12   1/1       Running    0          30s
jupyter-jh-2dload-2dtest3-2dnw99z13   1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z14   1/1       Running    0          25s
jupyter-jh-2dload-2dtest3-2dnw99z15   1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z16   1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z17   1/1       Running    0          30s
jupyter-jh-2dload-2dtest3-2dnw99z18   0/1       Init:0/1   0          0s
jupyter-jh-2dload-2dtest3-2dnw99z19   1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z2    1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z20   1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z21   1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z22   1/1       Running    0          9s
jupyter-jh-2dload-2dtest3-2dnw99z23   1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z24   1/1       Running    0          59s
jupyter-jh-2dload-2dtest3-2dnw99z25   1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z26   1/1       Running    0          48s
jupyter-jh-2dload-2dtest3-2dnw99z27   1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z28   1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z29   1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z3    1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z30   1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z31   1/1       Running    0          29s
jupyter-jh-2dload-2dtest3-2dnw99z32   1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z33   1/1       Running    0          55s
jupyter-jh-2dload-2dtest3-2dnw99z34   1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z35   1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z36   1/1       Running    0          48s
jupyter-jh-2dload-2dtest3-2dnw99z37   1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z38   1/1       Running    0          50s
jupyter-jh-2dload-2dtest3-2dnw99z39   1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z4    1/1       Running    0          56s
jupyter-jh-2dload-2dtest3-2dnw99z40   1/1       Running    0          16s
jupyter-jh-2dload-2dtest3-2dnw99z41   1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z42   1/1       Running    0          27s
jupyter-jh-2dload-2dtest3-2dnw99z43   1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z44   1/1       Running    0          46s
jupyter-jh-2dload-2dtest3-2dnw99z45   1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z46   1/1       Running    0          40s
jupyter-jh-2dload-2dtest3-2dnw99z47   1/1       Running    0          17s
jupyter-jh-2dload-2dtest3-2dnw99z48   1/1       Running    0          44s
jupyter-jh-2dload-2dtest3-2dnw99z49   1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z5    1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z50   1/1       Running    0          23s
jupyter-jh-2dload-2dtest3-2dnw99z51   1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z52   1/1       Running    0          38s
jupyter-jh-2dload-2dtest3-2dnw99z53   1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z54   1/1       Running    0          14s
jupyter-jh-2dload-2dtest3-2dnw99z55   1/1       Running    0          39s
jupyter-jh-2dload-2dtest3-2dnw99z56   1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z57   1/1       Running    0          20s
jupyter-jh-2dload-2dtest3-2dnw99z58   1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z59   1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z6    1/1       Running    0          5s
jupyter-jh-2dload-2dtest3-2dnw99z7    1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z8    1/1       Running    0          1m
jupyter-jh-2dload-2dtest3-2dnw99z9    1/1       Running    0          50s
proxy-9d77c5577-pvvg7                 2/2       Running    0          48m
```

```
helm delete --purge test3
```


## Test 3
60 users using nfs-provisioner.

jupyterhub config
```
singleuser:
  memory:
    guarantee: 1G
    limit: 8G
  storage:
    type: dynamic
    capacity: 1Gi
    dynamic:
      storageClass: managed-nfs-storage
```      

```
helm upgrade --install --wait --namespace=test3 test3 loadtest
```

As with the prior test, all 60 instances are running within 2 minutes without error.

```
$ kubectl get pods -n esiphub3
NAME                                  READY     STATUS    RESTARTS   AGE
hub-597686b64d-gdrtg                  1/1       Running   0          7m
jupyter-jh-2dload-2dtest3-2dx2z840    1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z841    1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z8410   1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z8411   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z8412   1/1       Running   0          3m
jupyter-jh-2dload-2dtest3-2dx2z8413   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z8414   1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z8415   1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z8416   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z8417   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z8418   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z8419   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z842    1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z8420   1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z8421   1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z8422   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z8423   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z8424   1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z8425   1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z8426   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z8427   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z8428   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z8429   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z843    1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z8430   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z8431   1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z8432   1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z8433   1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z8434   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z8435   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z8436   1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z8437   1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z8438   1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z8439   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z844    1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z8440   1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z8441   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z8442   1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z8443   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z8444   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z8445   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z8446   1/1       Running   0          3m
jupyter-jh-2dload-2dtest3-2dx2z8447   1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z8448   1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z8449   1/1       Running   0          3m
jupyter-jh-2dload-2dtest3-2dx2z845    1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z8450   1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z8451   1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z8452   1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z8453   1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z8454   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z8455   1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z8456   1/1       Running   0          3m
jupyter-jh-2dload-2dtest3-2dx2z8457   1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z8458   1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z8459   1/1       Running   0          1m
jupyter-jh-2dload-2dtest3-2dx2z846    1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z847    1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z848    1/1       Running   0          2m
jupyter-jh-2dload-2dtest3-2dx2z849    1/1       Running   0          2m
proxy-9d77c5577-pvvg7                 2/2       Running   0          1h
```

All PVCs are bound and exist on the NFS mount:
```
$ kubectl get pvc -n esiphub3
NAME                                STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS          AGE
claim-jh-2dload-2dtest3-2dx2z840    Bound     pvc-9686914f-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z841    Bound     pvc-af357f67-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8410   Bound     pvc-a09b3626-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8411   Bound     pvc-7f1e4602-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   3m
claim-jh-2dload-2dtest3-2dx2z8412   Bound     pvc-70a55f52-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   3m
claim-jh-2dload-2dtest3-2dx2z8413   Bound     pvc-880c333f-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   3m
claim-jh-2dload-2dtest3-2dx2z8414   Bound     pvc-b1065790-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8415   Bound     pvc-a308fcf1-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8416   Bound     pvc-95ed25fc-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8417   Bound     pvc-75e9aa27-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   3m
claim-jh-2dload-2dtest3-2dx2z8418   Bound     pvc-79cd0cf5-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   3m
claim-jh-2dload-2dtest3-2dx2z8419   Bound     pvc-86f4a9c5-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   3m
claim-jh-2dload-2dtest3-2dx2z842    Bound     pvc-b3730700-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8420   Bound     pvc-a248e820-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8421   Bound     pvc-a13db312-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8422   Bound     pvc-8371733a-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   3m
claim-jh-2dload-2dtest3-2dx2z8423   Bound     pvc-8a56f949-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   3m
claim-jh-2dload-2dtest3-2dx2z8424   Bound     pvc-b3464a37-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8425   Bound     pvc-a54c6e98-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8426   Bound     pvc-94e9e86a-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8427   Bound     pvc-8e3a79fe-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   3m
claim-jh-2dload-2dtest3-2dx2z8428   Bound     pvc-956a5f67-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8429   Bound     pvc-927bf328-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z843    Bound     pvc-74312c1e-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   3m
claim-jh-2dload-2dtest3-2dx2z8430   Bound     pvc-8305cbb9-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   3m
claim-jh-2dload-2dtest3-2dx2z8431   Bound     pvc-a295cc7e-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8432   Bound     pvc-98223f61-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8433   Bound     pvc-ad235f8b-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8434   Bound     pvc-83ddd1f6-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   3m
claim-jh-2dload-2dtest3-2dx2z8435   Bound     pvc-8ac5cf0c-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   3m
claim-jh-2dload-2dtest3-2dx2z8436   Bound     pvc-98a240df-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8437   Bound     pvc-af8006cc-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8438   Bound     pvc-abab4a51-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8439   Bound     pvc-91115f0b-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z844    Bound     pvc-a5fd4db2-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8440   Bound     pvc-9ac64e7a-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8441   Bound     pvc-7ceecf30-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   3m
claim-jh-2dload-2dtest3-2dx2z8442   Bound     pvc-9d2f5a99-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8443   Bound     pvc-80ac99c8-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   3m
claim-jh-2dload-2dtest3-2dx2z8444   Bound     pvc-7ab63cdf-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   3m
claim-jh-2dload-2dtest3-2dx2z8445   Bound     pvc-87c00620-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   3m
claim-jh-2dload-2dtest3-2dx2z8446   Bound     pvc-73bc1faa-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   3m
claim-jh-2dload-2dtest3-2dx2z8447   Bound     pvc-9ddcd72e-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8448   Bound     pvc-9d92d75b-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8449   Bound     pvc-71f66ed4-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   3m
claim-jh-2dload-2dtest3-2dx2z845    Bound     pvc-7494a29f-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   3m
claim-jh-2dload-2dtest3-2dx2z8450   Bound     pvc-b3fdf780-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8451   Bound     pvc-9e52e894-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8452   Bound     pvc-aea674c4-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8453   Bound     pvc-9edbe1b8-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8454   Bound     pvc-81652fb9-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   3m
claim-jh-2dload-2dtest3-2dx2z8455   Bound     pvc-9a3882b9-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8456   Bound     pvc-6f888af2-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   3m
claim-jh-2dload-2dtest3-2dx2z8457   Bound     pvc-acca3a90-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z8458   Bound     pvc-7e64461e-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   3m
claim-jh-2dload-2dtest3-2dx2z8459   Bound     pvc-9c05e3ab-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z846    Bound     pvc-97dafb7f-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   2m
claim-jh-2dload-2dtest3-2dx2z847    Bound     pvc-77cd2bdc-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   3m
claim-jh-2dload-2dtest3-2dx2z848    Bound     pvc-84b8f542-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   3m
claim-jh-2dload-2dtest3-2dx2z849    Bound     pvc-7ec3af41-8afb-11e8-8696-fa163e61f35e   1Gi        RWO            managed-nfs-storage   3m
```

I tried logging in as a standard esip user while this was running, and the startup time was quite slow, but I was in the end able to access the instance.
