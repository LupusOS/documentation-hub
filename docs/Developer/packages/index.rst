.. LupusOS Packaging Guide

Packaging for LupusOS
====================

This guide provides detailed instructions for creating, building, and maintaining packages for LupusOS using the PISI (Packages Installed Successfully as Intended) package management system. Packaging is a core activity for LupusOS contributors, enabling the distribution of software, documentation, and data in a consistent and reliable format.

Overview
--------

The PISI package management system allows LupusOS to install, remove, and manage software packages efficiently. A PISI package is a compressed archive containing compiled binaries, configuration files, and metadata, created from source code and build instructions. The packaging process, referred to as "pisilemek," involves preparing source files, defining build and runtime dependencies, and generating a `.pisi` package file.

Key components of a PISI package include:

- **Source Code**: The uncompiled software, typically downloaded as a compressed archive.
- **pspec.xml**: An XML file containing metadata (e.g., package name, version, dependencies, and file paths).
- **actions.py**: A Python script defining the build and installation steps.
- **Additional Files**: Optional patches, configuration files, or scripts (e.g., `translations.xml`, `comar` scripts).

This section outlines the steps to create a PISI package, best practices, and examples of build files.

Packaging Workflow
-----------------

Creating a PISI package involves the following steps:

1. **Identify the Software**:
   - Gather information about the software, including its source archive URL, build system (e.g., autotools, cmake, meson), and dependencies.
   - Check the software’s official website for the latest version and SHA1 checksum.

2. **Prepare Build Files**:
   - Create a directory structure for the package:

     .. code-block:: none

        package_name/
            ├── pspec.xml
            ├── actions.py
            ├── translations.xml
            ├── files/
            │   └── [patch or config files]
            └── comar/
                ├── package.py
                └── service.py

   - Copy template `pspec.xml`, `actions.py`, and `translations.xml` files from the LupusOS documentation or repository (e.g., `developer.lupusos.org` equivalents).
   - Customize `pspec.xml` with package details (e.g., name, version, source URL, dependencies).
   - Adapt `actions.py` to the software’s build system.
   - Add translations to `translations.xml` for localized summaries and descriptions.
   - Include patches or additional files in the `files/` directory and `comar/` scripts for system integration (e.g., user creation, service management).

3. **Build the Package**:
   - Use a Docker container configured for LupusOS (see :doc:`../contributing`) to ensure a consistent build environment.
   - Run the build command:
     .. code-block:: bash
     
        pisi bi -d --ignore-safety /git/main/<path-to-pspec.xml>
        
   - If the SHA1 checksum fails, compute the correct checksum:
     .. code-block:: bash
     
        sha1sum /var/cache/pisi/archives/<source-archive>
        
     Update `pspec.xml` with the new checksum and retry the build.

4. **Resolve Dependencies**:
   - If the build fails due to missing build dependencies, identify them from the error messages or by referencing the software’s documentation or Fedora’s `.spec` file (e.g., `lxqt-panel.spec`).
   - Add dependencies to `pspec.xml` under `<BuildDependencies>` and retry the build.
   - Use the `--install` flag to resume a failed build after fixing issues:
     .. code-block:: bash
     
        pisi bi -d --ignore-safety /git/main/<path-to-pspec.xml> --install

5. **Test the Package**:
   - Install the built package:
     .. code-block:: bash
     
        pisi it <package-file>.pisi --ignore-safety
        
   - Check runtime dependencies:
     .. code-block:: bash
     
        checkelf -s -x <package-file>.pisi
        
   - Add any missing runtime dependencies to `pspec.xml` under `<RuntimeDependencies>` and rebuild.
   - Verify the package’s functionality by running the software.

6. **Finalize the Package**:
   - Generate the final `.pisi` package:
     .. code-block:: bash
     
        pisi bi -d --ignore-safety /git/main/<path-to-pspec.xml> --package
        
   - Validate `pspec.xml` for XML correctness:
     .. code-block:: bash
     
        xmllint --valid pspec.xml

