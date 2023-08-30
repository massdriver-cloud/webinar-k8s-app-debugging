# OOMKilled

This application is failing to run due to an `OOMKilled` error. The "OOM" is an abbreviation for "out of memory". This issue arises if the container attempts to acquire more memory than it is allowed to.

When defining the containers in your Kubernetes pod, you can specify two levels of CPU and memory resource allocation: `requests` and `limits`. Resource requests should always be specified, and they should be your best estimate of the expected resource utilization of the container. This helps the Kubernetes schedule find an approprate node to schedule your pod, as well as reserving those resources for the pod. Resource limits should be used sparingly, as they are a **hard** limit on CPU and/or memory available to the pod. Attempts to use CPU higher than the limit will result in CPU throttling. Attempts to access more memory than the resource limit will result in the pod being terminated with `OOMKilled` status.

First lets confirm thats the issue by checking on the pod.

```shell
kubectl get pod -n debugging -l app=oomkilled
```

Let's check on the pod configuration to see what the memory limit is set to.

```shell
kubectl get pod -n debugging -l app=oomkilled -o yaml
```

10 MiB seems a bit low. The script we are running requires right around 50MiB. Let's update the limit to 100MiB in [`deployment.yaml`](deployment.yaml) to give it plenty of headroom, and redeploy.

```shell
helm upgrade debug . -n debugging -i --create-namespace
```

Check the pod status again

```shell
kubectl get pod -n debugging -l app=oomkilled
```

Now the pod is running successfully.