---
layout: lesson
root: .  # Is the only page that doesn't follow the pattern /:path/index.html
permalink: index.html  # Is the only page that doesn't follow the pattern /:path/index.html
---

This lesson provides an introduction to using the [{{site.software.name}} container platform]({{site.software.url}}). {{site.software.name}} is particularly suited to running containers on infrastructure where users don't have administrative privileges, for example shared infrastructure such as High Performance Computing (HPC) clusters. 

This lesson will introduce Apptainer from scratch showing you how to run a simple container and building up to creating your own containers and running parallel scientific workloads on HPC infrastructure.

<!-- this is an html comment -->

{% comment %} This is a comment in Liquid {% endcomment %}

> ## Prerequisites
>
> You will need a NeSI account, and be a member of an active project.
> See the [setup page]({{ site.url }}/setup) for more information on how to use the NeSI Jupyter service.
>
> For further developing containers you may wish to have access to a local or remote Linux-based system on which you have administrator (root) access > and can install the {{site.software.name}} software.
{: .prereq}

{% include links.md %}
