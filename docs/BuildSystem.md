## Contents

* **[Contents](BuildSystem.md#contents)**
* **[Introduction](BuildSystem.md#introduction)**
* **[Obtaining BLIS](BuildSystem.md#obtaining-blis)**
* **[Step 1: Chose a framework configuration](BuildSystem.md#step-1-choose-a-framework-configuration)**
* **[Step 2: Running `configure`](BuildSystem.md#step-2-running-configure)**
* **[Step 3: Compilation](BuildSystem.md#step-3-compilation)**
* **[Step 3b: Testing (optional)](BuildSystem.md#step-3b-testing-optional)**
* **[Step 4: Installation](BuildSystem.md#step-4-installation)**
* **[Cleaning out build products](BuildSystem.md#cleaning-out-build-products)**
* **[Linking against BLIS](BuildSystem.md#linking-against-blis)**
* **[Uninstalling](BuildSystem.md#uninstalling)**
* **[Conclusion](BuildSystem.md#conclusion)**

## Introduction

This document describes how to configure, compile, and install a BLIS library on your local system.

The BLIS build system was designed for use with GNU/Linux (or some other sane UNIX). Other requirements are:

  * Python (2.7 or later)
  * GNU `bash` (3.2 or later)
  * GNU `make`
  * a working C compiler

We also require various other shell utilities that are so ubiquitous that they are not worth mentioning (such as `mv`, `mkdir`, `find`, and so forth). If you are missing these utilities, then you have much bigger problems than not being able to build BLIS.


## Obtaining BLIS

Before starting, you must obtain a copy of BLIS.

If you are an end-user (i.e., not a developer), you can download a tarball or zip file of the latest tagged version by returning to the main [BLIS homepage](https://github.com/flame/blis) and clicking on the [releases](https://github.com/flame/blis/releases) link. **However**, we highly recommend that you instead clone a copy using the command:
```
$ git clone https://github.com/flame/blis.git
```

Cloning a repository allows users and developers alike to quickly and easily pull in new commits as they are available, including commits that occur **between** tagged releases.

Once you download the BLIS distribution, the top-level directory should look something like:
```
$ ls
CHANGELOG  Makefile      common.mk        configure  mpi_test     testsuite
CREDITS    README.md     config           frame      obj          version
INSTALL    bli_config.h  config.mk        kernels    ref_kernels  windows
LICENSE    build         config_registry  lib        test
```


## Step 1: Choose a framework configuration

The first step is to choose how to configure BLIS. Specifically, a user must decide which configuration to use, or whether to allow `configure` to automatically guess the best configuration for your hardware. (Note: This automatic configuration selection only applies to x86_64 systems.)

Configurations are described in detail in the [Configuration Guide](ConfigurationHowTo.md).

Generally speaking, a configuration consists of several files that reside in a sub-directory of the `config` directory. To see a list of the available configurations, you may inspect this directory, or run `configure` with no arguments. Here are the current (as of this writing) contents of the `config` directory:
```
$ ls config
amd64      cortexa15  excavator  intel64  old         power7       template
bgq        cortexa57  generic    knc      penryn      sandybridge  zen
bulldozer  cortexa9   haswell    knl      piledriver  steamroller
```
There is one additional configuration available that is not present in the `config` directory, and that is `auto`.
By targeting the `auto` configuration (i.e., `./configure auto`), the user is requesting that `configure` select a configuration automatically based on the detected features of the processor.

Another special configuration (one that, unlike `auto`, _is_ present in `config`) is the `generic` configuration. This configuration, like its name suggests, is architecture-agnostic and may be targeted in virtually any environment that supports the minimum build requirements of BLIS. The `generic` configuration uses a set of built-in, portable reference kernels (written in C99) that should work without modification on most, if not all, architectures. These reference kernels, however, should be expected to yield relatively low performance since they do not employ any architecture-specific optimizations beyond those the compiler provides automatically. (Historical note: The `generic` configuration corresponds to the `reference` configuration of previous releases of BLIS.)

If you are a BLIS developer and wish to create your own configuration, either from scratch or using an existing configuration as a starting point, please read the BLIS [Configuration Guide](ConfigurationHowTo.md).

## Step 2: Running `configure`

This step should be somewhat familiar to many people who use open source software. To configure the build system, simply run:
```
$ ./configure <configname>
```
where `<configname>` is the configuration sub-directory name you chose in [Step 1](BuildSystem.md#step-1-choose-a-framework-configuration) above. If `<configname>` is not given, a helpful message is printed reminding you to explicit specify a configuration name along with a list of valid configuration families and their implied sub-configurations. For more information on sub-configurations and families, please see the BLIS [Configuration Guide](ConfigurationHowTo.md).

Alternatively, `configure` can automatically select a configuration based on your hardware:
```
$ ./configure auto
```
However, as of this writing, only a limited number of architectures are detected. If the `configure` script is not able to detect your architecture, the `generic` configuration will be used. 

Upon running configure, you will get output similar to the following. The exact output will depend on whether you cloned BLIS from a `git` repository or whether you obtained BLIS via a downloadable tarball from the [releases](https://github.com/flame/blis/releases) page.
```
$ ./configure haswell
configure: using 'gcc' compiler.
configure: found gcc version 5.4.0 (maj: 5, min: 4, rev: 0).
configure: checking for blacklisted configurations due to gcc 5.4.0.
configure: warning: gcc 5.4.0 does not support 'skx'; adding to blacklist.
configure: found assembler ('as') version 2.26.1 (maj: 2, min: 26, rev: 1).
configure: checking for blacklisted configurations due to as 2.26.1.
configure: configuration blacklist:
configure:   skx
configure: reading configuration registry...done.
configure: determining default version string.
configure: found '.git' directory; assuming git clone.
configure: executing: git describe --tags.
configure: got back 0.3.2-16-gb699bb1f.
configure: truncating to 0.3.2-16.
configure: starting configuration of BLIS 0.3.2-16.
configure: configuring with official version string.
configure: found shared library .so version '0.0.0'.
configure:   .so major version: 0
configure:   .so minor.build version: 0.0
configure: manual configuration requested; configuring with 'haswell'.
configure: checking configuration against contents of 'config_registry'.
configure: configuration 'haswell' is registered.
configure: 'haswell' is defined as having the following sub-configurations:
configure:    haswell
configure: which collectively require the following kernels:
configure:    haswell zen
configure: checking sub-configurations:
configure:   'haswell' is registered...and exists.
configure: checking sub-configurations' requisite kernels:
configure:   'haswell' kernels...exist.
configure:   'zen' kernels...exist.
configure: no install prefix option given; defaulting to '/u/field/blis'.
configure: no install libdir option given; defaulting to PREFIX/lib.
configure: no install includedir option given; defaulting to PREFIX/include.
configure: final installation directories:
configure:   libdir:     /u/field/blis/lib
configure:   includedir: /u/field/blis/include
configure: debug symbols disabled.
configure: disabling verbose make output. (enable with 'make V=1'.)
configure: building BLIS as a static library.
configure: threading is disabled.
configure: internal memory pools for packing buffers are enabled.
configure: libmemkind not found; disabling.
configure: the BLAS compatibility layer is enabled.
configure: the CBLAS compatibility layer is disabled.
configure: the internal integer size is automatically determined.
configure: the BLAS/CBLAS interface integer size is 32-bit.
configure: creating ./config.mk from ./build/config.mk.in
configure: creating ./bli_config.h from ./build/bli_config.h.in
configure: creating ./obj/haswell
configure: creating ./obj/haswell/config
configure: creating ./obj/haswell/config/haswell
configure: creating ./obj/haswell/kernels
configure: creating ./obj/haswell/kernels/haswell
configure: creating ./obj/haswell/kernels/zen
configure: creating ./obj/haswell/ref_kernels
configure: creating ./obj/haswell/ref_kernels/haswell
configure: creating ./obj/haswell/frame
configure: creating ./obj/haswell/blastest
configure: creating ./obj/haswell/testsuite
configure: creating ./lib/haswell
configure: creating ./include/haswell
configure: mirroring ./config/haswell to ./obj/haswell/config/haswell
configure: mirroring ./kernels/haswell to ./obj/haswell/kernels/haswell
configure: mirroring ./kernels/zen to ./obj/haswell/kernels/zen
configure: mirroring ./ref_kernels to ./obj/haswell/ref_kernels/haswell
configure: mirroring ./frame to ./obj/haswell/frame
configure: creating makefile fragments in ./config/haswell
configure: creating makefile fragments in ./kernels/haswell
configure: creating makefile fragments in ./kernels/zen
configure: creating makefile fragments in ./ref_kernels
configure: creating makefile fragments in ./frame
configure: configured to build within top-level directory of source distribution.
```
The installation prefix can be specified via the `--prefix=PREFIX` option:
```
  $ ./configure --prefix=/usr <configname>
```
This will cause libraries to eventually be installed (via `make install`) to `PREFIX/lib` and development headers to be installed to `PREFIX/include`. (The default value of `PREFIX` is `$(HOME)/blis`.) You can also specify the library install directory separately from the development header install directory with the `--libdir=LIBDIR` and `--includedir=INCDIR` options, respectively:
```
  $ ./configure --libdir=/usr/lib --includedir=/usr/include <configname>
```
The `--libdir=LIBDIR` and `--includedir=INCDIR` options will override any `PREFIX` path, whether it was specified explicitly via `--prefix` or implicitly (via the default). That is, `LIBDIR` defaults to `PREFIX/lib` and `INCDIR` defaults to `PREFIX/include`, but each will be overriden by their respective `--libdir`/`--includedir` options. So,
```
  $ ./configure --libdir=/usr/lib <configname>

```
will configure BLIS to install libraries to `/usr/lib` and header files to the default location (`$HOME/blis/include`).
Also, note that `configure` will create any installation directories that do not already exist.

For a complete list of supported `configure` options and arguments, run `configure` with the `-h` option:
```
  $ ./configure -h
```
The output from this invocation of `configure` should give you an up-to-date list of options and their descriptions.


## Step 3: Compilation

Once `configure` is finished, you are ready to instantiate (compile) BLIS into a library by running `make`. Running `make` will result in output similar to:
```
$ make
Generating monolithic blis.h.........................................................
.....................................................................................
.....................................................................................
.....................................................................................
.....................................................................................
..........................................
Generated include/haswell/blis.h
Compiling obj/haswell/config/haswell/bli_cntx_init_haswell.o ('haswell' CFLAGS for config code)
Compiling obj/haswell/kernels/zen/1/bli_amaxv_zen_int.o ('haswell' CFLAGS for kernels)
Compiling obj/haswell/kernels/zen/1/bli_axpyv_zen_int.o ('haswell' CFLAGS for kernels)
Compiling obj/haswell/kernels/zen/1/bli_axpyv_zen_int10.o ('haswell' CFLAGS for kernels)
Compiling obj/haswell/kernels/zen/1/bli_dotv_zen_int.o ('haswell' CFLAGS for kernels)
Compiling obj/haswell/kernels/zen/1/bli_dotv_zen_int10.o ('haswell' CFLAGS for kernels)
```
If you want to see the individual command line invocations of the compiler, you can run `make` as follows:
```
$ make V=1
```
Also, if you are compiling on a multicore system, you can get parallelism via:
```
$ make -j<n>
```
where `<n>` is the number of jobs `make` is allowed to run simultaneously. Generally, you should limit `<n>` to p+1, where p is the number of processor cores on your system.

### Running into the ARG_MAX limit

On some systems, you may observe an error message when the build system attempts to archive BLIS object files into the static library (or perhaps when the linker attempts to generate the shared library):
```
Archiving lib/x86_64/libblis.a
bash: ar: Argument list too long
Makefile:584: recipe for target 'lib/x86_64/libblis.a' failed
make: *** [lib/x86_64/libblis.a] Error 126
```
This error message results when the user attempts to execute a program with too many arguments (or more specifically, a program-argument string that occupies too many bytes)--that is, when the command exceeds the [ARG_MAX limit](https://www.in-ulm.de/~mascheck/various/argmax/). This doesn't occur very often, but if it does, don't worry--we have a workaround. Simply rerun `configure` as you did previously, except this time include an addition option: `--enable-arg-max-hack`. You will see confirmation that the option was accepted as configure runs:
```
configure: enabling ARG_MAX hack.
```
The archiver and/or linker should no longer choke when creating the libraries.

## Step 3b: Testing (optional)

If you would like to run some ready-made tests that exercise BLIS in a number of ways, including through its BLAS compatibility layer, run `make check`:
```
$ make check
```
Watch the output near the end. You should see the following messages, though not necessarily in immediate succession:
```
All BLIS tests passed!
All BLAS tests passed!
```
Please see the [Testsuite](Testsuite.md) document for more details on running either the BLIS testsuite or the BLAS test drivers. If you have any trouble, please report your problem to BLIS developers by opening a [new issue](https://github.com/flame/blis/issues/).


## Step 4: Installation

Toward the end of compilation, you should get output similar to:
```
Compiling obj/haswell/frame/thread/bli_thread.o ('haswell' CFLAGS for framework code)
Compiling obj/haswell/frame/thread/bli_thrinfo.o ('haswell' CFLAGS for framework code)
Compiling obj/haswell/frame/util/bli_util_check.o ('haswell' CFLAGS for framework code)
Compiling obj/haswell/frame/util/bli_util_oapi.o ('haswell' CFLAGS for framework code)
Compiling obj/haswell/frame/util/bli_util_oapi_wc.o ('haswell' CFLAGS for framework code)
Compiling obj/haswell/frame/util/bli_util_oapi_woc.o ('haswell' CFLAGS for framework code)
Compiling obj/haswell/frame/util/bli_util_tapi.o ('haswell' CFLAGS for framework code)
Compiling obj/haswell/frame/util/bli_util_unb_var1.o ('haswell' CFLAGS for framework code)
Archiving lib/haswell/libblis.a
Dynamically linking lib/haswell/libblis.so
```
Now you have a BLIS library (in static and shared forms) residing in the `lib/<configname>/` directory. To install the libraries and the header files associated with it, simply execute:
```
$ make install
```
This installs copies of the libraries and header files, and also creates conventional symbolic links of shared libraries:
```
Installing libblis.a into /u/field/blis/lib/
Installing libblis.so.0.0.0 into /u/field/blis/lib/
Installing symlink libblis.so into /u/field/blis/lib/
Installing symlink libblis.so.0 into /u/field/blis/lib/
Installing blis.h into /u/field/blis/include/blis/
```
This results in your `PREFIX` directory looking like:
```
# Check the contents of 'PREFIX'.
$ ls -l $HOME/blis
drwxr-xr-x 3 field dept 4096 May 10 17:36 include
drwxr-xr-x 2 field dept 4096 May 10 17:42 lib
# Check the contents of 'PREFIX/include'.
$ ls -l $HOME/blis/include
drwxr-xr-x 2 field dept 4096 May 10 17:42 blis
$ ls -l $HOME/blis/include/blis
-rw-r--r-- 1 field dept 915324 May 10 17:42 blis.h
# Check the contents of 'PREFIX/lib'.
$ ls -l $HOME/blis/lib
-rw-r--r-- 1 field dept 2979052 May 10 17:42 libblis.a
lrwxrwxrwx 1 field dept      16 May 10 17:42 libblis.so -> libblis.so.0.0.0
lrwxrwxrwx 1 field dept      16 May 10 17:42 libblis.so.0 -> libblis.so.0.0.0
-rw-r--r-- 1 field dept 2185976 May 10 17:42 libblis.so.0.0.0
```

## Cleaning out build products

If you want to remove various build products, you can use one of the `make` targets already defined for you in the BLIS Makefile:
```
$ make clean
Removing flattened header files from ./include/haswell.
Removing object files from ./obj/haswell.
Removing libraries from ./lib/haswell.
```
Executing the `clean` target will remove all binary object files and library builds from the `obj` and `lib` directories, as well as any flattened header files. Any other configurations' build products are left untouched.
```
$ make cleanmk
Removing makefile fragments from ./config.
Removing makefile fragments from ./frame.
Removing makefile fragments from ./ref_kernels.
Removing makefile fragments from ./kernels.
```
The `cleanmk` target results in removal of all makefile fragments from the framework source tree. (Makefile fragments are named `.fragment.mk` and are generated at configure-time.)
```
$ make distclean
Removing makefile fragments from ./config.
Removing makefile fragments from ./frame.
Removing makefile fragments from ./ref_kernels.
Removing makefile fragments from ./kernels.
Removing flattened header files from ./include/haswell.
Removing object files from ./obj/haswell.
Removing libraries from ./lib/haswell.
Removing object files from ./obj/haswell/blastest.
Removing libf2c.a from ./obj/haswell/blastest.
Removing binaries from ./obj/haswell/blastest.
Removing driver output files 'out.*'.
Removing object files from ./blastest/obj.
Removing libf2c.a from ./blastest.
Removing binaries from ./blastest.
Removing driver output files 'out.*' from ./blastest.
Removing object files from ./obj/haswell/testsuite.
Removing binary test_libblis.x.
Removing output.testsuite.
Removing object files from testsuite/obj.
Removing binary testsuite/test_libblis.x.
Removing ./bli_config.h.
Removing config.mk.
Removing obj.
Removing lib.
Removing include.
```
Running the `distclean` target is like saying, "Remove anything ever created by the build system."


## Linking against BLIS

Once you have instantiated (configured, compiled, and installed) a BLIS library, you can link to it in your application's makefile as you would any other library. The following is an abbreviated makefile for a small hypothetical application that has just two external dependencies: BLIS and the standard C math library.
```make
BLIS_PREFIX = $(HOME)/blis
BLIS_INC    = $(BLIS_PREFIX)/include/blis
BLIS_LIB    = $(BLIS_PREFIX)/lib/libblis.a

OTHER_LIBS  = -L/usr/lib -lm

CC          = gcc
CFLAGS      = -O2 -g -I$(BLIS_INC)
LINKER      = $(CC)

OBJS        = main.o util.o other.o

%.o: %.c
    $(CC) $(CFLAGS) -c $< -o $@

all: $(OBJS) 
    $(LINKER) $(OBJS) $(BLIS_LIB) $(OTHER_LIBS) -o my_program.x
```
The above example assumes you will want to include BLIS definitions and function prototypes into your application via `#include blis.h`. (If you are only using the BLIS via the BLAS compatibility layer, including `blis.h` is not necessary.) Since BLIS headers are installed into a `blis` subdirectory of `PREFIX/include`, you must make sure that the compiler knows where to find the `blis.h` header file. This is typically accomplished by inserting `#include "blis.h"` into your application's source code files and compiling the code with `-I PREFIX/include/blis`.

The makefile shown above a very simple example. If you need help linking your application to your BLIS library, please [open an issue](https://github.com/flame/blis/issues).


## Uninstalling

If you decide that you want to uninstall BLIS, simply run `make uninstall`
```
$ make uninstall
Uninstalling libraries libblis.a libblis.so.0.0.0 from /u/field/blis/lib/.
Uninstalling symlinks libblis.so libblis.so.0 from /u/field/blis/lib/.
Uninstalling directory 'blis' from /u/field/blis/include/.
```
This removes the libraries, symlinks, and header directory that was installed by `make install`. Before running `make uninstall`, however, make sure that BLIS is configured the with the same `LIBDIR` and `INCDIR` paths used during installation.



## Conclusion

If you have feedback, please consider keeping in touch with the project maintainers, contributors, and other users by joining and posting to the [BLIS mailing lists](https://github.com/flame/blis#discussion).

Thanks for using BLIS!
