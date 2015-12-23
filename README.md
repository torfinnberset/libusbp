# libusbp - Pololu USB Library

Version: 1.0.0<br/>
Release date: 2015 Dec 24<br/>
[www.pololu.com](https://www.pololu.com/)

**libusbp** is a cross-platform C library for accessing USB devices.

## Features

- Can retrieve the vendor ID, product ID, revision, and serial number for each USB device on the system.
- Can perform I/O on a generic (vendor-defined) USB interface:
  - Synchronous control transfers.
  - Synchronous and asynchronous bulk/interrupt transfers on IN endpoints.
- Can retrieve the names of virtual serial ports provided by a specified USB device (e.g. "COM5").
- Provides detailed error information to the caller.
  - Each error includes one or more English sentences describing the error, including error codes from underlying APIs.
  - Some errors have libusbp-defined error codes that can be used to programmatically decide how to handle the error.
- Provides an object-oriented C++ wrapper (using features of C++11).
- Provides access to underlying identifiers, handles, and file descriptors.


## Comparison to libusb

This library has a lot in common with [libusb 1.0](http://libusb.info/), which has been around for longer and has a lot more features.  However, libusbp does have some useful features that libusb lacks, such as listing the serial number of a USB device without performing I/O or getting information about a USB device's virtual serial ports.  Currently, libusbp lacks some features you would expect from a USB library, such as bulk/interrupt transfers on OUT endpoints.


## Supported platforms

The library runs on:

* Microsoft Windows (Windows Vista and later)
* Linux
* Mac OS X (10.11 and later)


## Supported USB devices and interfaces

libusbp only supports certain types of USB devices.

On Windows, any generic interface that you want to access with this library must use WinUSB as its driver, so you will need to provide a driver package or use some other mechanism to make sure WinUSB is installed.  The generic interface must have a registry key named "DeviceInterfaceGUIDs" which is a REG_MULTI_SZ, and the first string in the key must be a valid GUID.  The "DeviceInterfaceGUID" key is not supported.

Both composite and non-composite devices are supported.

There is no support for switching device configurations or switching between alternate settings of an interface.

There is no support for accessing a generic interface that is included in an interface association and is not the first interface in the association.


## Platform differences

libusbp is a relatively simple wrapper around low-level USB APIs provided by each supported platform.  Because these different APIs have different capabilities, libusbp will also behave differently on different operating systems.  You should not assume your program will work on an operating system on which it has not been tested.  Some notable differences are documented below.


### Permissions

On Linux, a udev rule is typically needed in order to grant permissions for non-root users to directly access USB devices.  The simplest way to do this would be to add a file named `usb.rules` in the directory `/etc/udev/rules.d` with the following contents, which grants all users permission to access all USB devices:

    SUBSYSTEM=="usb", MODE="0666"


### Multiple handles to the same generic interface

On Linux, you can have multiple simultaneous handles open to the same generic interface of a device.  On Windows and Mac OS X, this is not possible, and you will get an error with the code `LIBUSBP_ERROR_ACCESS_DENIED` when trying to open the second handle with `libusbp_generic_handle_open`.


### Timeouts for interrupt IN endpoints

On Mac OS X, you cannot specify a timeout for an interrupt IN endpoint.  Doing so will result in the following error when you try to read from the endpoint:

    Failed to read from pipe.  (iokit/common) invalid argument.  Error code 0xe00002c2.


### Serial numbers

On Windows, calling `libusbp_device_get_serial_number` on a device that does not actually have a serial number will retrieve some other type of identifier that contains andpersands (&) and numbers.  On other platforms, an error will be returned with the code `LIBUSBP_ERROR_NO_SERIAL_NUMBER`.


### Synchronous operations and Ctrl+C

On Linux, a synchronous (blocking) USB transfer cannot be interrupted by pressing Ctrl+C.  Other signals will probably not work either.


### Interface-specific control transfers

Performing control transfers that are directed to a specific interface might not work correctly on all systems, especially if the device is a composite device and the interface you are connected to is not the one addressed in your control transfer.  For example, see [this message](https://sourceforge.net/p/libusb/mailman/message/34414447/) from the libusb mailing list.


## Building from source on Windows with MSYS2

The recommended way to build this library on Windows is to use [MSYS2](http://msys2.github.io/).  After installing MSYS2, start an MSYS2 Win32 shell.

Run this command to install the required packages:

    pacman -S mingw-w64-i686-toolchain mingw-w64-i686-cmake

Download the source code of this library and navigate to the top-level directory using `cd`.

Run these commands to build the library and install it:

    mkdir build
    cd build
    MSYS2_ARG_CONV_EXCL= cmake .. -G"MSYS Makefiles" -DCMAKE_INSTALL_PREFIX=/mingw32
    make install DESTDIR=/
    mv /mingw32/lib/libusbp*.dll /mingw32/bin

Building for 64-bit Windows is also supported, in which case you would replace `i686` with `x86_64` in the package names above, and replace `mingw32` with `mingw64`.

We currently do not provide any build files for Visual Studio, but it might be possible to generate Visual Studio build files using CMake.

## Building from source on Linux

First, you will need to make sure that you have a suitable compiler installed, such as gcc.  You can run `gcc -v` to make sure it is available.

Then you will need to install cmake and libudev.  On Ubuntu, the command to do this is:

    sudo apt-get install cmake libudev-dev

On Arch Linux, libudev should already be installed, and you can install cmake by running:

    sudo pacman -S cmake

Download the source code of this library and navigate to the top-level directory using `cd`.

Run these commands to build the library and install it:

    mkdir build
    cd build
    cmake ..
    make
    sudo make install


## Building from source on Mac OS X

First, install [homebrew](http://brew.sh/).

Then use brew to install cmake:

    brew install cmake

Download the source code of this library and navigate to the top-level directory using `cd`.

Run these commands to build the library and install it:

    mkdir build
    cd build
    cmake ..
    make
    sudo make install


## Incorporating libusbp into a C/C++ project

The first step to incorporating libusbp into another project is to add the libusbp header folder to your project's include search path.
The header folder is the folder that contains `libusbp.h` and `libusbp.hpp`, and it will typically be named `libusbp-1`.
On systems with `pkg-config`, assuming libusbp has been installed properly, you can run the following command to get the include directory:

    pkg-config --cflags libusbp-1

The output of that command is formatted so that it can be directly added to the command-line arguments for most compilers.

Next, you should include the appropriate libusbp header in your source code by adding an include directive at the top of one of your source files.
To use the C API from a C or C++ project, you should write:

    #include <libusbp.h>

To use the C++ API from a C++ project, you should write:

    #include <libusbp.hpp>

After making the changes above, you should be able compile your project successfully.
If the compiler says that the libusbp header file cannot be found, make sure you have specified the include path correctly.

However, if you add a call to any of the libusbp functions and rebuild, you will probably get an undefined reference error from the linker.  To fix this, you need to add libusbp's linker settings to your project.  To get the linker settings, run:

    pkg-config --libs libusbp-1

If you are using GCC and a shell that supports Bash-like syntax, here is an example command that compiles a single-file C program using the correct compiler and linker settings:

    gcc program.c `pkg-config --cflags --libs libusbp-1`

Here is an equivalent command for C++.  Note that we use the `--std=gnu++11` option because the libusbp C++ API requires features from C++11:

    g++ --std=gnu++11 program.cpp `pkg-config --cflags --libs libusbp-1`

The order of the arguments above matters: the user program must come first because it relies on symbols that are defined by libusbp.

The `examples` folder of this repository contains some example code that uses libusbp.  These examples can serve as a starting point for your own project.


## Versioning

This library uses Semantic Versioning 2.0.0 from semver.org.

A backwards-incompatible version of this library might be released in the future.
If that happens, the new version will have a different major version number: its version number will be 2.0.0 and it will be known as libusbp-2 to pkg-config.
This library was designed to support having different major versions installed side-by-side on the same computer, so you could have both libusbp-1 and libusbp-2 installed.

If you write software that depends on libusbp, we recommend specifying which major version of libusbp your software uses in both the documentation of your software and in the scripts used to build it.  Scripts or instructions for downloading the source code of libusbp should use a branch name to ensure that they downlod the latest version of the code for a given major version number.  For example, the v1 branch of this repository will always point to the latest stable release of libusbp-1.


## Version history

* 1.0.0 (2015 Dec 24): Original release.