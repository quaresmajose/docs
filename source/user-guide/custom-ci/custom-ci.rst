.. _ug-custom-ci:

Custom CI
=========

FoundriesFactory is like a swiss army knife that includes everything you need to manage lifecycle of
a Linux-based operating system and containerized applications running on it. In particular, it includes:

1. Git repositories for your operating system (OE layers/recipes) and containerized applications.
2. The CI service to build your operating system and containerized applications.
3. The TUF compliant OTA service to securely update the OS and the Apps on your devices.
4. Utilities to configure and manage the aforementioned services.

While it all works together like a charm, in some cases you may wanna use just some subsets of the provided services.
In particular, you may want to build OS and applications by yourself and still use the Foundries OTA service.
The following two guides walk you through the steps to accomplish it.

.. toctree::
   :maxdepth: 1

   custom-ci-for-rootfs
   custom-ci-for-apps
