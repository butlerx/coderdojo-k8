# Services

## Adding new services

To add a new service you'll need to first create a deployment and service yaml config. These can
both be in the same file separated by `---` or in separate files.
To see how to write a deployment and service config you can refer to
[Run a Stateless Application Using a Deployment](https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/)
and [Run a Single-Instance Stateful Application](https://kubernetes.io/docs/tasks/run-application/run-single-instance-stateful-application/)
you can also look at how our current services are written my looking on
`s3://clusters.coderdojo.com`.

The Basic config should look something like this,
``` yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
    io.kompose.service: # Service name
  name: # Service name
spec:
  clusterIP: None
  ports: # You can have multiple port object if youre exposeing multiple ports
  - name: headless
    port: # The Port you'd want to Listen publically, keep the same as targetPort if not public facing
    targetPort: # The Port on the container you have exposed and want to except traffic on
status: # This Should be configured after initial creation of the service, general load balancer settings
  loadBalancer: {}
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  creationTimestamp: null
  name: # Service Name
spec:
  replicas: 1
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        io.kompose.service: eventbright
    spec:
      containers:
      - env: # env variables for production see deployment.env for service specific env
        # check Netork.js to see which hosts the service needs the value should be the name of the service on k8
        - name: CD_USERS
          value: users
        - name: CD_DOJOS
          value: dojos
        - name: CD_EVENTS
          value: events
        - name: POSTGRES_HOST
          value: # AWS Postgres address
        - name: POSTGRES_NAME
          value: # Database name
        - name: POSTGRES_USERNAME
          value: platform
        - name: POSTGRES_PASSWORD
          value: # says it in the name
        - name: POSTGRES_PORT
          value: "5432"
        # The Following mail info is for staging, should be changed for production
        - name: MAILTRAP_ENABLED
          value: "true"
        - name: MAIL_HOST
          value: mailtrap.io
        - name: MAIL_PORT
          value: "2525"
        - name: MAIL_USER
          value: 397746d4abc52902b
        - name: MAIL_PASS
          value: 0383c445ef22d4
        - name: NEW_RELIC_ENABLED
          value: "true"
        - name: LOGENTRIES_ENABLED
          value: "true"
        - name: HOSTNAME
          value: zen.coderdojo.com
        image: coderdojo/cp-eventbright-service:latest # Use the docker image relating to this service. When first creating set to the latest build. once using circle this will be changed to tag to refect the commit
        name: # service name
        resources: {}
      restartPolicy: Always
status: {}
```

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
