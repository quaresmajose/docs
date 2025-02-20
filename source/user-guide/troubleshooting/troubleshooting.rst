.. _ref-troubleshooting:

Troubleshooting and FAQ
=======================

This page covers a variety of topics falling under addressing specific :ref:`errors <ref-ts-errors>`, more :ref:`general how to <ref-ts-howto>`, and then :ref:`tips/about <ref-ts-tips>`.

.. _ref-ts-errors:

Errors and Solutions
---------------------

Aktualizr-Lite Common Reports
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This section shows some common returns from ``aktualizr-lite`` operations.

.. tip::
   To get the ``aktualizr-lite`` logs, run::

      sudo journalctl -fu aktualizr-lite

* **curl error 3**

.. code-block::

   curl error 3 (http code 0): URL using bad/illegal format or missing URL

This could mean that the device is not properly registered, but there is a faulty ``/var/sota/sql.db`` file available. This file may be created when ``aktualizr-lite`` is manually run before registering the device.

**Solution:** Re-flash or :ref:`ref-ts-re-register`. Make sure the device is properly registered before running ``aktualizr-lite``.

* **curl error 6 or 56**

.. code-block::

   curl error 6 (http code 0): Couldn't resolve host name

Or:

.. code-block::

   curl error 56 (http code 0): Failure when receiving data from the peer

These could mean that there is no networking available and/or the device cannot talk to the device gateway. This could be due to a broken registration or a faulty ``/var/sota/sql.db`` file. These can also mean that the DNS is not working correctly (for instance, IPv6 only).

**Solution:** Re-flash or :ref:`ref-ts-re-register`. If you are :ref:`Setting up your Device Gateway PKI <ref-rm-pki>`, make sure all operations have succeeded.

* **curl error 7 or 28**

.. code-block::

   curl error 7 (http code 0): Couldn't connect to server

Or:

.. code-block::

   curl error 28 (http code 0): Timeout was reached

These could mean that a device cannot reach the server.

**Solution:** Make sure your device has a good connection. Check for proxies or firewalls in the network. If you are :ref:`Setting up your Device Gateway PKI <ref-rm-pki>`, make sure all operations have succeeded.

.. tip::
   The `openssl s_client <https://www.openssl.org/docs/man1.0.2/man1/openssl-s_client.html>`_ command can be very useful for troubleshooting network issues. For example::

       openssl s_client -connect <dg>:8443 -cert client.pem -key pkey.pem -CAfile root.crt

   Where:

   * ``<dg>``: Device gateway address, defaults to ``ota-lite.foundries.io``. The actual address can be found in ``/var/sota/sota.toml``, ``[tls].server`` field.


* **Failed to update Image repo metadata**

.. code-block::

   Failed to update Image repo metadata: The root metadata was expired.

This means your TUF root key has expired.

**Solution:** Rotate your :ref:`ref-offline-keys`.

.. code-block::

   Failed to update Image repo metadata: The timestamp metadata was expired.

This means the Target to update to has expired.

.. tip::
   The Target metadata freshness can be checked on the host with::

      curl -H "osf-token: <token>" "https://api.foundries.io/ota/repo/<factory>/api/v1/user_repo/timestamp.json?tag=<tag>[&production=1]" | jq ."signed"."expires"

   Where:

   * ``<tag>``: Device tag.
   * ``<token>``: API Token with ``targets:read`` scope.
   * ``<factory>``: Factory name.

**Solution:** Create a new Target for the same tag.

.. code-block::

   Failed to update Image repo metadata: Failed to fetch role timestamp in image repository.

This could mean that there is no Target available to update to. If this is a production device, it could mean that there are no :ref:`ref-production-targets`/waves available for that tag.

.. tip::
   The Target metadata available for the device can be checked with the following commands:

   On the device::

      curl -H "x-ats-tags: <tag>" https://<dg>:8443/repo/targets.json --cert client.pem --key pkey.pem --cacert root.crt

   Or on the host::

      fioctl targets list --by-tag <tag> --production

   Where:

   * ``<tag>``: Device tag.
   * ``<dg>``: Device gateway address, defaults to ``ota-lite.foundries.io``. The actual address can be found in ``/var/sota/sota.toml``, ``[tls].server`` field.
   * ``<token>``: API Token with ``targets:read`` scope.

   Check :ref:`ref-troubleshooting_network-connectivity` for a reference on running ``curl`` commands on the device.

