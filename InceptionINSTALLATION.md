# Inception - Debian VM Installation Guide

---

## Step 1: Download Debian ISO

Go to your search engine and type **debian.org**.  
A page will show up with a **Download** button — click it.

**The subject says that you should use the previous version of debian or alpine and not the current one but be careful debian 12 is lagging when u try to do it with gui**

---

## Step 2: Set Up VirtualBox

Open **Oracle VirtualBox** and click **New**. A page will show up:

- **Name:** `inception` (or anything you want)
- **VM Folder:** By default it will show something like `/home/yourusername/VirtualBox VMs`.  
  Since you do not have space in your workstation, delete `/home/yourusername/VirtualBox VMs` and type:
  ```
  /goinfre/yourusername/
  ```
- **ISO Image:** Select the Debian ISO you downloaded
- **Uncheck** "Proceed with Unattended Installation"

---

## Step 3: Hardware Settings

- **Base Memory:** `4096 MB`
- **Number of CPUs:** `2`
- **Disk Size:** `20 GB`

Click **Finish** and run the VM.

---

## Step 4: Graphical Installation

Choose **Graphical Installation** for this project.

Go through the following steps:
- Choose your **language**
- Choose your **location**
- Choose your **keyboard layout**
- **Hostname:** whatever you want
- **Domain name:** just click Continue (leave it empty)
- Set a **root password**
- **Full name:** put your 42 username
- **Username:** it will already be filled with the full name you chose — just continue
- Set your **user password**

---

## Step 5: Disk Partitioning

A page will appear asking how to partition the disk. Choose:

1. **Guided - use entire disk**
2. **All files in one partition**
3. **Finish partitioning and write changes to disk** → confirm **Yes**

> No encryption needed — skip the encrypted LVM option entirely.

---

## Step 6: Extra Installation Media

A page will pop up asking about extra installation media — click **No**.

---

## Step 7: Mirror Country

A mirror is just a download server for packages — pick your country or a nearby country for faster speeds. It does not affect the project at all.

- Select your mirror country
- Choose **deb.debian.org**
- Proxy information page will show up — just click **Continue**

---

## Step 8: Package Survey

A page will ask: *"Participate in the package usage survey?"*  
Click **No** — this is just an anonymous statistics survey, irrelevant to the project.

---

## Step 9: Software Selection ⚠️ Important

This is the most important step of the installation.

Since the project requires you to create a web infrastructure through Dockerfiles, you will need a GUI to verify your website in a browser.

Check **only** the following:

| Option | Select |
|--------|--------|
| Debian desktop environment | ✅ |
| GNOME **or** Xfce | ✅ |
| SSH server | ✅ |
| standard system utilities | ✅ |

> **GNOME vs Xfce:**  
> - **Xfce** uses less RAM but is more minimal  
> - **GNOME** uses more RAM but looks and feels similar to Fedora Linux — recommended if you come from Fedora  
> - Since we allocated 4096 MB of RAM, either choice works fine

---

## Step 10: GRUB Boot Loader

GRUB is the bootloader that starts Debian when the VM powers on.

- Select **Yes** to install GRUB
- Select **/dev/sda** as the device
- Click **Continue**

---

## Step 11: Finish

The installation is complete. Click **Reboot** and your Debian VM is ready.

---

*This guide was written as part of the 42 curriculum Inception project by P3T4G0GU.*
