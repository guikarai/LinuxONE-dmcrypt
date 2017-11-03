## Using dm-crypt Volumes as LVM Physical Volumes

Pervasive Encryption for Linux provides
– fast and consumable data protection for data in-flight and data at-rest
– transparent fast encryption for data at rest
– extended security for data at-rest with protected key dm-crypt
– a trusted computing environment for sensitive appliances through Secure Service Containers

Objectives of the following is to describe how to migrate business data of a Linux environment from a clear to an encrypted file system. 

![Overview](https://github.com/guikarai/LinuxONE-dmcrypt/blob/master/data-migration-objective.png?raw=true)

### Existing configuration overview
Un docker et un container Redis qui rendent des services. Le file systeme docker est le Volume Group VG.

```
df -H
Filesystem                            Size  Used Avail Use% Mounted on
udev                                  940M     0  940M   0% /dev
tmpfs                                 189M  5.3M  184M   3% /run
/dev/disk/by-path/ccw-0.0.0100-part1  7.2G  1.8G  5.0G  27% /
tmpfs                                 945M     0  945M   0% /dev/shm
tmpfs                                 5.3M     0  5.3M   0% /run/lock
tmpfs                                 945M     0  945M   0% /sys/fs/cgroup
tmpfs                                 103k     0  103k   0% /run/lxcfs/controllers
tmpfs                                 189M     0  189M   0% /run/user/0
/dev/mapper/vg4730-lv4730             6.3G  141M  5.8G   3% /var/lib/docker
none                                  6.3G  141M  5.8G   3% /var/lib/docker/aufs/mnt/dab339d4999bf9f3f07794d75120ad741353630c475f7c54aac0b5158cced8aa
shm                                    68M     0   68M   0% /var/lib/docker/containers/524183a882a0131b14afa618ef0cf6736493153ad600ec759fb74e27108aa67c
 ```

```
vgdisplay
  --- Volume group ---
  VG Name               vg4730
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               6.00 GiB
  PE Size               4.00 MiB
  Total PE              1535
  Alloc PE / Size       1535 / 6.00 GiB
  Free  PE / Size       0 / 0
  VG UUID               DNDKh0-i7Bf-h5HP-JDdY-jrHV-qfPl-uNu5BN
```

```
lvdisplay
  --- Logical volume ---
  LV Path                /dev/vg4730/lv4730
  LV Name                lv4730
  VG Name                vg4730
  LV UUID                kHtJNf-kXcS-rALH-kS2B-5WLJ-w8Dq-NVJ3oX
  LV Write Access        read/write
  LV Creation host, time dmcrypto.mop.fr.ibm.com, 2017-11-02 15:13:49 +0100
  LV Status              available
  # open                 1
  LV Size                6.00 GiB
  Current LE             1535
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     1024
  Block device           252:0
```


```
pvdisplay
  --- Physical volume ---
  PV Name               /dev/dasde1
  VG Name               vg4730
  PV Size               6.00 GiB / not usable 4.52 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              1535
  Free PE               0
  Allocated PE          1535
  PV UUID               WgTHMe-0dU8-Fgz0-RQj3-Ed6b-6eIw-oBzE1M
 ``` 

#### Simple software stack
#### Existing Logical Volume
Use lvdisplay command as shown below, to view the available logical volumes with its attributes.
```
sudo lvdysplay
```
#### Acronysm for the following
* LV1 is
* VG is 
* DV1 is
* PV1 is
* V1 is

#### Reference Performance Footprint
root@dmcrypto:/var/lib/docker# docker exec -it redis bash
root@524183a882a0:/data# redis-benchmark -q -n 100000
PING_INLINE: 101832.99 requests per second
PING_BULK: 99304.87 requests per second
SET: 94428.70 requests per second
GET: 93808.63 requests per second
INCR: 92421.45 requests per second
LPUSH: 92506.94 requests per second
RPUSH: 96525.09 requests per second
LPOP: 95969.29 requests per second
RPOP: 98039.22 requests per second
SADD: 101317.12 requests per second
HSET: 97943.20 requests per second
SPOP: 102564.10 requests per second
LPUSH (needed to benchmark LRANGE): 89605.73 requests per second
LRANGE_100 (first 100 elements): 41736.23 requests per second
LRANGE_300 (first 300 elements): 16801.08 requests per second
LRANGE_500 (first 450 elements): 11799.41 requests per second
LRANGE_600 (first 600 elements): 9602.46 requests per second
MSET (10 keys): 73475.38 requests per second
 
#### Dismount the volume to be protected with dm-crypt
```
umount /dev/dasdd1
```

### Use DM-Crypt to Create an Encrypted Volume

#### Installing the dm-crypt Tools
You need to get the necessary tools by updating our local package index and installing the dm-crypt tools:
```
apt-get update
apt-get install cryptsetup
```
This will pull in all of the required dependencies and helper utilities needed to work with a dm-crypt volume.

#### 1) Create luks partition
```
cryptsetup luksFormat --hash=sha512 --key-size=512 --cipher=aes-xts-plain64 --verify-passphrase /dev/dasdd1
```
You can use different settings for the luksFormat command; I recommand to use aes-xts based ciphers for the best performance. After that you will be asked to enter a password for the encryption, it doesn't matter if it's not very secure now, because we will only use this device as random data generator.

#### 2) Open the encrypted device
```
cryptsetup luksOpen /dev/dasdd1 dasdd1_crypt
```
The command below opens the luks device and maps it as "dasdd1_crypt"

#### 3) Use the device as physical volume
```
pvcreate /dev/mapper/dasdd1_crypt
```
```
pvdisplay
  --- Physical volume ---
  PV Name               /dev/dasde1
  VG Name               vg4730
  PV Size               6.00 GiB / not usable 4.52 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              1535
  Free PE               0
  Allocated PE          1535
  PV UUID               WgTHMe-0dU8-Fgz0-RQj3-Ed6b-6eIw-oBzE1M

  "/dev/mapper/dasdd1_crypt" is a new physical volume of "6.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/mapper/dasdd1_crypt
  VG Name
  PV Size               6.00 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               osm0yD-gvOq-inc5-81a5-oR39-feh6-IyQ0G4
```

#### 4) Add the dm-crypt based physical volume to the existing volume group
```
vgextend VG /dev/mapper/dasdd1_crypt
```

#### 5) Migrate data clear to encrypted
```
pvmove /dev/mapper/dasdd1_clear /dev/mapper/dasdd1_crypt
```


