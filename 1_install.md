Install:

(For windows users, use [WSL](https://docs.microsoft.com/en-us/windows/wsl/install-win10), I recommend Debian as a distro)

* Python:
  * https://blog.smallsec.ca/2017/09/07/installing-python3-6-on-wsl/
  * sudo -H pip3 install --upgrade pip
  * sudo apt install python3.6-dev

* yosys, Symbiyosys, yices2, boolector

Follow the [instructions to install these](https://symbiyosys.readthedocs.io/en/latest/quickstart.html). It is highly recommended to follow those instructions, especially for yosys since the git repo has many more fixes than the official release.

* nMigen
  * pip3 install --user git+https://github.com/m-labs/nmigen.git

* Signal viewer for simulation and formal verification
  * [gtkwave](https://sourceforge.net/projects/gtkwave/) (for WSL, get the Windows version unless you want to run an [X server](https://askubuntu.com/questions/993225/whats-the-easiest-way-to-run-gui-apps-on-windows-subsystem-for-linux-as-of-2018))

If you are planning to synthesize for an FPGA using open source tools, you need to install utilities for your chosen platform:
*  [Lattice ice40 devices](http://www.clifford.at/icestorm/)
*  [Lattice ECP5 devices](https://github.com/SymbiFlow/prjtrellis)

This tutorial does not cover using these tools, but it does cover using nMigen to generate output for these tools.