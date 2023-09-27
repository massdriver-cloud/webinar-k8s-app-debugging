# CrashLoopBackOff

This application will fail from a CrashLoopBackOff. To debug and fix the issue, follow these steps:

Check on the pod status.

```shell
kubectl get pod -n debugging -l app=crashloopbackoff
```

You'll see that the application is in a CrashLoopBackOff. This means that the pod is starting to run, crashing, being restarted by Kubernetes, and continuing to fail. Thats the "CrashLoop" part. The "BackOff" means that Kubernetes will wait an increasing amount of time (up to 5 minutes by default) between restarts to prevent unnecessary thrashing of workloads.

A pod crashing is the result of one or more docker containers within the pod crashing. This means the problem is an application error, not a kubernetes error. To diagnose the application, let's check the application logs:

```shell
kubectl logs -n debugging -l app=crashloopbackoff
```

You should see an error similar to the following:

```shell
Error: Database is uninitialized and superuser password is not specified.
       You must specify POSTGRES_PASSWORD to a non-empty value for the
       superuser. For example, "-e POSTGRES_PASSWORD=password" on "docker run".

       You may also use "POSTGRES_HOST_AUTH_METHOD=trust" to allow all
       connections without a password. This is *not* recommended.

       See PostgreSQL documentation about "trust":
       https://www.postgresql.org/docs/current/auth-trust.html
```

It looks like we failed to specify a necessary environment variable `POSTGRES_PASSWORD`. Open the [`deployment.yaml`](deployment.yaml), and uncomment lines 20-22 which will set the `POSTGRES_PASSWORD` environment variable. Redeploy the helm chart by running:

```shell
helm upgrade debug . -n debugging -i --create-namespace
```

Check the pod status again

```shell
kubectl get pod -n debugging -l app=crashloopbackoff
```

And you'll see it should be running healthy now.