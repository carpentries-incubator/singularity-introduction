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


## Preparing to use Singularity for building images

So far you've been able to work with Singularity from your own user account as a non-privileged user. This part of the Singularity material requires that you use Singularity in an environment where you have administrative (root) access.

There are a couple of different ways to work around this restriction.

<table>
<thead>
  <tr>
    <th></th>
    <th></th>
    <th>pros</th>
    <th>cons</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>Install Singularity locally on a system where you do have administrative access.</td>
    <td></td>
    <td></td>
    <td>Not possible for many people</td>
  </tr>
  <tr>
    <td>Build your container from within another container</td>
    <td></td>
    <td></td>
    <td>A bit contrived</td>
  </tr>
  <tr>
    <td>Use a 'remote build service' to build your container</td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>Simulate root access using the `--fakeroot` feature</td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
</tbody>
</table>

We'll focus on the last two options in this part of the course - _Use a 'remote build service' to build your container_ and _Simulate root access using the `--fakeroot` feature_.


### Remote build

What if you need to build an image from a system where you don't have admin privileges, *i.e.* you can't run commands with *sudo*?

Singularity offers the option to run build remotely, using a **Remote Builder** we will be using the default provided by Sylabs; You will need a Sylabs account and a token to use this feature.

```
singularity remote login
```
{: .bash}

```
Generate an API Key at https://cloud.sylabs.io/auth/tokens, and paste here:
API Key:
```
{: .output}

Now paste the token you had copied to the clipboard end press `Enter`:

```
INFO:    API Key Verified!
```
{: .output}

With this set up, you may use `singularity build -r` to start the remote build. Once finished, the image will be downloaded so that it's ready to use:

```
singularity build -r lolcow_remote.sif lolcow.def
```
{: .bash}

```
INFO:    Remote "default" added.
INFO:    Authenticating with remote: default
INFO:    API Key Verified!
INFO:    Remote "default" now in use.
INFO:    Starting build...
[..]
INFO:    Running post scriptlet
[..]
INFO:    Adding help info
INFO:    Adding labels
INFO:    Adding environment to container
INFO:    Adding runscript
INFO:    Creating SIF file...
INFO:    Build complete: /tmp/image-699539270
WARNING: Skipping container verifying
 67.07 MiB / 67.07 MiB  100.00% 14.18 MiB/s 4s
```
{: .output}

At the time of writing, when using the Remote Builder you won't be able to use the `%files` header in the def file, to copy host files into the image.

You are now ready to push your image to the Cloud Library, *e.g.* via `singularity push`:

```
singularity push -U lolcow.sif library://<YOUR-SYLABS-USERNAME>/default/lolcow:30oct19
```
{: .bash}

```
WARNING: Skipping container verifying
 67.08 MiB / 67.08 MiB [==================================================================================================================================] 100.00% 6.37 MiB/s 10s
```
{: .output}

Note the use of the flag `-U` to allow pushing unsigned containers (see further down).  
Also note once again the format for the registry: <user>/<user-collection>/<name>:<tag>.

Finally, you (or other peers) are now able to pull your image from the Cloud Library:

```
singularity pull -U library://<YOUR-SYLABS-USERNAME>/default/lolcow:30oct19
```
{: .bash}

```bash
INFO:    Downloading library image
 67.07 MiB / 67.07 MiB [===================================================================================================================================] 100.00% 8.10 MiB/s 8s
WARNING: Skipping container verification
INFO:    Download complete: lolcow_30oct19.sif
```
{: .output}

> ### Service Status
> 
> Mahuika's new nodes are in an **Early Access Programme (EAP) phase** and not fully in production.
> 
> See [Mahuika Extension Onboarding](/hc/en-gb/articles/5002335382543) for more information about it.

