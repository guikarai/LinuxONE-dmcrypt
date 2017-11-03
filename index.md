## Using dm-crypt Volumes as LVM Physical Volumes

Pervasive Encryption for Linux provides
– fast and consumable data protection for data in-flight and data at-rest
– transparent fast encryption for data at rest
– extended security for data at-rest with protected key dm-crypt
– a trusted computing environment for sensitive appliances through Secure Service Containers

Objectives of the following is to describe how to migrate business data of a Linux environment from a clear to an encrypted file system. 

![Overview](https://github.com/guikarai/LinuxONE-dmcrypt/blob/master/data-migration-objective.png?raw=true)

### Existing configuration overview
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
 vgdisplay
 

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

#### 4) Add the dm-crypt based physical volume to the existing volume group
```
vgextend VG /dev/mapper/dasdd1_crypt
```

#### 5) Migrate data clear to encrypted
```
pvmove /dev/mapper/dasdd1_clear /dev/mapper/dasdd1_crypt
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
