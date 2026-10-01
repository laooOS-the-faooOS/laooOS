# laooOS — Building an Experimental Linux System (GCC PROFILE)

### 1.0 Host Compiler

> **NOTE:** laooOS can be bootstrapped from either **GCC** or **LLVM/Clang** on the host.
>
> The host compiler is only used to start the bootstrap process. It does not determine which compiler the final laooOS system uses.
>
> The two main paths are:
>
> ```text
> GCC host
>     ↓
> GCC bridge
>     ↓
> GCC native
> ```
>
> or:
>
> ```text
> Clang host
>     ↓
> Clang bridge
>     ↓
> Clang native
> ```
>
> A GCC host can therefore build the LLVM-based laooOS profile, and a Clang host can be used to bootstrap the GCC-based profile, provided the required target toolchain is configured correctly.
>
> For the GCC laooOS profile, GCC is ultimately used as the native compiler.
>
> For the Mainstream laooOS profile, LLVM/Clang is ultimately used as the native compiler.
>
> The host compiler is simply the first tool used to cross the gap from the existing Linux system to the laooOS toolchain.

> An experimental Linux system built from source.
>
> Keep it small. Make it fast. Break things. Fix them. Repeat.

> **NOTE — Using distcc**
>
> If you have another machine available, this is a good point to use **distcc** to speed up the native toolchain build.
>
> Distcc can distribute compilation jobs to other machines while the main machine handles the build.
>
> Make sure the remote machines use a compatible compiler, target architecture, headers, and build environment. Distributed compilation does **not** replace the local linker or target sysroot, so the target environment still needs to be correctly configured on the main build machine.
>
> For example:
>
> ```text
> --------------------------------
> $ export DISTCC_HOSTS="localhost 192.168.1.20 192.168.1.21"
> --------------------------------
> ```
>
> Then use distcc as the compiler wrapper:
>
> ```text
> --------------------------------
> $ export CC="distcc gcc"
> $ export CXX="distcc g++"
> --------------------------------
> ```
>
> The exact setup depends on the machines being used. If the remote machines are not configured for the same target environment, do not distribute the build.
>
> **For laooOS, distcc is optional.** The native toolchain can be built entirely on the main machine.

::


laooOS is an experimental Linux system built from source.

The project is not trying to be another giant Linux distribution.

Instead, it is a place to experiment with:

* Linux
* compilers
* libc
* linkers
* optimization
* system design
* minimal userspace
* building everything from source

The build process is part of the project.

Things will break.

That's fine.

Fix them, learn from them, and keep going.

---

# I. Introduction

## 1. What is laooOS?

laooOS is an experimental Linux system built from source.

The idea is simple:

```text
Build the system ourselves.
Keep it small.
Make it fast.
Experiment with the toolchain.
Remove things we don't need.
```

laooOS isn't trying to follow the traditional Linux distribution model.

It's more like a playground for experimenting with the entire system, from the compiler and linker all the way to userspace.

The project has several different directions instead of one fixed "correct" way to build the system.

The three main ones are:

* **Mainstream laooOS**
* **GCC laooOS**
* **Tiny laooOS**

---

## 2. The Three laooOSes

### 2.1 Mainstream laooOS

This is the LLVM/Clang version of laooOS.

The idea is to use LLVM as the main compiler environment and see how far we can push a small, modern Linux system with it.

```text
LLVM/Clang + Musl + Mold
```

This version is focused on experimenting with:

* Clang
* LLVM tools
* modern compiler features
* aggressive optimization
* custom toolchain layouts
* a lightweight Musl system

It doesn't have to follow normal distribution conventions.

---

### 2.2 GCC laooOS

GCC laooOS takes the same idea in a different direction.

Instead of making LLVM the center of the system, GCC becomes the main compiler.

```text
GCC + Musl + Mold
```

This version is an experiment in building a small and fast Musl system around GCC.

The goal is to see how far we can push a GCC-based system while keeping the rest of the system lightweight.

---

### 2.3 Tiny laooOS

Tiny laooOS is where we start removing things.

The question is basically:

> "How little do we actually need?"

Tiny laooOS is an experiment in reducing the system down to the important parts.

The goals are:

```text
less software
less dependencies
less disk space
less memory usage
less background stuff
```

while still keeping a useful Linux environment.

Tiny laooOS can also act as a base for other experiments.

---

## 3. One Thing All laooOS Builds Have in Common

**Mold.**

Mold is the main linker of laooOS.

The linker is normally something most users never think about.

When building an entire operating system from source, however, it gets used constantly.

The basic idea is:

