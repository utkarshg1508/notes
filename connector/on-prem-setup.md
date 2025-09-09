# 🚀 On-Premises Connector Setup Guide

This guide walks you through setting up an on-premises connector by requesting and configuring a virtual machine.

---

## 📋 Step 1: Request a Virtual Machine

Follow these steps to request a VM through NetApp's ServiceNow portal:

### 1. 🌐 Access ServiceNow Portal
Navigate to the [NetApp Engineering ServiceNow](https://netappeng.service-now.com/sp?id=sc_category&sys_id=204a073bdb8410507adaf1fcbf9619d1&catalog_id=-1&spa=1) portal.

### 2. 📝 Create New Request
1. Go to **My Requests** → **New Request**

### 3. ⚙️ Configure VM Settings
Complete the following configuration:

| Setting | Value |
|---------|-------|
| **Zone** | Openlab |
| **Site** | RTP |
| **Template** | gec_ubuntu22.04_ol - Ubuntu 22.04 Jammy Jellyfish |
| **Flavor** | m8.Small |
| **Quantity** | 1 (default) |
| **Purpose** | *Enter your specific use case* |

### 4. 📤 Submit and Monitor Request
1. Click **Submit Request**
2. Monitor your request under **My Requests**
3. Once approved, expand the request row to view:
   - 🌐 VM IP address
   - 🔐 Access credentials (click **Show Credentials**)

---

## 🔧 Step 2: Install BlueXP Connector

### 1. 🔌 Connect to Your VM

SSH into your provisioned VM using the credentials from Step 1:

```shell
ssh root@<VM_IP_ADDRESS>
```

When prompted, enter the password obtained from **Show Credentials** in your ServiceNow request.

### 2. 💾 Increase Filesystem Size

Check current filesystem usage and expand storage volumes for the BlueXP Connector requirements:

```shell
# Check current filesystem sizes
df -h
```

#### 📈 Extend LVM Volumes

Expand the root, var, and var-log logical volumes:

```shell
# Extend logical volumes by 30GB each
lvextend -L +30G /dev/ubuntu-vg/ubuntu-lv
lvextend -L +30G /dev/ubuntu-vg/var-lv
lvextend -L +30G /dev/ubuntu-vg/var-log-lv
```

#### 🔄 Resize Filesystems

Apply the volume extensions to the ext4 filesystems:

```shell
# Resize filesystems to use the extended space
resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
resize2fs /dev/mapper/ubuntu--vg-var--lv
resize2fs /dev/mapper/ubuntu--vg-var--log--lv
```

#### ✅ Verify Changes

Confirm the filesystem expansion:

```shell
# Verify new filesystem sizes
df -h
```

### 3. 🐳 Verify Docker Installation

Check the current Docker version:

```shell
docker --version
```

> ⚠️ **Important**: If Docker version is greater than 26 or less than 23, you'll need to downgrade it. 

### 🔄 Docker Downgrade Steps

#### 1. 🗑️ Uninstall Current Docker

```shell
# Remove existing Docker packages
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do 
    sudo apt-get remove $pkg
done

# Purge Docker CE components
sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras

# Remove Docker directories
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```

#### 2. 📦 Setup Docker Repository

```shell
# Update package index and install prerequisites
sudo apt-get update
sudo apt-get install ca-certificates curl

# Create keyrings directory
sudo install -m 0755 -d /etc/apt/keyrings

# Add Docker's official GPG key
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add Docker repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update package index again
sudo apt-get update
```

#### 3. ⬇️ Install Specific Docker Version

```shell
# Check available versions
apt-cache madison docker-ce | awk '{ print $3 }'

# Install Docker 25.0.0
VERSION_STRING=5:25.0.5-1~ubuntu.22.04~jammy
sudo apt-get install docker-ce=$VERSION_STRING docker-ce-cli=$VERSION_STRING containerd.io docker-buildx-plugin docker-compose-plugin

# Prevent auto-updates
apt-mark hold docker-ce
apt-mark hold docker-ce-cli
```

#### 4. Disable Firewall

```shell
systemctl  stop ufw 
systemctl   disable ufw 
ufw stop 
```

### 4. 📥 Download BlueXP Installer

Get the latest staging installer from TeamCity:

```shell
wget <INSTALLER_LINK>
```

> 📋 **Note**: Get your installer link from the [TeamCity Build Page](https://teamcity.platform.bluexp.netapp.com/buildConfiguration/OccmServiceAgentInfrastructure_ClientDeployment_ServiceManagerV2_CreateServiceMangerV2?mode=builds#all-projects). Click the latest build number → **Artifacts** → `installer_details.txt`

### 5. 🏃‍♂️ Set Permissions and Run Installer

Make the installer executable and run it:

```shell
# Make installer executable
chmod 777 <INSTALLER_FILE>

# Run the installer
./<INSTALLER_FILE>
```

**Example**:
```shell
chmod 777 sm2_installer_3.0_1983_staging
./sm2_installer_3.0_1983_staging
```

### 6. Deploy the Agent

1. Go to that the vm's IP in the browser
2. Choose the Connector name and the organization where you want to deploy the connector.


## 📚 Additional Resources

- 📖 [Confluence Documentation](https://confluence.ngage.netapp.com/display/UMF/Setting+VM+for+Connector%2C+Restricted+site+and+Dark+site)
- 🏗️ [TeamCity Build System](https://teamcity.platform.bluexp.netapp.com/)

