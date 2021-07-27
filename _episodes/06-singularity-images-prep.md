---
title: "Preparing to build Singularity images"
teaching: 15
exercises: 20
questions:
- "What environment do I need to build a Singularity image and how do I set it up?"
objectives:
- "Understand how to the Docker Singularity image provides an environment for building Singularity images."
- "Understand different ways to run containers based on the Docker Singularity image."
keypoints:
- "A Docker image is provided to run Singularity - this avoids the need to have a local Singularity installation on your system."
- "The Docker Singularity image can be used to build containers on Linux, macOS and Windows."
- "You can also run Singularity containers within the Docker Singularity image."
---

# Singularity - Part II

## Brief recap

In the five episodes covering Part I of this Singularity material we've seen how Singularity can be used on a computing platform where you don't have any administrative privileges. The software was pre-installed and it was possible to work with existing images such as Singularity image files already stored on the platform or images obtained from a remote image repository such as Singularity Hub or Docker Hub.

It is clear that between Singularity Hub and Docker Hub there is a huge array of images available, pre-configured with a wide range of software applications, tools and services. But what if you want to create your own images or customise existing images?

In this first of three episodes in Part II of the Singularity material, we'll look at preparing to build Singularity images.

## Preparing to use Singularity for building images

So far you've been able to work with Singularity from your own user account as a non-privileged user. This part of the Singularity material requires that you use Singularity in an environment where you have administrative (root) access. While it is possible to build Singularity containers without root access, it is highly recommended that you do this as the _root_ user, as highlighted in [this section](https://sylabs.io/guides/3.5/user-guide/build_a_container.html#creating-writable-sandbox-directories) of the Singularity documentation. Bear in mind that the system that you use to build containers doesn't have to be the system where you intend to run the containers. If, for example, you are intending to build a container that you can subsequently run on a Linux-based cluster, you could build the container on your own Linux-based desktop or laptop computer. You could then transfer the built image directly to the target platform or upload it to an image repository and pull it onto the target platform from this repository.

There are **three** different options for accessing a suitable environment to undertake the material in this part of the course:

 1. Run Singularity from within a Docker container - this will enable you to have the required privileges to build images
 1. Install Singularity locally on a system where you have administrative access
 1. Use Singularity on a system where it is already pre-installed and you have administrative (root) access

We'll focus on the first option in this part of the course - _running singularity from within a Docker container_. If you would like to install Singularity directly on your system, see the box below for some further pointers. However, please note that the installation process is an advanced task that is beyond the scope of this course so we won't be covering this.