7. **Submit to LupusOS**:
   - Commit build files to your forked repository and submit a pull request (see :doc:`../contributing`).
   - Ensure all changes are documented in the pull request description.

Packaging Rules
---------------

To ensure consistency and reliability, adhere to the following guidelines:

- **Indentation**: Use spaces instead of tabs in `pspec.xml` and `actions.py`.
- **XML Alignment**: Ensure XML tags in `pspec.xml` are properly aligned for readability.
- **File Paths**: Avoid overlapping file paths in `pspec.xml` (e.g., `/usr` should not include `/usr/bin`).
- **Use `pisitools`**: Prefer `pisitools` functions in `actions.py` over `shelltools` for safer builds.
- **Dependency Changes**: Open an issue to discuss new dependencies before submitting a pull request.
- **Systemd**: Since LupusOS does not use systemd, configure packages with `--with-systemdsystemunitdir=no`.
- **Libexec Directory**: Set `--libexecdir=/usr/lib/<package-name>` in `actions.py` for consistency.
- **Icons**: Add an `<Icon>` tag in `pspec.xml` for packages with icons in `/usr/share/pixmaps/icons`. Use `development` for `-devel` packages, `library` for library-only packages, and `terminal` for console applications. If an icon is missing, add it to the `iconcan` package and rebuild it.
- **Reverse Dependencies**: Check for reverse dependencies using:
  .. code-block:: bash
  
     revdep-rebuild -p <package-name>

### Security Best Practices

To ensure the security of LupusOS packages, follow these guidelines:

- **Verify Source Checksums**:
  - Always compare the SHA1 checksum of the source archive against the official checksum provided by the upstream project to prevent supply chain attacks.
  - Example:
    .. code-block:: bash
    
       sha1sum /var/cache/pisi/archives/<source-archive>
       # Compare with upstream checksum from the project’s website

- **Check for CVEs**:
  - Before packaging, check for known vulnerabilities in the software or its dependencies using tools like `cve-check-tool` (if available in LupusOS) or online CVE databases (e.g., `cve.mitre.org`).
  - Example:
    .. code-block:: bash
    
       cve-check-tool <package-name>
    
    .. note::
    
       **To be filled**: Confirm whether `cve-check-tool` or similar tools are available in LupusOS repositories. If unavailable, contributors should consult CVE databases manually.

- **Set Secure Permissions**:
  - Restrict executable file permissions in `pspec.xml` to prevent unauthorized access. For example, use `fileType="executable"` only for necessary binaries.
  - Example `pspec.xml`:
    .. code-block:: xml
    
       <Files>
           <Path fileType="executable" permission="0755">/usr/bin/<binary></Path>
           <Path fileType="data" permission="0644">/usr/share/<package></Path>
       </Files>

- **Monitor Upstream Security Advisories**:
  - Subscribe to the software’s security mailing list or check its website for advisories before packaging new versions.
  - Document any addressed CVEs in the `pspec.xml` `<History>` section:
    .. code-block:: xml
    
       <Update release="2">
           <Date>2025-05-03</Date>
           <Version>1.2.3</Version>
           <Comment>Fixed CVE-2025-1234</Comment>
           <Name>LupusOS Community</Name>
           <Email>admins@lupusos.org</Email>
       </Update>

### Advanced Packaging Scenarios

For complex packaging needs, LupusOS supports advanced scenarios such as multi-architecture packages, conditional dependencies, and cross-compilation.

- **Multi-Architecture Packages**:
  - Specify supported architectures in `pspec.xml` using the `<Architecture>` tag to restrict package builds to specific platforms (e.g., `x86_64`, `i686`).
  - Example `pspec.xml`:
    .. code-block:: xml
    
       <Source>
           <Name>example-package</Name>
           <Architecture>x86_64</Architecture>
           <Architecture>i686</Architecture>
           <!-- Other metadata -->
       </Source>
    
    Use `get.ARCH()` in `actions.py` to adapt build steps for different architectures:
    .. code-block:: python
    
       from pisi.actionsapi import get
       from pisi.actionsapi import autotools
       
       def setup():
           if get.ARCH() == "x86_64":
               autotools.configure("--enable-64bit")
           else:
               autotools.configure("--enable-32bit")

