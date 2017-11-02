## Using dm-crypt Volumes as LVM Physical Volumes

Pervasive Encryption for Linux provides
– fast and consumable data protection for data in-flight and data at-rest
– transparent fast encryption for data at rest
– extended security for data at-rest with protected key dm-crypt
– a trusted computing environment for sensitive appliances through Secure Service Containers

Objectives of the following is to describe how to migrate business data of a Linux environment from clear to encrypted file system.

### Existing configuration overview

#### Simple software stack
#### Existing Logical Volume (LV1 and LV2)
#### Existing Volume Group (VG)
#### Existing Physical Device (PV1)
#### Dismount the volume to be protected with dm-crypt
umount /dev/dasdd1

### Use DM-Crypt to Create an Encrypted Volume

#### 1) Create luks partition
cryptsetup luksFormat --hash=sha512 --key-size=512 --cipher=aes-xts-plain64 --verify-passphrase /dev/dasdd1

#### 2) Open the encrypted device: the command below opens the luks device and maps it as "dasdd1_crypt"
cryptsetup luksOpen /dev/dasdd1 dasdd1_crypt

#### 3) Use the device as physical volume (PV2)
pvcreate /dev/mapper/dasdd1_crypt

#### 4) Add the dm-crypt based physical volume (PV) to the existing volume group (VG)
vgextend VG /dev/mapper/dasdd1_crypt

#### 5) Migrate data from V1 to V2
