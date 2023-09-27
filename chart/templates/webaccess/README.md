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
helm upgrade debug . -n debugging
```

Check the pod status again

```shell
kubectl get pod -n debugging -l app=webaccess
```

Now we're ready. In another terminal, let's port-forward the pod:

```shell
kubectl port-forward pod/<pod name> 8080:80
```

In another terminal, let's curl localhost:8080 to see if we get a response
```shell
curl localhost:8080
```

We're getting a good response!

```shell
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Okay let's move up the stack to the service next. We can port-forward the service very similar

```shell
kubectl port-forward svc/<service name> 8080:80
```

Uh oh, this is hanging. Thats not a good sign. Let's take a look at the service status.

```shell
kubectl describe service -n debugging -l app=webaccess
```

Looks like the `endpoints` field list is empty. This means the service isn't mapped to any pods. Let's check the selector.

```shell
kubectl get service -n debugging -l app=webaccess -o yaml
```

Okay, we found it. The `app` label isn't supposed to be hyphenated. Let's fix that and redeploy. Let's port-forward again.

```shell
helm upgrade debug . -n debugging
```

Check the service again

```shell
kubectl describe service -n debugging -l app=webaccess
```

Now we have an endpoint. Let's port-forward the service and let's see if we're getting a response

```shell
kubectl port-forward svc/<service name> 8080:80
```

Curl in another terminal.

```shell
curl localhost:8080
```

Service is working, lets try our website. Still not working. Okay, last piece is the ingress, lets take a look!

```shell
kubectl get ingress -n debugging -l app=webaccess -o yaml
```

Looks like the port number is wrong. This is a great place to use the port “name” instead of number. Let’s fix and redeploy

```shell
helm upgrade debug . -n debugging
```

Now lets check the website. Working!