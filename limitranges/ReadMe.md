# LimitRange

Create the demo namespace

```bash

kubectl apply -f podqos/src/main/kubernetes/010-khool-kpl.ns.yaml

```

## default & defaultRequest

### With guaranteed QoS

Let's create the Limit 

```bash
kubectl apply -f limitranges/src/main/kubernetes/020-lmr-cpu.yaml
# limitrange/d-limit-range created
```

```bash
kubectl apply -f limitranges/src/main/kubernetes/020-d-qos-guaranteed.deploy.yaml
#deployment.apps/d-qos-guaranteed created
kubectl -n khool-kpl-lmr get deploy,rs,po -l app=d-qos-guaranteed
# NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
# deployment.apps/d-qos-guaranteed   2/2     2            2           3s
#
# NAME                                         DESIRED   CURRENT   READY   AGE
# replicaset.apps/d-qos-guaranteed-77c95b654   2         2         2       3s
#
# NAME                                   READY   STATUS    RESTARTS   AGE
# pod/d-qos-guaranteed-77c95b654-9wmwz   1/1     Running   0          3s
# pod/d-qos-guaranteed-77c95b654-fljgt   1/1     Running   0          3s
kubectl -n khool-kpl-lmr get po -l app=d-qos-guaranteed -o custom-columns=NAME:.metadata.name,CPU_REQ:.spec.containers[0].resources.requests.cpu,CPU_LIM:.spec.containers[0].resources.limits.cpu,MEM_REQ:.spec.containers[0].resources.requests.memory,MEM_LIM:.spec.containers[0].resources.limits.memory
# NAME                               CPU_REQ   CPU_LIM   MEM_REQ   MEM_LIM
# d-qos-guaranteed-77c95b654-9wmwz   500m      500m      100Mi     100Mi
# d-qos-guaranteed-77c95b654-fljgt   500m      500m      100Mi     100Mi
```

Let's try a high request on CPU

```bash
kubectl apply -f limitranges/src/main/kubernetes/022-d-qos-guaranteed-upper.deploy.yaml
# deployment.apps/d-qos-guaranteed-upper created
kubectl -n khool-kpl-lmr get po -l app=d-qos-guaranteed-upper -o custom-columns=NAME:.metadata.name,CPU_REQ:.spec.containers[0].resources.requests.cpu,CPU_LIM:.spec.containers[0].resources.limits.cpu,MEM_REQ:.spec.containers[0].resources.requests.memory,MEM_LIM:.spec.containers[0].resources.limits.memory
# NAME                                      CPU_REQ   CPU_LIM   MEM_REQ   MEM_LIM
# d-qos-guaranteed-upper-64bb664f5b-c27gj   750m      2         100Mi     100Mi
# d-qos-guaranteed-upper-64bb664f5b-qcrgv   750m      2         100Mi     100Mi
```

Let's try a lower request on CPU

```bash
kubectl apply -f limitranges/src/main/kubernetes/023-d-qos-guaranteed-downer.deploy.yaml
# deployment.apps/d-qos-guaranteed-downer created
kubectl -n khool-kpl-lmr get po -l app=d-qos-guaranteed-downer -o custom-columns=NAME:.metadata.name,CPU_REQ:.spec.containers[0].resources.requests.cpu,CPU_LIM:.spec.containers[0].resources.limits.cpu,MEM_REQ:.spec.containers[0].resources.requests.memory,MEM_LIM:.spec.containers[0].resources.limits.memory
# NAME                                       CPU_REQ   CPU_LIM   MEM_REQ   MEM_LIM
# d-qos-guaranteed-downer-85d4c848cb-2zs4k   250m      250m      100Mi     100Mi
# d-qos-guaranteed-downer-85d4c848cb-zrrn7   250m      250m      100Mi     100Mi
```

### With BestEffort QoS

