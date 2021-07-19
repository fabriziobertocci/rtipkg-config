# RTI Connext DDS Config

A small command-line utility, similar to pkg-config that help supply build information for a given architecture



## Usage

```
RTI Connext DDS Config version 1.0.4
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Usage:
    connext-config -h|--help        Show this help
    connext-config -V|--version     Prints version number
    connext-config --list-all       List all platform architectures supported
    connext-config --list-installed List the installed architectures
    connext-config --dump-all       Dump all platforms and all settings (testing only)
    connext-config [modifiers] <what> [targetArch]

Where [modifiers] are:
    --static      use static linking against RTI Connext DDS
    --debug       use debug version of the RTI Connext DDS libraries
    --sh          use shell-like variable expansion (vs. make-like variables)
    --noexpand    do not expand environment variables in output
    --libmsg      include libraries for building request/reply apps

Required argument <what> is one of:
  C API:
    --ccomp       output the C compiler to use
    --cflags      output all pre-processor and compiler flags
    --clink       output the C linker to use
    --ldflags     output the linker flags
    --ldlibs      output the required libraries
  Traditional C++ API:
    --cxxcomp     output the C++ compiler to use
    --cxxflags    output all pre-processor and compiler flags
    --cxxlink     output the C++ linker to use
    --ldxxflags   output the linker flags
    --ldxxlibs    output the required libraries
  Modern C++ API (C++-03):
    --cxx03comp   output the C++ compiler to use
    --cxx03flags  output all pre-processor and compiler flags
    --cxx03link   output the C++ linker to use
    --ldxx03flags output the linker flags
    --ldxx03libs  output the required libraries
  Modern C++ API (C++-11):
    --cxx11comp   output the C++ compiler to use
    --cxx11flags  output all pre-processor and compiler flags
    --cxx11link   output the C++ linker to use
    --ldxx11flags output the linker flags
    --ldxx11libs  output the required libraries
  Miscellaneous:
    --os          output the OS (i.e. UNIX, ANDROID, IOS, ...)
    --platform    output the Platform (i.e. i86, x64, armv7a, ...)

Optional argument [targetArch] is one of the supported target architectures.
If not specified, uses environment variable NDDSARCH.
Use `--list-all` or `--list-installed` to print a list of architectures
```



The tool can be invoked in two ways:

1. To list all the supported targets (including targets NOT installed): 
   `connext-config --list-all`
2. List all the installed targets:
   `connext-config --list-installed`
3. To obtain the right tool name, flags and libraries for a given architecture. 

In general, when determining the tools required to build a RTI Connext DDS application, you may need to use this tool few times. For example, to build a C program:

* Use the `--ccomp` to determine the correct compiler to use for the specified API (api=c/cxx/cxx03/cxx11)
* Use the `--cflags` to determine the right flags for the C compiler 
* Use the `--clink` to determine the correct linker to use
* Use the `--ldflags` and `--ldlibs` to get the flags and libraries for the linker. Place the output of the `--ldflags` at the beginning of the argument list, and the output of `--ldlibs` at the end.

NOTE: there is also an additional way you can invoke the tool that is accessible only when you build the tool from the source code, and you use the `--enable-debug` flag with the `./configure` script. This enables the following use:`connext-config --dump-all` to dump all the platforms and settings available. This mode is only meant to debugging and troubleshooting.

If a target does not support a specific language, the tool will print an error. E.g.:
```
$ connext-config --cxx11flags i86Linux2.6gcc4.4.5
Error: target 'i86Linux2.6gcc4.4.5' does not support C++11
```



### Examples

Show all the supported Linux 64-bit targets: 

```
$ ./connext-config --list-all | grep x64Linux
x64Linux2.6gcc3.4.5
x64Linux2.6gcc4.1.1
x64Linux2.6gcc4.1.2
...
```

Show all the installed targets:
```
$ connext-config --list-installed
armv6vfphLinux3.xgcc4.7.2
x64Linux4gcc7.3.0
x64Darwin17clang9.0
```


Show the C compiler to use for the target architecture `ppc4xxFPLinux2.6gcc4.5.1`:

