# GCP instructions

### Create a kubernetes cluster

First sign up for GCP account if you don't have one. You can do it by heading
over to [console page][gcp_console]. Once signed up, create a project called
"dockup" and complete setting up billing.

**NOTE: You will get free $300 credit when you sign up for the first time**

After you setup your GCP account, head over to [kubernetes][gcp_kubernetes]
cluster page, enable kubernetes api, and create a cluster. It's okay to go
with all default values. Please make a note of the name, and zone as we will
be using them later. Create page looks like this:


![Kubernetes create page](/images/gcp_kubernetes_create.png)


Cluster creation will take sometime. Please wait till it is
complete.


### Verify connection to cluster

~~~sh
# Install google cloud SDK. Using homebrew:
> brew cask install google-cloud-sdk

# Authorize yourself so that you can access projects
> gcloud auth application-default login

# See if you can get a list of projects. You should see the project that
# you have created here.
> gcloud projects list

# Set the current project.
> gcloud config set project dockup

# Get credentials for newly created kubernetes cluster.
> gcloud container clusters get-credentials <name-of-cluster> \
  --region=<region-of-cluster>

# Install kubectl and ensure that you are able to list pods
# without any errors.
> brew install kubectl
> kubectl get pod --all-namespaces

# Install helm. Using homebrew:
> brew install kubernetes-helm

# Install helm on kubernetes cluster:
> helm init
> kubectl create serviceaccount --namespace kube-system tiller
> kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin \
    --serviceaccount=kube-system:tiller
> kubectl patch deploy --namespace kube-system tiller-deploy \
    -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
> kubectl create clusterrolebinding <your-name>-cluster-admin-binding \
    --clusterrole=cluster-admin --user=<your-email-used-for-gke>

# You should be able to list helm installations without any error:
> helm list
~~~

### Install dockup agent

Once you have `kubectl` and `helm` installed and configured, you need to install
dockup agent. Fetch your Agent API key by going to https://getdockup.com.

~~~
> helm repo add dockup https://helm-charts.getdockup.com
> helm install \
    --set agent.dockupApiKey="<agent-api-key-from-settings>" \
    --name=dockup-agent \
    dockup/agent
~~~

Once agent is installed, refresh https://getdockup.com and ensure that agent status is
online.


### Configure DNS

Next, you will install traefik which will be the loadbalancer for your kubernetes cluster.
For auto-renewed letsencrypt SSL, follow these instructions before setting up traefik: [How to set up SSL][ssl_setup].
The following command installs traefik and enables SSL (modify it as per the SSL instructions for your DNS provider):

~~~
> helm install \
    --set ssl.enabled=true \
    --set acme.enabled=true \
    --set acme.email="<your-email"> \
    --set acme.staging=false \
    --set rbac.enabled=true \
    --name=traefik \
    stable/traefik

# Get "EXTERNAL-IP" for traefik from the output of the following command.
> kubectl get service
~~~

Add an `A` record in your DNS provider to point to the `EXTERNAL_IP` you got
from the previous command.

For example, if you own "acme.com" and you want your deployments to be running on
"<random_url>.dockup.acme.com", then your base domain is dockup.acme.com and your
DNS A record will have `key`: `*.dockup` and value: `<value of EXTERNAL-IP>`

Now you can go back to https://getdockup.com and update the base domain.
In the above example, the value will be "dockup.acme.com". Once this is done,
you are ready to start deploying!


[gcp_console]: http://console.cloud.google.com/
[gcp_kubernetes]: https://console.cloud.google.com/kubernetes/list
[ssl_setup]: https://docs.traefik.io/v1.6/configuration/acme/#wildcard-certificates
