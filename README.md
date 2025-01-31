# try

<img src="docs/try_logo.png" alt="try logo" width="100" height="130">

"Do, or do not. There is no try."

We're setting out to change that.

## Description
[![LocalTests](https://github.com/binpash/try/actions/workflows/test.yaml/badge.svg)](https://github.com/binpash/try/actions/workflows/test.yaml)
[![License](https://img.shields.io/badge/License-MIT-blue)](#license)
[![issues - try](https://img.shields.io/github/issues/binpash/try)](https://github.com/binpash/try/issues)

`try` lets you run a command and inspect its effects before changing your live system. `try` uses Linux's [namespaces (via `unshare`)](https://docs.kernel.org/userspace-api/unshare.html) and the [overlayfs](https://docs.kernel.org/filesystems/overlayfs.html) union filesystem.

Please note that `try` is a prototype and not a full sandbox, and should not be used to execute
commands that you don't already trust on your system, (i.e. devices in `/dev` are
mounted in the sandbox, and network calls are all allowed.) Please do not
attempt any commands that will remove everything in /dev or write zeros to your
disks.

<img src="docs/try_pip_install_example.gif" alt="try gif">

## Getting Started

### Dependencies

Has been tested on the following distributions:
* `Ubuntu 20.04 LTS` or later
* `Debian 12`
* `Centos 9 Stream 5.14.0-325.el9`
* `Arch 6.1.33-1-lts`
* `Alpine 6.1.34-1-lts`
* `Rocky 9 5.14.0-284.11.1.el9_2`

### Installing

You only need the [`try` script](https://raw.githubusercontent.com/binpash/try/main/try), which you can download by cloning this repository:

```ShellSession
$ git clone https://github.com/binpash/try.git
```

## Example Usage

`try` is a higher-order command, like `xargs`, `exec`, `nohup`, or `find`. For example, to install a package via `pip3`, you can invoke `try` as follows:

```ShellSession
$ try pip3 install libdash
... # output continued below
```

By default, *try* will ask you to commit the changes made at the end of its execution.

```ShellSession
...
Defaulting to user installation because normal site-packages is not writeable
Collecting libdash
  Downloading libdash-0.3.1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (254 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 254.6/254.6 KB 2.1 MB/s eta 0:00:00
Installing collected packages: libdash
Successfully installed libdash-0.3.1
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv

Changes detected in the following files:

/tmp/tmp.zHCkY9jtIT/upperdir/home/gliargovas/.local/lib/python3.10/site-packages/libdash/ast.py (modified/added)
/tmp/tmp.zHCkY9jtIT/upperdir/home/gliargovas/.local/lib/python3.10/site-packages/libdash/_dash.py (modified/added)
/tmp/tmp.zHCkY9jtIT/upperdir/home/gliargovas/.local/lib/python3.10/site-packages/libdash/__init__.py (modified/added)
/tmp/tmp.zHCkY9jtIT/upperdir/home/gliargovas/.local/lib/python3.10/site-packages/libdash/__pycache__/printer.cpython-310.pyc (modified/added)
/tmp/tmp.zHCkY9jtIT/upperdir/home/gliargovas/.local/lib/python3.10/site-packages/libdash/__pycache__/ast.cpython-310.pyc (modified/added)
<snip>

Commit these changes? [y/N] y
```

Sometimes, you might want to pre-execute a command and commit its result at a later time. Invoking *try* with the -n flag will return the overlay directory, without committing the result.

```ShellSession
$ try -n "curl https://sh.rustup.rs | sh"
/tmp/tmp.uCThKq7LBK
```

Alternatively, you can specify your own overlay directory as follows (note that you'll have to create the sandbox directory first)

```ShellSession
$ try -D rustup-sandbox "curl https://sh.rustup.rs | sh"
$ ls rustup-sandbox
temproot  upperdir  workdir
```

As you can see from the output above, *try* has created an overlay environment in the *rustup-sandbox* directory.

Manually inspecting upperdir reveals the changes to the files made inside the overlay during the execution of the previous command with *try*:

```ShellSession
~/try/rustup-sandbox/upperdir$ du -hs .
1.2G    .
```

You can inspect the changes made inside a given overlay directory using *try*:

```ShellSession
$ try summary rustup-sandbox/ | head

Changes detected in the following files:

rustup-sandbox//upperdir/home/ubuntu/.profile (modified/added)
rustup-sandbox//upperdir/home/ubuntu/.bashrc (modified/added)
rustup-sandbox//upperdir/home/ubuntu/.rustup/update-hashes/stable-x86_64-unknown-linux-gnu (modified/added)
rustup-sandbox//upperdir/home/ubuntu/.rustup/settings.toml (modified/added)
rustup-sandbox//upperdir/home/ubuntu/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/libstd-8389830094602f5a.so (modified/added)
rustup-sandbox//upperdir/home/ubuntu/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/etc/lldb_commands (modified/added)
rustup-sandbox//upperdir/home/ubuntu/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/etc/gdb_lookup.py (modified/added)
```

You can also choose to commit the overlay directory contents:

```ShellSession
$ try commit rustup-sandbox
```

## Known Issues
Any command that interacts with other users/groups will fail since only the
current user's UID/GID are mapped. However, the [future
branch](https://github.com/binpash/try/tree/future) has support for uid/mapping,
please refer to the that branch's readme for installation instructions for the
uid/gidmapper.

Please also report any issue you run into while using the future branch!

## Version History

* 0.1.0 - 2023-06-25
    * Initial release.

## License

This project is licensed under the MIT License - see LICENSE for details.

Copyright (c) 2023 The PaSh Authors.
