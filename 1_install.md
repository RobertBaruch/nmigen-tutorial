# Install

## Note for Windows Users

For windows users, install [WSL](https://docs.microsoft.com/en-us/windows/wsl/install-win10). I recommend Ubuntu 20.04 LTS as a distro. Make sure you also enable WSL 2.

## Install all the things

> It's unfortunate that installation of everything you need isn't a one-click process... although [FPGAWars' Apio](http://fpgawars.github.io/) goes a long way towards doing this. I just think that while yosys and nMigen are undergoing constant positive updates, Apio doesn't keep up with them. In the meantime, we just install all the tools separately.

* Python: See below for Python 3.8 on WSL.

* Install yosys, Symbiyosys, yices2, and z3.
  * `sudo apt install curl`
  * Now follow the [instructions to install these](https://symbiyosys.readthedocs.io/en/latest/install.html). It is highly recommended to follow those instructions, especially for yosys since the git repo has many more fixes than the official release.
  * Note that when you see `-j$(nproc)`, it means to specify the number of processors your CPU has. You can't really go wrong by using `-j4`. You can go higher if you know you have more.
  * z3 takes a long time to compile. Go watch a YouTube video in the meantime.

* Install nMigen
  * `pip3 install wheel`
  * `pip3 install git+https://github.com/nmigen/nmigen`

* Signal viewer for simulation and formal verification
  * [gtkwave](https://sourceforge.net/projects/gtkwave/)
    * For WSL, get the Windows version unless you want to run an [X server](https://medium.com/@japheth.yates/the-complete-wsl2-gui-setup-2582828f4577).
    * The package for gtkwave for Windows gives you no clue how to install for Windows. Unzip the zip file into, say, C:, and then add C:\gtkwave\bin to your Windows path.

### For the FPGA developer

If you are planning to synthesize for an FPGA using open source tools, you need to install utilities for your chosen platform:

* [Lattice ice40 devices](http://www.clifford.at/icestorm/)
  * Install this prerequisite that is not in the instructions: `sudo apt-get install libeigen3-dev`
  * Install the other prerequisites as in the instructions.
  * Holy Odin, g++-8.3.0 that was installed on WSL **coredumps** when compiling icestorm. Le sigh.
    * `sudo apt-get install g++-7 gcc-7`
    * `sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 60 --slave /usr/bin/g++ g++ /usr/bin/g++-8`
    * `sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 50 --slave /usr/bin/g++ g++ /usr/bin/g++-7`
    * Switch to g++-7 before compiling icestorm: `sudo update-alternatives --config gcc`
  * **Now** follow the given instructions for installing icestorm and NextPNR.
  * For WSL, you will also need the Windows version of iceprog.exe, which is inside a zip file from [FPGAWars' toolchain-icestorm repository](https://github.com/FPGAwars/toolchain-icestorm/releases).
  * For Debian on WSL, you need to "fix" libQt5Core.so.5: `sudo strip --remove-section=.note.ABI-tag /usr/lib/x86_64-linux-gnu/libQt5Core.so.5`. See [bug](https://github.com/Microsoft/WSL/issues/3023). So... use Ubuntu for WSL instead?

* [Lattice ECP5 devices](https://github.com/SymbiFlow/prjtrellis)

This tutorial does not cover using these tools, but it does cover using nMigen to generate output for these tools.

## Python and WSL

Python 2 and Python 3 are not compatible. This is why this happens:

* `python`, `pip` -> Python 2
* `python3`, `pip3` -> Python 3

On my freshly-installed WSL, the only version present is Python 3.8. If it is not on your version, you will want to upgrade to 3.8. Upgrading is beyond the scope of this document.

Now install some bare minimum tools:

```sh
sudo apt install python3-pip build-essential libssl-dev libffi-dev
```

In installing the yosys prerequisites, Python 2 will be installed. So be aware that when you want to run anything under Python 3, you must use `python3` (or `pip3` for installing), not `python` (or `pip`).

# Tip for vscode users:

Open File > Preferences > Settings, look for pylint args, and add:

```
--contextmanager-decorators=contextlib.contextmanager,nmigen.hdl.dsl._guardedcontextmanager
```

This is because pylint doesn't recognize that `nmigen.hdl.dsl._guardedcontextmanager` is a valid context manager. Otherwise pylint will complain for every `with m.If` statement.
