# OpenShift and Kubernetes

## Overview

You write YAML files that describe what you'd like to have.

You apply those YAML files to Kubernetes.

Kubernetes stores your data in a database called `etcd` - this stores the desired state of your application.

The Kubernetes control plane looks at `etcd`, looks at the nodes, then makes the nodes match the desired state.

## Project Setup

### Create the config directory

`cd` into the project you created in the docker lesson and make a `k8s` folder:

```
cd ~/static-site
mkdir k8s
code .
```

### Create the OpenShift Project

Make sure you are logged into the cluster, either with `icc` or via `Copy Login Command`. 

Then run the following command, replacing `<NAME>` with your name, all lowercase, no spaces:

```
oc new-project <NAME>
```

For example `oc new-project john-smith`

## Kubernetes Objects

### Deployments

The most basic Kubernetes object you'll work with is a Deployment, which tells Kubernetes:

- which image you would like to run
- how many replicas (instances) you would like to run

#### Create the file

1. Create a file named `k8s/deployment.yaml`
1. Paste the following YAML into the file
1. Replace the word `<USERNAME>` with your quay.io username

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: static-site
  labels:
    app: static-site
spec:
  replicas: 1
  selector:
    matchLabels:
      app: static-site
  template:
    metadata:
      labels:
        app: static-site
    spec:
      containers:
      - name: static-site
        image: quay.io/<USERNAME>/static-site:v1
        ports:
        - containerPort: 8080
```

#### Apply the file

```
oc apply -f k8s/deployment.yaml
```

#### Verify the deployment works

```
oc get deploy
```

You should see the following:

```
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
static-site   1/1     1            1           24s
```

### Pods

A Deployment creates Pods. You can see pods by running:

```
oc get pods
```

### Services

Services allow pods within a Kubernetes cluster to access the pods from your deployment.

#### Create the file

Create a file named `k8s/service.yaml`

Paste the following YAML into it:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: static-site
spec:
  selector:
    app: static-site
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

#### Apply the file

```
oc apply -f k8s/service.yaml
```

#### Verify 

```
oc get svc
```

You should see something like this (the IP will be different):

```
NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
static-site   ClusterIP   172.21.141.7   <none>        80/TCP    2m6s
```

### Routes

Routes are an OpenShift concept. They serve a similar purpose to Kubernetes Ingresses.

#### Create the file

Create a file named `k8s/route.yaml` and paste the following contents:

```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: static-site
spec:
  tls:
    termination: edge
  to:
    kind: Service
    name: static-site
    weight: 100
  wildcardPolicy: None
```

#### Apply

```
oc apply -f k8s/route.yaml
```

#### Verify

```
oc get routes
```

This will print something like this:

```
NAME          HOST/PORT                       PATH  SERVICES      PORT    TERMINATION   WILDCARD
static-site   static-site-...appdomain.cloud        static-site   <all>   edge          None
```

Copy the `HOST/PORT` and open it in a browser. You should be able to see your site! 🎉

## UI

Inspect the app in the UI

## Cleanup

Run the following command to delete your project, replacing `<NAME>` with your project name:

```
oc project default
oc delete <NAME>
```

## References

- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Services](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Routes](https://docs.openshift.com/container-platform/3.9/architecture/networking/routes.html)