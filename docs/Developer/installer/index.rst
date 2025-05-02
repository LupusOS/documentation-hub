.. LupusOS System Configuration Guide

System Configuration and Installer Development
=============================================

This guide provides instructions for performing system configuration tasks on LupusOS, such as creating swap space, listing installed packages, and restoring packages on a new system. These tasks are essential for contributors working on system-level enhancements or preparing environments for LupusOS deployments. Additionally, this section serves as a placeholder for future documentation on LupusOS installer development, which may involve creating or modifying installation tools.

Overview
--------

LupusOS relies on the PISI package management system for software installation and system maintenance. System configuration tasks, such as managing swap space or tracking installed packages, are critical for ensuring system stability and reproducibility. While LupusOS does not currently have a dedicated installer package documented in this guide, this section includes system-level tasks that contributors may encounter during development or deployment.

This section covers:

- Creating and managing swap space.
- Listing and restoring installed packages.
- Guidelines for future installer development contributions.

Prerequisites
-------------

Before performing system configuration tasks, ensure you have:

- A LupusOS system with administrative privileges.
- Installed PISI tools (available by default in LupusOS).
- Familiarity with Linux system administration commands.
- A configured development environment with Git and Docker for contributions (see :doc:`../contributing`).

System Configuration Tasks
--------------------------

### Creating Swap Space

Swap space is a disk partition or file used to extend virtual memory when physical RAM is insufficient. LupusOS supports both swap partitions and swap files.

#### Creating a Swap File

To create a 512 MB swap file:

1. **Allocate Space**:
   Create a file with the desired size (e.g., 512 MB):

   .. code-block:: bash

      sudo dd if=/dev/zero of=/swapfile bs=1M count=512

2. **Set Permissions**:
   Restrict access to the swap file:

   .. code-block:: bash

      sudo chmod 600 /swapfile

3. **Format as Swap**:
   Initialize the file as swap space:

   .. code-block:: bash

      sudo mkswap /swapfile

4. **Enable Swap**:
   Activate the swap file:

   .. code-block:: bash

      sudo swapon /swapfile

5. **Make Permanent**:
   Add an entry to `/etc/fstab` to enable the swap file at boot:

   .. code-block:: bash

      echo "/swapfile none swap sw 0 0" | sudo tee -a /etc/fstab

6. **Verify**:
   Confirm the swap space is active:

   .. code-block:: bash

      swapon -s

#### Creating a Swap Partition

To create a swap partition (e.g., on `/dev/sdX`):

1. **Partition the Disk**:
   Use `fdisk` or `parted` to create a new partition with the `swap` type (type `82` in `fdisk`).

   .. code-block:: bash

      sudo fdisk /dev/sdX

   Follow prompts to create a new partition and set its type to `82`.

2. **Format as Swap**:
   Initialize the partition as swap space:

   .. code-block:: bash

      sudo mkswap /dev/sdX1

3. **Enable Swap**:
   Activate the swap partition:

   .. code-block:: bash

      sudo swapon /dev/sdX1

4. **Make Permanent**:
   Add an entry to `/etc/fstab`:

   .. code-block:: bash

      echo "/dev/sdX1 none swap sw 0 0" | sudo tee -a /etc/fstab

5. **Verify**:
   Check the swap partition status:

   .. code-block:: bash

      swapon -s

### Listing Installed Packages

To maintain or replicate a LupusOS system, you may need to list all installed packages.

1. **List Packages**:
   Use the `pisi` command to generate a list of installed packages:

   .. code-block:: bash

      pisi li -i > installed_packages.txt

   This creates a file (`installed_packages.txt`) containing the names of all installed packages.

2. **Review the List**:
   Inspect the output file to verify the package list:

   .. code-block:: bash

      cat installed_packages.txt

### Restoring Packages on a New System

To replicate a LupusOS system’s software configuration on another machine:

1. **Export Package List**:
   On the source system, generate the list of installed packages:

   .. code-block:: bash

      pisi li -i > installed_packages.txt

2. **Transfer the List**:
   Copy `installed_packages.txt` to the target system (e.g., via `scp` or a USB drive).

3. **Install Packages**:
   On the target system, install the packages listed in the file:

   .. code-block:: bash

      sudo pisi it -y $(cat installed_packages.txt)

   The `-y` flag automatically confirms the installation.

4. **Verify Installation**:
   Check that all packages are installed:

   .. code-block:: bash

      pisi li -i

Installer Development
--------------------

Currently, LupusOS does not have a dedicated installer package documented in this guide. However, contributors interested in developing or enhancing installer tools should follow these guidelines:

1. **Repository Setup**:
   Fork and clone the LupusOS `core` or `main` repository, depending on where installer-related packages are hosted (see :doc:`../contributing`).

2. **Package Structure**:
   Create a new PISI package for the installer, including:
   - `pspec.xml`: Define metadata, dependencies, and file paths.
   - `actions.py`: Implement installation logic using the Actions API (see :doc:`../modules/index`).
   - Additional scripts or configuration files as needed.

3. **Build and Test**:
   Build the installer package in a Docker environment:

   .. code-block:: bash

      pisi bi -dy --ignore-safety /git/main/<installer-package>/pspec.xml
      pisi it <installer-package>.pisi --ignore-safety

   Test the installer in a virtual machine or isolated environment to ensure it correctly configures a LupusOS system.

4. **Submit Contributions**:
   Commit changes and submit a pull request with detailed documentation of the installer’s functionality and dependencies (see :doc:`../contributing`).

Future documentation will expand this section as installer development progresses. Contributors are encouraged to open issues on the LupusOS GitHub repository to discuss new installer features.

Best Practices
--------------

- **Backup Before Changes**:
   Before modifying system configurations (e.g., `/etc/fstab`), create backups to prevent data loss:

   .. code-block:: bash

      sudo cp /etc/fstab /etc/fstab.bak

- **Test in Isolation**:
   Perform configuration changes in a virtual machine or test system to avoid disrupting production environments.
- **Validate Swap**:
   After creating swap space, verify its functionality with `swapon -s` and monitor system performance.
- **Use PISI Commands**:
   Rely on `pisi` commands for package management to ensure compatibility with LupusasOS’s package system.
- **Document Changes**:
   Include detailed notes in pull requests or package history for system configuration changes.
- **Discuss New Features**:
   Open a GitHub issue to discuss new installer or configuration tools before development.

Troubleshooting
---------------

- **Swap Space Not Active**:
  - Verify the `/etc/fstab` entry and ensure the swap file or partition is correctly formatted:

    .. code-block:: bash

       sudo mkswap /swapfile
       sudo swapon /swapfile

  - Check for errors with `dmesg | grep swap`.

- **Package Installation Fails**:
  - Ensure the target system’s package repository is up-to-date:

    .. code-block:: bash

       sudo pisi ur

  - Check for missing dependencies and add them to the installation command:

    .. code-block:: bash

       sudo pisi it -y --ignore-safety $(cat installed_packages.txt)

- **Permission Issues**:
  - Verify that commands are run with `sudo` when administrative privileges are required.
  - Check file permissions (e.g., `chmod 600 /swapfile` for swap files).

Next Steps
----------

For general packaging instructions, refer to :doc:`../packages/index`. For Actions API details, see :doc:`../modules/index`. For kernel development, consult :doc:`../kernel/index`.
