# Fixing LVM Issues on CentOS After BIOS Updates

## Problem Overview
After installing BIOS updates, CentOS failed to boot. The system was stuck at the booting stage, 
and older kernel versions also encountered the same issue. 
Even the rescue kernel image failed to boot.

## Issue Diagnosis

1. **Enter GRUB Edit Mode**
   - Press `e` during boot to access GRUB edit mode.
   ![grub2-menu](https://github.com/Tinsae-Tadesse/fix-lvm-issues-after-bios-updates/blob/94c77ef89cfa2d82576ff3175566f0d1257044da/resources/grub-menu.jpg)
   
2. **Update GRUB Parameters**
   - Modify the kernel parameters by removing `rhgb quiet` from the line that starts with `linux` (this enables printing detailed boot logs).
   1) 
   ![grub-kernel-parameter](https://github.com/Tinsae-Tadesse/fix-lvm-issues-after-bios-updates/blob/23344f37f85917ea4880ee3bbea796b3cb06699a/resources/grub-kernel-parameter.jpg)

   2) 
   ![kernel-rhgb-quiet-parameters](https://github.com/Tinsae-Tadesse/fix-lvm-issues-after-bios-updates/blob/dacf8ae12b82c8cd581b0c93e1479f71de5c53e9/resources/kernel-rhgb-quiet-parameters.jpg)
   - Save the changes and reboot using `CTRL + x`.

3. **Observe Boot Logs**
   - Look for messages similar to the following:
     ```
     [TIME] Timed out waiting for device /dev/mapper/cs-home.
     [DEPEND] Dependency failed for /home.
     [DEPEND] Dependency failed for Local File System.
     ```
4. **Create a CentOS Image**
   - Boot from a CentOS installation image on a USB drive and access the troubleshooting menu.

5. **Enable and Unlock the Root User**
   - Run the following commands:
     ```bash
     sudo passwd root
     sudo passwd -u root
     ```
   - Reboot into the failing kernels; this should now allow access to maintenance mode.
   
6. **Check Journal Logs**
   - Use `journalctl -xb` and look for errors similar to:
     ```
     lvm[813]: /dev/nvme0n1p6 excluded: device is not in devices file.
     ```

7. **Verify the /home Mount Point**
   - Confirm that an entry for `/home` exists in `/etc/fstab`.
   - Check mounted filesystems with:
     ```bash
     df -lHT
     ```
   - Ensure that `/home` is **not** currently mounted.

8. **Check Volume Groups**
   - Run `sudo vgs` and verify that there are no entries for the `cs` volume group.
   - Check that there are no entries for `/dev/nvme0n1p6` by running `sudo lvmdevices`.

## Solution Steps

1. **Add Device to the LVM Devices List**
   - Add `/dev/nvme0n1p6` to the devices list:
     ```bash
     sudo lvmdevices --adddev /dev/nvme0n1p6
     ```

2. **Scan Volume Groups**
    - Run `sudo vgscan` and verify that the `cs` volume group is now detected.

3. **Activate the Volume Group**
    - Activate the `cs` volume group with:
      ```bash
      sudo vgchange -ay cs
      ```

4. **Mount All File Systems**
    - Mount all mount points, including `/dev/mapper/cs-home` to `/home` using:
      ```bash
      sudo mount -a
      ```

After completing the above steps, you should be able to complete the boot process normally.

## Conclusion
The LVM issue can be resolved by ensuring the volume group is recognized and activated, allowing the `/home` mount point to function correctly again.

