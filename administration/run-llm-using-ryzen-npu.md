# HOWTO run LLM using Ryzen AI NPU on Debian 13

This guide shows how to install and run neural networks locally using AMD Ryzen
AI NPU. For example using AMD Ryzen AI 350 with 32Gb of RAM one can easily run
[gemma-4-E4B-it](https://fastflowlm.com/docs/models/gemma/#-model-card-gemma-4-e4b-it).
It is also possible use ROCm backend and run for instance `Flux-2-Klein-4B` by
loading model into RAM.

The stack includes AMD XDNA2 driver, Xilinx XRT library, FastFlowLM and ROCm
backends and Lemonade server. The full list of NPU models supported by used
stack can be found [here](https://fastflowlm.com/docs/models/) and
[here](https://lemonade-server.ai/models.html).

# Install Linux Kernel 7.0

Edit APT sources list:
```
sudo apt edit-sources
```

Append the following lines to the end of the file:
```
# Enables backports
deb http://deb.debian.org/debian trixie-backports main contrib non-free non-free-firmware
deb-src http://deb.debian.org/debian trixie-backports main contrib non-free non-free-firmware
```

Update APT cache and install latest version of kernel from backports:
```
sudo apt update
sudo apt install -t trixie-backports linux-image-amd64 linux-headers-amd64 firmware-linux
```

# Setup Lemonade

## Install FastFlowLM

Download FLM from [GitHub releases](https://github.com/FastFlowLM/FastFlowLM/releases).

Install FLM:
```
sudo apt install ./<flm_package.deb>
```
This automatically installs Xilinx XRT libraries needed.

Validate installation:
```
sudo flm validate
```

If your are going to run FastFlowLM out of Lemonade server then you need to
update `memlock` limits. Add the following lines at the end of the
`/etc/security/limits.conf`:
```
*    soft    memlock    unlimited
*    hard    memlock    unlimited
```
Log out and log in back to apply the changes.

## Install Lemonade

Get the .deb from the latest release:
[lemonade-server_\<version\>-debian13_amd64.deb](https://github.com/lemonade-sdk/lemonade/releases)

Install the package:
```
sudo apt install ./lemonade-server_*-debian13_amd64.deb
```

Enable and start the server:
```
sudo systemctl enable --now lemond
```

## Tune the amount of RAM available to the ROCm

This step is required if you want to run the models which are bigger than half
of the RAM using ROCm (not NPU) backend.

Install `pipx` and `amd-debug-tools`:
```
sudo apt install pipx
pipx ensurepath
pipx install amd-debug-tools
```
Log in and log out or open new console to apply the `PATH` variable changes.

Query the current shared memory configuration:
```
amd-ttm
```
Usually it is half of the overall system RAM.

Change the limit:
```
amd-ttm --set <GB>
```
This command asks for `sudo` password as it tunes the `ttm` module options and
adds it into the initramfs. After updating the parameters you need to reboot
the system.

## Download and try models

Go to [http://localhost:13305/](http://localhost:13305/) using your browser,
switch to the models tab on the left and download one of the "FastFlowLM NPU"
models to check if NPU works. You also can run any other models using ROCm
backend. See instruction above about shared memory tuning to be able running
models which are bigger than half of your RAM.

# Optional: Compile latest XDNA2 and Xilinx XRT

It is possible to compile the latest XDNA2 driver for the Linux kernels
starting from version 6.10. Unfortunately many stable kernels doesn't work with
this driver properly because of the security path which disables `SVA` on x86
platforms (see [XDNA2 issue
#1028](https://github.com/amd/xdna-driver/issues/1028) and [Linux kernel mail
list](https://lore.kernel.org/regressions/870872aa-28e9-412a-bac6-8020bf560e4f@amd.com/))
For example Debian 13 stable kernel (6.12.94) also has this issue it is the
reason why one needs to update it to the backport version. Usually compilation
is not required if Linux kernel 7.0 or above is used.

Clone XDNA2 driver repository with XRT submodule:
```
git clone --recurse-submodules https://github.com/amd/xdna-driver.git
```

Install XRT dependencies:
```
sudo xdna-driver/xrt/src/runtime_src/tools/scripts/xrtdeps.sh
```

Build and install XRT:
```
cd xdna-driver/xrt/build
./build.sh -npu -opt
sudo apt reinstall ./Release/xrt_*.deb
cd ../../..
```

Install XDNA2 driver dependencies:
```
sudo xdna-driver/tools/amdxdna_deps.sh
```

Build and install XDNA2 driver:
```
cd xdna-driver/build
./build.sh -release
sudo apt reinstall ./Release/xrt_plugin*-amdxdna.deb
cd ../..
```

# Links

- [Backports - Debian Wiki](https://wiki.debian.org/Backports#Using_the_command_line)
- [Lemonade Documentation](https://lemonade-server.ai/docs/)
- [Debian - Lemonade Server Documentation](https://lemonade-server.ai/docs/guide/install/debian/)
- [LLMs on Linux with FastFlowLM](https://lemonade-server.ai/flm_npu_linux.html)
- [FastFlowLM Documentation](https://fastflowlm.com/docs/)
- [AMD RDNA3.5 system optimization](https://rocm.docs.amd.com/en/latest/how-to/system-optimization/rdna3-5.html#configuring-shared-memory-limits-on-linux)
- [AMD XDNA2 driver](https://github.com/amd/xdna-driver)
- [Xilinx XRT library](https://github.com/Xilinx/XRT)
- [Building the XRT Software Stack](https://xilinx.github.io/XRT/master/html/build.html)
- [List of FastFlowLM models](https://fastflowlm.com/docs/models/)
- [List of Lemonade models](https://lemonade-server.ai/models.html)
