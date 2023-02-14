---
title: "Creating Container Images"
teaching: 15
exercises: 20
questions:
- "How can I make my own Singularity container images?"
- "How do I document the 'recipe' for a Singularity container image"
objectives:
- "Explain the purpose of a `Singularity Definition File` and show some simple examples."
- "Understand the different Singularity container file formats."
- "Understand how to build and share your own Singularity containers."
- "??Compare the steps of creating a container image interactively versus a `Dockerfile`."

keypoints:
- "`Definition files`s specify what is within Singularity container images."
- "The `singularity build` command is used to build a container image from a `Definition file`."
- "Singularity definition files are used to define the build process and configuration for an image."
- "Existing images from remote registries such as Docker Hub and Singularity Hub can be used as a base for creating new Singularity images."
---

There are lots of reasons why you might want to create your **own** {{ site.software.name }} container image.

- You can't find an existing container image with all the tools you need.
- You want to have a container image to "archive" all the specific software versions you ran for a project.
- You want to share your workflow with someone else.

## Sandbox installation

The most intuative way to build a container is to do so interactively, this allows you to install packages, configure applications and test commands, then when finished export as an image.

This is possible using the `--sandbox` flag, for example 

```bash
sudo {{ site.software.cmd }} build --sandbox ubuntu docker://ubuntu
```

This creates an image called  `ubuntu` bootstrapped from `docker://ubuntu`
You can then run

```bash
sudo {{ site.software.cmd }} shell --writable ubuntu
```

To start setting up your workflow.

However, there are two big problems with this approach, firstly building a sandbox image requires you to have root access on your machine and therefore unavailable to many people, secondly it doesn't provide the best support for our ultimate goal of _reproducibility_.

This is because, even though you can share the image, the steps taken to create it are unclear.
`-sandbox` should only be used for initial prototyping of your image, the rest of the time you should use a _definition file_.

<!-- 
## Sandbox installation

You can build a container interactively within a sandbox environment. This means you get a shell within the container environment and install and configure packages and code as you wish before exiting the sandbox and converting it into a container image.

> ## Sandbox Root Privilege
>
> Building a sandbox image requires you to have root access on your machine.

To build into a sandbox use the `build --sandbox` command and option:

```bash
sudo singularity build --sandbox ubuntu/ library://ubuntu
```

This command creates a directory called `ubuntu/` with an entire Ubuntu Operating System and some Singularity metadata in your current working directory.

You can use commands like shell, exec , and run with this directory just as you would with a Singularity image. If you pass the --writable option when you use your container you can also write files within the sandbox directory (provided you have the permissions to do so).

Can you apt-get? if so, include example.

```bash
sudo singularity exec --writable ubuntu touch /foo
```

```bash
singularity exec ubuntu/ ls /foo
/foo
```

> > ## Discussion
> > The sandbox approach is great for prototyping and testing out an image configuration but it doesn't provide the best support for our ultimate goal of _reproducibility_. If you spend time sitting at your terminal in front of a shell typing different commands to add configuration, maybe you realise you made a mistake so you undo one piece of configuration and change it. This goes on until you have your completed, working configuration but there's no explicit record of exactly what you did to create that configuration. 
> > 
> > Say your container image file gets deleted by accident, or someone else wants to create an equivalent image to test something. How will they do this and know for sure that they have the same configuration that you had?
> > With a definition file, the configuration steps are explicitly defined and can be easily stored (and re-run).
> > 
> > Definition files are small text files while container files may be very large, multi-gigabyte files that are difficult and time consuming to move around. This makes definition files ideal for storing in a version control system along with their revisions.
> {: .solution}
{: .challenge} -->

### Creating a {{ site.software.name }} Definition File

A {{ site.software.name }} Definition File is a text file that contains a series of statements that are used to create a container image. In line with the _configuration as code_ approach mentioned above, the definition file can be stored in your code repository alongside your application code and used to create a reproducible image. This means that for a given commit in your repository, the version of the definition file present at that commit can be used to reproduce a container with a known state. It was pointed out earlier in the course, when covering Docker, that this property also applies for Dockerfiles.

We'll now look at a very simple example of a definition file:

~~~
Bootstrap: docker
From: ubuntu:20.04

%post
    apt-get -y update && apt-get install -y python

%runscript
    python -c 'print("Hello World! Hello from our custom {{ site.software.name }} image!")'
~~~
{: .language-bash}

A definition file has a number of optional sections, specified using the `%` prefix, that are used to define or undertake different configuration during different stages of the image build process. You can find full details in {{ site.software.name }}'s [Definition Files documentation](https://sylabs.io/guides/3.5/user-guide/definition_files.html). In our very simple example here, we only use the `%post` and `%runscript` sections.

Let's step through this definition file and look at the lines in more detail:

~~~
Bootstrap: docker
From: ubuntu:20.04
~~~
{: .language-bash}

