
# `debug_quirks`

The `sdhci.debug_quirks` parameter in Linux is used to **debug or tweak the behavior of the SDHCI (Secure Digital Host Controller Interface) driver**, which manages SD and eMMC card interfaces on Linux systems.

---

### 🧠 What are "quirks"?

In the context of Linux kernel drivers, *quirks* are **special adjustments to deal with hardware peculiarities**, especially for controllers or devices that don’t fully comply with standards. These tweaks can enable or disable specific features, fix bugs, or improve compatibility with certain chips.

---

### 📌 `sdhci.debug_quirks` — Function

The `sdhci.debug_quirks` parameter allows you to **manually force the activation of specific quirks** in the `sdhci` driver. This can be useful for resolving issues like SD card read/write failures, performance problems, device initialization errors, and other anomalous behaviors.

It’s typically used as a kernel boot parameter (e.g., via GRUB):

```bash
sdhci.debug_quirks=0xVALUE
```

---

### 🔢 Possible Values

You define `sdhci.debug_quirks` as a **bitmask** (in hexadecimal), where each bit or group of bits enables a specific quirk. These values vary by kernel version, but common ones include:

| Value (bit) | Quirk                                                   |
|-------------|----------------------------------------------------------|
| `0x01`      | Ignores card state bit (`NO_CARD_NO_RESET`)              |
| `0x02`      | Forces reset on every card insertion                     |
| `0x04`      | Disables DMA (uses PIO instead of DMA)                   |
| `0x08`      | Disables tuning (avoids fine clock adjustment)           |
| `0x10`      | Avoids access to certain registers                       |
| `0x20`      | Disables High Speed (HS) support                         |

---

### 🔧 How to Use

1. **Test SD card issues:**
   If your SD card fails to mount or has I/O errors, try disabling some features:

   ```bash
   sdhci.debug_quirks=0x04
   ```

2. **Temporarily edit GRUB:**
   At the GRUB menu, edit the kernel line (`linux ...`) and add at the end:

   ```
   sdhci.debug_quirks=0x04
   ```

3. **Make it persistent:**
   Edit the GRUB config file:

   ```bash
   sudo nano /etc/default/grub
   ```

   Add at the end of the `GRUB_CMDLINE_LINUX_DEFAULT` line:

   ```
   sdhci.debug_quirks=0x04
   ```

   Then update GRUB:

   ```bash
   sudo update-grub
   ```

---

### 📎 Tip: How to Discover Available Values

You can check the kernel source code (drivers `sdhci.c` or `sdhci-pci.c`) or run:

```bash
modinfo sdhci
```

Or search for quirks in the kernel repository:

```bash
grep -i quirk /usr/src/linux/drivers/mmc/host/sdhci*
```

---

# Values

Here is a more complete overview of the possible values (flags) you can use with `sdhci.debug_quirks` (and with `sdhci.debug_quirks2`, available in newer kernels):

---

## 🛠️ Main flags for `debug_quirks` (bitmask)

Although the exact list may vary by kernel version, commonly found values in `drivers/mmc/host/sdhci.h` include:

- `0x01` — **SDHCI_QUIRK_BROKEN_CARD_DETECTION**: ignores card state when GPIO detection fails.  
- `0x02` — **SDHCI_QUIRK_NO_POWER_ON_RESET**: avoids automatic reset on card insertion.  
- `0x04` — **SDHCI_QUIRK_BROKEN_DMA**: disables DMA (uses PIO).  
- `0x08` — **SDHCI_QUIRK_NO_HISPD**: disables High Speed support.  
- `0x10` — **SDHCI_QUIRK_NO_ENDATTR_IN_NOPDESC**: avoids access to certain registers.  
- `0x20` — **SDHCI_QUIRK_BROKEN_TIMEOUT_VAL**: fixes incorrect timeout values.  
- `0x40` — **SDHCI_QUIRK_BROKEN_ADMA**: disables ADMA (as recommended on AskUbuntu).

These flags are especially useful for working around card detection issues, timeouts, DMA failures, or high-speed mode bugs.

---

## 🛠️ Flags for `debug_quirks2`

Introduced in more recent kernels, this parameter allows additional quirks:

- `0x04` (example): downgrade from Ultra-HS to HS.
- Other bits like `SDHCI_QUIRK2_NO_1_8_V`, `SDHCI_QUIRK2_SLOW_INT_CLR`, `SDHCI_QUIRK2_ALWAYS_USE_BASE_CLOCK`, etc., control voltage behavior, interrupt handling, clock use, and retuning logic.

---

## 📌 Practical Examples

- `sdhci.debug_quirks=0x40` — forces ADMA off; useful for Broadcom readers or DMA issues.
- `sdhci.debug_quirks=0x20000` — disables CQE (Command Queue Engine); useful when formatting eMMC.
- `sdhci.debug_quirks2=0x4` — downgrades Ultra-HS to HS for compatibility.

---

## 🧩 How to Find Individual Bits

1. Check the driver header in your local kernel source:

   ```bash
   grep -R "SDHCI_QUIRK_" /usr/src/linux/drivers/mmc/host/sdhci.h
   ```

2. Or consult your kernel version’s documentation.

---

## 📝 Summary

- **`debug_quirks`**: bitmask (0x…) to enable main quirks — detection, reset, DMA/ADMA, speed, timeout, etc.
- **`debug_quirks2`**: more advanced quirks for newer kernels.
- Hexadecimal values can combine multiple flags (e.g., `0x40 | 0x04 = 0x44`).
