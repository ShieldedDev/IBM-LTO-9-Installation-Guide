# IBM TS2900 LTO-9 Tape Autoloader --- Installation & Basic Configuration Guide

> Practical revision guide for the IBM TS2900 LTO-9 SAS tape-autoloader
> installation carried out in the lab/environment. This document
> separates what was actually observed from vendor-documented procedure
> and does not treat senior-engineer Commvault work as personally
> performed.

## 1. Environment

### Tape library

-   IBM TS2900 Tape Autoloader
-   Machine type/model: 3572-S9H
-   LTO-9 SAS tape drive
-   SAS host interface
-   Ethernet management interface

### Host

-   Dell PowerEdge R760xs
-   Windows Server
-   Server was used as an AD DS Domain Controller in the environment
-   Dell HBA355e external SAS HBA
-   IBM AGKB, 3 m Mini-SAS HD to Mini-SAS HD cable

### Backup software

-   Commvault
-   Commvault storage-pool configuration was performed by senior
    engineers.

## 2. Architecture

``` text
                    MANAGEMENT PATH
TS2900 ───── Ethernet ───── Management LAN

                    DATA PATH
TS2900 LTO-9 ── AGKB SAS cable ── Dell HBA355e ── Windows Server ── Commvault
```

**SAS is the tape data/control path. Ethernet is the TS2900
management/Web UI path.**

## 3. Physical installation

1.  Assemble/install the TS2900 and LTO-9 drive.
2.  If rack-mounted, install and secure the rack rails/library.
3.  Remove the accessor locking screw before powering the library.
4.  Insert the cartridge magazine and load the LTO media.
5.  Connect the approved power cable.
6.  Power on the TS2900.
7.  Allow the library to complete initialization.

IBM's Setup, Operator, and Service Guide states that the accessor
locking screw must be removed before power-on and documents the
rack/cabling procedure.

## 4. Media used

The deployment included: - Green Ultrium 9 / 18 TB cartridges for
data. - Black Universal Cleaning Cartridge for cleaning.

The cleaning cartridge is **not normal backup media**.

## 5. Initial library startup

After power-on, the library was allowed to initialize until it reached
**Ready**.

IBM documents that the Ready/Activity LED flashes during initialization
and becomes steady when mechanical initialization is complete.

**Practical result:** the TS2900 reached Ready and could be operated
from the front panel.

## 6. Inventory

The cartridge inventory was run from the operator panel:

``` text
Commands
  └── Inventory
        └── Execute
```

The operation completed successfully.

IBM documents inventory as a way to refresh the library map of the
cartridge magazine, accessor, and tape drive. Inventory is also
automatically performed when power is first turned on or the magazine is
inserted.

## 7. Verify library/drive state

Before moving to backup software, verify: - Library state - Drive
state - Cartridge presence - Magazine state - Error messages -
Ready/Activity status

The goal is to have a healthy, initialized library before host-side
configuration.

## 8. SAS connection to the Windows server

The actual host used a **Dell HBA355e** external SAS HBA.

Physical topology:

``` text
TS2900 LTO-9 SAS port
        │
        │ SFF-8644 / Mini-SAS HD
        │
       AGKB
        │
        │ SFF-8644 / Mini-SAS HD
        │
        ▼
Dell HBA355e external SAS HBA
        │
        ▼
Windows Server
```

Dell documents the HBA355e as a non-RAID external data controller for
tape drives and lists external LTO-6/7/8/9 support.

**Do not confuse this with the server's PERC H755 RAID controller or
QLogic Fibre Channel adapters.**

## 9. SAS connection sequence

IBM recommends shutting down and turning off the associated server
before connecting the SAS interface cable.

Recommended sequence:

1.  Shut down the Windows Server.
2.  Connect AGKB from the Dell HBA355e to the TS2900 LTO-9 SAS
    connector.
3.  Ensure the TS2900 is powered on and has completed initialization.
4.  Start the Windows Server.
5.  Check Windows device detection.

## 10. Windows verification

Before configuring Commvault, verify the operating system.

Open:

``` text
Device Manager
```

Check: - Tape drives - Medium changers - Storage controllers - Other
devices

The expected logical result is approximately:

``` text
Tape drives
  └── LTO-9 tape drive

Medium changers
  └── TS2900 / medium changer
```

The exact names depend on device firmware, HBA drivers, Windows, and
device identification.

**Troubleshooting rule:** if Windows does not see the tape hardware,
troubleshoot the physical/SAS/HBA/driver layer before configuring
Commvault.

## 11. TS2900 Ethernet management

The TS2900 also has an Ethernet management interface.

During the deployment, the library initially showed:

``` text
000.000.000.000
```

This means an IPv4 management address had not been assigned/configured.

Ethernet is used for: - Web User Interface - Management - Monitoring -
Configuration

It is **not** the LTO tape data path.

## 12. Commvault hand-off

After the OS recognizes the tape library/drive, the library can be
configured in Commvault.

The general Commvault workflow is:

1.  Attach the tape library to the appropriate MediaAgent.
2.  Verify the OS recognizes the library and drives.
3.  Select the MediaAgent in Commvault.
4.  Run **Scan Hardware**.
5.  Detect the tape library.
6.  Configure the detected library.
7.  Configure/verify drives and media.
8.  Configure the required storage pools/media.
9.  Perform backup and restore validation.

