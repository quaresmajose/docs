.. _ref-toolchain:

Toolchain
=========

The toolchain can be configured globaly or per recipe, since version v89
LmP uses by default ``clang`` as the compiler and ``llvm`` as the runtime.

.. prompt:: text

  TOOLCHAIN ?= "clang"
  RUNTIME = "llvm"

Exists however some issues when using the ``clang`` and ``llvm`` as default,
because some recipes are refusing to be build with this configuration. To adresss this issues
is recomendend to add a new file where we will add the tweeks needed to build
sucessfully. In the public LmP we use the ``meta-lmp/meta-lmp-base/conf/distro/include/non-clangable.inc``
for that propuse.

Adding the same on the ``meta-subscriber-overrides`` can be a solution
where we collect all the ``clang`` tweeks needed. This configuration
can be added in ``meta-subscriber-overrides/conf/layer.conf`` or somewere in another global
configurations file.

.. prompt:: text

  require meta-subscriber-overrides/conf/non-clangable.inc


Changing the toolchain
----------------------

To change the default values disabling the ``clang`` compiler globaly and use the ``gcc``
we need to change the ``TOOLCHAIN`` value in ``meta-subscriber-overrides/conf/layer.conf``

.. prompt:: text

  TOOLCHAIN = "gcc"

Changing the compiler per recipe is also possible and it can be done using a bbappend
with ``TOOLCHAIN = "gcc"`` or using the ``meta-subscriber-overrides/conf/non-clangable.inc``

.. prompt:: text

  TOOLCHAIN:pn-<recipe> = "gcc"


Costumize the default toolchain
-------------------------------

When using the clang toolchain we have the ``toolchain-clang`` override that can be used
to do the costumization and this will only take effect when clang is in use.
All of this costumizations can be placed on ``meta-subscriber-overrides/conf/non-clangable.inc``

.. prompt:: text

  STRIP:pn-<recipe>:toolchain-clang = "${HOST_PREFIX}strip"
  OBJCOPY:pn-<recipe>:toolchain-clang = "${HOST_PREFIX}objcopy"
  COMPILER_RT:pn-<recipe>:toolchain-clang = "-rtlib=libgcc --unwindlib=libgcc"


Some more infiornation can be found on https://github.com/kraj/meta-clang