These first two lines define where to _bootstrap_ our image from. Why can't we just put some application binaries into a blank image? Any applications or tools that we want to run will need to interact with standard system libraries and potentially a wide range of other libraries and tools. These need to be available within the image and we therefore need some sort of operating system as the basis for our image. The most straightforward way to achieve this is to start from an existing base image containing an operating system. In this case, we're going to start from a minimal Ubuntu 20.04 Linux Docker image. Note that we're using a Docker image as the basis for creating a {{ site.software.name }} image. This demonstrates the flexibility in being able to start from different types of images when creating a new {{ site.software.name }} image.

The `Bootstrap: docker` line is similar to prefixing an image path with `docker://` when using, for example, the `{{ site.software.cmd }} pull` command. A range of [different bootstrap options](https://sylabs.io/guides/3.5/user-guide/definition_files.html#preferred-bootstrap-agents) are supported. `From: ubuntu:20.04` says that we want to use the `ubuntu` image with the tag `20.04` from Docker Hub.

Next we have the `%post` section of the definition file:

~~~
%post
    apt-get -y update && apt-get install -y python3
~~~
{: .language-bash}

In this section of the file we can do tasks such as package installation, pulling data files from remote locations and undertaking local configuration within the image. The commands that appear in this section are standard shell commands and they are run _within_ the context of our new container image. So, in the case of this example, these commands are being run within the context of a minimal Ubuntu 20.04 image that initially has only a very small set of core packages installed.

Here we use Ubuntu's package manager to update our package indexes and then install the `python3` package along with any required dependencies. The `-y` switches are used to accept, by default, interactive prompts that might appear asking you to confirm package updates or installation. This is required because our definition file should be able to run in an unattended, non-interactive environment.

Finally we have the `%runscript` section:

~~~
%runscript
    python3 -c 'print("Hello World! Hello from our custom {{ site.software.name }} image!")'
~~~
{: .language-bash}

This section is used to define a script that should be run when a container is started based on this image using the `{{ site.software.cmd }} run` command. In this simple example we use `python3` to print out some text to the console.

### More advanced definition files

Here we've looked at a very simple example of how to create an image. At this stage, you might want to have a go at creating your own definition file for some code of your own or an application that you work with regularly. There are several definition file sections that were _not_ used in the above example, these are:

 - `%setup`
 - `%files`
 - `%environment`
 - `%startscript`
 - `%test`
 - `%labels`
 - `%help`

The [`Sections` part of the definition file documentation](https://sylabs.io/guides/3.5/user-guide/definition_files.html#sections) details all the sections and provides an example definition file that makes use of all the sections.


### Useful base images

At the time of writing, [Docker Hub](https://hub.docker.com) is the most popular web registry for general purpose container images. Therefore all images mentioned below are hosted in this registry.

#### CUDA

[nvidia/cuda](https://hub.docker.com/r/nvidia/cuda) has images to build GPU enabled applications. There are different image types for different needs. Tags containing `runtime` are suitable for binary applications that are ready to run; if you need to compile GPU code, pick tags containing `devel` instead. Different OS flavours are available, too.

#### MPI

As you can see in the episode on MPI applications, when containerising this type of software the MPI libraries in the image need to be ABI compatible with the MPI libraries in the host. The Pawsey Supercomputing Centre maintains some **MPICH** base images at [pawsey/mpi-base](https://hub.docker.com/r/pawsey/mpi-base), for building images that will run on our HPC systems.

#### Python

[python](https://hub.docker.com/_/python) hosts the official Python images. Different versions are available for some OS flavours. At the time of writing the default image tag corresponds to Python 3.8 on Debian 10. Smaller base images have tags ending with `-slim`.

[continuumio/miniconda3](https://hub.docker.com/r/continuumio/miniconda3) are images provided by the maintainers of the [Anaconda](https://anaconda.org) project. They ship with Python 3, as well as `pip` and `conda` to install and manage packages. At the time of writing, the most recent version is `4.7.12`, based on Python `3.7.4`.

If you need interactive Jupyter Notebooks, [Jupyter Docker Stacks](https://jupyter-docker-stacks.readthedocs.io/en/latest/) maintain a series of dedicated container images. Among others, there is the base SciPy image [jupyter/scipy-notebook](https://hub.docker.com/r/jupyter/scipy-notebook), the data science image [jupyter/datascience-notebook](https://hub.docker.com/r/jupyter/datascience-notebook), and the machine learning image [jupyter/tensorflow-notebook](https://hub.docker.com/r/jupyter/tensorflow-notebook).

#### R

The [Rocker Project](https://www.rocker-project.org) maintains a number of good R base images. Of particular relevance is [rocker/tidyverse](https://hub.docker.com/r/rocker/tidyverse), which embeds the basic R distribution, an RStudio web-server installation and the [tidyverse](https://www.tidyverse.org) collection of packages for data science. At the time of writing, the most recent version is `3.6.1`.

Other more basic images are [rocker/r-ver](https://hub.docker.com/r/rocker/r-ver) (R only) and [rocker/rstudio](https://hub.docker.com/r/rocker/rstudio) (R + RStudio).
