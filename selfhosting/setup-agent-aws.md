# AWS agent setup instructions

### Create a kubernetes cluster

First sign up for AWS account if you don't have one. You can do it by heading
over to [console page][aws_console].

After you setup your AWS account, head over to [kubernetes][aws_kubernetes]
cluster page, and create a cluster called 'Dockup'. It's okay to go with all
default values. Please make sure that you create a role which is attached to
EKS.


![Kubernetes create page](/images/aws_kubernetes_create.png)


Cluster creation will take sometime. Please wait till it is complete.


### Verify connection to cluster

This section is consolidated version of official aws documentation. You can
refer this [page][aws_eks] for more details.


~~~sh
# Install AWS cli using pip:
> pip install awscli

# Install AWS iam authenticator. Using brew:
> brew install aws-iam-authenticator

# Install kubectl using brew:
> brew install kubectl

# Configure aws cli with client id and secret id. They can be generated via
# console
> aws configure

# Retrieve kubernetes configuration.
> aws eks --region us-east-1 update-kubeconfig --name Dockup
# Added new context arn:aws:eks:us-east-1:...:cluster/Dockup to ~/.kube/config

# Set the context for kubernetes
> kubectl config use arn:aws:eks:us-east-1:...:cluster/Dockup

# Ensure that you are able to list pods without any errors.
> kubectl get pod --all-namespaces
~~~

Now EKS is active without any nodes. You need to add worker nodes by following
instructions as per [documentation][aws_eks_workers]. Please make sure that
security group, VPC and subnets match EKS settings. Note down `NodeInstanceRole`
when stack creation is successful.

~~~
# Download, configure and update cloud formation template to connect to nodes
> curl -O https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2019-02-11/aws-auth-cm.yaml

# Replace the <ARN of instance role (not instance profile)> snippet with
# the `NodeInstanceRole` value
> kubectl apply -f aws-auth-cm.yaml

# Check node statuses until any node reaches ready
> kubectl get nodes --watch

# Check pod status, and ensure they are running
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
If you need auto-renewed Letsencrypt SSL, contact support@getdockup.com mentioning your DNS provider.
The following command installs traefik without SSL.

~~~
> helm install \
    --set rbac.enabled=true \
    --name=traefik \
    stable/traefik

# Get "LoadBalancer Ingress" for traefik from the output of the following command.
> kubectl describe service traefik
~~~

Add a `CNAME` record in your DNS provider to point to the `EXTERNAL_IP` you got
from the previous command.

For example, if you own "acme.com" and you want your deployment URLs to be
"<random_urls>.dockup.acme.com", then your base domain is dockup.acme.com and your
DNS A record will have key: `*.dockup` and value: `<value of EXTERNAL-IP>`

Now you can go back to https://getdockup.com and update the base domain.
In the above example, the value will be "dockup.acme.com". Once this is done,
you are ready to start deploying!



[aws_console]: http://console.aws.amazon.com/
[aws_kubernetes]: https://console.aws.amazon.com/eks/home?region=us-east-1
[aws_eks]: https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html
[aws_eks_workers]: https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html#eks-launch-workers
