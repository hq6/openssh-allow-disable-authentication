# OpenSSH Allow Disable Authentication

This fork includes a patch on openssh that adds a new sshd option called
DisableAuthentication.

This version allows one to put the option `DisableAuthentication yes` into an
sshd_config.

When this option is given, sshd will behave as follows:
 * If sshd is running as an ordinary user, then this user can login
   without authentication.
 * If sshd is running as root, then all ordinary users can login without
   authentication. If `PermitRootLogin yes` is also in the config, then
   root can login without authentication.

It is recommended to only use this option where authentication is
achieved by other means and ssh is only used for its other features,
such as port forwarding and properly signalling processes on disconnect.

For example, you might run this on an unprivileged port inside a docker
container that requires authenticated kubectl port-forward to reach at all.

## Why is this useful?

Openssh has useful capabilities such as remote and local port-forwarding, as
well as better terminal management compared to older tools like telnet that
support anonymous login.  With this option, we can use openssh in scenarios
where authentication is not required, without having to setup authentication
infrastructure for it.

One example: We can combine sshd on an unprivileged port with kubectl
port-forward to replace kubectl exec for shelling into containers running in a
secure Kubernetes environment. Kubectl exec does not kill processes on
disconnect, and does not support remote port forwarding, while ssh does both of
these things.

## Why is this useful when openssh already has PermitEmptyPassword?

PermitEmptyPasswords is a reasonable option for many uses, but it requires that
the user actually has an empty password, which is not desirable if we also want
a user to be accessible externally without the risk of a misconfigured ssh
server on port 22.

This additional option allows a user to be accessible without a password in
environments where authorization is granted by other means, even if they
otherwise have a password.

***

# Portable OpenSSH

[![C/C++ CI](https://github.com/openssh/openssh-portable/actions/workflows/c-cpp.yml/badge.svg)](https://github.com/openssh/openssh-portable/actions/workflows/c-cpp.yml)
[![Fuzzing Status](https://oss-fuzz-build-logs.storage.googleapis.com/badges/openssh.svg)](https://bugs.chromium.org/p/oss-fuzz/issues/list?sort=-opened&can=1&q=proj:openssh)
[![Coverity Status](https://scan.coverity.com/projects/21341/badge.svg)](https://scan.coverity.com/projects/openssh-portable)

OpenSSH is a complete implementation of the SSH protocol (version 2) for secure remote login, command execution and file transfer. It includes a client ``ssh`` and server ``sshd``, file transfer utilities ``scp`` and ``sftp`` as well as tools for key generation (``ssh-keygen``), run-time key storage (``ssh-agent``) and a number of supporting programs.

This is a port of OpenBSD's [OpenSSH](https://openssh.com) to most Unix-like operating systems, including Linux, OS X and Cygwin. Portable OpenSSH polyfills OpenBSD APIs that are not available elsewhere, adds sshd sandboxing for more operating systems and includes support for OS-native authentication and auditing (e.g. using PAM).

## Documentation

The official documentation for OpenSSH are the man pages for each tool:

* [ssh(1)](https://man.openbsd.org/ssh.1)
* [sshd(8)](https://man.openbsd.org/sshd.8)
* [ssh-keygen(1)](https://man.openbsd.org/ssh-keygen.1)
* [ssh-agent(1)](https://man.openbsd.org/ssh-agent.1)
* [scp(1)](https://man.openbsd.org/scp.1)
* [sftp(1)](https://man.openbsd.org/sftp.1)
* [ssh-keyscan(8)](https://man.openbsd.org/ssh-keyscan.8)
* [sftp-server(8)](https://man.openbsd.org/sftp-server.8)

## Stable Releases

Stable release tarballs are available from a number of [download mirrors](https://www.openssh.com/portable.html#downloads). We recommend the use of a stable release for most users. Please read the [release notes](https://www.openssh.com/releasenotes.html) for details of recent changes and potential incompatibilities.

## Building Portable OpenSSH

### Dependencies

Portable OpenSSH is built using autoconf and make. It requires a working C compiler, standard library and headers.

``libcrypto`` from either [LibreSSL](https://www.libressl.org/) or [OpenSSL](https://www.openssl.org) may also be used.  OpenSSH may be built without either of these, but the resulting binaries will have only a subset of the cryptographic algorithms normally available.

[zlib](https://www.zlib.net/) is optional; without it transport compression is not supported.

FIDO security token support needs [libfido2](https://github.com/Yubico/libfido2) and its dependencies and will be enabled automatically if they are found.

In addition, certain platforms and build-time options may require additional dependencies; see README.platform for details about your platform.

### Building a release

Releases include a pre-built copy of the ``configure`` script and may be built using:

```
tar zxvf openssh-X.YpZ.tar.gz
cd openssh
./configure # [options]
make && make tests
```

See the [Build-time Customisation](#build-time-customisation) section below for configure options. If you plan on installing OpenSSH to your system, then you will usually want to specify destination paths.

### Building from git

If building from git, you'll need [autoconf](https://www.gnu.org/software/autoconf/) installed to build the ``configure`` script. The following commands will check out and build portable OpenSSH from git:

```
git clone https://github.com/openssh/openssh-portable # or https://anongit.mindrot.org/openssh.git
cd openssh-portable
autoreconf
./configure
make && make tests
```

### Build-time Customisation

There are many build-time customisation options available. All Autoconf destination path flags (e.g. ``--prefix``) are supported (and are usually required if you want to install OpenSSH).

For a full list of available flags, run ``./configure --help`` but a few of the more frequently-used ones are described below. Some of these flags will require additional libraries and/or headers be installed.

Flag | Meaning
--- | ---
``--with-pam`` | Enable [PAM](https://en.wikipedia.org/wiki/Pluggable_authentication_module) support. [OpenPAM](https://www.openpam.org/), [Linux PAM](http://www.linux-pam.org/) and Solaris PAM are supported.
``--with-libedit`` | Enable [libedit](https://www.thrysoee.dk/editline/) support for sftp.
``--with-kerberos5`` | Enable Kerberos/GSSAPI support. Both [Heimdal](https://www.h5l.org/) and [MIT](https://web.mit.edu/kerberos/) Kerberos implementations are supported.
``--with-selinux`` | Enable [SELinux](https://en.wikipedia.org/wiki/Security-Enhanced_Linux) support.

## Development

Portable OpenSSH development is discussed on the [openssh-unix-dev mailing list](https://lists.mindrot.org/mailman/listinfo/openssh-unix-dev) ([archive mirror](https://marc.info/?l=openssh-unix-dev)). Bugs and feature requests are tracked on our [Bugzilla](https://bugzilla.mindrot.org/).

## Reporting bugs

_Non-security_ bugs may be reported to the developers via [Bugzilla](https://bugzilla.mindrot.org/) or via the mailing list above. Security bugs should be reported to [openssh@openssh.com](mailto:openssh.openssh.com).
