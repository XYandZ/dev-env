# Development Environment

## What is this?

A guide to setting up my preferred _tabula rasa_ development environment.

### Project Layout

```text
├── http # Ubuntu 2x.xx autoinstall files
│   ├── meta-data # Empty but required for builds
│   └── user-data # Describes machine state of canonical reference image
├── README.md # This file
└── ubuntu-20.04-uefi.pkr.hcl # Canonical reference image build steps (Ubuntu 20.04)
└── ubuntu-20.04-uefi.pkr.hcl # Canonical reference image build steps (Ubuntu 22.04)
```

### Considerations

This development environments is set up to conform to [SOC 2](https://www.imperva.com/learn/data-security/soc-2-compliance/) best practices

### Required Packages

#### General Tools

| Package | Use |
|---------|-----|
| Git | Code repository management |
| rustup | Rust version manager |
| A browser (Firefox, Chrome, etc.) | Access G Suite utilities |

#### DevOps and Provisioning Tools

| Package | Use                    |
|---------|------------------------|
| docker | Used for parts of the CI/CD pipeline and elsewhere|
| docker-compose | Prototyping internal infrastructure|
| Terraform | Provisioning internal infrastructure|
| Packer | Building machine images|

#### Common "Gotchas"

* All storage devices containing code must be _at rest_ encrypted.
* SSH keys must be encrypted (via `ssh-keygen -t ed25519 -C <user name>-<RFC 3339 date>@<email>` or similar and included a password).

## Canonical Development Environment

This repo contains provisioning files for a canonical development environment to demonstrate
the core features any dev environment will need.
The canonical environment is an Ubuntu 20.04 QEMU image that can be used as a VM or used
to provision a bare metal machine. This image uses a UEFI partitioning scheme
and full disk encryption via `dm-crypt` in order to meet certain SOC 2 _in situ_ encryption requirements.
Further the image installs all required development and communication tools during installation.

### Pre-Reqs

The following packages will need to be installed in order to build a canonical development environment image
locally.

* packer
* KVM / QEMU

### Building an Image

`packer build ./ubuntu-20.04-uefi.pkr.hcl -var "output_directory=<directory for generated image>"`

Note `ubuntu-20.04-uefi.pkr.hcl` defines many more configuration variables than `output_directory` which
may or may not be useful to set.

### System Variables

| Variable     | Default     |
|--------------|-------------|
| Host Name    | dev-machine |
| User Name    | dev         |
| Password     | change      |
| DM Crypt Key | change      |

* Note these variables are all set within `./http/user-data`
* Password _must_ be in crypted format which can be accomplished via 
`printf '<password>' | openssl passwd -6 -salt 'FhcddHFVZ7ABA4Gi' -stdin` or similar

### Copying to Physical Media

The raw disk image generated by packer can either be run as a QEMU virtual machine or copied onto
physical media to run on a bare metal system.

Since we're working with raw disk images we copy the entire generated disk over using a command like `dd`:
```shell
dd if=/<path to disk image>/canonical_dev of=/dev/sd<X> bs=4k
```

Where `sd<X>` corresponds to the dev file for a physical device like a hard drive, USB stick etc.

As with anything involving `dd` be _extremely cautious_ about identifying the correct dev file before running the command.

Further once the disk image is copied over you'll need to re-size the root partition in order to fill the physical media completely.

### Further Reading

Ubuntu 2x.xx autoinstall is fairly opaque; below are some resources to reference:

* [Ubuntu autoinstall quick start](https://ubuntu.com/server/docs/install/autoinstall-quickstart)
* [Github project showing packer templates for Ubuntu 18.xx and 2x.xx](https://github.com/tylert/packer-build/tree/master/source/ubuntu)

## Things to Do After First Boot

* Install [RustUp](https://rustup.rs/)

### Special Considerations

* When running the provided machine images with KVM you _must_ use a UEFI bios. Legacy BIOSes like SeaBIOS will not work.