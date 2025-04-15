# SqueezeDisk

SqueezeDisk device driver / device-mapper

Usage,

`make [KDOOT=linux_kernel_dir] [ovbd-loop|ovbd-dm]`

`KROOT` set as kernel module develop kit path, ignored to use host kernel build environment

## ovbd-loop

This is a basic repo for SqueezeDisk block-device driver,
currently implemented like loop-device in kernel, reading image file via vfs to feed bio
read requests.

### Build

`make ovbd-loop`

Or just 

`make`

### Use

The image file should be LSMT-File on ZFile(__without Tar file header currently.__), image path can be configured by parameter.

`insmod ./vbd.ko backfile=<absolute path0>,<absolute path1>,<absolute path2>...`

if succeed, a device called `/dev/vbd0` should appeared, 

`mount /dev/vdb0 <mount point>`

## ovbd-dm

This is another example, driven in device-mapper framework, take a device which is filled by LSMT-File blobs.

### Build

To build it, using
`make ovbd-dm`

To use it, currently should know the block-device size. (Which should be read by LSMT-File tailer, but the cli tools are not ready by now).

### Use

First of all, install the module

`insmod vbd.ko`

It can be simply get ready by loop device:

`losetup -f --show -r --direct-io <LSMTFile absolute path>`

Say the loop device is `/dev/loop0` for example. Now able to create mapped device using `dmsetup`

`dmsetup create --concise "zfile0,,,ro,0 <zfile unzip size> zfile_target /dev/loop0 <lsmt actual size>"`

`dmsetup create --concise "lsmt0,,,ro,0 2290872 lsmt_target /dev/zfile0 <lsmt actual size>"`

Here the `vbd0` is device name for mapped-device (as `/dev/mapper/vbd0`). 
In table part, there are tow target types `zfile_target` and `lsmt_target`, then follows underlay lsmt format block parameter, path and length in bytes.

After the mapped-device ready, it could able to mount

`mount /dev/mapper/vbd0 <mount point>`
