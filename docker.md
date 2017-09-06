# Cheatsheet

From [Docker Tutorial](https://docs.docker.com/get-started/).

Create image using this directory's Dockerfile:
`docker build -t friendlyname .`

Run "friendlyname" mapping port 4000 to 80:
`docker run -p 4000:80 friendlyname`

Same thing, but in detached mode:
`docker run -d -p 4000:80 friendlyname

List all running containers
`docker container ls`

List all containers, even those not running:
`docker container ls -a`

Gracefully stop the specified container:
`docker container stop <hash>`

Force shutdown of the specified container:
`docker container kill <hash>`

Remove specified container from this machine:
`docker container rm <hash>`

Remove all containers:
`docker container rm $(`docker container ls -a -q)`

List all images on this machine:
`docker image ls -a`

Remove specified image from this machine:
`docker image rm <image id>`

Remove all images from this machine:
`docker image rm $(`docker image ls -a -q)`

Log in this CLI session using your `Docker credentials:
`docker login`

Tag <image> for upload to registry:
`docker tag <image> username/repository:tag`

Upload tagged image to registry:
`docker push username/repository:tag`

Run image from a registry:
`docker run username/repository:tag`
