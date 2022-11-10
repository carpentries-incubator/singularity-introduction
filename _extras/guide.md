---
layout: page
title: "Instructor Notes"
---

## Resouces for Instructors

## Workshop Structure

[Instructors, please add notes on your experience with the workshop structure here.]

## Technical tips and tricks

If using an HPC with this workshop, it is recommended that workshop organizers make sure that attendees have access to the HPCs before the workshop

#### Installation

This workshop is designed to be run on either high performance computing clusters (HPCs i.e. clusters that run SLURM, SGE, or other job scheduling software), or on the learners computer. However, due to a large part of Singularity containers being used on an HPC it is recommended that HPC be used in the teaching of this workshop. In such a case, if Singularity is not already on the cluster, admin rights would be required to install Singularity cluster-wide, practically meaning that a support ticket would be needed for most clusters. 

## Common problems

Some installs of Singularity require the use of `--bind` for compatibility between the container and the host system, if the host system does not have a directory that is a default or required directory in the container that directory will need to be bound elsewhere in order to work correctly: (i.e. `--bind /data:/mnt`)