```text
compiler
   |
   v
object files
   |
   v
  mold
   |
   v
program
```

The compiler can change between laooOS profiles.

The linker philosophy stays the same.

---

## 4. Why Mold?

Because waiting for the linker is boring.

laooOS is an experimental project, so build speed matters.

When rebuilding hundreds or thousands of files, the linker gets used again and again.

Mold gives the project a fast linker without forcing the entire system to use one particular compiler.

This means both the LLVM and GCC versions can share the same basic linker setup.

---

## 5. Optimization

Optimization is another major part of laooOS.

We don't just want:

```text
"it compiles"
```

We want to experiment with:

```text
"how far can we push it?"
```

Depending on the package and profile, this can include:

* `-O3`
* `-Ofast`
* CPU-specific tuning
* vectorization
* loop optimization
* aggressive inlining
* interprocedural optimization
* linker optimization
* LTO
* PGO
* size optimization
* reduced runtime overhead

Not every flag belongs everywhere.

Sometimes an aggressive optimization makes a build slower, larger, or less reliable.

That's part of the experiment.

laooOS is about testing these things instead of blindly copying a distribution's default flags.

---

## 6. Musl

The main libc direction of laooOS is **Musl**.

Musl fits the project because it is small, simple, and portable.

It also makes the toolchain experiments more interesting.

Instead of only building around the traditional:

```text
GCC + glibc
```

combination, laooOS experiments with:

```text
GCC   + Musl
Clang + Musl
```

and eventually builds the system around the toolchain created during the project.

---

## 7. Build It Yourself

Most Linux distributions give you a finished system.

laooOS starts with the opposite idea:

> What happens if we build the system ourselves?

The general progression is:

```text
Host
  |
  v
Bootstrap
  |
  v
Compiler
  |
  v
libc
  |
  v
Libraries
  |
  v
Userspace
  |
  v
laooOS
```

The host is only the starting point.

As the build progresses, more of the final system is produced using the new toolchain.

---

## 8. Experimental by Design

laooOS is allowed to change.

There isn't one sacred architecture that can never be touched.

A build might use GCC today and Clang tomorrow.

A package might be removed.

A library might be replaced.

A compiler flag might make the system faster.

Or it might completely break the build.

That's fine.

The project is experimental.

The build process itself is part of the project.

---

## 9. The Three Directions

The three profiles are basically three different questions.

### Mainstream laooOS

> "What can we do with a modern LLVM-based system?"

### GCC laooOS

> "What can we do with GCC and Musl?"

### Tiny laooOS

> "How little can we get away with?"

They share the same basic spirit, but they don't have to end up identical.

---

## 10. The laooOS Idea

laooOS is basically an experiment:

```text
Build Linux ourselves.
Pick our own tools.
Remove what we don't need.
Optimize what we keep.
Break things.
Fix them.
Repeat.
```

There is no requirement for laooOS to look like a normal distribution.

If something works better, we try it.

If something is unnecessary, we remove it.

If something interesting happens, we keep experimenting with it.

That's laooOS.

---

# II. Before the Build

Before we build laooOS, we need a clean host and a clean build environment.

The host is only used to bootstrap the system.

The goal is to eventually build laooOS using its own toolchain.

---

## 2. Preparing the Host

A 64-bit Linux host is required.

The distribution does not matter much.

It only needs the tools required to build the initial system.

### 2.1 Host system requirements

The host should have:

```text
x86_64 CPU
Working compiler
Binutils
Make
Bash
Python
Basic Unix utilities
Internet access
Enough disk space
Enough RAM for large builds
```

---

### 2.2 Required packages

Install the basic build tools provided by your distribution.

Common requirements include:

```text
gcc
binutils
make
cmake
ninja
python
perl
pkgconf
git
tar
xz
gzip
bzip2
patch
diffutils
coreutils
findutils
sed
awk
grep
aria2c
```

Some tools are only needed during specific parts of the build.

The final laooOS system does not have to contain all of them.

---

### 2.3 Disk space

Source builds require considerably more space than the final system.

GCC and LLVM can use a particularly large amount of storage during compilation.

laooOS keeps the build under:

```text
/mnt/lfs
```

Create the basic directories:

```text
--------------------------------
$ mkdir -pv /mnt/lfs
$ mkdir -pv /mnt/lfs/sources
--------------------------------
```

---

### 2.4 Memory requirements

Large packages can consume a lot of memory when built in parallel.

Check the system:

```text
--------------------------------
$ nproc
$ free -h
--------------------------------
```

For example:

```text
--------------------------------
$ make -j8
--------------------------------
```

