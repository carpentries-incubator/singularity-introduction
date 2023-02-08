---
title: "The Apptainer cache"
teaching: 10
exercises: 0
questions:
- "Why does Apptainer use a local cache?"
- "Where does Apptainer store images?"
objectives:
- "Learn about Apptainer's image cache."
- "Learn how to manage Apptainer images stored locally."
keypoints:
- "Apptainer caches downloaded images so that an unchanged image isn't downloaded again when it is requested using the `apptainer pull` command."
- "You can free up space in the cache by removing all locally cached images or by specifying individual images to remove."
---

## Apptainer's image cache

While Apptainer doesn't have a local image repository in the same way as Docker, it does cache downloaded image files. As we saw in the previous episode, images are simply `.sif` files stored on your local disk.

If you delete a local `.sif` image that you have pulled from a remote image repository and then pull it again, if the image is unchanged from the version you previously pulled, you will be given a copy of the image file from your local cache rather than the image being downloaded again from the remote source. This removes unnecessary network transfers and is particularly useful for large images which may take some time to transfer over the network. To demonstrate this, remove the `lolcow_latest.sif` file stored in your `test` directory and then issue the `pull` command again:

~~~
rm lolcow_latest.sif
apptainer pull library://sylabsed/examples/lolcow
~~~
{: .language-bash}

~~~
INFO:    Use image from cache
~~~
{: .output}

As we can see in the above output, the image has been returned from the cache and we don't see the output that we saw previously showing the image being downloaded from Apptainer Hub.

How do we know what is stored in the local cache? We can find out using the `apptainer cache` command:

~~~
apptainer cache list
~~~
{: .language-bash}

~~~
There are 1 container file(s) using 62.65 MB and 0 oci blob file(s) using 0.00 kB of space
Total space used: 62.65 MB
~~~
{: .output}

This tells us how many container files are stored in the cache and how much disk space the cache is using but it doesn't tell us _what_ is actually being stored. To find out more information we can add the `-v` verbose flag to the `list` command:

~~~
apptainer cache list -v
~~~
{: .language-bash}

~~~
NAME                     DATE CREATED           SIZE             TYPE
lolcow_latest.sif   2020-04-03 13:20:44    62.65 MB         shub

There are 1 container file(s) using 62.65 MB and 0 oci blob file(s) using 0.00 kB of space
Total space used: 62.65 MB
~~~
{: .output}

This provides us with some more useful information about the actual images stored in the cache. In the `TYPE` column we can see that our image type is `shub` because it's a `SIF` image that has been pulled from Apptainer Hub.

> ## Cleaning the Apptainer image cache
> We can remove images from the cache using the `apptainer cache clean` command. Running the command without any options will display a warning and ask you to confirm that you want to remove everything from your cache.
>
> You can also remove specific images or all images of a particular type. Look at the output of `apptainer cache clean --help` for more information.
{: .callout}

> ## Cache location
> By default, Apptainer uses `$HOME/.apptainer/cache` as the location for the cache. You can change the location of the cache by setting the `APPTAINER_CACHEDIR` environment variable to the cache location you want to use.
{: .callout}
