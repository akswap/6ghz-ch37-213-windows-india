# Windows 6 GHz Wi‑Fi Testing in India — Intel AX411 / BE200

This repository documents my **real Windows PC testing of 6 GHz Wi‑Fi channels from the lower to upper 6 GHz range**, using Intel Wi‑Fi adapters **without modifying Windows system files, Intel firmware, regulatory databases, or patched Wi‑Fi drivers**.

> **Important:** This is a technical test record, not a regulatory-bypass guide. Channel availability and permitted operation depend on the AP, client, firmware, driver, and local spectrum regulations. Use only frequencies/channels permitted for your location and equipment.

## Tested Platform

| Component | Tested hardware |
|---|---|
| Motherboard | **Gigabyte B760M DS3H AX Rev. 1.2** |
| Wi‑Fi adapters | **Intel Wi‑Fi 6E AX411** and **Intel Wi‑Fi 7 BE200 320MHz** |
| Operating system | Windows |
| Confirmed full-band/high-channel result | **Intel BE200 (standalone PCIe/M.2 adapter)** |
| Built-in AX411 result | **Tested, but the same full-band behavior was not reproduced** |
| Tested Intel driver | **23.0.6.4** |
| Windows Home Location | **India (GeoId 113)** |

The confirmed motherboard is **Gigabyte B760M DS3H AX Rev. 1.2**. Similar Intel 12th Gen or newer systems may behave similarly, but other platforms have not all been individually verified.

## Important AX411 vs BE200 Hardware Note

The motherboard-integrated **Intel AX411** did **not** reproduce the same full 6 GHz/high-channel behavior in this system, even after applying the same normal Intel driver settings and test procedure.

The successful result documented in this repository is from a **standalone Intel BE200 installed as a PCIe/M.2 Wi‑Fi adapter**.

This is an important distinction:

```text
Built-in Intel AX411
→ CNVio2 / platform-integrated path
→ Tested on this motherboard
→ Same full-band behavior NOT reproduced

Standalone Intel BE200
→ PCIe/M.2 adapter path
→ Intel driver 23.0.6.4
→ High 6 GHz channel behavior reproduced
→ Ch 101 active connection confirmed
→ Ch 165 scan visibility confirmed
```

This strongly suggests that the result depends on the **specific adapter architecture plus Intel driver/firmware path**, and is not simply the result of a generic Windows registry setting.

### AX210 — strong candidate for testing

The **Intel AX210** is a standalone PCIe/USB M.2 Wi‑Fi 6E adapter, so architecturally it is a much better candidate for this experiment than CNVio2-based AX211/AX411.

There are already multiple public user reports showing that AX210 6 GHz operation can depend strongly on Intel driver generation. In several Intel Community and Microsoft Q&A reports, users found that older AX210 driver versions restored 6 GHz visibility/connectivity when newer drivers did not.

Examples reported publicly include older AX210 drivers such as **22.45.1.1** restoring 6 GHz on systems where later 22.x releases only showed 2.4/5 GHz. Some users also reported that 6 GHz discovery with the older driver could take up to a few minutes.

These reports **support AX210 as a plausible standalone-PCIe candidate**, but they do **not yet prove the same Channel 37–213 / Ch 101 / Ch 165 behavior documented here with BE200**. That specific high-channel behavior still needs direct AX210 testing.

Current compatibility view:

```text
AX210 (PCIe/USB M.2) → Not yet tested here; strong candidate. Public users have reported 6 GHz working with older Intel drivers.
AX211 (CNVio2)       → Different platform-integrated architecture.
AX411 (CNVio2)       → Tested here; full-band/high-channel behavior not reproduced.
BE200 (PCIe/USB M.2) → Confirmed working in this project.
```

If further AX210, AX211, AX411 or other Intel adapter results are obtained, this section will be updated.

## What "No Modification" Means

