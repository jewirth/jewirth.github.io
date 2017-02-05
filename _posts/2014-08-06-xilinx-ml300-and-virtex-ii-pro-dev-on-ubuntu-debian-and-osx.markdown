---
layout: post
title: "Xilinx ML300 and Virtex-II Pro Development on Ubuntu Desktop 14.04.1, Debian 7.6 and OS X"
---

Recently, I tried to implement a design for the [Xilinx ML300](http://www.xilinx.com/products/boards/ml300/) board which is an approx. 12 years old FPGA development board. It is built around the Virtex-II Pro XC2VP7 FPGA which holds a physical PowerPC 405 processor inside ready to be routed to some logic.

Well, I ran into some dead ends while setting up the the Xilinx Design Tools on Ubuntu 14.04.1 to be able to work with the Virtex-II pro. Officially, only RHEL 4/5 and SUSE LE 10 are supported but I found the following way to get everything working.

<p style="margin-bottom:0.5em">I have tested these steps successfully with following setups:</p>
- Ubuntu 14.04.1 Desktop 32-bit
	- as guest OS under OS X 10.9.4 with VirtualBox 4.3.14 (incl. Extension Pack)
- Ubuntu 14.04.1 Desktop 64-bit
	- as guest OS under OS X 10.9.4 with VirtualBox 4.3.14 (incl. Extension Pack)
	- as main OS
- Debian 7.6.0 64-bit
	- as main OS

Here are the steps:

1. <a href="#step1">Get the proper version of the design tools</a>
2. <a href="#step2">Install ISE & EDK</a>
3. <a href="#step3">Configure the environment</a>
4. <a href="#step4">Check if ISE works</a>
5. <a href="#step5">Check if EDK works</a>
6. <a href="#step6">Platform Cable USB</a>
7. <a href="#step7">Verify by programming the FPGA</a>


<a name="step1"></a>
###1. Get the proper version of the design tools

The latest release of the Vivado or ISE Design Tools do not support the Virtex-II Pro architecture. So, don't waste time and bandwidth by downloading it. The latest support for the Virtex-II Pro is within the **ISE Design Tools 10.1 Service Pack 3**. You have to download and install ISE 10.1 and the Service Pack 3 separately from xilinx.com. The ISE and EDK tarballs include support for 32- and 64-bit systems. The Service Pack 3 update comes in zip files. Some zips are valid for both architectures and some zips are separately available for 32- and 64-bit. Be sure to get the right ones for your system.

At the download page for ISE 10.1 there is a link named *Obtain a v10.1 Registration ID*. If you don't have a Registration ID then you can get one for free there.

<p style="font-size:12pt; line-height: 16pt; font-style: italic;">Note: If you are not familiar with the Xilinx Design Tools, here is a short summary. The ISE Design Tools is the classical tools package from Xilinx. The most important parts are ISE Foundation (just called ISE) and the Embedded Development Kit (EDK). ISE allows implementing logic by using VHDL, Verilog and schematics. The EDK provides a GUI called Xilinx Platform Studio (XPS), a SDK for PowerPC and MicroBlaze and a catalog of IP-cores. The EDK uses ISE and allows system level design. Don't get confused: the terms EDK and XPS are often used synonymously. Also, consider that these notes explain the naming conventions for the ISE Design Tools 10.1 and not any earlier or later release.</p>


<a name="step2"></a>
###2. Install ISE & EDK

When extracting the downloaded tar files do not extract to a Windows- or a Mac- filesystem. This will destroy the installer consistency. The installer GUI will look garbled and it will tell you that your Registration ID is not valid. This is very very very misleading and annoying. Simply extract the tar and zip files somewhere beneath your home directory, which should be an ext4 filesystem if you are on Ubuntu-14.04.1. Also, don't use spaces in the path where the installers are extracted to. Otherwise you'll get an error message saying your processor platform is not supported.

All archives contain a programm called `setup` in the root of the archive. Simply run it with `./setup`. There is no need to use sudo for the installion as long as you install it somewhere in your home directory. The directory I use is `/home/jens/Xilinx/10.1`.

When installing ISE and EDK do not select to install the cable drivers. It won't work with your kernel. We will catch up on this later.

<a name="step3"></a>
###3. Configure the environment

After the installation of ISE and EDK each installer will show you some environment variables that have to be set for ISE and EDK. These are stored as shell scripts in:

    ~/Xilinx/10.1/ISE/settings32.sh
    ~/Xilinx/10.1/EDK/settings32.sh


on 64-bit OS:

	~/Xilinx/10.1/ISE/settings64.sh
	~/Xilinx/10.1/EDK/settings64.sh

For EDK, both have be sourced (see next step). For ISE, just the ISE script is sufficient.

Additionally, there are some manual steps to do for EDK:

Crucial for EDK's functionality is to create symlinks for `gmake` and `libdb-4.1.so`:

	sudo ln -s /usr/bin/make /usr/bin/gmake

on Ubuntu desktop 14.04.1 64-bit:

	sudo ln -s /usr/lib/x86_64-linux-gnu/libdb-5.3.so /usr/lib/x86_64-linux-gnu/libdb-4.1.so

on Debian 7.6.0 64-bit:

	sudo ln -s /usr/lib/x86_64-linux-gnu/libdb-5.1.so /usr/lib/x86_64-linux-gnu/libdb-4.1.so

on Ubuntu desktop 14.04.1 32-bit:

	sudo ln -s /usr/lib/i386-linux-gnu/libdb-5.3.so /usr/lib/i386-linux-gnu/libdb-4.1.so


Also very useful: Install `xpdf` and create a symlink with the name `acroread` to it. EDK offers a button to open the PDF datasheet for the IP-core you are currently editing/viewing. It tries to open the PDF file with `acroread`, which is not available on Ubuntu. So we want to use xpdf instead:

	sudo apt-get install xpdf
	sudo ln -s xpdf /usr/bin/acroread

Finally, the EDK is using a `$LANGUAGE` variable but the same variable is set to your default locale by Ubuntu, e.g. `en_US`. Thus, EDK will throw a lot of warnings during synthesis, so we have to unset it with `unset LANGUAGE` before starting EDK. I put this at the end of the `settings32.sh` respectively `settings64.sh` of EDK with:

On a 32-bit OS:

	echo "unset LANGUAGE" >>  ~/Xilinx/10.1/EDK/settings32.sh

...or on 64-bit OS:

	echo "unset LANGUAGE" >>  ~/Xilinx/10.1/EDK/settings64.sh

Now, on 64-bit OS, install libc6-i386:

	sudo apt-get install libc6-i386

Finally, change the default shell to bash:

	sudo ln -sf bash /bin/sh


<a name="step4"></a>
###4. Check if ISE works

Now, let's start ISE from the shell with:

	source Xilinx/10.1/ISE/settings32.sh
	ise

...or if you are on 64-bit OS:

	source Xilinx/10.1/ISE/settings64.sh
	ise

The first line executes ISE's setting script in the context of our shell and makes necessary environment variables available to ISE.

<p style="font-size:12pt; line-height: 16pt; font-style: italic;">NOTE: Randomly, I got a "Segmentation fault (core dumped)" message and ISE returned early. The solution was to disable the "Tip of the Day" feature. BTW, this note would make a great Tip of the Day!</p>

<p style="margin-bottom:0.5em">Now, let's check if ISE really works by creating a new design and light up the first LED in GPIO2/TEST area on the ML300 board:</p>
- Create a new project with HDL as top-level source type
- Set the device properties according to the target hardware. <br>In case of the ML300 that is:
	- Product Category: General Purpose
	- Family: Virtex2P
	- Device: XC2VP7
	- Package: FF672
	- Speed: -6
	- TOP-Level Source Type: HDL
	- Synthesis Tool: XST (VHDL/Verilog)
	- Simulator: Modelsim-SE Mixed
	- Preferred Language: VHDL
	- Enable Enhanced Design Summary: YES
	- Enable Message Filtering: NO
	- Display Incremental Messages: NO
- Create two new source files:
	- VHDL Module "top.vhd"
	- Implementation Constraints File: "top.ucf"

top.vhd:
{% highlight vhdl %}
    library IEEE;
    use IEEE.STD_LOGIC_1164.ALL;
    use IEEE.STD_LOGIC_ARITH.ALL;
    use IEEE.STD_LOGIC_UNSIGNED.ALL;

    entity top is
        Port ( led : out  STD_LOGIC );
    end top;
    
    architecture Behavioral of top is
    begin
        led <= '1';
    end Behavioral;
{% endhighlight %}


{% highlight plain %}
    # T8 is connected the first LED in the GPIO2/TEST area.
    NET "led"  LOC = "T8"  |  IOSTANDARD = LVCMOS33;
{% endhighlight %}

Right-click "Generate Programming File" and select "Run" to initiate the process of generating a programming file for the FPGA. The ISE console should print

	Process "Generate Programming File" completed successfully

if the process finished and there should be a `top.bit` file in your project directory, which is the programming file you just generated. In <a href="#step7">step 7</a> we will upload it to the FPGA to verify if it works as expected.


<a name="step5"></a>
###5. Check if EDK works

We start EDK, respectively XPS, from the shell with

	source Xilinx/10.1/ISE/settings32.sh
	xps

...or if you are on 64-bit OS:

	source Xilinx/10.1/ISE/settings64.sh
	xps

<p style="margin-bottom:0.5em">We are asked to create a new or open an existing project. Let's create a new project with the Base System Builder wizard:</p>
- Choose to a directory to store the project file (many files will be generated there, so I reommend to create a new directory)
- Select to create new design for the Xilinx ML300 board
- Select the PowerPC as processor
- Keep the PowerPC configuration as preselected
- With the next four pages "Configure IO Devices" deselect everything but RS232_Uart_1
- Click *next* on the following pages, then *generate* and *finish*
- Select *Device Configuration* from the menu, then *Update Bitstream*

Now, the EDK creates a `system.bit` file for the hardware design (similar step as for the `top.bit` in ISE). Then, it builds the example applications and merges them with the `system.bit` to a file named `download.bit`. This new bitstream contains the FPGA logic utilizing the PowerPC core, including software which will be put into the FPGA's block ram area. When everything is ready you should read "Done!" in ISE's output console. In <a href="#step7">step 7</a> we will upload it to the FPGA to verify if it works as expected.


<a name="step6"></a>
###6. Platform Cable USB

To get the Xilinx Platform Cable USB working we need an alternative for the official drivers from Xilinx because they won't work with the current kernel. Luckily, a guy named [Michael Gernoth solved this issue by implementing a library called *libusb-driver*, which emulates the original driver](http://rmdir.de/~michael/xilinx/). As stated on his page, this should work with more cables than just the Platform Cable USB. However, I only have this one available.

The following commands will get and install Michael's cable drivers:

	sudo apt-get install git libusb-dev fxload
	cd ~/Xilinx
	git clone git://git.zerfleddert.de/usb-driver
	cd usb-driver
	make

Then on a 32-bit Ubuntu or Debian:

	source ~/Xilinx/10.1/ISE/settings32.sh
	./setup_pcusb

... or on 64-bit OS:

	source ~/Xilinx/10.1/ISE/settings64.sh
	./setup_pcusb

Now, we set the `LD_PRELOAD` environment variable and test our cable connection with iMPACT:

	export LD_PRELOAD=/home/jens/Xilinx/usb-driver/libusb-driver.so
	impact

- Double-click "Boundary Scan"
- Right click in the big white area which is labeled "Right click to Add Device or Initialize JTAG chain"
- Select "Cable Auto Connect"

If you didn't get any error and if the iMPACT console doesn't say "Cable autodetection failed." then everything is fine.

I appended the line with `LD_PRELOAD` at the end of the `settings32.sh` respectively `settings64.sh` of ISE with:

	echo "export LD_PRELOAD=/home/jens/Xilinx/usb-driver/libusb-driver.so" >> ~/Xilinx/10.1/ISE/settings32.sh

respectively:

	echo "export LD_PRELOAD=/home/jens/Xilinx/usb-driver/libusb-driver.so" >> ~/Xilinx/10.1/ISE/settings32.sh


<a name="step7"></a>
###7. Verify by programming the FPGA

To verify all previous steps we will program our FPGA by using the programming files top.bit (generated with ISE) and download.bit (generated with EDK).

Start iMPACT from the shell with:

	source xilinx_env
	impact

- Double-click "Boundary Scan"
- Right click in the big white area which is labeled "Right click to Add Device or Initialize JTAG chain"
- Select "Initialize Chain"
- The first device in the chain is the ACE controller, which we don't want to programm. Click "Bypass"
- Now we are asked to select a configuration file for the FPGA. Browse and select the top.bit file generated by ISE and click "Open".
- The next window asks if we want to add additional files for block RAM information and software executables. We don't want that. Click "OK" twice.
- iMPACT shows the JTAG chain and indicates that `top.bit` is associated to the xc2vp7 device, which is the FPGA
- Right-click xc2vp7 and select "Program" to start the programming procedure.
- The "FPGA done" LED on the ML300 should light up within some seconds to indicate that the FPGA has been programmed. Same for the first LED in the GPIO2/TEST area because that what we told in our little example above with `led <= '1'`.

Now, right-click the xc2vp7 device and select "Assign New Configuration File". Remove `top.bit` and add the `download.bit` file generated by EDK and click "Open". The file is in the subdirectory `impelementation/` of your EDK project directory.

Hook up a serial RS232 connection to the first RS232 port of the ML300 board and observe the output at 9600,8,n,1. Then right-click the xc2vp7 device and select "Program" to start the programming procedure. After programming the FPGA you should see:

	-- Entering main() --
	-- Exiting main() --

If so, well done. You've set up an environment ready to develop for the ML300 and Virtex-II Pro. It works directly with Ubuntu Desktop 14.04.1 64-bit, Debian 7.6.0 64-bit as well as virtual machine with VirtualBox under OS X 10.9.4 with VirtualBox 4.3.14 (incl. Extension Pack).