- **Conditional Dependencies**:
  - Use the `condition` attribute in `<Dependency>` tags to specify dependencies that apply only under certain conditions (e.g., specific kernel versions or optional features).
  - Example `pspec.xml`:
    .. code-block:: xml
    
       <Package>
           <Name>example-package</Name>
           <RuntimeDependencies>
               <Dependency condition="kernel_version >= 5.15">libnftables</Dependency>
               <Dependency condition="feature_opengl">mesa</Dependency>
           </RuntimeDependencies>
           <!-- Other package details -->
       </Package>
    
    .. note::
    
       **To be filled**: Confirm the syntax and supported conditions for the `condition` attribute in LupusOS’s PISI implementation. Contributors should verify with the LupusOS PISI documentation.

- **Cross-Compilation**:
  - For packages targeting different architectures (e.g., ARM on an x86_64 host), configure the build environment with cross-compilation tools and set appropriate flags in `actions.py`.
  - Example `actions.py` for cross-compiling a library:
    .. code-block:: python
    
       from pisi.actionsapi import autotools
       from pisi.actionsapi import get
       
       def setup():
           autotools.configure(
               "--host=arm-linux-gnueabihf",
               "--prefix=/usr",
               "CC=arm-linux-gnueabihf-gcc",
               "CXX=arm-linux-gnueabihf-g++"
           )
       
       def build():
           autotools.make()
       
       def install():
           autotools.rawInstall("DESTDIR=%s" % get.installDIR())
    
    Ensure the cross-compilation toolchain (e.g., `arm-linux-gnueabihf-gcc`) is installed in the build environment:
    .. code-block:: bash
    
       sudo pisi it cross-arm-gcc

    .. note::
    
       **To be filled**: Confirm the availability of cross-compilation toolchains in LupusOS repositories and the specific `--host` triplets supported. Contributors should check the LupusOS toolchain documentation.

### Versioning and Changelog Standards

To maintain consistency and traceability, follow these guidelines for versioning and changelogs in `pspec.xml`:

- **Versioning Conventions**:
  - Use the upstream software version (e.g., `1.2.3`) as the `<Version>` in `pspec.xml`.
  - For LupusOS-specific patches or rebuilds, append a release number (e.g., `1.2.3-1`, `1.2.3-2`) in the `<Update release>` attribute.
  - Example:
    .. code-block:: xml
    
       <Source>
           <Name>example-package</Name>
           <Version>1.2.3</Version>
           <!-- Other metadata -->
       </Source>
       <History>
           <Update release="2">
               <Date>2025-05-03</Date>
               <Version>1.2.3</Version>
               <Comment>Applied LupusOS-specific patch for compatibility</Comment>
               <Name>LupusOS Community</Name>
               <Email>admins@lupusos.org</Email>
           </Update>
           <Update release="1">
               <Date>2025-04-01</Date>
               <Version>1.2.3</Version>
               <Comment>Initial release</Comment>
               <Name>LupusOS Community</Name>
               <Email>admins@lupusos.org</Email>
           </Update>
       </History>

- **Changelog Format**:
  - Each `<Update>` entry in the `<History>` section must include:
    - `release`: Incremental number for each update (e.g., `1`, `2`).
    - `Date`: ISO format (e.g., `2025-05-03`).
    - `Version`: Upstream version or patched version.
    - `Comment`: Detailed description of changes (e.g., “Fixed CVE-2025-1234”, “Updated to version 1.2.3”, “Added support for feature X”).
    - `Name`: Packager’s name or “LupusOS Community”.
    - `Email`: Packager’s email or `admins@lupusos.org`.
  - Use clear, concise `<Comment>` entries to document bug fixes, security updates, or new features.
  - Example:
    .. code-block:: xml
    
       <Update release="3">
           <Date>2025-06-01</Date>
           <Version>1.2.4</Version>
           <Comment>Updated to version 1.2.4, fixed CVE-2025-5678</Comment>
           <Name>LupusOS Community</Name>
           <Email>admins@lupusos.org</Email>
       </Update>

