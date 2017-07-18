# Setup

_This is work in progress and open to change and fine tunning_

## What We Use

We use Kubernetes(k8) running on AWS EC2 instances. They are setup to have one master instance running
on a m3-medium, with a scaling group of nodes running on t2-small. These nodes are set to scale
based on our needs. At minimum there will always be 1 node instance but this can increase up too 2
as needed.

## What You'll Need

- [Aws-cli](https://aws.amazon.com/cli/) - for making changes to aws.
- [kops](https://github.com/kubernetes/kops) - for modifying the rules controlling the EC2 k8
  instance. Configures Route53, EC2 instances including scaling, and s3 Bucket.
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl) - used for configuring the services running on kubernetes

## Setting up Kubernetes

### Resources

- [aws-blog - Manage Kubernetes Clusters on AWS Using Kops](https://aws.amazon.com/blogs/compute/kubernetes-clusters-aws-kops/)
- [Kubernetes docs - Installing Kubernetes on AWS with kops](https://kubernetes.io/docs/getting-started-guides/kops/)
- [kops docs](https://github.com/kubernetes/kops/tree/master/docs)  *These are the docs I found the most useful with setup*

### Setup

Theres is an IAM user called `kops` on aws with the required permissions.
Make sure to export the aws ID and Secret before starting as kops cant access them from
`~/.aws/config`

```
export AWS_ACCESS_KEY_ID=<access key>
export AWS_SECRET_ACCESS_KEY=<secret key>
```

### Using Kops

Due to kops being in active development, and the exact process changing, you sure refer to the
official guides above for exact commands.

Kops is going to do a few things during setup.
Kops will define DNS entries on route53 that will be routed to the k8 cluster. This is why route 53
must be set as the authoritative source for `zen.coderdojo.com`.
A s3 bucket will need to be created as well. This Bucket will be used as the persistent storage for the k8
cluster. It is where k8 will store deployment, and service configs, as well as secrets to be used by
containers. Use the aws-cli to create the bucket. Ours is called `clusters.coderdojo.com`.

Once you have the bucket created and route53 set as the DNS for `zen.coderdojo.com` follow [this guide](https://github.com/kubernetes/kops/blob/master/docs/aws.md).
Once completed you'll have a 3 node k8 cluster.
Before going forward this would be a good time to think about the ec2 instances and scaling group.
Kops will have come with a default scaling group of one master instance and 2 nodes that can scale
up too 3 as needed. Depending on the need of the stack at the time it might be better to use a
smaller master with more nodes, or smaller nodes. To save money the master node should also be
reserved from aws and could probably not be set to scale.

Kops will also have setup the k8s to use aws Elastic load balancer(elb) for load balancing. This
will be useful latter when adding a public service facing service later.