**Solution:** :ref:`Create a wave <ref-rm-wave>` for the wanted tag.

* **Configuration file wrong or corrupted**

.. code-block::

   Configuration file wrong or corrupted
   warning: Failed resetting bootcount

This means that the device cannot access the U-Boot environment.

**Solution:** Check if ``fstab`` is properly set.

* **KeyId is not valid**

.. code-block::

   KeyId xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx is not valid to sign for this role (root).

This is not an actual error. It only indicates that the TUF root key has been rotated. It can be shown more than once in the ``aktualizr-lite`` logs depending on how many times the TUF root key has been rotated.

**Solution:** No fix needed, this log can be ignored as this is expected behavior.

OTA Update Fails Because of Missing SPL Keys
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When updating to a newer base ``lmp-manifest``, your Factory may face issues with OTA upgrades from **v85** to the next release.
It manifests as a failed boot attempt and error in the u-boot log:

.. code-block::

    U-Boot SPL 2021.04+fio+g38c3083e39 (Feb 16 2022 - 14:50:02 +0000)
    power_pca9450b_init
    DDRINFO: start DRAM init
    DDRINFO: DRAM rate 3000MTS
    DDRINFO:ddrphy calibration done
    DDRINFO: ddrmix config done
    Normal Boot
    Trying to boot from MMC2
    SPL: Booting secondary boot path: using 0x1300 offset for next boot image
    ## Checking hash(es) for config config-1 ... fit_config_verify_required_sigs: No signature node found: FDT_ERR_NOTFOUND
    SPL_FIT_SIGNATURE_STRICT needs a valid config node in FIT
    ### ERROR ### Please RESET the board ###

This suggests that the SPL key is missing from the factory.
The key is defined in the OE recipe and it defaults to ``spldev``.

.. prompt::

    UBOOT_SPL_SIGN_KEYNAME="spldev"

This can be confirmed by checking whether files ``spldev.key`` or ``spldev.crt`` are missing from the ``lmp-manifest/factory-keys`` directory.
If so, the easiest fix is to generate the keys and add them to the repository.

.. prompt::

    cd factory-keys
    openssl genpkey -algorithm RSA -out spldev.key \
          -pkeyopt rsa_keygen_bits:2048 \
          -pkeyopt rsa_keygen_pubexp:65537
    openssl req -batch -new -x509 -key spldev.key -out spldev.crt

Once the ``spldev.key`` and ``spldev.crt`` are created, add them to the repository.

.. prompt::

    git add factory-keys/spldev.key
    git add factory-keys/spldev.crt
    git commit

Once the commit is pushed upstream, the FoundriesFactory® CI will generate a build that fixes the issue.

.. _ref-troubleshooting_request-entity-too-large:

Request Entity Too Large Error
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This error occurs when your Factory has accumulated too much Target metadata to be signed by The Update Framework (TUF).
All of your Targets contained in :term:`targets.json` can grow large over time::

  Signing local TUF targets
  == 2020-11-24 23:12:53 Running: garage-sign targets sign --repo /root/tmp.dNLAIH
  --key-name targets
  |  signed targets.json to /root/tmp.dNLAIH/roles/targets.json
  |--
  Publishing local TUF targets to the remote TUF repository
  == 2020-11-24 23:12:55 Running: garage-sign targets push --repo /root/tmp.dNLAIH
  |  An error occurred
  |  com.advancedtelematic.libtuf.http.SHttpjServiceClient$HttpjClientError:
  ReposerverHttpClient|PUT|http/413|https://api.foundries.io/ota/repo/magicman//api/v1/user_repo/targets|<html>
  |  <head><title>413 Request Entity Too Large</title></head>
  |  <body>
  |  <center><h1>413 Request Entity Too Large</h1></center>
  |  <hr><center>nginx/1.19.3</center>
  |  </body>
  |  </html>

Solution
""""""""

Over time, the manual pruning (deletion) of Targets is maintenance you should consider.

Pruning can be done using Fioctl™.
This removes outdated Targets from your Factory's :term:`targets.json`, allowing the production of new Targets.

.. warning::

   Ensure there are no important devices running on a Target that is about to be pruned.
   If you are intending on pruning production tags, be cautious and mindful of what you are doing.