If the machine starts swapping heavily, reduce the number of parallel jobs.

---

### 2.5 CPU considerations

laooOS currently targets x86_64.

CPU-specific optimization is applied later in the build.

The bootstrap environment should remain stable first.

Once the compiler works, more aggressive optimization can be introduced.

---

# 3. Creating the Build Environment

The main build directory is:

```text
/mnt/lfs
```

Set `LFS` to point to it:

```text
--------------------------------
$ export LFS=/mnt/lfs
$ echo $LFS
/mnt/lfs
--------------------------------
```

---

## 3.1 Creating $LFS

Create the root of the new system:

```text
--------------------------------
$ mkdir -pv "$LFS"
--------------------------------
```

---

## 3.2 Creating the sources directory

All downloads and source trees are kept here:

```text
--------------------------------
$ mkdir -pv "$LFS/sources"
--------------------------------
```

The source tree should remain inside `$LFS/sources`.

This keeps the build organized and makes it easier to restart individual packages.

---

## 3.3 Creating the build user

Building as root is avoided whenever possible.

Create a dedicated build group and user:

```text
--------------------------------
# groupadd lfs
# useradd -s /bin/bash -g lfs -m -k /dev/null lfs
# passwd lfs
--------------------------------
```

---

## 3.4 Setting ownership

Give the build user ownership of the build directory:

```text
--------------------------------
# chown -R lfs:lfs "$LFS"
--------------------------------
```

---

## 3.5 Setting environment variables

Switch to the build user:

```text
--------------------------------
# su - lfs
--------------------------------
```

Set the main build variables:

```text
--------------------------------
$ export LFS=/mnt/lfs
$ export LC_ALL=C
$ export LFS_TGT=x86_64-laoo-linux-musl
--------------------------------
```

The target triplet may change depending on the toolchain being built.

Keep the build environment explicit.

---

## 3.6 Setting PATH

The temporary laooOS tools will be installed under `$LFS/tools`.

Put them before the host tools:

```text
--------------------------------
$ mkdir -pv "$LFS/tools"
$ export PATH="$LFS/tools/bin:$PATH"
--------------------------------
```

Check the result:

```text
--------------------------------
$ echo "$PATH"
--------------------------------
```

---

## 3.7 Build flags

laooOS is an experimental optimization project.

The exact flags can change between packages, but the general direction is aggressive optimization.

A starting environment can look like:

```text
--------------------------------
$ export CFLAGS="-O3 -pipe"
$ export CXXFLAGS="$CFLAGS"
$ export LDFLAGS="-fuse-ld=mold"
--------------------------------
```

Later stages can experiment with stronger options such as:

```text
-Ofast
-march=
-mtune=
-ftree-vectorize
-funroll-loops
-flto
```

Do not assume that every aggressive flag improves every package.

---

# 4. Getting the Sources

All source archives belong in:

```text
$LFS/sources
```

Enter the directory:

```text
--------------------------------
$ cd "$LFS/sources"
--------------------------------
```

---

## 4.1 Source mirrors

Sources should come from the upstream project or a trusted mirror.

Keep the original archives.

Do not modify downloaded source archives directly.

---

## 4.2 Downloading with aria2c

laooOS uses `aria2c` for fast and resumable downloads.

A typical configuration is:

```text
--------------------------------
$ aria2c -x15 -s15 -k1M -c
--------------------------------
```

For multiple packages, put the URLs into an input file.

```text
--------------------------------
$ joe sources.list
--------------------------------
```

Then download everything:

```text
--------------------------------
$ aria2c -x15 -s15 -k1M -c -i sources.list
--------------------------------
```

This keeps the download command short and makes the source list easy to reuse.

---

## 4.3 Source checksums

Verify downloaded archives before extracting them.

For SHA-256:

```text
--------------------------------
$ sha256sum package-version.tar.xz
--------------------------------
```

Compare the result with the checksum published by the upstream project.

A different checksum means the archive should not be trusted until the difference is understood.

---

## 4.4 Extracting sources

Extract packages directly inside `$LFS/sources`:

```text
--------------------------------
$ cd "$LFS/sources"
$ tar -xf package-version.tar.xz
--------------------------------
```

This should produce a normal source directory:

```text
--------------------------------
$ ls
package-version
package-version.tar.xz
--------------------------------
```

---

## 4.5 Keeping the source tree clean

Keep source archives and source trees inside `$LFS/sources`.

Do not scatter build trees around the host.

When a package supports an out-of-tree build, keep the build directory inside the package's source area:

```text
--------------------------------
$ mkdir -pv package-version/build
$ cd package-version/build
--------------------------------
```

