---
layout: post
title: "Setting up a Virtual Machine to run ML applications"
date: 2021-03-20
---

I have often found myself needing to provision a new VM with a GPU to run Machine Learning workloads. Modern cloud services and containerization have made it easy to go from provisioning to running experiments in a matter of minutes. Here I have consolidated how to set up a VM to run ML applications with minimal configuration by using TensorFlow's Docker containers.

Before we begin there are a few things you need to take care of when you SSH into your machine:

```bash
$ ssh -L <host port>:localhost:<vm port> <user>@<address>
```

The -L flag forwards requests from localhost:\<vm port\> on your VM to the \<host port\> on your host. This will be useful later when we are trying to access a Jupyter notebook or Tensorboard in a browser on our host machine. It is best to use something simple such as 9999 for both the values.

Run the usual update commands on the VM and get the NVIDIA GPU drivers:

```bash
$ sudo apt update
$ sudo apt dist-upgrade

# Get pip3
$ sudo apt install python3-pip

# Get latest Nvidia drivers
$ sudo apt install ubuntu-drivers-common
$ ubuntu-drivers list

# Here install nvidia-driver-<ver>, perferably the latest
$ sudo apt install nvidia-driver-<ver>
```

At this point, we will need to set up Docker on our VM:

```bash
# Install Docker
$ sudo apt install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

# Add Docker's official GPG key
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# Add the Docker repository
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

# Update and finally install Docker
$ sudo apt update
$ sudo apt install docker-ce docker-ce-cli containerd.io
```

For GPU support on Linux, we need to install NVIDIA Docker support libraries. Check out NVIDIA's [documentation](https://nvidia.github.io/nvidia-docker/) for more information.

```bash
# Add the package repositories
$ curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
$ distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
$ curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker. \ list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

# Install the Nvidia container toolkit
$ sudo apt update && sudo apt install -y nvidia-container-toolkit
$ sudo systemctl restart docker
```
Now that we are done installing all dependencies, let's get the docker container and start doing some parameter optimization. We will get the latest GPU-supported version. Check Tensorflow's [documentation](https://www.tensorflow.org/install/docker) to see all available versions. 

```bash
$ sudo docker pull tensorflow/tensorflow:latest-gpu  # latest release w/ GPU support
```

With the setup done, here's how to start the container:

```bash
$ sudo docker run -it --gpus all -p <vm port>:<container port> -v <vm dir path>:/data tensorflow/tensorflow:latest-gpu bash
```

This command forwards requests from \<container port\> to your \<vm port\>. Again, using the same value (such as 9999) for both is the easiest. This also mounts \<vm dir path\> inside the `/data` directory in your container, making required project files accessible.

Lastly, the bash keyword opens a root bash terminal inside the container.

You can now get started with your code. For example, to run a Jupyter notebook inside your VM, navigate to the folder that you want to start a notebook in and run:

```bash
$ jupyter notebook --ip 0.0.0.0 --port <container port> --allow-root
```

Navigate to a browser on your host computer and go to localhost:\<host port\>. Copy the token that you see in the container running Jupyter and you will be able to access your Jupyter notebook instance.

### Sources:
* <https://www.tensorflow.org/install/docker>
* <https://docs.docker.com/engine/install/ubuntu/>
* <https://github.com/NVIDIA/nvidia-docker>