# AP configuration discovery (test feature)

This feature targets an OpenWrt router controlling access points over SSH, including 802.11s mesh-only backhaul nodes.

## Workflow

1. Add each AP under **Devices** with its actual management address and SSH credentials.
2. Open the device actions menu and select **Read Wi-Fi / mesh configuration**.
3. Inspect radios (band, channel, channel width), AP interfaces and protected 802.11s interfaces.
4. Select **Import client Wi-Fi to controller**. This creates a group bound only to the source AP and never changes the AP.
5. Edit imported **Wi-Fi** sections in the controller. The imported group has **Delete all** disabled.
6. Only when needed, use **Send** on the imported group. This is a separate, potentially disruptive action.

The read operation retrieves AP Wi-Fi passwords, solely when explicitly requested, so that encryption credentials can be preserved in editable controller entries. **Mesh passwords are never retrieved.** Controller configuration must be accessible only to trusted administrators.

On imported AP entries, `source_host`, `source_section`, `source_ssid`, `source_network`, and `source_radio` are saved so writes are restricted to the original AP and validated against the previously imported interface identity. If its state changes externally, the update is refused; re-read the AP before proceeding.

## What is not changed

- Existing 802.11s mesh interface options, mesh keys, backhaul topology, country/channel/HT mode on the radios
- Other AP interfaces outside the imported entry
- Installed `usteer` settings, unless explicitly activated in the AP group options

Mesh interfaces are read-only. AP client SSIDs remain editable after import.

## Limitations and recovery

**Wi-Fi reload can still interrupt an AP's 802.11s backhaul temporarily.** This experimental branch does not implement a connectivity watchdog or automatic rollback. Test first on a wired AP; keep alternate recovery access to the mesh gateway.

The controller's stock `OpenWrt` host placeholder is resolved from the controller's LAN IP by its polling script. Manually configured remote AP addresses are not automatically changed, since a reliable AP identity (MAC/hostname) is needed to resolve their current DHCP address.

Import currently supports AP interfaces with a standard single logical network and a radio whose 2g/5g/6g band can be identified. Unsupported interfaces are skipped rather than changed. Reimport skips existing source sections to avoid overwriting unsaved edits.

The config read agent uses `/usr/share/libubox/jshn.sh` and OpenWrt UCI. This feature has not been validated on the user's physical Google AP devices yet.
