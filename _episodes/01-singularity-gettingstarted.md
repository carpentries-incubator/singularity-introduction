---
title: "Getting Started with Containers"
start: true
teaching: 20
exercises: 0
questions:
- "What is a container and why might I want to use it?"
objectives:
- "Understand what a container is and when you might want to use it."
keypoints:
- "Containers enable you to package up an application and its dependencies."
- "By using containers, you can better enforce reproducibility, portability and share-ability of your computational workflows."
- "Apptainer (and Singularity) are container platforms and are often used in cluster/HPC/research environments."
- "Apptainer has a different security model to other container platforms, one of the key reasons that it is well suited to HPC and cluster environments."
- "Apptainer has its own container image format based off the original Singularity Image Format (SIF)."
- "The `apptainer` command can be used to pull images from Docker Hub or other locations such as a website and run a container from an image file."
---

The episodes in this lesson will introduce you to the [{{ site.software.name }}]({{ site.software.url }}) container platform and demonstrate how to set up and use {{ site.software.name }}.

### What are Containers

A container is an entity providing an isolated software environment (or filesystem) for an application and its dependencies.  

If you have already used a Virtual Machine, or VM, you're actually already familiar with some of the concepts of a container.

<div style="text-align: center;">
<img src="{{ page.root }}/fig/container_vs_vm.png" alt="Containers vs. VMs" width="619" height="331"/>
<div><em>Credit: Pawsey Centre, <a href='https://pawseysc.github.io/sc19-containers/'>Containers in HPC</a></em></div>
</div>
<br>

The key difference here is that VMs virtualise **hardware** while containers virtualise **operating systems**.  There are other differences (and benefits), in particular containers are:

* lighter weight to run (less CPU and memory usage, faster start-up times)
* smaller in size (thus easier to transfer and share)
* modular (possible to combine multiple containers that work together)

Since containers do not virtualise the hardware, containers must be built using the same architecture
as the machine they are going to be deployed on.
Containers built for one architecture cannot run on the other.

### Containers and your workflow

There are a number of reasons for using containers in your daily work:

* Data reproducibility/provenance
* Cross-system portability
* Simplified collaboration
* Simplified software dependencies and management
* Consistent testing environment

## Terminology

We'll start with a brief note on the terminology used in this section of the course. We refer to both ***images*** and ***containers***. What is the distinction between these two terms?

***Images*** are bundles of files including an operating system, software and potentially data and other application-related files. They may sometimes be referred to as a *disk image* or *container image* and they may be stored in different ways, perhaps as a single file, or as a group of files. Either way, we refer to this file, or collection of files, as an image.

A ***container*** is a virtual environment that is based on an image. That is, the files, applications, tools, etc that are available within a running container are determined by the image that the container is started from. It may be possible to start multiple container instances from an image. You could, perhaps, consider an image to be a form of template from which running container instances can be started.

A ***registry*** is a server application where images are stored and can be accessed by users.  It can be public (*e.g.* *Docker Hub*) or private.

To build an image we need a recipe.  A recipe file is called a **Definition File**, or **def file**, in the *Apptainer* jargon and a **Dockerfile** in the *Docker* world.

### Container engines

A number of tools are available to create, deploy and run containerised applications.  Some of these will be covered throughout this tutorial:

* **Docker**: The first engine to gain popularity, still widely used in the IT industry.  Not very suitable for HPC as it requires *root* privileges to run. We'll use it mostly to build container images. See the extensive [online documentation](https://docs.docker.com/) for more information.

* **Singularity**: a simple, powerful *root*-less container engine for the HPC world. The main focus of this workshop. See the [user guide](https://sylabs.io/guides/latest/user-guide/) for extensive documentation.

* **Apptainer**: an open-source offshoot of **Singularity**. Provides all the same functionality as **Singularity** and moving forward will likely become the open-source standard. See the [user guide](https://apptainer.org/docs/user/main/) for extensive documentation.

That concludes this container overview. The next episode looks in more detail at setting up your environment for running containers on the NeSI cluster.
