# Realme GT5 Deep Testing and Fastboot

This guide covers the Windows workflow described in NeFeroN’s February 2024 Boosty post. Deep Testing submits an unlock request for the Realme GT5 phone. After Realme approves the request, Fastboot sends the bootloader unlock command over USB. The workflow ends with a factory reset and an unlocked bootloader.

**Unlocking erases all user data on the phone. Back up photos, files, app data, and authenticator recovery codes before you start. Confirm you can sign in to the Google and Realme accounts you use on the phone after the reset.**

## 1. Confirm your phone and firmware

- On the phone, open **Settings → About device** and record the model and software build. The guide concerns the **Realme GT5 (RMX3820 or RMX3823)**. Use model-specific instructions if you have a GT5 Pro, GT Neo 5, or another model.
- The Boosty post was written for the **Chinese Android 13 firmware**. Realme’s approval service and app compatibility may have changed since 2024. If you have a different firmware region or Android version, confirm the supported procedure with Realme before using these APKs or changing firmware.
- You need an internet connection on the phone for the HeyTap account and approval request, plus a Windows PC and a USB cable that supports data transfer.

## 2. Identify the four files in the post

| File | Where it goes | What it does |
| --- | --- | --- |
| `GT5-DeepTesting.apk` | Realme GT5 | The post uses this APK to sign in and apply for unlock approval. Prefer a copy obtained through Realme’s support or community channels. |
| `DeepTesting_old.apk` | Realme GT5 | In the post’s 2024 workflow, the older app was installed after approval to query the status and open the bootloader. Use the approved app’s **Start deep test** button if it is available. Before substituting an older APK, confirm with Realme that the build is compatible and comes from a trusted source. |
| `usb_driver_r13-windows.zip` | Windows PC | The post’s USB driver bundle. A separate driver is needed only if Windows cannot communicate with the phone in bootloader mode. Use a signed Realme/OEM driver that matches the device if Windows does not install a working driver automatically. |
| `platform-tools_r34.0.5-windows.zip` | Windows PC | The post’s copy of Android SDK Platform-Tools, which contains `fastboot.exe`. Google’s current Windows package can be used in place of this older archive. |

The saved Boosty HTML may display the attachment names without working download buttons when opened locally. Open the original post to view its attachments. I verified the filenames in the saved post. The APK signatures, current Realme approval availability, and the process on a physical phone remain unverified.

