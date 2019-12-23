# Install

(For windows users, use [WSL](https://docs.microsoft.com/en-us/windows/wsl/install-win10), I recommend Debian as a distro)

> It's unfortunate that installation of everything you need isn't a one-click process... although [FPGAWars' Apio](http://fpgawars.github.io/) goes a long way towards doing this. I just think that while yosys and nMigen are undergoing constant positive updates, Apio doesn't keep up with them. In the meantime, we just install all the tools separately.

* Python: See below for installing Python 3.7 on WSL.

* yosys, Symbiyosys, yices2, boolector
  * `sudo apt-get install curl`
  * Now follow the [instructions to install these](https://symbiyosys.readthedocs.io/en/latest/quickstart.html). It is highly recommended to follow those instructions, especially for yosys since the git repo has many more fixes than the official release.

* nMigen
  * Change to your virtual environment! (see below)
  * `pip3 install wheel`
  * `pip3 install git+https://github.com/m-labs/nmigen.git`

* Signal viewer for simulation and formal verification
  * [gtkwave](https://sourceforge.net/projects/gtkwave/) (for WSL, get the Windows version unless you want to run an [X server](https://askubuntu.com/questions/993225/whats-the-easiest-way-to-run-gui-apps-on-windows-subsystem-for-linux-as-of-2018))
  * gtkwave gives you no clue how to install for Windows. Unzip the zip file into, say, C:, and then add C:\gtkwave\bin to your Windows path.
  * Alternatively, you can install [Xming X Server for Windows](https://sourceforge.net/projects/xming/) and then install gtkwave on Linux via `sudo apt-get install gtkwave`. Be sure to add this to your `~/.profile` file: `export DISPLAY=:0`, and be sure to actually run Xming!

If you are planning to synthesize for an FPGA using open source tools, you need to install utilities for your chosen platform:

* [Lattice ice40 devices](http://www.clifford.at/icestorm/)
  * `sudo apt-get install libeigen3-dev`
  * Follow the given instructions for installing icestorm and NextPNR.
  * Holy Odin, g++-8.3.0 that was installed on WSL **coredumps** when compiling icestorm. Le sigh.
    * `sudo apt-get install g++-7 gcc-7`
    * `sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 60 --slave /usr/bin/g++ g++ /usr/bin/g++-8`
    * `sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 50 --slave /usr/bin/g++ g++ /usr/bin/g++-7`
    * Switch to g++-7 before compiling icestorm: `sudo update-alternatives --config gcc`
  * For WSL, you will also need the Windows version of iceprog.exe, which is inside a zip file from [FPGAWars' toolchain-icestorm repository](https://github.com/FPGAwars/toolchain-icestorm/releases).

* [Lattice ECP5 devices](https://github.com/SymbiFlow/prjtrellis)

This tutorial does not cover using these tools, but it does cover using nMigen to generate output for these tools.

## Python, WSL, and virtual environments

Debian on WSL doesn't come with any Python versions installed. So first install the default Python:

```sh
sudo apt-get update
sudo apt-get install python3
```

On my freshly-installed WSL, this installs Python 3.7.

Now install some bare minimum tools:

```sh
sudo apt-get install python3-pip build-essential libssl-dev libffi-dev python-dev python3-venv
```

We want to keep any Python packages you install for this project isolated from any other project and from the base installation. This means setting up a Python virtual environment. Switching to a particular virtual environment gives you that environment's specific version of Python, and any packages that were installed in that environment.

```sh
cd
mkdir environments
```

It is important to `cd` to your WSL home directory first! If you try to create files and directories in a Windows directory (such as `/mnt/f/something`) then the file or directory will be owned by root, which means that you will have to do everything as root, including creating virtual environments.

To avoid that, you want to create the `environments` directory in your WSL home directory.

Note that you may get a warning that virtualenv was installed to a directory that isn't on your PATH, specifically `~/.local/bin`. I found that I had to exit WSL and start it again, because while `~/.local/bin` *is* put in the PATH by `~/.profile`, it wasn't picked up.

Anyway, now we can create a virtual environment:

```sh
cd environments
python3.7 -m venv n6800  # I named my environment "n6800".
```

This will create a virtual environment with Python 3.7 as its default Python version.

To enter an environment:

```sh
source ~/environments/n6800/bin/activate
```

You can now run Python just as `python` rather than `python3`. You can also install packages via `pip`, for example `pip install numpy`. This installs numpy in `environments/n6800`.

To exit an environment:

```sh
deactivate
```

If you ever screw up your environment beyond repair, simply delete the `~/environments/<yourenv>` directory and start again!

### Another WSL hint

Open WSL, right-click on an empty part of the title bar, and select `Properties`. There should be an option `Use Ctrl+Shift+C/V as Copy/Paste`. Enable that. Now you can copy/paste using the usual Linux copy/paste shortcuts when in a terminal window. This option even sticks between invocations of WSL, so you don't need to set the option every time you open WSL.
