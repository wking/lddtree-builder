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

[ccon]: https://github.com/wking/ccon
[lddtree-homepage]: https://wiki.gentoo.org/wiki/Hardened/PaX_Utilities
[lddtree-source]: https://gitweb.gentoo.org/proj/pax-utils.git/tree/lddtree.py
[runtime-spec]: https://github.com/opencontainers/runtime-spec