- **Best Practices**:
  - Increment the `release` number for each update, even if the upstream version remains unchanged.
  - Document all changes, including minor patches, in the `<History>` section for transparency.
  - Align versioning with upstream releases unless LupusOS-specific modifications require divergence.

.. note::

   **To be filled**: Confirm LupusOS’s official versioning policy (e.g., handling snapshot releases, pre-release versions). Contributors should consult the LupusOS packaging guidelines or community for specific conventions.

Example Build Files
------------------

### pspec.xml

The `pspec.xml` file defines package metadata, source information, and file paths. Below is an example for the `acl` package:

.. code-block:: xml

   <?xml version="1.0" ?>
   <!DOCTYPE PISI SYSTEM "https://www.lupusos.org/projects/pisi/pisi-spec.dtd">
   <PISI>
       <Source>
           <Name>acl</Name>
           <Homepage>https://savannah.nongnu.org/projects/acl</Homepage>
           <Packager>
               <Name>LupusOS Community</Name>
               <Email>admins@lupusos.org</Email>
           </Packager>
           <License>GPLv2+</License>
           <License>LGPLv2.1</License>
           <IsA>app:console</IsA>
           <IsA>library</IsA>
           <Icon>library</Icon>
           <Summary>Access control list utilities</Summary>
           <Description>Utilities and libraries for manipulating access control lists.</Description>
           <Archive sha1sum="6c9e46602adece1c2dae91ed065899d7f810bf01" type="targz">http://download.savannah.gnu.org/releases/acl/acl-2.2.53.tar.gz</Archive>
           <BuildDependencies>
               <Dependency>attr-devel</Dependency>
               <Dependency versionFrom="8.0.1">readline-devel</Dependency>
           </BuildDependencies>
       </Source>
       <Package>
           <Name>acl</Name>
           <RuntimeDependencies>
               <Dependency>attr</Dependency>
           </RuntimeDependencies>
           <Files>
               <Path fileType="executable">/bin</Path>
               <Path fileType="library">/lib</Path>
               <Path fileType="localedata">/usr/share/locale</Path>
               <Path fileType="doc">/usr/share/doc</Path>
               <Path fileType="man">/usr/share/man</Path>
           </Files>
       </Package>
       <Package>
           <Name>acl-devel</Name>
           <PartOf>system.devel</PartOf>
           <Icon>development</Icon>
           <Summary>Development files for acl</Summary>
           <RuntimeDependencies>
               <Dependency release="current">acl</Dependency>
           </RuntimeDependencies>
           <Files>
               <Path fileType="header">/usr/include</Path>
               <Path fileType="library">/usr/lib/pkgconfig</Path>
               <Path fileType="man">/usr/share/man/man3</Path>
           </Files>
       </Package>
       <History>
           <Update release="1">
               <Date>2016-03-01</Date>
               <Version>2.2.52</Version>
               <Comment>First release</Comment>
               <Name>LupusOS Community</Name>
               <Email>admins@lupusos.org</Email>
           </Update>
       </History>
   </PISI>

### actions.py

The `actions.py` script defines the build and installation process. Below is an example for a package using the autotools build system:

.. code-block:: python

   #!/usr/bin/python
   # -*- coding: utf-8 -*-
   #
   # Licensed under the GNU General Public License, version 2.
   # See http://www.gnu.org/copyleft/gpl.txt.

   from pisi.actionsapi import autotools
   from pisi.actionsapi import pisitools
   from pisi.actionsapi import get

   def setup():
       autotools.configure("--libexecdir=/usr/lib/%s" % get.srcNAME())

   def build():
       autotools.make()

   def install():
       autotools.rawInstall("DESTDIR=%s" % get.installDIR())
       pisitools.dodoc("AUTHORS", "ChangeLog", "README", "NEWS")

