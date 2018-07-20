### PyTorch Docker image

[![Docker Build Status](https://img.shields.io/docker/build/splashblot/docker-pytorch.svg)](https://hub.docker.com/r/splashblot/docker-pytorch/)

... forked for our purposes from https://hub.docker.com/r/anibali/pytorch/ (see below for original `README.md` content)

The purpose of this image is to expedite our experimentation with PyTorch based convnets.  They two projects we've started with are:

  * [Robosat](https://github.com/mapbox/robosat/) by Mapbox
  * [TernausNetV2](https://github.com/ternaus/TernausNetV2) by [Vladimir Iglovikov](https://github.com/ternaus)

#### Pre-requisites

Before you proceed, get these working:

  * Docker (duh!)
  * [NVIDIA Docker 2.0](https://github.com/nvidia/nvidia-docker/wiki/Installation-(version-2.0)) 

Yes, this means you don't have to worry about installing CUDA!  GPU drivers should be all you need.

To test that you're ready to roll try:

```
docker run splashblot/docker-pytorch:cuda91 nvidia-smi
```

... and you should see something along the lines of:

```
Fri Jul 02 17:02:24 2018
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 396.24.02              Driver Version: 396.24.02                 |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Tesla V100-PCIE...  Off  | 00000000:81:00.0 Off |                    0 |
| N/A   42C    P0    39W / 250W |   1609MiB / 16160MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
+-----------------------------------------------------------------------------+
```

#### Robosat testing

To start playing around, you can just kick off a container like so:

```
docker run -ti -p 8899:8899 -p 5000:5000 -v tiles-swarm_data-volume-datasets:/datasets/ splashblot/docker-pytorch:cuda91 bash
```

... port 8899 will give you access to a Juyter installation, and port 5000 is left open for the times you might want to use `rs serve`.


You will need a MAPBOX token, use this for now:

```
export MAPBOX_ACCESS_TOKEN=pk.eyJ1IjoiamptYXRhIiwiYSI6ImNqanVneWd3eDR3b2szbG14ZTR1YjExNDUifQ.g5xdbem-iixR1svLi9QpRg
```

And start training away ...

* * *
Ubuntu [14.04|16.04] + PyTorch + CUDA [7.5|8.0|9.1|none]

#### Requirements

In order to use this image you must have Docker Engine installed. Instructions
for setting up Docker Engine are
[available on the Docker website](https://docs.docker.com/engine/installation/).

#### Building

This image can be built on top of multiple different base images derived from
Ubuntu. Which base you choose depends on whether you have an NVIDIA
graphics card which supports CUDA and you want to use GPU acceleration or not.

If you are running Ubuntu, you can install proprietary NVIDIA drivers
[from the PPA](https://launchpad.net/~graphics-drivers/+archive/ubuntu/ppa)
and CUDA [from the NVIDIA website](https://developer.nvidia.com/cuda-downloads).
These are only required if you want to use GPU acceleration.

If you are running Windows or OSX then you will find it difficult/impossible to
use GPU acceleration due to the fact that containers run in a virtual machine.
Use the image without CUDA on those platforms.

##### With CUDA

Firstly ensure that you have a supported NVIDIA graphics card with the
appropriate drivers and CUDA libraries installed.

Build the image using the following command:

```sh
# If you have CUDA 9.1:
docker build -t pytorch ./cuda-9.1
# If you have CUDA 8.0:
docker build -t pytorch ./cuda-8.0
# If you have CUDA 7.5:
docker build -t pytorch ./cuda-7.5
```

You will also need to install `nvidia-docker`, which we will use to start the
container with GPU access. This can be found at
[NVIDIA/nvidia-docker](https://github.com/NVIDIA/nvidia-docker).

##### Without CUDA

Build the image using the following command:

```sh
docker build -t torch ./no-cuda
```

#### Usage

##### Running PyTorch scripts

It is also possible to run PyTorch programs inside a container using the
`python3` command. For example, if you are within a directory containing
some PyTorch project with entrypoint `main.py`, you could run it with
the following command:

```sh
docker run --rm -it --init \
  --runtime=nvidia \
  --ipc=host \
  --user="$(id -u):$(id -g)" \
  --volume=$PWD:/app \
  -e NVIDIA_VISIBLE_DEVICES=0 \
  pytorch python3 main.py
```

Here's a description of the Docker command-line options shown above:

* `--runtime=nvidia`: Required if using CUDA, optional otherwise. Passes the
  graphics card from the host to the container.
* `--ipc-host`: Required if using multiprocessing, as explained at
  https://github.com/pytorch/pytorch#docker-image.
* `--user="$(id -u):$(id -g)"`: Sets the user inside the container to match your
  user and group ID. Optional, but is useful for writing files with correct
  ownership.
* `--volume=$PWD:/app`: Mounts the current working directory into the container.
  The default working directory inside the container is `/app`. Optional.
* `-e NVIDIA_VISIBLE_DEVICES=0`: Sets an environment variable to restrict which
  graphics cards are seen by programs running inside the container. Set to `all`
  to enable all cards. Optional, defaults to all.

You may wish to consider using [Docker Compose](https://docs.docker.com/compose/)
to make running containers with many options easier. At the time of writing,
only version 2.3 of Docker Compose configuration files supports the `runtime`
option.

##### Running graphical applications

If you are running on a Linux host, you can get code running inside the Docker
container to display graphics using the host X server (this allows you to use
OpenCV's imshow, for example). Here we describe a quick-and-dirty (but INSECURE)
way of doing this. For a more comprehensive guide on GUIs and Docker check out
http://wiki.ros.org/docker/Tutorials/GUI.

On the host run:

```sh
sudo xhost +local:root
```

You can revoke these access permissions later with `sudo xhost -local:root`.
Now when you run a container make sure you add the options `-e "DISPLAY"` and
`--volume="/tmp/.X11-unix:/tmp/.X11-unix:rw"`. This will provide the container
with your X11 socket for communication and your display ID. Here's an
example:

```sh
docker run --rm -it --init \
  --runtime=nvidia \
  -e "DISPLAY" --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" \
  pytorch python3 -c "import tkinter; tkinter.Tk().mainloop()"
```
