# EKS
``` bash
eksctl get cluster
aws eks update-kubeconfig --region region-code --name cluster-name
```

?> **Tips** none at the moment

# install
``` yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: storecluster
  region: eu-west-2

nodeGroups:
  - name: nodegroup-sc
    instanceType: m5.large
    desiredCapacity: 3
    iam:
      withAddonPolicies:
        ebs: true
        fsx: true
        efs: true

vpc:
  subnets:
    # must provide 'private' and/or 'public' subnets by availibility zone as shown
    public:
      eu-west-2b:
```

