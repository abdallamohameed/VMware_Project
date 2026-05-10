# VMware vSphere Infrastructure Implementation Project

> A comprehensive lab project demonstrating the deployment and configuration of an enterprise-grade virtualized data center using VMware vSphere — including ESXi, vCenter, clustering (HA/DRS), shared NFS storage, vMotion, and Fault Tolerance.

![VMware vSphere](https://img.shields.io/badge/VMware-vSphere%206.7-607078?logo=vmware&logoColor=white)
![ESXi](https://img.shields.io/badge/ESXi-6.7%20U1-brightgreen)
![Status](https://img.shields.io/badge/status-completed-success)
![Track](https://img.shields.io/badge/ITI-System%20Administration%20Track-9c2a2a)

---

## Table of Contents

- [About the Project](#about-the-project)
- [Project Objectives](#project-objectives)
- [Architecture](#architecture)
- [Lab Environment](#lab-environment)
- [Implementation Steps](#implementation-steps)
  - [1. Configuring ESXi Hosts](#1-configuring-esxi-hosts)
  - [2. vCenter Server Deployment](#2-vcenter-server-deployment)
  - [3. Cluster, HA & DRS](#3-cluster-ha--drs)
  - [4. Content Library](#4-content-library)
  - [5. Virtual Machine Operations](#5-virtual-machine-operations)
  - [6. Virtual Switch Configuration](#6-virtual-switch-configuration)
  - [7. vMotion Migration](#7-vmotion-migration)
  - [8. NFS Shared Datastore](#8-nfs-shared-datastore)
  - [9. Testing High Availability](#9-testing-high-availability)
  - [10. Fault Tolerance](#10-fault-tolerance)
- [Results & Validation](#results--validation)
- [Conclusion](#conclusion)
- [Team](#team)

---

## About the Project

This project simulates a production-grade virtualized data center built on **VMware vSphere 6.7**. It walks through every phase of a real-world deployment: from installing hypervisors, through centralized management, to advanced availability features such as **High Availability (HA)**, **Distributed Resource Scheduler (DRS)**, and **Fault Tolerance (FT)**.

The lab is built using **VMware Workstation** as the outer hypervisor, with two nested ESXi hosts and a separate Linux VM serving as the NFS storage server.

## Project Objectives

- Deploy and manage multiple **ESXi** hypervisors.
- Centralize management with **vCenter Server Appliance (VCSA)**.
- Build a **vSphere Cluster** with HA and DRS enabled.
- Provide **shared NFS storage** accessible by both hosts.
- Logically segregate network traffic (Management, vMotion, Storage).
- Demonstrate **VM lifecycle operations** (create, clone, template, snapshot).
- Validate **live migration (vMotion)** between hosts.
- Test **HA failover** and **Fault Tolerance** continuous availability.

## Architecture

The environment is composed of two ESXi hosts running on separate physical machines, connected via an access point. The vCenter Server Appliance centralizes management and unlocks clustering features. Shared storage is provided by an NFS server on a third machine, enabling vMotion, HA, and FT. Network traffic is logically segmented into management, vMotion, and storage networks.

```
                          ┌────────────────────┐
                          │   vCenter Server   │
                          │     (VCSA 6.7)     │
                          └─────────┬──────────┘
                                    │ Management Network
                ┌───────────────────┼───────────────────┐
                │                                       │
        ┌───────▼────────┐                     ┌────────▼───────┐
        │   ESXi Host 1  │◄──── vMotion ──────►│  ESXi Host 2   │
        │  192.168.1.6   │                     │  192.168.1.5   │
        └───────┬────────┘                     └────────┬───────┘
                │                                       │
                └────────────────┬──────────────────────┘
                                 │ Storage Network
                          ┌──────▼───────┐
                          │  NFS Server  │
                          │ 192.168.1.14 │
                          └──────────────┘
```

## Lab Environment

### IP Address Plan

| Device          | IP Address     | Role                              |
|-----------------|---------------|-----------------------------------|
| ESXi-01         | 192.168.1.6   | Hypervisor host #1                |
| ESXi-02         | 192.168.1.5   | Hypervisor host #2                |
| vCenter Server  | 192.168.1.11  | Centralized management appliance  |
| NFS Storage     | 192.168.1.14  | Shared datastore (NFS v3)         |

### Software Stack

- VMware Workstation (outer hypervisor)
- VMware ESXi 6.7 Update 1 (Build 10302608)
- VMware vCenter Server Appliance 6.7
- CentOS 7 (Linux NFS server / guest VM)
- Windows Server 2016 (FT test VM)

---

## Implementation Steps

### 1. Configuring ESXi Hosts

Two ESXi hosts were deployed as nested VMs in VMware Workstation. Each was provisioned with sufficient CPU, memory, and storage, and given a network adapter in **Bridged** mode so it received its own IP from the access point. ESXi was then installed and prepared for vCenter integration.

### 2. vCenter Server Deployment

The **vCenter Server Appliance (VCSA)** was deployed onto ESXi-01. After deployment, the second host (ESXi-02) was joined to vCenter using the *Add Host* wizard with the host's credentials.

### 3. Cluster, HA & DRS

A **Datacenter** object was created in vCenter, followed by a **Cluster** with **DRS** and **vSphere HA** enabled at creation time. Both ESXi hosts were then added to the cluster, which:

- Pools compute resources across hosts.
- Enables automatic load balancing (DRS).
- Provides automatic VM restart on host failure (HA).

### 4. Content Library

A **Content Library** was created to centralize ISO images and VM templates, simplifying VM deployment and ensuring consistent assets across hosts.

- Created the library backed by local storage.
- Uploaded a **CentOS 7 minimal** ISO to the library.

### 5. Virtual Machine Operations

The full VM lifecycle was exercised:

| Operation         | Description                                                         |
|-------------------|---------------------------------------------------------------------|
| **Create**        | A Linux VM was built on ESXi-01 from the CentOS ISO in the library. |
| **Clone**         | The VM was cloned to produce an identical copy.                     |
| **Template**      | The VM was converted into a reusable **template**.                  |
| **Deploy**        | A new VM was deployed *from* the template.                          |
| **Snapshot**      | A snapshot was captured to enable rollback.                         |

### 6. Virtual Switch Configuration

Each ESXi host was configured with two **standard virtual switches** to isolate traffic by purpose:

| Switch    | Purpose                                | Port Groups                                |
|-----------|----------------------------------------|--------------------------------------------|
| vSwitch0  | Management + VM traffic                | `Management Network`, `VM Network`         |
| vSwitch1  | vMotion + Storage (NFS)                | VMkernel: `vmotion`, `storage`             |

VMkernel ports were tagged with the appropriate services (Management, vMotion, Fault Tolerance logging) and pinned to dedicated physical uplinks for performance and isolation.

### 7. vMotion Migration

A running VM was **live-migrated** from ESXi-01 to ESXi-02 using vMotion. Because the VM resided on the **shared NFS datastore**, only its compute state was transferred — confirming seamless mobility with no service interruption.

### 8. NFS Shared Datastore

A CentOS VM was configured as an NFS v3 server:

```bash
# /etc/exports
/NFSdatastore *(rw,sync)

# Open NFS through the firewall
firewall-cmd --permanent --add-service=nfs
firewall-cmd --reload
firewall-cmd --list-all
```

The exported share was then mounted on both ESXi hosts as a **NFS datastore (`NFS_DS`)**, providing a single shared volume required for vMotion, HA, and FT.

### 9. Testing High Availability

To validate HA, one of the ESXi hosts running an active VM was shut down. Within seconds vCenter detected the host failure and **automatically restarted** the affected VM on the surviving host — confirming HA behavior end-to-end. DRS then re-balanced workloads across the cluster.

### 10. Fault Tolerance

Fault Tolerance was enabled on a **Windows Server 2016** VM:

1. On both ESXi hosts, the **Fault Tolerance logging** and **vMotion** services were enabled on the appropriate VMkernel ports.
2. FT was turned on for the VM, with the secondary placed on the partner host.
3. A **shadow (secondary) VM** began mirroring the primary in lockstep.

**Failover test:** the host running the primary VM was shut down. The secondary VM took over **instantly**, with no observable downtime — preserving the running session and IP address (`192.168.1.113`).

| State                 | Primary Host    |
|-----------------------|-----------------|
| Before failure        | ESXi-02 (`192.168.1.5`) |
| After failure         | ESXi-01 (`192.168.1.6`) |

## Results & Validation

| Capability                   | Outcome   |
|------------------------------|-----------|
| Centralized management       | ✅ vCenter manages both hosts |
| Cluster with DRS + HA        | ✅ Enabled and operational    |
| Shared storage (NFS v3)      | ✅ Mounted on both hosts      |
| vMotion live migration       | ✅ Successful, zero downtime  |
| HA host failover             | ✅ VM auto-restarted on peer  |
| Fault Tolerance failover     | ✅ Instant secondary takeover |

## Conclusion

This project successfully delivered a fully functional VMware vSphere environment that demonstrates the foundations of enterprise virtualization: hypervisor deployment, centralized management, clustering, shared storage, virtual networking, and the complete spectrum of availability features from vMotion to HA to Fault Tolerance. The result is a working reference lab and hands-on experience covering resource management, high availability, and infrastructure optimization in a virtualized data center.

## Team

**Supervised by:** Eng. Ekram Abd Elwahab

**Prepared by:**
- Ahmed Gamal
- Abdallah Mohamed
- Sohila Ahmed
- Alaa Bayoumi

**Program:** Information Technology Institute (ITI) — System Administration Track — 2026

---
