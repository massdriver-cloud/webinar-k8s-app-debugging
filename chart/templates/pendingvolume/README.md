# Pending (Resources)

This pod is stuck in a pending state. This means that the kubernetes scheduler can't find an appropriate node to place the pod on. Let's investigate why.

First lets confirm thats the issue by checking on the pod.

```shell
kubectl get pod -n debugging -l app=pendingvolume
```

We've confirmed the pod is stuck in pending. Let's describe the pod to see if there are some useful events to help us understand why it can't be scheduled.

```shell
kubectl describe pod -n debugging -l app=pendingvolume
```

The events give us some helpful context:

```shell
Events:
  Type     Reason             Age                   From                Message
  ----     ------             ----                  ----                -------
  Warning  FailedScheduling   3m38s (x5 over 18m)   default-scheduler   0/3 nodes are available: 3 pod has unbound immediate PersistentVolumeClaims. preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling.
  Normal   NotTriggerScaleUp  3m30s (x91 over 18m)  cluster-autoscaler  pod didn't trigger scale-up: 1 pod has unbound immediate PersistentVolumeClaims
```

So it looks like there are unbound PersistantVolumeClaims (PVC). A PVC is kubernetes way of representing "persistant" storage in an environment where your application may get scheduled on any arbitrary node. When your pod moves from one node to another, kuberentes will handle the process of unbinding your volume from the old node, and binding it to the new node. It seems kubernetes is having issues with binding the volume. Let's investigate why.

```shell
kubectl get pvc -n debugging -l app=pendingvolume
```

We've confirmed the volume is stuck in a `Pending` state, just like the pod. Let's describe the PVC to see if there is some useful information about why.

```shell
kubectl describe pvc -n debugging -l app=pendingvolume
```

It looks like the storage class isn't found

```shell
Events:
  Type     Reason              Age                    From                         Message
  ----     ------              ----                   ----                         -------
  Warning  ProvisioningFailed  118s (x56 over 1h)  persistentvolume-controller  storageclass.storage.k8s.io "not-real-sc" not found
```

A storage class is a way of representing "types" of storage that are available for persistent volumes in the kubernetes cluster. For example, in AWS there can be one storage class for EBS volumes and another one for an EFS volume. Let's see what storage classes are available to us.

```shell
kubectl get storageclass
```

Select a storage class that is available and update line 32 of [`statefulset.yaml`](statefulset.yaml) to use that storage class. Let's try to redeploy.

```shell
helm upgrade debug . -n debugging -i --create-namespace
```

Looks like Kubernetes won't allow us to make that change.

```shell
Error: UPGRADE FAILED: cannot patch "debug-massdriver-debugging-webinar-pendvol" with kind StatefulSet: StatefulSet.apps "debug-massdriver-debugging-webinar-pendvol" is invalid: spec: Forbidden: updates to statefulset spec for fields other than 'replicas', 'template', 'updateStrategy', 'persistentVolumeClaimRetentionPolicy' and 'minReadySeconds' are forbidden
```

This is actually a good thing. Kubernetes only allows modifications to certain fields. Consider this: if we changed the storage class from one working storage class to another, that would actual create a problem. How is the data on the persistant volume going to be migrated from one volume to the other? Kubernetes actually prevented us from making a mistake here.

However, in this case we don't have any data to migrate. We can't update the statefulset in place, but we can destroy it and recreate it with a proper storage class.

```shell
kubectl delete statefulset -n debugging -l app=pendingvolume
kubectl delete pvc -n debugging -l app=pendingvolume
helm upgrade debug . -n debugging -i --create-namespace
```

Now the pod is running successfully.