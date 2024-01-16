Device Information Specification
================================

***Network Plumbing Working Group***

**Version 1.1.0**

This document describes the de-facto standard for sharing device information
between Device Plugins, NPWG Implementations and CNI plugins in Kubernetes.

Some parts of this specification have also been added to the NPWG
[Multi-Network CRD Specification](https://github.com/k8snetworkplumbingwg/multi-net-spec).

# 1 Document Versioning

This document shall be versioned in the MAJOR.MINOR.PATCH format. The
specification version will be managed by GitHub tags with the form “v1.0.0” in
this specification repository:

https://github.com/k8snetworkplumbingwg/device-info-spec/ 

The SPEC.md file is the definitive specification and any other formats (PDF,
etc) are only provided for convenience.

# 2 Definitions

## 2.1 Container Network Interface (CNI)

CNI consists of a specification and libraries for writing plugins to configure
network interfaces in Linux containers.

## 2.2 Device Plugin

A framework in Kubernetes for resources to be advertised to Kubelet. A general
rule is that Device Plugins manage limited resources.

## 2.3 Workload

Application running in a container.

## 2.4 Network Attachment Definition

Kubernetes allows custom resources to be created via a Custom Resource
Definition (CRD). The Network Plumbing Working Group specification uses a CRD
called a Network-Attachment-Definition to define a network, and configuration
data about that network.

## 2.5 Device

A “device”, as referred to in this specification, is one of possible several
interfaces being consumed by a container. A variety of technologies can be used
to back the device.

# 3 Device Information Format

The Device Information Format is the json format that all Device Information
files must follow. The fields are defined as follows.

## 3.1 “device-info” keys

The “device-info” map must contain the following keys.

### 3.1.1 “type”

This <ins>required</ins> key’s value (type string) shall indicate the type of
device that is being described and determines what additional information is
being included. This key’s value must contain one of the following values:

- **“pci”**: Indicates that a PCI device is being described.
- **“vdpa”**: Indicates that a vDPA device is being described.
- **“vhost-user”**: Indicates that a vhost-user device is being described.
- **“memif”**: Indicates that a memif device is being described.

### 3.1.2 “version”

This <ins>required</ins> key’s value (type string) must match the Device
Information Specification version that is being used. The format of the string
shall be “<MAJOR>.<MINOR>.<PATCH>” (i.e. “1.0.0”).

### 3.1.3 “pci”

This <ins>optional</ins> (<ins>required</ins> if “type” is set to “pci”) key’s
value (type map) shall contain PCI device information. The map may contain the
following keys.

#### 3.1.3.1 “pci-address”

This <ins>required</ins> key’s value (type string) shall contain the PCI Address
of the device (in standard BDF format “dddd:BB:DD.f”).

#### 3.1.3.2 “vhost-net”

This <ins>optional</ins> key’s value (type string) shall contain the path to
the vhost-net char device used as an exceptional path. See the DPDK guide for
details:
[Virtio_user as Exceptional Path](https://doc.dpdk.org/guides/howto/virtio_user_as_exceptional_path.html)

#### 3.1.3.3 “rdma-device”

This <ins>optional</ins> key’s value (type string) shall contain the name of the
RDMA device if used.

#### 3.1.3.4 “pf-pci-address”

This <ins>optional</ins> key’s value (type string) shall contain the PCI address
of the associated physical interface (in standard BDF format “dddd:BB:DD.f”) if
available and applicable.

#### 3.1.3.5 “representor-device”

This <ins>optional</ins> key’s value (type string) shall contain the name
of the representor device for PCI VF (e.g. "eth3") if available and applicable.

### 3.1.4 “vdpa”

This <ins>optional</ins> (<ins>required</ins> if “type” is set to “vdpa”) key’s
value (type map) shall contain vDPA device information. The map may contain the
following keys.

#### 3.1.4.1 “parent-device”

This <ins>required</ins> key’s value (type string) shall contain the vDPA device
as listed in the vDPA bus device directory (/sys/bus/vdpa/devices).

#### 3.1.4.2 “driver”

This <ins>required</ins> key’s value (type string) shall contain the vDPA driver
being used. This key’s value must contain one of the following values:

- **“vhost”**
- **“virtio”**

#### 3.1.4.3 “path”

This <ins>required</ins> key’s value (type string) shall contain the the
absolute device path to the vhost or virtio device. (eg: “/dev/vhost-vdpa4” or
“/dev/sys/bus/virtio/devices/virtio5”).

#### 3.1.4.4 “pci-address”

This <ins>optional</ins> key’s value (type string) shall contain the PCI Address
of the device (in standard BDF format “dddd:BB:DD.f”) if the device is backed by
a PCI device (VF or PF). 

#### 3.1.4.5 “pf-pci-address”

This <ins>optional</ins> key’s value (type string) shall contain the PCI address
of the associated physical interface (in standard BDF format “dddd:BB:DD.f”) if
available and applicable.

#### 3.1.4.6 “representor-device”

This <ins>optional</ins> key’s value (type string) shall contain the name
of the representor device for PCI VF (e.g. "eth3") if available and applicable.

### 3.1.5 “vhost-user”

This <ins>optional</ins> (<ins>required</ins> if “type” is set to “vhost-user”)
key’s value (type map) shall contain vhost-user device information. The map may
contain the following keys.

#### 3.1.5.1 “mode”

This <ins>required</ins> key’s value (type string) shall contain the mode in
which the vhost-user socket operates. This key’s value must contain one of the
following values:

- **“client”**: The host is responsible for creating the socket file and the
  application in the container attaches to the existing socket file.
- **“server”**: The application in the container is responsible for creating the
  socket file and the host attaches to the socket file once created.

#### 3.1.5.2 “path”

This <ins>required</ins> key’s value (type string) shall contain the path to the
vhost-user socket file.

### 3.1.6 “memif”

This <ins>optional</ins> (<ins>required</ins> if “type” is set to “memif”) key’s
value (type map) shall contain memif device information. The map may contain the
following keys.

#### 3.1.6.1 “role”

This <ins>required</ins> key’s value (type string) shall contain the role in
which the memif socket operates. This key’s value must contain one of the
following values:

- **“master”**: The application in the container is responsible for creating
  the socket file and the host attaches to the socket file once created.
- **“slave”**: The host is responsible for creating the socket file and the
  application in the container attaches to the existing socket file.

#### 3.1.6.2 “path”

This <ins>required</ins> key’s value (type string) shall contain the path to the
memif socket file.

#### 3.1.6.3 “mode”

This <ins>required</ins> key’s value (type string) shall contain the mode in
which the memif socket operates. This key’s value must contain one of the
following values:

- **“ethernet”**: Ethernet packets are transmitted and received.
- **“ip”**: IP packets are transmitted and received.
- **“inject-punt”**:  Reserved for future use. Not yet implemented.

# 4 Device Information Files

The device information files are used to store device information. These files
must contain a json-encoded device information map following the format
specified in section [3 Device Information Format](#3-device-information-format).

Device information files are used to share information between Device Plugins
and NPWG Implementations and between NPWG Implementations and CNI Plugins.
Depending on the technology used to back the device, either the Device Plugins
or the CNIs (NPWG Implementations and CNI Plugins together) are responsible for
managing the device information and the associated file. Therefore, there are
two different possible file locations, but for a given device, only one holds
the information that will eventually be passed to the pod.

## 4.1 Device Plugin Device Information File

The Device Plugin Device Information file that contains extended device
information (following the Device Information Format) for each resource managed
by a Device Plugin. It has the following path:


- **/var/run/k8s.cni.cncf.io/devinfo/dp/\<resourceName\>-\<deviceID\>-device.json**


Where:
- **\<resourceName\>** is the value of the resource provided by the Device
  Plugin and requested in the Network Attachment Definition annotation
  (k8s.v1.cni.cncf.io/resourceName). If \<resourceName\> contains “/”, they
  shall be replaced by “-” characters.
- **\<deviceID\>** is the ID of the device from a particular resource allocated
  by kubelet. It is generated by the Device Plugin for each resource it manages
  and made available to kubelet through the Device Plugin API. NPWG
  Implementations can retrieve them through the
  [Pod Resources API](https://pkg.go.dev/k8s.io/kubernetes/pkg/kubelet/apis/podresources?tab=doc). 

### 4.1.1 Device Plugin Requirements

The Device Plugin can write a Device Plugin Device Information file for each
resource it manages.

If the root directory (/var/run/k8s.cni.cncf.io/devinfo/dp/) does not exist at
the time the Device Plugin writes the Device Plugin Device Information File, it
must create it.

The Device Plugin is responsible for deleting the files it created when it stops
running.

## 4.2 CNI Plugin Device Information File

The CNI Plugin Device Information File is a file that is used to exchange
extended device information with CNI Plugins. It’s format is as follows:


- **/var/run/k8s.cni.cncf.io/devinfo/cni/\<UniqueFileName\>**


Where:
- **\<UniqueFileName\>** is a filename string that is unique per network
  attachment call.

### 4.2.1 NPWG Implementation Requirements

For each network attachment, the NPWG Implementation must generate a
*UniqueFileName* in order to compose a CNI Plugin Device Information File path
that is unique per network attachment.

If the Network Attachment Definition contains a *resourceName* key, the NPWG
Implementation must query Kubelet to find the DeviceID of the allocated resource
and copy the content of the corresponding Device Plugin Device Information File
into the generated CNI Plugin Device Information File.

If the root directory (*/var/run/k8s.cni.cncf.io/devinfo/cni/*) does not exist
when the NPWG Implementation writes the CNI Plugin Device Information File, it
must create it.

When CNI DELETE action is called, the NPWG Implementation must delete the CNI
Plugin Device Information File.

# 4.3 Device Information File Examples

```
{
    "type": "pci",
    "version": "1.1.0",
    "pci": {
        "pci-address": "0000:01:02.2",
        “pf-pci-address”: "0000:01:02.0"
    }
}
```

```
{
    "type": "vdpa",
    "version": "1.1.0",
    "vdpa": {
        “parent-device”: “vdpa3”,
        "driver": "vhost",
        "path: "/dev/vdpa-vhost0"
    }
}
```

```
{
    "type": "vhost-user",
    "version": "1.1.0",
    "vhost-user": {
        "mode": "server",
        "path": "/var/run/vhost.sock"
    }
}
```

# 5 New CNI Plugin Arguments

## 5.1 CNIDeviceInfoFile

In order to allow the CNI plugin know where it can read or write device
information, a new CNI Capability is introduced.

| Key               | Value                                                      |
| ----------------- | ---------------------------------------------------------- |
| CNIDeviceInfoFile | (String) A file path that is unique per network attachment |

### 5.1.1 NPWG Implementation Requirements

If a CNI Plugin in the plugin chain declares the CNIDeviceInfoFile capability,
the NPWG Implementation must set it to the full path of the CNI Plugin Device
Information File as described in the
[4.2 CNI Plugin Device Information File](#42-cni-plugin-Device-information-file)
section.

### 5.1.2 CNI Plugin Requirements

If the CNIDeviceInfoFile capability is present in the Network Attachment
Definition, the CNI plugin should receive the value of the CNI Plugin Device
Information File as part of the RuntimeConfig section of the CNI Args.

If a file exists in the CNI Plugin Device Information File path, the CNI Plugin
can read (the file has the same content as the associated Device Plugin Device Information File) 
and if required update the Device Information it contains.

If a file does not exist in the CNI Plugin Device Information File path, the CNI
Plugin can write json-encoded device information to it following the Device
Information Format.

If the root directory (/var/run/k8s.cni.cncf.io/devinfo/cni/) does not exist
when the CNI Plugin writes the CNI Plugin Device Information File, it must
create it.

# 6 "network-status" Annotation

## 6.1 “device-info”

This <ins>optional</ins> key’s value (type map) shall contain device information
gathered as a result of the network attachment. Such map must follow the
[3 Device Information Format](#3-device-information-format).

### 6.1.1 NPWG Implementation requirements

The NPWG Implementation must collect the device information data from the CNI
Device Information File after calling the CNI Plugin chain and add its content
to the “device-info” key in the "network-status" annotation.

### 6.1.2 Example

<pre>
kind: Pod
metadata:
  name: my-pod
  namespace: my-namespace
  annotations:
    k8s.v1.cni.cncf.io/network-status: |
      [
        {
          "name": "cluster-wide-default",
          "interface": "eth0",
          "ips": [ "192.0.2.2/24", "2001:db8::2230/32" ],
          "mac": "02:11:22:33:44:54",
          "default-route": ["192.0.2.1”]
        },
        {
          "name": "sriov-network_a",
          "interface": "net1",
          <b>"device-info": 
             {
               "type": "pci",
               "version": "1.1.0",
               "pci":
                 { 
                   "pci-address": "0000:18:02.5"
                   "pf-pci-address": "0000:18:00.0"
                 }
             }</b>
        },
        {
          "name": "sriov-network_b",
          "interface": "net2",
          <b>"device-info":
              {
                "type": "pci",
                "version": "1.1.0",
                "pci":
                  { 
                    "pci-address": "0000:18:0a.2"
                    "vhost-net": "/dev/vhost-net",  
                    "pf-pci-address": "0000:18:00.1"
                  }
              }</b>
        },
        {
          "name": "userspace_net1",
          "interface": "net3",
          "mac": " 64:1f:74:32:2e:6d",
          <b>"device-info":
              {
                "type": "vhost-user",
                "version": "1.1.0",
                "vhost-user":
                  { 
                    "mode": "client",
                    "path": "/var/run/net3/vhost.sock”
                  }
              }</b>
        },
        {
          "name": "vdpa_net1",
          "interface": "virtio0",
          "mac": " 64:1f:74:32:2e:6e",
          <b>"device-info":
              {
                "type": "vdpa”,
                "version": "1.1.0",
                "vdpa":
                  { 
                    "parent-device": "vdpa2",
                    "driver": "vhost",
                    "path": "/dev/vhost-vdpa0”,
                    "pci-address": "0000:02:01:6",
                    "pf-pci-address": "0000:02:01:0"
                  }
              }</b>
         }
      ]
</pre>
