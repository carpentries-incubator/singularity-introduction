---
title: "Singularity: Getting started"
start: true
teaching: 30
exercises: 20
questions:
- "What is Singularity and why might I want to use it?"
objectives:
- "Understand what Singularity is and when you might want to use it."
- "Undertake your first run of a simple Singularity container."
keypoints:
- "Containers enable you to package up an application and its dependencies."
- "By using containers, you can better enforce reproducibility, portability and share-ability of your computational workflows."
- "Singularity is another container platform and it is often used in cluster/HPC/research environments."
- "Singularity has a different security model to other container platforms, one of the key reasons that it is well suited to HPC and cluster environments."
- "Singularity has its own container image format (SIF)."
- "The `singularity` command can be used to pull images from Singularity Hub and run a container from an image file."
---

The episodes in this lesson will introduce you to the [Singularity](https://sylabs.io/singularity/) container platform and demonstrate how to set up and use Singularity.

This material is split into 2 parts:

*Part I: Basic usage, working with images*

 1. **Singularity: Getting started**: This introductory episode

Working with Singularity containers:
<ol start="2">
 <li><strong>The singularity cache: </strong> Why, where and how does Singularity cache images locally?</li>
 <li><strong>Running commands within a Singularity container: </strong> How to run commands within a Singularity container.</li>
 <li><strong>Working with files and Singularity containers: </strong> Moving files into a Singularity container; accessing files on the host from within a container.</li>
 <li><strong>Using Docker images with Singularity: </strong>How to run Singularity containers from Docker images.</li>
 </ol>
 *Part II: Creating images, running parallel codes*
 <ol start="6">
   <li><strong>Preparing to build Singularity images</strong>: Getting started with the Docker Singularity container.</li>
   <li><strong>Building Singularity images</strong>: Explaining how to build and share your own Singularity images.</li>
   <li><strong>Running MPI parallel jobs using Singularity containers</strong>: Explaining how to run MPI parallel codes from within Singularity containers.</li>
</ol>

> ## Work in progress

> This lesson is new material that is under ongoing development. We will introduce Singularity and demonstrate how to work with it. As the tools and best practices continue to develop, elements of this material are likely to evolve. We welcome any comments or suggestions on how the material can be improved or extended.
{: .callout}

# Singularity - Part I

### Containers vs Virtual Machines

A container is an entity providing an isolated software environment (or filesystem) for an application and its dependencies.  

If you have already used a Virtual Machine, or VM, you're actually already familiar with some of the concepts of a container.

<!-- ![Containers vs. VMs]({{ page.root }}/fig/container_vs_vm.png) -->
<img src="{{ page.root }}/fig/container_vs_vm.png" alt="Containers vs. VMs" width="619" height="331"/>

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

A **registry** is a server application where images are stored and can be accessed by users.  It can be public (*e.g.* *Docker Hub*) or private.

To build an image we need a recipe.  A recipe file is called a **Definition File**, or **def file**, in the *Singularity* jargon and a **Dockerfile** in the *Docker* world.

### Container engines

A number of tools are available to create, deploy and run containerised applications.  Some of these will be covered throughout this tutorial:

* **Docker**: the first engine to gain popularity, still widely used in the IT industry.  Not very suitable for HPC as it requires *root* privileges to run. We'll use it mostly to build container images. See the extensive [online documentation](https://docs.docker.com/) for more information.

* **Singularity**: a simple, powerful *root*-less container engine for the HPC world. The main focus of this workshop. See the [user guide](https://sylabs.io/guides/latest/user-guide/) for extensive documentation.

* **Apptainer**: an open-source offshoot of **Singularity**. Provides all the same functionality as **Singularity** and moving forward will likely become the open-source standard. See the [user guide](https://apptainer.org/docs/user/main/) for extensive documentation.

## What is Docker

> ## Loading a module

> HPC systems often use *modules* to provide access to software on the system so you may need to use the command:
>
> ~~~
> $ module load Singularity
> ~~~
>
> {: .language-bash}
> before you can use the `singularity` command on the system.
{: .callout}

~~~
singularity --version
~~~

{: .language-bash}

~~~
singularity version 3.5.3
~~~

{: .output}

Depending on the version of Singularity installed on your system, you may see a different version. At the time of writing, `v3.5.3` is the latest release of Singularity.

## Getting an image and running a Singularity container

Let's begin by creating a `test` directory, changing into it and *pulling* a test *Hello World* image from Singularity Hub:

~~~
mkdir test
cd test
singularity pull library://sylabsed/examples/lolcow
~~~

{: .language-bash}

~~~
INFO:    Downloading shub image
 59.75 MiB / 59.75 MiB [===============================================================================================================] 100.00% 52.03 MiB/s 1s
~~~

{: .output}

What just happened?! We pulled a SIF image from Singularity Hub using the `singularity pull` command and directed it to store the image file using the name`lolcow_latest.sif`in the current directory. If you run the `ls` command, you should see that the `lolcow_latest.sif` file is now present in the current directory. This is our image and we can now run a container based on this image:

~~~
singularity run lolcow_latest.sif
~~~

{: .language-bash}

~~~
_____________________________
/ This was the most unkindest cut of all. \
|                                         |
\ -- William Shakespeare, "Julius Caesar" /
 -----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
~~~

{: .output}

Most images are also directly executable

~~~
./lolcow_latest.sif
~~~

{: .language-bash}

~~~
 _________________________________________
/ This was the most unkindest cut of all. \
|                                         |
\ -- William Shakespeare, "Julius Caesar" /
 -----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
~~~

{: .output}

> ## Whats with the errors?

> This is to do with a monitoring library used on NeSI, it can be fixed by unloading the `XALT` module.
> This can be done with the command.
>
> ~~~
> $ module unload XALT
> ~~~
>
> {: .language-bash}
{: .callout}

How did the container determine what to do when we ran it?! What did running the container actually do to result in the displayed output?

When you run a container from a Singularity image without using any additional command line arguments, the container runs the default run script that is embedded within the image. This is a shell script that can be used to run commands, tools or applications stored within the image on container startup. We can inspect the image's run script using the `singularity inspect` command:

~~~
singularity inspect -r lolcow_latest.sif
~~~

{: .language-bash}

~~~
#!/bin/sh

    fortune | cowsay | lolcat
~~~

{: .output}

This shows us the script within the `lolcow_latest.sif` image configured to run by default when we use the `singularity run` command.

That concludes this introductory Singularity episode. The next episode looks in more detail at running containers.
