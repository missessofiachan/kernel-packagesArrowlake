# kernel-packages (Arrow Lake-S & Nvidia Edition)

[![Build (fedora)](https://github.com/missessofiachan/kernel-packagesArrowlake/actions/workflows/fedora.yaml/badge.svg)](https://github.com/missessofiachan/kernel-packagesArrowlake/actions/workflows/fedora.yaml)
[![License: GPL-2.0](https://img.shields.io/badge/License-GPL%202.0-blue.svg)](LICENSE)

An optimized [OpenGamingCollective (OGC)](https://github.com/OpenGamingCollective/linux) Linux kernel compilation pipeline tailored specifically for **Intel Core Ultra 7 265K (Arrow Lake-S)** hybrid desktop CPUs, **NVIDIA GeForce RTX (Ada Lovelace / RTX 4080 Super)**, and high-performance **NVMe PCIe 4.0** gaming rigs.

Published as signed, bootc-compatible OCI image artifacts on GitHub Container Registry (GHCR) for downstream integration into [bazzite-arrowlake](https://github.com/missessofiachan/bazzite-arrowlake).

---

## 🚀 Key Highlights & Hardware Tuning

### 1. Arrow Lake-S Native Compilation (`-march=arrowlake-s`)
The kernel spec ([`fedora/kernel.spec`](fedora/kernel.spec)) injects native compiler flags directly into rpmbuild's `%build` section:
```bash
export KCFLAGS="-O3 -march=arrowlake-s -mtune=arrowlake-s -pipe"
export KCPPFLAGS="-O3 -march=arrowlake-s -mtune=arrowlake-s -pipe"
export HOSTCFLAGS="-O3 -march=arrowlake-s -mtune=arrowlake-s -pipe"
export HOSTCXXFLAGS="-O3 -march=arrowlake-s -mtune=arrowlake-s -pipe"
```
This unlocks hardware instruction sets natively supported by Arrow Lake-S (Lion Cove P-cores & Skymont E-cores), including **AVX-VNNI-INT8/INT16**, **AVX-IFMA**, **CMPccXADD**, and tuned scheduler branches, avoiding generic x86-64 overhead.

---

### 2. Tailored Kconfig Profile ([`config/ogc.config.set`](config/ogc.config.set))

#### ⚡ Intel Core Ultra 7 265K Hybrid Architecture
- **Hardware Feedback Interface (HFI / Thread Director):** `CONFIG_INTEL_HFI_THERMAL=y` and `CONFIG_X86_INTEL_PSTATE=y` provide real-time runtime efficiency and thermal classification to the scheduler.
- **Skymont L2 Cluster Awareness:** `CONFIG_SCHED_CLUSTER=y`, `CONFIG_SCHED_MC=y`, `CONFIG_SCHED_MC_PRIO=y`, `CONFIG_SCHED_SMT=y`, and `CONFIG_SCHED_AUTOGROUP=y` prioritize latency-critical threads to Lion Cove P-cores while balancing batch/background jobs across 4-core Skymont E-core clusters.
- **sched_ext / scx:** `CONFIG_SCHED_CLASS_EXT=y` enables Bazzite eBPF gaming schedulers (`scx_lavd`, `scx_bpfland`, `scx_rustland`).
- **Intel Virtualization & Thermal:** `CONFIG_KVM_INTEL=m`, `CONFIG_INTEL_IOMMU=y`, `CONFIG_INTEL_IOMMU_DEFAULT_ON=y`, `CONFIG_SENSORS_CORETEMP=m`.
- **Integrated Xe Media & NPU:** `CONFIG_DRM_XE=m`, `CONFIG_DRM_I915=m`, `CONFIG_DRM_ACCEL_IVPU=m` for QuickSync video encoding/decoding and AI inference.

#### 🎮 Gaming & Low-Latency Desktop Timers
- **Built-in NTSYNC:** `CONFIG_NTSYNC=y` compiles Windows NT synchronization primitives directly into `vmlinuz` so `/dev/ntsync` is available instantly on boot without modprobe or external dkms modules.
- **1000Hz Preemptive Kernel:** `CONFIG_HZ_1000=y`, `CONFIG_HZ=1000`, `CONFIG_PREEMPT=y`, `CONFIG_PREEMPT_DYNAMIC=y`, `CONFIG_NO_HZ_IDLE=y` provide precise sub-millisecond timer resolution and minimum input jitter.
- **Futex & Futex PI:** `CONFIG_FUTEX=y`, `CONFIG_FUTEX_PI=y` for low-overhead game engine synchronization.

#### 💽 Samsung 990 / 980 PRO PCIe 4.0 NVMe Storage
- **Built-in NVMe Core:** `CONFIG_NVME_CORE=y`, `CONFIG_BLK_DEV_NVME=y`, `CONFIG_NVME_MULTIPATH=y`.
- **Multi-Queue I/O & Throttling:** `CONFIG_BLK_WBT=y`, `CONFIG_BLK_WBT_MQ=y`, `CONFIG_IO_URING=y` prevent storage writeback starvation during high disk I/O in games.
- **NVMe Hardware Monitoring:** `CONFIG_NVME_HWMON=y` for direct temperature sensor telemetry.

#### 🧠 32GB DDR5-6000 Memory & Cache Subsystem
- **Multi-Gen LRU (MGLRU):** `CONFIG_LRU_GEN=y`, `CONFIG_LRU_GEN_ENABLED=y` prevent high page-fault stalls and frame drops under memory pressure.
- **Transparent Hugepages:** `CONFIG_TRANSPARENT_HUGEPAGE=y`, `CONFIG_TRANSPARENT_HUGEPAGE_MADVISE=y`, `CONFIG_COMPACTION=y`.
- **ZRAM Compression:** `CONFIG_ZRAM=m`, `CONFIG_ZSMALLOC=y`, `CONFIG_CRYPTO_ZSTD=y`.

#### 🟢 NVIDIA GeForce RTX 4080 Super Integration
- **Nouveau Disabled:** `CONFIG_DRM_NOUVEAU=n` prevents driver collision with NVIDIA kernel drivers (`akmod-nvidia`).
- **HMM & Memory Sharing:** `CONFIG_ZONE_DEVICE=y`, `CONFIG_HMM_MIRROR=y`, `CONFIG_DMABUF_HEAPS=y`, `CONFIG_DMABUF_HEAPS_SYSTEM=y`, `CONFIG_SYNC_FILE=y`.
- **Boot Splash:** `CONFIG_SYSFB_SIMPLEFB=y`, `CONFIG_DRM_SIMPLEDRM=y` for seamless Wayland handover.

#### 🛡️ Bazzite, Bootc & Security Primitives (Mandatory)
- `CONFIG_BPF_SYSCALL=y` & `CONFIG_DEBUG_INFO_BTF=y` for Bazzite sched_ext, bpftune, and cardwire.
- `CONFIG_SECURITY_SELINUX=y`, `CONFIG_OVERLAY_FS=y`, `CONFIG_BTRFS_FS=y` for immutable ostree/bootc rootfs support.
- `CONFIG_ANDROID_BINDER_IPC=y` for Waydroid emulation support.

#### ✂️ Stripped Unused AMD & Handheld Hardware
To minimize attack surface, reduce binary bloat, and accelerate build times, all non-relevant hardware drivers are disabled:
- **AMD GPU & Acceleration:** `CONFIG_DRM_AMDGPU=n`, `CONFIG_DRM_RADEON=n`, `CONFIG_DRM_ACCEL_AMDXDNA=n`, `CONFIG_HSA_AMD=n`, `CONFIG_X86_AMD_PSTATE=n`, `CONFIG_KVM_AMD=n`, `CONFIG_AMD_IOMMU=n`.
- **Handhelds & Chromebooks:** Steam Deck (`CONFIG_MFD_STEAMDECK=n`, `CONFIG_DRM_AMD_COLOR_STEAMDECK=n`), ROG Ally (`CONFIG_HID_ASUS_ALLY=n`), Legion Go (`CONFIG_HID_LENOVO_GO=n`), AyaNeo (`CONFIG_AYANEO_EC=n`, `CONFIG_SND_SOC_AW87XXX=n`), OneXPlayer, GPD Win, and ChromeOS EC.

---

## 📦 Output Artifacts & OCI Registry

Builds are packaged via ORAS as an OCI artifact container image and published to GHCR:

```
ghcr.io/missessofiachan/kernel-packagesarrowlake-fedora:<VERSION>-<TARGET>
```

| Tag Stream | Description |
| :--- | :--- |
| `latest-fc44` | Latest stable kernel release built against Fedora 44 (Bazzite base) |
| `sha-<HASH>-fc44` | Content-hashed immutable build artifact |
| `latest` | Default OCI stream tag |

### Package Inspection
Pull and inspect the built RPMs using `oras`:
```bash
oras pull ghcr.io/missessofiachan/kernel-packagesarrowlake-fedora:latest
ls -la *.rpm
```

---

## 🔄 Consuming in downstream images (`bazzite-arrowlake`)

In [`bazzite-arrowlake`](https://github.com/missessofiachan/bazzite-arrowlake), the kernel RPMs are extracted directly into the container build context:

```dockerfile
# Containerfile
COPY --from=ghcr.io/missessofiachan/kernel-packagesarrowlake-fedora:latest / /tmp/ogc-rpms

RUN /ctx/build_files/build.sh
```

During `build.sh`:
1. Stock Fedora kernel packages are removed.
2. Custom Arrow Lake-S kernel RPMs from `/tmp/ogc-rpms` are installed.
3. Pre-compiled matching NVIDIA kernel modules (`kmod-nvidia` from Negativo17 615.x) and auxiliary kmods (`xone`, `v4l2loopback`) are installed directly with self-healing fallback:
   - If the prebuilt kmod matches Bazzite's `nvidia-driver`, it installs in seconds.
   - If mismatched, `build.sh` automatically compiles the matching driver in-place with `akmods`.
4. Dracut rebuilds a bootc-compatible reproducible initramfs.

---

## 🛠️ GitHub Actions Workflow

The builder is defined in [`.github/workflows/fedora.yaml`](.github/workflows/fedora.yaml):
- **Triggers:**
  - Automatic on push to `main`.
  - Manual execution via `workflow_dispatch` with an optional version tag override.
- **Signing & Attestation:**
  - Keyless Sigstore signing via Cosign (`cosign sign --yes`).
  - GitHub build provenance attestation (`actions/attest-build-provenance`).
  - Non-blocking signing (`continue-on-error: true`) for personal fork reliability.

---

## 📄 License

GPL-2.0. Upstream patches, source code, and configurations are maintained by the [OpenGamingCollective](https://github.com/OpenGamingCollective).