```bash
kubectl apply -f limitranges/src/main/kubernetes/031-d-qos-besteffort.deploy.yaml
# deployment.apps/d-qos-besteffort created
kubectl -n khool-kpl-lmr get po -l app=d-qos-besteffort -o custom-columns=NAME:.metadata.name,CPU_REQ:.spec.containers[0].resources.requests.cpu,CPU_LIM:.spec.containers[0].resources.limits.cpu,MEM_REQ:.spec.containers[0].resources.requests.memory,MEM_LIM:.spec.containers[0].resources.limits.memory
# NAME                                CPU_REQ   CPU_LIM   MEM_REQ   MEM_LIM
# d-qos-besteffort-74676dd9f8-cq6z7   500m      1         <none>    <none>
# d-qos-besteffort-74676dd9f8-g8sgf   500m      1         <none>    <none>
```

If there is no defaultRequest defined

```bash
kubectl delete -f limitranges/src/main/kubernetes/031-d-qos-besteffort.deploy.yaml
# deployment.apps "d-qos-besteffort" deleted
kubectl apply -f limitranges/src/main/kubernetes/032-lmr-cpu.yaml
# limitrange/d-limit-range configured
kubectl apply -f limitranges/src/main/kubernetes/031-d-qos-besteffort.deploy.yaml
# deployment.apps/d-qos-besteffort created
kubectl -n khool-kpl-lmr get po -l app=d-qos-besteffort -o custom-columns=NAME:.metadata.name,CPU_REQ:.spec.containers[0].resources.requests.cpu,CPU_LIM:.spec.containers[0].resources.limits.cpu,MEM_REQ:.spec.containers[0].resources.requests.memory,MEM_LIM:.spec.containers[0].resources.limits.memory
# NAME                                CPU_REQ   CPU_LIM   MEM_REQ   MEM_LIM
# d-qos-besteffort-74676dd9f8-gd2pb   1         1         <none>    <none>
# d-qos-besteffort-74676dd9f8-qmphv   1         1         <none>    <none>
```

## Max and Min

### With BestEffort QoS

Let's clean firts

```bash
kubectl -n khool-kpl-lmr delete deploy -l hol.khool.net/scope
# deployment.apps "d-qos-besteffort" deleted
# deployment.apps "d-qos-guaranteed-downer" deleted
# deployment.apps "d-qos-guaranteed-upper" deleted
# deployment.apps "d-qos-guaranteed" deleted
```

Configure with only min and max value

```bash
kubectl apply -f limitranges/src/main/kubernetes/033-lmr-cpu.yaml
# limitrange/d-limit-range configured
kubectl apply -f limitranges/src/main/kubernetes/031-d-qos-besteffort.deploy.yaml
#deployment.apps/d-qos-besteffort created
kubectl -n khool-kpl-lmr get po -l app=d-qos-besteffort -o custom-columns=NAME:.metadata.name,CPU_REQ:.spec.containers[0].resources.requests.cpu,CPU_LIM:.spec.containers[0].resources.limits.cpu,MEM_REQ:.spec.containers[0].resources.requests.memory,MEM_LIM:.spec.containers[0].resources.limits.memory
# NAME                                CPU_REQ   CPU_LIM   MEM_REQ   MEM_LIM
# d-qos-besteffort-74676dd9f8-5sbgm   1500m     1500m     <none>    <none>
# d-qos-besteffort-74676dd9f8-xbjxl   1500m     1500m     <none>    <none>
```

Configure the min and max value with default and defaultRequest

```bash
kubectl apply -f limitranges/src/main/kubernetes/040-lmr-cpu.yaml
# limitrange/d-limit-range configured
```

```bash
kubectl apply -f limitranges/src/main/kubernetes/031-d-qos-besteffort.deploy.yaml
# deployment.apps/d-qos-besteffort created
kubectl -n khool-kpl-lmr get po -l app=d-qos-besteffort -o custom-columns=NAME:.metadata.name,CPU_REQ:.spec.containers[0].resources.requests.cpu,CPU_LIM:.spec.containers[0].resources.limits.cpu,MEM_REQ:.spec.containers[0].resources.requests.memory,MEM_LIM:.spec.containers[0].resources.limits.memory
# NAME                                CPU_REQ   CPU_LIM   MEM_REQ   MEM_LIM
# d-qos-besteffort-74676dd9f8-79rp6   500m      1         <none>    <none>
# d-qos-besteffort-74676dd9f8-vmwvt   500m      1         <none>    <none>
```

