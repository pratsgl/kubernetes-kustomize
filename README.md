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


