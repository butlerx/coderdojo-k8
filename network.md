# Networking

## CoderDojo System Architecture

```graphviz
digraph coderdojo {
    # Style
    rankdir=LR
    nodesep=1.0;
    node [color=green,fontname=Courier,shape=Mrecord,style=filled];
    edge [color=Blue, style=dashed];

    # Microservices
    events[label="cp-events-service | seneca"]
    dojos[label="cp-dojos-service | seneca"]
    users[label="cp-users-service | seneca"]
    badges[label="cp-badges-service | seneca"]
    zen[label="cp-zen-platform | hapi | api"]
    frontend2[label="cp-frontend | veu.js" color=lightblue]
    frontend1[label="cp-zen-platform/web | angular 1" color=lightblue]
    forumn[label="Forumn | nodebb" color=lightblue]
    ninja[label="Ninja forumn | nodebb" color=lightblue]
    badgekit[label="Mozilla badgekit" color=purple]
    eventdb[label="Events Database | postgresql" color=purple]
    dojodb[label="Dojos Database | postgresql" color=purple]
    userdb[label="User Database | postgresql" color=purple]
    queue[label="Message queue | redis" color=red]
    wordpress[label="cd-theme | wordpress" color=lightblue]
    wpdb[label="Wordpress Database | mysql" color=yellow]
    forumndb[label="Forumn Database | redis" color=red]

    # Connections
    zen -> { events users dojos badges } [color=red];
    badges -> { dojos events users badgekit }
    { frontend1 frontend2 } -> zen
    { ninja forumn } -> { zen forumndb }
    users ->{ dojos badges userdb }
    dojos -> { users events dojodb queue }
    events -> { dojos badges users eventdb queue }
    wordpress -> { zen wpdb }

    {rank=same; userdb dojodb eventdb badgekit wpdb forumndb};
    {rank=same; wordpress frontend1 frontend2 ninja forumn }
}
```

![network](./network.svg)

## Public Facing

cp-zen-platform is the only service that should be public facing.

When setting a service to be public facing we want to make a load balancer for it this is done by
running `kubectl edit service service-name`. You'll need to edit the service file  and add the
following configs

``` yaml
metadata:
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: http
    service.beta.kubernetes.io/aws-load-balancer-ssl-cert: arn:aws:acm:eu-west-1:116766832461:certificate/3cf7c463-72cd-4323-8702-a1e7a0b93428  # This Cert name may be different for you if the cert is changed or replaced
spec:
  ports:
  - name: https
    port: 443
    protocol: TCP
    targetPort: $PORT_OF_SERVICE
  - name: http
    port: 80
    protocol: TCP
    targetPort: $PORT_OF_SERVICE
  type: LoadBalancer
```

k8 will then create a elb for the service. This eld will expose the ports specified, and send them
to ports on the node that k8 will choose which will then be sent to the specified port on the
container. The elb will provide ssl for all connections. Youll need to have the service redirect to
https it self, on zen this is done by setting the env PROTOCOL to https.

Once the elb is created go to route 53 and create the domain you want, set it as an alias for the
elb you've just created.
