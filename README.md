# VMware-Ubuntu-Setup

The following instructions will guide you through the process of installing VMware Workstation Pro on an Ubuntu system, with Secure Boot enabled. This setup is inspired by the following resources:

- (https://knowledge.broadcom.com/external/article/344595/downloading-vmware-workstation-pro.html)[https://knowledge.broadcom.com/external/article/344595/downloading-vmware-workstation-pro.html]

## Step 1: Install Required Packages

Open a terminal and run the following command to install the necessary build tools and kernel headers:

```bash
sudo apt-get install build-essential linux-headers-$(uname -r)
```

## Step 2: Download VMware Workstation Pro

You can download VMware Workstation Pro from the official Broadcom support site:

(https://support.broadcom.com/group/ecx/productdownloads?subfamily=VMware%20Workstation%20Pro&freeDownloads=true)[https://support.broadcom.com/group/ecx/productdownloads?subfamily=VMware%20Workstation%20Pro&freeDownloads=true]

## Step 3: Install VMware Workstation Pro

Install VMware Workstation Pro using the downloaded bundle file. Replace `<VERSION>` with the actual version number of the downloaded file.

```bash
sudo sh VMware-Workstation-Full-<VERSION>.bundle
sudo vmware-modconfig --console --install-all
```

## Step 4: Sign VMware Modules for Secure Boot

Generate a signing key and sign the VMware kernel modules to allow them to load with Secure Boot enabled:

```bash
openssl req -new -x509 -newkey rsa:2048 -keyout VMWARE.priv -outform DER -out VMWARE.der -nodes -days 36500 -subj "/CN=VMWARE/"
sudo /usr/src/linux-headers-$(uname -r)/scripts/sign-file sha256 ./VMWARE.priv ./VMWARE.der $(modinfo -n vmmon)
sudo /usr/src/linux-headers-$(uname -r)/scripts/sign-file sha256 ./VMWARE.priv ./VMWARE.der $(modinfo -n vmnet)
sudo mokutil --import VMWARE.der
```

## Step 5: Enroll MOK and Reboot

Reboot your system to enroll the Machine Owner Key (MOK) you just created:

```bash
sudo reboot now
```

In the UEFI menu, navigate to the "Enroll MOK" option and follow the prompts:

- Enroll MOK
- Continue
- Yes
- Enter the password you created during the `mokutil --import` step
- Reboot

## Extra: Install Open VM Tools on a VM in Workstation Pro

If you are running a virtual machine within VMware Workstation Pro, it is recommended to install Open VM Tools for better performance and integration. You can do this by running the following commands in the terminal of your Ubuntu VM:

```bash
sudo apt update
sudo apt install open-vm-tools open-vm-tools-desktop -y

sudo systemctl enable open-vm-tools
sudo systemctl start open-vm-tools
systemctl status open-vm-tools

sudo reboot now

sudo mkdir -p /mnt/hgfs
sudo vmhgfs-fuse .host:/ /mnt/hgfs -o allow_other
```
