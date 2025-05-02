.. LupusOS Kernel Development Guide

Kernel Development for LupusOS
==============================

This guide provides instructions for configuring, building, and maintaining the LupusOS kernel, as well as developing out-of-tree kernel modules. Kernel development is a specialized task for advanced contributors, requiring familiarity with Linux kernel internals and the LupusOS build environment. The PISI package management system includes the `kerneltools` module within the Actions API to streamline these processes.

Overview
--------

The LupusOS kernel is a customized Linux kernel tailored to the distribution’s requirements. Contributors may need to build the kernel, apply patches, configure kernel options, or develop external modules to support specific hardware or features. The `kerneltools` module in the PISI Actions API provides functions to automate kernel configuration, compilation, and installation tasks.

This section covers:

- Setting up a kernel development environment.
- Using `kerneltools` functions for kernel and module builds.
- Best practices for kernel contributions.

Prerequisites
-------------

Before starting kernel development, ensure you have:

- A configured LupusOS development environment with Git and Docker (see :doc:`../contributing`).
- The kernel source code, typically available in the LupusOS `core` repository.
- Familiarity with Linux kernel configuration and compilation processes.
- Administrative privileges for installing kernel images and modules.

Setting Up the Environment
--------------------------

1. **Clone the Core Repository**:
   Fork and clone the `core` repository, which contains kernel source and build files:

   .. code-block:: bash

      mkdir ~/pisi-2.0
      cd ~/pisi-2.0
      git clone git@github.com:<your-username>/core.git
      cd core
      git remote add upstream git@github.com:pisilinux/core.git
      git pull upstream master

2. **Set Up Docker**:
   Use a Docker container for a consistent build environment:

   .. code-block:: bash

      sudo service docker start
      sudo docker pull pisilinux/chroot
      sudo docker run -v ~/pisi-2.0:/git -v ~/pisi-2.0/build:/root -v /var/cache/pisi/archives:/var/cache/pisi/archives -v /var/cache/pisi/packages:/var/cache/pisi/packages -itd --security-opt=seccomp:unconfined pisilinux/chroot bash
      sudo docker start <container-id>
      sudo docker attach <container-id>
      service dbus start
      cd /git
      pisi ur
      pisi up -dvsy --ignore-safety

3. **Locate Kernel Build Files**:
   Navigate to the kernel package directory (e.g., `/git/core/kernel/<kernel-package>`), which contains `pspec.xml` and `actions.py` for the kernel.

Kernel Development Workflow
---------------------------

### 1. Configure the Kernel

Use the `kerneltools.configure` function to prepare the kernel source for compilation. This function cleans the source tree of `.orig` files and initiates the configuration process.

**Example** `actions.py`:

.. code-block:: python

   from pisi.actionsapi import kerneltools

   def setup():
       kerneltools.configure()

If you need to modify kernel configuration options, use `kerneltools.updateKConfig` to set new symbols to their default values after editing configuration files.

**Example**:

.. code-block:: python

   def setup():
       kerneltools.configure()
       # Edit .config file (e.g., via sed or manual changes)
       kerneltools.updateKConfig()

### 2. Build the Kernel

Compile the kernel using `kerneltools.build`. By default, debug symbols are disabled to reduce the binary size, but they can be enabled if needed.

**Example**:

.. code-block:: python

   def build():
       kerneltools.build(debugSymbols=False)

### 3. Install the Kernel

Install the kernel image, modules, and optional firmware using `kerneltools.install`. This function checks for loadable module support and installs the kernel to the virtual installation directory.

**Example**:

.. code-block:: python

   def install():
       kerneltools.install(installFirmwares=True)

To exclude firmware (e.g., for a lightweight package), set `installFirmwares=False`.

### 4. Install Kernel Headers

For out-of-tree module development, install kernel headers using `kerneltools.installHeaders`. Specify additional directories if needed (e.g., for specific drivers).

**Example**:

.. code-block:: python

   def install():
       kerneltools.installHeaders(extra=["drivers/media/dvb/dvb-core", "drivers/media/dvb/frontends"])

### 5. Install Libc Headers

Install Linux libc headers, excluding specific directories if necessary, using `kerneltools.installLibcHeaders`.

**Example**:

.. code-block:: python

   def install():
       kerneltools.installLibcHeaders(excludes=["scsi"])

### 6. Install Kernel Source

To package the kernel source for development purposes, use `kerneltools.installSource`.

**Example**:

.. code-block:: python

   def install():
       kerneltools.installSource()

### 7. Create Initramfs

Generate and install an initramfs image using `kerneltools.mkinitramfs`.

**Example**:

.. code-block:: python

   def install():
       kerneltools.mkinitramfs()

### 8. Clean Module Files

