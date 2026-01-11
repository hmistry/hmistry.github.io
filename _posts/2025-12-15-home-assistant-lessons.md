---
layout: post
title: "Home Assistant: Lessons Learned"
date:  2025-12-15
categories: home-diy
tags: home-assistant raspberry-pi
excerpt_separator: <!--more-->
---

My first 3 months of using Home Assistant (HA) on a Raspberry Pi 5 were unreliable: frequent database corruption, failed boots and data loss. After researching and addressing the root causes, Home Assistant has now been stable and running smoothly for 6+ months.

Below are my experience-based tips that may help you build a stable Home Assistant platform.

<!--more-->

1. SD Card → SSD (Most Important Fix)

   - Problem:
     - SD cards failed repeatedly once backups exceeded ~100 MB
     - Resulted in database file corruption and failed boots

   - Fix:
     - Avoid SD cards entirely for long-term HA installs
     - Use a NVMe SSD for main disk

2. SQLite → MariaDB (Second Most Important Fix)

   - Problem:
     - SQLite database corruption occurred frequently
     - Unsure whether root cause was SD card I/O, SQLite itself, or bad (entity) data
     - Lost historical data multiple times and had to start fresh

   - Fix:
     - Switch recorder to MariaDB (HA add-on)
     - Much more stable, even with large backup sizes and data migrations during updates

3. Backups (Off-Device) (Third Most Important Thing)

   - Problem:
     - On-device backups don’t help when storage fails or file corruption occurs

   - Fix:
     - Back up frequently
     - Download backups off the HA disk periodically
     - Download and store backup encryption key off-device, see `Settings → System → Backup → Configure Backup Settings → Download Emergency Kit`
     - Worst-case you minimize data loss to your most recent backup or easily restore all the devices, integrations, and configurations for a fresh start

4. "Disable → Change → Enable" Strategy For Modifying Entities

   - Problem:
     - Modifying or deleting entities while active sometimes preceded DB issues
     - Using uninitialized entities in templates caused unpredictable behavior

   - Fix:
     - Disable entities before modifying or deleting
     - Re-enable after changes made and allow time for update events if applicable

5. Be Patient With Boot / Shutdown

   - Problem:
     - HA appears “hung” during boot or shutdown
     - Power-cycling too early made things worse by increasing file corruption risk

   - Fix:
     - Wait several minutes for Raspberry Pi to shutdown processes or boot up
     - Raspberry Pi is not a high-performance computer

6. HA Upgrades Can Fail On Reboot

   - Problem:
     - Failed boots after HA upgrades
     - Components with custom logic can fail on reboot after due to missing or uninitialized states

   - Fix:
     - Backup before every upgrade (download to off-device)
     - Harden custom logic/formulas against uninitialized, unknown, unavailable, none, or null states (see this [post on states][haUnknownStates])
     - Use the Template Editor under `Developer Tools → Template` to test your formulas or state values
     - Check `Developer Tools → Statistics` and fix any issues
     - Check logs for any critical errors before and after upgrading under `Settings → System → Logs` and select the services/add-ons in the dropdown button
     - Use "Disable → change → enable" strategy when modifying entities
     - Sometimes starting fresh (without importing bad historical data) is necessary

7. Always Validate Changes to `configuration.yaml`

   - Problem:
     - Invalid YAML configuration can prevent HA from booting

   - Fix:
     - Run "Check Configuration" under `Developer Tools → YAML → Check Configuration` after making changes and before restarting HA

8. Clear Cache to Fix Broken Dashboards

   - Problem:
     - Energy, Logs, or History pages fail to load but other dashboard works

   - Fix:
     - Test dashboard in a private browser window
     - If it works, clear caches
     - Desktop browser: hard reload / clear cache
     - iOS app: `Settings → Debug → Reset Frontend Cache`

9. Pairing Bluetooth Keyboard / Mouse via CLI to RPi

   - Reference: [Pairing Bluetooth Devices][pairBTDevices]
   - Shell Commands used:

     ```shell
     # On your computer terminal
     ssh pi@raspberrypi.local

     # On the Raspberry Pi
     sudo bluetoothctl
     agent on
     default agent
     scan on
     # Wait for device address
     scan off
     devices # To print list of devices found and their addresses
     # Replace AA:BB:CC:DD:EE:FF with your device address below
     pair AA:BB:CC:DD:EE:FF
     connect AA:BB:CC:DD:EE:FF
     trust AA:BB:CC:DD:EE:FF # Allows RPi to trust device and reconnect on reboot
     ```

10. NVMe SSD Installation

    - References:
      - [Raspberry Pi Boot Guide][rpiBootGuide]
      - [HA install on Raspberry Pi 5 with NVMe SSD][rpiSSDHAInstall]

11. Recover Files From a Failed HAOS Boot and No Off-Device Backup

    - Notes
      - Credits to HA community forum contributors for the solution (sorry, I lost the link to where I found it)
      - You can recover all the config files or on-device backups if you have the encryption key to unlock it on restore
      - [FileBrowser add-on][haFileBrowser]: GUI File browser
      - [Advanced SSH & Terminal add-on][haTerminal]: SSH Terminal
    - Steps:
      1. Original HA SD card fails to boot - put aside
      2. Use a new SD card and install a fresh copy of Home Assistant on it
      3. Put the new SD card in Raspberry Pi and setup new HA instance (no restoring backups)
      4. Install Advanced SSH/Terminal and FileBrowser add-ons
      5. Config SSH/Terminal and disable protection mode
      6. Insert failed SD card via USB port on Raspberry Pi
      7. Wait for notification in Settings saying multiple HAOS detected... make note of the USB path eg /dev/sda8 but do not fix
      8. In HA SSH Terminal, do the following:

         ```shell
         # login to correct container shell to access usb
         sudo docker exec -it homeassistant /bin/bash
         mkdir /config/tmp # create a tmp folder
         mount /dev/sda8 /config/tmp # mount SD card usb to tmp folder
         # => now you can browse files on bad SD card in FileBrowser under the mounted folder /config/tmp

         # To copy files over to new SD card HAOS:
         cd /config
         mkdir old_data
         cp -r /config/tmp/supervisor old_data/
         # => now all the files from the bad SD card are in the new HAOS /config/old_data

         # unmount USB
         umount /config/tmp
         exit # exits the docker bash shell and brings you back to your terminal shell
         ```

      9. Use FileBrowser to download /config/old_data
      10. Re-enable SSH/Terminal protection mode

## Summary

What ultimately stabilized my Home Assistant setup:

- SSD instead of SD card
- MariaDB instead of SQLite
- Defensive entity handling strategy
- Hardened custom logic/formula use in components
- Taking precautions when upgrading and having backup copies

Now Home Assistant behaves like a reliable system!

[pairBTDevices]: https://www.youtube.com/watch?v=UEmSsscijKE

[rpiBootGuide]: https://warmestrobot.com/blog/2024/06/27/raspberry-pi-network-boot-guide-2/
[rpiSSDHAInstall]: https://www.martinrowan.co.uk/2024/02/installing-home-assistant-on-raspberry-pi-5-nvme-storage/

[haFileBrowser]: https://community.home-assistant.io/t/home-assistant-addon-filebrowser/282108
[haTerminal]: https://github.com/hassio-addons/addon-ssh
[haUnknownStates]: https://community.home-assistant.io/t/how-to-handle-unknown-or-unavailable-states-in-template-entities-or-automations/411632/15