This article describes a technique to build [Apptainer](https://apptainer.org/) containers using Mahuika Extension nodes, via a Slurm job. You can also build [Singularity](/hc/en-gb/articles/360001107916) container using this technique.

## Build Environment Variables

To build containers, you need to ensure that Apptainer has enough storage space to create intermediate files. It also requires a cache folder to save images pulled from a different location (e.g. DockerHub). By default both of these locations are set to `/tmp` which has limited space, large builds may exceed this limitation causing the builder to crash.

The environment variables `APPTAINER_TMPDIR` and `APPTAINER_CACHEDIR` environment can be used to overwrite the default location of these directories.

```bash
export APPTAINER_CACHEDIR=/nesi/nobackup/PROJECTID/apptainer_cache
export APPTAINER_TMPDIR=/nesi/nobackup/PROJECTID/apptainer_tmpdir
mkdir -p $APPTAINER_CACHEDIR $APPTAINER_TMPDIR
setfacl -b $APPTAINER_TMPDIR
```

where `PROJECTID` is your NeSI project ID.

## Building container via Slurm

The new Mahuika Extension nodes can be used to build Apptainer containers using the [fakeroot feature](https://apptainer.org/docs/user/main/fakeroot.html). This functionality is only available on these nodes at the moment due to their operating system version.

To illustrate this functionality, create an example container definition file `my_container.def` from a shell session on NeSI as follows:

```bash
cat << EOF > my_container.def
BootStrap: docker
From: ubuntu:20.04
%post
    apt-get -y update
    apt-get install -y wget
EOF
```

Then, from a Mahuika login node, you can build the container using the `srun` command as follows:

```bash
module load Apptainer
module unload XALT
srun -p milan --time 0-00:30:00 --mem 4GB --cpus-per-task 2 apptainer build --fakeroot my_container.sif my_container.def
```

This command will start an interactive Slurm job for 30 minutes using 2 cores and 4 GB of memory to build the image. Make sure to set these resources correctly, some containers can take hours to build and require tens of GB of memory.

If you need more resources to build your container, please consider submitting your job using a Slurm submission script, for example:

```bash
#!/bin/bash -e
#SBATCH --job-name=apptainer_build
#SBATCH --partition=milan
#SBATCH --time=0-00:30:00
#SBATCH --mem=4GB
#SBATCH --cpus-per-task=2

module load Apptainer
module unload XALT

apptainer build --fakeroot my_container.sif my_container.def
```

# Known limitations

If your container uses RPM to install packages, i.e. is based on CentOS or Rocky Linux, you need to disable the `APPTAINER_TMPDIR` environment variable (use `unset APPTAINER_TMPDIR`) and request more memory for your Slurm job. Otherwise, RPM will crash due to an incompatibility with the `nobackup` filesystem.

> ### Other limitations
> 
> This method, using fakeroot, is known to **not** work for all types of Apptainer/Singularity containers.
> 
<!-- ### Other build options -->

<!-- The def file specification has a number of other interesting features, to know more about them you can visit the [Sylabs docs on def files](https://sylabs.io/guides/3.3/user-guide/definition_files.html).

In the episode on GUI applications we'll see how to use `%startscript` to configure the behaviour of containers running in background.

If you are in a development phase, where you don't know yet what you will include in your final container image, you can start with a *sandbox* image. This is a special type of image designed for development purposes, consisting not of a single file, but instead of a directory. To create one, run something like:

```
sudo singularity build --sandbox playbox/ docker://ubuntu:18.04
```
{: .bash}

Then to open it and play, run:

```
sudo singularity shell --writable playbox/
```
{: .bash}

More information on sandbox images can be found at the [Sylabs docs on building images](https://sylabs.io/guides/3.3/user-guide/build_a_container.html#creating-writable-sandbox-directories).

One last notable feature is the ability to use PGP keys to sign and verify container images. In this way, users of 3rd party containers can double check that the image they're running is bit-by-bit equivalent to the one that the author originally built, largely reducing the possibility to run containers infected by malware. you can find more on this topic at the [Sylabs docs on signing and verifying containers](https://sylabs.io/guides/3.3/user-guide/signNverify.html).  -->

