Boot Issues
===========

Problem: System does not boot after installation
-----------------------------------------------

**Symptoms:**

- System shows "No bootable device found"
- GRUB screen does not appear
- Boots into a black screen or BIOS menu

**Cause:**

This typically happens when the bootloader (GRUB) was not installed properly, or if UEFI/BIOS settings are misconfigured.

**Solutions:**

1. Boot into the Live ISO.
2. Open a terminal and mount your root partition:

   .. code-block:: bash

      sudo mount /dev/sda2 /mnt
      sudo mount --bind /dev /mnt/dev
      sudo mount --bind /proc /mnt/proc
      sudo mount --bind /sys /mnt/sys
      sudo chroot /mnt

3. Reinstall GRUB:

   .. code-block:: bash

      grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=LupusOS
      grub-mkconfig -o /boot/grub/grub.cfg

4. Exit chroot and reboot:

   .. code-block:: bash

      exit
      sudo reboot

**Note:**

- Make sure "Secure Boot" is disabled in BIOS.
- If using legacy BIOS mode, use `--target=i386-pc` instead of `x86_64-efi`.

**Related Logs:**

- `/boot/grub/grub.cfg`
- `/var/log/syslog`
- `efibootmgr -v` output

