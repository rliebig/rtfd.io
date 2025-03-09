---
title: Installation
---

Qiling Framework is written in Python which allows it to work on different operating system and CPU architectures. There are several methods to install and use Qiling which we will cover here:
  1. Install Qiling using `pip`
  2. Checking out Qiling from github
  3. Using a Docker image

## Prerequisities

### Python 3
Qiling requires Python 3.8 or newer. If you intend to develop your own scripts and utilites on top of Qiling, consider using Python 3.11 as Qiling will soon shift to that release.

Most chances that Python 3 is already installed on your system, as most systems come with Python pre-installed. However, if Python 3 needs to be installed:
 - on Windows, use *Microsoft Store* to locate and install Python 3.8 or newer.
 - on Linux, use the system package manager. On Ubuntu / Debian systems run:
   ```bash
   sudo apt install python3
   ```

### Python virtual environment (recommended)
It is very recommended, yet not obligatory, to work in a dedicated Python virtual environment. That helps different Python programs co-exist even if they have contradicting requirements for the same packages. In other words, it keeps Qiling's Python packages isolated from other programs that might require different releases of the same packages.

Creating the environment is done once. Every time you want to use Qiling you will need to *activate* the environment.

#### 1. Creating a virtual environment
Browse to a directory where the environment should be created and choose a name for it (assuming here `qlenv`). Then run the following command to create it:
```sh
python3 -m venv qlenv
```

#### 2. Activating the virtual environment
To use Qiling with the virtual environment you will need to switch to it. If you created the environment as part of the installation process, switch to it to continue the process:

 - on Windows:
   ```dos
   qlenv\Scripts\activate
   ```
 - on Linux:
   ```sh
   . qlenv/bin/activate
   ```

Once the virtual environment has been activated, your prompt should change accordingly.

## Installation

### Install Qiling using `pip`
The most straightforward installation is done by `pip`. If you are using a virtual Python environment, make sure to switch to it first before proceeding (see above).

 - To install *stable* release, use:
   ```
   python3 -m pip install qiling
   ```
 - To stay more up to date with bug fixes and features, you may install using *dev* branch:
   ```
   python3 -m pip install --user https://github.com/qilingframework/qiling/archive/dev.zip
   ```

### Checking out Qiling from github
This method is a bit more advanced but most recommended for users who intend to develop their own tools on top of Qiling, or want to stay up to date with latest fixes. If you are using a virtual Python environment, make sure to switch to it first before proceeding (see above).

```
git clone -b dev https://github.com/qilingframework/qiling.git
cd qiling
git submodule update --init --recursive
pip3 install .
```

### Using a Docker image
For users who wish to minimize the clutter on their system there is a Docker image available.

 - Pull Qiling docker image from dockerhub by running:
   ```
   docker pull qilingframework/qiling:latest
   ```
 - Attach to the running docker container:
   ```
   docker exec -it qiling bash
   ```

Docker container port can be published with the `-p` switch; this is useful for emulating network services such as `httpd` of a router.

## Workarounds

### Emulating Windows programs
Due to legal reasons Qiling Framework cannot bundle Microsoft Windows DLL files and registry hives. If you own a legal Windows copy, you may use it to copy the required DLL files and resigtry hives to your Qiling installation. For you convinience we included a script named `dllscollector.bat` that collects all the required files from the system it runs on and place them in their appropriate location within the Qiling `rootfs` folder.

Run the script on your local Windows installation with **Administrator** privileges:
```cmd
examples\scripts\dllscollector.bat
```

Though the script is safe to use, it is advised that before running a script with administrator privileges on your system one should understand what it is about to do. To do so you may read the script carefully.

If using a Docker image, DLLs can be bind-mounted to Qiling Framework container. Assuming DLLs and registry hive files are located in a sub-directorie named `/analysis/win/rootfs`:
```sh
docker run -dt --name qiling -v /analysis/win/rootfs/x86_windows:/qiling/examples/rootfs/x86_windows -v /analysis/win/rootfs/x8664_windows:/qiling/examples/rootfs/x8664_windows qilingframework/qiling:latest
```

### Keystone engine on macOS >= 10.14
Keystone-engine compilation from py-pip fails (on Mojave at least) because i386 architecture is deprecated for macOS.
```
CMake Error at /usr/local/Cellar/cmake/3.15.4/share/cmake/Modules/CMakeTestCCompiler.cmake:60 (message):
  The C compiler

    "/Library/Developer/CommandLineTools/usr/bin/cc"

  is not able to compile a simple test program.

  It fails with the following output:
```

To work around that, install keystone-engine from source:
```sh
git clone https://github.com/keystone-engine/keystone
cd keystone
mkdir build
cd build
../make-share.sh
cd ../bindings/python
sudo make install
```

Once completed workaround installation, run Qiling Framework setup.
