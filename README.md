# Bazzite-Discover-Sys-Ext
Instructions to add Plasma Discover package manager back into Bazzite using a Systemd Sys-Ext. Based on Travier's Fedora Sys-Ext work at https://travier.github.io/fedora-sysexts/ and relies on his base images on quay.

I restored Discover to my Bazzite install using a Systemd-Sysext. Essentially a sub disk image that is loaded during boot of the immutable system, that way you dont have to layer all the plasma components this would usually require! Layering 400mb worth of plasma stuff everytime you do an update to Bazzite is not fun and slows your updates down. Smarter people than me can explain it better

No shade to the bazaar devs, but on plasma it looks really out of place!

**It's hard to break your system with this, but if you do, not my fault**

**since the last bazzite compose my test machine segfaults discover about 50% of the time, not sure why yet**

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

just build quay.io/fedora-ostree-desktops/base-atomic:42 x86_64
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

# This is too hard

I put my current raw file in the releases page, but I wont necessarily update it there and I'd prefer you built it yourself. It's good digital hygene and you just might learn a thing
