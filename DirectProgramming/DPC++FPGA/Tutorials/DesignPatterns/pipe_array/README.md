
# Data Transfers Using Pipe Arrays
This FPGA tutorial showcases a design pattern that makes it possible to create arrays of pipes.

| Optimized for                     | Description
---                                 |---
| OS                                | Linux* Ubuntu* 18.04/20.04 <br> RHEL*/CentOS* 8 <br> SUSE* 15 <br> Windows* 10
| Hardware                          | Intel&reg; Programmable Acceleration Card (PAC) with Intel Arria&reg; 10 GX FPGA <br> Intel&reg; FPGA Programmable Acceleration Card (PAC) D5005 (with Intel Stratix&reg; 10 SX) <br> Intel&reg; FPGA 3rd party / custom platforms with oneAPI support <br> *__Note__: Intel&reg; FPGA PAC hardware is only compatible with Ubuntu 18.04*
| Software                          | Intel&reg; oneAPI DPC++ Compiler <br> Intel&reg; FPGA Add-On for oneAPI Base Toolkit
| What you will learn               | A design pattern to generate an array of pipes using SYCL* <br> Static loop unrolling through template metaprogramming
| Time to complete                  | 15 minutes



## Purpose
In certain situations, it is useful to create a collection of pipes that can be indexed like an array in a SYCL-compliant FPGA design. If you are not yet familiar with pipes, refer to the prerequisite tutorial "Data Transfers Using Pipes".

In SYCL*, each pipe defines a unique type with static methods for reading data (`read`) and writing data (`write`). Since pipes are not objects but *types*, defining a collection of pipes requires C++ template meta-programming. This is somewhat non-intuitive but yields highly efficient code.

This tutorial provides a convenient pair of header files defining an abstraction for an array of pipes. The headers can be used in any SYCL-compliant design and can be extended as necessary.

### Example 1: A simple array of pipes

To create an array of pipes, include the pipe_utils.hpp header from the DirectProgramming/DPC++FPGA/include/ directory in your design:

```c++
#include "pipe_utils.hpp"
```

As with regular pipes, an array of pipes needs template parameters for an ID, for the `min_capacity` of each pipe, and each pipe's data type. An array of pipes additionally requires one or more template parameters to specify the array size. The following code declares a one dimensional array of 10 pipes, each with `capacity=32`, that operate on `int` values.

```c++
using MyPipeArray = PipeArray<     // Defined in "pipe_utils.hpp".
    class MyPipe,                  // An identifier for the pipe.
    int,                           // The type of data in the pipe.
    32,                            // The capacity of each pipe.
    10                             // array dimension.
    >;
```

The uniqueness of a pipe array is derived from a combination of all template parameters.

Indexing inside a pipe array can be done via the `PipeArray::PipeAt` type alias, as shown in the following code snippet:

```c++
MyPipeArray::PipeAt<3>::write(17);
auto x = MyPipeArray::PipeAt<3>::read();
```
The template parameter `<3>` identifies a specific pipe within the array of pipes.  The index of the pipe being accessed *must* be determinable at compile time.

In most cases, we want to use an array of pipes so that we can iterate over them in a loop. To respect the requirement that all pipe indices are uniquely determinable at compile time, we must use a static form of loop unrolling based on C++ templates. A simple example is shown in the code snippet:

```c++
// Write 17 to every pipe in the array
Unroller<0, 10>::Step([](auto i) {
  MyPipeArray::PipeAt<i>::write(17);
});
```
While this may initially feel foreign to those unaccustomed to C++ template metaprogramming, this is a simple and powerful pattern common to many C++ libraries. It is easy to reuse. This code sample includes a simple header file `unroller.hpp`, which implements the  `Unroller` functionality.

### Example 2: A 2D array of pipes

This code sample defines a `Producer` kernel that reads data from host memory and forwards this data into a two dimensional pipe matrix.

