# BredOS Image build files.
[![Latest GitHub Release](https://img.shields.io/github/release/BredOS/images.svg?label=Latest%20Release)](https://github.com/BredOS/images/releases/latest)


These are the officially supported images.</br>
For the downstream / experimental images, check the [downstream](https://github.com/BredOS/images/tree/downstream) branch.</br>

## Building
The general steps and mechanisms are:
 - Clone this repository, which contains board configuration files for various SBCs.
 - get the mkimage.py script at [https://github.com/BredOS/mkimage](https://github.com/BredOS/mkimage), which ingests the board configuration files and creates the image.
 - the specific kernel will be installed from the BredOS repo with pacman in the board configuration files/`packages.aarch64` file
 - the repo and mirrorlist for the kernel are contained in the board configuration files


Then, for specific hardware(Instructions for building are in each subfolder):

For example, to build the opi5-image:
```
mkimage.py -w ./work/ -o ./out/ -c ./opi5-image/
```

For the fydetabduo-image:
```
mkimage.py -w ./work/ -o ./out/ -c ./fydetabduo-image
```
## development and testing
Steps to build and test the images:
- Clone the kernel repository at https://github.com/BredOS/linux-rockchip
- build the kernel package according to armbian's instructions
- 


In armbian, the steps are:
- Clone the build repo mentioned in https://zuckerbude.org/armbian-using-create-patches/
- the compile.sh script will download the kernel source and u-boot source
- cd into the source directory and make the changes
- the compile.sh can apply patches to the source


## the mkimage.py script
I copied the mkimage.py script from the [BredOS/mkimage](https://github.com/BredOS/mkimage/commit/88358575e61cb8b9452f70c6dcdc3f099074f255) repository to this repository for convenience. 
In addition, I added some comments to the script to make it easier to understand.

To create an image for your chosen SBC, run the mkimage.py script with the appropriate arguments. The basic usage is:

```bash
./mkimage.py -w /tmp/work -o ./output -c <board-cfg>
```

Where:
- `-w`: the working directory to use
- `-o`: the output directory for the resulting image
- `-c`: the board configuration file to use
    
## **WARNING:** If your system has less than 16 GB of RAM, it is recommended to use a different directory for the working directory, as using `/tmp/work` can cause performance issues due to the limited space in the `/tmp` directory.

For example, to create an image for the Rock 5 board, using the lxqt-rock5b-image configuration, with a working directory of /tmp/work and an output directory of ./output, you would run:

```bash
./mkimage.py -w /tmp/work -o ./output -c ./lxqt-rock5b-image
```
### analysis of the mkimage.py script
The mkimage.py script is a Python script that creates an image for a specific SBC. It uses the board configuration file to determine the necessary files and settings for the image.

steps:
1. **Parse Script Arguments**  
   - Use argparse to capture essential parameters like working directory, compression options, config directory, and output directory.

2. **Set Up Directories**  
   - Convert provided paths to absolute paths.
   - Ensure required directories (`work_dir`, `config_dir`, `out_dir`) are accessible.

3. **Check Root Permissions**  
   - Exit if the script is not run with root permissions.

4. **Define Logging Format**  
   - Set logging format and date style for script messages.

5. **Load Configuration (`profiledef.py`)**  
   - Verify if `config_dir` and `out_dir` exist.
   - Copy and import `profiledef.py` from `config_dir`.
   - Extract key settings, such as architecture, filesystem, and image details.

6. **Validate Configuration**  
   - Ensure valid settings for architecture, filesystem, image type, and image backend.
   - Confirm existence of the `packages` file for the given architecture.
   - Validate image name and version.

7. **Prepare Installation Directory**  
   - Create an installation directory inside `work_dir`.

8. **Read Package List**  
   - Read and parse the package file, excluding comments.

9. **Define Utility Functions**  
   - Functions include:
     - `runonce`: Check if a command has already been run.
     - `get_fsline`: Get the UUID of a filesystem.
     - `get_parttype`: Determine partition type for a device.
     - `realpath`: Get absolute path of an item.
     - `fixperms`: Fix ownership and permissions on files.

10. **Install Packages Using Pacstrap**  
    - Install specified packages into `install_dir` using pacstrap and configuration from `pacman_conf`.

11. **Create Image File**  
    - Create an image file with specified filesystem and size, attach it to a loop device.


## Board Configuration Files

Board configuration files are for a specific SBC. These files are located, for example, in the opi5-image or fydetabduo-image directories.

A different SBC need a new configuration file. You can use an existing configuration file as a starting point.

Below is the structure in each board configuration directory:
- packages.aarch64: a list of packages to be installed on the image, including the kernel.
- profiledef: a file containing basic information about the image, such as the version number, device, architecture, file system type, image name, image type, backend, and cmdline
- alarmimg/: the directory containing the basic system files for the image
- fixperms.sh: a shell script for fixing file permissions on the system files
- idbloader.img and u-boot.itb: files needed for U-Boot on Rock 5 boards
- pacman.conf.aarch64: the configuration file used to create the image itself

Some boards like the Rock 4C+ have more files, containing patches, or extra firmware.

Specifically, the `packages.aarch64` file contains:
- The kernel package, like `linux-rockchip-rkr3`. This package is the kernel for the Rock 5 board.
## Usage and details
Instructions for building in each subfolder.</br>
Build dependencies:
```
python-prettytable arch-install-scripts grub parted
```
</br>
› To build from x86_64, you need to install:

```
qemu-user-static-binfmt qemu-user-static
```

and run:
```
systemctl restart systemd-binfmt
```

</br>
Also make sure your system has the BredOS gpg keys and mirrorlist.

```
sudo pacman-key --recv-keys 77193F152BDBE6A6 BF0740F967BA439D DAEAD1E6D799C638
sudo pacman-key --lsign-key 77193F152BDBE6A6 BF0740F967BA439D DAEAD1E6D799C638
echo -e '# --> BredOS Mirrorlist <-- #\n\n# BredOS Main mirror\nServer = https://repo.bredos.org/repo/$repo/$arch\n' |sudo tee /etc/pacman.d/bredos-mirrorlist
```