No custom or patched Intel driver is used. No registry hack, Windows system-file modification, regulatory database edit, Wi‑Fi card firmware patch, Linux `iw reg set`, Magisk/root, or modified client software is required on the Windows PC.

The working driver package is an Intel Windows Wi‑Fi driver package, and the successful test uses the normal Intel driver stack reported by Windows as **23.0.6.4 / Netwtw14**.

The registry export from the working BE200 installation was also checked for obvious user-level regulatory overrides. No explicit `DisableLAR`, `CountryCode`, `RegDomain`, `RegulatoryDomain`, FCC/world-domain override, or similar country-forcing value was found. The visible 6 GHz settings were ordinary adapter options such as 6 GHz enabled and 6 GHz channel width set to Auto.

This is important because the result is being documented as **driver behavior observed on an unmodified Windows installation**, not as the result of a hidden registry or country-code hack.

The AP/router must still be capable of operating on the selected channel and must be configured in accordance with the rules that apply in your location.

## Required / Tested Intel Driver

My reproducible result uses:

```text
Intel Wi‑Fi Driver: 23.0.6.4
Driver branch/service: Netwtw14
```

For the closest reproduction, use the same driver version shown in the test screenshot. If a different Intel Wi‑Fi driver is already installed, completely remove that package before installing **23.0.6.4**.

### Clean driver installation

1. Temporarily disconnect the PC from the Internet so Windows Update does not immediately replace the driver.
2. Open **Device Manager → Network adapters**.
3. Select the Intel Wi‑Fi adapter.
4. Choose **Uninstall device**.
5. If Windows shows **Attempt to remove the driver for this device**, select it.
6. Reboot if requested.
7. Install Intel Wi‑Fi driver **23.0.6.4**.
8. Reboot Windows.
9. Verify the version under **Device Manager → adapter → Properties → Driver**.

If Windows keeps reinstalling another Intel package from the Driver Store, remove the unwanted package first and then reinstall 23.0.6.4. Do not let Windows Update replace the test driver while reproducing the result.

## Why the older Intel driver can behave differently

A strings-level comparison was made between the **working Netwtw14** driver binary and a **newer Netwtw18** branch binary.

Both binaries contain Intel LAR/regulatory-related logic. Therefore, the working result should **not** be described as "LAR removed" or "LAR disabled". LAR-related code exists in the working Netwtw14 driver too.

However, the newer Netwtw18 branch contains newer regulatory/LARI interfaces and additional channel-filtering-related routines.

Examples observed in the binaries include:

```text
Working Netwtw14:
  mhiffLariChangeConfigCmdVer6
  EV_MMAC_LAR_MCC_NOTIFICATION
  EV_MMAC_LAR_MCC_TEST_MODE_STATE_CHANGED
  7001 - 11d Location MCC value (WRDD)
  mUtilReadNewRegulatoryLimits
  mhiffTranslatRegulatoryTasConfigVer4

Newer Netwtw18:
  mhiffLariChangeConfigCmdVer12
  mhiffLariChangeConfigCmdVer13
  mhiffLariConfigOverrideCmdVer1
  prvLarRemoveBssEntriesWithInvalidChannels
  prvUpdateHe160RatesAcordingToLar
  getWifiCountryRegionList
  prvBssVifTelemetryGetCountryCodeFromIeMap
  prvhSpctrmHandleCountryIe
  prvhSpctrmSet11dCountryOid
  mhiffTranslatRegulatoryTasConfigVer5
```

The most interesting newer-driver symbol is:

```text
prvLarRemoveBssEntriesWithInvalidChannels
```

Its name strongly suggests that newer LAR logic can remove BSS scan entries that the driver considers to be on invalid channels.

This matches the practical observation that the older **23.0.6.4 / Netwtw14** branch exposes more of the tested 6 GHz channel range, while newer Intel branches can behave more restrictively.

### Important interpretation

