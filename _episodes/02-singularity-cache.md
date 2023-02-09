---
title: "The Apptainer cache"
teaching: 10
exercises: 0
questions:
- "Why does Apptainer use a local cache?"
- "Where does Apptainer store images?"
objectives:
- "Learn about Apptainer's image cache."
keypoints:
- "Apptainer caches downloaded images so that an unchanged image isn't downloaded again when it is requested using the `apptainer pull` command."
- "You can free up space in the cache by removing all locally cached images or by specifying individual images to remove."
---

## Apptainer's image cache

While Apptainer doesn't have a local image repository in the same way as Docker, it does cache downloaded image files. As we saw in the previous episode, images are simply `.sif` files stored on your local disk.

If you delete a local `.sif` image that you have pulled from a remote image repository and then pull it again, if the image is unchanged from the version you previously pulled, you will be given a copy of the image file from your local cache rather than the image being downloaded again from the remote source. This removes unnecessary network transfers and is particularly useful for large images which may take some time to transfer over the network. To demonstrate this, remove the `lolcow_latest.sif` file stored in your `test` directory and then issue the `pull` command again:

~~~
rm lolcow_latest.sif
apptainer pull docker://ghcr.io/apptainer/lolcow
~~~
{: .language-bash}

~~~
INFO:    Using cached SIF image
~~~
{: .output}

As we can see in the above output, the image has been returned from the cache and we don't see the output that we saw previously showing the image being downloaded from Apptainer Hub.

## Cleaning the Apptainer image cache
We can remove images from the cache using the `apptainer cache clean` command. Running the command without any options will display a warning and ask you to confirm that you want to remove everything from your cache.  This is very useful if you are running low on space or do not want to keep old images on disk.
>
> You can also remove specific images or all images of a particular type. Look at the output of `apptainer cache clean --help` for more information.
{: .callout}

## Cache location
 ## Cache  and temp file location
 By default, Apptainer uses `$HOME/.apptainer` as the location for cache and temporary files. However, on NeSI, our home directories are quite small, so we need to move these to a more appropriate location such as our nobackup storage.

You can change the location of the cache by setting the `APPTAINER_CACHEDIR` environment variable to the cache location you want to use.
{: .callout}