You can prune/delete individual Targets by using their TUF Target name::

  fioctl targets prune <TUF_Target_name>

Or, you can prune by tag, such as ``devel`` or ``experimental``::

  fioctl targets prune --by-tag <tag>

We highly recommend not pruning all Targets from a tag to avoid container builds failing from the lack of platform builds for this tag.
To keep the last ``<number>`` of the Targets from a tag, use::

  fioctl targets prune --by-tag <tag> --keep-last <number>

There is also the ``--dryrun`` option.
This lets you can check the pruned targets before running the actual command::

  fioctl targets prune --by-tag <tag> --keep-last <number> --dryrun

.. _ref-ts-howto:

How Tos
--------

Aktualizr-Lite and Fioconfig Polling Time
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Both ``fioconfig`` and ``aktualizr-lite`` poll for new configuration and updates every 5 minutes by default.
It can be helpful to decrease this interval for development purposes.
Following are two ways to achieve this.

Option A: Changing Interval in Runtime
""""""""""""""""""""""""""""""""""""""

1. On your device, create a settings file in the ``/etc/sota/conf.d/`` folder to configure ``aktualizr-lite``.

   .. prompt:: bash device:~$

       sudo mkdir -p /etc/sota/conf.d/
       sudo sh -c 'printf "[uptane]\npolling_sec = <time-sec>" > /etc/sota/conf.d/90-sota-fragment.toml'

2. Next, create a settings file in the ``/etc/default/`` folder to configure ``fioconfig``.

   .. prompt:: bash device:~$

       sudo sh -c 'printf "DAEMON_INTERVAL=<time-sec>" > /etc/default/fioconfig'

3. Restart both services:

   .. prompt:: bash device:~$

       sudo systemctl restart aktualizr-lite
       sudo systemctl restart fioconfig

.. note::
    Make sure to replace ``<time-sec>`` with the expected poll interval in seconds.

Option B: Changing Interval Included in the Build
"""""""""""""""""""""""""""""""""""""""""""""""""

First, configure the **aktualizr-lite** polling interval:

1. Create the ``sota-fragment`` folder in ``meta-subscriber-overrides`` repo:

   .. prompt:: bash host:~$

       cd meta-subscriber-overrides
       mkdir -p recipes-sota/sota-fragment

2. Add a new file under this directory:

   .. prompt:: bash host:~$

        touch recipes-sota/sota-fragment/sota-fragment_0.1.bb

3. Include the content below to the file created in the last step:

   .. code-block:: none

       SUMMARY = "SOTA configuration fragment"
       SECTION = "base"
       LICENSE = "MIT"
       LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"

       inherit allarch

       SRC_URI = " \
               file://90-sota-fragment.toml \
       "

       S = "${WORKDIR}"

       do_install() {
               install -m 0700 -d ${D}${libdir}/sota/conf.d
               install -m 0644 ${WORKDIR}/90-sota-fragment.toml ${D}${libdir}/sota/conf.d/90-sota-fragment.toml
       }

       FILES:${PN} += "${libdir}/sota/conf.d/90-sota-fragment.toml"

4. Create another directory under the one we just created so we can supply the source file (``90-sota-fragment.toml``) for the recipe above:

   .. prompt:: bash host:~$

       cd meta-subscriber-overrides
       mkdir -p recipes-sota/sota-fragment/sota-fragment

5. Create the ``90-sota-fragment.toml`` file under this new directory:

   .. code-block::

       [uptane]
       polling_sec = <time-sec>

.. note::
    Make sure to replace ``<time-sec>`` with the expected poll interval in seconds.

6. In the ``recipes-samples/images/lmp-factory-image.bb`` file, include this new package under ``CORE_IMAGE_BASE_INSTALL``.
   For example:

   .. code-block::

       --- a/recipes-samples/images/lmp-factory-image.bb
       +++ b/recipes-samples/images/lmp-factory-image.bb
       @@ -24,9 +24,10 @@ CORE_IMAGE_BASE_INSTALL += " \
            networkmanager-nmcli \
            git \
            vim \
       +    sota-fragment \
          ..."

7. Next, we configure the ``fioconfig`` daemon interval.
   Create the ``fioconfig`` folder in ``meta-subscriber-overrides`` repo

   .. prompt:: bash host:~$

       cd meta-subscriber-overrides
       mkdir -p recipes-support/fioconfig

