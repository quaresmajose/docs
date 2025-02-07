Creating the U-Boot configuration fragments (``lmp-base.cfg``, ``lmp.cfg``)
===========================================================================

The ``meta-lmp`` layer uses ``.cfg`` configuration fragments to append and override
configurations in the board ``defconfig`` file.

``lmp-base.cfg`` is the configuration used for ``DISTRO=lmp-base`` and ``lmp.cfg``
is the one for ``DISTRO=lmp``.

The ``.cfg`` file enables the features required for SPL and U-Boot proper to
support LmP, such as booting SPL, verifying the signature of the
consequent loaded images, enabling eMMC secure storage, and custom boot
command.

To create this file use the reference board support in LmP and modify
the configuration files as needed. The list of supported boards and
their ``.cfg`` files can be found `here <https://github.com/foundriesio/meta-lmp/tree/master/meta-lmp-bsp/recipes-bsp/u-boot/u-boot-fio>`_.

For example, see ``imx8mmevk/lmp-base.cfg``:

.. prompt:: text

    CONFIG_SPL_DM=y
    CONFIG_SPL_OF_CONTROL=y
    CONFIG_SPL_FIT=y
    CONFIG_SPL_LOAD_FIT=y
    # CONFIG_SPL_FIT_IMAGE_TINY is not set
    # CONFIG_CMD_DEKBLOB is not set
    # CONFIG_SPL_DM_USB is not set
    CONFIG_OF_LIBFDT_OVERLAY=y
    CONFIG_FIT=y
    CONFIG_FIT_SIGNATURE=y
    CONFIG_FIT_VERBOSE=y
    CONFIG_LEGACY_IMAGE_FORMAT=y
    CONFIG_PARTITION_UUIDS=y
    CONFIG_CMD_XIMG=y
    CONFIG_SUPPORT_EMMC_RPMB=y
    CONFIG_SUPPORT_EMMC_BOOT=y
    # CONFIG_ENV_IS_IN_MMC is not set
    CONFIG_ENV_IS_IN_FAT=y
    CONFIG_ENV_FAT_INTERFACE="mmc"
    CONFIG_ENV_FAT_DEVICE_AND_PART="2:1"
    CONFIG_ENV_SIZE=0x4000
    CONFIG_CMD_IMPORTENV=y
    CONFIG_CMD_EDITENV=y
    CONFIG_CMD_SAVEENV=y
    CONFIG_USE_BOOTCOMMAND=y
    CONFIG_BOOTCOMMAND="fatload mmc ${emmc_dev}:1 ${loadaddr} /boot.scr;
    source ${loadaddr}; reset"

Most of the configuration is generic to all boards. The user needs to
pay attention to board-specific configuration, in this case:

.. prompt:: text

  CONFIG_ENV_FAT_DEVICE_AND_PART="2:1"

Where ``2`` represents the uSDHC index for the eMMC device in imx8mmevk.
The user should align this with the expected for their board.

.. note::

   It is recommended to set an eMMC device as the boot target if applicable.
   In i.MX targets, ``lmp-base`` distro does not support SD card boot at this time.
