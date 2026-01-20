# Fixing LVM Issues on CentOS After BIOS Updates

## Problem Overview
After installing BIOS updates, CentOS failed to boot. The system was stuck at the booting stage, 
and older kernel versions also encountered the same issue. 
Even the rescue kernel image failed to boot.

## Issue Diagnosis

1. **Enter GRUB Edit Mode**
   - Press `e` during boot to access GRUB edit mode.

2. **Update GRUB Parameters**
   - Modify the `GRUB_CMDLINE_LINUX` argument by removing `rhgb quiet` in `/etc/default/grub` to print boot logs.
   - Save changes and rebooted using `CTRL + x`.

3. **Observe Boot Logs**
   - Check for the following kernel logs:
     ```
     [TIME] Timed out waiting for device /dev/mapper/cs-home.
     [DEPEND] Dependency failed for /home.
     [DEPEND] Dependency failed for Local File System.
     ```
4. **Create a CentOS Image**
   - Boot from a CentOS image on a USB drive to access the troubleshooting menu.

5. **Enable and Unlock Root User**
   - Run the commands:
     ```bash
     sudo passwd root
     sudo passwd -u root
     ```
   - Reboot to the failing kernels, which will allow access to maintenance mode.
   
6. **Check Journal Logs**
   - Use `journalctl -xb` and find the following error:
     ```
     lvm[813]: /dev/nvme0n1p6 excluded: device is not in devices file.
     ```

7. **Verify /home Mount Point**
   - Confirm the entry for `/home` exists in `/etc/fstab`.
   - Check mounted filesystems with:
     ```bash
     df -lHT
     ```
   - Make sure that `/home` is not mounted.

8. **Check Volume Groups**
   - Use `vgs` to find that there are no entries for the `cs` volume group.
   - Check if there are no entries for `/dev/nvme0n1p6` with `lvmdevices`.

## Solution Steps

1. **Add Device to LVM Devices List**
   - Add `/dev/nvme0n1p6` to the devices list using:
     ```bash
     lvmdevices --adddev /dev/nvme0n1p6
     ```

2. **Scan Volume Groups**
    - Run `vgscan` and confirm that `cs` is now recognized.

3. **Activate the Volume Group**
    - Activate the `cs` volume group with:
      ```bash
      vgchange -ay cs
      ```

4. **Mount All File Systems**
    - Mount all mount points, including `/dev/mapper/cs-home` to `/home` using:
      ```bash
      mount -a
      ```

5. **Final Reboot**
    - Try to reboot and log into one of the existing CentOS kernels.

## Conclusion
The LVM issue can be resolved by ensuring the volume group is recognized and activated, allowing the `/home` mount point to function correctly again.

