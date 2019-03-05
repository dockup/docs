# GCP instructions

### Create a kubernetes cluster

First sign up for GCP account if you don't have one. You can do it by heading
over to [cloud page][gcp_console]. Once signed up, create a project called
"dockup" and complete setting up billing.

**NOTE: You will get free $300 credit when you sign up for the first time**

After you setup your GCP account, head over to [kubernetes][gcp_kubernetes]
cluster page, enable kubernetes api, and create a cluster. It's okay to go
with all default values. Please make a note of name, and zone as we will
be using them later. Create page looks something like this:


![Kubernetes create page](/images/gcp_kubernetes_create.png)


Cluster creation will take sometime. Please wait till cluster creation is
complete.


### Verify connection to cluster

Install kubernetes and helm to get started. Follow these instructions:

~~~sh
# Install google cloud sdk on Mac
> brew cask install google-cloud-sdk

# Authorize yourself so that you can access projects
> gcloud auth application-default login

# See if you can get list of projects. You should see the project that
# you have created here.
> gcloud projects list

# Set the project as context. Useful for talking to kubernetes cluster
> gcloud config set project dockup

# Get credentials for kubernetes cluster created.
> gcloud container clusters get-credentials <name-of-cluster> \
  --region=<region-of-cluster>

# Install kubectl and ensure that you are able to list post in the
# cluster
> brew install kubectl
> kubectl get pod --all-namespaces

# Install helm on Mac
> brew install kubernetes-helm

# Install server component of helm, and eleveate your credentials.
> helm init
> kubectl create serviceaccount --namespace kube-system tiller
> kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin \
    --serviceaccount=kube-system:tiller
> kubectl patch deploy --namespace kube-system tiller-deploy \
    -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
> kubectl create clusterrolebinding <your-name>-cluster-admin-binding \
    --clusterrole=cluster-admin --user=<your-email-used-for-gke>

# You should be able to list all helm installations (none actually)
> helm list
~~~

### Install dockup agent

Once you have `kubectl` and `helm` installed and configured, you need to install
traefik and dockup agent. Please have a domain ready so Dockup can start using it.

~~~
# Install dockup agent. Dockup agent should have the ability to create
# pods, push/pull images from registry.
> helm repo add dockup https://helm-charts.getdockup.com
> helm install \
    --set agent.dockupBackend="kubernetes" \
    --set agent.dockupApiKey="<agent-api-key-from-settings>" \
    --set agent.dockupHost="getdockup.com" \
    --name=dockup-agent \
    dockup/agent
~~~

Once agent is installed, go to settings and ensure that agent status is
online. This means that your agent is successfully connected.


### Configure DNS

Install traefik so that traffic can be routed to several deployments.

~~~
# Install traefik.
> helm install \
    --set ssl.enabled=true \
    --set acme.enabled=true \
    --set acme.email="<your-email"> \
    --set acme.staging=false \
    --set rbac.enabled=true \
    --name=traefik \
    stable/traefik

# Get loadbalancer IP (external IP) for traefik, and update your DNS settings.
> kubectl get service
~~~

Update DNS settings with `A` record, `key` is `*.basedomain.com` and value
is the IP from `kubectl get service`.


Now you are ready to create a blueprint and do deployments.


[gcp_console]: http://console.cloud.google.com/
[gcp_kubernetes]: https://console.cloud.google.com/kubernetes/list
