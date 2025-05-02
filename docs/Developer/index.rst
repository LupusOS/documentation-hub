.. LupusOS Developer Documentation

Developer Documentation
======================

Welcome to the developer documentation for LupusOS, a Linux distribution featuring the PISI (Packages Installed Successfully as Intended) binary package management system. This guide provides comprehensive instructions for contributors, covering package creation, system configuration, and development workflows.

Overview
-------

LupusOS leverages the PISI package management system, originally developed for the Pardus project before 2011. PISI enables advanced package operations, such as installation, removal, and system rollback to a previous state. This documentation is designed for developers and packagers aiming to contribute to LupusOS by creating, maintaining, or updating packages.

Key Concepts
~~~~~~~~~~~~

- **Package**: A software, document, or data bundle installable on the system.
- **Source File**: Compressed, downloadable files containing uncompiled code, data, or documentation.
- **PISI Packaging ("Pisilemek")**: The process of converting software into a PISI package.
- **Build**: The process of compiling source code written in languages like C or C++ into executable binaries.
- **Dependency**: Additional packages required for a software’s compilation or execution.
- **Checksum**: A SHA1 hash used to verify the integrity of downloaded source files against the official key provided by the software’s maintainers.

Getting Started
---------------

To contribute to LupusOS, developers should familiarize themselves with the package creation process, which involves:

1. Gathering information about the software to be packaged.
2. Creating PISI build files (e.g., `pspec.xml`, `actions.py`).
3. Testing the package and resolving errors.
4. Maintaining the package by tracking updates and fixes.

Packagers are encouraged to treat their packages as ongoing responsibilities, ensuring they remain up-to-date in the fast-evolving Linux ecosystem. The mantra "package only what you can maintain" underscores the importance of sustainable contributions.

Collaborative Development
~~~~~~~~~~~~~~~~~~~~~~~~

LupusOS thrives on collaborative development. Packagers must share their build files publicly, typically via the LupusOS GitHub repository, to enable community maintenance and review. The repository hosts build files and source code for all LupusOS packages, serving as a valuable resource for new contributors.

Documentation Structure
----------------------

This documentation is organized into the following sections:

.. toctree::
   :maxdepth: 2

   contributing
   installer/index
   kernel/index
   modules/index
   packages/index

Each section provides detailed guidance on specific aspects of LupusOS development, from package creation to kernel configuration and system installer contributions.

Next Steps
----------

Begin by exploring the :doc:`contributing` guide to understand how to set up your development environment and contribute effectively to LupusOS. For specific tasks, refer to the relevant sections, such as :doc:`packages/index` for package creation workflows.