For other build systems (e.g., cmake, qt5, meson), refer to the :doc:`../modules/index` section for tailored `actions.py` examples.

### translations.xml

The `translations.xml` file provides localized summaries and descriptions:

.. code-block:: xml

   <?xml version="1.0" ?>
   <PISI>
       <Source>
           <Name>acl</Name>
           <Summary xml:lang="tr">Erişim denetim listesi için çeşitli araçlar</Summary>
           <Description xml:lang="tr">Erişim denetim listelerini yönetmeye yarayan araçlar ve kütüphaneler.</Description>
           <Summary xml:lang="fr">Utilitaires et bibliothèques pour les listes de contrôle d'accès.</Summary>
       </Source>
       <Package>
           <Name>acl-devel</Name>
           <Summary xml:lang="tr">acl için geliştirme dosyaları</Summary>
       </Package>
   </PISI>

Troubleshooting
---------------

- **SHA1 Checksum Mismatch**:
  - Compute the correct checksum using `sha1sum` and update `pspec.xml`.
- **Missing Build Dependencies**:
  - Check error messages or Fedora’s `.spec` file for required `-devel` packages.
  - Add dependencies to `pspec.xml` and retry the build.
- **Runtime Dependency Issues**:
  - Use `checkelf -s -x <package-file>.pisi` to identify missing dependencies.
  - Update `pspec.xml` and rebuild.
- **XML Errors**:
  - Validate `pspec.xml` with `xmllint --valid pspec.xml` to catch syntax issues.

### PISI Tools and Debugging

LupusOS provides additional PISI commands to streamline packaging and troubleshoot issues. Below are key tools and debugging strategies:

- **pisi delta**:
  - Creates delta packages to reduce download sizes by packaging only the differences between two package versions.
  - Example:
    .. code-block:: bash
    
       pisi delta old-package.pisi new-package.pisi
    
    This generates a `.delta.pisi` file for efficient updates.

- **pisi graph**:
  - Visualizes package dependencies as a graph, useful for identifying dependency loops or missing dependencies.
  - Example:
    .. code-block:: bash
    
       pisi graph <package-name> > dependencies.dot
       dot -Tpng dependencies.dot -o dependencies.png
    
    This creates a visual dependency graph.

- **pisi check**:
  - Validates installed packages for file integrity and consistency.
  - Example:
    .. code-block:: bash
    
       pisi check <package-name>
    
    This reports any corrupted or missing files.

- **Debugging Common Issues**:
  - **Dependency Loops**:
    - Use `pisi graph` to identify circular dependencies.
    - Adjust `<BuildDependencies>` or `<RuntimeDependencies>` in `pspec.xml` to break loops.
  - **File Conflicts**:
    - Check for overlapping file paths in `pspec.xml` using:
      .. code-block:: bash
      
         pisi bi --check-file-conflicts
         
    - Modify `<Files>` to ensure unique paths.
  - **Build Failures**:
    - Review build logs in `/var/log/pisi` for detailed error messages.
    - Test builds with verbose output:
      .. code-block:: bash
      
         pisi bi -d --verbose /git/main/<path-to-pspec.xml>

.. note::

   **To be filled**: Confirm the availability of `pisi delta`, `pisi graph`, and `pisi check` in the current LupusOS PISI implementation. Contributors should verify these commands in the LupusOS repository or documentation.

For advanced debugging, consult the LupusOS community or open an issue on GitHub.

Next Steps
----------

To learn about the Actions API used in `actions.py`, see the :doc:`../modules/index` section. For kernel development or system configuration tasks, refer to :doc:`../kernel/index` or :doc:`../installer/index`, respectively.
