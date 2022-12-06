# nerdydaysofRTL

This repository contains some verilog code I have learnt in my early days of working in vlsi.

## Introduction

This repository will be helpful for beginner to start from the scratch.

## Toolchain installation for windows

### (1) Verilog compiler - Icarus Varilog (for simulation)

Icarus Varilog is a free compuler implementation for the IEEE-1364 Verilog hardware description language.

To install it go to the link: [iverilog](https://bleyer.org/icarus/) and download the appropriate version.

Run installer.

Select full installation in Select Components.

Don't forget to tick "Add executable folder(s) to the user PATH".

Type ```iverilog``` in cmd to check it is working or not.

### (2) yosys (for synthesis)

Yosys is a framework for Verilog RTL synthesis.

To install it go to the link: [yosys](https://github.com/YosysHQ/oss-cad-suite-build/releases/tag/2022-11-29)

Extract it.

Go to ```*\oss-cad-suite\bin``` folder and add it to system path. To add it to system path type exact ```advancedsystemsettings``` in windows startmenu search bar and Select "Environment Variables" button, Select Path variable and press edit button, press new in pop up dialog box, and paste the path i.e. ```*\oss-cad-suite\bin```

Type ```yosys``` in cmd to check whether it is working or not.

### (3) Graphviz (for converting diagrams generated by yosys)

Graphviz is open source graph visualization software. Graph visualization is a way of representing structural information as diagrams of abstract graphs and networks.

To install it go to the link: [Graphviz](https://graphviz.org/download/) and download appropriate version.

Run installer.

Don't forget to select Add Graphviz to the system PATH.

Type ```dot -V``` in cmd to check it is working or not.

### (4) make (for managing the infra)

Make: GNU make utility to maintain groups of programs.

To install it go to the link: [make](https://gnuwin32.sourceforge.net/packages/make.htm) and download appropriate version.

Run installer.

Select full installation in Select Components.

Go to ```*\GnuWin32\bin``` folder and add it to system path. To add it to system path type exact "advancedsystemsettings" in windows startmenu search bar and Select "Environment Variables" button, Select Path variable and press edit button, press new in pop up dialog box, and paste the path i.e. ```*\GnuWin32\bin```

Type yosys in ```make -v``` to check whether it is working or not.

## Simulation and Synthesis for windows

### (1) Verilog module simulation introduction

To write a verilog module;

Write following code in whatever text editor you prefer,

```verilog
module inverter(y,a);

  output y;
	input a;

	assign y=~a;

endmodule
```

and save it as inverter.v

To simulate this module with Icarus Verilog, open a command propmt, change directory to the folder with the file 'inverter.v' and type the following command and execute: ```iverilog -o inverter.out inverter.v```

This will make Icarus to compile the design and generate any error message in case there are syntax errors. If everything is fine, the compiled design is written to the 'inverter.out' file. To simulate the desing the 'vpp' command included with Icarus Verilog: ```vvp inverter.out```

This will execute the simulation of the module.

### (2) Simulation with test bench

To write a test bench module;

Write following code in text editor,

```verilog
module test;
  reg a;
  wire y;

  //Design Instance
  inverter uut(y,a);

  //enabling the wave dump
  initial
  begin
    $dumpfile("inverter.vcd"); $dumpvars;
  end

	initial
	begin

    $display ("RESULT \ta\ty");

		a = 1; # 100; // Another value
    if (y==0)
      $display ("PASS \t%d\t%d",a,y);
    else
      $display ("PASS \t%d\t%d",a,y);

		a = 0; # 100; // Initial value is set
    if (y==1)
      $display ("PASS \t%d\t%d",a,y);
    else
      $display ("PASS \t%d\t%d",a,y);

		a = 1; # 50; // Another value
    if (y==0)
      $display ("PASS \t%d\t%d",a,y);
    else
      $display ("PASS \t%d\t%d",a,y);

		a = 0; # 100; // Initial value is set
    if (y==1)
      $display ("PASS \t%d\t%d",a,y);
    else
      $display ("PASS \t%d\t%d",a,y);

	end

endmodule
```

and save it as inverter_tb.v

A test bench is a module without inputs or outputs. It typically includes only statements to generate test signals and simulator directives to control the simulation process and the generation of results, together with an instance of the module that is to be tested: the Unit Under Test (UUT).

Compile the whole design (inverter module and test bench) with the following command: ```iverilog -o inverter.out inverter.v inverter_tb.v```

Note that the module 'test' in 'inverter_tb.v' uses the module 'inverter' defined in 'inverter.v'. The Verilog compilar automatically sorts out the dependencies between modules provided that it is given the files containing all the needed modules. The compiled design is written to file 'inverter.out'.

Simulate the design with: ```vvp inverter.out```

Take a look to the simulation results with: ```gtkwave inverter.vcd```

### (3) Make things easy using makefile

To create a makefile, write following code in text editor and save it without any extension (or remove extension if any)

```cmake
module = inverter

TOOLCMD = iverilog -o $(module).out -Wall -Winfloop -g2012

compile: clean
	$(TOOLCMD) $(module).v

sim: clean
	$(TOOLCMD) $(module).v $(module)_tb.v
	vvp $(module).out
	gtkwave $(module).vcd -r ../gtkwaverc &

build: clean
	copy nul "synth.ys"
	echo read -sv $(module).v > synth.ys
	echo hierarchy -top $(module) >> synth.ys
	echo proc; opt; techmap; opt >> synth.ys
	echo write_verilog synth.v >> synth.ys
	echo show -prefix $(module) >> synth.ys

synth: build
	yosys synth.ys

clean:
	del $(module).dot /Q
```

