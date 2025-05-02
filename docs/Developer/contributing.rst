.. LupusOS Contributing Guide

Contributing to LupusOS
======================

This guide outlines the process for contributing to LupusOS, a Linux distribution powered by the PISI package management system. Contributors are essential to maintaining and expanding LupusOS’s package ecosystem and system features. This document details how to set up your development environment, configure necessary tools, and submit contributions via GitHub.

Prerequisites
-------------

Before contributing, ensure you have:

- A GitHub account.
- Basic familiarity with Git and Docker.
- A Linux environment (preferably LupusOS) with administrative privileges.
- Installed packages: `git` and `openssh` (install via `sudo pisi it git openssh`).

Setting Up Your Development Environment
--------------------------------------

### 1. Configure Git

LupusOS uses GitHub for version control and collaboration. Follow these steps to set up Git:

1. **Create a GitHub Account**: If you don’t have an account, sign up at `github.com <https://github.com>`_.
2. **Fork LupusOS Repositories**: Fork the `main` and `core` repositories from `github.com/LupusOS` to your GitHub account.
3. **Clone Repositories Locally**:
   Create a working directory (e.g., `~/pisi-2.0`) and clone your forked repositories:

   .. code-block:: bash

      mkdir ~/pisi-2.0
      cd ~/pisi-2.0
      git clone git@github.com:<your-username>/main.git
      git clone git@github.com:<your-username>/core.git

4. **Configure Git Settings**:
   Set your Git identity and credential caching:

   .. code-block:: bash

      git config --global user.name "<your-github-username>"
      git config --global user.email "<your-email@example.com>"
      git config --global credential.helper cache
      git config --global credential.helper 'cache --timeout=3600'

5. **Add Upstream Remotes**:
   Link your local repositories to the official LupusOS repositories to stay updated:

   .. code-block:: bash

      cd ~/pisi-2.0/main
      git remote add upstream git@github.com:LupusOS/main.git
      cd ~/pisi-2.0/core
      git remote add upstream git@github.com:LupusOS/core.git

6. **Sync with Upstream**:
   Before starting work, pull the latest changes from the upstream repositories:

   .. code-block:: bash

      git pull upstream master
      git push origin master

7. **Set Up SSH Keys**:
   Generate an SSH key pair and add the public key to your GitHub account:

   .. code-block:: bash

      ssh-keygen

   Press Enter to accept defaults (no passphrase). Copy the public key from `~/.ssh/id_rsa.pub` and add it to GitHub under *Settings > SSH and GPG Keys > New SSH Key*.

### 2. Set Up Docker

LupusOS uses Docker to provide a consistent build environment for packages. Follow these steps to configure Docker:

1. **Start Docker Service**:
   Ensure Docker is running:

   .. code-block:: bash

      sudo service docker start

2. **Pull LupusOS Chroot Image**:
   Download the official LupusOS Docker image:

   .. code-block:: bash

      sudo docker pull LupusOS/chroot

3. **Run Docker Container**:
   Mount your local directories and start the container:

   .. code-block:: bash

      sudo docker run -v ~/pisi-2.0:/git -v ~/pisi-2.0/build:/root -v /var/cache/pisi/archives:/var/cache/pisi/archives -v /var/cache/pisi/packages:/var/cache/pisi/packages -itd --security-opt=seccomp:unconfined LupusOS/chroot bash

4. **Access the Container**:
   Start and attach to the container:

   .. code-block:: bash

      sudo docker ps -a  # Note the container ID or name
      sudo docker start <container-id>
      sudo docker attach <container-id>

5. **Initialize the Build Environment**:
   Inside the container, start D-Bus and update the package repository:

   .. code-block:: bash

      service dbus start
      cd /git
      pisi ur
      pisi up -dvsy --ignore-safety

Your environment is now ready for building packages.

Contribution Workflow
--------------------

### 1. Prepare Your Changes

1. **Sync with Upstream**:
   Before making changes, ensure your local repository is up-to-date:

   .. code-block:: bash

      git pull upstream master

2. **Make Changes**:
   Edit or create package build files (e.g., `pspec.xml`, `actions.py`) in the appropriate repository (`main` or `core`).

3. **Build and Test**:
   Build the package in the Docker container:

   .. code-block:: bash

      pisi bi -dy --ignore-safety /git/main/<path-to-pspec.xml>

   Install and test the package, checking runtime dependencies with:

   .. code-block:: bash

      pisi it <package-file>.pisi --ignore-safety
      checkelf -s -x <package-file>.pisi

   Update `pspec.xml` with any missing dependencies and rebuild as needed.

### 2. Testing and Continuous Integration

To ensure high-quality contributions, test packages thoroughly and leverage continuous integration (CI) workflows where possible.

- **Local Testing**:
  - Validate package integrity using `pisi check`:
    .. code-block:: bash
    
       pisi check <package-name>
    
    This verifies file consistency and detects corruption.
  - Test package installation in a clean LupusOS environment using QEMU:
    .. code-block:: bash
    
       qemu-system-x86_64 -cdrom lupusos.iso -m 2048 -hda test-disk.img
       # Install and test the package in the QEMU environment
    
    Create a disk image for testing:
    .. code-block:: bash
    
       qemu-img create -f qcow2 test-disk.img 20G
    
  - Check for file conflicts and dependency issues:
    .. code-block:: bash
    
       pisi bi --check-file-conflicts
       checkelf -s -x <package-file>.pisi

