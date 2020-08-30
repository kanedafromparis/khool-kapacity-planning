# ResourceQuotas

Create the demo namespace

```bash

kubectl apply -f ressourcesquotas/src/main/kubernetes/010-khool-kpl-resourcequota.ns.yaml
# namespace/khool-kpl-resourcequota created
```

# Simple ResourceQuotas

```bash
# deployment.apps/d-resourcequota-hard configured
kubectl -n khool-kpl-resourcequota get deploy,rs,pod
# NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
# deployment.apps/d-resourcequota-hard   3/3     3            3           7h36m
# 
# NAME                                              DESIRED   CURRENT   READY   AGE
# replicaset.apps/d-resourcequota-hard-55d468bd54   3         3         3       7h36m
# 
# NAME                                        READY   STATUS    RESTARTS   AGE
# pod/d-resourcequota-hard-55d468bd54-4cpr8   1/1     Running   0          7h30m
# pod/d-resourcequota-hard-55d468bd54-hrndl   1/1     Running   0          7h30m
# pod/d-resourcequota-hard-55d468bd54-kn69s   1/1     Running   0          7h30m
```

Let's apply the hard quota for 2 pods and try to scale the deploy to 4

```bash
kubectl apply -f ressourcesquotas/src/main/kubernetes/021-d-hard-pod-nb.quotas.yaml
# resourcequota/demo-quota created
# 08:32:26 kubectl -n khool-kpl-resourcequota get deploy,rs,pod
# NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
# deployment.apps/d-resourcequota-hard   3/3     3            3           7h36m
# 
# NAME                                              DESIRED   CURRENT   READY   AGE
# replicaset.apps/d-resourcequota-hard-55d468bd54   3         3         3       7h36m
# 
# NAME                                        READY   STATUS    RESTARTS   AGE
# pod/d-resourcequota-hard-55d468bd54-4cpr8   1/1     Running   0          7h31m
# pod/d-resourcequota-hard-55d468bd54-hrndl   1/1     Running   0          7h31m
# pod/d-resourcequota-hard-55d468bd54-kn69s   1/1     Running   0          7h31m
kubectl -n khool-kpl-resourcequota scale deployment.apps/d-resourcequota-hard --replicas=4
# deployment.apps/d-resourcequota-hard scaled
# 
# NAME                                              DESIRED   CURRENT   READY   AGE
# replicaset.apps/d-resourcequota-hard-55d468bd54   4         3         3       7h37m
# 
# NAME                                        READY   STATUS    RESTARTS   AGE
# pod/d-resourcequota-hard-55d468bd54-4cpr8   1/1     Running   0          7h31m
# pod/d-resourcequota-hard-55d468bd54-hrndl   1/1     Running   0          7h31m
# pod/d-resourcequota-hard-55d468bd54-kn69s   1/1     Running   0          7h31m
# 08:33:05 kubectl -n khool-kpl-resourcequota get events | grep Warning
# 8s          Warning   FailedCreate        replicaset/d-resourcequota-hard-55d468bd54   (combined from similar events): Error creating: pods "d-resourcequota-hard-55d468bd54-89hc2" is forbidden: exceeded quota: demo-quota, requested: pods=1, used: pods=3, limited: pods=2
# 12m         Warning   FailedCreate        replicaset/d-resourcequota-hard-55d468bd54   Error creating: pods "d-resourcequota-hard-55d468bd54-4n5lt" is forbidden: exceeded quota: demo-quota, requested: pods=1, used: pods=3, limited: pods=2
# 18s         Warning   FailedCreate        replicaset/d-resourcequota-hard-55d468bd54   Error creating: pods "d-resourcequota-hard-55d468bd54-gb8mp" is forbidden: exceeded quota: demo-quota, requested: pods=1, used: pods=3, limited: pods=2
# 18s         Warning   FailedCreate        replicaset/d-resourcequota-hard-55d468bd54   Error creating: pods "d-resourcequota-hard-55d468bd54-c4bd2" is forbidden: exceeded quota: demo-quota, requested: pods=1, used: pods=3, limited: pods=2
# 17s         Warning   FailedCreate        replicaset/d-resourcequota-hard-55d468bd54   Error creating: pods "d-resourcequota-hard-55d468bd54-d74sc" is forbidden: exceeded quota: demo-quota, requested: pods=1, used: pods=3, limited: pods=2
kubectl -n khool-kpl-resourcequota get po -l hol.khool.net/scope=qos-burstable
# NAME                                    READY   STATUS    RESTARTS   AGE
# d-resourcequota-hard-55d468bd54-4cpr8   1/1     Running   0          7h32m
# d-resourcequota-hard-55d468bd54-hrndl   1/1     Running   0          7h32m
# d-resourcequota-hard-55d468bd54-kn69s   1/1     Running   0          7h32m
kubectl -n khool-kpl-resourcequota delete po -l hol.khool.net/scope=qos-burstable
# pod "d-resourcequota-hard-55d468bd54-4cpr8" deleted
# pod "d-resourcequota-hard-55d468bd54-hrndl" deleted
# pod "d-resourcequota-hard-55d468bd54-kn69s" deleted
kubectl -n khool-kpl-resourcequota get po -l hol.khool.net/scope=qos-burstable
# NAME                                    READY   STATUS    RESTARTS   AGE
# d-resourcequota-hard-55d468bd54-qjbmc   1/1     Running   0          3s
# d-resourcequota-hard-55d468bd54-t4jz4   1/1     Running   0          3s
kubectl -n khool-kpl-resourcequota get po -l hol.khool.net/scope=qos-burstable
# NAME                                    READY   STATUS    RESTARTS   AGE
# d-resourcequota-hard-6dd6457558-bzvlt   1/1     Running   0          23s
```

