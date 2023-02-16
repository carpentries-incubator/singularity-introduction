---
title: "Building Container Images"
teaching: 10
exercises: 25
questions:
- "How do I create my own Apptainer images?"
objectives:
- "Understand the different Apptainer container file formats."
- "Understand how to build and share your own Apptainer containers."
keypoints:
- "Apptainer definition files are used to define the build process and configuration for an image."
- "Apptainer's Docker container provides a way to build images on a platform where Apptainer is not installed but Docker is available."
- "Existing images from remote registries such as Docker Hub and Singularity Hub can be used as a base for creating new Apptainer images."
---

So far you've been able to work with {{ site.software.name }} from your own user account as a non-privileged user. This part of the {{ site.software.name }} material requires that you use {{ site.software.name }} in an environment where you have administrative (root) access.

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
    <td>Install {{ site.software.name }} locally on a system where you do have administrative access. And then copy to HPC.</td>
    <td></td>
    <td>Building a contanier locally first is great for testing.</td>
    <td>Not possible for many people. Local machine must have same architecture as HPC. Container image might be quite large and take a long time to copy. </td>
  </tr>
  <tr>
    <td>Build your container from within another container</td>
    <td></td>
    <td>No root access required.</td>
    <td>A bit contrived, requires already having a built container with {{ site.software.name }} installed.</td>
  </tr>
  <tr>
    <td>Use a 'remote build service' to build your container</td>
    <td></td>
    <td></td>
    <td>Requires access to a remote build service. Build image must still be downloaded over network.</td>
  </tr>
  <tr>
    <td>Simulate root access using the <code>--fakeroot</code> feature</td>
    <td></td>
    <td></td>
    <td>Not possible with all operating systems. (On NeSI, only our newer nodes with Rocky8 installed have this functionality.)</td>

  </tr>
</tbody>
</table>

We'll focus on the last option in this part of the course - _Simulate root access using the `--fakeroot` feature_.

## Building container via Slurm

The new Mahuika Extension nodes can be used to build Apptainer containers using the [fakeroot feature](https://apptainer.org/docs/user/main/fakeroot.html). This functionality is only available on these nodes at the moment due to their operating system version.

Since the Mahuika Extension nodes are not directly accessable, you will have to create a slurm script.
Create a new file called `build.sh` and enter the following.

```
#!/bin/bash -e
#SBATCH --job-name=apptainer_build
#SBATCH --partition=milan
#SBATCH --time=0-00:30:00
#SBATCH --mem=4GB
#SBATCH --cpus-per-task=2

module load Apptainer

apptainer build --fakeroot my_container.sif my_container.def
```
{: .language-bash}

Submit your new script with


```
{{ site.machine.prompt }} sbatch build.sh
```
{: .language-bash}


```
Submitted batch job 33031078
```
{: .output}

You can check the status of your job using `squeue`

```
{{ site.machine.prompt }} squeue --me
```
{: .language-bash}

```
JOBID         USER     ACCOUNT   NAME        CPUS MIN_MEM PARTITI START_TIME     TIME_LEFT STATE    NODELIST(REASON)    
33031074      cwal219  nesi99991 spawner-jupy   2      4G interac 2023-02-17T1     7:54:00 RUNNING  wbn003              
33031086      cwal219  nesi99999 apptainer_bu   2      4G milan   2023-02-17T1       29:53 RUNNING  wmc005     
```
{: .output}

Note, the first job shown there is your Jupyter session.

Once the job is finished you should see the built container file.

```
{{ site.machine.prompt }} ls
```
{: .language-bash}

Note the slurm output, if you don't have `my_container.sif` you will want to check here first.

We can test our new container by running.

```
{{ site.machine.prompt }} {{ site.software.cmd }} run my_container.sif
```
{: .language-bash}


```
Hello World!
```
{: .output}

### Known limitations

This method, ( i.e using fakeroot), is known to **not** work for all types of Apptainer/Singularity containers.

If your container uses RPM to install packages, i.e. is based on CentOS or Rocky Linux, you need to disable the `APPTAINER_TMPDIR` environment variable (use `unset APPTAINER_TMPDIR`) and request more memory for your Slurm job. Otherwise, RPM will crash due to an incompatibility with the `nobackup` filesystem.

## Remote build

> ## Apptainer remote builder
>
> Currently there are no freely available remote builders for Apptainer.
{: .callout}

{{ site.software.name }} offers the option to run build remotely, using a **Remote Builder** we will be using the default provided by Sylabs; You will need a Sylabs account and a token to use this feature.

```
{{ site.machine.prompt }} {{ site.software.cmd }} remote login
```
{: .language-bash}

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

With this set up, you may use `{{ site.software.cmd }} build -r` to start the remote build. Once finished, the image will be downloaded so that it's ready to use:

```
{{ site.machine.prompt }} {{ site.software.cmd }} build -r lolcow_remote.sif lolcow.def
```
{: .language-bash}

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

You are now ready to push your image to the Cloud Library, *e.g.* via `{{ site.software.cmd }} push`:

```
{{ site.machine.prompt }} {{ site.software.cmd }} push -U lolcow.sif library://<YOUR-SYLABS-USERNAME>/default/lolcow:30oct19
```
{: .language-bash}

```
WARNING: Skipping container verifying
 67.08 MiB / 67.08 MiB [==================================================================================================================================] 100.00% 6.37 MiB/s 10s
```
{: .output}

Note the use of the flag `-U` to allow pushing unsigned containers (see further down).  
Also note once again the format for the registry: <user>/<user-collection>/<name>:<tag>.

Finally, you (or other peers) are now able to pull your image from the Cloud Library:

```
{{ site.machine.prompt }} {{ site.software.cmd }} pull -U library://<YOUR-SYLABS-USERNAME>/default/lolcow:30oct19
```
{: .language-bash}

```
INFO:    Downloading library image
 67.07 MiB / 67.07 MiB [===================================================================================================================================] 100.00% 8.10 MiB/s 8s
WARNING: Skipping container verification
INFO:    Download complete: lolcow_30oct19.sif
```
{: .output}

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
