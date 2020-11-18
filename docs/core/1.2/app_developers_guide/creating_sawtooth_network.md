# **Creating a Sawtooth Test Network** #

This section describes how to create a Sawtooth network for an application development environment. Each node is similar to the single-node environment described [earlier in this guide](http://172.26.96.1:4000/docs/core/1.2/app_developers_guide/installing_sawtooth.html), but the network uses a consensus algorithm that is appropriate for a Sawtooth network (instead of Devmode consensus). For more information, see [About Sawtooth Networks](http://172.26.96.1:4000/docs/core/1.2/app_developers_guide/about_sawtooth_networks.html).

Note

For a single-node test environment, see [Setting Up a Sawtooth Node for Testing](http://172.26.96.1:4000/docs/core/1.2/app_developers_guide/installing_sawtooth.html).

Use the procedures in this section to create a Sawtooth test network using prebuilt [Docker](https://www.docker.com/) containers, a [Kubernetes](https://kubernetes.io/) cluster inside a virtual machine on your computer, or a native [Ubuntu](https://ubuntu.com/) installation.

To get started, read [About Sawtooth Networks](http://172.26.96.1:4000/docs/core/1.2/app_developers_guide/about_sawtooth_networks.html), then choose the guide for the platform of your choice.

```
.. toctree::
   :maxdepth: 2

   about_sawtooth_networks
   docker_test_network
   kubernetes_test_network
   ubuntu_test_network
```
