Here is the exact procedure to reset a lost root password on CentOS 

1. **Access the GRUB Menu:**
Reboot the server. When the GRUB boot menu appears, use the up/down arrow keys immediately to stop the countdown and select the kernel you normally boot into.


2. **Edit the Boot Parameters:**
Press `e` on your keyboard to edit the selected boot entry.


3. **Append rd.break:**
Use your arrow keys to scroll down until you find the line that starts with `linux16` or `linux`. Go to the very end of this line and type a space followed by:

```bash
rd.break

```

Once added, press **Ctrl + X** to boot the system into emergency mode.


4. **Remount the Filesystem:** The filesystem mounts as read-only by default.
You will be dropped into a root shell. First, remount the system root with read and write permissions:

```bash
mount -o remount,rw /sysroot

```


5. **Access the System Environment:**
Change the root directory to your actual system environment so you can run standard commands:

```bash
chroot /sysroot

```


6. **Reset the Password:**
Type the `passwd` command to change the root password:

```bash
passwd root

```

You will be prompted to type your new password twice.


7. **Reconfigure SELinux:** Skipping this will lock you out of the system.
Because CentOS 7 uses SELinux, changing the password while the system is offline breaks the file's security context. You must force the system to relabel its files on the next boot:

```bash
touch /.autorelabel

```


8. **Reboot:**
Type `exit` twice to leave the chroot environment and reboot the system:

```bash
exit
exit

```

