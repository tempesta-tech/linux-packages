# Packaging Linux Kernel for Debian

Supported Debian versions:
* 9 --- Stretch;
* 8 --- Jessie, with updates from jessie-backports.

Note, that Debian Jessie is supported only with backports updates installed due
to different packaging and dependencies schemes. To install updates add the
repo to `/etc/apt/sources.list`:
```sh
deb https://deb.debian.org/debian jessie-backports main contrib non-free
```
And install all the available updates:
```sh
apt-get update
apt-get -t jessie-backports dist-upgrade
```

## Prerequisites

Sources ropository must be exist in `/etc/apt/sources.list`:
```sh
# For Debian 9 Stretch
deb-src https://deb.debian.org/debian stretch main
# For Debian 8 Jessie
deb-src https://deb.debian.org/debian jessie-backports main
```
All the necessary packages can be installed with single command:
```sh
apt-get build-dep linux
```

Variables `DEBEMAIL` and `DEBFULLNAME` must be exported before updating
change logs. E.g.:
```sh
export DEBEMAIL="info@tempesta-tech.com"
export DEBFULLNAME="Tempesta Technologies, Inc."
```


## Packaging

Sadly, packaging Linux kernel is not so simple. It is packaged as foreign
package, so packaging directory contains a number of patches from Debian team.
Packaging system was forked from `git://anonscm.debian.org/kernel/linux.git`.

Copy `pkg/linux/debian` to separate empty directory. At least 25GB of free space
must be available.

Prepare _orig_ tarball - tarball that contains sources of original package.
Unpack it to package directory and apply patches from
`debian/pacthes/`. This can be done in one of the following ways:

- `uscan`. This command prepares origin tarball automatically, if original git
repo have tag with newer version than stated in  `debian/changelog`. Refer
to `debian/watch` for more info.

- `debian/rules get-orig-source && debian/rules orig` to make the same for the
current version.

Generate control file:
```sh
debian/rules debian/control
```

Finally create the set of the packages:
```sh
dpkg-buildpackage -uc -us -jN
```
Where `N` - number of concurrent jobs.

That will take a while. For more info refer to
[Rebuilding official Debian kernel packages](https://kernel-handbook.alioth.debian.org/ch-common-tasks.html#s-common-official)
manual.


# Publishing on Github

Once package is build, commit used to build package **must** be marked with
tag formatted as:
```
<distro_name>-<distro_version>/<sources_version>

e.g:
debian-9/0.5.0-1
debian-8/0.5.0-pre7
centos-7.2/0.5.0-1
```

After tag is pushed to Github, new release must be assigned to the tag. That
allows to publish packages and changelog in one place. Normally changelog
on Releases page should follow sources changelog.


# TODOs

1. Instead of publishing packages on Github Releases page, distribute software
to end users using custom Debian/Centos/other repositories.

2. When packaging _linux_, use vanilla sources and apply Tempesta-Tech patches
in the same way Debian/Centos maintainers applies their own patches. This
differs from current aproach, when the prepatched kernel is downloaded from
Tempesta-Tech kernel repo.

