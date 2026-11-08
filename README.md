# pearOS Generic GPU Driver (virtio)

`virtio_gpu` for FreeBSD (the module and DRM driver name stay `virtio_gpu`; the description it reports is "pearOS Generic GPU Driver (virtio)").

A port of Linux's `virtio_gpu` DRM driver (6.1.128) to FreeBSD 14 through LinuxKPI, so that `/dev/dri/card0` and `renderD128` exist on a QEMU `virtio-vga` / `virtio-vga-gl` (virgl) and Mesa's `virgl` driver can accelerate OpenGL in a VM. Used by [pearOS](https://github.com/pearOS-bsd/misc) (`wsd`, the window server).

Status, honestly: it works (KMS display, hotplug/resize, GL through virgl renders pixel-identical to llvmpipe in the test of `wsbsd/test/glcmp.c`), but with GL on virgl and the dumb-buffer scan-out each frame is read back, so it is **slower than a software compositor** in the VM we measured (39 against 65 FPS, 1080p, 8 glass windows). Zero-copy scan-out would need PRIME import and blob resources, which are missing. See `PORTING.md` (design, milestone log and "Corrections").

Build (inside a checkout of [freebsd/drm-kmod](https://github.com/freebsd/drm-kmod) branch `6.1-lts`):

    cp -r <this repo> drm-kmod/virtio_gpu
    cd drm-kmod/virtio_gpu && make SYSDIR=/usr/src/sys
    devctl detach virtio_pci1      # the device must be free (find N with pciconf -l, vendor 0x1af4 device 0x1050)
    kldload ./virtio_gpu.ko

Do not `kldunload` it (a later `kldload` panics on duplicate devfs aliases; BAR mappings leak on unload).

License: GPL-2.0-only (Linux code, kept as is), like drm-kmod.
