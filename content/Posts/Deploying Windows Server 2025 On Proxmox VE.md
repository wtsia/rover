Deploying Windows Server 2025 on Proxmox VE 9.x is a straightforward process, but if you skip the VirtIO storage drivers, the Windows installer will completely fail to detect your virtual hard drive.

### Phase 1: Preparation

Before you start, you need two files on your Proxmox server:

1. **The Windows Server 2025 ISO** (downloaded from the Microsoft Evaluation Center or your Volume License portal).
    
2. **The VirtIO Drivers ISO** (downloaded from the official Fedora/Proxmox repositories). Windows does not natively include the drivers to see Proxmox's high-performance virtual hardware.
    
    Upload both of these ISOs to your Proxmox storage (usually under `local` -> ISO Images).
    

### Phase 2: Building the VM Shell

In your Proxmox web GUI, right-click your node and select **Create VM**. Fill out the wizard exactly like this:

- **OS Tab:** Select your Windows Server 2025 ISO. Make sure the Guest OS Type is set to "Microsoft Windows" and Version is "11/2022/2025".
    
- **System Tab (Critical for 2025):** * **Machine:** `q35` (the modern chipset).
    
    - **BIOS:** `OVMF (UEFI)` (Required for modern Windows security).
        
    - **Add EFI Disk:** Check the box and select your storage.
        
    - **Add TPM:** Check the box, select your storage, and ensure version is 2.0.
        
    - **QEMU Agent:** Check this box (this allows Proxmox to cleanly shut down the VM later).
        
- **Disks Tab:** * **Bus/Device:** `VirtIO Block` or `VirtIO SCSI`. (Do not use IDE or SATA; VirtIO is significantly faster).
    
    - **Cache:** `Write back` (Best performance).
        
    - **Discard:** Checked (Allows TRIM to keep your SSDs healthy).
        
- **CPU Tab:** * **Cores:** Give it at least 4.
    
    - **Type:** `host` (This passes your physical CPU's exact architecture and features through to the VM, unlocking maximum performance).
        
- **Memory Tab:** At least `4096` MB. (If this is going to be your SCCM or SQL server later, you'll want `8192` MB or more).
    
- **Network Tab:** Change the Model to `VirtIO (paravirtualized)`.
    

Click **Finish** to create the VM, but **do not start it yet.**

### Phase 3: The Double CD-ROM Trick

We need to give the Windows installer access to those VirtIO drivers you downloaded earlier.

1. Select your new VM in the left menu and go to the **Hardware** tab.
    
2. Click **Add** -> **CD/DVD Drive**.
    
3. Select the **VirtIO Drivers ISO** and click Add.
    
    _(You now have two virtual CD drives attached to the VM: one with the Windows OS, and one with the drivers)._
    

### Phase 4: The Installation & The "Missing Disk"

1. Start the VM and immediately open the **Console**. Press any key to boot from the CD when prompted.
    
2. Proceed through the Windows Server 2025 setup until you reach the screen asking: _"Where do you want to install Windows?"_ The list will be completely blank.
    
3. Click the **Load driver** button, then click **Browse**.
    
4. Navigate to your second CD drive (the VirtIO disk).
    
5. Drill down to this specific folder: `vioscsi` -> `2k25` (or `2k22` if 2025 isn't in your specific driver build yet) -> `amd64`.
    
6. Click OK. It will find the "Red Hat VirtIO SCSI pass-through controller." Click **Next** to install it.
    
7. Your virtual hard drive will magically appear! Select it, click Next, and let Windows finish installing.
    

### Phase 5: Post-Install Cleanup

Once you are looking at the Windows Server desktop, the network icon in the bottom right will likely show no internet because the network card also needs a driver.

1. Open File Explorer and open the VirtIO CD Drive.
    
2. Double-click **`virtio-win-guest-tools.exe`**.
    
3. Next-next-finish through the wizard. This installs the network drivers, memory ballooning drivers, and the QEMU Guest Agent all at once. Your internet will instantly connect.
    
4. Shut down the Windows VM.
    
5. In the Proxmox **Hardware** tab, select that second CD/DVD drive (the VirtIO one) and click **Remove** to clean up the hardware profile.
    

---

Since you are building out an entire enterprise lab (Domain Controllers, SQL, SCCM), you will likely need to deploy Windows Server multiple times. Would you like me to walk through how to `Sysprep` this fresh VM and convert it into a reusable Proxmox Template so your future Windows deployments only take 30 seconds?