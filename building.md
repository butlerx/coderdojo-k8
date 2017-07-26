# Building

We have 3 different type of docker images, `latest`, `dev`, and tags based of git commit that the image was built off.
All the images are on [Docker Hub](https://hub.docker.com/u/coderdojo/)

## Latest

In each repo that gets deployed as a service there will be a `Dockerfile`. When a new commit is merged to master dockerhub will pull the updates and build a new latest container.
This is set up as an automated build in dockerhub. These builds can take up too 20 minutes to finish due to being on the free tier which makes them poor for use in production, but good for using in testing.

## Dev

As well as `Dockerfile` you should also find a `dev.Dockerfile` in each repo. These are built in the same way as `latest` and at the same time. The difference between these is that dev does not have any code in it but volumes to mount the code in, aswell as a startup script to install dep and link up cp-translations internally.
This should only be used with [cp-localdev](https://github.com/CoderDojo/cp-local-development).

## Production

Production images are built using the same `Dockerfile` as latest. They are built on circleci. When new commit is made to master, circle will test master, if all test pass circle will build a docker image of the code it tested and will tag is with the commit sha1 hash. Once it succesfully builds it will push the image to dockerhub. Then tell kubernetes the image has changed and to use that hash, this is covered more in [Deployment](./deployment.md).
