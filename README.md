# Getting Started with OpenShift GitOps

OpenShift GitOps is an add-on on OpenShift which provides Argo CD and other tooling to enable teams to implement GitOps workflows for cluster configuration and application delivery. OpenShift GitOps provides [Argo CD](https://argo-cd.readthedocs.io/en/stable/) as the core of the GitOps workflow and [GitOps Application Manager CLI](https://github.com/redhat-developer/kam) in order to help developers bootstrap a GitOps workflow for delivering applications.

This repository contains a brief guide for trying out OpenShift GitOps.

## Install OpenShift GitOps 

Log into OpenShift Web Console as a cluster admin and navigate to the **Administrator** perspective and then **Operators** &rarr; **OperatorHub**. 

In the **OperatorHub**, search for *OpenShift GitOps* and follow the operator install flow to install it.

Once OpenShift GitOps is installed, an instance of Argo CD is automatically installed on the cluster in the `openshift-gitops` namespace and link to this instance is added to the application launcher in OpenShift Web Console.

## Log into Argo CD dashboard

Argo CD upon installation generates an initial admin password which is stored in a Kubernetes secret. In order to retrieve this password, run the following command to decrypt the admin password:

```
oc extract secret/openshift-gitops-cluster -n openshift-gitops --to=-
```

Click on Argo CD from the OpenShift Web Console application launcher and then log into Argo CD with `admin` username and the password retrieved from the previous step.

## Configure OpenShift with Argo CD

In the current Git repository, the [cluster](cluster/) directory contains OpenShift cluster configurations such as an OpenShift Web Console customization as well as namespaces that should be created. Let's configure Argo CD to recursively sync the content of the [cluster](cluster/) directory to the OpenShift cluster. Initially, we can set the sync policy to manual in order to be able to review changes before rolling out configurations to the cluster. 

In the Argo CD dashboard, click on the **New App** button to add a new Argo CD application that syncs a Git repository containing cluster configurations with the OpenShift cluster.

Enter the following details and click on **Create**.

* Application Name: `cluster-configs`
* Project: `default`
* Sync Policy: `Manual`
* Repository URL: `https://github.com/ralvares/openshift-gitops-hacking`
* Revision: `HEAD`
* Path: `cluster`
* Destination: `https://kubernetes.default.svc`
* Namespace: `default`
* Directory Recurse: `checked`

Looking at the Argo CD dashboard, you would notice that the **cluster-configs** Argo CD application is created by is out of sync, since we configured it with manual sync policy.

Click on the **Sync** button on the **cluster-configs** application and then on **Synchronize** button after reviewing the changes that will be rolled out to the cluster.

Once the sync is completed successfully, you would see that Argo CD reports a the configurations to be currently in sync with the Git repository and healthy. You can click on the **cluster-configs** application to check the details of sync resources and their status on the cluster. 

You can  check that a namespace called `hello-openshift` is created on the cluster with the label openshift.io/run-level.

Now that the configuration sync is in place, any changes in the Git repository will be automatically detect by Argo CD and would change the status of the **cluster-configs** to `OutOfSync`, which implies a drift from the desired configuration.

## Deploy Applications with Argo CD

In addition to configuring OpenShift clusters, many teams use GitOps workflows for continuous delivery and deploying applications in multi-cluster Kubernetes environments.

The [app](app/) directory in the current Git repository contains the Kubernetes manifests using Kustomize for deploying the sample hello-openshift application. Let's configure Argo CD to automatically and recursively deploy any changes made to these manifests on the OpenShift cluster in the `hello-openshift` namespace that was created by Argo CD in the previous step.

In the Argo CD dashboard, click on the **New App** button to add a new Argo CD application that syncs a Git repository containing cluster configurations with the OpenShift cluster.

Create a new Argo CD application by clicking on the **New App** button in the Argo CD dashboard and entering the following details.

* Application Name: `hello-openshift`
* Project: `default`
* Sync Policy: `Automatic`
* Self-heal: `checked`
* Repository URL: `https://github.com/ralvares/openshift-gitops-hacking`
* Revision: `HEAD`
* Path: `app`
* Destination: `https://kubernetes.default.svc`
* Namespace: `hello-openshift`
* Directory Recurse: `checked`

> You can also create the Argo CD application by importing the following file:
>  ```
>  oc create -f argo/app.yaml
>  ```

Because we set up the sync policy to `Automatic`, as soon as the Argo CD application is created, a sync is started in order to rollout the manifests to the `hello-openshift` namespace.

In the OpenShift Web Console, go to the **Developer** perspective to review the deployed application.