```
$ ./connext-config --ccomp ppc4xxFPLinux2.6gcc4.5.1
powerpc-ngc-linux-gcc
```



Shows the libraries to use when statically linking (`--static`) C applications on `x64Linux2.6gcc4.5.1` with debugging information (`--debug`):

```
$ ./connext-config --debug --static --ldlibs x64Linux2.6gcc4.5.1
-L/Users/fabrizio/working/rti_connext_dds-6.0.0/lib/x64Linux2.6gcc4.5.1 -lnddsczd -lnddscorezd -ldl -lnsl -lm -lpthread -lrt
```



### How to use it in projects

Inside the `examples` directory you can find some projects that uses this tool to automatically configure the build system.

* `examples/autoconf-hello`: this is the same `hello_builtin` example located in the RTI workspace area (typically `~/rti_workspace`) under the `examples/connext_dds/c/hello_builtin` directory. This example uses the popular autotools (autoconf, automake) to invoke `connext-config` to determine the correct build settings. 
  Refer to the [README.md](examples/autoconf-hello/README.md) file for additional information on this example.
* `example/autoconf-hello-idl-cpp`: this is the same `hello_idl` example located in the RTI workspace area, under the `examples/connext_dds/c++/hello_idl` directory. Similar to the previous example, this project uses autotools (autoconf, automake) to automatically invoke `rtiddsgen` to generate type support code from the included IDL file, and`connext-config` to obtain build settings for a given platform.
* `examples/cmake-requestreply`: this is the same `hello_world_request_reply` example located in the RTI workspace area under the `examples/connext_dds/c/hello_world_request_reply` directory. This example uses cmake to invoke `connext-config` and generate the appropriate build files. Refer to the [README.md](examples/cmake-requestreply/README.md) file for additional information on this example





## Additional Notes

* The tool operates by looking up the requested information from the platform file used by `rtiddsgen`. This file is located under: `$NDDSHOME/resource/app/app_support/rtiddsgen/templates/projectfiles/platforms.vm`.

  where `$NDDSHOME` is the directory where RTI Connext DDS is installed.
  Because of this dependency, `connext-config` needs to be able to locate this directory.

* `connext-config` will first use the environment variable `$NDDSHOME` to locate the platform file above. If `$NDDSHOME` is not defined, it will attempt to determine the location of the platform file from the location of the `connext-config` application (assumes this tool is invoked from the `$NDDSHOME/bin` directory).
  If it cannot locate the `platforms.vm` file, it will report an error.

* Starting from version 1.0.1, `connext-config` the target architecture can be omitted from the command line and specified in the environment variable `NDDSARCH`. For example: the following two lines are equivalent:
  
```
  $ ./connext-config --cflags x64Linux4gcc7.3.0
  $ NDDSARCH=x64Linux4gcc7.3.0 ./connext-config --cflags
  ```
  
* All the error messages are printed to `stderr`, while the normal results are printed to `stdout` (help is printed to `stdout`).

* The `--debug` option affects only the name of the required libraries and NOT the C or C++ compiler flags (for example, it does not include `-g` or disable any optimization, nor define any debug-related macros). 

* Many platforms are not supported: Windows, iOS, Integrity, and in general all the platform where a project is normally not built through Makefile. All the unix-based architectures are supported.

* To build `connext-config` from the the git source tree, just run:

  ```
  $ autoreconf -ivf
  $ ./configure
  $ make
  ```

  You need to have the traditional autoconf, automake, make, buildtools installed in your system.

* To build `connext-config` from a source release, you don't need to invoke `autoreconf` as the .tar.gz already include the `./configure` script.
  
* To install this application on your existing RTI Connext DDS installation:
  
  * Copy the script: `bin/connext-config` under `$NDDSHOME/bin`
  * Copy the compiled version of connext-config under: `$NDDSHOME/resource/app/bin/<hostPlatform>` (where <hostPlatform> is the name of the directory containing platform-specific binaries).
  
  Alternatively just un-tar one of the pre-built binaries (see the [Release page](https://github.com/rticommunity/connext-config/releases))
