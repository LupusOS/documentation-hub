.. LupusOS Actions API Reference

Actions API Reference
=====================

The Actions API is a Python-based library provided by the PISI package management system to facilitate the creation of LupusOS packages. It offers a set of functions used in the `actions.py` script to define the build, configuration, and installation steps for a package. This document serves as a reference for the Actions API, detailing its modules and functions to assist packagers in writing effective build scripts.

Overview
--------

The `actions.py` script, a core component of a PISI package, orchestrates the process of transforming source code into a binary `.pisi` package. The Actions API provides modular functions to handle tasks such as configuring source code, compiling binaries, and installing files into a virtual installation directory. These functions are organized into modules, each tailored to specific build systems or tasks.

Key modules include:

- **pisitools**: Core utilities for file operations, symlink creation, and documentation installation.
- **autotools**: Functions for packages using the GNU Autotools build system.
- **cmaketools**: Functions for packages using the CMake build system.
- **shelltools**: Low-level utilities for direct system operations (use sparingly).
- **libtools**: Tools for managing libraries and linker scripts.
- **get**: Functions to retrieve build environment variables and package metadata.
- **kde**: Functions for KDE-based applications.
- **kerneltools**: Tools for kernel configuration and module compilation.

Packagers should prioritize `pisitools` for most operations, as it ensures safe and consistent interactions within the PISI build environment. Other modules are used based on the package’s build system or specific requirements.

Best Practices
--------------

- **Prefer `pisitools`**: Use `pisitools` functions over `shelltools` to maintain consistency and avoid unintended system modifications.
- **Relative Paths**: `pisitools` functions operate with relative paths within the build environment, reducing errors.
- **Minimal External Dependencies**: Avoid using Python modules outside the Actions API to ensure portability.
- **Validate Inputs**: Ensure parameters (e.g., file paths, environment variables) are correct to prevent build failures.

Module Reference
---------------

### pisitools

The `pisitools` module provides essential functions for file operations, documentation installation, and symlink creation within the PISI build environment. It automatically prefixes paths with the working or installation directory, ensuring safe operations.

.. list-table:: pisitools Functions
   :widths: 25 75
   :header-rows: 1

   * - Function
     - Description
   * - ``dobin(sourceFile, destinationDirectory='/usr/bin')``
     - Copies an executable from the work directory to the specified installation directory.
   * - ``dodir(destinationDirectory)``
     - Creates a directory in the installation directory.
   * - ``dodoc(*sourceFiles)``
     - Copies documentation files to ``/usr/share/doc/<package>``. Supports wildcards.
   * - ``doexe(sourceFile, destinationDirectory)``
     - Copies an executable to the specified directory with appropriate permissions. Supports wildcards.
   * - ``dohtml(*sourceFiles)``
     - Copies HTML files to ``/usr/share/doc/<package>/html``. Supports wildcards.
   * - ``doinfo(*sourceFiles)``
     - Copies info files to ``/usr/share/info``. Supports wildcards.
   * - ``dolib(sourceFile, destinationDirectory='/usr/lib')``
     - Copies a library to the specified directory.
   * - ``dolib_a(sourceFile, destinationDirectory='/usr/lib')``
     - Copies a static library with appropriate permissions.
   * - ``dolib_so(sourceFile, destinationDirectory='/usr/lib')``
     - Copies a shared library with appropriate permissions.
   * - ``doman(*sourceFiles)``
     - Copies man pages to ``/usr/share/man``. Supports wildcards.
   * - ``domo(sourceFile, locale, destinationFile)``
     - Compiles a ``.po`` file into a ``.mo`` file for the specified locale and installs it to ``/usr/share/locale/<locale>/LC_MESSAGES``.
   * - ``domove(sourceFile, destination, destinationFile='')``
     - Moves a file within the installation directory, optionally renaming it.
   * - ``dosed(sourceFile, findPattern, replacePattern='')``
     - Performs text replacement in a file using regular expressions.
   * - ``dosbin(sourceFile, destinationDirectory='/usr/sbin')``
     - Copies an executable to the specified system binary directory.
   * - ``dosym(sourceFile, destinationFile)``
     - Creates a symlink in the installation directory.
   * - ``insinto(destinationDirectory, sourceFile, destinationFile='', sym=True)``
     - Copies a file to the specified directory, preserving permissions. Supports wildcards if ``destinationFile`` is unset.
   * - ``newdoc(sourceFile, destinationFile)``
     - Copies a documentation file with a new name to ``/usr/share/doc/<package>``.
   * - ``newman(sourceFile, destinationFile)``
     - Copies a man page with a new name to ``/usr/share/man/man<suffix>``.
   * - ``remove(sourceFile)``
     - Deletes a file from the installation directory.
   * - ``rename(sourceFile, destinationFile)``
     - Renames a file within the installation directory.
   * - ``removeDir(destinationDirectory)``
     - Deletes a directory and its contents from the installation directory.

**Example**:

