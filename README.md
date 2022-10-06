# ycrc_vglrun
A wrapper script for vglrun used at YCRC

# Requirement

1. NVIDIA GPU
1. TurboVNC server
1. VirutalGL: version 3.0 and above 

# Motivation

There are three methods of setting up VirtualGL for remote visualization, based on the discussion in 
this [link](https://discourse.openondemand.org/t/virtualgl-in-ood-apps/750).
They are listed in the table below with pros and cons.

| Method   |      Pros    | Cons        |
|----------|--------------|-------------|
|Starting X server by root at boot time   | - Simple to implement <br> - Easy to use | - Potential security concerns. <br> - X server eats up part of GPU resources even nobody is using it. | 
|Starting X server as the user in SLURM TaskProlog | - X server is launched as the user when needed <br> - It is controlled by the user's Cgroup | - Difficult to implement |
|Direct access to GPU via the EGL API   | - Simple to implement  <br> - No need to start X server  |  - Some applications may not render properly |

The wrapper script tries to provide a solution that combines all the advantages of the three methods, while keeping their disadvantages minimal.

# How to use

See our VirtualGL [user guide](https://docs.ycrc.yale.edu/clusters-at-yale/guides/virtualgl/).

# How to set up VirtualGL

1. Install the NVIDIA driver
1. Configure the NVIDIA GPU for display
```{bash}
nvidia-xconfig -a
```
3. Configure VirtualGL server
```{bash}
vglserver_config
```
4. Modify PAM xserver to allow users to start an X server from a non-consol shell session 
```{bash}
$ cat /etc/pam.d/xserver 
#%PAM-1.0
auth       sufficient    pam_rootok.so
#auth       required    pam_console.so     # remove this 
auth       required     pam_permit.so      # add this
account    required    pam_permit.so
session    optional    pam_keyinit.so force revoke
```
# Sample `vglserver_config` configuration

```{bash}
1) Configure server for use with VirtualGL (GLX + EGL back ends)
2) Unconfigure server for use with VirtualGL (GLX + EGL back ends)
3) Configure server for use with VirtualGL (EGL back end only)
4) Unconfigure server for use with VirtualGL (EGL back end only)
X) Exit
Choose:
1
Restrict 3D X server access to vglusers group (recommended)?
[Y/n]
n
Restrict framebuffer device access to vglusers group (recommended)?
[Y/n]
n
Disable XTEST extension (recommended)?
[Y/n]
Y
... Modifying /etc/security/console.perms to disable automatic permissions
    for DRI devices ...
... Creating /etc/modprobe.d/virtualgl.conf to set requested permissions for
    /dev/nvidia* ...
... Attempting to remove nvidia module from memory so device permissions
    will be reloaded ...
rmmod: ERROR: Module nvidia is in use by: nv_peer_mem gdrdrv nvidia_modeset nvidia_uvm
... Granting write permission to /dev/nvidia0 /dev/nvidia1 /dev/nvidia2 /dev/nvidia3 /dev/nvidiactl /dev/nvidia-modeset /dev/nvidia-uvm /dev/nvidia-uvm-tools for all users ...
... Granting write permission to /dev/dri/card0 /dev/dri/card1 /dev/dri/card2 /dev/dri/card3 for all users ...
... Granting write permission to /dev/dri/renderD128 /dev/dri/renderD129 /dev/dri/renderD130 /dev/dri/renderD131 for all users ...
... Modifying /etc/X11/xorg.conf.d/99-virtualgl-dri to enable DRI permissions
    for all users ...
... /etc/gdm/Init/Default has been saved as /etc/gdm/Init/Default.orig.vgl ...
... Adding xhost +LOCAL: to /etc/gdm/Init/Default script ...
... Creating /usr/share/gdm/greeter/autostart/virtualgl.desktop ...
... /etc/gdm/custom.conf has been saved as /etc/gdm/custom.conf.orig.vgl ...
... Disabling XTEST extension in /etc/gdm/custom.conf ...
... Setting default run level to 5 (enabling graphical login prompt) ...
... Commenting out DisallowTCP line (if it exists) in /etc/gdm/custom.conf ...

Done. You must restart the display manager for the changes to take effect.

IMPORTANT NOTE: Your system uses modprobe.d to set device permissions.  You
must execute 'rmmod nvidia_drm nvidia_modeset nvidia' with the display manager
stopped in order for the new device permission settings to become effective.

1) Configure server for use with VirtualGL (GLX + EGL back ends)
2) Unconfigure server for use with VirtualGL (GLX + EGL back ends)
3) Configure server for use with VirtualGL (EGL back end only)
4) Unconfigure server for use with VirtualGL (EGL back end only)
X) Exit
Choose:
X
```