> ## Installing Singularity on your local system (optional) \[Advanced task\]
>
> If you are running Linux and would like to install Singularity locally on your system, the source code is provided via the [The Next Generation of High Performance Computing (HPCng) community](https://github.com/hpcng)'s [Singularity repository](https://github.com/hpcng/singularity). See the releases [here](https://github.com/hpcng/singularity/releases). You will need to install various dependencies on your system and then build Singularity from source code.
>
> _If you are not familiar with building applications from source code, it is strongly recommended that you use the Docker Singularity image, as described below in the "Getting started with the Docker Singularity image" section rather than attempting to build and install Singularity yourself. The installation process is an advanced task that is beyond the scope of this session._
> 
> However, if you have Linux systems knowledge and would like to attempt a local install of Singularity, you can find details in the [INSTALL.md](https://github.com/hpcng/singularity/blob/master/INSTALL.md) file within the Singularity repository that explains how to install the prerequisites and build and install the software. Singularity is written in the [Go](https://golang.org/) programming language and Go is the main dependency that you'll need to install on your system. The process of installing Go and any other requirements is detailed in the INSTALL.md file.
> 
{: .callout}

> ## Note
> If you do not have access to a system with Docker installed, or a Linux system where you can build and install Singularity but you have administrative privileges on another system, you could look at installing a virtualisation tool such as [VirtualBox](https://www.virtualbox.org/) on which you could run a Linux Virtual Machine (VM) image. Within the Linux VM image, you will be able to install Singularity. Again this is beyond the scope of the course.
>
> If you are not able to access/run Singularity yourself on a system where you have administrative privileges, you can still follow through this material as it is being taught (or read through it in your own time if you're not participating in a taught version of the course) since it will be helpful to have an understanding of how Singularity images can be built.
> 
> You could also attempt to follow this section of the lesson without using root and instead using the `singularity` command's [`--fakeroot`](https://sylabs.io/guides/3.5/user-guide/fakeroot.html) option. However, you may encounter issues with permissions when trying to build images and run your containers and this is why running the commands as root is strongly recommended and is the approach described in this lesson.
{: .callout}

## Getting started with the Docker Singularity image

The [Singularity Docker image](https://quay.io/repository/singularity/singularity) is available from [Quay.io](https://quay.io/).

> ## Familiarise yourself with the Docker Singularity image
> - Using your previously acquired Docker knowledge, get the Singularity image for `v3.5.3` and ensure that you can run a Docker container using this image. For this exercise, we recommend using the image with the `v3.5.3-slim` tag since it's a much smaller image.
> 
> - Create a directory (e.g. `$HOME/singularity_data`) on your host machine that you can use for storage of _definition files_ (we'll introduce these shortly) and generated image files. 
> 
>   This directory should be bind mounted into the Docker container at the location `/home/singularity` every time you run it - this will give you a location in which to store built images so that they are available on the host system once the container exits. (take a look at the `-v` switch to the `docker run` command)
> 
> _Hint: To be able to build an image using the Docker Singularity container, you'll need to add the `--privileged` switch to your docker command line._
> 
> _Hint: If you want to run a shell within the Docker Singularity container, you'll need to override the entrypoint to tell the container to run `/bin/bash` - take a look at Docker's `--entrypoint` switch._
> 
> 
> Questions / Exercises:
> 
> 1. Can you run a container from the Docker Singularity image? What is happening when you run the container?
> 1. Can you run an interactive `/bin/sh` shell in the Docker Singularity container?
> 1. Can you run an interactive Singularity shell in a Singularity container, within the Docker Singularity container?!
> 
> > ## Running a container from the image
> > Answers:
> > 1. _Can you run a container from the Docker Singularity image? What is happening when you run the container?_
> >   
> >     The name/tag of the Docker Singularity image we'll be using is: `quay.io/singularity/singularity:v3.5.3-slim`
> >   
> >     Having a bound directory from the host system accessible within your running Docker Singularity container will give you somewhere to place created Singularity images so that they are accessible on the host system after the container exits. Begin by changing into the directory that you created above for storing your definiton files and built images (e.g. `$HOME/singularity_data`). 
> >   
> >     Running a Docker container from the image and binding the current directory to `/home/singularity` within the container can be achieved as follows:
> >   
> >     ```
> >     docker run --privileged --rm -v ${PWD}:/home/singularity quay.io/singularity/singularity:v3.5.3-slim
> >     ```
> >     Note that the image is configured to run the `singularity` command by default. So, when you run a container from it with no arguments, you see the singularity help output as if you had Singularity installed locally and had typed `singularity` on the command line.
> >     
> >     To run a Singularity command, such as `singularity cache list`, within the docker container directly from the host system's terminal you'd enter:
> >     
> >     ```
> >     docker run --privileged --rm -v ${PWD}:/home/singularity quay.io/singularity/singularity:v3.5.3-slim cache list
> >     ```
> >     
> >     The following diagram shows how the Docker Singularity image is being used to run a container on your host system and how a Singularity container can, in turn, be started within the Docker container:
> >     
> >     ![](/fig/SingularityInDocker.png)
> >     
> > 1. _Can you run an interactive shell in the Docker Singularity container?_
> >    
> >    To start a shell within the Singularity Docker container where you can then run the `singularity` command directly:
> >    ```
> >    docker run -it --entrypoint=/bin/sh --privileged --rm -v ${PWD}:/home/singularity quay.io/singularity/singularity:v3.5.3-slim
> >    ```
> >    Here we use the `--entrypoint` switch to the `docker` command to override the default behaviour when starting the container and instead of running the `singularity` command directly, we run a 'sh' shell. We also add the `-it` switch to provide an interactive terminal connection.
> >     
> >         
> > 1. _Can you run an interactive Singularity shell in a Singularity container, within the Docker Singularity container?!_
> >     
> >    As shown in the diagram above, you can do this. It is necessary to run `singularity shell <image file name>` within the Docker Singularity container. You would use a command similar to the following (assuming that `my_test_image.sif` is in the current directory where you run this command):
> >     
> >    ```
> >    docker run --rm -it --privileged -v ${PWD}:/home/singularity quay.io/singularity/singularity:v3.5.3-slim shell --contain /home/singularity/my_test_image.sif
> >    ```
> >    
> >    You may notice there's a flag being passed to singularity shell (`--contain` - `-c` is the short form and also works). What is this doing? When running a singularity container, you may remember that we highlighted that some key files/directories from the host system are mapped into containers by default when you start them. The configuration in the Docker Singularity container attempts to mount the file `/etc/localtime` into the Singularity container but there is not a timezone configuration present in the Docker Singularity container and this file doesn't exist resulting in an error. `--contain` prevents the default mounting of some key files/directories into the container and prevents this error from occurring. Later in this material, there's an example of how to rectify the issue by creating a timezone configuration in the Docker Singularity container so that the `--contain` switch is no longer needed.
> > 
> > _Summary / Comments:_
> > 
> > You may choose to:
> >   - open a shell within the Docker image so you can work at a command prompt and run the `singularity` command directly
> >   - use the `docker run` command to run a new container instance every time you want to run the `singularity` command (the Docker Singularity image is configured with the `singularity` command as its entrypoint).
> > 
> > Either option is fine for this section of the material.
> > 
> > To make things easier to read in the remainder of the material, command examples will use the `singularity` command directly, e.g. `singularity cache list`. If you're running a shell in the Docker Singularity container, you can enter the commands as they appear. If you're using the container's default run behaviour and running a container instance for each run of the command, you'll need to replace `singularity` with `docker run --privileged -v ${PWD}:/home/singularity quay.io/singularity/singularity:v3.5.3-slim` or similar.
> > 
> > This can be a little cumbersome to work with. However, if you're using Linux or macOS on your host system, you can add a _command alias_ to alias the command `singularity` on your host system to run the Docker Singularity container, e.g. (for bash shells - syntax for other shells varies):
> > 
> > ```
> > alias singularity='docker run --privileged -v ${PWD}:/home/singularity quay.io/singularity/singularity:v3.5.3-slim'
> > ```
> > 
> > This means you'll only have to type `singularity` at the command line as shown in the examples throughout this section of the material
> >
> >     
> {: .solution}
{: .challenge}