This comparison is **evidence of a driver-branch implementation change**, not proof of the exact internal decision path. A strings comparison alone cannot prove which function is ultimately responsible for a particular channel being shown or hidden.

The safe conclusion is:

> Both Netwtw14 and Netwtw18 contain Intel LAR/regulatory logic, but Netwtw18 includes newer LARI interfaces and explicit channel/BSS filtering-related routines. This indicates that Intel substantially revised regulatory filtering in the newer driver branch.

That distinction is important: this project does **not** rely on a patched Intel `.sys` file, a disabled-LAR registry trick, or a forced country-domain value. The difference being documented is the behavior of different Intel driver generations.

## Tested with Windows Home Location set to India

Before the final retest, Windows Home Location was changed to India:

```powershell
Get-WinHomeLocation
```

Confirmed result:

```text
GeoId HomeLocation
----- ------------
113   India
```

The PC was then rebooted before testing again.

After reboot, the Intel BE200 still connected successfully on **6 GHz Channel 101**, and the normal Windows WLAN scan still detected an **802.11be AP on Channel 165**.

> Windows Home Location and the Wi‑Fi radio's regulatory domain are not necessarily the same mechanism. This section only documents the actual Windows Home Location used during the successful retest.

## 6 GHz Channel Observations

For my 320 MHz / Wi‑Fi 7 testing, the most useful channel positions have been:

| Channel | Practical observation |
|---:|---|
| **37** | Best / easiest visibility in my tests |
| **101** | Very easy to detect and connect; confirmed active Wi‑Fi 7 link |
| **165** | Works and is detectable, but discovery has been less consistent in repeated tests |

Channels **37 and 101** have been the easiest for repeated client discovery. Channel **165** can also be detected successfully.

## Active Connection Proof — Channel 101

Windows Network Properties on the Intel BE200 reports:

```text
SSID           : TP-Link_5G-6G_MLO
Protocol       : 802.11be
Adapter        : Intel(R) Wi‑Fi 7 BE200 320MHz
Driver Version : 23.0.6.4
Network Band   : 6 GHz
Channel        : 101
Link Speed     : 4323 / 4323 Mbps
Security       : WPA3-Personal
```

This confirms an active **802.11be / Wi‑Fi 7 connection on 6 GHz Channel 101**.

### Link-speed observation

The screenshot currently published in the test record shows **4323 / 4323 Mbps**. In my testing, the Intel BE200 320 MHz link has also reached approximately **5764 Mbps** under favorable conditions.

The exact reported PHY/link rate changes dynamically with signal quality, MCS, spatial streams and channel conditions, so the value shown by `netsh` can be lower at any particular moment.

> A dedicated screenshot showing the ~5764 Mbps result can be added as additional proof when available.

## Live Interface Proof — `netsh wlan show interfaces`

Command:

```powershell
netsh wlan show interfaces
```

Relevant output from the India-region retest:

```text
Description            : Intel(R) Wi-Fi 7 BE200 320MHz
State                  : connected
SSID                   : TP-Link_5G-6G_MLO
AP BSSID               : ba:6e:84:e3:5d:f5
Band                   : 6 GHz
Channel                : 101
Radio type             : 802.11be
Authentication         : WPA3-Personal (H2E)
Receive rate (Mbps)    : 3458.8
Transmit rate (Mbps)   : 1729.4
Signal                 : 89%
Rssi                   : -49
```

The same interface output also exposes colocated AP information for related 5 GHz links.

The `netsh` receive/transmit values are live rates at the moment the command is run and therefore do not need to match the higher aggregated link-speed value shown in Windows Network Properties.

## PowerShell / NETSH Scan Evidence

Standard Windows command used:

```powershell
netsh wlan show networks mode=bssid
```

### Channel 101 — Wi‑Fi 7 / 6 GHz visible

Relevant output:

