# Advanced debugging

## PD

### Check PD state

Example on Framework 13 AMD Ryzen AI 300

```
> sudo framework_tool.exe --pd-info
Right / Ports 01
  Silicon ID:     0x3580
  Mode:           MainFw
  Flash Row Size: 256 B
  Ports Enabled:  0, 1
  Bootloader Version:   Base: 3.6.0.009,  App: 0.0.01
  FW1 (Backup) Version: Base: 3.7.0.197,  App: 0.0.0B
  FW2 (Main)   Version: Base: 3.7.0.197,  App: 0.0.0B
Left / Ports 23
  Silicon ID:     0x3580
  Mode:           MainFw
  Flash Row Size: 256 B
  Ports Enabled:  0, 1
  Bootloader Version:   Base: 3.6.0.009,  App: 0.0.01
  FW1 (Backup) Version: Base: 3.7.0.197,  App: 0.0.0B
  FW2 (Main)   Version: Base: 3.7.0.197,  App: 0.0.0B
```

### Disable/enable/reset PD

```
# Disable all ports on PD 0
> sudo framework_tool --pd-disable 0

# Reset PD 0 (enables all ports again)
> sudo framework_tool --pd-reset 0

# Or enable all ports on PD 0 without resetting it
> sudo framework_tool --pd-enable 0
```

### Check EFI Resource Table

On Framework Desktop:

```
> sudo framework_tool --esrt
ESRT Table
  ResourceCount:        1
  ResourceCountMax:     1
  ResourceVersion:      1
ESRT Entry 0
  GUID:                 EB68DBAE-3AEF-5077-92AE-9016D1F0C856
  GUID:                 DesktopAmdAi300Bios
  Type:                 SystemFirmware
  Version:              0x204 (516)
  Min FW Version:       0x100 (256)
  Capsule Flags:        0x0
  Last Attempt Version: 0x108 (264)
  Last Attempt Status:  Success
```

## Flashing EC firmware

**IMPORTANT** Flashing EC firmware yourself is not recommended. It may render
your hardware unbootable. Please update your firmware using the official BIOS
update methods (Windows .exe, LVFS/FWUPD, EFI updater)!

This command has not been thoroughly tested on all Framework Computer systems

```
# Simulate flashing RW (to see which blocks are updated)
> framework_tool --flash-rw-ec ec.bin --dry-run

# Actually flash RW
> framework_tool --flash-rw-ec ec.bin

# Boot into EC RW firmware (will crash your OS and reboot immediately)
# EC will boot back into RO if the system turned off for 30s
> framework_tool --reboot-ec jump-rw
```

## Flashing Expansion Bay EEPROM (Framework 16)

This will render your dGPU unsuable if you flash the wrong file!
It's intended for advanced users who build their own expansion bay module.
The I2C address of the EEPROM is hardcoded to 0x50.

```
# Dump current descriptor (e.g. for backup)
> framework_tool --dump-gpu-descriptor-file foo.bin
Dumping to foo.bin
Wrote 153 bytes to foo.bin

# Update just the serial number
> framework_tool --flash_gpu_descriptor GPU FRAKMQCP41500ASSY1
> framework_tool --flash_gpu_descriptor 13 FRAKMQCP41500ASSY1
> framework_tool --flash_gpu_descriptor 0x0D FRAKMQCP41500ASSY1

# Update everything from a file
> framework_tool --flash-gpu-descriptor-file pcie_4x2.bin
```

## Analyzing binaries

### EC

Note that only since Framework 13 Intel Core Ultra (and later) the version number embedded in the ED binary is meaningful. As you can see below, in this example on Intel Core 12th/13th Gen (hx30) it's always 0.0.1.
The commit hash though is accurate and reflects the git commit it was built from.

```
> framework-tool --ec--bin ec.bin
File
  Size:                     524288 B
  Size:                        512 KB
EC
  Version:     hx30_v0.0.1-7a61a89
  RollbackVer:                   0
  Platform:                   hx30
  Version:                   0.0.1
  Commit:                  7a61a89
  Size:                       2868 B
  Size:                          2 KB
```

### PD

```
> framework_tool --pd-bin pd-0.1.14.bin
File
  Size:                      65536 B
  Size:                         64 KB
FW 1
  Silicon ID:               0x3000
  Version:                  0.1.14
  Row size:                    128 B
  Start Row:                    22
  Rows:                         95
  Size:                      12160 B
  Size:                         11 KB
FW 2
  Silicon ID:               0x3000
  Version:                  0.1.14
  Row size:                    128 B
  Start Row:                   118
  Rows:                        381
  Size:                      48768 B
  Size:                         47 KB
```

### UEFI Capsule

```
> framework_tool --capsule retimer23.cap
File
  Size:                    2232676 B
  Size:                       2180 KB
Capsule Header
  Capsule GUID: (ba2e4e6e, 3b0c, 4f25, [8a,59,4c,55,3f,c8,6e,a2])
  Header size:                  28 B
  Flags:                   0x50000
    Persist across reset  (0x10000)
    Initiate reset        (0x40000)
  Capsule Size:            2232676 B
  Capsule Size:               2180 KB
  Type:   Framework Retimer23 (Right)
```