.. code-block:: python

   from pisi.actionsapi import pisitools

   def install():
       pisitools.dobin("sed/sed", "/bin")
       pisitools.dodoc("README", "ChangeLog")
       pisitools.dosym("gzip", "/bin/gunzip")
       pisitools.remove("/usr/share/man/man1/mt.1")

### autotools

The `autotools` module supports packages using the GNU Autotools build system (e.g., `configure`, `make`).

.. list-table:: autotools Functions
   :widths: 25 75
   :header-rows: 1

   * - Function
     - Description
   * - ``configure(parameters='')``
     - Configures the source with default PISI parameters (e.g., prefix, mandir) and user-provided parameters.
   * - ``rawConfigure(parameters='', prefix='')``
     - Configures the source without default parameters.
   * - ``compile(parameters='')``
     - Compiles the source with default GCC flags and user-provided parameters.
   * - ``make(parameters='')``
     - Builds the source with user-provided parameters.
   * - ``install(parameters='')``
     - Installs the source to the installation directory with default parameters.
   * - ``rawInstall(parameters='')``
     - Installs the source without default parameters.
   * - ``aclocal(parameters='')``
     - Generates `aclocal.m4` from `configure.in`.
   * - ``autoconf(parameters='')``
     - Generates the `configure` script.
   * - ``autoreconf(parameters='')``
     - Regenerates the `configure` script.
   * - ``automake(parameters='')``
     - Generates the `Makefile`.
   * - ``autoheader(parameters='')``
     - Generates template files for `configure`.

**Example**:

.. code-block:: python

   from pisi.actionsapi import autotools

   def setup():
       autotools.configure("--enable-nls --bindir=/bin")

   def build():
       autotools.make()

   def install():
       autotools.rawInstall("DESTDIR=%s" % get.installDIR())

### cmaketools

The `cmaketools` module supports packages using the CMake build system.

.. list-table:: cmaketools Functions
   :widths: 25 75
   :header-rows: 1

   * - Function
     - Description
   * - ``configure(parameters='', sourceDir='', installPrefix='%s' % get.defaultprefixDIR())``
     - Configures the source with CMake.
   * - ``make(parameters='')``
     - Builds the source with CMake.
   * - ``install(parameters='', argument='install')``
     - Installs the source with default parameters.
   * - ``rawInstall(parameters='', argument='install')``
     - Installs the source without default parameters.

**Example**:

.. code-block:: python

   from pisi.actionsapi import cmaketools
   from pisi.actionsapi import shelltools

   def setup():
       shelltools.makedirs("build")
       shelltools.cd("build")
       cmaketools.configure("-DCMAKE_INSTALL_PREFIX=/usr", sourceDir="..")

   def build():
       shelltools.cd("build")
       cmaketools.make()

   def install():
       shelltools.cd("build")
       cmaketools.rawInstall("DESTDIR=%s" % get.installDIR())

### shelltools

The `shelltools` module provides low-level utilities for direct system operations. Use sparingly, as `pisitools` is preferred for most tasks.

.. list-table:: shelltools Functions
   :widths: 25 75
   :header-rows: 1

   * - Function
     - Description
   * - ``can_access_file(sourceFile)``
     - Checks if a file is accessible.
   * - ``can_access_directory(destinationDirectory)``
     - Checks if a directory is accessible.
   * - ``makedirs(destinationDirectory)``
     - Creates a directory.
   * - ``chmod(sourceFile, mode=0755)``
     - Changes file permissions. Supports wildcards.
   * - ``unlink(sourceFile)``
     - Deletes a file.
   * - ``unlinkDir(sourceDirectory)``
     - Deletes a directory and its contents.
   * - ``move(sourceFile, destinationFile)``
     - Moves a file or directory. Supports wildcards.
   * - ``copy(sourceFile, destinationFile)``
     - Copies a file. Supports wildcards.
   * - ``copytree(source, destination, sym=False)``
     - Copies a directory tree.
   * - ``touch(sourceFile)``
     - Updates a file’s access time or creates an empty file.
   * - ``cd(directoryName='')``
     - Changes the working directory.
   * - ``ls(source)``
     - Lists files or directories. Supports wildcards.
   * - ``export(key, value)``
     - Sets an environment variable.
   * - ``system(command)``
     - Executes a shell command.
   * - ``isLink(sourceFile)``
     - Checks if a file is a symlink.
   * - ``realPath(sourceFile)``
     - Resolves a symlink to its target path.
   * - ``baseName(sourceFile)``
     - Returns the file name from a path.
   * - ``dirName(sourceFile)``
     - Returns the directory path.
   * - ``sym(sourceFile, destinationFile)``
     - Creates a symlink.

**Example**:

.. code-block:: python

   from pisi.actionsapi import shelltools

   def setup():
       shelltools.makedirs("%s/build" % get.workDIR())
       shelltools.export("WANT_AUTOCONF", "2.5")
       shelltools.system("./update-pciids.sh")

### libtools

The `libtools` module manages libraries and linker scripts.

