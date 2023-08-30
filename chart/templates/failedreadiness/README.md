# Failed Readiness Probe

This application begin running, but the readiness check will fail to pass. [Readiness probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-readiness-probes) are useful for informing the Kubernetes cluster when your pod is "ready" to serve traffic. If a pod isn't ready, it won't receive traffic.

Check on the pod status.

```shell
kubectl get pod -n debugging -l app=failedreadiness
```

Notice that under "Ready" it said `0/1`. This means that 0 out of 1 pods are in the "Ready" state. There are multiple ways for pods to implement readiness checks, but in this case we are using a HTTP test. Kubernetes will periodically call an HTTP endpoint and expect to receive a 2XX or 3XX status code. Any other code will be treated as "not ready".

Let's dig in a little further by "describing" the pod to get more detailed status.

```shell
kubectl describe pod -n debugging -l app=failedreadiness
```

Scroll to the bottom and look under the events section. You should see something like:

```shell
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  ...
  Normal   Created    28s                kubelet            Created container main
  Normal   Started    28s                kubelet            Started container main
  Warning  Unhealthy  1s (x10 over 25s)  kubelet            Readiness probe failed: HTTP probe failed with statuscode: 404
```

It appears our readiness check is failing because of a `404` status, which means the endpoint doesn't exist. Let's check on the pod configuration

```shell
kubectl get pod -n debugging -l app=failedreadiness -o yaml
```

Looks like we are using an invalid endpoint. Let's open [`deployment.yaml`](deployment.yaml) and fix it. Ideally your application should have a dedicated health endpoint, but in this case since we are running a base nginx image we can just use the root path `/`. Delete or comment line 22 and uncomment the line below it and redeploy.

```shell
helm upgrade debug . -n debugging -i --create-namespace
```

Check the pod status again

```shell
kubectl get pod -n debugging -l app=failedreadiness
```

And you'll see that ready is now `1/1`.