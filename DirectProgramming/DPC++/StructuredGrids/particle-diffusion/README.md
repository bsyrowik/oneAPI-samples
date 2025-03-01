﻿# `Water Molecule Diffusion` Sample

This code sample implements a simple example of a Monte Carlo simulation of water molecule diffusion in tissue. This kind of computational experiment can be used to simulate the acquisition of a diffusion signal for dMRI.

For comprehensive information in using oneAPI programming, see the [Intel&reg; oneAPI Programming Guide](https://software.intel.com/en-us/oneapi-programming-guide), and use search or the table of contents to find relevant information.

| Property                     | Description
|:---                               |:---
| What you will learn               | How to offload the computation to GPU using the Intel&reg; oneAPI DPC++/C++ Compiler
| Time to complete                  | 20 minutes

## Purpose

The simulation model consists of water molecules moving through a 2D array of cells in a tissue sample (water molecule diffusion). In this code sample, we use a uniform rectilinear 2D array of digital cells, where cells are spaced regularly along each direction and are represented by circles.

Water molecule diffusion is simulated by defining a number of particles P (simulated water molecules) at random positions in the grid, followed by random walks of these particles in the ensemble of cells in the grid. During the random walks, particles can move randomly inside or outside simulated cells. The positions of these particles at every time step in the simulation, the number of times they go through a cell membrane (in/out), and the time every particle spends inside and outside cells can be recorded. These measurements are a simple example of useful information that can be used to simulate an MR signal.

> **Note**: You can find more information and a walk through for this sample at [Code Sample: Particle Diffusion – An Intel® oneAPI DPC++/C++ Compiler Example](https://www.intel.com/content/www/us/en/developer/articles/code-sample/oneapi-dpcpp-compiler-example-particle-diffusion.html).

Performance number tabulation, if applicable.

| motionsim sample                  | Performance data
|:---                               |:---
| Scalar baseline -O2               | 1.0
| SYCL                              |
| OpenMP offload                    |

## Prerequisites
| Optimized for                     | Description
|:---                               |:---
| OS                                | Ubuntu* 18.04 <br>Windows* 10 <br>Windows* Server 2017
| Hardware                          | Kaby Lake with Gen9 or newer
| Software                          | Intel&reg; oneAPI DPC++/C++ Compiler

## Key Implementation Details

SYCL* implementation explained in this sample includes:
- SYCL* queues (including device selectors and exception handlers).
- SYCL buffers and accessors.
- The ability to call a function inside a kernel definition and pass accessor arguments as pointers.
- Optimization using API-based programming and Atomic Functions.

SYCL implementation is explained in further detail in the source code.

Intel&reg; Math Kernel Library (Intel&reg; MKL) is used to generate random numbers on the CPU and device. Precise generators are used within this library to ensure that the numbers generated on the CPU and device are relatively equivalent (relative accuracy 10E-07).

## Build the `particle-diffusion` Program for CPU and GPU

> **Note**: If you have not already done so, set up your CLI
> environment by sourcing  the `setvars` script located in
> the root of your oneAPI installation.
>
> Linux:
> - For system wide installations: `. /opt/intel/oneapi/setvars.sh`
> - For private installations: `. ~/intel/oneapi/setvars.sh`
>
> Windows:
> - `C:\Program Files(x86)\Intel\oneAPI\setvars.bat`
>
>For more information on environment variables, see Use the setvars Script for [Linux or macOS](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-programming-guide/top/oneapi-development-environment-setup/use-the-setvars-script-with-linux-or-macos.html), or [Windows](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-programming-guide/top/oneapi-development-environment-setup/use-the-setvars-script-with-windows.html).

### Include Files
The include folder is located at `%ONEAPI_ROOT%\dev-utilities\latest\include` on your
development system.

### Use Visual Studio Code*  (Optional)

You can use Visual Studio Code (VS Code) extensions to set your environment,
create launch configurations, and browse and download samples.

The basic steps to build and run a sample using VS Code include:
- Download a sample using the extension **Code Sample Browser for Intel&reg; oneAPI Toolkits**.
- Configure the oneAPI environment with the extension **Environment Configurator for Intel&reg; oneAPI Toolkits**.
- Open a Terminal in VS Code (**Terminal>New Terminal**).
- Run the sample in the VS Code terminal using the instructions below.

To learn more about the extensions and how to configure the oneAPI environment, see
[Using Visual Studio Code with Intel&reg; oneAPI Toolkits User Guide](https://software.intel.com/content/www/us/en/develop/documentation/using-vs-code-with-intel-oneapi/top.html).

## Build the Sample
### On Linux*

1. Change to the particle diffusion directory.
```
    $ cd particle-diffusion
```
2. Build the program.
```
    $ mkdir build 
    $ cd build 
    $ cmake .. 
    $ make -s -j
```

### On Windows*
1. Use `nmake` to build the sample. 

> **Note**: `nmake clean` removes temporary files.

### Run Samples in Intel&reg; DevCloud
If running a sample in the Intel&reg; DevCloud, you must specify the compute node (CPU, GPU, FPGA) and whether to run in batch or interactive mode. For more information, see the Intel&reg; oneAPI Base Toolkit [Get Started Guide](https://devcloud.intel.com/oneapi/get_started/).

## Run the Sample

### On Windows*
1. Run the sample.
   ```
   motionsim.exe 10000 256 22 777 0 1
   ```

### On Linux*
1. Run the sample using default parameters.
    ```
    $ make run
    ```
2. Alternatively, run the sample using custom parameters shown below.

The following table describes each command line parameter (applies to Linux builds only)

|    Flag and Argument          |    Description               |    Range of Possible Values    |    Default
|:---                           |:---                          |:---                            |:---
| `-i num_iterations`           | Number of iterations         | [1, &#8734;]                   | 10000
| `-p num_particles`            | Number of particles          | [1, &#8734;]                   | 256
| `-g grid_size`                | Size of square grid          | [1, &#8734;]                   | 22
| `-r rng_seed`                 | Random number generator seed | [-&#8734;, &#8734;]            | 777
| `-c cpu_flag`                 | Turns cpu comparison on/off  | [1 \| 0]                       | 0
| `-o output_flag`              | Turns grid output on/off     | [1 \| 0]                       | 1
| `-h`                          | Help message.                |                                |

> **Note**: 
>
>- If the grid size specified is greater than **44**, the application will not print the grid even if the grid output flag is on.
>- If a particular parameter is not specified, the application will choose the default value for that parameter.
>- Typing `$ ./src/motionsim.exe -h` displays a brief help message and exits the program.

You can run the program using the above parameters with the application binary:
```
$ ./src/motionsim.exe
```
Example usage:
```
$ ./src/motionsim.exe -i 1000 -p 200 -g 30 -r 777 -c 1 -o 0
```
## Example Output
```
    **Running with default parameters**

    Running on: Intel(R) Gen9
    Device Max Work Group Size: 256
    Device Max EUCount: 24
    Number of iterations: 10000
    Number of particles: 256
    Size of the grid: 22
    Random number seed: 777

    Device Offload time: 0.4128 s


    **********************************************************
    *                           DEVICE                       *
    **********************************************************

    **************** PARTICLE ACCUMULATION: ****************

    0   0   0   0   0   0   0   0   0   0  57   0   0   0   0   0   0   0   0   0   0   0
    0   0   0   0   0   0   0   0   0  67 206 115   0   0   0   0   0   0   0   0   0   0
    0   0   0   0   0   0   0   0  96  19 294  58   0  28   4   0   0   0   0   0   0   0
    0   0   0   0   0   0   0   0 190 425 762 279 295 1189 360   0   0   0   0   0   0   0
    0   0   0   0   0   0   0   0 188 459 270 299 1322 1112   2   0   0   0   0   0   0   0
    0   0   0   0   0   3   0   0 106 1052 1420 1965 1322 3727 1401   0   0   0   0   0   0   0
    0   0   0   0   0 364 113 369 2304 1751 2331 4417 4566 5189 5347 749   0   0   0   0   0   0
    0   0   0   0   0 614 1512 4425 5751 4703 7985 9204 12470 7068 3322 1184 295   0   0   0   0   0
    0   0   0   0   0   1 2304 6365 9364 8978 11351 21867 14009 14881 12119 3735 738 227   0   0   0   0
    0   0   0   0   0   0 2260 3721 10615 19659 31735 39856 33540 27154 17662 8117 5268 1453  42   0   0   0
    0   0   0   0   0   0 375 2706 12957 25556 59223 83841 63499 46326 30569 12369 6702 2343 322   0   0   0
    0   0   0   0   3 208 2812 5794 16767 36975 76524 176569 77169 45581 24982 10339 5834 6639 1726   0   0   0
    0   0   0   0 402 1367 3606 7850 20228 38542 59977 89857 54812 28349 19833 11819 9471 4919 1273 404   0   0
    0   0   0   0 406 1205 4792 6854 15010 31872 33418 31999 31918 17756 12250 11218 5525 296 1223 957   0   0
    0   0   0  45 884 294 2093 4473 10284 17923 17643 21078 18451 11740 11821 5437 2206 610  55  10   0   0
    0   0   0   0 142   0   8 3142 5201 4911 6353 10991 8210 2382 3650 2327 1386 802   2   0   0   0
    0   0   0   0   0 119 246 1080 1650 3119 1354 4303 4063 1138 750 1265 577 686 261   0   0   0
    0   0   0   0   0 1055 827 685 646 688 615 2212 1114 505 235 268   0   1  42   0   0   0
    0   0   0   0   0  25   0 280   0   0   0 197 206   0 158 185   0   0   0   0   0   0
    0   0   0   0   0   0   0   0   0   0   0   0   0 528 675  89   0   0   0   0   0   0
    0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0
    0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0

    ***************** FINAL SNAPSHOT: *****************

    0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0
    0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0
    0   0   0   0   0   0   0   0   1   0   0   0   0   0   0   0   0   0   0   0   0   0
    0   0   0   0   0   0   0   0   0   0   0   0   1   1   0   0   0   0   0   0   0   0
    0   0   0   0   0   0   0   0   0   1   0   0   0   0   0   0   0   0   0   0   0   0
    0   0   0   0   0   0   0   0   0   1   0   1   0   0   1   0   0   0   0   0   0   0
    0   0   0   0   0   0   1   0   0   0   0   1   1   0   2   0   0   0   0   0   0   0
    0   0   0   0   0   1   0   1   2   0   5   1   3   1   1   0   0   0   0   0   0   0
    0   0   0   0   0   0   0   2   1   1   0   4   1   4   1   1   0   1   0   0   0   0
    0   0   0   0   0   0   0   0   2   4   1   4   2   1   3   2   0   1   0   0   0   0
    0   0   0   0   0   0   0   2   1   2   3   3   3   6   3   4   4   1   0   0   0   0
    0   0   0   0   0   0   1   0   2   1   4   0   1   4   0   1   0   1   0   0   0   0
    0   0   0   0   0   1   3   1   2   3   5   3   2   2   1   0   0   2   0   0   0   0
    0   0   0   0   1   1   1   3   1   2   2   4   3   2   4   3   0   1   0   1   0   0
    0   0   0   0   0   0   0   1   3   2   4   0   1   0   2   0   0   1   0   0   0   0
    0   0   0   0   0   0   0   0   1   0   2   2   0   1   0   1   0   0   0   0   0   0
    0   0   0   0   0   0   0   0   0   0   0   0   1   3   1   0   0   0   1   0   0   0
    0   0   0   0   0   0   0   1   1   1   0   2   0   0   0   0   0   0   0   0   0   0
    0   0   0   0   0   0   0   1   0   0   0   0   0   0   0   0   0   0   0   0   0   0
    0   0   0   0   0   0   0   0   0   0   0   0   0   1   0   0   0   0   0   0   0   0
    0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0
    0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0
    Number of particles inside snapshot: 198

    ************* NUMBER OF PARTICLE ENTRIES: *************

    0   0   0   0   0   0   0   0   0   0  12   0   0   0   0   0   0   0   0   0   0   0
    0   0   0   0   0   0   0   0   0   7  18   4   0   0   0   0   0   0   0   0   0   0
    0   0   0   0   0   0   0   0   9   9  18  13   0   7   2   0   0   0   0   0   0   0
    0   0   0   0   0   0   0   0   5  54  51  34  10  43  17   0   0   0   0   0   0   0
    0   0   0   0   0   0   0   0  23  33  18  14  93  63   2   0   0   0   0   0   0   0
    0   0   0   0   0   3   0   0   4  53  91 116  84 173  67   0   0   0   0   0   0   0
    0   0   0   0   0  17  12  22  59  92 158 209 242 284 191  59   0   0   0   0   0   0
    0   0   0   0   0  20  73 221 259 290 272 578 565 356 200  79  23   0   0   0   0   0
    0   0   0   0   0   1 101 311 454 405 517 1142 909 721 589 226  27  17   0   0   0   0
    0   0   0   0   0   0  54 181 502 915 1580 1903 1656 1381 932 398 233  45   6   0   0   0
    0   0   0   0   0   0  19 193 646 1448 2881 4255 3042 2176 1398 627 327 164  18   0   0   0
    0   0   0   0   1  11  90 315 854 1618 3692 7064 3659 2264 1477 577 249 301  86   0   0   0
    0   0   0   0  34  58 191 396 1005 1784 2975 4161 2658 1307 908 590 539 260  66  11   0   0
    0   0   0   0  11  73 193 344 810 1469 1763 1588 1488 870 619 436 286  31  37  24   0   0
    0   0   0   8  41  35  82 215 526 673 818 942 808 609 477 266 130  39  12   4   0   0
    0   0   0   0  10   0   4 142 235 285 294 569 295 183 170 143  51  12   2   0   0   0
    0   0   0   0   0   3  12  37  84 108  79 286 186  93  40  56  10  27  20   0   0   0
    0   0   0   0   0  21  26  24  36  36  34 124  62  19  22  13   0   1   2   0   0   0
    0   0   0   0   0   1   0   3   0   0   0  13  19   0  18  20   0   0   0   0   0   0
    0   0   0   0   0   0   0   0   0   0   0   0   0   5  19  15   0   0   0   0   0   0
    0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0
    0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0
    **********************************************************
    *                        END DEVICE                      *
    **********************************************************

    Success.
    Built target run
```

## License
Code samples are licensed under the MIT license. See
[License.txt](https://github.com/oneapi-src/oneAPI-samples/blob/master/License.txt) for details.

Third party program Licenses can be found here: [third-party-programs.txt](https://github.com/oneapi-src/oneAPI-samples/blob/master/third-party-programs.txt).