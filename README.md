# Getting GOD-MODE with OpenShift GitOps

OpenShift GitOps is an add-on on OpenShift which provides Argo CD and other tooling to enable teams to implement GitOps workflows for cluster configuration and application delivery. OpenShift GitOps provides [Argo CD](https://argo-cd.readthedocs.io/en/stable/) as the core of the GitOps workflow and [GitOps Application Manager CLI](https://github.com/redhat-developer/kam) in order to help developers bootstrap a GitOps workflow for delivering applications.

After installing the OpenShift GitOps operator, an instance of Argo CD is installed in the openshift-gitops namespace which has sufficient privileges for managing cluster configurations, the reason you can exploit it to get some extra privileges.. 

PS: CODE-REVIEW is super-important, dont overlook anything, TIP: even a simple annotation or LABEL :).

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

## Configure OpenShift with Argo CD - Creating Namespaces and rolebindings

In the current Git repository, the [cluster](cluster/) directory contains OpenShift cluster configurations such as an OpenShift Web Console customization as well as namespaces that should be created. Let's configure Argo CD to recursively sync the content of the [cluster](cluster/) directory to the OpenShift cluster. 

PS: the user developer will have admin access to the namespace hello-openshift, we will use the user developer to run a pod inside the namespace later on.

Create a new Argo CD application:

>  ```
>  oc create -f argo/cluster.yaml
>  ```

Once the sync is completed successfully, you would see that Argo CD reports a the configurations to be currently in sync with the Git repository and healthy. You can click on the **cluster-configs** application to check the details of sync resources and their status on the cluster. 

You can  check that a namespace called `hello-openshift` is created on the cluster with the label openshift.io/run-level.

Now that the configuration sync is in place, any changes in the Git repository will be automatically detect by Argo CD and would change the status of the **cluster-configs** to `OutOfSync`, which implies a drift from the desired configuration.

## Deploy GOD-MODE Applications

The [app](app/) directory in the current Git repository contains the Kubernetes manifests for deploying the sample hello-openshift application. If you are using openshift-local, LOGIN using the user developer ( NON_ADMIN), and you can deploy the superupperhack.yaml without a problem and no restrictions, therefore GOD-MODE activated. 


>  ```
>  oc apply -f app/superupperhack.yaml
>  ```






