## Using dm-crypt Volumes as LVM Physical Volumes

Pervasive Encryption for Linux provides
– fast and consumable data protection for data in-flight and data at-rest
– transparent fast encryption for data at rest
– extended security for data at-rest with protected key dm-crypt
– a trusted computing environment for sensitive appliances through Secure Service Containers

Objectives of the following is to describe how to migrate business data of a Linux environment from a clear to an encrypted file system.
![GitHub Logo](/images/logo.png)

### Existing configuration overview

#### Simple software stack
#### Existing Logical Volume (LV1 and LV2)
Use lvdisplay command as shown below, to view the available logical volumes with its attributes.
```
sudo lvdysplay
```
#### Existing Volume Group (VG)
#### Existing Physical Device (PV1)

#### Installing the dm-crypt Tools
You need to get the necessary tools by updating our local package index and installing the dm-crypt tools:
```
apt-get update
apt-get install cryptsetup
```
This will pull in all of the required dependencies and helper utilities needed to work with a dm-crypt volume.

#### Dismount the volume to be protected with dm-crypt
```
umount /dev/dasdd1
```

### Use DM-Crypt to Create an Encrypted Volume

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

#### 3) Use the device as physical volume (PV2)
```
pvcreate /dev/mapper/dasdd1_crypt
```

#### 4) Add the dm-crypt based physical volume (PV) to the existing volume group (VG)
```
vgextend VG /dev/mapper/dasdd1_crypt
```

#### 5) Migrate data from V1 to V2
```
pvmove /dev/mapper/dasdd1_clear /dev/mapper/dasdd1_crypt
```

#### Summary
LV1 -
LV2 -
VG -
PV1 -
PV2 -
DMV -
V1 -
V2 -
