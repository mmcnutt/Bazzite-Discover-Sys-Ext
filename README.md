# Bazzite-Discover-Sys-Ext
Instructions to add Plasma Discover package manager back into Bazzite using a Systemd Sys-Ext. Based on Travier's Fedora Sys-Ext work at https://travier.github.io/fedora-sysexts/ 

I restored Discover to my Bazzite install using a Systemd-Sysext. Essentially a sub disk image that is loaded during boot of the immutable system, that way you dont have to layer all the plasma components this would usually require! Smarter people than me can explain it better

No shade to the bazaar devs, but on plasma it looks really out of place!

# Build the Sys-Ext

You will need a Distrobox container unbound by selinux
```bash
distrobox create --name sysex --image registry.fedoraproject.org/fedora-toolbox:42 --additional-flags "--security-opt label=disable"
```

on your host (not the container) set selinux to permissive for good measure
```bash
sudo setenforce 0

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
git https://github.com/mmcnutt/Bazzite-Discover-Sys-Ext.git

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
