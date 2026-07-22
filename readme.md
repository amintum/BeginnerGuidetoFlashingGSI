## Comprehensive A-to-Z Beginner's Guide to Flashing a GSI (Generic System Image)
Flashing a GSI is a fantastic way to breathe new life into your Android device with the latest OS versions. This guide breaks down every single step—from checking compatibility to your first boot—ensuring you don't miss a thing.

### Phase 1: Pre-Requisites & Compatibility Check
Before touching any cables or commands, make sure your device is compatible and you have everything prepared.
 * **Backup Your Data:** Flashing a GSI requires a factory reset, and unlocking your bootloader **wipes all user data** automatically. Back up your photos, contacts, messages, and files to cloud storage or a PC.
 * **Charge Your Phone:** Ensure your battery is at least **60% charged** to prevent unexpected shutdowns during the flashing process.
 * **Check Compatibility Using "Treble Info":**
   * Download and install the **Treble Info** app (available on F-Droid or GitHub) on your phone.
   * Open the app to check your exact architecture specifications so you know which GSI file to download:
     * **Architecture:** Almost all modern smartphones use **ARM64**. Download the ARM64 variant.
     * **Seamless Updates (A/B) / Partition Style:** If it says **Yes / A/B** (or Virtual A/B), download the **AB** variant (e.g., ARM64-AB). If it says **No**, download the **A-only** variant. *(Most devices launched with Android 10 or higher use AB).*
     * **Dynamic Partitions:** If this says **Yes**, your phone uses logical partitions (which means you may encounter the "not enough space to resize partition" error later, covered in Phase 4).
 * **Choose Your GSI and Download it.**
 * **Get Platform Tools (ADB & Fastboot):**
   * Download the official SDK Platform-Tools from the Android developer website for your operating system (Windows or Linux).
   * Extract the downloaded ZIP file into an easily accessible folder (e.g., C:\platform-tools on Windows or your Home directory on Linux).
 * **Install Fastboot Drivers (Windows Only):**
   * Ensure your PC recognizes your phone in fastboot mode. If Windows shows a yellow exclamation mark in Device Manager, install Google USB Drivers or a universal fastboot driver installer.
 * **Unlock Your Bootloader:**
   * Your device **must** have an unlocked bootloader. This varies by manufacturer and usually requires enabling **OEM Unlocking** and **USB Debugging** in Developer Options, then running an OEM-specific unlock command via fastboot.
 * **Download Your Files:**
   * **GSI Image:** Download the correct gsi matching your Treble Info results (e.g., ARM64-AB ). Extract the .img file if compressed.
   * **Vbmeta Image:** You have two ways to get this:
     * *Option A (Safest/Recommended):* Extract vbmeta.img directly from the **exact official stock ROM/fastboot firmware package** designed for your specific phone model, matching your current Android version and security patch level.
     * *Option B (Generic Alternative):* If you cannot find your stock firmware, you can download a generic unsigned/keyed vbmeta.img provided by Google's release signing keys or extract a generic one from a developer community build for your device brand. *(Warning: Always prefer your exact stock ROM's vbmeta to avoid bootloops or verification errors).*

### Phase 2: Setting Up the Workspace
 1. Move your downloaded **GSI image** (rename it to something simple like system.img to make typing easier) and your **vbmeta.img** directly into your **platform-tools** folder where adb and fastboot executables live.
 2. On your phone:
   * Go to **Settings > About Phone** and tap **Build Number** 7 times until developer options are unlocked.
   * Go back, open **Developer Options**, and enable **USB Debugging**.
 3. Plug your phone into your computer using a reliable USB cable.

### Phase 3: Executing the Commands
 1. Open your terminal or Command Prompt (cmd) inside the folder where your platform-tools, GSI, and vbmeta are located.
   * *Windows:* Type cmd in the folder path bar and hit Enter.
   * *Linux:* Open terminal and use the cd command to navigate to the folder (e.g., cd /path/to/platform-tools).
 2. Verify the connection by typing:
   ```text
   adb devices
   ```
   * *Note:* Look at your phone screen; a prompt will appear asking to **Allow USB debugging**. Check "Always allow from this computer" and tap **Allow**. Run adb devices again until it shows your device's serial number followed by device.
 3. Reboot your phone into the bootloader/fastboot mode:
   ```text
   adb reboot bootloader
   ```
   * Wait for the phone screen to change to the fastboot mode screen.
 4. Disable Android Verified Boot (AVB) by flashing the vbmeta image:
   ```text
   fastboot flash vbmeta vbmeta.img --disable-verity --disable-verification
   ```
 5. Enter fastbootd mode (userspace fastboot required for modern dynamic partition devices):
   ```text
   fastboot reboot fastboot
   ```
   * Your phone screen should change to a different fastboot menu (often showing text like fastbootd).

### Phase 4: Flashing the GSI and Handling Space Errors
 1. Erase the existing system partition to clear out old data:
   ```text
   fastboot erase system
   ```
 2. Flash your GSI image file:
   ```text
   fastboot flash system system.img
   ```
   *(Replace system.img with your actual file name if you didn't rename it).*

> [!IMPORTANT]
> **Dealing with "Not Enough Space" Errors:**
> If the command above fails and throws the error: Error: not enough space to resize partition, your device's logical partition layout is too tight for the new GSI. Free up space by deleting the product logical partitions using these commands:
> ```text
> fastboot delete-logical-partition product_a
> fastboot delete-logical-partition product_b
> ```
> If the error still persists, delete the system_ext partitions as well:
> ```text
> fastboot delete-logical-partition system_ext_a
> fastboot delete-logical-partition system_ext_b
> ```
> After running these deletion commands, re-run the flash command: fastboot flash system system.img.

### Phase 5: Post-Flash Wipe & First Boot
 1. Once the system image finishes flashing successfully, reboot your device directly into recovery mode:
   ```text
   fastboot reboot recovery
   ```
 2. Once your phone boots into recovery mode (Stock Recovery or TWRP/OrangeFox):
   * Navigate using your volume keys and select **Wipe data/factory reset** (or format data).
   * Confirm the action to clear cache and user data. This step is crucial to prevent boot loops caused by leftover data from your previous ROM.
 3. After the wipe is complete, select the option to **Reboot system now**.

### Phase 6: First Boot Success
 * Your phone will now boot up. **The first boot can take anywhere from 5 to 15 minutes** because Android is optimizing apps and building internal caches for the new architecture.
 * Do not panic if you see the boot logo for a while. Once it loads, complete the standard Android setup wizard, and enjoy your new GSI!
