Introduction
------------

This document provides guidelines and some suggestions on how to add
support for a machine not already supported by FoundriesFactory. The
list of supported machines can be found at :ref:`ref-linux-supported`.

Needing to port a new board could be due to a new project bringing new
hardware to their FoundriesFactory, an existing project that is migrating or adding new hardware.

Another possible reason is to customize hardware aspects of well known
hardware, but with another machine file.

The strategy shown on this document is to use a FoundriesFactory to add
support for a new machine on top of the existing structure provided by
the Linux micro Platform (LmP) which provides the needed configuration for the packages,
images and classes. Also provided is the container runtime and OTA
infrastructure integration.

As the LmP uses the Yocto Project tools, the
packages and configuration files following the Yocto
Project concepts which are used in this guide. Some knowledge of the
Yocto Project and embedded Linux development are required.

A `support request <https://foundriesio.atlassian.net/servicedesk/customer/portals>`_ can be submitted in
case help is needed.

In the next section there is a description on how to find a similar
reference board to base the porting work. After that, there is a list of
what is needed before starting and how to plan the porting. The next
part is the list of the needed packages with needed configuration and
suggestions on how to proceed with each one. At the end, a checklist is
provided to help ensure all required steps have been taken.
