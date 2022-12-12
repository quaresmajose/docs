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
sucessfully. In the public LmP layer ``meta-lmp`` we use the
``meta-lmp-base/conf/distro/include/non-clangable.inc``
for that propuse.

Adding the same on the factory layer ``meta-subscriber-overrides`` can be a solution
where we will collect all the ``clang`` tweeks needed. This configuration
can be added in ``conf/machine/include/lmp-factory-custom.inc``
or somewere in another global configurations file.

.. prompt:: text

  require conf/machine/include/non-clangable.inc

So we will use the factory file ``conf/machine/include/non-clangable.inc``
to customize everything clang related.


Changing the toolchain
----------------------

To change the default values disabling the ``clang`` compiler globaly and use the ``gcc``
we need to change the ``TOOLCHAIN`` value.

.. prompt:: text

  TOOLCHAIN = "gcc"

Changing the compiler per recipe is also possible and it can be done using a bbappend or
with ``TOOLCHAIN = "gcc"`` or in the global factory ``non-clangable.inc``

.. prompt:: text

  TOOLCHAIN:pn-<recipe> = "gcc"


Costumize the default toolchain
-------------------------------

When using the clang toolchain we have the ``toolchain-clang`` override that can be used
to do the costumization and this will only take effect when clang is in use.
All of this costumizations can be placed on the factory ``non-clangable.inc``

.. prompt:: text

  STRIP:pn-<recipe>:toolchain-clang = "${HOST_PREFIX}strip"
  OBJCOPY:pn-<recipe>:toolchain-clang = "${HOST_PREFIX}objcopy"
  COMPILER_RT:pn-<recipe>:toolchain-clang = "-rtlib=libgcc --unwindlib=libgcc"


Some more infiornation can be found on https://github.com/kraj/meta-clang
