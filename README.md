# ImgGuard

This `imgguard` python script provides a easy and safe way to mount qcow2 images on your host machine. 

## Usage

### Pre-requisites:
   - `nbd` kernel module must have been loaded
   - `qemu-nbd` and `qemu-img` must be installed


### 1. Mounting the image


```zsh
# note that this script must be run as root

sudo imgguard /path/to/img.qcow2 /path/to/mountpoint

```

The script implements a serialized mount manager to automatically choose available nbd device node to connect. It also would prevent the user from re-mounting the same image or mounting different images on the same mountpoint.

### 2. Unmounting the image

Now you would see that the scripts hangs itself and waits for SIGINT or SIGTERM. When you have done your work, you can just press `Ctrl+C` to interrupt the script, and it will safely unmount the image.

## Installation

Arch Linux users can just build and install the `codeback` package using the provided `PKGBUILD`.

## Some other notes

#### 1. What the script would do:

  - This script essentially executes the following commands for mounting:
  ```zsh
  sudo qemu-nbd -c /dev/nbdX /path/to/img.qcow2
  sudo partprobe /dev/nbdX
  sudo mount /dev/nbdX /path/to/mountpoint
  ```

  - And for unmounting:
  ```zsh
  sudo umount /path/to/mountpoint
  sudo qemu-nbd -d /dev/nbdX
  ```

  - And checking:
  ```zsh
  qemu-img check /path/to/img.qcow2
  ```

#### 2. Compressing the saved image

  - The script could also help you compressing the saved image. Just simple call with `-c` option:

  ```zsh
  sudo imgguard -c /path/to/img.qcow2 /path/to/mountpoint
  ```

  - After you interrupt the script, it will compress the image and save it as `/path/to/img.compressed.qcow2` in the same directory.

#### 3. Tips for making a ext4-qcow2 image:

```zsh
qemu-img create -f qcow2 /path/to/img.qcow2 10G
sudo qemu-nbd -c /dev/nbdX /path/to/img.qcow2
sudo mkfs.ext4 /dev/nbdX
sudo qemu-nbd -d /dev/nbdX
```

There is also a bundle script `mkimg` to automate the process:

```zsh
mkimg /path/to/img.qcow2 10G
```