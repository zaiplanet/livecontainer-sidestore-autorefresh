# LiveContainer + SideStore Auto-Refresh Shortcut (iOS 26.4)

A guide to fully automate the 7-day re-signing process for LiveContainer + SideStore on iOS 26.4 using iOS Shortcuts and Personal Automation. No more manual VPN toggling or tapping "Refresh All" every week.

## Background

On iOS 26.4, Apple changed how lockdown connections are validated, breaking SideStore's standard refresh method. The community workaround requires connecting to **two VPNs in sequence** (VPN Super via IKEv2, then LocalDevVPN) before SideStore can successfully re-sign apps.

This guide automates that entire flow into a single iOS Shortcut that runs on a schedule.

## Prerequisites

- iOS 26.4 (stable or beta)
- LiveContainer 3.7.2 (with bundled SideStore)
- LocalDevVPN installed and configured
- VPN Super installed and configured with IKEv2
- Pairing file already imported into SideStore
- JIT-Less Mode tested and working in LiveContainer

## How the Shortcut Works

The shortcut performs these steps in order:

1. Connect to VPN Super (IKEv2)
2. Reconnect VPN Super after a short delay (workaround for iOS VPN switching bug)
3. Connect to LocalDevVPN
4. Open LiveContainer (required — SideStore inside LiveContainer must be active for refresh to work)
5. Open `sidestore://` URL scheme — this routes into SideStore inside LiveContainer
6. Trigger SideStore's native "Refresh All Apps" Shortcuts action
7. Wait for re-signing to complete
8. Disconnect both VPNs

## Building the Shortcut

Open the **Shortcuts** app and create a new shortcut named `Refresh LC SideStore` with the following actions in order:

| # | Action | Configuration |
|---|--------|---------------|
| 1 | Set VPN | Connect to **VPN Super**, Status: **On** |
| 2 | Wait | **3 seconds** |
| 3 | Set VPN | Connect to **VPN Super**, Status: **On** (intentional duplicate) |
| 4 | Wait | **5 seconds** |
| 5 | Set VPN | Connect to **LocalDevVPN**, Status: **On** |
| 6 | Wait | **5 seconds** |
| 7 | Open App | **LiveContainer** |
| 8 | Wait | **4 seconds** |
| 9 | Open URLs | `sidestore://` |
| 10 | Wait | **6 seconds** |
| 11 | Refresh All Apps | SideStore native action — **disable "Show When Run"** |
| 12 | Wait | **45 seconds** |
| 13 | Set VPN | Connect to **LocalDevVPN**, Status: **Off** |
| 14 | Set VPN | Connect to **VPN Super**, Status: **Off** |

### Notes on Specific Actions

**Action 3 (duplicate VPN connect)** — On iOS, connecting to a VPN while a different VPN was previously active sometimes only switches the selected VPN without activating it. Reconnecting after a short delay forces proper activation.

**Action 7 (Open LiveContainer)** — Without opening LiveContainer first, the `sidestore://` URL scheme has nothing to route to, and SideStore's refresh action will silently fail. See [LiveContainer issue #929](https://github.com/LiveContainer/LiveContainer/issues/929).

**Action 11 (Refresh All Apps)** — This action is exposed by SideStore as a native Shortcuts action. Search "SideStore" or "Refresh" in the action picker. **Important**: tap the action to expand it and turn off **Show When Run**, otherwise the shortcut will stop and wait for user input.

**Action 12 (45 second wait)** — If your apps are not actually being refreshed, increase this to 60–90 seconds. Re-signing takes time depending on how many apps are installed.

## Setting Up the Automation

1. Open the **Shortcuts** app
2. Tap the **Automation** tab
3. Tap **+** then **Create Personal Automation**
4. Choose **Time of Day**
5. Set time to **3:00 AM**, Repeat **Weekly**, pick a day (e.g. Thursday)
6. Switch from **Run after confirmation** to **Run Immediately**
7. Tap **Next**
8. Search for and select your **Refresh LC SideStore** shortcut
9. Run the shortcut manually once from the Library tab to grant all required permissions (VPN, Open App, etc.)

## Verifying It Works

After the automation runs, manually verify the refresh succeeded:

1. Open LiveContainer
2. Tap the SideStore icon in the top-left corner
3. Go to **My Apps**
4. Confirm the expiration timer has reset to **7 DAYS**

Track this for 2–3 cycles before fully trusting the automation.

## Troubleshooting

**The "Refresh All Apps" action shows as "unavailable"** — Each SideStore instance is unique to your Apple ID. If you imported this shortcut from someone else, you must delete that action and re-add the one tied to your local SideStore install. Search "SideStore" in the action picker to find yours.

**Apps don't actually refresh despite the shortcut completing** — Most common causes:
- LiveContainer was not opened before the URL scheme was triggered (Action 7 missing or wait time too short)
- VPN Super or LocalDevVPN failed to activate (check VPN icon in status bar)
- 45-second wait is too short — increase Action 12

**VPN won't toggle from Shortcuts** — The VPN must be installed as a system VPN profile (Configuration Profile), not as an in-app NetworkExtension. LocalDevVPN, VPN Super (IKEv2), and StosVPN all qualify.

**"Run Immediately" still asks for confirmation** — Some iOS versions require Face ID for the first VPN toggle in an automation. After granting it once, subsequent runs should be silent.

## Future Simplification

SideStore has released an Alpha build that fixes the iOS 26.4 networking issue, allowing refresh to work with just LocalDevVPN (no VPN Super needed). Once the stable release ships, you can simplify the shortcut by removing Actions 1–4. Track [SideStore releases](https://github.com/SideStore/SideStore/releases) for updates.

## Credits

- [SideStore](https://github.com/SideStore/SideStore)
- [LiveContainer](https://github.com/LiveContainer/LiveContainer)
- iOS 26.4 workaround originally documented by Đỗ Hoàng Lâm in [SideStore issue #1174](https://github.com/SideStore/SideStore/issues/1174)
- VPN reconnect workaround from [Sonia Malki's gist](https://gist.github.com/SoniaMalki/2b34cfeba427a75e53659cb25fd0289d)
- Automation structure inspired by [Kitsah's guide](https://kitsah.bearblog.dev/how-to-setup-livecontainer-sidestore-with-automatic-refreshing/)

## License

This guide is provided as-is for personal use. Tools referenced have their own licenses (SideStore: AGPL-3.0, LiveContainer: see repo).