The following code snippet creates a two dimensional pipe array.
``` c++
constexpr size_t kNumRows = 2;
constexpr size_t kNumCols = 2;
constexpr size_t kDepth = 2;

using ProducerToConsumerPipeMatrix = PipeArray<  // Defined in "pipe_utils.hpp".
    class ProducerConsumerPipe,                  // An identifier for the pipe.
    uint64_t,                                    // The type of data in the pipe.
    kDepth,                                      // The capacity of each pipe.
    kNumRows,                                    // array dimension.
    kNumCols                                     // array dimension.
    >;
```
The producer kernel writes `num_passes` units of data into each of the `kNumRows * kNumCols` pipes. Note that the unrollers' lambdas must capture certain variables from their outer scope.

```c++
h.single_task<ProducerTutorial>([=]() {
  size_t input_idx = 0;
  for (size_t pass = 0; pass < num_passes; pass++) {
    // Template-based unroll (outer "i" loop)
    Unroller<0, kNumRows>::Step([&input_idx, input_accessor](auto i) {
      // Template-based unroll (inner "j" loop)
      Unroller<0, kNumCols>::Step([&input_idx, i, input_accessor](auto j) {
        // Write a value to the <i,j> pipe of the pipe array
        ProducerToConsumerPipeMatrix::PipeAt<i, j>::write(
            input_accessor[input_idx++]);
      });
    });
  }
});
```

The code sample also defines an array of `Consumer` kernels that each read from a unique pipe in `ProducerToConsumerPipeMatrix`, process the data, and write the result to the host memory.

```c++
// The consumer kernel reads from a single pipe, determined by consumer_id
h.single_task<ConsumerTutorial<consumer_id>>([=]() {
  constexpr size_t x = consumer_id / kNumCols;
  constexpr size_t y = consumer_id % kNumCols;
  for (size_t i = 0; i < num_elements; ++i) {
    auto input = ProducerToConsumerPipeMatrix::PipeAt<x,y>::read();
    uint64_t answer = ConsumerWork(input); // do some processing
    output_accessor[i] = answer;
  }
});
```

The host must thus enqueue the producer kernel and `kNumRows * kNumCols` separate consumer kernels. The latter is achieved through another static unroll.
```c++
{
  queue q(device_selector, dpc_common::exception_handler);

  // Enqueue producer
  buffer<uint64_t,1> producer_buffer(producer_input);
  Producer(q, producer_buffer);

  // Use template-based unroll to enqueue multiple consumers
  std::vector<buffer<uint64_t,1>> consumer_buffers;
  Unroller<0, kNumberOfConsumers>::Step([&](auto consumer_id) {
    consumer_buffers.emplace_back(consumer_output[consumer_id].data(), items_per_consumer);
    Consumer<consumer_id>(q, consumer_buffers.back());
  });
}
```

