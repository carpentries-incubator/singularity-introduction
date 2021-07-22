---
title: "Working with Singularity containers"
teaching: 30
exercises: 20
questions:
- "How do I run a shell or different commands within a container?"
- "Where does Singularity store images?"
objectives:
- "Learn about Singularity's image cache."
- "Understand how to run different commands when starting a container and open an interactive shell within a container environment."
- "Learn more about how singularity handles users and binds directories from the host filesystem."
- "Learn how to run Singularity containers based on Docker images."
keypoints:
- "Singularity caches downloaded images so that an image isn't downloaded again when it is requested using the `singularity pull` command."
- "The `singularity exec` and `singularity shell` commands provide different options for starting containers."
- "Singularity can start a container from a Docker image which can be pulled directly from Docker Hub."
---

## Singularity's image cache

While Singularity doesn't have a local image repository in the same way as Docker, it does cache downloaded image files. As we saw in the previous episode, images are simply `.sif` files stored on your local disk. 

If you delete a local `.sif` image that you have pulled from a remote image repository and then pull it again, if the image is unchanged from the version you previously pulled, you will be given a copy of the image file from your local cache rather than the image being downloaded again from the remote source. This removes unnecessary network transfers and is particularly useful for large images which may take some time to transfer over the network. To demonstrate this, remove the `hello-world.sif` file stored in your `test` directory and then issue the `pull` command again:

~~~
$ rm hello-world.sif
$ singularity pull hello-world.sif shub://vsoch/hello-world
~~~
{: .language-bash}

~~~
INFO:    Use image from cache
~~~
{: .output}

As we can see in the above output, the image has been returned from the cache and we don't see the output that we saw previously showing the image being downloaded from Singularity Hub.

How do we know what is stored in the local cache? We can find out using the `singularity cache` command:

~~~
$ singularity cache list
~~~
{: .language-bash}

~~~
There are 1 container file(s) using 62.65 MB and 0 oci blob file(s) using 0.00 kB of space
Total space used: 62.65 MB
~~~
{: .output}

This tells us how many container files are stored in the cache and how much disk space the cache is using but it doesn't tell us _what_ is actually being stored. To find out more information we can add the `-v` verbose flag to the `list` command:

~~~
$ singularity cache list -v
~~~
{: .language-bash}

~~~
NAME                     DATE CREATED           SIZE             TYPE
hello-world_latest.sif   2020-04-03 13:20:44    62.65 MB         shub

There are 1 container file(s) using 62.65 MB and 0 oci blob file(s) using 0.00 kB of space
Total space used: 62.65 MB
~~~
{: .output}

This provides us with some more useful information about the actual images stored in the cache. In the `TYPE` column we can see that our image type is `shub` because it's a `SIF` image that has been pulled from Singularity Hub. 

> ## Cleaning the Singularity image cache
> We can remove images from the cache using the `singularity cache clean` command. Running the command without any options will display a warning and ask you to confirm that you want to remove everything from your cache.
>
> You can also remove specific images or all images of a particular type. Look at the output of `singularity cache clean --help` for more information.
{: .callout}

## Working with containers

### Running specific commands within a container

We saw earlier that we can use the `singularity inspect` command to see the run script that a container is configured to run by default. What if we want to run a different command within a container, or we want to open a shell within a container that we can interact with?

If we know the path of an executable that we want to run within a container, we can use the `singularity exec` command. For example, using the `hello-world.sif` container that we've already pulled from Singularity Hub, we can run the following within the `test` directory where the `hello-world.sif` file is located:

~~~
$ singularity exec hello-world.sif /bin/echo Hello World!
~~~
{: .language-bash}

~~~
Hello World!
~~~
{: .output}

Here we see that a container has been started from the `hello-world.sif` image and the `/bin/echo` command has been run within the container, passing the input `Hello World!`. The command has echoed the provided input to the console and the container has terminated.

> ## Basic exercise: Running a different command within the "hello-world" container
>
> Can you run a container based on the `hello-world.sif` image that **prints the current date and time**?
> 
> > ## Solution
> >
> > ~~~
> > $ singularity exec hello-world.sif /bin/date
> > ~~~
> > {: .language-bash}
> > 
> > ~~~
> > Fri Jun 26 15:17:44 BST 2020
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

### Running a shell within a container

If you want to open an interactive shell within a container, Singularity provides the `singularity shell` command. Again, using the `hello-world.sif` image, and within our `test` directory, we can run a shell within a container from the hello-world image:

~~~
$ singularity shell hello-world.sif
~~~
{: .language-bash}

~~~
Singularity> whoami
[<your username>]
Singularity> ls
hello-world.sif
Singularity> 
~~~
{: .output}

As shown above, we have opened a shell in a new container started from the `hello-world.sif` image.

