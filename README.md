HAPROXY 1.7.9 from source
=========================

dependency: liblua5.3-0

Install liblua, liblua-dev, libpcre, libpcre-dev, make and gcc packges before trying to compile Haproxy on Ubuntu 16.04 Xenial.

Download source version 1.7.9 stable from: http://www.haproxy.org/download/1.7/src/haproxy-1.7.9.tar.gz

After extracting source files from gzipped file modify Makefile to add CFLAGS options to CFLAGS line (in additions to existing options):

```
-fPIE -fstack-protector-strong -Wformat -Werror=format-security -Wdate-time -D_FORTIFY_SOURCE=2
```

Add Make file bug patch (patch 78):
-----------------------------------

```
# wget https://patch-diff.githubusercontent.com/raw/haproxy/haproxy/pull/78.patch
```

Make command
------------
```
# make USE_GETADDRINFO=1 USE_ZLIB=1 USE_REGPARM=1 USE_OPENSSL=1 USE_LUA=1 USE_PCRE=1 USE_NS=1 LUA_LIB_NAME=lua5.3 TARGET=linux2628 LUA_INC=/usr/include/lua5.3
```

this will create the "haproxy" and "haproxy-systemd-wrapper" binaries in the folder where Makefile was present and from where make command was executed.

Sneaky deb packages
-------------------

Now this is a short-cut or sneaky way to create deb package for Ubuntu using newly compiled haproxy binaries.

Download (but not install) the default deb packages provided by Ubuntu repos in Ubuntu 16.04 (versio 1.6.3):

```
# apt-get --download-only install haproxy
```

This will download the package but not install it under /var/cache/apt/.

__Copy the downloaded deb package out of cache folder and 'explode'  (extract deb package file/folder contents):__
```
# cp /var/cache/apt/haproxy_1.6.3-1ubuntu0.1_amd64.deb /tmp/package
# cd /tmp/package
# dpkg-deb -x haproxy_1.6.3-1ubuntu0.1_amd64.deb extract
```
__Copy the files I compiled under the extracted package folder in temp replacing the package provided binary files (overwrite versio 1.6.3 wirh version 1.7.9 binaries):__
```
# cp /root/haproxy-1.7.9/haproxy /tmp/package/extract/sbin/
# cp /root/haproxy-1.7.9/haproxy-systemd-wrapper /tmp/package/extract/sbin/
```

__Create the DEBIAN control file:__
```
# vi /tmp/package/extract/DEBIAN/control
Package: haproxy
Architecture: all
Maintainer: arghyanator@localhost
Depends: liblua5.3-0
Priority: optional
Version: 1.7.9
Description: haproxy 1.7.9 from source
```
__Now I can build the new DEB package:__
```
# dpkg-deb --build extract
```

This will create extract.deb package file with all the required files, folders and install scripts that are required to install and configure default haproxy on Ubuntu 16.04. You may rename the deb file to ```haproxy_1.7.9-1ubuntu0.1_amd64.deb``` (or to your liking)