PPP-over-Ethernet redirector for pppd (Linux, MacOS)
====================================================

## Introduction

pppoe is a user-space redirector which permits the use of PPPoE
(Point-to-Point Over Ethernet) with Linux.  PPPoE is used by many
DSL service providers.

## Installation

#### Requirements

1. Linux, MacOS Catalina
2. pppd 2.4.2 or later.

#### QUICKSTART

If you're lucky, the "quickstart" method will work.  After clone, become root and type:

```sh
$ sudo ./go
```

This should configure, compile and install the software and set up your
DSL connection.  You'll have to answer a few questions along the way.

#### Compiling

Compile and install pppd if you don't already have it.  Then:

```sh
// Clone:
$ git clone https://github.com/dmamontov/rp-pppoe.git

// Change to source directory:
$ cd rp-pppoe/src

// Configure:
$ ./configure

// Compile:
$ make

// Install (this step must be done as root)
sudo make install
```

#### Now read
- [HOW-TO-CONNECT](doc/HOW-TO-CONNECT.md)

--
- Dmitry Mamontov <d.slonyara@gmail.com>
- Dianne Skoll <dfs@roaringpenguin.com>
