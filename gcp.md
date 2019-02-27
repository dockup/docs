# Documentation

This documentation is broadly split into 3 parts:

- Setting up kubernetes cluster
  Prerequisite: Signup for GCP account
- Setting up docker image builds
- Deploying your apps


### Setting up kubernetes cluster.

First sign up for GCP account if you don't have one. You can do it by heading
over to [cloud page][gcp_console]. Once signed up, create a project called
"Dockup" and complete setting up billing.

**NOTE: You will get free $300 credit when you sign up for the first time**

After you setup your GCP account, head over to [kubernetes][gcp_kubernetes]
cluster page, enable kubernetes api, and create a cluster. Its okay to go
with all default values. Please make a note of name, and zone as we will
be using them later. Create page looks something like this:


![Kubernetes create page](/images/gcp_kubernetes_create.png)


Cluster creation will take sometime. Please wait till cluster creation is
complete.



[gcp_console]: http://console.cloud.google.com/
[gcp_kubernetes]: https://console.cloud.google.com/kubernetes/list
