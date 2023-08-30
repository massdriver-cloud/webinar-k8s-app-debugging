# Debugging Applications in Kubernetes

This webinar will walk you through the process of debugging failures in applications running in a Kubernetes cluster. The scenarios this webinar will investigate are:

* CrashLoopBackOff
* Failing Readiness Checks
* ImagePullBackOff
* OOMKilled
* Pods stuck in "Pending" state

While this is far from an exhaustive list, by investigating and addressing these issues it should provide a foundation for resolving other issues you may encounter while running applications in Kubernetes.

## Setup

You'll need a running Kubernetes cluster. For information on deploying a Kubernetes cluster using [Massdriver](https://massdriver.cloud), check out our [video tutorial](https://www.youtube.com/watch?v=seRBnT-Axfw).

You'll also need to install [Helm](https://helm.sh/) and [kubectl](https://kubernetes.io/docs/reference/kubectl/).
* For instructions on installing Helm, [check out the Helm docs](https://helm.sh/docs/intro/install/).
* For instructions on installing kubectl, [check out the Kubernetes docs](https://kubernetes.io/docs/tasks/tools/).

Once the tools are installed, cd into the `chart` directory, and install the applications using the following commands:

```shell
cd chart
helm upgrade debug . -n debugging -i --create-namespace
```

This command will install the applications into your kubernetes cluster in the namespace `debugging` using the release name `debug` (if needed feel free to change either of these values to something more appropriate for your cluster).

If you check on the install, you should see a list of failing pods.

```shell
kubectl get pods -n debugging
```

## Debugging

Information on how to debug each of the failing applications is in a README in each of the applications subdirectory.

* [CrashLoopBackOff](chart/templates/crashloopbackoff)
* [Failed Readiness Check](chart/templates/failedreadiness)
* [ImagePullBackOff](chart/templates/imagepullbackoff)
* [OOMKilled](chart/templates/oomkilled)
* [Pod stuck in pending (insufficent resources)](chart/templates/pendingresources)
* [Pod stuck in pending (persistent volume binding)](chart/templates/pendingvolume)
