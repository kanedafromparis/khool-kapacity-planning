# Pod Qos

Create the demo namespace

```bash

kubectl apply -f podqos/src/main/kubernetes/010-khool-kpl.ns.yaml

```

## QoS Guaranteed

request and limits are identical

```bash

kubectl apply -f podqos/src/main/kubernetes/020-qos-guaranteed.pod.yaml
# pod/qos-guaranteed created
kubectl -n khool-kpl-podqos get po qos-guaranteed -o yaml | grep qosClass -C 5
#   hostIP: 192.168.99.119
#   phase: Pending
#   podIP: 172.17.0.3
#   podIPs:
#   - ip: 172.17.0.3
#   qosClass: Guaranteed
#   startTime: "2020-07-29T10:47:41Z"
```

## QoS Burstable

At least one request or limits is define for at least one containers

```bash

kubectl apply -f podqos/src/main/kubernetes/022-qos-burstable.pod.yaml
#pod/qos-burstable created
kubectl -n khool-kpl-podqos get po qos-burstable -o yaml | grep qosClass -C 5
#  hostIP: 192.168.99.119
#  phase: Running
#  podIP: 172.17.0.4
#  podIPs:
#  - ip: 172.17.0.4
#  qosClass: Burstable
#  startTime: "2020-07-29T11:04:40Z"
```

## QoS BestEffort

Neither resource request nor resource limits is define for any containers

```bash

kubectl apply -f podqos/src/main/kubernetes/024-qos-besteffort.pod.yaml
# pod/qos-besteffort created
kubectl -n khool-kpl-podqos get po qos-besteffort -o yaml | grep qosClass -C 5
#  hostIP: 192.168.99.119
#  phase: Running
#  podIP: 172.17.0.5
#  podIPs:
#  - ip: 172.17.0.5
#  qosClass: BestEffort
#  startTime: "2020-07-29T11:11:02Z"
```

## Flow

Remember that thoses pods useally are created by upper object, for instance deploy will create replicatset that will create pods.

```bash

kubectl apply -f podqos/src/main/kubernetes/030-d-qos-guaranteed.deploy.yaml
# deployment.apps/d-qos-guaranteed created
kubectl apply -f podqos/src/main/kubernetes/032-d-qos-burstable.deploy.yaml
# deployment.apps/d-qos-burstable created
kubectl apply -f podqos/src/main/kubernetes/034-d-qos-besteffort.deploy.yaml
# deployment.apps/d-qos-besteffort created
kubectl -n khool-kpl-podqos get deploy
# NAME               READY   UP-TO-DATE   AVAILABLE   AGE
# d-qos-besteffort   2/2     2            2           31s
# d-qos-burstable    2/2     2            2           36s
# d-qos-guaranteed   2/2     2            2           46s
kubectl -n khool-kpl-podqos get rs
# NAME                          DESIRED   CURRENT   READY   AGE
# d-qos-besteffort-74676dd9f8   2         2         2       37s
# d-qos-burstable-55d468bd54    2         2         2       42s
# d-qos-guaranteed-77c95b654    2         2         2       52s
kubectl -n khool-kpl-podqos get po
# NAME                                READY   STATUS    RESTARTS   AGE
# d-qos-besteffort-74676dd9f8-c92hw   1/1     Running   0          41s
# d-qos-besteffort-74676dd9f8-lnngr   1/1     Running   0          41s
# d-qos-burstable-55d468bd54-4q97b    1/1     Running   0          46s
# d-qos-burstable-55d468bd54-4z2rd    1/1     Running   0          46s
# d-qos-guaranteed-77c95b654-dtw9p    1/1     Running   0          56s
# d-qos-guaranteed-77c95b654-wh77n    1/1     Running   0          56s
# qos-besteffort                      1/1     Running   0          16m
# qos-burstable                       1/1     Running   0          13m
# qos-guaranteed                      1/1     Running   0          14m
kubectl -n khool-kpl-podqos get po -l app=d-qos-guaranteed -o yaml | grep qosClass -C 1
#    - ip: 172.17.0.6
#    qosClass: Guaranteed
#    startTime: "2020-08-29T20:36:53Z"
#--
#--
#    - ip: 172.17.0.7
#    qosClass: Guaranteed
#    startTime: "2020-08-29T20:36:53Z"
kubectl -n khool-kpl-podqos get po -l app=d-qos-burstable -o yaml | grep qosClass -C 1
#    - ip: 172.17.0.11
#    qosClass: Burstable
#    startTime: "2020-08-29T20:37:03Z"
#--
#--
#    - ip: 172.17.0.10
#    qosClass: Burstable
#    startTime: "2020-08-29T20:37:03Z"
kubectl -n khool-kpl-podqos get po -l app=d-qos-besteffort -o yaml | grep qosClass -C 1
#    - ip: 172.17.0.13
#    qosClass: BestEffort
#    startTime: "2020-08-29T20:37:08Z"
#--
#--
#    - ip: 172.17.0.12
#    qosClass: BestEffort
#    startTime: "2020-08-29T20:37:08Z"
```