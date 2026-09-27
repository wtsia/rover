Setting up an "all-in-one" server combining Active Directory, DNS, and DHCP is the quintessential way to build a highly functional, resource-efficient lab environment.

To make this work flawlessly, the order of operations is critical. Active Directory relies heavily on DNS, and DHCP needs Active Directory to be authorized.

Here is the exact blueprint to configure your VM (using `192.168.1.152` and `tree.local`) as the central nervous system of your isolated lab.

### **Phase 1: The Foundation (Static IP & DNS)**

A Domain Controller cannot have a dynamically assigned IP address. You must lock down its network adapter first.

1. Open **Network Connections** (`ncpa.cpl`) on the VM.
2. Open the IPv4 Properties of your Ethernet adapter.
3. Configure the following settings specifically for an isolated lab:
* **IP Address:** `192.168.1.152`
* **Subnet Mask:** `255.255.255.0`
* **Default Gateway:** *[Leave completely blank]*
* **Preferred DNS Server:** `127.0.0.1` *(This loopback address tells the server to look at itself for DNS resolution).*


4. Click **OK** twice to apply.

---

### **Phase 2: Install the Roles**

You can install all three roles simultaneously. Doing this via PowerShell is significantly faster than clicking through the Server Manager wizard.

1. Open **PowerShell as Administrator**.
2. Run the following command to pull down the binaries for AD, DNS, and DHCP along with their management consoles:
```powershell
Install-WindowsFeature AD-Domain-Services, DNS, DHCP -IncludeManagementTools

```


3. Wait for the success message (it will indicate that a restart is not yet needed).

---

### **Phase 3: Promote to Domain Controller**

Promoting the server will create your `tree.local` domain and automatically configure the DNS zones to support Active Directory.

**The PowerShell Method:**
Run this command, and enter a recovery password when prompted:

```powershell
Install-ADDSForest -DomainName "tree.local" -InstallDns:$true

```

*Note: The server will automatically reboot when this completes.*

**The GUI Method (Alternative):**

1. Open **Server Manager**, click the yellow flag at the top, and select **Promote this server to a domain controller**.
2. Select **Add a new forest** and type `tree.local`.
3. Set your DSRM recovery password.
4. Click **Next** through the DNS Delegation, Paths, and Prerequisites screens.
5. Click **Install**. The server will reboot.

---

### **Phase 4: Authorize and Configure DHCP**

Now that the server is a Domain Controller, you need to configure DHCP to hand out IPs and tell client VMs to use this server for DNS. In a domain environment, a DHCP server must be "Authorized" in AD before it is allowed to hand out leases.

**Step 1: Authorize the DHCP Server**

1. Once logged back in, open **Server Manager**.
2. Click the yellow warning flag at the top and select **Complete DHCP configuration**.
3. Click **Next**, ensure your Domain Admin credentials are listed, and click **Commit** to authorize the server in Active Directory. Click **Close**.

**Step 2: Create the IPv4 Scope**

1. Open the Start menu, type **dhcpmgmt.msc**, and press Enter.
2. Expand your server name, right-click **IPv4**, and select **New Scope...**
3. **Name:** Give it a name (e.g., "Lab Subnet") and click Next.
4. **IP Range:** * Start IP: `192.168.1.100`
* End IP: `192.168.1.200`
* Length/Subnet Mask: `24` / `255.255.255.0`


5. **Exclusions:** Leave blank and click Next.
6. **Lease Duration:** Leave as the default (8 days) and click Next.
7. **DHCP Options:** Select **Yes, I want to configure these options now**.
8. **Router (Default Gateway):** *Crucial:* Leave this blank and click Next. (Since this is an offline lab, handing out a gateway will break the client network profiles).
9. **Domain Name and DNS Servers:**
* Parent domain: `tree.local`
* IP Address: Ensure `192.168.1.152` is listed. If not, add it. Click Next.


10. **WINS Servers:** Leave blank.
11. **Activate Scope:** Select **Yes**, click Next, and click Finish.

Your all-in-one server is now fully operational. When you spin up a new VM (like Windows 11) and attach it to your Private Virtual Switch, it will automatically pull an IP from the `192.168.1.100` pool, receive the `tree.local` DNS suffix, and immediately be ready to join the domain.


Here is the granular, step-by-step execution guide for building out this Active Directory environment. I have included both the standard GUI methods and the PowerShell commands, which are often faster for these specific deployment and configuration tasks.

## Step 3 – Create a New Active Directory Domain

**GUI Method:**

1. Open **Server Manager** and click **Add roles and features**.
    
2. Click **Next** until you reach Server Roles, check **Active Directory Domain Services**, click **Add Features**, and complete the wizard to install.
    
3. Once installed, click the yellow warning flag at the top of Server Manager and select **Promote this server to a domain controller**.
    
4. Select **Add a new forest**.
    
5. Enter your Root domain name (for example, `tree.local`) and click **Next**.
    
6. Enter a Directory Services Restore Mode (DSRM) password.
    
7. Click **Next** through the remaining prompts (leaving default paths for NTDS and SYSVOL) and click **Install**. The server will automatically reboot.
    

**PowerShell Method (Faster):**

