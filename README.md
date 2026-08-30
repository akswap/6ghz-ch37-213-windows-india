# Windows 6 GHz Wi‑Fi Testing in India — Intel AX411 / BE200

This repository documents my **real Windows PC testing of 6 GHz Wi‑Fi channels from the lower to upper 6 GHz range**, using Intel Wi‑Fi adapters **without modifying Windows system files, Intel firmware, regulatory databases, or patched Wi‑Fi drivers**.

> **Important:** This is a technical test record, not a regulatory-bypass guide. Channel availability and permitted operation depend on the AP, client, firmware, driver, and local spectrum regulations. Use only frequencies/channels permitted for your location and equipment.

## Tested Platform

| Component | Tested hardware |
|---|---|
| Motherboard | **Gigabyte B760M DS3H AX Rev. 1.2** |
| Wi‑Fi adapters | **Intel Wi‑Fi 6E AX411** and **Intel Wi‑Fi 7 BE200 320MHz** |
| Operating system | Windows |
| Confirmed Wi‑Fi 7 test | Intel BE200 /intel ax411 |
| Tested Intel driver | **23.0.6.4** |

The confirmed motherboard is **Gigabyte B760M DS3H AX Rev. 1.2**. Similar Intel 12th Gen or newer systems may behave similarly, but other platforms have not all been individually verified.

## What "No Modification" Means

No custom or patched Intel driver is used. No registry hack, Windows system-file modification, regulatory database edit, Wi‑Fi card firmware patch, Linux `iw reg set`, Magisk/root, or modified client software is required on the Windows PC.

The AP/router must still be capable of operating on the selected channel and must be configured in accordance with the rules that apply in your location.

## Required / Tested Intel Driver

My reproducible result uses:

```text
Intel Wi‑Fi Driver: 23.0.6.4
```

For the closest reproduction, use the same driver version shown in the test screenshot. If a different Intel Wi‑Fi driver is already installed, completely remove that package before installing **23.0.6.4**.

### Clean driver installation

1. Temporarily disconnect the PC from the Internet so Windows Update does not immediately replace the driver.
2. Open **Device Manager → Network adapters**.
3. Select **Intel(R) Wi‑Fi 6E AX411** or **Intel(R) Wi‑Fi 7 BE200 320MHz**.
4. Choose **Uninstall device**.
5. If Windows shows **Attempt to remove the driver for this device**, select it.
6. Reboot if requested.
7. Install Intel Wi‑Fi driver **23.0.6.4**.
8. Reboot Windows.
9. Verify the version under **Device Manager → adapter → Properties → Driver**.

If Windows keeps reinstalling another Intel package from the Driver Store, remove the unwanted package first and then reinstall 23.0.6.4. Do not let Windows Update replace the test driver while reproducing the result.

## 6 GHz Channel Observations

For my 320 MHz / Wi‑Fi 7 testing, the most useful channel positions have been:

| Channel | Practical observation |
|---:|---|
| **37** | Best / easiest visibility in my tests |
| **101** | Very easy to detect and connect; confirmed active Wi‑Fi 7 link |
| **165** | Works and is detectable, but discovery has been less consistent in repeated tests |

Channels **37 and 101** have been the easiest for repeated client discovery. Channel **165** can also be detected successfully; one scan below shows it at **92% signal**.

## Active Connection Proof — Channel 101

Windows Network Properties on the Intel BE200 reports:

```text
Protocol       : 802.11be
Adapter        : Intel(R) Wi‑Fi 7 BE200 320MHz
Driver Version : 23.0.6.4
Network Band   : 6 GHz
Channel        : 101
Link Speed     : 4323 / 4323 Mbps
Security       : WPA3-Personal
```

This confirms an active **802.11be / Wi‑Fi 7 connection on 6 GHz Channel 101**.

> Windows Network Properties does not explicitly print the negotiated channel width on this page. The adapter name contains “320MHz”, and the observed 4323/4323 Mbps link is consistent with a very high-rate Wi‑Fi 7 connection, but I keep the screenshot claim limited to what Windows directly reports.

## PowerShell / NETSH Scan Evidence

Standard Windows command used:

```powershell
netsh wlan show networks mode=bssid
```

### Channel 101 — Wi‑Fi 7 / 6 GHz visible

