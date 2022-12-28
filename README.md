# ocp-virtualization-workshop

Welcome to the OpenShift Virtualization. We've put this together to give you an overview and hands-on technical deep dive into how OpenShift Virtualization works, and how it continues to bring advanced virtualization capabilities to OpenShift.

OpenShift Virtualization is the product name for the Container-native Virtualization operator for OpenShift. This is often referred to as "CNV" and is the downstream offering for the upstream [Kubevirt project](https://kubevirt.io/). At the same time, some aspects of this lab have references to "CNV", any reference to "CNV", "Container-native Virtualization" and "OpenShift Virtualization" can be somewhat used interchangeably.

In these labs, you'll utilize a virtual environment. In this hands-on lab, you won't need to deploy OpenShift, it's already deployed for you! However, aside from this lab guide, it's a completely empty cluster, ready to be configured and used with OpenShift Virtualization.

## Topics

- An overview of this Lab
- Prerequisites
- Exploring the OpenShift CLI and Web Console
- Validating the Lab environment
- Setting up storage for OpenShift Virtualization
- Deploying workloads on OpenShift Virtualization
- Performing Live Migration and Node Maintenance
- Cloning a Virtual Machine
- An overview of OpenShift console
- Exploring virtual machine templates and boot sources
- Deploying a DNS application base Galera DB
- Performing Backup and Restore of Virtual Machines
- Add a hot-plug disk to Virtual Machines

## To do the exercises, follow the steps presented in the workshop’s [presentation](https://docs.google.com/presentation/d/12zjIQTX8l54CMMRRsCl7ZlTsnx5PrnUqaFtQ9qP73Aw/edit#slide=id.gb710eccc1a_0_2256).

~~~
NOTE: Variables in this workshop will be presented this way: < variable >
For example: oc project < your name > --> oc project almog
~~~
