---
title: "Using containers to run commands"
teaching: 10
exercises: 5
questions:
- "How do I use container software on the cluster?"
- "How do I run different commands within a container?"
- "How do I access an interactive shell within a container?"
objectives:
- "Learn how to run different commands when starting a container."
- "Learn how to open an interactive shell within a container environment."
keypoints:
- "The `apptainer exec` is an alternative to `apptainer run` that allows you to start a container running a specific command."
- "The `apptainer shell` command can be used to start a container and run an interactive shell within it."
---

## Pulling a new image and running a container

We will be working from the training project directory `{{ site.machine.working_dir }}`.

```
{{ site.machine.prompt }} cd {{ site.machine.working_dir }}
```
{: .language-bash}


Let's begin by creating a directory with *your username*, changing into it and *pulling* a test *Hello World* image from a Docker image repository:

```
{{ site.machine.prompt }} mkdir {{ site.machine.working_dir }}/$USER
{{ site.machine.prompt }} cd {{ site.machine.working_dir }}/$USER
{{ site.machine.prompt }} {{ site.software.cmd }} pull docker://ghcr.io/apptainer/lolcow
```
{: .language-bash}

```
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
```
{: .output}

We pulled a Docker image from a Docker image repo using the `{{ site.software.cmd }} pull` command and directed it to store the image file using the default name `lolcow_latest.sif` in the current directory. If you run the `ls` command, you should see that the `lolcow_latest.sif` file is now present in the current directory. This is our image and we can now run a container based on this image:

```
{{ site.machine.prompt }} {{ site.software.cmd }} run lolcow_latest.sif
```
{: .language-bash}

```
 _________________________
< Wed Feb 8 23:36:16 2023 >
 -------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
{: .output}

Most images are also directly executable

```
{{ site.machine.prompt }} ./lolcow_latest.sif
```
{: .language-bash}

```
 _________________________
< Wed Feb 8 23:36:36 2023 >
 -------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
{: .output}

How did the container determine what to do when we ran it?! What did running the container actually do to result in the displayed output?

When you run a container from a sif image without using any additional command line arguments, the container runs the default run script that is embedded within the image. This is a shell script that can be used to run commands, tools or applications stored within the image on container startup. We can inspect the image's run script using the `{{ site.software.cmd }} inspect` command:

```
{{ site.machine.prompt }} {{ site.software.cmd }} inspect -r lolcow_latest.sif | head
```
{: .language-bash}

```
#!/bin/sh
OCI_ENTRYPOINT='"/bin/sh" "-c" "date | cowsay | lolcat"'
OCI_CMD=''

# When SINGULARITY_NO_EVAL set, use OCI compatible behavior that does
# not evaluate resolved CMD / ENTRYPOINT / ARGS through the shell, and
# does not modify expected quoting behavior of args.
if [ -n "$SINGULARITY_NO_EVAL" ]; then
    # ENTRYPOINT only - run entrypoint plus args
    if [ -z "$OCI_CMD" ] && [ -n "$OCI_ENTRYPOINT" ]; then
```
{: .output}

This shows us the first 10 lines of the script within the `lolcow_latest.sif` image configured to run by default when we use the `{{ site.software.cmd }} run` command.

## Running specific commands within a container

We saw earlier that we can use the `{{ site.software.cmd }} inspect` command to see the run script that a container is configured to run by default. What if we want to run a different command within a container?

If we know the path of an executable that we want to run within a container, we can use the `{{ site.software.cmd }} exec` command. For example, using the `lolcow_latest.sif` container that we've already pulled from Singularity Hub, we can run the following within the `test` directory where the `lolcow_latest.sif` file is located:

```
{{ site.machine.prompt }} {{ site.software.cmd }} exec lolcow_latest.sif echo Hello World!
```
{: .language-bash}

```
Hello World!
```
{: .output}

Here we see that a container has been started from the `lolcow_latest.sif` image and the `echo` command has been run within the container, passing the input `Hello World!`. The command has echoed the provided input to the console and the container has terminated.

Note that the use of `{{ site.software.cmd }} exec` has overriden any run script set within the image metadata and the command that we specified as an argument to `{{ site.software.cmd }} exec` has been run instead.

> ## Basic exercise: Running a different command within the "lolcow" container
>
> Can you run a container based on the `lolcow_latest.sif` image that **prints the current date and time**?
>
> > ## Solution
> >
> > ```
> > {{ site.machine.prompt }} {{ site.software.cmd }} exec lolcow_latest.sif date
> > ```
> > {: .language-bash}
> > 
> > ```
> > Mon Dec 12 03:43:31  2022
> > ```
> > {: .output}
> {: .solution}
{: .challenge}

### **The difference between `{{ site.software.cmd }} run` and `{{ site.software.cmd }} exec`**

Above we used the `{{ site.software.cmd }} exec` command. In earlier episodes of this
course we used `{{ site.software.cmd }} run`. To clarify, the difference between these
two commands is:

- `{{ site.software.cmd }} run`: This will run the default command set for containers
   based on the specfied image. This default command is set within
   the image metadata when the image is built (we'll see more about this
   in later episodes). You do not specify a command to run when using
   `{{ site.software.cmd }} run`, you simply specify the image file name. As we saw 
   earlier, you can use the `{{ site.software.cmd }} inspect` command to see what command
   is run by default when starting a new container based on an image.

- `{{ site.software.cmd }} exec`: This will start a container based on the specified
   image and run the command provided on the command line following
   `{{ site.software.cmd }} exec <image file name>`. This will override any default
   command specified within the image metadata that would otherwise be
   run if you used `{{ site.software.cmd }} run`.

## Opening an interactive shell within a container

If you want to open an interactive shell within a container, Singularity provides the `{{ site.software.cmd }} shell` command. Again, using the `lolcow_latest.sif` image, and within our `test` directory, we can run a shell within a container from the hello-world image:

```
{{ site.machine.prompt }} {{ site.software.cmd }} shell lolcow_latest.sif
```
{: .language-bash}

```
{{ site.software.prompt }} whoami
```
{: .language-bash}

```
<your username>
```
{: .output}

```
{{ site.software.prompt }} ls
```
{: .language-bash}

```
lolcow_latest.sif
```
{: .output}

As shown above, we have opened a shell in a new container started from the `lolcow_latest.sif` image. Note that the shell prompt has changed to show we are now within the Singularity container.

> ## Discussion: Running a shell inside a Singularity container
>
> Q: What do you notice about the output of the above commands entered within the Singularity container shell?
>
> Q: Does this differ from what you might see within a Docker container?
{: .discussion}

Use the `exit` command to exit from the container shell.
