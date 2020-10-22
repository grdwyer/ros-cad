# ros-cad

Docker files for building ros along with opencascade and addtional packages.

## Running
My approach (2.3 from the [ROS guide](http://wiki.ros.org/docker/Tutorials/GUI))
```
docker run -it \
    --user=$(id -u $USER):$(id -g $USER) \
    --group-add dialout --group-add sudo \
    --env="DISPLAY" \
    --workdir="/dev_ws/src" \
    --volume="/home/$USER:/home/$USER" \
    --volume="/etc/group:/etc/group:ro" \
    --volume="/etc/passwd:/etc/passwd:ro" \
    --volume="/etc/shadow:/etc/shadow:ro" \
    --volume="/etc/sudoers.d:/etc/sudoers.d:ro" \
    --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" \
    --privileged \
    ros-cad:latest
```
The container needs to be privileged to run as it needs access to the UART, the user added also needs to be in dialout and sudo.
This should be logging your host user in the container, mounting your home directory within the image and other things like x server info and sudo access.  
The other perk of this is your ssh keys are hopefully in `~/.ssh` so you can then push your changes. 

if you are going to use this container for a while then give it name with: `--name ros-cad`

Lastly the repo has been added in the docker process and is owned by root so the user id you've added won't be able to use it.
Change ownership to the user with 
`sudo chown -R $UID /dev_ws/`

## Building

### AMD64 (computer with Intel or AMD processors)

There is a docker file that contains the packages and all the dependencies. 
Where the general workflow to build the image:  

- Clone the repo
- Build the docker image

```
git clone git@github.com:grdwyer/ros-cad.git
cd ros-cad
docker build --pull --rm -f ./.docker/amd_64/Dockerfile  -t gdwyer/ros-cad:latest-amd64 .
```
If you are changing the Dockerfile remove the `--rm` tag to keep your intermediate builds. 

### ARM64

Same as above with a tiny change to the build command  
```
git clone git@github.com:grdwyer/ros-cad.git
cd ros-cad
docker build --pull --rm -f ./.docker/arm_64/Dockerfile  -t gdwyer/ros-cad:latest-arm64 .
```

### Combined Images
Images are combined using docker manifest  
TODO: move to docker buildx from [here](docker.com/blog/multi-arch-build-and-images-the-simple-way/)

```
docker manifest create \
gdwyer/ros-cad:latest \
--amend gdwyer/ros-cad:latest-amd64 \
--amend gdwyer/ros-cad:latest-arm64

docker manifest push gdwyer/ros-cad:latest
```

#### Cross compilation

The ARM image can be built on an X86-64 machine using qemu, run this before the docker build step.  
Not sure how to get this to build automatically on docker hub.

```
sudo apt-get install qemu binfmt-support qemu-user-static
docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
```
