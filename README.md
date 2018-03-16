# BioInformatics Docker Images

## Useful Links:
 * [github-site](https://github.com/c-omics) for source
 * [Docker Hub](https://hub.docker.com/u/comics) for docker images

## Dockerfiles
 1. The respective license of the applications is located at /usr/share/licenses/ within the docker image. 
 2. Most ```comics``` dockerfiles inherit the base ```comics/centos``` image, which in turn is based on the official centos:latest image.
 3. At build-time, the images will download and install 3rd party software. While we try to verify the authenticity of such software, this is not easily achieved, particularly when the 3rd party software itself pulls in dependencies.
 4. Many images will install software from source. To keep the images light-weight, where possible the dockerfile will download the software, verify it, build it, and remove the source again within a single RUN instruction. 
 5. In certain cases, writing to a bind-mounted folder will write the files to the host as root-owned. This is somewhat undesirable. To overcome this, a package called gosu is used in all our images, which will allow the files to be written to the host as any desired user. To do this you will need to set extra environment variables when running the container - further usage instructions are defined below.
 
## Bind-Mounting
It is often desirable mount a local folder inside the docker container (bind-mount). This is acheived using the ```-v``` option to the ```docker run``` command, for example:
```bash
docker run -it -v $PWD:/mnt/host comics/centos:latest bash
``` 
Here we assume a linux host, starting the latest comics/centos7 container, and bind-mounting the current working directory to the path /mnt/host within the container. The problem with this is that our default user is root, and therefore any files/folders created under the new mount point will be owned by root (both within the container and on the host). This is sometimes undersirable, instead you may want the new files/folders to be owned by a different user, likely the host user. To deal with this, our containers have an entrypoint which will optionally create a new user within the container, and then run under that user within the container. To create this user we pass two environment variables to the docker run command:
```bash
docker run -it -v $PWD:/mnt/host -e HOST_USER_ID=`id -u $USER` -e HOST_GROUP_ID=`id -g $USER`  comics/centos:latest bash
```
Here the environment variables 'HOST_USER_ID' and 'HOST_GROUP_ID' are passed to the container (respectively holding the uid and gid of the desired user), the entrypoint recognises this and deals with it accordingly. The container will now start under a user with those uid and gid specified at the command line. Now any files/folders written to the mount point will be written as the desired user.