If a package needs to be rebuilt from a clean source tree, remove the extracted directory and extract the original archive again.

The goal is simple:

```text
sources stay in sources
tools stay in tools
the host stays the host
laooOS stays inside $LFS
```

---

# III. Building the Bootstrap System

# 5. The Bootstrap Toolchain

---

The bootstrap toolchain is the first toolchain built for laooOS.

Its purpose is to create a compiler that:

```text
runs on:
    host Linux / glibc

produces binaries for:
    x86_64-laoo-linux-musl
```

The bridge is temporary. It gets us from the host environment to a Musl-based, increasingly self-hosting laooOS environment.

The bootstrap order is:

```text
Linux kernel headers
        ↓
      Musl
        ↓
    Binutils
     (bridge)
        ↓
      GCC
     (bridge)
        ↓
    libstdc++
     (bridge)
```

---

## 5.1 Linux Kernel Headers

The kernel headers provide the userspace API exposed by Linux.

Download the current stable Linux source into `$LFS/sources` and extract it:

```text
--------------------------------
$ cd "$LFS/sources"
$ tar -xf linux-<version>.tar.xz
$ cd linux-<version>
--------------------------------
```

Clean the source tree:

```text
--------------------------------
$ make mrproper
--------------------------------
```

Install the exported userspace headers:

```text
--------------------------------
$ make headers_install INSTALL_HDR_PATH="$LFS/usr"
--------------------------------
```

This installs the headers into:

```text
$LFS/usr/include
```

We do not build the kernel yet.

---

## 5.2 Musl

Musl is the target C library.

The bridge compiler will eventually use these headers, startup files, and libraries when producing target programs.

Enter the Musl source tree:

```text
--------------------------------
$ cd "$LFS/sources/musl-<version>"
--------------------------------
```

Configure Musl:

```text
--------------------------------
$ ./configure --prefix=/usr --target="$LFS_TGT"
--------------------------------
```

### Configure flags

`--prefix=/usr`

Installs Musl as if `/usr` is the root of the target system.

Because we use `DESTDIR="$LFS"` during installation, the actual installation goes into:

```text
$LFS/usr
```

`--target="$LFS_TGT"`

Selects the target architecture and ABI.

For laooOS:

```text
x86_64-laoo-linux-musl
```

The exact target triplet is less important than the fact that it describes our x86_64 Musl target.

Build and install:

```text
--------------------------------
$ make -j"$(nproc)"
$ make DESTDIR="$LFS" install
--------------------------------
```

The target dynamic linker will be installed as:

```text
/lib/ld-musl-x86_64.so.1
```

---

## 5.3 Binutils Bridge

Binutils provides the assembler, linker and object-file utilities needed by GCC.

Create a separate build directory:

```text
--------------------------------
$ cd "$LFS/sources/binutils-<version>"
$ mkdir -pv build-bridge
$ cd build-bridge
--------------------------------
```

Configure:

```text
--------------------------------
$ ../configure --prefix="$LFS/tools" --target="$LFS_TGT" --with-sysroot="$LFS" --disable-nls --disable-werror --disable-gprofng
--------------------------------
```

### Configure flags

`--prefix="$LFS/tools"`

Installs the bridge tools into the temporary toolchain directory:

```text
$LFS/tools
```

This keeps bootstrap tools separate from both the host and final system.

`--target="$LFS_TGT"`

Tells Binutils that its target is:

```text
x86_64-laoo-linux-musl
```

This produces tools such as:

```text
x86_64-laoo-linux-musl-as
x86_64-laoo-linux-musl-ld
x86_64-laoo-linux-musl-ar
```

These programs still **run on the host**.

`--with-sysroot="$LFS"`

Tells the target tools that:

```text
$LFS
```

is the root of the target filesystem.

When GCC asks the linker to find target headers or libraries, the tools can therefore search inside:

```text
$LFS/usr/include
$LFS/usr/lib
$LFS/lib
```

instead of accidentally using host libraries.

This is one of the most important options in the bridge.

`--disable-nls`

Disables Native Language Support in Binutils.

This removes unnecessary translation infrastructure from the bootstrap tools and keeps the bootstrap smaller.

`--disable-werror`

Prevents warnings from being treated as fatal errors.

Bootstrap builds should be tolerant of compiler warnings because the host compiler may differ from the compiler expected by the upstream project.

`--disable-gprofng`

Disables Gprofng, the GNU profiling component of Binutils.

The bootstrap linker/assembler does not need it.

Build and install:

