---
title: "Singularity: Getting started"
start: true
teaching: 30
exercises: 20
questions:
- "What is a container and why might I want to use it?"
objectives:
- "Understand what a container is and when you might want to use it."
- "Undertake your first run of a simple Apptainer container."
keypoints:
- "Containers enable you to package up an application and its dependencies."
- "By using containers, you can better enforce reproducibility, portability and share-ability of your computational workflows."
- "Apptainer (and Singularity) are container platforms and are often used in cluster/HPC/research environments."
- "Apptainer has a different security model to other container platforms, one of the key reasons that it is well suited to HPC and cluster environments."
- "Apptainer has its own container image format based off the original Singularity Image Format (SIF)."
- "The `apptainer` command can be used to pull images from Docker Hub or other locations such as a website and run a container from an image file."
---

The episodes in this lesson will introduce you to the [Apptainer](https://apptainer.org/) container platform and demonstrate how to set up and use Apptainer.

This material is split into 2 parts:

*Part I: Basic usage, working with images*

 1. **Containers: Getting started**: This introductory episode

Working with containers:
<ol start="2">
 <li><strong>The apptainer cache: </strong> Why, where and how does Apptainer cache images locally?</li>
 <li><strong>Running commands within a container: </strong> How to run commands within a container.</li>
 <li><strong>Working with files and containers: </strong> Moving files into a container; accessing files on the host from within a container.</li>
 <li><strong>Using Docker images with Apptainer: </strong>How to run containers from Docker images.</li>
 </ol>
 *Part II: Creating images, running parallel codes*
 <ol start="6">
   <li><strong>Preparing to build container images</strong>: Getting started with the container.</li>
   <li><strong>Building container images</strong>: Explaining how to build and share your own container images.</li>
</ol>

> ## Work in progress
> This lesson is new material that is under ongoing development. We will introduce Apptainer and demonstrate how to work with it. As the tools and best practices continue to develop, elements of this material are likely to evolve. We welcome any comments or suggestions on how the material can be improved or extended.
{: .callout}

# Singularity - Part I

### Containers vs Virtual Machines

A container is an entity providing an isolated software environment (or filesystem) for an application and its dependencies.  

If you have already used a Virtual Machine, or VM, you're actually already familiar with some of the concepts of a container.

<!-- ![Containers vs. VMs]({{ page.root }}/fig/container_vs_vm.png) -->
<div>
<img src="{{ page.root }}/fig/container_vs_vm.png" alt="Containers vs. VMs" width="619" height="331"/>
<em format="display:block;text-align: center;margin:-20px 0 20px 0;>Credit: Pawsey Centre, <a href="https://pawseysc.github.io/sc19-containers/">Containers in HPC</a></em>
</div>
  
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

To build an image we need a recipe.  A recipe file is called a **Definition File**, or **def file**, in the *Apptainer* jargon and a **Dockerfile** in the *Docker* world.

### Container engines

A number of tools are available to create, deploy and run containerised applications.  Some of these will be covered throughout this tutorial:

* **Docker**: the first engine to gain popularity, still widely used in the IT industry.  Not very suitable for HPC as it requires *root* privileges to run. We'll use it mostly to build container images. See the extensive [online documentation](https://docs.docker.com/) for more information.

* **Singularity**: a simple, powerful *root*-less container engine for the HPC world. The main focus of this workshop. See the [user guide](https://sylabs.io/guides/latest/user-guide/) for extensive documentation.

* **Apptainer**: an open-source offshoot of **Singularity**. Provides all the same functionality as **Singularity** and moving forward will likely become the open-source standard. See the [user guide](https://apptainer.org/docs/user/main/) for extensive documentation.

## What is Apptainer

> ## Loading a module
> HPC systems often use *modules* to provide access to software on the system so you may need to use the command:
>
> ~~~
> $ module load Apptainer
> ~~~
>
> {: .language-bash}
> before you can use the `apptainer` command on the system.
{: .callout}

~~~
apptainer --version
~~~
{: .language-bash}

~~~
apptainer version 1.1.5-dirty
~~~
{: .output}

Depending on the version of Apptainer installed on your system, you may see a different version. At the time of writing, `v3.5.3` is the latest release of Singularity.

## Getting an image and running a container

We will be working from the training project directory `/nesi/project/nesi99991/singularity_ernzz2023`.

```
cd /nesi/project/nesi99991/singularity_ernzz2023
```


Let's begin by creating a directory with *your username*, changing into it and *pulling* a test *Hello World* image from Singularity Hub:

~~~
mkdir test usrname123
cd usrname123
apptainer pull docker://ghcr.io/apptainer/lolcow
~~~
{: .language-bash}

~~~
INFO:    Converting OCI blobs to SIF format
INFO:    Starting build...
Getting image source signatures
Copying blob 5ca731fc36c2 done
Copying blob 16ec32c2132b done
Copying config fd0daa4d89 done
Writing manifest to image destination
Storing signatures
2023/02/09 12:20:21  info unpack layer: sha256:16ec32c2132b43494832a05f2b02f7a822479f8250c173d0ab27b3de78b2f058
2023/02/09 12:20:24  info unpack layer: sha256:5ca731fc36c28789c5ddc3216563e8bfca2ab3ea10347e07554ebba1c953242e
INFO:    Creating SIF file...
~~~
{: .output}

What just happened?! We pulled a Docker image from Docker Hub using the `apptainer pull` command and directed it to store the image file using the name`lolcow_latest.sif`in the current directory. If you run the `ls` command, you should see that the `lolcow_latest.sif` file is now present in the current directory. This is our image and we can now run a container based on this image:

~~~
apptainer run lolcow_latest.sif
~~~
{: .language-bash}

~~~
 _________________________
< Wed Feb 8 23:36:16 2023 >
 -------------------------
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
 _________________________
< Wed Feb 8 23:36:36 2023 >
 -------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
~~~
{: .output}

How did the container determine what to do when we ran it?! What did running the container actually do to result in the displayed output?

When you run a container from a sif image without using any additional command line arguments, the container runs the default run script that is embedded within the image. This is a shell script that can be used to run commands, tools or applications stored within the image on container startup. We can inspect the image's run script using the `apptainer inspect` command:

~~~
apptainer inspect -r lolcow_latest.sif | head
~~~
{: .language-bash}

~~~
#!/bin/sh
OCI_ENTRYPOINT='"/bin/sh" "-c" "date | cowsay | lolcat"'
OCI_CMD=''

# When SINGULARITY_NO_EVAL set, use OCI compatible behavior that does
# not evaluate resolved CMD / ENTRYPOINT / ARGS through the shell, and
# does not modify expected quoting behavior of args.
if [ -n "$SINGULARITY_NO_EVAL" ]; then
    # ENTRYPOINT only - run entrypoint plus args
    if [ -z "$OCI_CMD" ] && [ -n "$OCI_ENTRYPOINT" ]; then
~~~
{: .output}

This shows us the first 10 lines of the script within the `lolcow_latest.sif` image configured to run by default when we use the `apptainer run` command.

That concludes this introductory container episode. The next episode looks in more detail at running containers.
