This information is provided for the public domain. Do with it as you please. Linked Stack Exchange posts have whatever license is in effect, which has nothing to do with me.

Intro
-----
Sometimes Android apps, like [Simple Gallery](https://play.google.com/store/apps/details?id=com.simplemobiletools.gallery) will hide files without actually deleting them. In the particular case that motivated this document, the Recycle Bin folder in simple gallery disappeared. The photos in question were still in memory, but they were no longer accessible via normal means through the Android UI or MTP via USB. This method is likely to work for other similar cases, where files have been made inaccesible due to a bug or idiosyncracy on the Android side, without actually being deleted. Hopefully this saves some future reader (probably myself) some frustration.

Disclaimer
----------
I have no idea how most of the tools in this pipeline really work. The commands here are derived from Stack Overflow and poking around until I got what I wanted, not an exhaustive perusal of documentation. Please forgive any non-optimal steps, bad flag selections, or just plain bad advice. Or correct them.

Tools
-----
This process, like anything with your phone beyond simple file transfer, requires Android SDK. On Arch Linux, install [`android-tools`](https://archlinux.org/packages/community/x86_64/android-tools/) via `pacman`, `yay`, or whatever.

Enable Developer Mode
---------------------
This part is done on the phone

1. Main menu
2. Settings
3. About Phone (last item in Settings)
4. Build Number
5. Keep tapping on Build Number until the password entry shows up
6. Enter the password

You are now a developer! To access developer options:

1. Main Menu
2. Settings
3. System
4. Advanced
5. Developer Options

Scroll down to the Debugging Section

1. Enable USB Debugging

Create Backup
-------------
1. Connect the phone to the PC via USB.
2. On the PC, run

   ```
   adb devices
   ```
3. The phone will ask if you accept the PC fingerprint. Select "Always"
4. On the PC, run

   ```
   adb backup -apk -shared -all -f backup.ab
   ```
5. On the phone, authorize the backup
6. Wait, don't jiggle the USB cable

Extract Backup
--------------
The file `backup.ab` is a slightly modified gzipped tarfile. To make it into a not, slightly modified one, use https://stackoverflow.com/a/46500482/2988730. The first 16 bytes need to be deleted, and the next 8 need to be replaced with `1F 8B 08 00 00 00 00 00`. I used GHex, but you can just as easily run

```
( printf "\x1f\x8b\x08\x00\x00\x00\x00\x00" ; tail -c +25 backup.ab ) |  tar xfvz -
```

This will result in an "unrecoverable" error from tar caused by a premature EOF reported by GZ. You can ignore those, the output is fine. Now you have a folder with all the dumped bit in it.

For the Simple Gallery example, the path of interest was

```
apps/com.simplemobiletools.gallery/f/storage/emulated/0/DCIM/Camera
```

This contained 122 images and videos, while `shared/0/DCIM/Camera` contained a subset of only 55.

Other Things
------------
I also tried `adb pull /dev/block/mmcblk0 mmcblk0.img` in recovery mode a number of times, but got a permission denied error every time. I expect that this would allow me to make a complete image that I could pass through PhotoRec or similar, which would allow for recovery of more severe issues. The avenue I pursued was to delete `~/.android` on the PC side, revoke debugging permissions in the phone's Developer Options, and eventually add the PC's fingerprint permanently to the phone. See the comments under https://stackoverflow.com/a/41422730/2988730.

To enter recovery mode, simply boot the phone by holding the Power and Volume Up buttons at the same time. Some phones may require Power, Volume Up and Home all together.
