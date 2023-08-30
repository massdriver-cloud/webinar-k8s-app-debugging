# Web Accessibility

In order to fully complete this scenario you'll need an ingress controller (settings assume the default [ingress-nginx](https://github.com/kubernetes/ingress-nginx) ingress controller), along with [external-dns](https://github.com/kubernetes-sigs/external-dns) and [cert-manager](https://cert-manager.io/) installed in the kubernetes cluster.

In this scenario, we have a deployment, service and ingress defined which should allow us to reach the website from an internet browser.

```shell
kubectl get pod -n debugging -l app=webaccess
```

Notice that under "Ready" it said `0/1`. We know this is probably due to a bad readiness check. Let's investigate.

```shell
kubectl describe pod -n debugging -l app=webaccess
```

Looks like we're getting connection refused:
```shell
Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  ...
  Normal   Created    20m                  kubelet            Created container main
  Normal   Started    20m                  kubelet            Started container main
  Warning  Unhealthy  15s (x139 over 20m)  kubelet            Readiness probe failed: Get "http://10.1.19.249:81/": dial tcp 10.1.19.249:81: connect: connection refused
```

Unlike earlier where we were getting a 404, this means that the port doesn't seem to be open or available. Let's check the pod readiness check.

```shell
kubectl get pod -n debugging -l app=webaccess -o yaml
```

There is the culprit! We're opening port 80, but putting the readiness check on 81. It's a port mismatch. This is a great opportunity to use the port "name" so we only have to define the port number once, and we can reference it by name everywhere else. Let's patch that and deploy.

```shell
helm upgrade debug . -n debugging -i --create-namespace
```

Check the pod status again

```shell
kubectl get pod -n debugging -l app=webaccess
```

Now we're ready. In another terminal, let's port-forward the pod:

```shell
kubectl port-forward pod/<pod name> 8080:80
```

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
kubectl get pod -n debugging -l app=webaccess -o yaml
```

Looks like we are using an invalid endpoint. Let's open [`deployment.yaml`](deployment.yaml) and fix it. Ideally your application should have a dedicated health endpoint, but in this case since we are running a base nginx image we can just use the root path `/`. Delete or comment line 22 and uncomment the line below it and redeploy.

```shell
helm upgrade debug . -n debugging -i --create-namespace
```

Check the pod status again

```shell
kubectl get pod -n debugging -l app=webaccess
```

And you'll see that ready is now `1/1`.