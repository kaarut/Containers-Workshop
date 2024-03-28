# iSCSI

## Introduction

**Internet Small Computer Systems Interface (iSCSI)** is an [Internet Protocol](https://en.wikipedia.org/wiki/Internet_Protocol)-based storage networking standard for linking data storage facilities. iSCSI provides block-level access to storage devices by carrying SCSI commands over a TCP/IP network.

The protocol allows clients (called initiators) to send SCSI commands ([CDBs](https://en.wikipedia.org/wiki/SCSI_CDB)) to storage devices (targets) on remote servers. It is a [storage area network](https://en.wikipedia.org/wiki/Storage_area_network) (SAN) protocol, allowing organizations to consolidate storage into storage arrays while providing clients (such as database and web servers) with the illusion of locally attached SCSI disks.


## Terminology

### Initiator

A client in an iSCSI storage network is called the iSCSI Initiator Node (or simply, 'iSCSI Initiator'). This iSCSI Initiator can connect to a server (the [iSCSI Target](#target)). In doing so, the iSCSI Initiator sends SCSI commands to the iSCSI Target. These SCSI commands are packaged in IP packets for this purpose.

### IQN

iSCSI uses a special unique name to identify an iSCSI node, either target or initiator.

iSCSI names are formatted in two different ways. The most common is the IQN format.

The iSCSI Qualified Name (IQN) format takes the form **iqn.yyyy-mm.naming-authority:unique** name, where:

- _yyyy-mm_ is the year and month when the naming authority was established.
- _naming-authority_ is the reverse syntax of the Internet domain name of the naming authority. For example, the `iscsi.vmware.com` naming authority can have the iSCSI qualified name form of `iqn.1998-01.com.vmware.iscsi`. The name indicates that the `vmware.com` domain name was registered in January of 1998, and `iscsi` is a subdomain, maintained by `vmware.com`.
- _unique name_ is any name you want to use, for example, the name of your host.

### LUN

An iSCSI LUN is a logical unit of storage.
[iSCSI Target](#target) can provide one or more so-called logical units (LUs). The abbreviation “LUN” is often used for the term “logical unit” (although this abbreviation actually means “LU Number” or “logical unit number”).

### Target

A server in an iSCSI storage network is called the iSCSI Target Node (or simply, 'iSCSI Target').


## Example

An iSCSI exampl;e can be found on [the official Kubernetes repository](https://github.com/kubernetes/examples/tree/master/volumes/iscsi).
