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


## Preparing to use {{ site.software.name }} for building images

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
    <td>Install {{ site.software.name }} locally on a system where you do have administrative access.</td>
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

{{ site.software.name }} offers the option to run build remotely, using a **Remote Builder** we will be using the default provided by Sylabs; You will need a Sylabs account and a token to use this feature.

```
{{ site.software.cmd }} remote login
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

With this set up, you may use `{{ site.software.cmd }} build -r` to start the remote build. Once finished, the image will be downloaded so that it's ready to use:

```
{{ site.software.cmd }} build -r lolcow_remote.sif lolcow.def
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

You are now ready to push your image to the Cloud Library, *e.g.* via `{{ site.software.cmd }} push`:

```
{{ site.software.cmd }} push -U lolcow.sif library://<YOUR-SYLABS-USERNAME>/default/lolcow:30oct19
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
{{ site.software.cmd }} pull -U library://<YOUR-SYLABS-USERNAME>/default/lolcow:30oct19
```
{: .bash}

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

