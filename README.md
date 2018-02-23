# lddtree-builder

This project uses `lddtree` ([homepage][lddtree-homepage], [Git
source][lddtree-source]) to make it easy to construct filesystems for
[OCI containers][runtime-spec] and similar systems.  For example:

```console
$ lddtree-builder nginx
$ tree .
.
├── bin
├── dev
├── etc
│   └── resolv.conf
├── lib64
│   ├── ld-linux-x86-64.so.2
│   ├── libc.so.6
│   ├── libcom_err.so.2
│   ├── libcrypt.so.1
│   ├── libdl.so.2
│   ├── libkeyutils.so.1
│   ├── libpcre.so.1
│   ├── libpthread.so.0
│   ├── libresolv.so.2
│   └── libz.so.1
├── proc
├── run
├── sys
├── tmp
├── usr
│   ├── lib64
│   │   ├── libcrypto.so.1.0.0
│   │   ├── libk5crypto.so.3
│   │   ├── libkrb5.so.3
│   │   ├── libkrb5support.so.0
│   │   └── libssl.so.1.0.0
│   └── sbin
│       └── nginx
└── var

12 directories, 17 files
$ du -hs .
7.4M	.
```

Testing with [ccon][]:

```console
$ CONFIG=$(cat <<EOF
> {
>   "version": "0.4.0",
>   "namespaces": {
>     "user": {
>       "setgroups": false,
>       "uidMappings": [
>         {
>           "containerID": 0,
>           "hostID": 1000,
>           "size": 1
>         }
>       ],
>       "gidMappings": [
>         {
>           "containerID": 0,
>           "hostID": 1000,
>           "size": 1
>         }
>       ]
>     },
>     "mount": {
>       "mounts": [
>         {
>           "source": ".",
>           "target": ".",
>           "flags": [
>             "MS_BIND"
>           ]
>         },
>         {
>           "source": "/dev",
>           "target": "dev",
>           "flags": [
>             "MS_BIND",
>             "MS_REC"
>           ]
>         },
>         {
>           "target": "proc",
>           "type": "proc"
>         },
>         {
>           "source": "/sys",
>           "target": "sys",
>           "flags": [
>             "MS_BIND",
>             "MS_REC"
>           ]
>         },
>         {
>           "source": "/etc/resolv.conf",
>           "target": "etc/resolv.conf",
>           "flags": [
>             "MS_BIND"
>           ]
>         },
>         {
>           "source": ".",
>           "type": "pivot-root"
>         },
>         {
>           "target": "/",
>           "flags": [
>             "MS_REMOUNT",
>             "MS_RDONLY",
>             "MS_BIND"
>           ]
>         },
>         {
>           "target": "/run",
>           "type": "tmpfs"
>         },
>         {
>           "target": "/tmp",
>           "type": "tmpfs"
>         }
>       ]
>     },
>     "pid": {}
>   },
>   "process": {
>     "user": {
>       "uid": 0,
>       "gid": 0
>     },
>     "cwd": "/",
>     "capabilities": [],
>     "args": ["nginx", "-V"],
>     "env": [
>       "PATH=/usr/sbin"
>     ]
>   }
> }
> EOF
> )
$ ccon -s "${CONFIG}"
nginx version: nginx/1.12.2
…
```

If you're running from a [Gentoo][]-based system and have [Gentoolkit][] installed, you can also check the licenses of the content you've copied over:

```console
$ gentoo-licenses
packages:
  app-crypt/mit-krb5-1.15.2-r1: openafs-krb5-a BSD MIT OPENLDAP BSD-2 HPND BSD-4 ISC RSA CC-BY-SA-3.0 || ( BSD-2 GPL-2+ )
  dev-libs/libpcre-8.41-r1: BSD
  dev-libs/openssl-1.0.2n: openssl
  sys-apps/keyutils-1.5.9-r4: GPL-2 LGPL-2.1
  sys-libs/e2fsprogs-libs-1.43.6: GPL-2
  sys-libs/glibc-2.25-r9: LGPL-2.1+ BSD HPND ISC inner-net rc PCRE
  sys-libs/zlib-1.2.11-r1: ZLIB
  www-servers/nginx-1.12.2-r1: BSD-2 BSD SSLeay MIT GPL-2 GPL-2+
```

[ccon]: https://github.com/wking/ccon
[Gentoo]: https://gentoo.org/
[Gentoolkit]: https://wiki.gentoo.org/wiki/Gentoolkit
[lddtree-homepage]: https://wiki.gentoo.org/wiki/Hardened/PaX_Utilities
[lddtree-source]: https://gitweb.gentoo.org/proj/pax-utils.git/tree/lddtree.py
[runtime-spec]: https://github.com/opencontainers/runtime-spec