> ## Running a shell inside a Singularity container
>
> Q: What do you notice about the output of the above commands entered within the Singularity container shell?
> 
> Q: Does this differ from what you might see within a Docker container?
{: .discussion}

Use the `exit` command to exit from the container shell.

### Users, files and directories within a Singularity container

The first thing to note is that when you run `whoami` within the container you should see the username that you are signed in as on the host system when you run the container. For example, if my username is `jc1000`:

~~~
$ singularity shell hello-world.sif
Singularity> whoami
jc1000
~~~
{: .language-bash}

But hang on! I downloaded the standard, public version of the `hello-world.sif` image from Singularity Hub. I haven't customised it in any way. How is it configured with my own user details?!

If you have any familiarity with Linux system administration, you may be aware that in Linux, users and their Unix groups are configured in the `/etc/passwd` and `/etc/group` files respectively. In order for the shell within the container to know of my user, the relevant user information needs to be available within these files within the container.

Assuming this feature is enabled on your system, when the container is started, Singularity appends the relevant user and group lines from the host system to the `/etc/passwd` and `/etc/group` files within the container [\[1\]](https://www.intel.com/content/dam/www/public/us/en/documents/presentation/hpc-containers-singularity-advanced.pdf).

Singularity also _binds_ some directories from the host system where you are running the `singularity` command into the container that you're starting. Note that this bind process isn't copying files into the running container, it is simply making an existing directory on the host system visible and accessible within the container environment. If you write files to this directory within the running container, when the container shuts down, those changes will persist in the relevant location on the host system.

There is a default configuration of which files and directories are bound into the container but ultimate control of how things are set up on the system where you're running Singularity is determined by the system administrator. As a result, this section provides an overview but you may find that things are a little different on the system that you're running on.

One directory that is likely to be accessible within a container that you start is your _home directory_. The mapping of file content and directories from a host system into a Singularity container is illustrated in the example below showing a subset of the directories on the host Linux system and in a Singularity container:

~~~
Host system:                                                      Singularity container:
-------------                                                     ----------------------
/                                                                 /
├── bin                                                           ├── bin
├── etc                                                           ├── etc
│   ├── ...                                                       │   ├── ...
│   ├── group  ─> user's group added to group file in container ─>│   ├── group
│   └── passwd ──> user info added to passwd file in container ──>│   └── passwd
├── home                                                          ├── usr
│   └── jc1000 ───> user home directory made available ──> ─┐     ├── sbin
├── usr                 in container via bind mount         │     ├── home
├── sbin                                                    └────────>└── jc1000
└── ...                                                           └── ...

~~~
{: .output}

> ## Questions and exercises: Files in Singularity containers
>
> **Q1:** What do you notice about the ownership of files in a container started from the hello-world image? (e.g. take a look at the ownership of files in the root directory (`/`))
> 
> **Exercise 1:** In this container, try editing (for example using the editor `vi` which should be avaiable in the container) the `/rawr.sh` file. What do you notice?
>
> _If you're not familiar with `vi` there are many quick reference pages online showing the main commands for using the editor, for example [this one](http://web.mit.edu/merolish/Public/vi-ref.pdf)._
> 
> **Exercise 2:** In your home directory within the container shell, try and create a simple text file. Is it possible to do this? If so, why? If not, why not?! If you can successfully create a file, what happens to it when you exit the shell and the container shuts down?
>
> > ## Answers
> >
> > **A1:** Use the `ls -l` command to see a detailed file listing including file ownership and permission details. You should see that all the files are owned by you. This looks good - you should be ready to edit something in the exercise that follows...
> >
> > **A Ex1:** Unfortunately, it's not so easy, depending on how you tried to edit `/rawr.sh` you probably saw an error similar to the following: `Can't open file for writing` or `Read-only file system`
> > 
> > **A Ex2:** Within your home directory, you _should_ be able to successfully create a file. Since you're seeing your home directory on the host system which has been bound into the container, when you exit and the container shuts down, the file that you created within the container should still be present when you look at your home directory on the host system.
> {: .solution}
{: .challenge}

## Using Docker images with Singularity

Singularity can also start containers from Docker images, opening up access to a huge number of existing container images available on [Docker Hub](https://hub.docker.com/) and other registries.

While Singularity doesn't support running Docker images directly, it can pull them from Docker Hub and convert them into a suitable format for running via Singularity. When you pull a Docker image, Singularity pulls the slices or _layers_ that make up the Docker image and converts them into a single-file Singularity SIF image.

For example, moving on from the simple _Hello World_ examples that we've looked at so far, let's pull one of the [official Docker Python images](https://hub.docker.com/_/python). We'll use the image with the tag `3.8.2-slim-buster` which has Python 3.8.2 installed on Debian's [Buster](https://www.debian.org/releases/buster/) (v10) Linux distribution:

~~~
$ singularity pull python-3.8.2.sif docker://python:3.8.2-slim-buster
~~~
{: .language-bash}

~~~
INFO:    Converting OCI blobs to SIF format
INFO:    Starting build...
Copying blob 54fec2fa59d0 done
Copying blob cd3f35d84cab done
Copying blob a0afc8e92ef0 done
Copying blob 9691f23efdb7 done
Copying blob 6512e60b314b done
Copying config 1c498e093b done
Writing manifest to image destination
Storing signatures
2020/07/03 11:16:22  info unpack layer: sha256:54fec2fa59d0a0de9cd2dec9850b36c43de451f1fd1c0a5bf8f1cf26a61a5da4
2020/07/03 11:16:24  info unpack layer: sha256:cd3f35d84caba5a287676eeaea3d371e1ed5af8c57c33532228a456e0505b2d5
2020/07/03 11:16:24  info unpack layer: sha256:a0afc8e92ef0f5e56ddda03f8af40a4396226443a446e457ab6ed2dcdec62619
2020/07/03 11:16:25  info unpack layer: sha256:9691f23efdb7fd2829d06ad8fb9c8338487c183bb1aefa0d737cece2a612f51b
2020/07/03 11:16:25  info unpack layer: sha256:6512e60b314b980bce8ece057d15292db0f50ca12dbe6dd5752e1e54c64ccca2
INFO:    Creating SIF file...
INFO:    Build complete: python-3.8.2.sif
~~~
{: .output}

Note how we see singularity saying that it's "_Converting OCI blobs to SIF format_". We then see the layers of the Docker image being downloaded and unpacked and written into a single SIF file. Once the process is complete, we should see the python-3.8.2.sif image file in the current directory.

We can now run a container from this image as we would with any other singularity image.

> ## Running the Python 3.8.2 image that we just pulled from Docker Hub
>
> Try running the Python 3.8.2 image. What happens?
> 
> Try running some simple Python statements...
> 
> > ## Running the Python 3.8.2 image
> >
> > ~~~
> > $ singularity run python-3.8.2.sif
> > ~~~
> > {: .language-bash}
> > 
> > This should put you straight into a Python interactive shell within the running container:
> > 
> > ~~~
> > Python 3.8.2 (default, Apr 23 2020, 14:32:57)
> > [GCC 8.3.0] on linux
> > Type "help", "copyright", "credits" or "license" for more information.
> > >>> 
> > ~~~
> > Now try running some simple Python statements:
> > ~~~
> > >>> import math
> > >>> math.pi
> > 3.141592653589793
> > >>> 
> > ~~~
> > {: .language-python}
> {: .solution}
{: .challenge}

In addition to running a container and having it run the default run script, you could also start a container running a shell in case you want to undertake any configuration prior to running Python. This is covered in the following exercise:

> ## Open a shell within a Python container
>
> Try to run a shell within a singularity container based on the `python-3.8.2.sif` image. That is, run a container that opens a shell rather than the default Python interactive console as we saw above.
> See if you can find more than one way to achieve this.
> 
> Within the shell, try starting the Python interactive console and running some Python commands.
> 
> > ## Solution
> >
> > Recall from the earlier material that we can use the `singularity shell` command to open a shell within a container. To open a regular shell within a container based on the `python-3.8.2.sif` image, we can therefore simply run:
> > ~~~
> > $ singularity shell python-3.8.2.sif
> > ~~~
> > {: .language-bash}
> > 
> > ~~~
> > Singularity> echo $SHELL
> > /bin/bash
> > Singularity> cat /etc/issue
> > Debian GNU/Linux 10 \n \l
> > 
> > Singularity> exit
> > $ 
> > ~~~
> > {: .output}
> > 
> > It is also possible to use the `singularity exec` command to run an executable within a container. We could, therefore, use the `exec` command to run `/bin/bash`:
> > 
> > ~~~
> > $ singularity exec python-3.8.2.sif /bin/bash
> > ~~~
> > {: .language-bash}
> > 
> > ~~~
> > Singularity> echo $SHELL
> > /bin/bash
> > ~~~
> > {: .output}
> > 
> > You can run the Python console from your container shell simply by running the `python` command.
> {: .solution}
{: .challenge}

This concludes the second episode and Part I of the Singularity material. Part II contains a further two episodes where we'll look creating your own images and then more advanced use of containers for running MPI parallel applications.

## References

\[1\] Gregory M. Kurzer, Containers for Science, Reproducibility and Mobility: Singularity P2. Intel HPC Developer Conference, 2017. Available at: https://www.intel.com/content/dam/www/public/us/en/documents/presentation/hpc-containers-singularity-advanced.pdf
