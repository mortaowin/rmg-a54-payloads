# a54x-A546EXXSNFZI1 — Galaxy A54 5G (Exynos 1380)

| | |
| --- | --- |
| Models | SM-A546E / SM-A546B / SM-A546S / SM-A5460 |
| Firmware | A546EXXSNFZI1 (One UI 8.5, Android 16) |
| Kernel | `5.15.189-android13-3-33470412` |
| Status | **Device-tested 2026-10-03**: full root + KernelSU late-load, `su` under enforcing |

## Device-tested chain

1. Payload must run on a **fresh boot** (`requiresFreshP0Session`) — succeeds on
   attempt 1/8 at low uptime, 4/4 boots reproduced.
2. Exploit = July-fork physical-P0 mechanism **rebuilt with real SNFZI1 data offsets**
   (`cve-2026-43499-app.so`, 131992 B, md5 `1b71d10b3e2dd9c8a0bb0acd3ba5be8d`).
3. KernelSU late-load via `ksud-a54x-A546EXXSKFZF4-kdp` (embeds the
   `android13-5.15.189_kernelsu` LKM; kernel Image is byte-identical KFZF4↔SNFZI1).
   The KSU manager app reports working; `su` stays available under SELinux enforcing.

## Porting notes (SNFZI1 vs the KFZF4 target.h)

- `.text` symbols byte-identical; **data symbols shifted**:
  `misc_fops +0x50`, ashmem ioctl family `+0xCC`, `ashmem_fops −0xd8`,
  `kmalloc_caches −0x100`, `anon_pipe_buf_ops −0xc0`.
- `MM_STRUCT_SZ` must stay **0x400** (slab stride, not `sizeof` 0x3E0) or the
  mm_struct scanner finds 0 collisions.
- Do **not** port the main-branch `SLIDE_STACK_WRITER` PI dance to this kernel
  (android13-3): both mcast and sigreturn routes panic. The fork mechanism is
  the working one.
- Root is volatile (RAM-only exploit, bootloader stays locked, nothing written
  to boot partition). Re-run after every reboot.

## Warning

Do not OTA/update firmware off SNFZI1 — offsets are build-specific.