Let's switch to requests and limits quotas

```bash
kubectl apply -f ressourcesquotas/src/main/kubernetes/031-d-hard-cpu-mem.quotas.yaml
# resourcequota/demo-quota configured
kubectl -n khool-kpl-resourcequota delete po -l hol.khool.net/scope=qos-burstable
# pod "d-resourcequota-hard-55d468bd54-qjbmc" deleted
# pod "d-resourcequota-hard-55d468bd54-t4jz4" deleted
kubectl -n khool-kpl-resourcequota get po -l hol.khool.net/scope=qos-burstable
# No resources found in khool-kpl-resourcequota namespace.
kubectl -n khool-kpl-resourcequota get events | grep Warning
# 24s         Warning   FailedCreate        replicaset/d-resourcequota-hard-55d468bd54   (combined from similar events): Error creating: pods "d-resourcequota-hard-55d468bd54-ljp4h" is forbidden: failed quota: demo-quota: must specify limits.cpu,requests.cpu
# ...
```

Let's define requests.cpu and limits.cpu.

```bash
kubectl apply -f ressourcesquotas/src/main/kubernetes/032-d-resourcequota-hard.deploy.yaml
# deployment.apps/d-resourcequota-hard configured
kubectl -n khool-kpl-resourcequota get events | grep Warning
# ...
# 3s          Warning   FailedCreate        replicaset/d-resourcequota-hard-6dd6457558   Error creating: pods "d-resourcequota-hard-6dd6457558-pkzl9" is forbidden: exceeded quota: demo-quota, requested: requests.cpu=250m, used: requests.cpu=250m, limited: requests.cpu=250m
# 2s          Warning   FailedCreate        replicaset/d-resourcequota-hard-6dd6457558   Error creating: pods "d-resourcequota-hard-6dd6457558-mv85b" is forbidden: exceeded quota: demo-quota, requested: requests.cpu=250m, used: requests.cpu=250m, limited: requests.cpu=250m
# 2s          Warning   FailedCreate        replicaset/d-resourcequota-hard-6dd6457558   (combined from similar events): Error creating: pods "d-resourcequota-hard-6dd6457558-7swm6" is forbidden: exceeded quota: demo-quota, requested: requests.cpu=250m, used: requests.cpu=250m, limited: rerequests.cpu=250m
kubectl -n khool-kpl-resourcequota get resourcequota
# NAME         AGE   REQUEST                                                 LIMIT
# demo-quota   21m   requests.cpu: 500m/250m, requests.memory: 100Mi/250Mi   limits.cpu: 1/3, limits.memory: 200Mi/1Gi
```

## Notice Openshift (DeploymentConfig)



```bash
oc new-project khool-kpl-resourcequota
# Now using project "khool-kpl-resourcequota" on server "https://master:8443".
# ...
oc apply -f ressourcesquotas/src/main/kubernetes/031-d-hard-cpu-mem.quotas.yaml
# resourcequota/demo-quota created
oc get resourcequota
# NAME         CREATED AT
# demo-quota   2020-07-30T19:12:25Z
# oc apply -f ressourcesquotas/src/main/kubernetes/042-d-resourcequota-hard.dc.yaml
# deploymentconfig.apps.openshift.io/dc-resourcequota-hard created
oc get po
#No resources found.
oc get events
# LAST SEEN   FIRST SEEN   COUNT     NAME                                     KIND               SUBOBJECT   TYPE      REASON        SOURCE                        MESSAGE
#28s         28s          1         dc-resourcequota-hard.1630219b859199c4   DeploymentConfig               Normal    DeploymentCreated   deploymentconfig-controller   Created new replication controller "dc-resourcequota-hard-1" for version 1
#7s          28s          13        dc-resourcequota-hard.1630219b86c78a5f   DeploymentConfig               Warning   FailedCreate        deployer-controller           Error creating deployer pod: pods "dc-resourcequota-hard-1-deploy" is forbidden: failed quota: demo-quota: must specify limits.cpu,limits.memory,requests.cpu,requests.memory)
```

deploymentconfig-controller create a lauching pod that is affect by quotas
