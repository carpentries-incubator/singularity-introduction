---
title: "The Container Cache"
teaching: 10
exercises: 0
questions:
- "Why does Apptainer use a local cache?"
- "Where does Apptainer store images?"
- "How do I configure my cache to work on NeSI?"
objectives:
- "Learn about Apptainer's image cache."
- "Learn howto setup your cache on Mahuika"
keypoints:
- "Apptainer caches downloaded images so that an unchanged image isn't downloaded again when it is requested using the `apptainer pull` command."
- "You can free up space in the cache by removing all locally cached images or by specifying individual images to remove."
- "Cache location and configuration requirements on Mahuika cluster"
---

## {{ site.software.name }}'s image cache and temporary files

{{ site.software.name }} doesn't have a local image repository in the same way as Docker, however, it does cache downloaded image files. {{ site.software.name }} also uses a temporary directory for building images.

By default, {{ site.software.name }} uses `$HOME/.{{ site.software.name }}` as the location for cache and temporary files. However, on NeSI, our home directories are quite small, so we need to move these to a more appropriate location such as our nobackup storage.

You can change the location of the cache by setting environment variables to the cache and temporary directory locations you want to use.  Those environment variables are:
`{{ site.software.name | upcase }}_CACHEDIR` & `{{ site.software.name | upcase }}_TMPDIR`

We will now setup our {{ site.software.name }} environment for use on NeSI.

## Create a cache and temporary directory for use on NeSI

Due to our backend high-performance filesystem, special handling of your cache and temporary directories for building and storing container images is required.  What we will do in the following exercise is create a temporary and cache directory, reconfigure the permissions on those directories and then declare special environment variables that will tell {{ site.software.name }} where it should store files and images.

```
{{ site.machine.prompt }} export {{ site.software.name | upcase }}_CACHEDIR={{ site.machine.working_dir }}/$USER/{{ site.software.cmd }}_cache
{{ site.machine.prompt }} export {{ site.software.name | upcase }}_TMPDIR={{ site.machine.working_dir }}/$USER/{{ site.software.cmd }}_tmp
{{ site.machine.prompt }} mkdir -p ${{ site.software.name | upcase }}_CACHEDIR ${{ site.software.name | upcase }}_TMPDIR
{{ site.machine.prompt }} setfacl -b ${{ site.software.name | upcase }}_TMPDIR
{{ site.machine.prompt }} ls -l /nesi/nobackup/nesi99991/20230217_nzrse/$USER
```
{: .language-bash}

```
total 1
drwxrws---+ 2 user001 nesi99991 4096 Feb 10 13:42 apptainer_cache
drwxrws---  2 user001 nesi99991 4096 Feb 10 13:42 apptainer_tmp
```
{: .output}

## Testing that {{ site.software.name }} will run on the NeSI Mahuika cluster

### Loading the module

Before you can use the `{{ site.software.cmd }}` command on the system, you must load the latest module.

```
{{ site.machine.prompt }} module purge
{{ site.machine.prompt }} module load {{ site.software.module }}
```
{: .language-bash}

### Showing the version

```
{{ site.machine.prompt }} {{ site.software.cmd }} --version
```
{: .language-bash}

```
{{ site.software.cmd }} {{ site.software.version }}
```
{: .output}

Depending on the version of {{ site.software.name }} installed on your system, you may see a different version. At the time of writing, `{{ site.software.version }}` is the latest release of {{ site.software.name }}.

## Using the image cache and temporary directories

Let's pull an Ubuntu Linux image from DockerHub:

```
{{ site.machine.prompt }} {{ site.software.cmd }} pull docker://ubuntu
```
{: .language-bash}

```
INFO:    Converting OCI blobs to SIF format
INFO:    Starting build...
Getting image source signatures
Copying blob 677076032cca done
Copying config 58db3edaf2 done
Writing manifest to image destination
Storing signatures
2023/02/10 14:05:20  info unpack layer: sha256:677076032cca0a2362d25cf3660072e738d1b96fe860409a33ce901d695d7ee8
INFO:    Creating SIF file...
```
{: .output}

So what we did here was to use the `docker://` URL to tell {{ site.software.cmd }} to go to DockerHub and pull the Ubuntu Docker image.  {{ site.software.name }} pulls the image and converts it into the image file format used by {{ site.software.name }} and Singularity: `.sif`.  The image file is save in our current directory as `ubuntu_latest.sif` and a cached copy is in our `${{ site.software.name | upcase }}_CACHEDIR`

If you delete the `.sif` image that you have pulled from a remote image repository such as DockerHub, and then pull it again, provided the image is unchanged from the version you previously pulled, you will be given a copy of the image file from your local cache rather than the image being downloaded again from the remote source. This removes unnecessary network transfers and is particularly useful for large images which may take some time to transfer over the network. To demonstrate this, remove the `ubuntu_latest.sif` file stored in your directory and then issue the `pull` command again:

```
{{ site.machine.prompt }} rm ubuntu_latest.sif
{{ site.machine.prompt }} {{ site.software.cmd }} pull docker://ubuntu
```
{: .language-bash}

```
INFO:    Using cached SIF image
```
{: .output}

As we can see in the above output, the image has been returned from the cache and we don't see the output that we saw previously showing the image being downloaded and converted from Docker Hub.

## Cleaning the {{ site.software.name }} image cache
We can remove images from the cache using the `{{ site.software.cmd }} cache clean` command. Running the command without any options will display a warning and ask you to confirm that you want to remove everything from your cache.  This is very useful if you are running low on space or do not want to keep old images on disk.
>
> You can also remove specific images or all images of a particular type. Look at the output of `{{ site.software.cmd }} cache clean --help` for more information.
{: .callout}


## Setup {{ site.software.name }} for your project
When you want to setup an {{ site.software.name }} environment for your own project, you can replace the `{{ site.machine.working_dir }}` path with your project nobackup path. Once done you can also add the environment variables to your personal configuration files, eg.
```
echo "export {{ site.software.name | upcase }}_CACHEDIR=/nesi/nobackup/PROJECTID/{{ site.software.cmd }}_cache" >> $HOME/.bashrc
echo "export {{ site.software.name | upcase }}_TMPDIR=/nesi/nobackup/PROJECTID/{{ site.software.cmd }}_tmp" >> $HOME/.bashrc
```

The `.bashrc` file is read each time you login, ensuring your {{ site.software.name }} environment variables are set.
