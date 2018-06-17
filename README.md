# freebsd-mkmnfst
Basic MANIFEST generator for building `pkg-create(8)` packages

## Disclaimer
The recommended and "official" FreeBSD way of building and maintaining packages is through the Ports Collection, and is described in [Porter's Handbook](https://www.freebsd.org/doc/en/books/porters-handbook/index.html). If you're building packages for anyone other than yourself, you should probably stick with that approach.

## Description
The `mkmnfst` script generates the minimal viable MANIFEST file, which can be used to build packages using `pkg-create(8)`. It expects one or more (exact) package names as arguments, which will be resolved as dependencies upon installation. Refer to the manual page of `pkg-create(8)` for information on how to extend the generated MANIFEST file for more advanced uses, such as packaging and distributing arbitrary files with your package.

## Example usage
Let's assume you want to build a very basic package that installs other packages (i.e a metapackage), say `bash`, `tmux`, `curl` and `vim-tiny`:

```
$ ./mkmnfst bash tmux curl vim-tiny > basic_metapkg
```

The `basic_metapkg` at this point contains only the dependency definition and the minimal valid structure required to produce a package. You still need to edit it and replace the `FIXME` placeholders:

```
$ cat basic_metapkg 
name: FIXME
version: FIXME
origin: FIXME
comment: FIXME
www: FIXME
maintainer: FIXME
prefix: /usr/local
desc: <<EOD
    FIXME
EOD
deps: {
    bash: {origin: shells/bash, version: 4.4.19},
    tmux: {origin: sysutils/tmux, version: 2.6_1},
    curl: {origin: ftp/curl, version: 7.60.0},
    vim-tiny: {origin: editors/vim-tiny, version: 8.0.1638},
}
``` 

Eventually, you would end-up with something in the lines of the following:

```
name: dep-basic
version: 20180617_01
origin: local
comment: Metapackage for basic dependency resolution
www: http://www.example.com
maintainer: email@example.com
prefix: /usr/local
desc: <<EOD
    Metapackage for resolution of basic dependencies.

    This package installs the following applications:
    - bash
    - tmux
    - curl
    - vim-tiny
EOD
deps: {
    bash: {origin: shells/bash, version: 4.4.19},
    tmux: {origin: sysutils/tmux, version: 2.6_1},
    curl: {origin: ftp/curl, version: 7.60.0},
    vim-tiny: {origin: editors/vim-tiny, version: 8.0.1638},
}

```

Now that you have a valid MANIFEST file, you can use it to produce a package with `pkg-create(8)`:

```
$ pkg create -M ./basic_metapkg
```

You should now have `dep-basic-20180617_01.txz` in your current working directory. You can use this package as any other FreeBSD package:

```
$ pkg info -F dep-basic-20180617_01.txz 
dep-basic-20180617_01
Name           : dep-basic
Version        : 20180617_01
Origin         : local
Architecture   : FreeBSD:11:amd64
Prefix         : /usr/local
Categories     : 
Licenses       : 
Maintainer     : email@example.com
WWW            : http://www.example.com
Comment        : Metapackage for basic dependency resolution
Annotations    :
	FreeBSD_version: 1101001
Flat size      : 0.00B
Description    :
Metapackage for resolution of basic dependencies.

    This package installs the following applications:
    - bash
    - tmux
    - curl
    - vim-tiny
```

At this point, you can either copy the package to your custom repository (preferred), or just distribute it to your systems and install it directly:

```
$ pkg install ./dep-basic-20180617_01.txz 
Updating FreeBSD repository catalogue...
FreeBSD repository is up to date.
All repositories are up to date.
Checking integrity... done (0 conflicting)
The following 10 package(s) will be affected (of 0 checked):

New packages to be INSTALLED:
	dep-basic: 20180617_01
	bash: 4.4.19
	indexinfo: 0.3.1
	gettext-runtime: 0.19.8.1_1
	tmux: 2.6_1
	libevent: 2.1.8_1
	curl: 7.60.0
	libnghttp2: 1.31.1
	ca_root_nss: 3.37.3
	vim-tiny: 8.0.1638

Number of packages to be installed: 10

The process will require 19 MiB more space.

Proceed with this action? [y/N]: y
[pkg] [1/1] Installing indexinfo-0.3.1...
[pkg] Extracting indexinfo-0.3.1: 100%
[pkg] [1/10] Installing gettext-runtime-0.19.8.1_1...
[pkg] [1/10] Extracting gettext-runtime-0.19.8.1_1: 100%
[pkg] [2/10] Installing libevent-2.1.8_1...
[pkg] [2/10] Extracting libevent-2.1.8_1: 100%
[pkg] [3/10] Installing libnghttp2-1.31.1...
[pkg] [3/10] Extracting libnghttp2-1.31.1: 100%
[pkg] [4/10] Installing ca_root_nss-3.37.3...
[pkg] [4/10] Extracting ca_root_nss-3.37.3: 100%
[pkg] [5/10] Installing bash-4.4.19...
[pkg] [5/10] Extracting bash-4.4.19: 100%
[pkg] [6/10] Installing tmux-2.6_1...
[pkg] [6/10] Extracting tmux-2.6_1: 100%
[pkg] [7/10] Installing curl-7.60.0...
[pkg] [7/10] Extracting curl-7.60.0: 100%
[pkg] [8/10] Installing vim-tiny-8.0.1638...
[pkg] [8/10] Extracting vim-tiny-8.0.1638: 100%
[pkg] [9/10] Installing dep-basic-20180617_01...
```
