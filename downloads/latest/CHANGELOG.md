CHANGELOG
=========
Kobalt Kernel
Copyright (c) 2026 Abhranil Dasgupta
Licensed under the GNU General Public License v3.0


=========================================
 Release 1.0.4-B
=========================================

This is the initial public release of Kobalt. Every item listed below
represents work built from scratch (or near-scratch) for this release.


--- Core Kernel ---

  - Monolithic kernel written entirely in C
  - Targets x86-64; built with clang + lld + nasm (no GCC in kernel path)
  - Freestanding build: no sysroot, no stdlib, no red-zone, no SSE
  - Large code model (-mcmodel=large); kernel linked by custom LLD script
  - PVH boot entry; post-link NOTE segment verified by readelf


--- Scheduler ---

  - EEVDF (Earliest Eligible Virtual Deadline First) scheduler
  - Designed for low-latency AI/ML batch and inference workloads
  - SMP-aware; tested on up to 4 CPUs under QEMU Q35


--- AMX Support (x86-64) ---

  - Full Intel AMX integration: detection, XCR0 management, per-thread
    tile state save/restore via #NM handler
  - amx_context.S handles tile context in assembly
  - Tile configuration and TILECFG management
  - TMUL instruction support
  - BF16 matrix operations
  - INT8 matrix operations
  - Security model: AMX state is isolated per-thread; kernel never
    touches tile registers directly


--- Memory Subsystem ---

  - Physical memory manager built from scratch
  - Virtual memory and paging subsystem
  - Interrupt model documented and stable


--- TYKID (Tell Your Kernel that I'm Driver) ---

  - Driver security and attestation framework
  - Five-stage threat analysis pipeline:
      Stage 1: ELF validation
      Stage 2: Shannon entropy check (packing/encryption detection)
      Stage 3: HMAC-BLAKE2s integrity verification
      Stage 4: BearSSL SHA-256 attestation digest
      Stage 5: Bad-pattern table (52 entries, rolling SipHash) +
               BearSSL EC certificate verification
  - Gate seal: 64-bit identity derived from KOBALT_KERNEL_IDENT
  - Extended hyper-seal: 256-bit, four hardware identity vectors,
    Threefish-256-inspired ARX mixing, 24 rounds
  - IOMMU domain per driver; canary page trap on DMA overrun
  - Capability-based policy layer; compile-time allowlist
  - Runtime sandbox: DMA cap 256 MB, policy violation threshold 16
  - Tamper-evident audit ring: 512 entries, hash-chained, HMAC per record
  - Remote attestation export: 96-byte blob, HMAC-BLAKE2s signed
  - Watchdog kernel thread: RT priority 90, 30s ± 20% sweep interval,
    dead-man timer, two-stage escalation to safe mode then shutdown
  - 36 builtin drivers; critical drivers bypass load pipeline
  - USB class hot-plug integration
  - ACPI cross-validation against DMAR/MADT
  - libFuzzer harness for PCI classifier and audit ring (TYKID_FUZZ)


--- Filesystems ---

  - FlatFS: custom flat filesystem with host-side mkfs_flatfs and
    fsck_flatfs tools
  - exFAT and FAT support via Chan's FatFs


--- POSIX Layer (kposixz) ---

  - POSIX compatibility layer for syscall ABI
  - Documented and stable ABI


--- USB Stack ---

  - USB host controller support
  - EHCI driver (from FreeBSD, original license headers retained)
  - USB class enumeration; hot-plug events forwarded to TYKID


--- Networking ---

  - lwIP (LightweightIP) network stack integrated
  - Intel e1000, ixgbe, igc NIC drivers
    (FreeBSD/OpenBSD origin; original license headers retained)
  - IOMMU-gated: NIC drivers will not load without IOMMU present


--- Audio ---

  - Intel HDA controller driver
  - Codec enumeration and path routing
  - PCM output and input streams with DMA ring buffer
  - Master volume, per-channel volume, mute
  - Volume ramp (5% steps, ~1ms delay) to prevent click artefacts
  - Jack detection via unsolicited HDA responses
  - Parametric EQ: up to 10 peaking biquad bands, Q15 fixed-point,
    pure integer arithmetic (no FP in kernel)
  - TYKID-gated: hda_init() only called after driver attestation passes


--- Graphics ---

  - VESA/GOP framebuffer graphics
  - kprintf output to framebuffer console and UART


--- Debug Shell ---

  - ksh: kernel-side debug shell available after boot
  - /init.cfg support for automated startup commands
  - Debug dump functions for HDA controller, codec, and mixer state
    (compiled out in release builds with NDEBUG)


--- Build System ---

  - make          — build kernel ELF
  - make run      — 4 SMP, VirtIO-Net, VirtIO block, xHCI, Intel HDA
  - make run-debug
  - make run-ahci / run-nvme / run-e1000 / run-smp1
  - make run-tap / run-tap-e1000 / run-igc / run-tap-igc
  - make clean / clean-disk
  - SMP_CPU_COUNT override supported


--- Third-Party Components ---

  FreeBSD / OpenBSD  — EHCI, e1000, ixgbe, igc drivers
                       (BSD license; headers intact in source files)
  BearSSL            — Cryptographic library used by TYKID
  uACPI              — ACPI library
  LightweightIP      — Network stack
  Chan's FatFs       — exFAT and FAT filesystem support