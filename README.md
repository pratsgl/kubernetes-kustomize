## Getting Started With Kustomize
### Declarative management of Kubernetes resources with a hands-on example
Kubernetes is one of the well-known container orchestrators and has led to the development of an entire ecosystem around it. It has allowed organizations to manage their container applications with ease by providing several resources to manage container deployments, replicas, scaling, service discovery, and networking through a single API Interface.
Most organizations have multiple environments to develop and test these applications before deploying them into production. Configuration between these environments might differ, and there might be several aspects that you may want to tweak.
There are various ways to manage your Kubernetes resources for multiple environments, such as Helm and Kustomize. Helm is the most dynamic template-based Kubernetes resource manager. However, you may not need Helm for everything. Kustomize is a Kubernetes native method of managing your Kubernetes resource manifests for multiple environments. It works on the overlay principle to do that. Let’s understand both approaches in the next section.

### Template vs. Overlay
A template-based engine works on the principle of substituting variables with values. Helm, a template-based engine, allows you to define a generic template-based manifest by declaring variable placeholders at specific places of the generic manifest. You then use a variable file to substitute the variable with values.
An overlay-based engine works on the principle of find and replace, i.e., it searches for specific sections in the manifest and replaces them with the required values using a find, replace, and merge-based strategy. That allows you to create base manifest files, which are pretty much standard Kubernetes resource definition files and can be deployed on their own. You can then use Kustomize to further customize those files according to your requirements.

### How Kustomize Works
Kustomize uses a file called kustomization.yaml that contains declarative specifications to what resources need to be imported from what manifest files and what changes need to be made. Once it has processed the resources, it emits them to the standard output, which can be stored in a file or directly used with kubectl to apply it to a particular cluster.
One of the excellent use cases of Kustomize is to manage Kubernetes resources for multiple environments. For Kustomize to work in that scenario, you would need a base directory that would contain all manifest files with all the common elements and an overlays directory that contains all the differences for a particular environment.
To understand better, let’s look at a hands-on example.

### Prerequisites
You will need a Kubernetes cluster running Kubernetes version 1.14 or later and the kubectl CLI of a similar version.

You would also need to fork https://github.com/bharatmicrosystems/kustomize-example-app repository.

We would also need to install Kustomize. So, let’s look at that in the next section.

### Installing Kustomize
There are various ways to install Kustomize, which you can find in https://kubectl.docs.kubernetes.io/installation/kustomize/. In our case, we’ll use the binary method of installing it.
To do so, run the following commands:

```ssh
$ curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash
$ sudo mv kustomize /usr/bin/
```

As we’ve installed Kustomize, let’s understand the problem statement and what we’re planning to implement.

### Problem Statement

we have a web application that we want to deploy in several environments. It has the following environment-specific differences:
1. It should contain an env label corresponding to the environment it is being deployed on (i.e., dev, test, and prod).
2. It should be deployed on namespaces corresponding to the environment (i.e., dev, test, and prod).
3. The number of replicas in the dev environment should be one, two in test, and three in prod.
4. In the prod environment, we will implement a rolling update strategy with maxSurge 1 and maxUnavailable 1.

So, let’s go ahead and look at the directory structure for that.

### Directory Structure
We will create a ``` Deployment ``` and a ``` Service ``` resource in two files — ``` deployment.yaml ```, and ``` service.yaml``` . The directory structure looks like the following:

``` 
kustomize-example-app/
├── base
│   ├── deployment.yaml
│   ├── kustomization.yaml
│   └── service.yaml
└── overlays
    ├── dev
    │   ├── deployment.yaml
    │   ├── kustomization.yaml
    │   ├── namespace.yaml
    │   └── service.yaml
    ├── prod
    │   ├── deployment.yaml
    │   ├── kustomization.yaml
    │   ├── namespace.yaml
    │   └── service.yaml
    └── test
        ├── deployment.yaml
        ├── kustomization.yaml
        ├── namespace.yaml
        └── service.yaml
   ```
   
The ``` base ``` directory contains the base manifests and a ``` kustomization.yaml ``` file that contains all the common elements between the three environments. The ``` overlays ``` directory includes a directory each for ``` dev, test ```, and ``` prod ```. As we are simply changing the properties of both ``` Deployment ``` and ``` Service ``` resources in the dev, test, and prod environments, we have a manifest each for them. However, the manifests would only contain the differences between the resources.
Let’s look at the ``` base ``` directory first.

### The Base Configuration

In the ``` base ``` directory, the ``` deployment.yaml ``` file looks like the following:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

The ``` service.yaml ``` file looks like the below:

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    name: http
  type: LoadBalancer
  selector:
    app: nginx
```

Now, for Kustomize to work, we need to create a ``` kustomization.yaml ``` file that will look like the following:

```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
```

As we can see, the Kustomization file contains the ``` apiVersion ``` and kind attributes like any other Kubernetes resource. The file can also have several sections based on what you’re planning to do. In this case, as we’re just trying to import the ``` Deployment ``` and  ``` Service ``` in the Kustomization, we use the ``` resources ``` section to do so.
When we run the ``` kustomize build ``` command on the ``` base ``` directory, Kustomize emits a manifest yaml that is a combination of the two.

```
$ kustomize build base
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
  name: nginx-service
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
```

Now, let’s go ahead and let’s look at the environment-specific configs.

### The Environment-Specific Configurations
Now, we will focus on the ``` overlays ``` directory. We have directories within the overlays directory for the ``` dev, test ```, and ``` prod ``` environments. Let’s look at the contents of the ``` dev ``` directory first.

#### The dev configuration

The ``` namespace.yaml ``` file looks like the following:

```
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```
Now, this wasn’t there in the base configuration as the namespace will change with the environment.
Let’s look at the ``` kustomization.yaml ``` file now:

```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

bases:
- ../../base

commonLabels:
  env: dev

resources:
  - namespace.yaml

namespace: de
```

Here, you can see that there are several sections. Let’s look at each of them in detail:
* ``` bases ```  — This section contains the reference to the base directory.
* ``` commonLables ``` — This section defines any common labels that we want to apply to all resources generated by Kustomize. As per our requirement, we’ve given the label env:dev.
* ``` resources ``` — This section contains any additional resources we want to add to the Kustomize configuration. As we have the namespace.yaml file as an additional manifest file that we’ve specified here.
* ``` namespace ``` — This section defines any common namespace where the resources should be deployed. In this case, it is dev.

Now, let’s run ``` kustomize build ``` on the dev directory and see what we get:

```
$ kustomize build overlays/dev
apiVersion: v1
kind: Namespace
metadata:
  labels:
    env: dev
  name: dev
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
    env: dev
  name: nginx-service
  namespace: dev
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
    env: dev
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
    env: dev
  name: nginx-deployment
  namespace: dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
      env: dev
  template:
    metadata:
      labels:
        app: nginx
        env: dev
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
```