```text
--------------------------------
$ make -j"$(nproc)"
$ make install
--------------------------------
```

---

## 5.4 GCC Bridge

Now we build GCC.

The important concept is:

```text
GCC executable:
    host / glibc

GCC output:
    x86_64-laoo-linux-musl
```

Create the build directory:

```text
--------------------------------
$ cd "$LFS/sources/gcc-<version>"
$ mkdir -pv build-bridge
$ cd build-bridge
--------------------------------
```

Configure the first C compiler:

```text
--------------------------------
$ ../configure --target="$LFS_TGT" --prefix="$LFS/tools" --with-sysroot="$LFS" --disable-nls --disable-shared --disable-multilib --disable-decimal-float --disable-threads --disable-libatomic --disable-libgomp --disable-libquadmath --disable-libssp --disable-libvtv --disable-libsanitizer --disable-libstdcxx --enable-languages=c
--------------------------------
```

### Configure flags

`--target="$LFS_TGT"`

Sets the compiler's target:

```text
x86_64-laoo-linux-musl
```

The resulting compiler is therefore a cross compiler rather than a compiler for the host's normal glibc environment.

`--prefix="$LFS/tools"`

Installs the temporary GCC into:

```text
$LFS/tools
```

The bridge compiler is intentionally kept separate from the eventual compiler installed in the final system.

`--with-sysroot="$LFS"`

Makes `$LFS` the target filesystem root.

This is what lets GCC find the Musl environment we have just created.

Instead of accidentally finding:

```text
/usr/include
/usr/lib
```

from the host, target searches can resolve inside:

```text
$LFS/usr/include
$LFS/usr/lib
```

`--disable-nls`

Disables GCC's translation infrastructure.

The compiler does not need localized diagnostic messages during bootstrap.

`--disable-shared`

Avoids building shared GCC components for the bootstrap compiler.

The bridge only needs enough compiler functionality to continue constructing the target system.

`--disable-multilib`

Builds only the primary target architecture.

For our x86_64 build, we do not need additional 32-bit or alternate ABI libraries.

This reduces build time and complexity.

`--disable-decimal-float`

Disables GCC's decimal floating-point support.

This is not required for the initial C bootstrap compiler.

`--disable-threads`

Disables GCC thread runtime support during the initial bootstrap.

Thread support can be added when building the proper target compiler.

`--disable-libatomic`

Does not build GCC's `libatomic` runtime during the minimal bootstrap.

`--disable-libgomp`

Does not build the OpenMP runtime.

OpenMP is not required to create the initial target compiler.

`--disable-libquadmath`

Does not build GCC's extended-precision floating-point runtime.

`--disable-libssp`

Does not build GCC's stack-smashing-protection runtime library as a separate bootstrap component.

`--disable-libvtv`

Disables GCC's virtual table verification runtime.

It is not needed for the initial C compiler.

`--disable-libsanitizer`

Disables GCC's sanitizer runtimes.

AddressSanitizer, UndefinedBehaviorSanitizer and related runtimes are useful later, but they are unnecessary for bootstrapping.

`--disable-libstdcxx`

Do not build the C++ standard library as part of the initial compiler stage.

We build target libstdc++ separately once the C compiler and Musl environment are working.

`--enable-languages=c`

Build only the C compiler.

C is enough to bootstrap the initial target environment.

This keeps the first GCC build significantly smaller than building the complete GCC language suite.

---

## 5.5 Building GCC

Build the bridge:

```text
--------------------------------
$ make -j"$(nproc)"
$ make install
--------------------------------
```

Check the compiler:

```text
--------------------------------
$ "$LFS/tools/bin/$LFS_TGT-gcc" --version
--------------------------------
```

Check its target:

```text
--------------------------------
$ "$LFS/tools/bin/$LFS_TGT-gcc" -v
--------------------------------
```

The important result is that GCC reports:

```text
Target: x86_64-laoo-linux-musl
```

The compiler itself is still a host executable.

That is exactly what we want at this stage.

---

## 5.6 libstdc++ Bridge

Once the bridge C compiler exists, we can build the C++ standard library for the target.

The build process still executes on the host, but the resulting library belongs to:

```text
x86_64-laoo-linux-musl
```

From the GCC build directory:

```text
--------------------------------
$ cd "$LFS/sources/gcc-<version>/build-bridge"
$ make -j"$(nproc)" all-target-libstdc++-v3
--------------------------------
```

Install it:

```text
--------------------------------
$ make install-target-libstdc++-v3
--------------------------------
```

The important distinction is:

