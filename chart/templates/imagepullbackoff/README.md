# ImagePullBackOff

This application is failing to run due to an `ImagePullBackOff`. This means that Kubernetes is failing to pull the proper image, and it is entering an exponential backoff on retries.

First let's confirm that is the issue

```shell
kubectl get pod -n debugging -l app=imagepullbackoff
```

Failing to pull an image could be due to multiple issues, such as network connectivity, attempting to pull a private image without specifying credentials, or just a good old-fashioned fat-finger of the image name. Let's check the deployment manifest to see if we can see what is wrong.

```shell
kubectl get pod -n debugging -l app=imagepullbackoff -o yaml
```

If you check the image field, you'll see that the image name is `xnign`. That doesn't seem right! We probably meant `nginx`. Open the local [`deployment.yaml`](deployment.yaml), and check if that is the source of the issue. Delete or comment line 19, and uncomment line 20. Let's redeploy and see if that fixes it.

```shell
helm upgrade debug . -n debugging -i --create-namespace
```

Check the pod status again

```shell
kubectl get pod -n debugging -l app=imagepullbackoff
```

Now the pod is running successfully.