8. Add a new recipe file, ``fioconfig_git.bbappend``, under this directory and include the following:

   .. code-block:: none

       FILESEXTRAPATHS:prepend := "${THISDIR}/${PN}:"
       SRC_URI:append = " \
           file://fioconfig.conf \
       "

       do_install:append() {
           install -Dm 0644 ${WORKDIR}/fioconfig.conf ${D}${sysconfdir}/default/fioconfig
       }

9. Create another directory under the one we just created so we can supply the source file (``fioconfig.conf``) for the recipe above:

   .. prompt:: bash host:~$

       cd meta-subscriber-overrides
       mkdir -p recipes-support/fioconfig/fioconfig

10. Create the ``fioconfig.conf`` file under this new directory including:

   .. code-block::

       DAEMON_INTERVAL=<time-sec>

.. note::
    Make sure to replace ``<time-sec>`` with the expected poll interval in seconds.

Commit and trigger a new build to include these new changes and have a new polling interval.

.. _ref-ts-re-register:

Re-Register a Device
^^^^^^^^^^^^^^^^^^^^

During development, you may need to re-register the same device.
Follow these steps to do so:

1. Delete the device from the UI ``Devices`` tab or with:

   .. prompt:: bash host:~$

       fioctl device delete <device-name>

2. Stop ``aktualizr-lite`` and ``fioconfig`` on the device:

   .. prompt:: bash device:~#

       systemctl stop aktualizr-lite
       systemctl stop fioconfig.path
       systemctl stop fioconfig.service

3. Delete both ``sql.db`` and ``client.pem`` on the device:

   .. prompt:: bash device:~#

       rm /var/sota/sql.db
       rm /var/sota/client.pem

4. Lastly, perform the registration again.

.. _ref-ts-tips:

Tips and Abouts
---------------

Allowed Characters for Device Names and Tags
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Along with [a–z], [A–Z], and [0–9], `_`, `-`, and `.` may be used for device names and tags.
In addition, tags also support `+`.

Bind Mounting a File Into a Container
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When bind mounting a file into a container, the parent directory needs to be bind mounted.
If a bind mount destination does not exist, Docker will create the endpoint as an empty directory rather than a file.

The Docker documentation on `containers and bind mounting <https://docs.docker.com/storage/bind-mounts/>`_ is a good place to start if you wish to learn more about this.

.. _ref-troubleshooting_systemd-service:

NXP SE05X Secure Element and PKCS#11 Trusted Application
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There are two memory limits to be aware of.
One is the Secure Element's non-volatile memory.
The other is the built-time configurable PKCS#11 Trusted Application (TA) heap size.

When RSA and EC keys are created using the TA, a request is sent to the Secure Element (SE) for the creation of those keys.
On success, a key is created in the SE's non volatile memory.
The public key is then read back from the SE to the TA persistent storage.
Note only a handle to the private key in the Secure Element is provided and stored by the TA.

During that creation process the TA also keeps a copy of the key on its heap.

This means that a system that creates all of its keys during boot may run out of heap before running out of SE storage.

To avoid this issue, configure OP-TEE with a large enough ``CFG_PKCS11_TA_HEAP_SIZE``.
It should allow the client to fill the SE NVM before an out of memory condition is raised by the TA.
This will help avoid a secure world panic.

An experimental way to validate the thresholds is to loop on RSA or EC key creation until it fails.
If there is a panic or a PKCS#11 OOM fault, ``CFG_PKCS11_TA_HEAP_SIZE`` can then be increased as there is still room in the SE NVM to store more keys.

.. _ref-troubleshooting_network-connectivity:

Debugging Network Connectivity
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When debugging network connectivity and access issues, it can be helpful to use ``curl``.
However, LmP does not ship with the command.

Rather than including ``curl`` on the host device, a simple approach is to run it via a Alpine Linux® container::

    docker run --rm -it alpine
    / # apk add curl
    / # curl

Updates To etc
^^^^^^^^^^^^^^

Files created or modified in ``/etc`` during runtime are not handled by OSTree during an OTA.
For this reason, we suggest setting system-wide configs in ``/usr`` rather than ``/etc`` whenever possible so that these changes are covered by OTA updates.

We suggest managing files that live in ``/etc`` with a systemd service (:ref:`ref-troubleshooting_systemd-service`).
The runtime service should handle the needed updates to the ``/etc`` files.