Open an elevated PowerShell prompt and run these two commands sequentially, entering your desired DSRM password when prompted:

`Install-WindowsFeature AD-Domain-Services -IncludeManagementTools`

`Install-ADDSForest -DomainName "tree.local"`

## Step 4 – Create a Test AD User

**GUI Method:**

1. Open the Start menu, type **dsa.msc**, and hit Enter to open Active Directory Users and Computers (ADUC).
    
2. Expand your domain on the left pane.
    
3. Right-click the **Users** container, select **New**, and click **User**.
    
4. Enter a First Name and User logon name (e.g., `TestUser`).
    
5. Assign a password, uncheck **User must change password at next logon**, and check **Password never expires** for lab convenience.
    

**PowerShell Method:**

`New-ADUser -Name "TestUser" -GivenName "Test" -Surname "User" -SamAccountName "TestUser" -UserPrincipalName "TestUser@tree.local" -AccountPassword (ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force) -Enabled $true`

## Step 5 – Create the Windows 11 Client VM

1. Open **Hyper-V Manager**, right-click your host, select **New**, and click **Virtual Machine**.
    
2. Name the VM and choose **Generation 2** (Strictly required for Windows 11).
    
3. Assign at least **4096 MB** of Startup memory.
    
4. On the Configure Networking screen, select your dedicated internal/private virtual switch.
    
5. Create a virtual hard disk (at least 64GB).
    
6. Under Installation Options, select **Install an operating system from a bootable image file** and map your Windows 11 ISO.
    
7. Before booting, right-click the new VM, go to **Settings**, select **Security**, and check **Enable Secure Boot** and **Enable Trusted Platform Module** (TPM is required for Win 11).
    
8. Boot the VM and complete the standard Windows 11 installation.
    

## Step 6 – Join the Client to the Domain

1. Log into the local admin account on the Windows 11 VM.
    
2. Press **Win + R**, type **ncpa.cpl**, and press Enter.
    
3. Right-click the Ethernet adapter, select **Properties**, and double-click **Internet Protocol Version 4 (TCP/IPv4)**.
    
4. Set the Preferred DNS server to the IP address of your new Domain Controller (e.g., `10.1.20.5`). Leave the secondary DNS blank or set to `8.8.8.8`. Click OK.
    
5. Open an elevated PowerShell prompt. Ensure your network profile isn't locked into "Public" mode due to an Unidentified Network status, which will block the domain join ping. If needed, force it to Private:
    
    `Set-NetConnectionProfile -NetworkCategory Private`
    
6. Join the domain using PowerShell:
    
    `Add-Computer -DomainName "tree.local" -Restart`
    
7. Enter the Domain Admin credentials when prompted. The VM will reboot automatically.
    

## Step 7 – Validate Domain Login

1. When the Windows 11 VM reboots, click **Other user** at the login screen.
    
2. Sign in using your AD test user credentials. Format the username as `DOMAIN\TestUser` (e.g., `TREE\TestUser`).
    
3. Allow Windows a moment to provision the new user profile desktop.
    

## Step 8 – Create and Test Group Policy

1. On your Domain Controller, open the Start menu, type **gpmc.msc**, and press Enter.
    
2. Expand Forest > Domains > your domain.
    
3. Right-click your domain (or a specific Organizational Unit) and select **Create a GPO in this domain, and Link it here**.
    
4. Name it "Restrict Control Panel".
    
5. Right-click the new GPO and select **Edit**.
    
6. Navigate to **User Configuration > Policies > Administrative Templates > Control Panel**.
    
7. Double-click **Prohibit access to Control Panel and PC settings**, select **Enabled**, and click **OK**.
    
8. Switch back to your Windows 11 client VM.
    
9. Open Command Prompt and run:
    
    `gpupdate /force`
    
10. Attempt to open the Settings app or Control Panel on the client. It should be blocked by a restrictions prompt, confirming successful application.
    

## Step 9 – Build Secondary DC and Transfer FSMO Roles

1. Follow Step 5 to create a new VM (using Windows Server ISO instead of Win 11) and Step 6 to join it to the domain.
    
2. Log into the new server using Domain Admin credentials.
    
3. Install the AD DS role:
    
    `Install-WindowsFeature AD-Domain-Services -IncludeManagementTools`
    
4. Promote it to a secondary Domain Controller. Note the different PowerShell command for an existing domain:
    
    `Install-ADDSDomainController -DomainName "tree.local" -Credential (Get-Credential)`
    
5. Once the secondary DC reboots, transfer the five Flexible Single Master Operations (FSMO) roles. Doing this via the GUI requires opening three separate MMCs. Doing it via PowerShell is a single line. Run this on the _new_ secondary DC:
    
    `Move-ADDirectoryServerOperationMasterRole -Identity "NAME-OF-NEW-DC" -OperationMasterRole SchemaMaster, DomainNamingMaster, PDCEmulator, RIDMaster, InfrastructureMaster`
    
6. Press **Y** to confirm the prompts.
    
7. Validate the transfer by running this in Command Prompt:
    
    `netdom query fsmo`
    
8. Verify that the output lists your new secondary server's hostname for all five roles.