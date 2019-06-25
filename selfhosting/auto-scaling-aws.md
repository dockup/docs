# AWS EKS auto scaling instructions

Unlinke GCP, AWS EKS doesn't enable autoscaling by default. You need to install
[autoscaler][k8s-autoscaler] specific to AWS EKS cluster. Thanks to helm, we
can install cluster-autoscaler chart. This doc outlines steps as suggested in
the readme of [helm chart][helm-cluster-autoscaler].


### Update IAM role

Once worker nodes are created using cloud formation, go to "Resources" tab and
search for `IAM::Role`. You should see something like this:

![IAM Role](/images/cf-resources-iam-role.png)

Visit the instance role, and add a inline policy in JSON format.

![Edit IAM Role](/images/edit-iam-role-add-inline-policy.png)

![Update IAM Role](/images/update-iam-role-apply-inline-policy.png)


Snippet to paste is:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "autoscaling:DescribeAutoScalingGroups",
                "autoscaling:DescribeAutoScalingInstances",
                "autoscaling:DescribeLaunchConfigurations",
                "autoscaling:DescribeTags",
                "autoscaling:SetDesiredCapacity",
                "autoscaling:TerminateInstanceInAutoScalingGroup"
            ],
            "Resource": "*"
        }
    ]
}
```


Review and apply the policy.


### Add tags to node group for auto discovery

Again go to "Resources" tab of worker nodes cloud formation stack and search
for "NodeGroup". You should see something like this:

![Node group](/images/cf-resources-node-group.png)


Apply the following tags

```
k8s.io/cluster-autoscaler/enabled = true
kubernetes.io/cluster/<ClusterName> = owned
```

![Autoscaler tags](/images/tag-autoscaler-node-group.png)


### Install autoscaler helm chart

Run cluster autoscaler helm chart and you should have autoscaling in place.

~~~
> helm repo add dockup https://helm-charts.getdockup.com
> helm install \
    --set autoDiscovery.clusterName="<cluser-name-that-you-have-created>" \
    --set awsRegion="<region-where-cluster-exists>" \
    --set sslCertPath=/etc/kubernetes/pki/ca.crt \
    --set rbac.create=true \
    --name=dockup-autoscaler \
    stable/cluster-autoscaler
~~~


As you deploy, you can see nodes coming up and going down using watch
command `kubectl get nodes --watch`




[k8s-autoscaler]: https://github.com/kubernetes/autoscaler
[helm-cluster-autoscaler]: https://github.com/helm/charts/tree/master/stable/cluster-autoscaler