### Additional Documentation
- [Explore SYCL* Through Intel&reg; FPGA Code Samples](https://software.intel.com/content/www/us/en/develop/articles/explore-dpcpp-through-intel-fpga-code-samples.html) helps you to navigate the samples and build your knowledge of FPGAs and SYCL.
- [FPGA Optimization Guide for Intel&reg; oneAPI Toolkits](https://software.intel.com/content/www/us/en/develop/documentation/oneapi-fpga-optimization-guide) helps you understand how to target FPGAs using SYCL and Intel&reg; oneAPI Toolkits.
- [Intel&reg; oneAPI Programming Guide](https://software.intel.com/en-us/oneapi-programming-guide) helps you understand target-independent, SYCL-compliant programming using Intel&reg; oneAPI Toolkits.

## Key Concepts
* A design pattern to generate an array of pipes.
* Static loop unrolling through template metaprogramming.

## Building the `pipe_array` Tutorial

> **Note**: If you have not already done so, set up your CLI
> environment by sourcing  the `setvars` script located in
> the root of your oneAPI installation.
>
> Linux*:
> - For system wide installations: `. /opt/intel/oneapi/setvars.sh`
> - For private installations: `. ~/intel/oneapi/setvars.sh`
>
> Windows*:
> - `C:\Program Files(x86)\Intel\oneAPI\setvars.bat`
>
>For more information on environment variables, see **Use the setvars Script** for [Linux or macOS](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-programming-guide/top/oneapi-development-environment-setup/use-the-setvars-script-with-linux-or-macos.html), or [Windows](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-programming-guide/top/oneapi-development-environment-setup/use-the-setvars-script-with-windows.html).

### Include Files
The included header `dpc_common.hpp` is located at `%ONEAPI_ROOT%\dev-utilities\latest\include` on your development system.

### Running Samples in Intel&reg; DevCloud
If running a sample in the Intel&reg; DevCloud, remember that you must specify the type of compute node and whether to run in batch or interactive mode. Compiles to FPGA are only supported on fpga_compile nodes. Executing programs on FPGA hardware is only supported on fpga_runtime nodes of the appropriate type, such as fpga_runtime:arria10 or fpga_runtime:stratix10.  Neither compiling nor executing programs on FPGA hardware are supported on the login nodes. For more information, see the Intel&reg; oneAPI Base Toolkit Get Started Guide ([https://devcloud.intel.com/oneapi/documentation/base-toolkit/](https://devcloud.intel.com/oneapi/documentation/base-toolkit/)).

When compiling for FPGA hardware, it is recommended to increase the job timeout to 12h.

### Using Visual Studio Code*  (Optional)

You can use Visual Studio Code (VS Code) extensions to set your environment, create launch configurations,
and browse and download samples.

The basic steps to build and run a sample using VS Code include:
 - Download a sample using the extension **Code Sample Browser for Intel&reg; oneAPI Toolkits**.
 - Configure the oneAPI environment with the extension **Environment Configurator for Intel&reg; oneAPI Toolkits**.
 - Open a Terminal in VS Code (**Terminal>New Terminal**).
 - Run the sample in the VS Code terminal using the instructions below.

To learn more about the extensions and how to configure the oneAPI environment, see the
[Using Visual Studio Code with Intel&reg; oneAPI Toolkits User Guide](https://software.intel.com/content/www/us/en/develop/documentation/using-vs-code-with-intel-oneapi/top.html).

### On a Linux* System

1. Generate the `Makefile` by running `cmake`.
     ```
   mkdir build
   cd build
   ```
   To compile for the Intel&reg; PAC with Intel Arria&reg; 10 GX FPGA, run `cmake` using the command:
    ```
    cmake ..
   ```
   Alternatively, to compile for the Intel&reg; FPGA PAC D5005 (with Intel Stratix&reg; 10 SX), run `cmake` using the command:

   ```
   cmake .. -DFPGA_BOARD=intel_s10sx_pac:pac_s10
   ```
   You can also compile for a custom FPGA platform. Ensure that the board support package is installed on your system. Then run `cmake` using the command:
   ```
   cmake .. -DFPGA_BOARD=<board-support-package>:<board-variant>
   ```

2. Compile the design through the generated `Makefile`. The following build targets are provided, matching the recommended development flow:

   * Compile for emulation (fast compile time, targets emulated FPGA device):
      ```
      make fpga_emu
      ```
   * Generate the optimization report:
     ```
     make report
     ```
   * Compile for FPGA hardware (longer compile time, targets FPGA device):
     ```
     make fpga
     ```
3. (Optional) As the above hardware compile may take several hours to complete, FPGA precompiled binaries (compatible with Linux* Ubuntu* 18.04) can be downloaded <a href="https://iotdk.intel.com/fpga-precompiled-binaries/latest/pipe_array.fpga.tar.gz" download>here</a>.

### On a Windows* System

1. Generate the `Makefile` by running `cmake`.
     ```
   mkdir build
   cd build
   ```
   To compile for the Intel&reg; PAC with Intel Arria&reg; 10 GX FPGA, run `cmake` using the command:
    ```
    cmake -G "NMake Makefiles" ..
   ```
   Alternatively, to compile for the Intel&reg; FPGA PAC D5005 (with Intel Stratix&reg; 10 SX), run `cmake` using the command:

   ```
   cmake -G "NMake Makefiles" .. -DFPGA_BOARD=intel_s10sx_pac:pac_s10
   ```
   You can also compile for a custom FPGA platform. Ensure that the board support package is installed on your system. Then run `cmake` using the command:
   ```
   cmake -G "NMake Makefiles" .. -DFPGA_BOARD=<board-support-package>:<board-variant>
   ```

2. Compile the design through the generated `Makefile`. The following build targets are provided, matching the recommended development flow:

   * Compile for emulation (fast compile time, targets emulated FPGA device):
     ```
     nmake fpga_emu
     ```
   * Generate the optimization report:
     ```
     nmake report
     ```
   * Compile for FPGA hardware (longer compile time, targets FPGA device):
     ```
     nmake fpga
     ```

> **Note**: The Intel&reg; PAC with Intel Arria&reg; 10 GX FPGA and Intel&reg; FPGA PAC D5005 (with Intel Stratix&reg; 10 SX) do not support Windows*. Compiling to FPGA hardware on Windows* requires a third-party or custom Board Support Package (BSP) with Windows* support.

> **Note**: If you encounter any issues with long paths when compiling under Windows*, you may have to create your ‘build’ directory in a shorter path, for example c:\samples\build.  You can then run cmake from that directory, and provide cmake with the full path to your sample directory.

### Troubleshooting

If an error occurs, you can get more details by running `make` with
the `VERBOSE=1` argument:
``make VERBOSE=1``
For more comprehensive troubleshooting, use the Diagnostics Utility for
Intel&reg; oneAPI Toolkits, which provides system checks to find missing
dependencies and permissions errors.
[Learn more](https://software.intel.com/content/www/us/en/develop/documentation/diagnostic-utility-user-guide/top.html).


 ### In Third-Party Integrated Development Environments (IDEs)

You can compile and run this tutorial in the Eclipse* IDE (in Linux*) and the Visual Studio* IDE (in Windows*). For instructions, refer to the following link: [FPGA Workflows on Third-Party IDEs for Intel&reg; oneAPI Toolkits](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-oneapi-dpcpp-fpga-workflow-on-ide.html)


## Examining the Reports
Locate `report.html` in the `pipe_array_report.prj/reports/` directory. Open the report in any of Chrome*, Firefox*, Edge*, or Internet Explorer*.

You can visualize the kernels and pipes generated by looking at the "System Viewer" section of the report. However, it is recommended that you first reduce the array dimensions `kNumRows` and `kNumCols` to small values (2 or 3) to facilitate visualization.

## Running the Sample

 1. Run the sample on the FPGA emulator (the kernel executes on the CPU):
     ```
     ./pipe_array.fpga_emu     (Linux)
     pipe_array.fpga_emu.exe   (Windows)
     ```
2. Run the sample on the FPGA device:
     ```
     ./pipe_array.fpga         (Linux)
     pipe_array.fpga.exe       (Windows)
     ```

### Example of Output
```
Input Array Size:  1024
Enqueuing producer...
Enqueuing consumer 0...
Enqueuing consumer 1...
Enqueuing consumer 2...
Enqueuing consumer 3...
PASSED: The results are correct
```

## License
Code samples are licensed under the MIT license. See
[License.txt](https://github.com/oneapi-src/oneAPI-samples/blob/master/License.txt) for details.

Third party program Licenses can be found here: [third-party-programs.txt](https://github.com/oneapi-src/oneAPI-samples/blob/master/third-party-programs.txt).