.. list-table:: libtools Functions
   :widths: 25 75
   :header-rows: 1

   * - Function
     - Description
   * - ``preplib(sourceDirectory='/usr/lib')``
     - Runs `ldconfig` in the specified directory.
   * - ``gnuconfig_update()``
     - Copies updated `config.sub` and `config.guess` files to the source.
   * - ``libtoolize(parameters='')``
     - Prepares the source for `libtool` usage.
   * - ``gen_usr_ldscript(dynamicLib)``
     - Creates a placeholder dynamic library in `/usr/lib` to resolve linking issues.

**Example**:

.. code-block:: python

   from pisi.actionsapi import libtools

   def setup():
       libtools.libtoolize("--force")
       libtools.gen_usr_ldscript("libhandle.so")

### get

The `get` module retrieves build environment variables and package metadata.

.. list-table:: get Functions
   :widths: 25 75
   :header-rows: 1

   * - Function
     - Description
   * - ``curDIR()``
     - Returns the current working directory.
   * - ``curKERNEL()``
     - Returns the running kernel version.
   * - ``curPYTHON()``
     - Returns the Python version.
   * - ``curPERL()``
     - Returns the Perl version.
   * - ``ENV(environ)``
     - Returns the value of an environment variable.
   * - ``pkgDIR()``
     - Returns the binary package directory (e.g., `/var/cache/pisi/packages`).
   * - ``workDIR()``
     - Returns the package working directory.
   * - ``installDIR()``
     - Returns the package installation directory.
   * - ``lsbINFO()``
     - Returns the contents of `/etc/lsb-release`.
   * - ``srcNAME()``
     - Returns the package source name.
   * - ``srcVERSION()``
     - Returns the package source version.
   * - ``srcDIR()``
     - Returns the package source directory name.
   * - ``ARCH()``
     - Returns the architecture (e.g., `i686`).
   * - ``HOST()``
     - Returns the host triplet (e.g., `i686-pc-linux-gnu`).
   * - ``CFLAGS()``
     - Returns the default C compiler flags.
   * - ``CXXFLAGS()``
     - Returns the default C++ compiler flags.
   * - ``LDFLAGS()``
     - Returns the default linker flags.
   * - ``makeJOBS()``
     - Returns the default number of build jobs.
   * - ``buildTYPE()``
     - Returns the build type from `pspec.xml`.
   * - ``docDIR()``, ``sbinDIR()``, ``infoDIR()``, ``manDIR()``, ``dataDIR()``, ``confDIR()``, ``localstateDIR()``, ``libexecDIR()``, ``defaultprefixDIR()``, ``kdeDIR()``, ``qtDIR()``
     - Return default directory paths for various file types.
   * - ``AR()``, ``AS()``, ``CC()``, ``CXX()``, ``LD()``, ``NM()``, ``RANLIB()``, ``F77()``, ``GCJ()``
     - Return GNU binutils or compiler executable names.

**Example**:

.. code-block:: python

   from pisi.actionsapi import get

   def setup():
       print(f"Building {get.srcNAME()} version {get.srcVERSION()} for {get.ARCH()}")

### kde

The `kde` module supports KDE-based applications.

.. list-table:: kde Functions
   :widths: 25 75
   :header-rows: 1

   * - Function
     - Description
   * - ``configure(parameters='')``
     - Configures a KDE package with default and user-provided parameters.
   * - ``make(parameters='')``
     - Builds a KDE package.
   * - ``install(parameters='install')``
     - Installs a KDE package.

**Example**:

.. code-block:: python

   from pisi.actionsapi import kde

   def setup():
       kde.configure("--with-libsamplerate")

   def build():
       kde.make()

   def install():
       kde.install()

### kerneltools

The `kerneltools` module supports kernel configuration and module compilation.

.. list-table:: kerneltools Functions
   :widths: 25 75
   :header-rows: 1

   * - Function
     - Description
   * - ``getKernelVersion(flavour=None)``
     - Returns the kernel version for module compilation.
   * - ``configure()``
     - Cleans and starts kernel configuration.
   * - ``updateKConfig()``
     - Sets default values for new kernel configuration symbols.
   * - ``dumpVersion()``
     - Writes the kernel version to `/etc/kernel`.
   * - ``build(debugSymbols=False)``
     - Builds the kernel with optional debug symbols.
   * - ``install(installFirmwares=True)``
     - Installs the kernel image and optional firmwares.
   * - ``installHeaders(extra=[])``
     - Installs kernel headers for out-of-tree modules.
   * - ``installLibcHeaders(excludes=[])``
     - Installs Linux libc headers, excluding specified items.
   * - ``installSource()``
     - Installs kernel source files.
   * - ``cleanModuleFiles()``
     - Removes module files generated by `depmod`.
   * - ``mkinitramfs()``
     - Creates and installs an initramfs image.

**Example**:

.. code-block:: python

   from pisi.actionsapi import kerneltools

   def build():
       kerneltools.build()

   def install():
       kerneltools.install()
       kerneltools.installHeaders(extra=["drivers/media/dvb/dvb-core"])

Next Steps
----------

For detailed packaging instructions, refer to the :doc:`../packages/index` section. For kernel development, see :doc:`../kernel/index`. For system configuration tasks, consult :doc:`../installer/index`.