- [Original Boosty post and its attachments](https://boosty.to/neferon/posts/d7a2fd50-da4f-4ff0-839f-d53b9373a210)
- [Realme Community](https://c.realme.com/global/)
- [Google SDK Platform-Tools for Windows](https://developer.android.com/tools/releases/platform-tools)
- [Android OEM USB driver installation](https://developer.android.com/studio/run/oem-usb)

### Check the APK source before signing in

If you already have an APK and Android SDK Build-Tools, `apksigner` can verify its signature and display the signer’s certificate fingerprint. Run this from Command Prompt with `apksigner.bat` on your PATH or invoke it from the Build-Tools directory. Replace the APK path with yours.

```bat
apksigner.bat verify --verbose --print-certs "C:\Downloads\GT5-DeepTesting.apk"
```

Compare the signer’s SHA-256 certificate fingerprint with a trusted Realme reference for that specific release. A successful signature check by itself only confirms that the APK matches its included signature. If you cannot establish the APK’s provenance, obtain a trusted copy before entering your HeyTap credentials. [Android `apksigner` documentation](https://developer.android.com/tools/apksigner)

## 3. Prepare Windows

1. Download the Windows Platform-Tools ZIP from Google. In Explorer, right-click it and choose **Extract All**. The commands below assume the folder containing `fastboot.exe` is `C:\Android\platform-tools`. Change the path if you extracted it elsewhere.
2. Open **Command Prompt** and verify that the executable starts.

   ```bat
   cd /d "C:\Android\platform-tools"
   fastboot.exe --version
   ```

3. Keep the folder intact. Its other files include dependencies used by the executables. You can do the USB driver check after the phone enters bootloader mode.

## 4. Request approval on the phone

1. While Android is running, open **Settings → About device → Version** and tap **Build number** seven times to enable Developer options. Enter your screen-lock PIN if prompted.
2. Find **Developer options** under **Additional settings** or **System settings**. Enable **OEM unlocking**. The post also calls this **Factory unlocking**.
3. Install the trusted GT5 Deep Testing APK. If Android asks, temporarily allow **Install unknown apps** for the file manager you are using. Turn that permission off after installation.
4. Open **Deep Testing**, sign in with your **HeyTap / Realme account**, and submit the application after reading the on-screen terms. The post switches the phone’s language to English so the labels are easier to follow.
5. Check the application status and wait for the app to show approval. A submitted request or displayed review date does not by itself mean approval has been granted.
6. Once approved, choose **Start deep test**. The phone should restart into the bootloader. The post’s older-APK workflow uninstalls the first Deep Testing app, installs `DeepTesting_old.apk`, and uses **Query approval status** followed by **Start deep test**. Only use an older build if Realme confirms it for your firmware and you can verify its source.

## 5. Check the Fastboot connection

Connect the phone directly to the PC while the phone is on its bootloader screen. In the same Command Prompt folder, run:

```bat
fastboot.exe devices
```

You should see one device serial with `fastboot` beside it. If the output is blank, open **Device Manager** while the phone is still in bootloader mode. Unplug and reconnect the phone to identify the entry that appears. If it has a warning icon or is listed under **Other devices**, right-click it, choose **Update driver**, and browse to the extracted folder for the signed, compatible driver. Reconnect the phone and rerun `fastboot.exe devices`. A phone that appears in Windows while Android is running may still need a separate driver in bootloader mode. [Android OEM USB driver installation](https://developer.android.com/studio/run/oem-usb)

## 6. Unlock the bootloader

After you have a backup, confirmed approval, and confirmed the correct device serial, run:

```bat
fastboot.exe flashing unlock
```

Read the confirmation screen on the phone. If you intend to proceed with the factory reset, use the volume keys to select the unlock option and press the power button to confirm. Let the phone complete the reset and restart. If it remains on the bootloader screen after Fastboot reports success, select **Start** on the phone or run `fastboot.exe reboot`.

The first startup may take longer than usual. The phone may display an unlocked-bootloader warning during startup. Complete Android setup and restore your backup after it boots. [Android bootloader locking and unlocking](https://source.android.com/docs/core/architecture/bootloader/locking_unlocking)

## Troubleshooting

| Symptom | What to check |
| --- | --- |
| Windows says `fastboot.exe` is not recognized | Run `dir fastboot.exe` in Command Prompt. Change to the extracted folder that contains the executable. In PowerShell, use `./fastboot.exe` from that folder. |
| `fastboot devices` returns nothing or the command waits for a device | Verify the phone shows its bootloader screen. Try another data cable and a direct USB port. Check Device Manager while the phone is in that mode. |
| More than one device is listed | Disconnect the other Android devices and verify the remaining serial before issuing the unlock command. |
| APK fails to install or closes immediately | Confirm model, firmware region, Android version, and APK provenance. A package or signature conflict can occur when swapping builds. Contact Realme if the applicable release is unclear. |
| OEM unlocking is unavailable or Fastboot reports `Flashing Unlock is not allowed` | Recheck the OEM unlocking toggle and the Deep Testing approval status. If the toggle stays unavailable or the request is rejected, confirm eligibility with Realme for your exact model, firmware, and region. |
| Deep Testing is stuck pending or says the model is unsupported | Confirm account sign-in, network connectivity, and the exact model/build. A USB driver or different Fastboot version cannot change Realme’s account or server approval. |