```text
host
 └── executes the build

bridge GCC
 └── runs on host
 └── generates Musl target code

libstdc++
 └── is target code
 └── belongs to x86_64-laoo-linux-musl
```

---

## 5.7 What the Bridge Gives Us

After these stages, we have the basic bootstrap environment:

```text
                    HOST
             Linux + glibc
                    │
                    │ executes
                    ▼
             ┌──────────────┐
             │   Binutils   │
             │    bridge    │
             └──────┬───────┘
                    │
                    ▼
             ┌──────────────┐
             │  GCC bridge  │
             │     C        │
             └──────┬───────┘
                    │
                    ▼
              Musl target
                    │
                    ▼
             target libstdc++
```

The bridge has two different identities:

```text
Execution environment:
    host / glibc

Compilation environment:
    x86_64-laoo-linux-musl
```

That distinction is the entire point of this bootstrap stage.

The host gives us something that can execute.

The bridge gives us something that can **build for Musl**.

Later stages progressively replace the host dependencies until laooOS can build itself.


---

## 6. The native toolchain

---

The bootstrap stage gave us a working bridge:

```text
host / glibc
      │
      ▼
bridge GCC + Binutils
      │
      ▼
x86_64-laoo-linux-musl
```

The bridge is not the final toolchain.

It exists so that we can use host-executable tools to build the actual Musl toolchain.

This stage turns the bridge into the target toolchain.

---

## 6.1 Host Compiler

The host compiler is the compiler provided by the system used to build laooOS.

For example:

```text
host:
    x86_64
    Linux
    glibc
    host GCC
```

The host compiler builds the initial bootstrap components.

It is not intended to become the final laooOS compiler.

The important distinction is:

```text
host compiler
    ↓
builds bridge tools

bridge tools
    ↓
build target toolchain
```

The host is therefore only the starting point.

---

## 6.2 Target Compiler

The target compiler is the compiler that produces binaries for laooOS.

Our target is:

```text
x86_64-laoo-linux-musl
```

During bootstrap, the first target compiler is the bridge GCC.

It runs on the host:

```text
host / glibc
```

but produces programs for:

```text
x86_64 / Linux / Musl
```

We then use that bridge compiler to build the real target GCC.

The progression is:

```text
host GCC
    ↓
bridge GCC
    ↓
native Musl GCC
```

The final compiler will support:

```text
C
C++
```

and will be used to build the rest of laooOS.

---

## 6.3 Target Architecture

laooOS currently targets:

```text
x86_64
```

The target therefore produces 64-bit x86 Linux binaries.

The host and target can use the same CPU architecture while using completely different C libraries:

```text
HOST

x86_64
Linux
glibc
```

```text
TARGET

x86_64
Linux
Musl
```

The architecture does not change during the bootstrap.

The libc and toolchain environment do.

---

## 6.4 Target Triplet

The laooOS target triplet is:

```text
x86_64-laoo-linux-musl
```

The components describe the target:

```text
x86_64
    target architecture

laoo
    laooOS identifier

linux
    kernel/system environment

musl
    target libc
```

The triplet is used by the bootstrap compiler and Binutils to identify the target.

For example:

```text
x86_64-laoo-linux-musl-gcc
x86_64-laoo-linux-musl-g++
x86_64-laoo-linux-musl-ld
x86_64-laoo-linux-musl-as
```

The target triplet keeps the host and target environments separated.

---

## 6.5 Glibc Host / Musl Target

The bootstrap crosses a libc boundary.

The host uses glibc:

```text
host
  │
  └── glibc
```

The target uses Musl:

```text
target
  │
  └── Musl
```

The bridge connects them:

```text
                 HOST
              glibc system
                  │
                  │
             bridge tools
                  │
                  ▼
             MUSL TARGET
```

The bridge tools themselves are host executables.

Their output is target code.

This means we can build the Musl system before we have a complete Musl system capable of building itself.

That is the purpose of the bootstrap.

---

## 6.6 Entering the Target Environment

Now we use the bridge to build the actual target toolchain.

The order is deliberately simple:

```text
bridge Binutils
      ↓
Binutils for Musl
      ↓
GCC for Musl
 C + C++
      ↓
rebuild Musl
      ↓
native Musl toolchain
```

The bridge is used to build each component.

---

### 6.6.1 Binutils — Musl Target

Start with Binutils.

The bridge compiler and bridge linker are used to build the target-side Binutils.

Create a new build directory:

```text
--------------------------------
$ cd "$LFS/sources/binutils-<version>"
$ mkdir -pv build-musl
$ cd build-musl
--------------------------------
```

Configure:

