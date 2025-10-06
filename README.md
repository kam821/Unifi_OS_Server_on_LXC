# Unifi OS Server on LXC
### UniFI OS Server on LXC / Proxmox container

The inability to install UniFi OS Server on a LXC container is solely due to UOS installer implementation.

The installer aborts when attempting to set some sysctl parameters that are not absolutely necessary for its operation.

To ignore these errors, you can e.g. create a wrapper at /usr/local/sbin/sysctl, which will have higher priority in $PATH and will forward everything to the original sysctl, but ignore potential errors:

```
#!/bin/sh

/usr/sbin/sysctl $@ || true

exit 0
```
Then
```
chmod +x /usr/local/sbin/sysctl
```
then reboot or reconnect to the shell.

Additionally, you need to grant the LXC container the appropriate permissions so that the Podman can create the tun interface.

In the case of Proxmox, go to the Container Resources > add > Device Passthrough:
```
Advanced checked
Device path: /dev/net/tun
Access mode in CT: 0666
```
or

edit /etc/pve/lxc/CT_ID.conf and add:
```
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```
This way, you can successfully run UniFi OS Server installer on the LXC.

This instruction only work for privileged LXC container, it won't work on unprivileged container.
