# ARK-OS Kernel Compilation Station

## Overview

The **Kernel Compilation Station** is an automated CI/CD build subsystem dedicated to compiling, packaging, and distributing the Linux Zen kernel for ARK-OS.

It tracks the official [Linux Zen Kernel](https://github.com/zen-kernel/zen-kernel) repository, continuously monitors for new upstream tags, and automates the production of pre-compiled kernel images and kernel modules for x86_64 hardware and virtual machine environments.

---

## Architecture and Automation

### 1. Automated Upstream Tracking
- **Upstream Repository:** `https://github.com/zen-kernel/zen-kernel.git`
- **Tag Detection:** The GitHub Actions workflow polls for new `v*-zen*` tags every 6 hours.
- **Triggering:** If a new upstream tag is released that does not yet exist as a release in this repository, the workflow executes automatically. Manual triggering is also available via `workflow_dispatch`.

### 2. Build Pipeline
The workflow in `.github/workflows/compile-kernel.yml`:
1. Clones the upstream Zen kernel at the detected tag.
2. Applies the selected target configuration (`config/laptop-zen.config` or `config/qemu-minimal.config`).
3. Executes `olddefconfig` to resolve any new configuration symbols deterministically.
4. Compiles the 64-bit kernel executable image (`bzImage`) and kernel modules.
5. Stages stripped modules (`INSTALL_MOD_STRIP=1`), symbol map (`System.map`), and kernel configuration.
6. Packages the distribution directory into `kernel.tar.xz`.
7. Splits the archive into chunks of at most 2,000 MB (`img_00`, `img_01`, etc.) to comply with release asset constraints.
8. Generates cryptographic `SHA256SUMS.txt` hashes and publishes a GitHub Release.

---

## Configuration Profiles

- **`config/laptop-zen.config`** (Default): Complete distribution configuration extracted from a physical x86_64 host running Linux Zen. Supports comprehensive hardware drivers (storage, networking, graphics, audio, USB, Wi-Fi) as well as QEMU/KVM virtual devices.
- **`config/qemu-minimal.config`**: Minimal kernel configuration optimized for fast compilation and headless QEMU emulation testing.

---

## Reassembling and Using Release Artifacts

Release artifacts are split into sequential parts prefixed with `img_`. Follow these steps to reassemble and extract the kernel:

```bash
# 1. Download all img_* parts and SHA256SUMS.txt into a single directory

# 2. Verify archive integrity
sha256sum -c SHA256SUMS.txt

# 3. Concatenate the split parts into the compressed archive
cat img_* > kernel.tar.xz

# 4. Extract the distribution tree
tar -xvf kernel.tar.xz
```

### Extracted Directory Structure

```text
dist/
├── boot/
│   ├── bzImage             # 64-bit kernel binary (for QEMU -kernel or EFI boot)
│   ├── config              # Kernel configuration used for compilation
│   ├── kernel-version.txt  # Upstream release tag string
│   └── System.map          # Kernel symbol map
└── modules/
    └── lib/
        └── modules/
            └── <version>/  # Stripped kernel driver modules
```

---

## License

This repository and build station assets are licensed under the Apache License, Version 2.0. See [LICENSE](file:///home/aarav/repo/Stations/Kernel/LICENSE) for the complete license text. The compiled Linux Zen kernel itself is licensed under the GNU General Public License, Version 2 (GPL-2.0).