- **Continuous Integration with GitHub Actions**:
  - LupusOS may use GitHub Actions to automate package validation. A typical CI workflow builds packages, validates `pspec.xml`, and tests installation in a Docker container.
  - Example GitHub Actions workflow (place in `.github/workflows/pisi-build.yml`):
    .. code-block:: yaml
    
       name: PISI Package Build and Test
       
       on:
         push:
           branches: [ master ]
         pull_request:
           branches: [ master ]
       
       jobs:
         build:
           runs-on: ubuntu-latest
           
           steps:
           - name: Checkout repository
             uses: actions/checkout@v3
           
           - name: Set up Docker
             uses: docker/setup-buildx-action@v2
           
           - name: Pull LupusOS chroot image
             run: docker pull LupusOS/chroot
           
           - name: Build and test package
             run: |
               docker run --rm \
                 -v $(pwd):/git \
                 -v $(pwd)/build:/root \
                 -v /var/cache/pisi/archives:/var/cache/pisi/archives \
                 -v /var/cache/pisi/packages:/var/cache/pisi/packages \
                 LupusOS/chroot bash -c "
                   service dbus start &&
                   cd /git &&
                   pisi ur &&
                   pisi bi -dy --ignore-safety /git/main/<path-to-pspec.xml> &&
                   xmllint --valid /git/main/<path-to-pspec.xml> &&
                   pisi it /var/cache/pisi/packages/*.pisi --ignore-safety &&
                   checkelf -s -x /var/cache/pisi/packages/*.pisi
                 "
    
    This workflow:
    - Checks out the repository.
    - Sets up Docker.
    - Pulls the `LupusOS/chroot` image.
    - Builds the package, validates `pspec.xml`, installs the package, and checks dependencies.
  - Replace `<path-to-pspec.xml>` with the actual package path or use a script to detect modified packages.

.. note::

   **To be filled**: Confirm whether LupusOS uses GitHub Actions or another CI/CD system for package validation. Contributors should verify the official CI infrastructure and adapt the workflow (e.g., specific paths, additional tests) as needed.

### 3. Commit Changes

1. **Index Changes**:
   Add modified files to the Git index:

   .. code-block:: bash

      pisi ix --skip-signing
      git add <package-name>  # Or git add . for all changes

2. **Commit**:
   Create a commit with a descriptive message in English:

   .. code-block:: bash

      git commit -m "Update <package-name> to version X.Y.Z"

3. **Push to Your Fork**:
   Push changes to your GitHub repository:

   .. code-block:: bash

      git push origin master

### 4. Submit a Pull Request

1. **Create a Pull Request**:
   - Navigate to your forked repository on GitHub.
   - Select the repository (`main` or `core`) and click *New Pull Request*.
   - Ensure the base repository is `LupusOS/<repository>` and the base branch is `master`.
   - Review the changes and click *Create Pull Request*.
   - Provide a clear summary of your changes in the pull request description.

2. **Discuss and Revise**:
   Respond to feedback from maintainers. If new dependencies are introduced, open an issue for discussion before submitting the pull request.

### 5. Clean Up

After building a package, clean up dependencies installed during testing:

.. code-block:: bash

   pisi hs -t <transaction-id>

Stop and remove the Docker container when done:

.. code-block:: bash

   sudo docker stop <container-id>
   sudo docker rm <container-id>

Community Resources
------------------

LupusOS thrives on community collaboration. Engage with other contributors through the following channels:

- **GitHub Repository**: `github.com/LupusOS <https://github.com/LupusOS>`_ (official repositories for `main` and `core`).
- **Community Forums**: LupusOS forums (to be confirmed).
- **IRC Channel**: `#lupusos` on Libera.Chat (to be confirmed).
- **Mailing List**: LupusOS developer mailing list (to be confirmed).
- **Developer Portal**: LupusOS developer portal (to be confirmed, e.g., `developer.lupusos.org`).

.. note::

   **To be filled**: The official community channels (forums, IRC, mailing list, developer portal) need to be verified by the LupusOS community. Contributors should check the LupusOS website or GitHub for up-to-date contact information.

Join these channels to discuss packaging issues, propose new features, or seek assistance with contributions.

Best Practices
-------------

- **Use Spaces, Not Tabs**: In `pspec.xml` and `actions.py`, use spaces for indentation to maintain consistency.
- **Validate XML**: Use `xmllint --valid pspec.xml` to check `pspec.xml` files for errors.
- **Prefer `pisitools`**: In `actions.py`, use `pisitools` functions over `shelltools` for safer and more consistent builds.
- **Discuss New Dependencies**: Open an issue to discuss new dependencies before submitting a pull request.
- **Maintain Packages**: Only package software you can actively maintain, ensuring timely updates and bug fixes.
- **Follow Packaging Rules**: Adhere to LupusOS packaging guidelines, such as setting `--with-systemdsystemunitdir=no` for builds and using `--libexecdir=/usr/lib/<package-name>`.

Next Steps
----------

To start packaging, refer to the :doc:`../packages/index` guide for detailed instructions on creating PISI packages. For advanced topics, explore the :doc:`../modules/index` section for Actions API documentation or the :doc:`../kernel/index` section for kernel development.
