# trivialt

[![Go Report Card](https://goreportcard.com/badge/kvaps/trivialt)](https://goreportcard.com/report/kvaps/trivialt)
[![GoDoc](https://godoc.org/github.com/kvaps/trivialt?status.svg)](http://godoc.org/github.com/kvaps/trivialt)
[![MIT licensed](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/kvaps/trivialt/master/LICENSE)


trivialt is a cross-platform, concurrent TFTP server and client. It can be used as a standalone executable or included in a Go project as a library.


### Standards Implemented

- [X] Binary Transfer ([RFC 1350](https://tools.ietf.org/html/rfc1350))
- [X] Netascii Transfer ([RFC 1350](https://tools.ietf.org/html/rfc1350))
- [X] Option Extension ([RFC 2347](https://tools.ietf.org/html/rfc2347))
- [X] Blocksize Option ([RFC 2348](https://tools.ietf.org/html/rfc2348))
- [X] Timeout Interval Option ([RFC 2349](https://tools.ietf.org/html/rfc2349))
- [X] Transfer Size Option ([RFC 2349](https://tools.ietf.org/html/rfc2349))
- [X] Windowsize Option ([RFC 7440](https://tools.ietf.org/html/rfc7440))

### Unique Features

- __Single Port Mode__

    TL;DR: It allows TFTP to work through firewalls.

    A standard TFTP server implementation receives requests on port 69 and allocates a new high port (over 1024) dedicated to that request.
    In single port mode, trivialt receives and responds to requests on the same port. If trivialt is started on port 69, all communication will
    be done on port 69.
    
    The primary use case of this feature is to play nicely with firewalls. Most firewalls will prevent the typical case where the server responds
    back on a random port because they have no way of knowing that it is in response to a request that went out on port 69. In single port mode,
    the firewall will see a request go out to a server on port 69 and that server respond back on the same port, which most firewalls will allow.
    
    Of course if the firewall in question is configured to block TFTP connections, this setting won't help you.
    
    Enable single port mode with the `--single-port` flag. This is currently marked experimental as is diverges from the TFTP standard.

## Installation

If you have the Go toolchain installed you can simply `go get` the packages. This will download the source into your `$GOPATH` and install the binary to `$GOPATH/bin/trivialt`.

``` bash
go get -u github.com/kvaps/trivialt/...
```

Pre-built binaries can be downloaded from the [release page](https://github.com/kvaps/trivialt/releases).

## Command Usage

Running as a server:
```
# trivialt serve --help
NAME:
   trivialt serve - Serve files from the filesystem.

USAGE:
   trivialt serve [bind address] [root directory]

DESCRIPTION:
   Serves files from the local file systemd.

   Bind address is in form "ip:port". Omitting the IP will listen on all interfaces.
   If not specified the server will listen on all interfaces, port 69.app

   Files will be served from root directory. If omitted files will be served from
   the current directory.

OPTIONS:
   --writeable, -w	    Enable file upload.
   --single-port, --sp	Enable single port mode. [Experimental]
```

```
# trivialt serve :6900 /tftproot --writable
Starting TFTP Server on ":6900", serving "/tftproot"
Read Request from 127.0.0.1:61877 for "ubuntu-16.04-server-amd64.iso"
Write Request from 127.0.0.1:51205 for "ubuntu-16.04-server-amd64.iso"

```

Downloading a file:
```
# trivialt get --help
NAME:
   trivialt get - Download file from a server.

USAGE:
   trivialt get [command options] [server:port] [file]

OPTIONS:
   --blksize, -b "512"      Number of data bytes to send per-packet.
   --windowsize, -w "1"     Number of packets to send before requiring an acknowledgement.
   --timeout, -t "10"       Number of seconds to wait before terminating a stalled connection.
   --tsize                  Enable the transfer size option. (default)
   --retransmit, -r "10"    Maximum number of back-to-back lost packets before terminating the connection.
   --netascii               Enable netascii transfer mode.
   --binary, --octet, -i    Enable binary transfer mode. (default)
   --quiet, -q              Don't display progress.
   --output, -o             Sets the output location to write the file. If not specified the
                            file will be written in the current directory.
                            Specifying "-" will write the file to stdout. ("-" implies "--quiet")
```

```
# trivialt get localhost:6900 ubuntu-16.04-server-amd64.iso
ubuntu-16.04-server-amd64.iso:
 655.00 MB / 655.00 MB [=====================================================] 100.00% 16.76 MB/s39s
```

Uploading a file:
```
# trivialt get --help
NAME:
   trivialt get - Download file from a server.

USAGE:
   trivialt get [command options] [server:port] [file]

OPTIONS:
   --blksize, -b "512"      Number of data bytes to send per-packet.
   --windowsize, -w "1"     Number of packets to send before requiring an acknowledgement.
   --timeout, -t "10"       Number of seconds to wait before terminating a stalled connection.
   --tsize                  Enable the transfer size option. (default)
   --retransmit, -r "10"	Maximum number of back-to-back lost packets before terminating the connection.
   --netascii               Enable netascii transfer mode.
   --binary, --octet, -i	Enable binary transfer mode. (default)
   --quiet, -q              Don't display progress.
   --output, -o             Sets the output location to write the file. If not specified the
                            file will be written in the current directory.
                            Specifying "-" will write the file to stdout. ("-" implies "--quiet")
```

```
# trivialt put localhost:6900 ubuntu-16.04-server-amd64.iso --blksize 1468 --windowsize 16
ubuntu-16.04-server-amd64.iso:
 655.00 MB / 655.00 MB [=====================================================] 100.00% 178.41 MB/s3s
```
