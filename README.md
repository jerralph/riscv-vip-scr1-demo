# riscv-vip-scr1-demo Overview
This repo demonstrates the RISCV-VIP integrated with the Syntacore SCR1 RISC-V core.  The RISC-V VIP adds the following capabilites:
 * instruction and register trace based on UVM
 * functional coverage based on the RISC-VIP coverage model
 * passive monitoring of instruction execution in a convienient object model for implementing your own coverage and analsysis


# riscv-vip-scr1-demo submodules
The demo uses submodules to pull in the following github repositories

submodule | github source | description
--------- | ------------- | -----------
riscv-vip | https://github.com/jerralph/riscv-vip | riscv-vip project
scr1      | https://github.com/jerralph/scr1/tree/riscv-vip-demo | fork of scr1 with riscv-vip instantiation and connections on riscv-vip-demo branch
riscv-compliance | https://github.com/riscv/riscv-compliance | RISC-V Foundation compliance test suite as used by SCR1 tests.  Commit [9273836](https://github.com/riscv/riscv-compliance/commit/9273836251cc53069f9cc48543fa9c1417e98cb7) from Sept 2018.

# Getting Running

## Prerequisites

### Linux
This assumes you are running on linux

### Mentor Questa Simulator
This demo assumes you have the Questa/Modelsim simulator installed.  Other simulators may work but will require updates to the SCR1 Makefiles. 

### Clone the repository and submodules
Clone the riscv-vip-scr1-demo repo and initialize the submodules.  This assumes SSH and the clone path can be adjusted for https if you prefer. 
```
git clone git@github.com:jerralph/riscv-vip-scr1-demo.git
cd riscv-vip-scr1-demo
git submodule update --init
```

### Setup the environment variables needed by the Make process
In the root of the riscv-vip-scr1-demo repo, source the setup script:
```
. setup.sh
```

### RISC-V GCC toolchain
The RISC-V GCC toolchain is required, as documented in the SCR1 project.  [See the SCR1 project documentation](https://github.com/jerralph/scr1/tree/riscv-vip-demo#prerequisites) for specific details.  The bin directory will need to be on the PATH so the compiler can be executed durring the SCR1 make process.

## Running the simulation
For whatever reason, the make process for the SCR1 has some issues for me and repeately requires me to try and run the simulation and clean targets a few times until things are in order.  Once things are in order it works as expected.

From the root of the riscv-vip-scr1-demo repo cd to the scr1 directory and run the simulation:
```
cd scr1
```

Do iterations of the following until it runs properly (need to figure out and fix in scr1 scripting at some point so clean iterations aren't required)
```
make run_modelsim BUS=AXI ARCH=I IPIC=0 CROSS_PREFIX=riscv32-unknown-elf-
make clean
```

When it eventually runs correctly, you should see an output that looks as follows:

```
# Test passed
# ---Test: compliance_I-RF_x0-01.hex
# Test passed

...

# ---Test: compliance_I-ANDI-01.hex
# Test passed
# ---Test: compliance_I-SRL-01.hex
# Test passed
# ---Test: compliance_I-SLT-01.hex
# Test passed
# ---Test: compliance_I-LB-01.hex
# Test passed
# 
#--------------------------------------
# Summary: 51/51 tests passed
#--------------------------------------
# 
# ** Note: $finish    : /riscv-vip-scr1-demo/scr1/src/tb/scr1_top_tb_axi.sv(242)
#    Time: 515365 ns  Iteration: 1  Instance: /scr1_top_tb_axi
# End time: 14:46:08 on Jan 08,2019, Elapsed time: 0:00:16
# Errors: 0, Warnings: 8
```

## View the RISCV-VIP generated trace file

Open the scr1/build/riscv_tracker_0.log file and you should see the instructions executed and the related register values listed in the trace. 

## View the functional coverage report

After running the simulation open udbc file generated into the build director
```
vsim -viewcov coverage.ucdb
```

The Questa GUI will open.  Next ensure the covergroups view is checked via View > Coverage > Covergroups.
Explore the coverage report

The following image shows some of the different covergroups.
![riscv-vip covergroups](doc/img/scr1-cov0.png)

More details of the individual bins can be viewed by expanding into the details.  For example, the image below shows the cross coverage for read of a register after a write by a previous instuction -- a RAW data hazard.  The cross includes the instruction enumeration, the source register read case (RS1, RS2, or both RS1 and RS2), and the number of cycles between the writing instruction and the reading instruciton. More infromation on this specific cross is avilabile in the riscv-vip users' guide, under the [RAW Hazard coverage example heading](https://jerralph.github.io/riscv-vip/doc/index.html#h.j1blccquh4o5). The highlighted bin in the image, has not been hit by the testing and is for the case where a BGE instruction has an RS1 field that was written by the previous instruction.  Alternatively, the comparable bin for the BNE instruction is one row above and has been hit 51 times. 

![riscv-vip read-after-write covergroups](doc/img/scr1-cov-raw.png)

# Integration details
See the modifications that were made to the SCR1 using one of the two following techniques:
 1. in the Linux console in the riscv-vip-scr1-demo/scr1 directory, run the following
 '''
 git diff master
 '''
 1. look at the diff in github using this link: https://github.com/syntacore/scr1/compare/master...jerralph:riscv-vip-demo

# Future enhancments
 * figure out and fix weird make issues that require clean sim interations
 * point out the details of integration and how that was done
 