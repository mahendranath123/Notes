

### Step 1: Edit the Boot Line

1. Use your arrow keys to move the cursor down to the second-to-last line visible in your image. It contains:
`/boot/vmlinuz-6.17.0-19-generic root=UUID=17ad54b3-83a4-4189-bdbe-fe93733917eb ro quie...`
2. Use the right arrow key to move your cursor all the way to the end of that line.
3. Delete the **`ro quie...`** (it likely says `ro quiet splash`).
4. Replace it with the following so the end of the line looks exactly like this:
```text
rw init=/bin/bash

```


*(Ensure there is a space between the end of the UUID and `rw`)*.

### Step 2: Boot into the Shell

1. As the prompt at the bottom of your screen indicates, press **`Ctrl` + `X**` or **`F10`** to boot with these modified settings.
2. The screen will go black for a moment, and you will be dropped into a root shell prompt that looks something like `root@(none):/#`.

### Step 3: Reset the Password

1. Once at the root prompt, reset the password for your specific user account. If you know your exact username, type:
```bash
passwd <your_username>

```


*(If you don't remember your exact username, you can type `ls /home` and hit Enter to see the user folder names, which match the usernames).*
2. You will be prompted to enter a new password and then re-type it to confirm. (The characters won't show up on screen as you type—this is normal).

### Step 4: Save and Reboot

1. Ensure the new password and file system changes are written to the disk by syncing:
```bash
sync

```


2. Reboot the system normally to return to the login screen:
```bash
exec /sbin/init

```


*(If the system hangs here, you can safely perform a hard reboot by holding down the physical power button on your machine).*

Once it boots back up, you should be able to log in with the new password.
