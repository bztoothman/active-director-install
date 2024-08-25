# Setting up Active Directory in a Virtual Machine (VM)

This guide will walk you through setting up Active Directory in a Virtual Machine using either VirtualBox or Hyper-V Manager. If you’re using VirtualBox, ensure you download and install the VirtualBox Extension Pack. This guide uses Hyper-V.

## Prerequisites

- **Oracle VirtualBox (VM):** [Download Link](https://www.virtualbox.org/wiki/Downloads)
- **Server 2019 ISO (Domain Controller):** [Download Link](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2019)
- **Windows 10 ISO (Client):** [Download Link](https://www.microsoft.com/en-us/software-download/windows10)

## Setting Up the Network on Hyper-V

1. **Create a New Virtual Switch:**
   - Go to **Action** -> **Virtual Switch Manager** -> **Virtual Switches** -> **New virtual network switch** -> **Internal** -> **Create Virtual Switch**.
   - Rename the switch to Internal Network
   - The internal network allows the Domain Controller and the Client to communicate.

## Creating a New VM in Hyper-V (Domain Controller)

1. **Create the VM:**
   - Go to **Action** -> **New** -> **Virtual Machine**.
   - Rename the VM to **DC-1** (Domain Controller).
   - **Specify Generation** -> Generation 2.
   - **Assign Memory** -> Allocate at least 2GB (2048MB) of RAM, though 4GB (4096MB) is recommended.
   - For the network, select **Default Switch**. You will add the internal network later.
   - **Create a virtual hard disk** -> Allocate at least 35GB of storage space. -> Choose a location.
   - Choose **Install an operating system from a bootable CD/DVD-ROM** -> **Image file (.iso)** -> Browse to the Windows Server 2019 ISO.
   - **Summary** -> Finish.

2. **Adjust VM Settings:**
   - Before starting the Windows Server installation, go to **VM settings** -> Set processors to 2.
   - Add a new network adapter: **Add Hardware** -> **Network Adapter** -> **Virtual switch** from **Not connected** to **Internal Network** Apply and OK.

3. **Install Windows Server 2019:**
   - Right-click **DC-1** -> **Connect** -> **Start**.
   - Proceed with the installation by selecting -> **Next** -> **Install now** -> **Windows Server 2019 Standard Evaluation (Desktop Experience)**.
   - Accept the license terms -> **Custom: Install Windows only (advanced)** -> Select **Drive 0 Unallocated Space** -> Next.
   - After the installation is complete, set the password for the built-in administrator account to 'Password1'.

## Configuring the Network

1. **Change Network Names:**
   - Right-click the network icon on the bottom right -> **Open Network & Internet settings** -> **Change adapter options**.
   - Right-click **Ethernet** with 'Unidentified network' (Internal network) -> **Properties** -> Highlight **Internet Protocol Version 4 (TCP/IPv4)** -> **Properties**.
   - Under 'Use the following IP address' Set IP address: `172.16.0.1`, Subnet Mask: `255.255.255.0`, Gateway: `<empty>`, Preferred DNS: `127.0.0.1`.
   - Rename this network to `X_Internal_X`, and rename the other to `_INTERNET_`.

2. **Rename the Windows Server:**
   - Right-click the Windows icon -> **System** -> **Rename this PC** -> Rename to `DC-1` -> **Restart now**.

## Setting Up Active Directory

1. **Add Roles and Features:**
   - After reboot, you should see **Server Manager**.
   - Click **Add roles and features** -> Click **Next** until the **Select server roles** page.
   - Check **Active Directory Domain Services** -> **Add Features** -> **Next** until the confirmation page -> **Install**.

2. **Promote Server to Domain Controller:**
   - After installation, in **Server Manager**, click the yellow flag in the top right -> Under **Post-deployment Configuration**, click **Promote this server to a domain controller**.
   - Select **Add a new forest** -> Set the root domain name to `mydomain.com`-> **Next** -> Use Password: `Password1` -> **Next** -> Uncheck **Create DNS delegation'** -> **Next** until the prerequisites check -> **Install**.
   - The VM will automatically restart.

3. **Create Domain Admin Account:**
   - After reboot, login with `Administrator`, Password: `Password1`.
   - Go to **Start** -> **Windows Administrative Tools** -> **Active Directory Users and Computers**.
   - Create an Organizational Unit (OU): Right-click `mydomain.com` -> **New** -> **Organizational Unit** -> Name it `_ADMINS` -> Uncheck **Protect container from accidental deletion**.
   - Right-click `_ADMINS` -> **New** -> **User** -> Create a user.
     - **First name:** Brandon
     - **Last name:** Toothman
     - **Full name:** Brandon Toothman
     - **User logon name:** a-btoothman
   - Use `Password1` -> Uncheck **User must change password at next login** -> Check **Password never expires**.
   - Make the new user a domain admin: Right-click the user -> **Properties** -> **Member Of** -> **Add** -> Add **Domain Admins** -> **Check Names** -> OK -> Apply and OK.

4. **Sign Out and Log In with New Admin Account:**
   - **Sign Out of Administrator Account:**
     - Click on the **Start** menu.
     - Click on your **profile icon** (typically at the top of the Start menu).
     - Select **Sign out** from the dropdown menu.
   - **Log In with New Admin Account:**
     - On the login screen, select the newly created admin account (e.g., `a-btoothman`).
     - Enter the password (`Password1`) and log in.
     - Verify that the account has the correct permissions by accessing **Active Directory Users and Computers** and confirming the user has Domain Admin rights.

5. **Install Remote Access:**
   - Go to **Add roles and features** -> **Install Remote Access** -> **DirectAccess and VPN (RAS)** and **Routing** -> **Next** until you can install.
   - After installation, go to **Server Manager** -> **Tools** -> **Routing and Remote Access** -> Right-click **DC-1 (local)** -> **Configure and Enable Routing and Remote Access** -> Select **Network address translation (NAT)** -> Use the **_INTERNET_** interface.

6. **Set Up DHCP:**
   - Go to **Server Manager** -> **Add roles and features** -> Install **DHCP Server** -> **Add Features** -> Next until DHCP is installed.
   - Go to **Server Manager** -> **Tools** -> **DHCP** -> Under `dc-1.mydomain.com` -> Right-click **IPv4** -> **New Scope** -> Name it `172.16.0.100-200` (Range of IPs DHCP will distribute).
   - Configure the DHCP server: Set Start IP `172.16.0.100`, End IP `172.16.0.200`, Subnet mask `255.255.255.0`
   - Router (Default Gateway) `172.16.0.1` -> **Add** -> **Next** until til **Finish**.
   - Right-click `dc-1.mydomain.com` and **Authorize** -> Refresh.

## Setting Up the Client PC (Windows 10)

1. **Create the VM:**
   - Go to **Action** -> **New** -> **Virtual Machine**.
   - Name it **Client1** -> Choose a location -> Allocate at least 35GB of storage space.

2. **Configure Hardware:**
   - **Generation 1** -> Allocate at least 2GB (2048MB) of RAM, though 4GB (4096MB) is recommended.
   - Select **Internal Network** for the network.
   - Install Windows from a bootable CD/DVD-ROM -> Choose **Image file (.iso)** -> Browse to the Windows 10 ISO.

3. **Install Windows 10 Pro:**
   - Right-click **Client1** -> **Connect** -> **Start**.
   - Proceed with the installation by selecting **Windows 10 Pro**.
   - Accept the license terms -> **Custom: Install Windows only (advanced)** -> Select **Drive 0 Unallocated Space** -> Next.
  
4. **Configure Windows 10 Pro Initial Setup:**
   - On the "How would you like to set up?" screen, select **Set up for personal use**.
   - On the "Let's add your account" screen, choose **Offline account**.
   - On the "Sign in to enjoy the full range of Microsoft apps and services" screen, select **Limited experience**.
   - Name the Client1 VM **Client1**.
   - Skip the password setup by hitting **Next**.
   - On the "Always have access to your recent browsing data" screen, select **Not now**.
   - Under Choose privacy settings for your device, turn all settings from **Yes** to **No**.
   - Click **Skip** under "Let's customize your experience."
   - Click **Not now** under "Let Cortana help you get things done."
   - Windows will now start setting up until you land on the Windows desktop.

5. **Join the Domain:**
   - After installation, Right-click the Windows icon -> **System** -> **Rename this PC (advanced)** -> **Change** -> Set **Computer name** to `CLIENT1`.
   - Join the domain: Select **Domain** -> Enter `mydomain.com`.
   - Log in with your domain admin account: Username `a-btoothman`, Password `Password1`.
   - When prompted with "Do you want to allow your PC to be discoverable by other PCs and devices on this network?" select **Yes**.

6. **Complete Setup:**
   - Once the Client1 VM reboots, you can log in with your domain account.
