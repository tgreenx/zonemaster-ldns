Zonemaster LDNS
===============

**Table of contents**

* [Introduction](#introduction)
* [Dependencies and Compatibility](#dependencies-and-compatibility)
* [Installation and verification](#installation-and-verification)
  * [Recommended installation](#recommended-installation)
  * [Installation from source](#installation-from-source)
  * [Post-installation sanity check](#post-installation-sanity-check)
  * [Testing](#testing)
* [Docker](#docker)
* [Optional features](#optional-features)
  * [Ed25519]
  * [IDN]
  * [Internal ldns]
  * [Randomized capitalization](#randomized-capitalization)


## Introduction

This module provides a Perl interface to the [ldns library](https://www.nlnetlabs.nl/projects/ldns/) from [NLnet Labs](https://www.nlnetlabs.nl/). This module includes the necessary C code from ldns, so the library does not need to be globally installed. It does, however, rely on a sufficiently recent version of OpenSSL being present.

This module is written as part of the [Zonemaster project](http://github.com/zonemaster/zonemaster), and therefore primarily exposes the functionality needed for that. Since Zonemaster is a diagnostic tool, that means the functions most used are those for looking things up and inspecting them.

If you want a module that specifically aims to be a complete and transparent interface to ldns, [DNS::LDNS](http://search.cpan.org/~erikoest/DNS-LDNS/) is a better fit than this module.

Initially this module was named Net::LDNS.

## Dependencies and compatibility

Run-time dependencies:
 * `openssl` (openssl >= 1.1.1 unless [Ed25519] is disabled)
 * `libidn` (if [IDN] is enabled)
 * `libldns` (if [Internal ldns] is disabled; libldns >= 1.7.0, or libldns >= 1.7.1 if [Ed25519] is enabled)

Compile-time dependencies (only when installing from source):
 * `make`
 * `Devel::CheckLib`
 * `Module::Install`
 * `Module::Install::XSUtil`
 * `Test::More >= 1.302015`
 * `git` (if [Internal ldns] is enabled)
 * `libtool` (if [Internal ldns] is enabled)
 * `autoconf` (if [Internal ldns] is enabled)
 * `automake` (if [Internal ldns] is enabled)

Test-time dependencies:
 * `Test::Fatal`

There is a small part in the code that may not be compatible with non-Unix operating systems, in that it assumes that the file /dev/null exists.

## Installation and verification

### Recommended installation

The recommended way to install Zonemaster::LDNS is to install it from CPAN as a dependency to Zonemaster::Engine. If you follow the [installation instructions for Zonemaster::Engine](https://github.com/zonemaster/zonemaster-engine/blob/master/docs/Installation.md) you will get all the prerequisites and dependencies for Zonemaster::LDNS before installing it from CPAN.

### Installation from source

Override the default set of features by appending `--FEATURE` and/or
`--no-FEATURE` options to the `perl Makefile.PL` command.

```sh
git clone https://github.com/zonemaster/zonemaster-ldns
cd zonemaster-ldns
perl Makefile.PL
make
make test
make install
```

> **Note:** The source ZIP files downloaded from Github are broken with
> respect to this instruction.

### Post-installation sanity check

```sh
perl -MZonemaster::LDNS -E 'say Zonemaster::LDNS->new("8.8.8.8")->query("zonemaster.net")->string'
```

The above command should print some `dig`-like output.

### Testing

Some of the unit tests depend on data on Internet, which may change. To avoid false 
fails, those unit tests are only run if the environment variable `TEST_WITH_NETWORK` is `true`. By default that variable
is unset (those tests are not run). To run all tests, execute

```sh
TEST_WITH_NETWORK=1 make test
```

## Docker

Build a local base image:

```sh
make all dist docker-build
```

Tag the local base image with the current version number:

```sh
make docker-tag-version
```

Tag the local base image as the latest version:

```sh
make docker-tag-latest
```

Perform smoke test on the local base image:

```sh
docker run zonemaster/ldns:local perl -MZonemaster::LDNS -E 'say Zonemaster::LDNS->new("9.9.9.9")->query("zonemaster.net")->string'
```

## Optional features

When installing from source, you can choose to enable or disable a number
of optional features using command line options to the `perl Makefile.PL`
commands.

### Ed25519

Enabled by default.
Disabled with `--no-ed25519`

Requires support for Ed25519 in both openssl and ldns.

>
> *Note:* Zonemaster Engine relies on this feature for its analysis when Ed25519
> (algorithm 15) is being used in DNS records.
>

### IDN

Enabled by default.
Disable with `--no-idn`.

If the IDN feature is enabled, the GNU `libidn` library will be used to
add a simple function that converts strings from Perl's internal encoding
to IDNA domain name format.
In order to convert strings from whatever encoding you have to Perl's
internal format, use L<Encode>.
If you need any kind of control or options, use L<Net::LibIDN>.
The included function here is only meant to assist in the most basic case,
although that should cover a lot of real-world use cases.

> **Note:** The Zonemaster Engine test suite assumes this feature
> is enabled.

### Internal ldns

Enabled by default.
Disable with `--no-internal-ldns`.

When enabled, an included version of ldns is statically linked into
Zonemaster::LDNS.
When disabled, libldns is dynamically linked just like other dependencies.

### Randomized capitalization

Disabled by default.
Enable with `--randomize`.

> **Note:** This feature is experimental.

Randomizes the capitalization of returned domain names.


### Custom OpenSSL

Disabled by default.
Enabled with `--prefix-openssl=/path/to/openssl`.

Enabling this makes the build tools look for OpenSSL in a non-standard place.

Technically this does two things:
 * Libcrypto is sought in the `lib` directory under the given directory.
 * The `include` directory under the given directory is added to the include
   path.

> **Note:** The `lib` directory under the given path must be known to the
> dynamic linker or feature checks will fail.


[IDN]: #idn
[Internal ldns]: #internal-ldns
[Ed25519]: #ed25519

