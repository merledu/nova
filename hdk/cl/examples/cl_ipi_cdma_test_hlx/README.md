# HLx Flow for IPI CDMA Test Example

## Table of Contents

1. [Overview](#overview)
2. [Setup HLx Environment](#env)
3. [Create Example Design (GUI)](#createbdgui)
4. [Create Example Design (Command line)](#createbd)
5. [Simulation](#sim)
6. [Implementing the Design](#impl)
7. [AFI Creation](#aficreation)
8. [CL Example Software and executing on F1](#swf1)

<a name="overview"></a>
### Overview

AXIL_SDA AXI GPIO input connected to DDR4_A/B/D and SH calibration done signal, processor polls register

DMA_PCIS writes 1K data pattern for DDR4_SH source buffer

AXIL_USR sets AXI CDMA for 1K DMA operation from DDR4_SH to DDR4_B

AXIL_USR polls AXI CDMA status register to determine when transfer is complete

DMA_PCIS reads from destination buffer(DDR4_B) 1K and compares vs data pattern

AXIL_OCL AXI GPIO output written 16-bits for VLED (16-bit patter of 0xAAAA)

Read from VLED from Verilog task or linux command to read VLED to verify pattern

<a name="env"></a>
### Setup HLx Environment

* Clone the github and setup the HDK environment
   ```
   $ git clone https://github.com/aws/aws-fpga.git $AWS_FPGA_REPO_DIR
   $ cd $AWS_FPGA_REPO_DIR
   $ source hdk_setup.sh
   ```
* To setup the HLx Environment, run the following commands:
   ```
   $ mkdir -p ~/.Xilinx/Vivado
   $ echo 'source $::env(HDK_SHELL_DIR)/hlx/hlx_setup.tcl' >> ~/.Xilinx/Vivado/Vivado_init.tcl
   ```
   **NOTE**: *This modifies Vivado defaults, it is recommended you remove this if you wish to run non-HLx examples.* 
   
   For more information please see: [HLx Setup Instructions](../../../../hdk/docs/IPI_GUI_Vivado_Setup.md).

* You will also need to [setup AWS CLI and S3 Bucket](../../../../SDAccel/docs/Setup_AWS_CLI_and_S3_Bucket.md) to enable AFI creation

<a name="createbdgui"></a>
### Create Example Design (GUI)

* To launch Vivado GUI
   * Change directories to the cl/examples/cl_ipi_cdma_test_hlx directory
   * Invoke Vivado by typing `vivado` in the console
   * In the Vivado TCL Console type in the following to create the cl_ipi_cdma_test example. The example will be generated in cl/examples/cl_ipi_cdma_test/example_projects. The vivado project is examples_projects/cl_ipi_cdma_test.xpr
   ```
   aws::make_ipi -examples cl_ipi_cdma_test
   ```
   * Once the Block Diagram is opened, review the different IP blocks especially the settings in the AWS IP

<a name="createbd"></a>
### Create Example Design (Command line)

* Alternatively, to run the Vivado GUI from command line (Linux only)
   * Make sure your $CL_DIR is pointing to the example directory. The following will generate the IPI Block Design (BD)
   ```
   $ cd $HDK_DIR/cl/examples/cl_ipi_cdma_test_hlx
   $ export CL_DIR=$(pwd)
   $ cd $CL_DIR/build/scripts
   $ ./aws_build_dcp_from_cl.sh -gui
   ```    
   **NOTE**: *The "-gui" switch is optional. It allows you to modify the example design, you will need to have a DISPLAY setup for the GUI to launch. To run the full creation and default implementation flow without the GUI, remove this switch.*

<a name="sim"></a>
### Simulation

The simulation settings are already configured.

* To launch simulation from within the Vivado GUI, 
   * In the Sources/Hierarchy tab, under sim_1->IP, disable the 3 IPs for the cl_ipi_example design. (These IPs are included with the AWS IP and are needed when using no DDR4s in the CL for the SH models for DDR4)
   * Click on Simulation->Run Simulation->Run Behavioral Simulation
   * Add signals needed in the simulation
   * Type `run -all` in the TCL console (could take 30 mins to 1 hour based upon machine)

<a name="impl"></a>
### Implementing the Design

* To run implmentation from within the GUI is opened, in the Design Runs tab:
   * Right click on impl\_1 in the Design Runs tab and select Launch Runs???
   * Click OK in the Launch Runs Dialog Box.
   * Click OK in the Missing Synthesis Results Dialog Box

This will run both synthesis and implementation.

<a name="aficreation"></a>
### AFI Creation

The completed .tar file is located in: 
```
$CL_DIR/build/scripts/example_projects/cl_ipi_cdma_test.runs/faas_1/build/checkpoints/to_aws/<timestamp>.Developer_CL.tar  
```
For information on how to create AFI from this tar file, follow the [steps outlined here](../../../README.md#step3).

<a name="swf1"></a>
### CL Example Software and executing on F1

The runtime software must be compiled for the AFI to run on F1.

Copy the software directory to any directory and compile with the following commands:
```
$ cp -r $HDK_COMMON_DIR/shell_stable/hlx/hlx_examples/build/IPI/cl_ipi_cdma_test/software <any_directory>
$ cd software
$ make all
$ sudo ./test_cl
```
