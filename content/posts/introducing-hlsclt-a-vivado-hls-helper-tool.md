+++
title = "Introducing hlsclt: a Vivado HLS Helper Tool"
type = ["post","posts"]
date = "2017-07-03T08:52:43+00:00"
categories = ["posts"]
tags = [
  "fpga",
  "xilinx",
  "hls",
  "vivado",
  "hlsclt",
]
[ author ]
  name = "Ben Marshall"
+++

**Introduction**  
In my [last blog]({{< ref "using-vivado-hls-on-the-command-line.md" >}}) I considered the trade offs of moving to a command line driven approach to development when using Xilinx's High Level Synthesis tool, Vivado HLS. As a quick summary I suggested that moving to the command line increases how the underlying tool performs and allows for easier integration with source control tools. However, I conceded that we do lose access to useful features such as 'go to definition' and the optimisation views and that the Vivado HLS built-in Tcl interface isn't overly user-friendly.

Today I want to introduce a tool designed to offer a pleasant, efficient, and user-friendly command line driven environment for development using Vivado HLS.

**The Vivado HLS Command Line Helper Tool (hlsclt)**  
[hlsclt](https://github.com/benjmarshall/hlcclt) is an open source tool built on Python, and hosted on Github, which can streamline development of any project using Xilinx's High Level Synthesis tool. It can be installed on any Linux system which has a (reasonably modern) Python install, takes 2 minutes to learn, and provides all the main functions we need to develop a complex IP core from start to finish.

**Installing hlsclt**  
Installation couldn't be simpler, hlsclt is hosted on PyPi which means you can simply use pip to download and install the tool:

```
pip install hlsclt
```

hlsclt is compatible with Python2 and Python3 and has only one dependency, Click, which pip will install for you.

**Using hlsclt**  
Getting started with hlsclt is easy, once installed you can simply invoke the tool using the command `hlsclt`. To take a quick look at all the available controls you can use the 'help' argument:

```
[ben@localhost]$ hlsclt --help
Usage: hlsclt [OPTIONS] COMMAND [ARGS]...

  Helper tool for using Vivado HLS through the command line. If no arguments
  are specified then a default run is executed which includes C simulation,
  C synthesis, Cosimulation and export for both Vivado IP Catalog and System
  Generator. If any of the run options are specified then only those
  specified are performed.

Options:
  --version  Show the version and exit.
  --help     Show this message and exit.

Commands:
  build     Run HLS build stages.
  clean     Remove generated files.
  open_gui  Open the Vivado HLS GUI and load the project.
  report    Open reports.
  status    Print out the current project status.
```

For a quick run through of the tool's features it is a good idea to look at the example Vivado HLS project which ships with hlsclt. The example projects are found within the 'examples' directory in the install folder. You can take a copy of this directory in your user area to test out the main features:

```
[ben@localhost]$ pip show hlsclt
Name: hlsclt
Version: 1.0.0a2
Summary: A Vivado HLS Command Line Helper Tool
Home-page: https://github.com/benjmarshall/hlsclt
Author: Ben Marshall
Author-email: sayhello@benmarshall.co.uk
License: MIT
Location: /usr/lib/python3.6/site-packages
Requires: Click
[ben@localhost]$ cp -r /usr/lib/python3.6/site-packages/hlsclt/examples/simple_adder ~/simple_adder
[ben@localhost]$ cd ~/simple_adder/
[ben@localhost]$ ls
hls_config.py  __pycache__  src tb
```

The only requirement for using hlsclt with a project is a simple configuration file 'config.py' at the top level of the project structure. This file holds the sort of information you have to provide the Vivado HLS GUI when you create a new project. The configuration file for the example project is shown below, for detailed information see the config file section of the [hlsclt readme](https://github.com/benjmarshall/hlsclt/blob/master/README.md#project-configuration). The tool detects the config.py file when invoked and loads in all the settings. It is important that the `hlsclt` command is called from the top level of your project structure, where your config.py is located.

View the code on [Gist](https://gist.github.com/benjmarshall/0f2e8563298718ac1d9fbc29b2cad653).

hlsclt is built using the concept of 'nested' commands (similar to the git CLI), where the main command `hlsclt` has a group of subcommands, some of which in turn have their own subcommands. The `build` subcommand is where most of the action happens. If you invoke the `build` subcommand with the help argument you can see it has further subcommands which allow you to run any of the HLS build stages:

```
[ben@localhost]$ hlsclt build --help
Usage: hlsclt build [OPTIONS] COMMAND1 [ARGS]... [COMMAND2 [ARGS]...]...

  Runs the Vivado HLS tool and executes the specified build stages.

Options:
  -k, --keep    Preserves existing solutions and creates a new one.
  -r, --report  Open build reports when finished.
  --help        Show this message and exit.

Commands:
  cosim   Runs the Vivado HLS cosimulation stage.
  csim    Runs the Vivado HLS C simulation stage.
  export  Runs the Vivado HLS export stage.
  syn     Runs the Vivado HLS C synthesis stage.
```

Lets run a C simulation:

```
[ben@localhost]$ hlsclt build csim
================================================================
  Vivado(TM) HLS - High-Level Synthesis from C, C++ and SystemC
  Version 2017.1
  Build 1846317 on Fri Apr 14 19:19:38 MDT 2017
  Copyright (C) 1986-2017 Xilinx, Inc. All Rights Reserved.
================================================================
INFO: [HLS 200-10] Running '/opt/Xilinx/Vivado_HLS/2017.1/bin/unwrapped/lnx64.o/vivado_hls'
INFO: [HLS 200-10] For user 'ben' on host 'localhost' (Linux_x86_64 version 3.10.0-514.21.1.el7.x86_64) on Mon Jun 26 15:51:34 NZST 2017
INFO: [HLS 200-10] In directory '/mnt/centos_share/Vivado_Projects/hlsclt/hlsclt/examples/simple_adder'
INFO: [HLS 200-10] Creating and opening project '/mnt/centos_share/Vivado_Projects/hlsclt/hlsclt/examples/simple_adder/proj_simple_adder'.
INFO: [HLS 200-10] Adding design file 'src/dut.h' to the project
INFO: [HLS 200-10] Adding design file 'src/dut.cpp' to the project
INFO: [HLS 200-10] Adding test bench file 'tb/testbench.cpp' to the project
INFO: [HLS 200-10] Creating and opening solution '/mnt/centos_share/Vivado_Projects/hlsclt/hlsclt/examples/simple_adder/proj_simple_adder/solution1'.
INFO: [HLS 200-10] Setting target device to '{xc7z020clg484-1}'
INFO: [SYN 201-201] Setting up clock 'default' with a period of 10ns.
INFO: [SIM 211-2] *************** CSIM start ***************
INFO: [SIM 211-4] CSIM will launch GCC as the compiler.
   Compiling ../../../../tb/testbench.cpp in debug mode
   Compiling ../../../../src/dut.cpp in debug mode
   Generating csim.exe
Expected result: 100, Got Result: 100
Expected result: 103, Got Result: 103
Expected result: 106, Got Result: 106
Expected result: 109, Got Result: 109
Expected result: 112, Got Result: 112
Expected result: 115, Got Result: 115
Expected result: 118, Got Result: 118
Expected result: 121, Got Result: 121
Expected result: 124, Got Result: 124
Expected result: 127, Got Result: 127
INFO: [SIM 211-1] CSim done with 0 errors.
INFO: [SIM 211-3] *************** CSIM finish ***************
```

It's as simple as that! Under the hood the tool reads in the configuration from your config.py, writes out all the appropriate Tcl commands to a script and calls the Vivado HLS tool to run that generated script. Any of the build subcommands can be 'chained' together to form a full run, go ahead and run `hlsclt build syn cosim export --type ip --evaluate` to see what happens.

As well as the build command hlsclt offers the ability to open a specific report using the `report` command and launch the Vivado HLS GUI to view the project with the `open_gui` command (useful to have a quick look at the optimisation views). The tool can also show you the current status of your project, which stages have been run and are passing, by using the `status` command. Finally to allow for really easy integration with source control tools, hlsclt also has a `clean` command, which removes all the generated files, leaving just your source and 'config.py' to be incorporated into a new check in/commit.

**Summary**  
So that's hlsclt, a simple, user friendly way to interact with Vivado HLS on the command line. Give it a go and see if you can improve your productivity by harnessing better build times! If up until now you've been scratched your head wondering how best to manage your C/C++ source for HLS projects, then hlsclt might just be your answer.

I started this post by referring to my last one, where I laid out some of the advantages of moving away from the Vivado HLS GUI, for at least some of your development time. My next post will be a run down of my current development process including my tools, techniques and procedures. This will hopefully shine more light on how to maximise productivity with HLS and put a bit more context around how to use hlsclt.
