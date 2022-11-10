So far we've deployed some basic workloads and shown you the main interfaces for OpenShift Virtualization, but now we're going to move into a more realistic "real-world" scenario. This lab section introduces you to the architecture of the **DNS** application used throughout this workshop, to get a better understanding of the things you'll be doing from a *developer* perspective. We will use OpenShift virtualization to run VMs with MariaDB and PowerDNS. The following diagram shows the environment setup.

The setup uses five VMs. Two VMs (mariadb-0 and mariadb-1) will host the Glare cluster, and two separate VMs (power-dns-0 and powser-dns-1) will act as DNS servers. The fifth VM is a workstation., roughly resembling the following architecture:

<img width="674" alt="Screen Shot 2022-11-10 at 14 20 25" src="https://user-images.githubusercontent.com/64369864/201090138-e05e6152-f45f-441e-8c49-5a76767d9749.png">


The main service is a DNS application which has a MongoDB. In order to showcase how virtual machines can run with High Availability in an OpenShift Environment.
