---
title: "Using Docker images with Singularity"
teaching: 5
exercises: 5
questions:
- "How do I use Docker images with Singularity?"
objectives:
- "Learn how to run Singularity containers based on Docker images."
keypoints:
- "Singularity can start a container from a Docker image which can be pulled directly from Docker Hub."
---

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
