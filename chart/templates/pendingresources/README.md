# Pending (Resources)

This pod is stuck in a pending state. This means that the kubernetes scheduler can't find an appropriate node to place the pod on. Let's investigate why.

First lets confirm thats the issue by checking on the pod.

```shell
kubectl get pod -n debugging -l app=pendingresources
```

We've confirmed the pod is stuck in pending. Let's describe the pod to see if there are some useful events to help us understand why it can't be scheduled.

```shell
kubectl describe pod -n debugging -l app=pendingresources
```

The events give us some helpful context:

```shell
Events:
  Type     Reason             Age   From                Message
  ----     ------             ----  ----                -------
  Warning  FailedScheduling   29s   default-scheduler   0/3 nodes are available: 3 Insufficient cpu. preemption: 0/3 nodes are available: 3 No preemption victims found for incoming pod.
  Normal   NotTriggerScaleUp  20s   cluster-autoscaler  pod didn't trigger scale-up: 1 Insufficient cpu
```

It looks like it's a CPU constraint. The cluster can't find any available nodes with enough CPU, and the cluster-autoscaler is saying that adding a brand new node wouldn't address the issue. Let's check to see what the requested CPU resources are for the pod.

```shell
kubectl get pod -n debugging -l app=pendingresources -o yaml
```

20 CPUs seems like a lot. Let's see how many CPUs the nodes have.

```shell
kubectl get nodes
kubectl describe node <node name>
```

If we check the `Capacity` section, you'll see the CPU's available on the node.

```shell
Capacity:
  attachable-volumes-aws-ebs:  25
  cpu:                         2
  ephemeral-storage:           20959212Ki
  hugepages-1Gi:               0
  hugepages-2Mi:               0
  memory:                      3943368Ki
  pods:                        17
```

So the nodes only have 2 CPUs available. Technically probably less since there are "daemonsets" (which are pods that run on every node) with will require minimal resources. In [`deployment.yaml`](deployment.yaml) let's lower the requested resources to something more reasonable, like 1/10th of a CPU.

```shell
helm upgrade debug . -n debugging -i --create-namespace
```

Check the pod status again

```shell
kubectl get pod -n debugging -l app=pendingresources
```

Now the pod is running successfully.