Relevant output:

```text
SSID 3 : TP-Link_5G-6G_MLO
    Network type            : Infrastructure
    Authentication          : WPA3-Personal
    Encryption              : CCMP
    BSSID 1                 : ba:6e:84:e3:5d:f5
         Signal             : 89%
         Radio type         : 802.11be
         Band               : 6 GHz
         Channel            : 101
         Details            : (H2E Required) (MLD)
         MLD Address        : a8:6e:84:e3:5d:f5
```

A separate 6 GHz SSID from the same AP was also visible:

```text
SSID 4 : TP-Link_6G_be
    Authentication          : WPA3-Personal
    Encryption              : CCMP
    BSSID 1                 : ba:6e:84:e3:5d:f4
         Signal             : 88%
         Radio type         : 802.11be
         Band               : 6 GHz
         Channel            : 101
         Details            : (H2E Required) (MLD)
```

The scan also exposes colocated 5 GHz/6 GHz BSS information and MLD metadata, showing that Windows is parsing the Wi‑Fi 7 multi-link information advertised by the AP.

### Channel 165 — Wi‑Fi 7 / 6 GHz visible

A later scan detected another 6 GHz Wi‑Fi 7 AP on Channel 165:

```text
SSID 1 : MobSoftAP_Router
    Network type            : Infrastructure
    Authentication          : WPA3-Personal
    Encryption              : CCMP
    BSSID 1                 : 6e:8c:62:dd:77:dc
         Signal             : 92%
         Radio type         : 802.11be
         Band               : 6 GHz
         Channel            : 165
         Details            : (H2E Required) (MLD)
         MFP Required       : 1
```

So Channel 165 is not just theoretical in this setup: the normal Windows WLAN scanner can see an **802.11be AP on 6 GHz Channel 165** without modifying the Windows client stack.

## Current Evidence Summary

| Evidence | Ch 37 | Ch 101 | Ch 165 |
|---|:---:|:---:|:---:|
| Seen in my testing | ✅ | ✅ | ✅ |
| NETSH scan excerpt published here | To add | ✅ | ✅ |
| `802.11be` shown by Windows | To add | ✅ | ✅ |
| Active connection proof published | To add | ✅ | To add |
| Discovery consistency | Excellent | Excellent | Less consistent |

More raw results/screenshots can be added as additional channels are retested and documented.

## Save Your Own Scan Result

PowerShell can save the complete scan output directly:

```powershell
netsh wlan show networks mode=bssid > 6ghz-scan.txt
```

For the current connection, also run:

```powershell
netsh wlan show interfaces
```

These commands are useful because they preserve the actual Windows-reported **radio type, band, channel, signal, BSSID, H2E and MLD information**.

## Tested Adapters

| Adapter | Standard | 6 GHz | Status |
|---|---|:---:|---|
| Intel AX411 | Wi‑Fi 6E | ✅ | Tested |
| Intel BE200 | Wi‑Fi 7 / 802.11be | ✅ | Tested; Ch 101 and Ch 165 evidence shown |

## Compatibility Notes

This result should not be treated as a guarantee for every PC. Behavior can vary with:

- Intel CPU/platform generation
- motherboard M.2 / CNVio / PCIe implementation
- AX411 vs BE200 hardware interface requirements
- BIOS version
- Windows build
- Intel driver version
- router/AP firmware
- AP regulatory configuration

The **Gigabyte B760M DS3H AX Rev. 1.2** system is the confirmed reference platform.

## Why This Repository Exists

The goal is to document actual Windows client behavior using normal Intel hardware and the normal Windows WLAN stack, with reproducible screenshots and `netsh` output instead of relying on assumptions about what a client should or should not see.

If you reproduce the test on another Intel platform, please include:

- motherboard and CPU
- Intel Wi‑Fi adapter model
- exact driver version
- Windows version/build
- AP/router model and firmware
- selected 6 GHz channel
- `netsh wlan show networks mode=bssid` output
- connected-link screenshot or `netsh wlan show interfaces` output

## Disclaimer

This repository documents laboratory/personal test observations. It does **not** grant permission to operate on frequencies that are restricted in your jurisdiction. Regulatory limits differ by country and can change over time. Always follow the current rules applicable to your location and equipment.
