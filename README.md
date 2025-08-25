# Bazzite-Discover-Sys-Ext
Instructions to add Plasma Discover package manager back into Bazzite using a Systemd Sys-Ext. Based on Travier's Fedora Sys-Ext work at https://travier.github.io/fedora-sysexts/ and relies on his base images on quay.

I restored Discover to my Bazzite install using a Systemd-Sysext. No shade to the bazaar and Bazzite devs, they do good work! 

I chose to use a Sysext as it is a more efficient way to add packages to an Atomic Distro than Layering, which requires the system to redownload and install the packages everytime you apply a new snapshot!

Adds back:
* Plasma Discover App
* rpm-ostree backend - this will allow you to fire system ostree snapshot updates from discover if you choose
* Access to KNew Stuff items
* Leaves Bazaar in place if you also want it available

**It's hard to break your system with this, but if you do, not my fault**

**I have fixed the segfaulting issue and updated the prepackaged version to 6.4.4 and reduced its size by building against Kinoite image istead of coreOS**

# Quick Install Instructions

I put my current raw file in the releases page, but I wont necessarily update it there and I'd prefer you built it yourself. 

Todo - do a build and install script

Enable sysexts if not already

`sudo install -d -m 0755 -o 0 -g 0 /var/lib/extensions /var/lib/extensions.d
sudo restorecon -RFv /var/lib/extensions /var/lib/extensions.d
sudo systemctl enable --now systemd-sysext.service`

Copy raw file from releases to correct folder

`sudo cp plasma-discover.raw /var/lib/extensions/plasma-discover.raw`

restart sysext service, or just reboot

`sudo systemctl restart systemd-sysext.service`



# Build the Sys-Ext

You will need a toolbox/Distrobox container unbound by selinux, I will not go into installing distrobox or toolbox here
```bash
distrobox create --name sysex --image registry.fedoraproject.org/fedora-toolbox:42 --additional-flags "--security-opt label=disable"
```

on your host (not the container) set selinux to permissive for good measure
```bash
sudo setenforce 0
```
Enter the distrobox container
```bash
distrobox enter sysex
```
install some dependencies
```bash
sudo dnf install git cpio erofs-utils jq just wget -y
```
You will now need to make the container issue commands to the host's podman binary.
```bash
cat > /usr/local/bin/podman
#!/bin/bash
executable="$(basename ${0})"
exec flatpak-spawn --host "${executable}" "${@}"

sudo chmod +x /usr/local/bin/podman
```

pull my simple just and container files
```bash
git clone https://github.com/mmcnutt/Bazzite-Discover-Sys-Ext.git

cd Bazzite-Discover-Sys-Ext/plasma-discover

just build quay.io/fedora-ostree-desktops/kinoite:42 x86_64

#just build quay.io/fedora-ostree-desktops/base-atomic:42 x86_64
```
If that build works you should get a raw disk image file called something like plasma-discover-6.4.3-1.fc42-42-x86-64.raw depending on what version Fedora is up to
```
mv plasma-discover-6.4.3-1.fc42-42-x86-64.raw plasma-discover.raw
```
# Enable Sys-Exts on your Bazzite System

```
sudo install -d -m 0755 -o 0 -g 0 /var/lib/extensions /var/lib/extensions.d
sudo restorecon -RFv /var/lib/extensions /var/lib/extensions.d
sudo systemctl enable --now systemd-sysext.service
```
and copy our disk image from earlier into /var/lib/extensions
```
sudo cp plasma-discover.raw /var/lib/extensions/
```
reboot and if all goes well, discover should be returned to your system

# Uninstall

Delete the raw file from /var/lib/extensions and then reboot

