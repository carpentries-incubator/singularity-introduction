---
layout: page
title: "Instructor Notes"
---

## Resouces for Instructors

## Workshop Structure

_[Instructors, please add notes here reporting on your experiences of teaching this module, either standalone, or as part of a wider workshop.]_

 - **Containers course covering Docker and Singularity:** This Singularity module is regularly taught alongside the [Introduction to Docker](https://github.com/carpentries-incubator/docker-introduction) module as part of a 2-day course "_Reproducible computational environments using containers_" run through the [ARCHER2 training programme](https://www.archer2.ac.uk/training/) in the UK. [See an example](https://www.archer2.ac.uk/training/courses/221207-containers/) of this course run in December 2022.

  This course has been run both online and in person. Experience suggests that this Singularity module requires between 5 and 6 hours of time to run effectively. The main aspect that takes a significant amount of time is the material at the end of the module looking at building a Singularity image containing an MPI code and then running this in parallel on an HPC platform. The variation in timing depends on how much experience the learners already have with running parallel jobs on HPC platforms and how much they wish to go into the details of the processes of running parallel jobs. For some groups of learners, the MPI use case is not something that is relevant and they may request to cover this material without the section on running parallel MPI jobs. In this case, the material can comfortably be compelted in 4 hours.



## Technical tips and tricks

 - **HPC access:** Many learners will be keen to learn Singularity so that they can make use of it on a remote High Performance Computing (HPC) cluster. It is therefore strongly recommended that workshop organizers provide course attendees with access to an HPC platform that has Singularity pre-installed for undertaking this module. Where necessary, it is also recommended that guest accounts are set up and learners are asked to test access to the platform before the workshop.

 - **Use of the Singularity Docker container:** Singularity is a Linux tool. The optimal approach to building Singularity images, a key aspect of the material in this module, requires that the learner have a platform with Singularity installed, on which they have admin/root access. Since it's likely that many learners undertaking this module will not be using Linux as the main operating system on the computer on which they are undertaking the training, we need an alternative option. To address this, we use Docker to run the [Singularity Docker container](https://quay.io/repository/singularity/singularity). This ensures that learners have access to a local Singularity deployment (running within a Docker container) on which they have root access and can build Singularity images. The layers of indirection that this requires can prove confusing and we are looking at alternatives but from experience of teaching the module so far, this has proved to be the most reasonable solution at present.

#### Pre-workshop Planning / Installation / Setup

As highlighted above, this module is designed to support learners who wish to use Singularity on an HPC cluster. Elements of the module are therefore designed to be run on a High Performance Computing cluster (i.e. clusters that run SLURM, SGE, or other job scheduling software). It is possible for learners to undertake large parts of the module on their own computer. However, since a key use case for Singularity containers is their use on HPC infrastructure, it is recommended that an HPC platform be used in the teaching of this module. In such a case, if Singularity is not already on the cluster, admin rights would be required to install Singularity cluster-wide. Practically, it is likely that this will require a support ticket to be raised with cluster administrators and may require some time for them to investigate the software if they are unfamiliar with Singularity.

## Common problems

Some installs of Singularity require the use of `--bind` for compatibility between the container and the host system, if the host system does not have a directory that is a default or required directory in the container that directory will need to be bound elsewhere in order to work correctly: (i.e. `--bind /data:/mnt`)



