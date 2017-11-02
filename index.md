## Using dm-crypt Volumes as LVM Physical Volumes

Pervasive Encryption for Linux provides
– fast and consumable data protection for data in-flight and data at-rest
– transparent fast encryption for data at rest
– extended security for data at-rest with protected key dm-crypt
– a trusted computing environment for sensitive appliances through Secure Service Containers

Objectives of the following is to describe how to migrate business data of a Linux environment from clear to encrypted file system.

### Existing configuration overview

#### Simple software stack
#### VG
#### PV
#### Dismount the disk
umount /dev/dasdd1

### Use DM-Crypt to Create an Encrypted Volume

#### 1) Create luks partition
cryptsetup luksFormat --hash=sha512 --key-size=512 --cipher=aes-xts-plain64 --verify-passphrase /dev/dasdd1

#### 2) Open the encrypted device: the command below opens the luks device and maps it as "sda_crypt"
cryptsetup luksOpen /dev/sda sda_crypt

#### 
#### 


Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/guikarai/LinuxONE-dmcrypt/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