```text
--------------------------------
$ ../configure --prefix=/usr --target="$LFS_TGT" --with-sysroot=/ --disable-nls --disable-werror --disable-gprofng
--------------------------------
```

The important options are:

`--prefix=/usr`

Install Binutils into the normal target prefix:

```text
/usr
```

With:

```text
DESTDIR="$LFS"
```

the actual installation goes into:

```text
$LFS/usr
```

This is different from the bootstrap Binutils, which lived under:

```text
$LFS/tools
```

`--target="$LFS_TGT"`

Sets the target to:

```text
x86_64-laoo-linux-musl
```

The resulting tools understand the Musl target.

`--with-sysroot=/`

Sets the target sysroot to `/`.

Inside the target filesystem, `/` is the root of the actual laooOS environment.

The toolchain therefore searches locations such as:

```text
/usr/include
/usr/lib
/lib
```

within the target environment.

`--disable-nls`

Disables localization support.

The bootstrap toolchain does not need translated Binutils messages.

`--disable-werror`

Prevents warnings from being treated as fatal errors.

This makes the bootstrap less dependent on quirks of the compiler used to build it.

`--disable-gprofng`

Disables Gprofng.

Profiling tools are not required for the core target toolchain.

Build and install:

```text
--------------------------------
$ make -j"$(nproc)"
$ make DESTDIR="$LFS" install
--------------------------------
```

We now have target Binutils installed under:

```text
$LFS/usr
```

---

### 6.6.2 GCC — Musl Target

With target Binutils available, we can build the real GCC.

This GCC supports both:

```text
C
C++
```

Create the build directory:

```text
--------------------------------
$ cd "$LFS/sources/gcc-<version>"
$ mkdir -pv build-musl
$ cd build-musl
--------------------------------
```

Configure:

```text
--------------------------------
$ ../configure --prefix=/usr --target="$LFS_TGT" --with-sysroot=/ --disable-nls --disable-multilib --disable-werror --enable-languages=c,c++
--------------------------------
```

### GCC configure flags

`--prefix=/usr`

Installs GCC into the normal target prefix.

With `DESTDIR="$LFS"`:

```text
$LFS/usr
```

`--target="$LFS_TGT"`

Builds GCC for:

```text
x86_64-laoo-linux-musl
```

`--with-sysroot=/`

Tells GCC that the target filesystem root is `/`.

The compiler will therefore look for target headers and libraries relative to the target root.

`--disable-nls`

Disables GCC localization support.

`--disable-multilib`

Builds only the x86_64 target.

We do not create additional 32-bit or alternate ABI compiler libraries.

`--disable-werror`

Prevents compiler warnings from aborting the bootstrap build.

`--enable-languages=c,c++`

Enables the two languages required for the normal laooOS compiler:

```text
C
C++
```

Unlike the bridge compiler, this is no longer a C-only bootstrap compiler.

Build and install:

```text
--------------------------------
$ make -j"$(nproc)"
$ make DESTDIR="$LFS" install
--------------------------------
```

Check the compiler:

```text
--------------------------------
$ "$LFS/usr/bin/$LFS_TGT-gcc" --version
$ "$LFS/usr/bin/$LFS_TGT-g++" --version
--------------------------------
```

The compiler now belongs to the target toolchain:

```text
GCC
 ├── C
 └── C++
       │
       ▼
x86_64-laoo-linux-musl
```

---

### 6.6.3 Rebuild Musl

The original Musl installation was required to bootstrap the bridge.

Now that the target Binutils and GCC exist, Musl can be rebuilt using the new target toolchain.

This replaces the initial bootstrap libc with the version built by the new toolchain.

Enter the Musl source tree:

```text
--------------------------------
$ cd "$LFS/sources/musl-<version>"
--------------------------------
```

Clean the previous build:

```text
--------------------------------
$ make distclean
--------------------------------
```

Configure again:

```text
--------------------------------
$ ./configure --prefix=/usr --target="$LFS_TGT"
--------------------------------
```

Build:

```text
--------------------------------
$ make -j"$(nproc)"
--------------------------------
```

Install into the target root:

```text
--------------------------------
$ make DESTDIR="$LFS" install
--------------------------------
```

Musl is now rebuilt using the target toolchain.

The bootstrap transition is complete:

```text
HOST
glibc
  │
  ▼
BRIDGE
GCC + Binutils
  │
  ▼
TARGET BINUTILS
  │
  ▼
TARGET GCC
C + C++
  │
  ▼
REBUILT MUSL
  │
  ▼
NATIVE MUSL TOOLCHAIN
```

The bridge got us into the target environment.

