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


## Test 3
60 users using nfs-provisioner.


