---
title: "Files in Singularity containers"
teaching: 10
exercises: 10
questions:
- "How do I make data available in a Singularity container?"
- "What data is made available by default in a Singularity container?"
objectives:
- "Understand that some data from the host system is usually made available by default within a container"
- "Learn more about how Singularity handles users and binds directories from the host filesystem."
keypoints:
- "Your current directory and home directory are usually available by default in a container."
- "You have the same username and permissions in a container as on the host system."
- "You can specify additional host system directories to be available in the container."
---


## Users within a Singularity container

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

## Files and directories within a Singularity container

Singularity also _binds_ some directories from the host system where you are running the `singularity` command into the container that you're starting. Note that this bind process is not copying files into the running container, it is making an existing directory on the host system visible and accessible within the container environment. If you write files to this directory within the running container, when the container shuts down, those changes will persist in the relevant location on the host system.

There is a default configuration of which files and directories are bound into the container but ultimate control of how things are set up on the system where you're running Singularity is determined by the system administrator. As a result, this section provides an overview but you may find that things are a little different on the system that you're running on.

Two directories that are likely to be accessible within a container that you start are your _home directory_ and the directory from which you issued the `singularity` command.

The mapping of file content and directories from a host system into a Singularity container is illustrated in the example below showing a subset of the directories on the host Linux system and in a Singularity container:

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

## References

\[1\] Gregory M. Kurzer, Containers for Science, Reproducibility and Mobility: Singularity P2. Intel HPC Developer Conference, 2017. Available at: https://www.intel.com/content/dam/www/public/us/en/documents/presentation/hpc-containers-singularity-advanced.pdf
