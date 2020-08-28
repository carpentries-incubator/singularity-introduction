---
title: "Building Singularity images"
teaching: 30
exercises: 30
questions:
- "How do I create my own Singularity images?"
objectives:
- "Understand the different Singularity container file formats."
- "Understand how to build and share your own Singularity containers."
keypoints:
- "Singularity definition files are used to define the build process and configuration for an image."
- "Singularity's Docker container provides a way to build images on a platform where Singularity is not installed but Docker is available."
- "Existing images from remote registries such as Docker Hub and Singularity Hub can be used as a base for creating new Singularity images."
---

# Singularity - Part II

## Brief recap

In the two episodes covering Part I of this Singularity material we've seen how Singularity can be used on a computing platform where you don't have any administrative privileges. The software was pre-installed and it was possible to work with existing images such as Singularity image files already stored on the platform or images obtained from a remote image repository such as Singularity Hub or Docker Hub.

It is clear that between Singularity Hub and Docker Hub there is a huge array of images available but what if you want to create your own images or customise existing images?

In this first of two episodes in Part II of the Singularity material, we'll look at building Singularity images.

## Preparing to use Singularity for building images

So far you've been able to work with Singularity from your own user account as a non-privileged user. This part of the Singularity material requires that you use Singularity in an environment where you have administrative (root) access. While it is possible to build Singularity containers without root access, it is highly recommended that you do this as the _root_ user, as highlighted in [this section](https://sylabs.io/guides/3.5/user-guide/build_a_container.html#creating-writable-sandbox-directories) of the Singularity documentation. Bear in mind that the system that you use to build containers doesn't have to be the system where you intend to run the containers. If, for example, you are intending to build a container that you can subsequently run on a Linux-based cluster, you could build the container on your own Linux-based desktop or laptop computer. You could then transfer the built image directly to the target platform or upload it to an image repository and pull it onto the target platform from this repository.

There are three different options for accessing a suitable environment to undertake the material in this part of the course:

 1. Run Singularity from within a Docker container - this will enable you to have the required privileges to build images
 1. Install Singularity locally on a system where you have administrative access
 1. Use Singularity on a system where it is already pre-installed and you have administrative (root) access

We'll focus on the first option in this part of the course. If you would like to install Singularity directly on your system, see the box below for some further pointers. Note that the installation process is an advanced task that is beyond the scope of this course so we won't be covering this.