### With Guaranteed QoS

```bash
kubectl apply -f limitranges/src/main/kubernetes/021-d-qos-guaranteed.deploy.yaml
#deployment.apps/d-qos-guaranteed created
kubectl -n khool-kpl-lmr get po -l app=d-qos-guaranteed -o custom-columns=NAME:.metadata.name,CPU_REQ:.spec.containers[0].resources.requests.cpu,CPU_LIM:.spec.containers[0].resources.limits.cpu,MEM_REQ:.spec.containers[0].resources.requests.memory,MEM_LIM:.spec.containers[0].resources.limits.memory
# NAME                               CPU_REQ   CPU_LIM   MEM_REQ   MEM_LIM
# d-qos-guaranteed-77c95b654-4l4xv   500m      500m      100Mi     100Mi
# d-qos-guaranteed-77c95b654-lgkmk   500m      500m      100Mi     100Mi
```

Let's try a hight request on CPU

```bash
kubectl apply -f limitranges/src/main/kubernetes/022-d-qos-guaranteed-upper.deploy.yaml
# deployment.apps/d-qos-guaranteed-upper created
kubectl -n khool-kpl-lmr get po -l app=d-qos-guaranteed-upper -o custom-columns=NAME:.metadata.name,CPU_REQ:.spec.containers[0].resources.requests.cpu,CPU_LIM:.spec.containers[0].resources.limits.cpu,MEM_REQ:.spec.containers[0].resources.requests.memory,MEM_LIM:.spec.containers[0].resources.limits.memory
# NAME                                      CPU_REQ   CPU_LIM   MEM_REQ   MEM_LIM
kubectl -n khool-kpl-lmr get events | grep Warning
# ...
# 78s         Warning   FailedCreate        replicaset/d-qos-guaranteed-upper-64bb664f5b    Error creating: pods "d-qos-guaranteed-upper-64bb664f5b-6f9jx" is forbidden: maximum cpu usage per Container is 1500m, but limit is 2
# 78s         Warning   FailedCreate        replicaset/d-qos-guaranteed-upper-64bb664f5b    Error creating: pods 
# 37s         Warning   FailedCreate        replicaset/d-qos-guaranteed-upper-64bb664f5b    (combined from similar events): Error creating: pods "d-qos-guaranteed-upper-64bb664f5b-rldj9" is forbidden: maximum cpu usage per Container is 1500m, but limit is 2
```

Let's try a lower request on CPU

```bash
kubectl apply -f limitranges/src/main/kubernetes/023-d-qos-guaranteed-downer.deploy.yaml
# deployment.apps/d-qos-guaranteed-downer created
kubectl -n khool-kpl-lmr get po -l app=d-qos-guaranteed-downer -o custom-columns=NAME:.metadata.name,CPU_REQ:.spec.containers[0].resources.requests.cpu,CPU_LIM:.spec.containers[0].resources.limits.cpu,MEM_REQ:.spec.container
s[0].resources.requests.memory,MEM_LIM:.spec.containers[0].resources.limits.memory
# NAME   CPU_REQ   CPU_LIM   MEM_REQ   MEM_LIM
kubectl -n khool-kpl-lmr get events | grep Warning
# ...
# 17s         Warning   FailedCreate        replicaset/d-qos-guaranteed-downer-85d4c848cb   Error creating: pods "d-qos-guaranteed-downer-85d4c848cb-9jmhl" is forbidden: minimum cpu usage per Container is 300m, but request is 250m
# 8s          Warning   FailedCreate        replicaset/d-qos-guaranteed-downer-85d4c848cb   (combined from similar events): Error creating: pods "d-qos-guaranteed-downer-85d4c848cb-qgl24" is forbidden: minimum cpu usage per Container is 300m, but request is 250m
```