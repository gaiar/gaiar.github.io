# Debootstrap :: Debian for Lichee Pi Nano


In this post we will explain how you can create your own Debian rootfs with pre-installed packages of your choice, which to allow tiny Linux images to be created.

All steps below should work on any Debian host (Debian/Ubuntu etc) and are verified with Ubuntu 16.04 LTS.

First of all you need to install the support packages on your pc

```bash
sudo apt-get install qemu-user-static debootstrap binfmt-support
```


Next you need to choose the version of Debian in this case we are building a wheezy image.

```bash
targetdir=debian
distro=stretch
```


Now we will build first stage of Debian rootfs :

```bash
mkdir $targetdir
sudo debootstrap --arch=armel --foreign $distro $targetdir
```


Next copy the `qemu-arm-static` binary into the right place for the binfmt packages to find it and copy in `resolv.conf` from the host.

```bash
sudo cp /usr/bin/qemu-arm-static $targetdir/usr/bin/
sudo cp /etc/resolv.conf $targetdir/etc
```


If everything is right we now have a minimal Debian Rootfs

```bash
sudo chroot $targetdir
```


Inside the chroot we need to set up the environment again

```bash
distro=stretch
export LANG=C
```


Now we are setup the second stage of debootstrap needs to run install the packages downloaded earlier

```bash
/debootstrap/debootstrap --second-stage
```


Once the package installation has finished, setup some support files and apt configuration.


```bash
cat <<EOT > /etc/apt/sources.list
deb http://ftp.uk.debian.org/debian $distro main contrib non-free
deb-src http://ftp.uk.debian.org/debian $distro main contrib non-free
deb http://ftp.uk.debian.org/debian $distro-updates main contrib non-free
deb-src http://ftp.uk.debian.org/debian $distro-updates main contrib non-free
deb http://security.debian.org/debian-security $distro/updates main contrib non-free
deb-src http://security.debian.org/debian-security $distro/updates main contrib non-free
EOT
```

Update Debian package database:

```bash
apt-get update
```

set up locales dpkg scripts tend to complain otherwise, note in jessie you will also need to install the dialog package as well.


```bash
apt-get install locales dialog
dpkg-reconfigure locales
```


Install some useful packages inside the chroot

```bash
apt-get install openssh-server ntpdate
```


Set a root password so you can login

```bash
passwd
```


Build a basic network interface file so that the board will DHCP on eth0

```bash
echo <<EOT >> /etc/network/interfaces
allow-hotplug eth0
iface eth0 inet static
address 192.168.1.254
netmask 255.255.255.248
gateway 192.168.1.1
EOT
```


*Note:* Your board will be accessible over SSH on IP address defined above !



Set the hostname

```bash
echo nameme > /etc/hostname
```


Enable the serial console, Debian sysvinit way

```bash
echo T0:2345:respawn:/sbin/getty -L ttyS0 115200 vt100 >> /etc/inittab
```


We are done inside the chroot, so quit the chroot shell

```bash
exit
```


Tidy up the support files

```bash
sudo rm $targetdir/etc/resolv.conf
sudo rm $targetdir/usr/bin/qemu-arm-static
```


Now you have your Debian rootfs. Next step is to build Kernel, Uboot and to make your SD-card as explained in our early posts and Build instructions but instead to use the rootfs in the posts you can use your own minimal rootfs which you created above. The rootfs image created above is approx 150MB, it could be made smaller if you remove more packages.


Special thanks to Dimitar Gamishev (aka HEHOPMAJIEH) for creating this tutorial.