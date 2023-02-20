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

> ## Work in progress...
> This lesson is new material that is under ongoing development. We will introduce Singularity and demonstrate how to work with it. As the tools and best practices continue to develop, elements of this material are likely to evolve. We welcome any comments or suggestions on how the material can be improved or extended.
{: .callout}

# Singularity - Part I

## What is Singularity?

[Singularity](https://sylabs.io/singularity/) is a container platform that allows software engineers and researchers to easily share their work with others by packaging and deploying their software applications in a portable and reproducible manner. When you download a Singularity container image, you essentially receive a virtual computer disk that contains all of the necessary software, libraries and configuration to run one or more applications or undertake a particular task, e.g. to support a specific research project. This saves you the time and effort of installing and configuring software on your own system or setting up a new computer from scratch, as you can simply run a Singularity container from the image and have a virtual environment that is identical to the one used by the person who created the image. Container platforms like Singularity provide a convenient and consistent way to access and run software and tools. Singularity is increasingly widely used in the research community for supporting research projects as it allows users to isolate their software environments from the host operating system and can simplify tasks such as running multiple experiments simultaneously.

You may be familiar with Docker, another container platform that is now used widely. If you are, you will see that in some ways, Singularity is similar to Docker. However, in others, particularly in the system's architecture, it is fundamentally different. These differences mean that Singularity is particularly well-suited to running on distributed, High Performance Computing (HPC) infrastructure, as well as a Linux laptop or desktop! 

_Later in this material, when we come to look at building Singularity images ourselves, we will make use of Docker to provide an environment in which we can run Singularity with administrative privileges. In this context, some basic knowledge of Docker is strongly recommended. If you are covering this module independently, or as part of a course that hasn't covered Docker, you can find an introduction to Docker in the "[Reproducible Computational Environments Using Containers: Introduction to Docker](https://carpentries-incubator.github.io/docker-introduction/index.html)" lesson._

System administrators will not, generally, install Docker on shared computing platforms such as lab desktops, research clusters or HPC platforms because the design of Docker presents potential security issues for shared platforms with multiple users. Singularity, on the other hand, can be run by end-users entirely within "user space", that is, no special administrative privileges need to be assigned to a user in order for them to run and interact with containers on a platform where Singularity has been installed.

## What is the relationship between Singularity, SingularityCE and Apptainer?

Singularity is open source and was initially developed within the research community. The company [Sylabs](https://sylabs.io/) was founded in 2018 to provide commercial support for Singularity.  In [May 2021](https://sylabs.io/2021/05/singularity-community-edition/), Sylabs "forked" the codebase to create a new project called [SingularityCE]((https://sylabs.io/singularity)) (where CE means "Community Edition").  This in effect marks a common point from which two projects---SingularityCE and Singularity---developed. Sylabs continue to develop both the free, open source SingularityCE and a Pro/Enterprise edition of the software. In November 2021, the original open source Singularity project [renamed itself to Apptainer](https://apptainer.org/news/community-announcement-20211130/) and [joined the Linux Foundation](https://www.linuxfoundation.org/press/press-release/new-linux-foundation-project-accelerates-collaboration-on-container-systems-between-enterprise-and-high-performance-computing-environments).

At the time of writing, in the context of the material covered in this lesson, Apptainer and Singularity are effectively interchangeable. If you are working on a platform that now has Apptainer installed, you might find that the only change you need to make when working through this material is to use the the command `apptainer` instead of `singularity`.  This course will continue to refer to Singularity until differences between the projects warrant choosing one project or the other for the course material.

## Getting started with Singularity
Initially developed within the research community, Singularity is open source and the [repository](https://github.com/hpcng/singularity) is currently available in the "[The Next Generation of High Performance Computing](https://github.com/hpcng)" GitHub organisation. Part I of this Singularity material is intended to be undertaken on a remote platform where Singularity has been pre-installed. 

_If you're attending a taught version of this course, you will be provided with access details for a remote platform made available to you for use for Part I of the Singularity material. This platform will have the Singularity software pre-installed._

> ## Installing Singularity on your own laptop/desktop
> If you have a Linux system on which you have administrator access and you would like to install Singularity on this system, some information is provided at the start of [Part II of the Singularity material]({{ page.root }}/06-singularity-images-prep/index.html#installing-singularity-on-your-local-system-optional-advanced-task).
{: .callout}

Sign in to the remote platform, with Singularity installed, that you've been provided with access to. Check that the `singularity` command is available in your terminal:

> ## Loading a module
> HPC systems often use *modules* to provide access to software on the system so you may need to use the command:
> ~~~
> $ module load singularity
> ~~~
> {: .language-bash}
> before you can use the `singularity` command on the system.
{: .callout}

~~~
$ singularity --version
~~~
{: .language-bash}

~~~
singularity version 3.5.3
~~~
{: .output}

Depending on the version of Singularity installed on your system, you may see a different version. At the time of writing, `v3.5.3` is the latest release of Singularity.

## Images and containers

We'll start with a brief note on the terminology used in this section of the course. We refer to both **_images_** and **_containers_**. What is the distinction between these two terms? 

**_Images_** are bundles of files including an operating system, software and potentially data and other application-related files. They may sometimes be referred to as a _disk image_ or _container image_ and they may be stored in different ways, perhaps as a single file, or as a group of files. Either way, we refer to this file, or collection of files, as an image.

A **_container_** is a virtual environment that is based on an image. That is, the files, applications, tools, etc that are available within a running container are determined by the image that the container is started from. It may be possible to start multiple container instances from an image. You could, perhaps, consider an image to be a form of template from which running container instances can be started.

## Getting an image and running a Singularity container

If you recall from learning about Docker, Docker images are formed of a set of _layers_ that make up the complete image. When you pull a Docker image from Docker Hub, you see the different layers being downloaded to your system. They are stored in your local Docker repository on your system and you can see details of the available images using the `docker` command.

Singularity images are a little different. Singularity uses the [Singularity Image Format (SIF)](https://github.com/sylabs/sif) and images are provided as single `SIF` files (with a `.sif` filename extension). Singularity images can be pulled from [Singularity Hub](https://singularity-hub.org/), a registry for container images. Singularity is also capable of running containers based on images pulled from [Docker Hub](https://hub.docker.com/) and some other sources. We'll look at accessing containers from Docker Hub later in the Singularity material.

> ## Singularity Hub
> Note that in addition to providing a repository that you can pull images from, [Singularity Hub](https://singularity-hub.org/) can also build Singularity images for you from a _**recipe**_ - a configuration file defining the steps to build an image. We'll look at recipes and building images later.
{: .callout}

Let's begin by creating a `test` directory, changing into it and _pulling_ a test _Hello World_ image from Singularity Hub:

~~~
$ mkdir test
$ cd test
$ singularity pull hello-world.sif shub://vsoch/hello-world
~~~
{: .language-bash}

~~~
INFO:    Downloading shub image
 59.75 MiB / 59.75 MiB [===============================================================================================================] 100.00% 52.03 MiB/s 1s
~~~
{: .output}

What just happened?! We pulled a SIF image from Singularity Hub using the `singularity pull` command and directed it to store the image file using the name `hello-world.sif` in the current directory. If you run the `ls` command, you should see that the `hello-world.sif` file is now present in the current directory. This is our image and we can now run a container based on this image:

~~~
$ singularity run hello-world.sif
~~~
{: .language-bash}

~~~
RaawwWWWWWRRRR!! Avocado!
~~~
{: .output}

The above command ran the _hello-world_ container from the image we downloaded from Singularity Hub and the resulting output was shown. 


How did the container determine what to do when we ran it?! What did running the container actually do to result in the displayed output?

When you run a container from a Singularity image without using any additional command line arguments, the container runs the default run script that is embedded within the image. This is a shell script that can be used to run commands, tools or applications stored within the image on container startup. We can inspect the image's run script using the `singularity inspect` command:

~~~
$ singularity inspect -r hello-world.sif
~~~
{: .language-bash}

~~~
#!/bin/sh 

exec /bin/bash /rawr.sh

~~~
{: .output}

This shows us the script within the `hello-world.sif` image configured to run by default when we use the `singularity run` command.

That concludes this introductory Singularity episode. The next episode looks in more detail at running containers.