> ## Installing Singularity on your local system (optional) \[Advanced task\]
>
> If you are running Linux and would like to install Singularity locally on your system, Singularity provide the free, open source [Singularity Community Edition](https://github.com/hpcng/singularity/releases). You will need to install various dependencies on your system and then build Singularity from source code.
>
> _If you are not familiar with building applications from source code, it is strongly recommended that you use the Docker Singularity image, as described below in the "Getting started with the Docker Singularity image" section rather than attempting to build and install Singularity yourself. The installation process is an advanced task that is beyond the scope of this session._
> 
> However, if you have Linux systems knowledge and would like to attempt a local install of Singularity, you can find details in the [INSTALL.md](https://github.com/sylabs/singularity/blob/master/INSTALL.md) file within the Singularity repository that explains how to install the prerequisites and build and install the software. Singularity is written in the [Go](https://golang.org/) programming language and Go is the main dependency that you'll need to install on your system. The process of installing Go and any other requirements is detailed in the INSTALL.md file.
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
> - Using your previously acquired Docker knowledge, get the Singularity image for `v3.5.3` and ensure that you can run a Docker container using this image.
> 
> - Create a directory (e.g. `$HOME/singularity_data`) on your host machine that you can use for storage of _definition files_ (we'll introduce these shortly) and generated image files. 
> 
>   This directory should be bind mounted into the Docker container at the location `/home/singularity` every time you run it - this will give you a location in which to store built images so that they are available on the host system once the container exits. (take a look at the `-v` switch)
> 
> _Hint: To be able to build an image using the Docker Singularity container, you'll need to add the `--privileged` switch to your docker command line._
> 
> Questions:
> 
> - What is happening when you run the container?
> - Can you run an interactive shell in the container?
> 
> > ## Running the image
> > Having a bound directory from the host system accessible within your running Singularity container will give you somewhere to place created images so that they are accessible on the host system after the container exits. Begin by changing into the directory that you created above for storing your definiton files and built images (e.g. `$HOME/singularity_data`). 
> >
> > You may choose to:
> >   - open a shell within the Docker image so you can work at a command prompt and run the `singularity` command directly
> >   - use the `docker run` command to run a new container instance every time you want to run the `singularity` command.
> > 
> > Either option is fine for this section of the material.
> > 
> > _Some examples:_
> > 
> > To run the `singularity` command within the docker container directly from the host system's terminal:
> > ```
> > docker run --privileged --rm -v ${PWD}:/home/singularity quay.io/singularity/singularity:v3.5.3 cache list
> > ```
> > 
> > To start a shell within the Singularity Docker container where the `singularity` command can be run directly:
> > ```
> > docker run -it --entrypoint=/bin/bash --privileged --rm -v ${PWD}:/home/singularity quay.io/singularity/singularity:v3.5.3
> > ```
> > 
> > To make things easier to read in the remainder of the material, command examples will use the `singularity` command directly, e.g. `singularity cache list`. If you're running a shell in the Docker container, you can enter the commands as they appear. If you're using the container's default run behaviour and running a container instance for each run of the command, you'll need to replace `singularity` with `docker run --privileged -v ${PWD}:/home/singularity quay.io/singularity/singularity:v3.5.3` or similar.
> {: .solution}
{: .challenge}

## Building Singularity images

### Introduction

As a platform that is widely used in the scientific/research software and HPC communities, Singularity provides great support for reproducibility. If you build a Singularity container for some scientific software, it's likely that you and/or others will want to be able to reproduce exactly the same environment again. Maybe you want to verify the results of the code or provide a means that others can use to verify the results to support a paper or report. Maybe you're making a tool available to others and want to ensure that they have exactly the right version/configuration of the code.

Similarly to Docker and many other modern software tools, Singularity follows the "Configuration as code" approach and a container configuration can be stored in a file which can then be committed to your version control system alongside other code. Assuming it is suitably configured, this file can then be used by you or other individuals (or by automated build tools) to reproduce a container with the same configuration at some point in the future.

### Different approaches to building images

There are various approaches to building Singularity images. We highlight two different approaches here and focus on one of them:

 - _Building within a sandbox:_ You can build a container interactively within a sandbox environment. This means you get a shell within the container environment and install and configure packages and code as you wish before exiting the sandbox and converting it into a container image.
- _Building from a [Singularity Definition File](https://sylabs.io/guides/3.5/user-guide/build_a_container.html#creating-writable-sandbox-directories)_: This is Singularity's equivalent to building a Docker container from a `Dockerfile` and we'll discuss this approach in this section.

You can take a look at Singularity's "[Build a Container](https://sylabs.io/guides/3.5/user-guide/build_a_container.html#creating-writable-sandbox-directories)" documentation for more details on different approaches to building containers.

> ## Why look at Singularity Definition Files?
> Why do you think we might be looking at the _definition file approach_ here rather than the _sandbox approach_?
>
> > ## Discussion
> > The sandbox approach is great for prototyping and testing out an image configuration but it doesn't provide the best support for our ultimate goal of _reproducibility_. If you spend time sitting at your terminal in front of a shell typing different commands to add configuration, maybe you realise you made a mistake so you undo one piece of configuration and change it. This goes on until you have your completed configuration but there's no explicit record of exactly what you did to create that configuration. 
> > 
> > Say your container image file gets deleted by accident, or someone else wants to create an equivalent image to test something. How will they do this and know for sure that they have the same configuration that you had?
> > With a definition file, the configuration steps are explicitly defined and can be easily stored (and re-run).
> > 
> > Definition files are small text files while container files may be very large, multi-gigabyte files that are difficult and time consuming to move around. This makes definition files ideal for storing in a version control system along with their revisions.
> {: .solution}
{: .challenge}

### Creating a Singularity Definition File

A Singularity Definition File is a text file that contains a series of statements that are used to create a container image. In line with the _configuration as code_ approach mentioned above, the definition file can be stored in your code repository alongside your application code and used to create a reproducible image. This means that for a given commit in your repository, the version of the definition file present at that commit can be used to reproduce a container with a known state. It was pointed out earlier in the course, when covering Docker, that this property also applies for Dockerfiles.

We'll now look at a very simple example of a definition file:

~~~
Bootstrap: docker
From: ubuntu:20.04

%post
    apt-get -y update && apt-get install -y python

%runscript
    python -c 'print("Hello World! Hello from our custom Singularity image!")'
~~~
{: .language-bash}

A definition file has a number of optional sections, specified using the `%` prefix, that are used to define or undertake different configuration during different stages of the image build process. You can find full details in Singularity's [Definition Files documentation](https://sylabs.io/guides/3.5/user-guide/definition_files.html). In our very simple example here, we only use the `%post` and `%runscript` sections.

Let's step through this definition file and look at the lines in more detail:

~~~
Bootstrap: docker
From: ubuntu:20.04
~~~
{: .language-bash}

These first two lines define where to _bootstrap_ our image from. Why can't we just put some application binaries into a blank image? Any applications or tools that we want to run will need to interact with standard system libraries and potentially a wide range of other libraries and tools. These need to be available within the image and we therefore need some sort of operating system as the basis for our image. The most straightforward way to achieve this is to start from an existing base image containing an operating system. In this case, we're going to start from a minimal Ubuntu 20.04 Linux Docker image. Note that we're using a Docker image as the basis for creating a Singularity image. This demonstrates the flexibility in being able to start from different types of images when creating a new Singularity image.

The `Bootstrap: docker` line is similar to prefixing an image path with `docker://` when using, for example, the `singularity pull` command. A range of [different bootstrap options](https://sylabs.io/guides/3.5/user-guide/definition_files.html#preferred-bootstrap-agents) are supported. `From: ubuntu:20.04` says that we want to use the `ubuntu` image with the tag `20.04`.

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
    python3 -c 'print("Hello World! Hello from our custom Singularity image!")'
~~~
{: .language-bash}

This section is used to define a script that should be run when a container is started based on this image using the `singularity run` command. In this simple example we use `python3` to print out some text to the console.

We can now save the contents of the simple defintion file shown above to a file and build an image based on it. In the case of this example, the definition file has been named `my_test_image.def`. (Note that the instructions here assume you've bound the image output directory you created to the `/home/singularity` directory in your Docker Singularity container):

~~~
$ singularity build /home/singularity/my_test_image.sif /home/singularity/my_test_image.def
~~~
{: .language-bash}

Recall from the details at the start of this section that if you are running your command from the host system command line, running an instance of a Docker container for each run of the command, your command will look something like this:

~~~
$ docker run --privileged --rm -v ${PWD}:/home/singularity quay.io/singularity/singularity:v3.5.3 build /home/singularity/my_test_image.sif /home/singularity/my_test_image.def
~~~
{: .language-bash}

The above command requests the building of an image based on the `my_test_image.def` file with the resulting image saved to the `my_test_image.sif` file. Note that you will need to prefix the command with `sudo` if you're running a locally installed version of Singularity and not running via Docker because it is necessary to have administrative privileges to build the image. You should see output similar to the following:

~~~
INFO:    Starting build...
Getting image source signatures
Copying blob d51af753c3d3 skipped: already exists
Copying blob fc878cd0a91c skipped: already exists
Copying blob 6154df8ff988 skipped: already exists
Copying blob fee5db0ff82f skipped: already exists
Copying config 95c3f3755f done
Writing manifest to image destination
Storing signatures
2020/04/29 13:36:35  info unpack layer: sha256:d51af753c3d3a984351448ec0f85ddafc580680fd6dfce9f4b09fdb367ee1e3e
2020/04/29 13:36:36  info unpack layer: sha256:fc878cd0a91c7bece56f668b2c79a19d94dd5471dae41fe5a7e14b4ae65251f6
2020/04/29 13:36:36  info unpack layer: sha256:6154df8ff9882934dc5bf265b8b85a3aeadba06387447ffa440f7af7f32b0e1d
2020/04/29 13:36:36  info unpack layer: sha256:fee5db0ff82f7aa5ace63497df4802bbadf8f2779ed3e1858605b791dc449425
INFO:    Running post scriptlet
+ apt-get -y update
Get:1 http://archive.ubuntu.com/ubuntu focal InRelease [265 kB]
...
  [Package update output truncated]
...
Fetched 13.4 MB in 2s (5575 kB/s)                            
Reading package lists... Done
+ apt-get install -y python3
Reading package lists... Done
...
  [Package install output truncated]
...Processing triggers for libc-bin (2.31-0ubuntu9) ...
INFO:    Adding runscript
INFO:    Creating SIF file...
INFO:    Build complete: my_test_image.sif
$ 
~~~
{: .output}

You should now have a `my_test_image.sif` file in the current directory. Note that in the above output, where it says `INFO:  Starting build...` there is a series of `skipped: already exists` messages for the `Copying blob` lines. This is because the Docker image slices for the Ubuntu 20.04 image have previously been downloaded and are cached on the system where this example is being run. On your system, if the image is not already cached, you will see the slices being downloaded from Docker Hub when these lines of output appear.

> ## Permissions of the created image file
>
> You may find that the created Singularity image file on your host filesystem is owned by the `root` user and not your user. In this case, you won't be able to change the ownership/permissions of the file directly if you don't have root access.
>
> However, the image file will be readable by you and you should be able to take a copy of the file under a new name which you will then own. You will then be able to modify the permissions of this copy of the image and delete the original root-owned file since the default permissions should allow this.
> 
{: .callout}

Now move your created `.sif` image file to a platform with an installation of Singularity. You could, for example, do this using the command line secure copy command `scp`.

> ## Cluster platform configuration for running Singularity containers
>
> On the cluster platform that we're using for the course, it is necesary to setup a shared temporary storage space for Singularity to use because it is not possible for it to use the standard `/tmp` directory on this platform.
>
> First create a directory in the `/lustre/home/shared` directory. It is recommended that you create a directory named `$USER-singularity`. We then need to set Singularity's temporary directory environment variable to point to this location. Run the following commands:
>
> ~~~
> mkdir /lustre/home/shared/$USER-singularity
> export TMPDIR=/lustre/home/shared/$USER-singularity
> export SINGULARITY_TMPDIR=$TMPDIR
> ~~~
> {: .language-bash}
>
> When running Singularity containers on this platform, you'll need to set `SINGULARITY_TMPDIR` in each shell session that you open. However, you could add these commands to your `~/.bashrc` or `~/.bash_profile` so that the values are set by default in each shell that you open.
> 
{: .callout}

It is recommended that you move the create `.sif` file to a platform with an installation of Singularity, rather than attempting to run the image using the Docker container. However, if you do try to use the Docker container, see the notes below on "_Using singularity run from within the Docker container_" for further information.

Now that we've built an image, we can attempt to run it:

~~~
$ singularity run my_test_image.sif
~~~
{: .language-bash}

If everything worked successfully, you should see the message printed by Python:

~~~
Hello World! Hello from our custom Singularity image!
~~~
{: .output}

> ## Using `singularity run` from within the Docker container
>
> It is strongly recommended that you don't use the Docker container for running Singularity images, only for creating then, since the Singularity command runs within the container as the root user.
> 
> However, for the purposes of this simple example, if you are trying to run the container using the `singularity` command from within the Docker container, it is likely that you will get an error relating to `/etc/localtime` similar to the following:
>
> ~~~
> WARNING: skipping mount of /etc/localtime: no such file or directory
> FATAL:   container creation failed: mount /etc/localtime->/etc/localtime error: while mounting /etc/localtime: mount source /etc/localtime doesn't exist
> ~~~
> {: .output}
> 
> This occurs because the `/etc/localtime` file that provides timezone configuration is not present within the Docker container. If you want to use the Docker container to test that your newly created image runs, you'll need to open a shell in the Docker container and add a timezone configuration as described in the [Alpine Linux documentation](https://wiki.alpinelinux.org/wiki/Setting_the_timezone):
>
> ~~~
> $ apk add tzdata
> $ cp /usr/share/zoneinfo/Europe/London /etc/localtime
> ~~~
> {: .language-bash}
> 
> The `singularity run` command should now work successfully.
{: .callout}


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

### Additional Singularity features

Singularity has a wide range of features. You can find full details in the [Singularity User Guide](https://sylabs.io/guides/3.5/user-guide/index.html) and we highlight a couple of key features here that may be of use/interest:

**Remote Builder Capabilities:** If you have access to a platform with Singularity installed but you don't have root access to create containers, you may be able to use the [Remote Builder](https://cloud.sylabs.io/builder) functionality to offload the process of building an image to remote cloud resources. You'll need to register for a _cloud token_ via the link on the [Remote Builder](https://cloud.sylabs.io/builder) page.

**Signing containers:** If you do want to share container image (`.sif`) files directly with colleagues or collaborators, how can the people you send an image to be sure that they have received the file without it being tampered with or suffering from corruption during transfer/storage? And how can you be sure that the same goes for any container image file you receive from others? Singularity supports signing containers. This allows a digital signature to be linked to an image file. This signature can be used to verify that an image file has been signed by the holder of a specific key and that the file is unchanged from when it was signed. You can find full details of how to use this functionality in the Singularity documentation on [Signing and Verifying Containers](https://sylabs.io/guides/3.0/user-guide/signNverify.html).

