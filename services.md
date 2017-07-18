# Services

## Adding new services

To add a new service you'll need to first create a deployment and service yaml config. These can
both be in the same file separated by `---` or in separate files.
To see how to write a deployment and service config you can refer to
[Run a Stateless Application Using a Deployment](https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/)
and [Run a Single-Instance Stateful Application](https://kubernetes.io/docs/tasks/run-application/run-single-instance-stateful-application/)
you can also look at how our current services are written my looking on
`s3://clusters.coderdojo.com`.

Once you written the config run `kubctl create -f ./new-sericve.yaml`, if you used to seperate files
pass both yaml files at the same time. This will create the service on the cluster.
Run `kubectl get services` and `kubectl get deployment new-service` to see if you've any errors.

If you have any errors, run `kubectl logs new-service` to get the services logs. Then to modify the
config run `kubectl edit deployment my-service`, once the file is saved it'll be deployed k8 should
apply it instantly.

## Modifying Services

To modify an already running service you can run `kubectl edit service $SERVICE_NAME` or `kubectl edit deployment $SRVICE_NAME`
depending on what you are changing.
Service is used when modifying ports and namespace the container runs.
Deployment is used when modifying env variables, images settings, such as tag or when to pull, and
replication of the service, how many of the container should run, what is the min requirements.
