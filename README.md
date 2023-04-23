# ArkSys-ISO

ArkSys-ISO is an [ArchISO](https://wiki.archlinux.org/title/Archiso) profile to generate a disk ISO image. ArkSys-ISO is similar to [XeroLinux-ISO](https://github.com/xerolinux/xero_iso), a disk image with Arch Linux, KDE and Calamares installer to install the system in a graphical and easy way.

## How to build ArkSys-ISO

0. Install archiso tool with pacman, necessary to build an Arch Linux live ISO image.
```sh
sudo pacman -S archiso
```

1. Download arksys with git
```
git clone https://github.com/David7ce/arksys-iso.git
```

> If you want to build a basic Arch Linux system, copy the archiso basic profile `cp -r /usr/share/archiso/configs/releng/ archlive` and follow [archiso documentation](https://wiki.archlinux.org/title/Archiso).

2. To build your own ISO you need to create a work directory and an output directory for the ISO. Then us `mkarchiso` and do one of these:
```sh
# sudo mkarchiso -v -w  /path-to-dir/work -o /path-to-dir/ /path-to-dir/arksys-iso
```
> Tip: If memory allows, it is preferred to place the working directory on tmpfs (/tmp/archiso-tmp)

- To rebuild ISO, just remove files of work directory with`sudo rm -rf ./work/*`

## Calamares installer
If you want to install the system permanently on your machine you need something like Calamares installer, because archiso is a live environment that is onlye stored in RAM.

### Configure and build Calamares
There are two options to configure and build the framework installer:

- A. Download calamares from GitHub, just clone the github repo.
```sh
git clone https://github.com/calamares/calamares.git
```

After that you can configure the appareance (branding/) and the confiuration of the system inside (modules/.*conf) and the packages list (settings.conf). To view the tree structure of only .conf files use `tree -P "*.conf"`
```
./  # calamares directory
├── src/
|   ├── branding/
|   |   └── distroname/
|   └── modules/
|       └── *.conf
└── settings.conf
```

Once you have configured it you can build the app inside build directory.
```sh
mkdir calamares/build && cd calamares/build
cmake -DCMAKE_BUILD_TYPE=Debug .. # cmake ..
make
```

- B. For Arch you can use a package build (PKGBUILD) like some distributions do, for example [calamares-config of XeroLinux](https://github.com/xerolinux/calamares-cfg). Clone te repo, edit the PKBUILD with your configuration and build. To build use the make package command `makepkg`.

This will generate a compressed package `pkgname-pkgver-arch.pkg.tar.zst` that you can install with pacman or add it to your database of packages.
```sh
sudo pacman -U calamares-cfg-*.pkg.tar.zst
```
> To build and install the package you can also use `makepg -si`

### How install Calamares
Calamares can be installed using pacman from a repository (local or remote) or importing the necessary files in the system.

#### A. Calamares installation with pacman (recommended)
Calamares is not in the Arch repo, so you need to have your own repository, configure the mirrorlist, the keys and the database. This is beter if you plan on updating Calamares frequently. To download the package with pacman you can use an online or local repository:

- To use an **online repo** configure pacman.conf and add the repository of your calamares-installer package:
```
# XeroLinux Repository
#[valen_repo]
#SigLevel = Never
#Server = https://keyaedisa.github.io/$repo/$arch
```
And add the package name to `packages.x86_64`.

- To use a **local repo** you need to generate a database. The database is a tar file with the extension .db or .files followed by an archive extension of .tar, .tar.gz, .tar.bz2, .tar.xz, .tar.zst, .tar.Z.

```sh
# add a package to the database
repo-add /path/to/my_repo.db.tar.gz /path/to/package-1.0-1-x86_64.pkg.tar.zst

# add all packages in that path to the database
repo-add /path/to/my_repo.db.tar.gz /path/to/*.pkg.tar.zst
```

Use a text editor and add the name of the package (`package-1.0-1-x86_64.pkg.tar.zst`) to `packages.x86_64`. For example with the package `calamares-arksys-1-x86_64.pkg.tar.zst` add calamares-arksys to the list of packages.

Then add the database to the pacman.conf
```
[my_repo]
SigLevel = Optional TrustAll
Server = file:///Workspaces/repos/ArkSys-project/arksys-repo
```

> If you choose another "reponame.db.tar.gz" the Server = file:///Workspaces/repos/ArkSys-project/reponame.db


## Tree of archiso (important files)
```
./
├── airootfs/
│   ├── etc/
│   │   ├── calamares/
│   │   │   └── settings.conf
│   │   ├── gshadow
│   │   ├── hostname
│   │   ├── locale.conf
│   │   ├── mkinitcpio.conf
│   │   │   └── broadcom-wl.conf
│   │   ├── motd
│   │   ├── pacman.d
│   │   │   └── mirrorlist
│   │   ├── passwd
│   │   ├── resolv.conf -> /run/systemd/resolve/stub-resolv.conf
│   │   ├── shadow
│   │   ├── ssh/
│   │   │   └── sshd_config
│   │   ├── systemd/
│   │   │   ├── journald.conf.d
│   │   │   │   └── volatile-storage.conf
│   │   │   ├── logind.conf.d
│   │   │   │   └── do-not-suspend.conf
│   │   │   ├── network
│   │   │   ├── system
│   │   │   │   ├── getty@tty1.service.d
│   │   │   │   │   └── autologin.conf
│   │   │   │   ├── reflector.service.d
│   │   │   │   │   └── archiso.conf
│   │   │   │   └── systemd-networkd-wait-online.service.d
│   │   │   │       └── wait-for-only-one-interface.conf
│   │   │   └── system-generators
│   │   └── xdg/
│   │       └── reflector
│   │           └── reflector.conf
│   ├── root/
│   │   ├── custom/  # --- added ---
│   │   │   ├── customize_rootfs.sh
│   │   │   └── packages.both
│   │   └── run_archiso.sh
│   └── usr
├── bootstrap_packages.x86_64
├── efiboot/
│   └── loader
│       ├── entries
│       │   ├── 01-archiso-x86_64-linux.conf
│       │   └── 02-archiso-x86_64-speech-linux.conf
│       └── loader.conf
├── grub/
│   └── grub.cfg
├── packages.x86_64
├── pacman.conf
├── profiledef.sh
├── syslinux/
│   └── splash.png
└── README.md
```
