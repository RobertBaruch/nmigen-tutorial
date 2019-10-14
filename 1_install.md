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

* For simulation
  * [gtkwave](https://sourceforge.net/projects/gtkwave/)
  