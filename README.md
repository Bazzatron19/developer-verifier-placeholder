# If you wanna help me

<a href="https://www.buymeacoffee.com/daboynb" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" height="41" width="174"></a>

# Download

**[Get the APK from the Releases page](https://github.com/daboynb/developer-verifier-placeholder/releases/latest)** — it is not in the file list above.

# Instructions
1) Download the APK from the [Releases page](https://github.com/daboynb/developer-verifier-placeholder/releases/latest).
2) Install it.

Enjoy!

If Google tries to replace the APK with the updated Developer Verifier app, it will fail due to a signature mismatch.

# Troubleshooting

Nearly every issue opened on this repo is the same one, in one of two shapes:

- `App not installed as package conflicts with an existing package` — installing by tapping the APK
- `INSTALL_FAILED_UPDATE_INCOMPATIBLE: Existing package com.google.android.verifier signatures do not match newer version` — installing via adb

Both mean the real Developer Verifier is still registered on your device. Android refuses to replace
a package with one signed by a different key ([AOSP](https://cs.android.com/android/platform/superproject/main/+/main:frameworks/base/core/java/android/content/pm/PackageManager.java):
*"a previously installed package of the same name has a different signature than the new package"*).

Whether this is fixable **depends on how the Verifier got onto your device**, and that differs
between vendors and Android builds. Run the triage below to find out which case you are in.

## Step 0 - Connect your device with ADB

**A. Download ADB**

Download the Android SDK Platform Tools for your operating system:

https://developer.android.com/tools/releases/platform-tools

Extract the downloaded ZIP file.

**B. Download the APK**

Download the latest APK from this repository's **Releases** page and copy it into the extracted `platform-tools` folder.

**C. Open a terminal in the `platform-tools` folder**

Open Command Prompt, PowerShell, Terminal, or your preferred command-line tool and navigate to the `platform-tools` directory.

**D. Enable Developer Options on your phone**

Go to:

`Settings > About phone > Software information`

Tap **Build number** seven times, then enter your PIN when prompted.

**E. Enable USB debugging**

Go to:

`Settings > System > Developer options`

Enable **USB debugging**.

Wireless debugging can also be used, but USB is simpler.

**F. Connect your phone**

Connect the phone to your computer using a USB cable that supports data transfer.

On the phone, open the notification tray, tap the USB connection notification, and select **File Transfer**.

**G. Check the ADB connection**

In your terminal, run:

```bash
adb devices
```

On Windows, you may need to use:

```powershell
.\adb.exe devices
```

If prompted on your phone, approve the USB debugging connection.

Your device should appear in the list with a device ID. If it does, ADB is connected and you can continue.

## Step 1 — run the triage
Now we are going to check if the "official" verifier is installed.
Run the following command:

Windows:
```
.\adb.exe pm list packages -u | findstr verifier
```
Linux/MacOS:
```
adb shell pm list packages -u | grep verifier
```
If your terminal responds with an empty list - the verifier is not installed and we don't need to remove it.
If your terminal lists a package like `package:com.google.android.verifier` then we can proceed with uninstalling it.

Windows:
```
.\adb.exe uninstall com.google.android.verifier
```

Linux/MacOS
```
adb uninstall com.google.android.verifier
```

Do **not** add `--user 0`. By default `adb uninstall` removes the package for *every* user on the
device, which is what you want — `--user 0` only touches the primary profile and is the reason many
people get stuck with `Failure [not installed for 0]`.

## Step 2 — read what it answered

**`Success`** → the Verifier was an ordinary updatable app and is now really gone. Install the placeholder:

Windows:
```
.\adb.exe install -r -d developer-verifier-v3000000000000.apk
```
Linux/MacOS:
```
adb install -r -d developer-verifier-v3000000000000.apk
```

**Anything other than `Success`, while step 1 still lists the package** → the Verifier is
preinstalled as a system component. On modern Pixel and Samsung builds (Android 16) it is baked in,
so the uninstall does not really remove it — you'll see one of:

- `Failure [not installed for 0]`
- `Failure [DELETE_FAILED_INTERNAL_ERROR]`
- the command "succeeds" but only strips *updates*, and `pm list` still shows
  `com.google.android.verifier` (often together with `com.google.android.verifier.overlay`)

All of these are the same case. Android never actually deletes a system package: it marks it
`installed=false` for the user and keeps the package record *and its original signature* in the
package database ([AOSP `DeletePackageHelper`](https://cs.android.com/android/platform/superproject/main/+/main:frameworks/base/services/core/java/com/android/server/pm/DeletePackageHelper.java)).
Any differently signed APK is rejected from then on. There is no non-root workaround — please do not
open an issue for this. On older builds (Android 13, e.g. OnePlus) the same package is usually an
ordinary app and uninstalls cleanly, so it really does vary by device.

Both outcomes have been reported on similar hardware, so do not assume from someone else's report:
run the two commands on your own device.

## Notes

| Situation | What to do |
|---|---|
| No PC available | Use [Shizuku](https://shizuku.rikka.app/) + aShell to run the same commands on-device |
| `adb shell pm install file.apk` says `Unable to open file` | Wrong command. `adb shell pm` cannot see files on your computer — use `adb install` |
| Samsung | Also uninstall it inside Knox Secure Folder (Secure Folder → Settings → Apps), and turn off Settings → Security and privacy → **Auto Blocker** |
| "Verify apps over USB" in Developer options | Turning it off does not fix a signature mismatch. Only step 2 decides the outcome |

## Changed your mind?

To bring the real Developer Verifier back:

```
adb uninstall com.google.android.verifier
adb shell cmd package install-existing com.google.android.verifier
```

## VirusTotal flags the APK

Some engines report `Android.Riskware.Repack.*`. This is a false positive: the APK contains no code
at all — just a manifest, see [`app/src/main/AndroidManifest.xml`](app/src/main/AndroidManifest.xml).
Heuristics fire because it declares a Google package name while being signed with a different key,
which is precisely the point of a placeholder. Build it yourself if you prefer.

# Changelog :

- v3000000000000: Initial release. `versionName` is `3000000000000` — it is a free-form string, so it
  can be arbitrarily large. `versionCode` stays at `2000000000`: it is a 32-bit integer, and
  [the greatest value Google Play allows is 2100000000](https://developer.android.com/studio/publish/versioning).
  The `versionCode` is what actually blocks the update.
