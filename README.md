# Kubernetes Network Plumbing Working Group

The Kubernetes Network Plumbing Working Group is an informal group working to
enable flexible networking for Kubernetes.

# Device Information Specification

This repository contains the officially approved versions of the Kubernetes
Network Plumbing Working Group's Device Information Specification.

The [SPEC.md](SPEC.md) file is the definitive specification and any other
formats (PDF, etc) are only provided for convenience. The specification version
will be managed by GitHub tags with the form “v1.0.0” in this specification
repository.

# Context for Device Information Specification

## Objectives

Provide a standard way of sharing extended device information between Device
Plugins, CNIs, NPWG Implementations (e.g: Multus) and workloads.

## Anti-objectives

This proposal does not specify the ways in which devices are created by a CNI or
Device Plugin or consumed by workloads.

## Current State / Limitations

* Device Plugins have no context for network attachment mapping to device
  resources so it cannot put meaningful information into the environment
  variable name/values.

* CNIs have that context (via annotations, resource names, etc) but cannot
  modify container environment variables or mounts.

* Device Plugins’ responses are not standardized:
  * SR-IOV Device Plugin adds
    ```
    ENV var (key := fmt.Sprintf("%s_%s_%s", "PCIDEVICE", rs.resourceNamePrefix, rs.resourcePool.GetResourceName()))
    ```
  * RDMA-Device-Plugin returns only DeviceSpecs.

* NPWG implementation (like Multus) match up information provided by the
  Device Plugin with a specific NetworkAttachmentDefinition through the
  "resource name" that the Device Plugin returns and kubelet device manager
  provides as the resourceMap.
  * The only information the CNI gets is the “deviceID” returned by the Device
    Plugin. (e.g: in the case of the SR-IOV Device Plugin, this is ID the PCI
    address).

## Use Cases

### Device Information from the Device Plugin to Pod

The Device Plugin needs to expose information to the Pod about the type of
device it has allocated and how to consume it.

Examples of this use case:

* vDPA devices which can be:
  * vDPA kernel device: /dev/vhost-vdpa0
  * vhost-user socket file. Both client and server mode should be supported.

* SR-IOV device (vfio) with an exceptional path (/dev/vhost-net) device.

* Network Accelerator (e.g: FPGA).

### Device Information from the Device Plugin to CNI

Currently, CNIs have to alter their behavior depending on some device
properties. Since the information about the device is very limited (“deviceID”),
they have to find the device properties by themselves. CNIs need a consistent
way of obtaining device information.

An example of this user case is:
* The SR-IOV CNI has to find out whether the device “is in DPDK mode”.
  Currently, it only receives the DeviceID through the ResourceMap. That
  “DeviceID” happens to be the PCI Address (convention).

### Device Information from CNIs to Pod

Some CNIs such as the Userspace CNI create devices by themselves and those
devices need to be exposed to the Pod for it to consume it.

An example of this use case is:

* The Userspace CNI creates vhost-user sockets that need to be properly exposed
  to the Pod.

* Other device information: e.g: vhost mode, etc

### Device - Network Relationship

Regardless of who creates the device resource (Device Plugin or CNI), if there
are multiple networks and multiple device resources, the pod needs a way to
determine which device corresponds to which network.

### Device Information to Container Runtime

Some runtimes might need extra device information.

An example of this use case:

* [Kata Containers](https://katacontainers.io/) is an OCI runtime that runs the
  containers inside a VM. Devices are passed through using vfio-pci. However,
  the PCI Address information that is seen from the container is no longer valid
  as the PCI hierarchy is completely different. For the runtime to offer a
  consistent device information remapping feature it would need that information
  to be standardized.

### Device Information from the DevicePlugin/CNI to a SmartNic

Smart NICs have another CPU inside the card itself that is used to manage flows
and control device allocation. Typically, an instance of OpenvSwitch would be
running in the Smart NIC.

In order to add a device (e.g VF) to a pod, certain device information (e.g: PF
and VF) might need to reach the Smart NIC in order for it to perform certain
tasks (e.g: add the correspondent representor to an OvS bridge). However, the
security model of the smart NIC might consider the worker node an untrusted
entity. In such a case, the information has to be stored in and read from the
Kube API.

However, the interface has to be fully operational when the Pod starts,
therefore the SmartNIC CNI would have to block until the entire process (which
ensures communicating with the SmartNIC) has concluded. For that reason, this
use case falls out of the scope of the current specification. 

# Workflow Summary

The procedures specified in [SPEC.md](SPEC.md) can be summarized as follows:

* The Device Plugin creates a file in a specified path for each device it
  discovers.

* The NPWG Implementation:
  * Generates a unique file (the *CNIDeviceInfoFile*) for each network
    attachment.
  * If a resourceName is present in the Network Attachment Definition, the NPWG
    Implementation copies the file created by the Device Plugin into the
    *CNIDeviceInfoFile*.
  * Passes the value of *CNIDeviceInfoFile* down to the CNI Plugins in the
    chain as a RuntimeConfig option.

* The CNI Plugins that exports the *CNIDeviceInfoFile* capability can:
  * Read the data coming from the Device Plugin from the *CNIDeviceInfoFile*
    if it exists.
  * Write it’s own device information data into that file if it does not exist.

* After calling the CNI Plugin, the NPWG Implementation:
  * Reads the *CNIDeviceInfoFile* file and adds its contents to the
    *network-status* annotation.

* After calling DELETE on a specific network attachment the NPWG Implementation
  deletes the file *CNIDeviceInfoFile*.

* The container workload can leverage any of the data in the *network-status*
  annotation if the Downward API annotations are specified in the pod
  specification.   