Remove module files generated by `depmod` using `kerneltools.cleanModuleFiles`.

**Example**:

.. code-block:: python

   def install():
       kerneltools.cleanModuleFiles()

### 9. Version Management

Use `kerneltools.getKernelVersion` to retrieve the kernel version for module compilation, and `kerneltools.dumpVersion` to write the version to `/etc/kernel`.

**Example**:

.. code-block:: python

   def setup():
       version = kerneltools.getKernelVersion()
       print(f"Building kernel version {version}")

   def install():
       kerneltools.dumpVersion()

Building and Testing
-------------------

1. **Build the Kernel Package**:
   In the Docker container, build the kernel package:

   .. code-block:: bash

      pisi bi -dy --ignore-safety /git/core/kernel/<kernel-package>/pspec.xml

2. **Test the Package**:
   Install the package to verify functionality:

   .. code-block:: bash

      pisi it <kernel-package>.pisi --ignore-safety

   Check runtime dependencies:

   .. code-block:: bash

      checkelf -s -x <kernel-package>.pisi

   Update `pspec.xml` with any missing dependencies and rebuild if necessary.

3. **Validate XML**:
   Ensure `pspec.xml` is syntactically correct:

   .. code-block:: bash

      xmllint --valid pspec.xml

4. **Clean Up**:
   Remove temporary dependencies after testing:

   .. code-block:: bash

      pisi hs -t <transaction-id>

   Stop and remove the Docker container:

   .. code-block:: bash

      sudo docker stop <container-id>
      sudo docker rm <container-id>

Submitting Changes
-----------------

1. **Commit Changes**:
   Add and commit modified files to your local repository:

   .. code-block:: bash

      pisi ix --skip-signing
      git add .
      git commit -m "Update kernel package to version X.Y.Z"
      git push origin master

2. **Submit a Pull Request**:
   Follow the contribution workflow in :doc:`../contributing` to submit a pull request to the `core` repository. Clearly document changes, especially if new kernel options or modules are introduced.

Best Practices
-------------

- **Test Thoroughly**: Verify kernel functionality on a test system before submitting changes, as kernel updates can impact system stability.
- **Document Changes**: Include detailed notes in `pspec.xml` `<History>` and pull request descriptions for transparency.
- **Check Dependencies**: Use `checkelf` to ensure all runtime dependencies are specified in `pspec.xml`.
- **Avoid Systemd**: Configure kernel builds with `--with-systemdsystemunitdir=no`, as LupusOS does not use systemd.
- **Use Spaces**: In `actions.py` and `pspec.xml`, use spaces instead of tabs for indentation.
- **Discuss New Features**: Open an issue to discuss new kernel modules or configuration changes before submitting a pull request.

Kerneltools Reference
--------------------

The `kerneltools` module provides the following functions:

.. list-table:: kerneltools Functions
   :widths: 25 75
   :header-rows: 1

   * - Function
     - Description
   * - ``getKernelVersion(flavour=None)``
     - Returns the kernel version for module compilation (e.g., from `/etc/kernel/kernel` or running kernel).
   * - ``configure()``
     - Cleans the source tree and starts kernel configuration.
   * - ``updateKConfig()``
     - Sets default values for new configuration symbols.
   * - ``dumpVersion()``
     - Writes the kernel version to `/etc/kernel`.
   * - ``build(debugSymbols=False)``
     - Compiles the kernel with optional debug symbols.
   * - ``install(installFirmwares=True)``
     - Installs the kernel image, modules, and optional firmwares.
   * - ``installHeaders(extra=[])``
     - Installs kernel headers for out-of-tree module development.
   * - ``installLibcHeaders(excludes=[])``
     - Installs Linux libc headers, excluding specified directories.
   * - ``installSource()``
     - Installs kernel source files.
   * - ``cleanModuleFiles()``
     - Removes module files generated by `depmod`.
   * - ``mkinitramfs()``
     - Creates and installs an initramfs image.

**Example** `actions.py` for a Kernel Package:

.. code-block:: python

   from pisi.actionsapi import kerneltools
   from pisi.actionsapi import pisitools

   def setup():
       kerneltools.configure()
       kerneltools.updateKConfig()

   def build():
       kerneltools.build()

   def install():
       kerneltools.install()
       kerneltools.installHeaders(extra=["drivers/media/dvb/dvb-core"])
       kerneltools.installSource()
       kerneltools.mkinitramfs()
       kerneltools.cleanModuleFiles()
       pisitools.dodoc("README", "MAINTAINERS")

Next Steps
----------

For general packaging instructions, refer to :doc:`../packages/index`. For Actions API details, see :doc:`../modules/index`. For system configuration tasks, consult :doc:`../installer/index`.