In this deployment, **storage-pool creation/configuration was performed
by senior engineers**. This guide records it as the next stage, not as
work personally completed.

## 13. Actual workflow followed

``` text
Assemble/install TS2900 and drive
        ↓
Insert cartridges
        ↓
Connect power
        ↓
Power on
        ↓
Wait for initialization / Ready
        ↓
Commands → Inventory → Execute
        ↓
Verify cartridges/library/drive
        ↓
Connect AGKB SAS cable
        ↓
Dell HBA355e
        ↓
Windows Server
        ↓
Verify OS device detection
        ↓
Commvault configuration
        ↓
Storage pools/media configuration
```

## 14. Learning outcomes

### Tape library vs tape drive

The TS2900 is an autoloader containing a tape drive, robotic accessor,
magazine/slots, library controller, and management interface. The host
therefore interacts with more than just a tape drive.

### SAS vs Ethernet

-   **SAS:** host-to-LTO data/control path.
-   **Ethernet:** TS2900 management path.

### HBA requirement

The SAS cable alone is not enough. The host needs a compatible SAS HBA.
The deployment used a Dell HBA355e.

### RAID controller vs HBA

The server also has a PERC H755. That is distinct from the HBA355e used
for the external tape path.

### Inventory

Inventory refreshes the library's view of its magazine, accessor, and
tape drive. It is not a backup operation.

### OS before Commvault

A reliable troubleshooting hierarchy is:

``` text
Physical hardware
    ↓
SAS cable/link
    ↓
HBA
    ↓
Windows
    ↓
Tape drive + medium changer
    ↓
Commvault
    ↓
Storage pools/media
    ↓
Backup
    ↓
Restore test
```

## 15. Common mistakes

-   Do not confuse Ethernet with SAS.
-   Do not use the cleaning cartridge as backup media.
-   Do not begin Commvault troubleshooting when Windows cannot see the
    physical devices.
-   Do not assume every SAS connector on a server is equivalent.
-   Do not change PERC RAID settings simply because the tape device is
    not detected.
-   Confirm the actual HBA and its external port before troubleshooting
    the cable path.

## 16. Quick revision checklist

### Hardware

-   [ ] TS2900 installed/racked
-   [ ] LTO-9 drive installed
-   [ ] Accessor locking screw removed before power-on
-   [ ] Cartridge magazine installed
-   [ ] LTO-9 media loaded
-   [ ] Cleaning cartridge identified separately
-   [ ] Power connected

### Library

-   [ ] Powered on
-   [ ] Initialization completed
-   [ ] Ready state confirmed
-   [ ] Inventory executed successfully
-   [ ] Library/drive state checked

### SAS

-   [ ] Dell HBA355e identified
-   [ ] Correct external SAS port used
-   [ ] AGKB connected securely at both ends
-   [ ] Server connection made according to IBM procedure

### Windows

-   [ ] HBA detected
-   [ ] Tape drive visible
-   [ ] Medium changer visible
-   [ ] No relevant driver errors
-   [ ] Devices report operational status

### Commvault

-   [ ] Correct MediaAgent identified
-   [ ] Library attached to MediaAgent
-   [ ] Hardware scan completed
-   [ ] Library detected/configured
-   [ ] Drives configured
-   [ ] Media/storage pools configured
-   [ ] Backup test completed
-   [ ] Restore test completed

## 17. References

### IBM

**IBM TS2900 Tape Autoloader: Setup, Operator, and Service Guide ---
Machine Type 3572, GC27-2212-08**

https://www.ibm.com/support/pages/system/files/inline-files/GC27-2212-08_3.pdf

Useful sections: - Installation and configuration - Attaching the
library to a server - Operator Panel / Inventory - Library status and
LEDs

**IBM TS2900 Tape Autoloader Installation Quick Reference --- Machine
Type 3572**

https://www.ibm.com/support/pages/ibm-ts2900-tape-autoloader-installation-quick-reference-machine-type-3572

### Dell

**Dell HBA355e Adapter documentation**

https://www.dell.com/support/manuals/en-us/poweredge-t560/hba355_ug/dell-hba355e-adapter

Relevant topics: - HBA355e external SAS ports - HBA installation -
External tape-drive support - Windows driver support

### Commvault

**Tape Libraries --- Getting Started**

https://documentation.commvault.com/2024e/commcell-console/tape_libraries_getting_started.html

**Driver Configurations**

https://documentation.commvault.com/v11/commcell-console/driver_configurations.html

**Configuring a Tape Storage**

https://documentation.commvault.com/v11/software/configuring_tape_storage.html

## 18. Final mental model

> **Install → Power → Initialize → Inventory → Verify → SAS-connect →
> OS-detect → Commvault-configure → Pool/media configuration → Backup →
> Restore test**

Keep the three layers separate during troubleshooting:

``` text
TS2900 hardware state
        ↓
Windows device state
        ↓
Commvault state
```

A problem should normally be isolated at the lowest layer where the
expected state is not being achieved.
# IBM-LTO-9-Installation-Guide