The target Binutils and GCC now provide the foundation for building the rest of laooOS.

From here onward, the build can progressively move away from the host and toward a self-hosting Musl system.


---

# IV. The Core Toolchain

## 7. Binutils

### 7.1 Building binutils

### 7.2 Installing binutils

### 7.3 Testing the linker tools

## 8. GCC

### 8.1 Building the GCC bridge

### 8.2 GCC and Musl

### 8.3 Building the target GCC

### 8.4 libgcc

### 8.5 libstdc++

### 8.6 GCC runtime libraries

## 9. LLVM / Clang

### 9.1 Building LLVM

### 9.2 Building Clang

### 9.3 LLVM runtime components

### 9.4 Clang and Musl

### 9.5 LLVM toolchain layout

## 10. Mold

### 10.1 Building Mold

### 10.2 Installing Mold

### 10.3 Making Mold the default linker

### 10.4 Testing Mold

### 10.5 Mold in the final system

---

# V. The C Library

## 11. Musl

### 11.1 Building Musl

### 11.2 Installing Musl

### 11.3 crt objects

### 11.4 Dynamic linker

### 11.5 Musl GCC specs

### 11.6 Testing the new libc

---

# VI. Building the Base System

## 12. Essential Libraries

### 12.1 Zlib

### 12.2 Zstd

### 12.3 Libdeflate

### 12.4 Libarchive

### 12.5 Other base libraries

## 13. Build Tools

### 13.1 CMake

### 13.2 Ninja

### 13.3 Meson

### 13.4 pkgconf

### 13.5 Autotools

### 13.6 Python build tools

## 14. Core Userspace

### 14.1 BusyBox

### 14.2 Core utilities

### 14.3 Shell

### 14.4 File utilities

### 14.5 Text utilities

### 14.6 Process utilities

---

# VII. The laooOS System

## 15. Linux

### 15.1 Kernel sources

### 15.2 Kernel configuration

### 15.3 Building the kernel

### 15.4 Installing the kernel

### 15.5 Modules

### 15.6 Firmware

## 16. Init

### 16.1 Why not systemd?

### 16.2 OpenRC

### 16.3 Alternative init systems

### 16.4 Boot sequence

### 16.5 Services

## 17. Device Management

### 17.1 /dev

### 17.2 Device discovery

### 17.3 udev alternatives

### 17.4 Firmware loading

## 18. Networking

### 18.1 Network configuration

### 18.2 Ethernet

### 18.3 Wi-Fi

### 18.4 DNS

### 18.5 SSH

---

# VIII. Optimization

## 19. Compiler Optimization

### 19.1 O2

### 19.2 O3

### 19.3 Ofast

### 19.4 CPU tuning

### 19.5 Vectorization

### 19.6 Loop optimization

### 19.7 Function inlining

### 19.8 Interprocedural optimization

## 20. Link-Time Optimization

### 20.1 LTO

### 20.2 ThinLTO

### 20.3 LTO trade-offs

### 20.4 PGO

### 20.5 When not to use LTO

## 21. Linker Optimization

### 21.1 Mold

### 21.2 Linker flags

### 21.3 Section handling

### 21.4 Dead code elimination

## 22. Size Optimization

### 22.1 Reducing dependencies

### 22.2 Stripping

### 22.3 Smaller binaries

### 22.4 Static vs dynamic linking

---

# IX. Mainstream laooOS

## 23. Building Mainstream laooOS

### 23.1 LLVM toolchain

### 23.2 Clang

### 23.3 Musl

### 23.4 Mold

### 23.5 Base userspace

### 23.6 Final system

## 24. Mainstream laooOS Environment

### 24.1 Compiler environment

### 24.2 Package building

### 24.3 Runtime environment

### 24.4 Testing

---

# X. GCC laooOS

## 25. Building GCC laooOS

### 25.1 GCC bootstrap

### 25.2 GCC bridge

### 25.3 GCC + Musl

### 25.4 Final GCC

### 25.5 Mold

### 25.6 Final system

## 26. GCC laooOS Environment

### 26.1 GCC configuration

### 26.2 Runtime libraries

### 26.3 Package building

### 26.4 Testing

---

# XI. Tiny laooOS

## 27. Designing Tiny laooOS

### 27.1 What can be removed?

### 27.2 Minimal dependencies

### 27.3 Minimal userspace

### 27.4 Minimal services

## 28. Building Tiny laooOS

### 28.1 Minimal toolchain

### 28.2 Musl

### 28.3 Mold

### 28.4 Kernel

### 28.5 Userspace

### 28.6 Booti


# laooOS
My laooOS project
