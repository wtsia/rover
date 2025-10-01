Using Proxmox VE

Assuming your using Proxmox VE, the following steps are for setting up a VM that hosts a Sons of The Forest Server. While there are many online guides for deploying this on Linux, I opted for a simple solution of just running it in a Windows VM with Steam downloaded.

### What You'll Need
- Windows ISO
- VirtISO for windows drivers
# Overview
1. Install Windows on VM
2. Install Steam Client on the VM
3. Download 'SoTF Dedicated Server'

###
locallow server config
lanonly true


## Automating Server Startup on VM Startup
### Method 1: Using the Startup Folder (Easiest)

This folder contains shortcuts to all the applications that you want to launch automatically when you log in.

1. Press the **Windows Key + R** to open the Run dialog box.
    
2. Type `shell:startup` and press Enter. This will open the Startup folder for your user account.
    
3. Find the program you want to add. You can either find its shortcut on your desktop or find the program's main `.exe` file in its installation folder (usually in `C:\Program Files`).
    
4. **Copy** the program's shortcut or `.exe` file.
    
5. Go back to the Startup folder you opened and **Paste shortcut**.
    

The next time you log into Windows, this program will start automatically.

**Pro Tip:** If you want the program to start automatically for **every user** on the computer, use `shell:common startup` in the Run dialog instead.