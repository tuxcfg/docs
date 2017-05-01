Disk encryption
===============

Partitions:

* `/dev/sda1` - [EFI system partition](https://en.wikipedia.org/wiki/EFI_system_partition) (fat32, boot, esp, 128Mb)
* `/dev/sda2` - boot partition (ext4, 512Mb)
* `/dev/sda3` - LUKS partition

Fill the device with random data:

```bash
sudo dd if=/dev/urandom of=/dev/sda3 bs=1M status=progress
```

Alternative disk random filling approach:

```bash
sudo cryptsetup luksFormat /dev/sda3
sudo cryptsetup luksOpen /dev/sda3 crypto
sudo dd if=/dev/zero of=/dev/mapper/crypto bs=8M status=progress
```

Init crypto partition:

```bash
sudo cryptsetup --key-size=512 --hash=sha256 --iter-time=3000 luksFormat /dev/sda3
```

Open partition:

```bash
sudo cryptsetup luksOpen /dev/sda3 crypto
```

Use the device as physical volume:

```bash
sudo lvm pvcreate /dev/mapper/crypto
```

```bash
sudo lvm vgcreate crypto /dev/mapper/crypto
```

Create the logical volumes:

```bash
sudo lvm lvcreate --size 8G crypto --name swap
sudo lvm lvcreate --size 20G crypto --name root
sudo lvm lvcreate --size 2G crypto --name logs
sudo lvm lvcreate --extents 100%FREE crypto --name home
```

Create file systems and set labels:

```bash
sudo mkswap /dev/mapper/crypto-swap
sudo mkfs.ext4 /dev/mapper/crypto-root
sudo e2label /dev/mapper/crypto-root root
sudo mkfs.ext4 /dev/mapper/crypto-logs
sudo e2label /dev/mapper/crypto-logs logs
sudo mkfs.ext4 /dev/mapper/crypto-home
sudo e2label /dev/mapper/crypto-home home
sync
```

Install operating system, do not reboot and then finalize setup:

```bash
sudo mount /dev/mapper/crypto-root /mnt
sudo mount /dev/sda2 /mnt/boot
sudo mount -o bind /dev /mnt/dev
sudo mount -t proc proc /mnt/proc
sudo mount -t sysfs sys /mnt/sys
sudo chroot /mnt /bin/bash
echo "crypto UUID=`blkid -s UUID -o value /dev/sda3` none luks,discard" >> /etc/crypttab
update-initramfs -u -k all
exit
```

Useful links:

* [Self-Encrypting Drives](https://wiki.archlinux.org/index.php/Self-Encrypting_Drives)
* [Drive Trust Alliance](https://github.com/Drive-Trust-Alliance)
* [Use the hardware-based full disk encryption of your TCG Opal SSD with msed](https://vxlabs.com/2015/02/11/use-the-hardware-based-full-disk-encryption-your-tcg-opal-ssd-with-msed/)
* [Guide to Full Disk Encryption with Ubuntu](http://thesimplecomputer.info/full-disk-encryption-with-ubuntu)
* [dm-crypt/Device encryption](https://wiki.archlinux.org/index.php/Dm-crypt/Device_encryption)
