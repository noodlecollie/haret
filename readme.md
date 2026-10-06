Handheld Reverse Engineering Tool, useful for investigating old Windows Mobile devices.

This fork is updated to build with the [CeGCC toolchain](https://github.com/enlyze/cegcc-build).
Only Linux is supported at the moment.

## Building

### Prerequisites

On Ubuntu, you will need these packages:

```bash
# Required:
sudo apt install \
	build-essential \
	cmake

# Required if building the CeGCC toolchain locally (see below):
sudo apt install \
	texinfo \
	flex \
	bison \
	libgmp-dev \
	libmpfr-dev \
	libmpc-dev
```

### Installing the CeGCC Toolchain

For convenience, there is a build target to download and build the CeGCC toolchain on your system.

```bash
mkdir build
cd build

cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build . --target cegcc --config Release
```

The toolchain script isn't very well separated into build and install phases, so the above command
will place its output under `build/cegcc-prefix/install/cegcc`. To fully install this on your system,
simply run:

```bash
# From the build directory, to install to /opt/cegcc:
sudo cp -r cegcc-prefix/install/cegcc /opt/
```

### Building HaRET

To build HaRET itself, invoke CMake as follows:

```bash
# Remove your old build directory first if you've just built and installed CeGCC
rm -rf build
mkdir build
cd build

cmake -DCMAKE_TOOLCHAIN_FILE=../arm-mingw32ce.toolchain -DCMAKE_BUILD_TYPE=Release ..
cmake --build . --config Release
```

If you've installed CeGCC to somewhere other than `/opt/cegcc`, you may also need
to pass `-DCMAKE_SYSROOT=<your path>`.

## Quirks

As HaRET is a very carefully constructed utility, it requires very specific compiler settings.
The required settings, and documentation of the behaviour, are described in [quirks.md](quirks.md).