```text
SSID : TP-Link_5G-6G_MLO
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
SSID : TP-Link_6G_be
    Authentication          : WPA3-Personal
    Encryption              : CCMP
    BSSID 1                 : ba:6e:84:e3:5d:f4
         Signal             : 90%
         Radio type         : 802.11be
         Band               : 6 GHz
         Channel            : 101
         Details            : (H2E Required) (MLD)
```

The scan also exposes colocated 5 GHz/6 GHz BSS information and MLD metadata, showing that Windows is parsing the Wi‑Fi 7 multi-link information advertised by the AP.

### Channel 165 — Wi‑Fi 7 / 6 GHz visible after India-region reboot

The same post-reboot scan detected another 6 GHz Wi‑Fi 7 AP on Channel 165:

```text
SSID 1 : MobSoftAP_Router
    Network type            : Infrastructure
    Authentication          : WPA3-Personal
    Encryption              : CCMP
    BSSID 1                 : 36:63:b2:35:86:52
         Signal             : 86%
         Radio type         : 802.11be
         Band               : 6 GHz
         Channel            : 165
         Details            : (H2E Required) (MLD)
         MFP Required       : 1
```

So Channel 165 is not just theoretical in this setup: the normal Windows WLAN scanner can see an **802.11be AP on 6 GHz Channel 165** after reboot with **Windows Home Location set to India**.

## Current Evidence Summary

| Evidence | Ch 37 | Ch 101 | Ch 165 |
|---|:---:|:---:|:---:|
| Seen in my BE200 testing | ✅ | ✅ | ✅ |
| NETSH scan excerpt published here | To add | ✅ | ✅ |
| `802.11be` shown by Windows | To add | ✅ | ✅ |
| Active connection proof published | To add | ✅ | To add |
| Tested after Windows Home Location = India + reboot | To add | ✅ | ✅ visibility |
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

## Tested / Candidate Adapters

| Adapter | Interface type | 6 GHz | Full-band/high-channel status |
|---|---|:---:|---|
| Intel AX210 | Standalone PCIe/USB M.2 | ✅ | **Not tested here; strong candidate. Public user reports confirm 6 GHz can work with older Intel drivers** |
| Intel AX211 | CNVio2 | ✅ | Different platform-integrated architecture; not tested here for this behavior |
| Intel AX411 | Built-in / CNVio2 | ✅ | ❌ Same behavior not reproduced on this system |
| Intel BE200 | Standalone PCIe/USB M.2 | ✅ | ✅ Confirmed; Ch 101 active link and Ch 165 visibility evidence shown |

## Compatibility Notes

This result should not be treated as a guarantee for every PC or every Intel adapter. Behavior can vary with:

- Intel CPU/platform generation
- motherboard M.2 / CNVio / PCIe implementation
- adapter architecture
- BIOS version
- Windows build
- Intel driver version
- router/AP firmware
- AP regulatory configuration

The **Gigabyte B760M DS3H AX Rev. 1.2 + standalone Intel BE200** combination is the confirmed reference platform for the full-band/high-channel result documented here.

## Why This Repository Exists

The goal is to document actual Windows client behavior using normal Intel hardware and the normal Windows WLAN stack, with reproducible screenshots and `netsh` output instead of relying on assumptions about what a client should or should not see.

If you reproduce the test on another Intel platform, please include:

- motherboard and CPU
- whether the Wi‑Fi adapter is built-in/CNVio or standalone PCIe/M.2
- Intel Wi‑Fi adapter model
- exact driver version
- Windows version/build
- Windows Home Location
- AP/router model and firmware
- selected 6 GHz channel
- `netsh wlan show networks mode=bssid` output
- connected-link screenshot or `netsh wlan show interfaces` output

## Disclaimer

This repository documents laboratory/personal test observations. It does **not** grant permission to operate on frequencies that are restricted in your jurisdiction. Regulatory limits differ by country and can change over time. Always follow the current rules applicable to your location and equipment.
