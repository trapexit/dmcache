# dmcache

A tool to help enable setting up and using dm-cache.


### Usage

```
usage: dmcache [-h] {enable,disable,make,wipe-meta} ...

positional arguments:
  {enable,disable,make,wipe-meta}
      enable            setup caches
      disable           tear down and flush caches
      make              create cache logical volumes and corresponding dmctab
                        file
      wipe-meta         run after using 'make' to zero out meta partitions

optional arguments:
  -h, --help          show this help message and exit
```

### Setup

#### 0: Install tooling

Make sure that **python3**, **dmsetup**, **sfdisk**, and **dd** are installed.

#### 1: Partition cache drive

* cache-device = /dev/sda
* origin-devices = /dev/disk/by-label/{disk0,disk1,disk2}

```
$ sudo dmcache make -h
usage: dmcache make [-h] [-t {equal,proportional}] [-o DMCTAB]
                    [-m {default,writeback,writethrough}] [-b BLOCKSIZE] [-e]
                                        target sources [sources ...]

positional arguments:
  target                cache drive
  sources               origin devices

optional arguments:
  -h, --help            show this help message and exit
  -t {equal,proportional}
                        how to split up cache space among input drives
  -o DMCTAB             file to write dmctab to
  -m {default,writeback,writethrough}
                        cache mode to use
  -b BLOCKSIZE          cache block in 512b sectors (multiple of 64,
                        default=512)
  -e                    execute commands to setup cache drive

$ sudo dmcache -t proportional -e /dev/sda /dev/disk/by-label/{disk0,disk1,disk2}

<sets up the partitions on /dev/sda>
<generates /etc/dmctab>
```

#### 2: Wipe metadata partitions

```
$ sudo dmcache wipe-meta
Do you wish to wipe /dev/disk/by-partuuid/<uuid0>? [y/N] y
dd if=/dev/zero of=/dev/disk/by-partuuid/<uuid0>
dd: writing to '/dev/disk/by-partuuid/<uuid0>': No space left on device
12289+0 records in
12288+0 records out
6291456 bytes (6.3 MB, 6.0 MiB) copied, 0.134249 s, 46.9 MB/s

...
```

#### 3: Enable

##### 3.1: systemd

**TBD**

##### 3.2: rc.d

**TBD**

##### 3.3: manual

```
$ sudo dmcache enable -h
usage: dmcache enable [-h] [-f DMCTAB] [name [name ...]]

positional arguments:
  name        specific entries to enable

optional arguments:
  -h, --help  show this help message and exit
  -f DMCTAB   dmctab file to use (default: /etc/dmctab)

$ sudo dmcache enable
dmsetup create disk0 --table 0 73768960 cache /dev/disk/by-partuuid/<uuid0> /dev/disk/by-partuuid/<uuid1> /dev/disk/by-label/disk0 512 1 writeback default 0
dmsetup create disk1 --table 0 73768960 cache /dev/disk/by-partuuid/<uuid2> /dev/disk/by-partuuid/<uuid3> /dev/disk/by-label/disk1 512 1 writeback default 0
dmsetup create disk2 --table 0 73768960 cache /dev/disk/by-partuuid/<uuid4> /dev/disk/by-partuuid/<uuid5> /dev/disk/by-label/disk1 512 1 writeback default 0
```

#### 4: mount

```
$ sudo mount /dev/mapper/disk0 /mnt/disk0
$ sudo mount /dev/mapper/disk1 /mnt/disk1
$ sudo mount /dev/mapper/disk2 /mnt/disk2
```

#### 5: umount

```
$ sudo umount /mnt/disk0
$ sudo umount /mnt/disk1
$ sudo umount /mnt/disk2
```

#### 6: Disable

```
$ sudo dmcache disable
dmsetup suspend disk0
dmsetup reload disk0 --table 0 75952128 cache 8:23 8:24 8:4 512 1 writeback cleaner 0
dmsetup resume disk0
dmsetup remove disk0
dmsetup suspend disk1
dmsetup reload disk1 --table 0 75952128 cache 8:23 8:24 8:4 512 1 writeback cleaner 0
dmsetup resume disk1
dmsetup remove disk1
dmsetup suspend disk2
dmsetup reload disk2 --table 0 75952128 cache 8:23 8:24 8:4 512 1 writeback cleaner 0
dmsetup resume disk2
dmsetup remove disk2
```
